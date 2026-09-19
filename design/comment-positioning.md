# Comment Positioning Design

## Problem

LLMs are bad at absolute line numbers. They are decent at quoting the code they mean.

## Contract

`code_comment` tool parameters (see `specs/tools.json`):

- `path` — relative file path
- `existing_code` — consecutive lines of **newly added** code as they appear in the change (no deleted/context-only lines preferred)
- `content`, `category`, `severity`, optional `suggestion_code`

The host resolves `StartLine` / `EndLine` after the tool returns.

## Resolution order (do not reorder casually)

```
code_comment args
      │
      ▼
 schema parse (+ optional JSON repair)
      │
      ▼
 RelocateAcrossFiles(existing_code, all_reviewed_diffs)
      │ unique hit? rewrite path+lines
      ▼
 ResolveComment(path's Diff):
      1. hunk new-side sliding window
      2. hunk old-side sliding window
      3. NewFileContent sliding window
      │
      ▼ fail?
 RE_LOCATION_TASK (LLM)
      │ returns fenced verbatim snippet
      ▼
 ResolveComment again
      │
      ▼ still fail?
 drop / quarantine (do not publish with invented lines)
```

## Why re-location is a last resort

`prompts/re_location_task_user.md` forces:

1. Copy lines **verbatim** from the diff
2. Strip diff markers
3. Minimal contiguous range only
4. Output **only** a fenced code block

Even then, the snippet is re-validated by `ResolveComment`. The LLM never writes `StartLine` directly.

## Implementation pointers

| File | Role |
|---|---|
| `algorithms/diff/hunk.go` | Parse hunk headers and line kinds |
| `algorithms/diff/resolver.go` | `ResolveComment`, `RelocateAcrossFiles`, normalize/match |
| `algorithms/diff/relocation.go` | Build re-location messages + apply LLM snippet |
| `algorithms/tool/code_comment.go` | Parse/validate tool args |
| `algorithms/tool/comment_args_repair.go` | Deterministic repair of mangled JSON string args |
| `algorithms/tool/comment_collector.go` | Thread-safe collection per path |

## Testing ideas for rebuilds

1. Snippet matches added line → correct new-file line
2. Snippet has leading `+` → still matches after normalize
3. Blank lines inside snippet → ignored in window
4. Snippet unique in sibling file → path rewritten
5. Snippet in two files → no rewrite
6. Wrong path + unique snippet elsewhere → refile before re-location
7. Unresolvable after re-location → not published with fake lines
