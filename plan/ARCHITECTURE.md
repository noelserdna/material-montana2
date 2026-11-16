# Architecture — material-montana2

> Spec v1.0 · 2026-11-16 · Style: monolito modular en capas, un proceso Node (ADR-DRAFT-001)
> Inputs: spec/ yes · design/ yes · ux/ yes · audits/ yes

## 1. Decisions

| Aspect | Decision | Ref |
|--------|----------|-----|
| Stack profile | JavaScript ES modules + Vite + Vitest; sin kit | AGENTS.md |
| Topology | SPA + API en un proceso; `/api/*` como middleware de Vite en dev y `main.js` en prod | ADR-DRAFT-001 |
| Storage | SQLite `node:sqlite`, WAL, migraciones numeradas, sin ORM | ADR-DRAFT-002 |
| Auth | sesión en servidor, scrypt + sal, freno por cuenta, cierre y caducidad | ADR-DRAFT-003 |
| Dominio | JS puro sin I/O; reloj y repo inyectados | TEST-PLAN R-1..R-3 |
| Estado de FASEs | operaciones nuevas entran en `rutas.js` fase a fase | D-PA-001 |
| Plan-level | reloj de pruebas solo con `MONTANA_TEST_CLOCK=1` | D-PA-009 |

## 2. C4 Views

### 2.1 System Context (L1)

```
 socio ──┐
 encargado ─┼─ navegador ──HTTPS──> [ material-montana2 ] ──> data/club.db
 junta ──┘                          (sin sistemas externos)
```

| Element | Type | Interaction | Ref |
|---------|------|-------------|-----|
| socio | Person | reserva, ve y cancela las suyas | UC-001..005 |
| encargado | Person | inventario, entregas, multas, Hoy | UC-006..013 |
| junta | Person | consulta; sin escritura | UC-014, UC-015 |

### 2.2 Containers (L2)

```
 Navegador (SPA src/ui) ──POST /api/<op> JSON + cookie──> Proceso Node
   Proceso Node: http (sesión, rol, validación) → domain → repo ──> data/club.db
   Detrás de proxy inverso con TLS en producción (escucha en 127.0.0.1)
```

| Container | Technology | Purpose | Ref |
|-----------|-----------|---------|-----|
| SPA | JS vanilla + Vite | pantallas SCR-001..009 | ADR-DRAFT-001 |
| API | Node + `src/server/http` | 14 operaciones RPC | OPERATION-MAPPING |
| Almacén | SQLite fichero | datos compartidos | ADR-DRAFT-002 |

### 2.3 Components (L3)

```
src/
├─ shared/textos.js
├─ domain/   reloj · acceso · inventario · reservas · multas
├─ server/   main.js · vite-plugin.js · log-seguridad.js
│   ├─ http/   rutas · sesion · permisos · acceso · inventario · reservas · multas
│   ├─ repo/   almacen (interfaz) · sqlite · memoria · migrations/NNN-*.sql
│   └─ cli/    cuentas.js
└─ ui/       main · router · api · cabecera · mensaje · campos · pantallas/*
```

| Component | Module | Responsibility (ids) | Key interfaces | FASE |
|-----------|--------|----------------------|----------------|------|
| `domain/acceso.js` | acceso | hash, freno, sesión (INV-ACC-001, 002, 006, 007) | `iniciarSesion`, `cerrarSesion` | 0, 6 |
| `domain/inventario.js` | inventario | alta, baja, activas (INV-INV-*) | `anadirMaterial`, `darDeBajaUnidades`, `listarMaterial` | 0, 4 |
| `domain/reservas.js` | reservas | stock, SM-001, Hoy (INV-RES-*) | `crearReserva`, `cancelarReserva`, `entregarReserva`, `devolverReserva`, `verHoy` | 1–4 |
| `domain/multas.js` | multas | cálculo y bloqueo (INV-FIN-*) | `calcularMulta`, `marcarMultaPagada`, `verHistorialSocio` | 3, 5 |
| `http/permisos.js` | http | matriz de roles y dueño (INV-ACC-004/005) | `exigirRol`, `esDuenoOStaff` | 0, 5 |
| `repo/almacen.js` | repo | puerto de persistencia con transacción | `transaccion(fn)` | 0 |

## 3. Deployment

```
 Dev:  npx vite :5173 (SPA + API)      Prod: proxy TLS ──> node src/server/main.js :8080 ──> club.db
```

| Environment | Purpose | Infrastructure | Entry |
|-------------|---------|----------------|-------|
| Development | local | Node ≥ 22.13 | `http://127.0.0.1:5173` |
| Test | Vitest / Playwright | BD `:memory:` o fichero temporal | `npm test` |
| Production | club | VPS pequeño + proxy | URL del club |

Pipeline: `npm ci → npm test → vite build → playwright (Smoke) → desplegar`. Copia diaria `.backup`, 30 días.

## 4. Data Model

```sql
cuenta(correo PK, hash, sal, rol CHECK IN('socio','encargado','junta'))
material(nombre PK, unidades_activas INTEGER CHECK(>=0))
reserva(id PK, socio→cuenta, material→material, desde, hasta, estado CHECK IN(...), devuelta_el)
multa(id PK, reserva_id UNIQUE→reserva, socio, dias, importe, estado CHECK IN('Pendiente','Pagada'))
sesion(correo PK→cuenta, token_hash, ultima_actividad)      -- una por cuenta; FASE-6 añade ultima_actividad
fallo_acceso(cuenta_correo, instante)                       -- FASE-0
CREATE INDEX idx_reserva_material_estado ON reserva(material, estado, desde, hasta);
CREATE INDEX idx_reserva_socio ON reserva(socio);
```

| Data | Store | Format | Retention |
|------|-------|--------|-----------|
| Cuentas y reservas | SQLite | `Dia` `YYYY-MM-DD`, `Instante` ISO 8601 UTC | decisión del club |
| Fallos de acceso | SQLite | una fila por fallo | ventana 15 min, limpieza al acertar |

Migraciones: `001-base.sql` (FASE-0), `002-reservas.sql`, `003-multas.sql`, `004-sesion-actividad.sql`; se aplican en orden al arrancar y no tienen vuelta atrás automática.

## 5. Integration Map

Sin sistemas externos.

## 6. Security & Error Handling

```
petición → Content-Type JSON + Origin → validar esquema → exigirSesion → exigirRol → dominio → {ok,…} | {ok:false,codigo}
```

| Data class | At rest | In transit | Access control | Ref |
|-----------|---------|-----------|----------------|-----|
| Contraseñas | scrypt + sal | TLS en proxy | nunca se devuelven | CRYPTO-001, REQ-NF-002 |
| Reservas y multas | SQLite | TLS en proxy | dueño, encargado, junta | INV-ACC-004 |

Errores: mapeo en `design/OPERATION-MAPPING.md` y TECHNICAL-DESIGN §8; `E_AUTH_LOCKED` → 429, `E_SESSION_EXPIRED` → 401.

## 7. Quality Attributes

| Attribute | Target | Strategy | Ref |
|-----------|--------|----------|-----|
| Security | 0 P0 | freno por cuenta, denegar por defecto, SQL parametrizado | SECURITY-AUDIT-BASELINE |
| Reliability | sin sobre-reserva ni escritura parcial | transacción `BEGIN IMMEDIATE` | INV-RES-001 |
| Testability | 90 % líneas de dominio | dominio puro, reloj y repo inyectados | TEST-PLAN §2 |
| Maintainability | un módulo de textos | `src/shared/textos.js` | REQ-C-001 |

---

> Generated by sdd-plan-architect | Spec v1.0 | 2026-11-16
