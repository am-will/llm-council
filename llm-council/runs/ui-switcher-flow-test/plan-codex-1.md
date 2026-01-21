# Plan

## Overview
Add a compact UI switcher to the LLM Council UI that lets users toggle between major output panels (Planner, Judge, Final Plan) without leaving the page. The plan updates the UI markup/CSS and wires stateful switching in `app.js`, using existing server state and event streams.

## Scope
- In:
  - Add switcher UI controls in `scripts/ui/index.html` aligned with the current layout and typography.
  - Add CSS for switcher styling and responsive behavior in `scripts/ui/index.html`.
  - Add JS logic for switching panels, preserving selection across updates in `scripts/ui/app.js`.
  - Define integration points with current state/events (planner/judge/final plan updates) in `scripts/ui/app.js`.
- Out:
  - Server-side behavior changes to planner/judge computation.
  - Persisting switcher selection to disk or run state (beyond lightweight local storage).
  - Major layout redesigns outside of switcher and minimal panel visibility tweaks.

## Phases
### Phase 1: UI Switcher Design
**Goal**: Add a visible, accessible switcher that matches existing visual language and works on mobile/desktop.

#### Task 1.1: Add switcher markup
- Location: `scripts/ui/index.html`
- Description: Insert a switcher (segmented control/tabs) near panel headers or above the grid to select active view.
- Estimated Tokens: 450
- Perceived Complexity: 3
- Dependencies: None
- Steps:
  - Add a switcher container with buttons (Planner / Judge / Final Plan) and `data-view` attributes.
  - Add `aria` attributes and `role="tablist"`/`role="tab"` semantics for accessibility.
- Acceptance Criteria:
  - Switcher renders consistently with existing UI style.
  - Buttons are focusable and labeled correctly for screen readers.

#### Task 1.2: Add switcher styling
- Location: `scripts/ui/index.html`
- Description: Add CSS for the switcher, including active/inactive states and responsive layout.
- Estimated Tokens: 500
- Perceived Complexity: 4
- Dependencies: Task 1.1
- Steps:
  - Create CSS variables and styles for the switcher container and buttons matching the existing bold visual style.
  - Add responsive rules to keep the switcher legible on small screens and align with the grid.
- Acceptance Criteria:
  - Switcher is visually distinct and fits the current design language.
  - Switcher wraps/stack gracefully at <768px without overflow.

### Phase 2: Switcher Integration
**Goal**: Wire switcher logic to show/hide panels and keep selection stable during live updates.

#### Task 2.1: Toggle panel visibility
- Location: `scripts/ui/app.js`, `scripts/ui/index.html`
- Description: Add panel identifiers and JS handlers to show only the active panel (or highlight active panel) when the switcher changes.
- Estimated Tokens: 650
- Perceived Complexity: 5
- Dependencies: Task 1.1
- Steps:
  - Add `data-panel="planner|judge|final"` attributes to the panel elements in `scripts/ui/index.html`.
  - In `scripts/ui/app.js`, add a `setActiveView(view)` function to toggle panel visibility (`hidden` class or `aria-hidden`).
- Acceptance Criteria:
  - Clicking switcher buttons consistently displays the selected panel.
  - Non-active panels are hidden or de-emphasized without breaking layout.

#### Task 2.2: Preserve selection and integrate with events
- Location: `scripts/ui/app.js`
- Description: Persist the active view in memory (and optionally localStorage) so it survives SSE updates and page reload.
- Estimated Tokens: 550
- Perceived Complexity: 5
- Dependencies: Task 2.1
- Steps:
  - Add `activeView` state, defaulting to `final` (or last used from localStorage).
  - On `applyState` and event handlers, avoid resetting view; only update content.
  - Store selection to localStorage on change and read on load.
- Acceptance Criteria:
  - Active panel stays selected as new data arrives.
  - Reloading the page restores the last active panel.

#### Task 2.3: Integration points documentation
- Location: `scripts/llm_council.py`, `scripts/ui_server.py` (reference-only), `SKILL.md` (optional)
- Description: Document where switcher interacts with UI state and events so future changes can hook in cleanly.
- Estimated Tokens: 300
- Perceived Complexity: 2
- Dependencies: Task 2.2
- Steps:
  - Add concise inline comments in `scripts/ui/app.js` referencing event types and UI state fields used by the switcher.
  - Optionally add a brief note in `SKILL.md` about the switcher’s behavior and persistence.
- Acceptance Criteria:
  - Clear pointers for future integration without changing server logic.

## Testing Strategy
- Manual UI smoke test: run `python3 scripts/llm_council.py run --spec <spec>` and verify switcher toggles panels during live updates.
- Resume UI test: run `python3 scripts/llm_council.py ui --run-dir <run>` and verify switcher persists selection across reload.
- Accessibility check: keyboard navigation (Tab/Enter/Space) on switcher buttons, `aria-selected` updates correctly.

## Risks
- UI layout regression on small screens due to panel hiding or grid changes; mitigate with responsive CSS and `hidden` class that preserves layout flow.
- SSE updates could overwrite user state if `applyState` re-renders the panel structure; mitigate by isolating view toggling from data updates.
- Self-critique: The plan assumes a simple show/hide switcher fits the UX, but users might expect multi-panel visibility; this could reduce visibility of live statuses.
- Self-critique: LocalStorage persistence could conflict with multi-run sessions opened in different tabs; the plan doesn’t namespace by run ID.

## Rollback Plan
- Revert changes in `scripts/ui/index.html` and `scripts/ui/app.js` to remove switcher markup, CSS, and JS state.
- Verify that the UI still renders all panels simultaneously and SSE updates function normally.

## Edge Cases
- No planners returned yet: switcher should still allow switching without errors.
- Empty final plan or judge output: panel content should display placeholders without layout glitches.
- SSE disconnect/reconnect: selected view should remain stable.
- Mobile viewport: switcher should wrap without hiding buttons or overlapping the hero header.

## Open Questions
- None.