# BDD-UC-015 — La junta solo consulta

> Refs: UC-015; REQ-F-015; INV-ACC-005

Scenario: AC-015-01 — sin botones [REQ-F-015 AC1]
  Given una persona de la junta con sesión
  When abre Historial del socio
  Then ve "Historial del socio" y ningún botón de reservar, cancelar, entregar, devolver, dar de baja ni pagar

Scenario: AC-015-02 — pagar multa [REQ-F-015 AC2]
  When la junta pide marcar pagada la multa de 6 € de lucia
  Then error `E_FORBIDDEN_ROLE` y la multa sigue "Pendiente"

Scenario: AC-015-03 — reservar [REQ-F-015 AC3]
  When la junta pide reservar "Casco"
  Then error `E_FORBIDDEN_ROLE` y no se crea reserva

Scenario: AC-015-04 — repetir
  When la junta repite la petición
  Then error `E_FORBIDDEN_ROLE` otra vez
