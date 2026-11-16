# BDD-UC-001 — Entrar con correo y contraseña

> Refs: UC-001; REQ-F-001, REQ-NF-002; INV-ACC-001, INV-ACC-002

Scenario: AC-001-01 — acceso de lucia [REQ-F-001 AC1]
  Given la cuenta "lucia@club.es" con contraseña "Cumbre2026"
  When envía ese correo y contraseña en Iniciar sesión
  Then ve "Hola, lucia@club.es" en Material

Scenario: AC-001-02 — acceso de pablo [REQ-F-001 AC2]
  Given la cuenta "pablo@club.es" con contraseña "Hielo2026"
  When envía ese correo y contraseña
  Then ve "Hola, pablo@club.es"

Scenario: AC-001-03 — contraseña incorrecta [REQ-F-001 AC3]
  When lucia envía la contraseña "incorrecta"
  Then error `E_AUTH_INVALID`, ve "Correo o contraseña incorrectos" en Iniciar sesión
  And no hay sesión

Scenario: AC-001-04 — repetir el formulario [REQ-F-001 AC4]
  Given lucia acaba de entrar
  When envía otra vez el mismo formulario
  Then tiene una sola sesión y ve "Hola, lucia@club.es" en Material

Scenario: AC-001-05 — campos vacíos
  When envía correo y contraseña vacíos
  Then error `E_AUTH_INVALID` y no hay sesión

Scenario: AC-001-06 — hash con sal [REQ-NF-002 AC1]
  Given la cuenta "lucia@club.es" creada con "Cumbre2026" y otra cuenta con la misma contraseña
  When se lee el almacén
  Then lo guardado no contiene "Cumbre2026" y difiere entre las dos cuentas (INV-ACC-001)
