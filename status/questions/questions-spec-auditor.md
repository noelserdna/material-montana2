# Questions — spec-auditor

<!-- Vista generada desde questions-spec-auditor.json por `sdd questions`: no se edita a mano (se regenera). Pregunta nueva: `sdd questions add --role spec-auditor`. -->

## Q-spec-auditor-001 [OPEN] skill=sdd-spec-auditor context=SIL-001 · spec/use-cases/UC-005:35, UC-006:33, domain/04-STATES.md:12 tema=estados-entregar-cancelar
Question: No se especifica qué ocurre al cancelar una reserva Entregada (posible si se entrega antes de `desde`, hoy < desde) ni al entregar una reserva Devuelta. INV-RES-004 solo exige hoy < desde; E_RES_CANCELLED solo cubre Cancelada. Defecto escrito: cancelar solo desde Reservada.
Para el cliente: ¿Una reserva que ya se ha entregado puede cancelarse antes de su fecha de inicio, y se puede entregar una reserva que ya fue devuelta?
Options:
  A) Solo se cancela lo Reservada y solo se entrega lo Reservada; en cualquier otro estado se rechaza con un error de estado nuevo y la reserva no cambia (recommended)
     efecto: ninguno
     cliente: No: cancelar y entregar solo valen para reservas que aún no se han entregado; en otro caso se avisa y no cambia nada
  B) Se permite cancelar una Entregada antes de `desde`, liberando la unidad
     efecto: req-change
     cliente: Sí, se puede cancelar aunque ya se haya entregado
Blocks: SIL-001 (UC-005, UC-006, SM-001, catálogo de errores)
Answer:

## Q-spec-auditor-002 [OPEN] skill=sdd-spec-auditor context=SIL-002 · spec/domain/05-INVARIANTS.md:13, use-cases/UC-007:15, UC-013:15 tema=fechas-pasadas-y-devolucion
Question: No se especifica: (1) si se puede reservar un rango que empieza antes de hoy; (2) qué día vale como `devueltaEl` (¿lo teclea el encargado? ¿puede ser anterior a `desde` o futuro?); (3) qué es «hoy» (fecha del servidor, zona horaria). Defecto escrito: sin restricciones salvo hasta ≥ desde.
Para el cliente: ¿Se puede reservar o devolver con fechas pasadas o futuras raras (por ejemplo reservar para ayer, o marcar una devolución con una fecha anterior a la reserva)?
Options:
  A) Sin restricciones nuevas: el encargado indica la fecha de devolución; el día de hoy es la fecha del servidor (recommended)
     efecto: ninguno
     cliente: Se deja como está: no se añaden más comprobaciones de fechas
  B) Rechazar reservas que empiecen antes de hoy y devoluciones con fecha anterior al inicio o futura, con errores nuevos
     efecto: req-change
     cliente: Sí, no se aceptan esas fechas
Blocks: SIL-002 (INV-RES-002, UC-002, UC-007)
Answer:

## Q-spec-auditor-003 [OPEN] skill=sdd-spec-auditor context=AMB-001 · spec/use-cases/UC-014:16, REQ-F-014 tema=historial-que-es-prestamo
Question: UC-014 lista `prestamos` pero no define qué reservas cuentan: ¿Entregada y Devuelta, o también Reservada y Cancelada? «Préstamo» no está en el glosario. Defecto escrito: solo Entregada y Devuelta.
Para el cliente: En el historial de un socio, ¿salen solo los materiales que de verdad se le entregaron, o también sus reservas pendientes y canceladas?
Options:
  A) Solo reservas Entregada y Devuelta (lo realmente prestado); las demás no salen (recommended)
     efecto: ninguno
     cliente: Solo lo que se le entregó
  B) Todas las reservas del socio con su estado
     efecto: req-change
     cliente: Todas sus reservas, también pendientes y canceladas
Blocks: AMB-001 (UC-014, glosario)
Answer:
