## 1. Refactor Timer Logic into Factory Function

- [x] 1.1 Extract all global timer state (`remainingSeconds`, `intervalId`, `appState`, `name`) into a `createTimer(id, name, scheme)` factory function that returns a self-contained timer object
- [x] 1.2 Move all timer DOM references inside the factory (no more singleton IDs like `#time-display`) — generate unique IDs per instance using the `id` param
- [x] 1.3 Extract `createTimerDOM(id, name, scheme)` helper that returns a fully-built timer card element with name displayed in the header and scheme colors/patterns applied
- [x] 1.4 Ensure all timer methods (`start`, `pause`, `reset`, `tick`) are self-contained within the factory closure

## 2. Color Schemes

- [x] 2.1 Define a `TIMER_SCHEMES` array with 6 entries, each specifying accent-a color, accent-b color, and a unique SVG background pattern string for the card header
- [x] 2.2 Schemes: terracotta/amber (diagonal stripes), teal/emerald (dots), indigo/violet (chevrons), rose/pink (waves), slate/steel (crosshatch), olive/moss (hexagons)
- [x] 2.3 Apply the scheme to a card by injecting `style` attributes (or CSS custom properties) onto the card's root element in `createTimerDOM`
- [x] 2.4 Assign scheme by `timerId % 6` so each timer gets a distinct scheme and wrap-around is deterministic

## 3. Timer Naming

- [x] 3.1 Add an inline name-prompt UI to the page (hidden by default): a styled text input (maxlength=15) and a "Create" confirm button
- [x] 3.2 Show the name prompt when the user clicks "Add Timer"; hide it and call `addTimer(name)` when the user confirms
- [x] 3.3 If the user confirms with an empty name, default to "Timer N" where N is the one-based count of timers ever created
- [x] 3.4 Render the name as a static `<span>` in the card header (not an input), positioned between the timer icon and the remove button

## 4. Speech Alarm

- [x] 4.1 Remove the `AudioContext` oscillator alarm implementation
- [x] 4.2 Implement `speakAlarm(name)`: creates a `SpeechSynthesisUtterance` with text `"Sweetie, the timer <name> is done. Do whatever you need to do and not burn the thing!"` and calls `speechSynthesis.speak()`
- [x] 4.3 Implement `cancelAlarm()`: calls `speechSynthesis.cancel()` to stop in-progress speech
- [x] 4.3a Select a Scottish female voice: after `voiceschanged` fires (or synchronously if voices are already loaded), filter `speechSynthesis.getVoices()` for a voice with `lang` starting with `"en-GB"` and name containing "Scottish", "Female", "Fiona", or "Moira"; fall back to first `en-GB` voice, then browser default
- [x] 4.4 Add a fallback: if `window.speechSynthesis` is undefined, play a single short oscillator beep instead
- [x] 4.5 Call `cancelAlarm()` in the timer's `reset()` method when transitioning out of alarm state

## 5. Build the Timer Manager

- [x] 5.1 Add a `#timers-container` div to `index.html` with `display: flex; flex-wrap: wrap; gap: 16px` layout
- [x] 5.2 Implement `addTimer(name)` function: assigns next scheme, calls `createTimer()`, appends its DOM to `#timers-container`, registers all event listeners, and updates "Add Timer" button disabled state
- [x] 5.3 Implement `removeTimer(id)` function: stops the timer, cancels speech alarm, removes its DOM element, and updates "Add Timer" and remove button disabled states
- [x] 5.4 Disable the remove button on a card when it is the last remaining timer
- [x] 5.5 Disable the "Add Timer" button when 6 timers are present

## 6. Page-Level UI Changes

- [x] 6.1 Add a page header above `#timers-container` containing the "Add Timer" button and the theme toggle
- [x] 6.2 Add a remove ("×") button to each timer card header, styled to match the header aesthetic
- [x] 6.3 Update the page `<body>` layout to accommodate a row of cards (remove single-card centering)
- [x] 6.4 Remove the theme toggle from individual timer card headers (it now lives in the page header only)

## 7. Keyboard Shortcuts

- [x] 7.1 Remove the global `document` Space/Escape keydown listener
- [x] 7.2 Add `tabindex="0"` to each timer card element so it can receive focus
- [x] 7.3 Attach keydown listeners to each card: Space = start/pause, Escape = reset (only when card is focused)

## 8. Initialization and Smoke Testing

- [x] 8.1 On page load, call `addTimer("Timer 1")` once to initialize the default single-timer experience with scheme 0
- [ ] 8.2 Verify single timer: start, pause, resume, preset, reset, alarm speech all work
- [ ] 8.3 Verify timer name appears correctly in the card header as read-only text
- [ ] 8.4 Verify each of the 6 color schemes renders correctly and is visually distinct
- [ ] 8.5 Verify speech alarm says "Sweetie, the timer <name> is done. Do whatever you need to do and not burn the thing!" when a timer reaches zero
- [ ] 8.6 Verify `speechSynthesis.cancel()` is called on reset from alarm state
- [ ] 8.7 Verify two timers run simultaneously without affecting each other
- [ ] 8.8 Verify alarm on one timer does not affect other timers
- [ ] 8.9 Verify add/remove buttons respect the 1-minimum and 6-maximum constraints
- [ ] 8.10 Verify name prompt defaults to "Timer N" when confirmed empty
- [ ] 8.11 Verify theme toggle still applies to the whole page correctly
