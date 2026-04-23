# AGENTS.md

## Project Overview

**Claude Context** is an MCP (Model Context Protocol) plugin that adds semantic code search to Claude Code and other AI coding agents. It indexes codebases into a vector database and enables natural-language search over millions of lines of code.

Repository: https://github.com/zilliztech/claude-context

---

## Repository Structure

```
claude-context/
├── packages/
│   ├── core/              # @zilliz/claude-context-core — core indexing engine
│   ├── mcp/               # @zilliz/claude-context-mcp  — MCP server
│   ├── vscode-extension/  # VSCode extension (semanticcodesearch)
│   └── chrome-extension/  # Chrome extension
├── examples/
│   └── basic-usage/       # Basic Node.js usage example
├── docs/                  # Documentation
├── scripts/               # Build scripts
└── python/                # Python bindings (if any)
```

### Package Descriptions

| Package | Name | Purpose |
|---------|------|---------|
| `packages/core` | `@zilliz/claude-context-core` | Embedding providers, vector DB client, AST/LangChain code splitters, synchronizer |
| `packages/mcp` | `@zilliz/claude-context-mcp` | MCP server exposing `index_codebase`, `search_code`, `clear_index`, `get_indexing_status` tools |
| `packages/vscode-extension` | `semanticcodesearch` | VS Code extension wrapping the core package |
| `packages/chrome-extension` | — | Chrome extension for browser-based usage |

---

## Prerequisites

- **Node.js** >= 20.0.0 and **< 24.0.0** (Node 24 is not supported)
- **pnpm** >= 10.0.0

---

## Setup

```bash
pnpm install
```

---

## Build Commands

```bash
# Build all packages (respects dependency order)
pnpm build

# Build individual packages
pnpm build:core      # packages/core
pnpm build:mcp       # packages/mcp
pnpm build:vscode    # packages/vscode-extension

# Watch mode (all packages)
pnpm dev

# Watch individual packages
pnpm dev:core
pnpm dev:mcp
pnpm dev:vscode

# Clean build artifacts
pnpm clean
```

---

## Lint & Type-Check

```bash
# Lint all packages
pnpm lint

# Auto-fix lint issues
pnpm lint:fix

# TypeScript type-check (no emit)
pnpm typecheck
```

---

## Tests

Tests live in `packages/core` and use **Jest** with `ts-jest`.

```bash
# Run tests for core package
cd packages/core
pnpm test
```

No project-wide test script exists; run tests from inside the relevant package directory.

---

## Environment Variables

Copy `.env.example` to `~/.context/.env` (global) or supply variables directly when starting the MCP server. **Do not place this file in a codebase directory being indexed.**

| Variable | Default | Description |
|----------|---------|-------------|
| `EMBEDDING_PROVIDER` | `OpenAI` | `OpenAI`, `VoyageAI`, `Gemini`, or `Ollama` |
| `EMBEDDING_MODEL` | provider-specific | Embedding model name |
| `EMBEDDING_BATCH_SIZE` | `100` | Batch size for embedding calls |
| `OPENAI_API_KEY` | — | Required for OpenAI provider |
| `OPENAI_BASE_URL` | — | Optional custom OpenAI endpoint |
| `VOYAGEAI_API_KEY` | — | Required for VoyageAI provider |
| `GEMINI_API_KEY` | — | Required for Gemini provider |
| `GEMINI_BASE_URL` | — | Optional custom Gemini endpoint |
| `OLLAMA_HOST` | `http://127.0.0.1:11434` | Ollama server host |
| `OLLAMA_MODEL` | `nomic-embed-text` | Ollama model (alternative to `EMBEDDING_MODEL`) |
| `MILVUS_ADDRESS` | — | Milvus/Zilliz Cloud public endpoint |
| `MILVUS_TOKEN` | — | Milvus/Zilliz authentication token |
| `SPLITTER_TYPE` | `ast` | `ast` (syntax-aware) or `langchain` (character-based) |
| `CUSTOM_EXTENSIONS` | — | Comma-separated extra file extensions (e.g. `.vue,.svelte`) |
| `CUSTOM_IGNORE_PATTERNS` | — | Comma-separated extra ignore patterns |
| `HYBRID_MODE` | — | `true` enables dense + BM25 hybrid search |

---

## MCP Tools (packages/mcp)

The MCP server exposes four tools:

| Tool | Required params | Description |
|------|----------------|-------------|
| `index_codebase` | `path` (absolute) | Index a codebase directory; optional `force`, `splitter`, `customExtensions`, `ignorePatterns` |
| `search_code` | `path`, `query` | Semantic search; optional `limit` (max 50), `extensionFilter` |
| `clear_index` | `path` (absolute) | Remove an index |
| `get_indexing_status` | `path` (absolute) | Check indexing progress / completion status |

---

## Core Package Architecture (packages/core/src)

```
src/
├── context.ts          # Main Context class (entry point)
├── types.ts            # Shared TypeScript types
├── embedding/          # Embedding providers
│   ├── base-embedding.ts
│   ├── openai-embedding.ts
│   ├── voyageai-embedding.ts
│   ├── gemini-embedding.ts
│   └── ollama-embedding.ts
├── vectordb/           # Vector database clients
│   ├── milvus-vectordb.ts
│   └── milvus-restful-vectordb.ts
├── splitter/           # Code chunking
│   ├── ast-splitter.ts      # Tree-sitter AST-based splitter
│   └── langchain-splitter.ts
├── sync/               # File-system synchronizer
└── utils/
```

---

## Commit Guidelines

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `perf`, `chore`  
**Scopes:** `core`, `vscode`, `mcp`, `chrome`, `examples`, `docs`

Examples:
```
feat(core): add new embedding provider
fix(mcp): handle missing MILVUS_ADDRESS gracefully
docs(readme): update quick-start instructions
```

---

## Key Development Notes

- **MCP stdio protocol**: All logs in `packages/mcp` must go to **stderr** (`console.log` is redirected; never write JSON to stdout except for MCP protocol messages).
- **Absolute paths**: All MCP tool `path` parameters must be absolute filesystem paths.
- **Node 24 incompatibility**: Ensure Node.js version stays between 20 and 23 (inclusive).
- **pnpm workspaces**: This is a monorepo — always run commands from the repo root or use the workspace filter flags (`pnpm --filter <package>`).
- **TypeScript strict mode**: All packages use strict TypeScript. Avoid `any` where possible.
