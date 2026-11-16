# BDD-UC-007 — Marcar la devolución

> Refs: UC-007; REQ-F-007; INV-RES-005, INV-FIN-002

Scenario: AC-007-01 — en plazo [REQ-F-007 AC1]
  Given "Casco" de lucia entregado, vence 2026-12-12
  When el encargado lo marca devuelto el 2026-12-12
  Then ve "Devuelta el 2026-12-12 sin retraso" en Entregas y devoluciones

Scenario: AC-007-02 — devolver dos veces [REQ-F-007 AC2]
  Given una reserva entregada aún no devuelta
  When el encargado la marca devuelta dos veces
  Then hay un solo registro de devolución, ninguna segunda multa y ve "Ya estaba devuelta"

Scenario: AC-007-03 — nunca entregada [REQ-F-007 AC3]
  Given una reserva que nunca se entregó
  When el encargado la marca devuelta
  Then error `E_NOT_DELIVERED`, ve "Solo se puede devolver lo que se ha entregado"

Scenario: AC-007-04 — unidad libre tras devolver
  Given 1 unidad de "Casco" entregada y devuelta
  When pablo reserva "Casco" para días posteriores a la devolución
  Then se crea la reserva

Scenario: AC-007-05 — socio no devuelve [REQ-F-015]
  When un socio intenta marcar una devolución
  Then error `E_FORBIDDEN_ROLE`

Scenario: AC-007-06 — no existe
  When el encargado devuelve una reserva inexistente
  Then error `E_RES_NOT_FOUND`
