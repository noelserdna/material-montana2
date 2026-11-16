# BDD-UC-003 — Ver el catálogo

> Refs: UC-003; REQ-F-003

Scenario: AC-003-01 — varios materiales [REQ-F-003 AC1]
  Given 4 unidades activas de "Piolet" y 3 de "Casco"
  When lucia abre Material
  Then ve "Piolet" con "4 unidades" y "Casco" con "3 unidades"

Scenario: AC-003-02 — una unidad [REQ-F-003 AC2]
  Given 1 unidad activa de "Tienda"
  When pablo abre Material
  Then ve "Tienda" con "1 unidades"

Scenario: AC-003-03 — sin sesión [REQ-F-016 AC4]
  When alguien sin sesión pide Material
  Then error `E_AUTH_REQUIRED`, ve "Iniciar sesión" en Iniciar sesión y no recibe datos

Scenario: AC-003-04 — abrir otra vez
  When lucia abre Material dos veces seguidas
  Then ve la misma lista
