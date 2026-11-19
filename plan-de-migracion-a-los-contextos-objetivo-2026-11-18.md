# Plan de migración a los contextos objetivo — ERP Millennium Mobility

**Estado:** propuesta para validar por el responsable (18/11/2026), junto con el [mapa de contextos objetivo](mapa-de-contextos-objetivo-millennium-mobility.md).

**Fuente:** el inventario del código de `develop` (backend `ed5adc12` y frontend) hecho el 18/11/2026, las reglas de arquitectura de las skills `backend-module`, `frontend-feature` y `architecture-review`, y *Domain-Driven Design* de Eric Evans (las referencias «cap. n» son capítulos del libro).

**Objetivo.** Llevar lo que hoy está construido a los contextos del mapa objetivo, por fases pequeñas que dejan el sistema funcionando al final de cada una, sin cambiar lo que hace el sistema salvo donde una fase lo dice. El bloque 12 (clientes, contratos, cobranza y traspasos) no se migra: se construye después, sobre la estructura que deja este plan, y aquí solo se prepara (fase 10).

---

## 1. Lo que no cambia en ninguna fase

Estas garantías hacen que cada fase sea un cambio de estructura y no de comportamiento:

| Qué | Por qué se mantiene |
|---|---|
| **Las tablas, sus nombres y sus migraciones** | Un contexto es dueño de sus tablas por el código que las escribe, no por su nombre. Ninguna fase crea, renombra ni borra una tabla, salvo la 8, que retira dos columnas después de dejar de usarlas. `fleet_equipments` conserva su nombre aunque pase a `equipment` |
| **Row Level Security y `data_versions`** | Las políticas van por tabla y siguen iguales; solo cambia qué `Jdbc<Contexto>DataVersions` nombra cada tabla |
| **Las rutas de la API** | Las rutas no llevan el contexto (convenciones §2.1): `/equipments`, `/funds` o `/bank-accounts` siguen en su lugar aunque las sirva otro módulo. Las únicas que cambian de forma son las de la fase 5, y se acuerdan en las convenciones antes de construirlas |
| **Los códigos de los permisos, de los errores y las claves de los mensajes** | Son contratos por nombre con el catálogo, el frontend y los bundles. Renombrarlos es una decisión aparte (mapa objetivo, §7) |
| **El comportamiento** | Cada fase es una refactorización: las mismas pruebas pasan antes y después. Las excepciones —el recurso de la unidad en la fase 5 y la pantalla de las cuentas bancarias en la 6— lo dicen en su fase |

---

## 2. Cómo se trabaja

- **Pasos pequeños que dejan el sistema sano** (cap. 13, *Refactoring Toward Deeper Insight*). Una fase se integra completa o no se integra; nunca queda un contexto a medio mover en `develop`.
- **Una fase, una rama por repositorio:** `refactor/<fase>` en el backend y su pareja en el frontend si la fase lo toca, integradas juntas.
- **El CI corre en la rama antes de integrar.** Cada push a `develop` despliega `dev` sin esperar al CI, así que la suite completa se corre en la rama (*Run workflow*) y se integra solo en verde.
- **El mapa se actualiza en la misma fase** (cap. 14, *Context Map*). Lo que la fase mueve pasa del mapa objetivo al mapa vigente, a `contextos/` y al diccionario §3 en el mismo cambio. Un mapa desactualizado es peor que no tener mapa.
- **Sin trabajo funcional en el área que se mueve mientras dura su fase.** La auditoría del 09/11 ya mostró los conflictos de ramas paralelas.
- **Las pruebas, a la medida del cambio:**
  - en local, en una base nueva de `erp_mm_owner` y nunca con el `.env` remoto: las pruebas del contexto que se mueve, las de quien lo consume y las de arquitectura (`ModularityTests`, `LayerDependencyTests`, `ForeignAclStaysInTranslationLayerTests`, `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`, `DeadlockRetryTests`, `JdbcDataVersionsTests`, `NoDeleteEndpointsTests`);
  - en el frontend, el build, las specs de lo movido y `lint:styles`;
  - todo lo demás, en el CI.

---

## 3. Las fases

| Fase | Qué | Contexto que forma o ajusta | Costo | Riesgo | Depende de |
|---|---|---|---|---|---|
| 0 | Preparación | — | 1–2 días | Bajo | — |
| 1 | La bitácora y las cargas salen de `shared` | `audit`, `imports` | 2–3 días | Bajo | 0 |
| 2 | Equipamiento | `equipment` | 7–10 días | Medio | 0 |
| 3 | La migración inicial como capa anticorrupción | `legacyimport` | 3–4 días | Bajo | 1 |
| 4 | Financiamiento | `financing` | 7–9 días | Medio-bajo | 1 |
| 5 | El servicio publicado de compras | `procurement`, `fleet`, `reporting` | 7–11 días | Medio | 3 |
| 6 | Las cuentas bancarias pasan a cuentas por pagar | `procurement`, `organization` | 4–6 días | Medio-bajo | 0 |
| 7 | La salida y el regreso del colaborador, en `organization` | `organization`, `iam` | 5–8 días | **Alto** | 0 |
| 8 | Los catálogos preguntan en vez de recibir banderas | `vehiclecatalog`, `equipment` | 2–3 días | Bajo-medio | 2 |
| 9 | Renombrar `location` a `reference` (opcional) | `reference` | 3–5 días | Bajo | 0 |
| 10 | Preparar el bloque 12 | Diseño, sin código | 2–3 días | — | Respuestas del negocio |

Son entre 43 y 64 días de trabajo de una persona, sin calibrar. El orden va de lo barato y seguro a lo delicado, con una excepción deliberada: **equipamiento va segundo** porque el próximo módulo anunciado al cliente es el monitoreo del GPS, y separarlo antes de construirlo cuesta mucho menos que después (cap. 14).

---

## 4. Fase por fase

### Fase 0 — Preparación

- **Aprobar** el mapa objetivo y este plan, y resolver las decisiones 2 a 5 del mapa objetivo (§7): códigos de permiso, pantalla de las cuentas bancarias, renombre de `location` y requisitos de la habilitación.
- **Línea base:** la suite completa en verde con las dos cuentas de la base, el backend y el frontend.
- **Quitar de `procurement` la dependencia `iam::acl`,** que declara y no usa.
- **Las skills son tres copias** —`.claude` del backend, `.claude` del frontend y `.cursor` del backend— y nombran los contextos y sus `allowedDependencies`: cada fase las actualiza en las tres.

### Fase 1 — `audit` e `imports` salen de `shared`

**Por qué.** Un *Shared Kernel* debe ser pequeño y estable (cap. 14). La bitácora y las cargas tienen tabla, endpoint y permiso propios: son *Generic Subdomains* (cap. 15), como `documents` y `notifications`, que ya se separaron (D-125, D-143).

**Backend — `audit`** (las 26 clases de `shared/changelog`):
- `package-info.java` con `@ApplicationModule(displayName = "Audit", allowedDependencies = {})`: solo usa `shared`, que es abierto.
- `audit/interfaces/acl`, con `@NamedInterface("acl")`, es su contrato público: `ChangeLogCommandService`, `ChangeLogHistoryLookup`, los tres puertos (`AuditActorLookupPort`, `AuditCompanyLookupPort`, `ConfidentialChangeLogFieldsPort`) y sus records. Los puertos salen de `domain/ports` hacia la fachada, porque otros contextos los implementan.
- `ChangeLogsController` se mueve con la misma ruta (`GET /api/v1/change-logs`) y el mismo permiso (`bitacora.ver`).
- **Consumidores que cambian sus importaciones:**
  - `procurement` (15 archivos), `fleet` (6) y `location` (2: la corrección de las tasas de cambio);
  - `iam` (1: el adaptador de `AuditActorLookupPort`) y `organization` (1: el de `AuditCompanyLookupPort`);
  - dos clases de `shared` que nombran la bitácora, que se revisan una por una.

**Backend — `imports`** (las 43 clases de `shared/importbatches`):
- `package-info.java` con `allowedDependencies = {"documents::acl"}`.
- `imports/interfaces/acl` expone `ImportBatchCommandService`, `ImportLocks`, `ImportWorkbookReader`, `ImportReportWriter`, `ImportSheetSpec`, `ImportCell`, `ImportType` e `ImportCounts`.
- **`ImportBatchFileStoreAdapter` sale de `documents`** (`documents/application/internal/storage`) y pasa a la infraestructura de `imports`, sobre `KeptFileStore` de `documents::acl`. Si se quedara, `documents` tendría que depender de `imports` y dejaría de ser independiente.
- `ImportBatchesController` se mueve con sus rutas (`/api/v1/imports`) y su permiso (`habilitacion.ver_cargas`).
- **Consumidores:** `fleet` (5 archivos) y `procurement` (5).

**`allowedDependencies` después de la fase:**

| Módulo | Agrega |
|---|---|
| `fleet` | `audit::acl`, `imports::acl` |
| `procurement` | `audit::acl`, `imports::acl` |
| `location` | `audit::acl` |
| `iam`, `organization` | `audit::acl` (implementan sus puertos) |
| `documents` | Queda en `{}` |

**Pruebas.**
- Se mueven las 8 de la bitácora y las 5 de las cargas.
- `ForeignAclStaysInTranslationLayerTests` suma `audit::acl` e `imports::acl` a sus excepciones de plataforma, como `documents::acl`.
- `DeadlockRetryTests` comprueba que los servicios de comandos movidos siguen envueltos: conservan el patrón `*CommandServiceImpl` de `application.internal.commandservices`.

**Frontend.** Nada: `change-log` e `imports` ya son carpetas propias.

**Documentación.**
- El mapa vigente, con `audit` e `imports` como módulos.
- El diccionario: §3 y §11.8, donde las reglas SHR-R01 a SHR-R13 conservan su número aunque ahora sean de `imports`.
- `contextos/README.md` y las skills.

**Terminado cuando:** `shared` no tiene `changelog` ni `importbatches`, `documents` sigue sin dependencias, y la bitácora y las cargas responden igual.

### Fase 2 — `equipment`

**Por qué.** Los dispositivos tienen lenguaje y actores propios (serie, IMEI, ICCID, taller, monitoreo) y van a crecer con el monitoreo. `Equipment#installOnVehicle` no tiene quién lo llame: el inventario todavía no está unido a la instalación, así que es el momento más barato para separarlo (cap. 14).

**Backend.**
- **`package-info.java`** con `allowedDependencies = {"vehiclecatalog::acl", "documents::acl"}`: lee los tipos de vehículo a los que aplica un equipo.
- **Desde `fleet`, el inventario** (26 clases):
  - `Equipment`, `EquipmentStatus`, `EquipmentRepository` con su persistencia sobre `fleet_equipments`, `EquipmentCommandService`, `EquipmentQueryService`, `EquipmentViewFactory` y `EquipmentsController` (`/equipments`, permisos `stock.*_equipamiento`), con sus comandos, recursos y ensambladores.
  - `EquipmentPeriod`, `EquipmentCost` y `EquipmentRemovalReason` **se quedan en `fleet`**: los usa la instalación en la unidad (`VehicleEquipment` y sus servicios), no el inventario.
- **Desde `vehiclecatalog`, el catálogo de equipos** (59 clases):
  - `EquipmentType` con su aplicabilidad (`equipment_types`, `equipment_type_vehicle_types`) y `EquipmentModel` (`equipment_models`), con sus servicios, sus controladores (`/equipment-types`, `/equipment-models`, permisos `catalogos.*_equipamiento`) y `MarkEquipmentTypeInUseCommand`;
  - el tipo de documento `equipment-types` de `VehicleCatalogDocumentKinds`;
  - las tablas de equipos de su `Jdbc…DataVersions`, que pasan a `JdbcEquipmentDataVersions`.
- **La fachada nueva `EquipmentContextFacade`** recibe:
  - los miembros de equipamiento de `VehicleCatalogContextFacade`: `fetchEquipmentType*`, `fetchEquipmentModel*` y `markEquipmentTypeInUse`;
  - un miembro nuevo para los tipos de equipo que exige y que admite cada tipo de vehículo.
- **El perfil de habilitación de un modelo se parte.** `fetchEnablementProfilesByModelIds` de `vehiclecatalog` deja solo lo del modelo (gas, pantalla de fábrica, tipo de vehículo). `fleet` lo compone con lo que le da `equipment` en `ModelEnablementProfile`.
- **El puerto `EquipmentSuppliers`** pasa de `fleet/interfaces/acl` a `equipment/interfaces/acl`. `SupplierEquipmentSuppliersAdapter` de `procurement` lo implementa desde ahí: `procurement` agrega `equipment::acl`.
- **En `fleet` cambian, para leer de `equipment` por un `ExternalEquipmentService` nuevo:** `ExternalVehicleCatalogService`, `ModelEnablementProfile`, `RequirementHolds`, `VehicleEnablementList`, `VehicleEnablementViewFactory`, `VehicleEquipmentCommandServiceImpl`, `VehicleEquipmentInstallations`, `VehicleEquipmentViewFactory` y `MigratedUnitRegistrations`. `fleet` agrega `equipment::acl`.
- **Nombres que chocan.** `EquipmentModelReference` y `EquipmentTypeReference` existen en `fleet` (su traducción) y en `vehiclecatalog` (los records de la fachada). En `equipment` quedan como records de su fachada, y la traducción de `fleet` conserva sus nombres en su paquete. Antes de integrar se revisa que no queden dos clases con el mismo nombre simple anotadas como bean de Spring.

**Frontend.**
- **Carpeta nueva `src/app/equipment`,** con las cuatro capas. Recibe los 36 archivos de equipamiento de `fleet` (el inventario, `EquipmentInventoryStore` y su formulario, que pide los proveedores a `ProcurementFacade`) y los 25 de `vehiclecatalog` (`EquipmentCatalogStore` y Gestionar equipamiento).
- **`EquipmentFacade`,** solo con los miembros que usan los diálogos de habilitación de `fleet`: las opciones de tipos y de modelos. `VehicleCatalogFacade` pierde los suyos.
- **Las rutas pasan a `/equipment/…`,** con redirecciones desde `/fleet/equipments` y `/vehicle-catalog/equipment-types`. Las opciones del menú no cambian de grupo ni de permiso.
- **Pruebas:** se actualizan `e2e/screens.ts` y las specs movidas, y las imágenes de referencia se regeneran en el CI.

**Documentación.**
- `contextos/equipment.md`, con lo de equipamiento de `fleet.md` y de `vehiclecatalog.md`. Las reglas conservan su id (`FLT-R…`, `VCT-R…`): un id no se reutiliza ni se renumera.
- El mapa vigente y el diccionario §3.

**Pruebas del backend.** Se mueven las 5 del inventario y las 12 del catálogo de equipos. Las de la habilitación (`EnablementFixtures`, `EnablementConcurrencyTests`) se corren completas, porque la puerta lee los requisitos de equipamiento.

**Terminado cuando:** la habilitación de una unidad da el mismo estado que antes para las mismas unidades, y `vehiclecatalog` ya no sabe de equipos.

### Fase 3 — `legacyimport`

**Por qué.** El libro de stock de 52 columnas es un modelo heredado. El libro pide traducirlo en una *Anticorruption Layer* para que no contamine el modelo nuevo (cap. 14). Hoy esa traducción vive dentro de `procurement` y de `fleet`.

**Backend.**
- **`package-info.java`** con `allowedDependencies = {"fleet::acl", "procurement::acl", "organization::acl", "location::acl", "vehiclecatalog::acl", "imports::acl", "documents::acl"}`.
- **Desde `procurement` se mueve la traducción:**
  - `InitialMigrationImportsController`, con sus rutas (`/initial-migration-imports` y `/preview`) y su permiso (`stock.cargar_migracion_inicial`);
  - `InitialMigrationImportCommandService` con su implementación, `InitialMigrationRun`, `InitialMigrationEffects`, `InitialMigrationCatalogs`, `InitialMigrationPreview` con su recurso, `ImportInitialMigrationCommand` con su recurso, `StockRow`, `StockRowGroups`, `StockRowMapper` y los errores `INITIALMIGRATION_*`.
- **En `procurement` se quedan las reglas de compras de lo migrado.** `MigratedInvoiceWriter`, `MigratedCreditNoteWriter` y `MigratedPaymentWriter` son reglas de compras: la factura sin los 20 días, la nota de campaña sin mes y el pago de dealer con su inicial (D-177). Se exponen por la primera fachada de `procurement`, `procurement::acl`, con tres miembros (registrar una factura, una nota y un pago migrados) que devuelven `Result`.
- **En `fleet` se quedan las reglas de flota de lo migrado:** `MigratedUnitRegistrations`, `MigrationTargetPolicy` y los miembros de su fachada (`matchMigratedUnits`, `resolveMigrationLocation`, `registerMigratedUnits`), porque registrar una unidad en su estado es regla de flota (D-202).
- **Las transacciones siguen como hoy.** El orquestador no es transaccional y abre una transacción por factura, por nota y por pago. Los miembros de las fachadas corren en la transacción de quien los llama. El candado de la carga sale de `imports::acl`.

**Una decisión:** si la carga inicial se ejecuta antes de esta fase, la fase se reemplaza por borrar el código después de la puesta en marcha.

**Después de la puesta en marcha, el módulo se borra:**
- sale el controlador, junto con los miembros de las fachadas de `procurement` y de `fleet` que solo usaba la carga;
- se conservan los datos, los caminos `MIGRACION_INICIAL`, `legacy_number` y las migraciones V117 a V119.

**Frontend.** Opcional: el diálogo de la carga (4 archivos en `procurement`) puede quedarse hasta el borrado; la pantalla Cargas lo abre por la fachada.

**Pruebas.** Se mueven las 8 (`InitialMigrationIntegrationTests`, `StockWorkbook` y las de las filas).

### Fase 4 — `financing`

**Por qué.** Otro lenguaje (fondo, cesión, fideicomiso, garantía) y otro actor, Contabilidad (cap. 14). Su única invariante con la unidad —no se anula con una cesión abierta (D-182)— se resuelve con un puerto.

**Backend.**
- **`package-info.java`** con `allowedDependencies = {"fleet::acl", "organization::acl", "imports::acl", "documents::acl"}`, más `audit::acl` si la corrección del código de un fondo escribe la bitácora.
- **Desde `fleet`** (unas 55 clases):
  - `Fund` y `VehicleFundAssignment`, con sus servicios, repositorios, persistencia, vistas, recursos y ensambladores;
  - `FundsController` (`/funds`), `VehicleFundAssignmentsController` (`/vehicles/{id}/fund-assignments`, la misma ruta servida por otro módulo) y `FundAssignmentsController` (`/fund-assignments/{id}/closure`);
  - la carga de la garantía: `GuaranteeListPolicy`, `ImportGuaranteeListCommand` y su recurso, y su parte de `FleetImportCommandServiceImpl` y `FleetImportsController` (`/fund-assignment-imports`);
  - el tipo de documento `funds` y las tablas de fondos de los `DataVersions`.
- **`fleet` declara el puerto `UnitFundAssignments`** (si una unidad tiene una cesión abierta), que implementa `financing`. Lo usa `VehicleAnnulments`.
- **`fleet` deja de calcular `registerFundAssignment`.** Sale de `VehicleAllowedActions`, de su vista y de su recurso. Las acciones de las cesiones de una unidad las devuelve `GET /vehicles/{id}/fund-assignments`.
  - **Este es un cambio del recurso de la unidad:** se acuerda en las convenciones y el frontend lo adapta en la misma fase.
- **`fleet` conserva:** la carga del impuesto vehicular en `FleetImportCommandServiceImpl`, y los errores que son suyos en `FleetErrors`; los de fondos se mudan.
- **`financing` pide a `fleet` por su fachada:**
  - las unidades de una empresa por placa, con un miembro nuevo para la lista de garantía;
  - el bloqueo de las unidades (`lockUnitsByIds`, que ya existe).
- **El orden de los candados no cambia:** primero la unidad, después el fondo en modo compartido (patrones de persistencia).

**Frontend.**
- **Carpeta nueva `src/app/financing`,** con los 26 archivos de fondos de `fleet`: `FundsStore`, `VehicleFundAssignmentsStore`, la lista de fondos, los diálogos de ceder y cerrar, y el de la carga de garantía.
- **La ruta pasa a `/financing/funds`,** con redirección desde `/fleet/funds`.
- **`FinancingFacade`:**
  - `openGuaranteeImport`, que hoy está en `FleetFacade` y usa la pantalla Cargas;
  - la lectura de las cesiones de una unidad y los flujos de ceder y cerrar, que abre la pestaña Fondos de la ficha de la unidad (regla 1.3 del frontend: una fachada abre los flujos de su propio contexto).

**Documentación.** `contextos/financing.md`, con lo de fondos de `fleet.md` y los mismos ids; el mapa vigente y el diccionario §3.

**Pruebas.**
- Se mueven las 8 de fondos.
- La anulación de una unidad con una cesión abierta se prueba en `fleet` contra el puerto, y la cesión de punta a punta, en `financing`.

### Fase 5 — El servicio publicado de `procurement`

**Por qué.** Hoy `fleet` intermedia los datos de `procurement`. Su puerto `UnitPurchases` define en el lenguaje de `fleet` los pagos, las notas y los usos del saldo, y `fleet` reexpone el costo a `reporting`. El dueño del costo es `procurement`, y debe publicarlo él (*Open Host Service* con *Published Language*, cap. 14).

**Backend.**
- **`procurement::acl`** (creada en la fase 3) suma el costo y la posición de pago de cada unidad, por id y por lote. Recibe los records que hoy están en `UnitPurchases` de `fleet`: `UnitPurchaseCost`, `UnitPaymentPosition`, `UnitPaymentRecord`, `UnitCreditNote`, `UnitBalanceUse` y `UnitAmount`.
- **`UnitPurchases` de `fleet` queda en lo que necesitan sus reglas:** `hasCurrentLine`, `withCurrentLine`, `unitIdsWithCurrentLine`, `valueIsZero`, `returnedUnitIds` y `fetchDataVersion`. Salen `purchaseCostOf` y `paymentFacts`.
- **`fleet` pierde:**
  - de `VehicleViewFactory` y `VehicleQueryServiceImpl`, la posición de pago de la unidad. Se quedan `hasCurrentInvoice` y `returnedToSupplier`, que salen del puerto reducido;
  - `FleetContextFacade.fetchPurchaseCosts`;
  - la parte de pagos de `UnitReportFactsReader`.
- **`GET /vehicles/{id}/purchase-cost`** pasa a un controlador de `procurement`, con la misma ruta y el mismo permiso (`stock.ver_costos`).
- **La posición de pago de la unidad** también la sirve `procurement`. La ruta y su versión por lote se acuerdan primero en las convenciones (§2.2).
- **`reporting`** agrega `procurement::acl` y un `ExternalProcurementService`, y calcula la brecha de días con la entrega de `fleet` y el primer pago de `procurement`. `DayGapPolicy` pasa de `fleet` a `reporting`.

**Frontend.**
- La entidad `Vehicle` pierde la posición de pago.
- **`ProcurementFacade` suma la compra y la posición de pago de una unidad**, por id y por lote, con su propio estado de carga y de error.
- La ficha de la unidad compone el panel «Compra y costo» y la banda de pago desde `ProcurementFacade`, y el Inventario también, si muestra el estado de pago.

**Documentación.** Las convenciones (§2.2, §4.2: el recurso de la unidad cambia), `contextos/fleet.md`, `procurement.md` y `reporting.md`, y el mapa vigente.

**Pruebas.** Las vistas de la unidad, `PurchaseInvoiceUnitPurchasesAdapter`, los reportes, las specs de la ficha de la unidad y el e2e de flota.

**Riesgo:** es la única fase que cambia el contrato de un recurso muy usado. Se integra backend y frontend juntos.

### Fase 6 — Las cuentas bancarias pasan a cuentas por pagar

**Por qué.** Solo las usan los pagos, y hoy `organization` tiene que preguntarle a `procurement` si una cuenta tiene pagos (`BankAccountPayments`): el concepto pertenece a cuentas por pagar.

**Backend.**
- **Desde `organization`** (unas 32 clases), al módulo de cuentas por pagar de `procurement`:
  - `BankAccount` con su servicio, repositorio y persistencia (`company_bank_accounts`);
  - `CompanyBankAccountsController` (`/companies/{id}/bank-accounts`) y `BankAccountsController` (`/bank-accounts/{id}`), con las mismas rutas.
- **Salen:**
  - el puerto `BankAccountPayments` y `SupplierPaymentBankAccountPaymentsAdapter`: `procurement` lee sus propios pagos;
  - los cuatro miembros de cuentas de `OrganizationContextFacade` y su uso en `ExternalOrganizationService`;
  - la tabla de los `DataVersions` de `organization`.
- **Permisos:** los `configuracion.*_cuentas_bancarias` no cambian de código (decisión 2 del mapa objetivo).

**Frontend.** Los 12 archivos de cuentas bancarias pasan de `organization` a `procurement`. La pantalla depende de la decisión 3 del mapa objetivo:
- **Opción propia** en Configuración › Compras y flota, con `mm-company-filter` y `?empresa=`, enlazada desde «Más acciones» de la ficha de la empresa. Es la regla de las colecciones hijas con otro dueño, ya aplicada en Ubicaciones y Colaboradores.
- **Pestaña prestada por `procurement`,** lo que exige una excepción nueva a la regla 1.2 del frontend y su revisión de `DESIGN.md`.

**Pruebas.** Las 6 de cuentas bancarias, y las de los pagos con transferencia y cheque: el bloqueo compartido de la cuenta deja de pasar por la fachada.

### Fase 7 — La salida y el regreso del colaborador, en `organization`

**Por qué.** El dueño de un agregado es el dueño de los comandos que lo cambian (cap. 6). Hoy `iam` sirve `/employees/{id}/termination` y `/reactivation` y le ordena a `organization` dar de baja y reactivar al colaborador. Por eso `OrganizationContextFacade` tiene miembros que solo existen para eso.

**Backend.**
- **`EmployeeTerminationsController` y `EmployeeReactivationsController` pasan a `organization`,** con las mismas rutas, el mismo permiso (`seguridad.registrar_salidas_y_regresos`) y la misma respuesta. Lo que hoy orquesta `EmployeeTenureCommandServiceImpl` de `iam` pasa a un servicio de comandos de `organization`.
- **`organization` declara el puerto `EmployeeAccountConsequences`** en su `interfaces/acl`, y lo implementa `iam`. Tiene dos partes:
  - **la negativa,** antes de escribir nada: los cuatro `403` de la administración de cuentas y la guardia del último superadministrador;
  - **la consecuencia,** en la misma transacción: retirar los roles de esa empresa con `baja_colaborador` y, si la cuenta no tiene otro colaborador activo, abrir el bloqueo `baja`, retirar el resto con `baja_cuenta` y cerrar sus sesiones.
- **El orden de los candados no cambia:** primero el colaborador y después la cuenta (patrones de persistencia).
- **Por qué un puerto y no un evento:** el sistema no publica eventos de integración, la negativa tiene que llegar como `Result` antes de escribir, y la consecuencia es atómica con la salida. Es el patrón que ya usan `CurrencyUsage` y `BankAccountPayments`.
- **Salen de `OrganizationContextFacade`** `terminateEmployee` y `reactivateEmployee`.
- **Se quedan en `iam`:** el vínculo de la cuenta al dar acceso (`linkUserToEmployee`), porque lo decide `iam` y lo escribe el dueño, y el desvínculo (`/employees/{id}/account-unlinking`), que es una decisión sobre la cuenta.
- **Subfase opcional:** las acciones de cuenta de cada fila de Colaboradores las calcula `iam` y la pantalla las compone, en vez de que `EmployeeAllowedActions` de `organization` las calcule con `ProtectedAccounts` y `PendingAccountActivations`.

**Frontend.** Nada: las rutas y la respuesta no cambian. La subfase opcional sí cambia la lista de Colaboradores.

**Pruebas.** Antes de mover nada se escriben pruebas de caracterización de la salida y el regreso, con sus negativas, sus consecuencias y sus carreras (`EmployeeLockTests`, la del último superadministrador). Después se mueven las de `iam` y se corren completas las de `iam` y las de `organization`.

**Riesgo alto:** toca la seguridad de las cuentas. Va con revisión de arquitectura (skill `architecture-review`) antes de integrar.

### Fase 8 — Los catálogos preguntan en vez de recibir banderas

**Por qué.** Hoy `fleet` le escribe a `vehiclecatalog` `has_units` (`markModelsInUse`) y a los tipos de equipo `has_installations` (`markEquipmentTypeInUse`): el proveedor guarda un hecho de su cliente. `location` resuelve el mismo problema preguntando por un puerto (`CurrencyUsage`). Un solo patrón para un solo problema hace el modelo más predecible.

**Backend.**
- **`vehiclecatalog` declara `VehicleModelUsage`**, que implementa `fleet` (si un modelo tiene unidades), y deja de recibir `markModelsInUse`.
- **`equipment` declara `EquipmentTypeUsage`**, que implementa `fleet` (si un tipo se instaló alguna vez).
- **Las columnas `has_units` y `has_installations` se retiran en dos tiempos:**
  1. el código deja de leerlas y escribirlas;
  2. una migración nueva las elimina en una entrega posterior. Una migración aplicada nunca se edita.
- **Los requisitos de la habilitación** se mueven solo si así lo decide el responsable (decisión 5 del mapa objetivo).

### Fase 9 — Renombrar `location` a `reference` (opcional)

**Por qué.** El nombre es parte del lenguaje (cap. 2). `location` no describe monedas, tasas ni tipos de documento, y choca con `Location` de `organization`.

**Qué cambia.**
- **Backend:**
  - el paquete y el `displayName`;
  - `"location::acl"` en los `package-info.java` de `fleet`, `organization`, `procurement` y `legacyimport`;
  - la fachada y las pruebas.
- **Frontend:** la carpeta `location`, las rutas `/location/…` (con redirecciones) y el menú.
- **Documentación y skills.**

**Qué no cambia:** las rutas de la API, las tablas y los códigos de los permisos.

Se hace al final, o no se hace: es el cambio con más archivos y menos efecto sobre el modelo.

### Fase 10 — Preparar el bloque 12

Sin código. Se resuelven con el negocio las preguntas del mapa objetivo (§7, punto 6) y se registran como decisiones en el diccionario. Con eso se diseña:
- **el servicio publicado de `fleet` para las transiciones comerciales** —reservar, firmar, entregar, resolver y completar—, que llamará `leasing`;
- **los puertos `UnitContracts`** (que reemplaza la deducción de D-250) **y `UnitTitleTransfers`** (que reemplaza a `VEHICLE_TRANSFER_REQUIRED` cuando exista el trámite);
- **la identidad del cliente,** separada de `iam`;
- **el lugar de la regla de las 48 horas** desde la firma.

Después, cada contexto nuevo se construye con el orden de la skill `backend-module`, en este orden: `customers` (mínimo), `leasing`, `collections`, `titletransfer` y el portal.

---

## 5. Lista de control de cada fase

**Backend**
- [ ] El módulo tiene su `package-info.java` con `allowedDependencies` y su `interfaces/acl` con `@NamedInterface("acl")`, y `ModularityTests` pasa.
- [ ] Ninguna clase de otro contexto importa su interior: solo su `::acl`, desde su `External<Contexto>Service` (`ForeignAclStaysInTranslationLayerTests`).
- [ ] Los servicios de comandos conservan el patrón `*CommandServiceImpl` en `application.internal.commandservices` (`DeadlockRetry`), y todo `@Scheduled` sigue dentro de `SingleInstanceRun`.
- [ ] Cada tabla queda nombrada por un solo `Jdbc<Contexto>DataVersions`, y cada exportación es un `DocumentKind` del módulo que la sirve.
- [ ] Los controladores movidos conservan su ruta, su `@PermissionRequired` y sus códigos de error.
- [ ] No quedan dos clases con el mismo nombre simple anotadas como bean.
- [ ] Las pruebas se mueven con su código, espejando los paquetes.

**Frontend**
- [ ] La carpeta del contexto tiene sus cuatro capas, y la fachada, solo los miembros que alguien usa.
- [ ] Las rutas declaran los mismos permisos, el menú conserva grupo y acceso, y las URL viejas redirigen.
- [ ] Se actualizan `e2e/screens.ts` y las specs; las imágenes de referencia se regeneran en el CI.

**Documentación**
- [ ] Lo movido pasa del mapa objetivo al vigente, a `contextos/` y al diccionario §3, en el mismo cambio.
- [ ] Las convenciones, si cambió un recurso.
- [ ] Las tres copias de las skills.

---

## 6. Vuelta atrás

Las fases 1 a 7 y la 9 no cambian datos ni esquema: deshacer una es revertir su commit de integración en los dos repositorios. La fase 8 retira columnas en dos tiempos, y la migración que las elimina se aplica solo después de que una entrega completa haya funcionado sin leerlas.

---

## 7. Calendario orientativo

| Semana | Fases |
|---|---|
| 1 | 0 y 1 |
| 2–3 | 2 |
| 4 | 3 |
| 5 | 4 |
| 6–7 | 5 |
| 8 | 6 |
| 9–10 | 7 |
| 11 | 8 |
| Cuando se decida | 9 |
| En paralelo, con el negocio | 10 |

---

## 8. Riesgos

| Riesgo | Mitigación |
|---|---|
| Un acoplamiento oculto que solo aparece al mover | `ModularityTests` lo detecta al compilar el módulo; las pruebas de quien consume se corren en cada fase |
| `reporting` lee tablas por SQL | Las tablas no cambian de nombre; sus pruebas se corren en las fases 2, 4 y 5 |
| La fachada de un contexto crece sin control | Un miembro nuevo solo con un consumidor que lo use (skill `backend-module`, paso 12) |
| Conflictos con trabajo funcional en paralelo | Una fase a la vez, ramas cortas, sin trabajo funcional en el área que se mueve |
| Las skills y el mapa quedan atrás del código | Se actualizan en la misma fase, en las tres copias |
| `dev` se despliega antes del CI | El CI corre en la rama antes de integrar a `develop` |
| La fase 7 debilita una regla de seguridad | Pruebas de caracterización antes de mover y revisión de arquitectura antes de integrar |
