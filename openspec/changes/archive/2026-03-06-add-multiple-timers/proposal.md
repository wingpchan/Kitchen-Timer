## Why

The app currently supports only one timer at a time, which is limiting for kitchen use where multiple dishes need simultaneous tracking. Adding multiple independent timers lets users run parallel countdowns without losing any active timer.

## What Changes

- Replace the single timer widget with a dynamic list of timer instances
- Each timer instance is fully independent: its own countdown, controls, label, alarm, and state
- Users can add new timers (up to a reasonable limit, e.g. 6)
- When user add new timer allow them to give the timer a name, up to 15 characters long, once rendered it will be a read only text
- Each timer should have a different colour scheme and pattern
- Users can remove individual timers
- Each timer retains all existing functionality: presets, manual input, start/pause/resume/reset, alarm
- The alarm should change from beeps to actually be spoken words of the name given by the user

## Capabilities

### New Capabilities

- `multi-timer-manager`: Manages a collection of timer instances — add, remove, and render multiple timers independently on the page

### Modified Capabilities

- `timer-instance`: The existing single-timer logic is extracted into a reusable per-instance component (state, DOM, audio) instead of being global

## Impact

- `index.html`: Significant refactor — global timer state becomes per-instance, layout expands horizontally or wraps to accommodate multiple cards
- No new dependencies
- Theme toggle and theme persistence remain unchanged
