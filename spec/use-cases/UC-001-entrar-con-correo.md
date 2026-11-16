# UC-001 — Entrar con correo y contraseña

| Field | Value |
|---|---|
| Version | 1.1 / 2026-11-16 |
| Refs | REQ-F-001 (primary), REQ-F-017, REQ-NF-002; API-001-01; INV-ACC-001, INV-ACC-002, INV-ACC-003, INV-ACC-006; RN-001, RN-005; BDD-UC-001 |
| Actors | Primary: socio, encargado o junta. Secondary: almacén de cuentas |
| Trigger | Envía el formulario de Iniciar sesión |
| Priority / Status | Must / Approved |

La persona obtiene una sesión y llega a Material. Las cuentas las prepara el encargado (RN-001).

## Input / Output
```typescript
interface IniciarSesionInput  { correo: Correo /* VO-001 */; contrasena: string }
interface IniciarSesionOutput { saludo: string /* "Hola, {correo}" */; rol: Rol }
```

## Preconditions
1. La cuenta existe con su hash (INV-ACC-001).

## Postconditions
- Success: sesión única de la cuenta (INV-ACC-002); se ve "Hola, {correo}" en Material.
- Failure: no hay sesión (también con la cuenta bloqueada); se queda en Iniciar sesión.

## Main flow
1. Persona: envía correo y contraseña.
2. System: comprueba que la cuenta no está bloqueada (INV-ACC-006) y verifica la contraseña contra el hash (INV-ACC-001); un fallo suma al contador de la cuenta, un acierto lo pone a cero; con la cuenta bloqueada, intentar más no alarga la espera (contada desde el 5.º fallo).
3. System: abre la sesión y muestra Material.

## Extensions
- 2a. Repite el formulario con sesión abierta → mantiene una sola sesión (INV-ACC-002). (AC-001-04)

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | correo o contraseña no coinciden | `E_AUTH_INVALID` | sin sesión; ve "Correo o contraseña incorrectos" | AC-001-03 |
| E2 | 2 | correo o contraseña vacíos | `E_AUTH_INVALID` | mismo mensaje, sin sesión | AC-001-05 |
| E4 | 2 | la cuenta tiene 5 fallos en 15 min y no han pasado 15 min desde el último | `E_AUTH_LOCKED` | sin sesión, aunque la contraseña sea correcta; ve "Demasiados intentos fallidos: espera 15 minutos" | AC-001-07, AC-001-08 |
| E3 | 3 | replay: mismo formulario con sesión abierta | — | una sola sesión; "Hola, {correo}" | AC-001-04 |
