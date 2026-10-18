# Retroalimentación de implementaciones

Aquí se guardan las revisiones de código de cada contexto: qué se encontró, por qué importa y qué conviene hacer distinto en la próxima implementación.

Una revisión no es una evaluación de personas. Sirve para que el equipo comparta el mismo criterio y para que un error no se repita en la tabla siguiente.

## Cómo se escribe una revisión

- Un archivo por revisión, con nombre `AAAA-MM-DD-<contexto>.md`.
- Cada hallazgo lleva su tipo y tres líneas:
  - **Error:** qué estaba mal, con la evidencia (archivo, línea o commit) y por qué importa.
  - **Corrección:** cómo se corrigió y en qué commit.
  - **Próxima vez:** cómo se hace en la siguiente implementación.
- El tipo es **error** cuando incumple el diccionario o DESIGN.md, que están en los repositorios, y **aprendizaje** cuando la regla no estaba escrita o no estaba al alcance de quien implementó, como las skills.
- El mensaje de un commit describe solo lo que su diff trae.
- Se registra también lo que está bien hecho, para mantenerlo.
- La fuente de verdad es el diccionario de datos; después, las skills del proyecto (`backend-module`, `frontend-feature`, `architecture-review`) y [patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md).

## Revisiones

| Fecha | Contexto | Alcance |
|---|---|---|
| 30/09/2026 | `location` | [Monedas y países](2026-09-30-location.md) |
| 05/10/2026 | `location`, `organization` | [Escalones, divisiones, direcciones y tipos de documento tributario](2026-10-05-location.md) |
| 15/10/2026 | `location` | [Cierre del paso 1](2026-10-15-location.md) |
| 16/10/2026 | `iam`, `organization` | [Modelo de acceso del paso 3](2026-10-16-iam.md) |
| 18/10/2026 | `location`, `organization` | [Cierre del paso 1: dígito de control, direcciones de las ubicaciones y sonda en vivo](2026-10-18-location.md) |
