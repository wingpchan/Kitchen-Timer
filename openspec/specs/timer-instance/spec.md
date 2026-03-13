## ADDED Requirements

### Requirement: Timer instance encapsulation
Each timer instance SHALL maintain its own isolated state: remaining seconds, interval handle, app state (idle | running | paused | alarm), and name. The object returned by the timer factory SHALL expose a `getState()` method that returns the current `remainingSeconds` and `appState` so external components (such as the timer manager) can read timer state without direct access to internal variables.

#### Scenario: Independent state
- **WHEN** a timer instance is created
- **THEN** it starts in idle state with 0 remaining seconds, independent of any other instance

#### Scenario: getState reflects current values
- **WHEN** `getState()` is called on a running timer
- **THEN** it SHALL return an object with the current `remainingSeconds` (as a number) and `appState` (as a string equal to `'running'`)

#### Scenario: getState after pause
- **WHEN** `getState()` is called on a paused timer
- **THEN** it SHALL return `appState: 'paused'` and `remainingSeconds` equal to the value at the moment of pausing

#### Scenario: getState on idle timer
- **WHEN** `getState()` is called on a timer that has never been started or has been reset
- **THEN** it SHALL return `appState: 'idle'` and `remainingSeconds: 0`

### Requirement: Start, pause, and resume
A timer instance SHALL support starting from idle or alarm state, pausing while running, resuming while paused, and restarting from completed state using the original duration.

#### Scenario: Start from idle
- **WHEN** the user presses Start on a timer with a non-zero time set
- **THEN** the timer transitions to running state and begins counting down; the original duration is saved internally for use by a later Restart

#### Scenario: Pause while running
- **WHEN** the user presses Pause on a running timer
- **THEN** the countdown halts and the timer enters paused state

#### Scenario: Resume from paused
- **WHEN** the user presses Resume on a paused timer
- **THEN** the countdown continues from where it was paused

#### Scenario: Restart from completed
- **WHEN** a timer is in completed state and the user clicks Restart
- **THEN** the timer SHALL transition to running state, restoring the original duration that was set when the timer was first started, and the display SHALL show that duration counting down

### Requirement: Reset
A timer instance SHALL return to idle state, clear the display to 00:00, unlock inputs, and cancel any active speech alarm when reset.

#### Scenario: Reset running timer
- **WHEN** the user clicks Reset on a running timer
- **THEN** the countdown stops, display shows 00:00, inputs are unlocked, and state is idle

#### Scenario: Reset alarm timer
- **WHEN** the user clicks Reset on a timer in alarm state
- **THEN** any active speech is cancelled, the blinking display clears, and state returns to idle

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

### Requirement: Preset time selection
A timer instance SHALL provide quick-select preset buttons (1m, 3m, 5m, 10m, 15m) that populate the time inputs when the timer is idle.

#### Scenario: Select preset while idle
- **WHEN** the user clicks a preset button while the timer is idle
- **THEN** the minutes input is set to the preset value, seconds set to 0, and the display updates

#### Scenario: Preset disabled while running
- **WHEN** the timer is in running or paused state
- **THEN** all preset buttons SHALL be disabled

### Requirement: Speech alarm
When a timer instance reaches zero, it SHALL speak the timer's name aloud using the Web Speech API and display a blinking alarm state.

#### Scenario: Countdown reaches zero
- **WHEN** the countdown reaches 0 seconds
- **THEN** the timer enters alarm state, the display blinks in the alarm color, and `speechSynthesis` speaks "Sweetie, the timer" the timer's name followed by "is done. Do whatever you need to do and not burn the thing!" in an angry Scottish lady's voice (e.g. "Sweetie, the timer Pasta is done. Do whatever you need to do and not burn the thing!")

#### Scenario: Speech fallback when unavailable
- **WHEN** `window.speechSynthesis` is not available in the browser
- **THEN** the timer SHALL fall back to a short oscillator beep alarm

#### Scenario: Speech cancelled on reset
- **WHEN** the user resets a timer that is in alarm state
- **THEN** `speechSynthesis.cancel()` is called to stop any in-progress speech immediately

### Requirement: onTick callback
The timer factory SHALL accept an optional `onTick` callback as a third argument. If provided, this callback SHALL be invoked at the end of each one-second tick (after `remainingSeconds` is decremented and the display updated, but before entering alarm state logic).

#### Scenario: onTick called each second while running
- **WHEN** a timer is running and one second elapses
- **THEN** the `onTick` callback SHALL be called once per tick

#### Scenario: onTick not called when timer is not running
- **WHEN** a timer is idle, paused, or in alarm state
- **THEN** the `onTick` callback SHALL NOT be called

#### Scenario: No onTick provided
- **WHEN** `createTimer` is called without a third argument
- **THEN** the timer SHALL function normally with no errors
