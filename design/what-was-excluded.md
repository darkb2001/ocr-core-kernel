# What Was Excluded (and why)

This kernel deliberately omits large parts of the upstream repo. Knowing what is missing prevents rebuilders from thinking they are required for review quality.

## Excluded packages

| Upstream | Why excluded from kernel |
|---|---|
| `cmd/opencodereview` | CLI surface — rebuild your own interface |
| `internal/viewer` | Browser session UI — UX, not review quality |
| `internal/telemetry` | OTel exporters — observability, optional |
| `internal/mcp` | Dynamic external tools — extension point |
| `internal/delegate` | Host-agent mode — useful product feature, not core loop |
| `internal/scan` | Full-file audit mode — parallel product; reuse same loop/tools |
| `internal/session` (full) | Resume/manifest complexity — valuable later, not first |
| `internal/llm/*` clients | Provider SDKs — swap for your stack |
| `internal/release`, `npm/`, `bin/` | Distribution |
| `plugins/`, `extensions/`, `skills/` | IDE integrations |
| `examples/`, `pages/`, `docs/` | Docs & CI samples |
| `*_test.go` | Tests are huge; behavior is described in docs + algorithms |

## Still useful later

Copy from upstream when you need them:

1. **Session resume** (`internal/session`) — large PR interruption recovery
2. **Scan mode** (`internal/scan` + `scan_template.json`) — audit without meaningful diffs
3. **GitHub Action** (`action.yml` + `scripts/github-actions/`) — CI posting
4. **Delegate mode** — zero API key when Cursor/Claude already has one

## Coupling warning

Algorithm `.go` files still import upstream module paths (`github.com/alibaba/open-code-review/...`). They are **reference implementations**, not a buildable module. When rebuilding:

- Re-implement in your language/module, OR
- Rewrite imports and stub peripherals (`telemetry` → no-op, `session` → minimal, etc.)

Prefer reading for algorithms + copying prompts/specs/rules verbatim.
