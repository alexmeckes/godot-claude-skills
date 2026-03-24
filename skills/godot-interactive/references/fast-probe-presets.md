# Fast Probe Presets

Use this file when you need a temporary, deterministic runtime target quickly. These presets are for feedback speed, not final UI design.

Rules:

- Prefer these presets before docs lookup for common `Control` probes.
- Put probes under a live parent you already confirmed from `godot_editor_get_scene_tree`.
- Save and validate after adding the probe.
- Remove the probe and save again after verification unless the user wants to keep it.

## Shared Geometry

These payloads assume a top-left anchored control:

```json
{
  "visible": true,
  "anchor_left": 0,
  "anchor_top": 0,
  "anchor_right": 0,
  "anchor_bottom": 0,
  "position": { "_type": "Vector2", "x": 32, "y": 32 },
  "size": { "_type": "Vector2", "x": 360, "y": 48 }
}
```

This makes the probe easy to hit near the upper-left corner of the viewport and easy to spot in screenshots.

## Label Probe

Use this for quick visual proof.

```text
godot_editor_add_node {
  "parentPath": "Main/UI",
  "name": "FeedbackProbeLabel",
  "type": "Label",
  "properties": {
    "visible": true,
    "anchor_left": 0,
    "anchor_top": 0,
    "anchor_right": 0,
    "anchor_bottom": 0,
    "position": { "_type": "Vector2", "x": 32, "y": 32 },
    "size": { "_type": "Vector2", "x": 360, "y": 48 },
    "text": "FAST LOOP OK"
  }
}
```

## Button Probe

Use this when you need a visible clickable control but do not need text entry.

```text
godot_editor_add_node {
  "parentPath": "Main/UI",
  "name": "FeedbackProbeButton",
  "type": "Button",
  "properties": {
    "visible": true,
    "anchor_left": 0,
    "anchor_top": 0,
    "anchor_right": 0,
    "anchor_bottom": 0,
    "position": { "_type": "Vector2", "x": 32, "y": 32 },
    "size": { "_type": "Vector2", "x": 360, "y": 48 },
    "text": "CLICK PROBE"
  }
}
```

## LineEdit Probe

Use this for focus and text-entry verification.

```text
godot_editor_add_node {
  "parentPath": "Main/UI",
  "name": "FeedbackProbeLineEdit",
  "type": "LineEdit",
  "properties": {
    "visible": true,
    "anchor_left": 0,
    "anchor_top": 0,
    "anchor_right": 0,
    "anchor_bottom": 0,
    "position": { "_type": "Vector2", "x": 32, "y": 32 },
    "size": { "_type": "Vector2", "x": 360, "y": 48 },
    "placeholder_text": "TYPE HERE",
    "text": ""
  }
}
```

## Minimal Runtime Loop

Use this after the probe was added, saved, and validated:

```text
godot_editor_run_scene {}
godot_runtime_mouse_move { "x": 120, "y": 55 }
godot_runtime_click {}
godot_runtime_type_text { "text": "feedback loop" }
godot_runtime_wait { "frames": 15 }
godot_runtime_capture_screenshot { "path": "res://tmp/runtime-probe.png" }
```

Notes:

- The click point above assumes the default probe geometry at `(32, 32)` with size `(360, 48)`.
- If the project scales or repositions the control at runtime, inspect the screenshot or runtime state and adjust the click point instead of switching to broad docs lookup.
- If the probe fails to appear, inspect the exact node path and properties before escalating to broader log review.
