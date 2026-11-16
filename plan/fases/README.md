# Fases de Implementación - material-montana2 v1.0

> **Propósito:** Índices de navegación para implementar el sistema por fases.
> **Fuente de verdad:** Las especificaciones en `spec/` (no estos índices).

`spec/` = fuente única de verdad → `plan/fases/` = índices → `src/` = derivado.

## Diagrama de Dependencias

```
FASE-0 (entrar + catálogo)
   └─ FASE-1 (reservar)
         └─ FASE-2 (entregar/devolver)
               ├─ FASE-3 (multas) ── FASE-5 (junta y datos propios)
               │                          └─ FASE-6 (Salir y caducidad)
               └─ FASE-4 (Hoy y bajas)
```

Orden de ejecución: 0 → 1 → 2 → 3 → 4 → 5 → 6 (4 solo depende de 2).

## Resumen de Fases

| Fase | Documento | Incremento | Requisitos | Dependencias |
|------|-----------|------------|------------|--------------|
| **0** | [FASE-0-SKELETON.md](FASE-0-SKELETON.md) | Entrar, añadir material y verlo en el catálogo | REQ-F-001, REQ-F-017, REQ-F-011, REQ-F-003, REQ-NF-002 | — |
| **1** | [FASE-1-RESERVAS.md](FASE-1-RESERVAS.md) | Reservar, ver y cancelar | REQ-F-002, REQ-F-004, REQ-F-005 | 0 |
| **2** | [FASE-2-ENTREGAS.md](FASE-2-ENTREGAS.md) | Entregar y recibir el material | REQ-F-006, REQ-F-007, REQ-NF-001 | 1 |
| **3** | [FASE-3-MULTAS.md](FASE-3-MULTAS.md) | Multas por retraso | REQ-F-008, REQ-F-009, REQ-F-010 | 2 |
| **4** | [FASE-4-HOY-Y-BAJAS.md](FASE-4-HOY-Y-BAJAS.md) | Ver el día y dar de baja unidades | REQ-F-012, REQ-F-013 | 2 |
| **5** | [FASE-5-JUNTA.md](FASE-5-JUNTA.md) | La junta consulta; datos solo del dueño | REQ-F-014, REQ-F-015, REQ-F-016 | 3 |
| **6** | [FASE-6-SESION.md](FASE-6-SESION.md) | Salir y caducidad de sesión | REQ-F-018, REQ-F-019 | 5 |

**Requisitos Must sin fase:** ninguno (V9). REQ-C-001..003 son criterios transversales de todas las fases.

## Cobertura de Casos de Uso

| Fase | UCs Incluidos |
|------|---------------|
| 0 | UC-001, UC-011, UC-003 |
| 1 | UC-002, UC-004, UC-005 |
| 2 | UC-006, UC-007 (+ AC-005-08) |
| 3 | UC-008, UC-009, UC-010 (+ AC-002-08) |
| 4 | UC-012, UC-013 |
| 5 | UC-014, UC-015, UC-016 |
| 6 | UC-017, UC-018 |

**Total:** 18 casos de uso cubiertos (UC-001 a UC-018).

## Cobertura de ADRs y diseño

No hay `spec/adr/`: las decisiones son `design/ADR-DRAFT-001` (SPA + API monolito, FASE-0), `ADR-DRAFT-002` (SQLite, FASE-0 y migraciones de cada fase) y `ADR-DRAFT-003` (sesión, scrypt, freno y caducidad, FASE-0 y FASE-6).

## Cobertura de Tests

BDD `AC-001-01..11` … `AC-018-05` repartidos como en la línea `Escenarios` de cada fase (109 escenarios, ninguno repetido). E2E por fase en `test/E2E-SCENARIOS.md`: S0 (FASE-0), S1 (FASE-2), S2 (FASE-3).
