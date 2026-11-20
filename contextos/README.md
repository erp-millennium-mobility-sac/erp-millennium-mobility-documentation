# Contextos — ERP Millennium Mobility

**Fuente:** diccionario de datos, [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) y el código. Si un documento de esta carpeta y el diccionario no coinciden, manda el diccionario.

Los demás documentos están ordenados por tema: los datos (diccionario), la API (convenciones), las fronteras (mapa), los procesos de negocio y las pantallas (procesos y pantallas). Cada documento de esta carpeta explica un contexto a nivel de negocio: qué es, cómo vive cada agregado, qué hace cada persona y qué reglas lo gobiernan. Cada hecho vive en un solo lugar:

| Hecho | Dónde vive |
|---|---|
| Los datos y las decisiones | El diccionario |
| El contrato de la API: las rutas, los errores y las exportaciones | Las convenciones: §2.2, §7.2 y §8.1 |
| La pantalla | Procesos y pantallas, y el `DESIGN.md` del frontend |
| Los bloqueos y la escritura, con el orden de los candados | Los [patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md) |
| El comportamiento de un contexto | Su documento, en esta carpeta |

| Documento | Contexto |
|---|---|
| [location.md](location.md) | `location`: monedas, países con su estructura territorial, divisiones y ubigeo, direcciones, tipos de documento tributario y tasas de cambio |
| [iam.md](iam.md) | `iam`: cuentas, roles, permisos y sesiones; dar acceso con su código de activación, bloqueos y el enlace para restablecer la contraseña |
| [organization.md](organization.md) | `organization`: empresas, marcas comerciales, sedes, ubicaciones, cuentas bancarias y colaboradores con su historial de ingresos |
| [vehiclecatalog.md](vehiclecatalog.md) | `vehiclecatalog`: tipos de vehículo, marcas y modelos, y colores |
| [equipment.md](equipment.md) | `equipment`: los tipos, las marcas y los modelos de equipamiento, con los tipos de vehículo a los que aplica cada tipo, y el inventario de los equipos físicos de cada empresa. Desde la fase 2 del plan de migración (D-278) |
| [fleet.md](fleet.md) | `fleet`: el vehículo, sus estados, sus caminos y su kilometraje, la habilitación —con la instalación de los equipos y lo que exige de ellos—, los fondos y las cesiones |
| [procurement.md](procurement.md) | `procurement`: proveedores, plantillas y órdenes de compra, facturas de compra, notas de crédito, pagos a proveedores y Por pagar |
| [reporting.md](reporting.md) | `reporting`: la capa de lectura, los reportes de Logística y el exportable de stock (D-232) |
| [notifications.md](notifications.md) | `notifications`: avisos por correo, SMS y push. Módulo de la plataforma; hoy, el correo del enlace para restablecer la contraseña y el del código para activar una cuenta (D-242, D-243, D-255) |
| [audit.md](audit.md) | `audit`: la bitácora de cambios, quién cambió qué y cuándo. Módulo de la plataforma, fuera de `shared` desde la fase 1 del plan de migración (D-129, D-262) |
| [imports.md](imports.md) | `imports`: el registro de las cargas de archivos, con el lector y el escritor de libros. Módulo de la plataforma, fuera de `shared` desde la fase 1 del plan de migración (D-200, D-262) |

Las tolerancias de los caminos entre estados de una unidad están en la [especificación de las tolerancias](../especificacion-tolerancias-de-los-caminos-2026-11-09.md).

## Qué lleva cada documento

1. **Qué es y qué no es.** Breve: el propósito, las tablas que posee y lo que parece suyo y no lo es.
2. **Ciclo de vida de cada agregado.** Sus estados y transiciones, qué queda fijo y desde cuándo, y cómo se retira. Cita las reglas por su identificador, sin volver a enunciarlas.
3. **Flujos funcionales**, a nivel de negocio. Quién lo hace (el rol o el permiso), qué pasos da y qué revisa el sistema en cada uno, citando el identificador de la regla. Si algo falla, una línea que lo diga, citando como mucho el código de error. Un flujo puede nombrar la pantalla («Logística › Facturas») y, para ubicar cada paso, el endpoint (método y ruta).
4. **Reglas de negocio.** Una tabla con cada regla, su identificador, su fuente, la capa que la impone, la prueba que la cubre y su estado (abajo).
5. **Relación con otros contextos.** Los miembros de la fachada y los puertos, quién los usa y para qué, en forma breve; y lo que el contexto necesita de otros.
6. **Estado y pendientes.** Solo lo que falta de verdad, y quién lo resuelve o cuándo.
7. **Preguntas abiertas.** Con el documento donde se lleva cada una.

Un mismo hecho no se enuncia en dos secciones del mismo documento. Se enuncia una vez, como regla en la sección 4, y el ciclo de vida y los flujos la citan por su identificador.

## Qué no lleva

- **Las columnas de las tablas.** Están en el diccionario, que es la fuente única. Aquí se cita la sección.
- **La lista de endpoints, el catálogo de errores y las exportaciones.** Están en convenciones §2.2, §7.2 y §8.1. Un flujo nombra el endpoint de un paso solo para ubicarlo, y un error solo por su código.
- **Las columnas de las listas y de las exportaciones.** Están en convenciones §8.1 y en procesos y pantallas.
- **El detalle de pantalla:** los rótulos de los botones, las pestañas, los píxeles, los textos de los mensajes y los cuerpos JSON. Están en procesos y pantallas, en el `DESIGN.md` del frontend y en las convenciones.
- **La estructura de paquetes, las clases y las restricciones.** Los nombres de clases, de restricciones de la base, los grants y las migraciones que imponen una regla los dice el código; la estructura, las skills `backend-module` y `frontend-feature`.
- **Los métodos de prueba.** Una regla nombra la clase o el archivo de prueba que la cubre, nunca la lista de sus métodos.
- **El orden de los candados.** Vive en los patrones de persistencia.
- **La historia.** Cómo se llegó a una regla, cuándo se construyó, en qué rama o commit, qué hallazgo la pidió y qué decía antes: lo guarda git. Un documento dice solo lo que rige hoy.

Copiar cualquiera de esas fuentes garantiza que este documento y la fuente terminen diciendo cosas distintas.

## Las reglas y sus identificadores

Cada regla lleva un identificador estable con el prefijo del contexto (`LOC-`, `IAM-`, `ORG-`, `VCT-`, `FLT-`, `PRC-`, `RPT-`, `NTF-`, `AUD-` para `audit`, `SHR-` para `imports`, cuyas reglas de las cargas de archivos están en el diccionario, §11.8, y conservan el prefijo de cuando eran de `shared`, y `EQP-` para `equipment`, que conserva además los `VCT-` de las reglas que vinieron de `vehiclecatalog`) y un número que no se reutiliza. Así una prueba, un commit o una revisión pueden citarla sin repetirla. Una regla que queda sin efecto conserva su número en una línea que dice qué la sustituye mientras algo la cite; si nadie la cita, se quita de la tabla.

La tabla de reglas tiene las columnas `Id | Regla | Fuente | Capa | Prueba | Estado`:

| Columna | Qué lleva |
|---|---|
| Id | El identificador de la regla |
| Regla | Qué se exige y, en breve, por qué. Sin nombres de clases, restricciones, grants ni migraciones |
| Fuente | La sección del diccionario y la decisión (D-n) de donde sale |
| Capa | La capa que la impone: Base, Dominio, Servicio o Frontend, una o varias, sin nombres de clases |
| Prueba | Solo el nombre de la clase o del archivo de prueba que la cubre (`InvoiceDatePolicyTests`, `purchase-invoices.store.spec.ts`), nunca la lista de sus métodos |
| Estado | Si la regla se cumple hoy: ✅ aplicada y probada, ⚠️ aplicada sin prueba o solo en parte, ❌ no aplicada |

La **capa** dice qué tan protegida está la regla:

| Capa | Qué significa |
|---|---|
| Base | Una restricción de la base de datos (`CHECK`, `UNIQUE`, llave foránea). La respeta cualquier camino de escritura |
| Dominio | El agregado se niega a quedar en un estado inválido |
| Servicio | El servicio de comandos lo revisa antes de guardar. Otro camino de escritura podría saltársela |
| Frontend | La pantalla no deja hacerlo. Nunca alcanza por sí sola: la API sigue abierta |

## Cuándo se actualiza

Un documento de contexto se actualiza en el mismo cambio que modifica una regla, un flujo o lo que falta de su contexto, y se reescribe a su estado vigente: la regla nueva reemplaza el texto de la anterior en vez de sumarse a él, y lo que se construyó deja de figurar como pendiente.
