# Infrastructure Context

## Device Scale
- ~1000 ESP32 IoT devices across arenas
- Devices ping every 30 seconds when active
- Peak: all devices active during live events = ~2000 pings/minute
- Data retention: 30 days minimum for troubleshooting

## Data Volume Estimates
- 1000 devices × 2 pings/min × 60 min × 8 hours/day = ~960K records/day
- Each record: ~200 bytes (device ID, timestamp, latency ms, arena ID)
- Monthly: ~29M records, ~5.8 GB raw data

## Infrastructure Preferences
- Time-series data (ping logs, metrics): prefer S3 + Athena over DynamoDB (cost)
- Real-time lookups (current device status): DynamoDB is fine (small dataset)
- Terraform for all AWS resources
- Lambda for API handlers

