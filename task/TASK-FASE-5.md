# Tasks: FASE-5 - La junta consulta y cada dato es de su dueño

> **Input:** plan/fases/FASE-5-JUNTA.md + plan/fase-plans/PLAN-FASE-5.md
> **Generated:** 2026-11-16
> **Total tasks:** 7
> **Parallel capacity:** 1 work Stream (A); base serial
> **Critical path:** 7 tasks

---

## Summary

| Metric | Value |
|--------|-------|
| Total tasks | 7 |
| Parallelizable | 0 (0%) |
| Work Streams | 1 (A: 5 tasks) |
| Setup phase | 0 tasks |
| Foundation phase | 1 task |
| Slices phase | 5 tasks |
| Integration phase | 0 tasks |
| Verification phase | 1 task |

## Traceability

| Spec Reference | Task Coverage |
|---------------|---------------|
| UC-014 | TASK-F5-002, 003, 004 |
| UC-015 | TASK-F5-004, 005 |
| UC-016 | TASK-F5-006 |
| API-004-03 | TASK-F5-002, 003 |
| INV-ACC-004 | TASK-F5-001, 003, 006 |
| INV-ACC-005 | TASK-F5-001, 005 |
| REQ-F-014 / REQ-F-015 / REQ-F-016 | TASK-F5-002..004 / TASK-F5-004, 005 / TASK-F5-006 |
| Q-spec-auditor-003 | TASK-F5-002 |

---

## Phase 2: Foundation

**Purpose:** Matriz de permisos completa y regla de dueño, que comparten historial, junta y datos propios.
**Checkpoint:** UNIT-T-003 y las pruebas de permisos en verde.

- [ ] TASK-F5-001 Complete role matrix and owner-or-staff rule, test-first | `src/server/http/permisos.js`, `tests/unit/http/permisos.test.js`
  - **Commit:** `feat(http): complete permission matrix and owner-or-staff rule`
  - **Acceptance:**
    - Test first: `UNIT-T-003 INV-ACC-005 siete escrituras` — las 7 operaciones de escritura (`crearReserva`, `cancelarReserva`, `entregarReserva`, `devolverReserva`, `anadirMaterial`, `darDeBajaUnidades`, `marcarMultaPagada`) rechazan el rol junta con `E_FORBIDDEN_ROLE`
    - Test first: `INV-ACC-004 esDuenoOStaff` — `esDuenoOStaff(actor, correo)` es verdadero para el dueño, el encargado y la junta, y falso para otro socio; una operación sin fila en `PERMISOS` se deniega
    - Test first: `AUTHZ-002 SEC-T-002` — un socio que pide un dato ajeno recibe el mismo resultado exista o no el dato
  - **Refs:** FASE-5, REQ-F-015, REQ-F-016, INV-ACC-004, INV-ACC-005, PERMISSIONS-MATRIX, AUTHZ-002, SEC-T-002
  - **Revert:** COUPLED — todas las operaciones consultan esta matriz
  - **Review:**
    - [ ] Denegar por defecto
    - [ ] Matriz igual a PERMISSIONS-MATRIX

---

## Phase 3: Slices

**Purpose:** Un corte vertical por caso de uso, con la prueba escrita antes.
**Checkpoint:** Pruebas de UC-014, UC-015 y UC-016 en verde.

### UC-014 — Historial del socio

- [ ] TASK-F5-002 Member history in the domain: delivered loans and fines, test-first | `src/domain/multas.js`, `src/server/repo/multas.js`, `tests/integration/historial-permisos.test.js`
  - blocked-by: TASK-F5-001
  - **Commit:** `feat(multas): member history with delivered loans and fines`
  - **Acceptance:**
    - Test first: `AC-014-01 REQ-F-014` — `verHistorialSocio(correo)` devuelve `{ prestamos, multas }` con las reservas que llegaron a entregarse (`Entregada` o `Devuelta`) y las multas del socio pedido
    - Test first: `AC-014-02` — un socio sin multas devuelve `multas: []` (la pantalla dice «Sin multas»)
    - Test first: `AC-014-06 Q-spec-auditor-003` — las reservas `Reservada` o `Cancelada` no salen como préstamos
    - Test first: `AC-014-03 INV-ACC-004` — un socio que pide el historial de otro recibe `E_FORBIDDEN_DATA`; pedir el suyo funciona
  - **Refs:** FASE-5, REQ-F-014, UC-014, INV-ACC-004, Q-spec-auditor-003
  - **Revert:** COUPLED — la operación HTTP TASK-F5-003 la usa
  - **Review:**
    - [ ] Solo lo entregado cuenta como préstamo
    - [ ] Lectura sin escrituras

- [ ] TASK-F5-003 Member history operation (API-004-03), test-first | `src/server/http/multas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/historial-permisos.test.js`
  - blocked-by: TASK-F5-002
  - **Commit:** `feat(multas): verHistorialSocio operation for board, encargado and owner`
  - **Acceptance:**
    - Test first: `AC-014-04 AC-014-05 INV-ACC-003` — sin sesión da `E_AUTH_REQUIRED` sin datos; pedirlo dos veces devuelve lo mismo y no cambia nada
    - Test first: `AC-014-01 AC-014-03 API-004-03` — `POST /api/verHistorialSocio` da 200 a junta y encargado para cualquier socio, y 403 `E_FORBIDDEN_DATA` a un socio que pide otro correo
  - **Refs:** FASE-5, REQ-F-014, UC-014, API-004-03, INV-ACC-003, INV-ACC-004
  - **Revert:** COUPLED — la pantalla TASK-F5-004 la usa
  - **Review:**
    - [ ] Entrada en la matriz para los tres roles
    - [ ] Mismo resultado exista o no el socio ajeno

- [ ] TASK-F5-004 Member history screen SCR-008 without write buttons for the board, and SCR-009 | `src/ui/pantallas/historial.js`, `src/ui/pantallas/sin-permiso.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js`
  - blocked-by: TASK-F5-003
  - **Commit:** `feat(ui): member history screen and no-permission screen`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/historial` pide un correo y pinta Préstamos y Multas (o «Sin multas»); con rol junta la pantalla y la cabecera no muestran ningún botón de escritura (AC-015-01, SEC-T-005)
    - `#/sin-permiso` (SCR-009) muestra el literal de `E_FORBIDDEN_ROLE`/`E_FORBIDDEN_DATA`; comprobado por el journey TASK-F5-007
  - **Refs:** FASE-5, REQ-F-014, REQ-F-015, UC-014, UC-015, WIREFRAMES SCR-008, SCR-009
  - **Revert:** SAFE — pantallas nuevas
  - **Review:**
    - [ ] La junta no ve controles de escritura
    - [ ] El servidor sigue siendo quien rechaza

### UC-015 — La junta solo consulta

- [ ] TASK-F5-005 Board is read-only on the server, test-first | `src/server/http/rutas.js`, `tests/integration/historial-permisos.test.js`
  - blocked-by: TASK-F5-004
  - **Commit:** `test(permisos): board role is rejected on every write operation`
  - **Acceptance:**
    - Test first: `AC-015-02 AC-015-03 INV-ACC-005` — con rol junta, `marcarMultaPagada` y `crearReserva` dan `E_FORBIDDEN_ROLE` (403) y no cambian nada
    - Test first: `AC-015-04` — repetir el intento da el mismo resultado
    - Test first: `AC-015-01 REQ-F-015 UNIT-T-003` — por HTTP, las 7 operaciones de escritura rechazan a la junta; la junta puede leer historial, catálogo y Mis reservas
    - `rutas.js` aplica la matriz antes de llamar al dominio; cualquier ajuste va en esta tarea
  - **Refs:** FASE-5, REQ-F-015, UC-015, INV-ACC-005, SEC-T-005
  - **Revert:** SAFE — prueba y comprobación de rol
  - **Review:**
    - [ ] Ninguna escritura queda sin comprobar
    - [ ] Sin cambios en el almacén tras el rechazo

### UC-016 — Datos solo para su dueño

- [ ] TASK-F5-006 Data only for its owner across reservations, history and fines, test-first | `src/server/http/multas.js`, `src/server/http/permisos.js`, `tests/integration/historial-permisos.test.js`
  - blocked-by: TASK-F5-005
  - **Commit:** `test(permisos): every member's data is visible only to its owner or staff`
  - **Acceptance:**
    - Test first: `AC-016-01 AC-016-02 REQ-F-016` — lucia no lee las reservas ni el historial de pablo, y al revés
    - Test first: `AC-016-03 API-004-01` — el encargado ve las multas de todos
    - Test first: `AC-016-04 AC-016-05 INV-ACC-003` — sin sesión no hay datos; repetir la petición da lo mismo
    - Test first: `AUTHZ-002 SEC-T-002` — pedir un dato ajeno da el mismo resultado exista o no
  - **Refs:** FASE-5, REQ-F-016, UC-016, INV-ACC-004, API-004-01, AUTHZ-002
  - **Revert:** SAFE — prueba y ajustes de permisos
  - **Review:**
    - [ ] Probado en ambos sentidos entre dos socios
    - [ ] Sin pistas sobre si el dato existe

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-5 verificados.

- [ ] TASK-F5-007 Journey FASE-5: la junta consulta y cada dato es de su dueño through the user's route | `tests/e2e/fase-5.spec.js`
  - blocked-by: TASK-F5-006
  - **Commit:** `test(e2e): FASE-5 journey with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-014-01 …`, `AC-014-02`, `AC-014-03`, `AC-014-04`, `AC-014-05`, `AC-014-06`, `AC-015-01`, `AC-015-02`, `AC-015-03`, `AC-015-04`, `AC-016-01`, `AC-016-02`, `AC-016-03`, `AC-016-04`, `AC-016-05` (REQ-F-014, REQ-F-015, REQ-F-016)
    - El título del journey lleva `FASE-5` (la FASE no nombra workflow): es el id del vídeo y lo que busca `sdd gate --fase 5`
    - Entra por el paso 1 de la Demo: ana (junta) abre Historial del socio y escribe `lucia@club.es`, nunca por un punto de entrada interno
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra: el préstamo entregado y «Pendiente» en la multa, «Sin multas», la ausencia de botones de escritura para la junta, el literal de `E_FORBIDDEN_DATA`, `E_FORBIDDEN_ROLE`
    - Guarda `evidencias/FASE-5/{AC-NNN-NN}.png` por criterio y un vídeo con `FASE-5` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-5, REQ-F-014, REQ-F-015, REQ-F-016, AC-014-01..06, AC-015-01..04, AC-016-01..05
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Cada escenario con un `test` que lleva su id
    - [ ] Capturas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/server/repo/multas.js` | consultas simples | `tests/integration/historial-permisos.test.js` |
| `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js` | constantes y cableado | MNT-T-001 y journey TASK-F5-007 |
| `src/ui/pantallas/historial.js`, `src/ui/pantallas/sin-permiso.js` | pantallas | journey TASK-F5-007 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─► 002 ─► 003 ─► 004 ─► 005 ─► 006 ─► 007
```

### Critical Path

1. TASK-F5-001 → 002 → 003 → 004 → 005 → 006 → 007
   (7 tasks on critical path)

### Parallel Execution Plan

**Stream A:** historial, junta de solo lectura y datos de su dueño, en serie (comparten `multas.js`, `rutas.js` y `historial-permisos.test.js`).

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | TASK-F5-001 | src/server/http/permisos.js, tests/unit/http/permisos.test.js | main checkout, before worktrees (checkpoint `fase-5-foundation`) |
| A | TASK-F5-002, TASK-F5-003, TASK-F5-004, TASK-F5-005, TASK-F5-006 | src/domain/multas.js, src/server/repo/multas.js, src/server/http/multas.js, src/server/http/rutas.js, src/shared/textos.js, src/ui/pantallas/historial.js, src/ui/pantallas/sin-permiso.js, src/ui/router.js, src/ui/cabecera.js, tests/integration/historial-permisos.test.js | worktree `feat/fase-5-a` |
| integración | — | — | main checkout, after `--integrate --fase 5` |
| verificación | TASK-F5-007 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | TASK-F5-001 | `fase-5-foundation` | main checkout |
| Verified | TASK-F5-007 | `fase-5-verified` | main checkout |
