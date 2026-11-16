# BDD-UC-010 — Marcar la multa como pagada

> Refs: UC-010; REQ-F-010; INV-FIN-003

Scenario: AC-010-01 — única multa [REQ-F-010 AC1]
  Given la única multa de lucia, de 6 €
  When el encargado la marca pagada
  Then ve "Multa de 6 € pagada" en Multas y lucia puede reservar "Piolet"

Scenario: AC-010-02 — una de dos [REQ-F-010 AC2]
  Given pablo con multas de 2 € y 4 €
  When el encargado marca solo la de 2 €
  Then la de 4 € sigue "Pendiente" en Multas y pablo no puede reservar

Scenario: AC-010-03 — ya pagada [REQ-F-010 AC3]
  Given una multa "Pagada"
  When el encargado la marca pagada otra vez
  Then sigue pagada una vez y ve "Ya estaba pagada"

Scenario: AC-010-04 — socio no paga [REQ-F-010 AC4]
  When un socio intenta marcar su propia multa pagada
  Then error `E_FORBIDDEN_ROLE` y sigue "Pendiente"

Scenario: AC-010-05 — no existe
  When el encargado marca una multa inexistente
  Then error `E_FINE_NOT_FOUND`
