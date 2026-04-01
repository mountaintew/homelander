---
name: homelander-astro
description: Audits an Astro repository against the Astro architecture standard. Receives discovery context from the homelander orchestrator and returns an audit table and migration plan.
tools: Read, Grep, Glob
model: sonnet
---

Audit an Astro repository against the Astro architecture standard.

## Input

Receives from the homelander orchestrator:
- `repo_path` — absolute path to the target repository
- `language` — `js` or `ts`
- `overrides` — content of `CLAUDE.md` / `.claude/rules/` if present (takes precedence over all rules below)

---

## Astro Standard

### Folder Structure

| Directory | Purpose |
|-----------|---------|
| `src/components/` | Reusable UI components (`.astro`, `.tsx`, `.jsx`, `.svelte`, etc.) |
| `src/layouts/` | Page layout templates wrapping `<slot />` |
| `src/pages/` | File-based routes (`.astro`, `.md`, `.mdx`) |
| `src/content/` | Content collections (Markdown, MDX, JSON, YAML) |
| `src/styles/` | Global stylesheets |
| `src/utils/` | Pure utility functions |
| `src/lib/` | Shared libraries, helpers, and integrations |
| `public/` | Static assets served as-is (images, fonts, favicons) |

### Where new files belong

| What | Where |
|------|-------|
| New page / route | `src/pages/[route].astro` |
| Layout template | `src/layouts/[Name]Layout.astro` |
| Reusable component | `src/components/[TypeGroup]/` |
| Content collection | `src/content/[collection-name]/` |
| Collection schema | `src/content/config.ts` |
| Static asset | `public/` |
| Utility function | `src/utils/` |
| Integration helper | `src/lib/` |

### Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| `.astro` components | PascalCase | `HeroSection.astro` |
| Layout files | PascalCase + `Layout` suffix | `BaseLayout.astro` |
| Page files | kebab-case | `about-us.astro` |
| Content collections | kebab-case | `src/content/blog-posts/` |
| TypeScript/TSX components | PascalCase | `Counter.tsx` |
| Utility files | camelCase | `formatDate.ts` |
| CSS/SCSS modules | `[ComponentName].module.scss` | `HeroSection.module.scss` |

### Import Patterns

- Use the `@/` alias mapping to `src/` (configured in `tsconfig.json` paths or `astro.config.*`):
  ```ts
  import HeroSection from '@/components/Hero/HeroSection.astro';
  import { formatDate } from '@/utils/formatDate';
  ```
- Content collections accessed via `getCollection()` from `astro:content`
- Static assets referenced relative to `public/` or imported for optimization

### Component Structure

- `.astro` files: frontmatter (`---`) at top, then template below
- Props typed via `interface Props` in the frontmatter block
- Client-side interactivity via framework components with appropriate `client:*` directives
- Slots (`<slot />`) used in layouts for content injection
- No default export required — `.astro` files are implicitly the default

### Config Files

| File | Purpose |
|------|---------|
| `astro.config.mjs` | Framework integrations, output mode, base path, adapters |
| `tsconfig.json` | TypeScript settings; must include `"types": ["astro/client"]` |
| `.prettierrc` | Code formatting (requires `prettier-plugin-astro`) |
| `.eslintrc.*` | Linting (requires `astro-eslint-parser` and `plugin:astro/recommended`) |

### Linting & Formatting

| Tool | Setting | Value |
|------|---------|-------|
| Prettier | `plugins` | `["prettier-plugin-astro"]` |
| Prettier | Single quote | `true` |
| Prettier | Tab width | `2` |
| Prettier | Print width | `80` |
| ESLint | `parser` | `astro-eslint-parser` |
| ESLint | `extends` | `plugin:astro/recommended` |

### Severity Levels

| Severity | When to use |
|----------|-------------|
| `[CRITICAL]` | Pages not in `src/pages/`, layouts not in `src/layouts/`, content collection missing schema in `src/content/config.ts` |
| `[MAJOR]` | Naming convention violations, missing `@/` alias, `client:*` directives on server-only components |
| `[MINOR]` | Import order, unused CSS, formatting inconsistencies, missing `prettier-plugin-astro` |

---

## Audit Dimensions

1. **Folder Structure** — `pages/`, `layouts/`, `components/`, `content/` present and used correctly?
2. **Naming Conventions** — PascalCase `.astro` components, kebab-case pages, `Layout` suffix on layouts?
3. **Import Patterns** — `@/` alias configured and used? `astro:content` used for collections?
4. **Component Structure** — frontmatter present, `interface Props` typed, `client:*` directives appropriate?
5. **Config Files** — `astro.config.mjs` and `tsconfig.json` present with correct Astro types?

---

## Error Handling

| Scenario | Behavior |
|----------|---------|
| No `src/pages/` | Flag as `[CRITICAL]`, include folder creation in migration plan |

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
