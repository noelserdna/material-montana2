# Questions — task-generator

<!-- Vista generada desde questions-task-generator.json por `sdd questions`: no se edita a mano (se regenera). Pregunta nueva: `sdd questions add --role task-generator`. -->

## Q-task-generator-001 [ANSWERED] skill=sdd-task-generator context=FASE-2 / UC-006, UC-007, REQ-NF-001 AC1 (PLAN-FASE-2 §5) tema=listado-reservas-encargado
Question: FASE-2 puts a table of every member's reservations on #/entregas (SCR-004, REQ-NF-001 AC1) but the 14 contracted operations have none that lists all reservations: API-002-02 listarMisReservas returns only the actor's (INV-ACC-004) and API-002-06 verHoy is FASE-4 and per day. Default taken in the task file: [PLAN GAP] task TASK-F2-005 and dependants stay blocked.
Para el cliente: ¿Cómo ve el encargado en Entregas y devoluciones las reservas de todos los socios?
Options:
  A) Add a read-only operation listarReservas (encargado only, filter Por entregar / Entregadas / Todas) and its contract (recommended)
     efecto: req-change
     cliente: Una lista nueva, solo para el encargado, con el filtro Por entregar, Entregadas o Todas
  B) Make listarMisReservas return every reservation when the actor is the encargado
     efecto: req-change
     cliente: La misma lista de reservas, pero el encargado ve las de todos
  C) Move the listing to Hoy (FASE-4) and keep Entregas with a reservation-id field only
     efecto: req-change
     cliente: El encargado entrega buscando la reserva en la pantalla Hoy
Blocks: TASK-F2-005, TASK-F2-006, TASK-F2-007
Answer: A — opción recomendada, aceptada por Jev (jev-1.14.0) (Agente automático (modo auto)) (2026-11-16)
