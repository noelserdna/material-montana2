# API-reservas

| Field | Value |
|---|---|
| Module | API-002 |
| Style | operations |
| Auth | sesión; rol por operación en `PERMISSIONS-MATRIX.md` |
| Refs | REQ-F-002, REQ-F-004..009, REQ-F-013, REQ-F-015, REQ-F-016, REQ-F-020; UC-002, UC-004..007, UC-009, UC-013, UC-019; WF-001; INV-RES-001..005, INV-FIN-001..003 |
| Errors | catalog in `domain/03-VALUE-OBJECTS.md` § Error catalog |
| Transport | see `design/OPERATION-MAPPING.md` |

## Operations

| ID | Operation | Actor | Input (VO) | Effect / post (INV) | Domain errors | UC |
|---|---|---|---|---|---|---|
| API-002-01 | crearReserva | socio, encargado | material, desde: Dia, hasta: Dia (VO-003) | reserva `Reservada` (INV-RES-001..003, INV-FIN-003) | `E_DATE_RANGE`, `E_NO_STOCK`, `E_DUP_RESERVATION`, `E_FINE_PENDING`, `E_MATERIAL_NOT_FOUND`, `E_FORBIDDEN_ROLE` | UC-002, UC-009 |
| API-002-02 | listarMisReservas | socio, encargado | — | solo reservas del actor; sin cambios (INV-ACC-004) | `E_FORBIDDEN_DATA`, `E_AUTH_REQUIRED` | UC-004, UC-016 |
| API-002-03 | cancelarReserva | socio (dueño) | reservaId | `Cancelada`, unidad libre (INV-RES-004) | `E_CANCEL_TOO_LATE`, `E_CANCEL_STATE`, `E_FORBIDDEN_DATA`, `E_FORBIDDEN_ROLE`, `E_RES_NOT_FOUND`; `O_ALREADY_CANCELLED` | UC-005 |
| API-002-04 | entregarReserva | encargado | reservaId | `Entregada` (INV-RES-005) | `E_RES_CANCELLED`, `E_DELIVER_STATE`, `E_RES_NOT_FOUND`, `E_FORBIDDEN_ROLE`; `O_ALREADY_DELIVERED` | UC-006 |
| API-002-05 | devolverReserva | encargado | reservaId, devueltaEl: Dia | `Devuelta`; multa si hay retraso (INV-FIN-001, INV-FIN-002) | `E_NOT_DELIVERED`, `E_RES_NOT_FOUND`, `E_FORBIDDEN_ROLE`, `E_STORAGE_UNAVAILABLE`; `O_ALREADY_RETURNED` | UC-007, UC-008 |
| API-002-06 | verHoy | encargado | dia?: Dia | prestado y vuelve del día; sin cambios | `E_FORBIDDEN_ROLE`, `E_AUTH_REQUIRED` | UC-013 |
| API-002-07 | listarReservas | encargado | filtro?: `Por entregar` \| `Entregadas` \| `Todas` (por defecto `Por entregar`) | reservas de todos los socios según el filtro; sin cambios; solo encargado (INV-ACC-005) | `E_FORBIDDEN_ROLE`, `E_AUTH_REQUIRED` | UC-019 |

## Errors

| Code | Operations | Condition | Message (VO) |
|---|---|---|---|
| `E_DATE_RANGE` | 01 | `hasta` < `desde` | VO-010 |
| `E_NO_STOCK` | 01 | algún día sin unidad libre | VO-010 |
| `E_DUP_RESERVATION` | 01 | mismo socio, material y fechas | VO-010 |
| `E_FINE_PENDING` | 01 | multa Pendiente del socio | VO-010 |
| `E_MATERIAL_NOT_FOUND` | 01 | material inexistente | VO-010 |
| `E_CANCEL_TOO_LATE` | 03 | hoy ≥ `desde` | VO-010 |
| `E_CANCEL_STATE` | 03 | reserva Entregada o Devuelta | VO-010 |
| `E_RES_CANCELLED` | 04 | reserva Cancelada | VO-010 |
| `E_DELIVER_STATE` | 04 | reserva Devuelta | VO-010 |
| `E_NOT_DELIVERED` | 05 | no estaba Entregada | VO-010 |
| `E_RES_NOT_FOUND` | 03, 04, 05 | reserva inexistente | VO-010 |
| `E_FORBIDDEN_ROLE` | 01, 03, 04, 05, 06, 07 | rol sin permiso | VO-010 |
| `E_FORBIDDEN_DATA` | 02, 03 | reserva ajena | VO-010 |
| `E_AUTH_REQUIRED` | all | sin sesión | VO-010 |
| `E_STORAGE_UNAVAILABLE` | all | almacén no accesible | VO-010 |
