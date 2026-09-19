# Prompt Serialization — Exact Formats

These are the runtime-computed strings that fill each `{{placeholder}}` in the prompt templates. Getting these formats wrong silently degrades review quality (wrong XML structure, model misreads the diff block, etc.).

---

## `{{diffs}}` — unified diff block for the current group

Filled by `buildConcatenatedDiffs(group.Diffs)`:

```
<file path="path/to/file1.go">
--- a/path/to/file1.go
+++ b/path/to/file1.go
@@ -10,5 +10,6 @@
 unchanged line
+added line
-removed line
 unchanged line
</file>

<file path="path/to/file2.go">
...raw unified diff text...
</file>
```

Rules:
- Each file wrapped in `<file path="...">` / `</file>`
- The diff text is raw unified diff from git, no further wrapping
- Files separated by a blank line (`\n\n`) between them
- No outer container tag — the user prompt template wraps it in `<review_files>{{diffs}}</review_files>`

---

## `{{change_files}}` — other files changed but NOT in this group

Filled by `buildChangeFilesExceptGroup(group.Diffs)`:

```
MODIFIED   src/main/java/Foo.java (+24/-3)
ADDED   src/main/java/Bar.java (+81/-0)
DELETED   src/old/Legacy.java (+0/-45)
RENAMED   src/pkg/OldName.go (+2/-2)
```

Rules:
- One line per file: `STATUS   path (+insertions/-deletions)`
- Status is one of: `MODIFIED`, `ADDED`, `DELETED`, `RENAMED`
- Three spaces between status and path
- Binary files excluded entirely
- Files in the current group excluded entirely
- Lines joined with `\n` (no trailing newline)

Same format used in the grouping file list (with `[index] ` prefix prepended by `buildFileList`):

```
[0] MODIFIED   src/main.go (+12/-3)
[1] ADDED   src/handler.go (+55/-0)
```

---

## `{{confirmed_comments}}` — findings from prior review rounds

Filled by `buildConfirmedCommentsBlock(comments)`:

When empty (round 1): entire `### Previously Confirmed Findings` section is stripped from the user message by regex: `^### [^\n]*Confirmed Findings[^\n]*\n\{\{confirmed_comments\}\}\n\n?`

When non-empty:

```
The following issues were already identified and confirmed in a prior review pass. Do not repeat them. Continue reviewing all files in <review_files> and report any other real issues you find.

<confirmed_findings>
1. path/to/file.go
   code: functionName(x, y) → returns wrong value when...  (truncated to 200 runes if longer)
   issue: Possible nil dereference when result is...  (truncated to 300 runes if longer)
2. path/to/other.go
   code: ...
   issue: ...
</confirmed_findings>
```

Rules:
- Max 30 entries (cap is `confirmedCap = 30`)
- Multi-line `ExistingCode` and `Content` flattened to single lines (newlines → spaces)
- ExistingCode truncated to 200 runes with `...`
- Content truncated to 300 runes with `...`

When `planResult == ""` (round ≥ 2 or plan skipped), the `### Review Plan` block is also stripped:
Pattern: `^### [^\n]*Review Plan[^\n]*\n\{\{plan_guidance\}\}\n\n?`

---

## `{{system_rule}}` — review checklist for the group

Single-rule group (all files match same language): inject rule body directly (bare text, no wrapper).

Multi-rule group (e.g. Go + YAML in the same group):

```
<rules for="path/to/file.go, path/to/other.go">
#### Correctness
Is the logic correct? ...

#### Security
...
</rules>
<rules for="path/to/config.yaml">
#### YAML Structure
...
</rules>
```

Rules:
- Paths sorted alphabetically (for prompt cache affinity)
- One `<rules for="...">` block per distinct rule body
- Paths comma-separated inside the `for` attribute
- Bare text (no wrapper) when only one distinct rule

---

## `{{plan_tools}}` — tool descriptions for plan phase

The plan prompt gets tool **descriptions** (not JSON schemas) to tell the agent which tools it *could* use, without letting it call them. Format: name + description string, assembled from the same `tools.json` entries that are enabled for `plan_task: true`.

Each enabled tool's description is injected. Exact assembly: concatenate `name: description\n` pairs, or pass the tool defs array as-is to the LLM (provider handles rendering). The key constraint is plan tools are **read-only context tools only** — `code_comment` and `task_done` are not included in plan-phase tool defs.

---

## `{{current_system_date_time}}` — timestamp

Format: `"2006-01-02 15:04"` (Go time format), local server time at run start.

---

## Filter task: comment IDs format

Comments are serialized into `{{comments}}` for the filter task as a numbered list with IDs:

```
c-0. [high][bug] path/to/file.go:42
  Existing code: `return nil, err`
  Comment: Error is silently swallowed — caller gets nil without knowing why

c-1. [medium][performance] path/to/file.go:88
  Existing code: `for _, v := range items {`
  Comment: N+1 query pattern inside loop
```

Comment ID format: `c-{index}` (zero-based). The filter response refers to IDs in `comment_ids` array.
