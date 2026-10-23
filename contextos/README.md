# Contextos — ERP Millennium Mobility

**Actualizado:** 23/10/2026 (`procurement.md`: proveedores, tipos, plantillas y órdenes, con el paso 7 todavía abierto) · 20/10/2026 (la segunda fase de la segunda ronda, integrada en `develop` en los dos repositorios: `iam.md` y `organization.md` verificados contra el código, sin marcas pendientes) · 20/10/2026 (la segunda ronda de la auditoría, diccionario v1.25, en construcción: `iam.md`, `organization.md` y `fleet.md` al día, y una línea en `location.md` y `vehiclecatalog.md`) · 20/10/2026 (`vehiclecatalog.md`, la primera versión, con lo que aprobó la auditoría del 20/10/2026; diccionario v1.24) · 20/10/2026 (`fleet.md`: el paso 5, construido en `feature/fleet` e integrado en `develop` el mismo día) · 19/10/2026 (diccionario v1.23; la matriz de permisos, construida como paso 3b e integrada en `develop` el 19/10/2026) · 18/10/2026 (diccionario v1.22; `vehiclecatalog` ya tiene código) · 14/10/2026 · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.25, [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) y el código de las ramas revisadas. Si un documento de esta carpeta y el diccionario no coinciden, manda el diccionario.

Los demás documentos están ordenados por tema: los datos (diccionario), la API (convenciones), las fronteras (mapa), los procesos de negocio (procesos y pantallas). Para entender un contexto completo hay que leer partes de los cuatro y después el código. Esta carpeta junta, por contexto, lo que no está en ninguno de ellos.

| Documento | Contexto | Paso |
|---|---|---|
| [location.md](location.md) | `location`: geografía, monedas y documentos tributarios | 1 |
| [iam.md](iam.md) | `iam`: cuentas, roles, permisos y sesiones | 0, 3 y 3b |
| [organization.md](organization.md) | `organization`: empresas, marcas, sedes, ubicaciones y colaboradores | 2 y 3 |
| [vehiclecatalog.md](vehiclecatalog.md) | `vehiclecatalog`: tipos de vehículo, marcas y modelos, colores y equipamiento | 4 |
| [fleet.md](fleet.md) | `fleet`: el vehículo, sus estados, sus caminos y su kilometraje | 5 |
| [procurement.md](procurement.md) | `procurement`: proveedores, plantillas y órdenes de compra. Facturas, notas y pagos siguen sin código | 7 |

El paso 3b, la matriz de permisos, se construyó el 19/10/2026 en `feature/permissions-matrix`, sobre `feature/organization-followup`; las dos ramas se integraron en `develop` ese mismo día, y lo que cada una cambia está en la sección 6 de [iam.md](iam.md) y de [organization.md](organization.md). `vehiclecatalog` tiene documento desde el 20/10/2026: su paso, el 4, está construido en `develop` y sigue abierto, a falta de los modelos reales conciliados ([orden de implementación](../orden-de-implementacion-millennium-mobility.md), paso 4), y lo que la auditoría de ese día le sumó está construido e integrado en las ramas de integración de la auditoría, con su segunda fase, la exportación. `fleet` tiene documento desde el 20/10/2026: su paso, el 5, se construyó en `feature/fleet`, en el backend y en el frontend, y se integró en `develop` ese mismo día, con el paso 4 todavía abierto, por decisión del responsable. La segunda ronda de la auditoría (20/10/2026; D-117 a D-128) toca a todos. La primera fase está integrada en `develop` del backend (merge `e245186`): la exportación de cada contexto pasa por el módulo `documents` (D-125), con la versión de sus datos por su propio puerto y por los contadores de `data_versions` (D-128), los códigos de los permisos por `PermissionCodes`, de `shared`, y `fleet` mueve y parte sus piezas de acceso. La segunda también está integrada en `develop`, en los dos repositorios (merges `007151dd` en el backend y `11235e0` en el frontend, sin push): Seguridad pasa a permisos (D-117 a D-124) y Roles y permisos suma la matriz (D-126), con V56 a V58. Las marcas que los documentos dejaron donde dependían del código se resolvieron contra él al integrar; no queda ninguna. `procurement` tiene documento desde el 23/10/2026: su paso, el 7, está en curso en `develop`, con proveedores, tipos, plantillas y órdenes; facturas, notas de crédito y pagos siguen sin código, y el documento los nombra como pendientes en vez de describir un flujo que el código todavía no decidió. El documento de un contexto se escribe cuando empieza su paso.

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

Cada regla lleva un identificador estable con el prefijo del contexto (`LOC-`, `IAM-`, `ORG-`, `VCT-`, `FLT-`, `PRC-`) y un número que no se reutiliza. Así una prueba, un commit o una revisión pueden citarla sin repetirla. Una regla que se retira queda en la tabla marcada como retirada.

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
