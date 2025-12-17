# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
make build          # Build to bin/claude-share
make install        # Build and install to /usr/local/bin (requires sudo)
make clean          # Remove build artifacts
```

## Testing

```bash
./bin/claude-share --project "$PWD" --output test.html
open test.html
```

## Architecture

This is a Claude Code plugin that exports conversation threads to shareable HTML files.

### Data Flow

1. **Session Discovery** (`internal/parser/jsonl.go`): Locates JSONL session files in `~/.claude/projects/{encoded-project-path}/`. Project paths are encoded by replacing `/` and `.` with `-`.

2. **JSONL Parsing** (`internal/parser/jsonl.go`): Parses Claude Code session files. Each line contains a JSON object with type, uuid, timestamp, and message fields. Content blocks can be text, thinking, tool_use, or tool_result.

3. **HTML Conversion** (`internal/converter/html.go`): Converts parsed messages to HTML. Handles:
   - Tool result merging (inserts results after their corresponding tool_use)
   - Markdown rendering (headers, bold, code blocks, lists, links)
   - Tool-specific rendering (WebFetch/WebSearch show URLs, Read shows file paths, Bash shows commands with descriptions, Edit shows diffs)

4. **Template** (`internal/template/template.go`): Self-contained HTML template with inline CSS and Prism.js for syntax highlighting.

### Multi-Session Support

When multiple Claude Code sessions run in parallel, the plugin uses a SessionStart hook (`.claude-plugin/scripts/capture-session.sh`) to capture the session ID. The `/share` slash command (`commands/share.md`) passes `$CLAUDE_SESSION_ID` to ensure the correct session is exported.

### Key Types

- `parser.Message`: Parsed message with ID, Role, Timestamp, and Blocks
- `parser.ContentBlock`: Content block with Type, Content, ToolName, ToolUseID, ToolInput, IsError
- `converter.Config`: HTML generation config with Title, Username, UserInitials, ProjectPath

### Coding Guidelines
- do not comment code unless abosolutely necessary
- use `any` instead `interface{}`
- do not use json tags in every struct, create separate struct for marshaling/unmarshaling