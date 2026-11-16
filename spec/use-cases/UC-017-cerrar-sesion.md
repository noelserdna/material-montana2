# UC-017 — Cerrar sesión con un botón

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-018 (primary); API-001-02; INV-ACC-002, INV-ACC-003, INV-ACC-007; BDD-UC-017 |
| Actors | Primary: socio, encargado o junta |
| Trigger | Elige "Salir" |
| Priority / Status | Must / Review |

La sesión termina y la persona vuelve a Iniciar sesión.

## Input / Output
```typescript
interface CerrarSesionInput  { /* sesión actual */ }
interface CerrarSesionOutput { destino: 'Iniciar sesión' }
```

## Preconditions
1. Hay una sesión (INV-ACC-003). Sin ella el resultado es el mismo destino, sin cambios.

## Postconditions
- Success: la sesión queda cerrada y no vale más (INV-ACC-007); ve "Iniciar sesión".
- Failure: ninguna variante cambia datos.

## Main flow
1. Persona: elige "Salir".
2. System: cierra la sesión (INV-ACC-007).
3. System: muestra Iniciar sesión.

## Extensions
- 2a. Repite "Salir" con la sesión ya cerrada → mismo destino, nada más cambia. (AC-017-03)

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | uso de la sesión cerrada en otra petición | `E_AUTH_REQUIRED` | sin datos; ve "Iniciar sesión" | AC-017-02 |
| E2 | 2 | replay: "Salir" con sesión ya cerrada | — | ve "Iniciar sesión"; nada cambia | AC-017-03 |
