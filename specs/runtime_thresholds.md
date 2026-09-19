# Runtime Thresholds — Complete Reference

All numeric policies in one place. Sources: `compression.go`, `agent.go`, `task_template.json`, `grouping.go`.

## Token budgets

| Threshold | Value | Meaning |
|---|---|---|
| `tokenSoftThreshold` | **60%** of `MAX_TOKENS` | Trigger **background async** memory compression |
| `tokenWarningThreshold` | **80%** of `MAX_TOKENS` | Trigger **immediate sync** compression; also used as max prompt size per group |
| Per-group diff-size limit | **80%** of `MAX_TOKENS` | Individual file with diff > this is excluded as `ExcludeTooLarge` before review |
| Budget check frequency | Before each group dispatch | Aggregate `used + estimate(group) > MaxTokensBudget` → stop scheduling |

Prompt token limit formula: `int(float64(MAX_TOKENS) * 0.80)`

With default `MAX_TOKENS = 200000`:
- Soft threshold: 120,000 tokens → async compression starts
- Hard threshold / per-group max: 160,000 tokens → sync compression / file excluded

## Tool call caps

| Cap | Value | Scope |
|---|---|---|
| `MAX_TOOL_REQUEST_TIMES` | **100** | Per main-loop conversation (one group × one round) |
| Tool failure streak | **3 consecutive failures** | Same (taskKey, toolName) pair → stop retrying, return "accepted" |

Tool failure escalation policy (from `tool_failure_streak.go`):
- Failure 1: return plain error message
- Failure 2: error + "This is the second consecutive failure calling X. Fix your arguments, or call task_done."
- Failure 3+: "X has failed repeatedly and has been skipped. Move on or call task_done." (silent discard of finding)

## Group formation

| Parameter | Default | File |
|---|---|---|
| `GROUPING_MIN_FILES` | **4** | Below this → skip LLM grouping entirely |
| `GROUPING_BUNDLE_LINE_THRESHOLD` | **200** | Below this (and < min files) → bundle all into one group |
| `maxFilesPerGroup` | **10** | Hard cap; split any larger group |
| Default concurrency | **8** groups simultaneously | Semaphore limit |

## Plan phase triggers

`PlanRequired(fileCount, totalChanged, maxFileChanged)` returns true if EITHER:

- `maxFileChanged >= PLAN_MODE_LINE_THRESHOLD (50)` — any single file changed ≥ 50 lines, OR
- `fileCount >= 2 AND totalChanged >= PLAN_MODE_GROUP_LINE_THRESHOLD (100)` — multi-file group with ≥ 100 total changed lines

## Review rounds

| Setting | Default | Meaning |
|---|---|---|
| `MAX_REVIEW_ROUNDS` | **2** | Max main-task passes per group |
| Plan stripping | Round ≥ 2 | Plan text removed from user prompt on round 2+ |
| Confirmed findings | Round 2+ | Prior round findings injected as `<confirmed_findings>` |

## Output caps

| Setting | Default |
|---|---|
| `MAX_COMPLETION_TOKENS` | **16384** |
| Confirmed comments max | **30** entries, ExistingCode ≤ 200 runes, Content ≤ 300 runes |

## Tokenizer

OCR uses **cl100k_base** (tiktoken) for token counting. Use the same tokenizer for `DiffTokens` calculation in `selectFiles` and for all prompt budget checks. Approximation (characters / 4) is not accurate enough for the 80% threshold math.

## Token counting scope

`CountMessagesTokens(msgs)` = sum of `ExtractText()` for each message. Includes system prompt on every count (the system message repeats across rounds and is counted each time).
