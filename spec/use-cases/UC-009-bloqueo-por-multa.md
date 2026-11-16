# UC-009 — Sin reservar mientras haya multa pendiente

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-009 (primary); API-002-01; INV-FIN-003; BDD-UC-009 |
| Actors | Primary: socio. Secondary: multas |
| Trigger | Intenta reservar con alguna multa Pendiente |
| Priority / Status | Must / Approved |

Se rechaza toda reserva nueva del socio con multas Pendientes; las reservas ya hechas no cambian.

## Input / Output
```typescript
interface BloqueoOutput { importePendiente: number /* suma de multas Pendientes, "{importe} €" */ }
```

## Preconditions
1. Sesión del socio.

## Postconditions
- Success (rechazo): nada reservado; reservas previas siguen `Reservada` (INV-FIN-003).
- Failure: n/a.

## Main flow
1. Socio: pide reservar (UC-002).
2. System: suma sus multas Pendientes (INV-FIN-003).
3. System: si suma > 0, rechaza y muestra "No puedes reservar hasta pagar tu multa de {importe} €" en Material.

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 3 | una o más multas pendientes | `E_FINE_PENDING` | importe = suma de todas; nada reservado | AC-009-02 |
| E2 | 3 | otra multa se paga durante la comprobación | `E_FINE_PENDING` o se crea, según el estado al decidir | decide el estado al confirmar | AC-009-05 |
| E3 | 3 | replay: reintento con multa aún pendiente | `E_FINE_PENDING` | mismo rechazo | AC-009-01 |
