# Tasks: FASE-6 - Salir y caducidad de la sesión

> **Input:** plan/fases/FASE-6-SESION.md + plan/fase-plans/PLAN-FASE-6.md
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
| UC-017 | TASK-F6-002, 003 |
| UC-018 | TASK-F6-004, 005, 006 |
| API-001-02 | TASK-F6-002 |
| INV-ACC-002 | TASK-F6-001, 002 |
| INV-ACC-003 | TASK-F6-002, 004 |
| INV-ACC-007 | TASK-F6-001, 004 |
| REQ-F-018 / REQ-F-019 | TASK-F6-002, 003 / TASK-F6-004..006 |
| D-PA-009 | TASK-F6-005 |

---

## Phase 2: Foundation

**Purpose:** Columna de última actividad y repositorio de sesiones, que comparten salir y caducar.
**Checkpoint:** Prueba de repositorio en verde.

- [ ] TASK-F6-001 Session activity column and sessions repository, test-first | `src/server/repo/migrations/004-sesion-actividad.sql`, `src/server/repo/sesiones.js`, `tests/integration/sesion.test.js`
  - **Commit:** `feat(repo): session last-activity column and sessions repository`
  - **Acceptance:**
    - Test first: `INV-ACC-007 ultima_actividad` — abrir una sesión guarda `ultima_actividad` con el instante del reloj; `tocar(correo, ahora)` la actualiza; `borrar(correo)` quita la fila; sigue habiendo una sola fila por cuenta (INV-ACC-002)
    - `004-sesion-actividad.sql` añade `ultima_actividad` a `sesion` con las sesiones existentes puestas al instante de la migración; el doble en memoria tiene la misma interfaz
  - **Refs:** FASE-6, REQ-F-018, REQ-F-019, INV-ACC-002, INV-ACC-007, ADR-DRAFT-003
  - **Revert:** MIGRATION — la columna no tiene vuelta atrás automática; borrar `data/club.db` de desarrollo tras revertir
  - **Review:**
    - [ ] SQL parametrizado
    - [ ] Migración aplicable sobre una BD de FASE-5

---

## Phase 3: Slices

**Purpose:** Un corte vertical por caso de uso, con la prueba escrita antes.
**Checkpoint:** Pruebas de UC-017 y UC-018 en verde.

### UC-017 — Cerrar sesión

- [ ] TASK-F6-002 Sign out (API-001-02), test-first | `src/domain/acceso.js`, `src/server/http/acceso.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/unit/domain/sesion.test.js`, `tests/integration/sesion.test.js`
  - blocked-by: TASK-F6-001
  - **Commit:** `feat(acceso): sign out deletes the session and the cookie`
  - **Acceptance:**
    - Test first: `AC-017-01 REQ-F-018` — `POST /api/cerrarSesion` da 200, borra la fila de sesión y envía `Set-Cookie` con `Max-Age=0`
    - Test first: `AC-017-02 INV-ACC-003` — tras salir, cualquier operación da `E_AUTH_REQUIRED` sin datos
    - Test first: `AC-017-03` — salir otra vez, sin sesión, da 200 sin cambios
    - `cerrarSesion({ repo }, ctx)` en el dominio, sin tocar datos de otras cuentas; la operación entra en la matriz para los tres roles
  - **Refs:** FASE-6, REQ-F-018, UC-017, API-001-02, INV-ACC-002, INV-ACC-003
  - **Revert:** COUPLED — el botón TASK-F6-003 la usa
  - **Review:**
    - [ ] Repetir no falla
    - [ ] Cookie realmente expirada

- [ ] TASK-F6-003 «Salir» button in the common header and redirect to sign-in | `src/ui/cabecera.js`, `src/ui/api.js`, `src/shared/textos.js`
  - blocked-by: TASK-F6-002
  - **Commit:** `feat(ui): Salir button in the common header`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; la cabecera común de toda pantalla con sesión muestra «Salir»; al pulsarlo llama a `cerrarSesion` y lleva a `#/iniciar-sesion` sin mensaje
    - Pulsarlo con la sesión ya cerrada también va a Iniciar sesión; comprobado por el journey TASK-F6-007 (AC-017-01, AC-017-03)
  - **Refs:** FASE-6, REQ-F-018, UC-017, WIREFRAMES cabecera común
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Botón accesible con nombre «Salir»
    - [ ] Sin mensaje tras salir

### UC-018 — Caducidad por inactividad

- [ ] TASK-F6-004 Idle expiry after 30 minutes and renewal on each operation, test-first | `src/domain/acceso.js`, `src/server/http/sesion.js`, `tests/unit/domain/sesion.test.js`, `tests/integration/sesion.test.js`
  - blocked-by: TASK-F6-003
  - **Commit:** `feat(acceso): sessions expire after 30 idle minutes and renew with activity`
  - **Acceptance:**
    - Test first: `AC-018-01 REQ-F-019 UNIT-T-018` — con el reloj fijo avanzado más de 30 min sin actividad, la operación da `E_SESSION_EXPIRED` «Tu sesión ha caducado» (401); a 30:00 exactos la sesión aún vale
    - Test first: `AC-018-05 REQ-F-019 AC2` — otro socio caducado recibe el mismo mensaje
    - Test first: `AC-018-02 AC-018-03 INV-ACC-007` — dentro del plazo la operación funciona y renueva `ultima_actividad`; una sesión caducada no se renueva
    - Test first: `AC-018-04 INV-ACC-002` — entrar de nuevo tras caducar abre una sesión normal y deja una sola fila por cuenta
    - `estadoSesion(sesion, ahora, minutosInactividad)` devuelve `'vigente'` o `'caducada'` con `SESSION_IDLE_MIN = 30`; «caducada» y «sin sesión» dan errores distintos sin filtrar datos de la cuenta; ninguna prueba espera tiempo real (D-T-011)
  - **Refs:** FASE-6, REQ-F-019, UC-018, INV-ACC-003, INV-ACC-007, SESSION_IDLE_MIN, D-T-011
  - **Revert:** COUPLED — cambia cómo todas las operaciones tratan la sesión
  - **Review:**
    - [ ] Reloj inyectable, sin esperas
    - [ ] Caducada y ausente se distinguen

- [ ] TASK-F6-005 Test-only clock control enabled only with MONTANA_TEST_CLOCK=1, test-first | `src/server/http/rutas.js`, `tests/integration/sesion.test.js`
  - blocked-by: TASK-F6-004
  - **Commit:** `feat(http): clock control for tests, disabled unless MONTANA_TEST_CLOCK=1`
  - **Acceptance:**
    - Test first: `D-PA-009 R3 reloj de pruebas` — con `MONTANA_TEST_CLOCK=1`, `POST /api/__reloj` avanza el reloj del servidor N minutos; sin esa variable la ruta responde 404 como cualquier operación desconocida
    - Test first: `D-PA-009 solo servidor` — el navegador nunca lee la hora: avanzar el reloj hace que la siguiente operación dé `E_SESSION_EXPIRED` (AC-018-01)
    - La ruta no está en la matriz de producción y el arranque de producción (`npm start`) no la registra
  - **Refs:** FASE-6, REQ-F-019, D-PA-009, R3
  - **Revert:** CONFIG — depende de la variable de entorno `MONTANA_TEST_CLOCK`
  - **Review:**
    - [ ] Ruta inexistente sin la variable
    - [ ] No documentada en el README de producción como función normal

- [ ] TASK-F6-006 Expiry message on the sign-in screen | `src/ui/api.js`, `src/ui/pantallas/iniciar-sesion.js`, `src/shared/textos.js`
  - blocked-by: TASK-F6-005
  - **Commit:** `feat(ui): show the session-expired message on sign-in and clear it on entry`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `llamar` lleva a `#/iniciar-sesion` con el mensaje «Tu sesión ha caducado» ante `E_SESSION_EXPIRED` y sin mensaje ante `E_AUTH_REQUIRED`; el mensaje desaparece al entrar
    - Comprobado por el journey TASK-F6-007 (AC-018-01, AC-018-04)
  - **Refs:** FASE-6, REQ-F-019, UC-018, INTERACTION-MODEL v1.1
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Mensaje en región `role="alert"`
    - [ ] Se borra tras entrar

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-6 verificados.

- [ ] TASK-F6-007 Journey FASE-6: salir y caducidad de sesión through the user's route | `tests/e2e/fase-6.spec.js`
  - blocked-by: TASK-F6-006
  - **Commit:** `test(e2e): FASE-6 journey with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-017-01 …`, `AC-017-02`, `AC-017-03`, `AC-018-01`, `AC-018-02`, `AC-018-03`, `AC-018-04`, `AC-018-05` (REQ-F-018, REQ-F-019)
    - El título del journey lleva `FASE-6` (la FASE no nombra workflow): es el id del vídeo y lo que busca `sdd gate --fase 6`
    - Entra por el paso 1 de la Demo: lucia entra desde `/` y pulsa «Salir», nunca por un punto de entrada interno; la caducidad avanza el reloj con el control de pruebas (`MONTANA_TEST_CLOCK=1`), sin esperas reales
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra: Iniciar sesión sin mensaje tras «Salir», «Tu sesión ha caducado» tras 31 min, la operación que sí funciona dentro del plazo
    - Guarda `evidencias/FASE-6/{AC-NNN-NN}.png` por criterio y un vídeo con `FASE-6` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-6, REQ-F-018, REQ-F-019, AC-017-01..03, AC-018-01..05
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Cada escenario con un `test` que lleva su id
    - [ ] Capturas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/server/http/acceso.js`, `src/server/http/rutas.js` | cableado | `tests/integration/sesion.test.js` |
| `src/ui/cabecera.js`, `src/ui/api.js`, `src/ui/pantallas/iniciar-sesion.js`, `src/shared/textos.js` | cliente y constantes | journey TASK-F6-007 y MNT-T-001 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─► 002 ─► 003 ─► 004 ─► 005 ─► 006 ─► 007
```

### Critical Path

1. TASK-F6-001 → 002 → 003 → 004 → 005 → 006 → 007
   (7 tasks on critical path)

### Parallel Execution Plan

**Stream A:** salir y caducidad en serie (comparten `acceso.js`, `rutas.js`, `sesion.test.js` y `api.js`).

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | TASK-F6-001 | src/server/repo/migrations/004-sesion-actividad.sql, src/server/repo/sesiones.js | main checkout, before worktrees (checkpoint `fase-6-foundation`) |
| A | TASK-F6-002, TASK-F6-003, TASK-F6-004, TASK-F6-005, TASK-F6-006 | src/domain/acceso.js, src/server/http/acceso.js, src/server/http/sesion.js, src/server/http/rutas.js, src/shared/textos.js, src/ui/cabecera.js, src/ui/api.js, src/ui/pantallas/iniciar-sesion.js, tests/unit/domain/sesion.test.js, tests/integration/sesion.test.js | worktree `feat/fase-6-a` |
| integración | — | — | main checkout, after `--integrate --fase 6` |
| verificación | TASK-F6-007 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | TASK-F6-001 | `fase-6-foundation` | main checkout |
| Verified | TASK-F6-007 | `fase-6-verified` | main checkout |
