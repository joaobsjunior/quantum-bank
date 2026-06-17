# Workflow Migration

This file migrates `.planning/config.json` into OpenSpec-oriented workflow
guidance.

## Source

- GSD project document: `.planning/PROJECT.md`
- GSD config: `.planning/config.json`
- Migration date: 2026-06-17

## OpenSpec Settings

- Use OpenSpec specs and changes as the source of truth for planning.
- Keep `.planning/` as legacy history unless removal is explicitly requested.
- Use `/opsx:propose` or `openspec new change <change-id>` for new planned work.
- Use `/opsx:apply` for implementation after the change artifacts are ready.
- Use `/opsx:archive` or `openspec archive <change-id>` once work is complete
  and validated.

## Preserved GSD Preferences

- Planning should remain in the superproject.
- Implementation belongs in sub-repositories:
  - `api-gateway`
  - `backend`
  - `infrastructure`
  - `mobile-app`
  - `pki`
- Prefer parallelizable investigation and validation when it does not obscure
  results.
- Keep code review and verification as part of the workflow.
- Do not auto-advance major work without explicit change context.

## Legacy Mapping

- `.planning/PROJECT.md` -> `openspec/project.md` plus capability specs in
  `openspec/specs/`.
- `.planning/config.json` -> this workflow migration note and `AGENTS.md`
  OpenSpec workflow instructions.
- GSD commands such as `/gsd-quick`, `/gsd-debug`, and
  `/gsd-execute-phase` -> OpenSpec proposal/apply/archive flow.
