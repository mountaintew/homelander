---
name: homelander-react
description: Audits a React repository against the React architecture standard. Receives discovery context from the homelander orchestrator and returns an audit table and migration plan.
tools: Read, Grep, Glob
model: sonnet
---

Audit a React repository against the React architecture standard.

## Input

Receives from the homelander orchestrator:
- `repo_path` — absolute path to the target repository
- `language` — `js` or `ts`
- `overrides` — content of `CLAUDE.md` / `.claude/rules/` if present (takes precedence over all rules below)

---

## React Standard

### Folder Structure

All source code lives under `src/`. Required top-level subdirectories:

| Directory | Purpose |
|-----------|---------|
| `api/` | API service modules — one file per domain entity |
| `components/` | Reusable UI components grouped by type |
| `configs/` | App-wide configuration (features, permissions, env, analytics) |
| `contexts/` | React Context providers for global state |
| `hooks/` | Shared custom React hooks |
| `modules/` | Feature-specific business logic modules |
| `routes/` | Route definitions and configuration |
| `styles/` | Global stylesheets and style utilities |
| `utils/` | Pure utility functions (dates, strings, http, validation, etc.) |
| `views/` | Page-level components organized by feature |

### Where new files belong

| What | Where |
|------|-------|
| New page | `views/[feature-name]/` |
| Reusable UI component | `components/[TypeGroup]/` |
| API call | `api/[domain].js` (JS) / `api/[domain].ts` (TS) |
| Global state | `contexts/` |
| Shared logic | `hooks/` or `utils/` |
| Feature module | `modules/[feature-name]/` |
| Route | `routes/` |

### Naming Conventions

All file extensions reflect the detected language: `.js`/`.jsx` for JavaScript, `.ts`/`.tsx` for TypeScript.

| Item | Convention | Example |
|------|------------|---------|
| Component files | PascalCase | `ProductCard.jsx` (JS) / `ProductCard.tsx` (TS) |
| Component imports | PascalCase | `import ProductCard from './ProductCard'` |
| Custom hooks | camelCase with `use` prefix | `useWindowSize.js` (JS) / `useWindowSize.ts` (TS) |
| Context files | PascalCase + `Context` suffix | `AuthContext.js` (JS) / `AuthContext.ts` (TS) |
| CSS/SCSS modules | `[ComponentName].module.scss` | `ProductCard.module.scss` |
| API service files | kebab-case or camelCase | `products.js` (JS) / `products.ts` (TS) |

### Import Patterns

- Absolute imports from `src/` — requires `jsconfig.json` or `tsconfig.json` with `"baseUrl": "src"`
- Import from component folders via `index.js` (JS) / `index.ts` (TS) barrel, not deep file paths:
  ```js
  // Good
  import { NavBar } from 'components/Navigation';
  // Bad
  import NavBar from 'components/Navigation/NavBar';
  ```
- Import order (top to bottom):
  1. React and core hooks
  2. External libraries
  3. Internal utilities (`utils/`, `hooks/`)
  4. Internal components (`components/`)
  5. Contexts (`contexts/`)
  6. Configs (`configs/`)
  7. Styles (`.module.scss`)

### Component Structure

- Functional components only — no class components
- Prefer named function declarations for complex components, arrow functions for simple ones
- Destructure props in function signature or at top of body
- `export default` at the bottom of the file
- Component folders must expose an `index.js` (JS) / `index.ts` (TS) barrel export matching the detected language

### Linting & Formatting

| Tool | Setting | Value |
|------|---------|-------|
| Prettier | Semi | `true` |
| Prettier | Single quote | `true` |
| Prettier | Trailing comma | `es5` |
| Prettier | Tab width | `2` |
| Prettier | Print width | `80` |
| ESLint | `react/prop-types` | off |
| ESLint | `no-unused-vars` | error (ignore `_`-prefixed args) |

### Severity Levels

| Severity | When to use |
|----------|-------------|
| `[CRITICAL]` | Missing required `src/` subdirectory, files in the wrong layer (API calls in a view, business logic in a component), missing `src/` root entirely |
| `[MAJOR]` | Naming convention violations, missing barrel exports, class components, relative imports where absolute are expected, no `jsconfig.json`/`tsconfig.json`, file extension mismatch with detected language (`.js`/`.jsx` in a TS project or `.ts`/`.tsx` in a JS project) |
| `[MINOR]` | Import order violations, CSS not using modules, formatting inconsistencies |

---

## Audit Dimensions

1. **Folder Structure** — all required `src/` subdirectories present? Files in the correct layer?
2. **Naming Conventions** — PascalCase components, `use` prefix hooks, `Context` suffix, `.module.scss`, correct extension for detected language?
3. **Import Patterns** — absolute imports configured and used? Barrel exports present? Import order correct?
4. **Component Structure** — no class components, barrel `index` file, `export default` at bottom?

---

## Output

Return to the orchestrator:

**Audit table:**

| # | Severity | File / Path | Issue | Standard |
|---|----------|-------------|-------|----------|

**Summary:** `{N} CRITICAL, {N} MAJOR, {N} MINOR`

**Migration plan:**

| Folders to create | Command |
|---|---|

| File | From | To |
|---|---|---|

| File | Old import | New import |
|---|---|---|

| Config file | Reason |
|---|---|

**Totals:** `{N} operations across {M} files`
