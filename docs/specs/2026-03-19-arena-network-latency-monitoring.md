# Feature: Arena Network Latency Monitoring

## Summary
ESP32 buttons in the arena continuously ping AWS and report latency metrics to prevent connectivity issues from affecting live broadcasts. Arena operators can see real-time ping data in the backoffice dashboard to proactively troubleshoot network problems.

## User Journeys
1. Arena operator opens backoffice dashboard before/during live event
2. Operator sees table showing each ESP32 device with current ping times to AWS
3. When latency spikes or device goes offline, operator can troubleshoot before it affects the broadcast
4. ESP32 buttons automatically ping AWS backend at regular intervals and send results

## Acceptance Criteria
- [ ] ESP32 buttons ping AWS backend and send latency metrics
- [ ] Backend stores ping data from all arena devices  
- [ ] Backoffice displays real-time table of device ping times
- [ ] Operators can identify connectivity issues before broadcast problems occur

## Affected Projects
- smashlive--button: Add ping functionality to ESP32 code
- smashlive--infra: Backend endpoint to receive and store ping metrics
- smashlive--backoffice: Dashboard table displaying device latency data

## Constraints
V1 scope: Simple ping times only, displayed in basic dashboard table. No alerts, historical charts, or advanced troubleshooting features.