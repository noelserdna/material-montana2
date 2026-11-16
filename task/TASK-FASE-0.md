# Tasks: FASE-0 - Entrar, añadir material y verlo en el catálogo

> **Input:** plan/fases/FASE-0-SKELETON.md + plan/fase-plans/PLAN-FASE-0.md
> **Generated:** 2026-11-16
> **Total tasks:** 18
> **Parallel capacity:** 1 work Stream (A); base serial
> **Critical path:** 13 tasks

---

## Summary

| Metric | Value |
|--------|-------|
| Total tasks | 18 |
| Parallelizable | 3 (17%) |
| Work Streams | 1 (A: 8 tasks) |
| Setup phase | 3 tasks |
| Foundation phase | 6 tasks |
| Slices phase | 8 tasks |
| Integration phase | 0 tasks |
| Verification phase | 1 task |

## Traceability

| Spec Reference | Task Coverage |
|---------------|---------------|
| UC-001 | TASK-F0-010, 011, 012, 013 |
| UC-003 | TASK-F0-014, 015 |
| UC-011 | TASK-F0-016, 017 |
| API-001-01 | TASK-F0-010, 011 |
| API-003-01 / API-003-02 | TASK-F0-014 / TASK-F0-016 |
| ADR-DRAFT-001 | TASK-F0-002, 008 |
| ADR-DRAFT-002 | TASK-F0-006 |
| ADR-DRAFT-003 | TASK-F0-007, 010, 012 |
| INV-ACC-001..003, 005, 006 | TASK-F0-007, 010, 011, 012 |
| INV-INV-001, 002 | TASK-F0-014, 016 |
| REQ-F-001 / REQ-F-017 | TASK-F0-010, 013 / TASK-F0-011 |
| REQ-F-011 / REQ-F-003 | TASK-F0-016, 017 / TASK-F0-014, 015 |
| REQ-NF-002 | TASK-F0-010, 012 |
| REQ-C-001, REQ-C-002 | TASK-F0-001, 004 |

---

## Phase 1: Setup

**Purpose:** Scaffold, scripts de producción y configuración de pruebas. Deja la app lista para ejecutarse como en producción desde el primer commit (`npm run build`, `npm start`, `PORT`/`HOST`, README).
**Checkpoint:** `npm run build` genera `dist/`, `npm test` arranca y `npm start` sirve la app construida.

- [ ] TASK-F0-001 Scaffold the app with pinned dependencies and production scripts | `package.json`, `vite.config.js`, `vitest.config.js`, `index.html`, `src/ui/main.js`, `README.md`, `.gitignore`
  - **Commit:** `chore(build): scaffold Vite + Vitest app with build and start scripts`
  - **Acceptance:**
    - Verify: `npm ci && npm run build` → crea `dist/index.html` sin errores (build de Vite)
    - Verify: `npx vitest run --passWithNoTests --maxWorkers=2` → termina con código 0; `vitest.config.js` declara `test: { maxWorkers: 2 }` (REQ-C-002, MNT-T-002)
    - `package.json` declara `"type": "module"`, `engines.node >= 22.13`, dependencias con versión fija (Vite, Vitest, `@playwright/test`) y los scripts `dev` (`vite`), `build` (`vite build`), `prestart` (`npm run build`), `start` (`node src/server/main.js`, nunca el servidor de desarrollo), `test` (`vitest run`) y `cuentas` (`node src/server/cli/cuentas.js`)
    - `README.md` explica en español cómo instalar (`npm ci`), probar (`npm test`), crear cuentas (`npm run cuentas -- alta <correo> <rol>`) y arrancar (`npm start`, variables `PORT`, `HOST`, `DB_PATH`)
    - `.gitignore` excluye `node_modules/`, `dist/`, `data/`, `evidencias/`, `test-results/`
  - **Refs:** FASE-0, REQ-C-002, REQ-NF-001, ADR-DRAFT-001
  - **Revert:** CONFIG — hay que reinstalar dependencias (`npm ci`) tras revertir
  - **Review:**
    - [ ] `start` nunca lanza `vite` en modo desarrollo
    - [ ] README cubre instalar, probar y arrancar

- [ ] TASK-F0-002 Production entry listens on PORT and HOST and serves the built app, test-first | `src/server/main.js`, `src/server/config.js`, `tests/integration/arranque.test.js`
  - blocked-by: TASK-F0-001
  - **Commit:** `feat(server): serve dist/ and /api/salud on process.env.PORT and HOST`
  - **Acceptance:**
    - Test first: `arranque: PORT y HOST` — `leerConfig({})` devuelve `{ port: 8080, host: '0.0.0.0' }`; `leerConfig({ PORT: '3000', HOST: '127.0.0.1' })` los respeta; nunca hay `127.0.0.1` fijo en el código
    - Test first: `arranque: sirve lo construido` — arrancar `crearServidor` con `PORT=0` responde 200 a `GET /` con el `index.html` de `dist/` y `{ ok: true }` a `GET /api/salud`; una ruta `/api/*` desconocida responde 404 JSON
    - `crearServidor({ manejadorApi })` acepta el manejador de API que cablea TASK-F0-008; sin él, solo atiende `/api/salud`
    - Verify: `npm run build && PORT=0 node src/server/main.js` → imprime el puerto y la dirección en una línea de log JSON, y se cierra con Ctrl+C
  - **Refs:** FASE-0, ADR-DRAFT-001, ARCHITECTURE §3
  - **Revert:** SAFE — solo quita el arranque de producción
  - **Review:**
    - [ ] El test falló antes de la implementación
    - [ ] Host por defecto `0.0.0.0` y puerto desde `process.env.PORT`

- [ ] TASK-F0-003 [P] Configure Playwright and the evidence helper | `playwright.config.js`, `tests/e2e/evidencia.js`
  - blocked-by: TASK-F0-001
  - **Commit:** `chore(e2e): configure Playwright against the Vite dev server with screenshots and video`
  - **Acceptance:**
    - Verify: `npx playwright test --list` → lista cero pruebas sin error; `webServer` usa `npx vite --host 127.0.0.1 --port 5173` con `MONTANA_TEST_CLOCK=1` y una BD temporal
    - `evidencia.js` exporta `capturar(page, testInfo, id)` que guarda `evidencias/FASE-N/{id}.png` y lo adjunta a la prueba; vídeo activado (`video: 'on'`) con el id del flujo en el nombre
    - Reporter JUnit activado para que el informe incluya capturas y vídeo
  - **Refs:** FASE-0, TEST-PLAN §2, REQ-C-002
  - **Revert:** SAFE — solo configuración de pruebas
  - **Review:**
    - [ ] Carpeta de evidencias fuera de git
    - [ ] Configuración sin rutas absolutas

---

## Phase 2: Foundation

**Purpose:** Infraestructura que comparten varios casos de uso de este incremento: textos, reloj, almacén, permisos, transporte HTTP y esqueleto de la SPA.
**Checkpoint:** Pruebas de textos, reloj, almacén, permisos y transporte en verde; `npx vite build` sin errores.

- [ ] TASK-F0-004 [P] Single Spanish text module and input validation, test-first | `src/shared/textos.js`, `src/shared/validacion.js`, `tests/unit/shared/textos.test.js`, `tests/unit/shared/validacion.test.js`
  - blocked-by: TASK-F0-001
  - **Commit:** `feat(shared): single text module and shared validators`
  - **Acceptance:**
    - Test first: `MNT-T-001 textos únicos` — cada código de error y aviso de OPERATION-MAPPING (`E_AUTH_INVALID`, `E_AUTH_LOCKED`, `E_UNITS_MIN`, `E_FORBIDDEN_ROLE`, `E_AUTH_REQUIRED`, …) tiene su literal en español exportado por `textos.js` (VO-010) y ningún otro módulo de `src/` define esos literales
    - Test first: `UNIT-T-015/016 validación` — correo (VO-001), contraseña no vacía, nombre de material (VO-006) y unidades enteras ≥ 1 (VO-007) se validan con el literal de VO-010 cuando fallan
  - **Refs:** FASE-0, REQ-C-001, VO-001, VO-006, VO-007, VO-010
  - **Revert:** SAFE — módulo nuevo sin consumidores todavía
  - **Review:**
    - [ ] Solo términos del glosario
    - [ ] Literales idénticos a VO-010

- [ ] TASK-F0-005 [P] Injectable clock, test-first | `src/domain/reloj.js`, `tests/unit/domain/reloj.test.js`
  - blocked-by: TASK-F0-001
  - **Commit:** `feat(domain): injectable clock with fixed test clock`
  - **Acceptance:**
    - Test first: `D-T-001 reloj` — `crearRelojFijo('2026-12-01T10:00:00Z').hoy()` devuelve `2026-12-01`; `avanzar(31)` mueve `ahora()` 31 minutos; `crearReloj()` sin argumentos usa la hora del sistema
    - `Dia` es `YYYY-MM-DD` e `Instante` ISO 8601 UTC (VO-003)
  - **Refs:** FASE-0, D-T-001, D-T-011, REQ-C-003
  - **Revert:** SAFE — módulo nuevo
  - **Review:**
    - [ ] El dominio no lee `Date` directamente
    - [ ] Formatos de `Dia` e `Instante` correctos

- [ ] TASK-F0-006 SQLite store with numbered migrations and in-memory double, test-first | `src/server/repo/almacen.js`, `src/server/repo/sqlite.js`, `src/server/repo/memoria.js`, `src/server/repo/migrations/001-base.sql`, `tests/integration/almacen.test.js`
  - blocked-by: TASK-F0-001
  - **Commit:** `feat(repo): SQLite store, 001-base migration and in-memory double`
  - **Acceptance:**
    - Test first: `almacen: persiste y reabre` — con un fichero temporal, una cuenta escrita sigue ahí al reabrir el almacén (ADR-DRAFT-002, REQ-NF-001); `abrirAlmacen` aplica `001-base.sql` (`cuenta`, `material`, `sesion`, `fallo_acceso`) una sola vez
    - Test first: `almacen: transacción` — `transaccion(fn)` hace rollback si `fn` lanza; el doble en memoria tiene la misma interfaz y puede fallar a demanda con `E_STORAGE_UNAVAILABLE` (D-T-002) sin escritura parcial
    - Repositorios `cuentas`, `sesiones` (una fila por correo, INV-ACC-002), `fallosAcceso` y `material` con SQL parametrizado; `CHECK` de `rol` y de `unidades_activas >= 0`
    - Ruta del fichero desde `DB_PATH` (por defecto `data/club.db`), WAL activado
  - **Refs:** FASE-0, ADR-DRAFT-002, INV-ACC-002, INV-INV-001, D-T-002, REQ-NF-001
  - **Revert:** MIGRATION — el esquema no tiene vuelta atrás automática; borrar `data/club.db` de desarrollo tras revertir
  - **Review:**
    - [ ] Todo SQL parametrizado
    - [ ] Migración numerada e idempotente al arrancar

- [ ] TASK-F0-007 Session handling and role matrix (deny by default), test-first | `src/server/http/sesion.js`, `src/server/http/permisos.js`, `tests/unit/http/permisos.test.js`
  - blocked-by: TASK-F0-004, TASK-F0-006
  - **Commit:** `feat(http): session lookup and deny-by-default role matrix`
  - **Acceptance:**
    - Test first: `INV-ACC-003 sin sesión` — `exigirSesion` sin cookie o con token desconocido da `E_AUTH_REQUIRED` (401) sin datos
    - Test first: `INV-ACC-005 denegar por defecto` — una operación sin fila en `PERMISOS` da `E_FORBIDDEN_ROLE`; `exigirRol(ctx, ['encargado'])` rechaza a un socio; `anadirMaterial` solo encargado, `listarMaterial` cualquier rol con sesión
    - La cookie es `sid`, `HttpOnly; SameSite=Strict` y el token se guarda solo como hash
  - **Refs:** FASE-0, ADR-DRAFT-003, INV-ACC-002, INV-ACC-003, INV-ACC-005, PERMISSIONS-MATRIX
  - **Revert:** SAFE — módulos nuevos
  - **Review:**
    - [ ] Ninguna operación queda fuera de la matriz
    - [ ] El token nunca se guarda en claro

- [ ] TASK-F0-008 RPC transport, Vite dev plugin and production wiring, test-first | `src/server/http/rutas.js`, `src/server/vite-plugin.js`, `src/server/main.js`, `vite.config.js`, `tests/integration/transporte.test.js`
  - blocked-by: TASK-F0-002, TASK-F0-004, TASK-F0-005, TASK-F0-006, TASK-F0-007
  - **Commit:** `feat(http): POST /api/<operation> JSON transport shared by Vite and production`
  - **Acceptance:**
    - Test first: `transporte: validación` — cuerpo > 16 KB da 400; `Content-Type` distinto de `application/json` da 400; una escritura con `Origin` ajeno da 403; JSON inválido da 400
    - Test first: `transporte: errores` — `E_AUTH_LOCKED` → 429, `E_AUTH_INVALID` y `E_AUTH_REQUIRED` → 401, `E_STORAGE_UNAVAILABLE` → 503, operación desconocida → 404; la respuesta es `{ ok:false, codigo, mensaje }` con el literal de `textos.js`
    - `crearManejador({ repo, reloj, textos })` lo usan el plugin de Vite (`vite.config.js`) y `main.js` (R2): mismo código en dev y producción; log JSON por línea sin datos sensibles
    - `GET /api/salud` responde `{ ok: true }`
  - **Refs:** FASE-0, ADR-DRAFT-001, OPERATION-MAPPING, INPUT-001, R2
  - **Revert:** COUPLED — revertir junto con las operaciones que se registran en `rutas.js`
  - **Review:**
    - [ ] Mismo manejador en dev y producción
    - [ ] Ningún mensaje de error filtra datos internos

- [ ] TASK-F0-009 SPA shell: hash router, API client, common header and message region | `src/ui/router.js`, `src/ui/api.js`, `src/ui/cabecera.js`, `src/ui/mensaje.js`, `src/ui/main.js`
  - blocked-by: TASK-F0-004, TASK-F0-008
  - **Commit:** `chore(ui): SPA shell with hash router, API client and common header`
  - **Acceptance:**
    - Verify: `npx vite build` → termina sin errores; `index.html` arranca `src/ui/main.js`
    - `llamar(operacion, cuerpo)` envía `POST /api/<operación>` con cookie y lleva a `#/iniciar-sesion` ante `E_AUTH_REQUIRED`; `mensaje.js` pinta el literal del servidor en una región `role="alert"`
    - La cabecera común muestra navegación según rol (sin «Salir» hasta FASE-6); lo cubre el journey TASK-F0-018 (Exclusión de cobertura del plan §7.4)
  - **Refs:** FASE-0, ADR-DRAFT-001, WIREFRAMES cabecera común, REQ-C-001
  - **Revert:** SAFE — solo cliente
  - **Review:**
    - [ ] Textos solo desde `textos.js`
    - [ ] Región de mensaje accesible

---

## Phase 3: Slices

**Purpose:** Un corte vertical por operación: dominio, HTTP y pantalla, con la prueba escrita antes.
**Checkpoint:** Todas las pruebas de integración de UC-001, UC-003 y UC-011 en verde.

### UC-001 — Entrar con correo y contraseña

- [ ] TASK-F0-010 Sign in with email and password (API-001-01), test-first | `src/domain/acceso.js`, `src/server/http/acceso.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/unit/domain/acceso.test.js`, `tests/integration/acceso.test.js`
  - blocked-by: TASK-F0-008
  - **Commit:** `feat(acceso): sign in with scrypt-hashed password and single session`
  - **Acceptance:**
    - Test first: `AC-001-01 AC-001-02 REQ-F-001 correo y contraseña correctos` — `POST /api/iniciarSesion` da 200, `Set-Cookie` `sid` y la sesión lleva a Material (socio) o a la vista del encargado
    - Test first: `AC-001-03 mismo error` — contraseña errónea y correo inexistente dan el mismo `E_AUTH_INVALID` (401), sin sesión ni cookie, y tardan lo mismo (CWE-204; `timingSafeEqual`)
    - Test first: `AC-001-04 INV-ACC-002 una sesión` — entrar dos veces deja una sola fila de sesión por cuenta
    - Test first: `AC-001-05 campos vacíos` — correo o contraseña vacíos se rechazan en el servidor con el literal de VO-010
    - Test first: `AC-001-06 REQ-NF-002 AC1 hash con sal` — `SEC-T-001` lee el almacén: ningún campo contiene la contraseña; dos cuentas con la misma contraseña tienen `hash` y `sal` distintos (scrypt, sal de 16 B)
  - **Refs:** FASE-0, REQ-F-001, REQ-NF-002, UC-001, API-001-01, INV-ACC-001, INV-ACC-002, ADR-DRAFT-003
  - **Revert:** COUPLED — la pantalla TASK-F0-013 depende de esta operación
  - **Review:**
    - [ ] Los tres primeros tests fallaron antes
    - [ ] Contraseña nunca en claro ni en logs

- [ ] TASK-F0-011 Login brake: 5 failures in 15 min lock the account for 15 min, test-first | `src/domain/acceso.js`, `src/server/http/acceso.js`, `src/shared/textos.js`, `tests/unit/domain/acceso.test.js`, `tests/integration/acceso.test.js`
  - blocked-by: TASK-F0-010
  - **Commit:** `feat(acceso): per-account lockout after five failed sign-ins`
  - **Acceptance:**
    - Test first: `AC-001-07 REQ-F-017 5 fallos bloquean` — con reloj fijo, el 5.º fallo en 15 min da `E_AUTH_LOCKED` (429) «Demasiados intentos fallidos: espera 15 minutos»; con la cuenta bloqueada ni la contraseña buena entra
    - Test first: `AC-001-08 espera no se alarga` — más intentos durante la espera no insertan fallos nuevos; el bloqueo acaba 15 min después del 5.º fallo (Q-spec-auditor-004)
    - Test first: `AC-001-09 AC-001-11 4 fallos y acierto` — 4 fallos no bloquean; un acierto borra los fallos de esa cuenta; fallos de hace más de 15 min no cuentan
    - Test first: `AC-001-10 por cuenta` — con la cuenta de lucia bloqueada, marta sigue entrando
    - `evaluarBloqueo(fallos, ahora)` → `{ bloqueada, hasta }` con `LOGIN_MAX_FAILURES=5`, `LOGIN_WINDOW_MIN=15`, `LOCKOUT_MIN=15`; solo cuentan fallos de correos existentes (UNIT-T-002, UNIT-T-017)
  - **Refs:** FASE-0, REQ-F-017, UC-001, INV-ACC-006, Q-spec-auditor-004, LOGIN_MAX_FAILURES, LOCKOUT_MIN
  - **Revert:** COUPLED — comparte `acceso.js` con TASK-F0-010
  - **Review:**
    - [ ] Constantes leídas de un único sitio
    - [ ] Reloj inyectable, sin esperas reales

- [ ] TASK-F0-012 Account seeding CLI without any HTTP route, test-first | `src/server/cli/cuentas.js`, `tests/integration/cuentas.test.js`
  - blocked-by: TASK-F0-011
  - **Commit:** `feat(cli): npm run cuentas to create accounts and set passwords`
  - **Acceptance:**
    - Test first: `RN-001 REQ-NF-002 cuentas por CLI` — `alta <correo> <rol>` lee la contraseña por stdin, crea la cuenta con scrypt + sal y rechaza roles distintos de socio, encargado, junta; `clave <correo>` cambia la contraseña; la cuenta creada puede entrar con `iniciarSesion`
    - Test first: `RN-001 sin alta por HTTP` — `POST /api/alta`, `/api/registro` y similares responden 404; no existe pantalla de registro (Q-requirements-engineer-001)
    - Verify: `printf 'secreto-largo\n' | npm run cuentas -- alta marta@club.es encargado` → la cuenta aparece en `data/club.db` sin la contraseña en claro
  - **Refs:** FASE-0, REQ-NF-002, REQ-F-001, RN-001, Q-requirements-engineer-001, D-PA-010
  - **Revert:** SAFE — herramienta de línea de órdenes
  - **Review:**
    - [ ] La contraseña no pasa por argumentos de la línea de órdenes
    - [ ] No hay ruta HTTP de alta

- [ ] TASK-F0-013 Sign-in screen SCR-001 and login redirect | `src/ui/pantallas/iniciar-sesion.js`, `src/ui/router.js`, `src/shared/textos.js`
  - blocked-by: TASK-F0-009, TASK-F0-012
  - **Commit:** `feat(ui): sign-in screen with server messages and role-based landing`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; el formulario tiene etiquetas Correo y Contraseña y envía a `iniciarSesion`
    - El literal de `E_AUTH_INVALID` y de `E_AUTH_LOCKED` aparece en la región `role="alert"`; tras entrar, lleva a `#/material`
    - Comportamiento comprobado por el journey TASK-F0-018 (AC-001-01, AC-001-03, AC-001-07, AC-003-03)
  - **Refs:** FASE-0, REQ-F-001, REQ-F-017, UC-001, WIREFRAMES SCR-001
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] Etiquetas y errores asociados al campo
    - [ ] Sin textos fuera de `textos.js`

### UC-003 — Ver el catálogo

- [ ] TASK-F0-014 List the catalogue with active units (API-003-01), test-first | `src/domain/inventario.js`, `src/server/http/inventario.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/unit/domain/inventario.test.js`, `tests/integration/catalogo.test.js`
  - blocked-by: TASK-F0-013
  - **Commit:** `feat(inventario): list material with active units`
  - **Acceptance:**
    - Test first: `AC-003-01 AC-003-02 REQ-F-003 catálogo` — `listarMaterial` devuelve cada material con sus unidades activas (los cinco tipos del club tras sembrarlos) a cualquier rol con sesión
    - Test first: `AC-003-03 INV-ACC-003 sin sesión` — sin cookie da `E_AUTH_REQUIRED` (401) y no devuelve datos
    - Test first: `AC-003-04 sin efectos` — pedirlo dos veces devuelve lo mismo y no cambia el almacén; `INV-INV-002` solo unidades activas
  - **Refs:** FASE-0, REQ-F-003, UC-003, API-003-01, INV-INV-002, INV-ACC-003
  - **Revert:** COUPLED — lo usa TASK-F0-015
  - **Review:**
    - [ ] Lectura sin escrituras
    - [ ] Sin sesión no se filtra ningún dato

- [ ] TASK-F0-015 Catalogue screen SCR-002 (no reservation form yet) | `src/ui/pantallas/material.js`, `src/ui/router.js`, `src/shared/textos.js`
  - blocked-by: TASK-F0-014
  - **Commit:** `feat(ui): catalogue screen with the material table`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/material` pinta una tabla Catálogo con columnas Material y Unidades activas
    - Sin sesión, `#/material` redirige a `#/iniciar-sesion` (AC-003-03); comprobado por el journey TASK-F0-018
  - **Refs:** FASE-0, REQ-F-003, UC-003, WIREFRAMES SCR-002
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] Tabla con cabeceras accesibles
    - [ ] Sin formulario de reserva (llega en FASE-1)

### UC-011 — Añadir material al inventario

- [ ] TASK-F0-016 Add material or sum units (API-003-02), test-first | `src/domain/inventario.js`, `src/server/http/inventario.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `tests/unit/domain/inventario.test.js`, `tests/integration/inventario.test.js`
  - blocked-by: TASK-F0-015
  - **Commit:** `feat(inventario): add material and sum units, encargado only`
  - **Acceptance:**
    - Test first: `AC-011-01 AC-011-02 REQ-F-011 alta y suma` — «Casco» 5 crea el material con 5; «Casco» 2 después lo deja en 7
    - Test first: `AC-011-03 INV-INV-001 E_UNITS_MIN` — 0 unidades da `E_UNITS_MIN` (400) y el inventario no cambia
    - Test first: `AC-011-04 sin idempotencia` — enviar «Casco» 2 otra vez suma otra vez (D-T-008)
    - Test first: `AC-011-05 INV-ACC-005` — un socio recibe `E_FORBIDDEN_ROLE` (403) y no cambia nada
  - **Refs:** FASE-0, REQ-F-011, UC-011, API-003-02, INV-INV-001, INV-ACC-005, D-T-008
  - **Revert:** COUPLED — lo usa TASK-F0-017
  - **Review:**
    - [ ] Alta y suma en una transacción
    - [ ] Rol comprobado en servidor

- [ ] TASK-F0-017 Inventory screen SCR-006 with the Add form (encargado only) | `src/ui/pantallas/inventario.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `src/shared/textos.js`
  - blocked-by: TASK-F0-016
  - **Commit:** `feat(ui): inventory screen with add-material form for the encargado`
  - **Acceptance:**
    - Verify: `npx vite build` → sin errores; `#/inventario` tiene formulario Material y Unidades con botón «Añadir» solo para el encargado; el enlace Inventario de la cabecera solo sale al encargado
    - Tras añadir, el catálogo y la tabla muestran la nueva cifra; el error `E_UNITS_MIN` aparece en la región de mensaje; lo comprueba el journey TASK-F0-018 (AC-011-01..03)
  - **Refs:** FASE-0, REQ-F-011, UC-011, WIREFRAMES SCR-006
  - **Revert:** SAFE — pantalla nueva
  - **Review:**
    - [ ] Un socio no ve el formulario
    - [ ] Error visible y asociado

---

## Phase 4: Integration

**Purpose:** Sin tareas de cableado separadas: el registro de operaciones y rutas viaja dentro de cada corte (un solo Stream serial).
**Checkpoint:** n/a

---

## Phase 5: Verification

**Purpose:** Journey de la FASE desde la ruta del usuario, con capturas y vídeo.
**Checkpoint:** Todos los criterios de FASE-0 verificados.

- [ ] TASK-F0-018 Journey FASE-0: entrar, añadir material y verlo en el catálogo through the user's route | `tests/e2e/fase-0.spec.js`
  - blocked-by: TASK-F0-003, TASK-F0-017
  - **Commit:** `test(e2e): FASE-0 journey from the sign-in page with captures`
  - **Acceptance:**
    - Test first: nombrado `AC-001-01 …`, `AC-001-02`, `AC-001-03`, `AC-001-04`, `AC-001-05`, `AC-001-06`, `AC-001-07`, `AC-001-08`, `AC-001-09`, `AC-001-10`, `AC-001-11`, `AC-011-01`, `AC-011-02`, `AC-011-03`, `AC-011-04`, `AC-011-05`, `AC-003-01`, `AC-003-02`, `AC-003-03`, `AC-003-04` (REQ-F-001, REQ-F-017, REQ-F-011, REQ-F-003)
    - El título del journey lleva `FASE-0` (la FASE no nombra workflow): es el id del vídeo y lo que busca `sdd gate --fase 0`
    - Entra por el paso 1 de la Demo: abrir `/` sin sesión y comprobar Iniciar sesión (AC-003-03), nunca por un punto de entrada interno
    - Comprueba el texto de cada criterio sobre el elemento que lo muestra (`toHaveText`/`toContainText`): «Correo o contraseña incorrectos», «Demasiados intentos fallidos: espera 15 minutos», «Casco» con «5», luego «7», el literal de `E_UNITS_MIN` con «7» intacto
    - Guarda `evidencias/FASE-0/{AC-NNN-NN}.png` por criterio y un vídeo con `FASE-0` en el nombre, adjuntos a la prueba (JUnit)
  - **Refs:** FASE-0, REQ-F-001, REQ-F-017, REQ-F-011, REQ-F-003, REQ-NF-002, AC-001-01..11, AC-011-01..05, AC-003-01..04
  - **Revert:** SAFE — solo prueba
  - **Review:**
    - [ ] Cada escenario cubierto por un `test` con su id
    - [ ] Capturas guardadas para cada criterio

### Test Exclusions

| File | Reason | Verified By |
|------|--------|-------------|
| `src/server/vite-plugin.js` | arranque del servidor de desarrollo | `tests/integration/transporte.test.js` y el journey TASK-F0-018 |
| `src/ui/router.js`, `src/ui/cabecera.js`, `src/ui/mensaje.js`, `src/ui/api.js`, `src/ui/main.js` | cableado de la SPA | journey TASK-F0-018 |
| `src/ui/pantallas/*.js` | pantallas | journey TASK-F0-018 |

---

## Dependencies

### Task Dependency Graph

```text
001 ─┬─► 002 ──────────────┐
     ├─► 003 ─────────────────────────────────────────────┐
     ├─► 004 ─┬────────────┤                               │
     ├─► 005 ─┼────────────┤                               │
     └─► 006 ─┴─► 007 ─────┴─► 008 ─► 009 ─┐               │
                              008 ─► 010 ─► 011 ─► 012 ─► 013 ─► 014 ─► 015 ─► 016 ─► 017 ─► 018
```

### Critical Path

1. TASK-F0-001 → 006 → 007 → 008 → 010 → 011 → 012 → 013 → 014 → 015 → 016 → 017 → 018
   (13 tasks on critical path)

### Parallel Execution Plan

**Stream base:** scaffold, textos, reloj, almacén, permisos, transporte y esqueleto de la SPA (004, 005 y 003 en paralelo tras 001).
**Stream A:** cortes de UC-001, UC-003 y UC-011 en serie (comparten `rutas.js`, `textos.js` y `router.js`).

## Stream Ownership

| Stream | Tasks | Owns (write-set) | Runs in |
|--------|-------|------------------|---------|
| base | TASK-F0-001, TASK-F0-002, TASK-F0-003, TASK-F0-004, TASK-F0-005, TASK-F0-006, TASK-F0-007, TASK-F0-008, TASK-F0-009 | package.json, vite.config.js, vitest.config.js, index.html, .gitignore, README.md, playwright.config.js, src/shared/validacion.js, src/domain/reloj.js, src/server/main.js, src/server/config.js, src/server/vite-plugin.js, src/server/repo/**, src/server/http/sesion.js, src/server/http/permisos.js, src/ui/main.js, src/ui/api.js, src/ui/mensaje.js, tests/e2e/evidencia.js, tests/unit/shared/**, tests/unit/domain/reloj.test.js, tests/unit/http/**, tests/integration/arranque.test.js, tests/integration/almacen.test.js, tests/integration/transporte.test.js | main checkout, before worktrees (checkpoint `fase-0-foundation`) |
| A | TASK-F0-010, TASK-F0-011, TASK-F0-012, TASK-F0-013, TASK-F0-014, TASK-F0-015, TASK-F0-016, TASK-F0-017 | src/domain/acceso.js, src/domain/inventario.js, src/server/http/acceso.js, src/server/http/inventario.js, src/server/http/rutas.js, src/server/cli/**, src/shared/textos.js, src/ui/pantallas/**, src/ui/router.js, src/ui/cabecera.js, tests/unit/domain/acceso.test.js, tests/unit/domain/inventario.test.js, tests/integration/acceso.test.js, tests/integration/catalogo.test.js, tests/integration/inventario.test.js, tests/integration/cuentas.test.js | worktree `feat/fase-0-a` |
| integración | — | — | main checkout, after `--integrate --fase 0` |
| verificación | TASK-F0-018 | — | main checkout, Phase 9 |

### Rollback Checkpoints

| Checkpoint | After Task | Tag | Runs in |
|-----------|------------|-----|---------|
| Foundation | TASK-F0-009 | `fase-0-foundation` | main checkout |
| Verified | TASK-F0-018 | `fase-0-verified` | main checkout |
