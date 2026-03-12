## Context

The app is a single `index.html` file with all CSS and JS inline. Currently, all timer state (`remainingSeconds`, `intervalId`, `appState`) is stored as global variables and all DOM references are singleton IDs (`#time-display`, `#btn-start`, etc.). There are no modules, no build tools, and no external dependencies.

## Goals / Non-Goals

**Goals:**
- Support 1–6 simultaneous, fully independent timers on the page
- Each timer has its own state, display, controls, presets, and alarm
- Each timer is assigned a distinct color scheme and decorative pattern
- Users can name a timer when adding it (up to 15 characters); the name displays read-only in the card header
- Users can add a new timer and remove any existing timer
- Alarm speaks the timer's name aloud using the Web Speech API instead of beeping
- Theme toggle remains in the page header, not per-timer

**Non-Goals:**
- Persisting timers across page reloads
- Drag-and-drop reordering
- Timer grouping or linking
- Editing a timer's name after it has been added

## Decisions

### 1. Timer state as a class / factory function

**Decision**: Encapsulate each timer's state and DOM in a `createTimer(id, name, scheme)` factory that returns an object with its own `remainingSeconds`, `intervalId`, `appState`, DOM root element, and methods (`start`, `pause`, `reset`, `tick`).

**Rationale**: The existing global-state approach cannot be duplicated without conflicts. A factory function keeps the code self-contained without introducing a build step or ES modules.

**Alternative considered**: ES module with a `Timer` class — rejected because it requires either a `<script type="module">` tag (CORS restrictions for local files) or a dev server.

### 2. Dynamic DOM generation

**Decision**: Each timer's HTML is generated programmatically in JS (`createTimerDOM(id, name, scheme)`) rather than duplicating markup. The timer container (`#timers-container`) holds all timer cards in a flex-wrap layout.

**Rationale**: Avoids maintaining N copies of the same HTML. Makes add/remove trivial.

### 3. Layout: horizontal flex-wrap

**Decision**: Timer cards wrap horizontally using `display: flex; flex-wrap: wrap; gap: 16px`. Cards keep the existing 290px width.

**Rationale**: Minimal CSS change. Wraps naturally on smaller screens.

### 4. Max timers cap at 6

**Decision**: Disable the "Add Timer" button when 6 timers exist.

**Rationale**: Prevents layout chaos and keeps the predefined color scheme array manageable (one scheme per slot).

### 5. Timer naming — prompt then read-only

**Decision**: When the user clicks "Add Timer", show a small inline prompt (a styled `<input>` + confirm button rendered into the page, not a browser `prompt()` dialog) to enter a name. The name is capped at 15 characters client-side. Once confirmed, the name renders as static text in the card header and cannot be changed.

**Rationale**: Browser `prompt()` is visually inconsistent and blocks the thread. An inline prompt fits the card-based aesthetic. Read-only after creation keeps state simple (no edit mode).

**Alternative considered**: Editable inline header text — rejected to keep timer state model simple and avoid accidental edits.

### 6. Per-timer color schemes

**Decision**: Define an array of 6 predefined color scheme objects (`TIMER_SCHEMES`), each specifying a gradient pair (accent-a / accent-b) and a unique CSS background pattern for the card header. Schemes are assigned by insertion order (timer 1 gets scheme 0, timer 2 gets scheme 1, etc., wrapping if timers are removed and re-added).

Proposed scheme palette (header gradient pairs):
- 0: Terracotta → Amber (existing default)
- 1: Teal → Emerald
- 2: Indigo → Violet
- 3: Rose → Pink
- 4: Slate → Steel
- 5: Olive → Moss

Each scheme also applies a distinct SVG background pattern to the card header (diagonal stripes, dots, chevrons, waves, crosshatch, hexagons).

**Rationale**: Predefined schemes are visually cohesive, require no color picker UI, and are trivially applied via inline style overrides on the card element.

### 7. Speech alarm via Web Speech API

**Decision**: Replace the `AudioContext` oscillator alarm with `window.speechSynthesis`. The utterance text is: `"Sweetie, the timer [name] is done. Do whatever you need to do and not burn the thing!"`. The implementation SHALL attempt to select a Scottish female voice from `speechSynthesis.getVoices()` — specifically look for a voice whose `lang` starts with `"en-GB"` and whose `name` contains keywords like `"Female"`, `"Scottish"`, or `"Fiona"`/`"Moira"` as a heuristic. If no Scottish female voice is found, use the first available `en-GB` voice, then fall back to the browser default. If `speechSynthesis` is not available at all, fall back to a single short oscillator beep.

**Rationale**: Speaking the timer's name is far more useful in a kitchen context — the user knows which dish is ready without looking at the screen. The specific phrase and voice persona add character. The Web Speech API is supported in all major modern browsers (Chrome, Firefox, Safari, Edge). Voice selection is best-effort since available voices vary by OS and browser.

**Alternative considered**: Keep oscillator beeps with a visual label — rejected per the proposal requirement.

## Risks / Trade-offs

- [Speech synthesis voice loading] `speechSynthesis.getVoices()` may return an empty array synchronously until the `voiceschanged` event fires → Mitigation: attempt voice selection after `voiceschanged`; if voices are already loaded (sync), use them immediately; fall back to browser default if no Scottish voice is found
- [No Scottish voice available] The target voice persona depends on OS-installed voices and is not guaranteed → Mitigation: voice selection is best-effort; any voice delivering the message is acceptable
- [Speech synthesis mobile] iOS Safari requires a user gesture before speech can play → Mitigation: alarm is always triggered by the countdown reaching zero after the user pressed Start, which counts as a user gesture chain; acceptable
- [Color scheme wrap-around] If a user adds/removes timers repeatedly, scheme assignment by slot index may repeat → Mitigation: assign scheme by `timerId % 6`; visible repetition only occurs with 7+ timers, which is blocked by the 6-timer cap
- [Layout overflow on narrow screens] 6 × 290px cards need ~1800px at min → Mitigation: flex-wrap handles this; on mobile only 1-2 cards visible at a time

## Migration Plan

1. Refactor `index.html`: extract timer logic into `createTimer(id, name, scheme)` factory
2. Define `TIMER_SCHEMES` array with 6 color/pattern entries
3. Replace single `.card` markup with a dynamic container
4. Add page-level header with "Add Timer" button and inline name prompt flow
5. Add "Remove" button per card
6. Replace oscillator alarm with `speechSynthesis` (with oscillator fallback)
7. Initialize with one timer on load using a default name (e.g. "Timer 1")
8. Manual smoke-test: naming, color schemes, speech alarm, multi-timer independence, theme toggle
