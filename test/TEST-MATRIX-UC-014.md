# Test Matrix: UC-014 — La junta consulta el historial de un socio

> Refs: UC-014, API-004-03, BDD-UC-014 (AC-014-01..05), INV-ACC-003, INV-ACC-004, INV-ACC-005
> Techniques: EP, BVA, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| actor (sesión) | rol | junta (ana), encargado (marta), socio dueño (lucia→lucia) | socio ajeno (lucia→pablo), sin sesión | — |
| correo | Correo (VO-001) | socio existente con préstamos y/o multas | correo sin cuenta; formato inválido | undefined in spec |
| préstamos del socio | lista | ≥1 / ninguno | — | 0, 1 |
| multas del socio | lista | ≥1 (`Pendiente`, `Pagada`) / ninguna ("Sin multas") | — | 0, 1 |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | ana · lucia con "Casco" 2026-12-10..12 devuelto 2026-12-15, multa 6 € Pendiente | ok · "Casco" "2026-12-10" "2026-12-12" y "Multa de 6 € (Pendiente)" · sin cambios | happy | AC-014-01 |
| T02 | ana · pablo con préstamo "Cuerda", sin multas | ok · "Cuerda" y "Sin multas" · sin cambios | boundary | AC-014-02 |
| T03 | lucia (socio) abre historial de pablo | `E_FORBIDDEN_DATA` · "No tienes permiso para ver estos datos" · ningún dato de pablo | error | AC-014-03, INV-ACC-004 |
| T04 | sin sesión pide historial | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos | error | AC-014-04, INV-ACC-003 |
| T05 | ana abre el mismo historial dos veces | ok · mismo resultado las dos veces · sin cambios | replay | AC-014-05 |
| T06 | marta (encargado) · historial de lucia | ok · préstamos y multas de lucia · sin cambios | derived | INV-ACC-004 (rol ∈ {encargado, junta}) |
| T07 | lucia abre su propio historial | ok · sus préstamos y multas · sin cambios | derived | API-004-03 (el socio solo el suyo), INV-ACC-004 |
| T08 | ana · socio con multa `Pagada` | ok · "Multa de {importe} € (Pagada)" | derived | UC-014 §Main flow 2 |
| T09 | ana · socio con reserva `Reservada`/`Entregada`/`Cancelada` | undefined in spec · qué estados son «préstamos» | boundary | — (GAP-004) |
| T10 | ana · correo sin cuenta / formato inválido | undefined in spec | error | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-014-01 | GAP-004: no define qué estados de reserva cuentan como «préstamos» en el historial | Fijar estados en UC-014 |
| F-014-02 | API-004-03/UC-014 no definen error para correo inexistente o inválido | Añadir error y AC |
