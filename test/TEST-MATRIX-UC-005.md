# Test Matrix: UC-005 — Cancelar una reserva hasta el día antes

> Refs: UC-005, API-002-03, BDD-UC-005 (AC-005-01..07), INV-RES-004, INV-RES-001, INV-ACC-004, INV-ACC-003, SM-001, GAP-002, GAP-003
> Techniques: EP, BVA, decision table, state transition, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| reservaId | número | reserva existente del actor | inexistente | — |
| Dueño de la reserva | estado | actor = dueño | otro socio | — |
| Rol | actor | socio | encargado, junta, sin sesión | — |
| hoy vs `desde` | fecha | hoy < `desde` | hoy = `desde`; hoy > `desde` (no definido) | `desde`−1, `desde`, `desde`+1 |
| Estado de la reserva (SM-001) | estado | Reservada | Cancelada; Entregada, Devuelta (no definido) | — |
| Unidades del material | estado | stock agotado antes de cancelar | — | n de n ocupadas |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | hoy=2026-12-09 · lucia cancela su "Casco" desde 2026-12-10 | ok · "Reserva cancelada" · estado `Cancelada` | happy | AC-005-01, INV-RES-004 |
| T02 | hoy=2026-12-09 · 3 de 3 "Casco" el 2026-12-10 (una de lucia) · lucia cancela y pablo reserva 2026-12-10 | ok · "Reserva creada: Casco del 2026-12-10 al 2026-12-10" · unidad libre | state | AC-005-02, INV-RES-001 |
| T03 | hoy=2026-12-10 · pablo cancela "Cuerda" desde 2026-12-10 | `E_CANCEL_TOO_LATE` · "Ya no se puede cancelar: la reserva empieza hoy" · sigue `Reservada` | boundary | AC-005-03, INV-RES-004 |
| T04 | lucia ya canceló · la cancela otra vez | `O_ALREADY_CANCELLED` · sigue "Cancelada" · nada más cambia | replay | AC-005-04 |
| T05 | lucia cancela la reserva de pablo | `E_FORBIDDEN_DATA` · "No tienes permiso para ver estos datos" · la de pablo sigue `Reservada` | error | AC-005-05, INV-ACC-004 |
| T06 | ana (junta) intenta cancelar | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin cambios | error | AC-005-06 |
| T07 | lucia cancela reservaId inexistente | `E_RES_NOT_FOUND` · "La reserva no existe" · sin cambios | error | AC-005-07 |
| T08 | dos peticiones simultáneas de lucia sobre la misma reserva | una `Cancelada` ok; la otra `O_ALREADY_CANCELLED` · una sola cancelación · unidad liberada una vez | race | AC-005-04 |
| T09 | hoy=2026-12-09 · reserva desde 2026-12-10 (hoy = `desde`−1) | ok · `Cancelada` · unidad libre | boundary | AC-005-01, INV-RES-004 |
| T10 | hoy=2026-12-11 · reserva desde 2026-12-10 (hoy > `desde`) | undefined in spec (¿`E_CANCEL_TOO_LATE`?) · — | boundary | — |
| T11 | sin sesión cancela | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin cambios | error | derived: INV-ACC-003, VO-010 |
| T12 | marta (encargado) cancela una reserva | `E_FORBIDDEN_ROLE` · sin cambios | error | derived: PERMISSIONS-MATRIX |
| T13 | reserva `Entregada`, hoy < `desde` o hoy ≥ `desde` | undefined in spec · — | state | GAP-002 |
| T14 | reserva `Devuelta` | undefined in spec · — | state | GAP-002 |
| T15 | reserva ajena Y hoy ≥ `desde` | undefined in spec (precedencia dueño vs plazo) · — | derived | — |
| T16 | falla el almacén al cancelar | `E_STORAGE_UNAVAILABLE` · "No se puede acceder a los datos; inténtalo de nuevo" · sigue `Reservada` | error | derived: D-T-002, REQ-NF-001 |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| SM-001 | Reservada | cancelar, dueño, hoy < `desde` | Cancelada | unidad liberada; "Reserva cancelada" | T01, T02, T09 |
| SM-001 | Reservada | cancelar, hoy = `desde` | Reservada | sin cambio; `E_CANCEL_TOO_LATE` | T03 |
| SM-001 | Reservada | cancelar, no dueño / rol no socio | Reservada | sin cambio; `E_FORBIDDEN_*` | T05, T06, T12 |
| SM-001 | Cancelada | cancelar otra vez | Cancelada | se mantiene; `O_ALREADY_CANCELLED` | T04, T08 |
| SM-001 | Entregada | cancelar | undefined in spec | GAP-002 | T13 |
| SM-001 | Devuelta | cancelar | undefined in spec | GAP-002 | T14 |
| SM-001 | Reservada | cancelar, hoy > `desde` | undefined in spec | GAP-003 | T10 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-005-01 | Cancelar Entregada/Devuelta sin definir (GAP-002) | Definir error o transición |
| F-005-02 | hoy > `desde` sin definir (GAP-003); E1 solo cubre hoy ≥ `desde` literal "empieza hoy" | Fijar código y texto para fechas pasadas |
| F-005-03 | Sin precedencia entre `E_FORBIDDEN_DATA` y `E_CANCEL_TOO_LATE` (T15) | Fijar orden de comprobaciones como D-T-004 |
| F-005-04 | UC-005 no cita `E_AUTH_REQUIRED` ni rol encargado; contrato 002-03 tampoco lo lista | Añadir filas E y AC |
