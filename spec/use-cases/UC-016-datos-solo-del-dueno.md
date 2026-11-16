# UC-016 — Datos de los socios solo para su dueño, el encargado y la junta

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-016 (primary), REQ-NF-002; API-002-02, API-004-01, API-004-03; INV-ACC-003, INV-ACC-004; BDD-UC-016 |
| Actors | Primary: socio. Secondary: encargado, junta |
| Trigger | Pide datos de otro socio o sin sesión |
| Priority / Status | Must / Approved |

Cada socio ve solo lo suyo; el encargado ve las multas de todos en Multas.

## Input / Output
```typescript
interface AccesoDatos { actor: Correo | null; dueno: Correo } // permitido ⇔ actor = dueno ∨ rol(actor) ∈ {encargado, junta}
```

## Preconditions
1. Ninguna.

## Postconditions
- Failure: ningún dato ajeno devuelto (INV-ACC-004).

## Main flow
1. Persona: pide datos de un socio.
2. System: sin sesión → Iniciar sesión (INV-ACC-003); con sesión, aplica INV-ACC-004.
3. System: encargado ve en Multas "{correo}" y "Multa de {importe} €".

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 2 | socio pide datos de otro | `E_FORBIDDEN_DATA` | "No tienes permiso para ver estos datos" | AC-016-01 |
| E2 | 2 | sin sesión | `E_AUTH_REQUIRED` | enviado a Iniciar sesión, sin datos | AC-016-04 |
| E3 | 2 | replay: repite la petición | `E_FORBIDDEN_DATA` | mismo rechazo | AC-016-05 |
