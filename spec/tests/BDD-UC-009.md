# BDD-UC-009 — Bloqueo por multa pendiente

> Refs: UC-009; REQ-F-009; INV-FIN-003

Scenario: AC-009-01 — una multa [REQ-F-009 AC1]
  Given lucia con multa pendiente de 6 €
  When reserva "Piolet" del 2027-01-10 al 2027-01-11
  Then error `E_FINE_PENDING`, ve "No puedes reservar hasta pagar tu multa de 6 €" en Material
  And no se reserva nada

Scenario: AC-009-02 — varias multas [REQ-F-009 AC2]
  Given pablo con multas pendientes de 2 € y 4 €
  When intenta reservar cualquier cosa
  Then ve "No puedes reservar hasta pagar tu multa de 6 €"

Scenario: AC-009-03 — otro socio sin multa [REQ-F-009 AC3]
  Given lucia con multa y marta sin ninguna
  When marta reserva "Piolet"
  Then ve "Reserva creada: Piolet del 2027-01-10 al 2027-01-11" en Mis reservas

Scenario: AC-009-04 — reservas previas [REQ-F-009 AC4]
  Given lucia con reservas anteriores
  When se crea su multa
  Then sus reservas siguen "Reservada" en Mis reservas

Scenario: AC-009-05 — pago a la vez
  Given pablo con una multa pendiente que se paga mientras reserva
  Then el resultado depende del estado de la multa al confirmar y nunca queda una reserva creada con multa pendiente
