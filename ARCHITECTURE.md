# Architecture: Deterministic Engineering × Agent Hybrid

## The core thesis

A general-purpose coding agent + a markdown prompt is **not** enough for reliable PR review. Three failure modes dominate:

1. **Incomplete coverage** — agent skips files on large changesets
2. **Position drift** — comments land on wrong lines / wrong files
3. **Unstable quality** — small prompt changes swing precision wildly

OCR's answer: **never ask the LLM to do the parts that must not go wrong.** Put hard constraints in code; leave the LLM only the parts that need judgment.

```
┌─────────────────────────────────────────────────────────────┐
│ DETERMINISTIC (must not fail)                               │
│  • git diff parse                                           │
│  • file selection (binary/secret/ext/size gates)            │
│  • group size / token budget enforcement                    │
│  • path → review-rule matching (glob, first-match-wins)     │
│  • comment line resolution (sliding-window string match)    │
│  • cross-file comment refile (unique ExistingCode hit)      │
│  • schema validation on tool args                           │
└───────────────────────────┬─────────────────────────────────┘
                            │ feeds constrained context into
┌───────────────────────────▼─────────────────────────────────┐
│ AGENT (judgment + dynamic context)                          │
│  • GROUPING_TASK  — semantic clusters (optional)            │
│  • PLAN_TASK      — risk plan + intended tool calls         │
│  • MAIN_TASK      — tool loop: search/read → code_comment   │
│  • RE_LOCATION    — regenerate ExistingCode when match fails│
│  • REVIEW_FILTER  — remove only factually-wrong comments    │
│  • MEMORY_COMPRESS— summarize long tool conversations       │
└─────────────────────────────────────────────────────────────┘
```

## Component map

```
git range / commit / workspace
        │
        ▼
   [diff.parse] ──────────────► model.Diff[]
        │
        ▼
   [selectFiles] ─────────────► kept diffs + ExcludeReason[]
        │                         (algorithms/agent/selection.go)
        ▼
   [groupDiffs] ──────────────► FileGroup[]
        │                         GroupingPlan → bundle|per-file|LLM
        │                         (algorithms/agent/grouping.go)
        ▼
   per group (concurrent, semaphore):
        │
        ├─ resolve system_rule for paths in group
        │     (specs/system_rules.json + rules/rule_docs/*)
        │
        ├─ [PLAN_TASK] if PlanRequired(churn)
        │     output: structured risk plan (plain text, not tools)
        │
        ├─ [MAIN_TASK] × MaxReviewRounds
        │     tool loop until task_done or budget
        │     tools: code_comment | file_read | file_read_diff
        │            | code_search | file_find | task_done
        │     (specs/tools.json, algorithms/llmloop/)
        │
        ├─ for each code_comment:
        │     parse → RelocateAcrossFiles → ResolveComment
        │     → if fail: RE_LOCATION_TASK → ResolveComment again
        │     (algorithms/diff/resolver.go, relocation.go)
        │
        └─ [REVIEW_FILTER_TASK]
              approve by default; remove only Ground A/B
              (prompts/review_filter_task_*.md)
        │
        ▼
   LlmComment[]  (path, start/end line, category, severity, content)
```

## Why this beats "one .md prompt"

| Concern | One prompt + agent | This architecture |
|---|---|---|
| Coverage | Soft request in prose | `selectFiles` + coverage seal + per-group dispatch |
| Line accuracy | Hope the model cites correctly | `existing_code` → sliding window → optional re-location |
| Large PR | One giant context | Groups ≤10 files, token budget split, concurrent sub-agents |
| Noise | Model free to invent | Filter only removes *proven-wrong* comments; protected subjects veto removal |
| Rule focus | Dump all rules always | Glob map injects only the matching language checklist |

## Critical invariants (do not weaken)

1. **LLM never chooses which files to review** — only which semantic groups, and only after selection.
2. **Comments must carry `existing_code`**, not free-form line numbers from the model.
3. **Line numbers are computed by code**, never trusted from the model.
4. **Filter is asymmetric** — false negative (dropping a real bug) is worse than false positive; default is approve.
5. **Plan is guidance, not a coverage ceiling** — round 2+ strips the plan so the agent re-scans.
6. **Cross-file refile is deterministic and unique-hit only** — never guess among multiple matches.

## Tunable thresholds (from `specs/task_template.json`)

| Key | Default | Meaning |
|---|---|---|
| `MAX_TOKENS` | 200000 | Prompt context budget |
| `MAX_COMPLETION_TOKENS` | 16384 | Output cap |
| `MAX_TOOL_REQUEST_TIMES` | 100 | Tool calls per main-loop conversation |
| `PLAN_MODE_LINE_THRESHOLD` | 50 | Plan if any single file churn ≥ this |
| `PLAN_MODE_GROUP_LINE_THRESHOLD` | 100 | Plan if group total churn ≥ this (≥2 files) |
| `GROUPING_MIN_FILES` | 4 | Below this, skip LLM grouping |
| `GROUPING_BUNDLE_LINE_THRESHOLD` | 200 | Below this (and few files), bundle all into one group |
| `MAX_REVIEW_ROUNDS` | 2 | Main-task passes per group |
| `maxFilesPerGroup` | 10 | Hard cap in grouping.go |
