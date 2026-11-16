# WF-001 — Ciclo de una reserva

| Field | Value |
|---|---|
| Trigger | Un socio reserva material (UC-002) |
| Actors | socio, encargado |
| Refs | REQ-F-002, REQ-F-005..010; UC-002, UC-005..008, UC-010; SM-001, SM-002; INV-RES-005, INV-FIN-001..003 |

## Steps

| # | Name | Type | Input → Output | On failure |
|---|---|---|---|---|
| 1 | Reservar | UC-002 | socio, material, fechas → `Reservada` | no se crea; `E_*` de API-002-01 |
| 2 | Entregar | UC-006 | reserva → `Entregada` (o cancelar antes, UC-005) | sin cambios |
| 3 | Devolver | UC-007 | reserva, fecha → `Devuelta` | sin cambios |
| 4 | Multar | UC-008 | si devuelta > `hasta` → multa Pendiente | devolución y multa se deshacen juntas |
| 5 | Pagar | UC-010 | multa → `Pagada`; el socio puede reservar (INV-FIN-003) | sin cambios |

## Error scenarios

Compensation: ninguna entre pasos; cada paso es una operación independiente. `E_AUTH_REQUIRED` y `E_STORAGE_UNAVAILABLE` aplican a todos los pasos y se describen aquí una sola vez.
