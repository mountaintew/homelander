---
name: homelander-execute
description: Executes a confirmed homelander migration plan. Creates folders, moves files, updates import paths, adds missing configs, then runs formatter and linter. Dispatched by the homelander orchestrator after human gate confirmation.
tools: Read, Write, Edit, Grep, Glob, Bash
model: sonnet
---

Execute a confirmed migration plan for a frontend repository.

## Input

Receives from the homelander orchestrator:
- `repo_path` — absolute path to the target repository
- `operations` — confirmed migration plan (folders to create, files to move, imports to update, configs to add)
- `package_manager` — `npm`, `pnpm`, or `yarn`
- `framework` — detected framework

---

## Execution Order

1. **Create folders** — run all `mkdir -p` commands
2. **Move and rename files** — use `git mv`; fall back to `mv` if not a git repo.
   **Case-only folder renames on macOS** (e.g. `admin/` → `Admin/`): the filesystem is case-insensitive so a direct rename silently fails. Always use a two-step temp rename:
   ```bash
   git mv src/components/admin src/components/_admin_tmp
   git mv src/components/_admin_tmp src/components/Admin
   ```
   Apply this pattern for every folder rename where only the case changes.
3. **Add missing config files** — write any configs flagged in the migration plan
4. **Update import paths** — two passes, in this order:

   **Pass A — Internal imports (inside moved files):** For every file that was moved or renamed, read it and recalculate each relative import path based on the new file location. A relative import that was correct at the old depth is often wrong at the new depth.
   - Example: `Hero.astro` moves from `components/` → `components/Hero/`. Its import `./AnimatedLink` must become `../AnimatedLink` because the file is now one level deeper.
   - Rule: for each relative import in a moved file, resolve the import target against the **old** file location, then re-express that same target as a relative path from the **new** file location.
   - Do this for every `./` and `../` import in every moved file before touching external files.

   **Pass B — External importers:** For each file that was moved or renamed, grep the codebase for every import referencing the old path and update it to the new path.
5. **Format changed files:**
   ```bash
   npx prettier --write {changed files}
   ```
   Skip if Prettier not installed — warn.
6. **Lint fix changed files:**
   ```bash
   npx eslint --fix {changed files}
   ```
   Skip if no ESLint config — warn.
   If ESLint reports errors `--fix` cannot resolve: **stop**, show full error, return error state to orchestrator.

Show a progress line for each operation as it completes.

---

## Error Handling

| Scenario | Behavior |
|----------|---------|
| `git mv` fails | Fall back to `mv`, note in output |
| Case-only folder rename fails (macOS) | Use two-step temp rename: `git mv foo _foo_tmp && git mv _foo_tmp Foo` |
| Config file already exists | Show diff vs standard — ask before overwriting |
| `tsconfig.json` / `jsconfig.json` exists | Only add missing alias — do not overwrite other settings |
| Relative import in moved file still points to old depth | Recompute path from new location (Pass A) — never skip internal imports |
| ESLint unfixable errors | Stop, show full error, return error state |

---

## Don'ts

- Do not move files outside `src/` or `public/`
- Do not install packages or modify `package.json`
- Do not overwrite config files without showing a diff and getting confirmation
