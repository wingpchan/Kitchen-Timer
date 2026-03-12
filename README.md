# Kitchen Timer

A browser-based kitchen timer supporting up to six independent, named countdown timers running simultaneously. No installation, no dependencies — open `index.html` and go.

## Usage

Open `index.html` in any modern browser. No server or build step required.

## Features

### Multiple independent timers
- Start with one timer on load; add up to six via **+ Add Timer**
- Each timer is named (up to 15 characters) at creation; names cannot be changed after
- Each card gets a distinct colour scheme and decorative header pattern
- Remove any timer at any time — the last remaining timer cannot be removed
- All timers run fully independently; one reaching zero does not affect the others

### Setting time
- Type minutes and seconds directly into the inputs
- Or use the quick-preset pills: **1m · 3m · 5m · 10m · 15m**

### Controls
| Button | Behaviour |
|--------|-----------|
| **Start** | Begin countdown from the entered time |
| **Pause** | Freeze the countdown; label changes to **Resume** |
| **Resume** | Continue from where it was paused |
| **Reset** | From running/paused → return to idle. From alarm → move to Completed row. From completed → return to idle |
| **Restart** | (appears in alarm state) Start again from the original time |

### Three-row layout
Timers are automatically sorted and grouped into three rows:

| Row | Contents |
|-----|----------|
| **Row 1 — Active** | Running, paused, and idle timers. Sorted by time remaining (most urgent first); idle timers follow alphabetically |
| **Row 2 — Alarm** | Timers whose countdown has reached zero — display flashes, speech alarm fires. Row and divider appear only when needed |
| **Row 3 — Completed** | Timers acknowledged via Reset from alarm state — static `00:00`, inputs locked. Row and divider appear only when needed |

### Speech alarm
When a timer reaches zero the app speaks the timer's name aloud using the Web Speech API. If speech synthesis is unavailable in the browser a short beep plays instead.

### Light / dark theme
- Automatically follows the OS `prefers-color-scheme` setting on first load
- Toggle manually with the moon/sun button in the page header
- Preference is saved in `localStorage` and restored on reload

### Responsive layout
Cards resize fluidly between 160px (minimum) and 280px (maximum) wide. The countdown font scales with card width. All three rows fit within the visible viewport at any common screen height (≥ 700px) with no scrollbar.

## Browser support

Requires a modern evergreen browser (Chrome, Edge, Firefox, Safari). The speech alarm uses the Web Speech API — a beep fallback is provided for browsers where this is unavailable.
