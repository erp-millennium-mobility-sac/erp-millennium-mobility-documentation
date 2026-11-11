# Especificación: facturas de compra (fase 2, punto 5)

**Versión 11 · 10/11/2026: lo que pasó a saldo a favor y lo de cada vehículo, construido (D-248).** Se construyó el mismo día: backend, merge `ba5bbb15` (V127); frontend, merge `96b1a3f6`, con la revisión 76 de `DESIGN.md`; diccionario v1.54. En esta especificación, la versión de la exportación (§4.2.2) y la ficha (§6.4) suman su **Como se construyó (10/11/2026)**. **Versión 10 · 10/11/2026: lo que pasó a saldo a favor y lo de cada vehículo en el pago de la factura (D-248), decidido.** El responsable revisó «Registrar pago» en `dev` ese día y pidió que la nota de un vehículo ya pagado pueda pagar otra factura del mismo proveedor mientras la factura original sigue debiendo el monto completo de sus otros vehículos. Se decidió en el diccionario v1.53 antes de construirlo: el saldo se mide por vehículo (D-248), con V127, y queda registrado en qué se usó cada nota (D-249), con V128 ([especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md), versión 12, §2.2 y §2.3). Se construye en el backend con `feature/unit-level-balance` y en el frontend con `feature/balance-trace-screens` y la revisión 76 de `DESIGN.md`. En esta especificación cambian la versión de la exportación (§4.2.2) y la ficha: el resumen del pago, que dice «Pasó a saldo a favor» donde decía «Pagado de más», y el panel de líneas, con lo de cada vehículo (§6.4), con la marca **Como cambia (10/11/2026, D-248)**. **Versión 9 · 10/11/2026: el pie y lo que falta de un formulario de página, el estado centrado y el ícono de un dato.** El responsable pidió ese día que las acciones al final de un formulario de página se vieran como en Registrar factura, y que todo formulario de página se comportara como el nuevo Registrar pago (D-246): nada dicho al abrir, un campo que dice su error una vez editado y dejado, el botón principal nunca retenido, un resumen de lo que falta arriba, el foco en el primer campo con error y «Guardando los datos…» mientras guarda (revisiones 70 y 71 de `DESIGN.md`; frontend, merges `6a02be91` y `2aa8c4c5`). Pidió también, con capturas, el estado de toda lista centrado bajo su encabezado (revisión 72, merge `d59b2f31`) y el ícono de un dato de una ficha junto a su texto (revisión 73, merge `de2d1c56`). Sin cambios en el backend. En esta especificación cambian: el formulario, cuyo «Registrar factura» ya no se retiene (§6.3), y las revisiones, con la lista y la ficha (§6.8). Cada cambio lleva la marca **Como cambió (10/11/2026)**. **Versión 8 · 09/11/2026: las listas, las hojas y la revisión de los pagos.** El mismo día, después de revisar los pagos, el responsable mandó capturas de las listas y pidió centrar los encabezados de toda lista y que los días alcancen la línea de los filtros, en el componente compartido (revisión 64 de `DESIGN.md`, merge `52cf8b2b`); Comprobante centrado con el tipo de todo documento bajo el número, y las hojas de la orden y de la plantilla sin el azul primario (revisión 65, merge `69887f1b`); y aprobó un solo vocabulario en Compras (D-245, diccionario v1.50; revisión 67, merge `7ae68c2d`). Sin cambios en el backend de las facturas. En esta especificación cambian: la lista, con Comprobante, «UND», los filtros y la barra (§6.2); el resumen y las líneas de la ficha, que dicen «Pendiente» y «Pagado» (§6.4); el aviso de lo facturado de la orden (§6.5); y las revisiones 64, 65 y 67 (§6.8). Cada cambio lleva la marca **Como cambió (09/11/2026)**. **Versión 7 · 09/11/2026: el estado de pago de una factura (D-244) y la revisión del responsable.** El responsable revisó ese día la lista y la ficha de las facturas con datos de demostración en `dev`, y pidió ordenar los botones de la cabecera, dar a los pagos una pantalla más apropiada y saber por qué la lista dice «Factura» en cada fila. Decidió un botón principal y un menú agrupado en toda ficha, el resumen del pago sobre los datos de la factura con solo la tabla de sus pagos en su panel, y el estado de pago en la lista, con el cambio del backend: D-244, en el diccionario v1.49. Se construyó el mismo día, sin migraciones: backend `edd12f9a` a `6230346b`, en `develop` con el merge `4d15256d`; frontend, la revisión 62 de `DESIGN.md`, las acciones de una ficha (`83aff699` a `e6b48de8`, merge `2fcbecbe`), y la revisión 63, el pago de una factura (`8fa29b6e` a `6cbf7243`, merge `cf862569`). En esta especificación cambian: los campos y el filtro de la lista y los campos de la ficha (§4.1, §4.2.1, §4.2.3); la columna «Estado de pago» de la exportación (§4.2.2); las acciones de las filas (§4.5); la lista y la ficha como se construyeron (§6.2, §6.4); las revisiones 62 y 63 (§6.8); y las sentencias de una página de la lista (§7.2). Cada cambio lleva la marca **Como cambió (09/11/2026)**. **Versión 6 · 09/11/2026: la auditoría de punta a punta.** Sin cambios en el código de las facturas. §4.1 tacha la marca **(propuesta)** de la ruta 9, construida el 03/11/2026, y suma la posición de pago de una factura, que llegó con los pagos; §10.3 registra el CI con las dos cuentas, resuelto ese día. **Versión 5 · 04/11/2026: la segunda revisión de punta a punta.** El mismo día, cuatro revisores independientes volvieron a leer el plan v4 de punta a punta, y el responsable pidió tres cambios de pantalla. Las correcciones y los pedidos se integraron en `develop` el 04/11/2026, sin migraciones: backend `851b4d09`, el merge de `feature/review-2-findings` (de `a126216f` a `607e6756`, sobre `1bd767f1`), y frontend `3a49f141`, el merge de su rama (de `a8ccc252` a `c64888e9`, sobre `2908405f`), con las revisiones 40 y 41 de `DESIGN.md`. En esta fase cambian: la línea de concepto en la anulación y en el `PUT` (§2.2); la fila de `kept_files` antes que el objeto en Spaces (§2.5.5); el contrato C10, «Importar desde Excel», que reemplaza «Pegar columnas» (§11, con §4.1, §4.3, §5.1 y §6.3); la barra de la lista (revisión 40, §6.2); los grupos de la tarjeta de datos (revisión 41, §6.3); y el estado del formulario (§6.3). Cada cambio lleva la marca **Como se corrigió (04/11/2026, segunda revisión)**, y §10.4 resume lo que la revisión halló en esta fase. **El diccionario, 04/11/2026:** el responsable aprobó ese día la propuesta que esta especificación dejaba pendiente, y la documentación `7cc29f4` la aplica en el diccionario v1.39: §2.7 dice una sola regla de descarte para los archivos de las facturas, de las notas y de los documentos de una unidad, en el párrafo y en la fila de `storage_key` (§2.5.5, §9, §10.3). **Versión 4 · 04/11/2026:** las correcciones de la revisión de punta a punta del plan v4 (03 y 04/11/2026), integradas en `develop` el 04/11/2026: backend `44136ae1` (la rama `feature/review-findings`, de `5c77bc8f` a `df6b7276`, sin migraciones) y frontend `cc782649` (la rama, de `6386f2fa` a `1ef10642`). Lo que cambió de comportamiento lleva la marca **Como se corrigió (04/11/2026)** con su commit: el descarte del archivo al revertir (§2.5.5), las unidades que se vinculan leídas en una sentencia (§2.3), el permiso de facturas antiguas por marca (§4.2.4), la búsqueda por el número normalizado (§4.2.1), lo que una factura trae a una línea de la orden sin año (PRC-R50), la línea de concepto que se lee y se pasa por alto (§2.2), el alcance de la sesión en el formulario y lo que la pantalla busca en el servidor (§6). También corrige dos textos que el código nunca siguió, el orden de los candados (§2.4) y el modo de la orden, y las citas de pruebas que quedaron viejas con las fases 4 y 5 y con la revisión (§5, §7). §10 resume lo que la revisión halló en esta fase y cómo se cerró. **Versión 3 · 03/11/2026:** el registro de lo construido. La fase se integró en `develop` ese día: backend `16f69af3`, con V92 a V94 (la rama, hasta `c37dda82`), y frontend `2dc6505c` (la rama, hasta `b70af927`). Esta versión corrige lo que el código hizo distinto de lo especificado y nombra la prueba real de cada regla (§5, §7). **Versión 2 · 03/11/2026** (las ocho preguntas de la versión 1, del 02/11/2026, quedan decididas por D-178 a D-182, delegadas por el responsable) · **Estado:** construida e integrada, con las correcciones de las dos revisiones del 04/11/2026 y el contrato C10; falta la sonda en vivo, que necesita la contraseña de verificación del responsable (§7.4, §10) · **Punto de partida:** backend `b1264ed5` (hasta V91), frontend `88312e13`, documentación `6ebb900` con el diccionario v1.38 (D-157 a D-177), las decisiones D-178 a D-182 y el [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) v3. Si esta especificación y el diccionario no coinciden, manda el diccionario.

**Fuentes:** diccionario §2.5, §2.6, §2.7, §4.8, §5.6, §8.1 a §8.3, §9.2 a §9.8, §12.2, §13.1 y las decisiones D-21 a D-24, D-101, D-102, D-104, D-132, D-146, D-151 a D-182 y, desde la versión 7, D-244 (diccionario v1.49); [convenciones de la API](convenciones-de-la-api-millennium-mobility.md); [contextos/procurement.md](contextos/procurement.md), [fleet.md](contextos/fleet.md) y [location.md](contextos/location.md); [procesos y pantallas](procesos-y-pantallas-millennium-mobility.md) P4 y §3.2; las skills `backend-module`, `frontend-feature` y `architecture-review`; la guía funcional del cliente, columnas 4, 6, 35 a 37 y 46 a 49.

**Cómo leerlo.**
- §1 dice qué decisiones implementa y a qué pasos del plan sirve.
- §2 es el modelo del backend: paquetes, clases, transacciones, candados y contratos entre contextos.
- §3 es el esquema, migración por migración.
- §4 es el contrato de la API, que el paso 5.0 pasa a las convenciones.
- §5 son las reglas con sus pruebas, en ✅ desde el 03/11/2026, cada una con el nombre real de su prueba.
- §6 son las pantallas.
- §7, las pruebas; §8, la secuencia de commits; §9, las preguntas de la versión 1 y la decisión que cerró cada una.
- §10, las dos revisiones de punta a punta del 03 y 04/11/2026: lo que halló cada una en esta fase, cómo se cerró y lo que queda.
- §11, el contrato C10: la plantilla de las líneas y su importación desde Excel (04/11/2026).
- Lo marcado **(propuesta)** es un nombre que ninguna decisión fija.
- En la columna *Fuente* de las tablas de reglas, §*n* es del diccionario, como en los contextos. En el texto, una sección del diccionario o de las convenciones lo dice; un §*n* sin más es de esta especificación.

---

## 1. Alcance

**Implementa** D-157 (registro a mano), D-158 (ISC por factura), D-159 (precio con impuestos, costo sin IGV calculado), D-160 (número, fecha, líneas y vínculos), D-161 (lo fiscal no se edita, anulación que no anula unidades), D-162 y D-172 (la orden), D-163 (permisos y alcance), D-169 (moneda funcional), D-173 (solo líneas de vehículo), D-174 (archivos), D-175 (rastro y filtro de Inventario), D-176 (siembra en dos tiempos), D-178 (revisión, alcance y `PUT` de lo no fiscal), D-179 (la unidad que se vincula conserva su color y su motor; valor de factura mayor que cero), D-180 (base e IGV calculados en la exportación), D-181 (tipo de tasa de compra de cada país) y D-182 (una unidad con cesión abierta no se anula). Aplica las ya cerradas D-21 a D-24, D-104 (vía explícita de anulación de la unidad, que llega con este paso), D-132 (patrón de `has_discount_column`), D-146 (escala de los montos), D-151 a D-153 (tasa propuesta, tasa de venta en Perú, la unidad nace con la factura).

**Sirve a los pasos del plan:**

| Paso | Qué cubre esta especificación |
|---|---|
| 5.0 | §4 completo: rutas, cuerpos, códigos, avisos, `allowedActions`, versiones |
| 5.1 | §3.2: la migración de permisos y sus constantes |
| 5.2 | §2.5 (fachada de `location`) y §3.3 (`countries.purchase_exchange_rate_type`) |
| 5.3 | §2.5: `FleetContextFacade`, los puertos `UnitPurchases` y `UnitContracts`, `VehicleRegistrations`; las lecturas por lote de `vehiclecatalog` y `organization` |
| 5.4 | §2.1 (fleet), §4.2.10, FLT-R48 a FLT-R50 y FLT-R54: la anulación explícita de la unidad, su guarda y la cesión abierta (D-182) |
| 5.5 | §3.4: las dos tablas, sus disparadores, RLS, permisos de la cuenta y `data_versions`; `CurrencyUsage` |
| 5.6 | §2.2 a §2.4 y §4.2.4: el agregado y el registro en una transacción |
| 5.7 | §4.2.5 y §4.2.6: cambiar archivo u orden, anular |
| 5.8 | §4.2.1 a §4.2.3, §4.2.7, §4.2.9 y §4.2.10: lista, ficha, archivos, exportación, lo pendiente de la orden, el filtro de Inventario |
| 5.9 a 5.11 | §6 |
| 5.12 | §7.4 (sondas) y el cierre documental de §8 |

**Queda fuera:** el lector del XML (candidato, §9.8 del diccionario); las líneas de concepto (D-173); las notas de crédito y los pagos, que solo se anticipan con el puerto «dependientes de la factura» (§2.5.4); el costo por unidad en la ficha de la unidad (fase 4).

---

## 2. Modelo del backend

### 2.1 Paquetes y clases

Las capas son las de la skill `backend-module`: dominio sin marco, infraestructura con JPA, fachadas en `interfaces/acl`. Lo nuevo va en piezas con nombre (F5), nunca engordando `PurchaseOrderCommandServiceImpl` ni `VehicleCommandServiceImpl` (hallazgo A28).

**`procurement`** (`ky.millennium.mobility.erp.procurement`). `package-info.java` suma `"fleet::acl"` a `allowedDependencies`, que queda `{"shared", "organization::acl", "location::acl", "iam::acl", "documents::acl", "vehiclecatalog::acl", "fleet::acl"}`. No forma ciclo: `fleet` no depende de `procurement`, y lo que `fleet` necesita de compras son puertos que `fleet` declara y `procurement` implementa, como `CurrencyUsage` (§2.5).

| Capa | Clases nuevas |
|---|---|
| `domain/model/aggregates` | `PurchaseInvoice` |
| `domain/model/entities` | `PurchaseInvoiceItem` |
| `domain/model/valueobjects` | `InvoiceDocumentType`, `InvoiceNumber`, `InvoiceAmount`, `AppliedExchangeRate`, `TaxRate`, `InvoiceLineStatus`, `InvoiceAnnulmentReason`. `UploadedFileType` no va aquí: vive en `shared/domain/model/valueobjects`, porque también lo usa `fleet` en la habilitación, y lo crea la fase que se integre primero, extrayéndolo de `PurchaseOrderTemplateDocument.sniffMediaType`, que pasa a delegar en él |
| `domain/model/commands` | `RegisterPurchaseInvoiceCommand`, `PurchaseInvoiceItemInput`, `UpdatePurchaseInvoiceCommand`, `AnnulPurchaseInvoiceCommand`, `InvoiceFileUpload` |
| `domain/model/queries` | `GetPurchaseInvoiceByPublicIdQuery`, `SearchPurchaseInvoicesQuery`, `PurchaseInvoiceSearchFilter`, `CheckPurchaseInvoiceVinsQuery` |
| `domain/services` | `InvoiceDatePolicy` (hoy en la zona de la empresa, fecha futura, antigüedad), `OrderInvoicingPolicy` (lo pendiente por modelo y año y las desviaciones) |
| `domain/repositories` | `PurchaseInvoiceRepository` |
| `domain/exceptions` | `PurchaseInvoiceTotalMismatchException`, `PurchaseInvoiceIscRequiredException`, `PurchaseInvoiceIscForbiddenException`, `PurchaseInvoiceIscExceedsAmountException`, `PurchaseInvoiceDuplicateUnitException`, `PurchaseInvoiceAnnulledException`; `NaturalKeyTakenException.NaturalKey` suma `PURCHASE_INVOICE_NUMBER` y `PURCHASE_INVOICE_UNIT` |
| `application/commandservices` | `PurchaseInvoiceCommandService` |
| `application/queryservices` | `PurchaseInvoiceQueryService`, `PurchaseInvoiceView`, `PurchaseInvoiceItemView`, `PurchaseInvoiceFile`, `VinCheckView`, `OrderInvoicingView` |
| `application/internal/commandservices` | `PurchaseInvoiceCommandServiceImpl` y sus piezas: `PurchaseInvoiceReferences` (empresa, proveedor, moneda, marcas y modelos, con sus candados), `PurchaseInvoiceRates` (tasa 1 o tasa vigente, y la comparación con la vista), `PurchaseInvoiceUnits` (el cruce de VIN y el alta o vínculo por la fachada de `fleet`), `PurchaseInvoiceOrderLinks` (vínculo, desviaciones), `PurchaseInvoiceFiles` (verificación, guardado y descarte al revertir), `PurchaseInvoiceAnnulments` (anulación y bitácora) |
| `application/internal/queryservices` | `PurchaseInvoiceQueryServiceImpl`, `PurchaseInvoiceViewFactory`, `PurchaseInvoiceAllowedActions`, `PurchaseInvoiceVinChecks`, `PurchaseOrderInvoicingReader` (lo que la ficha de la orden muestra) |
| `application/internal/policies` | `PurchaseInvoiceAccessPolicy`; `ProcurementReadPermissions` suma `INVOICES = compras.ver` |
| `application/internal/errors` | `PurchaseInvoiceErrors`; `PurchaseOrderErrors.hasCurrentInvoices()` |
| `application/internal/outboundservices/acl` | `ExternalFleetService` (nuevo) con sus records de traducción `UnitMatch`, `InvoicedUnit`, `UnitFacts`; `ExternalLocationService`, `ExternalOrganizationService` y `ExternalVehicleCatalogService` suman los miembros de §2.5 |
| `application/internal/outboundservices/dependents` | `PurchaseInvoiceDependents` (puerto, §2.5.4) |
| `application/internal/outboundservices/files` | `XmlWellFormedness` (puerto, implementado en infraestructura) |
| `application/internal/outboundservices` | `ProcurementDataVersions` suma `purchaseInvoices()` |
| `application/documents` | `ProcurementDocumentKinds` suma `PURCHASE_INVOICES = "purchase-invoices"` |
| `infrastructure/persistence/jpa` | `PurchaseInvoicePersistenceEntity` (extiende `RevisedAbstractPersistenceEntity`: `created_at`, `updated_at`, `created_by` y `updated_by`, D-42 y D-178), `PurchaseInvoiceItemPersistenceEntity` (sin clase base: el cuadro no le da columnas de registro), `PurchaseInvoiceJpaRepository`, `PurchaseInvoiceItemJpaRepository`, `PurchaseInvoiceRepositoryImpl`, `PurchaseInvoicePersistenceAssembler` |
| `infrastructure/persistence/jdbc` | `JdbcProcurementDataVersions` suma `purchase_invoices` y `purchase_invoice_items` |
| `infrastructure/currencies` | `PurchaseInvoiceCurrencyUsageAdapter` |
| `infrastructure/fleet` | `PurchaseInvoiceUnitPurchasesAdapter` (implementa `UnitPurchases` de `fleet`) |
| `infrastructure/xml` | `JaxpXmlWellFormedness` (SAX con `FEATURE_SECURE_PROCESSING`, `disallow-doctype-decl` y sin entidades externas) |
| `interfaces/rest` | `PurchaseInvoicesController`, `PurchaseInvoiceVinChecksController` |
| `interfaces/rest/resources` | `RegisterPurchaseInvoiceResource`, `PurchaseInvoiceItemInputResource`, `UpdatePurchaseInvoiceResource`, `AnnulPurchaseInvoiceResource`, `PurchaseInvoiceResource`, `PurchaseInvoiceItemResource`, `VinCheckResource`, `OrderInvoicingResource`; reutiliza `MoneyResource` |
| `interfaces/rest/transform` | `RegisterPurchaseInvoiceCommandFromResourceAssembler`, `UpdatePurchaseInvoiceCommandFromResourceAssembler`, `PurchaseInvoiceResourceFromViewAssembler`, `PurchaseInvoiceSpreadsheetColumns` |

**`fleet`**. `package-info.java` no cambia. Se abre `fleet/interfaces/acl` con su `package-info.java` y `@NamedInterface("acl")`.

| Capa | Clases nuevas o que cambian |
|---|---|
| `interfaces/acl` | `FleetContextFacade` y sus records (`PurchasedUnitMatch`, `PurchasedUnitMatchKind`, `ExistingUnit`, `RegisterPurchasedUnitsRequest`, `PurchasedUnitLine`, `PurchasedUnit`, `UnitSummary`); los puertos `UnitPurchases` y `UnitContracts` |
| `application/acl` | `FleetContextFacadeImpl` |
| `application/internal/commandservices` | `VehicleRegistrations` (extraído del alta de `VehicleCommandServiceImpl`: búsqueda del VIN con una elevación, estado inicial, camino de alta, primera fila del historial, `markModelInUse`), `PurchasedUnitRegistrations` (el lote, sobre `VehicleRegistrations`), `VehicleAnnulments` (valor cero, sin contratos y sin cesión abierta, D-182) |
| `application/internal/outboundservices/purchases` | `UnitPurchaseFacts` y `UnitContractFacts`: reúnen los beans de los puertos (`List<UnitPurchases>`, `List<UnitContracts>`), como `location` reúne los de `CurrencyUsage` |
| `domain/model/aggregates` | `Vehicle.annul(Instant)` y `Vehicle.isAnnulled()`; `annulledAt` deja de ser `final` |
| `domain/model/commands` | `AnnulVehicleCommand` |
| `domain/repositories` | `VehicleFundAssignmentRepository.existsOpen(Long vehicleId)`: una cesión sin fecha de fin (D-182) |
| `domain/model/queries` | `VehicleSearchFilter` suma `@Nullable Boolean hasCurrentInvoice` y `Set<Long> unitsWithCurrentInvoice`: las unidades con línea vigente, que el servicio pide antes a `UnitPurchaseFacts`, para que el repositorio filtre sin leer las tablas de compras |
| `application/internal/errors` | `FleetErrors.vehicleAnnulled()`, `modelYearLocked()`, `valueNotZero()`, `hasContracts()`, `fundAssignmentOpen()` |
| `application/queryservices` | `VehicleView` y `VehicleExportRow` suman `hasCurrentInvoice`; `VehicleAllowedActionsView` suma `annul` |
| `interfaces/rest` | `VehiclesController`: `POST /{id}/annulment` con `@PermissionRequired(PermissionCodes.Stock.ANNUL_VEHICLE)`; filtro `hasCurrentInvoice` en la lista y la exportación |
| `application/documents` | `FleetDocumentKinds`: la versión del Inventario suma `UnitPurchaseFacts.fetchDataVersion()` |

**`location`, `organization`, `vehiclecatalog`, `documents`, `iam`, `shared`:**

| Contexto | Cambio |
|---|---|
| `location` | `Country.purchaseExchangeRateType` (reutiliza `ExchangeRateType`); `ExchangeRateRepository.findEffective(country, from, to, type, onOrBefore)`; `EffectiveExchangeRateQueryService` (`application/queryservices`, con su `…Impl`), que lee la tasa vigente; `LocationContextFacade.fetchEffectiveRate`, que la pide a ese servicio, y el record `EffectiveRate`; `CountrySummary` suma `taxName`, `taxRate` y `purchaseExchangeRateType`; `CountryResource` suma `purchaseExchangeRateType`, de solo lectura hasta que haga falta editarlo (D-181) |
| `organization` | `OrganizationContextFacade` suma cuatro miembros y el record `PurchasingCompany` (§2.5.2) |
| `vehiclecatalog` | `VehicleCatalogContextFacade` suma tres miembros (§2.5.3) |
| `documents` | `KeptFileStore.discardRolledBack(String storageKey)` (§2.5.5) |
| `iam` | Solo la migración V(n) |
| `shared` | `PermissionCodes.Procurement.REGISTER_INVOICES`, `REGISTER_OLD_INVOICES`, `ANNUL_INVOICES` y `PermissionCodes.Stock.ANNUL_VEHICLE`, en el mismo commit que su migración (`PermissionCatalogConsistencyTests`) |

### 2.2 El agregado y sus reglas

**`PurchaseInvoice`** es la raíz; sus líneas son entidades del agregado (`PurchaseInvoiceItem`) y se guardan con él (`cascade` `PERSIST` y `MERGE`, nunca `ALL` ni `orphanRemoval`). Dos constructores, como pide la skill: el que registra una factura nueva validando, y el que la reconstituye desde la base.

| Miembro | Regla que protege |
|---|---|
| Constructor de registro | De 1 a 100 líneas (D-160); ningún vehículo dos veces (`PurchaseInvoiceDuplicateUnitException`, diccionario §9.4); `grossTotal` igual a la suma exacta de `invoicedAmount` (`PurchaseInvoiceTotalMismatchException`, D-159); con `hasIscColumn`, cada línea de vehículo con `iscAmount` de 0 al monto de la línea, y sin él, ninguna (`…IscRequired…`, `…IscForbidden…`, `…IscExceedsAmount…`, D-158); `exchangeRateDate` no posterior a `invoiceDate`; en la moneda funcional, tasa 1 y `exchangeRateDate = invoiceDate` (D-169). Los montos llegan como `InvoiceAmount`, que ya validó su escala contra los decimales de la moneda (D-146) y que es mayor que cero (D-179) |
| `relinkOrder(@Nullable Long orderId)` | Una factura anulada no cambia (`PurchaseInvoiceAnnulledException`, D-172). Desvincular se puede siempre |
| `replaceFile(String key)`, `replaceXml(String key)` | Igual: solo una factura vigente. El archivo, el XML y la orden son lo único que cambia, nada fiscal (D-178) |
| `annul(InvoiceAnnulmentReason, Instant, Long annulledBy)` | Una sola vez; marca cada línea vigente `ANULADA` con el mismo momento, porque la anulación es de toda la factura (D-161) y una unidad tiene a lo sumo una línea vigente (D-160): si las líneas siguieran vigentes, la factura que la reemplaza no podría vincular sus unidades |
| `isAsOpenedAt(Instant)` | El candado de `updatedAt`, comparado al milisegundo (convenciones §3), como el borrador de la orden (D-138, D-178) |
| `calculatedBase()`, `calculatedTax()` | D-159 y D-180: en una factura, base = total ÷ (1 + `taxRate`), redondeada mitad hacia arriba a los decimales de la moneda, e IGV = total − base; en una boleta u otro, base = total e IGV = 0. Nunca se guardan (diccionario §2.2) |
| `iscTotal()` | La suma del ISC de las líneas, o nada sin `hasIscColumn` |
| `unitsByModelYear()` | Cuántas líneas vigentes hay por modelo y año, para lo pendiente de la orden (D-162) |

**`PurchaseInvoiceItem`**: `vehicleId`, `modelId`, `modelYear`, `commercialBrandId`, `invoicedAmount`, `iscAmount`, `status`, `annulledAt`. Se construye solo como línea de vehículo (D-173): la estructura admite `concept` (D-22), y el constructor de esta versión no lo recibe. `annul(Instant)` es el único cambio que admite.

**Como se corrigió (04/11/2026)** (backend `a7afefd8`, hallazgo B17). La base admite una línea de concepto, sin unidad, modelo ni marca (D-22), pero el dominio se negaba a leerla y varias reglas juntaban su unidad o su marca nulas: una línea así, escrita por SQL o por una carga, respondía `500` al leer la factura. Ahora `PurchaseInvoiceItem` la reconstituye con `vehicleId`, `modelId` y `commercialBrandId` nulos y año 0, e `isConcept()` lo dice. Las reglas de las unidades la pasan por alto: no repite una unidad, no cuenta en `unitsByModelYear()` ni en las marcas de `allowedActions`, y la ficha la muestra sin unidad. Las notas, su anulación y los pagos también la pasan por alto. Registrar una línea de concepto sigue sin soporte (D-173). Prueba: `ConceptLinesTests.aConceptIsNoUnitOfTheInvoice`. Desde las fases 4 y 5 una línea cambia también por sus notas: pasa a `ANULADA` cuando sus notas la dejan en cero (D-191) y vuelve a vigente cuando la anulación de una nota le devuelve valor (D-193).

**Como se corrigió (04/11/2026, segunda revisión)** (backend `76fa8f0d`). La primera corrección dejó dos lugares que todavía juntaban la marca nula de una línea de concepto: `PurchaseInvoiceCommandServiceImpl.brandsOf` armaba un conjunto inmutable con la marca comercial de cada línea, que no admite un nulo, así que anular una factura con un concepto o cambiar su orden respondía `500`. Ahora deja fuera las marcas nulas, como ya hacían la ficha y las notas. Prueba: `PurchaseInvoiceChangesIntegrationTests.anInvoiceWithAConceptChangesItsOrderAndIsAnnulled`.

**Objetos de valor:**

| Clase | Qué valida |
|---|---|
| `InvoiceDocumentType` | `FACTURA("factura")`, `BOLETA("boleta")`, `OTRO("otro")`, guardados en minúscula como toda lista cerrada del diccionario; se aceptan en cualquier mayúscula. `grantsTaxCredit()` es verdadero solo en `FACTURA` (D-159) |
| `InvoiceNumber` | `normalize(String)`: recorta, pasa a mayúsculas y quita los espacios y separadores Unicode, como §5.3 de las convenciones, pero conserva el guion: lleva a `-` los guiones de Unicode (`\p{Pd}`) y el signo menos (U+2212), igual en el backend y en el frontend (`invoice-number.ts`), antes de revisar el formato de Perú; si el texto termina en un guion seguido de dígitos, quita los ceros a la izquierda de ese correlativo (`F001-00000123` → `F001-123`; `F001-0` se rechaza). `holdsPeruvianFormat()`: serie de exactamente cuatro caracteres `[A-Z0-9]`, un guion y un correlativo de 1 a 8 dígitos mayor que cero (D-160). Fuera de Perú no se verifica el formato, pero sí se normaliza. Desde el 04/11/2026, `searchForm(String)` da el texto de una búsqueda normalizado como número, o nada si no puede serlo: vacío, de más de 30 caracteres o un correlativo de ceros (§4.2.1) |
| `InvoiceAmount` | Un monto de la moneda de la factura: no nulo, mayor que cero, a lo sumo `999999999999.99` y con a lo sumo los decimales de la moneda |
| `AppliedExchangeRate` | `rate` mayor que cero con a lo sumo seis decimales (`NUMERIC(12,6)`) y su `rateDate` |
| `TaxRate` | De 0 a menos de 1, con a lo sumo cuatro decimales (`NUMERIC(5,4)`) |
| `InvoiceLineStatus` | `VIGENTE`, `ANULADA`, como los estados guardados de la orden (D-132) |
| `InvoiceAnnulmentReason` | `ERROR_REGISTRO`, `ANULADA_POR_EMISOR` (D-161) |
| `UploadedFileType` (de `shared`) | PDF, PNG, JPEG o WebP leídos de los primeros bytes, como hoy la plantilla (D-174). Lo comparten compras y flota; si la fase 3 se integra antes, ya existe |

**Políticas puras:**
- `InvoiceDatePolicy` (`domain/services`), con `Clock` y la zona IANA del país de la empresa (diccionario §2.6; hallazgo A29): `today(zone)`, `isFuture(date, zone)`, `ageInDays(date, zone)` y `isOld(date, zone)`, con «antigua» = más de 20 días (§9.3; la guía: «si supera 20 días»).
- `OrderInvoicingPolicy` (`domain/services`): recibe las líneas de vehículo de la orden agrupadas por `(modelo, año)` y las unidades facturadas por grupo, y devuelve lo pendiente y las desviaciones de una factura nueva (§4.4). Una línea de orden sin año, anterior a V82, cuenta para el modelo en cualquier año que ninguna otra línea de esa orden nombre. **Como se corrigió (04/11/2026)** (backend `10850e34`, hallazgo B12). Cada año de la factura se comparaba solo con lo que le falta a la línea sin año, así que 3 unidades de 2025 y 3 de 2026 pasaban contra 5 sin año. Ahora lo que la factura trae a una misma línea de la orden se suma antes de comparar: 6 contra 5 es una desviación. La pregunta sigue con una fila por modelo y año, cada una con las cifras de su línea. Prueba: `OrderInvoicingPolicyTests.theYearsAnInvoiceBringsIntoALineWithoutYearAreAddedUp`.
- `PurchaseInvoiceAccessPolicy` (`application/internal/policies`), sobre `AccessScope`:
  - `sees(scope, companyId)`: `compras.ver` actúa en la empresa (`actsOn`). Una factura se ve por empresa (D-163), sin marca ni sede.
  - `registers(scope, companyId, brandIds)`: `compras.registrar_factura` actúa en toda la empresa (`actsOnWholeCompany`) y en cada marca, sin sede (`actsOnWholeCompanyBrand`, D-101, D-102, D-163).
  - `registersOld(scope, companyId, brandIds)`: lo mismo con `compras.registrar_factura_antigua`. Desde el 04/11/2026, si no alcanza la marca de una línea, el registro responde `422 PURCHASEINVOICE_TOO_OLD`, como la fecha, y no un `403` (§4.2.4).
  - `annuls(...)` y `edits(...)`: con `compras.anular_factura` y `compras.registrar_factura`, con el mismo alcance que registrar: toda la empresa y la marca de cada línea, sin sede (D-178).

### 2.3 Servicios de aplicación y transacciones

`PurchaseInvoiceCommandServiceImpl` (`@Service @Transactional`) queda envuelto por `DeadlockRetry` por su nombre y paquete. Todo lo que hace va dentro de su transacción; lo único que sale de ella es el objeto de Spaces, que se descarta si la transacción se revierte (§2.5.5).

| Operación | Transacción | Qué escribe |
|---|---|---|
| `handle(RegisterPurchaseInvoiceCommand)` | Una, `REQUIRED`. La fachada de `fleet` corre dentro (`MANDATORY`) | Las unidades nuevas y su primera fila del historial (por `fleet`), la factura, sus líneas, `kept_files` |
| `handle(UpdatePurchaseInvoiceCommand)` | Una | La factura (`purchase_order_id`, `file_path`, `xml_path`), `kept_files`, filas de la bitácora |
| `handle(AnnulPurchaseInvoiceCommand)` | Una | La factura y sus líneas, la fila delicada de la bitácora |
| `VehicleCommandServiceImpl.handle(AnnulVehicleCommand)` | Una, en `fleet` | `vehicles.annulled_at` |

**El registro, paso a paso.** Cada paso responde el primer fallo y no escribe nada; el orden fija qué código sale cuando fallan varios (§4.2.4):
1. Forma del pedido (Bean Validation, `400`) y forma de los archivos (`422`, sin leer la base).
2. Empresa (`fetchPurchasingCompanyForShare`), proveedor (`SupplierRepository.findByPublicIdForShare`, que ya existe) y moneda (`fetchCurrencySummaryByIsoCodeForShare`, que ya existe): existen, se leen, actúan y están activos. La moneda en modo compartido sostiene el candado de decimales (PRC-R26).
3. Montos: escala, suma e ISC (dominio).
4. Número: normalización, formato en Perú y duplicado entre las no anuladas.
5. Fecha: futura y antigua.
6. Tasa: 1 en la moneda funcional; si no, la vigente por `fetchEffectiveRate`, comparada con la que vio la persona (A19).
7. Marcas y modelos de las líneas, en lote (§2.5.2, §2.5.3); permiso sobre cada marca y, en una factura antigua, el de facturas antiguas sobre cada marca (desde el 04/11/2026, §4.2.4).
8. VIN repetidos dentro del pedido.
9. Orden, si viene: `PurchaseOrderRepository.findByPublicIdForUpdate`, enviada, de la misma empresa y proveedor.
10. Unidades: `FleetContextFacade.matchPurchasedUnits(…, lockExisting = true)`, que bloquea las unidades existentes de la empresa; con ese resultado, otra empresa, anulada, modelo, año o marca distintos, y otra línea vigente (consulta propia sobre las unidades ya bloqueadas). El motor y el color de una línea que vincula no se comparan: la unidad conserva los suyos (D-179).
11. Desviaciones de la orden, salvo `confirmOrderDeviation: true`.
12. Los colores de las líneas nuevas, en lote (`fetchColorSummariesByPublicIds`), y `FleetContextFacade.registerOrLinkPurchasedUnits`, que valida lo que es de `fleet` para las unidades nuevas —con sus marcas comerciales en modo compartido, el último candado de §2.4— y recién entonces escribe.
13. Guarda la factura con sus líneas (la cabecera antes que las líneas, porque el disparador del ISC de cada línea lee su cabecera, como PRC-R08); un único que la base detiene en una carrera se traduce con `UniqueViolations` a `409`.
14. Guarda los archivos en `kept_files` bajo `purchase-invoices/{publicId}/document/{sha256}` y `purchase-invoices/{publicId}/xml/{sha256}`, apunta `file_path` y `xml_path`, y registra el descarte para el caso de reversión.
15. Responde lo que guardó (convenciones §6.2: la respuesta de una escritura no se vuelve a filtrar). Desde la fase 5, antes de responder pasa a la factura los pagos de su orden (`PurchaseInvoiceOrderLinks.paymentsPassTo`, [especificación de los pagos](especificacion-pagos-a-proveedores-2026-11-03.md)); si eso falla, todo lo anterior se revierte (PRC-R65).

**Rendimiento.** Las llaves son `GENERATED ALWAYS AS IDENTITY`, así que Hibernate no agrupa las inserciones: hay una por unidad nueva, una por fila del historial y una por línea. Todo lo demás es constante por factura: una lectura por clase de referencia, una elevación por llamada a la fachada (nunca una por unidad, hallazgo A4) y un `markModelsInUse` por lote. Si la línea base (§7.2) pasa del objetivo, el adaptador escribe las líneas con un `INSERT … SELECT unnest(…)`, sin cambiar el contrato.

**Como se corrigió (04/11/2026)** (backend `61ebde79`, hallazgo B15). La frase «todo lo demás es constante» no valía para las líneas que vinculan unidades existentes: después del candado, cada unidad se leía y se refrescaba una por una, en `procurement` y otra vez en `fleet`, así que las sentencias crecían con esas líneas. La prueba de sentencias solo registraba líneas nuevas y no lo veía. Ahora `VehicleRepositoryImpl.readAllLocked` lee en una sentencia, por `id`, las unidades que el candado devolvió: una copia que la transacción ya leyó bajo ese candado se usa tal cual, una leída antes se refresca y las demás se leen juntas. Vincular N unidades cuesta el candado y una lectura, sea cual sea N, y el segundo candado, el de `fleet`, ya no las vuelve a leer. Prueba: `PurchaseInvoiceRegistrationStatementsTests.theQueriesOfARegistrationThatLinksExistingUnitsDoNotGrowWithItsLines`.

### 2.4 Orden de los candados

Una sola secuencia para todas las escrituras de facturas, para que dos de ellas, o una factura y otra operación, no se interbloqueen:

| # | Qué | Modo | Quién más lo toma |
|---|---|---|---|
| 1 | La factura, en `PUT` y anulación | `FOR UPDATE` | Las escrituras de facturas; desde las fases 4 y 5, también el registro y la anulación de una nota y el registro de un pago |
| 2 | La empresa | `FOR SHARE` (por `organization`, `fetchPurchasingCompanyForShare`) | Su desactivación |
| 3 | El proveedor | `FOR SHARE` | Su cambio de estado y sus contactos (PRC-R24) |
| 4 | La moneda | `FOR SHARE` | La corrección de sus decimales (PRC-R26) |
| 5 | La orden vinculada | `FOR NO KEY UPDATE` (`PurchaseOrderJpaRepository.lockByPublicId`) | El envío, el cumplimiento y la anulación de la orden; desde la fase 5, un pago a la orden |
| 6 | Las unidades existentes de la empresa, por `id` ascendente | `FOR NO KEY UPDATE`, el mismo de `VehicleRepository.findByIdForUpdate`. `procurement` las bloquea al cruzar los VIN (paso 10 de §2.3) y `fleet` otra vez al vincularlas, en la misma transacción | Toda escritura de la unidad: edición, caminos, VIN, anulación |
| 7 | Las marcas comerciales de las unidades nuevas, por `id` ascendente | `FOR SHARE` (por `organization`, `fetchCommercialBrandReferencesByIdsForShare`), que toma `PurchasedUnitRegistrations` en `fleet` (paso 12 de §2.3) | Su desactivación y su reactivación (`FOR NO KEY UPDATE` sobre la marca) |

**Como se corrigió (04/11/2026), solo el texto.** La versión 3 ponía las marcas de las unidades nuevas en la fila 2, junto con la empresa, y la orden en `FOR UPDATE`. El código nunca siguió ese texto: la orden se bloquea en `FOR NO KEY UPDATE` desde la fase 2, y `fleet` toma las marcas en modo compartido al final, después de la orden y de las unidades existentes. No se interbloquea: desactivar una marca bloquea solo la marca, y reactivarla lee después la empresa en modo compartido, que no choca con el de la factura. La revisión del 03/11/2026 lo halló; no hubo que cambiar código.

Ninguna transacción toma estos candados en otro orden: la anulación de una orden bloquea solo la orden y lee las facturas sin candado; la edición de una unidad bloquea la unidad y pregunta al puerto sin candado. Lo que no cubre un candado lo cubre un único parcial: dos facturas con el mismo VIN nuevo chocan en `vehicles_vin_uq`, y dos que vinculan la misma unidad, en `purchase_invoice_items_vehicle_current_uq` (§3.4).

### 2.5 Puertos y fachadas entre contextos

Toda fachada devuelve `Optional`, tipos simples o records de su propio paquete `acl`, nunca agregados (regla 1.4). Lo que `procurement` lee de otra fachada se traduce en `application/internal/outboundservices/acl` a records propios (`ForeignAclStaysInTranslationLayerTests`).

#### 2.5.1 `fleet` (dueño: `fleet`; consumidor: `procurement`)

```java
package ky.millennium.mobility.erp.fleet.interfaces.acl;

public interface FleetContextFacade {

    /**
     * Qué es cada VIN para una empresa: nuevo, una unidad existente de esa empresa o una de otra.
     * Eleva el alcance una sola vez, y de las otras empresas responde solo que el VIN es suyo.
     * Con lockExisting, exige una transacción abierta y bloquea las unidades existentes de la
     * empresa por id ascendente hasta su fin. Sin él, solo lee (la vista previa del formulario).
     */
    List<PurchasedUnitMatch> matchPurchasedUnits(Long companyId, List<String> vins, boolean lockExisting);

    /**
     * Da de alta las unidades nuevas de una factura y devuelve las existentes que vincula.
     * Propagation.MANDATORY. Valida todo antes de escribir; vincula solo unidades de la misma
     * empresa y no anuladas; las nuevas nacen en el estado inicial por el camino de alta, sin
     * ubicación, firmadas por la cuenta de la sesión (D-42, D-153, D-160). De una unidad que
     * vincula no escribe nada: el motor y el color de su línea se ignoran (D-179).
     */
    Result<List<PurchasedUnit>, ApplicationError> registerOrLinkPurchasedUnits(RegisterPurchasedUnitsRequest request);

    /** Las unidades de las líneas, en lote, para la ficha de la factura. */
    Map<Long, UnitSummary> fetchUnitsByIds(Set<Long> vehicleIds);
}

public enum PurchasedUnitMatchKind { NEW, EXISTING, OTHER_COMPANY }

public record PurchasedUnitMatch(String vin, PurchasedUnitMatchKind kind, @Nullable ExistingUnit unit) {}

public record ExistingUnit(Long id, UUID publicId, Long modelId, int modelYear, Long commercialBrandId,
        Long colorId, @Nullable String engineNumber, boolean annulled) {}

public record RegisterPurchasedUnitsRequest(Long companyId, List<PurchasedUnitLine> lines) {}

public record PurchasedUnitLine(int line, String vin, @Nullable String engineNumber, Long modelId,
        int modelYear, Long colorId, Long commercialBrandId) {}

public record PurchasedUnit(int line, Long vehicleId, UUID vehiclePublicId, boolean created) {}

public record UnitSummary(Long id, UUID publicId, Long companyId, String vin, @Nullable String engineNumber,
        @Nullable String plate, Long modelId, int modelYear, Long commercialBrandId, Long colorId,
        boolean annulled) {}
```

`registerOrLinkPurchasedUnits` responde, cada uno con `context.lines` (§4.3): `400 VALIDATION_ERROR` por un VIN mal formado o repetido; `409 VEHICLE_VIN_CONFLICT` por un VIN de otra empresa o uno que otra factura dio de alta mientras tanto; `422 VEHICLE_ANNULLED`; y, para las nuevas, los códigos que ya responde el alta a mano (`VEHICLEMODEL_NOT_FOUND`, `VEHICLEMODEL_INACTIVE`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE`, `COLOR_NOT_FOUND`, `COLOR_INACTIVE`, `COMMERCIALBRAND_NOT_FOUND`, `COMMERCIALBRAND_INACTIVE`, `VEHICLESTATUS_INITIAL_MISSING`, `STATUSCHANGEREASON_REGISTRATION_MISSING`). El motor y el color de una línea que vincula una unidad existente no se revisan ni se escriben: la unidad conserva los suyos, y se corrigen en Inventario con `stock.editar_vehiculo` (D-179). La carrera del VIN deja la transacción abortada: la fachada la marca para revertir, como hoy `discardWrites()` del alta. No pide `stock.crear_vehiculo`: el permiso de registrar la factura basta (D-153), y por eso es un comando aparte, nunca una bandera del alta a mano (A6).

**Los puertos que `fleet` declara y `procurement` implementa:**

```java
package ky.millennium.mobility.erp.fleet.interfaces.acl;

/** Lo que compras sabe de la compra de una unidad. fleet nunca depende de quien lo implementa. */
public interface UnitPurchases {
    /** Si la unidad está en una línea de factura vigente (D-160: fija su modelo y su año). */
    boolean hasCurrentLine(Long vehicleId);
    /** De las unidades dadas, las que están en una línea vigente (el indicador de una página). */
    Set<Long> withCurrentLine(Set<Long> vehicleIds);
    /** Las unidades con línea vigente de la empresa, o de todas las que alcanza la transacción (el filtro, D-175). */
    Set<Long> unitIdsWithCurrentLine(@Nullable Long companyId);
    /** Si el valor de la unidad es cero (§9.6): sin línea vigente; desde la fase 4, también por sus notas. */
    boolean valueIsZero(Long vehicleId);
    /** La versión de lo que responde, para la exportación del Inventario (D-128). */
    String fetchDataVersion();
}

/** Si una unidad tiene contratos (§9.6). Lo implementará ventas. */
public interface UnitContracts {
    boolean hasContracts(Long vehicleId);
}
```

`UnitPurchaseFacts` recibe `List<UnitPurchases>` y responde «alguno» en `hasCurrentLine`, la unión en los conjuntos y «todos» en `valueIsZero`; `UnitContractFacts` recibe `List<UnitContracts>` y responde «ninguno» con la lista vacía, que es lo que pasa mientras ventas no exista (plan 5.4). `PurchaseInvoiceUnitPurchasesAdapter` (`procurement/infrastructure/fleet`) implementa `UnitPurchases` con consultas sobre `purchase_invoice_items` (`status = 'VIGENTE'`), bajo el alcance de quien llama: la Row Level Security de la tabla deja ver solo las empresas que la transacción alcanza. Los puertos esperan el esquema del paso 5.5 (hallazgo M5).

#### 2.5.2 `organization` (dueño: `organization`; consumidores: `procurement` y `fleet`)

```java
/** La empresa que registra una factura, leída en modo compartido: activa, su país, su zona y su moneda funcional. */
Optional<PurchasingCompany> fetchPurchasingCompanyForShare(UUID companyPublicId);
/** Lo mismo, en lote y sin candado, para las vistas y la exportación. */
Map<Long, PurchasingCompany> fetchPurchasingCompaniesByIds(Set<Long> companyIds);
/** Las marcas de una empresa nombradas por su identificador público; una de otra empresa no aparece. */
Map<UUID, CommercialBrandReference> fetchCommercialBrandReferencesByPublicIds(Long companyId, Set<UUID> publicIds);
/** Las marcas de las unidades nuevas, en modo compartido (D-84), en lote: lo que hoy hace resolveScope por una. */
Map<Long, CommercialBrandReference> fetchCommercialBrandReferencesByIdsForShare(Set<Long> commercialBrandIds);

public record PurchasingCompany(Long id, UUID publicId, Long countryId, Long functionalCurrencyId,
        String timeZone, boolean isActive) {}
```

Los dos primeros los usa `procurement`; el tercero, `procurement`; el cuarto, `fleet` en `PurchasedUnitRegistrations`. `CommercialBrandReference` ya existe (`id`, `publicId`, `companyId`, `name`, `isActive`).

#### 2.5.3 `vehiclecatalog` (dueño: `vehiclecatalog`)

```java
Map<UUID, VehicleModelSummary> fetchVehicleModelSummariesByPublicIds(Set<UUID> publicIds); // procurement
Map<UUID, ColorSummary> fetchColorSummariesByPublicIds(Set<UUID> publicIds);               // procurement
void markModelsInUse(Set<Long> modelIds);                                                   // fleet, D-107
```

`fleet` sigue leyendo la vigencia de modelos y colores con `fetchVehicleModelSummariesByIds` y `fetchColorSummariesByIds`, que ya existen.

**Desde el 04/11/2026 (segunda revisión, contrato C10;** backend `6eee6878`**).** La plantilla de las líneas y su importación (§11) leen el catálogo por dos métodos más:

```java
List<VehicleModelSummary> fetchActiveVehicleModelSummaries(); // procurement: los modelos que una unidad nueva puede tomar (D-108)
List<ColorSummary> fetchActiveColorSummaries();               // procurement: los colores activos, por nombre
```

El primero da cada modelo activo de una marca activa y de un tipo de vehículo activo, por el nombre de su marca y el suyo; el segundo, cada color activo. Cada uno lee en una consulta por clase de dato, sobre los servicios de consulta del catálogo.

#### 2.5.4 `location` y los puertos internos de `procurement`

```java
// location.interfaces.acl
/** La tasa del país, el par y el tipo, la más reciente de la fecha o anterior (D-151). */
Optional<EffectiveRate> fetchEffectiveRate(Long countryId, Long fromCurrencyId, Long toCurrencyId,
        String rateType, LocalDate onOrBefore);

public record EffectiveRate(BigDecimal rate, LocalDate rateDate, String rateType, String source) {}

public record CountrySummary(Long id, String isoAlpha2, String name, String defaultTimezone,
        @Nullable String taxName, @Nullable BigDecimal taxRate, String purchaseExchangeRateType) {}
```

`fetchEffectiveRate` lee `exchange_rates` con `rate_date <= :onOrBefore ORDER BY rate_date DESC LIMIT 1`, por el índice único de V91, cuyo último campo es el día. No toma el candado de la serie: la factura copia la tasa que lee, y una corrección posterior no la cambia (diccionario §4.8). Las pruebas que construyen `CountrySummary` se ajustan en el mismo commit. **Como se construyó:** la fachada lee la tasa por `EffectiveExchangeRateQueryService`, un servicio de consulta aparte del de la lista de tasas. La lista nombra a quien corrigió cada tasa con las cuentas de `iam`; la tasa que copia un documento no las necesita, así que la fachada no depende de ellas y no se forma un ciclo de beans.

`procurement` implementa `CurrencyUsage` (`location.interfaces.acl`) con `PurchaseInvoiceCurrencyUsageAdapter`: una factura de cualquier estado en una moneda fija sus decimales (diccionario §4.1, PRC-R26).

**`PurchaseInvoiceDependents`** (`procurement/application/internal/outboundservices/dependents`, hallazgo H3):

```java
/** Lo que depende de una factura y le impide anularse: notas de crédito y pagos aplicados (D-161). */
public interface PurchaseInvoiceDependents {
    boolean anyFor(Long purchaseInvoiceId);
}
```

`PurchaseInvoiceAnnulments` recibe `List<PurchaseInvoiceDependents>` y responde «ninguno» con la lista vacía. Las fases 4 y 5 agregan sus beans. Una prueba fija que hoy la lista está vacía, para que el primer bean obligue a revisarla. **Al día con las fases 4 y 5:** la lista tiene los dos beans, `CreditNoteInvoiceDependentsAdapter` y `PaymentInvoiceDependentsAdapter`, y `PurchaseInvoiceDependentsTests.creditNotesAndPaymentsAreTheDependents` fija que son exactamente esos dos; un tercero obliga a revisarla. La prueba de la lista vacía se retiró con la fase 4 (backend `cf667307`).

#### 2.5.5 `documents` (dueño: `documents`)

```java
/**
 * Guarda el archivo y dice si esta escritura puso la llave primero: su fila no existía y se insertó.
 * Solo entonces quien llama puede registrar el descarte para una reversión (desde el 04/11/2026).
 */
boolean put(Long companyId, String storageKey, byte[] bytes, String mediaType);

/**
 * Descarta el objeto de una llave escrita por una transacción que se revirtió (plan E15).
 * En Spaces borra el objeto; en la base no hace nada, porque la reversión ya quitó la fila.
 * Solo para una llave que ninguna transacción confirmada pudo apuntar: la que put escribió primero.
 */
void discardRolledBack(String storageKey);
```

`PurchaseInvoiceFiles` registra un `TransactionSynchronization` que llama a `discardRolledBack` en `afterCompletion(STATUS_ROLLED_BACK)` por cada llave que su `put` escribió primero.

**Como se corrigió (04/11/2026)** (backend `7ad20a4a`, hallazgo B10). La versión 3 decía: «Es seguro sin leer `kept_files`: la llave lleva el `public_id` de una factura que nunca se confirmó». Valía para el alta y era falso para un `PUT`. La llave de un `PUT` lleva el `public_id` de una factura confirmada, así que los mismos bytes enviados otra vez, o un archivo anterior enviado de vuelta, dan una llave que la factura ya apunta. Si ese `PUT` se revertía, el descarte borraba de Spaces el objeto de la factura confirmada, que quedaba sin su archivo. En la base no pasaba, porque ahí la reversión solo quita la fila que la transacción escribió.

Ahora `KeptFileStore.put` devuelve si escribió la llave primero, y lo sabe de forma atómica por la misma sentencia que guarda la fila: `INSERT … ON CONFLICT … DO UPDATE … RETURNING (xmax = 0)`, que es verdadero solo en una fila insertada. `PurchaseInvoiceFiles` registra el descarte solo entonces. Lo mismo hacen `CreditNoteFiles` (las notas) y `VehicleDocumentFiles` (los documentos de la habilitación), que ya no descarga el objeto para decidirlo.

Queda un caso, el mismo que documenta la [especificación de la habilitación](especificacion-habilitacion-2026-11-02.md) para sus documentos: dos `PUT` de la misma factura con los mismos bytes, a la vez, cuando uno se revierte y el otro confirma. Los dos esperan el candado de la factura, pero el descarte del que se revirtió corre después de soltarlo, así que puede borrar de Spaces el objeto que el otro acaba de volver a escribir con la misma llave. En la base no pasa: ahí la reversión ya quitó su fila.

~~§2.7 del diccionario todavía dice la frase de la versión 3 (documentación `caaada3`). Corregirla es una propuesta para el responsable, que es quien cambia el diccionario.~~ El responsable aprobó la corrección el 04/11/2026: desde la v1.39 (documentación `7cc29f4`), §2.7 registra el descarte solo de una llave que la transacción escribió primero, lo que dice `KeptFileStore.put`, y nunca el de una llave que un registro confirmado ya apunta. Un archivo de la plantilla todavía puede dejar un objeto sin referencia: ella no registra el descarte.

Pruebas: `PurchaseInvoiceFilesTests.aRolledBackRegistrationDiscardsItsObject`, `theSameBytesSentAgainAreNeverDiscarded`, `aFormerFileSentBackIsNeverDiscarded`, `aCommittedRegistrationKeepsItsObject`; `KeptFileStoreUpsertTests.theDatabaseStoreTellsWhetherAPutWroteTheKeyFirst`, `theSpacesStoreTellsWhetherAPutWroteTheKeyFirst`; `SpacesKeptFileStoreTests.aPutTellsWhetherItsUpsertInsertedTheRow`, `theObjectOfARolledBackKeyIsDeleted`.

**Como se corrigió (04/11/2026, segunda revisión): primero la fila, después el objeto** (backend `5df515b8`). En Spaces, `SpacesKeptFileStore.put` escribía el objeto antes de guardar su fila de `kept_files`. Si la fila fallaba, el `put` no llegaba a decir que había escrito la llave, nadie registraba el descarte y el objeto quedaba en el bucket sin ninguna fila que lo nombrara. Ahora guarda primero la fila (`INSERT … ON CONFLICT … RETURNING (xmax = 0)`) y después el objeto: si `putObject` falla, el `put` falla y la fila se revierte con la transacción; si quien llama se revierte después, el descarte que registró borra el objeto. Vale para las facturas, las notas y los documentos de una unidad, que comparten `KeptFileStore`. Pruebas: `SpacesKeptFileStoreTests.aRowThatFailsToWriteLeavesNoObject`, `theObjectIsWrittenAfterItsRowAndItsFailureFailsThePut`; y, de la misma revisión, `KeptFileStoreUpsertTests.aKeyAnEarlierTransactionCommittedIsNotWrittenFirstAgain` (backend `5f967ae3`), que confirma el primer `put` en una transacción propia y comprueba que uno posterior de la misma llave responde que no la escribió primero, en los dos almacenes.

**La plantilla de las líneas** (§11) también la genera `documents`, como un documento generado (D-106): `ProcurementDocumentKinds.PURCHASE_INVOICE_LINE_TEMPLATE` (backend `23fb6bba`). No pasa por `KeptFileStore`.

---

## 3. Esquema

### 3.1 Numeración

Las fases 2 y 3 pueden integrarse en cualquier orden, así que los números se asignan al integrar en `develop`, después de V91 (plan §4). Aquí van relativos: V(n), V(n+1), V(n+2), en este orden dentro de la fase. Cada migración empieza con `SELECT set_config('app.access_scope', 'SYSTEM', true);` y `SET LOCAL lock_timeout = '5s';`, como V90 y V91.

| Migración | Contexto | Paso | Contenido |
|---|---|---|---|
| V(n) `seed_iam_permissions_of_purchase_invoices` | `iam` | 5.1 | Cuatro permisos y sus marcas |
| V(n+1) `add_country_purchase_exchange_rate_type` | `location` | 5.2 | La columna del país |
| V(n+2) `create_procurement_purchase_invoices_schema` | `procurement` | 5.5 | Las dos tablas, sus restricciones, disparadores, RLS, permisos de la cuenta y `data_versions` |

### 3.2 V(n): los permisos (D-176)

```sql
INSERT INTO permissions (code, module, description, category)
VALUES ('compras.registrar_factura', 'compras', 'Registrar una factura de compra con sus líneas', 'crear'),
       ('compras.registrar_factura_antigua', 'compras', 'Una factura de más de veinte días', 'acciones_especiales'),
       ('compras.anular_factura', 'compras', 'Anular un comprobante completo', 'desactivar_anular'),
       ('stock.anular_vehiculo', 'stock', 'Anular una unidad por acción explícita, con el candado de §9.6', 'desactivar_anular')
ON CONFLICT (code) DO NOTHING;

INSERT INTO role_permissions (role_id, permission_id, granted_by)
SELECT role.id, permission.id, system_account.id
FROM (VALUES ('SUPERADMIN', 'compras.registrar_factura'),
             ('SUPERADMIN', 'compras.registrar_factura_antigua'),
             ('SUPERADMIN', 'compras.anular_factura'),
             ('SUPERADMIN', 'stock.anular_vehiculo'),
             ('ADMIN_EMPRESA', 'stock.anular_vehiculo')) AS matrix (role_code, permission_code)
JOIN roles role ON role.code = matrix.role_code AND role.is_system AND role.company_id IS NULL
JOIN permissions permission ON permission.code = matrix.permission_code
CROSS JOIN (SELECT id FROM users WHERE email = 'system@millennium.ky') AS system_account
ON CONFLICT DO NOTHING;
```

- Las descripciones son las de la tabla del diccionario §5.6 sin la referencia entre paréntesis, como V79.
- Los tres de facturas van solo al superadministrador (D-176). La migración de la fase 4 marca los de AE, OP y CO de la matriz.
- `stock.anular_vehiculo` no es un permiso de facturas, así que sigue la matriz desde ya: SA y AE.

### 3.3 V(n+1): el tipo de tasa de compra del país (D-152)

```sql
ALTER TABLE countries
    ADD COLUMN purchase_exchange_rate_type VARCHAR(10) NOT NULL DEFAULT 'oficial',
    ADD CONSTRAINT countries_purchase_exchange_rate_type_ck
        CHECK (purchase_exchange_rate_type IN ('compra', 'venta', 'oficial'));

UPDATE countries SET purchase_exchange_rate_type = 'venta' WHERE iso_alpha_2 = 'PE';
```

Perú propone `venta`; todo otro país, también los que se creen después, `oficial`, la tasa única que publican la TRM y el FIX (diccionario §4.8, D-181). Los tres valores son los que ya tiene `ExchangeRateType` y el `CHECK` de V91. La API del país lo devuelve de solo lectura hasta que haga falta editarlo (D-181). `countries` ya está versionada en `data_versions`.

### 3.4 V(n+2): las tablas de compras

```sql
ALTER TABLE purchase_orders
    ADD CONSTRAINT purchase_orders_id_supplier_company_uq UNIQUE (id, supplier_id, company_id);

CREATE TABLE purchase_invoices
(
    id                  BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id           UUID           NOT NULL DEFAULT uuidv7(),
    company_id          BIGINT         NOT NULL REFERENCES companies (id),
    supplier_id         BIGINT         NOT NULL,
    purchase_order_id   BIGINT,
    document_type       VARCHAR(20)    NOT NULL,
    document_number     VARCHAR(30)    NOT NULL,
    invoice_date        DATE           NOT NULL,
    currency_id         BIGINT         NOT NULL REFERENCES currencies (id),
    exchange_rate_date  DATE           NOT NULL,
    exchange_rate       NUMERIC(12, 6) NOT NULL,
    tax_rate            NUMERIC(5, 4)  NOT NULL,
    gross_total         NUMERIC(14, 2) NOT NULL,
    has_isc_column      BOOLEAN        NOT NULL DEFAULT false,
    file_path           TEXT,
    xml_path            TEXT,
    annulled_at         TIMESTAMPTZ,
    annulled_by         BIGINT         REFERENCES users (id),
    annulment_reason    VARCHAR(30),
    created_at          TIMESTAMPTZ    NOT NULL DEFAULT now(),
    created_by          BIGINT         REFERENCES users (id),
    updated_at          TIMESTAMPTZ    NOT NULL DEFAULT now(),   -- D-178
    updated_by          BIGINT         REFERENCES users (id),    -- D-178

    CONSTRAINT purchase_invoices_public_id_uq UNIQUE (public_id),
    CONSTRAINT purchase_invoices_id_company_uq UNIQUE (id, company_id),
    CONSTRAINT purchase_invoices_supplier_company_fk
        FOREIGN KEY (supplier_id, company_id) REFERENCES suppliers (id, company_id),
    CONSTRAINT purchase_invoices_order_supplier_company_fk
        FOREIGN KEY (purchase_order_id, supplier_id, company_id)
        REFERENCES purchase_orders (id, supplier_id, company_id),
    CONSTRAINT purchase_invoices_document_type_ck CHECK (document_type IN ('factura', 'boleta', 'otro')),
    CONSTRAINT purchase_invoices_document_number_ck
        CHECK (length(document_number) > 0 AND document_number = upper(document_number) AND document_number !~ '\s'),
    CONSTRAINT purchase_invoices_exchange_rate_ck CHECK (exchange_rate > 0),
    CONSTRAINT purchase_invoices_exchange_rate_date_ck CHECK (exchange_rate_date <= invoice_date),
    CONSTRAINT purchase_invoices_tax_rate_ck CHECK (tax_rate >= 0 AND tax_rate < 1),
    CONSTRAINT purchase_invoices_gross_total_ck CHECK (gross_total > 0),
    CONSTRAINT purchase_invoices_annulment_reason_ck
        CHECK (annulment_reason IS NULL OR annulment_reason IN ('ERROR_REGISTRO', 'ANULADA_POR_EMISOR')),
    CONSTRAINT purchase_invoices_annulment_ck
        CHECK ((annulled_at IS NULL AND annulled_by IS NULL AND annulment_reason IS NULL)
            OR (annulled_at IS NOT NULL AND annulled_by IS NOT NULL AND annulment_reason IS NOT NULL))
);

-- El número no se repite en el proveedor y el tipo entre las no anuladas (D-160)
CREATE UNIQUE INDEX purchase_invoices_number_uq
    ON purchase_invoices (company_id, supplier_id, document_type, document_number) WHERE annulled_at IS NULL;
CREATE INDEX idx_purchase_invoices_company_date ON purchase_invoices (company_id, invoice_date DESC, id DESC);
CREATE INDEX idx_purchase_invoices_supplier ON purchase_invoices (company_id, supplier_id);
CREATE INDEX idx_purchase_invoices_order ON purchase_invoices (purchase_order_id) WHERE annulled_at IS NULL;
CREATE INDEX idx_purchase_invoices_currency ON purchase_invoices (currency_id);

CREATE TABLE purchase_invoice_items
(
    id                   BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id            UUID           NOT NULL DEFAULT uuidv7(),
    purchase_invoice_id  BIGINT         NOT NULL,
    company_id           BIGINT         NOT NULL REFERENCES companies (id),
    vehicle_id           BIGINT,
    concept              VARCHAR(150),
    model_id             BIGINT         REFERENCES vehicle_models (id),
    model_year           SMALLINT,
    commercial_brand_id  BIGINT,
    invoiced_amount      NUMERIC(14, 2) NOT NULL,
    isc_amount           NUMERIC(14, 2),
    status               VARCHAR(20)    NOT NULL DEFAULT 'VIGENTE',
    annulled_at          TIMESTAMPTZ,

    CONSTRAINT purchase_invoice_items_public_id_uq UNIQUE (public_id),
    CONSTRAINT purchase_invoice_items_invoice_company_fk
        FOREIGN KEY (purchase_invoice_id, company_id) REFERENCES purchase_invoices (id, company_id),
    CONSTRAINT purchase_invoice_items_vehicle_company_fk
        FOREIGN KEY (vehicle_id, company_id) REFERENCES vehicles (id, company_id),
    CONSTRAINT purchase_invoice_items_brand_company_fk
        FOREIGN KEY (commercial_brand_id, company_id) REFERENCES commercial_brands (id, company_id),
    -- Una línea de vehículo lleva unidad, modelo, año y marca; una de concepto, solo su texto (D-22, D-160)
    CONSTRAINT purchase_invoice_items_vehicle_or_concept_ck
        CHECK ((vehicle_id IS NOT NULL AND model_id IS NOT NULL AND model_year IS NOT NULL
                    AND commercial_brand_id IS NOT NULL AND concept IS NULL)
            OR (vehicle_id IS NULL AND model_id IS NULL AND model_year IS NULL
                    AND commercial_brand_id IS NULL AND concept IS NOT NULL)),
    CONSTRAINT purchase_invoice_items_model_year_ck CHECK (model_year IS NULL OR model_year >= 2017),
    CONSTRAINT purchase_invoice_items_invoiced_amount_ck CHECK (invoiced_amount > 0),  -- D-179
    CONSTRAINT purchase_invoice_items_isc_amount_ck
        CHECK (isc_amount IS NULL OR (isc_amount >= 0 AND isc_amount <= invoiced_amount)),
    CONSTRAINT purchase_invoice_items_status_ck CHECK (status IN ('VIGENTE', 'ANULADA')),
    CONSTRAINT purchase_invoice_items_annulment_ck CHECK ((status = 'ANULADA') = (annulled_at IS NOT NULL))
);

-- Una unidad, una línea vigente (D-160); y una sola vez en el mismo comprobante (§9.4)
CREATE UNIQUE INDEX purchase_invoice_items_vehicle_current_uq
    ON purchase_invoice_items (vehicle_id) WHERE status = 'VIGENTE' AND vehicle_id IS NOT NULL;
CREATE UNIQUE INDEX purchase_invoice_items_invoice_vehicle_uq
    ON purchase_invoice_items (purchase_invoice_id, vehicle_id) WHERE vehicle_id IS NOT NULL;
CREATE INDEX idx_purchase_invoice_items_invoice ON purchase_invoice_items (purchase_invoice_id, id);
CREATE INDEX idx_purchase_invoice_items_company_current
    ON purchase_invoice_items (company_id, vehicle_id) WHERE status = 'VIGENTE';
```

**Lo que la base no decide y queda en el servicio:** la orden enviada (estado), la moneda funcional con tasa 1 (vive en `companies`), el formato del número en Perú (vive en `countries`), la fecha futura y los 20 días (dependen del momento), las 100 líneas y que la primera versión registre solo líneas de vehículo (D-173): la tabla ya admite conceptos, para que habilitarlos no pida una migración.

**Los disparadores del ISC**, como V69 y V70 con la columna de descuento:

```sql
CREATE FUNCTION enforce_purchase_invoice_item_isc() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
DECLARE
    includes_isc BOOLEAN;
BEGIN
    SELECT has_isc_column INTO includes_isc FROM purchase_invoices WHERE id = NEW.purchase_invoice_id;
    IF includes_isc IS NULL THEN
        RAISE EXCEPTION 'purchase invoice % not found for its line', NEW.purchase_invoice_id;
    END IF;
    IF NEW.vehicle_id IS NOT NULL AND includes_isc AND NEW.isc_amount IS NULL THEN
        RAISE EXCEPTION 'A vehicle line of an invoice with the ISC column requires isc_amount';
    END IF;
    IF (NOT includes_isc OR NEW.vehicle_id IS NULL) AND NEW.isc_amount IS NOT NULL THEN
        RAISE EXCEPTION 'A line of an invoice without the ISC column, or a concept line, forbids isc_amount';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER purchase_invoice_items_isc_tg
    BEFORE INSERT OR UPDATE ON purchase_invoice_items
    FOR EACH ROW EXECUTE FUNCTION enforce_purchase_invoice_item_isc();

CREATE FUNCTION enforce_purchase_invoice_isc_column() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    IF NEW.has_isc_column IS DISTINCT FROM OLD.has_isc_column THEN
        IF NEW.has_isc_column AND EXISTS (SELECT 1 FROM purchase_invoice_items
                WHERE purchase_invoice_id = NEW.id AND vehicle_id IS NOT NULL AND isc_amount IS NULL) THEN
            RAISE EXCEPTION 'Turning on the ISC column requires isc_amount on every vehicle line';
        END IF;
        IF NOT NEW.has_isc_column AND EXISTS (SELECT 1 FROM purchase_invoice_items
                WHERE purchase_invoice_id = NEW.id AND isc_amount IS NOT NULL) THEN
            RAISE EXCEPTION 'Turning off the ISC column forbids isc_amount on every line';
        END IF;
    END IF;
    RETURN NEW;
END
$$;

CREATE CONSTRAINT TRIGGER purchase_invoices_isc_tg
    AFTER UPDATE OF has_isc_column ON purchase_invoices
    DEFERRABLE INITIALLY DEFERRED
    FOR EACH ROW EXECUTE FUNCTION enforce_purchase_invoice_isc_column();
```

El de la cabecera no se alcanza hoy, porque el disparador de abajo no deja cambiar `has_isc_column`. Se conserva igual que en la orden, para que una migración futura que lo cambie quede vigilada. Las pruebas son `PurchaseInvoiceIscTriggerTests` y `PurchaseInvoiceFiscalTriggerTests`: un `UPDATE` directo de `has_isc_column` lo rechaza antes el disparador fiscal, así que ninguna prueba alcanza el de la cabecera (`PurchaseInvoiceIscTriggerTests.theIscOfALineIsNotTakenAwayNorTheColumnTurnedOff`).

**Lo fiscal no se edita, también en la base** (D-161, D-172):

```sql
CREATE FUNCTION keep_purchase_invoice_fiscal_data() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    IF OLD.annulled_at IS NOT NULL THEN
        RAISE EXCEPTION 'An annulled purchase invoice never changes';
    END IF;
    IF (NEW.company_id, NEW.supplier_id, NEW.document_type, NEW.document_number, NEW.invoice_date,
        NEW.currency_id, NEW.exchange_rate_date, NEW.exchange_rate, NEW.tax_rate, NEW.gross_total,
        NEW.has_isc_column, NEW.created_at, NEW.created_by)
       IS DISTINCT FROM
       (OLD.company_id, OLD.supplier_id, OLD.document_type, OLD.document_number, OLD.invoice_date,
        OLD.currency_id, OLD.exchange_rate_date, OLD.exchange_rate, OLD.tax_rate, OLD.gross_total,
        OLD.has_isc_column, OLD.created_at, OLD.created_by) THEN
        RAISE EXCEPTION 'The fiscal data of a purchase invoice are not edited: annul it and register another';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER purchase_invoices_fiscal_tg
    BEFORE UPDATE ON purchase_invoices FOR EACH ROW EXECUTE FUNCTION keep_purchase_invoice_fiscal_data();

CREATE FUNCTION keep_purchase_invoice_item_fiscal_data() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    IF (NEW.purchase_invoice_id, NEW.company_id, NEW.vehicle_id, NEW.concept, NEW.model_id, NEW.model_year,
        NEW.commercial_brand_id, NEW.invoiced_amount, NEW.isc_amount)
       IS DISTINCT FROM
       (OLD.purchase_invoice_id, OLD.company_id, OLD.vehicle_id, OLD.concept, OLD.model_id, OLD.model_year,
        OLD.commercial_brand_id, OLD.invoiced_amount, OLD.isc_amount) THEN
        RAISE EXCEPTION 'A line of a purchase invoice is never edited';
    END IF;
    IF OLD.status = 'ANULADA' AND (NEW.status, NEW.annulled_at) IS DISTINCT FROM (OLD.status, OLD.annulled_at) THEN
        RAISE EXCEPTION 'An annulled line of a purchase invoice never changes';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER purchase_invoice_items_fiscal_tg
    BEFORE UPDATE ON purchase_invoice_items FOR EACH ROW EXECUTE FUNCTION keep_purchase_invoice_item_fiscal_data();
```

En la cabecera solo cambian `purchase_order_id`, `file_path`, `xml_path`, los datos de la anulación y las columnas de revisión; en la línea, solo el paso de `VIGENTE` a `ANULADA` con su momento.

**Row Level Security, como V83:**

```sql
ALTER TABLE purchase_invoices ENABLE ROW LEVEL SECURITY;
ALTER TABLE purchase_invoices FORCE ROW LEVEL SECURITY;
CREATE POLICY purchase_invoices_select ON purchase_invoices FOR SELECT USING (access_scope_reaches(company_id));
CREATE POLICY purchase_invoices_insert ON purchase_invoices FOR INSERT WITH CHECK (access_scope_reaches(company_id));
CREATE POLICY purchase_invoices_update ON purchase_invoices FOR UPDATE
    USING (access_scope_reaches(company_id)) WITH CHECK (access_scope_reaches(company_id));
-- Las mismas tres en purchase_invoice_items: purchase_invoice_items_select, _insert y _update
```

Ninguna política de borrado. **Permisos de la cuenta de la aplicación**, con el bloque `DO` de V69 sobre `${app_role}`: `REVOKE ALL` y `GRANT SELECT, INSERT, UPDATE` en las dos tablas, sin `DELETE`.

**`data_versions`**: las dos tablas muestran la exportación, así que siembran sus 64 casillas (`generate_series(0, 63)`) y crean sus dos disparadores por sentencia, `purchase_invoices_data_version_insert_tg`, `…_update_tg`, `purchase_invoice_items_data_version_insert_tg` y `…_update_tg`, como V69.

---

## 4. Contrato de la API

### 4.1 Rutas

| # | Método y ruta | Permiso de la ruta | Respuesta | Paso |
|---|---|---|---|---|
| 1 | `GET /purchase-invoices` | `compras.ver` | `200` página o arreglo | 5.8 |
| 2 | `GET /purchase-invoices` con el `Accept` de Excel | `compras.ver` | `303` a `/documents/{id}` | 5.8 |
| 3 | `GET /purchase-invoices/{id}` | `compras.ver` | `200` ficha | 5.8 |
| 4 | `POST /purchase-invoices` (multipart) | `compras.registrar_factura` | `201` con `Location` | 5.6 |
| 5 | `PUT /purchase-invoices/{id}` (multipart) | `compras.registrar_factura` | `200` ficha | 5.7 |
| 6 | `POST /purchase-invoices/{id}/annulment` | `compras.anular_factura` | `200` ficha | 5.7 |
| 7 | `GET /purchase-invoices/{id}/file` | `compras.ver` | `200` bytes | 5.8 |
| 8 | `GET /purchase-invoices/{id}/xml` | `compras.ver` | `200` bytes | 5.8 |
| 9 | `GET /purchase-invoice-vin-checks` ~~**(propuesta)**~~ | `compras.registrar_factura` | `200` arreglo | 5.6 |
| 10 | `POST /vehicles/{id}/annulment` | `stock.anular_vehiculo` | `200` unidad | 5.4 |
| 11 | `GET /purchase-invoices/line-template` con el `Accept` de Excel (C10, 04/11/2026) | `compras.registrar_factura` | `303` a `/documents/{id}` | §11.1 |
| 12 | `POST /purchase-invoices/line-import` (multipart, C10, 04/11/2026) | `compras.registrar_factura` | `200` líneas y errores de las filas; no guarda nada | §11.2 |
| 13 | `GET /purchase-invoices/{id}/payment-position` (los pagos, 03/11/2026) | `pagos.ver` | `200` | [Especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md) §4.2.8 |

**Como se construyó: la ruta 9 y la 13** (auditoría del 09/11/2026). La ruta 9 se construyó con este nombre el 03/11/2026 (backend `a37962bf`, en `PurchaseInvoiceVinChecksController`, con `compras.ver` en la clase y `compras.registrar_factura` en el método), y su marca **(propuesta)** queda tachada. La 13 no es de esta fase: la sumaron los pagos (backend `56f992c8`) en `PurchaseInvoicesController`, donde `pagos.ver` en el método reemplaza el `compras.ver` de la clase; esta tabla no la tenía.

**Como cambió (09/11/2026, D-244).** Ninguna ruta nueva. Las rutas 1 y 2 suman el filtro `paymentStatus`; la 1 y la 3, en cada factura, `paymentStatus`, `appliedTotal` y `owed`; y las filas de la 1 traen sus `allowedActions` (§4.2.1, §4.2.3, §4.5). La 2 suma la columna «Estado de pago» (§4.2.2), y la 13 nombra la cuenta de cada aplicación, `bankAccountName` ([especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md) §4.2.8).

**Rutas que ya existen y cambian:** `GET /vehicles` (filtro `hasCurrentInvoice`, campo `hasCurrentInvoice`, columna de la exportación); `PUT /vehicles/{id}` (el año queda fijo con una línea vigente); `GET /purchase-orders` (filtro `supplierId`); `GET /purchase-orders/{id}` (campo `invoicing`); `POST /purchase-orders/{id}/annulment` (rechaza la orden con facturas vigentes); `GET /countries` y `/countries/{id}` (campo `purchaseExchangeRateType`).

`PurchaseInvoicesController` lleva `@PermissionRequired(PermissionCodes.Procurement.VIEW)` en la clase y el suyo en cada escritura, que reemplaza al de la clase. Las rutas 11 y 12 van en su propio controlador, `PurchaseInvoiceLinesController`, con `compras.registrar_factura` en la clase (backend `d3b1e6ae`, `23fb6bba`). El servicio exige además leer con `compras.ver` donde escribe (D-134): escribir sobre una factura que no se lee responde `404`. No hay `DELETE` (`NoDeleteEndpointsTests`).

### 4.2 Cada ruta

Los montos viajan como `{ "amount", "currency" }` (convenciones §5.2): el pedido acepta `amount` como texto decimal o número, y la respuesta lo escribe como `BigDecimal` con los decimales de la moneda, igual que la orden. `exchangeRate` y `taxRate` siguen la misma regla, como hoy `rate` de la tasa (convenciones §13, fila 7). Las fechas, `AAAA-MM-DD`; los momentos, ISO en UTC.

#### 4.2.1 `GET /purchase-invoices`

| `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|
| El número normalizado, la razón social y el documento tributario del proveedor, sin distinguir mayúsculas ni tildes (`fold_search_text`, D-142) | `companyId`; `supplierId`; `purchaseOrderId`; `documentType` (`factura`, `boleta`, `otro`, en cualquier mayúscula); `status` (`VIGENTE` o `ANULADA`, derivado de `annulled_at`); `currency` (código ISO); `from` y `to`, el período `[from, to)` sobre `invoiceDate`; y desde el 09/11/2026, `paymentStatus` (`SIN_PAGO`, `PARCIAL`, `PAGADA`, `SALDADA` o `ANULADA_POR_NOTAS`, en cualquier mayúscula; D-244). Un valor fuera de lista o una fecha mal escrita, `400 VALIDATION_ERROR`; un `companyId` fuera del alcance, `404 COMPANY_NOT_FOUND` | `invoiceDate`, `documentNumber`, `supplierLegalName`, `grossTotal`, `createdAt` | `invoiceDate` desc; los empates, `id` desc |

Lista solo lo que quien llama lee con `compras.ver` (`companyReach`). **Como se construyó:** cada fila es la ficha entera (§4.2.3), con `items: null` y ~~`allowedActions: null`~~, desde el 09/11/2026, sus `allowedActions` (§4.5), y no el resumen que proponía esta sección (`{id, companyId, …, unitCount, …, allowedActions: null}`): la lista y la ficha comparten un solo recurso, y los dos traen `unitCount`, las líneas de la factura. `supplierLegalName` ordena por la razón social del proveedor, y en todo orden los empates van por `id` descendente. Las referencias se resuelven en lote.

**Como se corrigió (04/11/2026)** (backend `53f11cd3`, C7 de la revisión). `q` buscaba el texto tal como se escribió, así que `F001-00000123` no encontraba `F001-123`, que es como el alta guarda el número. Ahora `q` encuentra también el texto normalizado como número (`InvoiceNumber.searchForm`: mayúsculas, sin espacios, guiones de Unicode llevados a `-` y sin ceros a la izquierda del correlativo), además del número, la razón social y el documento tributario. Un texto que no puede ser número busca solo como antes. Los pagos ya buscaban así su número de operación, y las notas de crédito buscan igual desde el mismo commit. Prueba: `PurchaseInvoiceReadsIntegrationTests.theFiltersNarrowTheListAndAValueOutOfItsListIsRefused` (`F001-00000003` y `f001 – 0002`).

**Como cambió (09/11/2026, D-244)** (backend `b250636d`, `eb80ced8` y `6230346b`). Cada fila trae su estado de pago, `paymentStatus`, a quien la lee con `compras.ver`, nulo en una factura anulada; lo aplicado, `appliedTotal`, y lo que se debe, `owed`, solo a quien tiene `pagos.ver` en su empresa, nulos para cualquier otro y también en una factura anulada; y sus `allowedActions`, con la regla de la ficha (§4.5). Con `paymentStatus`, la lista lee todas las filas de los demás filtros, deja las que tienen ese estado, nunca una anulada, y corta la página en memoria, con `totalItems` contando solo esas, como «Por pagar» y los reportes (D-236). Un valor fuera de los cinco responde `400 VALIDATION_ERROR` en `paymentStatus`. Las filas de una página leen una sola vez lo que su pago y sus acciones necesitan —sus dependientes, por `PurchaseInvoiceDependents.invoicesWithAny(Set<Long>)`, y la zona de cada empresa, por `ExternalOrganizationService.fetchCompanyTimeZones`—, así que la página prepara las mismas sentencias con cualquier número de filas (§7.2). Pruebas: `PurchaseInvoiceReadsIntegrationTests.eachRowTellsItsStatusOfPaymentAndOnlyWhoReadsThePaymentsSeesTheAmounts`, `aDealerInvoiceThatOwesOnlyTheInitialsOfItsSettledUnitsIsSaldada`, `theRowsOfTheListOfferWhatTheirRecordOffers` y `theListIsFilteredByTheStatusOfPaymentAndItsPageIsCutInMemory`; `InvoicePaymentStatusPolicyTests`, con los cinco estados, la factura pagada de más y `theStatusIsReadInAnyCaseAndOnlyOneOfTheFive`.

#### 4.2.2 Exportación

El mismo `GET` con `Accept: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/json;q=0.9`, con los mismos `q`, filtros y `sort`, sin página (convenciones §8.1). Tipo de documento `purchase-invoices`, nombre base `facturas-de-compra`, alcance `DocumentScopes.grantsOf(scope, compras.ver)`, versión `purchase_invoices` + `purchase_invoice_items` + `purchase_orders` + `suppliers` + la de `organization` + la de `location`. Columnas, en este orden (claves `export.purchase-invoices.*`):

| # | Encabezado | Valor |
|---|---|---|
| 1 | Empresa | Nombre comercial |
| 2 | Documento del proveedor | Su número, con la sigla de su tipo: `RUC 20100128056` |
| 3 | Proveedor | Razón social |
| 4 | Tipo de comprobante | Traducido: Factura, Boleta, Otro |
| 5 | Número | Normalizado |
| 6 | Fecha del comprobante | Celda de fecha |
| 7 | Moneda | Código ISO |
| 8 | Tipo de cambio | Número, seis decimales |
| 9 | Día del tipo de cambio | Celda de fecha |
| 10 | Tasa de impuesto | Número, cuatro decimales |
| 11 | Total | Número, decimales de la moneda |
| 12 | Base sin IGV (calculado) | `calculatedBase()` (§2.2), número: en una factura, el total ÷ (1 + tasa de impuesto); en una boleta u otro, el total (D-180) |
| 13 | IGV (calculado) | `calculatedTax()`, número: en una factura, el total menos la base; en una boleta u otro, 0 (D-180) |
| 14 | Moneda de la empresa | Código ISO de la moneda funcional |
| 15 | Total en la moneda de la empresa (calculado) | Total × tipo de cambio, redondeado mitad hacia arriba a los decimales de esa moneda |
| 16 | Incluye ISC | Sí / No |
| 17 | ISC | La suma de las líneas, o la celda vacía sin ISC |
| 18 | Unidades | Número de líneas |
| 19 | Orden de compra | Su número, o vacía |
| 20 | Estado | Traducido: Vigente, Anulada |
| 21 | Estado de pago (desde el 09/11/2026, D-244) | Traducido: Sin pago, Pago parcial, Pagada, Saldada, Anulada por notas; vacía en una factura anulada. Sin montos |
| ~~21~~ 22 | Motivo de anulación | Traducido: Error de registro, Anulada por el emisor |
| ~~22~~ 23 | Anulada (zona) | Momento, con la zona en el encabezado |
| ~~23~~ 24 | Registrada (zona) | Momento, con la zona en el encabezado |

Las columnas 12, 13 y 15 llevan «(calculado)» en su encabezado (D-180), porque pueden diferir en céntimos de lo impreso (hallazgo M11). La zona de los momentos es la del país de la empresa por la que filtra la exportación o, si no filtra, la de la única que ve quien pide; si ve varias, `America/Lima` (diccionario §2.6, como el Inventario). Nunca sale el `id` ni el `public_id`.

**Como cambió (09/11/2026, D-244)** (backend `8fc04028` y `eb80ced8`). La columna 21, «Estado de pago», sigue a «Estado», traducida con las claves `export.purchase-invoices.payment-status.sin-pago`, `parcial`, `pagada`, `saldada` y `anulada-por-notas`, y sin montos, porque exportar solo pide `compras.ver`. La exportación toma también el filtro `paymentStatus`. La versión del archivo empieza por la de sus columnas, `purchase-invoices-layout=2`, y suma la de las notas y la de los pagos, que cambian el estado aunque la factura no cambie. Pruebas: `PurchaseInvoicesExportIntegrationTests.theStatusOfPaymentIsAColumnAndAFilterOfTheFile` y las columnas de la exportación, con la nueva.

**Como cambia (10/11/2026, D-248)** (decidida en el diccionario v1.53; se construye en `feature/unit-level-balance`). El estado de pago de una factura se calcula con la regla por vehículo, así que la versión del archivo pasa a `purchase-invoices-layout=3` (`PURCHASE_INVOICES_LAYOUT`), para que no se sirva un archivo generado con la regla anterior. Las columnas no cambian. **Como se construyó (10/11/2026)** (backend `c5444af2`): así, y lo prueba `PurchaseInvoicesExportIntegrationTests.theStatusMeasuredByUnitIsPartOfTheVersionOfTheExport`.

#### 4.2.3 `GET /purchase-invoices/{id}`

`404 PURCHASEINVOICE_NOT_FOUND` si no existe o quien llama no la lee. Responde:

```json
{
  "id": "019a…", "companyId": "0199…", "companyName": "Millennium Perú",
  "supplierId": "0199…", "supplierLegalName": "Crosland Automotriz S.A.C.", "supplierTaxId": "20100128056",
  "documentType": "factura", "documentNumber": "F001-123", "invoiceDate": "2026-11-02",
  "currency": "USD", "exchangeRate": 3.742000, "exchangeRateDate": "2026-10-31", "taxRate": 0.1800,
  "grossTotal": { "amount": 472000.00, "currency": "USD" },
  "calculatedBase": { "amount": 400000.00, "currency": "USD" },
  "calculatedTax": { "amount": 72000.00, "currency": "USD" },
  "hasIscColumn": true, "iscTotal": { "amount": 33600.00, "currency": "USD" },
  "purchaseOrderId": "019a…", "purchaseOrderNumber": "OC-LBV-2026-0014",
  "hasFile": true, "hasXml": false,
  "status": "VIGENTE", "annulledAt": null, "annulledBy": null, "annulmentReason": null,
  "createdAt": "2026-11-02T15:04:00Z", "createdBy": { "id": "0199…", "name": "Tere Ramos" },
  "updatedAt": "2026-11-02T15:04:00Z",
  "items": [
    { "id": "019a…", "vehicleId": "019a…", "vin": "MD2A11CY6LWA12345", "engineNumber": "JLZCCL12345",
      "plate": null, "modelId": "0199…", "modelName": "DISCOVER 125-ST", "brandName": "BAJAJ", "modelYear": 2026,
      "commercialBrandId": "0199…", "commercialBrandName": "Lima Bikes", "colorId": "0199…", "colorName": "NEGRO",
      "invoicedAmount": { "amount": 5900.00, "currency": "USD" }, "iscAmount": { "amount": 420.00, "currency": "USD" },
      "status": "VIGENTE", "annulledAt": null, "unitAnnulled": false }
  ],
  "allowedActions": ["edit", "annul", "annulAndCopy", "downloadFile"]
}
```

- Las líneas llegan en el orden en que se registraron, por su `id` (como PRC-R30).
- El VIN, el motor, la placa, el color y si la unidad está anulada son los de hoy de la unidad (`fetchUnitsByIds`): un VIN corregido con `stock.corregir_vin` se ve corregido (D-161).
- `createdBy` y `annulledBy` se nombran por `AuditActorLookupPort`, entre los colaboradores que ve quien lee, como `correctedBy` de la tasa.
- `updatedAt` es el de la fila, el que el `PUT` y la anulación devuelven (D-178).

**Como cambió (09/11/2026, D-244)** (backend `edd12f9a` y `b250636d`). La ficha suma `paymentStatus`, `appliedTotal` y `owed`, con las reglas de las filas de la lista (§4.2.1). `appliedTotal` es lo que cubren los pagos vigentes, sus montos y sus retenciones; `owed`, el total menos sus notas vigentes menos `appliedTotal`, negativo cuando se pagó de más; los dos como `{ "amount", "currency" }` en la moneda de la factura. El `owed` de una factura de un dealer sigue contando las iniciales de sus unidades saldadas, que la empresa nunca paga: una factura `SALDADA` puede tener `owed` mayor que cero, y lo explica `settledInitials` de su posición de pago ([especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md) §4.2.8). El estado lo decide `InvoicePaymentStatusPolicy` y lo nombra `InvoicePaymentStatus`, con las reglas de D-244.

#### 4.2.4 `POST /purchase-invoices`

`multipart/form-data` con tres partes: `invoice` (JSON, `application/json`, obligatoria), `file` (opcional) y `xml` (opcional). El tipo que declara cada parte no se cree: se verifica el contenido (D-174).

```json
{
  "companyId": "0199…",
  "supplierId": "0199…",
  "documentType": "factura",
  "documentNumber": "F001-00000123",
  "invoiceDate": "2026-11-02",
  "currency": "USD",
  "exchangeRate": "3.742000",
  "exchangeRateDate": "2026-10-31",
  "taxRate": "0.1800",
  "grossTotal": { "amount": "11800.00", "currency": "USD" },
  "hasIscColumn": false,
  "purchaseOrderId": null,
  "confirmOrderDeviation": false,
  "items": [
    { "vin": "MD2A11CY6LWA12345", "engineNumber": "JLZCCL12345", "modelId": "0199…", "modelYear": 2026,
      "colorId": "0199…", "commercialBrandId": "0199…",
      "invoicedAmount": { "amount": "5900.00", "currency": "USD" }, "iscAmount": null }
  ]
}
```

| Campo | Tipo | Obligatorio | Restricción (`400 VALIDATION_ERROR` si no se cumple) |
|---|---|---|---|
| `companyId`, `supplierId` | UUID | Sí | — |
| `documentType` | texto | Sí | `factura`, `boleta` u `otro`, en cualquier mayúscula |
| `documentNumber` | texto | Sí | De 1 a 30 caracteres después de normalizar; uno hecho solo de separadores queda vacío y responde `400` |
| `invoiceDate`, `exchangeRateDate` | fecha | Sí | ISO |
| `currency` | texto | Sí | Tres letras |
| `exchangeRate` | decimal | Sí | Mayor que 0, menor que 1 000 000, hasta seis decimales: la tasa que la pantalla mostró |
| `taxRate` | decimal | Sí | De 0 a menos de 1, hasta cuatro decimales |
| `grossTotal` | monto | Sí | Mayor que 0, hasta `999999999999.99`, en `currency` |
| `hasIscColumn` | booleano | No | `false` por omisión |
| `purchaseOrderId` | UUID | No | — |
| `confirmOrderDeviation` | booleano | No | Se manda solo después de que la persona confirmó (§4.4) |
| `items` | arreglo | Sí | De 1 a 100 (D-160); un elemento `null` responde en `items[n]` |
| `items[n].vin` | texto | Sí | Se recorta y pasa a mayúsculas; 17 caracteres `[A-HJ-NPR-Z0-9]`, sin I, O ni Q (D-160) |
| `items[n].engineNumber` | texto | No | Hasta 50. En una línea que vincula una unidad existente se ignora (D-179) |
| `items[n].modelId`, `commercialBrandId` | UUID | Sí | — |
| `items[n].colorId` | UUID | Sí | En una línea que vincula una unidad existente se ignora, y no se resuelve (D-179) |
| `items[n].modelYear` | entero | Sí | De 2017 al año siguiente al de hoy en la zona de la empresa, como la unidad |
| `items[n].invoicedAmount` | monto | Sí | Mayor que 0 (D-179), hasta el tope, en `currency` |
| `items[n].iscAmount` | monto | No | Desde 0, en `currency` |

**Revisiones, en este orden.** Cada una responde el primer fallo, con `context.lines` cuando nombra líneas (§4.3):
1. Forma: `400 VALIDATION_ERROR`. Archivos: `422 PURCHASEINVOICE_FILE_TYPE`, `PURCHASEINVOICE_FILE_SIZE`, `PURCHASEINVOICE_XML_INVALID`, `PURCHASEINVOICE_XML_SIZE`; más de 10 MB por parte o 21 MB por pedido corta antes el servidor (`413 CONTENT_TOO_LARGE`).
2. Sin `compras.registrar_factura` en ninguna empresa: `403 ACCESS_DENIED`.
3. Empresa: `404 COMPANY_NOT_FOUND` si no existe, no se alcanza o no se lee con `compras.ver`; `403 ACCESS_DENIED` si `compras.registrar_factura` no actúa en toda la empresa; `422 COMPANY_INACTIVE`.
4. Proveedor: `404 SUPPLIER_NOT_FOUND` si no es de la empresa; `422 SUPPLIER_INACTIVE` (D-84).
5. Moneda: `404 CURRENCY_NOT_FOUND`; `422 CURRENCY_INACTIVE`.
6. Montos: `422 PURCHASEINVOICE_AMOUNT_SCALE`; `422 PURCHASEINVOICE_TOTAL_MISMATCH`; `422 PURCHASEINVOICE_ISC_REQUIRED`, `PURCHASEINVOICE_ISC_FORBIDDEN` o `PURCHASEINVOICE_ISC_EXCEEDS_AMOUNT`.
7. Número: `422 PURCHASEINVOICE_NUMBER_FORMAT` (solo en Perú); `409 PURCHASEINVOICE_NUMBER_CONFLICT`.
8. Fecha: `422 PURCHASEINVOICE_DATE_IN_FUTURE`; `422 PURCHASEINVOICE_TOO_OLD` sin `compras.registrar_factura_antigua` en la empresa sin sede: de sistema o de la empresa, con o sin marca (`actsOnWholeCompany`); las marcas se miran en el paso 10.
9. Tasa: `422 PURCHASEINVOICE_EXCHANGE_RATE_MISSING`; `422 PURCHASEINVOICE_EXCHANGE_RATE_CHANGED`.
10. Marcas y modelos: `404 COMMERCIALBRAND_NOT_FOUND` (una marca que no es de la empresa); `403 ACCESS_DENIED` (una marca donde `compras.registrar_factura` no actúa, D-163); en una factura antigua, `422 PURCHASEINVOICE_TOO_OLD` con `context.ageInDays` (una marca donde `compras.registrar_factura_antigua` no actúa); `404 VEHICLEMODEL_NOT_FOUND`.

   **Como se corrigió (04/11/2026)** (backend `5f0a01b1`, C4 de la revisión). La revisión por marca del permiso de facturas antiguas respondía un `403 ACCESS_DENIED` genérico. Ahora responde lo mismo que la fecha, `422 PURCHASEINVOICE_TOO_OLD` con los días de la factura: quien tiene ese permiso solo para otra marca pasa el paso 8, que no mira las marcas, y se detiene aquí. El `403` del paso 10 queda para `compras.registrar_factura`. Prueba: `PurchaseInvoicesIntegrationTests.anOldInvoiceWithThePermissionOfAnotherBrandOnlyAnswersTooOld`.
11. `422 PURCHASEINVOICE_DUPLICATE_VIN`.
12. Orden: `404 PURCHASEORDER_NOT_FOUND`; `422 PURCHASEINVOICE_ORDER_NOT_SENT`; `422 PURCHASEINVOICE_ORDER_MISMATCH` (la base lo sostiene con su llave compuesta).
13. Unidades existentes: `409 VEHICLE_VIN_CONFLICT` (de otra empresa, sin sus datos); `422 VEHICLE_ANNULLED`; `422 PURCHASEINVOICE_UNIT_MISMATCH` (solo el modelo, el año y la marca comercial: el motor y el color no se comparan, D-179); `409 PURCHASEINVOICE_UNIT_CONFLICT`.
14. `422 PURCHASEINVOICE_ORDER_DEVIATION_UNCONFIRMED`, salvo `confirmOrderDeviation: true`.
15. Unidades nuevas: sus colores, en lote (`404 COLOR_NOT_FOUND`), y por la fachada los códigos de catálogo y de estado de §2.5.1, y `409 VEHICLE_VIN_CONFLICT` en una carrera.
16. Guardado: `409 PURCHASEINVOICE_NUMBER_CONFLICT` o `409 PURCHASEINVOICE_UNIT_CONFLICT` en una carrera que la base detiene, sin dejar nada escrito.

Responde `201` con `Location: /api/v1/purchase-invoices/{id}` y la ficha (§4.2.3).

#### 4.2.5 `PUT /purchase-invoices/{id}`

Cambia lo no fiscal de una factura vigente: el archivo, el XML y la orden, y nada fiscal (D-161, D-172, D-178). `multipart/form-data`: `invoice` (JSON) y, si se agregan o se reemplazan, `file` y `xml`.

```json
{ "updatedAt": "2026-11-02T15:04:00.123Z", "purchaseOrderId": "019a…", "confirmOrderDeviation": false }
```

- `purchaseOrderId` es el valor nuevo: el mismo no cambia nada, otro vincula y `null` desvincula. Desvincular se puede siempre, y una orden ya cumplida sigue cumplida (D-172). **Como se construyó** (`c37dda82`): un `PUT` que manda la orden que la factura ya tiene no cambia nada de la orden: no la vuelve a revisar ni cuenta sus desviaciones, aunque la orden ya esté cumplida, y el archivo o el XML cambian igual. Solo otra orden, o la primera, se revisa como en el alta; `null` sigue desvinculando.
- Un archivo que no viaja se conserva. No se quita: se reemplaza.
- `updatedAt` es obligatorio: el que leyó quien edita (D-178).

**Revisiones:** forma (`400`, también sin `updatedAt`); archivos (`422`, como el alta); la factura (`404 PURCHASEINVOICE_NOT_FOUND`), que se bloquea; `compras.registrar_factura` con el alcance de registrar, toda la empresa y la marca de cada línea (`403`, D-178); `422 PURCHASEINVOICE_OUTDATED`; `422 PURCHASEINVOICE_ANNULLED`; si cambia la orden, la orden nueva como en el alta (`404`, `422 …ORDER_NOT_SENT`, `422 …ORDER_MISMATCH`) y sus desviaciones, contadas sin esta factura (`422 …ORDER_DEVIATION_UNCONFIRMED`).

Cada campo que cambia deja una fila en la bitácora, no delicada (diccionario §13.1): `purchase_order_id`, con el número de la orden anterior y el de la nueva; `file_path` y `xml_path`, con la llave anterior y la nueva. Responde `200` con la ficha.

#### 4.2.6 `POST /purchase-invoices/{id}/annulment`

```json
{ "reason": "ERROR_REGISTRO", "updatedAt": "2026-11-02T15:04:00.123Z" }
```

- `reason`: `ERROR_REGISTRO` o `ANULADA_POR_EMISOR`, en cualquier mayúscula; otro valor, `400`.
- `updatedAt`: obligatorio, el que leyó quien anula (D-178).

**Revisiones:** forma (`400`); la factura (`404`), que se bloquea con sus líneas; `compras.anular_factura` con el alcance de registrar, toda la empresa y la marca de cada línea (`403`, D-178); `422 PURCHASEINVOICE_OUTDATED`; `422 PURCHASEINVOICE_ANNULLED`; `422 PURCHASEINVOICE_HAS_DEPENDENTS`.

Marca `annulled_at`, `annulled_by` (la cuenta de la sesión) y `annulment_reason`, y cada línea vigente pasa a `ANULADA`. Escribe una fila delicada en la bitácora (D-175): `entity = purchase_invoices`, `field_name = annulled_at`, `old_value = null`, `new_value` = el momento en ISO, `reason` = el código del motivo, `company_id` = el de la factura y `commercial_brand_id` nulo, porque la factura no es de una marca. Las unidades no se anulan (D-161). Responde `200` con la ficha.

#### 4.2.7 `GET /purchase-invoices/{id}/file` y `/xml`

Entregan los bytes desde `KeptFileStore.open`, con su `Content-Type` y `Content-Disposition: attachment; filename="factura-F001-123.pdf"`: el número normalizado y la extensión de su tipo; el XML, `.xml`. Responden `404 PURCHASEINVOICE_NOT_FOUND` si no se lee la factura, y `404 PURCHASEINVOICE_FILE_NOT_FOUND` si no tiene ese archivo o falta en el almacén.

#### 4.2.8 `GET /purchase-invoice-vin-checks` ~~(propuesta)~~, construida el 03/11/2026 (`a37962bf`)

Lo que la pantalla muestra por línea antes de guardar: nueva, se vincula o rechazada, y por qué.

`?companyId={uuid}&vin=…&vin=…`, de 1 a 100 `vin` (`400` fuera de ese rango). La clase pide `compras.ver` y el método `compras.registrar_factura`; el servicio, como el alta: la empresa se lee (`404`) y el permiso actúa en toda ella (`403`). No escribe ni bloquea. Responde un arreglo en el orden pedido:

```json
[
  { "vin": "MD2A11CY6LWA12345", "status": "new", "cause": null, "unit": null, "currentInvoice": null },
  { "vin": "MD2A11CY6LWA12346", "status": "link", "cause": null,
    "unit": { "id": "019a…", "modelId": "0199…", "modelName": "DISCOVER 125-ST", "brandName": "BAJAJ",
              "modelYear": 2026, "commercialBrandId": "0199…", "commercialBrandName": "Lima Bikes",
              "colorId": "0199…", "colorName": "NEGRO", "engineNumber": "JLZCCL12346" },
    "currentInvoice": null },
  { "vin": "MD2A11CY6LWA12347", "status": "rejected", "cause": "PURCHASEINVOICE_UNIT_CONFLICT", "unit": null,
    "currentInvoice": { "id": "019a…", "documentNumber": "F001-98", "supplierLegalName": "Crosland Automotriz S.A.C." } }
]
```

- `status` es un dato derivado, en el estilo de `allowedActions`: `new`, `link` o `rejected`.
- `cause` es el código que respondería el alta: `VALIDATION_ERROR` (mal formado), `PURCHASEINVOICE_DUPLICATE_VIN` (desde su segunda aparición), `VEHICLE_VIN_CONFLICT` (de otra empresa), `VEHICLE_ANNULLED` o `PURCHASEINVOICE_UNIT_CONFLICT`. Así la pantalla traduce cada causa con el mismo texto que el error.
- `unit` llega solo si quien llama ve la unidad con `stock.ver` en su marca (D-102); si no, `link` con `unit: null`.
- `currentInvoice` nombra la factura vigente que ya tiene la unidad; quien pregunta lee esa empresa con `compras.ver`.

#### 4.2.9 Lo que cambia en la orden

- **`GET /purchase-orders/{id}`** suma `invoicing`, `null` en una orden sin líneas de vehículo:

  ```json
  "invoicing": { "allInvoiced": false, "lines": [
      { "modelId": "0199…", "modelYear": 2026, "description": "BAJAJ DISCOVER 125-ST, AÑO 2026",
        "ordered": 80, "invoiced": 78, "pending": 2 } ] }
  ```

  `invoiced` cuenta las líneas vigentes de las facturas vigentes vinculadas; `allInvoiced` es verdadero cuando ningún grupo tiene pendiente (D-162). Lo lee `PurchaseOrderInvoicingReader` en una consulta agrupada.
- **`GET /purchase-orders`** acepta `supplierId`, para el formulario de la factura (convenciones §6.2).
- **`POST /purchase-orders/{id}/annulment`** responde `422 PURCHASEORDER_HAS_CURRENT_INVOICES` si la orden, bloqueada, tiene una factura vigente (D-162). `allowedActions` de la orden deja de ofrecer `annul` en ese caso.

#### 4.2.10 Lo que cambia en la unidad

- **`POST /vehicles/{id}/annulment`**, sin cuerpo, como un cierre en el momento del pedido (convenciones §3.2). **Revisiones:** la unidad (`404 VEHICLE_NOT_FOUND` si `stock.ver` no la ve), que se bloquea; `stock.anular_vehiculo` donde está la unidad (`403`, D-101, D-102); `422 VEHICLE_ANNULLED`; `422 VEHICLE_VALUE_NOT_ZERO` (`UnitPurchaseFacts.valueIsZero`); `422 VEHICLE_HAS_CONTRACTS` (`UnitContractFacts`); `422 VEHICLE_FUND_ASSIGNMENT_OPEN` si tiene una cesión abierta a un fondo, sin fecha de fin (diccionario §12.2): primero se cierra la cesión (D-182). La cesión se lee bajo el candado de la unidad, que también toman ceder y cerrar una cesión, así que una cesión nueva y la anulación no se cruzan. Marca `annulled_at`; el autor queda en `updated_by`. No escribe en la bitácora, porque es una baja (diccionario §13.1: «Las altas y las bajas no dejan filas aquí»). Responde `200` con la unidad.
- **Toda escritura de una unidad anulada** responde `422 VEHICLE_ANNULLED`, después de su `404` y de su `403`: `PUT /vehicles/{id}`, `POST /vehicles/{id}/status-changes`, `/mileage-readings`, `PUT /vehicles/{id}/plate`, `/vin`, `POST /vehicles/{id}/fund-assignments` y `POST /fund-assignments/{id}/closure`, sin excepción. El regreso automático se salta las unidades anuladas. Como una unidad con cesión abierta no se anula (D-182), una anulada no tiene ninguna cesión que cerrar.
- **`allowedActions`** de una unidad anulada: todas las escrituras en falso, `walkableReasonIds` vacío y, en sus cesiones, `close` en falso. `viewChangeLog` conserva su valor, porque solo lee. Suma `annul`: verdadero si `stock.anular_vehiculo` actúa donde está la unidad, no está anulada, su valor es cero, no tiene contratos y no tiene una cesión abierta (D-182).
- **`PUT /vehicles/{id}`** con otro `modelYear` en una unidad con línea vigente responde `422 VEHICLE_MODEL_YEAR_LOCKED` (D-160), bajo el candado de la unidad. El modelo ya no cambia por esa ruta (FLT-R08).
- **`GET /vehicles`** acepta `hasCurrentInvoice` (`true` o `false`) y cada unidad trae `hasCurrentInvoice`, en la lista y en la ficha. La exportación suma la columna «Factura vigente» (Sí/No) después de «Estado», y su versión suma la de `UnitPurchases` (D-175).

### 4.3 Códigos de error

El estado sale del sufijo (convenciones §7.2): `_NOT_FOUND` es `404`, `_CONFLICT` es `409` y los demás propios, `422`. Los textos van en `error.<código-en-kebab>.message` de los dos `messages*.properties` y en `es.json`. **`context.lines`**, en los códigos que nombran líneas, es `[{ "line": 3, "vin": "MD2A…" }]`, con `line` desde 1 en el orden de `items`; los que comparan campos suman `fields`, y nunca llevan datos de una unidad de otra empresa.

| Código | Estado | Dónde | Mensaje en español |
|---|---|---|---|
| `PURCHASEINVOICE_NOT_FOUND` | 404 | Ficha, escrituras, archivos | Factura de compra no encontrada |
| `PURCHASEINVOICE_FILE_NOT_FOUND` | 404 | `/file`, `/xml` | La factura no tiene ese archivo |
| `PURCHASEINVOICE_NUMBER_CONFLICT` | 409 | Alta | Ya hay una factura vigente de ese proveedor con ese tipo y número |
| `PURCHASEINVOICE_UNIT_CONFLICT` | 409 | Alta | La unidad ya está en otra factura vigente (`context.lines`) |
| `PURCHASEINVOICE_NUMBER_FORMAT` | 422 | Alta | En Perú el número es la serie de cuatro caracteres, un guion y el correlativo de hasta ocho dígitos, como F001-123 |
| `PURCHASEINVOICE_DATE_IN_FUTURE` | 422 | Alta | La fecha del comprobante no puede ser posterior a hoy (`context.today`) |
| `PURCHASEINVOICE_TOO_OLD` | 422 | Alta | Una factura de más de 20 días la registra solo quien tiene el permiso de facturas antiguas (`context.ageInDays`). Desde el 04/11/2026 responde también cuando el permiso no alcanza la marca de una línea (§4.2.4, paso 10) |
| `PURCHASEINVOICE_AMOUNT_SCALE` | 422 | Alta | Un monto tiene más decimales de los que permite la moneda (`context.lines`, o sin líneas si es el total) |
| `PURCHASEINVOICE_TOTAL_MISMATCH` | 422 | Alta | La suma de las líneas no coincide con el total del comprobante (`context.linesTotal`, `context.grossTotal`, dos números sin moneda, no objetos `{amount, currency}`) |
| `PURCHASEINVOICE_ISC_REQUIRED` | 422 | Alta | Con «Incluye ISC», cada línea lleva su ISC (`context.lines`) |
| `PURCHASEINVOICE_ISC_FORBIDDEN` | 422 | Alta | Sin «Incluye ISC», ninguna línea lleva ISC (`context.lines`) |
| `PURCHASEINVOICE_ISC_EXCEEDS_AMOUNT` | 422 | Alta | El ISC de una línea no puede superar su monto (`context.lines`) |
| `PURCHASEINVOICE_DUPLICATE_VIN` | 422 | Alta | Un VIN aparece más de una vez en el comprobante (`context.lines`) |
| `PURCHASEINVOICE_UNIT_MISMATCH` | 422 | Alta | La unidad ya registrada tiene otro modelo, año o marca comercial que la línea (`context.lines[].fields`) |
| `PURCHASEINVOICE_EXCHANGE_RATE_MISSING` | 422 | Alta | No hay tasa registrada para esa moneda en la fecha del comprobante ni antes (`context.country`, `fromCurrency`, `toCurrency`, `rateType`, `date`) |
| `PURCHASEINVOICE_EXCHANGE_RATE_CHANGED` | 422 | Alta | La tasa de la fecha cambió mientras se registraba la factura (`context.exchangeRate`, `context.exchangeRateDate`) |
| `PURCHASEINVOICE_ORDER_NOT_SENT` | 422 | Alta, `PUT` | Solo se vincula una orden enviada |
| `PURCHASEINVOICE_ORDER_MISMATCH` | 422 | Alta, `PUT` | La orden es de otro proveedor o de otra empresa |
| `PURCHASEINVOICE_ORDER_DEVIATION_UNCONFIRMED` | 422 | Alta, `PUT` | La factura trae modelos o cantidades que la orden no pide (`context.deviations`, §4.4) |
| `PURCHASEINVOICE_FILE_TYPE` | 422 | Alta, `PUT` | El archivo del comprobante debe ser PDF, PNG, JPEG o WebP |
| `PURCHASEINVOICE_FILE_SIZE` | 422 | Alta, `PUT` | El archivo del comprobante pesa más de 5 MB |
| `PURCHASEINVOICE_XML_INVALID` | 422 | Alta, `PUT` | El XML del comprobante no está bien formado |
| `PURCHASEINVOICE_XML_SIZE` | 422 | Alta, `PUT` | El XML del comprobante pesa más de 2 MB |
| `PURCHASEINVOICE_OUTDATED` | 422 | `PUT`, anulación | La factura cambió desde que se abrió (D-178) |
| `PURCHASEINVOICE_ANNULLED` | 422 | `PUT`, anulación | La factura está anulada: ya no cambia |
| `PURCHASEINVOICE_HAS_DEPENDENTS` | 422 | Anulación | La factura tiene notas de crédito o pagos aplicados: no se anula |
| `PURCHASEORDER_HAS_CURRENT_INVOICES` | 422 | Anular una orden | La orden tiene facturas vigentes: no se anula |
| `VEHICLE_ANNULLED` | 422 | Escrituras de la unidad; alta de factura | La unidad está anulada: ya no se modifica |
| `VEHICLE_MODEL_YEAR_LOCKED` | 422 | `PUT /vehicles/{id}` | La unidad está en una factura vigente: su año no cambia |
| `VEHICLE_VALUE_NOT_ZERO` | 422 | Anular la unidad | La unidad tiene valor de compra: se anula solo con valor cero |
| `VEHICLE_HAS_CONTRACTS` | 422 | Anular la unidad | La unidad tiene contratos: no se anula |
| `VEHICLE_FUND_ASSIGNMENT_OPEN` | 422 | Anular la unidad | La unidad tiene una cesión abierta a un fondo: ciérrela antes de anularla. (D-182) |
| `PURCHASEINVOICE_IMPORT_FILE_TYPE` | 422 | Importar líneas (C10, §11.2) | El archivo de líneas no es un libro de Excel (xlsx): descargue la plantilla y llénela |
| `PURCHASEINVOICE_IMPORT_FILE_SIZE` | 422 | Importar líneas | El archivo de líneas pesa más de 2 MB (`context.maxBytes`) |
| `PURCHASEINVOICE_IMPORT_HEADERS` | 422 | Importar líneas | La primera hoja no tiene la cabecera de la plantilla: VIN, Motor, Color, Marca, Modelo, Año, Valor e ISC (`context.expected`, `context.found`) |
| `PURCHASEINVOICE_IMPORT_ROWS` | 422 | Importar líneas | El archivo trae más de 100 líneas, el máximo que admite una factura (`context.max`, `context.found`) |

**Reutilizados**, con `context.lines` cuando nombran una línea: `VALIDATION_ERROR` (400), `ACCESS_DENIED` (403), `CONTENT_TOO_LARGE` (413), `COMPANY_NOT_FOUND`, `SUPPLIER_NOT_FOUND`, `CURRENCY_NOT_FOUND`, `COMMERCIALBRAND_NOT_FOUND`, `VEHICLEMODEL_NOT_FOUND`, `COLOR_NOT_FOUND`, `PURCHASEORDER_NOT_FOUND`, `VEHICLE_NOT_FOUND` (404), `VEHICLE_VIN_CONFLICT` (409), `COMPANY_INACTIVE`, `SUPPLIER_INACTIVE`, `CURRENCY_INACTIVE`, `COMMERCIALBRAND_INACTIVE`, `VEHICLEMODEL_INACTIVE`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE`, `COLOR_INACTIVE`, `VEHICLESTATUS_INITIAL_MISSING`, `STATUSCHANGEREASON_REGISTRATION_MISSING` (422). Las formas de `context` se documentan en `ErrorResource` y en las respuestas `422` de cada operación (convenciones §7.1).

### 4.4 Avisos que se confirman

Hay uno, con el patrón de `EXCHANGERATE_LARGE_CHANGE_UNCONFIRMED`. Facturar un modelo o un año que la orden no pide, o más unidades que lo pendiente, responde `422 PURCHASEINVOICE_ORDER_DEVIATION_UNCONFIRMED` y no guarda nada (D-162):

```json
"context": { "deviations": [
  { "modelId": "0199…", "modelYear": 2026, "description": "BAJAJ DISCOVER 125-ST, AÑO 2026",
    "ordered": 80, "invoiced": 78, "invoicing": 5, "pending": 2 },
  { "modelId": "0199…", "modelYear": 2027, "description": "BAJAJ DISCOVER 125-ST, AÑO 2027",
    "ordered": 0, "invoiced": 0, "invoicing": 1, "pending": 0 } ] }
```

La pantalla pregunta con esa tabla; «Registrar igual» manda el mismo pedido con `confirmOrderDeviation: true`, y «Revisar» vuelve al formulario sin guardar. El precio no se compara (D-162).

El aviso de la tasa anterior en un día hábil (D-151) **no** es un aviso del backend: lo muestra el formulario, y no impide seguir.

### 4.5 `allowedActions`

Solo la ficha y las respuestas de las escrituras lo traen; ~~las filas de la lista, `null`~~ desde el 09/11/2026, también cada fila de la lista, con la misma regla (D-244). Siempre en este orden, cada acción con las mismas reglas que su operación:

| Acción | Cuándo se ofrece |
|---|---|
| `edit` | Factura vigente y `compras.registrar_factura` con el alcance de registrar: toda la empresa y la marca de cada línea (D-178) |
| `annul` | Factura vigente, `compras.anular_factura` con ese alcance y sin dependientes (`PurchaseInvoiceDependents`) |
| `annulAndCopy` | `annul`, más `compras.registrar_factura` y, si la factura tiene más de 20 días, `compras.registrar_factura_antigua` (hallazgo B6) |
| `downloadFile` | Tiene archivo |
| `downloadXml` | Tiene XML |

Todas exigen además leer la factura con `compras.ver`. Lo calcula `PurchaseInvoiceAllowedActions`.

**Como cambió (09/11/2026, D-244)** (backend `b250636d`). Las notas de crédito y los pagos sumaron `registerCreditNote` y `registerPayment` al final, en ese orden. Las filas de la lista traen sus acciones con la misma regla que la ficha: lo que la regla necesita de cada factura —sus dependientes y la zona de su empresa— se lee una vez por página, y el menú de la fila ofrece de ellas «Registrar pago», «Registrar nota de crédito» y «Descargar comprobante» (§6.2). Prueba: `PurchaseInvoiceReadsIntegrationTests.theRowsOfTheListOfferWhatTheirRecordOffers`.

### 4.6 Versiones y concurrencia

- **La tasa vista** (A19). El alta lleva `exchangeRate` y `exchangeRateDate` tal como los mostró la pantalla. El servicio calcula la vigente, en la moneda funcional `(1, invoiceDate)`, y si el valor o el día difieren responde `422 PURCHASEINVOICE_EXCHANGE_RATE_CHANGED` con la vigente en `context`; la pantalla la muestra y pide guardar otra vez.
- **El candado de `updatedAt`** en `PUT` y en la anulación, comparado al milisegundo bajo el candado de la fila (convenciones §3): el `PUT` del archivo, del XML o de la orden y la anulación llevan el `updatedAt` que leyó quien opera, y uno que ya no es el de la fila se rechaza sin guardar nada, como el borrador de la orden (D-138, D-178). La pantalla, ante `PURCHASEINVOICE_OUTDATED`, vuelve a leer la ficha, la muestra como está y lo dice. **Como se corrigió (04/11/2026)** (frontend `ea714437`). El diálogo «Cambiar archivo u orden» seguía con la factura con que se abrió, así que un segundo intento volvía a mandar el `updatedAt` viejo. Ahora toma la factura leída otra vez: su orden y su `updatedAt` reemplazan los del diálogo, el texto del rechazo sigue a la vista y los archivos ya elegidos se conservan, porque son de la persona y no de la ficha. Nunca manda el candado nuevo con la orden vieja. Prueba: `edit-purchase-invoice-dialog.spec` ("after PURCHASEINVOICE_OUTDATED takes the order and the updatedAt read again, and keeps the file picked").
- **El doble envío** del mismo alta choca en `purchase_invoices_number_uq` y responde `409 PURCHASEINVOICE_NUMBER_CONFLICT`.
- **La versión de la API** no cambia (convenciones §1). Los cambios de §4.2.9 y §4.2.10 agregan campos y filtros, y el nuevo `422` de la anulación de la orden lo traduce el frontend en el mismo cambio.

---

## 5. Reglas

Cada regla pasa a ✅ cuando su prueba existe y pasa. Las clases de prueba del backend siguen los paquetes de hoy (`procurement/domain/...`, `.../infrastructure/persistence/...`, `.../interfaces/rest/...`); las del frontend van junto a su archivo.

### 5.1 `procurement`

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| PRC-R31 | La factura se registra a mano desde el comprobante; el XML, si se adjunta, se guarda y no se lee | D-157 | Servicio: `PurchaseInvoiceFiles` guarda `xml_path` sin leer su contenido | `PurchaseInvoicesIntegrationTests.anAttachedXmlIsKeptAndNothingIsReadFromIt` | ✅ |
| PRC-R32 | El número se guarda normalizado, sin ceros a la izquierda del correlativo, y no se repite en el proveedor y el tipo entre las facturas no anuladas; anular una libera su número. Desde el 04/11/2026, la búsqueda de la lista también lo encuentra escrito como se escribiría al registrarlo (C7) | D-160 | Dominio: `InvoiceNumber`, `InvoiceNumber.searchForm`. Base: `purchase_invoices_number_uq`. Adaptador: `UniqueViolations` → `NaturalKey.PURCHASE_INVOICE_NUMBER` → `409`; `PurchaseInvoiceRepositoryImpl` (búsqueda, backend `53f11cd3`). Frontend: vista previa del número | `InvoiceNumberTests.leadingZerosOfTheCorrelativeAreDropped`; `PurchaseInvoicesIntegrationTests.theSameRequestTwiceAnswersNumberConflict`, `anAnnulledInvoiceFreesItsNumber`; `PurchaseInvoiceReadsIntegrationTests.theFiltersNarrowTheListAndAValueOutOfItsListIsRefused` (`F001-00000003` encuentra `F001-3`); `PurchaseInvoiceSchemaTests.theNumberDoesNotRepeatAmongTheInvoicesNotAnnulledAndAnnullingOneFreesIt`; `PurchaseInvoiceLockTests.twoRegistrationsOfTheSameNumberLeaveOneInvoice`; `invoice-number.spec` ("shows the number as it will be saved", "folds the dashes of Unicode and the minus sign into a hyphen, as the backend saves them") | ✅ |
| PRC-R33 | En Perú la serie tiene cuatro caracteres y el correlativo de 1 a 8 dígitos; en otro país se acepta sin verificar el formato | D-160 | Dominio: `InvoiceNumber.holdsPeruvianFormat`. Servicio: con el país de la empresa | `InvoiceNumberTests.aPeruvianNumberHasASeriesOfFourAndACorrelativeOfUpToEight`; `PurchaseInvoicesIntegrationTests.aNumberOutsidePeruIsNotCheckedForItsFormat`; `invoice-number.spec` ("checks the form of Peru: a series of four, a hyphen and up to eight digits") | ✅ |
| PRC-R34 | La fecha no es futura en la zona de la empresa; la factura guarda el día de la tasa que copió | D-160, §2.6 | Dominio: `InvoiceDatePolicy`, con `Clock`. Base: `purchase_invoices_exchange_rate_date_ck`. Frontend: calendario hasta hoy en la zona de la empresa | `InvoiceDatePolicyTests.todayIsTheDayOfTheZoneOfTheCompany`; `ProcurementDatesInLimaTests.anInvoiceDatedTomorrowInLimaIsRefusedEvenIfUtcIsAlreadyThere`, con el reloj fijo a las 20:00 de Lima del 03/11/2026, ya 04/11/2026 en UTC (backend `6165ba5d`: el servicio lee la hora de `ProcurementCalendar`, que la prueba espía, y no de un `Clock.systemUTC()` fijo en el código) | ✅ |
| PRC-R35 | Una factura de más de 20 días exige `compras.registrar_factura_antigua`, en la empresa y en la marca de cada línea; si no alcanza una marca, responde `422 PURCHASEINVOICE_TOO_OLD`, como la fecha (desde el 04/11/2026). «Anular y copiar» de una así, también | §9.3, B6, D-178 | Dominio: `InvoiceDatePolicy.isOld`. Servicio: `PurchaseInvoiceAccessPolicy.registersOld`, también por marca en `PurchaseInvoiceUnits.resolve` (backend `5f0a01b1`). Frontend: aviso bajo la fecha, con el permiso mirado en la empresa y en las marcas de las líneas (frontend `fdf14036`), y `annulAndCopy` | `InvoiceDatePolicyTests.twentyDaysIsNotOldAndTwentyOneIs`; `PurchaseInvoicesIntegrationTests.withoutRegisterInvoicesAnswersAccessDeniedAndAnOldInvoiceNeedsItsPermission`, `anOldInvoiceWithThePermissionOfAnotherBrandOnlyAnswersTooOld`; `PurchaseInvoiceAllowedActionsTests.annulAndCopyOfAnOldInvoiceNeedsTheOldPermission`; `purchase-invoice-form-view.spec` ("holds back a date of more than 20 days without the permission of old invoices, and only warns with it", "holds an old invoice back when the permission of old invoices does not reach the brand of a line (D-178)"); e2e ("a role limited to a brand offers its company, and registers an old invoice only with the lines of its brand") | ✅ |
| PRC-R36 | Una factura tiene de 1 a 100 líneas, y en esta versión solo de vehículo. Una línea de concepto que la base admita (D-22) se lee y las reglas de las unidades la pasan por alto, sin `500` (desde el 04/11/2026) | D-160, D-173, D-22 | Recurso: `@NotEmpty @Size(max = 100)`, sin campo `concept`. Dominio: constructor; `PurchaseInvoiceItem.isConcept` (backend `a7afefd8`). Servicio: la anulación y el `PUT` dejan fuera las marcas nulas (`brandsOf`, backend `76fa8f0d`, segunda revisión). Base: `purchase_invoice_items_vehicle_or_concept_ck` | `PurchaseInvoiceTests.anInvoiceHasOneToOneHundredLines`; `PurchaseInvoicesIntegrationTests.aHundredAndOneLinesAnswerValidationError`; `ConceptLinesTests.aConceptIsNoUnitOfTheInvoice`; `PurchaseInvoiceChangesIntegrationTests.anInvoiceWithAConceptChangesItsOrderAndIsAnnulled` | ✅ |
| PRC-R37 | Cada línea guarda el precio de la unidad con impuestos, y el total es la suma exacta de las líneas | D-159, §9.3 | Dominio: constructor (`PurchaseInvoiceTotalMismatchException`). Frontend: totales en enteros (`BigInt`) | `PurchaseInvoiceTests.theGrossTotalIsTheExactSumOfItsLines`; `purchase-invoice-amounts.spec` ("adds a hundred lines to the cent"); e2e ("pastes 100 lines with the brand, model and year for all of them, and registers the invoice squared to the cent (revision 39)") | ✅ |
| PRC-R38 | Todo monto está en la moneda de la factura, con a lo sumo sus decimales; el valor de factura de cada línea es mayor que cero, en la base y en la API | D-146, D-179, §2.2 | Recurso: `currency` de cada monto. Dominio: `InvoiceAmount`, que acepta ceros más allá de los decimales de la moneda (`5900.00` en una moneda sin decimales). Base: `NUMERIC(14,2)` y los `CHECK`. Frontend: acepta esos ceros igual que el backend (frontend `6386f2fa`) | `InvoiceAmountTests.anAmountHasAtMostTheDecimalsOfItsCurrency`; `PurchaseInvoicesIntegrationTests.theAmountsTheVinsAndTheFileAreCheckedBeforeAnythingIsWritten`, `aLineOfZeroIsRefusedByTheApiAndByTheDatabase` (desde `7984202f`: una línea de 0,00 la rechazan la API, sin escribir nada, y la base, escrita a mano); `purchase-invoice-amounts.spec` ("takes zeros past the decimals of the currency, as the backend does") | ✅ |
| PRC-R39 | El ISC se decide por factura: con `has_isc_column`, cada línea de vehículo lo exige, de 0 al monto de la línea; sin ella, ninguna lo lleva | D-158 | Dominio: constructor. Base: `purchase_invoice_items_isc_tg` (inmediato), `purchase_invoices_isc_tg` (diferido) y `purchase_invoice_items_isc_amount_ck`. Frontend: «Incluye ISC», con «aplicar a todas» | `PurchaseInvoiceTests.withTheIscColumnEveryVehicleLineNeedsItsIscAndWithoutItNoLineCarriesIsc`; `PurchaseInvoiceIscTriggerTests.aLineWithoutIscOfAnInvoiceWithTheColumnIsRefusedByTheDatabase`, `aLineWithIscOfAnInvoiceWithoutTheColumnIsRefusedByTheDatabase`, `aLineOfConceptNeverCarriesIsc`, `theIscOfALineIsNotTakenAwayNorTheColumnTurnedOff`. El disparador de la cabecera no se alcanza con un `UPDATE` directo: el disparador fiscal lo rechaza antes | ✅ |
| PRC-R40 | En la moneda funcional de la empresa, tasa 1 y el día de la factura como día de la tasa, sin consultar el catálogo | D-169 | Servicio: `PurchaseInvoiceRates`. Dominio: constructor. Frontend: tasa fija «1 · moneda de la empresa» | `PurchaseInvoiceTests.inTheFunctionalCurrencyTheRateIsOneOfItsOwnDate`; `PurchaseInvoicesIntegrationTests.registeringWithoutCreateVehicleBringsTheUnitsToLifeAndAnInvoiceInSolesKeepsRateOneAndItsDate`, que desde `7984202f` lee también en la base que la tasa guardada es 1, no solo su día, y que otra tasa en la moneda funcional responde `422 PURCHASEINVOICE_EXCHANGE_RATE_CHANGED` con 1 | ✅ |
| PRC-R41 | En otra moneda, la tasa es la del país de la empresa, del tipo de compra del país, la más reciente de la fecha o anterior, y la factura la copia; la tasa vista viaja y un cambio responde `422` | D-151, D-152, A19 | `location`: `fetchEffectiveRate`. Servicio: `PurchaseInvoiceRates`. Frontend: `LocationFacade.loadPurchaseRate`; sin los países leídos no adivina el tipo de tasa ni lee ninguna (frontend `add2c336`) | `ExchangeRateRepositoryImplTests.theEffectiveRateIsTheLatestOnOrBeforeTheDay`; `PurchaseInvoicesIntegrationTests.aSaturdayCopiesTheRateOfFriday`, `aRateRegisteredWhileTheFormWasOpenAnswersExchangeRateChanged`; `purchase-invoice-form-view.spec` ("proposes the rate of the country in another currency, and says an earlier day of a business day", "says the session does not read the rates, without a retry, when their read is refused (catalogos.ver)", "never guesses the kind of rate: without the countries it reads none, says so and reads the countries again") | ✅ |
| PRC-R42 | La tasa de impuesto se guarda en cada factura; la del país solo se propone | D-24, §9.3 | Dominio: `TaxRate`. Frontend: propone `taxRate` del país | `PurchaseInvoicesIntegrationTests.theTaxRateIsTheOneOfTheInvoiceNotOfTheCountry` | ✅ |
| PRC-R43 | La unidad nace con la factura por `FleetContextFacade`, en el estado inicial por «Registro de compra», sin ubicación, firmada por la cuenta de la sesión; registrar la factura basta, sin `stock.crear_vehiculo` | D-153, D-160 | `fleet`: `PurchasedUnitRegistrations` sobre `VehicleRegistrations`, `MANDATORY`. `procurement`: `PurchaseInvoiceUnits` | `FleetContextFacadeImplTests.aNewVinIsBornInTheInitialStatusWithoutLocation`; `PurchaseInvoicesIntegrationTests.registeringWithoutCreateVehicleBringsTheUnitsToLifeAndAnInvoiceInSolesKeepsRateOneAndItsDate` | ✅ |
| PRC-R44 | Un VIN existente se vincula solo si es de la misma empresa, no está anulado y no tiene otra línea vigente; su modelo, su año y su marca mandan, y una línea que diga otros se rechaza. Un VIN de otra empresa responde `409` sin sus datos | D-160 | `fleet`: `matchPurchasedUnits` (una elevación, solo «es de otra»). `procurement`: `PurchaseInvoiceUnits` | `FleetContextFacadeImplTests.anExistingUnitOfTheCompanyIsLinkedAndOneOfAnotherCompanyIsOnlyForeign`; `PurchaseInvoicesIntegrationTests.aVinOfAnotherCompanyAnswersConflictWithoutItsData`, `anAnnulledUnitIsNotLinked`, `aLineThatSaysAnotherYearOfALinkedUnitIsRefused`, `anExistingUnitWithoutCurrentLineIsLinkedAndKeepsItsColorAndEngine` | ✅ |
| PRC-R45 | Una unidad aparece una vez en un comprobante y está en una sola línea vigente | §9.4, D-160 | Dominio: constructor. Base: `purchase_invoice_items_invoice_vehicle_uq`, `purchase_invoice_items_vehicle_current_uq`. Adaptador: `NaturalKey.PURCHASE_INVOICE_UNIT` → `409` | `PurchaseInvoiceTests.aUnitAppearsOnceInAVoucher`; `PurchaseInvoiceSchemaTests.aUnitIsInOneCurrentLineAndALineNamesAUnitOfItsCompany`; `PurchaseInvoiceLockTests.twoInvoicesLinkingTheSameUnitLeaveOneLine`; `PurchaseInvoicesIntegrationTests.theAmountsTheVinsAndTheFileAreCheckedBeforeAnythingIsWritten` | ✅ |
| PRC-R46 | Cada línea copia la empresa, el modelo, el año y la marca de su unidad | D-160 | Dominio: `PurchaseInvoiceItem`. Base: llaves compuestas con `company_id` hacia la factura, la unidad y la marca | `ProcurementCompanyIsolationIntegrationTests.aLineCannotNameAUnitOfAnotherCompany`; `PurchaseInvoiceSchemaTests.aUnitIsInOneCurrentLineAndALineNamesAUnitOfItsCompany` | ✅ |
| PRC-R47 | Registrar exige actuar con `compras.registrar_factura` en toda la empresa y en la marca de cada unidad, sin sede. Desde el 04/11/2026 la sesión dice ese alcance por empresa (`wholeCompanyPermissions`, `brandPermissions`), y el formulario ofrece solo las empresas donde se registra sin sede y retiene las líneas de una marca que el permiso no alcanza (C8) | D-163, D-101, D-102 | `PurchaseInvoiceAccessPolicy.registers`. Sesión: `SessionCompanyView` (backend `df6b7276`). Frontend: `IamFacade.actsOnWholeCompany` (frontend `3742271f`); el formulario (`fdf14036`, `18e03774`, `5fad3ceb`) | `PurchaseInvoiceAccessPolicyTests.aBranchGrantDoesNotRegister`, `aBrandGrantRegistersOnlyItsBrand`; `PurchaseInvoicesIntegrationTests.aLineOfABrandTheCallerDoesNotActOnAnswersAccessDenied`, `withoutRegisterInvoicesAnswersAccessDeniedAndAnOldInvoiceNeedsItsPermission`; `AuthenticatedUserIntegrationTests.theSessionTellsTheReachOfEachPermissionInACompany`, `aSystemRoleActsOnTheWholeOfEveryCompany`; `purchase-invoice-form-view.spec` ("holds back the lines of a brand the registration does not reach, by the names of their brands (C8)", "says it when no company is one where the registration acts over the whole company (C8)" y, desde la segunda revisión, frontend `cbf35bf6`, "holds back a line that links a unit of a brand the registration does not reach, by the brand of the unit (C8)", "offers as the default commercial brand only the brands where the session registers invoices (C8)"); e2e ("a role of a branch never offers its company: the permission acts on no whole company", "a session without the scopes falls back to the permissions of the company, and the backend decides") | ✅ |
| PRC-R48 | Las facturas se leen por empresa con `compras.ver`; escribir exige leer | D-163, D-134 | `PurchaseInvoiceQueryServiceImpl` (`companyReach`), servicio de comandos | `PurchaseInvoicesIntegrationTests.aWriterWithoutVerDoesNotReadNorWriteTheInvoicesOfACompany`; `PurchaseInvoiceReadsIntegrationTests.withoutViewTheReadsAreDenied` | ✅ |
| PRC-R49 | Una factura se vincula a lo sumo a una orden enviada de su misma empresa y su mismo proveedor; desvincular se puede siempre | D-162, D-172 | Base: `purchase_invoices_order_supplier_company_fk`. Servicio: `PurchaseInvoiceOrderLinks`, con la orden `FOR NO KEY UPDATE` (§2.4). Frontend: las órdenes enviadas del proveedor, buscadas en el servidor por número, de la más reciente a la más antigua (frontend `9f011f06`) | `PurchaseInvoicesIntegrationTests.onlyASentOrderOfTheSameSupplierIsLinked`, `unlinkingIsAlwaysPossibleAndAFulfilledOrderStaysFulfilled`; `purchase-orders.store.spec` ("searches the sent orders of a supplier on the server, the newest first, a page at a time"); `purchase-invoice-form-view.spec` ("searches the sent orders of the supplier on the server, and says only an empty answer to no text means none"); `edit-purchase-invoice-dialog.spec` ("searches the sent orders on the server, and keeps the order linked now first, sent or not"); e2e ("finds a sent order past the newest fifty by its number, searched on the server") | ✅ |
| PRC-R50 | Facturar un modelo o un año que la orden no pide, o más de lo pendiente, da un aviso que se confirma; el precio no se compara. Lo que la factura trae de varios años a una línea de la orden sin año se suma antes de comparar (desde el 04/11/2026) | D-162 | Dominio: `OrderInvoicingPolicy` (backend `10850e34`). Servicio: `PurchaseInvoiceOrderLinks`. Frontend: diálogo de desviaciones | `OrderInvoicingPolicyTests.aModelTheOrderDoesNotAskForIsADeviation`, `moreThanPendingIsADeviation`, `aLineWithoutYearCountsForTheYearsNoOtherLineNames`, `theYearsAnInvoiceBringsIntoALineWithoutYearAreAddedUp`; `PurchaseInvoicesIntegrationTests.aDeviationIsSavedOnlyOnceConfirmed`; `purchase-invoice-form-view.spec` ("asks about the deviations from the order and sends the same registration confirmed") | ✅ |
| PRC-R51 | Lo pendiente de una orden se cuenta por modelo y año con las facturas vigentes vinculadas; la orden muestra «todo facturado» y se sigue cumpliendo a mano | D-162 | `PurchaseOrderInvoicingReader`, `OrderInvoicingPolicy`. Frontend: ficha de la orden | `PurchaseOrderInvoicesIntegrationTests.theOrderTellsWhatItStillLacksByModelAndYear`, `anAnnulledInvoiceGivesItsUnitsBackToThePending`, `anOrderWithoutLinesOfVehicleHasNoInvoicing` | ✅ |
| PRC-R52 | Una orden con facturas vigentes no se anula | D-162 | `PurchaseOrderCommandServiceImpl` (anulación, bajo el candado de la orden); `PurchaseOrderAllowedActions` | `PurchaseOrderInvoicesIntegrationTests.anOrderWithACurrentInvoiceIsNotAnnulledAndTheOrdersAreFilteredBySupplier`; `PurchaseOrderLockTests.anAnnulmentAndALinkOfTheSameOrderDoNotCross` | ✅ |
| PRC-R53 | Lo fiscal no se edita: solo cambian el archivo y la orden, con `compras.registrar_factura`; una factura anulada ya no cambia | D-161, D-172 | Dominio: `relinkOrder`, `replaceFile`. Base: `purchase_invoices_fiscal_tg`, `purchase_invoice_items_fiscal_tg` | `PurchaseInvoiceFiscalTriggerTests.theDatabaseRefusesToChangeTheTotalOfAnInvoice`, `everyOtherFiscalColumnIsRefusedAndTheOrderTheFileAndTheXmlChange`, `anAnnulledInvoiceNeverChanges`, `aLineIsNeverEditedAndAnAnnulledLineKeepsItsMoment`, `aLineComesBackOnlyWithValueAndNeverOnAnAnnulledInvoice` (las dos últimas, con esos nombres desde la fase 4, que deja volver a vigente una línea con valor, D-193); `PurchaseInvoiceChangesIntegrationTests.theFileAndTheOrderChangeAndNothingElse`; `PurchaseInvoicesIntegrationTests.theSameOrderOfAFulfilledOrderStaysLinkedWhileTheFileChanges`, `switchingToAnotherOrderNotSentIsRefused`; `edit-purchase-invoice-dialog.spec` ("after PURCHASEINVOICE_OUTDATED takes the order and the updatedAt read again, and keeps the file picked") | ✅ |
| PRC-R54 | Anular exige `compras.anular_factura` y un motivo, guarda quién y cuándo, es de toda la factura —sus líneas pasan a anuladas— y procede solo sin dependientes: desde las fases 4 y 5, una nota de crédito vigente o un pago aplicado no anulado | D-161, H3 | Dominio: `annul`. Servicio: `PurchaseInvoiceAnnulments` con `List<PurchaseInvoiceDependents>`: `CreditNoteInvoiceDependentsAdapter` y `PaymentInvoiceDependentsAdapter`. Base: `purchase_invoices_annulment_ck` | `PurchaseInvoiceTests.annullingMarksEveryLineAnnulledOnce`; `PurchaseInvoiceDependentsTests.creditNotesAndPaymentsAreTheDependents`, `anInvoiceWithAPaymentHasDependents`; `PurchaseInvoiceAnnulmentsIntegrationTests.anInvoiceWithACurrentNoteAnswersHasDependents`, `anInvoiceWhoseNoteWasAnnulledIsAnnulled`; `PurchaseInvoicesIntegrationTests.annullingNeedsItsPermissionAndAReason`; `PurchaseInvoiceAllowedActionsTests.anInvoiceWithDependentsIsNotAnnulled` | ✅ |
| PRC-R55 | Anular la factura no anula sus unidades: quedan sin factura vigente y se vinculan por su VIN a la que la reemplaza | D-161 | Servicio: ninguna llamada a `fleet` en la anulación | `PurchaseInvoiceAnnulmentsIntegrationTests.anAnnulledInvoiceLeavesItsUnitsWithoutCurrentInvoiceTheChangeLogKeepsItsReasonAndTheNewOneLinksThem` | ✅ |
| PRC-R56 | La anulación deja su fila delicada en la bitácora, con el motivo | D-175, §13.1 | `PurchaseInvoiceAnnulments` con `ChangeLogCommandService.recordChange` en la misma transacción | `PurchaseInvoiceAnnulmentsTests.theAnnulmentLeavesItsDelicateRowWithItsReasonAndItsMoment`; `PurchaseInvoiceAnnulmentsIntegrationTests.anAnnulledInvoiceLeavesItsUnitsWithoutCurrentInvoiceTheChangeLogKeepsItsReasonAndTheNewOneLinksThem`, que desde `7984202f` lee también que la fila no tiene valor anterior y que su valor nuevo es el momento que guarda la factura | ✅ |
| PRC-R57 | El archivo es PDF, PNG, JPEG o WebP de hasta 5 MB, verificado por su contenido; el XML, bien formado y de hasta 2 MB. Se guardan en `kept_files` con la empresa de la factura, después de validar todo, y se descartan si la transacción se revierte, solo cuando la transacción escribió su llave primero: una llave que la factura confirmada ya apunta nunca se descarta (desde el 04/11/2026, §2.5.5) | D-174, §2.7, E15 | Dominio: `UploadedFileType`. Servicio: `PurchaseInvoiceFiles`, `XmlWellFormedness`. `documents`: `KeptFileStore.put`, que dice si escribió la llave primero, y `discardRolledBack` (backend `7ad20a4a`); en Spaces, la fila antes que el objeto (backend `5df515b8`, segunda revisión) | `PurchaseInvoiceUploadLimitsTests.aPdfRenamedPngIsReadAsAPdfAndAnHtmlIsRefused`, `aFileOverFiveMegabytesAnswersFileSize`, `anXmlThatIsNotWellFormedAnswersXmlInvalid`, `anXmlOverTwoMegabytesAnswersXmlSize`; `PurchaseInvoiceFilesTests.aRolledBackRegistrationDiscardsItsObject`, `theSameBytesSentAgainAreNeverDiscarded`, `aFormerFileSentBackIsNeverDiscarded`, `aCommittedRegistrationKeepsItsObject`; `KeptFileStoreUpsertTests.theDatabaseStoreTellsWhetherAPutWroteTheKeyFirst`, `theSpacesStoreTellsWhetherAPutWroteTheKeyFirst`, `aKeyAnEarlierTransactionCommittedIsNotWrittenFirstAgain`; `SpacesKeptFileStoreTests.aPutTellsWhetherItsUpsertInsertedTheRow`, `theObjectOfARolledBackKeyIsDeleted`, `aRowThatFailsToWriteLeavesNoObject`, `theObjectIsWrittenAfterItsRowAndItsFailureFailsThePut`; `PurchaseInvoiceRollbackTests.aFailureOnLineOneHundredLeavesNoRowAndNoFile` | ✅ |
| PRC-R58 | La exportación lleva las 23 columnas de §4.2.2, con la base, el IGV y el total en la moneda de la empresa rotulados «(calculado)»: en una factura, la base y el IGV salen de la tasa; en una boleta u otro, la base es el total y el IGV es 0. Tiene el alcance de `compras.ver` | D-163, D-159, D-180, M11, D-106 | `PurchaseInvoicesController`, `PurchaseInvoiceSpreadsheetColumns`, `ProcurementDocumentKinds` | `PurchaseInvoicesExportIntegrationTests.theListIsExportedWithItsColumnsAndTheCalculatedBaseAndTax` (también que un permiso de registrar en otra empresa no la trae al archivo), `aTypeOutOfItsListIsRefusedBeforeAnyFile`, `theTotalInTheCurrencyOfTheCompanyRoundsHalfUp` (desde `7984202f`: USD 1,00 a 3,745 da S/ 3,75, mitad hacia arriba); `PurchaseInvoiceTests.theBaseAndTheTaxAreCalculatedOnlyForAFactura`; `purchase-invoice-amounts.spec` ("takes the total as the base of a boleta or another document, with no tax (D-180)") | ✅ |
| PRC-R59 | `allowedActions` de la factura sale en el orden `edit`, `annul`, `annulAndCopy`, `downloadFile`, `downloadXml`, con las reglas de cada operación | D-115 | `PurchaseInvoiceAllowedActions` | `PurchaseInvoiceAllowedActionsTests.theActionsComeInTheOrderOfTheSpecification`, `anAnnulledInvoiceOffersOnlyItsDownloads`, `annullingWithoutRegisteringOffersNoCopyAndWithoutViewNothingIsOffered`; `PurchaseInvoiceReadsIntegrationTests.theRecordSaysWhatTheSessionMayDoAndAnInvoiceOfAnotherCompanyIsNotFound` | ✅ |
| PRC-R60 | Las escrituras de facturas toman los candados en el orden de §2.4, y una carrera que la base detiene responde el código de su llave | Esta especificación, §2.4 | Servicio y adaptadores | `PurchaseInvoiceLockTests.twoInvoicesWithTheSameNewVinLeaveOneUnitAndNoOrphan`, `aSupplierDeactivatedMeanwhileIsSeenByTheRegistration`, `twoRegistrationsOfTheSameNumberLeaveOneInvoice`, `twoInvoicesLinkingTheSameUnitLeaveOneLine`, `anAnnulmentAndAChangeOfTheSameInvoiceDoNotCross`; `PurchaseOrderLockTests.anAnnulmentAndALinkOfTheSameOrderDoNotCross`; `VehicleLockTests.aYearEditAndALinkOfTheSameUnitDoNotCross`; `VehicleAnnulmentLockTests.anAssignmentAndAnAnnulmentOfTheSameUnitDoNotCross` | ✅ |
| PRC-R61 | Una factura en una moneda, de cualquier estado, fija sus decimales | §4.1, D-138 | `PurchaseInvoiceCurrencyUsageAdapter` | `CurrencyDecimalPlacesLockTests.anInvoiceFixesTheDecimalPlacesOfItsCurrency` | ✅ |
| PRC-R62 | Compras responde a `fleet` si una unidad tiene línea vigente, si su valor es cero y qué unidades tienen línea vigente, con su versión | D-160, §9.6, D-175 | `PurchaseInvoiceUnitPurchasesAdapter` | `PurchaseInvoiceUnitPurchasesAdapterTests.aUnitOfAnAnnulledInvoiceHasNoCurrentLineAndZeroValue` | ✅ |
| PRC-R63 | Las dos tablas llevan las tres políticas de V83, la cuenta de la aplicación no borra, y las dos se versionan en `data_versions` | §2.5, V83, D-128 | Base: V(n+2) | `ProcurementCompanyIsolationIntegrationTests.theInvoicesOfAnotherCompanyAreNotSeen`; `ProcurementRowsAreNeverDeletedTests` (las dos tablas); `JdbcDataVersionsTests.everyVersionedTableHasItsSixtyFourSlotsAndItsTwoTriggers` | ✅ |
| PRC-R64 | Los tres permisos de facturas se siembran solo al superadministrador hasta las notas de crédito, y desde V104 (fase 4) siguen la matriz; `stock.anular_vehiculo`, como la matriz desde el principio | D-176, §5.6 | Base: V92 y V104. `PermissionCodes` | `InvoicePermissionsMatrixMigrationTests.theInvoicePermissionsFollowTheMatrixOnceCreditNotesExist`; `PurchaseInvoicePermissionsSeedMigrationTests.theAnnulmentOfAUnitFollowsTheMatrixFromNow`; `PermissionCatalogConsistencyTests`. La prueba de «solo el superadministrador» se retiró con V104 (backend `9137fc62`) | ✅ |
| PRC-R65 | Una factura de 100 líneas se registra en una transacción; las sentencias que Hibernate prepara, además de las inserciones, no crecen con las líneas, también cuando las líneas vinculan unidades existentes (desde el 04/11/2026). Las escrituras de `fleet` y de las versiones por JDBC no las cuenta Hibernate | Plan §5, A4, A5 | `PurchaseInvoiceCommandServiceImpl`, `PurchasedUnitRegistrations`, `VehicleRepositoryImpl.readAllLocked` (backend `61ebde79`) | `PurchaseInvoiceRegistrationStatementsTests.theQueriesOfARegistrationDoNotGrowWithItsLines`, que fija 64 sentencias además de las inserciones con 1, 10 y 100 líneas nuevas (las 57 de la fase 2, 3 de las notas y 4 de los pagos), `theQueriesOfARegistrationThatLinksExistingUnitsDoNotGrowWithItsLines`, con 1, 10 y 100 líneas que vinculan; `PurchaseInvoiceRollbackTests.aFailureOnLineOneHundredLeavesNoRowAndNoFile`, que falla en una transacción real después de escribir las 100 unidades, su historial, la factura y su archivo, y no deja nada; desde la segunda revisión registra sobre la escena de las carreras (`InvoiceRaces.root`, backend `607e6756`), sin confirmar un país ni una empresa nuevos en cada corrida (§10.4); `PurchaseInvoicesIntegrationTests.aRefusalOnLineOneHundredIsAnsweredBeforeAnythingIsWritten` | ✅ |
| PRC-R66 | «Anular y copiar» anula y abre el formulario con la copia: proveedor, tipo, número, fecha, moneda, tasa de impuesto, total, ISC, orden, archivos y líneas; la tasa se vuelve a proponer, no se copia | D-161, D-152 | Frontend: `PurchaseInvoiceFormView` con `?copiaDe=`; la orden de la copia se lee por su id, aunque no esté en la primera página, y se vuelve a vincular solo si sigue enviada (frontend `9f011f06`) | `purchase-invoice-form-view.spec` ("copies an annulled invoice and proposes its rate again", "links again the order a copy names, read by its id past the first page, only while it is still sent", "never links again an order of the copy that is no longer sent"); e2e ("annuls an invoice with its reason, and annuls and copies another proposing its rate again") | ✅ |
| PRC-R67 | Una línea que vincula una unidad existente no toma el color ni el motor de lo escrito o pegado: la unidad conserva los suyos y el formulario los muestra bloqueados. Si difieren de los de la unidad, o la unidad no tiene motor, la línea lleva un aviso que no bloquea y dice que conserva los datos de la unidad, para que se corrija en Inventario | D-179 | Backend: `PurchaseInvoiceUnits` y `PurchasedUnitRegistrations` ignoran el motor y el color de una línea que vincula, y no resuelven su color. Frontend: `purchase-invoice-lines`, con `vin-check` | `PurchaseInvoicesIntegrationTests.aLinkedUnitKeepsItsColorAndEngineWhateverTheLineSays`, `anExistingUnitWithoutCurrentLineIsLinkedAndKeepsItsColorAndEngine`; `purchase-invoice-lines.spec` ("warns without blocking when the engine or the color differs from the linked unit", "warns when the linked unit has no engine", "shows no warning without the unit"); e2e ("a linked unit keeps its engine when another is pasted, with a notice that does not block", que desde el contrato C10 se llama "a linked unit keeps its engine when the workbook brings another, with a notice that does not block", frontend `4e7d440b`) | ✅ |
| PRC-R155 | La plantilla de las líneas es un documento generado: la piden quienes tienen `compras.registrar_factura`, un pedido igual la reusa y un cambio del catálogo de vehículos genera otra. Lista los modelos que una unidad nueva puede tomar, las marcas y los colores activos, dice qué columnas son obligatorias y trae un ejemplo (contrato C10, 04/11/2026; 05/11/2026) | D-106, D-108; decisión del responsable del 04/11/2026 | `PurchaseInvoiceLinesController.template`, `ProcurementDocumentKinds` (`purchase-invoice-line-template`), `FastexcelInvoiceLineWorkbooks.template`; `vehiclecatalog`: `fetchActiveVehicleModelSummaries`, `fetchActiveColorSummaries`. Commits: backend `6eee6878`, `d3b1e6ae`, `23fb6bba`; frontend `3d5387b9` | `PurchaseInvoiceLinesIntegrationTests.theTemplateHasItsSheetsItsHeaderItsFormatsAndTheActiveCatalog`, `anEqualRequestReusesTheTemplateAndAChangeOfTheCatalogGeneratesAnother`, `theTemplateAsDownloadedImportsWithNothing`, `bothNeedTheRegistrationOfInvoices`; `purchase-invoices-api-endpoint.spec` ("downloads the template of the lines as a generated document, as the Excel of a list (C10, D-106)") | ✅ |
| PRC-R156 | La importación rechaza el archivo entero en este orden: sin archivo, `400`; no es un `.xlsx` por sus bytes, o no se lee o se infla a más de 32 MB, `PURCHASEINVOICE_IMPORT_FILE_TYPE`; más de 2 MB, `_FILE_SIZE`; otra cabecera, `_HEADERS` (se admite sin ISC; una columna de más, no); más de 100 filas que no están en blanco, `_ROWS` | C10 (§11.2) | `PurchaseInvoiceLineImportQueryServiceImpl`, `InvoiceLineImportPolicy.headerMatches`, `FastexcelInvoiceLineWorkbooks.looksLikeWorkbook`, `inflatesWithin`. Commits: backend `d3b1e6ae` | `PurchaseInvoiceLinesIntegrationTests.theFileIsRefusedInTheOrderOfTheContract`, `aHundredRowsAreReadAndAHundredAndOneRefused`, `aWorkbookWithoutTheIscColumnIsRead`; `InvoiceLineImportPolicyTests.theHeaderIsTheOneOfTheTemplateWithoutCaseNorAccentsAndIscMayBeLeftOut` | ✅ |
| PRC-R157 | Cada fila que no está en blanco da su línea o sus errores, nunca las dos: los nombres se resuelven sin mayúsculas, tildes ni espacios de más contra los colores activos, las marcas y los modelos activos de cada marca; cada error lleva su fila, su columna, su código y el texto de su celda. Nada se guarda: no es una carga de archivos | C10, D-108, D-200, D-208 | `InvoiceLineImportPolicy.read`, `fold`. Frontend: `import-invoice-lines-dialog`, `purchase-invoice-lines` (`4e7d440b`) | `InvoiceLineImportPolicyTests` (8: `aRowResolvesItsNamesWithoutCaseNorAccentsAndReadsItsNumbers`, `anAmountMayComeAsTextWithADotAndTheEngineAndTheIscMayBeEmpty`, `everyRequiredCellOfAnEmptyRowIsNamedInTheOrderOfTheColumns`, `eachCellThatDoesNotHoldWhatItMustIsNamedWithItsText`, `aBrandNotFoundLeavesItsModelUnjudgedAndAModelOfAnotherBrandIsNotFound`, `aYearOrAnAmountOutOfItsRuleIsInvalid`, `aVinRepeatedIsNamedOnTheLaterRowAndTheIscOfASheetWithoutItIsNotRead` y la de la cabecera); `PurchaseInvoiceLinesIntegrationTests.aFilledWorkbookGivesItsLinesResolvedAndSkipsItsBlankRows`, `eachErrorOfARowIsNamedAndTheRowGivesNoLine`; `import-invoice-lines-dialog.spec`; e2e (§11.4) | ✅ |

### 5.2 `fleet`

Continúan la numeración de [contextos/fleet.md](contextos/fleet.md), cuya última es FLT-R47.

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| FLT-R48 | Una unidad se anula por acción explícita con `stock.anular_vehiculo`, solo si su valor es cero, no tiene contratos y no tiene una cesión abierta (FLT-R54); mientras ventas no exista, ninguna tiene contratos | D-104, D-161, §9.6 | Dominio: `Vehicle.annul`. Servicio: `VehicleAnnulments` con `UnitPurchaseFacts` y `UnitContractFacts`. Ruta: `POST /vehicles/{id}/annulment` | `VehicleTests.aUnitIsAnnulledOnce`; `VehicleAnnulmentsIntegrationTests.aUnitWithACurrentLineIsNotAnnulled`, `aUnitWithoutCurrentLineIsAnnulled`; `UnitPurchaseFactsTests.untilSalesExistsNoUnitHasAContract`, `withoutBeansNoUnitHasALineAndEveryValueIsZero`, `theBeansAreGatheredByAnyUnionAndAll` | ✅ |
| FLT-R49 | Una unidad anulada no admite ninguna escritura, sin excepción —tampoco ceder ni cerrar una cesión—, y su `allowedActions` no ofrece ninguna; el regreso automático la salta | D-104, D-182 | `VehicleCommandServiceImpl`, `VehicleMileageReadingCommandServiceImpl`, `VehicleFundAssignmentCommandServiceImpl`, `VehicleMovements`, `VehicleAllowedActions`, `VehicleFundAssignmentViewFactory` | `VehicleAnnulmentsIntegrationTests.everyWriteOfAnAnnulledUnitAnswersVehicleAnnulledAndItsActionsOfferNone`, también ceder la unidad y, desde `7b92a7de`, el cambio de estado, la placa y el cierre de una cesión abierta antes de la anulación; `vehicle-profile-view.spec` ("annuls a unit after asking, only as its allowed actions offer it, and says a refusal (D-104, D-182)") | ✅ |
| FLT-R50 | Una unidad con línea de factura vigente no cambia de año, y su modelo ya no cambia por ninguna ruta | D-160 | `VehicleCommandServiceImpl` (edición), bajo el candado de la unidad, con `UnitPurchaseFacts.hasCurrentLine`. Frontend: el diálogo de la unidad deja fijo el año de una unidad con `hasCurrentInvoice` | `VehiclesIntegrationTests.theYearOfAnInvoicedUnitDoesNotChange`; `VehicleInvoiceFactsIntegrationTests.theYearOfAUnitWithACurrentLineDoesNotChangeAndTheRestDoes`; `VehicleLockTests.aYearEditAndALinkOfTheSameUnitDoNotCross`; `vehicle-form-dialog.spec` ("fixes the year of a unit in a current invoice and says why, sending the year it has (D-160)", "leaves the year of a unit without a current invoice to correct") | ✅ |
| FLT-R51 | Inventario muestra y filtra las unidades sin factura vigente, también en su exportación | D-175 | `VehicleQueryServiceImpl` (`hasCurrentInvoice`), `FleetDocumentKinds`. Frontend: insignia y filtro. `VehicleSearchFilter` lleva las unidades con línea vigente (`unitsWithCurrentInvoice`), que responde `UnitPurchaseFacts` | `VehicleInvoiceFactsIntegrationTests.theInventoryFiltersTheUnitsWithoutCurrentInvoice`; `VehicleInventoryExportTests.theFileSaysAndFiltersWhichUnitsHaveACurrentInvoice` (desde `7b92a7de`: la columna dice Sí y No, y el filtro deja en el archivo solo las unidades pedidas); `vehicles-view.spec` ("says which units have no current invoice and filters them"); e2e ("Inventario flags and filters the units without a current invoice, and a unit is annulled from its record") | ✅ |
| FLT-R52 | El alta y el vínculo por lote validan todo antes de escribir, exigen una transacción abierta, elevan el alcance una vez por llamada y bloquean las unidades existentes por `id`; desde el 04/11/2026 las leen en una sentencia después del candado | Plan 5.3, A4, A5 | `FleetContextFacadeImpl`, `PurchasedUnitRegistrations`, `VehicleRepositoryImpl.readAllLocked` (backend `61ebde79`) | `FleetContextFacadeTransactionTests.registeringOutsideATransactionFails` (desde `1ac94658`, en su propia clase, sin confirmar filas); `FleetContextFacadeImplTests.aRefusedLineWritesNoUnit`, `oneElevationPerCallWhateverTheLines`, `aRepeatedOrMalformedVinIsAValidationErrorOfItsLines`, `theUnitsOfTheLinesAreReadInOneBatch`; `PurchaseInvoiceRegistrationStatementsTests.theQueriesOfARegistrationThatLinksExistingUnitsDoNotGrowWithItsLines`; desde la segunda revisión (backend `eed0c204`), `FleetContextFacadeLockTests.aUnitReadBeforeTheLockAndChangedMeanwhileIsReadAgainOnceLocked`, con dos transacciones reales: una unidad leída antes del candado y cambiada por otra transacción confirmada se vuelve a leer una vez bloqueada | ✅ |
| FLT-R53 | `fleet` no depende de `procurement`: lo que necesita de compras lo pregunta por `UnitPurchases`, y exactamente un bean lo responde | Mapa §5, regla 1.1 | `fleet/interfaces/acl`, `UnitPurchaseFacts` | `ModularityTests`; `UnitPurchasesWiringTests.procurementIsTheOnlyAnswerOfThePurchasesOfAUnit` | ✅ |
| FLT-R54 | Una unidad con una cesión abierta a un fondo, sin fecha de fin, no se anula: primero se cierra la cesión. La cesión se lee bajo el candado de la unidad, el mismo que toman ceder y cerrar | D-182, §12.2 | Servicio: `VehicleAnnulments` con `VehicleFundAssignmentRepository.existsOpen`, `422 VEHICLE_FUND_ASSIGNMENT_OPEN`; `VehicleAllowedActions` no ofrece `annul`. Frontend: «Anular unidad» no se ofrece | `VehicleAnnulmentsIntegrationTests.aUnitWithAnOpenFundAssignmentIsNotAnnulledAndOneWhoseAssignmentWasClosedIs`, también `allowedActions.annul` en falso; `VehicleAnnulmentLockTests.anAssignmentAndAnAnnulmentOfTheSameUnitDoNotCross`; `vehicle-profile-view.spec` ("annuls a unit after asking, only as its allowed actions offer it, and says a refusal (D-104, D-182)"); e2e ("an open assignment to a fund keeps a unit from annulment, and its refusal says to close it first") | ✅ |

### 5.3 `location`

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| LOC-R53 | Cada país dice qué tipo de tasa propone para sus compras; Perú, `venta` | D-152 | Base: V(n+1). `Country`, `CountrySummary`, `CountryResource` | `CountriesControllerTests.peruProposesTheSellRateForItsPurchases`, `aNewCountryProposesTheOfficialRateForItsPurchases` | ✅ |
| LOC-R54 | La fachada da la tasa vigente de un día: la más reciente de ese día o anterior | D-151 | `LocationContextFacadeImpl.fetchEffectiveRate`, `ExchangeRateRepository.findEffective`. La fachada la lee por `EffectiveExchangeRateQueryService`, para no formar un ciclo de beans | `LocationContextFacadeImplTest.theEffectiveRateOfASundayIsTheOneOfFriday`, `aTypeOutsideTheListHasNoEffectiveRate` | ✅ |

---

## 6. Pantallas

### 6.1 Rutas y menú

| Ruta del navegador | Vista | `data.permissions` | Encabezado |
|---|---|---|---|
| `/procurement/invoices` | `PurchaseInvoicesView` | `anyOf: [compras.ver]` | El del módulo, `modules.invoices` (ya existe) |
| `/procurement/invoices/new` (y `?copiaDe={id}`) | `PurchaseInvoiceFormView` | `allOf: [compras.ver]`, `anyOf: [compras.registrar_factura]` | `purchase-invoices.heading.new-title` y `new-description` |
| `/procurement/invoices/{invoiceId}` | `PurchaseInvoiceProfileView` | `anyOf: [compras.ver]` | El número y el proveedor, con `overridePageHeading` |

En `app-navigation.ts`, la opción `/procurement/invoices` pierde `pending: true` y declara `access: anyOf(PermissionCodes.Procurement.VIEW)`. `PermissionCodes` del frontend (`iam/application/permission-codes.ts`) suma `REGISTER_INVOICES`, `REGISTER_OLD_INVOICES`, `ANNUL_INVOICES` y `Stock.ANNUL_VEHICLE`.

### 6.2 Lista

- **Acciones** (`.mm-page-actions`): «Registrar factura», solo con `hasPermission(REGISTER_INVOICES)`, y «Exportar».
- **Filtros:** empresa (`mm-company-filter`), proveedor (`mm-search-select` con los proveedores de la empresa, activos e inactivos; desde el 04/11/2026, buscados en el servidor por razón social o documento, una página a la vez, y el elegido sigue nombrado aunque la búsqueda siguiente no lo traiga, frontend `3a995e95` sobre `6c779bc3`), estado (Todas, Vigentes, Anuladas) y período Desde–Hasta: Desde solo es ese día; Desde y Hasta, el intervalo completo, con los dos días; Hasta solo, todo hasta ese día (`dayPeriod`; la pantalla manda `to` como el día siguiente).

  **Como se corrigió (04/11/2026, segunda revisión): la barra de una lista con varios filtros** (pedido del responsable; frontend `83a83cce`, `71ccb43c`, `bebbab4f`; revisión 40 de `DESIGN.md`, `89f9f6c5`). Con seis filtros, la barra de una sola fila empujaba los filtros al final con un hueco antes, dejaba las acciones solas en una tercera fila y mostraba la etiqueta de un filtro vacío dentro de la caja, como un texto de ayuda. La lista toma la variante `filterRows` de `mm-data-manager`, con el orden de las revisiones 42 y 44 (`58d55bd8`, `6721267e`), también por pedido del responsable:
  - **Fila de arriba:** la búsqueda y los filtros, que llenan el ancho.
  - **Los filtros,** en el orden de las listas de compras: Empresa (si la sesión llega a más de una), Proveedor (`.mm-filter--wide`) y Estado (`--narrow`). Desde el 09/11/2026, Tipo (`--short`) después de Proveedor y Pago (`.mm-filter`) después de Estado (D-244, abajo), y desde la revisión 64, Proveedor en `.mm-filter--medium` y Estado en `--short` (abajo).
  - **Fila de abajo:** ~~Desde y Hasta (`--day`) y «Limpiar filtros» a la izquierda; a la derecha «Exportar», delineado, y «Registrar factura», relleno.~~ Desde el 09/11/2026 (revisión 64), Desde y Hasta (`--day`) siguen a los filtros, y las acciones cierran la barra: «Limpiar filtros», «Exportar», delineado, y «Registrar factura», relleno (abajo). Cada filtro muestra un valor: «Todos …» o «Todas …» con el valor `all`, y los días con la etiqueta flotando sobre la caja vacía (`floatLabel="always"`).
  - **«Limpiar filtros»** (`filter_alt_off`), solo mientras un filtro difiere de su valor inicial: vuelve los filtros atrás, no la búsqueda.
  - **En el teléfono:** la búsqueda, «Filtros» con la cuenta de los activos, que despliega los filtros y las fechas (`aria-expanded`, `aria-controls`), y las acciones, que comparten el ancho.
- **Columnas:**

  | Columna | Contenido | Ordena |
  |---|---|---|
  | Fecha | `dd/mm/aaaa` | Sí |
  | Comprobante | ~~Tipo y número; el número abre la ficha~~ Desde el 09/11/2026, solo el número, que abre la ficha, ~~y la insignia «Boleta» u «Otro» cuando no es una factura~~ y debajo el tipo de todo documento, Factura, Boleta u Otro, como texto; centrada entera (revisión 65, abajo) | Sí, por número |
  | Proveedor | Razón social y, debajo, el documento | Sí |
  | Total | Con su moneda | Sí |
  | Unidades | Cuántas; desde el 09/11/2026, bajo el encabezado «UND», centrada entera (revisión 64, abajo) | No |
  | Orden | Su número; desde el 09/11/2026, un enlace a la orden | No |
  | Estado | Insignia Vigente o Anulada | No |
  | Pago (desde el 09/11/2026) | Insignia del estado de pago, o «—» en una factura anulada (D-244) | No |

- **Menú de la fila:** ~~«Ver ficha»~~ «Ver detalles» y, desde el 09/11/2026, según las `allowedActions` de la fila, «Registrar pago», «Registrar nota de crédito» y «Descargar comprobante» (D-244, abajo).
- **Estados:** esqueleto (`mm-skeleton`) mientras carga; vacío, «Todavía no hay facturas registradas», con «Registrar factura» si se puede; error, con «Reintentar».

**Como cambió (09/11/2026, D-244): el pago de cada factura, su tipo y lo que ofrece su fila** (decisión del responsable; revisión 63 de `DESIGN.md`; frontend `8fa29b6e`, `0c59fd66` y `5700020a`, merge `cf862569`). El responsable revisó la lista con datos de demostración en `dev` y preguntó por qué dice «Factura» en cada fila. Hay tres tipos de comprobante, Factura, Boleta y Otro (D-159), y solo la factura da crédito fiscal; la serie «F…» ya lo dice, así que Comprobante muestra solo el número, en una línea, y la insignia neutra «Boleta» u «Otro» cuando no es una factura, que pasa debajo del número solo si la columna es angosta. Además:
- **La columna «Pago»** dice el estado de pago de la fila, con los tonos de la unidad: «Sin pago» y «Anulada por notas» en `neutral`, «Pago parcial» en `warning`, «Pagada» en `success` y «Saldada» en `info`.
- **Dos filtros:** Tipo (Todos, Factura, Boleta, Otro), con el ancho nuevo `.mm-filter--short` (7rem), y Pago (Todos y los cinco estados), que se mandan como `documentType` y `paymentStatus` a la lista y a su exportación. Cuentan entre los filtros activos y «Limpiar filtros» los suelta. Tipo vuelve a la lista: la revisión 44 lo había quitado (plan de compras y habilitación, §2.13). A 1440 px la búsqueda, Proveedor, Tipo, Estado y Pago quedan en una línea; en el teléfono, «Filtros» los pliega con los demás.
- **Lo que no se parte:** los días, los números, los montos y las insignias van en una línea; solo el proveedor puede partirse, y la tabla toma al menos 1120 px antes de desplazarse de lado en su región.
- **El menú de la fila** ofrece lo que sus `allowedActions` dicen: «Registrar pago» abre `/procurement/supplier-payments/new?factura=`, «Registrar nota de crédito» abre `/procurement/credit-notes/new?facturaId=`, y «Descargar comprobante» baja el archivo como la ficha.
- **Queda pendiente:** la lista no guarda sus filtros en la dirección, ni los dos nuevos ni los anteriores.

Pruebas: `purchase-invoices-view.spec` ("filters by the kind of document and the payment status, counts them, and «Limpiar filtros» lets them go (D-244)", "offers on each row Ver and what its allowed actions say: a payment, a credit note and its document (D-244)", "downloads the document of a row as its record does"); `purchase-invoices-api-endpoint.spec` ("reads the payment status of each row in any case, what it owes and its allowed actions (D-244)", "filters the list and its export by the kind of document and the payment status (D-244)"); e2e `purchase-invoices.spec.ts` ("the list says how each invoice is paid, filters by its kind and its payment, and a row offers what its record does (D-244)") y `toolbar-rows.spec.ts`, la línea a 1440 px; la imagen de referencia de `purchase-invoices`.

**Como cambió (09/11/2026, después de la revisión de los pagos): los encabezados, la barra y Comprobante** (pedidos del responsable; revisión 64 de `DESIGN.md`, frontend `c9bb84f4`, `6dcb303b` y `0d51b2b1`, merge `52cf8b2b`; revisión 65, frontend `fca1fec8`, merge `69887f1b`). El responsable mandó capturas de las listas: en Facturas, a 1440, 1680 y 1920 px, los días nunca llegaban a la línea de la búsqueda y los filtros, y poner un filtro los mandaba a otra línea, porque «Limpiar filtros» iba con ellos. Lo pidió para toda lista de `mm-data-manager`, y dijo que el cambio va en el componente compartido:
- **Los encabezados,** centrados sobre su columna en toda lista de `mm-data-manager`; las celdas conservan su lado, el texto al inicio y los montos al final, y un encabezado que ordena deja antes de su etiqueta el lugar de su flecha, para que quede en el centro con o sin ella. Una columna puede centrarse entera, su encabezado y sus celdas (`centered` de `mm-data-column`).
- **«UND»:** las unidades de Facturas y de Notas de crédito, centradas enteras, bajo un encabezado corto que se lee «Unidades» al pasar el puntero y con el lector de pantalla (`fullHeader`).
- **Un solo flujo:** la búsqueda, los filtros y después los días, Desde y Hasta juntos, tras el último filtro de su línea; al final, las acciones juntas en el extremo derecho de la última línea, «Limpiar filtros» primero, y en una línea propia si no caben con los días. Así, poner un filtro no mueve los días. Reemplaza el bloque de la revisión 46 que no se partía. La búsqueda parte del espacio que pide su texto de ayuda, nunca menos de 16rem, y no de un tercio de la barra.
- **Facturas:** Estado toma `.mm-filter--short` (7rem), como Tipo, y Proveedor un ancho nuevo, `.mm-filter--medium` (12,5rem), el más angosto que lee entero «Todos los proveedores»; Pago conserva `.mm-filter`. A 1440 px la búsqueda y los cuatro filtros llenan la primera línea, y los días abren la segunda con las acciones al final; a 1680 y 1920 px los días entran en la primera línea y las acciones toman la segunda; desde 1980 px cabe toda en una.
- **Comprobante,** centrada entera: el número, que abre la ficha, y debajo el tipo de todo documento, Factura, Boleta u Otro, en `body-sm` y `text-secondary`: texto y no la insignia neutra, para que la factura, el documento de siempre, no resalte. El tipo va bajo el número en cualquier ancho.

Pruebas: `purchase-invoices-view.spec` (la columna centrada y el tipo de los tres documentos, como texto) y las del administrador de datos; con la revisión 64, la e2e `toolbar-rows.spec.ts` en las tres anchuras y de 1280 a 2560 px —cada filtro en su límite, los días juntos tras el último filtro de su línea, las acciones juntas al final con «Limpiar filtros» primero, y los días de Facturas en su primera línea a 1680 y 1920 px— y las imágenes de toda lista, tomadas otra vez (`1491f217`). La revisión 65 ya no corrió e2e, por la regla del responsable del mismo día: la aserción del tipo en `purchase-invoices.spec.ts` se cambió sin correrla, y la imagen de `purchase-invoices` sigue mostrando la insignia.

### 6.3 Formulario

Una página con dos paneles. Se registra en `UnsavedWork`: salir con cambios pregunta con `UnsavedChangesQuestion`.

**Panel «Datos del comprobante»:**

| Campo | Comportamiento |
|---|---|
| Empresa | Solo si la sesión registra en más de una. **Como se corrigió (04/11/2026)** (frontend `3742271f`, `fdf14036`, `18e03774`; backend `df6b7276`, C8). Ofrecía toda empresa donde la sesión tenía el permiso, también por un rol de sede, y el backend rechazaba ese registro. Ahora ofrece solo las empresas donde `compras.registrar_factura` está en `wholeCompanyPermissions` o en alguna entrada de `brandPermissions` de la sesión (convenciones §9.1). Si ninguna lo está: «Su permiso de registrar facturas no alcanza a toda una empresa: el de una sede no basta. Pida que se lo den sobre la empresa o una de sus marcas.». Una sesión sin esos campos usa `permissions`, y decide el backend |
| Proveedor | `mm-search-select` con los activos de la empresa; desde el 04/11/2026, buscados en el servidor, y el que nombra un enlace se lee por su id aunque no esté en la primera página (frontend `3a995e95`) |
| Tipo | Factura prellenado; Boleta; Otro |
| Número | Muestra debajo cómo se guardará (`Se guarda como F001-123`); en Perú avisa el formato antes de enviar |
| Fecha | Calendario hasta hoy en la zona de la empresa. Si tiene más de 20 días: sin el permiso de facturas antiguas, «Más de 20 días: la registra quien tiene el permiso de facturas antiguas» y no se envía; con él, solo el aviso. Desde el 04/11/2026 el permiso se mira en la empresa y en la marca comercial de cada línea, como el backend (frontend `fdf14036`) |
| Moneda | Empieza en la moneda funcional (`CompanyOption.functionalCurrencyCode`) |
| Tipo de cambio | En la moneda funcional: «1 · moneda de la empresa», fijo. En otra: la tasa con su día y su fuente («3.742000 · venta del 31/10/2026, SBS»). Si en un día hábil la más reciente es anterior a la fecha: «No hay tasa del 02/11/2026: se usa la del 31/10/2026» y «Registrar tasa del 02/11/2026», solo con `catalogos.registrar_tipo_cambio`; si no hay ninguna: «No hay tasa registrada para esa moneda en esa fecha ni antes» con el mismo botón o, sin el permiso, «Pídala a quien registra las tasas». El botón abre el diálogo de la tasa en modo fijado: país, par, tipo y día fijos, y solo la tasa y la fuente editables. Al guardarse, o si responde `409 EXCHANGERATE_CONFLICT`, la pantalla vuelve a leer la tasa vigente (plan 5.2). **Como se construyó:** la tasa se lee con `GET /exchange-rates`, que pide `catalogos.ver`. Todos los roles sembrados lo tienen; a una sesión sin él el formulario le dice «No tiene permiso para leer los tipos de cambio (catalogos.ver).», sin «Reintentar», porque repetir no lo arregla. **Como se corrigió (04/11/2026)** (frontend `add2c336`). Si los países no se leían, el formulario tomaba `oficial` por omisión, y en Perú, que compra a la `venta` (D-181), proponía una tasa de otro tipo. Ahora no lee ninguna tasa hasta saberlo: dice «No se pudo leer el país de la empresa, que dice con qué tipo de tasa compra: no se lee ninguna tasa hasta saberlo.», con «Reintentar», que vuelve a leer los países, y retiene el registro con «Falta leer con qué tipo de tasa compra el país de la empresa: reintente.» |
| Impuesto | Propuesta del país (`CountrySummary.taxRate`), editable, en porcentaje |
| Total | Monto, con los decimales de la moneda. Desde el 04/11/2026 acepta ceros más allá de ellos, como el backend: `5900.00` en una moneda sin decimales (frontend `6386f2fa`) |
| Incluye ISC | Casilla en un cuadro de la altura de un campo, resaltado mientras está marcada. Marcada, suma la columna ISC a las líneas, su total y la acción «Aplicar a todas», y no cambia ningún monto (D-158) |
| Orden de compra | Opcional: las órdenes enviadas del proveedor (`GET /purchase-orders?companyId&supplierId&status=ENVIADA`). Elegida, muestra lo pendiente por modelo y año (`invoicing`). **Como se corrigió (04/11/2026)** (frontend `9f011f06`, sobre `6c779bc3`). Leía de una vez las 100 órdenes enviadas más recientes, y una más antigua no se podía elegir. Ahora es un `mm-search-select` que busca en el servidor por número, de la más reciente a la más antigua, 50 por página, con «Sin orden» primero; su ayuda dice cuántas más coinciden. La orden de una copia se lee por su id aunque no esté en la primera página |
| Archivo del comprobante y XML | «Adjuntar comprobante» y «Adjuntar XML», cada uno en un cuadro de la altura de un campo, con su tipo y su tope en el borde (D-174); cargados, con su nombre y su peso. En el teléfono, la zona de `mm-image-slot` |

**Como se corrigió (04/11/2026, segunda revisión): la tarjeta de datos en grupos** (pedido del responsable; frontend `94c1c6a3`; revisión 41 de `DESIGN.md`, `f6510432`). La captura del responsable mostraba una grilla despareja: Proveedor ancho y Tipo y Número apretados a su derecha, «Tipo de cambio» suelto entre los campos, «Incluye ISC» solo, la orden, el archivo y el XML repartidos en tres filas, y Proveedor y Número con la etiqueta dentro de la caja. El panel agrupa ahora sus campos por lo que dicen (`.mm-form-groups`; cada grupo es un `role="group"` nombrado por su `h3`, separado del siguiente por una línea), sobre una grilla de 12 columnas desde 1024 px, 6 por debajo y una en el teléfono; cada campo lleva la etiqueta flotando (`floatLabel="always"`):
- **Comprobante:** Proveedor (4), Tipo (2), Número (3) y Fecha (3). Empresa, cuando la sesión registra en más de una, ocupa su propia fila antes de Proveedor.
- **Importes:** Moneda (4) y «Tipo de cambio» (8), que se lee como un dato (`dl.mm-form-fact`, con `currency_exchange`): la tasa con su tipo, su día y su fuente, y debajo sus avisos; después Tasa de impuesto, Total e «Incluye ISC» (4 cada uno), el interruptor junto al total que califica.
- **Orden de compra:** la orden (4) y lo que tiene pendiente a su lado (8).
- **Archivos:** el archivo y el XML lado a lado (6 cada uno), cada uno con su tipo y su tope debajo.

No cambian la validación, el comportamiento, el orden de los campos ni su orden de tabulación. Prueba: `purchase-invoice-form-view.spec` ("groups its data by what it says, every label floated and the rate read as data in Importes (revision 41)"); las imágenes de referencia de `purchase-invoice-form` en las tres anchuras (`abb8a556`).

**Como se corrigió (05/11/2026): tres filas y el comprobante al lado** (decisión del responsable, sobre el lienzo de opciones; frontend `bd0e4a54`; revisión 47 de `DESIGN.md`; plan §2.16). Reemplaza los grupos de arriba, que ocupaban la pantalla antes de la primera línea.
- **Una tarjeta, sin títulos de grupo:** Proveedor, Tipo, Número y Fecha; Orden de compra, Moneda, Total e Impuesto; el archivo, el XML, «Tipo de cambio» e «Incluye ISC». Lo pendiente de la orden va en una fila bajo la suya. Las filas siguen el ancho de la tarjeta, no el de la ventana: por debajo de 56rem, Proveedor y la orden van a todo el ancho y los demás, de a tres por fila. Moneda muestra su código y la lista dice su nombre.
- **El comprobante al lado:** desde la tableta, un PDF o una imagen se abren a la izquierda de los datos, para copiarlos mirándolo. El PDF se ve con el visor del navegador, que ya pagina, acerca e imprime, así que no se suma ninguna librería; la imagen tiene su zoom. El archivo se abre con su tipo y ningún otro: uno que se llame .pdf y no lo sea falla en el visor. En el teléfono no se previsualiza, ni un PDF donde el navegador no tiene visor: el archivo se nombra en su cuadro. Su marco es un panel como la tarjeta de datos, con la cabecera clara (revisión 48, frontend `d7acbcb0`); la barra oscura de dentro es la del visor y la dibuja el navegador.
- **Minimizado:** «Minimizar» lo reduce a la tarjeta de `mm-image-slot`, con su nombre y su peso y sin su contenido, y «Expandir» lo abre otra vez. Cada archivo nuevo empieza abierto. Un archivo que no se acepta se dice sin quitar de la vista el que sigue en el formulario.
- **La barra del pie:** desde la tableta, los totales, «Cancelar» y «Registrar factura» quedan fijos al pie de la pantalla, y lo que toma el teclado se detiene encima de ella.
- **Las líneas:** todos los encabezados van centrados, y también las celdas menos VIN, Motor, Marca y Modelo. Marca y Modelo siguen en dos columnas porque cada una es un campo. La tabla llega a los dos bordes de su marco, y se desplaza por debajo de sus 89rem.

No cambian la validación, el envío ni las reglas de la factura. Prueba: `e2e/invoice-form.spec.ts`, en los tres anchos y en ocho anchos reales, y `mm-document-preview`, `mm-image-slot` y el formulario en sus pruebas unitarias.

**Panel «Unidades»:**
- **Barra:** «Marca comercial por omisión» (las marcas de la empresa donde la sesión registra), con el ISC encendido «Aplicar a todas» para el ISC, «Importar desde Excel» (desde el 04/11/2026, donde estaba «Pegar columnas») y «Agregar línea».
- **Tabla de líneas**, fuera de `mm-data-manager` (revisión de DESIGN): N.°, VIN, Motor, Marca, Modelo, Año, Color, Marca comercial, Valor con impuestos, ISC, Estado del VIN y quitar.
- **Importar desde Excel** (contrato C10, §11; revisión 41 de `DESIGN.md`): descarga la plantilla, lee el libro completado y agrega las líneas que trae, con su color, su marca, su modelo, su año, su valor y su ISC.

  **Como se corrigió (04/11/2026, segunda revisión): «Importar desde Excel» reemplaza «Pegar columnas»** (decisión del responsable; frontend `1184be6f`, `6a25fabe`, `4e7d440b`, `3d5387b9`, `1a1775d7`). Hasta entonces, «Pegar columnas» (revisión 34, punto 2) abría un diálogo donde se pegaban desde una hoja de cálculo el VIN, el motor y el color, con una vista previa fila por fila; la revisión 39 (frontend `8be386af`) le sumó «Para todas las líneas (opcional)», con la marca, el modelo y el año para todas las líneas pegadas. Las dos se retiraron con su analizador (`pasted-columns.ts`) y su diálogo (`paste-columns-dialog`): cada fila de la plantilla trae ya su color, su marca, su modelo, su año y su valor, y nada se completa línea por línea después.
- **Las líneas de una unidad que se vincula y la marca por omisión. Como se corrigió (04/11/2026, segunda revisión)** (frontend `cbf35bf6`, C8). El formulario retenía las líneas de una marca comercial que el permiso no alcanza leyendo el valor del formulario, que deja fuera los campos deshabilitados; una línea que vincula una unidad deshabilita los que toma de ella (D-179), así que su marca no se revisaba y el backend la rechazaba con `403`. Ahora cuenta la marca que el registro manda: la de la unidad que la línea vincula o, si no, la elegida. Y «Marca comercial por omisión» ofrece solo las marcas donde la sesión registra facturas (`brandPermissions`), no todas las de la empresa.
- **Las marcas de un rechazo y las revisiones del VIN. Como se corrigió (04/11/2026, segunda revisión)** (frontend `10b31b28`, `36b4d052`). Las líneas que nombra un rechazo (`context.lines`) se marcaban por su posición, así que quitar una línea anterior pasaba la marca a la siguiente; ahora se marcan por la línea misma, un formulario nuevo olvida las que dejó otro en el store, y al cambiar de empresa las líneas empiezan con las marcas de la otra. Una revisión del VIN que respondía después de cambiar de empresa, cuando los veredictos ya se habían olvidado, se aplicaba igual; ahora se descarta.
- **Estado del VIN**, al escribirlo o pegarlo, en lotes de hasta 100 por `GET /purchase-invoice-vin-checks`:

  | Estado | Qué hace la línea |
  |---|---|
  | «Nueva» | — |
  | «Se vincula» | Llena el modelo, el año, la marca comercial, el color y el motor desde la unidad, y los bloquea: la línea no toma el color ni el motor de lo escrito o pegado, y la unidad conserva los suyos (D-179). Si lo escrito o pegado difiere del motor o del color de la unidad, o la unidad no tiene motor, muestra el aviso de abajo |
  | «Rechazada» | Dice por qué debajo del VIN: «Es de otra empresa», «Está anulada», «Ya está en la factura F001-98 de Crosland», «Repetida en esta factura», «VIN inválido» |

- **El aviso de la unidad que se vincula** (D-179, PRC-R67). No bloquea el registro: la línea queda marcada con un aviso, como el texto de una advertencia bajo la fila, y la factura se guarda igual. Se compara sin distinguir mayúsculas ni espacios alrededor; el color, por su identificador. Los textos:
  - motor distinto: «El comprobante dice el motor JLZCCL99999 y la unidad tiene JLZCCL12346. La línea conserva el de la unidad: corríjalo en Inventario si el comprobante es el correcto.»;
  - color distinto: «El comprobante dice el color ROJO y la unidad es NEGRO. La línea conserva el de la unidad: corríjalo en Inventario si el comprobante es el correcto.»;
  - unidad sin motor: «La unidad no tiene motor registrado. La línea conserva los datos de la unidad: regístrelo en Inventario.».

  Los dos primeros juntos se dicen en un solo aviso. La pantalla no ofrece corregir la unidad desde la factura: el VIN de la línea abre la ficha de la unidad en Inventario, en otra pestaña, donde se corrige con `stock.editar_vehiculo`. Sin `unit` en la respuesta de la vista previa —quien registra no ve la unidad con `stock.ver`—, no hay con qué comparar y no se muestra aviso.
- **Totales,** desde la tableta en una barra fija al pie de la pantalla, con «Cancelar» y «Registrar factura», y en el teléfono en su barra (revisión 47): suma de las líneas frente al total del comprobante, con «Cuadra» o la diferencia («Faltan USD 5 900,00»); unidades; ISC total; base sin IGV e IGV, con «(calculado)», como la exportación (D-180). Se calculan con `computed` y `BigInt`.
- **Registrar factura:** no se envía mientras no cuadre o haya líneas rechazadas, y dice por qué. Desde el 04/11/2026 tampoco con líneas de una marca comercial que el permiso de registrar no alcanza: «Su permiso de registrar facturas no alcanza a Lima Bikes: cambie la marca comercial de esas líneas o quítelas.» (frontend `5fad3ceb`, C8; el backend las rechaza igual, PRC-R47). El `422` de desviaciones abre el diálogo de §4.4 («Revisar» o «Registrar igual»). Un `422 …EXCHANGE_RATE_CHANGED` actualiza la tasa y pide guardar otra vez. Un error con `context.lines` marca esas líneas.

**Como cambió (10/11/2026): el pie y lo que falta** (pedidos del responsable; revisiones 70 y 71 de `DESIGN.md`; frontend `68509826` y `9bcdc597`, merge `6a02be91`; `7fdb937a`, `ff464f5f` y `8a56e7ee`, merge `2aa8c4c5`). Lo que impide registrar no cambia; cambia cómo se dice:
- **El pie** lo dibuja `mm-form-foot`, el componente compartido que la revisión 70 hizo de esta misma barra para todo formulario de página, con las mismas líneas, ids, identificadores de prueba y aspecto.
- **Nada al abrir:** ningún campo en error ni resumen al abrir, tampoco en una copia; un campo dice su error una vez editado y dejado, o después de pulsar (`PageFormErrors`). Siguen como estaban el VIN, que se juzga al escribir sus 17 caracteres, y el VIN rechazado, que se dice siempre.
- **«Registrar factura» ya no se retiene,** salvo mientras registra. Pulsado con algo pendiente, no envía: `mm-error-summary`, arriba de la página, lista todo lo que falta en el orden de la página, «Faltan 4 datos para registrar la factura» —los campos de la tarjeta, los archivos, el XML y la tasa, y después las líneas: ninguna, una marca que el registro no alcanza, líneas rechazadas, líneas que no cuadran, VIN en revisión—, cada entrada un enlace a su campo; y el foco va al primer campo con error, en el medio de la pantalla, o, sin ninguno, al del primer problema («Agregar línea», el total).
- **El pie dice lo que falta** sin retener nada: la primera razón de las líneas, «Agregue al menos una línea.» o «Las líneas no cuadran con el total: …». Mientras registra, «Guardando los datos…», «Registrando factura…» con su indicador, y Cancelar deshabilitado.
- **Un rechazo** sigue en la línea de error del pie: el VIN de la primera línea rechazada o la tasa leída otra vez toman el foco, y si no nombra nada, la línea de error.
- **El foco al registrar nunca había funcionado:** buscaba `.mat-mdc-form-field-invalid`, una clase que Angular Material ya no pone en sus campos, así que se quedaba en el botón. `focusFirstError` lo busca por lo que el campo dice de sí —`aria-invalid`, `.mat-form-field-invalid`— y lo prueban sus unitarias ([retroalimentación del 10/11/2026](retroalimentacion/2026-11-10-registrar-pago.md), hallazgo 6).

Pruebas: las unitarias de `mm-form-foot`, `mm-error-summary`, `PageFormErrors`, `focusFirstError` y del formulario y sus líneas; `ng build` sin advertencias y `npm run lint:styles`. `purchase-invoices.spec.ts` suma un caso —nada al abrir, el resumen, el foco y sus enlaces— y se llevó sin correrlo, por la regla del responsable; la imagen de `purchase-invoice-form` no cambia con el pie.

### 6.4 Ficha

- **Panel de datos**, encabezado por el número y el proveedor: empresa, tipo, fecha, moneda, tipo de cambio con su día, tasa de impuesto, total, base e IGV calculados, ISC, orden (enlace a su ficha) y estado.
- **Línea de estado:** una factura anulada lo dice una vez junto a su insignia: «Anulada el 02/11/2026 15:04 por Tere Ramos: error de registro.».
- **Cabecera**, según `allowedActions`:

  | Botón | Acción |
  |---|---|
  | «Cambiar archivo u orden» | Un diálogo de edición (`CreateAndEditDialog`) con el archivo, el XML y la orden. Desde la revisión 43, el archivo y el XML son las zonas de la tarjeta, y cada una dice lo que la factura conserva hasta que se elija otro. Manda `updatedAt` y trata `OUTDATED` volviendo a leer. Desde el 04/11/2026: después de `OUTDATED` toma la factura leída otra vez, con su orden y su `updatedAt`, y conserva los archivos elegidos (§4.6, frontend `ea714437`); la orden se busca en el servidor por número, con la vinculada ahora primero aunque ya no esté enviada (frontend `9f011f06`) |
  | «Anular» | Un diálogo con el motivo en dos opciones, «Error de registro» y «Anulada por el emisor», y el texto «Sus N unidades no se anulan: quedan sin factura vigente y se vinculan por su VIN a la factura que la reemplace.». Botón destructivo «Anular factura» |
  | «Anular y copiar» | El mismo diálogo; al anular, abre `/procurement/invoices/new?copiaDe={id}` con la copia (PRC-R66). El formulario descarga el archivo y el XML de la anulada y los vuelve a enviar como partes del alta |
  | «Descargar comprobante», «Descargar XML» | Descargan el archivo |

- **Panel «Unidades»:** la misma tabla de líneas, de solo lectura: VIN, que abre la ficha de la unidad (`/fleet/vehicles/{id}`, con su propio acceso), placa, marca y modelo, año, color, marca comercial, valor, ISC y estado de la línea.
- **Estados:** esqueleto, «No encontramos esa factura» con vuelta a la lista, error con «Reintentar».

**Como cambió (09/11/2026): la cabecera, un botón principal y un menú** (decisión del responsable; revisión 62 de `DESIGN.md`; frontend `83aff699` a `e6b48de8`, 14 commits, merge `2fcbecbe`). La ficha llegó a ofrecer seis botones delineados en la cabecera de sus datos, que en el teléfono se apilaban en una columna junto al título. El responsable pidió ordenarlos —hasta ocho botones en dos filas, con «Anular» junto a «Registrar pago»— y decidió, para toda ficha, un botón principal y un menú agrupado. La cabecera muestra «Registrar pago», relleno, y «Descargar comprobante», delineado; el menú ⋮ «Más acciones de la factura» agrupa «Registrar nota de crédito» y «Registrar ingreso»; después, «Descargar XML» y «Cambiar archivo u orden»; y al final, en rojo, «Anular y copiar» y «Anular». Cada acción sigue saliendo de `allowedActions`, y sus diálogos, los de la tabla de arriba, no cambian. Un elemento deshabilitado del menú dice por qué debajo de su nombre. Las demás fichas siguen la misma regla (§6.8). Pruebas: `record-actions.spec.ts`, `purchase-invoice-profile-view.spec` y la e2e de las fichas, que elige las acciones del menú con `e2e/support/record-actions.ts`.

**Como cambió (09/11/2026, D-244): el pago de la factura sobre sus datos** (decisión del responsable; revisión 63 de `DESIGN.md`; frontend `8fa29b6e`, `3dfca05d` y `93da593f`, merge `cf862569`). El responsable pidió a los pagos una pantalla más apropiada y aprobó un resumen arriba y la tabla de los pagos en su panel:
- **El resumen del pago** abre la ficha de una factura vigente, antes de «Datos de la factura»: la insignia del estado de pago, Total, «Notas de crédito», Pagado y ~~Saldo~~ «Pendiente» desde la revisión 67 (D-245, abajo), o «Pagado de más», con su monto en positivo y «Saldo a favor del proveedor.» debajo; y una barra quieta, sin animación, con «N % pagado», la parte pagada del total después de notas, redondeada hacia abajo para que una factura no pagada entera nunca diga 100 %. En un dealer con unidades saldadas, «De eso, iniciales de clientes de…» va debajo del saldo, del que es parte. Sin `pagos.ver`, solo la insignia y Total (D-212); una factura anulada no tiene resumen. El estado viene de la factura, y las cifras, de su posición de pago, la lectura que ya hacían sus líneas y «Pagos».
- **El panel de datos** deja «Notas de crédito» y «Total después de notas», que dice el resumen.
- **«Pagos»** es solo la tabla de sus pagos, con la cuenta de cada uno ([especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md) §6.5).
- **Los montos** de la ficha, de sus paneles y de la ficha de la nota de crédito se escriben con el código ISO de su moneda, «USD 7,734.00», como la lista y la ficha del pago, y nunca con su símbolo; las tasas, con sus decimales significativos, al menos tres («3.423», «3.7521», «1.000»), en lugar de seis.
- **En el teléfono,** el resumen, los datos, «Unidades», «Notas de crédito» y «Pagos» se apilan, y las cifras van de a dos por fila.
- **Queda pendiente:** ~~en el catálogo, «Desgloses» escribe todavía con su símbolo los de la unidad y el proveedor, y con su código los de la factura; y~~ la columna «Monto en USD» de las líneas de la nota de crédito sigue sin moneda en sus montos. Los desgloses de la unidad y del proveedor del catálogo toman el código el mismo día, con la revisión 67 (frontend `0cebc479` y `e10d7873`).

Pruebas: `invoice-payment-summary.spec` ("says the status, the total, the notes, what was paid and the balance, with the code of the currency", "draws the share paid of the total after notes as a bar that says it, rounded down", "says «Pagado de más» with the amount positive and the bar full when it was paid in excess", "says under the balance of a dealer the initials of its settled units", "without pagos.ver shows only the status and the total"); `purchase-invoice-profile-view.spec` ("draws no summary on an annulled invoice, which has no payment status", "reads its amounts with the code of the currency and its rate with its significant decimals"); e2e `supplier-payments.spec.ts` ("the invoice record says how it is paid above its data, paid in excess, and its payments; «Registrar pago» opens the form on it", "a dealer invoice settled by the down payment of a client says so, and what is left is that initial (D-244)"); las imágenes de referencia de `purchase-invoice`, con F001-98 pagada en dos quintos en dólares desde una cuenta, y de `purchase-invoice-credit-notes`.

**Como cambió (09/11/2026, D-245): las palabras del pago** (decisión del responsable, un solo vocabulario en Compras; revisión 67 de `DESIGN.md`; frontend `dc442309`, merge `7ae68c2d`). El resumen dice «Pendiente» donde decía Saldo, y en «Unidades», con `pagos.ver`, la columna ~~«Cubierto»~~ dice «Pagado». Solo cambian valores de `es.json`, nunca una clave; la imagen de `purchase-invoice` sigue mostrando las palabras anteriores ([especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md), versión 9, §6.5).

**Como cambia (10/11/2026, D-248): lo que pasó a saldo, en el resumen y en cada vehículo** (decidida en el diccionario v1.53; se construye en el backend con `feature/unit-level-balance` y en el frontend con `feature/balance-trace-screens` y la revisión 76 de `DESIGN.md`). El saldo se mide por vehículo ([especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md), §2.2): una nota sobre un vehículo ya pagado pasa a saldo lo que baja de su valor, y la factura sigue debiendo lo que les falta a sus otros vehículos.
- **El resumen del pago** dice «Pasó a saldo a favor» donde decía «Pagado de más», como una resta: Total − Notas de crédito − Pagado + Pasó a saldo a favor = Pendiente. Lo lee de `toBalance` de la posición de pago, y no de un `owed` negativo (`invoice-payment-summary`, `invoice-payments-panel`). Una factura puede tener a la vez algo pendiente y algo que pasó a saldo, como la del ejemplo 8 de la especificación de pagos (§2.3).
- **El panel de líneas** (`purchase-invoice-lines`), con `pagos.ver`, da por vehículo lo cubierto, la inicial, lo que pasó a saldo y lo pendiente, con la identidad valor − cubierto − inicial + pasó a saldo = pendiente; sin `pagos.ver`, solo el estado de pago de cada línea, como hasta ahora (D-212).
- **`GET /purchase-invoices/{id}`** y las filas de la lista suman `toBalance`, con `pagos.ver`, como `owed`; `owed` es lo que les falta a sus vehículos y conserva su signo, que un pago registrado antes del cambio puede dejar negativo.
- **Como se construyó (10/11/2026)** (backend `c5444af2`, merge `ba5bbb15`; frontend `1796c27c`, `9025e808`, `a6247590`, `051c421b`, `8be18746` y `0272b66e`, merge `96b1a3f6`). El resumen y su desglose dicen «Pasó a saldo a favor» y un pendiente que nunca baja de cero. La tabla de vehículos junta VIN y placa; marca y modelo, con año, color y marca comercial debajo; los montos; y estado y pago, para que «Pendiente» entre a 1440 px. La e2e registra una nota de 590,00 sobre el vehículo pagado de F001-123, que pasa a saldo mientras la factura sigue debiendo su otro vehículo, y F001-200 dice «Pasó a saldo a favor» donde decía «Pagado de más»; las imágenes de la factura se regeneraron y se miraron una por una. En el backend, la ficha y las filas de la lista traen `toBalance` (`PurchaseInvoicesIntegrationTests`, `PurchaseInvoiceReadsIntegrationTests`).
- **Pruebas que se escriben:** las e2e de `purchase-invoices` (el resumen y las líneas), las unitarias de cada componente tocado, también la de `purchase-invoice-credit-notes`, que falta, y las imágenes de `purchase-invoice` en las tres anchuras, revisadas una a una.

### 6.5 `fleet` y la orden

- **Inventario:** el filtro «Factura vigente» (Todas, Con factura vigente, Sin factura vigente) y, en cada fila sin línea vigente, la insignia «Sin factura vigente» (D-175).
- **Editar unidad:** el diálogo deja fijo el año de una unidad con `hasCurrentInvoice`, dice por qué y manda el año que tiene (D-160, FLT-R50); el backend lo rechaza igual con `422 VEHICLE_MODEL_YEAR_LOCKED`.
- **Ficha de la unidad:** «Anular unidad» ~~en la cabecera~~ al final del menú «Más acciones de la unidad», en rojo, desde el 09/11/2026 (revisión 62, §6.8), con `allowedActions.annul` y `ConfirmationDialog`; no se ofrece mientras la unidad tenga una cesión abierta (D-182), y si el backend responde `VEHICLE_FUND_ASSIGNMENT_OPEN`, la ficha muestra «La unidad tiene una cesión abierta a un fondo: ciérrela antes de anularla.». Una unidad anulada lo dice junto a su insignia: «Anulada el dd/mm/aaaa: ya no se modifica.».
- **Ficha de la orden:** «Todo facturado» o lo pendiente por modelo y año. Sin `annul` cuando tiene facturas vigentes. Desde el 09/11/2026 «Anular» va, con «Descargar PDF», en el menú del pie de su hoja (revisión 62, §6.8). El mismo día, el aviso «Todo facturado» toma solo la altura de sus líneas: ocupaba unos 300 px (frontend `01a951be`, merge `52cf8b2b`).

### 6.6 A 375 px y accesibilidad

**A 375 px:**
- La lista es la de `mm-data-manager`, que no tiene modo de tarjetas: su tabla se desplaza de lado.
- En el teléfono, por debajo de 768 px, cada línea es una tarjeta, como las de la orden; desde 768 px, la tabla de líneas mantiene fijos a la izquierda el N.° y el VIN mientras se desplaza de lado, y la barra de totales se pliega en una fila de 48 px con «Ver los totales» (DESIGN.md, revisión 34).
- El diálogo de pegar ocupaba la pantalla entera, y desde la revisión 39 apilaba sus tres campos de «Para todas las líneas». Desde el 04/11/2026 lo reemplaza «Importar desde Excel» (§11.3), cuya tabla de filas con errores se desplaza dentro de su propia región, de 16rem o 40dvh como máximo.
- La tarjeta de datos, desde la revisión 41, pasa a una columna en el teléfono, y la lista pliega sus filtros detrás de «Filtros» (revisión 40).
- Los totales quedan en una barra fija al pie.
- La página no tiene desplazamiento horizontal.

**Accesibilidad:**
- Cada celda editable tiene su `aria-label` («VIN de la línea 3»), y su error queda enlazado con `aria-describedby`.
- El estado del VIN es texto, no solo color.
- La diferencia de los totales y el resultado de la vista previa se anuncian en una región `aria-live="polite"`.
- Después de agregar las líneas de un libro, el foco va al VIN de la primera línea agregada (antes, después de pegar); «Cancelar» lo devuelve a «Importar desde Excel». El resultado de la importación se dice una vez en una región `role="status"`.
- Los diálogos atrapan el foco y lo devuelven al botón que los abrió.
- axe no encuentra violaciones en las tres anchuras.

### 6.7 Stores y fachadas

| Pieza | Contenido |
|---|---|
| `procurement/domain/model` | `purchase-invoice.entity.ts` (`PurchaseInvoice`, `PurchaseInvoiceLine`), `vin-check.ts`, `commands/register-purchase-invoice.command.ts`, `update-purchase-invoice.command.ts`, `annul-purchase-invoice.command.ts`, `purchase-invoice-amounts.ts` (usa `amountMinor`, `sumMinor` y `fromMinor` de `purchase-order-amounts.ts`), `invoice-number.ts`, `pasted-columns.ts` hasta el 04/11/2026; desde entonces `invoice-line-import.ts` (C10, frontend `1184be6f`) |
| `procurement/infrastructure/api` | `purchase-invoices-api-endpoint.ts`, `purchase-invoices-response.ts`, `purchase-invoice-vin-checks-response.ts`, `register-purchase-invoice.request.ts`, `update-purchase-invoice.request.ts`, `annul-purchase-invoice.request.ts`; `procurement-api.ts` suma sus miembros. Desde C10: `purchase-invoice-line-import-response.ts`, y el endpoint suma `downloadLineTemplate()`, que pide el documento generado con `downloadDocument` (`3d5387b9`), e `importLines(file)` |
| `procurement/infrastructure/assemblers` | `purchase-invoice-assembler.ts`; desde C10, `invoice-line-import-assembler.ts` |
| `procurement/application` | `purchase-invoices.store.ts`: `invoicePage` (`PagedList`), `invoice`, `vinChecks`, `loading`, `saving`, `exporting`, `error`, `notice`, y los métodos `register()` (devuelve el id nuevo o `null`), `update()`, `annul()` y `checkVins()` (devuelven `Promise<boolean>`), `exportInvoices()`. Elige el texto del error con un `switch` por `ApiError.code`. Desde C10, `downloadLineTemplate()` e `importLines(file)`, que devuelve las líneas o el rechazo del libro con lo que nombra |
| `procurement/presentation` | Vistas `purchase-invoices-view`, `purchase-invoice-form-view` y `purchase-invoice-profile-view`; componentes `purchase-invoice-lines` (edición y lectura), `paste-columns-dialog` (reemplazado el 04/11/2026 por `import-invoice-lines-dialog`), `annul-purchase-invoice-dialog`, `edit-purchase-invoice-dialog`, `purchase-invoice-status-tone.ts`; servicio `OrderDeviationQuestion`, como `LargeRateChangeQuestion` |
| `LocationFacade` | `loadPurchaseRate({country, fromCurrency, toCurrency, rateType, date})` sobre `GET /exchange-rates?…&to={día siguiente}&page=0&size=1`; `purchaseRate`, `purchaseRateLoading`, `purchaseRateError` y `clearPurchaseRateError()`, su propio estado (frontend-feature §4); `registerFixedRate(fixed): Promise<FixedRateResult \| null>`, que abre el diálogo de `location` en modo fijado con `import()` (frontend-rules 1.3, «Puede abrir un flujo de su propio contexto»); `countryPurchaseRateType(country)` y `countryTaxRate(country)`. `RegisterExchangeRateDialogData` suma `fixed?: {country, fromCurrency, toCurrency, rateType, rateDate}` |
| `OrganizationFacade` | Los miembros que ya tiene: `loadCompanyOptions` (con `countryCode` y `functionalCurrencyCode`) y `loadCommercialBrands` |
| `VehicleCatalogFacade` | Los que ya tiene: marcas, modelos de una marca, colores y un modelo por su identificador |
| `IamFacade` | `hasPermission`, `hasPermissionIn` y, desde el 04/11/2026, `actsOnWholeCompany(code, companyId, brandIds)`, que lee `wholeCompanyPermissions` y `brandPermissions` de la sesión (C8) |
| Búsquedas en el servidor (04/11/2026) | `procurement/presentation/services`: `searched-options.ts`, `searched-suppliers.ts` y `searched-sent-orders.ts`, sobre `mm-search-select` con `remote`; `PurchaseOrdersStore.searchSentOrders` |
| `fleet` | No ofrece fachada: la ficha enlaza a `/fleet/vehicles/{id}`. Sus pantallas suman `hasCurrentInvoice` y `annul` en su propio store |

### 6.8 Revisión de `DESIGN.md`

**Escritas el 03/11/2026 como revisión 34 de DESIGN.md.** Donde esta lista y la revisión difieren en lo visual, manda la revisión. Por ejemplo, la insignia del VIN reutiliza los tonos existentes, y el aviso por línea de D-179 usa el estilo nuevo `.mm-notice`. La lista original:

| # | Qué decide | Por qué |
|---|---|---|
| 1 | La tabla de líneas de la factura, editable y fuera de `mm-data-manager`: cabecera fija, hasta 100 filas, celdas con su control, columna de estado, pie de totales y tarjetas en el teléfono. Su modo de lectura en la ficha | Hallazgo A22: una tabla nueva fuera de `mm-data-manager` exige una decisión (frontend-feature §3) |
| 2 | El diálogo «Pegar columnas», con su vista previa y los errores por fila | Patrón nuevo |
| 3 | La insignia del estado del VIN (Nueva, Se vincula, Rechazada) y su motivo bajo el campo | Tono y texto nuevos de `mm-status-badge` |
| 4 | El campo del tipo de cambio con su día, el aviso de tasa anterior y «Registrar tasa»; el modo fijado del diálogo de la tasa | Patrón nuevo, prestado por la fachada |
| 5 | El diálogo de avisos confirmables con una tabla dentro | `ConfirmationDialog` hoy no lleva tabla |
| 6 | El diálogo de anulación con su motivo y «Anular y copiar» | Diálogo destructivo con una elección |
| 7 | La barra de totales fija al pie a 375 px | Patrón nuevo |
| 8 | La insignia «Sin factura vigente» y el filtro del Inventario; «Anular unidad» y el estado anulado de la ficha de la unidad | Cambian pantallas de `fleet` |

**Revisión 39 (04/11/2026).** La revisión de punta a punta completa el punto 2 de la revisión 34: «Pegar columnas» suma «Para todas las líneas (opcional)», con la marca, el modelo y el año que van a todas las líneas pegadas (§6.3; frontend `8be386af`). El resto de la revisión 34 no cambia. La misma revisión 39 nombra por su método un pago sin número de operación, que es de la [especificación de los pagos](especificacion-pagos-a-proveedores-2026-11-03.md).

**Revisiones 40 y 41 (04/11/2026, segunda revisión).** Las escribió el frontend con los pedidos del responsable; aquí se citan, no se repiten:
- **Revisión 40** (`89f9f6c5`, `bebbab4f`): la barra de una lista con tres filtros o más, en dos filas, con «Limpiar filtros» y, en el teléfono, «Filtros» (§6.2). La toman Facturas, Notas de crédito, Pagos y las demás listas de varios filtros; Órdenes de compra y las listas de una ficha, de dos filtros o menos, conservan su barra.
- **Revisión 41** (`f6510432`): «Importar desde Excel» reemplaza «Pegar columnas» —enmienda el punto 2 de la revisión 34 y la revisión 39— (§6.3, §11.3), y la tarjeta de datos de la factura, de la nota y del pago agrupa sus campos (§6.3).

**Revisiones 42 y 43 (04/11/2026, pedidos del responsable; frontend `863817c3`).** Plan de compras y habilitación, §2.12:
- **Revisión 42** (`3da41678`): los filtros arriba y, abajo, las fechas y «Limpiar filtros» al inicio y las acciones al final (§6.2).
- **Revisión 46** (frontend `4a0cfd49`): las fechas a la izquierda y los botones a la derecha; corrige las revisiones 44 y 45.
- **Revisión 44** (`3738bc51`, `b1b60754`): un solo flujo y solo los filtros necesarios. Enmienda la revisión 42; Facturas deja Tipo y Moneda (plan de compras y habilitación, §2.13).
- **Revisión 43** (`d25f3fe6`): una sola zona de archivo, una sola voz, un ejemplo o una indicación en cada campo vacío y una sola grilla, en las tres tarjetas, en «Importar desde Excel» y en los dos diálogos que toman un archivo. Enmienda la revisión 18 por decisión del responsable, y corrige en la revisión 40 el texto de Habilitación, que empieza en «Todas».

**Revisiones 62 y 63 (09/11/2026, decisiones del responsable después de revisar las facturas con datos de demostración).** Las escribió el frontend; aquí se citan, no se repiten:
- **Revisión 62** (`f52a3913`, `55e2e21f`; merge `2fcbecbe`): las acciones de una ficha. A lo sumo dos botones en la cabecera de sus datos, el principal relleno y uno secundario delineado, y el resto en el menú ⋮ «Más acciones de la …», en grupos: lo que registra algo relacionado; el archivo y las correcciones; los enlaces «Ver …»; y lo que retira la ficha, al final y en rojo, con la clase global `.mm-menu-item-destructive`. Lo dibuja el componente compartido `mm-record-actions` (`shared/presentation/components/record-actions`). Vale para las fichas de la factura, del pago —«Registrar otro pago» pasa a ser su acción principal—, de la nota de crédito, del proveedor, de la unidad y de la empresa —su Desactivar pasa al menú, en rojo—, y para el pie de la orden: Cancelar, Enviar o Cumplir, Guardar y el menú con «Descargar PDF» y «Anular»; «Descargar PDF» sigue como botón cuando no se ofrece ni Enviar ni Cumplir. Reemplaza, para dónde van los botones, la decisión del responsable del 19/10/2026 de las revisiones 13 y 15 (§6.4).
- **Revisión 63** (`ac5569e1`; merge `cf862569`): el pago de una factura. El resumen sobre sus datos, que enmienda la revisión 13 para la factura; «Pagos», solo su tabla, que enmienda el ítem 6 de la revisión 37; el código ISO en los montos de la factura y de la nota, que enmienda las revisiones 34, 36 y 37 en esas fichas; y la lista con «Pago», Tipo y Pago, que enmienda las revisiones 40, 44 y 46 (§6.2, §6.4).

**Revisiones 64, 65 y 67 (09/11/2026, pedidos y decisiones del responsable después de revisar los pagos).** Las escribió el frontend; aquí se citan, no se repiten:
- **Revisión 64** (`5afff451`; merge `52cf8b2b`): los encabezados centrados en toda lista, una columna centrada entera, «UND», y la barra en un solo flujo, con los días tras los filtros y las acciones al final, «Limpiar filtros» primero. Enmienda las revisiones 16 y 17 en los encabezados, reemplaza la 46 en los días y en «Limpiar filtros», y el bloque de la 44 que no se partía en las acciones; el límite de un filtro de la 44 sigue (§6.2).
- **Revisión 65** (`a5aa53cb`, `e282f746`; merge `69887f1b`): el número de una orden abre su ficha y la lista de órdenes lee el RUC del proveedor bajo su razón social (`supplierTaxId`, backend `600c4859`, merge `7f76c5bc`); las hojas de la orden y de la plantilla toman en sus bandas y en la cabecera de sus líneas el tono de la cabecera de una tabla, `table-header`, en lugar del azul primario, TOTAL su peso sin relleno y el título «ORDEN DE COMPRA» `text-primary`, mientras el PDF conserva sus colores; y Comprobante centrada con el tipo de todo documento bajo el número (§6.2). Enmienda las revisiones 28, 32 y 63.
- **Revisión 67** (`92c447e7`; merge `7ae68c2d`): un solo vocabulario fuera del formulario del pago (D-245), también en la ficha de la factura (§6.4); el resto es de la [especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md), versión 9.

**Revisiones 70 a 73 (10/11/2026, pedidos del responsable).** Las escribió el frontend; aquí se citan, no se repiten:
- **Revisión 70** (`82511e4f`; merge `6a02be91`), «Pie de un formulario de página»: las acciones al final de un formulario que ocupa la página se ven como en Registrar factura, dibujadas por `mm-form-foot`, Cancelar delineado primero y la principal al final. Enmienda la 47 y, en el pie de la orden, la 62 (§6.3).
- **Revisión 71** (`e062439f`; merge `2aa8c4c5`), «Lo que falta en un formulario de página»: nada al abrir, un campo dicho una vez editado y dejado, la principal nunca retenida, `mm-error-summary` arriba, el foco en el primer campo con error y «Guardando los datos…». Enmienda las revisiones 34 y 47 en «Registrar factura» retenido, y la 70 (§6.3).
- **Revisión 72** (`8c016f01`; merge `d59b2f31`): el estado de toda lista va centrado bajo su encabezado; en Facturas, «Estado» y «Pago» (§6.2). Enmienda las revisiones 64 y 68.
- **Revisión 73** (`671933e4`; merge `de2d1c56`): el ícono de un dato de una ficha se queda centrado en su etiqueta y la primera línea de su valor, aunque su celda crezca por otra más alta, también en la ficha de la factura (§6.4). Enmienda la revisión 13.

---

## 7. Pruebas

### 7.1 Los criterios de aceptación del plan

| Criterio (plan §5, fase 2) | Prueba |
|---|---|
| Una factura de 100 líneas se guarda en una transacción | `PurchaseInvoiceRegistrationStatementsTests.theQueriesOfARegistrationDoNotGrowWithItsLines`, que registra facturas de 1, 10 y 100 líneas nuevas, cada una en un solo pedido; `PurchaseInvoiceRollbackTests.aFailureOnLineOneHundredLeavesNoRowAndNoFile` |
| Un fallo en la línea 100 no deja filas ni archivo | Desde el 04/11/2026 (backend `7984202f`, hallazgo B20), `PurchaseInvoiceRollbackTests.aFailureOnLineOneHundredLeavesNoRowAndNoFile`: sin la transacción de la prueba, falla en el último paso, después de escribir las 100 unidades con su historial, la factura con sus 100 líneas y su archivo, y comprueba que `purchase_invoices`, `purchase_invoice_items`, `vehicles`, `vehicle_status_history` y `kept_files` no tienen ninguna fila de su empresa y que el objeto se descartó. La prueba de la versión 3, que fallaba antes de escribir nada, se llama ahora `PurchaseInvoicesIntegrationTests.aRefusalOnLineOneHundredIsAnsweredBeforeAnythingIsWritten`; `PurchaseInvoiceFilesTests.aRolledBackRegistrationDiscardsItsObject` |
| El doble envío da 409 | `PurchaseInvoicesIntegrationTests.theSameRequestTwiceAnswersNumberConflict` |
| El número de sentencias, fijado con las estadísticas de Hibernate | `PurchaseInvoiceRegistrationStatementsTests.theQueriesOfARegistrationDoNotGrowWithItsLines` y, desde el 04/11/2026, `theQueriesOfARegistrationThatLinksExistingUnitsDoNotGrowWithItsLines` (§7.2) |
| Se mide primero la línea base; 100 líneas en menos de 2 s es el objetivo provisional | `PurchaseInvoiceRegistrationBaselineTests` (§7.2) |
| Una factura en soles guarda tasa 1 y su fecha | `PurchaseInvoicesIntegrationTests.registeringWithoutCreateVehicleBringsTheUnitsToLifeAndAnInvoiceInSolesKeepsRateOneAndItsDate` |
| Dos facturas con el mismo VIN nuevo: una gana y no queda ninguna unidad huérfana | `PurchaseInvoiceLockTests.twoInvoicesWithTheSameNewVinLeaveOneUnitAndNoOrphan`, con dos transacciones reales |
| Un VIN de otra empresa da 409 sin revelar sus datos | `PurchaseInvoicesIntegrationTests.aVinOfAnotherCompanyAnswersConflictWithoutItsData` (comprueba que `context` lleva solo la línea y el VIN enviado) |
| Sin `stock.crear_vehiculo` funciona | `PurchaseInvoicesIntegrationTests.registeringWithoutCreateVehicleBringsTheUnitsToLifeAndAnInvoiceInSolesKeepsRateOneAndItsDate` |
| Sin `compras.registrar_factura`, o fuera de la marca, da 403 | `PurchaseInvoicesIntegrationTests.withoutRegisterInvoicesAnswersAccessDeniedAndAnOldInvoiceNeedsItsPermission`, `aLineOfABrandTheCallerDoesNotActOnAnswersAccessDenied` |
| AE, OP y CO todavía no tienen el permiso | Valió hasta V104, de la fase 4, que da los permisos de facturas a los roles de la matriz; desde entonces lo prueba `InvoicePermissionsMatrixMigrationTests.theInvoicePermissionsFollowTheMatrixOnceCreditNotesExist`, y la prueba de «solo el superadministrador» se retiró (backend `9137fc62`) |
| Anular deja la unidad sin factura vigente, y la nueva la vincula por su VIN | `PurchaseInvoiceAnnulmentsIntegrationTests.anAnnulledInvoiceLeavesItsUnitsWithoutCurrentInvoiceTheChangeLogKeepsItsReasonAndTheNewOneLinksThem` |
| La anulación queda en la bitácora | `PurchaseInvoiceAnnulmentsIntegrationTests.anAnnulledInvoiceLeavesItsUnitsWithoutCurrentInvoiceTheChangeLogKeepsItsReasonAndTheNewOneLinksThem` |
| Cambiar el modelo de una unidad con línea vigente da 422 | `VehiclesIntegrationTests.theYearOfAnInvoicedUnitDoesNotChange` y `VehicleInvoiceFactsIntegrationTests.theYearOfAUnitWithACurrentLineDoesNotChangeAndTheRestDoes` (el modelo ya no tiene ruta que lo cambie, FLT-R08; lo que se prueba es el año, que sí la tiene) |
| El puerto de dependientes responde «ninguno» | Valió hasta la fase 4. Desde las fases 4 y 5 los dependientes son las notas y los pagos: `PurchaseInvoiceDependentsTests.creditNotesAndPaymentsAreTheDependents`, `anInvoiceWithAPaymentHasDependents`; `PurchaseInvoiceAnnulmentsIntegrationTests.anInvoiceWithACurrentNoteAnswersHasDependents` |
| Un PDF renombrado `.png`, o uno de más de 5 MB, da 422 | `PurchaseInvoiceUploadLimitsTests.aPdfRenamedPngIsReadAsAPdfAndAnHtmlIsRefused`, `aFileOverFiveMegabytesAnswersFileSize`. Un PDF con nombre `.png` es un PDF válido por su contenido y se acepta: el `422` es para lo que no es PDF, PNG, JPEG ni WebP, con cualquier nombre |
| El total cuadra al céntimo con 100 líneas pegadas (desde el 04/11/2026, importadas) | `purchase-invoice-amounts.spec` ("adds a hundred lines to the cent"); e2e ("pastes 80 VINs with engine and color, refuses two and squares the total; a hundred lines square to the cent", que desde el contrato C10 es "imports 78 lines of a workbook that refuses two rows, and a hundred lines square to the cent; a 101st is refused (revision 41)", frontend `4e7d440b`) |
| axe no encuentra violaciones | `e2e/accessibility.spec.ts`, que pasa axe por las pantallas de `e2e/screens.ts`, con las tres nuevas: `purchase-invoices`, `purchase-invoice-form` y `purchase-invoice` |
| Salir pregunta antes de descartar | `purchase-invoice-form-view.spec` ("asks before discarding, and leaves at once without changes"); e2e ("leaving with lines asks first") |
| `ModularityTests`, RLS y filas que no se borran | `ModularityTests`; `ProcurementCompanyIsolationIntegrationTests`; `ProcurementRowsAreNeverDeletedTests`; `NoDeleteEndpointsTests` |

### 7.2 Rendimiento

1. **Línea base, antes de optimizar nada.** `PurchaseInvoiceRegistrationBaselineTests`, etiquetada `performance` y fuera de la suite normal, corre sobre `erp_mm_db_verify` con la cuenta de la aplicación. Registra cinco veces facturas de 1, 10, 50 y 100 líneas nuevas, y otras de 100 líneas que vinculan unidades existentes, y escribe la mediana y el máximo de cada tamaño. El resultado se anota en el cierre (paso 5.12) con su commit.

   **Como se construyó:** la prueba lleva la etiqueta `performance` y corre solo con `-Dperformance=true` (`@EnabledIfSystemProperty`), fuera de la suite normal. **La línea base del 03/11/2026**, sobre el código de `16f69af3` y `erp_mm_db_verify`, medida con la cuenta `postgres` y no con la de la aplicación, así que no pasa por Row Level Security:

   | Factura | Mediana | Máximo |
   |---|---|---|
   | 1 línea nueva | 582 ms | — |
   | 10 líneas nuevas | 916 ms | — |
   | 50 líneas nuevas | 1 150 ms | — |
   | 100 líneas nuevas | 1 731 ms | 2 063 ms |
   | 100 líneas que vinculan unidades existentes | 1 025 ms | — |

   La mediana de 100 líneas nuevas cumple el objetivo provisional de 2 s; el máximo lo pasa. Las líneas se siguen insertando una por una: el `INSERT … SELECT unnest(…)` de §2.3 no hizo falta. Falta medirla con la cuenta de la aplicación, que sí pasa por Row Level Security. Al 04/11/2026 sigue siendo la única medición: la revisión no la repitió, y la lectura en una sentencia de las unidades que se vinculan (`61ebde79`) llegó después de medir la fila de 100 líneas que vinculan (§10).
2. **Objetivo provisional:** 100 líneas en menos de 2 s de mediana. Si la línea base lo pasa, se ataca en este orden: las lecturas por lote que se hayan escapado (la prueba de sentencias lo dice), la inserción de líneas con `unnest` (§2.3) y el historial de las unidades nuevas por lote.
3. **Sentencias.** `PurchaseInvoiceRegistrationStatementsTests` activa `hibernate.generate_statistics` (`@TestPropertySource`), registra facturas de 1, 10 y 100 líneas, y comprueba que `getPrepareStatementCount() − getEntityInsertCount()` es el mismo número en los tres tamaños. Ese número queda escrito en la prueba: 57 sentencias preparadas, además de las inserciones, con 1, 10 y 100 líneas (`STATEMENTS_BESIDES_INSERTS`).

   **Al día al 04/11/2026.** El número es 64: las 57 de la fase 2, 3 que sumaron las notas de crédito (la lectura de las notas vigentes que impiden anular y las dos lecturas agrupadas de lo que las notas le quitan a la factura, backend `ccf4dffb`) y 4 de los pagos (la lectura de los pagos que impiden anular, `ecc963ec`, y las tres de la posición de pago de sus unidades en la ficha, `56f992c8`). Hibernate cuenta lo que prepara: las escrituras que van por JDBC, como las de `fleet` y las de las versiones de los datos, no entran en la cuenta. La prueba solo registraba líneas nuevas; las líneas que vinculan unidades existentes crecían una lectura y un refresco por unidad, dos veces, y la revisión lo halló (B15). Desde `61ebde79`, `theQueriesOfARegistrationThatLinksExistingUnitsDoNotGrowWithItsLines` registra facturas de 1, 10 y 100 líneas que vinculan y comprueba que el número es el mismo en los tres tamaños, sin fijarlo (§2.3).
4. **Una página de la lista (09/11/2026, D-244).** Desde que las filas dicen su estado de pago y traen sus acciones, `PurchaseInvoiceListStatementsTests` activa `hibernate.generate_statistics` (`@TestPropertySource`), lee páginas de 2, 20 y 80 filas, con un dealer y una unidad saldada para que corran todas las lecturas de los pagos, y comprueba que cada página prepara el mismo número de sentencias, fijado en la prueba: 17 una página (`STATEMENTS_OF_A_PAGE`) y 20 una página filtrada por el estado de pago (`STATEMENTS_OF_A_FILTERED_PAGE`), que lee además todas las filas antes de cortarla (backend `6230346b`).

### 7.3 Extremo a extremo

**Antes de las pruebas nuevas:**
1. **Commit mecánico de mocks.** `e2e/support/api.ts` (7 842 líneas) conserva `mockApi`, los tipos de sesión y las ayudas compartidas. Cada contexto pasa a `e2e/support/mocks/<contexto>.ts` (`location.ts`, `iam.ts`, `organization.ts`, `vehiclecatalog.ts`, `fleet.ts`, `procurement.ts`), con una función `register<Contexto>Routes(page, state)`. No cambia ninguna respuesta, y la suite completa pasa igual (hallazgo A23).
2. **La pantalla «En construcción»** de `e2e/screens.ts` pasa de `/procurement/invoices` a `/procurement/credit-notes`, con su encabezado; se regeneran solo las tres imágenes `under-construction-*`.

**Escenarios de `e2e/purchase-invoices.spec.ts`**, con los mocks de facturas en `mocks/procurement.ts` y los de la unidad en `mocks/fleet.ts`:
1. La lista filtra por empresa, proveedor, estado y período, y exporta con los filtros de la pantalla.
2. Registra una factura en dólares de tres líneas: la tasa propuesta con su día, una línea que se vincula y dos nuevas.
3. Pega 80 VIN con motor y color; la vista previa rechaza dos; suma 78 y cuadra el total. Con 100 líneas, cuadra al céntimo.
4. «Incluye ISC» exige el ISC y «Aplicar a todas» lo llena.
5. En soles, la tasa es 1 y no se edita.
6. Sin tasa del día: el aviso, «Registrar tasa» en modo fijado (`201`) y la tasa propuesta nueva; con `409`, la vuelve a leer.
7. El `422` de desviaciones abre la pregunta; «Registrar igual» reenvía con `confirmOrderDeviation: true`.
8. Una fecha de más de 20 días sin el permiso lo dice y no envía.
9. Ficha: anular con motivo; «Anular y copiar» precarga la copia y propone otra vez la tasa.
10. «Cambiar archivo u orden»; con `PURCHASEINVOICE_OUTDATED`, vuelve a leer.
11. Salir con líneas pregunta antes de descartar.
12. Sin `compras.registrar_factura` no aparece «Registrar factura».
13. Inventario: la insignia y el filtro «Sin factura vigente»; anular una unidad desde su ficha; con una cesión abierta, «Anular unidad» no se ofrece y el `422 VEHICLE_FUND_ASSIGNMENT_OPEN` dice que primero se cierre (D-182).
14. Una línea que se vincula con un motor pegado distinto del de la unidad conserva el de la unidad y muestra el aviso, sin impedir el registro (D-179).

**Los que sumó la revisión (04/11/2026):**
15. Pega 100 líneas con la marca, el modelo y el año para todas, y registra la factura cuadrada al céntimo (revisión 39; frontend `8be386af`).
16. Encuentra por su número una orden enviada más antigua que las 50 más recientes, buscada en el servidor (`9f011f06`).
17. Un rol de sede nunca ofrece su empresa: el permiso no actúa en toda la empresa (C8; `c8b1d359`).
18. Una sesión sin los campos del alcance usa los permisos de la empresa, y decide el backend (C8; `c8b1d359`).
19. Un rol limitado a una marca ofrece su empresa, registra una factura antigua solo con las líneas de su marca y retiene una línea de otra marca (C8; `c8b1d359`, `1ef10642`).

`e2e/accessibility.spec.ts` pasa axe, además, por «Pegar columnas» con los valores para todas las líneas, por las órdenes enviadas buscadas y abiertas y por el formulario sin empresa donde la sesión registre en toda ella.

**Lo que cambió la segunda revisión (04/11/2026; frontend `4e7d440b`, `1a1775d7`, `045170c4`, `abb8a556`).** Con el contrato C10, los escenarios 3, 14 y 15 importan un libro en lugar de pegar columnas: "imports 78 lines of a workbook that refuses two rows, and a hundred lines square to the cent; a 101st is refused (revision 41)", "a linked unit keeps its engine when the workbook brings another, with a notice that does not block" e "imports 100 lines with their brand, model and year, and registers the invoice squared to the cent (revision 41)". Suma uno:

20. Dice por qué se rechaza un libro entero: de otro tipo, o con otras columnas que la plantilla ("says why a workbook is refused whole: of another type, or with other columns than the template (C10)").

axe pasa por «Importar desde Excel» con líneas listas y filas rechazadas ("«Importar desde Excel» with lines ready and rows refused has no WCAG A/AA violations (revision 41)"), en lugar de «Pegar columnas», y en el teléfono despliega «Filtros» antes de escanear (`045170c4`). Las imágenes de referencia de `purchase-invoices` y `purchase-invoice-form`, en las tres anchuras, se tomaron otra vez con la barra de la revisión 40 y los grupos de la 41 (`abb8a556`), y cada una se miró.

`e2e/screens.ts` suma `purchase-invoices`, `purchase-invoice-form` (con líneas) y `purchase-invoice`, con sus imágenes en las tres anchuras y su pasada de axe. Las e2e corren en el puerto 4300, con el equipo enchufado.

### 7.4 Sondas en vivo

`e2e-live/purchase-invoices.live.spec.ts`, contra el backend real sobre `erp_mm_db_verify`, con la cuenta del superadministrador de verificación (D-176):
1. Registra una factura en soles con dos VIN nuevos, con su PDF; la ficha muestra sus unidades, que nacieron En habilitación y sin ubicación.
2. El mismo número otra vez responde `409`.
3. Una factura en dólares: registra en modo fijado la tasa que falta y se guarda con ella.
4. Vincula una orden enviada: la pregunta de desviaciones confirmada, y la orden muestra lo pendiente.
5. Anula con motivo: la bitácora tiene su fila, y el Inventario filtrado muestra las dos unidades sin factura vigente.
6. «Anular y copiar» vincula los mismos VIN a la factura nueva.
7. La exportación descarga un Excel.
8. Ningún pedido de la corrida usó `DELETE`, y el registro del backend no tiene ningún `ERROR`.

**Pendiente al 03/11/2026.** Las sondas no corrieron: necesitan la contraseña de la cuenta de verificación del responsable. `e2e-live/purchase-invoices.live.spec.ts`, el commit F17, existe desde el frontend `10696eed`, y `d126d8a2` toma el año del modelo de la sonda del calendario de Lima. Cuando corran cubren los ocho casos de arriba, numerados como ellos, después de un caso que entra, encuentra la empresa de Perú y un proveedor suyo y prepara lo que necesitan las líneas, y además:
- la lista, la exportación, la ficha, el archivo y el XML, dentro de los casos 1, 3 y 7;
- `GET /purchase-invoice-vin-checks`: "says what each VIN would do before an invoice is registered (GET /purchase-invoice-vin-checks)";
- `invoicing` de la orden y su acción `annul`, y la anulación de una orden con facturas vigentes, que responde `422 PURCHASEORDER_HAS_CURRENT_INVOICES`: el caso 4, "links a sent order: its deviations confirmed, the order says what is pending, and it cannot be annulled meanwhile";
- un `PUT` de la factura que conserva una orden `CUMPLIDA`: "accepts a change of the file of an invoice that keeps its order, fulfilled since";
- un `PUT /vehicles/{id}` con otro año en una unidad facturada, que responde `422 VEHICLE_MODEL_YEAR_LOCKED`: "keeps the year of a unit in a current invoice: another one is refused with VEHICLE_MODEL_YEAR_LOCKED".

El caso 8 de la sonda comprueba que ningún pedido usó `DELETE`; que el registro del backend no tenga ningún `ERROR` lo mira quien la corre. La sonda de la habilitación, `e2e-live/enablement.live.spec.ts` (frontend `4ca16d02`), suma un caso de facturas: "the record of a purchase invoice registers the arrival of its units that have not arrived" ([especificación de habilitación](especificacion-habilitacion-2026-11-02.md) §7.6). Tampoco corrió, por la misma razón.

**Al 04/11/2026** la sonda sigue sin correr, por la misma razón: sus 12 casos, el que prepara la corrida, los ocho numerados y los tres de arriba que no llevan número. La revisión no la cambió (§10).

### 7.5 Lo que corrió al integrar (03/11/2026)

- **Backend:** la suite completa, 3 090 pruebas, sobre `erp_mm_db_verify` con la cuenta `postgres`. Solo fallan las que se esperan con un superusuario, que se salta Row Level Security.
- **Frontend:** las pruebas unitarias, y las e2e de facturas de compra, órdenes de compra, flota y tipos de cambio.
- **Carreras**, con dos transacciones reales: `PurchaseInvoiceLockTests` (el mismo número, la misma unidad, el mismo VIN nuevo, el proveedor desactivado mientras tanto, y una anulación contra un cambio de la misma factura), `PurchaseOrderLockTests.anAnnulmentAndALinkOfTheSameOrderDoNotCross`, `VehicleLockTests.aYearEditAndALinkOfTheSameUnitDoNotCross` y `VehicleAnnulmentLockTests.anAssignmentAndAnAnnulmentOfTheSameUnitDoNotCross` (PRC-R60).
- **Disparadores:** `PurchaseInvoiceIscTriggerTests` y `PurchaseInvoiceFiscalTriggerTests` (PRC-R39, PRC-R53). El de la columna del ISC de la cabecera no se alcanza con un `UPDATE` directo, porque el disparador fiscal lo rechaza antes (§3.4).
- **Límites de los archivos:** `PurchaseInvoiceUploadLimitsTests`, con el PDF en 5 MB y el XML en 2 MB (PRC-R57).
- **Rendimiento:** la línea base y las sentencias de §7.2.

---

## 8. Secuencia de commits

Ramas `feature/procurement-invoices` en los dos repositorios, desde `develop`, en worktrees hermanos. Cada commit compila y pasa las pruebas que alcanza, siempre con `ModularityTests`; con tablas nuevas, también las de RLS y de filas que no se borran.

**Documentación**
1. `docs(conventions): add the contract of purchase invoices` — 5.0. Pasa §4 a las convenciones, con D-178 a D-182.

**Backend**

| # | Commit | Paso | Depende de |
|---|---|---|---|
| B1 | `feat(iam): seed the permissions of purchase invoices for the superadministrator` | 5.1 | — |
| B2 | `feat(location): add the purchase exchange rate type of each country` | 5.2 | — |
| B3 | `feat(location): offer the effective rate of a day and the tax rate of a country through the facade` | 5.2 | B2 |
| B4 | `refactor(fleet): extract VehicleRegistrations from the vehicle command service` | 5.3 | — |
| B5 | `feat(vehiclecatalog): read models and colors by public ids and mark several models in use` | 5.3 | — |
| B6 | `feat(organization): read the purchasing company and commercial brands in batches` | 5.3 | — |
| B7 | `feat(fleet): open FleetContextFacade with the match and the batch registration of purchased units` | 5.3 | B1, B4, B5, B6 |
| B8 | `feat(fleet): declare UnitPurchases and lock the model year of an invoiced unit` | 5.3 | B7 |
| B9 | `feat(fleet): flag and filter the units without a current invoice` | 5.3 | B8 |
| B10 | `feat(fleet): annul a unit through the explicit path of D-104` | 5.4 | B8 |
| B11 | `feat(fleet): refuse every write on an annulled unit` | 5.4 | B10 |
| B12 | `feat(procurement): create the schema of purchase invoices and their lines` | 5.5 | — |
| B13 | `feat(procurement): implement CurrencyUsage and UnitPurchases with purchase invoices` | 5.5 (puertos de 5.3) | B8, B12 (M5) |
| B14 | `refactor(shared): read the type of an uploaded file from its content in shared` — `UploadedFileType` en `shared`, y la plantilla delega en él. Si la fase 3 se integró antes, ya existe y este commit no hace falta | 5.6 | — |
| B15 | `feat(procurement): model the purchase invoice with its lines, amounts, number and dates` | 5.6 | B14 |
| B16 | `feat(procurement): persist purchase invoices and translate their races` | 5.6 | B12, B15 |
| B17 | `feat(documents): discard the object of a kept file whose transaction rolled back` | 5.6 | — |
| B18 | `feat(procurement): register a purchase invoice with its units in one transaction` | 5.6 | B3, B7, B13, B16, B17 |
| B19 | `feat(procurement): check the VINs of an invoice before it is registered` | 5.6 | B18 |
| B20 | `test(procurement): measure the registration of a hundred lines and fix its statements` | 5.6 | B18 |
| B21 | `feat(procurement): annul a purchase invoice with its reason and its row in the change log` | 5.7 | B18 |
| B22 | `feat(procurement): change the file and the order of a current purchase invoice` | 5.7 | B18 |
| B23 | `feat(procurement): list, read and export purchase invoices with their allowed actions` | 5.8 | B21, B22 |
| B24 | `feat(procurement): tell what an order lacks by model and year and keep an invoiced order from annulment` | 5.8 | B18 |
| B25 | `feat(procurement): filter purchase orders by supplier` | 5.8 | — |

**Frontend**

| # | Commit | Paso | Depende de |
|---|---|---|---|
| F1 | `test(e2e): split the API mocks by context` | 5.11 (va primero) | — |
| F2 | `test(e2e): move the under-construction screen to credit notes` | 5.11 (va primero) | F1 |
| F3 | `feat(procurement): model purchase invoices and their amounts in integers` | 5.9 | Documentación 1 |
| F4 | `feat(procurement): parse pasted columns without guessing thousands` | 5.9 | F3 |
| F5 | `feat(procurement): add the API and the store of purchase invoices` | 5.9 | F3 |
| F6 | `feat(location): propose the purchase rate and register a missing one in fixed mode` | 5.9 | B3 |
| F7 | `feat(procurement): add the invoice form with its header, rate and ISC switch` | 5.10 | F5, F6, facturas reales (plan §3.2) |
| F8 | `feat(procurement): add the lines table with paste preview and VIN statuses` | 5.10 | F4, F7, B19 |
| F9 | `feat(procurement): confirm the order deviations and guard unsaved work` | 5.10 | F8 |
| F10 | `feat(procurement): list and export purchase invoices` | 5.11 | F5, B23 |
| F11 | `feat(procurement): show the invoice record, annul it and copy it` | 5.11 | F9, F10 |
| F12 | `feat(procurement): change the file and the order of an invoice` | 5.11 | F11 |
| F13 | `feat(procurement): show what an order lacks and refuse annulling an invoiced one` | 5.11 | B24, B25 |
| F14 | `feat(fleet): flag the units without a current invoice and annul a unit from its record` | 5.11 | B9, B11 |
| F15 | `feat(app-shell): open Logística › Facturas` | 5.11 | F10 |
| F16 | `test(e2e): cover purchase invoices on three breakpoints with axe` | 5.11 | F15 |
| F17 | `test(live): probe purchase invoices against the real backend` | 5.12 | todo |

**Cierre (5.12), documentación**
2. `docs(procurement): close purchase invoices with their rules and commits` — `contextos/procurement.md` (PRC-R31 a PRC-R67 en ✅, la decisión 5 del mapa cerrada) y `contextos/fleet.md` (FLT-R48 a FLT-R54).
3. `docs(processes): mark P4 and the screen of invoices as built` — procesos P4 y §3.2; plan; §2.7 del diccionario sobre el descarte (solo con la aprobación del responsable, que es quien cambia el diccionario); la línea base de §7.2 y las sondas.

**Migraciones:** B1, B2 y B12 llevan V(n), V(n+1) y V(n+2), numeradas al integrar en `develop` después de V91 y de lo que la fase 3 haya integrado antes (plan §4).

**Como se integró (03/11/2026).** La fase 2 se integró antes que la fase 3, así que sus migraciones son V92 (`seed_iam_permissions_of_purchase_invoices`), V93 (`add_country_purchase_exchange_rate_type`) y V94 (`create_procurement_purchase_invoices_schema`). Merges en `develop`: backend `16f69af3` (rama hasta `c37dda82`) y frontend `2dc6505c` (rama hasta `b70af927`). La documentación 1, 2 y 3 se hizo el mismo día, salvo dos cosas: el texto de §2.7 del diccionario sobre el descarte, que el responsable aprobó el mismo día (documentación `caaada3`), y las sondas: su commit F17 llegó el mismo día, en `10696eed` y `d126d8a2`, y no corrieron (§7.4).

**Las correcciones de la revisión (04/11/2026)** no siguen esta secuencia: van en la rama `feature/review-findings` de cada repositorio, integrada en `develop` en `44136ae1` (backend) y `cc782649` (frontend), sin migraciones. §10 dice cada una con su commit. **Las de la segunda revisión y el contrato C10** van en `feature/review-2-findings`, integrada el mismo día en `851b4d09` (backend) y `3a49f141` (frontend), también sin migraciones (§10.4, §11).

---

## 9. Preguntas abiertas

No queda ninguna. Las ocho preguntas de la versión 1 (02/11/2026) quedaron decididas el 03/11/2026, por delegación del responsable, y se registran en el diccionario como D-178 a D-182. Esta versión las aplica, cada una donde corresponde.

| # | Pregunta de la versión 1 | Decisión |
|---|---|---|
| P1 | ¿`purchase_invoices` lleva `updated_at` y `updated_by`, para el candado de `updatedAt` del cambio de archivo u orden y de la anulación? | **D-178.** Sí, con la clase base de D-42. Cambiar el archivo, el XML o la orden y anular llevan el `updatedAt` que leyó quien opera; uno viejo se rechaza como en D-138, con `422 PURCHASEINVOICE_OUTDATED` |
| P2 | ¿Anular y cambiar archivo u orden exigen el mismo alcance que registrar: toda la empresa y la marca de cada línea? | **D-178.** Sí: la empresa y la marca comercial de cada línea, sin sede |
| P3 | ¿Una línea que vincula una unidad existente con otro color o motor se rechaza o se ignora? | **D-179.** Se ignora: la unidad conserva los suyos y el formulario los muestra bloqueados. Cambia la recomendación: si el motor o el color escritos o pegados difieren de los de la unidad, o la unidad no tiene motor, la línea lleva un aviso que no bloquea y dice que conserva los datos de la unidad, para corregirla en Inventario (§6.3, PRC-R67). El backend ignora esos campos en una unidad existente |
| P4 | ¿El valor de factura de una línea debe ser mayor que cero? | **D-179.** Sí, en la base y en la API |
| P5 | ¿En una boleta u otro comprobante, la exportación muestra base igual al total e IGV cero? | **D-180.** Sí. En una factura, la base y el IGV salen de la tasa; las tres columnas calculadas llevan «(calculado)» |
| P6 | ¿El `PUT` de la factura también agrega o reemplaza el XML, además del archivo? | **D-178.** Sí: el `PUT` cambia el archivo, el XML y la orden, y nada fiscal |
| P7 | ¿Qué tipo de tasa de compra llevan los países distintos de Perú y los que se creen después? | **D-181.** Perú, `venta`; todo otro país, `oficial` por omisión, valor que ya tienen `ExchangeRateType` y el `CHECK` de V91. De solo lectura en la API del país hasta que haga falta |
| P8 | ¿Una unidad anulada todavía deja cerrar su cesión abierta a un fondo? | **D-182.** Cambia la recomendación: una unidad con una cesión abierta no se anula; primero se cierra la cesión (`422 VEHICLE_FUND_ASSIGNMENT_OPEN`, FLT-R54). Así la guarda de la unidad anulada vale para toda escritura, sin excepción (FLT-R49) |

**El hallazgo E15 del plan** se mantiene como lo describe §2.5.5: `KeptFileStore.discardRolledBack`, construido el 03/11/2026, y desde el 04/11/2026 registrado solo para una llave que la transacción escribió primero. El texto de §2.7 del diccionario lo recoge desde el 03/11/2026 (documentación `caaada3`). La frase que la revisión halló falsa para un `PUT`, «Es seguro sin leer la tabla», la corrigió la v1.39, aprobada por el responsable el 04/11/2026 (documentación `7cc29f4`).

---

## 10. Revisión de punta a punta (03 y 04/11/2026)

§10.1 a §10.3 son de la primera revisión; §10.4 y §10.5, de la segunda, del mismo 04/11/2026.

El responsable pidió una revisión crítica de punta a punta del plan v4, y después ordenó «Resuelve todos los hallazgos como experto». Cuatro revisores, uno por fase, leyeron el código; cada hallazgo clave se verificó en el código antes de corregirlo. Las decisiones tomadas bajo esa orden se llaman C1 a C8, y el [plan](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) las reúne. Los hallazgos llevan el código que les dio la revisión y que citan las pruebas (B10, B12, …). Las correcciones se integraron en `develop` el 04/11/2026: backend `44136ae1` y frontend `cc782649`.

### 10.1 Lo que halló en esta fase y cómo se cerró

| Hallazgo | Corrección | Commits | Pruebas |
|---|---|---|---|
| **B10.** El descarte al revertir borraba de Spaces el archivo de una factura confirmada cuando un `PUT` revertido enviaba los mismos bytes o un archivo anterior. §2.5.5 decía que era seguro | `KeptFileStore.put` dice si escribió la llave primero, por el `RETURNING (xmax = 0)` de su fila, y solo entonces se registra el descarte; igual en las notas y en los documentos de la habilitación (§2.5.5, PRC-R57) | Backend `7ad20a4a` | `PurchaseInvoiceFilesTests.theSameBytesSentAgainAreNeverDiscarded`, `aFormerFileSentBackIsNeverDiscarded`; `KeptFileStoreUpsertTests`; `SpacesKeptFileStoreTests.aPutTellsWhetherItsUpsertInsertedTheRow` |
| **C8.** El formulario ofrecía las empresas donde un rol de sede tenía el permiso, y el backend rechazaba ese registro; no retenía las líneas de una marca fuera del permiso ni miraba por marca el permiso de facturas antiguas | La sesión dice por empresa `wholeCompanyPermissions` y `brandPermissions`; el formulario ofrece solo las empresas donde se registra sin sede, lo dice si no hay ninguna, retiene las líneas de otra marca y mira por marca las facturas antiguas (§6.3, PRC-R35, PRC-R47) | Backend `df6b7276`; frontend `3742271f`, `fdf14036`, `18e03774`, `5fad3ceb`, `c8b1d359`, `1ef10642` | `AuthenticatedUserIntegrationTests.theSessionTellsTheReachOfEachPermissionInACompany`; `purchase-invoice-form-view.spec`; e2e de C8 (§7.3, 17 a 19) |
| **C4.** Sin el permiso de facturas antiguas en la marca de una línea respondía un `403` genérico | Responde `422 PURCHASEINVOICE_TOO_OLD` con `context.ageInDays`, como la fecha (§4.2.4, paso 10) | Backend `5f0a01b1` | `PurchaseInvoicesIntegrationTests.anOldInvoiceWithThePermissionOfAnotherBrandOnlyAnswersTooOld` |
| **B12.** Los años que una factura trae a una línea de la orden sin año se comparaban uno por uno, y una desviación pasaba sin preguntar | Se suman antes de comparar (§2.2, PRC-R50) | Backend `10850e34` | `OrderInvoicingPolicyTests.theYearsAnInvoiceBringsIntoALineWithoutYearAreAddedUp` |
| **B15.** Las líneas que vinculan unidades existentes leían y refrescaban cada unidad, dos veces: las sentencias crecían con ellas, y la prueba de sentencias no lo veía | `VehicleRepositoryImpl.readAllLocked` las lee en una sentencia (§2.3, PRC-R65, FLT-R52) | Backend `61ebde79` | `PurchaseInvoiceRegistrationStatementsTests.theQueriesOfARegistrationThatLinksExistingUnitsDoNotGrowWithItsLines` |
| **B17.** Una línea de concepto, que la base admite, respondía `500` al leer la factura | Se lee y las reglas de las unidades la pasan por alto; construirla sigue sin soporte (§2.2, PRC-R36) | Backend `a7afefd8` | `ConceptLinesTests.aConceptIsNoUnitOfTheInvoice` |
| **C7.** La búsqueda no encontraba el número escrito con ceros o separadores | `q` encuentra también el texto normalizado como número (§4.2.1, PRC-R32) | Backend `53f11cd3` | `PurchaseInvoiceReadsIntegrationTests.theFiltersNarrowTheListAndAValueOutOfItsListIsRefused` |
| **B20.** Pruebas más débiles que su regla: la de mañana en Lima dependía de la hora de la suite; el fallo en la línea 100 nunca escribía nada antes de fallar; la tasa 1, la línea de cero, la fila delicada y el redondeo de la exportación no se leían en la base o en el archivo | `ProcurementCalendar` reemplaza el `Clock.systemUTC()` fijo, y cada prueba lee lo que su regla dice (PRC-R34, R38, R40, R56, R57, R58, R65) | Backend `6165ba5d`, `7984202f` | `ProcurementDatesInLimaTests`; `PurchaseInvoiceRollbackTests`; `PurchaseInvoiceAnnulmentsTests`; `PurchaseInvoicesIntegrationTests.aLineOfZeroIsRefusedByTheApiAndByTheDatabase`; `PurchaseInvoicesExportIntegrationTests.theTotalInTheCurrencyOfTheCompanyRoundsHalfUp` |
| «Cambiar archivo u orden» seguía, después de `PURCHASEINVOICE_OUTDATED`, con la factura con que se abrió | Toma la leída otra vez, con su orden y su `updatedAt`, y conserva los archivos elegidos (§4.6) | Frontend `ea714437` | `edit-purchase-invoice-dialog.spec` |
| «Orden de compra» leía solo las 100 órdenes enviadas más recientes, y los proveedores se leían todos de una vez | Las órdenes y los proveedores se buscan en el servidor; la orden de una copia y el proveedor de un enlace se leen por su id (§6.2, §6.3) | Frontend `6c779bc3`, `3a995e95`, `9f011f06` | `purchase-invoice-form-view.spec`, `edit-purchase-invoice-dialog.spec`, `purchase-orders.store.spec`; e2e 16 |
| Sin los países leídos, el formulario tomaba `oficial` como tipo de tasa | No lee ninguna tasa hasta saberlo, lo dice y ofrece «Reintentar» (§6.3, PRC-R41) | Frontend `add2c336` | `purchase-invoice-form-view.spec` ("never guesses the kind of rate: without the countries it reads none, says so and reads the countries again") |
| La pantalla rechazaba ceros más allá de los decimales de la moneda, que el backend acepta | Los acepta, como el backend (PRC-R38) | Frontend `6386f2fa` | `purchase-invoice-amounts.spec` ("takes zeros past the decimals of the currency, as the backend does") |
| «Pegar columnas» dejaba cada línea sin marca, modelo ni año | «Para todas las líneas (opcional)», revisión 39 de `DESIGN.md` (§6.3) | Frontend `8be386af` | `paste-columns-dialog.spec` ("gives every pasted line the vehicle brand, model and year chosen for all of them (revision 39)"); e2e 15 |
| §2.4 ponía las marcas de las unidades nuevas junto con la empresa y la orden en `FOR UPDATE` | Solo el texto: el orden real, con las marcas al final y la orden en `FOR NO KEY UPDATE` (§2.4) | — | Las de PRC-R60, sin cambio |

Las citas de pruebas que quedaron viejas con las fases 4 y 5 también se corrigieron: PRC-R53, PRC-R54, PRC-R64, FLT-R52 y §7.1.

**Lo que no cambió, por decisión.** El agrupamiento global de JDBC: las inserciones siguen siendo una por fila (§2.3), y lo que no crece con las líneas son las demás sentencias.

### 10.2 Lo que corrió el 04/11/2026

- **Backend:** la suite completa, 3 570 pruebas, sobre `erp_mm_db_verify` con la cuenta `postgres`: fallan 78, todas de la línea base del superusuario (las de Row Level Security y de filas que no se borran, que necesitan las dos cuentas).
- **Frontend:** 3 285 pruebas unitarias en 275 archivos; stylelint limpio; la e2e completa de escritorio, 657 pasan, 3 se saltan y ninguna falla; axe limpio en las tres anchuras en las pantallas que cambiaron. No se regeneró ninguna imagen de referencia.

### 10.3 Lo que queda abierto

- **La sonda en vivo** `e2e-live/purchase-invoices.live.spec.ts`, 12 casos, nunca corrió: necesita la contraseña de verificación del responsable (§7.4).
- **La línea base de 100 líneas** se midió con la cuenta `postgres`, que no pasa por Row Level Security: mediana 1 731 ms, máximo 2 063 ms. El máximo pasa el objetivo provisional de 2 s, y falta medirla con la cuenta de la aplicación (§7.2).
- ~~**La integración continua con las dos cuentas:** las pruebas de Row Level Security y de filas que no se borran de las tablas nuevas no han corrido en verde allí.~~ Una sonda de SQL revertida, como `erp_mm_app` sobre `erp_mm_db_verify` (04/11/2026), confirmó en las 11 tablas nuevas de empresa la Row Level Security forzada, la falta de `DELETE` y el aislamiento entre empresas. Resuelto el 09/11/2026 (rama `fix/suite-under-two-accounts`, integrada en `integration/audit-2026-11-09`): la suite completa pasa con `erp_mm_app` y `erp_mm_owner`, 3 846 pruebas, 0 fallos, 0 errores y 2 omitidas, y esa es la línea base desde entonces. De la línea base de `develop` `d7c9b007` con esas cuentas, 13 fallos y 7 errores, ninguno era de esta fase ni del producto: eran defectos de prueba que el superusuario ocultaba.
- ~~**El texto de §2.7 del diccionario** sobre el descarte (§2.5.5): corregirlo es del responsable.~~ Aprobado por el responsable y aplicado el 04/11/2026: documentación `7cc29f4`, diccionario v1.39.
- **La validación del contador** no toca esta fase: las decisiones que esperan su visto bueno son de las notas y los pagos.

### 10.4 La segunda revisión (04/11/2026): lo que halló en esta fase y cómo se cerró

Con la primera revisión integrada, cuatro revisores independientes volvieron a leer el plan v4 de punta a punta, y el responsable sumó tres pedidos de pantalla. Todo se integró en `develop` el 04/11/2026, sin migraciones: backend `851b4d09` (de `a126216f` a `607e6756`) y frontend `3a49f141` (de `a8ccc252` a `c64888e9`). El [plan](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) §2.11 reúne las cuatro fases. La severidad sigue la escala del plan §2.

| Severidad | Hallazgo | Corrección | Commits | Pruebas |
|---|---|---|---|---|
| Medio | El formulario no retenía una línea que vincula una unidad de una marca que el permiso no alcanza: leía el valor del formulario, que deja fuera los campos deshabilitados de esa línea, y el backend la rechazaba con `403`. «Marca comercial por omisión» ofrecía marcas donde la sesión no registra | Cuenta la marca que el registro manda, la de la unidad; la marca por omisión ofrece solo las de la sesión (§6.3, PRC-R47) | Frontend `cbf35bf6` | `purchase-invoice-form-view.spec` ("holds back a line that links a unit of a brand the registration does not reach, by the brand of the unit (C8)", "offers as the default commercial brand only the brands where the session registers invoices (C8)") |
| Bajo | Una línea de concepto respondía `500` al anular la factura o al cambiar su orden: la primera corrección (B17) no alcanzó `brandsOf` | Deja fuera las marcas nulas (§2.2, PRC-R36) | Backend `76fa8f0d` | `PurchaseInvoiceChangesIntegrationTests.anInvoiceWithAConceptChangesItsOrderAndIsAnnulled` |
| Bajo | En Spaces, una fila de `kept_files` que fallaba después de escribir el objeto lo dejaba en el bucket, sin fila y sin descarte | Primero la fila, después el objeto (§2.5.5, PRC-R57) | Backend `5df515b8` | `SpacesKeptFileStoreTests.aRowThatFailsToWriteLeavesNoObject`, `theObjectIsWrittenAfterItsRowAndItsFailureFailsThePut` |
| Bajo | Las marcas de un rechazo seguían la posición de la línea, y quitar una anterior las pasaba a otra; un formulario nuevo mostraba las que dejó otro | Se marcan por la línea misma; un formulario nuevo las olvida; al cambiar de empresa, las líneas empiezan con sus marcas (§6.3) | Frontend `10b31b28` | `purchase-invoice-lines.spec` ("keeps the mark of a refused line on that line when a line before it is removed"); `purchase-invoice-form-view.spec` ("stay on their lines when a line before them is removed", "are never shown by a new form: it forgets those another form left in the store", "starts the lines on the brands of another company when the company changes (B5)") |
| Bajo | Una revisión del VIN que respondía después de cambiar de empresa se aplicaba igual | Se descarta (§6.3) | Frontend `36b4d052` | `purchase-invoices.store.spec` ("drops the answer of a check the verdicts were forgotten during, as another company was chosen") |
| Bajo | El selector buscado en el servidor no volvía a preguntar un texto escrito después de elegir; un proveedor o una orden leídos para las opciones de una empresa que cambió mientras tanto quedaban entre las siguientes | Pregunta otra vez sus primeras opciones al elegir, y lo leído para opciones que se borraron no se guarda | Frontend `842c98b5`, `21cf2277` | `search-select.spec` ("asks again for a text typed after choosing, though the page searched by itself meanwhile"); `searched-options.spec` ("never lands among the options of the company chosen while it was read", "never keeps the order a copy names among the options of the supplier chosen while it was read", y dos más) |
| Bajo | El ingreso de las unidades desde la ficha de la factura no distinguía una unidad anulada mientras tanto de una que ya ingresó | La anulada ya no se lista, y lo dice | Frontend `cb2b7efd` | `register-invoice-arrival-dialog.spec` ("tells a unit annulled meanwhile from one that arrived: it is no longer listed") |
| Bajo | Pruebas más débiles que su regla, o que dejaban datos: `PurchaseInvoiceRollbackTests` confirmaba una empresa en un país de prueba nuevo en cada corrida; el candado de las unidades que se vinculan (`readAllLocked`) no se probaba con transacciones reales; el `put` repetido de una llave, solo dentro de la misma transacción | La prueba del fallo en la línea 100 registra sobre la escena de las carreras; el candado, con dos transacciones reales; el `put`, después de una transacción confirmada | Backend `607e6756`, `dd225f1a`, `eed0c204`, `5f967ae3` | `PurchaseInvoiceRollbackTests.aFailureOnLineOneHundredLeavesNoRowAndNoFile`; `FleetContextFacadeLockTests.aUnitReadBeforeTheLockAndChangedMeanwhileIsReadAgainOnceLocked` (FLT-R52); `KeptFileStoreUpsertTests.aKeyAnEarlierTransactionCommittedIsNotWrittenFirstAgain` |

**Los pedidos del responsable (04/11/2026):**

| Pedido | Cómo quedó | Commits |
|---|---|---|
| La barra de una lista con varios filtros | Revisión 40 de `DESIGN.md`: la búsqueda y las acciones en una fila, los filtros en otra, «Limpiar filtros» y, en el teléfono, «Filtros» (§6.2) | Frontend `89f9f6c5`, `83a83cce`, `71ccb43c`, `bebbab4f` |
| Importar las líneas desde Excel, solo en las facturas de compra | Contrato C10: la plantilla como documento generado y una importación que no guarda nada (§11); reemplaza «Pegar columnas» | Backend `6eee6878`, `d3b1e6ae`, `23fb6bba`; frontend `1184be6f`, `6a25fabe`, `4e7d440b`, `3d5387b9`, `1a1775d7` |
| La tarjeta de datos en grupos | Revisión 41: Comprobante, Importes, Orden de compra y Archivos (§6.3) | Frontend `f6510432`, `94c1c6a3` |

**Lo que se comprobó y estaba bien en esta fase:** el contrato entre el frontend y el backend, los candados de §2.4 y las pruebas que citan §5 y §7, que existen todas.

### 10.5 Lo que corrió el 04/11/2026, después de la segunda revisión

- **Backend:** la suite completa dos veces seguidas, 3 619 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`: fallan 78, todas de la línea base del superusuario, y ninguna corrida gastó un país de prueba.
- **Frontend:** 3 327 pruebas unitarias en 275 archivos; stylelint limpio; la compilación, bien. La e2e completa de escritorio: 659 pasan, 3 se saltan y una falla de forma intermitente, la del armazón (`e2e/shell.spec.ts`), que medía la barra lateral y la superior a mitad de su transición y ya fallaba así en `develop`. `c64888e9` las mide cuando las dos se asientan, y pasó 116 de 116 en cuatro repeticiones, en escritorio y en tableta.
- **Las capturas** de las barras nuevas y de las tarjetas en grupos, a 1 440 y a 375 px, se revisaron a ojo.

Lo que queda abierto en esta fase sigue siendo lo de §10.3.

---

## 11. Importar líneas desde Excel (contrato C10, 04/11/2026)

**Por qué.** El 04/11/2026 el responsable pidió llenar las líneas de una factura desde un Excel, y solo en las facturas de compra. La coordinación lo llamó contrato C10. Reemplaza «Pegar columnas» (revisión 34, punto 2, y revisión 39 de `DESIGN.md`) y lo dibuja la revisión 41. No cambia el diccionario: no guarda nada, no es una carga de archivos —no escribe `import_batches` ni deja informe de errores (D-200, D-208)— y la plantilla es un documento generado (D-106).

**Commits:** backend `6eee6878` (la fachada de `vehiclecatalog`), `d3b1e6ae` (la plantilla y la importación) y `23fb6bba` (la plantilla como documento generado); frontend `1184be6f`, `6a25fabe`, `4e7d440b`, `3d5387b9` y `1a1775d7`, con la revisión 41 (`f6510432`). Reglas: PRC-R155 a PRC-R157 (§5.1).

### 11.1 La plantilla, un documento generado

`GET /api/v1/purchase-invoices/line-template` con el `Accept` de una exportación (convenciones §8.1). Pide `compras.registrar_factura`; sin él, `403 ACCESS_DENIED`. Responde `303 See Other` a `/api/v1/documents/{id}`, como cualquier exportación: un pedido igual reusa el documento, y uno que llega con muchos documentos generándose responde `503 DOCUMENTS_BUSY`.
- **Tipo:** `purchase-invoice-line-template` (`ProcurementDocumentKinds.PURCHASE_INVOICE_LINE_TEMPLATE`).
- **Nombre:** `export.purchase-invoice-line-template.name`, en español «plantilla-lineas-factura»; el archivo se descarga como `plantilla-lineas-factura-AAAA-MM-DD.xlsx`.
- **Alcance:** tener `compras.registrar_factura` (`DocumentScopes.holding`): el mismo archivo para todos los que lo tienen.
- **Versión:** la del diseño de la plantilla (`line-template-layout=2`) y la de los datos de `vehiclecatalog`, que cuenta tipos de vehículo, marcas, modelos y colores. Un cambio del catálogo genera otra plantilla.

**Como se cambió (05/11/2026, decisión del responsable; backend `7504e45f`, frontend `da92edb6`):** la plantilla dice qué columnas son obligatorias y trae un ejemplo. El ejemplo toma los primeros nombres del catálogo, así que se importa tal cual, y va en «Instrucciones», ahora la segunda hoja, porque toda fila de «Líneas» que no está en blanco se importa: uno olvidado ahí sería una unidad falsa.

La escribe `FastexcelInvoiceLineWorkbooks` con fastexcel, la biblioteca de las exportaciones; el proyecto no usa Apache POI. Tiene tres hojas:

| Hoja | Contenido |
|---|---|
| «Líneas» | La cabecera en negrita y fija, sin filas de datos. Cada columna dice si una fila la necesita: VIN, Color, Marca, Modelo, Año y Valor llevan «(obligatorio)», y Motor e ISC, «(opcional)» y otro color. Sus 1 000 primeras filas llevan el formato de cada columna: VIN y Motor como texto, para que Excel no los lea como números; Valor e ISC con dos decimales (`0.00`); Marca y Color, con la lista de su columna de «Catálogo», que avisa sin impedir |
| «Instrucciones» | Una unidad por fila, desde la fila 2, hasta 100. Una tabla con cada columna: si es obligatoria u opcional, un ejemplo y qué poner —el VIN de 17 caracteres sin I, O ni Q; el motor, de hasta 50; los nombres como en «Catálogo», sin importar mayúsculas ni tildes; el año, de 2017 a 2100; el valor con impuestos, en la moneda de la factura, con punto decimal y hasta dos decimales; el ISC solo si la factura lo incluye—. La marca comercial se elige en el formulario, y nada se guarda hasta registrar la factura |
| «Catálogo» | Marca, Modelo y Tipo de vehículo de cada modelo que una unidad nueva puede tomar —activo, de una marca activa y de un tipo de vehículo activo (D-108)—, por marca y nombre; y aparte las columnas Marcas y Colores, que alimentan las listas |

`vehiclecatalog` da los modelos y los colores por su fachada (§2.5.3).

### 11.2 La importación

`POST /api/v1/purchase-invoices/line-import`, multipart con la parte `file`, con `compras.registrar_factura`. Lee la primera hoja del libro y responde sus líneas para que el formulario las agregue. No guarda nada.

**Las revisiones, en este orden** (`PurchaseInvoiceLineImportQueryServiceImpl`):
1. Sin archivo, o vacío: `400 VALIDATION_ERROR` en `file`. Uno que pasa el límite de las subidas responde `413` antes.
2. No es un libro `.xlsx` por sus bytes —un zip con `xl/workbook.xml`—: `422 PURCHASEINVOICE_IMPORT_FILE_TYPE`. Un `.xls` o un PDF responden lo mismo.
3. Más de 2 MB: `422 PURCHASEINVOICE_IMPORT_FILE_SIZE`, con `context.maxBytes = 2097152`.
4. Antes de leerlo, el libro se infla una vez, con un techo de 32 MB para todas sus partes juntas. Uno que lo pasa, un zip roto o uno que no se lee responden `PURCHASEINVOICE_IMPORT_FILE_TYPE`.
5. La cabecera de la primera hoja no es la de la plantilla: `422 PURCHASEINVOICE_IMPORT_HEADERS`, con `context.expected` y `context.found`. Se compara sin mayúsculas, tildes ni espacios de más; valen las ocho columnas, o las siete primeras sin ISC, con «(obligatorio)» u «(opcional)» o sin ellos, como en una plantilla anterior al 05/11/2026; una columna de más se rechaza.
6. Más de 100 filas que no están en blanco: `422 PURCHASEINVOICE_IMPORT_ROWS`, con `context.max = 100` y `context.found`.

**La respuesta, `200`:**

```json
{
  "lines": [
    { "row": 2, "vin": "MD2A11CY6LWA12345", "engineNumber": "JLZCCL12345",
      "colorId": "0199…", "colorName": "NEGRO", "vehicleBrandName": "BAJAJ",
      "modelId": "0199…", "modelName": "DISCOVER 125-ST", "modelYear": 2026,
      "amount": 5900.00, "iscAmount": null }
  ],
  "errors": [
    { "row": 3, "column": "VIN", "code": "VIN_REPEATED", "value": "MD2A11CY6LWA12345" }
  ]
}
```

- Las filas en blanco se saltan. Una fila con algún error no da línea, y cada error suyo se lista con su fila (la cabecera es la 1), su columna por su encabezado, su código y el texto de su celda, o `null` si estaba vacía.
- `vin` va en mayúsculas. `amount` e `iscAmount` van con dos decimales y sin moneda: la moneda es la de la factura, que se elige en el formulario.

**Los códigos de las filas** (`InvoiceLineImportPolicy`):

| Columna | Código | Cuándo |
|---|---|---|
| VIN | `VIN_REQUIRED`, `VIN_INVALID`, `VIN_REPEATED` | Vacío; no son 17 letras y números sin I, O ni Q; repetido en el archivo, nombrado en la fila posterior |
| Motor | `ENGINE_TOO_LONG` | Más de 50 caracteres. Vacío se admite |
| Color | `COLOR_REQUIRED`, `COLOR_NOT_FOUND` | Vacío; no es un color activo |
| Marca | `BRAND_REQUIRED`, `BRAND_NOT_FOUND` | Vacía; no es la marca de ningún modelo que una unidad nueva puede tomar |
| Modelo | `MODEL_REQUIRED`, `MODEL_NOT_FOUND` | Vacío; no es un modelo activo de esa marca. Si la marca no se encontró, el modelo no se juzga |
| Año | `YEAR_REQUIRED`, `YEAR_INVALID` | Vacío; no es un entero de 2017 a 2100 |
| Valor | `AMOUNT_REQUIRED`, `AMOUNT_INVALID` | Vacío; no es un monto mayor que cero, con punto decimal y hasta dos decimales, o pasa 999 999 999 999,99, lo que cabe en `NUMERIC(14,2)` |
| ISC | `ISC_INVALID` | No es un monto válido o pasa el Valor de su fila. Vacío, o sin la columna, la línea no lleva ISC |

Los nombres se comparan sin mayúsculas ni tildes y con los espacios de más colapsados (`InvoiceLineImportPolicy.fold`). Lo que la importación no revisa lo revisa el registro, como siempre (§4.2.4): el VIN contra las unidades (`GET /purchase-invoice-vin-checks`), los decimales de la moneda, el ISC según «Incluye ISC», el total y las 100 líneas de una factura.

### 11.3 La pantalla

La revisión 41 de `DESIGN.md` la dibuja; esto es lo que hace:
- **El botón:** «Importar desde Excel» (`upload_file`), delineado y compacto, en la barra de «Unidades», antes de «Agregar línea», donde estaba «Pegar columnas».
- **El diálogo** `import-invoice-lines-dialog`, un marco `md`, «Importar desde Excel», con tres pasos en una pantalla:
  1. **«Descargue la plantilla»:** qué lleva una fila y «Descargar plantilla», que pide el documento generado y lo descarga como el Excel de una lista. Si falla: «No se pudo descargar la plantilla. Inténtelo de nuevo.».
  2. **«Elija el archivo completado»:** `mm-image-slot` con un `.xlsx`, que nombra un libro XLSX y lee uno sin tipo por su extensión (`6a25fabe`). Elegido, se envía enseguida.
  3. **«Revise el resultado»:** «78 líneas listas · 2 filas con errores», dicho una vez, y la tabla «Filas con errores» con Fila, Columna, Valor y Motivo, un texto por código («El VIN se repite en el archivo.», «El modelo no es de esa marca o no está en el catálogo.»). Un libro rechazado entero es el error del diálogo, con los valores de su texto: el tope, las columnas esperadas y las que llegaron, el máximo de filas.
- **«Agregar 78 líneas»:** retenido mientras no haya líneas listas o mientras pasarían las 100 de una factura: «El archivo trae 120 líneas y la factura solo admite 98 más: una factura lleva hasta 100. No se agrega ninguna.».
- **Al agregar:** cada línea toma la marca comercial por omisión, como una agregada a mano, y de su fila el color, el modelo con su marca, el año, el VIN, el motor, el valor y, solo si la factura incluye ISC, el ISC. Sus VIN se revisan enseguida, en lotes de hasta 100; el diálogo se cierra y el foco va al VIN de la primera línea agregada. Cerrar no pregunta: el libro sigue en el equipo de la persona.
- **Una línea que vincula una unidad** toma la marca, el modelo y el año de su unidad (D-179), y lleva el aviso del motor o del color (PRC-R67).
- **El catálogo de componentes** («Líneas de factura») abre el diálogo sobre una respuesta de ejemplo (`1a1775d7`).

### 11.4 Pruebas

- **Backend:** `InvoiceLineImportPolicyTests`, 8 pruebas de la regla pura, y `PurchaseInvoiceLinesIntegrationTests`, 9 sobre las dos rutas: las hojas, la cabecera, los formatos y el catálogo de la plantilla; la plantilla tal como se descarga, que importa sin nada; un libro lleno, con filas en blanco; un libro sin la columna ISC; los errores de las filas; el orden de los rechazos; 100 filas leídas y 101 rechazadas; el permiso de las dos rutas; y la plantilla que un pedido igual reusa y un cambio del catálogo genera otra vez (PRC-R155 a PRC-R157).
- **Frontend:** `purchase-invoices.store.spec` ("reads the lines of a workbook, and keeps its refusal to itself with what it names (C10)", "downloads the template of the lines, and returns why it could not"); `purchase-invoices-api-endpoint.spec` ("downloads the template of the lines as a generated document, as the Excel of a list (C10, D-106)"); `import-invoice-lines-dialog.spec`, 5 casos ("downloads the template the backend builds, and says when it could not", "waits for a workbook, then sends it at once and offers to add the lines it brought", "lists the rows refused by row, column, value and reason, and adds none while no line is ready", "says why a workbook is refused whole, with what its refusal names", "adds none when the lines of the workbook would pass the hundred of an invoice, and says so"); `purchase-invoice-lines.spec` ("adds the lines of a workbook with the default brand and the brand of each model, and checks their VINs (revision 41)", "adds nothing from a workbook that would pass the hundred lines of an invoice"); `procurement-design-system-samples.spec` ("opens «Importar desde Excel» over a sample workbook, with the room the lines leave (revision 41)").
- **Extremo a extremo:** los escenarios 3, 14, 15 y 20 de §7.3 y la pasada de axe del diálogo.
- **Sonda en vivo:** `e2e-live/purchase-invoices.live.spec.ts` no cubre la importación; como el resto de la sonda, no corrió (§10.3).
