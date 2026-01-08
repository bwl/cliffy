# Cliffy

**The fastest way to get LLM intelligence into your terminal.**

```
$ cliffy "how much energy from the sun reaches Earth each day?"

Breakdown:
  - Solar constant: ~1361 W/m² at the top of Earth's atmosphere.
  - Earth's cross-sectional area: πR² ≈ 1.27 × 10¹⁴ m².
  - Total power intercepted: ~1.74 × 10¹⁷ W.
  - Per day: 1.74 × 10¹⁷ W × 86,400 s ≈ 1.5 × 10²² J/day.

Context:
  - About 30% is reflected back to space.
  - Roughly 50–55% is absorbed by Earth's surface; the rest by the atmosphere.
  - This daily energy exceeds all human energy use by ~10,000×.

≈1.5 × 10²² joules per day reach Earth from the Sun.
```

---

## What is Cliffy?

Cliffy is a headless LLM assistant that lives in your terminal. It brings AI knowledge and action directly into your filesystem and standard I/O streams.

**Core philosophy:**
- **Fast** — Sub-second startup. No database. No sessions. Just ask.
- **Unix-native** — Plays nice with pipes, stdin, stdout, stderr
- **Parallel** — Send multiple tasks, get results back efficiently
- **Focused** — One job: bridge your terminal to LLM intelligence

---

## The Name

Cliffy is a **CLI** tool. The name also evokes tennis — volleying tasks back and forth. The tool uses tennis-themed symbols throughout: rackets `◍`, balls, volleys.

```
   ◍═══════════════════════════╕
   │  CLIFFY                   │
   │  ᕕ( ᐛ )ᕗ                  │
   │  LLM volleys for your CLI │
   ╰═══════════════════════════╛
```

---

## Quick Examples

### Direct queries
```bash
cliffy "explain the CAP theorem in one paragraph"
cliffy "what's the mass of Jupiter in kilograms?"
```

### Working with code
```bash
cliffy "find all TODO comments in this project"
cliffy "review auth.go for security issues"
cliffy "commit the feature we just finished"
```

### Batch tasks (Volley Mode)
```bash
cliffy "analyze auth.go" "analyze db.go" "analyze api.go"
cliffy "make 20 mobs for the NPC system. look at the 5 we built for inspo"
```

### Pipeline integration
```bash
cat error.log | cliffy "what went wrong here?"
cliffy "list all exported functions" | grep -i auth
git diff | cliffy "write a commit message for this"
```

---

## Core Features

### 1. Single Task Execution

The simplest usage. Ask a question, get an answer.

```bash
cliffy "<your prompt>"
```

Output goes to stdout. Errors go to stderr. Clean and composable.

### 2. Volley Mode (Parallel Execution)

Send multiple tasks at once. Cliffy handles:
- Parallel execution with configurable concurrency
- Automatic retry with exponential backoff
- Rate limit handling (you don't worry about 429s)
- Progress tracking
- Ordered results

```bash
cliffy "task one" "task two" "task three"
```

**Why "Volley"?** Like tennis — you serve tasks, Cliffy returns results. Back and forth, fast and efficient.

### 3. Tool Execution

Cliffy can take action, not just answer questions. Built-in tools:

| Tool | Purpose |
|------|---------|
| `bash` | Execute shell commands |
| `view` | Read file contents |
| `edit` | Modify existing files |
| `write` | Create new files |
| `glob` | Find files by pattern |
| `grep` | Search file contents |

When you say "commit the feature we just finished", Cliffy will:
1. Run `git diff` to see changes
2. Analyze the changes
3. Write an appropriate commit message
4. Execute `git commit`

### 4. Context Awareness

Cliffy automatically loads project context from common files:
- `.cursorrules`
- `CLAUDE.md`
- `.github/copilot-instructions.md`
- And others

You can also provide explicit context:
```bash
cliffy --context "You are a security expert" "review this code"
cliffy --context-file system-prompt.txt "analyze the architecture"
```

---

## Command Line Interface

### Basic Syntax

```
cliffy [flags] <task> [task...]
```

### Model Selection

| Flag | Description |
|------|-------------|
| `--fast`, `-f` | Use smaller, faster model |
| `--smart`, `-s` | Use larger, more capable model |
| `--model`, `-m` | Specify exact model ID |

### Output Control

| Flag | Description |
|------|-------------|
| `--quiet`, `-q` | Results only. No progress, no stats. |
| `--verbose`, `-v` | Everything. Tool traces, thinking, stats. |
| `--output-format`, `-o` | Format: `text`, `json`, or `diff` |
| `--stats` | Show token usage and cost |
| `--show-thinking`, `-t` | Display LLM reasoning process |

### Execution Control

| Flag | Description |
|------|-------------|
| `--max-concurrent` | Worker pool size (default: 3) |
| `--task-timeout` | Per-task timeout (e.g., `60s`, `2m`) |

### Input Methods

| Flag | Description |
|------|-------------|
| `-` | Read task from stdin |
| `--tasks-file` | Read tasks from file (one per line) |
| `--json` | Parse input as JSON array |

---

## Configuration

### Environment Variables

```bash
# Primary API key (OpenRouter recommended for multi-model access)
export CLIFFY_API_KEY="sk-or-..."

# Or provider-specific
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Config File

Location: `~/.config/cliffy/cliffy.json` or `.cliffy.json` in project root.

```json
{
  "models": {
    "large": {
      "model": "claude-3-opus",
      "provider": "anthropic"
    },
    "small": {
      "model": "gpt-4o-mini",
      "provider": "openai"
    }
  },
  "providers": {
    "openai": {
      "api_key": "${OPENAI_API_KEY}",
      "type": "openai"
    },
    "anthropic": {
      "api_key": "${ANTHROPIC_API_KEY}",
      "type": "anthropic"
    }
  }
}
```

### Provider Types

Cliffy supports two provider API formats:
- **OpenAI-compatible** — Works with OpenAI, OpenRouter, Ollama, vLLM, etc.
- **Anthropic** — Claude models via Anthropic API

---

## Output Formats

### Text (Default)

Clean, human-readable output.

**Single task:**
```
The answer to your question is 42.
```

**Multiple tasks (Volley):**
```
◍ Task 1/3: analyze auth.go
Authentication looks solid. Uses bcrypt with cost factor 12...
[2.1k tokens in 3.2s]

◍ Task 2/3: analyze db.go
Database layer has a potential SQL injection at line 45...
[1.8k tokens in 2.8s]

◍ Task 3/3: analyze api.go
API endpoints are well-structured...
[2.4k tokens in 3.5s]

═══ Summary ═══
3 tasks completed in 9.5s
6.3k total tokens ($0.019)
```

### JSON

Machine-readable for pipelines and automation.

```bash
cliffy --output-format json "list all functions" | jq '.results[0].result'
```

```json
{
  "results": [
    {
      "task": "list all functions",
      "status": "success",
      "result": "...",
      "metadata": {
        "duration_ms": 2450,
        "tokens": { "input": 500, "output": 200 },
        "model": "gpt-4o",
        "tools_used": ["glob", "view"]
      }
    }
  ],
  "summary": {
    "total_tasks": 1,
    "succeeded": 1,
    "failed": 0,
    "total_duration_ms": 2450,
    "total_tokens": 700
  }
}
```

---

## Visual Language

Cliffy uses Unicode symbols for a clean, informative display.

### Status Indicators

| Symbol | Meaning |
|--------|---------|
| `◍` | Task complete (tennis racket head) |
| `○` | Task queued |
| `◴` `◵` `◶` `◷` | Task in progress (spinner) |
| `✗` | Task failed |

### Tool Traces

When `--verbose`, tool execution shows as a tree:

```
◍ Task 1/2: fix the bug in parser.go
╭─────────────────────────────
├───▣ view    parser.go (245 lines)
├───▣ grep    "parseToken" (3 matches)
├───▣ edit    parser.go:67-89
╰───▣ bash    go test ./... (passed)

Fixed the off-by-one error in parseToken()...
```

| Symbol | Meaning |
|--------|---------|
| `▣` | Tool succeeded |
| `☒` | Tool failed |
| `├───` | Tool in sequence |
| `╰───` | Last tool |

### Progress Display

During volley execution:

```
◍═╕   3 tasks volleyed
  ╰─╮ Using claude-3-opus
1   ◴ analyze auth.go
2   ◵ analyze db.go
3   ○ analyze api.go (queued)
```

---

## Retry & Error Handling

### Automatic Retry

Cliffy automatically retries on:
- Rate limits (HTTP 429)
- Timeouts
- Network errors

Retry uses exponential backoff: 1s → 2s → 4s → 8s (capped at 60s).

### No Retry

These errors fail immediately:
- Authentication errors (401, 403)
- Bad request (400)
- Model not found

### Exit Codes

| Code | Meaning |
|------|---------|
| `0` | All tasks succeeded |
| `1` | At least one task failed |

---

## Power User Workflows

### Hotkey Integration

Bind Cliffy to a hotkey in your terminal or editor. Quick commands without breaking flow:

```bash
# In your shell config
alias c='cliffy'

# Or with tmux/terminal hotkey
bind-key C run-shell "cliffy"
```

### Delegation Pattern

Working with a full coding agent (like Claude Code)? Use Cliffy for side tasks:

```bash
# While Claude Code handles the main feature...
cliffy "update the changelog for this release"
cliffy "run the linter and summarize issues"
cliffy "check if any tests are flaky"
```

### Batch Processing

```bash
# Process a file of tasks
cliffy --tasks-file review-checklist.txt --max-concurrent 5

# Generate from a list
echo "write a haiku about ${topic}" | cliffy -
```

### CI/CD Integration

```bash
# In your pipeline
cliffy --json --quiet "analyze this diff for security issues" > report.json
```

---

## Specification Details

This section provides implementation-level detail for building Cliffy.

### Input Processing

1. **Arguments**: Each positional argument after flags is a task
2. **Stdin**: When `-` is provided, read from stdin
3. **File**: When `--tasks-file` is provided, read line by line (skip empty lines and `#` comments)
4. **JSON mode**: Parse input as JSON array of strings

### Message Format

Each task becomes a conversation:

```
System: [loaded context files + any --context]
User: [task text]
Assistant: [LLM response, may include tool calls]
```

Tool calls follow a loop:
1. LLM responds with tool call request
2. Cliffy executes tool
3. Tool result sent back to LLM
4. Repeat until LLM gives final response

### Tool Schema

Each tool must define:
- **name**: Unique identifier
- **description**: What the tool does (shown to LLM)
- **parameters**: JSON Schema of accepted parameters
- **execute**: Function that runs the tool and returns result

### Streaming

- Response tokens stream to stdout as they arrive
- Tool traces go to stderr
- Progress indicators update in-place on stderr

### Volley Scheduler

The parallel execution engine:

1. Create worker pool (size = `--max-concurrent`)
2. Queue all tasks
3. Workers pull tasks and execute
4. On failure, check if retryable
5. Retryable errors: re-queue with backoff delay
6. Collect results, preserve original order
7. Output as tasks complete (text) or all at once (json)

### Configuration Loading

Order of precedence (later overrides earlier):
1. Built-in defaults
2. `~/.config/cliffy/cliffy.json`
3. `.cliffy.json` or `cliffy.json` in project
4. Environment variables
5. Command-line flags

### Provider Interface

Implementations must support:
- `stream_response(messages, tools) -> AsyncIterator[Event]`
- Events: `content_delta`, `tool_use`, `error`, `done`

---

## Design Principles

1. **Speed over features** — Fast startup, minimal dependencies
2. **Unix philosophy** — Do one thing well, compose with other tools
3. **Transparent** — Show what's happening (tools, tokens, cost)
4. **Forgiving** — Auto-retry, helpful errors, sensible defaults
5. **Delightful** — Tennis theme, clean output, satisfying UX

---

## Example Session

```bash
$ cliffy "make 20 mobs for the NPC system. look at the 5 we built for inspo"

◍ Analyzing existing NPCs...
├───▣ glob    src/npcs/*.json (5 files)
├───▣ view    src/npcs/goblin.json
├───▣ view    src/npcs/skeleton.json
├───▣ view    src/npcs/wolf.json
├───▣ view    src/npcs/bandit.json
├───▣ view    src/npcs/slime.json

Creating 20 new mobs based on the existing patterns...

├───▣ write   src/npcs/orc.json
├───▣ write   src/npcs/troll.json
├───▣ write   src/npcs/wraith.json
... (17 more)

Created 20 new NPCs:
- orc, troll, wraith, vampire, werewolf, zombie, ghost,
- spider, scorpion, bat, rat, snake, bear, boar, deer,
- elemental_fire, elemental_ice, elemental_earth, golem, mimic

All follow the established schema with balanced stats.
[8.2k tokens in 12.4s]
```

---

## Summary

Cliffy is:
- A **CLI tool** for LLM queries and actions
- **Fast** — no startup overhead, no persistence
- **Parallel** — volley multiple tasks efficiently
- **Unix-native** — stdin, stdout, stderr, pipes, exit codes
- **Configurable** — env vars, JSON config, flags
- **Visual** — tennis-themed Unicode, progress tracking
- **Practical** — auto-retry, error handling, cost tracking

```
   ᕕ( ᐛ )ᕗ

   $ cliffy "let's go"
```
