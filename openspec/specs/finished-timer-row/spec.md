## ADDED Requirements

### Requirement: Alarm timers appear in a dedicated Alarm row
The system SHALL display timers in `alarm` state in a dedicated Alarm row (Row 2) below all active and idle timers, visually separated by a full-width divider labelled "Alarm".

#### Scenario: Timer reaching zero moves to Alarm row
- **WHEN** a running timer reaches zero and enters alarm state
- **THEN** its card SHALL appear below the Alarm divider, separated from active and idle timers, with the `00:00` display flashing

#### Scenario: Alarm divider visible only when needed
- **WHEN** at least one timer is in alarm state
- **THEN** the Alarm divider SHALL be present in the DOM and visible
- **WHEN** no timers are in alarm state
- **THEN** the Alarm divider SHALL be absent from the DOM

#### Scenario: Reset from alarm transitions to completed state
- **WHEN** the user clicks Reset on an alarm-state timer
- **THEN** the flashing SHALL stop, the alarm sound SHALL cancel, and the card SHALL move to the Completed row (Row 3) — it SHALL NOT return directly to idle

### Requirement: Completed timers appear in a dedicated Completed row
The system SHALL display timers in `completed` state in a dedicated Completed row (Row 3) below the Alarm row, visually separated by a full-width divider labelled "Completed".

#### Scenario: Acknowledged alarm moves to Completed row
- **WHEN** the user presses Reset on an alarm-state timer
- **THEN** the timer enters `completed` state and its card SHALL appear below the Completed divider showing a static (non-flashing) `00:00`

#### Scenario: Completed divider visible only when needed
- **WHEN** at least one timer is in completed state
- **THEN** the Completed divider SHALL be present in the DOM and visible
- **WHEN** no timers are in completed state
- **THEN** the Completed divider SHALL be absent from the DOM

#### Scenario: Reset from completed returns timer to idle in Row 1
- **WHEN** the user clicks Reset on a completed-state timer
- **THEN** the timer SHALL return to `idle` state and its card SHALL move back to Row 1, with inputs unlocked and display showing `00:00`

### Requirement: Row dividers span full width and are accessible
Each row divider SHALL span the full container width and carry `aria-hidden="true"`.

#### Scenario: Divider forces a new flex row
- **WHEN** a row divider is present in `#timers-container`
- **THEN** it SHALL occupy 100% of the container width, forcing subsequent cards onto a new flex line

#### Scenario: Dividers not announced by screen readers
- **WHEN** a row divider is rendered
- **THEN** it SHALL carry `aria-hidden="true"` so assistive technology skips it

### Requirement: Timer cards resize fluidly with a consistent time display
Timer cards SHALL grow and shrink as the browser window is resized. Cards SHALL have a minimum width of 160px, a maximum width of 280px, and a flex base of 200px. The time display font SHALL use `clamp(1.8rem, 10cqw, 3.6rem)` — at 280px max-width the container query value `10cqw` is always below the `1.8rem` floor, so all cards render at the same font size regardless of which row they occupy, ensuring consistent card heights across rows.

#### Scenario: Cards fill available width on wide viewport
- **WHEN** the viewport is wide enough for multiple cards
- **THEN** cards SHALL expand to share the available row width rather than leaving unused space

#### Scenario: Cards respect minimum width on narrow viewport
- **WHEN** the viewport is narrowed
- **THEN** cards SHALL not shrink below 160px wide

#### Scenario: Cards respect maximum width on very wide viewport
- **WHEN** the viewport is very wide
- **THEN** cards SHALL not grow beyond 280px wide

#### Scenario: All cards are the same height regardless of which row they occupy
- **WHEN** timers are present in all three rows simultaneously
- **THEN** cards in the Alarm and Completed rows SHALL be the same height as cards in the Active/Idle row — no row SHALL contain visually taller or shorter cards than another

#### Scenario: Time display font is uniform across cards
- **WHEN** a card is alone in its row (e.g., a single alarm or completed timer)
- **THEN** its time display font SHALL be the same size as cards in a row containing multiple timers

### Requirement: All three rows fit the viewport without a visible scrollbar
When all three rows (Active, Alarm, Completed) are simultaneously populated, no scrollbar SHALL be visible. The page SHALL use a two-tier `@media (max-height)` compact system to reduce card spacing so all rows fit within common viewport heights (≥ 700px dvh). The body SHALL be constrained to `100dvh` with `overflow: hidden` and the timer container SHALL hide its scrollbar indicator.

#### Scenario: No scrollbar visible with all three rows
- **WHEN** timers are present in all three rows at the same time
- **THEN** no scrollbar SHALL be visible anywhere on the page

#### Scenario: Page height is contained to viewport
- **WHEN** the page is rendered
- **THEN** the outer page height SHALL be constrained to the viewport height (`100dvh`) so no browser-level scroll appears

#### Scenario: Moderate compact layout at ≤ 1100px viewport height
- **WHEN** the viewport height is 1100px or less
- **THEN** the system SHALL apply reduced card padding and spacing so three rows fit within a 951px+ viewport, with all controls and text still visible

#### Scenario: Aggressive compact layout at ≤ 950px viewport height
- **WHEN** the viewport height is 950px or less
- **THEN** the system SHALL apply further reduced padding and SHALL hide the decorative "set time" section divider, the status text line, and the card footer strip so three rows fit within a 700px+ viewport; all interactive controls (inputs, preset pills, Start/Reset buttons) SHALL remain visible and usable
