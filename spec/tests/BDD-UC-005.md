# BDD-UC-005 — Cancelar una reserva

> Refs: UC-005; REQ-F-005; INV-RES-004

Scenario: AC-005-01 — cancelar a tiempo [REQ-F-005 AC1]
  Given hoy es 2026-12-09 y lucia tiene "Casco" desde 2026-12-10
  When la cancela
  Then ve "Reserva cancelada" en Mis reservas y la reserva muestra "Cancelada"

Scenario: AC-005-02 — la unidad vuelve a estar libre [REQ-F-005 AC2]
  Given hoy es 2026-12-09 y 3 de 3 "Casco" reservados el 2026-12-10, una de lucia
  When lucia la cancela y pablo reserva "Casco" para 2026-12-10
  Then ve "Reserva creada: Casco del 2026-12-10 al 2026-12-10" en Mis reservas

Scenario: AC-005-03 — el mismo día [REQ-F-005 AC3]
  Given hoy es 2026-12-10 y pablo tiene "Cuerda" desde 2026-12-10
  When intenta cancelarla
  Then error `E_CANCEL_TOO_LATE`, ve "Ya no se puede cancelar: la reserva empieza hoy" en Mis reservas
  And sigue "Reservada"

Scenario: AC-005-04 — cancelar dos veces [REQ-F-005 AC4]
  Given lucia canceló su reserva
  When la cancela otra vez
  Then sigue "Cancelada" en Mis reservas y ninguna otra reserva cambia

Scenario: AC-005-05 — reserva ajena [REQ-F-005 AC5]
  When lucia intenta cancelar la reserva de pablo
  Then error `E_FORBIDDEN_DATA` y la de pablo sigue "Reservada"

Scenario: AC-005-06 — junta no cancela [REQ-F-015]
  When la junta intenta cancelar una reserva
  Then error `E_FORBIDDEN_ROLE`

Scenario: AC-005-07 — no existe
  When lucia cancela una reserva inexistente
  Then error `E_RES_NOT_FOUND`
