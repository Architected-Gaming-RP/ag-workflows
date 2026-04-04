# ag-workflows

Org-level shared workflows for Architected Gaming RP.

## Reusable Workflows

### `lint-reusable.yml`

Lua linter for FiveM resources using [iLLeniumStudios/fivem-lua-lint-action@v2](https://github.com/iLLeniumStudios/fivem-lua-lint-action).

**Features:**
- Lints all `.lua` files with QBX globals (`ox_lib`, `mysql`, `qblocales`, `qbox`, `qbox_playerdata`, `qbox_lib`)
- Generates JUnit report with inline annotations on the PR/commit
- Posts results to Discord via `DISCORD_LINT_WEBHOOK` repo secret (rich embed with error details)
- Fails on lint errors (`fail_on_failure: true`)

**Usage — add this as `.github/workflows/lint.yml` in your repo:**

```yaml
name: Lint
on:
  push:
    paths: ['**.lua', '.github/workflows/lint.yml']
  pull_request_target:
    paths: ['**.lua', '.github/workflows/lint.yml']
permissions:
  contents: read
  checks: write
jobs:
  lint:
    uses: Architected-Gaming-RP/ag-workflows/.github/workflows/lint-reusable.yml@main
    secrets: inherit
```

**Discord notifications:** Set the `DISCORD_LINT_WEBHOOK` secret on each repo (or skip it — the workflow handles missing secrets gracefully).

**Changing to failure-only Discord notifications:** Edit `lint-reusable.yml` and change the Discord step's `if: always()` to `if: failure()`.
