# Infrastructure Context

## IoT Architecture
- Each arena has an **IoT Edge mini PC** (running Greengrass) that manages local devices
- The mini PC pings all ESP32 buttons on the local network and measures latency
- The mini PC reports ping results to AWS via **IoT Core** messages (MQTT)
- The ESP32 buttons do NOT send pings themselves — they are passive targets
- Changes to ping monitoring go in **smashlive--infra** (Greengrass components + IoT Core + API), NOT smashlive--button

## Device Scale
- ~1000 ESP32 IoT devices across 50+ arenas
- Each arena has 1 IoT Edge mini PC that pings ~20 devices every 30 seconds
- Peak: all arenas active during live events = ~2000 pings/minute total
- Data retention: 30 days minimum for troubleshooting

## Data Volume Estimates
- ~2000 pings/min × 60 min × 8 hours/day = ~960K records/day
- Each record: ~200 bytes (device MAC, timestamp, latency ms, arena ID)
- Monthly: ~29M records, ~5.8 GB raw data

## Infrastructure Preferences
- Time-series data (ping logs): S3 + Athena (not DynamoDB — too expensive at this volume)
- Real-time status (latest ping per device): DynamoDB is fine (small dataset)
- IoT Edge components: Greengrass recipes in smashlive--infra/iot-edge/
- Terraform for all AWS resources
- Lambda for API handlers

