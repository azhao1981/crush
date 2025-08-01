# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Running
```bash
# Run the application
go run .

# Run with profiling enabled (enables pprof at localhost:6060)
task dev
# or
CRUSH_PROFILE=true go run .

# Generate JSON schema for configuration
task schema
# or
go run main.go schema > schema.json
```

### Testing
```bash
# Run all tests
task test
# or
go test ./...

# Run tests with specific arguments
task test -- -run TestSpecificFunction -v
```

### Code Quality
```bash
# Run linters
task lint

# Run linters and auto-fix issues
task lint-fix

# Format code with gofumpt
task fmt
```

### Profiling
```bash
# Start profiling server
task dev

# View CPU profile (10s)
task profile:cpu

# View heap profile
task profile:heap

# View allocations profile
task profile:allocs
```

## Architecture Overview

Crush is a terminal-based AI assistant built with Go and the Bubble Tea TUI framework. The application provides an interactive chat interface with AI capabilities, code analysis, and LSP integration.

### Core Components

**Application Structure (`internal/app/`):**
- `app.go` - Main application orchestration, manages services and lifecycle
- Handles LSP clients, sessions, messages, permissions, and agent coordination
- Uses event-driven architecture with pub/sub pattern

**Command Structure (`internal/cmd/`):**
- `root.go` - Cobra CLI setup and main command structure
- `run.go` - Main application execution logic
- `logs.go` - Log viewing commands
- `schema.go` - Configuration schema generation

**Configuration (`internal/config/`):**
- JSON-based configuration with cascading priority (local → global)
- Provider management for different LLM services
- LSP and MCP server configurations
- Permission system for tool execution

**TUI (`internal/tui/`):**
- Bubble Tea-based terminal interface
- Chat interface with message rendering
- File picker, dialogs, and session management
- Syntax highlighting and markdown rendering

**LLM Integration (`internal/llm/`):**
- Provider abstraction for multiple AI services (OpenAI, Anthropic, Gemini, etc.)
- Agent system for tool execution and reasoning
- Prompt management and templating
- MCP (Model Context Protocol) support

**Database (`internal/db/`):**
- SQLite storage for sessions and messages
- Schema migrations with goose
- Query generation with sqlc

**LSP Integration (`internal/lsp/`):**
- Client for Language Server Protocol communication
- Provides code intelligence and context
- File watching and workspace management

### Key Design Patterns

**Service Architecture:** Each major component (sessions, messages, history, etc.) implements service interfaces for clean separation of concerns.

**Event-Driven:** Uses pub/sub pattern for communication between components, with channels for message passing.

**Tool System:** Extensible tool framework where AI agents can execute various operations (file operations, shell commands, web requests, etc.).

**Configuration Hierarchy:** Supports multiple configuration files with defined precedence: `.crush.json` → `crush.json` → `~/.config/crush/crush.json`.

### Configuration

The application uses JSON configuration with schema validation. Key sections include:
- `providers` - LLM service configurations
- `lsp` - Language server settings
- `mcp` - Model Context Protocol servers
- `permissions` - Tool execution permissions
- `options` - Debug and runtime settings

### Development Notes

- The application uses SQLite for data persistence with migrations in `internal/db/migrations/`
- LSP clients are managed per language and support hot-reloading
- The tool permission system requires explicit approval for potentially dangerous operations
- Session management allows multiple conversation contexts per project
- Logging is available via `crush logs` command with real-time following support