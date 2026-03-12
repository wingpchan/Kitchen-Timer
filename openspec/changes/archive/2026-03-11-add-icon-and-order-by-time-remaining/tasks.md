## 1. Favicon

- [x] 1.1 Add `<link rel="icon" type="image/svg+xml">` to `<head>` with an inline data-URI SVG clock matching the existing page-title icon shape

## 2. Timer Instance — expose state and onTick

- [x] 2.1 Update `createTimer(name, schemeClass, onTick)` signature to accept optional third `onTick` argument
- [x] 2.2 Add `getState()` to the object returned by `createTimer`, returning `{ remainingSeconds, appState, name }`
- [x] 2.3 Call `onTick?.()` at the end of each `tick()` invocation (after display update, before alarm logic fires)

## 3. Timer Manager — sort by time remaining

- [x] 3.1 Implement `sortTimers()` in the manager scope: read `getState()` from all entries in the `timers` Map, sort active timers (running/paused) ascending by `remainingSeconds` with alphabetical-by-name tiebreak, place idle/alarm timers after
- [x] 3.2 Apply the sort by calling `timersContainer.appendChild(timer.card)` for each timer in sorted order
- [x] 3.3 Update `addTimer` to pass `sortTimers` as the `onTick` argument when calling `createTimer`

## 4. Verification

- [x] 4.1 Verify favicon appears in the browser tab when the file is opened locally
- [x] 4.2 Verify two running timers reorder correctly each second (shorter time moves to front)
- [x] 4.3 Verify tiebreak: two timers with identical remaining seconds sort alphabetically by name
- [x] 4.4 Verify idle and alarm timers always appear after all running/paused timers
- [x] 4.5 Verify a single-timer page works normally with no errors
