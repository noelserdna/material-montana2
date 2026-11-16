# BDD-UC-019 — Ver las reservas de todos los socios

> Refs: UC-019; REQ-F-020

Scenario: AC-019-01 — por entregar [REQ-F-020 AC1]
  Given "Casco" de lucia Reservada del 2026-12-10 al 2026-12-12 y "Cuerda" de pablo Entregada
  When el encargado abre Entregas y devoluciones
  Then ve "Casco" "lucia@club.es" y no la Cuerda

Scenario: AC-019-02 — entregadas [REQ-F-020 AC2]
  When el encargado elige el filtro "Entregadas"
  Then ve "Cuerda" "pablo@club.es" y no el Casco

Scenario: AC-019-03 — todas [REQ-F-020 AC3]
  When el encargado elige el filtro "Todas"
  Then ve "Casco" "lucia@club.es" y "Cuerda" "pablo@club.es"

Scenario: AC-019-04 — sin reservas [REQ-F-020 AC4]
  Given ninguna reserva coincide con el filtro
  When el encargado abre Entregas y devoluciones
  Then ve "No hay reservas" en Entregas y devoluciones

Scenario: AC-019-05 — socio rechazado [REQ-F-020 AC5]
  When un socio pide la lista de todas las reservas
  Then error `E_FORBIDDEN_ROLE`

Scenario: AC-019-06 — junta rechazada [REQ-F-020 AC6]
  When la junta pide la lista de todas las reservas
  Then error `E_FORBIDDEN_ROLE`

Scenario: AC-019-07 — sin sesión
  When alguien sin sesión pide la lista
  Then error `E_AUTH_REQUIRED`

Scenario: AC-019-08 — abrir otra vez
  When el encargado abre la lista dos veces
  Then ve lo mismo
