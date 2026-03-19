# Feature: Arena Network Latency Monitoring

## Summary
Real-time network latency monitoring between arena IoT devices and the edge mini PC to help arena operators proactively identify connectivity issues during live events before streams break.

## User Journeys
1. Arena operator notices a stream quality issue during a live event
2. Operator opens backoffice and navigates to their arena details page
3. Operator clicks the new "Latency" tab to see current device status
4. Operator sees all cameras and buttons with latest ping results (green/yellow/red status indicators)
5. Operator identifies devices with high latency or offline status and can take corrective action

## Acceptance Criteria
- [ ] Edge mini PC pings all arena devices (cameras + buttons) every 30 seconds
- [ ] Mini PC dynamically discovers device IPs (cameras from DynamoDB, buttons via MAC→IP ARP resolution)
- [ ] Ping results stored in AWS with 30-day retention
- [ ] Backoffice shows "Latency" tab on arena details page
- [ ] Device status displayed with color coding: green (good), yellow (high latency), red (offline/timeout)
- [ ] Summary view shows overall arena connectivity health

## Scale & Volume
- 1000 devices across 50+ arenas
- Each mini PC pings ~20 devices every 30 seconds
- Peak load: 2000 pings/minute system-wide
- 30 days of historical data retention

## Affected Projects
- smashlive--infra: New Greengrass component for ping monitoring, new Lambda API for latency data, DynamoDB table for ping results
- smashlive--backoffice: New latency tab and summary page components

## Constraints
- V1 scope: Basic real-time status display only
- Out of scope: Historical charts, automated alerts, device grouping/filtering