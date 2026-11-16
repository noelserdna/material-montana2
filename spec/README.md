# Specifications — material-montana2

> From `requirements/REQUIREMENTS.md` v1.1 (2026-11-16). Decisions: `CLARIFICATIONS.md`. Prosa en español; ids, código y literales de la interfaz como en los requisitos.

Web del club de montaña para reservar material, entregarlo, devolverlo y multar retrasos. Tres roles: socio, encargado, junta.

| Path | Content | Start here if… |
|---|---|---|
| `domain/` | glosario, entidades, objetos de valor + catálogo de errores, estados, invariantes | escribes o revisas algo |
| `use-cases/UC-001..018` | un caso de uso por requisito funcional (REQ-F-018 y REQ-F-019 con uno cada uno) | implementas una función |
| `workflows/WF-001` | ciclo de una reserva | quieres la visión de conjunto |
| `contracts/` | API-acceso, -reservas, -inventario, -multas; permisos; eventos | implementas una operación |
| `tests/` | BDD-UC-001..018 (AC-NNN-NN) | escribes tests |
| `nfr/` | disponibilidad, seguridad, restricciones | miras los no funcionales |
| `VALUE-REGISTRY.md`, `DERIVED-SPECS.md`, `TRACEABILITY-MATRIX.md` | valores, derivados, trazabilidad | auditas |
