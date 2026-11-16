# BDD-UC-011 — Añadir material

> Refs: UC-011; REQ-F-011; INV-INV-001

Scenario: AC-011-01 — material nuevo [REQ-F-011 AC1]
  Given no existe "Mosquetón"
  When el encargado añade "Mosquetón" con 10 unidades
  Then ve "Mosquetón" con "10 unidades" en Inventario

Scenario: AC-011-02 — sumar a existente [REQ-F-011 AC2]
  Given "Casco" con 3 unidades
  When el encargado añade 2 más
  Then ve "Casco" con "5 unidades"

Scenario: AC-011-03 — cero unidades [REQ-F-011 AC3]
  When el encargado envía "Mosquetón" con 0 unidades
  Then error `E_UNITS_MIN`, ve "Indica al menos 1 unidad" en Inventario y no se añade nada

Scenario: AC-011-04 — mismo formulario otra vez [REQ-F-011 AC4]
  Given el encargado acaba de añadir 2 "Casco" (5 unidades)
  When envía otra vez el mismo formulario
  Then se añaden 2 más y ve "Casco" con "7 unidades"

Scenario: AC-011-05 — socio no añade [REQ-F-015]
  When un socio o la junta intenta añadir material
  Then error `E_FORBIDDEN_ROLE`
