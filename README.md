# .github

Shared GitHub Actions workflows for the homelab repositories. Callers pin `@v1`, never `main`. `v1` is an annotated tag moved forward by hand to the new `main` commit after each merged change, so a breaking change needs a new major tag instead.

## pr-title.yml

Checks the pull request title, which is the squash-merge commit subject, against conventional commits: `type(scope)!: subject` with scope and `!` optional. It fails, naming every rule broken, when the type is not an allowed lower-case type, the scope is empty or not lower-case, the subject is empty, the title ends with a full stop, contains an em dash, or is longer than `max-length`.

| Input | Default | Meaning |
| --- | --- | --- |
| `runs-on` | `'"ubuntu-latest"'` | Runner as JSON: a label or a list of labels |
| `types` | `feat fix perf refactor docs style test build ci chore revert` | Allowed types, space-separated |
| `max-length` | `100` | Maximum title length in characters |

## gitleaks.yml

Scans the commits the pull request adds to its base with gitleaks 8.30.1, downloaded from the release and verified against its published SHA-256. Findings are redacted and any finding fails the job. A `.gitleaks.toml` at the caller's repository root is picked up automatically. It needs `contents: read`.

| Input | Default | Meaning |
| --- | --- | --- |
| `runs-on` | `'"ubuntu-latest"'` | Runner as JSON: a label or a list of labels |

## Calling them

`edited` is what reruns the title check when only the title changes.

```yaml
name: PR checks

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  pr-title:
    uses: iamrobertyoung/.github/.github/workflows/pr-title.yml@v1
    with:
      runs-on: '["self-hosted", "homelab"]'

  gitleaks:
    uses: iamrobertyoung/.github/.github/workflows/gitleaks.yml@v1
    with:
      runs-on: '["self-hosted", "homelab"]'
```

Drop the `with` blocks to run on GitHub-hosted runners. Public repositories must not use self-hosted runners.

## Dependabot titles

Set a `commit-message` prefix on every `updates` entry so Dependabot's titles, such as `chore(deps): bump x from 1 to 2`, pass the title check:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    commit-message:
      prefix: chore(deps)
```
