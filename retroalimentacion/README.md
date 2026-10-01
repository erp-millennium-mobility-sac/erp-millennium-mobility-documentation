# Retroalimentación de implementaciones

Aquí se guardan las revisiones de código de cada contexto: qué se encontró, por qué importa y qué conviene hacer distinto en la próxima implementación.

Una revisión no es una evaluación de personas. Sirve para que el equipo comparta el mismo criterio y para que un error no se repita en la tabla siguiente.

## Cómo se escribe una revisión

- Un archivo por revisión, con nombre `AAAA-MM-DD-<contexto>.md`.
- Cada hallazgo indica **qué pasó**, **la evidencia** (archivo, línea o commit), **la regla que incumple** —o, si es aprendizaje, que esa regla no estaba escrita o no estaba al alcance de quien implementó— y **por qué importa para el negocio**.
- Se registra también lo que está bien hecho, para mantenerlo.
- La fuente de verdad es el diccionario de datos; después, las skills del proyecto (`backend-module`, `frontend-feature`, `architecture-review`) y [patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md).

## Revisiones

| Fecha | Contexto | Alcance |
|---|---|---|
| 30/09/2026 | `location` | [Monedas y países](2026-09-30-location.md) |
