# Maintainability — restricciones

| Field | Value |
|---|---|
| Refs | REQ-C-001, REQ-C-002, REQ-C-003 |

| Rule | Value | Verification |
|---|---|---|
| Textos de la interfaz en español, desde un único módulo de textos (literales de `domain/03-VALUE-OBJECTS.md`) | todo en español | test de textos |
| Stack | JavaScript ES modules + Vite + Vitest, `maxWorkers: 2` | `package.json`, `vitest.config` |
| Fechas en días naturales y tarifa única | `FINE_RATE_EUR` en `VALUE-REGISTRY.md` | test del cálculo de días |
