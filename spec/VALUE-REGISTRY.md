# Value Registry

| Name | Value | Unit | Category | Source | Used in (ids) |
|---|---|---|---|---|---|
| `FINE_RATE_EUR` | 2 | € por día | limit | REQ-C-003 | INV-FIN-001, UC-008, API-002-05 |
| `ROLES` | socio, encargado, junta | enum | enum | VO-002 | INV-ACC-004, INV-ACC-005, PERMISSIONS-MATRIX |
| `ESTADO_RESERVA` | Reservada, Entregada, Devuelta, Cancelada | enum | enum | VO-004 | SM-001, UC-004 |
| `ESTADO_MULTA` | Pendiente, Pagada | enum | enum | VO-005 | SM-002, UC-010, UC-014 |
| `DATE_FORMAT` | YYYY-MM-DD | día natural | format | REQ-C-003 | VO-003, UC-002 |
| `CANCEL_DEADLINE` | día anterior a `desde` | día | rule | REQ-F-005 | INV-RES-004 |
| `LOGIN_MAX_FAILURES` | 5 | intentos | limit | REQ-F-017 | INV-ACC-006, UC-001 |
| `LOGIN_WINDOW_MIN` | 15 | minutos | limit | REQ-F-017 | INV-ACC-006, UC-001 |
| `LOCKOUT_MIN` | 15 | minutos | limit | REQ-F-017 | INV-ACC-006, UC-001 |
| `SESSION_IDLE_MIN` | 30 | minutos | limit | REQ-F-019 | INV-ACC-007, UC-018 |
