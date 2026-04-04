# AG Code Updates
<!-- Created: 2026-04-04 | Format reference — see REFS/AG-Code-Updates.template.md -->

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
