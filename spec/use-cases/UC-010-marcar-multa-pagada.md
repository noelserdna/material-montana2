# UC-010 — Marcar la multa como pagada

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-010 (primary); API-004-02; WF-001 step 5; INV-FIN-003, INV-ACC-005; BDD-UC-010 |
| Actors | Primary: encargado. Secondary: multas |
| Trigger | Marca pagada en Multas |
| Priority / Status | Must / Approved |

La multa pasa a Pagada; si el socio no tiene otra pendiente, puede reservar de nuevo.

## Input / Output
```typescript
interface PagarInput  { multaId: number }
interface PagarOutput { mensaje: string /* "Multa de {importe} € pagada" */; estado: 'Pagada' }
```

## Preconditions
1. Sesión del encargado.

## Postconditions
- Success: `Pagada`; otras multas del socio intactas (INV-FIN-003).
- Failure: sin cambios.

## Main flow
1. Encargado: marca una multa pagada.
2. System: comprueba rol y existencia.
3. System: pasa a `Pagada` y muestra "Multa de {importe} € pagada" en Multas.

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | rol socio o junta | `E_FORBIDDEN_ROLE` | sigue Pendiente | AC-010-04 |
| E2 | 2 | multa inexistente | `E_FINE_NOT_FOUND` | sin cambios | AC-010-05 |
| E3 | 3 | replay: ya Pagada | `O_ALREADY_PAID` | queda pagada una vez; "Ya estaba pagada" | AC-010-03 |
