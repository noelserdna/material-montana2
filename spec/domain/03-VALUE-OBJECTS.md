# 03 — Value Objects

```typescript
type Correo = string;                 // VO-001: correo de la cuenta, único
type Rol = 'socio' | 'encargado' | 'junta';   // VO-002 (ROLES)
type Dia = string;                    // VO-003: 'YYYY-MM-DD', día natural completo (REQ-C-003)
type EstadoReserva = 'Reservada' | 'Entregada' | 'Devuelta' | 'Cancelada';  // VO-004
type EstadoMulta = 'Pendiente' | 'Pagada';      // VO-005
type Instante = string;               // VO-007: fecha y hora ISO 8601; solo para sesión y bloqueo (INV-ACC-006, INV-ACC-007); el resto del dominio cuenta días (REQ-C-003)
type Unidades = number;               // VO-006: entero ≥ 1 en altas y bajas (INV-INV-001)
```

## Error catalog (VO-010)

| Code | Class | Message (literal) | Raised by | Refs |
|---|---|---|---|---|
| `E_AUTH_INVALID` | AuthError | Correo o contraseña incorrectos | API-001-01 | REQ-F-001 |
| `E_AUTH_LOCKED` | AuthError | Demasiados intentos fallidos: espera 15 minutos | API-001-01 | REQ-F-017 |
| `E_AUTH_REQUIRED` | AuthError | Iniciar sesión | all | REQ-F-016, REQ-F-018 |
| `E_FORBIDDEN_ROLE` | PermissionError | No tienes permiso para hacer esto | API-002-04..06, API-003-02..03, API-004-01..02, API-002-01, API-002-03, API-002-07 | REQ-F-015, REQ-F-006, REQ-F-010, REQ-F-020 |
| `E_FORBIDDEN_DATA` | PermissionError | No tienes permiso para ver estos datos | API-002-02, API-002-03, API-004-03 | REQ-F-005, REQ-F-016 |
| `E_DATE_RANGE` | ValidationError | La fecha de fin no puede ser anterior a la de inicio | API-002-01 | REQ-F-002 |
| `E_NO_STOCK` | ConflictError | No quedan unidades libres de {material} esas fechas | API-002-01 | REQ-F-002, REQ-F-012 |
| `E_DUP_RESERVATION` | ConflictError | Ya tienes esa reserva | API-002-01 | REQ-F-002 |
| `E_FINE_PENDING` | RuleError | No puedes reservar hasta pagar tu multa de {importe} € | API-002-01 | REQ-F-009 |
| `E_CANCEL_TOO_LATE` | RuleError | Ya no se puede cancelar: la reserva empieza hoy | API-002-03 | REQ-F-005 |
| `E_RES_NOT_FOUND` | NotFoundError | La reserva no existe | API-002-03..05 | REQ-F-005 |
| `E_RES_CANCELLED` | StateError | No se puede entregar una reserva cancelada | API-002-04 | REQ-F-006 |
| `E_NOT_DELIVERED` | StateError | Solo se puede devolver lo que se ha entregado | API-002-05 | REQ-F-007 |
| `E_CANCEL_STATE` | StateError | Solo se puede cancelar una reserva que aún no se ha entregado | API-002-03 | REQ-F-005 |
| `E_DELIVER_STATE` | StateError | Solo se puede entregar una reserva pendiente de entrega | API-002-04 | REQ-F-006 |
| `E_UNITS_MIN` | ValidationError | Indica al menos 1 unidad | API-003-02..03 | REQ-F-011 |
| `E_UNITS_INSUFFICIENT` | ValidationError | No hay tantas unidades activas de {material} | API-003-03 | REQ-F-012 |
| `E_MATERIAL_NOT_FOUND` | NotFoundError | Ese material no existe | API-002-01, API-003-03 | REQ-F-012 |
| `E_FINE_NOT_FOUND` | NotFoundError | La multa no existe | API-004-02 | REQ-F-010 |
| `E_SESSION_EXPIRED` | AuthError | Tu sesión ha caducado | all (con sesión) | REQ-F-019 |
| `E_STORAGE_UNAVAILABLE` | SystemError | No se puede acceder a los datos; inténtalo de nuevo | all | REQ-NF-001 |

## Idempotent outcomes (not errors; message literal, state unchanged)

| Code | Message | Raised by |
|---|---|---|
| `O_ALREADY_CANCELLED` | Cancelada (se mantiene) | API-002-03 |
| `O_ALREADY_DELIVERED` | Ya estaba entregada | API-002-04 |
| `O_ALREADY_RETURNED` | Ya estaba devuelta | API-002-05 |
| `O_ALREADY_PAID` | Ya estaba pagada | API-004-02 |
