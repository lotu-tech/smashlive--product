# Feature: Arena Network Latency Monitoring

## Summary
Enable real-time monitoring of network latency between IoT devices in arenas and AWS. Arena operators can view device health status in the backoffice to proactively troubleshoot connectivity issues before they impact live streams.

## User Journeys
1. **Arena Operator - Device Health Check**: Operator opens arena details page in backoffice, clicks "Latency" tab, sees list of all devices with latest ping times and color-coded health status (green: <100ms, yellow: 100-500ms, red: >500ms or offline)

2. **Arena Operator - Arena Overview**: Operator views summary dashboard showing all arenas with overall health status, can quickly identify which arenas have connectivity issues

3. **Troubleshooting During Event**: During a live stream, operator notices stream quality issues, checks latency tab to see if specific devices have high ping times, can proactively address network problems

## Acceptance Criteria
- [ ] IoT Edge mini PC pings all ESP32 buttons on local network every 30 seconds
- [ ] Ping results (latency in ms, timestamp, success/failure) are sent to AWS
- [ ] Backoffice arena details page has new "Latency" tab showing device health
- [ ] Device health uses color coding: green (<100ms), yellow (100-500ms), red (>500ms or offline)
- [ ] Summary dashboard shows arena-level health rollup
- [ ] 30 days of ping data retained for historical reference
- [ ] Real-time updates in backoffice (refresh every 30-60 seconds)

## Scale & Volume
- 1000 devices across 50+ arenas
- ~20 devices per arena
- Ping frequency: every 30 seconds
- Peak load: ~2000 pings per minute (~3M pings per day)
- Data retention: 30 days

## Affected Projects
- smashlive--infra: New Greengrass component for ping monitoring, IoT Core topic, Lambda for data ingestion, S3+Athena for time-series storage
- smashlive--backoffice: New latency tab on arena details page, arena health summary dashboard

## Constraints
- v1 scope: Basic device status display only
- Out of scope: Historical charts, alerting/notifications, device grouping by court
- Must not impact existing button functionality or network performance
- Data storage cost-optimized for high-volume time-series data