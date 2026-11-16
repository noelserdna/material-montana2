# UC-019 — Ver las reservas de todos los socios

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-020 (primary); API-002-07; INV-ACC-004, INV-ACC-005, INV-RES-005; BDD-UC-019 |
| Actors | Primary: encargado. Secondary: reservas |
| Trigger | Abre Entregas y devoluciones o cambia el filtro |
| Priority / Status | Must / Review |

El encargado ve las reservas de todos los socios, filtradas por estado.

## Input / Output
```typescript
type FiltroReservas = 'Por entregar' | 'Entregadas' | 'Todas'
interface ListarReservasInput  { filtro?: FiltroReservas /* por defecto 'Por entregar' */ }
interface ListarReservasOutput { reservas: { id: number; material: string; correo: Correo; desde: Dia; hasta: Dia; estado: EstadoReserva }[]; mensaje?: string /* "No hay reservas" */ }
```

## Preconditions
1. Sesión del encargado.

## Postconditions
- Success: lectura sin cambios. "Por entregar" = reservas `Reservada`; "Entregadas" = reservas `Entregada`; "Todas" = todas las reservas (Reservada, Entregada, Devuelta, Cancelada).
- Failure: sin datos.

## Main flow
1. Encargado: abre Entregas y devoluciones (filtro "Por entregar") o elige otro filtro.
2. System: comprueba rol (INV-ACC-004).
3. System: lista "{material}" "{correo}" con fechas y estado; si no hay ninguna, "No hay reservas".

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | rol socio o junta | `E_FORBIDDEN_ROLE` | sin datos | AC-019-05, AC-019-06 |
| E2 | 2 | sin sesión | `E_AUTH_REQUIRED` | enviado a Iniciar sesión | AC-019-07 |
| E3 | 3 | replay: abrir otra vez | — | misma lista | AC-019-08 |
