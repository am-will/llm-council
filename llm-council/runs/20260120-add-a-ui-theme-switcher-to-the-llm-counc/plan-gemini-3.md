# Plan

## Overview
Implement a client-side theme switcher mechanism in the `llm-council` skill UI. This involves defining new CSS theme classes (Dark, High Contrast) that override existing CSS variables, adding a "Theme" button to the header which triggers a popup menu for selection, and using `localStorage` to persist the user's preference across sessions.

## Scope
- In:
  - CSS definitions for 'Dark' and 'High Contrast' themes.
  - HTML markup for the theme switcher button and popup menu in `index.html`.
  - JavaScript logic to handle toggling the popup, applying selected themes, and persisting state to `localStorage`.
- Out:
  - Backend persistence (themes are a local user preference).
  - Changes to other skills or global system configurations.

## Phases
### Phase 1: Theme Styles
**Goal**: Define visual themes using CSS variables to ensure consistent styling across the application.

#### Task 1.1: CSS Theme Definitions
- Location: `scripts/ui/styles.css`
- Description: Add `.theme-dark` and `.theme-contrast` classes attached to the `:root` or `body` scope. Override specific variables (`--bg`, `--ink`, `--panel`, `--accent`, etc.) to create distinct, accessible visual styles. Ensure complex background gradients are adjusted or simplified for dark mode.
- Estimated Tokens: 400
- Dependencies: None
- Steps:
  - Analyze existing variables to identify all necessary overrides.
  - Create `.theme-dark` with inverted colors (dark bg, light text) and adjusted shadows.
  - Create `.theme-contrast` with high contrast ratios (white/black, thick borders).
- Acceptance Criteria:
  - Manually applying the class `theme-dark` to the `<html>` tag changes the UI to a dark theme.
  - Manually applying `theme-contrast` changes the UI to a high-contrast theme.

### Phase 2: UI Implementation & Logic
**Goal**: Add the interactive elements and logic to allow users to switch themes.

#### Task 2.1: Add Switcher Component
- Location: `scripts/ui/index.html`
- Description: Add a "Theme" button to the `.status-controls` section of the header. Add a hidden `dialog` or absolute-positioned `div` container for the theme selection menu (Radio buttons or list items).
- Estimated Tokens: 300
- Dependencies: None
- Steps:
  - Insert a `<button>` element inside `.status-controls`.
  - Add the menu markup (e.g., `<div id="themeMenu" hidden>...</div>`) containing options for 'System', 'Light', 'Dark', 'Contrast'.
- Acceptance Criteria:
  - The "Theme" button appears in the header.
  - The menu markup is present but hidden by default.

#### Task 2.2: Implement Switcher Logic
- Location: `scripts/ui/app.js`
- Description: Implement the logic to toggle the menu visibility, handle theme selection, apply the class to `document.documentElement`, and save/load from `localStorage`.
- Estimated Tokens: 500
- Dependencies: Task 1.1, Task 2.1
- Steps:
  - Initialize theme on load: Check `localStorage` first, then `window.matchMedia('(prefers-color-scheme: dark)')`.
  - Add event listener to the "Theme" button to toggle the menu visibility.
  - Add event listeners to menu options to apply the selected theme and close the menu.
  - Save selection to `localStorage`.
  - Handle "click outside" to close the menu.
- Acceptance Criteria:
  - Clicking "Theme" opens the menu.
  - Selecting a theme updates the UI immediately.
  - Refreshing the page retains the selected theme.

## Testing Strategy
- **Visual Validation**: Manually cycle through all themes to ensure all text is readable and no UI elements are broken (especially shadows and borders in Dark mode).
- **Persistence Test**: Select a non-default theme, reload the page, and verify the theme persists.
- **Interaction Test**: Verify the menu closes when clicking outside of it or selecting an option.

## Risks
- **Visual Regression**: Dark mode might make standard shadows invisible or look "muddy" against a dark background.
  - *Mitigation*: Redefine `--shadow` and `--panel-shadow` variables specifically for the dark theme block (e.g., using lighter or colored shadows, or relying more on borders).
- **Self-critique 1**: The plan assumes existing CSS variables cover all color usages. If hardcoded colors exist in `styles.css` (outside variables), they won't update.
  - *Mitigation*: Grep `styles.css` for hex codes and `rgb/a` values to ensure everything is tokenized into variables before starting Phase 1.
- **Self-critique 2**: The "popup" might be clipped if the header `overflow` is hidden or if z-index stacking context is tricky.
  - *Mitigation*: Use a native `<dialog>` element or ensure the popup is appended to `body` and positioned absolutely to avoid overflow issues, or check header CSS to ensure `overflow: visible`.

## Rollback Plan
- Revert changes to `scripts/ui/index.html`, `scripts/ui/styles.css`, and `scripts/ui/app.js` to their previous git state.

## Edge Cases
- **No LocalStorage**: If `localStorage` is disabled (e.g., privacy mode), catch the exception and fall back to the session-only or default theme without crashing.
- **System Preference Change**: If the user selects "System" (or hasn't selected anything) and changes their OS theme while the app is open, listen to the `matchMedia` change event to update dynamically.

## Open Questions
- None.