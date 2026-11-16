# Test Matrix: UC-003 — Ver el catálogo de material

> Refs: UC-003, API-003-01, BDD-UC-003 (AC-003-01..04), INV-ACC-003, INV-INV-002
> Techniques: EP, BVA, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| Sesión / rol | actor | socio, encargado, junta | sin sesión | — |
| Unidades activas por material | entero ≥ 0 (estado) | varios (4, 3), una (1) | 0 activas (no definido) | 0, 1, 2 |
| Unidades dadas de baja | estado | material con bajas: no cuentan | — | activas = total − bajas |
| Número de materiales | estado | 1, varios | catálogo vacío (no definido) | 0, 1, 2 |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | lucia con sesión · Piolet 4 activas, Casco 3 activas | ok · "Piolet" "4 unidades" y "Casco" "3 unidades" · sin cambios | happy | AC-003-01 |
| T02 | pablo con sesión · Tienda 1 activa | ok · "Tienda" "1 unidades" (literal sin singular) · sin cambios | boundary | AC-003-02 |
| T03 | sin sesión | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos de material | error | AC-003-03, INV-ACC-003 |
| T04 | lucia abre Material dos veces seguidas | ok · misma lista las dos veces · almacén sin cambios | replay | AC-003-04 |
| T05 | Casco con 5 unidades y 2 dadas de baja | ok · "Casco" "3 unidades" (solo activas) · sin cambios | derived | INV-INV-002 |
| T06 | marta (encargado) con sesión · materiales sembrados | ok · misma lista que ve un socio · sin cambios | derived | PERMISSIONS-MATRIX (API-003-01) |
| T07 | ana (junta) con sesión · materiales sembrados | ok · lista completa · sin cambios | derived | PERMISSIONS-MATRIX (API-003-01) |
| T08 | material con 0 unidades activas (todas de baja) | undefined in spec · ¿se lista "0 unidades" u se oculta? · — | boundary | — |
| T09 | catálogo sin ningún material | undefined in spec · lista vacía sin texto definido · — | boundary | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-003-01 | UC-003 no define si un material con 0 unidades activas se lista (T08) | Definir "0 unidades" u ocultar |
| F-003-02 | UC-003 no define texto de catálogo vacío (T09) | Añadir literal en VO-010 o declarar fuera de alcance |
| F-003-03 | "1 unidades" (AC-003-02) fija un plural incorrecto como literal | Confirmar que es intencional |
