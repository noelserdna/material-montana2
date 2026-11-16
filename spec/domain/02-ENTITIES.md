# 02 — Entities

```typescript
/** ENT-001 — Cuenta. Invariants: INV-ACC-001..005. Creada por el encargado (RN-001). */
interface Cuenta { correo: string /* VO-001, único */; hash: string /* salted, INV-ACC-001 */; rol: Rol /* VO-002 */ }
/** ENT-002 — Material. Unidades activas = altas − bajas (INV-INV-001..002). */
interface Material { nombre: string /* único */; unidadesActivas: number /* ≥ 0 */ }
/** ENT-003 — Reserva. Lifecycle: SM-001. Invariants: INV-RES-001..005. */
interface Reserva { id: number; socio: string /* correo */; material: string; desde: Dia; hasta: Dia; estado: EstadoReserva; devueltaEl: Dia | null }
/** ENT-004 — Multa. Lifecycle: SM-002. Invariants: INV-FIN-001..003. */
interface Multa { id: number; reservaId: number /* única por reserva */; socio: string; dias: number; importe: number /* dias × FINE_RATE_EUR */; estado: EstadoMulta }
/** ENT-005 — Sesión. Una activa por cuenta (INV-ACC-002). */
interface Sesion { correo: string }
```

Relationships: Cuenta 1→0..n Reserva; Material 1→0..n Reserva; Reserva 1→0..1 Multa; Cuenta 1→0..n Multa.
