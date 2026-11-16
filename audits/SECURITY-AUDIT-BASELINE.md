# Security Audit: material-montana2

> Date: 2026-11-16 · Spec version: 1.0 · Security docs analysed: 14 · Total docs: 53 · OWASP ASVS 4.0.3
> Decisiones del cliente aplicadas, no impugnadas: RN-001 (el encargado prepara las cuentas; sin registro) y RN-002 (la baja de unidades mantiene la reserva y el encargado avisa). Ningún hallazgo va contra ellas.

## Security Posture Scorecard

| # | Dimension | Score | Grade | Findings (C/H/M/L) |
|---|---|---|---|---|
| D1 | Authentication Completeness | 65 | C | 0/2/1/0 |
| D2 | Authorization | 90 | A | 0/0/2/0 |
| D3 | Data Protection | 85 | B | 0/1/0/0 |
| D4 | Input Validation | 95 | A | 0/0/1/0 |
| D5 | Cryptographic Rigor | 85 | B | 0/1/0/0 |
| D6 | Incident Readiness | 95 | A | 0/0/1/0 |
| D7 | Regulatory Compliance | 95 | A | 0/0/1/0 |
| D8 | Security Test Coverage | 98 | A | 0/0/0/1 |
| D9 | Threat Model Coverage | 98 | A | 0/0/0/1 |
| D10 | Security Decision Documentation | 98 | A | 0/0/0/1 |
| **GLOBAL** | Weighted average | **88** | **B** | **0/4/6/3** |

ASVS chapters assessed: V1, V2, V3, V4, V5, V6, V8 con elemento aplicable; V7, V12, V13 sin cobertura → `owasp_coverage` 70%

## Summary by Category

| Category | Total | Crítico (P0) | Alto (P1) | Medio (P2) | Bajo (P3) |
|---|---|---|---|---|---|
| SEC-CAT-01 Threats without model | 1 | 0 | 0 | 0 | 1 |
| SEC-CAT-02 Incomplete authentication | 3 | 0 | 2 | 1 | 0 |
| SEC-CAT-03 Insufficient authorization | 2 | 0 | 0 | 2 | 0 |
| SEC-CAT-04 Unprotected data | 1 | 0 | 1 | 0 | 0 |
| SEC-CAT-05 Missing input validation | 1 | 0 | 0 | 1 | 0 |
| SEC-CAT-06 Weak/incomplete crypto | 1 | 0 | 1 | 0 | 0 |
| SEC-CAT-07 Incomplete incident response | 1 | 0 | 0 | 1 | 0 |
| SEC-CAT-08 Incomplete regulatory compliance | 1 | 0 | 0 | 1 | 0 |
| SEC-CAT-09 Missing security tests | 1 | 0 | 0 | 0 | 1 |
| SEC-CAT-10 Security decision without ADR | 1 | 0 | 0 | 0 | 1 |
| **TOTAL** | 13 | 0 | 4 | 6 | 3 |

## Baseline Delta

Compared against: N/A (first audit) — new 13 · persistent 0 · regression 0 · resolved since last run 0 · excluded by baseline 0

## Threat Surface

| Surface | Elements | Covered | Uncovered |
|---|---|---|---|
| Operations / endpoints | 16 (API-001-01, API-002-01..06, API-003-01..03, API-004-01..03) | 16 con rol en la matriz; sesión exigida por INV-ACC-003 | sin límite de intentos en API-001-01 |
| Sensitive fields | correo, hash, multas e historial por socio | hash (INV-ACC-001), propiedad (INV-ACC-004) | tránsito, registros, retención |
| External integrations | 0 | — | — |
| Trust boundaries | navegador→servidor; servidor→almacén | rol y dueño en servidor | no se declaran ni se modelan (THR-001) |

## Traceability

| Chain | Total | Complete | Incomplete |
|---|---|---|---|
| Security REQ / SEC row → INV or ADR | 2 REQ + 2 SEC | 4 | 0 (ningún ADR de seguridad; ver SADR-001) |
| Security REQ / SEC row → UC → BDD | 2 REQ + 2 SEC | 4 | 0 |
| Security ADR → runbook | 0 | — | — |

## Findings by Category

### SEC-CAT-01: Threats without model

#### THR-001: Sin modelo de amenazas ni fronteras de confianza

| Field | Value |
|---|---|
| **Severity** | Bajo (P3) |
| **Status** | new |
| **Category** | SEC-CAT-01: Threats without model |
| **OWASP ASVS** | V1.1.2 (4.0.3) |
| **CWE** | CWE-1008 |
| **Where** | `spec/nfr/SECURITY.md` (2 filas) |
| **Threat** | Un atacante con sesión de socio puede probar operaciones de escritura o de otros roles porque la frontera cliente→servidor no está declarada |
| **What** | `SECURITY.md` solo recoge hash y acceso a datos; no declara fronteras ni actores hostiles |
| **Fix** | Declarar las fronteras (navegador→servidor, servidor→almacén) y que toda comprobación de rol y dueño se hace en el servidor |
| **Evidence** | `nfr/SECURITY.md`; `contracts/PERMISSIONS-MATRIX.md`; INV-ACC-003/004 (se dice «API», no que el cliente no es de fiar) |
| **Existing mitigation** | partial: INV-ACC-003..005 se aplican en API |

### SEC-CAT-02: Incomplete authentication

#### AUTH-001: Sin freno a los intentos fallidos de acceso

| Field | Value |
|---|---|
| **Severity** | Alto (P1) |
| **Status** | new |
| **Category** | SEC-CAT-02: Incomplete authentication |
| **OWASP ASVS** | V2.2.1 (4.0.3) |
| **CWE** | CWE-307 |
| **Where** | `spec/contracts/API-acceso.md` API-001-01; `spec/use-cases/UC-001-entrar-con-correo.md` E1/E2 |
| **Threat** | Un atacante anónimo puede adivinar contraseñas de socios (los correos son conocibles) porque no hay límite de intentos y entra a historiales y multas |
| **What** | UC-001 define el error pero no un número de fallos ni espera; `SECURITY.md` no lo cubre |
| **Fix** | Especificar tras cuántos fallos y durante cuánto se frena un correo, y qué ve la persona (pregunta Q-security-auditor-001) |
| **Evidence** | UC-001 expone la operación sin sesión; `nfr/SECURITY.md` SEC-hash sin bloqueo |
| **Existing mitigation** | none |

#### AUTH-002: Ciclo de vida de la sesión sin definir

| Field | Value |
|---|---|
| **Severity** | Alto (P1) |
| **Status** | new |
| **Category** | SEC-CAT-02: Incomplete authentication |
| **OWASP ASVS** | V3.3.1, V3.3.2 (4.0.3) |
| **CWE** | CWE-613 |
| **Where** | `spec/domain/05-INVARIANTS.md` INV-ACC-002; `spec/domain/02-ENTITIES.md` ENT-005 |
| **Threat** | Quien use un ordenador compartido del club tras un socio puede seguir con su sesión porque no hay cierre de sesión ni caducidad |
| **What** | Solo se define «una sesión por cuenta»; no hay cerrar sesión, caducidad por inactividad ni identificador no adivinable |
| **Fix** | Especificar cierre de sesión (operación y UC), caducidad y qué ocurre al volver a entrar desde otro dispositivo (¿sustituye a la anterior?) |
| **Evidence** | INV-ACC-002; ningún API ni UC de cierre; UC-001 abre sesión desde cualquier dispositivo (REQ-NF-001) |
| **Existing mitigation** | partial: una sola sesión por cuenta |

#### AUTH-003: Sin cambio ni restablecimiento de contraseña, y enumeración de cuentas

| Field | Value |
|---|---|
| **Severity** | Medio (P2) |
| **Status** | new |
| **Category** | SEC-CAT-02: Incomplete authentication |
| **OWASP ASVS** | V2.1.5, V2.5.1, V3.2.1 (4.0.3) |
| **CWE** | CWE-640 |
| **Where** | `spec/contracts/API-acceso.md` (nota RN-001); `spec/CLARIFICATIONS.md` RN-001 |
| **Threat** | El encargado, que prepara las cuentas fuera de la web, conoce o fija la contraseña inicial del socio y este no puede cambiarla; ante una pérdida no hay vía definida |
| **What** | RN-001 se respeta (sin registro), pero no dice cómo el socio sustituye la contraseña inicial ni cómo se recupera. El mismo mensaje en E1/E2 evita enumerar cuentas, aunque la igualdad de respuesta (tiempo) no se exige |
| **Fix** | Especificar el cambio de contraseña del socio y el procedimiento de recuperación a cargo del encargado; fijar que existe/no existe el correo no se distingue |
| **Evidence** | `API-acceso.md` (única operación); UC-001 precondición 1 |
| **Existing mitigation** | partial: mensaje único `E_AUTH_INVALID` |

### SEC-CAT-03: Insufficient authorization

#### AUTHZ-001: Matriz sin denegación por defecto

| Field | Value |
|---|---|
| **Severity** | Medio (P2) |
| **Status** | new |
| **Category** | SEC-CAT-03: Insufficient authorization |
| **OWASP ASVS** | V4.1.1, V4.1.5 (4.0.3) |
| **CWE** | CWE-862 |
| **Where** | `spec/contracts/PERMISSIONS-MATRIX.md` |
| **Threat** | Una operación futura o no listada (p. ej. API-004-01 para junta) puede quedar accesible porque la matriz no dice que lo no listado se deniega |
| **What** | La matriz lista permisos pero no declara denegar el resto ni qué error se devuelve (`E_FORBIDDEN_ROLE`) |
| **Fix** | Declarar deny-by-default y el error para cualquier operación fuera de la fila del rol |
| **Evidence** | `PERMISSIONS-MATRIX.md`; INV-ACC-005 solo cubre a la junta |
| **Existing mitigation** | partial: INV-ACC-005 y errores por operación |

#### AUTHZ-002: Existencia de reservas ajenas distinguible por el error

| Field | Value |
|---|---|
| **Severity** | Medio (P2) |
| **Status** | new |
| **Category** | SEC-CAT-03: Insufficient authorization |
| **OWASP ASVS** | V4.2.1 (4.0.3) |
| **CWE** | CWE-639 |
| **Where** | `spec/contracts/API-reservas.md` API-002-03; `spec/domain/02-ENTITIES.md` ENT-003 (`id: number`) |
| **Threat** | Un socio puede recorrer `reservaId` numéricos y distinguir `E_FORBIDDEN_DATA` (existe, ajena) de `E_RES_NOT_FOUND` (no existe) porque los identificadores son correlativos |
| **What** | No se especifica el orden de las comprobaciones ni que ambos casos sean indistinguibles para un socio |
| **Fix** | Especificar la respuesta para un socio ante reserva ajena o inexistente (misma o distinta) |
| **Evidence** | API-002-03 errores 03; INV-RES-004 |
| **Existing mitigation** | partial: se rechaza la acción ajena |

### SEC-CAT-04: Unprotected data

#### DATA-001: Sin requisitos de protección en tránsito ni de registros

| Field | Value |
|---|---|
| **Severity** | Alto (P1) |
| **Status** | new |
| **Category** | SEC-CAT-04: Unprotected data |
| **OWASP ASVS** | V8.3.4, V9.1.1 (4.0.3) |
| **CWE** | CWE-319 |
| **Where** | `spec/nfr/SECURITY.md` |
| **Threat** | Un atacante en la red del club puede leer contraseñas y sesiones porque no se exige canal cifrado; los registros podrían guardar contraseñas o datos de multas |
| **What** | Ninguna regla sobre transporte, sobre qué no se escribe en registros ni sobre cifrado del almacén |
| **Fix** | Especificar que el acceso solo se hace por canal cifrado, que contraseñas y sesiones no aparecen en registros ni mensajes de error, y la protección del almacén de cuentas |
| **Evidence** | `nfr/SECURITY.md`; UC-001 envía la contraseña; REQ-NF-001 comparte datos entre dispositivos |
| **Existing mitigation** | none |

### SEC-CAT-05: Missing input validation

#### INPUT-001: Entradas sin esquema ni límites

| Field | Value |
|---|---|
| **Severity** | Medio (P2) |
| **Status** | new |
| **Category** | SEC-CAT-05: Missing input validation |
| **OWASP ASVS** | V5.1.3, V5.1.4 (4.0.3) |
| **CWE** | CWE-20 |
| **Where** | `spec/domain/03-VALUE-OBJECTS.md` VO-001, VO-003, `contrasena`; API-002/003 `material` |
| **Threat** | Un usuario puede enviar `contrasena` o `material` de longitud enorme, fechas mal formadas o `unidades` no enteras porque no hay esquema |
| **What** | VO-001 y VO-003 dan formato en comentario, sin error `E_*` por formato inválido; `contrasena` y `material` no tienen límites |
| **Fix** | Especificar longitud máxima de contraseña y de material, validez de `Dia` (fecha real) y `Unidades` entero, y el error devuelto |
| **Evidence** | VO-001/003/006; API-001-01 `contrasena: string`; catálogo VO-010 sin error de formato |
| **Existing mitigation** | partial: `E_DATE_RANGE`, `E_UNITS_MIN` |

### SEC-CAT-06: Weak/incomplete crypto

#### CRYPTO-001: Hash «con sal» sin algoritmo de contraseñas ni coste

| Field | Value |
|---|---|
| **Severity** | Alto (P1) |
| **Status** | new |
| **Category** | SEC-CAT-06: Weak/incomplete crypto |
| **OWASP ASVS** | V2.4.1, V2.4.2 (4.0.3) |
| **CWE** | CWE-916 |
| **Where** | `spec/nfr/SECURITY.md`; `spec/domain/05-INVARIANTS.md` INV-ACC-001; REQ-NF-002 |
| **Threat** | Quien obtenga el almacén de cuentas puede recuperar contraseñas deprisa porque «hash con sal» admite un resumen rápido |
| **What** | La verificación (AC-001-06) solo comprueba «≠ texto» y «difiere entre cuentas»; no exige función lenta ni coste mínimo |
| **Fix** | Especificar la clase de función exigida (derivación de claves de contraseña, coste ajustable) y su coste mínimo, y cómo se verifica |
| **Evidence** | INV-ACC-001; `tests/BDD-UC-001.md` AC-001-06; ENT-001 `hash` |
| **Existing mitigation** | partial: sal por cuenta |

### SEC-CAT-07: Incomplete incident response

#### INCIDENT-001: Sin registro de seguridad ni de acciones del encargado

| Field | Value |
|---|---|
| **Severity** | Medio (P2) |
| **Status** | new |
| **Category** | SEC-CAT-07: Incomplete incident response |
| **OWASP ASVS** | V7.1.3, V7.2.1 (4.0.3) |
| **CWE** | CWE-778 |
| **Where** | `spec/nfr/SECURITY.md`; UC-010, UC-012 |
| **Threat** | El encargado (o una cuenta suya robada) puede marcar multas pagadas o dar de baja unidades sin dejar rastro porque no hay registro de auditoría |
| **What** | No se registran accesos fallidos, rechazos de permisos ni escrituras privilegiadas, ni su retención |
| **Fix** | Especificar qué eventos se registran (quién, qué, cuándo), su retención y que no se pueden editar desde la web |
| **Evidence** | API-004-02 y API-003-03 son escrituras sin rastro; `nfr/SECURITY.md` |
| **Existing mitigation** | none |

### SEC-CAT-08: Incomplete regulatory compliance

#### COMPLY-001: Datos personales de los socios sin retención ni derechos

| Field | Value |
|---|---|
| **Severity** | Medio (P2) |
| **Status** | new |
| **Category** | SEC-CAT-08: Incomplete regulatory compliance |
| **OWASP ASVS** | V8.3.1 (4.0.3) |
| **CWE** | CWE-359 |
| **Where** | `spec/domain/02-ENTITIES.md` ENT-001/003/004 |
| **Threat** | Los datos de un socio que deja el club (correo, historial, multas) se conservan sin límite porque no hay retención ni baja de cuenta |
| **What** | RGPD arts. 5.1.e, 15 y 17: no hay plazo de conservación ni UC de acceso o supresión |
| **Fix** | Decidir y especificar plazo de conservación y qué se hace con la cuenta e historial de un socio que se va (decisión del cliente) |
| **Evidence** | ENT-001..004; UC-014 consulta historial sin límite temporal |
| **Existing mitigation** | none |

### SEC-CAT-09: Missing security tests

#### STEST-001: Faltan pruebas de «sin sesión» en las escrituras

| Field | Value |
|---|---|
| **Severity** | Bajo (P3) |
| **Status** | new |
| **Category** | SEC-CAT-09: Missing security tests |
| **OWASP ASVS** | V4.1.3 (4.0.3) |
| **CWE** | CWE-306 |
| **Where** | `spec/tests/BDD-UC-005..012` |
| **Threat** | Una escritura (cancelar, entregar, devolver, pagar, baja) podría quedar sin comprobar sesión porque ningún escenario lo prueba |
| **What** | `E_AUTH_REQUIRED` solo se prueba en UC-003, 004, 013, 014 y 016, aunque INV-ACC-003 cubre toda operación |
| **Fix** | Un escenario de «sin sesión → `E_AUTH_REQUIRED`, estado intacto» por operación de escritura, o uno parametrizado; y de bloqueo/caducidad cuando se decida AUTH-001/002 |
| **Evidence** | INV-ACC-003; grep de BDD |
| **Existing mitigation** | partial: AC-016-04 genérico |

### SEC-CAT-10: Security decision without ADR

#### SADR-001: Decisiones de autenticación y almacenamiento de credenciales sin ADR

| Field | Value |
|---|---|
| **Severity** | Bajo (P3) |
| **Status** | new |
| **Category** | SEC-CAT-10: Security decision without ADR |
| **OWASP ASVS** | V1.2.1 (4.0.3) |
| **CWE** | CWE-1059 |
| **Where** | `spec/adr/` (no existe) |
| **Threat** | Sin decisión escrita, el diseño puede elegir un mecanismo de sesión o de hash débil sin que se detecte |
| **What** | Ni la forma de la sesión ni la función de hash ni el cierre de sesión tienen ADR; `design/` aún no existe |
| **Fix** | Registrar en el diseño las decisiones de sesión, hash de contraseñas y canal, con sus consecuencias |
| **Evidence** | `spec/` sin `adr/`; AUTH-002, CRYPTO-001 |
| **Existing mitigation** | none |

## Remediation

Findings are fixed through `/sdd-req-change` (one CR per finding or group; the CR's `Refs:` cites the finding IDs). Order: P0 before implementation → P1 → P2 backlog → P3 continuous improvement. Ningún hallazgo se ha pasado aún a req-change; AUTH-001/AUTH-002 esperan la respuesta a Q-security-auditor-001.
- P0: ninguno.
- P1: AUTH-001, AUTH-002, DATA-001, CRYPTO-001.

## Baseline

| ID | Title | Status | Decided by / date | Re-evaluate on | Reason |
|---|---|---|---|---|---|
