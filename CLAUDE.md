# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development
- `bun install` - Install all dependencies across the monorepo
- `bun run dev` - Start the main opencode CLI in development mode (runs packages/opencode/src/index.ts)
- `bun run typecheck` - Run TypeScript type checking across all packages

### Testing
- `bun test` - Run tests using Bun's built-in test framework
- `bun test packages/opencode/test/tool/tool.test.ts` - Run specific test file
- Tests are located in `packages/opencode/test/` and use snapshot testing for tool outputs
- Test fixtures are in `packages/opencode/test/fixtures/`

### Go TUI Development
- `cd packages/tui && go run cmd/opencode/main.go` - Run the Go TUI client
- Requires Go 1.24.x
- The TUI is a separate Go application that communicates with the TypeScript server

### SDK Development
- **Go SDK**: Located in `packages/sdk/go/` with standard Go tooling
- **JavaScript SDK**: Located in `packages/sdk/js/` with TypeScript support
- API client generation uses Stainless SDK generator

## Architecture Overview

### High-Level Structure
This is a client-server architecture AI coding agent with the following key components:

- **packages/opencode/**: Core TypeScript/Node.js server and CLI
- **packages/tui/**: Go-based Terminal User Interface client  
- **packages/web/**: Astro-based documentation website
- **packages/sdk/**: Client SDKs (Go and JavaScript)
- **packages/function/**: Serverless function handlers

### Core Server Components (packages/opencode/src/)

- **cli/**: Command-line interface with commands like `run`, `auth`, `agent`, `serve`, etc.
- **server/**: Hono-based HTTP API server with OpenAPI specs
- **session/**: Chat session management with message handling and modes
- **tool/**: Built-in tools (bash, edit, grep, read, write, lsp-*, etc.)
- **provider/**: AI model provider integrations (Anthropic, OpenAI, etc.)
- **lsp/**: Language Server Protocol client implementation
- **agent/**: AI agent orchestration
- **mcp/**: Model Context Protocol support

### Key Patterns

- **Tool System**: Extensible tool registry in `src/tool/registry.ts` with individual tools like `bash.ts`, `edit.ts`, etc.
- **Provider Agnostic**: Multiple AI providers supported through `src/provider/`
- **Bus System**: Event-driven architecture using `src/bus/`
- **Session Management**: Stateful chat sessions with message history
- **Permission System**: Fine-grained permissions in `src/permission/`

### Client-Server Communication
- The Go TUI client communicates with the TypeScript server via HTTP API
- Server exposes routes defined in `src/server/server.ts`
- Real-time updates via Server-Sent Events (SSE)

### Configuration
- Config management in `src/config/config.ts`
- Supports multiple model providers and custom configurations
- Hook system for pre/post command execution

## Package Management
- Uses Bun as package manager with exact versions (`bunfig.toml`)
- Workspace configuration in root `package.json`
- Shared catalog for common dependencies

## Important Development Notes

### Model Integration
- New AI providers should be added to `src/provider/`
- Model definitions in `src/provider/models.ts` with capabilities and costs
- Provider-specific authentication in `src/auth/`

### Tool Development
- New tools go in `src/tool/` with corresponding `.txt` files for prompts
- Register tools in `src/tool/registry.ts`
- Tools use Zod schemas for input validation

### LSP Integration
- Language server support in `src/lsp/`
- Provides hover info and diagnostics to AI models
- Supports multiple programming languages

### Testing Approach
- Bun test framework for TypeScript code
- Test files in `packages/opencode/test/`
- Snapshot testing for tool outputs in `__snapshots__/` directories
- Integration tests for CLI commands
- Go tests use standard `go test` tooling in TUI and SDK packages

## Configuration and Runtime

### Configuration Management
- Global config file: `opencode.json` in project root
- Supports custom model providers (OpenRouter, HuggingFace, local models)
- MCP (Model Context Protocol) server configurations
- Provider-specific authentication via `src/auth/`

### Package Management
- Uses Bun with exact versions (`bunfig.toml` sets `exact = true`)
- Workspace catalog for shared dependencies in root `package.json`
- Prettier config: `semi: false, printWidth: 120`

## Build and Deployment

### Binary Compilation
- `bun build src/index.ts --compile --outfile ./opencode` - Compile to standalone binary
- Docker deployment uses Bun for TypeScript execution or binary compilation
- Platform-specific packaging for distribution

### Distribution
- npm package: `opencode-ai` 
- Homebrew: `brew install sst/tap/opencode`
- Install script: `curl -fsSL https://opencode.ai/install | bash`