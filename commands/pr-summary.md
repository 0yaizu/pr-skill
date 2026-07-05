---
name: pr-summary
argument-hint: "[base-branch]"
description: |
  Generate a PR description from the current branch diff and optionally create a draft PR.

  Usage:
    /pr-summary                  Auto-detect base branch (main → develop → master)
    /pr-summary <base-branch>    Use specified branch as base (e.g. /pr-summary develop)

  What it does:
    1. Diffs current branch against base branch
    2. Reads .github/pull_request_template.md if present
    3. Generates PR title + body from commits and diff
    4. Asks whether to create a GitHub draft PR via `gh`
allowed-tools: [Read, Bash, Glob]
---

# PR Summary

Generate a pull request description and optionally create a draft PR.

## Arguments

`$ARGS` — optional base branch name. If omitted, auto-detect from `main` / `develop` / `master`.

**Examples:**
- `/pr-summary` — auto-detect base branch
- `/pr-summary develop` — use `develop` as base
- `/pr-summary release/1.0` — use `release/1.0` as base

## Steps

### 1. Determine base branch

If `$ARGS` is provided, use it as the base branch.

Otherwise, check in order and use the first that exists:
```bash
git show-ref --verify --quiet refs/heads/main && echo main
git show-ref --verify --quiet refs/heads/develop && echo develop
git show-ref --verify --quiet refs/heads/master && echo master
```

Also try the remote default:
```bash
git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}'
```

### 2. Get current branch

```bash
git branch --show-current
```

If on base branch already, abort with: `Already on base branch. Checkout a feature branch first.`

### 3. Gather diff information

```bash
git log <base>..<current> --oneline
git diff <base>..<current> --stat
git diff <base>..<current>
```

Also scan commit messages for issue references (`#123`, `closes #123`, `fixes #123`).

### 4. Find PR template

Check these paths in order and use the first that exists:
- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE/default.md`
- `.github/pr-template.md`
- `.github/pr-template`

### 5. Generate PR content

Derive a concise title from the branch name and commits (strip prefixes like `feature/`, `fix/`, `chore/`).

**If template found:**
- Read the template
- Fill every section with content from the actual commits and diffs
- Preserve all checkboxes (`- [ ]`) and section headers exactly
- Do not add or remove sections

**If no template:**
Use this format:
```markdown
## Summary

- <bullet points describing what changed>

## Changes

<technical description of modified files and what was changed>

## Related Issues

<issue links from commits, e.g., Closes #123 — or "None" if none found>

## Notes for Reviewers

<context that helps reviewers understand the change>
```

### 6. Display the generated PR

Show the title and body clearly to the user.

### 7. Ask user

Ask the user: "Create a draft PR with this content? (yes/no)"

### 8. Create or exit

**If user says yes:**
```bash
gh pr create --draft --title "<title>" --body "<body>" --base <base>
```

Print the PR URL returned by the command.

**If user says no:**
Print: `Aborted. PR description above is ready to copy-paste.`
Exit without creating anything.
