# Plan

## Overview
Implement a **Run Switcher** in the LLM Council UI to allow users to navigate between the current active session and historical runs stored in the `runs/` directory. The approach involves extending `ui_server.py` to discover and load run states and adding a dropdown control to the `index.html` header.

## Scope
- In:
  - Backend: New endpoints to list available runs and load a specific run's state.
  - Frontend: New UI component (dropdown) in the header to select runs.
  - Frontend: Logic to populate the list and handle state transitions.
- Out:
  - "Forking" or resuming historical runs (view-only for history is assumed).
  - Authentication/Authorization for viewing specific runs.

## Phases

### Phase 1: Backend Run Management
**Goal**: Enable `ui_server.py` to identify available runs and serve their state on demand.

#### Task 1.1: Run Discovery & State Loading
- Location: `scripts/ui_server.py`
- Description:
  - Modify `UIServer.__init__` to accept a `runs_dir` path (defaulting to `../runs` relative to script).
  - Add `_handle_runs_list()` method to scan `runs_dir` for subdirectories containing `ui-state.json`.
  - Add `_handle_switch_run()` method to load a target `ui-state.json` into `self.state`.
  - Update `do_GET` to route `/api/runs`.
  - Update `do_POST` to route `/api/switch`.
- Estimated Tokens: 400
- Perceived Complexity: 4
- Dependencies: None
- Steps:
  1. Import `pathlib.Path` and `os`.
  2. Implement `list_runs` logic: return sorted list of run IDs (directory names) with timestamps.
  3. Implement `switch_run` logic: validation of run ID, reading JSON, and updating `self.state`. **Crucial**: Ensure thread safety when replacing state data.
- Acceptance Criteria:
  - `GET /api/runs` returns JSON list of run IDs.
  - `POST /api/switch` with `{"run_id": "..."}` updates the server's current state content.

### Phase 2: UI Integration
**Goal**: Expose the run switching capability to the user via the web interface.

#### Task 2.1: Switcher Component
- Location: `scripts/ui/index.html`
- Description: Add a `<select>` element to the `.brand-meta` section in the header, styled consistently with the existing `.planner-select`.
- Estimated Tokens: 100
- Perceived Complexity: 2
- Dependencies: None
- Steps:
  1. Insert `<select id="runSelect" class="run-select">` inside the header.
  2. Add CSS in `<style>` block for `.run-select` (mimic `.planner-select` but smaller/cleaner for header).
- Acceptance Criteria:
  - Dropdown appears in the header next to the Run ID or Phase.

#### Task 2.2: Frontend Logic
- Location: `scripts/ui/app.js`
- Description: Fetch available runs on load and handle switching.
- Estimated Tokens: 300
- Perceived Complexity: 3
- Dependencies: Task 1.1, Task 2.1
- Steps:
  1. Define `runsEndpoint = '/api/runs'` and `switchEndpoint = '/api/switch'`.
  2. Create `fetchRuns()` function to populate `#runSelect`.
  3. Add event listener to `#runSelect`:
     - On change, `POST` to `/api/switch` with selected ID.
     - On success, `fetchState()` to refresh UI.
  4. Update `applyState` to sync the `#runSelect` value with `state.run_id`.
- Acceptance Criteria:
  - Dropdown populates with real directory names.
  - Selecting a different run updates the entire UI to reflect that run's state.

## Testing Strategy
- **Backend Unit Test**:
  - Mock `runs_dir`.
  - Create dummy `ui-state.json` in a subdir.
  - Call `GET /api/runs` and verify list.
  - Call `POST /api/switch` and verify `GET /api/state` returns new data.
- **Manual Verification**:
  - Start server.
  - Verify "Current" run is selected.
  - Switch to a historical run (if exists).
  - Verify UI updates (Task Brief, Planners, etc.).

## Risks
- **Concurrency & State Desync**: If the main application (`llm_council.py`) holds a reference to the `UIState` object and writes to it, replacing the state in `ui_server` might break the link.
  - *Mitigation*: The `switch` logic should update the *attributes* of the existing state object rather than replacing the object instance, OR the UI should clearly indicate "History Mode" (read-only).
  - *Self-critique*: Replacing the backend state "live" is dangerous if an active process is running. It effectively "hijacks" the visualization. A better approach might be a separate "view mode" in the UI that doesn't change the server's canonical state, but the brief asks for a switcher integration. I will proceed with state replacement but note this architectural tension.
- **Thread Safety**: Reading/Writing state file during a switch.
  - *Mitigation*: Use `server.clients_lock` or a new `state_lock` if modifying the shared state object.
- **Self-critique (UX)**: The plan doesn't explicitly handle the "Active" run distinct from "History". If I switch to history, how do I get back to "Live"? The `runs` list needs to include the "current" (in-memory) run, or the user is trapped in history until restart.
  - *Mitigation*: Ensure the run list includes an entry for the currently active run (or "Latest").

## Rollback Plan
- Revert changes to `ui_server.py`.
- Revert changes to `index.html` and `app.js`.

## Edge Cases
- **No Runs Directory**: `runs/` folder doesn't exist. Server should handle gracefully (return empty list).
- **Corrupt State File**: `ui-state.json` in history is invalid. Server should return error, UI should show toast.
- **Active Run Not on Disk Yet**: If the current run hasn't saved state to disk, switching away might make it impossible to return without logic to "Keep Current in Memory".

## Open Questions
- Does `llm_council.py` actively write to the state object during the "Review" phase, or is it static? (Assumed static enough for viewing).