# Feature: Arena Network Ping Monitoring Dashboard

## Summary
Add network ping monitoring to the backoffice so technical support teams can proactively identify arena connectivity issues before button failures impact live events. The dashboard will track ping times between arena devices and AWS to catch degradation caused by terminal corrosion or other network issues.

## User Journeys
1. Technical support team member logs into backoffice
2. Views summary dashboard showing all arenas with current ping status and highlights any alarming connectivity
3. Clicks on specific arena to see detailed ping metrics on arena details page
4. Identifies arena with degraded ping times before complete button failure
5. Takes corrective action to resolve connectivity issue proactively

## Acceptance Criteria
- [ ] Arena details page displays current ping time from arena devices to AWS
- [ ] Summary dashboard lists all arenas with ping status and highlights those exceeding alarm threshold
- [ ] System collects baseline ping data to establish normal operating ranges
- [ ] Initial alarm threshold set at 10ms (adjustable based on baseline data)
- [ ] Dashboard accessible to technical support role in backoffice
- [ ] Ping data refreshes automatically to show current status

## Affected Projects
- backoffice: Add ping monitoring views to arena details page and new summary dashboard
- arena-infrastructure: Implement ping collection and reporting from arena devices to AWS

## Constraints
- 10-day delivery timeline
- Must baseline existing ping data before setting final alarm thresholds
- Out of scope: Automated remediation actions, historical trend analysis