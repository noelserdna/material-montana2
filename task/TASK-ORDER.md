# Implementation Order

> **Generated:** 2026-11-16
> **Total FASEs:** 7
> **Recommended approach:** Incremental delivery per FASE (plan vertical: cada FASE termina con su demo)
> **Critical path:** TASK-F0-001 → … → TASK-F0-018 → FASE-1 → FASE-2 → FASE-3 → FASE-5 → FASE-6 (FASE-4 cuelga de FASE-2)

## FASE Dependency Graph

```text
FASE-0 ──► FASE-1 ──► FASE-2 ──┬──► FASE-3 ──► FASE-5 ──► FASE-6
                               └──► FASE-4
```

## Recommended Implementation Sequence

### Wave 1: Foundation
**FASE-0** (No dependencies — start here)
- 18 tasks, 3 parallelizable (001 → 003 ∥ 004 ∥ 005)
- Critical path: 13 sequential tasks
- Streams: serial
- Entrega desplegable desde el primer commit: TASK-F0-001 y TASK-F0-002 dejan `npm run build`, `npm start` (sirve lo construido, `PORT`/`HOST`, por defecto `0.0.0.0`) y el README.

### Wave 2: Reservas
**FASE-1** (depends on: FASE-0)
- 8 tasks, 0 parallelizable
- Critical path: 8 sequential tasks
- Streams: serial

### Wave 3: Entregas
**FASE-2** (depends on: FASE-1)
- 8 tasks, 0 parallelizable
- Critical path: 6 sequential tasks
- Streams: serial
- **Bloqueo:** TASK-F2-005 es `[PLAN GAP]` (Q-task-generator-001); bloquea TASK-F2-006, 007 y 008 hasta que el cliente decida y se rehaga el plan de la FASE.

### Wave 4: Multas y Hoy
**FASE-3** (depends on: FASE-2)
- 10 tasks, 0 parallelizable
- Critical path: 9 sequential tasks
- Streams: serial

**FASE-4** (depends on: FASE-2) — puede ir en paralelo con FASE-3 si hay dos personas; sus write-sets de `src/domain/reservas.js`, `rutas.js`, `textos.js` y `router.js` se solapan, por lo que lo seguro es secuencial.
- 7 tasks, 0 parallelizable
- Critical path: 7 sequential tasks
- Streams: serial

### Wave 5: Junta
**FASE-5** (depends on: FASE-3)
- 7 tasks, 0 parallelizable
- Critical path: 7 sequential tasks
- Streams: serial

### Wave 6: Sesión
**FASE-6** (depends on: FASE-5)
- 7 tasks, 0 parallelizable
- Critical path: 7 sequential tasks
- Streams: serial

## Cross-FASE Dependencies

| From (Stream) | To (Stream) | Reason |
|---------------|-------------|--------|
| TASK-F0-006 (base) | TASK-F1-001 (base) | el almacén y su sistema de migraciones |
| TASK-F0-007 (base) | TASK-F1-002 (A) | `exigirSesion` y `exigirRol` |
| TASK-F0-005 (base) | TASK-F1-006 (A) | «hoy» del reloj inyectable para cancelar |
| TASK-F1-002 (A) | TASK-F2-002 (A) | `crearReserva` y `ocupadas` dan las reservas que entregar |
| TASK-F2-003 (A) | TASK-F3-004 (A) | la multa se añade a `devolverReserva` en la misma transacción |
| TASK-F1-002 (A) | TASK-F3-006 (A) | el bloqueo por multa entra en `crearReserva` |
| TASK-F2-002 (A) | TASK-F4-004 (A) | `verHoy` clasifica por estado de la reserva |
| TASK-F3-008 (A) | TASK-F5-006 (A) | `listarMultas` y `marcarMultaPagada` entran en la matriz completa |
| TASK-F0-007 (base) | TASK-F5-001 (base) | se amplía `permisos.js` |
| TASK-F0-010 (A) | TASK-F6-002 (A) | `iniciarSesion` y la fila única de sesión |
| TASK-F5-004 (A) | TASK-F6-003 (A) | la cabecera común donde va «Salir» |

## MVP Strategy

El MVP es FASE-0..FASE-3: entrar, reservar, entregar y devolver con multa. FASE-4 (Hoy y bajas) y FASE-5 (junta) completan el uso diario; FASE-6 cierra la seguridad de sesión (salir y caducar).

## Delivery Checkpoints

| After | Checkpoint | What the customer can do |
|-------|-----------|--------------------------|
| FASE-0 | `fase-0-verified` | el encargado entra, añade material y los socios lo ven; la app arranca con `npm start` |
| FASE-1 | `fase-1-verified` | un socio reserva, ve y cancela |
| FASE-2 | `fase-2-verified` | el encargado entrega y recibe (tras resolver la plan gap) |
| FASE-3 | `fase-3-verified` | multas por retraso, bloqueo y pago |
| FASE-4 | `fase-4-verified` | Hoy y bajas de unidades |
| FASE-5 | `fase-5-verified` | la junta consulta; datos solo del dueño |
| FASE-6 | `fase-6-verified` | Salir y caducidad de sesión |

## Streams

- FASE-0: Streams: serial — base(9) → A(8) → verificación(1)
- FASE-1: Streams: serial — base(1) → A(6) → verificación(1)
- FASE-2: Streams: serial — base(1) → A(6) → verificación(1)
- FASE-3: Streams: serial — base(2) → A(7) → verificación(1)
- FASE-4: Streams: serial — A(6) → verificación(1)
- FASE-5: Streams: serial — base(1) → A(5) → verificación(1)
- FASE-6: Streams: serial — base(1) → A(5) → verificación(1)
