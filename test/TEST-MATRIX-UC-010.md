# Test Matrix: UC-010 — Marcar la multa como pagada

> Refs: UC-010, API-004-02, BDD-UC-010 (AC-010-01..05), INV-FIN-003, INV-ACC-005, INV-ACC-003, SM-002
> Techniques: EP, BVA, decision table, state transition, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión/rol | Rol | encargado (marta) | socio (lucia), junta (ana), sin sesión | — |
| multaId | number | id de multa Pendiente; id de multa Pagada | id inexistente | — |
| multas del socio | estado | 1 Pendiente; 2 Pendientes (pablo, 2 € y 4 €) | — | 1 vs 2 pendientes (INV-FIN-003) |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · única multa de lucia 6 € Pendiente · marcar pagada | ok · "Multa de 6 € pagada", estado `Pagada` · lucia puede reservar "Piolet" | happy | AC-010-01, REQ-F-010 AC1 |
| T02 | marta · pablo con multas 2 € y 4 € · marcar la de 2 € | ok · "Multa de 2 € pagada" · la de 4 € sigue `Pendiente`; pablo reservar → `E_FINE_PENDING` | boundary | AC-010-02, INV-FIN-003 |
| T03 | marta · multa ya `Pagada` · marcar otra vez | `O_ALREADY_PAID` · "Ya estaba pagada" · sigue `Pagada` una sola vez | replay | AC-010-03 |
| T04 | lucia (socio) · su multa Pendiente · marcar pagada | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sigue `Pendiente` | error | AC-010-04, INV-ACC-005 |
| T05 | ana (junta) · multa Pendiente · marcar pagada | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sigue `Pendiente` | error | AC-010-04, INV-ACC-005 |
| T06 | marta · multaId inexistente | `E_FINE_NOT_FOUND` · "La multa no existe" · sin cambios | error | AC-010-05 |
| T07 | sin sesión · marcar pagada | `E_AUTH_REQUIRED` · "Iniciar sesión" · multa sin cambios | derived | INV-ACC-003 |
| T08 | rol socio + multaId inexistente | undefined in spec (precedencia rol vs existencia; UC dice «comprueba rol y existencia») | error | — |
| T09 | dos encargados marcan a la vez la misma multa Pendiente | undefined in spec (GAP-005) | race | — |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| SM-002 | Pendiente | UC-010 · rol encargado | Pagada | otras multas intactas; sin pendientes → puede reservar | T01, T02 |
| SM-002 | Pendiente | UC-010 · rol socio/junta (inválida) | Pendiente | sin cambios | T04, T05 |
| SM-002 | Pagada | UC-010 · replay (inválida/idempotente) | Pagada | `O_ALREADY_PAID`, sin cambios | T03 |
| SM-002 | Pagada | volver a Pendiente (no existe transición) | — | undefined in spec: no hay operación | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-010-01 | Precedencia rol vs multa inexistente no definida (T08) | Fijar orden de comprobaciones en UC-010 |
| F-010-02 | Pago simultáneo por dos encargados sin AC (GAP-005) | Definir resultado (una pagada, otra `O_ALREADY_PAID`) |
