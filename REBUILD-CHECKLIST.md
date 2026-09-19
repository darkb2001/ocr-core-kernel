# Rebuild Checklist

Use this as an implementation backlog. Check off only when behavior matches the invariant.

## Must-have (parity with OCR's effectiveness)

### Data & parse
- [ ] `Diff` model with unified diff + new file content + flags
- [ ] Hunk parser (`@@ -old +new @@`) with line types: context / added / deleted
- [ ] Git range / commit / workspace input resolution

### Deterministic gates
- [ ] `selectFiles` with ordered ExcludeReasons (binary → secret → user → ext → default path → size)
- [ ] Secret paths cannot be force-included
- [ ] Preview uses the **same** selection function as run
- [ ] Extension allowlist + default exclude patterns loaded from config

### Grouping
- [ ] `GroupingPlan` thresholds (min files / bundle line threshold)
- [ ] LLM grouping returns **integer indices**, not paths
- [ ] Fallback to per-file on any grouping error
- [ ] Enforce max 10 files/group + token budget split

### Rules
- [ ] Ordered glob `path_rule_map`, first match wins
- [ ] Inject matched checklist into plan + main prompts
- [ ] Ship language rule docs (start from `rules/rule_docs/`)

### Agent loop
- [ ] Plan phase gated by churn thresholds; failure is non-fatal
- [ ] Main tool loop with hard tool-call cap
- [ ] Round 2+ strips plan text
- [ ] Memory compression when conversation grows past budget
- [ ] Concurrent groups with semaphore + optional aggregate token budget

### Comment quality (the precision engine)
- [ ] `code_comment` requires `existing_code` + path + category + severity
- [ ] Schema repair for common LLM JSON mistakes (see `comment_args_repair.go`)
- [ ] Sliding-window resolve against hunks, then file content
- [ ] Deterministic cross-file refile on unique ExistingCode hit
- [ ] LLM re-location only after deterministic match fails
- [ ] Review filter with Ground A/B + protected-subject veto + approve-default

### Prompts
- [ ] Port all 12 files under `prompts/` (system+user pairs)
- [ ] Preserve placeholder names (`{{diffs}}`, `{existing_code}`, etc.)
- [ ] Keep main_task strict focus rules and filter asymmetry verbatim until you have eval data to change them

## Should-have (operational)

- [ ] Resume / session persistence
- [ ] JSON output for CI / host agents
- [ ] Cost estimate before large runs
- [ ] Per-group timeout
- [ ] Tool failure streak breaker (stop repeating a broken tool)

## Nice-to-have (after core works)

- [ ] Full-file `scan` mode (separate template)
- [ ] Delegation mode (host agent runs review; you only select+rules)
- [ ] MCP dynamic tools
- [ ] Viewer UI / telemetry exporters
- [ ] IDE plugins

## Eval gate before claiming parity

Do **not** claim "as strong as OCR" without measuring:

1. **Precision** — fraction of comments that are real defects
2. **Recall** — fraction of known defects found
3. **Position accuracy** — comments whose lines match intended code
4. **Coverage** — % of reviewable files that received a pass
5. **Tokens / wall time** per review

OCR's published trade-off: higher precision & F1 than general agents, lower recall, ~1/9 tokens. Rebuilds that maximize recall by loosening the filter will regress the product's point.

## Suggested build order

1. Diff parse + hunk model + sliding-window `ResolveComment`
2. Tool schemas + main_task loop with only `code_comment` + `task_done`
3. `file_read` / `code_search` / `file_find` / `file_read_diff`
4. `selectFiles` + allowlists
5. Grouping + concurrency
6. Plan phase
7. Re-location + cross-file refile
8. Review filter
9. Rule map + language docs
10. Budgets, compression, resume
