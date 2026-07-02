# Consolidated Skills

Table of contents:

- link-workspace-packages — Link workspace packages in monorepos
- nx-run-tasks — Run Nx tasks (run, run-many, affected)
- nx-plugins — Find and add Nx plugins
- nx-generate — Generate code using Nx generators
- nx-import — Import/merge repositories into an Nx workspace
- nx-workspace — Explore and understand Nx workspaces

---

<!-- Begin: link-workspace-packages -->

---
name: link-workspace-packages
description: 'Link workspace packages in monorepos (npm, yarn, pnpm, bun). USE WHEN: (1) you just created or generated new packages and need to wire up their dependencies, (2) user imports from a sibling package and needs to add it as a dependency, (3) you get resolution errors for workspace packages (@org/*) like "cannot find module", "failed to resolve import", "TS2307", or "cannot resolve". DO NOT patch around with tsconfig paths or manual package.json edits - use the package manager''s workspace commands to fix actual linking.'
---

# Link Workspace Packages

Add dependencies between packages in a monorepo. All package managers support workspaces but with different syntax.

## Detect Package Manager

Check whether there's a `packageManager` field in the root-level `package.json`.

Alternatively check lockfile in repo root:

- `pnpm-lock.yaml` → pnpm
- `yarn.lock` → yarn
- `bun.lock` / `bun.lockb` → bun
- `package-lock.json` → npm

## Workflow

1. Identify consumer package (the one importing)
2. Identify provider package(s) (being imported)
3. Add dependency using package manager's workspace syntax
4. Verify symlinks created in consumer's `node_modules/`

---

## pnpm

Uses `workspace:` protocol - symlinks only created when explicitly declared.

```bash
# From consumer directory
pnpm add @org/ui --workspace

# Or with --filter from anywhere
pnpm add @org/ui --filter @org/app --workspace
```

Result in `package.json`:

```json
{ "dependencies": { "@org/ui": "workspace:*" } }
```

---

## yarn (v2+/berry)

Also uses `workspace:` protocol.

```bash
yarn workspace @org/app add @org/ui
```

Result in `package.json`:

```json
{ "dependencies": { "@org/ui": "workspace:^" } }
```

---

## npm

No `workspace:` protocol. npm auto-symlinks workspace packages.

```bash
npm install @org/ui --workspace @org/app
```

Result in `package.json`:

```json
{ "dependencies": { "@org/ui": "*" } }
```

npm resolves to local workspace automatically during install.

---

## bun

Supports `workspace:` protocol (pnpm-compatible).

```bash
cd packages/app && bun add @org/ui
```

Result in `package.json`:

```json
{ "dependencies": { "@org/ui": "workspace:*" } }
```

---

## Examples

**Example 1: pnpm - link ui lib to app**

```bash
pnpm add @org/ui --filter @org/app --workspace
```

**Example 2: npm - link multiple packages**

```bash
npm install @org/data-access @org/ui --workspace @org/dashboard
```

**Example 3: Debug "Cannot find module"**

1. Check if dependency is declared in consumer's `package.json`
2. If not, add it using appropriate command above
3. Run install (`pnpm install`, `npm install`, etc.)

## Notes

- Symlinks appear in `<consumer>/node_modules/@org/<package>`
- **Hoisting differs by manager:**
  - npm/bun: hoist shared deps to root `node_modules`
  - pnpm: no hoisting (strict isolation, prevents phantom deps)
  - yarn berry: uses Plug'n'Play by default (no `node_modules`)
- Root `package.json` should have `"private": true` to prevent accidental publish

<!-- End: link-workspace-packages -->

---


<!-- Begin: nx-run-tasks -->

---
name: nx-run-tasks
description: Helps with running tasks in an Nx workspace. USE WHEN the user wants to execute build, test, lint, serve, or run any other tasks defined in the workspace.
---

You can run tasks with Nx in the following way.

Keep in mind that you might have to prefix things with npx/pnpx/yarn if the user doesn't have nx installed globally. Look at the package.json or lockfile to determine which package manager is in use.

For more details on any command, run it with `--help` (e.g. `nx run-many --help`, `nx affected --help`).

## Understand which tasks can be run

You can check those via `nx show project <projectname> --json`, for example `nx show project myapp --json`. It contains a `targets` section which has information about targets that can be run. You can also just look at the `package.json` scripts or `project.json` targets, but you might miss out on inferred tasks by Nx plugins.

## Run a single task

```
nx run <project>:<task>
```

where `project` is the project name defined in `package.json` or `project.json` (if present).

## Run multiple tasks

```
nx run-many -t build test lint typecheck
```

You can pass a `-p` flag to filter to specific projects, otherwise it runs on all projects. You can also use `--exclude` to exclude projects, and `--parallel` to control the number of parallel processes (default is 3).

Examples:

- `nx run-many -t test -p proj1 proj2` — test specific projects
- `nx run-many -t test --projects=*-app --exclude=excluded-app` — test projects matching a pattern
- `nx run-many -t test --projects=tag:api-*` — test projects by tag

## Run tasks for affected projects

Use `nx affected` to only run tasks on projects that have been changed and projects that depend on changed projects. This is especially useful in CI and for large workspaces.

```
nx affected -t build test lint
```

By default it compares against the base branch. You can customize this:

- `nx affected -t test --base=main --head=HEAD` — compare against a specific base and head
- `nx affected -t test --files=libs/mylib/src/index.ts` — specify changed files directly

## Useful flags

These flags work with `run`, `run-many`, and `affected`:

- `--skipNxCache` — rerun tasks even when results are cached
- `--verbose` — print additional information such as stack traces
- `--nxBail` — stop execution after the first failed task
- `--configuration=<name>` — use a specific configuration (e.g. `production`)

<!-- End: nx-run-tasks -->

---

<!-- Begin: nx-plugins -->

---
name: nx-plugins
description: Find and add Nx plugins. USE WHEN user wants to discover available plugins, install a new plugin, or add support for a specific framework or technology to the workspace.
---

## Finding and Installing new plugins

- List plugins: `pnpm nx list`
- Install plugins `pnpm nx add <plugin>`. Example: `pnpm nx add @nx/react`.

<!-- End: nx-plugins -->

---

<!-- Begin: nx-generate -->

---
name: nx-generate
description: Generate code using nx generators. INVOKE IMMEDIATELY when user mentions scaffolding, setup, structure, creating apps/libs, or setting up project structure. Trigger words - scaffold, setup, create a new app, create a new lib, project structure, generate, add a new project. ALWAYS use this BEFORE calling nx_docs or exploring - this skill handles discovery internally.
---

# Run Nx Generator

Nx generators are powerful tools that scaffold projects, make automated code migrations or automate repetitive tasks in a monorepo. They ensure consistency across the codebase and reduce boilerplate work.

This skill applies when the user wants to:

- Create new projects like libraries or applications
- Scaffold features or boilerplate code
- Run workspace-specific or custom generators
- Do anything else that an nx generator exists for

## Key Principles

1. **Always use `--no-interactive`** - Prevents prompts that would hang execution
2. **Read the generator source code** - The schema alone is not enough; understand what the generator actually does
3. **Match existing repo patterns** - Study similar artifacts in the repo and follow their conventions
4. **Verify with lint/test/build/typecheck etc.** - Generated code must pass verification. The listed targets are just an example, use what's appropriate for this workspace.

## Steps

### 1. Discover Available Generators

Use the Nx CLI to discover available generators:

- List all generators for a plugin: `npx nx list @nx/react`
- View available plugins: `npx nx list`

This includes plugin generators (e.g., `@nx/react:library`) and local workspace generators.

### 2. Match Generator to User Request

Identify which generator(s) could fulfill the user's needs. Consider what artifact type they want, which framework is relevant, and any specific generator names mentioned.

**IMPORTANT**: When both a local workspace generator and an external plugin generator could satisfy the request, **always prefer the local workspace generator**. Local generators are customized for the specific repo's patterns.

If no suitable generator exists, you can stop using this skill. However, the burden of proof is high—carefully consider all available generators before deciding none apply.

### 3. Get Generator Options

Use the `--help` flag to understand available options:

```bash
npx nx g @nx/react:library --help
```

Pay attention to required options, defaults that might need overriding, and options relevant to the user's request.

### 4. Read Generator Source Code

**This step is critical.** The schema alone does not tell you everything. Reading the source code helps you:

- Know exactly what files will be created/modified and where
- Understand side effects (updating configs, installing deps, etc.)
- Identify behaviors and options not obvious from the schema
- Understand how options interact with each other

To find generator source code:

- For plugin generators: Use `node -e "console.log(require.resolve('@nx/<plugin>/generators.json'));"` to find the generators.json, then locate the source from there
- If that fails, read directly from `node_modules/<plugin>/generators.json`
- For local generators: Typically in `tools/generators/` or a local plugin directory. Search the repo for the generator name.

After reading the source, reconsider: Is this the right generator? If not, go back to step 2.

> **⚠️ `--directory` flag behavior can be misleading.**
> It should specify the full path of the generated library or component, not the parent path that it will be generated in.
>
> ```bash
> # ✅ Correct - directory is the full path for the library
> nx g @nx/react:library --directory=libs/my-lib
> # generates libs/my-lib/package.json and more
>
> # ❌ Wrong - this will create files at libs and libs/src/...
> nx g @nx/react:library --name=my-lib --directory=libs
> # generates libs/package.json and more
> ```

### 5. Examine Existing Patterns

Before generating, examine the target area of the codebase:

- Look at similar existing artifacts (other libraries, applications, etc.)
- Identify naming conventions, file structures, and configuration patterns
- Note which test runners, build tools, and linters are used
- Configure the generator to match these patterns

### 6. Dry-Run to Verify File Placement

**Always run with `--dry-run` first** to verify files will be created in the correct location:

```bash
npx nx g @nx/react:library --name=my-lib --dry-run --no-interactive
```

Review the output carefully. If files would be created in the wrong location, adjust your options based on what you learned from the generator source code.

Note: Some generators don't support dry-run (e.g., if they install npm packages). If dry-run fails for this reason, proceed to running the generator for real.

### 7. Run the Generator

Execute the generator:

```bash
nx generate <generator-name> <options> --no-interactive
```

> **Tip:** New packages often need workspace dependencies wired up (e.g., importing shared types, being consumed by apps). The `link-workspace-packages` skill can help add these correctly.

### 8. Modify Generated Code (If Needed)

Generators provide a starting point. Modify the output as needed to:

- Add or modify functionality as requested
- Adjust imports, exports, or configurations
- Integrate with existing code patterns

**Important:** If you replace or delete generated test files (e.g., `*.spec.ts`), either write meaningful replacement tests or remove the `test` target from the project configuration. Empty test suites will cause `nx test` to fail.

### 9. Format and Verify

Format all generated/modified files:

```bash
nx format --fix
```

This example is for built-in nx formatting with prettier. There might be other formatting tools for this workspace, use these when appropriate.

Then verify the generated code works. Keep in mind that the changes you make with a generator or subsequent modifications might impact various projects so it's usually not enough to only run targets for the artifact you just created.

```bash
# these targets are just an example!
nx run-many -t build,lint,test,typecheck
```

If verification fails with manageable issues (a few lint errors, minor type issues), fix them. If issues are extensive, attempt obvious fixes first, then escalate to the user with details about what was generated, what's failing, and what you've attempted.

<!-- End: nx-generate -->

---

<!-- Begin: nx-import -->

---
name: nx-import
description: Import, merge, or combine repositories into an Nx workspace using nx import. USE WHEN the user asks to adopt Nx across repos, move projects into a monorepo, or bring code/history from another repository.
---

## Quick Start

- `nx import` brings code from a source repository or folder into the current workspace, preserving commit history.
- After nx `22.6.0`, `nx import` responds with .ndjson outputs and follow-up questions. For earlier versions, always run with `--no-interactive` and specify all flags directly.
- Run `nx import --help` for available options.
- Make sure the destination directory is empty before importing.
  EXAMPLE: target has `libs/utils` and `libs/models`; source has `libs/ui` and `libs/data-access` — you cannot import `libs/` into `libs/` directly. Import each source library individually.

Primary docs:

- https://nx.dev/docs/guides/adopting-nx/import-project
- https://nx.dev/docs/guides/adopting-nx/preserving-git-histories

Read the nx docs if you have the tools for it.

## Import Strategy

**Subdirectory-at-a-time** (`nx import <source> apps --source=apps`):

- **Recommended for monorepo sources** — files land at top level, no redundant config
- Caveats: multiple import commands (separate merge commits each); dest must not have conflicting directories; root configs (deps, plugins, targetDefaults) not imported
- **Directory conflicts**: Import into alternate-named dir (e.g. `imported-apps/`), then rename

**Whole repo** (`nx import <source> imported --source=.`):

- **Only for non-monorepo sources** (single-project repos)
- For monorepos, creates messy nested config (`imported/nx.json`, `imported/tsconfig.base.json`, etc.)
- If you must: keep imported `tsconfig.base.json` (projects extend it), prefix workspace globs and executor paths

### Directory Conventions

- **Always prefer the destination's existing conventions.** Source uses `libs/` but dest uses `packages/`? Import into `packages/` (`nx import <source> packages/foo --source=libs/foo`).
- If dest has no convention (empty workspace), ask the user.

### Application vs Library Detection

Before importing, identify whether the source is an **application** or a **library**:

- **Applications**: Deployable end products. Common indicators:
  - _Frontend_: `next.config.*`, `vite.config.*` with a build entry point, framework-specific app scaffolding (CRA, Angular CLI app, etc.)
  - _Backend (Node.js)_: Express/Fastify/NestJS server entrypoint, no `"exports"` field in `package.json`
  - _JVM_: Maven `pom.xml` with `<packaging>jar</packaging>` or `<packaging>war</packaging>` and a `main` class; Gradle `application` plugin or `mainClass` setting
  - _.NET_: `.csproj`/`.fsproj` with `<OutputType>Exe</OutputType>` or `<OutputType>WinExe</OutputType>`
  - _General_: Dockerfile, a runnable entrypoint, no public API surface intended for import by other projects
- **Libraries**: Reusable packages consumed by other projects. Common indicators: `"main"`/`"exports"` in `package.json`, Maven/Gradle packaging as a library jar, .NET `<OutputType>Library</OutputType>`, named exports intended for import by other packages.

**Destination directory rules**:

- Applications → `apps/<name>`. Check workspace globs (e.g. `pnpm-workspace.yaml`, `workspaces` in root `package.json`) for an existing `apps/*` entry.
  - If `apps/*` is **not** present, add it before importing: update the workspace glob config and commit (or stage) the change.
  - Example: `nx import <source> apps/my-app --source=packages/my-app`
- Libraries → follow the dest's existing convention (`packages/`, `libs/`, etc.).

## Common Issues

### pnpm Workspace Globs (Critical)

`nx import` adds the imported directory itself (e.g. `apps`) to `pnpm-workspace.yaml`, **NOT** glob patterns for packages within it. Cross-package imports will fail with `Cannot find module`.

**Fix**: Replace with proper globs from the source config (e.g. `apps/*`, `libs/shared/*`), then `pnpm install`.

### Root Dependencies and Config Not Imported (Critical)

`nx import` does **NOT** merge from the source's root:

- `dependencies`/`devDependencies` from `package.json`
- `targetDefaults` from `nx.json` (e.g. `"@nx/esbuild:esbuild": { "dependsOn": ["^build"] }` — critical for build ordering)
- `namedInputs` from `nx.json` (e.g. `production` exclusion patterns for test files)
- Plugin configurations from `nx.json`

**Fix**: Diff source and dest `package.json` + `nx.json`. Add missing deps, merge relevant `targetDefaults` and `namedInputs`.

### TypeScript Project References

After import, run `nx sync --yes`. If it reports nothing but typecheck still fails, `nx reset` first, then `nx sync --yes` again.

### Explicit Executor Path Fixups

Inferred targets (via Nx plugins) resolve config relative to project root — no changes needed. Explicit executor targets (e.g. `@nx/esbuild:esbuild`) have workspace-root-relative paths (`main`, `outputPath`, `tsConfig`, `assets`, `sourceRoot`) that must be prefixed with the import destination directory.

### Plugin Detection

- **Whole-repo import**: `nx import` detects and offers to install plugins. Accept them.
- **Subdirectory import**: Plugins NOT auto-detected. Manually add with `npx nx add @nx/PLUGIN`. Check `include`/`exclude` patterns — defaults won't match alternate directories (e.g. `apps-beta/`).
- Run `npx nx reset` after any plugin config changes.

### Redundant Root Files (Whole-Repo Only)

Whole-repo import brings ALL source root files into the dest subdirectory. Clean up:

- `pnpm-lock.yaml` — stale; dest has its own lockfile
- `pnpm-workspace.yaml` — source workspace config; conflicts with dest
- `node_modules/` — stale symlinks pointing to source filesystem
- `.gitignore` — redundant with dest root `.gitignore`
- `nx.json` — source Nx config; dest has its own
- `README.md` — optional; keep or remove

**Don't blindly delete** `tsconfig.base.json` — imported projects may extend it via relative paths.

### Root ESLint Config Missing (Subdirectory Import)

Subdirectory import doesn't bring the source's root `eslint.config.mjs`, but project configs reference `../../eslint.config.mjs`.

**Fix order**:

1. Install ESLint deps first: `pnpm add -wD eslint@^9 @nx/eslint-plugin typescript-eslint` (plus framework-specific plugins)
2. Create root `eslint.config.mjs` (copy from source or create with `@nx/eslint-plugin` base rules)
3. Then `npx nx add @nx/eslint` to register the plugin in `nx.json`

Install `typescript-eslint` explicitly — pnpm's strict hoisting won't auto-resolve this transitive dep of `@nx/eslint-plugin`.

### ESLint Version Pinning (Critical)

**Pin ESLint to v9** (`eslint@^9.0.0`). ESLint 10 breaks `@nx/eslint` and many plugins with cryptic errors like `Cannot read properties of undefined (reading 'version')`.

`@nx/eslint` may peer-depend on ESLint 8, causing the wrong version to resolve. If lint fails with `Cannot read properties of undefined (reading 'allow')`, add `pnpm.overrides`:

```json
{ "pnpm": { "overrides": { "eslint": "^9.0.0" } } }
```

### Dependency Version Conflicts

After import, compare key deps (`typescript`, `eslint`, framework-specific). If dest uses newer versions, upgrade imported packages to match (usually safe). If source is newer, may need to upgrade dest first. Use `pnpm.overrides` to enforce single-version policy if desired.

### Module Boundaries

Imported projects may lack `tags`. Add tags or update `@nx/enforce-module-boundaries` rules.

### Project Name Collisions (Multi-Import)

Same `name` in `package.json` across source and dest causes `MultipleProjectsWithSameNameError`. **Fix**: Rename conflicting names (e.g. `@org/api` → `@org/teama-api`), update all dep references and import statements, `pnpm install`. The root `package.json` of each imported repo also becomes a project — rename those too.

### Workspace Dep Import Ordering

`pnpm install` fails during `nx import` if a `"workspace:*"` dependency hasn't been imported yet. File operations still succeed. **Fix**: Import all projects first, then `pnpm install --no-frozen-lockfile`.

### `.gitkeep` Blocking Subdirectory Import

The TS preset creates `packages/.gitkeep`. Remove it and commit before importing.

### Frontend tsconfig Base Settings (Critical)

The TS preset defaults (`module: "nodenext"`, `moduleResolution: "nodenext"`, `lib: ["es2022"]`) are incompatible with frontend frameworks (React, Next.js, Vue, Vite). After importing frontend projects, verify the dest root `tsconfig.base.json`:

- **`moduleResolution`**: Must be `"bundler"` (not `"nodenext"`)
- **`module`**: Must be `"esnext"` (not `"nodenext"`)
- **`lib`**: Must include `"dom"` and `"dom.iterable"` (frontend projects need these)
- **`jsx`**: `"react-jsx"` for React-only workspaces, per-project for mixed frameworks

For **subdirectory imports**, the dest root tsconfig is authoritative — update it. For **whole-repo imports**, imported projects may extend their own nested `tsconfig.base.json`, making this less critical.

If the dest also has backend projects needing `nodenext`, use per-project overrides instead of changing the root.

**Gotcha**: TypeScript does NOT merge `lib` arrays — a project-level override **replaces** the base array entirely. Always include all needed entries (e.g. `es2022`, `dom`, `dom.iterable`) in any project-level `lib`.

### `@nx/react` Typings for Libraries

React libraries generated with `@nx/react:library` reference `@nx/react/typings/cssmodule.d.ts` and `@nx/react/typings/image.d.ts` in their tsconfig `types`. These fail with `Cannot find type definition file` unless `@nx/react` is installed in the dest workspace.

**Fix**: `pnpm add -wD @nx/react`

### Jest Preset Missing (Subdirectory Import)

Nx presets create `jest.preset.js` at the workspace root, and project jest configs reference it (e.g. `../../jest.preset.js`). Subdirectory import does NOT bring this file.

**Fix**:

1. Run `npx nx add @nx/jest` — registers `@nx/jest/plugin` in `nx.json` and updates `namedInputs`
2. Create `jest.preset.js` at workspace root (use the Jest guidance in this section) — `nx add` only creates this when a generator runs, not on bare `nx add`
3. Install test runner deps: `pnpm add -wD jest jest-environment-jsdom ts-jest @types/jest`
4. Install framework-specific test deps as needed based on your framework stack

For deeper Jest issues (tsconfig.spec.json, Babel transforms, CI atomization, Jest vs Vitest coexistence), keep project-specific notes close to this section.

### Target Name Prefixing (Whole-Repo Import)

When importing a project with existing npm scripts (`build`, `dev`, `start`, `lint`), Nx plugins auto-prefix inferred target names to avoid conflicts: e.g. `next:build`, `vite:build`, `eslint:lint`.

**Fix**: Remove the Nx-rewritten npm scripts from the imported `package.json`, then either:

- Accept the prefixed names (e.g. `nx run app:next:build`)
- Rename plugin target names in `nx.json` to use unprefixed names

## Non-Nx Source Issues

When the source is a plain pnpm/npm workspace without `nx.json`.

### npm Script Rewriting (Critical)

Nx rewrites `package.json` scripts during init, creating broken commands (e.g. `vitest run` → `nx test run`). **Fix**: Remove all rewritten scripts — Nx plugins infer targets from config files.

### `noEmit` → `composite` + `emitDeclarationOnly` (Critical)

Plain TS projects use `"noEmit": true`, incompatible with Nx project references.

**Symptoms**: "typecheck target is disabled because one or more project references set 'noEmit: true'" or TS6310.

**Fix** in **all** imported tsconfigs:

1. Remove `"noEmit": true`. If inherited via extends chain, set `"noEmit": false` explicitly.
2. Add `"composite": true`, `"emitDeclarationOnly": true`, `"declarationMap": true`
3. Add `"outDir": "dist"` and `"tsBuildInfoFile": "dist/tsconfig.tsbuildinfo"`
4. Add `"extends": "../../tsconfig.base.json"` if missing. Remove settings now inherited from base.

### Stale node_modules and Lockfiles

`nx import` may bring `node_modules/` (pnpm symlinks pointing to the source filesystem) and `pnpm-lock.yaml` from the source. Both are stale.

**Fix**: `rm -rf imported/node_modules imported/pnpm-lock.yaml imported/pnpm-workspace.yaml imported/.gitignore`, then `pnpm install`.

### ESLint Config Handling

- **Legacy `.eslintrc.json` (ESLint 8)**: Delete all `.eslintrc.*`, remove v8 deps, create flat `eslint.config.mjs`.
- **Flat config (`eslint.config.js`)**: Self-contained configs can often be left as-is.
- **No ESLint**: Create both root and project-level configs from scratch.

### TypeScript `paths` Aliases

Nx uses `package.json` `"exports"` + pnpm workspace linking instead of tsconfig `"paths"`. If packages have proper `"exports"`, paths are redundant. Otherwise, update paths for the new directory structure.

## Technology-specific Guidance

Identify technologies in the source repo and keep guidance directly in this section to avoid broken cross-file links.

<!-- End: nx-import -->

---

<!-- Begin: nx-workspace -->

---
name: nx-workspace
description: "Explore and understand Nx workspaces. USE WHEN answering questions about the workspace, projects, or tasks. ALSO USE WHEN an nx command fails or you need to check available targets/configuration before running a task. EXAMPLES: 'What projects are in this workspace?', 'How is project X configured?', 'What depends on library Y?', 'What targets can I run?', 'Cannot find configuration for task', 'debug nx task failure'."
---

# Nx Workspace Exploration

This skill provides read-only exploration of Nx workspaces. Use it to understand workspace structure, project configuration, available targets, and dependencies.

Keep in mind that you might have to prefix commands with `npx`/`pnpx`/`yarn` if nx isn't installed globally. Check the lockfile to determine the package manager in use.

## Listing Projects

Use `nx show projects` to list projects in the workspace.

The project filtering syntax (`-p`/`--projects`) works across many Nx commands including `nx run-many`, `nx release`, `nx show projects`, and more. Filters support explicit names, glob patterns, tag references (e.g. `tag:name`), directories, and negation (e.g. `!project-name`).

```bash
# List all projects
nx show projects

# Filter by pattern (glob)
nx show projects --projects "apps/*"
nx show projects --projects "shared-*"

# Filter by tag
nx show projects --projects "tag:publishable"
nx show projects -p 'tag:publishable,!tag:internal'

# Filter by target (projects that have a specific target)
nx show projects --withTarget build
nx show projects --withTarget e2e
```

## Project Configuration

Use `nx show project <name> --json` to get the full resolved configuration for a project.

**Important**: Do NOT read `project.json` directly - it only contains partial configuration. The `nx show project --json` command returns the full resolved config including inferred targets from plugins.

You can read the full project schema at `node_modules/nx/schemas/project-schema.json` to understand nx project configuration options.

```bash
# Get full project configuration
nx show project my-app --json

# Extract specific parts from the JSON
nx show project my-app --json | jq '.targets'
nx show project my-app --json | jq '.targets.build'
nx show project my-app --json | jq '.targets | keys'

# Check project metadata
nx show project my-app --json | jq '{name, root, sourceRoot, projectType, tags}'
```

## Target Information

Targets define what tasks can be run on a project.

```bash
# List all targets for a project
nx show project my-app --json | jq '.targets | keys'

# Get full target configuration
nx show project my-app --json | jq '.targets.build'

# Check target executor/command
nx show project my-app --json | jq '.targets.build.executor'
nx show project my-app --json | jq '.targets.build.command'

# View target options
nx show project my-app --json | jq '.targets.build.options'

# Check target inputs/outputs (for caching)
nx show project my-app --json | jq '.targets.build.inputs'
nx show project my-app --json | jq '.targets.build.outputs'

# Find projects with a specific target
nx show projects --withTarget serve
nx show projects --withTarget e2e
```

## Workspace Configuration

Read `nx.json` directly for workspace-level configuration.
You can read the full project schema at `node_modules/nx/schemas/nx-schema.json` to understand nx project configuration options.

```bash
# Read the full nx.json
cat nx.json

# Or use jq for specific sections
cat nx.json | jq '.targetDefaults'
cat nx.json | jq '.namedInputs'
cat nx.json | jq '.plugins'
cat nx.json | jq '.generators'
```

Key nx.json sections:

- `targetDefaults` - Default configuration applied to all targets of a given name
- `namedInputs` - Reusable input definitions for caching
- `plugins` - Nx plugins and their configuration
- ...and much more, read the schema or nx.json for details

## Affected Projects

If the user is asking about affected projects, use the `nx show projects --affected --json` and `nx affected -t <target>` commands documented in this section.

## Common Exploration Patterns

### "What's in this workspace?"

```bash
nx show projects
nx show projects --type app
nx show projects --type lib
```

### "How do I build/test/lint project X?"

```bash
nx show project X --json | jq '.targets | keys'
nx show project X --json | jq '.targets.build'
```

### "What depends on library Y?"

```bash
# Use the project graph to find dependents
nx graph --print | jq '.graph.dependencies | to_entries[] | select(.value[].target == "Y") | .key'
```

## Programmatic Answers

When processing nx CLI results, use command-line tools to compute the answer programmatically rather than counting or parsing output manually. Always use `--json` flags to get structured output that can be processed with `jq`, `grep`, or other tools you have installed locally.

### Listing Projects

```bash
nx show projects --json
```

Example output:

```json
["my-app", "my-app-e2e", "shared-ui", "shared-utils", "api"]
```

Common operations:

```bash
# Count projects
nx show projects --json | jq 'length'

# Filter by pattern
nx show projects --json | jq '.[] | select(startswith("shared-"))'

# Get affected projects as array
nx show projects --affected --json | jq '.'
```

### Project Details

```bash
nx show project my-app --json
```

Example output:

```json
{
  "root": "apps/my-app",
  "name": "my-app",
  "sourceRoot": "apps/my-app/src",
  "projectType": "application",
  "tags": ["type:app", "scope:client"],
  "targets": {
    "build": {
      "executor": "@nx/vite:build",
      "options": { "outputPath": "dist/apps/my-app" }
    },
    "serve": {
      "executor": "@nx/vite:dev-server",
      "options": { "buildTarget": "my-app:build" }
    },
    "test": {
      "executor": "@nx/vite:test",
      "options": {}
    }
  },
  "implicitDependencies": []
}
```

Common operations:

```bash
# Get target names
nx show project my-app --json | jq '.targets | keys'

# Get specific target config
nx show project my-app --json | jq '.targets.build'

# Get tags
nx show project my-app --json | jq '.tags'

# Get project root
nx show project my-app --json | jq -r '.root'
```

### Project Graph

```bash
nx graph --print
```

Example output:

```json
{
  "graph": {
    "nodes": {
      "my-app": {
        "name": "my-app",
        "type": "app",
        "data": { "root": "apps/my-app", "tags": ["type:app"] }
      },
      "shared-ui": {
        "name": "shared-ui",
        "type": "lib",
        "data": { "root": "libs/shared-ui", "tags": ["type:ui"] }
      }
    },
    "dependencies": {
      "my-app": [
        { "source": "my-app", "target": "shared-ui", "type": "static" }
      ],
      "shared-ui": []
    }
  }
}
```

Common operations:

```bash
# Get all project names from graph
nx graph --print | jq '.graph.nodes | keys'

# Find dependencies of a project
nx graph --print | jq '.graph.dependencies["my-app"]'

# Find projects that depend on a library
nx graph --print | jq '.graph.dependencies | to_entries[] | select(.value[].target == "shared-ui") | .key'
```

## Troubleshooting

### "Cannot find configuration for task X:target"

```bash
# Check what targets exist on the project
nx show project X --json | jq '.targets | keys'

# Check if any projects have that target
nx show projects --withTarget target
```

### "The workspace is out of sync"

```bash
nx sync
nx reset  # if sync doesn't fix stale cache
```

<!-- End: nx-workspace -->

