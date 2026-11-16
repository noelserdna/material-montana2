# Test Matrix: UC-018 — La sesión se cierra sola tras 30 minutos sin actividad

> Refs: UC-018, BDD-UC-018 (AC-018-01..05), REQ-F-019, INV-ACC-003, INV-ACC-007, `SESSION_IDLE_MIN`
> Techniques: EP, BVA, state transition, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| `ahora − ultimaActividad` | minutos (Instante, VO-007) | 0–29 min: vigente | > 30 min: caducada | `expand: BVA(30)`: 29:59, 30:00, 30:01 |
| operación con sesión | API-* | cualquiera de las 13 con sesión (muestra: Material, Mis reservas, Hoy) | — | — |
| rol | enum | socio · encargado · junta | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived. The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | última actividad de lucia 10:00 · abre Material 10:31 | `E_SESSION_EXPIRED` · "Tu sesión ha caducado" en Iniciar sesión · sin datos · sesión terminada | error | AC-018-01 |
| T02 | última actividad de pablo 09:00 · abre Material 09:45 | `E_SESSION_EXPIRED` · mismo mensaje | error | AC-018-05 |
| T03 | lucia 10:00 · abre Material 10:29 | ok · "Hola, lucia@club.es" · `ultimaActividad`=10:29 | happy | AC-018-02 |
| T04 | lucia activa 10:00, 10:20, 10:40 · abre 10:55 | ok · "Hola, lucia@club.es" · cada operación renueva | state | AC-018-03 |
| T05 | sesión caducada · lucia entra con `Cumbre2026` | ok · "Hola, lucia@club.es" · sesión nueva | state | AC-018-04 |
| T06 | `expand: BVA(30)` 29:59 y 30:01 sin actividad | 29:59 ok · 30:01 `E_SESSION_EXPIRED` | boundary | AC-018-01, AC-018-02 |
| T07 | exactamente 30:00 sin actividad | ok (UC-018 §main.3: «≤ 30 min»); el AC no lo cubre | boundary | — |
| T08 | expand: enum(marta=encargado, ana=junta) caducada | `E_SESSION_EXPIRED` · mismo mensaje | derived | AC-018-01, INV-ACC-007 |
| T09 | caducada · expand: enum(API-002-02, 002-06, 003-01, 004-03) | `E_SESSION_EXPIRED` · sin datos en cada una | derived | AC-018-01, INV-ACC-003 |
| T10 | T01 repetido tras caducar (sesión ya terminada) | `E_AUTH_REQUIRED` o `E_SESSION_EXPIRED`: undefined in spec · sin datos | replay | — |
| T11 | operación de escritura (reservar) con sesión caducada | `E_SESSION_EXPIRED` · sin escritura · sin reserva creada | error | AC-018-01 |
| T12 | caducada · "Salir" | undefined in spec ("Tu sesión ha caducado" o "Iniciar sesión") | boundary | — |
| T13 | dos operaciones simultáneas de una sesión a 29:59 | ambas ok y actividad renovada; a 30:01 ambas `E_SESSION_EXPIRED` | race | AC-018-02, AC-018-01 |

## State Transitions

| SM | From | Event / guard | To | Postcondition (≤ 80 chars) | Cases |
|----|------|---------------|----|-----------------------------|-------|
| sesión | abierta | operación, ≤ 30 min | abierta | `ultimaActividad` = ahora | T03, T04 |
| sesión | abierta | operación, > 30 min | terminada | sin datos; "Tu sesión ha caducado" | T01, T02 |
| sesión | terminada | acceso con contraseña | abierta | sesión nueva | T05 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F01 | Instante exacto de 30:00: UC dice «≤ 30 min», ningún AC lo fija | Añadir AC de límite |
| F02 | Sesión ya terminada tras caducar: ¿`E_AUTH_REQUIRED` o `E_SESSION_EXPIRED` en la 2.ª petición? "Salir" con sesión caducada sin regla | Definir en UC-018 / UC-017 |
