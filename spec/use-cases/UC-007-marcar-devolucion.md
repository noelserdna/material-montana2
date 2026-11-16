# UC-007 — Marcar la devolución

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-007 (primary), REQ-F-008; API-002-05; WF-001 step 3; INV-RES-005, INV-FIN-002, INV-ACC-005; RN-004; BDD-UC-007 |
| Actors | Primary: encargado. Secondary: reservas, multas (UC-008) |
| Trigger | Marca devuelta en Entregas y devoluciones |
| Priority / Status | Must / Approved |

El encargado registra la fecha real de devolución y la unidad queda libre.

## Input / Output
```typescript
interface DevolverInput  { reservaId: number; devueltaEl: Dia }
interface DevolverOutput { mensaje: string /* "Devuelta el {fecha} sin retraso" o multa (UC-008) */; estado: 'Devuelta' }
```

## Preconditions
1. Sesión del encargado; reserva `Entregada`.

## Postconditions
- Success: `Devuelta` con `devueltaEl`; unidad libre; multa si hay retraso (INV-FIN-002).
- Failure: sin cambios.

## Main flow
1. Encargado: marca la devolución con la fecha.
2. System: comprueba rol y estado (INV-RES-005).
3. System: guarda `devueltaEl` y pasa a `Devuelta`.
4. System: si `devueltaEl` > `hasta`, aplica UC-008; si no, muestra "Devuelta el {fecha} sin retraso".

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | rol socio o junta | `E_FORBIDDEN_ROLE` | sin cambios | AC-007-05 |
| E2 | 2 | no estaba Entregada | `E_NOT_DELIVERED` | "Solo se puede devolver lo que se ha entregado" | AC-007-03 |
| E3 | 2 | reserva inexistente | `E_RES_NOT_FOUND` | sin cambios | AC-007-06 |
| E4 | 3 | replay: ya Devuelta | `O_ALREADY_RETURNED` | un solo registro, sin segunda multa; "Ya estaba devuelta" | AC-007-02 |
