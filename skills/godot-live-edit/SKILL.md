# Godot Live Edit Skill

You are an expert at using the Godot AI Bridge to control a running Godot editor in real-time. This skill guides you through live editing workflows.

## Connection Workflow

### Step 1: Connect to Godot
```
First, ensure:
1. Godot 4.x is running with your project open
2. The godot-ai-bridge plugin is installed and enabled
3. The plugin shows "AI Bridge listening on port 6550" in output

Then use: godot_connect
```

### Step 2: Verify Connection
```
Use: godot_connection_status

Expected response:
{
  "connected": true,
  "port": 6550,
  "projectPath": "/path/to/project"
}
```

### Step 3: Explore the Scene
```
Use: godot_editor_get_scene_tree

This shows all nodes in the current scene with their types and hierarchy.
```

## Common Live Edit Operations

### Adding UI Elements
```
To add a health bar:

1. godot_editor_add_node
   - parent: "HUD" (or ".")
   - name: "HealthBar"
   - type: "ProgressBar"
   - properties: { "value": 100, "max_value": 100 }

2. godot_editor_modify_node
   - nodePath: "HUD/HealthBar"
   - properties: {
       "position": {"_type": "Vector2", "x": 20, "y": 20},
       "size": {"_type": "Vector2", "x": 200, "y": 30}
     }
```

### Modifying Node Properties
```
To reposition a player:

godot_editor_modify_node
- nodePath: "Player"
- properties: {
    "position": {"_type": "Vector2", "x": 100, "y": 200},
    "scale": {"_type": "Vector2", "x": 2, "y": 2}
  }
```

### Inspecting Nodes
```
To see all properties of a node:

godot_editor_get_node_properties
- nodePath: "Player"
- filter: "exported"  (or "all" for everything)
```

### Building Scene Hierarchies
```
To create a character scene structure:

1. Root: godot_editor_add_node (type: CharacterBody2D, name: Player)
2. Sprite: godot_editor_add_node (parent: Player, type: Sprite2D)
3. Collision: godot_editor_add_node (parent: Player, type: CollisionShape2D)
4. Camera: godot_editor_add_node (parent: Player, type: Camera2D)
```

### Running and Testing
```
Test your changes:

1. godot_editor_save_scene  - Save current work
2. godot_editor_run_scene   - Run and test
3. godot_editor_stop_scene  - Stop when done
4. godot_editor_get_errors  - Check for issues
```

## Property Type Formatting

When modifying properties via live editing, use these JSON formats:

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

### Simple Values
```json
{
  "visible": true,
  "rotation": 1.57,
  "speed": 200.0,
  "text": "Hello World"
}
```

## Workflow Patterns

### Iterative UI Design
```
1. Connect to editor
2. Get current scene tree
3. Add container nodes (VBoxContainer, HBoxContainer)
4. Add UI elements (Label, Button, ProgressBar)
5. Adjust positions/sizes with modify_node
6. Run scene to preview
7. Iterate until satisfied
8. Save scene
```

### Level Building
```
1. Connect to editor
2. Instantiate pre-made scenes (enemies, items, platforms)
3. Position them with modify_node
4. Duplicate nodes as needed
5. Run to test gameplay
6. Adjust and iterate
```

### Debugging
```
1. Connect to editor
2. Run scene
3. Use get_errors to check for problems
4. Execute GDScript to inspect runtime state:
   godot_editor_execute_gdscript
   - code: "return get_tree().current_scene.get_node('Player').position"
5. Stop scene
6. Fix issues
```

## Best Practices

1. **Always save before running** - `godot_editor_save_scene` first
2. **Work incrementally** - Add one node, verify, then continue
3. **Use descriptive names** - "PlayerHealthBar" not "ProgressBar2"
4. **Check the scene tree** - Use `godot_editor_get_scene_tree` to verify structure
5. **Inspect before modifying** - Use `godot_editor_get_node_properties` to see current values
6. **Use undo if needed** - All operations support Ctrl+Z in the editor
7. **Refresh filesystem** - After external file changes, use `godot_editor_refresh_filesystem`

## Error Handling

Common issues:

| Error | Cause | Solution |
|-------|-------|----------|
| "Not connected" | Plugin not running | Start Godot, enable plugin |
| "Node not found" | Wrong path | Use get_scene_tree to find correct path |
| "No scene open" | No scene loaded | Open a scene in Godot first |
| "Parent not found" | Invalid parent path | Check parent exists in tree |
| "Failed to create node" | Invalid node type | Verify type name (case-sensitive) |

## Available Node Types

Use `godot_editor_get_node_types` to list all instantiable node types:
```
godot_editor_get_node_types
- base_type: "Control"  // List all UI nodes
- base_type: "Node2D"   // List all 2D nodes
- base_type: "Node3D"   // List all 3D nodes
```

## Scene Management

```
# List all scenes in project
godot_editor_list_assets_by_type
- type: "scenes"

# Open a different scene
godot_editor_open_scene
- scenePath: "res://scenes/main_menu.tscn"

# Save as new scene
godot_editor_save_scene_as
- path: "res://scenes/level_2.tscn"

# Instantiate a scene as child
godot_editor_instantiate_scene
- scene_path: "res://scenes/enemy.tscn"
- parent: "Enemies"
- name: "Enemy1"
```
