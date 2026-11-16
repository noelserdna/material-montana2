# Questions — security-auditor

<!-- Vista generada desde questions-security-auditor.json por `sdd questions`: no se edita a mano (se regenera). Pregunta nueva: `sdd questions add --role security-auditor`. -->

## Q-security-auditor-001 [ANSWERED] skill=sdd-security-auditor context=SEC AUTH-001, AUTH-002 (audits/SECURITY-AUDIT-BASELINE.md) tema=bloqueo-y-caducidad-de-sesion
Question: API-001-01 has no failed-login throttling/lockout and INV-ACC-002 defines no session expiry or logout. Which policy should the spec adopt?
Para el cliente: ¿Qué hacemos si alguien falla la contraseña muchas veces, y cuánto tiempo puede quedar abierta una sesión?
Options:
  A) Temporary lockout after repeated failures (e.g. 5 in 15 min) and session expiry by inactivity plus explicit logout (recommended)
     efecto: req-change requirements/REQUIREMENTS.md spec/nfr/SECURITY.md
     cliente: Tras varios fallos se espera unos minutos; la sesión se cierra sola por inactividad y hay botón de salir
  B) Only explicit logout, no lockout (small closed club)
     efecto: req-change requirements/REQUIREMENTS.md spec/nfr/SECURITY.md
     cliente: Solo botón de salir; sin esperas tras fallar
  C) Accept the risk and leave as is
     efecto: ninguno
     cliente: Se deja como está
Blocks: Remediación AUTH-001, AUTH-002
Answer: A — opción recomendada, aceptada por Andres Leon (Cliente) (2026-11-16)
