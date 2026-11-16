# FASE 4: Ver el día y dar de baja unidades rotas

> **Estado:** Implementable
> **Incremento:** El encargado ve lo prestado y lo que vuelve cada día y da de baja unidades rotas; el catálogo deja de ofrecer lo roto y Hoy señala los días con más reservas que unidades
> **Requisitos:** REQ-F-012, REQ-F-013
> **Escenarios:** AC-012-01, AC-012-02, AC-012-03, AC-012-04, AC-012-05, AC-012-06, AC-012-07, AC-012-08, AC-013-01, AC-013-02, AC-013-03, AC-013-04, AC-013-05, AC-013-06
> **Workflows:** —
> **Necesidades:** N-008, N-009
> **Dependencias:** Fase 2

---

## Objetivo

Permitir que el **encargado** consulte en Hoy qué material está prestado y qué vuelve un día cualquiera, y dé de baja unidades rotas. La baja no toca reservas (RN-002, decisión del cliente: la reserva se mantiene y el encargado avisa a quien corresponda), por eso Hoy marca los días en que las reservas activas superan las unidades.

## Criterios de Éxito

### UC-012 — Dar de baja unidades rotas
- [ ] Una baja normal resta de las unidades activas (AC-012-01, REQ-F-012)
- [ ] Dar de baja más de las activas da `E_UNITS_INSUFFICIENT` y no cambia nada (AC-012-02, INV-INV-002)
- [ ] Con 0 activas no se puede reservar el material (AC-012-03, REQ-F-012 AC3)
- [ ] Cero unidades da `E_UNITS_MIN`; material inexistente `E_MATERIAL_NOT_FOUND`; un socio no da de baja (AC-012-04, AC-012-05, AC-012-06)
- [ ] Una reserva futura sigue `Reservada`: el sistema no la cancela ni envía avisos (AC-012-07, RN-002)
- [ ] Repetir la baja resta otra vez, como acción nueva (AC-012-08, D-T-008)

### UC-013 — Ver lo prestado y las devoluciones de cada día
- [ ] Hoy lista lo entregado en curso con material y correo (AC-013-01, REQ-F-013)
- [ ] Un día con devoluciones previstas las muestra (AC-013-02)
- [ ] Un día sin nada muestra el texto de vacío (AC-013-03)
- [ ] Un socio no ve Hoy (`E_FORBIDDEN_ROLE`) y sin sesión no hay datos (AC-013-04, AC-013-05)
- [ ] Abrir otra vez no cambia nada (AC-013-06)

### Transversal
- [ ] Hoy marca el día y material con más reservas activas que unidades activas, para que el encargado avise a mano (RN-002, AC-012-07, REQ-F-013)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-012-dar-de-baja-unidades.md` | flujo, AC-012-01..08 | baja |
| `use-cases/UC-013-ver-hoy.md` | flujo, AC-013-01..06 | Hoy |

### Dominio y contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/05-INVARIANTS.md` | INV-INV-001, INV-INV-002, INV-RES-005 | reglas |
| `contracts/API-inventario.md`, `API-reservas.md` | API-003-03, API-002-06 | firmas |
| `CLARIFICATIONS.md` | RN-002 | baja sin tocar reservas |

### Diseño y plan de tests

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/TECHNICAL-DESIGN.md` | §6 «Baja de unidades» | señalar días excedidos |
| `design/OPERATION-MAPPING.md` | API-002-06, API-003-03 | rutas |
| `ux/WIREFRAMES.md` | SCR-006 (baja), SCR-007 | pantallas |
| `test/TEST-MATRIX-UC-012.md`, `UC-013.md`; `test/TEST-PLAN.md` | UNIT-T-013, D-T-009 | integración |

## Invariantes Aplicables

> Acumulativas: hereda las de FASE-0..FASE-3.

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-INV-001 | A · `inventario.darDeBajaUnidades` |
| INV-INV-002 | A · `inventario.darDeBajaUnidades` (≤ activas; no toca reservas) |
| INV-RES-001 | A · `reservas.crearReserva` lee unidades activas |
| INV-RES-005 | A · `reservas.verHoy` (clasifica por estado) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | hoy y bajas (serial) | `src/domain/inventario.js`, `src/domain/reservas.js`, `src/server/repo/inventario.js`, `src/server/repo/reservas.js`, `src/server/http/inventario.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `src/ui/pantallas/hoy.js`, `src/ui/pantallas/inventario.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `tests/unit/domain/inventario.test.js`, `tests/integration/hoy-bajas.test.js`, `tests/e2e/fase-4.spec.js` | `src/domain/acceso.js` | FASE-2 |

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-003-03 | `POST /api/darDeBajaUnidades` · `darDeBajaUnidades(material, unidades)` | resta activas; reservas intactas |
| API-002-06 | `POST /api/verHoy` · `verHoy(dia?)` | `{prestado, vuelven}` del día + días excedidos |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/hoy` | `src/ui/pantallas/hoy.js` | UC-013 | WIREFRAMES §SCR-007 | tablas Prestado y Vuelven, selector «Día» |
| `#/inventario` | `src/ui/pantallas/inventario.js` | UC-012 | WIREFRAMES §SCR-006 | añade formulario Dar de baja |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `avisoExceso` (`src/ui/aviso-exceso.js`) | `#/hoy` | marca días con más reservas que unidades |

## Verificación

```bash
npx vitest run tests/unit/domain/inventario.test.js --maxWorkers=2   # UNIT-T-013 bajas
npx vitest run tests/integration/hoy-bajas.test.js --maxWorkers=2    # AC-012-*, AC-013-*
npm test                                                             # suite completa verde
npx playwright test tests/e2e/fase-4.spec.js                         # C-012-*, C-013-*
```

## Demo

> Desde un checkout limpio: igual que FASE-2, con «Casco» 3 unidades, dos reservas para 2026-12-10..12 (una entregada) y el reloj en 2026-12-10.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | Marta abre Hoy el 2026-12-10 | Casco prestado con el correo de lucia | AC-013-01 · N-009 |
| 2 | Marta abre Hoy el 2026-12-12 | la devolución prevista de lucia | AC-013-02 · N-009 |
| 3 | Marta abre Hoy en un día vacío | texto de vacío | AC-013-03 · N-009 |
| 4 | Marta da de baja 1 «Casco» | quedan 2 activas | AC-012-01 · N-008 |
| 5 | Marta da de baja 5 «Casco» | error de unidades insuficientes; sigue 2 | AC-012-02 · N-008 |
| 6 | Marta da de baja 2 «Casco» y abre Material | Casco con 0 y no se puede reservar | AC-012-03 · N-008 |
| 7 | Lucia abre Mis reservas | la reserva futura sigue Reservada | AC-012-07 · N-008 |
| 8 | Marta abre Hoy el día de esa reserva | día marcado con más reservas que unidades | AC-012-07 · N-009 |
| 9 | Lucia abre `#/hoy` | sin permiso | AC-013-04 · N-011 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-012, UC-013 | Avisos automáticos o correo al afectado (RN-002: lo hace el encargado) |
| Aviso visual de días excedidos en Hoy | Reubicar o cancelar reservas por la baja (no pedido) |
