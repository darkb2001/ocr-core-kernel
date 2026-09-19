# OCR Core Kernel

Distilled core of [alibaba/open-code-review](https://github.com/alibaba/open-code-review) for rebuilding a similar AI code-review tool.

**This folder is not runnable.** It is a deliberate extract of the parts that make OCR effective — so another AI (or engineer) can read it end-to-end and rebuild an equivalent system.

## How to use this folder

Read in this order:

1. **[ARCHITECTURE.md](ARCHITECTURE.md)** — philosophy + component map
2. **[PIPELINE.md](PIPELINE.md)** — exact runtime sequence for one review
3. **[REBUILD-CHECKLIST.md](REBUILD-CHECKLIST.md)** — what you must implement
4. **`prompts/`** — every prompt template (highest leverage content)
5. **`specs/tools.json`** — main tool schemas (6 tools)
6. **`specs/filter_tools.json`** — filter-phase tool schemas (2 tools, separate)
7. **`specs/task_template.json`** — task wiring + numeric thresholds
8. **`specs/runtime_thresholds.md`** — ALL numeric policies in one place
9. **`specs/prompt_serialization.md`** — EXACT format of every `{{placeholder}}`
10. **`specs/system_rules.json`** + **`rules/rule_docs/`** — language checklists
11. **`specs/user_rule_format.json`** — user-side rule.json schema
12. **`algorithms/`** — reference Go implementations of hard constraints
13. **`design/`** — why each hard constraint exists
14. **[CURSOR-INTEGRATION.md](CURSOR-INTEGRATION.md)** — Cursor Skill + delegate mode

## What was kept vs dropped

| Kept (core) | Dropped (peripheral) |
|---|---|
| Diff parse + file selection | CLI / TUI / npm packaging |
| Semantic file grouping | Session viewer (HTTP UI) |
| Plan → main tool-loop → filter | OpenTelemetry wiring |
| Comment positioning (sliding window) | MCP server extensions |
| Re-location + cross-file refile | IDE plugins (Cursor/Claude/…) |
| Review-filter reflection | CI examples, docs site, release |
| Tool schemas + prompts | Full LLM provider clients |
| Path→rule matching + rule docs | Scan mode (full-file audit) |
| Token/group budgets | Delegation mode |

## Source provenance

Extracted from `open-code-review` (Apache-2.0). Algorithm files retain original copyright headers. Rebuilds should keep license attribution.
