# Specs

| File | Purpose |
|---|---|
| `tools.json` | Full tool definitions + which phase enables each tool |
| `task_template.json` | Task wiring + numeric budgets/thresholds |
| `system_rules.json` | Glob → rule_doc map (first match wins) |
| `model_diff.go` | `Diff` struct |
| `model_review.go` | `LlmComment` / result structs |

Load `tools.json` and `task_template.json` as your runtime config. Do not hardcode tool schemas in prompts — the model receives JSON Schema from the host API.
