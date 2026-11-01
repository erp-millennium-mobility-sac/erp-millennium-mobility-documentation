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
| 18/10/2026 | `organization`, `iam` | [Cierre de los pasos 2 y 3: identidad de las cuentas, padres inactivos y sonda en vivo](2026-10-18-organization.md) |
| 18/10/2026 | `organization` | [Seguimiento del cierre después de integrarlo: quién lee los colaboradores, reactivar bajo una empresa inactiva y fallas de carga](2026-10-18-organization-seguimiento.md) |
| 19/10/2026 | `iam`, `shared`, `location`, `vehiclecatalog`, `organization` | [La matriz de permisos del paso 3b: los hallazgos del panel de diseño que la fijaron y lo que queda abierto](2026-10-19-permisos.md) |
| 26/10/2026 | `fleet` | [Fondos: el catálogo y las cesiones de `feat/fleet-funds`, cerrados con la auditoría del 25/10/2026, y cómo limpiar la base local](2026-10-26-fondos.md) |
| 30/10/2026 | `procurement`, `fleet`, `location`, `organization`, `iam` | [Sondas en vivo contra el código del 30/10/2026: 50 de 50, y un aviso de rendimiento del formulario de la orden](2026-10-30-sondas.md) |
| 31/10/2026 | `vehiclecatalog`, `fleet`, `shared`, `procurement`, `organization`, `location`, `iam` | [Sondas en vivo contra el código del 31/10/2026: 72 de 72 en dos corridas, con las nuevas de los pasos 4, 5 y 6, y tres hallazgos menores cerrados](2026-10-31-sondas.md) |
| 31/10/2026 | `iam`, `location`, `procurement`, `fleet`, `shared` | [El cierre de la revisión completa: el freno del inicio de sesión, la contraseña en bytes, las monedas de dos decimales, la traducción de las fachadas, el camino de estado en el dominio y los textos, con la corrección de H3](2026-10-31-revision.md) |
