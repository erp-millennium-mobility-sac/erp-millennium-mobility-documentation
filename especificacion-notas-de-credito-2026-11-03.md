# Especificación: notas de crédito (fase 4, punto 6)

**Versión 2 · 03/11/2026** (las diez preguntas de la versión 1, del mismo día, quedan decididas por D-203 a D-205, delegadas por el responsable, y por la corrección del plan §3.2 y del diccionario §5.6) · **Estado:** borrador para validar · **Punto de partida:** backend `b1264ed5` (hasta V91), frontend `88312e13`, documentación `6ebb900`, más el diccionario v1.38 con las decisiones D-178 a D-205 del 03/11/2026, la [especificación de facturas de compra](especificacion-facturas-de-compra-2026-11-02.md) y la [de habilitación](especificacion-habilitacion-2026-11-02.md), las dos en su versión 2 del 03/11/2026, y el [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) v3. Esta fase se construye sobre la fase 2 integrada en `develop`: usa sus tablas, su agregado, sus puertos y su vía de anulación de la unidad, que hoy existen solo en su especificación. De la fase 3 no depende. Si esta especificación y el diccionario no coinciden, manda el diccionario.

**Fuentes:** diccionario §2.2, §2.5 a §2.7, §5.6, §8.1, §9.3 a §9.7, §10.4, §12.2, §13.1 y las decisiones D-23, D-42, D-84, D-101, D-102, D-104, D-134, D-146, D-152, D-157 a D-163, D-169, D-174 a D-176, D-178, D-182, D-189, D-190 a D-199 y D-203 a D-205; [convenciones de la API](convenciones-de-la-api-millennium-mobility.md); [contextos/procurement.md](contextos/procurement.md) y [fleet.md](contextos/fleet.md); [mapa de contextos](mapa-de-contextos-millennium-mobility.md) §6.1 y decisión 7; [procesos y pantallas](procesos-y-pantallas-millennium-mobility.md) P4 y §3.3; las skills `backend-module` y `frontend-feature`; la guía funcional del cliente, columnas 36 a 41 y 46 a 49, §7.1 y las decisiones de auditoría A-04 y A-06 de su §10.

**Cómo leerlo.**
- §1 dice qué decisiones implementa, qué cambia de la fase 2 y a qué pasos sirve.
- §2 es el modelo del backend: paquetes, clases, transacciones, candados, contratos entre contextos y las dos reglas de cálculo, el efecto de una nota y el costo de una unidad.
- §3 es el esquema, migración por migración.
- §4 es el contrato de la API, que el paso 6.0 pasa a las convenciones.
- §5 son las reglas con sus pruebas, todas en ❌ hasta construirse.
- §6 son las pantallas; §7, las pruebas; §8, la secuencia de commits; §9, las preguntas de la versión 1 y lo que cerró cada una.
- Lo marcado **(propuesta)** es un nombre que ninguna decisión fija.
- En la columna *Fuente* de las tablas de reglas, §*n* es del diccionario, como en los contextos. En el texto, una sección del diccionario, de las convenciones o de la especificación de la fase 2 lo dice; un §*n* sin más es de esta especificación.

---

## 1. Alcance

**Implementa** D-190 en lo que toca a esta fase (la nota resta de su factura y de su línea, y entrega esas cifras a los pagos), D-191 (una línea que queda en cero se anula; solo «anulación de unidad» anula la unidad, salvo con contratos o una cesión abierta), D-192 (la tasa en cada línea, varias facturas hasta el corte y una sola desde el 01/01/2027, con el corte configurable), D-193 (registro a mano, reparto en partes iguales, permiso y alcance, anulación con motivo, `updatedAt` y sus tres impedimentos), D-199 en sus tres primeros montos (valor de factura, notas y valor del vehículo) más el costo sin IGV y el costo en la moneda de la empresa, D-203 (ese costo en la ficha de la unidad, por el puerto `UnitPurchases`), D-204 (las reglas de la nota que D-191 y D-193 dejaban abiertas, entre ellas el cambio de su archivo) y D-205 (la fecha de la nota sin la excepción de 20 días). Aplica las ya cerradas D-23 (solo notas de crédito), D-104 (la primera vía de anulación de una unidad, que gobierna `compras.registrar_nota_credito`), D-152 y D-169 (la tasa de la factura, 1 en la moneda funcional), D-159 (el costo sin IGV), D-160 (el número normalizado), D-161 y la fase 2 (una factura con notas no se anula), D-163 y D-178 (el alcance de la empresa y de la marca de cada línea), D-174 y D-189 (el archivo, en `kept_files`), D-176 (los permisos de facturas pasan a AE, OP y CO) y D-182 (una unidad con cesión abierta no se anula).

**Sirve a los pasos de la fase 4**, que el plan resume sin numerar (plan §5, «Fase 4 · Punto 6»). Se numeran aquí como `6.x`, igual que la fase 2 usa `5.x` y la fase 3, `8.x`:

| Paso | Qué cubre esta especificación |
|---|---|
| 6.0 | §4 completo en las convenciones: rutas, cuerpos, códigos, `allowedActions` y concurrencia. Corrige también la línea de convenciones §2.2 que dice que solo «anulación de unidad» anula una línea (D-191) |
| 6.1 | §3.2: el permiso de las notas y su constante |
| 6.2 | §3.3: las dos tablas, sus disparadores, Row Level Security, permisos de la cuenta y `data_versions`; el disparador de las líneas de factura que deja volver una línea a vigente |
| 6.3 | §2.1 y §2.2: el agregado, sus objetos de valor, la política del corte y la persistencia |
| 6.4 | §2.5.1: los dos miembros nuevos de `FleetContextFacade` |
| 6.5 | §2.3 y §4.2.4: el registro en una transacción, con sus efectos sobre la factura y la unidad; el bean de `PurchaseInvoiceDependents` y `valueIsZero` (§2.5.2, §2.5.5) |
| 6.6 | §4.2.5 y §4.2.6: la anulación de la nota, la guarda del saldo que implementa la fase 5 (§2.5.3) y el cambio de su archivo (D-204) |
| 6.7 | §4.2.1 a §4.2.3, §4.2.7 a §4.2.9: lista, ficha, archivo, exportación, el corte y lo que cambia en la ficha de la factura |
| 6.8 | §2.7, §4.2.10: el costo de compra de una unidad y la unidad devuelta, por el puerto `UnitPurchases` (D-203, D-204) |
| 6.9 a 6.11 | §6 |
| 6.12 | §3.4: los permisos de facturas en AE, OP y CO (D-176), la última migración de la fase |
| 6.13 | §7.4 (sondas) y el cierre documental de §8 |

**Lo que cambia de la fase 2.** Todo es aditivo, salvo lo que se marca:
- `PurchaseInvoiceDependents` recibe su primer bean; la prueba que fijaba la lista vacía pasa a fijar exactamente uno (§2.5.5).
- `UnitPurchases` suma dos miembros, y `valueIsZero` cuenta las notas (§2.5.2).
- `PurchaseInvoiceItem` suma `reinstate()`, y la función `keep_purchase_invoice_item_fiscal_data` se reemplaza para dejar volver a vigente una línea de una factura vigente que recupera valor. Una línea de una factura anulada sigue sin cambiar nunca (§3.3, PRC-R96). **Cambia** lo que la fase 2 fijaba: «`annul(Instant)` es el único cambio que admite» (su §2.2).
- La ficha de la factura suma lo que restan las notas, su panel de notas y la acción `registerCreditNote` (§4.2.9).
- `FleetContextFacade` suma dos miembros (§2.5.1); la unidad suma `returnedToSupplier` y `allowedActions.viewPurchaseCost` (§4.2.10).
- `PurchaseInvoicePermissionsSeedMigrationTests.onlyTheSuperadministratorHoldsTheInvoicePermissions` deja de valer con V(n+2) y se reemplaza en ese commit (§3.4).
- La pantalla «En construcción» de `e2e/screens.ts`, que la fase 2 movió a `/procurement/credit-notes`, pasa a `/procurement/supplier-payments` (§7.3).

**Queda fuera:**
- **Los pagos, lo que se debe de una factura y el saldo del proveedor** (fase 5, D-190, D-194 a D-198): esta fase entrega las notas vigentes por factura y por línea (§2.5.4) y declara el puerto de la guarda del saldo, que la fase 5 implementa (§2.5.3). Lo cubierto y la diferencia de una unidad (D-199), también.
- **Las notas de débito** (D-23): no hay tipo de nota ni montos negativos.
- **«Anular y copiar» una nota**: ninguna decisión lo pide. Una nota mal registrada se anula y se registra otra; su archivo sí se cambia (§4.2.6, D-204).
- **La base y el IGV calculados de una nota** en su exportación: D-180 los fija para las facturas, no para las notas.
- **La carga inicial de las notas** (D-168, D-177): es del paquete de cargas.
- **La columna de costo del exportable de stock y los indicadores** (capa de lectura, paso 11).
- **La exportación de facturas** no cambia.

---

## 2. Modelo del backend

### 2.1 Paquetes y clases

Las capas son las de la skill `backend-module`. Lo nuevo va en piezas con nombre (F5), nunca engordando `PurchaseInvoiceCommandServiceImpl` de la fase 2.

**`procurement`** (`ky.millennium.mobility.erp.procurement`). `package-info.java` no cambia: la fase 2 ya le suma `"fleet::acl"`. Hoy declara `shared`, `organization::acl`, `location::acl`, `iam::acl`, `documents::acl` y `vehiclecatalog::acl` (`procurement/package-info.java`).

| Capa | Clases nuevas o que cambian |
|---|---|
| `domain/model/aggregates` | `CreditNote`. `PurchaseInvoice` suma `annulLineCreditedToZero(Long itemId, Instant at)` y `reinstateLine(Long itemId)` (§2.2) |
| `domain/model/entities` | `CreditNoteItem`. `PurchaseInvoiceItem` suma `reinstate()` |
| `domain/model/valueobjects` | `CreditNoteReason`, `CreditNoteAnnulmentReason`, `CampaignMonth`, `LineValue`, `CreditedTotal`. Reutiliza `InvoiceNumber` e `InvoiceAmount` de la fase 2, que normalizan un número de comprobante y validan un monto de su moneda |
| `domain/model/commands` | `RegisterCreditNoteCommand`, `CreditNoteItemInput`, `UpdateCreditNoteCommand`, `AnnulCreditNoteCommand`, `CreditNoteFileUpload` |
| `domain/model/queries` | `GetCreditNoteByPublicIdQuery`, `SearchCreditNotesQuery`, `CreditNoteSearchFilter`, `GetCreditNoteRulesQuery` |
| `domain/services` | `CreditNoteRegistrationPolicy` (las reglas que cruzan la nota con sus facturas), `SingleInvoiceCutoff` (el corte de D-192), `UnitPurchaseCostCalculator` (§2.7) |
| `domain/repositories` | `CreditNoteRepository`. `PurchaseInvoiceRepository` suma `findItemFactsByPublicIds(Set<UUID>)` (sin candado: la línea, su factura, su unidad y su marca), `findAllByIdsForUpdate(Set<Long>)` (por `id` ascendente) y `saveLineStatuses(PurchaseInvoice)`, que escribe solo el estado y `annulled_at` de las líneas y no mueve la revisión de la factura |
| `domain/exceptions` | `CreditNoteTotalMismatchException`, `CreditNoteDuplicateLineException`, `CreditNoteExceedsValueException`, `CreditNoteDateBeforeInvoiceException`, `CreditNoteSingleInvoiceRequiredException`, `CreditNoteUnitReturnPartialException`, `CreditNoteInvoiceAnnulmentPartialException`, `CreditNoteAnnulledException`; `NaturalKeyTakenException.NaturalKey` suma `CREDIT_NOTE_NUMBER` |
| `application/commandservices` | `CreditNoteCommandService` |
| `application/queryservices` | `CreditNoteQueryService`, `CreditNoteView`, `CreditNoteItemView`, `CreditNoteFile`, `CreditNoteRulesView`. `PurchaseInvoiceView` y `PurchaseInvoiceItemView` de la fase 2 suman lo de §4.2.9 |
| `application/internal/commandservices` | `CreditNoteCommandServiceImpl` y sus piezas: `CreditNoteReferences` (empresa, proveedor y moneda), `CreditedLines` (las líneas, el candado de sus facturas y lo que ya restan sus notas), `CreditNoteLineEffects` (anular o devolver a vigente las líneas de factura), `CreditNoteUnits` (las unidades devueltas, por la fachada de `fleet`), `CreditNoteFiles` (verificación, guardado y descarte al revertir, en el alta y en el cambio del archivo, como `PurchaseInvoiceFiles`), `CreditNoteAnnulments` (la anulación, sus impedimentos y la bitácora) |
| `application/internal/queryservices` | `CreditNoteQueryServiceImpl`, `CreditNoteViewFactory`, `CreditNoteAllowedActions`, `PurchaseInvoiceCreditsReader` (lo que la ficha de la factura muestra de sus notas). `PurchaseInvoiceAllowedActions` suma `registerCreditNote` |
| `application/internal/policies` | `CreditNoteAccessPolicy`; `ProcurementReadPermissions` suma `CREDIT_NOTES = compras.ver` |
| `application/internal/errors` | `CreditNoteErrors` |
| `application/internal/outboundservices/acl` | `ExternalFleetService` suma `annulReturnedUnits` y `lockUnits`, con sus records de traducción `ReturnedUnit` y `LockedUnit` |
| `application/internal/outboundservices/payments` | `SupplierBalanceAfterAnnulment` (puerto, §2.5.3) |
| `application/internal/outboundservices` | `ProcurementDataVersions` suma `creditNotes()` |
| `application/documents` | `ProcurementDocumentKinds` suma `CREDIT_NOTES = "credit-notes"` |
| `infrastructure/persistence/jpa` | `CreditNotePersistenceEntity` (extiende `RevisedAbstractPersistenceEntity`: el cuadro de §9.5 pide `created_at`, `created_by`, `updated_at` y `updated_by`, D-42 y D-193), `CreditNoteItemPersistenceEntity` (sin clase base: su cuadro no tiene columnas de registro), `CreditNoteJpaRepository`, `CreditNoteItemJpaRepository`, `CreditNoteRepositoryImpl`, `CreditNotePersistenceAssembler` |
| `infrastructure/persistence/jdbc` | `JdbcProcurementDataVersions` suma `credit_notes` y `credit_note_items` |
| `infrastructure/dependents` | `CreditNoteInvoiceDependentsAdapter` (implementa `PurchaseInvoiceDependents`, §2.5.5) |
| `infrastructure/fleet` | `PurchaseInvoiceUnitPurchasesAdapter`, de la fase 2: `valueIsZero` cuenta las notas, y suma `purchaseCostOf` y `returnedUnitIds` (§2.5.2) |
| `infrastructure/configuration` | `CreditNoteCutoffConfiguration`: el bean `SingleInvoiceCutoff`, armado con `procurement.credit-notes.single-invoice-from` (§2.2) |
| `interfaces/rest` | `CreditNotesController`, `CreditNoteRulesController` |
| `interfaces/rest/resources` | `RegisterCreditNoteResource`, `CreditNoteItemInputResource`, `UpdateCreditNoteResource`, `AnnulCreditNoteResource`, `CreditNoteResource`, `CreditNoteItemResource`, `CreditNoteRulesResource`; reutiliza `MoneyResource`. `PurchaseInvoiceResource` y `PurchaseInvoiceItemResource` suman lo de §4.2.9 |
| `interfaces/rest/transform` | `RegisterCreditNoteCommandFromResourceAssembler`, `UpdateCreditNoteCommandFromResourceAssembler`, `CreditNoteResourceFromViewAssembler`, `CreditNoteSpreadsheetColumns` |

**`fleet`**. `package-info.java` no cambia.

| Capa | Clases nuevas o que cambian |
|---|---|
| `interfaces/acl` | `FleetContextFacade` suma `annulReturnedUnits` y `lockUnitsByIds`, con `ReturnedUnitOutcome` y `ReturnedUnitHold` (§2.5.1). `UnitPurchases` suma `purchaseCostOf` y `returnedUnitIds`, con `UnitPurchaseCost`, `UnitCreditNote` y `UnitAmount` (§2.5.2) |
| `application/acl` | `FleetContextFacadeImpl`: los dos miembros, sobre `VehicleAnnulments` de la fase 2 |
| `application/internal/commandservices` | `VehicleAnnulments` (fase 2) separa su guarda en `annulmentHold(Vehicle)`, que responde la causa —anulada, valor no cero, contratos o cesión abierta— o nada, para que la usen la ruta explícita y la fachada sin repetirla |
| `application/internal/outboundservices/purchases` | `UnitPurchaseFacts` suma `purchaseCostOf` (la respuesta del único bean, FLT-R53 de la fase 2) y `returnedUnitIds` (la unión) |
| `domain/model/queries` | `GetVehiclePurchaseCostQuery` |
| `application/queryservices` | `VehiclePurchaseCostView`; `VehicleView` suma `returnedToSupplier`; `VehicleAllowedActionsView` suma `viewPurchaseCost` |
| `application/internal/queryservices` | `VehicleQueryServiceImpl` atiende `GetVehiclePurchaseCostQuery`; `VehicleViewFactory` pide `returnedUnitIds` una vez por página; `VehicleAllowedActions` calcula `viewPurchaseCost` |
| `application/internal/policies` | `VehicleAccessPolicy` suma `seesCosts(scope, companyId, commercialBrandId)`: `stock.ver_costos` lee su empresa y su marca, como `sees` con `stock.ver` (`fleet/application/internal/policies/VehicleAccessPolicy.java`) |
| `interfaces/rest` | `VehiclesController`: `GET /{id}/purchase-cost` con `@PermissionRequired(PermissionCodes.Stock.VIEW_COSTS)`, que reemplaza al de la clase (`@PermissionRequired(PermissionCodes.Stock.VIEW)`, `VehiclesController.java:81`) |
| `interfaces/rest/resources` | `VehiclePurchaseCostResource`; `VehicleResource` y `VehicleAllowedActionsResource` suman sus campos |

**`iam`, `shared`, `organization`, `location`, `documents`, `vehiclecatalog`:**

| Contexto | Cambio |
|---|---|
| `iam` | Solo las migraciones V(n) y V(n+2) |
| `shared` | `PermissionCodes.Procurement.REGISTER_CREDIT_NOTES = "compras.registrar_nota_credito"`, en el mismo commit que V(n) (`PermissionCatalogConsistencyTests`). `PermissionCodes.Stock.VIEW_COSTS` ya existe (`shared/application/security/PermissionCodes.java:100`), sembrado en V46 a SA, AE y CO |
| `organization`, `location`, `documents`, `vehiclecatalog` | Ninguno: se usan los miembros que ya existen o que crea la fase 2 (§2.5.6) |

### 2.2 El agregado y sus reglas

**`CreditNote`** es la raíz; sus líneas son entidades del agregado (`CreditNoteItem`) y se guardan con ella (`cascade` `PERSIST` y `MERGE`, nunca `ALL` ni `orphanRemoval`). Dos constructores, como pide la skill: el que registra una nota nueva validando y el que la reconstituye.

| Miembro | Regla que protege |
|---|---|
| Constructor de registro | De 1 a 100 líneas (D-204); ninguna línea de factura dos veces (`CreditNoteDuplicateLineException`); el total igual a la suma exacta de las líneas (`CreditNoteTotalMismatchException`, §9.5). Los montos llegan como `InvoiceAmount`, mayores que cero y con a lo sumo los decimales de la moneda (D-146): no hay montos negativos porque no hay notas de débito (D-23). Cada línea trae la tasa de su factura, que el servicio copia, nunca el pedido (D-192) |
| `replaceFile(String key)` | Solo una nota vigente (`CreditNoteAnnulledException`). El archivo es lo único que cambia, nada de lo registrado (D-204, como D-172 y D-178 en la factura) |
| `annul(CreditNoteAnnulmentReason, Instant, Long annulledBy)` | Una sola vez (`CreditNoteAnnulledException`). La nota anulada deja de restar en todas sus líneas a la vez: las líneas no tienen estado propio |
| `isAsOpenedAt(Instant)` | El candado de `updatedAt`, comparado al milisegundo (convenciones §3), en el cambio del archivo y en la anulación, como la factura (D-178, D-193, D-204) |
| `returnsUnits()` | Verdadero solo con «anulación de unidad» (D-191) |
| `totalInFunctionalCurrency(int decimals)` | La suma de cada línea por su tasa, redondeada cada una mitad hacia arriba a los decimales de la moneda de la empresa (§2.7). Nunca se guarda |

**`CreditNoteItem`**: `purchaseInvoiceItemId`, `purchaseInvoiceId` (no se guarda: lo trae la línea de factura y sirve al dominio), `exchangeRate`, `amount`. No admite ningún cambio.

**Objetos de valor:**

| Clase | Qué valida |
|---|---|
| `CreditNoteReason` | `DESCUENTO_CAMPANA`, `ANULACION_UNIDAD`, `ANULACION_FACTURA`, `AJUSTE_PRECIO` (D-204), como los motivos de anulación de la factura (D-161); se aceptan en cualquier mayúscula. `returnsUnits()` en `ANULACION_UNIDAD`; `annulsInvoice()` en `ANULACION_FACTURA` |
| `CreditNoteAnnulmentReason` | `ERROR_REGISTRO`, `ANULADA_POR_EMISOR` (D-193), los mismos de la factura |
| `CampaignMonth` | Un `LocalDate` con día 1 (diccionario §2.6, «un mes es un `DATE` con día 1»); otro día responde `400`. Solo acompaña a «descuento de campaña» (D-204) |
| `LineValue` | El valor de una línea de factura: `invoicedAmount` menos lo que ya restan sus notas vigentes. `after(amount)` responde el valor que queda, `exceededBy(amount)` si la nota la llevaría bajo cero y `reachesZero(amount)` si la deja exactamente en cero |
| `CreditedTotal` | Un monto restado por notas con su moneda (`amount`, `currencyId`), para lo que la fase 5 lee (§2.5.4) |

**Políticas puras** (`domain/services`):
- **`SingleInvoiceCutoff`** (D-192). Recibe la fecha de corte; `requiresSingleInvoice(LocalDate issueDate)` es verdadero desde ella, por la fecha de emisión de la nota. La fecha sale de `procurement.credit-notes.single-invoice-from` en `application.yaml`, `${PROCUREMENT_CREDIT_NOTES_SINGLE_INVOICE_FROM:2027-01-01}`, con un comentario que cita la RS 000048-2026/SUNAT y su aplazamiento por la RS 000143-2026, como `location.exchange-rates.confirmation-threshold`. Si SUNAT mueve la fecha otra vez, cambia la variable de entorno y no el código (plan §6, riesgo «SUNAT vuelve a mover la fecha»).
- **`CreditNoteRegistrationPolicy`**. Recibe la nota que se registra y, por línea, lo que se sabe de su línea de factura bajo candado (`CreditedLineFacts`: factura, fecha de la factura, tasa, `LineValue` y si la factura y la línea están vigentes) y, por factura, sus líneas vigentes. Responde, en este orden, la primera regla que no se cumple: la fecha de emisión anterior a una factura (`CreditNoteDateBeforeInvoiceException`, D-193); más de una factura desde el corte (`CreditNoteSingleInvoiceRequiredException`, D-192); una línea que bajaría de cero (`CreditNoteExceedsValueException`, D-193); con «anulación de unidad», una línea que no queda en cero (`CreditNoteUnitReturnPartialException`, D-204); con «anulación de la factura», una línea vigente de alguna de sus facturas que no queda en cero o que la nota no nombra (`CreditNoteInvoiceAnnulmentPartialException`, D-204). Responde también qué líneas quedan en cero. La antigüedad de la fecha no se mira: la excepción de 20 días de la factura no se aplica a la nota (D-205).
- **`UnitPurchaseCostCalculator`** (§2.7).

**Lo que suma `PurchaseInvoice`**, en el dominio de la fase 2:

| Miembro | Regla que protege |
|---|---|
| `annulLineCreditedToZero(Long itemId, Instant at)` | Solo en una factura vigente y sobre una línea vigente: la marca `ANULADA` con ese momento (D-191). El servicio la llama solo cuando `CreditNoteRegistrationPolicy` dice que la línea queda en cero |
| `reinstateLine(Long itemId)` | Solo en una factura vigente y sobre una línea anulada: la vuelve `VIGENTE` sin momento. El servicio la llama solo cuando la anulación de una nota le devuelve valor (PRC-R85). Una línea de una factura anulada nunca vuelve (D-161) |

**`CreditNoteAccessPolicy`** (`application/internal/policies`), sobre `AccessScope`:
- `sees(scope, companyId)`: `compras.ver` actúa en la empresa (`actsOn`). Una nota se ve por empresa, como la factura (D-163).
- `writes(scope, companyId, brandIds)`: `compras.registrar_nota_credito` actúa en toda la empresa (`actsOnWholeCompany`) y en cada marca de las líneas que nombra (`actsOnWholeCompanyBrand`), sin sede (D-193, D-163, D-178). Registrar, cambiar el archivo y anular piden lo mismo (D-204). La marca de una línea es la que copió la línea de factura (`purchase_invoice_items.commercial_brand_id`), la misma que usa la fase 2 para anular la factura (su §2.2).

### 2.3 Servicios de aplicación y transacciones

`CreditNoteCommandServiceImpl` (`@Service @Transactional`) queda envuelto por `DeadlockRetry` por su nombre y su paquete (`application/internal/commandservices`). Todo lo que hace va dentro de su transacción; lo único que sale de ella es el objeto de Spaces del archivo, que se descarta si la transacción se revierte (`KeptFileStore.discardRolledBack`, de la fase 2).

| Operación | Transacción | Qué escribe |
|---|---|---|
| `handle(RegisterCreditNoteCommand)` | Una, `REQUIRED`. La fachada de `fleet` corre dentro (`MANDATORY`) | La nota y sus líneas; el estado de las líneas de factura que quedan en cero; con «anulación de unidad», `vehicles.annulled_at` de las unidades que se anulan (por `fleet`); `kept_files` |
| `handle(UpdateCreditNoteCommand)` | Una | La nota (`file_path` y revisión), `kept_files`, la fila de la bitácora |
| `handle(AnnulCreditNoteCommand)` | Una | La nota (anulación y revisión), el estado de las líneas de factura que vuelven a vigentes, la fila delicada de la bitácora |

**El registro, paso a paso.** Cada paso responde el primer fallo y no escribe nada; el orden fija qué código sale cuando fallan varios (§4.2.4):
1. Forma del pedido (Bean Validation, `400`) y forma del archivo (`422`, sin leer la base).
2. Empresa (`fetchPurchasingCompanyForShare`, de la fase 2): existe, se lee con `compras.ver`, el permiso actúa en toda ella y está activa (D-84).
3. Proveedor de la empresa (`SupplierRepository.findByPublicId`, sin candado): no se exige activo (D-204).
4. Moneda por su código (`fetchCurrencySummaryByIsoCode`, que ya existe en `LocationContextFacade.java:47`): existe; no se exige activa (D-204). No se lee en modo compartido: sus decimales ya los fija su factura (PRC-R95).
5. Montos: escala y suma exacta (dominio).
6. Número: normalización, formato en Perú (D-204) y duplicado del proveedor entre las no anuladas.
7. Fecha: no futura en la zona de la empresa (`InvoiceDatePolicy.isFuture`, de la fase 2). Sin la excepción de 20 días (D-205).
8. Líneas repetidas dentro del pedido.
9. Las líneas de factura: `findItemFactsByPublicIds`, sin candado, para saber sus facturas; una que no existe o cuya factura no se lee responde `404`. Después, `findAllByIdsForUpdate` bloquea esas facturas por `id` ascendente, y las líneas se vuelven a leer bajo el candado con lo que ya restan sus notas vigentes (`currentCreditsByInvoiceItemIds`, §2.5.4).
10. Con lo leído bajo candado: empresa, proveedor y moneda de cada factura; línea o factura anulada; permiso sobre la marca de cada línea; y `CreditNoteRegistrationPolicy` con el corte.
11. Guarda la nota con sus líneas: la cabecera antes que las líneas, porque el disparador de cada línea lee su cabecera (§3.3). Un único que la base detiene en una carrera se traduce con `UniqueViolations` a `409`.
12. Anula las líneas de factura que quedan en cero (`annulLineCreditedToZero`) y las guarda (`saveLineStatuses`), con su `flush`, para que el paso siguiente las vea.
13. Con «anulación de unidad»: `FleetContextFacade.annulReturnedUnits` con las unidades de esas líneas. Las anula o las devuelve retenidas, sin fallar (D-191).
14. Guarda el archivo en `kept_files` bajo `credit-notes/{publicId}/document/{sha256}`, con la empresa de la nota (D-189), apunta `file_path` y registra el descarte para el caso de reversión.
15. Responde lo que guardó (convenciones §6.2).

**La anulación, paso a paso:**
1. Forma (`400`, también sin `updatedAt`).
2. La nota, que se bloquea (`findByPublicIdForUpdate`); `404` si no existe o no se lee.
3. El permiso con el alcance de registrar: toda la empresa y la marca de cada línea (`403`, D-193).
4. `updatedAt` (`422 CREDITNOTE_OUTDATED`) y anulada (`422 CREDITNOTE_ANNULLED`).
5. Bloquea sus facturas por `id` ascendente. Todas están vigentes: una factura con notas vigentes no se anula (PRC-R87).
6. Bloquea el proveedor en `FOR NO KEY UPDATE`: es el candado del saldo, el mismo con que la fase 5 calcula y consume el saldo a favor (D-195).
7. Calcula qué líneas volverían a vigentes: las anuladas cuyo valor, sin esta nota, queda mayor que cero.
8. Bloquea las unidades de todas sus líneas por la fachada (`lockUnitsByIds`). Una unidad anulada, por cualquier vía, responde `422 CREDITNOTE_UNIT_ANNULLED` (D-193, D-204); una línea del paso 7 cuya unidad ya está en otra línea vigente, `422 CREDITNOTE_LINE_RELINKED` (D-193).
9. Pregunta a `SupplierBalanceAfterAnnulment` (§2.5.3): si el saldo quedaría negativo, `422 CREDITNOTE_SUPPLIER_BALANCE_NEGATIVE`.
10. Anula la nota y la guarda con su `flush`; después devuelve a vigentes las líneas del paso 7 (`reinstateLine`, `saveLineStatuses`). En ese orden: el disparador de la línea cuenta las notas vigentes y tiene que ver la nota ya anulada (§3.3).
11. Escribe la fila delicada de la bitácora y responde la ficha.

**El cambio del archivo, paso a paso** (D-204), como el `PUT` de la factura (fase 2, su §4.2.5):
1. Forma (`400`, también sin `updatedAt` o sin archivo) y forma del archivo (`422`, sin leer la base).
2. La nota, que se bloquea; `404` si no existe o no se lee.
3. El permiso con el alcance de registrar: toda la empresa y la marca de cada línea (`403`).
4. `updatedAt` (`422 CREDITNOTE_OUTDATED`) y anulada (`422 CREDITNOTE_ANNULLED`).
5. Guarda el archivo en `kept_files` bajo una llave nueva de su contenido y apunta `file_path` (`replaceFile`). La llave anterior y sus bytes se quedan (diccionario §2.7). Registra el descarte para el caso de reversión.
6. Escribe una fila no delicada en la bitácora (§13.1): `field_name = file_path`, con la llave anterior y la nueva. Responde la ficha.

**Rendimiento.** Una nota de 100 líneas escribe una fila por línea (las llaves son `GENERATED ALWAYS AS IDENTITY`, sin lotes) y una actualización por línea que queda en cero o vuelve a vigente. Todo lo demás es constante por nota: una lectura de las líneas, un candado de las facturas, una suma de las notas vigentes por lote, una llamada a la fachada de `fleet`. `CreditNoteRegistrationStatementsTests` lo fija como en la fase 2 (§7.2).

### 2.4 Orden de los candados

Extiende la secuencia de la fase 2 (su §2.4). La nota va primero, y la factura deja de ser una sola y pasa detrás de la empresa: ninguna transacción de la fase 2 toma las dos, así que ninguna cambia su orden.

| # | Qué | Modo | Quién lo toma |
|---|---|---|---|
| 1 | La nota, en su anulación y en el cambio de su archivo | `FOR UPDATE` | Solo esas dos escrituras de la nota |
| 2 | La empresa y las marcas de las unidades nuevas | `FOR SHARE` (por `organization`) | El registro de la nota (solo la empresa) y el de la factura |
| 3 | Las facturas que una nota descuenta, por `id` ascendente | `FOR UPDATE` | El registro y la anulación de la nota; el `PUT` y la anulación de una factura (#1 de la fase 2) |
| 4 | El proveedor | `FOR SHARE` al registrar una factura u orden; `FOR NO KEY UPDATE` al anular una nota, el candado del saldo (D-195) | Su cambio de estado y sus contactos; en la fase 5, los pagos con saldo y su anulación |
| 5 | La moneda | `FOR SHARE` | El registro de la factura. La nota no la toma (PRC-R95) |
| 6 | La orden vinculada | `FOR UPDATE` | Las escrituras de la factura y de la orden |
| 7 | Las unidades, por `id` ascendente | `FOR NO KEY UPDATE`, el de `VehicleRepository.findByIdForUpdate` | Las dos fachadas nuevas de §2.5.1; el registro de una factura; toda escritura de la unidad |

**Por qué no hay ciclos.** El registro de una nota toma 2, 3 y 7; su anulación, 1, 3, 4 y 7; el cambio de su archivo, solo 1; el registro de una factura, 2, 4, 5, 6 y 7; su `PUT`, 3 y 6; su anulación, 3. Nadie que tenga una unidad espera una factura ni el proveedor, y nadie que tenga el proveedor en modo exclusivo espera algo que tenga el registro de una factura. La anulación de una unidad y la edición de su año bloquean solo la unidad y preguntan a `UnitPurchases` sin candado, así que ven el estado de las líneas que confirmó la transacción que tenía la unidad antes que ellas. **La fase 5 respeta este orden:** un pago toma las facturas a las que se aplica antes que el proveedor.

Lo que no cubre un candado lo cubre la base: dos notas con el mismo número chocan en `credit_notes_number_uq`; una línea que vuelve a vigente cuando su unidad ya está en otra, en `purchase_invoice_items_vehicle_current_uq` de la fase 2, que aquí se traduce a `422 CREDITNOTE_LINE_RELINKED`, porque bajo el candado de la unidad solo lo alcanzaría un defecto.

### 2.5 Puertos y fachadas entre contextos

Toda fachada devuelve `Optional`, tipos simples o records de su propio paquete `acl` (regla 1.4). Lo que `procurement` lee de `fleet` se traduce en `application/internal/outboundservices/acl` (`ForeignAclStaysInTranslationLayerTests`); el puerto de `fleet` que `procurement` implementa queda fuera de esa guarda, como `CurrencyUsage` ([contextos/procurement.md](contextos/procurement.md), sección 5).

#### 2.5.1 `FleetContextFacade` (dueño: `fleet`; consumidor: `procurement`)

```java
package ky.millennium.mobility.erp.fleet.interfaces.acl;

public interface FleetContextFacade {

    // … matchPurchasedUnits, registerOrLinkPurchasedUnits y fetchUnitsByIds, de la fase 2 …

    /**
     * Anula las unidades que una nota de crédito con «anulación de unidad» devolvió al proveedor: la primera vía de
     * D-104, gobernada por compras.registrar_nota_credito, que no pide stock.anular_vehiculo (D-191).
     * Propagation.MANDATORY. Bloquea las unidades de la empresa por id ascendente, con el candado de
     * VehicleRepository.findByIdForUpdate, y aplica a cada una la guarda de VehicleAnnulments: valor cero
     * (UnitPurchaseFacts, que ya ve las líneas que anuló esta transacción), sin contratos y sin cesión abierta (D-182).
     * La que pasa queda anulada; la que no, se devuelve retenida con su causa, sin escribir nada en ella.
     */
    List<ReturnedUnitOutcome> annulReturnedUnits(Long companyId, Set<Long> vehicleIds);

    /**
     * Las unidades dadas de la empresa, bloqueadas por id ascendente con el mismo candado, para la anulación de una
     * nota que devuelve sus líneas a vigentes. Propagation.MANDATORY. Una unidad de otra empresa no aparece.
     */
    Map<Long, UnitSummary> lockUnitsByIds(Long companyId, Set<Long> vehicleIds);
}

public enum ReturnedUnitHold { CONTRACTS, FUND_ASSIGNMENT_OPEN }

public record ReturnedUnitOutcome(Long vehicleId, UUID vehiclePublicId, @Nullable ReturnedUnitHold hold) {
    public boolean annulled() { return hold == null; }
}
```

- `UnitSummary` es el record de la fase 2, con `annulled`.
- `annulReturnedUnits` y la ruta explícita `POST /vehicles/{id}/annulment` usan la misma guarda, `VehicleAnnulments.annulmentHold`, así que la regla de §9.6 vive en un solo lugar. La ruta sigue exigiendo `stock.anular_vehiculo` (D-104, la segunda vía).
- Una unidad sin valor cero o ya anulada es un defecto de quien llama: la fachada lanza `IllegalStateException` y la transacción se revierte. No pasa si se cumple el orden del registro (§2.3, pasos 12 y 13).
- La unidad anulada por la nota guarda su autor en `updated_by`, la cuenta de la sesión, como la ruta explícita. No deja fila en la bitácora: es una baja (diccionario §13.1).

#### 2.5.2 `UnitPurchases` (dueño: `fleet`; lo implementa `procurement`)

```java
package ky.millennium.mobility.erp.fleet.interfaces.acl;

public interface UnitPurchases {

    // … hasCurrentLine, withCurrentLine, unitIdsWithCurrentLine y fetchDataVersion, de la fase 2 …

    /** Si el valor de la unidad es cero (§9.6): sin línea vigente, o con una línea vigente que sus notas dejaron en cero. */
    boolean valueIsZero(Long vehicleId);

    /** El costo de compra de la unidad por su línea vigente (§9.6, D-159, D-199), o vacío si no la tiene. */
    Optional<UnitPurchaseCost> purchaseCostOf(Long vehicleId);

    /** De las unidades dadas, las que una nota vigente con «anulación de unidad» devolvió al proveedor (D-191). */
    Set<Long> returnedUnitIds(Set<Long> vehicleIds);
}

public record UnitAmount(BigDecimal amount, String currency) {}

public record UnitCreditNote(UUID creditNotePublicId, String documentNumber, LocalDate issueDate, String reason,
        UnitAmount amount) {}

public record UnitPurchaseCost(UUID invoicePublicId, String documentType, String documentNumber,
        LocalDate invoiceDate, String supplierLegalName, BigDecimal exchangeRate, BigDecimal taxRate,
        UnitAmount invoicedAmount, List<UnitCreditNote> creditNotes, UnitAmount creditNotesTotal,
        UnitAmount vehicleValue, UnitAmount costWithoutTax, UnitAmount vehicleValueInFunctionalCurrency,
        UnitAmount costWithoutTaxInFunctionalCurrency) {}
```

`PurchaseInvoiceUnitPurchasesAdapter` (fase 2) responde:
- **`valueIsZero`:** sin línea vigente, o con una línea vigente cuyo valor de factura menos sus notas vigentes es cero. Lo segundo no persiste —la línea que queda en cero se anula en la misma transacción (PRC-R79)—, y la consulta lo cubre igual. Es lo que la fase 2 anunciaba: «desde la fase 4, también por sus notas».
- **`purchaseCostOf`:** la línea vigente de la unidad con su factura, sus notas vigentes ordenadas por fecha de emisión e `id`, y los montos de §2.7. Lee la moneda funcional de la empresa por `fetchPurchasingCompaniesByIds` (fase 2) y los decimales de las dos monedas por `fetchCurrencySummariesByIds` (`LocationContextFacade.java:38`).
- **`returnedUnitIds`:** las unidades con una línea anulada de una factura vigente que tiene una nota vigente con «anulación de unidad». Por D-204, esa nota siempre deja la línea en cero. `fleet` le resta las que ya están anuladas: lo que queda es la unidad «Devuelta al proveedor» (D-204).

Todo bajo el alcance de quien llama: la Row Level Security de las tablas deja ver solo las empresas que la transacción alcanza. `UnitPurchaseFacts` responde `purchaseCostOf` con el único bean que lo implementa (FLT-R53 de la fase 2) y `returnedUnitIds` con la unión. `fetchDataVersion` no cambia: lo que muestra la exportación del Inventario, «Factura vigente», depende solo de `purchase_invoice_items`, cuya versión ya sube cuando una nota anula una línea.

#### 2.5.3 La guarda del saldo (dueño: `procurement`; la implementa la fase 5)

```java
package ky.millennium.mobility.erp.procurement.application.internal.outboundservices.payments;

/**
 * Lo que los pagos dicen de anular una nota de crédito (D-193): si, sin ella, el saldo a favor del proveedor en la
 * moneda de la nota quedaría negativo (§10.4, D-190, D-195). Lo implementa la fase 5 con sus pagos. Se pregunta con
 * el proveedor bloqueado (§2.4, #4), el mismo candado con que la fase 5 calcula y consume el saldo.
 */
public interface SupplierBalanceAfterAnnulment {
    boolean leavesBalanceNegative(Long creditNoteId);
}
```

**Hasta la fase 5 ningún bean lo implementa.** `CreditNoteAnnulments` recibe `List<SupplierBalanceAfterAnnulment>` y, con la lista vacía, responde «no queda negativo»: sin pagos no hay nada aplicado de más ni saldo consumido, así que ninguna anulación puede dejarlo negativo. Es el patrón de `PurchaseInvoiceDependents` en la fase 2. `SupplierBalanceAfterAnnulmentTests.untilPaymentsExistNoAnnulmentLeavesTheBalanceNegative` fija que hoy la lista está vacía, para que el primer bean obligue a revisarla; `CreditNoteAnnulmentsTests.aGuardThatSaysNegativeRefusesTheAnnulment` prueba el camino con un bean de prueba.

#### 2.5.4 Lo que la fase 5 lee de las notas

```java
// procurement.domain.repositories.CreditNoteRepository
/** Lo que las notas vigentes restan de cada factura dada, en su moneda; una factura sin notas no aparece (D-190). */
Map<Long, CreditedTotal> currentCreditsByInvoiceIds(Set<Long> purchaseInvoiceIds);

/** Lo mismo por línea de factura: lo que se resta del valor de cada unidad (§9.6, D-199). */
Map<Long, CreditedTotal> currentCreditsByInvoiceItemIds(Set<Long> purchaseInvoiceItemIds);
```

Los dos son una consulta agrupada sobre `credit_note_items` de notas no anuladas, por el índice de §3.3. Esta fase ya los usa: el registro, para el valor de cada línea; la ficha de la factura; y el costo de una unidad. La fase 5 los usa para lo que se debe de una factura (su total, menos sus notas vigentes, menos lo aplicado, §10.4) y para repartir lo cubierto en proporción al valor de cada unidad (D-199). La nota no lleva saldo propio: el saldo es del proveedor (D-190).

#### 2.5.5 `PurchaseInvoiceDependents` (fase 2, su §2.5.4)

`CreditNoteInvoiceDependentsAdapter` (`procurement/infrastructure/dependents`) responde `anyFor(purchaseInvoiceId)` con una consulta: alguna línea de la factura tiene una línea de una nota no anulada. Una nota anulada ya no cuenta, así que anular la nota libera la factura. La anulación de la factura lee los dependientes bajo su candado (fase 2), y la nota bloquea sus facturas antes de escribir (§2.4), así que una nota y la anulación de su factura no se cruzan.

La prueba de la fase 2 `PurchaseInvoiceDependentsTests.untilCreditNotesAndPaymentsExistNoInvoiceHasDependents` pasa a `untilPaymentsExistOnlyCreditNotesAreDependents`: exactamente un bean, el de las notas.

#### 2.5.6 Lo que se reutiliza

- **`organization`**: `fetchPurchasingCompanyForShare` y `fetchPurchasingCompaniesByIds`, de la fase 2 (empresa activa, país, zona y moneda funcional).
- **`location`**: `fetchCurrencySummaryByIsoCode` y `fetchCurrencySummariesByIds`, que ya existen (`LocationContextFacade.java:38` y `:47`), con `decimalPlaces` de `CurrencySummary`.
- **`fleet`**: `fetchUnitsByIds`, de la fase 2, para la ficha de la nota (VIN, placa, modelo y si la unidad está anulada).
- **`documents`**: `KeptFileStore.put` (`documents/interfaces/acl/KeptFileStore.java:26`), `open` (`:33`) y `discardRolledBack`, de la fase 2.
- **`shared`**: `ChangeLogCommandService.recordChange` (`shared/changelog/application/commandservices/ChangeLogCommandService.java`), y `AuditActorLookupPort` para nombrar `createdBy` y `annulledBy`.

### 2.6 Lo que una nota hace a la línea, a la unidad y a la orden

| Motivo | La línea no llega a cero | La línea llega a cero |
|---|---|---|
| Descuento de campaña, ajuste de precio | Baja el valor de la unidad | La línea se anula y la unidad queda sin factura vigente (D-191) |
| Anulación de la factura | No se admite (D-204) | La línea se anula y la unidad queda sin factura vigente; la factura que la reemplaza la vincula por su VIN (D-191, D-160) |
| Anulación de unidad | No se admite (D-204) | La línea se anula y `fleet` anula la unidad; con contratos o una cesión abierta, la unidad queda devuelta y sin anular, marcada «Devuelta al proveedor» para revisar (D-191, D-182, D-204) |

Además, en los tres casos con línea en cero:
- La unidad aparece en el filtro «Sin factura vigente» del Inventario (D-175, fase 2).
- Lo pendiente de su orden vuelve a contarla, porque cuenta las líneas vigentes (D-162, fase 2).
- La factura no cambia su `updated_at`: el estado de una línea no es uno de los cambios que lo mueven (§9.3; D-178).

**La anulación de una nota** devuelve a vigentes las líneas que, sin ella, recuperan valor, con los impedimentos de D-193 y D-204: una unidad de sus líneas anulada por cualquier vía, y la unidad de una línea que vuelve ya vinculada a otra factura vigente. Una línea que sigue en cero porque otras notas vigentes la cubren entera no vuelve.

### 2.7 Valor y costo de una unidad

Ninguno se guarda (§9.6, §2.2). Los calcula `UnitPurchaseCostCalculator` con la línea vigente, su factura y sus notas vigentes:

| Monto | Cómo sale | Moneda |
|---|---|---|
| Valor de factura | `invoiced_amount` de la línea, fijo (§9.4) | La de la factura |
| Notas de crédito | La suma de las líneas vigentes de notas sobre esa línea | La de la factura (§9.5) |
| Valor del vehículo | Valor de factura menos notas, exacto (§9.6, guía columna 41) | La de la factura |
| Costo sin IGV (calculado) | En una factura: valor del vehículo ÷ (1 + `tax_rate`), redondeado mitad hacia arriba a los decimales de su moneda. En una boleta u otro: el valor del vehículo (D-159) | La de la factura |
| Valor en la moneda de la empresa (calculado) | Valor del vehículo × `exchange_rate` de la factura, redondeado mitad hacia arriba a los decimales de la moneda funcional (guía, columna 48) | La funcional |
| Costo sin IGV en la moneda de la empresa (calculado) | En una factura: valor del vehículo × tasa ÷ (1 + `tax_rate`), con `MathContext.DECIMAL128` y redondeado una sola vez al final (guía, columna 49: «conservar precisión interna suficiente»). En una boleta u otro: el valor en la moneda de la empresa | La funcional |

**La tasa es la de la factura** (D-159, D-192, D-199): cada nota copió la misma en su línea, así que el valor del vehículo en la moneda de la empresa es su valor por esa tasa. En la moneda funcional la tasa es 1 (D-169). La guía divide entre 1,18; el sistema divide entre 1 + la tasa guardada en la factura (D-24).

**Ejemplo** — un auto facturado en dólares que recibe una nota:

| Dato | Valor |
|---|---|
| Factura F002-981, USD, tasa 3,742000, IGV 0,1800 | Valor de factura 13 100,00 |
| Nota FC02-17, descuento de campaña | 500,00 |
| Valor del vehículo | 12 600,00 USD |
| Costo sin IGV | 12 600,00 ÷ 1,18 = 10 677,97 USD |
| Valor en soles | 12 600,00 × 3,742 = 47 149,20 PEN |
| Costo sin IGV en soles | 47 149,20 ÷ 1,18 = 39 956,95 PEN |

---

## 3. Esquema

### 3.1 Numeración

Los números se asignan al integrar en `develop`, después de V91 y de lo que las fases 2 y 3 integren antes (plan §4). Aquí van relativos, en este orden dentro de la fase. Cada migración empieza con `SELECT set_config('app.access_scope', 'SYSTEM', true);` y `SET LOCAL lock_timeout = '5s';`, como V90 (`V90__seed_iam_permissions_of_exchange_rates.sql`).

| Migración | Contexto | Paso | Contenido |
|---|---|---|---|
| V(n) `seed_iam_permissions_of_credit_notes` | `iam` | 6.1 | El permiso de las notas y sus tres marcas |
| V(n+1) `create_procurement_credit_notes_schema` | `procurement` | 6.2 | Las dos tablas, sus restricciones, disparadores, Row Level Security, permisos de la cuenta y `data_versions`; la llave `(id, company_id)` de las líneas de factura y la función que deja volver una línea a vigente |
| V(n+2) `mark_invoice_permissions_on_the_roles_of_the_matrix` | `iam` | 6.12 | Las siete marcas de facturas en AE, OP y CO (D-176) |

### 3.2 V(n): el permiso de las notas

```sql
INSERT INTO permissions (code, module, description, category)
VALUES ('compras.registrar_nota_credito', 'compras',
        'Registrar y anular una nota de crédito y su reparto; la línea que queda en cero se anula y, con «anulación de unidad», también la unidad',
        'crear')
ON CONFLICT (code) DO NOTHING;

INSERT INTO role_permissions (role_id, permission_id, granted_by)
SELECT role.id, permission.id, system_account.id
FROM (VALUES ('SUPERADMIN', 'compras.registrar_nota_credito'),
             ('ADMIN_EMPRESA', 'compras.registrar_nota_credito'),
             ('CONTABILIDAD', 'compras.registrar_nota_credito')) AS matrix (role_code, permission_code)
JOIN roles role ON role.code = matrix.role_code AND role.is_system AND role.company_id IS NULL
JOIN permissions permission ON permission.code = matrix.permission_code
CROSS JOIN (SELECT id FROM users WHERE email = 'system@millennium.ky') AS system_account
ON CONFLICT DO NOTHING;
```

- Las tres marcas son las de la matriz (§5.6): SA, AE y CO. El permiso no es de facturas, así que sigue la matriz desde ya, como `stock.anular_vehiculo` en la fase 2.
- La descripción es la de §5.6, ya corregida después de D-191 y D-193, sin la referencia entre paréntesis, como V79 y V90.

### 3.3 V(n+1): las tablas de las notas

```sql
-- La llave a la que apunta cada línea de una nota, con su empresa
ALTER TABLE purchase_invoice_items
    ADD CONSTRAINT purchase_invoice_items_id_company_uq UNIQUE (id, company_id);

CREATE TABLE credit_notes
(
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id         UUID           NOT NULL DEFAULT uuidv7(),
    company_id        BIGINT         NOT NULL REFERENCES companies (id),
    supplier_id       BIGINT         NOT NULL,
    document_number   VARCHAR(30)    NOT NULL,
    issue_date        DATE           NOT NULL,
    currency_id       BIGINT         NOT NULL REFERENCES currencies (id),
    total_amount      NUMERIC(14, 2) NOT NULL,
    reason            VARCHAR(30)    NOT NULL,
    campaign_month    DATE,
    file_path         TEXT,
    annulled_at       TIMESTAMPTZ,
    annulled_by       BIGINT         REFERENCES users (id),
    annulment_reason  VARCHAR(30),
    created_at        TIMESTAMPTZ    NOT NULL DEFAULT now(),
    created_by        BIGINT         REFERENCES users (id),
    updated_at        TIMESTAMPTZ    NOT NULL DEFAULT now(),   -- D-193
    updated_by        BIGINT         REFERENCES users (id),    -- D-193

    CONSTRAINT credit_notes_public_id_uq UNIQUE (public_id),
    CONSTRAINT credit_notes_id_company_uq UNIQUE (id, company_id),
    CONSTRAINT credit_notes_supplier_company_fk
        FOREIGN KEY (supplier_id, company_id) REFERENCES suppliers (id, company_id),
    CONSTRAINT credit_notes_document_number_ck
        CHECK (length(document_number) > 0 AND document_number = upper(document_number) AND document_number !~ '\s'),
    CONSTRAINT credit_notes_total_amount_ck CHECK (total_amount > 0),          -- D-23: no hay notas de débito
    CONSTRAINT credit_notes_reason_ck
        CHECK (reason IN ('DESCUENTO_CAMPANA', 'ANULACION_UNIDAD', 'ANULACION_FACTURA', 'AJUSTE_PRECIO')),
    CONSTRAINT credit_notes_campaign_month_ck                                   -- día 1 (§2.6); el motivo, D-204
        CHECK (campaign_month IS NULL OR (extract(DAY FROM campaign_month) = 1 AND reason = 'DESCUENTO_CAMPANA')),
    CONSTRAINT credit_notes_annulment_reason_ck
        CHECK (annulment_reason IS NULL OR annulment_reason IN ('ERROR_REGISTRO', 'ANULADA_POR_EMISOR')),
    CONSTRAINT credit_notes_annulment_ck
        CHECK ((annulled_at IS NULL AND annulled_by IS NULL AND annulment_reason IS NULL)
            OR (annulled_at IS NOT NULL AND annulled_by IS NOT NULL AND annulment_reason IS NOT NULL))
);

-- El número no se repite en el proveedor entre las no anuladas (D-193)
CREATE UNIQUE INDEX credit_notes_number_uq
    ON credit_notes (company_id, supplier_id, document_number) WHERE annulled_at IS NULL;
CREATE INDEX idx_credit_notes_company_date ON credit_notes (company_id, issue_date DESC, id DESC);
CREATE INDEX idx_credit_notes_supplier ON credit_notes (company_id, supplier_id);
CREATE INDEX idx_credit_notes_currency ON credit_notes (currency_id);

CREATE TABLE credit_note_items
(
    id                        BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id                 UUID           NOT NULL DEFAULT uuidv7(),
    credit_note_id            BIGINT         NOT NULL,
    company_id                BIGINT         NOT NULL REFERENCES companies (id),
    purchase_invoice_item_id  BIGINT         NOT NULL,
    exchange_rate             NUMERIC(12, 6) NOT NULL,
    amount                    NUMERIC(14, 2) NOT NULL,

    CONSTRAINT credit_note_items_public_id_uq UNIQUE (public_id),
    CONSTRAINT credit_note_items_note_company_fk
        FOREIGN KEY (credit_note_id, company_id) REFERENCES credit_notes (id, company_id),
    CONSTRAINT credit_note_items_invoice_item_company_fk
        FOREIGN KEY (purchase_invoice_item_id, company_id) REFERENCES purchase_invoice_items (id, company_id),
    CONSTRAINT credit_note_items_exchange_rate_ck CHECK (exchange_rate > 0),
    CONSTRAINT credit_note_items_amount_ck CHECK (amount > 0)
);

-- Una línea de factura, una vez por nota; y lo que restan las notas de cada línea, por el índice
CREATE UNIQUE INDEX credit_note_items_note_line_uq ON credit_note_items (credit_note_id, purchase_invoice_item_id);
CREATE INDEX idx_credit_note_items_invoice_item ON credit_note_items (purchase_invoice_item_id);
CREATE INDEX idx_credit_note_items_note ON credit_note_items (credit_note_id, id);
```

**Las dos llaves compuestas** obligan a que la línea de la nota, la nota y la línea de factura sean de la misma empresa (§2.5). **Lo que la base no decide y queda en el servicio:** que la empresa esté activa, el formato del número en Perú, la fecha futura (depende del momento y de la zona), el corte de una factura (depende de la configuración), las 100 líneas y el permiso.

**La línea de una nota descuenta lo que su factura permite** (D-192, D-193; §2.2: la regla de moneda «debería ser imposible violarla»):

```sql
CREATE FUNCTION enforce_credit_note_item_invoice() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
DECLARE
    note RECORD;
    line RECORD;
BEGIN
    SELECT supplier_id, currency_id, issue_date INTO note FROM credit_notes WHERE id = NEW.credit_note_id;
    IF NOT FOUND THEN
        RAISE EXCEPTION 'credit note % not found for its line', NEW.credit_note_id;
    END IF;
    SELECT item.status, invoice.supplier_id, invoice.currency_id, invoice.exchange_rate, invoice.invoice_date,
           invoice.annulled_at
    INTO line
    FROM purchase_invoice_items item
    JOIN purchase_invoices invoice ON invoice.id = item.purchase_invoice_id
    WHERE item.id = NEW.purchase_invoice_item_id;
    IF NOT FOUND THEN
        RAISE EXCEPTION 'purchase invoice line % not found for a credit note', NEW.purchase_invoice_item_id;
    END IF;
    IF line.annulled_at IS NOT NULL OR line.status <> 'VIGENTE' THEN
        RAISE EXCEPTION 'A credit note discounts only a current line of a current invoice';
    END IF;
    IF (line.supplier_id, line.currency_id) IS DISTINCT FROM (note.supplier_id, note.currency_id) THEN
        RAISE EXCEPTION 'A credit note discounts only invoices of its supplier and its currency';
    END IF;
    IF NEW.exchange_rate <> line.exchange_rate THEN
        RAISE EXCEPTION 'A line of a credit note copies the exchange rate of its invoice';
    END IF;
    IF note.issue_date < line.invoice_date THEN
        RAISE EXCEPTION 'A credit note is not dated before an invoice it discounts';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER credit_note_items_invoice_tg
    BEFORE INSERT ON credit_note_items FOR EACH ROW EXECUTE FUNCTION enforce_credit_note_item_invoice();

CREATE FUNCTION enforce_credited_line_value() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
DECLARE
    invoiced NUMERIC(14, 2);
    credited NUMERIC(14, 2);
BEGIN
    SELECT invoiced_amount INTO invoiced FROM purchase_invoice_items WHERE id = NEW.purchase_invoice_item_id;
    SELECT coalesce(sum(item.amount), 0) INTO credited
    FROM credit_note_items item
    JOIN credit_notes note ON note.id = item.credit_note_id
    WHERE item.purchase_invoice_item_id = NEW.purchase_invoice_item_id AND note.annulled_at IS NULL;
    IF credited > invoiced THEN
        RAISE EXCEPTION 'The credit notes of a line never take its value below zero';
    END IF;
    RETURN NULL;
END
$$;

CREATE CONSTRAINT TRIGGER credit_note_items_line_value_tg
    AFTER INSERT ON credit_note_items
    DEFERRABLE INITIALLY DEFERRED
    FOR EACH ROW EXECUTE FUNCTION enforce_credited_line_value();
```

El segundo es una guarda contra un defecto, no contra una carrera: dos notas sobre la misma línea las ordena el candado de su factura (§2.4), y sin él el disparador diferido de una no vería las filas sin confirmar de la otra.

**Lo registrado no se edita, también en la base** (D-193):

```sql
CREATE FUNCTION keep_credit_note_data() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    IF OLD.annulled_at IS NOT NULL THEN
        RAISE EXCEPTION 'An annulled credit note never changes';
    END IF;
    IF (NEW.company_id, NEW.supplier_id, NEW.document_number, NEW.issue_date, NEW.currency_id, NEW.total_amount,
        NEW.reason, NEW.campaign_month, NEW.created_at, NEW.created_by)
       IS DISTINCT FROM
       (OLD.company_id, OLD.supplier_id, OLD.document_number, OLD.issue_date, OLD.currency_id, OLD.total_amount,
        OLD.reason, OLD.campaign_month, OLD.created_at, OLD.created_by) THEN
        RAISE EXCEPTION 'The data of a credit note are not edited: annul it and register another';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER credit_notes_data_tg
    BEFORE UPDATE ON credit_notes FOR EACH ROW EXECUTE FUNCTION keep_credit_note_data();

CREATE FUNCTION keep_credit_note_items() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    RAISE EXCEPTION 'A line of a credit note is never edited';
END
$$;

CREATE TRIGGER credit_note_items_data_tg
    BEFORE UPDATE ON credit_note_items FOR EACH ROW EXECUTE FUNCTION keep_credit_note_items();
```

En la cabecera solo cambian `file_path` —que el registro apunta después de guardar la nota, como la factura (fase 2, su §2.3, paso 14), y que reemplaza el `PUT` de §4.2.6 (D-204)—, los datos de la anulación y las columnas de revisión.

**Una línea de factura vuelve a vigente solo si recupera valor** (D-193). Reemplaza la función de la fase 2 (su §3.4), que no dejaba cambiar nunca una línea anulada:

```sql
CREATE OR REPLACE FUNCTION keep_purchase_invoice_item_fiscal_data() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
DECLARE
    invoice_annulled BOOLEAN;
    credited NUMERIC(14, 2);
BEGIN
    IF (NEW.purchase_invoice_id, NEW.company_id, NEW.vehicle_id, NEW.concept, NEW.model_id, NEW.model_year,
        NEW.commercial_brand_id, NEW.invoiced_amount, NEW.isc_amount)
       IS DISTINCT FROM
       (OLD.purchase_invoice_id, OLD.company_id, OLD.vehicle_id, OLD.concept, OLD.model_id, OLD.model_year,
        OLD.commercial_brand_id, OLD.invoiced_amount, OLD.isc_amount) THEN
        RAISE EXCEPTION 'A line of a purchase invoice is never edited';
    END IF;
    IF OLD.status = 'ANULADA' AND NEW.status = 'VIGENTE' THEN
        SELECT annulled_at IS NOT NULL INTO invoice_annulled FROM purchase_invoices WHERE id = NEW.purchase_invoice_id;
        IF invoice_annulled THEN
            RAISE EXCEPTION 'An annulled line of an annulled purchase invoice never changes';
        END IF;
        SELECT coalesce(sum(item.amount), 0) INTO credited
        FROM credit_note_items item
        JOIN credit_notes note ON note.id = item.credit_note_id
        WHERE item.purchase_invoice_item_id = NEW.id AND note.annulled_at IS NULL;
        IF credited >= NEW.invoiced_amount THEN
            RAISE EXCEPTION 'A line comes back to current only when its credit notes leave it some value';
        END IF;
    ELSIF OLD.status = 'ANULADA' AND (NEW.status, NEW.annulled_at) IS DISTINCT FROM (OLD.status, OLD.annulled_at) THEN
        RAISE EXCEPTION 'An annulled line of a purchase invoice never changes';
    END IF;
    RETURN NEW;
END
$$;
```

`purchase_invoice_items_annulment_ck` de la fase 2 ya exige que una línea vigente no tenga `annulled_at`. El disparador `purchase_invoice_items_fiscal_tg` no cambia: sigue llamando a esta función.

**Row Level Security, como V83** (`V83__isolate_procurement_with_the_policies_of_v29.sql`):

```sql
ALTER TABLE credit_notes ENABLE ROW LEVEL SECURITY;
ALTER TABLE credit_notes FORCE ROW LEVEL SECURITY;
CREATE POLICY credit_notes_select ON credit_notes FOR SELECT USING (access_scope_reaches(company_id));
CREATE POLICY credit_notes_insert ON credit_notes FOR INSERT WITH CHECK (access_scope_reaches(company_id));
CREATE POLICY credit_notes_update ON credit_notes FOR UPDATE
    USING (access_scope_reaches(company_id)) WITH CHECK (access_scope_reaches(company_id));
-- Las mismas tres en credit_note_items: credit_note_items_select, _insert y _update
```

Ninguna política de borrado. **Permisos de la cuenta de la aplicación**, con el bloque `DO` de V69 sobre `${app_role}` (`V69__create_purchase_orders.sql:161`): `REVOKE ALL` y `GRANT SELECT, INSERT, UPDATE` en las dos tablas, sin `DELETE`. Las líneas reciben `UPDATE` como toda tabla de empresa, y su disparador lo rechaza.

**`data_versions`**: las dos tablas muestran la exportación, así que siembran sus 64 casillas (`generate_series(0, 63)`) y crean sus dos disparadores por sentencia, `credit_notes_data_version_insert_tg`, `…_update_tg`, `credit_note_items_data_version_insert_tg` y `…_update_tg`, como V69.

### 3.4 V(n+2): los permisos de facturas en la matriz (D-176)

```sql
INSERT INTO role_permissions (role_id, permission_id, granted_by)
SELECT role.id, permission.id, system_account.id
FROM (VALUES ('ADMIN_EMPRESA', 'compras.registrar_factura'),
             ('ADMIN_EMPRESA', 'compras.registrar_factura_antigua'),
             ('ADMIN_EMPRESA', 'compras.anular_factura'),
             ('OPERACIONES', 'compras.registrar_factura'),
             ('CONTABILIDAD', 'compras.registrar_factura'),
             ('CONTABILIDAD', 'compras.registrar_factura_antigua'),
             ('CONTABILIDAD', 'compras.anular_factura')) AS matrix (role_code, permission_code)
JOIN roles role ON role.code = matrix.role_code AND role.is_system AND role.company_id IS NULL
JOIN permissions permission ON permission.code = matrix.permission_code
CROSS JOIN (SELECT id FROM users WHERE email = 'system@millennium.ky') AS system_account
ON CONFLICT DO NOTHING;
```

- Son las siete marcas de la matriz (§5.6) que la fase 2 dejó para después (su §3.2). La matriz final no cambia (D-176).
- Es la migración con la que las facturas pasan a uso real, así que va en el último commit del backend, que se integra por orden del responsable. El contador solo valida lo decidido, en paralelo (plan §3.2): nada espera su respuesta.
- `PurchaseInvoicePermissionsSeedMigrationTests.onlyTheSuperadministratorHoldsTheInvoicePermissions`, de la fase 2, se reemplaza en el mismo commit por `InvoicePermissionsMatrixMigrationTests.theInvoicePermissionsFollowTheMatrixOnceCreditNotesExist`.

---

## 4. Contrato de la API

### 4.1 Rutas

| # | Método y ruta | Permiso de la ruta | Respuesta | Paso |
|---|---|---|---|---|
| 1 | `GET /credit-notes` | `compras.ver` | `200` página o arreglo | 6.7 |
| 2 | `GET /credit-notes` con el `Accept` de Excel | `compras.ver` | `303` a `/documents/{id}` | 6.7 |
| 3 | `GET /credit-notes/{id}` | `compras.ver` | `200` ficha | 6.7 |
| 4 | `POST /credit-notes` (multipart) | `compras.registrar_nota_credito` | `201` con `Location` | 6.5 |
| 5 | `POST /credit-notes/{id}/annulment` | `compras.registrar_nota_credito` | `200` ficha | 6.6 |
| 6 | `PUT /credit-notes/{id}` (multipart) | `compras.registrar_nota_credito` | `200` ficha | 6.6 |
| 7 | `GET /credit-notes/{id}/file` | `compras.ver` | `200` bytes | 6.7 |
| 8 | `GET /credit-note-rules` **(propuesta)** | `compras.ver` | `200` el corte | 6.7 |
| 9 | `GET /vehicles/{id}/purchase-cost` (D-203) | `stock.ver_costos` | `200` costo | 6.8 |

**Rutas que ya existen, o que crea la fase 2, y cambian:** `GET /purchase-invoices/{id}` (lo que restan las notas, su panel y `registerCreditNote`, §4.2.9); `POST /purchase-invoices/{id}/annulment` (su `422 PURCHASEINVOICE_HAS_DEPENDENTS` deja de ser inalcanzable); `GET /vehicles` y `GET /vehicles/{id}` (`returnedToSupplier` y `allowedActions.viewPurchaseCost`, §4.2.10).

`CreditNotesController` lleva `@PermissionRequired(PermissionCodes.Procurement.VIEW)` en la clase y el suyo en cada escritura. El servicio exige además leer con `compras.ver` donde escribe (D-134): escribir sobre una nota que no se lee responde `404`. No hay `DELETE` (`NoDeleteEndpointsTests`).

### 4.2 Cada ruta

Los montos viajan como `{ "amount", "currency" }` (convenciones §5.2): el pedido acepta `amount` como texto decimal o número, y la respuesta lo escribe con los decimales de la moneda, como la factura. `exchangeRate` y `taxRate` siguen la misma regla. Las fechas, `AAAA-MM-DD`; un mes, `AAAA-MM-DD` con día 1 (convenciones §5.1); los momentos, ISO en UTC.

#### 4.2.1 `GET /credit-notes`

| `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|
| El número normalizado, la razón social y el documento tributario del proveedor, sin distinguir mayúsculas ni tildes (`fold_search_text`, D-142) | `companyId`; `supplierId`; `reason` (los cuatro de `CreditNoteReason`, en cualquier mayúscula); `status` (`VIGENTE` o `ANULADA`, derivado de `annulled_at`); `currency` (código ISO); `from` y `to`, el período `[from, to)` sobre `issueDate`. Un valor fuera de lista o una fecha mal escrita, `400 VALIDATION_ERROR`; un `companyId` fuera del alcance, `404 COMPANY_NOT_FOUND` | `issueDate`, `documentNumber`, `supplierLegalName`, `totalAmount`, `createdAt` | `issueDate` desc; los empates, `id` desc |

Lista solo lo que quien llama lee con `compras.ver` (`companyReach`). Cada fila: `{id, companyId, companyName, supplierId, supplierLegalName, supplierTaxId, documentNumber, issueDate, currency, totalAmount, reason, campaignMonth, invoiceNumbers, unitCount, status, annulledAt, createdAt, allowedActions: null}`. `invoiceNumbers` son los números de sus facturas, sin repetir, en el orden de sus líneas. Las referencias se resuelven en lote.

#### 4.2.2 Exportación

El mismo `GET` con `Accept: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/json;q=0.9`, con los mismos `q`, filtros y `sort`, sin página (convenciones §8.1). Tipo de documento `credit-notes`, nombre base `notas-de-credito`, alcance `DocumentScopes.grantsOf(scope, compras.ver)`, versión `credit_notes` + `credit_note_items` + `purchase_invoices` + `suppliers` + la de `organization` + la de `location`. Columnas, en este orden (claves `export.credit-notes.*`):

| # | Encabezado | Valor |
|---|---|---|
| 1 | Empresa | Nombre comercial |
| 2 | Documento del proveedor | Su número, con la sigla de su tipo: `RUC 20100128056` |
| 3 | Proveedor | Razón social |
| 4 | Número | Normalizado |
| 5 | Fecha de emisión | Celda de fecha |
| 6 | Motivo | Traducido: Descuento de campaña, Anulación de unidad, Anulación de la factura, Ajuste de precio |
| 7 | Mes de campaña | Celda de fecha con el día 1, como se guarda (§2.6), o vacía |
| 8 | Moneda | Código ISO |
| 9 | Total | Número, decimales de la moneda |
| 10 | Moneda de la empresa | Código ISO de la moneda funcional |
| 11 | Total en la moneda de la empresa (calculado) | `totalInFunctionalCurrency` (§2.2): cada línea por la tasa de su factura |
| 12 | Facturas | Sus números, separados por coma |
| 13 | Unidades | Número de líneas |
| 14 | Estado | Traducido: Vigente, Anulada |
| 15 | Motivo de anulación | Traducido: Error de registro, Anulada por el emisor |
| 16 | Anulada (zona) | Momento, con la zona en el encabezado |
| 17 | Registrada (zona) | Momento, con la zona en el encabezado |

La columna 11 lleva «(calculado)», como las de la factura (D-180), porque suma montos redondeados por línea. La zona de los momentos sigue la regla de la fase 2 (su §4.2.2). Nunca sale el `id` ni el `public_id`.

#### 4.2.3 `GET /credit-notes/{id}`

`404 CREDITNOTE_NOT_FOUND` si no existe o quien llama no la lee. Responde:

```json
{
  "id": "019a…", "companyId": "0199…", "companyName": "Millennium Perú",
  "supplierId": "0199…", "supplierLegalName": "Crosland Automotriz S.A.C.", "supplierTaxId": "20100128056",
  "documentNumber": "FC01-45", "issueDate": "2026-09-15", "currency": "PEN",
  "totalAmount": { "amount": 10000.00, "currency": "PEN" },
  "functionalCurrency": "PEN",
  "totalInFunctionalCurrency": { "amount": 10000.00, "currency": "PEN" },
  "reason": "DESCUENTO_CAMPANA", "campaignMonth": "2026-09-01",
  "hasFile": true,
  "status": "VIGENTE", "annulledAt": null, "annulledBy": null, "annulmentReason": null,
  "createdAt": "2026-11-03T15:04:00Z", "createdBy": { "id": "0199…", "name": "Tere Ramos" },
  "updatedAt": "2026-11-03T15:04:00Z",
  "invoices": [
    { "id": "019a…", "documentNumber": "F001-123", "invoiceDate": "2026-03-15", "exchangeRate": 1.000000,
      "amount": { "amount": 10000.00, "currency": "PEN" } }
  ],
  "items": [
    { "id": "019a…", "purchaseInvoiceId": "019a…", "purchaseInvoiceItemId": "019a…", "invoiceDocumentNumber": "F001-123",
      "vehicleId": "019a…", "vin": "MD2A11CY6LWA12345", "plate": null, "modelName": "DISCOVER 125-ST",
      "brandName": "BAJAJ", "modelYear": 2026, "commercialBrandName": "Lima Bikes",
      "invoicedAmount": { "amount": 5900.00, "currency": "PEN" },
      "amount": { "amount": 2500.00, "currency": "PEN" }, "exchangeRate": 1.000000,
      "amountInFunctionalCurrency": { "amount": 2500.00, "currency": "PEN" },
      "lineValue": { "amount": 3400.00, "currency": "PEN" }, "lineStatus": "VIGENTE",
      "unitAnnulled": false, "returnedToSupplier": false }
  ],
  "allowedActions": ["annul", "downloadFile"]
}
```

- Las líneas llegan en el orden en que se registraron, por su `id` (como PRC-R30).
- `invoices` resume, por factura, lo que la nota le resta.
- `lineValue` y `lineStatus` son los de hoy de la línea de factura, con todas sus notas vigentes, no solo esta.
- El VIN, la placa y si la unidad está anulada son los de hoy de la unidad (`fetchUnitsByIds`).
- `returnedToSupplier` es verdadero en la línea de una nota vigente con «anulación de unidad» cuya unidad no se anuló (D-204).
- `createdBy` y `annulledBy` se nombran por `AuditActorLookupPort`, como en la factura.

#### 4.2.4 `POST /credit-notes`

`multipart/form-data` con dos partes: `creditNote` (JSON, `application/json`, obligatoria) y `file` (opcional). El tipo que declara la parte no se cree: se verifica el contenido (D-174).

```json
{
  "companyId": "0199…",
  "supplierId": "0199…",
  "documentNumber": "FC01-00000045",
  "issueDate": "2026-09-15",
  "currency": "PEN",
  "totalAmount": { "amount": "10000.00", "currency": "PEN" },
  "reason": "DESCUENTO_CAMPANA",
  "campaignMonth": "2026-09-01",
  "items": [
    { "purchaseInvoiceItemId": "019a…", "amount": { "amount": "2500.00", "currency": "PEN" } },
    { "purchaseInvoiceItemId": "019a…", "amount": { "amount": "2500.00", "currency": "PEN" } },
    { "purchaseInvoiceItemId": "019a…", "amount": { "amount": "2500.00", "currency": "PEN" } },
    { "purchaseInvoiceItemId": "019a…", "amount": { "amount": "2500.00", "currency": "PEN" } }
  ]
}
```

| Campo | Tipo | Obligatorio | Restricción (`400 VALIDATION_ERROR` si no se cumple) |
|---|---|---|---|
| `companyId`, `supplierId` | UUID | Sí | — |
| `documentNumber` | texto | Sí | De 1 a 30 caracteres después de normalizar; uno hecho solo de separadores queda vacío y responde `400` |
| `issueDate` | fecha | Sí | ISO |
| `currency` | texto | Sí | Tres letras |
| `totalAmount` | monto | Sí | Mayor que 0, hasta `999999999999.99`, en `currency` |
| `reason` | texto | Sí | Uno de los cuatro de `CreditNoteReason`, en cualquier mayúscula |
| `campaignMonth` | fecha | No | Con día 1, y solo con `DESCUENTO_CAMPANA` (D-204) |
| `items` | arreglo | Sí | De 1 a 100 (D-204); un elemento `null` responde en `items[n]` |
| `items[n].purchaseInvoiceItemId` | UUID | Sí | La línea de factura que descuenta |
| `items[n].amount` | monto | Sí | Mayor que 0 (D-23), hasta el tope, en `currency` |

El pedido no lleva tasa: cada línea copia la de su factura (D-192). No lleva tipo de nota: solo hay notas de crédito (D-23).

**Revisiones, en este orden.** Cada una responde el primer fallo, con `context.lines` cuando nombra líneas (§4.3):
1. Forma: `400 VALIDATION_ERROR`. Archivo: `422 CREDITNOTE_FILE_TYPE` o `CREDITNOTE_FILE_SIZE`; más de 10 MB por parte o 21 MB por pedido corta antes el servidor (`413 CONTENT_TOO_LARGE`).
2. Sin `compras.registrar_nota_credito` en ninguna empresa: `403 ACCESS_DENIED`.
3. Empresa: `404 COMPANY_NOT_FOUND` si no existe, no se alcanza o no se lee con `compras.ver`; `403 ACCESS_DENIED` si el permiso no actúa en toda la empresa; `422 COMPANY_INACTIVE` (D-84).
4. Proveedor: `404 SUPPLIER_NOT_FOUND` si no es de la empresa. Uno inactivo se acepta (D-204).
5. Moneda: `404 CURRENCY_NOT_FOUND`. Una inactiva se acepta (D-204).
6. Montos: `422 CREDITNOTE_AMOUNT_SCALE`; `422 CREDITNOTE_TOTAL_MISMATCH`.
7. Número: `422 CREDITNOTE_NUMBER_FORMAT` (solo en Perú, D-204); `409 CREDITNOTE_NUMBER_CONFLICT`.
8. Fecha: `422 CREDITNOTE_DATE_IN_FUTURE`.
9. `422 CREDITNOTE_DUPLICATE_LINE`.
10. Líneas de factura: `404 PURCHASEINVOICE_NOT_FOUND`, con `context.lines`, si una no existe o su factura no se lee con `compras.ver`.
11. Bajo el candado de las facturas, por línea: `422 CREDITNOTE_INVOICE_MISMATCH` (otra empresa, otro proveedor u otra moneda, con `context.lines[].fields`); `422 CREDITNOTE_LINE_ANNULLED` (la línea o su factura); `403 ACCESS_DENIED` (una marca donde el permiso no actúa, D-193).
12. `CreditNoteRegistrationPolicy`: `422 CREDITNOTE_DATE_BEFORE_INVOICE`; `422 CREDITNOTE_SINGLE_INVOICE_REQUIRED`; `422 CREDITNOTE_EXCEEDS_VALUE`; `422 CREDITNOTE_UNIT_RETURN_PARTIAL`; `422 CREDITNOTE_INVOICE_ANNULMENT_PARTIAL` (los dos últimos, D-204).
13. Guardado: `409 CREDITNOTE_NUMBER_CONFLICT` en una carrera que la base detiene, sin dejar nada escrito.

Responde `201` con `Location: /api/v1/credit-notes/{id}` y la ficha (§4.2.3). Una unidad devuelta y retenida no es un error: la ficha la marca con `returnedToSupplier: true` (D-191).

#### 4.2.5 `POST /credit-notes/{id}/annulment`

```json
{ "reason": "ERROR_REGISTRO", "updatedAt": "2026-11-03T15:04:00.123Z" }
```

- `reason`: `ERROR_REGISTRO` o `ANULADA_POR_EMISOR`, en cualquier mayúscula; otro valor, `400`.
- `updatedAt`: obligatorio, el que leyó quien anula (D-193).

**Revisiones:** forma (`400`); la nota (`404 CREDITNOTE_NOT_FOUND`), que se bloquea; `compras.registrar_nota_credito` en toda la empresa y en la marca de cada línea (`403`, D-193); `422 CREDITNOTE_OUTDATED`; `422 CREDITNOTE_ANNULLED`; `422 CREDITNOTE_UNIT_ANNULLED` si la unidad de alguna de sus líneas está anulada, por cualquier vía (D-204), y `422 CREDITNOTE_LINE_RELINKED` si la unidad de una línea que volvería a vigente ya está en otra factura vigente, los dos con `context.lines`; por último `422 CREDITNOTE_SUPPLIER_BALANCE_NEGATIVE` (§2.5.3).

Marca `annulled_at`, `annulled_by` (la cuenta de la sesión) y `annulment_reason`, y devuelve a vigentes las líneas de factura que recuperan valor (§2.6). Escribe una fila delicada en la bitácora (D-193, §13.1): `entity = credit_notes`, `field_name = annulled_at`, `old_value = null`, `new_value` = el momento en ISO, `reason` = el código del motivo, `company_id` = el de la nota y `commercial_brand_id` nulo, porque una nota puede alcanzar varias marcas. Responde `200` con la ficha.

#### 4.2.6 `PUT /credit-notes/{id}`

Cambia el archivo de una nota vigente, y nada de lo registrado (D-204, como la factura por D-172 y D-178). `multipart/form-data` con dos partes, las dos obligatorias: `creditNote` (JSON) y `file`.

```json
{ "updatedAt": "2026-11-03T15:04:00.123Z" }
```

- `updatedAt` es obligatorio: el que leyó quien cambia el archivo (D-204).
- El archivo se verifica como en el alta (D-174). No se quita: se reemplaza.
- Cualquier otro campo del JSON se ignora: el número, la fecha, el motivo y las líneas no cambian por ninguna ruta.

**Revisiones:** forma (`400`, también sin `updatedAt` o sin `file`); archivo (`422 CREDITNOTE_FILE_TYPE`, `CREDITNOTE_FILE_SIZE`; `413 CONTENT_TOO_LARGE` por encima del tope del servidor); la nota (`404 CREDITNOTE_NOT_FOUND`), que se bloquea; `compras.registrar_nota_credito` en toda la empresa y en la marca de cada línea (`403`, D-204); `422 CREDITNOTE_OUTDATED`; `422 CREDITNOTE_ANNULLED`.

Guarda el archivo bajo una llave nueva de su contenido, apunta `file_path` y mueve `updated_at` y `updated_by`. Escribe una fila no delicada en la bitácora (§13.1): `entity = credit_notes`, `field_name = file_path`, la llave anterior —o nada, si no tenía archivo— y la nueva, sin motivo. Responde `200` con la ficha.

#### 4.2.7 `GET /credit-notes/{id}/file`

Entrega los bytes desde `KeptFileStore.open`, con su `Content-Type` y `Content-Disposition: attachment; filename="nota-de-credito-FC01-45.pdf"`: el número normalizado y la extensión de su tipo. Responde `404 CREDITNOTE_NOT_FOUND` si no se lee la nota, y `404 CREDITNOTE_FILE_NOT_FOUND` si no tiene archivo o falta en el almacén.

#### 4.2.8 `GET /credit-note-rules` (propuesta)

```json
{ "singleInvoiceFrom": "2027-01-01" }
```

La fecha de `SingleInvoiceCutoff` (D-192). La pide el formulario para no ofrecer otra factura desde el corte (§6.3), y así la fecha se configura en un solo lugar. Responde lo mismo a todos: es una regla del sistema, no de una empresa.

#### 4.2.9 Lo que cambia en la factura

**`GET /purchase-invoices/{id}`** (fase 2, su §4.2.3) suma:

```json
"creditedTotal": { "amount": 10000.00, "currency": "PEN" },
"totalAfterCreditNotes": { "amount": 462000.00, "currency": "PEN" },
"creditNotes": [
  { "id": "019a…", "documentNumber": "FC01-45", "issueDate": "2026-09-15", "reason": "DESCUENTO_CAMPANA",
    "status": "VIGENTE", "amountOnInvoice": { "amount": 10000.00, "currency": "PEN" } }
],
"items": [ { "…": "…", "creditedAmount": { "amount": 2500.00, "currency": "PEN" },
             "value": { "amount": 3400.00, "currency": "PEN" } } ]
```

- `creditedTotal` y `creditedAmount` suman solo las notas vigentes; `value` es el valor de la línea hoy (§9.6).
- `totalAfterCreditNotes` es el total menos `creditedTotal`. No es lo que se debe, que resta además los pagos (fase 5, §10.4).
- `creditNotes` lista todas las notas que tocan la factura, vigentes y anuladas, por fecha de emisión e `id`, con lo que cada una le resta.
- Lo lee `PurchaseInvoiceCreditsReader`, en dos consultas agrupadas.
- `allowedActions` suma al final `registerCreditNote` (§4.5). El orden de la fase 2 no cambia, y `annul` y `annulAndCopy` dejan de ofrecerse con notas vigentes (PRC-R87).

**`POST /purchase-invoices/{id}/annulment`**: una factura con notas vigentes responde `422 PURCHASEINVOICE_HAS_DEPENDENTS` (fase 2), que deja de ser inalcanzable.

#### 4.2.10 Lo que cambia en la unidad

**`GET /vehicles/{id}/purchase-cost` (D-203)** responde el costo de compra de la unidad. **Revisiones:** la unidad (`404 VEHICLE_NOT_FOUND` si `stock.ver` no la ve); `stock.ver_costos` en su empresa y su marca (`403 ACCESS_DENIED`). Responde:

```json
{
  "vehicleId": "019a…",
  "returnedToSupplier": false,
  "purchase": {
    "invoice": { "id": "019a…", "documentType": "factura", "documentNumber": "F002-981", "invoiceDate": "2026-03-22",
                 "supplierLegalName": "Derco Perú S.A.", "exchangeRate": 3.742000, "taxRate": 0.1800 },
    "invoicedAmount": { "amount": 13100.00, "currency": "USD" },
    "creditNotes": [ { "id": "019a…", "documentNumber": "FC02-17", "issueDate": "2026-08-15",
                       "reason": "DESCUENTO_CAMPANA", "amount": { "amount": 500.00, "currency": "USD" } } ],
    "creditNotesTotal": { "amount": 500.00, "currency": "USD" },
    "vehicleValue": { "amount": 12600.00, "currency": "USD" },
    "costWithoutTax": { "amount": 10677.97, "currency": "USD" },
    "vehicleValueInFunctionalCurrency": { "amount": 47149.20, "currency": "PEN" },
    "costWithoutTaxInFunctionalCurrency": { "amount": 39956.95, "currency": "PEN" }
  }
}
```

`purchase` es `null` sin línea vigente: no hay montos que mostrar, y un monto nunca viaja en `null` (convenciones §5.2). El recurso de la unidad no lleva montos (`/vehicles` sigue sin ellos, [contextos/fleet.md](contextos/fleet.md), sección 1); los trae solo esta ruta.

**`GET /vehicles` y `GET /vehicles/{id}`** suman `returnedToSupplier`: verdadero si la unidad no está anulada y `UnitPurchases.returnedUnitIds` la nombra (D-204). `allowedActions` suma al final `viewPurchaseCost`: verdadero si `stock.ver_costos` lee la empresa y la marca de la unidad. `annul` (fase 2) no cambia: una unidad devuelta con su cesión ya cerrada lo ofrece, porque su valor es cero.

### 4.3 Códigos de error

El estado sale del sufijo (convenciones §7.2): `_NOT_FOUND` es `404`, `_CONFLICT` es `409` y los demás propios, `422`. Un solo prefijo, `CREDITNOTE_`, como pide D-114. Los textos van en `error.<código-en-kebab>.message` de los dos `messages*.properties` y en `es.json`. **`context.lines`**, en los códigos que nombran líneas, es `[{ "line": 3, "purchaseInvoiceItemId": "019a…" }]`, con `line` desde 1 en el orden de `items`; los que comparan suman `fields`, `remainingValue` o `invoiceDate`.

| Código | Estado | Dónde | Mensaje en español |
|---|---|---|---|
| `CREDITNOTE_NOT_FOUND` | 404 | Ficha, `PUT`, anulación, archivo | Nota de crédito no encontrada |
| `CREDITNOTE_FILE_NOT_FOUND` | 404 | `/file` | La nota no tiene archivo |
| `CREDITNOTE_NUMBER_CONFLICT` | 409 | Alta | Ya hay una nota vigente de ese proveedor con ese número |
| `CREDITNOTE_NUMBER_FORMAT` | 422 | Alta | En Perú el número es la serie de cuatro caracteres, un guion y el correlativo de hasta ocho dígitos, como FC01-45 |
| `CREDITNOTE_DATE_IN_FUTURE` | 422 | Alta | La fecha de emisión no puede ser posterior a hoy (`context.today`) |
| `CREDITNOTE_DATE_BEFORE_INVOICE` | 422 | Alta | La fecha de emisión no puede ser anterior a la de una factura que descuenta (`context.lines[].invoiceDate`) |
| `CREDITNOTE_AMOUNT_SCALE` | 422 | Alta | Un monto tiene más decimales de los que permite la moneda (`context.lines`, o sin líneas si es el total) |
| `CREDITNOTE_TOTAL_MISMATCH` | 422 | Alta | La suma de lo repartido no coincide con el total de la nota (`context.linesTotal`, `context.totalAmount`) |
| `CREDITNOTE_DUPLICATE_LINE` | 422 | Alta | Una unidad aparece más de una vez en la nota (`context.lines`) |
| `CREDITNOTE_INVOICE_MISMATCH` | 422 | Alta | Una nota descuenta solo facturas de su empresa, su proveedor y su moneda (`context.lines[].fields`) |
| `CREDITNOTE_LINE_ANNULLED` | 422 | Alta | La nota descuenta una línea o una factura anulada (`context.lines`) |
| `CREDITNOTE_EXCEEDS_VALUE` | 422 | Alta | El monto de una unidad supera lo que le queda de valor (`context.lines[].remainingValue`) |
| `CREDITNOTE_SINGLE_INVOICE_REQUIRED` | 422 | Alta | Desde el corte, una nota descuenta una sola factura (`context.singleInvoiceFrom`) |
| `CREDITNOTE_UNIT_RETURN_PARTIAL` | 422 | Alta | Con «Anulación de unidad», cada unidad recibe todo lo que le queda de valor (`context.lines[].remainingValue`) |
| `CREDITNOTE_INVOICE_ANNULMENT_PARTIAL` | 422 | Alta | Con «Anulación de la factura», la nota deja en cero todas las líneas vigentes de cada factura (`context.invoices`, `context.lines`) |
| `CREDITNOTE_FILE_TYPE` | 422 | Alta, `PUT` | El archivo de la nota debe ser PDF, PNG, JPEG o WebP |
| `CREDITNOTE_FILE_SIZE` | 422 | Alta, `PUT` | El archivo de la nota pesa más de 5 MB |
| `CREDITNOTE_OUTDATED` | 422 | `PUT`, anulación | La nota cambió desde que se abrió |
| `CREDITNOTE_ANNULLED` | 422 | `PUT`, anulación | La nota está anulada: ya no cambia |
| `CREDITNOTE_UNIT_ANNULLED` | 422 | Anulación | Una unidad que la nota dejó sin valor ya está anulada: la nota no se anula (`context.lines`) |
| `CREDITNOTE_LINE_RELINKED` | 422 | Anulación | Una unidad que la nota dejó sin factura vigente ya está en otra factura vigente: la nota no se anula (`context.lines`) |
| `CREDITNOTE_SUPPLIER_BALANCE_NEGATIVE` | 422 | Anulación | Sin esta nota, el saldo a favor del proveedor quedaría negativo: la nota no se anula |

**Reutilizados:** `VALIDATION_ERROR` (400), `ACCESS_DENIED` (403), `CONTENT_TOO_LARGE` (413), `COMPANY_NOT_FOUND`, `SUPPLIER_NOT_FOUND`, `CURRENCY_NOT_FOUND`, `PURCHASEINVOICE_NOT_FOUND` (con `context.lines`) y `VEHICLE_NOT_FOUND` (404), `COMPANY_INACTIVE` y `PURCHASEINVOICE_HAS_DEPENDENTS` (422). Las formas de `context` se documentan en `ErrorResource` y en las respuestas `422` de cada operación (convenciones §7.1).

### 4.4 Avisos que se confirman

No hay ninguno. Una unidad devuelta que no se puede anular no se pregunta: D-191 dice que la nota se registra igual, y el formulario no puede saber antes si la unidad tiene una cesión abierta o contratos. La ficha lo dice después (§6.4).

### 4.5 `allowedActions`

Solo la ficha y las respuestas de las escrituras lo traen; las filas de la lista, `null`. Siempre en este orden:

| Acción | Cuándo se ofrece |
|---|---|
| `edit` | Nota vigente y `compras.registrar_nota_credito` en toda la empresa y en la marca de cada línea: cambiar su archivo (D-204) |
| `annul` | Nota vigente; el mismo permiso con el mismo alcance; ninguna unidad de sus líneas está anulada, y ninguna línea que volvería a vigente tiene su unidad vinculada a otra factura vigente; y `SupplierBalanceAfterAnnulment` no responde «negativo» |
| `downloadFile` | Tiene archivo |

Las tres exigen además leer la nota con `compras.ver`. Lo calcula `CreditNoteAllowedActions`, sin candado: la operación vuelve a revisar todo bajo los suyos.

**En la factura**, `registerCreditNote` va al final, después de `downloadXml`: factura vigente, con al menos una línea vigente, y `compras.registrar_nota_credito` en toda su empresa y en la marca de alguna de sus líneas vigentes. **En la unidad**, `viewPurchaseCost` va al final de `VehicleAllowedActionsView` (§4.2.10).

### 4.6 Versiones y concurrencia

- **El candado de `updatedAt`** en el cambio del archivo y en la anulación, comparado al milisegundo bajo el candado de la fila (convenciones §3, D-193, D-204). La pantalla, ante `CREDITNOTE_OUTDATED`, vuelve a leer la ficha, la muestra como está y lo dice.
- **Dos notas sobre la misma línea** se ordenan en el candado de su factura (§2.4): la segunda ve lo que restó la primera y, si se pasa, responde `422 CREDITNOTE_EXCEEDS_VALUE`.
- **El doble envío** del mismo alta espera al primero en el candado de las facturas y responde `409 CREDITNOTE_NUMBER_CONFLICT`.
- **La anulación de una nota y el registro de una factura** que vincula la misma unidad se ordenan en el candado de la unidad: gana la primera; la otra responde `422 CREDITNOTE_LINE_RELINKED` o `409 PURCHASEINVOICE_UNIT_CONFLICT`.
- **La versión de la API** no cambia (convenciones §1): los cambios de la factura y de la unidad agregan campos, y las rutas son nuevas.

---

## 5. Reglas

Cada regla pasa a ✅ cuando su prueba existe y pasa. Las clases de prueba del backend siguen los paquetes de hoy; las del frontend van junto a su archivo.

### 5.1 `procurement`

Continúan la numeración de la fase 2, cuya última es PRC-R67.

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| PRC-R68 | La nota se registra a mano desde el documento, con su cabecera y su reparto por unidad. El sistema nunca reparte por su cuenta entre las unidades de la factura, y una nota nunca reemplaza a otra | §9.5, D-193, guía §7.1 y A-04 | Servicio: `CreditNoteCommandServiceImpl`. Recurso: cada línea nombra su línea de factura | `CreditNotesIntegrationTests.aNoteDiscountsOnlyTheUnitsItNames`, `aSecondNoteOfAUnitKeepsTheFirst` | ❌ |
| PRC-R69 | El número se guarda normalizado y no se repite en el proveedor entre las notas no anuladas; anular una libera su número. En Perú sigue el formato de D-160 | D-193, D-160, D-204 | Dominio: `InvoiceNumber`. Base: `credit_notes_number_uq`. Adaptador: `NaturalKey.CREDIT_NOTE_NUMBER` → `409` | `CreditNotesIntegrationTests.theSameNumberOfTheSupplierAnswersNumberConflict`, `anAnnulledNoteFreesItsNumber`, `aPeruvianNoteNumberFollowsTheFormatOfInvoices`; `CreditNoteLockTests.twoRegistrationsOfTheSameNumberLeaveOneNote` | ❌ |
| PRC-R70 | La fecha de emisión no es futura en la zona de la empresa ni anterior a ninguna factura que la nota descuenta. No hay excepción de antigüedad: una nota de más de 20 días se registra con el mismo permiso | D-193, D-205, §9.5, guía columna 39 | Dominio: `InvoiceDatePolicy`, `CreditNoteRegistrationPolicy`. Base: `credit_note_items_invoice_tg` | `CreditNoteRegistrationPolicyTests.aNoteIsNotDatedBeforeItsInvoice`; `CreditNotesIntegrationTests.aNoteDatedTomorrowInLimaIsRefused`, `aNoteOfMoreThanTwentyDaysNeedsNoOtherPermission` | ❌ |
| PRC-R71 | Una nota tiene de 1 a 100 líneas y nombra cada línea de factura una sola vez | §9.5, D-204 | Recurso: `@NotEmpty @Size(max = 100)`. Dominio: constructor. Base: `credit_note_items_note_line_uq` | `CreditNoteTests.aNoteHasOneToOneHundredLinesAndNoneTwice`; `CreditNotesIntegrationTests.aHundredAndOneLinesAnswerValidationError` | ❌ |
| PRC-R72 | Una nota descuenta solo líneas vigentes de facturas vigentes de su empresa, su proveedor y su moneda: nunca una línea ni una factura anulada | D-193, §9.5, D-192, §2.2 y A-06 | Servicio: `CreditedLines`, bajo el candado de las facturas. Base: llaves compuestas y `credit_note_items_invoice_tg` | `CreditNotesIntegrationTests.anAnnulledLineReceivesNoCredit`, `aLineOfAnAnnulledInvoiceReceivesNoCredit`, `aLineOfAnotherSupplierAnswersInvoiceMismatch`, `aLineInAnotherCurrencyAnswersInvoiceMismatch`; `CreditNoteTriggerTests.theDatabaseRefusesALineOfAnAnnulledInvoice`, `theDatabaseRefusesALineInAnotherCurrency`; `ProcurementCompanyIsolationIntegrationTests.aNoteCannotNameALineOfAnotherCompany` | ❌ |
| PRC-R73 | El archivo de una nota vigente se cambia después con `PUT`, con el `updatedAt` leído y el alcance de registrar; nada de lo registrado cambia, la llave anterior y sus bytes se quedan, y el cambio deja su fila no delicada en la bitácora | D-204, D-172, D-178, §13.1 | Dominio: `CreditNote.replaceFile`, `isAsOpenedAt`. Servicio: `CreditNoteCommandServiceImpl` con `CreditNoteFiles`. Base: `credit_notes_data_tg`. Frontend: «Cambiar archivo» en la ficha | `CreditNoteTests.anAnnulledNoteKeepsItsFile`; `CreditNotesIntegrationTests.theFileOfACurrentNoteIsReplacedAndNothingElseChanges`, `aReplacementOpenedBeforeTheLastChangeIsOutdated`, `aReplacementLeavesItsRowInTheChangeLog`, `aBrandGrantOfAnotherLineCannotReplaceTheFile`; `CreditNoteFilesTests.aRolledBackReplacementDiscardsItsObjectAndKeepsTheOldKey`; `credit-note-profile-view.spec` ("replaces the file and re-reads on outdated") | ❌ |
| PRC-R74 | Cada línea copia la tasa de su factura —1 en la moneda funcional—, y la tasa no viaja en el pedido | D-152, D-169, D-192 | Servicio: `CreditedLines`. Base: `credit_note_items_invoice_tg` | `CreditNotesIntegrationTests.eachLineCopiesTheRateOfItsInvoiceAcrossTwoInvoices`; `CreditNoteTriggerTests.theDatabaseRefusesARateThatIsNotTheOneOfTheInvoice` | ❌ |
| PRC-R75 | Todo monto es mayor que cero, en la moneda de la nota y con a lo sumo sus decimales, y el total es la suma exacta de las líneas. No hay notas de débito | §9.5, D-146, D-23 | Dominio: `InvoiceAmount`, constructor. Base: `NUMERIC(14,2)` y los `CHECK` | `CreditNoteTests.theTotalIsTheExactSumOfItsLines`; `CreditNotesIntegrationTests.aNegativeAmountAnswersValidationError`, `anAmountWithMoreDecimalsThanItsCurrencyAnswersAmountScale` | ❌ |
| PRC-R76 | Una línea no baja de cero: lo que restan sus notas vigentes no supera su valor de factura, y se cuenta bajo el candado de su factura | D-193, §9.6 | Dominio: `LineValue`, `CreditNoteRegistrationPolicy`. Servicio: `CreditedLines`. Base: `credit_note_items_line_value_tg` | `LineValueTests.aCreditAboveTheRemainingValueIsRefused`; `CreditNoteLockTests.twoNotesOnTheSameLineDoNotTakeItBelowZero` | ❌ |
| PRC-R77 | Desde la fecha de corte, configurable y hoy 01/01/2027, una nota descuenta una sola factura, según su fecha de emisión; antes, varias del mismo proveedor, empresa y moneda | D-192 | Dominio: `SingleInvoiceCutoff`. Configuración: `procurement.credit-notes.single-invoice-from`. Ruta: `GET /credit-note-rules` | `SingleInvoiceCutoffTests.theLastDayOf2026AllowsTwoInvoicesAndTheFirstOf2027DoesNot`; `CreditNotesIntegrationTests.aCutoffMovedByConfigurationIsTheOneApplied` | ❌ |
| PRC-R78 | El motivo es descuento de campaña, anulación de unidad, anulación de la factura o ajuste de precio, guardado como `DESCUENTO_CAMPANA`, `ANULACION_UNIDAD`, `ANULACION_FACTURA` o `AJUSTE_PRECIO`. El mes de campaña se guarda con día 1 y solo con descuento de campaña | §9.5, D-204 | Dominio: `CreditNoteReason`, `CampaignMonth`. Base: `credit_notes_reason_ck`, `credit_notes_campaign_month_ck` | `CampaignMonthTests.aMonthIsItsFirstDay`; `CreditNotesIntegrationTests.aCampaignMonthWithAnotherReasonAnswersValidationError` | ❌ |
| PRC-R79 | Una línea que las notas dejan en cero se anula en la misma transacción; su unidad queda sin factura vigente y la vincula la factura que la reemplace, y lo pendiente de su orden la vuelve a contar | D-191, D-160, D-162 | Dominio: `PurchaseInvoice.annulLineCreditedToZero`. Servicio: `CreditNoteLineEffects`. Repositorio: `saveLineStatuses` | `CreditNotesIntegrationTests.aLineLeftAtZeroIsAnnulledAndItsUnitIsLinkedByTheNextInvoice`, `aLineLeftAtZeroCountsAgainInThePendingOfItsOrder` | ❌ |
| PRC-R80 | Solo «anulación de unidad» anula la unidad, por `fleet` y sin `stock.anular_vehiculo`. Con contratos o una cesión abierta, la nota se registra igual y la unidad queda devuelta y sin anular | D-104, D-191, D-182 | Servicio: `CreditNoteUnits`, con `FleetContextFacade.annulReturnedUnits` | `CreditNotesIntegrationTests.aUnitReturnedToItsSupplierIsAnnulledWithoutTheStockPermission`, `aReturnedUnitWithAnOpenAssignmentIsKeptAndTheNoteIsRegistered`, `aCampaignNoteLeavingALineAtZeroDoesNotAnnulTheUnit` | ❌ |
| PRC-R81 | «Anulación de unidad» deja en cero cada línea que descuenta, y «anulación de la factura», todas las líneas vigentes de cada factura que nombra; un monto parcial con esos motivos se rechaza | D-191, D-204 | Dominio: `CreditNoteRegistrationPolicy`. Frontend: los montos fijos en el formulario | `CreditNoteRegistrationPolicyTests.aUnitReturnCreditsTheWholeRemainingValue`, `anInvoiceAnnulmentCreditsEveryCurrentLineOfItsInvoices` | ❌ |
| PRC-R82 | Registrar y anular exigen `compras.registrar_nota_credito` en toda la empresa y en la marca de cada línea, sin sede. Las notas se leen por empresa con `compras.ver`, y escribir exige leer | D-193, D-163, D-178, D-134 | `CreditNoteAccessPolicy` | `CreditNoteAccessPolicyTests.aBranchGrantDoesNotRegister`, `aBrandGrantRegistersOnlyItsBrand`; `CreditNotesIntegrationTests.aLineOfABrandTheCallerDoesNotActOnAnswersAccessDenied`, `aWriterWithoutVerDoesNotReadNorWriteTheNotesOfACompany` | ❌ |
| PRC-R83 | El archivo es PDF, PNG, JPEG o WebP de hasta 5 MB, verificado por su contenido. Se guarda en `kept_files` con la empresa de la nota, después de validar todo, y se descarta si la transacción se revierte | D-174, D-189, E15 | `CreditNoteFiles`, `UploadedFileType`, `KeptFileStore.discardRolledBack` | `CreditNoteUploadLimitsTests.anHtmlRenamedPdfIsRefused`, `aFileOverFiveMegabytesAnswersFileSize`; `CreditNoteFilesTests.aRolledBackRegistrationDiscardsItsObject` | ❌ |
| PRC-R84 | Anular exige un motivo y el `updatedAt` leído, procede una sola vez y deja su fila delicada en la bitácora | D-193, §13.1 | Dominio: `CreditNote.annul`, `isAsOpenedAt`. Servicio: `CreditNoteAnnulments` con `ChangeLogCommandService.recordChange` | `CreditNotesIntegrationTests.annullingNeedsItsReasonAndTheUpdatedAtItRead`, `theAnnulmentLeavesItsRowInTheChangeLogWithItsReason`, `anAnnulledNoteIsNotAnnulledAgain` | ❌ |
| PRC-R85 | Anular la nota devuelve a vigentes las líneas que recuperan valor. No procede si la unidad de alguna de sus líneas está anulada, por cualquier vía, o si la de una línea que vuelve ya está en otra factura vigente | D-193, D-204, §9.6 | Servicio: `CreditNoteAnnulments`, con `FleetContextFacade.lockUnitsByIds`. Dominio: `PurchaseInvoice.reinstateLine` | `CreditNotesIntegrationTests.annullingTheNoteBringsItsLinesBack`, `aNoteWhoseUnitWasAnnulledIsNotAnnulled`, `aNoteWhoseUnitIsInAnotherInvoiceIsNotAnnulled`, `aLineStillCoveredByAnotherNoteStaysAnnulled` | ❌ |
| PRC-R86 | Anular la nota no procede si el saldo del proveedor quedaría negativo: se pregunta a `SupplierBalanceAfterAnnulment` con el proveedor bloqueado, y hasta la fase 5 ningún bean responde | D-193, D-190, D-195 | Servicio: `CreditNoteAnnulments` con `List<SupplierBalanceAfterAnnulment>` | `SupplierBalanceAfterAnnulmentTests.untilPaymentsExistNoAnnulmentLeavesTheBalanceNegative`; `CreditNoteAnnulmentsTests.aGuardThatSaysNegativeRefusesTheAnnulment` | ❌ |
| PRC-R87 | Una factura con notas vigentes no se anula, y su `allowedActions` no ofrece `annul` ni `annulAndCopy`; anular la nota la libera | D-161, H3 | `CreditNoteInvoiceDependentsAdapter`, `PurchaseInvoiceAllowedActions` | `PurchaseInvoiceDependentsTests.untilPaymentsExistOnlyCreditNotesAreDependents`; `PurchaseInvoicesIntegrationTests.anInvoiceWithACurrentNoteAnswersHasDependents`, `anInvoiceWhoseNoteWasAnnulledIsAnnulled`; `CreditNoteLockTests.aNoteAndTheAnnulmentOfItsInvoiceDoNotCross` | ❌ |
| PRC-R88 | Las escrituras de notas toman los candados en el orden de §2.4, y una carrera que la base detiene responde el código de su llave | Esta especificación, §2.4 | Servicio y adaptadores | `CreditNoteLockTests.anAnnulmentAndAnInvoiceLinkingTheSameUnitDoNotCross`, `aNoteAndAPutOfItsInvoiceDoNotDeadlock` | ❌ |
| PRC-R89 | La ficha de la factura dice lo que sus notas vigentes restan a cada línea y al total, lista sus notas vigentes y anuladas, y ofrece registrar una | §9.6, D-199 | `PurchaseInvoiceCreditsReader`, `PurchaseInvoiceViewFactory`, `PurchaseInvoiceAllowedActions` | `PurchaseInvoicesIntegrationTests.theRecordTellsWhatItsNotesTakeFromEachLine`, `registeringANoteIsOfferedOnlyWithACurrentLineAndThePermission` | ❌ |
| PRC-R90 | Compras responde a `fleet` si el valor de una unidad es cero —sin línea vigente, o con su línea en cero por notas—, su costo de compra y qué unidades devolvió una nota | §9.6, D-159, D-191, D-199 | `PurchaseInvoiceUnitPurchasesAdapter` | `PurchaseInvoiceUnitPurchasesAdapterTests.aUnitWhoseLineNotesLeftAtZeroHasZeroValue`, `theCostOfAUnitComesFromItsCurrentLineAndItsCurrentNotes`, `aReturnedUnitIsNamedUntilItsNoteIsAnnulled` | ❌ |
| PRC-R91 | El valor del vehículo es su valor de factura menos sus notas vigentes. El costo sin IGV lo divide entre 1 + la tasa en una factura, y no en una boleta u otro. En la moneda de la empresa se convierte con la tasa de la factura y se redondea una sola vez | D-159, D-192, D-199, guía columnas 41, 48 y 49 | Dominio: `UnitPurchaseCostCalculator` | `UnitPurchaseCostCalculatorTests.aCarInDollarsCostsWhatTheExampleSays`, `aBoletaCostsItsWholeValue`, `theCostInSolesIsRoundedOnceAtTheEnd` | ❌ |
| PRC-R92 | Compras entrega a la fase 5 lo que las notas vigentes restan de cada factura y de cada línea, en su moneda | D-190, D-199 | `CreditNoteRepository.currentCreditsByInvoiceIds`, `currentCreditsByInvoiceItemIds` | `CreditNoteRepositoryImplTests.anAnnulledNoteTakesNothingFromItsInvoiceNorItsLines` | ❌ |
| PRC-R93 | `allowedActions` de la nota sale en el orden `edit`, `annul`, `downloadFile`, con las reglas de cada operación | D-115, D-204 | `CreditNoteAllowedActions` | `CreditNoteAllowedActionsTests.theActionsComeInTheOrderOfTheSpecification`, `anAnnulledNoteOffersOnlyItsDownload`, `aNoteWhoseUnitWasAnnulledDoesNotOfferAnnul` | ❌ |
| PRC-R94 | La lista se busca, se filtra, se ordena y se exporta con las 17 columnas de §4.2.2, con el total en la moneda de la empresa rotulado «(calculado)» y el alcance de `compras.ver` | D-106, D-163, D-180 | `CreditNotesController`, `CreditNoteSpreadsheetColumns`, `ProcurementDocumentKinds` | `CreditNotesExportIntegrationTests.theListIsExportedWithItsColumnsAndTheCalculatedTotal`, `aWriterWithoutVerInBDoesNotGetBInTheExport`; `CreditNoteSearchIntegrationTests.theSearchFindsANoteWhateverItsAccents` | ❌ |
| PRC-R95 | Las dos tablas llevan las tres políticas de V83, la cuenta de la aplicación no borra y las dos se versionan en `data_versions`. Una nota anulada no cambia, sus datos no se editan y sus líneas nunca se tocan. Los decimales de la moneda de una nota quedan fijos por su factura, sin otro adaptador de `CurrencyUsage` | §2.5, V83, D-128, D-193, §4.1 | Base: V(n+1). La fase 2: `PurchaseInvoiceCurrencyUsageAdapter` | `ProcurementCompanyIsolationIntegrationTests.theNotesOfAnotherCompanyAreNotSeen`; `ProcurementRowsAreNeverDeletedTests` (las dos tablas); `RowLevelSecurityMigrationTests`; `JdbcDataVersionsTests.everyVersionedTableHasItsSixtyFourSlotsAndItsTwoTriggers`; `CreditNoteTriggerTests.anAnnulledNoteNeverChanges`, `aLineOfANoteIsNeverEdited`; `CurrencyDecimalPlacesLockTests.aCreditNoteFixesTheDecimalPlacesThroughItsInvoice` | ❌ |
| PRC-R96 | Una línea de una factura vigente vuelve a vigente solo cuando sus notas vigentes le dejan valor, y la de una factura anulada nunca cambia | D-193, D-161 | Base: `keep_purchase_invoice_item_fiscal_data` reemplazada en V(n+1) | `PurchaseInvoiceFiscalTriggerTests.aLineComesBackOnlyWithValueAndNeverOnAnAnnulledInvoice` | ❌ |
| PRC-R97 | `compras.registrar_nota_credito` se siembra a SA, AE y CO, y los tres permisos de facturas se marcan en AE, OP y CO según la matriz, en la última migración de la fase | D-176, §5.6 | Base: V(n) y V(n+2). `PermissionCodes` | `CreditNotePermissionSeedMigrationTests.theNotePermissionFollowsTheMatrix`; `InvoicePermissionsMatrixMigrationTests.theInvoicePermissionsFollowTheMatrixOnceCreditNotesExist`; `PermissionCatalogConsistencyTests` | ❌ |
| PRC-R98 | Una nota de 100 líneas se registra en una transacción, y las sentencias que no son inserciones ni actualizaciones de líneas no crecen con ellas | Plan §5, A4, A5 | `CreditNoteCommandServiceImpl`, `CreditedLines` | `CreditNoteRegistrationStatementsTests.theQueriesOfARegistrationDoNotGrowWithItsLines`; `CreditNotesIntegrationTests.aFailureOnLineOneHundredLeavesNoRowAndNoFile` | ❌ |
| PRC-R99 | El formulario reparte en partes iguales entre las unidades elegidas, con los céntimos sobrantes en la primera, y muestra el valor que le queda a cada una; no envía hasta que lo repartido cuadra con el total y ninguna línea se pasa | D-193, guía columna 40 | Frontend: `credit-note-split.ts` con `BigInt`, `credit-note-lines` | `credit-note-split.spec` ("splits 10 000 among four as 2 500 each", "gives the leftover cent of 100.00 among three to the first"); `credit-note-form-view.spec` ("does not send while a line exceeds its remaining value") | ❌ |

### 5.2 `fleet`

Continúan la numeración de la fase 3, cuya última es FLT-R93.

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| FLT-R94 | `fleet` anula las unidades que una nota devuelve por la primera vía de D-104, sin pedir `stock.anular_vehiculo`, con la misma guarda que la ruta explícita: valor cero, sin contratos y sin cesión abierta. La que no pasa se devuelve retenida con su causa y no se escribe | D-104, D-191, D-182, §9.6 | `FleetContextFacadeImpl.annulReturnedUnits`, `VehicleAnnulments.annulmentHold` | `FleetContextFacadeImplTests.aReturnedUnitIsAnnulledInsideTheTransactionOfTheNote`, `aReturnedUnitWithAnOpenAssignmentIsHeldAndNotWritten`, `annullingReturnedUnitsOutsideATransactionFails` | ❌ |
| FLT-R95 | La anulación de una nota bloquea las unidades de las líneas que vuelven, por `id` ascendente y con el candado de toda escritura de la unidad, y una unidad de otra empresa no aparece | D-193, §2.4 | `FleetContextFacadeImpl.lockUnitsByIds` | `FleetContextFacadeImplTests.lockingUnitsHoldsThemUntilTheEndOfTheTransaction`, `aUnitOfAnotherCompanyIsNotLocked` | ❌ |
| FLT-R96 | `fleet` pregunta a compras el costo de compra y las unidades devueltas por `UnitPurchases`, sin depender de `procurement`, y exactamente un bean responde | Mapa §5, regla 1.1; FLT-R53 de la fase 2 | `fleet/interfaces/acl`, `UnitPurchaseFacts` | `ModularityTests`; `UnitPurchasesWiringTests.procurementIsTheOnlyAnswerOfThePurchasesOfAUnit` | ❌ |
| FLT-R97 | El costo de compra de una unidad lo lee solo quien tiene `stock.ver_costos` en su empresa y su marca, por su propia ruta; el recurso de la unidad nunca lleva montos, y `allowedActions.viewPurchaseCost` lo anuncia | §5.6, D-102, D-203 | `VehiclesController`, `VehicleAccessPolicy.seesCosts`, `VehicleAllowedActions` | `VehiclePurchaseCostIntegrationTests.aReaderWithoutCostsGetsAccessDenied`, `aUnitNotSeenIsNotFound`, `aUnitWithoutCurrentInvoiceHasNoPurchase`; `VehicleAllowedActionsTests.viewPurchaseCostFollowsTheCostsOfItsBrand` | ❌ |
| FLT-R98 | El Inventario y la ficha de la unidad dicen qué unidad volvió al proveedor y sigue sin anular, sin guardarlo | D-191, D-204 | `VehicleViewFactory` (una llamada por página), `VehicleQueryServiceImpl`. Frontend: insignia «Devuelta al proveedor» | `VehiclesIntegrationTests.aReturnedUnitIsFlaggedUntilItIsAnnulled`; `vehicles-view.spec` ("flags a unit returned to its supplier") | ❌ |
| FLT-R99 | Una unidad devuelta y retenida se anula después por la vía explícita, con `stock.anular_vehiculo`, en cuanto se cierra su cesión: su valor ya es cero | D-104, D-182, D-191 | `VehicleAnnulments` con `UnitPurchaseFacts.valueIsZero` | `VehicleAnnulmentsIntegrationTests.aReturnedUnitIsAnnulledOnceItsAssignmentIsClosed` | ❌ |

---

## 6. Pantallas

### 6.1 Rutas y menú

| Ruta del navegador | Vista | `data.permissions` | Encabezado |
|---|---|---|---|
| `/procurement/credit-notes` | `CreditNotesView` | `anyOf: [compras.ver]` | El del módulo, `modules.credit-notes` (ya existe en `es.json`) |
| `/procurement/credit-notes/new` (y `?facturaId={id}`) | `CreditNoteFormView` | `allOf: [compras.ver]`, `anyOf: [compras.registrar_nota_credito]` | `credit-notes.heading.new-title` y `new-description` |
| `/procurement/credit-notes/{creditNoteId}` | `CreditNoteProfileView` | `anyOf: [compras.ver]` | El número y el proveedor, con `overridePageHeading` |

- **`app-navigation.ts`:** la opción `/procurement/credit-notes` (hoy con `pending: true`, `app-navigation.ts:140-146`) pierde `pending` y declara `access: anyOf(PermissionCodes.Procurement.VIEW)`.
- **`iam/application/permission-codes.ts`** suma `Procurement.REGISTER_CREDIT_NOTES`. `Stock.VIEW_COSTS` ya existe (`permission-codes.ts:81`).
- **La matriz de permisos:** `purchasingPlace` de `permission-matrix-groups.ts` lleva hoy todo lo que no termina en `_plantillas` ni `_ordenes` a Proveedores (`permission-matrix-groups.ts:178-186`). Suma `_nota_credito` → `notas-de-credito`, pantalla que `SCREEN_ORDER` ya declara (`:29`). Si la fase 2 no llevó `_factura` y `_factura_antigua` a `facturas`, este commit lo hace. `es.json` suma `iam.permission-matrix.items.compras.registrar_nota_credito`: «Registrar y anular notas de crédito».

### 6.2 Lista

- **Acciones** (`.mm-page-actions`): «Registrar nota», solo con `hasPermission(REGISTER_CREDIT_NOTES)`, y «Exportar».
- **Filtros:** empresa (`mm-company-filter`), proveedor (`mm-search-select`, activos e inactivos), motivo, estado (Todas, Vigentes, Anuladas), moneda y período Desde–Hasta, que incluye el último día elegido (la pantalla manda `to` como el día siguiente).
- **Columnas:**

  | Columna | Contenido | Ordena |
  |---|---|---|
  | Fecha | `dd/mm/aaaa` | Sí |
  | Nota | Número; abre la ficha | Sí |
  | Proveedor | Razón social y, debajo, el documento | Sí |
  | Motivo | Traducido | No |
  | Facturas | La primera y «y N más» | No |
  | Unidades | Cuántas | No |
  | Total | Con su moneda | Sí |
  | Estado | Insignia Vigente o Anulada | No |

- **Menú de la fila:** «Ver ficha».
- **Estados:** esqueleto (`mm-skeleton`, revisión 33) mientras carga; vacío, «Todavía no hay notas de crédito registradas», con «Registrar nota» si se puede; error, con «Reintentar».

### 6.3 Formulario

Una página con dos paneles, registrada en `UnsavedWork`: salir con cambios pregunta con `UnsavedChangesQuestion`. Al abrirse lee el corte (`GET /credit-note-rules`). Con `?facturaId=` llega con la empresa, el proveedor, la moneda y esa factura elegidos, y abre el diálogo de unidades sobre ella.

**Panel «Datos de la nota»:**

| Campo | Comportamiento |
|---|---|
| Empresa | Solo si la sesión registra notas en más de una |
| Proveedor | `mm-search-select` con los proveedores de la empresa, activos e inactivos (D-204) |
| Número | Muestra debajo cómo se guardará (`Se guarda como FC01-45`); en Perú avisa el formato antes de enviar (D-204) |
| Fecha de emisión | Calendario hasta hoy en la zona de la empresa, y desde la fecha de la factura más reciente de las líneas elegidas |
| Motivo | Cuatro opciones. Debajo, una línea dice su efecto: «Descuento de campaña: baja el valor de las unidades.»; «Ajuste de precio: baja el valor de las unidades.»; «Anulación de la factura: deja en cero todas las líneas de la factura; sus unidades quedan sin factura vigente.»; «Anulación de unidad: las unidades vuelven al proveedor y se anulan, salvo las que tengan contratos o una cesión abierta.» |
| Mes de campaña | Solo con descuento de campaña: mes y año (D-204). Cambiar de motivo lo vacía |
| Moneda | La de la primera factura elegida, de solo lectura. Antes de elegir una, «Se toma de la factura» |
| Total de la nota | Monto, con los decimales de la moneda |
| Archivo | Selector con su nombre y su peso; avisa del tipo y del tope antes de enviar |

**Panel «Unidades»:**
- **Barra:** «Agregar unidades», «Repartir en partes iguales» y, con las líneas de una sola factura, la factura elegida con su número.
- **«Agregar unidades»** abre un diálogo:
  - Elige una factura vigente del proveedor en la moneda de la nota (`GET /purchase-invoices?companyId&supplierId&status=VIGENTE&currency`), con `mm-search-select` por su número.
  - Lista sus líneas vigentes (`GET /purchase-invoices/{id}`) con VIN, modelo, año, valor de factura, lo que ya restan sus notas y el valor que le queda, cada una con su casilla y una búsqueda por VIN.
  - «Agregar N unidades» suma las elegidas. Con «Anulación de la factura» suma todas, sin casillas (D-204).
  - Desde el corte, si la nota ya tiene una factura, el diálogo ofrece solo esa y lo dice: «Desde el 01/01/2027 una nota descuenta una sola factura.». Antes del corte ofrece otras del mismo proveedor y moneda.
- **Tabla de líneas**, fuera de `mm-data-manager` como la de la factura (revisión 34): N.°, Factura, VIN, Modelo, Año, Valor de factura, Notas anteriores, Valor actual, Monto de esta nota, Valor después y quitar.
- **Los montos:**
  - Con «Anulación de unidad» o «Anulación de la factura», cada monto es el valor actual de su línea, fijo (D-204).
  - Con los otros motivos se escriben, o los llena «Repartir en partes iguales»: el total de la nota entre las líneas, en enteros, con los céntimos sobrantes en la primera (D-193, guía columna 40). 100,00 entre tres da 33,34, 33,33 y 33,33.
  - Una línea que se pasa de su valor actual se marca: «Supera lo que le queda a la unidad: 3 400,00».
- **Totales,** al pie y fijos en el teléfono: lo repartido frente al total de la nota, con «Cuadra» o la diferencia («Faltan PEN 2 500,00»); unidades; y el total en la moneda de la empresa con «(calculado)». Se calculan con `computed` y `BigInt` (`amountMinor`, `sumMinor` y `fromMinor` de `purchase-order-amounts.ts`).
- **Registrar nota:** no se envía mientras no cuadre, falten líneas o una se pase, y dice por qué. Un error con `context.lines` marca esas líneas. `CREDITNOTE_SINGLE_INVOICE_REQUIRED` marca las facturas sobrantes con el texto del corte.

### 6.4 Ficha

- **Panel de datos**, encabezado por el número y el proveedor: empresa, fecha, motivo, mes de campaña, moneda, total, total en la moneda de la empresa con «(calculado)», facturas (enlaces a su ficha) y estado.
- **Línea de estado:** una nota anulada lo dice una vez junto a su insignia: «Anulada el 03/11/2026 15:04 por Tere Ramos: error de registro.».
- **El aviso de las unidades devueltas** (`.mm-notice--warning`), si alguna línea trae `returnedToSupplier`: «La unidad MD2A11CY6LWA12345 volvió al proveedor y no se anuló: tiene contratos o una cesión abierta. Ciérrela y anule la unidad desde Inventario.». Con varias, una lista.
- **Cabecera**, según `allowedActions`:

  | Botón | Acción |
  |---|---|
  | «Anular» | Un diálogo con el motivo en dos opciones, «Error de registro» y «Anulada por el emisor», y el texto «Sus unidades recuperan lo que la nota descontaba; las líneas que dejó en cero vuelven a su factura.». Botón destructivo «Anular nota». Manda `updatedAt` y trata `OUTDATED` volviendo a leer. `CREDITNOTE_UNIT_ANNULLED` y `CREDITNOTE_LINE_RELINKED` nombran el VIN de la línea |
  | «Cambiar archivo» | Un diálogo de edición (`CreateAndEditDialog`) con el selector de archivo, que nombra los tipos y el peso admitidos: «Reemplaza el documento de la nota; el anterior se conserva.». Manda `updatedAt` y trata `OUTDATED` volviendo a leer (D-204) |
  | «Descargar documento» | Descarga el archivo |

- **Panel «Unidades»:** la tabla de líneas, de solo lectura: factura (enlace), VIN (abre `/fleet/vehicles/{id}`), placa, marca y modelo, año, valor de factura, monto de esta nota, tipo de cambio, monto en la moneda de la empresa, valor de la línea hoy y su estado. La unidad: «Anulada», «Devuelta, sin anular» o nada.
- **Estados:** esqueleto, «No encontramos esa nota» con vuelta a la lista, error con «Reintentar».

### 6.5 La ficha de la factura

Cambia la ficha de la fase 2 (su §6.4):
- **Panel de datos:** suma «Notas de crédito», el total que restan, y «Total después de notas».
- **Panel «Unidades»:** suma las columnas «Notas» y «Valor». Una línea anulada por notas en una factura vigente lo dice en su estado: «Anulada por notas».
- **Panel «Notas de crédito»**, nuevo, después de «Unidades»: una tabla simple con Nota (enlace a su ficha), Fecha, Motivo, Monto en esta factura y Estado; vacía, «Esta factura no tiene notas de crédito.».
- **Su cabecera** ofrece «Registrar nota de crédito» por `allowedActions.registerCreditNote`, que abre `/procurement/credit-notes/new?facturaId={id}`.
- **«Anular» y «Anular y copiar»** ya no se ofrecen con notas vigentes. Si el backend responde `PURCHASEINVOICE_HAS_DEPENDENTS`, la ficha muestra «La factura tiene notas de crédito vigentes: anúlelas antes de anular la factura.».

### 6.6 La unidad

- **Inventario:** la insignia «Devuelta al proveedor» en la fila de una unidad con `returnedToSupplier`, junto a «Sin factura vigente» de la fase 2 (D-204).
- **Ficha de la unidad:** la misma insignia, con la línea «Volvió al proveedor y sigue sin anular: cierre sus cesiones abiertas y anúlela.». «Anular unidad» aparece por `allowedActions.annul` en cuanto la guarda de la fase 2 lo deja (FLT-R99).
- **Panel «Compra y costo»**, nuevo, entre el panel de datos y las pestañas (`vehicle-profile-view.html:121-127`), solo con `allowedActions.viewPurchaseCost`. Se lee al abrir la ficha, por su ruta.
  - Muestra la factura (número, proveedor y fecha; el número enlaza a su ficha en otra pestaña, que pide `compras.ver`), el valor de factura, las notas de crédito (número, fecha, motivo y monto), el valor del vehículo, el costo sin IGV con «(calculado)» y, en la moneda de la empresa, el valor y el costo sin IGV con «(calculado)» y la tasa de la factura: «3,742000 de la factura».
  - Sin `purchase`: «Sin factura vigente: su valor es cero.».
  - En la moneda funcional no repite los montos convertidos.

### 6.7 A 375 px y accesibilidad

**A 375 px:**
- La lista es la de `mm-data-manager`, que no tiene modo de tarjetas: su tabla se desplaza de lado.
- La tabla de líneas mantiene fijos a la izquierda el N.° y el VIN mientras se desplaza de lado, y los totales van en la barra fija al pie de la revisión 34 (`.mm-foot-bar`), plegada en una fila de 48 px con «Ver los totales».
- El diálogo «Agregar unidades» ocupa la pantalla entera, con su lista desplazable y su botón fijo al pie.
- El panel «Compra y costo» apila cada monto en una fila: el nombre al inicio y el valor al final.
- La página no tiene desplazamiento horizontal.

**Accesibilidad:**
- Cada monto editable tiene su `aria-label` («Monto de la línea 3, VIN MD2A11CY6LWA12345»), y su error queda enlazado con `aria-describedby`.
- El efecto del motivo es texto enlazado al campo con `aria-describedby`.
- Lo repartido frente al total y el resultado de «Repartir en partes iguales» se anuncian en una región `aria-live="polite"`.
- Después de agregar unidades, el foco va al primer monto agregado.
- Los diálogos atrapan el foco y lo devuelven al botón que los abrió.
- Las insignias («Vigente», «Anulada», «Devuelta al proveedor», «Anulada por notas») son texto, no solo color.
- axe no encuentra violaciones en las tres anchuras.

### 6.8 Stores y fachadas

| Pieza | Contenido |
|---|---|
| `procurement/domain/model` | `credit-note.entity.ts` (`CreditNote`, `CreditNoteLine`), `credit-note-reason.ts` (los cuatro códigos, su efecto y si fijan el monto), `credit-note-split.ts` (`splitEqually(totalMinor, count): MinorUnits[]`), `commands/register-credit-note.command.ts`, `update-credit-note.command.ts`, `annul-credit-note.command.ts`. `purchase-invoice.entity.ts` (fase 2) suma `creditedTotal`, `totalAfterCreditNotes`, `creditNotes` y, por línea, `creditedAmount` y `value` |
| `procurement/infrastructure/api` | `credit-notes-api-endpoint.ts`, `credit-notes-response.ts`, `credit-note-rules-response.ts`, `register-credit-note.request.ts`, `update-credit-note.request.ts`, `annul-credit-note.request.ts`; `procurement-api.ts` suma sus miembros |
| `procurement/infrastructure/assemblers` | `credit-note-assembler.ts`; `purchase-invoice-assembler.ts` suma los campos nuevos |
| `procurement/application` | `credit-notes.store.ts`: `notePage` (`PagedList`), `note`, `rules`, `invoiceOptions`, `invoiceLines`, `loading`, `saving`, `exporting`, `error`, `notice`, y los métodos `register()` (devuelve el id nuevo o `null`), `replaceFile()` y `annul()` (devuelven `Promise<boolean>`), `loadRules()`, `loadInvoiceLines(invoiceId)` y `exportCreditNotes()`. Elige el texto del error con un `switch` por `ApiError.code` |
| `procurement/presentation` | Vistas `credit-notes-view`, `credit-note-form-view` y `credit-note-profile-view`; componentes `credit-note-lines` (edición y lectura), `add-invoice-lines-dialog`, `replace-credit-note-file-dialog`, `annul-credit-note-dialog`, `purchase-invoice-credit-notes` (el panel de la ficha de la factura), `credit-note-status-tone.ts` |
| `fleet` | `vehicle-purchase-cost.entity.ts`; `vehicles-api-endpoint` suma `getPurchaseCost(id)`; `vehicles.store.ts` suma `purchaseCost`, `purchaseCostLoading`, `purchaseCostError`, `clearPurchaseCostError()` y `loadPurchaseCost(id)`; `vehicle.entity.ts` suma `returnedToSupplier`; `vehicle-allowed-actions.ts` suma `viewPurchaseCost`; el componente `vehicle-purchase-cost-panel` |
| `IamFacade` | `hasPermission`, `hasPermissionIn` |
| Entre contextos | Ninguna fachada nueva. El formulario de la nota lee facturas con el store de su propio contexto; la ficha de la nota y la de la unidad se enlazan por su URL, como en la fase 2 |

### 6.9 Revisión de `DESIGN.md`

**Lo que necesita la revisión 36**, escrita en `DESIGN.md` el 03/11/2026. Las líneas de la nota conservan su tabla a 375 px, a diferencia de las de la factura, porque tienen menos columnas. Donde esta lista y la revisión difieran en lo visual, mandará la revisión.

| # | Qué decide | Por qué |
|---|---|---|
| 1 | La tabla de líneas de la nota, editable y fuera de `mm-data-manager`: sus columnas de valor (valor de factura, notas anteriores, valor actual, valor después), el monto fijo de una «anulación» y la línea que se pasa. Su modo de lectura en la ficha | Una tabla nueva fuera de `mm-data-manager` exige una decisión (frontend-feature §3). Extiende la de la factura (revisión 34) |
| 2 | El diálogo «Agregar unidades»: la factura elegida, sus líneas con casilla y búsqueda por VIN, el aviso del corte y su forma a 375 px | Patrón nuevo; reutiliza la casilla de selección de la revisión 35 |
| 3 | El motivo con su efecto debajo, y el mes de campaña que aparece y se vacía | Un campo que explica una consecuencia que no se deshace |
| 4 | «Repartir en partes iguales» y la barra de totales al pie, con el total en la moneda de la empresa rotulado «(calculado)» | Reutiliza `.mm-foot-bar` (revisión 34) con un dato más |
| 5 | El aviso de las unidades devueltas en la ficha de la nota, con su lista | Reutiliza `.mm-notice--warning` (revisión 34) |
| 6 | El panel «Notas de crédito» de la ficha de la factura y las columnas «Notas» y «Valor» de sus líneas | Cambia una pantalla de la revisión 34 |
| 7 | El panel «Compra y costo» de la ficha de la unidad: los montos, los «(calculado)», la tasa de la factura y el caso sin factura vigente | Un panel de montos que no existe; cambia una pantalla de `fleet` |
| 8 | Las insignias «Devuelta al proveedor» y «Anulada por notas», y la línea de la ficha de la unidad | Tonos y textos nuevos de `mm-status-badge` |

---

## 7. Pruebas

### 7.1 Los criterios de aceptación del plan

El plan resume la fase 4 sin criterios con número (plan §5); cada punto de su contenido tiene aquí su prueba, más los criterios que la fase 2 dejó para esta (su hallazgo H3).

| Criterio (plan §5, fase 4) | Prueba |
|---|---|
| Las notas, con su anulación y su número único | `CreditNotesIntegrationTests.theSameNumberOfTheSupplierAnswersNumberConflict`, `annullingNeedsItsReasonAndTheUpdatedAtItRead` |
| La tasa de cada factura (D-152) | `CreditNotesIntegrationTests.eachLineCopiesTheRateOfItsInvoiceAcrossTwoInvoices` |
| Sin notas de débito (D-23) | `CreditNotesIntegrationTests.aNegativeAmountAnswersValidationError`; `credit_notes_total_amount_ck` y `credit_note_items_amount_ck` |
| El valor nunca baja de cero | `LineValueTests.aCreditAboveTheRemainingValueIsRefused`; `CreditNoteLockTests.twoNotesOnTheSameLineDoNotTakeItBelowZero` |
| Una línea anulada no recibe más créditos | `CreditNotesIntegrationTests.anAnnulledLineReceivesNoCredit` |
| La anulación de una unidad la pide compras a `fleet` | `CreditNotesIntegrationTests.aUnitReturnedToItsSupplierIsAnnulledWithoutTheStockPermission`; `FleetContextFacadeImplTests.aReturnedUnitIsAnnulledInsideTheTransactionOfTheNote` |
| Desde el 01/01/2027, una nota para una sola factura | `SingleInvoiceCutoffTests.theLastDayOf2026AllowsTwoInvoicesAndTheFirstOf2027DoesNot` |
| La ficha muestra el costo sin IGV | `VehiclePurchaseCostIntegrationTests.theCostOfAUnitIsItsValueAfterNotesWithoutTax`; `UnitPurchaseCostCalculatorTests.aCarInDollarsCostsWhatTheExampleSays` |
| Anular una factura con notas da 422 | `PurchaseInvoicesIntegrationTests.anInvoiceWithACurrentNoteAnswersHasDependents` |
| AE, OP y CO reciben los permisos de facturas (D-176) | `InvoicePermissionsMatrixMigrationTests.theInvoicePermissionsFollowTheMatrixOnceCreditNotesExist` |
| El reparto en partes iguales cuadra al céntimo | `credit-note-split.spec` ("gives the leftover cent of 100.00 among three to the first"); e2e ("splits 10 000 among four units") |
| axe no encuentra violaciones | `e2e/accessibility.spec.ts` con las tres pantallas nuevas y los paneles nuevos de las fichas de la factura y de la unidad |
| Salir pregunta antes de descartar | `credit-note-form-view.spec` ("asks before discarding"); e2e ("leaving with lines asks first") |
| `ModularityTests`, RLS y filas que no se borran | `ModularityTests`; `ProcurementCompanyIsolationIntegrationTests`; `ProcurementRowsAreNeverDeletedTests`; `RowLevelSecurityMigrationTests`; `NoDeleteEndpointsTests` |

### 7.2 Rendimiento

`CreditNoteRegistrationStatementsTests` activa `hibernate.generate_statistics` (`@TestPropertySource`), registra notas de 1, 10 y 100 líneas sobre facturas ya registradas, y comprueba que `getPrepareStatementCount() − getEntityInsertCount() − getEntityUpdateCount()` es el mismo número en los tres tamaños. Ese número queda escrito en la prueba. No hace falta otra línea base: el registro de una nota no da de alta unidades, y lo caro de la factura (su §7.2) ya está medido.

### 7.3 Extremo a extremo

**Antes de las pruebas nuevas:** la pantalla «En construcción» de `e2e/screens.ts` (hoy en `/procurement/invoices`, `screens.ts:163-164`, y en `/procurement/credit-notes` desde la fase 2) pasa a `/procurement/supplier-payments`, con su encabezado; se regeneran solo las tres imágenes `under-construction-*`.

**Escenarios de `e2e/credit-notes.spec.ts`**, con los mocks de notas en `e2e/support/mocks/procurement.ts` (la división de la fase 2) y los de la unidad en `mocks/fleet.ts`:
1. La lista filtra por empresa, proveedor, motivo, estado y período, y exporta con los filtros de la pantalla.
2. Registra un descuento de campaña: elige la factura, agrega cuatro unidades y reparte 10 000 en partes iguales, 2 500 cada una; con 100,00 entre tres, 33,34 en la primera. Cuadra y registra.
3. Una línea que se pasa de su valor se marca y no se envía; un `422 CREDITNOTE_EXCEEDS_VALUE` del servidor marca su línea.
4. «Anulación de unidad» fija los montos. La respuesta con una unidad devuelta muestra el aviso de la ficha.
5. «Anulación de la factura» agrega todas las líneas vigentes de la factura.
6. Con fecha desde el corte, el diálogo ofrece solo la factura de la nota y lo dice; antes del corte acepta una segunda del mismo proveedor y moneda.
7. Ficha: anula con motivo; con `CREDITNOTE_OUTDATED`, vuelve a leer; `CREDITNOTE_UNIT_ANNULLED` nombra el VIN.
8. Ficha: «Cambiar archivo» reemplaza el documento; con `CREDITNOTE_OUTDATED`, vuelve a leer; una nota anulada no lo ofrece.
9. Ficha de la factura: el panel de notas, las columnas «Notas» y «Valor», «Registrar nota de crédito» abre el formulario con la factura elegida, y «Anular» no se ofrece con notas vigentes.
10. Ficha de la unidad: «Compra y costo» con `stock.ver_costos`, sin el panel sin él; la insignia «Devuelta al proveedor».
11. Salir con líneas pregunta antes de descartar.
12. Sin `compras.registrar_nota_credito` no aparece «Registrar nota».

`e2e/screens.ts` suma `credit-notes`, `credit-note-form` (con líneas) y `credit-note`, con sus imágenes en las tres anchuras y su pasada de axe. Las e2e corren en el puerto 4300, con el equipo enchufado.

### 7.4 Sondas en vivo

`e2e-live/credit-notes.live.spec.ts`, contra el backend real sobre `erp_mm_db_verify`, con la cuenta del superadministrador de verificación:
1. Registra una factura en dólares de cuatro unidades nuevas y una nota de descuento de campaña sobre las cuatro, repartida en partes iguales; la ficha de la factura muestra la nota, lo que resta a cada línea y el total después de notas.
2. La ficha de una de las unidades muestra su costo en dólares y en soles con la tasa de la factura.
3. El mismo número otra vez responde `409`; un monto que se pasa responde `422 CREDITNOTE_EXCEEDS_VALUE`.
4. Una nota con «anulación de unidad» sobre otra unidad la anula: el Inventario la muestra anulada.
5. Una nota con «anulación de unidad» sobre una unidad con cesión abierta se registra, y la unidad queda «Devuelta al proveedor»; cerrada la cesión, se anula desde su ficha.
6. Anular la factura con notas vigentes responde `422 PURCHASEINVOICE_HAS_DEPENDENTS`.
7. Cambia el archivo de la nota de campaña: la ficha descarga el nuevo y la bitácora tiene su fila con las dos llaves.
8. Anula la nota de campaña: las líneas recuperan su valor y la bitácora tiene su fila; anular la nota de la unidad anulada responde `422 CREDITNOTE_UNIT_ANNULLED`.
9. La exportación descarga un Excel.
10. Después de V(n+2), la matriz de permisos muestra los de facturas en AE, OP y CO, y `compras.registrar_nota_credito` en SA, AE y CO.
11. Ningún pedido de la corrida usó `DELETE`, y el registro del backend no tiene ningún `ERROR`.

---

## 8. Secuencia de commits

Ramas `feature/procurement-credit-notes` en los dos repositorios, desde `develop` con la fase 2 integrada, en worktrees hermanos. Cada commit compila y pasa las pruebas que alcanza, siempre con `ModularityTests`; con tablas nuevas, también las de Row Level Security y de filas que no se borran.

**Documentación**
1. `docs(conventions): add the contract of credit notes` — 6.0. Pasa §4 a las convenciones y corrige su §2.2, que dice que solo «anulación de unidad» anula una línea de factura (D-191).

**Backend**

| # | Commit | Paso | Depende de |
|---|---|---|---|
| B1 | `feat(iam): seed the permission of credit notes` — V(n) y `PermissionCodes.Procurement.REGISTER_CREDIT_NOTES` | 6.1 | Fase 2 integrada |
| B2 | `feat(procurement): create the schema of credit notes and let a credited line come back` — V(n+1) | 6.2 | Fase 2 (su B12) |
| B3 | `feat(procurement): model the credit note with its lines, its reasons and the single-invoice cutoff` | 6.3 | — |
| B4 | `feat(procurement): persist credit notes and translate their races` | 6.3 | B2, B3 |
| B5 | `feat(fleet): annul the units a credit note returns and lock the units of a note being annulled` | 6.4 | Fase 2 (su B10) |
| B6 | `feat(procurement): register a credit note and annul the lines it leaves at zero` | 6.5 | B1, B4, B5 |
| B7 | `feat(procurement): count credit notes as dependents of an invoice and in the value of a unit` | 6.5 | B6 |
| B8 | `test(procurement): fix the statements of a hundred-line credit note` | 6.5 | B6 |
| B9 | `feat(procurement): annul a credit note with its reason, its row in the change log and the guard of the balance` | 6.6 | B6 |
| B10 | `feat(procurement): replace the file of a current credit note` — `PUT /credit-notes/{id}` (D-204) | 6.6 | B6 |
| B11 | `feat(procurement): list, read and export credit notes and tell their cutoff` | 6.7 | B9, B10 |
| B12 | `feat(procurement): show what credit notes take from an invoice and offer registering one` | 6.7 | B7 |
| B13 | `feat(fleet): ask the purchase cost and the returned units of a unit through UnitPurchases` — el puerto y su implementación en `procurement`, en el mismo commit para que compile | 6.8 | B7 |
| B14 | `feat(fleet): serve the purchase cost of a unit and flag a unit returned to its supplier` | 6.8 | B13 |
| B15 | `feat(iam): mark the invoice permissions on the roles of the matrix` — V(n+2); reemplaza la prueba de la fase 2 que fijaba solo al superadministrador | 6.12 | Todo; se integra por orden del responsable |

**Frontend**

| # | Commit | Paso | Depende de |
|---|---|---|---|
| F1 | `test(e2e): move the under-construction screen to supplier payments` | 6.11 (va primero) | Fase 2 (su F2) |
| F2 | `feat(procurement): model credit notes and split an amount equally to the cent` | 6.9 | Documentación 1 |
| F3 | `feat(procurement): add the API and the store of credit notes` | 6.9 | F2 |
| F4 | `feat(iam): place the permission of credit notes on its screen of the matrix` | 6.9 | B1 |
| F5 | `feat(procurement): add the credit note form with its invoices, lines and totals` | 6.10 | F3, B11, revisión 36 de `DESIGN.md` |
| F6 | `feat(procurement): list and export credit notes` | 6.11 | F3, B11 |
| F7 | `feat(procurement): show the credit note record and annul it` | 6.11 | F6, B9 |
| F8 | `feat(procurement): replace the file of a credit note from its record` | 6.11 | F7, B10 |
| F9 | `feat(procurement): show the credit notes of an invoice and register one from it` | 6.11 | F5, B12, fase 2 (su F11) |
| F10 | `feat(fleet): show the purchase cost of a unit and flag a unit returned to its supplier` | 6.11 | B14 |
| F11 | `feat(app-shell): open Logística › Notas de crédito` | 6.11 | F6 |
| F12 | `test(e2e): cover credit notes on three breakpoints with axe` | 6.11 | F11 |
| F13 | `test(live): probe credit notes against the real backend` | 6.13 | Todo |

**Cierre (6.13), documentación**
2. `docs(procurement): close credit notes with their rules and commits` — `contextos/procurement.md` (PRC-R68 a PRC-R99 en ✅) y `contextos/fleet.md` (FLT-R94 a FLT-R99); la decisión 7 del mapa, cerrada por D-203, y convenciones §14.19.
3. `docs(processes): mark P4 and the screen of credit notes as built` — procesos P4, §3.2 y §3.3 (los costos de la ficha de la unidad); plan §5, fase 4 (su puerta ya no espera al contador, plan §3.2); el orden de implementación, punto 6; las sondas.

**Migraciones:** B1, B2 y B15 llevan V(n), V(n+1) y V(n+2), numeradas al integrar en `develop` después de V91 y de lo que las fases 2 y 3 hayan integrado antes (plan §4).

---

## 9. Preguntas de la versión 1, decididas

No queda ninguna. Las diez preguntas de la versión 1 (03/11/2026) quedaron decididas el mismo día, por delegación del responsable: ocho en el diccionario, como D-203 a D-205, y dos por la corrección del plan y del diccionario. Esta versión las aplica, cada una donde corresponde, y las reglas ya no llevan marcas de pregunta.

| # | Pregunta de la versión 1 | Decisión |
|---|---|---|
| P1 | ¿Se cierra la decisión 7 del mapa leyendo el costo de la unidad por el puerto `UnitPurchases`, en `GET /vehicles/{id}/purchase-cost`, en vez de en la capa de lectura? | **D-203.** Sí: `fleet` lee el costo de compras por `UnitPurchases` y lo sirve en `GET /vehicles/{id}/purchase-cost`, solo con `stock.ver_costos`, con el valor de factura, sus notas, el valor del vehículo y el costo sin IGV en su moneda y en soles con la tasa de la factura. Cierra la decisión 7 del mapa; convenciones §14.19 se corrige en el cierre (§4.2.10, FLT-R97) |
| P2 | ¿«Anulación de unidad» deja en cero cada línea que descuenta, y «anulación de la factura», todas las líneas vigentes de sus facturas? | **D-204.** Sí, y un monto parcial con esos motivos se rechaza: `CREDITNOTE_UNIT_RETURN_PARTIAL` y `CREDITNOTE_INVOICE_ANNULMENT_PARTIAL` (PRC-R81) |
| P3 | ¿La unidad devuelta y retenida queda «marcada para revisar» (D-191) como dato calculado, sin columna? | **D-204.** Sí: «Devuelta al proveedor», calculada, que se resuelve por la vía explícita (FLT-R98, FLT-R99) |
| P4 | ¿Se adopta para la nota la excepción de 20 días de la guía (columna 39), que D-193 no incluye? | **D-205.** No: todos los que registran notas tienen también `compras.registrar_factura_antigua`, y una nota de campaña suele llegar semanas después de la entrega (PRC-R70) |
| P5 | ¿Valen los códigos de motivo propuestos, el tope de 100 líneas, el mes de campaña solo con descuento de campaña y el formato peruano del número? | **D-204.** Sí a los cuatro (PRC-R69, PRC-R71, PRC-R78) |
| P6 | ¿Se registra una nota cuyo proveedor o moneda se desactivaron después de sus facturas? | **D-204.** Sí, con la empresa activa (D-84) (§4.2.4) |
| P7 | ¿Se puede cambiar después el archivo de una nota registrada, como el de la factura? | **D-204.** Sí, con un `PUT`, su `updatedAt` y el alcance de registrar, como la factura (D-172, D-178). Esta versión lo especifica entero: §4.2.6, PRC-R73, la acción `edit`, «Cambiar archivo» en la ficha y los commits B10 y F8 |
| P8 | ¿D-193 («si anuló una unidad») impide también anular la nota cuando la unidad la anuló después la vía explícita? | **D-204.** Sí: una nota no se anula si una unidad de sus líneas quedó anulada, por cualquier vía (`CREDITNOTE_UNIT_ANNULLED`, PRC-R85) |
| P9 | ¿D-192 cierra la puerta del contador del plan §3.2? | **Plan §3.2, corregido el 03/11/2026.** El contador solo valida tres tratamientos ya decididos (D-190, D-192 y D-194), en paralelo: nada espera su respuesta. B15, la migración que da los permisos de facturas a AE, OP y CO (D-176), sigue siendo el último commit del backend y se integra por orden del responsable |
| P10 | ¿Se corrige antes de sembrarla la descripción de `compras.registrar_nota_credito` en §5.6, anterior a D-191 y D-193? | **Diccionario §5.6, corregido el 03/11/2026.** Dice «Registrar y anular una nota de crédito y su reparto; la línea que queda en cero se anula y, con «anulación de unidad», también la unidad», y V(n) la siembra así (§3.2) |
