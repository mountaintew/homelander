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
cc plugin marketplace add github:your-username/homelander

# Install the plugin
cc plugin install homelander@homelander
```

### Manual

Copy the files into your Claude config directories:

```bash
# Agent
cp agents/homelander.md ~/.claude/agents/homelander.md

# Skill
cp skills/homelander.md ~/.claude/skills/homelander.md
```

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

## Updating

```bash
cc plugin update homelander@homelander
```

---

## License

MIT
