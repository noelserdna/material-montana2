# Change Report CHG-2026-11-16-002

**Source:** intent/INT-003.md · **Requirements:** v1.1 → v1.2 (`Status: Review`; pending approval by a person — gate de requisitos; new tag `requirements-v1.2`).

## Applied
- CR-003 ADD REQ-F-020 — lista de reservas de todos los socios para el encargado (Need N-005), filtro Por entregar / Entregadas / Todas.
- Spec: UC-019, BDD-UC-019, API-002-07 listarReservas, PERMISSIONS-MATRIX, INV-ACC-004/005, VO error catalog, README, DERIVED-SPECS; TRACEABILITY-MATRIX regenerated. Header approval lines removed from REQUIREMENTS.md until v1.2 is approved.

## Defaults written without asking (class R)
Filtro por defecto "Por entregar"; "Todas" incluye Devuelta y Cancelada; texto "No hay reservas"; la junta y el socio quedan rechazados (`E_FORBIDDEN_ROLE`).

## Other decisions in the request
Q-spec-auditor-001..004 and Q-requirements-engineer-001/002: already reflected in v1.1 (see CHG-2026-11-16-001); no further requirement change.

## Open / for the cascade
- Approval of v1.2 and customer review of REQ-F-020 examples (a person).
- Stale: spec-auditor, test-planner, plan-architect, task-generator, task-implementer, tech-designer, ux-designer. SCR-004 (UC-019, REQ-F-020), OPERATION-MAPPING and FASE-2 task TASK-F2-005 (the [PLAN GAP]) need regeneration.
- No MODIFY, so no acceptance reopened. `lint --needs`: 0 errors; `lint --stale`: 0 findings.
