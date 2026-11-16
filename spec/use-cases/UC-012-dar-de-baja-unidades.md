# UC-012 — Dar de baja unidades rotas

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-012 (primary); API-003-03; INV-INV-001, INV-INV-002, INV-RES-001, INV-ACC-005; RN-002; BDD-UC-012 |
| Actors | Primary: encargado. Secondary: inventario, reservas |
| Trigger | Envía la baja en Inventario |
| Priority / Status | Must / Approved |

Las unidades dadas de baja dejan de poder reservarse; las reservas ya hechas no se tocan.

## Input / Output
```typescript
interface BajaInput  { material: string; unidades: Unidades }
interface BajaOutput { material: string; unidadesActivas: number }
```

## Preconditions
1. Sesión del encargado.

## Postconditions
- Success: activas −= `unidades`; reservas existentes sin cambios (RN-002, INV-INV-002).
- Failure: activas sin cambios.

## Main flow
1. Encargado: indica material y unidades rotas.
2. System: valida rol, `unidades ≥ 1` y `unidades ≤ activas` (INV-INV-001, INV-INV-002).
3. System: reduce las activas y muestra "{material}" y "{n} unidades" en Inventario.
4. Encargado: avisa por su cuenta a los socios afectados si algún día queda sin unidad (RN-002).

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | `unidades` > activas | `E_UNITS_INSUFFICIENT` | "No hay tantas unidades activas de Cuerda"; cuenta intacta | AC-012-02 |
| E2 | 2 | `unidades` < 1 | `E_UNITS_MIN` | nada cambia | AC-012-04 |
| E3 | 2 | material inexistente | `E_MATERIAL_NOT_FOUND` | nada cambia | AC-012-05 |
| E4 | 2 | rol socio o junta | `E_FORBIDDEN_ROLE` | nada cambia | AC-012-06 |
| E5 | 3 | baja de una unidad con reserva futura | — | baja permitida; la reserva sigue (RN-002) | AC-012-07 |
| E6 | 3 | replay: misma baja otra vez | — | resta otras unidades por diseño; puede dar `E_UNITS_INSUFFICIENT` | AC-012-08 |
