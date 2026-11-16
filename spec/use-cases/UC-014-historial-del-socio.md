# UC-014 — La junta consulta el historial de un socio

| Field | Value |
|---|---|
| Version | 1.0 / 2026-11-16 |
| Refs | REQ-F-014 (primary), REQ-F-016; API-004-03; INV-ACC-004; BDD-UC-014 |
| Actors | Primary: junta (también encargado). Secondary: reservas, multas |
| Trigger | Elige un socio en Historial del socio |
| Priority / Status | Must / Approved |

Ve los préstamos y multas del socio elegido. Préstamo = reserva Entregada o Devuelta; las Reservada y Cancelada no salen (glosario).

## Input / Output
```typescript
interface HistorialInput  { correo: Correo }
interface HistorialOutput { prestamos: { material: string; desde: Dia; hasta: Dia }[]; multas: { importe: number; estado: EstadoMulta }[] }
```

## Preconditions
1. Sesión de junta o encargado (INV-ACC-004).

## Postconditions
- Success: historial del socio, solo lectura.
- Failure: sin datos del socio.

## Main flow
1. Junta: elige un socio.
2. System: muestra, de sus reservas Entregada o Devuelta, "{material} {desde} {hasta}" y "Multa de {importe} € ({estado})", o "Sin multas".

## Exceptions & errors

| # | Step | Condition | Error code | Effect | AC |
|---|---|---|---|---|---|
| E1 | 1 | actor es otro socio | `E_FORBIDDEN_DATA` | sin datos | AC-014-03 |
| E2 | 1 | sin sesión | `E_AUTH_REQUIRED` | enviado a Iniciar sesión | AC-014-04 |
| E3 | 2 | replay: abrir otra vez | — | mismo historial | AC-014-05 |
