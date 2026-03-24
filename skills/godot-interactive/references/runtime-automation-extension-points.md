# Runtime Automation Extension Points

Use this file when a task asks for Playwright-like interaction, screenshot evidence, or repeatable gameplay checks.

## Hard Boundary

`godot-interactive` is a skill layered on top of transports.

- A skill can coordinate existing MCP tools and project workflows.
- A skill cannot by itself create new runtime transport primitives that the MCP or project does not already expose.
- `godot_editor_execute_gdscript` runs in the editor/plugin process, not as a general command channel into the running game process.

## What Exists Today

Current `godot-mcp` is strong at:

- scene inspection and live editor edits
- open/save/run/stop loops
- runtime output and error capture
- editor-side diagnostics
- runtime input automation and screenshot capture through `godot_runtime_*`

Current `godot-mcp` still has meaningful runtime gaps:

- project-specific state snapshots
- reset hooks for deterministic test setup
- drag-and-drop and richer pointer semantics
- multi-window focus and advanced OS-level interactions

## What The Skill Should Prefer

When runtime automation is needed, prefer these sources in order:

1. Dedicated `godot_runtime_*` MCP tools when they fit the task.
2. A project-provided runtime automation harness or deterministic test runner for project-specific state/setup.
3. Manual in-engine verification.

Do not present editor-only evidence as if it were runtime automation evidence.

## Recommended Project Harness Shape

If a project needs more than the stock `godot_runtime_*` tools, the best pattern is a debug-only runtime harness in the running game process. The exact transport is project-specific, but the capability surface should be close to this:

- `wait_frames(count)`
- `wait_seconds(seconds)`
- `press_action(name, strength=1.0)`
- `release_action(name)`
- `tap_action(name, frames=1)`
- `mouse_move(x, y)`
- `mouse_button(button, pressed)`
- `click_at(x, y, button=1)`
- `type_text(text)`
- `capture_screenshot(path)`
- `snapshot_state()`
- `reset_test_state()`

Expected behavior:

- UI-focused checks use real input events, not only `Input.action_press`.
- Screenshot capture writes to a deterministic path and returns the resolved path plus basic metadata.
- State snapshots return structured dictionaries with only stable, test-relevant fields.
- Timing helpers wait on frames or explicit readiness conditions instead of arbitrary long sleeps.

## Current MCP Runtime Tool Set

The current first-class runtime surface is:

- `godot_runtime_status`
- `godot_runtime_wait`
- `godot_runtime_press_action`
- `godot_runtime_release_action`
- `godot_runtime_tap_action`
- `godot_runtime_mouse_move`
- `godot_runtime_click`
- `godot_runtime_type_text`
- `godot_runtime_capture_screenshot`

The next useful additions would be:

- `godot_runtime_get_state`
- `godot_runtime_reset_test_state`
- richer pointer gestures like drag-and-drop

These target the running game process, not the editor tree.

## Evidence Rules

For automation-backed claims:

- Pair input commands with explicit postconditions.
- Pair screenshots with logs, scene state, or structured runtime state.
- Prefer deterministic test scenes and seeded content when possible.
- If the project lacks a runtime harness, say so plainly and fall back to manual verification.
