# Clarifications (business rules)

> Decididas por el cliente (preguntas de requisitos) o por defecto del arnés · 2026-11-16. Vinculantes para todo `spec/`.

## Format and structure decisions

| # | Question | Decision | Rejected |
|---|---|---|---|
| D-001 | Formato de especificación | árbol modular `spec/` con casos de uso, BDD y contratos `Style: operations` | SRS monolítico |
| D-002 | Modo de ejecución | secuencial, una hebra | lanes (ver highlights) |

## Business rules

| RN | Source | Question | Rule adopted | Rejected | Tier |
|---|---|---|---|---|---|
| RN-001 | REQ-F-001 (Q-requirements-engineer-001) | ¿Cómo se dan de alta socios y roles? | El encargado prepara las cuentas y roles; no hay pantalla de registro | autoregistro | 2 |
| RN-002 | REQ-F-012 (Q-requirements-engineer-002) | ¿Reserva futura de una unidad que se da de baja? | La baja se permite; la reserva sigue y el encargado avisa a quien corresponda | rechazar la baja | 2 |
| RN-003 | REQ-F-002 | ¿Ocupa unidad el rango `desde..hasta` inclusivo? | Sí, ambos días inclusive, mientras la reserva sea Reservada o Entregada | solo noches | 2 |
| RN-004 | REQ-F-008 | ¿Días de retraso? | `devueltaEl − hasta` en días naturales; devolver en `hasta` o antes = sin multa | contar el día de entrega | 2 |
| RN-005 | REQ-F-017..019 (Q-security-auditor-001) | ¿Qué pasa tras fallar la contraseña muchas veces y cuánto dura una sesión? | Bloqueo de la cuenta 15 min tras 5 fallos en 15 min; sesión caduca a los 30 min sin actividad; botón "Salir" | solo botón de salir; dejarlo como está | 2 |
