# Hard Constraints — What Must Stay in Code

These steps are **not** prompt suggestions. If you move them into natural language, you recreate the failure modes OCR was built to avoid.

## 1. File selection (`algorithms/agent/selection.go`)

**Invariant:** The set of files under review is computed before any LLM call that reviews code.

Why: Models skip files under context pressure. Selection must be exhaustive and explainable (`ExcludeBinary`, `ExcludeSecret`, …).

## 2. Group size & token split (`algorithms/agent/grouping.go`)

**Invariant:** No review conversation may silently swallow unbounded files/tokens.

Even when the grouping LLM clusters poorly, `enforceMaxFilesPerGroup` and `enforceGroupTokenBudget` still split. Missing files get singleton groups — coverage over cleverness.

## 3. Comment location via `existing_code` (`algorithms/diff/resolver.go`)

**Invariant:** The model never supplies authoritative line numbers. It supplies a verbatim snippet; code maps snippet → lines.

Why: Line drift is the #1 credibility killer for AI review. Sliding window on normalized hunk/file lines is cheap and deterministic.

Algorithm sketch:

```
normalize(line) = trim; strip leading +/-; trim again; drop if empty
targets = normalize each line of existing_code
scan consecutive windows in hunk new-side, then old-side, then full file
on hit → StartLine/EndLine = absolute file lines of window ends
```

## 4. Cross-file refile before LLM re-location (`RelocateAcrossFiles`)

**Invariant:** If `existing_code` uniquely appears in another reviewed file, move the comment there in code — do not ask the model.

Why: Asking the model to "fix path" while showing the **wrong** file's diff causes it to overwrite the only good evidence (`existing_code`) with a lookalike from the wrong file.

Unique hit only. Ambiguous → do nothing.

## 5. Review filter asymmetry (`prompts/review_filter_task_*.md`)

**Invariant:** Filter may only remove comments the **diff itself proves wrong**. Default is keep.

Why: Dropping a real security/concurrency finding is silent data loss. Keeping a mediocre style comment costs seconds.

Protected subjects (memory safety, concurrency, linkage, behavioral change, unused params) are vetoes — never remove even if you think they are wrong.

## 6. Plan is not a coverage ceiling

**Invariant:** After round 1, strip plan text from the main prompt.

Why: Plans list risk points; models then only check those points. Round 2+ forces a fresh pass over `<review_files>`.

## 7. Strict comment scope

**Invariant:** Comments may only target paths inside the current group's `<review_files>`.

Context tools can read the rest of the repo; filing comments on them creates unowned findings and breaks coverage accounting.
