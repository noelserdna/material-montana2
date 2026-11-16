# Audit Baseline — material-montana2

> **Audit:** AUDIT-v1.0 · **Date:** 2026-11-16 · **Specs:** `spec/` v1.0 (2026-11-16) · **Docs audited:** 53 · **Mode:** sequential · **Cycles:** 1 Discovery → 2 Fix (solo inequívocos)
> **Gate:** **CONDITIONAL PASS** — 0 P0, 1 P1 abierto (SIL-001, pregunta al cliente), 2 P2 abiertos
> **Findings:** 17 · P0 0 · P1 1 · P2 7 · P3 9 · batched 0 · cross-validated 0 · excluded by baseline 0 (8 corregidos en esta pasada; 9 abiertos)
> **Delta vs none (first audit):** new 17 · persistent 0 · regression 0 · resolved since 8
> **Top categories:** CAT-05 (4), CAT-03 (5), CAT-07 (1)
> **Decisiones del cliente aplicadas, no impugnadas:** RN-001 (el encargado prepara las cuentas, sin registro) y RN-002 (baja permitida; el encargado avisa). Sin hallazgos contra ellas.

## Gate detail

| Check | Result |
|---|---|
| 3C Completeness (SC01–SC05) | PASS 5/5 (`adr/` es condicional; ver ADR-001) |
| 3C Correctness (SR01–SR05) | PASS tras el Fix (SR02/SR05 → CON-001, CON-003 corregidos) |
| 3C Coherence (SH01–SH05) | WARN: SH01 → SEM-001; SH03 → INC-001, INC-002 |
| Spec defect density | 0.02 (P0+P1)/doc (target < 2) PASS |
| Traceability coverage | 100% (21/21) PASS |
| Orphan rate | 0% PASS |
| Clarification density | 0/doc PASS (`CLARIFICATIONS-PENDING.md` vacío, sin marcadores NC) |
| Audit pass rate | 94% (3 docs con P1: UC-005, UC-006, 04-STATES) PASS |
| Cross-reference validity | ~98% (`design/OPERATION-MAPPING.md` aún no existe) WARN |

## Findings P0–P2

### SIL-001: Cancelar una Entregada / entregar una Devuelta sin definir — P1 · CAT-03 · new
- **Where:** `use-cases/UC-005-cancelar-reserva.md:35` · `use-cases/UC-006-entregar-reserva.md:35` · `domain/04-STATES.md:12-17`
- **What:** INV-RES-004 solo exige «hoy < `desde`»; UC-006 no tiene guarda de fecha, así que una reserva Entregada antes de `desde` cumple ese guard. Entregar una Devuelta tampoco tiene error (E_RES_CANCELLED solo cubre Cancelada).
- **Why:** Cancelar una Entregada liberaría una unidad que sigue en manos del socio y rompería INV-RES-001. Ni SM-001 ni el catálogo (`03-VALUE-OBJECTS.md:14-32`) dicen qué pasa.
- **Fix:** Pregunta Q-spec-auditor-001. Recomendada: cancelar y entregar solo desde Reservada; resto → error de estado nuevo, sin cambios.

### SIL-002: Fechas pasadas, `devueltaEl` y «hoy» sin definir — P2 · CAT-03 · new
- **Where:** `domain/05-INVARIANTS.md:13` · `use-cases/UC-007-marcar-devolucion.md:15` · `use-cases/UC-013-ver-hoy.md:15`
- **What:** No se dice si `desde` puede ser anterior a hoy, si `devueltaEl` puede ser anterior a `desde` o futuro, ni de dónde sale «hoy» (zona horaria).
- **Why:** INV-RES-004, RN-004 y el cálculo de multa dependen de «hoy» y de `devueltaEl`; una fecha absurda genera multas o reservas sin sentido.
- **Fix:** Pregunta Q-spec-auditor-002. Recomendada: sin restricciones nuevas; hoy = fecha del servidor.

### AMB-001: «Préstamos» del historial sin definir — P2 · CAT-01 · new
- **Where:** `use-cases/UC-014-historial-del-socio.md:16` · `domain/01-GLOSSARY.md` (término ausente)
- **What:** `prestamos` no dice qué estados cuentan (REQ-F-014 «loan history»); «Préstamo» no está en el glosario.
- **Why:** Con dos lecturas razonables (solo Entregada/Devuelta, o todas) el test AC-014-01 no se puede cerrar sin suponer.
- **Fix:** Pregunta Q-spec-auditor-003. Recomendada: solo Entregada y Devuelta; añadir «Préstamo» al glosario con esa definición.

### CON-001: Matriz sin API-004-03 para el socio — P2 · CAT-05 · new — RESOLVED (`PERMISSIONS-MATRIX.md`)
- **Where:** `contracts/PERMISSIONS-MATRIX.md:5` · `contracts/API-multas.md:18`
- **What:** El contrato dice «el socio solo el suyo» y UC-014 E1 trata al socio como actor; la matriz no daba API-004-03 al socio.
- **Fix aplicado:** socio ← API-004-03, regla de fila «reservas e historial» propios.

### CON-003: Matriz daba API-002-03 al encargado; contrato y UC-005 solo al socio — P2 · CAT-05 · new — RESOLVED (`PERMISSIONS-MATRIX.md`)
- **Where:** `contracts/PERMISSIONS-MATRIX.md:6` · `contracts/API-reservas.md:19` · `use-cases/UC-005:7`
- **What:** `API-002-01..06` incluía cancelar. Regla de la minoría (3 docs vs 1): se corrige la matriz.
- **Fix aplicado:** encargado ← `API-002-01..02, 04..06`. Residuo: que el encargado cancele sus propias reservas queda en la regla Tier 1 pendiente («el encargado también reserva», `DERIVED-SPECS.md:18`).

### IMP-001: Alta de cuentas sin operación ni nota — P2 · CAT-02 · new — RESOLVED (`API-acceso.md`)
- **Where:** `domain/02-ENTITIES.md:4` · `contracts/API-acceso.md`
- **What:** ENT-001 «creada por el encargado (RN-001)» sin operación en contratos ni matriz.
- **Fix aplicado:** nota en API-acceso: no hay operación de alta; las cuentas existen antes del primer acceso (RN-001).

### SIL-003: Duplicado vs stock sin orden — P2 · CAT-03 · new — RESOLVED (`UC-002`)
- **Where:** `use-cases/UC-002-reservar-material.md:30,46`
- **What:** Con 1 sola unidad, repetir el formulario daba `E_NO_STOCK` en vez de «Ya tienes esa reserva» (REQ-F-002 AC5).
- **Fix aplicado:** el duplicado se comprueba antes que el stock (paso 4, E7).

### SIL-004: Día por defecto de «Hoy» — P2 · CAT-03 · new — RESOLVED (`UC-013`)
- **Where:** `use-cases/UC-013-ver-hoy.md:27` · `contracts/API-reservas.md:21`
- **What:** `dia?` opcional sin valor por defecto.
- **Fix aplicado:** sin elegir día, el día es hoy.

## Findings P3

| ID | Cat | Where | What | Fix |
|---|---|---|---|---|
| CON-002 | CAT-05 | `domain/03-VALUE-OBJECTS.md:19` | E_FORBIDDEN_DATA no listaba API-002-03 | RESUELTO: añadido |
| CON-004 | CAT-05 | `domain/05-INVARIANTS.md:17` | Validación con `dias*2` en vez de `FINE_RATE_EUR` | RESUELTO |
| INV-001 | CAT-07 | `domain/05-INVARIANTS.md:9-20` | Columna UCs incompleta (ACC-003/004/005, RES-005, FIN-003, INV-002) | RESUELTO |
| SIL-005 | CAT-03 | `contracts/API-reservas.md:17` | Junta en Mis reservas sin error; E_FORBIDDEN_DATA sin entrada | Definir E_FORBIDDEN_ROLE o lista vacía |
| SIL-006 | CAT-03 | `domain/03-VALUE-OBJECTS.md:9` · `UC-011:28` | Unidades no enteras, nombre vacío o mayúsculas sin regla | Añadir validación a VO-006 y UC-011 |
| SEM-001 | CAT-04 | `domain/01-GLOSSARY.md` | Faltan «Préstamo», «Sesión»; «activa» vale para reserva y unidad | Añadir al glosario (ligado a AMB-001) |
| ADR-001 | CAT-09 | `spec/` (sin `adr/`) | Almacén de servidor y hash/sesión sin ADR | Lo cubre `sdd-tech-designer` |
| INC-001 | CAT-06 | `contracts/API-*.md:6` | `design/OPERATION-MAPPING.md` aún no existe | Se crea en la fase de diseño |
| INC-002 | CAT-06 | `TRACEABILITY-MATRIX.md:11-18` | REQ-F-006..008, 010 sin WF-001 (WF-001:7 los cita) | Regenerar con `sdd trace matrix` |

## Coverage

| Document | P0 | P1 | P2 | P3 | IDs |
|---|---|---|---|---|---|
| use-cases/UC-005-cancelar-reserva.md | 0 | 1 | 0 | 0 | SIL-001 |
| use-cases/UC-006-entregar-reserva.md | 0 | 1 | 0 | 0 | SIL-001 |
| domain/04-STATES.md | 0 | 1 | 0 | 0 | SIL-001 |
| use-cases/UC-002 · UC-007 · UC-013 · UC-014 | 0 | 0 | 4 | 0 | SIL-002, SIL-003, SIL-004, AMB-001 |
| contracts/PERMISSIONS-MATRIX.md · API-acceso.md | 0 | 0 | 3 | 0 | CON-001, CON-003, IMP-001 |
| domain/03-VALUE-OBJECTS.md · 05-INVARIANTS.md · 01-GLOSSARY.md | 0 | 0 | 0 | 5 | CON-002, CON-004, INV-001, SIL-006, SEM-001 |
| contracts/API-reservas.md · TRACEABILITY-MATRIX.md · API-* | 0 | 0 | 0 | 3 | SIL-005, INC-001, INC-002 |
| Clean (≈ 35) | 0 | 0 | 0 | 0 | resto de UC, BDD-UC-001..016, WF-001, nfr/*, CLARIFICATIONS*, DERIVED-SPECS, VALUE-REGISTRY, README, 02-ENTITIES |

## Baseline

### Accepted
| ID | Short description | Reason | Audit |
|---|---|---|---|

### Won't fix
| ID | Short description | Reason | Audit |
|---|---|---|---|

### Deferred
| ID | Short description | Reason | Audit | Re-evaluate on |
|---|---|---|---|---|

### Resolved (last 2 audits)
| ID | Short description | Resolved in | Audit |
|---|---|---|---|
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

## History

| Audit | Date | Docs | Total | P0 | P1 | P2 | P3 | Gate | Mode | Report chars |
|---|---|---|---|---|---|---|---|---|---|---|
| AUDIT-v1.0 | 2026-11-16 | 53 | 17 | 0 | 1 | 7 | 9 | CONDITIONAL | sequential | 9240 |
