## ADDED Requirements

### Requirement: SVG favicon in browser tab
The app SHALL declare an SVG favicon via a `<link rel="icon" type="image/svg+xml">` element in `<head>` so the Kitchen Timer is visually identifiable in browser tabs and bookmarks without any external file dependency.

#### Scenario: Favicon visible in tab
- **WHEN** the user opens the app in a modern browser
- **THEN** the browser tab SHALL display a clock-shaped icon matching the app's visual identity

#### Scenario: No external file required
- **WHEN** the app is opened as a standalone HTML file with no server
- **THEN** the favicon SHALL load correctly using an inline data-URI SVG value on the `href` attribute
