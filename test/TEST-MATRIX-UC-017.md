# Test Matrix: UC-017 — Cerrar sesión con un botón

> Refs: UC-017, API-001-02, BDD-UC-017 (AC-017-01..03), REQ-F-018, INV-ACC-002, INV-ACC-003, INV-ACC-007
> Techniques: EP, state transition, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión actual | estado | abierta de socio · encargado · junta | ya cerrada · inexistente | 0 / 1 sesiones por correo |
| destino tras salir | pantalla | Iniciar sesión | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived. The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | lucia con sesión · "Salir" | ok · "Iniciar sesión" · 0 sesiones de lucia | happy | AC-017-01 |
| T02 | expand: enum(pablo=socio, marta=encargado, ana=junta) · "Salir" | ok · "Iniciar sesión" · sin sesión | derived | PERMISSIONS-MATRIX (API-001-02 todos los roles) |
| T03 | lucia salió · pide Material con la sesión anterior | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos | error | AC-017-02 |
| T04 | T03 con otra operación cualquiera (reservas, catálogo) | `E_AUTH_REQUIRED` · sin datos | derived | AC-017-02, INV-ACC-003 |
| T05 | lucia salió · "Salir" otra vez con la sesión anterior | ok · "Iniciar sesión" · nada más cambia (sesiones de otros intactas) | replay | AC-017-03 |
| T06 | lucia y pablo con sesión · lucia sale | sesión de pablo sigue vigente · "Hola, pablo@club.es" | state | AC-017-01, INV-ACC-002 |
| T07 | lucia salió · entra otra vez con `Cumbre2026` | ok · sesión nueva, la anterior sigue sin valer | state | AC-017-01, AC-001-01 |
| T08 | "Salir" sin ninguna sesión | ok · "Iniciar sesión" · sin cambios | boundary | UC-017 §Preconditions |
| T09 | almacén caído al salir | `E_STORAGE_UNAVAILABLE` · sesión: undefined in spec | error | — |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| sesión | abierta | "Salir" | cerrada | no vale más; "Iniciar sesión" | T01, T03 |
| sesión | cerrada | "Salir" otra vez | cerrada | mismo destino, sin cambios | T05 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F01 | `E_STORAGE_UNAVAILABLE` en API-001-02 sin AC: ¿queda la sesión abierta o cerrada? | Añadir E# y AC o retirar el código |
