## MODIFIED Requirements

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

## ADDED Requirements

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
