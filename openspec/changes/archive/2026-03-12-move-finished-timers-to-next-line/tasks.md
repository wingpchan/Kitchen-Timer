## 1. CSS — Row Dividers

- [x] 1.1 Add `.row-divider` base rule: `width: 100%; flex-basis: 100%` to force a flex line-break, plus ornamental label styling matching the existing `.divider` style used inside cards
- [x] 1.2 Add `.row-divider--alarm` variant with an amber/red accent colour
- [x] 1.3 Add `.row-divider--completed` variant with a muted/neutral accent colour

## 2. CSS — Responsive Card Sizing & Viewport Fit

- [x] 2.1 Update `.card` sizing: `flex: 1 1 200px; min-width: 160px; max-width: 340px`
- [x] 2.2 Replace fixed `font-size: 3.6rem` on `.time-display` with `clamp(1.8rem, 10cqw, 3.6rem)` and add `container-type: inline-size` to `.card`
- [x] 2.3 Set `body` to `height: 100dvh; overflow: hidden; display: flex; flex-direction: column` and `#timers-container` to `flex: 1; overflow-y: scroll; scrollbar-width: none; min-height: 0`
- [x] 2.4 Replace the existing single `@media (max-height: 1100px)` compact block with a two-tier system:
  - **Tier 1 `@media (max-height: 1100px)`** — moderate compact: reduce `body` padding to 10px, `#page-header` margin-bottom to 10px, container gap to 8px, `.row-divider` margin to 0, `.card-header` padding to 8px top/bottom, `.card-body` padding to 8px, `.dial` padding to 6px and margin-bottom to 6px, `.preset-btn` padding to 3px, `.presets` margin-bottom to 5px, `.divider` margin-bottom to 5px, `.time-setter` margin-bottom to 6px, `.btn` padding to 7px top/bottom, `.status` margin-top to 4px, `.card-footer` height to 4px
  - **Tier 2 `@media (max-height: 800px)`** — aggressive compact (cumulative): reduce `body` padding to 6px, `#page-header` margin-bottom to 6px, container gap to 6px, `.card-header` padding to 6px, `.card-body` padding to 6px, `.dial` padding to 4px and margin-bottom to 4px, `.preset-btn` padding to 2px, `.presets` margin-bottom to 3px, `.time-setter` margin-bottom to 4px, `.btn` padding to 5px top/bottom; and set `.divider { display: none }`, `.status { display: none }`, `.card-footer { display: none }`

## 3. JS — Timer State Machine: add `completed` state

- [x] 3.1 Add `completed` to the state comment: `idle | running | paused | alarm | completed`
- [x] 3.2 In `completed` state, inputs and preset buttons remain locked (same as `running`/`alarm`)

## 4. JS — Context-sensitive Reset behaviour

- [x] 4.1 When `appState === 'alarm'`, Reset transitions to `completed`: stop flashing, cancel alarm, set `appState = 'completed'`, keep display at `00:00`, set status to "Finished"
- [x] 4.2 When `appState === 'completed'`, Reset transitions to `idle`: unlock inputs, reset display to `00:00`, set button label to "Start", set status to "Set a time and press Start"

## 5. JS — sortTimers(): three-row sort with two dividers

- [x] 5.1 Create two persistent divider DOM elements (`alarmDivider`, `completedDivider`) once outside `sortTimers()`, each with `aria-hidden="true"` and the correct class and label text
- [x] 5.2 Split the existing two-bucket logic into four buckets: `active` (running/paused), `idle`, `alarm`, `completed`
- [x] 5.3 Remove both dividers from the container at the start of each sort pass
- [x] 5.4 Append in order: active cards → idle cards → `alarmDivider` + alarm cards (only if alarm.length > 0) → `completedDivider` + completed cards (only if completed.length > 0)

## 6. Bug Fix — Button Click Race Condition

- [x] 6.0 Add a `clickInProgress` flag: set to `true` on `document` `pointerdown`, cleared to `false` on `pointerup`; add an early-return guard at the top of `sortTimers()` so DOM reordering is skipped while a pointer press is active

## 7. Fix — Viewport Overflow & Card Size Inconsistency

- [x] 7.1 Change `.card { max-width: 340px }` → `max-width: 280px` so `10cqw` never exceeds the `1.8rem` clamp floor — all cards render at the same font size and height regardless of row population
- [x] 7.2 Change Tier 2 media query from `@media (max-height: 800px)` → `@media (max-height: 950px)` to close the 800–950px overflow gap where Tier 1 needs ~895px but the viewport cannot fit it

## 8. Verification

- [x] 8.1 Running timer reaches zero → card moves below Alarm divider with flashing display
- [x] 8.2 Reset on alarm timer → flashing stops, card moves below Completed divider showing static `00:00`
- [x] 8.3 Reset on completed timer → card returns to Row 1 in idle state with unlocked inputs
- [x] 8.4 Alarm divider absent when no alarm timers; Completed divider absent when no completed timers
- [X] 8.5 Cards resize fluidly on window resize; minimum 160px and maximum 280px widths are respected
- [x] 8.6 Focus on an input is not lost when another timer changes state (existing fix still works)
- [X] 8.7 All cards display at the same height — cards in the Alarm and Completed rows are not taller or shorter than cards in the Active row
- [X] 8.8 With timers in all three rows simultaneously, no scrollbar appears on any viewport ≥ 700px dvh
- [x] 8.9 Pause and Reset buttons respond reliably to clicks even when a sort tick fires during the pointer press
