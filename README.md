# TBD Methodology Checks GitHub Action

This repo provides a reusable **GitHub Action** that validates PRs comply with Trunk-based Development Process by:

- **Validating cherry-pick branch names** for PRs targeting release branches
- **Enforcing branch name format** (configurable via regex)
- **Validating commit message format** (configurable via regex)
- **Automatically closes PRs** that don't match the required cherry-pick format
- **Posts helpful comments** explaining the required formats

The action is implemented as a composite action using `actions/github-script@v7`.

## Usage

Add this to your repository workflow to validate branch names and commit messages:

```yaml
name: Validate PR
on:
  pull_request:
    branches: [ main, release/* ]

permissions:
  contents: read
  pull-requests: write

jobs:
  methodology-checks:
    runs-on: ubuntu-latest
    steps:
      - name: TBD Methodology Checks
        uses: ensembleip/tbd-methodology-checks@v1
```

## Versioning / pinning

`uses: ensembleip/tbd-methodology-checks@vx` requires that this repository has a **Git tag** named `vx` (or a branch named `vx`).

- For initial testing you can use `@main`.
- For production usage, prefer pinning to a commit SHA, or use a major tag like `@v1` that you keep updated to the latest `v1.x.y`.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `cherry-pick-branch-regex` | Regular expression pattern for validating cherry-pick branch names | No | `^bug/QD-\\d+-.+-cherry-pick$` |
| `example-cherry-pick-branch` | Example branch name to display in error messages | No | `bug/QD-1234-bug-description-cherry-pick` |
| `branch-pattern` | Regular expression pattern for validating branch name format | No | `^(bug\|story\|task\|spike)/QD-\\d+-[a-z0-9-]+` |
| `commit-pattern` | Regular expression pattern for validating commit message format | No | `^\\[QD-\\d+\\] .+` |

## How It Works

The action performs three validation steps:

### 1. Check PR Branches (for PRs targeting release branches)

- **Checks if the PR targets a release branch** (branches starting with `release/`)
- **Validates the head branch** against the configured cherry-pick regex pattern
- If validation **fails**:
  - Creates a PR comment explaining the required format
  - Automatically closes the PR
  - Fails the GitHub Action check
- If validation **passes** or the PR doesn't target a release branch:
  - The check passes silently

### 2. Validate Branch Name Format

- **Validates the branch name** against the configured branch-pattern
- Default pattern enforces: `(bug|story|task|spike)/QD-<number>-<description>`
- Fails the check if the branch name doesn't match the pattern

### 3. Validate Commit Message Format

- **Validates all commit messages** in the PR against the configured commit-pattern
- Default pattern enforces: `[QD-<number>] <message>`
- Fails the check if any commit message doesn't match the pattern

## Customization

You can customize any of the validation patterns:

### Custom Cherry-Pick Branch Regex

Customize the cherry-pick branch naming pattern for release branches:

```yaml
- name: TBD Methodology Checks
  uses: ensembleip/tbd-methodology-checks@v1
  with:
    cherry-pick-branch-regex: "^hotfix/.*-cp$"
    example-cherry-pick-branch: "hotfix/my-fix-cp"
```

### Custom Branch Pattern

Customize the general branch naming pattern:

```yaml
- name: TBD Methodology Checks
  uses: ensembleip/tbd-methodology-checks@v1
  with:
    branch-pattern: "^(feature|fix)/[A-Z]+-\\d+-.*$"
```

### Custom Commit Pattern

Customize the commit message format:

```yaml
- name: TBD Methodology Checks
  uses: ensembleip/tbd-methodology-checks@v1
  with:
    commit-pattern: "^(feat|fix|docs|chore):\\s.+"
```

## Default Behavior

By default, the action:
- **For PRs targeting release branches**: expects cherry-pick branch names matching `bug/QD-<number>-<description>-cherry-pick`
- **For all PRs**: expects branch names matching `(bug|story|task|spike)/QD-<number>-<description>` (all lowercase letters, numbers, and hyphens)
- **For all PRs**: expects commit messages matching `[QD-<number>] <message>`

## Example PR Comment

When validation fails, the action posts a comment like this and closes the PR:

```markdown
This PR targets `release/v1.2.3`, so **only cherry-pick PRs are allowed** for release branches.

Your head branch is `bug/my-branch`, which does not match the required format:

- `bug/QD-1234-bug-description-cherry-pick`

Please rename your branch to the correct format and open a new PR. Closing this PR now.
```

## Permissions

The workflow using this action requires the following permissions:

- `contents: read` - to read repository contents
- `pull-requests: write` - to post comments and close pull requests

The action automatically uses `github.token` provided by GitHub Actions, so no token input is required.

## Notes

- The action only runs on pull request events. It will skip gracefully for other event types.
- PRs targeting branches starting with `release/` must use cherry-pick branch names and are automatically closed if they don't match.
- All PRs are validated for branch name format and commit message format.
- The action uses the GitHub REST API to manage PRs efficiently.