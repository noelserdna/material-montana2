# API-inventario

| Field | Value |
|---|---|
| Module | API-003 |
| Style | operations |
| Auth | sesión; rol por operación en `PERMISSIONS-MATRIX.md` |
| Refs | REQ-F-003, REQ-F-011, REQ-F-012; UC-003, UC-011, UC-012; INV-INV-001, INV-INV-002; RN-002 |
| Errors | catalog in `domain/03-VALUE-OBJECTS.md` § Error catalog |
| Transport | see `design/OPERATION-MAPPING.md` |

## Operations

| ID | Operation | Actor | Input (VO) | Effect / post (INV) | Domain errors | UC |
|---|---|---|---|---|---|---|
| API-003-01 | listarMaterial | cualquier rol con sesión | — | materiales con unidades activas; sin cambios | `E_AUTH_REQUIRED` | UC-003 |
| API-003-02 | anadirMaterial | encargado | material, unidades: Unidades (VO-006) | activas += unidades (INV-INV-001) | `E_UNITS_MIN`, `E_FORBIDDEN_ROLE` | UC-011 |
| API-003-03 | darDeBajaUnidades | encargado | material, unidades: Unidades (VO-006) | activas −= unidades; reservas intactas (INV-INV-002, RN-002) | `E_UNITS_MIN`, `E_UNITS_INSUFFICIENT`, `E_MATERIAL_NOT_FOUND`, `E_FORBIDDEN_ROLE` | UC-012 |

## Errors

| Code | Operations | Condition | Message (VO) |
|---|---|---|---|
| `E_UNITS_MIN` | 02, 03 | unidades < 1 | VO-010 |
| `E_UNITS_INSUFFICIENT` | 03 | unidades > activas | VO-010 |
| `E_MATERIAL_NOT_FOUND` | 03 | material inexistente | VO-010 |
| `E_FORBIDDEN_ROLE` | 02, 03 | rol sin permiso | VO-010 |
| `E_AUTH_REQUIRED` | all | sin sesión | VO-010 |
| `E_STORAGE_UNAVAILABLE` | all | almacén no accesible | VO-010 |
