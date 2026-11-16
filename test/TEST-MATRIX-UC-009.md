# Test Matrix: UC-009 — Sin reservar mientras haya multa pendiente

> Refs: UC-009, API-002-01, BDD-UC-009 (AC-009-01..05), INV-FIN-003, SM-002
> Techniques: EP, BVA, decision table, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| multas del socio | SM-002 | ninguna · solo Pagadas | ≥ 1 Pendiente | 0 · 1 · 2 Pendientes; Pendiente + Pagada |
| importe por multa | number € | > 0 (p. ej. 2, 4, 6) | — | suma exacta de todas las Pendientes |
| actor | sesión · rol | socio sin multa, encargado sin multa | socio con multa | multa de otro socio |
| reservas previas | SM-001 | Reservada creadas antes de la multa | — | — |
| pago concurrente | UC-010 | pago antes / después de decidir | — | pago en medio de la comprobación |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | lucia con 1 multa Pendiente 6 € · Piolet 2027-01-10..11 | `E_FINE_PENDING` · "No puedes reservar hasta pagar tu multa de 6 €" · sin reserva | error | AC-009-01, INV-FIN-003 |
| T02 | pablo con multas Pendientes 2 € y 4 € · reserva cualquier material | `E_FINE_PENDING` · "No puedes reservar hasta pagar tu multa de 6 €" (suma) · sin reserva | error | AC-009-02, INV-FIN-003 |
| T03 | lucia con multa, marta sin multa · marta reserva Piolet 2027-01-10..11 | ok · "Reserva creada: Piolet del 2027-01-10 al 2027-01-11" · reserva de marta; multa de lucia intacta | happy | AC-009-03 |
| T04 | lucia con reservas Reservada · se crea su multa | sus reservas siguen Reservada en Mis reservas · sin cambios | state | AC-009-04, INV-FIN-003 |
| T05 | pablo con multa Pendiente que se paga durante la reserva | `E_FINE_PENDING` o reserva creada según estado al confirmar; nunca reserva con multa Pendiente | race | AC-009-05, INV-FIN-003 |
| T06 | race con pago confirmado antes de decidir | ok · reserva creada · multa Pagada | race | AC-009-05 |
| T07 | race con pago confirmado después de decidir | `E_FINE_PENDING` · sin reserva · multa Pagada | race | AC-009-05 |
| T08 | lucia con multa aún Pendiente · reintenta el mismo formulario | `E_FINE_PENDING` · mismo mensaje e importe · sin reserva | replay | AC-009-01 |
| T09 | expand: BVA(Pendientes=0 → ok, =1 → `E_FINE_PENDING`) | según valor · sin reserva en error | boundary | AC-009-01, INV-FIN-003 |
| T10 | solo multas Pagadas | ok · reserva creada | derived | INV-FIN-003 (count Pendiente = 0), SM-002 |
| T11 | 1 Pendiente 6 € + 1 Pagada 4 € | `E_FINE_PENDING` · importe 6 € (solo Pendientes) | derived | INV-FIN-003, SM-002 |
| T12 | tras pagar la única multa · reintenta | ok · reserva creada | derived | SM-002 Pendiente→Pagada |
| T13 | con 2 Pendientes se paga 1 · reintenta | `E_FINE_PENDING` · importe = la restante | derived | INV-FIN-003 |
| T14 | multa Pendiente + rango inválido / sin stock | `E_FINE_PENDING` (multa antes que duplicado y stock) · sin reserva | derived | D-T-004, AC-002-08 |
| T15 | rechazo por multa · estado del stock | sin unidad ocupada; reservas previas Reservada | state | INV-FIN-003, UC-009 Postconditions |
| T16 | encargado con multa propia | `undefined in spec` (el UC define el bloqueo solo para socio) | derived | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F01 | INV-FIN-003 habla de «el socio»; no dice si un encargado con multa también se bloquea | Aclarar el alcance del bloqueo por rol |
| F02 | AC-009-05 no fija el resultado concreto; solo exige no crear reserva con multa Pendiente | Aceptar como invariante o añadir orden de confirmación |
