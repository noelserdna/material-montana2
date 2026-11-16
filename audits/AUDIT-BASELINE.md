# Audit Baseline — material-montana2

> **Audit:** AUDIT-v1.1 (modo actualización: REQ-F-017, REQ-F-018, REQ-F-019) · **Date:** 2026-11-16 · **Specs:** `spec/` (requisitos v1.1) · **Docs audited:** 53 · **Mode:** sequential (cambio acotado a 3 REQ; ver highlights) · **Cycles:** 1 Discovery → 2 Fix (solo inequívocos)
> **Gate:** **PASS** — 0 P0, 0 P1, 4 P2 abiertos (SIL-007 con pregunta al cliente)
> **Findings:** 11 · P0 0 · P1 0 · P2 4 · P3 7 · excluded by baseline 1 (SIL-002) · 7 corregidos o cerrados en esta pasada
> **Delta vs AUDIT-v1.0:** new 3 (SIL-007, CON-005, CON-006) · persistent 3 (SIL-005, SIL-006, INC-002) · regression 0 · resolved 7 (SIL-001, AMB-001, SEM-001, INC-001, ADR-001, CON-005, CON-006)
> **Decisiones del cliente aplicadas, no impugnadas:** Q-001 (cancelar/entregar solo desde Reservada), Q-002 (sin más comprobaciones de fechas), Q-003 (historial = solo lo entregado), RN-001, RN-002.

## Gate detail

| Check | Result |
|---|---|
| 3C Completeness (SC01–SC05) | PASS 5/5 (REQ-F-017..019 → UC-001/017/018, API-001-01..02, INV-ACC-006/007, BDD) |
| 3C Correctness (SR01–SR05) | PASS (SR03/SR04/SR05 verificados para los INV, estados y permisos nuevos) |
| 3C Coherence (SH01–SH05) | WARN: SH04 → INC-002 (matriz generada sin WF-001) |
| Traceability coverage | 100% (19/19 funcionales) PASS |
| Clarification density | 0/doc PASS (sin marcadores NC) |
| Cross-reference validity | ~99% PASS (`design/OPERATION-MAPPING.md` ya existe) |

## Findings P0–P2

### SIL-007: Intento fallido durante el bloqueo sin definir — P2 · CAT-03 · new
- **Where:** `domain/05-INVARIANTS.md:9` (INV-ACC-006) · `use-cases/UC-001-entrar-con-correo.md:36` · `requirements/REQUIREMENTS.md` REQ-F-017
- **What:** «15 minutos desde el último fallo»: no se dice si un intento erróneo hecho durante el bloqueo cuenta como fallo y reinicia la espera.
- **Why:** Con la lectura literal, el bloqueo por cuenta se podría alargar sin fin a quien conozca el correo; con la otra, no. BDD-UC-001 no lo cubre.
- **Fix:** Pregunta Q-spec-auditor-004. Recomendada: los intentos durante el bloqueo no cuentan ni alargan la espera.

### SIL-005: Junta en Mis reservas sin error definido — P2 (escalado desde P3) · CAT-03 · persistent
- **Where:** `contracts/API-reservas.md:17` — `listarMisReservas` es de socio y encargado; la junta no tiene error ni lista vacía definidos.
- **Fix:** Definir `E_FORBIDDEN_ROLE` o lista vacía (requiere decisión de contrato; sin arreglo inequívoco).

### SIL-006: Unidades no enteras, nombre vacío o mayúsculas sin regla — P2 (escalado desde P3) · CAT-03 · persistent
- **Where:** `domain/03-VALUE-OBJECTS.md:9` · `use-cases/UC-011:28`
- **Fix:** Añadir validación a VO-006 y UC-011 (cambia comportamiento: pendiente de decisión).

### INC-002: Matriz de trazabilidad sin WF-001 en REQ-F-006..008, 010 — P2 (escalado desde P3) · CAT-06 · persistent
- **Where:** `TRACEABILITY-MATRIX.md:11-18` · `workflows/WF-001:7`
- **Fix:** Regenerar con `sdd trace matrix` (el fichero es generado; no se edita a mano).

## Findings P3

| ID | Cat | Where | What | Fix |
|---|---|---|---|---|
| CON-005 | CAT-05 | `tests/BDD-UC-018.md`, `nfr/SECURITY.md` | RESUELTO: etiquetas AC2..AC4 desfasadas una posición; NFR citaba AC-018-01..04 | retag y AC-018-01..05 |
| CON-006 | CAT-05 | `DERIVED-SPECS.md` | RESUELTO: UC-001..016 y `E_SESSION_EXPIRED` ligado a REQ-NF-001 | UC-001..018; `E_SESSION_EXPIRED` sale de esa fila |

## Coverage

| Document | P0 | P1 | P2 | P3 | IDs |
|---|---|---|---|---|---|
| domain/05-INVARIANTS.md · use-cases/UC-001 · tests/BDD-UC-001.md | 0 | 0 | 1 | 0 | SIL-007 |
| contracts/API-reservas.md · domain/03-VALUE-OBJECTS.md · UC-011 | 0 | 0 | 2 | 0 | SIL-005, SIL-006 |
| TRACEABILITY-MATRIX.md | 0 | 0 | 1 | 0 | INC-002 |
| tests/BDD-UC-018.md · nfr/SECURITY.md · DERIVED-SPECS.md | 0 | 0 | 0 | 2 | CON-005, CON-006 |
| Limpios (REQ-F-017..019) | 0 | 0 | 0 | 0 | UC-017, UC-018, BDD-UC-017, API-acceso, VALUE-REGISTRY, 02-ENTITIES, 01-GLOSSARY |

## Baseline

### Accepted
| ID | Short description | Reason | Audit |
|---|---|---|---|

### Won't fix
| ID | Short description | Reason | Audit |
|---|---|---|---|
| SIL-002 | Fechas pasadas, `devueltaEl` y «hoy» sin definir | Q-spec-auditor-002: se deja como está, sin más comprobaciones de fechas | v1.1 |

### Deferred
| ID | Short description | Reason | Audit | Re-evaluate on |
|---|---|---|---|---|

### Resolved (last 2 audits)
| ID | Short description | Resolved in | Audit |
|---|---|---|---|
| SIL-001 | Cancelar una Entregada / entregar una Devuelta | `E_CANCEL_STATE`, `E_DELIVER_STATE` en UC-005, UC-006, 03, 04, 05, API-reservas, BDD-UC-005/006 (Q-001) | v1.1 |
| AMB-001 | «Préstamos» del historial | UC-014, glosario, BDD-UC-014 AC-014-06 (Q-003) | v1.1 |
| SEM-001 | Faltaban «Préstamo» y «Sesión» | 01-GLOSSARY | v1.1 |
| INC-001 | `design/OPERATION-MAPPING.md` no existía | ya existe (tech-designer) | v1.1 |
| ADR-001 | Almacén y sesión sin ADR | `design/ADR-DRAFT-002/003` (tech-designer) | v1.1 |
| CON-005 | Etiquetas REQ-F-019 AC desfasadas en BDD-UC-018; NFR sin AC-018-05 | BDD-UC-018, nfr/SECURITY | v1.1 |
| CON-006 | DERIVED-SPECS con rango UC-001..016 y `E_SESSION_EXPIRED` atribuido a REQ-NF-001 | DERIVED-SPECS | v1.1 |
| CON-001 | Matriz sin API-004-03 para el socio | PERMISSIONS-MATRIX.md | v1.0 |
| CON-002 | E_FORBIDDEN_DATA sin API-002-03 | 03-VALUE-OBJECTS.md | v1.0 |
| CON-003 | Matriz daba API-002-03 al encargado | PERMISSIONS-MATRIX.md | v1.0 |
| CON-004 | `dias*2` en vez de FINE_RATE_EUR | 05-INVARIANTS.md | v1.0 |
| IMP-001 | Alta de cuentas sin nota | API-acceso.md | v1.0 |
| INV-001 | Columna UCs de invariantes incompleta | 05-INVARIANTS.md, UC-012 | v1.0 |
| SIL-003 | Orden duplicado/stock | UC-002 | v1.0 |
| SIL-004 | Día por defecto de Hoy | UC-013 | v1.0 |

## Fix cycle (AUDIT-v1.0)
- Fixed 8 (CON-001..004, IMP-001, INV-001, SIL-003, SIL-004) · skipped 9 (SIL-001, SIL-002, AMB-001: pregunta al cliente; 6 P3: sin arreglo inequívoco o de otra etapa) · open P2 2 · deferred P3 0
- New artifacts: `audits/AUDIT-BASELINE.md`, `.sdd/questions-spec-auditor.json`
- Documents modified: 7 · breaking changes: 0 · commits: 6 · tag: —

## Fix cycle (AUDIT-v1.1)
- Fixed 7 (SIL-001, AMB-001, SEM-001 aplicando Q-001/Q-003; CON-005, CON-006; INC-001 y ADR-001 cerrados por el diseño) · skipped 4 (SIL-007: pregunta al cliente; SIL-005, SIL-006, INC-002: sin arreglo inequívoco o regenerable) · Won't fix 1 (SIL-002, Q-002)
- Mensajes literales de `E_CANCEL_STATE` y `E_DELIVER_STATE`: decisión por defecto (clase R), redactados en el estilo del catálogo.
- Documents modified: 14 · breaking changes: 0

## History

| Audit | Date | Docs | Total | P0 | P1 | P2 | P3 | Gate | Mode | Report chars |
|---|---|---|---|---|---|---|---|---|---|---|
| AUDIT-v1.0 | 2026-11-16 | 53 | 17 | 0 | 1 | 7 | 9 | CONDITIONAL | sequential | 9240 |
| AUDIT-v1.1 | 2026-11-16 | 53 | 11 | 0 | 0 | 4 | 7 | PASS | sequential | 7100 |
