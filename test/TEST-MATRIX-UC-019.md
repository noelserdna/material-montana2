# Test Matrix: UC-019 — Ver las reservas de todos los socios

> Refs: UC-019, API-002-07, BDD-UC-019 (AC-019-01..08), REQ-F-020, INV-RES-005, INV-ACC-003, INV-ACC-005
> Techniques: EP, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| sesión/rol | Rol | encargado (marta) | socio (lucia), junta (ana), sin sesión | — |
| filtro | FiltroReservas opcional | omitido (= Por entregar) · Por entregar · Entregadas · Todas | valor fuera del enum (undefined in spec) | `expand: enum(FiltroReservas)` |
| reservas | estado | Reservada · Entregada · Devuelta · Cancelada | — | 0 coincidentes → "No hay reservas" |

## Cases

`Type`: happy · error · boundary · state · replay · derived. The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | marta · Casco de lucia Reservada 2026-12-10..12-12, Cuerda de pablo Entregada · sin `filtro` | lista "Casco" "lucia@club.es" con fechas y estado; sin Cuerda · sin cambios | happy | AC-019-01, REQ-F-020 AC1 |
| T02 | mismo estado · `filtro` = Entregadas | "Cuerda" "pablo@club.es"; sin Casco · sin cambios | happy | AC-019-02, REQ-F-020 AC2 |
| T03 | mismo estado · `filtro` = Todas | "Casco" "lucia@club.es" y "Cuerda" "pablo@club.es" · sin cambios | happy | AC-019-03, REQ-F-020 AC3 |
| T04 | ninguna reserva coincide con el filtro (p. ej. solo Entregadas y filtro Por entregar) | "No hay reservas" · sin cambios | boundary | AC-019-04, REQ-F-020 AC4 |
| T05 | lucia (socio) · pide la lista de todas | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin datos | error | AC-019-05, REQ-F-020 AC5, INV-ACC-005 |
| T06 | ana (junta) · pide la lista de todas | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin datos | error | AC-019-06, REQ-F-020 AC6, INV-ACC-005 |
| T07 | sin sesión · pide la lista | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos | error | AC-019-07, INV-ACC-003 |
| T08 | marta · abre la lista dos veces | misma lista las dos veces · sin cambios | replay | AC-019-08 |
| T09 | una reserva de cada estado (Reservada, Entregada, Devuelta, Cancelada) · cada filtro | Por entregar = solo Reservada; Entregadas = solo Entregada; Todas = las 4 | state | UC-019 postcond., INV-RES-005 (derived) |
| T10 | reservas de 3 socios · `Todas` | aparecen las de todos, no solo las del encargado | derived | UC-019 postcond., REQ-F-020 |
| T11 | `filtro` fuera del enum | undefined in spec | error | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-019-01 | Valor de `filtro` fuera del enum sin error definido (T11) | Definir error o aceptar que la interfaz no lo permite |
