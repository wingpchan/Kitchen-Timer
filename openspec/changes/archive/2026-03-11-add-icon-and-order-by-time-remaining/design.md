## Context

The app is a single self-contained HTML file with inline CSS and JavaScript. Timer cards are appended to `#timers-container` in creation order and never reordered. The `createTimer` factory closes over `remainingSeconds` and `appState` privately; the manager currently only receives `{ id, card, btnRemove, destroy }`. There is no favicon.

## Goals / Non-Goals

**Goals:**
- Embed an SVG favicon in `<head>` so the app is identifiable in browser tabs with no external file dependency.
- Re-sort timer cards in the DOM after every tick: ascending remaining seconds among active (running/paused) timers, with alphabetical tiebreak by name; idle and alarm timers follow after.

**Non-Goals:**
- Animated or dynamic favicon reflecting live countdown state.
- User-controlled drag-to-reorder.
- Persisting sort order across page reloads.
- Visual transitions/animations when cards reorder (can be added later independently).

## Decisions

### 1. Favicon as inline SVG data-URI

Add `<link rel="icon" type="image/svg+xml" href="data:image/svg+xml,...">` in `<head>`, reusing the existing clock SVG shape from the page title. No external file; works as a local file with no server.

**Alternatives considered:**
- External `.svg` or `.ico` file — breaks the single-file constraint.
- PNG favicon — lower quality at small sizes and also requires an external file.

### 2. Expose state via `getState()` on the returned timer object

Add `getState()` to the object returned by `createTimer`. Returns `{ remainingSeconds, appState, name }`. The manager reads this to sort without accessing internal closure variables directly.

**Alternatives considered:**
- Reading the displayed time string from the DOM — fragile and error-prone.
- Event emitter / callback on state change — overkill; the manager only needs to sort once per second.

### 3. Inject `onTick` callback into `createTimer`

`createTimer(name, schemeClass, onTick)` accepts an optional third argument. At the end of each `tick()`, after decrementing and updating the display, `onTick?.()` is called. The manager passes its `sortTimers` function here.

**Alternatives considered:**
- Polling with a shared `setInterval` in the manager — introduces a timing race with individual timer intervals; wasteful.
- Calling `sortTimers` only on start/stop/pause events — misses continuous re-ranking during active countdowns.

### 4. Sort implementation in the manager

`sortTimers()` collects all timer entries from the `timers` Map, calls `getState()` on each, and builds a sort key:

- Active timers (`running` or `paused`): sort key = `remainingSeconds` (ascending), tiebreak = `name` (alphabetical ascending).
- Inactive timers (`idle` or `alarm`): sort key = `Infinity`, tiebreak = `name` (alphabetical ascending).

Reordering is done by calling `timersContainer.appendChild(timer.card)` for each timer in sorted order. Since `appendChild` moves an existing node, no cloning is needed. With up to 6 cards this is negligible DOM work.

## Risks / Trade-offs

- **Visual card jumping**: Cards will swap positions each second while countdowns run, which may feel disorienting. Acceptable for this change; CSS transitions on card movement can be added later independently.
- **SVG favicon browser support**: All modern browsers support SVG favicons (Chrome 80+, Firefox 72+, Safari 12+). Not supported in IE, which is already unsupported by the app.
- **DOM reorder on every tick**: Reordering 6 nodes once per second is negligible performance-wise.

## Migration Plan

1. Add `<link rel="icon">` to `<head>` — purely additive.
2. Update `createTimer(name, schemeClass, onTick)` to accept the optional `onTick` parameter and call it each tick; add `getState()` to the returned object — backwards compatible (existing callers pass no `onTick`).
3. Update `addTimer` in the manager to pass `sortTimers` as the `onTick` argument.
4. Implement `sortTimers()` in the manager scope.

No server changes. No data migration. Rollback = revert the single HTML file.
