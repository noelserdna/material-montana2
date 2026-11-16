# Derived Specifications

| Metric | Value | Threshold | Status |
|---|---|---|---|
| Tier 1 pending REQ | 3 | ≤ 3 | PASS |
| Tier 1 accepted without REQ | 0 | — | Advisory |
| Tier 2 registered | 5 | — | Info |

## Spec-engineer derived

| Artifact | Type | Derived from | Tier | Justification (≤ 8 words) | Status |
|---|---|---|---|---|---|
| INV-ACC-001..007, INV-RES-001..005, INV-FIN-001..003, INV-INV-001..002 | Invariant | REQ-F-001..019 | 2 | formalises requirement constraints | Registered |
| `E_*` rows in UC-001..019 | Exception flow | REQ-F-001..020 | 2 | validation, permission, state errors | Registered |
| `E_STORAGE_UNAVAILABLE`, `E_AUTH_REQUIRED` | Exception flow | REQ-NF-001 | 2 | shared failures | Registered |
| `E_RES_NOT_FOUND`, `E_MATERIAL_NOT_FOUND`, `E_FINE_NOT_FOUND` | Exception flow | REQ-F-005, 006, 010, 012 | 2 | unknown ids | Registered |
| `O_ALREADY_*` outcomes | Replay rows | REQ-F-005..007, 010 | 2 | idempotent writes | Registered |
| El encargado también reserva y ve Mis reservas | Rule | REQ-F-002, REQ-F-004 | 1 | requisitos solo dicen "socio" | **[PENDING REQ]** |
| Una unidad Entregada y sin devolver tras `hasta` no ocupa días posteriores | Rule | REQ-F-002, REQ-F-007 | 1 | disponibilidad solo por rango | **[PENDING REQ]** |
| Tras una baja puede haber más reservas que unidades en un día; el encargado avisa a mano | Rule | REQ-F-012 | 1 | RN-002 por el cliente, sin aviso en el sistema | **[PENDING REQ]** |

## Audit derived
None.

## Resolution log
None.
