## MODIFIED Requirements

### Requirement: Sort timer cards by time remaining
The timer manager SHALL re-sort the visible timer cards in `#timers-container` after every tick so that cards are ordered: (1) active timers (running/paused) by ascending remaining seconds, (2) idle timers alphabetically by name, (3) the Alarm divider + alarm-state timers (if any), (4) the Completed divider + completed-state timers (if any).

#### Scenario: Running timer with less time moves to front
- **WHEN** two timers are running and timer B has fewer seconds remaining than timer A
- **THEN** timer B's card SHALL appear before timer A's card in the DOM and visually on screen

#### Scenario: Idle timers sort after active timers, before Alarm row
- **WHEN** a mix of running, paused, and idle timers are present with no alarm or completed timers
- **THEN** running and paused timers SHALL appear before idle timers, and neither row divider SHALL be present

#### Scenario: Alarm timers appear in Row 2 below active and idle timers
- **WHEN** one or more timers are in alarm state alongside active or idle timers
- **THEN** the Alarm divider SHALL appear after all active and idle cards, with alarm-state cards following it

#### Scenario: Completed timers appear in Row 3 below Alarm row
- **WHEN** one or more timers are in completed state
- **THEN** the Completed divider SHALL appear after any alarm-state cards (or after idle cards if no alarm timers), with completed-state cards following it

#### Scenario: Both Alarm and Completed rows present simultaneously
- **WHEN** at least one timer is in alarm state AND at least one is in completed state
- **THEN** the order SHALL be: active/idle cards → Alarm divider → alarm cards → Completed divider → completed cards

#### Scenario: Timers with equal remaining time sort alphabetically by name
- **WHEN** two timers have the same remaining seconds
- **THEN** sort them alphabetically by name, such as 'a' before 'b'

#### Scenario: Single timer — no reorder needed
- **WHEN** only one timer is present
- **THEN** no sorting operation is performed and the card position is unchanged

#### Scenario: Sort updates on every tick
- **WHEN** a running timer's tick fires
- **THEN** the manager SHALL re-evaluate and reorder all cards immediately after that tick

### Requirement: All rows visible within viewport simultaneously
When all three rows are populated, the timer container layout SHALL ensure all cards remain visible within the viewport without producing a vertical scrollbar.

#### Scenario: Three populated rows fit without scrollbar
- **WHEN** Row 1 (Active), Row 2 (Alarm), and Row 3 (Completed) all contain at least one timer card
- **THEN** all rows SHALL be visible on screen simultaneously and no vertical scrollbar SHALL appear on the page
