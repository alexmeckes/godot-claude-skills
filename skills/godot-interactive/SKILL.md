---
name: godot-interactive
description: Persistent `godot-mcp` and AI Bridge workflows for Godot 4.x projects. Use this skill for `.tscn`, `.gd`, `.gdshader`, `.tres`, and live editor tasks that need scene inspection, iterative node edits, refresh/run/test loops, runtime diagnostics, or hybrid file-plus-editor validation.
---

# Godot Interactive

Use a persistent `godot-mcp` session to work on Godot 4.x projects without rediscovering the editor state every turn. This skill is the Godot analogue of `playwright-interactive`: it keeps one live editor connection alive when possible, combines it with file-based scene/script/shader tools, and requires explicit validation evidence before signoff.

Prefer the smallest evidence-producing loop. For live-editor tasks, stay on the live `godot_editor_*` and `godot_runtime_*` surface unless a file-only detail is truly required.

## Preconditions

- `godot-mcp` must be configured for the current project.
- For live editing, Godot 4.x must be running with the target project open and the AI Bridge plugin enabled.
- The bridge target should come from the active `godot_mcp` MCP configuration. `127.0.0.1:6550` is a common default, but local setups may override it.
- For automated gameplay input or screenshots, prefer the first-class `godot_runtime_*` tools when they are present. If they are absent, fall back to a project-provided runtime harness or manual verification.
- If tool choice is unclear, call `godot_help` first.
- If the bridge is unavailable, continue with file-based tools and manual Godot review instead of inventing live-editor results.
- Treat a dropped bridge connection as a session problem. Reconnect before doing more live-editor work.
- If the user explicitly targets the running editor or currently open scene, treat the connected editor project as the source of truth. Do not burn time reconciling it with the shell workspace unless the task also depends on workspace files or the user suggests the wrong editor is open.

## Workflow Decision Tree

- Use the live-editor workflow for node hierarchy changes, layout tuning, quick property edits, opening/running scenes, and immediate runtime inspection.
- Use the file-based workflow for larger structural edits, generated scenes/scripts/shaders/resources, or any work done before Godot is open.
- Use the runtime-automation workflow whenever dedicated `godot_runtime_*` tools exist. If they do not, only use it when the project already exposes a real runtime harness.
- Use the hybrid workflow for most real tasks: edit files, refresh the editor, inspect the scene tree, run the scene, collect output/errors, and iterate.

## Core Workflow

1. Build a QA inventory before changing anything.
   - Combine the user's requirements, the behaviors you implement, and the claims you expect to make in the final response.
   - For each claim, decide what evidence will prove it: scene tree state, validation output, runtime output/errors, manual in-engine verification, or a saved screenshot if the project supports it.
   - Add at least 2 off-happy-path checks for fragile gameplay, scene state, or editor synchronization.
2. Confirm the active project and current scene state.
   - Prefer `godot_connection_status`, `godot_editor_get_project_info`, and `godot_editor_get_scene_tree`.
   - If no live session exists yet, decide whether to start with file-based edits or connect immediately.
3. Inspect before editing.
   - For files: use `godot_read_scene`, `godot_list_scene_nodes`, `godot_read_script`, `godot_read_shader`, or `godot_read_resource`.
   - For live scenes: use `godot_editor_get_scene_tree` and optionally `godot_editor_select_node`.
   - For file-based scene checks, prefer `godot_list_scene_nodes` before `godot_read_scene`.
   - Do not read the full `.tscn` with `godot_read_scene` during a live task unless the live tree cannot answer the question. Large scene dumps slow the feedback loop.
   - Do not read the full `.tscn` during a file task unless you need serialized details like connections, external resources, or exact property payloads that `godot_list_scene_nodes` cannot provide.
4. Make the smallest meaningful change.
   - Prefer incremental edits over large speculative rewrites.
   - Keep node names and paths stable when possible so follow-up changes remain cheap.
   - When the goal is fast feedback rather than final UI quality, prefer a temporary probe node over exploratory edits to production nodes.
5. Refresh or save based on how the change was made.
   - External file edits while Godot is open: `godot_editor_refresh_filesystem`.
   - Live scene edits you intend to keep: `godot_editor_save_scene`.
6. Validate before running.
   - Use `godot_validate_scene` on touched scenes.
   - Use `godot_validate_script` on touched scripts.
   - Use `godot_help` or docs tools if a class/property choice is still uncertain.
7. Run and inspect.
   - Use `godot_editor_run_scene`.
   - Inspect `godot_editor_get_errors`, `godot_editor_get_output`, and, when needed, `godot_editor_get_log_file`.
   - Use `godot_runtime_*` tools for gameplay input, runtime waits, and screenshot capture when they are available.
   - Do not call `godot_runtime_status` before a scene is running unless you are checking whether a previous run is still active. Start the scene first.
   - After a live node add, prefer `godot_editor_modify_node` or structured `properties` on `godot_editor_add_node` before reaching for `godot_editor_execute_gdscript`.
   - Use `godot_editor_execute_gdscript` only for editor-side diagnostics or setup. It is not a general substitute for runtime gameplay input or frame capture.
   - If a screenshot or runtime assertion fails, inspect the specific node path and its properties first. Only expand to broad log review if the node-level check does not explain the failure.
8. Update the QA inventory if exploration reveals new states, controls, failure modes, or visible claims.
9. Sign off only from explicit evidence, and note any remaining manual verification gaps.

## Fast Probe Presets

Use this path when the user wants faster feedback, a runtime target, or proof of interaction rather than final polish.

- Prefer a temporary `Label`, `Button`, or `LineEdit` probe under a known live parent instead of spending turns rediscovering common `Control` properties.
- Use the copyable presets in [references/fast-probe-presets.md](references/fast-probe-presets.md) for deterministic geometry and obvious text.
- Default to top-left anchored controls with fixed `position`, fixed `size`, and loud text or placeholder values so runtime click and screenshot evidence are unambiguous.
- Save and validate immediately after adding the probe, then run the scene and gather evidence.
- Remove the probe, save again, and revalidate before signoff unless the user explicitly wants to keep it.
- Do not call docs tools or `godot_help` for common probe controls unless the preset fails, the project has unusual container behavior, or the user needs final-quality UI behavior rather than a temporary target.

## Start Or Reuse Live Session

Keep the same bridge connection alive across iterations when possible.

Typical attach sequence:

```text
godot_connect {}
godot_connection_status {}
godot_editor_get_project_info {}
godot_editor_get_scene_tree {}
```

If you need to override the configured bridge target for troubleshooting or multiple editors:

```text
godot_connect { "host": "127.0.0.1", "port": 6550 }
```

If you need a specific scene:

```text
godot_editor_open_scene { "scenePath": "res://scenes/main.tscn" }
godot_editor_get_scene_tree {}
```

Rules:

- Reuse the existing connection instead of reconnecting on every turn.
- If Godot was restarted, the project changed, or the bridge timed out, reconnect and reacquire the scene tree before making assumptions.
- If a node path changed, rerun `godot_editor_get_scene_tree` instead of guessing the new path.

## File-Based Workflow

Use file-based tools when the bridge is unavailable or when broad edits are easier outside the editor.

Common loop:

1. Read the current artifact.
2. Edit with the appropriate `godot_*` file tool.
3. Validate the changed file.
4. If Godot is open, call `godot_editor_refresh_filesystem`.
5. Open or rerun the scene in the editor to verify behavior.

Typical pairings:

- Scene structure: `godot_read_scene`, `godot_add_node`, `godot_modify_node`, `godot_validate_scene`
- Scripts: `godot_read_script`, `godot_write_script`, `godot_validate_script`
- Shaders: `godot_read_shader`, `godot_write_shader`
- Resources: `godot_read_resource`, `godot_write_resource`
- UI generation: `godot_ui_*` tools followed by validation and a live run pass

Use file reads only when you need file-only structure such as external resources, connections, or exact serialized properties. For most live scene checks, `godot_editor_get_scene_tree` is the faster baseline.

## Refresh And Rerun Decision

- External `.tscn`, `.gd`, `.gdshader`, `.tres`, or `project.godot` changes while the editor is open: run `godot_editor_refresh_filesystem`.
- Live scene changes you want persisted: run `godot_editor_save_scene`.
- Runtime behavior changes, script changes, or scene logic changes: rerun the scene.
- Bridge disconnect, editor restart, or project reopen: reconnect and reacquire state.
- Stale node paths: reread the scene tree before issuing more live node commands.

## Runtime Automation Workflow

Use this whenever the current MCP surface exposes `godot_runtime_*` tools. If those tools are absent, only use it when a real project-side runtime harness already exists.

- First inspect the available automation surface.
  - Check the MCP tool list for dedicated `godot_runtime_*` tools first.
  - Search the project for an existing debug/test harness, local automation server, screenshot helper, or deterministic test runner.
  - Read the harness entrypoint before using it so you know whether it targets the editor process or the running game process.
- Treat editor-side and runtime-side code as different environments.
  - `godot_editor_execute_gdscript` runs in the editor/plugin context.
  - The current `godot_runtime_*` tools target the running game process through the AI Bridge runtime harness.
  - A project-specific fallback still needs its own harness, test scene, autoload, CLI, or transport to accept input commands and emit screenshots or state snapshots.
- Prefer automation evidence in this order:
  - dedicated `godot_runtime_*` tools
  - project-provided deterministic harness that writes artifacts or structured results
  - manual in-engine verification
- For input automation:
  - Prefer real input-event injection for UI, cursor, and focus-sensitive checks.
  - Prefer action-level helpers only for gameplay semantics where position and event routing do not matter.
  - Make the target control easy to hit before running: deterministic position, visible size, and obvious text or placeholder.
  - For temporary UI targets, start with the preset `Label`, `Button`, or `LineEdit` payloads instead of discovering properties from scratch.
- For screenshot automation:
  - Prefer deterministic output paths and stable capture timing.
  - Pair screenshots with structured state, logs, or scene assertions so visual claims are not based on a single image alone.
- If no runtime harness exists, do not pretend the editor bridge can replace it. Call out the verification limit explicitly.

## Checklists

### Session Loop

- Keep one live connection alive when the editor is available.
- Reinspect current state before a new edit burst.
- Make one coherent change set.
- Refresh or save based on where the change happened.
- Validate touched files.
- Run the relevant scene.
- Inspect runtime output and errors.
- Update the QA inventory if new states or claims appear.

### Validation And Runtime Checks

- Structural changes: `godot_validate_scene`
- Script changes: `godot_validate_script`
- Runtime/editor issues: `godot_editor_get_errors`
- General trace output: `godot_editor_get_output`
- Longer or incremental log review: `godot_editor_get_log_file`
- Focused introspection: `godot_editor_execute_gdscript`
- Runtime automation: `godot_runtime_status`, `godot_runtime_wait`, `godot_runtime_press_action`, `godot_runtime_release_action`, `godot_runtime_tap_action`, `godot_runtime_mouse_move`, `godot_runtime_click`, `godot_runtime_type_text`, `godot_runtime_capture_screenshot`

When using `godot_editor_execute_gdscript`:

- Keep scripts short and diagnostic.
- Prefer returning structured values.
- Treat the result as editor-side inspection evidence, not proof that a player-facing interaction works.

### Signoff

- Coverage is explicit against the QA inventory.
- Touched scenes and scripts were validated.
- The final scene tree or file structure was rechecked after edits.
- A runtime pass was completed for behavior-affecting changes.
- The response clearly distinguishes between validated behavior and assumptions.
- If the claim depends on player input, timing, or visuals, manual in-engine verification or an existing automated harness was used.
- Any missing evidence is called out explicitly.

## Visual Evidence And Limits

This is still the largest difference from `playwright-interactive`, even after adding runtime tools.

- `godot-mcp` now exposes first-class runtime input and screenshot tools through `godot_runtime_*`, but the workflow is still less browser-like than Playwright.
- The bridge remains split across editor-side control and runtime-side automation. Keep those environments conceptually separate.
- Do not claim visual QA is complete from scene tree and log output alone.
- If a task depends on visuals, layout, animation timing, or gameplay feel, use `godot_runtime_capture_screenshot` or an equivalent project artifact when possible; otherwise inspect the running Godot window manually.
- If the project can save screenshots to disk, review them directly and reference the path.
- If no visual artifact exists, describe the verification limit plainly.

## Common Failure Modes

- `godot_connect` times out: Godot is not running, the AI Bridge plugin is disabled, or the port/host is wrong.
- `Not connected to Godot editor`: reconnect before using `godot_editor_*` tools.
- The shell workspace and the connected editor project differ: if the user asked to work against the running editor, continue on the connected project and mention the difference briefly instead of treating it as a blocker.
- `No scene open`: open a scene explicitly with `godot_editor_open_scene`.
- `Node not found`: reacquire the scene tree and verify the path before issuing another command.
- External edits do not appear in the editor: run `godot_editor_refresh_filesystem`.
- `godot_editor_execute_gdscript` compile failures: inspect `godot_editor_get_errors` and `godot_editor_get_log_file`.
- Runtime automation tools time out or fail: make sure the scene is actually running, the runtime harness autoload is loaded, and the request targets the running game rather than the editor.
- Older Godot or Claude-oriented notes may mention tools that are not in the current MCP surface. Use the reference file below and `godot_help` as the source of truth.

## References

Read [references/live-editor-tool-map.md](references/live-editor-tool-map.md) when you need the current tool names, common payload shapes, or drift notes from the older Godot skill repo.
Read [references/fast-probe-presets.md](references/fast-probe-presets.md) when you need a fast temporary `Label`, `Button`, or `LineEdit` target for runtime or screenshot verification.
Read [references/runtime-automation-extension-points.md](references/runtime-automation-extension-points.md) when you need the current limits, the recommended runtime harness shape, or the future runtime tool set this skill should prefer when it exists.
