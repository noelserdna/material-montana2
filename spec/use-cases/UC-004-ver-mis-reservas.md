# UC-004 — Ver mis reservas

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-004 (primary), REQ-F-016; API-002-02; INV-ACC-004, INV-ACC-003; BDD-UC-004 |
| Actors | Primary: socio (o encargado). Secondary: reservas |
| Trigger | Abre Mis reservas |
| Priority / Status | Must / Approved |

Ve solo sus reservas con material, fechas y estado.

## Input / Output
```typescript
interface MisReservasOutput { reservas: { material: string; desde: Dia; hasta: Dia; estado: EstadoReserva /* VO-004 */ }[] }
```

## Preconditions
1. Sesión abierta.

## Postconditions
- Success: solo reservas del propio actor (INV-ACC-004); sin cambios.
- Failure: sin datos.

## Main flow
1. Socio: abre Mis reservas.
2. System: lista sus reservas con "{material} {desde} {hasta} {estado}"; si no hay, "Todavía no tienes reservas".

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 1 | sin sesión | `E_AUTH_REQUIRED` | enviado a Iniciar sesión | AC-004-04 |
| E2 | 2 | intenta ver las de otro socio | `E_FORBIDDEN_DATA` | sin datos ajenos | AC-004-05 |
| E3 | 2 | replay: abrir de nuevo | — | misma lista | AC-004-06 |
