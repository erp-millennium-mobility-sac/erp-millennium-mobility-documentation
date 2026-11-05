# Especificación: cargas de archivos (paquete de cargas)

**Versión 2 · 03/11/2026** (las nueve preguntas de la versión 1, del mismo día, quedan decididas por D-206 a D-209, delegadas por el responsable: §10) · **Estado:** borrador para validar · **Punto de partida:** backend `b1264ed5` (hasta V91), frontend `88312e13`, documentación `6ebb900` con el diccionario v1.38 (D-157 a D-209), las especificaciones de [facturas de compra](especificacion-facturas-de-compra-2026-11-02.md) y de [habilitación](especificacion-habilitacion-2026-11-02.md), las dos en su versión 2, y el [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) v3. Las especificaciones de [notas de crédito](especificacion-notas-de-credito-2026-11-03.md) (versión 2) y de [pagos a proveedores](especificacion-pagos-a-proveedores-2026-11-03.md) (versión 1) se escribieron en paralelo con esta, que se alinea con ellas: los motivos y el tope de líneas de la nota (D-204), y los pagos del Excel registrados con el modelo de la fase 5, como propone su P11. Si esta especificación y el diccionario no coinciden, manda el diccionario.

**Fuentes:**
- el diccionario: §2.2, §2.5 a §2.7, §5.6, §8.1 a §8.4, §9.3 a §9.6, §10, §11, §12 y §13; las decisiones D-18, D-23, D-30, D-133, D-134, D-153, D-160, D-168, D-169, D-176, D-177, D-183, D-186, D-189, D-190 a D-199 y D-200 a D-209;
- el [mapa de contextos](mapa-de-contextos-millennium-mobility.md), §5.1, §5.3, §6.5 y §11, decisiones 3, 6 y 8 (esta última, cerrada por D-200);
- las [convenciones de la API](convenciones-de-la-api-millennium-mobility.md), §2.2, §6.2, §7, §8 y §14.18;
- [contextos/fleet.md](contextos/fleet.md), sección 6 f, y [contextos/procurement.md](contextos/procurement.md);
- [procesos y pantallas](procesos-y-pantallas-millennium-mobility.md), P9;
- las skills `backend-module` y `frontend-feature`;
- la guía funcional del cliente v2.0 (§3.1, §5 columnas 1 a 52, §6, §7, §8 y §11) y la plantilla `STOCK_MILLENNIUM_2026_EXPORTABLE FINAL.xlsx`, leída el 03/11/2026: una hoja `STOCK` con la tabla `StockMillennium2026` (`A1:AZ2001`), los 52 encabezados en la fila 1 y 2 000 filas vacías con formato, sin fórmulas ni validaciones.

**Cómo leerlo.**
- §1 fija el alcance, el primer paso (perfilar los archivos reales) y el orden.
- §2 a §4 son para el equipo: el modelo del backend, el esquema y el contrato de la API, que los commits D2, D7 y D10, uno por entrega, pasan a las convenciones antes del código.
- §5 es el formato de cada archivo, con el destino de cada una de las 52 columnas.
- §6 son las reglas, todas en ❌ hasta construirse: `SHR-R01` en adelante para `shared`, que no tenía prefijo propio; `FLT-R100` en adelante para `fleet`; `PRC-R140` en adelante para `procurement`.
- §7 son las pantallas; §8, las pruebas y sus archivos; §9, la secuencia de commits, en las tres entregas de D-208; §10, las preguntas de la versión 1 y la decisión que cerró cada una.
- Lo marcado **(propuesta)** es un nombre o una regla que ninguna decisión fija.
- En la columna *Fuente* de las tablas de reglas, §*n* es del diccionario. En el texto, un §*n* que existe en esta especificación es suyo y los demás son del diccionario; uno del diccionario que se confundiría con uno de aquí lo dice («§2.6 del diccionario»).

---

## 1. Alcance

### 1.1 Qué se construye

Tres cargas de archivos y su registro común (D-200, D-201, D-202):

| Carga | Contexto | Qué hace | Cómo entra |
|---|---|---|---|
| **Impuesto vehicular (SAT)** | `fleet` | Registra la declaración de cada auto que ya existe, cruzada por placa (§11.7) | Fila por fila |
| **Garantía mobiliaria** | `fleet` | Abre las cesiones de las placas de la lista y cierra las de las que ya no figuran, sin borrar la historia (§12.2, D-30) | Fila por fila |
| **Carga inicial** | `procurement`, con la fachada de `fleet` | Da de alta cada unidad del Excel de stock con su factura, sus notas de crédito, su habilitación y su estado (D-168, D-177, D-202), y registra sus bloques de pago con el modelo de la fase 5 | Por factura: las filas de una factura entran o fallan juntas |
| **Registro de las cargas** | `shared` | `import_batches`, sus archivos en `kept_files` y la pantalla Logística › Cargas de archivos (`/imports`) | — |

Cada carga vive en el contexto dueño de lo que actualiza y registra su lote por la fachada de `shared` (D-200). Ningún archivo auxiliar crea unidades: solo la carga inicial las da de alta (§11.8).

### 1.2 Decisiones que implementa

| Decisión | Qué toma esta especificación |
|---|---|
| D-18 | `legacy_number` llega solo con la carga inicial (FLT-R125, PRC-R140) |
| D-23, D-177 | Las notas de débito van al archivo de errores y nunca se registran (PRC-R147) |
| D-30, §12.2 | La lista de garantía actualiza la historia: abre y cierra cesiones, nunca las borra ni las reescribe (FLT-R111, FLT-R112) |
| D-133 | La llave de `vehicle_fund_assignments.import_batch_id` hacia `import_batches` llega con este paquete (FLT-R118) |
| D-134 | Escribir exige leer: cada carga exige también `habilitacion.ver_cargas`, y la de garantía, `fondos.ver` (SHR-R05). Toda escritura de cesiones lee su fondo en modo compartido (FLT-R115) |
| D-153, D-160 | Las unidades nacen con su factura; las reglas del número, la fecha, las líneas y la línea vigente única valen también en la carga inicial (PRC-R145) |
| D-168, D-177 | La carga inicial trae la factura, las notas y la habilitación, con sus excepciones: la tasa copiada de la fila, sin los 20 días, las notas agrupadas por código y proveedor, y al archivo de errores las notas de débito y lo que no está en el catálogo (PRC-R143, PRC-R144, PRC-R148) |
| D-169 | En la moneda funcional la tasa es 1 (PRC-R143) |
| D-176 | La carga inicial registra facturas reales: por eso espera a la fase 4 (§1.4, D-208) |
| D-183 | La unidad migrada entra a Disponible solo si no le falta nada (FLT-R123) |
| D-186 | Ninguna fecha de la habilitación es futura, y una instalación no es anterior al ingreso (FLT-R124) |
| D-189 | Los archivos de las cargas van a `kept_files` con la empresa de la carga (SHR-R03) |
| D-192, D-193, D-204, D-205 | Cada línea de una nota migrada copia la tasa de su factura; la nota respeta las reglas del registro a mano, con su motivo guardado como código y hasta 100 líneas, y sin el límite de 20 días, que la nota no tiene (PRC-R148) |
| D-190, D-194 a D-199 | Cada bloque de pago del Excel es un pago de la fase 5, aplicado a sus facturas hasta lo que se debe de ellas (PRC-R153) |
| D-206 | La TIVE y el SOAT migrados sin archivo y el GPS sin serial, con el serial que se completa una sola vez; las fechas que el Excel no trae, deducidas de la fila; la ubicación que elige quien carga; las llaves con etiqueta provisional; y la unidad de otro propietario, que entra como de la empresa (FLT-R124, FLT-R127, PRC-R154) |
| D-207 | Las notas migradas con motivo `DESCUENTO_CAMPANA`, y las que dejarían una línea en cero, a mano; los pagos migrados, uno por bloque, por transferencia, con su tasa y, en el canal de dealers, imputados por unidad y saldados (PRC-R148, PRC-R153) |
| D-208 | Tres entregas, cada una con su puerta; la carga inicial corre en producción una sola vez, después de la fase 5; la carga es síncrona, con un tope de filas medido (§1.4, §9, PRC-R151) |
| D-209 | La lista de garantía cierra también las cesiones a mano; la de una unidad sin placa no se cierra y se informa; una placa de otro fondo falla sin tocar nada; el fondo y la fecha quedan en el informe (FLT-R112, FLT-R113) |
| D-200 | Cada carga vive en su contexto dueño y registra su lote en `shared`, que sirve `/imports` (§2.1) |
| D-201 | Archivo .xlsx con encabezados exactos; el impuesto y la garantía, fila por fila; la carga inicial, por factura, y un VIN ya migrado se informa sin repetirse (SHR-R07, PRC-R141, PRC-R142) |
| D-202 | El camino «Migración inicial» y el estado de cada unidad: Activo si se entregó, Disponible si cumple la habilitación, si no En habilitación (FLT-R120, FLT-R123) |

### 1.3 Primer paso: perfilar los archivos reales (D-177)

**Nada de la carga inicial se construye antes de este paso.** Los archivos reales —Flota Lizzo New (autos), Flota Lima Bikes (motos y trimotos), un archivo del SAT y una lista de garantía mobiliaria— no estaban disponibles al escribir esta especificación. Sus formatos y su calidad deciden reglas que aquí quedan provisionales.

**Cómo se hace.**
- El responsable entrega una copia de los cuatro archivos. No se suben a ningún repositorio ni a la base: llevan placas, VIN y montos reales.
- Un script desechable, fuera de los repositorios, lee cada archivo y produce solo agregados: conteos, valores distintos de los catálogos y ejemplos enmascarados.
- La salida es `retroalimentacion/<fecha>-perfil-de-los-archivos-reales.md`, sin placas, VIN ni montos de unidades concretas. Cada hallazgo dice qué regla de esta especificación confirma o cambia.

**Lo que se mide y qué cambia cada hallazgo:**

| # | Archivo | Qué se mide | Qué cambia según el resultado |
|---|---|---|---|
| 1 | Stock (los dos) | En qué hoja están las filas y si su fila 1 trae los 52 encabezados de la plantilla, carácter por carácter: `N°` (signo de grado) o `Nº` (ordinal), tildes, dobles espacios, saltos de línea dentro de la celda | Si difieren, el cliente copia sus filas en la plantilla: los encabezados son exactos (D-201) y el lector no adivina. El perfil lista cada diferencia para que la corrija |
| 2 | Stock | Filas con datos, filas vacías intercaladas, filas de totales o de notas al pie | El tope de filas por archivo (PRC-R151) y la regla de la fila vacía (SHR-R07). Una fila de totales tiene que salir antes de cargar |
| 3 | Stock | El tipo de celda de cada columna: fecha como fecha o como texto, montos como número o como texto, separadores de miles, cuántos decimales trae el TC | Los formatos que acepta SHR-R08. Un formato de fecha que no sea `dd/mm/aaaa` ni `aaaa-mm-dd` se corrige en el archivo o se suma al lector con una decisión |
| 4 | Stock | Los centinelas de cada columna: `-`, `NO`, `NO TIENE`, `NO CORRESPONDE`, `DE FÁBRICA`, vacío | La lista de centinelas de SHR-R08 y de §5.4 |
| 5 | Stock | `N°`: entero, único en cada archivo, y si los dos archivos repiten números | Si los dos archivos repiten números, confirma que el número es único por marca y no por empresa (FLT-R125) |
| 6 | Stock | VIN: 17 caracteres, sin I, O ni Q, repetidos dentro de un archivo y entre los dos, y cuántos ya existen en `erp_mm_db_verify` y en producción | Los grupos que fallarán por un VIN repetido o registrado fuera de la migración (PRC-R142). Si hay unidades reales dadas de alta a mano desde el 20/10/2026, el responsable decide si sus facturas se registran a mano antes de migrar |
| 7 | Stock | Proveedores distintos (columna 11) contra la razón social y el nombre comercial de los proveedores de la empresa | Los proveedores que hay que crear antes de migrar; un nombre que coincide con dos proveedores (PRC-R144) |
| 8 | Stock | Pares marca y modelo (7 y 8) y colores (9) contra el catálogo; el tipo que da el catálogo contra la columna 52 | La lista de modelos conciliada que pide el plan (plan §3.2). Un tipo distinto del de la columna 52 delata un modelo mal elegido |
| 9 | Stock | Facturas: formato del número (D-160), filas por factura (máximo), facturas cuyas filas no coinciden en fecha, moneda o TC, y números que aparecen en los dos archivos | Una factura de más de 100 filas choca con D-160 y la decide el responsable. Las incoherencias anticipan los errores de PRC-R146 |
| 10 | Stock | Moneda y TC: valores, filas en PEN con TC distinto de 1, TC distintos para un mismo día y moneda | En PEN, un TC distinto de 1 hace fallar la factura (PRC-R143). Dentro de una factura, dos TC distintos también (PRC-R146); entre facturas distintas, cada una conserva el suyo (D-177) y el perfil solo los cuenta, como la alerta de la guía |
| 11 | Stock | Precio de factura: mayor que cero, a lo sumo dos decimales | PRC-R145 |
| 12 | Stock | NC/ND: códigos, valores negativos (notas de débito), notas que cruzan facturas, notas anteriores a su factura, notas que dejan una unidad en cero, y la columna 41 contra el precio menos la nota | Los errores de PRC-R147 y PRC-R148; si una unidad recibió dos notas, el Excel solo guarda una y la diferencia aparece aquí. Las notas que dejan una unidad en cero se registran a mano con su motivo |
| 13 | Stock | Pagos (42 a 44): códigos con `/`, un mismo código con fechas o montos distintos, bloques que cruzan proveedores o monedas | Los errores de PRC-R153: un bloque cuyas filas no coinciden en fecha, monto o moneda falla. Cuántos bloques son de un proveedor del canal de dealers, que exige el detalle por unidad (D-197), y si la columna 45 es la diferencia prorrateada que calcula la guía, que es lo que D-207 imputa; cuántos están en otra moneda y no tienen la tasa del día en el catálogo; y el máximo de facturas y de unidades por bloque, contra los topes de la fase 5 (100 aplicaciones y 500 imputaciones, su §2.5) |
| 14 | Stock | Habilitación (13 a 23): `EN PLANTA=NO` con GPS, `PLACAS=SI` sin placa, placa sin TIVE o sin título, `ENTREGADO=SI` sin fecha o sin requisitos, valores de PANTALLA, fechas futuras | Los errores de FLT-R124. Además, cuántas unidades caerían en Activo, Disponible y En habilitación, comparado con la columna 5: si no coinciden, el mapeo de §5.4 tiene un error |
| 15 | Stock | SOAT (25 a 27): `SI` con sus dos fechas, el fin a un año del inicio | Cuántos SOAT entran sin archivo (D-206) y cuántos ya vencieron, que la pantalla de vencimientos mostrará desde el primer día |
| 16 | Stock | Llaves (2 y 12): grupos, `NO TIENE`, seriales | Cuántas llaves entran con etiqueta provisional (D-206) |
| 17 | Stock | Propietario (50): valores distintos de Millennium | Las unidades que entran como de la empresa y esperan su traspaso (D-206, D-20) |
| 18 | Stock | Declaración de stock (51), y las fechas del GPS, la pantalla, la entrega y el título | Cuántos ingresos y cuántas placas físicas se deducen de cada fecha (D-206), y cuántas entradas a Disponible toman su momento de la columna 51 |
| 19 | SAT | Hoja y fila de encabezados; el texto exacto de las columnas de placa, estado y fecha de inscripción; los valores distintos del estado; formato de la fecha; placas con guion; placas repetidas; motos | Fija los encabezados provisionales de §5.2. Si el archivo trae una fila por placa y por año, FLT-R105 cambia de «la placa repetida falla» a «vale la fila más reciente» |
| 20 | Garantía | Encabezados; si trae una columna de fondo, una fecha por placa o una fecha de corte en alguna celda; placas repetidas | Fija §5.3. Una fecha por placa reemplaza la fecha de la lista como inicio de cada cesión nueva (FLT-R111); una columna de fondo permite validar que la lista es del fondo elegido |
| 21 | Los cuatro | Tamaño en bytes y en filas | Los topes de SHR-R06 y PRC-R151 |

**Después de construir, y antes de producción,** se hace un ensayo con los archivos reales sobre `erp_mm_db_verify` (§8.7).

### 1.4 Orden y dependencias

| Parte | Necesita | Por qué |
|---|---|---|
| `shared`: registro y pantalla | Nada | — |
| Impuesto vehicular | Fase 3 | La TIVE vigente (`vehicle_documents`, el tipo `TIVE`) y la placa registrada como hito (FLT-R65 de la fase 3) |
| Garantía mobiliaria | Nada más que lo construido (V75) | Las cesiones ya existen |
| Carga inicial: facturas, unidades y habilitación | Fases 2 y 3 | `FleetContextFacade`, `PurchaseInvoice`, `UploadedFileType`, `KeptFileStore.discardRolledBack`, `EnablementPolicy` y las tablas de la habilitación |
| Carga inicial: notas de crédito | **Fase 4** | `credit_notes` y `credit_note_items`, y D-176: ninguna factura real entra antes que las notas |
| Carga inicial: pagos | **Fase 5** | `SupplierPayment` con sus aplicaciones y sus imputaciones, y lo que se debe de cada factura (D-190) |

**Tres entregas** (D-208), cada una con su puerta (§9):
1. el registro de cargas, el impuesto vehicular y la garantía mobiliaria, después de la fase 3;
2. la carga inicial, después de la fase 4;
3. el paso de pagos de la carga inicial, después de la fase 5.

**En producción, la carga inicial corre una sola vez, después de la fase 5,** con el ensayo de §8.7 aprobado: los datos entran completos, con sus pagos.

### 1.5 Lo que queda fuera

- **Deshacer una carga.** El lote guarda su archivo y las filas que escribió llevan su `import_batch_id`, así que deshacerla se puede construir después sin cambiar tablas. Mientras tanto: una lista del SAT equivocada se corrige cargando la buena, cuya declaración pasa a ser la vigente; y la carga de garantía pide confirmar los cierres antes de escribir (FLT-R114).
- **Las pólizas** (columnas 28 a 30): siguen como candidato (§11.6).
- **El exportable de stock** con las columnas SAT y garantía: es de la capa de lectura (paso 11).
- **Cargas sin empresa**, como las divisiones territoriales (procesos, pregunta 7).

---

## 2. Modelo del backend

### 2.1 Contextos y dependencias

| Contexto | Qué cambia | `allowedDependencies` |
|---|---|---|
| `shared` | `shared/importbatches`, como `shared/changelog`: el agregado `ImportBatch`, su servicio de comandos —la fachada de D-200—, sus consultas, el lector de libros, el escritor del informe y `/imports` | Ninguna: es el módulo compartido (`@Modulithic(sharedModules = "shared")` en `ErpMillenniumMobilityBackendApplication.java`; `shared/package-info.java`, `Type.OPEN`) |
| `documents` | `ImportBatchFileStoreAdapter` implementa el puerto `ImportBatchFileStore` de `shared` sobre `kept_files` | Sin cambio (`{}`): implementa un puerto de `shared`, como `iam` implementa `AuditActorLookupPort` |
| `fleet` | Las cargas del impuesto y de la garantía; `vehicle_tax_declarations`; dos miembros de `FleetContextFacade` para la carga inicial; los caminos «Migración inicial» | Sin cambio sobre las fases 2 y 3 |
| `procurement` | La carga inicial | Sin cambio sobre la fase 2, que ya suma `fleet::acl` |
| `organization`, `vehiclecatalog`, `location` | Ninguno propio: se usan los miembros de las fases 2 y 3 | Sin cambio |
| `iam` | La migración de los permisos | Sin cambio |

**Por qué `shared` no depende de `documents`.** `documents` ya depende de `shared` (`Result`, `ApplicationError`): si `shared` importara `documents::acl` para guardar los archivos de la carga, cerraría un ciclo. Por eso `shared/importbatches/domain/ports` declara `ImportBatchFileStore` y `documents` lo implementa, con el mismo sentido que `AuditActorLookupPort` (`shared/changelog/domain/ports/AuditActorLookupPort.java`), que implementa `iam`.

**Por qué el impuesto y la garantía no viven en `shared`.** Escriben `vehicle_tax_declarations` y `vehicle_fund_assignments`, que son de `fleet` (regla 1.3; mapa §6.5). La carga inicial la orquesta `procurement` porque las unidades nacen con su factura (D-153), y escribe en `fleet` solo por su fachada.

### 2.2 `shared`: el registro de las cargas

**La fachada** (D-200) es el servicio de comandos de `shared/importbatches`, como `ChangeLogCommandService` lo es de la bitácora (`shared/changelog/application/commandservices/ChangeLogCommandService.java`):

```java
package ky.millennium.mobility.erp.shared.importbatches.application.commandservices;

public interface ImportBatchCommandService {

    /**
     * Registra una carga terminada: guarda su archivo y su informe en kept_files, con la empresa de la carga,
     * y escribe su fila. Corre en la transacción de quien llama (MANDATORY): si la carga se revierte, la fila
     * se va con ella y las llaves nuevas se descartan. La cuenta que la hizo es la de la sesión.
     *
     * @return el lote, con su id interno para las filas que lo nombran y su public_id para la respuesta;
     * o VALIDATION_ERROR (400) si los conteos no cuadran
     */
    Result<RegisteredImportBatch, ApplicationError> register(RegisterImportBatchCommand command);
}

public record RegisterImportBatchCommand(Long companyId, ImportType importType, String fileName,
        byte[] sourceBytes, ImportCounts counts, @Nullable byte[] reportBytes) {}

public record ImportCounts(int rowsTotal, int rowsOk, int rowsFailed) {}   // rowsUnchanged = total − ok − failed

public record RegisteredImportBatch(Long id, UUID publicId, Instant importedAt) {}
```

**Por qué el lote se escribe al final.** `import_batches` no tiene estado ni conteos provisionales (§11.8), y solo se inserta (§3.3). Cada carga decide primero todo lo que hará, arma su informe y recién entonces registra el lote y escribe las filas que lo nombran. La carga inicial, que corre en varias transacciones, registra el lote en una transacción propia al terminar (§2.8). Si se cae a la mitad, lo que entró queda sin lote, pero volver a subir el archivo lo informa como ya migrado (D-201).

**Las piezas:**

| Capa (`shared/importbatches`) | Clases |
|---|---|
| `domain/model/aggregates` | `ImportBatch`: empresa, tipo, nombre del archivo, llave del archivo, conteos, llave del informe, quién y cuándo. Su constructor exige `rowsTotal > 0`, `rowsOk + rowsFailed ≤ rowsTotal` y el informe si hay filas fallidas. `rowsUnchanged()` se calcula |
| `domain/model/valueobjects` | `ImportType` (`IMPUESTO_VEHICULAR("impuesto_vehicular")`, `GARANTIA_MOBILIARIA("garantia_mobiliaria")`, `MIGRACION_INICIAL("migracion_inicial")`, en minúscula como toda lista cerrada, D-133), `ImportCounts` |
| `domain/model/commands` y `queries` | `RegisterImportBatchCommand`; `SearchImportBatchesQuery`, `GetImportBatchByPublicIdQuery`, `GetImportBatchFileQuery` (el archivo o el informe) |
| `domain/repositories` | `ImportBatchRepository`: `save`, `findByPublicId`, `search`, `findReferencesByIds`. Sin `delete` ni actualización |
| `domain/ports` | `ImportBatchFileStore`: `put(companyId, key, bytes, mediaType)`, `open(key)` y `discardRolledBack(key)` |
| `application/commandservices` | `ImportBatchCommandService` |
| `application/queryservices` | `ImportBatchQueryService`, `ImportBatchView`, `ImportBatchFile`, `ImportBatchReference` (`id`, `publicId`, `importedAt`), para que `fleet` nombre la carga de una cesión o de una declaración |
| `application/spreadsheets` | Los puertos de lectura y escritura: `ImportWorkbookReader`, `ImportSheetSpec`, `ImportRow`, `ImportCell`; `ImportReportWriter`, `ImportReport`, `ImportRowFailure` |
| `application/internal/commandservices` | `ImportBatchCommandServiceImpl` |
| `application/internal/queryservices` | `ImportBatchQueryServiceImpl`, `ImportBatchViewFactory`; quién cargó se resuelve con `AuditActorLookupPort` de la bitácora, que implementa `iam` (mapa, decisión 6, opción A, que D-200 deja en `shared`) |
| `application/internal/policies` | `ImportBatchAccessPolicy` (pura): `reads(scope, companyId)`, `habilitacion.ver_cargas` con `actsOn`; `reach(scope)`, con `companyReach`. La usan la consulta y las tres cargas |
| `infrastructure/persistence/jpa` | `ImportBatchPersistenceEntity`, sin clase base: `imported_by` e `imported_at` son datos del hecho, como `uploaded_by` en los documentos (especificación de habilitación, 2.10); su ensamblador, repositorio de Spring Data y adaptador |
| `infrastructure/spreadsheets` | `FastexcelImportWorkbookReader`, con `XlsxContainerCheck`; `FastexcelImportReportWriter` |
| `interfaces/rest` | `ImportBatchesController` (`/imports`), `ImportBatchResource`, `ImportBatchResourceFromViewAssembler`, y `ImportResultResource` (el lote con los efectos), que devuelven las tres cargas |

`fastexcel-reader` pasa del alcance `test` al de compilación en `pom.xml` (hoy, líneas 205 a 208), con la misma versión que `fastexcel` (`0.19.0`). No se agrega Apache POI.

**La llave de cada archivo**, por su contenido, como las de la plantilla (D-134): `import-batches/{publicId de la empresa}/source/{sha256}` y `import-batches/{publicId de la empresa}/report/{sha256}`. No lleva el `public_id` del lote porque el archivo se guarda antes que la fila. Los mismos bytes dan la misma llave, y `put` no duplica nada. `discardRolledBack` se pide solo para una llave que esta transacción escribió por primera vez: la que `open` no encontraba antes de `put`, como los documentos del vehículo (especificación de habilitación, 2.6).

### 2.3 La lectura de un archivo

`ImportWorkbookReader.read(byte[] bytes, ImportSheetSpec spec)` devuelve `Result<ImportWorkbook, ApplicationError>`. Revisa en este orden y responde el primer fallo, sin leer la base:

1. **El contenedor** (`XlsxContainerCheck`, antes de inflar nada): un ZIP; `[Content_Types].xml` declara `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet.main+xml`; no hay `vbaProject.bin` ni el tipo con macros; a lo sumo 200 entradas, 100 MB descomprimidos en total y una relación de compresión de 100 a 1 por entrada. Si no, `422 IMPORTBATCH_FILE_TYPE` o `IMPORTBATCH_FILE_UNREADABLE`. El tamaño lo corta antes el servidor: 10 MB por archivo y 21 MB por pedido (`application.yaml`, líneas 44 a 46), con `413 CONTENT_TOO_LARGE`.
2. **El XML** se lee sin DTD ni entidades externas. Un `DOCTYPE` es `IMPORTBATCH_FILE_UNREADABLE`.
3. **La hoja:** la primera, en el orden del libro, cuya fila 1 trae los encabezados que pide `spec`. Ninguna: `422 IMPORTBATCH_HEADERS_MISMATCH`, con los de la primera hoja en `context.found` y los esperados en `context.expected`.
4. **Los encabezados.** Cada celda de la fila 1 se normaliza solo en sus espacios: forma NFC, sin espacios al borde, el espacio duro como espacio y los espacios seguidos como uno. Todo lo demás se compara exacto, también las tildes y el signo de `N°`.
   - **La carga inicial** exige los 52 de la plantilla, en su orden, desde la columna A, y ninguno más con texto.
   - **El SAT y la garantía** son exportaciones de otra plataforma: las columnas que se leen se encuentran por su encabezado exacto, y las demás se ignoran.
   - En los dos casos, una columna que falta, sobra o cambia de lugar responde `IMPORTBATCH_HEADERS_MISMATCH` con `context.missing`, `context.unexpected` y `context.sheet`.
5. **Las filas:** desde la 2. Una fila sin ningún valor se salta y no cuenta. Ninguna fila con datos: `422 IMPORTBATCH_EMPTY`. Más que el tope del tipo: `422 IMPORTBATCH_TOO_MANY_ROWS`, con `context.maxRows`.

**Las celdas** (`ImportCell`), con su texto tal como llegó para el informe:
- **Centinelas** (§2.2 del diccionario: no se almacenan): vacío, `-`, `—`, `NO TIENE`, `NO CORRESPONDE` y, en las columnas de sí o no, `NO`. La lista final sale del perfilado (§1.3, punto 4).
- **Sí o no:** `SI`, `SÍ` o `NO`, en cualquier mayúscula. Otro valor es `IMPORTBATCH_CELL_INVALID`.
- **Fechas:** una celda de fecha, con el sistema de 1900 o de 1904 que declare el libro, o un texto `dd/mm/aaaa` o `aaaa-mm-dd`. Una fecha nunca se lee como `mm/dd`.
- **Números:** una celda numérica, o un texto con un solo separador decimal y sin separador de miles; un texto con punto y coma a la vez se rechaza, como el analizador del texto pegado de la fase 2. El valor binario de Excel se redondea a la escala de la columna solo si la diferencia es menor que 10⁻⁹; si no, la escala sobra y la fila falla.
- **Texto:** sin espacios al borde y con los seguidos colapsados.

Los topes de filas (propuesta, fijados por la línea base de §8.4): 5 000 para el SAT y para la garantía, 1 000 para la carga inicial. Se configuran por tipo (`imports.max-rows.<tipo>`).

### 2.4 El informe de la carga (el archivo de errores)

`ImportReportWriter` arma un .xlsx con fastexcel, en memoria, con tres hojas:

| Hoja | Contenido |
|---|---|
| **Filas** (en la carga inicial, `STOCK`, como la plantilla) | Las filas que fallaron, tal como llegaron —mismo tipo de celda, mismo texto— bajo los mismos encabezados, en su orden original. Corregidas, la hoja se vuelve a subir: la lectura la encuentra por sus encabezados (§2.3, punto 3) |
| **Errores** | Una línea por fallo: `Fila` (la del archivo original, como la ve Excel), la llave (`Placa` o `VIN` y `N°`), `Columna`, `Código` y `Motivo`, el mensaje en español de `messages_es.properties` |
| **Resumen** | Tipo, empresa, archivo, fecha y hora de la carga en la zona de la empresa, y los cuatro conteos. En la garantía, el fondo y la fecha de la lista; en la carga inicial, la marca comercial y la ubicación |

La garantía suma una cuarta hoja, **Cesiones cerradas**, con la placa, el VIN, el inicio, el fin y el resultado de cada cesión que la lista cerró o no pudo cerrar.

Hay informe cuando alguna fila falló. La garantía lo escribe siempre: sus cierres y su fondo son lo que se audita. La descarga se llama `errores-<tipo>-<aaaa-mm-dd>.xlsx`, con la fecha de la carga en la zona de la empresa y el tipo sin tildes (`impuesto-vehicular`, `garantia-mobiliaria`, `migracion-inicial`).

**Qué cuenta cada número**, igual en las tres cargas:

| Conteo | Una fila cuenta aquí si… |
|---|---|
| `rows_ok` (Entraron) | escribió algo en esta carga y no está en el informe |
| Sin cambios (se calcula) | todo lo que pide ya estaba, y no está en el informe |
| `rows_failed` (Fallaron) | está en la hoja de errores |

### 2.5 `fleet`: la carga del impuesto vehicular

`TaxDeclarationImportCommandServiceImpl.handle(ImportTaxDeclarationsCommand(UUID companyId, String fileName, byte[] bytes))`, `@Transactional`, una sola transacción:

1. Lee el libro (§2.3) con la hoja del SAT (§5.2).
2. La empresa existe y se ve (`404 COMPANY_NOT_FOUND`); `habilitacion.cargar_archivo_sat` y `habilitacion.ver_cargas` actúan en toda la empresa y en todas sus marcas (`AccessScope.actsOnWholeCompanyEveryBrand`, línea 265; si no, `403`); la empresa está activa (`422 COMPANY_INACTIVE`).
3. Toma el candado de la carga (SHR-R10).
4. Normaliza cada placa con `LicensePlate` (`fleet/domain/model/valueobjects/LicensePlate.java`: de 3 a 10 letras o números, sin guiones, espacios ni puntos). Una placa repetida en el archivo hace fallar todas sus filas.
5. Lee las unidades de la empresa con esas placas, bloqueadas en orden de `id` (`FOR NO KEY UPDATE`), y en lote: el tipo de vehículo de sus modelos (`VehicleModelSummary.vehicleTypeCode`, de `vehiclecatalog`), si tienen la TIVE vigente (fase 3) y su última declaración.
6. `TaxDeclarationRowPolicy` (`domain/services`, pura) decide cada fila: no encontrada, no es auto, anulada, sin TIVE, sin cambios o nueva.
7. Arma el informe, registra el lote por la fachada de `shared` y guarda las declaraciones nuevas con su `import_batch_id`, en lotes de JDBC.

**Lo que se guarda** (`VehicleTaxDeclaration`, `domain/model/aggregates`): la unidad, su empresa, el estado tal como lo informa el archivo —recortado, en mayúsculas, hasta 30 caracteres—, la fecha de inscripción si viene y el lote. **Una declaración nueva se escribe solo si el estado o la fecha difieren de la última de la unidad**; si no, la fila cuenta como sin cambios. La última es la de mayor `id`: las cargas de una empresa no se cruzan (SHR-R10), así que el orden de inserción es el de las cargas, y `fleet` no lee `import_batches` para ordenarlas.

**El estado SAT de una unidad se calcula, no se guarda** (guía, columnas 31 y 32; §2.2 del diccionario), con `VehicleTaxStatusPolicy` (`domain/services`):

| Orden | Si… | Código | Fecha |
|---|---|---|---|
| 1 | el tipo de vehículo del modelo no es `AUTO` | `NO_CORRESPONDE` | — |
| 2 | no tiene la TIVE vigente | `SIN_TIVE` | — |
| 3 | no tiene ninguna declaración | `PENDIENTE` | — |
| 4 | tiene declaraciones | `DECLARADA`, con el estado del archivo de la última (`COMPLETADO`, `INSCRITO`…) | La de inscripción de la última |

`AUTO` es el código del tipo que siembra V35 y que las reglas leen; no cambia nunca (`VehicleTypeResource`, «Short code the system's rules read; it never changes»). Se lee en `GET /vehicles/{id}/tax-declarations` y en la página de habilitación de la unidad (§7.5).

### 2.6 `fleet`: la carga de garantía mobiliaria

**Qué es el archivo.** La lista de las placas que un fondo tiene en garantía a una fecha: una fotografía (guía, §8.2). Quien carga elige el fondo y la fecha de la lista. **La fotografía no reemplaza la historia** (D-30, §12.2): la carga la traduce en cesiones que se abren y se cierran.

`FundAssignmentImportCommandServiceImpl.handle(ImportGuaranteeListCommand(UUID companyId, UUID fundId, LocalDate listDate, boolean confirmClosures, String fileName, byte[] bytes))`, `@Transactional`, una sola transacción:

1. Lee el libro con la hoja de la garantía (§5.3).
2. La empresa (`404`); `fondos.cargar_archivo_garantia` y `habilitacion.ver_cargas` en toda la empresa y en todas sus marcas, y `fondos.ver` en la empresa (`403`); la empresa activa.
3. El fondo, leído en modo compartido (`FundRepository.findByPublicIdForShare`, como la cesión a mano, `VehicleFundAssignmentCommandServiceImpl.java`): de la empresa y visto con `fondos.ver` (`404 FUND_NOT_FOUND`), activo (`422 FUND_INACTIVE`). La fecha de la lista no es futura en la zona de la empresa (`422 VEHICLEFUNDASSIGNMENT_LIST_DATE_IN_FUTURE`).
4. El candado de la carga (SHR-R10).
5. Bloquea en orden de `id`, en una sentencia, las unidades de la empresa con las placas del archivo y las que tienen una cesión abierta al fondo. Una cesión a mano de esas unidades espera, porque bloquea la unidad primero (`contextos/fleet.md`, 6 f).
6. Lee las cesiones de esas unidades, de cualquier fondo.
7. `GuaranteeListPolicy` (`domain/services`, pura) decide:

| Caso | Efecto |
|---|---|
| Placa de la lista sin cesión abierta | Abre `[fecha de la lista, )` al fondo, con el `import_batch_id` y la cuenta de la sesión como autor |
| Placa de la lista con la cesión abierta a este fondo | Sin cambios |
| Placa de la lista con una cesión abierta a otro fondo | Falla la fila (`VEHICLEFUNDASSIGNMENT_OTHER_FUND`, con el código del otro fondo). No se cierra ni se abre nada: una unidad no está en dos fondos a la vez (§12.2), y la lista de un fondo no decide sobre otro |
| Placa de la lista cuya cesión nueva se cruzaría con una que ya existe, abierta o futura | Falla la fila (`VEHICLEFUNDASSIGNMENT_CONFLICT`) |
| Placa que no está en la empresa, o unidad anulada | Falla la fila (`VEHICLE_NOT_FOUND`, `VEHICLE_ANNULLED`) |
| Cesión abierta al fondo, de una unidad con placa que no está en la lista | Se cierra con la fecha de la lista, excluida como todo fin de período (§2.6 del diccionario), y deja su fila en la bitácora |
| Ese cierre no queda posterior al inicio de la cesión | No se cierra; va a la hoja Cesiones cerradas como fallido (`VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START`) |
| Cesión abierta al fondo de una unidad sin placa | No se cierra: la lista se cruza por placa y esa unidad no puede figurar. Va a la hoja Cesiones cerradas como «sin placa» |

8. **Los cierres se confirman** (propuesta, el patrón de los avisos de la fase 2, su §4.4). Si la carga cerraría al menos una cesión y el pedido no trae `confirmClosures: true`, responde `422 VEHICLEFUNDASSIGNMENT_IMPORT_UNCONFIRMED` con `context.toOpen`, `toClose`, `unchanged`, `failed` y `withoutPlate`, y no escribe nada. La pantalla muestra esos números y vuelve a enviar el archivo con la confirmación. Así una lista equivocada no cierra cesiones sin que nadie lo vea.
9. Arma el informe, registra el lote, escribe las cesiones nuevas con `saveAll` y cierra las que corresponden con `VehicleFundAssignment.close`. Cada cierre deja su fila en la bitácora con `ChangeLogCommandService.recordChanges`, en una llamada: el período anterior y el nuevo, como el cierre a mano, y como motivo «Carga de garantía mobiliaria del {fecha de la lista}: la placa no figura en la lista».

**Las cesiones a mano también se cierran** si su placa no figura (D-209): la lista es la fuente de lo que el fondo tiene en garantía.

**El constructor de la carga.** `VehicleFundAssignment` suma un constructor de alta por carga, con `importBatchId`. El de alta a mano lo deja vacío, como hoy (`fleet/domain/model/aggregates/VehicleFundAssignment.java`), y la columna sigue sin poder actualizarse (`VehicleFundAssignmentPersistenceEntity.java:56`, `updatable = false`).

### 2.7 `fleet`: la fachada para la carga inicial

`FleetContextFacade`, que abre la fase 2 (su §2.5.1), suma dos miembros. No cambia los que ya tiene.

```java
/**
 * Qué es cada VIN para la carga inicial de una empresa. Eleva el alcance una sola vez, y de otra empresa
 * responde solo que el VIN es suyo. No bloquea: el registro vuelve a validar dentro de su transacción.
 */
Map<String, MigratedUnitMatch> matchMigratedUnits(Long companyId, Set<String> vins);

/**
 * Da de alta las unidades de una factura migrada, cada una con su habilitación y en su estado, por el
 * camino «Migración inicial» (D-202). Propagation.MANDATORY. Valida todas antes de escribir; si una falla,
 * no escribe ninguna y responde todas las que fallan en context.rows. Firma la cuenta de la sesión. No pide
 * stock.crear_vehiculo ni los permisos de la habilitación: quien llama ya exigió stock.cargar_migracion_inicial.
 */
Result<List<MigratedUnit>, ApplicationError> registerMigratedUnits(RegisterMigratedUnitsRequest request);

public enum MigratedUnitMatchKind { NEW, MIGRATED, REGISTERED, OTHER_COMPANY }

public record MigratedUnitMatch(String vin, MigratedUnitMatchKind kind, @Nullable Long vehicleId) {}

public record RegisterMigratedUnitsRequest(Long companyId, Long commercialBrandId, @Nullable Long locationId,
        List<MigratedUnitLine> lines) {}

public record MigratedUnitLine(int row, String vin, @Nullable String engineNumber, Long modelId, int modelYear,
        Long colorId, int legacyNumber, MigratedEnablement enablement) {}

public record MigratedEnablement(
        @Nullable LocalDate arrivalDate,                                 // vacía: no ingresó
        @Nullable String titleNumber, @Nullable LocalDate titleDate,
        @Nullable LocalDate tiveIssueDate, boolean hasTive,              // D-206: sin archivo
        @Nullable String plate, @Nullable LocalDate platesReceivedDate,
        @Nullable LocalDate gpsInstalledOn,                              // D-206: sin serial
        @Nullable LocalDate screenInstalledOn, boolean screenFromFactory,
        @Nullable MigratedSoat soat,                                     // D-206: sin archivo
        @Nullable MigratedKey duplicateKey,                              // D-206: etiqueta provisional
        @Nullable LocalDate deliveredOn,
        @Nullable LocalDate readyOn) {}                                  // columna 51

public record MigratedSoat(LocalDate startDate, LocalDate endDate) {}
public record MigratedKey(String tagCode, String storageGroup, @Nullable String serial) {}
public record MigratedUnit(int row, Long vehicleId, UUID vehiclePublicId, String statusCode) {}
```

`fleet` recibe fechas y etiquetas explícitas. Lo que el Excel no trae lo deduce `StockRowMapper`, de `procurement`, porque es una lectura del formato ajeno (D-206, PRC-R154):
- **el ingreso:** la fecha más temprana entre el GPS (19), la pantalla (21), la primera entrega (23) y la declaración de stock (51), o la de la factura (36) si no hay ninguna; así ninguna instalación queda antes del ingreso (D-186);
- **las placas físicas:** la más temprana entre la primera entrega y la declaración de stock, o la del título (15);
- **la etiqueta de la llave:** el código de la marca comercial y el `N°`, unidos por un guion (`LIZZO-1050`), con `holder = empresa`;
- **el propietario** (50) no se lee: la unidad entra como de la empresa, y su traspaso se registra después (D-20).

**`matchMigratedUnits`:** `NEW`, el VIN no existe; `MIGRATED`, existe en la empresa con `legacy_number`, que solo pone esta carga (FLT-R11, `Vehicle.java:50` y `:61`); `REGISTERED`, existe en la empresa sin él, porque nació con una factura o en Inventario; `OTHER_COMPANY`, existe en otra empresa.

**`registerMigratedUnits`** corre en `MigratedUnitRegistrations` (`application/internal/commandservices`), sobre las piezas de las fases 2 y 3:

1. **Referencias, en lote:** la marca comercial, en modo compartido y de la empresa (`fetchCommercialBrandReferencesByIdsForShare`, fase 2); la ubicación, de la empresa y activa, si alguna línea ingresó; los modelos y colores, activos con su marca y su tipo (D-108); el perfil de habilitación de cada modelo (`fetchEnablementProfilesByModelIds`, fase 3); los tipos de documento del país con código `TIVE` y `SOAT`; los equipos `GPS` y `PANTALLA`, de código fijo (D-74); los caminos «Migración inicial».
2. **Validación de cada línea**, sin escribir, con las reglas que la fase 3 aplica a mano:
   - el título y su fecha (FLT-R64);
   - la placa: exige el título y la TIVE de la línea (FLT-R65), se normaliza y no se repite en el país, con una elevación que solo responde sí o no, como el alta (FLT-R01);
   - las placas físicas exigen la placa (FLT-R66);
   - el ingreso exige la ubicación;
   - el GPS y la pantalla exigen el ingreso y no son anteriores a él (FLT-R76);
   - el SOAT vence después de empezar (FLT-R68);
   - ninguna fecha es futura en la zona de la empresa (D-186), tampoco la de entrega.
3. **La pantalla:** se instala solo si el perfil del modelo la exige (`requiredEquipments` trae `PANTALLA`). Si la exige y la línea dice «de fábrica», la línea falla: el catálogo y el Excel no coinciden. Si no la exige, la columna se ignora: la guía la prellena con `SI` en los autos que la traen de fábrica.
4. **El estado** (D-202), con `MigrationTargetPolicy` (`domain/services`, pura):
   - entregada (`deliveredOn`): **Activo**;
   - si no, `EnablementPolicy.evaluate` sobre los hechos de la línea —ingresó, documentos que bloquean y aplican, equipos que exige el modelo, placas físicas—, armados en memoria con el mismo `EnablementFacts` de la fase 3: completa (`isComplete()`, D-183), **Disponible**;
   - si no, **En habilitación**.
5. **La escritura**, en este orden:
   - cada unidad, con `Vehicle.migrated(…)`: una fábrica nueva, la única que recibe `legacyNumber`, los hitos y el estado elegido;
   - su única fila del historial, por el camino «Migración inicial» hacia ese estado (§2.7.1);
   - la TIVE y el SOAT, sin archivo, rotulados «Migrado, sin archivo» (D-206): la TIVE con su emisión en la fecha del título, y el SOAT con su inicio y su fin;
   - el GPS, sin serial (D-206), y la pantalla, con costo 0 en la moneda funcional de la empresa (D-187) y `markEquipmentTypeInUse`;
   - la llave duplicada, con su etiqueta provisional, su caja y su serial (D-206); una etiqueta que ya existe en la empresa hace fallar la línea (`VEHICLEKEY_CONFLICT`);
   - `markModelsInUse` de la fase 2, una vez por lote.

   Nada deja fila en la bitácora: son altas (§13.1, «Qué no cubre»).
6. **La prueba de coherencia:** después de escribir, `VehicleEnablementFactsReader` y `EnablementPolicy` responden para cada unidad lo mismo que decidió el paso 4 (FLT-R123).

Un fallo de la base en una carrera —el VIN o la placa que otra carga tomó mientras tanto— marca la transacción para revertir, como `discardWrites()` del alta, y responde el código de su recurso (`VEHICLE_VIN_CONFLICT`, `VEHICLE_PLATE_CONFLICT`) con `context.rows`.

#### 2.7.1 El camino «Migración inicial»

D-202 y §8.3 del diccionario lo agregan: `(alta) → En habilitación, Disponible o Activo`. Choca con dos reglas que hoy protegen el alta:
- **Un solo camino de alta abierto por estado.** Lo exige `status_change_reasons_registration_uq` (V48) y lo revisa `StatusChangeReasonCommandServiceImpl.java:90-94`. En habilitación ya tiene «Registro de compra».
- **Un camino de alta lleva al estado inicial** (`StatusChangeReasonCommandServiceImpl.java:73-75`). Disponible y Activo no lo son.

**Propuesta:**
- los tres caminos llevan el código reservado `MIGRACION_INICIAL` (`StatusChangeReason.INITIAL_MIGRATION_CODE`);
- las dos reglas los dejan fuera: el índice de V48 se redefine con `AND code <> 'MIGRACION_INICIAL'` (§3.5), y `findRegistrationTo`, que usa el alta de la fase 2, no los devuelve;
- la API no acepta ese código en un camino nuevo ni lo deja cambiar (`422 STATUSCHANGEREASON_CODE_RESERVED`), como el motivo automático (D-110);
- su nombre se corrige y el camino se cierra o se reabre en Catálogos › Estados y motivos, como cualquier camino (D-19). Cerrado, la carga inicial responde `422 STATUSCHANGEREASON_INITIAL_MIGRATION_MISSING`;
- no nombra permiso: un camino de alta no lo hace (V48), y la carga inicial pide `stock.cargar_migracion_inicial` en su ruta.

**El momento de la fila del historial:**
- **Activo:** la fecha de primera entrega, a las 00:00 en la zona de la empresa (D-202). Es la entrega que leen el estado Entregado y la brecha de días (§11.9).
- **Disponible:** la declaración de stock (columna 51), a las 00:00 en la zona de la empresa, si viene; si no, el momento de la carga. La recomendación de P2 lo proponía y D-206 la aceptó, aunque su texto no lo nombra: D9 propone sumarlo a §11.8 del diccionario.
- **En habilitación:** el momento de la carga.

`changed_by` es la cuenta de la sesión. El camino no tiene tolerancia, así que la fila no lleva vencimiento.

#### 2.7.2 Lo que la carga inicial deja para después (D-206)

- **El archivo de un documento migrado** se sube con la ruta de la fase 3 (`POST /vehicles/{id}/documents`): el documento nuevo queda vigente y el migrado se conserva, como todo reemplazo (D-28). No hace falta nada nuevo.
- **El serial de un GPS migrado** se completa una sola vez, con `PUT /vehicle-equipments/{id}/serial` **(propuesta)** y `CompleteVehicleEquipmentSerialCommand(UUID equipmentId, String serial)` en `VehicleEquipmentCommandServiceImpl`:
  - exige `habilitacion.gestionar_equipos` donde está la unidad, como las demás escrituras de equipos (FLT-R86 de la fase 3);
  - bloquea la unidad y después el equipo, en el orden de la fase 3 (su 2.6);
  - procede solo si el serial está vacío; si no, `422 VEHICLEEQUIPMENT_SERIAL_ALREADY_SET`. Un serial mal escrito se corrige reemplazando el equipo;
  - el serial tiene hasta 100 caracteres, recortado, como el de la instalación;
  - deja una fila en la bitácora, con `vehicle_equipments`, sin motivo.

  `VehicleEquipment.completeSerial(String)` es la única escritura del serial fuera del alta, y `allowedActions` del equipo suma `completeSerial`.

### 2.8 `procurement`: la carga inicial

`InitialMigrationImportCommandServiceImpl.handle(ImportInitialMigrationCommand(UUID companyId, UUID commercialBrandId, @Nullable UUID locationId, String fileName, byte[] bytes))`. **No es transaccional:** orquesta una transacción por factura, una por nota, una por pago y una para el lote, con `TransactionTemplate` y `PROPAGATION_REQUIRES_NEW`. `DeadlockRetry` lo envuelve por su nombre, pero no repite nada, porque ninguna excepción sale de él (`DeadlockRetry.java`: solo la operación más externa se repite). Cada transacción interna que pierde un interbloqueo se repite una vez desde el orquestador, con `DeadlockRetry.isDeadlock`, que pasa de privado del paquete a público.

**Los pasos:**

1. **Lectura** (§2.3) con la hoja de 52 columnas (§5.4), y `StockRowMapper` (`application/internal/commandservices`) traduce cada fila a un `StockRow` (`domain/model/valueobjects`). Es el adaptador del formato ajeno (mapa §6.5): ninguna otra clase conoce las columnas del Excel. Un error de celda deja la fila marcada y sigue.
2. **Alcance:**
   - la empresa (`404`);
   - `stock.cargar_migracion_inicial` en toda la empresa y en la marca comercial elegida (`AccessScope.actsOnWholeCompanyBrand`, línea 231; `PermissionCodes.Stock.LOAD_INITIAL_MIGRATION`, línea 116), y `habilitacion.ver_cargas` (`403`);
   - la empresa activa;
   - la marca de la empresa y activa (`404 COMMERCIALBRAND_NOT_FOUND`, `422 COMMERCIALBRAND_INACTIVE`);
   - la ubicación, si viene, de la empresa y activa (`404 LOCATION_NOT_FOUND`, `422 LOCATION_OTHER_COMPANY`, `LOCATION_INACTIVE`);
   - el candado de la carga (SHR-R10).
3. **Los grupos** (`StockRowGroups`, `domain/services`, pura): una factura es el par (proveedor tal como lo escribe la columna 11, normalizado; número de la columna 35, normalizado con `InvoiceNumber.normalize` de la fase 2). Antes de escribir nada:
   - un VIN repetido en el archivo hace fallar los grupos de todas sus filas;
   - un `N°` repetido en el archivo, igual;
   - las filas de un grupo que no coinciden en fecha, moneda o TC hacen fallar el grupo (PRC-R146);
   - un grupo de más de 100 filas falla (D-160).
4. **Catálogos, en una lectura por clase** (`InitialMigrationCatalogs`):
   - **proveedores** de la empresa por razón social o nombre comercial, normalizados con `fold_search_text` (V86) y los espacios colapsados: uno solo, activo; ninguno, `SUPPLIER_NOT_FOUND`; dos, `INITIALMIGRATION_SUPPLIER_AMBIGUOUS`;
   - **marcas y modelos** del vehículo por nombre, el modelo dentro de su marca;
   - **colores** por nombre;
   - **la moneda** por su código ISO (D-13).

   Lo que no está, o está inactivo, hace fallar el grupo (D-177, D-108).
5. **VIN:** `matchMigratedUnits` en una llamada. Por grupo:
   - todos `MIGRATED`, y la factura del proveedor con ese número ya registrada con exactamente esas unidades: **sin cambios**;
   - alguno `REGISTERED`: falla el grupo (`INITIALMIGRATION_VIN_REGISTERED`). La unidad ya está en el sistema sin venir de la migración, y su factura se registra a mano, que la vincula por su VIN (D-153);
   - alguno `OTHER_COMPANY`: falla (`VEHICLE_VIN_CONFLICT`, sin revelar la otra empresa);
   - alguno `MIGRATED` en un grupo con alguno `NEW`, o la factura ya registrada con otras unidades: falla (`INITIALMIGRATION_INVOICE_REGISTERED`).
6. **Cada factura, en su transacción** (`MigratedInvoiceWriter`). Toma los candados en el orden de la fase 2 (su §2.4): la empresa y la marca en modo compartido, el proveedor y la moneda en modo compartido.
   - Llama a `registerMigratedUnits`.
   - Arma la factura con el constructor de registro de `PurchaseInvoice` (fase 2): tipo `factura`; el número; la fecha; la moneda; **la tasa de la fila** y, como día de la tasa, la fecha de la factura (D-177; en la moneda funcional, 1, D-169); la tasa de impuesto del país (`CountrySummary.taxRate`); sin ISC; sin orden; sin archivo ni XML; y una línea por fila, con su unidad, modelo, año, marca y precio.
   - La guarda. El constructor ya exige la suma, montos mayores que cero y su escala, y de 1 a 100 líneas.
   - **No revisa los 20 días** de la factura antigua ni pide `compras.registrar_factura` (D-177, D-202).
   - Un fallo de la base —el número o la unidad que otra factura tomó mientras tanto, por los únicos parciales de la fase 2— lo traduce `UniqueViolations` al código de la fase 2, y el grupo falla entero.
7. **Cada nota de crédito, en su transacción** (`MigratedCreditNoteWriter`), después de todas las facturas. Una nota es el par (proveedor, número de la columna 38 normalizado) con las filas que lo nombran; sus líneas son la unidad y el valor de la columna 40 de cada fila.
   - Un valor negativo es una **nota de débito**: va al informe y no se registra (D-23, D-177).
   - Las facturas de la nota existen y tienen esas unidades vigentes. Si alguna falló, la nota falla con `INITIALMIGRATION_CREDIT_NOTE_INVOICE_FAILED`.
   - Se registra con el constructor de registro de `CreditNote`, de la fase 4: misma moneda que sus facturas; fecha no futura ni anterior a sus facturas, y sin límite de 20 días (D-205); cada línea con la tasa de su factura (D-192); hasta 100 líneas (D-204); el total, la suma de sus líneas; el motivo, `DESCUENTO_CAMPANA`, sin mes de campaña (D-207); sin archivo; desde el 01/01/2027, una sola factura. Los códigos `CREDITNOTE_*` de la fase 4 van al informe.
   - **Una nota que dejaría una línea en cero falla** (`INITIALMIGRATION_CREDIT_NOTE_ZEROES_LINE`): anular una unidad o dejarla sin factura vigente depende del motivo (D-191), que el Excel no trae, así que se registra a mano.
   - Ya registrada con las mismas líneas: sin cambios. Con otras: falla (`INITIALMIGRATION_CREDIT_NOTE_REGISTERED`).
8. **Cada bloque de pago, en su transacción** (`MigratedPaymentWriter`), después de todas las notas, porque lo que se debe de una factura ya descuenta sus notas (D-190). Un bloque es el par (proveedor, texto de la columna 42 normalizado con `OperationNumber` de la fase 5): el pago segmentado `PAGO1/PAGO2/PAGO3` es un solo bloque, porque el Excel guarda solo su total (guía, §7.3).
   - Sus filas coinciden en la fecha (43), el monto (44) y la moneda (46); si no, falla (`INITIALMIGRATION_PAYMENT_INCONSISTENT`).
   - Sus facturas entraron; si no, falla (`INITIALMIGRATION_PAYMENT_INVOICE_FAILED`).
   - Se registra con el constructor de registro de `SupplierPayment`, de la fase 5 (su §2.5), que ya exige sus reglas: de 1 a 100 aplicaciones, ningún destino dos veces, la suma de los equivalentes no mayor que el monto y, con imputaciones, su suma igual a lo cubierto. Los datos del pago (D-207):
     - **la fecha**, la del primer voucher (43), no futura (su P7); puede ser anterior a la factura;
     - **el monto**, el acumulado del bloque (44), en la moneda de sus facturas;
     - **el método**, `TRANSFERENCIA`, que exige el número de operación, y **el número**, el texto de la columna 42 normalizado con `OperationNumber`;
     - **la tasa del pago**, 1 en la moneda funcional (D-169); en otra, la de venta vigente para el día del pago (`fetchEffectiveRate`, con su día), y si el catálogo no tiene ninguna, el TC de las filas del bloque, con la fecha del pago como su día (D-177). Si en ese caso las filas traen dos TC distintos, el bloque falla (`INITIALMIGRATION_PAYMENT_INCONSISTENT`, columna TC);
     - **una aplicación por factura**, en la moneda de la factura, que es la del pago, con la tasa del pago como su tasa (su §2.4 y su P2), sin retención;
     - fuera del canal de dealers, cada aplicación reparte el monto en proporción al valor del vehículo (D-199) de las unidades del bloque en esa factura, con los céntimos en la primera, como §10.3, y **sin imputaciones**: la fase 5 reparte lo aplicado en proporción a lo que le falta a cada unidad (su P6);
     - en el canal de dealers (`suppliers.is_dealer`, D-197), **una imputación por unidad**, por su valor menos la diferencia por inicial de su fila (columna 45), con `settlesUnit` verdadero; cada aplicación es la suma de las imputaciones de sus unidades, y los céntimos que separan esa suma del monto del bloque van a la primera unidad, como §10.3;
     - si una aplicación supera lo que se debe de su factura (D-190) o una imputación lo que le falta a su unidad (su P6), el bloque falla con el código de la fase 5 (`SUPPLIERPAYMENT_EXCEEDS_OWED`, `SUPPLIERPAYMENT_ALLOCATION_EXCEEDS_UNIT`).
   - **El `requestId`** es un UUID versión 5 de la empresa, el proveedor y el número de operación, y la fase 5 lo trata como cualquier envío (su §4.6): volver a subir el bloque encuentra el mismo pago con el mismo contenido y cuenta sin cambios; un pago con ese `requestId` y otro contenido (`SUPPLIERPAYMENT_REQUEST_CONFLICT`), o un pago a mano con ese número de operación (`SUPPLIERPAYMENT_OPERATION_CONFLICT`), hace fallar el bloque, que el informe nombra `INITIALMIGRATION_PAYMENT_REGISTERED`.
   - **No pide** `pagos.registrar_pago`, `pagos.ver` ni `compras.ver`: basta el permiso de la carga, como en las facturas (D-202; recomendación de P5, aceptada por D-207). Los demás `SUPPLIERPAYMENT_*` de la fase 5 van al informe.
9. **El lote**, en su transacción: el informe, los conteos de §2.4 y `ImportBatchCommandService.register`. Se registra también si todo falló, para que el informe se pueda descargar.
10. **La respuesta:** el lote y sus efectos: facturas, unidades por estado, notas y pagos.

Las facturas, las notas y los pagos migrados implementan `CurrencyUsage` con los adaptadores de las fases 2, 4 y 5: fijan los decimales de su moneda, sin nada nuevo.

### 2.9 Transacciones, candados y concurrencia

| Operación | Transacciones | Candados, en orden |
|---|---|---|
| Impuesto vehicular | Una | El de la carga; las unidades de las placas, por `id` |
| Garantía mobiliaria | Una | El de la carga; el fondo en modo compartido; las unidades, por `id` |
| Carga inicial, cada factura | Una por factura | Empresa y marca, proveedor y moneda en modo compartido; las unidades nuevas no existen todavía y el único del VIN las protege |
| Carga inicial, cada nota | Una por nota | Proveedor y moneda en modo compartido; las facturas de la nota, por `id`, como la fase 4 |
| Carga inicial, cada pago | Una por pago | El `requestId`, el proveedor y lo demás en el orden de la fase 5 (su §2.7) |
| Carga inicial, el lote | Una | — |

**El candado de la carga** (SHR-R10): `pg_try_advisory_xact_lock(hashtext('import:' || tipo), company_id::integer)` al empezar la transacción del impuesto y de la garantía. Si otra carga del mismo tipo y empresa lo tiene, responde `409 IMPORTBATCH_CONFLICT` sin esperar: una espera larga terminaría en el corte de Cloudflare. La carga inicial, que corre en varias transacciones, toma la variante de sesión (`pg_try_advisory_lock`) en una conexión propia del orquestador, la suelta en `try/finally` al terminar y responde lo mismo. Dos cargas de empresas distintas no se esperan.

---

## 3. Esquema

### 3.1 Numeración

Las migraciones se numeran al integrar en `develop`, después de las de las fases 2 a 4 (plan §4). Aquí son relativas. Cada una abre como V90: con el alcance `SYSTEM` de su transacción (`set_config('app.access_scope', 'SYSTEM', true)`) y una espera de candado de a lo sumo 5 s (`lock_timeout`). Una que encuentra datos que no cumplen se detiene diciéndolo, como V84.

| Migración | Contexto | Qué hace |
|---|---|---|
| V(n) | `iam` | Los tres permisos y sus nueve marcas |
| V(n+1) | `shared` | `import_batches` |
| V(n+2) | `fleet` | La llave de `vehicle_fund_assignments.import_batch_id`; `vehicle_tax_declarations` |
| V(n+3) | `fleet` | Los caminos «Migración inicial» y el índice de V48 |
| V(n+4) | `fleet` | Los documentos migrados sin archivo (D-206) |

### 3.2 V(n) — Los permisos (`iam`)

```sql
INSERT INTO permissions (code, module, description, category)
VALUES ('habilitacion.cargar_archivo_sat', 'habilitacion', 'La declaración del impuesto vehicular', 'acciones_especiales'),
       ('habilitacion.ver_cargas', 'habilitacion', 'Las cargas de archivos y sus errores', 'ver'),
       ('fondos.cargar_archivo_garantia', 'fondos', 'El archivo de garantía mobiliaria', 'acciones_especiales')
ON CONFLICT (code) DO NOTHING;

INSERT INTO role_permissions (role_id, permission_id, granted_by)
SELECT role.id, permission.id, system_account.id
FROM (VALUES ('SUPERADMIN', 'habilitacion.cargar_archivo_sat'),
             ('ADMIN_EMPRESA', 'habilitacion.cargar_archivo_sat'),
             ('SUPERADMIN', 'habilitacion.ver_cargas'),
             ('ADMIN_EMPRESA', 'habilitacion.ver_cargas'),
             ('OPERACIONES', 'habilitacion.ver_cargas'),
             ('CONTABILIDAD', 'habilitacion.ver_cargas'),
             ('SUPERADMIN', 'fondos.cargar_archivo_garantia'),
             ('ADMIN_EMPRESA', 'fondos.cargar_archivo_garantia'),
             ('CONTABILIDAD', 'fondos.cargar_archivo_garantia')) AS matrix (role_code, permission_code)
JOIN roles role ON role.code = matrix.role_code AND role.is_system AND role.company_id IS NULL
JOIN permissions permission ON permission.code = matrix.permission_code
CROSS JOIN (SELECT id FROM users WHERE email = 'system@millennium.ky') AS system_account
ON CONFLICT DO NOTHING;
```

- Las descripciones y las categorías son las de §5.6, sin la referencia entre paréntesis, como V79. Las marcas son las de la matriz: **nueve**, firmadas por `SYSTEM`.
- **`stock.cargar_migracion_inicial` ya existe:** lo sembró V46 con sus dos marcas, superadministrador y administrador de empresa, que son las de la matriz. Un bloque `DO` al final verifica las nueve marcas nuevas y esas dos, y se detiene nombrando la que falte, como V59.
- Ningún rol propio los recibe: no reemplazan a un permiso grueso.
- `PermissionCodes` suma, en el mismo commit (`PermissionCatalogConsistencyTests`): `Enablement.LOAD_SAT_FILE`, `Enablement.VIEW_IMPORTS` y `Funds.LOAD_GUARANTEE_FILE`. El grupo `Enablement` lo crea la fase 3.

### 3.3 V(n+1) — `import_batches` (`shared`)

```sql
CREATE TABLE import_batches
(
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id         UUID         NOT NULL DEFAULT uuidv7(),
    company_id        BIGINT       NOT NULL REFERENCES companies (id),
    import_type       VARCHAR(30)  NOT NULL,
    file_name         VARCHAR(255) NOT NULL,
    file_path         TEXT         NOT NULL,
    rows_total        INTEGER      NOT NULL,
    rows_ok           INTEGER      NOT NULL,
    rows_failed       INTEGER      NOT NULL,
    error_report_path TEXT,
    imported_by       BIGINT       NOT NULL REFERENCES users (id),
    imported_at       TIMESTAMPTZ  NOT NULL DEFAULT now(),

    CONSTRAINT import_batches_public_id_uq UNIQUE (public_id),
    -- La carga y su empresa: el destino de las llaves compuestas de las declaraciones y las cesiones
    CONSTRAINT import_batches_id_company_uq UNIQUE (id, company_id),
    -- Lista cerrada en minúsculas (D-133); un tipo nuevo llega con una migración que amplía el CHECK
    CONSTRAINT import_batches_import_type_ck
        CHECK (import_type IN ('impuesto_vehicular', 'garantia_mobiliaria', 'migracion_inicial')),
    CONSTRAINT import_batches_file_name_ck CHECK (length(btrim(file_name)) > 0),
    -- Las filas sin cambios no se guardan: se calculan (§2.2 del diccionario)
    CONSTRAINT import_batches_rows_ck
        CHECK (rows_total > 0 AND rows_ok >= 0 AND rows_failed >= 0 AND rows_ok + rows_failed <= rows_total),
    -- Una carga con filas fallidas devuelve su archivo de errores (§11.8)
    CONSTRAINT import_batches_report_ck CHECK (rows_failed = 0 OR error_report_path IS NOT NULL)
);

CREATE INDEX idx_import_batches_company_imported ON import_batches (company_id, imported_at DESC);

ALTER TABLE import_batches ENABLE ROW LEVEL SECURITY;
ALTER TABLE import_batches FORCE ROW LEVEL SECURITY;
CREATE POLICY import_batches_select ON import_batches FOR SELECT
    USING (access_scope_reaches(company_id));
CREATE POLICY import_batches_insert ON import_batches FOR INSERT
    WITH CHECK (access_scope_reaches(company_id));
```

- **Solo se inserta.** La cuenta de la aplicación recibe `SELECT` e `INSERT`, como `change_logs` (V59), sin política de `UPDATE` ni de `DELETE`. `scripts/database/hand-over-database.sql` le quita `UPDATE` al entregar una base y suma la tabla a su verificación (hoy, líneas 102 a 151).
- **`file_path` y `error_report_path`** guardan la llave de `kept_files`, no una llave foránea (§2.7 del diccionario). La API nunca las devuelve (convenciones §8).
- **Sin versión de los datos:** ningún documento generado lee la tabla.

### 3.4 V(n+2) — Las declaraciones y la llave de las cesiones (`fleet`)

```sql
-- 1. La cesión nombra su carga (D-133). Hasta hoy ninguna carga existe: si una cesión ya nombra una, se decide
DO
$$
    BEGIN
        IF EXISTS (SELECT 1 FROM vehicle_fund_assignments WHERE import_batch_id IS NOT NULL) THEN
            RAISE EXCEPTION 'V(n+2): an assignment already names a load that does not exist; decide it first';
        END IF;
    END
$$;

ALTER TABLE vehicle_fund_assignments
    ADD CONSTRAINT vehicle_fund_assignments_import_batch_company_fk
        FOREIGN KEY (import_batch_id, company_id) REFERENCES import_batches (id, company_id);
CREATE INDEX idx_vehicle_fund_assignments_import_batch ON vehicle_fund_assignments (import_batch_id)
    WHERE import_batch_id IS NOT NULL;

-- 2. La declaración del impuesto vehicular (§11.7, D-165)
CREATE TABLE vehicle_tax_declarations
(
    id               BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id        UUID        NOT NULL DEFAULT uuidv7(),
    vehicle_id       BIGINT      NOT NULL,
    company_id       BIGINT      NOT NULL REFERENCES companies (id),
    status           VARCHAR(30) NOT NULL,
    declaration_date DATE,
    import_batch_id  BIGINT      NOT NULL,

    CONSTRAINT vehicle_tax_declarations_public_id_uq UNIQUE (public_id),
    CONSTRAINT vehicle_tax_declarations_vehicle_company_fk
        FOREIGN KEY (vehicle_id, company_id) REFERENCES vehicles (id, company_id),
    CONSTRAINT vehicle_tax_declarations_import_batch_company_fk
        FOREIGN KEY (import_batch_id, company_id) REFERENCES import_batches (id, company_id),
    -- Una declaración por unidad y por carga
    CONSTRAINT vehicle_tax_declarations_vehicle_batch_uq UNIQUE (vehicle_id, import_batch_id),
    -- El estado del archivo, recortado y en mayúsculas
    CONSTRAINT vehicle_tax_declarations_status_ck CHECK (length(btrim(status)) > 0 AND status = upper(btrim(status)))
);

-- La última declaración de cada unidad
CREATE INDEX idx_vehicle_tax_declarations_vehicle ON vehicle_tax_declarations (vehicle_id, id DESC);
CREATE INDEX idx_vehicle_tax_declarations_company ON vehicle_tax_declarations (company_id);
```

- **RLS** con las políticas de `select` e `insert` sobre su `company_id`, como V83, y **solo `SELECT` e `INSERT`** para la aplicación: una declaración no se edita ni se borra. Se suma a `hand-over-database.sql`.
- **`vehicles (id, company_id)`** ya es único: lo usan las cesiones (V75).
- **Sin `created_at`:** el cuadro de §11.7 no lo pide. El momento es el de su carga.

### 3.5 V(n+3) — Los caminos «Migración inicial» (`fleet`)

```sql
-- Un camino de alta abierto por estado (V48), sin contar los de la carga inicial
DROP INDEX status_change_reasons_registration_uq;
CREATE UNIQUE INDEX status_change_reasons_registration_uq ON status_change_reasons (to_status_id)
    WHERE from_status_id IS NULL AND is_active AND code <> 'MIGRACION_INICIAL';

INSERT INTO status_change_reasons (from_status_id, to_status_id, code, name)
SELECT NULL, status.id, 'MIGRACION_INICIAL', 'Migración inicial'
FROM vehicle_statuses status
WHERE status.code IN ('HABILITACION', 'DISPONIBLE', 'ACTIVO')
ON CONFLICT ON CONSTRAINT status_change_reasons_path_code_uq DO NOTHING;
```

- Antes de insertar, un bloque `DO` se detiene si falta alguno de los tres estados. Después, verifica los tres caminos abiertos y que ninguno nombra permiso.
- `status_change_reasons_path_code_uq` es `UNIQUE NULLS NOT DISTINCT` (V47:68), así que el `ON CONFLICT` reconoce el camino de alta.
- La tabla es un catálogo general sin RLS ni versión de los datos (V47, especificación de habilitación, 3.3).

### 3.6 V(n+4) — Los documentos migrados sin archivo (`fleet`, D-206)

```sql
ALTER TABLE vehicle_documents
    ALTER COLUMN file_path DROP NOT NULL,
    ALTER COLUMN file_name DROP NOT NULL,
    ALTER COLUMN mime_type DROP NOT NULL,
    -- Sin archivo, ninguno de sus tres datos: solo la carga inicial escribe un documento así (D-206)
    ADD CONSTRAINT vehicle_documents_file_all_or_none_ck
        CHECK ((file_path IS NULL) = (file_name IS NULL) AND (file_path IS NULL) = (mime_type IS NULL));
```

- La API de la fase 3 sigue exigiendo el archivo: solo `MigratedUnitRegistrations` construye un `VehicleDocument` sin él, con una fábrica propia (`VehicleDocument.migratedWithoutFile`). El cuadro de §11.3 todavía marca las tres columnas como obligatorias; su regla nueva (D-206) es la que manda, y el cierre propone ajustar la columna *Nulo*.
- El serial del GPS ya admite vacío en la base (§11.4): la exigencia es del dominio (FLT-R77), y D-206 la exceptúa solo en la carga inicial, con `VehicleEquipment.migratedWithoutSerial`.

### 3.7 Lo que comprueban las pruebas del esquema

- `RowLevelSecurityMigrationTests` (`shared`): `import_batches` y `vehicle_tax_declarations` tienen las políticas de V29 y ninguna de `UPDATE` ni de `DELETE`.
- `ImportTablesAreInsertOnlyTests`: la cuenta de la aplicación no actualiza ni borra una fila de las dos tablas.
- `HandOverDatabaseScriptTests`: el script les quita `UPDATE`.
- `FileLoadsMigrationTests`:
  - una cesión que nombra una carga de otra empresa se rechaza;
  - una declaración de una unidad y una carga de empresas distintas se rechaza;
  - dos declaraciones de la misma unidad en una carga se rechazan;
  - el alta de una unidad sigue teniendo un solo camino de alta hacia En habilitación.

---

## 4. Contrato de la API

### 4.1 Rutas

| Método y ruta | Contexto | Permiso | Responde |
|---|---|---|---|
| `GET /imports` | `shared` | `habilitacion.ver_cargas` | Página (§6.2 de las convenciones) o arreglo |
| `GET /imports/{id}` | `shared` | `habilitacion.ver_cargas` | El lote |
| `GET /imports/{id}/error-report` | `shared` | `habilitacion.ver_cargas` | El informe (.xlsx) |
| `GET /imports/{id}/file` **(propuesta)** | `shared` | `habilitacion.ver_cargas` | El archivo cargado, con su nombre original: §11.8 lo guarda para auditar la carga sin pedirlo de nuevo |
| `POST /tax-declaration-imports` | `fleet` | `habilitacion.cargar_archivo_sat` | `201` con el resultado |
| `POST /fund-assignment-imports` | `fleet` | `fondos.cargar_archivo_garantia` | `201` con el resultado |
| `POST /initial-migration-imports` **(propuesta)** | `procurement` | `stock.cargar_migracion_inicial` | `201` con el resultado |
| `GET /vehicles/{id}/tax-declarations` | `fleet` | `stock.ver` | El estado SAT y sus declaraciones |
| `PUT /vehicle-equipments/{id}/serial` **(propuesta)** | `fleet` | `habilitacion.gestionar_equipos` | El equipo, con su serial completado una sola vez (§2.7.2, D-206) |

D-200 cierra §14.18 de las convenciones con su opción A: la carga entra por el contexto dueño de los datos. No hay `POST /imports` ni `DELETE`.

### 4.2 Cada ruta

#### 4.2.1 `GET /imports`

- **Filtros:** `companyId` (obligatorio, como todo recurso de empresa sin anidar), `importType` (`impuesto_vehicular`, `garantia_mobiliaria`, `migracion_inicial`), `from` y `to`, el período `[from, to)` de `importedAt`, y `q`, que busca el nombre del archivo con `fold_search_text`.
- **Orden:** `importedAt` (por omisión, descendente), `importType` y `fileName`. Página de 1 a 200.
- **Alcance:** las empresas donde `habilitacion.ver_cargas` actúa (`companyReach`). Una empresa fuera de él no da filas.
- **Sin exportación:** nadie la pide.

```json
{
  "id": "0193…",
  "companyId": "0192…",
  "importType": "garantia_mobiliaria",
  "fileName": "gm_2026_03.xlsx",
  "rowsTotal": 1250,
  "rowsOk": 12,
  "rowsUnchanged": 1236,
  "rowsFailed": 2,
  "hasErrorReport": true,
  "importedBy": "0191…",
  "importedByName": "Tere Ríos",
  "importedAt": "2026-11-03T15:20:11Z"
}
```

`rowsUnchanged` se calcula (total − entraron − fallaron). `importedBy` y `importedByName` siguen a `changedBy` y `changedByName` de la bitácora (`ChangeLogResource`), resueltos con `AuditActorLookupPort`. Ni el `id` interno ni las llaves de `kept_files` salen del servidor.

#### 4.2.2 `GET /imports/{id}`, `/error-report` y `/file`

- **El lote:** el mismo recurso. Uno de otra empresa, o que quien llama no lee, responde `404 IMPORTBATCH_NOT_FOUND`.
- **`/error-report`:** `Content-Type` de .xlsx y `Content-Disposition: attachment; filename="errores-<tipo>-<aaaa-mm-dd>.xlsx"`. Sin informe, `404 IMPORTBATCH_REPORT_NOT_FOUND`.
- **`/file`:** el archivo con su nombre original, saneado para la cabecera.

#### 4.2.3 Las tres cargas

`multipart/form-data`: el archivo en `file` y los datos en `data`, como JSON (convenciones §8).

| Carga | `data` |
|---|---|
| `POST /tax-declaration-imports` | `{ "companyId": "…" }` |
| `POST /fund-assignment-imports` | `{ "companyId": "…", "fundId": "…", "listDate": "2026-11-03", "confirmClosures": false }` |
| `POST /initial-migration-imports` | `{ "companyId": "…", "commercialBrandId": "…", "locationId": "…" }`. `locationId` es opcional: hace falta solo si alguna fila ingresó |

**Responde** `201` con `Location: /api/v1/imports/{id}` y el resultado:

```json
{
  "batch": { "id": "0193…", "importType": "migracion_inicial", "rowsTotal": 412, "rowsOk": 380, "rowsUnchanged": 20, "rowsFailed": 12, "hasErrorReport": true, "…": "…" },
  "effects": {
    "invoices": 380,
    "units": { "HABILITACION": 41, "DISPONIBLE": 37, "ACTIVO": 302 },
    "creditNotes": 55,
    "payments": 41
  }
}
```

Los efectos no se guardan: los calcula la carga. El impuesto no trae `effects`; la garantía trae `{ "opened": 12, "closed": 3, "closuresFailed": 0, "withoutPlate": 1 }`.

**El orden de las respuestas.** Cada paso responde su primer fallo y no escribe nada:
1. la forma del pedido (`400`): las dos partes, `data` válido, el archivo no vacío;
2. la forma del archivo (`422`): el tipo, la lectura, los encabezados y las filas, sin leer la base;
3. la empresa, el fondo, la marca o la ubicación que no existen o no se ven (`404`);
4. los permisos (`403`);
5. lo inactivo y la fecha de la lista (`422`);
6. el candado de la carga (`409`);
7. la confirmación de los cierres (`422`, solo la garantía);
8. el resultado, con las filas que fallaron en el informe.

**Una carga en la que todas las filas fallan** responde `201` igual: el lote existe y su informe dice por qué. Lo que no forma un lote —un archivo ilegible, sin permiso— responde el error y no registra nada.

#### 4.2.4 `GET /vehicles/{id}/tax-declarations`

Con `stock.ver` donde se ve la unidad, marca incluida (FLT-R61 de la fase 3). Una unidad que no se ve responde `404 VEHICLE_NOT_FOUND`.

```json
{
  "status": { "code": "DECLARADA", "fileStatus": "INSCRITO", "declarationDate": "2026-03-12" },
  "declarations": [
    { "id": "…", "status": "INSCRITO", "declarationDate": "2026-03-12", "importId": "0193…", "importedAt": "2026-03-20T14:02:00Z" }
  ]
}
```

`status.code` es uno de `NO_CORRESPONDE`, `SIN_TIVE`, `PENDIENTE` o `DECLARADA` (§2.5), y `fileStatus` y `declarationDate` vienen solo en `DECLARADA`. Las declaraciones van de la más reciente a la más antigua.

### 4.3 Códigos de error

El estado sale del sufijo (convenciones §7.2): `_NOT_FOUND` es `404`, `_CONFLICT` es `409`, los demás propios, `422`. Los textos van en `error.<código-en-kebab>.message` de los dos `messages*.properties` y en `es.json`.

| Código | Estado | Dónde | Mensaje en español |
|---|---|---|---|
| `IMPORTBATCH_NOT_FOUND` | 404 | `/imports/{id}` y sus archivos | Carga no encontrada |
| `IMPORTBATCH_REPORT_NOT_FOUND` | 404 | `/imports/{id}/error-report` | La carga no tiene archivo de errores |
| `IMPORTBATCH_CONFLICT` | 409 | Las tres cargas | Ya hay una carga de ese tipo en curso para la empresa. Espere a que termine |
| `IMPORTBATCH_FILE_TYPE` | 422 | Las tres cargas | El archivo debe ser un libro de Excel (.xlsx) sin macros |
| `IMPORTBATCH_FILE_UNREADABLE` | 422 | Las tres cargas | No se pudo leer el archivo: está dañado o es demasiado grande por dentro |
| `IMPORTBATCH_HEADERS_MISMATCH` | 422 | Las tres cargas | Los encabezados no son los esperados (`context.expected`, `found`, `missing`, `unexpected`, `sheet`) |
| `IMPORTBATCH_EMPTY` | 422 | Las tres cargas | El archivo no trae filas |
| `IMPORTBATCH_TOO_MANY_ROWS` | 422 | Las tres cargas | El archivo trae más filas de las que se cargan de una vez (`context.maxRows`, `context.rows`). Pártalo en varios |
| `VEHICLEFUNDASSIGNMENT_LIST_DATE_IN_FUTURE` | 422 | Garantía | La fecha de la lista no puede ser posterior a hoy (`context.today`) |
| `VEHICLEFUNDASSIGNMENT_IMPORT_UNCONFIRMED` | 422 | Garantía | La lista cerraría cesiones: confirme la carga (`context.toOpen`, `toClose`, `unchanged`, `failed`, `withoutPlate`) |
| `STATUSCHANGEREASON_CODE_RESERVED` | 422 | Alta y edición de un camino | `MIGRACION_INICIAL` es el código de los caminos de la carga inicial: no se usa en otro ni se cambia |
| `STATUSCHANGEREASON_INITIAL_MIGRATION_MISSING` | 422 | Carga inicial | El camino «Migración inicial» hacia ese estado está cerrado o no existe (`context.status`) |
| `VEHICLEEQUIPMENT_SERIAL_ALREADY_SET` | 422 | `PUT /vehicle-equipments/{id}/serial` | El equipo ya tiene serial: uno mal escrito se corrige reemplazando el equipo |

**Reutilizados:**
- `VALIDATION_ERROR` (400), `ACCESS_DENIED` (403), `CONTENT_TOO_LARGE` (413);
- `COMPANY_NOT_FOUND`, `FUND_NOT_FOUND`, `COMMERCIALBRAND_NOT_FOUND`, `LOCATION_NOT_FOUND`, `VEHICLE_NOT_FOUND` (404);
- `COMPANY_INACTIVE`, `FUND_INACTIVE`, `COMMERCIALBRAND_INACTIVE`, `LOCATION_OTHER_COMPANY`, `LOCATION_INACTIVE` (422).

### 4.4 Códigos de fila

No son respuestas HTTP: van en la hoja Errores del informe, con su mensaje en español. Se reutiliza el código del recurso cuando la regla ya lo tiene.

| Código | Carga | Motivo |
|---|---|---|
| `IMPORTBATCH_CELL_REQUIRED` | Todas | Falta el valor de la columna |
| `IMPORTBATCH_CELL_INVALID` | Todas | El valor no es una fecha, un número, un sí o no, o excede su largo |
| `IMPORTBATCH_ROW_REPEATED` | Todas | La placa, el VIN o el N° se repite en las filas que nombra |
| `VEHICLE_NOT_FOUND` | SAT, garantía | La placa no es de ninguna unidad de la empresa |
| `VEHICLE_ANNULLED` | SAT, garantía | La unidad está anulada |
| `VEHICLETAXDECLARATION_NOT_A_CAR` | SAT | La unidad no es un auto: el impuesto no corresponde |
| `VEHICLETAXDECLARATION_IDENTIFICATION_DOCUMENT_MISSING` | SAT | La unidad no tiene la TIVE vigente: no puede tener declaración |
| `VEHICLETAXDECLARATION_DATE_IN_FUTURE` | SAT | La fecha de inscripción es posterior a hoy |
| `VEHICLEFUNDASSIGNMENT_OTHER_FUND` | Garantía | La unidad está cedida a otro fondo (`fondo`) |
| `VEHICLEFUNDASSIGNMENT_CONFLICT` | Garantía | La cesión nueva se cruzaría con otra de la unidad |
| `VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START` | Garantía, cierres | La cesión empezó en la fecha de la lista o después: no se cierra |
| `SUPPLIER_NOT_FOUND`, `SUPPLIER_INACTIVE`, `INITIALMIGRATION_SUPPLIER_AMBIGUOUS` | Inicial | El proveedor no está, está inactivo o su nombre coincide con dos |
| `VEHICLEMODEL_NOT_FOUND`, `VEHICLEMODEL_INACTIVE`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE`, `COLOR_NOT_FOUND`, `COLOR_INACTIVE`, `CURRENCY_NOT_FOUND`, `CURRENCY_INACTIVE` | Inicial | Lo que el catálogo no tiene o tiene inactivo (D-177, D-108) |
| `PURCHASEINVOICE_NUMBER_FORMAT`, `PURCHASEINVOICE_NUMBER_CONFLICT`, `PURCHASEINVOICE_DATE_IN_FUTURE`, `PURCHASEINVOICE_AMOUNT_SCALE`, `PURCHASEINVOICE_UNIT_CONFLICT` | Inicial | Las reglas de la factura de la fase 2 |
| `INITIALMIGRATION_INVOICE_TOO_MANY_LINES` | Inicial | La factura tiene más de 100 filas (D-160) |
| `INITIALMIGRATION_INVOICE_INCONSISTENT` | Inicial | Las filas de la factura no coinciden en fecha, moneda o TC (`columna`) |
| `INITIALMIGRATION_RATE_NOT_ONE` | Inicial | En la moneda de la empresa el TC es 1 (D-169) |
| `INITIALMIGRATION_INVOICE_FAILED` | Inicial | La factura no entró por la fila que nombra |
| `INITIALMIGRATION_INVOICE_REGISTERED` | Inicial | La factura ya está registrada con otras unidades |
| `INITIALMIGRATION_VIN_REGISTERED` | Inicial | La unidad ya está en el sistema y no vino de la migración: registre su factura a mano |
| `VEHICLE_VIN_CONFLICT` | Inicial | El VIN es de otra empresa, o lo tomó otra carga mientras tanto |
| `INITIALMIGRATION_LEGACY_NUMBER_TAKEN` | Inicial | El N° ya es de otra unidad migrada de la marca |
| `VEHICLE_PLATE_CONFLICT`, `VEHICLE_PLATE_TITLE_MISSING`, `VEHICLE_PLATE_IDENTIFICATION_DOCUMENT_MISSING`, `VEHICLE_PLATES_RECEIVED_WITHOUT_PLATE`, `VEHICLE_MILESTONE_IN_FUTURE`, `VEHICLEEQUIPMENT_NOT_ARRIVED`, `VEHICLEEQUIPMENT_BEFORE_ARRIVAL`, `VEHICLEDOCUMENT_EXPIRY_NOT_AFTER_ISSUE` | Inicial | Las reglas de la habilitación de la fase 3 |
| `VEHICLE_ARRIVED_WITHOUT_LOCATION` | Inicial | La unidad ingresó y la carga no eligió ubicación (el código de la fase 3) |
| `VEHICLEKEY_CONFLICT` | Inicial | La etiqueta provisional de la llave ya existe en la empresa (el código de la fase 3) |
| `INITIALMIGRATION_DELIVERY_DATE_MISSING` | Inicial | `ENTREGADO=SI` sin fecha de primera entrega |
| `INITIALMIGRATION_SCREEN_NOT_FROM_FACTORY` | Inicial | El Excel dice «de fábrica» y el modelo exige instalar la pantalla |
| `INITIALMIGRATION_DEBIT_NOTE` | Inicial | Es una nota de débito: el sistema no las registra (D-23). La unidad entró sin ella |
| `INITIALMIGRATION_CREDIT_NOTE_INCONSISTENT` | Inicial | Las filas de la nota no coinciden en fecha o moneda |
| `INITIALMIGRATION_CREDIT_NOTE_INVOICE_FAILED` | Inicial | La nota descuenta una factura que no entró |
| `INITIALMIGRATION_CREDIT_NOTE_ZEROES_LINE` | Inicial | La nota deja la unidad en cero: regístrela a mano con su motivo (D-191) |
| `INITIALMIGRATION_CREDIT_NOTE_REGISTERED` | Inicial | La nota ya está registrada con otras unidades |
| Los `CREDITNOTE_*` de la fase 4 | Inicial | El número, la fecha y el saldo de la nota |
| `INITIALMIGRATION_PAYMENT_INCONSISTENT` | Inicial | Las filas del bloque de pago no coinciden en fecha, monto o moneda, o en el TC cuando el catálogo no tiene la tasa del día (`columna`) |
| `INITIALMIGRATION_PAYMENT_INVOICE_FAILED` | Inicial | El bloque paga una factura que no entró |
| `INITIALMIGRATION_PAYMENT_REGISTERED` | Inicial | Ya hay un pago del proveedor con ese número de operación y otro contenido |
| Los `SUPPLIERPAYMENT_*` de la fase 5 | Inicial | Lo que se debe, la tasa, el canal de dealers |

### 4.5 Lo que cambia en recursos existentes

| Recurso | Cambio |
|---|---|
| `VehicleFundAssignmentResource` | Suma `importId`, el lote que abrió la cesión, o `null` si fue a mano. Lo resuelve `ImportBatchQueryService.fetchReferencesByIds`, de `shared`, en lote |
| `StatusChangeReasonResource` | Suma `reserved`: verdadero en los caminos `MIGRACION_INICIAL`, cuyo código la pantalla no deja cambiar |
| `VehicleEnablementView` (fase 3) | Suma el estado SAT de la unidad (§2.5) |
| Documentos y equipos de la unidad (fase 3) | `fileName`, `mediaType` y `serial` pueden venir vacíos, y `migrated: true` lo explica (D-206). El equipo suma `completeSerial` a sus `allowedActions` |

---

## 5. Formato de cada archivo

### 5.1 Lo común

- **Formato:** un libro .xlsx, de a lo sumo 10 MB, leído como dice §2.3.
- **Encabezados:** exactos (D-201).
- **Filas:** desde la 2. Una vacía se salta. El número de fila del informe es el que ve Excel.
- **Plantillas:** la de la carga inicial es la del cliente, sin cambios (`STOCK_MILLENNIUM_2026_EXPORTABLE FINAL.xlsx`). Las del SAT y la garantía son las exportaciones de sus plataformas. La pantalla ofrece descargar la de la carga inicial (§7.4).

### 5.2 Impuesto vehicular (SAT)

Encabezados **provisionales**, tomados de la guía (columnas 31 y 32). El perfilado (§1.3, punto 19) fija los reales y la hoja.

| Encabezado | Obligatorio | Destino | Cómo se lee |
|---|---|---|---|
| `PLACA` | Sí | El cruce | Normalizada con `LicensePlate` |
| `ESTADO` | Sí | `vehicle_tax_declarations.status` | Texto recortado, en mayúsculas, hasta 30 caracteres |
| `FECHA DE INSCRIPCIÓN` | No | `vehicle_tax_declarations.declaration_date` | Fecha no futura en la zona de la empresa |

Las demás columnas del archivo se ignoran.

### 5.3 Garantía mobiliaria

Encabezado **provisional**: `PLACA`, obligatorio, normalizada con `LicensePlate`. El fondo y la fecha de la lista los elige quien carga. Las demás columnas se ignoran. El perfilado (§1.3, punto 20) dice si el archivo trae el fondo o una fecha por placa.

### 5.4 Carga inicial: las 52 columnas

Los encabezados son los de la plantilla, en este orden, desde la columna A. *Celda* es el formato que la plantilla le da: entero (`0`), texto (`@`), general, fecha (`dd/mm/yyyy`) o monto (`#,##0.00`).

| # | Encabezado exacto | Celda | Destino | Cómo se lee y qué regla aplica |
|---|---|---|---|---|
| 1 | `N°` | Entero | `vehicles.legacy_number` | Entero mayor que cero, obligatorio, único en el archivo y entre las unidades migradas de la marca (D-18, FLT-R125) |
| 2 | `LLAVE` | Texto | La caja de la llave duplicada (`vehicle_keys.storage_group`), con su etiqueta provisional (D-206) | `NO TIENE` o vacío: sin duplicado. Si no, el grupo (`G01`), hasta 10 caracteres |
| 3 | `PLACA` | Texto | `vehicles.plate` | Normalizada; exige el título y la TIVE (FLT-R65); única en el país. Vacía: sin placa |
| 4 | `VIN` | Texto | `vehicles.vin`; la llave de la idempotencia | 17 caracteres sin I, O ni Q (D-160), obligatorio |
| 5 | `ESTADO DE HABILITACIÓN` | General | **No se importa** | Se calcula (§11.9). El perfilado lo compara con el que calcula el sistema |
| 6 | `MOTOR` | Texto | `vehicles.engine_number` | Opcional, hasta 50 |
| 7 | `MARCA` | General | Con la 8, el modelo | Nombre de la marca del vehículo |
| 8 | `MODELO` | General | `vehicles.model_id` y la línea de factura | Nombre del modelo dentro de su marca; inexistente o inactivo: falla el grupo (D-177, D-108) |
| 9 | `COLOR` | General | `vehicles.color_id` | Por nombre; inexistente o inactivo: falla el grupo |
| 10 | `AÑO` | Entero | `vehicles.model_year` y la línea | De 2017 al año siguiente |
| 11 | `PROVEEDOR` | General | `purchase_invoices.supplier_id`; la llave del grupo | Por razón social o nombre comercial; desconocido: falla el grupo (D-177) |
| 12 | `DUPLICADO DE LLAVES` | General | El serial de la llave duplicada (D-206) | `SI`, `NO` o el serial |
| 13 | `EN PLANTA` | General | `vehicles.arrival_date`, deducida de la fila, y `current_location_id`, la elegida (D-206) | Sí o no. `SI` exige la ubicación elegida |
| 14 | `INMATRICULACIÓN / TÍTULO` | Texto | `vehicles.title_number` | `NO` o vacío: sin título. Hasta 50 |
| 15 | `FECHA DE TÍTULO` | Fecha | `vehicles.title_date`; la emisión de la TIVE (D-206) | Solo con título (FLT-R64); no futura |
| 16 | `TIVE` | General | La TIVE vigente, sin archivo (D-206) | Sí o no |
| 17 | `PLACAS` | General | `vehicles.plates_received_date`, deducida de la fila (D-206) | `SI` exige la placa (FLT-R66) |
| 18 | `GPS` | General | Un GPS instalado, sin serial (D-206) | `SI` exige la fecha 19 y el ingreso |
| 19 | `FECHA DE INSTALACIÓN DE GPS` | Fecha | Inicio del período del GPS | No futura ni anterior al ingreso (D-186) |
| 20 | `PANTALLA` | General | Una pantalla instalada, solo si el modelo la exige | `SI`, `NO` o `NO CORRESPONDE`. Si el modelo no la exige, se ignora |
| 21 | `FECHA DE INSTALACIÓN DE PANTALLA` | Fecha | Inicio del período de la pantalla | `DE FÁBRICA` en un modelo que exige instalarla: falla (FLT-R124) |
| 22 | `ENTREGADO` | General | El estado Activo (D-202) | `SI` exige la fecha 23 |
| 23 | `FECHA DE PRIMERA ENTREGA` | Fecha | El momento de la fila del historial hacia Activo (D-202) | No futura |
| 24 | `BRECHA DE DÍAS DE HABILITACIÓN` | General | **No se importa** | Se calcula (D-199) |
| 25 | `SOAT` | General | El SOAT vigente, sin archivo (D-206) | `SI` exige las fechas 26 y 27 |
| 26 | `FECHA INICIO SOAT` | Fecha | `vehicle_documents.issue_date` | No futura |
| 27 | `FECHA FIN SOAT` | Fecha | `vehicle_documents.expiry_date` | Posterior al inicio (FLT-R68) |
| 28 | `PÓLIZA` | Texto | **No se importa** | Las pólizas son candidato (§11.6) |
| 29 | `FECHA INICIO DE PÓLIZA` | Fecha | **No se importa** | Ídem |
| 30 | `FECHA FIN DE PÓLIZA` | Fecha | **No se importa** | Ídem |
| 31 | `SAT` | General | **No se importa** | Lo trae la carga del impuesto vehicular, que es la fuente (§11.7) |
| 32 | `FECHA DECLARACIÓN SAT` | Fecha | **No se importa** | Ídem |
| 33 | `GARANTÍA MOBILIARIA` | General | **No se importa** | Lo trae la carga de garantía: el Excel no dice el fondo ni desde cuándo |
| 34 | `ORDEN DE COMPRA` | Texto | **No se importa** | Las órdenes del Excel son anteriores al sistema. Si el perfilado encuentra números de órdenes ya registradas, vincularlas es una versión posterior (D-172) |
| 35 | `FACTURA` | Texto | `purchase_invoices.document_number`; la llave del grupo | Normalizado (D-160); en Perú, serie de cuatro y correlativo de hasta ocho dígitos |
| 36 | `FECHA DE FACTURA` | Fecha | `invoice_date` y `exchange_rate_date` | No futura; sin los 20 días (D-177); igual en todo el grupo |
| 37 | `PRECIO DE FACTURA` | Monto | `purchase_invoice_items.invoiced_amount`; su suma, `gross_total` | Mayor que cero, con impuestos (D-159), a lo sumo los decimales de la moneda |
| 38 | `NC / ND` | Texto | `credit_notes.document_number`; con el proveedor, la llave de la nota | Normalizado como el de la factura; `-` o vacío: sin nota |
| 39 | `FECHA NC / ND` | Fecha | `credit_notes.issue_date` | No futura ni anterior a sus facturas; igual en toda la nota |
| 40 | `VALOR NC / ND` | Monto | `credit_note_items.amount`; su suma, `total_amount` | Positivo: nota de crédito. Negativo: nota de débito, al informe (D-23, D-177) |
| 41 | `VALOR DEL VEHÍCULO` | Monto | **No se importa** | Se calcula (§9.6). El perfilado lo compara con el precio menos la nota |
| 42 | `OPERACIÓN DE PAGO` | Texto | `supplier_payments.operation_number`; con el proveedor, la llave del bloque (fase 5) | Normalizado con `OperationNumber`; `PAGO1/PAGO2` es un solo bloque; `-` o vacío: sin pago (PRC-R153) |
| 43 | `FECHA DE PAGO` | Fecha | `supplier_payments.payment_date` | La del primer voucher; no futura; igual en todo el bloque |
| 44 | `MONTO DE PAGO` | General | `supplier_payments.amount`, repartido en sus aplicaciones | El total del bloque, repetido en sus filas (guía, A-05); igual en todo el bloque |
| 45 | `DIFERENCIA POR INICIAL` | General | Solo en el canal de dealers: la imputación de la unidad es su valor menos esta diferencia (D-207) | Fuera de ese canal no se lee: se calcula (D-199) |
| 46 | `MONEDA` | General | `purchase_invoices.currency_id` y la de su nota | Código ISO del catálogo, activo (la guía: `PEN` o `USD`); igual en el grupo |
| 47 | `TC` | Monto | `purchase_invoices.exchange_rate`; la de cada línea de su nota (D-192) | En la moneda funcional, 1 o vacío (D-169). En otra, mayor que cero, hasta seis decimales, igual en el grupo (D-177) |
| 48 | `PRECIO DE COMPRA PEN` | Monto | **No se importa** | Se calcula |
| 49 | `VALOR CONTABLE PEN` | Monto | **No se importa** | Se calcula (D-159) |
| 50 | `PROPIETARIO` | General | **No se importa** | La unidad entra como de la empresa, y su traspaso (D-20) se registra después con su fecha y su motivo, que el Excel no trae (D-206) |
| 51 | `DECLARACIÓN DE STOCK` | Fecha | `vehicle_status_history.changed_at` de la entrada a Disponible; con la 23, deduce el ingreso y las placas físicas | No es un campo: el historial la guarda (§11.9). A las 00:00 en la zona de la empresa (D-206, §2.7.1) |
| 52 | `TIPO DE VEHÍCULO` | General | **No se importa** | Sale del modelo (§7.3 del diccionario). El perfilado lo compara con el del catálogo |

**En resumen:** 38 columnas llegan a un destino del sistema y 14 no se importan:
- se calculan: 5, 24, 41, 48, 49 y 52;
- las traen otras cargas: 31 a 33;
- son candidato: 28 a 30;
- no tienen dónde ir en esta versión: 34, la orden anterior al sistema, y 50, el propietario, que D-206 deja para el traspaso.

La 45 se lee solo en el canal de dealers (D-207), y de la 42 a la 44 llegan en la tercera entrega, con la fase 5 (D-208).

### 5.5 El informe de cada carga

| Carga | Hoja de filas | Llave de la hoja Errores | Hojas propias |
|---|---|---|---|
| Impuesto vehicular | Las filas que fallaron, con los encabezados del archivo del SAT | `Placa` | — |
| Garantía | Las placas que fallaron | `Placa` | Cesiones cerradas |
| Carga inicial | `STOCK`, con los 52 encabezados | `VIN` y `N°` | — |

En la carga inicial, cuando una factura falla, **todas sus filas** van a la hoja `STOCK`. La que causó el fallo lleva su código, y las demás `INITIALMIGRATION_INVOICE_FAILED` con el número de esa fila. Una fila cuya unidad entró pero cuya nota falló también va: subirla de nuevo no repite la unidad (D-201) y vuelve a intentar la nota.

---

## 6. Reglas

Capas y estados según el [README de los contextos](contextos/README.md). Las pruebas del backend van en `src/test/java/.../<contexto>`; las del frontend, junto a cada archivo; las de extremo a extremo, en `e2e/imports.spec.ts`. Ninguna está construida: todas están ❌. `SHR` es el prefijo nuevo de `shared`: hasta hoy ninguna regla de `shared` tenía uno, y el README suma `SHR-` a su lista.

**Los números de `fleet`.** La especificación de pagos propone para la fase 5 FLT-R100 a FLT-R102, o los siguientes libres si otra fase los toma antes (su P12). Esta especificación reserva FLT-R100 a FLT-R127, porque su primera entrega se integra antes que la fase 5 (D-208) y la segunda puede cruzarse con ella; la fase 5 toma desde FLT-R128.

### 6.1 `shared`

| ID | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| SHR-R01 | Toda carga registra su lote una sola vez, al terminar, con su empresa, su tipo, su archivo, sus conteos, su informe, quién y cuándo. La fila no se edita ni se borra | §11.8, D-200 | Base: V(n+1), privilegios `SELECT` e `INSERT`. Servicio: `ImportBatchCommandServiceImpl` | `ImportBatchRegistrationIntegrationTests.aLoadIsRegisteredOnceWithItsFilesAndCounts`; `ImportTablesAreInsertOnlyTests.theApplicationCannotUpdateNorDeleteABatch` | ❌ |
| SHR-R02 | Los conteos cuadran: hay filas, entraron más fallaron no pasa del total, y las sin cambios se calculan. Con filas fallidas hay informe | §11.8, §2.2 | Base: `import_batches_rows_ck`, `_report_ck`. Dominio: `ImportBatch` | `ImportBatchTests.theCountsAddUpAndTheUnchangedAreDerived`, `aBatchWithFailedRowsNeedsItsReport` | ❌ |
| SHR-R03 | El archivo y el informe van a `kept_files` con la empresa de la carga, bajo una llave de su contenido; se escriben dentro de la transacción, y una llave nueva se descarta si se revierte | D-189, D-134, §2.7 | Puerto: `ImportBatchFileStore`, implementado por `documents`. Servicio: `ImportBatchCommandServiceImpl` | `ImportBatchFilesIntegrationTests.theSourceAndTheReportAreKeptWithTheCompanyOfTheLoad`, `aRolledBackLoadDiscardsOnlyTheKeysItWroteFirst` | ❌ |
| SHR-R04 | Las cargas se leen con `habilitacion.ver_cargas` donde actúa en su empresa; la lista se acota con `companyReach`; una carga de otra empresa no existe para quien pregunta; ninguna llave de archivo sale del servidor | §5.6, D-99, §2.5 | Servicio: `ImportBatchAccessPolicy`, `ImportBatchQueryServiceImpl`. Base: RLS | `ImportBatchQueryIntegrationTests.aBatchOfAnotherCompanyIsNotFound`, `theListShowsOnlyTheCompaniesWhereTheReadingActs`; `ImportBatchResourceTests.noStorageKeyLeavesTheServer` | ❌ |
| SHR-R05 | Cargar exige el permiso de la carga y además `habilitacion.ver_cargas` en la empresa: escribir exige leer. Las tres actúan sobre toda la empresa, sin sede | D-134, D-101, §5.6 | Servicio: cada carga, con `ImportBatchAccessPolicy` | `TaxDeclarationImportIntegrationTests.aLoaderWithoutTheReadingIsDenied`; las de §6.2 y §6.3 | ❌ |
| SHR-R06 | El archivo es un .xlsx de a lo sumo 10 MB, el límite del servidor por archivo, verificado por su contenido: ZIP, tipo de libro sin macros, límites de entradas y de tamaño descomprimido, XML sin DTD. Si no, `422` antes de leer filas | D-174 (el mismo criterio), D-201 | Infraestructura: `XlsxContainerCheck`, `FastexcelImportWorkbookReader` | `ImportWorkbookReaderTests.aPdfRenamedXlsxIsRefused`, `aMacroWorkbookIsRefused`, `anInflationBombIsRefusedBeforeInflating`, `aDoctypeIsRefused` | ❌ |
| SHR-R07 | Los encabezados se comparan exactos, normalizados solo en sus espacios; la hoja es la primera que los trae; la carga inicial exige los 52 en orden, y las exportaciones ajenas, los que lee. Una fila vacía se salta; pasar el tope responde `422` | D-201 | Infraestructura: `FastexcelImportWorkbookReader`, `ImportSheetSpec` | `ImportWorkbookReaderTests.theHeadersAreComparedExactlyAfterTheirSpaces`, `aDegreeSignIsNotAnOrdinalSign`, `theSheetIsFoundByItsHeaders`, `anEmptyRowIsSkipped`, `moreRowsThanTheCapAreRefused` | ❌ |
| SHR-R08 | Las celdas: fechas de celda (1900 o 1904) o texto `dd/mm/aaaa` y `aaaa-mm-dd`; números sin separador de miles, redondeados solo en su ruido binario; los centinelas son vacío | §2.2, §2.6 | Aplicación: `ImportCell` | `ImportCellTests.aDateIsNeverReadMonthFirst`, `the1904SystemIsHonoured`, `aThousandsSeparatorIsRefused`, `binaryNoiseIsRoundedAndARealExtraDecimalIsNot`, `theSentinelsAreEmpty` | ❌ |
| SHR-R09 | El informe devuelve las filas fallidas tal como llegaron, bajo los mismos encabezados, y explica cada fallo en español; corregido, se vuelve a subir | §11.8, D-201 | Infraestructura: `FastexcelImportReportWriter` | `ImportReportWriterTests.theFailedRowsGoBackUnderTheSameHeaders`, `aReportReadsBackThroughTheReader`, `eachFailureNamesItsRowColumnAndCode` | ❌ |
| SHR-R10 | Una carga de un tipo a la vez por empresa: la segunda responde `409 IMPORTBATCH_CONFLICT` sin esperar | Plan §6 (riesgo de concurrencia) | Servicio: `pg_try_advisory_xact_lock` y su variante de sesión en la carga inicial | `ImportBatchConcurrencyTests.aSecondLoadOfTheSameTypeAndCompanyAnswersConflict`, `loadsOfTwoCompaniesDoNotWait` | ❌ |
| SHR-R11 | Quién cargó se muestra por el nombre de su colaborador, resuelto en lote | Mapa, decisión 6; D-129 | Servicio: `ImportBatchViewFactory` con `AuditActorLookupPort` | `ImportBatchQueryIntegrationTests.theLoaderIsNamedByTheEmployee` | ❌ |
| SHR-R12 | Los permisos del paquete siguen la matriz de §5.6: tres códigos y nueve marcas; `stock.cargar_migracion_inicial` conserva las dos de V46 | §5.6, D-95 | Base: V(n) | `FileLoadPermissionsMigrationTests.theThreeCodesAndTheirNineMarksFollowTheMatrix`; `PermissionCatalogConsistencyTests` | ❌ |
| SHR-R13 | `import_batches` tiene las políticas de V29 sobre su `company_id` y ninguna de `UPDATE` ni de `DELETE` | §2.5, D-66 | Base: V(n+1) | `RowLevelSecurityMigrationTests`; `ImportBatchRowLevelSecurityTests.aCompanyScopeSeesOnlyItsBatches` | ❌ |

### 6.2 `fleet`

| ID | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| FLT-R100 | El archivo del SAT solo actualiza unidades de la empresa que ya existen, cruzadas por su placa normalizada; nunca crea una. Una placa sin unidad va al informe | §11.7, §11.8 | Servicio: `TaxDeclarationImportCommandServiceImpl` | `TaxDeclarationImportIntegrationTests.aPlateWithoutUnitGoesToTheReportAndNothingIsCreated`, `aPlateWrittenWithHyphensFindsItsUnit` | ❌ |
| FLT-R101 | La declaración solo aplica a autos: el tipo del modelo es `AUTO` | §11.7; guía, columna 31 | Dominio: `TaxDeclarationRowPolicy` | `TaxDeclarationRowPolicyTests.aMotorcycleIsNotDeclared` | ❌ |
| FLT-R102 | Una unidad sin la TIVE vigente no tiene declaración | §11.7 | Dominio: `TaxDeclarationRowPolicy`. Servicio: la TIVE leída en lote | `TaxDeclarationImportIntegrationTests.aCarWithoutTiveIsReportedAndNotDeclared` | ❌ |
| FLT-R103 | Ninguna carga escribe sobre una unidad anulada | D-104, D-182 (fase 2, su FLT-R49) | Dominio: las dos políticas de las cargas | `TaxDeclarationImportIntegrationTests.anAnnulledUnitIsReported`; `GuaranteeImportIntegrationTests.anAnnulledUnitIsReported` | ❌ |
| FLT-R104 | Una declaración guarda el estado del archivo, recortado y en mayúsculas, y la fecha si viene, no futura; se escribe solo si difiere de la última de la unidad; una por unidad y carga; no se edita | §11.7, D-186 | Base: `vehicle_tax_declarations_status_ck`, `_vehicle_batch_uq`, privilegios. Dominio: `VehicleTaxDeclaration` | `TaxDeclarationImportIntegrationTests.anUnchangedDeclarationIsNotWrittenAgain`, `aChangedStatusAddsADeclarationAndKeepsThePrevious` | ❌ |
| FLT-R105 | Una placa repetida en el archivo del SAT o de la garantía hace fallar todas sus filas | Guía, columna 31 («no se espera duplicidad») | Dominio: las dos políticas | `TaxDeclarationRowPolicyTests.aRepeatedPlateFailsEveryRow`; `GuaranteeListPolicyTests.aRepeatedPlateFailsEveryRow` | ❌ |
| FLT-R106 | El estado SAT de una unidad se calcula: `NO_CORRESPONDE`, `SIN_TIVE`, `PENDIENTE` o `DECLARADA` con el estado y la fecha de su última declaración; no se guarda | §11.7, §2.2; guía, columnas 31 y 32 | Dominio: `VehicleTaxStatusPolicy`. Servicio: `GET /vehicles/{id}/tax-declarations` | `VehicleTaxStatusPolicyTests.theFourCodesInTheirOrder`; `VehicleTaxDeclarationQueryIntegrationTests.theLatestDeclarationIsTheStatus` | ❌ |
| FLT-R107 | Las declaraciones llevan la empresa de su unidad, con llaves compuestas hacia la unidad y hacia la carga, Row Level Security, y solo se insertan | §11.7, D-165, §2.5 | Base: V(n+2) | `FleetRowLevelSecurityTests.aCompanyScopeSeesOnlyItsDeclarations`, `aDeclarationOfAnotherCompanyThanItsUnitOrLoadIsRefused` | ❌ |
| FLT-R108 | Cargar el SAT exige `habilitacion.cargar_archivo_sat` y `habilitacion.ver_cargas` en toda la empresa y en todas sus marcas, y la empresa activa | §5.6, D-134, D-84 | Servicio: `actsOnWholeCompanyEveryBrand` | `TaxDeclarationImportIntegrationTests.aBranchOrBrandAssignmentIsDenied`, `anInactiveCompanyIsRefused` | ❌ |
| FLT-R109 | La declaración de una unidad dice de qué carga salió | §11.7 | Servicio: `ImportBatchQueryService.fetchReferencesByIds` | `VehicleTaxDeclarationQueryIntegrationTests.eachDeclarationNamesItsLoad` | ❌ |
| FLT-R110 | La lista de garantía es la de un fondo a una fecha; solo actualiza unidades existentes y nunca crea una | §12.2, §11.8; guía, §8.2 | Servicio: `FundAssignmentImportCommandServiceImpl` | `GuaranteeImportIntegrationTests.aPlateWithoutUnitGoesToTheReportAndNothingIsCreated` | ❌ |
| FLT-R111 | Una placa sin cesión abierta abre una al fondo desde la fecha de la lista, con su carga; con la cesión abierta a ese fondo, queda sin cambios | §12.2, D-30 | Dominio: `GuaranteeListPolicy`, `VehicleFundAssignment` (alta por carga) | `GuaranteeListPolicyTests.aNewPlateOpensAndAnAssignedOneIsUnchanged`; `GuaranteeImportIntegrationTests.theNewAssignmentNamesItsLoad` | ❌ |
| FLT-R112 | Una cesión abierta al fondo cuya unidad tiene placa y no figura en la lista se cierra en la fecha de la lista, con su fila en la bitácora; la historia se conserva. Vale también para las cesiones registradas a mano. La de una unidad sin placa no se cierra y se informa | §12.2, D-30, D-209, §13.1 | Dominio: `GuaranteeListPolicy`. Servicio: `ChangeLogCommandService.recordChanges` | `GuaranteeImportIntegrationTests.aPlateThatLeavesTheListClosesItsAssignmentAndKeepsIt`, `aUnitWithoutPlateIsNotClosed`, `eachClosureLeavesItsRowInTheChangeLog` | ❌ |
| FLT-R113 | Una placa cedida a otro fondo, o cuya cesión nueva se cruzaría con otra, falla y no cambia nada de esa unidad; un cierre que no queda posterior al inicio no se hace y se informa. El fondo y la fecha de la lista quedan en el informe | §12.2, D-133, D-209 | Dominio: `GuaranteeListPolicy`. Base: `vehicle_fund_assignments_no_overlap_excl` | `GuaranteeListPolicyTests.aPlateOfAnotherFundFails`, `aClosureNotAfterTheStartIsReported` | ❌ |
| FLT-R114 | Si la carga cerraría cesiones, exige la confirmación; sin ella responde los efectos y no escribe nada | Especificación de facturas, §4.4 (avisos que se confirman); D-30 | Servicio: `422 VEHICLEFUNDASSIGNMENT_IMPORT_UNCONFIRMED`. Frontend: `guarantee-import-dialog` | `GuaranteeImportIntegrationTests.closuresWaitForTheirConfirmationAndNothingIsWritten` | ❌ |
| FLT-R115 | El fondo es de la empresa, activo y se lee en modo compartido, como toda escritura de cesiones; la fecha de la lista no es futura en la zona de la empresa | §12.1, `contextos/fleet.md` 6 f, §2.6 | Servicio: `FundRepository.findByPublicIdForShare` | `GuaranteeImportIntegrationTests.anInactiveFundIsRefused`, `aListOfTomorrowInLimaIsRefused`, `theFundCodeIsNotCorrectedWhileALoadUsesIt` | ❌ |
| FLT-R116 | Cargar la garantía exige `fondos.cargar_archivo_garantia`, `fondos.ver` y `habilitacion.ver_cargas` en toda la empresa y en todas sus marcas | §5.6, D-134 | Servicio | `GuaranteeImportIntegrationTests.accountingLoadsAndOperationsDoesNot` | ❌ |
| FLT-R117 | Las unidades se bloquean en orden de `id` antes de decidir: una cesión a mano en paralelo espera y ve lo que la carga escribió | Patrones de persistencia | Persistencia: un `SELECT … ORDER BY id FOR NO KEY UPDATE` | `GuaranteeImportConcurrencyTests.aManualAssignmentWaitsForTheLoad` | ❌ |
| FLT-R118 | Una cesión nombra la carga que la abrió, con una llave compuesta con su empresa; la de una cesión a mano queda vacía | §12.2, D-133 | Base: V(n+2). Recurso: `importId` | `FileLoadsMigrationTests.anAssignmentCannotNameALoadOfAnotherCompany`; `VehicleFundAssignmentQueryIntegrationTests.anAssignmentNamesItsLoad` | ❌ |
| FLT-R120 | Los caminos «Migración inicial», hacia En habilitación, Disponible y Activo, llevan un código reservado: no cuentan para el camino de alta único ni para el estado inicial, no nombran permiso, y la API no deja usar ni cambiar su código | §8.3, D-202, D-110 (el mismo criterio) | Base: V(n+3). Dominio: `StatusChangeReason.INITIAL_MIGRATION_CODE`. Servicio: `422 STATUSCHANGEREASON_CODE_RESERVED` | `StatusChangeReasonIntegrationTests.theMigrationCodeIsReserved`; `VehicleRegistrationIntegrationTests.theRegistrationStillTakesThePurchasePath` | ❌ |
| FLT-R121 | `matchMigratedUnits` dice de cada VIN si es nuevo, ya migrado, registrado fuera de la migración o de otra empresa, sin revelar nada de la otra | D-153, D-201 | Fachada: una elevación por llamada | `FleetContextFacadeTests.theMatchTellsMigratedFromRegisteredAndHidesOtherCompanies` | ❌ |
| FLT-R122 | `registerMigratedUnits` valida todas las unidades de la factura antes de escribir y escribe todas o ninguna, con su número del Excel y las reglas del alta (modelo, marca, tipo y color activos) | D-202, D-108, D-18 | Fachada: `MigratedUnitRegistrations`, `MANDATORY` | `MigratedUnitRegistrationsIntegrationTests.aFailureInTheLastLineLeavesNoUnit`, `anInactiveModelIsRefused` | ❌ |
| FLT-R123 | La unidad migrada entra a Activo si se entregó, con la fecha de entrega como momento; si no, a Disponible si no le falta nada; si no, a En habilitación. Después de escribir, el estado de habilitación leído coincide con el decidido | D-202, D-183, §11.9 | Dominio: `MigrationTargetPolicy`, `EnablementPolicy` | `MigrationTargetPolicyTests.deliveredIsActiveCompleteIsAvailableElseEnabling`; `MigratedUnitRegistrationsIntegrationTests.theReadEnablementMatchesTheDecidedStatus`, `theDeliveryMomentIsTheStartOfItsDayInTheCompanyZone` | ❌ |
| FLT-R124 | Los datos de la habilitación migrados cumplen las reglas de la fase 3 —título, placa con título y TIVE, placas físicas con placa, ingreso con ubicación, instalaciones no anteriores al ingreso, fechas no futuras, pantalla solo si el modelo la exige—, salvo dos excepciones de la carga inicial: la TIVE y el SOAT sin archivo, y el GPS sin serial | §11.1, §11.3, §11.4, D-186, D-188, D-206 | Fachada: `MigratedUnitRegistrations` | `MigratedUnitRegistrationsIntegrationTests.aPlateWithoutTitleIsRefused`, `aGpsBeforeTheArrivalIsRefused`, `aFactoryScreenOnAModelThatNeedsOneIsRefused` | ❌ |
| FLT-R125 | El número del Excel es un entero positivo, único entre las unidades migradas de la empresa y la marca **(propuesta)** | D-18 | Fachada: `MigratedUnitRegistrations` | `MigratedUnitRegistrationsIntegrationTests.aLegacyNumberTakenInTheBrandIsRefusedAndAnotherBrandMayRepeatIt` | ❌ |
| FLT-R126 | Una unidad migrada tiene una sola fila en el historial y no deja filas en la bitácora: es un alta | §8.4, §13.1 | Fachada | `MigratedUnitRegistrationsIntegrationTests.aMigratedUnitHasOneHistoryRowAndNoChangeLog` | ❌ |
| FLT-R127 | El serial de un GPS migrado sin él se completa una sola vez, con `habilitacion.gestionar_equipos` donde está la unidad y su fila en la bitácora; uno que ya tiene serial no se cambia así | D-206, §11.4 | Dominio: `VehicleEquipment.completeSerial`. Servicio: `422 VEHICLEEQUIPMENT_SERIAL_ALREADY_SET`. Recurso: `completeSerial` | `VehicleEquipmentTests.aSerialIsCompletedOnlyOnce`; `VehicleEquipmentIntegrationTests.aMigratedGpsGetsItsSerialAndLeavesItsRow` | ❌ |

### 6.3 `procurement`

| ID | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| PRC-R140 | El archivo de la carga inicial trae los 52 encabezados de la guía, en orden; las columnas que no se importan solo se leen para devolverlas en el informe | D-201; guía §5 | Servicio: `StockRowMapper` con `ImportSheetSpec` | `StockRowMapperTests.eachOfTheFiftyTwoColumnsGoesWhereSection54Says`, `theComputedColumnsAreNeverRead` | ❌ |
| PRC-R141 | Las filas se agrupan por proveedor y número de factura; cada factura entra en su transacción con sus líneas, sus unidades y su habilitación, o falla entera y todas sus filas van al informe | D-201, D-168 | Servicio: `StockRowGroups`, `MigratedInvoiceWriter` | `InitialMigrationIntegrationTests.aFailingRowLeavesNothingOfItsInvoiceAndTheOtherInvoicesEnter` | ❌ |
| PRC-R142 | Idempotente por VIN: una factura ya migrada con las mismas unidades queda sin cambios; un VIN registrado fuera de la migración, de otra empresa o repetido en el archivo hace fallar su factura; una factura ya registrada con otras unidades falla | D-201, D-153, D-160 | Servicio: con `matchMigratedUnits` | `InitialMigrationIntegrationTests.uploadingTheSameFileTwiceChangesNothing`, `aVinRegisteredByAnInvoiceFailsItsGroup`, `aRepeatedVinFailsBothGroups` | ❌ |
| PRC-R143 | La tasa es la de la fila, una por factura, y su día es la fecha de la factura; en la moneda funcional, 1. No se aplican los 20 días ni se pide `compras.registrar_factura` | D-177, D-169, D-202 | Servicio: `MigratedInvoiceWriter` | `InitialMigrationIntegrationTests.theRateOfTheRowIsKept`, `aSolesInvoiceWithAnotherRateFails`, `anInvoiceOfLastYearEnters` | ❌ |
| PRC-R144 | El proveedor se encuentra por razón social o nombre comercial, uno solo y activo; el modelo, dentro de su marca; el color, por nombre. Lo que no está o está inactivo hace fallar la factura | D-177, D-108; guía, columnas 7 a 11 | Servicio: `InitialMigrationCatalogs` | `InitialMigrationCatalogsTests.aSupplierMatchingTwoIsAmbiguous`; `InitialMigrationIntegrationTests.anUnknownModelFailsItsInvoice` | ❌ |
| PRC-R145 | Valen las reglas de la factura de la fase 2: número normalizado con el formato de Perú, único entre las no anuladas; fecha no futura; de 1 a 100 líneas; montos mayores que cero con su escala; total igual a la suma; tipo factura, sin ISC ni orden | D-160, D-179 | Dominio: `PurchaseInvoice`, `InvoiceNumber`. Base: los únicos de la fase 2 | `InitialMigrationIntegrationTests.aWrongNumberFormatFailsItsInvoice`, `anInvoiceOfMoreThanAHundredRowsFails` | ❌ |
| PRC-R146 | Las filas de una factura coinciden en fecha, moneda y TC; si no, falla | Guía, columna 47; D-177 | Dominio: `StockRowGroups` | `StockRowGroupsTests.rowsOfOneInvoiceWithTwoRatesFail` | ❌ |
| PRC-R147 | Una nota de débito va al informe y nunca se registra; la unidad entra sin ella | D-23, D-177 | Servicio: `MigratedCreditNoteWriter` | `InitialMigrationIntegrationTests.aDebitNoteIsReportedAndItsUnitEnters` | ❌ |
| PRC-R148 | Las notas de crédito se agrupan por proveedor y código, después de las facturas, cada una en su transacción; cada línea copia la tasa de su factura; misma moneda; fecha no futura ni anterior a sus facturas; hasta 100 líneas; el motivo, `DESCUENTO_CAMPANA`; una nota que dejaría una línea en cero falla; una ya registrada con las mismas líneas queda sin cambios | D-177, D-190 a D-193, D-204, D-205 | Servicio: `MigratedCreditNoteWriter` con el agregado de la fase 4 | `InitialMigrationIntegrationTests.aNoteAcrossTwoInvoicesCopiesEachRate`, `aNoteThatZeroesALineIsLeftForTheHand`, `aNoteWhoseInvoiceFailedIsReported` | ❌ |
| PRC-R149 | La carga inicial exige `stock.cargar_migracion_inicial` en toda la empresa y en la marca elegida, y `habilitacion.ver_cargas`; la marca y la ubicación son de la empresa y activas | D-202, D-101, D-102, D-84 | Servicio: `InitialMigrationImportCommandServiceImpl` | `InitialMigrationIntegrationTests.aBrandAssignmentLoadsOnlyItsBrand`, `aLocationOfAnotherCompanyIsRefused` | ❌ |
| PRC-R150 | Los conteos: entró lo que escribió algo ahora, sin cambios lo que ya estaba, fallido lo que está en el informe. El lote se registra al final, también si todo falló | §11.8, D-201 | Servicio | `InitialMigrationIntegrationTests.theCountsFollowSection24`, `aLoadWhereEverythingFailsStillHasItsReport` | ❌ |
| PRC-R151 | La carga es síncrona, y un archivo de la carga inicial trae a lo sumo el tope de filas que fija su línea base; uno más grande se parte | D-208, D-201; D-147 (Cloudflare) | Servicio: `IMPORTBATCH_TOO_MANY_ROWS` | `InitialMigrationIntegrationTests.moreRowsThanTheCapAreRefused` | ❌ |
| PRC-R152 | Cada factura, cada nota y cada pago corren en su transacción, y un interbloqueo se repite una vez desde el orquestador, que no es transaccional | D-128 | Servicio: `TransactionTemplate`, `DeadlockRetry.isDeadlock` | `InitialMigrationDeadlockTests.aDeadlockedInvoiceIsRetriedOnce` | ❌ |
| PRC-R153 | Cada bloque de pago —proveedor y operación— es un pago de la fase 5, después de las notas, en su transacción: sus filas coinciden en fecha, monto y moneda; se aplica a sus facturas en proporción al valor de sus unidades, nunca más de lo que se debe; su `requestId` sale de la empresa, el proveedor y la operación, así que volver a subirlo no lo repite. Va por transferencia, con su tasa (1, la del catálogo o la del Excel) y, en el canal de dealers, una imputación saldada por unidad | D-190, D-194 a D-199, D-207; guía, columnas 42 a 45 y §7.3; P11 de la fase 5 | Servicio: `MigratedPaymentWriter` con el agregado de la fase 5 | `InitialMigrationIntegrationTests.aSegmentedBlockIsOnePayment`, `aBlockAcrossTwoInvoicesIsSplitByValue`, `aBlockWithTwoAmountsFails`, `aBlockUploadedTwiceIsOnePayment` | ❌ |
| PRC-R154 | Lo que el Excel no trae se deduce de la misma fila: el ingreso, de la fecha más temprana entre el GPS, la pantalla, la primera entrega y la declaración de stock, o de la factura; las placas físicas, de la primera entrega o la declaración de stock, o del título; la etiqueta de la llave, del código de la marca comercial y el `N°`. El propietario no se lee | D-206; guía, columnas 13, 17, 2, 12 y 50 | Aplicación: `StockRowMapper`, sin base de datos | `StockRowMapperTests.theArrivalIsTheEarliestDateOfTheRowOrTheInvoice`, `thePlatesAreTheDeliveryOrTheStockDeclarationOrTheTitle`, `theKeyTagIsTheBrandCodeAndTheNumber` | ❌ |

---

## 7. Pantallas

### 7.1 Rutas y menú

| Ruta del navegador | Vista | `data.permissions` | Encabezado |
|---|---|---|---|
| `/imports` | `ImportsView` | `anyOf: [habilitacion.ver_cargas]` | El del módulo, `modules.imports` (ya existe en `public/i18n/es.json:73`) |
| `/imports/{importId}` | `ImportBatchView` | `anyOf: [habilitacion.ver_cargas]` | El tipo y el archivo, con `overridePageHeading` |

- **Menú:** en `app-navigation.ts`, la opción `/imports` (línea 179) pierde `pending: true` y declara `access: anyOf(PermissionCodes.Enablement.VIEW_IMPORTS)`. Se queda en la sección de flota, debajo de Habilitación.
- **Rutas:** `app.routes.ts` carga `imports/presentation/imports.routes` bajo demanda, como `change-log`.
- **Permisos:** `PermissionCodes` del frontend suma `Enablement.LOAD_SAT_FILE`, `Enablement.VIEW_IMPORTS` y `Funds.LOAD_GUARANTEE_FILE`. `Stock.LOAD_INITIAL_MIGRATION` ya existe (`permission-codes.ts:97`).

**La carpeta.** `src/app/imports` es de la plataforma, como `src/app/change-log` (D-200, D-129): espejo de `shared/importbatches`, con sus cuatro capas. Los diálogos de carga son de sus dueños: los dos de `fleet` y el de `procurement`. `imports` los abre por sus fachadas (frontend-rules 1.3, «Puede abrir un flujo de su propio contexto»).

### 7.2 Logística › Cargas de archivos

- **Acciones** (`.mm-page-actions`): «Cargar archivo», un botón con menú. Ofrece solo las cargas que la sesión puede hacer en la empresa elegida, con `hasPermissionIn`:
  - «Impuesto vehicular (SAT)»;
  - «Garantía mobiliaria»;
  - «Migración inicial».

  Sin ninguna, el botón no aparece.
- **Filtros:** empresa (`mm-company-filter`), tipo (Todas y los tres), período Desde–Hasta (Desde solo es ese día; Desde y Hasta, el intervalo completo, con los dos días; Hasta solo, todo hasta ese día), y la búsqueda por nombre del archivo.
- **Columnas** (`mm-data-manager`, paginada en el servidor):

  | Columna | Contenido | Ordena |
  |---|---|---|
  | Fecha | `dd/mm/aaaa hh:mm` en la zona de la empresa | Sí |
  | Tipo | Insignia: Impuesto vehicular, Garantía mobiliaria, Migración inicial | Sí |
  | Archivo | Su nombre; abre la ficha | Sí |
  | Filas | El total | No |
  | Entraron · Sin cambios · Fallaron | Tres números; Fallaron en el tono de error si es mayor que cero | No |
  | Quién | El nombre | No |

- **Menú de la fila:** «Ver carga» y «Descargar errores», solo si hay informe.
- **Estados:**
  - esqueleto (`mm-skeleton`) mientras carga;
  - vacío: «Todavía no hay cargas en esta empresa», con «Cargar archivo» si se puede;
  - error, con «Reintentar».

### 7.3 La ficha de una carga

- **Datos:** tipo, archivo, quién, cuándo y empresa.
- **Conteos:** cuatro tarjetas: Filas, Entraron, Sin cambios y Fallaron.
- **Acciones:**
  - «Descargar archivo de errores», con «Descargar informe» como texto en la garantía, que siempre lo tiene;
  - «Descargar archivo cargado».
- **La garantía** dice además que el informe trae el fondo, la fecha de la lista y las cesiones cerradas.

### 7.4 Los diálogos de carga

Los tres comparten la forma:

| Parte | Comportamiento |
|---|---|
| **Empresa** | La del filtro de la lista, de solo lectura |
| **Archivo** | `mm-image-slot` con `accept` de .xlsx y `maxBytes` de 10 MB (revisión 38). Un archivo de otro tipo o más pesado se rechaza antes de enviarlo |
| **Cargar** | Deshabilitado hasta completar el formulario. Al enviar, el diálogo no se cierra con Escape ni con el fondo, y muestra una barra indeterminada con «Procesando el archivo. No cierre esta ventana» |
| **Resultado** | Las cuatro cifras, los efectos de la carga y «Ver la carga», que cierra el diálogo y abre la ficha, donde se descarga el informe. La lista se recarga |
| **Errores del pedido** | En un aviso dentro del diálogo, con el texto de su código. `IMPORTBATCH_HEADERS_MISMATCH` lista los encabezados que faltan y los que sobran |

**Impuesto vehicular** (`fleet/presentation/components/tax-declaration-import-dialog`). Título «Cargar el archivo del impuesto vehicular». Texto: «El archivo que exporta la plataforma del SAT. Solo actualiza autos que ya existen: cada fila se cruza por su placa».

**Garantía mobiliaria** (`fleet/presentation/components/guarantee-import-dialog`).
- **Campos:**
  - Fondo: los activos de la empresa, por el store de fondos de `fleet`;
  - Fecha de la lista: hoy en la zona de la empresa por omisión, sin fechas futuras.
- **«Revisar y cargar»** envía sin confirmar. Si la respuesta es `VEHICLEFUNDASSIGNMENT_IMPORT_UNCONFIRMED`, abre `ConfirmationDialog`: «La lista abrirá 12 cesiones y cerrará 3 el {fecha}. Las cesiones cerradas conservan su historia. 2 filas fallarán y 1 cesión de una unidad sin placa no se cierra». Con «Cargar y cerrar», reenvía el mismo archivo con `confirmClosures: true`.

**Migración inicial** (`procurement/presentation/components/initial-migration-import-dialog`).
- **Campos:**
  - Marca comercial: las activas de la empresa, por `OrganizationFacade.loadCommercialBrands`;
  - Ubicación de las unidades que ya ingresaron: las activas de la empresa; opcional.
- **Avisos:**
  - «Cada factura entra completa o no entra. Volver a subir el archivo corregido no repite lo que ya entró»;
  - «Use la plantilla de 52 columnas», con un enlace que la descarga desde los recursos estáticos del frontend (`public/templates/stock-millennium-2026.xlsx`, una copia de la plantilla del cliente, solo con sus encabezados).

### 7.5 La unidad: impuesto vehicular y origen de la cesión

- **La página de habilitación de la unidad** (fase 3, revisión 35) suma un panel «Impuesto vehicular»:
  - la insignia del estado SAT: No corresponde (neutro), Sin TIVE (advertencia), Pendiente (información) o el estado del archivo (éxito);
  - la fecha de inscripción;
  - un historial plegable con cada declaración y su carga.
- **La pestaña de cesiones de la ficha de la unidad** dice «Carga del dd/mm/aaaa» en una cesión que abrió una carga, enlazada a su ficha si la sesión tiene `habilitacion.ver_cargas`.
- **Lo migrado sin archivo ni serial** (D-206): un documento migrado dice «Migrado, sin archivo» y ofrece «Subir archivo», que lo reemplaza (D-28); un GPS migrado dice «Migrado, sin serial» y ofrece «Completar serial», un diálogo de un campo que llama a `PUT /vehicle-equipments/{id}/serial` y que solo aparece con `completeSerial`.

### 7.6 Frontend por capas

| Pieza | Contenido |
|---|---|
| `imports/domain/model` | `import-batch.entity.ts` (`ImportBatch`, con `rowsUnchanged`), `import-type.ts` |
| `imports/infrastructure` | `imports-api-endpoint.ts`, `imports-response.ts`, `imports-api.ts` y `import-batch-assembler.ts` |
| `imports/application` | `imports.store.ts`: `batchPage`, `batch`, `loading`, `downloading`, `error`, y `loadPage()`, `loadBatch()`, `downloadReport()` y `downloadSource()` |
| `imports/presentation` | `imports.routes.ts`; vistas `imports-view` e `import-batch-view`; `import-type-badge` |
| `fleet` | `fleet-imports.store.ts` (`uploadTaxDeclarations`, `uploadGuaranteeList`), sus dos diálogos, el panel del impuesto, el origen de la cesión y, en la habilitación de la unidad, lo migrado sin archivo ni serial con el diálogo `complete-equipment-serial-dialog` (D-206). `FleetFacade` suma `openTaxDeclarationImport(companyId)` y `openGuaranteeImport(companyId)`, que abren sus diálogos con `import()` y resuelven `Promise<string \| null>`: el id de la carga |
| `procurement` | `initial-migration.store.ts` y su diálogo. **`ProcurementFacade`** es nueva (`procurement/application/procurement.facade.ts`), con un solo miembro, `openInitialMigrationImport(companyId)`, que tiene su consumidor |
| `shared/presentation` | `import-result-summary`: las cuatro cifras de un resultado, sin saber de qué carga son (frontend-rules 1.4) |

Los mensajes de cada código, en `es.json`, bajo `imports.*`, `fleet.tax-declaration-import.*`, `fleet.guarantee-import.*` y `procurement.initial-migration.*`.

### 7.7 A 375 px y accesibilidad

- **La lista** es la de `mm-data-manager`, que no tiene modo de tarjetas: se desplaza de lado (revisiones 35 y 38 de `DESIGN.md`).
- **Los diálogos** ocupan la pantalla; el área del archivo se toca entera.
- **El progreso y el resultado** se anuncian (`aria-live="polite"`). Un error del pedido se anuncia con `assertive`.
- **axe** sin violaciones en la lista, la ficha y los tres diálogos.

### 7.8 Revisión 38 de `DESIGN.md`

La revisión 38 de `DESIGN.md`, escrita el 03/11/2026, decide lo visual y manda donde difiera de esta lista. Por ejemplo, usa un solo tono para el tipo de carga. Lo que debía decidir:

| # | Qué decide | Por qué |
|---|---|---|
| 1 | `mm-image-slot` acepta un libro .xlsx de hasta 10 MB: sin vista previa, con su nombre y su peso | Hoy acepta imágenes y PDF de hasta 5 MB |
| 2 | El diálogo de carga ocupado: barra indeterminada, sin cierre por Escape ni por el fondo, y el texto que pide no cerrar | Patrón nuevo: una operación de hasta un minuto |
| 3 | El resultado de una carga dentro del diálogo y en la ficha: cuatro cifras, Fallaron en el tono de error | Pieza nueva de `shared/presentation` |
| 4 | El botón «Cargar archivo» con menú de tipos | Una acción principal con varias variantes |
| 5 | La insignia del tipo de carga y sus tres tonos | Tono nuevo de `mm-status-badge` |
| 6 | La confirmación de los cierres de la garantía, con sus cifras | `ConfirmationDialog` con una lista de efectos |
| 7 | El panel del impuesto vehicular y su insignia de cuatro estados en la habilitación de la unidad | Cambia la revisión 35 |
| 8 | El origen de una cesión en la ficha de la unidad | Cambia la pestaña de cesiones |
| 9 | El documento migrado sin archivo, el GPS sin serial y el diálogo «Completar serial» (D-206) | Estado y acción nuevos de la revisión 35 |

---

## 8. Pruebas

### 8.1 Los criterios de aceptación

| Criterio | Prueba |
|---|---|
| Ningún archivo auxiliar crea una unidad | `TaxDeclarationImportIntegrationTests.aPlateWithoutUnitGoesToTheReportAndNothingIsCreated`; ídem en la garantía |
| Una placa que desaparece de la lista cierra su cesión y la historia queda | `GuaranteeImportIntegrationTests.aPlateThatLeavesTheListClosesItsAssignmentAndKeepsIt` |
| Las filas de una factura entran o fallan juntas | `InitialMigrationIntegrationTests.aFailingRowLeavesNothingOfItsInvoiceAndTheOtherInvoicesEnter` |
| Volver a subir el archivo corregido es seguro | `InitialMigrationIntegrationTests.uploadingTheSameFileTwiceChangesNothing`, `theCorrectedReportEntersOnlyWhatWasMissing` |
| Cada unidad migrada queda en su estado | `MigratedUnitRegistrationsIntegrationTests.theReadEnablementMatchesTheDecidedStatus` |
| El informe se corrige y se vuelve a subir | `ImportReportWriterTests.aReportReadsBackThroughTheReader` |
| La frontera entre empresas | `ImportBatchRowLevelSecurityTests`, `FleetRowLevelSecurityTests` (declaraciones) |
| Arquitectura | `ModularityTests` (sin ciclo entre `shared` y `documents`), `ForeignAclStaysInTranslationLayerTests`, `NoDeleteEndpointsTests`, `PermissionCatalogConsistencyTests` |

### 8.2 Backend, por pieza

| Pieza | Pruebas sin contexto de Spring | Pruebas de integración (PostgreSQL de verificación) |
|---|---|---|
| Lector y escritor | `ImportWorkbookReaderTests`, `ImportCellTests`, `ImportReportWriterTests`, `XlsxContainerCheckTests` | — |
| `ImportBatch` | `ImportBatchTests` | `ImportBatchRegistrationIntegrationTests`, `ImportBatchQueryIntegrationTests`, `ImportBatchFilesIntegrationTests`, `ImportBatchConcurrencyTests`, `ImportBatchRowLevelSecurityTests` |
| SAT | `TaxDeclarationRowPolicyTests`, `VehicleTaxStatusPolicyTests`, `VehicleTaxDeclarationTests` | `TaxDeclarationImportIntegrationTests`, `VehicleTaxDeclarationQueryIntegrationTests` |
| Garantía | `GuaranteeListPolicyTests` | `GuaranteeImportIntegrationTests`, `GuaranteeImportConcurrencyTests`, `VehicleFundAssignmentQueryIntegrationTests` |
| Fachada de `fleet` | `MigrationTargetPolicyTests` | `FleetContextFacadeTests`, `MigratedUnitRegistrationsIntegrationTests`, `StatusChangeReasonIntegrationTests` |
| Carga inicial | `StockRowMapperTests`, `StockRowGroupsTests`, `InitialMigrationCatalogsTests` | `InitialMigrationIntegrationTests`, `InitialMigrationDeadlockTests` |
| Esquema | — | `FileLoadsMigrationTests`, `FileLoadPermissionsMigrationTests`, `ImportTablesAreInsertOnlyTests`, `HandOverDatabaseScriptTests` |

Las pruebas de integración corren sobre `erp_mm_db_verify` y se conectan con la cuenta de la aplicación, que no se salta Row Level Security.

### 8.3 Archivos de prueba

**Ninguno lleva datos reales.** Casi todos los arma en memoria un ayudante de prueba, `XlsxFixtures` (`src/test/java/.../shared/importbatches`), con el escritor de fastexcel: encabezados, filas y tipos de celda. Así un caso se lee junto a su prueba. Los que no se pueden armar así se guardan en `src/test/resources/imports/`.

| Archivo o constructor | Para qué |
|---|---|
| `XlsxFixtures.stock(rows…)` | Un libro con los 52 encabezados de la plantilla y las filas dadas |
| `XlsxFixtures.sat(rows…)`, `XlsxFixtures.guarantee(rows…)` | Los dos archivos externos, con sus encabezados provisionales y una columna ajena que se ignora |
| `XlsxFixtures.withHeaders(…)` | Encabezados con `Nº`, dobles espacios, espacio duro, una columna de más o fuera de orden |
| `XlsxFixtures.withDates1904(…)` | El sistema de fechas de 1904 |
| `stock-template-headers.xlsx` | Una copia de la plantilla del cliente, solo con sus encabezados: prueba que se lee tal cual llega |
| `macro-enabled.xlsm` | Un libro con `vbaProject.bin` |
| `pdf-renamed.xlsx` | Un PDF con extensión .xlsx |
| `inflation-bomb.xlsx` | Una entrada de 1 MB que se infla a más de 100 MB |
| `doctype.xlsx` | Un `sharedStrings.xml` con `DOCTYPE` y una entidad externa |
| `corrupt.xlsx` | Un ZIP truncado |
| `two-sheets-second-matches.xlsx` | La hoja con los encabezados no es la primera |
| `report-roundtrip.xlsx` | Generado por la prueba del escritor y vuelto a leer por el lector |

**Los escenarios de la carga inicial**, armados con `XlsxFixtures.stock`:
- una factura de un auto, entregado;
- una factura de 80 motos, unas ingresadas y otras no;
- una factura en dólares y otra en soles;
- una nota que cruza dos facturas;
- una nota de débito;
- una nota que deja una unidad en cero;
- un VIN repetido entre dos facturas;
- un proveedor y un modelo desconocidos;
- una factura de 101 filas;
- un pago segmentado (`PAGO1/PAGO2`) que cubre dos facturas, y un bloque con dos montos distintos;
- un archivo subido dos veces, y su informe corregido.

**Para el frontend:**
- `e2e/fixtures/imports/` guarda tres libros chicos generados por `XlsxFixtures`, para las pruebas con el backend simulado, donde el contenido no importa;
- `e2e-live/fixtures/imports/` guarda los que la sonda sube al backend real, también generados y sin datos reales.

### 8.4 Rendimiento

Primero se mide la línea base, como pide el plan para las facturas (plan, hallazgo B1). Objetivos provisionales sobre la base de verificación:

| Carga | Volumen | Objetivo |
|---|---|---|
| SAT | 5 000 filas | Menos de 10 s |
| Garantía | 5 000 placas, 100 cierres | Menos de 10 s |
| Carga inicial | 1 000 filas de autos (1 000 facturas) | Menos de 60 s |
| Carga inicial | 1 000 filas de motos (13 facturas de 80) | Menos de 30 s |

El número de sentencias por factura se fija con las estadísticas de Hibernate, como el registro de 100 líneas de la fase 2. **El tope de PRC-R151** sale de esta medición: el mayor número de filas que termina en menos de 60 s, para quedar lejos del corte de Cloudflare.

### 8.5 Frontend y extremo a extremo

- **Unitarias:**
  - `imports.store.spec`, `import-batch-assembler.spec`;
  - cada diálogo: «does not close while it uploads», «shows the four counts and opens the load», «lists the missing headers»;
  - el de la garantía: «asks before closing and resends with the confirmation»;
  - `procurement.facade.spec` y `fleet.facade.spec`: «a load opened from imports leaves no error on the owner's screens».
- **Extremo a extremo** (`e2e/imports.spec.ts`, con los mocks divididos por contexto de la fase 2):
  - la lista con sus filtros;
  - la ficha y sus dos descargas;
  - las tres cargas;
  - la confirmación de la garantía;
  - el menú, que ofrece solo lo que la sesión puede;
  - axe y tres anchos: 375, 768 y 1280 px.

### 8.6 Sondas en vivo

`e2e-live/imports.live.spec.ts`, contra `erp_mm_db_live` en el puerto 8090:
1. carga una lista de garantía de un fondo de prueba y confirma un cierre;
2. carga un archivo del SAT con una placa conocida y otra desconocida, y descarga el informe;
3. carga un archivo de la carga inicial con dos facturas, una con un modelo desconocido, y lo vuelve a subir;
4. comprueba que ninguna petición usó `DELETE`.

### 8.7 Ensayo con los archivos reales

Antes de producción, sobre `erp_mm_db_verify` y con los catálogos conciliados:
1. carga los dos Excel de stock;
2. después, el SAT;
3. después, la garantía.

El resultado —conteos, códigos de error agrupados, unidades por estado comparadas con la columna 5 y el tiempo de cada carga— se escribe en `retroalimentacion/<fecha>-ensayo-de-las-cargas.md`, sin datos de unidades concretas. Producción espera a que el responsable lo apruebe.

---

## 9. Secuencia de commits

El paquete se entrega en tres partes, cada una con su puerta (D-208). Cada entrega tiene su rama en los dos repositorios, desde `develop`, en worktrees hermanos: `feature/file-loads`, `feature/initial-load` y `feature/initial-load-payments`. Cada commit compila y pasa las pruebas que alcanza, siempre con `ModularityTests`; con tablas nuevas, también las de RLS y de filas que no se borran. Una entrega se integra en `develop` con sus suites completas, y la siguiente empieza desde ahí.

### 9.1 Primera entrega: registro, impuesto vehicular y garantía mobiliaria

**Puerta:**
- la fase 3 integrada en `develop`: la TIVE vigente y la placa como hito (§1.4);
- el archivo del SAT y la lista de garantía reales perfilados (§1.3, puntos 19 a 21), que fijan sus encabezados;
- la revisión 38 de `DESIGN.md` escrita, antes del frontend.

**Documentación**

| # | Commit | Qué |
|---|---|---|
| D1 | `docs(feedback): profile the real tax and guarantee files` | §1.3, puntos 19 a 21 |
| D2 | `docs(conventions): add the contract of the file loads registry and the fleet loads` | §4 para `/imports`, el SAT y la garantía, a las convenciones §2.2, §7.2 y §8, y §14.18 cerrada por D-200 |
| D3 | `docs(contexts): give shared its rule prefix` | `SHR-` en `contextos/README.md` |

**Backend**

| # | Commit | Depende de |
|---|---|---|
| B1 | `build: read workbooks in the application with fastexcel-reader` | — |
| B2 | `feat(iam): seed the permissions of the file loads` | — |
| B3 | `feat(shared): create the schema of the import batches` | — |
| B4 | `feat(shared): read an uploaded workbook by its exact headers` | B1 |
| B5 | `feat(shared): write the report of a load with its failed rows` | B4 |
| B6 | `feat(documents): keep the files of the import batches` | B3 |
| B7 | `feat(shared): register an import batch with its files and counts` | B3, B5, B6 |
| B8 | `feat(shared): list and read the import batches and download their files` | B7 |
| B9 | `feat(fleet): create the tax declarations and key the assignments to their load` | B3 |
| B10 | `feat(fleet): load the tax declarations file by plate` | B7, B9 |
| B11 | `feat(fleet): answer the tax status of a unit` | B10 |
| B12 | `feat(fleet): load the guarantee list of a fund and confirm its closures` | B7, B9 |
| B13 | `feat(fleet): say which load opened an assignment` | B8, B12 |
| B14 | `test(fleet): measure the tax and guarantee loads and fix their row caps` | B10, B12 |

**Frontend**

| # | Commit | Depende de |
|---|---|---|
| F1 | `feat(imports): model the import batches with their API and store` | D2 |
| F2 | `feat(imports): list the import batches by company, type and period` | F1, B8 |
| F3 | `feat(imports): show an import batch and download its files` | F2 |
| F4 | `feat(shared): show the result of a load and accept a workbook in the file slot` | revisión 38 |
| F5 | `feat(fleet): upload the tax declarations file from its dialog` | F4, B10 |
| F6 | `feat(fleet): upload the guarantee list and confirm its closures` | F4, B12 |
| F7 | `feat(fleet): show the tax status of a unit and the load of an assignment` | B11, B13 |
| F8 | `feat(app-shell): open Logística › Cargas de archivos` | F2 |
| F9 | `test(e2e): cover the registry, tax and guarantee loads on three breakpoints with axe` | F8 |
| F10 | `test(live): probe the tax and guarantee loads against the real backend` | todo lo anterior |

**Cierre**

| # | Commit | Qué |
|---|---|---|
| D4 | `docs(contexts): close the registry, tax and guarantee loads with their rules and commits` | `contextos/fleet.md` (FLT-R100 a FLT-R118) en ✅; la sección de `shared` en el README de los contextos (SHR-R01 a SHR-R13) |
| D5 | `docs(processes): mark the registry, tax and guarantee loads as built` | Procesos P9 y §3.2, el plan y el mapa (§6.5, y la decisión 3 en lo que toca a `/imports`). Al responsable, que cambia el diccionario, se le proponen dos textos: §2.7 nombra los archivos de las cargas en `kept_files`, y §11.8, el informe con sus hojas |

**Migraciones:** B2, B3 y B9 llevan V(n), V(n+1) y V(n+2).

### 9.2 Segunda entrega: la carga inicial

**Puerta:**
- la primera entrega integrada en `develop`;
- la fase 4 integrada: la carga inicial registra notas de crédito y facturas reales (D-176);
- los Excel de stock reales perfilados (§1.3, puntos 1 a 18), y los catálogos conciliados con lo que encontró el perfil: los proveedores, los modelos y los colores que faltaban (plan §3.2).

**Documentación**

| # | Commit | Qué |
|---|---|---|
| D6 | `docs(feedback): profile the real stock workbooks` | §1.3, puntos 1 a 18 |
| D7 | `docs(conventions): add the contract of the initial load and the serial of a migrated GPS` | `POST /initial-migration-imports`, `PUT /vehicle-equipments/{id}/serial` y sus códigos |

**Backend**

| # | Commit | Depende de |
|---|---|---|
| B15 | `feat(fleet): seed the initial migration paths and reserve their code` | — |
| B16 | `feat(fleet): tell migrated from registered VINs through the facade` | fase 2 |
| B17 | `feat(fleet): register the migrated units of an invoice in their status` | B15, B16 |
| B18 | `feat(fleet): keep a migrated document without its file and a migrated GPS without its serial` | B17 |
| B19 | `feat(fleet): complete once the serial of a migrated GPS` | B18 |
| B20 | `feat(procurement): map the fifty-two columns of the stock workbook and deduce what it lacks` | B4 |
| B21 | `feat(procurement): group the stock rows by invoice and resolve their catalogs` | B20 |
| B22 | `refactor(shared): let the orchestrators tell a deadlock` — `DeadlockRetry.isDeadlock` pasa a público | — |
| B23 | `feat(procurement): migrate each invoice with its units in its own transaction` | B17, B21, B22 |
| B24 | `feat(procurement): migrate the credit notes grouped by code and supplier` | B23 |
| B25 | `feat(procurement): register the initial migration with its report` | B7, B24 |
| B26 | `test(procurement): measure the initial migration and fix its row cap` | B25 |

**Frontend**

| # | Commit | Depende de |
|---|---|---|
| F11 | `feat(fleet): show what a migrated unit lacks and complete the serial of its GPS` | B19 |
| F12 | `feat(procurement): upload the initial migration from its dialog` | F4, B25 |
| F13 | `test(e2e): cover the initial load and the migrated unit on three breakpoints with axe` | F11, F12 |
| F14 | `test(live): probe the initial load against the real backend` | todo lo anterior |

**Cierre**

| # | Commit | Qué |
|---|---|---|
| D8 | `docs(contexts): close the initial load with its rules and commits` | `contextos/fleet.md` (FLT-R120 a FLT-R127) y `contextos/procurement.md` (PRC-R140 a PRC-R152 y PRC-R154) en ✅ |
| D9 | `docs(processes): mark the initial load as built` | Procesos P9, el plan. Al responsable se le proponen tres textos del diccionario: §8.3 nombra el código reservado `MIGRACION_INICIAL`; §11.3 marca `file_path`, `file_name` y `mime_type` como vacíos solo en un documento migrado; §11.8 nombra la columna 51 como el momento de la entrada a Disponible |

**Migraciones:** B15 y B18 llevan V(n+3) y V(n+4).

### 9.3 Tercera entrega: los pagos de la carga inicial

**Puerta:**
- la segunda entrega integrada en `develop`;
- la fase 5 integrada: `SupplierPayment`, sus aplicaciones y sus imputaciones;
- el perfil de los bloques de pago (§1.3, punto 13) revisado contra los topes de la fase 5.

**Documentación**

| # | Commit | Qué |
|---|---|---|
| D10 | `docs(conventions): add the payments to the result of the initial load` | `effects.payments` y los códigos de fila de los pagos |

**Backend**

| # | Commit | Depende de |
|---|---|---|
| B27 | `feat(procurement): migrate the payment blocks of the stock workbook as payments` | B25, fase 5 |
| B28 | `test(procurement): measure the initial migration with its payments` | B27 |

**Frontend**

| # | Commit | Depende de |
|---|---|---|
| F15 | `feat(procurement): show the migrated payments in the result of the initial load` | B27 |
| F16 | `test(live): probe the payments of the initial load against the real backend` | F15 |

**Cierre**

| # | Commit | Qué |
|---|---|---|
| D11 | `docs(contexts): close the payments of the initial load` | `contextos/procurement.md` (PRC-R153) en ✅; procesos P9 y el plan |
| D12 | `docs(feedback): record the rehearsal with the real files` | §8.7: es la puerta de producción |

Esta entrega no trae migraciones.

### 9.4 La corrida en producción

No es un commit. **Puerta:** las tres entregas desplegadas, el ensayo de §8.7 aprobado por el responsable y la base respaldada (plan §7).

**La carga inicial corre una sola vez** (D-208):
1. los dos Excel de stock, partidos si pasan el tope de filas;
2. el informe de cada uno, corregido y vuelto a subir hasta que solo queden las filas que se resuelven a mano;
3. después, el archivo del SAT;
4. después, la lista de garantía de cada fondo.

---

## 10. Preguntas de la versión 1, decididas

No queda ninguna. Las nueve preguntas de la versión 1 (03/11/2026) quedaron decididas el mismo día, por delegación del responsable, con la recomendación de esta especificación, y se registran en el diccionario como D-206 a D-209. Esta versión las aplica, cada una donde corresponde.

| # | Pregunta de la versión 1 | Decisión | Dónde se aplica |
|---|---|---|---|
| P1 | ¿Puede la carga inicial guardar la TIVE y el SOAT sin archivo, y el GPS sin serial? | **D-206.** Sí, solo en la carga inicial. El documento migrado sin archivo se rotula «Migrado, sin archivo» y su archivo se sube después como reemplazo (D-28); el serial del GPS se completa una sola vez. V(n+4) deja de ser condicional | §2.7, §2.7.2, §3.6, FLT-R124, FLT-R127 |
| P2 | ¿Qué fechas toman el ingreso, las placas físicas y la entrada a Disponible, que el Excel no fecha? | **D-206.** Se deducen de la misma fila: el ingreso, de la más temprana entre el GPS, la pantalla, la primera entrega y la declaración de stock, o de la factura; las placas físicas, de la primera entrega o la declaración de stock, o del título. La ubicación la elige quien carga, y el perfil cuenta cada dato deducido. La entrada a Disponible toma la columna 51, como recomendaba la pregunta; el texto de D-206 no la nombra, y D9 propone sumarla a §11.8 | §2.7, §2.7.1, PRC-R154 |
| P3 | ¿Se migran las llaves, si el Excel no trae la etiqueta que exige §11.5? | **D-206.** Sí, con una etiqueta provisional: el código de la marca comercial y el `N°` (`LIZZO-1050`) | §2.7, PRC-R154 |
| P4 | ¿Qué motivo llevan las notas de crédito migradas? | **D-207.** `DESCUENTO_CAMPANA`, sin mes de campaña; las que dejarían una línea en cero se registran a mano con su motivo | §2.8 paso 7, PRC-R148 |
| P5 | ¿Qué método, tasa y detalle lleva un pago migrado, que el Excel no trae? | **D-207.** Uno por bloque de la columna 42, por transferencia y con ese código como número de operación; tasa 1 en la moneda funcional y, en otra, la de venta del catálogo para el día del pago o, si falta, la del Excel; en el canal de dealers, cada unidad imputada por su valor menos la columna 45 y saldada. Basta el permiso de la carga | §2.8 paso 8, PRC-R153 |
| P6 | ¿La carga inicial se construye después de la fase 4, y no después de la 3? | **D-208.** Tres entregas: el registro, el SAT y la garantía después de la fase 3; la carga inicial después de la fase 4; sus pagos después de la fase 5. En producción, la carga inicial corre una sola vez, después de la fase 5 | §1.4, §9 |
| P7 | ¿La lista de garantía cierra también las cesiones registradas a mano? | **D-209.** Sí, si la unidad tiene placa y no figura; la de una unidad sin placa no se cierra y se informa; una placa de otro fondo falla sin tocar nada; el fondo y la fecha de la lista quedan en el informe | §2.6, FLT-R112, FLT-R113 |
| P8 | ¿Qué se hace con una unidad cuyo propietario no es la empresa (columna 50)? | **D-206.** Entra como de la empresa, y su traspaso se registra después (D-20) | §2.7, §5.4 |
| P9 | ¿Basta un tope de filas por archivo para no pasar el límite de Cloudflare? | **D-208.** Sí: la carga es síncrona, con un tope medido en su línea base, y un archivo más grande se parte | §2.3, §8.4, PRC-R151 |
