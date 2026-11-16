# Test Matrix: UC-002 — Reservar material para unas fechas

> Refs: UC-002, API-002-01, BDD-UC-002 (AC-002-01..10), INV-RES-001, INV-RES-002, INV-RES-003, INV-FIN-003, INV-ACC-003, INV-ACC-005, RN-003, SM-001
> Techniques: EP, BVA, decision table, state transition, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| material | string | existente (Casco, Piolet) | inexistente · vacío | — |
| desde / hasta | Dia (VO-003) | `hasta` ≥ `desde` | `hasta` < `desde` | `hasta`=`desde` (1 día); `hasta`=`desde`-1 |
| unidades activas | persistido | ≥ 1 | 0 (todas de baja) | 1 · 3 |
| ocupadas por día | Reservada/Entregada | < activas todos los días | = activas en algún día | activas-1 · activas; día extremo; hueco central |
| reservas no activas | Cancelada/Devuelta | no ocupan | — | misma clave que una nueva |
| duplicado | (socio, material, desde, hasta) activa | ninguno | uno activo | mismas fechas / fechas desplazadas 1 día |
| multas del socio | Pendiente | ninguna | ≥ 1 | 0 / 1 |
| actor | sesión · rol | socio, encargado | junta · sin sesión | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | 3 Casco libres · lucia 2026-12-10..12 | ok · "Reserva creada: Casco del 2026-12-10 al 2026-12-12" · 1 reserva Reservada de lucia | happy | AC-002-01, INV-RES-001 |
| T02 | 3 Casco, 2 reservadas 12-10..12 · pablo 12-11..13 | ok · "Reserva creada: Casco del 2026-12-11 al 2026-12-13" · reserva creada | happy | AC-002-02, RN-003 |
| T03 | 3 Casco, 3 reservadas 12-10..12 · pablo 12-12..14 | `E_NO_STOCK` · "No quedan unidades libres de Casco esas fechas" · sin reserva | error | AC-002-03, INV-RES-001 |
| T04 | 3 Casco reservadas 12-10..12 · socio 12-13..14 | ok · "Reserva creada: Casco del 2026-12-13 al 2026-12-14" · reserva creada | happy | AC-002-04 |
| T05 | lucia reservó Casco 12-10..12 · mismo formulario (3 unidades) | `E_DUP_RESERVATION` · "Ya tienes esa reserva" · una sola reserva | replay | AC-002-05, INV-RES-003 |
| T06 | 1 sola unidad, lucia ya la tiene reservada · mismo formulario | `E_DUP_RESERVATION` (no `E_NO_STOCK`) · una sola reserva | replay | AC-002-05, INV-RES-003 |
| T07 | lucia 12-11..12-09 | `E_DATE_RANGE` · "La fecha de fin no puede ser anterior a la de inicio" · nada reservado | error | AC-002-06, INV-RES-002 |
| T08 | rol junta pide reservar Casco | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin reserva | error | AC-002-07, INV-ACC-005 |
| T09 | lucia con multa Pendiente 6 € · Piolet 2027-01-10..11 | `E_FINE_PENDING` · "No puedes reservar hasta pagar tu multa de 6 €" · sin reserva | error | AC-002-08, INV-FIN-003 |
| T10 | lucia pide material inexistente | `E_MATERIAL_NOT_FOUND` · "Ese material no existe" · sin reserva | error | AC-002-09 |
| T11 | 1 unidad libre · dos socios a la vez | una reserva ok; la otra `E_NO_STOCK` · 1 reserva en total, gana el primero confirmado | race | AC-002-10, INV-RES-001 |
| T12 | `desde`=`hasta` (12-10..12-10), stock libre | ok · "Reserva creada: Casco del 2026-12-10 al 2026-12-10" | boundary | AC-002-01, INV-RES-002 |
| T13 | expand: BVA(hasta=desde-1 → `E_DATE_RANGE`, hasta=desde → ok) | según valor | boundary | AC-002-06, INV-RES-002 |
| T14 | expand: BVA(ocupadas=activas-1 → ok, ocupadas=activas → `E_NO_STOCK`) en un solo día del rango | según valor · sin reserva en error | boundary | AC-002-02, AC-002-03 |
| T15 | 3 Casco; día central lleno, extremos libres · 12-10..12 | `E_NO_STOCK` · sin reserva | boundary | AC-002-03, INV-RES-001 |
| T16 | ocupadas: expand: enum(Reservada, Entregada → cuentan; Cancelada, Devuelta → no cuentan) | cuentan: `E_NO_STOCK`; no cuentan: ok | state | INV-RES-001 |
| T17 | unidades dadas de baja reducen activas a ocupadas | `E_NO_STOCK` · sin reserva | derived | INV-RES-001 (unidades activas) |
| T18 | lucia con reserva Cancelada de misma clave · repite | ok · nueva reserva creada | derived | INV-RES-003 (solo activas) |
| T19 | lucia reserva misma clave con fechas desplazadas 1 día | ok · segunda reserva distinta | boundary | INV-RES-003 |
| T20 | pablo reserva misma clave que lucia (otro socio) | ok si hay stock · reserva propia | derived | INV-RES-003 (clave incluye socio) |
| T21 | multa Pendiente + duplicado activo | `E_FINE_PENDING` · sin cambios | derived | D-T-004, INV-FIN-003 |
| T22 | duplicado + sin stock (1 unidad) | `E_DUP_RESERVATION` | derived | AC-002-05, D-T-004 |
| T23 | multa + material inexistente | `E_FINE_PENDING` o `E_MATERIAL_NOT_FOUND`: `undefined in spec` | derived | — |
| T24 | encargado marta reserva con stock libre | ok · reserva Reservada de marta | derived | PERMISSIONS-MATRIX, GAP-006 |
| T25 | sin sesión | `E_AUTH_REQUIRED` · "Iniciar sesión" · sin datos ni reserva | error | INV-ACC-003 |
| T26 | fechas pasadas (< hoy 2026-12-01) | `undefined in spec` | boundary | — (GAP-003) |
| T27 | almacén caído | `E_STORAGE_UNAVAILABLE` · sin escritura parcial | error | — (D-T-002; sin AC) |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|-----|-----------------------------|-------|
| SM-001 | — | crearReserva con rango válido, sin multa, sin duplicado, stock | Reservada | visible en Mis reservas; ocupa unidad cada día | T01, T02, T04 |
| SM-001 | — | cualquier guard falla | (sin reserva) | no se crea reserva ni se ocupa unidad | T03, T07, T08, T09, T10 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F01 | Precedencia multa vs material inexistente no definida (D-T-004 solo cubre rol, rango, multa, dup., stock) | Fijar posición de `E_MATERIAL_NOT_FOUND` en el flujo |
| F02 | `E_AUTH_REQUIRED` y `E_STORAGE_UNAVAILABLE` sin AC en UC-002 | Añadir E# y AC |
| F03 | Fechas pasadas sin regla (GAP-003) | Resolver SIL-002 |
