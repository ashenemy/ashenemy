# Canonical Agent Rules

This file is the canonical merging of `.cursorrules` and `.clinerules`.
Agents should prefer reading this file at session start.

---

<!-- Begin .cursorrules content -->

# START SEEDED CONTEXT: At the very start of ANY new session or chat the AI agent MUST first read `.ai/project_map.md`.
# It is forbidden to generate code or propose solutions without consulting and updating the project map.

# CursorRules for autonomous AI agent operation
# PURPOSE: Machine-readable instructions for automated coding agents interacting with this repository.

# 1) Autonomy and confirmations
autonomy: true
require_confirmation: false

# 2) Language rules
# - Chat and operator messages MUST be in Russian.
# - All produced source code, code comments, UI labels, placeholders, and application logs MUST be in English.
chat_language: ru
code_language: en

# 3) Forbidden outputs
forbidden:
  - "*.spec.ts"
  - "README.md"

# 4) Self-persistence and crash recovery
# Agents MUST write incremental job state to the repo root file `.ai_job_state.json`.
# Always perform atomic write: write to `.ai_job_state.json.tmp` then rename to `.ai_job_state.json`.
state_file: .ai_job_state.json
state_tmp_suffix: .tmp

# 5) Build / generation policy
# - Use Nx generators only. Example:
#   nx g @nx/angular:component my-comp --project=web --skipTests true --style=scss
# - Do not invoke framework specific generators directly (ng g), always use `nx g`.
generation_tool: nx
generation_examples:
  - "nx g @nx/angular:component example --project=web --skipTests true"

# 6) Validation before commit
# Run the repository validation pipeline before committing changes (use workspace package manager: yarn 4.x.x):
#   - yarn nx lint --all
#   - yarn nx build --all --configuration=production
validation_commands:
  - "yarn nx lint --all"
  - "yarn nx build --all --configuration=production"

# 7) Commit policy
# Use internal scoped-commit helper located at `packages/scoped-commit` if present.
# Fallback to: git commit -m "scope: brief description" and follow Conventional Commits with scope.
scoped_commit_tool: packages/scoped-commit || git
commit_message_format: "<scope>(<area>): <short summary>"

# 8) File generation limits
# - Do not generate new top-level README.md files.
# - Do not generate test spec files (*.spec.ts).
# - New files must be minimal and accompanied by Nx target changes when applicable.

# 9) Workspace rules
# - All generated code must respect existing tsconfig paths and workspace boundaries.
# - When adding a new package, add it under `packages/` and register by running package manager install and linking.

# 10) Logging behaviour
# - Log every action to `.ai_job_state.json` (append step with timestamp and outcome).
# - Keep logs concise and machine-readable (JSON entries).

# 11) Examples and safe rollbacks
# - Before any destructive change, create a patch file and store it under `.ai/snapshots/`.
# - Example safe generator + validation sequence:
#   1) yarn nx g @nx/angular:library ui --directory=libs --skipTests true
#   2) yarn nx lint ui
#   3) yarn nx build ui --configuration=production
#   4) packages/scoped-commit commit --message "ui: add ui library"

# 12) Failure protection and restart protocol (MANDATORY)
# Agents MUST follow the strict failure protection protocol below for any non-trivial task.
#  - Before starting any complex task the agent MUST create or open the repository root file
#    `.ai_job_state.json` and record an initial job entry. Use atomic writes (write to .tmp then rename).
#  - Required structure of `.ai_job_state.json` (JSON object) MUST contain at least the following keys:
#    - `jobId` (string) — unique job identifier
#    - `current_task` (string) — short machine-readable description of the current task
#    - `status` (string) — one of: "in_progress", "completed", "failed"
#    - `steps` (array) — ordered array of step objects; each step object MUST contain:
#        - `id` (number)
#        - `name` (string)
#        - `ready` (boolean) — flag indicating whether the step is completed/ready
#        - `ts` (ISO8601 timestamp)
#        - `detail` (string) optional human-readable detail
#    - `last_updated` (ISO8601 timestamp)
#
# Reanimation rule (MANDATORY):
#  - If the agent or environment crashes (IDE freeze, process crash, machine reboot, agent restart),
#    the agent MUST, on first restart, open `.ai_job_state.json`, locate the last step with `ready=false`
#    and continue strictly from that step. The agent MUST NOT restart the job from the beginning unless
#    the job status is explicitly `failed` and an operator confirmed reset (operators are out-of-band).
#  - When resuming, the agent MUST append a new step entry noting the resume attempt and timestamp,
#    and must update `last_updated` and `current_task` accordingly.

<!-- End .cursorrules content -->

---

<!-- Begin .clinerules content -->

# CLI Rules for automated agents
# Machine-oriented specification for CLI-driven AI tasks.

# Run-mode
autonomous: true
confirm_before_action: false

# Language constraints
# - CLI/chat messages: Russian (ru)
# - Generated code, comments, UI labels and app logs: English (en)
chat_language: ru
code_language: en

# Generation constraints
use_nx_generators_only: true
forbidden_patterns:
  - "**/*.spec.ts"
  - "**/README.md"

# Failure protection / persistence
state_file: .ai_job_state.json
atomic_write: true

# Default generator flags (enforced by agent when calling nx g)
default_generator_flags:
  skipTests: true

# Validation steps run automatically after generation (use workspace package manager: yarn 4.x.x)
post_generation:
  - "yarn nx lint --all"
  - "yarn nx build --all --configuration=production"

# Commit integration
# Preferred: internal scoped commit tool at packages/scoped-commit
commit_tool: packages/scoped-commit
fallback_commit: git

<!-- End .clinerules content -->

