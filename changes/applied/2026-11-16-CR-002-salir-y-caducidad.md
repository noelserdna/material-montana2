# CR-002 — Botón Salir y caducidad por inactividad

- **Status:** APPLIED · **Type:** ADD · **ISO 14764:** perfective (security hardening) · **Source:** intent/INT-002.md (Q-security-auditor-001, option A)
- **After:** REQ-F-018 (botón "Salir") and REQ-F-019 (sesión caduca a los 30 min sin actividad) added. Spec: INV-ACC-007, `E_SESSION_EXPIRED`, UC-017, UC-018, API-001-02, BDD-UC-017/018, ENT-005 (`ultimaActividad`), `SESSION_IDLE_MIN`.
- **Defaults chosen:** 30 min de inactividad; cada operación renueva el plazo.
