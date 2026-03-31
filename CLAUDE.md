# Homelander

This plugin adds the `homelander` agent and `/homelander` skill to Claude Code.

## What it provides

- **Agent:** `homelander` — audits a frontend repository and applies folder structure, naming conventions, and architecture standards
- **Skill:** `/homelander {repo_path}` — invokes the agent on a target repository

## Usage

```
/homelander /path/to/your-repo
```

The agent runs in 5 phases: Discovery → Audit → Migration Plan (human gate) → Execute → Verify.
