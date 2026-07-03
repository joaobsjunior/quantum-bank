## Context

Quantum Bank uses OpenSpec for project planning and change control. The
repository still contained parallel planning artifacts and process notes, which
could lead contributors or agents to follow a non-OpenSpec process.

## Goals / Non-Goals

**Goals:**
- Make `openspec/` the only planning artifact tree in the superproject.
- Remove non-OpenSpec planning files and references from contributor-facing docs.
- Preserve the product specs already represented in OpenSpec.

**Non-Goals:**
- Change product requirements, APIs, runtime services, or submodule pointers.
- Rework OpenSpec-generated Codex skills.
- Delete implementation contracts inside submodules.

## Decisions

- Keep baseline product requirements in `openspec/specs/` instead of preserving
  migration-only notes.
  Rationale: OpenSpec should be directly actionable without requiring readers to
  understand the prior planning tool.

- Remove the parallel planning artifact tree entirely instead of marking it as
  archived.
  Rationale: The user explicitly wants planning through OpenSpec, and keeping a
  parallel planning tree creates ambiguity.

- Add `planning-governance` as a base capability.
  Rationale: The planning rule is now a project contract and should be
  validated through OpenSpec like other requirements.

## Risks / Trade-offs

- Earlier planning detail is no longer present in the repository.
  Mitigation: All active product requirements were already represented in
  OpenSpec baseline specs before removing non-OpenSpec artifacts.

- The cleanup touches repository governance docs instead of runtime code.
  Mitigation: Validate OpenSpec strictly and keep runtime files unchanged.
