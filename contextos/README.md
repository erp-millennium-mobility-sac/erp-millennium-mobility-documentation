# Contextos — ERP Millennium Mobility

**Actualizado:** 19/10/2026 (diccionario v1.23; la matriz de permisos, construida como paso 3b e integrada en `develop` el 19/10/2026) · 18/10/2026 (diccionario v1.22; `vehiclecatalog` ya tiene código) · 14/10/2026 · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.23, [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) y el código de las ramas revisadas. Si un documento de esta carpeta y el diccionario no coinciden, manda el diccionario.

Los demás documentos están ordenados por tema: los datos (diccionario), la API (convenciones), las fronteras (mapa), los procesos de negocio (procesos y pantallas). Para entender un contexto completo hay que leer partes de los cuatro y después el código. Esta carpeta junta, por contexto, lo que no está en ninguno de ellos.

| Documento | Contexto | Paso |
|---|---|---|
| [location.md](location.md) | `location`: geografía, monedas y documentos tributarios | 1 |
| [iam.md](iam.md) | `iam`: cuentas, roles, permisos y sesiones | 0, 3 y 3b |
| [organization.md](organization.md) | `organization`: empresas, marcas, sedes, ubicaciones y colaboradores | 2 y 3 |

El paso 3b, la matriz de permisos, se construyó el 19/10/2026 en `feature/permissions-matrix`, sobre `feature/organization-followup`; las dos ramas esperan su integración en `develop`, y lo que cada una cambia está en la sección 6 de [iam.md](iam.md) y de [organization.md](organization.md). `vehiclecatalog` ya tiene código —su paso, el 4, está en curso en `develop`, y en `feature/permissions-matrix` sus lecturas y escrituras exigen `catalogos.ver` y `catalogos.gestionar_flota`— pero todavía no tiene documento: escribirlo, con sus reglas, es parte de cerrar el paso ([orden de implementación](../orden-de-implementacion-millennium-mobility.md), paso 4). Los contextos que todavía no tienen código (`fleet`, `procurement`) no tienen documento. Su diseño está en el diccionario, y escribir sus flujos antes de construirlos sería adivinar lo que el código decidirá. El documento de un contexto se escribe cuando empieza su paso.

## Qué lleva cada documento

1. **Qué es y qué no es.** El propósito, las tablas que posee y lo que parece suyo y no lo es.
2. **Ciclo de vida de cada agregado.** Quién lo crea, qué se puede cambiar y qué queda fijo desde cuándo, cómo se retira y qué nunca pasa.
3. **Flujos funcionales.** Lo que hace una persona, paso a paso: con qué rol, en qué pantalla, qué revisa el sistema en cada paso y qué ve si algo falla.
4. **Reglas de negocio.** Cada regla con un identificador, su fuente en el diccionario, dónde se aplica y qué prueba la cubre.
5. **Relación con otros contextos.** Quién usa su fachada y para qué; lo que el contexto necesita de otros.
6. **Estado y pendientes para cerrar.** Lo construido, lo que falta y quién lo resuelve.
7. **Preguntas abiertas.** Con un enlace al documento donde se lleva cada una.

## Qué no lleva

- **Las columnas de las tablas.** Están en el diccionario, que es la fuente única. Aquí se cita la sección.
- **La lista de endpoints.** Está en convenciones §2.2. Un flujo nombra el endpoint de cada paso solo para ubicarlo.
- **La estructura de paquetes y las clases.** Están en las skills `backend-module` y `frontend-feature`.

Copiar cualquiera de las tres garantiza que este documento y la fuente terminen diciendo cosas distintas.

## Las reglas y sus identificadores

Cada regla lleva un identificador estable con el prefijo del contexto (`LOC-`, `IAM-`, `ORG-`) y un número que no se reutiliza. Así una prueba, un commit o una revisión pueden citarla sin repetirla. Una regla que se retira queda en la tabla marcada como retirada.

La columna **Dónde se aplica** dice qué capa la impone, porque de eso depende qué tan protegida está:

| Capa | Qué significa |
|---|---|
| Base | Una restricción de la base de datos (`CHECK`, `UNIQUE`, llave foránea). La respeta cualquier camino de escritura |
| Dominio | El agregado se niega a quedar en un estado inválido |
| Servicio | El servicio de comandos lo revisa antes de guardar. Otro camino de escritura podría saltársela |
| Frontend | La pantalla no deja hacerlo. Nunca alcanza por sí sola: la API sigue abierta |

La columna **Estado** dice si la regla se cumple hoy: ✅ aplicada y probada, ⚠️ aplicada sin prueba o solo en parte, ❌ no aplicada.

## Cuándo se actualiza

Un documento de contexto se actualiza en el mismo cambio que modifica una regla, un flujo o el estado de su paso, igual que el orden de implementación. Al cerrar un paso, la sección 6 de su contexto queda sin pendientes de código o con cada pendiente asignado a un paso posterior.
