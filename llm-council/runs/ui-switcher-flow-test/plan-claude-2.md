# Plan

## Overview
Implement a UI switcher component in the LLM Council skill UI that allows users to toggle between different views or modes within the interface. The approach involves creating a reusable switcher component, integrating it into the existing HTML/JS architecture, and ensuring proper state management.

## Scope
- In:
  - Design and implement a UI switcher component (HTML/CSS/JS)
  - Integrate switcher into existing `index.html` and `app.js`
  - Add state persistence for switcher selection
  - Style consistency with existing UI
- Out:
  - Backend API changes to `ui_server.py` or `llm_council.py`
  - New view/page content (only the switcher mechanism itself)
  - Database or persistent storage beyond session/localStorage

## Phases

### Phase 1: Analyze Current UI Structure
**Goal**: Understand existing UI architecture to determine optimal integration points

#### Task 1.1: Review existing HTML structure
- Location: `scripts/ui/index.html`
- Description: Examine current DOM structure, identify header/navigation areas suitable for switcher placement, and note existing CSS classes/patterns
- Estimated Tokens: 500
- Perceived Complexity: 2
- Dependencies: None
- Steps:
  - Read `index.html` to understand layout structure
  - Identify container element for switcher placement (likely header or toolbar area)
  - Document existing CSS class naming conventions
- Acceptance Criteria:
  - Integration point identified with specific element selector
  - CSS naming pattern documented

#### Task 1.2: Review existing JavaScript patterns
- Location: `scripts/ui/app.js`
- Description: Analyze current JS architecture, event handling patterns, and state management approach
- Estimated Tokens: 800
- Perceived Complexity: 3
- Dependencies: Task 1.1
- Steps:
  - Read `app.js` to understand module/class structure
  - Identify how UI state is currently managed
  - Note event binding patterns used
- Acceptance Criteria:
  - State management approach documented
  - Event binding pattern identified for reuse

### Phase 2: Implement Switcher Component
**Goal**: Create a functional, styled UI switcher component

#### Task 2.1: Add switcher HTML markup
- Location: `scripts/ui/index.html`
- Description: Add semantic HTML for the switcher component with appropriate ARIA attributes for accessibility
- Estimated Tokens: 300
- Perceived Complexity: 2
- Dependencies: Task 1.1
- Steps:
  - Add container div with class `ui-switcher` in identified location
  - Add button/tab elements for each view option
  - Include `role="tablist"`, `aria-selected`, and `tabindex` attributes
  - Add `data-view` attributes to identify each option
- Acceptance Criteria:
  - Switcher markup renders in browser
  - Passes basic accessibility audit (ARIA roles present)

#### Task 2.2: Add switcher CSS styles
- Location: `scripts/ui/index.html` (inline styles or style block) or separate CSS file if exists
- Description: Style the switcher to match existing UI aesthetic with clear active/inactive states
- Estimated Tokens: 400
- Perceived Complexity: 3
- Dependencies: Task 2.1
- Steps:
  - Add base styles for `.ui-switcher` container (flexbox layout)
  - Style individual switcher buttons with hover/focus states
  - Add `.active` class styles for selected state
  - Ensure responsive behavior for smaller screens
- Acceptance Criteria:
  - Switcher visually matches existing UI style
  - Active state is clearly distinguishable
  - Hover and focus states provide visual feedback

#### Task 2.3: Implement switcher JavaScript logic
- Location: `scripts/ui/app.js`
- Description: Add event handling and state management for switcher functionality
- Estimated Tokens: 600
- Perceived Complexity: 5
- Dependencies: Task 1.2, Task 2.1
- Steps:
  - Create `UISwitcher` class or function following existing patterns
  - Bind click events to switcher buttons
  - Implement `setActiveView(viewName)` method that updates DOM classes
  - Dispatch custom event `viewChanged` for other components to react
  - Store selection in `localStorage` for persistence
  - Initialize switcher state from `localStorage` on page load
- Acceptance Criteria:
  - Clicking switcher button updates active state visually
  - `viewChanged` event fires with correct view name
  - Selection persists across page refresh

### Phase 3: Integration and View Management
**Goal**: Connect switcher to actual view/content visibility

#### Task 3.1: Add view container structure
- Location: `scripts/ui/index.html`
- Description: Wrap existing content in view containers that can be toggled
- Estimated Tokens: 400
- Perceived Complexity: 4
- Dependencies: Task 2.1
- Steps:
  - Add `data-view-content` attribute to existing main content areas
  - Ensure each view container has unique identifier matching switcher `data-view` values
  - Add `.view-hidden` CSS class for hiding inactive views
- Acceptance Criteria:
  - View containers identifiable via `data-view-content` attribute
  - Hidden class properly hides content (display: none or visibility)

#### Task 3.2: Wire switcher to view visibility
- Location: `scripts/ui/app.js`
- Description: Connect switcher state changes to showing/hiding view content
- Estimated Tokens: 400
- Perceived Complexity: 4
- Dependencies: Task 2.3, Task 3.1
- Steps:
  - Listen for `viewChanged` event
  - Query all `[data-view-content]` elements
  - Add `.view-hidden` to all, remove from matching view
  - Handle edge case where view doesn't exist (log warning, show default)
- Acceptance Criteria:
  - Switching views shows correct content
  - Invalid view name shows default view with console warning
  - No content flash during switch

## Testing Strategy
- Manual testing: Click each switcher option and verify correct view displays
- Persistence test: Select non-default view, refresh page, verify selection persists
- Accessibility test: Navigate switcher using keyboard (Tab, Enter, Arrow keys)
- Edge case test: Manually call `setActiveView('nonexistent')` in console, verify graceful handling
- Browser test: Verify in Chrome, Firefox, and Safari (if available)
- Responsive test: Resize browser to mobile width, verify switcher remains usable

## Risks
- **Risk**: Existing JS may use conflicting global variables or event names
  - Mitigation: Use namespaced events (e.g., `council:viewChanged`) and encapsulate switcher in IIFE or class
- **Risk**: CSS specificity conflicts with existing styles
  - Mitigation: Use BEM-style naming (`.ui-switcher__button--active`) and scope styles appropriately
- **Risk**: localStorage unavailable in private browsing
  - Mitigation: Wrap localStorage access in try/catch, fall back to session-only state
- **Self-critique**: This plan assumes a simple tab-based switcher but doesn't specify what views are being switched between. The actual view content and their purposes are undefined, which could lead to implementing a switcher with no meaningful content to switch.
- **Self-critique**: The plan lacks specifics about animation/transitions between views, which could result in jarring UX. Should consider adding CSS transitions for `.view-hidden` changes but this adds complexity not accounted for in time estimates.

## Rollback Plan
- Revert changes to `index.html` and `app.js` using git: `git checkout HEAD -- scripts/ui/index.html scripts/ui/app.js`
- If localStorage state causes issues, clear via browser dev tools or add `localStorage.removeItem('council-active-view')` to console
- If partial rollback needed, remove only the switcher DOM element and its event listeners without touching view containers

## Edge Cases
- User has JavaScript disabled: Switcher won't function; ensure default view is visible without JS
- User rapidly clicks switcher: Debounce or ignore clicks during transition to prevent race conditions
- View container missing for a switcher option: Show warning, remain on current view or fall back to first available
- localStorage quota exceeded: Catch exception, proceed without persistence
- Multiple switcher instances on page (future): Use instance IDs or singleton pattern to prevent conflicts

## Open Questions
- None (per instructions to not ask questions, but noted: actual view names/purposes should be clarified before implementation)