# Exact Runtime Pipeline

One `review` run, step by step. Rebuild this sequence; do not invent a looser flow.

## Phase 0 — Resolve input

Accept one of:

- **workspace**: staged + unstaged + untracked
- **range**: `--from A --to B` (merge-base mode recommended)
- **commit**: single commit vs parent

Produce `[]Diff` with at least: `OldPath`, `NewPath`, unified `Diff` text, `NewFileContent`, flags (`IsBinary`, `IsDeleted`, `IsNew`, `IsRenamed`), `Insertions`, `Deletions`.

See `specs/model_diff.go` and `algorithms/diff/parser.go` + `hunk.go`.

## Phase 1 — Deterministic file selection

For **every** changed file, apply gates in order (`algorithms/agent/selection.go`):

1. Binary → exclude
2. Secret path (`.env`, keys, credentials patterns) → exclude — **cannot be overridden by include globs**
3. User exclude globs → exclude
4. User include globs (if any) → force include
5. Extension not in allowlist → exclude
6. Default exclude path patterns (tests, generated, vendor) → exclude
7. Diff token count > prompt limit → exclude as too-large

Output: kept diffs + per-file `ExcludeReason`. Preview mode must use **the same function** as the real run (no drift).

## Phase 2 — Semantic grouping

Input: non-deleted kept diffs.

`GroupingPlan(fileCount, totalChanged)` (`algorithms/template/template.go`):

| Condition | Strategy |
|---|---|
| `fileCount >= GROUPING_MIN_FILES` (or min=0) | Call `GROUPING_TASK` LLM |
| else if `totalChanged < GROUPING_BUNDLE_LINE_THRESHOLD` | One bundle group |
| else | One group per file |

After LLM grouping:

- Parse JSON array of `{label, files: [int indices]}`
- Cover missing indices with single-file groups
- Enforce `maxFilesPerGroup = 10`
- Enforce per-group token budget (split if over)

Prompts: `prompts/grouping_task_*.md`.

## Phase 3 — Concurrent per-group review

Default concurrency = 8. Optional aggregate token budget: before scheduling a group, if `used + estimate(group) > budget`, stop dispatching remaining groups.

### 3a — Rule injection

For paths in the group, resolve checklist via `path_rule_map` (first glob match wins) else `default_rule`.

Map: `specs/system_rules.json`  
Bodies: `rules/rule_docs/*.md`

Inject into plan/main prompts as `{{system_rule}}`.

### 3b — Plan phase (conditional)

`PlanRequired` if:

- any file churn ≥ `PLAN_MODE_LINE_THRESHOLD` (50), OR
- (≥2 files AND total churn ≥ `PLAN_MODE_GROUP_LINE_THRESHOLD` (100))

Plan agent gets tool **descriptions** but must **not** call them — output plain-text structured plan only.

Prompts: `prompts/plan_task_*.md`.

On plan failure: continue without plan (do not abort group).

### 3c — Main task loop (up to `MAX_REVIEW_ROUNDS`)

Build messages from `prompts/main_task_*.md` with placeholders:

| Placeholder | Content | Exact format |
|---|---|---|
| `{{change_files}}` | Other changed files not in this group | `STATUS   path (+N/-M)` per line — see `specs/prompt_serialization.md` |
| `{{diffs}}` | Unified diffs of this group's files | `<file path="...">RAW_DIFF</file>` blocks — see `specs/prompt_serialization.md` |
| `{{system_rule}}` | Matched checklist | Bare text (1 rule) or `<rules for="...">` blocks (multi-rule) |
| `{{plan_guidance}}` | Plan text (empty on round ≥ 2) | Entire `### Review Plan` section stripped when empty |
| `{{confirmed_comments}}` | Findings kept from earlier rounds | `<confirmed_findings>` block; entire section stripped on round 1 |
| `{{requirement_background}}` | Optional user business context | Raw string from `--background` flag |

**Exact serialization formats are in `specs/prompt_serialization.md` — do not guess.**

Run tool-use loop (`algorithms/llmloop/loop.go`):

```
while tool_calls < MAX_TOOL_REQUEST_TIMES (100) and not budget_exceeded:
    response = LLM(messages, MainToolDefs)
    if no tool calls: break  # or nudge once then break
    for each tool_call:
        if code_comment → parse + queue for positioning
        if task_done → mark complete, exit
        else → execute context tool, append tool result
        on consecutive failure of same tool:
            1st fail: return error message
            2nd fail: error + "fix your arguments or call task_done"
            3rd+ fail: silently accept, tell model to move on
    total_tokens = CountMessagesTokens(messages)
    if total_tokens > 60% of MAX_TOKENS → trigger async compression (background)
    if total_tokens > 80% of MAX_TOKENS → trigger sync compression NOW
        → send MEMORY_COMPRESSION_TASK, replace history with summary
```

Token thresholds: see `specs/runtime_thresholds.md`.

**Strict focus rule (in system prompt):** comments may only target files inside `<review_files>`. Context tools may read elsewhere, but comments must not.

Round ≥ 2: strip plan so it cannot act as a coverage ceiling; keep prior confirmed findings visible.

### 3d — Comment positioning pipeline

For each parsed `code_comment` item (`specs/model_review.go`):

```
1. Validate category/severity/path/content/existing_code
2. RelocateAcrossFiles(existing_code, all reviewed diffs)
   - unique hit → rewrite Path + StartLine + EndLine
   - 0 or >1 hits → leave unchanged
3. ResolveComment against target Diff:
   a. matchConsecutive on hunk new-side (context + added)
   b. matchConsecutive on hunk old-side (context + deleted)
   c. sliding window on NewFileContent (skip blank lines)
4. If still unresolved → RE_LOCATION_TASK
   - LLM returns fenced verbatim snippet from diff
   - retry ResolveComment
5. If still unresolved → drop or keep without lines (policy choice;
   OCR prefers not to publish floating comments)
```

Normalization for match: trim, strip leading `+`/`-`, skip empty lines.

Reference: `algorithms/diff/resolver.go`, `relocation.go`, `prompts/re_location_task_*.md`.

### 3e — Review filter (reflection)

Send group diffs + produced comments to `REVIEW_FILTER_TASK`.

Filter task uses its own two tools (NOT in `specs/tools.json` — see `specs/filter_tools.json`):
- `approve_all_comments` — keep everything (default; expected outcome for most files)
- `report_incorrect_comments` — remove only provably-wrong ones

**Field order in `report_incorrect_comments` matters**: `analysis` is declared before `comment_ids` so the model reasons through every comment before committing to an ID list. Reordering breaks this chain-of-thought constraint.

**Asymmetric policy** (`prompts/review_filter_task_user.md`):

- Default = approve everything
- Remove only if:
  - **Ground A**: comment targets code absent from its subject file's diff
  - **Ground B**: a specific diff line literally contradicts the claim
- **Protected subjects** (memory safety, concurrency, linkage, behavioral change, unused params) → never remove
- Style/true-but-low-value → keep (filtering by value is not the filter's job)

## Phase 4 — Aggregate output

Collect `[]LlmComment` across groups. Emit structured result (JSON for hosts/CI).

Optional but valuable for rebuilds: session JSONL for resume + debugging.

## Minimal tool surface (do not expand casually)

From `specs/tools.json` — production-distilled set:

| Tool | Plan | Main | Role |
|---|---|---|---|
| `task_done` | | ✓ | Terminate conversation |
| `code_comment` | | ✓ | Emit findings (via existing_code) |
| `file_read` | | ✓ | Read post-change file ranges |
| `file_read_diff` | ✓ | ✓ | Peek other files' diffs |
| `code_search` | ✓ | ✓ | Grep / regex across repo |
| `file_find` | ✓ | ✓ | Find files by name/path |

Extra tools (generic shell, edit, web) increase noise and attack surface. Add only with production evidence.
