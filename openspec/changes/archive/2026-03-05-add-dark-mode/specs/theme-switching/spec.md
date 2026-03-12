## ADDED Requirements

### Requirement: OS preference detection
The app SHALL read the `prefers-color-scheme` media query on startup and apply the matching theme (light or dark) when no user preference has been persisted.

#### Scenario: Dark OS preference with no stored override
- **WHEN** the user opens the app and `prefers-color-scheme` is `dark` and no `themePreference` key exists in `localStorage`
- **THEN** the app SHALL render in dark mode

#### Scenario: Light OS preference with no stored override
- **WHEN** the user opens the app and `prefers-color-scheme` is `light` (or not set) and no `themePreference` key exists in `localStorage`
- **THEN** the app SHALL render in light mode

### Requirement: Manual theme toggle
The app SHALL provide a visible toggle control that allows the user to switch between light and dark themes at any time.

#### Scenario: User switches to dark mode
- **WHEN** the user activates the theme toggle while in light mode
- **THEN** the app SHALL immediately switch to dark mode without a page reload

#### Scenario: User switches to light mode
- **WHEN** the user activates the theme toggle while in dark mode
- **THEN** the app SHALL immediately switch to light mode without a page reload

### Requirement: Preference persistence
The app SHALL persist the user's manually selected theme in `localStorage` under the key `themePreference`.

#### Scenario: Manual preference is restored on reload
- **WHEN** the user has previously selected a theme via the toggle and reloads the app
- **THEN** the app SHALL apply the persisted theme preference, overriding the OS preference

#### Scenario: Persisted value takes priority over OS preference
- **WHEN** the user's `localStorage` contains `themePreference: "light"` and the OS is set to dark mode
- **THEN** the app SHALL render in light mode

### Requirement: Dark color palette
The app SHALL define a complete dark color palette applied when dark mode is active, covering all visible UI surfaces.

#### Scenario: Dark theme applied to all UI areas
- **WHEN** dark mode is active
- **THEN** all backgrounds, text, borders, input fields, buttons, and the timer display SHALL use the dark palette tokens

#### Scenario: WCAG AA contrast in dark mode
- **WHEN** dark mode is active
- **THEN** all text/background foreground pairs SHALL meet a minimum contrast ratio of 4.5:1 for normal text and 3:1 for large text (WCAG AA)

### Requirement: Theme applied before first paint
The app SHALL apply the correct theme before the browser paints the first frame to prevent a flash of the wrong theme.

#### Scenario: No flash on load with dark preference
- **WHEN** the user opens the app with a dark theme preference stored or OS dark mode active
- **THEN** the initial rendered frame SHALL show the dark theme (no white flash)
