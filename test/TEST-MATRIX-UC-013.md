# Test Matrix: UC-013 — Ver lo prestado y las devoluciones de cada día

> Refs: UC-013, API-002-06, BDD-UC-013 (AC-013-01..06), INV-RES-005, INV-ACC-005, INV-ACC-003
> Techniques: EP, BVA, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión/rol | Rol | encargado (marta) | socio, junta, sin sesión | — |
| dia | Dia (VO-003) opcional | omitido (= hoy 2026-12-01); día con devoluciones; día sin ninguna | formato inválido (undefined in spec); pasado (GAP-003) | hasta = dia / dia±1 |
| reservas | estado | Entregada; Reservada/Devuelta/Cancelada (no prestadas) | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · Casco de lucia Entregada hasta 2026-12-12, Cuerda de pablo Entregada hasta 2026-12-13 · sin `dia` | "Prestado: Casco (lucia@club.es)", "Prestado: Cuerda (pablo@club.es)" · sin cambios | happy | AC-013-01, REQ-F-013 AC1 |
| T02 | mismo estado · `dia` = 2026-12-12 | "Vuelve: Casco (lucia@club.es)", sin Cuerda · sin cambios | happy | AC-013-02, REQ-F-013 AC2 |
| T03 | nada Entregado · sin `dia` | "No hay nada prestado" · sin cambios | boundary | AC-013-03, REQ-F-013 AC3 |
| T04 | Entregada con hasta = 2026-12-01 · sin `dia` | Vuelve incluye esa reserva (día por defecto = hoy) | derived | UC-013 flujo 1 (D-T-001) |
| T05 | Entregada con hasta = 2026-12-02 · sin `dia` | Prestado la incluye; Vuelve no | boundary | UC-013 postcond. (derived) |
| T06 | solo Reservada, Devuelta y Cancelada · sin `dia` | "No hay nada prestado"; no aparecen en Prestado | state | INV-RES-005, UC-013 postcond. |
| T07 | lucia (socio) · abre Hoy | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin datos | error | AC-013-04, INV-ACC-005 |
| T08 | ana (junta) · abre Hoy | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin datos | error | AC-013-04, INV-ACC-005 |
| T09 | sin sesión · pide Hoy | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos | error | AC-013-05, INV-ACC-003 |
| T10 | marta · abre Hoy dos veces | misma lista las dos veces · sin cambios | replay | AC-013-06 |
| T11 | `dia` = fecha sin devoluciones, con prestados | Prestado lista; Vuelve vacío: texto undefined in spec | boundary | — |
| T12 | `dia` pasado o con formato inválido | undefined in spec (GAP-003) | error | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-013-01 | Sin texto para Vuelve vacío con prestados (T11) | Definir mensaje o lista vacía |
| F-013-02 | `dia` inválido/pasado sin error (T12, GAP-003) | Definir error o aceptar |
