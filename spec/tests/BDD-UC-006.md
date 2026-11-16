# BDD-UC-006 — Entregar lo reservado

> Refs: UC-006; REQ-F-006; INV-RES-005

Scenario: AC-006-01 — entregar a lucia [REQ-F-006 AC1]
  Given la reserva de "Casco" de lucia del 2026-12-10 al 2026-12-12
  When el encargado la marca entregada
  Then ve "Entregada a lucia@club.es" en Entregas y devoluciones

Scenario: AC-006-02 — entregar a pablo [REQ-F-006 AC2]
  Given la reserva de "Cuerda" de pablo
  When el encargado la marca entregada
  Then ve "Entregada a pablo@club.es"

Scenario: AC-006-03 — ya entregada [REQ-F-006 AC3]
  Given una reserva "Entregada"
  When el encargado la marca entregada otra vez
  Then sigue "Entregada" una vez y ve "Ya estaba entregada"

Scenario: AC-006-04 — cancelada [REQ-F-006 AC4]
  Given una reserva "Cancelada"
  When el encargado intenta entregarla
  Then error `E_RES_CANCELLED`, ve "No se puede entregar una reserva cancelada"

Scenario: AC-006-05 — socio no entrega [REQ-F-006 AC5]
  When lucia (socio) intenta marcar una entrega
  Then error `E_FORBIDDEN_ROLE` y la reserva no cambia

Scenario: AC-006-06 — no existe
  When el encargado entrega una reserva inexistente
  Then error `E_RES_NOT_FOUND`

Scenario: AC-006-07 — devuelta
  Given una reserva "Devuelta"
  When el encargado intenta entregarla
  Then error `E_DELIVER_STATE`, ve "Solo se puede entregar una reserva pendiente de entrega"
  And sigue "Devuelta"
