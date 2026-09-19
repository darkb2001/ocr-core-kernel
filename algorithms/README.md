# Algorithms — Reference Implementations

These are **source extracts** from upstream OCR (Go). They are not a buildable package (imports still point at `github.com/alibaba/open-code-review/...` and peripheral deps).

Read them as the ground-truth algorithms behind the docs.

## Priority order for a rebuilder

1. `diff/resolver.go` — sliding-window line resolution + cross-file refile
2. `diff/hunk.go` + `diff/parser.go` — unified diff structure
3. `diff/relocation.go` — LLM re-location wrapper
4. `agent/selection.go` — deterministic file gates
5. `agent/grouping.go` — semantic groups + hard caps
6. `template/template.go` — `GroupingPlan` / `PlanRequired` / token limits
7. `tool/code_comment.go` + `comment_args_repair.go` + `comment_collector.go`
8. `tool/definitions.go` — reserved tool names
9. `llmloop/loop.go` — main tool-use loop (large; skim for control flow)
10. `llmloop/compression.go` — memory compression trigger
11. `llmloop/tool_failure_streak.go` — stop hammering a broken tool
12. `rules/system_rules.go` — glob resolve + ordered path rules

## When porting

- Prefer **behavioral fidelity** over line-by-line translation.
- Keep function contracts (`ResolveComment` returns bool; unique-hit refile only).
- Stub or delete telemetry/session calls if they obscure the algorithm.
