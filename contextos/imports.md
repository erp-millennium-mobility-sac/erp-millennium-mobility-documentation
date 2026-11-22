# Contexto `imports` — ERP Millennium Mobility

**Fuente:** diccionario de datos (versión vigente; §11.8, D-125, D-189, D-200, D-201 y D-262), el [mapa de contextos](../mapa-de-contextos-millennium-mobility.md) (§5.1, §5.3 y §6.5) y el código de `develop`. Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** `imports` es el registro de las cargas de archivos, un módulo de la plataforma desde la fase 1 del plan de migración (D-262); antes vivía en `shared`. Sus reglas son las **SHR-R01 a SHR-R13 del diccionario, §11.8**, que conservan su número: este documento las cita y no las copia. Las columnas de `import_batches` están en el diccionario (§11.8), la ruta y sus errores en las convenciones (§2.2 y §7.2), y la pantalla Logística › Cargas de archivos en procesos y pantallas.

**Las cargas están construidas y no se usan todavía** (§11.8): antes de usarlas se ensayan con los archivos reales.

---

## 1. Qué es y qué no es

**Qué es.** El registro de cada carga masiva de un archivo —la declaración del impuesto vehicular, la lista de la garantía mobiliaria de un fondo y la migración inicial desde el libro de stock—, con su archivo, su informe de fallos y sus conteos (SHR-R01, SHR-R02). Y el mecanismo que comparten las cargas para leer sus libros y escribir su informe: un *Cohesive Mechanism* que viaja con su registro (SHR-R06 a SHR-R09).

**Tabla que posee:** `import_batches` (§11.8, V115).

**Lo que parece suyo y no lo es:**

- **Lo que cada carga actualiza** es del contexto dueño: el impuesto vehicular, de `fleet`; la garantía mobiliaria, de `financing` (D-279); la carga inicial la orquesta `procurement`, porque las unidades nacen con su factura (D-200). `imports` no escribe ninguna de esas tablas ni conoce sus reglas: solo registra el lote.
- **Los archivos** se guardan en `kept_files`, del módulo `documents`, por `KeptFileStore` (D-189).
- **El nombre de quien cargó y la empresa** son de `iam` y de `organization`, que `imports` pide por los puertos de `audit::acl` (SHR-R11).

## 2. Ciclo de vida del agregado

**`ImportBatch`** es una fila por carga terminada. Se escribe una sola vez, al final de la carga, y no cambia ni se borra (SHR-R01). Una carga que falló en todas sus filas también deja su fila, con su informe. No tiene estados.

## 3. Flujos funcionales

### F1 · Una carga registra su lote

- **Quién:** la carga de un contexto —`fleet` (`/tax-declaration-imports`), `financing` (`/fund-assignment-imports`) o `procurement` (`/initial-migration-imports`)—, con el permiso de su carga y `habilitacion.ver_cargas` en la empresa (SHR-R05).
- **Pasos:** toma el candado de su tipo y su empresa (`ImportLocks`, SHR-R10); lee el libro con `ImportWorkbookReader` (SHR-R06 a SHR-R08); aplica sus filas con sus propias reglas; escribe el informe con `ImportReportWriter` (SHR-R09); y registra el lote con `ImportBatchCommandService`, que guarda el archivo y el informe en `kept_files` (SHR-R03).
- **Si falla:** una carga del mismo tipo que ya corre en la empresa responde `409 IMPORTBATCH_CONFLICT` (`ImportRefusals.conflict()`); un archivo que no es un libro válido, `422` antes de leer filas.

### F2 · Leer las cargas

- **Quién:** quien tiene `habilitacion.ver_cargas` donde actúa (SHR-R04), en Logística › Cargas de archivos (`GET /api/v1/imports`, `/{id}`, `/{id}/error-report` y `/{id}/file`).
- **Resultado:** la lista acotada a las empresas que alcanza, con quién cargó por el nombre de su colaborador (SHR-R11), el informe de fallos y el archivo cargado.

## 4. Reglas de negocio

Son las del diccionario, §11.8, con su número: **SHR-R01 a SHR-R13**. Las pruebas que las cubren son las de este módulo —`FileLoadsMigrationTests`, `ImportCellTests`, `ImportReportWriterTests` e `ImportWorkbookReaderTests`— y las de cada carga en su contexto: `FleetImportsIntegrationTests` (`fleet`), `FundAssignmentImportsIntegrationTests` (`financing`) e `InitialMigrationIntegrationTests` (`procurement`). Lo que falta probar de cada carga está en [fleet.md](fleet.md), [financing.md](financing.md) y [procurement.md](procurement.md).

## 5. Relación con otros contextos

**`imports::acl`** es una puerta repartida (mapa de contextos §5.1): cada tipo lleva `@NamedInterface("acl")` sin dejar su capa, y los de sus firmas entran con él (`propagate`); `ModularityTests` la verifica.

| Miembro | Para qué | Quién lo usa |
|---|---|---|
| `ImportBatchCommandService` | Registrar el lote de una carga | `fleet` y `procurement` |
| `ImportBatchQueryService` | Leer una carga y nombrar las cargas de una lista | `fleet` (la respuesta de sus cargas, la carga que abrió una cesión y la de una declaración) y `procurement` (la respuesta de la carga inicial) |
| `ImportOutcome`, `ImportResultResource` y su ensamblador | La respuesta de una carga | `FleetImportsController` e `InitialMigrationImportsController` |
| `ImportLocks` | Una carga de un tipo a la vez por empresa | `fleet` y `procurement` |
| `application/spreadsheets` | Leer el libro y escribir el informe | `fleet` y `procurement` |
| `ImportReading`, `ImportRefusals` (`interfaces/acl`) | Quién lee las cargas de una empresa y la negativa de una carga que ya corre | `fleet` y `procurement` |

**Lo que usa:** de `audit::acl`, `AuditActorLookupPort` y `AuditCompanyLookupPort`, para el nombre de quien cargó y la empresa de cada carga; de `documents::acl`, `KeptFileStore`, sobre el que implementa su propio puerto `ImportBatchFileStore` (`ImportBatchFileStoreAdapter`). Declara `allowedDependencies = {"audit::acl", "documents::acl"}`.

`imports::acl` sale de `procurement` en el borrado 3b del plan de migración, cuando se borra el código de la carga inicial, después de la puesta en marcha: la carga corre antes, así que no pasa a un módulo `legacyimport` (decisión 4 de la fase 0 del plan).

## 6. Estado y pendientes para cerrar

| Qué | Cuándo |
|---|---|
| El ensayo de las tres cargas con los archivos reales, y los encabezados reales del SAT y de la garantía (§11.8) | Antes de usarlas, con los archivos que entrega el negocio |
| Deshacer una carga no existe; el lote guarda su archivo y las filas que cargó lo nombran, así que se puede construir sin cambiar tablas ([fleet.md](fleet.md), sección 6) | Sin paso asignado |

## 7. Preguntas abiertas

Ninguna propia. Las de cada carga están en [fleet.md](fleet.md) y [procurement.md](procurement.md).
