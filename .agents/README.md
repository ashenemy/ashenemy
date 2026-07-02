# Agents skills canonical README

This directory `.agents/` is the canonical source of truth for agent skills and skill documentation.

Guidelines:
- Canonical long-form skill documentation lives in `.agents/SKILLS.md`.
- Other skill locations (e.g. `.github/skills`, `.opencode/skills`, `.cursor/skills`) are pointers and may be overwritten.
- Keep app/lib short descriptions updated in `.agents/PROJECT-CATALOG.md`.

- Agent runtime artifacts:
  - Canonical rules: `.agents/RULES.md`
  - Session checkpoint (temp): `.agents/state/session.tmp.json`
  - Project app/lib catalog: `.agents/PROJECT-CATALOG.md`

