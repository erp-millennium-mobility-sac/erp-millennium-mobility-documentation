# Plan de migración a los contextos objetivo — ERP Millennium Mobility

**Estado:** propuesta para validar por el responsable, junto con el [mapa de contextos objetivo](mapa-de-contextos-objetivo-millennium-mobility.md). Versión 2, del 18/11/2026: incorpora la auditoría del mismo día, que verificó cada fase contra el código (§9).

**Fuente:** el inventario del código de `develop` (backend `ed5adc12` y frontend) del 18/11/2026, las reglas de arquitectura de las skills `backend-module`, `frontend-feature` y `architecture-review`, y *Domain-Driven Design* de Eric Evans (las referencias «cap. n» son capítulos del libro). Las rutas de archivo y los números de línea citados son los de ese `develop`: antes de ejecutar una fase se comprueban otra vez, porque el código sigue cambiando.

**Objetivo.** Llevar lo que hoy está construido a los contextos del mapa objetivo, por fases pequeñas que dejan el sistema funcionando al final de cada una, sin cambiar lo que hace el sistema salvo donde una fase lo dice. El bloque 12 (clientes, contratos, cobranza y traspasos) no se migra: se construye después, sobre la estructura que deja este plan, y aquí solo se prepara (fase 10).

---

## 1. Lo que no cambia en ninguna fase

Estas garantías hacen que cada fase sea un cambio de estructura y no de comportamiento:

| Qué | Por qué se mantiene |
|---|---|
| **Las tablas, sus nombres y sus migraciones** | Un contexto es dueño de sus tablas por el código que las escribe, no por su nombre. Ninguna fase crea, renombra ni borra una tabla. Las únicas migraciones nuevas son de la fase 8: un índice en su primera entrega y el retiro de dos columnas en la segunda, después de dejar de usarlas. `fleet_equipments` conserva su nombre aunque pase a `equipment` |
| **Row Level Security y `data_versions`** | Las políticas van por tabla y siguen iguales; solo cambia qué `Jdbc<Contexto>DataVersions` nombra cada tabla, y ninguno de un contexto de negocio nombra una tabla de otro |
| **Las rutas de la API** | Las rutas no llevan el contexto (convenciones §2.1). Una ruta puede pasar a otro módulo sin cambiar: `/vehicles/{id}/fund-assignments` a `financing`, `/vehicles/{id}/purchase-cost` a `procurement` y `/employees/{id}/termination` y `/reactivation` a `organization`. El controlador viejo sale en el mismo commit en que entra el nuevo: una ruta mapeada dos veces impide arrancar |
| **Los códigos de los permisos, de los errores, las claves de los mensajes y los nombres del esquema OpenAPI** | Son contratos por nombre con el catálogo, el frontend, los bundles y las pruebas `*OpenApiTests`. Renombrar permisos es una decisión aparte (mapa objetivo, §7) |
| **El orden de los candados** | Cada fase conserva las mismas sentencias de bloqueo en el mismo orden, aunque una pase por otra fachada; las fases que cruzan una frontera nueva lo dicen y lo prueban |
| **El comportamiento** | Cada fase es una refactorización: las mismas pruebas pasan antes y después. Las excepciones son el recurso de la unidad, que cambia solo si se elige la opción B de la fase 4 o la B de la fase 5, y la regla del inventario de equipos de la fase 2 §2.6, que se construye con el monitoreo y no en esta migración |

---

## 2. Cómo se trabaja

- **Pasos pequeños que dejan el sistema sano** (cap. 13, *Refactoring Toward Deeper Insight*). Una fase se integra completa o no se integra; nunca queda un contexto a medio mover en `develop`.
- **Una fase, una rama por repositorio:** `refactor/<fase>` en el backend y su pareja en el frontend si la fase lo toca, integradas juntas.
- **Primero se caracteriza.** Las fases que tocan reglas delicadas —la habilitación (2), la carga inicial (3), las cesiones (4), el costo de la unidad (5) y la salida del colaborador (7)— empiezan escribiendo pruebas de caracterización contra el código de hoy, en verde en `develop`, y terminan con esas mismas pruebas sin cambios.
- **Cada fachada nueva trae su prueba de contrato.** En el libro, la relación *Customer/Supplier* se sostiene con pruebas automáticas de aceptación en la frontera, que corre el proveedor (cap. 14). Con un solo equipo eso se traduce en una prueba por fachada o puerto nuevo, en el módulo que lo ofrece, que fija lo que el consumidor espera.
- **El CI corre en la rama antes de integrar.** Cada push a `develop` despliega `dev` sin esperar al CI, así que la suite completa se corre en la rama (*Run workflow*) y se integra solo en verde. `gh` no está autenticado en la máquina de trabajo: el resultado del CI lo mira el responsable en GitHub, o se autentica `gh` antes de empezar.
- **El mapa se actualiza en la misma fase** (cap. 14, *Context Map*). Lo que la fase mueve pasa del mapa objetivo al mapa vigente, a `contextos/` y al diccionario §3 en el mismo cambio.
- **Sin trabajo funcional en el área que se mueve mientras dura su fase.** Las fases 4 y 5 tocan los mismos archivos de `fleet` (`VehicleAllowedActions`, `VehicleAnnulments`, la ficha de la unidad) y van en ese orden, una después de la otra.
- **Las pruebas, a la medida del cambio:**
  - en local, en una base nueva de `erp_mm_owner` y nunca con el `.env` remoto: las pruebas del contexto que se mueve, las de quien lo consume, las de arquitectura (`ModularityTests`, `LayerDependencyTests`, `ForeignAclStaysInTranslationLayerTests`, `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`, `DeadlockRetryTests`, `JdbcDataVersionsTests`, `NoDeleteEndpointsTests`, `OwnErrorCodesHaveMessagesTests`) y `ErpMillenniumMobilityBackendApplicationTests`, que arranca el contexto y detecta los beans repetidos;
  - en el frontend, el build, las specs de lo movido y de lo que cambia, y `lint:styles`;
  - todo lo demás, en el CI. Las imágenes de referencia del e2e se nombran por la pantalla (`screen.name`), no por la ruta: **no se regeneran**, y una que cambie es una regresión.
- **Las skills son cuatro copias** —`.claude` y `.cursor` del backend y del frontend— y nombran los contextos, sus `allowedDependencies` y la lista de contextos de la búsqueda de la regla 1.2 de `architecture-review`. Cada fase las actualiza en las cuatro.

---

## 3. Las fases

| Fase | Qué | Contexto que forma o ajusta | Costo | Riesgo | Depende de |
|---|---|---|---|---|---|
| 0 | Preparación | — | 2–3 días | Bajo | — |
| 1 | La bitácora y las cargas salen de `shared` | `audit`, `imports` | 3–4 días | Bajo | 0 |
| 2 | Equipamiento | `equipment`, `fleet`, `vehiclecatalog`, `procurement` | 10–14 días | Medio | 0, y D-260 construida |
| 3 | La migración inicial como capa anticorrupción | `legacyimport`, `procurement` | 5–7 días | Medio | 1 |
| 4 | Financiamiento | `financing`, `fleet` | 9–12 días | Medio | 1, con `imports::acl` completo, y las decisiones de la fase 4 |
| 5 | El servicio publicado de compras | `procurement`, `fleet`, `reporting` | 6–9 días | Medio | 0 (crea `procurement::acl` si la fase 3 no lo hizo); después de la 4 |
| 6 | Las cuentas bancarias se quedan en `organization`, con un puerto por consumidor | `organization`, `procurement` | Menos de 1 día | Bajo | 0 |
| 7 | La salida y el regreso del colaborador, en `organization` | `organization`, `iam` | 6–9 días | **Alto** | 0 |
| 8 | Los catálogos preguntan en vez de recibir banderas | `vehiclecatalog`, `equipment`, `fleet` | 3–5 días, en dos entregas | Medio | 2 |
| 9 | Renombrar `location` a `reference` (opcional) | `reference` | 6–10 días | Bajo | Todas las demás |
| 10 | Preparar el bloque 12 | Diseño, sin código | 2–3 días | — | Respuestas del negocio |

Son entre 52 y 77 días de trabajo de una persona, sin calibrar. El orden va de lo barato y seguro a lo delicado, con una excepción deliberada: **equipamiento va segundo**, porque el próximo módulo anunciado al cliente es el monitoreo del GPS, y separarlo antes de construirlo cuesta mucho menos que después (cap. 14). D-260 no entra en la cuenta: es trabajo funcional, antes de la fase 2, de 6 a 9 días junto con las correcciones de la rama `refactor/remove-equipment-type-flags`.

---

## 4. Fase por fase

### Fase 0 — Preparación

- **Sincronizar las cuatro copias de las skills**, que ya divergen: las de `.cursor` son versiones viejas de `backend-module`, `architecture-rules`, `frontend-feature`, `frontend-rules` y `architecture-review`, y `frontend-feature` de `.claude` es más reciente en el frontend que en el backend. La fuente es `.claude` del frontend para `frontend-feature` y `.claude` del backend para las demás; las otras tres copias se sobrescriben con ellas.
- **Línea base:** la suite completa en verde con las dos cuentas de la base, en el backend y en el frontend.
- **Quitar de `procurement` la dependencia `iam::acl`,** que declara y no usa.
- **Aprobar** el mapa objetivo y este plan, y resolver con el responsable:
  1. **Los códigos de los permisos** no cambian en la migración (decisión 2 del mapa objetivo).
  2. **Renombrar `location`** a `reference`: sí o no (decisión 4).
  3. **Los requisitos de la habilitación:** resuelta (decisión 5 del mapa objetivo, D-260). Qué equipo es requisito pasa a una configuración de `fleet`; la aplicabilidad y la pantalla de fábrica se quedan con el catálogo de equipos. D-260 se construye como trabajo funcional antes de la fase 2 (fase 2, «Antes de empezar»).
  4. **La carga inicial:** si se ejecuta antes de la fase 3, la fase 3 se reemplaza por borrar su código después de la puesta en marcha.
  5. **Fase 4, opción A o B:** quién calcula `registerFundAssignment`. A, recomendada: `fleet` lo pregunta a `financing` por su puerto y el recurso de la unidad no cambia. B: sale del recurso de la unidad y lo responde un subrecurso de `financing`.
  6. **Fase 4, opción A1 o A2:** quién dibuja la pestaña de cesiones de la ficha de la unidad. A1, recomendada: `fleet`, alimentada por `FinancingFacade`. A2: una pieza prestada de `financing`, con su excepción en la regla 1.2 del frontend.
  7. **Fase 5, opción A o B:** quién es dueño de la brecha de días. A, recomendada: sigue en `fleet` y el recurso de la unidad no cambia. B: pasa a `procurement`, nunca a `reporting`, y cambia el recurso de la unidad.
  8. **El destino de un dispositivo al retirarlo de una unidad**, para cuando el monitoreo una el inventario a la instalación (fase 2 §2.6): la propuesta es `falla` a mantenimiento, `robo` a baja y `reemplazo` a disponible.

### Fase 1 — `audit` e `imports` salen de `shared`

**Por qué.** Un *Shared Kernel* debe ser pequeño y estable (cap. 14). La bitácora y el registro de las cargas tienen tabla, endpoint y permiso propios: son *Generic Subdomains* (cap. 15), como `documents` y `notifications`, que ya se separaron (D-125, D-143). El lector y el escritor de libros de `imports` son un *Cohesive Mechanism* (cap. 15) que viaja con su registro.

**Orden dentro de la fase.** Primero `audit`, porque `imports` depende de él; después `imports`; después las pruebas de arquitectura. Todo se integra junto.

**Backend — `audit`** (las 26 clases de `shared/changelog` pasan a `ky.millennium.mobility.erp.audit`, con los mismos subpaquetes):
- `package-info.java` con `@ApplicationModule(displayName = "Audit", allowedDependencies = {})` y `@NullMarked`: solo usa `shared`, que es abierto, como `documents`, que con `{}` lo usa en 16 clases.
- `audit/interfaces/acl`, con `@NamedInterface("acl")`, es su contrato público. Recibe los 8 tipos que usan los demás módulos:
  - `ChangeLogCommandService` (hoy en `application/commandservices`) y `RecordChangeLogCommand` (hoy en `domain/model/commands`), su parámetro, que nombran 21 archivos;
  - `ChangeLogHistoryLookup` (hoy en `application/queryservices`);
  - los puertos `AuditActorLookupPort` con `AuditActor`, `AuditCompanyLookupPort` y `ConfidentialChangeLogFieldsPort` con `ConfidentialChangeLogField` (hoy en `domain/ports` y `domain/model/valueobjects`). Salen hacia la fachada porque otros contextos los implementan o los leen.

  Ninguno lo usan `ChangeLogEntry` ni `ChangeLogRepository`: moverlos no hace que el dominio importe `interfaces`.
- Se quedan internos los demás. `ChangeLogCommandServiceImpl` sigue en `application/internal/commandservices`, envuelto por `DeadlockRetry`.
- `ChangeLogsController` se mueve con la misma ruta (`GET /api/v1/change-logs`), el mismo permiso (`PermissionCodes.ChangeLog.VIEW`, `bitacora.ver`) y su `@Tag`.
- **Consumidores que cambian el paquete.** Se reemplaza el nombre completo en todo el código, no solo las líneas `import`:
  - `procurement` (15):
    - en `application/internal/commandservices`: `CreditNoteAnnulments`, `CreditNoteCommandServiceImpl`, `OrderPaymentTransfers`, `PurchaseInvoiceAnnulments`, `PurchaseInvoiceCommandServiceImpl`, `PurchaseOrderTemplateChanges`, `SupplierCommandServiceImpl`, `SupplierPaymentAnnulments`, `SupplierPaymentRequests` y `SupplierPaymentVoucherReplacements`;
    - en `application/internal/queryservices`: `CreditNoteViewFactory`, `InvoicePaymentPositions`, `PurchaseInvoiceViewFactory` (incluido el nombre completo de `AuditActor` de la línea 320) y `SupplierPaymentViewFactory`;
    - en `infrastructure/changelog`: `ProcurementConfidentialChangeLogFieldsAdapter`.
  - `fleet` (6): `FleetImportCommandServiceImpl`, `MilestoneChangeLog`, `VehicleFundAssignmentCommandServiceImpl`, `VehicleIdentifierCorrections`, `VehicleTransfers` y `FleetConfidentialChangeLogFieldsAdapter`.
  - `location` (2): `ExchangeRateCommandServiceImpl`, que escribe la corrección de las tasas de cambio, y `ExchangeRateViewFactory`, que lee quién corrigió por `AuditActorLookupPort`.
  - `iam` (1): `IamAuditActorLookupAdapter`, el adaptador de `AuditActorLookupPort`.
  - `organization` (1): `OrganizationAuditCompanyLookupAdapter`, el de `AuditCompanyLookupPort`.
  - `imports` (2): `ImportBatchCommandServiceImpl` (`AuditCompanyLookupPort`) e `ImportBatchQueryServiceImpl` (`AuditActorLookupPort` y `AuditCompanyLookupPort`). Son las dos clases de `shared` que nombran la bitácora, y por ellas `imports` depende de `audit::acl`.
- **Nota de diseño.** Los dos puertos de búsqueda —el nombre de una cuenta y la clave de una empresa— no son de la bitácora. Los implementan `iam` y `organization`, y los leen también `location`, `procurement` e `imports`. Se quedan en `audit::acl`, porque llevarlos a `shared` contradiría la regla 6.1 y el objetivo de esta fase, y el mapa vigente lo registra.

**Backend — `imports`** (las 43 clases de `shared/importbatches` pasan a `ky.millennium.mobility.erp.imports`, más el adaptador que hoy está en `documents`):
- `package-info.java` con `@ApplicationModule(displayName = "Imports", allowedDependencies = {"audit::acl", "documents::acl"})` y `@NullMarked`.
- **`ImportBatchFileStoreAdapter` sale de `documents`** (`documents/application/internal/storage`) y pasa a `imports/infrastructure/storage`, sobre `KeptFileStore` de `documents::acl`. `KeptFileStore` ya ofrece lo que pide el puerto: `put`, con su respuesta de «lo escribí primero»; `open`; y `discardRolledBack`.
  - El descarte tras un rollback sigue en `ImportBatchCommandServiceImpl.keep` y no cambia.
  - El paquete vacío de `documents` se borra, y `documents` sigue en `{}`: si el adaptador se quedara, `documents` tendría que depender de `imports`.
  - Se reescribe el Javadoc de `ImportBatchFileStore`, que habla de `shared` y del ciclo de D-189.
- **`imports::acl`.** Spring Modulith solo expone de un paquete anotado sus propias clases públicas, sin subpaquetes ni los tipos de sus firmas. Por eso se expone exactamente lo que usan `fleet` y `procurement`, 25 tipos:
  - **El registro de la carga:** `ImportBatchCommandService` con `RegisterImportBatchCommand` y `RegisteredImportBatch`; `ImportBatchQueryService` (de él se usan `handle(GetImportBatchByPublicIdQuery)` y `fetchReferencesByIds`) con `GetImportBatchByPublicIdQuery`, `ImportBatchView` e `ImportBatchReference`; `ImportOutcome`; `ImportLocks`; `ImportType` e `ImportCounts`.
  - **Lo interno que hoy usan los consumidores:** `ImportBatchErrors.conflict()` e `ImportBatchAccessPolicy.reads(…)`. Se publican en `imports/interfaces/acl` como `ImportRefusals.conflict()` e `ImportReading.reads(AccessScope, Long)`, que delegan en los internos; los consumidores dejan de nombrar `application.internal`.
  - **El libro:** el paquete `application/spreadsheets` entero, con sus 11 tipos: `ImportWorkbookReader`, `ImportWorkbook`, `ImportRow`, `ImportCell`, `ImportCellException`, `ImportSheetSpec`, `ImportReportWriter`, `ImportReport` (con `ExtraSheet`), `ImportRowFailure`, `ImportSummaries` e `ImportFailureMessages`. Su `package-info.java` lleva `@NamedInterface("acl")`.
  - **La respuesta de una carga:** `ImportResultResource`, `ImportBatchResource` e `ImportBatchResourceFromViewAssembler`, que usan `FleetImportsController` e `InitialMigrationImportsController`. Siguen siendo una sola clase cada una, para que el esquema OpenAPI `ImportResult` no se duplique.
  - **El mecanismo:** el dominio usa `ImportType`, `ImportCounts`, `RegisterImportBatchCommand` y `GetImportBatchByPublicIdQuery` (`ImportBatch`, `ImportBatchRepository`), así que no se mudan a `interfaces`. Esos tipos, y los demás que no están en `spreadsheets`, llevan `@NamedInterface("acl")` a nivel de tipo, sin cambiar de paquete. Modulith junta todas las declaraciones `acl`, y `ModularityTests` lo comprueba.
- `ImportBatchesController` se mueve con sus cuatro rutas (`GET /api/v1/imports`, `/{id}`, `/{id}/error-report` y `/{id}/file`), su permiso (`PermissionCodes.Enablement.VIEW_IMPORTS`, `habilitacion.ver_cargas`) y su `@Tag`.
- **Consumidores:**
  - `fleet` (5): `FleetImportCommandService`, `FleetImportCommandServiceImpl`, `VehicleFundAssignmentViewFactory`, `VehicleTaxDeclarationQueryServiceImpl` y `FleetImportsController`;
  - `procurement` (5, todos de la carga inicial, que sale en la fase 3): `InitialMigrationImportCommandService`, `InitialMigrationImportCommandServiceImpl`, `InitialMigrationRun`, `StockRowMapper` e `InitialMigrationImportsController`.

**`allowedDependencies` después de la fase:**

| Módulo | Agrega |
|---|---|
| `audit` | Nuevo: `{}` |
| `imports` | Nuevo: `{"audit::acl", "documents::acl"}` |
| `fleet` | `audit::acl`, `imports::acl` |
| `procurement` | `audit::acl`, `imports::acl` (`imports::acl` sale en la fase 3) |
| `location` | `audit::acl` |
| `iam`, `organization` | `audit::acl` (implementan sus puertos) |
| `documents` | Queda en `{}` |

No hay ciclos: `audit` no depende de nadie, e `imports` solo de `audit` y `documents`.

**Pruebas.**
- **Se mueven**, espejando los paquetes:
  - las 8 de la bitácora: `ChangeLogCommandServiceTests`, `ChangeLogAccessPolicyTests`, `ChangeLogHistoryLookupTests`, `ChangeLogEntryTests`, `ChangeLogRowLevelSecurityTests`, `ChangeLogsControllerTenantTests`, `ChangeLogsPolicyTests` y `ChangeLogsSearchFilteringTests`;
  - las 4 de las cargas: `FileLoadsMigrationTests`, `ImportCellTests`, `ImportReportWriterTests` e `ImportWorkbookReaderTests`;
  - el fixture `XlsxFixtures`, que pasa a `ky.millennium.mobility.erp.imports.XlsxFixtures`. El recurso `/imports/stock-template-headers.xlsx` no cambia.
- **Cambian el paquete, sin moverse:**
  - de la bitácora: `ExchangeRateQueryServiceImplTests` (`location`); `ConceptLinesTests`, `CreditNoteAnnulmentsTests`, `PurchaseInvoiceAnnulmentsTests` y `ReactivationCompanyTests`, con su nombre completo de la línea 91 (`procurement`);
  - de las cargas: `FleetImportsIntegrationTests` (`fleet`); `StockRowMapperTests`, `InitialMigrationIntegrationTests` y `StockWorkbook` (`procurement`).
- **`ForeignAclStaysInTranslationLayerTests`:** `PLATFORM_CONTRACTS = Set.of("documents", "notifications", "audit", "imports")`, y su Javadoc. Sin ese cambio fallan todas las clases consumidoras, y también `IamAuditActorLookupAdapter`, que nombra `AuditActor`, un tipo que no implementa.
- **Se corren sin cambios:**
  - `ModularityTests`;
  - `LayerDependencyTests`, que desde ahora también revisa estos dos módulos y pasa, porque ninguna clase de `application` importa `infrastructure`;
  - `DeadlockRetryTests`, porque el pointcut no nombra paquetes;
  - `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`, `JdbcDataVersionsTests` y `NoDeleteEndpointsTests`;
  - `RowLevelSecurityMigrationTests` y `HandOverDatabaseScriptTests`, que solo nombran tablas;
  - `OwnErrorCodesHaveMessagesTests` y `ErpMillenniumMobilityBackendApplicationTests`.
- **No cambian** Flyway (`V115__create_shared_import_batches_schema.sql` conserva su nombre), `hand-over-database.sql`, `application*.yaml`, los `messages*.properties` ni la configuración de OpenAPI: ninguno nombra los paquetes.

**Frontend.** Nada: `change-log` e `imports` ya son carpetas propias, y ninguna ruta cambia.

**Documentación.**
- El mapa vigente, con `audit` e `imports` como módulos y la arista `imports → audit::acl`.
- El diccionario: §3 y §11.8, donde las reglas SHR-R01 a SHR-R13 conservan su número aunque ahora sean de `imports`.
- `contextos/README.md`, y `contextos/audit.md` e `contextos/imports.md`, con el patrón de `contextos/notifications.md`.
- El `README.md` del backend: el árbol de paquetes, líneas 333 a 338.
- Las skills, en sus cuatro copias (`.claude` y `.cursor` del backend y del frontend):
  - `backend-module/SKILL.md`: §1 («La bitácora de cambios» y «Las cargas y los reportes») y §2 (la lista de `allowedDependencies`);
  - `references/architecture-rules.md`: §1.1 (las excepciones de `ForeignAclStaysInTranslationLayerTests`) y §1.2 («la escribe y la sirve `shared`»);
  - `architecture-review/SKILL.md`, línea 40.

**Terminado cuando:**
- `shared` no tiene `changelog` ni `importbatches`;
- `documents` sigue sin dependencias;
- `ModularityTests` pasa con `audit::acl` e `imports::acl` como únicas puertas;
- la bitácora, las tres cargas y `/imports` responden igual.

### Fase 2 — `equipment`

**Por qué.** Los dispositivos tienen lenguaje y actores propios (serie, IMEI, ICCID, taller, monitoreo) y van a crecer con el monitoreo. `Equipment#installOnVehicle` no tiene quién lo llame y `vehicle_equipments` no apunta a `fleet_equipments`: el inventario todavía no está unido a la instalación, así que es el momento más barato para separarlo (cap. 14). La instalación en la unidad se queda en `fleet`, porque la puerta de Disponible la lee en la misma transacción que el estado (cap. 6).

**Antes de empezar.** D-260 está construida e integrada, como trabajo funcional propio y no como parte de esta fase, porque cambia lo que hace el sistema. Va sobre la rama `refactor/remove-equipment-type-flags`, corregida así:
- **La configuración de los requisitos, en `fleet`,** sembrada desde `equipment_types.blocks_availability` para que ninguna unidad cambie de estado de habilitación, con `RequirementHolds`, la lista de la habilitación y la puerta leyéndola; las marcas `requires_security_equipment` y `blocks_equipment_changes` de `vehicle_statuses`; el paso automático por «Pierde un requisito»; y sus pruebas, también las de extremo a extremo y las en vivo.
- **`EquipmentType.blocksEnablementAvailability()`** de la rama, que fija el GPS y la pantalla en el código, se reemplaza por la configuración de `fleet`.
- **`blocks_availability` sale de `equipment_types` en una entrega posterior,** cuando nada la lee. La rama la quita (V138) en la misma entrega que el código que deja de leerla.
- **V140 se corrige o se descarta** según dónde se guarda un equipo (diccionario §14, abiertas). Tal como está, deja sin cambiar el equipo cuya ubicación tiene el mismo número que alguna sede, que pasa a apuntar a esa sede, y lleva a la primera sede de la empresa el que está en una ubicación sin sede.
- **`requires_serial` sale solo si el responsable decide** que todo equipo instalado exige su serial (diccionario §14, abiertas).
- **La rama suma** `EquipmentBrandAuditTests`, sus pruebas de extremo a extremo y la documentación de las marcas de equipo.

Así esta fase mueve un catálogo de equipos que ya no decide qué se exige, y los requisitos no se mueven dos veces.

#### 2.1 Backend — el módulo

- **`equipment/package-info.java`:** `@ApplicationModule(displayName = "Equipment", allowedDependencies = {"shared", "organization::acl", "vehiclecatalog::acl", "documents::acl"})`, con `@NullMarked`. `organization::acl` da la empresa y la ubicación de cada equipo; `vehiclecatalog::acl`, los tipos de vehículo y lo que la habilitación necesita de cada modelo. `equipment/interfaces/acl/package-info.java` lleva `@NamedInterface("acl")`.
- **Desde `fleet`, el inventario** (30 archivos, al mismo paquete relativo):
  - **dominio:** `Equipment`, `EquipmentStatus`, `CreateEquipmentCommand`, `UpdateEquipmentCommand`, `ChangeEquipmentStatusCommand`, `GetEquipmentByPublicIdQuery`, `SearchEquipmentsQuery`, `EquipmentRepository` y `EquipmentIdentifierTakenException`;
  - **aplicación:** `EquipmentCommandService`, `EquipmentCommandServiceImpl`, `EquipmentQueryService`, `EquipmentView`, `EquipmentQueryServiceImpl`, `EquipmentViewFactory` y `ExternalSupplierService`;
  - **infraestructura:** `EquipmentPersistenceEntity`, `EquipmentPersistenceAssembler`, `EquipmentJpaRepository` y `EquipmentRepositoryImpl`;
  - **interfaces:** `EquipmentsController` (`/equipments`, con `stock.ver` para leer y `stock.crear_equipamiento`, `stock.editar_equipamiento` y `stock.cambiar_estado_equipamiento` para escribir); `EquipmentResource`, `CreateEquipmentResource`, `UpdateEquipmentResource` y `ChangeEquipmentStatusResource`; sus cuatro ensambladores; y el puerto `EquipmentSuppliers`, que pasa a `equipment/interfaces/acl`.
  - **Los errores:** las 20 fábricas del inventario en `FleetErrors` (`equipment*`, `supplierNotFound`, `supplierInactive` y `deniedOnEquipment`) pasan a `EquipmentErrors`, con los mismos códigos. `companyNotFound` y `locationNotFound` se copian, porque `fleet` las sigue usando.
- **Se quedan en `fleet`:** `VehicleEquipment` con sus servicios, controlador, recursos y errores; `EquipmentPeriod`, `EquipmentCost`, `EquipmentRemovalReason` y `VehicleEquipmentOverlapException`; `VehicleEquipmentCurrencyUsageAdapter`; y su traducción `EquipmentTypeReference` y `ModelEnablementProfile`. `EquipmentModelReference` de `fleet` se borra: solo la usaba el inventario.
- **Desde `vehiclecatalog`, el catálogo de equipos:**
  - las 59 clases `Equipment*`, con los controladores `/equipment-types` y `/equipment-models`, que leen con `catalogos.ver` y escriben con `catalogos.crear_equipamiento`, `catalogos.editar_equipamiento` y `catalogos.desactivar_equipamiento`;
  - `VehicleTypeApplicability` (`domain/model/entities`);
  - `GetModelEnablementProfilesQuery`, que pasa a `GetEquipmentApplicabilityQuery`;
  - lo que quede de equipos en `ModelEnablementProfiles` y `ModelEnablementView` después de D-260, que pasa a `EquipmentApplicability` y `EquipmentApplicabilityView`: qué equipo admite cada modelo y cuál trae de fábrica, con la regla de la pantalla de fábrica (VCT-R28) escrita una sola vez. Qué se exige ya lo decide `fleet`;
  - una `NaturalKeyTakenException` propia de `equipment`, con `EQUIPMENT_TYPE_CODE` y `EQUIPMENT_MODEL_NAME`; la de `vehiclecatalog` pierde esas dos claves.
- **Lo que el catálogo leía del interior de `vehiclecatalog`** (`VehicleTypeRepository` y `VehicleType` en el servicio de comandos, la fábrica de vistas, el servicio de consultas, la vista y el controlador) pasa por un `ExternalVehicleCatalogService` de `equipment`. Los tipos de vehículo de una vista son un objeto de valor propio, `ApplicableVehicleType` (regla 1.1). `VEHICLETYPE_NOT_FOUND` sigue nombrando los identificadores que faltan, ordenados.
- **Dentro de `equipment`, el inventario lee sus tipos y modelos por sus propios repositorios**, nunca por su fachada. Los códigos de error (`EQUIPMENTTYPE_NOT_FOUND`, `EQUIPMENT_TYPE_INACTIVE`, …) no cambian.
- **La placa de la unidad que muestra un equipo** sale de un puerto nuevo, `EquipmentUnits` (en lote: la unidad por su id, con `publicId` y placa), que implementa `fleet` en `fleet/infrastructure/equipment/VehicleEquipmentUnitsAdapter`. Así `equipment` no depende de `fleet` y no hay ciclo. `EquipmentResource` conserva `currentVehicleId` y `currentVehiclePlate`.
- **Los beans de traducción llevan nombre**, como los de `fleet` y `procurement`: `equipmentExternalOrganizationService`, `equipmentExternalVehicleCatalogService` y `externalSupplierService`. `iam` ya tiene un `externalOrganizationService` por omisión.
- **Las exportaciones y sus versiones:**
  - `EquipmentDocumentKinds` recibe el bean `equipmentTypesDocumentKind`, con el mismo `kind` (`equipment-types`) y la misma clave (`export.equipment-types.name`); su versión es `dataVersions.equipmentTypes() + ";" + externalVehicleCatalogService.fetchDataVersion()`.
  - `JdbcEquipmentDataVersions` nombra solo `equipment_types` y `equipment_type_vehicle_types`, en `equipmentTypes()` y en `enablement()`. Nunca `vehicle_types` (regla 6.2).
  - En `vehiclecatalog`, `enablement()` queda en `vehicle_models` y `vehicle_types`, y `equipmentTypes()` sale.

#### 2.2 Backend — las fachadas

- **`vehiclecatalog::acl` suma**, porque `equipment` los necesita:
  - `fetchVehicleTypeSummariesByPublicIds(Set<UUID>)`, `fetchVehicleTypeSummariesByIds(Set<Long>)` y el record `VehicleTypeSummary(id, publicId, code, name, isActive)`;
  - `fetchEnablementProfilesByModelIds`, que devuelve `ModelEnablementProfile(modelId, vehicleTypeId, runsOnGas, hasFactoryScreen)`.
- **`vehiclecatalog::acl` pierde:** `fetchEquipmentTypeReferenceByPublicId`, `fetchEquipmentTypeReferencesByIds`, `fetchEquipmentModelReferenceByPublicId`, `fetchEquipmentModelReferencesByIds` y `markEquipmentTypeInUse`, y sus records `EquipmentTypeReference` y `EquipmentModelReference`. `fetchEnablementDataVersion` queda solo en lo del modelo.
- **`EquipmentContextFacade`, solo con lo que `fleet` usa:**
  - `fetchEquipmentTypeReferenceByPublicId` (`VehicleEquipmentCommandServiceImpl:78`);
  - `fetchEquipmentTypeReferencesByIds` (`VehicleEquipmentCommandServiceImpl:103`, `VehicleEnablementList:125`, `VehicleEquipmentViewFactory:66`);
  - `fetchEquipmentApplicabilityByModelIds(Set<Long>)`: el equipo que admite cada modelo, con `factoryFitted`, leído una vez por clase; qué se exige lo lee `fleet` de su propia configuración (D-260);
  - `markEquipmentTypeInUse` (hasta la fase 8);
  - `fetchEnablementDataVersion`.
  - Sin `fetchEquipmentModel*`: no tienen consumidor.
- **Puertos que declara `equipment`:** `EquipmentSuppliers`, que implementa `procurement`, y `EquipmentUnits`, que implementa `fleet`.

#### 2.3 Backend — consumidores que cambian

- **`fleet`:**
  - `package-info.java` suma `"equipment::acl"`.
  - Un `ExternalEquipmentService` nuevo (`@Service("fleetExternalEquipmentService")`), que compone `ModelEnablementProfile` con `runsOnGas` de `vehiclecatalog` y lo que admite cada modelo según `equipment`: una lectura por contexto, nunca una por modelo. Los requisitos los cruza `fleet` con su propia configuración (D-260).
  - `ExternalVehicleCatalogService` pierde sus miembros de equipos (l.126-201).
  - Cambian a `ExternalEquipmentService`:
    - quien pide el perfil: `RequirementHolds:47`, `VehicleDocumentCommandServiceImpl:121`, `VehicleEquipmentInstallations:74`, `VehicleEnablementFactsReader:63`, `VehicleEnablementList:82,103`, `VehicleEquipmentViewFactory:102` y `MigratedUnitRegistrations:283`;
    - quien lee tipos: `VehicleEquipmentCommandServiceImpl:78,103`, `VehicleEnablementList:125` y `VehicleEquipmentViewFactory:66`;
    - quien marca el uso: `VehicleEquipmentInstallations:174` y `MigratedUnitRegistrations:204`;
    - la versión: `FleetDocumentKinds:67`, que compone la versión de la habilitación de los dos catálogos.
  - `UnitEnablement`, `VehicleEnablementViewFactory` y `EnablementPolicy` no cambian si `ModelEnablementProfile` conserva su forma.
  - `FleetErrors` pierde las 20 fábricas.
  - Adaptador nuevo: `fleet/infrastructure/equipment/VehicleEquipmentUnitsAdapter`.
- **`procurement`:** `package-info.java` suma `"equipment::acl"`, y `SupplierEquipmentSuppliersAdapter` pasa a `procurement/infrastructure/equipment/`, implementando `equipment.interfaces.acl.EquipmentSuppliers`.
- **No cambian:** `reporting`, que lee `equipment_types` y `equipment_type_vehicle_types` por SQL (`JdbcReportsReadRepository.java:151-170`) con los mismos nombres; `hand-over-database.sql`; las políticas de RLS; los campos confidenciales de la bitácora (el costo es de `vehicle_equipments`, que se queda); el `CurrencyUsage` de la instalación; y los permisos (decisión 2).

#### 2.4 Lo que no cambia, explícitamente

- **Ninguna tabla, columna ni migración.**
- **Ningún candado:** las mismas sentencias en el mismo orden, solo por otra fachada.
- **Ninguna ruta, recurso ni código de error.**
- **El alcance:** el inventario se lee con `stock.ver` por empresa (`companyReach` y `actsOn`) y se escribe con `actsOnWholeCompanyEveryBrand`. `fleet_equipments` conserva su `company_id`, su RLS forzado y sus 3 políticas (V132); `equipment_types` y `equipment_models` siguen sin RLS.

#### 2.5 Orden dentro de la fase

1. **Caracterizar.** Guardar el estado de la habilitación de todas las unidades de la base local (`GET /vehicle-enablements` exportado) y tener la suite completa en verde.
2. **El esqueleto de `equipment`:** sus dos `package-info.java` y la fachada vacía.
3. **Sumar** los miembros nuevos de `vehiclecatalog::acl`, sin quitar nada.
4. **Mover el catálogo de equipos,** con sus versiones, su tipo de documento y su fachada.
5. **Mover el inventario,** con `EquipmentErrors`, `ExternalSupplierService`, `EquipmentSuppliers` y el puerto `EquipmentUnits`.
6. **Reapuntar `fleet` y `procurement`,** con sus adaptadores.
7. **Retirar de `vehiclecatalog`** sus miembros de equipos y partir `ModelEnablementProfiles`.
8. **Mover y partir las pruebas** (§2.7) y correr el paso 1 otra vez: el mismo estado para las mismas unidades.
9. **Documentación y skills.**

#### 2.6 Regla para cuando el monitoreo una el inventario a la instalación (se escribe ahora y se construye con el monitoreo)

- **Fuente de verdad.**
  - Qué dispositivo está en qué unidad, y desde cuándo, lo dice la instalación de `fleet`: el período abierto de `vehicle_equipments`, que suma `equipment_id` como FK a `fleet_equipments`. Queda nulo para lo migrado y lo de fábrica.
  - `fleet_equipments.status = INSTALLED` y `current_vehicle_id` son el estado actual del dispositivo, escrito en la misma transacción que la instalación. Es una excepción a la regla 4.1, como el estado actual de la unidad, y se registra como decisión.
- **Quién llama a quién.** Solo `fleet` instala, reemplaza y retira, con `habilitacion.gestionar_equipos` sobre la unidad. En la misma transacción llama a `EquipmentContextFacade.installDevice(dispositivo, unidad, empresa, tipo)` y `releaseDevice(dispositivo, destino, motivo)`, que devuelven `Result`. `equipment` escribe sus filas. Nunca escribe nadie en las tablas de otro contexto.
- **Orden de los candados.** La unidad (`vehicles`, `FOR NO KEY UPDATE`, `EnablementUnitLocks`) → la instalación (`vehicle_equipments`) → los dispositivos (`fleet_equipments`, `FOR NO KEY UPDATE`, en orden de id cuando un reemplazo toma dos) → la escritura de la instalación. `equipment` nunca bloquea una unidad ni una instalación: así no hay ciclo con las escrituras del inventario, que solo bloquean el dispositivo.
- **Invariantes.**
  - Un dispositivo está en una sola unidad: índice único parcial `vehicle_equipments (equipment_id) WHERE equipment_id IS NOT NULL AND upper_inf(period)`.
  - Está `INSTALLED` si y solo si tiene una instalación abierta.
  - Es de la empresa de la unidad y del tipo de la instalación: FK `(equipment_id, company_id, equipment_type_id)` a `fleet_equipments (id, company_id, equipment_type_id)`, con su `UNIQUE`. `current_vehicle_id` pasa a ser FK compuesta `(current_vehicle_id, company_id)` a `vehicles (id, company_id)`.
  - El serial de la instalación es el del dispositivo. Ya no se edita mientras el dispositivo no está `AVAILABLE` (`updateIdentifiers`).
- **El inventario no saca un dispositivo de `INSTALLED`.**
  - Se retira `INSTALLED → MAINTENANCE` de `Equipment.canTransitionTo` y de `moveToMaintenance`, y de las convenciones (§ de `PATCH /equipments/{id}/status`). Es un cambio del contrato de `PATCH /equipments/{id}/status`, que hoy nadie puede ejercer.
  - El inventario responde `422 EQUIPMENT_INSTALLED`. No se da de baja, no se manda a mantenimiento y no se corrige un dispositivo instalado.
- **Las reglas del retiro siguen en `fleet`** (D-260). Un dispositivo solo sale de una unidad por el retiro o el reemplazo de `fleet`, que antes aplica `RequirementHolds`: en Contrato firmado y en Activo un equipo de seguridad no se retira sin reemplazo, en Robado no se retira nada, y en Disponible el retiro lleva la unidad a En habilitación en la misma transacción. Así el inventario no puede invalidar en silencio lo que la puerta cuenta.
- **El destino al retirar** lo decide el responsable. Propuesta: `falla` → `MAINTENANCE`, `robo` → `DISCARDED`, `reemplazo` → `AVAILABLE`. También queda por decidir qué pasa con un dispositivo instalado cuando la unidad se anula o pasa a Inactivo: hoy `VehicleAnnulments` y `VehicleTransfers` no miran las instalaciones.

#### 2.7 Pruebas del backend

- **Se mueven las 5 del inventario:** `EquipmentTests`, `EquipmentCommandServiceImplTests` y `EquipmentQueryServiceImplTests` (con sus mocks reescritos: ya no hay `ExternalVehicleCatalogService` de `fleet`, y `EquipmentSupplierSummary` sale de `equipment.interfaces.acl`), `EquipmentsControllerTests` y `EquipmentsIntegrationTests`.
- **Se mueven las 12 del catálogo:** `EquipmentTypeTests`, `EquipmentModelTests`, `EquipmentTypeCommandServiceImplTests` (que ya no mockea `VehicleTypeRepository`), `EquipmentModelCommandServiceImplTests`, `EquipmentTypeAuditTests`, `EquipmentTypeQueryServiceImplTests`, `EquipmentModelQueryServiceImplTests`, `EquipmentTypeApplicabilityConstraintTests`, `EquipmentTypeApplicabilityPersistenceTests`, `EquipmentTypeLockTests`, `EquipmentTypesControllerTests` y `EquipmentModelsControllerTests`.
- **Se parten:**
  - `VehicleCatalogContextFacadeTests`: sus cinco casos de equipos pasan a `EquipmentContextFacadeTests`, y se suma el perfil reducido del modelo;
  - de `VehicleCatalogRacedDuplicatesTests`, el caso de la línea 146;
  - de `VehicleCatalogExportsIntegrationTests`, el de la línea 136;
  - `VehicleCatalogFixtures.equipmentType` y `equipmentModel`, que pasan a `EquipmentFixtures`;
  - `VehicleCatalogTestData.gpsForCars`;
  - la parte de equipos de `VehicleCatalogMigrationTests`.
- **Nuevas:** el adaptador `EquipmentUnits`, `EquipmentAuditTests` (regla 4.8) y `fleet_equipments` en una prueba de filas que nunca se borran.
- **Se corren completas:**
  - toda la habilitación de `fleet`: `EnablementGateIntegrationTests`, `EnablementConcurrencyTests`, los seis `VehicleEnablement*`, `VehicleEquipmentIntegrationTests`, `VehicleEquipmentOverlapTests`, `VehicleEquipmentAuditTests`, `VehicleArrivalIntegrationTests`, `EnablementAnnulledUnitTests`, `EnablementDatesInLimaTests` y `EnablementPermissionsIntegrationTests`;
  - `InitialMigrationIntegrationTests`, `ReportsIntegrationTests` y `StockRowsTests`;
  - `RowLevelSecurityMigrationTests`, `FleetRowLevelSecurityTests` y `PermissionsMigrationTests`;
  - las de arquitectura y `ErpMillenniumMobilityBackendApplicationTests`, que detecta los nombres de bean repetidos.
- **`EnablementFixtures` (l.148-155) repite en SQL la regla de los requisitos,** con la pantalla de fábrica: se mantiene alineada.

#### 2.8 Frontend

- **Carpeta nueva `src/app/equipment`,** con sus cuatro capas.
- **Recibe los 35 archivos del inventario de `fleet`:**
  - `application`: `equipment-inventory.store.ts` y su spec;
  - `domain/model`: `equipment-item.entity.ts`, `equipment-status.ts`, `equipment-filters.ts`, `equipment-telematics.vo.ts` y `commands/{create-equipment, update-equipment, change-equipment-status}.command.ts` con `equipment-command-helpers.ts`;
  - `infrastructure/api`: `equipments-api-endpoint.ts`, `equipments-response.ts` y `{create-equipment, update-equipment, change-equipment-status}.request.ts`;
  - `infrastructure/assemblers`: `equipment-assembler.ts` y su spec;
  - `presentation`: `views/equipment-inventory-view/` (4), `components/equipment-form-dialog/` (4), `components/equipment-details-dialog/` (4), `components/equipment-status-dialog/` (4), `equipment-status-tone.ts` y `_equipment-dialog-shared.scss`.
- **Recibe los 35 archivos del catálogo de `vehiclecatalog`:**
  - `application`: `equipment-catalog.store.ts` y su spec;
  - `domain/model`: `equipment-type.entity.ts`, `equipment-model.entity.ts` y sus 6 comandos;
  - `infrastructure/api`: `equipment-types-api-endpoint.ts`, `equipment-types-response.ts`, `equipment-models-api-endpoint.ts`, `equipment-models-response.ts` y sus 6 peticiones;
  - `infrastructure/assemblers`: `equipment-type-assembler.ts` y `equipment-model-assembler.ts`;
  - `presentation`: `views/equipment-types-view/` (4), `components/equipment-types-dialog/` (5, con `equipment-type-form.ts`) y `components/equipment-model-form-dialog/` (4).
- **Archivos nuevos:**
  - `equipment/infrastructure/api/equipment-api.ts` (`EquipmentApi`), con lo que sale de `fleet-api.ts:366-381` y de `vehiclecatalog-api.ts:174+`;
  - `equipment/presentation/equipment.routes.ts` y su spec, con `inventory` (`stock.ver`) y `types` (`catalogos.ver`), con las mismas constantes de acceso y los mismos `title`;
  - una copia de `deactivation-question.ts` en `equipment/presentation/components/`, o su paso a `shared` con una decisión.
- **Sin `EquipmentFacade`:** ningún contexto la usaría.
  - El inventario lee los tipos y modelos de `EquipmentCatalogStore`, con su estado externo, en vez de `VehicleCatalogFacade` (`equipment-inventory-view.ts:26,56` y `equipment-form-dialog.ts:66`).
  - Los tipos de equipamiento leen los tipos de vehículo de `VehicleCatalogFacade`, en vez de `VehicleTypesStore` (`equipment-types-view.ts:38,78` y `equipment-types-dialog.ts:40,103`).
- **`VehicleCatalogFacade`:**
  - pierde sus miembros de equipos (l.37-56 y l.150-206) y la inyección de `EquipmentCatalogStore`;
  - **gana `vehicleTypeOptions`:** todos los tipos, activos o no, con `isActive`, porque el diálogo ofrece los inactivos que un equipo ya tiene.
- **Las rutas pasan a `/equipment/inventory` y `/equipment/types`:**
  - `app.routes.ts` suma `{ path: 'equipment', loadChildren: equipmentRoutes }`;
  - `fleet.routes.ts` y `vehiclecatalog.routes.ts` dejan una redirección (`redirectTo: '/equipment/inventory'` y `'/equipment/types'`, `pathMatch: 'full'`);
  - `app-navigation.ts:175,345` cambia los enlaces sin cambiar grupo, permiso, título ni descripción.
- **Specs:**
  - las 10 que se mueven con su código;
  - `vehiclecatalog.facade.spec.ts`;
  - `app-shell.spec.ts:316`, y en `:385-397` se suma `equipmentRoutes` a las rutas que compone;
  - `fleet.routes.spec.ts:115-118` pasa a `equipment.routes.spec.ts`;
  - `equipment-form-dialog.spec`, `equipment-inventory-view.spec`, `equipment-types-view.spec` y `equipment-types-dialog.spec`, con sus mocks nuevos.
- **e2e:**
  - las rutas en `e2e/screens.ts:533`, `e2e/vehicle-catalog.spec.ts:233,243,269,283,636,679`, `e2e/exports.spec.ts:232` y `e2e-live/vehiclecatalog.live.spec.ts:517`;
  - un caso que pruebe cada redirección;
  - los mocks (`support/mocks/equipments.ts` y `vehiclecatalog.ts`) no cambian, porque la API es la misma;
  - **las imágenes de referencia no se regeneran:** su nombre es `screen.name` (`visual.spec.ts:44`), y una que cambie es una regresión;
  - opcional: sumar la pantalla `equipment-inventory` a `screens.ts`, con su imagen nueva revisada.
- **i18n:** las claves (`fleet.equipment.*`, `vehicle-catalog.equipment-types.*`, `vehicle-catalog.status.*`, `vehicle-catalog.confirm.*`, `fleet.vehicles.no-plate`) no se renombran en esta fase.

#### 2.9 Documentación

- `contextos/equipment.md`, con lo de equipamiento de `fleet.md` y de `vehiclecatalog.md`. Las reglas conservan su id (`FLT-R…`, `VCT-R…`); VCT-R28 pasa entera a `equipment`.
- El mapa vigente y el diccionario §3, con la fila 9 del bloque 9: D-254 sigue abierta.
- Las convenciones: la tabla de rutas por contexto (§2) y el texto de los modelos de equipamiento («catálogo general de `vehiclecatalog`»).
- **Las cuatro copias de las skills** (`.claude` y `.cursor` de los dos repositorios), con `equipment` en la búsqueda de la regla 1.2 de `architecture-review/SKILL.md:141` y en la lista de `allowedDependencies` de `backend-module`.

**Terminado cuando:**
- la habilitación de cada unidad da el mismo estado que en el paso 1;
- `vehiclecatalog` ya no sabe de equipos;
- `ModularityTests` pasa con `fleet` → `equipment`, `procurement` → `equipment` y `equipment` → `vehiclecatalog` y `organization`, sin ciclos;
- la aplicación arranca, sin dos beans con el mismo nombre.

### Fase 3 — `legacyimport`

**Por qué.** El libro de stock de 52 columnas es un modelo heredado. El libro pide traducirlo en una *Anticorruption Layer* (cap. 14) —fachadas, adaptadores y traductores— para que no contamine el modelo nuevo. Hoy esa traducción vive dentro de `procurement` y de `fleet`, y a medias: los writers de compras hablan de filas y de columnas del libro.

Después de esta fase, `legacyimport` es el único que conoce las columnas, las filas y el informe. Las reglas de cada dueño se quedan en su contexto y se exponen por su fachada:
- la factura migrada sin los 20 días (D-177);
- la nota de campaña sin mes (D-191, D-192, D-207);
- el pago de dealer con su inicial (D-197, D-199, D-248);
- la unidad registrada en su estado (D-202).

**Una decisión:** si la carga inicial se ejecuta antes de esta fase, la fase se reemplaza por borrar el código después de la puesta en marcha, y la fase 5 crea `procurement::acl`.

**Quién llama a quién.**
- `legacyimport` lee el libro, lo agrupa, resuelve los nombres y orquesta las transacciones.
- De `fleet::acl` usa solo `matchMigratedUnits` y `resolveMigrationLocation`.
- `registerMigratedUnits` lo sigue llamando el registro de la factura en `procurement`, dentro de su transacción. Así no cambian:
  - el orden de los candados: empresa, proveedor y moneda en modo compartido; después la marca y las unidades en `fleet`; después la factura (§2.4 de la especificación, patrones de persistencia);
  - qué rechazo gana en el informe cuando una factura tiene más de uno.

**Backend — `legacyimport`.**
- **`package-info.java`** con `@ApplicationModule(displayName = "LegacyImport", allowedDependencies = {"fleet::acl", "procurement::acl", "organization::acl", "location::acl", "vehiclecatalog::acl", "imports::acl"})` y `@NullMarked`.
  - Sin `documents::acl`: no exporta ni guarda archivos; los guarda `imports`.
  - Sin `interfaces/acl`: nadie depende de él.
- **Desde `procurement` se mueven:**
  - en `interfaces/rest`: `InitialMigrationImportsController`, con sus rutas (`POST /api/v1/initial-migration-imports` y `/preview`), su permiso (`PermissionCodes.Stock.LOAD_INITIAL_MIGRATION`, `stock.cargar_migracion_inicial`) y su `@Tag("Imports")`; `ImportInitialMigrationResource` e `InitialMigrationPreviewResource`;
  - en `application/commandservices`: `InitialMigrationImportCommandService`;
  - en `application/internal/commandservices`:
    - `InitialMigrationImportCommandServiceImpl`, con el mismo nombre y el mismo paquete relativo, para que lo siga envolviendo `DeadlockRetry`;
    - `InitialMigrationRun`, `InitialMigrationCatalogs` y `StockRowMapper`;
    - `MigrationTransactions`: `REQUIRES_NEW` y el reintento de cada paso (PRC-R152);
  - en `application/queryservices`: `InitialMigrationEffects` e `InitialMigrationPreview`;
  - en el dominio: `ImportInitialMigrationCommand` (`domain/model/commands`), `StockRow` (`domain/model/valueobjects`) y `StockRowGroups` (`domain/services`);
  - en `application/internal/outboundservices/acl`: `MigrationUnitMatch`, y una copia de `ImportableModel`. El original se queda, porque lo usa la importación de las líneas de una factura (`PurchaseInvoiceLineImportQueryServiceImpl`).
- **Clases nuevas de `legacyimport`:**
  - su alcance de la carga: la empresa, su nombre, la marca con su código y la sede;
  - `LegacyImportErrors.companyInactive()`, con el mismo código `COMPANY_INACTIVE`;
  - un calendario sobre el `Clock` UTC, con el «hoy» en la zona de la empresa;
  - `LegacyTextFolding.fold` en `application/internal/policies`, copia pura de `InvoiceLineImportPolicy.fold`, que pliega los dos lados del cruce de nombres;
  - `ExternalProcurementService`, `ExternalFleetService`, `ExternalOrganizationService`, `ExternalLocationService` y `ExternalVehicleCatalogService` en `application/internal/outboundservices/acl`, con records propios y nombre de bean explícito (`@Service("legacyimportExternal…Service")`). Los de `iam` y `organization` no llevan nombre explícito: un homónimo sin nombre chocaría con `externalOrganizationService` y `externalLocationService`.
- **Lo que esas clases toman hoy del interior de `procurement`, y cómo queda:**

| Hoy | Después |
|---|---|
| `StockRow` lleva `InvoiceNumber` y `OperationNumber` | Lleva `String` ya normalizados |
| `InvoiceNumber.normalize` en `StockRowMapper` | `normalizeVoucherNumber` de `procurement::acl`, por `ExternalProcurementService` |
| `OperationNumber.normalize` | `TextIdentifier.normalize` de `shared`, que es lo que hace |
| `InvoiceLineImportPolicy.fold` | `LegacyTextFolding.fold` |
| `PurchaseInvoice.MAX_LINES` en `StockRowGroups` | `maxInvoiceLines()` de la fachada, pasado a `StockRowGroups.refusals` |
| `SupplierRepository.findAllByCompanyId` y `Supplier` en `InitialMigrationCatalogs` | `fetchMigrationSuppliers(companyId)` |
| `PurchaseInvoiceRepository.findCurrentByNumber` y `PurchaseInvoiceItem` en el orquestador | `fetchCurrentInvoiceByNumber(companyId, supplierId, number)` |
| `ProcurementErrors.companyInactive()` | `LegacyImportErrors.companyInactive()` |
| `ProcurementCalendar` | El calendario de `legacyimport` |
| `ExternalOrganizationService`, `ExternalLocationService`, `ExternalVehicleCatalogService` y `ExternalFleetService` de `procurement`, con `InvoiceCompany`, `InvoiceCountry`, `BrandReference`, `OrderCurrency` y `LineColor` | Los de `legacyimport` |
| `ImportBatch*` de `shared.importbatches` | `imports::acl` (fase 1) |

- **`procurement::acl`, la primera fachada de `procurement`:** `procurement/interfaces/acl/package-info.java` con `@NamedInterface("acl")`, `ProcurementContextFacade`, y su implementación `procurement/application/acl/ProcurementContextFacadeImpl`. Devuelve tipos simples o records del propio paquete `acl`, nunca un tipo del dominio de compras ni de `fleet::acl`.
  - **Lectura:**
    - `fetchMigrationSuppliers(Long companyId)` → `List<MigrationSupplier(id, publicId, legalName, tradeName, isActive)>`;
    - `fetchCurrentInvoiceByNumber(Long companyId, Long supplierId, String number)` → `Optional<MigratedInvoiceReference(invoiceId, Set<Long> currentVehicleIds)>`;
    - `maxInvoiceLines()`;
    - `normalizeVoucherNumber(String raw)` → `Optional<String>`.
  - **Escritura,** con `@Transactional(propagation = MANDATORY)`, como `registerMigratedUnits` de `fleet`:
    - `registerMigratedInvoice(RegisterMigratedInvoiceRequest)` → `Result<MigratedInvoice, ApplicationError>`, con el `invoiceId`, los vehículos por VIN y el estado en que entró cada unidad;
    - `registerMigratedCreditNote(RegisterMigratedCreditNoteRequest)` → `Result<Boolean, ApplicationError>`;
    - `registerMigratedPayment(RegisterMigratedPaymentRequest)` → `Result<Boolean, ApplicationError>`.
  - **La petición de la factura** lleva la empresa, la marca, la sede, el proveedor, la moneda, el número, la fecha y la tasa. Por cada línea lleva el precio y los datos de la unidad que hoy viajan a `fleet`: VIN, motor, modelo, año, color, número del libro, propietario, la habilitación, el SOAT y la llave con su etiqueta, que calcula `legacyimport` con el código de la marca.
  - **Los rechazos** van en `MigratedRecordRefusals(List<Refusal(int line, String code, MigratedRecordField field)>)`.
    - `MigratedRecordField` es un enum de `procurement`: `INVOICE_NUMBER`, `INVOICE_DATE`, `PRICE`, `RATE`, `NOTE_NUMBER`, `NOTE_DATE`, `NOTE_VALUE`, `OPERATION`, `PAYMENT_DATE` y `PAYMENT_AMOUNT`.
    - Los rechazos de `fleet` llegan traducidos por su campo (`MigratedUnitField`).
    - `legacyimport` traduce línea → fila y campo → columna del libro (`StockRowMapper.columnOf`).
- **En `procurement` se quedan las reglas de compras de lo migrado, sin el libro:**
  - `MigratedInvoiceWriter`, `MigratedCreditNoteWriter` y `MigratedPaymentWriter` dejan de nombrar `StockRow` y `StockRowMapper`: reciben los records de la fachada y rechazan por `MigratedRecordField`.
  - `MigrationScope` se queda y se arma dentro del miembro de la fachada, empezando por la misma lectura `FOR SHARE` de la empresa que ya hace cada writer, para que el orden de los candados no cambie.
  - Se quedan `MigrationRefusals` y `MigrationUnit`, y `MigrationUnitLine` se redefine sin `StockRow`.
  - De `ExternalFleetService` se quedan `registerMigrated` y `fetchUnits`; `matchMigrated` y `resolveMigrationLocation` salen a `legacyimport`.
  - Se quedan los códigos que levantan los writers: `INITIALMIGRATION_RATE_NOT_ONE`, `INITIALMIGRATION_CREDIT_NOTE_REGISTERED`, `INITIALMIGRATION_CREDIT_NOTE_ZEROES_LINE`, `INITIALMIGRATION_PAYMENT_REGISTERED` e `INITIALMIGRATION_PAYMENT_INCONSISTENT`. Los demás `INITIALMIGRATION_*` son literales del orquestador, de `InitialMigrationRun`, de `StockRowMapper`, de `StockRowGroups` y de `InitialMigrationCatalogs`, y se van con ellos. Las claves de los `messages*.properties` no cambian.
  - `procurement` retira `"imports::acl"` de su `package-info.java`: ya no lo usa.
- **En `fleet` no cambia nada:** `MigratedUnitRegistrations`, `MigrationTargetPolicy` y los tres miembros de su fachada (`matchMigratedUnits`, `resolveMigrationLocation` y `registerMigratedUnits`), porque registrar una unidad en su estado es regla de flota (D-202). Siguen ahí `INITIALMIGRATION_LEGACY_NUMBER_TAKEN` e `INITIALMIGRATION_SCREEN_NOT_FROM_FACTORY`.
- **Las transacciones y los candados siguen como hoy.**
  - El orquestador no es transaccional y sigue envuelto por `DeadlockRetry`.
  - `MigrationTransactions` abre una transacción `REQUIRES_NEW` para cada uno de estos pasos: el alcance, los catálogos con los VIN, la búsqueda de cada factura existente, cada factura, cada nota, cada pago y el registro de la carga. Cada paso se repite una vez si pierde un interbloqueo.
  - Los miembros de las fachadas corren en la transacción de quien los llama.
  - El candado de la carga es `ImportLocks.tryOnSession`, de `imports::acl`, en su propia conexión.
  - Row Level Security no cambia: el alcance lo fija `AccessScopeJpaTransactionManager` en cada transacción, sea cual sea el módulo.
- **El contrato no cambia:** las rutas, el permiso, el `@Tag`, los recursos y los códigos de error son los mismos.

**Después de la puesta en marcha, el módulo se borra:**
- sale `legacyimport` entero, con los miembros de migración de `ProcurementContextFacade` y los tres writers, y `matchMigratedUnits`, `resolveMigrationLocation` y `registerMigratedUnits` de `fleet`, con `MigratedUnitRegistrations`;
- si para entonces la fase 5 ya puso miembros en `procurement::acl`, la fachada se queda con ellos;
- se conservan los datos, los caminos `MIGRACION_INICIAL`, `legacy_number` y las migraciones V117 a V119 y V127.

**Frontend.** Opcional, porque las rutas no cambian. El diálogo de la carga son 7 archivos en `procurement`:
- `application/initial-migration.store.ts` y su spec;
- `domain/model/initial-migration.ts`;
- `infrastructure/api/initial-migration-api-endpoint.ts`;
- `presentation/components/initial-migration-import-dialog/`, con su `.ts`, su `.html` y su `.scss`.

Además está `ProcurementFacade.openInitialMigrationImport`, que llama la pantalla Cargas (`imports-view.ts`). Pueden quedarse hasta el borrado. Si se mudan a `src/app/legacyimport`, la pantalla Cargas lo abre por `LegacyImportFacade`. `e2e/screens.ts` (`initial-migration-preview`) y `e2e/support/mocks/imports.ts` no cambian.

**Pruebas.**
- **Se mueven a `legacyimport`**, espejando los paquetes:
  - `InitialMigrationIntegrationTests` (27 casos), con `StockWorkbook`, que es package-private y va al mismo paquete;
  - `StockRowMapperTests` y `StockRowGroupsTests`;
  - `InitialMigrationCatalogsTests`, que se reescribe sobre `MigrationSupplier`, porque hoy construye el agregado `Supplier`.
- **Se quedan:** `MigrationTargetPolicyTests` (`fleet`) y `SpreadAndCustomerInitialMigrationTests` (el esquema de V127, en `procurement`).
- **Nuevas, en `procurement`:** los miembros de `ProcurementContextFacade` (exigen transacción; un rechazo vuelve con su línea y su campo) y los writers sin `StockRow`.
- **Se corren:**
  - las de arquitectura (`ModularityTests`, `ForeignAclStaysInTranslationLayerTests`, `DeadlockRetryTests` y las demás de §2);
  - `ErpMillenniumMobilityBackendApplicationTests`, por los nombres de bean;
  - en el CI, `e2e/imports.spec.ts` y `e2e-live/imports.live.spec.ts`.

**Documentación.**
- `contextos/legacyimport.md`, con PRC-R142 a PRC-R154, PRC-R158 y PRC-R159, que conservan su id: un id no se reutiliza ni se renumera.
- `contextos/procurement.md` y `contextos/fleet.md`.
- El mapa vigente y el diccionario §3.
- El `README.md` del backend.
- Las skills, en sus cuatro copias: §1 de `backend-module/SKILL.md` («Las cargas y los reportes») y §2 (`allowedDependencies`).

**Terminado cuando:**
- `procurement` no nombra el libro: ni `StockRow`, ni sus columnas, ni sus filas;
- `ModularityTests` y `ForeignAclStaysInTranslationLayerTests` pasan;
- `InitialMigrationIntegrationTests` da los mismos conteos, el mismo informe y los mismos códigos que antes.

### Fase 4 — `financing`

**Por qué.** Otro lenguaje (fondo, cesión, fideicomiso, garantía) y otro actor, Contabilidad (cap. 14). La invariante que cruza va en los dos sentidos: una unidad con una cesión abierta no se anula (D-182), y una unidad anulada no se cede ni se le cierra una cesión. Se cumple en una sola transacción con el candado de la unidad, que toma `fleet`, y con un puerto que declara `fleet` y que implementa `financing` (cap. 16). La relación se parece más a una *Partnership* que a un *Customer/Supplier* puro, y el mapa la registra así.

#### Antes de empezar

- **La fase 1 está integrada** y `imports::acl` expone todo lo que usa la carga de garantía:
  - `ImportBatchCommandService` con `RegisterImportBatchCommand`;
  - `ImportBatchQueryService` (`handle(GetImportBatchByPublicIdQuery)` y `fetchReferencesByIds`), con `GetImportBatchByPublicIdQuery` e `ImportBatchView`;
  - `ImportOutcome`, `ImportLocks`, `ImportType` e `ImportCounts`;
  - `ImportRefusals.conflict()` e `ImportReading.reads(…)`;
  - el libro (`ImportWorkbookReader`, `ImportWorkbook`, `ImportRow`, `ImportCellException`, `ImportSheetSpec`, `ImportReportWriter`, `ImportReport`, `ImportRowFailure`, `ImportSummaries`, `ImportFailureMessages`);
  - la respuesta de una carga (`ImportBatchResourceFromViewAssembler`, `ImportResultResource`).
- **Decisión A o B: quién calcula `registerFundAssignment`** (convenciones §4.2):
  - **A, recomendada.** El puerto de `fleet` pregunta a `financing` si la sesión puede ceder la unidad ahora. `VehicleResource.allowedActions.registerFundAssignment` sigue igual y el contrato no cambia: la fase es solo de estructura, como promete el §1.
  - **B.** `registerFundAssignment` sale del recurso de la unidad y `financing` sirve la acción en un subrecurso nuevo (por ejemplo `GET /vehicles/{id}/fund-standing` → `{ "allowedActions": ["register"] }`). La forma se acuerda en convenciones §4.2 y §6.2 antes de construirla, el §1 y el §8 del plan la declaran como excepción, y backend y frontend se integran juntos. `GET /vehicles/{id}/fund-assignments` no sirve para eso: responde una página o un arreglo, sin lugar para una acción de la colección.
- **Decisión A1 o A2: quién dibuja la pestaña Cesiones** de la ficha de la unidad (`?seccion=cesiones`):
  - **A1, recomendada, sin excepción nueva.** La lista se queda en `fleet` y la alimenta `FinancingFacade`, con tipos propios y su propia carga y error (regla 1.3). Los flujos de ceder y cerrar los abre la fachada.
  - **A2.** Una pieza de UI prestada en `financing/presentation/public/`. Exige su excepción en la regla 1.2 del frontend y su revisión de `DESIGN.md`.

#### Backend — el módulo

- **`financing/package-info.java`** con `@ApplicationModule(displayName = "Financing", allowedDependencies = {"shared", "fleet::acl", "organization::acl", "iam::acl", "audit::acl", "imports::acl", "documents::acl"})` y `@NullMarked`. Para qué es cada dependencia:
  - `iam::acl`: el nombre de quien registró una cesión;
  - `audit::acl`: el cierre a mano y el que hace la carga escriben la bitácora (`VehicleFundAssignment.FIELD_PERIOD`). La corrección del código de un fondo no la escribe;
  - `imports::acl`: la carga de garantía y el lote de cada cesión;
  - `documents::acl`: la exportación de los fondos.
- **Sin fachada propia en esta fase:** nadie la consume. `reporting` sigue leyendo `vehicle_fund_assignments` por SQL, como modelo de lectura (regla 1.2).

#### Backend — lo que se mueve desde `fleet` (54 clases)

Mismo nombre y mismos subpaquetes, bajo `ky.millennium.mobility.erp.financing`:

| Capa | Clases |
|---|---|
| `domain/model` | `Fund` y `VehicleFundAssignment`; `FundType`; `AssignVehicleToFundCommand`, `ChangeFundStatusCommand`, `CloseVehicleFundAssignmentCommand`, `CreateFundCommand`, `ImportGuaranteeListCommand` y `UpdateFundCommand`; `FundSearchFilter`, `GetFundByPublicIdQuery`, `GetVehicleFundAssignmentsQuery` y `SearchFundsQuery` |
| `domain` (resto) | `FundCodeTakenException` y `VehicleFundAssignmentOverlapException`; `FundRepository` y `VehicleFundAssignmentRepository`; `GuaranteeListPolicy` |
| `application` | `FundCommandService`, `VehicleFundAssignmentCommandService`, `FundQueryService`, `FundView`, `VehicleFundAssignmentQueryService`, `VehicleFundAssignmentView`; `FundCommandServiceImpl`, `VehicleFundAssignmentCommandServiceImpl`, `FundQueryServiceImpl`, `FundViewFactory`, `VehicleFundAssignmentQueryServiceImpl` y `VehicleFundAssignmentViewFactory` |
| `infrastructure` | `FundRepositoryImpl`, `VehicleFundAssignmentRepositoryImpl`, `FundPersistenceAssembler`, `VehicleFundAssignmentPersistenceAssembler`, `FundPersistenceEntity`, `VehicleFundAssignmentPersistenceEntity`, `FundJpaRepository` y `VehicleFundAssignmentJpaRepository` |
| `interfaces/rest` | `FundsController` (`/funds`), `VehicleFundAssignmentsController` (`/vehicles/{vehicleId}/fund-assignments`, la misma ruta servida por otro módulo, con `Stock.VIEW` en la clase y `Funds.REGISTER_ASSIGNMENT` en el `POST`) y `FundAssignmentsController` (`/fund-assignments/{id}/closure`); `ChangeFundStatusResource`, `CloseVehicleFundAssignmentResource`, `CreateFundResource`, `CreateVehicleFundAssignmentResource`, `FundResource`, `ImportGuaranteeListResource`, `UpdateFundResource` y `VehicleFundAssignmentResource`; `CreateFundCommandFromResourceAssembler`, `CreateVehicleFundAssignmentCommandFromResourceAssembler`, `FundResourceFromViewAssembler`, `UpdateFundCommandFromResourceAssembler` y `VehicleFundAssignmentResourceFromViewAssembler` |

#### Backend — lo que se parte

- **La carga de garantía.**
  - `FleetImportCommandService.handle(ImportGuaranteeListCommand)` y su implementación en `FleetImportCommandServiceImpl` (el método, más `fundCodesOf`, `closureRows`, `closureRow`, `CLOSURE_HEADERS` y `maxGuaranteeRows`) pasan a `FundAssignmentImportCommandService` y `FundAssignmentImportCommandServiceImpl`.
  - La implementación va en `financing/application/internal/commandservices`, para que `DeadlockRetry` la envuelva, con `@Transactional` por método.
  - La propiedad sigue siendo `imports.max-rows.garantia-mobiliaria`.
  - Los ayudantes privados (`refusalToLoad`, `outcome`, `rollBack`, `fail` y `report`) se copian.
  - `fleet` conserva la carga del impuesto vehicular y deja de inyectar `FundRepository`, `VehicleFundAssignmentRepository` y `ChangeLogCommandService` en esa clase.
- **`FleetImportsController.importGuaranteeList`** pasa a `FundAssignmentImportsController`, con `POST /fund-assignment-imports` y `Funds.LOAD_GUARANTEE_FILE`.
- **La exportación de los fondos.** `FleetDocumentKinds.FUNDS` y `fundsDocumentKind` pasan a `FinancingDocumentKinds`, con el mismo `kind` `funds`, el mismo alcance (`DocumentScopes.grantsOf(scope, Funds.VIEW)`) y la misma versión: `funds()` más la de `organization`.
- **Las versiones de datos.** `FleetDataVersions.funds()` y `JdbcFleetDataVersions.funds()` pasan a `FinancingDataVersions` y `JdbcFinancingDataVersions` (`jdbcDataVersions.of("funds")`). `vehicle_fund_assignments` lo sigue nombrando solo `JdbcReportingDataVersions`.
- **Los errores.**
  - Pasan a `FinancingErrors` los de fondos de `FleetErrors`: `deniedOnCompany`, `fundNotFound`, `vehicleFundAssignmentNotFound`, los dos `fundConflict`, `fundCodeLocked`, `fundInactive`, `vehicleFundAssignmentCompanyMismatch`, `vehicleFundAssignmentConflict`, `vehicleFundAssignmentAlreadyClosed` y `vehicleFundAssignmentEndNotAfterStart`. También los códigos que hoy van en línea: `VEHICLEFUNDASSIGNMENT_LIST_DATE_IN_FUTURE` y `VEHICLEFUNDASSIGNMENT_IMPORT_UNCONFIRMED`.
  - `FinancingErrors` copia, con el mismo código, los que ceder, cerrar, listar y cargar responden y que también usa `fleet`: `VEHICLE_NOT_FOUND`, `VEHICLE_ANNULLED`, `ACCESS_DENIED` (como `deniedOnUnit`), `COMPANY_NOT_FOUND` y `COMPANY_INACTIVE`.
  - `VEHICLE_FUND_ASSIGNMENT_OPEN` es de la anulación y se queda en `FleetErrors`.

#### Backend — lo nuevo en `financing`

- **La traducción**, en `application/internal/outboundservices/acl`, con nombre de bean explícito: `iam` ya tiene un `ExternalOrganizationService` con el nombre por defecto, y otro igual impide arrancar.
  - `ExternalFleetService`, `@Service("financingExternalFleetService")`.
  - `ExternalOrganizationService`, `@Service("financingExternalOrganizationService")`: la empresa por su identificador público, `resolveScope` (la empresa `FOR SHARE`), las referencias de las empresas, sus zonas y `fetchDataVersion`.
  - `ExternalIamService`, `@Service("financingExternalIamService")`: `fetchAccountReferencesByIds`.
  - Sus records: `FinancedUnit`, `OwningCompany` y `AccountReference`.
- **`FinancingSessionAccount`** (con `CurrentAccessScope`) y **`FinancingCalendar`**, el «hoy» y la zona de la empresa (D-40). Reemplaza a `CompanyCalendar` de `fleet`, y no lleva ese nombre para no repetir el bean.
- **El adaptador del puerto:** `infrastructure/fleet/VehicleFundAssignmentUnitFundAssignmentsAdapter implements UnitFundAssignments`. Solo depende de `VehicleFundAssignmentRepository`, y de `CurrentAccessScope` y `FinancingCalendar` si se elige la opción A. Así no hay un ciclo de beans con `FleetContextFacade`.

#### Backend — lo que cambia en `fleet`

- **El puerto `interfaces/acl/UnitFundAssignments`:**
  - `boolean hasOpenAssignment(Long vehicleId)`;
  - con la opción A, también `boolean offersCession(Long vehicleId, Long companyId, Long commercialBrandId)`. Es la regla de hoy: `fondos.registrar_cesion` en toda la empresa o en la marca de la unidad (DP-11), `fondos.ver` en su empresa, y ninguna cesión abierta ni que termine después de hoy en la zona de la empresa (D-40).
- **`application/internal/outboundservices/funds/UnitFundFacts`** envuelve el puerto. Inyecta **un solo bean obligatorio**, no una lista: sin el bean, una unidad cedida se anularía sin aviso (D-182).
- **Miembros nuevos de `FleetContextFacade`**, con el record nuevo `FinancedUnit(id, publicId, companyId, commercialBrandId, plate, vin, annulled)`. No se toca `UnitSummary`, que usa `procurement`.
  - `Optional<FinancedUnit> lockVisibleUnitByPublicId(UUID)` y `lockVisibleUnitById(Long)`: con `Propagation.MANDATORY` y `FOR NO KEY UPDATE`, y vacío si la sesión no ve la unidad con `stock.ver` en su marca (`VehicleAccessPolicy.sees`). La regla de la visibilidad la decide `fleet`.
  - `Optional<FinancedUnit> fetchVisibleUnitByPublicId(UUID)` y `fetchVisibleUnitById(Long)`: lo mismo sin candado, para la lista y la respuesta de una escritura.
  - `Map<Long, FinancedUnit> lockUnitsOfCompanyByPlatesOrIds(Long companyId, Set<String> plates, Set<Long> vehicleIds)`: con `Propagation.MANDATORY`, en una sentencia y por `id` ascendente.
  - `Optional<String> normalizePlate(String written)`: la regla de `LicensePlate`, para que la carga marque una placa mal escrita sin copiar la regla.
- **El repositorio de la unidad.**
  - Se agrega `VehicleJpaRepository.lockByCompanyIdAndPlatesOrIds`.
  - Se borran `VehicleJpaRepository.lockByCompanyIdAndPlatesOrFund`, `VehicleRepository.findAllByCompanyIdAndPlatesOrAssignedToFundForUpdate` y su implementación en `VehicleRepositoryImpl`. Leen `vehicle_fund_assignments`, que deja de ser de `fleet`.
- **Quién usa el puerto:**
  - `VehicleAnnulments`: la guarda de la anulación. Por ella pasan la anulación explícita y la de las unidades que devuelve una nota de crédito (`ReturnedUnits`, que no cambia).
  - `VehicleAllowedActions`: `mayAnnul` pasa por `UnitFundFacts`. Con la opción A, `mayCede` delega en `offersCession`; con la B, se borra.
- **Solo con la opción B:** `registerFundAssignment` sale de `VehicleAllowedActionsView`, de `VehicleAllowedActionsResource` y de `VehicleResourceFromViewAssembler`.
- **Javadoc:** `package-info.java` de `fleet`.

#### Otros módulos

- **`reporting`** no cambia de código. `JdbcReportsReadRepository` sigue leyendo `vehicle_fund_assignments` para la columna «GARANTÍA MOBILIARIA» (`export.stock.column-33`), y `JdbcReportingDataVersions` sigue nombrándola. Su `package-info.java` dice que esa tabla se lee como modelo de lectura (*Conformist*).
- **Javadoc:** en `location/interfaces/acl/CurrencyUsage` y en `procurement` (`PurchaseOrderIssuing` y `ExternalOrganizationService`), «fleet» pasa a «financing» donde hablan de los fondos.
- **No cambian:**
  - `PermissionCodes.Funds`, `ImportType.GARANTIA_MOBILIARIA`, las claves de los mensajes (`export.funds.*` y `error.*`) y `scripts/database/hand-over-database.sql`, que no tiene líneas de fondos;
  - la bitácora: la entidad `vehicle_fund_assignments`, el campo `period` y ningún campo confidencial. `financing` sigue anotando la marca de la unidad en cada fila (D-102), que toma de `FinancedUnit`;
  - `ForeignAclStaysInTranslationLayerTests`: el adaptador implementa el puerto, y los records de `fleet` solo se nombran en la traducción.

#### Los candados

Pasan a una sección `financing` de los patrones de persistencia:

| Operación | Candados, en orden | Lo que lee después del candado |
|---|---|---|
| Ceder una unidad | La unidad, `FOR NO KEY UPDATE`, por la fachada de `fleet`; el fondo, `FOR SHARE` | Si la unidad está anulada y las cesiones de la unidad |
| Cerrar una cesión | La cesión sin candado, para saber su unidad; la unidad, por la fachada | La cesión, que vuelve a leer |
| La carga de garantía | El fondo, `FOR SHARE`; el candado consultivo de la carga, que no espera; la lectura de los ids de las unidades con una cesión abierta al fondo; en una sentencia y por `id`, las unidades de la empresa con las placas del archivo y esos ids, por la fachada | Las cesiones de esas unidades, de cualquier fondo |
| Crear un fondo | La empresa, `FOR SHARE`, por `resolveScope` de `organization` | Si la empresa está activa (D-84) |
| Editar un fondo o cambiar su estado | El fondo, `FOR NO KEY UPDATE`; al reactivar, la empresa `FOR SHARE`, después del fondo | Si alguna cesión usa el fondo (DP-12) |
| Anular una unidad (`fleet`) | La unidad | Su valor, sus contratos y, por el puerto, su cesión abierta |

- La carga toma el fondo antes que las unidades, como hoy: esta fase no lo introduce.
- Dos `FOR SHARE` sobre el fondo no chocan. La carrera con una edición del fondo en cola la cubre `DeadlockRetry`.
- Una cesión a mano que llega durante la carga espera en el candado de la unidad y ve lo que la carga escribió (FLT-R117). Lo mismo que hoy.

#### La API

- **Opción A:** nada cambia. Rutas, permisos, códigos de error y cuerpos son los mismos.
- **Opción B:**
  - `VehicleResource.allowedActions` pierde `registerFundAssignment`, también en las respuestas de las escrituras de la unidad, y se suma el subrecurso acordado;
  - la ficha sigue releyendo la unidad después de ceder o cerrar, porque `allowedActions.annul` depende de la cesión abierta;
  - se actualizan convenciones §4.2 (la unidad) y §6.2, y el §1 y el §8 del plan.

#### Frontend

- **La carpeta nueva `src/app/financing` recibe 43 archivos de `fleet`:**
  - `application/`: `funds.store.ts` y `vehicle-fund-assignments.store.ts`, con sus specs;
  - `domain/model/`: `fund.entity.ts`, `vehicle-fund-assignment.entity.ts` y `commands/{assign-vehicle-fund,change-fund-status,close-vehicle-fund-assignment,create-fund,update-fund}.command.ts`;
  - `infrastructure/api/`: `{change-fund-status,close-vehicle-fund-assignment,create-fund,create-vehicle-fund-assignment,update-fund}.request.ts`, `funds-api-endpoint.ts`, `funds-response.ts`, `vehicle-fund-assignments-api-endpoint.ts` y `vehicle-fund-assignments-response.ts`;
  - `infrastructure/assemblers/`: `fund-assembler.ts` y `vehicle-fund-assignment-assembler.ts`;
  - `presentation/components/`: `assign-fund-dialog`, `close-fund-assignment-dialog` y `fund-form-dialog` (cuatro archivos cada uno), `guarantee-import-dialog` (dos) y `vehicle-fund-assignments` (tres). Con A1, este último se queda en `fleet`, reescrito contra la fachada;
  - `presentation/views/funds-view` (cuatro).
- **Archivos nuevos en `financing`:**
  - `infrastructure/api/financing-api.ts`, que reemplaza en `FleetApi` lo de fondos y cesiones;
  - `infrastructure/api/guarantee-imports-api-endpoint.ts` y `guarantee-imports-response.ts`, `domain/model/guarantee-import.ts` (`GuaranteeEffects` y `GuaranteeClosures`) y `application/guarantee-imports.store.ts` con su spec;
  - `application/financing.facade.ts`, con su spec:
    - `openGuaranteeImport(ImportTarget)`, que hoy está en `FleetFacade` y usa la pantalla Cargas;
    - con A1, `loadUnitAssignments`, `cedeUnit` y `closeAssignment`, con tipos propios, carga y error separados de las pantallas del dueño (regla 1.3) y la señal de lo que cambió mientras tanto;
  - `presentation/financing.routes.ts`, con su spec: `funds`, con `data.permissions` `anyOf(PermissionCodes.Funds.VIEW)`;
  - un tipo propio de filtros para las cesiones de una unidad. Hoy se importa `VehicleRecordFilters` de `fleet/application/vehicles.store`, y desde otro contexto eso viola la regla 1.1.
- **Archivos que se parten o cambian:**
  - `fleet/infrastructure/api/fleet-api.ts`: salen los extremos de fondos y cesiones;
  - `fleet/infrastructure/api/fleet-imports-api-endpoint.ts` (`GuaranteeImportRequest` y `uploadGuaranteeList`), `fleet-imports-response.ts` (`GuaranteeEffectsResource` y `GuaranteeImportResource`) y `fleet/domain/model/fleet-import.ts` (`GuaranteeEffects` y `GuaranteeClosures`);
  - `fleet/application/fleet-imports.store.ts` y `fleet.facade.ts` (`openGuaranteeImport`), con sus specs;
  - `fleet/presentation/views/vehicle-profile-view/vehicle-profile-view.{ts,html,spec.ts}`: deja de importar el store y los componentes de cesiones, y usa `FinancingFacade` (A1) o la pieza prestada (A2);
  - solo con la opción B: `vehicle-allowed-actions.ts`, `vehicles-response.ts`, `vehicle-assembler.ts` con su spec, y `vehicles.store.spec.ts`;
  - `fleet/presentation/fleet.routes.ts` y su spec: `{ path: 'funds', redirectTo: '/financing/funds' }`, que conserva `?empresa=`, con una spec que lo pruebe;
  - `app.routes.ts`: `{ path: 'financing', loadChildren: financingRoutes }`;
  - `app-shell/app-navigation.ts`: el enlace de Fondos va a `/financing/funds`, con el mismo grupo y el mismo acceso. Se ajusta `app-shell.spec.ts`, en la lista de fondos y en la comparación de rutas con el menú;
  - `imports/presentation/views/imports-view/imports-view.ts`: abre la garantía con `FinancingFacade.openGuaranteeImport`;
  - `environments/environment.ts` y `environment.development.ts`: `fleetFundsEndpointPath` y `fleetFundAssignmentsEndpointPath` pasan a `financing…`.
- **i18n.** Se conservan las claves `fleet.funds.*`, `fleet.guarantee-import.*`, `fleet.profile.fund-assignments.*`, `navigation.funds` y `modules.funds.*`, que son contrato por nombre, como en el backend. Si se renombran a `financing.*`, se renombran todas en el mismo commit.
- **e2e.**
  - `e2e/screens.ts`: la pantalla `funds` pasa a `/financing/funds`; `vehicle-profile-assignments` no cambia de ruta.
  - Specs: `funds.spec.ts` (las rutas, el selector `mm-vehicle-fund-assignments` y cuándo se ofrece ceder), `exports.spec.ts` (la exportación de fondos) e `imports.spec.ts` (la garantía).
  - Mocks: las rutas de fondos y cesiones de `e2e/support/mocks/fleet.ts` pasan a un `mocks/financing.ts`, registrado en `e2e/support/api.ts` respetando el orden de los manejadores de Playwright. `registerFundAssignment` en el mock de la unidad cambia según la opción; `/fund-assignment-imports` sigue en `mocks/imports.ts`.
  - Capturas: `funds-*` y `vehicle-profile-assignments-*` conservan su nombre y se regeneran en el CI solo si cambia lo dibujado.
  - Sondas: `e2e-live/funds.live.spec.ts` navega a `/financing/funds`. `imports.live.spec.ts` no cambia, porque usa la API.

#### Pruebas

- **Se mueven a `financing`:** `FundRacedCodeTests`, `GuaranteeListPolicyTests`, `FundAuditTests`, `FundLockTests`, `VehicleFundAssignmentAuditTests`, `FundsExportIntegrationTests`, `FundsIntegrationTests` y `VehicleFundAssignmentsIntegrationTests`. Siguen usando `FleetFixtures` y `AccessFixtures`.
- **Se parten:**
  - de `FleetImportsIntegrationTests`, los casos de la garantía van a `FundAssignmentImportsIntegrationTests`;
  - de `FleetRowLevelSecurityTests`, los dos casos de fondos y cesiones van a `FinancingRowLevelSecurityTests`;
  - de `FleetRowsAreNeverDeletedTests`, las entradas `funds` y `vehicle_fund_assignments` van a `FinancingRowsAreNeverDeletedTests`.
- **Se reubican o ajustan:**
  - `VehicleAnnulmentLockTests` pasa a una prueba de la raíz (`ky.millennium.mobility.erp`), que juega la anulación contra la cesión por la API o por las interfaces públicas, sin importar el interior de ninguno de los dos;
  - `VehicleAllowedActionsTodayTests` se ajusta al puerto. Con la opción B, el caso de «hoy en la zona de la empresa» pasa a `financing`;
  - `VehicleAllowedActionsTests`, el caso de la cesión: con la opción A se queda; con la B pasa a `financing`;
  - `VehicleAnnulmentsIntegrationTests`: con la opción B sale la línea de `registerFundAssignment`.
- **Se corren sin cambios:**
  - `FleetContextFacadeImplTests`: la unidad devuelta con su cesión abierta;
  - `CreditNotesIntegrationTests`: la nota que devuelve una unidad cedida;
  - `RowLevelSecurityMigrationTests`, las de reporting y las de arquitectura.
- **Nuevas:**
  - `UnitFundAssignmentsWiringTests`: un solo bean, el de `financing`;
  - las de los miembros nuevos de la fachada de `fleet`: candado por `id` ascendente, la fila después del candado y la visibilidad por marca;
  - la columna «GARANTÍA MOBILIARIA» del exportable de stock, que hoy ninguna prueba cubre;
  - con la opción B, las del subrecurso nuevo.

#### Pasos dentro de la fase

Cada paso compila y pasa sus pruebas antes del siguiente. La fase se integra completa.

1. **`fleet`:** el puerto, `UnitFundFacts` y un adaptador provisional en `fleet` sobre su propio repositorio. `VehicleAnnulments` y `VehicleAllowedActions` pasan por él.
2. **`fleet`:** los miembros nuevos de la fachada y `lockByCompanyIdAndPlatesOrIds`, con sus pruebas.
3. **Todavía dentro de `fleet`:** los servicios de fondos y la carga de garantía se reescriben para usar solo la fachada y el puerto, sin `Vehicle`, `VehicleRepository`, `VehicleAccessPolicy`, `FleetSessionAccount` ni `CompanyCalendar`. Se borra la consulta que bloquea por placas o por fondo.
4. **Se crea `financing`:** se mueven las 54 clases y las piezas que se parten, se crean las clases nuevas con sus nombres de bean y el adaptador del puerto pasa a `financing`.
5. **Solo con la opción B:** el cambio del recurso de la unidad y el subrecurso nuevo.
6. **Las pruebas.**
7. **El frontend.**
8. **La documentación.**

#### Documentación

- `contextos/financing.md`, con lo de fondos de `fleet.md` y los mismos ids (`FLT-R…`), y `contextos/README.md`.
- El mapa vigente (con la relación en los dos sentidos) y el diccionario §3.
- Los patrones de persistencia: la regla de lo que se referencia en modo compartido y la sección nueva de `financing` con la tabla de arriba, que sale de la de `fleet`.
- Procesos y pantallas: Fondos en `/financing/funds`.
- Convenciones, solo con la opción B (§4.2 y §6.2).
- Las cuatro copias de las skills: en `backend-module`, §1 (la carga de garantía es de `financing`) y §2 (la lista de `allowedDependencies`); y en el frontend, la regla 1.2 si se elige A2.

### Fase 5 — El servicio publicado de `procurement`

**Por qué.** Hoy `fleet` intermedia los montos de `procurement`. Su puerto `UnitPurchases` define en el lenguaje de `fleet` el costo, las notas, los usos del saldo y los pagos con sus montos. `fleet` sirve `GET /vehicles/{id}/purchase-cost` y le reexpone el costo a `reporting` (`FleetContextFacade.fetchPurchaseCosts`).

El dueño del costo es `procurement`, y debe publicarlo él (*Open Host Service* con *Published Language*, cap. 14): por REST para la ficha, y por `procurement::acl` para `reporting` y, más adelante, para la conciliación de la inicial (D-31). Lo que `fleet` necesita para sus propias reglas sigue llegando por su puerto.

**Decisión previa (fase 0): quién es dueño de la brecha de días.** Un reporte dice lo que dice la ficha (D-232), así que la regla tiene un solo dueño:
- **A (recomendada):** sigue en `fleet` (`DayGapPolicy`, FLT-R131). El puerto conserva el estado de pago y la fecha base, sin montos (D-212). El recurso de la unidad no cambia.
- **B:** `fleet` deja de saber de pagos, y la brecha pasa a `procurement`, nunca a `reporting`. Cambia el recurso de la unidad. Lo que agrega B está al final de esta fase.

**Backend (A).**

- **`procurement::acl`.** Si la fase 3 no la creó, esta fase crea:
  - `procurement/interfaces/acl/package-info.java`, con `@NamedInterface("acl")` y `@NullMarked`;
  - `ProcurementContextFacade`;
  - `procurement/application/acl/ProcurementContextFacadeImpl`.

  Suma un solo miembro, por lote, porque su único consumidor es `reporting`:
  ```java
  /**
   * El costo de compra de cada unidad con línea vigente (D-159, D-199, D-232, D-248), en una lectura por tipo de hecho,
   * en la transacción de quien llama y bajo su Row Level Security. Una unidad sin línea vigente no está.
   */
  Map<Long, UnitPurchaseCost> fetchUnitPurchaseCosts(Set<Long> vehicleIds);
  ```
  Sus records salen de `UnitPurchases`: `UnitPurchaseCost`, `UnitCreditNote` **sin** `usedIn`, `UnitPaymentPosition`, `UnitPaymentRecord` y `UnitAmount`. `UnitBalanceUse` y `usedIn` no se publican: solo los usa la lectura de una unidad, que es de `procurement`.

- **El cálculo** sale de `PurchaseInvoiceUnitPurchasesAdapter` (`costs`, `paymentOf`, `creditNotes`, `usedIn`) a un componente de `procurement/application/internal/queryservices` (por ejemplo, `UnitPurchaseCosts`). Lo usan la fachada y la ruta. El adaptador queda solo con el puerto reducido.

- **`UnitPurchases` de `fleet` queda en:** `hasCurrentLine`, `withCurrentLine`, `unitIdsWithCurrentLine`, `valueIsZero`, `returnedUnitIds`, `paymentFacts` con `UnitPaymentFacts(status, baseDate)`, y `fetchDataVersion`.
  - **Salen** `purchaseCostOf`, el método por omisión `purchaseCostsOf` y los records `UnitPurchaseCost`, `UnitPaymentPosition`, `UnitPaymentRecord`, `UnitCreditNote`, `UnitBalanceUse` y `UnitAmount`.
  - `UnitPurchaseFacts` pierde `purchaseCostOf` y `purchaseCostsOf`.

- **`FleetContextFacade`** pierde `fetchPurchaseCosts` y suma un miembro, para que `procurement` no lea `vehicles` (regla 1.2) ni copie la regla de quién ve una unidad:
  ```java
  /**
   * La unidad que quien llama ve con stock.ver en su empresa y su marca (VehicleAccessPolicy.sees), o vacío si no existe o
   * no la ve. Lee bajo el Row Level Security de quien llama, en su transacción.
   */
  Optional<SeenUnit> fetchSeenUnit(UUID vehiclePublicId);
  ```
  El record va en `fleet/interfaces/acl`:
  ```java
  /** @param seesCosts si stock.ver_costos actúa en su empresa y su marca (VehicleAccessPolicy.seesCosts) */
  public record SeenUnit(Long id, UUID publicId, Long companyId, Long commercialBrandId, boolean annulled, boolean seesCosts) {
  }
  ```

- **`GET /vehicles/{id}/purchase-cost`** pasa a `procurement/interfaces/rest/UnitPurchaseCostsController`. No puede llamarse `VehiclesController`, porque choca como bean con el de `fleet`.
  - Lleva `@RequestMapping("/api/v1/vehicles")` y `@PermissionRequired(PermissionCodes.Stock.VIEW_COSTS)`.
  - Su servicio de consultas es `@Transactional(readOnly = true)`: fuera de una transacción, la cuenta de la aplicación no ve filas de ninguna empresa y el costo saldría `null` sin error.
  - Revisa en el orden de hoy (convenciones §7.2):
    1. el `403` de la ruta;
    2. la unidad por `fetchSeenUnit` (`404 VEHICLE_NOT_FOUND`, `ApplicationError.notFound("Vehicle", id)`);
    3. `seesCosts` (`403 ACCESS_DENIED`, con el texto de `FleetErrors.deniedOnUnit`).
  - `returnedToSupplier` es «no anulada» y «devuelta por una nota vigente».
  - `usedIn` viaja solo para quien tiene `pagos.ver` en la empresa de la factura.
  - Conserva los nombres de esquema (`VehiclePurchaseCost`, `VehiclePurchaseAmount`, `VehiclePurchaseInvoice`, `VehiclePurchaseCreditNote`, `VehiclePurchaseCreditNoteUse`, `VehiclePurchase`, `VehiclePurchasePayment`, `VehiclePurchasePaymentRecord`) y la descripción OpenAPI.

- **En el mismo commit sale de `fleet`:** `VehiclesController.getVehiclePurchaseCost`, `GetVehiclePurchaseCostQuery`, `VehicleQueryService.handle(GetVehiclePurchaseCostQuery)`, los traductores de `VehicleQueryServiceImpl`, `VehiclePurchaseCostView`, `VehiclePurchaseCostResource` y su ensamblador. Dos métodos con la misma ruta impiden arrancar.

- **Se quedan en `fleet`:**
  - `allowedActions.viewPurchaseCost`, que es la misma regla que `seesCosts` y también usa la pantalla de habilitación para el costo de los equipos;
  - `hasCurrentInvoice`, `returnedToSupplier` y `payment` del recurso de la unidad;
  - el filtro `hasCurrentInvoice` del Inventario;
  - `DayGapPolicy`;
  - `fetchReportFacts`, con su fecha base y su brecha;
  - la versión de la exportación del Inventario, que sigue sumando `UnitPurchases.fetchDataVersion()` por la columna «con factura vigente».

- **`reporting`:**
  - suma `"procurement::acl"` a su `package-info.java`;
  - suma `ExternalProcurementService`, con `@Service("reportingExternalProcurementService")` y `fetchPurchases(Set<Long>)`, que es la traducción a `ReportUnitPurchase` que hoy hace `ExternalFleetService` (este la pierde);
  - `ReportQueryServiceImpl` la usa en la diferencia por inicial y en la exportación de stock;
  - `ReportingDataVersions` no cambia: ya nombra las tablas de compras.

- **Dependencias:** `reporting` → `procurement::acl` es nueva; `procurement` → `fleet::acl` ya existe; `fleet` nunca depende de `procurement::acl`, porque cerraría un ciclo.

- **Transacciones y candados:** todo es lectura, en la transacción de solo lectura de quien llama. No hay candados nuevos.

**Frontend (A).**

- **`procurement`** recibe de `fleet`:
  - `vehicle-purchase-cost.entity.ts`, `vehicle-purchase-cost-response.ts` y `vehicle-purchase-cost-assembler.ts` con su spec;
  - la lectura de `GET /vehicles/{id}/purchase-cost`, que hoy está en `VehiclesApiEndpoint.getPurchaseCost` y `FleetApi.getVehiclePurchaseCost`.

  Un store nuevo, `unit-purchase-costs.store.ts`, guarda el costo con `loadingExternal` y `externalError`.

- **`ProcurementFacade`** suma estos miembros, con tipos definidos junto a ella (nunca la entidad del dominio):
  - `unitPurchaseCost: Signal<UnitPurchaseCostView | null>`
  - `unitPurchaseCostLoading: Signal<boolean>`
  - `unitPurchaseCostError: Signal<string | null>`
  - `unitPurchaseCostDenied: Signal<boolean>`
  - `loadUnitPurchaseCost(vehicleId: string): void`
  - `clearUnitPurchaseCostError(): void`

  No lleva miembros por lote: nadie los usa, porque el Inventario no muestra el estado de pago. Su spec incluye el caso «una lectura ajena no deja su error en la pantalla dueña».

- **`fleet`:**
  - `VehiclePurchaseCostPanel` lee de `ProcurementFacade` y decide «Reintentar» por `unitPurchaseCostDenied`, no comparando una clave de otro contexto.
  - `VehiclePurchaseCostBreakdown`, `breakdowns-catalog-sample.ts` y `fleet-design-system-samples.ts` usan `UnitPurchaseCostView`.
  - `VehiclesStore` pierde `purchaseCost`, `purchaseCostError`, `loadPurchaseCost` y `clearPurchaseCostError`.
  - En `public/i18n/es.json`, las claves `fleet.purchase-cost.error-*` pasan a `procurement.unit-purchase-cost.*`.
  - La banda de pago de la ficha y el Inventario no cambian.

- **e2e:** la ruta es la misma, así que los mocks no cambian; las specs de la ficha prueban la fachada.

**Pruebas de caracterización, antes de mover nada (en `develop`, contra `fleet`).** Pasan igual después de cada paso:
1. El JSON completo de `GET /vehicles/{id}/purchase-cost`:
   - de una unidad en dólares con dos notas, un pago parcial y un uso del saldo, leído con y sin `pagos.ver` (`usedIn` presente o `null`);
   - de una unidad sin línea vigente (`purchase: null`).
2. El orden de las negativas:
   - sin `stock.ver_costos` en ninguna empresa: `403`, aunque la unidad no exista;
   - con `stock.ver_costos` pero sin `stock.ver` sobre la unidad: `404 VEHICLE_NOT_FOUND`;
   - con los dos en la empresa pero los costos solo en otra marca: `403 ACCESS_DENIED`.
3. `returnedToSupplier`: devuelta y sin anular, `true`; anulada, `false`.
4. La exportación de stock, la diferencia por inicial y la brecha de una empresa con unidades con notas y pagos, con `stock.ver_costos` en todas sus marcas y en una sola: las columnas de costo y de pago, celda por celda.

**Pruebas que cambian.**
- **Se mudan a `procurement`:** `VehiclePurchaseCostIntegrationTests` (salvo `viewPurchaseCostFollowsTheCostsOfItsBrand`, que es de `fleet`) y los casos de costo de `PurchaseInvoiceUnitPurchasesAdapterTests`.
- **Cambian:** `UnitPurchaseFactsTests`, porque su doble `Listed` implementa los métodos que salen.
- **Se corren:** `BalanceTraceIntegrationTests`, `SupplierPaymentsIntegrationTests`, `UnitLevelBalanceIntegrationTests`, `UnitPurchasesWiringTests`, `VehicleInvoiceFactsIntegrationTests`, `ReportsIntegrationTests`, `StockRowsTests`, `ProcurementOpenApiTests` y las de arquitectura (`ModularityTests`, `LayerDependencyTests`, `ForeignAclStaysInTranslationLayerTests`, `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`).
- **Nuevas:** `fetchSeenUnit` (marca, anulada, sin ver) y la fachada de `procurement` comparada con el resultado del adaptador viejo.
- **Frontend:** las specs del store y la fachada nuevos, del panel, del ensamblador y de la ficha.
- **e2e:** `fleet.spec.ts`, `credit-notes.spec.ts` y `supplier-payments.spec.ts`. **e2e-live:** `credit-notes.live.spec.ts`.

**Orden dentro de la fase.**
1. Las pruebas de caracterización, en verde en `develop`.
2. `procurement::acl`, si falta, y el componente de cálculo con `fetchUnitPurchaseCosts`, probados contra el adaptador viejo.
3. `fetchSeenUnit` en `fleet`.
4. La ruta en `procurement` y su retiro de `fleet`, en el mismo commit.
5. `reporting` lee de `procurement::acl`.
6. Se reducen `UnitPurchases`, `UnitPurchaseFacts` y `FleetContextFacade`.
7. Frontend.
8. Documentación y las cuatro copias de las skills (`.claude` y `.cursor` del backend y del frontend).

**Documentación.**
- Una decisión nueva del diccionario que reemplaza D-203: la ruta la sirve `procurement`, que lee la unidad de `fleet::acl`.
- Convenciones: §2.2 (la ruta pasa de la fila de `fleet` a la de `procurement`, y las líneas de `purchase-cost`, `hasCurrentInvoice` y `returnedToSupplier`), §4.2, §7.2 (el orden de `purchase-cost`), §9.2 y §14.19.
- `contextos/fleet.md`: el cuadro de lo que no es suyo, FLT-R53, FLT-R96, FLT-R130, FLT-R146 y la sección de `UnitPurchases`.
- `contextos/procurement.md`: PRC-R62, PRC-R90 y la fachada nueva.
- `contextos/reporting.md`: §1 y §5, con sus dependencias.
- El mapa vigente: §4, §5 y §6.
- El mapa objetivo: §3.1 y §4. El puerto reducido también responde el estado de pago y la fecha base, sin montos.
- `procesos-y-pantallas`: el panel «Compra y costo».
- Las skills: `SKILL.md` §1 y §2 (las dependencias de `reporting`) y `architecture-rules.md` 1.2.

**Terminado cuando:**
- `fleet` no nombra costos, notas ni montos de pagos;
- `GET /vehicles/{id}/purchase-cost` responde el mismo JSON con el mismo orden de negativas;
- los reportes dan las mismas celdas;
- ninguna prueba de caracterización cambió.

**Si se elige B, además:**
- `UnitPurchases` pierde también `paymentFacts` y `UnitPaymentFacts`.
- `DayGapPolicy` y `DayGapPolicyTests` pasan a `procurement/domain/services`.
- `procurement::acl` suma un miembro, con la brecha calculada por `procurement` con la primera entrega que da `fleet` y la zona de la empresa (`America/Lima` si no se lee, como `CompanyCalendar`):
  ```java
  Map<Long, UnitPaymentFacts> fetchUnitPaymentFacts(Set<Long> vehicleIds);

  record UnitPaymentFacts(String status, LocalDate baseDate, long dayGap) {
  }
  ```
- `fleet::acl` suma el primer paso a Activo (FLT-R58), y `SeenUnit` suma `@Nullable Instant firstDeliveredAt`:
  ```java
  Map<Long, Instant> fetchFirstDeliveries(Set<Long> vehicleIds);
  ```
- `UnitReportFacts` y `ReportUnitFacts` pierden `baseDate` y `dayGap`. `UnitReportFactsReader` pierde `UnitPurchaseFacts`, `DayGapPolicy` y `CompanyCalendar`. `ReportQueryServiceImpl.gapOf` y la columna de la brecha de `StockRows` leen de `ExternalProcurementService`.
- Ruta nueva `GET /vehicles/{id}/payment-status` en `procurement`, con `@PermissionRequired(PermissionCodes.Stock.VIEW)`: `404 VEHICLE_NOT_FOUND` si `stock.ver` no ve la unidad, y `200 {vehicleId, payment: {status, baseDate, dayGap} | null}`. Sin versión por lote: el Inventario no muestra el pago.
- `fleet` pierde `VehicleView.payment` (`withRecord(enablement, actions)`), `VehiclePaymentView`, `VehiclePaymentResource`, `VehicleResource.payment` y `VehicleViewFactory.paymentOf`. Es un cambio del recurso de la unidad (convenciones §4.2), de D-212 y de FLT-R131.
- **Frontend:**
  - `Vehicle`, `vehicles-response.ts` y `vehicle-assembler.ts` pierden `payment`, y `vehicle-payment.ts` pasa junto a la fachada;
  - `ProcurementFacade` suma `unitPaymentStatus`, su carga, su error y `loadUnitPaymentStatus(vehicleId)`;
  - la ficha vuelve a pedir la banda cada vez que el store reemplaza la unidad, porque un paso a Activo congela la brecha;
  - `e2e/support/mocks/fleet.ts` saca `payment` del registro y atiende la ruta nueva;
  - se corre `e2e-live/supplier-payments.live.spec.ts`.
- **Caracterización adicional:**
  - la banda de hoy (`$.payment` de la ficha) es igual a la de la ruta nueva para una unidad entregada, una sin entregar, una con `stock.ver` limitado a su marca y una sin línea vigente;
  - `GET /reports/day-gaps` da las mismas filas.
- **Documentación:** además D-212, FLT-R131 y el mapa objetivo §3.1 y §3.14 (la brecha la calcula `procurement`, y `reporting` la lee).

### Fase 6 — Las cuentas bancarias se quedan en `organization`, con un puerto por consumidor

**Por qué no se mueven.**
- **Un concepto que van a usar dos contextos hermanos va aguas arriba** (cap. 14). Hoy las cuentas de la empresa solo las usan los pagos a proveedores, pero el bloque 12 trae `collections`, y el mapa objetivo (§3.4) le da la conciliación bancaria y los pagos recibidos. Una cuenta de la empresa también recibe cobros. Si las cuentas fueran de `procurement` (cuentas por pagar), `collections` (cuentas por cobrar) tendría que depender de él para un concepto que no es de pagos, y el mapa (§3.4 y §6) dice que esos dos modelos son opuestos.
- **Son datos de lo que la empresa tiene, no de una operación** (cap. 15). Su lugar es `organization`, en la capa de potencial (mapa objetivo §5), donde ya están la empresa, sus marcas y sus sedes. La cuenta no lleva saldos ni movimientos (D-214).
- **Cada consumidor responde por un puerto que declara `organization`** (cap. 16), como `location` con `CurrencyUsage`. La regla que necesita los datos de otro contexto —ORG-R65: una cuenta con registros ya no cambia su número ni su moneda— se cumple igual, en la misma transacción y bajo el candado de la cuenta, sin mover el agregado.
- **El diccionario ya lo dice:** §3 pone las cuentas en el bloque 3, de `organization` (§6.2.1, `company_bank_accounts`). Si el mapa no coincide con el diccionario, manda el diccionario.

**Qué cambia.** El puerto `BankAccountPayments` pasa a llamarse por lo que pregunta —si una cuenta está en uso— y no por quien hoy lo responde. Así `collections` lo implementa después sin tocar `organization`. `BankAccountPaymentFacts` ya junta una lista de beans, como `CurrencyUsage`; lo que falta es el nombre, el contrato escrito y la prueba que lo protege.

#### Backend — `organization`

- **El puerto.** `organization/interfaces/acl/BankAccountPayments` pasa a `BankAccountUsage`:

  ```java
  public interface BankAccountUsage {
      /**
       * @param bankAccountIds internal identifiers of accounts, such as the ones of a page
       * @return those of them that a record of this context names, current or annulled
       */
      Set<Long> accountsInUse(Set<Long> bankAccountIds);
  }
  ```

  Su javadoc fija el contrato de quien lo implemente:
  - **Responde con el alcance de quien llama**, por la RLS: una cuenta y los registros que la nombran son de la misma empresa (llave `(bank_account_id, company_id)`, V110), así que quien alcanza la cuenta alcanza sus registros. No usa `SYSTEM`, a diferencia de `CurrencyUsage`, que pregunta por una moneda de todas las empresas.
  - **Cuenta también lo anulado:** un pago anulado salió igual de su cuenta y conserva el número y la moneda con que se registró.
  - **Lee una vez por llamada,** en lote: una página de cuentas pregunta una sola vez.
  - **Toda escritura que nombra una cuenta la lee antes en modo compartido**, con `fetchBankAccountForShare` de la fachada, y así la corrección del número o de la moneda, que bloquea la cuenta para actualizar, la ve o la espera. Hoy lo hace el pago; lo hará el cobro de `collections`.
- **El envoltorio.** `application/internal/outboundservices/payments/BankAccountPaymentFacts` pasa a `application/internal/outboundservices/usage/BankAccountUsageFacts`, con `Set<Long> inUse(Set<Long>)`:
  - recibe `List<BankAccountUsage>` y une lo que responde cada bean;
  - sin pedido no pregunta, y sin ningún bean ninguna cuenta está en uso, como hoy.
- **Quienes lo usan.**
  - `BankAccountCommandServiceImpl`: `hasPayments(accountId)` pasa a `inUse(accountId)`. La corrección sigue respondiendo `422 BANKACCOUNT_IN_USE`, un código que ya es genérico.
  - `BankAccountViewFactory`: `paymentFacts.withPayments(…)` pasa a `usageFacts.inUse(…)`.
- **Javadoc:** `package-info.java` de `organization` nombra `BankAccountUsage`, que implementan `procurement` y, más adelante, `collections`.

#### Backend — `procurement`

- **El adaptador.** `infrastructure/organization/SupplierPaymentBankAccountPaymentsAdapter` pasa a `SupplierPaymentBankAccountUsageAdapter implements BankAccountUsage`. Responde con `SupplierPaymentRepository.findBankAccountsWithPayments`, que no cambia: los pagos de cualquier estado.
- **Sin otros cambios.** `ForeignAclStaysInTranslationLayerTests` no cambia: la clase implementa el puerto.

#### El riesgo de la lista vacía

`organization` no depende de quien implementa el puerto, así que el arranque no puede exigir que haya un bean: `location` tampoco lo exige con `CurrencyUsage`. Pero si el bean de `procurement` faltara —se renombra o se mueve sin la anotación—:
- ninguna cuenta quedaría en uso;
- ORG-R65 dejaría de cumplirse sin que nada falle: se podría cambiar el número o la moneda de una cuenta de la que ya salieron pagos;
- `hasPayments` diría `false`.

Lo protege una prueba de arranque, no una cuenta de beans:
- `BankAccountPaymentsWiringTests` pasa a `BankAccountUsageWiringTests`.
- Exige que la lista **contenga** a `SupplierPaymentBankAccountUsageAdapter`. Hoy exige que sea el único elemento (`singleElement`), lo que haría fallar la prueba cuando llegue el bean de `collections`.
- Sigue comprobando que sin cuentas no pregunta nada.

#### Lo que no cambia

- **Las tablas y sus migraciones:** `company_bank_accounts` (V108), su RLS, su disparador `keep_company_bank_account_company()` y su línea en `hand-over-database.sql`, la llave de los pagos (V110) y sus casillas en `data_versions`.
- **Las rutas y los permisos:**
  - `GET` y `POST /companies/{companyId}/bank-accounts`, `GET` y `PUT /bank-accounts/{id}` y `PATCH /bank-accounts/{id}/status`, todas con `configuracion.ver_estructura`, y las escrituras con `configuracion.crear_cuentas_bancarias`, `editar_cuentas_bancarias` y `desactivar_cuentas_bancarias`;
  - `GET /supplier-payment-accounts`, con `pagos.ver`.
- **El contrato:** el campo `hasPayments` de la cuenta, los códigos `BANKACCOUNT_*` y las claves de los mensajes. Cuando exista `collections`, las convenciones deciden si `hasPayments` se renombra o se suma `inUse`.
- **La regla del número completo (ORG-R66):**
  - el número lo lee solo quien gestiona las cuentas de esa empresa, con uno de sus tres permisos en toda ella;
  - los demás leen los últimos cuatro, y no buscan ni ordenan por el número;
  - lo que sale por la fachada (`BankAccountSummary`) lleva solo `lastDigits`, así que ni los pagos ni otro contexto reciben el número completo.
- **La fachada de `organization`** (`fetchBankAccountForShare`, `fetchBankAccountsByIds`, `fetchBankAccountsByPublicIds` y `fetchBankAccountsOfCompany`) y su uso en `ExternalOrganizationService` de `procurement`.
- **El orden de los candados del pago:** la empresa, las facturas, el proveedor, la moneda, las órdenes y al final la cuenta, `FOR SHARE`, por la fachada de `organization`.
- **Las versiones de datos:** `company_bank_accounts` sigue en `JdbcOrganizationDataVersions.whatTheFacadeResolves()`, que versiona la exportación de los pagos.
- **El frontend:** la pestaña Cuentas bancarias de la ficha de la empresa (`?seccion=cuentas`), sus 20 archivos en `organization`, el formulario de pago y sus pruebas e2e. No cambia ningún archivo.

#### Pruebas

- **Se renombra y se ajusta:** `BankAccountPaymentsWiringTests` pasa a `BankAccountUsageWiringTests`, con «contiene» en vez de «único».
- **Nuevas:**
  - `BankAccountUsageFactsTests`, sin Spring: con dos beans de prueba, une sus respuestas; sin beans, ninguna cuenta está en uso; sin cuentas, no pregunta;
  - `BankAccountLockTests`: un pago con transferencia que todavía no confirma hace esperar la corrección del número de su cuenta, que después responde `BANKACCOUNT_IN_USE`; una desactivación de la cuenta espera al pago. Hoy ninguna prueba cubre el candado compartido de la cuenta.
- **Se corren:**
  - `BankAccountTests`, `BankAccountAuditTests`, `BankAccountRowLevelSecurityTests` y `BankAccountsIntegrationTests`;
  - `SupplierPaymentAccountsIntegrationTests` (`hasPayments` y `BANKACCOUNT_IN_USE`), `SupplierPaymentsIntegrationTests`, `SupplierPaymentsExportIntegrationTests` y `SupplierPaymentLockTests`;
  - las de arquitectura (`ModularityTests`, `LayerDependencyTests` y `ForeignAclStaysInTranslationLayerTests`).
- **Frontend:** nada. No cambia ningún archivo.

#### Documentación

- **`contextos/organization.md`:**
  - «El puerto que declara» pasa a `BankAccountUsage`, con su contrato y quiénes lo implementan;
  - la nota de los pagos de una cuenta;
  - ORG-R65 conserva su id y dice «una cuenta que nombra un registro de otro contexto, vigente o anulado —hoy un pago—».
- **`contextos/procurement.md`:** en la tabla de los puertos que implementa, `BankAccountUsage` en lugar de `BankAccountPayments` (PRC-R164 no cambia).
- **El mapa vigente:** el párrafo de `company_bank_accounts`, «`organization` no llama a `procurement`» y la fila del puerto en la tabla de las fachadas.
- **El mapa objetivo:**
  - §3.6: las cuentas bancarias salen de la responsabilidad de `procurement`, que suma `BankAccountUsage` a los puertos que implementa;
  - §3.9: `organization` conserva las cuentas y declara `BankAccountUsage`; sale «las cuentas bancarias salen porque…»;
  - §4: la relación `procurement → organization` por `BankAccountUsage`, y la futura `collections → organization`, y la última frase nombra `BankAccountUsage` en lugar de `BankAccountPayments`;
  - §7: la decisión 3 queda cerrada, porque la pantalla no cambia.
- **El diccionario:** sin cambios; §3 y §6.2.1 ya ponen las cuentas en `organization`.
- **Los patrones de persistencia:** una línea en la regla de lo que se referencia en modo compartido: toda escritura que nombra una cuenta la lee `FOR SHARE` por la fachada, también la que traiga `collections`.
- **Las skills:** no nombran el puerto; no cambian.

**Costo y riesgo.** Menos de un día; riesgo bajo. Depende de la fase 0. No cambia comportamiento, contrato ni pantalla.

### Fase 7 — La salida y el regreso del colaborador, en `organization`

**Por qué.** La salida y el regreso son hechos del colaborador (D-38). El contexto dueño de un término es dueño del caso de uso que lo cambia (cap. 14), y el cambio entra por la raíz `Employee` (cap. 6).

Hoy `iam` sirve las rutas y decide quién puede con reglas que son de `organization`:
- el permiso donde está el colaborador (D-122);
- nadie registra la suya (D-70);
- quien opera tiene que estar vinculado a un colaborador activo (ORG-R50).

`EmployeeAllowedActions` ya calcula esas reglas. Además, `iam` le pide a `organization` registrar el hecho por dos miembros de la fachada que solo existen para eso.

Esta fase reemplaza la decisión 2, opción A, del mapa vigente, con una decisión nueva del diccionario.

**Por qué un puerto y no un evento.** El sistema no publica eventos de integración, la negativa del último superadministrador tiene que llegar como `Result` antes de escribir, y la consecuencia sobre la cuenta se confirma o se revierte con la salida. La consistencia síncrona entre dos agregados de dos contextos es una excepción deliberada (cap. 6), que justifica la invariante de seguridad: quien sale no conserva el acceso. Es el patrón que ya usan `UnitPurchases`, `CurrencyUsage` y `BankAccountPayments`, que la fase 6 generaliza a `BankAccountUsage`: el contexto de abajo declara el puerto y el de arriba lo implementa.

**Lo que no cambia:**
- las rutas;
- el permiso `seguridad.registrar_salidas_y_regresos`, donde está el colaborador;
- el cuerpo y la respuesta (`EmployeeTenureResponse`: `id`, `isActive`, `hireDate`, `terminationDate`);
- los códigos y su orden;
- los textos OpenAPI;
- lo que le pasa a la cuenta (IAM-R21, IAM-R40 a IAM-R43);
- el orden de los candados.

**Backend.**

- **Servicio nuevo en `organization`:** `application/commandservices/EmployeeTenureCommandService` y `application/internal/commandservices/EmployeeTenureCommandServiceImpl`, con `@Transactional`. Queda envuelto por `DeadlockRetry`. No va en `EmployeeCommandServiceImpl`, por dos razones:
  - pasa de 300 líneas;
  - `OrganizationContextFacadeImpl` lo inyecta, y un puerto implementado por `iam` en él podría cerrar un ciclo de beans.

- **Controladores:** `EmployeeTerminationsController` y `EmployeeReactivationsController` pasan a `organization/interfaces/rest`, con:
  - sus rutas;
  - `@PermissionRequired(PermissionCodes.Security.REGISTER_EXITS_AND_RETURNS)` en la clase;
  - `@Tag(name = "Employees")`;
  - las mismas descripciones (`OrganizationOpenApiTests` las lee).

  Con ellos van `TerminateEmployeeResource` (`TerminateEmployeeRequest`), `ReactivateEmployeeResource` (`ReactivateEmployeeRequest`), `EmployeeTenureResource` (`EmployeeTenureResponse`) y su ensamblador, armado desde el colaborador. Los de `iam` se borran en el mismo commit: tienen los mismos nombres de bean y las mismas rutas.

- **El puerto `EmployeeAccountConsequences`, en `organization/interfaces/acl`.** Solo usa tipos de Java y de `shared`, así que la clase de `iam` que lo implementa no nombra otro tipo de la fachada (`ForeignAclStaysInTranslationLayerTests`). Todos sus métodos corren en la transacción de quien llama (`Propagation.MANDATORY`):
  ```java
  /**
   * Lo que la salida y el regreso de un colaborador le hacen a su cuenta (§5.3, §5.4, §5.9). organization lo declara y lo
   * llama dentro de la transacción de la salida o del regreso, después de bloquear al colaborador; iam lo implementa.
   * Hay exactamente una implementación: sin ella la aplicación no arranca.
   */
  public interface EmployeeAccountConsequences {

      /** Bloquea la cuenta hasta que termine la transacción: el segundo candado, después del colaborador. */
      void lockAccount(Long accountId);

      /**
       * Solo cuando la salida cierra la cuenta: la guardia del último superadministrador (IAM-R21), con su candado
       * consultivo. No escribe nada.
       *
       * @return LAST_ACTIVE_SUPERADMIN (422), o vacío
       */
      Optional<ApplicationError> exitRefusal(Long accountId, Instant now);

      /**
       * Después de que organization registró la salida: retira los roles vigentes de la cuenta en la empresa con
       * baja_colaborador y, si closesAccount, la cierra (baja salvo una en curso, baja_cuenta, línea laboral y sesiones;
       * IAM-R40, IAM-R41), firmado por requesterId.
       *
       * @return ACCESS_SUSPENSION_CONFLICT (409) si un bloqueo concurrente ocupó el período de la baja; antes de devolverlo
       *         marca la transacción para revertir, así que la salida tampoco queda
       */
      Optional<ApplicationError> applyExit(Long accountId, Long companyId, boolean closesAccount, Long requesterId, Instant now);

      /** Después de que organization registró el regreso: cierra la baja en curso, si la hay, firmada por requesterId (IAM-R43). */
      void applyReturn(Long accountId, Long requesterId, Instant now);
  }
  ```

- **La implementación en `iam`.** Va en `iam/application/internal/commandservices`, junto a `AccountClosure`, que es de paquete. Su nombre no termina en `CommandServiceImpl`, para que `DeadlockRetry` repita solo la transacción de `organization`. Depende solo de beans de `iam` —`UserRepository`, `AccessSuspensionRepository`, `LastSuperadminGuard`, `AccountClosure` y `SystemAccessScope`—, nunca de su `ExternalOrganizationService`: así no hay ciclo de beans. Lee la cuenta con `findByIdForUpdate` en cada método, porque entrar en `SYSTEM` vacía lo leído, y guarda con la elevación de hoy cuando cierra la cuenta.

- **Un solo bean, sin valores por omisión.** `organization` inyecta `EmployeeAccountConsequences`, nunca una lista: sin implementación, la aplicación no arranca y nunca registra una salida que no cierra la cuenta. `EmployeeAccountConsequencesWiringTests` comprueba que el único bean es el de `iam`.

- **La salida, en este orden:**
  1. Quien opera es `currentAccessScope.current().userId()`. Sin cuenta, `401`.
  2. El colaborador, bloqueado (`findByPublicIdForUpdate`) bajo el alcance de quien opera. Si no está, `404 EMPLOYEE_NOT_FOUND`.
  3. `scope.actsWhere(REGISTER_EXITS_AND_RETURNS, empresa, sede)`. Si no actúa, `403 ACCESS_DENIED`, con el texto de hoy.
  4. Si el colaborador está vinculado a la cuenta de quien opera, `403 OWN_ACCOUNT_FORBIDDEN`.
  5. Si quien opera no tiene un colaborador activo, `403 ACTOR_EMPLOYEE_REQUIRED`. Se lee como hoy: bajo su alcance, sin elevación.
  6. Si el colaborador tiene cuenta, `lockAccount`.
  7. `closesAccount`: el colaborador tiene cuenta, está activo y la cuenta no tiene otro colaborador activo en otra empresa. Se lee en `SYSTEM`, y solo salen identificadores.
  8. Si `closesAccount`, `exitRefusal`: `422 LAST_ACTIVE_SUPERADMIN`.
  9. La salida con las reglas de hoy de `EmployeeCommandServiceImpl.handle(TerminateEmployeeCommand)`: `EMPLOYEE_ALREADY_TERMINATED`, `EMPLOYEE_TERMINATION_BEFORE_HIRE`, `EMPLOYEE_TERMINATION_DATE_IN_FUTURE` y `BUSINESS_RULE_VIOLATION`. Si falla, no se escribió nada.
  10. Si tiene cuenta, `applyExit`. Un `409 ACCESS_SUSPENSION_CONFLICT` revierte todo.
  11. `200` con `EmployeeTenureResource`.

- **El regreso:**
  1. Los pasos 1 a 6 de la salida.
  2. Las reglas de hoy: `EMPLOYEE_ALREADY_ACTIVE`, `EMPLOYEE_REHIRE_BEFORE_TERMINATION`, la empresa y la sede `FOR SHARE` con `COMPANY_INACTIVE` y `BRANCH_INACTIVE`, y `EMPLOYEE_REHIRE_DATE_IN_FUTURE`; y el período que cierra (D-241).
  3. Si tiene cuenta, `applyReturn`.
  4. `200`.

- **Los candados quedan en el orden de hoy:** el colaborador, la cuenta, el candado del último superadministrador y, en el regreso, la empresa y la sede en modo compartido. Por eso `lockAccount` va antes de cualquier validación de `organization`.

- **Salen:**
  - `terminateEmployee` y `reactivateEmployee`, de `OrganizationContextFacade` y su implementación;
  - de `iam`: `EmployeeTenureCommandService` con su implementación, sus comandos `TerminateEmployeeCommand` y `ReactivateEmployeeCommand`, los dos controladores, sus recursos y su ensamblador, y `terminate` y `reactivate` de su `ExternalOrganizationService`.

- **Se quedan en `iam`:** dar acceso (`linkUserToEmployee`, `linkExistingUserToEmployee`), el desvínculo (`/employees/{id}/account-unlinking`), `fetchEmployeeSummaryForUpdate`, `AccountClosure`, `AccountReachReader` (los usa el desvínculo), `LastSuperadminGuard` y `AccountAdministrationGuard`, que la salida no usa.

- **`EmployeeAllowedActions` no cambia:** ofrece la baja y la reincorporación con las mismas tres reglas de los pasos 3 a 5, y una prueba lo ata a la operación.

- **Dependencias:** ninguna nueva. `iam` ya depende de `organization::acl`, y `organization` no nombra a `iam`.

**Frontend.** Nada: las rutas, el cuerpo y los códigos no cambian (`EmployeesStore` y los diálogos de baja y de reincorporación). Solo cambian los comentarios de `employees.store.ts` que dicen «IAM».

**Las acciones de cuenta de cada fila no entran en esta fase.** `EmployeeAllowedActions` las calcula con `ProtectedAccounts` y `PendingAccountActivations`. Moverlas a `iam` cambia el contrato de `allowedActions` de la lista de colaboradores, la regla «las acciones por fila las calcula el backend» y lo construido el 15/11 (`bb02f4cf`, `11c29a46`): es una fase propia, con su decisión.

**Pruebas de caracterización, antes de mover nada (contra las rutas de `iam`, sin cambiarlas después).**
1. **El orden de las negativas**, en la salida y en el regreso:
   - fuera del alcance, `404`;
   - el permiso solo en otra empresa, en otra sede, o solo en una sede para un colaborador sin sede, `403 ACCESS_DENIED`;
   - su propio colaborador, `403 OWN_ACCOUNT_FORBIDDEN`, también para un superadministrador;
   - sin colaborador activo, `403 ACTOR_EMPLOYEE_REQUIRED`, también un superadministrador sin vincular (D-89) y quien tiene su colaborador en una empresa que sus roles no alcanzan;
   - el último superadministrador antes que una fecha inválida.
2. La salida de una cuenta protegida (con un rol reservado) la registra el administrador de su empresa y la cierra (IAM-R41): nunca responde `ACCOUNT_MANAGED_BY_SUPERADMIN`.
3. **Lo que le pasa a la cuenta:**
   - los roles de esa empresa se retiran con `baja_colaborador`, y los de otra empresa se quedan si trabaja allí;
   - con la última empresa: `baja` desde ahora (sin una segunda si ya hay una en curso), los demás roles con `baja_cuenta` (también los de alcance sistema), las sesiones cerradas, las vacaciones en curso terminadas, lo programado de la línea laboral cancelado y los bloqueos de seguridad intactos.
4. **La reversión:** el cierre choca con un bloqueo concurrente, responde `409` y el colaborador sigue activo, como en `EmployeeAccountUnlinkingRollbackTests`. La prueba queda en el paquete de `iam`, porque `AccountClosure` es de paquete.
5. **El regreso:** cierra solo la `baja`, a nombre de quien lo registra; los roles no vuelven; una empresa o una sede inactiva responde `422` y la cuenta sigue cerrada; guarda su período.
6. **Carreras, con transacciones confirmadas:**
   - dos salidas de la misma cuenta en dos empresas a la vez la cierran una vez, con una sola `baja`;
   - una salida y un acceso al mismo colaborador;
   - las salidas de los dos únicos superadministradores a la vez: una responde `422`.
7. **El orden de los candados:** con la cuenta bloqueada por otra transacción, la salida ya tiene bloqueado al colaborador mientras espera.
8. `EmployeeAllowedActions` ofrece `terminate` o `reactivate` exactamente cuando la operación pasa las tres negativas.

**Pruebas que se mueven o se corren.**
- `EmployeeTenureIntegrationTests` pasa a `organization`.
- En `iam` se quedan y se corren `SecurityOperationsBySystemRoleIntegrationTests`, `CrossCompanyAccountAdministrationIntegrationTests`, `AccessAdministratorIntegrationTests`, `UsersIntegrationTests`, `LastSuperadminGuardTests`, `SuperadministratorsLockTests` y `AccountLockTests`.
- En `organization` se corren `EmployeeCommandServiceImplTests`, `EmployeeAllowedActionsTests`, `EmployeeAllowedActionsIntegrationTests`, `EmployeeTenurePeriodsIntegrationTests`, `EmployeeLockTests` y `OrganizationOpenApiTests`.
- Las de arquitectura y `DeadlockRetryTests`.
- **Frontend:** `e2e/access.spec.ts` y el e2e de colaboradores. **e2e-live:** `organization.live.spec.ts`.

**Orden dentro de la fase.**
1. Las pruebas de caracterización, en verde en `develop`.
2. El puerto en `organization` y su implementación en `iam`, extrayendo el código de hoy, sin cambiar el comportamiento, con su prueba de cableado.
3. El servicio, los controladores y los recursos en `organization`, y el retiro de `iam`, en el mismo commit.
4. Salen los miembros de la fachada.
5. Documentación y las cuatro copias de las skills (`.claude` y `.cursor` del backend y del frontend).
6. Revisión de arquitectura (skill `architecture-review`) antes de integrar.

**Documentación.**
- Una decisión nueva del diccionario que reemplaza la decisión 2, opción A.
- El mapa vigente: §4, §5, §6, §10 y §11.
- Las convenciones: §2.2 (las rutas pasan a la fila de `organization`), §7.2 (las tablas de la salida y el regreso dicen que revisa `organization`), §9.2 y §14.17.
- `contextos/iam.md`: F9, IAM-R21, IAM-R34, IAM-R40 a IAM-R43, las elevaciones a `SYSTEM` y la tabla de la fachada.
- `contextos/organization.md`: F7, ORG-R22 a ORG-R24, ORG-R50 y el puerto nuevo.
- Las cuatro copias de las skills (`.claude` y `.cursor` del backend y del frontend): `SKILL.md` §4, dónde cuenta `registrar_salidas_y_regresos`.

**Terminado cuando:**
- `iam` no sirve las rutas ni nombra `registrar_salidas_y_regresos`;
- `OrganizationContextFacade` no tiene miembros de salida ni de regreso;
- las pruebas de caracterización pasan sin cambios;
- la revisión de arquitectura no encuentra hallazgos.

**Riesgo alto:** toca la seguridad de las cuentas. Lo cuidan el orden de las negativas, el bean único sin valores por omisión, la reversión cuando falla la consecuencia y las carreras.

### Fase 8 — Los catálogos preguntan en vez de recibir banderas

**Por qué.**
- Hoy `fleet` le escribe a `vehiclecatalog` `has_units` (`markModelsInUse`) y a `equipment` `has_installations` (`markEquipmentTypeInUse`): el proveedor guarda un hecho de su cliente, y es un valor derivado (regla 4.1).
- `location` resuelve el mismo problema preguntando por un puerto (`CurrencyUsage`).
- La pregunta equivale a la bandera porque `vehicles.model_id` y `vehicle_equipments.equipment_type_id` no cambian y ninguna fila se borra. Cuentan las unidades anuladas y las instalaciones cerradas.

**Contextos:** `vehiclecatalog`, `equipment` y `fleet`, que implementa los dos puertos.

#### 3.1 Los puertos

- **`vehiclecatalog/interfaces/acl/VehicleModelUsage`:** `Set<Long> modelIdsWithUnits(Set<Long> modelIds)`. Lo implementa `fleet/infrastructure/vehiclecatalog/VehicleModelUsageAdapter`, con un método nuevo de `VehicleRepository`: `SELECT DISTINCT model_id FROM vehicles WHERE model_id IN (:ids)`.
- **`equipment/interfaces/acl/EquipmentTypeUsage`:** `Set<Long> typeIdsWithInstallations(Set<Long> typeIds)`. Lo implementa `fleet/infrastructure/equipment/VehicleEquipmentTypeUsageAdapter`, con `VehicleEquipmentRepository`: `SELECT DISTINCT equipment_type_id FROM vehicle_equipments WHERE equipment_type_id IN (:ids)`.
- **Los dos son por lote**, porque los lee cada fila de una lista. Quien pregunta inyecta `List<…>`, como `CurrencyUsage`.

#### 3.2 Lo que cambia, completo

**Lecturas:**
- `VehicleModel.isVehicleTypeLocked()` y `canChangeVehicleTypeTo(id)` pasan a recibir `hasUnits`, y `EquipmentType.isCodeLocked()` y `canCorrectCodeTo(code)` a recibir `hasInstallations`. Es el patrón de `VehicleDocumentType.isCodeLocked(usedByADocument)` de `fleet`.
- `VehicleModelView` y su fábrica calculan `vehicleTypeLocked` por lote; `VehicleModelResourceFromEntityAssembler:22` lo lee de la vista.
- `EquipmentTypeViewFactory` calcula `codeLocked` como «`PROGRAM_CODES` o instalado», por lote.

**Escrituras que se retiran:**
- en `vehiclecatalog`: `markModelsInUse` de la fachada, `MarkVehicleModelsInUseCommand`, `VehicleModelCommandService(Impl).handle(Mark…)`, `VehicleModelRepository.markAllInUse` y la consulta nativa de `VehicleModelJpaRepository.java:57`;
- en `equipment`: `markEquipmentTypeInUse`, `MarkEquipmentTypeInUseCommand`, `EquipmentTypeRepository.markInstalled` y `EquipmentTypeJpaRepository.java:60`;
- en `fleet`: `ExternalVehicleCatalogService.markModelsInUse`, `VehicleRegistrations.markModelsInUse` (l.127), `VehicleCommandServiceImpl:153`, `PurchasedUnitRegistrations:160`, `MigratedUnitRegistrations:202-204`, `VehicleEquipmentInstallations:174` y `ExternalEquipmentService.markEquipmentTypeInUse`.

**El mapeo que se retira:**
- `hasUnits` de `VehicleModel`, `VehicleModelPersistenceEntity` y `VehicleModelPersistenceAssembler:28`;
- `hasInstallations` de `EquipmentType`, `EquipmentTypePersistenceEntity` y `EquipmentTypePersistenceAssembler:40`.

**Lo que no cambia:** la API (`vehicleTypeLocked`, `codeLocked`, `VEHICLEMODEL_TYPE_LOCKED` y `EQUIPMENTTYPE_CODE_LOCKED` son los mismos) y el frontend.

#### 3.3 El alcance

Se pregunta siempre con `systemAccessScope.call(...)`, como `CurrencyCommandServiceImpl.java:102`: `vehicles` y `vehicle_equipments` tienen RLS por `company_id`, y la respuesta debe contar todas las empresas. Solo sale un dato del catálogo, nunca las filas.

#### 3.4 Los candados

La bandera que hoy escribe el alta es lo que serializa la corrección con la primera unidad o la primera instalación. Sin ella:
- la corrección del tipo de vehículo de un modelo, y la del código de un tipo de equipo, toman la fila **`FOR UPDATE`**, y no `FOR NO KEY UPDATE`, **antes** de preguntar al puerto;
- el alta de una unidad o de una instalación pasa por la FK (`vehicles.model_id` y `vehicle_equipments.equipment_type_id`), que toma `FOR KEY SHARE` sobre esa fila y choca con `FOR UPDATE`;
- así el alta confirma antes y el puerto la ve, o espera a la corrección;
- después de preguntar se relee el registro, porque entrar al alcance `SYSTEM` limpió lo leído (`CurrencyCommandServiceImpl.java:99-103`).

La alternativa de `CurrencyUsage` es que los cuatro caminos que escriben (`VehicleCommandServiceImpl`, `PurchasedUnitRegistrations`, `MigratedUnitRegistrations` y `VehicleEquipmentInstallations`) lean el modelo o el tipo `ForShare`. El orden no cambia: primero la unidad y después el catálogo; el catálogo nunca bloquea una unidad.

#### 3.5 Índice

Una migración nueva crea `vehicle_equipments (equipment_type_id)`, para la consulta por lote.

#### 3.6 Pruebas

- **Se reescriben**, para crear una unidad o una instalación en vez de escribir la bandera: `VehicleModelsControllerTests:462,477`, `EquipmentTypesControllerTests:296,309`, `VehicleEquipmentIntegrationTests:494,509`, `FleetContextFacadeImplTests:121` y `VehicleRetiredReferencesTests:83`.
- **Salen:** `VehicleCatalogContextFacadeTests` (l.146), `EquipmentContextFacadeTests` (el caso de la línea 257 de hoy) y `EquipmentCommandServiceImplTests:116`.
- **Se ajusta** `EquipmentTypeLockTests`.
- **Nuevas:**
  - los dos adaptadores con unidades de dos empresas, leídos con el alcance de una sola, como `VehicleEquipmentCurrencyUsageTests`;
  - dos carreras: corregir contra la primera unidad, y corregir contra la primera instalación;
  - la prueba en vivo de D-185 (`e2e-live/vehiclecatalog.live.spec.ts:582`) sigue pasando sin cambios.

#### 3.7 En dos entregas

1. **Antes de integrar,** en `dev`: `SELECT id FROM vehicle_models m WHERE has_units <> EXISTS (SELECT 1 FROM vehicles v WHERE v.model_id = m.id)` y su par para `equipment_types`. Toda diferencia es un cambio de comportamiento y se explica antes.
2. **Entrega N:** §3.1 a §3.6. Las columnas siguen (`NOT NULL DEFAULT FALSE`): las altas no fallan y `ddl-auto: validate` no exige mapearlas.
3. **Entrega N+1,** cuando N ya corre en las dos instancias y su CI está en verde: una migración nueva hace `DROP COLUMN` de `vehicle_models.has_units` y de `equipment_types.has_installations`, con `lock_timeout`. Se ajusta `VehicleCatalogMigrationTests:78`; se actualizan el diccionario (§7.x, la fila de `has_units` y la de `has_installations`) y D-107 y D-185 con la decisión nueva.
4. **Vuelta atrás:** revertir la entrega N antes del drop exige recalcular las banderas (`UPDATE … SET has_units = EXISTS (…)`, y lo mismo para los tipos), porque dejaron de escribirse.

**Sobre `CurrencyUsage`:** es el patrón análogo (el contexto de abajo declara el puerto y el de arriba lo implementa; se pregunta bajo el candado de la fila y con `SYSTEM`), con tres diferencias:
- aquí también se lee en cada lista, así que el puerto es por lote;
- la serialización puede venir de la FK con `FOR UPDATE`, en vez de lecturas `ForShare` en los escritores;
- hay un solo implementador.

### Fase 9 — Renombrar `location` a `reference` (opcional)

**Por qué.** El nombre es parte del lenguaje (cap. 2). `location` no describe monedas, tasas ni tipos de documento, y choca con `Location` de `organization`.

**El tamaño real.** En el backend, 275 archivos del paquete y 175 que lo importan; en el frontend, 72 archivos que importan su carpeta y 86 que usan las rutas `/location/…`.

**Qué cambia.**
- **Backend:** el paquete y el `displayName`; `"location::acl"` en los `package-info.java` de `fleet`, `organization`, `procurement` y `legacyimport` (si todavía existe); la fachada (`LocationContextFacade`, que puede conservar su nombre) y las pruebas.
- **Frontend:** la carpeta `location`, las rutas `/location/…` con sus redirecciones, el menú, `e2e/screens.ts` y las specs.
- **Documentación y las cuatro copias de las skills.**

**Qué no cambia:** las rutas de la API, las tablas, los códigos de los permisos y los nombres de las imágenes de referencia.

Se hace al final, o no se hace: es el cambio con más archivos y menos efecto sobre el modelo.

### Fase 10 — Preparar el bloque 12

Sin código. Se resuelven con el negocio las preguntas del mapa objetivo (§7, punto 8) y se registran como decisiones en el diccionario. Con eso se diseña:
- **el servicio publicado de `fleet` para las transiciones comerciales** —reservar, firmar, entregar, resolver y completar—, que llamará `leasing`;
- **los puertos `UnitContracts`** (que reemplaza la deducción de D-250) **y `UnitTitleTransfers`** (que reemplaza a `VEHICLE_TRANSFER_REQUIRED` cuando exista el trámite);
- **la identidad del cliente,** separada de `iam`;
- **el lugar de la regla de las 48 horas** desde la firma;
- **las cuentas bancarias en la cobranza:** `collections` implementa `BankAccountUsage` (fase 6) para sus cobros, y todo cobro que nombre una cuenta la lee antes `FOR SHARE` por la fachada de `organization`, como el pago.

Después, cada contexto nuevo se construye con el orden de la skill `backend-module`: `customers` (mínimo), `leasing`, `collections`, `titletransfer` y el portal.

---

## 5. Lista de control de cada fase

**Backend**
- [ ] El módulo tiene su `package-info.java` con `allowedDependencies`, y su `interfaces/acl` con `@NamedInterface("acl")` solo si otro módulo depende de él. `ModularityTests` pasa y no hay ciclos.
- [ ] La fachada expone todo lo que usan sus consumidores. Spring Modulith publica de un paquete anotado solo sus propias clases, sin subpaquetes ni los tipos de sus firmas: un tipo que vive en otro paquete y debe ser público lleva `@NamedInterface("acl")` a nivel de tipo, y el dominio nunca importa `interfaces`.
- [ ] Ninguna clase de otro contexto importa su interior: solo su `::acl`, desde su `External<Contexto>Service` (`ForeignAclStaysInTranslationLayerTests`). `audit::acl`, `imports::acl`, `documents::acl` y `notifications::acl` son contratos de plataforma y se usan sin traductor; cada uno está en `PLATFORM_CONTRACTS` de esa prueba.
- [ ] Cada `External<Contexto>Service` lleva nombre de bean explícito, `@Service("<contexto>External<X>Service")`: hoy ya hay varios con el mismo nombre simple, y uno sin nombre impide arrancar. `ErpMillenniumMobilityBackendApplicationTests` arranca el contexto.
- [ ] Ninguna ruta queda mapeada dos veces: el controlador viejo sale en el mismo commit en que entra el nuevo, y un controlador nuevo no repite el nombre simple de otro.
- [ ] Los controladores movidos conservan su ruta, su `@PermissionRequired`, su `@Tag`, sus descripciones OpenAPI, los nombres de sus `@Schema` y sus códigos de error, con el mismo orden de revisiones.
- [ ] Un puerto que protege una invariante con un único implementador (`UnitFundAssignments`, `EmployeeAccountConsequences`) se inyecta como bean obligatorio, nunca como una lista con valores por omisión, y tiene su prueba de cableado. Uno que se responde con una lista de beans (`CurrencyUsage`, `BankAccountUsage`) tiene su prueba de arranque que exige a cada implementador.
- [ ] Los servicios de comandos conservan el patrón `*CommandServiceImpl` en `application.internal.commandservices` (`DeadlockRetry`), y todo `@Scheduled` sigue dentro de `SingleInstanceRun`.
- [ ] Ningún `Jdbc<Contexto>DataVersions` de un contexto de negocio nombra una tabla de otro contexto (`reporting` es la capa de lectura); cada exportación es un `DocumentKind` del módulo que la sirve, y su versión compone la de cada contexto cuyos datos muestra.
- [ ] Un servicio de consultas que lee tablas de empresa corre en una transacción `readOnly`: fuera de una transacción, la cuenta de la aplicación no ve ninguna fila.
- [ ] Las pruebas se mueven con su código, espejando los paquetes; las de caracterización de la fase pasan sin cambios.

**Frontend**
- [ ] La carpeta del contexto tiene sus cuatro capas, y la fachada, solo los miembros que alguien usa (no se crea una fachada sin consumidor).
- [ ] Las rutas declaran los mismos permisos; el menú conserva grupo, acceso, título y descripción; las URL viejas redirigen, con un e2e que prueba cada redirección.
- [ ] Se actualizan `e2e/screens.ts`, las specs que construyen las rutas (como `app-shell.spec.ts`) y las de e2e-live que usan las rutas viejas; las imágenes de referencia no se regeneran.
- [ ] Las claves de `es.json` no se renombran en la fase que mueve; un renombre es su propio cambio.

**Documentación**
- [ ] Lo movido pasa del mapa objetivo al vigente, a `contextos/` y al diccionario §3, en el mismo cambio; una decisión que se reemplaza (D-203, la decisión 2 del mapa vigente) queda reemplazada por una nueva.
- [ ] Las convenciones, si cambió un recurso, una ruta de módulo o la tabla de rutas por contexto.
- [ ] Las cuatro copias de las skills y el README del backend.

---

## 6. Vuelta atrás

Las fases 1 a 7 y la 9 no cambian datos ni esquema: deshacer una es revertir su commit de integración en los dos repositorios. La fase 8 va en dos entregas: la primera deja de escribir dos columnas, y la migración que las elimina se aplica solo cuando esa entrega corre en las dos instancias y su CI está en verde. Revertir la primera entrega antes de eliminar las columnas exige recalcularlas, porque dejaron de escribirse (fase 8 §3.7).

---

## 7. Calendario orientativo

| Semana | Fases |
|---|---|
| 1 | 0, 1 y 6 |
| 2–4 | 2 |
| 5 | 3 |
| 6–7 | 4 |
| 8–9 | 5 |
| 10–11 | 7 |
| 12 | 8, primera entrega (la segunda, una entrega después) |
| Cuando se decida | 9 |
| En paralelo, con el negocio | 10 |

---

## 8. Riesgos

| Riesgo | Mitigación |
|---|---|
| Un acoplamiento oculto que solo aparece al mover | `ModularityTests` lo detecta al compilar el módulo; las pruebas de quien consume se corren en cada fase |
| Una fachada que no expone un tipo que usa su consumidor | La lista de control de §5 y `ModularityTests`; cada fase enumera los tipos de su fachada |
| Dos beans o dos rutas con el mismo nombre impiden arrancar | Nombres de bean explícitos; el controlador viejo sale en el mismo commit; `ErpMillenniumMobilityBackendApplicationTests` |
| Un puerto se queda sin quien lo implemente y su regla deja de cumplirse sin que nada falle (D-182 con `UnitFundAssignments`, ORG-R65 con `BankAccountUsage`, la salida del colaborador) | Bean obligatorio con prueba de cableado, o lista con prueba de arranque (§5) |
| `reporting` lee tablas por SQL | Las tablas no cambian de nombre; sus pruebas se corren en las fases 2, 4 y 5 |
| La versión de los datos de una exportación deja de cubrir una tabla y el documento se reutiliza viejo | Cada fase que parte un `DataVersions` compone las versiones de los dos contextos y prueba la exportación después de cambiar un dato de cada uno |
| La fachada de un contexto crece sin control | Un miembro nuevo solo con un consumidor que lo use (skill `backend-module`, paso 12) |
| Conflictos con trabajo funcional en paralelo | Una fase a la vez, ramas cortas, sin trabajo funcional en el área que se mueve; las fases 4 y 5, en ese orden |
| Las skills y el mapa quedan atrás del código | Se sincronizan en la fase 0 y se actualizan en cada fase, en las cuatro copias |
| `dev` se despliega antes del CI | El CI corre en la rama antes de integrar a `develop` |
| Las fases que cambian el recurso de la unidad (la 4 o la 5, solo con su opción B) | Se integran backend y frontend juntos, con las convenciones §4.2 acordadas en orden |
| La fase 7 debilita una regla de seguridad | Pruebas de caracterización antes de mover, bean único, prueba de reversión, carreras y revisión de arquitectura antes de integrar |

---

## 9. Lo que corrigió la auditoría

El 18/11/2026 cada fase se verificó contra el código: las clases, los consumidores, las fachadas, los candados, las pruebas y el frontend. La versión 1 del plan no se habría podido ejecutar tal como estaba: tenía **17 errores bloqueantes** —que rompían la compilación, `ModularityTests`, el arranque o una regla de seguridad— y más de 30 de severidad alta. Esta versión los incorpora. Lo principal:

| Fase | Lo que estaba mal | Cómo queda |
|---|---|---|
| 0 y todas | Las skills eran «tres copias»; son cuatro y ya divergían. `ForeignAclStaysInTranslationLayerTests` tiene los contratos de plataforma escritos a mano | Se sincronizan en la fase 0; la fase 1 suma `audit` e `imports` a la prueba |
| 1 | `imports` depende de `audit`, y las dos fachadas no exponían lo que usan sus consumidores (Modulith no expone subpaquetes ni los tipos de las firmas) | `audit` primero; `imports` con `{"audit::acl", "documents::acl"}`; las fachadas, con la lista completa de tipos |
| 2 | El inventario lee empresas y ubicaciones de `organization` y la placa de la unidad de `fleet`; el catálogo de equipos usa el interior de `vehiclecatalog`; la versión de la habilitación mezcla tablas de los dos catálogos; un bean sin nombre impediría arrancar | `equipment` depende de `organization`; la placa llega por un puerto que implementa `fleet`; `vehiclecatalog::acl` suma los tipos de vehículo; las versiones se parten y se componen; nombres de bean explícitos |
| 3 | Los writers de compras usaban las filas del libro que la fase sacaba de `procurement`, y `procurement::acl` necesitaba más de tres miembros | Una capa anticorrupción de verdad: `procurement::acl` habla su lenguaje, `legacyimport` traduce, y el registro de la factura sigue llamando a `fleet` para no invertir el orden de los candados |
| 4 | Faltaban dependencias de `financing` (`iam::acl`, `audit::acl`); `fleet` lee por SQL la tabla que se muda; el recurso de la unidad cambiaba sin decirlo | Dependencias completas; miembros nuevos en la fachada de `fleet`; la opción A no cambia el recurso |
| 5 | La brecha de días se perdía en la ficha o se duplicaba; `procurement` no tenía cómo resolver la unidad | La brecha sigue en `fleet` (opción A); `fleet::acl` suma la unidad que la sesión ve |
| 6 | Mover las cuentas bancarias a cuentas por pagar contradecía el propio mapa: la cobranza también las usará | Se quedan en `organization`, con un puerto por consumidor (`BankAccountUsage`), el patrón de `CurrencyUsage` |
| 7 | Las negativas estaban mal: con el plan, un administrador de empresa no podría registrar salidas y quien sale con un rol reservado conservaría el acceso | Las tres negativas son de `organization`; el puerto de `iam` solo bloquea la cuenta, aplica la guardia del último superadministrador y las consecuencias |
| 8 | La escritura de las banderas es lo que hoy serializa una corrección del catálogo con la primera unidad; un puerto que solo lee reabría la carrera | La corrección toma la fila `FOR UPDATE` antes de preguntar, el puerto lee con el alcance de sistema y va por lote |
| 9 | Subestimada | 6–10 días, con su tamaño real |

**Precisiones sobre el libro.**
- *Customer/Supplier* es, en el libro, una relación entre equipos con requisitos negociados y pruebas automáticas de aceptación en la frontera (cap. 14). Con un solo equipo se usa para decir quién define y quién traduce, y se sostiene con una prueba de contrato por fachada o puerto (§2).
- El libro no tiene un patrón *Partnership* (aparece en escritos posteriores de Evans). La dependencia en los dos sentidos entre `fleet` y `financing` se escribe como dos relaciones: `financing` aguas abajo de `fleet` (las unidades y su candado) y `fleet` que pregunta por un puerto (la cesión abierta).
- La consistencia sincrónica entre agregados de dos contextos —la cesión y la anulación (D-182), la salida y la cuenta, el dispositivo y su instalación— es una excepción deliberada al consejo del cap. 6, que suele preferir la consistencia eventual entre agregados. La justifica una invariante que no admite una ventana (una unidad cedida no se anula, quien sale no conserva el acceso, un dispositivo está en una sola unidad), y la permite una sola base de datos con un orden fijo de candados.
- En la fase 7, lo que se mueve es el caso de uso —el servicio de aplicación (cap. 4)—, que pertenece al contexto dueño del término (cap. 14); el cambio ya entraba por la raíz `Employee`.
- `reporting` se conforma con lo que publican los demás, pero además lee sus tablas por SQL, una integración por base compartida: su contrato lo sostienen sus pruebas, y más adelante puede pasar a vistas publicadas por cada contexto.
