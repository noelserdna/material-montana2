# Tasks: FASE-1 - Reservar, ver y cancelar mis reservas

> **Input:** plan/fases/FASE-1-RESERVAS.md + plan/fase-plans/PLAN-FASE-1.md
> **Generated:** 2026-11-16
> **Total tasks:** 8
> **Parallel capacity:** 1 work Stream (A); base serial
> **Critical path:** 8 tasks

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

## Traceability

| Spec Reference | Task Coverage |
|---------------|---------------|
| UC-002 | TASK-F1-002, 003 |
| UC-004 | TASK-F1-004, 005 |
| UC-005 | TASK-F1-006, 007 |
| API-002-01 / 02 / 03 | TASK-F1-002 / TASK-F1-004 / TASK-F1-006 |
| INV-RES-001 | TASK-F1-001, 002 |
| INV-RES-002, INV-RES-003 | TASK-F1-002 |
| INV-RES-004 | TASK-F1-006 |
| INV-ACC-004, INV-ACC-005 | TASK-F1-004, 006 |
| REQ-F-002 / REQ-F-004 / REQ-F-005 | TASK-F1-001..003 / TASK-F1-004, 005 / TASK-F1-006, 007 |
| REQ-C-001, REQ-C-003 | TASK-F1-002, 003, 006 |

---

## Phase 2: Foundation

**Purpose:** Tabla de reservas y comprobación de stock por día, que comparten crear, listar y cancelar.
**Checkpoint:** Prueba de stock y de carrera en verde.

- [ ] TASK-F1-001 Reservation table and per-day stock check in one BEGIN IMMEDIATE transaction, test-first | `src/server/repo/migrations/002-reservas.sql`, `src/server/repo/reservas.js`, `tests/integration/reservas.test.js`
  - **Commit:** `feat(repo): reservation table and stock-checked insert`
  - **Acceptance:**
    - Test first: `AC-002-10 INV-RES-001 última unidad` — dos `crearSiHayStock` simultáneos sobre la última unidad de un día: solo una entra, la otra devuelve «sin stock»; la comprobación y la escritura van en la misma transacción `BEGIN IMMEDIATE`
    - Test first: `AC-002-04 RN-003 rango inclusivo` — días sin solape no compiten y `desde` y `hasta` cuentan ambos; solo `Reservada` y `Entregada` ocupan unidad (`Cancelada` y `Devuelta` no)
    - `002-reservas.sql` crea `reserva` (`estado` con `CHECK`, `devuelta_el`) y los índices `idx_reserva_material_estado` e `idx_reserva_socio`; `listarPorSocio(tx, correo)` y `cambiarEstado(tx, id, estado)` con SQL parametrizado
  - **Refs:** FASE-1, REQ-F-002, INV-RES-001, ADR-DRAFT-002, RN-003, R4
  - **Revert:** MIGRATION — tabla nueva sin vuelta atrás automática; borrar `data/club.db` de desarrollo tras revertir
  - **Review:**
    - [ ] Stock calculado por día, no con un contador
    - [ ] Sin SQL concatenado

---

## Phase 3: Slices

**Purpose:** Un corte vertical por operación, con la prueba escrita antes.
**Checkpoint:** Pruebas de UC-002, UC-004 y UC-005 en verde.

### UC-002 — Reservar material

- [ ] TASK-F1-002 Create a reservation (API-002-01), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/unit/domain/reservas.test.js`, `tests/integration/reservas.test.js`
  - blocked-by: TASK-F1-001
  - **Commit:** `feat(reservas): create a reservation with per-day stock`
  - **Acceptance:**
    - Test first: `AC-002-01 REQ-F-002 reserva libre` — con todo libre `crearReserva('Casco','2026-12-10','2026-12-12')` da 200 con la reserva `Reservada` y aparece en `listarMisReservas`
    - Test first: `AC-002-02 AC-002-03 INV-RES-001` — con una unidad libre ese día se reserva; sin unidades da `E_NO_STOCK` (409)
    - Test first: `AC-002-04 RN-003` — rangos sin solape no compiten por stock y el rango incluye ambos extremos
    - Test first: `AC-002-05 INV-RES-003 D-T-004` — repetir el formulario da `E_DUP_RESERVATION` antes que `E_NO_STOCK`; orden de errores: sesión → rol → rango → duplicado → stock
    - Test first: `AC-002-06 INV-RES-002 E_DATE_RANGE` — `hasta` anterior a `desde` da `E_DATE_RANGE` (400); no se añaden más comprobaciones de fechas pasadas ni raras (Q-spec-auditor-002)
    - Test first: `AC-002-07 AC-002-09` — la junta recibe `E_FORBIDDEN_ROLE` (403) y un material inexistente `E_MATERIAL_NOT_FOUND`
    - Test first: `AC-002-10 carrera HTTP` — dos peticiones simultáneas por la última unidad: una 200 y otra `E_NO_STOCK`; «hoy» viene del reloj inyectable (REQ-C-003)
  - **Refs:** FASE-1, REQ-F-002, UC-002, API-002-01, INV-RES-001, INV-RES-002, INV-RES-003, INV-ACC-005, D-T-004, Q-spec-auditor-002
  - **Revert:** COUPLED — las siguientes operaciones comparten `reservas.js`
  - **Review:**
    - [ ] Orden de errores igual a D-T-004
    - [ ] Textos nuevos solo en `textos.js`

- [ ] TASK-F1-003 Reserve form on the catalogue screen with date fields | `src/ui/pantallas/material.js`, `src/ui/campos.js`, `src/shared/textos.js`
  - blocked-by: TASK-F1-002
  - **Commit:** `feat(ui): reservation form with material, from and to dates`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/material` añade el formulario Material, Desde, Hasta con `campoFecha` (`Dia` `YYYY-MM-DD`, etiqueta y error asociado)
    - Los literales de `E_NO_STOCK`, `E_DUP_RESERVATION` y `E_DATE_RANGE` salen en la región de mensaje; tras reservar se va a Mis reservas; comprobado por el journey TASK-F1-008 (AC-002-01..06)
  - **Refs:** FASE-1, REQ-F-002, UC-002, WIREFRAMES SCR-002
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Campo de fecha con etiqueta y error ligado
    - [ ] Sin textos fuera de `textos.js`

### UC-004 — Ver mis reservas

- [ ] TASK-F1-004 List my reservations (API-002-02), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `tests/unit/domain/reservas.test.js`, `tests/integration/reservas.test.js`
  - blocked-by: TASK-F1-003
  - **Commit:** `feat(reservas): list only the actor's reservations`
  - **Acceptance:**
    - Test first: `AC-004-01 AC-004-02 REQ-F-004` — cada socio ve las suyas con material, fechas y estado
    - Test first: `AC-004-03 AC-004-04` — sin reservas devuelve lista vacía (la pantalla dice «Sin reservas»); sin sesión da `E_AUTH_REQUIRED` sin datos
    - Test first: `AC-004-05 INV-ACC-004` — las reservas de otro socio no salen nunca; la operación no admite parámetro de correo
    - Test first: `AC-004-06` — abrirla otra vez no cambia nada
  - **Refs:** FASE-1, REQ-F-004, UC-004, API-002-02, INV-ACC-004
  - **Revert:** COUPLED — la pantalla TASK-F1-005 la usa
  - **Review:**
    - [ ] Filtra por el correo de la sesión
    - [ ] Lectura sin escrituras

- [ ] TASK-F1-005 My reservations screen SCR-003 | `src/ui/pantallas/mis-reservas.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js`
  - blocked-by: TASK-F1-004
  - **Commit:** `feat(ui): my reservations screen with state and dates`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/mis-reservas` pinta tabla con Material, Desde, Hasta y Estado, o «Sin reservas»; el enlace sale en la cabecera para socio y encargado
    - Comprobado por el journey TASK-F1-008 (AC-004-01, AC-004-03, AC-004-05)
  - **Refs:** FASE-1, REQ-F-004, UC-004, WIREFRAMES SCR-003
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] Tabla accesible
    - [ ] Estado visible en texto, no solo color

### UC-005 — Cancelar una reserva

- [ ] TASK-F1-006 Cancel a reservation before its start day (API-002-03), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/unit/domain/reservas.test.js`, `tests/integration/reservas.test.js`
  - blocked-by: TASK-F1-005
  - **Commit:** `feat(reservas): cancel a reservation until the day before it starts`
  - **Acceptance:**
    - Test first: `AC-005-01 AC-005-02 REQ-F-005 INV-RES-004` — con hoy < `desde` la reserva pasa a `Cancelada` y la unidad queda libre para otra reserva
    - Test first: `AC-005-03 E_CANCEL_TOO_LATE` — el mismo día de `desde` da `E_CANCEL_TOO_LATE` (409) y la reserva sigue `Reservada`
    - Test first: `AC-005-04 O_ALREADY_CANCELLED` — cancelar dos veces da 200 con aviso `O_ALREADY_CANCELLED` y sin cambios
    - Test first: `AC-005-05 SEC-T-004` — la reserva ajena da `E_FORBIDDEN_DATA` (403) y sigue `Reservada`
    - Test first: `AC-005-06 AC-005-07` — la junta recibe `E_FORBIDDEN_ROLE`; una reserva inexistente `E_RES_NOT_FOUND`; «hoy» del reloj inyectable
  - **Refs:** FASE-1, REQ-F-005, UC-005, API-002-03, INV-RES-004, INV-ACC-004, INV-ACC-005, SEC-T-004
  - **Revert:** COUPLED — comparte `reservas.js` con TASK-F1-002 y 004
  - **Review:**
    - [ ] Solo `Reservada` y `hoy < desde`
    - [ ] Error o aviso sin ningún efecto

- [ ] TASK-F1-007 Cancel button on my reservations | `src/ui/pantallas/mis-reservas.js`, `src/shared/textos.js`
  - blocked-by: TASK-F1-006
  - **Commit:** `feat(ui): cancel button on my reservations`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; cada fila `Reservada` tiene botón Cancelar; tras cancelar la fila dice `Cancelada`; repetir muestra el aviso de ya cancelada
    - Comprobado por el journey TASK-F1-008 (AC-005-01, AC-005-04)
  - **Refs:** FASE-1, REQ-F-005, UC-005, WIREFRAMES SCR-003
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Botón con nombre accesible que incluye el material
    - [ ] Aviso en región `role="alert"`

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-1 verificados.

- [ ] TASK-F1-008 Journey FASE-1 WF-001: reservar, ver y cancelar through the user's route | `tests/e2e/fase-1.spec.js`
  - blocked-by: TASK-F1-007
  - **Commit:** `test(e2e): FASE-1 journey WF-001 with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-002-01 …`, `AC-002-02`, `AC-002-03`, `AC-002-04`, `AC-002-05`, `AC-002-06`, `AC-002-07`, `AC-002-09`, `AC-002-10`, `AC-004-01`, `AC-004-02`, `AC-004-03`, `AC-004-04`, `AC-004-05`, `AC-004-06`, `AC-005-01`, `AC-005-02`, `AC-005-03`, `AC-005-04`, `AC-005-05`, `AC-005-06`, `AC-005-07` (REQ-F-002, REQ-F-004, REQ-F-005)
    - El título del journey lleva `WF-001` (línea `Workflows:` de la FASE): es el id del vídeo y lo que busca `sdd gate --fase 1`
    - Entra por el paso 1 de la Demo: lucia entra desde `/` y reserva «Casco» del 2026-12-10 al 2026-12-12 en `#/material`, nunca por un punto de entrada interno
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra: «Reservada» en Mis reservas, el literal de `E_NO_STOCK`, `E_DUP_RESERVATION`, `E_DATE_RANGE`, «Cancelada», el aviso de ya cancelada, «Sin reservas»
    - Guarda `evidencias/FASE-1/{AC-NNN-NN}.png` por criterio y un vídeo con `WF-001` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-1, REQ-F-002, REQ-F-004, REQ-F-005, WF-001, AC-002-01..10, AC-004-01..06, AC-005-01..07
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Cada escenario con un `test` que lleva su id
    - [ ] Capturas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/ui/router.js`, `src/shared/textos.js` | constantes y cableado | MNT-T-001 y journey TASK-F1-008 |
| `src/ui/pantallas/*.js`, `src/ui/campos.js` | pantallas y componentes | journey TASK-F1-008 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─► 002 ─► 003 ─► 004 ─► 005 ─► 006 ─► 007 ─► 008
```

### Critical Path

1. TASK-F1-001 → 002 → 003 → 004 → 005 → 006 → 007 → 008
   (8 tasks on critical path)

### Parallel Execution Plan

**Stream A:** crear, ver y cancelar en serie: todos los cortes escriben `reservas.js`, `rutas.js`, `textos.js` y las pantallas.

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | TASK-F1-001 | src/server/repo/migrations/002-reservas.sql, src/server/repo/reservas.js | main checkout, before worktrees (checkpoint `fase-1-foundation`) |
| A | TASK-F1-002, TASK-F1-003, TASK-F1-004, TASK-F1-005, TASK-F1-006, TASK-F1-007 | src/domain/reservas.js, src/server/http/reservas.js, src/server/http/rutas.js, src/shared/textos.js, src/ui/pantallas/material.js, src/ui/pantallas/mis-reservas.js, src/ui/campos.js, src/ui/router.js, src/ui/cabecera.js, tests/unit/domain/reservas.test.js, tests/integration/reservas.test.js | worktree `feat/fase-1-a` |
| integración | — | — | main checkout, after `--integrate --fase 1` |
| verificación | TASK-F1-008 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | TASK-F1-001 | `fase-1-foundation` | main checkout |
| Verified | TASK-F1-008 | `fase-1-verified` | main checkout |
