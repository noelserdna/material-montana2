# API-acceso

| Field | Value |
|---|---|
| Module | API-001 |
| Style | operations |
| Auth | ninguna para iniciar sesión; el resto exige sesión (INV-ACC-003) |
| Refs | REQ-F-001, REQ-NF-002; UC-001; INV-ACC-001..003; RN-001 |
| Errors | catalog in `domain/03-VALUE-OBJECTS.md` § Error catalog |
| Transport | see `design/OPERATION-MAPPING.md` |

## Operations

| ID | Operation | Actor | Input (VO) | Effect / post (INV) | Domain errors | UC |
|---|---|---|---|---|---|---|
| API-001-01 | iniciarSesion | cualquiera | correo: Correo (VO-001), contrasena | sesión única de la cuenta (INV-ACC-002); contraseña contrastada con el hash (INV-ACC-001) | `E_AUTH_INVALID`, `E_STORAGE_UNAVAILABLE` | UC-001 |

No hay operación de alta de cuentas: las cuentas existen antes del primer acceso y las prepara el encargado fuera de la web (RN-001).

## Errors

| Code | Operations | Condition | Message (VO) |
|---|---|---|---|
| `E_AUTH_INVALID` | 01 | par correo/contraseña no coincide o vacío | VO-010 |
| `E_STORAGE_UNAVAILABLE` | 01 | almacén no accesible | VO-010 |
