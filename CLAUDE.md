@AGENTS.md

## Solo para Claude Code
- Para fases de documentación, trabaja en modo plan hasta que el documento esté listo para escribirse.
- Abre una sesión nueva (`/clear`) para cada fase y para cada tarea de implementación: el estado vive en `specs/`, no en el chat.
- Al cerrar una tarea, muestra la salida de `pytest -q` como evidencia.
