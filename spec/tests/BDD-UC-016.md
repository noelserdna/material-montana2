# BDD-UC-016 — Datos solo del dueño

> Refs: UC-016; REQ-F-016; INV-ACC-003, INV-ACC-004

Scenario: AC-016-01 — lucia mira a pablo [REQ-F-016 AC1]
  When lucia abre la dirección del historial de pablo
  Then error `E_FORBIDDEN_DATA`, ve "No tienes permiso para ver estos datos" en Historial del socio y ningún dato de pablo

Scenario: AC-016-02 — pablo mira a lucia [REQ-F-016 AC2]
  When pablo abre el historial de lucia
  Then ve "No tienes permiso para ver estos datos"

Scenario: AC-016-03 — encargado ve multas [REQ-F-016 AC3]
  Given lucia con multa de 6 €
  When el encargado abre Multas
  Then ve "lucia@club.es" con "Multa de 6 €"

Scenario: AC-016-04 — sin sesión [REQ-F-016 AC4]
  When nadie ha entrado y se pide Material
  Then error `E_AUTH_REQUIRED`, ve "Iniciar sesión" en Iniciar sesión y no se devuelven datos

Scenario: AC-016-05 — repetir
  When lucia repite la petición del historial de pablo
  Then error `E_FORBIDDEN_DATA` otra vez
