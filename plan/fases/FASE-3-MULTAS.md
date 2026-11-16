# FASE 3: Multas por retraso

> **Estado:** Implementable
> **Incremento:** Devolver tarde genera una multa de 2 € por día, el socio con multa pendiente no puede reservar y el encargado marca la multa como pagada
> **Requisitos:** REQ-F-008, REQ-F-009, REQ-F-010
> **Escenarios:** AC-008-01, AC-008-02, AC-008-03, AC-008-04, AC-008-05, AC-009-01, AC-009-02, AC-009-03, AC-009-04, AC-009-05, AC-002-08, AC-010-01, AC-010-02, AC-010-03, AC-010-04, AC-010-05
> **Workflows:** WF-001
> **Necesidades:** N-006, N-007
> **Dependencias:** Fase 2

---

## Objetivo

Permitir que el **encargado** devuelva con retraso y la devolución cree una multa `Pendiente` en la misma transacción, que el **socio** con multa pendiente reciba `E_FINE_PENDING` al reservar y que el encargado vea la lista de Multas y las marque `Pagada`. Cierra los pasos 4 y 5 de WF-001.

## Criterios de Éxito

### UC-008 — Multa por retraso
- [ ] Multa = (`devueltaEl` − `hasta`) × 2 €: tres días dan 6 € y un día 2 € (AC-008-01, AC-008-02, REQ-F-008)
- [ ] Sin retraso no hay multa (AC-008-03, INV-FIN-001)
- [ ] Devolver otra vez no crea otra multa (AC-008-04, INV-FIN-002)
- [ ] Si falla el almacén, ni devolución ni multa quedan escritas (AC-008-05, `E_STORAGE_UNAVAILABLE`, D-T-002)
- [ ] El 2 € sale solo de `FINE_RATE_EUR` (REQ-C-003, MNT-T-003)

### UC-009 — Sin reservar con multa pendiente
- [ ] Con una multa pendiente, reservar da `E_FINE_PENDING` y no crea nada (AC-009-01, AC-002-08, REQ-F-009)
- [ ] Con varias, pagar una no desbloquea (AC-009-02, INV-FIN-003)
- [ ] Otro socio sin multa sigue reservando; las reservas previas no cambian (AC-009-03, AC-009-04)
- [ ] Pagar a la vez que reserva no deja estado a medias (AC-009-05)

### UC-010 — Marcar la multa como pagada
- [ ] La multa pasa a `Pagada` y el socio vuelve a poder reservar (AC-010-01, AC-010-02, REQ-F-010)
- [ ] Pagar una ya pagada da `O_ALREADY_PAID`; inexistente da `E_FINE_NOT_FOUND` (AC-010-03, AC-010-05)
- [ ] Un socio no marca multas (`E_FORBIDDEN_ROLE`) (AC-010-04, INV-ACC-005)

### Transversal
- [ ] `listarMultas` muestra correo, importe y estado, o «Sin multas» (API-004-01, REQ-F-008)

## Specs a Leer

### Casos de Uso

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `use-cases/UC-008-multar-retraso.md` | flujo, AC-008-01..05 | cálculo |
| `use-cases/UC-009-bloqueo-por-multa.md` | flujo, AC-009-01..05 | bloqueo |
| `use-cases/UC-010-marcar-multa-pagada.md` | flujo, AC-010-01..05 | pago |
| `use-cases/UC-002-reservar-material.md` | AC-002-08 | error por multa |

### Dominio y contratos

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `domain/02-ENTITIES.md`, `04-STATES.md` | ENT-004 Multa, SM-002 | tabla y estados |
| `domain/05-INVARIANTS.md` | INV-FIN-001..003 | reglas |
| `contracts/API-reservas.md`, `API-multas.md` | API-002-05, API-004-01, API-004-02 | firmas |
| `CLARIFICATIONS.md` | RN-004 | días de retraso |

### Diseño y plan de tests

| Documento | Sección / ids | Para |
|-----------|---------------|------|
| `design/OPERATION-MAPPING.md` | API-002-05, API-004-01..02 | rutas |
| `ux/WIREFRAMES.md` | SCR-005 | pantalla Multas |
| `test/TEST-PLAN.md` | UNIT-T-010..012, D-T-002 | cálculo y doble de fallo |
| `test/TEST-MATRIX-UC-008.md`, `UC-009.md`, `UC-010.md` | filas | integración |

## Invariantes Aplicables

> Acumulativas: hereda las de FASE-0..FASE-2.

| ID | Dónde se aplica (bloque · función) |
|----|------------------------------------|
| INV-FIN-001 | A · `multas.calcularMulta` |
| INV-FIN-002 | A · `repo.multas` (único `reserva_id`) y `reservas.devolverReserva` |
| INV-FIN-003 | A · `reservas.crearReserva` (cuenta `Pendiente`) |

## Módulos y Conjuntos de Escritura

| Bloque | Módulo / directorio | Escribe (write-set) | No escribe | Depende de |
|--------|---------------------|---------------------|------------|------------|
| A | multas (serial) | `src/domain/multas.js`, `src/domain/reservas.js`, `src/server/repo/multas.js`, `src/server/repo/reservas.js`, `src/server/repo/migrations/003-multas.sql`, `src/server/http/multas.js`, `src/server/http/reservas.js`, `src/server/http/rutas.js`, `src/shared/textos.js`, `src/ui/pantallas/multas.js`, `src/ui/router.js`, `src/ui/cabecera.js`, `tests/unit/domain/multas.test.js`, `tests/unit/ui/formato.test.js`, `tests/integration/multas.test.js`, `tests/e2e/fase-3.spec.js` | `src/domain/acceso.js` | FASE-2 |

## Contenido Específico

### Cálculo de la multa

`dias = devueltaEl − hasta` (días naturales, REQ-C-003); `dias > 0 → importe = dias × FINE_RATE_EUR`; si no, sin multa. La devolución y la multa se escriben en una transacción.

## Contratos Resultantes

| Contrato | Firma / Ruta | Descripción (≤ 80 chars) |
|----------|--------------|--------------------------|
| API-002-05 | `POST /api/devolverReserva` · `devolverReserva(reservaId, devueltaEl)` | añade `multa?` en la misma transacción |
| API-004-01 | `POST /api/listarMultas` · `listarMultas()` | multas de todos; solo encargado |
| API-004-02 | `POST /api/marcarMultaPagada` · `marcarMultaPagada(multaId)` | `Pendiente` → `Pagada` |

## Entregables de UI

### Páginas / Rutas

| Ruta | Componente | UC | Wireframe | Descripción |
|------|------------|----|-----------|-------------|
| `#/multas` | `src/ui/pantallas/multas.js` | UC-008, UC-010 | WIREFRAMES §SCR-005 | tabla Multas y botón Marcar pagada |
| `#/entregas` | `src/ui/pantallas/entregas.js` | UC-008 | WIREFRAMES §SCR-004 | muestra la multa creada al devolver |

### Componentes Compartidos

| Componente | Usado en | Descripción |
|------------|----------|-------------|
| `importeEuros` (`src/ui/formato.js`) | `#/multas`, `#/entregas` | formato del importe en euros |

## Verificación

```bash
npx vitest run tests/unit/domain/multas.test.js --maxWorkers=2     # UNIT-T-010..012 (BVA hasta−1, hasta, hasta+1, +3)
npx vitest run tests/integration/multas.test.js --maxWorkers=2     # AC-008-*, AC-009-*, AC-010-*, AC-002-08
npm test                                                           # suite completa verde
npx playwright test tests/e2e/fase-3.spec.js                       # S2, C-008-*, C-009-*, C-010-*
```

## Demo

> Desde un checkout limpio: igual que FASE-2, con lucia con «Casco» entregada (`hasta` 2026-12-12) y el reloj en 2026-12-15.

| # | Acción | Resultado esperado | Escenario |
|---|--------|--------------------|-----------|
| 1 | Marta devuelve el Casco con `devueltaEl` 2026-12-15 | Devuelta y multa de 6 € | AC-008-01 · N-006 |
| 2 | Marta abre Multas | fila lucia@club.es, 6 €, Pendiente | AC-008-01 · N-006 |
| 3 | Marta repite la devolución | aviso de ya devuelta, sigue una multa | AC-008-04 · N-006 |
| 4 | Lucia intenta reservar «Casco» | error de multa pendiente | AC-009-01 · N-006 |
| 5 | Pablo (sin multa) reserva «Casco» | se acepta | AC-009-03 · N-006 |
| 6 | Marta pulsa Marcar pagada | multa Pagada | AC-010-01 · N-007 |
| 7 | Lucia reserva «Casco» otra vez | se acepta | AC-010-01 · N-007 |
| 8 | Lucia (socio) intenta marcar una multa por petición directa | `E_FORBIDDEN_ROLE` | AC-010-04 · N-011 |

## Alcance

| Incluye | Excluye |
|---------|---------|
| UC-008, UC-009, UC-010 | Historial de multas por socio (FASE-5) |
| `listarMultas` | Cobro real o notificaciones (fuera del producto) |

## Notas

- UC-002 solo aporta AC-002-08 (error por multa pendiente, REQ-F-009 AC1); por eso el lint cuenta 4 UC (P-SIZE) aunque la FASE tiene 3 journeys.
