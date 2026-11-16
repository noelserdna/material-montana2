# Operation Mapping — material-montana2

> Stack: JavaScript (ES modules) + Vite + Vitest (SDD Stack Profile de AGENTS.md; no hay kit de stack instalado) · Conventions: AGENTS.md § SDD Stack Profile, `design/TECHNICAL-DESIGN.md` §8 · Contracts: spec/contracts/API-*.md (14 operations)

Idioma de transporte: RPC JSON. Cada operación es `POST /api/<operación>` con cuerpo JSON, manejada por `src/server/http/rutas.js` (tabla operación → manejador) que llama a `src/domain/<modulo>.js`. Éxito: HTTP 200 `{ ok:true, datos }`. Error de dominio: `{ ok:false, codigo, mensaje }` con el estado de la familia (sesión → 401, rol/dato ajeno → 403, no encontrado → 404, regla de negocio → 409, entrada inválida → 400, almacén → 503). La SPA es JS-only (no hay fallback sin JS; REQ-C-002 fija una SPA Vite).

## Mapping

| API-op | Idiom | Route / action | Verb | Success | Validation error | No-JS fallback | Accessible element |
|--------|-------|----------------|------|---------|------------------|----------------|--------------------|
| API-001-01 iniciarSesion | RPC op + cookie de sesión | `POST /api/iniciarSesion` → `acceso.iniciarSesion` (`src/server/http/acceso.js`) | POST | 200, `Set-Cookie` sesión, la SPA va a Catálogo | 401 `E_AUTH_INVALID` (mismo mensaje con cuenta inexistente o clave errónea), 429 `E_AUTH_LOCKED` (cuenta bloqueada, REQ-F-017), 503 `E_STORAGE_UNAVAILABLE`; mensaje junto al formulario | JS required — REQ-C-002 | botón «Entrar» |
| API-001-02 cerrarSesion | RPC op + borrado de cookie | `POST /api/cerrarSesion` → `acceso.cerrarSesion` | POST | 200, `Set-Cookie` con `Max-Age=0`, la SPA va a Iniciar sesión; con sesión ya cerrada, 200 sin cambios | 503 `E_STORAGE_UNAVAILABLE` | JS required — REQ-C-002 | botón «Salir» |
| API-002-01 crearReserva | RPC op | `POST /api/crearReserva` → `reservas.crearReserva` | POST | 200 `{reserva}`, la SPA muestra Mis reservas | 400 `E_DATE_RANGE`; 409 `E_NO_STOCK`, `E_DUP_RESERVATION`, `E_FINE_PENDING`; 404 `E_MATERIAL_NOT_FOUND`; 403 `E_FORBIDDEN_ROLE`; mensaje VO-010 junto al formulario | JS required — REQ-C-002 | botón «Reservar» |
| API-002-02 listarMisReservas | RPC op | `POST /api/listarMisReservas` → `reservas.listarMisReservas` | POST (lectura) | 200 `{reservas}` solo del actor | 403 `E_FORBIDDEN_DATA`, 401 `E_AUTH_REQUIRED` | JS required — REQ-C-002 | lista «Mis reservas» |
| API-002-03 cancelarReserva | RPC op | `POST /api/cancelarReserva` → `reservas.cancelarReserva` | POST | 200 `{reserva}` Cancelada; repetir = 200 con aviso `O_ALREADY_CANCELLED` | 409 `E_CANCEL_TOO_LATE`; 403 `E_FORBIDDEN_DATA`/`E_FORBIDDEN_ROLE`; 404 `E_RES_NOT_FOUND` | JS required — REQ-C-002 | botón «Cancelar» por reserva |
| API-002-04 entregarReserva | RPC op | `POST /api/entregarReserva` → `reservas.entregarReserva` | POST | 200 `{reserva}` Entregada; `O_ALREADY_DELIVERED` como aviso | 409 `E_RES_CANCELLED`; 404 `E_RES_NOT_FOUND`; 403 `E_FORBIDDEN_ROLE` | JS required — REQ-C-002 | botón «Entregar» por reserva |
| API-002-05 devolverReserva | RPC op, una transacción (devolución + multa) | `POST /api/devolverReserva` → `reservas.devolverReserva` | POST | 200 `{reserva, multa?}`; `O_ALREADY_RETURNED` como aviso | 409 `E_NOT_DELIVERED`; 404 `E_RES_NOT_FOUND`; 403 `E_FORBIDDEN_ROLE`; 503 `E_STORAGE_UNAVAILABLE` sin escritura parcial | JS required — REQ-C-002 | botón «Devolver» por reserva |
| API-002-06 verHoy | RPC op | `POST /api/verHoy` → `reservas.verHoy` | POST (lectura) | 200 `{prestado, vuelven}` del día | 403 `E_FORBIDDEN_ROLE` | JS required — REQ-C-002 | tabla «Hoy» |
| API-003-01 listarMaterial | RPC op | `POST /api/listarMaterial` → `inventario.listarMaterial` | POST (lectura) | 200 `{material}` con unidades activas | 401 `E_AUTH_REQUIRED` | JS required — REQ-C-002 | tabla «Catálogo» |
| API-003-02 anadirMaterial | RPC op | `POST /api/anadirMaterial` → `inventario.anadirMaterial` | POST | 200 `{material}` | 400 `E_UNITS_MIN`; 403 `E_FORBIDDEN_ROLE` | JS required — REQ-C-002 | botón «Añadir» |
| API-003-03 darDeBajaUnidades | RPC op; no toca reservas (RN-002) | `POST /api/darDeBajaUnidades` → `inventario.darDeBajaUnidades` | POST | 200 `{material}` con la nueva cifra activa | 400 `E_UNITS_MIN`; 409 `E_UNITS_INSUFFICIENT`; 404 `E_MATERIAL_NOT_FOUND`; 403 `E_FORBIDDEN_ROLE` | JS required — REQ-C-002 | botón «Dar de baja» |
| API-004-01 listarMultas | RPC op | `POST /api/listarMultas` → `multas.listarMultas` | POST (lectura) | 200 `{multas}` o «Sin multas» | 403 `E_FORBIDDEN_ROLE` | JS required — REQ-C-002 | tabla «Multas» |
| API-004-02 marcarMultaPagada | RPC op | `POST /api/marcarMultaPagada` → `multas.marcarMultaPagada` | POST | 200 `{multa}` Pagada; `O_ALREADY_PAID` como aviso | 404 `E_FINE_NOT_FOUND`; 403 `E_FORBIDDEN_ROLE` | JS required — REQ-C-002 | botón «Marcar pagada» |
| API-004-03 verHistorialSocio | RPC op | `POST /api/verHistorialSocio` → `multas.verHistorialSocio` | POST (lectura) | 200 `{prestamos, multas}` del correo pedido | 403 `E_FORBIDDEN_DATA` (socio pidiendo otro correo) | JS required — REQ-C-002 | lista «Historial» |

Nota de coherencia: los contratos enumeran 14 operaciones (API-001-01..02, API-002-01..06, API-003-01..03, API-004-01..03); no hay operación de alta de cuentas (RN-001).

## Requirement-mandated transport

| URL / mechanic | REQ | API-op | Implemented as |
|----------------|-----|--------|----------------|
| Cookie de sesión `HttpOnly` | REQ-F-001, REQ-NF-002 | API-001-01 | `Set-Cookie: sid=…; HttpOnly; SameSite=Strict` |
| Caducidad por inactividad | REQ-F-019 | todas con sesión | 401 `E_SESSION_EXPIRED` «Tu sesión ha caducado»; la SPA redirige a Iniciar sesión con ese mensaje |
| `GET /api/salud` (no es operación de contrato) | REQ-NF-001 | — | comprobación del almacén para monitorización |

## Deviations from kit conventions

| API-op | Deviation | Reason (ADR / RN) |
|--------|-----------|-------------------|
| todas | RPC `POST /api/<operación>` también en lecturas, en lugar de REST con verbos | contratos `Style: operations`, un único formato de cuerpo, ADR-DRAFT-001 |
