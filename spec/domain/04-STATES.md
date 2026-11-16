# 04 — State Machines

## SM-001: Reserva

| State | Initial | Final | Description |
|---|---|---|---|
| Reservada | Yes | No | Unidad apartada para el rango |
| Entregada | No | No | Unidad en manos del socio |
| Devuelta | No | Yes | Unidad devuelta, multa calculada |
| Cancelada | No | Yes | Anulada por el socio, unidad liberada |

| From | To | Trigger | Guard | Action |
|---|---|---|---|---|
| — | Reservada | UC-002 | INV-RES-001, INV-FIN-003 | unidad ocupada |
| Reservada | Cancelada | UC-005 | hoy < desde (INV-RES-004) | unidad liberada |
| Reservada | Entregada | UC-006 | — | cuenta como prestada |
| Entregada | Devuelta | UC-007 | — | guarda `devueltaEl`; UC-008 |

Refs: INV-RES-004, INV-RES-005.

## SM-002: Multa

| State | Initial | Final | Description |
|---|---|---|---|
| Pendiente | Yes | No | Sin pagar; bloquea reservar |
| Pagada | No | Yes | Cerrada por el encargado |

| From | To | Trigger | Guard | Action |
|---|---|---|---|---|
| — | Pendiente | UC-008 | devuelta > hasta | importe = días × `FINE_RATE_EUR` |
| Pendiente | Pagada | UC-010 | encargado | si no quedan pendientes, puede reservar |

Refs: INV-FIN-001..003.
