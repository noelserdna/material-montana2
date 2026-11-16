# Implementation Plan — FASE-0: Entrar, añadir material y verlo en el catálogo

> Spec v1.0 · 2026-11-16 · Depends on: — · Enables: FASE-1
> Blocks: A (serial; ver FASE-0 §Módulos y Conjuntos de Escritura)

## 1. FASE Overview

**Objective:** el encargado entra, añade material y los socios lo ven · **Scope:** esqueleto completo (SPA, API, dominio, SQLite, CLI de cuentas) · **Delivers:** API-001-01, API-003-01, API-003-02

## 2. Specs to Implement

| Spec | Type | Implementation note |
|------|------|---------------------|
| UC-001 | UC | `iniciarSesion`; mismo error para correo inexistente y contraseña mala |
| UC-011 | UC | alta que suma si el material existe |
| UC-003 | UC | catálogo solo con unidades activas |
| INV-ACC-006 | INV | 5 fallos en 15 min; espera desde el 5.º fallo, no se alarga |
| ADR-DRAFT-002 | ADR | `node:sqlite`, migración `001-base.sql` al arrancar |
| ADR-DRAFT-003 | ADR | cookie de sesión `HttpOnly`, scrypt con sal de 16 B |

## 3. Technical Decisions (FASE-Specific)

| ID | Decision | Ref |
|----|----------|-----|
| D-PA-002 | el freno entra con el login | CLARIFY-LOG |
| D-PA-009 | `MONTANA_TEST_CLOCK=1` habilita el avance de reloj en pruebas | CLARIFY-LOG |
| D-PA-010 | CLI `npm run cuentas -- alta <correo> <rol>` | CLARIFY-LOG |

## 4. Component Implementation

### 4.1 `src/domain/reloj.js` (A)

**Responsibility:** «hoy» y hora inyectables · **Refs:** D-T-001, D-T-011

```js
export function crearReloj(fuente = () => new Date()) // { ahora(): Instante, hoy(): Dia }
export function crearRelojFijo(instante)              // { ...reloj, avanzar(minutos) }
```

### 4.2 `src/domain/acceso.js` (A)

**Responsibility:** UC-001, INV-ACC-001/002/006 · **Refs:** ADR-DRAFT-003

```js
export async function iniciarSesion({ repo, reloj }, correo, contrasena) // → { ok, sesion } | { ok:false, codigo }
export function evaluarBloqueo(fallos, ahora)  // → { bloqueada, hasta }
export function hashContrasena(contrasena)     // { hash, sal }
```

**Notes:**
- Comparación con `timingSafeEqual`; un correo inexistente cuesta lo mismo que uno existente (CWE-204).
- Solo cuentan fallos de correos existentes; un acierto borra los fallos de esa cuenta.
- Con la cuenta bloqueada no se inserta fallo nuevo: la espera no se alarga (Q-spec-auditor-004).

### 4.3 `src/domain/inventario.js` (A)

**Responsibility:** UC-011, UC-003 · **Refs:** INV-INV-001, INV-INV-002

```js
export function anadirMaterial({ repo }, material, unidades) // suma o crea
export function listarMaterial({ repo })                     // solo unidadesActivas
```

### 4.4 `src/server/repo/almacen.js`, `sqlite.js`, `memoria.js` (A)

**Responsibility:** puerto de persistencia con `transaccion(fn)`; doble en memoria y doble que falla (`E_STORAGE_UNAVAILABLE`) · **Refs:** D-T-002, ADR-DRAFT-002

```js
export function abrirAlmacen({ ruta })       // aplica migrations/NNN-*.sql en orden
export const cuentas, sesiones, fallosAcceso, material // repositorios con SQL parametrizado
```

### 4.5 `src/server/http/*` (A)

**Responsibility:** transporte RPC, sesión, permisos, errores · **Refs:** OPERATION-MAPPING, INPUT-001

```js
export function crearManejador({ repo, reloj, textos }) // (req, res) para el plugin de Vite y main.js
export function exigirSesion(ctx), exigirRol(ctx, roles)
```

**Notes:**
- Cuerpo JSON ≤ 16 KB, `Content-Type: application/json`, comprobación de `Origin` en escrituras.
- Denegar por defecto: una operación sin entrada en la tabla de permisos responde `E_FORBIDDEN_ROLE`.
- `E_AUTH_LOCKED` → 429; `E_AUTH_INVALID` → 401; resto según OPERATION-MAPPING.

### 4.6 `src/server/cli/cuentas.js` (A)

`alta <correo> <rol>` y `clave <correo>`; contraseña por stdin; no existe ruta HTTP (RN-001).

### 4.7 Puertos con doble

No external ports in this FASE.

## 5. API Implementation Notes

| API-op | Handler | Note |
|--------|---------|------|
| API-001-01 | `http/acceso.js#iniciarSesion` | sin sesión previa; `Set-Cookie` con `SESSION_SECURE` según entorno |
| API-003-01 | `http/inventario.js#listarMaterial` | cualquier rol con sesión |
| API-003-02 | `http/inventario.js#anadirMaterial` | solo encargado |

## 6. Data Changes

| Migration | Description | Reversible |
|-----------|-------------|-----------|
| 001-base | `cuenta`, `material`, `sesion`, `fallo_acceso` | No |

## 7. Test Strategy

### 7.1 Unit Tests

| Component | Test ids (test/) | Block |
|-----------|------------------|-------|
| `src/domain/acceso.js` | UNIT-T-001, UNIT-T-002, UNIT-T-017 | A |
| `src/domain/inventario.js` | UNIT-T-013, UNIT-T-014 | A |
| `src/shared/validacion.js` | UNIT-T-015, UNIT-T-016 | A |

### 7.2 Integration Tests

| Scenario | Test ids (test/) | Setup | Block |
|----------|------------------|-------|-------|
| Acceso y freno | TEST-MATRIX-UC-001, AC-001-01..11 | in-memory repository, reloj fijo | A |
| Alta y catálogo | TEST-MATRIX-UC-011, UC-003, AC-011-01..05, AC-003-01..04 | in-memory repository | A |
| Seguridad | SEC-T-001, SEC-T-003, INT-T-002 | in-memory repository | A |

### 7.3 BDD / E2E Scenarios

| Source | Scenarios | Tier / Priority | Block |
|--------|-----------|-----------------|-------|
| `tests/BDD-UC-001.md`, `UC-011`, `UC-003` | AC-001-01..11, AC-011-01..05, AC-003-01..04 | P0/P1 | A |
| `test/E2E-SCENARIOS.md` | Smoke: S0 · Critical: C-001-*, C-011-*, C-003-* | por nivel | A |

### 7.4 Test Coverage Map (Source → Test)

| Source File | Type | Test File | Priority |
|------------|------|-----------|----------|
| `src/domain/acceso.js` | logic | `tests/unit/domain/acceso.test.js` (A) | HIGH |
| `src/domain/inventario.js` | logic | `tests/unit/domain/inventario.test.js` (A) | HIGH |
| `src/domain/reloj.js` | logic | `tests/unit/domain/reloj.test.js` (A) | MEDIUM |
| `src/shared/validacion.js` | logic | `tests/unit/shared/validacion.test.js` (A) | MEDIUM |
| `src/server/http/permisos.js` | service | `tests/unit/http/permisos.test.js` (A) | HIGH |
| `src/server/http/rutas.js` | service | `tests/integration/acceso.test.js` (A) | HIGH |
| `src/server/repo/sqlite.js` | infrastructure | `tests/integration/almacen.test.js` (A) | MEDIUM |
| `src/ui/pantallas/iniciar-sesion.js` | page | `tests/e2e/fase-0.spec.js` (A) | MEDIUM |
| `src/ui/pantallas/material.js` | page | `tests/e2e/fase-0.spec.js` (A) | MEDIUM |
| `src/ui/pantallas/inventario.js` | page | `tests/e2e/fase-0.spec.js` (A) | MEDIUM |

**Exclusions (no unit test needed):**

| File | Reason |
|------|--------|
| `src/shared/textos.js` | constantes; lo cubre MNT-T-001 |
| `src/server/main.js`, `src/server/vite-plugin.js` | arranque; lo cubre la prueba de humo de la FASE |
| `src/server/cli/cuentas.js` | envoltorio de CLI; lo cubre `tests/integration/cuentas.test.js` |
| `src/ui/router.js`, `src/ui/cabecera.js`, `src/ui/mensaje.js` | cubiertos por E2E |

## 8. Dependencies on Shared Components

Ninguna: esta FASE crea reloj, almacén, permisos, textos y cabecera que usan las siguientes.

## 9. Acceptance Criteria

- [ ] AC-001-01..11, AC-011-01..05, AC-003-01..04 pasan
- [ ] Ningún campo del almacén contiene una contraseña en claro (SEC-T-001)
- [ ] `package.json` y `vitest.config` cumplen REQ-C-002 (MNT-T-002)
- [ ] Todo literal sale de `textos.js` (MNT-T-001)
- [ ] No new security findings

---

> Generated by sdd-plan-architect | Spec v1.0 | 2026-11-16
