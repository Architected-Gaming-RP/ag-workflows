# AG Code Updates
<!-- Created: 2026-04-04 | Format reference — see REFS/AG-Code-Updates.template.md -->

================================================================================
AG UPDATE - 04APR26 - Fix: JUnit fallback parsing + errors webhook job_status gate
================================================================================

## 04APR26: Fix Malformed JUnit XML Handling + Errors Webhook Gate

**Problem Report:** On a syntax error (`config/config.lua:7:26: expected '=' near 'CODE'`), the all-messages webhook showed a bare "Failed / 0 passed" with no error details. The errors-only webhook didn't fire at all.

**Investigation:** GH Actions log showed `##[error] Failed to parse file (junit.xml) with error Error: Text data outside of root node.` [tool: GH Actions log]. Luacheck's JUnit formatter produces invalid XML on syntax errors. The Python `ET.parse()` hit `ET.ParseError`, the `except` block silently swallowed it [read: lint-reusable.yml:112], leaving `failed=0` — so no error details were collected and the errors webhook gate (`if failed > 0`) blocked it.

**Root Cause:** Two issues: (1) No fallback when JUnit XML is malformed — errors silently lost. (2) Errors webhook gated solely on JUnit `failed` count, not on actual job failure status.

**What Changed:**
- `lint-reusable.yml:112-124` — Added raw-text fallback in the `except` block: reads `junit.xml` as plain text, extracts lines matching `.lua:` + error patterns, populates `errors[]` and increments `failed`.
- `lint-reusable.yml:176` — Errors webhook gate changed from `if failed > 0` to `if failed > 0 or job_status == "failure"` so it fires even if JUnit parsing completely fails.

---

================================================================================
AG UPDATE - 04APR26 - Lint webhook: main WH fires on all runs, errors WH on failures only
================================================================================

## 04APR26: Split Lint Discord Webhook Behavior

**Problem Report:** Both `DISCORD_LINT_WEBHOOK` and `DISCORD_LINT_ERRORS_WEBHOOK` were gated behind `if failed > 0`, meaning successful lint runs produced no Discord notification at all. User wanted one webhook to report all results (pass + fail) for visibility.

**Investigation:** Reading `lint-reusable.yml:155-172` confirmed both webhooks shared a single `if failed > 0` gate with a loop over both URLs [read: lint-reusable.yml:156-158].

**Root Cause:** Original implementation treated both webhooks identically — both errors-only. No webhook covered passing runs.

**What Changed:**
- `lint-reusable.yml:155-175` — Extracted a `send()` helper function. `DISCORD_LINT_WEBHOOK` now fires unconditionally (every run). `DISCORD_LINT_ERRORS_WEBHOOK` remains gated behind `if failed > 0`.

---
<!-- Prepend new updates above this line -->
