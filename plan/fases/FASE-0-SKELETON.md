# FASE 0: Entrar, añadir material y verlo en el catálogo

> **Estado:** Implementable
> **Incremento:** El encargado entra con su correo y contraseña, añade material al inventario y cualquier socio lo ve en el catálogo, también tras recargar y desde otro navegador
> **Requisitos:** REQ-F-001, REQ-F-017, REQ-F-011, REQ-F-003, REQ-NF-002
> **Escenarios:** AC-001-01, AC-001-02, AC-001-03, AC-001-04, AC-001-05, AC-001-06, AC-001-07, AC-001-08, AC-001-09, AC-001-10, AC-001-11, AC-011-01, AC-011-02, AC-011-03, AC-011-04, AC-011-05, AC-003-01, AC-003-02, AC-003-03, AC-003-04
> **Workflows:** —
> **Necesidades:** N-002, N-003, N-008, N-011
> **Dependencias:** Ninguna (fase inicial)

---

## Objetivo

Permitir que un **encargado** entre, añada unidades de un material y que un **socio** entre y vea el catálogo. Recorre todas las capas (SPA → API `POST /api/<operación>` → dominio → SQLite) y deja montado lo que usan las demás fases: sesión en servidor, freno de intentos, reloj inyectable, módulo de textos y semilla de cuentas por CLI.

## Criterios de Éxito

### UC-001 — Entrar con correo y contraseña
- [ ] Un correo y contraseña correctos abren sesión y llevan a Material (AC-001-01, AC-001-02, REQ-F-001)
- [ ] Contraseña errónea y correo inexistente dan el mismo `E_AUTH_INVALID`, sin abrir sesión (AC-001-03, REQ-F-001)
- [ ] Repetir el acceso mantiene una sola sesión por cuenta (AC-001-04, INV-ACC-002)
- [ ] Campos vacíos se rechazan en servidor con el literal de VO-010 (AC-001-05)
- [ ] La contraseña se guarda solo como hash con sal; dos cuentas iguales difieren (AC-001-06, REQ-NF-002 AC1)
- [ ] 5 fallos en 15 min bloquean la cuenta 15 min desde el último fallo; la espera no se alarga (AC-001-07, AC-001-08, REQ-F-017)
- [ ] 4 fallos no bloquean, un acierto pone el contador a cero y los fallos viejos no cuentan (AC-001-09, AC-001-11, REQ-F-017)
- [ ] El bloqueo es por cuenta: otra cuenta sigue entrando (AC-001-10, REQ-F-017)

### UC-011 — Añadir material al inventario
- [ ] El encargado añade material nuevo o suma unidades a uno existente (AC-011-01, AC-011-02, REQ-F-011)
- [ ] Cero unidades da `E_UNITS_MIN` y el inventario no cambia (AC-011-03, INV-INV-001)
- [ ] Enviar otra vez suma otra vez, sin idempotencia (AC-011-04, D-T-008)
- [ ] Un socio no puede añadir material: `E_FORBIDDEN_ROLE` (AC-011-05, INV-ACC-005)

### UC-003 — Ver el catálogo
- [ ] El catálogo lista los cinco tipos con sus unidades activas (AC-003-01, AC-003-02, REQ-F-003)
- [ ] Sin sesión no hay datos y se va a Iniciar sesión (AC-003-03, INV-ACC-003, REQ-F-016 AC4)
- [ ] Abrir el catálogo otra vez no cambia nada (AC-003-04)

### Transversal
- [ ] Textos de interfaz y errores salen del módulo único `src/shared/textos.js`, en español (REQ-C-001, MNT-T-001)
- [ ] `package.json` declara módulos ES, Vite y Vitest; `maxWorkers: 2` (REQ-C-002, MNT-T-002)
- [ ] El almacén de servidor sobrevive al reinicio y lo leen todas las sesiones (ADR-DRAFT-002, REQ-NF-001)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-001-entrar-con-correo.md` | flujo, AC-001-01..11 | login, bloqueo |
| `use-cases/UC-011-anadir-material.md` | flujo, AC-011-01..05 | alta de material |
| `use-cases/UC-003-ver-catalogo.md` | flujo, AC-003-01..04 | catálogo |

### Dominio y transversales

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/01-GLOSSARY.md`, `CLARIFICATIONS.md` | RN-001, RN-005 | lenguaje y cuentas por CLI |
| `domain/02-ENTITIES.md` | ENT-001 Cuenta, ENT-002 Material, ENT-005 Sesión | tablas |
| `domain/03-VALUE-OBJECTS.md` | VO-001, VO-006, VO-007, VO-010 | formatos y literales |
| `VALUE-REGISTRY.md` | LOGIN_MAX_FAILURES, LOGIN_WINDOW_MIN, LOCKOUT_MIN, SESSION_IDLE_MIN | constantes |

### Contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `contracts/API-acceso.md` | API-001-01 | iniciarSesion |
| `contracts/API-inventario.md` | API-003-01, API-003-02 | listar, añadir |
| `contracts/PERMISSIONS-MATRIX.md` | filas de acceso e inventario | rol por operación |

### Diseño y UX

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/TECHNICAL-DESIGN.md` | §4–§7, §9 | capas, SQLite, sesión, freno |
| `design/OPERATION-MAPPING.md` | API-001-01, API-003-01..02 | rutas RPC |
| `ux/WIREFRAMES.md` | SCR-001, SCR-002, SCR-006, cabecera común | pantallas |

### Plan de tests (test/)

| Documento | ids | Para |
|-----------|-----|------|
| `test/TEST-PLAN.md` | D-T-001, D-T-003, D-T-005, D-T-011, D-T-012, R-1..R-5, R-8 | reloj, almacén, textos |
| `test/TEST-PLAN.md` | UNIT-T-001, 002, 013, 014, 015, 017; SEC-T-001, 003 | unit y seguridad |
| `test/TEST-MATRIX-UC-001.md`, `UC-011`, `UC-003`; `test/E2E-SCENARIOS.md` | S0, C-001-*, C-011-*, C-003-* | integración y E2E |

## Invariantes Aplicables

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-ACC-001 | A · `acceso.hashContrasena` / `repo.cuentas` |
| INV-ACC-002 | A · `repo.sesiones.abrir` (una fila por cuenta) |
| INV-ACC-003 | A · `http/sesion.exigirSesion` (todas las operaciones) |
| INV-ACC-005 | A · `http/permisos.exigirRol` (denegar por defecto) |
| INV-ACC-006 | A · `acceso.evaluarBloqueo` / `repo.fallosAcceso` |
| INV-INV-001 | A · `inventario.anadirMaterial` |
| INV-INV-002 | A · `inventario.listarMaterial` (solo activas) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | todo (fase inicial, serial) | `package.json`, `vite.config.js`, `vitest.config.js`, `index.html`, `src/shared/**`, `src/domain/**`, `src/server/**`, `src/ui/**`, `tests/unit/**`, `tests/integration/**`, `tests/e2e/**`, `.github/workflows/ci.yml` | `spec/`, `plan/` | — |

## Contenido Específico

### Constantes del freno (de `VALUE-REGISTRY.md`)

| Constante | Valor | Uso |
|-----------|-------|-----|
| `LOGIN_MAX_FAILURES` | 5 | fallos que bloquean |
| `LOGIN_WINDOW_MIN` / `LOCKOUT_MIN` | 15 / 15 | ventana y espera, contadas desde el 5.º fallo (Q-spec-auditor-004) |

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-001-01 | `POST /api/iniciarSesion` · `iniciarSesion(correo, contrasena)` | abre sesión; `E_AUTH_INVALID`, `E_AUTH_LOCKED` |
| API-003-01 | `POST /api/listarMaterial` · `listarMaterial()` | materiales con unidades activas |
| API-003-02 | `POST /api/anadirMaterial` · `anadirMaterial(material, unidades)` | suma unidades; solo encargado |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/iniciar-sesion` | `src/ui/pantallas/iniciar-sesion.js` | UC-001 | WIREFRAMES §SCR-001 | formulario Correo y Contraseña |
| `#/material` | `src/ui/pantallas/material.js` | UC-003 | WIREFRAMES §SCR-002 | tabla Catálogo (sin formulario de reserva aún) |
| `#/inventario` | `src/ui/pantallas/inventario.js` | UC-011 | WIREFRAMES §SCR-006 | formulario Añadir (solo encargado) |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `cabecera` (`src/ui/cabecera.js`) | `#/material`, `#/inventario` | navegación por rol; el botón «Salir» llega en FASE-6 |
| `mensaje` (`src/ui/mensaje.js`) | todas | región `role="alert"` para el literal del servidor |

## Verificación

```bash
npm test                                        # unit + integración verdes (UNIT-T-001/002/013..015/017, matrices UC-001/011/003)
npx vitest run tests/integration/acceso.test.js --maxWorkers=2   # AC-001-01..11
npx vite build                                  # build sin errores
curl -s -X POST http://127.0.0.1:5173/api/listarMaterial         # 401 E_AUTH_REQUIRED sin cookie
npx playwright test tests/e2e/fase-0.spec.js    # Smoke S0 y C-001-*, C-011-*, C-003-* con capturas
```

## Demo

> Desde un checkout limpio: `npm ci` · `npm run cuentas -- alta marta@club.es encargado` (y `lucia@club.es` socio) · `npx vite --host 127.0.0.1 --port 5173`.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | Abrir `/` sin sesión | se muestra Iniciar sesión | AC-003-03 · N-002 |
| 2 | Entrar como lucia con contraseña errónea | «Correo o contraseña incorrectos» sin abrir sesión | AC-001-03 · N-002 |
| 3 | Entrar como lucia con la contraseña buena | Material con el catálogo | AC-001-01 · N-002, N-003 |
| 4 | Salir del navegador y entrar como marta; ir a Inventario | formulario Añadir visible | AC-001-02 · N-008 |
| 5 | Añadir «Casco», 5 unidades | Casco con 5 unidades | AC-011-01 · N-008 |
| 6 | Añadir «Casco», 2 unidades | Casco con 7 unidades | AC-011-02 · N-008 |
| 7 | Añadir «Casco», 0 unidades | error de mínimo de unidades; sigue 7 | AC-011-03 · N-008 |
| 8 | Recargar y abrir el catálogo como lucia en otro navegador | Casco con 7 unidades | AC-003-02, AC-003-04 · N-003 |
| 9 | Fallar 5 veces con lucia | «Demasiados intentos fallidos: espera 15 minutos» | AC-001-07 · N-011 |
| 10 | Entrar como marta durante esa espera | entra: el bloqueo es por cuenta | AC-001-10 · N-011 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-001 (con freno REQ-F-017), UC-011, UC-003; esqueleto Vite + API + SQLite | Reservar (FASE-1), «Salir» y caducidad (FASE-6) |
| Semilla de cuentas por CLI (RN-001) | Pantalla de registro (no existe por decisión del cliente) |

## Notas

- Q-requirements-engineer-001: el encargado prepara las cuentas; no hay ruta HTTP de alta.
- La baja de unidades (UC-012) llega en FASE-4; el catálogo ya lee «unidades activas».
