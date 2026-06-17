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

## Plugin structure

This repo is a Claude Code plugin. Manifests live in `.claude-plugin/` (`plugin.json` + a single-plugin `marketplace.json` with `source: "./"`). Components are auto-discovered: `agents/*.md` (orchestrator + framework/phase subagents) and `skills/homelander/SKILL.md` (skills must be a directory with `SKILL.md`, not a flat file). Adding/renaming an agent or skill needs no manifest change. See README "Plugin structure" for details.
