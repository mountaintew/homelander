# Homelander

A Claude Code plugin that audits any frontend repository against an established architecture standard — and fixes it.

Point it at a repo. It scans, reports violations by severity, proposes a migration plan, and applies all changes after your confirmation.

---

## What it does

Homelander runs in **5 phases**:

| Phase | Name | What happens |
|-------|------|--------------|
| 1 | **Discovery** | Scans the target repo tree, reads config files, summarizes current state |
| 2 | **Audit** | Compares against the standard — outputs a severity-tagged gap report |
| 3 | **Migration Plan** | Lists every folder create, file move, rename, and import update needed — **requires your confirmation before anything runs** |
| 4 | **Execute** | Applies all confirmed changes, runs Prettier + ESLint fix |
| 5 | **Verify** | Runs lint, optional build check, outputs a final change summary |

### What gets audited

- **Folder structure** — required `src/` subdirectories (`api/`, `components/`, `configs/`, `contexts/`, `hooks/`, `modules/`, `routes/`, `styles/`, `utils/`, `views/`)
- **Naming conventions** — PascalCase components, `useX` hooks, `XContext` contexts, `.module.scss` styles
- **Import patterns** — absolute imports from `src/`, barrel exports via `index.js`, import order
- **Component structure** — functional components only, `export default` at bottom, `index.js` barrel exports

### Severity levels

| Level | Meaning |
|-------|---------|
| `[CRITICAL]` | Wrong layer, missing required directory, misplaced files |
| `[MAJOR]` | Naming violations, missing barrel exports, class components, relative imports |
| `[MINOR]` | Import order, missing CSS modules, formatting |

---

## Installation

### Via Claude Code plugin system

```bash
# Add this repo as a marketplace
cc plugin marketplace add github:mountaintew/homelander

# Install the plugin
cc plugin install homelander@homelander
```

### Manual

Copy the components into your Claude config directories:

```bash
# Agents (orchestrator + framework/phase subagents)
cp agents/*.md ~/.claude/agents/

# Skill (a directory containing SKILL.md)
mkdir -p ~/.claude/skills/homelander
cp skills/homelander/SKILL.md ~/.claude/skills/homelander/SKILL.md
```

---

## Plugin structure

Homelander is a Claude Code plugin: a repository whose root holds a `.claude-plugin/` manifest plus component directories that Claude Code discovers automatically. There is no build step.

```text
homelander/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest (name, version, description, author)
│   └── marketplace.json    # single-plugin marketplace catalog (plugin source: "./")
├── agents/                 # one .md per agent, frontmatter: name / description / tools / model
│   ├── homelander.md       # orchestrator
│   └── homelander-*.md     # framework + phase subagents
└── skills/
    └── homelander/
        └── SKILL.md        # the /homelander skill, frontmatter: name / description
```

How the pieces fit, if you want to build your own:

- **`.claude-plugin/plugin.json`** — the plugin manifest. Only `name` (kebab-case) is required; set `version` to pin releases (without it, every commit is treated as a new version).
- **`.claude-plugin/marketplace.json`** — lets this same repo double as a one-plugin marketplace. The entry's `source: "./"` points the `homelander` plugin at the repo root (paths resolve relative to the directory containing `.claude-plugin/`), which is what makes `cc plugin marketplace add github:mountaintew/homelander` resolve.
- **`agents/` and `skills/`** are auto-discovered on install — they do **not** need to be listed in `plugin.json`. Agents are flat `.md` files; a skill must be a directory containing `SKILL.md`.

The marketplace name and plugin name are both `homelander`, so the install target is `homelander@homelander` (`plugin@marketplace`).

---

## Usage

```
/homelander /path/to/your-repo
```

The agent will ask for the path if you don't provide one.

### Example

```
/homelander /Users/you/projects/my-react-app
```

**Phase 1 — Discovery output:**
```
Target: /Users/you/projects/my-react-app
Stack: React 18, Vite, no TypeScript
Structure: src/ present, missing contexts/ hooks/ modules/
Config: .prettierrc found, no jsconfig.json
```

**Phase 2 — Audit output:**

| # | Severity | File / Path | Issue | Standard |
|---|----------|-------------|-------|----------|
| 1 | [CRITICAL] | src/ | Missing: contexts/, hooks/, modules/ | Required subdirectories |
| 2 | [MAJOR] | src/components/card.js | Filename not PascalCase | Should be Card.js |
| 3 | [MAJOR] | src/components/ | No index.js barrel export | All component folders need index.js |
| 4 | [MINOR] | src/views/Home.js | Relative import ../utils/dates | Should use absolute: utils/dates |

**Phase 3 — Migration Plan (human gate):**
```
Ready to apply 8 operations across 5 files? [y/n]
```

---

## Compatible plugins

Homelander works alongside other Claude Code plugins. The following are known compatible plugins that extend or complement it:

| Plugin | Author | What it adds |
|--------|--------|-------------|
| [superpowers](https://github.com/obra/superpowers) | Jesse Vincent | Skill orchestration, brainstorming, TDD, debugging, and planning workflows — pairs well with homelander's audit-and-fix cycle |
| [figma](https://github.com/figma/claude-code-figma) | Figma | Translate Figma designs into components; useful after homelander restructures your `components/` layer |
| [code-review](https://github.com/anthropics/claude-code-plugins) | Anthropic | Post-migration PR review to validate homelander's changes against your standards |
| [ui-ux-pro-max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | nextlevelbuilder | UI/UX design intelligence for generating components that land in the correct homelander-standardized folders |

> To suggest a plugin, open an issue or PR.

---

## Updating

```bash
cc plugin update homelander@homelander
```

---

## License

MIT
