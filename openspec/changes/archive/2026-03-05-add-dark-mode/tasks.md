## 1. Color Token Foundation

- [x] 1.1 Define CSS custom property tokens for the light theme (backgrounds, text, borders, button states, timer display) and replace all hardcoded color values in existing styles
- [x] 1.2 Define the dark theme palette as a second set of values for the same custom property tokens
- [x] 1.3 Verify WCAG AA contrast ratios (4.5:1 normal text, 3:1 large text) for all dark-palette foreground/background pairs

## 2. Theme Application

- [x] 2.1 Add logic to apply the theme by setting `data-theme="dark"` (or removing it) on `<html>` / `<body>`
- [x] 2.2 Inject an inline `<script>` in `<head>` that reads `localStorage.themePreference` (and falls back to `prefers-color-scheme`) and sets the theme attribute synchronously before first paint

## 3. Toggle Control

- [x] 3.1 Create a sun/moon icon toggle button component and add it to the app header/toolbar
- [x] 3.2 Wire the toggle to switch the active theme and update `localStorage.themePreference`

## 4. Persistence & Preference Resolution

- [x] 4.1 Implement preference resolution order on app load: (1) `localStorage.themePreference`, (2) `prefers-color-scheme`, (3) light fallback
- [x] 4.2 Ensure the toggle control reflects the currently active theme (correct icon state) on initial render

## 5. QA & Verification

- [x] 5.1 Test light ↔ dark toggle switching across all UI states (idle, counting down, alarm, paused)
- [x] 5.2 Test that a persisted `themePreference` is restored correctly on page reload, overriding OS preference
- [x] 5.3 Verify no flash of wrong theme on initial load with both light and dark preferences
