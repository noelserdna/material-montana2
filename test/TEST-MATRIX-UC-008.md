# Test Matrix: UC-008 — Multa de 2 euros por día de retraso

> Refs: UC-008, API-002-05, API-004-01, BDD-UC-008 (AC-008-01..05), INV-FIN-001, INV-FIN-002, RN-004, REQ-C-003, SM-001, SM-002
> Techniques: EP, BVA, decision table, state transition, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| devueltaEl − hasta (días naturales) | int | ≤ 0 (sin multa); > 0 (multa) | `devueltaEl` < `desde` o futuro (undefined) | -1, 0, 1, 3 |
| FINE_RATE_EUR | number | 2 | — | — |
| multas del socio | store | ninguna; una Pendiente | segunda por la misma reserva | una por reserva |
| almacén | store | disponible | `E_STORAGE_UNAVAILABLE` al crear | — |
| sesión (rol) | enum | encargado ve Multas | socio, junta en API-004-01 | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | "Casco" de lucia hasta 2026-12-12 · devuelta 2026-12-15 (+3) | "Multa de 6 € por 3 días de retraso" en Multas · una multa Pendiente de 6 € | boundary | AC-008-01, INV-FIN-001 |
| T02 | "Cuerda" de pablo hasta 2026-12-21 · devuelta 2026-12-22 (+1) | "Multa de 2 € por 1 días de retraso" · multa Pendiente de 2 € | boundary | AC-008-02, INV-FIN-001 |
| T03 | "Tienda" hasta 2026-12-20 · devuelta 2026-12-20 (0) | "Sin multas" en Multas · ninguna multa | boundary | AC-008-03 |
| T04 | "Tienda" hasta 2026-12-20 · devuelta 2026-12-19 (-1) | "Sin multas" en Multas · ninguna multa | boundary | AC-008-03 |
| T05 | expand: BVA(`devueltaEl − hasta` = -1, 0, 1, 3 con FINE_RATE_EUR=2) → importe 0, 0, 2, 6 € | multa solo si > 0; importe = días × 2 | boundary | AC-008-01..03, INV-FIN-001, RN-004 |
| T06 | multa de 6 € creada; se repite la devolución | `O_ALREADY_RETURNED` · "Multa de 6 € por 3 días de retraso" · sigue una multa | replay | AC-008-04, INV-FIN-002, UC-008 E3 |
| T07 | falla el almacén al crear la multa | `E_STORAGE_UNAVAILABLE` · "No se puede acceder a los datos; inténtalo de nuevo" · reserva sigue Entregada, sin multa | error | AC-008-05 |
| T08 | multa creada · lucia consulta sus multas | multa del socio dueño, estado Pendiente | state | INV-FIN-001, SM-002 |
| T09 | marta abre Multas sin multas existentes | "Sin multas" | happy | AC-008-03 |
| T10 | lucia (socio) o ana (junta) abre API-004-01 | `E_FORBIDDEN_ROLE` · — · sin cambios | error | UC-008 contrato API-004-01 |
| T11 | dos encargados devuelven la misma reserva con retraso a la vez | undefined in spec | race | — |
| T12 | `devueltaEl` < `desde` o futuro | undefined in spec | boundary | — |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| SM-002 | — | devuelta > hasta | Pendiente | importe = días × FINE_RATE_EUR | T01, T02, T05 |
| SM-002 | — | devuelta ≤ hasta | — | no se crea multa | T03, T04, T05 |
| SM-002 | Pendiente | repetir devolución | Pendiente | una sola multa por reserva | T06 |
| SM-002 | — | fallo de almacén | — | devolución y multa deshechas juntas | T07 |
| SM-001 | Entregada | devolver con retraso | Devuelta | solo si la multa se crea | T01, T07 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-008-01 | AC-008-02 fija "1 días" (plural fijo) | Confirmar literal o pluralizar |
| F-008-02 | Dos encargados devolviendo a la vez sin AC (GAP-005) | Definir race |
| F-008-03 | `devueltaEl` anterior a `desde`/futuro sin definir (GAP-003) | Definir validación |
| F-008-04 | AC-008-04 replay con código `O_ALREADY_RETURNED` pero AC solo exige una multa | Aclarar salida |
