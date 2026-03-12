## Context

The timer container currently uses a single `flex-wrap` flex container. The `sortTimers()` function buckets timers into `active` (running/paused) and `inactive` (idle/alarm), then appends them all to one container. Alarm timers sit alongside idle ones with no visual separation, and there is no way to distinguish an alarm that is actively firing from one the user has acknowledged. Cards also have a fixed width of 290px, causing overflow on narrow viewports and wasted space on wide ones. When all three rows are populated they must all be visible within the viewport with no scrollbar.

A full vertical height audit of the card (computed precisely; base font 16px, `line-height: normal = 1.2×`):

| Section | Default | Tier 1 (≤1100px) | Tier 2 (≤950px) |
|---------|---------|------------------|-----------------|
| Card header (padding + btn-remove 26px + border) | 52px | 42px | 38px |
| Card body padding (top + bottom) | 34px | 16px | 12px |
| Dial (padding + `clamp` font + margin) | 78px | 52px | 46px |
| Preset pills (height + margin) | 36px | 24px | 20px |
| "Set time" ornamental divider + margin | 24px | 17px | hidden |
| Time setter (label: input + text + gap; + margin) | 67px | 59px | 57px |
| Control buttons (padding + font) | 36px | 30px | 26px |
| Status text (margin + min-height) | 25px | 18px | hidden |
| Card footer | 9px | 5px | hidden |
| **Total card** | **~362px** | **~263px** | **~199px** |
| Card height with `max-width: 280px`* | **~357px** | **~258px** | **~194px** |

\* At `max-width: 280px` the container query width is ≤278px → `10cqw ≤ 27.8px < 28.8px (1.8rem)`, so the `clamp()` expression always hits its floor. All cards use the same 28.8px font regardless of how many cards share a row — eliminating height inconsistency between rows.

With `max-width: 280px`:
- Three rows at Tier 1 + two dividers + four 8px gaps + 62px overhead = **~895px**
- Three rows at Tier 2 + two dividers + four 6px gaps + 50px overhead = **~687px**

A critical gap exists with Tier 2 at `max-height: 800px`: viewports of 800–895px receive Tier 1 layout (needs 895px) but cannot fit it. Raising Tier 2 to `max-height: 950px` closes this gap — all viewports 700–950px use the Tier 2 compact layout (needs 687px), and all viewports ≥951px use Tier 1 (needs 895px, fits with 56px headroom at 951px).

## Goals / Non-Goals

**Goals:**
- Three visually distinct rows: Active (Row 1), Alarm (Row 2), Completed (Row 3)
- Row 2 labelled "Alarm" — contains timers actively firing (flashing `00:00`)
- Row 3 labelled "Completed" — contains timers whose alarm the user has acknowledged via Reset
- Each row separator is only visible when its row is populated
- Reset from `alarm` → `completed` (stops flashing, moves to Row 3)
- Reset from `completed` → `idle` (clears display, moves back to Row 1)
- Timer cards resize fluidly with the viewport; the time display font scales with card width using `clamp()` so the most prominent element always looks proportional
- All three rows fit simultaneously within the viewport with no scrollbar at any common viewport size (≥ 700px `dvh`)
- The page itself never produces a browser scrollbar

**Non-Goals:**
- Pixel-perfect layout below 700px dvh
- Persisting timer history across page reloads
- Animations for row transitions
- Changes to the alarm sound or blinking behaviour on individual cards

## Decisions

**Decision: Two persistent sentinel divider elements injected into the single flex container**

Two `div.row-divider` nodes (`alarmDivider`, `completedDivider`) are created once outside `sortTimers()`. On each sort pass both are removed from the container, then re-inserted only when their respective row is non-empty. This keeps all card management inside `#timers-container` and avoids creating/destroying nodes on every tick.

Alternatives considered:
- *Three separate flex containers*: Cleaner semantically but requires structural HTML changes and coordinating three containers.
- *CSS `order` property alone*: Cannot force a line-break between groups without a sentinel element anyway.

**Decision: New `completed` app-state (distinct from `idle`)**

Adding `completed` as a fifth timer state (`idle | running | paused | alarm | completed`) avoids overloading `idle`. A `completed` timer shows static `00:00`, has locked inputs, and sits in Row 3. This makes the state machine unambiguous and the sort buckets clean.

**Decision: Two-tier `@media (max-height)` compact system — no scrollbar at any common viewport**

A single compact ruleset is insufficient: the card content totals ~362px and three rows need ~1286px, far exceeding a typical 900–1000px viewport. The solution is two progressive breakpoints:

- **`@media (max-height: 1100px)` — Moderate compact**: Reduces all internal card padding and spacing. Keeps all controls and status text visible. Card height reduces to ~258px (with 280px max-width). Three rows + overhead = ~895px; fits on any viewport ≥951px.
- **`@media (max-height: 950px)` — Aggressive compact**: Cumulative with the above. Additionally hides the decorative "set time" divider, the status text line, and the card footer strip (all cosmetic — controls remain fully usable). Card height reduces to ~194px. Three rows + overhead = ~687px; fits on any viewport ≥700px.

The threshold of 950px (not 800px as originally designed) is required because Tier 1 needs ~895px — viewports between 800px and 950px would otherwise receive Tier 1 but overflow. At viewports ≥951px Tier 1 alone suffices (895px needed, 889px container available at 951px — 56px headroom).

The `#timers-container` uses `overflow-y: scroll; scrollbar-width: none` so no scrollbar indicator ever appears, and the body uses `height: 100dvh; overflow: hidden` to prevent any page-level scroll. In the rare case of a viewport below 700px, content scrolls invisibly with the mouse wheel — the scrollbar itself is never shown.

**Decision: Proportional feel via `clamp()` on the time display font, not `aspect-ratio` on the card**

The card height is determined by its content — form elements have largely fixed heights. Enforcing `aspect-ratio` on the card would silently clip controls because the card already has `overflow: hidden` for its border radius.

Instead, `.time-display` uses `font-size: clamp(1.8rem, 10cqw, 3.6rem)` with `container-type: inline-size` on `.card`. This scales the countdown digits with card width, giving a proportional feel without clipping any controls.

The base flex hint is `flex: 1 1 200px`; `max-width` is capped at **280px**.

The 280px limit is load-bearing for height consistency: `10cqw` at 278px content width = 27.8px, which is below the `1.8rem` (28.8px) clamp floor. This means the time-display font is always 28.8px regardless of how many cards share a row. Without this cap, a single alarm card alone in Row 2 could grow to 340px (old max), rendering at a 33.8px font and producing a card ~5px taller than multi-card rows — a visible inconsistency across rows.

Alternatives considered:
- *`aspect-ratio` on `.card`*: Clips controls at small widths. Rejected.
- *CSS `zoom` / `transform: scale()`*: Would scale the entire card including interactive elements, causing hit-target issues. Rejected.
- *Single compact breakpoint*: Saves ~75px per card — insufficient. Rejected.

## Risks / Trade-offs

- [DOM mutation on every tick] Existing code already moves all cards on every tick; inserting/removing two dividers adds negligible overhead. A race condition was found during testing: if `setInterval` fires between `pointerdown` and `pointerup` on a button, `appendChild` moves the card away from the cursor before `click` fires, making the button unresponsive. Mitigated by a `clickInProgress` flag (set on `pointerdown`, cleared on `pointerup`) that causes `sortTimers()` to skip DOM reordering during an active pointer press. → Mitigated.
- [Accessibility] Both dividers are decorative/structural and carry `aria-hidden="true"`. → Mitigated.
- [Status text hidden at ≤950px dvh] The status line ("Running…", "Paused", etc.) is hidden in aggressive compact mode. Timer state is still fully communicated by the Start/Pause button label and the row the card is in. → Accepted.
- ["Set time" divider hidden at ≤950px dvh] The ornamental section label is decorative. Inputs remain fully visible and usable. → Accepted.
- [Card height inconsistency across rows] When rows contain different numbers of cards, flex-grow causes solo cards (typically alarm/completed rows) to be wider. With `clamp(1.8rem, 10cqw, 3.6rem)`, wider cards produced taller time-display fonts and thus taller cards. Fixed by capping `max-width: 280px` so `10cqw` never exceeds the `1.8rem` floor — all cards render the same font size and height. → Mitigated.
- [Below 700px dvh] Content may overflow the container; it scrolls invisibly (no scrollbar shown). This covers extreme edge cases only. → Accepted.
