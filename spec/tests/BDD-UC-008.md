# BDD-UC-008 — Multa por retraso

> Refs: UC-008; REQ-F-008; INV-FIN-001, INV-FIN-002

Scenario: AC-008-01 — tres días [REQ-F-008 AC1]
  Given "Casco" de lucia con fin 2026-12-12
  When el encargado lo marca devuelto el 2026-12-15
  Then la multa es 3 días × 2 € y ve "Multa de 6 € por 3 días de retraso" en Multas

Scenario: AC-008-02 — un día [REQ-F-008 AC2]
  Given "Cuerda" de pablo con fin 2026-12-21
  When se devuelve el 2026-12-22
  Then ve "Multa de 2 € por 1 días de retraso"

Scenario: AC-008-03 — sin retraso [REQ-F-008 AC3]
  Given una "Tienda" con fin 2026-12-20
  When se devuelve el 2026-12-19 o el 2026-12-20
  Then no se crea multa y ve "Sin multas" en Multas

Scenario: AC-008-04 — misma devolución otra vez [REQ-F-008 AC4]
  Given una multa de 6 € creada por esa devolución
  When se repite la devolución
  Then el socio sigue con una multa y ve "Multa de 6 € por 3 días de retraso"

Scenario: AC-008-05 — fallo de almacén
  When falla el almacén al crear la multa
  Then error `E_STORAGE_UNAVAILABLE` y la reserva sigue Entregada
