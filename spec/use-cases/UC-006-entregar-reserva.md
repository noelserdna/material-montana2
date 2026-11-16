# UC-006 — Entregar lo reservado

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-006 (primary); API-002-04; WF-001 step 2; INV-RES-005, INV-ACC-005; BDD-UC-006 |
| Actors | Primary: encargado. Secondary: reservas |
| Trigger | Pulsa entregar en Entregas y devoluciones |
| Priority / Status | Must / Approved |

El encargado marca la reserva como entregada; la unidad cuenta como prestada.

## Input / Output
```typescript
interface EntregarInput  { reservaId: number }
interface EntregarOutput { mensaje: string /* "Entregada a {correo}" */; estado: 'Entregada' }
```

## Preconditions
1. Sesión del encargado.

## Postconditions
- Success: `Entregada` (INV-RES-005).
- Failure: la reserva no cambia.

## Main flow
1. Encargado: marca la reserva como entregada.
2. System: comprueba rol y estado (INV-RES-005).
3. System: pasa a `Entregada` y muestra "Entregada a {correo}" en Entregas y devoluciones.

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | rol socio o junta | `E_FORBIDDEN_ROLE` | sin cambios | AC-006-05 |
| E2 | 2 | reserva Cancelada | `E_RES_CANCELLED` | "No se puede entregar una reserva cancelada" | AC-006-04 |
| E3 | 2 | reserva inexistente | `E_RES_NOT_FOUND` | sin cambios | AC-006-06 |
| E5 | 2 | reserva Devuelta | `E_DELIVER_STATE` | sin cambios; "Solo se puede entregar una reserva pendiente de entrega" | AC-006-07 |
| E4 | 3 | replay: ya Entregada | `O_ALREADY_DELIVERED` | queda Entregada una vez; "Ya estaba entregada" | AC-006-03 |
