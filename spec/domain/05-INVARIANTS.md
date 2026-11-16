# 05 — Invariants

> Areas: ACC, RES, FIN, INV. Enforcement: API | REPO.

| ID | Rule | Enforced at | UCs | Violation error | Validation |
|---|---|---|---|---|---|
| INV-ACC-001 | La contraseña se guarda solo como hash con sal; dos cuentas con la misma contraseña difieren en lo guardado | REPO | UC-001 | — | stored ≠ plain |
| INV-ACC-002 | Una cuenta tiene como mucho una sesión activa; repetir el acceso la mantiene | API | UC-001, UC-017 | — | sesiones por correo ≤ 1 |
| INV-ACC-006 | 5 intentos fallidos de una cuenta en 15 minutos la bloquean: durante 15 minutos desde el último fallo no se abre sesión, ni con la contraseña correcta; un acceso correcto pone el contador a cero; el bloqueo es por cuenta (REQ-F-017) | API | UC-001 | `E_AUTH_LOCKED` | `fallos(15 min) ≥ LOGIN_MAX_FAILURES → rechazo` |
| INV-ACC-007 | La sesión termina al elegir "Salir" o tras 30 minutos sin actividad; cada operación con sesión renueva los 30 minutos; una sesión terminada no vale (REQ-F-018, REQ-F-019) | API | UC-017, UC-018 | `E_SESSION_EXPIRED` | `ahora − ultimaActividad ≤ SESSION_IDLE_MIN` y no cerrada |
| INV-ACC-003 | Toda operación requiere sesión vigente (INV-ACC-007); sin ella se envía a Iniciar sesión sin datos | API | UC-001, UC-002, UC-003, UC-004, UC-016, UC-017, UC-018 | `E_AUTH_REQUIRED` | sesión presente |
| INV-ACC-004 | Los datos de un socio (reservas, préstamos, multas) solo los ven su dueño, el encargado y la junta | API | UC-004, UC-005, UC-014, UC-016, UC-019 | `E_FORBIDDEN_DATA` | `actor = dueño ∨ rol ∈ {encargado, junta}` |
| INV-ACC-005 | La junta no puede reservar, cancelar, entregar, devolver, cambiar inventario ni marcar multas | API | UC-002, UC-006, UC-007, UC-010, UC-011, UC-012, UC-013, UC-015, UC-019 | `E_FORBIDDEN_ROLE` | `rol ≠ junta` en toda escritura |
| INV-RES-001 | Cada día del rango: reservas activas (Reservada/Entregada) del material < unidades activas al crear | API, REPO | UC-002, UC-005, UC-012 | `E_NO_STOCK` | ocupadas(d) < activas ∀ d ∈ [desde, hasta] |
| INV-RES-002 | `hasta` ≥ `desde`; los días son naturales completos | API | UC-002 | `E_DATE_RANGE` | `hasta >= desde` |
| INV-RES-003 | Un socio no tiene dos reservas activas del mismo material y mismas fechas | API | UC-002 | `E_DUP_RESERVATION` | único (socio, material, desde, hasta) activas |
| INV-RES-004 | Solo se cancela si hoy < `desde`, y solo el dueño; solo se cancela lo Reservada | API | UC-005 | `E_CANCEL_TOO_LATE`, `E_FORBIDDEN_DATA`, `E_CANCEL_STATE` | `estado = Reservada` y `hoy < desde` |
| INV-RES-005 | Se entrega solo lo Reservada; se devuelve solo lo Entregada | API | UC-006, UC-007, UC-013 | `E_RES_CANCELLED`, `E_DELIVER_STATE`, `E_NOT_DELIVERED` | transiciones de SM-001 |
| INV-FIN-001 | Multa = días de retraso × `FINE_RATE_EUR`; solo si `devueltaEl` > `hasta`; días = `devueltaEl − hasta` | API | UC-008 | — | `dias > 0 → importe = dias × FINE_RATE_EUR` |
| INV-FIN-002 | Como mucho una multa por reserva; repetir la devolución no crea otra | API, REPO | UC-007, UC-008 | — | único `reservaId` |
| INV-FIN-003 | Con alguna multa Pendiente el socio no crea reservas; las existentes no cambian | API | UC-002, UC-009, UC-010 | `E_FINE_PENDING` | `count(Pendiente) = 0` |
| INV-INV-001 | Alta y baja piden al menos 1 unidad | API | UC-011, UC-012 | `E_UNITS_MIN` | `n ≥ 1` |
| INV-INV-002 | No se dan de baja más unidades que las activas; la baja no toca reservas ya entregadas ni cancela otras (RN-002) | API | UC-003, UC-012 | `E_UNITS_INSUFFICIENT` | `n ≤ activas` |
