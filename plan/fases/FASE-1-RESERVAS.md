# FASE 1: Reservar, ver y cancelar mis reservas

> **Estado:** Implementable
> **Incremento:** Un socio reserva material para unas fechas si hay unidades libres, ve sus reservas y las cancela hasta el día antes
> **Requisitos:** REQ-F-002, REQ-F-004, REQ-F-005
> **Escenarios:** AC-002-01, AC-002-02, AC-002-03, AC-002-04, AC-002-05, AC-002-06, AC-002-07, AC-002-09, AC-002-10, AC-004-01, AC-004-02, AC-004-03, AC-004-04, AC-004-05, AC-004-06, AC-005-01, AC-005-02, AC-005-03, AC-005-04, AC-005-05, AC-005-06, AC-005-07
> **Workflows:** WF-001
> **Necesidades:** N-003, N-004
> **Dependencias:** Fase 0

---

## Objetivo

Permitir que un **socio** reserve un material para un rango de días sin sobrepasar las unidades activas (INV-RES-001), consulte solo sus reservas y cancele una reserva que aún no ha empezado. Arranca el paso 1 de WF-001.

## Criterios de Éxito

### UC-002 — Reservar material
- [ ] Reservar con todo libre crea una reserva `Reservada` visible en Mis reservas (AC-002-01, REQ-F-002)
- [ ] Queda una unidad libre un día y se reserva; sin unidades da `E_NO_STOCK` (AC-002-02, AC-002-03, INV-RES-001)
- [ ] Días sin solape no compiten por stock; el rango incluye ambos extremos (AC-002-04, RN-003)
- [ ] Repetir el formulario da `E_DUP_RESERVATION` antes que `E_NO_STOCK` (AC-002-05, INV-RES-003, D-T-004)
- [ ] `hasta` anterior a `desde` da `E_DATE_RANGE`; no se añaden más comprobaciones de fecha (AC-002-06, Q-spec-auditor-002)
- [ ] La junta no reserva (`E_FORBIDDEN_ROLE`) y un material inexistente da `E_MATERIAL_NOT_FOUND` (AC-002-07, AC-002-09)
- [ ] Dos reservas simultáneas por la última unidad: solo una entra (AC-002-10, INV-RES-001, `BEGIN IMMEDIATE`)

### UC-004 — Ver mis reservas
- [ ] Cada socio ve solo las suyas, con material, fechas y estado (AC-004-01, AC-004-02, REQ-F-004)
- [ ] Sin reservas se muestra «Sin reservas»; sin sesión no hay datos (AC-004-03, AC-004-04)
- [ ] Las reservas de otro socio no salen nunca (AC-004-05, INV-ACC-004)
- [ ] Abrir otra vez no cambia nada (AC-004-06)

### UC-005 — Cancelar una reserva
- [ ] Cancelar antes del día `desde` deja `Cancelada` y libera la unidad (AC-005-01, AC-005-02, INV-RES-004, REQ-F-005)
- [ ] El mismo día de `desde` da `E_CANCEL_TOO_LATE` y la reserva sigue (AC-005-03, INV-RES-004)
- [ ] Cancelar dos veces da el aviso `O_ALREADY_CANCELLED` sin cambios (AC-005-04)
- [ ] Reserva ajena: `E_FORBIDDEN_DATA` y sigue `Reservada` (AC-005-05, SEC-T-004)
- [ ] La junta no cancela y una reserva inexistente da `E_RES_NOT_FOUND` (AC-005-06, AC-005-07)

### Transversal
- [ ] Reservar y cancelar usan el reloj inyectable para «hoy» (INV-RES-004, D-T-001, REQ-C-003)
- [ ] Textos nuevos solo en `src/shared/textos.js` (REQ-C-001)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-002-reservar-material.md` | flujo, AC-002-01..10 | reservar |
| `use-cases/UC-004-ver-mis-reservas.md` | flujo, AC-004-01..06 | listado |
| `use-cases/UC-005-cancelar-reserva.md` | flujo, AC-005-01..07 | cancelar |
| `workflows/WF-001-ciclo-de-reserva.md` | paso 1 | ciclo |

### Dominio y contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/02-ENTITIES.md`, `04-STATES.md` | ENT-003 Reserva, SM-001 | tabla y transiciones |
| `domain/03-VALUE-OBJECTS.md` | VO-003 Dia, VO-004, VO-010 | formatos y literales |
| `contracts/API-reservas.md` | API-002-01..03 | firmas |
| `CLARIFICATIONS.md` | RN-003 | rango inclusivo |

### Diseño y plan de tests

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/OPERATION-MAPPING.md` | API-002-01..03 | rutas RPC |
| `ux/WIREFRAMES.md` | SCR-002 (formulario), SCR-003 | pantallas |
| `test/TEST-PLAN.md` | D-T-004, UNIT-T-004..008, SEC-T-004 | orden de errores |
| `test/TEST-MATRIX-UC-002.md`, `UC-004`, `UC-005` | filas, race | integración |

## Invariantes Aplicables

> Acumulativas: hereda las de FASE-0.

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-RES-001 | A · `reservas.crearReserva` + `repo.reservas.crearSiHayStock` |
| INV-RES-002 | A · `reservas.validarRango` |
| INV-RES-003 | A · `reservas.crearReserva` (antes de stock) |
| INV-RES-004 | A · `reservas.cancelarReserva` |
| INV-ACC-004 | A · `http/permisos.esDuenoOStaff` |
| INV-ACC-005 | A · `http/permisos` (junta sin escritura) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | reservas (serial) | `src/domain/reservas.js`, `src/server/repo/reservas.js`, `src/server/repo/migrations/002-reservas.sql`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `src/ui/pantallas/material.js`, `src/ui/pantallas/mis-reservas.js`, `src/ui/router.js`, `tests/unit/domain/reservas.test.js`, `tests/integration/reservas.test.js`, `tests/e2e/fase-1.spec.js` | `src/domain/acceso.js` | FASE-0 |

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-002-01 | `POST /api/crearReserva` · `crearReserva(material, desde, hasta)` | reserva con stock por día |
| API-002-02 | `POST /api/listarMisReservas` · `listarMisReservas()` | solo las del actor |
| API-002-03 | `POST /api/cancelarReserva` · `cancelarReserva(reservaId)` | `Cancelada` si hoy < desde |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/material` | `src/ui/pantallas/material.js` | UC-002 | WIREFRAMES §SCR-002 | añade formulario Material, Desde, Hasta |
| `#/mis-reservas` | `src/ui/pantallas/mis-reservas.js` | UC-004, UC-005 | WIREFRAMES §SCR-003 | tabla y botón Cancelar |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `campoFecha` (`src/ui/campos.js`) | `#/material` | entrada `Dia` `YYYY-MM-DD` con etiqueta y error asociado |

## Verificación

```bash
npx vitest run tests/unit/domain/reservas.test.js --maxWorkers=2     # UNIT-T-005..008 verdes
npx vitest run tests/integration/reservas.test.js --maxWorkers=2     # AC-002-*, AC-004-*, AC-005-* verdes
npm test                                                             # toda la suite verde
npx playwright test tests/e2e/fase-1.spec.js                         # C-002-*, C-004-*, C-005-*
```

## Demo

> Desde un checkout limpio: `npm ci` · cuentas lucia, pablo (socio), marta (encargado) · material «Casco» 2 unidades · `npx vite --host 127.0.0.1 --port 5173`.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | Lucia reserva «Casco» 2026-12-10 a 2026-12-12 | aparece en Mis reservas como Reservada | AC-002-01, AC-004-01 · N-003, N-004 |
| 2 | Pablo reserva «Casco» las mismas fechas | reserva creada; queda 0 libres | AC-002-02, AC-004-02 · N-003 |
| 3 | Lucia reserva otro «Casco» con otro rango solapado | `E_NO_STOCK` con su literal | AC-002-03 · N-003 |
| 4 | Lucia repite 2026-12-10 a 2026-12-12 | error de reserva duplicada | AC-002-05 · N-003 |
| 5 | Lucia pone `hasta` antes de `desde` | error de rango de fechas | AC-002-06 · N-003 |
| 6 | Pablo abre Mis reservas | solo la suya | AC-004-05 · N-004 |
| 7 | Lucia cancela su reserva (hoy < desde) | Cancelada; la unidad vuelve a estar libre | AC-005-01, AC-005-02 · N-004 |
| 8 | Lucia pulsa Cancelar otra vez | aviso de ya cancelada, sin cambios | AC-005-04 · N-004 |
| 9 | Pablo intenta cancelar la reserva de lucia (petición directa) | `E_FORBIDDEN_DATA`, sigue Reservada | AC-005-05 · N-011 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-002, UC-004, UC-005 | Bloqueo por multa (AC-002-08 → FASE-3), cancelar lo ya entregado (AC-005-08 → FASE-2) |
| Reloj inyectable en reservas | Comprobaciones de fechas pasadas (Q-spec-auditor-002: se deja como está) |
