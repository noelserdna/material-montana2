# Audit Baseline — material-montana2

> **Audit:** AUDIT-v1.2 (modo actualización: REQ-F-020) · **Date:** 2026-11-16 · **Specs:** `spec/` (requisitos v1.2) · **Docs audited:** 55 · **Mode:** sequential (cambio acotado a 1 REQ; ver highlights) · **Cycles:** 1 Discovery → 2 Fix (solo inequívocos)
> **Gate:** **PASS** — 0 P0, 0 P1, 3 P2 abiertos (SIL-007 resuelto por Q-004)
> **Findings:** 6 · P0 0 · P1 0 · P2 3 · P3 3 · excluded by baseline 1 (SIL-002) · 2 cerrados en esta pasada (SIL-007, CON-007)
> **Delta vs AUDIT-v1.1:** new 3 (CON-007, SIL-008, SEM-002) · persistent 3 (SIL-005, SIL-006, INC-002) · regression 0 · resolved 2 (SIL-007, CON-007)
> **Decisiones del cliente aplicadas, no impugnadas:** Q-001 (cancelar/entregar solo desde Reservada), Q-002 (sin más comprobaciones de fechas), Q-003 (historial = solo lo entregado), Q-004 (la espera sigue contando desde el 5.º fallo), RN-001, RN-002.

## Gate detail

| Check | Result |
|---|---|
| 3C Completeness (SC01–SC05) | PASS 5/5 (REQ-F-020 → UC-019, API-002-07, BDD-UC-019, INV-ACC-004/005) |
| 3C Correctness (SR01–SR05) | PASS (permisos de API-002-07 = PERMISSIONS-MATRIX; INV y estados de UC-019 resuelven) |
| 3C Coherence (SH01–SH05) | WARN: SH04 → INC-002; SH05 → SEM-002 |
| Traceability coverage | 100% (20/20 funcionales) PASS |
| Clarification density | 0/doc PASS (sin marcadores NC) |
| Cross-reference validity | ~99% PASS (`design/OPERATION-MAPPING.md` ya existe) |

## Findings P0–P2

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
| CON-007 | CAT-05 | `use-cases/UC-019:30`, `contracts/API-reservas.md:22` | RESUELTO: UC-019 y API-002-07 citaban INV-ACC-004 (que deja ver a la junta) para un rechazo a la junta | citan INV-ACC-005 y PERMISSIONS-MATRIX (solo encargado) |
| SIL-008 | CAT-03 | `use-cases/UC-019:35` | No se dice el orden de la lista de reservas | pendiente: sin arreglo inequívoco (decisión de contrato) |
| SEM-002 | CAT-04 | `use-cases/UC-019:9` | Estado `Review` mientras el resto de UC aprobados dicen `Approved` | lo cambia la aprobación, no la auditoría |
| CON-005 | CAT-05 | `tests/BDD-UC-018.md`, `nfr/SECURITY.md` | RESUELTO: etiquetas AC2..AC4 desfasadas una posición; NFR citaba AC-018-01..04 | retag y AC-018-01..05 |
| CON-006 | CAT-05 | `DERIVED-SPECS.md` | RESUELTO: UC-001..016 y `E_SESSION_EXPIRED` ligado a REQ-NF-001 | UC-001..018; `E_SESSION_EXPIRED` sale de esa fila |

## Coverage

| Document | P0 | P1 | P2 | P3 | IDs |
|---|---|---|---|---|---|
| contracts/API-reservas.md · domain/03-VALUE-OBJECTS.md · UC-011 | 0 | 0 | 2 | 0 | SIL-005, SIL-006 |
| TRACEABILITY-MATRIX.md | 0 | 0 | 1 | 0 | INC-002 |
| UC-019 · API-reservas.md (API-002-07) | 0 | 0 | 0 | 3 | CON-007, SIL-008, SEM-002 |
| tests/BDD-UC-018.md · nfr/SECURITY.md · DERIVED-SPECS.md | 0 | 0 | 0 | 2 | CON-005, CON-006 |
| Limpios (REQ-F-017..020) | 0 | 0 | 0 | 0 | BDD-UC-019, PERMISSIONS-MATRIX, 03-VALUE-OBJECTS, TRACEABILITY-MATRIX, UC-017, UC-018, BDD-UC-017, API-acceso, VALUE-REGISTRY, 02-ENTITIES, 01-GLOSSARY |

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
| SIL-007 | Intento fallido durante el bloqueo | Q-004 aplicada: INV-ACC-006 y UC-001 dicen «desde el 5.º fallo»; BDD-UC-001 AC-001-12 | v1.2 |
| CON-007 | UC-019/API-002-07 citaban INV-ACC-004 para rechazar a la junta | UC-019, API-reservas | v1.2 |
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

## Fix cycle (AUDIT-v1.2)
- Fixed 2 (CON-007; SIL-007 aplicando Q-004: INV-ACC-006, UC-001, BDD-UC-001) · skipped 2 P3 (SIL-008, SEM-002) · open P2 3 (SIL-005, SIL-006, INC-002; sin cambios, no tocados por REQ-F-020)
- Documents modified: 5 (UC-019, API-reservas, INV, UC-001, BDD-UC-001) · breaking changes: 0

## History

| Audit | Date | Docs | Total | P0 | P1 | P2 | P3 | Gate | Mode | Report chars |
|---|---|---|---|---|---|---|---|---|---|---|
| AUDIT-v1.0 | 2026-11-16 | 53 | 17 | 0 | 1 | 7 | 9 | CONDITIONAL | sequential | 9240 |
| AUDIT-v1.1 | 2026-11-16 | 53 | 11 | 0 | 0 | 4 | 7 | PASS | sequential | 7100 |
| AUDIT-v1.2 | 2026-11-16 | 55 | 6 | 0 | 0 | 3 | 3 | PASS | sequential | 7653 |
