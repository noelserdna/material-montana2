# Test Plan — material-montana2

> Spec v1.0 · audit gate CONDITIONAL · Project type: WEB-APP (sin `ux/`; contratos `Style: operations`)
> Companion files: TEST-MATRIX-UC-001..018.md (18), E2E-SCENARIOS.md. Sin PERF-SCENARIOS.md: la ruta lo omite (ningún REQ-NF habla de velocidad o carga).

## 1. Strategy Summary

| Dimension | Target | Current | Source |
|-----------|--------|---------|--------|
| UC main + exception flows with BDD | 100% | 100% (18/18 UC, 0 excepciones sin AC) | spec/tests/ |
| Invariants with property tests | 100% of INV-* | 0% en spec (no hay PROPERTY-TESTS.md); planificado 18/18 en §7 | domain/05-INVARIANTS.md |
| Contract operations (API-NNN-NN) with contract tests | 100% | 100% planificado (14/14 en matrices) | spec/contracts/ |
| Quantified NFRs with a scenario | 100% | n/a: 0 NFR cuantificados de velocidad; REQ-NF-001/002 con AC | spec/nfr/ |
| Applicable security controls | 100% | 100% planificado (§7 SEC-T) | nfr/SECURITY.md |
| User-facing WF-* with E2E | 100% | 100% (WF-001) | spec/workflows/ |
| REQ-F criteria with a capturing E2E | 100% de los visibles | 67/67 visibles + REQ-NF-001; 5 `No visible` sin captura | requirements/ |

## 2. Test Levels

| Level | Scope | Technique | Framework / runner | Coverage target | Runs on |
|-------|-------|-----------|--------------------|-----------------|---------|
| Unit | INV-*, VO-*, cálculo de días y multa | propiedades (fast-check si el implementer lo añade; si no, tablas de ejemplos) | Vitest | 90% líneas del dominio | every commit |
| Integration | UC-001..018, operaciones API-001..004, almacén | BDD Given/When/Then + matrices; almacén en memoria con la misma interfaz que el real | Vitest (`maxWorkers: 2`) | 100% flujos principales y de excepción | every commit |
| E2E | WF-001 + pantallas de los 19 REQ-F | Smoke / Critical / Full; captura por criterio, vídeo por WF | Playwright (Chromium) sobre `npx vite` en 127.0.0.1:5173 | 100% WF y criterios visibles | PR / main / nightly |
| Performance | — | — | — | exento: ningún objetivo numérico de velocidad en spec/nfr | — |
| Security | SECURITY.md, REQ-NF-002, INV-ACC-* | checklist ASVS v4 (V2, V4), error guessing | Vitest + Playwright | controles aplicables | release candidate |

## 3. Design Decisions

| ID | Decision | Applies to | Refs |
|----|----------|------------|------|
| D-T-001 | «Hoy» entra por un reloj inyectable; por defecto los tests fijan hoy = 2026-12-01 salvo que el AC diga otro día | Unit, Integration, E2E | INV-RES-004, UC-005, UC-013 |
| D-T-002 | El almacén es una interfaz inyectable con un doble que falla a demanda (`E_STORAGE_UNAVAILABLE`) y comprueba que no queda escritura parcial | Integration | AC-008-05, REQ-NF-001 |
| D-T-003 | Cada test arranca con almacén propio y cuentas sembradas (lucia, pablo = socio; marta = encargado; ana = junta, todos `@club.es`); sin registro: las cuentas las prepara el encargado | todos | RN-001, API-001-01 |
| D-T-004 | Precedencia de errores de UC-002 = orden de pasos del UC: sesión → rol → rango → multa → duplicado → stock; combinaciones rol+rango no se prueban (no definidas) | UC-002 | UC-002 §Main flow, RN-003 |
| D-T-005 | Los textos esperados salen de los literales de VO-010 y del módulo de textos; las etiquetas de campo («Correo», «Contraseña», «Material», «Desde», «Hasta», «Devuelta el», «Unidades», «Día», «Socio») las fija el implementer en ese módulo | E2E, Integration | REQ-C-001, VO-010 |
| D-T-006 | Rechazos de sesión (`E_AUTH_REQUIRED`) se prueban una vez por pantalla en E2E y en cada operación en integración | UC-003/004/013/014/016 | INV-ACC-003 |
| D-T-007 | Operaciones de solo lectura (API-001-01 salvo sesión, 002-02, 002-06, 003-01, 004-01, 004-03) llevan fila `replay` solo si el UC la define (AC «abrir otra vez»); sin fila `race` | matrices | UC-003, 004, 013, 014 |
| D-T-008 | Replay de UC-011 y UC-012 se prueba como acción nueva (suma/resta otra vez), no como idempotente | UC-011, UC-012 | AC-011-04, AC-012-08 |
| D-T-009 | Baja con reservas futuras (RN-002): la reserva sigue `Reservada`, el sistema no avisa ni la cancela; el aviso es manual del encargado y no se prueba | UC-012 | RN-002, AC-012-07 |
| D-T-010 | Casos sin definición en spec llevan `Expected: undefined in spec`, `Refs —` y un hueco en §4; nadie inventa la semántica | matrices | Art. 12 |
| D-T-011 | El reloj inyectable (D-T-001) lleva también hora del día (Instante, VO-007) y avanza a demanda; bloqueo y caducidad se prueban con él, nunca con esperas reales | UC-001, UC-017, UC-018, E2E | INV-ACC-006, INV-ACC-007, `LOGIN_WINDOW_MIN`, `LOCKOUT_MIN`, `SESSION_IDLE_MIN` |
| D-T-012 | Los tests que provocan fallos de acceso usan cuentas propias de ese test (el contador es por cuenta) para no bloquear a las demás; "Salir" (API-001-02) no lleva fila `race`: solo toca la sesión propia | UC-001, UC-017 | INV-ACC-006, INV-ACC-002 |
| D-T-E2E | Cada criterio REQ-F visible: E2E por la ruta del usuario (Iniciar sesión → pantalla), aserta su `Se ve:`, captura `evidencias/FASE-{N}/{AC id}.png`; un vídeo por WF | E2E | § Visual Evidence |

## 4. Gaps

| Gap ID | Type | Spec element | Missing (≤ 100 chars) | Priority |
|--------|------|--------------|------------------------|----------|
| GAP-001 | MISSING-PROPERTY-TEST | INV-ACC-001..005, INV-RES-001..005, INV-FIN-001..003, INV-INV-001..002 | spec/tests/PROPERTY-TESTS.md no existe; los casos van en §7 UNIT-T-001..016 | Medium |
| GAP-002 | INCOMPLETE-BDD | UC-005, UC-006, SM-001 | Cancelar una Entregada / entregar una Devuelta sin resultado definido (SIL-001, Q-spec-auditor-001) | High |
| GAP-003 | INCOMPLETE-BDD | UC-002, UC-005, UC-007, UC-013 | Fechas pasadas, `devueltaEl` < `desde` o futuro y origen de «hoy» sin definir (SIL-002, Q-spec-auditor-002) | Medium |
| GAP-004 | INCOMPLETE-BDD | UC-014 | «Préstamos» del historial: qué estados cuentan (AMB-001, Q-spec-auditor-003) | Medium |
| GAP-005 | MISSING-REPLAY-SPEC | API-002-04, API-002-05, API-004-02, API-003-03 | Dos encargados a la vez (race) sin resultado definido, salvo cancelar (AC-005-04) | Low |
| GAP-006 | INCOMPLETE-BDD | UC-002, UC-004 | El encargado reserva y ve Mis reservas (Tier 1 PENDING REQ) sin AC; no se planifica E2E | Low |
| GAP-007 | INCOMPLETE-BDD | UC-002, SM-001 | Entregada sin devolver tras `hasta` y su efecto en stock (Tier 1 PENDING REQ) sin AC | Low |
| GAP-008 | INCOMPLETE-BDD | UC-002, UC-007 | Material, Desde/Hasta o Devuelta el vacíos: sin AC ni error definido | Medium |
| GAP-009 | INCOMPLETE-BDD | UC-001, 003, 004, 016, API-004-01 | Hallazgos de matrices: login concurrente, catálogo vacío, API-002-02 sin id ajeno (`E_FORBIDDEN_DATA` inalcanzable), junta en API-004-01 | Low |
| GAP-010 | INCOMPLETE-BDD | UC-001, INV-ACC-006 | Instante exacto +15:00, intentos con la cuenta bloqueada, correo sin cuenta y fallos simultáneos sin AC | Medium |
| GAP-011 | INCOMPLETE-BDD | UC-017, UC-018 | Exactamente 30:00 sin actividad; "Salir" con sesión caducada; 2.ª petición tras caducar; almacén caído al salir | Medium |

Los 5 criterios `No visible` (REQ-F-005 AC5, REQ-F-006 AC5, REQ-F-010 AC4, REQ-F-015 AC2 y AC3) y REQ-NF-002 se prueban en integración por API, sin captura: decisión del cliente en requisitos, no es hueco.

## 5. Targets by Use Case

REQ-F-0NN ↔ UC-0NN (uno a uno; REQ-F-017 va en UC-001); los AC ids son los de `spec/tests/BDD-UC-NNN.md`. «Matriz» = `TEST-MATRIX-UC-NNN.md`; «E2E» = filas de `E2E-SCENARIOS.md`; UNIT = §7. Orden de dependencias.

| UC | REQs | Scenarios (AC ids) | Unit | Integration | E2E |
|----|------|--------------------|------|-------------|-----|
| UC-001 | REQ-F-001, REQ-F-017, REQ-NF-002 | AC-001-01..11 | UNIT-T-001, 002, 015, 017 | matriz UC-001 | E2E-WF-001-S0, C-001-* |
| UC-011 | REQ-F-011 | AC-011-01..05 | UNIT-T-013 | matriz UC-011 | C-011-* |
| UC-003 | REQ-F-003 | AC-003-01..04 | UNIT-T-014 | matriz UC-003 | C-003-* |
| UC-002 | REQ-F-002 | AC-002-01..10 | UNIT-T-005, 006, 007 | matriz UC-002 | S1, S2, C-002-* |
| UC-004 | REQ-F-004 | AC-004-01..06 | UNIT-T-004 | matriz UC-004 | C-004-* |
| UC-005 | REQ-F-005 | AC-005-01..07 | UNIT-T-008 | matriz UC-005 | C-005-* |
| UC-006 | REQ-F-006 | AC-006-01..06 | UNIT-T-009 | matriz UC-006 | S1, C-006-* |
| UC-007 | REQ-F-007 | AC-007-01..06 | UNIT-T-009 | matriz UC-007 | S1, S2, C-007-* |
| UC-008 | REQ-F-008, REQ-C-003 | AC-008-01..05 | UNIT-T-010, 011 | matriz UC-008 | S2, C-008-* |
| UC-009 | REQ-F-009 | AC-009-01..05 | UNIT-T-012 | matriz UC-009 | S2, C-009-* |
| UC-010 | REQ-F-010 | AC-010-01..05 | UNIT-T-012 | matriz UC-010 | S2, C-010-* |
| UC-012 | REQ-F-012 | AC-012-01..08 | UNIT-T-013 | matriz UC-012 | C-012-* |
| UC-013 | REQ-F-013 | AC-013-01..06 | UNIT-T-009 | matriz UC-013 | C-013-* |
| UC-014 | REQ-F-014 | AC-014-01..05 | UNIT-T-004 | matriz UC-014 | C-014-* |
| UC-015 | REQ-F-015 | AC-015-01..04 | UNIT-T-003 | matriz UC-015 | C-015-* |
| UC-016 | REQ-F-016 | AC-016-01..05 | UNIT-T-004 | matriz UC-016 | C-016-* |
| UC-017 | REQ-F-018 | AC-017-01..03 | UNIT-T-018 | matriz UC-017 | C-017-* |
| UC-018 | REQ-F-019 | AC-018-01..05 | UNIT-T-018 | matriz UC-018 | C-018-* |

## 6. Traceability REQ → tests

| REQ | UC / WF | Matrix rows | E2E | Perf / Sec |
|-----|---------|-------------|-----|------------|
| REQ-F-001..016, 018, 019 | UC-001..016, 017, 018 (REQ-F-002, 005..008, 010 en WF-001) | TEST-MATRIX-UC-0NN (mismo número) | E2E-WF-001-S0..S2 + C-0NN-* | SEC-T-001..005 |
| REQ-F-017 | UC-001 | TEST-MATRIX-UC-001 T14..T23 | C-001-05..09 | SEC-T-006 |
| REQ-NF-001 | UC-002, UC-006 | AC-NF1-01 en matriz UC-006 | E2E-WF-001-C-NF1 | — |
| REQ-NF-002 | UC-001 | UC-001 AC-001-06 | — (No visible) | SEC-T-001 |
| REQ-C-001 | todos | — | texto de cada captura | SEC-T-006 | límite de intentos y sesión | la contraseña correcta no abre sesión bloqueada; sesión cerrada o caducada no devuelve datos | AC-001-07, AC-017-02, AC-018-01, CWE-307, CWE-613 |
| MNT-T-001 |
| REQ-C-002 | — | — | — | MNT-T-002 |
| REQ-C-003 | UC-008 | UC-008 | — | MNT-T-003 |

## 7. Cross-cutting Test IDs

| ID | Object | Verifies (≤ 120 chars) | Refs |
|----|--------|-------------------------|------|
| UNIT-T-001 | hash de contraseña | stored ≠ texto; dos cuentas iguales difieren | INV-ACC-001, AC-001-06 |
| UNIT-T-002 | sesiones | por correo ≤ 1 tras N accesos | INV-ACC-002 |
| UNIT-T-003 | permisos | escritura con rol junta → `E_FORBIDDEN_ROLE` en las 7 operaciones | INV-ACC-005, PERMISSIONS-MATRIX |
| UNIT-T-004 | acceso a datos | actor = dueño ∨ rol ∈ {encargado, junta}, resto `E_FORBIDDEN_DATA` | INV-ACC-004 |
| UNIT-T-005 | stock por día | ocupadas(d) < activas ∀ d del rango, ambos días inclusive | INV-RES-001, RN-003 |
| UNIT-T-006 | rango | `hasta >= desde` (BVA 0, 1 día) | INV-RES-002 |
| UNIT-T-007 | duplicado | único (socio, material, desde, hasta) entre activas; se comprueba antes del stock | INV-RES-003 |
| UNIT-T-008 | cancelación | solo `hoy < desde` y dueño (BVA hoy = desde−1, desde) | INV-RES-004 |
| UNIT-T-009 | SM-001 | transiciones válidas y clases de inválidas | INV-RES-005 |
| UNIT-T-010 | multa | `dias>0 → dias×FINE_RATE_EUR`; BVA devuelta = hasta−1, hasta, hasta+1, +3 | INV-FIN-001, RN-004 |
| UNIT-T-011 | multa única | una por `reservaId`; devolver otra vez no crea otra | INV-FIN-002 |
| UNIT-T-012 | bloqueo | `count(Pendiente)=0` para reservar; pagar una de dos no desbloquea | INV-FIN-003 |
| UNIT-T-013 | inventario | alta/baja `n≥1`; baja `n≤activas`; unidades como entero | INV-INV-001, INV-INV-002 |
| UNIT-T-014 | catálogo | cuenta solo activas | INV-INV-002 |
| UNIT-T-015 | Correo/Dia | formatos VO-001, VO-003 `YYYY-MM-DD` | VO-001, VO-003 |
| UNIT-T-016 | catálogo de errores | cada `E_*` y `O_*` de VO-010 con su literal exacto | VO-010 |
| UNIT-T-017 | bloqueo de cuenta | 5 fallos en 15 min bloquean 15 min desde el último; acierto pone a cero; por cuenta | INV-ACC-006, AC-001-07..11 |
| UNIT-T-018 | sesión | caduca con `ahora − ultimaActividad > 30`; cada operación renueva; "Salir" la termina | INV-ACC-007, AC-017-01, AC-018-01..04 |
| INT-T-001 | almacén compartido | escritura de una sesión visible en la siguiente petición de otra | REQ-NF-001, AC-NF1-01 |
| INT-T-002 | operaciones sin sesión | las 14 operaciones salvo API-001-01 → `E_AUTH_REQUIRED` sin datos; con sesión caducada → `E_SESSION_EXPIRED` | INV-ACC-003 |
| SEC-T-001 | almacén de cuentas | ningún campo contiene la contraseña en claro | REQ-NF-002, CWE-256 |
| SEC-T-002 | acceso a datos ajenos | lucia no lee de pablo (reservas, historial); socio sin multas ajenas | AC-016-01..04, CWE-639 |
| SEC-T-003 | mensajes de login | `E_AUTH_INVALID` idéntico con correo inexistente y con contraseña errónea | AC-001-03, CWE-204 |
| SEC-T-004 | cancelar ajeno | reserva de otro socio sigue Reservada | AC-005-05, CWE-639 |
| SEC-T-005 | botones de escritura | pantalla de la junta sin botones de escritura | AC-015-01 |
| MNT-T-001 | textos | todo literal de interfaz sale del módulo de textos, en español | REQ-C-001 |
| MNT-T-002 | stack | `package.json` + `vitest.config` con `maxWorkers: 2` | REQ-C-002 |
| MNT-T-003 | tarifa | `FINE_RATE_EUR` es la única fuente de 2 € | REQ-C-003, VALUE-REGISTRY |

## 8. Regression Policy

| Trigger | Runs |
|---------|------|
| every commit | unit + integración afectada |
| FASE completion | integración completa + E2E Critical (capturas y vídeos) |
| post-deploy | sin `staging_url`: `smoke-deploy` sin escenarios |
| release candidate | suite completa + SEC-T |

## 9. Inputs for sdd-plan-architect

| ID | Requirement (≤ 140 chars) | Needed by | Refs |
|----|----------------------------|-----------|------|
| R-1 | Reloj inyectable en el dominio (`hoy` y hora del día) | D-T-001, D-T-011, UNIT-T-008/010/017/018 | INV-RES-004, INV-ACC-006, INV-ACC-007 |
| R-2 | Almacén tras una interfaz con doble en memoria y doble que falla | D-T-002, D-T-003 | REQ-NF-001 |
| R-3 | Operaciones del dominio sin dependencia de Vite; tests en `tests/unit`, `tests/integration`, `tests/e2e` | todos | AGENTS.md |
| R-4 | Módulo único de textos con los literales de VO-010 y las etiquetas de campo | D-T-005, MNT-T-001 | REQ-C-001 |
| R-5 | Semilla de cuentas por script, sin pantalla de registro | D-T-003 | RN-001 |
| R-6 | Ayudante `captura(page, testInfo, id, { ve })` y vídeo activado en E2E | D-T-E2E | § Visual Evidence |
| R-7 | Orden vertical sugerido: UC-001+011+003 → UC-002+004+005 → UC-006+007+008+009+010 → UC-012+013+014+015+016 → UC-017+018 | §5 | — |
| R-8 | Reloj controlable también en el navegador (E2E de bloqueo y caducidad) y botón "Salir" en cada pantalla con sesión | C-001-05..09, C-017-*, C-018-* | D-T-011, REQ-F-018 |

## 10. Metrics

| Metric | Value |
|--------|-------|
| Matrices / cases | 18 / 240 (22 replay, 15 race) |
| E2E scenarios Smoke / Critical / Full | 3 / 72 / 9 |
| PERF scenarios | 0 |
| Gaps | 11 |
| Chars: this file / test/ total | ~15 200 / ~102 000 bytes (sobre el límite de 65 000 por 18 matrices en lugar de ~7) |
