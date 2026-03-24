# Godot Claude Skills

A Claude Code skill pack for Godot 4.x game development. It provides reusable knowledge and workflow skills that complement `godot-mcp`.

> **🎮 Best used with [godot-mcp](https://github.com/alexmeckes/godot-mcp)**
>
> This plugin provides the *knowledge* (best practices, patterns, workflows), while **godot-mcp** provides the *tools* (reading/writing scenes, scripts, shaders, and live editor control). Install both for the full Godot + Claude experience.

## Skills Included

| Skill | Description |
|-------|-------------|
| `godot-code-gen` | GDScript best practices, type hints, signals, state machines |
| `godot-live-edit` | Lightweight live-editor guidance via the AI Bridge plugin |
| `godot-interactive` | Advanced persistent `godot-mcp` + AI Bridge workflow for inspect/edit/run/debug loops |
| `godot-scene-design` | Scene files (.tscn), node hierarchies, level layouts |
| `godot-shader` | Shader authoring for 2D/3D effects and post-processing |

## Which Live Skill To Use

- Use `godot-interactive` when you are working through `godot-mcp` and want a persistent, evidence-driven editor/runtime loop.
- Use `godot-live-edit` when you want simpler live-edit guidance without the fuller session workflow.

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
- **Interactive live sessions** - Inspect, edit, run, debug, and automate a Godot project through `godot-mcp`
- **Live editing** - Control the Godot editor in real-time via the AI Bridge plugin
- **Scene design** - Best practices for .tscn files, node hierarchies, collision layers
- **Shaders** - 2D/3D shader patterns, uniforms, post-processing effects

## Requirements

- [Claude Code](https://claude.ai/code) with plugin support
- [Godot 4.x](https://godotengine.org/)
- **[godot-mcp](https://github.com/alexmeckes/godot-mcp)** - MCP server that gives Claude the ability to read/write Godot files and control the editor (highly recommended)

## Plugin Structure

```
godot-claude-skills/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest
├── skills/
│   ├── godot-code-gen/   # GDScript best practices
│   ├── godot-live-edit/  # Lightweight live-editor guidance
│   ├── godot-interactive/ # Advanced persistent MCP + AI Bridge workflow
│   ├── godot-scene-design/ # Scene file patterns
│   └── godot-shader/     # Shader authoring
└── README.md
```

## License

MIT
