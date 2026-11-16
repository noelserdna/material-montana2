# Technical Research — material-montana2

> Phase 3 skipped: no hay NEEDS_RESEARCH abiertos; todo lo decide una ADR-DRAFT o el diseño. Force it with `/sdd-plan-architect --research`.

| Item | Category | Decided by | Decision (≤ 80 chars) | Ref |
|------|----------|------------|------------------------|-----|
| Almacén de datos | CL-DATA | ADR-DRAFT-002 | SQLite con `node:sqlite` | D-PA-005 |
| Hash de contraseñas | CL-SEC | ADR-DRAFT-003 | `crypto.scrypt` con sal | D-PA-005 |
| Framework de interfaz | CL-UI | TECHNICAL-DESIGN §3 | JS vanilla, sin framework | D-PA-007 |
| Servidor de API | CL-ARCH | ADR-DRAFT-001 | plugin de Vite + `main.js` | D-PA-005 |
| Reloj en E2E | CL-TEST | TEST-PLAN R-8 | punto de control solo con variable de entorno | D-PA-009 |
