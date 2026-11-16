# Test Matrix: UC-001 — Entrar con correo y contraseña

> Refs: UC-001, API-001-01, BDD-UC-001 (AC-001-01..06), INV-ACC-001, INV-ACC-002, INV-ACC-003, RN-001
> Techniques: EP, BVA, decision table, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| correo | Correo (VO-001) | cuenta existente (lucia, pablo, marta, ana) | correo sin cuenta · vacío · solo espacios | `""`; mayúsculas/minúsculas del mismo correo |
| contrasena | string | la de la cuenta | distinta · vacía | `""`; prefijo/sufijo de la correcta |
| sesión previa | estado | ninguna · una abierta de la misma cuenta | — | 0 / 1 sesiones por correo |
| almacén de cuentas | persistido | hash con sal por cuenta | — | dos cuentas con la misma contraseña |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | `lucia@club.es` / `Cumbre2026` · sin sesión | ok · saludo "Hola, lucia@club.es", rol socio · 1 sesión de lucia | happy | AC-001-01 |
| T02 | `pablo@club.es` / `Hielo2026` | ok · "Hola, pablo@club.es", rol socio · 1 sesión de pablo | happy | AC-001-02 |
| T03 | expand: enum(marta=encargado, ana=junta) con su contraseña | ok · "Hola, {correo}" con el rol de la cuenta · 1 sesión | derived | PERMISSIONS-MATRIX (API-001-01 todos los roles) |
| T04 | lucia + `incorrecta` | `E_AUTH_INVALID` · "Correo o contraseña incorrectos" · sin sesión, se queda en Iniciar sesión | error | AC-001-03 |
| T05 | correo sin cuenta + contraseña cualquiera | `E_AUTH_INVALID` · mismo mensaje que T04 · sin sesión | error | AC-001-03 |
| T06 | expand: enum(correo vacío, contraseña vacía, ambos vacíos) | `E_AUTH_INVALID` · "Correo o contraseña incorrectos" · sin sesión | error | AC-001-05 |
| T07 | correcta con una letra cambiada / con espacio final / correo con otra capitalización | `E_AUTH_INVALID` según regla de comparación; capitalización de correo: `undefined in spec` | boundary | AC-001-03 |
| T08 | lucia ya con sesión · reenvía el mismo formulario | ok · "Hola, lucia@club.es" · sigue 1 sola sesión | replay | AC-001-04, INV-ACC-002 |
| T09 | lucia con sesión · reenvía con contraseña incorrecta | `E_AUTH_INVALID` · sesión previa: `undefined in spec` | state | — |
| T10 | dos envíos simultáneos válidos de lucia | `undefined in spec` salvo invariante: sesiones de lucia ≤ 1 | race | INV-ACC-002 |
| T11 | cuentas lucia y otra con la misma contraseña · leer almacén | lo guardado no contiene `Cumbre2026` y difiere entre cuentas · sin cambios | derived | AC-001-06, INV-ACC-001 |
| T12 | almacén caído al iniciar sesión | `E_STORAGE_UNAVAILABLE` · "No se puede acceder a los datos; inténtalo de nuevo" · sin sesión | error | `undefined in spec` (sin AC, ver F01) |
| T13 | intento de alta con correo nuevo | ningún camino de alta: `E_AUTH_INVALID` · sin cuenta ni sesión | derived | RN-001, D-T-003 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F01 | `E_STORAGE_UNAVAILABLE` en API-001-01 sin AC ni excepción en UC-001 | Añadir E# y AC o retirar el código del contrato |
| F02 | Sin regla sobre sesión previa al fallar un reenvío, ni sobre mayúsculas/espacios del correo | Definir en UC-001 y VO-001 |
| F03 | Dos accesos simultáneos de la misma cuenta sin AC | Añadir AC de race o declarar INV-ACC-002 suficiente |
