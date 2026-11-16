# BDD-UC-013 — Ver lo prestado y lo que vuelve

> Refs: UC-013; REQ-F-013

Scenario: AC-013-01 — prestado [REQ-F-013 AC1]
  Given "Casco" de lucia entregado con fin 2026-12-12 y "Cuerda" de pablo entregada con fin 2026-12-13
  When el encargado abre Hoy
  Then ve "Prestado: Casco (lucia@club.es)" y "Prestado: Cuerda (pablo@club.es)"

Scenario: AC-013-02 — vuelve un día [REQ-F-013 AC2]
  When el encargado elige 2026-12-12
  Then ve "Vuelve: Casco (lucia@club.es)" y no la Cuerda

Scenario: AC-013-03 — nada prestado [REQ-F-013 AC3]
  Given nada prestado
  When el encargado abre Hoy
  Then ve "No hay nada prestado" en Hoy

Scenario: AC-013-04 — socio no ve Hoy
  When un socio abre Hoy
  Then error `E_FORBIDDEN_ROLE`

Scenario: AC-013-05 — sin sesión
  When alguien sin sesión pide Hoy
  Then error `E_AUTH_REQUIRED`

Scenario: AC-013-06 — abrir otra vez
  When el encargado abre Hoy dos veces
  Then ve lo mismo
