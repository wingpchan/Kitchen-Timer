## Why

The app lacks a favicon, making it hard to identify among browser tabs, and when multiple timers are running simultaneously the most urgent timer can be buried anywhere in the grid. Adding a favicon improves tab recognition, and automatically ordering timer cards by time remaining lets users see the most urgent timers first at a glance.

## What Changes

- Add an inline SVG favicon to the page `<head>` so the Kitchen Timer app has a recognizable icon in browser tabs and bookmarks.
- After each tick, re-sort the DOM order of timer cards by ascending remaining seconds (least time remaining → shown first), keeping idle timers grouped after active ones.

## Capabilities

### New Capabilities
- `favicon`: SVG favicon displayed in the browser tab derived from the existing clock SVG used in the page title.

### Modified Capabilities
- `timer-instance`: Each timer instance must expose its current `remainingSeconds` and `appState` so the manager can sort cards. The sort key logic lives in the manager layer, not inside individual timer instances.
- `multi-timer-manager`: The manager gains responsibility for re-sorting the `#timers-container` DOM after every tick across all timers, ordering cards by remaining seconds ascending (running/paused timers with time > 0 first, then idle/alarm timers).

## Impact

- `index.html` — `<head>` gains a `<link rel="icon">` with a data-URI SVG favicon.
- `createTimer` factory — must return (or expose) `remainingSeconds` and `appState` so the manager can read them for sorting.
- `addTimer` / timer manager — gains a `sortTimers()` function that reads each timer's state and re-orders the DOM children of `#timers-container`.
- No new dependencies; no breaking changes to the public timer API visible to end users.
