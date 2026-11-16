# Test Matrix: UC-004 — Ver mis reservas

> Refs: UC-004, API-002-02, BDD-UC-004 (AC-004-01..06), INV-ACC-004, INV-ACC-003, VO-004
> Techniques: EP, BVA, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| Sesión / rol | actor | socio, encargado | sin sesión; junta (sin API-002-02) | — |
| Reservas del actor | estado | 0, 1, varias; estados Reservada/Entregada/Devuelta/Cancelada | — | 0, 1, 2 |
| Reservas de otros socios | estado | existen y no se muestran | petición de las de otro socio | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | lucia: Casco 2026-12-10..2026-12-12 · pablo: Cuerda 2026-12-20..2026-12-21 · lucia abre | ok · "Casco" "2026-12-10" "2026-12-12" "Reservada"; no "Cuerda" · sin cambios | happy | AC-004-01, INV-ACC-004 |
| T02 | mismo estado · pablo abre | ok · "Cuerda" "2026-12-20" "2026-12-21" "Reservada"; no "Casco" · sin cambios | happy | AC-004-02, INV-ACC-004 |
| T03 | socio sin reservas | ok · "Todavía no tienes reservas" · sin cambios | boundary | AC-004-03 |
| T04 | sin sesión | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos | error | AC-004-04, INV-ACC-003 |
| T05 | lucia pide las reservas de pablo | `E_FORBIDDEN_DATA` · "No tienes permiso para ver estos datos" · sin datos de pablo | error | AC-004-05, INV-ACC-004 |
| T06 | lucia abre Mis reservas dos veces | ok · misma lista · sin cambios | replay | AC-004-06 |
| T07 | lucia con reservas en cada estado (Reservada, Entregada, Devuelta, Cancelada) | ok · cada una con su `estado` literal VO-004 · sin cambios | derived | VO-004 |
| T08 | ana (junta) pide Mis reservas | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin datos | error | PERMISSIONS-MATRIX (derived) |
| T09 | marta (encargado) abre Mis reservas, con reservas propias y de socios | ok · solo las propias o todas: undefined in spec · sin cambios | derived | — |
| T10 | lucia con reserva Cancelada por ella · abre | ok · la reserva aparece con "Cancelada" (no se oculta) · sin cambios | derived | REQ-F-004, SM-001 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-004-01 | UC-004 dice "socio (o encargado)" pero no define qué ve el encargado (T09) | Definir: solo las suyas o todas |
| F-004-02 | AC-004-05: API-002-02 no tiene entrada de id; "pide las de pablo" no tiene forma de invocación | Definir parámetro o reformular el AC |
| F-004-03 | Orden de las reservas en la lista no definido | Fijar orden (p. ej. por `desde`) |
