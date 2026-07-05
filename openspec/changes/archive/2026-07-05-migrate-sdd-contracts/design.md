## Context

The repository carries two spec-driven-development trees:

1. `openspec/specs/` — nine OpenSpec capability specs (the declared source of
   truth via `planning-governance`).
2. `*/docs/contracts/*.md` — ten submodule contract files with a separate
   requirement-ID vocabulary and much of the real implementation detail.

The two trees were never reconciled: the contract requirement IDs
(`CONT-*`, `AUTH-*`, `PKI-*`, `MTLS-*`, `D-*`) appear nowhere under `openspec/`.

## Goals

- Make OpenSpec the single, authoritative SDD source in fact.
- Preserve every substantive, testable requirement the contracts encode.
- Avoid inventing new behavior — migrate, do not redesign.

## Non-Goals

- No change to application code, gateway config, or the OpenAPI contract.
- No renumbering of the existing OpenSpec requirements.
- No attempt to preserve the `CONT-*/D-*` identifier scheme; OpenSpec identifies
  requirements by name, so the identifiers are dropped and their intent is
  carried by named requirements and scenarios.

## Decisions

### Migrate as ADDED requirements, not MODIFIED

The existing OpenSpec requirements are correct but thin; the contracts add
detail rather than contradict. Each migrated contract concern becomes a new,
narrowly-named `ADDED` requirement in the most relevant existing spec. This keeps
the delta specs self-contained (no need to restate whole existing requirements)
and avoids accidental semantic drift in the requirements already shipped.

### Contract-to-spec routing

| Contract file | Target spec |
| --- | --- |
| `api-gateway/gateway-boundary.md` | secure-gateway-communication |
| `api-gateway/oauth2-gateway-policy.md` | secure-gateway-communication |
| `backend/oauth2-backend-policy.md` | secure-gateway-communication |
| `backend/otk-csr-contract.md` | onboarding-pki |
| `pki/certificate-lifecycle.md` | onboarding-pki |
| `mobile-app/client-bootstrap.md` | onboarding-pki |
| `backend/api-implementation-map.md` | backend-simulation-api |
| `mobile-app/api-client-contract.md` | mobile-banking-journeys |
| `infrastructure/local-runtime-checklist.md` | deployment-infrastructure |
| `infrastructure/oauth2-local-issuer.md` | deployment-infrastructure |

The OTK/CSR bootstrap concerns are consolidated in `onboarding-pki` even when
they span backend, PKI, and mobile ownership, because the spec captures the
end-to-end onboarding behavior rather than per-repo ownership. Ownership is
recorded in the requirement text where it matters (who validates, who issues).

### Concrete local values are captured as scenarios, not hard requirements

Environment-specific values (Keycloak on `8180`, gateway listeners `8080`/`8443`,
5-minute OTK TTL, profile `quantum-bank-mobile-client-v1`) are local-v1
defaults. They are migrated as the current expected values but framed so that
configuration may override them, matching how the contracts phrased them.

### Deletion is part of this change

Because `planning-governance` forbids a parallel planning tree, leaving the
contracts in place after migration would immediately violate that spec. The
tasks therefore include deleting the ten files once the specs carry their
content, and repointing any README references to the OpenSpec specs.
