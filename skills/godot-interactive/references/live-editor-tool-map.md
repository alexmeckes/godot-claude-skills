# Live Editor Tool Map

Use this file only when you need the exact current `godot-mcp` live-editor surface. The main skill stays workflow-focused; this file is the low-level reference.

## Connection

```text
godot_connect {}
godot_connection_status {}
godot_disconnect {}
```

Notes:

- Prefer `godot_connect {}` so the active `godot_mcp` configuration chooses the correct host and port.
- `host` defaults to `127.0.0.1` when provided explicitly.
- `port` often defaults to `6550`, but local MCP config may override it.
- Reconnect after Godot restarts or the bridge closes

## Scene And Editor State

```text
godot_editor_get_project_info {}
godot_editor_get_scene_tree {}
godot_editor_open_scene { "scenePath": "res://scenes/main.tscn" }
godot_editor_select_node { "nodePath": "Player/Sprite2D" }
godot_editor_save_scene {}
godot_editor_run_scene {}
godot_editor_run_scene { "scenePath": "res://scenes/test_room.tscn" }
godot_editor_stop_scene {}
godot_editor_refresh_filesystem {}
```

## Live Node Editing

```text
godot_editor_add_node {
  "parentPath": ".",
  "name": "HUD",
  "type": "CanvasLayer"
}

godot_editor_modify_node {
  "nodePath": "HUD",
  "properties": {
    "visible": true
  }
}

godot_editor_remove_node {
  "nodePath": "HUD/DebugLabel"
}
```

Important argument names:

- `parentPath` for add-node parent selection
- `nodePath` for select/modify/remove
- `scenePath` for open-scene and optional run-scene targeting

## Typed Property Payloads

The current AI Bridge converts a few structured dictionaries automatically.

### Vector2

```json
{
  "_type": "Vector2",
  "x": 100.0,
  "y": 200.0
}
```

### Vector3

```json
{
  "_type": "Vector3",
  "x": 1.0,
  "y": 2.0,
  "z": 3.0
}
```

### Color

```json
{
  "_type": "Color",
  "r": 1.0,
  "g": 0.5,
  "b": 0.0,
  "a": 1.0
}
```

Example:

```text
godot_editor_modify_node {
  "nodePath": "Player",
  "properties": {
    "position": { "_type": "Vector2", "x": 100, "y": 200 },
    "scale": { "_type": "Vector2", "x": 2, "y": 2 }
  }
}
```

## Runtime Diagnostics

```text
godot_editor_get_errors {
  "includeRuntime": true,
  "includeScript": true,
  "includeLogFile": true,
  "severity": "all",
  "clear": false
}

godot_editor_get_output {
  "lines": 100,
  "level": "all",
  "source": "all",
  "includeMetadata": true,
  "clear": false
}

godot_editor_get_log_file {
  "lines": 200,
  "filter": "all",
  "sinceLine": 0,
  "includeMetadata": true
}

godot_editor_execute_gdscript {
  "code": "return editor_interface.get_edited_scene_root().name"
}
```

Guidance:

- Use `clear: false` unless you are deliberately starting a fresh capture window.
- Prefer short, diagnostic `godot_editor_execute_gdscript` snippets that return data.
- Use `godot_editor_get_log_file` when you need incremental polling or broader context than the in-memory output buffer.

## Runtime Automation

```text
godot_runtime_status {}

godot_runtime_wait {
  "frames": 6
}

godot_runtime_press_action {
  "action": "move_right",
  "strength": 1.0
}

godot_runtime_release_action {
  "action": "move_right"
}

godot_runtime_tap_action {
  "action": "jump",
  "frames": 1
}

godot_runtime_mouse_move {
  "x": 120,
  "y": 180
}

godot_runtime_click {
  "x": 120,
  "y": 180,
  "button": 1,
  "holdFrames": 1
}

godot_runtime_type_text {
  "text": "Player One"
}

godot_runtime_capture_screenshot {
  "path": "tmp/runtime-smoke.png"
}
```

Guidance:

- `godot_runtime_*` targets the running game process, not the editor tree.
- Call `godot_editor_run_scene` before runtime automation.
- Use `godot_runtime_status` after the scene is running when you need viewport size, pointer state, or confirmation that a previous run is still active.
- For common temporary UI targets, read [fast-probe-presets.md](fast-probe-presets.md) before querying docs or `godot_help`.
- Prefer `godot_runtime_mouse_move` + `godot_runtime_click` for UI interactions.
- Prefer `godot_runtime_tap_action` or press/release pairs for InputMap-driven gameplay.
- `godot_runtime_capture_screenshot` accepts absolute paths plus `res://`, `user://`, and project-relative paths.
- Pair screenshots with logs, runtime status, or state assertions when making behavioral claims.

## File-Based Tools To Pair With Live Editing

Use these when the live bridge is not enough or when batch edits are cleaner in files:

- `godot_read_scene`
- `godot_write_scene`
- `godot_add_node`
- `godot_modify_node`
- `godot_validate_scene`
- `godot_read_script`
- `godot_write_script`
- `godot_validate_script`
- `godot_read_shader`
- `godot_write_shader`
- `godot_read_resource`
- `godot_write_resource`
- `godot_ui_*`
- `godot_animation_*`
- `godot_input_*`
- `godot_audio_*`
- `godot_navigation_*`

When the exact tool choice is unclear, call `godot_help`.

## Known Drift From Older Godot Skill Notes

Older Claude-oriented skill text may mention tools that are not present in the current `godot-mcp` surface. Do not assume those still exist.

Known mismatches:

- `godot_editor_get_node_properties` is not present
- `godot_editor_get_node_types` is not present
- `godot_editor_list_assets_by_type` is not present
- The current live-edit surface uses `parentPath`, `nodePath`, and `scenePath`

Use `godot_help` and the actual MCP tool list as the source of truth.
