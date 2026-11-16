# Change Report CHG-2026-11-16-001

**Source:** intent/INT-002.md · **Requirements:** v1.0 → v1.1 (`Status: Review`; pending approval by a person — gate de requisitos; new tag `requirements-v1.1`).

## Applied
- CR-001 ADD REQ-F-017 — bloqueo 5 fallos/15 min, 15 min de espera.
- CR-002 ADD REQ-F-018 (Salir), REQ-F-019 (caducidad 30 min).
- Spec propagated: UC-001 v1.1, UC-017, UC-018, BDD-UC-001/017/018, API-acceso, INV-ACC-006/007, errors `E_AUTH_LOCKED`/`E_SESSION_EXPIRED`, glossary, entities, VALUE-REGISTRY, RN-005, SECURITY, PERMISSIONS-MATRIX, DERIVED-SPECS, README; TRACEABILITY-MATRIX regenerated.

## Defaults written without asking (class R)
5 intentos, ventana de 15 min, bloqueo de 15 min, caducidad de 30 min con renovación por actividad, bloqueo por cuenta, mensajes "Demasiados intentos fallidos: espera 15 minutos" y "Tu sesión ha caducado".

## Other answered questions (no requirement change)
Q-spec-auditor-001/002/003 and Q-requirements-engineer-001/002 have option effect `ninguno` (the chosen option is the default already written; RN-001 and RN-002 record the last two). No requirement text changes; the cancel/deliver state errors (001) and history scope (003) belong to a spec-auditor fix pass in `spec/` (UC-005, UC-006, UC-014), still not written there.

## Open / for the cascade
- Approval of v1.1 and customer review of the new examples (pending, a person).
- Stale stages: spec-auditor, test-planner, plan-architect, task-generator, task-implementer, tech-designer, ux-designer, security-auditor. ux/design need the "Salir" button, lockout and expiry messages, session timestamp.
- `lint --needs`: 0 errors; `lint --stale`: 0 findings. No tests quote changed criteria (no code yet). No MODIFY, so no acceptance reopened.
