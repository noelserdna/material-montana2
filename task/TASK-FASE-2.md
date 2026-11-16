# Tasks: FASE-2 - Entregar y recibir el material

> **Input:** plan/fases/FASE-2-ENTREGAS.md + plan/fase-plans/PLAN-FASE-2.md
> **Generated:** 2026-11-16
> **Total tasks:** 8
> **Parallel capacity:** 1 work Stream (A); base serial
> **Critical path:** 7 tasks

---

## Summary

| Metric | Value |
|--------|-------|
| Total tasks | 8 |
| Parallelizable | 0 (0%) |
| Work Streams | 1 (A: 6 tasks) |
| Setup phase | 0 tasks |
| Foundation phase | 1 task |
| Slices phase | 6 tasks |
| Integration phase | 0 tasks |
| Verification phase | 1 task |

> **Cambio REQ-F-020:** el plan de la FASE ya define `listarReservas` (API-002-07, UC-019). TASK-F2-005 deja de ser `[PLAN GAP]` y pasa a ser su corte vertical. Source: CASCADE-MANUAL.

## Traceability

| Spec Reference | Task Coverage |
|---------------|---------------|
| UC-019 / API-002-07 | TASK-F2-005 |
| UC-006 | TASK-F2-002 |
| UC-007 | TASK-F2-003 |
| UC-005 (AC-005-08) | TASK-F2-004 |
| API-002-04 / API-002-05 | TASK-F2-002 / TASK-F2-003 |
| SM-001, INV-RES-005 | TASK-F2-001, 002, 003 |
| INV-RES-004 | TASK-F2-004 |
| INV-ACC-005 | TASK-F2-002, 003 |
| INCIDENT-001 | TASK-F2-001, 002, 003 |
| REQ-F-006 / REQ-F-007 | TASK-F2-002, 006 / TASK-F2-003, 006 |
| REQ-F-020 | TASK-F2-005, 006, 008 |
| REQ-NF-001 | TASK-F2-005, 007 |
| INV-ACC-005 (listar) | TASK-F2-005 |

---

## Phase 2: Foundation

**Purpose:** La tabla de transiciones SM-001 y el registro de seguridad, que comparten entregar y devolver.
**Checkpoint:** UNIT-T-009 y la prueba del registro en verde.

- [ ] TASK-F2-001 SM-001 transition table and security log, test-first | `src/domain/reservas.js`, `src/server/log-seguridad.js`, `tests/unit/domain/reservas-estados.test.js`, `tests/unit/log-seguridad.test.js`
  - **Commit:** `feat(reservas): SM-001 transition table and security log`
  - **Acceptance:**
    - Test first: `UNIT-T-009 SM-001 transiciones` — `transicion('Reservada','entregar')` → `Entregada`; `transicion('Entregada','devolver')` → `Devuelta`; repetir una acción ya aplicada → `{ aviso }` (`O_ALREADY_DELIVERED`, `O_ALREADY_RETURNED`); `Cancelada` + entregar → `E_RES_CANCELLED`; `Devuelta` + entregar → `E_DELIVER_STATE`; `Reservada` + devolver → `E_NOT_DELIVERED`; `Entregada` + cancelar → `E_CANCEL_STATE`
    - Test first: `INCIDENT-001 registro` — `registrar({ actor, operacion, id, instante })` escribe una línea JSON por llamada en stdout sin contraseñas ni tokens
    - La tabla es única: ningún otro módulo repite las reglas de transición
  - **Refs:** FASE-2, REQ-F-006, REQ-F-007, SM-001, INV-RES-005, INCIDENT-001
  - **Revert:** SAFE — funciones nuevas sin consumidores todavía
  - **Review:**
    - [ ] Una sola tabla de transiciones
    - [ ] El log no contiene datos sensibles

---

## Phase 3: Slices

**Purpose:** Un corte vertical por operación, con la prueba escrita antes.
**Checkpoint:** Pruebas de UC-019, UC-006, UC-007 y AC-005-08 en verde.

### UC-006 — Entregar lo reservado

- [ ] TASK-F2-002 Deliver a reservation (API-002-04), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/entregas.test.js`
  - blocked-by: TASK-F2-001
  - **Commit:** `feat(reservas): deliver a reservation, encargado only`
  - **Acceptance:**
    - Test first: `AC-006-01 AC-006-02 REQ-F-006` — entregar una `Reservada` la pasa a `Entregada` y anota actor, operación e id en el registro de seguridad
    - Test first: `AC-006-03 O_ALREADY_DELIVERED` — entregar otra vez da 200 con aviso y sin cambios
    - Test first: `AC-006-04 AC-006-07 Q-spec-auditor-001` — una reserva cancelada da `E_RES_CANCELLED` y una ya devuelta `E_DELIVER_STATE` (409), ambas sin cambios
    - Test first: `AC-006-05 AC-006-06 INV-ACC-005` — un socio recibe `E_FORBIDDEN_ROLE`; una reserva inexistente `E_RES_NOT_FOUND` (404)
  - **Refs:** FASE-2, REQ-F-006, UC-006, API-002-04, SM-001, INV-RES-005, INV-ACC-005, Q-spec-auditor-001
  - **Revert:** COUPLED — comparte `reservas.js` con 003 y 004
  - **Review:**
    - [ ] Error o aviso sin efectos
    - [ ] Rol comprobado en servidor

### UC-007 — Marcar la devolución

- [ ] TASK-F2-003 Mark a reservation as returned (API-002-05), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/entregas.test.js`
  - blocked-by: TASK-F2-002
  - **Commit:** `feat(reservas): mark the return with its date, in one transaction`
  - **Acceptance:**
    - Test first: `AC-007-01 REQ-F-007 devolución` — devolver una `Entregada` con `devueltaEl` la pasa a `Devuelta`; `devueltaEl` se acepta tal cual, sin comprobaciones de fechas raras (Q-spec-auditor-002)
    - Test first: `AC-007-02 INV-RES-005 O_ALREADY_RETURNED` — devolver otra vez da el aviso, sin cambios
    - Test first: `AC-007-03 E_NOT_DELIVERED` — devolver lo nunca entregado da `E_NOT_DELIVERED` y no cambia
    - Test first: `AC-007-04 INV-RES-001` — la unidad devuelta vuelve a estar libre: la misma reserva se acepta de nuevo (se calcula desde `ocupadas`)
    - Test first: `AC-007-05 AC-007-06` — un socio recibe `E_FORBIDDEN_ROLE`; una reserva inexistente `E_RES_NOT_FOUND`
    - La devolución se hace en una sola transacción (D-PA-003: FASE-3 le suma la multa) y se anota en el registro de seguridad; sin multa todavía
  - **Refs:** FASE-2, REQ-F-007, UC-007, API-002-05, SM-001, INV-RES-001, INV-RES-005, D-PA-003, Q-spec-auditor-002
  - **Revert:** COUPLED — FASE-3 amplía esta operación
  - **Review:**
    - [ ] Una transacción para la devolución
    - [ ] Sin multa (llega en FASE-3)

### UC-005 — Cancelar tras entregar

- [ ] TASK-F2-004 Refuse to cancel a delivered reservation, test-first | `src/domain/reservas.js`, `src/shared/textos.js`, `tests/integration/entregas.test.js`
  - blocked-by: TASK-F2-003
  - **Commit:** `fix(reservas): cancelling a delivered reservation gives E_CANCEL_STATE`
  - **Acceptance:**
    - Test first: `AC-005-08 Q-spec-auditor-001 E_CANCEL_STATE` — cancelar una reserva `Entregada` da `E_CANCEL_STATE` (409) y no cambia nada; los casos de FASE-1 (AC-005-01..07) siguen en verde
  - **Refs:** FASE-2, REQ-F-005, UC-005, INV-RES-004, SM-001, Q-spec-auditor-001
  - **Revert:** SAFE — una rama del dominio
  - **Review:**
    - [ ] Usa la tabla SM-001
    - [ ] No altera los códigos de FASE-1

### UC-019 — Ver las reservas de todos los socios

- [ ] TASK-F2-005 List every member's reservations for the encargado (API-002-07), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/listar-reservas.test.js`
  - blocked-by: TASK-F2-004
  - **Commit:** `feat(reservas): list every member's reservations, encargado only`
  - **Acceptance:**
    - Test first: `AC-019-01 REQ-F-020 AC1` — sin filtro, `listarReservas` devuelve solo las `Reservada` de todos los socios, cada una con material y correo del socio (filtro por defecto `Por entregar`)
    - Test first: `AC-019-02 AC-019-03 AC-019-08` — los filtros `Entregadas` y `Todas` (las cuatro estados) cambian la lista y no modifican ningún dato
    - Test first: `AC-019-04 REQ-F-020 AC4` — sin coincidencias devuelve `reservas: []` y el mensaje «No hay reservas»; un filtro desconocido da 400
    - Test first: `AC-019-05 AC-019-06 AC-019-07 INV-ACC-005` — socio y junta reciben `E_FORBIDDEN_ROLE` antes de leer datos y sin ninguna reserva; sin sesión `E_AUTH_REQUIRED`
    - Proyección por estado sobre la tabla `reserva`, sin índice ni esquema nuevos (D-PA-012); `tests/integration/entregas.test.js` queda en el Stream de 002..004, esta prueba va en su propio fichero
  - **Refs:** FASE-2, REQ-F-020, REQ-NF-001, UC-019, API-002-07, INV-ACC-005, D-PA-012
  - **Revert:** COUPLED — comparte `reservas.js`, `rutas.js` y `textos.js` con 002..004
  - **Review:**
    - [ ] Rol comprobado antes de leer datos
    - [ ] Lectura pura: sin efectos ni registro de escritura

### REQ-NF-001 — Datos compartidos y pantalla

- [ ] TASK-F2-006 Deliveries and returns screen SCR-004 with the shared reservations table | `src/ui/pantallas/entregas.js`, `src/ui/tabla-reservas.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js`
  - blocked-by: TASK-F2-002, TASK-F2-003, TASK-F2-005
  - **Commit:** `feat(ui): deliveries and returns screen for the encargado`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/entregas` pinta la tabla con material, correo, fechas y estado de las reservas de todos (por defecto «Por entregar»; filtro «Entregadas» / «Todas»; «No hay reservas» si vacío), botón Entregar y campo «Devuelta el» con botón Devolver; un socio o la junta que llega ve SCR-009 (sin permiso)
    - Los avisos `O_ALREADY_DELIVERED` y los errores `E_CANCEL_STATE` salen en la región de mensaje; comprobado por el journey TASK-F2-008
  - **Refs:** FASE-2, REQ-F-006, REQ-F-007, REQ-F-020, UC-019, UC-006, UC-007, AC-019-01, AC-019-04, WIREFRAMES SCR-004
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] Solo el encargado ve los botones
    - [ ] Tabla reutilizable en Mis reservas

- [ ] TASK-F2-007 Shared server store between two sessions, test-first | `tests/integration/entregas.test.js`
  - blocked-by: TASK-F2-005
  - **Commit:** `test(reservas): reservation made in one session is visible to the encargado in another`
  - **Acceptance:**
    - Test first: `REQ-NF-001 AC1 INT-T-001 AC-NF1-01` — con dos sesiones sobre el mismo almacén, lo que reserva lucia lo ve marta en la petición siguiente, también tras reabrir el almacén
  - **Refs:** FASE-2, REQ-NF-001, INT-T-001, AC-NF1-01
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Dos sesiones y cookies distintas
    - [ ] Sin esperas reales

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-2 verificados.

- [ ] TASK-F2-008 Journey FASE-2 WF-001: entregar y recibir el material through the user's route | `tests/e2e/fase-2.spec.js`
  - blocked-by: TASK-F2-006, TASK-F2-007
  - **Commit:** `test(e2e): FASE-2 journey WF-001 with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-006-01 …`, `AC-006-02`, `AC-006-03`, `AC-006-04`, `AC-006-05`, `AC-006-06`, `AC-006-07`, `AC-007-01`, `AC-007-02`, `AC-007-03`, `AC-007-04`, `AC-007-05`, `AC-007-06`, `AC-019-01`, `AC-019-02`, `AC-019-03`, `AC-019-04`, `AC-019-05`, `AC-019-06`, `AC-019-07`, `AC-019-08`, `AC-005-08`, `REQ-NF-001 AC1` (REQ-F-006, REQ-F-007, REQ-F-020, REQ-NF-001)
    - El título del journey lleva `WF-001`: es el id del vídeo y lo que busca `sdd gate --fase 2`
    - Entra por el paso 1 de la Demo: lucia reserva «Casco» en un navegador y marta abre `#/entregas` en otro; nunca por un punto de entrada interno
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra: «Casco» y «lucia@club.es» en la tabla de marta (filtros «Por entregar», «Entregadas», «Todas»), «No hay reservas» en la lista vacía, el literal de `E_FORBIDDEN_ROLE` para un socio, «Entregada», el aviso de ya entregada, el literal de `E_CANCEL_STATE`, «Devuelta», el literal de `E_NOT_DELIVERED`
    - Guarda `evidencias/FASE-2/{AC-NNN-NN | REQ-NF-001-AC1}.png` por criterio y un vídeo con `WF-001` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-2, REQ-F-006, REQ-F-007, REQ-F-020, REQ-NF-001, WF-001, AC-006-01..07, AC-007-01..06, AC-019-01..08, AC-005-08
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Dos contextos de navegador distintos
    - [ ] Capturas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js` | constantes y cableado | MNT-T-001 y journey TASK-F2-008 |
| `src/ui/pantallas/entregas.js`, `src/ui/tabla-reservas.js` | pantalla y componente | journey TASK-F2-008 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─► 002 ─► 003 ─► 004 ─► 005 ─┬─► 006 ─┐
                                  └─► 007 ─┴─► 008
```

### Critical Path

1. TASK-F2-001 → 002 → 003 → 004 → 005 → 006 → 008
   (7 tasks on critical path)

### Parallel Execution Plan

**Stream A:** entregar, devolver, cancelar entregada, listar reservas, pantalla y prueba de datos compartidos, en serie (comparten `reservas.js`, `rutas.js` y `textos.js`).

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | TASK-F2-001 | src/server/log-seguridad.js, tests/unit/domain/reservas-estados.test.js, tests/unit/log-seguridad.test.js | main checkout, before worktrees (checkpoint `fase-2-foundation`) |
| A | TASK-F2-002, TASK-F2-003, TASK-F2-004, TASK-F2-005, TASK-F2-006, TASK-F2-007 | src/domain/reservas.js, src/server/http/reservas.js, src/server/http/rutas.js, src/shared/textos.js, src/ui/pantallas/entregas.js, src/ui/tabla-reservas.js, src/ui/router.js, src/ui/cabecera.js, tests/integration/entregas.test.js, tests/integration/listar-reservas.test.js | worktree `feat/fase-2-a` |
| integración | — | — | main checkout, after `--integrate --fase 2` |
| verificación | TASK-F2-008 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | TASK-F2-001 | `fase-2-foundation` | main checkout |
| Verified | TASK-F2-008 | `fase-2-verified` | main checkout |
