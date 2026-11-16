# CR-001 — Espera tras fallar la contraseña varias veces

- **Status:** APPLIED · **Type:** ADD · **ISO 14764:** perfective (security hardening) · **Source:** intent/INT-002.md (Q-security-auditor-001, option A)
- **After:** REQ-F-017 added (5 fallos en 15 min → cuenta bloqueada 15 min). Spec: INV-ACC-006, `E_AUTH_LOCKED`, UC-001 (v1.1), API-001-01, BDD AC-001-07..11, RN-005, VALUE-REGISTRY.
- **Defaults chosen (the request left them open):** 5 intentos / ventana 15 min / bloqueo 15 min desde el último fallo; contador por cuenta; un acierto lo pone a cero.
