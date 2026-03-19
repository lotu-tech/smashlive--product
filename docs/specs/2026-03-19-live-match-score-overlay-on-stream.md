# Feature: Live Match Score Overlay

## Summary
A real-time score overlay system that displays team names, current score, and match time at the bottom of YouTube live streams. Arena operators control the overlay through the backoffice, and updates appear on stream within 2-3 seconds to create a professional viewing experience.

## User Journeys
1. Arena operator opens backoffice during live match
2. Operator manually starts match clock from backoffice
3. When team scores, operator clicks increment button for that team in backoffice
4. Score overlay on YouTube stream updates within 2-3 seconds
5. Operator can toggle overlay on/off as needed
6. Operator can manually correct scores if needed during match

## Acceptance Criteria
- [ ] Overlay appears as simple bar at bottom of YouTube stream
- [ ] Displays team names, current score, and elapsed match time
- [ ] Score updates appear on stream within 2-3 seconds of backoffice input
- [ ] Works for any sport (futsal, basketball, volleyball)
- [ ] Arena operators can toggle overlay visibility on/off
- [ ] Arena operators can manually start/stop match clock
- [ ] Arena operators can correct scores manually during match

## Affected Projects
- smashlive--backoffice: Score input interface, overlay controls
- smashlive--infra: Real-time data pipeline for score updates
- smashlive--create-youtube-stream: Overlay rendering on stream

## Constraints
- Must update within 2-3 seconds
- V1 does not include automatic period/quarter tracking
- Simple bar design at bottom of stream