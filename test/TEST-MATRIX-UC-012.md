# Test Matrix: UC-012 — Dar de baja unidades rotas

> Refs: UC-012, API-003-03, BDD-UC-012 (AC-012-01..08), INV-INV-001, INV-INV-002, INV-RES-001, INV-ACC-005, RN-002, D-T-009
> Techniques: EP, BVA, decision table, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión/rol | Rol | encargado (marta) | socio, junta, sin sesión | — |
| material | string | existente ("Casco", "Cuerda", "Piolet") | inexistente | — |
| unidades | Unidades (VO-006) | 1 ≤ n ≤ activas | n < 1; n > activas; no entero (undefined in spec) | 0 / 1 / activas / activas+1 |
| reservas del material | estado | sin reservas; Reservada futura sobre la única unidad activa | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · "Casco" 5 activas · baja 1 | ok · "Casco", "4 unidades" · activas = 4 | happy | AC-012-01, REQ-F-012 AC1 |
| T02 | marta · "Cuerda" 2 activas · baja 3 | `E_UNITS_INSUFFICIENT` · "No hay tantas unidades activas de Cuerda" · activas = 2 | error | AC-012-02, INV-INV-002 |
| T03 | marta · "Cuerda" 2 activas · baja 2 | ok · "Cuerda", "0 unidades" · activas = 0 | boundary | INV-INV-002 (derived: n = activas) |
| T04 | "Piolet" 1 activa dada de baja · lucia reserva "Piolet" | `E_NO_STOCK` · "No quedan unidades libres de Piolet esas fechas" · sin reserva nueva | state | AC-012-03, INV-RES-001 |
| T05 | marta · baja 0 | `E_UNITS_MIN` · "Indica al menos 1 unidad" · activas sin cambios | boundary | AC-012-04, INV-INV-001 |
| T06 | marta · baja -1 (expand: BVA(<1)) | `E_UNITS_MIN` · "Indica al menos 1 unidad" · sin cambios | boundary | AC-012-04, INV-INV-001 |
| T07 | marta · material inexistente · baja 1 | `E_MATERIAL_NOT_FOUND` · "Ese material no existe" · sin cambios | error | AC-012-05 |
| T08 | lucia (socio) · baja 1 | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · activas sin cambios | error | AC-012-06, INV-ACC-005 |
| T09 | ana (junta) · baja 1 | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · activas sin cambios | error | AC-012-06, INV-ACC-005 |
| T10 | única unidad activa con reserva futura de lucia · marta baja 1 | ok · baja hecha · reserva sigue `Reservada`; sin aviso ni cancelación automática | state | AC-012-07, RN-002, INV-INV-002, D-T-009 |
| T11 | "Casco" 1 activa ya dada de baja (0) · repetir baja 1 | `E_UNITS_INSUFFICIENT` · "No hay tantas unidades activas de Casco" · activas = 0 | replay | AC-012-08 |
| T12 | "Casco" 5 activas · repetir baja 1 dos veces | ok · "3 unidades" (resta de nuevo, acción nueva) | replay | AC-012-08, D-T-008 |
| T13 | sin sesión · baja | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin cambios | derived | INV-ACC-003 |
| T14 | marta · material inexistente y unidades = 0 | undefined in spec (precedencia `E_UNITS_MIN` vs `E_MATERIAL_NOT_FOUND`) | error | — |
| T15 | dos bajas simultáneas sobre las últimas unidades | undefined in spec (GAP-005 cubre solo dos encargados) | race | — |
| T16 | marta · baja 1.5 | undefined in spec | error | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-012-01 | Precedencia entre `E_UNITS_MIN` y `E_MATERIAL_NOT_FOUND` no definida (T14) | Fijar orden en UC-012 paso 2 |
| F-012-02 | Bajas concurrentes sin AC; riesgo de dejar activas < 0 (T15) | Definir resultado atómico |
| F-012-03 | No enteros en VO-006 sin error asociado (T16) | Añadir error/AC |
