# Judge Report

## Scores
- Plan 1: 8/10
- Plan 2: 7/10
- Plan 3: 3/10

## Comparative Analysis
Plan 1 best matches the brief: it targets the current UI, focuses on a switcher for existing panels, and calls out integration with current state/events. It is actionable and relatively concise. Plan 2 is solid but more generic and drifts into creating view containers without anchoring to the current LLM Council UI panels; it also introduces a custom event system not clearly tied to existing app state. Plan 3 is largely off-scope: it introduces a run switcher and backend changes, violating the “focus on UI switcher design and integration points” constraint and expanding scope into server endpoints.

## Missing Steps
- Identify the exact existing DOM elements/panels to be switched and how they map to “Planner/Judge/Final Plan” in the current UI.
- Define the minimal CSS class or `aria-hidden` strategy consistent with existing styles.
- Ensure switcher does not break live update rendering (SSE/applyState flow).

## Contradictions
- Plan 3 conflicts with scope by adding backend endpoints and run history features.
- Plan 2’s “no backend changes” vs. adding view containers could conflict with “focus on integration points” if the UI already has panels; it risks unnecessary HTML restructuring.

## Improvements
- Anchor the switcher to current panel IDs/classes in `scripts/ui/index.html`.
- Keep switching logic independent of data updates by applying visibility after `applyState`.
- Use minimal localStorage with a namespaced key to avoid cross-run clashes.

## Final Plan

# Plan

## Overview
Add a compact, accessible UI switcher to the current LLM Council UI that toggles between existing Planner, Judge, and Final Plan panels. Integrate it into the existing DOM and `app.js` state flow so it stays stable during live updates and optional reloads.

## Scope
- In:
  - Add switcher markup and styles in `scripts/ui/index.html`.
  - Wire switcher behavior and panel visibility in `scripts/ui/app.js`.
  - Preserve active view across UI updates and reloads (localStorage).
- Out:
  - Backend changes in `scripts/ui_server.py` or `scripts/llm_council.py`.
  - New content creation beyond showing/hiding existing panels.
  - Major layout redesign.

## Phases
### Phase 1: UI Switcher Design
**Goal**: Place a switcher that matches current UI styles and is keyboard accessible.

#### Task 1.1: Add switcher markup
- Location: `scripts/ui/index.html`
- Description: Insert a segmented control (Planner / Judge / Final Plan) near existing panel header or top bar, using `role="tablist"` and `role="tab"`.
- Estimated Tokens: 350
- Perceived Complexity: 3
- Dependencies: None
- Steps:
  - Identify the current panel container area and insert a switcher wrapper.
  - Add buttons with `data-view="planner|judge|final"` and `aria-selected`.
- Acceptance Criteria:
  - Switcher renders in the current UI without breaking layout.
  - Buttons are focusable and labeled for screen readers.

#### Task 1.2: Add switcher styling
- Location: `scripts/ui/index.html`
- Description: Add CSS for the switcher (active/inactive/hover/focus states) consistent with existing typography and spacing.
- Estimated Tokens: 400
- Perceived Complexity: 4
- Dependencies: Task 1.1
- Steps:
  - Define styles for container and buttons, matching existing color/spacing variables.
  - Add responsive rules to wrap or stack at small widths.
- Acceptance Criteria:
  - Active state is clearly visible.
  - Switcher remains usable on mobile widths.

### Phase 2: Switcher Integration
**Goal**: Toggle existing panels and keep selection stable through live updates.

#### Task 2.1: Map existing panels to views
- Location: `scripts/ui/index.html`
- Description: Add `data-panel="planner|judge|final"` to the existing panel root elements.
- Estimated Tokens: 200
- Perceived Complexity: 3
- Dependencies: Task 1.1
- Steps:
  - Identify current panel containers for Planner, Judge, Final Plan.
  - Add `data-panel` attributes for targeting.
- Acceptance Criteria:
  - Each panel has a unique view identifier.

#### Task 2.2: Implement switcher logic
- Location: `scripts/ui/app.js`
- Description: Add `setActiveView(view)` to update button states and show/hide panels using a `hidden` or `aria-hidden` class.
- Estimated Tokens: 500
- Perceived Complexity: 5
- Dependencies: Task 2.1
- Steps:
  - Query switcher buttons and panel containers on load.
  - On click, update active button and toggle panel visibility.
  - Store selection in `localStorage` with a namespaced key (e.g., `llm-council-active-view`).
- Acceptance Criteria:
  - Clicking a switcher button shows the correct panel.
  - Non-active panels are hidden without layout glitches.

#### Task 2.3: Integrate with state updates
- Location: `scripts/ui/app.js`
- Description: Ensure `applyState` or SSE updates do not reset active view.
- Estimated Tokens: 300
- Perceived Complexity: 4
- Dependencies: Task 2.2
- Steps:
  - After `applyState` renders content, re-apply `setActiveView(activeView)`.
  - Initialize `activeView` from localStorage on load.
- Acceptance Criteria:
  - Active view stays selected during live updates.
  - Reload restores last selection.

## Testing Strategy
- Manual UI test: toggle between Planner/Judge/Final Plan during live updates.
- Reload persistence: select non-default view, reload, verify restoration.
- Keyboard navigation: Tab + Enter/Space and check `aria-selected` updates.

## Risks
- SSE updates re-render panels and wipe visibility state; mitigate by reapplying `setActiveView` after `applyState`.
- CSS conflicts with existing styles; mitigate with scoped class names (e.g., `.ui-switcher`).

## Rollback Plan
- Revert changes in `scripts/ui/index.html` and `scripts/ui/app.js`.
- Confirm UI returns to showing all panels simultaneously.

## Edge Cases
- Empty/partial panel content: switcher still toggles without errors.
- Very small viewport: switcher wraps without overlapping header.

## Open Questions
- None.