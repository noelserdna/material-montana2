# UC-018 — La sesión se cierra sola tras 30 minutos sin actividad

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-019 (primary); API-001-01; INV-ACC-003, INV-ACC-007; `SESSION_IDLE_MIN`; BDD-UC-018 |
| Actors | Primary: socio, encargado o junta. Secondary: reloj del servidor |
| Trigger | Cualquier operación con una sesión inactiva más de 30 minutos |
| Priority / Status | Must / Review |

Cada operación con sesión renueva los 30 minutos; pasados sin actividad, la sesión no vale y hay que entrar de nuevo.

## Input / Output
```typescript
interface ComprobarSesionOutput { vigente: boolean; mensaje?: string /* "Tu sesión ha caducado" */ }
```

## Preconditions
1. Hay una sesión con `ultimaActividad` (ENT-005).

## Postconditions
- Success (vigente): `ultimaActividad` = ahora; la operación sigue.
- Failure (caducada): la sesión termina; ve "Tu sesión ha caducado" en Iniciar sesión; no se devuelven datos.

## Main flow
1. Persona: pide cualquier operación con sesión.
2. System: compara `ahora − ultimaActividad` con `SESSION_IDLE_MIN` (INV-ACC-007).
3. System: si es ≤ 30 min, renueva la actividad y ejecuta la operación.

## Extensions
- 2a. Entra de nuevo con sus datos tras caducar → sesión nueva y "Hola, {correo}". (AC-018-04)

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | más de 30 min sin actividad | `E_SESSION_EXPIRED` | sesión terminada; "Tu sesión ha caducado"; sin datos | AC-018-01 |
