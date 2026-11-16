# Security

| Field | Value |
|---|---|
| Refs | REQ-NF-002, REQ-F-016..019; UC-001, UC-017, UC-018; INV-ACC-001, INV-ACC-003, INV-ACC-004, INV-ACC-006, INV-ACC-007 |

| Rule | Value | Verification |
|---|---|---|
| Contraseñas solo como hash con sal | lo guardado no contiene el texto y difiere entre cuentas iguales | AC-001-06 |
| Acceso a datos ajenos | solo dueño, encargado y junta | AC-016-01..04 |
| Bloqueo tras fallos de acceso | 5 fallos en 15 min bloquean la cuenta 15 min (`LOGIN_MAX_FAILURES`, `LOGIN_WINDOW_MIN`, `LOCKOUT_MIN`) | AC-001-07..11 |
| Cierre de sesión | botón "Salir" termina la sesión | AC-017-01..03 |
| Caducidad por inactividad | 30 min sin actividad (`SESSION_IDLE_MIN`), renovados con cada operación | AC-018-01..04 |
