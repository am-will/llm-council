# Judge Report

## Scores
- Plan 1: 7.5/10
- Plan 2: 6.5/10
- Plan 3: 7/10 (if present)

## Comparative Analysis
Plan 1 is concise and feasible for a likely static UI (index.html + app.js) and keeps scope tight. Plan 3 is similar but adds a high-contrast theme and more edge handling; slightly more complete but also adds extra themes without requirement. Plan 2 is thorough but assumes a React app and introduces extra files and complexity; feasibility is lower without confirming the UI stack.

## Missing Steps
- Verify actual UI stack and file locations before choosing React vs vanilla implementation.
- Identify the true UI root element to scope theme variables so the change only affects this skill UI.
- Audit existing color usage to ensure themes cover all hardcoded colors.
- Define accessibility attributes/roles for popup/menu and focus management.

## Contradictions
- Framework assumptions conflict: Plan 2 assumes React; Plans 1/3 assume vanilla HTML/JS/CSS.
- Theme set differences: Plan 2 uses light/dark/system; Plan 3 adds high-contrast; Plan 1 leaves number unspecified.

## Improvements
- Start with a quick codebase scan to confirm entry points and CSS strategy, then pick the simplest implementation.
- Use a unique data attribute or class on the skill UI root to scope theme variables.
- Add minimal ARIA and keyboard support to the popup (Escape to close, focus return).
- Keep theme list to required presets unless explicitly requested.

## Final Plan

# Plan

## Overview
Add a popup theme switcher scoped to the llm-council skill UI by applying theme-specific CSS variables to the skill’s root container and persisting the selection in localStorage. Implement in the existing UI stack (likely `scripts/ui/*`), keeping changes self-contained.

## Scope
- In:
  - Popup theme switcher UI (button + menu/dialog)
  - Theme presets (light, dark, system) via CSS variables
  - LocalStorage persistence scoped to llm-council UI
  - Keyboard/ARIA basics for the popup
- Out:
  - Global theme changes outside this skill UI
  - Server-side persistence
  - New build tooling

## Phases
### Phase 1: Confirm UI entry points and scoping
**Goal**: Locate the correct UI files and root container to scope theme changes.

#### Task 1.1: Verify UI files and root element
- Location: `scripts/ui/index.html`, `scripts/ui/app.js`, `scripts/ui/styles.css` (confirm)
- Description: Identify the root element to scope theming (e.g., a wrapper div) and the existing CSS variable system.
- Estimated Tokens: 200
- Dependencies: None
- Steps:
  - Inspect the UI HTML to find the top-level container.
  - Scan CSS for existing color variables and hardcoded colors.
- Acceptance Criteria:
  - Root container is identified for scoping (e.g., `.llm-council-root` or a unique data attribute).
  - A list of theme-relevant variables is captured.

### Phase 2: Theme variables and base styles
**Goal**: Define light/dark/system themes using scoped CSS variables.

#### Task 2.1: Add theme variable blocks
- Location: `scripts/ui/styles.css` (or existing style block)
- Description: Define scoped CSS variables for light/dark/system under the root container using a `data-theme` attribute.
- Estimated Tokens: 400
- Dependencies: Task 1.1
- Steps:
  - Add `[data-theme="light"]`, `[data-theme="dark"]` under the root selector.
  - For system, rely on `prefers-color-scheme` when `data-theme="system"` or no explicit theme.
  - Replace any hardcoded colors with variables if needed for coverage.
- Acceptance Criteria:
  - Toggling `data-theme` on the root visually changes the UI.
  - Theme variables do not affect other skills.

### Phase 3: Popup switcher UI
**Goal**: Provide a popup for users to pick a theme.

#### Task 3.1: Add button and popup markup
- Location: `scripts/ui/index.html`
- Description: Insert a Theme button in the header and a hidden menu/popup with options.
- Estimated Tokens: 300
- Dependencies: Task 1.1
- Steps:
  - Add a button in the existing header controls.
  - Add a popup container with three options: Light, Dark, System.
- Acceptance Criteria:
  - Button and popup exist and are correctly positioned.

#### Task 3.2: Wire popup behavior and persistence
- Location: `scripts/ui/app.js`
- Description: Toggle popup visibility, apply selected theme to root, and persist via localStorage.
- Estimated Tokens: 500
- Dependencies: Task 3.1, Task 2.1
- Steps:
  - On load, read `localStorage` key (e.g., `llm-council-ui-theme`) and apply to root.
  - Toggle popup open/close with click and Escape.
  - Apply selected theme and close popup; handle click outside.
  - If theme is `system`, react to `matchMedia` changes.
- Acceptance Criteria:
  - Theme applies immediately and persists on reload.
  - Popup opens/closes reliably with mouse and keyboard.

## Testing Strategy
- Manual: Switch themes and confirm immediate visual changes.
- Manual: Refresh page and confirm persistence.
- Manual: Verify popup closes on Escape/click outside and focus returns to the button.
- Manual: Confirm other skills’ UIs are unaffected.

## Risks
- Risk: Some colors are hardcoded and won’t update in themes.
  - Mitigation: Audit CSS and migrate hardcoded colors to variables used by themes.

## Rollback Plan
- Remove popup markup from `scripts/ui/index.html`.
- Remove theme logic from `scripts/ui/app.js`.
- Remove theme variable blocks from `scripts/ui/styles.css`.

## Edge Cases
- localStorage blocked: fall back to default theme without crashing.
- System theme changes while “System” is selected: update via `matchMedia` listener.
- Popup clipping: ensure it is positioned outside any overflow-hidden container.

## Open Questions
- None.