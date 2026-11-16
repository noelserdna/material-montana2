# Test Matrix: UC-016 — Datos de los socios solo para su dueño, el encargado y la junta

> Refs: UC-016, API-002-02, API-004-01, API-004-03, BDD-UC-016 (AC-016-01..05), INV-ACC-003, INV-ACC-004, REQ-NF-002
> Techniques: EP, BVA, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| actor | Correo \| null | dueño, encargado (marta), junta (ana) | otro socio, null (sin sesión) | — |
| dueno | Correo | socio con datos (lucia, pablo) | — | — |
| multas | lista | ≥1 (lucia 6 €) | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | lucia pide historial de pablo (API-004-03) | `E_FORBIDDEN_DATA` · "No tienes permiso para ver estos datos" · ningún dato de pablo | error | AC-016-01, INV-ACC-004 |
| T02 | pablo abre historial de lucia | `E_FORBIDDEN_DATA` · "No tienes permiso para ver estos datos" | error | AC-016-02, INV-ACC-004 |
| T03 | lucia con multa 6 € · marta abre Multas (API-004-01) | ok · "lucia@club.es" con "Multa de 6 €" | happy | AC-016-03 |
| T04 | sin sesión pide Material (API-003-01) | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos | error | AC-016-04, INV-ACC-003 |
| T05 | lucia repite la petición del historial de pablo | `E_FORBIDDEN_DATA` otra vez · mismo literal | replay | AC-016-05 |
| T06 | actor = dueño (lucia→lucia), expand: enum(socio) | ok · sus datos | derived | INV-ACC-004 (actor = dueño) |
| T07 | marta / ana · historial de lucia | ok · datos de lucia | derived | INV-ACC-004 (rol ∈ {encargado, junta}) |
| T08 | sin sesión, expand: enum(API-002-02, API-004-01, API-004-03) | `E_AUTH_REQUIRED` · sin datos | derived | INV-ACC-003 |
| T09 | lucia (socio) pide listarMultas (API-004-01) | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin datos | derived | PERMISSIONS-MATRIX, INV-ACC-004 |
| T10 | ana (junta) pide listarMultas | `E_FORBIDDEN_ROLE` · UC-016 dice que la junta ve datos | boundary | — (matriz y UC divergen) |
| T11 | lucia · listarMisReservas (API-002-02) | ok · solo sus reservas, nunca las de pablo | derived | INV-ACC-004 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-016-01 | AC-016-01 y AC-016-02 son el mismo caso con actores intercambiados | Fusionar o diferenciar |
| F-016-02 | API-002-02 lista `E_FORBIDDEN_DATA` pero no recibe id ajeno: no es alcanzable | Quitar error o añadir input |
| F-016-03 | API-004-01: matriz solo encargado; UC-016 dice «encargado ve multas» y 004-01 no cita a junta | Aclarar acceso de junta |
