# BDD-UC-012 — Dar de baja unidades

> Refs: UC-012; REQ-F-012; INV-INV-002; RN-002

Scenario: AC-012-01 — baja normal [REQ-F-012 AC1]
  Given "Casco" con 5 unidades activas
  When el encargado da de baja 1
  Then ve "Casco" con "4 unidades" en Inventario

Scenario: AC-012-02 — más de las activas [REQ-F-012 AC2]
  Given "Cuerda" con 2 unidades activas
  When el encargado da de baja 3
  Then error `E_UNITS_INSUFFICIENT`, ve "No hay tantas unidades activas de Cuerda" en Inventario y la cuenta sigue en 2

Scenario: AC-012-03 — sin unidades no se reserva [REQ-F-012 AC3]
  Given "Piolet" con 1 unidad activa dada de baja
  When un socio reserva "Piolet"
  Then error `E_NO_STOCK`, ve "No quedan unidades libres de Piolet esas fechas" en Material

Scenario: AC-012-04 — cero unidades
  When el encargado da de baja 0 unidades
  Then error `E_UNITS_MIN`

Scenario: AC-012-05 — material inexistente
  When el encargado da de baja un material que no existe
  Then error `E_MATERIAL_NOT_FOUND`

Scenario: AC-012-06 — socio no da de baja
  When un socio intenta dar de baja unidades
  Then error `E_FORBIDDEN_ROLE`

Scenario: AC-012-07 — reserva futura se mantiene [RN-002]
  Given una unidad con reserva futura y esa unidad es la única activa
  When el encargado la da de baja
  Then la baja se hace y la reserva sigue "Reservada"

Scenario: AC-012-08 — baja repetida
  Given "Casco" con 1 unidad activa dada de baja
  When se repite la misma baja
  Then error `E_UNITS_INSUFFICIENT`
