# Godot Claude Skills

A Claude Code plugin for Godot 4.x game development. Provides specialized skills that enhance Claude's ability to work with Godot projects.

## Skills Included

| Skill | Description |
|-------|-------------|
| `godot-code-gen` | GDScript best practices, type hints, signals, state machines |
| `godot-live-edit` | Real-time Godot editor control via AI Bridge plugin |
| `godot-scene-design` | Scene files (.tscn), node hierarchies, level layouts |
| `godot-shader` | Shader authoring for 2D/3D effects and post-processing |

## Installation

### From GitHub (Recommended)

```bash
# Add the plugin marketplace
/plugin marketplace add alexmeckes/godot-claude-skills

# Install the plugin
/plugin install godot-claude-skills
```

### Local Development

```bash
# Clone the repository
git clone https://github.com/alexmeckes/godot-claude-skills.git

# Test locally with Claude Code
claude --plugin-dir ./godot-claude-skills
```

## Usage

Once installed, Claude will automatically use these skills when working on Godot projects. The skills provide context for:

- **GDScript patterns** - Type hints, signals, state machines, async/await, tweens
- **Live editing** - Control the Godot editor in real-time via the AI Bridge plugin
- **Scene design** - Best practices for .tscn files, node hierarchies, collision layers
- **Shaders** - 2D/3D shader patterns, uniforms, post-processing effects

## Requirements

- [Claude Code](https://claude.ai/code) with plugin support
- [Godot 4.x](https://godotengine.org/)
- [godot-mcp](https://github.com/alexmeckes/godot-mcp) - MCP server for Godot file operations (optional)
- [godot-ai-bridge](https://github.com/alexmeckes/godot-ai-bridge) - Godot plugin for real-time editor control (optional, for live editing)

## Plugin Structure

```
godot-claude-skills/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest
├── skills/
│   ├── godot-code-gen/   # GDScript best practices
│   ├── godot-live-edit/  # Real-time editor control
│   ├── godot-scene-design/ # Scene file patterns
│   └── godot-shader/     # Shader authoring
└── README.md
```

## License

MIT
