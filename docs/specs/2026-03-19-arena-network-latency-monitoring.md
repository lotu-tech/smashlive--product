# Feature: Arena Network Latency Monitoring

## Summary
Real-time dashboard in the backoffice showing ESP32 device ping latency to prevent stream failures during live events. Arena operators can spot network issues before they affect broadcasts, with color-coded status indicators and arena health summaries.

## User Journeys
1. Arena operator opens backoffice before a live event
2. Navigates to their arena page and clicks the "Latency" tab
3. Views table of all ESP32 devices with current ping times (green <200ms, yellow 200-500ms, red >500ms or offline)
4. Identifies devices with high latency or offline status
5. Physically checks problematic devices before event starts

6. Ops team member opens summary page showing all arenas
7. Views overall health status per arena to identify problem locations
8. Escalates issues to relevant arena operators

## Acceptance Criteria
- [ ] Backoffice displays "Latency" tab on arena pages showing all ESP32 devices
- [ ] Device table shows: device ID, last ping time, latency in ms, status color (green/yellow/red)
- [ ] Color coding: green (<200ms), yellow (200-500ms), red (>500ms or offline)
- [ ] Summary page shows all arenas with overall health indicators
- [ ] Data updates in real-time (refreshes every 30 seconds or less)
- [ ] 30 days of ping history stored for troubleshooting

## Scale & Volume
1000 ESP32 devices across 50+ arenas, each pinging every 30 seconds. Peak load: 2000 pings/minute during live events. Data retention: 30 days minimum (~29M records/month, ~5.8 GB).

## Affected Projects
- smashlive--backoffice: Add Latency tab to arena pages, summary health page
- smashlive--infra: API endpoints for ping data, time-series storage (S3 + Athena recommended)

## Constraints
Must integrate with existing React + Vite + MUI backoffice. No new frontend frameworks. Out of scope for v1: historical charts, alerting/notifications, device grouping by location.