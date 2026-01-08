# Cliffy Simplification Plan

## Current State

| Metric | Now | Minimal Target |
|--------|-----|----------------|
| Lines of code | ~15,246 | ~3,500-5,000 |
| Dependencies | 233 | ~60-80 |
| Providers | 6 (OpenAI, Anthropic, Gemini, Azure, Bedrock, VertexAI) | 2 (OpenAI, Anthropic) |
| Tools | 11 | 5-6 |
| Config files | 12 | 2-3 |

## What's Actually Good (Keep)

1. **Provider abstraction** (`internal/llm/provider/`) - Clean interface, just trim to 2 providers
2. **Volley scheduler** (`internal/volley/scheduler.go`) - Smart retry, parallel execution
3. **Message store** (`internal/message/`) - Already minimal, in-memory only
4. **Core tools** - bash, view, edit, grep, glob

## Crush Baggage to Remove

### Immediate Deletions (~1,500 lines)

| Target | Lines | Notes |
|--------|-------|-------|
| `internal/runner/` | 80 | Unused package |
| `cmd/cliffy/init.go` | ~150 | Setup wizard |
| `cmd/cliffy/doctor.go` | ~150 | Diagnostics |
| `cmd/cliffy/preset.go` | ~100 | Preset management |
| `internal/preset/` | 300 | Preset system |
| `internal/llm/prompt/title.go` | ~40 | Session titles |
| `internal/llm/prompt/summarizer.go` | ~240 | Summarization |
| `internal/llm/tools/sourcegraph.go` | 302 | Niche feature |
| `internal/llm/tools/diagnostics.go` | 213 | LSP diagnostics |
| `internal/llm/tools/download.go` | 176 | Rarely used |
| Agent title/summarize providers | ~250 | In agent.go |

### Provider Simplification

**Remove entirely:**
- `internal/llm/provider/gemini.go`
- `internal/llm/provider/azure.go`
- `internal/llm/provider/bedrock.go`
- `internal/llm/provider/vertexai.go`

**Keep:**
- `internal/llm/provider/openai.go` - Also works for OpenRouter
- `internal/llm/provider/anthropic.go`

This eliminates AWS SDK, Azure SDK, and most Google Cloud dependencies.

### Catwalk Replacement

Replace catwalk with ~100 lines of local types:

```go
type ProviderType string
const (
    TypeOpenAI    ProviderType = "openai"
    TypeAnthropic ProviderType = "anthropic"
)

type Model struct {
    ID               string
    Provider         string
    DefaultMaxTokens int64
    CanReason        bool
}
```

Eliminates Prometheus and remote provider fetching.

## Simplified Architecture

```
CLI args
  ↓
Minimal config (env vars + optional JSON)
  ↓
Provider (OpenAI or Anthropic)
  ↓
Agent loop (stream response, execute tools)
  ↓
stdout
```

### Essential Files (~3,500 lines)

```
cmd/cliffy/
  main.go           # CLI entry, flag parsing

internal/
  config/
    config.go       # Minimal types
    load.go         # Simple JSON + env loading

  provider/
    provider.go     # Interface + factory
    openai.go       # OpenAI/OpenRouter
    anthropic.go    # Claude

  agent/
    agent.go        # Core loop (stripped)

  tools/
    bash.go         # Shell commands
    view.go         # Read files
    edit.go         # Modify files
    grep.go         # Search content
    glob.go         # Find files
    write.go        # Create files

  message/
    store.go        # In-memory history

  volley/
    scheduler.go    # Parallel execution
    task.go         # Task types
```

## Minimal go.mod

```go
module github.com/bwl/cliffy

go 1.22

require (
    github.com/anthropics/anthropic-sdk-go v1.12.0
    github.com/openai/openai-go v1.12.0
    github.com/spf13/cobra v1.10.1
    github.com/google/uuid v1.6.0
    github.com/bmatcuk/doublestar/v4 v4.9.1
    mvdan.cc/sh/v3 v3.12.1
)
```

No catwalk, no AWS, no Azure, no GCP, no Prometheus, no MCP, no LSP.

## Implementation Phases

### Phase 1: Delete Dead Code
- Remove `internal/runner/`
- Remove unused CLI commands (init, doctor, preset)
- Remove unused tools (sourcegraph, diagnostics, download)
- Remove title/summarizer prompts

### Phase 2: Strip Agent
- Remove titleProvider, summarizeProvider
- Remove session queue logic
- Remove Summarize() method
- Simplify to single "coder" agent type

### Phase 3: Provider Reduction
- Remove Gemini, Azure, Bedrock, VertexAI providers
- Replace catwalk types with local definitions
- Update config to only support openai/anthropic

### Phase 4: Config Simplification
- Remove LSP config
- Remove MCP config
- Remove TUI options
- Remove preset loading
- Simple: `CLIFFY_API_KEY` + optional `~/.cliffy.json`

### Phase 5: Dependency Cleanup
- Remove catwalk from go.mod
- Run `go mod tidy`
- Verify no cloud SDKs remain

## Decisions

- **Volley scheduler**: Keep - parallel execution is a differentiating feature
- **Providers**: OpenAI + Anthropic (covers GPT-4, Claude, OpenRouter, Ollama)
- **Config**: Env vars + minimal JSON (`CLIFFY_API_KEY`, optional `~/.cliffy.json`)
- **MCP/LSP**: Remove for now (can add back later if needed)

## Final Target

```
~4,000 lines of Go
~60-80 dependencies (down from 233)
2 providers (OpenAI-compatible + Anthropic)
6 tools (bash, view, edit, grep, glob, write)
Parallel execution via volley
Config: CLIFFY_API_KEY + CLIFFY_MODEL + optional JSON
```

## Execution Order

1. **Delete dead code** - runner/, init.go, doctor.go, preset/, unused tools
2. **Strip agent.go** - remove title/summarize providers, session queue
3. **Remove providers** - gemini, azure, bedrock, vertexai
4. **Replace catwalk** - local types (~100 lines)
5. **Simplify config** - remove LSP/MCP/TUI options
6. **go mod tidy** - verify cloud SDKs gone
7. **Test** - ensure core flow works
