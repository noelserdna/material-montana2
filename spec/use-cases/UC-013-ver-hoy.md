# UC-013 — Ver lo prestado y las devoluciones de cada día

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-013 (primary); API-002-06; INV-RES-005, INV-ACC-005; BDD-UC-013 |
| Actors | Primary: encargado. Secondary: reservas |
| Trigger | Abre Hoy o elige un día |
| Priority / Status | Must / Approved |

El encargado ve lo prestado ahora y lo que vuelve el día elegido.

## Input / Output
```typescript
interface HoyInput  { dia?: Dia }
interface HoyOutput { prestado: { material: string; correo: Correo }[]; vuelve: { material: string; correo: Correo }[] }
```

## Preconditions
1. Sesión del encargado.

## Postconditions
- Success: lectura sin cambios; prestado = reservas `Entregada`; vuelve = Entregadas con `hasta` = día elegido.
- Failure: sin datos.

## Main flow
1. Encargado: abre Hoy y, opcionalmente, elige un día.
2. System: lista "Prestado: {material} ({correo})".
3. System: lista "Vuelve: {material} ({correo})" del día; si nada prestado, "No hay nada prestado".

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 1 | rol socio o junta | `E_FORBIDDEN_ROLE` | sin datos | AC-013-04 |
| E2 | 1 | sin sesión | `E_AUTH_REQUIRED` | enviado a Iniciar sesión | AC-013-05 |
| E3 | 2 | replay: abrir otra vez | — | misma lista | AC-013-06 |
