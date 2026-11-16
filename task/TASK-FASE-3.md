# Tasks: FASE-3 - Multas por retraso

> **Input:** plan/fases/FASE-3-MULTAS.md + plan/fase-plans/PLAN-FASE-3.md
> **Generated:** 2026-11-16
> **Total tasks:** 10
> **Parallel capacity:** 1 work Stream (A); base serial
> **Critical path:** 9 tasks

---

## Summary

| Metric | Value |
|--------|-------|
| Total tasks | 10 |
| Parallelizable | 0 (0%) |
| Work Streams | 1 (A: 7 tasks) |
| Setup phase | 0 tasks |
| Foundation phase | 2 tasks |
| Slices phase | 7 tasks |
| Integration phase | 0 tasks |
| Verification phase | 1 task |

## Traceability

| Spec Reference | Task Coverage |
|---------------|---------------|
| UC-008 | TASK-F3-003, 004, 005 |
| UC-009 | TASK-F3-006 |
| UC-010 | TASK-F3-008, 009 |
| API-002-05 (multa) | TASK-F3-004 |
| API-004-01 / API-004-02 | TASK-F3-007 / TASK-F3-008 |
| INV-FIN-001 | TASK-F3-003 |
| INV-FIN-002 | TASK-F3-001, 004 |
| INV-FIN-003 | TASK-F3-006 |
| INV-ACC-005 | TASK-F3-007, 008 |
| REQ-F-008 / REQ-F-009 / REQ-F-010 | TASK-F3-003, 004, 005, 007 / TASK-F3-006 / TASK-F3-008, 009 |
| REQ-C-003 | TASK-F3-003 |

---

## Phase 2: Foundation

**Purpose:** Tabla de multas con `reserva_id` único y formato de importe, que comparten varios cortes.
**Checkpoint:** Pruebas de repositorio y de formato en verde.

- [ ] TASK-F3-001 Fine table with unique reservation and fine repository, test-first | `src/server/repo/migrations/003-multas.sql`, `src/server/repo/multas.js`, `tests/integration/multas.test.js`
  - **Commit:** `feat(repo): fine table with one fine per reservation`
  - **Acceptance:**
    - Test first: `AC-008-04 INV-FIN-002 reserva única` — insertar una segunda multa para el mismo `reserva_id` falla por la restricción `UNIQUE` de la BD
    - `003-multas.sql` crea `multa` (`reserva_id` único → `reserva`, `socio`, `dias`, `importe`, `estado CHECK IN ('Pendiente','Pagada')`); el repositorio ofrece insertar, listar (todas), buscar por id, cambiar estado y contar `Pendiente` por socio, con SQL parametrizado y también en el doble en memoria
  - **Refs:** FASE-3, REQ-F-008, INV-FIN-002, ADR-DRAFT-002
  - **Revert:** MIGRATION — tabla nueva sin vuelta atrás automática; borrar `data/club.db` de desarrollo tras revertir
  - **Review:**
    - [ ] Unicidad en la BD, no solo en código
    - [ ] Doble en memoria con la misma interfaz

- [ ] TASK-F3-002 Euro amount formatter shared by Fines and Deliveries, test-first | `src/ui/formato.js`, `tests/unit/ui/formato.test.js`
  - **Commit:** `feat(ui): euro amount formatter`
  - **Acceptance:**
    - Test first: `formato importeEuros` — `importeEuros(6)` da «6 €» y `importeEuros(2)` da «2 €» (formato español, sin decimales innecesarios); un importe 0 da «0 €»
  - **Refs:** FASE-3, REQ-F-008, UC-008
  - **Revert:** SAFE — función nueva
  - **Review:**
    - [ ] Sin tarifa escrita aquí
    - [ ] Formato español

---

## Phase 3: Slices

**Purpose:** Un corte vertical por operación, con la prueba escrita antes.
**Checkpoint:** Pruebas de UC-008, UC-009 y UC-010 en verde.

### UC-008 — Multa por retraso

- [ ] TASK-F3-003 Fine calculation from FINE_RATE_EUR, test-first | `src/domain/multas.js`, `tests/unit/domain/multas.test.js`
  - blocked-by: TASK-F3-001
  - **Commit:** `feat(multas): fine calculation at 2 euros per natural day`
  - **Acceptance:**
    - Test first: `AC-008-01 AC-008-02 REQ-F-008 UNIT-T-010..012` — `calcularMulta('2026-12-12','2026-12-15', FINE_RATE_EUR)` → `{ dias: 3, importe: 6 }`; un día de retraso → `{ dias: 1, importe: 2 }`
    - Test first: `AC-008-03 INV-FIN-001` — `devueltaEl` = `hasta` y `hasta − 1` dan `null` (sin multa); valores límite hasta−1, hasta, hasta+1 y +3
    - Test first: `REQ-C-003 MNT-T-003` — `FINE_RATE_EUR = 2` se define una sola vez (`multas.js`) y ningún otro fichero de `src/` escribe `2` como tarifa
  - **Refs:** FASE-3, REQ-F-008, REQ-C-003, UC-008, INV-FIN-001, MNT-T-003
  - **Revert:** SAFE — función pura nueva
  - **Review:**
    - [ ] Días naturales, no laborables
    - [ ] Tarifa en un único sitio

- [ ] TASK-F3-004 Create the fine in the same transaction as the return (API-002-05), test-first | `src/domain/reservas.js`, `src/server/http/reservas.js`, `src/shared/textos.js`, `tests/integration/multas.test.js`
  - blocked-by: TASK-F3-003
  - **Commit:** `feat(reservas): late return creates a pending fine atomically`
  - **Acceptance:**
    - Test first: `AC-008-01 AC-008-02 REQ-F-008` — devolver con `devueltaEl` 2026-12-15 y `hasta` 2026-12-12 da 200 con `{ reserva, multa: { dias: 3, importe: 6, estado: 'Pendiente' } }`
    - Test first: `AC-008-03` — devolver en plazo no crea multa
    - Test first: `AC-008-04 INV-FIN-002` — devolver otra vez da `O_ALREADY_RETURNED` y sigue habiendo una sola multa
    - Test first: `AC-008-05 D-T-002 E_STORAGE_UNAVAILABLE` — con el almacén que falla al escribir la multa, la devolución también se deshace: ni `Devuelta` ni multa quedan escritas (503)
  - **Refs:** FASE-3, REQ-F-008, UC-008, API-002-05, INV-FIN-001, INV-FIN-002, D-PA-003, D-T-002
  - **Revert:** COUPLED — amplía `devolverReserva` de FASE-2
  - **Review:**
    - [ ] Devolución y multa en una transacción
    - [ ] Los tests de FASE-2 siguen en verde

- [ ] TASK-F3-005 Show the created fine on the returns screen | `src/ui/pantallas/entregas.js`, `src/shared/textos.js`
  - blocked-by: TASK-F3-004, TASK-F3-002
  - **Commit:** `feat(ui): returns screen shows the fine created on return`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; tras devolver tarde, `#/entregas` muestra la multa con `importeEuros` («6 €»); comprobado por el journey TASK-F3-010 (AC-008-01)
  - **Refs:** FASE-3, REQ-F-008, UC-008, WIREFRAMES SCR-004
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Importe con el formateador común
    - [ ] Sin textos fuera de `textos.js`

### UC-009 — Sin reservar con multa pendiente

- [ ] TASK-F3-006 Block reserving while a fine is pending, test-first | `src/domain/multas.js`, `src/domain/reservas.js`, `src/shared/textos.js`, `tests/integration/multas.test.js`
  - blocked-by: TASK-F3-005
  - **Commit:** `feat(reservas): pending fine blocks new reservations`
  - **Acceptance:**
    - Test first: `AC-009-01 AC-002-08 REQ-F-009` — con una multa `Pendiente` reservar da `E_FINE_PENDING` (409) y no crea nada; el paso va entre el rango y el duplicado (D-T-004)
    - Test first: `AC-009-02 INV-FIN-003` — con dos multas pendientes, pagar una no desbloquea
    - Test first: `AC-009-03 AC-009-04` — otro socio sin multa reserva con normalidad y las reservas previas del socio con multa no cambian
    - Test first: `AC-009-05` — pagar y reservar a la vez no deja estado a medias: el resultado es el de uno de los dos órdenes
  - **Refs:** FASE-3, REQ-F-009, UC-009, UC-002, INV-FIN-003, D-T-004
  - **Revert:** COUPLED — toca `crearReserva` de FASE-1
  - **Review:**
    - [ ] Orden de errores respeta D-T-004
    - [ ] Las pruebas de FASE-1 siguen en verde

### UC-010 — Marcar la multa como pagada

- [ ] TASK-F3-007 List fines for the encargado (API-004-01), test-first | `src/domain/multas.js`, `src/server/http/multas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/multas.test.js`
  - blocked-by: TASK-F3-006
  - **Commit:** `feat(multas): list every member's fines, encargado only`
  - **Acceptance:**
    - Test first: `API-004-01 REQ-F-008 listar` — `listarMultas` devuelve correo, importe y estado de todas las multas al encargado; sin multas devuelve lista vacía («Sin multas» en pantalla)
    - Test first: `INV-ACC-005` — un socio o la junta sin permiso reciben `E_FORBIDDEN_ROLE`; sin sesión `E_AUTH_REQUIRED`
  - **Refs:** FASE-3, REQ-F-008, UC-010, API-004-01, INV-ACC-005
  - **Revert:** COUPLED — la pantalla TASK-F3-009 la usa
  - **Review:**
    - [ ] Solo lectura
    - [ ] Entrada en la matriz de permisos

- [ ] TASK-F3-008 Mark a fine as paid (API-004-02), test-first | `src/domain/multas.js`, `src/server/http/multas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/integration/multas.test.js`
  - blocked-by: TASK-F3-007
  - **Commit:** `feat(multas): mark a fine as paid, encargado only`
  - **Acceptance:**
    - Test first: `AC-010-01 AC-010-02 REQ-F-010` — la multa pasa a `Pagada` y el socio sin otras pendientes vuelve a poder reservar; la acción se anota en el registro de seguridad
    - Test first: `AC-010-03 AC-010-05` — pagar una ya pagada da `O_ALREADY_PAID` sin cambios; una inexistente `E_FINE_NOT_FOUND` (404)
    - Test first: `AC-010-04 INV-ACC-005` — un socio recibe `E_FORBIDDEN_ROLE` y la multa sigue `Pendiente`
  - **Refs:** FASE-3, REQ-F-010, UC-010, API-004-02, INV-ACC-005, INCIDENT-001
  - **Revert:** COUPLED — comparte `multas.js` con 007
  - **Review:**
    - [ ] Aviso sin efectos al repetir
    - [ ] Rol comprobado en servidor

- [ ] TASK-F3-009 Fines screen SCR-005 with Mark as paid | `src/ui/pantallas/multas.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js`
  - blocked-by: TASK-F3-007, TASK-F3-008, TASK-F3-002
  - **Commit:** `feat(ui): fines screen for the encargado`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/multas` pinta Multas con correo, importe (`importeEuros`) y estado, o «Sin multas», y botón «Marcar pagada» solo para multas pendientes
    - Comprobado por el journey TASK-F3-010 (AC-008-01, AC-010-01)
  - **Refs:** FASE-3, REQ-F-008, REQ-F-010, UC-010, WIREFRAMES SCR-005
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] Un socio no ve el enlace Multas
    - [ ] Botón con nombre accesible

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-3 verificados.

- [ ] TASK-F3-010 Journey FASE-3 WF-001: multas por retraso through the user's route | `tests/e2e/fase-3.spec.js`
  - blocked-by: TASK-F3-009, TASK-F3-005
  - **Commit:** `test(e2e): FASE-3 journey WF-001 with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-008-01 …`, `AC-008-02`, `AC-008-03`, `AC-008-04`, `AC-008-05`, `AC-009-01`, `AC-009-02`, `AC-009-03`, `AC-009-04`, `AC-009-05`, `AC-002-08`, `AC-010-01`, `AC-010-02`, `AC-010-03`, `AC-010-04`, `AC-010-05` (REQ-F-008, REQ-F-009, REQ-F-010)
    - El título del journey lleva `WF-001`: es el id del vídeo y lo que busca `sdd gate --fase 3`
    - Entra por el paso 1 de la Demo: marta abre `#/entregas` con el reloj en 2026-12-15 y devuelve el Casco de lucia; nunca por un punto de entrada interno
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra: «6 €» y «Pendiente» en Multas, el literal de `E_FINE_PENDING` al reservar, «Pagada», el aviso de ya devuelta, el literal de `E_FORBIDDEN_ROLE`
    - Guarda `evidencias/FASE-3/{AC-NNN-NN}.png` por criterio y un vídeo con `WF-001` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-3, REQ-F-008, REQ-F-009, REQ-F-010, WF-001, AC-008-01..05, AC-009-01..05, AC-002-08, AC-010-01..05
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Cada escenario con un `test` que lleva su id
    - [ ] Capturas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js` | constantes y cableado | MNT-T-001 y journey TASK-F3-010 |
| `src/ui/pantallas/multas.js` | pantalla | journey TASK-F3-010 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─► 003 ─► 004 ─► 005 ─► 006 ─► 007 ─► 008 ─► 009 ─► 010
002 ──────────────────┘                          └──┘
```

### Critical Path

1. TASK-F3-001 → 003 → 004 → 005 → 006 → 007 → 008 → 009 → 010
   (9 tasks on critical path)

### Parallel Execution Plan

**Stream A:** cálculo, multa atómica, bloqueo, listado y pago, en serie (comparten `multas.js`, `reservas.js`, `rutas.js` y `textos.js`).

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | TASK-F3-001, TASK-F3-002 | src/server/repo/migrations/003-multas.sql, src/server/repo/multas.js, src/ui/formato.js, tests/unit/ui/formato.test.js | main checkout, before worktrees (checkpoint `fase-3-foundation`) |
| A | TASK-F3-003, TASK-F3-004, TASK-F3-005, TASK-F3-006, TASK-F3-007, TASK-F3-008, TASK-F3-009 | src/domain/multas.js, src/domain/reservas.js, src/server/http/multas.js, src/server/http/reservas.js, src/server/http/rutas.js, src/shared/textos.js, src/ui/pantallas/entregas.js, src/ui/pantallas/multas.js, src/ui/router.js, src/ui/cabecera.js, tests/unit/domain/multas.test.js, tests/integration/multas.test.js | worktree `feat/fase-3-a` |
| integración | — | — | main checkout, after `--integrate --fase 3` |
| verificación | TASK-F3-010 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | TASK-F3-002 | `fase-3-foundation` | main checkout |
| Verified | TASK-F3-010 | `fase-3-verified` | main checkout |
