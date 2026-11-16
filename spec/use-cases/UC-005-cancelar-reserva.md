# UC-005 — Cancelar una reserva hasta el día antes

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-005 (primary); API-002-03; WF-001; INV-RES-004, INV-RES-001, INV-ACC-004; BDD-UC-005 |
| Actors | Primary: socio. Secondary: reservas |
| Trigger | Pulsa cancelar en Mis reservas |
| Priority / Status | Must / Approved |

El socio anula su reserva y la unidad queda libre para otros.

## Input / Output
```typescript
interface CancelarInput  { reservaId: number }
interface CancelarOutput { mensaje: "Reserva cancelada"; estado: 'Cancelada' }
```

## Preconditions
1. Sesión abierta; la reserva es del actor (INV-RES-004).

## Postconditions
- Success: estado `Cancelada`; unidad libre (INV-RES-001).
- Failure: la reserva no cambia.

## Main flow
1. Socio: cancela una reserva suya.
2. System: comprueba dueño y que hoy < `desde` (INV-RES-004).
3. System: pasa a `Cancelada` y muestra "Reserva cancelada" en Mis reservas.

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | hoy ≥ `desde` | `E_CANCEL_TOO_LATE` | sigue Reservada; "Ya no se puede cancelar: la reserva empieza hoy" | AC-005-03 |
| E2 | 2 | reserva de otro socio | `E_FORBIDDEN_DATA` | la ajena sigue Reservada | AC-005-05 |
| E3 | 2 | rol junta | `E_FORBIDDEN_ROLE` | sin cambios | AC-005-06 |
| E4 | 2 | reserva inexistente | `E_RES_NOT_FOUND` | sin cambios | AC-005-07 |
| E5 | 3 | cancelada a la vez por dos peticiones | — | una sola cancelación; la otra ve `O_ALREADY_CANCELLED` | AC-005-04 |
| E6 | 3 | replay: cancelar otra vez | `O_ALREADY_CANCELLED` | sigue "Cancelada"; nada más cambia | AC-005-04 |
