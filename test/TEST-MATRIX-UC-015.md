# Test Matrix: UC-015 — La junta solo consulta

> Refs: UC-015, API-002-01, API-002-03..05, API-003-02..03, API-004-02, BDD-UC-015 (AC-015-01..04), INV-ACC-005, PERMISSIONS-MATRIX
> Techniques: EP, BVA, decision table, replay · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| actor | rol | — (ninguna escritura admite junta) | junta (ana) | — |
| operación de escritura | API-id | — | 002-01, 002-03, 002-04, 002-05, 003-02, 003-03, 004-02 | — |
| estado previo | store | reserva, stock, multa 6 € Pendiente de lucia | — | — |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | ana abre Historial del socio | "Historial del socio" · ningún botón reservar, cancelar, entregar, devolver, dar de baja ni pagar | happy | AC-015-01 |
| T02 | ana · crearReserva "Casco" (API-002-01) | `E_FORBIDDEN_ROLE` · "No tienes permiso para hacer esto" · sin reserva | error | AC-015-03, INV-ACC-005 |
| T03 | ana · cancelarReserva de lucia (API-002-03) | `E_FORBIDDEN_ROLE` · mismo literal · reserva sigue `Reservada` | error | INV-ACC-005, PERMISSIONS-MATRIX (derived) |
| T04 | ana · entregarReserva (API-002-04) | `E_FORBIDDEN_ROLE` · reserva sigue `Reservada` | error | INV-ACC-005, PERMISSIONS-MATRIX (derived) |
| T05 | ana · devolverReserva con devueltaEl (API-002-05) | `E_FORBIDDEN_ROLE` · reserva sigue `Entregada`, sin multa | error | INV-ACC-005, PERMISSIONS-MATRIX (derived) |
| T06 | ana · anadirMaterial (API-003-02) | `E_FORBIDDEN_ROLE` · unidades activas intactas | error | INV-ACC-005, PERMISSIONS-MATRIX (derived) |
| T07 | ana · darDeBajaUnidades (API-003-03) | `E_FORBIDDEN_ROLE` · unidades activas intactas | error | INV-ACC-005, PERMISSIONS-MATRIX (derived) |
| T08 | ana · marcarMultaPagada multa 6 € de lucia (API-004-02) | `E_FORBIDDEN_ROLE` · multa sigue "Pendiente" | error | AC-015-02, INV-ACC-005 |
| T09 | ana repite la misma petición rechazada, expand: enum(002-01, 002-03, 002-04, 002-05, 003-02, 003-03, 004-02) | `E_FORBIDDEN_ROLE` otra vez · mismo literal · estado intacto | replay | AC-015-04 |
| T10 | ana · escritura con entrada inválida (rango, unidades 0, id inexistente) | `E_FORBIDDEN_ROLE` · precedencia rol vs validación undefined in spec | boundary | — (D-T-004 solo cubre UC-002) |
| T11 | ana · escritura sin sesión | `E_AUTH_REQUIRED` · "Iniciar sesión" | derived | INV-ACC-003 |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F-015-01 | Solo AC-015-02/03 nombran operaciones; las otras 5 escrituras dependen de INV-ACC-005 sin AC propio | Añadir AC por operación |
| F-015-02 | Precedencia rol vs validación de entrada no definida fuera de UC-002 | Fijar orden en INV-ACC-005 |
