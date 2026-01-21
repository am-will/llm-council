# Plan

## Overview
Add a modal theme switcher to the LLM Council UI that lets users pick among defined themes by swapping CSS variables scoped to this UI, with persistence via localStorage.

## Scope
- In: Modal theme switcher UI, theme definitions via CSS variables, apply/persist theme selection in client JS.
- Out: Server-side changes, shared/global theming across other skills, new build tooling.

## Phases
### Phase 1: Theme Switcher UI & Styling
**Goal**: Add a pop-up theme picker and theme variable sets scoped to this UI.

#### Task 1.1: Add modal markup and trigger button
- Location: scripts/ui/index.html
- Description: Add a “Theme” button in the header controls and a hidden modal container with theme options.
- Estimated Tokens: 400
- Dependencies: None
- Steps:
  - Insert a theme trigger button in the header/status controls area.
  - Add modal HTML structure near the end of body (overlay + panel + theme option buttons + close).
- Acceptance Criteria:
  - Theme modal opens via button and closes via overlay/close action.

#### Task 1.2: Define theme variable sets and modal styling
- Location: scripts/ui/index.html (inline `<style>`)
- Description: Add CSS for modal and per-theme CSS variables on `:root[data-theme="..."]`.
- Estimated Tokens: 600
- Dependencies: Task 1.1
- Steps:
  - Add modal layout styling (overlay, panel, focusable buttons).
  - Define 3+ themes by overriding existing CSS variables (e.g., `--bg`, `--ink`, `--accent`).
- Acceptance Criteria:
  - Changing `data-theme` on the root updates UI colors consistently.

### Phase 2: Theme Selection Logic & Persistence
**Goal**: Wire UI actions to apply and persist theme selection.

#### Task 2.1: Implement theme switcher behavior
- Location: scripts/ui/app.js
- Description: Add handlers for opening/closing the modal, applying a theme, and storing it locally.
- Estimated Tokens: 500
- Dependencies: Task 1.1, Task 1.2
- Steps:
  - Read saved theme from localStorage on load and set `document.documentElement.dataset.theme`.
  - Add click handlers to theme buttons to apply theme + persist.
  - Add close handlers (close button, overlay click, Escape key).
- Acceptance Criteria:
  - Selected theme persists on reload; modal behaves predictably.

## Testing Strategy
- Manual: Run UI server, open UI, switch themes, refresh page, confirm persistence.
- Manual: Verify theme switcher affects only UI visuals and does not change API behavior/state.
- Manual: Check modal open/close with button, overlay click, and Escape.

## Risks
- Risk: Conflicting or unused CSS sources (inline vs `styles.css`) could cause inconsistent theming. Mitigation: keep all theme variables in the active inline CSS and avoid introducing new CSS files.
- Risk: Theme variable coverage may miss some colors, leaving mixed styling. Mitigation: audit all CSS vars and ensure each theme overrides the full set used by the UI.
- Self-critique: The plan assumes the inline `<style>` is the authoritative stylesheet; if the UI later switches to `styles.css`, updates may not apply.
- Self-critique: The plan does not include a full accessibility audit (contrast ratios, focus trapping), which could leave the modal partially inaccessible.

## Rollback Plan
- Remove modal HTML and theme button from scripts/ui/index.html.
- Remove theme-related CSS and JS logic from scripts/ui/index.html and scripts/ui/app.js.

## Edge Cases
- localStorage unavailable or blocked: fall back to default theme without errors.
- SSE updates while modal open: UI should remain responsive and not auto-close modal.
- Theme switch during active edits: ensure editor state remains unchanged.

## Open Questions
- None.