# SmashLive Domain Overview

SmashLive is a live sports streaming platform used in arenas for futsal, basketball, volleyball and other sports. It automates live YouTube streaming, instant replays, and arena management.

## Repositories and Responsibilities

| Repo | What it does | Stack |
|------|-------------|-------|
| smashlive--infra | ALL backend: APIs, databases, IoT, Greengrass edge components, media pipeline | Terraform + Node.js Lambdas + Python (edge) |
| smashlive--app | Mobile app for viewers (live streams, replays, check-ins, leaderboards) | Expo React Native + TypeScript |
| smashlive--backoffice | Admin dashboard for arena operators (device management, monitoring, config) | React + Vite + MUI + TypeScript |
| smashlive--button | ESP32 physical button firmware (2 buttons + LED strip) | Arduino C++ |
| smashlive--gcp--infra | GCP video streaming VMs (FFmpeg → YouTube) | Terraform + Packer + Node.js |
| smashlive--create-youtube-stream | Lambda: creates/stops YouTube broadcasts on button press | TypeScript Docker Lambda |
| smashlive--replay | Lambda: triggers replay clip capture on button press | TypeScript Docker Lambda |
| smashlive--youtube-credentials | Web app for arena owners to authorize YouTube accounts | React + Vite |

## Key Domain Concepts

- **Arena** — Physical venue with courts, cameras, buttons, and an edge mini PC
- **Court** — A playing area within an arena, has cameras and buttons assigned
- **Button** — ESP32 IoT device. Button 1 = replay, Button 2 = start/stop stream
- **Streamer** — A YouTube channel associated with an arena
- **Edge Mini PC** — Greengrass device in each arena managing local hardware
- **Live** — An active YouTube broadcast from an arena camera
- **Replay** — A short video clip captured from a camera's recent footage

## What Changes Go Where

- New API endpoint → smashlive--infra (platform-api/)
- New DynamoDB table → smashlive--infra (platform-api/)
- New edge device capability → smashlive--infra (iot-edge/)
- New backoffice page → smashlive--backoffice
- New mobile feature → smashlive--app
- Button firmware change → smashlive--button
- Streaming/encoding change → smashlive--gcp--infra
