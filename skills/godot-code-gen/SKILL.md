# Godot Code Generation Skill

You are an expert Godot 4.x game developer. When generating GDScript code, follow these patterns and best practices.

## GDScript 4.x Syntax

### Type Hints (Always Use)
```gdscript
var speed: float = 200.0
var health: int = 100
var player_name: String = "Player"
var items: Array[String] = []
var stats: Dictionary = {}

func calculate_damage(base: int, multiplier: float) -> int:
    return int(base * multiplier)
```

### Annotations
```gdscript
@export var speed: float = 200.0           # Visible in inspector
@export_range(0, 100) var health: int = 50 # With range slider
@export_enum("Easy", "Medium", "Hard") var difficulty: int = 1
@export_file("*.tscn") var scene_path: String
@export_node_path("Sprite2D") var sprite_path: NodePath

@onready var sprite: Sprite2D = $Sprite2D  # Initialized when ready
@onready var collision: CollisionShape2D = $CollisionShape2D

@tool  # Runs in editor
class_name MyClass  # Global class registration
```

### Signals (Godot 4.x Style)
```gdscript
# Declaration
signal health_changed(new_health: int)
signal died
signal item_collected(item_name: String, count: int)

# Emission
health_changed.emit(current_health)
died.emit()

# Connection (prefer callable syntax)
button.pressed.connect(_on_button_pressed)
player.health_changed.connect(_on_player_health_changed)

# With binds
timer.timeout.connect(_on_timeout.bind(extra_arg))

# One-shot connection
signal_name.connect(callable, CONNECT_ONE_SHOT)
```

### Common Node Patterns

#### CharacterBody2D Movement
```gdscript
extends CharacterBody2D

@export var speed: float = 200.0
@export var jump_force: float = 400.0

var gravity: float = ProjectSettings.get_setting("physics/2d/default_gravity")

func _physics_process(delta: float) -> void:
    # Gravity
    if not is_on_floor():
        velocity.y += gravity * delta

    # Jump
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = -jump_force

    # Horizontal movement
    var direction := Input.get_axis("move_left", "move_right")
    velocity.x = direction * speed

    move_and_slide()
```

#### CharacterBody3D Movement
```gdscript
extends CharacterBody3D

@export var speed: float = 5.0
@export var jump_velocity: float = 4.5

var gravity: float = ProjectSettings.get_setting("physics/3d/default_gravity")

func _physics_process(delta: float) -> void:
    if not is_on_floor():
        velocity.y -= gravity * delta

    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_velocity

    var input_dir := Input.get_vector("move_left", "move_right", "move_forward", "move_back")
    var direction := (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()

    if direction:
        velocity.x = direction.x * speed
        velocity.z = direction.z * speed
    else:
        velocity.x = move_toward(velocity.x, 0, speed)
        velocity.z = move_toward(velocity.z, 0, speed)

    move_and_slide()
```

#### State Machine
```gdscript
extends Node

enum State { IDLE, WALK, RUN, JUMP, FALL }

var current_state: State = State.IDLE

func _physics_process(delta: float) -> void:
    match current_state:
        State.IDLE:
            _process_idle(delta)
        State.WALK:
            _process_walk(delta)
        State.JUMP:
            _process_jump(delta)

func change_state(new_state: State) -> void:
    if new_state == current_state:
        return

    _exit_state(current_state)
    current_state = new_state
    _enter_state(new_state)

func _enter_state(state: State) -> void:
    match state:
        State.IDLE:
            animation_player.play("idle")
        State.WALK:
            animation_player.play("walk")

func _exit_state(state: State) -> void:
    pass
```

#### Resource Pattern
```gdscript
# item_data.gd
class_name ItemData
extends Resource

@export var name: String
@export var description: String
@export var icon: Texture2D
@export var value: int
@export var stackable: bool = true
@export var max_stack: int = 99

# Using resources
var sword: ItemData = preload("res://items/sword.tres")
print(sword.name)
```

#### Autoload/Singleton
```gdscript
# game_manager.gd - Add to Project Settings > Autoload
extends Node

signal game_paused
signal game_resumed

var score: int = 0
var is_paused: bool = false

func pause() -> void:
    is_paused = true
    get_tree().paused = true
    game_paused.emit()

func resume() -> void:
    is_paused = false
    get_tree().paused = false
    game_resumed.emit()

func add_score(points: int) -> void:
    score += points
```

### Async/Await (Replaces Yield)
```gdscript
# Wait for signal
await get_tree().create_timer(1.0).timeout

# Wait for animation
await animation_player.animation_finished

# Custom async function
func load_level_async(path: String) -> void:
    var loader := ResourceLoader.load_threaded_request(path)
    while ResourceLoader.load_threaded_get_status(path) == ResourceLoader.THREAD_LOAD_IN_PROGRESS:
        await get_tree().process_frame
    var scene := ResourceLoader.load_threaded_get(path)
    get_tree().change_scene_to_packed(scene)
```

### Tweens (Godot 4.x)
```gdscript
# Simple tween
var tween := create_tween()
tween.tween_property(sprite, "modulate:a", 0.0, 1.0)

# Chained tweens
var tween := create_tween()
tween.tween_property(node, "position", Vector2(100, 100), 0.5)
tween.tween_property(node, "scale", Vector2(2, 2), 0.3)
tween.tween_callback(func(): print("Done!"))

# Parallel tweens
var tween := create_tween()
tween.set_parallel(true)
tween.tween_property(node, "position:x", 100, 0.5)
tween.tween_property(node, "rotation", PI, 0.5)

# Easing and transitions
tween.set_trans(Tween.TRANS_BOUNCE)
tween.set_ease(Tween.EASE_OUT)
```

### Scene Instantiation
```gdscript
# Preload (compile-time, preferred for frequently used scenes)
const EnemyScene := preload("res://scenes/enemy.tscn")
var enemy := EnemyScene.instantiate()
add_child(enemy)

# Load (runtime, for dynamic loading)
var scene := load("res://scenes/enemy.tscn")
var enemy := scene.instantiate()

# With type hints
var enemy: Enemy = EnemyScene.instantiate() as Enemy
```

### Input Handling
```gdscript
func _input(event: InputEvent) -> void:
    if event.is_action_pressed("attack"):
        attack()

    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            shoot()

func _unhandled_input(event: InputEvent) -> void:
    # Only receives events not handled by UI
    if event.is_action_pressed("pause"):
        toggle_pause()

# Polling in _process/_physics_process
var direction := Input.get_vector("left", "right", "up", "down")
var is_jumping := Input.is_action_just_pressed("jump")
var is_attacking := Input.is_action_pressed("attack")
```

## Best Practices

1. **Always use type hints** - Improves performance and catches errors early
2. **Use @onready for node references** - Ensures nodes exist when accessed
3. **Prefer signals over direct method calls** - Loose coupling
4. **Use Resources for data** - Reusable, inspector-editable data containers
5. **Group related functionality** - Use composition over inheritance
6. **Name private variables with underscore** - `_health` not `health`
7. **Use PascalCase for classes, snake_case for functions/variables**
8. **Document with ## comments** - Creates in-editor documentation

## Common Gotchas

- `yield` is gone, use `await`
- `connect("signal", self, "method")` is gone, use `signal.connect(callable)`
- `instance()` is now `instantiate()`
- Physics layers start at 1, not 0
- `export` keyword is now `@export` annotation
- `onready` keyword is now `@onready` annotation
- `tool` keyword is now `@tool` annotation
