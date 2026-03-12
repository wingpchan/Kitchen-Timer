# Claude Code Instructions — KitchenTimer2

## Project overview

A single-file browser kitchen timer app. Everything — HTML, CSS, and JavaScript — lives in `index.html`. There are no dependencies, no build step, no package manager. To run: open `index.html` in a browser.

## Architecture

`index.html` is divided into three logical sections in order:

### 1. CSS (`<style>`)
- **Color tokens** — CSS custom properties for light and dark themes (`[data-theme="dark"]`)
- **Page layout** — `body` (flex column, `100dvh`, `overflow: hidden`), `#page-header`, `#timers-container` (flex wrap, `overflow-y: scroll`, scrollbar hidden)
- **Card styles** — `.card`, `.card-header`, `.card-body`, `.card-footer`, `.dial`, `.time-display`, `.presets`, `.preset-btn`, `.divider`, `.time-setter`, `.controls`, `.btn`, `.status`
- **Row dividers** — `.row-divider`, `.row-divider--alarm`, `.row-divider--completed`
- **Compact media queries** — two cumulative tiers:
  - `@media (max-height: 1100px)` — moderate compact (reduces padding/spacing; all controls visible)
  - `@media (max-height: 950px)` — aggressive compact (also hides `.divider`, `.status`, `.card-footer`)

### 2. HTML (`<body>`)
- `#page-header` — page title, `#name-prompt` (hidden until Add Timer is clicked), `#add-timer-btn`, `#theme-toggle`
- `#timers-container` — empty on load; timer cards and row-divider sentinels are injected by JS

### 3. JavaScript (`<script>`)

**Theme** — reads `localStorage.themePreference`, falls back to `prefers-color-scheme`. A separate inline `<script>` in `<head>` applies the theme before first paint to prevent flash.

**Color schemes** — `TIMER_SCHEMES` array of six CSS class names (`scheme-0` … `scheme-5`), applied round-robin on card creation.

**Speech alarm** — `speakAlarm(name)` / `cancelAlarm()` via Web Speech API; `playFallbackBeep()` as AudioContext fallback.

**Timer factory** — `createTimer(name, schemeClass, onTick)` returns a card element and `getState()`. Internal state: `remainingSeconds`, `intervalId`, `appState`.

Timer state machine:
```
idle → running → paused → running
running → alarm (at 0)
alarm → completed (Reset)
completed → idle (Reset)
running | paused | idle → idle (Reset)
```

**Timer Manager** — manages the `timers` Map (id → timer object) and the two persistent row-divider sentinel elements (`alarmDivider`, `completedDivider`).

Key functions:
- `sortTimers()` — re-buckets all timers into `active`, `idle`, `alarm`, `completed`; removes and re-inserts dividers; restores focus after DOM mutations; skips reorder when `clickInProgress` is true
- `clickInProgress` flag — set on `document` `pointerdown`, cleared on `pointerup`; prevents `sortTimers()` from moving cards during an active click (race condition guard)
- Add/remove timer logic — enforces the 6-timer maximum; disables the remove button on the last card

## Key constraints

- **Single file** — do not split into multiple files or introduce a build step
- **No dependencies** — no npm, no frameworks, no external scripts
- **Card max-width: 280px** — this keeps `10cqw` below the `1.8rem` clamp floor so all cards render at the same font size and height regardless of how many cards share a row. Do not increase this without re-auditing card height consistency
- **Tier 2 threshold: 950px** — three rows at Tier 1 need ~895px of viewport height; Tier 2 must activate before that. Do not lower this threshold without re-auditing the three-row height budget
- **`clickInProgress` guard** — always preserve this; removing it reintroduces the click race condition where `setInterval` moves cards mid-click

## OpenSpec workflow

Changes to this project use the OpenSpec spec-driven workflow. The required sequence is enforced in `openspec/config.yaml`:

1. **Proposal** — created with `/opsx:propose`
2. **Design**
3. **Specs** (one file per capability under `openspec/changes/<name>/specs/`)
4. **Tasks**
5. **Apply** — implemented with `/opsx:apply`; archive with `/opsx:archive`

**Never suggest or begin implementation until all prior artifacts are complete and the user has explicitly confirmed they are ready to proceed.**

Current specs live in `openspec/specs/`. Archived changes are in `openspec/changes/archive/`.

## Capabilities (current specs)

| Capability | Spec |
|---|---|
| Timer instance (state machine, controls, presets, speech) | `openspec/specs/timer-instance/spec.md` |
| Multi-timer manager (add/remove, sort, row layout) | `openspec/specs/multi-timer-manager/spec.md` |
| Three-row layout (Alarm, Completed rows, dividers, viewport fit) | `openspec/specs/finished-timer-row/spec.md` |
| Light/dark theme switching | `openspec/specs/theme-switching/spec.md` |
| Favicon | `openspec/specs/favicon/spec.md` |
