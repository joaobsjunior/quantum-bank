## Purpose

Establish OpenSpec as the single source of truth for planning, specs, and change
control in Quantum Bank, ensuring contributors and agents work from OpenSpec
artifacts and no parallel non-OpenSpec planning trees remain.

## Requirements

### Requirement: OpenSpec is the planning source of truth
The project SHALL use OpenSpec as the source of truth for planning, specs, and change control.

#### Scenario: Contributor starts planned work
- **WHEN** a contributor or agent starts planned work
- **THEN** they inspect or create OpenSpec artifacts under `openspec/`

#### Scenario: Project planning context is needed
- **WHEN** project planning context is needed
- **THEN** `openspec/project.md` and `openspec/specs/` provide the active planning context

### Requirement: Non-OpenSpec planning artifacts are absent
The repository SHALL NOT keep parallel non-OpenSpec planning artifact trees for active project planning.

#### Scenario: Repository planning files are inspected
- **WHEN** the repository planning files are inspected
- **THEN** active planning artifacts are found under `openspec/`
- **AND** no parallel planning artifact tree is present

#### Scenario: Documentation references planning workflow
- **WHEN** contributor-facing documentation references planning workflow
- **THEN** it points to OpenSpec commands and artifacts
- **AND** it does not direct contributors to non-OpenSpec planning commands
