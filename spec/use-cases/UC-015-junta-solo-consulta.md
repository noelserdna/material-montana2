# UC-015 — La junta solo consulta

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-015 (primary); API-002-01, API-002-03..05, API-003-02..03, API-004-02; INV-ACC-005; PERMISSIONS-MATRIX; BDD-UC-015 |
| Actors | Primary: junta. Secondary: todas las operaciones de escritura |
| Trigger | La junta intenta una operación de escritura |
| Priority / Status | Must / Approved |

Cualquier escritura de la junta se rechaza y su pantalla no ofrece botones de escritura.

## Input / Output
```typescript
type EscrituraJunta = never; // ninguna operación de escritura admite rol 'junta' (INV-ACC-005)
```

## Preconditions
1. Sesión con rol junta.

## Postconditions
- Failure (siempre): estado intacto.

## Main flow
1. Junta: abre Historial del socio (UC-014) sin botones de reservar, cancelar, entregar, devolver, dar de baja ni pagar.
2. System: rechaza toda escritura con `E_FORBIDDEN_ROLE` (INV-ACC-005).

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | pide pagar una multa | `E_FORBIDDEN_ROLE` | sigue "Pendiente" | AC-015-02 |
| E2 | 2 | pide reservar | `E_FORBIDDEN_ROLE` | sin reserva | AC-015-03 |
| E3 | 2 | replay: repite la petición | `E_FORBIDDEN_ROLE` | mismo rechazo | AC-015-04 |
