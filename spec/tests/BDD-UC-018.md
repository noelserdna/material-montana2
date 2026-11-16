# BDD-UC-018 — La sesión se cierra sola tras 30 minutos sin actividad

> Refs: UC-018; REQ-F-019; INV-ACC-007

Scenario: AC-018-01 — sesión caducada [REQ-F-019 AC1]
  Given la última actividad de lucia fue a las 10:00
  When abre Material a las 10:31
  Then error `E_SESSION_EXPIRED`, ve "Tu sesión ha caducado" en Iniciar sesión y no se devuelven datos

Scenario: AC-018-05 — otro socio, mismo mensaje [REQ-F-019 AC2]
  Given la última actividad de pablo fue a las 09:00
  When abre Material a las 09:45
  Then ve "Tu sesión ha caducado" en Iniciar sesión

Scenario: AC-018-02 — dentro del plazo [REQ-F-019 AC2]
  Given la última actividad de lucia fue a las 10:00
  When abre Material a las 10:29
  Then ve "Hola, lucia@club.es"

Scenario: AC-018-03 — la actividad renueva el plazo [REQ-F-019 AC3]
  Given lucia estuvo activa a las 10:00, 10:20 y 10:40
  When abre Material a las 10:55
  Then ve "Hola, lucia@club.es"

Scenario: AC-018-04 — entrar de nuevo [REQ-F-019 AC4]
  Given la sesión de lucia caducó
  When entra con "Cumbre2026"
  Then ve "Hola, lucia@club.es"
