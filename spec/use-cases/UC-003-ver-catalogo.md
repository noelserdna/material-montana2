# UC-003 — Ver el catálogo de material

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-003 (primary); API-003-01; INV-ACC-003; BDD-UC-003 |
| Actors | Primary: socio, encargado o junta. Secondary: inventario |
| Trigger | Abre Material |
| Priority / Status | Must / Approved |

Ve cada material con su número de unidades activas.

## Input / Output
```typescript
interface CatalogoOutput { materiales: { nombre: string; unidadesActivas: number }[] /* "{n} unidades" */ }
```

## Preconditions
1. Sesión abierta (INV-ACC-003).

## Postconditions
- Success: lista de materiales con unidades activas; sin cambios de estado.
- Failure: no se muestran datos.

## Main flow
1. Persona: abre Material.
2. System: lista cada material con "{n} unidades" (solo activas, INV-INV-002).

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 1 | sin sesión | `E_AUTH_REQUIRED` | enviado a Iniciar sesión, sin datos | AC-003-03 |
| E2 | 2 | replay: abrir de nuevo | — | misma lista; lectura sin efectos | AC-003-04 |
