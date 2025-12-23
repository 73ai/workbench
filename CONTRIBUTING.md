# Contributing to Claude Coding

Thank you for your interest in contributing! This document covers development setup, architecture, and coding guidelines.

## Development Setup

### Prerequisites

- [Go](https://go.dev/dl/) 1.20+
- `gh` CLI installed and authenticated
- Claude Code installed

### Build from Source

```bash
# Clone the repository
git clone https://github.com/73ai/workbench.git
cd workbench

# Build and install
go install ./cmd/workbench

# Install the plugin in Claude Code
claude
/plugin marketplace add ./
/plugin install workbench@73ai
```

### Testing

```bash
# Build the binary
go install ./cmd/workbench

# Test HTML export
workbench share --project "$PWD" --output test.html
open test.html

# Test gist creation
workbench share --project "$PWD" --gist
```

## Architecture

### Project Structure

```
workbench/
├── cmd/workbench/       # CLI entry point
│   └── main.go
├── internal/
│   ├── parser/              # JSONL session parsing
│   │   └── jsonl.go
│   ├── converter/           # HTML conversion
│   │   └── html.go
│   ├── gist/                # GitHub Gist operations
│   │   └── gist.go
│   └── template/            # HTML template
│       └── template.go
├── generic/
│   └── metadata/            # Session metadata storage
│       └── metadata.go
├── commands/                # Slash commands
│   └── share.md
└── .claude-plugin/          # Plugin configuration
    ├── manifest.json
    └── scripts/
        └── capture-session.sh
```

### Data Flow

1. **Session Discovery** (`internal/parser/jsonl.go`)
   - Locates JSONL session files in `~/.claude/projects/{encoded-project-path}/`
   - Project paths are encoded by replacing `/` and `.` with `-`

2. **JSONL Parsing** (`internal/parser/jsonl.go`)
   - Parses Claude Code session files
   - Each line contains a JSON object with type, uuid, timestamp, and message fields
   - Content blocks can be text, thinking, tool_use, or tool_result

3. **HTML Conversion** (`internal/converter/html.go`)
   - Converts parsed messages to HTML
   - Handles tool result merging (inserts results after their corresponding tool_use)
   - Markdown rendering (headers, bold, code blocks, lists, links)
   - Tool-specific rendering (WebFetch/WebSearch show URLs, Read shows file paths, Bash shows commands, Edit shows diffs)

4. **Template** (`internal/template/template.go`)
   - Self-contained HTML template with inline CSS
   - Prism.js for syntax highlighting

5. **Gist Operations** (`internal/gist/gist.go`)
   - Creates and updates GitHub Gists via `gh` CLI
   - Generates preview URLs using gistpreview.github.io

### Session Linking

Sessions are linked in a doubly-linked list structure stored in `workbench-metadata.json`:

```
~/.claude/projects/{encoded-project-path}/workbench-metadata.json
```

When `/clear` is used:
1. The current session ID is captured via the SessionStart hook
2. Metadata links the new session to the previous one (prev/next pointers)
3. When sharing, adjacent sessions' gists are updated with navigation links

Key metadata operations are in `generic/metadata/metadata.go`:
- `GetPrevSessionID` / `GetNextSessionID` - Navigate the session chain
- `GetGistID` / `SetGistID` - Track gist IDs for each session
- `WithLock` - Thread-safe metadata operations using file locks

### JSONL Format

Each line in a session file is a JSON object:

```json
{
  "type": "user|assistant",
  "uuid": "message-uuid",
  "parentUuid": "parent-message-uuid",
  "sessionId": "session-id",
  "timestamp": "ISO-8601 timestamp",
  "message": {
    "role": "user|assistant",
    "content": "string or array of content blocks"
  }
}
```

Content block types:
- `{"type": "text", "text": "..."}` - Plain text
- `{"type": "thinking", "thinking": "..."}` - Claude's thinking
- `{"type": "tool_use", "name": "...", "input": {...}}` - Tool invocation
- `{"type": "tool_result", "content": "..."}` - Tool output


You can read more about how claude code stores threads [here](https://kentgigger.com/posts/claude-code-conversation-history).

### Key Types

- `parser.Message` - Parsed message with ID, Role, Timestamp, and Blocks
- `parser.ContentBlock` - Content block with Type, Content, ToolName, ToolUseID, ToolInput, IsError
- `converter.Config` - HTML generation config with Title, Username, UserInitials, ProjectPath, PrevSessionURL, NextSessionURL
- `metadata.Session` - Session metadata with PrevSessionID, NextSessionID, GistID, UpdatedAt

## Coding Guidelines

- Do not comment code unless absolutely necessary
- Use `any` instead of `interface{}`
- Do not use json tags in every struct; create separate structs for marshaling/unmarshaling
- Keep functions focused and small
- Handle errors explicitly; don't ignore them silently

## Submitting Changes

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with `go install ./cmd/workbench`
5. Submit a pull request

## License

By contributing, you agree that your contributions will be licensed under the Apache 2.0 License.
