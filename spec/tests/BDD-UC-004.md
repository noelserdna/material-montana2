# BDD-UC-004 — Ver mis reservas

> Refs: UC-004; REQ-F-004; INV-ACC-004

Scenario: AC-004-01 — lucia [REQ-F-004 AC1]
  Given lucia con "Casco" 2026-12-10 a 2026-12-12 y pablo con "Cuerda" 2026-12-20 a 2026-12-21
  When lucia abre Mis reservas
  Then ve "Casco" "2026-12-10" "2026-12-12" "Reservada" y no ve "Cuerda"

Scenario: AC-004-02 — pablo [REQ-F-004 AC2]
  When pablo abre Mis reservas
  Then ve "Cuerda" "2026-12-20" "2026-12-21" "Reservada" y no ve "Casco"

Scenario: AC-004-03 — sin reservas [REQ-F-004 AC3]
  Given un socio sin reservas
  When abre Mis reservas
  Then ve "Todavía no tienes reservas" en Mis reservas

Scenario: AC-004-04 — sin sesión
  When alguien sin sesión abre Mis reservas
  Then error `E_AUTH_REQUIRED`

Scenario: AC-004-05 — reservas ajenas [REQ-F-016]
  When lucia pide las reservas de pablo
  Then error `E_FORBIDDEN_DATA` y no recibe datos de pablo

Scenario: AC-004-06 — abrir otra vez
  When lucia abre Mis reservas dos veces
  Then ve lo mismo
