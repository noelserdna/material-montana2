# BDD-UC-001 — Entrar con correo y contraseña

> Refs: UC-001; REQ-F-001, REQ-F-017, REQ-NF-002; INV-ACC-001, INV-ACC-002, INV-ACC-006

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

Scenario: AC-001-07 — bloqueo tras 5 fallos [REQ-F-017 AC1]
  Given la cuenta "lucia@club.es" con contraseña "Cumbre2026" y 5 intentos con "incorrecta" entre las 10:00 y las 10:10
  When envía "Cumbre2026" a las 10:11
  Then error `E_AUTH_LOCKED`, ve "Demasiados intentos fallidos: espera 15 minutos" en Iniciar sesión
  And no hay sesión

Scenario: AC-001-08 — fin del bloqueo [REQ-F-017 AC2]
  Given la cuenta bloqueada tras el 5.º fallo de las 10:10
  When envía "Cumbre2026" a las 10:26
  Then ve "Hola, lucia@club.es" en Material

Scenario: AC-001-09 — 4 fallos no bloquean y un acierto pone a cero [REQ-F-017 AC3]
  Given 4 intentos fallidos de "lucia@club.es" en 15 minutos
  When envía "Cumbre2026"
  Then ve "Hola, lucia@club.es" y sus fallos vuelven a contar desde cero

Scenario: AC-001-10 — el bloqueo es por cuenta [REQ-F-017 AC4]
  Given "lucia@club.es" bloqueada
  When "pablo@club.es" envía "Hielo2026"
  Then ve "Hola, pablo@club.es" en Material

Scenario: AC-001-11 — fallos fuera de la ventana [REQ-F-017 AC5]
  Given el primero de 5 fallos a las 10:00 y el último a las 10:16
  When envía "Cumbre2026" a las 10:17
  Then ve "Hola, lucia@club.es" en Material

Scenario: AC-001-12 — probar durante el bloqueo no lo alarga (Q-spec-auditor-004)
  Given la cuenta bloqueada tras el 5.º fallo de las 10:10 y un intento erróneo a las 10:20
  When envía "Cumbre2026" a las 10:26
  Then ve "Hola, lucia@club.es" en Material
