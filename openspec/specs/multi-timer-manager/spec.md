## ADDED Requirements

### Requirement: Add new timer with name
The system SHALL allow the user to add a new independent timer instance to the page (up to a maximum of 6 timers) by entering a name of up to 15 characters before the timer is created.

#### Scenario: Add first additional timer with name
- **WHEN** the page loads with one default timer and the user clicks "Add Timer", enters a name, and confirms
- **THEN** a second timer card appears with that name displayed in its header, fully initialized and ready to use

#### Scenario: Name capped at 15 characters
- **WHEN** the user types more than 15 characters into the name input
- **THEN** the input SHALL not accept characters beyond the 15th

#### Scenario: Add timer with empty name
- **WHEN** the user confirms the add prompt without entering a name
- **THEN** the timer SHALL be created with a default name (e.g. "Timer N" where N is the timer number)

#### Scenario: Add timer at maximum
- **WHEN** 6 timers are already present
- **THEN** the "Add Timer" button SHALL be disabled and no new timer is added on click

#### Scenario: Add timer below maximum
- **WHEN** fewer than 6 timers are present and the user confirms adding a timer
- **THEN** a new timer card is appended and the "Add Timer" button remains enabled if the count is still below 6

### Requirement: Timer name is read-only after creation
Once a timer card is rendered, its name SHALL be displayed as static text in the card header and SHALL NOT be editable.

#### Scenario: Name rendered as static text
- **WHEN** the timer card is created with a given name
- **THEN** the name appears in the card header as non-interactive text

### Requirement: Per-timer color scheme
Each timer card SHALL be assigned a distinct color scheme and decorative header pattern from a predefined set of 6 schemes.

#### Scenario: First timer gets first scheme
- **WHEN** the first timer is created on page load
- **THEN** it uses the first color scheme (terracotta/amber gradient with diagonal stripe pattern)

#### Scenario: Each additional timer uses a different scheme
- **WHEN** a second or subsequent timer is added
- **THEN** its card header gradient and pattern visually differ from all other currently visible timers

### Requirement: Remove individual timer
The system SHALL allow the user to remove any individual timer card from the page, provided at least one timer remains.

#### Scenario: Remove one of multiple timers
- **WHEN** 2 or more timers are present and the user clicks the remove button on a timer card
- **THEN** that timer card is destroyed (its interval stopped, speech alarm cancelled) and removed from the DOM

#### Scenario: Remove last timer blocked
- **WHEN** only 1 timer remains
- **THEN** the remove button on that timer card SHALL be disabled

### Requirement: Default single timer on load
The system SHALL initialize with exactly one timer card when the page first loads.

#### Scenario: Page load state
- **WHEN** the user opens the page
- **THEN** exactly one timer card is visible, in idle state, displaying 00:00, with a default name in its header

### Requirement: Timers are independent
Each timer instance SHALL operate completely independently of all other timers.

#### Scenario: Two timers run simultaneously
- **WHEN** two timers are both in the running state
- **THEN** each countdown proceeds at its own rate without affecting the other

#### Scenario: Alarm on one timer does not affect others
- **WHEN** one timer reaches zero and enters alarm state
- **THEN** all other timers continue their current state unchanged

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
