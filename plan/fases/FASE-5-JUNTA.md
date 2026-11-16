# FASE 5: La junta consulta y cada dato es de su dueño

> **Estado:** Implementable
> **Incremento:** La junta ve el historial de cualquier socio sin poder cambiar nada, y un socio solo ve lo suyo
> **Requisitos:** REQ-F-014, REQ-F-015, REQ-F-016
> **Escenarios:** AC-014-01, AC-014-02, AC-014-03, AC-014-04, AC-014-05, AC-014-06, AC-015-01, AC-015-02, AC-015-03, AC-015-04, AC-016-01, AC-016-02, AC-016-03, AC-016-04, AC-016-05
> **Workflows:** —
> **Necesidades:** N-010, N-011
> **Dependencias:** Fase 3

---

## Objetivo

Permitir que la **junta** (y el encargado) consulte el historial de préstamos y multas de un socio, que la junta no vea ningún botón de escritura ni pueda ejecutar escrituras, y que un socio no lea datos de otro. En el historial «préstamos» son solo las reservas que de verdad se entregaron (Q-spec-auditor-003).

## Criterios de Éxito

### UC-014 — Historial del socio
- [ ] El historial muestra los préstamos entregados y las multas del socio pedido (AC-014-01, REQ-F-014)
- [ ] Un socio sin multas muestra «Sin multas» (AC-014-02)
- [ ] Reservas no entregadas (Reservada, Cancelada) no salen como préstamos (AC-014-06, Q-spec-auditor-003)
- [ ] Un socio no pide el historial de otro: `E_FORBIDDEN_DATA` (AC-014-03, INV-ACC-004)
- [ ] Sin sesión no hay datos; abrir otra vez no cambia nada (AC-014-04, AC-014-05)

### UC-015 — La junta solo consulta
- [ ] La pantalla de la junta no tiene botones de escritura (AC-015-01, REQ-F-015 AC1, SEC-T-005)
- [ ] Marcar multa pagada o reservar con rol junta da `E_FORBIDDEN_ROLE` y no cambia nada (AC-015-02, AC-015-03)
- [ ] Repetir el intento da el mismo resultado (AC-015-04)
- [ ] Las 7 operaciones de escritura rechazan el rol junta (UNIT-T-003, INV-ACC-005, REQ-F-015)

### UC-016 — Datos solo para su dueño
- [ ] Lucia no lee las reservas ni el historial de pablo, y al revés (AC-016-01, AC-016-02, REQ-F-016)
- [ ] El encargado ve las multas de todos (AC-016-03, API-004-01)
- [ ] Sin sesión no hay datos; repetir da lo mismo (AC-016-04, AC-016-05)

### Transversal
- [ ] Un socio pidiendo un dato ajeno recibe el mismo resultado exista o no (AUTHZ-002, SEC-T-002, REQ-F-016)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-014-historial-del-socio.md` | flujo, AC-014-01..06 | historial |
| `use-cases/UC-015-junta-solo-consulta.md` | flujo, AC-015-01..04 | rol junta |
| `use-cases/UC-016-datos-solo-del-dueno.md` | flujo, AC-016-01..05 | dueño |

### Dominio y contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/05-INVARIANTS.md` | INV-ACC-004, INV-ACC-005 | acceso por fila y rol |
| `contracts/API-multas.md`, `PERMISSIONS-MATRIX.md` | API-004-03; matriz completa | firma y permisos |
| `audits/SECURITY-AUDIT-BASELINE.md` | AUTHZ-001, AUTHZ-002 | denegar por defecto |

### Diseño y plan de tests

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/OPERATION-MAPPING.md` | API-004-03 | ruta |
| `ux/WIREFRAMES.md` | SCR-008, SCR-009 | pantallas |
| `test/TEST-PLAN.md` | UNIT-T-003, UNIT-T-004, SEC-T-002, SEC-T-005, GAP-004 | permisos |
| `test/TEST-MATRIX-UC-014.md`, `UC-015.md`, `UC-016.md` | filas | integración |

## Invariantes Aplicables

> Acumulativas: hereda las de FASE-0..FASE-4.

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-ACC-004 | A · `http/permisos.esDuenoOStaff` en `verHistorialSocio`, `listarMisReservas`, `listarMultas` |
| INV-ACC-005 | A · `http/permisos` (matriz de roles; junta solo lectura) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | historial y permisos (serial) | `src/domain/multas.js`, `src/server/repo/multas.js`, `src/server/http/multas.js`, `src/server/http/permisos.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `src/ui/pantallas/historial.js`, `src/ui/pantallas/sin-permiso.js`, `src/ui/cabecera.js`, `src/ui/router.js`, `tests/unit/http/permisos.test.js`, `tests/integration/historial-permisos.test.js`, `tests/e2e/fase-5.spec.js` | `src/domain/acceso.js` | FASE-3 |

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-004-03 | `POST /api/verHistorialSocio` · `verHistorialSocio(correo)` | `{prestamos, multas}`; prestamos = entregadas |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/historial` | `src/ui/pantallas/historial.js` | UC-014, UC-016 | WIREFRAMES §SCR-008 | campo «Socio», tablas Préstamos y Multas |
| `#/sin-permiso` | `src/ui/pantallas/sin-permiso.js` | UC-014, UC-016 | WIREFRAMES §SCR-009 | estado de error |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `cabecera` (`src/ui/cabecera.js`) | todas | oculta botones de escritura al rol junta |

## Verificación

```bash
npx vitest run tests/unit/http/permisos.test.js --maxWorkers=2          # UNIT-T-003, UNIT-T-004
npx vitest run tests/integration/historial-permisos.test.js --maxWorkers=2   # AC-014-*, AC-015-*, AC-016-*
npm test                                                                # suite completa verde
npx playwright test tests/e2e/fase-5.spec.js                            # C-014-*, C-015-*, C-016-*
```

## Demo

> Desde un checkout limpio: igual que FASE-3, con ana (junta), lucia con una reserva entregada y devuelta tarde (multa pendiente) y otra solo reservada.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | Ana abre Historial del socio y escribe `lucia@club.es` | préstamo entregado y multa Pendiente | AC-014-01 · N-010 |
| 2 | Ana mira un socio sin multas | «Sin multas» | AC-014-02 · N-010 |
| 3 | Ana comprueba que la reserva solo reservada no sale | solo lo entregado | AC-014-06 · N-010 |
| 4 | Ana abre Material y Multas | sin botones de reservar ni de pagar | AC-015-01 · N-010 |
| 5 | Ana intenta marcar una multa pagada (petición directa) | `E_FORBIDDEN_ROLE`, sigue Pendiente | AC-015-02 · N-010 |
| 6 | Lucia abre su Mis reservas | solo las suyas | AC-016-01 · N-011 |
| 7 | Lucia pide el historial de pablo | `E_FORBIDDEN_DATA` y pantalla Sin permiso | AC-014-03, AC-016-01 · N-011 |
| 8 | Marta abre Multas | multas de todos | AC-016-03 · N-011 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-014, UC-015, UC-016 | Exportar el historial (no pedido) |
| Revisión de la matriz de permisos de las 14 operaciones | Roles nuevos |
