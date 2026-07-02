# Canonical Agent Rules

This file is the canonical single source of agent runtime rules for this repository.
Agents should prefer reading this file at session start.

---

<!-- Begin .cursorrules content -->

# START SEEDED CONTEXT: At the very start of ANY new session or chat the AI agent MUST first read `.agents/PROJECT-CATALOG.md`.
# It is forbidden to generate code or propose solutions without consulting the current apps/libs catalog.

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
# Agents MUST keep a lightweight session checkpoint in `.agents/state/session.tmp.json`.
# Always perform atomic write: write to `.agents/state/session.tmp.json.tmp` then rename.
state_file: .agents/state/session.tmp.json
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
# Use git commit with clear scoped Conventional Commit messages.
scoped_commit_tool: git
commit_message_format: "<scope>(<area>): <short summary>"

# 8) File generation limits
# - Do not generate new top-level README.md files.
# - Do not generate test spec files (*.spec.ts).
# - New files must be minimal and accompanied by Nx target changes when applicable.

# 9) Workspace rules
# - All generated code must respect existing tsconfig paths and workspace boundaries.
# - When adding a new package, add it under `packages/` and register by running package manager install and linking.

# 10) Logging behaviour
# - No long action history is required.
# - Update only the session checkpoint with current and pending subtasks.

# 11) Examples and safe rollbacks
# - Before any destructive change, create a patch file and store it under `.agents/snapshots/`.
# - Example safe generator + validation sequence:
#   1) yarn nx g @nx/angular:library ui --directory=libs --skipTests true
#   2) yarn nx lint ui
#   3) yarn nx build ui --configuration=production
#   4) git commit -m "ui: add ui library"

# 12) Failure protection and restart protocol (MANDATORY)
# Agents MUST follow the strict restart protocol below for any non-trivial task.
#  - Before starting any complex task, create or open `.agents/state/session.tmp.json`.
#  - Required structure of `.agents/state/session.tmp.json`:
#    - `sessionId` (string)
#    - `current_subtask` (string)
#    - `pending_subtasks` (array of strings)
#    - `last_updated` (ISO8601 timestamp)
#
# Reanimation rule (MANDATORY):
#  - On restart after crash/reboot, agent MUST read `.agents/state/session.tmp.json`
#    and continue from `current_subtask`, then proceed through `pending_subtasks` in order.
#  - At successful session end, agent MAY clear this temp file.

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
state_file: .agents/state/session.tmp.json
atomic_write: true

# Default generator flags (enforced by agent when calling nx g)
default_generator_flags:
  skipTests: true

# Validation steps run automatically after generation (use workspace package manager: yarn 4.x.x)
post_generation:
  - "yarn nx lint --all"
  - "yarn nx build --all --configuration=production"

# Commit integration
# Use git commit with explicit scoped message.
commit_tool: git
fallback_commit: git

<!-- End .clinerules content -->


