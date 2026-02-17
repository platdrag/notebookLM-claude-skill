# Notebooklm — Claude Code Plugin

A Claude Code plugin that lets you query your [Google NotebookLM](https://notebooklm.google.com) notebooks directly from your coding session. Ask domain-specific questions and get grounded, cited answers from your uploaded sources — without leaving your editor.

## Features

| Skill | Command | Description |
|-------|---------|-------------|
| **Setup** | `/notebooklm:setup` | Install the `notebooklm-py` CLI, authenticate with Google, discover notebooks, and configure your project |
| **Query** | `/notebooklm:query <question>` | Ask questions against your notebooks and get cited answers with source references |

- Multi-turn conversations with follow-up queries
- Per-project notebook configuration (default notebook, multiple named notebooks)
- Structured JSON output with inline citations and source passages
- Cross-platform support (macOS, Linux, Windows)

## Prerequisites

- [Claude Code](https://claude.com/claude-code) v1.0.33+
- Python 3.9+
- A Google account with access to [NotebookLM](https://notebooklm.google.com)

## Installation

### Option A: From GitHub (recommended)

Add the marketplace and install the plugin:

```
/plugin marketplace add platdrag/notebookLM-claude-skill
/plugin install notebooklm@notebooklm-marketplace
```

### Option B: Clone and install locally

```bash
git clone https://github.com/platdrag/notebookLM-claude-skill
```

Then add it as a local marketplace and install:

```
/plugin marketplace add ./notebooklm
/plugin install notebooklm@notebooklm-marketplace
```

To update later, `git pull` inside the clone and run `/plugin marketplace update notebooklm-marketplace`.

### Option C: Load for a single session (development / testing)

```bash
claude --plugin-dir ./notebooklm
```

Loads the plugin for the current session only — no marketplace needed. Useful for trying it out or active development.

### Option D: Manual (no plugin system)

Copy the skill folders into your project's `.claude/skills/` directory:

```bash
cp -r skills/setup  <your-project>/.claude/skills/notebooklm-setup
cp -r skills/query  <your-project>/.claude/skills/notebooklm-query
```

With this method, skills are invoked as `/notebooklm-setup` and `/notebooklm-query` (without the plugin namespace).

## Getting Started

1. Install the plugin (see above)
2. Run `/notebooklm:setup` inside Claude Code
3. Follow the interactive prompts to:
   - Install the `notebooklm-py` CLI
   - Authenticate with your Google account
   - Select and configure your notebooks
4. Start querying: `/notebooklm:query What are the key findings in my research?`

## Usage

```
# Query the default notebook
/notebooklm:query What is the architecture of the evaluation pipeline?

# Query a specific named notebook
/notebooklm:query How are agents structured? --notebook my-research

# Multi-word questions work naturally
/notebooklm:query What design decisions were made for the data layer?
```

## Configuration

After running `/notebooklm:setup`, a config file is created at `<project>/.claude/notebooklm-config.json`:

```json
{
  "defaultNotebook": "my-research",
  "notebooks": {
    "my-research": {
      "id": "abc123-...",
      "description": "Research notes and papers"
    }
  },
  "settings": {
    "mode": "concise",
    "responseLength": "default",
    "persona": null
  }
}
```

### Settings Reference

| Setting | Options | Description |
|---------|---------|-------------|
| `mode` | `default`, `learning-guide`, `concise`, `detailed` | Response style |
| `responseLength` | `default`, `longer`, `shorter` | Response length |
| `persona` | string or `null` | Custom instruction (e.g., "Act as a senior architect") |

A template config is available at [`templates/notebooklm-config.json`](templates/notebooklm-config.json).

## Troubleshooting

### Authentication expired
Auth sessions expire every few weeks. Re-authenticate by running `notebooklm login` in your terminal.

### Command not found
If `notebooklm` is not recognized, install it with:
```bash
pip install "notebooklm-py[browser]"
```

### No notebooks found
Create a notebook at [notebooklm.google.com](https://notebooklm.google.com) and add sources to it before querying.

### Unicode errors on Windows
The skill handles this automatically by setting `PYTHONUTF8=1`, but if you encounter issues, set it in your environment:
```powershell
$env:PYTHONUTF8 = "1"
```

## Plugin Structure

```
notebooklm/
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest
│   └── marketplace.json     # Marketplace catalog (for /plugin install)
├── skills/
│   ├── setup/
│   │   └── SKILL.md         # /notebooklm:setup
│   └── query/
│       └── SKILL.md         # /notebooklm:query
├── templates/
│   └── notebooklm-config.json
├── .gitignore
└── README.md
```

## Dependencies

This plugin relies on [`notebooklm-py`](https://pypi.org/project/notebooklm-py/) — a Python CLI for interacting with Google NotebookLM. The `/notebooklm:setup` skill handles its installation automatically.

## Contributing

1. Fork this repository
2. Create a feature branch
3. Test locally with `claude --plugin-dir ./notebooklm`
4. Submit a pull request

## License

MIT
