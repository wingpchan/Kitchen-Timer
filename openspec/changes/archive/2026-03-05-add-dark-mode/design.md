## Context

KitchenTimer2 is a timer application used in kitchen/cooking contexts, frequently in low-light environments. There is currently a single (light) color theme hardcoded throughout the UI. No theming infrastructure exists. This change introduces the foundational theme system: a dark and a light theme, OS-preference detection, a manual override toggle, and persistence.

## Goals / Non-Goals

**Goals:**
- Define a light and dark color palette as CSS custom properties (or equivalent token system)
- Read `prefers-color-scheme` media query on startup and apply the matching theme
- Provide a theme toggle control in the UI so the user can override the OS preference
- Persist the user's manual override in `localStorage` under the key `themePreference`
- Restore the persisted preference on subsequent app loads (takes priority over OS preference)

**Non-Goals:**
- Additional themes beyond light and dark
- Per-component theming or user-customizable colors
- Server-side rendering / SSR flash-of-wrong-theme mitigation (out of scope for now)
- Changing timer logic, alarm sounds, or any non-visual behavior

## Decisions

### 1. CSS Custom Properties for theming
**Decision**: Define the full color palette as CSS custom properties on `:root` and swap them by toggling a `data-theme="dark"` attribute on `<html>` (or `<body>`).

**Alternatives considered**:
- Separate CSS class per theme (e.g., `.dark`): functionally equivalent but attribute is more semantically aligned with ARIA and OS conventions.
- CSS-in-JS or runtime style objects: adds unnecessary complexity for a two-theme system.

**Rationale**: CSS custom properties are natively supported, require zero runtime overhead, and allow all component styles to use semantic tokens (`--color-background`, `--color-text`, etc.) without knowing which theme is active.

### 2. Theme resolution order
**Decision**: On load, resolve theme in this priority order:
1. `localStorage.getItem('themePreference')` — user's explicit override
2. `window.matchMedia('(prefers-color-scheme: dark)').matches` — OS preference
3. Light theme as fallback

**Rationale**: The user's intentional choice always wins. Falling back to OS keeps the default experience consistent with system settings.

### 3. Toggle placement
**Decision**: A single icon button (sun/moon) placed in the app header/toolbar area.

**Rationale**: Standard pattern; minimal screen real estate; icon conveys meaning without a label.

### 4. Persistence key
**Decision**: `localStorage` key `themePreference`, storing the string `"light"` or `"dark"`. When the user's manual override matches the OS preference, clear the key rather than writing a redundant value — but this is an optimization, not a requirement.

**Rationale**: `localStorage` is synchronous and available immediately on load, preventing a flash-of-wrong-theme before any async operation resolves.

## Risks / Trade-offs

- **Flash of wrong theme on cold load** → Mitigation: Apply the theme attribute synchronously in a `<script>` tag in `<head>` (before paint), reading from `localStorage` before the framework renders.
- **Custom property browser support** → No mitigation needed; all modern browsers support CSS custom properties.
- **Insufficient contrast on dark palette** → Mitigation: Validate all foreground/background pairs meet WCAG AA (4.5:1 for normal text, 3:1 for large text) before shipping.

## Migration Plan

1. Introduce CSS custom property tokens; all existing hardcoded colors replaced with tokens pointing to the same light-theme values — zero visual change.
2. Add the dark theme palette.
3. Add the toggle control and preference persistence logic.
4. QA both themes across all UI states (idle, counting down, alarm fired, paused).
5. No rollback complexity — removing the toggle and defaulting to the light theme restores prior behavior.
