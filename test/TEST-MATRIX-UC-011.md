# Test Matrix: UC-011 — Añadir material al inventario

> Refs: UC-011, API-003-02, BDD-UC-011 (AC-011-01..05), INV-INV-001, INV-ACC-005, INV-ACC-003
> Techniques: EP, BVA, decision table, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión/rol | Rol | encargado (marta) | socio, junta, sin sesión | — |
| material | string | nuevo ("Mosquetón"); existente ("Casco") | vacío (undefined in spec) | — |
| unidades | Unidades (VO-006) | entero ≥ 1 | 0, negativo; no entero (undefined in spec) | 0 / 1 / 2 |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · no existe "Mosquetón" · añadir 10 | ok · "Mosquetón", "10 unidades" · material creado con 10 activas | happy | AC-011-01, REQ-F-011 AC1 |
| T02 | marta · "Casco" 3 unidades · añadir 2 | ok · "Casco", "5 unidades" · activas = 5 | happy | AC-011-02, INV-INV-001 |
| T03 | marta · "Mosquetón" · unidades = 0 | `E_UNITS_MIN` · "Indica al menos 1 unidad" · inventario sin cambios | boundary | AC-011-03, INV-INV-001 |
| T04 | marta · "Casco" 5 · unidades = 1 | ok · "Casco", "6 unidades" · activas = 6 | boundary | INV-INV-001 (derived: mínimo válido) |
| T05 | marta · unidades = -1 (expand: BVA(<1)) | `E_UNITS_MIN` · "Indica al menos 1 unidad" · sin cambios | boundary | AC-011-03, INV-INV-001 |
| T06 | marta · acaba de añadir 2 "Casco" (5) · mismo formulario otra vez | ok · "Casco", "7 unidades" · activas = 7 (acción nueva) | replay | AC-011-04 |
| T07 | lucia (socio) · añadir "Casco" 2 | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin cambios | error | AC-011-05, INV-ACC-005 |
| T08 | ana (junta) · añadir "Casco" 2 | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin cambios | error | AC-011-05, INV-ACC-005 |
| T09 | sin sesión · añadir | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin cambios | derived | INV-ACC-003 |
| T10 | dos altas simultáneas del mismo material | undefined in spec (suma de ambas esperable; GAP-005 solo cubre encargados) | race | — |
| T11 | marta · unidades = 1.5 o material vacío | undefined in spec | error | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-011-01 | Sin AC para altas concurrentes del mismo material (T10) | Definir que ambas se suman |
| F-011-02 | VO-006 dice entero pero no hay error para no enteros ni material vacío (T11) | Añadir error/AC |
