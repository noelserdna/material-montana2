# Test Matrix: UC-007 — Marcar la devolución

> Refs: UC-007, API-002-05, BDD-UC-007 (AC-007-01..06), INV-RES-005, INV-FIN-002, INV-ACC-005, RN-004, SM-001
> Techniques: EP, BVA, decision table, state transition, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión (rol) | enum | encargado (marta) | socio, junta, sin sesión | — |
| reservaId | number | existente | inexistente | — |
| estado de la reserva | SM-001 | Entregada | Reservada, Cancelada, Devuelta (replay) | Entregada→Devuelta |
| devueltaEl − hasta (días) | int | ≤ 0 sin multa; > 0 con multa | `devueltaEl` < `desde` o futuro (undefined) | -1, 0, 1, 3 |
| stock de la unidad | int | libre tras devolver | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · "Casco" de lucia Entregada, hasta 2026-12-12 · `devueltaEl`=2026-12-12 | `Devuelta` · "Devuelta el 2026-12-12 sin retraso" · `devueltaEl` guardado, sin multa | happy | AC-007-01 |
| T02 | marta · Entregada · devolver dos veces | `O_ALREADY_RETURNED` · "Ya estaba devuelta" · un solo registro, sin segunda multa | replay | AC-007-02, INV-FIN-002, UC-007 E4 |
| T03 | marta · reserva Reservada (nunca entregada) | `E_NOT_DELIVERED` · "Solo se puede devolver lo que se ha entregado" · sigue Reservada | error | AC-007-03, INV-RES-005 |
| T04 | marta · reserva Cancelada | `E_NOT_DELIVERED` · "Solo se puede devolver lo que se ha entregado" · sigue Cancelada | error | AC-007-03, UC-007 E2 |
| T05 | 1 unidad "Casco" entregada y devuelta; pablo reserva días posteriores | se crea la reserva · unidad libre | state | AC-007-04 |
| T06 | lucia (socio) · reserva Entregada | `E_FORBIDDEN_ROLE` · — · sigue Entregada | error | AC-007-05, INV-ACC-005 |
| T07 | ana (junta) · reserva Entregada | `E_FORBIDDEN_ROLE` · — · sigue Entregada | error | AC-007-05, INV-ACC-005 |
| T08 | marta · reservaId inexistente | `E_RES_NOT_FOUND` · — · sin cambios | error | AC-007-06 |
| T09 | expand: BVA(`devueltaEl − hasta` = -1, 0, 1, 3 con FINE_RATE_EUR=2) -1,0 → "Devuelta el {fecha} sin retraso", sin multa; 1 → multa 2 €; 3 → multa 6 € | `Devuelta` · mensaje según UC-008 · multa Pendiente solo si > 0 | boundary | AC-007-01, AC-008-01..03, RN-004 |
| T10 | falla el almacén al devolver con retraso | `E_STORAGE_UNAVAILABLE` · "No se puede acceder a los datos; inténtalo de nuevo" · sigue Entregada, sin multa | error | AC-008-05 |
| T11 | dos encargados devuelven la misma reserva a la vez | undefined in spec | race | — |
| T12 | `devueltaEl` < `desde` o posterior a hoy | undefined in spec | boundary | — |
| T13 | sin sesión · reserva Entregada | undefined in spec | error | — |
| T14 | marta · Devuelta con `devueltaEl` distinto, devolver otra vez | `O_ALREADY_RETURNED` · "Ya estaba devuelta" · `devueltaEl` original conservado | replay | AC-007-02, INV-FIN-002 |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| SM-001 | Entregada | devolver · encargado | Devuelta | guarda `devueltaEl`; unidad libre; UC-008 | T01, T05, T09 |
| SM-001 | Devuelta | devolver otra vez | Devuelta | sin cambio ni segunda multa | T02, T14 |
| SM-001 | Reservada | devolver | Reservada | sin cambio; `E_NOT_DELIVERED` | T03 |
| SM-001 | Cancelada | devolver | Cancelada | sin cambio; `E_NOT_DELIVERED` | T04 |
| SM-001 | Entregada | devolver · rol no encargado | Entregada | sin cambio; `E_FORBIDDEN_ROLE` | T06, T07 |
| SM-002 | — | devueltaEl > hasta | Pendiente | multa creada por UC-008 | T09 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-007-01 | `devueltaEl` < `desde` o futuro sin definir (GAP-003) | Definir validación y error |
| F-007-02 | Dos encargados devolviendo a la vez sin AC (GAP-005) | Definir race de API-002-05 |
| F-007-03 | Sin sesión en API-002-05: `E_AUTH_REQUIRED` no figura en contrato | Añadir al contrato o aclarar |
| F-007-04 | Devolver Cancelada: E2 dice "no estaba Entregada", sin AC propio | Añadir AC o confirmar T04 |
