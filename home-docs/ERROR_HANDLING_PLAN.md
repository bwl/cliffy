# Error Handling Improvements Plan

## Goals
- Keep the volley progress display readable even when providers fail repeatedly.
- Detect misconfigured or unavailable models before volley execution so the CLI surfaces a single actionable error instead of repeated provider logs.

## Proposed Changes

### 1. Integrate Provider Errors Into Progress Tracker
- **Extend metadata collection** in `internal/volley/scheduler.go` so each failure emits a structured event (error class, HTTP status, message).
- **Augment `ProgressTracker`** (`internal/volley/progress.go`) with a new method, e.g. `TaskProviderError`, that:
  - Updates the task state to “failed” or “retrying”.
  - Prints a single condensed line (status icon + task prompt + human-friendly error summary).
  - Collapses repeated identical provider errors into a counter (`(x5)`), avoiding log spam.
- **Suppress raw slog output** during volley execution once progress tracker handles the message; errors still reach stderr after execution via a summary block.
- **Update verbose mode behaviour** so `--verbose` can optionally show the raw provider payload for debugging.
- **Cover with tests** in `internal/volley/progress_test.go` to verify:
  - Single errors render inline without breaking the spinner.
  - Repeated errors collapse into one entry.
  - Quiet mode still exits with appropriate status codes.

### 2. Fail Fast on Provider Misconfiguration
- **Validate models before execution** in `cmd/cliffy/main.go` by checking `config.Config.GetModelByType` and provider availability for the selected or preset-adjusted model.
- **Add checks in `applyPresetToConfig`** to ensure the preset’s model ID exists in the resolved provider list; return a descriptive error when missing.
- **Introduce a doctor helper** (shared with `cliffy doctor`) that confirms the provider advertises the requested model ID and endpoint.
- **Write unit tests** in `cmd/cliffy/main_test.go` (or new `config` focused tests) covering:
  - Missing model produces an immediate, clear error message.
  - Preset overrides detect stale model IDs.
  - Manual `--model` flag with invalid type fails before volley start.

## Implementation Order
1. Add configuration validation to catch missing models (fast feedback for users).
2. Introduce structured provider-error reporting in the scheduler and progress tracker.
3. Tidy slog routing so default runs rely on the tracker, while debug mode still exposes raw logs.
4. Update documentation (`README.md`, HELP output) describing the new behaviour and error troubleshooting flow.

## Validation Strategy
- Run unit tests and targeted integration tests (`internal/volley` suite) to confirm progress display behaviour.
- Manual smoke test with a known-bad model (`--model nonexistent`) to confirm fail-fast messaging.
- Manual volley against a provider forced to 404 to confirm condensed error rendering.
