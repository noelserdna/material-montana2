# FASE 6: Salir y caducidad de la sesión

> **Estado:** Implementable
> **Incremento:** Cualquier usuario cierra su sesión con «Salir», y la sesión se cierra sola tras 30 minutos sin actividad con un aviso al volver
> **Requisitos:** REQ-F-018, REQ-F-019
> **Escenarios:** AC-017-01, AC-017-02, AC-017-03, AC-018-01, AC-018-02, AC-018-03, AC-018-04, AC-018-05
> **Workflows:** —
> **Necesidades:** N-002, N-011
> **Dependencias:** Fase 5

---

## Objetivo

Permitir que un **socio, encargado o junta** termine su sesión con el botón «Salir» de la cabecera y que el servidor la cierre tras `SESSION_IDLE_MIN` (30) minutos sin actividad, distinguiendo «sesión caducada» de «sin sesión». Las pantallas y operaciones ya existen: esta fase cambia cómo cada una trata la sesión.

## Criterios de Éxito

### UC-017 — Cerrar sesión
- [ ] «Salir» borra la sesión y la cookie y lleva a Iniciar sesión sin mensaje (AC-017-01, REQ-F-018)
- [ ] Con la sesión cerrada, cualquier operación da `E_AUTH_REQUIRED` sin datos (AC-017-02, INV-ACC-003)
- [ ] Pulsar «Salir» otra vez devuelve 200 sin cambios y va a Iniciar sesión (AC-017-03)
- [ ] El botón «Salir» está en la cabecera común de toda pantalla con sesión (REQ-F-018, R-8)

### UC-018 — Caducidad por inactividad
- [ ] Con más de 30 min sin actividad (30:00 exactos aún vale) la operación da `E_SESSION_EXPIRED` «Tu sesión ha caducado» (AC-018-01, REQ-F-019)
- [ ] Otro socio caducado ve el mismo mensaje (AC-018-05, REQ-F-019 AC2)
- [ ] Dentro del plazo la operación funciona (AC-018-02, REQ-F-019 AC3)
- [ ] Cada operación con sesión vigente renueva los 30 minutos (AC-018-03, INV-ACC-007)
- [ ] Entrar de nuevo tras caducar abre sesión normal (AC-018-04)

### Transversal
- [ ] Las pruebas usan el reloj inyectable, nunca esperas reales (D-T-011, UNIT-T-018, REQ-F-019)
- [ ] La SPA muestra el mensaje de caducidad en Iniciar sesión y lo borra al entrar (INTERACTION-MODEL v1.1, AC-018-01, REQ-F-019)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-017-cerrar-sesion.md` | flujo, AC-017-01..03 | Salir |
| `use-cases/UC-018-sesion-caduca-por-inactividad.md` | flujo, AC-018-01..05 | caducidad |

### Dominio y contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/05-INVARIANTS.md` | INV-ACC-002, INV-ACC-003, INV-ACC-007 | sesión |
| `contracts/API-acceso.md` | API-001-02 | cerrarSesion |
| `VALUE-REGISTRY.md` | SESSION_IDLE_MIN | 30 min |

### Diseño y plan de tests

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/TECHNICAL-DESIGN.md` | §7 caducidad y cerrar sesión | `ultima_actividad`, `E_SESSION_EXPIRED` (el umbral lo fija INV-ACC-007) |
| `design/OPERATION-MAPPING.md` | API-001-02, transporte obligado por REQ-F-019 | ruta y 401 |
| `ux/INTERACTION-MODEL.md`, `WIREFRAMES.md` | v1.1, cabecera común | Salir y mensaje |
| `test/TEST-PLAN.md` | UNIT-T-018, INT-T-002, D-T-011, GAP-011 | pruebas con reloj |
| `test/TEST-MATRIX-UC-017.md`, `UC-018.md` | filas | integración |

## Invariantes Aplicables

> Acumulativas: hereda las de FASE-0..FASE-5.

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-ACC-007 | A · `acceso.cerrarSesion`, `http/sesion.exigirSesion` (renueva `ultima_actividad`) |
| INV-ACC-003 | A · `http/sesion.exigirSesion` (distingue caducada de ausente) |
| INV-ACC-002 | A · `repo.sesiones` (una fila por cuenta) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | sesión (serial) | `src/domain/acceso.js`, `src/server/repo/sesiones.js`, `src/server/repo/migrations/004-sesion-actividad.sql`, `src/server/http/acceso.js`, `src/server/http/sesion.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `src/ui/cabecera.js`, `src/ui/api.js`, `src/ui/pantallas/iniciar-sesion.js`, `tests/unit/domain/sesion.test.js`, `tests/integration/sesion.test.js`, `tests/e2e/fase-6.spec.js` | operaciones de negocio | FASE-5 |

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-001-02 | `POST /api/cerrarSesion` · `cerrarSesion()` | termina la sesión; repetir = 200 sin cambios |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/iniciar-sesion` | `src/ui/pantallas/iniciar-sesion.js` | UC-018 | WIREFRAMES §SCR-001 | muestra «Tu sesión ha caducado» |
| (todas con sesión) | `src/ui/cabecera.js` | UC-017 | WIREFRAMES cabecera común | botón «Salir» |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `api` (`src/ui/api.js`) | todas | convierte `E_SESSION_EXPIRED` y `E_AUTH_REQUIRED` en redirección |

## Verificación

```bash
npx vitest run tests/unit/domain/sesion.test.js --maxWorkers=2     # UNIT-T-018 con reloj inyectable
npx vitest run tests/integration/sesion.test.js --maxWorkers=2     # AC-017-*, AC-018-*, INT-T-002
npm test                                                           # suite completa verde
npx playwright test tests/e2e/fase-6.spec.js                       # C-017-*, C-018-* con reloj del navegador
```

## Demo

> Desde un checkout limpio: igual que FASE-5; el servidor arranca con reloj controlable solo para pruebas.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | Lucia entra y pulsa «Salir» | vuelve a Iniciar sesión sin mensaje | AC-017-01 · N-002 |
| 2 | Atrás en el navegador y abrir Mis reservas | Iniciar sesión, sin datos | AC-017-02 · N-011 |
| 3 | Pulsar «Salir» otra vez (petición repetida) | 200 sin cambios | AC-017-03 · N-002 |
| 4 | Lucia entra y la hora avanza 29 min; abre Material | funciona | AC-018-02 · N-002 |
| 5 | Pasan 29 min más desde esa actividad; abre Mis reservas | funciona: la actividad renovó | AC-018-03 · N-002 |
| 6 | Pasan 30 min sin actividad; abre Material | «Tu sesión ha caducado» en Iniciar sesión | AC-018-01 · N-011 |
| 7 | Pablo, también caducado, abre Material | mismo mensaje | AC-018-05 · N-011 |
| 8 | Lucia vuelve a entrar | sesión normal | AC-018-04 · N-002 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-017, UC-018 | Cambiar o restablecer contraseña (CLI `cuentas -- clave`) |
| Migración `ultima_actividad` | Cierre de sesiones en otros dispositivos |
