<!-- Agents canonical policy file -->

# Agents: Policies and Canonical Context

This document contains canonical rules and policies for automated agents operating in this repository. It is the primary seeded context that agents should consult before acting.

## 1. Rules & Policies

- Always operate with the principle of least surprise: do not modify unrelated files without explicit user consent.
- Prefer non-destructive actions: create commits with minimal changes and clear commit messages; avoid `git add -A` or committing unscoped changes.
- Never expose secrets or private data. If a task requires credentials or secrets, prompt the user and abort automated actions until explicit approval is given.
- Run autonomously for user-requested scope; do not ask for extra confirmations unless the request is ambiguous or potentially destructive beyond stated scope.

## 2. START SEEDED CONTEXT

Agents should begin with the following seeded context available in-memory for decision-making:

- Workspace root path and a list of top-level projects (from `nx.json` or package manifests).
- Current git branch and most recent commit SHA.
- Presence of Nx Cloud indicators (`nx.json` `nxCloudId` or `nxCloudAccessToken`).
- Known package manager by lockfile or `package.json.packageManager`.
- Location of agent assets in the repository: `.agents/` (skills, rules, state).
- Current app/lib short descriptions from `.agents/PROJECT-CATALOG.md`.

This context should be refreshed where appropriate before taking actions that depend on it.

## 3. State Persistence Protocol

- Long action history is not required.
- For crash recovery, use a temporary session checkpoint file: `.agents/state/session.tmp.json`.
- Required checkpoint fields: `sessionId`, `current_subtask`, `pending_subtasks`, `last_updated`.
- Keep checkpoint data minimal and temporary; do not store secrets.

## 4. Language Policy

- Human-facing messages (CLI output, PR descriptions, inline comments) must be in the user's preferred language. The user of this workspace prefers Russian for conversations and English for repository configuration files.
  - Repository configuration files, docs, and generated skill/config files SHOULD be written in English.
  - Chat interactions, prompts, and step-by-step explanations SHOULD be in Russian.

## 5. Generation / Validation / Commit Policy

- When generating code or configuration: produce minimal, focused changes and include a short rationale and verification commands.
- Always run static validation where available (linters, typecheck, unit tests) before committing generated code. If running tests is expensive, run a targeted subset relevant to the change.
- Commit messages must be explicit and scoped. Use the format:

  `agent(<scope>): <short description>\n\n<one-line summary of files changed>\n\n<verification commands run>`

- If automated fixes are applied (e.g., via scripts or tools), include a short before/after summary in commit message body.

## 6. Centralization under .agents/

- All agent runtime assets (skills, deterministic scripts, reference docs, and shared helpers) SHOULD be located under the `.agents/` directory. Agents MUST prefer `.agents/` for reads and writes to avoid scattering files across the repository.

## 7. Safety & Escalation

- If an agent encounters ambiguity or a potentially destructive operation, it must escalate to the user and pause.
- Provide clear remediation steps when an attempted automated fix fails. Include exact commands users can run to reproduce the attempted steps.

---

If multiple skill manifests exist in the repository, `.agents/` is the authoritative location. Consolidate existing skill files and references into `.agents/` and remove duplicates.

