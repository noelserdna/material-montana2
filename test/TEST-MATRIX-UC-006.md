# Test Matrix: UC-006 — Entregar lo reservado

> Refs: UC-006, API-002-04, BDD-UC-006 (AC-006-01..06), INV-RES-005, INV-ACC-005, REQ-NF-001 (AC-NF1-01), SM-001
> Techniques: EP, decision table, state transition, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión (rol) | enum | encargado (marta) | socio (lucia), junta (ana), sin sesión | — |
| reservaId | number | existente | inexistente | — |
| estado de la reserva | SM-001 | Reservada | Cancelada, Entregada (replay), Devuelta | Reservada→Entregada |
| socio dueño | correo | lucia, pablo | — | — |
| almacén compartido | store | lectura entre dos sesiones | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · reserva Reservada "Casco" de lucia 2026-12-10..12 | `Entregada` · "Entregada a lucia@club.es" · estado Entregada | happy | AC-006-01 |
| T02 | marta · reserva Reservada "Cuerda" de pablo | `Entregada` · "Entregada a pablo@club.es" · estado Entregada | happy | AC-006-02 |
| T03 | marta · reserva ya Entregada, entregar otra vez | `O_ALREADY_DELIVERED` · "Ya estaba entregada" · sigue Entregada una vez | replay | AC-006-03, UC-006 E4 |
| T04 | marta · reserva Cancelada | `E_RES_CANCELLED` · "No se puede entregar una reserva cancelada" · sigue Cancelada | error | AC-006-04, INV-RES-005 |
| T05 | lucia (socio) · reserva Reservada | `E_FORBIDDEN_ROLE` · — · reserva sigue Reservada | error | AC-006-05, INV-ACC-005 |
| T06 | ana (junta) · reserva Reservada | `E_FORBIDDEN_ROLE` · — · reserva sigue Reservada | error | AC-006-05, INV-ACC-005 |
| T07 | marta · reservaId inexistente | `E_RES_NOT_FOUND` · — · almacén sin cambios | error | AC-006-06 |
| T08 | lucia reservó "Casco" 2026-12-10..12 en sesión A; marta abre Entregas y devoluciones en sesión B (almacén compartido) | ve "Casco" y "lucia@club.es" · sin escritura | happy | AC-NF1-01, REQ-NF-001 |
| T09 | dos encargados entregan la misma reserva Reservada a la vez | undefined in spec | race | — |
| T10 | sin sesión · reserva Reservada | undefined in spec | error | — |
| T11 | marta · reserva Devuelta, entregar | undefined in spec | state | — |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| SM-001 | Reservada | entregar · encargado | Entregada | unidad cuenta como prestada | T01, T02 |
| SM-001 | Entregada | entregar otra vez | Entregada | sin cambio; `O_ALREADY_DELIVERED` | T03 |
| SM-001 | Cancelada | entregar | Cancelada | sin cambio; `E_RES_CANCELLED` | T04 |
| SM-001 | Reservada | entregar · rol no encargado | Reservada | sin cambio; `E_FORBIDDEN_ROLE` | T05, T06 |
| SM-001 | Devuelta | entregar | — | undefined in spec (GAP-002, SIL-001) | T11 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-006-01 | Entregar una Devuelta sin resultado definido (GAP-002) | Definir error/mensaje |
| F-006-02 | Dos encargados entregando a la vez sin AC (GAP-005) | Definir race de API-002-04 |
| F-006-03 | Sin sesión en API-002-04: `E_AUTH_REQUIRED` no figura en contrato ni UC | Añadir al contrato o aclarar |
