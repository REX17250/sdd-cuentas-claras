# AGENTS.md — cuentas-claras

## Proyecto
Aplicación de línea de comandos para registrar gastos compartidos entre personas de un grupo (amigos, compañeros de piso, un viaje) y calcular quién le debe a quién. Se construye con desarrollo guiado por especificaciones (SDD).

## Fases
1. Constitución — `docs/constitution.md` (decisiones en `docs/constitution-proposal.md`)
2. Aclaraciones y spec — `specs/001-cuentas-claras-mvp/clarifications.md` y `spec.md`
3. Revisión de QA — `specs/001-cuentas-claras-mvp/qa-review.md`
4. Resolución de hallazgos — nueva versión de `spec.md`
5. Plan técnico — `plan.md`, `data-model.md` y `contracts/`
6. Tareas — `tasks.md`

No se escribe código de la aplicación hasta terminar la fase 6. Cada fase se cierra con un commit en una rama de trabajo (nunca en `main`).

## Reglas
- Lee `docs/constitution.md` y la spec activa en `specs/` antes de tocar nada.
- Preguntas de una en una, con 2-4 opciones y la recomendada marcada. Con «usa tus recomendaciones», se elige la recomendada y se registra el motivo.
- La spec dice QUÉ y POR QUÉ; el plan dice CÓMO. Lo ambiguo se marca `[NECESITA ACLARACIÓN]`, no se supone.
- Numeración rastreable: `RF-x.y`, `RNF-x`, `CL-x`, `M-x` (mensajes), `D-x` (decisiones), `H-x` (hallazgos), `T-x` (tareas).
- Si algo contradice una fase cerrada, se avisa en vez de decidirlo.
- Código e identificadores en inglés; interfaz y documentación en español.

## Comandos (desde la fase 7)
- Ejecutar: `python -m cuentas <comando>`
- Tests: `pytest -q` (debe pasar entero al terminar cada tarea)

## Estado
- Fases 1-6 completas (2026-10-25). La implementación empieza por T-1 y sigue el orden de `tasks.md`.
