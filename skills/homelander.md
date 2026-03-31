---
name: homelander
description: Use when auditing or aligning a frontend repository against the project folder structure, naming conventions, and architecture standards. Triggers on requests to organize, standardize, or restructure a repo.
---

Invoke the `homelander` agent to audit and align the target repository.

## Usage

```
/homelander {repo_path}
```

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `repo_path` | string | yes | Absolute path to the target repository (e.g. `/path/to/your-repo`) |

## What it does

Runs in 5 phases:

| Phase | Name | Output |
|-------|------|--------|
| 1 | Discovery | Scans target repo, reads config files, summarizes current state |
| 2 | Audit | Gap report with `[CRITICAL]` / `[MAJOR]` / `[MINOR]` findings |
| 3 | Migration Plan | Tables of all folder creates, file moves, import updates — **HUMAN GATE** |
| 4 | Execute | Applies all confirmed changes, runs formatter + linter |
| 5 | Verify | Lint check, optional build check, final summary |

## Example

```
/homelander /path/to/your-repo
```

Dispatch the `homelander` agent with the provided `repo_path` as input.
