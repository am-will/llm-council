# Plan

## Overview
Add a self-contained theme switcher component to the llm-council skill UI that allows users to toggle between themes (e.g., light, dark, system) via a popup interface. The theme state will be scoped to this skill UI only using CSS custom properties and local storage for persistence.

## Scope
- In:
  - Theme switcher popup component with theme options (light, dark, system)
  - CSS custom properties for theme variables scoped to the skill UI root
  - Local storage persistence keyed specifically to llm-council skill
  - Visual indicator of current theme in the UI
  - Smooth transition between themes
- Out:
  - Global/system-wide theme changes affecting other skills or applications
  - Theme customization beyond predefined presets
  - Server-side theme storage or user account sync

## Phases

### Phase 1: Theme Infrastructure
**Goal**: Establish CSS custom properties and theme definitions scoped to the skill UI.

#### Task 1.1: Define theme CSS variables
- Location: `ui/src/styles/themes.css` (new file), `ui/src/index.css` or equivalent root styles
- Description: Create CSS custom property definitions for light, dark, and system themes covering colors, backgrounds, borders, and text. Scope variables under a `.llm-council-root` or similar container class.
- Estimated Tokens: 2000
- Dependencies: None
- Steps:
  - Create `themes.css` with `:root` and `[data-theme="light"]`, `[data-theme="dark"]` selectors
  - Define variables: `--lc-bg-primary`, `--lc-bg-secondary`, `--lc-text-primary`, `--lc-text-secondary`, `--lc-border`, `--lc-accent`, etc.
  - Add `prefers-color-scheme` media query for system theme fallback
  - Import `themes.css` in the skill UI's root style entry point
- Acceptance Criteria:
  - Changing `data-theme` attribute on skill UI container switches visual theme
  - Variables are namespaced to avoid conflicts with other skills

#### Task 1.2: Create theme context/state management
- Location: `ui/src/context/ThemeContext.tsx` (new file) or `ui/src/hooks/useTheme.ts`
- Description: Implement React context or hook to manage theme state, read/write to local storage with a skill-specific key, and apply `data-theme` attribute to the skill UI container.
- Estimated Tokens: 2500
- Dependencies: Task 1.1
- Steps:
  - Create `ThemeContext` with `theme` state and `setTheme` function
  - Initialize from `localStorage.getItem('llm-council-theme')` or default to 'system'
  - On theme change, update `localStorage` and set `data-theme` attribute on container element
  - Handle 'system' theme by detecting `prefers-color-scheme` and adding media query listener
  - Export `ThemeProvider` and `useTheme` hook
- Acceptance Criteria:
  - Theme persists across page reloads
  - 'system' theme responds to OS preference changes in real-time
  - State is isolated to llm-council skill UI container only

### Phase 2: Theme Switcher Component
**Goal**: Build the popup UI component for theme selection.

#### Task 2.1: Create ThemeSwitcher popup component
- Location: `ui/src/components/ThemeSwitcher.tsx` (new file), `ui/src/components/ThemeSwitcher.css`
- Description: Build a button that opens a popup/dropdown with theme options. Include icons or labels for each theme. Handle click-outside to close.
- Estimated Tokens: 3000
- Dependencies: Task 1.2
- Steps:
  - Create `ThemeSwitcher` component with `isOpen` state for popup visibility
  - Add trigger button (icon showing current theme, e.g., sun/moon/auto)
  - Render popup with three options: Light, Dark, System
  - Highlight currently selected theme
  - Use `useTheme` hook to get/set theme
  - Implement click-outside detection to close popup
  - Add keyboard accessibility (Escape to close, arrow keys for navigation)
  - Style with CSS using theme variables for self-theming
- Acceptance Criteria:
  - Popup opens on button click and closes on selection or click-outside
  - Selected theme is visually indicated
  - Component is keyboard accessible
  - Smooth open/close animation

#### Task 2.2: Integrate ThemeSwitcher into skill UI layout
- Location: `ui/src/App.tsx` or main layout component, `ui/src/components/Header.tsx` if exists
- Description: Add ThemeProvider wrapper and place ThemeSwitcher component in an appropriate UI location (e.g., header, toolbar, or corner).
- Estimated Tokens: 1500
- Dependencies: Task 2.1
- Steps:
  - Wrap skill UI root component with `ThemeProvider`
  - Add `data-theme` attribute binding to root container element
  - Place `ThemeSwitcher` in header/toolbar area
  - Ensure z-index and positioning don't conflict with other UI elements
- Acceptance Criteria:
  - Theme switcher is visible and accessible in the UI
  - Theme changes apply immediately to entire skill UI
  - No visual regressions in existing components

### Phase 3: Polish and Testing
**Goal**: Ensure robustness, accessibility, and smooth UX.

#### Task 3.1: Update existing components to use theme variables
- Location: All component CSS files in `ui/src/components/`
- Description: Replace hardcoded colors with CSS custom properties to ensure all UI elements respond to theme changes.
- Estimated Tokens: 3000
- Dependencies: Task 2.2
- Steps:
  - Audit existing component styles for hardcoded colors
  - Replace with corresponding `--lc-*` variables
  - Test each component in both light and dark themes
  - Fix any contrast or readability issues
- Acceptance Criteria:
  - All components properly themed in light and dark modes
  - No hardcoded colors remain in component styles
  - WCAG AA contrast ratios maintained in all themes

#### Task 3.2: Add transition animations
- Location: `ui/src/styles/themes.css`
- Description: Add smooth CSS transitions when switching themes to avoid jarring visual changes.
- Estimated Tokens: 500
- Dependencies: Task 3.1
- Steps:
  - Add `transition: background-color 0.2s, color 0.2s, border-color 0.2s` to themed elements
  - Use `@media (prefers-reduced-motion: reduce)` to disable for accessibility
- Acceptance Criteria:
  - Theme transitions are smooth (0.2-0.3s)
  - Reduced motion preference is respected

## Testing Strategy
- Unit tests for `useTheme` hook: verify state changes, localStorage reads/writes, system preference detection
- Component tests for `ThemeSwitcher`: verify popup opens/closes, theme selection triggers callback, keyboard navigation works
- Integration test: verify theme persists across simulated page reload
- Visual regression test: screenshot comparison of UI in light vs dark themes
- Manual testing: verify theme isolation (other skills unaffected), test system theme with OS preference toggle
- Accessibility audit: verify focus management, ARIA attributes on popup, color contrast in all themes

## Risks
- **Risk**: CSS variable scoping leaks to other skills if not properly namespaced.
  - Mitigation: Use unique prefix (`--lc-`) and scope all theme styles under a container class/attribute selector.
- **Risk**: Local storage key collision with other skills or future features.
  - Mitigation: Use fully qualified key `llm-council-ui-theme` with skill identifier.
- **Risk**: System theme detection fails silently in unsupported browsers.
  - Mitigation: Graceful fallback to light theme; feature-detect `matchMedia` support.
- **Self-critique**: The plan assumes React is the UI framework but doesn't verify this from the codebase—if the skill uses a different framework (Vue, Svelte, vanilla JS), the implementation approach for context/hooks would need significant changes.
- **Self-critique**: The plan doesn't account for SSR/hydration scenarios—if the UI is server-rendered, the initial theme flash (FOUC) could be jarring. Would need a blocking script or cookie-based approach to prevent flash.

## Rollback Plan
- Revert commits adding theme infrastructure (`themes.css`, `ThemeContext.tsx`, `ThemeSwitcher.tsx`)
- Remove `ThemeProvider` wrapper and `ThemeSwitcher` from layout
- Remove CSS variable references from component styles (restore original colors)
- Clear `llm-council-ui-theme` from localStorage in a cleanup migration if needed
- All changes are additive and isolated, so revert is straightforward via git

## Edge Cases
- User has localStorage disabled: fall back to in-memory state, theme won't persist
- User switches OS theme while 'system' is selected: must update in real-time via `matchMedia` listener
- Popup opens near viewport edge: ensure positioning logic prevents overflow
- Multiple tabs open: consider `storage` event listener to sync theme across tabs (optional enhancement)
- Theme CSS fails to load: ensure base styles provide readable fallback
- User rapidly toggles themes: debounce or ensure transitions don't stack/conflict

## Open Questions
- None—the brief provides sufficient direction for a self-contained implementation.