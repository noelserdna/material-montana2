# Tasks: FASE-4 - Ver el día y dar de baja unidades rotas

> **Input:** plan/fases/FASE-4-HOY-Y-BAJAS.md + plan/fase-plans/PLAN-FASE-4.md
> **Generated:** 2026-11-16
> **Total tasks:** 7
> **Parallel capacity:** 1 work Stream (A); sin tareas base
> **Critical path:** 7 tasks

---

## Summary

| Metric | Value |
|--------|-------|
| Total tasks | 7 |
| Parallelizable | 0 (0%) |
| Work Streams | 1 (A: 6 tasks) |
| Setup phase | 0 tasks |
| Foundation phase | 0 tasks |
| Slices phase | 6 tasks |
| Integration phase | 0 tasks |
| Verification phase | 1 task |

## Traceability

| Spec Reference | Task Coverage |
|---------------|---------------|
| UC-012 | TASK-F4-001, 002, 003 |
| UC-013 | TASK-F4-004, 005, 006 |
| API-003-03 / API-002-06 | TASK-F4-002 / TASK-F4-005 |
| INV-INV-001, INV-INV-002 | TASK-F4-001 |
| INV-RES-001 | TASK-F4-002 |
| INV-RES-005 | TASK-F4-004 |
| RN-002, Q-requirements-engineer-002 | TASK-F4-002, 004, 006 |
| REQ-F-012 / REQ-F-013 | TASK-F4-001..003 / TASK-F4-004..006 |

---

## Phase 3: Slices

**Purpose:** Un corte vertical por operación, con la prueba escrita antes. No hay infraestructura nueva compartida por varios cortes: el esquema no cambia.
**Checkpoint:** Pruebas de UC-012 y UC-013 en verde.

### UC-012 — Dar de baja unidades rotas

- [ ] TASK-F4-001 Retire units in the domain: errors in order, no reservation touched, test-first | `src/domain/inventario.js`, `src/server/repo/inventario.js`, `tests/unit/domain/inventario.test.js`
  - **Commit:** `feat(inventario): retire broken units without touching reservations`
  - **Acceptance:**
    - Test first: `AC-012-01 REQ-F-012 baja normal` — «Casco» con 5 activas y baja de 2 → 3 activas
    - Test first: `AC-012-02 INV-INV-002 E_UNITS_INSUFFICIENT` — dar de baja más de las activas da `E_UNITS_INSUFFICIENT` y el inventario no cambia
    - Test first: `AC-012-04 AC-012-05 AC-012-06` — 0 unidades da `E_UNITS_MIN`; material inexistente `E_MATERIAL_NOT_FOUND`; un socio `E_FORBIDDEN_ROLE`; orden de errores rol → `E_UNITS_MIN` → `E_MATERIAL_NOT_FOUND` → `E_UNITS_INSUFFICIENT`
    - Test first: `AC-012-08 D-T-008` — repetir la baja resta otra vez, como acción nueva
    - Test first: `AC-012-07 RN-002` — una reserva futura sigue `Reservada`: la baja no la cancela ni envía avisos (Q-requirements-engineer-002)
  - **Refs:** FASE-4, REQ-F-012, UC-012, INV-INV-001, INV-INV-002, RN-002, D-T-008, Q-requirements-engineer-002
  - **Revert:** COUPLED — la operación HTTP TASK-F4-002 la usa
  - **Review:**
    - [ ] No escribe en `reserva`
    - [ ] Unidades activas nunca por debajo de 0

- [ ] TASK-F4-002 Retire-units operation (API-003-03) and booking with zero active units, test-first | `src/server/http/inventario.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/hoy-bajas.test.js`
  - blocked-by: TASK-F4-001
  - **Commit:** `feat(inventario): darDeBajaUnidades operation for the encargado`
  - **Acceptance:**
    - Test first: `AC-012-03 REQ-F-012 AC3 INV-RES-001` — con 0 unidades activas, reservar ese material da `E_NO_STOCK`
    - Test first: `AC-012-06 INV-ACC-005` — `POST /api/darDeBajaUnidades` con rol socio da 403 `E_FORBIDDEN_ROLE`; sin sesión 401
    - Test first: `AC-012-07 RN-002` — tras la baja por HTTP, `listarMisReservas` del socio sigue mostrando la reserva `Reservada`
    - La respuesta trae el material con la nueva cifra activa; entrada en la matriz de permisos solo para encargado
  - **Refs:** FASE-4, REQ-F-012, UC-012, API-003-03, INV-RES-001, INV-ACC-005, RN-002
  - **Revert:** COUPLED — la pantalla TASK-F4-003 la usa
  - **Review:**
    - [ ] Rol comprobado en servidor
    - [ ] Textos nuevos solo en `textos.js`

- [ ] TASK-F4-003 Retire-units form on the inventory screen | `src/ui/pantallas/inventario.js`, `src/shared/textos.js`
  - blocked-by: TASK-F4-002
  - **Commit:** `feat(ui): retire-units form on the inventory screen`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/inventario` añade el formulario Material y Unidades con botón «Dar de baja» solo para el encargado y muestra la nueva cifra activa; los errores `E_UNITS_INSUFFICIENT` y `E_UNITS_MIN` salen en la región de mensaje
    - Comprobado por el journey TASK-F4-007 (AC-012-01, AC-012-02)
  - **Refs:** FASE-4, REQ-F-012, UC-012, WIREFRAMES SCR-006
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Error asociado al campo
    - [ ] Un socio no ve el formulario

### UC-013 — Ver lo prestado y las devoluciones de cada día

- [ ] TASK-F4-004 Today view in the domain: lent, returning and over-capacity days, test-first | `src/domain/reservas.js`, `src/server/repo/reservas.js`, `tests/integration/hoy-bajas.test.js`
  - blocked-by: TASK-F4-003
  - **Commit:** `feat(reservas): verHoy with lent items, returns and over-capacity days`
  - **Acceptance:**
    - Test first: `AC-013-01 REQ-F-013` — `verHoy(dia)` lista como `prestado` las reservas `Entregada` en curso con material y correo
    - Test first: `AC-013-02 AC-013-03` — `vuelven` trae las `Entregada` cuyo `hasta` es el día; un día sin nada devuelve listas vacías (la pantalla dice el texto de vacío)
    - Test first: `AC-013-06` — abrirlo otra vez no cambia nada
    - Test first: `AC-012-07 RN-002 excesos` — `excesosDeCapacidad` marca el día y material con más reservas activas que unidades activas (p. ej. «Casco» 3 → 2 activas con 3 reservas el 2026-12-10); se calcula por consulta, sin guardar ni notificar
  - **Refs:** FASE-4, REQ-F-013, UC-013, INV-RES-005, RN-002, AC-012-07
  - **Revert:** COUPLED — la operación HTTP TASK-F4-005 la usa
  - **Review:**
    - [ ] Lectura sin escrituras
    - [ ] Ocupadas desde el estado de las reservas, no de un contador

- [ ] TASK-F4-005 Today operation (API-002-06), test-first | `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/hoy-bajas.test.js`
  - blocked-by: TASK-F4-004
  - **Commit:** `feat(reservas): verHoy operation for the encargado`
  - **Acceptance:**
    - Test first: `AC-013-04 AC-013-05 INV-ACC-005` — un socio recibe `E_FORBIDDEN_ROLE` (403) y sin sesión `E_AUTH_REQUIRED` (401), sin datos
    - Test first: `AC-013-01 AC-013-02 API-002-06` — `POST /api/verHoy` con `dia` opcional (por defecto «hoy» del reloj) devuelve `{ prestado, vuelven, excesos }`
  - **Refs:** FASE-4, REQ-F-013, UC-013, API-002-06, INV-ACC-005
  - **Revert:** COUPLED — la pantalla TASK-F4-006 la usa
  - **Review:**
    - [ ] Fecha por defecto del reloj inyectable
    - [ ] Entrada en la matriz solo para encargado

- [ ] TASK-F4-006 Today screen SCR-007 with the over-capacity notice | `src/ui/pantallas/hoy.js`, `src/ui/aviso-exceso.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js`
  - blocked-by: TASK-F4-005
  - **Commit:** `feat(ui): Today screen with lent items, returns and over-capacity notice`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/hoy` pinta Prestado y Vuelven con material y correo, el texto de vacío cuando no hay nada y un selector de día; `aviso-exceso` muestra «Aviso: de {material} hay {n} reservas el {día} y solo {m} unidades activas. Avisa a: {correos}» (UX-TXT-07)
    - Un socio que llega a `#/hoy` ve SCR-009 (sin permiso); comprobado por el journey TASK-F4-007
  - **Refs:** FASE-4, REQ-F-013, UC-013, RN-002, WIREFRAMES SCR-007
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] El aviso no cancela ni notifica nada
    - [ ] Texto del aviso desde `textos.js`

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-4 verificados.

- [ ] TASK-F4-007 Journey FASE-4: ver el día y dar de baja unidades through the user's route | `tests/e2e/fase-4.spec.js`
  - blocked-by: TASK-F4-006
  - **Commit:** `test(e2e): FASE-4 journey with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-012-01 …`, `AC-012-02`, `AC-012-03`, `AC-012-04`, `AC-012-05`, `AC-012-06`, `AC-012-07`, `AC-012-08`, `AC-013-01`, `AC-013-02`, `AC-013-03`, `AC-013-04`, `AC-013-05`, `AC-013-06` (REQ-F-012, REQ-F-013)
    - El título del journey lleva `FASE-4` (la FASE no nombra workflow): es el id del vídeo y lo que busca `sdd gate --fase 4`
    - Entra por el paso 1 de la Demo: marta abre Hoy (`#/hoy`) el 2026-12-10, nunca por un punto de entrada interno
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra: «Casco» y «lucia@club.es» en Prestado, el texto de vacío, la cifra activa tras la baja, el literal de `E_UNITS_INSUFFICIENT`, el aviso de exceso con «3 reservas» y «2 unidades activas»
    - Guarda `evidencias/FASE-4/{AC-NNN-NN}.png` por criterio y un vídeo con `FASE-4` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-4, REQ-F-012, REQ-F-013, AC-012-01..08, AC-013-01..06
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Cada escenario con un `test` que lleva su id
    - [ ] Capturas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js` | constantes y cableado | MNT-T-001 y journey TASK-F4-007 |
| `src/ui/pantallas/hoy.js`, `src/ui/pantallas/inventario.js`, `src/ui/aviso-exceso.js` | pantallas y componente | journey TASK-F4-007 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─► 002 ─► 003 ─► 004 ─► 005 ─► 006 ─► 007
```

### Critical Path

1. TASK-F4-001 → 002 → 003 → 004 → 005 → 006 → 007
   (7 tasks on critical path)

### Parallel Execution Plan

**Stream A:** bajas y Hoy en serie (comparten `rutas.js`, `textos.js` y el dominio de reservas e inventario).

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | — | — | main checkout, before worktrees (checkpoint `fase-4-foundation`) |
| A | TASK-F4-001, TASK-F4-002, TASK-F4-003, TASK-F4-004, TASK-F4-005, TASK-F4-006 | src/domain/inventario.js, src/domain/reservas.js, src/server/repo/inventario.js, src/server/repo/reservas.js, src/server/http/inventario.js, src/server/http/reservas.js, src/server/http/rutas.js, src/shared/textos.js, src/ui/pantallas/hoy.js, src/ui/pantallas/inventario.js, src/ui/aviso-exceso.js, src/ui/router.js, src/ui/cabecera.js, tests/unit/domain/inventario.test.js, tests/integration/hoy-bajas.test.js | worktree `feat/fase-4-a` |
| integración | — | — | main checkout, after `--integrate --fase 4` |
| verificación | TASK-F4-007 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | — (sin tareas base: el tag `fase-4-foundation` se pone en el HEAD de entrada) | `fase-4-foundation` | main checkout |
| Verified | TASK-F4-007 | `fase-4-verified` | main checkout |
