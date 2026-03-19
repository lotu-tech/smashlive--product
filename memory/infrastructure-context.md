# SmashLive Infrastructure Overview

## System Architecture

SmashLive is a live sports streaming platform with IoT devices in arenas, cloud APIs on AWS, video streaming on GCP, and mobile + web frontends.

### Three Environments
- **Arena (edge)** — IoT Edge mini PCs (Greengrass), ESP32 buttons, IP cameras
- **AWS Cloud** — APIs (Lambda), databases (DynamoDB), IoT Core, media processing, CDN
- **GCP Cloud** — Video streaming VMs (FFmpeg → YouTube RTMP)

## Communication Patterns

### MQTT (IoT Core)
- ESP32 buttons publish to `buttons/<MAC>/action` (button presses, heartbeats every 10s)
- ESP32 buttons subscribe to `buttons/<MAC>/status` (LED state commands)
- Greengrass edge components receive commands via IoT Core topics
- IoT Core rules trigger Lambda functions on specific topic patterns

### REST APIs (API Gateway + Lambda)
- 24 Lambda API domains behind API Gateway
- Each domain has its own OpenAPI spec and Lambda handler in `platform-api/apis/<domain>-lambdas/`
- Terraform module `lambda-api` wires: API Gateway → Lambda → DynamoDB + other services
- All APIs use `operationName` routing (single Lambda per domain, routes internally)

### SQS / SNS
- Async processing via SQS queues (e.g., replay processing, notifications)
- Terraform module `lambda-sqs` wires: SQS → Lambda consumer

### S3 Events
- Media uploads trigger processing pipelines
- Replay clips, ad media, training labels stored in S3

## Edge Architecture (Arena)

### IoT Edge Mini PC (Greengrass)
- Each arena has 1 mini PC running AWS Greengrass
- Manages local devices (cameras, buttons) via local network
- **12 Greengrass components** (Python): health-check, manage-courts, start-live-manager, stop-live-manager, start-live-in-hk, stop-live-in-hk, upload-replay, replay-health-check, camera-reboot, button-firmware-update, credentials-check, remote-exec
- Components are defined in `smashlive--infra/iot-edge/`
- To add a new edge capability: create a Greengrass component recipe + Python script in `iot-edge/`

### ESP32 Buttons
- Physical devices with 2 buttons + 16 NeoPixel LEDs
- Connect via Ethernet (DM9051 SPI PHY) to arena LAN
- Communicate with AWS IoT Core over MQTT (TLS, port 8883)
- Button 1: replay trigger, Button 2: start/stop live stream
- Heartbeat every 10 seconds
- OTA firmware update capability
- **Buttons are passive targets for monitoring** — the edge mini PC can ping them on the local network
- Firmware lives in `smashlive--button/` (Arduino/C++)

### IP Cameras
- Managed by Greengrass components on the edge mini PC
- RTSP streams captured locally, forwarded to GCP VMs for YouTube streaming

## Cloud Services (AWS)

### DynamoDB Tables (27 tables)
Courts, Arenas, Buttons, Streamers, Cameras, Lives, Replays, CropJobs, YoutubeCredentials, Firmwares, Sponsors, Sponsorships, VirtualStreamers, AdsMedia, VideoAnalysis, TrainingLabels, CheckIns, UserEvents, UserProfiles, MonthlyLeaderboards, MonthlyInsights, OpsAgentSessions, Notifications, and more.

**When to use DynamoDB:** Low-volume key-value lookups, real-time state, entities with clear partition keys. PAY_PER_REQUEST pricing.

**When NOT to use DynamoDB:** High-volume time-series data (>100K writes/day), analytics queries, data that's write-heavy and read-infrequent. Use S3 + Athena instead.

### S3 Buckets
- Media storage (replays, ads, training data)
- Frontend static sites (backoffice, dashboard, scores, etc.)
- Can be used for time-series data with Athena queries (partitioned by date)

### Lambda Functions
- All APIs are Lambda-backed (Node.js 18.x, container images via ECR)
- Docker-deployed Lambdas: create-youtube-stream, replay
- Standard Lambdas: all 24 platform-api domains

### IoT Core
- MQTT broker for all device communication
- IoT rules trigger Lambda functions
- Cognito Identity for device auth (buttons) and browser MQTT (backoffice real-time health)

### Media Pipeline
- ECS Fargate for replay processing
- FFmpeg Lambda for video operations
- CloudFront CDN for media delivery
- S3 for storage

## Cloud Services (GCP)

### Video Streaming VMs
- Compute Engine VMs (e2-highcpu-4) running FFmpeg
- Stream RTSP (from cameras) → YouTube RTMP with scoreboard overlay
- Managed by `virtual-streamer-pool-manager` Lambda (AWS → GCP HTTP API)
- Each VM runs: streaming config service, scoreboard capture (Puppeteer), narrator WebSocket server
- **GCP is used for streaming because of cheaper egress bandwidth than AWS**

## Terraform Structure

### 5 State Roots (ADR-013)
1. `foundation/` — VPC, DNS, IAM, certs, OIDC
2. `platform-api/` — All Lambda APIs, DynamoDB, SQS, API Gateway
3. `iot-edge/` — Greengrass, IoT Core, edge components
4. `media-pipeline/` — ECS, FFmpeg, CloudFront, S3 media
5. `frontend/` — S3 + CloudFront for all static sites

### Terraform Modules
- `lambda-api` — API Gateway + Lambda + IAM + DynamoDB access
- `lambda-sqs` — SQS queue + Lambda consumer
- `component` — Greengrass component deployment
- `replay-lambda` — Specialized replay processing Lambda

### Adding New Infrastructure
- **New API endpoint:** Add handler in `platform-api/apis/<domain>-lambdas/main.js`, update OpenAPI spec, add Terraform in `platform-api/apis.tf`
- **New DynamoDB table:** Add in `platform-api/` Terraform root, add IAM policy
- **New Greengrass component:** Add recipe + script in `iot-edge/`, add Terraform in `iot-edge/`
- **New S3 bucket:** Add in appropriate Terraform root
- **New frontend:** Add S3 + CloudFront config in `frontend/`

## Data Flow Examples

### Live Stream Start
1. ESP32 button press → MQTT `buttons/<MAC>/action`
2. IoT Core rule → `create-youtube-stream` Lambda
3. Lambda: DynamoDB lookups (Button → Court → Camera → Streamer → YoutubeCredentials)
4. Lambda: Google API → create YouTube broadcast
5. Lambda: MQTT → edge mini PC (start-live-manager component)
6. Edge: starts RTSP camera capture
7. `virtual-streamer-pool-manager` Lambda → allocates GCP VM
8. GCP VM: receives config, starts FFmpeg (RTSP → YouTube RTMP + scoreboard overlay)

### Replay Capture
1. ESP32 button press → MQTT
2. IoT Core → `replay` Lambda
3. Lambda: determines time window from button/court/camera mapping
4. Lambda: MQTT → edge mini PC (upload-replay component)
5. Edge: captures clip from camera RTSP, uploads to S3
6. Media pipeline (ECS/FFmpeg Lambda): processes video
7. Processed clip → S3, metadata → DynamoDB

## Cost Considerations
- **DynamoDB:** PAY_PER_REQUEST — cheap for low-volume CRUD, expensive for high-volume writes (>100K/day)
- **S3 + Athena:** Cheap storage + pay-per-query — ideal for time-series, logs, analytics data
- **Lambda:** Pay per invocation — good for APIs, event handlers
- **GCP VMs:** Used specifically for streaming (cheaper egress than AWS)
- **IoT Core:** Pay per message — budget for heartbeats (1000 devices × 6/min = 6000 msgs/min)


## Device Discovery on Arena LAN

**Devices do NOT have static IPs.** The edge mini PC must discover devices dynamically.

### How existing components find devices
- **button-firmware-update** component: scans the local network (ARP discovery) to find ESP32 buttons by MAC address
- **Camera IPs**: stored in the Cameras DynamoDB table, associated with courts/arenas
- **Button MACs**: stored in the Buttons DynamoDB table, associated with courts

### Pattern for new components that need device IPs
1. Query DynamoDB for the arena devices (cameras by IP, buttons by MAC)
2. For buttons: use ARP to resolve MAC to IP on the local network
3. Cache results locally, refresh periodically (devices can change IPs via DHCP)
4. Do NOT hardcode IPs or pass them as static environment variables

### DynamoDB tables with device info
- Cameras: has ip field, linked to courts/arenas
- Buttons: has macAddress field, linked to courts
- Arenas: has arena metadata, the edge mini PC knows its arena ID from Greengrass config