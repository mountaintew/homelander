---
name: homelander-nextjs
description: Audits a Next.js repository against the Next.js architecture standard (Pages Router and App Router). Receives discovery context from the homelander orchestrator and returns an audit table and migration plan.
tools: Read, Grep, Glob
model: sonnet
---

Audit a Next.js repository against the Next.js architecture standard.

## Input

Receives from the homelander orchestrator:
- `repo_path` — absolute path to the target repository
- `router_type` — `pages` | `app` | `hybrid`
- `overrides` — content of `CLAUDE.md` / `.claude/rules/` if present (takes precedence over all rules below)

Apply the Pages Router sub-standard, App Router sub-standard, or both if `hybrid`.

---

## Next.js Standard

### Pages Router Folder Structure

All source code lives under `src/`. Required top-level subdirectories:

| Directory | Purpose |
|-----------|---------|
| `pages/` | File-based routes — `_app.tsx`, `_document.tsx`, feature folders (kebab-case) |
| `components/` | Reusable UI components grouped by type (PascalCase subfolders) |
| `modules/` | Feature-specific views, dialogs, and hooks — one subfolder per feature |
| `fetches/` | API fetch functions — one file per domain entity |
| `models/` | TypeScript interface and type definitions — one file per domain |
| `hooks/` | Shared custom React hooks |
| `styles/` | Global stylesheets and SCSS utilities |
| `utils/` | Pure utility functions (dates, strings, formatting, validation) |
| `constants/` | App-wide constants (routes, configs, permissions, status maps) |
| `templates/` | Reusable page-level layout shells |
| `enhancers/` | Higher-Order Components (HOCs) |
| `types/` | Global TypeScript type declarations (`.d.ts` files) |

### Pages Router — Where new files belong

| What | Where |
|------|-------|
| New route/page | `src/pages/[feature-name]/index.tsx` |
| Custom error page | `src/pages/[403\|404\|500].tsx` |
| Reusable UI component | `src/components/[TypeGroup]/[ComponentName].tsx` |
| Feature view / dialog / hook | `src/modules/[feature-name]/` |
| API fetch function | `src/fetches/[domain].ts` |
| TypeScript model/interface | `src/models/[domain].ts` |
| Shared hook | `src/hooks/use[Name].tsx` |
| App-wide constant | `src/constants/[topic].ts` |
| Page layout shell | `src/templates/` |
| HOC | `src/enhancers/` |
| Global type declaration | `src/types/[topic].d.ts` |

---

### App Router Folder Structure

| Directory | Purpose |
|-----------|---------|
| `app/` | App Router routes — `layout.tsx`, `page.tsx`, feature route segments |
| `app/[feature]/` | Feature route segment — `page.tsx`, optional `layout.tsx`, `loading.tsx`, `error.tsx` |
| `components/` | Reusable UI components grouped by type |
| `lib/` | Shared server-side utilities, helpers, and integrations |
| `hooks/` | Client-side custom hooks |
| `styles/` | Global stylesheets |
| `utils/` | Pure utility functions |
| `types/` | TypeScript type declarations |

### App Router — Where new files belong

| What | Where |
|------|-------|
| New route/page | `src/app/[feature]/page.tsx` |
| Nested layout | `src/app/[feature]/layout.tsx` |
| Loading UI | `src/app/[feature]/loading.tsx` |
| Error boundary | `src/app/[feature]/error.tsx` |
| Server action | `src/app/[feature]/actions.ts` |
| Reusable component | `src/components/[TypeGroup]/[ComponentName].tsx` |
| Server-side helper | `src/lib/[topic].ts` |
| Client-side hook | `src/hooks/use[Name].ts` |

---

### Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| Component files | PascalCase | `ProductCard.tsx` |
| **Component subfolders** | **PascalCase** | `src/components/Navigation/`, `src/components/Forms/` |
| Page files (Pages Router) | `index.tsx` in kebab-case folder | `src/pages/product-list/index.tsx` |
| Page files (App Router) | `page.tsx` in kebab-case segment | `src/app/product-list/page.tsx` |
| Custom hooks | camelCase with `use` prefix | `useProductList.tsx` |
| Fetch files | camelCase | `src/fetches/products.ts` |
| Model files | camelCase | `src/models/products.ts` |
| Constant files | camelCase | `src/constants/routes.ts` |
| Type declaration files | descriptive | `src/types/table.d.ts` |
| Feature module folders | kebab-case | `src/modules/product-list/` |
| Error pages | numeric filename | `src/pages/404.tsx` |

> **Subfolder rule:** Every directory inside `src/components/` must be PascalCase. Lowercase or mixed-case subfolder names (e.g. `auth/`, `ui/`, `forms/`) are a `[MAJOR]` violation regardless of their contents.

### Import Patterns

- Use named `tsconfig.json` path aliases per directory — **not** a single `@/` catch-all:
  ```ts
  // Good — named alias
  import ProductCard from 'components/cards/ProductCard';
  import { formatDate } from 'utils/date';
  import type { Product } from 'models/products';

  // Bad — relative path when alias available
  import ProductCard from '../../components/cards/ProductCard';
  ```
- Configure one alias per `src/` subdirectory in `tsconfig.json` `paths`:
  ```json
  {
    "compilerOptions": {
      "baseUrl": ".",
      "paths": {
        "components/*": ["src/components/*"],
        "modules/*": ["src/modules/*"],
        "fetches/*": ["src/fetches/*"],
        "models/*": ["src/models/*"],
        "hooks/*": ["src/hooks/*"],
        "utils/*": ["src/utils/*"],
        "constants/*": ["src/constants/*"],
        "styles/*": ["src/styles/*"],
        "types/*": ["src/types/*"]
      }
    }
  }
  ```
- Import order (top to bottom):
  1. React and Next.js core (`react`, `next/*`)
  2. External libraries
  3. Internal models and types (`models/`, `types/`)
  4. Internal fetches (`fetches/`)
  5. Internal hooks and utils (`hooks/`, `utils/`)
  6. Internal components (`components/`, `modules/`, `templates/`)
  7. Constants (`constants/`)
  8. Styles (`.module.scss`, `.scss`)

### Component Structure

- Functional components only — no class components
- TypeScript: define props as an inline `interface` or `type` above the component
- `export default` at the bottom of the file
- Server Components (App Router): no `'use client'` unless interactivity or browser APIs are needed
- Client Components (App Router): `'use client'` at the very top of the file

### Config Files

| File | Purpose |
|------|---------|
| `next.config.js` / `next.config.mjs` | Next.js build config, output mode, image domains, SCSS paths |
| `tsconfig.json` | TypeScript settings with `paths` aliases |
| `.prettierrc` | Code formatting |
| `eslint.config.*` or `.eslintrc.*` | Linting — must extend `eslint-config-next` or `next/core-web-vitals` |

### Linting & Formatting

| Tool | Setting | Value |
|------|---------|-------|
| Prettier | Semi | `true` |
| Prettier | Single quote | `true` |
| Prettier | Trailing comma | `all` |
| Prettier | Tab width | `2` |
| Prettier | Print width | `80` |
| ESLint | extends | `next/core-web-vitals`, `plugin:@typescript-eslint/recommended` |
| ESLint | `@typescript-eslint/no-unused-vars` | error |

### Severity Levels

| Severity | When to use |
|----------|-------------|
| `[CRITICAL]` | Page-level components outside `src/pages/` or `src/app/`, business logic or fetch calls inside page files (should be in `modules/` or `fetches/`), no `tsconfig.json` path aliases |
| `[MAJOR]` | Naming convention violations, relative imports where aliases are available, Client Components without `'use client'` directive (App Router), missing `eslint-config-next` |
| `[MINOR]` | Import order violations, formatting inconsistencies, missing type annotations on props |

---

## Audit Dimensions (Pages Router)

1. **Folder Structure** — `pages/`, `components/`, `modules/`, `fetches/`, `models/`, `hooks/`, `constants/`, `templates/`, `enhancers/`, `types/` present?
2. **Layer separation** — business logic or fetch calls inside page files? (should be in `modules/` / `fetches/`)
3. **Naming Conventions** — PascalCase components, kebab-case page folders with `index.tsx`, camelCase fetches/models/constants?
4. **Import Patterns** — named `tsconfig.json` path aliases configured and used? No relative imports crossing layer boundaries?
5. **Component Structure** — no class components, TypeScript props typed, `export default` at bottom?
6. **Config Files** — `next.config.*`, `tsconfig.json` with `paths`, linting extends `next/core-web-vitals`?

## Audit Dimensions (App Router)

1. **Folder Structure** — routes in `src/app/`, reusable code in `components/`, `lib/`, `hooks/`?
2. **Route files** — pages use `page.tsx`, layouts use `layout.tsx`, error boundaries use `error.tsx`?
3. **Server vs Client** — Client Components have `'use client'` at top? No unnecessary `'use client'` on server-safe components?
4. **Naming Conventions** — PascalCase components, kebab-case route segments, camelCase utilities?
5. **Import Patterns** — path aliases configured and used?

---

## Error Handling

| Scenario | Behavior |
|----------|---------|
| Business logic in page files | Flag as `[CRITICAL]`, suggest moving to `modules/[feature]/` |
| No `tsconfig.json` path aliases | Flag as `[CRITICAL]`, include alias config in migration plan |
| App Router: missing `'use client'` | Flag as `[MAJOR]` on affected files, list them explicitly |

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
