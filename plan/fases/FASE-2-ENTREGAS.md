# FASE 2: Entregar y recibir el material

> **Estado:** Implementable
> **Incremento:** El encargado entrega lo reservado y marca la devolución, y ve en Entregas y devoluciones lo que reservaron los socios desde otro dispositivo
> **Requisitos:** REQ-F-006, REQ-F-007, REQ-F-020, REQ-NF-001
> **Escenarios:** AC-006-01, AC-006-02, AC-006-03, AC-006-04, AC-006-05, AC-006-06, AC-006-07, AC-007-01, AC-007-02, AC-007-03, AC-007-04, AC-007-05, AC-007-06, AC-005-08, AC-019-01, AC-019-02, AC-019-03, AC-019-04, AC-019-05, AC-019-06, AC-019-07, AC-019-08, REQ-NF-001 AC1
> **Workflows:** WF-001
> **Necesidades:** N-001, N-005
> **Dependencias:** Fase 1

---

## Objetivo

Permitir que el **encargado** liste las reservas de todos los socios (filtro «Por entregar» por defecto, «Entregadas» o «Todas»), entregue una reserva y marque su devolución con fecha (`devueltaEl`), con la unidad libre otra vez. La devolución aún no calcula multa: eso lo añade FASE-3 dentro de la misma operación. Cierra los pasos 2 y 3 de WF-001.

## Criterios de Éxito

### UC-019 — Ver las reservas de todos los socios
- [ ] Entregas y devoluciones lista «material» «correo» de todos; por defecto «Por entregar» (AC-019-01, REQ-F-020 AC1)
- [ ] El filtro «Entregadas» o «Todas» cambia la lista sin modificar datos (AC-019-02, AC-019-03, AC-019-08)
- [ ] Sin coincidencias se ve «No hay reservas» (AC-019-04, REQ-F-020 AC4)
- [ ] Socio y junta reciben `E_FORBIDDEN_ROLE` sin datos; sin sesión `E_AUTH_REQUIRED` (AC-019-05..07, INV-ACC-005)

### UC-006 — Entregar lo reservado
- [ ] Entregar una `Reservada` la pasa a `Entregada` (AC-006-01, AC-006-02, REQ-F-006)
- [ ] Entregar otra vez da el aviso `O_ALREADY_DELIVERED`, sin cambios (AC-006-03)
- [ ] Una reserva cancelada o ya devuelta no se entrega: `E_RES_CANCELLED` / `E_DELIVER_STATE`, sin cambios (AC-006-04, AC-006-07, Q-spec-auditor-001)
- [ ] Un socio no entrega (`E_FORBIDDEN_ROLE`); reserva inexistente da `E_RES_NOT_FOUND` (AC-006-05, AC-006-06)

### UC-007 — Marcar la devolución
- [ ] Devolver una `Entregada` en plazo la pasa a `Devuelta` con su `devueltaEl` (AC-007-01, REQ-F-007)
- [ ] Devolver otra vez da `O_ALREADY_RETURNED` sin cambios (AC-007-02, INV-RES-005)
- [ ] Devolver lo nunca entregado da `E_NOT_DELIVERED` y no cambia (AC-007-03)
- [ ] La unidad devuelta vuelve a estar libre para nuevas reservas (AC-007-04, INV-RES-001)
- [ ] Un socio no devuelve; reserva inexistente da `E_RES_NOT_FOUND` (AC-007-05, AC-007-06)
- [ ] No se añaden comprobaciones de fechas raras en `devueltaEl` (Q-spec-auditor-002, AC-007-01)

### UC-005 (cancelar tras entregar)
- [ ] Cancelar una reserva ya entregada da `E_CANCEL_STATE` y no cambia nada (AC-005-08, Q-spec-auditor-001)

### Transversal
- [ ] Lo reservado por un socio en una sesión lo ve el encargado en la petición siguiente de otra sesión (REQ-NF-001 AC1, INT-T-001)
- [ ] Entregar y devolver se anotan en el registro de seguridad con actor, operación e id (INCIDENT-001, REQ-F-006, REQ-F-007)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-019-ver-reservas-de-todos.md` | flujo, AC-019-01..08 | listar |
| `use-cases/UC-006-entregar-reserva.md` | flujo, AC-006-01..07 | entregar |
| `use-cases/UC-007-marcar-devolucion.md` | flujo, AC-007-01..06 | devolver |
| `use-cases/UC-005-cancelar-reserva.md` | AC-005-08 | cancelar lo entregado |
| `workflows/WF-001-ciclo-de-reserva.md` | pasos 2–3 | ciclo |

### Dominio y contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/04-STATES.md` | SM-001 | transiciones y repetición idempotente |
| `domain/05-INVARIANTS.md` | INV-RES-005 | solo se entrega lo Reservada |
| `contracts/API-reservas.md` | API-002-07, API-002-04, API-002-05 | firmas |
| `nfr/AVAILABILITY.md`, `nfr/SECURITY.md` | sin escritura parcial, registro | NFR |

### Diseño y plan de tests

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/OPERATION-MAPPING.md` | API-002-07, API-002-04, API-002-05 | rutas RPC |
| `ux/WIREFRAMES.md` | SCR-004 | pantalla |
| `test/TEST-PLAN.md` | UNIT-T-009, INT-T-001, GAP-002, GAP-005 | estados y almacén compartido |
| `test/TEST-MATRIX-UC-006.md`, `UC-007.md`, `UC-019.md` | filas, T01..T10, AC-NF1-01 | integración |

## Invariantes Aplicables

> Acumulativas: hereda las de FASE-0..FASE-1.

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-RES-005 | A · `reservas.entregarReserva`, `reservas.devolverReserva` (máquina SM-001) |
| INV-RES-004 | A · `reservas.cancelarReserva` (solo `Reservada`) |
| INV-ACC-005 | A · `http/permisos` (solo encargado: entregar, devolver y `listarReservas`) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | entregas (serial) | `src/domain/reservas.js`, `src/server/repo/reservas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/server/log-seguridad.js`, `src/shared/textos.js`, `src/ui/pantallas/entregas.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `tests/unit/domain/reservas-estados.test.js`, `tests/unit/log-seguridad.test.js`, `tests/integration/entregas.test.js`, `tests/e2e/fase-2.spec.js` | `src/domain/acceso.js` | FASE-1 |

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-002-07 | `POST /api/listarReservas` · `listarReservas(filtro?)` | todas las reservas según filtro; solo encargado |
| API-002-04 | `POST /api/entregarReserva` · `entregarReserva(reservaId)` | `Reservada` → `Entregada` |
| API-002-05 | `POST /api/devolverReserva` · `devolverReserva(reservaId, devueltaEl)` | `Entregada` → `Devuelta`; sin multa hasta FASE-3 |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/entregas` | `src/ui/pantallas/entregas.js` | UC-019, UC-006, UC-007 | WIREFRAMES §SCR-004 | selector de filtro, tabla de todos los socios con Entregar y campo «Devuelta el» + Devolver, «No hay reservas» si vacía |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `tablaReservas` (`src/ui/tabla-reservas.js`) | `#/entregas`, `#/mis-reservas` | filas con material, correo, fechas y estado |

## Verificación

```bash
npx vitest run tests/unit/domain/reservas-estados.test.js --maxWorkers=2  # UNIT-T-009 verde
npx vitest run tests/integration/entregas.test.js --maxWorkers=2          # AC-006-*, AC-007-*, AC-019-*, AC-005-08, INT-T-001
npm test                                                                  # suite completa verde
npx playwright test tests/e2e/fase-2.spec.js                              # S1, C-006-*, C-007-*, C-019-*, C-NF1
```

## Demo

> Desde un checkout limpio: igual que FASE-1, con la reserva de lucia «Casco» 2026-12-10 a 2026-12-12 sembrada y el reloj en 2026-12-01.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | En el móvil, lucia reserva «Casco»; en el ordenador, marta abre Entregas y devoluciones | marta ve «Casco» y «lucia@club.es» (filtro «Por entregar») | AC-019-01, REQ-NF-001 AC1 · N-001, N-005 |
| 2 | Marta pulsa Entregar | Entregada | AC-006-01 · N-005 |
| 3 | Marta pulsa Entregar otra vez (petición repetida) | aviso de ya entregada | AC-006-03 · N-005 |
| 4 | Lucia intenta cancelar la reserva entregada | error de estado, sigue Entregada | AC-005-08 · N-004 |
| 5 | Marta marca Devuelta el 2026-12-12 | Devuelta | AC-007-01 · N-005 |
| 6 | Lucia reserva otra vez el mismo «Casco» y fechas | se acepta: la unidad volvió | AC-007-04 · N-005 |
| 7 | Marta intenta devolver una reserva nunca entregada | `E_NOT_DELIVERED`, sin cambios | AC-007-03 · N-005 |
| 8 | Lucia (socio) abre `#/entregas` o llama a `entregarReserva` / `listarReservas` | sin permiso | AC-006-05, AC-019-05 · N-011 |
| 9 | Marta elige «Entregadas» y luego «Todas» | solo lo entregado; después todas las reservas de todos | AC-019-02, AC-019-03 · N-005 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-019, UC-006, UC-007 (sin multa), AC-005-08 | Multa por retraso y su bloqueo (FASE-3) |
| Registro de seguridad de entregas y devoluciones | Ver lo prestado de cada día (UC-013 → FASE-4) |
