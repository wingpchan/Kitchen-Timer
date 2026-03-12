## Why

When a timer finishes it stays in its current position among running timers, making it hard to notice and cluttering the active view. Users also have no way to distinguish an alarm that is actively firing from one they have acknowledged. Introducing three distinct rows gives each timer lifecycle stage its own space and makes the overall state of all timers immediately clear.

Additionally, timer cards currently have a fixed width of 290px. On wider screens this wastes space, and on narrow screens cards overflow. Making cards fluid lets the layout fill the available window width gracefully while staying usable at any size.

## What Changes

- **Row 1 — Active**: Running, paused, and idle timers live here. Newly created timers appear here. Sort order: running/paused first (ascending time remaining), then idle (alphabetically by name).
- **Row 2 — Alarm**: When a timer reaches zero its card moves here. The `00:00` display flashes, the speech alarm fires, and the timer stays here until the user acknowledges it. A labelled separator ("Alarm") divides Row 1 from Row 2.
- **Row 3 — Completed**: When the user presses **Reset** on an alarm-state timer, the flashing stops and the card moves to this row as a static "done" record showing `00:00`. A labelled separator ("Completed") divides Row 2 from Row 3. From this row the user can press **Reset** again to return the timer to idle in Row 1, or remove it entirely.
- Each separator is only visible when its row contains at least one timer.
- A new `completed` app-state is introduced for timers in Row 3 (distinct from `idle`, `alarm`, `running`, `paused`).
- **Responsive card sizing**: Timer cards resize fluidly as the browser window is resized. The time display font scales with card width using `clamp()` so the countdown digits always look proportional. A maximum width of 280px ensures all cards render at the same size regardless of how many cards share a row. A minimum width is enforced so controls remain usable, and all three rows fit within the visible viewport simultaneously without triggering a scrollbar.

## Capabilities

### New Capabilities
- `timer-rows`: Three-row layout for the timer container — Active (Row 1), Alarm (Row 2), and Completed (Row 3) — each separated by a labelled full-width divider that appears only when the row is populated. All three rows SHALL be visible simultaneously within the viewport without scrolling.

### Modified Capabilities
- `multi-timer-manager`: Sort logic must now bucket timers into three groups (active, alarm, completed) and inject two conditional dividers between them.
- `timer-instance`: The Reset button must behave differently depending on state — from `alarm` it transitions to `completed` (Row 3); from `completed` it transitions to `idle` (Row 1). A new `completed` state is added to the timer state machine. Card layout must be fluid with proportional width-and-height scaling and a small minimum size.

## Impact

- `sortTimers()` in `index.html` — three-bucket sort with two injected divider elements
- `resetTimer()` in `index.html` — split into two actions based on current `appState`
- Timer state machine gains a `completed` state
- `.card` CSS — fluid sizing with `clamp()` font scaling and a 280px maximum width; all cards render consistently regardless of row, and all three rows fit the viewport
- `body` / `#timers-container` CSS — layout must prevent overflow scrollbars when all three rows are populated
- CSS additions for `.row-divider` and its two variants (alarm, completed)
- No new external dependencies; pure HTML/CSS/JS change
