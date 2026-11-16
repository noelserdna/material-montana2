# BDD-UC-002 — Reservar material

> Refs: UC-002; REQ-F-002; INV-RES-001..003

Scenario: AC-002-01 — reserva con todo libre [REQ-F-002 AC1]
  Given 3 unidades de "Casco" sin reservas
  When lucia reserva "Casco" del 2026-12-10 al 2026-12-12
  Then ve "Reserva creada: Casco del 2026-12-10 al 2026-12-12" en Mis reservas

Scenario: AC-002-02 — queda una libre [REQ-F-002 AC2]
  Given 3 "Casco", 2 reservadas del 2026-12-10 al 2026-12-12
  When pablo reserva "Casco" del 2026-12-11 al 2026-12-13
  Then ve "Reserva creada: Casco del 2026-12-11 al 2026-12-13" en Mis reservas

Scenario: AC-002-03 — sin unidades [REQ-F-002 AC3]
  Given 3 "Casco", 3 reservadas del 2026-12-10 al 2026-12-12
  When pablo reserva "Casco" del 2026-12-12 al 2026-12-14
  Then error `E_NO_STOCK`, ve "No quedan unidades libres de Casco esas fechas" en Material
  And no se crea reserva

Scenario: AC-002-04 — días sin solape [REQ-F-002 AC4]
  Given 3 "Casco" reservadas del 2026-12-10 al 2026-12-12
  When un socio reserva "Casco" del 2026-12-13 al 2026-12-14
  Then ve "Reserva creada: Casco del 2026-12-13 al 2026-12-14" en Mis reservas

Scenario: AC-002-05 — repetir el formulario [REQ-F-002 AC5]
  Given lucia acaba de reservar "Casco" del 2026-12-10 al 2026-12-12
  When se envía otra vez el mismo formulario
  Then error `E_DUP_RESERVATION`, ve "Ya tienes esa reserva" en Material
  And lucia tiene una sola reserva de "Casco" para esas fechas

Scenario: AC-002-06 — fin antes del inicio [REQ-F-002 AC6]
  When lucia envía inicio 2026-12-11 y fin 2026-12-09
  Then error `E_DATE_RANGE`, ve "La fecha de fin no puede ser anterior a la de inicio" en Material
  And no se reserva nada

Scenario: AC-002-07 — junta no reserva [REQ-F-015 AC3]
  When una persona de la junta pide reservar "Casco"
  Then error `E_FORBIDDEN_ROLE` y no se crea reserva

Scenario: AC-002-08 — con multa pendiente [REQ-F-009 AC1]
  Given lucia con multa pendiente de 6 €
  When reserva "Piolet" del 2027-01-10 al 2027-01-11
  Then error `E_FINE_PENDING`, ve "No puedes reservar hasta pagar tu multa de 6 €" en Material

Scenario: AC-002-09 — material inexistente
  When lucia reserva un material que no existe
  Then error `E_MATERIAL_NOT_FOUND` y no se crea reserva

Scenario: AC-002-10 — última unidad a la vez
  Given 1 unidad libre de "Casco" y dos socios que reservan a la vez
  Then solo una reserva se crea y la otra recibe `E_NO_STOCK`
