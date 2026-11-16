# E2E Acceptance Scenarios

> **Project:** material-montana2 · **Type:** WEB-APP · **Generated from:** spec/workflows/WF-001, spec/use-cases/, spec/contracts/, requirements/ (`Se ve:`)
> **UX enrichment:** No — sin `ux/`; localizadores por rol y etiqueta (D-T-005). No hay wireframes: V-FIELD-03/04/05 no aplican.

## E2E Strategy

| Dimension | Value |
|-----------|-------|
| Framework | Playwright (Chromium) contra `npx vite --host 127.0.0.1 --port 5173` |
| Selector strategy | getByRole > getByLabel > getByText > getByTestId |
| Auth strategy | 1 test de login (S0); el resto reutiliza `storageState` por cuenta (lucia, pablo, marta, ana) |
| Data strategy | almacén propio por test, cuentas sembradas (D-T-003); reloj fijado (D-T-001) |
| Accessibility | axe-core en cada pantalla de S1 y S2 |
| Visual evidence | por criterio: `evidencias/FASE-{N}/{AC id}.png` (fila `C-NNN-NN` = su AC `AC-NNN-NN`); vídeo del WF: `evidencias/FASE-{N}/WF-001-ciclo-de-reserva.webm` |
| Target environment | local para PR; `smoke-deploy`: none (sin `staging_url`) |

| Tier | Scenarios | Trigger |
|------|-----------|---------|
| Smoke | S0, S1, S2 (P0) | every PR |
| Critical | + filas `C-*` y variaciones (P1) | merge a main |
| Full | + lista F (P2) | nightly |

Viewport: móvil 375 px para Smoke y Critical; escritorio 1280 px para todo.

## Field Inventory: WF-001 y pantallas de los 16 REQ-F

| Field | UC input | Operation input | Required | Type | Validation | Conditional |
|-------|----------|-----------------|----------|------|-----------|-------------|
| Correo | UC-001 | API-001-01.correo | Yes | Correo (VO-001) | no vacío; coincide con la cuenta | No |
| Contraseña | UC-001 | API-001-01.contrasena | Yes | string | no vacía; coincide con el hash | No |
| Material | UC-002 | API-002-01.material | Yes | string | existente (`E_MATERIAL_NOT_FOUND`) | No |
| Desde / Hasta | UC-002 | API-002-01.desde/hasta | Yes | Dia (VO-003) | `hasta ≥ desde` (`E_DATE_RANGE`) | No |
| Devuelta el | UC-007 | API-002-05.devueltaEl | Yes | Dia | día natural; retraso → multa | multa si `> hasta` |
| Material, Unidades (Inventario) | UC-011, UC-012 | API-003-02/03 | Yes | string, Unidades (VO-006) | `n ≥ 1` (`E_UNITS_MIN`); baja `n ≤ activas` | No |
| Día (Hoy) | UC-013 | API-002-06.dia | No | Dia | por defecto hoy | No |
| Socio (Historial) | UC-014 | API-004-03.correo | Yes | Correo | solo junta/encargado | No |
| reservaId, multaId | UC-005..007, 010 | API-002-03..05, 004-02 | Yes | botón de fila | existente | No |

### Field Behavioral Matrix

| Field | VALID | EMPTY | INVALID | BOUNDARY | CONDITIONAL |
|-------|-------|-------|---------|----------|-------------|
| Correo / Contraseña | AC-001-01 | AC-001-05 `E_AUTH_INVALID` | AC-001-03 | — | — |
| Material | AC-002-01 | `E_MATERIAL_NOT_FOUND` (AC-002-09) | AC-002-09 | — | — |
| Desde / Hasta | AC-002-01, 04 | no definido en spec (HUECO: sin AC de vacío) | AC-002-06 | `hasta = desde` AC-005-02; matriz UC-002 | — |
| Devuelta el | AC-007-01 | no definido en spec | fecha pasada/futura: GAP-003 | `hasta` y `hasta+1`: AC-008-02, 03 | retraso → multa AC-008-01 |
| Unidades | AC-011-01, 012-01 | AC-011-03 | AC-012-02, 012-04 | 0, 1, activas (matriz UC-012) | — |
| Día / Socio | AC-013-02 / AC-014-01 | día por defecto AC-013-01 / — | — | — | — |

## Scenarios

> Fixtures: `stock1` = Casco con 1 unidad · `stock2` = Casco con 2 · `res(lucia)` = reserva Casco 2026-12-10..2026-12-12 de lucia · `tarde` = res devuelta 2026-12-15 (3 días) · hoy = 2026-12-01 (D-T-001) · `reloj` = hora del día fijada y avanzable (D-T-011): `10:00`, `+30 min`…

### E2E-WF-001-S0: Entrar (P0, smoke)

- **Acceptance:** AC-001-01 (test title: `E2E-WF-001-S0 AC-001-01 AC-001-03 entrar`) · **Requirements:** REQ-F-001 · **Fields:** Correo, Contraseña (2/2)

| # | Action | Target | Assertion | Evidence | Spec Ref |
|---|--------|--------|-----------|----------|----------|
| 1 | Abrir `/` sin sesión | — | heading "Iniciar sesión" | — | INV-ACC-003 |
| 2 | Rellenar Correo "lucia@club.es" y Contraseña de la cuenta y enviar | formulario | texto "Hola, lucia@club.es" en Material | AC-001-01.png | UC-001 §main |
| 3 | Salir, entrar con contraseña errónea | formulario | texto "Correo o contraseña incorrectos" en Iniciar sesión | AC-001-03.png | UC-001 E1 |

### E2E-WF-001-S1: Ciclo en plazo (P0, smoke)

- **Workflow:** WF-001 pasos 1-3 · **UC:** UC-002, UC-004, UC-006, UC-007 · **Acceptance:** AC-002-01, AC-004-01, AC-006-01, AC-007-01 (title: `E2E-WF-001-S1 AC-002-01 AC-004-01 AC-006-01 AC-007-01 ciclo en plazo`) · **REQ:** REQ-F-002, 004, 006, 007 · **Video:** `WF-001-ciclo-de-reserva.webm` · **Fields:** 7/7

| # | Action | Target | Assertion | Evidence | Spec Ref |
|---|--------|--------|-----------|----------|----------|
| 1 | lucia entra y abre Material | Material | "Casco" y "2 unidades" | — | UC-003 |
| 2 | Elegir Casco, Desde 2026-12-10, Hasta 2026-12-12, reservar | formulario | "Reserva creada: Casco del 2026-12-10 al 2026-12-12" en Mis reservas | AC-002-01.png | UC-002 §main.5 |
| 3 | Abrir Mis reservas | Mis reservas | "Casco" "2026-12-10" "2026-12-12" "Reservada" | AC-004-01.png | UC-004 |
| 4 | marta entra, abre Entregas y devoluciones, pulsa entregar | fila | "Entregada a lucia@club.es" | AC-006-01.png | UC-006 §main.3 |
| 5 | Devuelta el 2026-12-12, marcar devuelta | fila | "Devuelta el 2026-12-12 sin retraso" | AC-007-01.png | UC-007 §main.4 |
| 6 | Abrir Multas | Multas | "Sin multas" | AC-008-03.png | UC-008 E1 |

### E2E-WF-001-S2: Ciclo con retraso, multa, bloqueo y pago (P0, smoke)

- **Workflow:** WF-001 pasos 1-5 · **UC:** UC-002, 007, 008, 009, 010 · **Acceptance:** AC-008-01, AC-008-04, AC-009-01, AC-010-01, AC-009-03 (title: `E2E-WF-001-S2 AC-008-04 AC-009-01 AC-010-01 AC-009-03 ciclo con multa`) · **REQ:** REQ-F-008..010 · **Video:** mismo WF-001 (segunda parte)

| # | Action | Target | Assertion | Evidence | Spec Ref |
|---|--------|--------|-----------|----------|----------|
| 1 | `res(lucia)` entregada; marta devuelve con Devuelta el 2026-12-15 | fila | devolución aceptada | — | UC-007 |
| 2 | Abrir Multas | Multas | "Multa de 6 € por 3 días de retraso" | AC-008-04.png | UC-008 §main.3 |
| 3 | lucia intenta reservar Piolet 2027-01-10..2027-01-11 | Material | "No puedes reservar hasta pagar tu multa de 6 €" | AC-009-01.png | UC-009 E1 |
| 4 | marta marca pagada | Multas | "Multa de 6 € pagada" | AC-010-01.png | UC-010 §main.3 |
| 5 | lucia reserva de nuevo | Mis reservas | "Reserva creada: Piolet del 2027-01-10 al 2027-01-11" | AC-009-03.png | INV-FIN-003 |

### Critical: un escenario por criterio visible (P1)

> Cada fila es un test `E2E-WF-001-C-NNN-NN AC-NNN-NN …`: entra por Iniciar sesión, navega a la pantalla y guarda `{AC}.png` tras aserir el texto. `(S0/S1/S2)` = captura hecha dentro de ese recorrido. `{x}` = dato del ejemplo.

| Row | Given → route | Assertion (texto en pantalla) |
|-----|---------------|-------------------------------|
| C-001-01 (S0) | lucia, Iniciar sesión | "Hola, lucia@club.es" en Material |
| C-001-02 | pablo, Iniciar sesión | "Hola, pablo@club.es" en Material |
| C-001-03 (S0) | contraseña errónea | "Correo o contraseña incorrectos" en Iniciar sesión |
| C-001-04 | lucia con sesión envía otra vez | "Hola, lucia@club.es" en Material |
| C-001-05 | `reloj` 10:11, lucia con 5 fallos 10:00–10:10, envía la contraseña correcta | "Demasiados intentos fallidos: espera 15 minutos" en Iniciar sesión (AC-001-07) |
| C-001-06 | lo mismo, `reloj` 10:26 | "Hola, lucia@club.es" en Material (AC-001-08) |
| C-001-07 | lucia con 4 fallos, envía la correcta | "Hola, lucia@club.es" en Material (AC-001-09) |
| C-001-08 | lucia bloqueada; pablo entra | "Hola, pablo@club.es" en Material (AC-001-10) |
| C-001-09 | 5 fallos, 1.º a 10:00 y 5.º a 10:16; envía a 10:17 | "Hola, lucia@club.es" en Material (AC-001-11) |
| C-002-01 (S1) | `stock2`, lucia reserva | "Reserva creada: Casco del 2026-12-10 al 2026-12-12" en Mis reservas |
| C-002-02 | `stock2` con una ocupada parte del rango | "Reserva creada: Casco del 2026-12-11 al 2026-12-13" en Mis reservas |
| C-002-03 | `stock1` ocupada 12-10..12-12; pablo pide 12-11..12-13 | "No quedan unidades libres de Casco esas fechas" en Material |
| C-002-04 | `stock1` ocupada hasta 12-12; pablo pide 12-13..12-14 | "Reserva creada: Casco del 2026-12-13 al 2026-12-14" en Mis reservas |
| C-002-05 | `res(lucia)`; lucia repite | "Ya tienes esa reserva" en Material |
| C-002-06 | Desde 12-12, Hasta 12-10 | "La fecha de fin no puede ser anterior a la de inicio" en Material |
| C-003-01 | Material con Casco, Cuerda, Piolet | "{material}" y "{n} unidades" por cada uno en Material |
| C-003-02 | Tienda con 1 unidad | "Tienda" y "1 unidades" (literal del spec «{n} unidades») en Material |
| C-004-01 (S1) | lucia con `res(lucia)` | "Casco" "2026-12-10" "2026-12-12" "Reservada" en Mis reservas |
| C-004-02 | pablo con su reserva | su material, fechas y estado en Mis reservas |
| C-004-03 | socio sin reservas | "Todavía no tienes reservas" en Mis reservas |
| C-005-01 | `res(lucia)`, hoy 12-01, cancelar | "Reserva cancelada" en Mis reservas |
| C-005-02 | `stock1`; lucia cancela; pablo reserva 12-10..12-10 | "Reserva creada: Casco del 2026-12-10 al 2026-12-10" en Mis reservas |
| C-005-03 | `res(lucia)`, hoy = 12-10, cancelar | "Ya no se puede cancelar: la reserva empieza hoy" en Mis reservas |
| C-005-04 | lucia cancela dos veces | "Cancelada" en Mis reservas |
| C-006-01 (S1) | marta, `res(lucia)` | "Entregada a lucia@club.es" en Entregas y devoluciones |
| C-006-02 | marta, reserva de pablo | "Entregada a pablo@club.es" en Entregas y devoluciones |
| C-006-03 | ya Entregada, entregar otra vez | "Ya estaba entregada" en Entregas y devoluciones |
| C-006-04 | reserva Cancelada | "No se puede entregar una reserva cancelada" en Entregas y devoluciones |
| C-007-01 (S1) | Entregada, Devuelta el 2026-12-12 | "Devuelta el 2026-12-12 sin retraso" en Entregas y devoluciones |
| C-007-02 | ya Devuelta, devolver otra vez | "Ya estaba devuelta" en Entregas y devoluciones |
| C-007-03 | Reservada sin entregar, devolver | "Solo se puede devolver lo que se ha entregado" en Entregas y devoluciones |
| C-008-01 | `tarde` (3 días) | "Multa de 6 € por 3 días de retraso" en Multas |
| C-008-02 | devuelta 1 día tarde | "Multa de 2 € por 1 días de retraso" en Multas |
| C-008-03 (S1) | devuelta en `hasta` | "Sin multas" en Multas |
| C-008-04 (S2) | `tarde`, devolver otra vez | "Multa de 6 € por 3 días de retraso" en Multas (una sola) |
| C-009-01 (S2) | lucia con multa de 6 € | "No puedes reservar hasta pagar tu multa de 6 €" en Material |
| C-009-02 | lucia con multas de 2 € y 4 € | "No puedes reservar hasta pagar tu multa de 6 €" en Material |
| C-009-03 (S2) | pablo sin multa | "Reserva creada: Piolet del 2027-01-10 al 2027-01-11" en Mis reservas |
| C-009-04 | lucia con `res(lucia)` y multa nueva | "Reservada" en Mis reservas |
| C-010-01 (S2) | marta, multa de 6 € | "Multa de 6 € pagada" en Multas |
| C-010-02 | dos multas, pagar una | "Pendiente" (la otra) en Multas |
| C-010-03 | ya Pagada, pagar otra vez | "Ya estaba pagada" en Multas |
| C-011-01 | marta añade "Mosquetón" 5 | "Mosquetón" y "5 unidades" en Inventario |
| C-011-02 | marta suma 3 a Casco de 2 | "Casco" y "5 unidades" en Inventario |
| C-011-03 | unidades 0 | "Indica al menos 1 unidad" en Inventario |
| C-011-04 | mismo formulario otra vez | "Casco" y "8 unidades" en Inventario |
| C-012-01 | baja 1 de Cuerda con 3 | "Cuerda" y "2 unidades" en Inventario |
| C-012-02 | baja 5 de Cuerda con 2 | "No hay tantas unidades activas de Cuerda" en Inventario |
| C-012-03 | `stock1` dado de baja; pablo reserva | "No quedan unidades libres de Casco esas fechas" en Material |
| C-013-01 | marta, 1 Entregada | "Prestado: Casco (lucia@club.es)" en Hoy |
| C-013-02 | Entregada con `hasta` = día elegido | "Vuelve: Casco (lucia@club.es)" en Hoy |
| C-013-03 | nada prestado | "No hay nada prestado" en Hoy |
| C-014-01 | ana, socio con préstamo y multa | "Casco" "2026-12-10" "2026-12-12" y "Multa de 6 € (Pendiente)" en Historial del socio |
| C-014-02 | ana, socio sin multas | "Sin multas" en Historial del socio |
| C-015-01 | ana abre Historial del socio | "Historial del socio"; sin botones reservar, cancelar, entregar, devolver, baja ni pagar |
| C-016-01 | lucia pide el historial de pablo | "No tienes permiso para ver estos datos" en Historial del socio |
| C-016-02 | pablo pide el de lucia | "No tienes permiso para ver estos datos" en Historial del socio |
| C-016-03 | marta, multas de varios socios | "lucia@club.es" y "Multa de 6 €" en Multas |
| C-016-04 | sin sesión abre Material | "Iniciar sesión" en Iniciar sesión |
| C-017-01 | lucia con sesión elige "Salir" | "Iniciar sesión" en Iniciar sesión (AC-017-01) |
| C-017-02 | lucia sale; se restaura su sesión anterior y se abre Material | "Iniciar sesión" en Iniciar sesión, sin datos (AC-017-02) |
| C-017-03 | lucia sale; se repite "Salir" con la sesión anterior | "Iniciar sesión" en Iniciar sesión (AC-017-03) |
| C-018-01 | `reloj` 10:00 lucia activa; 10:31 abre Material | "Tu sesión ha caducado" en Iniciar sesión (AC-018-01) |
| C-018-02 | pablo activo 09:00; 09:45 abre Material | "Tu sesión ha caducado" en Iniciar sesión (AC-018-05) |
| C-018-03 | lucia activa 10:00; 10:29 abre Material | "Hola, lucia@club.es" en Material (AC-018-02) |
| C-018-04 | lucia activa 10:00, 10:20, 10:40; 10:55 abre Material | "Hola, lucia@club.es" en Material (AC-018-03) |
| C-018-05 | sesión caducada; lucia entra con su contraseña | "Hola, lucia@club.es" en Material (AC-018-04) |
| C-NF1 | lucia reserva en una sesión; marta abre Entregas y devoluciones en otra | "Casco" "lucia@club.es" en Entregas y devoluciones (REQ-NF-001 AC1, AC-NF1-01) |

### Variaciones de validación y accesibilidad (P1)

| Variant ID | Field / escenario | Action | Expected | Spec Ref |
|------------|-------------------|--------|----------|----------|
| E2E-WF-001-V01 | Correo y Contraseña vacíos | enviar | `E_AUTH_INVALID` "Correo o contraseña incorrectos", sin sesión | AC-001-05 |
| E2E-WF-001-V02 | Material inexistente | reservar | `E_MATERIAL_NOT_FOUND` "Ese material no existe" | AC-002-09 |
| E2E-WF-001-V03 | socio abre Hoy / Entregas | navegar | sin pantalla ni botones de encargado (`E_FORBIDDEN_ROLE`) | AC-013-04, AC-006-05 |
| E2E-WF-001-A11Y-01 | teclado en S1 | Tab por todos los campos, Enter envía | cada campo alcanzable, envío correcto | S1 |

### Full tier (P2) — lista

| Variant ID | Given | Action | Expected | Spec Ref |
|------------|-------|--------|----------|----------|
| E2E-WF-001-F01 | `stock1`, lucia y pablo | piden a la vez | uno "Reserva creada…", otro `E_NO_STOCK` | AC-002-10 |
| E2E-WF-001-F02 | `res(lucia)` | dos cancelaciones a la vez | una "Reserva cancelada", la otra `O_ALREADY_CANCELLED` | AC-005-04 |
| E2E-WF-001-F03 | almacén caído al devolver tarde | devolver | `E_STORAGE_UNAVAILABLE`; ni devolución ni multa | AC-008-05 |
| E2E-WF-001-F04 | lucia con 1 multa | reserva mientras marta paga | decide el estado al confirmar | AC-009-05 |
| E2E-WF-001-F05 | `res(lucia)` futura | baja de la unidad | reserva sigue Reservada; sin aviso del sistema | AC-012-07, RN-002 |
| E2E-WF-001-F06 | sin sesión | abrir Mis reservas, Hoy, Historial | `E_AUTH_REQUIRED` → Iniciar sesión | AC-004-04, 013-05, 014-04 |
| E2E-WF-001-F07 | ana (junta) | forzar pagar / reservar | `E_FORBIDDEN_ROLE`, estado intacto | AC-015-02, 015-03 |
| E2E-WF-001-F08 | cualquier lectura | abrir otra vez | misma lista | AC-003-04, 004-06, 013-06, 014-05, 016-05 |
| E2E-WF-001-F09 | altas y bajas repetidas | repetir formulario | suma/resta otra vez | AC-011-04, 012-08 |

### smoke-deploy

Target environment: none (sin `staging_url`); sin escenarios.

## Field Coverage Verification

| Field | Happy step | Empty | Invalid | Conditional | Status |
|-------|-----------|-------|---------|-------------|--------|
| Correo, Contraseña | S0 ✅ | V01 ✅ | C-001-03 ✅ | N/A | COMPLETE |
| Material | S1 #2 ✅ | HUECO (sin AC; V02 cubre inexistente) | V02 ✅ | N/A | INCOMPLETE → GAP-008 |
| Desde / Hasta | S1 #2 ✅ | HUECO (sin AC de vacío) | C-002-06 ✅ | N/A | INCOMPLETE → GAP-008 |
| Devuelta el | S1 #5 ✅ | HUECO | GAP-003 | S2 #1 ✅ | INCOMPLETE → GAP-003, GAP-008 |
| Unidades | C-011-01 ✅ | C-011-03 ✅ | C-012-02 ✅ | N/A | COMPLETE |
| Día, Socio | C-013-02, C-014-01 ✅ | N/A | N/A | N/A | COMPLETE |

## Coverage Matrix

| REQ ID | Type | E2E Coverage | Justification |
|--------|------|--------------|---------------|
| REQ-F-001..019 | UI-func | S0..S2 + C-NNN-* (67 criterios visibles; REQ-F-017..019 = C-001-05..09, C-017-*, C-018-*) | los 5 `No visible` en integración |
| REQ-NF-001 | UI-effect | C-NF1 | — |
| REQ-NF-002 | Security | — | EXEMPT-NFR: SEC-T-001 |
| REQ-C-001..003 | Constraint | texto de cada captura | EXEMPT-NFR: MNT-T-001..003 |
