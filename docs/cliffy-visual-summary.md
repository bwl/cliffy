# Cliffy Visual Design System - Summary

> A tennis-inspired visual language for parallel task processing

**Version**: 1.0-draft
**Status**: Design Complete, Ready for Implementation
**Full Docs**: See `cliffy-visual-*.md` files in this directory

---

## Core Concept

Cliffy visualizes parallel task execution using **Unicode symbols** and a **tennis/volley metaphor**:
- Tasks are tennis balls (○) volleyed across a court
- Workers are players (⬢) processing the balls
- Each ball fills up (◔◑◕) as it's processed
- Finished balls (●) exit the system

**Goal**: Instant visual understanding of task state without reading text.

---

## Essential Symbols (18 core)

### Task States - Progressive Filling
```
○  Queued       - Empty circle, waiting to start
◔  Initializing - 25% filled, loading context
◑  Processing   - 50% filled, actively working
◕  Finalizing   - 75% filled, wrapping up
●  Complete     - 100% filled, done
◌  Failed       - Dotted circle, error occurred
```

### Tool States
```
▣  Success      - Tool completed successfully
▥  Running      - Tool currently executing
▩  Failed       - Tool returned error
```

### Worker States
```
⬡  Idle         - Worker available
⬢  Active       - Worker processing task
⬣  Overloaded   - Worker at capacity
```

### Status & Flow
```
⚠  Warning      - Retry or attention needed
✗  Error        - Fatal error
⟲  Retry        - Task retrying
⤴  Return       - Error return path
⏸  Paused       - Waiting/delayed
```

### Structure
```
╮  Branch       - Task branches to tools
├  Mid          - Middle item
╰  Last         - Last item
───  Line       - Horizontal connector
◍  Racket       - Cliffy branding
```

---

## Visual Examples

### Simple Task
```
1   ● analyze auth.go (2.3s, 3.2k tokens)
```

### Task with Tools (Collapsed)
```
1 ╮ ● refactor auth [read grep edit]  4.5k tokens $0.0056  3.8s
```

### Task with Tools (Expanded)
```
1 ╮ ◑ refactor authentication (worker 1)
  ├───▣ read     auth.go  0.2s
  ├───▣ grep     password  0.1s
  ╰───▥ edit     auth.go  editing...
```

### Parallel Execution (3 tasks)
```
◍═══╕  3 tasks volleyed
    ╰──╮ Using claude-3-5-sonnet

1 ╮ ● analyze [read grep]  2.1k $0.0026  1.8s
2 ╮ ◕ refactor (worker 1)
  ├───▣ edit     db.go  0.4s
  ╰───▥ bash     go test  running...
3   ◑ update tests (worker 2)

◍ 1/3 complete, 2 running
```

### Retry Scenario
```
2   ◌ api call (attempt 2) ⟲
    ⤴ Error: rate limited (429)
    ⏸ Retrying in 2.0s...
```

### Worker Pool View (Optional Feature)
```
◍═══╕  5 tasks volleyed ⬢⬢⬢ (3 workers)

║ Worker 1 ⬢ ║ → 1  analyze auth.go
║ Worker 2 ⬢ ║ → 2  refactor db.go
║ Worker 3 ⬡ ║ (idle)
║═══════════║
║   Queue   ║ → 3  build project
              → 4  run tests
              → 5  deploy
```

---

## Design Principles

1. **Progressive Disclosure**: Empty (○) → Filled (●) shows natural progression
2. **At-a-Glance Status**: Scan for ● (done) vs ◌ (failed) instantly
3. **Hierarchical Display**: Tree structure (╮├╰) shows task→tool relationships
4. **Parallel Visibility**: Multiple tasks display simultaneously with worker assignments
5. **Terminal-Friendly**: Works everywhere Unicode does, no color required
6. **Distinctive Style**: Tennis metaphor creates memorable "Cliffy look"

---

## Implementation Status

### Current (v0.1 - Existing)
✅ Basic symbols: ○●◴◵◶◷▣☒╮├╰───◍
✅ Progress tracking with spinner animation
✅ Tool execution display
✅ Tree structure for task→tool hierarchy

### Planned (v1.0 - Core Enhancement)
🔲 Processing states: ◔◑◕◌
🔲 Tool states: ▥▩
🔲 Retry visualization: ⟲⤴⏸
🔲 Error indicators: ⚠✗

### Optional (v1.1+ - Advanced)
🔲 Worker view: ⬡⬢⬣
🔲 Flow arrows: →⇒⇨
🔲 Metrics display
🔲 Timeline view

---

## Implementation Phases

**Phase 1** (2-4 hrs): Add 40+ symbols to `internal/llm/tools/ascii.go`
**Phase 2** (4-6 hrs): Implement progressive states (○◔◑◕●)
**Phase 3** (3-5 hrs): Enhanced tool visualization (▥▩)
**Phase 4** (6-8 hrs): Worker pool display (⬡⬢⬣) - optional
**Phase 5** (4-6 hrs): Retry/error visualization (⟲⤴⚠)
**Phase 6** (8-12 hrs): Advanced features (metrics, timeline) - future

**Total Core Effort**: 2-3 weeks for Phases 1-3+5

---

## File Impact

### Files to Modify
```
internal/llm/tools/ascii.go       +40 lines   - Symbol constants
internal/volley/progress.go       +100 lines  - Rendering logic
internal/volley/task.go           +20 lines   - Task states
internal/llm/agent/agent.go       +20 lines   - Progress events
internal/volley/scheduler.go      +30 lines   - Event handling
```

### Files to Create (Optional)
```
internal/volley/worker_visualizer.go  ~150 lines  - Worker display
internal/llm/tools/execution.go       ~100 lines  - Tool state tracking
```

**Total**: ~850 new lines, ~220 modified lines

---

## Symbol Categories Summary

| Category | Count | Example | Priority |
|----------|-------|---------|----------|
| Task States | 8 | ○◔◑◕●◌⦿◉ | High |
| Tool States | 7 | ▤▥▣▦▩☒▫ | High |
| Worker States | 4 | ⬡⬢⬣⎔ | Low |
| Flow Indicators | 10 | →⇒⇨⟲⤴↻ | Medium |
| Status | 6 | ⚠✗⊗⚡⏸⊘ | Medium |
| Tree Structure | 4 | ╮├╰─── | Done |
| Containers | 11 | □▢█▮▯║═╬ | Low |
| Branding | 2 | ◍ ᕕ( ᐛ )ᕗ | Done |

**Total**: 60+ symbols across 8 categories

---

## Reading the Output

### Task Line Anatomy
```
[#] [tree] [state] [description] [worker] [tools] [metrics] [time]
 1    ╮      ◑     refactor auth  (worker 1)  [read edit]  2.1k  1.8s
```

### State Progression
```
Lifecycle: ○ → ◔ → ◑ → ◕ → ●
           queue  init  work  wrap  done

With retry: ○ → ◔ → ◌ ⟲ ⏸ → ◔ → ●
            queue init fail retry wait retry success
```

### Tool Traces
```
├───▣  Success (green in future)
├───▥  Running (yellow in future)
╰───▩  Failed (red in future)
```

---

## Quick Start

### For Users
1. **○ = queued**, **●●● = done**, **◌ = failed** - that's 90% of what you need
2. Watch circles fill up: ○ → ◔ → ◑ → ◕ → ●
3. Tree branches (╮) show tool details below
4. ⟲ means retrying, ⚠ means warning

### For Developers
1. Start with **Phase 1**: Add symbols to `ascii.go` (quick win)
2. Then **Phase 2**: Wire up state transitions (○◔◑◕●)
3. Test with: `./bin/cliffy --verbose "analyze auth.go"`
4. See full docs for implementation details

### For Designers
1. All symbols are Unicode standard
2. Monospace-safe (fixed width)
3. Terminal-tested across platforms
4. Future: Color mappings defined for TUI

---

## Resources

**Full Documentation** (in `docs/`):
- `cliffy-visual-system.md` - Complete design spec (800 lines)
- `cliffy-visual-mockups.md` - Visual examples (600 lines)
- `cliffy-visual-implementation-plan.md` - Technical roadmap (700 lines)
- `cliffy-visual-quick-reference.md` - User guide (250 lines)
- `cliffy-visual-inventory.md` - Asset inventory (600 lines)

**Code References**:
- `internal/llm/tools/ascii.go` - Symbol constants
- `internal/volley/progress.go` - Progress rendering
- `internal/volley/task.go` - Task definitions

**External**:
- [Unicode Character Table](https://unicode-table.com/)
- Terminal emulator testing list in implementation plan

---

## Key Decisions

✅ **Progressive states** (◔◑◕) over pure spinner - more informative
✅ **Worker view** is opt-in (`--workers-view` flag) - avoid clutter
✅ **Auto-collapse** completed tasks - save screen space
✅ **Monochrome first**, color later - works everywhere
✅ **Tennis metaphor** throughout - distinctive and fun

---

## Success Metrics

**Must Have**:
- [ ] Phases 1-2 implemented (symbols + states)
- [ ] All existing tests pass
- [ ] Works in major terminals (iTerm, GNOME Terminal, Windows Terminal)
- [ ] Users understand state at a glance

**Should Have**:
- [ ] Phase 3+5 implemented (tools + retry/error)
- [ ] Test coverage >80%
- [ ] User feedback positive

**Nice to Have**:
- [ ] Worker view (Phase 4)
- [ ] Advanced features (Phase 6)
- [ ] Color support (future TUI)

---

## The Essence

**Before** (current):
```
1   ● task completed (2.3s)
```

**After** (enhanced):
```
1 ╮ ○ task starting...
1 ╮ ◔ task initializing (worker 1)
  ├───▥ read     file.go  reading...
1 ╮ ◑ task processing (worker 1)
  ├───▣ read     file.go  0.2s
  ├───▥ edit     file.go  editing...
1 ╮ ◕ task finalizing (worker 1)
  ├───▣ read     file.go  0.2s
  ├───▣ edit     file.go  0.4s
  ╰───▥ bash     go test  running...
1 ╮ ● task completed [read edit bash]  3.2k tokens $0.0039  2.3s
```

**Value**: Transform task execution from "black box with spinner" to "transparent process you can follow step-by-step."

---

**Status**: 🎾 Design complete. Ready to volley!

**Next**: Implement Phase 1 (add symbols to `ascii.go`)
**Timeline**: 2-3 weeks for core features
**Effort**: ~1,000 lines of code + tests

See full documentation for complete details.
