## Why

The project planning source of truth must be OpenSpec only. Keeping parallel
planning files or process notes beside OpenSpec creates ambiguity about which
process agents and contributors should follow.

## What Changes

- Remove non-OpenSpec planning artifacts from the repository.
- Remove references to non-OpenSpec planning workflows from project-facing
  instructions and documentation.
- Add an explicit OpenSpec planning governance capability so future work keeps
  planning artifacts under `openspec/`.

## Capabilities

### New Capabilities
- `planning-governance`: Defines OpenSpec as the project planning source of
  truth and disallows non-OpenSpec planning artifacts.

### Modified Capabilities
- None.

## Impact

- Removes non-OpenSpec planning artifacts and migration-only OpenSpec notes.
- Updates `AGENTS.md`, `README.md`, `openspec/project.md`, and
  `openspec/specs/secure-gateway-communication/spec.md` to avoid non-OpenSpec
  planning references.
- Does not change runtime code, APIs, submodule pointers, or deployment
  behavior.
