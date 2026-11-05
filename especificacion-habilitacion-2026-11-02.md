# Especificación de la habilitación — ERP Millennium Mobility

**Versión 5 · 04/11/2026: la segunda revisión de punta a punta.** El mismo día, cuatro revisores independientes volvieron a leer el plan v4, y el responsable pidió tres cambios de pantalla. Todo se integró en `develop` el 04/11/2026, sin migraciones: backend `851b4d09`, el merge de `feature/review-2-findings` (de `a126216f` a `607e6756`, sobre `1bd767f1`), y frontend `3a49f141`, el merge de su rama (de `a8ccc252` a `c64888e9`, sobre `2908405f`), con las revisiones 40 y 41 de `DESIGN.md`. En esta fase cambian: el regreso automático desde un Vendido desmarcado, que ahora pasa por la puerta (§2.3); el aviso del proceso de las 48 h, una sola vez por unidad retenida (§2.3); los candados del catálogo de estados y caminos (§2.3, §4.3); el inventario antes de V96, que suma el estado inicial y los caminos (§3.3); la fila de `kept_files` antes que el objeto (§2.6); la barra de Habilitación y de Vencimientos y sus filtros (revisión 40, §6.3); y los diálogos de la habilitación (§6.2, §6.4, §6.5, §6.6). Cada cambio lleva la marca **Como se corrigió (04/11/2026, segunda revisión)**, y §10.4 resume lo que la revisión halló en esta fase. Las propuestas para el diccionario que deja, R23 y R24, esperan al responsable (§10.4). **El diccionario, 04/11/2026:** el responsable aprobó ese día las propuestas que esta especificación dejaba pendientes, y la documentación `7cc29f4` las aplica en el diccionario v1.39: §8.2, que ningún camino salvo el regreso desde Vendido mueve sola una unidad a un estado con puerta y que el estado inicial no se marca; §13.1, los campos cuyos valores solo muestra un permiso, con el costo y la moneda de una instalación por la marca de la unidad; y §2.7, una sola regla de descarte para los archivos, también los de un documento de una unidad (§2.3, §2.6, §4.7, §10.3). **Versión 4 · 04/11/2026:** las correcciones de la revisión de punta a punta del plan v4 (03 y 04/11/2026), integradas en `develop` ese día: backend `44136ae1`, el merge de `feature/review-findings` (`5c77bc8f` a `df6b7276`), y frontend `cc782649`, el merge de su rama (`6386f2fa` a `1ef10642`). Esta versión reescribe lo que cambió de comportamiento con la marca **Como se corrigió (04/11/2026)** y su commit: la puerta fuera del alcance de los caminos que mueven solas las unidades y del estado inicial (§2.3, §4.3), el estado actual en el recurso de la habilitación (§4.5, §4.9), el costo de una instalación por la marca de la unidad y en la bitácora (§2.5, §4.7), el descarte del archivo de un documento (§2.6), el inventario antes de V96 (§3.3) y las pantallas (§6); corrige lo que decía de los lotes de JDBC (§2.6, §7), cita las pruebas que se movieron o se sumaron (§5, §7) y registra lo que encontró la revisión y lo que queda (§10). **Versión 3 · 03/11/2026:** el registro de lo construido. La fase se integró en `develop` ese día, después de la fase 2: backend `8bfdb938`, con V95 a V101 (la rama, hasta `805b12f8`), y frontend `2659126a` (la rama, hasta `4aa0ff0b`). Esta versión corrige lo que el código hizo distinto de lo especificado, con la marca **Como se construyó**, nombra la prueba real de cada regla (§5, §7) y el commit de cada fila de §8, y cambia el texto de `VEHICLE_ENABLEMENT_HELD_BY_STATUS` por los cuatro que se decidieron ese día (§4.10). **Versión 2 · 03/11/2026** (las nueve preguntas de la versión 1, del 02/11/2026, quedan decididas por D-183 a D-189, delegadas por el responsable, y la especificación se concilia con la de las facturas de compra, [versión 2](especificacion-facturas-de-compra-2026-11-02.md)). Especificación para construir de punta a punta la fase 3 del [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) (§5, punto 8 del [orden de implementación](orden-de-implementacion-millennium-mobility.md)): el bloque 9 del [diccionario](diccionario-de-dtos-millennium-mobility.md), la habilitación de las unidades, hasta que una unidad entra a Disponible.

**Punto de partida:** el del plan, backend `b1264ed5` (migraciones hasta V91) y frontend `88312e13`. Toda afirmación sobre el código existente se verificó contra esos puntos y nombra su archivo.

**Fuente:** diccionario v1.38 (§2.5 a §2.7, §5.6, §7.1 a §7.5, §8.1 a §8.5, §11, §13.1; D-28, D-29, D-74, D-100 a D-110, D-139, D-143, D-150, D-160 a D-189), la [especificación de las facturas de compra](especificacion-facturas-de-compra-2026-11-02.md) (fase 2), [convenciones de la API](convenciones-de-la-api-millennium-mobility.md), [contextos/fleet.md](contextos/fleet.md), [contextos/vehiclecatalog.md](contextos/vehiclecatalog.md), [contextos/procurement.md](contextos/procurement.md), [procesos y pantallas](procesos-y-pantallas-millennium-mobility.md) (§1, P6, P7 y §3.2), las skills `backend-module`, `frontend-feature` y `architecture-review`, la guía funcional del cliente (columnas 2, 3, 5, 12 a 33 y §6) y el flujograma maestro de estados. Si este documento y el diccionario no coinciden, manda el diccionario.

**Estado:** construida e integrada el 03/11/2026, y corregida por las dos revisiones de punta a punta del 04/11/2026 (§10); faltan las sondas en vivo, que necesitan la contraseña de verificación del responsable (§7.6), y el inventario antes de V96 sobre la base desplegada (§3.3). Las preguntas de la versión 1 están decididas (sección 9). Lo que es de la fase 2 —la anulación de la unidad, `VEHICLE_ANNULLED`, `VEHICLE_FUND_ASSIGNMENT_OPEN`, el filtro «sin factura vigente» de Inventario, los puertos `UnitPurchases` y `UnitContracts` y `FleetContextFacade`— se nombra como «fase 2» y no se redefine aquí.

**Cómo leerlo.**
- La sección 1 fija el alcance.
- Las secciones 2 a 4 son para el equipo: el modelo del backend, el esquema y el contrato de la API.
- La sección 5 es la tabla de reglas, que continúa la de los contextos: `FLT-R55` en adelante, porque FLT-R48 a FLT-R54 son de la fase 2, y `VCT-R28` en adelante. Están en ✅ desde el 03/11/2026, cada una con el nombre real de su prueba.
- La sección 6 describe las pantallas; la 7, las pruebas y lo que corrió al integrar; la 8, los commits, con el de cada fila.
- La sección 9 registra cómo quedó decidida cada pregunta de la versión 1.
- La sección 10 junta lo que encontraron las dos revisiones de punta a punta del 03 y 04/11/2026 en esta fase, cómo se cerró cada hallazgo y lo que queda abierto.

---

## 1. Alcance

### 1.1 Qué se construye

Lo que hay que completar para que una unidad pueda entregarse (§11): que ingrese físicamente a una ubicación, se inmatricule, tenga su placa y sus placas físicas, sus documentos vigentes, su equipamiento instalado y sus llaves registradas; el estado de habilitación calculado con sus seis filas (§11.9), y la puerta que ese estado pone a la entrada manual de Disponible (D-164, D-170).

### 1.2 Decisiones que implementa

| Decisión | Qué toma esta especificación |
|---|---|
| D-28 | Un documento vigente por tipo y unidad; el anterior se conserva (`vehicle_documents_current_uq`, FLT-R67) |
| D-29 | Las llaves: la etiqueta única en la empresa, el serial repetible y la caja (FLT-R82) |
| D-74 | `TIVE` queda fijo desde la semilla; `GPS` y `PANTALLA` siguen fijos, y el código de otro equipo queda fijo desde su primera instalación (FLT-R72, VCT-R30) |
| D-100 | Cada camino sigue pidiendo su permiso; la puerta es independiente del permiso del camino (FLT-R56) |
| D-101, D-102 | Las escrituras de la habilitación actúan donde está la unidad y en su marca; el ingreso actúa también en la ubicación de destino (FLT-R86) |
| D-104, D-182 | La anulación y su guarda son de la fase 2 (FLT-R48, FLT-R49): las escrituras de la habilitación la respetan y la lista deja fuera las unidades anuladas (FLT-R89) |
| D-105 | Quien instaló un equipo viaja por su nombre, sin sus datos personales (FLT-R78) |
| D-106 | La lista de habilitación y el catálogo de tipos de documento se exportan por el módulo `documents` (FLT-R88, FLT-R72) |
| D-107 | Sirve de modelo a `markEquipmentTypeInUse`, que escribe `vehiclecatalog` cuando `fleet` se lo avisa (VCT-R30) |
| D-108 | Ni un tipo de documento ni un equipo desactivado se usa en un registro nuevo (FLT-R69, FLT-R75) |
| D-109 | Un modelo sin combustible no es a gas: no se le pide el certificado de gas (FLT-R70, VCT-R29) |
| D-110 | El regreso automático de Vendido no pasa por la puerta (FLT-R57) |
| D-139 | `vehicle_document_types` por país, con la semilla de Perú, la ruta `/vehicle-document-types` y sus tres permisos (FLT-R72, FLT-R73) |
| D-143, D-167 | No hay aviso de renovación: la pantalla de Habilitación muestra los vencimientos y filtra los de los próximos 30 días (FLT-R88) |
| D-150 | Los tres permisos de los tipos de documento solo se marcan en roles de alcance `SYSTEM`; la regla de V89 los alcanza sin cambiarla (FLT-R73) |
| D-160, D-163 | La unidad nace sin ubicación, Fuera de tienda, y la recibe con su ingreso; un rol de sede actúa sobre ella desde que ingresa a su sede (FLT-R62) |
| D-164, D-170 | `vehicle_statuses.requires_enablement`, fijo en Disponible, y la puerta de todo cambio manual hacia un estado marcado (FLT-R55, FLT-R56) |
| D-183 | La puerta exige que no falte nada —ninguna de las filas 1 a 4—, esté la unidad Listo para entrega o Entregado (FLT-R56, FLT-R58) |
| D-184 | La puerta se sostiene: en un estado marcado no se retira un requisito sin reemplazo; la semilla trae el camino «Pierde un requisito», de Disponible a En habilitación, y marca también Vendido, que se puede desmarcar (FLT-R55, FLT-R85, FLT-R93) |
| D-185 | `equipment_types.has_installations`, que marca `markEquipmentTypeInUse` (VCT-R30) |
| D-186 | Ninguna fecha de la habilitación es futura en la zona de la empresa, y una instalación no es anterior al ingreso (FLT-R62, FLT-R64, FLT-R68, FLT-R76) |
| D-187 | El costo lo escribe quien lo ve: sin `stock.ver_costos` se guarda en cero, y quien lo ve lo registra o lo corrige después (FLT-R80) |
| D-188 | Un documento bloqueante vencido cuenta; una unidad ingresada no queda sin ubicación; sin documento de identificación en el país, no hay placa (FLT-R58, FLT-R63, FLT-R65) |
| D-189 | Los archivos de la habilitación van a `kept_files` con la empresa de su unidad (FLT-R71) |
| D-165 | `company_id` en las tablas de la habilitación; lo bloqueante leído de los datos; las placas físicas; el ingreso por unidad o en lote, que fija la ubicación; el kilometraje sigue siendo la última lectura (FLT-R60, FLT-R62, FLT-R66, FLT-R83) |
| D-166 | El certificado de gas, solo a las unidades cuyo modelo usa GNV o GLP (FLT-R70) |
| D-171 | Los seis códigos estables del estado de habilitación y su detalle (FLT-R58) |
| D-174 | Cada documento es PDF, PNG, JPEG o WebP, de a lo sumo 5 MB, con el tipo leído de su contenido (FLT-R71) |
| D-175 | Solo se nota: el filtro «sin factura vigente» es de Inventario y de la fase 2 (su FLT-R51, `hasCurrentInvoice` sobre el puerto `UnitPurchases`). La lista de habilitación no lo repite ni usa ese puerto; si hace falta después, suma el mismo filtro sin cambiar su contrato |

### 1.3 Pasos del plan

| Paso | Qué cubre esta especificación | Sección |
|---|---|---|
| 8.0 (nuevo) | El contrato de la API en las convenciones y en `contextos/fleet.md`, copiado de la sección 4 antes de escribir código, como el paso 5.0 de las facturas | 4, 8 |
| 8.1 | Los siete permisos con sus constantes y sus diecisiete marcas | 3.2 |
| 8.2 | El esquema: las columnas de §11.1, `requires_enablement`, las cuatro tablas, la columna de `vehiclecatalog` | 3 |
| 8.3 | La fachada de `vehiclecatalog`: el perfil de habilitación de un lote de modelos, el tipo de equipo y `markEquipmentTypeInUse` | 2.7 |
| 8.4 | Los comandos: el ingreso, los hitos, la placa, los documentos, los equipos y las llaves | 2.6, 4 |
| 8.5 | `EnablementPolicy`, la puerta, lo que retiene un estado con puerta, `allowedActions` y la proyección de la lista | 2.2 a 2.5, 2.9 |
| 8.6 | Catálogos › Tipos de documento y la marca de Estados y motivos | 6.1, 6.2 |
| 8.7 | Logística › Habilitación: la lista, los vencimientos, la exportación y el ingreso en lote; el botón de la ficha de la factura, que depende del paso 5.11 (hallazgo M5) | 6.3, 6.6 |
| 8.8 | La habilitación de la unidad, en una página propia | 6.4, 6.5 |
| 8.9 | El cierre: el contexto de flota, P6, el plan, las sondas en vivo | 7.6, 8 |

### 1.4 Lo que queda fuera

- **El paquete de cargas de archivos** (plan §5): `import_batches`, `vehicle_tax_declarations` —que lleva `company_id` por D-165 cuando se cree—, la carga del impuesto vehicular, la de garantía mobiliaria y la carga inicial con sus facturas, sus notas y sus hitos (D-168, D-177), y sus permisos `habilitacion.cargar_archivo_sat`, `habilitacion.ver_cargas` y `fondos.cargar_archivo_garantia`, que esta fase no siembra.
- **El aviso de renovación** por correo, SMS o push: espera al módulo `notifications` (D-143, D-167).
- **Los costos de la unidad en su ficha** y el exportable de stock (capa de lectura, convenciones §14.19, paso 11). El costo de cada instalación sí se guarda y se muestra en la habilitación a quien tiene `stock.ver_costos` (FLT-R80).
- **Las pólizas de seguro**, que siguen como candidato (§11.6).
- **La fecha en que la unidad pasó a lista para entrega** (guía, columna 51): sale del historial de estados (§8.4, §11.9) y la muestra el exportable de stock, no esta fase.

---

## 2. Modelo del backend

### 2.1 Contextos y dependencias

| Contexto | Qué cambia | `allowedDependencies` |
|---|---|---|
| `fleet` | Dueño del bloque 9 (§3): las cuatro tablas, las columnas de `vehicles`, la puerta y la lista | Suma `"location::acl"`: la moneda del costo de cada equipo, que el mapa de contextos ya le permite (`contextos/fleet.md`, sección 5). Queda `{"shared", "organization::acl", "vehiclecatalog::acl", "iam::acl", "documents::acl", "location::acl"}` |
| `vehiclecatalog` | Nuevos miembros de su fachada y la columna `equipment_types.has_installations` | Sin cambio (`{"documents::acl"}`): no llama a `fleet` ni lee sus tablas; la escritura de `markEquipmentTypeInUse` va en el mismo sentido que las lecturas, como `markModelInUse` (VCT-R14) |
| `organization` | Ninguno propio: usa `fetchPurchasingCompaniesByIds` y `PurchasingCompany`, que define la fase 2 (su §2.5.2), por la moneda funcional y la zona de la empresa | Sin cambio |
| `location` | Ninguno: `fleet` implementa el puerto `CurrencyUsage` de `location::acl` | Sin cambio |
| `documents` | Ninguno propio: usa `KeptFileStore.discardRolledBack`, que define la fase 2 (su §2.5.5) | Sin cambio |
| `iam` | La migración de los permisos | Sin cambio |
| `shared` | `UploadedFileType`, el tipo de un archivo leído de su contenido, que también usa compras (sección 2.10) | — |
| `procurement` | Solo en el frontend: el botón de ingreso de la ficha de la factura (6.6) | — |

`ModularityTests` falla si `fleet` usa `location` sin declararlo, o si alguien abre el interior de otro contexto.

**Como se construyó.** `"location::acl"` y `ExternalLocationService` llegaron con el commit 17 (`7dd80e2b`) y no con el 19: los tipos de documento nombran su país por el código ISO, que es de `location` (`fetchCountrySummaryByIsoAlpha2`, `fetchCountrySummariesByIds` y su `fetchDataVersion()`). El commit 19 (`5aefa129`) le sumó la moneda del costo y `VehicleEquipmentCurrencyUsageAdapter`.

**Lo que esta fase no necesita de la fase 2.** No usa `fleet/interfaces/acl`, `FleetContextFacade` ni los puertos `UnitPurchases` y `UnitContracts`: ningún contexto llama a la habilitación por el backend, y el botón de la ficha de la factura usa la fachada del frontend (6.6). Si esta fase se integra primero, no crea ese paquete; si más adelante otro contexto necesita algo de la habilitación, sus miembros se suman a `FleetContextFacade`, que crea la fase 2, sin cambiar los que ya tiene.

**Los miembros de fachada que comparten las dos fases** son aditivos: los crea la fase que se integra primero, con la forma que fija su especificación, y la otra los reutiliza. Son `fetchPurchasingCompaniesByIds` con `PurchasingCompany` en `organization` y `KeptFileStore.discardRolledBack` en `documents`.

### 2.2 `EnablementPolicy`: el estado de habilitación

Una política pura, en `fleet/domain/services/EnablementPolicy`, sin Spring ni base, como `StatusPathPolicy` y `MileageSeriesPolicy`. Recibe los hechos de una unidad y devuelve su estado. No sabe de catálogos ni de otros contextos: lo que bloquea le llega ya resuelto, como datos (D-165).

**Lo que recibe** (`fleet/domain/model/valueobjects/EnablementFacts`):

| Campo | Tipo | De dónde sale |
|---|---|---|
| `arrived` | `boolean` | `vehicles.arrival_date IS NOT NULL` |
| `requiredDocuments` | `List<EnablementRequirement>` | Los tipos de documento del país de la unidad (`vehicles.country_id`) activos y con `blocks_availability`, salvo los de `requires_gas_conversion` cuando el modelo no es a gas (FLT-R60, FLT-R70) |
| `currentDocumentTypeIds` | `Set<Long>` | Los tipos de los documentos de la unidad con `is_current`. Un documento vigente cuenta aunque haya vencido: el vencimiento se sigue en Vencimientos (D-188, D-167) |
| `requiredEquipments` | `List<EnablementRequirement>` | Los equipos que exige el modelo, que responde `vehiclecatalog` (VCT-R28) |
| `installedEquipmentTypeIds` | `Set<Long>` | Los tipos de los equipos de la unidad con el período abierto (`upper_inf(period)`) |
| `platesReceived` | `boolean` | `vehicles.plates_received_date IS NOT NULL`, que la base solo admite con placa (`vehicles_plates_received_ck`) |
| `delivered` | `boolean` | Si el historial tiene un cambio hacia el estado `ACTIVO` (§11.9: la entrega es el paso a Activo) |

`EnablementRequirement` es `(Long id, String code, String name)`: el identificador interno, el código y el nombre de un tipo de documento o de equipo.

**Las seis filas**, en este orden; la primera que aplica es el estado (§11.9, D-171):

| Orden | Si… | `EnablementStatusCode` | `detail` |
|---|---|---|---|
| 1 | `!arrived` | `FUERA_DE_TIENDA` | vacío |
| 2 | algún `requiredDocuments` no está en `currentDocumentTypeIds` | `SIN_DOCUMENTO` | los documentos que faltan, por código |
| 3 | algún `requiredEquipments` no está en `installedEquipmentTypeIds` | `SIN_EQUIPO` | los equipos que faltan, por código |
| 4 | `!platesReceived` | `SIN_PLACAS` | vacío |
| 5 | `!delivered` | `LISTO_PARA_ENTREGA` | vacío |
| 6 | `delivered` | `ENTREGADO` | vacío |

**Lo que devuelve** (`EnablementStatus`): `code`, `detail` (lo que falta de la fila que respondió), y además el cuadro completo, para que la puerta y la pantalla nombren todo lo que falta de una vez: `arrived`, `missingDocuments`, `missingEquipments` y `platesReceived`. `isComplete()` es verdadero cuando ninguna de las filas 1 a 4 aplica: el código es `LISTO_PARA_ENTREGA` o `ENTREGADO` (D-183). Así una unidad reacondicionada, que ya se entregó alguna vez y por eso sale Entregado, vuelve a Disponible por «Reacondicionada».

**El estado de entrega.** El código `ACTIVO` lo lee esta regla, como lo leen §11.9 y el flujograma; el código de un estado nunca cambia (FLT-R14), así que no hace falta fijarlo de nuevo. La constante es `VehicleStatus.DELIVERY_CODE = "ACTIVO"`, y el servicio resuelve su identificador una vez por operación. Si el catálogo no tuviera ese estado, ninguna unidad estaría entregada. `deliveredAt`, el momento de la primera entrega (guía, columna 23), sale de la primera fila del historial hacia ese estado.

**Nada se guarda** (§11.9, §2.2): el estado se calcula en cada lectura, así que cambia solo cuando cambia una condición —un documento, un equipo, un hito, el catálogo—. Si el superadministrador cambia `has_factory_screen` o `fuel_type` de un modelo con unidades, o marca otro tipo de documento o de equipo como bloqueante, el estado de esas unidades cambia en la lectura siguiente, sin migración ni proceso (FLT-R91). Las unidades que ya están en un estado con puerta se quedan donde están: la puerta se aplica al entrar.

### 2.3 La puerta de los estados

**Dónde se engancha.** `Vehicle.transitionTo(Long statusId, StatusPath path, EntryGate gate)` reemplaza a `transitionTo(Long, StatusPath)` (`fleet/domain/model/aggregates/Vehicle.java:146`): la unidad se niega a moverse por un camino que no lo permite y, desde esta fase, también a entrar sin la habilitación completa a un estado que la exige. `EntryGate` (`domain/model/valueobjects`) es `EntryGate.none()` cuando el destino no tiene `requires_enablement`, o `EntryGate.requiring(EnablementStatus)` cuando lo tiene; `gate.refuses()` es verdadero cuando la exige y el estado no está completo. La unidad repite el veredicto de `StatusPathPolicy` y después consulta la puerta; si la puerta rechaza, lanza `EnablementIncompleteException(EnablementStatus)` y queda donde estaba. No queda una versión de `transitionTo` sin puerta: el único llamador es `VehicleMovements`, y las pruebas de `Vehicle` pasan `EntryGate.none()`.

**Quién arma la puerta.** `VehicleMovements.move` (`application/internal/commandservices/VehicleMovements.java:90`) sigue leyendo el destino y el camino y traduciendo el veredicto del camino, como hoy, pero ya no mueve la unidad: consulta `StatusPathPolicy.check` y devuelve el `Move`. La unidad se mueve en `VehicleMovements.recordMove`, que pasa a devolver un `Result`: si el destino tiene `requires_enablement`, pide a `VehicleEnablementFactsReader` los hechos de la unidad, ya bloqueada, y a `EnablementPolicy` su estado, arma `EntryGate.requiring(status)` y llama a `transitionTo`. Si el destino no la exige, no lee nada más. Traduce la excepción a `422 VEHICLE_ENABLEMENT_INCOMPLETE`, con el estado en `context` (sección 4.10).

**El orden de las respuestas** de `POST /vehicles/{id}/status-changes` (`VehicleCommandServiceImpl.handle(ChangeVehicleStatusCommand)`, línea 234). Los pasos 1 a 4 son los de hoy, y el 5 lo pone la fase 2, que fija la guarda de la unidad anulada después de su `404` y de su `403` (su §4.2.10):
1. La unidad, bloqueada (`FOR NO KEY UPDATE`) y vista con `stock.ver` (`404 VEHICLE_NOT_FOUND`).
2. El destino (`404 VEHICLESTATUS_NOT_FOUND`, `422 VEHICLESTATUS_INACTIVE`) y el camino (`404 STATUSCHANGEREASON_NOT_FOUND`).
3. El veredicto del camino (`422 STATUSCHANGEREASON_AUTOMATIC_NOT_SELECTABLE`, `STATUSCHANGEREASON_TARGET_MISMATCH`, `STATUS_TRANSITION_NOT_ALLOWED`).
4. El permiso del camino donde está la unidad (`403 ACCESS_DENIED`, D-100).
5. La unidad no anulada (`422 VEHICLE_ANNULLED`, fase 2).
6. **La puerta** (`422 VEHICLE_ENABLEMENT_INCOMPLETE`).
7. La escritura de la unidad y su fila del historial.

La puerta va al final de las revisiones: a una unidad anulada se le dice que está anulada, y a quien no puede recorrer el camino, que no puede, antes de nombrarle lo que falta.

**El regreso automático no pasa por la puerta** (D-164, D-110). `VehicleMovements.returnAlone` mueve con `Vehicle.transitionAlone`, que no consulta la puerta: deshace una venta que no ocurrió. Una unidad Vendida no pierde un requisito por una escritura (sección 2.4), pero puede quedar incompleta por un cambio del catálogo, como un tipo de documento que pasa a bloquear: vuelve igual a Disponible a las 48 h, y la lista de habilitación la muestra con lo que le falta (FLT-R57, FLT-R91). Es la única exención: ningún otro camino mueve sola una unidad a un estado con puerta (abajo, **Como se corrigió**). Desde la segunda revisión del 04/11/2026, la exención vale solo mientras Vendido tenga la marca (abajo).

**Las marcas.** `vehicle_statuses.requires_enablement` es de `VehicleStatus` (`domain/model/aggregates/VehicleStatus.java`), que suma el campo y la constante `AVAILABLE_CODE = "DISPONIBLE"`. La semilla lo marca en Disponible y en Vendido (D-164, D-184):
- **en Disponible queda fijo** (D-170): `update(name, category, generatesRevenue, isInitial, requiresEnablement)` rechaza dejarlo en falso con `EnablementGateLockedException`, que el servicio traduce a `422 VEHICLESTATUS_ENABLEMENT_LOCKED`, y la base lo repite con `vehicle_statuses_enablement_gate_ck` (sección 3.3);
- **en Vendido se puede desmarcar**, con `catalogos.editar_estados`, como en cualquier otro estado. Lo marca la guía, que exige todos los requisitos para entregar (columna 22): mientras la unidad está vendida, conserva lo que la habilitó.

El alta de un estado no recibe la marca: nace en falso, y se marca con `PUT`, con `catalogos.editar_estados` (D-170).

**Como se corrigió (04/11/2026): la puerta no se apaga desde el catálogo** (backend `345c1f75`; frontend `7d5e5dce`, `660d15dc`, `891f3fa0`). La revisión encontró dos caminos para saltarla sin tocar la marca de Disponible: un camino que espera en otro estado y, al vencerse, mueve sola la unidad a Disponible —el proceso de las 48 h no consulta la puerta—, y un estado inicial marcado, donde nace una unidad sin nada. Desde este commit:
- **Solo el regreso desde Vendido mueve solas unidades a un estado con puerta.** La regla es pura, `AutomaticReturnGate.applies(códigoDondeEspera, destinoExige)` (`domain/services`): aplica cuando el destino exige la habilitación y el camino no espera en Vendido, por el código fijo `VehicleStatus.SOLD_CODE = "VENDIDO"`, que no cambia (FLT-R14).
- **El catálogo rechaza los demás** con `422 STATUSCHANGEREASON_AUTO_TARGET_GATED` y el código del estado en `context.statusCode`: al declarar un camino, al editarlo abierto y al reabrirlo. Un camino cerrado no se revisa hasta que se reabre.
- **Un estado no se marca** si un camino abierto que espera en otro estado que Vendido lo tiene como destino al vencerse: `422 VEHICLESTATUS_ENABLEMENT_AUTO_TARGET`, con los códigos de esos caminos en `context.reasonCodes`. Solo se revisa cuando la marca pasa de falso a verdadero.
- **El estado inicial no se marca**, ni un estado marcado pasa a ser el inicial: `422 VEHICLESTATUS_ENABLEMENT_INITIAL`, sin `context`. Así el alta, que deja la unidad en el estado inicial, nunca cruza una puerta.
- **El proceso de las 48 h se defiende igual** de un camino guardado antes de este commit: si movería sola una unidad incompleta a un estado con puerta desde otro estado que Vendido, la deja donde está y lo registra en el log con `WARN` (`VehicleMovements.returnAlone`). El regreso desde Vendido sigue exento (FLT-R57). Desde la segunda revisión, el `WARN` sale solo en la primera corrida después del vencimiento, y el regreso desde Vendido queda exento solo mientras Vendido tenga la marca (abajo).

La semilla trae un solo camino que mueve sola una unidad, «Venta», que espera en Vendido y vuelve a Disponible, y sigue permitido; el 04/11/2026 era también el único en `erp_mm_db` y en `erp_mm_db_verify`. Lo prueban las cuatro de `AutomaticReturnGateTests` (§5, FLT-R55 y FLT-R57). §8.2 del diccionario lo dice desde la v1.39, aprobada por el responsable el 04/11/2026 (documentación `7cc29f4`).

**Como se corrigió (04/11/2026, segunda revisión): el regreso desde un Vendido desmarcado** (backend `a126216f`; hallazgo alto). El proceso eximía de la puerta, por su código, todo camino que espera en Vendido. Pero la exención se apoya en que Vendido, marcado, sostiene los requisitos de la unidad mientras espera (D-184, sección 2.4). Si el superadministrador desmarca Vendido, nada los sostiene: a una unidad vendida se le puede retirar el GPS, y a las 48 h el regreso la dejaba en Disponible sin la habilitación completa, lo que D-170 prohíbe. Ahora:
- **El proceso:** el regreso automático de Vendido no aplica la puerta mientras Vendido tenga `requires_enablement`, porque entonces la sostuvo completa (D-184). Si el superadministrador desmarca Vendido, el proceso lee la habilitación como en cualquier otro camino y deja en Vendido a la unidad a la que le falta algo (D-170). Es la regla pura `AutomaticReturnGate.appliesToTheReturn(códigoDondeEspera, esperaConMarca, destinoExige)`, que `VehicleMovements.returnAlone` consulta.
- **El catálogo** mantiene la exención por el código `VENDIDO` al guardar un camino (`AutomaticReturnGate.applies`, sin cambios): un camino de Vendido a Disponible sigue permitido aunque Vendido esté desmarcado, porque el proceso lo vigila al moverse.

Pruebas: `AutomaticReturnGateRuleTests` (`theCatalogExemptsTheReturnFromVendidoByItsCode`, `theJobExemptsTheReturnFromVendidoOnlyWhileVendidoHoldsTheRequirements`, `theJobGatesAnyOtherPathIntoAGatedStatusAndNoPathIntoAnUngatedOne`); `EnablementGateIntegrationTests.anUnmarkedVendidoReturnsAloneOnlyAUnitThatLacksNothing`, `aMarkedVendidoReturnsTheSoldUnitAlone`. §8.2 y D-164 del diccionario todavía dicen la exención sin esta condición: las propuestas R23 y R24 esperan al responsable (§10.4).

**Como se corrigió (04/11/2026, segunda revisión): un solo aviso por unidad retenida** (backend `62708a60`). Una unidad que el proceso deja donde está sigue vencida, así que cada corrida la volvía a encontrar y escribía el mismo `WARN`, para siempre. Ahora solo la primera corrida después del vencimiento avisa con `WARN`: la que mide dentro de un intervalo del proceso (`fleet.status-tolerances.check-interval`) después del momento en que venció la tolerancia (`VehicleMovements.firstRunAfterTheBreach`); las siguientes lo dicen en `DEBUG`. El intervalo cuenta desde el final de una corrida (`fixedDelay`), así que una corrida que tarda puede dejar el primer aviso en `DEBUG`. Pruebas: `VehicleMovementsBreachWarningTests` (`theRunWithinOneIntervalAfterTheBreachIsTheFirst`, `aRunMoreThanOneIntervalAfterTheBreachIsNot`), `AutomaticReturnGateTests.theJobWarnsAboutAHeldUnitOnlyOnTheFirstRunAfterItsBreach`.

**Como se corrigió (04/11/2026, segunda revisión): el catálogo, sin carreras** (backend `5bcd796b`; D-164, D-170). Marcar un estado leía sus caminos abiertos sin candado, y guardar o reabrir un camino leía sin candado si su destino automático estaba marcado. Dos pedidos a la vez pasaban cada uno su revisión y dejaban un estado marcado con un camino que mueve solas las unidades hacia él desde otro estado que Vendido. Ahora:
- **guardar o reabrir un camino** lee su destino automático `FOR SHARE` (`VehicleStatusRepository.findByIdForShare`, `findByPublicIdForShare`) hasta guardar el camino;
- **marcar un estado, o cambiar su actividad,** lo lee `FOR NO KEY UPDATE` (`findByPublicIdForUpdate`) antes de leer sus caminos.

El segundo pedido espera al primero y responde con lo que este confirmó. `FOR NO KEY UPDATE` choca con `FOR SHARE` como `FOR UPDATE`, y deja pasar los movimientos de unidades hacia el estado, que solo toman `FOR KEY SHARE` sobre él. Prueba: `StatusCatalogLockTests` (`markingAStatusWaitsForAPathReopenedIntoItAndSeesIt`, `reopeningAPathWaitsForTheMarkOfItsDestinationAndSeesIt`), con dos transacciones reales en los dos órdenes.

**La puerta también vale al entrar a Vendido.** La «Venta» lleva de Disponible a Vendido, un estado marcado, así que pasa por la puerta. Una unidad en Disponible conserva sus requisitos (sección 2.4), así que la puerta solo detiene una venta si un cambio del catálogo dejó incompleta a la unidad mientras estaba en Disponible; entonces se completa primero, o sale por «Pierde un requisito».

### 2.4 La puerta se sostiene

Mientras la unidad está en un estado con `requires_enablement` —hoy Disponible y Vendido—, `RequirementHoldPolicy` (`domain/services`, pura) rechaza lo que la dejaría sin un requisito de la habilitación, con `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS` (D-184):
- retirar sin reemplazo un equipo abierto de un tipo que la unidad exige (`requiredEquipments`): el GPS y la pantalla cuando corresponde; reemplazarlo sí se puede, porque cierra uno y abre otro en la misma operación;
- vaciar `plates_received_date`;
- vaciar `arrival_date`.

Un documento no se puede retirar (§11.3: se reemplaza), y el título no se vacía mientras la unidad tiene placa (FLT-R64), así que no hacen falta más casos. En un estado sin puerta —En habilitación, Activo, Mecánica, Custodia— el retiro de un GPS que falló se registra igual.

**La salida: «Pierde un requisito».** Cuando un GPS falla en una unidad Disponible y no hay repuesto, la unidad sale primero de Disponible por el camino que trae la semilla (sección 3.3, FLT-R93):

| Desde | Hacia | Código | Motivo | Tolerancia | Permiso |
|---|---|---|---|---|---|
| Disponible | En habilitación | `PIERDE_REQUISITO` | Pierde un requisito | — | `stock.habilitar` |

En habilitación no tiene puerta, así que allí el equipo se retira; cuando la unidad vuelve a estar completa, regresa a Disponible por «Habilitación completa». Es un camino como cualquier otro: un dato de `status_change_reasons` (D-19), con el permiso que nombra (D-100), que el superadministrador puede cerrar o editar en Estados y motivos. Una unidad Vendida no tiene un camino así: vuelve a Disponible por el regreso automático o se entrega; si el negocio lo pide, se carga otro camino, que es un dato.

Es lo que cumple el criterio del plan: retirar el GPS mientras otra persona habilita la unidad nunca deja una unidad Disponible sin GPS, sea cual sea el orden en que las dos operaciones toman la unidad (sección 2.6, FLT-R84).

### 2.5 `allowedActions`

**De la unidad** (`VehicleAllowedActionsView`, `application/queryservices`). `VehicleAllowedActions.of` (`application/internal/queryservices/VehicleAllowedActions.java`) calcula los caminos como hoy y separa los que lleva a un estado con puerta mientras la unidad no está completa:
- `walkableReasonIds` deja de incluir esos caminos: la operación los rechazaría;
- `heldReasonIds`, nuevo, los lista: el permiso del camino actúa, el camino está abierto y su destino activo, pero la habilitación no está completa. La pantalla los muestra deshabilitados con lo que falta, que viaja en `enablement` del mismo recurso.

Evalúa la habilitación una sola vez por ficha, solo si alguno de los caminos va a un estado con puerta.

**De la habilitación de una unidad** (`VehicleEnablementAllowedActionsView`), un objeto de booleanos como el de la unidad, calculado por `VehicleEnablementAllowedActions` con las mismas reglas que cada operación: el permiso donde está la unidad, su marca y, para una unidad en la ubicación propia de una sede, esa sede (D-101, D-102, `VehicleAccessPolicy.actsOn`). No repite los `422` (convenciones §4.2): la pantalla explica con los datos del mismo recurso por qué una acción no aplica todavía.

| Acción | Verdadera si |
|---|---|
| `registerArrival` | La unidad no ingresó y `habilitacion.registrar_hitos` actúa sobre ella |
| `correctArrival` | La unidad ingresó y `habilitacion.registrar_hitos` actúa sobre ella |
| `editMilestones` | `habilitacion.registrar_hitos` actúa sobre ella |
| `registerPlate` | La unidad no tiene placa y `habilitacion.registrar_hitos` actúa sobre ella |
| `correctPlate` | La unidad tiene placa y `stock.corregir_placa` actúa sobre ella (FLT-R03) |
| `uploadDocuments` | `habilitacion.gestionar_documentos` actúa sobre ella |
| `installEquipment` | `habilitacion.gestionar_equipos` actúa sobre ella |
| `registerKeys` | `habilitacion.gestionar_llaves` actúa sobre ella |

Ninguna se ofrece sobre una unidad anulada.

**De cada equipo**, una lista en orden fijo, como la de la cesión:
- `replace` y `remove`, solo con el período abierto y `habilitacion.gestionar_equipos` sobre la unidad; `remove` no se ofrece cuando el estado lo retiene (sección 2.4);
- `correctCost`, con ese permiso y `stock.ver_costos` en la empresa y la marca de la unidad (D-187, D-102). **Como se corrigió (04/11/2026)** (`f5eacfe7`): la versión 3 lo pedía solo en la empresa, así que una asignación de los costos de otra marca de la empresa leía y escribía el costo; ahora lo decide `VehicleAccessPolicy.seesCosts`, como el costo de compra de la unidad (§4.7).

**De cada llave**: `edit`, con `habilitacion.gestionar_llaves`.

**Como se construyó.** `VehicleEnablementAllowedActions` llegó con el commit 13 (`d8a4afd1`) y no con el 23, para que `GET /vehicles/{id}/enablement` respondiera sus acciones desde el principio. El 23 (`118f3b11`) sumó `heldReasonIds` y `enablement` a la unidad. `VehicleView` y `VehicleResource` llevan los dos campos, y en las filas de `GET /vehicles` `enablement` viaja en `null` (sección 4.4).

### 2.6 Comandos, transacciones y candados

**Las piezas.** Un servicio de comandos por agregado, en `application/internal/commandservices`, con el nombre que envuelve `DeadlockRetry` (`*CommandServiceImpl`) y `@Transactional`, cada uno por debajo de unas 300 líneas (F5). Los errores, en una clase por agregado en `application/internal/errors` (`EnablementErrors`, `VehicleDocumentTypeErrors`, `VehicleDocumentErrors`, `VehicleEquipmentErrors`, `VehicleKeyErrors`), porque `FleetErrors` ya pasa de las 290 líneas (hallazgo A28).

| Servicio | Comandos (`domain/model/commands`) | Piezas que usa |
|---|---|---|
| `VehicleEnablementCommandServiceImpl` | `RegisterArrivalsCommand(List<UUID> vehicleIds, LocalDate arrivalDate, UUID locationId)`; `CorrectArrivalCommand(UUID vehicleId, @Nullable LocalDate arrivalDate, @Nullable UUID locationId, Instant updatedAt)`; `UpdateVehicleMilestonesCommand(UUID vehicleId, Instant updatedAt, @Nullable LocalDate registrationStartDate, @Nullable String titleNumber, @Nullable LocalDate titleDate, @Nullable LocalDate platesReceivedDate)`; `RegisterVehiclePlateCommand(UUID vehicleId, String plate)` | `VehicleArrivals` (el lote), `MilestoneChangeLog` (las filas de la bitácora), `VehicleEnablementFactsReader`, `RequirementHoldPolicy` |
| `VehicleDocumentTypeCommandServiceImpl` | `CreateVehicleDocumentTypeCommand(String country, String code, String name, boolean hasExpiry, boolean blocksAvailability, boolean isVisibleToCustomer, boolean requiresGasConversion)`; `UpdateVehicleDocumentTypeCommand(UUID id, String code, String name, boolean hasExpiry, boolean blocksAvailability, boolean isVisibleToCustomer, boolean requiresGasConversion)`; `ChangeVehicleDocumentTypeStateCommand(UUID id, boolean isActive)` | — |
| `VehicleDocumentCommandServiceImpl` | `UploadVehicleDocumentCommand(UUID vehicleId, UUID documentTypeId, @Nullable LocalDate issueDate, @Nullable LocalDate expiryDate, String fileName, byte[] bytes)` | `VehicleDocumentFiles` (la llave y `KeptFileStore`) |
| `VehicleEquipmentCommandServiceImpl` | `InstallVehicleEquipmentCommand(UUID vehicleId, UUID equipmentTypeId, @Nullable String serial, LocalDate startDate, @Nullable UUID installedByEmployeeId, @Nullable CostInput cost)`; `ReplaceVehicleEquipmentCommand(UUID equipmentId, @Nullable String serial, LocalDate startDate, @Nullable UUID installedByEmployeeId, @Nullable CostInput cost)`; `RemoveVehicleEquipmentCommand(UUID equipmentId, LocalDate endDate, String removalReason)`; `CorrectVehicleEquipmentCostCommand(UUID equipmentId, CostInput cost)` (D-187) | `VehicleEquipmentInstallations` (las reglas comunes al alta y al reemplazo), `RequirementHoldPolicy` |
| `VehicleKeyCommandServiceImpl` | `RegisterVehicleKeyCommand(UUID vehicleId, String keyType, String tagCode, @Nullable String serial, @Nullable String storageGroup, String holder)`; `UpdateVehicleKeyCommand(UUID keyId, Instant updatedAt, String keyType, String tagCode, @Nullable String serial, @Nullable String storageGroup, String holder)` | — |
| `VehicleStatusCommandServiceImpl` (existe) | `UpdateVehicleStatusCommand` suma `boolean requiresEnablement` | — |
| `VehicleCommandServiceImpl` (existe) | `ChangeVehicleStatusCommand` sin cambio | `VehicleMovements` con la puerta |

`CostInput` es `(BigDecimal amount, String currency)`.

**La unidad anulada es de la fase 2.** Cada comando de la habilitación revisa, en este orden: la unidad (`404`), el permiso donde está (`403`) y, después, la guarda de la unidad anulada de la fase 2 (`Vehicle.isAnnulled()`, `422 VEHICLE_ANNULLED`, su FLT-R49), como fija la fase 2 para toda escritura de una unidad; recién entonces sus propias reglas (`422`). Si esta fase se integra primero, los comandos no la revisan todavía: ninguna unidad se puede anular antes de la fase 2, que no tiene ruta para hacerlo, y al integrarse la fase 2 suma la guarda a estas escrituras como a las demás. La lista de habilitación deja fuera las unidades anuladas en los dos casos, porque `annulled_at` existe desde V47.

**El orden de los candados.** Todo lo que escribe sobre una unidad la bloquea primero (`FOR NO KEY UPDATE`, `VehicleJpaRepository.lockByPublicId`, línea 35) y después, si hace falta, la fila hija (`FOR NO KEY UPDATE`), siempre en ese orden: unidad y luego hija. Lo que lee para decidir lo lee **después** del candado, así que en `READ COMMITTED` ve lo que confirmó la operación que lo tenía antes.

| Operación | Candados, en orden | Lo que lee después del candado |
|---|---|---|
| `POST /vehicle-arrivals` | Todas las unidades del lote con una sola sentencia, en orden de `id` (`SELECT id FROM vehicles WHERE public_id = ANY(:ids) ORDER BY id FOR NO KEY UPDATE`) | Las unidades, sus ubicaciones actuales en un lote, la ubicación de destino |
| `PUT /vehicles/{id}/arrival` | La unidad | Su `updatedAt`, su estado, el inicio más antiguo de sus equipos |
| `PUT /vehicles/{id}/milestones` | La unidad | Su `updatedAt`, su estado y su placa |
| `POST /vehicles/{id}/plate` | La unidad | Su título, su TIVE vigente; la unicidad en el país con una elevación a `SYSTEM` que solo responde sí o no, como el alta (FLT-R01) |
| `POST /vehicles/{id}/documents` | La unidad; después el documento vigente de ese tipo | El tipo, el modelo (si es a gas) |
| `POST /vehicles/{id}/equipments` | La unidad | Su ingreso, sus equipos abiertos del tipo, el tipo de equipo (sin candado, como el modelo en el alta, FLT-R44), la moneda en modo compartido (`fetchCurrencySummaryByIsoCodeForShare`, el contrato de `CurrencyUsage`) |
| `POST /vehicle-equipments/{id}/replacement` y `/removal` | La unidad del equipo; después el equipo | Su estado y sus requisitos (2.4), el período del equipo |
| `POST /vehicles/{id}/keys`, `PUT /vehicle-keys/{id}` | La unidad; en la edición, después la llave | — |
| `POST /vehicles/{id}/status-changes` | La unidad | Los hechos de la habilitación, solo si el destino tiene puerta |

**La carrera del GPS** (criterio del plan). La persona A mueve la unidad a Disponible y la persona B retira su GPS. Las dos bloquean la misma fila de `vehicles`, así que una espera a la otra:
- si B confirma primero, A lee después del candado que el GPS no está y recibe `422 VEHICLE_ENABLEMENT_INCOMPLETE`;
- si A confirma primero, B lee después del candado que la unidad está en Disponible y recibe `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS` (sección 2.4).

Ningún orden deja una unidad Disponible sin GPS. Un `INSERT` en una tabla hija toma `FOR KEY SHARE` sobre la unidad por su llave compuesta, que no choca con `FOR NO KEY UPDATE`: el candado de una persona no bloquea las lecturas de otra.

**El ingreso en lote** (`VehicleArrivals`). Valida todo antes de escribir y escribe todo o nada:
1. El cuerpo (`400`): de 1 a 100 identificadores distintos, la fecha y la ubicación.
2. Bloquea las unidades en orden de `id`. Las que no existen o no se ven con `stock.ver` responden `404 VEHICLE_NOT_FOUND`, con sus identificadores en `context.vehicleIds`.
3. Todas son de una empresa (`422 VEHICLE_ARRIVAL_COMPANY_MISMATCH`).
4. La ubicación existe y se ve (`404 LOCATION_NOT_FOUND`), es de esa empresa (`422 LOCATION_OTHER_COMPANY`) y está activa (`422 LOCATION_INACTIVE`).
5. `habilitacion.registrar_hitos` actúa sobre cada unidad donde está y en la ubicación de destino (`403 ACCESS_DENIED`): un rol de sede no recibe una unidad sin ubicación, que es de los roles de empresa (D-101), ni la manda fuera de su sede.
6. Ninguna anulada (`422 VEHICLE_ANNULLED`, de la fase 2; en el lote, con las unidades en `context.vehicleIds`).
7. Ninguna ingresada (`422 VEHICLE_ALREADY_ARRIVED`, con `context.vehicleIds`).
8. La fecha no es futura en la zona de la empresa (`422 VEHICLE_MILESTONE_IN_FUTURE`, D-186).

Después escribe: cada unidad con `Vehicle.registerArrival(fecha, ubicación)`, guardadas con `saveAll`, y la bitácora con una sola llamada a `ChangeLogCommandService.recordChanges`.

**Como se corrigió (04/11/2026): no hay lotes de JDBC.** La versión 3 decía que las unidades se guardaban «en lotes de JDBC» y que las sentencias no crecían «salvo las de los lotes». La revisión encontró que el backend no configura lotes (`hibernate.jdbc.batch_size`), así que lo cierto es esto:
- **las lecturas no crecen con el lote:** el candado de las unidades es una sentencia y su lectura otra (`VehicleRepositoryImpl.findAllByPublicIdsForUpdate`), cualesquiera que sean las unidades;
- **cada escritura es su propia sentencia:** la actualización de cada unidad (`saveAllAndFlush`) y la inserción de cada fila de la bitácora.

`VehicleArrivalBatchIntegrationTests.theStatementsDoNotGrowPerUnit` fija con las estadísticas de Hibernate que las sentencias que no son inserciones ni actualizaciones son las mismas con 1, 10 y 80 unidades. Las 80 ingresan en menos de 2 s en la base de verificación (`eightyUnitsArriveInOneOperationWithTheirLocation`), la meta del plan, así que la revisión decidió no configurar lotes en todo el backend: corrige el texto, no el código.

**La bitácora de los hitos** (§13.1: se registra el cambio de valores). `MilestoneChangeLog` escribe una fila por campo que cambia —`arrival_date`, `registration_start_date`, `title_number`, `title_date`, `plates_received_date` y el primer registro de `plate`—, con la entidad `vehicles`, su `public_id`, su empresa y su marca (D-102), sin motivo: no son correcciones delicadas. La corrección de una placa ya registrada sigue siendo delicada y sigue en `VehicleIdentifierCorrections` (FLT-R03). La edición de una llave deja también una fila por campo, con la entidad `vehicle_keys`. Los documentos y los equipos no la necesitan: sus filas ya son su historia (§13.2).

**Los archivos.** `VehicleDocumentFiles` valida el tipo por el contenido y el peso, calcula el SHA-256 y guarda con `KeptFileStore.put(empresa, llave, bytes, tipo)` dentro de la transacción y después de validar todo; la llave es `vehicle-documents/{publicId de la unidad}/{código del tipo, en minúsculas}/{sha256}`. Los bytes van a Spaces, y a la base solo donde no hay bucket (D-189). Si la transacción se revierte, con `storage = base` la fila de `kept_files` se va con ella; en Spaces, `VehicleDocumentFiles` registra un `TransactionSynchronization` que llama a `KeptFileStore.discardRolledBack(llave)` en `afterCompletion(STATUS_ROLLED_BACK)`, el miembro que define la fase 2 (su §2.5.5, hallazgo E15). A diferencia de la llave de una factura, la de un documento es de su contenido y puede repetirse: el descarte se pide solo para una llave que esta transacción escribió por primera vez, la que `KeptFileStore.open` no encontraba antes de `put`. Si esta fase se integra primero, crea `discardRolledBack` con la forma de la fase 2.

**Como se construyó** (`fa9b7534`; `VehicleDocumentFiles.java:59`). La llave es `vehicle-documents/{publicId de la unidad}/{código del tipo en minúsculas}/{sha256}`, y `company_id` es la empresa de la unidad. A diferencia de una factura, cuya llave lleva el `public_id` nuevo de una factura que nunca se confirmó, la de un documento lleva el de una unidad que ya existe: por eso el descarte se limita a la llave que la transacción escribió primero, y una llave que ya apunta un documento confirmado nunca se descarta. Queda un caso: los mismos bytes del mismo tipo para la misma unidad, subidos a la vez, cuando uno se revierte y el otro confirma. Las dos subidas esperan el candado de la unidad, pero el descarte del que se revirtió corre después de soltarlo, así que puede borrar de Spaces el objeto que el otro acaba de volver a escribir con la misma llave. En la base no pasa: ahí la reversión ya quitó su fila. §2.7 del diccionario lo dice desde la v1.39, aprobada por el responsable el 04/11/2026 (documentación `7cc29f4`), con una sola regla para los archivos de las facturas, de las notas y de los documentos.

**Como se corrigió (04/11/2026)** (`7ad20a4a`). Para saber si escribía la llave por primera vez, `VehicleDocumentFiles` la abría antes de guardarla (`KeptFileStore.open`), que en Spaces es descargar el objeto entero solo para saber si existe. Ahora `KeptFileStore.put` devuelve si escribió la llave primero, en la misma sentencia que guarda su fila de `kept_files`: `INSERT … ON CONFLICT … DO UPDATE … RETURNING (xmax = 0)`, que es verdadero solo en una fila que esa sentencia insertó. El descarte se registra solo entonces, y el documento ya no descarga nada para decidirlo. Las facturas y las notas usan el mismo `put` (especificación de facturas, §2.5.5). El caso de los mismos bytes subidos a la vez, de arriba, queda como está. Lo prueban `VehicleDocumentFilesTests` (que además exige que nunca se llame a `open`), `KeptFileStoreUpsertTests.theDatabaseStoreTellsWhetherAPutWroteTheKeyFirst` y `theSpacesStoreTellsWhetherAPutWroteTheKeyFirst`, y `SpacesKeptFileStoreTests` (`documents`).

**Como se corrigió (04/11/2026, segunda revisión)** (backend `5df515b8`, `6e71a19d`). En Spaces, `KeptFileStore.put` escribía el objeto antes de su fila de `kept_files`; si la fila fallaba, el objeto quedaba en el bucket sin fila y sin descarte. Ahora guarda primero la fila y después el objeto: un `putObject` que falla hace fallar el `put`, y la fila se revierte con la transacción (especificación de facturas, §2.5.5). Pruebas: `SpacesKeptFileStoreTests.aRowThatFailsToWriteLeavesNoObject`, `theObjectIsWrittenAfterItsRowAndItsFailureFailsThePut`. El comentario de `VehicleDocumentFiles`, que todavía decía que descartaba la llave que `KeptFileStore.open` no encontraba, ahora dice la que el `put` insertó.

### 2.7 La fachada de `vehiclecatalog`

`VehicleCatalogContextFacade` (`vehiclecatalog/interfaces/acl/VehicleCatalogContextFacade.java`) suma estos miembros. Cada uno responde en una lectura por clase, nunca una por modelo.

| Miembro | Qué devuelve |
|---|---|
| `Map<Long, ModelEnablementProfile> fetchEnablementProfilesByModelIds(Set<Long> modelIds)` | Por modelo: `modelId`, `vehicleTypeId`, `vehicleTypeCode`, `fuelType` (o `null`), `runsOnGas` (`fuel_type` es `gnv` o `glp`; vacío no es gas, D-109), `hasFactoryScreen`, `requiredEquipments` y `applicableEquipments`. Los dos últimos son listas de `EquipmentTypeReference` |
| `Optional<EquipmentTypeReference> fetchEquipmentTypeReferenceByPublicId(UUID publicId)` | El equipo que se instala, activo o no |
| `Map<Long, EquipmentTypeReference> fetchEquipmentTypeReferencesByIds(Set<Long> ids)` | Los equipos de una lista, para nombrarlos |
| `void markEquipmentTypeInUse(Long equipmentTypeId)` | La única escritura nueva: `UPDATE equipment_types SET has_installations = TRUE WHERE id = :id AND NOT has_installations`, idempotente y en la transacción de quien llama, como `markModelInUse`; una vez marcado, ya no bloquea la fila (D-185) |
| `String fetchEnablementDataVersion()` | La versión de `vehicle_models`, `vehicle_types`, `equipment_types` y `equipment_type_vehicle_types`, para la exportación de la lista de habilitación |

`EquipmentTypeReference` es `(Long id, UUID publicId, String code, String name, boolean blocksAvailability, boolean requiresSerial, boolean isActive, boolean factoryFitted)`.

**La regla de la pantalla, escrita una sola vez** (§7.5). `requiredEquipments` son los equipos activos, con `blocks_availability` y aplicabilidad activa al tipo del modelo, menos `PANTALLA` cuando el modelo la trae de fábrica. `applicableEquipments` son todos los activos con aplicabilidad activa a ese tipo; en ellos, `factoryFitted` es verdadero solo para `PANTALLA` en un modelo con `has_factory_screen`. La regla vive en `vehiclecatalog`, dueño de los dos datos (skill `backend-module`, «La regla del dueño la decide el dueño»): `fleet` no compara códigos de equipo ni lee `has_factory_screen`. `VehicleModelSummary` no recupera los campos que salieron en `040ad1e` (`contextos/vehiclecatalog.md`, sección 5): el perfil es otro record.

`fleet` lo usa por `ExternalVehicleCatalogService` (`application/internal/outboundservices/acl`), que lo traduce a records propios (`ModelEnablementProfile`, `EquipmentTypeReference` de `fleet`) y, para la política, a `EnablementRequirement`.

**`EquipmentType`** (`vehiclecatalog/domain/model/aggregates`) suma `hasInstallations`; `isCodeLocked()` pasa a ser «el programa lo lee (`GPS`, `PANTALLA`) o tiene instalaciones» (D-74). `EquipmentTypeViewFactory` deja el comentario que anunciaba un conteo de `vehicle_equipments` en lote: el dato viaja en el agregado, y `vehiclecatalog` sigue sin leer tablas de `fleet`.

### 2.8 Lo que `fleet` usa de `organization` y `location`

| Contexto | Miembro | Para qué |
|---|---|---|
| `organization` | `fetchLocation` (existe, por `ExternalOrganizationService`) | La ubicación de destino del ingreso: su empresa, si está activa y su sede |
| | `fetchLocationsByIds` (existe) | Las ubicaciones actuales de un lote, que deciden la sede de cada escritura |
| | `fetchCompanyTimeZone` (existe) | «Hoy» en la zona de la empresa (§2.6), para las fechas y el filtro de vencimientos |
| | `fetchEmployeeAttentionByPublicId` y `fetchEmployeeAttentionsByIds` (existen) | Quién instaló un equipo: de la empresa de la unidad y activo al instalar; se muestra solo su nombre, nunca su teléfono (D-105) |
| | `fetchPurchasingCompaniesByIds` y su record `PurchasingCompany` (fase 2, su §2.5.2) | `functionalCurrencyId`: la moneda del costo cuando quien instala no ve los costos (D-187, FLT-R79). Si esta fase se integra primero, crea ese miembro con la forma que fija la fase 2 |
| `location` | `fetchCurrencySummaryByIsoCodeForShare` (existe) | La moneda del costo, leída en modo compartido antes de escribir el monto (contrato de `CurrencyUsage`) |
| | `fetchCurrencySummariesByIds` (existe) | El código y los decimales de las monedas de una lista |
| | `fetchCountrySummaryByIsoAlpha2` y `fetchCountrySummariesByIds` (existen) | El país de un tipo de documento, por su código (D-13) |
| | `fetchDataVersion()` (existe) | La versión de los países que nombra la exportación de los tipos de documento |

**`CurrencyUsage`.** `fleet/infrastructure/currencies/VehicleEquipmentCurrencyUsageAdapter` implementa el puerto de `location::acl`, como `PurchaseOrderCurrencyUsageAdapter` de compras: `anyAmountIn(currencyId)` responde si alguna instalación, de cualquier empresa, tiene un costo en esa moneda. Desde la primera instalación, los decimales de esa moneda quedan fijos (D-146, convenciones §7.2).

### 2.9 Lecturas

**Una unidad.** `VehicleEnablementFactsReader` (`application/internal/queryservices`) arma los `EnablementFacts` de una unidad con una lectura por clase: sus documentos vigentes, sus equipos abiertos, la primera fila del historial hacia `ACTIVO`, los tipos de documento de su país y el perfil de su modelo. Lo usan la puerta, `VehicleAllowedActions`, `GET /vehicles/{id}` y `GET /vehicles/{id}/enablement`.

**La lista.** Cumple el disparador (a) de la skill: columnas de varias tablas propias que el agregado no carga. Va en `VehicleEnablementReadRepository` (`application/internal/outboundservices/readmodels`), que devuelve `VehicleEnablementSummary` y lo implementa `JdbcVehicleEnablementReadRepository` en `infrastructure/persistence/jdbc`. Lo que exige el catálogo, que es de otro contexto, entra a la consulta como dato y no por un `JOIN` a sus tablas:
1. Lee los modelos distintos de las unidades que la lista alcanza, con sus filtros de empresa y marca.
2. Pide sus perfiles a `vehiclecatalog`, en una llamada.
3. Pasa a la consulta un parámetro `jsonb` con `model_id`, `runs_on_gas` y `required_equipment_type_ids` de cada modelo.

```sql
WITH requirements AS (
    SELECT r.model_id, r.runs_on_gas, r.required_equipment_type_ids
    FROM jsonb_to_recordset(CAST(:requirements AS jsonb))
         AS r (model_id BIGINT, runs_on_gas BOOLEAN, required_equipment_type_ids BIGINT[])
), units AS (
    SELECT v.* FROM vehicles v
    WHERE v.company_id = :companyId AND v.annulled_at IS NULL
      -- más el alcance por marca, la búsqueda y los filtros de la lista
), missing_documents AS (
    SELECT u.id AS vehicle_id, array_agg(t.id ORDER BY t.code) AS type_ids
    FROM units u
    JOIN requirements r ON r.model_id = u.model_id
    JOIN vehicle_document_types t
      ON t.country_id = u.country_id AND t.is_active AND t.blocks_availability
     AND (NOT t.requires_gas_conversion OR r.runs_on_gas)
    WHERE NOT EXISTS (SELECT 1 FROM vehicle_documents d
                      WHERE d.vehicle_id = u.id AND d.document_type_id = t.id AND d.is_current)
    GROUP BY u.id
), missing_equipments AS (
    SELECT u.id AS vehicle_id, array_agg(req.type_id ORDER BY req.type_id) AS type_ids
    FROM units u
    JOIN requirements r ON r.model_id = u.model_id
    CROSS JOIN LATERAL unnest(r.required_equipment_type_ids) AS req (type_id)
    WHERE NOT EXISTS (SELECT 1 FROM vehicle_equipments e
                      WHERE e.vehicle_id = u.id AND e.equipment_type_id = req.type_id AND upper_inf(e.period))
    GROUP BY u.id
)
SELECT u.id, u.public_id, …,
       CASE WHEN u.arrival_date IS NULL THEN 'FUERA_DE_TIENDA'
            WHEN md.vehicle_id IS NOT NULL THEN 'SIN_DOCUMENTO'
            WHEN me.vehicle_id IS NOT NULL THEN 'SIN_EQUIPO'
            WHEN u.plates_received_date IS NULL THEN 'SIN_PLACAS'
            WHEN EXISTS (SELECT 1 FROM vehicle_status_history h
                         WHERE h.vehicle_id = u.id AND h.to_status_id = :deliveryStatusId) THEN 'ENTREGADO'
            ELSE 'LISTO_PARA_ENTREGA' END AS enablement_status,
       md.type_ids AS missing_document_type_ids, me.type_ids AS missing_equipment_type_ids
FROM units u
LEFT JOIN missing_documents md ON md.vehicle_id = u.id
LEFT JOIN missing_equipments me ON me.vehicle_id = u.id
```

La consulta exterior filtra por `enablement_status`, ordena y pagina en la base. Los nombres de los documentos y de los equipos que faltan, del modelo, de la marca comercial y de la ubicación se resuelven por página, una lectura por clase.

**La misma regla en dos lugares, con su guarda.** La política y la consulta dicen lo mismo de dos maneras: así lo pide el plan (paso 8.5), porque la lista se filtra y se pagina en la base. `VehicleEnablementProjectionTests.theProjectionAnswersWhatThePolicyAnswers` recorre las combinaciones de las seis filas —con y sin ingreso, documento, cada equipo, placas y entrega, un modelo a gas y uno sin pantalla de fábrica— y exige que la consulta responda, para cada unidad, el mismo código y lo mismo que falta que `EnablementPolicy`. Un cambio en una que no se haga en la otra rompe esa prueba.

**Los vencimientos.** `ExpiringVehicleDocumentReadRepository` lista los documentos vigentes de la empresa con vencimiento en una ventana de días contada desde «hoy» en la zona de la empresa, o ya vencidos (sección 4.6).

**Las versiones de los datos** (D-128), por `FleetDataVersions`, que implementa `JdbcFleetDataVersions` sin nombrar tablas en `application`:
- `enablements()`: `vehicles`, `vehicle_statuses`, `vehicle_status_history`, `vehicle_document_types`, `vehicle_documents` y `vehicle_equipments`, más `fetchEnablementDataVersion()` de `vehiclecatalog` y `fetchDataVersion()` de `organization`;
- `documentTypes()`: `vehicle_document_types`, más `fetchDataVersion()` de `location`.

**Como se construyó** (`ba0ce3f5`, `FleetDocumentKinds`). La versión de `vehicle-enablements` suma también `fetchDataVersion()` de `vehiclecatalog`, la del catálogo general: los nombres del modelo y de la marca que escribe el archivo viven fuera de las tablas que nombra `fetchEnablementDataVersion()`.

**Los tipos de documento** (`FleetDocumentKinds`, `application/documents`):
- `vehicle-enablements`, con `export.vehicle-enablements.name`, el alcance de cada asignación de `stock.ver` (`DocumentScopes.grantsOf`) y la versión `enablements()`;
- `vehicle-document-types`, con `export.vehicle-document-types.name`, el alcance de un catálogo general (`DocumentScopes.holding(scope, catalogos.ver)`) y la versión `documentTypes()`.

### 2.10 Las clases nuevas, por capa

| Capa (`fleet`) | Clases |
|---|---|
| `domain/model/aggregates` | `VehicleDocumentType`, `VehicleDocument`, `VehicleEquipment`, `VehicleKey`; en `Vehicle`, los cinco hitos con `registerArrival`, `correctArrival`, `updateMilestones`, `registerPlate` y `transitionTo(Long, StatusPath, EntryGate)`; en `VehicleStatus`, `requiresEnablement` |
| `domain/model/valueobjects` | `EnablementStatusCode`, `EnablementRequirement`, `EnablementFacts`, `EnablementStatus`, `EntryGate`, `EquipmentPeriod`, `EquipmentRemovalReason` (`falla`, `robo`, `reemplazo`), `EquipmentCost`, `KeyType` (`entregada`, `duplicado`, `otra`), `KeyHolder` (`empresa`, `cliente`, `perdida`), `DocumentFile` |
| `domain/model/queries` | `GetVehicleEnablementQuery`, `SearchVehicleEnablementsQuery` con `VehicleEnablementSearchFilter`, `GetVehicleDocumentsQuery`, `SearchExpiringVehicleDocumentsQuery`, `GetVehicleDocumentByPublicIdQuery`, `GetVehicleDocumentFileQuery`, `GetVehicleEquipmentsQuery`, `GetVehicleEquipmentByPublicIdQuery`, `GetVehicleKeysQuery`, `GetVehicleKeyByPublicIdQuery`, `SearchVehicleDocumentTypesQuery`, `GetVehicleDocumentTypeByPublicIdQuery` |
| `domain/repositories` | `VehicleDocumentTypeRepository`, `VehicleDocumentRepository`, `VehicleEquipmentRepository`, `VehicleKeyRepository`; `VehicleRepository` suma `findAllByPublicIdsForUpdate(Set<UUID>)`; `VehicleStatusHistoryRepository` suma `findFirstMoveInto(Long vehicleId, Long statusId)` |
| `domain/services` | `EnablementPolicy`, `RequirementHoldPolicy` |
| `domain/exceptions` | `EnablementIncompleteException`, `EnablementGateLockedException`, `VehicleEquipmentOverlapException`, `VehicleKeyTagTakenException` |
| `application/commandservices` | `VehicleEnablementCommandService`, `VehicleDocumentTypeCommandService`, `VehicleDocumentCommandService`, `VehicleEquipmentCommandService`, `VehicleKeyCommandService` |
| `application/queryservices` | Las cinco `…QueryService` correspondientes y sus vistas: `VehicleEnablementView`, `VehicleEnablementRowView`, `VehicleEnablementAllowedActionsView`, `EnablementStatusView`, `VehicleDocumentView`, `ExpiringVehicleDocumentView`, `VehicleEquipmentView`, `VehicleKeyView`, `VehicleDocumentTypeView`, `VehicleEnablementExportRow` |
| `application/internal/commandservices` | Los cinco `…CommandServiceImpl`, `VehicleArrivals`, `MilestoneChangeLog`, `VehicleDocumentFiles`, `VehicleEquipmentInstallations` |
| `application/internal/queryservices` | Los `…QueryServiceImpl`, `VehicleEnablementFactsReader`, `VehicleEnablementViewFactory`, `VehicleEnablementAllowedActions`, `VehicleDocumentViewFactory`, `VehicleEquipmentViewFactory` (el costo, solo con `stock.ver_costos`), `VehicleKeyViewFactory`, `VehicleDocumentTypeViewFactory` |
| `application/internal/outboundservices` | `readmodels/VehicleEnablementReadRepository` y `ExpiringVehicleDocumentReadRepository`; `acl/ExternalLocationService` (nuevo); nuevos métodos en `ExternalVehicleCatalogService` y `ExternalOrganizationService`; `FleetDataVersions` con `enablements()` y `documentTypes()` |
| `infrastructure/persistence/jpa` | Entidad, ensamblador, repositorio de Spring Data y adaptador de cada tabla nueva. `VehicleDocumentTypePersistenceEntity` y `VehicleKeyPersistenceEntity` extienden `AuditableAbstractPersistenceEntity`; `VehicleEquipmentPersistenceEntity`, `CreatedAbstractPersistenceEntity`; `VehicleDocumentPersistenceEntity` no extiende ninguna: `uploaded_by` y `uploaded_at` son datos del hecho, que pone el servicio con la cuenta de la sesión |
| `infrastructure/persistence/jdbc` | `JdbcVehicleEnablementReadRepository`, `JdbcExpiringVehicleDocumentReadRepository`, `JdbcFleetDataVersions` con sus tres tablas versionadas nuevas |
| `infrastructure/currencies` | `VehicleEquipmentCurrencyUsageAdapter` |
| `interfaces/rest` | `VehicleEnablementsController` (`/vehicle-enablements`, `/vehicle-arrivals`, `/vehicles/{id}/enablement`, `/arrival`, `/milestones`, `/plate`), `VehicleDocumentTypesController`, `VehicleDocumentsController`, `VehicleEquipmentsController`, `VehicleKeysController`, con sus recursos y transformadores |

**Fuera de `fleet`.** En `vehiclecatalog`, los cinco miembros de 2.7 con `ModelEnablementProfile` y `EquipmentTypeReference` en `interfaces/acl`, y `MarkEquipmentTypeInUseCommand`. En `shared`, `UploadedFileType` (`shared/domain/model/valueobjects`), con `sniff(byte[])` y las constantes de tipo que hoy están en `PurchaseOrderTemplateDocument.sniffMediaType` (`procurement/domain/model/aggregates/PurchaseOrderTemplateDocument.java:31`). **Es el único punto en que las dos especificaciones difieren:** la fase 2 extrae `UploadedFileType` a `procurement/domain/model/valueobjects` (su §2.1), y `fleet` no puede importar `procurement` (`ModularityTests`). Esta especificación lo pone en `shared`, y la plantilla y la factura lo importan de ahí. Si la fase 2 se integra primero con la clase en `procurement`, el commit 12 la mueve a `shared` sin cambiar su contrato; si esta fase va primero, la fase 2 la encuentra en `shared`. En `shared/application/security/PermissionCodes`, el grupo nuevo `Enablement` (`REGISTER_MILESTONES`, `MANAGE_DOCUMENTS`, `MANAGE_EQUIPMENTS`, `MANAGE_KEYS`) y en `Catalogs` los tres de los tipos de documento (`CREATE_DOCUMENT_TYPES`, `EDIT_DOCUMENT_TYPES`, `DEACTIVATE_DOCUMENT_TYPES`).

**Como se construyó.** `UploadedFileType` ya estaba en `shared` cuando llegó esta fase, porque la fase 2 se integró antes: el commit 12 no hizo falta. Dos piezas de `application/internal/commandservices` que la tabla no nombra: `RequirementHolds`, que aplica `RequirementHoldPolicy` a una unidad ya bloqueada con su estado y el perfil de su modelo, y `VehiclePlateRegistrations`, el primer registro de la placa con su fila de la bitácora. Los ayudantes que bloquean la unidad y, después, su fila hija se repiten un poco entre `VehicleEnablementCommandServiceImpl` y `VehicleEquipmentCommandServiceImpl`: es una duplicación menor que queda anotada.

**Como se corrigió (04/11/2026).** La revisión sumó `domain/services/AutomaticReturnGate` y la constante `VehicleStatus.SOLD_CODE` (`345c1f75`, sección 2.3); `infrastructure/changelog/FleetConfidentialChangeLogFieldsAdapter`, que marca el costo y la moneda de una instalación para la bitácora (`ad87b4d9`, sección 4.7); y `StatusChangeReasonRepository.findAllActiveMovingAloneTo`. En `shared`, `ConfidentialChangeLogField` suma `byBrand`; en `documents`, `KeptFileStore.put` devuelve si escribió la llave primero (`7ad20a4a`).

---

## 3. Esquema

### 3.1 Numeración

Las fases 2 y 3 pueden integrarse en cualquier orden, así que las migraciones se numeran al integrar: la fase que se integra primero toma V92 en adelante, y la segunda sigue desde la última que tenga `develop` en ese momento. Aquí se nombran relativas, como en la especificación de la fase 2: V(n), V(n+1)… en el orden en que tienen que correr. Cada una abre con `SELECT set_config('app.access_scope', 'SYSTEM', true);` y `SET LOCAL lock_timeout = '5s';`, como V90, y una que cambia una tabla con datos se detiene diciéndolo antes que limpiar nada, como V84 y V89.

| Migración | Contexto | Qué hace |
|---|---|---|
| V(n) | `iam` | Los siete permisos y sus diecisiete marcas |
| V(n+1) | `fleet` | Las cinco columnas de §11.1 en `vehicles`; `requires_enablement` en `vehicle_statuses`, marcado en Disponible y en Vendido; el camino «Pierde un requisito» |
| V(n+2) | `fleet` | `vehicle_document_types`, con la semilla de Perú |
| V(n+3) | `fleet` | `vehicle_documents` |
| V(n+4) | `vehiclecatalog` | `equipment_types.has_installations` (D-185) |
| V(n+5) | `fleet` | `vehicle_equipments` |
| V(n+6) | `fleet` | `vehicle_keys` |

**Como se construyó.** La fase 2 se integró antes, con V92 a V94, así que n es 95: V95 `seed_iam_permissions_of_the_enablement` (`53521c1d`), V96 `add_the_enablement_milestones_and_the_gate_of_the_statuses` (`9ca62ea4`), V97 `create_fleet_vehicle_document_types_schema` (`3eb04534`), V98 `create_fleet_vehicle_documents_schema` (`df5633db`), V99 `record_whether_an_equipment_type_has_installations` (`eee48459`), V100 `create_fleet_vehicle_equipments_schema` (`d43951eb`) y V101 `create_fleet_vehicle_keys_schema` (`8de8ab80`).

### 3.2 V(n) — Los permisos (`iam`)

`V(n)__seed_iam_permissions_of_the_enablement.sql`, con el patrón de V90:

```sql
INSERT INTO permissions (code, module, description, category)
VALUES ('habilitacion.registrar_hitos', 'habilitacion', 'Ingreso físico, inmatriculación y el primer registro de la placa', 'editar'),
       ('habilitacion.gestionar_documentos', 'habilitacion', 'Cargar los documentos del vehículo', 'gestionar'),
       ('habilitacion.gestionar_equipos', 'habilitacion', 'Instalar y retirar equipamiento', 'gestionar'),
       ('habilitacion.gestionar_llaves', 'habilitacion', 'Registrar las llaves y dónde están', 'gestionar'),
       ('catalogos.crear_tipos_documento', 'catalogos', 'Registrar tipos de documento del vehículo', 'crear'),
       ('catalogos.editar_tipos_documento', 'catalogos', 'Corregir tipos de documento del vehículo; el código TIVE no se corrige', 'editar'),
       ('catalogos.desactivar_tipos_documento', 'catalogos', 'Activar y desactivar tipos de documento del vehículo', 'desactivar_anular')
ON CONFLICT (code) DO NOTHING;
```

Las descripciones son las de §5.6 sin la referencia entre paréntesis, como V79. Las marcas, firmadas por `SYSTEM`, siguen la matriz de §5.6: `registrar_hitos` y `gestionar_documentos` a SA, AE y OP; `gestionar_equipos` y `gestionar_llaves` a SA, AE, OP y TA; los tres de los tipos de documento, solo a SA. Son diecisiete. Los tres de catálogos son de un módulo `catalogos` que no es de la categoría `ver`, así que la regla de D-150 los deja solo en roles de alcance `SYSTEM` sin cambiar el código (V89, `ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`). Ningún rol propio los recibe: no reemplazan a ningún permiso grueso. No siembra `habilitacion.cargar_archivo_sat`, `habilitacion.ver_cargas` ni `fondos.cargar_archivo_garantia`, que son del paquete de cargas (hallazgo E16).

### 3.3 V(n+1) — Los hitos, la puerta y su salida (`fleet`)

```sql
ALTER TABLE vehicles
    ADD COLUMN arrival_date            DATE,
    ADD COLUMN registration_start_date DATE,
    ADD COLUMN title_number            VARCHAR(50),
    ADD COLUMN title_date              DATE,
    ADD COLUMN plates_received_date    DATE,
    -- El número de título no es único (§11.1), pero no se guarda vacío
    ADD CONSTRAINT vehicles_title_number_ck CHECK (title_number IS NULL OR length(btrim(title_number)) > 0),
    -- La fecha del título no existe sin título (§11.1)
    ADD CONSTRAINT vehicles_title_date_ck CHECK (title_date IS NULL OR title_number IS NOT NULL),
    -- Las placas físicas exigen la placa registrada (§11.1, D-165)
    ADD CONSTRAINT vehicles_plates_received_ck CHECK (plates_received_date IS NULL OR plate IS NOT NULL),
    -- La placa exige el título (§11.1); el documento de identificación lo revisa la aplicación
    ADD CONSTRAINT vehicles_plate_title_ck CHECK (plate IS NULL OR title_number IS NOT NULL),
    -- Una unidad ingresada tiene ubicación (D-165, D-188)
    ADD CONSTRAINT vehicles_arrived_location_ck CHECK (arrival_date IS NULL OR current_location_id IS NOT NULL);

ALTER TABLE vehicle_statuses
    ADD COLUMN requires_enablement BOOLEAN NOT NULL DEFAULT FALSE;
-- Disponible y Vendido exigen la habilitación completa (D-164, D-184)
UPDATE vehicle_statuses SET requires_enablement = TRUE WHERE code IN ('DISPONIBLE', 'VENDIDO');
-- En Disponible la marca no se quita (D-170); en Vendido, sí
ALTER TABLE vehicle_statuses
    ADD CONSTRAINT vehicle_statuses_enablement_gate_ck CHECK (code <> 'DISPONIBLE' OR requires_enablement);

-- La salida de una unidad Disponible que pierde un requisito sin repuesto (D-184)
INSERT INTO status_change_reasons (from_status_id, to_status_id, code, name, requires_permission_id)
SELECT origin.id, destination.id, 'PIERDE_REQUISITO', 'Pierde un requisito', permission.id
FROM vehicle_statuses origin
         JOIN vehicle_statuses destination ON destination.code = 'HABILITACION'
         JOIN permissions permission ON permission.code = 'stock.habilitar'
WHERE origin.code = 'DISPONIBLE'
ON CONFLICT ON CONSTRAINT status_change_reasons_path_code_uq DO NOTHING;
```

- **El camino «Pierde un requisito»** no tiene tolerancia ni hace nada al vencerse, y nombra `stock.habilitar` (D-100), que la semilla da a SA, AE y OP. Si Disponible, En habilitación o `stock.habilitar` no existen, la migración se detiene diciéndolo: la semilla de V46 y V47 los trae. Si una persona ya declaró ese código en ese camino, lo deja como está, como V51 con `REGRESO_AUTOMATICO`. `status_change_reasons` no está versionada: ningún documento la lee.
- **`vehicles_arrived_location_ck` no encuentra filas que la rompan**: `arrival_date` nace vacía en todas.
- **Antes de `vehicles_plate_title_ck`**, la migración cuenta las unidades con placa y sin título. Hoy no debería haber ninguna —el alta no recibe placa (FLT-R02) y la corrección exige una que ya exista—, pero una base de pruebas puede tenerla. Si hay, se detiene y nombra sus VIN: si les falta el título o si sobra la placa se decide, no se limpia.
- **Los hitos y la placa no tienen índice nuevo**: la lista filtra por empresa (`vehicles_company_idx`) y la placa ya es única en el país (`vehicles_country_plate_uq`, V47).
- **`vehicle_statuses` es un catálogo general** sin Row Level Security (V47). `vehicles` conserva sus políticas de V47.
- **`current_mileage` no se crea** (D-165): el kilometraje es la última lectura, como ya lo calcula `VehicleViewFactory` (FLT-R31).
- **La versión de los datos.** `vehicles` y `vehicle_statuses` ya están versionadas (V53 a V55): sus disparadores suben la versión con estas actualizaciones.

**Como se construyó: V96 sobre la base de desarrollo.** El 03/11/2026, antes de la migración, `erp_mm_db` estaba en V85 y ninguna unidad tenía placa, así que la revisión de V96 pasa; el próximo `spring-boot:run` aplica V86 a V101. Desde V96, una unidad que ya está en Disponible sin TIVE, sin GPS o sin placas físicas no se vende —Vendido tiene puerta— hasta que se carguen los datos de su habilitación, y puede salir por «Pierde un requisito». La carga inicial (cargas, segunda entrega) llena esos datos antes de la única corrida en producción.

**Como se corrigió (04/11/2026): el inventario antes de V96** (`7bc29d52`). V96 se detiene sin cambiar nada, nombrando los VIN, si alguna unidad ya tiene placa: la placa exige el título (`vehicles_plate_title_ck`), y el título es una columna nueva que ninguna unidad tiene todavía. También se detiene si faltan Disponible, En habilitación o `stock.habilitar`. La revisión notó que el estado de la base desplegada no se conocía, y `develop` se despliega en cada push. Por eso el backend trae `scripts/database/inventory-plated-units.sql` y la sección «Inventario antes de V96» de su README:
- es de solo lectura (`BEGIN READ ONLY` … `ROLLBACK`) y lee con el alcance `SYSTEM`, porque `vehicles` tiene Row Level Security forzada también para la cuenta dueña;
- dice las últimas migraciones de la base y si V96 ya corrió, y entonces el inventario sobra;
- lista las unidades con placa, con su empresa y su VIN: vacía, V96 corre; cada unidad que aparezca pide una decisión del responsable —si le falta el título o le sobra la placa—, no una limpieza;
- lista los dos estados y el permiso del camino «Pierde un requisito», que deben ser tres filas.

El responsable lo corre contra la base desplegada, con la cuenta dueña, antes de que `develop` despliegue V96. `EnablementGateMigrationTests.theInventoryBeforeV96ChecksWhatV96StopsOn` falla si las condiciones del inventario y las de V96 dejan de coincidir.

**Como se corrigió (04/11/2026, segunda revisión): lo que V96 no detiene** (backend `ea5f342c`). V96 marca Disponible y Vendido con `requires_enablement`, y no se detiene en dos casos que después la aplicación ya no acepta:
- **el estado inicial es Disponible o Vendido:** el alta deja cada unidad nueva en el estado inicial sin pasar la puerta, así que toda unidad nacería en un estado marcado (D-164), y desde la primera revisión un estado marcado no puede ser el inicial;
- **un camino abierto que mueve sola una unidad (`mover_sola`) a Disponible o a Vendido mientras espera en otro estado que Vendido:** una vez marcado su destino, el catálogo ya no lo guarda y el proceso deja donde está a la unidad incompleta (D-164, D-170).

El inventario lista ahora los dos, y la sección del README dice qué hacer: si sale el estado inicial, el responsable mueve la marca de inicial —a En habilitación, como la trae la semilla— antes del merge; cada camino listado se cambia o se cierra antes del merge. Si las listas de las unidades con placa, del estado inicial y de los caminos salen vacías y salen las tres filas, V96 corre y no deja nada por corregir. `EnablementGateMigrationTests` corre las dos consultas nuevas sobre datos que las disparan.

### 3.4 V(n+2) — `vehicle_document_types` (`fleet`)

```sql
CREATE TABLE vehicle_document_types
(
    id                      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id               UUID        NOT NULL DEFAULT uuidv7(),
    country_id              BIGINT      NOT NULL REFERENCES countries (id),
    code                    VARCHAR(30) NOT NULL,
    name                    VARCHAR(80) NOT NULL,
    has_expiry              BOOLEAN     NOT NULL,
    blocks_availability     BOOLEAN     NOT NULL,
    is_visible_to_customer  BOOLEAN     NOT NULL,
    requires_gas_conversion BOOLEAN     NOT NULL DEFAULT FALSE,
    is_active               BOOLEAN     NOT NULL DEFAULT TRUE,
    created_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at              TIMESTAMPTZ NOT NULL DEFAULT now(),

    CONSTRAINT vehicle_document_types_public_id_uq UNIQUE (public_id),
    -- El código no se repite dentro de su país (§11.2, D-139)
    CONSTRAINT vehicle_document_types_country_code_uq UNIQUE (country_id, code),
    CONSTRAINT vehicle_document_types_code_ck CHECK (code ~ '^[A-Z0-9_]{1,30}$'),
    CONSTRAINT vehicle_document_types_name_ck CHECK (length(btrim(name)) > 0)
);
CREATE INDEX vehicle_document_types_country_idx ON vehicle_document_types (country_id, is_active);

INSERT INTO vehicle_document_types (country_id, code, name, has_expiry, blocks_availability,
                                    is_visible_to_customer, requires_gas_conversion)
SELECT c.id, seed.code, seed.name, seed.has_expiry, seed.blocks, TRUE, seed.gas
FROM (VALUES ('TIVE', 'Tarjeta de identificación vehicular', FALSE, TRUE, FALSE),
             ('SOAT', 'Seguro obligatorio', TRUE, FALSE, FALSE),
             ('CERT_GAS', 'Certificado de conversión a gas', TRUE, FALSE, TRUE),
             ('REV_TECNICA', 'Revisión técnica', TRUE, FALSE, FALSE))
         AS seed (code, name, has_expiry, blocks, gas)
JOIN countries c ON c.iso_alpha_2 = 'PE';
```

- **Perú tiene que existir.** Si la semilla no inserta cuatro filas, la migración se detiene: Perú lo cargan las migraciones de `location`, y sin él la semilla quedaría a medias.
- **Es un catálogo general**, como los documentos tributarios (§11.2, §2.5): sin `company_id` ni Row Level Security, y con fechas sin autor (D-42, D-73, D-139).
- **Permisos de la cuenta de la aplicación:** `SELECT`, `INSERT` y `UPDATE`, sin `DELETE`.
- **Versión de los datos:** sus 64 casillas y sus dos disparadores (§2.7), para su exportación y la de la lista de habilitación.

### 3.5 V(n+3) — `vehicle_documents` (`fleet`)

```sql
CREATE TABLE vehicle_documents
(
    id               BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id        UUID         NOT NULL DEFAULT uuidv7(),
    vehicle_id       BIGINT       NOT NULL,
    company_id       BIGINT       NOT NULL REFERENCES companies (id),
    document_type_id BIGINT       NOT NULL REFERENCES vehicle_document_types (id),
    file_path        TEXT         NOT NULL,
    file_name        VARCHAR(255) NOT NULL,
    mime_type        VARCHAR(100) NOT NULL,
    file_size        BIGINT,
    issue_date       DATE,
    expiry_date      DATE,
    is_current       BOOLEAN      NOT NULL DEFAULT TRUE,
    uploaded_by      BIGINT       NOT NULL REFERENCES users (id),
    uploaded_at      TIMESTAMPTZ  NOT NULL DEFAULT now(),

    CONSTRAINT vehicle_documents_public_id_uq UNIQUE (public_id),
    -- La unidad y el documento son de la misma empresa (D-165), como las cesiones (V75)
    CONSTRAINT vehicle_documents_vehicle_company_fk FOREIGN KEY (vehicle_id, company_id)
        REFERENCES vehicles (id, company_id),
    -- PDF, PNG, JPEG o WebP, de a lo sumo 5 MB (D-174)
    CONSTRAINT vehicle_documents_mime_type_ck
        CHECK (mime_type IN ('application/pdf', 'image/png', 'image/jpeg', 'image/webp')),
    CONSTRAINT vehicle_documents_file_size_ck CHECK (file_size IS NULL OR (file_size > 0 AND file_size <= 5242880)),
    CONSTRAINT vehicle_documents_file_name_ck CHECK (length(btrim(file_name)) > 0),
    CONSTRAINT vehicle_documents_expiry_ck CHECK (expiry_date IS NULL OR issue_date IS NULL OR expiry_date > issue_date)
);

-- Un solo documento vigente de cada tipo por unidad (§11.3, D-28)
CREATE UNIQUE INDEX vehicle_documents_current_uq ON vehicle_documents (vehicle_id, document_type_id) WHERE is_current;
CREATE INDEX vehicle_documents_vehicle_idx ON vehicle_documents (vehicle_id, uploaded_at DESC);
CREATE INDEX vehicle_documents_company_idx ON vehicle_documents (company_id);
-- Los vencimientos de una empresa (D-167)
CREATE INDEX vehicle_documents_expiry_idx ON vehicle_documents (company_id, expiry_date)
    WHERE is_current AND expiry_date IS NOT NULL;
```

- **Row Level Security** forzada, con las tres políticas de V75 sobre `company_id` (`vehicle_documents_select`, `_insert`, `_update`) y ninguna para borrar.
- **Permisos de la cuenta de la aplicación:** `SELECT`, `INSERT` y **`UPDATE (is_current)`**, solo esa columna. Un documento no se edita (§11.3); lo único que cambia es que deja de ser el vigente. El adaptador lo hace con una sentencia propia (`UPDATE vehicle_documents SET is_current = FALSE WHERE id = :id`), no con un guardado de la entidad entera.
- **Versión de los datos:** sus 64 casillas y sus dos disparadores.
- **Lo que la base no protege**, y valida la aplicación:
  - que el tipo sea del país de la empresa de la unidad, porque la tabla no tiene `country_id`;
  - que el vencimiento exista si y solo si el tipo vence.

### 3.6 V(n+4) — `equipment_types.has_installations` (`vehiclecatalog`)

```sql
ALTER TABLE equipment_types ADD COLUMN has_installations BOOLEAN NOT NULL DEFAULT FALSE;
```

Corre antes de `vehicle_equipments`, así que no hay instalaciones que leer. Si al integrar el orden quedara al revés, la migración la llena con los tipos que ya están en `vehicle_equipments.equipment_type_id`, y lo dice, como V49 llenó `has_units` desde `vehicles` (VCT-R14). La columna es la que registra D-185 en §7.5.

### 3.7 V(n+5) — `vehicle_equipments` (`fleet`)

```sql
CREATE TABLE vehicle_equipments
(
    id                       BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id                UUID          NOT NULL DEFAULT uuidv7(),
    vehicle_id               BIGINT        NOT NULL,
    company_id               BIGINT        NOT NULL REFERENCES companies (id),
    equipment_type_id        BIGINT        NOT NULL REFERENCES equipment_types (id),
    serial                   VARCHAR(100),
    period                   DATERANGE     NOT NULL,
    removal_reason           VARCHAR(30),
    installed_by_employee_id BIGINT,
    cost                     NUMERIC(12, 2) NOT NULL DEFAULT 0,
    currency_id              BIGINT        NOT NULL REFERENCES currencies (id),
    created_at               TIMESTAMPTZ   NOT NULL DEFAULT now(),

    CONSTRAINT vehicle_equipments_public_id_uq UNIQUE (public_id),
    CONSTRAINT vehicle_equipments_vehicle_company_fk FOREIGN KEY (vehicle_id, company_id)
        REFERENCES vehicles (id, company_id),
    -- Quien instaló es de la empresa de la unidad (employees_id_company_uq, V44)
    CONSTRAINT vehicle_equipments_installer_company_fk FOREIGN KEY (installed_by_employee_id, company_id)
        REFERENCES employees (id, company_id),
    -- Un período empieza en un día conocido, lo incluye y excluye su fin (§2.6), como V75
    CONSTRAINT vehicle_equipments_period_ck
        CHECK (NOT isempty(period) AND NOT lower_inf(period) AND lower_inc(period) AND NOT upper_inc(period)),
    CONSTRAINT vehicle_equipments_removal_reason_ck
        CHECK (removal_reason IS NULL OR removal_reason IN ('falla', 'robo', 'reemplazo')),
    -- Abierto, sin motivo de retiro; cerrado, con su motivo
    CONSTRAINT vehicle_equipments_removal_ck CHECK (upper_inf(period) = (removal_reason IS NULL)),
    CONSTRAINT vehicle_equipments_serial_ck CHECK (serial IS NULL OR length(btrim(serial)) > 0),
    CONSTRAINT vehicle_equipments_cost_ck CHECK (cost >= 0),
    -- Dos equipos del mismo tipo nunca a la vez en la misma unidad, tampoco en el pasado (§11.4)
    CONSTRAINT vehicle_equipments_no_overlap_excl
        EXCLUDE USING gist (vehicle_id WITH =, equipment_type_id WITH =, period WITH &&)
);
CREATE INDEX vehicle_equipments_vehicle_idx ON vehicle_equipments (vehicle_id);
CREATE INDEX vehicle_equipments_company_idx ON vehicle_equipments (company_id);
CREATE INDEX vehicle_equipments_currency_idx ON vehicle_equipments (currency_id);
```

- **Row Level Security** forzada, con las tres políticas de V75 y ninguna para borrar.
- **Permisos de la cuenta de la aplicación:** `SELECT`, `INSERT` y `UPDATE (period, removal_reason, cost, currency_id)`. Una instalación cambia solo al cerrar su período (§11.4) y, por D-187, al corregir su costo; el serial, el tipo y quién instaló no se editan.
- **`btree_gist`** ya está creada (D-67, V75).
- **Versión de los datos:** sus 64 casillas y sus dos disparadores.
- **Lo que la base no protege**, y valida la aplicación bajo el candado de la unidad:
  - que nada se instale antes del ingreso;
  - que el equipo aplique al tipo de vehículo;
  - que el serial exista cuando el tipo lo exige;
  - que el costo tenga a lo sumo los decimales de su moneda.

### 3.8 V(n+6) — `vehicle_keys` (`fleet`)

```sql
CREATE TABLE vehicle_keys
(
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id     UUID        NOT NULL DEFAULT uuidv7(),
    vehicle_id    BIGINT      NOT NULL,
    company_id    BIGINT      NOT NULL REFERENCES companies (id),
    key_type      VARCHAR(20) NOT NULL,
    tag_code      VARCHAR(30) NOT NULL,
    serial        VARCHAR(50),
    storage_group VARCHAR(10),
    holder        VARCHAR(20) NOT NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT now(),

    CONSTRAINT vehicle_keys_public_id_uq UNIQUE (public_id),
    CONSTRAINT vehicle_keys_vehicle_company_fk FOREIGN KEY (vehicle_id, company_id) REFERENCES vehicles (id, company_id),
    -- La etiqueta no se repite dentro de la empresa (§11.5, D-29)
    CONSTRAINT vehicle_keys_company_tag_uq UNIQUE (company_id, tag_code),
    CONSTRAINT vehicle_keys_key_type_ck CHECK (key_type IN ('entregada', 'duplicado', 'otra')),
    CONSTRAINT vehicle_keys_holder_ck CHECK (holder IN ('empresa', 'cliente', 'perdida')),
    -- Un código propio: recortado y en mayúsculas, con sus guiones (D-86)
    CONSTRAINT vehicle_keys_tag_code_ck CHECK (length(btrim(tag_code)) > 0 AND tag_code = upper(btrim(tag_code))),
    CONSTRAINT vehicle_keys_storage_group_ck
        CHECK (storage_group IS NULL OR (length(btrim(storage_group)) > 0 AND storage_group = upper(btrim(storage_group)))),
    -- Solo las llaves que están en la empresa tienen caja (§11.5)
    CONSTRAINT vehicle_keys_storage_holder_ck CHECK (storage_group IS NULL OR holder = 'empresa'),
    CONSTRAINT vehicle_keys_serial_ck CHECK (serial IS NULL OR length(btrim(serial)) > 0)
);
CREATE INDEX vehicle_keys_vehicle_idx ON vehicle_keys (vehicle_id);
```

- **Row Level Security** forzada, con las tres políticas de V75 y ninguna para borrar.
- **Permisos de la cuenta de la aplicación:** `SELECT`, `INSERT` y `UPDATE`, sin `DELETE`.
- **Sin versión de los datos:** ninguna exportación lee las llaves.
- **Las listas cerradas** se guardan en minúsculas y sin tildes, como las demás del proyecto (`contextos/fleet.md`, sección 6 b, propuesta 5); la API las acepta en cualquier mayúscula.
- **El serial de fábrica** puede repetirse entre unidades y no tiene índice (§11.5).

### 3.9 Lo que comprueban las pruebas del esquema

- `RowLevelSecurityMigrationTests` (`shared`) exige la forma de V75 en `vehicle_documents`, `vehicle_equipments` y `vehicle_keys`: política por operación y ninguna para borrar.
- `FleetRowsAreNeverDeletedTests.theApplicationCannotDeleteARowOf` suma las cuatro tablas nuevas.
- `JdbcDataVersionsTests.everyVersionedTableHasItsSixtyFourSlotsAndItsTwoTriggers` cubre las tres versionadas.
- `PermissionCatalogConsistencyTests` exige la constante de cada permiso sembrado.

---

## 4. Contrato de la API

Toda la API bajo `/api/v1`; cuerpos en JSON con UTF-8, salvo la subida de un documento (multipart). Fechas en `AAAA-MM-DD`, momentos en ISO 8601 con `Z`. Los identificadores son `public_id`. Las listas siguen §6.2 de las convenciones: con `page`, el sobre `{items, page, size, totalItems, totalPages}`; sin `page`, el arreglo.

### 4.1 Rutas

| Método y ruta | Permiso | Éxito |
|---|---|---|
| `GET /vehicle-document-types` (lista y exportación) | `catalogos.ver` | `200`; `303` con el `Accept` de Excel |
| `POST /vehicle-document-types` | `catalogos.crear_tipos_documento` | `201` con `Location` |
| `GET /vehicle-document-types/{id}` | `catalogos.ver` | `200` |
| `PUT /vehicle-document-types/{id}` | `catalogos.editar_tipos_documento` | `200` |
| `PATCH /vehicle-document-types/{id}/status` | `catalogos.desactivar_tipos_documento` | `200` |
| `GET /vehicle-statuses`, `GET /vehicle-statuses/{id}` (cambian) | `catalogos.ver` | `200`, con `requiresEnablement` |
| `PUT /vehicle-statuses/{id}` (cambia) | `catalogos.editar_estados` | `200`, recibe `requiresEnablement` |
| `GET /vehicles/{id}` y las respuestas de las escrituras de la unidad (cambian) | `stock.ver` | `200`, con `enablement` y `allowedActions.heldReasonIds` |
| `PUT /vehicles/{id}` (cambia) | `stock.editar_vehiculo` | `200`; no deja sin ubicación una unidad ingresada (`422 VEHICLE_ARRIVED_WITHOUT_LOCATION`, D-188) |
| `POST /vehicles/{id}/status-changes` (cambia) | El del camino (D-100) | `201`; puede responder `422 VEHICLE_ENABLEMENT_INCOMPLETE` |
| `GET /vehicle-enablements` (lista y exportación) | `stock.ver` | `200`; `303` con el `Accept` de Excel |
| `GET /vehicles/{id}/enablement` | `stock.ver` | `200` |
| `POST /vehicle-arrivals` **(propuesta: el nombre de la ruta)** | `habilitacion.registrar_hitos` | `200` |
| `PUT /vehicles/{id}/arrival` | `habilitacion.registrar_hitos` | `200` |
| `PUT /vehicles/{id}/milestones` | `habilitacion.registrar_hitos` | `200` |
| `POST /vehicles/{id}/plate` | `habilitacion.registrar_hitos` | `200` |
| `GET /vehicles/{id}/documents` | `stock.ver` | `200` |
| `POST /vehicles/{id}/documents` (multipart) | `habilitacion.gestionar_documentos` | `201` con `Location: /api/v1/vehicle-documents/{id}` |
| `GET /vehicle-documents` **(propuesta: la colección de una empresa)** | `stock.ver` | `200` |
| `GET /vehicle-documents/{id}` | `stock.ver` | `200` |
| `GET /vehicle-documents/{id}/file` | `stock.ver` | `200`, los bytes |
| `GET /vehicles/{id}/equipments` | `stock.ver` | `200` |
| `POST /vehicles/{id}/equipments` | `habilitacion.gestionar_equipos` | `201` con `Location: /api/v1/vehicle-equipments/{id}` |
| `GET /vehicle-equipments/{id}` | `stock.ver` | `200` |
| `POST /vehicle-equipments/{id}/replacement` | `habilitacion.gestionar_equipos` | `201` con el `Location` del equipo nuevo |
| `POST /vehicle-equipments/{id}/removal` | `habilitacion.gestionar_equipos` | `200` |
| `PUT /vehicle-equipments/{id}/cost` (D-187) | `habilitacion.gestionar_equipos` y `stock.ver_costos` | `200` |
| `GET /vehicles/{id}/keys` | `stock.ver` | `200` |
| `POST /vehicles/{id}/keys` | `habilitacion.gestionar_llaves` | `201` con `Location: /api/v1/vehicle-keys/{id}` |
| `GET /vehicle-keys/{id}` | `stock.ver` | `200` |
| `PUT /vehicle-keys/{id}` | `habilitacion.gestionar_llaves` | `200` |

Son 26 rutas nuevas y 5 que cambian.

**Rutas de las convenciones** (§2.2): las de documentos, equipos y llaves son las que ya tienen. Las demás son propuestas que el paso 8.0 lleva a §2.2:
- `/vehicle-equipments/{id}/replacement`, que el diccionario pide sin nombrarla: «Reemplazar un equipo cierra el período del anterior y abre uno nuevo» (§11.4);
- `/vehicle-enablements`, `/vehicle-arrivals` y la colección `GET /vehicle-documents`;
- los subrecursos `/vehicles/{id}/enablement`, `/arrival`, `/milestones` y el `POST` de `/plate`.

El primer registro de la placa es un `POST` sobre `/vehicles/{id}/plate` y su corrección sigue siendo el `PUT` de la misma ruta, con `stock.corregir_placa` y motivo (convenciones §3.2): registrar primero y corregir después, como el `POST` y el `PUT` de una dirección (§3.3).

**Escribir exige leer.** Toda escritura sobre una unidad la busca primero con `stock.ver` donde se ve, marca incluida (D-102). Una unidad que no se ve responde `404 VEHICLE_NOT_FOUND` antes que cualquier `403`; una que se ve, sobre la que el permiso no actúa, `403 ACCESS_DENIED` (D-99). Un documento, un equipo o una llave cuya unidad no se ve responden `404` con su propio código.

### 4.2 Tipos de documento del vehículo

**El recurso** (`VehicleDocumentTypeResource`):

```json
{
  "id": "019a3f10-5b1c-7a3e-9d4f-0a1b2c3d4e01",
  "country": "PE",
  "countryName": "Perú",
  "code": "TIVE",
  "name": "Tarjeta de identificación vehicular",
  "hasExpiry": false,
  "blocksAvailability": true,
  "isVisibleToCustomer": true,
  "requiresGasConversion": false,
  "isActive": true,
  "codeLocked": true,
  "createdAt": "2026-11-03T14:05:00Z",
  "updatedAt": "2026-11-03T14:05:00Z"
}
```

`codeLocked` es un dato derivado (convenciones §4.2): verdadero si el código es `TIVE` o si un documento usa el tipo (D-74, §2.4).

**`POST /vehicle-document-types`**, con `{country, code, name, hasExpiry, blocksAvailability, isVisibleToCustomer, requiresGasConversion}`:

| Campo | Regla |
|---|---|
| `country` | Obligatorio; dos letras ISO, en cualquier mayúscula. Uno que no existe, `404 COUNTRY_NOT_FOUND` |
| `code` | Obligatorio; se guarda recortado y en mayúsculas; letras, números y guion bajo, hasta 30 |
| `name` | Obligatorio; hasta 80, recortado |
| `hasExpiry`, `blocksAvailability`, `isVisibleToCustomer` | Obligatorios |
| `requiresGasConversion` | Opcional; por omisión `false` |

Responde `201` con `Location: /api/v1/vehicle-document-types/{id}` y el recurso, y lo revisa en este orden:
1. El formato (`400 VALIDATION_ERROR`).
2. El país (`404`).
3. El código repetido en el país (`409 VEHICLEDOCUMENTTYPE_CONFLICT`), también en una carrera que la base detiene sobre `vehicle_document_types_country_code_uq`.

**`PUT /vehicle-document-types/{id}`**, con los mismos campos sin `country`, que no cambia: un tipo de otro país es otro tipo; si llega, se ignora. Cambiar el código de un tipo con `codeLocked` responde `422 VEHICLEDOCUMENTTYPE_CODE_LOCKED`; el mismo código no es una corrección y el resto pasa. Un código nuevo repetido en el país, `409`.

**`PATCH /vehicle-document-types/{id}/status`**, con `{isActive}`. Desactivar no borra nada: los documentos ya cargados lo siguen nombrando, y el tipo deja de pedirse y de bloquear (FLT-R72).

**La lista**, `GET /vehicle-document-types`:

| `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|
| `code`, `name` | `country` (código ISO; uno que no nombra ningún país no coincide con ningún tipo: es un filtro, no el dueño de la lista), `isActive` | `code`, `name`, `country`, `hasExpiry`, `blocksAvailability`, `isVisibleToCustomer`, `requiresGasConversion`, `isActive` | `country` y `code` |

**Como se construyó** (`7dd80e2b`). La lista se ordena y se pagina en memoria, después de filtrar en la base: el código ISO del país, por el que se ordena, es de `location`, y la tabla solo guarda `country_id`. El catálogo tiene pocas filas, así que no pesa.

### 4.3 Estados del vehículo

`VehicleStatusResource` suma `requiresEnablement` (`boolean`). `PUT /vehicle-statuses/{id}` suma `requiresEnablement`, obligatorio (`@NotNull`, como `generatesRevenue` e `isInitial`): un cliente que no lo manda recibe `400 VALIDATION_ERROR`. Es un cambio incompatible, que se hace en los dos repositorios a la vez (convenciones §1).

`POST /vehicle-statuses` no lo recibe: el estado nace sin la marca y se marca con `PUT` (D-170); si un cliente lo manda, se ignora. Dejar en `false` la marca de `DISPONIBLE` responde `422 VEHICLESTATUS_ENABLEMENT_LOCKED` y no guarda nada; en cualquier otro estado se marca o se desmarca, también en `VENDIDO`, que la semilla marca (D-184).

`GET /status-change-reasons` lista además el camino `PIERDE_REQUISITO` que trae la semilla, de Disponible a En habilitación, con `stock.habilitar`; se edita y se cierra como cualquier camino (FLT-R93).

**Como se corrigió (04/11/2026): tres rechazos nuevos** (`345c1f75`; sección 2.3). Los cuerpos no cambian.
- **`PUT /vehicle-statuses/{id}`** revisa, en este orden: el estado (`404 VEHICLESTATUS_NOT_FOUND`); `422 VEHICLESTATUS_ENABLEMENT_INITIAL` si el pedido trae a la vez `isInitial` y `requiresEnablement`, sin `context`, antes de `409 VEHICLESTATUS_INITIAL_CONFLICT`; ese conflicto; `422 VEHICLESTATUS_ENABLEMENT_AUTO_TARGET` si la marca pasa de falso a verdadero y algún camino abierto que espera en otro estado que Vendido mueve solas unidades a este, con sus códigos en `context.reasonCodes`, en el orden de sus identificadores; y al final, en el dominio, el formato (`400`) y `422 VEHICLESTATUS_ENABLEMENT_LOCKED`.
- **`POST /status-change-reasons`**, **`PUT /status-change-reasons/{id}`** con el camino abierto, y **`PATCH /status-change-reasons/{id}/status`** al reabrirlo responden `422 STATUSCHANGEREASON_AUTO_TARGET_GATED`, con `context.statusCode`, cuando `autoTransitionToStatusId` exige la habilitación y el camino no espera en `VENDIDO`. Va en la revisión del estado al que el camino mueve sola la unidad, después de `404 VEHICLESTATUS_NOT_FOUND` y de `422 STATUSCHANGEREASON_AUTO_TARGET_INACTIVE`. En el alta, va después de las revisiones del camino de alta y antes de `404 PERMISSION_NOT_FOUND` y de los dos `409`; en la edición, después de `STATUSCHANGEREASON_AUTOMATIC_LOCKED` y `STATUSCHANGEREASON_REGISTRATION_PERMISSION_NOT_ALLOWED`; en la reapertura, después de `STATUSCHANGEREASON_AUTOMATIC_LOCKED` y de las dos del camino de alta, al final.

Los dos controladores responden estos `context` por `EnablementErrorContextAssembler`, como los demás rechazos de la habilitación. Los textos están en §4.10.

**Como se corrigió (04/11/2026, segunda revisión): los candados de estas revisiones** (backend `5bcd796b`; sección 2.3). Las respuestas y su orden no cambian. El alta, la edición y la reapertura de un camino leen su destino automático `FOR SHARE` hasta guardarlo; `PUT /vehicle-statuses/{id}` y el cambio de actividad de un estado lo leen `FOR NO KEY UPDATE` antes de leer sus caminos. Dos pedidos a la vez ya no pasan cada uno su revisión: el segundo espera y responde con lo que confirmó el primero (`StatusCatalogLockTests`).

### 4.4 La unidad y su cambio de estado

`VehicleResource` suma `enablement`, solo en la ficha y en las respuestas de sus escrituras, como `allowedActions`. En las filas de `GET /vehicles` viaja en `null`.

```json
"enablement": {
  "code": "SIN_EQUIPO",
  "detail": [{ "code": "GPS", "name": "Dispositivo GPS" }],
  "arrived": true,
  "missingDocuments": [],
  "missingEquipments": [{ "id": "019a…", "code": "GPS", "name": "Dispositivo GPS" }],
  "platesReceived": false,
  "deliveredAt": null
}
```

`allowedActions` suma `heldReasonIds`, los caminos que la puerta retiene (sección 2.5), y `walkableReasonIds` deja de incluirlos. Lo que la fase 2 suma al mismo recurso —`annul` en `allowedActions` y `hasCurrentInvoice` en la unidad (su §4.2.10)— no cambia: las dos fases agregan campos distintos.

`PUT /vehicles/{id}` no cambia su cuerpo. Dejar `currentLocationId` en `null` en una unidad que ya ingresó responde `422 VEHICLE_ARRIVED_WITHOUT_LOCATION` y no guarda nada; la base lo repite con `vehicles_arrived_location_ck` (D-188). El diálogo de edición deja de ofrecer «Sin ubicación» a una unidad ingresada. Se revisa bajo el candado de la unidad, junto con `422 VEHICLE_MODEL_YEAR_LOCKED` de la fase 2, y después de su `422 VEHICLE_ANNULLED`.

`POST /vehicles/{id}/status-changes` no cambia su cuerpo. Suma `422 VEHICLE_ENABLEMENT_INCOMPLETE`, en el orden de la sección 2.3, con este `context`:

```json
{
  "code": "VEHICLE_ENABLEMENT_INCOMPLETE",
  "message": "La unidad no cumple la habilitación que exige ese estado",
  "details": "…",
  "context": {
    "statusCode": "DISPONIBLE",
    "enablementStatus": "SIN_DOCUMENTO",
    "arrived": true,
    "missingDocuments": [{ "code": "TIVE", "name": "Tarjeta de identificación vehicular" }],
    "missingEquipments": [{ "code": "GPS", "name": "Dispositivo GPS" }],
    "platesReceived": false
  }
}
```

### 4.5 La habilitación de una unidad

**`GET /vehicles/{id}/enablement`** responde `VehicleEnablementResource`:

```json
{
  "vehicleId": "019a…",
  "companyId": "019a…",
  "vin": "MD2A11CY6LWA12345",
  "plate": null,
  "vehicleBrandName": "BAJAJ",
  "modelName": "DISCOVER 125-ST",
  "vehicleTypeCode": "MOTO",
  "commercialBrandName": "Lima Bikes",
  "currentStatusId": "019a…",
  "currentStatusName": "En habilitación",
  "currentStatusCode": "HABILITACION",
  "currentStatusRequiresEnablement": false,
  "status": {
    "code": "SIN_EQUIPO",
    "detail": [{ "code": "GPS", "name": "Dispositivo GPS" }],
    "arrived": true,
    "missingDocuments": [],
    "missingEquipments": [{ "id": "019a…", "code": "GPS", "name": "Dispositivo GPS" }],
    "platesReceived": false,
    "deliveredAt": null
  },
  "milestones": {
    "arrivalDate": "2026-11-03",
    "currentLocationId": "019a…",
    "currentLocationName": "Patio central",
    "registrationStartDate": null,
    "titleNumber": null,
    "titleDate": null,
    "platesReceivedDate": null
  },
  "documentRequirements": [
    { "documentTypeId": "019a…", "code": "TIVE", "name": "Tarjeta de identificación vehicular",
      "hasExpiry": false, "blocksAvailability": true,
      "current": { "id": "019a…", "fileName": "tive.pdf", "issueDate": "2026-11-01", "expiryDate": null } },
    { "documentTypeId": "019a…", "code": "SOAT", "name": "Seguro obligatorio",
      "hasExpiry": true, "blocksAvailability": false, "current": null }
  ],
  "equipmentRequirements": [
    { "equipmentTypeId": "019a…", "code": "GPS", "name": "Dispositivo GPS", "blocksAvailability": true,
      "requiresSerial": true, "required": true, "factoryFitted": false, "installed": null }
  ],
  "updatedAt": "2026-11-03T14:05:00Z",
  "allowedActions": {
    "registerArrival": false, "correctArrival": true, "editMilestones": true, "registerPlate": true,
    "correctPlate": false, "uploadDocuments": true, "installEquipment": true, "registerKeys": true
  }
}
```

**Las listas de requisitos.**
- `documentRequirements`: los tipos activos del país de la unidad que le aplican —el certificado de gas solo si el modelo es a gas—, ordenados por código, cada uno con su documento vigente o `null`.
- `equipmentRequirements`: los equipos que aplican a su tipo de vehículo (`applicableEquipments` de 2.7), cada uno con su instalación abierta (`{id, serial, startDate}`) o `null`. `required` dice si bloquea a esta unidad, y `factoryFitted` marca la pantalla que el modelo trae de fábrica («De fábrica», guía, columna 21).
- `updatedAt` es el de la unidad: el candado de `PUT /arrival` y `PUT /milestones` (sección 4.11).

**Como se corrigió (04/11/2026): el estado actual viaja con la habilitación** (backend `4aa250b9`; frontend `ab67dce4`). La página de la habilitación se abre con `stock.ver`, pero leía el código y la marca de su estado de `GET /vehicle-statuses`, que exige `catalogos.ver`: a un rol sin ese permiso la nota de la puerta y la del equipo retenido no le salían. `VehicleEnablementResource` suma, después de `currentStatusName`, `currentStatusCode` y `currentStatusRequiresEnablement`, y la fila de la lista (`VehicleEnablementRowResource`, sección 4.9), que ya traía el código, suma `currentStatusRequiresEnablement` después de `currentStatusName`. La pantalla ya no lee el catálogo de estados (sección 6.4).

**`POST /vehicle-arrivals`**, con `{vehicleIds, arrivalDate, locationId}`:

| Campo | Regla |
|---|---|
| `vehicleIds` | De 1 a 100 identificadores públicos distintos (`400` fuera de eso) |
| `arrivalDate` | Obligatoria; no futura en la zona de la empresa (D-186) |
| `locationId` | Obligatoria: el ingreso fija la ubicación (D-165) |

Responde `200` con `{"arrivedCount": 80, "vehicleIds": ["019a…", …]}`. Las revisiones, en orden, son las de la sección 2.6. No responde `201`: no crea un recurso que se pueda leer (convenciones §3.3, la mudanza). Escribe todo o nada.

**`PUT /vehicles/{id}/arrival`**, con `{arrivalDate, locationId, updatedAt}`, registra o corrige el ingreso de una unidad:

| Caso | Qué hace |
|---|---|
| `arrivalDate` con fecha y `locationId` con ubicación | Registra el ingreso o lo corrige, y fija la ubicación, como el lote. Una fecha posterior al inicio de un equipo de la unidad responde `422 VEHICLE_ARRIVAL_AFTER_INSTALLATION`, con ese inicio en `context.earliestInstallation` |
| `arrivalDate` en `null` | Vuelve a «no ingresada», solo para corregir un error (§11.1). La ubicación queda como está. Si la unidad tiene alguna instalación, abierta o cerrada, responde `422 VEHICLE_ARRIVAL_HAS_INSTALLATIONS`; en un estado con puerta, `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS` (D-184) |
| `arrivalDate` con fecha y `locationId` en `null` | `400 VALIDATION_ERROR` en `locationId` |

Responde `200` con `VehicleEnablementResource`. **Como se construyó** (`4a48376a`): con `arrivalDate` en `null`, la revisión del estado con puerta va antes que la de las instalaciones, así que una unidad en Disponible con equipos responde `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS` y no `VEHICLE_ARRIVAL_HAS_INSTALLATIONS`.

**`PUT /vehicles/{id}/milestones`**, con `{updatedAt, registrationStartDate, titleNumber, titleDate, platesReceivedDate}`. Reemplaza los cuatro: uno en `null` vacía el hito. Responde `200` con `VehicleEnablementResource`.

| Campo | Regla |
|---|---|
| `titleNumber` | Hasta 50, recortado; vacío o solo espacios es `null`. No es único (§11.1). No se vacía si la unidad tiene placa (`422 VEHICLE_TITLE_REQUIRED_BY_PLATE`) |
| `titleDate` | Solo con título (`422 VEHICLE_TITLE_DATE_WITHOUT_TITLE`) |
| `platesReceivedDate` | Solo con placa (`422 VEHICLE_PLATES_RECEIVED_WITHOUT_PLATE`). No se vacía en un estado con puerta (`422 VEHICLE_ENABLEMENT_HELD_BY_STATUS`, D-184) |
| Las tres fechas | No futuras en la zona de la empresa (`422 VEHICLE_MILESTONE_IN_FUTURE`, con el campo en `context.field`; D-186) |

**`POST /vehicles/{id}/plate`**, con `{plate}`, registra la primera placa. Responde `200` con `VehicleEnablementResource`. Revisa en este orden:
1. El formato (`400`): de 3 a 10 letras o números una vez normalizada, sin guiones, espacios ni puntos (convenciones §5.3, FLT-R02).
2. Que la unidad no tenga placa (`422 VEHICLE_PLATE_ALREADY_REGISTERED`: se corrige con el `PUT`).
3. Que tenga título (`422 VEHICLE_PLATE_TITLE_MISSING`).
4. Que tenga vigente el documento `TIVE` de su país (`422 VEHICLE_PLATE_IDENTIFICATION_DOCUMENT_MISSING`). En un país sin el tipo `TIVE` la placa no se registra hasta decidirlo con su primera empresa, como sus documentos tributarios (D-188, D-52).
5. Que la placa no la tenga otra unidad del país (`409 VEHICLE_PLATE_CONFLICT`), también en una carrera que la base detiene.

### 4.6 Documentos

**El recurso** (`VehicleDocumentResource`):

```json
{
  "id": "019a…",
  "vehicleId": "019a…",
  "documentTypeId": "019a…",
  "documentTypeCode": "SOAT",
  "documentTypeName": "Seguro obligatorio",
  "fileName": "soat_abc123_2026.pdf",
  "mimeType": "application/pdf",
  "fileSize": 182344,
  "issueDate": "2026-03-01",
  "expiryDate": "2027-02-28",
  "isCurrent": true,
  "uploadedBy": { "id": "019a…", "name": "Ana Pérez Rojas" },
  "uploadedAt": "2026-03-01T15:20:00Z"
}
```

Nunca viaja `file_path` (convenciones §8).

**`POST /vehicles/{id}/documents`**, en `multipart/form-data`, con la parte `file`, el archivo, y la parte `data`, de tipo `application/json`, con `{documentTypeId, issueDate, expiryDate}`. Revisa en este orden:
1. El formato (`400`): `documentTypeId` obligatorio, el archivo presente y no vacío.
2. La unidad y su permiso (`404`, `403`), y que no esté anulada (`422 VEHICLE_ANNULLED`, fase 2).
3. El tipo (`404 VEHICLEDOCUMENTTYPE_NOT_FOUND`); de otro país que la unidad (`422 VEHICLEDOCUMENT_TYPE_OTHER_COUNTRY`); inactivo (`422 VEHICLEDOCUMENTTYPE_INACTIVE`, D-108); el certificado de gas en una unidad que no es a gas (`422 VEHICLEDOCUMENT_TYPE_NOT_APPLICABLE`, D-166).
4. Las fechas: el vencimiento falta y el tipo vence (`422 VEHICLEDOCUMENT_EXPIRY_REQUIRED`); viene y el tipo no vence (`422 VEHICLEDOCUMENT_EXPIRY_NOT_ALLOWED`); no es posterior a la emisión (`422 VEHICLEDOCUMENT_EXPIRY_NOT_AFTER_ISSUE`); la emisión es futura (`422 VEHICLEDOCUMENT_ISSUE_DATE_IN_FUTURE`, D-186).
5. El archivo: el tipo leído de sus bytes no es PDF, PNG, JPEG ni WebP (`422 VEHICLEDOCUMENT_FILE_TYPE`); pesa más de 5 MB (`422 VEHICLEDOCUMENT_FILE_SIZE`). Más de 10 MB, o un pedido de más de 21 MB, lo corta el servidor con `413 CONTENT_TOO_LARGE` antes del servicio (convenciones §7.2).

Después retira el vigente de ese tipo (`is_current = false`), guarda el archivo y el documento nuevo, vigente, en la misma transacción, y responde `201` con `Location: /api/v1/vehicle-documents/{id}` y el recurso. `fileName` se guarda como llegó, recortado, hasta 255.

**Como se construyó** (`fa9b7534`):
- Sin la parte `data`, el pedido no llega al servicio: responde `400` con el código del marco, `BAD_REQUEST`, como la factura sin su parte `invoice` (convenciones §13, fila 18). Sin la parte `file`, o con un archivo vacío, responde `400 VALIDATION_ERROR` en `file`.
- Un archivo que llega sin nombre se guarda con el código del tipo en minúsculas y la extensión que dice su contenido: `tive.pdf`.

**`GET /vehicles/{id}/documents`** lista los de la unidad, vigentes y anteriores, con `page` opcional. Filtros `documentTypeId` e `isCurrent`. No busca ni ordena: descarta `q` y `sort`, como el historial (convenciones §6.2), y responde del más nuevo al más antiguo por `uploadedAt`.

**`GET /vehicle-documents/{id}/file`** entrega los bytes con su `Content-Type` y `Content-Disposition: inline; filename="…"`. Los cuatro tipos admitidos se ven en el navegador, como los archivos de la plantilla (convenciones §3.4). **Como se construyó:** un nombre que no es ASCII viaja también como `filename*=UTF-8''…`, y las comillas, las barras y los saltos de línea del nombre se cambian por un guion.

**`GET /vehicle-documents`**, los vencimientos de una empresa (D-167): los documentos vigentes con vencimiento, de las unidades no anuladas que se ven con `stock.ver`.

| Parámetro | Regla |
|---|---|
| `companyId` | Obligatorio (`400` sin él). Una empresa fuera del alcance no coincide con nada, como el filtro del Inventario (FLT-R33) |
| `expiresWithinDays` | Entero de 1 a 366: los que vencen en `[hoy, hoy + n)`, con «hoy» en la zona de la empresa (§2.6). La pantalla manda 30 |
| `expired` | `true`: los que vencieron (`expiry_date < hoy`). Con `expiresWithinDays`, `400` |
| `documentTypeId` | Filtro exacto |
| `q` | Busca en el VIN y en la placa, sin distinguir tildes ni mayúsculas (`fold_search_text`, D-142) |
| `sort` | `expiryDate`, `vin`, `plate`, `documentTypeCode`; orden propio `expiryDate` ascendente |

Cada fila (`ExpiringVehicleDocumentResource`) es `{id, vehicleId, vin, plate, vehicleBrandName, modelName, currentLocationName, documentTypeId, documentTypeCode, documentTypeName, issueDate, expiryDate}`.

### 4.7 Equipamiento

**El recurso** (`VehicleEquipmentResource`):

```json
{
  "id": "019a…",
  "vehicleId": "019a…",
  "equipmentTypeId": "019a…",
  "equipmentTypeCode": "GPS",
  "equipmentTypeName": "Dispositivo GPS",
  "serial": "8712-BC55",
  "startDate": "2026-07-04",
  "endDate": null,
  "removalReason": null,
  "installedByEmployeeId": "019a…",
  "installedByName": "Luis Torres Díaz",
  "cost": { "amount": 350.00, "currency": "PEN" },
  "createdAt": "2026-07-04T16:00:00Z",
  "allowedActions": ["replace", "remove", "correctCost"]
}
```

- **El período** viaja como `startDate` y `endDate`, con el fin excluido y `null` si sigue abierto, como la cesión a un fondo (`VehicleFundAssignmentResource`).
- **El costo** viaja como `{amount, currency}`, con el importe como número, como `MoneyResource` de compras. **Se omite**, no viaja en `null`, cuando quien lee no tiene `stock.ver_costos` en la empresa de la unidad (§2.5, hallazgo M8): es la segunda excepción con nombre a la regla de `null` (convenciones §4.1), y el paso 8.0 la registra junto a la de los colaboradores.

**Como se corrigió (04/11/2026): el costo es de la marca de la unidad** (backend `f5eacfe7`, `ad87b4d9`; frontend `ab67dce4`, `c510e131`). El costo de una instalación es un costo de la unidad, así que se lee y se escribe como su costo de compra (D-102, D-187, D-203):
- **el recurso, el alta, el reemplazo y `PUT /vehicle-equipments/{id}/cost`** piden `stock.ver_costos` en la empresa y en la marca de la unidad, por `VehicleAccessPolicy.seesCosts`; antes bastaba la empresa, y una asignación de la empresa limitada a otra marca leía y mandaba el costo. Sin él, el costo no viaja, `correctCost` no se ofrece y un costo enviado responde `403 ACCESS_DENIED` («`stock.ver_costos` does not act on the company and the brand of the unit»);
- **la bitácora retiene sus valores**: los campos `cost` y `currency_id` de `vehicle_equipments` son confidenciales por marca (`FleetConfidentialChangeLogFieldsAdapter`, `ConfidentialChangeLogField` con `byBrand`). Quien lee la bitácora sin `stock.ver_costos` en la empresa y la marca de la fila ve que el costo cambió, sin el valor anterior ni el nuevo, y la fila trae `valuesWithheld: true`; la pantalla dice por qué: «Valor reservado: requiere ver los costos de la unidad.». Antes los veía cualquiera con `bitacora.ver`. §13.1 del diccionario lo dice desde la v1.39, aprobada por el responsable el 04/11/2026 (documentación `7cc29f4`).

**`POST /vehicles/{id}/equipments`**, con `{equipmentTypeId, serial, startDate, installedByEmployeeId, cost}`:

| Campo | Regla |
|---|---|
| `equipmentTypeId` | Obligatorio. Uno que no existe, `404 EQUIPMENTTYPE_NOT_FOUND`; inactivo, `422 EQUIPMENTTYPE_INACTIVE`; que no aplica al tipo de vehículo de la unidad, `422 VEHICLEEQUIPMENT_TYPE_NOT_APPLICABLE` |
| `serial` | Hasta 100, recortado; obligatorio si el tipo lo exige (`422 VEHICLEEQUIPMENT_SERIAL_REQUIRED`) |
| `startDate` | Obligatoria. La unidad tiene que haber ingresado (`422 VEHICLEEQUIPMENT_NOT_ARRIVED`) y la fecha no puede ser anterior al ingreso (`422 VEHICLEEQUIPMENT_BEFORE_ARRIVAL`, con `context.arrivalDate`) ni futura (`422 VEHICLEEQUIPMENT_DATE_IN_FUTURE`, D-186) |
| `installedByEmployeeId` | Opcional. Un colaborador que no se ve, `404 EMPLOYEE_NOT_FOUND`; de otra empresa, `422 VEHICLEEQUIPMENT_INSTALLER_OTHER_COMPANY`; uno que salió, `422 EMPLOYEE_INACTIVE` |
| `cost` | Opcional (D-187). Si viene: `amount` de 0 a 9 999 999 999,99, con a lo sumo los decimales de su moneda (`422 VEHICLEEQUIPMENT_COST_SCALE`); `currency` de tres letras, en cualquier mayúscula (`404 CURRENCY_NOT_FOUND`, `422 CURRENCY_INACTIVE`); quien lo manda tiene `stock.ver_costos` en la empresa de la unidad (`403 ACCESS_DENIED`). Si no viene, el costo es 0 en la moneda funcional de la empresa |

Revisa además que no haya un equipo abierto del mismo tipo, ni un período que se cruce (`409 VEHICLEEQUIPMENT_CONFLICT`): primero la aplicación y, en una carrera, la exclusión de la base (`23P01`), que el adaptador traduce al mismo código, como las cesiones. Marca el tipo como usado (`markEquipmentTypeInUse`) en la misma transacción. Responde `201` con el recurso.

**`POST /vehicle-equipments/{id}/replacement`**, con `{serial, startDate, installedByEmployeeId, cost}`. Cierra el período del equipo en `startDate`, con motivo `reemplazo`, y abre uno nuevo del mismo tipo desde esa fecha, en la misma transacción (§11.4). Revisa:
- el equipo (`404 VEHICLEEQUIPMENT_NOT_FOUND`) y que esté abierto (`422 VEHICLEEQUIPMENT_ALREADY_REMOVED`);
- que `startDate` sea posterior al inicio del que se reemplaza (`422 VEHICLEEQUIPMENT_END_NOT_AFTER_START`);
- y las reglas del alta.

Se permite en un estado con puerta, porque la unidad nunca queda sin el equipo. Responde `201` con `Location` y el equipo nuevo.

**`POST /vehicle-equipments/{id}/removal`**, con `{endDate, removalReason}`:
- `removalReason` es `falla` o `robo`, en cualquier mayúscula; `reemplazo` lo escribe solo el reemplazo (`400` con cualquier otro);
- `endDate` es posterior al inicio (`422 VEHICLEEQUIPMENT_END_NOT_AFTER_START`) y no futura (D-186);
- un equipo ya cerrado responde `422 VEHICLEEQUIPMENT_ALREADY_REMOVED`;
- en un estado con puerta, un equipo que la unidad exige responde `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS` (D-184).

Responde `200` con el equipo cerrado. **Como se construyó** (`5aefa129`, `4a48376a`), las revisiones van en este orden: el motivo (`400`), el equipo y su unidad (`404`), el permiso (`403`), la unidad anulada (`422 VEHICLE_ANNULLED`), `VEHICLEEQUIPMENT_ALREADY_REMOVED`, `VEHICLEEQUIPMENT_END_NOT_AFTER_START`, `VEHICLEEQUIPMENT_DATE_IN_FUTURE` y, al final, `VEHICLE_ENABLEMENT_HELD_BY_STATUS`.

**`PUT /vehicle-equipments/{id}/cost`** (D-187), con `{cost: {amount, currency}}`. Registra o corrige el costo de una instalación, abierta o cerrada:
- pide `habilitacion.gestionar_equipos` sobre la unidad y `stock.ver_costos` en su empresa (`403`);
- revisa el importe, la moneda y su escala como el alta;
- deja una fila por campo que cambia en la bitácora (`cost`, `currency_id`), sin motivo, con la marca de la unidad.

Responde `200` con el equipo. **Como se construyó:** los campos de la bitácora se llaman `cost` y `currency_id`, y en `currency_id` el valor anterior y el nuevo son los códigos ISO de las monedas, no sus identificadores.

**`GET /vehicles/{id}/equipments`**, con `page` opcional y el filtro `isInstalled`. Descarta `q` y `sort`: los abiertos primero, después del inicio más nuevo al más viejo.

### 4.8 Llaves

**El recurso** (`VehicleKeyResource`): `{id, vehicleId, keyType, tagCode, serial, storageGroup, holder, createdAt, updatedAt, allowedActions}`, con `allowedActions` `["edit"]` o `[]`.

**`POST /vehicles/{id}/keys`**, con `{keyType, tagCode, serial, storageGroup, holder}`:

| Campo | Regla |
|---|---|
| `keyType` | `entregada`, `duplicado` u `otra`, en cualquier mayúscula; otro, `400` |
| `tagCode` | Obligatorio, hasta 30; se guarda recortado y en mayúsculas, con sus guiones (D-86): `LL-0450` |
| `serial` | Opcional, hasta 50; puede repetirse entre unidades (§11.5) |
| `storageGroup` | Opcional, hasta 10, recortado y en mayúsculas: `G01`. Solo con `holder` `empresa` (`422 VEHICLEKEY_STORAGE_GROUP_NOT_ALLOWED`) |
| `holder` | `empresa`, `cliente` o `perdida`, en cualquier mayúscula; otro, `400` |

Una etiqueta que ya tiene otra llave de la empresa responde `409 VEHICLEKEY_CONFLICT`, también en una carrera que la base detiene sobre `vehicle_keys_company_tag_uq`. Responde `201` con la llave.

**`PUT /vehicle-keys/{id}`**, con los mismos campos y `updatedAt`. Un `updatedAt` que no es el de la fila responde `422 VEHICLEKEY_OUTDATED` (sección 4.11). Responde `200` con la llave.

**`GET /vehicles/{id}/keys`**, sin página ni búsqueda: una unidad tiene pocas llaves. Las ordena por etiqueta.

### 4.9 La lista de habilitación

**`GET /vehicle-enablements`**: las unidades no anuladas de una empresa con su estado de habilitación, las que se ven con `stock.ver`, marca incluida (D-99, D-102).

| `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|
| `vin`, `plate`, `engineNumber`, plegados (D-142) | `companyId` (obligatorio, `400` sin él; una empresa fuera del alcance da una lista vacía); `enablementStatus`, repetible (`?enablementStatus=SIN_EQUIPO&enablementStatus=SIN_PLACAS`; un código fuera de los seis, `400`); `currentStatusId`; `currentLocationId`; `commercialBrandId`; `modelId`; `vehicleIds`, repetible, de 1 a 100 (el diálogo de ingreso de una factura, sección 6.6) | `enablementStatus` (por el orden de las filas), `vin`, `plate`, `arrivalDate`, `nearestExpiryDate` | `enablementStatus` y después `vin` |

Cada fila (`VehicleEnablementRowResource`):

```json
{
  "vehicleId": "019a…",
  "vin": "MD2A11CY6LWA12345",
  "plate": null,
  "vehicleBrandName": "BAJAJ",
  "modelName": "DISCOVER 125-ST",
  "vehicleTypeCode": "MOTO",
  "commercialBrandName": "Lima Bikes",
  "currentStatusId": "019a…",
  "currentStatusCode": "HABILITACION",
  "currentStatusName": "En habilitación",
  "currentStatusRequiresEnablement": false,
  "currentLocationId": "019a…",
  "currentLocationName": "Patio central",
  "arrivalDate": "2026-11-03",
  "platesReceivedDate": null,
  "enablementStatus": "SIN_EQUIPO",
  "detail": [{ "code": "GPS", "name": "Dispositivo GPS" }],
  "nearestExpiryDate": "2027-02-28",
  "nearestExpiryDocumentTypeName": "Seguro obligatorio"
}
```

`nearestExpiryDate` es el vencimiento más temprano de los documentos vigentes de la unidad, vencido o no; no depende del reloj, así que la exportación se reutiliza por la versión de los datos y no por la hora.

**La exportación.** El mismo `GET` con el `Accept` de Excel, con la búsqueda, los filtros y el orden de la pantalla y todas las filas que coinciden (convenciones §8.1). `kind` `vehicle-enablements`, archivo `habilitacion-<fecha>.xlsx`, columnas:

| Columna | Valor |
|---|---|
| Placa | Texto, vacío sin placa |
| VIN | Texto |
| Marca, Modelo | Del catálogo |
| Código de tipo de vehículo, Tipo de vehículo | El código y el nombre |
| Marca comercial | Su nombre |
| Código de ubicación, Ubicación | El código y el nombre, porque los nombres se repiten (convenciones §8.1) |
| Código de estado, Estado | Del ciclo de vida |
| Código de habilitación | `FUERA_DE_TIENDA` … `ENTREGADO` (D-171) |
| Habilitación | El nombre traducido: Fuera de tienda, Sin documento vehicular, Sin equipo, Sin placas, Listo para entrega, Entregado |
| Falta | Los nombres del detalle, separados por coma |
| Ingreso, Inicio de inmatriculación, Fecha de título, Placas recibidas | Celdas de fecha `AAAA-MM-DD` |
| N.° de título | Texto |
| Vencimiento más próximo | Fecha, y en otra columna el documento que vence |

No tiene momentos, así que no nombra zona. El alcance y la versión son los de la sección 2.9.

### 4.10 Códigos de error

Todos siguen §7.2 de las convenciones: el estado sale del sufijo (`_NOT_FOUND` `404`, `_CONFLICT` `409`, cualquier otro `422`), un solo prefijo por agregado (D-114), la clave `error.<código-en-kebab>.message` en `messages.properties` y `messages_es.properties`, y su texto en `es.json`, que el frontend elige por `code`. La columna **Texto** es el que ve la persona, en español.

| `code` | Estado | Operación | Cuándo | Texto |
|---|---|---|---|---|
| `VEHICLE_ENABLEMENT_INCOMPLETE` | 422 | `POST /vehicles/{id}/status-changes` | El destino exige la habilitación y la unidad no la tiene completa; lo que falta, en `context` | «No pasa a {estado}: a la unidad le falta {lo que falta}.» |
| `VEHICLE_ENABLEMENT_HELD_BY_STATUS` | 422 | Retiro de un equipo, `PUT /arrival` y `PUT /milestones` | La unidad está en un estado con puerta —Disponible o Vendido— y la escritura le quitaría un requisito (D-184); `context.statusCode` y `context.requirement`: el código del equipo, `platesReceivedDate` o `arrivalDate` | Uno de cuatro, según lo que retiene y el estado (abajo, decidido el 03/11/2026) |
| `VEHICLE_ALREADY_ARRIVED` | 422 | `POST /vehicle-arrivals` | Una o más unidades ya ingresaron; `context.vehicleIds` | «Algunas unidades ya tienen su ingreso: quítelas de la selección.» |
| `VEHICLE_ARRIVAL_COMPANY_MISMATCH` | 422 | `POST /vehicle-arrivals` | Las unidades son de más de una empresa | «Un ingreso en lote es de unidades de una sola empresa.» |
| `VEHICLE_ARRIVAL_AFTER_INSTALLATION` | 422 | `PUT /vehicles/{id}/arrival` | La fecha nueva es posterior al inicio de una instalación; `context.earliestInstallation` | «El ingreso no puede ser posterior a la instalación de un equipo de la unidad.» |
| `VEHICLE_ARRIVAL_HAS_INSTALLATIONS` | 422 | `PUT /vehicles/{id}/arrival` con `arrivalDate` `null` | La unidad tiene instalaciones | «La unidad ya tiene equipos instalados: su ingreso no se quita.» |
| `VEHICLE_MILESTONE_IN_FUTURE` | 422 | `POST /vehicle-arrivals`, `PUT /arrival`, `PUT /milestones` | Una fecha es posterior a hoy en la zona de la empresa; `context.field` (D-186) | «La fecha no puede ser posterior a hoy.» |
| `VEHICLE_TITLE_DATE_WITHOUT_TITLE` | 422 | `PUT /milestones` | Fecha de título sin número | «La fecha del título se registra con su número.» |
| `VEHICLE_TITLE_REQUIRED_BY_PLATE` | 422 | `PUT /milestones` | Se vacía el título de una unidad con placa | «La unidad ya tiene placa: el título no se quita.» |
| `VEHICLE_PLATES_RECEIVED_WITHOUT_PLATE` | 422 | `PUT /milestones` | Placas físicas sin placa registrada | «Registre la placa antes de marcar las placas físicas.» |
| `VEHICLE_PLATE_ALREADY_REGISTERED` | 422 | `POST /vehicles/{id}/plate` | La unidad ya tiene placa | «La unidad ya tiene placa: use «Corregir placa».» |
| `VEHICLE_PLATE_TITLE_MISSING` | 422 | `POST /vehicles/{id}/plate` | Sin título | «La placa se registra cuando la unidad tiene su título.» |
| `VEHICLE_PLATE_IDENTIFICATION_DOCUMENT_MISSING` | 422 | `POST /vehicles/{id}/plate` | Sin TIVE vigente | «La placa se registra cuando la unidad tiene vigente su tarjeta de identificación vehicular.» |
| `VEHICLE_OUTDATED` | 422 | `PUT /arrival`, `PUT /milestones` | El `updatedAt` no es el de la unidad | «La unidad cambió mientras la editaba: se muestra como está ahora. Revise y guarde otra vez.» Desde `ea714437`; antes, «… revise los datos y guarde otra vez.» |
| `VEHICLE_ARRIVED_WITHOUT_LOCATION` | 422 | `PUT /vehicles/{id}` | La edición deja sin ubicación una unidad ingresada (D-188) | «Una unidad que ya ingresó tiene ubicación: elija dónde está.» |
| `VEHICLESTATUS_ENABLEMENT_LOCKED` | 422 | `PUT /vehicle-statuses/{id}` | Se desmarca `requiresEnablement` de `DISPONIBLE` (D-170) | «Disponible exige siempre la habilitación completa: esa marca no se quita.» |
| `VEHICLESTATUS_ENABLEMENT_INITIAL` | 422 | `PUT /vehicle-statuses/{id}` | El estado inicial se marca, o un estado marcado pasa a inicial (D-164). Sin `context`; antes de `VEHICLESTATUS_INITIAL_CONFLICT`. Desde `345c1f75` | «El estado inicial no puede exigir la habilitación completa, y un estado que la exige no puede ser el inicial.» |
| `VEHICLESTATUS_ENABLEMENT_AUTO_TARGET` | 422 | `PUT /vehicle-statuses/{id}` | La marca pasa a verdadero y caminos abiertos que esperan en otro estado que Vendido mueven solas unidades a este (D-164, D-170); `context.reasonCodes`. Desde `345c1f75` | «Este estado es el destino automático de {caminos}: no puede exigir la habilitación completa. Cambie antes el destino de esos caminos.» |
| `STATUSCHANGEREASON_AUTO_TARGET_GATED` | 422 | `POST /status-change-reasons`, `PUT /status-change-reasons/{id}`, `PATCH /status-change-reasons/{id}/status` | El estado al que el camino mueve sola la unidad exige la habilitación y el camino no espera en Vendido (D-164, D-170); `context.statusCode`. Desde `345c1f75` | En el diálogo del camino, «{estado} exige la habilitación completa: solo un camino a Vendido puede llevar la unidad sola ahí. Elija otro estado.»; en un aviso fuera de él, «Un camino no puede llevar una unidad sola a {estado}, que exige la habilitación completa: solo la vuelta desde Vendido puede hacerlo. Elija otro estado.» |
| `VEHICLEDOCUMENTTYPE_NOT_FOUND` | 404 | Rutas del tipo; subida de un documento | No existe | «No encontramos ese tipo de documento: vuelva a cargar la página.» |
| `VEHICLEDOCUMENTTYPE_CONFLICT` | 409 | Alta y edición del tipo | El código ya existe en el país, también en una carrera | «Ya existe un tipo de documento con ese código en el país.» |
| `VEHICLEDOCUMENTTYPE_CODE_LOCKED` | 422 | `PUT /vehicle-document-types/{id}` | Se corrige `TIVE` o el código de un tipo con documentos (D-74) | «El código ya no se corrige: lo leen las reglas del sistema o lo usa un documento.» |
| `VEHICLEDOCUMENTTYPE_INACTIVE` | 422 | Subida de un documento | El tipo está desactivado | «Ese tipo de documento está desactivado: ya no se carga.» |
| `VEHICLEDOCUMENT_NOT_FOUND` | 404 | `/vehicle-documents/{id}` y su archivo | No existe o su unidad no se ve | «No encontramos ese documento: no existe o no está a su alcance.» |
| `VEHICLEDOCUMENT_TYPE_OTHER_COUNTRY` | 422 | Subida | El tipo es de otro país que la empresa de la unidad | «Ese tipo de documento es de otro país que la empresa de la unidad.» |
| `VEHICLEDOCUMENT_TYPE_NOT_APPLICABLE` | 422 | Subida | Un tipo solo para unidades a gas en una que no lo es (D-166) | «Ese documento se pide solo a las unidades a gas (GNV o GLP).» |
| `VEHICLEDOCUMENT_EXPIRY_REQUIRED` | 422 | Subida | El tipo vence y no trae vencimiento | «Indique la fecha de vencimiento: este documento vence.» |
| `VEHICLEDOCUMENT_EXPIRY_NOT_ALLOWED` | 422 | Subida | El tipo no vence y trae vencimiento | «Este documento no vence: no lleva fecha de vencimiento.» |
| `VEHICLEDOCUMENT_EXPIRY_NOT_AFTER_ISSUE` | 422 | Subida | El vencimiento no es posterior a la emisión | «El vencimiento tiene que ser posterior a la emisión.» |
| `VEHICLEDOCUMENT_ISSUE_DATE_IN_FUTURE` | 422 | Subida | La emisión es posterior a hoy (D-186) | «La fecha de emisión no puede ser posterior a hoy.» |
| `VEHICLEDOCUMENT_FILE_TYPE` | 422 | Subida | El contenido no es PDF, PNG, JPEG ni WebP (D-174) | «El archivo tiene que ser PDF, PNG, JPEG o WebP.» |
| `VEHICLEDOCUMENT_FILE_SIZE` | 422 | Subida | Pesa más de 5 MB y hasta 10 MB (D-174) | «El archivo pesa más de 5 MB.» |
| `VEHICLEEQUIPMENT_NOT_FOUND` | 404 | `/vehicle-equipments/{id}` y sus hechos | No existe o su unidad no se ve | «No encontramos ese equipo: no existe o no está a su alcance.» |
| `VEHICLEEQUIPMENT_CONFLICT` | 409 | Instalar y reemplazar | Otro del mismo tipo abierto o un período que se cruza, también en una carrera (`23P01`). Sin campo | «La unidad ya tiene un equipo de ese tipo en esas fechas: reemplácelo.» |
| `VEHICLEEQUIPMENT_NOT_ARRIVED` | 422 | Instalar | La unidad no ingresó (§11.4) | «Registre el ingreso de la unidad antes de instalarle un equipo.» |
| `VEHICLEEQUIPMENT_BEFORE_ARRIVAL` | 422 | Instalar y reemplazar | El inicio es anterior al ingreso; `context.arrivalDate` | «La instalación no puede ser anterior al ingreso de la unidad.» |
| `VEHICLEEQUIPMENT_TYPE_NOT_APPLICABLE` | 422 | Instalar | El equipo no aplica al tipo de vehículo (§7.5) | «Ese equipo no se instala en este tipo de vehículo.» |
| `VEHICLEEQUIPMENT_SERIAL_REQUIRED` | 422 | Instalar y reemplazar | Falta el serial de un tipo que lo exige | «Este equipo exige su número de serie.» |
| `VEHICLEEQUIPMENT_ALREADY_REMOVED` | 422 | Reemplazar y retirar | El período ya está cerrado, también por otro pedido que confirmó antes | «Ese equipo ya se retiró.» |
| `VEHICLEEQUIPMENT_END_NOT_AFTER_START` | 422 | Reemplazar y retirar | El fin no es posterior al inicio | «El retiro tiene que ser posterior a la instalación.» |
| `VEHICLEEQUIPMENT_DATE_IN_FUTURE` | 422 | Instalar, reemplazar y retirar | Una fecha posterior a hoy (D-186) | «La fecha no puede ser posterior a hoy.» |
| `VEHICLEEQUIPMENT_COST_SCALE` | 422 | Instalar y reemplazar | El costo tiene más decimales que su moneda | «El costo tiene más decimales que su moneda.» |
| `VEHICLEEQUIPMENT_INSTALLER_OTHER_COMPANY` | 422 | Instalar y reemplazar | Quien instaló es de otra empresa | «Quien instaló es colaborador de otra empresa.» |
| `EQUIPMENTTYPE_INACTIVE` | 422 | Instalar y reemplazar | El equipo está desactivado (D-108). Nombra el agregado de `vehiclecatalog`, como `VEHICLEMODEL_INACTIVE` | «Ese equipo está desactivado: ya no se instala.» |
| `VEHICLEKEY_NOT_FOUND` | 404 | `/vehicle-keys/{id}` | No existe o su unidad no se ve | «No encontramos esa llave: no existe o no está a su alcance.» |
| `VEHICLEKEY_CONFLICT` | 409 | Alta y edición de una llave | La etiqueta ya la tiene otra llave de la empresa, también en una carrera. Sin campo | «Otra llave de la empresa ya tiene esa etiqueta.» |
| `VEHICLEKEY_STORAGE_GROUP_NOT_ALLOWED` | 422 | Alta y edición | Caja en una llave que no está en la empresa | «Solo una llave que está en la empresa se guarda en una caja.» |
| `VEHICLEKEY_OUTDATED` | 422 | `PUT /vehicle-keys/{id}` | El `updatedAt` no es el de la fila | «La llave cambió mientras la editaba: revise los datos y guarde otra vez.» |

Son 45 códigos nuevos, todos de esta fase, más los tres que sumó la revisión el 04/11/2026: `VEHICLESTATUS_ENABLEMENT_INITIAL`, `VEHICLESTATUS_ENABLEMENT_AUTO_TARGET` y `STATUSCHANGEREASON_AUTO_TARGET_GATED` (sección 4.3). Ninguno coincide en nombre ni en sentido con los de la fase 2 (su §4.3): los archivos del documento tienen su prefijo propio, `VEHICLEDOCUMENT_FILE_*`, como la factura tiene `PURCHASEINVOICE_FILE_*` y la plantilla el suyo (D-114). Se reutilizan, con su texto de siempre:
- de la fase 2, que es su dueña: `VEHICLE_ANNULLED` («La unidad está anulada: ya no se modifica»), en toda escritura de la habilitación y en el cambio de estado, y `VEHICLE_MODEL_YEAR_LOCKED` en `PUT /vehicles/{id}`;
- de `fleet`: `VEHICLE_NOT_FOUND` (con `context.vehicleIds` en el lote) y `VEHICLE_PLATE_CONFLICT`;
- de `organization`: `LOCATION_NOT_FOUND`, `LOCATION_OTHER_COMPANY`, `LOCATION_INACTIVE`, `EMPLOYEE_NOT_FOUND` y `EMPLOYEE_INACTIVE`;
- de `location`: `COUNTRY_NOT_FOUND`, `CURRENCY_NOT_FOUND` y `CURRENCY_INACTIVE`;
- de `vehiclecatalog`: `EQUIPMENTTYPE_NOT_FOUND`;
- los generales: `VALIDATION_ERROR`, `ACCESS_DENIED`, `CONTENT_TOO_LARGE`, `DOCUMENT_*` y `DOCUMENTS_BUSY`.

**Los textos que compone la pantalla.** El de `VEHICLE_ENABLEMENT_INCOMPLETE` lo arma el frontend con `context`, en el orden de las filas:
- «el ingreso» si `arrived` es falso;
- los documentos por su nombre;
- los equipos con su nombre corto: «GPS» y «pantalla» para los códigos que conoce (D-171) y el nombre del catálogo para otro;
- «las placas físicas» si `platesReceived` es falso.

Por ejemplo: «No pasa a Disponible: a la unidad le falta la tarjeta de identificación vehicular, el GPS y las placas físicas.» **Como se construyó** (`4f0ca720`): el verbo concuerda con lo primero que falta, «Faltan las placas físicas.» cuando son lo primero (`change-vehicle-status-dialog.spec`, "agrees the verb of a refusal of the gate with what the unit lacks first"; `enablement-texts.spec`, "says the verb agrees in plural only when the physical plates are named first").

**El texto de `VEHICLE_ENABLEMENT_HELD_BY_STATUS`, decidido el 03/11/2026.** La versión 2 daba un solo texto, «La unidad está en {estado}, que exige la habilitación completa: reemplace el equipo o saque antes la unidad por «Pierde un requisito».». No sirve para las placas ni para el ingreso, que no se reemplazan, ni para Vendido, que no tiene el camino «Pierde un requisito»: V96 lo siembra solo desde Disponible. La coordinación del plan, por delegación del responsable, decidió cuatro textos, que el frontend elige por `context.requirement` y `context.statusCode` (`c7d4470b`):

| Lo que retiene | Estado | Texto |
|---|---|---|
| Un equipo | `DISPONIBLE` | «La unidad está en {estado}, que exige la habilitación completa: reemplace el equipo o saque antes la unidad por «Pierde un requisito».» |
| Un equipo | Otro estado con puerta | «La unidad está en {estado}, que exige la habilitación completa: reemplace el equipo.» |
| Las placas o el ingreso | `DISPONIBLE` | «La unidad está en {estado}, que exige la habilitación completa: saque antes la unidad por «Pierde un requisito».» |
| Las placas o el ingreso | Otro estado con puerta | «La unidad está en {estado}, que exige la habilitación completa: el dato no se quita mientras siga en ese estado.» |

`{estado}` es el nombre del estado; mientras los estados no se leyeron, su código. **Como se corrigió (04/11/2026)** (`ab67dce4`): el nombre sale del recurso de la habilitación de la unidad (`statusOfRecord`, con `currentStatusCode` y `currentStatusName`, sección 4.5), no del catálogo de estados, que pide `catalogos.ver`; si el estado del rechazo no es el que el recurso nombra, el texto dice su código. Una respuesta sin `context` muestra el texto del código: «La unidad está en un estado que exige la habilitación completa: reemplace el equipo o saque antes la unidad por «Pierde un requisito».». Lo prueban `enablement-texts.spec` ("offers «Pierde un requisito» for an equipment only in Disponible, and the replacement in both", "never asks to replace a milestone: the plates or the arrival", "names the status by its code while the statuses are not read") y `remove-vehicle-equipment-dialog.spec` ("says the status that holds a needed equipment by its name, and what to do in it (D-184)").

### 4.11 Versiones y concurrencia

- **El candado de `updatedAt`** (convenciones §3) lo llevan `PUT /vehicles/{id}/arrival`, `PUT /vehicles/{id}/milestones` (el de la unidad) y `PUT /vehicle-keys/{id}` (el de la llave). Se compara al milisegundo bajo el candado de la fila: uno distinto responde `422 VEHICLE_OUTDATED` o `VEHICLEKEY_OUTDATED` y no guarda nada; uno que falta, `400`. La pantalla, ante ese código, vuelve a leer y lo dice. **Como se corrigió (04/11/2026)** (`ea714437`): «Editar hitos» mandaba el `updatedAt` de la unidad leída otra vez con los valores leídos antes, así que un segundo «Guardar» pisaba el cambio de la otra persona. Ahora el formulario de los hitos y el diálogo del ingreso de una unidad guardan el `updatedAt` con el que leyeron sus valores y, después de `VEHICLE_OUTDATED`, toman la unidad leída otra vez, sus valores y su `updatedAt` juntos, y conservan el texto del rechazo.
- **El `updatedAt` de la unidad lo mueve también un cambio de estado o una edición de sus datos**: quien edita los hitos mientras otro mueve la unidad recibe `VEHICLE_OUTDATED` y vuelve a leer. Es el costo aceptado de un solo candado por fila.
- **Las escrituras sin `updatedAt`** —subir un documento, instalar, reemplazar y retirar un equipo, registrar la placa y el ingreso en lote— son hechos que se suman o se cierran. Las serializa el candado de la unidad, y las respuestas `_ALREADY_REMOVED`, `_ALREADY_ARRIVED` y `_PLATE_ALREADY_REGISTERED` dicen que otro llegó antes.
- **Los catálogos no llevan candado de `updatedAt`** (convenciones §14.7): los escribe solo el superadministrador.
- **El único parcial de los documentos vigentes no se traduce a propósito**: la subida corre bajo el candado de la unidad, así que una violación de `vehicle_documents_current_uq` es un defecto que tiene que verse, como la numeración de las órdenes.

---

## 5. Reglas

Capas y marcas según el [README de los contextos](contextos/README.md). Las pruebas del backend van en `src/test/java/.../fleet` salvo que se diga otro paquete; las del frontend, junto a cada archivo de `src/app/fleet`; las de extremo a extremo, en `e2e/enablement.spec.ts` y `e2e/vehicle-document-types.spec.ts`. Cada regla pasa a ✅ cuando su prueba existe y pasa: desde el 03/11/2026 lo están todas, con el nombre real de su prueba y, al final de **Dónde se aplica**, los commits que la construyeron. Donde la versión 2 nombraba una prueba que no existe, la fila lo dice y cita la que cubre la regla. Desde la versión 4 (04/11/2026), las filas que la revisión de punta a punta cambió dicen desde qué commit, y las pruebas que se movieron citan su clase de hoy: las de una fecha de mañana en Lima pasaron a `EnablementDatesInLimaTests`, con el reloj fijo (`6165ba5d`), y `7b92a7de` reforzó las de FLT-R49, FLT-R51, FLT-R70, FLT-R84 y FLT-R86 (las dos primeras, de la fase 2, en `contextos/fleet.md`).

### 5.1 `fleet`

| ID | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| FLT-R55 | `requires_enablement` marca los estados a los que solo se entra con la habilitación completa; nace en falso. La semilla lo marca en Disponible, donde no se desmarca, y en Vendido, donde sí; en cualquier estado salvo Disponible se marca y se desmarca con `catalogos.editar_estados`. Desde el 04/11/2026, el estado inicial no se marca ni un estado marcado pasa a inicial, y un estado no se marca mientras un camino abierto que espera en otro estado que Vendido mueve solas unidades a él. Desde la segunda revisión, marcar un estado y guardar un camino hacia él no se cruzan | §8.2, D-164, D-170, D-184; revisión del 04/11/2026 | Base: `vehicle_statuses_enablement_gate_ck`, la semilla de V(n+1). Dominio: `VehicleStatus.update`, `EnablementGateLockedException`. Servicio: `422 VEHICLESTATUS_ENABLEMENT_LOCKED`; desde `345c1f75`, `VEHICLESTATUS_ENABLEMENT_INITIAL` y `VEHICLESTATUS_ENABLEMENT_AUTO_TARGET` (sección 4.3); desde `5bcd796b`, el estado se lee `FOR NO KEY UPDATE` al marcarlo y `FOR SHARE` al guardar un camino hacia él (sección 2.3). Frontend: `vehicle-status-form-dialog` bloquea la casilla en `DISPONIBLE`, y desde `7d5e5dce` retiene «Estado inicial» mientras el estado exige la habilitación y la casilla mientras es inicial. Commits: backend `9ca62ea4`, `51070252`, `345c1f75`; frontend `1ee3f40d`, `7d5e5dce` | `VehicleStatusTests.disponibleAlwaysRequiresEnablement`, `anotherStatusMarksAndUnmarksItsGateAndANewOneIsBornWithout`; `VehicleStatusCatalogTests.theGateOfDisponibleIsNotUnmarkedAndAnotherStatusTogglesIt`, `theGateOfVendidoIsUnmarkedAndMarkedAgain`; `EnablementGateMigrationTests.theSeedMarksDisponibleAndVendidoAndTheDatabaseRefusesToUnmarkDisponible`; `AutomaticReturnGateTests.aStatusThatAnOpenPathReturnsUnitsToAloneIsNotGated`, `theInitialStatusNeverRequiresTheEnablementSoTheRegistrationIsNeverGated`; `vehicle-status-form-dialog.spec` ("locks the gate of DISPONIBLE and says why", "lets the gate of VENDIDO be unmarked", "is not offered to a new status, which is born without it", "holds «Estado inicial» while the status asks the enablement, and the gate while it is initial (C4)", "never holds a box already marked, so a status saved with both can be corrected", "says the refusal with what it names"); `vehicle-statuses.store.spec` ("says the gate of Disponible is never unmarked (D-170)", "says why a status cannot ask the enablement: initial, or the destination of paths it names (C4)"); e2e ("lets the mark of Vendido go, and keeps the one of Disponible", "keeps the gate of the enablement off the initial status and off where a path moves a unit by itself, as the backend does (C4)"); desde la segunda revisión, `StatusCatalogLockTests.markingAStatusWaitsForAPathReopenedIntoItAndSeesIt`, `reopeningAPathWaitsForTheMarkOfItsDestinationAndSeesIt`, con dos transacciones reales | ✅ |
| FLT-R56 | Un cambio de estado a mano hacia un estado con `requires_enablement` exige que a la unidad no le falte nada —ninguna de las filas 1 a 4—, esté Listo para entrega o Entregado, por cualquier camino —«Habilitación completa», «Reacondicionada», «Venta»—, y si no, se rechaza nombrando todo lo que falta. La puerta va después del permiso del camino y de la guarda de la unidad anulada de la fase 2 | §8.2, §11.9, D-164, D-183 | Dominio: `Vehicle.transitionTo(Long, StatusPath, EntryGate)`, `EntryGate`, `EnablementIncompleteException`. Servicio: `VehicleMovements.recordMove`, `422 VEHICLE_ENABLEMENT_INCOMPLETE` con su `context`. Frontend: `change-vehicle-status-dialog` muestra el camino retenido con lo que falta. Commits: backend `51070252`, `805b12f8`; frontend `2cf89e48`, `b1f382e9`, `4f0ca720` | `VehicleTests.theUnitRefusesToEnterAGatedStatusWithoutItsEnablement`; `EnablementGateIntegrationTests.aMissingRequirementRefusesTheManualMoveIntoDisponibleAlongBothPaths` (parametrizada: falta la TIVE, la pantalla de un auto que la requiere, el GPS o las placas físicas, por `HAB_COMPLETA` y por `REACONDICIONADA`; el historial no cambia), `aCompleteUnitEntersDisponibleAlongBothPaths`, `aReconditionedUnitDeliveredBeforeReturnsToDisponible` (desde `805b12f8`, la unidad de «Reacondicionada» recorre antes Vendido, Activo, Custodia y Puesta a punto), `aSaleIntoVendidoIsHeldWhileTheUnitLacksARequirement`, `aStatusMarkedLaterHoldsTheMovesIntoIt` y `theGateAnswersAfterThePermissionOfThePathAndTheAnnulment`; `change-vehicle-status-dialog.spec` ("shows a held path disabled with what the unit lacks", "names what the unit lacks when the gate refused the move meanwhile, and keeps the dialog open", "keeps the link to the enablement when the destination, touched and empty, says its error"); `vehicles.store.spec` ("names what the unit lacks when the gate refuses a move, and reads the unit again (VEHICLE_ENABLEMENT_INCOMPLETE)"); e2e ("shows Disponible held with what the unit lacks, and moves it once the unit is complete") | ✅ |
| FLT-R57 | El regreso automático de Vendido a Disponible no pasa por la puerta: deshace una venta que no ocurrió. Desde el 04/11/2026 es el único: ningún otro camino mueve sola una unidad a un estado con puerta, el catálogo no lo guarda y el proceso deja donde está a una unidad incompleta que un camino guardado antes movería. Desde la segunda revisión, el regreso desde Vendido no aplica la puerta solo mientras Vendido tenga `requires_enablement`: desmarcado, el proceso lee la habilitación y deja en Vendido a la unidad a la que le falta algo; el catálogo mantiene la exención por el código `VENDIDO`. El aviso sale una vez por unidad retenida | D-164, D-110, D-184, D-170; las dos revisiones del 04/11/2026 (R23 y R24, propuestas para el diccionario) | Dominio: `Vehicle.transitionAlone` no consulta la puerta; desde `345c1f75`, `AutomaticReturnGate` y `VehicleStatus.SOLD_CODE`; desde `a126216f`, `AutomaticReturnGate.appliesToTheReturn`. Servicio: `VehicleMovements.returnAlone`, que desde `345c1f75` deja la unidad incompleta con un `WARN`, y desde `62708a60` solo en la primera corrida después del vencimiento (`firstRunAfterTheBreach`); `422 STATUSCHANGEREASON_AUTO_TARGET_GATED` en el alta, la edición y la reapertura de un camino (sección 4.3). Frontend: `status-change-reason-form-dialog` muestra retenidos en «Mover a» los estados que exigen la habilitación, con «Exige la habilitación completa», salvo en un camino a Vendido, y nombra el estado por su nombre del catálogo. Commits: backend `51070252`, `805b12f8`, `345c1f75`; frontend `7d5e5dce`, `660d15dc`, `891f3fa0` | `EnablementGateIntegrationTests.theAutomaticReturnOfASoldUnitIsNeverHeld` (a una unidad Vendida se le vuelve bloqueante un tipo de documento que no tiene; vuelve igual a Disponible a las 48 h con el motivo `REGRESO_AUTOMATICO`, y la lista la muestra `SIN_DOCUMENTO`); `AutomaticReturnGateTests.onlyAPathThatWaitsInVendidoMovesUnitsAloneIntoAGatedStatus`, `theJobLeavesAnIncompleteUnitWhereAPathSavedBeforeTheGateWouldMoveIt` y, desde la segunda revisión, `theJobWarnsAboutAHeldUnitOnlyOnTheFirstRunAfterItsBreach`; `EnablementGateIntegrationTests.anUnmarkedVendidoReturnsAloneOnlyAUnitThatLacksNothing`, `aMarkedVendidoReturnsTheSoldUnitAlone`; `AutomaticReturnGateRuleTests` (las tres); `VehicleMovementsBreachWarningTests`; `status-change-reason-form-dialog.spec` ("holds a status that asks the enablement while the path waits elsewhere than Vendido", "lets the return from Vendido move the unit alone into Disponible", "holds none in a closed path: the backend checks it again when the path is reopened", "names the gated status a path cannot move a unit to by itself by the name of the catalog (C4)"); `status-change-reasons.store.spec` ("says a path reopened beside another registration path, or returning units to a retired status, is refused", que suma el destino con puerta); e2e ("keeps the gate of the enablement off the initial status and off where a path moves a unit by itself, as the backend does (C4)") | ✅ |
| FLT-R58 | El estado de habilitación se calcula, nunca se guarda: seis filas en orden, la primera que aplica, con su código estable y su detalle. Entregado es haber pasado a Activo; su momento, el de la primera vez. Un documento vigente cuenta aunque haya vencido | §11.9, D-171, D-183, D-188 | Dominio: `EnablementPolicy.evaluate`, `EnablementStatusCode`, `EnablementStatus`. Servicio: `VehicleEnablementFactsReader`. Frontend: `enablement-status-badge`. Commits: backend `d8a4afd1`; frontend `538c1534` | `EnablementPolicyTests.theSixRowsAreEvaluatedInTheirOrderAndTheFirstThatAppliesIsAnswered`, `aUnitThatHasNotArrivedIsOutOfStoreWhateverElseItLacks`, `aMissingBlockingDocumentIsNamedInTheDetail`, `everyMissingEquipmentIsNamedInTheOrderOfItsCode`, `theStatusCarriesEverythingMissingBeyondItsRow`, `aCompleteUnitIsReadyAndOnceDeliveredIsDelivered`, `aDeliveredUnitIsCompleteToo`, `anExpiredBlockingDocumentStillCounts`; `VehicleEnablementFactsReaderTests.theFirstMoveIntoActivoIsTheDelivery`, `anExpiredCurrentDocumentOfABlockingTypeCounts`; `enablement-status-badge.spec` ("says each status by the name of its code, never by its color alone", "takes the tone of revision 35: waiting outside neutral, work to do warning, ready success, delivered info") | ✅ |
| FLT-R59 | La consulta de la lista responde lo mismo que la política para cada unidad | Plan, paso 8.5; §7.5 («escrita una sola vez») | Persistencia: `JdbcVehicleEnablementReadRepository`. Commits: backend `ba0ce3f5` | `VehicleEnablementProjectionTests.theProjectionAnswersWhatThePolicyAnswers` | ✅ |
| FLT-R60 | Lo que bloquea sale de los datos: los tipos de documento y los equipos con `blocks_availability`, activos y que aplican a la unidad. Marcar otro como bloqueante lo suma a la fila 2 o a la 3 sin programar | §11.2, §11.9, D-165 | Servicio: `VehicleEnablementFactsReader`; la consulta de la lista. `vehiclecatalog`: `fetchEnablementProfilesByModelIds`. Commits: backend `dd67d588`, `d8a4afd1`, `51070252`, `ba0ce3f5` | `EnablementGateIntegrationTests.aDocumentTypeMarkedBlockingHoldsTheUnitWithoutProgramming`; `VehicleEnablementProjectionTests.aDeactivatedBlockingTypeNoLongerHoldsTheUnit`; `VehicleCatalogContextFacadeTests.aRetiredEquipmentOrApplicabilityIsNotRequired` | ✅ |
| FLT-R61 | La habilitación de una unidad se lee con `stock.ver` donde se ve, marca incluida; su lista exige la empresa. Desde el 04/11/2026, el recurso y la fila traen el código de su estado y si exige la habilitación, así que la pantalla no lee el catálogo de estados, que pide `catalogos.ver`, y la lista muestra su filtro por estado solo con ese permiso | §5.6 (`stock.ver`), D-14, D-99, D-102; revisión del 04/11/2026 | Servicio: `VehicleEnablementQueryServiceImpl`, `VehicleAccessPolicy.reach` y `sees`. Recurso: `currentStatusCode` y `currentStatusRequiresEnablement` (sección 4.5). Frontend: `vehicle-enablement-view` y `enablement-view` leen el estado del recurso. Commits: backend `d8a4afd1`, `ba0ce3f5`, `4aa250b9`; frontend `538c1534`, `5f66d62e`, `ab67dce4` | `VehicleEnablementIntegrationTests.aUnitOutOfReachIsNotFoundAndItsCompanyListsNothing`, `anAssignmentOfABrandSeesOnlyTheEnablementOfItsBrand`, `theRecordNamesTheStatusTheMilestonesTheRequirementsAndTheActions`, que desde `4aa250b9` exige `currentStatusCode` y `currentStatusRequiresEnablement`; `VehicleEnablementListIntegrationTests.theListFiltersByTheCodeOfItsStatus`, que exige `currentStatusRequiresEnablement` en la fila; `fleet.routes.spec` ("opens the enablement with stock.ver", "opens the enablement of a unit with stock.ver, under its own heading"). El filtro por estado de la lista no tiene prueba propia | ✅ |
| FLT-R62 | El ingreso físico se registra por unidad o en lote, hasta 100, todo o nada, y fija la ubicación, activa y de la empresa de las unidades; su fecha no es futura en la zona de la empresa. El permiso actúa donde está cada unidad y en la ubicación de destino | §11.1, D-165, D-160, D-101, D-163, D-186 | Dominio: `Vehicle.registerArrival`. Servicio: `VehicleArrivals`; `422 VEHICLE_ALREADY_ARRIVED`, `VEHICLE_ARRIVAL_COMPANY_MISMATCH`. Frontend: `register-arrival-dialog`. Commits: backend `4c279cc3`; frontend `9094fc55` | `VehicleArrivalBatchIntegrationTests.eightyUnitsArriveInOneOperationWithTheirLocation`, `theStatementsDoNotGrowPerUnit`, `aBatchWithOneUnitAlreadyArrivedWritesNothing`, `aBatchOfTwoCompaniesIsRefused`, `aRoleOfABranchDoesNotReceiveAUnitWithoutLocation`; `EnablementDatesInLimaTests.anArrivalOfTomorrowInLimaIsRefused`, con el reloj fijo a las 20:00 de Lima del 03/11/2026, ya el 04/11 en UTC (desde `6165ba5d`; antes en `VehicleArrivalBatchIntegrationTests`, que dependía de la hora en que corría la suite); `register-arrival-dialog.spec` ("proposes today in the zone of the company and offers its active locations", "refuses a day after today in the zone of the company (D-186)", "marks the units that arrived already and stays open with the reason"); `vehicle-enablements.store.spec` ("registers the arrival of the selection and marks the units the server says arrived already", "says a role of a branch does not receive a unit without a location"); e2e ("registers the arrival of three units out of the store at one location, and marks one that arrived meanwhile without saving") | ✅ |
| FLT-R63 | El ingreso se conserva como hito: corregirlo no lo lleva después de una instalación, y volver a «no ingresada» solo corrige un error, sin instalaciones. Una unidad ingresada no queda sin ubicación, tampoco al editarla en Inventario | §11.1, §8.1, D-165, D-188; guía, columna 13 | Base: `vehicles_arrived_location_ck`. Dominio: `Vehicle.correctArrival`, `Vehicle.updateGeneralData`. Servicio: `422 VEHICLE_ARRIVAL_AFTER_INSTALLATION`, `VEHICLE_ARRIVAL_HAS_INSTALLATIONS`, `VEHICLE_ARRIVED_WITHOUT_LOCATION`. Frontend: `vehicle-form-dialog` no ofrece «Sin ubicación» a una unidad ingresada. Commits: backend `9ca62ea4`, `4c279cc3`; frontend `9094fc55`, `5f66d62e`, `d3a03220`, `ea714437` | `VehicleArrivalIntegrationTests.anArrivalIsNotMovedAfterAnInstallation`, `anArrivalWithInstallationsIsNotCleared`, `anArrivedUnitIsNotLeftWithoutLocation`; `EnablementGateMigrationTests.theMilestonesKeepTheirRulesInTheDatabase`; desde la segunda revisión (backend `22b16143`), `EnablementDatesInLimaTests.aCorrectedArrivalOfTomorrowInLimaIsRefused`, con el reloj fijo: `PUT /vehicles/{id}/arrival` rechaza mañana en Lima y toma hoy; `vehicle-form-dialog.spec` ("does not offer Sin ubicación to a unit that arrived"); `register-arrival-dialog.spec` ("corrects the arrival of one unit by its own route, with the updatedAt read", "neither offers nor proposes a location deactivated since, the current one included: the backend refuses it", y desde `ea714437` "after VEHICLE_OUTDATED takes the arrival read again, with its updatedAt, and keeps the refusal said"; desde `5d6d34c5`, "after VEHICLE_OUTDATED clears a location the unit read again holds that was deactivated, and says it") | ✅ |
| FLT-R64 | El número de título no es único; la fecha del título existe solo con título; el título no se quita mientras la unidad tiene placa; ninguna fecha de los hitos es futura en la zona de la empresa | §11.1, D-186; guía, columnas 14 y 15 | Base: `vehicles_title_number_ck`, `vehicles_title_date_ck`, `vehicles_plate_title_ck`. Dominio: `Vehicle.updateMilestones`. Servicio: `422 VEHICLE_MILESTONE_IN_FUTURE`. Frontend: desde `ea714437`, el formulario guarda con el `updatedAt` de los valores que muestra. Commits: backend `9ca62ea4`, `1aaf61a8`; frontend `5f66d62e`, `ea714437` | `VehicleMilestonesTests.aTitleDateNeedsItsTitleAndATitleIsNotClearedUnderAPlate`; `VehicleMilestonesIntegrationTests.twoUnitsShareATitleNumber`; `EnablementDatesInLimaTests.aMilestoneOfTomorrowInLimaIsRefused` (desde `6165ba5d`; antes en `VehicleMilestonesIntegrationTests`); `EnablementGateMigrationTests.theMilestonesKeepTheirRulesInTheDatabase`; `vehicle-milestones-form.spec` ("refuses a day after today in the zone of the company and a title date without its title", "sends the four milestones with the updatedAt read, a blank title as none", y desde `ea714437` "takes the unit read again, so a second save never overwrites the other change (lost update)", "never sends a later updatedAt with the values read before, while no refusal said the unit changed") | ✅ |
| FLT-R65 | La primera placa la registra `habilitacion.registrar_hitos`, solo con el título y la TIVE vigente del país de la unidad; se normaliza y no se repite en el país. En un país sin el tipo `TIVE`, no se registra hasta decidirlo con su primera empresa. Corregirla sigue siendo `stock.corregir_placa` con motivo | §11.1, §7.6, §5.6, D-139, D-188, D-52 | Dominio: `Vehicle.registerPlate`. Servicio: `VehiclePlateRegistrations`, `422 VEHICLE_PLATE_ALREADY_REGISTERED`, `VEHICLE_PLATE_TITLE_MISSING`, `VEHICLE_PLATE_IDENTIFICATION_DOCUMENT_MISSING`; `409 VEHICLE_PLATE_CONFLICT`. Commits: backend `d6bee720`; frontend `5f66d62e` | `VehiclePlateRegistrationIntegrationTests.aPlateIsRegisteredOnlyWithTheTitleAndTheTive`, `aPlateTakenInTheCountryAnswersItsConflict`, `aPlateAlreadyRegisteredIsCorrectedNotRegisteredAgain`, `aUnitOfACountryWithoutTiveRegistersNoPlate` y `aPlateRegisteredByTwoUnitsAtOnceAnswersTheConflict`, que hace la carrera con un espía; la traducción del choque en la base la prueba `VehicleLockTests.aDuplicatePlateTheDatabaseRefusesIsRaisedAsThePlate`. La versión 2 ponía esta carrera en `VehicleRacedIdentifierTests`, donde no hay una prueba de la primera placa: es la de arriba; `register-plate-dialog.spec` ("registers the plate normalized, without hyphens or spaces, and closes", "refuses a plate that does not fit before sending it"); `vehicle-enablement-view.spec` ("holds Registrar placa until the title and the TIVE are there") | ✅ |
| FLT-R66 | Tener placas es tener las placas físicas (`plates_received_date`), que exigen la placa registrada | §11.1, §11.9, D-165; guía, columna 17 | Base: `vehicles_plates_received_ck`. Dominio: `Vehicle.updateMilestones`. Servicio: `422 VEHICLE_PLATES_RECEIVED_WITHOUT_PLATE`. Commits: backend `9ca62ea4`, `1aaf61a8`, `d8a4afd1`; frontend `5f66d62e` | `VehicleMilestonesTests.physicalPlatesNeedThePlate`; `EnablementPolicyTests.platesMeanThePhysicalOnes`; `EnablementGateMigrationTests.theMilestonesKeepTheirRulesInTheDatabase`; `vehicle-milestones-form.spec` ("asks the physical plates only of a unit with its plate (D-165)") | ✅ |
| FLT-R67 | Un solo documento vigente de cada tipo por unidad; cargar uno nuevo retira el anterior y lo conserva. Un documento no se borra ni se edita: se reemplaza | §11.3, D-28 | Base: `vehicle_documents_current_uq`, `UPDATE (is_current)` como único permiso de escritura. Servicio: `VehicleDocumentCommandServiceImpl` bajo el candado de la unidad. Commits: backend `df5633db`, `0b216d05`, `fa9b7534`; frontend `7354aee7`, `c94663bc` | `VehicleDocumentTests.aNewDocumentIsCurrentAndRetiringItKeepsIt`; `VehicleDocumentIntegrationTests.aNewDocumentReplacesTheCurrentOneAndKeepsIt`; `VehicleDocumentCurrentIndexTests.aSecondCurrentDocumentOfATypeIsRefusedByTheDatabase`, `retiringTheCurrentDocumentLetsANewOneInAndKeepsTheFirst`; `FleetRowLevelSecurityTests.aDocumentIsNeverEditedButForBeingRetired`; `upload-vehicle-document-dialog.spec` ("asks before replacing the current document, which passes to the history"); `vehicle-documents.store.spec` ("reads the current documents again once a document is loaded, and the history it unfolded") | ✅ |
| FLT-R68 | Si el tipo vence, el vencimiento es obligatorio; si no vence, no lleva; el vencimiento es posterior a la emisión, y la emisión no es futura en la zona de la empresa | §11.2, §11.3, D-186 | Base: `vehicle_documents_expiry_ck`. Dominio: `VehicleDocument`. Servicio: los tres `VEHICLEDOCUMENT_EXPIRY_*` y `VEHICLEDOCUMENT_ISSUE_DATE_IN_FUTURE`. Commits: backend `df5633db`, `fa9b7534`; frontend `7354aee7` | `VehicleDocumentTests.anExpiringTypeAsksItsExpiryAndAnotherRefusesOne`; `VehicleDocumentIntegrationTests.theFormatAndTheDatesFollowTheType`; `EnablementDatesInLimaTests.anIssueDateOfTomorrowInLimaIsRefused` (desde `6165ba5d`; antes en `VehicleDocumentIntegrationTests`); `upload-vehicle-document-dialog.spec` ("asks the expiry only of a type that expires", "refuses an issue after today in the zone of the company and an expiry not after the issue") | ✅ |
| FLT-R69 | Una unidad usa los tipos activos del país de su empresa | §11.2, D-139, D-108 | Servicio: `422 VEHICLEDOCUMENT_TYPE_OTHER_COUNTRY`, `VEHICLEDOCUMENTTYPE_INACTIVE`. La base no lo protege: la tabla no tiene país. Commits: backend `fa9b7534` | `VehicleDocumentIntegrationTests.aTypeOfAnotherCountryOrRetiredIsRefused` | ✅ |
| FLT-R70 | El certificado de gas se pide, y se carga, solo en las unidades cuyo modelo usa GNV o GLP; un modelo sin combustible no es a gas | §11.2, D-166, D-109 | `vehiclecatalog`: `runsOnGas`. Servicio: `VehicleDocumentType.appliesTo(runsOnGas)`, `422 VEHICLEDOCUMENT_TYPE_NOT_APPLICABLE`; la consulta de la lista. Commits: backend `3eb04534`, `dd67d588`, `fa9b7534`, `ba0ce3f5` | `VehicleDocumentTypeTests.aTypeOnlyForGasAppliesOnlyToAGasModel`; `VehicleDocumentIntegrationTests.theGasCertificateIsAskedOnlyOfUnitsOfAGasModel`, que desde `7b92a7de` carga el certificado también a una unidad a GLP y lo rechaza en una de un modelo sin combustible; `VehicleEnablementProjectionTests.aGasModelIsHeldByItsGasCertificateOnlyIfItBlocks`; `VehicleCatalogContextFacadeTests.aModelRunsOnGasOnlyWithGnvOrGlp` | ✅ |
| FLT-R71 | El archivo de un documento es PDF, PNG, JPEG o WebP, de a lo sumo 5 MB, con el tipo leído de sus bytes; se guarda en `kept_files` con la empresa de la unidad, bajo una llave de su contenido; los bytes van a Spaces, y a la base solo donde no hay bucket, y una llave nueva se descarta si la transacción se revierte | D-174, D-189, §2.7, §11.3 | Base: `vehicle_documents_mime_type_ck`, `_file_size_ck`. `shared`: `UploadedFileType.sniff`. Servicio: `VehicleDocumentFiles`, `422 VEHICLEDOCUMENT_FILE_TYPE`, `VEHICLEDOCUMENT_FILE_SIZE`; desde `7ad20a4a`, `KeptFileStore.put` dice si escribió la llave primero (sección 2.6). Commits: backend `fa9b7534`, `7ad20a4a`; frontend `7354aee7` | `VehicleDocumentIntegrationTests.aFileIsCheckedByItsContentNotByItsName`, `aFileOfMoreThanFiveMegabytesIsRefused`, `theFileOfAnotherCompanyIsNeverOpened`; `VehicleDocumentFilesTests.aRolledBackUploadDiscardsOnlyAKeyItWroteFirst`, `aKeyACommittedDocumentHoldsIsNeverDiscarded`, `aCommittedUploadKeepsItsObject`, `aFileIsReadByItsContentAndWeighsAtMostFiveMegabytes` (la versión 2 ponía la primera en `VehicleDocumentIntegrationTests`; desde `7ad20a4a` exigen que la llave escrita primero la diga `put`, sin abrir el objeto); `KeptFileStoreUpsertTests.theDatabaseStoreTellsWhetherAPutWroteTheKeyFirst`, `theSpacesStoreTellsWhetherAPutWroteTheKeyFirst` (`documents`); `UploadedFileTypeTests.theTypeIsReadFromTheFirstBytes` (`shared`); `upload-vehicle-document-dialog.spec` ("names the types and the weight a file takes, and refuses another type or a heavier file"); e2e ("habilitates a unit on its page: arrival, card, title and plate, plates, GPS and a key", que rechaza un `.txt` renombrado `.pdf`) | ✅ |
| FLT-R72 | Un tipo de documento es por país, con el código único en él; `TIVE` no se corrige, y el código de otro tipo queda fijo desde su primer documento; el país no cambia; no se borra, se desactiva. Se exporta | §11.2, §2.4, D-74, D-139, D-106 | Base: `vehicle_document_types_country_code_uq`, `_code_ck`. Dominio: `VehicleDocumentType`. Servicio: `409 VEHICLEDOCUMENTTYPE_CONFLICT`, `422 VEHICLEDOCUMENTTYPE_CODE_LOCKED`; `FleetDocumentKinds` (`vehicle-document-types`). Recurso: `codeLocked`. Frontend: `vehicle-document-type-form-dialog`. Commits: backend `3eb04534`, `7dd80e2b`; frontend `508b371f` | `VehicleDocumentTypeTests.tiveIsNeverCorrectedAndAnotherCodeLocksWithItsFirstDocument`, `aCodeOrANameOutOfShapeIsRefused`; `VehicleDocumentTypeCatalogTests.theSeedOfPeruBringsItsFourTypes`, `aCodeRepeatedInTheCountryIsAConflictAlsoInARace`, `tiveAndACodeInUseAreLocked`, `theCatalogIsExportedWithItsCountryCode`; `VehicleDocumentTypeAuditTests.aCodeTheCountryAlreadyHoldsIsRaisedAsTaken`, `savingATypeThatAlreadyExistsKeepsItsIdentifiersItsCountryAndItsCreationMoment`; `vehicle-document-type-form-dialog.spec` ("locks the code TIVE and a code in use", "keeps the country fixed when it corrects a type, and sends a locked code unchanged"); `vehicle-document-types.store.spec` ("says the code is taken in the country, or locked"); e2e ("registers a type, refuses a code repeated in its country, and corrects the name", "keeps the code of TIVE fixed, which the rules read", "exports the list with its filter") | ✅ |
| FLT-R73 | Los tipos de documento se leen con `catalogos.ver` y se escriben con `catalogos.crear_tipos_documento`, `editar_tipos_documento` y `desactivar_tipos_documento`, que solo un rol de alcance `SYSTEM` incluye | §5.6, D-139, D-141, D-150 | Servicio: `@PermissionRequired` en `VehicleDocumentTypesController`. La regla de V89, sin cambio. Frontend: la ruta y su opción, solo con `catalogos.ver`. Commits: backend `53521c1d`, `7dd80e2b`; frontend `508b371f`, `6e3e0125` | `VehicleDocumentTypeCatalogTests.onlyTheCatalogPermissionsWriteAndEveryRoleReads`; `RolePermissionsSystemScopeTests.theDocumentTypePermissionsAreMarkedOnlyOnSystemRoles` (`iam`); `fleet.routes.spec` ("opens the document types with catalogos.ver alone", "does not open the document types to a session that only writes them (D-141)"); `vehicle-document-types-view.spec` ("offers each write only with its permission"); `vehicle-document-types.store.spec` ("says the role does not switch types, and any other refusal is a failure to try again"); e2e ("offers each write only with its permission, and opens the screen only with catalogos.ver") | ✅ |
| FLT-R74 | Dos equipos del mismo tipo nunca están instalados a la vez en una unidad, tampoco con períodos que se crucen en el pasado | §11.4, §2.2 | Base: `vehicle_equipments_no_overlap_excl`. Servicio: `409 VEHICLEEQUIPMENT_CONFLICT`, también en una carrera (`23P01`). Commits: backend `d43951eb`, `5aefa129` | `VehicleEquipmentIntegrationTests.twoOfTheSameTypeAreNeverInstalledAtOnce`, `aCrossingInARaceAnswersTheConflict`; `VehicleEquipmentOverlapTests.aCrossingTheExclusionRefusesAnswersTheConflict`; `VehicleEquipmentTests.twoPeriodsOverlapWhenTheyShareADay` | ✅ |
| FLT-R75 | Un equipo se instala solo en un tipo de vehículo al que aplica hoy, y solo si está activo | §7.5, §11.4, D-108 | `vehiclecatalog`: `applicableEquipments`. Servicio: `422 VEHICLEEQUIPMENT_TYPE_NOT_APPLICABLE`, `EQUIPMENTTYPE_INACTIVE`. Commits: backend `5aefa129` | `VehicleEquipmentIntegrationTests.anEquipmentThatDoesNotApplyToTheVehicleTypeIsRefused`, `aRetiredEquipmentTypeIsNotInstalled` | ✅ |
| FLT-R76 | Nada se instala en una unidad que no ingresó, ni en un día anterior a su ingreso; ni la instalación ni el retiro son futuros en la zona de la empresa. La ventana de ±20 días y el «no antes de la factura» de la guía no se aplican | §11.1, §11.4, D-186; guía, columnas 13, 19 y 21 | Servicio: `VehicleEquipmentInstallations`, `422 VEHICLEEQUIPMENT_NOT_ARRIVED`, `VEHICLEEQUIPMENT_BEFORE_ARRIVAL`, `VEHICLEEQUIPMENT_DATE_IN_FUTURE`, bajo el candado de la unidad. Commits: backend `5aefa129`; frontend `d2ad8cf4` | `VehicleEquipmentIntegrationTests.nothingIsInstalledOnAUnitThatHasNotArrived`, `nothingIsInstalledOnADayBeforeTheArrival`; `EnablementDatesInLimaTests.anInstallationOfTomorrowInLimaIsRefused` (desde `6165ba5d`; antes en `VehicleEquipmentIntegrationTests`); `vehicle-equipment-dialog.spec` ("does not propose a day before the arrival, nor after today in the zone of the company"); `remove-vehicle-equipment-dialog.spec` ("refuses an end not after the start or after today"); `vehicle-enablement-view.spec` ("holds Instalar equipo until the unit arrived") | ✅ |
| FLT-R77 | Un equipo que exige serial no se instala sin él | §7.5, §11.4 | Dominio: `VehicleEquipment`. Servicio: `422 VEHICLEEQUIPMENT_SERIAL_REQUIRED`. Commits: backend `d43951eb`, `5aefa129`; frontend `d2ad8cf4` | `VehicleEquipmentTests.aGpsWithoutSerialIsRefused`; `VehicleEquipmentIntegrationTests.aGpsWithoutSerialIsRefused`; `vehicle-equipment-dialog.spec` ("asks the serial only of an equipment that requires it") | ✅ |
| FLT-R78 | Reemplazar cierra el período del anterior, con motivo `reemplazo`, el mismo día en que abre el nuevo; retirar cierra con `falla` o `robo`. Ni el período ni la fila se borran. Quien instaló es un colaborador de la empresa de la unidad, que viaja por su nombre | §11.4, §2.6, D-105 | Base: `vehicle_equipments_removal_ck`, `_removal_reason_ck`, `vehicle_equipments_installer_company_fk`. Servicio: `VehicleEquipmentCommandServiceImpl`. Commits: backend `d43951eb`, `5aefa129`; frontend `d2ad8cf4` | `VehicleEquipmentTests.aRemovalClosesThePeriodAfterItsStartAndOnlyOnce`, `aReplacementClosesWithItsOwnReasonAndTheCostIsCorrected`; `VehicleEquipmentIntegrationTests.aReplacementClosesThePreviousOnTheDayTheNewOneStarts`, `aRemovalNamesItsReasonAndNeverDeletes`, `anInstallerOfAnotherCompanyIsRefused`; `VehicleEquipmentAuditTests.savingAnInstallationThatAlreadyExistsKeepsItsIdentifiersAndItsCreationMoment`; `remove-vehicle-equipment-dialog.spec` ("asks the day and the reason, a failure or a theft, and asks before removing"); `vehicle-equipment-dialog.spec` ("replaces an open equipment from the day after its start") | ✅ |
| FLT-R79 | El costo de una instalación viaja con su moneda, con a lo sumo sus decimales; leer la moneda en modo compartido antes de escribir; desde la primera instalación, los decimales de esa moneda quedan fijos | §2.2, §11.4, D-146 | Base: `vehicle_equipments_cost_ck`, `currency_id NOT NULL`. Servicio: `422 VEHICLEEQUIPMENT_COST_SCALE`. Infraestructura: `VehicleEquipmentCurrencyUsageAdapter`. Frontend: desde `311dc136`, el diálogo del equipo pide la moneda de un costo escrito, que propone en la moneda funcional de la empresa, y nunca manda un costo sin ella. Commits: backend `d43951eb`, `5aefa129`; frontend `d2ad8cf4`, `311dc136` | `VehicleEquipmentIntegrationTests.aCostWithMoreDecimalsThanItsCurrencyIsRefused`; `VehicleEquipmentCurrencyUsageTests.aCurrencyOfAnInstallationKeepsItsDecimalPlaces`; `vehicle-equipment-dialog.spec` ("refuses more decimals than the currency of the cost", y desde `311dc136` "never drops a cost typed without its currency: it asks for it, on an installation and on a cost alone", "asks the currency of a cost registered alone, never sending a cost of none") | ✅ |
| FLT-R80 | El costo de una instalación lo ve y lo escribe solo quien tiene `stock.ver_costos` en la empresa y en la marca de la unidad (desde el 04/11/2026; antes, en la empresa); sin él, el campo no viaja y se guarda en cero en la moneda funcional, y quien lo ve lo registra o lo corrige después, con `habilitacion.gestionar_equipos` y su fila en la bitácora | §2.5, §5.6, D-102, D-134, D-187, D-203, hallazgo M8; revisión del 04/11/2026 | Servicio: `VehicleEquipmentViewFactory` y `VehicleEquipmentInstallations.seesCosts`, por `VehicleAccessPolicy.seesCosts` desde `f5eacfe7`; el alta, el reemplazo y `PUT /vehicle-equipments/{id}/cost` rechazan un costo sin el permiso (`403`). Frontend: desde `ab67dce4`, la página muestra la columna Costo según `allowedActions.viewPurchaseCost` de la unidad, no según el permiso en la empresa. Commits: backend `5aefa129`, `f5eacfe7`; frontend `d2ad8cf4`, `ab67dce4` | `VehicleEquipmentIntegrationTests.theCostTravelsOnlyToWhoSeesCosts`, `aCostIsSentOnlyByWhoSeesCosts`, `whoSeesCostsCorrectsTheCostWithItsRowInTheChangeLog`, `theCostIsReadAndWrittenOnlyWithTheCostsOfTheBrandOfTheUnit`; `vehicle-equipment-dialog.spec` ("shows the cost only to who sees costs", "registers only the cost of an installation, open or closed (D-187)"); `vehicle-enablement-view.spec` ("shows the cost of the installations as the backend says the costs of the unit are read, by its company and its brand (D-102)"); `vehicle-equipments.store.spec` ("reads the installations of a unit; a cost that does not travel is none, never zero", "installs without a cost for who does not see the costs, and reads the installations again") | ✅ |
| FLT-R81 | Instalar un equipo le avisa a `vehiclecatalog`, en la misma transacción, que su tipo ya tiene instalaciones | D-74, D-107, D-185 | Servicio: `ExternalVehicleCatalogService.markEquipmentTypeInUse`. Commits: backend `dd67d588`, `5aefa129` | `VehicleEquipmentIntegrationTests.anInstallationLocksTheCodeOfItsEquipmentType`, `aRefusedInstallationLeavesTheTypeUnmarked`; `VehicleCatalogContextFacadeTests.theFirstInstallationMarksTheTypeAndMarkingAgainChangesNothing` | ✅ |
| FLT-R82 | La etiqueta de una llave es única en la empresa; el serial de fábrica se repite; solo una llave en la empresa tiene caja; las listas cerradas en minúsculas | §11.5, D-29, D-86 | Base: `vehicle_keys_company_tag_uq`, `_storage_holder_ck`, `_key_type_ck`, `_holder_ck`. Servicio: `409 VEHICLEKEY_CONFLICT`, `422 VEHICLEKEY_STORAGE_GROUP_NOT_ALLOWED`, `VEHICLEKEY_OUTDATED`. Commits: backend `8de8ab80`, `afcea799`; frontend `38af9aaa` | `VehicleKeyTests.onlyAKeyAtTheCompanyHasAStorageGroup`, `theClosedListsTakeAnyCaseAndATagIsNeverBlank`; `VehicleKeyIntegrationTests.aTagIsUniqueInTheCompanyAndASerialRepeats`, `aKeyEditedMeanwhileAnswersOutdated`; `VehicleKeyRacedTagTests.aTagTakenMeanwhileAnswersTheConflict`; `VehicleKeyAuditTests.aTagTheCompanyAlreadyHoldsIsRaisedAsTaken`, `savingAKeyThatAlreadyExistsKeepsItsIdentifiersItsUnitAndItsCreationMoment`; `vehicle-key-form-dialog.spec` ("registers a key with its label in upper case, and a storage group only at the company"); `vehicle-keys.store.spec` ("registers a key and says a label taken in the company", "corrects a key with the updatedAt read, and reads the keys again when it changed meanwhile") | ✅ |
| FLT-R83 | Los documentos, los equipos y las llaves llevan la empresa de su unidad, con una llave compuesta hacia `vehicles (id, company_id)`, Row Level Security sobre ella y ninguna fila se borra | §2.5, D-165, D-66 | Base: V(n+3), V(n+5), V(n+6). Commits: backend `df5633db`, `d43951eb`, `8de8ab80`, `0b216d05` | `FleetRowLevelSecurityTests.theScopeOfACompanySeesOnlyItsDocumentsEquipmentsAndKeys`, `aRowNamingAnotherCompanyThanItsUnitIsRefused`; `FleetRowsAreNeverDeletedTests.theApplicationCannotDeleteARowOf`, con las cuatro tablas nuevas; `RowLevelSecurityMigrationTests` (`shared`). Con el superusuario `postgres`, que se salta Row Level Security y los permisos, la primera y los casos 8 a 11 de `theApplicationCannotDeleteARowOf` fallan: el CI las corre con las dos cuentas | ✅ |
| FLT-R84 | Toda escritura de la habilitación y el cambio de estado bloquean la unidad antes de leer lo que deciden; retirar el GPS mientras otra persona habilita la unidad nunca deja una unidad Disponible sin GPS | Plan, aceptación de la fase 3; patrones de persistencia | Persistencia: `VehicleJpaRepository.lockByPublicId` y `findAllByPublicIdsForUpdate`, en orden de `id`. Servicio: el orden de la sección 2.6. Commits: backend `4c279cc3`, `4a48376a`, `805b12f8` | `EnablementConcurrencyTests.removingTheGpsWhileAnotherEnablesTheUnitNeverLeavesItDisponibleWithoutGps` (los dos órdenes, con dos transacciones reales que confirman sus unidades sobre la escena de `InvoiceRaces`; desde `805b12f8`, con su propio tipo de vehículo, `CARHAB`, cuyo GPS aplica solo mientras corren), `aBatchLocksItsUnitsInOrderOfId` y, desde `7b92a7de`, `aBatchHoldsTheLowerIdWhileItWaitsForTheHigherWhateverTheOrderOfTheRequest`: otra transacción tiene la unidad de `id` mayor, el lote la nombra primero y, mientras la espera, ya tiene la de `id` menor, que un candado sin espera no consigue; con otro orden fallaría | ✅ |
| FLT-R85 | La puerta se sostiene: en un estado con `requires_enablement` —hoy Disponible y Vendido— no se retira sin reemplazo un equipo que la unidad exige —el GPS, la pantalla que corresponde—, ni se quitan las placas físicas ni el ingreso | D-184, D-170; plan, aceptación de la fase 3 | Dominio: `RequirementHoldPolicy`. Servicio: `RequirementHolds`, `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS`, con el código del equipo, `platesReceivedDate` o `arrivalDate` en `context.requirement`. Recurso: `remove` fuera de `allowedActions`. Frontend: el panel de equipamiento dice cómo retirarlo —desde `ab67dce4`, por el estado que trae el recurso, con «Pierde un requisito» solo en Disponible—, y el rechazo dice uno de los cuatro textos que se decidieron el 03/11/2026 (especificación de habilitación, §4.10). Commits: backend `4a48376a`; frontend `d2ad8cf4`, `c7d4470b`, `ab67dce4` | `RequirementHoldPolicyTests.aGatedStatusHoldsWhatTheUnitNeeds`; `EnablementGateIntegrationTests.aDisponibleUnitKeepsItsGpsUnlessReplaced`, `aSoldUnitKeepsItsPhysicalPlates`, `anUnmarkedVendidoNoLongerHoldsTheUnit`; `vehicle-equipments.store.spec` ("says a needed equipment is held by the status, and reads the installations as they are"); `vehicle-enablement.store.spec` ("keeps the status and what the gate held when the plates are not taken from a unit in it (D-184)"); `remove-vehicle-equipment-dialog.spec` ("says the status that holds a needed equipment by its name, and what to do in it (D-184)"); `enablement-texts.spec` ("offers «Pierde un requisito» for an equipment only in Disponible, and the replacement in both", "never asks to replace a milestone: the plates or the arrival"); `vehicle-enablement-view.spec` ("says how to remove a needed equipment of a unit in Disponible, in Vendido to replace it, and in another gated status never offers «Pierde un requisito»", con ese nombre desde `ab67dce4`) | ✅ |
| FLT-R86 | Cada escritura de la habilitación exige su permiso —`registrar_hitos`, `gestionar_documentos`, `gestionar_equipos`, `gestionar_llaves`— donde está la unidad y en su marca; un rol de sede, solo sobre las unidades de su sede | §5.6, D-99, D-101, D-102 | Servicio: `@PermissionRequired` y `VehicleAccessPolicy.actsOn` en cada servicio. Commits: backend, cada escritura en el suyo, de `4c279cc3` a `afcea799`, que trae `EnablementPermissionsIntegrationTests` | `EnablementPermissionsIntegrationTests.eachWriteAsksItsPermissionWhereTheUnitIs`, `theWorkshopInstallsOnlyAtItsBranch`, `operationsRegistersTheMilestonesAndTheWorkshopDoesNot` y, desde `7b92a7de`, `aBranchRoleRegistersMilestonesAndDocumentsOnlyAtItsBranch` (un rol de sede registra hitos y carga documentos solo de las unidades de su sede, no de otra ni de una sin ubicación); `VehicleDocumentIntegrationTests.onlyWhoManagesDocumentsUploadsThem`; `vehicle-enablement-view.spec` ("offers only what the record allows") | ✅ |
| FLT-R87 | La ficha y la habilitación de una unidad dicen qué puede hacer la sesión, con las mismas reglas que cada operación; los caminos que la puerta retiene salen de `walkableReasonIds` y van a `heldReasonIds` | Convenciones §4.2; plan, paso 8.5 | Servicio: `VehicleAllowedActions`, `VehicleEnablementAllowedActions`. Commits: backend `d8a4afd1`, `118f3b11`; frontend `5f66d62e`, `2cf89e48` | `VehicleAllowedActionsTests.aPathIntoAGatedStatusIsHeldWhileTheUnitIsNotComplete`; `VehicleEnablementAllowedActionsTests.eachActionFollowsItsOperation`; `VehicleEnablementIntegrationTests.theRecordNamesTheStatusTheMilestonesTheRequirementsAndTheActions`; `vehicle-enablement-view.spec` ("offers only what the record allows"); `vehicle-profile-view.spec` ("offers the move when the only path is held, and hands the dialog what the unit lacks"); `change-vehicle-status-dialog.spec` ("offers a walkable destination as always, and no link while nothing is held") | ✅ |
| FLT-R88 | La lista de habilitación es de una empresa, se busca, se filtra por el código de su estado y se pagina en el servidor, y se exporta; los vencimientos se listan aparte y la ventana de días se cuenta desde hoy en la zona de la empresa | D-167, D-171, D-106; convenciones §6.2, §8.1 | Servicio: `VehicleEnablementQueryServiceImpl`, `FleetDocumentKinds` (`vehicle-enablements`). Persistencia: las dos lecturas de 2.9. Commits: backend `fa9b7534`, `ba0ce3f5`; frontend `538c1534`, `610efebd` | `VehicleEnablementListIntegrationTests.theListFiltersByTheCodeOfItsStatus`, `theListSortsByTheOrderOfTheRows`; `ExpiringVehicleDocumentsIntegrationTests.theFilterShowsExactlyTheDocumentsThatExpireWithinThirtyDaysInTheZoneOfTheCompany`, `theCompanyIsRequiredAndTheWindowIsOneOfItsTwoForms`; `VehicleEnablementExportTests.theFileNamesTheCodeOfTheStatusAndWhatIsMissing`, `anEqualRequestReusesTheDocumentUntilTheDataChanges`; `vehicle-enablements.store.spec` ("asks the server for the enablement of a company, by its status codes", "asks the documents that expire within thirty days", "asks the documents already expired, never with a window", "exports the query on screen as the Excel file the backend builds"); `enablement-view.spec` ("counts the days to an expiry from today in the zone of the company"); e2e ("lists the units with their badge and what each lacks, and filters them by the code", "shows the documents that expire within 30 days, counted in the zone of Lima", "exports the list of the company") | ✅ |
| FLT-R89 | Una unidad anulada no entra a la lista de habilitación ni a la de vencimientos. Que no reciba ninguna escritura de la habilitación es la regla de la fase 2 (su FLT-R49), que alcanza a estas escrituras | D-104, D-182 | Persistencia: `annulled_at IS NULL` en las dos lecturas de 2.9. Servicio: la guarda de la fase 2 en cada comando (sección 2.6). Commits: backend `afcea799`, `ba0ce3f5`; frontend `4e407db9` | `VehicleEnablementListIntegrationTests.anAnnulledUnitIsLeftOutOfBothLists`; `EnablementAnnulledUnitTests.anAnnulledUnitTakesNoMilestoneDocumentEquipmentNorKey` | ✅ |
| FLT-R90 | Cada hito que cambia deja una fila en la bitácora, sin motivo, con la marca de la unidad; también el primer registro de la placa. La edición de una llave deja una fila por campo. Desde el 04/11/2026, los valores de `cost` y `currency_id` de una instalación los ve en la bitácora solo quien tiene `stock.ver_costos` en la empresa y la marca de la fila; los demás ven que cambiaron, con `valuesWithheld` | §13.1, D-102, D-105, D-187; revisión del 04/11/2026 | Servicio: `MilestoneChangeLog`, `VehicleKeyCommandServiceImpl`. Desde `ad87b4d9`, `FleetConfidentialChangeLogFieldsAdapter` marca los dos campos por marca, y `ChangeLogAccessPolicy.seesValuesOf` (`shared`) lee la marca de la fila. Frontend: desde `c510e131`, la bitácora dice por qué retiene cada valor. Commits: backend `4c279cc3`, `1aaf61a8`, `d6bee720`, `5aefa129`, `afcea799`, `ad87b4d9`; frontend `c510e131` | `VehicleMilestonesIntegrationTests.eachMilestoneLeavesItsRowInTheChangeLog`; `VehicleKeyIntegrationTests.anEditLeavesARowPerField`; `VehicleEquipmentIntegrationTests.whoSeesCostsCorrectsTheCostWithItsRowInTheChangeLog`, la del costo, con los campos `cost` y `currency_id`, y `theChangeLogWithholdsTheCostOfAnInstallationFromWhoDoesNotSeeTheCostsOfItsBrand`; `ChangeLogAccessPolicyTests.theValuesOfAFieldMarkedByBrandTravelWithItsPermissionOnTheBrandOfTheRow` (`shared`); `change-logs-view.spec` ("says why each value is withheld by its field: the costs of a unit, a payment, or another reach (C6)") | ✅ |
| FLT-R91 | Un cambio del catálogo —la pantalla de fábrica o el combustible de un modelo, un tipo que pasa a bloquear— cambia el estado de habilitación en la lectura siguiente, sin escribir nada; las unidades ya en un estado con puerta no se mueven | §11.9, D-165 | Por construcción: el estado no se guarda (FLT-R58). Commits: backend `ba0ce3f5`; frontend `5f66d62e` | `VehicleEnablementListIntegrationTests.aModelThatLosesItsFactoryScreenAsksItsScreenOnTheNextRead`; `EnablementGateIntegrationTests.aStatusMarkedLaterHoldsTheMovesIntoIt`; `vehicle-enablement-view.spec` ("says a unit in a gated status that lost its enablement, which only a change of the catalog does, as its record says it (C3)", con ese nombre desde `ab67dce4`) | ✅ |
| FLT-R92 | Los permisos de la fase siguen la matriz de §5.6: siete códigos y diecisiete marcas | §5.6, D-139 | Base: V(n). Commits: backend `53521c1d`; frontend `508b371f`, `58f9c6b4` | `EnablementPermissionsMigrationTests.theSevenPermissionsAndTheirMarksFollowTheMatrix` (`iam`); `PermissionsMigrationTests.everyPermissionHasTheCategoryOfTheDictionary`, `theMatrixOfTheSevenRolesIsTheOneOfTheDictionary` (`iam`); `PermissionCatalogConsistencyTests` (`shared`); `permission-matrix-groups.spec` ("places habilitacion under Logística › Flota › Habilitación", "places the document type permissions on Tipos de documento") | ✅ |
| FLT-R93 | La semilla trae el camino «Pierde un requisito» (`PIERDE_REQUISITO`), de Disponible a En habilitación, con `stock.habilitar` y sin tolerancia: la salida de una unidad Disponible que pierde un requisito sin repuesto. Es un camino como los demás, que se edita y se cierra | D-184, D-19, D-100 | Base: la semilla de V(n+1). Frontend: el panel de equipamiento lo nombra cuando el estado retiene el equipo. Commits: backend `9ca62ea4`, `4a48376a`; frontend `2cf89e48`, `d2ad8cf4` | `EnablementGateMigrationTests.theSeedBringsThePathOutOfDisponibleWithItsPermission`; `EnablementGateIntegrationTests.aUnitLeavesDisponibleWithoutItsGpsThroughItsOwnPath` (sale por el camino, se le retira el GPS, se le instala otro y vuelve por «Habilitación completa»); `VehicleStatusCatalogTests.theSeededStatusesAndPathsAreReadByEveryRoleAndTheSeededPathsNameTheirPermissions`. La versión 2 ponía en `VehicleStatusTransitionTests` una prueba propia del taller ante este camino, que no se escribió: que el taller no lo recorre lo dan la matriz, porque `TALLER` no tiene `stock.habilitar` (`PermissionsMigrationTests.theMatrixOfTheSevenRolesIsTheOneOfTheDictionary`), y la regla de todo camino (`VehicleStatusTransitionTests.thePermissionOfAPathIsItsOwnAndStockCambiarEstadoOpensOnlyThePathsWithoutOne`); `change-vehicle-status-dialog.spec` ("says what «Pierde un requisito» is for once it is chosen (D-184)"); e2e ("holds the GPS of an available unit, which leaves by «Pierde un requisito», loses it, gets another and returns to Disponible") | ✅ |

### 5.2 `vehiclecatalog`

| ID | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| VCT-R28 | Lo que exige cada modelo se calcula una sola vez, en `vehiclecatalog`: los equipos activos que bloquean y aplican a su tipo, sin la pantalla cuando el modelo la trae de fábrica; se responde en lote | §7.5, §11.9, D-165 | Fachada: `fetchEnablementProfilesByModelIds`. Servicio: `ModelEnablementProfiles` (`application/internal/queryservices`). Commits: backend `dd67d588`; frontend `5f66d62e`, `d2ad8cf4` | `VehicleCatalogContextFacadeTests.theScreenIsRequiredOnlyOfACarWithoutFactoryScreen`, `aRetiredEquipmentOrApplicabilityIsNotRequired`, `theProfilesOfABatchAreReadOncePerClass`; `enablement-checklist.spec` ("says a screen is from the factory on a car that brings it"); `e2e/enablement.spec.ts` ("asks a car without a factory screen for the screen, and says the factory one of a model that brings it") | ✅ |
| VCT-R29 | Un modelo es a gas si su combustible es `gnv` o `glp`; vacío no lo es | §7.3, D-166, D-109 | Fachada: `runsOnGas`. Commits: backend `dd67d588` | `VehicleCatalogContextFacadeTests.aModelRunsOnGasOnlyWithGnvOrGlp` | ✅ |
| VCT-R30 | El código de un equipamiento queda fijo desde su primera instalación, que `fleet` avisa por la fachada en su transacción y `vehiclecatalog` guarda en `has_installations`; `GPS` y `PANTALLA`, desde la semilla | D-74, D-185, §2.4, §7.5 | Base: `equipment_types.has_installations` (V(n+4)). Dominio: `EquipmentType.isCodeLocked`. Fachada: `markEquipmentTypeInUse`. Recurso: `codeLocked`. Frontend: el diálogo bloquea el código. Commits: backend `eee48459`, `dd67d588`, `5aefa129` | `EquipmentTypeTests.anEquipmentTypeInUseHasItsCodeLocked`; `VehicleCatalogContextFacadeTests.theFirstInstallationMarksTheTypeAndMarkingAgainChangesNothing`; `EquipmentTypesControllerTests.theCodeOfATypeInUseIsLocked`; `VehicleEquipmentIntegrationTests.anInstallationLocksTheCodeOfItsEquipmentType` (`fleet`) | ✅ |
| VCT-R31 | La versión de lo que la habilitación nombra del catálogo —modelos, tipos, equipamiento y aplicabilidad— la da la fachada | D-128 | Fachada: `fetchEnablementDataVersion`. Commits: backend `dd67d588` | `VehicleCatalogContextFacadeTests.theEnablementVersionMovesWithAnEquipmentType` | ✅ |

Con VCT-R28 y VCT-R30, VCT-R17 y VCT-R19 pasaron de ⚠️ a ✅ en `contextos/vehiclecatalog.md` el 03/11/2026: la regla de la pantalla ya combina los dos datos, y el código de un equipo en uso queda fijo.

---

## 6. Pantallas

### 6.1 Catálogos › Tipos de documento

**Ruta** `/fleet/document-types`, como la deja `app-navigation.ts`; se abre con `catalogos.ver` (D-141). La opción del menú pierde `pending: true` y declara `access: CATALOG_READERS`, como Estados y motivos.

**Patrón:** lista con diálogo (`mm-data-manager` y `mm-create-and-edit-dialog`).
- **Columnas:** País, Código, Tipo de documento, Vence, Bloquea la disponibilidad, Visible para el cliente, Solo unidades a gas, Estado.
- **Filtros:** País (los países del catálogo; empieza en el de la sesión cuando la sesión tiene una sola empresa) y Estado. Busca por código o nombre.
- **Exportar:** descarga `tipos-de-documento-<fecha>.xlsx`.

**«Nuevo tipo de documento»** se ofrece con `catalogos.crear_tipos_documento`. El diálogo pide:
- País y Código, con la pista «Letras, números y guion bajo; no cambia cuando un documento lo usa.»;
- Nombre;
- las casillas «Vence», «Bloquea la disponibilidad», «Visible para el cliente» y «Solo unidades a gas (GNV o GLP)».

Al editar, el país se muestra fijo; el código se bloquea cuando `codeLocked` es verdadero, con la pista «TIVE lo leen las reglas del sistema.» o «Ya hay documentos de este tipo.». «Editar» va con `catalogos.editar_tipos_documento`, y «Activar» y «Desactivar» con `catalogos.desactivar_tipos_documento`. Desactivar pregunta antes: «{nombre} dejará de pedirse y de bloquear a las unidades. Los documentos ya cargados se conservan.»

Los rechazos van en el diálogo, con el texto de su código (sección 4.10).

### 6.2 Catálogos › Estados y motivos

- **Cada estado muestra si exige la habilitación:** una insignia «Exige habilitación» en su fila del panel de estados.
- **El diálogo del estado** suma la casilla «Exige la habilitación completa para entrar» con la pista «Solo se entra a mano con la unidad lista para entrega; el regreso automático no la exige.».
- **En `DISPONIBLE`** la casilla viene marcada y deshabilitada, con la pista «Disponible la exige siempre (D-170).».
- **En `VENDIDO`** viene marcada por la semilla y se puede desmarcar, con la pista «Mientras está vendida, la unidad conserva lo que la habilitó: la entrega exige todos los requisitos.» (D-184).
- **El alta de un estado no la muestra:** se marca después, al editarlo (sección 4.3).
- **El camino «Pierde un requisito»**, de Disponible a En habilitación, aparece entre los que salen de Disponible, con `stock.habilitar`, y se edita o se cierra como cualquiera (FLT-R93).

**Como se corrigió (04/11/2026)** (`7d5e5dce`, `660d15dc`, `891f3fa0`; sección 2.3):
- **El estado inicial y la puerta se excluyen** en el diálogo del estado: mientras la casilla de la habilitación está marcada, «Estado inicial» se retiene con «Un estado que exige la habilitación no puede ser el inicial: quite primero esa exigencia.»; mientras el estado es el inicial, la casilla se retiene con «El estado inicial no la exige: una unidad recién registrada no está habilitada. Quite primero la marca de estado inicial.». Una casilla ya marcada nunca se retiene, para que un estado guardado con las dos marcas se pueda corregir.
- **«Mover a» de un camino** muestra deshabilitados los estados que exigen la habilitación, con «Exige la habilitación completa» debajo de su nombre, salvo en un camino a Vendido, y la pista «Solo un camino a Vendido lleva la unidad sola a un estado que exige la habilitación completa (D-164, D-170).». Un camino cerrado no retiene ninguno: el backend lo revisa al reabrirlo.
- **Los rechazos** `VEHICLESTATUS_ENABLEMENT_INITIAL`, `VEHICLESTATUS_ENABLEMENT_AUTO_TARGET` (con los caminos que nombra) y `STATUSCHANGEREASON_AUTO_TARGET_GATED` dicen sus textos (sección 4.10); el del camino nombra el estado por su nombre del catálogo, no por su código.
- **Accesibilidad:** `e2e/accessibility.spec.ts` ("the status dialog holding the gate of the initial status, the gated statuses held in «Mover a», and a refused path have no WCAG A/AA violations (C4)"), en los tres anchos.

**Como se corrigió (04/11/2026, segunda revisión)** (frontend `46d36540`, `4b24c845`, `d184bceb`, `1d278ad7`):
- **El aviso de un camino reabierto desde Estados** y rechazado por `STATUSCHANGEREASON_AUTO_TARGET_GATED` decía un `{{ status }}` literal: el servicio de los errores de diálogo no pasaba los valores del texto. Ahora los pasa, y el aviso nombra el estado por su nombre del catálogo. Pruebas: `vehicle-statuses-view.spec` ("names the status of a path reopened here and refused, by the name of the catalog, never a placeholder"), `dialog-errors.spec` ("passes the values its text names, so the toast never reads a placeholder").
- **Dentro del diálogo del camino,** el mismo rechazo dice su propio texto, el que dice qué hacer en el diálogo, como su campo, y no el de la lista de afuera (`status-change-reason-form-dialog.spec`); la pasada de axe lo lee así (`d184bceb`).
- **El mock e2e de `PUT /vehicle-statuses`** revisa en el orden del backend y rechaza un destino automático solo cuando su marca se enciende, de falso a verdadero, como el backend (`1d278ad7`, `e2e/fleet.spec.ts`).

### 6.3 Logística › Habilitación

**Ruta** `/fleet/enablement`; se abre con `stock.ver`. La opción pierde `pending: true` y declara `access: anyOf(PermissionCodes.Stock.VIEW)`.

**Patrón:** módulo con dos pestañas, «Unidades», la de omisión, y «Vencimientos» (`?seccion=vencimientos`). Son dos etiquetas cortas, que caben a 375 px (skill `frontend-feature`, §6). La empresa se elige con `mm-company-filter` y queda en `?empresa=`; la lista espera a su empresa y deja de esperar si no hay ninguna, como Fondos.

**Unidades.**
- **Columnas:** Placa («Sin placa»), VIN, Marca y modelo, Estado del ciclo de vida, Ubicación, Ingreso, Habilitación y Próximo vencimiento.
- **Habilitación** es una insignia con el nombre del código y, debajo, lo que falta: «Sin GPS · Sin pantalla», «Falta: Tarjeta de identificación vehicular».
- **Filtros:** Habilitación (selección múltiple de los seis), Ubicación, Marca comercial y Modelo. Busca por VIN, placa o número de motor.
- **Orden:** por las filas de la habilitación y después por VIN.
- **Exportar:** descarga `habilitacion-<fecha>.xlsx`.
- **Cada fila:** «Ver habilitación» abre la página de la unidad (6.4).

**«Registrar ingreso» (en lote).** Con `habilitacion.registrar_hitos` en la empresa, la lista ofrece seleccionar filas: una casilla por fila «Fuera de tienda» y una en el encabezado para las de la página. Al seleccionar, una barra dice «{n} unidades seleccionadas» y ofrece «Registrar ingreso» y «Quitar la selección». El diálogo pide:
- la fecha, que propone hoy en la zona de la empresa;
- la ubicación, entre las activas de la empresa;
- y lista las unidades por VIN.

Guarda con `POST /vehicle-arrivals`; un rechazo `VEHICLE_ALREADY_ARRIVED` marca las filas que nombra su `context` y deja el diálogo abierto. Un lote de más de 100 no se ofrece: «Seleccione hasta 100 unidades.». Si la sesión solo actúa en una sede, la selección se ofrece pero el servidor decide (`403`): «Su rol no registra el ingreso de unidades sin ubicación: lo hace un rol de la empresa.». **Requiere la revisión 35 de DESIGN.md** (6.9).

**Vencimientos.**
- **Columnas:** Documento, Vence (dd/mm/aaaa) con «en n días» o «vencido hace n días» calculado en la zona de la empresa, Placa, VIN, Modelo y Ubicación.
- **Filtro:** «Vencen en los próximos 30 días», el de omisión, o «Vencidos», más Documento.
- **Cada fila:** abre la página de la unidad en su sección Documentos.

No se exporta (D-167 pide mostrarlos y filtrarlos).

**Como se construyó** (`538c1534`, `610efebd`, `9094fc55`):
- el filtro de empresa se muestra en las dos pestañas;
- los filtros Modelo y Documento leen sus opciones en una sola página de 200 (`VehicleCatalogFacade` para los modelos; los tipos de documento del país, aparte de la página del catálogo);
- el número que anuncia la barra de selección y el estado oculto de cada requisito quedan dentro de su lista, con `position: relative`, para que el texto oculto no estire el documento (`b1bb3b2d`, `8bb6dfcb`).

**Como se corrigió (04/11/2026)** (`ab67dce4`): el filtro Estado lee el catálogo de estados, que pide `catalogos.ver`, así que la lista lo muestra, y lee el catálogo, solo con ese permiso. Una sesión de `stock.ver` sola recibe la lista sin ese filtro, en vez de un error al leer el catálogo.

**Como se corrigió (04/11/2026, segunda revisión): los filtros de los catálogos** (frontend `d2ae9682`). Modelo y Documento también leen catálogos (`GET /vehicle-models` y los tipos de documento), que piden `catalogos.ver`, y a una sesión de `stock.ver` sola le quedaban vacíos sin decirlo. Ahora la lista los muestra, como Estado, solo con ese permiso. Pruebas: `enablement-view.spec` ("leaves out the filters whose catalogs the session does not read, never one silently empty (catalogos.ver)", "offers the filters of the catalogs to who reads them"); e2e ("offers the filters of the catalogs only to who reads them: Estado, Modelo and Documento wait for catalogos.ver").

**Como se corrigió (04/11/2026, segunda revisión): la barra con varios filtros** (pedido del responsable; frontend `0036e50c`, `f5425b24`; revisión 40 de `DESIGN.md`). Unidades y Vencimientos toman la variante `filterRows` de `mm-data-manager`:
- **Fila 1:** la búsqueda, de al menos 22rem, y al final «Exportar», delineado. Habilitación no tiene acción principal, así que termina en «Exportar».
- **Fila 2, Unidades:** Empresa (si la sesión llega a más de una), Marca comercial, Habilitación, Estado, Ubicación y Modelo, alineados al inicio. Habilitación, la selección múltiple de los seis códigos, empieza en «Todas»: elegir un código la suelta, y elegirla otra vez suelta los códigos. Desde `f5425b24` dice «Todas», y no «Todos los estados», para no repetir el filtro Estado que tiene al lado. Ubicación y Modelo, con «Todas las ubicaciones» y «Todos los modelos» primero.
- **Fila 2, Vencimientos:** Empresa (si la sesión llega a más de una), Vencimiento («Vencen en los próximos 30 días» o «Vencidos») y Documento.
- **Cada filtro muestra un valor** y nunca su etiqueta como texto de ayuda; **«Limpiar filtros»** (`filter_alt_off`) aparece mientras uno difiere de su valor inicial, y los vuelve atrás sin tocar la búsqueda ni la empresa.
- **En el teléfono:** la búsqueda, «Exportar» y «Filtros» con la cuenta de los activos, que despliega los filtros.

Pruebas: `enablement-view.spec` ("counts the filters set, and «Limpiar filtros» sets every one back to «Todos …» (DESIGN revision 40)", "shows «Todas» in Habilitación while no code is chosen, and lets go of it for a code"); las imágenes de referencia de `enablement` en las tres anchuras (`abb8a556`). El texto de la revisión 40 de `DESIGN.md` todavía dice «Todos los estados» para este filtro; la pantalla dice «Todas» desde `f5425b24`.

**Como se corrigió (04/11/2026, segunda revisión): el rechazo del ingreso en lote** (frontend `b698c392`). Con `VEHICLE_ALREADY_ARRIVED`, el diálogo marcaba las filas y pedía quitarlas de la selección a mano. Ahora las unidades que ingresaron mientras tanto salen solas de la selección, la lista se vuelve a leer para mostrar su ingreso, y el mismo diálogo registra las demás: «Algunas unidades ya tienen su ingreso: salieron de la selección. Registre el de las demás.». Pruebas: `vehicle-enablements.store.spec`, `register-arrival-dialog.spec`, e2e de `enablement.spec.ts`.

### 6.4 La habilitación de una unidad

**Ruta** `/fleet/enablement/:vehicleId`, debajo del módulo, con `stock.ver`. El encabezado lo pone la vista con `overridePageHeading`: la placa —o el VIN sin placa— sobre la marca, el modelo y el VIN, como la ficha de la unidad. Un enlace «Ver ficha de la unidad» lleva a `/fleet/vehicles/{id}`.

**Patrón:** página de un registro con paneles apilados, sin pestañas, porque son cinco secciones y no caben a 375 px. Cada panel se lee y se escribe por su ruta, y las acciones salen de `allowedActions` (sección 2.5).

1. **Habilitación.**
   - **La insignia del estado** y debajo la lista de requisitos, cada uno con su icono y su texto: «Ingreso físico», cada documento que bloquea, cada equipo que exige la unidad, «Placas físicas». Cumplido: icono de verificación y la fecha o el dato. Falta: icono de alerta y «Falta».
   - **Si el estado actual de la unidad exige la habilitación y no está completa**, una línea lo dice: «Está en Disponible y dejó de cumplir la habilitación.» Solo puede pasar por un cambio del catálogo (FLT-R91).
   - **Requiere la revisión 35 de DESIGN.md** (6.9).
2. **Hitos.**
   - **Los datos:** Ingreso, Ubicación, Inicio de inmatriculación, N.° de título, Fecha de título, Placa y Placas recibidas.
   - **Las acciones**, cada una por su `allowedActions`: «Registrar ingreso» o «Corregir ingreso», que abren el diálogo de 6.3 con una unidad y `PUT /arrival`; «Editar hitos», una sección editable del panel, registrada en `UnsavedWork`; «Registrar placa»; «Corregir placa», el diálogo de hoy.
   - **«Registrar placa» se muestra deshabilitado** con su razón mientras falta el título o la TIVE: «Registre el título y la tarjeta de identificación vehicular primero.».
   - **«Placas recibidas»** se pide solo con placa.
3. **Documentos.**
   - **Una fila por tipo que aplica** a la unidad, con su documento vigente: Documento, Emitido, Vence, Archivo («Ver»), Cargado por y cuándo. Un tipo sin documento dice «Sin cargar», y uno que bloquea lo marca.
   - **«Cargar documento»** abre un diálogo con el tipo, entre los que aplican, el archivo («PDF, PNG, JPEG o WebP, hasta 5 MB»), la emisión y el vencimiento, que se pide solo si el tipo vence.
   - **Un documento que reemplaza a otro** avisa antes de guardar: «El documento vigente pasará al historial.». Es una confirmación del reemplazo, que cambia lo que ven otros.
   - **«Ver historial»** de un tipo despliega sus documentos anteriores, con `GET /vehicles/{id}/documents?documentTypeId=`.
4. **Equipamiento.**
   - **Columnas:** Equipo, Serial, Desde, Hasta, Motivo del retiro, Instaló y Costo; el costo solo cuando viaja (FLT-R80).
   - **Primero, los equipos que aplican a la unidad:** uno que exige y no tiene instalado dice «Falta»; la pantalla de un auto que la trae de fábrica dice «De fábrica».
   - **«Instalar equipo»** abre un diálogo con el equipo, entre los que aplican, el serial —obligatorio si el equipo lo exige—, la fecha de instalación, que propone hoy en la zona de la empresa y no deja una anterior al ingreso, quién instaló, entre los colaboradores de la empresa, y el costo con su moneda, solo para quien ve los costos. Con la unidad sin ingreso, el botón se muestra deshabilitado: «Registre el ingreso primero.».
   - **Cada equipo abierto ofrece «Reemplazar» y «Retirar»** según su `allowedActions`. Reemplazar abre el mismo diálogo con el equipo fijo. Retirar pide la fecha y el motivo, Falla o Robo, y pregunta antes: «El equipo quedará retirado desde esa fecha.».
   - **En Disponible o Vendido, un equipo que la unidad exige no ofrece «Retirar»** (D-184), y su fila lo dice: en Disponible, «Para retirarlo sin repuesto, saque antes la unidad por «Pierde un requisito».», con un enlace que abre «Cambiar estado» de la ficha; en Vendido, «Reemplácelo: la unidad vendida conserva sus requisitos.».
   - **«Registrar costo»** se ofrece por `correctCost`, también en un equipo cerrado, a quien ve los costos (D-187).
5. **Llaves.**
   - **Columnas:** Etiqueta, Tipo, Serial, Dónde está y Caja.
   - **«Registrar llave»** y «Editar», con el mismo diálogo. La caja se pide solo si la llave está en la empresa.

**Como se construyó** (`5f66d62e`, `7354aee7`, `d2ad8cf4`, `38af9aaa`):
- «Editar hitos» es un componente propio, `vehicle-milestones-form`, que la vista pone en el panel Hitos;
- la corrección del costo es un modo del diálogo del equipo, `vehicle-equipment-dialog`, que pide solo el costo;
- las tablas de Documentos, Equipamiento y Llaves son tablas nativas con las clases de `mat-mdc`, y la de Documentos es un componente propio, `vehicle-documents-table` (`c94663bc`);
- el diálogo de ingreso no ofrece quitar el ingreso (`arrivalDate` en `null`): la API lo admite para corregir un error, y la pantalla no lo pide;
- al corregir el ingreso, el diálogo ofrece solo las ubicaciones activas, sin proponer la actual si se desactivó, porque el backend la rechaza (`d3a03220`);
- los nombres que el backend lee de otros contextos —la marca, el modelo, el tipo de vehículo, la marca comercial, el estado y el país de un tipo de documento— y el `updatedAt` que sirve de candado pueden llegar en `null`: la pantalla los lee así y muestra «—» (`3bd18f93`);
- una unidad que no está o no se ve dice «No encontramos esa unidad: no existe o no está a su alcance.», en palabras de una unidad y no del lote (`90a45e11`).

**Como se corrigió (04/11/2026):**
- **El estado de la unidad sale de su recurso** (`ab67dce4`; sección 4.5): la línea de la habilitación perdida y la nota del equipo retenido leen `currentStatusCode` y `currentStatusRequiresEnablement`, sin pedir `GET /vehicle-statuses`. La nota ofrece «Pierde un requisito» solo en `DISPONIBLE`, el único estado del que sale ese camino; en Vendido dice que se reemplace, y en otro estado con puerta, «Reemplácelo: mientras la unidad esté en {estado}, que exige la habilitación completa, no se retira sin repuesto.».
- **El costo de las instalaciones** se muestra según `allowedActions.viewPurchaseCost` de la unidad, que el backend calcula con `stock.ver_costos` en su empresa y su marca (`ab67dce4`, sección 4.7); antes bastaba el permiso en la empresa, y una sesión con los costos de otra marca veía la columna con «—».
- **El diálogo del equipo** pide la moneda de un costo escrito, en el alta y al registrar solo el costo, con «Elija la moneda del costo.», y nunca manda un costo sin ella; antes lo descartaba sin decirlo (`311dc136`).
- **Editar hitos y corregir el ingreso**, después de `VEHICLE_OUTDATED`, toman la unidad leída otra vez, sus valores y su `updatedAt`, y conservan el rechazo (`ea714437`; sección 4.11).

**Como se corrigió (04/11/2026, segunda revisión):**
- **«Registrar costo»** decidía si pedir el costo por el acceso a los costos de la página, que podía no haberse leído todavía, y podía mandar un costo vacío. Ahora lo decide el `correctCost` de la propia instalación, y nunca manda un costo vacío, tampoco en una instalación que no ofrece corregirlo (frontend `7cd9f40f`; `vehicle-equipment-dialog.spec`: "asks the cost of an installation that offers «Registrar costo» before the page read the access to costs", "never sends a cost of none, even for an installation that does not offer to correct it").
- **Corregir el ingreso de una unidad** después de `VEHICLE_OUTDATED`: si la unidad leída otra vez tiene una ubicación que se desactivó mientras tanto, el diálogo la quita y lo dice, y nunca la manda; el backend la rechazaría (frontend `5d6d34c5`; `register-arrival-dialog.spec`: "after VEHICLE_OUTDATED clears a location the unit read again holds that was deactivated, and says it").
- **Los comentarios y la descripción de la API** del costo de una instalación dicen `stock.ver_costos` en la empresa y en la marca de la unidad (D-102), como hace el código desde la primera revisión (backend `6e71a19d`).

### 6.5 La ficha de la unidad y el cambio de estado

- **Los datos de la ficha** suman «Habilitación», con la insignia de `enablement`.
- **El menú ⋮** suma «Ver habilitación», que abre la página de 6.4.
- **El diálogo «Cambiar estado»** ofrece los destinos de `walkableReasonIds` como hoy, y además los de `heldReasonIds`, deshabilitados y con la razón debajo: «Disponible — falta la tarjeta de identificación vehicular y el GPS.», armada con `enablement`. Un enlace «Ver habilitación» lleva a la página de la unidad.
- **Si el servidor responde `VEHICLE_ENABLEMENT_INCOMPLETE`** porque algo cambió mientras tanto, el diálogo muestra el texto compuesto (4.10) y vuelve a leer la unidad.
- **Desde Disponible**, el diálogo ofrece también «Pierde un requisito», hacia En habilitación, a quien tiene `stock.habilitar` donde está la unidad, como cualquier camino. Su motivo lleva la pista «Para retirar un requisito sin repuesto; vuelve a Disponible al completarse.».
- **La «Venta»** entra a Vendido, que también tiene puerta: si la unidad quedó incompleta por un cambio del catálogo, se muestra retenida como cualquier otro camino.
- **Lo que suma la fase 2 a la ficha** —«Anular unidad» por `allowedActions.annul` y la insignia de factura vigente— convive con esto sin cambios.

**Como se construyó** (`2cf89e48`, `b1f382e9`, `4e407db9`): el menú ⋮ de la ficha se muestra siempre, porque lleva «Ver habilitación», que pide solo `stock.ver`; el enlace «Ver habilitación» del diálogo sigue visible mientras el destino muestra su error; un cambio de estado de una unidad anulada dice que está anulada (`VEHICLE_ANNULLED`); y `?accion=cambiar-estado` dice que «Cambiar estado» no pudo abrirse cuando los caminos no se leyeron (`1a4799f9`).

**Como se corrigió (04/11/2026, segunda revisión)** (frontend `e18a6754`). Después de `VEHICLE_ENABLEMENT_INCOMPLETE`, el diálogo mostraba el texto y dejaba elegido el destino, que se podía volver a mandar. Ahora el destino que la puerta rechazó queda retenido desde entonces, como uno retenido desde el principio, con lo que le falta a la unidad debajo, y deja de estar elegido (`change-vehicle-status-dialog.spec`).

### 6.6 La ficha de la factura (depende del paso 5.11)

El botón «Registrar ingreso» de la ficha de la factura (hallazgo M5) se construye después de esa ficha. Es de `procurement` y usa `fleet` solo por su fachada:
- **`FleetFacade.loadArrivalRows(vehicleIds)`** lee, por `GET /vehicle-enablements?companyId=&vehicleIds=…`, cuáles de las unidades de la factura ya ingresaron;
- **`FleetFacade.registerArrivals(vehicleIds, arrivalDate, locationId): Promise<boolean>`** las registra por `POST /vehicle-arrivals`;
- **`arrivalsError` y `clearArrivalsError()`** son su estado propio de error, apartado del de las pantallas de `fleet` (skill `frontend-feature`, §4).

El diálogo es de `procurement` (`register-invoice-arrival-dialog`): lista las unidades que faltan y pide la fecha y la ubicación, que lee por `OrganizationFacade`. No es una pieza que `fleet` presta: la lista cerrada de piezas prestadas no cambia (frontend-rules 1.2). Los miembros de la fachada se agregan en el commit que los usa, porque un miembro sin consumidor no se agrega.

**Como se construyó** (`d502c0aa`):
- `FleetFacade.loadArrivalRows(companyId, vehicleIds)` recibe también la empresa, que `GET /vehicle-enablements` exige;
- `registerArrivals` devuelve un `boolean`, y el aviso cuenta las unidades de la selección;
- ante un rechazo, el diálogo vuelve a leer las unidades y se queda con las que siguen sin ingresar; si ninguna ingresó mientras tanto, muestra el rechazo de `fleet`;
- la ficha ofrece «Registrar ingreso» mientras una unidad de sus líneas vigentes no ingresó, a quien tiene `habilitacion.registrar_hitos` en su empresa, y no lo ofrece en una factura anulada.

**Como se corrigió (04/11/2026, segunda revisión)** (frontend `cb2b7efd`). Ante un rechazo, el diálogo volvía a leer las unidades y se quedaba con las que seguían sin ingresar, así que una unidad anulada mientras tanto desaparecía como si hubiera ingresado. Ahora distingue las dos: la anulada ya no se lista, y el diálogo lo dice (`register-invoice-arrival-dialog.spec`: "tells a unit annulled meanwhile from one that arrived: it is no longer listed").

### 6.7 Estados, 375 px y accesibilidad

- **Carga y error.** Cada lista y cada panel muestra su carga (`mm-skeleton`, DESIGN revisión 33) y, si no pudo leer, lo dice con «Reintentar»: «No se pudo cargar la habilitación de las unidades.», «No se pudieron cargar los documentos.»… Una lista vacía lo dice: «No hay unidades con los criterios seleccionados.»; «No hay documentos que venzan en los próximos 30 días.».
- **Nada se muestra como `null`:** un hito vacío es «—», un documento sin vencimiento «No vence», una unidad sin placa «Sin placa».
- **Fechas** en `dd/mm/aaaa`, en la zona del país de la empresa de la unidad, también si la empresa dejó la sesión (FLT-R41).
- **A 375 px:**
  - las listas de `mm-data-manager`, que no tiene modo de tarjetas, se desplazan de lado;
  - la casilla de selección queda en cada fila y la barra de selección se fija al pie (DESIGN.md, revisión 35);
  - los paneles de 6.4 se apilan y sus acciones bajan bajo el título;
  - el diálogo de ingreso lista las unidades con desplazamiento propio.
- **Accesibilidad:**
  - la insignia de habilitación lleva texto y no solo color;
  - cada requisito de la lista lleva su estado como texto accesible («Cumplido», «Falta»);
  - cada casilla de selección se llama «Seleccionar {VIN}», y la del encabezado «Seleccionar las unidades de la página»;
  - la barra de selección anuncia su número con `aria-live="polite"`;
  - un camino retenido es una opción deshabilitada con su razón en `aria-describedby`;
  - el campo de archivo nombra los tipos y el peso admitidos;
  - axe no encuentra violaciones (`accessibility.spec.ts`).
- **Salir con cambios** pregunta con `UnsavedChangesQuestion`, en los diálogos y en la sección «Editar hitos».

### 6.8 Frontend por capas

| Capa (`src/app/fleet`) | Archivos |
|---|---|
| `domain/model` | `vehicle-document-type.entity.ts`, `vehicle-enablement.entity.ts`, `enablement-status.ts` (los seis códigos como unión de tipos y su orden), `vehicle-enablement-row.entity.ts`, `vehicle-document.entity.ts`, `expiring-vehicle-document.entity.ts`, `vehicle-equipment.entity.ts`, `vehicle-key.entity.ts`, `vehicle-enablement-allowed-actions.ts`; comandos: `register-arrivals`, `correct-arrival`, `update-milestones`, `register-plate`, `upload-vehicle-document`, `install-vehicle-equipment`, `replace-vehicle-equipment`, `remove-vehicle-equipment`, `register-vehicle-key`, `update-vehicle-key`, `create-vehicle-document-type`, `update-vehicle-document-type`. `vehicle.entity.ts` suma `enablement`; `vehicle-allowed-actions.ts`, `heldReasonIds`; `vehicle-status.entity.ts`, `requiresEnablement` |
| `infrastructure/api` | `vehicle-document-types-api-endpoint.ts`, `vehicle-enablements-api-endpoint.ts` (la lista, la unidad, el ingreso, los hitos y la placa), `vehicle-documents-api-endpoint.ts` (con la subida multipart), `vehicle-equipments-api-endpoint.ts`, `vehicle-keys-api-endpoint.ts`; sus `-response.ts` y una petición por archivo; `fleet-api.ts` los agrupa |
| `infrastructure/assemblers` | Uno por recurso nuevo |
| `application` | `vehicle-document-types.store.ts`; `vehicle-enablements.store.ts` (la página de unidades, la de vencimientos, la selección y el lote, con `exporting`); `vehicle-enablement.store.ts` (la unidad de 6.4: su registro, sus hitos y su placa); `vehicle-documents.store.ts`, `vehicle-equipments.store.ts` y `vehicle-keys.store.ts`, uno por agregado. La vista de 6.4 inyecta los cuatro; ningún store inyecta otro. `fleet.facade.ts` suma los miembros de 6.6 solo con ese commit |
| `presentation/views` | `vehicle-document-types-view`, `enablement-view` (6.3), `vehicle-enablement-view` (6.4) |
| `presentation/components` | `vehicle-document-type-form-dialog`, `register-arrival-dialog`, `register-plate-dialog`, `upload-vehicle-document-dialog`, `vehicle-equipment-dialog` (instalar y reemplazar), `remove-vehicle-equipment-dialog`, `vehicle-key-form-dialog`, `enablement-status-badge`, `enablement-checklist` |
| `presentation/fleet.routes.ts` | `document-types` (`catalogos.ver`), `enablement` y `enablement/:vehicleId` (`stock.ver`) |

**Fuera de `fleet`.**
- **`iam/application/permission-codes.ts`** suma `Enablement` (`REGISTER_MILESTONES`, `MANAGE_DOCUMENTS`, `MANAGE_EQUIPMENTS`, `MANAGE_KEYS`) y los tres de `Catalogs`, y pone `'habilitacion'` después de `'stock'` en `PERMISSION_MODULE_ORDER`.
- **`iam/presentation/components/permission-matrix/permission-matrix-groups.ts`:** `placePermission` lleva el módulo `habilitacion` a Logística › Flota › Habilitación, y `catalogPlace` traduce el sufijo `tipos_documento` a la pantalla `tipos-de-documento` que ya declara `SCREEN_ORDER`. Hoy lo traduciría a `tipos-documento`, que no está en esa lista.
- **`app-shell/app-navigation.ts`:** las dos opciones sin `pending: true`.
- **`public/i18n/es.json`:** `enablement.*`, `vehicle-document-types.*`, los textos de los 45 códigos y `iam.permission-matrix.items.<código>` de los siete permisos.
- **Stores:** cada uno elige el texto del fallo por `ApiError.code`, con el genérico de la operación para uno desconocido.

**Como se construyó.** Además de los componentes de la tabla: `vehicle-milestones-form` (Editar hitos), `vehicle-documents-table` (`c94663bc`), `enablement-texts.ts` (lo que falta y lo que retiene un estado, dichos con palabras) y, en el sistema de diseño, `enablement-catalog-sample` (`a2ae587e`). La corrección del costo no tiene diálogo propio: es un modo de `vehicle-equipment-dialog`. Las líneas base de la matriz de permisos y de la ficha de un rol se regeneraron con los permisos nuevos (`58f9c6b4`), y `app.spec` abre Reportes como el módulo pendiente (`7e87c683`).

### 6.9 Revisiones de DESIGN.md

La **revisión 35** de DESIGN.md, escrita el 03/11/2026, antes de construir las pantallas de 6.3 y 6.4; donde esta lista y la revisión difieren en lo visual, manda la revisión. Por ejemplo: <ul><li>Documentos, Equipamiento y Llaves son tablas simples, que enmiendan la revisión 24;</li><li>«Ver ficha de la unidad» va en la cabecera del primer panel;</li><li>la razón de un destino retenido va en una segunda línea.</li></ul> La revisión 34 es la de la fase 2 (la tabla de líneas de la factura, su §6.8). Este documento no edita DESIGN.md. Cubre:
1. **La selección de filas en `mm-data-manager`:** casilla por fila y de página, la barra de selección con su acción, su forma a 375 px y su anuncio. Hoy `mm-data-manager` no selecciona filas.
2. **La lista de requisitos** de la habilitación: icono y texto por requisito, cumplido o falta, sin depender del color.
3. **Los tonos de las seis insignias de habilitación**, sobre los de `status-badge`: Fuera de tienda y Sin placas en neutro o advertencia, Sin documento y Sin equipo en advertencia, Listo para entrega en éxito, Entregado en información. Los fija la revisión.
4. **La página de un registro con varias colecciones apiladas** (6.4), que extiende la ficha con una colección de la revisión 31.
5. **La opción deshabilitada con su razón** dentro de un `mat-select`, para los caminos retenidos.

Catálogos › Tipos de documento y la pestaña Vencimientos usan patrones que ya existen (lista con diálogo, pestañas de un módulo) y no necesitan revisión.

**Como se construyó.** El commit 27 no tocó el texto de DESIGN.md: la revisión 35 ya estaba escrita. Lo que construyó es su catálogo en la vista del sistema de diseño (`a2ae587e`): las seis insignias, el panel «Habilitación» con su lista de requisitos y los documentos con un historial desplegado, que `fleet` presta por un token `DESIGN_SYSTEM_SAMPLES` que dan las rutas de desarrollo, después de que `c94663bc` sacó `vehicle-documents-table` a un componente propio.

---

## 7. Pruebas

Corren como manda el plan (§7): las que alcanza cada cambio, siempre `ModularityTests`, y con tablas nuevas las de Row Level Security y de filas que no se borran; las de integración sobre la base de verificación (`erp_mm_db_verify`, conectadas como la cuenta de la aplicación); las e2e en el puerto 4300 con el equipo enchufado; las suites completas antes de integrar en `develop`.

### 7.1 Los criterios de aceptación del plan

| Criterio (plan, fase 3) | Pruebas |
|---|---|
| Una prueba pura fija el orden de las seis filas | `EnablementPolicyTests.theSixRowsAreEvaluatedInTheirOrderAndTheFirstThatAppliesIsAnswered`: una tabla de casos con cada combinación de ingreso, documento, equipos, placas y entrega, sin Spring ni base |
| La entrada manual a Disponible se rechaza si falta la TIVE, la pantalla cuando corresponde, el GPS o las placas físicas, por «Habilitación completa» y por «Reacondicionada» | `EnablementGateIntegrationTests.aMissingRequirementRefusesTheManualMoveIntoDisponibleAlongBothPaths`: parametrizada, cuatro faltantes por dos caminos, ocho casos. La unidad de «Reacondicionada» recorre antes Vendido → Activo → Custodia → Puesta a punto, así que sale Entregado y también prueba D-183: completa, pasa; con un faltante, no. Cada caso exige el `422`, el código y lo que falta en `context`, y que la unidad y su historial no cambien. `aCompleteUnitEntersDisponibleAlongBothPaths` cierra el caso feliz. **Como se construyó:** el recorrido de la unidad reacondicionada y la revisión de que su historial no cambia llegaron en `805b12f8`, después del commit 21, junto con `aReconditionedUnitDeliveredBeforeReturnsToDisponible` |
| El regreso automático de Vendido no se bloquea | `EnablementGateIntegrationTests.theAutomaticReturnOfASoldUnitIsNeverHeld`: a una unidad Vendida —Vendido retiene sus requisitos, así que no se le puede retirar el GPS (D-184)— se le vuelve bloqueante un tipo de documento que no tiene, y el proceso de las 48 h la vuelve igual a Disponible con el motivo `REGRESO_AUTOMATICO`; la lista la muestra `SIN_DOCUMENTO` |
| Retirar el GPS mientras otra persona habilita la unidad no deja una unidad Disponible sin GPS | `EnablementConcurrencyTests.removingTheGpsWhileAnotherEnablesTheUnitNeverLeavesItDisponibleWithoutGps`: dos transacciones con un `CountDownLatch`, como `VehicleRacedIdentifierTests`, en los dos órdenes. **Como se construyó:** reutiliza la escena de `InvoiceRaces` (`procurement`), y sus unidades se confirman de verdad en `erp_mm_db_verify`; desde `805b12f8` usan su propio tipo de vehículo, `CARHAB`, cuyo GPS aplica solo mientras corren, y el catálogo queda como estaba. Si confirma primero el retiro, el cambio responde `VEHICLE_ENABLEMENT_INCOMPLETE`; si confirma primero el cambio, el retiro responde `VEHICLE_ENABLEMENT_HELD_BY_STATUS`. Al final, nunca una unidad Disponible sin un GPS abierto. `EnablementGateIntegrationTests.aUnitLeavesDisponibleWithoutItsGpsThroughItsOwnPath` prueba la salida que D-184 deja para el GPS que falla sin repuesto |
| No se instala nada antes del ingreso | `VehicleEquipmentIntegrationTests.nothingIsInstalledOnAUnitThatHasNotArrived` y `nothingIsInstalledOnADayBeforeTheArrival`; `VehicleArrivalIntegrationTests.anArrivalIsNotMovedAfterAnInstallation` |
| 80 motos ingresan en una operación y quedan con su ubicación | `VehicleArrivalBatchIntegrationTests.eightyUnitsArriveInOneOperationWithTheirLocation`: una sola petición, las 80 con fecha, ubicación y su fila de bitácora, en menos de 2 s en la base de verificación. `theStatementsDoNotGrowPerUnit` fija con las estadísticas de Hibernate que las sentencias que no son inserciones ni actualizaciones son las mismas con 1, 10 y 80 unidades; cada actualización y cada fila de la bitácora es su propia sentencia, porque no hay lotes de JDBC (sección 2.6, corregido el 04/11/2026). `aBatchWithOneUnitAlreadyArrivedWritesNothing` prueba el todo o nada |
| El filtro de vencimientos muestra exactamente los documentos que vencen en 30 días en la zona de la empresa | `ExpiringVehicleDocumentsIntegrationTests.theFilterShowsExactlyTheDocumentsThatExpireWithinThirtyDaysInTheZoneOfTheCompany`, con el reloj fijo en `2026-11-03T03:30:00Z`, que en Lima es el 02/11 a las 22:30. Entran los que vencen el 02/11 y el 01/12; quedan fuera el 02/12, el 01/11 —que sale con `expired=true`— y un documento no vigente que vence el 15/11 |

### 7.2 Pruebas del backend, por pieza

- **Dominio, sin Spring:**
  - `EnablementPolicyTests`, `RequirementHoldPolicyTests`;
  - `VehicleMilestonesTests` (`Vehicle`), `VehicleStatusTests`, y `VehicleTests` con la puerta;
  - `VehicleDocumentTypeTests`, `VehicleDocumentTests`, `VehicleEquipmentTests`, `VehicleKeyTests`;
  - `UploadedFileTypeTests` (`shared`).
- **Guardado:** `VehicleDocumentTypeAuditTests`, `VehicleEquipmentAuditTests` y `VehicleKeyAuditTests`, que guardan uno que ya existe y conservan su `public_id` y su momento de creación.
- **Integración**, con los roles de la matriz y la cuenta de la aplicación:
  - `EnablementGateIntegrationTests`, `EnablementGateMigrationTests`, `EnablementConcurrencyTests`;
  - `VehicleArrivalBatchIntegrationTests`, `VehicleArrivalIntegrationTests`, `VehicleMilestonesIntegrationTests`, `VehiclePlateRegistrationIntegrationTests`;
  - `VehicleDocumentIntegrationTests`, `VehicleDocumentCurrentIndexTests`, `ExpiringVehicleDocumentsIntegrationTests`;
  - `VehicleEquipmentIntegrationTests`, `VehicleEquipmentOverlapTests`, `VehicleEquipmentCurrencyUsageTests`;
  - `VehicleKeyIntegrationTests`, `VehicleKeyRacedTagTests`;
  - `VehicleDocumentTypeCatalogTests`;
  - `VehicleEnablementIntegrationTests`, `VehicleEnablementListIntegrationTests`, `VehicleEnablementProjectionTests`, `VehicleEnablementExportTests`;
  - `EnablementPermissionsIntegrationTests`, `EnablementAnnulledUnitTests`, `VehicleEnablementAllowedActionsTests`.
- **Construidas además** de las que nombraba la versión 2: `VehicleDocumentFilesTests` (el descarte de la llave, FLT-R71) y `VehicleEnablementFactsReaderTests` (la entrega y el documento vencido, FLT-R58), en `application/internal`.
- **Sumadas por la revisión del 04/11/2026:** `AutomaticReturnGateTests` (`345c1f75`; FLT-R55, FLT-R57), `EnablementDatesInLimaTests` (`6165ba5d`; FLT-R62, FLT-R64, FLT-R68, FLT-R76), con el reloj de `CompanyCalendar` fijo a las 20:00 de Lima del 03/11/2026, y, en `documents`, `KeptFileStoreUpsertTests` (`7ad20a4a`; FLT-R71). Las pruebas de una fecha de mañana en Lima dependían de la hora en que corría la suite: la fecha de mañana en Lima y la de hoy en UTC coinciden solo entre las 19:00 y la medianoche de Lima.
- **Sumadas por la segunda revisión del 04/11/2026:** `AutomaticReturnGateRuleTests` y dos casos de `EnablementGateIntegrationTests` (`a126216f`; FLT-R57); `VehicleMovementsBreachWarningTests` y `AutomaticReturnGateTests.theJobWarnsAboutAHeldUnitOnlyOnTheFirstRunAfterItsBreach` (`62708a60`; FLT-R57); `StatusCatalogLockTests` (`5bcd796b`; FLT-R55), con dos transacciones reales; `EnablementDatesInLimaTests.aCorrectedArrivalOfTomorrowInLimaIsRefused` (`22b16143`; FLT-R63); las dos consultas nuevas del inventario en `EnablementGateMigrationTests` (`ea5f342c`); y, en `documents`, `SpacesKeptFileStoreTests.aRowThatFailsToWriteLeavesNoObject`, `theObjectIsWrittenAfterItsRowAndItsFailureFailsThePut` (`5df515b8`) y `KeptFileStoreUpsertTests.aKeyAnEarlierTransactionCommittedIsNotWrittenFirstAgain` (`5f967ae3`; FLT-R71). `VehicleLockTests.setUp` arma su empresa y su catálogo solo dentro de la transacción de la prueba, que los revierte; su carrera, sin transacción, juega sobre la escena de las carreras (`607e6756`).
- **`vehiclecatalog`:** `VehicleCatalogContextFacadeTests` (los cuatro casos de VCT-R28 a VCT-R31), `EquipmentTypeTests`, `EquipmentTypesControllerTests`.
- **`iam`:** `EnablementPermissionsMigrationTests`, `RolePermissionsSystemScopeTests`.
- **Transversales:** `ModularityTests`, `RowLevelSecurityMigrationTests`, `FleetRowsAreNeverDeletedTests`, `FleetRowLevelSecurityTests`, `JdbcDataVersionsTests`, `PermissionCatalogConsistencyTests`, `NoDeleteEndpointsTests`, `NoRoleAnnotationTests`.
- **Las pruebas de una exportación** siguen la redirección y descargan de `/documents/{id}/file`, como `VehicleInventoryExportTests`.

### 7.3 Pruebas que cambian

**Las que mueven una unidad a Disponible o la venden por la API** dejan de pasar con la puerta: la «Venta» entra a Vendido, que también la tiene (D-184). Las que, estando la unidad en Disponible o en Vendido, le quitan un requisito tampoco pasan (sección 2.4). Entre las que nombran `DISPONIBLE`, `VENDIDO`, `HAB_COMPLETA`, `VENTA` o `REACONDICIONADA` hay que revisar cuáles lo hacen:
- `AutomaticReturnReasonTests`, `StatusToleranceBreachTaskTests`, `StatusChangeReasonRacedPathCodeTests`;
- `VehicleStatusTransitionTests`, `VehicleAllowedActionsTests`, `VehicleAllowedActionsTodayTests`;
- `VehiclesIntegrationTests`, `VehicleLockTests`, `VehicleOutOfReachTests`, `VehicleRacedIdentifierTests`, `VehicleRetiredReferencesTests`, `VehicleStatusCatalogTests`;
- `VehicleInventoryExportTests`;
- `FundsIntegrationTests`, `FundLockTests`, `VehicleFundAssignmentsIntegrationTests`, `VehicleFundAssignmentAuditTests`.

Las que la mueven preparan una unidad completa con un soporte de prueba nuevo, `EnablementFixtures.complete(vehicleId)`, que registra el ingreso, la TIVE, el título, la placa, las placas físicas y los equipos que exige su modelo. Ninguna prueba se debilita para que pase.

**Los demás cambios:**
- `VehicleTests` pasa `EntryGate.none()` a `transitionTo`.
- `VehicleStatusCatalogTests` manda `requiresEnablement` en cada `PUT`, y las que cuentan los estados marcados o los caminos de la semilla suman Vendido y `PIERDE_REQUISITO`.
- Con la fase 2 integrada, sus pruebas de las escrituras de una unidad anulada (su FLT-R49) suman las de la habilitación.
- `VehicleCatalogContextFacadeTests` y las de `EquipmentType` suman `has_installations`.

### 7.4 Pruebas del frontend

Una `*.spec.ts` junto a cada archivo nuevo. Las que fijan las reglas:
- **Stores:**
  - `vehicle-document-types.store.spec` ("says the code is taken in the country, or locked");
  - `vehicle-enablements.store.spec` ("asks the server for the enablement of a company, by its status codes", "registers the arrival of the selection and marks the units the server says arrived already", "asks the documents that expire within thirty days");
  - `vehicle-documents.store.spec` ("uploads a document as multipart and says why a file was refused").
- **Componentes y vistas:**
  - `vehicle-equipment-dialog.spec` ("asks the serial only of an equipment that requires it", "does not propose a day before the arrival, nor after today in the zone of the company", "shows the cost only to who sees costs");
  - `vehicle-enablement-view.spec` ("offers only what the record allows", "holds Registrar placa until the title and the TIVE are there", "says a screen is from the factory on a car that brings it, and a needed equipment missing");
  - `change-vehicle-status-dialog.spec` ("shows a held path disabled with what the unit lacks");
  - `enablement-status-badge.spec`, `enablement-checklist.spec` ("says each requirement as text, not by color alone");
  - `vehicle-status-form-dialog.spec` ("locks the gate of DISPONIBLE and says why").
- **Rutas y matriz:**
  - `fleet.routes.spec` ("opens the enablement with stock.ver", "opens the document types with catalogos.ver alone");
  - `permission-matrix-groups.spec` ("places habilitacion under Logística › Flota › Habilitación", "places the document type permissions on Tipos de documento").

**Como se construyó.** Los nombres de arriba son los reales desde la versión 3; dos de la versión 2 decían solo el comienzo del nombre (`vehicle-equipment-dialog.spec` y `vehicle-enablement-view.spec`). Las pruebas de cada regla están en §5, y las unitarias del frontend pasan todas: 3 007 de 3 007 (§7.7).

**Como se corrigió (04/11/2026).** La revisión sumó o renombró, con sus commits: `vehicle-status-form-dialog.spec`, `vehicle-statuses.store.spec`, `status-change-reason-form-dialog.spec` y `status-change-reasons.store.spec` (la puerta fuera del estado inicial y de los caminos que mueven solas las unidades); `vehicle-enablement-view.spec` (el estado y el costo desde el backend); `vehicle-equipment-dialog.spec` (la moneda del costo); `vehicle-milestones-form.spec` y `register-arrival-dialog.spec` (la unidad leída otra vez después de `VEHICLE_OUTDATED`); y `change-logs-view.spec` (por qué se retiene un valor). Cada una está en su regla de §5.

### 7.5 Extremo a extremo

**El simulador.** Los mocks de la habilitación van en `e2e/support/enablement-api.ts`, registrado por `mockApi`, o en la parte de `fleet` si el paso 5.11 ya dividió los mocks por contexto (hallazgo A23). Responden como el backend:
- las listas en sus órdenes;
- los códigos de la sección 4.10;
- el `context` del lote y de la puerta;
- el handler de `POST /vehicles/{id}/status-changes` aplica la puerta, en Disponible y en Vendido, con el estado de habilitación que el simulador lleva por unidad;
- y los estados y caminos de la semilla suman la marca de Vendido y el camino `PIERDE_REQUISITO`.

`e2e/fleet.spec.ts` y `accessibility.spec.ts`, que hoy mueven una unidad a Disponible o la venden, preparan antes su unidad completa en el simulador.

**`e2e/vehicle-document-types.spec.ts`:**
- lista los cuatro tipos de Perú y filtra por país;
- registra uno, rechaza un código repetido y corrige el nombre;
- bloquea el código de `TIVE`;
- desactiva preguntando antes;
- ofrece cada escritura solo con su permiso y abre la pantalla solo con `catalogos.ver`;
- exporta la lista con su filtro.

**`e2e/enablement.spec.ts`:**
- lista las unidades con su insignia y lo que falta, y las filtra por código;
- selecciona tres unidades Fuera de tienda y registra su ingreso con una ubicación; un lote con una ya ingresada marca esa fila y no guarda;
- en Vencimientos muestra los que vencen en 30 días en la zona de Lima, con el navegador en `Asia/Tokyo`, como la e2e de la fase 1;
- exporta la lista;
- en la página de una unidad: registra el ingreso; carga la TIVE, y un archivo `.txt` renombrado `.pdf` se rechaza por su contenido; registra el título y la placa, y la placa se ofrece solo con los dos; marca las placas recibidas; instala el GPS, y sin serial se rechaza; en un auto sin pantalla de fábrica pide la pantalla, y en uno con pantalla dice «De fábrica»; reemplaza el GPS; registra y edita una llave;
- en una unidad Disponible, «Retirar» el GPS no se ofrece y la fila nombra «Pierde un requisito»; la unidad sale por ese camino, se le retira el GPS, se le instala otro y vuelve a Disponible;
- en Estados y motivos, la casilla de Vendido se desmarca y la de Disponible no;
- en la ficha, «Cambiar estado» muestra Disponible retenido con lo que falta y, completada la unidad, la mueve.

Las capturas de referencia de `visual.spec.ts` se regeneran solo para las pantallas nuevas y para la ficha y el diálogo que cambian, en los tres anchos (375, 1024 y 1440).

**Como se construyó** (`a448a459`, `b3b9ab3b`, `ac78a97c`, `4aa0ff0b`):
- **El simulador** está en `e2e/support/mocks/enablement.ts`, junto a los mocks de cada contexto que dividió la fase 2. Después de la revisión del contrato responde con las formas y en el orden del backend (`b3b9ab3b`), y su semilla marca la pantalla como bloqueante, como el catálogo (`ac78a97c`).
- **`e2e/enablement.spec.ts`:** "lists the units with their badge and what each lacks, and filters them by the code"; "registers the arrival of three units out of the store at one location, and marks one that arrived meanwhile without saving"; "exports the list of the company"; con el navegador en Tokio, "shows the documents that expire within 30 days, counted in the zone of Lima"; "habilitates a unit on its page: arrival, card, title and plate, plates, GPS and a key"; "asks a car without a factory screen for the screen, and says the factory one of a model that brings it"; "holds the GPS of an available unit, which leaves by «Pierde un requisito», loses it, gets another and returns to Disponible"; "lets the mark of Vendido go, and keeps the one of Disponible"; y "shows Disponible held with what the unit lacks, and moves it once the unit is complete". El reemplazo del GPS y la edición de una llave van dentro de la página de la unidad.
- **`e2e/vehicle-document-types.spec.ts`:** "lists the four types of Peru and filters by country"; "registers a type, refuses a code repeated in its country, and corrects the name"; "keeps the code of TIVE fixed, which the rules read"; "deactivates a type after asking"; "offers each write only with its permission, and opens the screen only with catalogos.ver"; y "exports the list with its filter".
- **Accesibilidad**, en el grupo «enablement states» de `e2e/accessibility.spec.ts`: el ingreso en lote con una unidad marcada, los vencimientos, los diálogos de la página de una unidad, un destino retenido en «Cambiar estado», el diálogo de un tipo de documento y el ingreso desde la ficha de la factura; además, las pantallas nuevas de `e2e/screens.ts` pasan por axe y por el orden de sus encabezados.
- **Facturas:** `e2e/purchase-invoices.spec.ts` suma "registers from its record the arrival of the units of the invoice that have not arrived (finding M5)" y "offers no arrival from the record without habilitacion.registrar_hitos" (`d502c0aa`).
- `4aa0ff0b` hace esperar a `e2e/fleet.spec.ts` a que se cierre la lista de orígenes antes de volver a abrirla: fallaba a veces.

**Lo que cambió la segunda revisión (04/11/2026):**
- `e2e/enablement.spec.ts`: el ingreso en lote con una unidad ya ingresada la saca de la selección y registra las otras dos en el mismo diálogo (`b698c392`); y "offers the filters of the catalogs only to who reads them: Estado, Modelo and Documento wait for catalogos.ver" (`d2ae9682`).
- `e2e/fleet.spec.ts` y su mock: `PUT /vehicle-statuses` revisa en el orden del backend (`1d278ad7`).
- `e2e/accessibility.spec.ts`: el diálogo del camino lee su propio texto para un estado con puerta (`d184bceb`), y en el teléfono despliega «Filtros» antes del filtro de la marca (`045170c4`).
- Las imágenes de referencia de `enablement` y `vehicles`, en las tres anchuras, con la barra de la revisión 40 (`abb8a556`), miradas una por una.

### 7.6 Sondas en vivo

`e2e-live/enablement.live.spec.ts` corre contra el backend real, sobre la base de verificación mientras `erp_mm_db_live` no exista; solo el responsable crea bases. Recorre:
1. Entra con la cuenta de verificación y encuentra la empresa de Perú.
2. Prepara por la API lo de la corrida: una marca comercial, un modelo de moto y uno de auto sin pantalla de fábrica, un color y tres unidades.
3. En Habilitación selecciona las tres, Fuera de tienda, y registra su ingreso con una ubicación de la empresa; las tres pasan a Sin documento.
4. En la página de una unidad, carga un PDF como TIVE, registra el título, una placa única de la corrida y las placas recibidas, e instala el GPS con su serial; en el auto, también la pantalla.
5. La mueve a Disponible por «Habilitación completa» y la lista la muestra Listo para entrega. En la página de la unidad, «Retirar» el GPS no se ofrece; la saca por «Pierde un requisito», que la lleva a En habilitación, y la vuelve a Disponible.
6. Carga a otra un SOAT que vence en 10 días y lo encuentra en Vencimientos.
7. Registra una llave.
8. Exporta la lista.
9. Comprueba que ninguna petición usó `DELETE`.

`e2e-live/fleet.live.spec.ts` cambia su caso «moves the unit along a declared path of its status»: desde En habilitación el único camino lleva a Disponible, que la puerta retiene. La sonda comprueba que el diálogo lo muestra retenido con lo que falta, y el movimiento lo prueba la sonda nueva. `e2e-live/vehiclecatalog.live.spec.ts` comprueba que el código de un equipo propio queda fijo después de instalarlo.

El cierre registra cada corrida en `retroalimentacion/`, con el commit del backend y del frontend y el resultado de dos corridas seguidas, como las sondas del 31/10/2026.

**Pendiente al 03/11/2026.** Las sondas existen desde el frontend `4ca16d02` y no corrieron: necesitan la contraseña de la cuenta de verificación del responsable. Son:
- `e2e-live/enablement.live.spec.ts`, con los nueve casos de arriba, numerados como ellos, y uno más antes del último: "the record of a purchase invoice registers the arrival of its units that have not arrived";
- en `e2e-live/fleet.live.spec.ts`, el caso "offers Disponible held by the gate, with what the unit lacks, when it leaves En habilitación", que reemplaza al que movía la unidad;
- en `e2e-live/vehiclecatalog.live.spec.ts`, "an equipment type of its own keeps its code fixed from its first installation (D-185)".

Cuando corran, cada corrida va a `retroalimentacion/` como dice el párrafo anterior.

**Al 04/11/2026 siguen sin correr** (§10): la revisión no cambió las sondas, y `e2e-live/enablement.live.spec.ts` tiene diez casos.

### 7.7 Lo que corrió al integrar (03/11/2026)

- **Backend:** la suite completa, 3 254 pruebas, sobre `erp_mm_db_verify` con la cuenta `postgres`. Contra la línea base de `develop` en `16f69af3`, de 67 fallos que se esperan con el superusuario, fallan solo seis nuevas, todas porque el superusuario se salta Row Level Security y los permisos: `FleetRowsAreNeverDeletedTests.theApplicationCannotDeleteARowOf`, casos 8 a 11, y `FleetRowLevelSecurityTests.theScopeOfACompanySeesOnlyItsDocumentsEquipmentsAndKeys` y `aDocumentIsNeverEditedButForBeingRetired`. El CI corre la suite con las dos cuentas.
- **Frontend:** las unitarias, 3 007 de 3 007. Las e2e en escritorio de habilitación, flota, facturas de compra y tipos de documento, 63 de 63; las de accesibilidad «enablement states» en los tres anchos, 18 de 18; y todas las capturas de `visual.spec.ts` en los tres anchos.
- **Sentencias del ingreso en lote:** las 80 unidades ingresan en menos de 2 s, y las sentencias que no son inserciones ni actualizaciones son las mismas con 1, 10 y 80 unidades (`VehicleArrivalBatchIntegrationTests.theStatementsDoNotGrowPerUnit`). La versión 3 decía «las sentencias son las mismas»: las escrituras sí crecen, una por unidad y por fila de la bitácora (sección 2.6).
- **La revisión del contrato**, el 03/11/2026, del frontend contra el backend: el backend respondía lo que dice esta especificación, y todas las diferencias estaban en el frontend. Una rotura —corregir el ingreso ofrecía ubicaciones desactivadas, que el backend rechaza (`d3a03220`)—; cuatro textos: los de lo que retiene un estado (`c7d4470b`), el cambio de estado de una unidad anulada (`4e407db9`), la unidad que no se encuentra (`90a45e11`) y el rol que no activa ni desactiva tipos de documento, «Su rol no activa ni desactiva tipos de documento.» (`6e3e0125`); diez respuestas del simulador que no tenían la forma ni el orden del backend (`b3b9ab3b`); y los tipos de los campos que el backend puede dejar en `null` (`3bd18f93`). Todas quedaron corregidas en esos commits.

---

## 8. Secuencia de commits

Una rama `feature/fleet-enablement` desde `develop` en cada repositorio, en worktrees hermanos (plan §7). Cada commit compila y trae sus pruebas; se integra en `develop` solo por orden del responsable, porque `develop` despliega. Los números de migración se fijan al integrar: la fase que se integra primero toma V92 en adelante (sección 3.1).

| N.º | Paso | Repositorio | Commit | Qué lleva | Depende de | Hecho en |
|---|---|---|---|---|---|---|
| 1 | 8.0 | Documentación | `docs(api): write the contract of the vehicle enablement` | Las rutas, los cuerpos, los códigos y la excepción del costo en las convenciones (§2.2, §4.1, §4.2, §6.2, §7.2, §8.1); los flujos F8 a F12 y FLT-R55 a FLT-R93 en `contextos/fleet.md`, después de FLT-R48 a FLT-R54 de la fase 2; VCT-R28 a VCT-R31 en `contextos/vehiclecatalog.md` | La validación de la v1.38, con D-183 a D-189 | Documentación `462388f` |
| 2 | 8.1 | Backend | `feat(iam): seed the permissions of the vehicle enablement and of its document types` | V(n), las constantes de `PermissionCodes`, `EnablementPermissionsMigrationTests` | 1 | `53521c1d` |
| 3 | 8.2 | Backend | `feat(fleet): add the enablement milestones to vehicles and the gate flag to statuses` | V(n+1), con la marca de Disponible y de Vendido y el camino `PIERDE_REQUISITO`; los campos en `Vehicle`, `VehicleStatus`, sus entidades y ensambladores, sin comportamiento nuevo; `EnablementGateMigrationTests` | 2 | `9ca62ea4` |
| 4 | 8.2 | Backend | `feat(fleet): create the vehicle document types with the seed of Peru` | V(n+2); agregado, repositorio y persistencia | 3 | `3eb04534` |
| 5 | 8.2 | Backend | `feat(fleet): create the vehicle documents with one current document per type` | V(n+3); agregado y persistencia, con la sentencia que retira el vigente | 4 | `df5633db` |
| 6 | 8.2 | Backend | `feat(vehiclecatalog): record whether an equipment type has installations` | V(n+4); `EquipmentType.hasInstallations` e `isCodeLocked` | 1 | `eee48459` |
| 7 | 8.2 | Backend | `feat(fleet): create the vehicle equipments with their periods and costs` | V(n+5); agregado y persistencia, con la traducción de `23P01` | 3, 6 | `d43951eb` |
| 8 | 8.2 | Backend | `feat(fleet): create the vehicle keys` | V(n+6); agregado y persistencia, con la traducción de la etiqueta repetida | 3 | `8de8ab80` |
| 9 | 8.2 | Backend | `test(fleet): prove the isolation and the kept rows of the enablement tables` | `FleetRowLevelSecurityTests`, `FleetRowsAreNeverDeletedTests`, `RowLevelSecurityMigrationTests`, `JdbcDataVersionsTests` | 5, 7, 8 | `0b216d05` |
| 10 | 8.3 | Backend | `feat(vehiclecatalog): answer the enablement profile of a batch of models` | `fetchEnablementProfilesByModelIds`, `fetchEquipmentTypeReference…`, `fetchEnablementDataVersion`, VCT-R28, VCT-R29, VCT-R31 | 6 | `dd67d588`, junto con el 11: los dos tocan los mismos archivos de la fachada |
| 11 | 8.3 | Backend | `feat(vehiclecatalog): mark an equipment type in use from fleet` | `markEquipmentTypeInUse`, VCT-R30 | 10 | `dd67d588`, junto con el 10 |
| 12 | 8.4 | Backend | `refactor(shared): read the type of an uploaded file from its content in shared` | `UploadedFileType` en `shared`; la plantilla de compras pasa a usarlo. Si la fase 2 se integró antes, ya existe en `shared` y este commit no hace falta (sección 2.10) | — | No hizo falta: `UploadedFileType` ya estaba en `shared`, porque la fase 2 se integró antes |
| 13 | 8.5 | Backend | `feat(fleet): compute the enablement status of a unit with its six rows` | `EnablementPolicy`, sus valores, `VehicleEnablementFactsReader`, `GET /vehicles/{id}/enablement` en su forma mínima | 5, 7, 10 | `d8a4afd1`, que trae también `VehicleEnablementAllowedActions` (§2.5) |
| 14 | 8.4 | Backend | `feat(fleet): register the arrival of a unit or a batch with its location` | `VehicleArrivals`, `POST /vehicle-arrivals`, `PUT /vehicles/{id}/arrival`, `MilestoneChangeLog`, `VEHICLE_ARRIVED_WITHOUT_LOCATION` en `PUT /vehicles/{id}`; la guarda de la unidad anulada, solo si la fase 2 ya está | 13 | `4c279cc3` |
| 15 | 8.4 | Backend | `feat(fleet): record the registration milestones of a unit` | `PUT /vehicles/{id}/milestones` | 14 | `1aaf61a8` |
| 16 | 8.4 | Backend | `feat(fleet): register the first plate of a unit with its title and its TIVE` | `POST /vehicles/{id}/plate` | 15, 17 | `d6bee720`, después del 17, del que depende |
| 17 | 8.4 | Backend | `feat(fleet): maintain the vehicle document types by country` | Las cinco rutas, `codeLocked`, la exportación | 4 | `7dd80e2b`, antes del 16; trae `ExternalLocationService` y `"location::acl"` (§2.1) |
| 18 | 8.4 | Backend | `feat(fleet): upload the documents of a unit, one current per type` | Subida, lista, lectura, archivo y vencimientos; el descarte de una llave nueva al revertir, con `KeptFileStore.discardRolledBack` (si la fase 2 no está, lo crea con su forma) | 5, 12, 13, 17 | `fa9b7534` |
| 19 | 8.4 | Backend | `feat(fleet): install, replace and remove the equipment of a unit` | Las seis rutas, con la del costo, `ExternalLocationService`, `"location::acl"` en `package-info.java`, `VehicleEquipmentCurrencyUsageAdapter`, el costo con `stock.ver_costos` y la moneda funcional de `PurchasingCompany` (si la fase 2 no está, crea `fetchPurchasingCompaniesByIds` con su forma) | 7, 11, 13 | `5aefa129` |
| 20 | 8.4 | Backend | `feat(fleet): register the keys of a unit` | Las cuatro rutas | 8 | `afcea799` |
| 21 | 8.5 | Backend | `feat(fleet): gate the manual moves into a status that requires enablement` | `EntryGate`, `Vehicle.transitionTo`, `VehicleMovements`, `requiresEnablement` en el `PUT` de los estados, `EnablementFixtures` y las pruebas de 7.3 | 13 a 20 | `51070252` |
| 22 | 8.5 | Backend | `feat(fleet): hold the requirements of a unit in a gated status` | `RequirementHoldPolicy` en el retiro, el ingreso y los hitos, en Disponible y en Vendido; la salida por `PIERDE_REQUISITO` (FLT-R93); `EnablementConcurrencyTests` | 21 | `4a48376a` |
| 23 | 8.5 | Backend | `feat(fleet): offer the enablement actions and hold the gated paths` | `VehicleAllowedActions` con `heldReasonIds`, `enablement` en la unidad, `VehicleEnablementAllowedActions` | 22 | `118f3b11` |
| 24 | 8.5 | Backend | `feat(fleet): list, filter and export the enablement of the units of a company` | La lectura de 2.9, `VehicleEnablementProjectionTests`, la exportación | 23 | `ba0ce3f5` |
| 25 | 8.6 | Frontend | `feat(fleet): maintain the vehicle document types in the catalogs` | La pantalla de 6.1, la ruta, la opción sin `pending`, las constantes y la matriz | 17 | `508b371f`, y `7e87c683`: `app.spec` abre Reportes como el módulo pendiente |
| 26 | 8.6 | Frontend | `feat(fleet): mark the statuses that require enablement` | 6.2 | 21 | `1ee3f40d` |
| 27 | 8.7 | Frontend | `docs(design): select rows, list the requirements and stack the collections of a record` | La revisión 35 de DESIGN.md (6.9). Si ya está redactada cuando empieza el paso, el commit solo completa lo que le falte de 6.9 | — | Sin cambio de texto: la revisión 35 ya estaba escrita. Su catálogo en el sistema de diseño, `a2ae587e`, después de `c94663bc` (§6.9) |
| 28 | 8.7 | Frontend | `feat(fleet): list the enablement of the units with its status and its filters` | 6.3, pestaña Unidades, con la exportación; la opción sin `pending` | 24, 27 | `538c1534` |
| 29 | 8.7 | Frontend | `feat(fleet): list the documents that expire within thirty days` | Pestaña Vencimientos | 18, 28 | `610efebd` |
| 30 | 8.7 | Frontend | `feat(fleet): register the arrival of a selection of units with its location` | La selección y el diálogo de ingreso; `vehicle-form-dialog` sin «Sin ubicación» para una unidad ingresada | 14, 27, 28 | `9094fc55` |
| 31 | 8.8 | Frontend | `feat(fleet): open the enablement page of a unit with its milestones` | 6.4, paneles Habilitación e Hitos, con el ingreso, los hitos y la placa | 15, 16, 23, 27 | `5f66d62e` |
| 32 | 8.8 | Frontend | `feat(fleet): upload and replace the documents of a unit` | Panel Documentos | 18, 31 | `7354aee7` |
| 33 | 8.8 | Frontend | `feat(fleet): install, replace and remove the equipment of a unit` | Panel Equipamiento | 19, 31 | `d2ad8cf4` |
| 34 | 8.8 | Frontend | `feat(fleet): register and edit the keys of a unit` | Panel Llaves | 20, 31 | `38af9aaa` |
| 35 | 8.8 | Frontend | `feat(fleet): hold the paths into a gated status when the status of a unit changes` | 6.5: la insignia de la ficha, «Ver habilitación» y el diálogo | 23, 31 | `2cf89e48`, y `b1f382e9`: el enlace «Ver habilitación» sigue visible mientras el destino muestra su error |
| 36 | 8.8 | Frontend | `test(fleet): cover the enablement end to end` | Los mocks, `enablement.spec.ts`, `vehicle-document-types.spec.ts`, los cambios de `fleet.spec.ts`, accesibilidad y capturas | 25 a 35 | `a448a459` |
| 37 | 8.7 | Frontend | `feat(procurement): register the arrival of the units of an invoice` | 6.6, con los miembros de `FleetFacade` | 30 y el paso 5.11 de la fase 2 | `d502c0aa` |
| 38 | 8.9 | Frontend | `test(fleet): probe the enablement against the real backend` | `e2e-live/enablement.live.spec.ts` y los cambios de `fleet.live.spec.ts` y `vehiclecatalog.live.spec.ts` | 36 | `4ca16d02`; no corrió (§7.6) |
| 39 | 8.9 | Documentación | `docs(fleet): close the vehicle enablement in its context, P6 and the plan` | `contextos/fleet.md` y `vehiclecatalog.md` con los commits y las sondas, P6 y §3.2 de procesos, el plan, las sondas en `retroalimentacion/` | 38 | Varios commits de documentación del 03/11/2026, uno por documento: la especificación, las convenciones, los contextos y el mapa, los procesos, el plan y el orden |

**Lo que comparte con la fase 2**, que se resuelve al integrar la segunda de las dos:
- **Lo que es de la fase 2 y aquí solo se usa:** la anulación de la unidad (`Vehicle.annul`, `VEHICLE_ANNULLED` y su guarda en toda escritura, sus FLT-R48 y FLT-R49), `VEHICLE_FUND_ASSIGNMENT_OPEN` (D-182), el filtro «sin factura vigente» de Inventario (D-175), los puertos `UnitPurchases` y `UnitContracts` y `FleetContextFacade`. Esta fase no los crea ni los redefine; si se integra primero, no tiene la guarda (sección 2.6) ni el paquete `fleet/interfaces/acl`, que no usa (sección 2.1).
- **Los miembros aditivos que crea la que va primero, con la forma de la fase 2:** `fetchPurchasingCompaniesByIds` y `PurchasingCompany` (commit 19) y `KeptFileStore.discardRolledBack` (commit 18).
- **`UploadedFileType`:** decidido el 03/11/2026, vive en `shared/domain/model/valueobjects` para las dos fases. Lo crea la que se integre primero (aquí, el commit 12), y la otra lo reutiliza.
- **Los mocks divididos por contexto** (paso 5.11).
- **`VehicleCommandServiceImpl`, `VehicleMovements` y `VehicleAllowedActions`**, que tocan las dos: el conflicto se resuelve a mano, con el orden de la sección 2.3 —el permiso, la guarda de la anulada y después la puerta— y los campos de las dos en `allowedActions`.
- **Los números de las migraciones:** la que va primero toma V92 en adelante.
- **DESIGN.md:** la revisión 34 es de la fase 2 y la 35 de esta.

**Los commits que la tabla no tenía.**
- **Backend:** `805b12f8`, `test(fleet): walk the reconditioned unit through its delivery and leave the catalog of the races as it was`. La unidad de «Reacondicionada» recorre antes Vendido, Activo, Custodia y Puesta a punto, el historial no cambia con un rechazo, el regreso automático nombra `REGRESO_AUTOMATICO` y la lista dice lo que falta; las carreras usan su propio tipo de vehículo, `CARHAB`.
- **Frontend, las líneas base de los roles:** `58f9c6b4`, la matriz de permisos y la ficha de un rol con los permisos nuevos.
- **Frontend, correcciones:** `4f0ca720`, «Faltan las placas físicas.», el verbo que concuerda; `1a4799f9`, `?accion=cambiar-estado` dice cuándo no se leyeron los caminos; `ac78a97c`, la pantalla bloquea la disponibilidad en la semilla del simulador, como dice el diccionario en §7.5 (`blocks_availability`: «la pantalla solo en los modelos que la necesitan»); `b1bb3b2d` y `8bb6dfcb`, `position: relative` para el texto oculto que estiraba el documento.
- **Frontend, lo que dejó la revisión del contrato** (§7.7): `d3a03220`, `c7d4470b`, `4e407db9`, `90a45e11`, `6e3e0125`, `b3b9ab3b` y `3bd18f93`.
- **Frontend, sondas:** `10696eed` y `d126d8a2`, la sonda de las facturas de la fase 2, su commit F17 (especificación de facturas, §7.4); y `4aa0ff0b`, la e2e de flota que fallaba a veces.
- **Las correcciones de la revisión de punta a punta** (04/11/2026), en la rama `feature/review-findings` de cada repositorio: la tabla de §10.1 nombra el commit de cada una.

**Como se integró (03/11/2026).** La fase 3 se integró después de la fase 2, así que sus migraciones son V95 a V101 (§3.1). Merges en `develop`: backend `8bfdb938`, con la rama `feature/fleet-enablement` hasta `805b12f8`, 22 commits desde `16f69af3`; y frontend `2659126a`, con la rama hasta `4aa0ff0b`, 33 commits y el merge de `develop` `c2ba853f`, que trajo la fase 2 antes de la fila 37. Antes de integrar se revisó el contrato del frontend contra el del backend, y lo que encontró quedó corregido en el frontend (§7.7). Lo que el código hizo distinto está en cada sección, con la marca **Como se construyó**: entre otras cosas, las filas 10 y 11 en un solo commit, la 12 que no hizo falta, la 17 antes de la 16, `VehicleEnablementAllowedActions` con la 13, la lista de los tipos de documento ordenada en memoria, el orden de las revisiones del retiro de un equipo y del ingreso que se quita, los cuatro textos de `VEHICLE_ENABLEMENT_HELD_BY_STATUS` y, en el frontend, `vehicle-milestones-form`, la corrección del costo como modo del diálogo del equipo, los filtros que leen sus opciones en una página de 200, el filtro de empresa en las dos pestañas, las tablas nativas y la fachada del ingreso que recibe la empresa. La documentación de la fila 39 se hizo el mismo día. Faltan las sondas en vivo (§7.6). El texto de §2.7 del diccionario sobre los archivos de un documento, que era una propuesta para el responsable (§2.6), lo aprobó el responsable y lo aplicó la v1.39 el 04/11/2026 (documentación `7cc29f4`).

---

## 9. Preguntas de la versión 1, decididas

Las nueve preguntas de la versión 1 las decidió el 03/11/2026, por delegación del responsable, la coordinación del plan, y el diccionario v1.38 las registra como D-183 a D-189 (§2.7, §7.5, §8.2, §11.1, §11.3, §11.4, §11.9 y sus filas de §14). Esta especificación ya las aplica donde corresponde; no queda ninguna pregunta abierta.

| N.º | Pregunta | Decisión | Cómo queda y dónde se aplica |
|---|---|---|---|
| 1 | ¿Una unidad que ya se entregó una vez puede volver a Disponible por «Reacondicionada», si el diccionario exige Listo para entrega y ella sale Entregado? | **Decidida: D-183**, la opción recomendada | La puerta exige que no falte nada —ninguna de las filas 1 a 4—, esté Listo para entrega o Entregado; Entregado sigue siendo haber pasado a Activo alguna vez. Secciones 2.2 y 2.3, FLT-R56 y FLT-R58 |
| 2 | ¿En un estado con puerta se puede retirar sin reemplazo el GPS, o quitar las placas físicas o el ingreso? | **Decidida: D-184**, la opción recomendada y dos agregados | No: `VEHICLE_ENABLEMENT_HELD_BY_STATUS`, y el equipo se reemplaza. La semilla trae el camino «Pierde un requisito» (`PIERDE_REQUISITO`, de Disponible a En habilitación, con `stock.habilitar`, sin tolerancia), y marca también Vendido, que se puede desmarcar, porque la entrega exige todos los requisitos (guía, columna 22). El regreso automático sigue exento. Secciones 2.3, 2.4, 3.3 y 6.4, FLT-R55, FLT-R85 y FLT-R93 |
| 3 | ¿Se agrega `equipment_types.has_installations` a §7.5 para que `markEquipmentTypeInUse` fije el código de un equipo en uso? | **Decidida: D-185**, sí | La columna está en §7.5. Secciones 2.7 y 3.6, VCT-R30 |
| 4 | ¿Pueden ser futuras las fechas del ingreso, de los hitos, de la emisión de un documento y de una instalación, y se aplica la ventana de ±20 días y el «no antes de la factura» de la guía (columnas 19 y 21)? | **Decidida: D-186**, la opción recomendada | Ninguna es futura en la zona de la empresa, y una instalación no es anterior al ingreso; la ventana y la fecha de factura no se adoptan. Secciones 2.6, 4.5, 4.6 y 4.7, FLT-R62, FLT-R64, FLT-R68 y FLT-R76 |
| 5 | ¿Cómo queda el costo de una instalación que registra el taller, que no ve los costos, y cómo se corrige un costo mal escrito? | **Decidida: D-187**, la opción recomendada | Sin `stock.ver_costos`, el costo se guarda en cero en la moneda funcional; quien lo ve lo registra o lo corrige con `PUT /vehicle-equipments/{id}/cost`, con `habilitacion.gestionar_equipos`, y su fila en la bitácora. Secciones 4.7 y 6.4, FLT-R80 |
| 6 | ¿Un documento vigente de un tipo que bloquea y ya venció cuenta como vigente para la fila 2? | **Decidida: D-188**, la opción recomendada | Cuenta: el vencimiento se sigue en Vencimientos (D-167). Sección 2.2, FLT-R58 |
| 7 | ¿Una unidad ingresada puede quedar sin ubicación si se edita en Inventario? | **Decidida: D-188**, la opción recomendada | No: `vehicles_arrived_location_ck` y `422 VEHICLE_ARRIVED_WITHOUT_LOCATION` en `PUT /vehicles/{id}`. Secciones 3.3 y 4.4, FLT-R63 |
| 8 | ¿Cómo registra la placa una unidad de un país que no tiene el tipo `TIVE`? | **Decidida: D-188**, la opción recomendada | No la registra hasta decidirlo con la primera empresa de ese país, como sus documentos tributarios (D-52). Sección 4.5, FLT-R65 |
| 9 | ¿Se registra en §2.7 y §11.3 que los archivos de la habilitación se guardan en `kept_files`, con la empresa de su unidad? | **Decidida: D-189**, sí | §2.7 y §11.3 lo dicen: los bytes van a Spaces, y a la base solo donde no hay bucket. Sección 2.6, FLT-R71 |

---

## 10. Revisión de punta a punta (03 y 04/11/2026)

El responsable pidió una revisión crítica y profunda del plan v4, de punta a punta, y después ordenó «Resuelve todos los hallazgos como experto» (04/11/2026); la coordinación decidió por esa orden lo que la revisión dejó abierto. Cuatro revisores leyeron una fase cada uno, y la coordinación verificó en el código cada hallazgo clave. Las correcciones de toda la revisión se integraron en `develop` el 04/11/2026: backend `44136ae1` (`feature/review-findings`, 23 commits, `5c77bc8f` a `df6b7276`) y frontend `cc782649` (35 commits, `6386f2fa` a `1ef10642`). El [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) junta las cuatro fases; aquí va lo de la habilitación.

### 10.1 Lo que encontró en esta fase y cómo se cerró

La gravedad es la del efecto en el negocio: **alta** si deja de cumplirse una regla del diccionario o se detiene un despliegue; **media** si una pantalla falla para un rol, se pierde un cambio o se ve un dato reservado; **baja** si es una prueba, un texto o un caso raro.

| Hallazgo | Gravedad | Cómo se cerró | Commits | Pruebas |
|---|---|---|---|---|
| La puerta de Disponible se apagaba desde el catálogo: un camino que espera en otro estado y mueve sola la unidad a Disponible al vencerse, o un estado inicial marcado, la dejaban entrar sin la habilitación completa | Alta | Solo el regreso desde Vendido mueve solas unidades a un estado con puerta; tres rechazos nuevos del catálogo y una defensa en el proceso de las 48 h (§2.3, §4.3) | Backend `345c1f75`; frontend `7d5e5dce`, `660d15dc`, `891f3fa0`, `bdcf9e5e` | `AutomaticReturnGateTests` (4); los `*.spec` de los dos diálogos y sus stores; e2e y axe de flota (FLT-R55, FLT-R57) |
| V96 se detiene ante cualquier unidad con placa, y no se sabía si la base desplegada tenía alguna; `develop` se despliega en cada push | Alta | El inventario de solo lectura y su sección del README (§3.3) | Backend `7bc29d52` | `EnablementGateMigrationTests.theInventoryBeforeV96ChecksWhatV96StopsOn` |
| «Editar hitos» mandaba el `updatedAt` nuevo con los valores viejos, y un segundo «Guardar» pisaba el cambio de otra persona; el ingreso de una unidad, igual | Media | El formulario y el diálogo guardan con el `updatedAt` de sus valores y toman la unidad leída otra vez después de `VEHICLE_OUTDATED` (§4.11) | Frontend `ea714437` | `vehicle-milestones-form.spec`, `register-arrival-dialog.spec` (FLT-R63, FLT-R64) |
| La página y la lista de la habilitación, que se abren con `stock.ver`, leían el catálogo de estados, que pide `catalogos.ver`: a un rol sin él le fallaban la nota de la puerta, la del equipo retenido y el filtro de estado | Media | El recurso y la fila traen `currentStatusCode` y `currentStatusRequiresEnablement` (contrato C3); el filtro, solo con `catalogos.ver` (§4.5, §6.3, §6.4) | Backend `4aa250b9`; frontend `ab67dce4` | `VehicleEnablementIntegrationTests.theRecordNamesTheStatusTheMilestonesTheRequirementsAndTheActions`, `VehicleEnablementListIntegrationTests.theListFiltersByTheCodeOfItsStatus`; `vehicle-enablement-view.spec` (FLT-R61, FLT-R91) |
| El costo de una instalación se leía y se escribía con `stock.ver_costos` en la empresa, no en la marca de la unidad (D-102, D-187) | Media | `VehicleAccessPolicy.seesCosts`, como el costo de compra; la pantalla sigue `allowedActions.viewPurchaseCost` (§2.5, §4.7) | Backend `f5eacfe7`; frontend `ab67dce4` | `VehicleEquipmentIntegrationTests.theCostIsReadAndWrittenOnlyWithTheCostsOfTheBrandOfTheUnit`; `vehicle-enablement-view.spec` (FLT-R80) |
| La bitácora mostraba el costo y la moneda de una instalación a cualquiera con `bitacora.ver` | Media | Los dos campos son confidenciales por marca, con `valuesWithheld` (contrato C6; §4.7) | Backend `ad87b4d9`; frontend `c510e131` | `VehicleEquipmentIntegrationTests.theChangeLogWithholdsTheCostOfAnInstallationFromWhoDoesNotSeeTheCostsOfItsBrand`, `ChangeLogAccessPolicyTests.theValuesOfAFieldMarkedByBrandTravelWithItsPermissionOnTheBrandOfTheRow`; `change-logs-view.spec` (FLT-R90) |
| Para decidir el descarte de un documento, el backend abría la llave antes de guardarla: en Spaces, una descarga del objeto entero | Baja | `KeptFileStore.put` dice si escribió la llave primero, en la misma sentencia (§2.6). El mismo cambio corrige el descarte de las facturas y las notas, que sí borraba objetos de registros confirmados | Backend `7ad20a4a` | `VehicleDocumentFilesTests`, `KeptFileStoreUpsertTests` (FLT-R71) |
| El diálogo del equipo descartaba sin decirlo un costo escrito sin moneda | Baja | Pide la moneda (§6.4) | Frontend `311dc136` | `vehicle-equipment-dialog.spec` (FLT-R79) |
| La nota del equipo retenido ofrecía «Pierde un requisito» en cualquier estado con puerta, y ese camino sale solo de Disponible | Baja | La nota sigue el código del estado (§6.4) | Frontend `ab67dce4` | `vehicle-enablement-view.spec` (FLT-R85) |
| Las pruebas de una fecha de mañana en Lima dependían de la hora en que corría la suite | Baja | Pasan a `EnablementDatesInLimaTests`, con el reloj fijo (§7.2) | Backend `6165ba5d` | FLT-R62, FLT-R64, FLT-R68, FLT-R76 |
| Pruebas más débiles que sus reglas: el gas a GLP, el orden de los candados del lote, el rol de sede en hitos y documentos | Baja | Se reforzaron, sin cambiar el código | Backend `7b92a7de` | FLT-R70, FLT-R84, FLT-R86 |
| La especificación decía «lotes de JDBC» y que las sentencias del lote no crecían; el backend no configura lotes | Baja | Se corrige el texto, no el código: las 80 unidades cumplen la meta de 2 s (§2.6, §7) | Ninguno | `VehicleArrivalBatchIntegrationTests.theStatementsDoNotGrowPerUnit`, `eightyUnitsArriveInOneOperationWithTheirLocation` |

### 10.2 Lo que corrió el 04/11/2026

- **Backend:** la suite completa, 3 570 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`; fallan 78, todas de la línea base del superusuario: las de Row Level Security y de filas que no se borran, que necesitan las dos cuentas.
- **Frontend:** las unitarias, 3 285 en 275 archivos; stylelint sin avisos; la e2e completa de escritorio, 657 pasan, 3 se saltan y ninguna falla; axe sin violaciones en los tres anchos en las pantallas que cambiaron. No se regeneró ninguna captura de referencia.

### 10.3 Lo que queda

- **Las sondas en vivo** (§7.6): `e2e-live/enablement.live.spec.ts`, con diez casos, y los casos de habilitación de `fleet.live.spec.ts` y `vehiclecatalog.live.spec.ts`. Nunca corrieron: necesitan la contraseña de la cuenta de verificación del responsable.
- **El CI con las dos cuentas:** las pruebas de Row Level Security y de filas que no se borran de `vehicle_documents`, `vehicle_equipments` y `vehicle_keys` (FLT-R83) no han pasado en verde allí. Una sonda de SQL revertida, como `erp_mm_app` sobre `erp_mm_db_verify`, el 04/11/2026, confirmó en las once tablas nuevas de empresa del plan, estas tres incluidas, Row Level Security forzada, ningún permiso de `DELETE` y el aislamiento entre empresas.
- **El inventario antes de V96** (§3.3): lo corre el responsable contra la base desplegada, con la cuenta dueña, antes de que `develop` despliegue V96; cada unidad con placa que liste pide su decisión.
- ~~**Las propuestas para el diccionario**, que decide el responsable: §8.2, que ningún camino salvo el regreso desde Vendido mueva sola una unidad a un estado con puerta y que el estado inicial no se marque; §13.1, las marcas de la bitácora por marca, con el costo de una instalación; y §2.7, la llave de un documento y su descarte (§2.6).~~ Aprobadas por el responsable y aplicadas el 04/11/2026: documentación `7cc29f4`, diccionario v1.39.

### 10.4 La segunda revisión (04/11/2026)

Con la primera revisión integrada, cuatro revisores independientes volvieron a leer el plan v4 de punta a punta, y el responsable pidió tres cambios de pantalla: la barra de las listas con varios filtros, la importación de las líneas de una factura desde Excel y la tarjeta de datos en grupos. De ellos, a esta fase le toca la barra. Todo se integró en `develop` el 04/11/2026, sin migraciones: backend `851b4d09` (`feature/review-2-findings`, 20 commits, `a126216f` a `607e6756`) y frontend `3a49f141` (50 commits, `a8ccc252` a `c64888e9`). El [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) §2.11 junta las cuatro fases. La gravedad sigue la escala de §10.1.

| Hallazgo | Gravedad | Cómo se cerró | Commits | Pruebas |
|---|---|---|---|---|
| El regreso automático saltaba la puerta en todo camino que espera en Vendido, también con Vendido desmarcado: a una unidad vendida se le podía retirar el GPS, y a las 48 h volvía a Disponible sin él (rompe D-170) | Alta | El proceso no aplica la puerta solo mientras Vendido tenga la marca; desmarcado, deja en Vendido a la unidad a la que le falta algo. El catálogo mantiene la exención por el código (§2.3) | Backend `a126216f` | `AutomaticReturnGateRuleTests` (3); `EnablementGateIntegrationTests.anUnmarkedVendidoReturnsAloneOnlyAUnitThatLacksNothing`, `aMarkedVendidoReturnsTheSoldUnitAlone` (FLT-R57) |
| El inventario antes de V96 no decía si el estado inicial es Disponible o Vendido, ni los caminos `mover_sola` hacia ellos desde otro estado que Vendido, que V96 no detiene y la aplicación después rechaza | Baja | El inventario y su sección del README los listan, con qué hacer antes del merge (§3.3) | Backend `ea5f342c` | `EnablementGateMigrationTests` |
| El proceso de las 48 h escribía el mismo `WARN` en cada corrida por una unidad retenida, para siempre | Baja | Solo la primera corrida después del vencimiento avisa con `WARN`; las demás, en `DEBUG` (§2.3) | Backend `62708a60` | `VehicleMovementsBreachWarningTests`; `AutomaticReturnGateTests.theJobWarnsAboutAHeldUnitOnlyOnTheFirstRunAfterItsBreach` (FLT-R57) |
| Marcar un estado y guardar o reabrir un camino hacia él revisaban sin candado: dos pedidos a la vez podían dejar un estado marcado con un camino que mueve solas las unidades hacia él | Baja | El camino lee su destino `FOR SHARE`; marcar un estado, o cambiar su actividad, lo lee `FOR NO KEY UPDATE` antes de sus caminos (§2.3, §4.3) | Backend `5bcd796b` | `StatusCatalogLockTests` (FLT-R55) |
| En Spaces, el objeto de un documento se escribía antes que su fila, y una fila que fallaba lo dejaba sin fila y sin descarte | Baja | Primero la fila, después el objeto (§2.6) | Backend `5df515b8` | `SpacesKeptFileStoreTests.aRowThatFailsToWriteLeavesNoObject`, `theObjectIsWrittenAfterItsRowAndItsFailureFailsThePut` (FLT-R71) |
| Los filtros Modelo y Documento de Habilitación leen catálogos que piden `catalogos.ver`, y a un rol de `stock.ver` le quedaban vacíos sin decirlo | Baja | Se muestran solo con `catalogos.ver`, como Estado (§6.3) | Frontend `d2ae9682` | `enablement-view.spec`; e2e de `enablement.spec.ts` |
| Los diálogos de la habilitación: «Registrar costo» decidía por el acceso de la página, que podía no haberse leído, y podía mandar un costo vacío; el aviso de un camino reabierto y rechazado decía un `{{ status }}` literal; el diálogo del camino decía el texto de la lista; el ingreso en lote obligaba a quitar a mano las unidades ya ingresadas; «Cambiar estado» dejaba elegido un destino que la puerta rechazó; corregir el ingreso podía mandar una ubicación desactivada mientras tanto; el ingreso desde la factura confundía una unidad anulada con una ingresada | Baja | Cada uno en su pantalla (§6.2, §6.3, §6.4, §6.5, §6.6) | Frontend `7cd9f40f`, `46d36540`, `4b24c845`, `b698c392`, `e18a6754`, `5d6d34c5`, `cb2b7efd` | `vehicle-equipment-dialog.spec`, `vehicle-statuses-view.spec`, `dialog-errors.spec`, `status-change-reason-form-dialog.spec`, `vehicle-enablements.store.spec`, `register-arrival-dialog.spec`, `change-vehicle-status-dialog.spec`, `register-invoice-arrival-dialog.spec`; e2e de `enablement.spec.ts` |
| El mock e2e de `PUT /vehicle-statuses` no revisaba en el orden del backend | Baja | Revisa en ese orden y rechaza un destino automático solo cuando su marca se enciende | Frontend `1d278ad7`, `d184bceb` | `e2e/fleet.spec.ts`, `e2e/accessibility.spec.ts` |
| Comentarios y descripciones de la API viejos: el costo de una instalación por la empresa sola; el descarte de `VehicleDocumentFiles` por `open` | Baja | Dicen la empresa y la marca de la unidad (D-102) y la llave que el `put` insertó | Backend `6e71a19d` | — |
| Pruebas más débiles que sus reglas, o que dejaban datos: la corrección del ingreso con mañana en Lima no tenía prueba; `VehicleLockTests.setUp` confirmaba una empresa en un país de prueba nuevo en cada corrida | Baja | `EnablementDatesInLimaTests` con el reloj fijo; la empresa y el catálogo de la prueba, solo dentro de su transacción (§7.2) | Backend `22b16143`, `607e6756`, `dd225f1a` | `EnablementDatesInLimaTests.aCorrectedArrivalOfTomorrowInLimaIsRefused` (FLT-R63); `VehicleLockTests` |

**El pedido del responsable que toca esta fase:** la barra de una lista con varios filtros, en Unidades y en Vencimientos (revisión 40 de `DESIGN.md`; §6.3), con Habilitación en «Todas» (frontend `0036e50c`, `f5425b24`). Inventario la toma también, con Empresa, Marca comercial, Estado, Tolerancia y Factura vigente ([contextos/fleet.md](contextos/fleet.md)).

**Lo que se comprobó y estaba bien en esta fase:** los candados de la unidad en el ingreso en lote y en el cambio de estado —los del catálogo eran el hallazgo de arriba—, Row Level Security de las tablas nuevas, los permisos y las pruebas que citan §5 y §7.

**Las propuestas para el diccionario** que deja esta revisión, R23 (§8.2) y R24 (D-164): el regreso desde Vendido salta la puerta solo mientras Vendido la exige, y desmarcado deja en Vendido a una unidad incompleta (D-170, D-184). No se aplicaron: las decide el responsable, y el diccionario sigue diciendo la exención sin la condición.

### 10.5 Lo que corrió el 04/11/2026, después de la segunda revisión

- **Backend:** la suite completa dos veces seguidas, 3 619 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`: fallan 78, todas de la línea base del superusuario, y ninguna corrida gastó un país de prueba.
- **Frontend:** 3 327 pruebas unitarias en 275 archivos; stylelint limpio; la compilación, bien. La e2e completa de escritorio: 659 pasan, 3 se saltan y una falla de forma intermitente, la del armazón (`e2e/shell.spec.ts`), que medía la barra lateral y la superior a mitad de su transición y ya fallaba así en `develop`; `c64888e9` las mide cuando las dos se asientan, y pasó 116 de 116 en cuatro repeticiones, en escritorio y en tableta.
- **Las capturas** de las barras nuevas, a 1 440 y a 375 px, se revisaron a ojo.

Lo que queda abierto en esta fase sigue siendo lo de §10.3, más las propuestas R23 y R24.
