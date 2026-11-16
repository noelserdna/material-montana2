# API-multas

| Field | Value |
|---|---|
| Module | API-004 |
| Style | operations |
| Auth | sesión; rol por operación en `PERMISSIONS-MATRIX.md` |
| Refs | REQ-F-008, REQ-F-010, REQ-F-014, REQ-F-016; UC-008, UC-010, UC-014, UC-016; INV-FIN-001..003, INV-ACC-004 |
| Errors | catalog in `domain/03-VALUE-OBJECTS.md` § Error catalog |
| Transport | see `design/OPERATION-MAPPING.md` |

## Operations

| ID | Operation | Actor | Input (VO) | Effect / post (INV) | Domain errors | UC |
|---|---|---|---|---|---|---|
| API-004-01 | listarMultas | encargado | — | multas de todos con correo, importe, estado; "Sin multas" si no hay | `E_FORBIDDEN_ROLE`, `E_AUTH_REQUIRED` | UC-008, UC-016 |
| API-004-02 | marcarMultaPagada | encargado | multaId | `Pagada`; otras multas intactas (INV-FIN-003) | `E_FINE_NOT_FOUND`, `E_FORBIDDEN_ROLE`; `O_ALREADY_PAID` | UC-010 |
| API-004-03 | verHistorialSocio | junta, encargado (el socio solo el suyo) | correo: Correo (VO-001) | préstamos y multas del socio; sin cambios (INV-ACC-004) | `E_FORBIDDEN_DATA`, `E_AUTH_REQUIRED` | UC-014, UC-016 |

## Errors

| Code | Operations | Condition | Message (VO) |
|---|---|---|---|
| `E_FINE_NOT_FOUND` | 02 | multa inexistente | VO-010 |
| `E_FORBIDDEN_ROLE` | 01, 02 | rol sin permiso | VO-010 |
| `E_FORBIDDEN_DATA` | 03 | datos de otro socio | VO-010 |
| `E_AUTH_REQUIRED` | all | sin sesión | VO-010 |
| `E_STORAGE_UNAVAILABLE` | all | almacén no accesible | VO-010 |
