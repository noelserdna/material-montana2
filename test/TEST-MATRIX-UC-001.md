# Test Matrix: UC-001 — Entrar con correo y contraseña

> Refs: UC-001, API-001-01, BDD-UC-001 (AC-001-01..11), REQ-F-017, INV-ACC-001, INV-ACC-002, INV-ACC-003, INV-ACC-006, RN-001, RN-005
> Techniques: EP, BVA, decision table, replay, race · Default level: integration · Conventions: TEST-PLAN.md §3

## Inputs

| Input | Type | Valid classes | Invalid classes | Boundaries |
|-------|------|---------------|-----------------|------------|
| correo | Correo (VO-001) | cuenta existente (lucia, pablo, marta, ana) | correo sin cuenta · vacío · solo espacios | `""`; mayúsculas/minúsculas del mismo correo |
| contrasena | string | la de la cuenta | distinta · vacía | `""`; prefijo/sufijo de la correcta |
| sesión previa | estado | ninguna · una abierta de la misma cuenta | — | 0 / 1 sesiones por correo |
| almacén de cuentas | persistido | hash con sal por cuenta | — | dos cuentas con la misma contraseña |
| fallos de la cuenta | contador en 15 min (`LOGIN_MAX_FAILURES`=5, `LOGIN_WINDOW_MIN`=15) | 0–4 en ventana · 5 fuera de ventana | 5 en ventana | `expand: BVA(fallos 4,5)` · último fallo a +14, +15, +16 min |
| hora del intento | Instante (VO-007), reloj inyectable (D-T-011) | 10:11 (bloqueada) · 10:26 (libre) | — | `LOCKOUT_MIN`=15 desde el último fallo |

## Cases

`Type`: happy · error · boundary · state · replay · race · derived (no AC of its own — cite the rule in Refs). The test implementing a row is named with the row's scenario id.

| ID | Precondition / Input | Expected (status · output · state) | Type | Refs |
|----|----------------------|-------------------------------------|------|------|
| T01 | `lucia@club.es` / `Cumbre2026` · sin sesión | ok · saludo "Hola, lucia@club.es", rol socio · 1 sesión de lucia | happy | AC-001-01 |
| T02 | `pablo@club.es` / `Hielo2026` | ok · "Hola, pablo@club.es", rol socio · 1 sesión de pablo | happy | AC-001-02 |
| T03 | expand: enum(marta=encargado, ana=junta) con su contraseña | ok · "Hola, {correo}" con el rol de la cuenta · 1 sesión | derived | PERMISSIONS-MATRIX (API-001-01 todos los roles) |
| T04 | lucia + `incorrecta` | `E_AUTH_INVALID` · "Correo o contraseña incorrectos" · sin sesión, se queda en Iniciar sesión | error | AC-001-03 |
| T05 | correo sin cuenta + contraseña cualquiera | `E_AUTH_INVALID` · mismo mensaje que T04 · sin sesión | error | AC-001-03 |
| T06 | expand: enum(correo vacío, contraseña vacía, ambos vacíos) | `E_AUTH_INVALID` · "Correo o contraseña incorrectos" · sin sesión | error | AC-001-05 |
| T07 | correcta con una letra cambiada / con espacio final / correo con otra capitalización | `E_AUTH_INVALID` según regla de comparación; capitalización de correo: `undefined in spec` | boundary | AC-001-03 |
| T08 | lucia ya con sesión · reenvía el mismo formulario | ok · "Hola, lucia@club.es" · sigue 1 sola sesión | replay | AC-001-04, INV-ACC-002 |
| T09 | lucia con sesión · reenvía con contraseña incorrecta | `E_AUTH_INVALID` · sesión previa: `undefined in spec` | state | — |
| T10 | dos envíos simultáneos válidos de lucia | `undefined in spec` salvo invariante: sesiones de lucia ≤ 1 | race | INV-ACC-002 |
| T11 | cuentas lucia y otra con la misma contraseña · leer almacén | lo guardado no contiene `Cumbre2026` y difiere entre cuentas · sin cambios | derived | AC-001-06, INV-ACC-001 |
| T12 | almacén caído al iniciar sesión | `E_STORAGE_UNAVAILABLE` · "No se puede acceder a los datos; inténtalo de nuevo" · sin sesión | error | `undefined in spec` (sin AC, ver F01) |
| T13 | intento de alta con correo nuevo | ningún camino de alta: `E_AUTH_INVALID` · sin cuenta ni sesión | derived | RN-001, D-T-003 |
| T14 | lucia: 5 × `incorrecta` 10:00–10:10 · envía `Cumbre2026` 10:11 | `E_AUTH_LOCKED` · "Demasiados intentos fallidos: espera 15 minutos" en Iniciar sesión · sin sesión | error | AC-001-07 |
| T15 | cuenta bloqueada por el 5.º fallo de 10:10 · envía `Cumbre2026` 10:26 | ok · "Hola, lucia@club.es" en Material · 1 sesión | state | AC-001-08 |
| T16 | C1 fallos=4 en 15 min · C2 contraseña correcta | ok · "Hola, lucia@club.es" · contador a 0 (4 fallos más no bloquean) | state | AC-001-09 |
| T17 | lucia bloqueada · pablo envía `Hielo2026` | ok · "Hola, pablo@club.es" en Material · lucia sigue bloqueada | state | AC-001-10 |
| T18 | 5 fallos, el 1.º a 10:00 y el 5.º a 10:16 · correcta 10:17 | ok · "Hola, lucia@club.es" · no bloqueada (no caben en 15 min) | boundary | AC-001-11 |
| T19 | `expand: BVA` 4 fallos (ok) y 5 (bloqueo) · envío a +14 min (`E_AUTH_LOCKED`) y +16 min (ok) del último fallo | por punto: ok o `E_AUTH_LOCKED` · sin sesión si bloqueada | boundary | AC-001-07, AC-001-08, AC-001-09 |
| T20 | bloqueada · envío exactamente a +15:00 del último fallo | undefined in spec ("hasta que pasen 15 min") | boundary | — |
| T21 | T14 enviado otra vez a 10:12 (aún bloqueada) | `E_AUTH_LOCKED` · mismo mensaje · sin sesión; si el intento alarga el bloqueo: undefined in spec | replay | AC-001-07 |
| T22 | 4 fallos · dos `incorrecta` simultáneos | undefined in spec (¿5 o 6 fallos?) | race | — |
| T23 | correo sin cuenta · 5 fallos y envío posterior | `E_AUTH_INVALID` según T05; ¿bloquea un correo sin cuenta?: undefined in spec | boundary | — |

## Findings for sdd-spec-auditor

| ID | Observation (≤ 120 chars) | Suggested action |
|----|----------------------------|------------------|
| F01 | `E_STORAGE_UNAVAILABLE` en API-001-01 sin AC ni excepción en UC-001 | Añadir E# y AC o retirar el código del contrato |
| F02 | Sin regla sobre sesión previa al fallar un reenvío, ni sobre mayúsculas/espacios del correo | Definir en UC-001 y VO-001 |
| F03 | Dos accesos simultáneos de la misma cuenta sin AC | Añadir AC de race o declarar INV-ACC-002 suficiente |
| F04 | Sin AC del instante exacto +15:00 tras el último fallo ni del límite de la ventana de 15 min | Definir `<` o `≤` en INV-ACC-006 |
| F05 | Sin regla sobre intentos hechos con la cuenta bloqueada (¿cuentan o alargan el bloqueo?) ni sobre correo sin cuenta | Definir en INV-ACC-006 |
| F06 | Dos fallos simultáneos con 4 acumulados sin AC de race | Declarar suficiente la invariante o añadir AC |
