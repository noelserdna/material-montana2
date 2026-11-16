# API-acceso

| Field | Value |
|---|---|
| Module | API-001 |
| Style | operations |
| Auth | ninguna para iniciar sesión; el resto exige sesión (INV-ACC-003) |
| Refs | REQ-F-001, REQ-F-017, REQ-F-018, REQ-F-019, REQ-NF-002; UC-001, UC-017, UC-018; INV-ACC-001..003, INV-ACC-006, INV-ACC-007; RN-001, RN-005 |
| Errors | catalog in `domain/03-VALUE-OBJECTS.md` § Error catalog |
| Transport | see `design/OPERATION-MAPPING.md` |

## Operations

| ID | Operation | Actor | Input (VO) | Effect / post (INV) | Domain errors | UC |
|---|---|---|---|---|---|---|
| API-001-01 | iniciarSesion | cualquiera | correo: Correo (VO-001), contrasena | sesión única de la cuenta (INV-ACC-002); contraseña contrastada con el hash (INV-ACC-001) | `E_AUTH_INVALID`, `E_AUTH_LOCKED`, `E_STORAGE_UNAVAILABLE` | UC-001 |
| API-001-02 | cerrarSesion | socio, encargado, junta | — (sesión actual) | la sesión termina y no vale más (INV-ACC-007) | `E_STORAGE_UNAVAILABLE` | UC-017 |

No hay operación de alta de cuentas: las cuentas existen antes del primer acceso y las prepara el encargado fuera de la web (RN-001).

## Errors

| Code | Operations | Condition | Message (VO) |
|---|---|---|---|
| `E_AUTH_INVALID` | 01 | par correo/contraseña no coincide o vacío | VO-010 |
| `E_AUTH_LOCKED` | 01 | 5 fallos en 15 min de esa cuenta y no han pasado 15 min desde el último (INV-ACC-006) | VO-010 |
| `E_SESSION_EXPIRED` | todas las que exigen sesión | 30 min sin actividad (INV-ACC-007, UC-018) | VO-010 |
| `E_STORAGE_UNAVAILABLE` | 01, 02 | almacén no accesible | VO-010 |
