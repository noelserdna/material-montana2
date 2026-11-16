# Questions — requirements-engineer

<!-- Vista generada desde questions-requirements-engineer.json por `sdd questions`: no se edita a mano (se regenera). Pregunta nueva: `sdd questions add --role requirements-engineer`. -->

## Q-requirements-engineer-001 [ANSWERED] skill=sdd-requirements-engineer context=REQ-F-001 / N-002 tema=alta-de-socios
Question: The request does not say how member accounts and the manager/board roles are created. Draft assumes accounts and roles are created by the manager outside this version.
Para el cliente: ¿Cómo se dan de alta los socios y quién es encargado o junta?
Options:
  A) Accounts and roles are loaded by the manager (no self-signup screen in this version) (recommended)
     efecto: ninguno
     cliente: El encargado prepara las cuentas; no hay pantalla de registro
  B) Members sign up themselves with email and password; roles assigned by the manager
     efecto: req-change
     cliente: Cada socio se registra solo y el encargado asigna roles
Blocks: Gate de requisitos
Answer: A — opción recomendada, aceptada por Andres Leon (Cliente) (2026-11-16)

## Q-requirements-engineer-002 [ANSWERED] skill=sdd-requirements-engineer context=REQ-F-012 / N-008 tema=baja-con-reservas
Question: What happens to future reservations when retiring units leaves fewer active units than reserved on some day? Draft only prevents retiring more units than active, without cancelling reservations.
Para el cliente: Si se rompe una unidad que ya estaba reservada para otro día, ¿qué hacemos con esa reserva?
Options:
  A) Allow the retirement; the manager handles affected reservations by hand (recommended)
     efecto: ninguno
     cliente: Se da de baja y el encargado avisa a quien corresponda
  B) Refuse the retirement while it would leave reservations without a unit
     efecto: req-change
     cliente: No se deja dar de baja si dejaría una reserva sin material
Blocks: Gate de requisitos
Answer: A — opción recomendada, aceptada por Andres Leon (Cliente) (2026-11-16)
