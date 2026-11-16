# BDD-UC-014 — Historial del socio

> Refs: UC-014; REQ-F-014; INV-ACC-004

Scenario: AC-014-01 — con multa [REQ-F-014 AC1]
  Given lucia con "Casco" 2026-12-10 a 2026-12-12 devuelto el 2026-12-15 con multa de 6 €
  When la junta abre su historial
  Then ve "Casco" "2026-12-10" "2026-12-12" y "Multa de 6 € (Pendiente)" en Historial del socio

Scenario: AC-014-02 — sin multas [REQ-F-014 AC2]
  Given pablo con un préstamo de "Cuerda" y sin multas
  When la junta abre su historial
  Then ve "Cuerda" y "Sin multas"

Scenario: AC-014-03 — otro socio [REQ-F-016 AC1]
  When lucia (socio) abre el historial de pablo
  Then error `E_FORBIDDEN_DATA`, ve "No tienes permiso para ver estos datos" en Historial del socio y ningún dato de pablo

Scenario: AC-014-04 — sin sesión
  When alguien sin sesión pide un historial
  Then error `E_AUTH_REQUIRED`

Scenario: AC-014-05 — abrir otra vez
  When la junta abre el mismo historial dos veces
  Then ve lo mismo

Scenario: AC-014-06 — solo lo entregado
  Given pablo con "Cuerda" Devuelta, "Casco" Reservada y "Tienda" Cancelada
  When la junta abre su historial
  Then ve "Cuerda" y no ve "Casco" ni "Tienda"
