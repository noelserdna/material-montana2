# Availability — datos compartidos

| Field | Value |
|---|---|
| Refs | REQ-NF-001; UC-002, UC-006; WF-001 |

| Rule | Value | Verification |
|---|---|---|
| Reservas, inventario, multas y cuentas viven en un almacén de servidor leído y escrito por todos | una escritura de una sesión la ve la siguiente petición de otra | BDD AC-NF1-01 (UC-006 con reserva de otra sesión) |
| Fallo del almacén | `E_STORAGE_UNAVAILABLE`, sin escritura parcial | UC-008 AC-008-05 |

AC-NF1-01: Given lucia reservó "Casco" 2026-12-10 a 2026-12-12 en una sesión; When el encargado abre Entregas y devoluciones en otra; Then ve "Casco" "lucia@club.es".
