# UC-008 — Multa de 2 euros por día de retraso

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-008 (primary), REQ-C-003; API-002-05, API-004-01; WF-001 step 4; INV-FIN-001, INV-FIN-002; RN-004; BDD-UC-008 |
| Actors | Primary: encargado (dispara al devolver). Secondary: multas |
| Trigger | UC-007 con `devueltaEl` posterior a `hasta` |
| Priority / Status | Must / Approved |

Al devolver tarde se crea una multa de `FINE_RATE_EUR` por día de retraso; se ve en Multas.

## Input / Output
```typescript
interface MultaOutput { dias: number; importe: number /* "Multa de {importe} € por {dias} días de retraso" */ }
```

## Preconditions
1. Devolución registrada (UC-007).

## Postconditions
- Success: multa `Pendiente` de `dias × FINE_RATE_EUR` del socio (INV-FIN-001); una por reserva (INV-FIN-002).
- Failure: ninguna multa.

## Main flow
1. System: calcula `dias = devueltaEl − hasta` en días naturales (RN-004).
2. System: si `dias > 0`, crea la multa Pendiente; si no, ninguna ("Sin multas" en Multas).
3. Encargado: ve "Multa de {importe} € por {dias} días de retraso" en Multas.

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | devuelta en `hasta` o antes | — | sin multa; "Sin multas" | AC-008-03 |
| E2 | 2 | almacén no disponible al crear | `E_STORAGE_UNAVAILABLE` | devolución y multa se deshacen juntas | AC-008-05 |
| E3 | 2 | replay: misma devolución | `O_ALREADY_RETURNED` | sigue una multa de 6 € | AC-008-04 |
