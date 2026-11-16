# CR-003 — Lista de todas las reservas para el encargado

- **Status:** APPLIED · **Type:** ADD · **ISO 14764:** perfective · **Source:** intent/INT-003.md (Q-task-generator-001, option A)
- **After:** REQ-F-020 (el encargado ve las reservas de todos los socios en Entregas y devoluciones, filtro Por entregar / Entregadas / Todas; solo encargado). Spec: UC-019, BDD-UC-019, API-002-07 `listarReservas`, PERMISSIONS-MATRIX, INV-ACC-004/005 (UC-019), error `E_FORBIDDEN_ROLE`.
- **Defaults chosen:** filtro por defecto "Por entregar"; "Todas" incluye también Devuelta y Cancelada; mensaje "No hay reservas" cuando el filtro no devuelve nada.
