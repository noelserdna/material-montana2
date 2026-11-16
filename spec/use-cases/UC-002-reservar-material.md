# UC-002 — Reservar material para unas fechas

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-002 (primary), REQ-F-009; API-002-01; WF-001 step 1; INV-RES-001, INV-RES-002, INV-RES-003, INV-FIN-003, INV-ACC-005; RN-003; BDD-UC-002 |
| Actors | Primary: socio (o encargado). Secondary: inventario, multas |
| Trigger | Envía el formulario de reserva en Material |
| Priority / Status | Must / Approved |

El socio aparta una unidad de un material entre dos días si queda alguna libre cada día del rango.

## Input / Output
```typescript
interface ReservarInput  { material: string; desde: Dia; hasta: Dia /* INV-RES-002 */ }
interface ReservarOutput { reserva: Reserva; mensaje: string /* "Reserva creada: {material} del {desde} al {hasta}" */ }
```

## Preconditions
1. Sesión abierta (INV-ACC-003).

## Postconditions
- Success: reserva `Reservada` creada y visible en Mis reservas (INV-RES-001).
- Failure: no se crea reserva ni se ocupa unidad.

## Main flow
1. Socio: elige material y fechas.
2. System: valida el rango (INV-RES-002) y el rol (INV-ACC-005).
3. System: comprueba que no hay multa pendiente (INV-FIN-003).
4. System: comprueba unidades libres cada día (INV-RES-001, RN-003) y que no hay duplicado (INV-RES-003).
5. System: crea la reserva y muestra "Reserva creada: …" en Mis reservas.

## Extensions
- 4a. Quedan unidades libres por solapes parciales → se crea. (AC-002-02)
- 4b. Rango sin solape con las ocupadas → se crea. (AC-002-04)

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | `hasta` < `desde` | `E_DATE_RANGE` | nada reservado | AC-002-06 |
| E2 | 2 | rol junta | `E_FORBIDDEN_ROLE` | nada reservado | AC-002-07 |
| E3 | 3 | multa pendiente | `E_FINE_PENDING` | nada reservado (UC-009) | AC-002-08 |
| E4 | 4 | ningún día libre en algún día | `E_NO_STOCK` | nada reservado | AC-002-03 |
| E5 | 4 | material inexistente | `E_MATERIAL_NOT_FOUND` | nada reservado | AC-002-09 |
| E6 | 4 | dos socios piden la última unidad a la vez | `E_NO_STOCK` | gana el primero confirmado; el otro ve el error | AC-002-10 |
| E7 | 5 | replay: mismo formulario | `E_DUP_RESERVATION` | sigue una sola reserva; ve "Ya tienes esa reserva" | AC-002-05 |
