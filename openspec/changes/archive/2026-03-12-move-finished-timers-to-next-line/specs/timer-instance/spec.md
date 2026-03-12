## ADDED Requirements

### Requirement: Timer supports a completed state
Each timer instance SHALL support a `completed` app-state (in addition to `idle`, `running`, `paused`, and `alarm`). In `completed` state the timer shows a static `00:00`, inputs and preset buttons are locked, and the card sits in the Completed row.

#### Scenario: Completed state locks inputs
- **WHEN** a timer is in completed state
- **THEN** the minutes input, seconds input, and all preset buttons SHALL be disabled

#### Scenario: Completed state shows static display
- **WHEN** a timer is in completed state
- **THEN** the time display SHALL show `00:00` without flashing

### Requirement: Reset button behaviour is context-sensitive
The Reset button SHALL behave differently depending on the current `appState`.

#### Scenario: Reset from alarm transitions to completed
- **WHEN** a timer is in alarm state and the user clicks Reset
- **THEN** the timer SHALL transition to `completed` state: the flashing animation stops, the speech alarm cancels, `remainingSeconds` stays at 0, and the button label remains "Reset"

#### Scenario: Reset from completed transitions to idle
- **WHEN** a timer is in completed state and the user clicks Reset
- **THEN** the timer SHALL transition to `idle` state: inputs are unlocked, the display shows `00:00`, the button label changes to "Start", and the status message reads "Set a time and press Start"

#### Scenario: Reset from running, paused, or idle is unchanged
- **WHEN** a timer is in running, paused, or idle state and the user clicks Reset
- **THEN** the existing reset behaviour applies: timer stops, display clears to `00:00`, inputs unlock, state returns to `idle`
