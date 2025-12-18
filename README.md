# Claude Coding

A Claude Code plugin that exports conversation threads to shareable links via GitHub Gists.

## Features

- Exports Claude Code sessions to self-contained HTML files
- Creates shareable preview links via GitHub Gists
- **Session Linking**: Navigate between related sessions with Previous/Next links
  - When you use `/clear` to start a new session, it automatically links to the previous session
  - Exported gists include navigation to browse your session history
- Clean, minimal UI inspired by [ampcode.com](https://ampcode.com) thread sharing
- Proper markdown rendering (headers, bold, code blocks, lists, links)
- Collapsible sections for thinking blocks and tool results
- Smart tool display:
  - **WebFetch/WebSearch**: Shows clickable URL + prompt
  - **Read/Glob/Grep**: Shows file path
  - **Bash**: Shows command description or truncated command
  - **Edit**: Shows file diffs
- Auto-extracts title from first user message or session summary
- Detects system username and generates avatar initials

## Installation

### Prerequisites
- [Go](https://go.dev/dl/) 1.20+
- Claude Code installed and set up
- `gh` CLI installed and authenticated (for Gist creation)

### Install as Claude Code Plugin
```bash
# 1. Install the CLI binary
go install github.com/priyanshujain/claude-coding/cmd/claude-coding@latest

# 2. Start Claude Code
claude

# 3. Add the marketplace from GitHub
/plugin marketplace add priyanshujain/claude-coding

# 4. Install the plugin
/plugin install claude-coding@priyanshujain
```

## Usage

### Slash Command (Recommended)

After installing the plugin, use the `/share` command in Claude Code:

```
/claude-coding:share
```

This creates a GitHub Gist and returns a shareable preview link.

### Session Linking with /clear

When you use `/clear` to start a new conversation within the same project, the plugin automatically tracks session relationships:

1. Your sessions form a linked chain (Session A → Session B → Session C)
2. When you `/share`, the exported HTML includes navigation links
3. Viewers can browse through your session history using Previous/Next links
4. All linked session gists are automatically updated with the correct navigation


## How It Works

Claude Code stores conversation data in JSONL files at:
```
~/.claude/projects/{encoded-project-path}/{session-id}.jsonl
```

When you run `/share`:
1. The plugin finds your current session
2. Parses the conversation messages
3. Converts to a self-contained HTML file with syntax highlighting
4. Creates/updates a GitHub Gist
5. Returns a preview URL via [gistpreview.github.io](https://gistpreview.github.io)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup, architecture details, and coding guidelines.

## License

Apache 2.0
