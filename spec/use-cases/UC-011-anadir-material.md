# UC-011 — Añadir material al inventario

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-011 (primary); API-003-02; INV-INV-001, INV-ACC-005; BDD-UC-011 |
| Actors | Primary: encargado. Secondary: inventario |
| Trigger | Envía el formulario de Inventario |
| Priority / Status | Must / Approved |

El encargado da de alta un material nuevo o suma unidades a uno existente.

## Input / Output
```typescript
interface AnadirInput  { material: string; unidades: Unidades /* VO-006 */ }
interface AnadirOutput { material: string; unidadesActivas: number /* "{n} unidades" */ }
```

## Preconditions
1. Sesión del encargado.

## Postconditions
- Success: unidades activas += `unidades` (INV-INV-001).
- Failure: inventario sin cambios.

## Main flow
1. Encargado: indica material y unidades.
2. System: valida rol y `unidades ≥ 1` (INV-INV-001).
3. System: crea el material o suma; muestra "{material}" y "{n} unidades" en Inventario.

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | `unidades` < 1 | `E_UNITS_MIN` | "Indica al menos 1 unidad"; nada añadido | AC-011-03 |
| E2 | 2 | rol socio o junta | `E_FORBIDDEN_ROLE` | sin cambios | AC-011-05 |
| E3 | 3 | replay: mismo formulario otra vez | — | se añaden otras unidades por diseño (acción nueva) | AC-011-04 |
