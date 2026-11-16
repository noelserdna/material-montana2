# BDD-UC-017 — Cerrar sesión con un botón

> Refs: UC-017; REQ-F-018; INV-ACC-007

Scenario: AC-017-01 — salir [REQ-F-018 AC1]
  Given lucia tiene la sesión abierta
  When elige "Salir"
  Then ve "Iniciar sesión" y no queda sesión

Scenario: AC-017-02 — la sesión cerrada no vale [REQ-F-018 AC2]
  Given lucia eligió "Salir"
  When se pide la dirección de Material con su sesión anterior
  Then error `E_AUTH_REQUIRED`, ve "Iniciar sesión" y no se devuelven datos

Scenario: AC-017-03 — repetir "Salir" [REQ-F-018 AC3]
  Given lucia eligió "Salir"
  When vuelve a elegir "Salir" con la sesión anterior
  Then ve "Iniciar sesión" y nada más cambia
