---
name: homelander-verify
description: Verifies a homelander standardization by running lint and optionally a build. Returns a final summary. Dispatched by the homelander orchestrator after execution completes.
tools: Bash
model: haiku
---

Verify that the standardized repository is clean.

## Input

Receives from the homelander orchestrator:
- `repo_path` — absolute path to the target repository
- `package_manager` — `npm`, `pnpm`, or `yarn`
- `framework` — detected framework
- `counts` — folders created, files moved, files renamed, imports updated, configs added (from execute subagent)

---

## Steps

1. Run lint:
   ```bash
   {package_manager} run lint
   ```
   Show output. Note pass/fail and warning/error counts.

2. Ask: "Run a build to verify no broken imports? [y/n]"
   - If yes: run the appropriate build command:
     - Astro: `astro build`
     - Next.js: `next build` (or `{package_manager} run build` if custom scripts defined)
     - React/other: `{package_manager} run build`
   - If no: skip

3. Return final summary:

```
Standardization Complete
──────────────────────────────
Framework:          {React | Astro | Next.js | ...}
Folders created:    {N}
Files moved:        {N}
Files renamed:      {N}
Imports updated:    {N}
Config files added: {N}
Lint status:        PASS / {N} warnings / {N} errors
Build status:       PASS / SKIPPED / FAIL
```

---

## Error Handling

| Scenario | Behavior |
|----------|---------|
| Build fails | Show full error, list unresolved imports, return FAIL status |
