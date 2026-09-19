# Prompt System Map

All prompts live in `prompts/`. Wiring is declared in `specs/task_template.json`.

## Task graph

```
GROUPING_TASK ──(optional)──► FileGroup[]
        │
        ▼
PLAN_TASK ──(optional, churn-gated)──► plan text
        │
        ▼
MAIN_TASK ◄──► tools (loop)
   │    │
   │    └── MEMORY_COMPRESSION_TASK (when history too long)
   │
   ▼ code_comment fails locate?
RE_LOCATION_TASK
   │
   ▼ comments collected
REVIEW_FILTER_TASK ──► keep / drop
```

## File pairs

| Task | System | User | Placeholders |
|---|---|---|---|
| Grouping | `grouping_task_system.md` | `grouping_task_user.md` | `{{file_list}}` |
| Plan | `plan_task_system.md` | `plan_task_user.md` | `{{plan_tools}}`, `{{change_files}}`, `{{diffs}}`, `{{system_rule}}`, `{{requirement_background}}`, `{{current_system_date_time}}` |
| Main | `main_task_system.md` | `main_task_user.md` | `{{change_files}}`, `{{diffs}}`, `{{system_rule}}`, `{{plan_guidance}}`, `{{confirmed_comments}}`, `{{requirement_background}}`, `{{current_system_date_time}}` |
| Memory | `memory_compression_task_system.md` | `memory_compression_task_user.md` | conversation body in user |
| Re-location | `re_location_task_system.md` | `re_location_task_user.md` | `{diff}`, `{existing_code}`, `{suggestion_content}` |
| Filter | `review_filter_task_system.md` | `review_filter_task_user.md` | `{{diff}}`, `{{comments}}` |

Note: re-location uses `{single_braces}`; others use `{{double}}`. Preserve both when porting.

## Non-negotiable prompt policies

Copy these policies even if you rewrite wording:

1. **Main — focus on added code; avoid deleted / unchanged / comments-about-comments**
2. **Main — every file in `<review_files>` gets its own pass before `task_done`**
3. **Main — never comment outside `<review_files>`**
4. **Plan — describe tool intent on `→` lines; do not actually call tools**
5. **Filter — approve by default; remove only Ground A/B; protected subjects veto**
6. **Grouping — refer to files by integer index; output JSON array only**
7. **Re-location — verbatim snippet only, fenced code block only**

## Customization guidance

Safe to customize early:

- Language of output (inject "respond in Vietnamese" into main system)
- Severity taxonomy labels (keep enums in sync with `tools.json`)
- Extra checklist bullets in `rules/rule_docs/*`

Dangerous to customize without eval:

- Softening filter Ground A/B
- Letting main invent line numbers
- Expanding toolset with shell/edit
- Removing round-2 plan stripping
