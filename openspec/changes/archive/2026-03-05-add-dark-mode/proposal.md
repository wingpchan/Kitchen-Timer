## Why

Users often run the KitchenTimer app in low-light environments (evening cooking, dim kitchens) where a bright white UI causes eye strain. Dark mode provides a comfortable viewing experience and aligns with the OS-level theme preference that users already set on their devices.

## What Changes

- Add a dark color theme alongside the existing light theme
- Detect and apply the user's OS-level `prefers-color-scheme` setting on startup
- Provide a manual toggle so users can override the OS preference
- Persist the user's manual override across sessions

## Capabilities

### New Capabilities
- `theme-switching`: User-facing toggle and automatic OS-preference detection for switching between light and dark color themes, with persistence of the chosen preference.

### Modified Capabilities
<!-- No existing spec-level requirements are changing; this is purely additive. -->

## Impact

- **UI layer**: All styled components need dark-mode color variants (backgrounds, text, borders, button states, timer display)
- **State/storage**: A new preference key (e.g., `themePreference`) must be persisted (localStorage or equivalent)
- **No API or data-model changes**: Dark mode is purely a presentation concern
- **No breaking changes**
