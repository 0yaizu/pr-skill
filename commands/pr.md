---
name: pr
description: Generate a pull request description for the current branch
allowed-tools: [Read, Bash, Glob]
---

# PR Description Generator

Generate a pull request description based on the current branch's changes.

## Steps

### 1. Get current branch info

```bash
git branch --show-current
```

### 2. Determine the default (base) branch

Check in this order and use the first one that exists:
```bash
git show-ref --verify --quiet refs/heads/main && echo main
git show-ref --verify --quiet refs/heads/develop && echo develop
git show-ref --verify --quiet refs/heads/master && echo master
```

### 3. Gather commit and diff information

```bash
git log <base>..<current> --oneline
git diff <base>..<current> --stat
```

Also scan commit messages for issue references (e.g., `#123`, `closes #123`, `fixes #123`).

### 4. Check for a PR template

Look for these files (in order):
- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE/default.md`

### 5. Generate the PR description

**If a template exists:**
- Read the template
- Fill in every section with content derived from the actual commits and diffs
- Preserve all checkboxes (`- [ ]`) and section headers exactly as-is
- Do not remove or add sections

**If no template exists:**
Use this default format:

```markdown
## Summary

- <bullet points describing what changed>

## Changes

<technical description of modified files and what was changed>

## Related Issues

<issue links found in commits, e.g., Closes #123 — or "None" if none found>

## Screenshots

<!-- Add before/after screenshots if there are UI changes -->

## Notes for Reviewers

<any context that would help reviewers understand the change>
```

### 6. Output

Print the complete PR description as a markdown code block, ready to copy-paste into GitHub.
