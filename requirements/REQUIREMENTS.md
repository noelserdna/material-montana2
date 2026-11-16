# Requirements Document

> **Project:** material-montana2
> **Version:** 1.0
> **Last updated:** 2026-11-16
> **Status:** Review
> **Customer needs:** [CUSTOMER-NEEDS.md](CUSTOMER-NEEDS.md)

## Functional Requirements

### REQ-F-001: Entrar con correo y contraseña
- **Statement:** WHEN a member submits their email and password THE system SHALL start their session if the pair matches a registered account
- **Para el cliente:** Entras con tu correo y tu contraseña; si no coinciden, se te avisa y no entras.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-002
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Todo lo demás depende de saber quién es cada persona y qué rol tiene (socio, encargado, junta).
- **Acceptance criteria:**
  - GIVEN the account "lucia@club.es" with password "Cumbre2026" WHEN she submits that email and password on the login screen THEN el usuario ve "Hola, lucia@club.es" en la pantalla Material
    - Se ve: "Hola, {correo}" en Material
  - GIVEN the account "pablo@club.es" with password "Hielo2026" WHEN he submits that email and password THEN el usuario ve "Hola, pablo@club.es"
    - Se ve: "Hola, {correo}" en Material
  - GIVEN the account "lucia@club.es" WHEN she submits the password "incorrecta" THEN el usuario ve "Correo o contraseña incorrectos" and no session is started
    - Se ve: "Correo o contraseña incorrectos" en Iniciar sesión
  - GIVEN lucia has just logged in WHEN she submits the same login form again THEN she keeps a single session and sees "Hola, lucia@club.es"
    - Se ve: "Hola, {correo}" en Material
- **Dependencies:** None

### REQ-F-002: Reservar material para unas fechas
- **Statement:** WHEN a member without pending fines requests one unit of an item type for a date range THE system SHALL create the reservation if at least one unit of that type is free on every day of the range
- **Para el cliente:** Eliges un material y unas fechas y lo reservas, siempre que quede alguna unidad libre todos esos días.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-003
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Es la función central de la web.
- **Acceptance criteria:**
  - GIVEN 3 "Casco" units exist and none is reserved WHEN lucia reserves "Casco" from 2026-12-10 to 2026-12-12 THEN el usuario ve "Reserva creada: Casco del 2026-12-10 al 2026-12-12" en Mis reservas
    - Se ve: "Reserva creada: {material} del {desde} al {hasta}" en Mis reservas
  - GIVEN 3 "Casco" units, 2 already reserved 2026-12-10 to 2026-12-12 WHEN pablo reserves "Casco" from 2026-12-11 to 2026-12-13 THEN the reservation is created because 1 unit is free
    - Se ve: "Reserva creada: Casco del 2026-12-11 al 2026-12-13" en Mis reservas
  - GIVEN 3 "Casco" units, 3 already reserved 2026-12-10 to 2026-12-12 WHEN pablo reserves "Casco" from 2026-12-12 to 2026-12-14 THEN el usuario ve "No quedan unidades libres de Casco esas fechas" and no reservation is created
    - Se ve: "No quedan unidades libres de {material} esas fechas" en Material
  - GIVEN 3 "Casco" units, 3 reserved 2026-12-10 to 2026-12-12 WHEN a member reserves "Casco" from 2026-12-13 to 2026-12-14 THEN the reservation is created (days do not overlap)
    - Se ve: "Reserva creada: Casco del 2026-12-13 al 2026-12-14" en Mis reservas
  - GIVEN lucia just reserved "Casco" 2026-12-10 to 2026-12-12 WHEN the same form is submitted again THEN el usuario ve "Ya tienes esa reserva" and lucia still has one reservation of "Casco" for those dates
    - Se ve: "Ya tienes esa reserva" en Material
  - GIVEN a range whose end 2026-12-09 is before its start 2026-12-11 WHEN lucia submits it THEN el usuario ve "La fecha de fin no puede ser anterior a la de inicio" and nothing is reserved
    - Se ve: "La fecha de fin no puede ser anterior a la de inicio" en Material
- **Dependencies:** REQ-F-001, REQ-F-011

### REQ-F-003: Catálogo con los cinco tipos de material
- **Statement:** WHEN a logged-in member opens the material screen THE system SHALL list the item types (piolets, crampones, cascos, cuerdas, tiendas and any the manager adds) with their number of active units
- **Para el cliente:** Ves qué tipos de material hay y cuántas unidades tiene el club.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-003
- **Origin:** method
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Sin ver el catálogo no se puede elegir qué reservar.
- **Acceptance criteria:**
  - GIVEN the inventory has 4 "Piolet" and 3 "Casco" active units WHEN lucia opens Material THEN el usuario ve "Piolet" with "4 unidades" and "Casco" with "3 unidades"
    - Se ve: "{material}" y "{n} unidades" en Material
  - GIVEN the inventory has 1 "Tienda" active unit WHEN pablo opens Material THEN el usuario ve "Tienda" with "1 unidades"
    - Se ve: "{material}" y "{n} unidades" en Material
- **Dependencies:** REQ-F-001

### REQ-F-004: Ver mis reservas
- **Statement:** WHEN a member opens "Mis reservas" THE system SHALL list only that member's reservations with material, dates and state
- **Para el cliente:** Ves tus reservas con el material, las fechas y si siguen activas, entregadas, devueltas o canceladas.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-004
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** El socio necesita saber qué tiene reservado.
- **Acceptance criteria:**
  - GIVEN lucia has "Casco" 2026-12-10 to 2026-12-12 and pablo has "Cuerda" 2026-12-20 to 2026-12-21 WHEN lucia opens Mis reservas THEN el usuario ve "Casco" "2026-12-10" "2026-12-12" "Reservada" and does not see "Cuerda"
    - Se ve: "{material}" "{desde}" "{hasta}" "{estado}" en Mis reservas
  - GIVEN the same data WHEN pablo opens Mis reservas THEN el usuario ve "Cuerda" "2026-12-20" "2026-12-21" "Reservada" and does not see "Casco"
    - Se ve: "{material}" "{desde}" "{hasta}" "{estado}" en Mis reservas
  - GIVEN a member with no reservations WHEN she opens Mis reservas THEN el usuario ve "Todavía no tienes reservas"
    - Se ve: "Todavía no tienes reservas" en Mis reservas
- **Dependencies:** REQ-F-002

### REQ-F-005: Cancelar una reserva hasta el día antes
- **Statement:** WHEN a member cancels one of their reservations THE system SHALL cancel it and free the unit if today is before the reservation's start date
- **Para el cliente:** Puedes cancelar tu reserva hasta el día antes de que empiece; el material queda libre para otros.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-004
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Evita bloquear material que no se va a usar.
- **Acceptance criteria:**
  - GIVEN today is 2026-12-09 and lucia has "Casco" from 2026-12-10 WHEN she cancels it THEN el usuario ve "Reserva cancelada" and the reservation shows "Cancelada"
    - Se ve: "Reserva cancelada" en Mis reservas
  - GIVEN today is 2026-12-09 and 3 of 3 "Casco" are reserved on 2026-12-10, one of them lucia's WHEN she cancels it THEN pablo can reserve "Casco" for 2026-12-10
    - Se ve: "Reserva creada: Casco del 2026-12-10 al 2026-12-10" en Mis reservas
  - GIVEN today is 2026-12-10 and pablo has "Cuerda" from 2026-12-10 WHEN he tries to cancel it THEN el usuario ve "Ya no se puede cancelar: la reserva empieza hoy" and it stays "Reservada"
    - Se ve: "Ya no se puede cancelar: la reserva empieza hoy" en Mis reservas
  - GIVEN lucia cancelled her reservation WHEN she cancels it again THEN the reservation stays "Cancelada" and no other reservation changes
    - Se ve: "Cancelada" en Mis reservas
  - GIVEN lucia's reservation and pablo's WHEN lucia tries to cancel pablo's reservation THEN it is refused and pablo's stays "Reservada"
    - No visible: lo comprueba la API al rechazar la petición; la pantalla de Lucía no ofrece cancelar reservas ajenas
- **Dependencies:** REQ-F-002, REQ-F-004

### REQ-F-006: Entregar lo reservado
- **Statement:** WHEN the manager marks a reservation as delivered THE system SHALL change its state to "Entregada" and count the unit as lent
- **Para el cliente:** El encargado marca que ha entregado el material reservado.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-005
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Distingue lo reservado de lo que ya está en manos del socio.
- **Acceptance criteria:**
  - GIVEN lucia's "Casco" reservation 2026-12-10 to 2026-12-12 WHEN the manager marks it delivered THEN el usuario ve "Entregada a lucia@club.es" en Entregas y devoluciones
    - Se ve: "Entregada a {correo}" en Entregas y devoluciones
  - GIVEN pablo's "Cuerda" reservation WHEN the manager marks it delivered THEN el usuario ve "Entregada a pablo@club.es"
    - Se ve: "Entregada a {correo}" en Entregas y devoluciones
  - GIVEN a reservation already "Entregada" WHEN the manager marks it delivered again THEN it stays "Entregada" once and el usuario ve "Ya estaba entregada"
    - Se ve: "Ya estaba entregada" en Entregas y devoluciones
  - GIVEN a "Cancelada" reservation WHEN the manager tries to deliver it THEN el usuario ve "No se puede entregar una reserva cancelada"
    - Se ve: "No se puede entregar una reserva cancelada" en Entregas y devoluciones
  - GIVEN lucia logged in as a member WHEN she tries to mark a delivery THEN it is refused
    - No visible: lo comprueba la API al rechazar la petición; el socio no ve el botón de entregar
- **Dependencies:** REQ-F-002

### REQ-F-007: Marcar la devolución
- **Statement:** WHEN the manager marks a delivered reservation as returned THE system SHALL record the return date and free the unit
- **Para el cliente:** El encargado marca cuándo se devuelve el material y la unidad queda libre otra vez.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-005
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** La fecha real de devolución decide si hay multa.
- **Acceptance criteria:**
  - GIVEN lucia's "Casco" delivered, due 2026-12-12 WHEN the manager marks it returned on 2026-12-12 THEN el usuario ve "Devuelta el 2026-12-12 sin retraso" en Entregas y devoluciones
    - Se ve: "Devuelta el {fecha} sin retraso" en Entregas y devoluciones
  - GIVEN a delivered reservation not yet returned WHEN the manager marks it returned twice THEN there is a single return record and no second fine
    - Se ve: "Ya estaba devuelta" en Entregas y devoluciones
  - GIVEN a reservation that was never delivered WHEN the manager marks it returned THEN el usuario ve "Solo se puede devolver lo que se ha entregado"
    - Se ve: "Solo se puede devolver lo que se ha entregado" en Entregas y devoluciones
- **Dependencies:** REQ-F-006

### REQ-F-008: Multa de 2 euros por día de retraso
- **Statement:** WHEN a reservation is marked returned after its end date THE system SHALL create a fine of 2 euros per day of delay for that member
- **Para el cliente:** Si alguien devuelve tarde, se le pone una multa de 2 € por cada día de retraso.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-006
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Disuade los retrasos.
- **Acceptance criteria:**
  - GIVEN lucia's "Casco" due 2026-12-12 WHEN the manager marks it returned on 2026-12-15 THEN la multa es de 3 días × 2 € and el usuario ve "Multa de 6 € por 3 días de retraso" en Multas
    - Se ve: "Multa de {importe} € por {dias} días de retraso" en Multas
  - GIVEN pablo's "Cuerda" due 2026-12-21 WHEN the manager marks it returned on 2026-12-22 THEN el usuario ve "Multa de 2 € por 1 días de retraso"
    - Se ve: "Multa de {importe} € por {dias} días de retraso" en Multas
  - GIVEN a "Tienda" due 2026-12-20 WHEN it is returned on 2026-12-19 or 2026-12-20 THEN no fine is created and el usuario ve "Sin multas" en Multas
    - Se ve: "Sin multas" en Multas
  - GIVEN a fine already created for a return WHEN the same return is submitted again THEN the member still has one fine of 6 €
    - Se ve: "Multa de 6 € por 3 días de retraso" en Multas
- **Dependencies:** REQ-F-007

### REQ-F-009: Sin reservar mientras haya multa pendiente
- **Statement:** IF a member has an unpaid fine THEN THE system SHALL reject any new reservation by that member
- **Para el cliente:** Quien debe una multa no puede reservar más hasta pagarla.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-006
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Es la consecuencia que hace efectiva la multa.
- **Acceptance criteria:**
  - GIVEN lucia has an unpaid fine of 6 € WHEN she reserves "Piolet" from 2027-01-10 to 2027-01-11 THEN el usuario ve "No puedes reservar hasta pagar tu multa de 6 €" and nothing is reserved
    - Se ve: "No puedes reservar hasta pagar tu multa de {importe} €" en Material
  - GIVEN pablo has unpaid fines of 2 € and 4 € WHEN he reserves anything THEN el usuario ve "No puedes reservar hasta pagar tu multa de 6 €"
    - Se ve: "No puedes reservar hasta pagar tu multa de {importe} €" en Material
  - GIVEN lucia has a fine and marta has none WHEN marta reserves "Piolet" THEN it is created
    - Se ve: "Reserva creada: Piolet del 2027-01-10 al 2027-01-11" en Mis reservas
  - GIVEN lucia's fine and her existing reservations WHEN the fine is created THEN her earlier reservations stay "Reservada" (only new ones are blocked)
    - Se ve: "Reservada" en Mis reservas
- **Dependencies:** REQ-F-008, REQ-F-002

### REQ-F-010: Marcar la multa como pagada
- **Statement:** WHEN the manager marks a fine as paid THE system SHALL set it to "Pagada" and, if the member has no other unpaid fine, allow them to reserve again
- **Para el cliente:** El encargado marca la multa como pagada y el socio vuelve a poder reservar.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-007
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Cierra el ciclo de la multa.
- **Acceptance criteria:**
  - GIVEN lucia's only fine of 6 € WHEN the manager marks it paid THEN el usuario ve "Multa de 6 € pagada" en Multas and lucia can reserve "Piolet"
    - Se ve: "Multa de {importe} € pagada" en Multas
  - GIVEN pablo has fines of 2 € and 4 € WHEN the manager marks only the 2 € one paid THEN pablo's 4 € fine stays "Pendiente" and he still cannot reserve
    - Se ve: "Pendiente" en Multas
  - GIVEN a fine already "Pagada" WHEN the manager marks it paid again THEN it stays paid once and el usuario ve "Ya estaba pagada"
    - Se ve: "Ya estaba pagada" en Multas
  - GIVEN a member WHEN she tries to mark her own fine as paid THEN it is refused
    - No visible: lo comprueba la API al rechazar la petición; el socio no ve el botón de pagar
- **Dependencies:** REQ-F-008

### REQ-F-011: Añadir material al inventario
- **Statement:** WHEN the manager adds an item type with a number of units THE system SHALL add those units to the inventory
- **Para el cliente:** El encargado puede dar de alta material nuevo y decir cuántas unidades hay.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-008
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** El club compra material nuevo.
- **Acceptance criteria:**
  - GIVEN no "Mosquetón" exists WHEN the manager adds "Mosquetón" with 10 units THEN el usuario ve "Mosquetón" with "10 unidades" en Inventario
    - Se ve: "{material}" y "{n} unidades" en Inventario
  - GIVEN "Casco" has 3 units WHEN the manager adds 2 more "Casco" THEN el usuario ve "Casco" with "5 unidades"
    - Se ve: "{material}" y "{n} unidades" en Inventario
  - GIVEN the manager submits "Mosquetón" with 0 units THEN el usuario ve "Indica al menos 1 unidad" and nothing is added
    - Se ve: "Indica al menos 1 unidad" en Inventario
  - GIVEN the manager just added 2 "Casco" WHEN the same form is submitted again THEN 2 more units are added (a second addition is a deliberate new action) and el usuario ve "Casco" with "7 unidades"
    - Se ve: "{material}" y "{n} unidades" en Inventario
- **Dependencies:** REQ-F-001

### REQ-F-012: Dar de baja unidades rotas
- **Statement:** WHEN the manager retires units of an item type THE system SHALL remove them from the available units without touching reservations already delivered
- **Para el cliente:** El encargado da de baja las unidades rotas y dejan de poder reservarse.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-008
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Evita prometer material que no existe.
- **Acceptance criteria:**
  - GIVEN "Casco" has 5 active units WHEN the manager retires 1 THEN el usuario ve "Casco" with "4 unidades" en Inventario
    - Se ve: "{material}" y "{n} unidades" en Inventario
  - GIVEN "Cuerda" has 2 active units WHEN the manager retires 3 THEN el usuario ve "No hay tantas unidades activas de Cuerda" and the count stays 2
    - Se ve: "No hay tantas unidades activas de {material}" en Inventario
  - GIVEN "Piolet" has 1 active unit and it is retired WHEN a member reserves "Piolet" THEN el usuario ve "No quedan unidades libres de Piolet esas fechas"
    - Se ve: "No quedan unidades libres de {material} esas fechas" en Material
- **Dependencies:** REQ-F-011

### REQ-F-013: Ver lo prestado y las devoluciones de cada día
- **Statement:** WHEN the manager opens the daily screen THE system SHALL list the items currently lent and the items due back on the chosen day
- **Para el cliente:** El encargado ve qué material está prestado y qué tiene que volver cada día.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-009
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Organiza la entrega y recogida.
- **Acceptance criteria:**
  - GIVEN lucia has "Casco" delivered due 2026-12-12 and pablo "Cuerda" delivered due 2026-12-13 WHEN the manager opens Hoy THEN el usuario ve "Prestado: Casco (lucia@club.es)" and "Prestado: Cuerda (pablo@club.es)"
    - Se ve: "Prestado: {material} ({correo})" en Hoy
  - GIVEN the same data WHEN the manager chooses 2026-12-12 THEN el usuario ve "Vuelve: Casco (lucia@club.es)" and not the Cuerda
    - Se ve: "Vuelve: {material} ({correo})" en Hoy
  - GIVEN nothing is lent WHEN the manager opens Hoy THEN el usuario ve "No hay nada prestado"
    - Se ve: "No hay nada prestado" en Hoy
- **Dependencies:** REQ-F-006

### REQ-F-014: La junta consulta el historial de cualquier socio
- **Statement:** WHEN a board member selects a member THE system SHALL show that member's loan and fine history
- **Para el cliente:** La junta puede ver los préstamos y las multas de cualquier socio.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-010
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Control del uso del material.
- **Acceptance criteria:**
  - GIVEN lucia borrowed "Casco" 2026-12-10 to 2026-12-12 returned 2026-12-15 with a 6 € fine WHEN the board opens her history THEN el usuario ve "Casco" "2026-12-10" "2026-12-12" and "Multa de 6 € (Pendiente)" en Historial del socio
    - Se ve: "{material}" "{desde}" "{hasta}" y "Multa de {importe} € ({estado})" en Historial del socio
  - GIVEN pablo with one loan and no fines WHEN the board opens his history THEN el usuario ve "Cuerda" and "Sin multas"
    - Se ve: "Sin multas" en Historial del socio
- **Dependencies:** REQ-F-007, REQ-F-008

### REQ-F-015: La junta solo consulta
- **Statement:** THE system SHALL NOT allow board members to reserve, cancel, deliver, return, change inventory or mark fines
- **Para el cliente:** La junta solo mira: no puede cambiar reservas, material ni multas.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-010
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** El cliente pidió que la junta solo consulte.
- **Acceptance criteria:**
  - GIVEN a board member logged in WHEN she opens Historial del socio THEN el usuario no ve botones de reservar, cancelar, entregar, devolver, dar de baja ni pagar
    - Se ve: "Historial del socio" en Historial del socio
  - GIVEN a board member WHEN she sends a request to mark lucia's 6 € fine as paid THEN it is refused and the fine stays "Pendiente"
    - No visible: lo comprueba la API al rechazar la petición
  - GIVEN a board member WHEN she sends a request to reserve "Casco" THEN it is refused and no reservation is created
    - No visible: lo comprueba la API al rechazar la petición
- **Dependencies:** REQ-F-001

### REQ-F-016: Datos de los socios solo para su dueño, el encargado y la junta
- **Statement:** IF a member requests the personal data, reservations, loans or fines of another member THEN THE system SHALL refuse the request
- **Para el cliente:** Cada socio solo ve lo suyo; solo el encargado y la junta ven los datos de los demás.
- **Category:** Functional
- **Priority:** Must have
- **Needs:** N-011
- **Verification:** test
- **Source:** intent/INT-001.md
- **Rationale:** Son datos personales.
- **Acceptance criteria:**
  - GIVEN lucia and pablo are members WHEN lucia opens the history address of pablo THEN el usuario ve "No tienes permiso para ver estos datos" and no data of pablo
    - Se ve: "No tienes permiso para ver estos datos" en Historial del socio
  - GIVEN the same WHEN pablo opens lucia's history address THEN el usuario ve "No tienes permiso para ver estos datos"
    - Se ve: "No tienes permiso para ver estos datos" en Historial del socio
  - GIVEN lucia has a 6 € fine WHEN the manager opens the Multas screen THEN el usuario ve "lucia@club.es" with "Multa de 6 €"
    - Se ve: "{correo}" y "Multa de {importe} €" en Multas
  - GIVEN no one is logged in WHEN the Material address is requested THEN the user is sent to the login screen and no data is returned
    - Se ve: "Iniciar sesión" en Iniciar sesión
- **Dependencies:** REQ-F-001

## Nonfunctional Requirements

### REQ-NF-001: Datos compartidos entre dispositivos
- **Statement:** THE system SHALL keep reservations, inventory, fines and accounts in a server store read and written by every user, whatever their device
- **Para el cliente:** Lo que reserva un socio desde su móvil lo ve el encargado desde su ordenador al momento.
- **Category:** Availability
- **Priority:** Must have
- **Needs:** N-001, N-005
- **Origin:** method
- **Verification:** test
- **Source:** intent/INT-001.md (varios roles desde dispositivos distintos)
- **Rationale:** Socio, encargado y junta usan la web cada uno desde su dispositivo y comparten los mismos datos.
- **Metric:** a reservation created in one session is returned to another session's next request
- **Acceptance criteria:**
  - GIVEN lucia reserved "Casco" 2026-12-10 to 2026-12-12 in one session WHEN the manager opens Entregas y devoluciones in another session THEN el usuario ve "Casco" "lucia@club.es"
    - Se ve: "{material}" "{correo}" en Entregas y devoluciones
- **Dependencies:** None

### REQ-NF-002: Contraseñas no guardadas en claro
- **Statement:** THE system SHALL store account passwords only as salted hashes
- **Para el cliente:** Nadie puede leer las contraseñas de los socios, ni siquiera quien administre los datos.
- **Category:** Security
- **Priority:** Must have
- **Needs:** N-002, N-011
- **Origin:** method
- **Verification:** test
- **Source:** método (protección de datos personales)
- **Rationale:** Evita filtrar contraseñas.
- **Metric:** stored value of "Cumbre2026" differs from the text and verifies with the hash function
- **Acceptance criteria:**
  - GIVEN the account "lucia@club.es" created with password "Cumbre2026" WHEN the stored record is read THEN it does not contain "Cumbre2026" and two accounts with the same password have different stored values
    - No visible: lo comprueba el test leyendo el almacén de datos
- **Dependencies:** REQ-F-001

## Constraints

### REQ-C-001: Textos de la interfaz en español
- **Statement:** All texts shown to users SHALL be in Spanish
- **Para el cliente:** Todo lo que ves en la web está en español.
- **Type:** Business
- **Source:** AGENTS.md (Convenciones)
- **Needs:** — (team/architecture source)
- **Verification:** test — los textos de la interfaz salen de un único módulo de textos y el test comprueba que están en español (sin cadenas inglesas de la lista de mensajes)

### REQ-C-002: Stack JavaScript con Vite y Vitest
- **Statement:** The project SHALL be JavaScript (ES modules) built with Vite and tested with Vitest using at most 2 workers
- **Para el cliente:** La web se construye con herramientas web estándar y sus pruebas se ejecutan con un solo comando.
- **Type:** Technical
- **Source:** AGENTS.md (SDD Stack Profile)
- **Needs:** — (team/architecture source)
- **Verification:** test — package.json declara "type": "module", vite y vitest, y vitest.config tiene maxWorkers: 2

### REQ-C-003: Fechas por día y multa fija de 2 euros
- **Statement:** The system SHALL count reservations and delays in whole calendar days, and the fine rate SHALL be 2 euros per day
- **Para el cliente:** Las fechas se cuentan por días completos y cada día de retraso cuesta 2 €.
- **Type:** Business
- **Source:** intent/INT-001.md
- **Needs:** N-006
- **Verification:** test — una constante única de tarifa (2) y el cálculo de días sin horas

## Traceability

| REQ ID | Type | Priority | Source | Needs | Verification | Dependencies |
|--------|------|----------|--------|-------|--------------|--------------|
| REQ-F-001 | Functional | Must | intent/INT-001.md | N-002 | test | None |
| REQ-F-002 | Functional | Must | intent/INT-001.md | N-003 | test | REQ-F-001, REQ-F-011 |
| REQ-F-003 | Functional | Must | intent/INT-001.md | N-003 | test | REQ-F-001 |
| REQ-F-004 | Functional | Must | intent/INT-001.md | N-004 | test | REQ-F-002 |
| REQ-F-005 | Functional | Must | intent/INT-001.md | N-004 | test | REQ-F-002, REQ-F-004 |
| REQ-F-006 | Functional | Must | intent/INT-001.md | N-005 | test | REQ-F-002 |
| REQ-F-007 | Functional | Must | intent/INT-001.md | N-005 | test | REQ-F-006 |
| REQ-F-008 | Functional | Must | intent/INT-001.md | N-006 | test | REQ-F-007 |
| REQ-F-009 | Functional | Must | intent/INT-001.md | N-006 | test | REQ-F-008, REQ-F-002 |
| REQ-F-010 | Functional | Must | intent/INT-001.md | N-007 | test | REQ-F-008 |
| REQ-F-011 | Functional | Must | intent/INT-001.md | N-008 | test | REQ-F-001 |
| REQ-F-012 | Functional | Must | intent/INT-001.md | N-008 | test | REQ-F-011 |
| REQ-F-013 | Functional | Must | intent/INT-001.md | N-009 | test | REQ-F-006 |
| REQ-F-014 | Functional | Must | intent/INT-001.md | N-010 | test | REQ-F-007, REQ-F-008 |
| REQ-F-015 | Functional | Must | intent/INT-001.md | N-010 | test | REQ-F-001 |
| REQ-F-016 | Functional | Must | intent/INT-001.md | N-011 | test | REQ-F-001 |
| REQ-NF-001 | Nonfunctional | Must | intent/INT-001.md | N-001, N-005 | test | None |
| REQ-NF-002 | Nonfunctional | Must | método | N-002, N-011 | test | REQ-F-001 |
| REQ-C-001 | Constraint | — | AGENTS.md | — | test | — |
| REQ-C-002 | Constraint | — | AGENTS.md | — | test | — |
| REQ-C-003 | Constraint | — | intent/INT-001.md | N-006 | test | — |
