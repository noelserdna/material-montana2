# 01 — Glossary

| Term | Definition (≤ 20 words) | Do not use |
|---|---|---|
| Socio | Persona con cuenta y rol `socio`; reserva material y ve lo suyo (ENT-001) | miembro |
| Encargado | Persona con rol `encargado`; gestiona inventario, entregas, devoluciones y multas (ENT-001) | administrador, gestor |
| Junta | Persona con rol `junta`; solo consulta datos de cualquier socio (ENT-001) | directiva, consejo |
| Material | Tipo de artículo prestable (Piolet, Casco…) con nombre único (ENT-002) | producto, artículo |
| Unidad | Ejemplar físico de un material; activa o dada de baja (ENT-002) | ejemplar, pieza |
| Reserva | Petición de una unidad de un material para un rango de días (ENT-003) | booking, pedido |
| Entrega | Acto del encargado de dar la unidad reservada al socio (UC-006) | préstamo físico |
| Préstamo | Reserva Entregada o Devuelta; las Reservada y Cancelada no cuentan (UC-014) | — |
| Sesión | Periodo en que una cuenta está identificada; termina al salir o caducar (ENT-005, INV-ACC-007) | — |
| Devolución | Acto del encargado de registrar que la unidad vuelve (UC-007) | retorno |
| Multa | Importe a pagar por retraso en una devolución (ENT-004) | sanción, recargo |
| Baja | Retirada de unidades rotas del inventario (UC-012) | borrado |
| Cuenta | Correo, contraseña con hash y rol de una persona (ENT-001) | perfil |
| Bloqueo | Espera de 15 minutos sin poder entrar tras 5 fallos de contraseña en 15 minutos (INV-ACC-006) | castigo |
| Sesión caducada | Sesión terminada por 30 minutos sin actividad; hay que entrar de nuevo (INV-ACC-007) | timeout |
