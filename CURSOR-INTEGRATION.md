# Cursor Integration Guide

For rebuilds targeting Cursor specifically. Two modes; choose one.

---

## Mode A — CLI + Cursor Skill (recommended)

The rebuilt tool ships as a CLI (like `ocr`). Cursor calls it via a Skill.

### What the Cursor Skill does

A Skill is a markdown file that tells Cursor's AI how to invoke your tool. It lives in `~/.cursor/plugins/local/<yourplugin>/skills/<skill-name>/SKILL.md`.

Minimum viable skill for a rebuilt OCR-like CLI:

```markdown
---
name: code-review
description: AI-powered code review on Git diffs
---

## Workflow

### Step 1: Run review

Always pass --audience agent to suppress progress UI:

```bash
your-cli review --audience agent --background "brief context" [flags]
```

Common flags:
- No flags: review staged + unstaged + untracked (workspace mode)
- --from main --to feature-branch: branch range
- --commit abc123: single commit

### Step 2: Report results

Group by severity (critical > high > medium > low). Discard low-severity items.

Format:
- **`path/file.go:42`** [bug/critical] — Issue description
  > Fix: What to change

### Step 3: Fix (only if user requested "review and fix")

Apply high/critical fixes. Describe medium fixes. Skip low.
```

### Output format your CLI must produce

When called with `--audience agent`, your CLI should emit JSON that the Skill can parse:

```json
{
  "comments": [
    {
      "path": "src/main.go",
      "start_line": 42,
      "end_line": 44,
      "content": "Nil pointer dereference: result is used without checking err",
      "category": "bug",
      "severity": "critical",
      "existing_code": "result := compute()\nreturn result.Value",
      "suggestion_code": "result, err := compute()\nif err != nil {\n    return nil, err\n}\nreturn result.Value"
    }
  ],
  "files_reviewed": 5,
  "files_skipped": 1
}
```

`start_line = 0, end_line = 0` means positioning failed — Skill should still show the comment but without line anchor.

### Cursor plugin manifest (`.cursor-plugin/plugin.json`)

```json
{
  "name": "your-code-review",
  "displayName": "Your Code Review",
  "version": "1.0.0",
  "description": "AI-powered code review on Git diffs",
  "skills": "../skills/"
}
```

Directory layout:

```
~/.cursor/plugins/local/your-code-review/
├── .cursor-plugin/
│   └── plugin.json
└── skills/
    ├── code-review/
    │   └── SKILL.md
    └── delegate-review/
        └── SKILL.md        ← optional, see Mode B
```

After copying: Cursor menu → Developer: Reload Window.

---

## Mode B — Delegation Mode (no LLM config needed on tool side)

In this mode, the tool does **only** deterministic work: file selection + rule injection. Cursor's built-in AI does the actual review.

### What the tool exposes

Two sub-commands:

```bash
# Which files to review + mode/ref metadata
your-cli delegate preview [--from A --to B] [--commit hash] [--format json]

# Review rules for specific file paths
your-cli delegate rule --format json path1 path2 ...
```

### Preview JSON output schema

```json
{
  "mode": "range",
  "from": "main",
  "to": "feature-branch",
  "merge_base": "abc123def",
  "reviewable_files": [
    { "path": "src/foo.go", "status": "MODIFIED", "insertions": 24, "deletions": 3 }
  ],
  "excluded_files": [
    { "path": "vendor/lib.go", "reason": "default_path" }
  ]
}
```

### Rule JSON output schema

```json
{
  "groups": [
    {
      "rule": "#### Correctness\nIs the logic correct?...",
      "files": ["src/foo.go", "src/bar.go"]
    },
    {
      "rule": "#### YAML Structure\n...",
      "files": ["config/app.yaml"]
    }
  ]
}
```

### Delegate Skill workflow

The Cursor agent skill for delegation does:

1. `your-cli delegate preview --format json` → get file list + refs
2. `your-cli delegate rule --format json path1 path2...` → get rule per group
3. `git diff <merge_base>..<to> -- <path>` per file
4. Review each file using Cursor's own AI + the injected rule
5. Report findings grouped by severity (same format as Mode A output)

---

## Critical Cursor-specific behaviors

- **Always use `--audience agent`** — the human-mode output streams progress UI that pollutes Cursor's context window
- **Use `--output /tmp/review.json`** for large reviews — prevents truncation in Cursor's tool output
- **Comments with `start_line = 0`** — show them without line anchor; Cursor's agent should read the file and find the relevant location from `existing_code`
- **Never pipe through `head`/`tail`** — drops early comments silently
- **Rule format**: User can place `.opencodereview/rule.json` in the repo root; see `specs/user_rule_format.json` for schema

---

## What "rebuild for Cursor" means concretely

You need:

1. A CLI binary that accepts `--audience agent` and outputs JSON
2. The full pipeline from `PIPELINE.md` implemented internally
3. A Cursor Skill SKILL.md that wraps the CLI call
4. Optionally: delegation sub-commands if users don't want to configure an LLM

You do **not** need: viewer UI, telemetry exporters, MCP server, npm packaging, IDE extension binaries.
