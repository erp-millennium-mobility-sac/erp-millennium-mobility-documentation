# Especificación: pagos a proveedores (fase 5, punto 7)

**Versión 5 · 04/11/2026:** la fecha base de la brecha de días, de cada unidad (R22, que corrige D-199 y §10.5 del diccionario), integrada en `develop` ese día: backend `c43d1bc1`, merge `1bd767f1`; el frontend no cambió. Cada unidad cuenta su brecha desde el primer pago que la cubre, y no desde el primer pago aplicado a su factura: §2.3, §2.8, §4.2.8, §4.2.11, PRC-R123, FLT-R131 y la fila 18 de §10.1. El mismo día el responsable aprobó las propuestas para el diccionario de esta fase y de la revisión, y la documentación `7cc29f4` las aplica en el diccionario v1.39 (§10.3). **Versión 4 · 04/11/2026:** las correcciones de la revisión de punta a punta del plan v4 (03 y 04/11/2026), integradas en `develop` el 04/11/2026: backend `44136ae1`, la rama `feature/review-findings` (`5c77bc8f` a `df6b7276`), y frontend `cc782649`, la rama del mismo nombre (`6386f2fa` a `1ef10642`). Lo que cambió de comportamiento lleva la marca **Como se corrigió (04/11/2026)** con su commit; §5 y §7 citan las pruebas nuevas y las que cambiaron de nombre; §10 resume lo que la revisión encontró en esta fase, cómo se cerró cada hallazgo y lo que sigue abierto. **Versión 3 · 03/11/2026:** el registro de lo construido. La fase se integró en `develop` ese día, después de las fases 2, 3 y 4: backend `601cc177`, con V105 a V107 (la rama, hasta `de8f9913`), y frontend `e0ed0f68` (la rama, hasta `4e079f12`). Esta versión corrige lo que el código hizo distinto de lo especificado y lo que el responsable decidió el 03/11/2026 sobre el contrato, con la marca **Como se construyó**, y nombra la prueba real de cada regla (§5, §7) y el commit de cada fila de §8. **Versión 2 · 03/11/2026** (las doce preguntas de la versión 1, del mismo día, quedan decididas por D-207 y D-210 a D-213, delegadas por el responsable; las reglas de `fleet` pasan a FLT-R130 a FLT-R132) · **Estado:** construida e integrada el 03/11/2026, y corregida por la revisión el 04/11/2026; falta la sonda en vivo, que necesita la contraseña de verificación del responsable (§7.4, §10) · **Punto de partida:** backend `b1264ed5` (hasta V91), frontend `88312e13`, documentación `6ebb900`, con el diccionario v1.38 y sus decisiones D-190 a D-213 del 03/11/2026, las especificaciones de las [facturas de compra](especificacion-facturas-de-compra-2026-11-02.md) (fase 2, versión 2) y de la [habilitación](especificacion-habilitacion-2026-11-02.md) (fase 3, versión 2), y el [plan de implementación](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md) v3, fase 5. Si esta especificación y el diccionario no coinciden, manda el diccionario.

**Fuentes:** diccionario §2.5, §2.6, §4.8, §5.6, §9.1 a §9.7, §10, §13.1 y las decisiones D-27, D-31, D-42, D-84, D-102, D-134, D-146, D-151, D-152, D-161 a D-163, D-169, D-172, D-176, D-178, D-190 a D-199, D-203, D-207 y D-210 a D-213; [convenciones de la API](convenciones-de-la-api-millennium-mobility.md); la [especificación de las cargas de archivos](especificacion-cargas-de-archivos-2026-11-03.md), para los pagos migrados; [contextos/procurement.md](contextos/procurement.md) y [fleet.md](contextos/fleet.md); [procesos y pantallas](procesos-y-pantallas-millennium-mobility.md) P5 y §3.2; las skills `backend-module` y `frontend-feature`; la guía funcional del cliente, columnas 24, 40 a 45 y 47, y §7.2 y §7.3.

**La fase 4.** Esta versión se alinea con la [especificación de las notas de crédito](especificacion-notas-de-credito-2026-11-03.md) del 03/11/2026: usa su puerto de la guarda del saldo, sus lecturas de las notas vigentes, su orden de candados y su ruta del costo de la unidad. §2.9 dice qué toma de ella y qué le pide.

**Cómo leerlo.**
- §1 fija el alcance y los pasos del plan.
- §2 es el modelo del backend: paquetes, el modelo de cuentas por pagar con sus ejemplos, los cinco montos de una unidad, monedas y tasas, el agregado, las transacciones, los candados, los puertos y los pagos migrados.
- §3 es el esquema, migración por migración.
- §4 es el contrato de la API, que el paso 7.0 pasa a las convenciones.
- §5 son las reglas con sus pruebas, en ✅ desde el 03/11/2026, cada una con el nombre real de su prueba y sus commits.
- §6 son las pantallas; §7, las pruebas y lo que corrió al integrar; §8, la secuencia de commits, con el de cada fila; §9, las preguntas de la versión 1 y la decisión que cerró cada una; §10, la revisión de punta a punta del 03 y el 04/11/2026.
- Lo marcado **(propuesta)** es un nombre que ninguna decisión fija.
- Las reglas que en la versión 1 seguían una recomendación ya citan la decisión que las fijó: D-207, D-210, D-211, D-212 o D-213.
- En la columna *Fuente* de las tablas de reglas, §*n* es del diccionario. En el texto, un §*n* sin más es de esta especificación.

---

## 1. Alcance

**Implementa:**
- D-190: lo que se debe de una factura, y el saldo a favor solo por lo aplicado de más;
- D-194: pagos en cualquier moneda, con su equivalente, y la diferencia de cambio aparte;
- D-195: el pago con saldo a favor, que consume el saldo del proveedor por moneda, sin apuntar a una nota;
- D-196: la retención registrada, no calculada;
- D-197: `suppliers.is_dealer`, la imputación obligatoria en el canal de dealers y la unidad saldada;
- D-198: `company_id` en las tres tablas, `request_id`, el número de operación único entre los pagos no anulados, el paso de la orden a la factura, la orden con pagos que no se anula y la anulación del pago;
- D-199: los cinco montos de una unidad, el dinero entregado y la brecha de días;
- D-207: los pagos migrados de la carga inicial, que se escriben con este modelo (§2.10);
- D-210: el día de la tasa y la revisión del pago, la tasa y el día de cada aplicación, la retención que se suma a `amount` y el pago que se aplica completo;
- D-211: las órdenes que reciben pagos y las imputaciones;
- D-212: las reglas del registro, el alcance y quién ve el estado de pago;
- D-213: la diferencia de cambio.

**Aplica:** D-27 y D-31 (indicadores calculados, la inicial que se valida con ventas), D-146 (escala de los montos), D-151, D-152 y D-169 (la tasa del pago), D-161 (la factura con pagos no se anula), D-162 y D-172 (el vínculo de la factura con la orden), D-163 y D-178 (el alcance, que D-212 extiende a los pagos), D-134 (escribir exige leer) y D-203 (el costo de la unidad por el puerto).

**La puerta del plan** (§5, fase 5) pedía tres cosas:
- **A24:** lo cierra D-195, porque el pago con saldo ya no apunta a una nota.
- **El canal de dealers:** lo define D-197; sigue vigente y se marca en el proveedor.
- **La retención:** se registra sin calcular (D-196), así que la respuesta del contador ya no impide construir. Decide si después se agrega el cálculo, sin cambiar tablas.

La fase 4 se construye antes (§2.9). Los permisos de pagos no siguen la siembra en dos tiempos de D-176, que es solo de las facturas: llegan después de las notas, y se siembran como marca la matriz.

**Pasos.** El plan resume la fase 5 sin pasos; esta especificación los numera como el punto 7:

| Paso | Qué | Dónde | Tamaño |
|---|---|---|---|
| 7.0 | El contrato pasa a las convenciones | §4 | S |
| 7.1 | Los tres permisos `pagos.*` | §3.2 | S |
| 7.2 | `suppliers.is_dealer`, en el esquema, la API, la exportación y la pantalla | §3.3, §4.2.9, §6.6 | S |
| 7.3 | El esquema de pagos | §3.4 | M |
| 7.4 | El dominio: el agregado y las políticas puras de §2.2 a §2.4 | §2.5 | M |
| 7.5 | El registro, con la idempotencia, las tasas, las aplicaciones, las imputaciones y el pago con saldo; los puertos de dependientes y de monedas | §2.6, §4.2.4 | L |
| 7.6 | La anulación, y el puerto de la fase 4 | §4.2.5, §2.8 | M |
| 7.7 | El paso de la orden a la factura; la orden con pagos | §2.6, §4.2.10 | M |
| 7.8 | Las consultas: lista, ficha, exportación, saldo, lo que se debe, la posición de la factura y el pago de la unidad en su costo de compra | §4.2.1 a §4.2.3, §4.2.6 a §4.2.8, §4.2.11 | M |
| 7.9 | `fleet`: el estado de pago y la brecha en la ficha de la unidad | §2.8, §4.2.11 | S |
| 7.10 | Frontend, base: dominio, API, store, fachada y permisos | §6.10 | M |
| 7.11 | Frontend, pantallas: lista, formulario, ficha, y los paneles de la factura, el proveedor y la unidad | §6 | L |
| 7.12 | Cierre: contextos, procesos P5, plan y sondas en vivo | §7.4, §8 | S |

**Queda fuera:**
- el cálculo de la retención (D-196);
- la validación de la diferencia por inicial contra el contrato del cliente, que espera a ventas (D-31, §10.5);
- los reportes de toda la flota —brecha de días y diferencia por inicial—, que esperan la decisión 3 del mapa de contextos;
- la cuenta bancaria del pago y los pagos programados, que siguen como candidatos (§10.6);
- reasignar una imputación sin anular el pago: se anula y se copia (D-211);
- el estado de pago en la lista y en los filtros del Inventario;
- escribir los pagos migrados: los escribe la carga inicial con este modelo (D-207, §2.10).

---

## 2. Modelo del backend

### 2.1 Paquetes y clases

Las capas son las de la skill `backend-module`. Lo nuevo va en piezas con nombre, nunca engordando `PurchaseInvoiceCommandServiceImpl` (fase 2) ni `PurchaseOrderCommandServiceImpl`, que ya tiene 377 líneas (hallazgo A28).

**`procurement`** (`ky.millennium.mobility.erp.procurement`). Su `package-info.java` ya declara `fleet::acl` desde la fase 2 y no cambia.

| Capa | Clases nuevas |
|---|---|
| `domain/model/aggregates` | `SupplierPayment` |
| `domain/model/entities` | `PaymentApplication`, `PaymentAllocation` |
| `domain/model/valueobjects` | `PaymentMethod`, `OperationNumber`, `PaymentAmount`, `WithheldAmount`, `PaymentTarget`, `PaymentAnnulmentReason`, `UnitPaymentStatus`. Reutiliza `AppliedExchangeRate` de la fase 2 |
| `domain/model/commands` | `RegisterSupplierPaymentCommand`, `PaymentApplicationInput`, `PaymentAllocationInput`, `AnnulSupplierPaymentCommand` |
| `domain/model/queries` | `GetSupplierPaymentByPublicIdQuery`, `SearchSupplierPaymentsQuery`, `SupplierPaymentSearchFilter`, `GetSupplierBalanceQuery`, `GetSupplierPayablesQuery`, `GetInvoicePaymentPositionQuery` |
| `domain/services` | `PaymentConversion`, `PaymentEquivalents` (desde `5c77bc8f`, §2.4), `AmountOwedPolicy`, `SupplierBalancePolicy`, `UnitCoveragePolicy`, `ExchangeDifferencePolicy`, `OrderPaymentTransferPolicy` (§2.2 a §2.4) |
| `domain/repositories` | `SupplierPaymentRepository` |
| `domain/exceptions` | `SupplierPaymentAnnulledException`, `PaymentAllocationMismatchException`, `PaymentTargetMismatchException`; `NaturalKeyTakenException.NaturalKey` suma `SUPPLIER_PAYMENT_REQUEST` y `SUPPLIER_PAYMENT_OPERATION` |
| `application/commandservices` | `SupplierPaymentCommandService` |
| `application/queryservices` | `SupplierPaymentQueryService`, `SupplierPaymentView`, `PaymentApplicationView`, `PaymentAllocationView`, `SupplierBalanceView`, `SupplierPayablesView`, `InvoicePaymentPositionView` |
| `application/internal/commandservices` | `SupplierPaymentCommandServiceImpl` y sus piezas: `SupplierPaymentRequests` (la idempotencia), `SupplierPaymentReferences` (empresa, proveedor y moneda, con sus candados), `SupplierPaymentRates` (la tasa del pago y la de cada aplicación, comparadas con las vistas), `SupplierPaymentApplications` (destinos, monedas y lo que se debe), `SupplierPaymentAllocations` (D-197), `SupplierPaymentAnnulments` (anulación y bitácora), `OrderPaymentTransfers` (D-198, la llama `PurchaseInvoiceOrderLinks` de la fase 2) |
| `application/internal/queryservices` | `SupplierPaymentQueryServiceImpl`, `SupplierPaymentViewFactory`, `SupplierPaymentAllowedActions`, `PayablePositions` (arma lo que se debe, el saldo y los cinco montos con las políticas puras), `SupplierPaymentSpreadsheetRows` |
| `application/internal/outboundservices/readmodels` | `PayableFactsReadRepository` y sus records `InvoicePayableFacts`, `LineFacts`, `ApplicationFacts`, `AllocationFacts`. Se justifica por el disparador (a) de la skill: las sumas cruzan facturas, líneas, pagos, aplicaciones e imputaciones, que ningún agregado carga juntos. Lo que restan las notas lo toma de `CreditNoteRepository.currentCreditsByInvoiceIds` y `currentCreditsByInvoiceItemIds` (fase 4, su §2.5.4) |
| `application/internal/policies` | `SupplierPaymentAccessPolicy`; `ProcurementReadPermissions` suma `PAYMENTS = pagos.ver` |
| `application/internal/errors` | `SupplierPaymentErrors`; `PurchaseOrderErrors.hasPayments()` |
| `infrastructure/dependents` | `PaymentInvoiceDependentsAdapter`, que implementa `PurchaseInvoiceDependents` (fase 2), junto a `CreditNoteInvoiceDependentsAdapter` de la fase 4 |
| `infrastructure/payments` | `PaymentSupplierBalanceAdapter`, que implementa `SupplierBalanceAfterAnnulment` (fase 4, su §2.5.3) con `PayableFactsReadRepository` y `SupplierBalancePolicy` (§2.8) |
| `application/internal/outboundservices` | `ProcurementDataVersions` suma `supplierPayments()` |
| `application/documents` | `ProcurementDocumentKinds` suma `SUPPLIER_PAYMENTS = "supplier-payments"` |
| `infrastructure/persistence/jpa` | `SupplierPaymentPersistenceEntity` (extiende `RevisedAbstractPersistenceEntity`: las cuatro columnas de registro, D-42 y D-210), `PaymentApplicationPersistenceEntity` y `PaymentAllocationPersistenceEntity` (sin clase base: el cuadro no les da columnas de registro), `SupplierPaymentJpaRepository`, `SupplierPaymentRepositoryImpl`, `SupplierPaymentPersistenceAssembler` |
| `infrastructure/persistence/jdbc` | `JdbcPayableFactsReadRepository`; `JdbcProcurementDataVersions` suma `supplier_payments` y `payment_applications` |
| `infrastructure/currencies` | `SupplierPaymentCurrencyUsageAdapter` |
| `infrastructure/fleet` | `PurchaseInvoiceUnitPurchasesAdapter` (fases 2 y 4) suma `paymentFacts`, y su `purchaseCostOf` suma el pago de la unidad (§2.8) |
| `interfaces/rest` | `SupplierPaymentsController`; `SuppliersController` suma `/balance` y `/payables`; `PurchaseInvoicesController` (fase 2) suma `/payment-position` |
| `interfaces/rest/resources` | `RegisterSupplierPaymentResource`, `PaymentApplicationInputResource`, `PaymentAllocationInputResource`, `AnnulSupplierPaymentResource`, `SupplierPaymentResource`, `PaymentApplicationResource`, `PaymentAllocationResource`, `SupplierBalanceResource`, `SupplierPayablesResource`, `InvoicePaymentPositionResource`; reutiliza `MoneyResource`. `CreateSupplierResource`, `UpdateSupplierResource` y `SupplierResource` suman `isDealer` |
| `interfaces/rest/transform` | `RegisterSupplierPaymentCommandFromResourceAssembler`, `SupplierPaymentResourceFromViewAssembler`, `SupplierPaymentSpreadsheetColumns`; `SupplierSpreadsheetColumns` suma la columna del canal |

**`fleet`.** `package-info.java` no cambia.

| Capa | Clases nuevas o que cambian |
|---|---|
| `interfaces/acl` | `UnitPurchases` (fases 2 y 4) suma `paymentFacts` y el record `UnitPaymentFacts`; `UnitPurchaseCost` (fase 4) suma `payment`, con los records `UnitPaymentPosition` y `UnitPaymentRecord` (§2.8) |
| `application/internal/outboundservices/purchases` | `UnitPurchaseFacts` suma `paymentFacts` |
| La ruta del costo de compra de la fase 4 | `GET /vehicles/{id}/purchase-cost` suma `purchase.payment` (§4.2.11) |
| `domain/services` | `DayGapPolicy`: la brecha de días entre la fecha base y la entrega, o hoy en la zona de la empresa, con `Clock` |
| `application/queryservices` | `VehicleView` suma `payment` (`VehiclePaymentView`), solo en la ficha |
| `application/internal/queryservices` | `VehicleViewFactory` lo arma con `UnitPurchaseFacts` y la lectura del primer paso a Activo de la fase 3 (FLT-R58) |
| `interfaces/rest/resources` | `VehicleResource` suma `payment` |

**`iam`, `shared` y los demás:**

| Contexto | Cambio |
|---|---|
| `iam` | Solo la migración V(n) |
| `shared` | `PermissionCodes.Payments`, con `VIEW`, `REGISTER_PAYMENTS` y `ANNUL_PAYMENTS`, en el mismo commit que su migración (`PermissionCatalogConsistencyTests`) |
| `location` | Nada nuevo: reutiliza `fetchEffectiveRate` de la fase 2 y el puerto `CurrencyUsage` |
| `organization` | Nada nuevo: reutiliza `fetchPurchasingCompanyForShare` y `fetchPurchasingCompaniesByIds` de la fase 2 |

**Como se construyó** (backend `25eb8f69` a `de8f9913`). Las clases son las de arriba, con estas diferencias:
- **Excepciones:** no hay `PaymentTargetMismatchException`: el servicio responde `SUPPLIERPAYMENT_TARGET_MISMATCH` al revisar los destinos. Se sumó `PaymentAppliedMismatchException`, la del pago que no se aplica completo (D-210).
- **Lecturas:** `PayableFactsReadRepository` suma los records `ConsumptionFacts`, lo pagado con saldo, y `NoteFacts`, las notas de las facturas del proveedor. La posición de una factura la arma una pieza propia, `InvoicePaymentPositions` (`56f992c8`), y `PayablePositions` reparte con `positionsOf` y `coveragesOf`.
- **Exportación:** las filas las arma `SupplierPaymentViewFactory.toExportRows`; no hay `SupplierPaymentSpreadsheetRows`.
- **Otras piezas:** `SupplierPaymentScopes`, la empresa y las marcas que un pago alcanza, para su anulación y sus acciones; `SupplierPaymentErrorContexts`, las formas de `context` (§4.3); y `PaymentMoneyResource`, el monto de un pedido.
- **`shared`:** la orden de la que pasó una aplicación se lee de la bitácora con `ChangeLogHistoryLookup.latestOldValues(entity, fieldName, ids)`, de `shared/changelog` (`ad7f9983`), la lectura que el dueño de una entidad hace de su propia historia, sin el alcance de `bitacora.ver`. Antes de la revisión, `procurement` nombraba la tabla `change_logs` en una consulta de JDBC. El nombre de la entidad y del campo viven en `PaymentApplication.ENTITY_NAME` e `INVOICE_FIELD`.

### 2.2 El modelo de cuentas por pagar (D-190)

**Definiciones.** Todas se calculan y ninguna se guarda (§10.4, regla «ningún valor derivado se guarda» de la skill):

| Concepto | Cómo sale | Moneda |
|---|---|---|
| Notas de una factura | La suma de las líneas de las notas no anuladas que descuentan sus líneas (§9.5), por `CreditNoteRepository.currentCreditsByInvoiceIds` (fase 4) | La de la factura |
| Aplicado a una factura | La suma de lo cubierto por sus aplicaciones de pagos no anulados. Lo cubierto por una aplicación es `amount + withheld_amount` (D-210) | La de la factura |
| **Lo que se debe** | Total del comprobante − notas − aplicado (D-190) | La de la factura |
| Aplicado de más | Lo que se debe, si es negativo, con el signo cambiado; si no, cero | La de la factura |
| Consumido | La suma de los montos de los pagos no anulados con método `saldo_a_favor`, en esa moneda | Esa moneda |
| **Saldo del proveedor, por moneda** | La suma de lo aplicado de más en sus facturas vigentes de esa moneda, menos lo consumido en esa moneda (D-190, D-195) | Esa moneda |

**Las reglas que lo sostienen:**
- Una aplicación no supera lo que se debe de su factura al aplicarse (D-190). Por eso un pago de dinero nunca crea saldo: el saldo solo nace cuando una nota llega después de pagar.
- Un pago de dinero tampoco consume saldo. Solo lo consumen los pagos con saldo, y solo lo bajan la anulación de un pago, que quita lo aplicado de más que creó, y la anulación de una nota (fase 4).
- Por eso, de todas las escrituras, solo tres miran el saldo y bloquean al proveedor en modo exclusivo: el pago con saldo, la anulación de un pago y la anulación de una nota (§2.7).
- La factura anulada no entra: nunca tiene pagos, porque la anulación lo impide (§9.3, D-161).

**Ejemplo 1 · La nota llega antes del pago.** Factura F001-200 de Crosland, en soles, con dos motos de 5 900,00:

| Fecha | Hecho | Total | Notas | Aplicado | Lo que se debe | Saldo del proveedor |
|---|---|---|---|---|---|---|
| 01/09 | Se registra la factura | 11 800,00 | 0,00 | 0,00 | 11 800,00 | 0,00 |
| 10/09 | Nota NC001-0050 de 400,00 a la moto 1 | 11 800,00 | 400,00 | 0,00 | 11 400,00 | 0,00 |
| 15/09 | Pago OP-1001 de 11 400,00, sin imputar | 11 800,00 | 400,00 | 11 400,00 | 0,00 | 0,00 |

Si el 15/09 se hubiera intentado aplicar 11 800,00, el registro responde `422 SUPPLIERPAYMENT_EXCEEDS_OWED`, porque lo que se debe es 11 400,00. La nota ya bajó lo que se debía: contarla también como saldo la descontaría dos veces, que es lo que corrigió D-190. Los cinco montos (§2.3): la moto 1 vale 5 500,00 y la 2, 5 900,00; los 11 400,00 se reparten en proporción a lo que le falta a cada una, así que cubren 5 500,00 y 5 900,00, la diferencia es 0,00 y las dos quedan **Pagada**.

**Ejemplo 2 · La nota llega después del pago.** La misma factura:

| Fecha | Hecho | Lo que se debe de F001-200 | Lo que se debe de F001-210 | Saldo del proveedor (PEN) |
|---|---|---|---|---|
| 01/09 | Se registra F001-200 | 11 800,00 | — | 0,00 |
| 15/09 | Pago OP-1001 de 11 800,00 a F001-200 | 0,00 | — | 0,00 |
| 30/09 | Nota NC001-0050 de 400,00 a la moto 1, el bono de campaña | −400,00: pagado de más | — | 400,00 |
| 05/10 | Se registra F001-210, una moto de 5 900,00 | −400,00 | 5 900,00 | 400,00 |
| 06/10 | Pago con saldo de 400,00 a F001-210 | −400,00 | 5 500,00 | 0,00 |
| 07/10 | Se intenta anular OP-1001 | — | — | Sin él, 0,00 − 400,00 = −400,00: `422 SUPPLIERPAYMENT_BALANCE_NEGATIVE` |
| 07/10 | Se intenta anular NC001-0050 (fase 4) | — | — | También dejaría −400,00: la nota no se anula (D-193) |

Para corregir OP-1001 primero se anula el pago con saldo, y el saldo vuelve a 400,00; después OP-1001, y el saldo queda en 0,00. En F001-200, las motos valen 5 500,00 y 5 900,00 y quedan cubiertas por su valor; los 400,00 que sobran no son de ninguna unidad: son el saldo. En F001-210 la moto queda **Parcial**, con 400,00 cubiertos con saldo y 0,00 de dinero entregado.

### 2.3 Los cinco montos de una unidad (D-199)

Para cada línea vigente de una factura vigente, `UnitCoveragePolicy` recibe sus hechos y responde, sin escribir nada:

| # | Monto | Cómo sale |
|---|---|---|
| 1 | Valor de factura | `invoiced_amount` de la línea. Es el único que se guarda, y no cambia (§9.4) |
| 2 | Notas | La suma de las líneas de notas no anuladas sobre esa línea, por `currentCreditsByInvoiceItemIds` (fase 4) |
| 3 | Valor del vehículo | 1 − 2. Es el costo (§9.6) |
| 4 | Cubierto | Lo imputado a la unidad, más su parte de lo aplicado a la factura sin imputar |
| 5 | Diferencia | 3 − 4. En el canal normal, lo pendiente de pago; en una unidad saldada del canal de dealers, la inicial del cliente (D-197) |

**Cómo se reparte lo cubierto:**
1. **Lo imputado:** cada imputación de una aplicación de un pago no anulado suma su monto a su unidad.
2. **Lo que falta a cada unidad:** su valor menos lo imputado, nunca menos de cero.
3. **El fondo sin imputar:** lo cubierto por las aplicaciones a la factura que no imputan unidades.
4. **El reparto del fondo:** entre las unidades vigentes de la factura que no están saldadas, en proporción a lo que le falta a cada una. Cuando ninguna unidad tiene imputaciones, lo que falta es su valor, y el reparto es el «en proporción a su valor» de D-199 y §10.3. Con imputaciones parciales, repartir por el valor cubriría dos veces las unidades ya pagadas (ejemplo 4, D-211).
5. **El redondeo:** mitad hacia arriba, a los decimales de la moneda de la factura. Los céntimos que sobran, o que faltan, se mueven de a uno desde la primera unidad con falta, la de la línea de menor `id`, sin pasar lo que le falta ni bajar de cero; una unidad con parte cero no recibe fila (§10.3; **Como se corrigió**, abajo).
6. **Lo que no cabe:** si el fondo supera lo que falta, cada unidad recibe lo que le falta, y el resto no es de ninguna unidad: es lo aplicado de más, el saldo del proveedor (§2.2).

**Como se construyó** (`3a1596b4`). Los céntimos del redondeo van a la primera unidad que recibe parte del reparto, la de menor `id` entre las que les falta algo, y no a la primera de la factura, que puede no recibir nada. El frontend reparte igual, con la mitad hacia arriba y los céntimos en la primera unidad con peso (`splitInProportion`, frontend `94b411de`).

**Como se corrigió (04/11/2026)** (backend `46f207b1`, frontend `f8efb41b`; decisión C2 de la revisión). Dar todos los céntimos a la primera unidad podía dejarla bajo cero o por encima de lo que le falta: 0,25 sobre cincuenta unidades que deben 100,00 redondea a 0,01 cada una, 0,50 en total, y la primera quedaba en −0,24. Ahora:
- **Los céntimos se mueven de a uno,** desde la primera unidad con falta, en el orden de las líneas. Una unidad toma céntimos mientras su parte sea menor que lo que le falta, y los devuelve mientras su parte sea mayor que cero, antes de tocar la siguiente.
- **Si la primera puede con todos,** se los lleva todos, como antes.
- **Una unidad cuya parte queda en cero** no recibe nada ni fila de pago. En el ejemplo, las 25 primeras quedan **Sin pago** y las 25 siguientes reciben 0,01.
- **El frontend reparte igual** (`supplier-payment-amounts.ts`), con las mismas cifras en sus pruebas.

Lo prueban `UnitCoveragePolicyTests.theCentsLeftNeverTakeAUnitBelowZeroNorAboveWhatItLacks` y `supplier-payment-amounts.spec` ("C2: 0,25 over 50 units lacking 100,00 each: the first 25 give their cent back, never below zero", "adds a missing cent to the first unit that still lacks one, never past what it lacks").

**El estado de pago** (`UnitPaymentStatus`, códigos **(propuesta)**, estables como los de D-171):

| Código | Cuándo | Texto |
|---|---|---|
| `SALDADA` | Una imputación de un pago no anulado declaró la unidad saldada (D-197) | Saldada |
| `PAGADA` | Lo cubierto alcanza su valor (D-199) | Pagada |
| `PARCIAL` | Cubierto mayor que cero y menor que su valor | Pago parcial |
| `SIN_PAGO` | Nada cubierto | Sin pago |

Una unidad sin línea vigente no tiene estado: la API responde `null`.

**Los otros dos indicadores:**
- **Dinero entregado** (D-199): lo cubierto, menos su parte de la retención, menos su parte de lo pagado con saldo. La parte de cada imputación sale en proporción a lo que la aplicación retuvo o pagó con saldo; la del fondo, en proporción al fondo. Se redondea igual.
- **Brecha de días** (D-199, D-27; guía, columnas 24 y 43): desde la fecha base hasta la primera entrega o, si no se entregó, hasta hoy en la zona de la empresa. La fecha base es de cada unidad: la del primer pago no anulado que cubre parte de ella —por su imputación, o por su parte de lo que su factura recibió sin imputar, también un pago que pasó de la orden— y, mientras ninguno la cubre, la de la factura. `procurement` da la fecha base; `fleet`, que sabe cuándo se entregó, cuenta los días (§2.8). Puede ser negativa: la unidad se entregó antes de pagarse. Nunca se guarda.

**Como se corrigió (04/11/2026): la fecha base de cada unidad** (backend `c43d1bc1`, merge `1bd767f1`; R22, aprobada por el responsable ese día con D-199 y §10.5 del diccionario v1.39). Antes, todas las unidades de una factura tomaban una sola fecha base, la del primer pago aplicado a la factura. La guía (columnas 24 y 43) fecha la brecha de una unidad desde el bloque de pago que la incluye: de 50 motos pagadas en dos bloques de 25, el segundo bloque contaba desde el pago del primero. Ahora `UnitCoveragePolicy` da a cada unidad su `UnitCoverage.baseDate`:
- **el primer pago que la cubre,** en el orden de la fecha del pago y el `id` de la aplicación: uno que le imputa algo, o uno que va al fondo sin imputar mientras la unidad recibe parte del fondo, aunque la parte redondeada de ese pago sea cero;
- **la fecha de la factura,** mientras ningún pago la cubre.

`PurchaseInvoiceUnitPurchasesAdapter` la pasa a `fleet` en `UnitPaymentFacts.baseDate`, que da el `payment.baseDate` y el `dayGap` de la ficha de la unidad, y en `UnitPaymentPosition.baseDate`, el de su costo de compra (§2.8, §4.2.11). La posición de la factura conserva su `baseDate`: el primer pago aplicado a la factura, `InvoiceCoverage.baseDate` (§4.2.8). La forma de la API no cambia; cambian el valor de cada unidad y las descripciones de OpenAPI. El frontend lee el valor de cada unidad del backend, así que no cambió. Lo prueban `UnitCoveragePolicyTests.eachUnitCountsItsGapFromTheFirstPaymentThatCoversIt` y `aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf` (ejemplo 4, que ahora comprueba la fecha de cada bloque), y `VehiclesIntegrationTests.eachUnitOfAnInvoicePaidInTwoBlocksCountsItsGapFromItsOwnPayment` (PRC-R123, FLT-R131).

**Ejemplo 3 · Sin imputar, en proporción a su valor.** F003-400, dos autos de 20 000,00 y 30 000,00. Pago de 48 000,00 sin imputar: cubren 19 200,00 y 28 800,00, con diferencias de 800,00 y 1 200,00, las dos **Pago parcial**. La guía (columna 45) reparte la diferencia en partes iguales, 1 000,00 cada una; manda D-199, que la reparte en proporción.

**Ejemplo 4 · Pagar 25 de 50 motos** (§10.3). Factura de 50 motos de 5 900,00, 295 000,00 en total:
- El pago 1, de 147 500,00, imputa 5 900,00 a cada una de 25 motos: esas quedan **Pagada**.
- El pago 2, de 100 000,00, no imputa. El fondo se reparte entre las 25 restantes, porque a las otras ya no les falta nada: 4 000,00 cada una, **Pago parcial**.
- Repartirlo por el valor, entre las 50, daría 2 000,00 a cada una, y las 25 ya pagadas quedarían cubiertas por 7 900,00. Por eso el paso 4 reparte por lo que falta (D-211).
- Cada bloque cuenta su brecha desde su propio pago: las 25 primeras desde el pago 1, y las otras 25 desde el pago 2 (D-199; **Como se corrigió**, arriba).

**Ejemplo 5 · Canal de dealers** (D-197). Autos del Norte tiene `is_dealer`. Factura F002-300, tres autos de 50 000,00. El cliente del auto A pagó su inicial de 8 000,00 al concesionario:

| Unidad | Valor de factura | Notas | Valor | Cubierto | Diferencia | Estado |
|---|---|---|---|---|---|---|
| A | 50 000,00 | 0,00 | 50 000,00 | 42 000,00, imputados con «salda la unidad» | 8 000,00, la inicial del cliente | Saldada |
| B | 50 000,00 | 0,00 | 50 000,00 | 50 000,00 | 0,00 | Pagada |
| C | 50 000,00 | 0,00 | 50 000,00 | 0,00 | 50 000,00, pendiente de pago | Sin pago |

El pago OP-2001, de 92 000,00, imputó 42 000,00 a A, saldándola, y 50 000,00 a B. Lo que se debe de la factura es 58 000,00: de eso, 8 000,00 son la inicial de A, que Millennium no paga. La ficha de la factura lo muestra aparte (§6.5). Un pago que imputa algo a A responde `422 SUPPLIERPAYMENT_UNIT_SETTLED`.

### 2.4 Monedas, tasas y diferencia de cambio (D-194)

**La tasa del pago** (D-151, D-152, D-169):
- En la moneda funcional de la empresa: 1, y la fecha del pago como día de la tasa.
- En otra moneda: la del país de la empresa, del tipo que el país usa para sus compras (`countries.purchase_exchange_rate_type`, venta en Perú, D-181), la más reciente de la fecha del pago o anterior, por `LocationContextFacade.fetchEffectiveRate` de la fase 2.
- La pantalla manda la tasa que mostró, y si cambió, `422` (hallazgo A19).

**La tasa de cada aplicación** (D-210). Un pago en soles lleva tasa 1 (D-169) y con ella no se convierten dólares, así que cada aplicación guarda su propia tasa y su día: la de venta del día del pago entre la moneda del pago y la del documento, o 1 si son iguales.

| Moneda del pago | Moneda del documento | Tasa de la aplicación | Equivalente en la moneda del pago |
|---|---|---|---|
| Funcional | Funcional | 1, del día del pago | El mismo monto |
| Extranjera | La misma extranjera | 1, del día del pago | El mismo monto |
| Extranjera | Funcional | La del pago | Monto ÷ tasa |
| Funcional | Extranjera | La vigente de esa moneda en la fecha del pago, como la del pago | Monto × tasa |
| Extranjera | Otra extranjera | — | `422 SUPPLIERPAYMENT_CURRENCY_PAIR` (D-194) |

El equivalente se redondea mitad hacia arriba a los decimales de la moneda del pago (`PaymentConversion`), y convierte solo `amount`: la retención no sale del pago (D-210). Si la suma de los equivalentes queda a unos céntimos del monto, por debajo o por encima, las aplicaciones convertidas absorben el residuo, de la última a la primera, cada una dentro de los equivalentes que al convertirse de vuelta dan su monto (`PaymentEquivalents`); se guarda el absorbido (**Como se corrigió**, abajo).

**La diferencia de cambio** (`ExchangeDifferencePolicy`, D-213). Por aplicación de dinero a una factura en moneda extranjera, en la moneda funcional:

> diferencia = redondeo(cubierto × tasa de la aplicación) − redondeo(cubierto × tasa de la factura)

- La tasa del día es la de la aplicación cuando el pago está en la moneda funcional. Cuando el pago está en la misma moneda extranjera que la factura, la aplicación tiene tasa 1 (D-210), y la tasa del día es la del pago.
- Positiva es pérdida: la empresa pagó en su moneda más de lo que registró la factura.
- Es cero en una factura en la moneda funcional, y en una aplicación con saldo, que no compra moneda (D-213).
- Una aplicación a una orden no tiene diferencia hasta pasar a su factura.
- Se informa en la ficha y en la exportación del pago, y **nunca toca el costo**: el valor del vehículo sigue en la moneda de la factura y a su tasa (D-194, §9.6). El contador la valida junto con la retención (D-213).

**Como se construyó** (`3a1596b4`, `a86065d6`). `exchangeDifference` es `null` en una aplicación a una orden y en una a un documento en la moneda funcional, también uno pagado en otra moneda, cuyo equivalente se divide; y es cero con saldo a favor. Un pago en la moneda funcional con una tasa que no es 1 de su fecha responde `422 SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED`, como la factura (decisión del responsable del 03/11/2026).

**Como se corrigió (04/11/2026): el equivalente absorbe el redondeo** (backend `5c77bc8f`, frontend `20cebe6f`; decisión C1 de la revisión, sobre D-210). Con el equivalente redondeado hacia adelante, un pago en soles de una factura en dólares solo cuadraba con cerca de un cuarto de los montos posibles: a 3,760000, USD 2 659,57 son PEN 9 999,98, y ningún monto en dólares daba PEN 10 000,00, el del voucher. Ahora:
- **Cada equivalente empieza** como su redondeo mitad hacia arriba (`PaymentConversion`).
- **Si la suma no es el monto del pago,** las aplicaciones convertidas absorben la diferencia, de la última a la primera en el orden del pedido. Cada una mueve su equivalente hacia el residuo, hasta donde lo admite, y el residuo baja en lo que movió.
- **Lo que una aplicación admite:** su redondeo hacia adelante, o un equivalente cuya conversión de vuelta, redondeada mitad hacia arriba a los decimales del documento, da su monto: dividido por la tasa si el documento se convierte multiplicando, multiplicado si se convierte dividiendo. En la misma moneda solo admite su monto; una aplicación dividida puede no admitir otro que su redondeo.
- **Si queda residuo,** el pago no se aplica completo: `422 SUPPLIERPAYMENT_APPLIED_MISMATCH`, con `context.applied` igual a la suma de los equivalentes hacia adelante, sin absorber (`PaymentEquivalents.Absorption.forwardSum`).
- **Se guarda el equivalente absorbido,** y `PaymentApplication` rechaza uno que no sea admisible para su monto. La API nunca recibe un equivalente del cliente: lo calcula el servicio con la tasa de la aplicación.
- **La pantalla calcula igual** (`absorbedEquivalents` y `admissibleEquivalents`, en `supplier-payment-amounts.ts`). Muestra el absorbido solo cuando absorber cuadra el pago, el único caso que el backend guarda; mientras el pago se llena, cada fila conserva su redondeo hacia adelante.

Con la absorción, la diferencia de cambio de una aplicación de un pago en la moneda funcional lee el equivalente guardado:

> diferencia = redondeo(cubierto × tasa de la aplicación) + (equivalente guardado − redondeo(monto × tasa de la aplicación)) − redondeo(cubierto × tasa de la factura)

Sin absorción, el término del medio es cero y vuelve la fórmula de arriba. Un pago en la misma moneda extranjera que la factura no absorbe, y sigue con la tasa del pago.

Los vectores V1 a V6 son los mismos en los dos repositorios. V1, de punta a punta: una factura de USD 2 659,57 a 3,742000, pagada en soles a 3,760000; PEN 10 000,01 se rechaza con `context.applied` 9 999,98, y PEN 10 000,00 se registra, guarda el equivalente 10 000,00 y una diferencia de cambio de 47,89, en el pago y en la posición de la factura. Lo prueban `PaymentEquivalentsTests` (V1 a V6 y `anEquivalentIsAdmissibleWhenItConvertsBackToTheAmount`), `PaymentApplicationTests.anAbsorbedEquivalentConvertsBackToTheAmount`, `ExchangeDifferencePolicyTests.theDifferenceReadsTheEquivalentThatAbsorbedTheResidue` y `SupplierPaymentsIntegrationTests.aPaymentInSolesAbsorbsTheResidueOfTheRounding`; en el frontend, `supplier-payment-amounts.spec` (`absorbedEquivalents (contract C1)`, V1 a V6), `payment-applications-table.spec` («with a dollar invoice paid in soles at 3,760000 (contract C1)») y la e2e "PEN 10,000.00 squares with USD 2,659.57 at 3.760000: the equivalent absorbs the rounding, and the payment is registered (C1)", que también comprueba la suma hacia adelante de un rechazo (`af8a3f16`). Ninguna migración: la columna ya guardaba el equivalente.

**Ejemplo 6.** Factura F002-981 de Derco, USD 13 100,00, tasa 3,742000 del 31/10/2026, que en soles es 49 020,20. Pago del 03/11/2026 en soles, con la venta del día a 3,760000:
- La aplicación guarda USD 13 100,00, la tasa 3,760000 del 03/11/2026 y el equivalente PEN 49 256,00.
- La diferencia de cambio es 49 256,00 − 49 020,20 = PEN 235,80, una pérdida.
- El costo de la unidad sigue siendo USD 13 100,00, que son PEN 49 020,20 a la tasa de la factura.

### 2.5 El agregado y sus reglas

**`SupplierPayment`** es la raíz; sus aplicaciones (`PaymentApplication`) y las imputaciones de cada una (`PaymentAllocation`) son entidades del agregado. Se guardan con él, con `cascade` `PERSIST` y `MERGE`, nunca `ALL` ni `orphanRemoval`. Tiene dos constructores, como pide la skill.

| Miembro | Regla que protege |
|---|---|
| Constructor de registro | <ul><li>De 1 a 100 aplicaciones, y a lo sumo 500 imputaciones en total **(propuesta)**.</li><li>Ningún destino dos veces.</li><li>En la moneda funcional, tasa 1 y día de la tasa igual a la fecha del pago (D-169); el día de la tasa nunca es posterior a la fecha del pago.</li><li>Con `SALDO_A_FAVOR`: solo facturas, sin número de operación ni retención (D-195, D-212). Con `TRANSFERENCIA` o `CHEQUE`, con número de operación (D-212).</li><li>La suma de los equivalentes es exactamente el monto: un pago se aplica completo (D-194, D-210).</li></ul> Los montos llegan como `PaymentAmount` y `WithheldAmount`, que ya validaron su escala contra los decimales de su moneda (D-146) |
| `annul(PaymentAnnulmentReason, Instant, Long annulledBy)` | Una sola vez (`SupplierPaymentAnnulledException`) |
| `isAsOpenedAt(Instant)` | El candado de `updatedAt`, comparado al milisegundo (convenciones §3), como la factura (D-178, D-198) |
| `appliedInPaymentCurrency()` | La suma de los equivalentes, que es el monto (D-210) |
| `sameContentAs(Content, Map<Long, Long> fromOrders)` | La idempotencia (§4.6): la empresa, el proveedor, la fecha, la moneda, el monto, el método, el número de operación, las notas y los destinos, cada uno con su monto, su retención, su comprobante y sus imputaciones —unidad, monto y si la salda—. Un destino coincide con el actual de la aplicación o con la orden de la que pasó, con sus partes sumadas. **Como se corrigió (04/11/2026)** (`bb355c8e`): antes comparaba solo los destinos con sus montos (§4.6) |

**`PaymentApplication`**:
- **Sus datos:** `target` (factura u orden), `amount`, `withheldAmount`, `withholdingCertificate`, `exchangeRate` (un `AppliedExchangeRate`, D-210) y `paymentAmount`, que calcula `PaymentConversion` al construirla y que, desde `5c77bc8f`, puede llegar ya absorbido: el constructor solo acepta un equivalente admisible para su monto (`PaymentEquivalents.isAdmissible`, §2.4).
- **Sus reglas:**
  - el monto es mayor que cero, y la retención, mayor o igual que cero;
  - una retención lleva su comprobante (D-196, D-210);
  - las imputaciones solo van en una aplicación a una factura;
  - ninguna unidad dos veces;
  - si imputa, la suma de las imputaciones es exactamente lo cubierto, `amount + withheldAmount` (`PaymentAllocationMismatchException`, D-210, D-211).
- **`giveToInvoice(Long invoiceId, BigDecimal covered)`:** solo en una aplicación a una orden. Si lo que pasa es todo, la aplicación se vuelve de la factura. Si no, devuelve la parte nueva y se queda con el resto. La retención pasa en proporción, redondeada, y el equivalente de lo que queda es el original menos el de la parte que pasa, para que la suma siga exacta (§2.6, D-198).

**`PaymentAllocation`**: `vehicleId`, `amount` mayor que cero y `settlesUnit`. No cambia nunca: la tabla solo recibe inserciones.

**Objetos de valor:**

| Clase | Qué valida |
|---|---|
| `PaymentMethod` | `TRANSFERENCIA("transferencia")`, `CHEQUE("cheque")`, `EFECTIVO("efectivo")`, `SALDO_A_FAVOR("saldo_a_favor")`, guardados en minúscula como `document_type` de la factura; se aceptan en cualquier mayúscula. `movesMoney()` es falso solo en el saldo |
| `OperationNumber` | `normalize(String)`, con `TextIdentifier.normalize` como el documento del proveedor: recorta, pasa a mayúsculas y quita espacios, guiones y puntos, también los Unicode (convenciones §5.3). De 1 a 100 caracteres después de normalizar. `OP-99213` y `op 99213` son el mismo número |
| `PaymentAmount` | Mayor que cero, a lo sumo `999999999999.99`, con a lo sumo los decimales de su moneda |
| `WithheldAmount` | Lo mismo, desde cero |
| `PaymentTarget` | Factura u orden, con su identificador; nunca los dos |
| `PaymentAnnulmentReason` | `ERROR_REGISTRO`, `DEVUELTO` (D-212) |
| `UnitPaymentStatus` | Los cuatro códigos de §2.3 |

**Políticas puras** (`domain/services`), sin Spring y probadas sin contexto:
- `PaymentConversion`: el equivalente de §2.4.
- `PaymentEquivalents`: la absorción del redondeo de §2.4, con lo que admite cada aplicación (`5c77bc8f`).
- `AmountOwedPolicy`: lo que se debe y lo aplicado de más de una factura.
- `SupplierBalancePolicy`: el saldo por moneda, con una exclusión opcional, un pago o una nota, para responder «¿y sin él?» (§2.6, §2.8).
- `UnitCoveragePolicy`: §2.3, también el estado y la fecha base.
- `ExchangeDifferencePolicy`: §2.4.
- `OrderPaymentTransferPolicy`: qué parte de cada aplicación de la orden pasa a la factura, de la más antigua a la más nueva por fecha del pago y después por `id`, hasta lo que se debe (D-198).
- Reutiliza `InvoiceDatePolicy` de la fase 2 para «hoy» en la zona de la empresa (D-212).

**`SupplierPaymentAccessPolicy`** (`application/internal/policies`), sobre `AccessScope` (D-212):
- `sees(scope, companyId)`: `pagos.ver` actúa en la empresa. Un pago se ve por empresa, como una factura (D-163).
- `registers(scope, companyId, brandIds, general)`: `pagos.registrar_pago` actúa en toda la empresa (`actsOnWholeCompany`) y en la marca comercial de cada línea vigente de las facturas que paga (`actsOnWholeCompanyBrand`). Una orden de una plantilla general, o un pago con saldo, que es del proveedor y de todas las marcas, exigen toda la empresa y todas sus marcas (`actsOnWholeCompanyEveryBrand`, D-134).
- `annuls(...)`: `pagos.anular_pago` con el mismo alcance que hizo falta para registrarlo (D-212, como D-178 en la factura).
- Escribir exige leer (D-134): `pagos.ver` donde actúa, y `compras.ver` sobre las facturas y órdenes que paga. Lo que no lee responde `404`.

### 2.6 Servicios de aplicación y transacciones

`SupplierPaymentCommandServiceImpl` (`@Service @Transactional`) queda envuelto por `DeadlockRetry` por su nombre y su paquete. Todo lo que hace va dentro de su transacción.

| Operación | Transacción | Qué escribe |
|---|---|---|
| `handle(RegisterSupplierPaymentCommand)` | Una, `REQUIRED` | El pago, sus aplicaciones y sus imputaciones |
| `handle(AnnulSupplierPaymentCommand)` | Una | El pago (`annulled_*`, `updated_*`) y la fila delicada de la bitácora |
| `OrderPaymentTransfers.transfer(invoiceId, orderId)` | `MANDATORY`, dentro del registro o del `PUT` de la factura (fase 2) | Las aplicaciones que pasan, y una fila de la bitácora por cada una |

**El registro, paso a paso.** Los candados se toman primero, en el orden de §2.7; después, cada revisión responde el primer fallo y no escribe nada, y el orden de las revisiones fija qué código sale cuando fallan varios (§4.2.4):
1. Forma del pedido (Bean Validation, `400`).
2. Sin `pagos.registrar_pago` en ninguna empresa: `403`.
3. **Idempotencia:** `SupplierPaymentRepository.lockRequest(requestId)`, un `pg_advisory_xact_lock` de la transacción sobre el `requestId`. Si ya hay un pago con ese `requestId` y el mismo contenido, responde ese pago y no escribe nada; si el contenido difiere, `409` (§4.6).
4. **Candados**, en el orden de §2.7:
   - la empresa (`fetchPurchasingCompanyForShare`);
   - las facturas que nombra, por `id` ascendente (`FOR UPDATE`), después de leer sus `id` sin candado, como la nota (fase 4, su §2.3, paso 9);
   - el proveedor: `FOR SHARE` con un pago de dinero y `FOR NO KEY UPDATE` con un pago con saldo (D-195);
   - la moneda del pago (`fetchCurrencySummaryByIsoCodeForShare`);
   - las órdenes que nombra, por `id` ascendente, con el candado de `PurchaseOrderJpaRepository.lockByPublicId` (`FOR NO KEY UPDATE`).

   Lo que no existe o no se ve queda sin candado, y lo responde su revisión.
5. Empresa, proveedor y moneda: existen, se leen, el permiso actúa y la empresa y la moneda están activas.
6. Montos: escala y tope.
7. Fecha (D-212) y tasa del pago.
8. Método y número de operación (D-212); el número repetido entre los no anulados, `409`.
9. Destinos: de qué proveedor son, su estado, la pareja de monedas y la tasa de cada aplicación frente a la vista.
10. Alcance sobre las marcas de las líneas (D-212).
11. **Lo que se debe**, con las facturas bloqueadas: cada aplicación hasta lo que se debe (D-190).
12. La suma de los equivalentes, igual al monto (D-194, D-210). Desde `5c77bc8f`, con el residuo del redondeo absorbido (§2.4).
13. **El saldo**, con el proveedor bloqueado: un pago con saldo, hasta el saldo de su moneda (D-195).
14. **Imputaciones:**
    - las unidades de las líneas vigentes de cada factura;
    - en un proveedor de dealers, obligatorias (D-197);
    - la suma, igual a lo cubierto;
    - ninguna unidad saldada;
    - nada más de lo que le falta (D-211);
    - `settlesUnit` solo con un proveedor de dealers.
15. Guarda. Un único que la base detiene en una carrera se traduce con `UniqueViolations` a su `409`.
16. Responde lo que guardó (convenciones §6.2).

**Como se construyó** (`a86065d6`; decisión del responsable del 03/11/2026). Los `400` que el servicio decide —el método, la moneda del monto, el número que no deja nada al normalizarlo, los destinos de cada aplicación, las monedas de su retención y sus imputaciones, el comprobante de más de 30 caracteres y más de 500 imputaciones— van antes del paso 3, y el `403` de la ruta, antes que ellos (convenciones §7.2). El orden completo, en §4.2.4.

**La anulación:**
1. Bloquea el pago (`FOR NO KEY UPDATE`).
2. Comprueba el permiso con el alcance de registrarlo, el `updatedAt` y que no esté anulado.
3. Bloquea al proveedor `FOR NO KEY UPDATE` y vuelve a leer las aplicaciones del pago, para ver un paso de la orden a la factura confirmado mientras tanto.
4. Calcula con `SupplierBalancePolicy` el saldo de cada moneda que el pago toca, sin él. Si alguno queda negativo, `422` (D-198).
5. Marca la anulación y escribe la fila delicada de la bitácora (§4.2.5).

**El paso de la orden a la factura** (D-198, `OrderPaymentTransfers`):
- **Cuándo:** lo llama `PurchaseInvoiceOrderLinks` de la fase 2 cada vez que una factura queda vinculada a una orden, al registrarla con `purchaseOrderId` o al cambiar su orden con el `PUT`. La orden ya está bloqueada por la fase 2.
- **Qué pasa:** las aplicaciones de pagos no anulados que siguen en la orden, de la más antigua a la más nueva, pasan a la factura hasta lo que se debe de ella. Lo que sobra queda en la orden, y el pago no se toca.
- **Cómo:** una aplicación que pasa entera cambia de destino; una que pasa en parte se parte en dos filas (`giveToInvoice`, §2.5). Cada aplicación que pasa deja una fila en la bitácora, no delicada: `entity = payment_applications`, `field_name = purchase_invoice_id`, con el número de la factura como valor nuevo y, si se partió, otra con `amount`.
- **Lo que no pasa** (D-211):
  - si la factura y la orden no tienen la misma moneda, nada pasa, y la orden lo muestra como pagos por aplicar;
  - desvincular la factura no devuelve nada a la orden.
- **Permisos:** no exige ningún permiso de pagos: es una consecuencia del vínculo, que exige `compras.registrar_factura` (D-198, «pasa sola»).

**Como se corrigió (04/11/2026): el monto que queda en la orden, en la bitácora** (backend `ad87b4d9`, frontend `c510e131`; decisión C6 de la revisión, D-105, D-212). La fila `amount` dice cuánto se quedó en la orden una aplicación que pasó en parte: es un monto de un pago. Quien lee la bitácora con `bitacora.ver` pero sin `pagos.ver` en la empresa de la fila ve que el campo cambió, sin el valor anterior ni el nuevo, y la fila trae `valuesWithheld: true`. `procurement` lo marca por su puerto de campos confidenciales (`ProcurementConfidentialChangeLogFieldsAdapter`, con `PaymentApplication.AMOUNT_FIELD`), y la marca se lee al consultar, así que cubre también las filas ya escritas. La fila `purchase_invoice_id`, con el número de la factura, sigue a la vista. La pantalla de la bitácora dice por qué se oculta cada valor: los costos de una unidad, un pago u otro alcance (`c510e131`). Lo prueban `PurchaseInvoicesIntegrationTests.theChangeLogWithholdsTheAmountOfAnApplicationFromWhoDoesNotReadThePayments`, `ChangeLogAccessPolicyTests.theValuesOfAConfidentialFieldTravelWithItsPermissionInTheCompanyOfTheRow` y `change-logs-view.spec` ("says why each value is withheld by its field: the costs of a unit, a payment, or another reach (C6)").

**Ejemplo 7 · El pago de una orden pasa a sus facturas.** Orden OC-LZV-2026-0007 de Derco, en dólares, enviada:

| Fecha | Hecho | En la orden | En la factura |
|---|---|---|---|
| 02/03 | Pago OP-3001 de USD 34 500,00, aplicado a la orden | 34 500,00 | — |
| 22/03 | Se registra F002-990, USD 13 100,00, vinculada a la orden | 21 400,00 | F002-990: 13 100,00; se debe 0,00 |
| 29/03 | Se vincula F002-991, USD 13 100,00, con el `PUT` | 8 300,00 | F002-991: 13 100,00; se debe 0,00 |
| 30/03 | Se intenta anular la orden | `422 PURCHASEORDER_HAS_PAYMENTS`: le quedan 8 300,00 | — |

El pago OP-3001 sigue siendo uno solo, de 34 500,00, con tres aplicaciones. Su fecha, 02/03, es la fecha base de la brecha de las dos unidades, porque es el primer pago que cubre cada una (§2.3).

**Rendimiento.** Las llaves son `GENERATED ALWAYS AS IDENTITY`, así que hay una inserción por fila. Todo lo demás es constante por pago: una lectura de hechos por clase en `PayableFactsReadRepository`, para las facturas, sus líneas, sus notas, sus aplicaciones y sus imputaciones; un bloqueo por clase con `id = ANY(...)`; y una elevación a lo sumo por llamada a `fleet`. `SupplierPaymentRegistrationStatementsTests` lo fija (§7.2).

### 2.7 Orden de los candados

Es la secuencia de la fase 4 (su §2.4), que extiende la de la fase 2, con los números de la fase 4. Esta fase solo suma el `requestId` y pone el pago donde está la nota:

| # | Qué | Modo | Quién lo toma |
|---|---|---|---|
| 0 | El `requestId` | `pg_advisory_xact_lock` | El registro de un pago |
| 1 | El pago, en su anulación, como la nota en la suya | `FOR NO KEY UPDATE` | Solo la anulación del pago |
| 2 | La empresa | `FOR SHARE` (por `organization`) | El registro de un pago, de una nota y de una factura. **Como se corrigió (04/11/2026):** las marcas de las unidades nuevas de una factura no se toman aquí: `fleet` las toma `FOR SHARE` después de la orden y de las unidades existentes (especificación de las facturas, §2.4). Un pago no toma marcas |
| 3 | Las facturas, por `id` ascendente | `FOR UPDATE` | El registro de un pago; el registro y la anulación de una nota; el `PUT` y la anulación de una factura |
| 4 | El proveedor | `FOR SHARE` en un pago de dinero y en el registro de una factura o una orden; `FOR NO KEY UPDATE` en un pago con saldo, la anulación de un pago y la anulación de una nota, el candado del saldo (D-195) | Las anteriores, y sus contactos y su estado |
| 5 | La moneda | `FOR SHARE` | El registro de un pago y de una factura (PRC-R26) |
| 6 | Las órdenes, por `id` ascendente | `FOR NO KEY UPDATE`, el de `PurchaseOrderJpaRepository.lockByPublicId` | El registro de un pago con órdenes; el vínculo de una factura, que lleva el paso de la orden a la factura; la anulación y el cumplimiento de la orden |
| 7 | Las unidades | — | Un pago no las toma: su unidad la nombra una línea vigente de una factura que ya bloqueó |

**Por qué el pago se bloquea en `FOR NO KEY UPDATE` y no en `FOR UPDATE`, como la nota.** Una parte de una aplicación que pasa de la orden a la factura inserta una fila que apunta al pago, y su llave foránea toma `FOR KEY SHARE` sobre él. Con `FOR UPDATE`, un vínculo que ya tiene el proveedor en modo compartido esperaría al pago, y la anulación del pago, que espera al proveedor en modo exclusivo, lo esperaría a él.

**Lo que cubre:**
- **Dos pagos a la misma factura:** se esperan en el candado 3, así que el segundo ve lo que aplicó el primero y no aplica de más.
- **La anulación de una factura y un pago a ella:** se esperan en el candado 3. El primero que llega gana, y el otro ve su resultado: dependientes o factura anulada.
- **Un pago con saldo, la anulación de un pago y la de una nota:** se esperan en el candado 4, así que el saldo que cada uno calcula es el que dejó el anterior.
- **Una orden que se anula y un pago a ella:** se esperan en el candado 6 (D-198).
- **Una nota que anula una línea y una imputación a esa unidad:** se esperan en el candado 3, que la nota toma antes de anular la línea (fase 4, su §2.3, pasos 9 y 12).

Lo que no cubre un candado lo cubre un único:
- dos registros con el mismo `requestId` chocan en `supplier_payments_request_uq`;
- dos con el mismo número de operación, en `supplier_payments_operation_uq`.

### 2.8 Puertos y fachadas entre contextos

**`PurchaseInvoiceDependents` para los pagos** (fase 2, su §2.5.4). `PaymentInvoiceDependentsAdapter` (`infrastructure/dependents`, junto al de las notas de la fase 4) responde verdadero si la factura tiene una aplicación de un pago no anulado, también una que pasó de la orden. `PurchaseInvoiceAnnulments` ya recibe la lista de beans. La prueba de la fase 4 `PurchaseInvoiceDependentsTests.untilPaymentsExistOnlyCreditNotesAreDependents` pasa a `creditNotesAndPaymentsAreTheDependents`: exactamente dos beans.

```java
@Component
class PaymentInvoiceDependentsAdapter implements PurchaseInvoiceDependents {
    /** Una factura con un pago aplicado no se anula (D-161): primero se anula el pago. */
    public boolean anyFor(Long purchaseInvoiceId) { /* SupplierPaymentRepository.anyCurrentApplicationTo */ }
}
```

**La guarda del saldo que declara la fase 4** (su §2.5.3, D-193). La fase 4 declara en `procurement/application/internal/outboundservices/payments` el puerto `SupplierBalanceAfterAnnulment`, que su `CreditNoteAnnulments` pregunta con el proveedor ya bloqueado `FOR NO KEY UPDATE` (candado 4), y con la lista vacía responde «no queda negativo». Esta fase lo implementa:

```java
// Declarado por la fase 4
public interface SupplierBalanceAfterAnnulment {
    boolean leavesBalanceNegative(Long creditNoteId);
}

// Esta fase, en infrastructure/payments
@Component
class PaymentSupplierBalanceAdapter implements SupplierBalanceAfterAnnulment {
    /**
     * Si, sin las líneas de la nota, el saldo del proveedor en la moneda de la nota queda negativo (D-190, D-193):
     * SupplierBalancePolicy con la nota excluida, sobre los hechos de PayableFactsReadRepository.
     */
    public boolean leavesBalanceNegative(Long creditNoteId) { … }
}
```

La prueba de la fase 4 `SupplierBalanceAfterAnnulmentTests.untilPaymentsExistNoAnnulmentLeavesTheBalanceNegative` pasa a `thePaymentsAreTheOnlyGuardOfTheBalance`: exactamente un bean, este.

**`CurrencyUsage`** (`location.interfaces.acl`). `SupplierPaymentCurrencyUsageAdapter` responde si algún pago, de cualquier estado, está en esa moneda. Las aplicaciones están en la moneda de su factura o su orden, que ya la fijan (PRC-R26 y la fase 2).

**`UnitPurchases`**, el puerto que `fleet` declara y `procurement` implementa (fases 2 y 4), suma un miembro, y el costo de compra de la fase 4 suma el pago:

```java
/** El estado de pago y la fecha base de la brecha de cada unidad con línea vigente (D-199), sin montos. */
Map<Long, UnitPaymentFacts> paymentFacts(Set<Long> vehicleIds);

/**
 * status: SIN_PAGO, PARCIAL, PAGADA o SALDADA. baseDate: el primer pago no anulado que cubre parte de la unidad, por su
 * imputación o por su parte de lo que su factura no imputó, o, sin ninguno, la fecha de la factura (D-199).
 */
public record UnitPaymentFacts(String status, LocalDate baseDate) {}

/** UnitPurchaseCost (fase 4, su §2.5.2) suma, al final, el pago de la unidad. */
public record UnitPurchaseCost(/* … los campos de la fase 4 … */, UnitPaymentPosition payment) {}

/** Los montos 4 y 5 de D-199, el dinero entregado y los pagos aplicados a su factura. */
public record UnitPaymentPosition(String status, LocalDate baseDate, UnitAmount covered, UnitAmount difference,
        @Nullable String differenceKind, UnitAmount delivered, UnitAmount withheld, UnitAmount coveredWithBalance,
        List<UnitPaymentRecord> payments) {}

public record UnitPaymentRecord(UUID paymentPublicId, @Nullable String operationNumber, LocalDate paymentDate,
        String method, UnitAmount appliedToUnit) {}
```

- **Quién lo responde:** `PurchaseInvoiceUnitPurchasesAdapter` (fases 2 y 4), con `PayablePositions`, bajo el alcance de quien llama. `UnitPurchaseFacts` toma la única respuesta, y una unidad sin línea vigente no aparece. Los montos 1 a 3 ya los trae `UnitPurchaseCost` de la fase 4 (`invoicedAmount`, `creditNotesTotal`, `vehicleValue`); `UnitAmount` es su record. Desde el 04/11/2026 (backend `c43d1bc1`), `paymentFacts` y `purchaseCostOf` dan la fecha base de cada unidad, `UnitCoverage.baseDate`, y no la de su factura (§2.3).
- **Lo que hace `fleet` con `paymentFacts`:** `DayGapPolicy` cuenta los días desde `baseDate` hasta el primer paso a Activo, la «Entregado» de la fase 3 (FLT-R58), en la zona de la empresa; si no se entregó, hasta hoy. El resultado va en la ficha de la unidad, sin montos (§4.2.11).
- **Lo que hace con `payment`:** lo devuelve en `GET /vehicles/{id}/purchase-cost`, la ruta de la fase 4 (su §4.2.9, D-203), que ya exige `stock.ver_costos` en la empresa y la marca de la unidad. Así la unidad de `/vehicles` sigue sin montos (convenciones §9.2), y el costo y el pago de la unidad se leen en una sola ruta.
- **Por qué es un puerto:** `fleet` sigue sin depender de `procurement` (FLT-R53, FLT-R96).

**Como se construyó** (`ecc963ec`, `64339927`, `56f992c8`, `1ea73d0d`). `UnitPaymentFacts`, `UnitPaymentPosition` y `UnitPaymentRecord` son records anidados en `UnitPurchases`, como los de la fase 4. `UnitPurchaseCost.payment` viaja siempre que la unidad tiene línea vigente. Las dos pruebas de la fase 4 cambiaron de nombre como se pedía: `PurchaseInvoiceDependentsTests.creditNotesAndPaymentsAreTheDependents` y `SupplierBalanceAfterAnnulmentTests.thePaymentsAreTheOnlyGuardOfTheBalance`. `UnitPurchasesWiringTests`, la del único bean de `paymentFacts`, vive en `procurement/infrastructure/fleet`.

**Las fachadas que ya existen:**
- `LocationContextFacade.fetchEffectiveRate` (fase 2), para la tasa del pago y la de cada aplicación;
- `OrganizationContextFacade.fetchPurchasingCompanyForShare` (fase 2);
- `FleetContextFacade.fetchUnitsByIds` (fase 2), para el VIN, el modelo y la marca de cada unidad imputada, en lote.

### 2.9 Lo que esta fase toma de la fase 4

La [especificación de las notas de crédito](especificacion-notas-de-credito-2026-11-03.md) ya expone lo que esta fase necesita:

| # | Lo que expone la fase 4 | Dónde | Cómo lo usa esta fase |
|---|---|---|---|
| 1 | `CreditNoteRepository.currentCreditsByInvoiceIds` y `currentCreditsByInvoiceItemIds`, con `CreditedTotal` | Su §2.5.4, PRC-R92 | Las notas de lo que se debe (§2.2) y de los cinco montos (§2.3) |
| 2 | El puerto `SupplierBalanceAfterAnnulment`, que pregunta con el proveedor bloqueado `FOR NO KEY UPDATE` | Su §2.5.3, PRC-R86 | Lo implementa `PaymentSupplierBalanceAdapter` (§2.8) |
| 3 | El orden de candados con la empresa antes que las facturas, y las facturas antes que el proveedor: «un pago toma las facturas a las que se aplica antes que el proveedor» | Su §2.4 | §2.7 lo sigue con sus mismos números |
| 4 | Una línea que una nota deja en cero se anula bajo el candado de su factura; anular la nota puede devolverla a vigente | Su §2.3 y §2.6 | Una imputación nueva solo nombra una línea vigente (PRC-R116), y una línea que vuelve a vigente vuelve a contar en los cinco montos |
| 5 | `UnitPurchaseCost` y `GET /vehicles/{id}/purchase-cost`, con `stock.ver_costos` | Su §2.5.2 y §4.2.9; D-203 | Suma el pago de la unidad (§2.8, §4.2.11) |
| 6 | `allowedActions` de la factura con `registerCreditNote` al final, después de `downloadXml` | Su §4.5 | `registerPayment` va después (§4.5) |
| 7 | Las reglas PRC hasta PRC-R99 y FLT-R94 a FLT-R99 | Su §5 | Esta fase toma PRC-R100 a PRC-R139 y FLT-R130 a FLT-R132; las cargas de archivos toman FLT-R100 a FLT-R126 y PRC-R140 en adelante |
| 8 | La revisión 36 de `DESIGN.md`, con el panel «Compra y costo» de la ficha de la unidad | Su §6.9, ítem 7 | La revisión 37 le suma la sección del pago (§6.11) |

**Lo que esta fase le pide a la fase 4:** nada nuevo. Una diferencia que se resuelve aquí: la fase 4 no bloquea al proveedor al registrar una nota; esta fase no lo necesita, porque una nota nueva solo puede subir el saldo, nunca bajarlo (§2.2).

### 2.10 Los pagos migrados (D-207)

La carga inicial escribe los pagos del Excel de stock con este modelo, en su paso de pagos, que se construye después de esta fase (D-208). Los escribe `MigratedPaymentWriter`, de la [especificación de las cargas de archivos](especificacion-cargas-de-archivos-2026-11-03.md) (su §2.8, paso 8, y su PRC-R153), con el constructor de registro de `SupplierPayment` y las revisiones de §2.6, salvo lo que esta tabla dice:

| Dato | En un pago migrado | Fuente |
|---|---|---|
| Cuántos | Uno por bloque: el proveedor y el texto de la columna 42 normalizado con `OperationNumber`. Un pago segmentado, `PAGO1/PAGO2/PAGO3`, es un solo pago, porque el Excel guarda solo su total (guía, §7.3) | D-207 |
| Método y número | `transferencia`, con el texto del bloque como número de operación | D-207, D-212 |
| Fecha, monto y moneda | La fecha del primer voucher (columna 43), el monto acumulado (columna 44) y la moneda de sus facturas (columna 46), iguales en todas las filas del bloque | Guía, columnas 43, 44 y 46 |
| Tasa del pago | 1 en la moneda funcional (D-169); en otra, la de venta del catálogo para el día del pago y, si el catálogo no la tiene, la del Excel, como la factura migrada (D-177). No hay una tasa vista que comparar, así que no se responde `SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED` | D-207 |
| Aplicaciones | Una por factura del bloque, todas en la moneda del pago, así que su tasa es 1 (D-210). Reparten el monto en proporción al valor de las unidades del bloque en cada factura, con los céntimos en la primera, sin pasar de lo que se debe de cada una (D-190). El pago se aplica completo (D-210): si el bloque no cabe en lo que se debe, falla | D-207, D-190, D-210 |
| Imputaciones | Ninguna en el canal normal: lo cubierto se reparte como §2.3. En el canal de dealers, cada unidad se imputa por su valor menos la diferencia de la columna 45, con «salda la unidad» (D-197). Cada aplicación es la suma de las imputaciones de sus unidades, y entre todas suman el monto del bloque, porque la columna 45 reparte la diferencia del bloque en partes iguales. El perfil señala que esa diferencia viene prorrateada. Una diferencia negativa daría una imputación mayor que el valor de la unidad, y el bloque falla (PRC-R119) | D-207, D-197 |
| `requestId` | Un UUID versión 5 de la empresa, el proveedor y el número de operación: volver a subir el bloque responde el mismo pago (PRC-R129) | Cargas, PRC-R153 |
| Permiso | El de la carga: no pide `pagos.registrar_pago`, como la factura migrada no pide `compras.registrar_factura` | Cargas, PRC-R153 |

**Lo demás no cambia:**
- un `SUPPLIERPAYMENT_*` va al informe de la carga, con las filas de su bloque;
- un pago migrado se lee, se exporta y se anula como cualquier otro (§4.2.5);
- con su pago, la brecha de días de una unidad migrada cuenta desde el primer voucher, como la guía (columna 24), y no desde la factura.

---

## 3. Esquema

### 3.1 Numeración

Los números se asignan al integrar en `develop`, después de lo que hayan integrado las fases 2, 3 y 4 (plan §4). Aquí van relativos: V(n), V(n+1) y V(n+2), en este orden. Cada migración empieza como V90 y V91: con `SELECT set_config('app.access_scope', 'SYSTEM', true);` y con un `lock_timeout` de cinco segundos para la transacción, para que no retenga los pedidos que esperan detrás de ella.

| Migración | Contexto | Paso | Contenido |
|---|---|---|---|
| V(n) `seed_iam_permissions_of_supplier_payments` | `iam` | 7.1 | Tres permisos y sus nueve marcas |
| V(n+1) `add_supplier_dealer_channel` | `procurement` | 7.2 | `suppliers.is_dealer` |
| V(n+2) `create_procurement_supplier_payments_schema` | `procurement` | 7.3 | Las tres tablas, sus restricciones, disparadores, RLS, permisos de la cuenta y `data_versions` |

**Como se construyó.** Las fases 2, 3 y 4 se integraron antes, hasta V104, así que n es 105: V105 `seed_iam_permissions_of_supplier_payments` (`32ff123d`), V106 `add_supplier_dealer_channel` (`3b463a6d`) y V107 `create_procurement_supplier_payments_schema` (`fd15e8cf`).

### 3.2 V(n): los permisos (§5.6)

```sql
INSERT INTO permissions (code, module, description, category)
VALUES ('pagos.ver', 'pagos', 'Pagos, aplicaciones, imputaciones y el saldo con cada proveedor, con sus importes', 'ver'),
       ('pagos.registrar_pago', 'pagos', 'Registrar, aplicar e imputar un pago', 'crear'),
       ('pagos.anular_pago', 'pagos', 'Anular un pago mal registrado', 'desactivar_anular')
ON CONFLICT (code) DO NOTHING;

INSERT INTO role_permissions (role_id, permission_id, granted_by)
SELECT role.id, permission.id, system_account.id
FROM (VALUES ('SUPERADMIN', 'pagos.ver'), ('SUPERADMIN', 'pagos.registrar_pago'), ('SUPERADMIN', 'pagos.anular_pago'),
             ('ADMIN_EMPRESA', 'pagos.ver'), ('ADMIN_EMPRESA', 'pagos.registrar_pago'), ('ADMIN_EMPRESA', 'pagos.anular_pago'),
             ('CONTABILIDAD', 'pagos.ver'), ('CONTABILIDAD', 'pagos.registrar_pago'), ('CONTABILIDAD', 'pagos.anular_pago'))
         AS matrix (role_code, permission_code)
JOIN roles role ON role.code = matrix.role_code AND role.is_system AND role.company_id IS NULL
JOIN permissions permission ON permission.code = matrix.permission_code
CROSS JOIN (SELECT id FROM users WHERE email = 'system@millennium.ky') AS system_account
ON CONFLICT DO NOTHING;
```

Las descripciones son las de §5.6 sin la referencia entre paréntesis, como V79 y V90. La matriz marca SA, AE y CO en los tres (§5.6): nueve marcas, firmadas por `SYSTEM`. No hay pares incompatibles en pagos (§5.6).

### 3.3 V(n+1): el canal de dealers (D-197)

```sql
ALTER TABLE suppliers ADD COLUMN is_dealer BOOLEAN NOT NULL DEFAULT false;
```

Todos los proveedores que ya existen quedan fuera del canal. Se marca en su ficha (§4.2.9). `suppliers` ya está versionada en `data_versions` por su exportación, así que la columna nueva mueve su versión sin más.

### 3.4 V(n+2): las tablas de pagos

```sql
CREATE TABLE supplier_payments
(
    id                  BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id           UUID           NOT NULL DEFAULT uuidv7(),
    company_id          BIGINT         NOT NULL REFERENCES companies (id),
    supplier_id         BIGINT         NOT NULL,
    payment_date        DATE           NOT NULL,
    currency_id         BIGINT         NOT NULL REFERENCES currencies (id),
    exchange_rate_date  DATE           NOT NULL,                  -- D-169, D-210: el día de la tasa del pago
    exchange_rate       NUMERIC(12, 6) NOT NULL,
    amount              NUMERIC(14, 2) NOT NULL,
    method              VARCHAR(30)    NOT NULL,
    operation_number    VARCHAR(100),
    request_id          UUID           NOT NULL,
    notes               TEXT,
    annulled_at         TIMESTAMPTZ,
    annulled_by         BIGINT         REFERENCES users (id),
    annulment_reason    VARCHAR(30),
    created_at          TIMESTAMPTZ    NOT NULL DEFAULT now(),
    created_by          BIGINT         REFERENCES users (id),
    updated_at          TIMESTAMPTZ    NOT NULL DEFAULT now(),    -- D-198, D-210: la anulación lleva el updated_at leído
    updated_by          BIGINT         REFERENCES users (id),     -- D-210

    CONSTRAINT supplier_payments_public_id_uq UNIQUE (public_id),
    CONSTRAINT supplier_payments_id_company_uq UNIQUE (id, company_id),
    CONSTRAINT supplier_payments_request_uq UNIQUE (request_id),
    CONSTRAINT supplier_payments_supplier_company_fk
        FOREIGN KEY (supplier_id, company_id) REFERENCES suppliers (id, company_id),
    CONSTRAINT supplier_payments_method_ck
        CHECK (method IN ('transferencia', 'cheque', 'efectivo', 'saldo_a_favor')),
    CONSTRAINT supplier_payments_amount_ck CHECK (amount > 0),
    CONSTRAINT supplier_payments_exchange_rate_ck CHECK (exchange_rate > 0),
    CONSTRAINT supplier_payments_exchange_rate_date_ck CHECK (exchange_rate_date <= payment_date),
    CONSTRAINT supplier_payments_operation_number_ck
        CHECK (operation_number IS NULL
            OR (length(operation_number) > 0 AND operation_number = upper(operation_number) AND operation_number !~ '[\s.-]')),
    -- D-212: la transferencia y el cheque llevan su número; el saldo a favor no lleva ninguno
    CONSTRAINT supplier_payments_operation_by_method_ck
        CHECK ((method IN ('transferencia', 'cheque') AND operation_number IS NOT NULL)
            OR (method = 'saldo_a_favor' AND operation_number IS NULL)
            OR method = 'efectivo'),
    CONSTRAINT supplier_payments_annulment_reason_ck
        CHECK (annulment_reason IS NULL OR annulment_reason IN ('ERROR_REGISTRO', 'DEVUELTO')),   -- D-212
    CONSTRAINT supplier_payments_annulment_ck
        CHECK ((annulled_at IS NULL AND annulled_by IS NULL AND annulment_reason IS NULL)
            OR (annulled_at IS NOT NULL AND annulled_by IS NOT NULL AND annulment_reason IS NOT NULL))
);

-- El número de operación no se repite en el proveedor entre los pagos no anulados (D-198)
CREATE UNIQUE INDEX supplier_payments_operation_uq
    ON supplier_payments (company_id, supplier_id, operation_number)
    WHERE annulled_at IS NULL AND operation_number IS NOT NULL;
CREATE INDEX idx_supplier_payments_company_date ON supplier_payments (company_id, payment_date DESC, id DESC);
CREATE INDEX idx_supplier_payments_supplier_currency
    ON supplier_payments (company_id, supplier_id, currency_id) WHERE annulled_at IS NULL;
CREATE INDEX idx_supplier_payments_currency ON supplier_payments (currency_id);

CREATE TABLE payment_applications
(
    id                       BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id                UUID           NOT NULL DEFAULT uuidv7(),
    payment_id               BIGINT         NOT NULL,
    company_id               BIGINT         NOT NULL REFERENCES companies (id),
    purchase_invoice_id      BIGINT,
    purchase_order_id        BIGINT,
    amount                   NUMERIC(14, 2) NOT NULL,
    payment_amount           NUMERIC(14, 2) NOT NULL,
    withheld_amount          NUMERIC(14, 2) NOT NULL DEFAULT 0,
    withholding_certificate  VARCHAR(30),
    exchange_rate_date       DATE           NOT NULL,              -- D-210
    exchange_rate            NUMERIC(12, 6) NOT NULL,              -- D-210

    CONSTRAINT payment_applications_public_id_uq UNIQUE (public_id),
    CONSTRAINT payment_applications_id_company_uq UNIQUE (id, company_id),
    CONSTRAINT payment_applications_payment_company_fk
        FOREIGN KEY (payment_id, company_id) REFERENCES supplier_payments (id, company_id),
    CONSTRAINT payment_applications_invoice_company_fk
        FOREIGN KEY (purchase_invoice_id, company_id) REFERENCES purchase_invoices (id, company_id),
    CONSTRAINT payment_applications_order_company_fk
        FOREIGN KEY (purchase_order_id, company_id) REFERENCES purchase_orders (id, company_id),
    -- Una factura o una orden, nunca las dos ni ninguna (§10.2)
    CONSTRAINT payment_applications_target_ck CHECK ((purchase_invoice_id IS NULL) <> (purchase_order_id IS NULL)),
    CONSTRAINT payment_applications_amount_ck CHECK (amount > 0 AND payment_amount > 0),
    CONSTRAINT payment_applications_withheld_ck CHECK (withheld_amount >= 0),
    CONSTRAINT payment_applications_certificate_ck
        CHECK (withheld_amount = 0 OR withholding_certificate IS NOT NULL),                 -- D-210
    CONSTRAINT payment_applications_exchange_rate_ck CHECK (exchange_rate > 0)
);

CREATE INDEX idx_payment_applications_payment ON payment_applications (payment_id, id);
CREATE INDEX idx_payment_applications_invoice ON payment_applications (purchase_invoice_id)
    WHERE purchase_invoice_id IS NOT NULL;
CREATE INDEX idx_payment_applications_order ON payment_applications (purchase_order_id)
    WHERE purchase_order_id IS NOT NULL;

CREATE TABLE payment_allocations
(
    id                       BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    public_id                UUID           NOT NULL DEFAULT uuidv7(),
    payment_application_id   BIGINT         NOT NULL,
    company_id               BIGINT         NOT NULL REFERENCES companies (id),
    vehicle_id               BIGINT         NOT NULL,
    amount                   NUMERIC(14, 2) NOT NULL,
    settles_unit             BOOLEAN        NOT NULL DEFAULT false,

    CONSTRAINT payment_allocations_public_id_uq UNIQUE (public_id),
    CONSTRAINT payment_allocations_application_company_fk
        FOREIGN KEY (payment_application_id, company_id) REFERENCES payment_applications (id, company_id),
    CONSTRAINT payment_allocations_vehicle_company_fk
        FOREIGN KEY (vehicle_id, company_id) REFERENCES vehicles (id, company_id),
    CONSTRAINT payment_allocations_application_vehicle_uq UNIQUE (payment_application_id, vehicle_id),
    CONSTRAINT payment_allocations_amount_ck CHECK (amount > 0)
);

CREATE INDEX idx_payment_allocations_vehicle ON payment_allocations (company_id, vehicle_id);
```

Las llaves compuestas usan únicos que ya existen: `suppliers_id_company_uq` (V65), `purchase_orders_id_company_uq` (V69), `vehicles_id_company_uq` (V50) y `purchase_invoices_id_company_uq` (fase 2).

**Lo que la base no decide y queda en el servicio:**
- lo que se debe y el saldo, que son sumas entre tablas (D-190);
- la pareja de monedas, porque la moneda funcional vive en `companies` (D-194);
- la fecha futura, porque depende del momento (D-212);
- la unidad saldada y el tope por unidad, porque dependen de pagos no anulados (D-197, D-211);
- que una factura vigente reciba pagos;
- que una orden esté enviada (D-211).

**Los disparadores:**

```sql
-- Un pago aplica solo a facturas y órdenes de su proveedor (§10.1: «Un pago no puede repartirse entre proveedores»)
CREATE FUNCTION enforce_payment_application_supplier() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
DECLARE
    paying   BIGINT;
    targeted BIGINT;
BEGIN
    SELECT supplier_id INTO paying FROM supplier_payments WHERE id = NEW.payment_id;
    IF NEW.purchase_invoice_id IS NOT NULL THEN
        SELECT supplier_id INTO targeted FROM purchase_invoices WHERE id = NEW.purchase_invoice_id;
    ELSE
        SELECT supplier_id INTO targeted FROM purchase_orders WHERE id = NEW.purchase_order_id;
    END IF;
    IF paying IS DISTINCT FROM targeted THEN
        RAISE EXCEPTION 'A payment applies only to invoices and orders of its own supplier';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER payment_applications_supplier_tg
    BEFORE INSERT OR UPDATE OF payment_id, purchase_invoice_id, purchase_order_id ON payment_applications
    FOR EACH ROW EXECUTE FUNCTION enforce_payment_application_supplier();

-- Una aplicación no se edita: solo la de una orden se da, entera o en parte, a una factura (D-198)
CREATE FUNCTION keep_payment_application_data() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    IF (NEW.payment_id, NEW.company_id, NEW.withholding_certificate, NEW.exchange_rate_date, NEW.exchange_rate)
       IS DISTINCT FROM
       (OLD.payment_id, OLD.company_id, OLD.withholding_certificate, OLD.exchange_rate_date, OLD.exchange_rate) THEN
        RAISE EXCEPTION 'An application keeps its payment, its company, its certificate and its rate';
    END IF;
    IF OLD.purchase_order_id IS NULL THEN
        RAISE EXCEPTION 'An application to an invoice never changes';
    END IF;
    IF NEW.purchase_order_id IS NOT NULL
       AND (NEW.purchase_order_id <> OLD.purchase_order_id OR NEW.purchase_invoice_id IS NOT NULL
            OR NEW.amount > OLD.amount OR NEW.payment_amount > OLD.payment_amount
            OR NEW.withheld_amount > OLD.withheld_amount) THEN
        RAISE EXCEPTION 'An application to an order only gives part of itself to an invoice';
    END IF;
    IF NEW.purchase_order_id IS NULL
       AND (NEW.amount, NEW.payment_amount, NEW.withheld_amount)
           IS DISTINCT FROM (OLD.amount, OLD.payment_amount, OLD.withheld_amount) THEN
        RAISE EXCEPTION 'An application that moves whole to an invoice keeps its amounts';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER payment_applications_data_tg
    BEFORE UPDATE ON payment_applications FOR EACH ROW EXECUTE FUNCTION keep_payment_application_data();

-- Una imputación detalla una aplicación a una factura, y nombra una unidad vigente de esa factura (§10.3)
CREATE FUNCTION enforce_payment_allocation_unit() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
DECLARE
    invoice BIGINT;
BEGIN
    SELECT purchase_invoice_id INTO invoice FROM payment_applications WHERE id = NEW.payment_application_id;
    IF invoice IS NULL THEN
        RAISE EXCEPTION 'An allocation details an application to an invoice, never one to an order';
    END IF;
    IF NOT EXISTS (SELECT 1 FROM purchase_invoice_items
                   WHERE purchase_invoice_id = invoice AND vehicle_id = NEW.vehicle_id AND status = 'VIGENTE') THEN
        RAISE EXCEPTION 'An allocation names a unit of a current line of its invoice';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER payment_allocations_unit_tg
    BEFORE INSERT ON payment_allocations FOR EACH ROW EXECUTE FUNCTION enforce_payment_allocation_unit();

-- Un pago no se edita: se anula y se registra otro (§10.1, D-198)
CREATE FUNCTION keep_supplier_payment_data() RETURNS TRIGGER LANGUAGE plpgsql AS
$$
BEGIN
    IF OLD.annulled_at IS NOT NULL THEN
        RAISE EXCEPTION 'An annulled payment never changes';
    END IF;
    IF (NEW.company_id, NEW.supplier_id, NEW.payment_date, NEW.currency_id, NEW.exchange_rate_date, NEW.exchange_rate,
        NEW.amount, NEW.method, NEW.operation_number, NEW.request_id, NEW.notes, NEW.created_at, NEW.created_by)
       IS DISTINCT FROM
       (OLD.company_id, OLD.supplier_id, OLD.payment_date, OLD.currency_id, OLD.exchange_rate_date, OLD.exchange_rate,
        OLD.amount, OLD.method, OLD.operation_number, OLD.request_id, OLD.notes, OLD.created_at, OLD.created_by) THEN
        RAISE EXCEPTION 'A payment is not edited: annul it and register another';
    END IF;
    RETURN NEW;
END
$$;

CREATE TRIGGER supplier_payments_data_tg
    BEFORE UPDATE ON supplier_payments FOR EACH ROW EXECUTE FUNCTION keep_supplier_payment_data();
```

**Qué cambia en cada tabla:**
- En el pago, solo los datos de la anulación y las columnas de revisión.
- En la aplicación, solo el paso de una orden a una factura.
- La imputación no cambia nunca: la tabla solo recibe inserciones.

**Row Level Security, como V83:**

```sql
ALTER TABLE supplier_payments ENABLE ROW LEVEL SECURITY;
ALTER TABLE supplier_payments FORCE ROW LEVEL SECURITY;
CREATE POLICY supplier_payments_select ON supplier_payments FOR SELECT USING (access_scope_reaches(company_id));
CREATE POLICY supplier_payments_insert ON supplier_payments FOR INSERT WITH CHECK (access_scope_reaches(company_id));
CREATE POLICY supplier_payments_update ON supplier_payments FOR UPDATE
    USING (access_scope_reaches(company_id)) WITH CHECK (access_scope_reaches(company_id));
-- Las mismas tres en payment_applications. En payment_allocations, solo _select e _insert: sus filas solo se insertan
```

**Permisos de la cuenta de la aplicación,** con el bloque `DO` de V63 sobre `${app_role}`:
- `REVOKE ALL` en las tres tablas;
- `GRANT SELECT, INSERT, UPDATE` en `supplier_payments` y `payment_applications`;
- `GRANT SELECT, INSERT` en `payment_allocations`.

Ninguna tabla lleva `DELETE`. `RowLevelSecurityMigrationTests` suma las tres a `COMPANY_TABLES`, `payment_allocations` también a `INSERT_ONLY_TABLES`, y una inserción de cada una a la prueba de que ninguna tabla toma una fila de otra empresa.

**`data_versions`:** `supplier_payments` y `payment_applications` muestran la exportación, así que siembran sus 64 casillas y crean sus dos disparadores por sentencia, como V69. `payment_allocations` no se exporta y no se versiona.

**Como se construyó** (`fd15e8cf`, V107). El esquema es el de arriba, con un disparador más, `keep_payment_allocations` (`payment_allocations_data_tg`, `BEFORE UPDATE`): una imputación no cambia nunca, aunque la cuenta de la aplicación tampoco tiene `UPDATE` sobre la tabla. `scripts/database/hand-over-database.sql` da a `erp_mm_owner` las cinco funciones nuevas —`enforce_payment_application_supplier`, `keep_payment_application_data`, `enforce_payment_allocation_unit`, `keep_supplier_payment_data` y `keep_payment_allocations`— y quita y comprueba el `UPDATE` de `payment_allocations`, así que ese guion se vuelve a correr donde se entrega la base. `RowLevelSecurityMigrationTests` suma las tres tablas, `payment_allocations` entre las que solo se insertan, y las dos primeras entre las que un disparador no deja cambiar de empresa.

---

## 4. Contrato de la API

### 4.1 Rutas

| # | Método y ruta | Permiso de la ruta | Respuesta | Paso |
|---|---|---|---|---|
| 1 | `GET /supplier-payments` | `pagos.ver` | `200` página o arreglo | 7.8 |
| 2 | `GET /supplier-payments` con el `Accept` de Excel | `pagos.ver` | `303` a `/documents/{id}` | 7.8 |
| 3 | `GET /supplier-payments/{id}` | `pagos.ver` | `200` ficha | 7.8 |
| 4 | `POST /supplier-payments` | `pagos.registrar_pago` | `201` con `Location`; `200` si repite un envío | 7.5 |
| 5 | `POST /supplier-payments/{id}/annulment` | `pagos.anular_pago` | `200` ficha | 7.6 |
| 6 | `GET /suppliers/{id}/balance` (propuesta en convenciones §2.2) | `pagos.ver` | `200` | 7.8 |
| 7 | `GET /suppliers/{id}/payables` **(propuesta)** | `pagos.ver` | `200` | 7.8 |
| 8 | `GET /purchase-invoices/{id}/payment-position` **(propuesta)** | `pagos.ver` | `200` | 7.8 |

**Rutas que ya existen y cambian:**
- `POST /suppliers`, `PUT /suppliers/{id}`, `GET /suppliers` y `/suppliers/{id}`, y su exportación: `isDealer`.
- `GET /purchase-invoices/{id}`: el estado de pago de cada línea y `registerPayment`.
- `POST /purchase-invoices` y `PUT /purchase-invoices/{id}`: el paso de la orden a la factura.
- `POST /purchase-invoices/{id}/annulment`: los pagos como dependientes.
- `POST /purchase-orders/{id}/annulment` y `/fulfillment`: la orden con pagos.
- `GET /purchase-orders/{id}`: `allowedActions`.
- `GET /vehicles/{id}`: `payment`, sin montos.
- `GET /vehicles/{id}/purchase-cost` (fase 4): `purchase.payment`, con los montos.

`SupplierPaymentsController` lleva `@PermissionRequired(PermissionCodes.Payments.VIEW)` en la clase y el suyo en cada escritura, que reemplaza al de la clase. Las rutas 6 y 7 van en `SuppliersController`, que declara el permiso en cada método (`SuppliersController.java`, sin anotación de clase), y declaran `pagos.ver`. La 8 va en `PurchaseInvoicesController` (fase 2), donde `pagos.ver` en el método reemplaza el `compras.ver` de la clase. En las tres, el servicio exige además leer el proveedor o la factura con `compras.ver` (D-134). No hay `DELETE` (`NoDeleteEndpointsTests`).

**Como se construyó: el orden de las revisiones** (decisiones del responsable del 03/11/2026). Toda ruta revisa primero lo que el marco lee del pedido (`400`, y `413` por encima del tope), después el permiso de la ruta en alguna empresa (`403`) y después el servicio, también sus propios `400`, como las convenciones fijan desde la fase 4 (§7.2). Así van también la anulación y `GET /vehicles/{id}/purchase-cost`: el `403` de la ruta justo después del `400` del marco. Los seis `@PermissionRequired` nuevos llevan el backend a 185.

### 4.2 Cada ruta

Los montos viajan como `{ "amount", "currency" }` (convenciones §5.2): el pedido acepta `amount` como texto decimal o número, y la respuesta lo escribe con los decimales de la moneda, como la factura. `exchangeRate` sigue la misma regla. Las fechas, `AAAA-MM-DD`; los momentos, ISO en UTC.

#### 4.2.1 `GET /supplier-payments`

| `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|
| El número de operación normalizado, la razón social y el documento del proveedor, sin distinguir mayúsculas ni tildes (`fold_search_text`, D-142) | `companyId`; `supplierId`; `method` (los cuatro códigos, en cualquier mayúscula); `status` (`VIGENTE` o `ANULADO`, derivado de `annulled_at`); `currency` (código ISO); `from` y `to`, el período `[from, to)` sobre `paymentDate`; `purchaseInvoiceId` y `purchaseOrderId`, los pagos con una aplicación a ese documento. Un valor fuera de lista o una fecha mal escrita, `400 VALIDATION_ERROR`; un `companyId` fuera del alcance, `404 COMPANY_NOT_FOUND` | `paymentDate`, `operationNumber`, `supplierLegalName`, `amount`, `createdAt` | `paymentDate` desc; los empates, `id` desc |

Lista solo lo que quien llama lee con `pagos.ver` (`companyReach`). Cada fila trae:

```json
{"id": "019b…", "companyId": "0199…", "companyName": "Millennium Perú",
 "supplierId": "0199…", "supplierLegalName": "Derco Perú S.A.", "supplierTaxId": "20605412344", "isDealer": false,
 "paymentDate": "2026-11-03", "method": "transferencia", "operationNumber": "OP99213",
 "amount": { "amount": 61056.00, "currency": "PEN" },
 "applied": { "amount": 61056.00, "currency": "PEN" },
 "applicationCount": 2, "status": "VIGENTE", "annulledAt": null, "createdAt": "2026-11-03T16:20:00Z",
 "allowedActions": null}
```

Las referencias se resuelven en lote.

**Como se construyó** (`b0330d09`; decisión del responsable del 03/11/2026). La lista y la ficha comparten un solo recurso, `SupplierPaymentResource`, armado desde una sola vista, como la factura y la nota: cada fila es la ficha sin sus aplicaciones, con `applications` y `allowedActions` en `null`. `q` busca el número de operación como se escribe o normalizado, y `currency` sigue la regla de los filtros (convenciones §6.2).

#### 4.2.2 Exportación

El mismo `GET`, con el `Accept` de Excel de convenciones §8.1 y los mismos `q`, filtros y `sort`, sin página:
- tipo de documento `supplier-payments` y nombre base `pagos-a-proveedores`;
- alcance `DocumentScopes.grantsOf(scope, pagos.ver)`;
- versión de `supplier_payments`, `payment_applications`, `purchase_invoices`, `purchase_orders` y `suppliers`, más la de `organization` y la de `location`.

**Una fila por aplicación**: las columnas del pago se repiten en cada una, como la guía repite el monto del bloque en cada fila (A-05). Columnas, en este orden (claves `export.supplier-payments.*`):

| # | Encabezado | Valor |
|---|---|---|
| 1 | Empresa | Nombre comercial |
| 2 | Documento del proveedor | `RUC 20605412344` |
| 3 | Proveedor | Razón social |
| 4 | Canal de dealers | Sí / No |
| 5 | Fecha del pago | Celda de fecha |
| 6 | Método | Traducido: Transferencia, Cheque, Efectivo, Saldo a favor |
| 7 | N.° de operación | Normalizado, o vacía |
| 8 | Moneda del pago | Código ISO |
| 9 | Monto del pago | Número, decimales de la moneda |
| 10 | Tipo de cambio del pago | Número, seis decimales |
| 11 | Estado del pago | Vigente / Anulado |
| 12 | Motivo de anulación | Traducido |
| 13 | Aplicado a | Factura / Orden |
| 14 | Documento | El número normalizado de la factura o el de la orden |
| 15 | Moneda del documento | Código ISO |
| 16 | Monto aplicado | Número |
| 17 | Retención | Número |
| 18 | Comprobante de retención | Texto, o vacía |
| 19 | Tasa de la aplicación | Número, seis decimales (D-210) |
| 20 | Día de la tasa | Celda de fecha |
| 21 | Equivalente en la moneda del pago | Número |
| 22 | Moneda de la empresa | Código ISO |
| 23 | Diferencia de cambio (calculada) | Número en la moneda de la empresa; vacía en una orden o en la moneda de la empresa (§2.4) |
| 24 | Unidades imputadas | Número |
| 25 | Registrado (zona) | Momento, con la zona en el encabezado |
| 26 | Anulado (zona) | Momento, o vacía |

La zona de los momentos es la del país de la empresa por la que filtra la exportación o, si no filtra, la de la única que ve quien pide; si ve varias, `America/Lima` (§2.6). Nunca sale el `id` ni el `public_id`.

#### 4.2.3 `GET /supplier-payments/{id}`

`404 SUPPLIERPAYMENT_NOT_FOUND` si no existe o quien llama no lo lee. Responde, para el ejemplo 6 con una segunda aplicación en soles:

```json
{
  "id": "019b…", "companyId": "0199…", "companyName": "Millennium Perú", "functionalCurrency": "PEN",
  "supplierId": "0199…", "supplierLegalName": "Derco Perú S.A.", "supplierTaxId": "20605412344", "isDealer": false,
  "paymentDate": "2026-11-03", "currency": "PEN", "exchangeRate": 1.000000, "exchangeRateDate": "2026-11-03",
  "amount": { "amount": 61056.00, "currency": "PEN" }, "method": "transferencia", "operationNumber": "OP99213",
  "notes": null,
  "applied": { "amount": 61056.00, "currency": "PEN" },
  "exchangeDifference": { "amount": 235.80, "currency": "PEN" },
  "applications": [
    { "id": "019b…", "target": "factura", "purchaseInvoiceId": "019a…", "documentNumber": "F002-981",
      "purchaseOrderId": null, "orderNumber": null,
      "amount": { "amount": 13100.00, "currency": "USD" }, "withheldAmount": { "amount": 0.00, "currency": "USD" },
      "withholdingCertificate": null, "exchangeRate": 3.760000, "exchangeRateDate": "2026-11-03",
      "paymentAmount": { "amount": 49256.00, "currency": "PEN" },
      "exchangeDifference": { "amount": 235.80, "currency": "PEN" }, "allocations": [] },
    { "id": "019b…", "target": "factura", "purchaseInvoiceId": "019a…", "documentNumber": "F001-123",
      "purchaseOrderId": null, "orderNumber": null,
      "amount": { "amount": 11800.00, "currency": "PEN" }, "withheldAmount": { "amount": 0.00, "currency": "PEN" },
      "withholdingCertificate": null, "exchangeRate": 1.000000, "exchangeRateDate": "2026-11-03",
      "paymentAmount": { "amount": 11800.00, "currency": "PEN" }, "exchangeDifference": null,
      "allocations": [
        { "id": "019b…", "vehicleId": "019a…", "vin": "MD2A11CY6LWA12345", "brandName": "BAJAJ",
          "modelName": "DISCOVER 125-ST", "amount": { "amount": 5900.00, "currency": "PEN" }, "settlesUnit": false },
        { "id": "019b…", "vehicleId": "019a…", "vin": "MD2A11CY6LWA12346", "brandName": "BAJAJ",
          "modelName": "DISCOVER 125-ST", "amount": { "amount": 5900.00, "currency": "PEN" }, "settlesUnit": false } ] }
  ],
  "status": "VIGENTE", "annulledAt": null, "annulledBy": null, "annulmentReason": null,
  "createdAt": "2026-11-03T16:20:00Z", "createdBy": { "id": "0199…", "name": "Tere Ramos" },
  "updatedAt": "2026-11-03T16:20:00.123Z",
  "allowedActions": ["annul", "annulAndCopy"]
}
```

- Las aplicaciones llegan en el orden en que se registraron, por su `id`; las imputaciones, igual.
- El VIN, la marca y el modelo son los de hoy de la unidad (`fetchUnitsByIds`).
- `createdBy` y `annulledBy` se nombran por `AuditActorLookupPort`, como en la factura.
- `exchangeDifference` del pago es la suma de las de sus aplicaciones, en la moneda funcional.

**Como se construyó** (`a86065d6`, `b0330d09`). La ficha trae también `applicationCount`, como la fila. `exchangeDifference` del pago es `null` si ninguna de sus aplicaciones la tiene (§2.4).

#### 4.2.4 `POST /supplier-payments`

```json
{
  "requestId": "019b2c6e-…",
  "companyId": "0199…", "supplierId": "0199…",
  "paymentDate": "2026-11-03", "currency": "PEN",
  "exchangeRate": "1.000000", "exchangeRateDate": "2026-11-03",
  "amount": { "amount": "61056.00", "currency": "PEN" },
  "method": "transferencia", "operationNumber": "OP-99213", "notes": null,
  "applications": [
    { "purchaseInvoiceId": "019a…", "purchaseOrderId": null,
      "amount": { "amount": "13100.00", "currency": "USD" },
      "withheldAmount": null, "withholdingCertificate": null,
      "exchangeRate": "3.760000", "exchangeRateDate": "2026-11-03", "allocations": [] },
    { "purchaseInvoiceId": "019a…", "purchaseOrderId": null,
      "amount": { "amount": "11800.00", "currency": "PEN" },
      "withheldAmount": null, "withholdingCertificate": null,
      "exchangeRate": "1.000000", "exchangeRateDate": "2026-11-03",
      "allocations": [ { "vehicleId": "019a…", "amount": { "amount": "5900.00", "currency": "PEN" }, "settlesUnit": false },
                       { "vehicleId": "019a…", "amount": { "amount": "5900.00", "currency": "PEN" }, "settlesUnit": false } ] }
  ]
}
```

| Campo | Tipo | Obligatorio | Restricción (`400 VALIDATION_ERROR` si no se cumple) |
|---|---|---|---|
| `requestId` | UUID | Sí | Lo genera la pantalla al abrir el formulario (§4.6) |
| `companyId`, `supplierId` | UUID | Sí | — |
| `paymentDate`, `exchangeRateDate` | fecha | Sí | ISO |
| `currency` | texto | Sí | Tres letras |
| `exchangeRate` | decimal | Sí | Mayor que 0, menor que 1 000 000, hasta seis decimales: la tasa que la pantalla mostró |
| `amount` | monto | Sí | Mayor que 0, hasta `999999999999.99`, en `currency` |
| `method` | texto | Sí | `transferencia`, `cheque`, `efectivo` o `saldo_a_favor`, en cualquier mayúscula |
| `operationNumber` | texto | No | De 1 a 100 caracteres después de normalizar |
| `notes` | texto | No | Hasta 1 000 caracteres **(propuesta)** |
| `applications` | arreglo | Sí | De 1 a 100; un elemento `null` responde en `applications[n]` |
| `applications[n].purchaseInvoiceId`, `purchaseOrderId` | UUID | Uno de los dos | Exactamente uno |
| `applications[n].amount` | monto | Sí | Mayor que 0, en la moneda del documento |
| `applications[n].withheldAmount` | monto | No | Desde 0, en la moneda del documento; `null` es 0 |
| `applications[n].withholdingCertificate` | texto | Con retención | Hasta 30, recortado y en mayúsculas |
| `applications[n].exchangeRate`, `exchangeRateDate` | decimal, fecha | Sí | La tasa de la aplicación que la pantalla mostró (§2.4, D-210) |
| `applications[n].allocations` | arreglo | No | A lo sumo 100 por aplicación y 500 en el pago **(propuesta)** |
| `allocations[m].vehicleId` | UUID | Sí | — |
| `allocations[m].amount` | monto | Sí | Mayor que 0, en la moneda del documento |
| `allocations[m].settlesUnit` | booleano | No | `false` por omisión |

**Revisiones, en este orden.** Cada una responde el primer fallo, con `context.applications` cuando nombra aplicaciones (`[{ "application": 2, "documentNumber": "F001-123" }]`, desde 1 en el orden de `applications`) y `context.allocations` cuando nombra imputaciones:
1. Forma: `400 VALIDATION_ERROR`.
2. Sin `pagos.registrar_pago` en ninguna empresa: `403 ACCESS_DENIED`.
3. **El mismo envío:** si ya hay un pago con ese `requestId`, responde `200` con ese pago si el contenido es el mismo, y `409 SUPPLIERPAYMENT_REQUEST_CONFLICT` con `context.paymentId` si no lo es, cuando quien llama lo lee.
4. Empresa: `404 COMPANY_NOT_FOUND` si no existe, no se alcanza o no se lee con `pagos.ver`; `403 ACCESS_DENIED` si `pagos.registrar_pago` no actúa en toda la empresa; `422 COMPANY_INACTIVE`.
5. Proveedor: `404 SUPPLIER_NOT_FOUND` si no es de la empresa o no se lee con `compras.ver`. Un proveedor inactivo sigue recibiendo pagos (D-212).
6. Moneda: `404 CURRENCY_NOT_FOUND`; `422 CURRENCY_INACTIVE`.
7. Montos: `422 SUPPLIERPAYMENT_AMOUNT_SCALE`, también cuando un equivalente redondea a cero.
8. Fecha: `422 SUPPLIERPAYMENT_DATE_IN_FUTURE` (D-212).
9. Tasa del pago: `422 SUPPLIERPAYMENT_EXCHANGE_RATE_MISSING`; `422 SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED`.
10. Método: `422 SUPPLIERPAYMENT_OPERATION_REQUIRED`, `SUPPLIERPAYMENT_OPERATION_FORBIDDEN` (D-212); `409 SUPPLIERPAYMENT_OPERATION_CONFLICT`.
11. Destinos:
    - `422 SUPPLIERPAYMENT_DUPLICATE_TARGET`;
    - `404 PURCHASEINVOICE_NOT_FOUND` o `PURCHASEORDER_NOT_FOUND` si no se lee con `compras.ver`;
    - `422 SUPPLIERPAYMENT_TARGET_MISMATCH` (de otro proveedor);
    - `422 SUPPLIERPAYMENT_INVOICE_ANNULLED`;
    - `422 SUPPLIERPAYMENT_ORDER_NOT_PAYABLE` y `SUPPLIERPAYMENT_DEALER_ORDER` (D-211);
    - `422 SUPPLIERPAYMENT_CURRENCY_PAIR`;
    - `422 SUPPLIERPAYMENT_BALANCE_TARGET` y `SUPPLIERPAYMENT_WITHHOLDING_FORBIDDEN` con saldo a favor;
    - `422 SUPPLIERPAYMENT_WITHHOLDING_CERTIFICATE`;
    - la tasa de cada aplicación: `422 SUPPLIERPAYMENT_EXCHANGE_RATE_MISSING` o `SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED`.
12. Alcance: `403 ACCESS_DENIED` si el permiso no actúa en la marca de una línea, o en todas las marcas cuando hace falta (D-212).
13. `422 SUPPLIERPAYMENT_EXCEEDS_OWED`, con `context.applications[].owed`.
14. `422 SUPPLIERPAYMENT_APPLIED_MISMATCH` si la suma de los equivalentes no es el monto (D-210), con `context.applied` y `context.amount`. Desde `5c77bc8f`, solo cuando la absorción del redondeo no alcanza (§2.4), y `context.applied` es la suma de los equivalentes hacia adelante, sin absorber.
15. `422 SUPPLIERPAYMENT_BALANCE_INSUFFICIENT`, con `context.balance`.
16. Imputaciones:
    - `422 SUPPLIERPAYMENT_ALLOCATION_ON_ORDER`;
    - `422 SUPPLIERPAYMENT_ALLOCATION_REQUIRED`;
    - `422 SUPPLIERPAYMENT_ALLOCATION_UNIT`, también para una unidad que no existe, para no revelar nada;
    - `422 SUPPLIERPAYMENT_ALLOCATION_DUPLICATE`;
    - `422 SUPPLIERPAYMENT_ALLOCATION_MISMATCH`;
    - `422 SUPPLIERPAYMENT_SETTLES_NOT_DEALER`;
    - `422 SUPPLIERPAYMENT_UNIT_SETTLED`;
    - `422 SUPPLIERPAYMENT_ALLOCATION_EXCEEDS_UNIT` (D-211).
17. Guardado: `409 SUPPLIERPAYMENT_OPERATION_CONFLICT` o `SUPPLIERPAYMENT_REQUEST_CONFLICT` en una carrera que la base detiene, sin dejar nada escrito.

Responde `201` con `Location: /api/v1/supplier-payments/{id}` y la ficha (§4.2.3).

**Como se construyó** (`a86065d6` a `ef015ffd`, `063ce0f2`; las decisiones 1, 3, 4, 7 y 9 del responsable del 03/11/2026). El orden queda así, y responde el primer fallo:
1. `400` por lo que el marco resuelve —también más de 100 aplicaciones, más de 100 imputaciones en una, `notes` de más de 1 000 caracteres o un comprobante de más de 30— y `413` por encima del tope;
2. `403 ACCESS_DENIED` sin `pagos.registrar_pago` en ninguna empresa;
3. los `400 VALIDATION_ERROR` del servicio: el método; `amount.currency` distinto de `currency`; el número de operación que no deja nada al normalizarlo; una aplicación sin exactamente un destino, con su retención o sus imputaciones en otra moneda que su monto, o con un comprobante de más de 30 caracteres, en `applications` con `context.applications`; y más de 500 imputaciones;
4. el `requestId`: con el mismo contenido y un pago que quien llama lee, `200`; si no, `409 SUPPLIERPAYMENT_REQUEST_CONFLICT`, con `context.paymentId` solo si lo lee;
5. los candados (§2.7);
6. la empresa (`404`, `403` si el permiso no actúa en toda ella, `422 COMPANY_INACTIVE`), el proveedor (`404`) y la moneda (`404`, `422`);
7. los montos: una aplicación en otra moneda que su documento responde `400 VALIDATION_ERROR` en `applications`, con `context.applications`, y `SUPPLIERPAYMENT_AMOUNT_SCALE` revisa también el equivalente calculado con la tasa enviada;
8. `SUPPLIERPAYMENT_DATE_IN_FUTURE`, con `context.today`;
9. la tasa del pago: `…_EXCHANGE_RATE_MISSING` y después `…_EXCHANGE_RATE_CHANGED`, que lista la del pago, sin `application`, y cada tasa de aplicación que difiere;
10. `…_OPERATION_REQUIRED`, `…_OPERATION_FORBIDDEN` y `409 …_OPERATION_CONFLICT`;
11. los destinos, en el orden de la revisión 11 de arriba, salvo que el comprobante va después de lo que el saldo a favor no admite: `…_DUPLICATE_TARGET`; `404` de la factura o la orden que no se lee —una orden, en la marca de su plantilla—; `…_TARGET_MISMATCH`, `…_INVOICE_ANNULLED`, `…_ORDER_NOT_PAYABLE`, `…_DEALER_ORDER` y `…_CURRENCY_PAIR`; `…_BALANCE_TARGET` y `…_WITHHOLDING_FORBIDDEN`; `…_WITHHOLDING_CERTIFICATE`, una retención sin su comprobante (`422`); y la tasa de cada aplicación;
12. a 17., como arriba: el alcance; `…_EXCEEDS_OWED`; `…_APPLIED_MISMATCH`; `…_BALANCE_INSUFFICIENT`; las imputaciones, con `…_ALLOCATION_MISMATCH` por aplicación y `…_ALLOCATION_EXCEEDS_UNIT` con `lacking` y `currency`; y el `409` de una carrera, sin `context.paymentId`.

Los montos y las tasas de `context` son números JSON, con el código ISO de su moneda al lado, y una llave en `null` no viaja (§4.3). Los límites que la tabla marcaba **(propuesta)** —`notes`, las 100 imputaciones por aplicación y las 500 por pago— los fijó el responsable.

**Como se corrigió (04/11/2026).** El orden no cambia. Cambian dos revisiones y lo que la pantalla ofrece:
- **`…_APPLIED_MISMATCH`** (revisión 14) responde solo si la absorción del redondeo no cuadra el pago, con la suma hacia adelante en `context.applied` (backend `5c77bc8f`, §2.4).
- **El reintento** (revisión 4) compara todo el contenido del pago: también la retención, su comprobante, las imputaciones y las notas (backend `bb355c8e`, §4.6).
- **El alcance** (revisión 12) no cambia en el backend. La sesión dice ahora dónde actúa cada permiso (`companies[].wholeCompanyPermissions` y `companies[].brandPermissions`, convenciones §9.1; backend `df6b7276`), y el formulario lo usa (§6.3): ofrece las empresas donde quien registra lee los pagos y tiene `pagos.registrar_pago` en toda la empresa o en alguna marca, nunca solo por una sede (frontend `fdf14036`, `18e03774`), y retiene «Saldo a favor» a quien solo lo tiene por marca, porque un pago con saldo es de todas las marcas (frontend `64581af4`).
- **Un límite que queda:** lo que se ofrece pagar (§4.2.7) no trae las marcas de las facturas, así que el formulario no puede retener una aplicación a una factura de una marca que el permiso no alcanza. La revisión 12 la rechaza con `403 ACCESS_DENIED`. No es un defecto: el backend es la autoridad.

#### 4.2.5 `POST /supplier-payments/{id}/annulment`

```json
{ "reason": "ERROR_REGISTRO", "updatedAt": "2026-11-03T16:20:00.123Z" }
```

- `reason`: `ERROR_REGISTRO` o `DEVUELTO` (D-212), en cualquier mayúscula; otro valor, `400`.
- `updatedAt`: obligatorio, el que leyó quien anula (D-198).

**Revisiones, en este orden:**
1. Forma: `400`.
2. El pago: `404`. Se bloquea.
3. `pagos.anular_pago` con el alcance que hizo falta para registrarlo: `403` (D-212).
4. `422 SUPPLIERPAYMENT_OUTDATED`.
5. `422 SUPPLIERPAYMENT_ANNULLED`.
6. Con el proveedor bloqueado, `422 SUPPLIERPAYMENT_BALANCE_NEGATIVE` con `context.balances`, `[{ "currency": "PEN", "balance": -400.00 }]`.

**Lo que escribe:**
- `annulled_at`, `annulled_by` (la cuenta de la sesión) y `annulment_reason`. Sus aplicaciones dejan de contar (§10.1) y sus imputaciones también: una unidad saldada por él vuelve a recibir pagos.
- Una fila delicada en la bitácora (D-198, §13.1): `entity = supplier_payments`, `field_name = annulled_at`, `old_value = null`, `new_value` = el momento en ISO, `reason` = el código del motivo, `company_id` = el del pago y `commercial_brand_id` nulo, porque el pago no es de una marca.

Responde `200` con la ficha.

**Como se construyó** (`4ac9730b`; decisiones 2 y 10 del responsable del 03/11/2026). El orden queda así: `400` por lo que el marco resuelve (sin motivo o sin `updatedAt`); `403` sin `pagos.anular_pago` en ninguna empresa; `400` por un motivo fuera de los dos, del servicio; el pago, que bloquea (`404`); el permiso con el alcance de registrarlo (`403`); `SUPPLIERPAYMENT_OUTDATED`; `SUPPLIERPAYMENT_ANNULLED`; y, con el proveedor bloqueado, `SUPPLIERPAYMENT_BALANCE_NEGATIVE` con `context.balances`, `[{currency, balance}]`. La fila de la bitácora es delicada, como la de una factura o una nota.

#### 4.2.6 `GET /suppliers/{id}/balance`

La ruta que convenciones §2.2 ya proponía. Exige leer el proveedor con `compras.ver` (`404`) y `pagos.ver` en su empresa (`403`). Responde cada moneda con movimiento:

```json
{ "supplierId": "0199…", "balances": [
  { "currency": "PEN", "overpaid": { "amount": 400.00, "currency": "PEN" },
    "consumed": { "amount": 400.00, "currency": "PEN" }, "balance": { "amount": 0.00, "currency": "PEN" },
    "owed": { "amount": 5500.00, "currency": "PEN" } } ] }
```

- `overpaid`: lo aplicado de más.
- `consumed`: lo pagado con saldo.
- `balance`: el saldo a favor (§2.2).
- `owed`: la suma de lo que se debe de sus facturas vigentes que todavía deben algo, «Por pagar» en la ficha. No resta las iniciales del canal de dealers.

#### 4.2.7 `GET /suppliers/{id}/payables` (propuesta)

Lo que el formulario ofrece pagar. Mismos permisos que la ruta anterior. `?currency=` filtra por la moneda del documento. Responde, de lo más antiguo a lo más nuevo:

```json
{ "invoices": [
    { "id": "019a…", "documentType": "factura", "documentNumber": "F002-981", "invoiceDate": "2026-10-31",
      "currency": "USD", "exchangeRate": 3.742000, "grossTotal": { "amount": 13100.00, "currency": "USD" },
      "creditNotes": { "amount": 0.00, "currency": "USD" }, "applied": { "amount": 0.00, "currency": "USD" },
      "owed": { "amount": 13100.00, "currency": "USD" }, "unitCount": 1, "settledInitials": null } ],
  "orders": [
    { "id": "019a…", "orderNumber": "OC-LZV-2026-0007", "issueDate": "2026-10-20", "currency": "USD",
      "subtotal": { "amount": 69000.00, "currency": "USD" }, "appliedOnOrder": { "amount": 0.00, "currency": "USD" } } ] }
```

- Las facturas son las vigentes con algo que se debe.
- Las órdenes son las enviadas, salvo con un proveedor de dealers (D-211).
- `settledInitials` es, en un proveedor de dealers, la suma de las diferencias de sus unidades saldadas (§2.3, ejemplo 5).
- `subtotal` es el de la orden (`PurchaseOrder.subtotal`), solo para mostrarlo: no es un tope (D-211).

**Como se construyó** (`3a4b7a33`). Las órdenes son solo las enviadas que quien llama lee, y ninguna con un proveedor de dealers. Las dos listas van por fecha y después por `id`. Un `currency` que no existe deja las dos listas vacías, sin `400`, como un filtro que no nombra nada (convenciones §6.2). Las dos rutas revisan el `403` de la ruta, el proveedor (`404 SUPPLIER_NOT_FOUND`) y `pagos.ver` en su empresa (`403`).

**Como se corrigió (04/11/2026)** (backend `96c62684`; decisión C5 de la revisión, D-197, D-199). Una factura de un proveedor de dealers con todas las unidades de sus líneas vigentes saldadas ya no se ofrece: lo que «debe» son solo las iniciales de los clientes, que Millennium no paga, y ningún pago la alcanza, porque una unidad saldada responde `SUPPLIERPAYMENT_UNIT_SETTLED`. La forma de la respuesta no cambia. Una factura con alguna unidad por pagar sigue, con `settledInitials`. El saldo (§4.2.6) no cambia: su `owed` sigue sumándola, porque no resta las iniciales. Lo prueban `SupplierBalancesIntegrationTests.aDealerOffersNoOrderAndShowsTheInitialsOfItsSettledUnits` y la e2e "a dealer invoice whose units are all settled is no longer offered: what it owes is its customers' down payments (C5)" (frontend `a52c6a71`).

#### 4.2.8 La posición de la factura

**`GET /purchase-invoices/{id}/payment-position` (propuesta),** con `pagos.ver`, para la ficha de la factura y para imputar en el formulario. Responde el ejemplo 2 después del 30/09:

```json
{ "purchaseInvoiceId": "019a…", "currency": "PEN",
  "grossTotal": { "amount": 11800.00, "currency": "PEN" }, "creditNotes": { "amount": 400.00, "currency": "PEN" },
  "applied": { "amount": 11800.00, "currency": "PEN" }, "owed": { "amount": -400.00, "currency": "PEN" },
  "settledInitials": null, "baseDate": "2026-09-15",
  "applications": [
    { "paymentId": "019b…", "paymentDate": "2026-09-15", "method": "transferencia", "operationNumber": "OP1001",
      "status": "VIGENTE", "amount": { "amount": 11800.00, "currency": "PEN" },
      "withheldAmount": { "amount": 0.00, "currency": "PEN" },
      "paymentAmount": { "amount": 11800.00, "currency": "PEN" }, "exchangeDifference": null, "fromOrder": null } ],
  "units": [
    { "lineId": "019a…", "vehicleId": "019a…", "vin": "MD2A11CY6LWA12345",
      "invoicedAmount": { "amount": 5900.00, "currency": "PEN" }, "creditNotes": { "amount": 400.00, "currency": "PEN" },
      "vehicleValue": { "amount": 5500.00, "currency": "PEN" }, "covered": { "amount": 5500.00, "currency": "PEN" },
      "difference": { "amount": 0.00, "currency": "PEN" }, "differenceKind": null,
      "delivered": { "amount": 5500.00, "currency": "PEN" }, "withheld": { "amount": 0.00, "currency": "PEN" },
      "coveredWithBalance": { "amount": 0.00, "currency": "PEN" }, "status": "PAGADA", "settled": false } ] }
```

- `applications` lista también las de pagos anulados, con su estado, para el rastro.
- `fromOrder` nombra la orden de la que pasó la aplicación, leída de la bitácora, o `null`.
- `differenceKind` es `PENDIENTE_DE_PAGO`, `INICIAL_DEL_CLIENTE` o `null` sin diferencia.
- `creditNotes` de cada unidad es su `creditedAmount` de la fase 4, y `vehicleValue`, su `value`: los mismos números que la ficha de la factura.
- `baseDate` es la fecha del primer pago no anulado aplicado a la factura o, sin pago, la de la factura. Desde el 04/11/2026 no es la fecha base de cada unidad: cada una cuenta su brecha desde el primer pago que la cubre, que la unidad responde en `payment.baseDate` (§2.3, §4.2.11). Las unidades de la posición no la traen (backend `c43d1bc1`).

**Como se construyó** (`56f992c8`, `ad7f9983`, `de8f9913`). Cada unidad suma `lacking`, después de `covered`: su valor menos lo imputado, nunca menos de cero, el mismo `UnitCoveragePolicy.lacking` que topa `SUPPLIERPAYMENT_ALLOCATION_EXCEEDS_UNIT`; el backend lo sumó para que la pantalla de imputación tope con lo mismo que el registro. `fromOrder` es el número de la orden, un texto, leído con `ChangeLogHistoryLookup` (§2.1). `applications` va por la fecha de su pago. La ruta revisa el `403` de la ruta, la factura (`404`) y `pagos.ver` en su empresa (`403`).

#### 4.2.9 Lo que cambia en el proveedor (D-197)

- **`POST /suppliers`:** acepta `isDealer`, booleano, `false` por omisión.
- **`PUT /suppliers/{id}`:** lo exige, como los demás campos que reemplaza. Es un cambio incompatible, hecho en los dos repositorios a la vez (convenciones §1).
- **El permiso:** se escribe con `compras.editar_proveedores` en toda la empresa y sin marca, como el resto de la ficha (D-134).
- **La bitácora:** un cambio de `is_dealer` deja su fila, no delicada, porque cambia cómo se registran los pagos a sus facturas. Los pagos ya registrados no cambian.
- **Lo que responde:** `GET /suppliers` y `/suppliers/{id}` responden `isDealer`. `allowedActions` del proveedor suma `registerPayment` al final, si `pagos.registrar_pago` actúa en toda su empresa y quien llama lee con `pagos.ver`.
- **La exportación** suma «Canal de dealers» (Sí/No) después de «Nombre comercial».

**Como se construyó** (`3b463a6d`, `3a4b7a33`; decisión 8 del responsable del 03/11/2026). `registerPayment` del proveedor exige `pagos.registrar_pago` en toda la empresa y en todas sus marcas, porque desde su ficha el formulario puede pagar con saldo o una orden general, y `pagos.ver`. El `PUT` sin `isDealer` responde `400`, y el frontend lo manda desde `605b4343`.

**Como se corrigió (04/11/2026): un proveedor con pagos en sus órdenes no pasa al canal de dealers** (backend `06faa05b`, frontend `7d5e5dce`; D-197, D-211). La orden de un dealer nunca recibe pagos, y los que ya tuviera pasarían a sus facturas sin las unidades que el canal exige. Con el proveedor bloqueado (`FOR UPDATE`, el candado del `PUT`), marcarlo de dealers —solo cuando `isDealer` pasa de falso a verdadero— responde `422 SUPPLIER_DEALER_ORDER_PAYMENTS` si alguna de sus órdenes tiene aplicaciones de pagos no anulados, con `context.orderNumbers`, los números de esas órdenes, ordenados. Va al final de las revisiones del `PUT`: el proveedor (`404`), `compras.editar_proveedores` (`403`), `SUPPLIER_OUTDATED`, los datos generales (`400`) y la dirección (`SUPPLIER_ADDRESS_MISSING`), y después esta. Su texto en el backend es «Una orden del proveedor todavía tiene pagos: no puede ser del canal de dealers hasta que pasen a sus facturas o se anulen». Quitar la marca no se revisa. Lo prueba `SupplierPaymentsIntegrationTests.aSupplierWithPaymentsOnItsOrdersIsNotMarkedOfTheDealerChannel`: con un pago a la orden responde `422` y no marca; anulado el pago, marca. El diálogo del proveedor lo dice nombrando las órdenes (`suppliers.store.spec`, "names the orders with payments that keep a supplier out of the dealer channel (C4)"). Su texto pide vincular sus facturas a esas órdenes, para que los pagos pasen a ellas, o anular esos pagos primero (frontend `89402f13`, merge `a1956861`): una orden con pagos no se anula (PRC-R127), así que lo que se anula son los pagos.

#### 4.2.10 Lo que cambia en la factura y en la orden

- **`GET /purchase-invoices/{id}`:**
  - cada línea suma `paymentStatus` (§2.3), con `compras.ver`, sin montos (D-212);
  - `allowedActions` suma `registerPayment` al final: la factura está vigente, algo se debe, `pagos.registrar_pago` actúa con el alcance de D-212 y quien llama lee con `pagos.ver`.
- **`POST /purchase-invoices` y `PUT /purchase-invoices/{id}`:** el vínculo con una orden lleva el paso de §2.6. La respuesta no cambia; lo que pasó se ve en la posición de la factura y en la bitácora.
- **`POST /purchase-invoices/{id}/annulment`:** con un pago aplicado responde el `422 PURCHASEINVOICE_HAS_DEPENDENTS` de la fase 2 (D-161).
- **`POST /purchase-orders/{id}/annulment`:** responde `422 PURCHASEORDER_HAS_PAYMENTS` si la orden, bloqueada, tiene aplicaciones de pagos no anulados (D-198). También `POST /purchase-orders/{id}/fulfillment` (D-211).
- **`allowedActions` de la orden:** deja de ofrecer `annul` y `fulfill` en ese caso.

**Como se construyó** (`94add284`, `ecc963ec`; decisión 6 del responsable del 03/11/2026). La anulación de la orden responde `PURCHASEORDER_HAS_PAYMENTS` después de `PURCHASEORDER_HAS_CURRENT_INVOICES`, y el cumplimiento, también, los dos bajo el candado de la orden. Su texto es «La orden tiene pagos aplicados: anule los pagos o vincúlela a sus facturas.», en el backend y en la pantalla. Las filas de `GET /purchase-orders` también dejan de ofrecer `fulfill` y `annul`. `PURCHASEINVOICE_HAS_DEPENDENTS` cuenta los pagos con `PaymentInvoiceDependentsAdapter`, y su texto en la pantalla los nombra (frontend `206d9283`).

#### 4.2.11 Lo que cambia en la unidad

`GET /vehicles/{id}`, y la respuesta de cada escritura de la unidad, suman:

```json
"payment": { "status": "PARCIAL", "baseDate": "2026-07-03", "dayGap": 123 }
```

- Viene de `UnitPurchaseFacts.paymentFacts` y de `DayGapPolicy` (§2.8), con `stock.ver`, sin montos (D-212).
- `baseDate` es la de la unidad: el primer pago no anulado que la cubre o, sin ninguno, la fecha de su factura (§2.3). Dos unidades de la misma factura pueden tener fechas distintas, desde el 04/11/2026 (backend `c43d1bc1`).
- Es `null` sin línea vigente.
- La lista de `/vehicles` no lo trae.

**`GET /vehicles/{id}/purchase-cost`** (fase 4, su §4.2.9) suma `payment` dentro de `purchase`, con las mismas revisiones de la fase 4: la unidad (`404`) y `stock.ver_costos` en su empresa y su marca (`403`). Para el auto A del ejemplo 5:

```json
"payment": {
  "status": "SALDADA", "baseDate": "2026-07-03",
  "covered": { "amount": 42000.00, "currency": "PEN" }, "difference": { "amount": 8000.00, "currency": "PEN" },
  "differenceKind": "INICIAL_DEL_CLIENTE",
  "delivered": { "amount": 42000.00, "currency": "PEN" }, "withheld": { "amount": 0.00, "currency": "PEN" },
  "coveredWithBalance": { "amount": 0.00, "currency": "PEN" },
  "payments": [ { "paymentId": "019b…", "operationNumber": "OP2001", "paymentDate": "2026-07-03",
                  "method": "transferencia", "appliedToUnit": { "amount": 42000.00, "currency": "PEN" } } ]
}
```

- Con `purchase` en `null`, sin línea vigente, no hay `payment`.
- `baseDate` es la misma de la ficha de la unidad: la de la unidad, no la de su factura (§2.3).
- `payments` lista los pagos no anulados aplicados a su factura, por fecha y `id`, con lo que le tocó a la unidad: lo imputado, o su parte del fondo (§2.3). Es la columna 42 de la guía sin concatenar.
- Es D-203, que cierra la decisión 7 del mapa: el costo y el pago de la unidad, por el puerto, en vez de en la capa de lectura (`/reports/vehicle-costs`, convenciones §14.19).

**Como se construyó** (`56f992c8`, `1ea73d0d`). `payment` de la unidad viaja en su ficha y en las respuestas de sus escrituras, y en `null` en las filas de `/vehicles`. `purchase.payment` está siempre que la unidad tiene línea vigente. El mismo `56f992c8` corrigió un defecto de la fase 4: la ruta respondía `400` por toda unidad de una factura en la moneda de la empresa ([especificación de notas de crédito](especificacion-notas-de-credito-2026-11-03.md), §8).

### 4.3 Códigos de error

El estado sale del sufijo (convenciones §7.2): `_NOT_FOUND` es `404`, `_CONFLICT` es `409` y los demás propios, `422`. Los textos van en `error.<código-en-kebab>.message` de los dos `messages*.properties` y en `es.json`.

| Código | Estado | Dónde | Mensaje en español |
|---|---|---|---|
| `SUPPLIERPAYMENT_NOT_FOUND` | 404 | Ficha, anulación | Pago no encontrado |
| `SUPPLIERPAYMENT_REQUEST_CONFLICT` | 409 | Alta | Ese envío ya registró otro pago (`context.paymentId`) |
| `SUPPLIERPAYMENT_OPERATION_CONFLICT` | 409 | Alta | Ya hay un pago vigente de ese proveedor con ese número de operación |
| `SUPPLIERPAYMENT_AMOUNT_SCALE` | 422 | Alta | Un monto tiene más decimales de los que permite su moneda, o su equivalente redondea a cero |
| `SUPPLIERPAYMENT_DATE_IN_FUTURE` | 422 | Alta | La fecha del pago no puede ser posterior a hoy (`context.today`) |
| `SUPPLIERPAYMENT_EXCHANGE_RATE_MISSING` | 422 | Alta | No hay tasa registrada para esa moneda en la fecha del pago ni antes (`context.country`, `fromCurrency`, `toCurrency`, `rateType`, `date`) |
| `SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED` | 422 | Alta | La tasa de la fecha cambió mientras se registraba el pago (`context.rates`) |
| `SUPPLIERPAYMENT_OPERATION_REQUIRED` | 422 | Alta | Una transferencia o un cheque lleva su número de operación |
| `SUPPLIERPAYMENT_OPERATION_FORBIDDEN` | 422 | Alta | Un pago con saldo a favor no lleva número de operación |
| `SUPPLIERPAYMENT_DUPLICATE_TARGET` | 422 | Alta | Una factura u orden aparece más de una vez en el pago |
| `SUPPLIERPAYMENT_TARGET_MISMATCH` | 422 | Alta | La factura o la orden es de otro proveedor |
| `SUPPLIERPAYMENT_INVOICE_ANNULLED` | 422 | Alta | La factura está anulada: no recibe pagos |
| `SUPPLIERPAYMENT_ORDER_NOT_PAYABLE` | 422 | Alta | Solo una orden enviada recibe pagos |
| `SUPPLIERPAYMENT_DEALER_ORDER` | 422 | Alta | A un proveedor del canal de dealers se le paga contra sus facturas, unidad por unidad |
| `SUPPLIERPAYMENT_CURRENCY_PAIR` | 422 | Alta | La moneda del pago o la del documento tiene que ser la de la empresa, o las dos iguales |
| `SUPPLIERPAYMENT_EXCEEDS_OWED` | 422 | Alta | Lo aplicado supera lo que se debe de la factura (`context.applications[].owed`) |
| `SUPPLIERPAYMENT_APPLIED_MISMATCH` | 422 | Alta | Lo aplicado no coincide con el monto del pago: un pago se aplica completo (`context.applied`, `context.amount`) |
| `SUPPLIERPAYMENT_BALANCE_INSUFFICIENT` | 422 | Alta | El saldo a favor del proveedor en esa moneda no alcanza (`context.balance`) |
| `SUPPLIERPAYMENT_BALANCE_TARGET` | 422 | Alta | Un pago con saldo a favor solo salda facturas de su misma moneda |
| `SUPPLIERPAYMENT_WITHHOLDING_FORBIDDEN` | 422 | Alta | Un pago con saldo a favor no lleva retención |
| `SUPPLIERPAYMENT_WITHHOLDING_CERTIFICATE` | 422 | Alta | Una retención lleva el número de su comprobante de retención |
| `SUPPLIERPAYMENT_ALLOCATION_ON_ORDER` | 422 | Alta | Las unidades se detallan solo en lo aplicado a una factura |
| `SUPPLIERPAYMENT_ALLOCATION_REQUIRED` | 422 | Alta | Un pago a un proveedor del canal de dealers detalla sus unidades |
| `SUPPLIERPAYMENT_ALLOCATION_UNIT` | 422 | Alta | La unidad no está vigente en esa factura (`context.allocations`) |
| `SUPPLIERPAYMENT_ALLOCATION_DUPLICATE` | 422 | Alta | Una unidad aparece dos veces en lo aplicado a la misma factura |
| `SUPPLIERPAYMENT_ALLOCATION_MISMATCH` | 422 | Alta | Lo imputado a las unidades no coincide con lo aplicado a la factura (`context.allocated`, `context.covered`) |
| `SUPPLIERPAYMENT_SETTLES_NOT_DEALER` | 422 | Alta | Solo una unidad de un proveedor del canal de dealers se declara saldada |
| `SUPPLIERPAYMENT_UNIT_SETTLED` | 422 | Alta | La unidad ya quedó saldada: no recibe más pagos |
| `SUPPLIERPAYMENT_ALLOCATION_EXCEEDS_UNIT` | 422 | Alta | Lo imputado supera lo que le falta a la unidad (`context.allocations[].lacking`) |
| `SUPPLIERPAYMENT_OUTDATED` | 422 | Anulación | El pago cambió desde que se abrió |
| `SUPPLIERPAYMENT_ANNULLED` | 422 | Anulación | El pago está anulado: ya no cambia |
| `SUPPLIERPAYMENT_BALANCE_NEGATIVE` | 422 | Anulación | Sin este pago, el saldo del proveedor quedaría negativo: anule antes el pago con saldo que lo usó (`context.balances`) |
| `PURCHASEORDER_HAS_PAYMENTS` | 422 | Anular o cumplir una orden | La orden tiene pagos aplicados: no se anula |
| `SUPPLIER_DEALER_ORDER_PAYMENTS` | 422 | `PUT /suppliers/{id}` que marca el canal de dealers (desde backend `06faa05b`, §4.2.9) | Una orden del proveedor todavía tiene pagos: no puede ser del canal de dealers hasta que pasen a sus facturas o se anulen (`context.orderNumbers`, ordenados) |

**Reutilizados:**
- `400`: `VALIDATION_ERROR`;
- `403`: `ACCESS_DENIED`;
- `404`: `COMPANY_NOT_FOUND`, `SUPPLIER_NOT_FOUND`, `CURRENCY_NOT_FOUND`, `PURCHASEINVOICE_NOT_FOUND`, `PURCHASEORDER_NOT_FOUND`, `VEHICLE_NOT_FOUND`;
- `422`: `COMPANY_INACTIVE`, `CURRENCY_INACTIVE` y `PURCHASEINVOICE_HAS_DEPENDENTS`.

Las formas de `context` se documentan en `ErrorResource` y en las respuestas `422` de cada operación (convenciones §7.1).

**Como se construyó** (`SupplierPaymentErrorContexts`; decisión 4 del responsable del 03/11/2026). Los textos de la tabla son los del backend. `SUPPLIERPAYMENT_ALLOCATION_MISMATCH` es por aplicación: `context.applications[{application, documentNumber, allocated, covered, currency}]`. Las imputaciones van en `context.allocations[{application, allocation, vehicleId, vin, lacking}]`, con `lacking` y `currency` solo en `…_ALLOCATION_EXCEEDS_UNIT`. `…_EXCHANGE_RATE_MISSING` lleva `{country, fromCurrency, toCurrency, rateType, date, application}`, y `…_EXCHANGE_RATE_CHANGED`, `context.rates[{application, fromCurrency, toCurrency, rate, rateDate}]`, sin `application` en la del pago. `…_APPLIED_MISMATCH` lleva `{applied, amount, currency}`, donde `applied` es, desde `5c77bc8f`, la suma de los equivalentes hacia adelante, sin absorber (§2.4); y `…_BALANCE_INSUFFICIENT`, `{currency, balance}`. Los montos y las tasas son números JSON, con su moneda al lado, y una llave en `null` no viaja. El texto de `PURCHASEORDER_HAS_PAYMENTS` es el de §6.8.

### 4.4 Avisos que se confirman

Ninguno nuevo en el backend.

El aviso de la tasa de un día anterior (D-151) lo muestra el formulario, como en la factura, y no impide seguir. El formulario no deja enviar un pago que no se aplica completo (D-210): muestra lo que falta aplicar.

### 4.5 `allowedActions`

Solo la ficha y las respuestas de las escrituras lo traen; las filas de la lista, `null`. **Como se construyó:** las filas de la lista de órdenes sí lo traen, y sin `annul` ni `fulfill` en una orden con pagos (`94add284`).

**Del pago,** siempre en este orden:

| Acción | Cuándo se ofrece |
|---|---|
| `annul` | Pago vigente, `pagos.anular_pago` con el alcance de D-212, y sin él ningún saldo queda negativo (`SupplierBalancePolicy`) |
| `annulAndCopy` | `annul`, más `pagos.registrar_pago` con el mismo alcance |

**En los demás recursos:**
- **Factura:** `registerPayment`, al final, después de lo que sume la fase 4 (§4.2.10).
- **Proveedor:** `registerPayment` (§4.2.9).
- **Orden:** sin `annul` ni `fulfill` mientras tenga pagos aplicados (D-211).

Todas exigen además leer con el permiso de lectura de su recurso. Lo calcula `SupplierPaymentAllowedActions`.

### 4.6 Versiones y concurrencia

**La idempotencia (D-198):**
1. **El `requestId`.** La pantalla lo genera con `crypto.randomUUID()` al abrir el formulario y lo conserva en cada reintento del mismo envío. «Anular y copiar» abre un formulario nuevo, con un `requestId` nuevo.
2. **El candado.** El servicio toma el candado consultivo de ese `requestId` antes de leer nada (§2.6, paso 3), así que dos envíos iguales a la vez se ordenan: el segundo espera y encuentra el pago del primero.
3. **El mismo contenido** (`sameContentAs`) responde `200` con ese pago, sin escribir nada, y la pantalla abre su ficha.
4. **Otro contenido** responde `409 SUPPLIERPAYMENT_REQUEST_CONFLICT` con el pago que ya existe. Pasa si el primer envío se guardó, su respuesta se perdió y la persona cambió algo.
5. **El último resguardo** es el único `supplier_payments_request_uq`.

**Como se corrigió (04/11/2026): qué es el mismo contenido** (backend `bb355c8e`). La comparación dejaba fuera la retención, su comprobante, las imputaciones y las notas, así que un reintento que los cambiaba respondía el primer pago con `200`, como si se hubiera guardado lo nuevo. Además, un reintento después de que una aplicación a una orden pasara a la factura vinculada (§2.6) ya no coincidía y respondía `409`. Ahora:
- **El contenido** es la empresa, el proveedor, la fecha, la moneda, el monto, el método, el número de operación, las notas y cada destino con su monto, su retención, su comprobante y sus imputaciones: unidad, monto y si la salda.
- **Cada destino pedido** coincide con el destino actual de una aplicación o con la orden de la que pasó, con sus partes sumadas. La orden de origen se lee de la bitácora de la propia aplicación, la fila que escribe `OrderPaymentTransfers`, con `ChangeLogHistoryLookup.latestOldValues`, de `shared`; quien reintenta ya lee el pago, así que esa lectura está autorizada.
- **Las unidades de las imputaciones** se resuelven contra las que el pago imputó, en una lectura por lote: otra unidad no puede ser el mismo contenido.

Lo prueban `SupplierPaymentTests.theWithholdingItsCertificateAndTheAllocationsArePartOfTheContent`, `anApplicationThatPassedToTheInvoiceStillMatchesTheOrderOfTheRequest`; `SupplierPaymentsIntegrationTests.theSameRequestWithAnotherWithholdingAllocationOrNotesAnswersRequestConflict`; y `PurchaseInvoicesIntegrationTests.aRetryAfterTheApplicationToTheOrderPassedToTheInvoiceAnswersThePayment`.

**La tasa vista** (hallazgo A19). El alta lleva la tasa del pago y la de cada aplicación tal como las mostró la pantalla. Si alguna difiere de la vigente, `422 SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED` con las vigentes en `context.rates`; la pantalla las muestra y pide guardar otra vez.

**El candado de `updatedAt`** en la anulación, comparado al milisegundo bajo el candado de la fila (convenciones §3, D-198). La pantalla, ante `SUPPLIERPAYMENT_OUTDATED`, vuelve a leer la ficha.

**La versión de la API** no cambia (convenciones §1):
- `isDealer` en el `PUT` del proveedor es incompatible y se hace en los dos repositorios a la vez;
- lo demás agrega campos y rutas;
- los `422` nuevos de la orden los traduce el frontend en el mismo cambio.

---

## 5. Reglas

Cada regla pasa a ✅ cuando su prueba existe y pasa: todas, desde el 03/11/2026. Las clases de prueba del backend siguen los paquetes de hoy; las del frontend van junto a su archivo. «e2e» es `e2e/supplier-payments.spec.ts`, salvo donde se nombra otro.

### 5.1 `procurement`

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| PRC-R100 | Los tres permisos de pagos se siembran a SA, AE y CO, como la matriz, sin la siembra en dos tiempos de las facturas | §5.6, D-176 | Base: V105, nueve marcas firmadas por `SYSTEM`. `PermissionCodes.Payments`. Frontend: la matriz los pone en Logística › Compras › Pagos a proveedores. Commits: backend `32ff123d`; frontend `278d6bfe` | `SupplierPaymentPermissionsSeedMigrationTests.theThreePermissionsFollowTheMatrix`; `PermissionCatalogConsistencyTests`; `permission-matrix-groups.spec` ("places the three permissions of payments on Logística › Compras › Pagos a proveedores") | ✅ |
| PRC-R101 | Los pagos se leen por empresa con `pagos.ver`. Escribir exige leer: el pago con `pagos.ver`, y las facturas, órdenes y proveedores que nombra con `compras.ver`. La posición de una factura, el saldo y lo que se ofrece pagar, con `pagos.ver`. Lo que se ofrece pagar deja fuera una factura de dealers con todas sus unidades saldadas (C5, 04/11/2026) | §5.6, D-134, D-163, D-197 | `SupplierPaymentQueryServiceImpl` (`companyReach`; `allSettled` en lo que se ofrece pagar), `SupplierPaymentAccessPolicy.sees`, servicio de comandos. Commits: backend `a86065d6`, `b0330d09`, `3a4b7a33`, `56f992c8`, `96c62684`; frontend `a52c6a71` | `SupplierPaymentsIntegrationTests.aWriterWithoutVerDoesNotReadNorWriteThePaymentsOfACompany`, `anInvoiceTheCallerDoesNotReadAnswersNotFound`, `theRecordIsReadWithPagosVerInItsCompany`, `thePaymentPositionIsReadWithPagosVerAndComprasVer`; `SupplierBalancesIntegrationTests.theBalanceIsReadWithComprasVerAndPagosVer`, `aDealerOffersNoOrderAndShowsTheInitialsOfItsSettledUnits`; e2e ("a dealer invoice whose units are all settled is no longer offered: what it owes is its customers' down payments (C5)") | ✅ |
| PRC-R102 | Registrar exige `pagos.registrar_pago` en toda la empresa y en la marca de cada línea de las facturas que paga y de la plantilla de cada orden; con saldo, o con una orden de una plantilla general, en todas las marcas. Anular, `pagos.anular_pago` con el mismo alcance | D-212, D-163, D-178, D-134 | `SupplierPaymentAccessPolicy.registers`, `annuls`; `SupplierPaymentApplications.reach`. Frontend, desde el 04/11/2026 (C8): el formulario ofrece las empresas donde `pagos.registrar_pago` actúa en toda la empresa o en alguna marca, según `wholeCompanyPermissions` y `brandPermissions` de la sesión, y retiene «Saldo a favor» sin toda la empresa; lo que se ofrece pagar no trae marcas, así que una factura de otra marca la rechaza el backend (`403`). Commits: backend `a86065d6`, `4ac9730b`, `df6b7276`; frontend `3742271f`, `fdf14036`, `18e03774`, `64581af4`, `1ef10642` | `SupplierPaymentAccessPolicyTests.aBranchGrantDoesNotPay`, `aBrandGrantPaysOnlyInvoicesOfItsBrand`, `aBrandGrantDoesNotPayWithBalance`; `SupplierPaymentAllowedActionsTests.aBrandGrantDoesNotAnnulAPaymentOfTheWholeCompany`; `iam.store.spec` ("acts on every brand only by a permission of the whole company, never one limited to a brand (C8)"); `supplier-payment-form-view.spec` ("offers only the companies where it reads payments and registers them on the whole company, never from a branch", "holds the method with balance for who registers payments of a brand only, and holds a copy that brings it (C8)"); e2e ("a role limited to a brand pays the documents of its brand, never with balance, which is of every brand", "a session without the scopes offers the payment with balance, and the backend decides") | ✅ |
| PRC-R103 | Las tres tablas llevan `company_id` con llaves compuestas, las políticas de V83 —la de las imputaciones, solo de lectura e inserción—, ningún `DELETE`, y los pagos y las aplicaciones se versionan | §2.5, D-198, D-128 | Base: V107. Commits: backend `fd15e8cf` | `ProcurementCompanyIsolationIntegrationTests.thePaymentsOfAnotherCompanyAreNotSeen`; `RowLevelSecurityMigrationTests.everyTableWithACompanyHasThePoliciesOfV29`, `noTableWithACompanyTakesARowOfAnotherCompanyUnderTheScopeOfOne`; `ProcurementRowsAreNeverDeletedTests.theApplicationCannotDeleteARowOf` (las tres tablas); `JdbcDataVersionsTests.everyVersionedTableHasItsSixtyFourSlotsAndItsTwoTriggers` | ✅ |
| PRC-R104 | Cada aplicación apunta a una factura o a una orden, nunca a las dos, del mismo proveedor y la misma empresa del pago | §10.1, §10.2 | Dominio: `PaymentTarget`. Base: `payment_applications_target_ck`, `payment_applications_supplier_tg`. Servicio: `SUPPLIERPAYMENT_TARGET_MISMATCH`. Commits: backend `fd15e8cf`, `25eb8f69`, `a86065d6` | `SupplierPaymentSchemaTests.anApplicationToAnInvoiceOfAnotherSupplierIsRefusedByTheDatabase`; `SupplierPaymentsIntegrationTests.anInvoiceOfAnotherSupplierAnswersTargetMismatch` | ✅ |
| PRC-R105 | `suppliers.is_dealer` nace en falso; se marca o se quita en la ficha, con `compras.editar_proveedores`, deja su fila en la bitácora y sale en la exportación. Los pagos ya registrados no cambian. Desde el 04/11/2026, no se marca mientras alguna de sus órdenes tenga aplicaciones de pagos no anulados | §9.1, D-197, D-211 | Base: V106. `Supplier`, recursos, `SupplierSpreadsheetColumns`. Servicio: `SupplierCommandServiceImpl`, con el proveedor bloqueado, `422 SUPPLIER_DEALER_ORDER_PAYMENTS` con `context.orderNumbers`. Frontend: la casilla «Canal de dealers» del diálogo del proveedor, que nombra las órdenes del rechazo. Commits: backend `3b463a6d`, `06faa05b`; frontend `605b4343`, `7d5e5dce` | `SuppliersIntegrationTests.aSupplierIsBornOutsideTheDealerChannelAndIsMarkedInItsRecord`, `markingTheDealerChannelLeavesItsRowInTheChangeLog`; `SuppliersExportIntegrationTests.exportsTheSuppliersOfTheListAsAnExcelFile`; `SupplierPaymentsIntegrationTests.aSupplierWithPaymentsOnItsOrdersIsNotMarkedOfTheDealerChannel`; `supplier-form-dialog.spec` ("sends whether the supplier is a dealer"); `suppliers.store.spec` ("names the orders with payments that keep a supplier out of the dealer channel (C4)") | ✅ |
| PRC-R106 | La tasa del pago: en la moneda funcional, 1 con la fecha del pago; en otra, la del tipo de compra del país, la más reciente de la fecha o anterior, que el pago copia. La tasa vista viaja, y un cambio responde `422`, también una tasa que no es 1 en la moneda funcional | D-151, D-152, D-169, A19 | `SupplierPaymentRates`, `fetchEffectiveRate`. Frontend: `LocationFacade.loadPurchaseRate`, una tasa a la vez, y todas otra vez ante `SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED`. Commits: backend `a86065d6`; frontend `2f0d3ef5` | `SupplierPaymentTests.inTheFunctionalCurrencyTheRateIsOneOfItsOwnDate`; `SupplierPaymentsIntegrationTests.aSaturdayPaymentCopiesTheRateOfFriday`, `aRateRegisteredWhileTheFormWasOpenAnswersExchangeRateChanged`; e2e ("without the rate of the day «Registrar tasa» registers it in fixed mode, and the payment and its applications read it") | ✅ |
| PRC-R107 | Una aplicación guarda su monto en la moneda del documento, su tasa y su día —la de venta del día del pago entre las dos monedas, o 1 si son iguales— y su equivalente en la del pago, redondeado a sus decimales; desde el 04/11/2026, el equivalente puede absorber el residuo del redondeo, siempre que al convertirse de vuelta dé su monto (§2.4). Una de las dos monedas es la funcional, o las dos son iguales | D-194, D-210 | `PaymentConversion`, `PaymentEquivalents`, `SupplierPaymentApplications`; `PaymentApplication` rechaza un equivalente que no es admisible. Base: `payment_applications.exchange_rate` y `exchange_rate_date`. Frontend: `absorbedEquivalents` y `admissibleEquivalents`. Commits: backend `3a1596b4`, `a86065d6`, `5c77bc8f`; frontend `20cebe6f` | `PaymentConversionTests.aDollarInvoicePaidInSolesConvertsWithTheRateOfThePaymentDay`, `aSolesInvoicePaidInDollarsDividesByTheRateOfThePayment`, `theSameCurrencyKeepsTheAmountAndTwoForeignOnesAreNotPaired`; `PaymentApplicationTests.theEquivalentConvertsWithTheRateOfTheApplication`, `anAbsorbedEquivalentConvertsBackToTheAmount`; `PaymentEquivalentsTests.anEquivalentIsAdmissibleWhenItConvertsBackToTheAmount`, `v5ADividedEquivalentWithAnEmptyIntervalOnlyAdmitsItsForwardRounding`; `SupplierPaymentsIntegrationTests.theExampleOfADollarInvoicePaidInSoles`, `aDollarPaymentOfAnEuroInvoiceAnswersCurrencyPair`; `supplier-payment-amounts.spec` ("converts the dollars of the example 6 to soles with the rate of the day of the payment") | ✅ |
| PRC-R108 | Cada monto tiene a lo sumo los decimales de su moneda: el del pago, los de la moneda del pago; los de una aplicación y sus imputaciones, los de la moneda del documento. Una aplicación en otra moneda que su documento responde `400` | D-146, §2.2 | `PaymentAmount`, `WithheldAmount`, `SupplierPaymentApplications.amounts`. Commits: backend `25eb8f69`, `a86065d6` | `PaymentAmountTests.anAmountHasAtMostTheDecimalsOfItsCurrency`; `SupplierPaymentsIntegrationTests.aWithholdingInTheCurrencyOfThePaymentAnswersValidationError` | ✅ |
| PRC-R109 | Un pago se aplica completo: la suma de los equivalentes es exactamente su monto. Desde el 04/11/2026, las aplicaciones convertidas absorben el residuo del redondeo, de la última a la primera; si no alcanza, `422` con la suma hacia adelante en `context.applied` | D-194, D-210, §10.1 | Dominio: constructor; `PaymentEquivalents.absorb`. Servicio: `SUPPLIERPAYMENT_APPLIED_MISMATCH`. Frontend: la barra de totales, con los equivalentes absorbidos cuando cuadran, y «Repartir el monto» desde el lado del pago. Commits: backend `25eb8f69`, `a86065d6`, `5c77bc8f`; frontend `94b411de`, `2f0d3ef5`, `20cebe6f`, `af8a3f16` | `SupplierPaymentTests.theAppliedEquivalentsAddUpToTheAmount`, `aPaymentWithARestToApplyIsRefused`; `PaymentEquivalentsTests.v1AnUnreachableAmountInSolesIsReachedByAbsorbingTheResidue`, `v2AResidueBeyondTheIntervalIsRefusedAndOneWithinItMovesDown`, `v3TheLastApplicationAbsorbsFirst`, `v4OnlyApplicationsInTheSameCurrencyAbsorbNothing`, `v6TheResidueSpreadsOverSeveralApplicationsTheLastFirst`; `SupplierPaymentsIntegrationTests.aPaymentWithARestToApplyAnswersAppliedMismatch`, `aPaymentInSolesAbsorbsTheResidueOfTheRounding`; `supplier-payment-amounts.spec` ("tells what remains to apply to the cent", "V1: PEN 10 000,00 for USD 2 659,57 at 3,760000 is reached, though the forward rounding is 9 999,98", "fills USD 2 659,57 of an invoice owing USD 13 100,00 with PEN 10 000,00, and squares (C1)"); `payment-applications-table.spec` ("shows the equivalent once it absorbs the rounding: USD 2 659,57 is PEN 10 000,00, and nothing is left", "keeps the forward rounding while absorbing does not square: PEN 10 000,01 for USD 2 659,57"); e2e ("PEN 10,000.00 squares with USD 2,659.57 at 3.760000: the equivalent absorbs the rounding, and the payment is registered (C1)") | ✅ |
| PRC-R110 | Lo que cubre una aplicación a una factura —su monto más su retención— no supera lo que se debe de ella al aplicarlo, con la factura bloqueada | D-190, D-210, §10.2 | `SupplierPaymentApplications.owed`, `AmountOwedPolicy`, candado 3 de §2.7. Frontend, desde el 04/11/2026: la fila juzga el monto con su retención («Con la retención, supera lo que se debe»), y «Aplicar lo que se debe» llena lo que se debe menos la retención y, en una factura de dealers, menos `settledInitials` (`payableOf`). Commits: backend `a86065d6`, `ef015ffd`, `fbd6f291` (prueba); frontend `c6c96b1e` | `SupplierPaymentsIntegrationTests.anApplicationOverWhatIsOwedAnswersExceedsOwed`, `anAmountPlusItsWithholdingOverWhatIsOwedAnswersExceedsOwed`; `SupplierPaymentLockTests.twoPaymentsOfTheSameInvoiceNeverApplyMoreThanItsOwed`; `payment-application-form.spec` ("refuses an amount whose withholding passes what is owed, as the backend checks it (D-210)", "judges the amount again when the withholding changes or is taken away", "leaves out the down payments of the settled units of a dealer invoice (D-197, example 5)"); `payment-applications-table.spec` ("«Aplicar lo que se debe» fills what is owed less the withholding and, of a dealer, its settled down payments"); e2e ("a row past what it owes says so and holds the payment back; «Aplicar lo que se debe» corrects it", "a withholding covers what is owed with the amount: past it the row says so, «Aplicar lo que se debe» leaves it out, and the payment is sent") | ✅ |
| PRC-R111 | Lo que se debe es el total menos las notas vigentes menos lo aplicado: una nota de una factura impaga baja lo que se debe y no es saldo | D-190, §10.4 | `AmountOwedPolicy`, `PayablePositions`, `PayableFactsReadRepository`. Commits: backend `3a1596b4`, `ca56b0fa`, `a86065d6` | `AmountOwedPolicyTests.aNoteBeforePaymentLowersWhatIsOwedAndCreatesNoBalance` (ejemplo 1); `SupplierPaymentsIntegrationTests.theExampleOfTheNoteBeforeThePayment` | ✅ |
| PRC-R112 | Solo lo aplicado de más, porque la nota llegó después de pagar, es saldo a favor del proveedor en esa moneda | D-190, D-195 | `SupplierBalancePolicy`. Commits: backend `3a1596b4`, `ef015ffd` | `SupplierBalancePolicyTests.aNoteAfterPaymentBecomesTheBalanceOfTheSupplier` (ejemplo 2); `SupplierPaymentsIntegrationTests.theExampleOfTheNoteAfterThePayment` | ✅ |
| PRC-R113 | El saldo es del proveedor y por moneda, se calcula y nunca se guarda | §10.4, D-195 | `SupplierBalancePolicy`, `GET /suppliers/{id}/balance`. Frontend: el panel «Saldo y por pagar» de la ficha del proveedor. Commits: backend `3a1596b4`, `3a4b7a33`; frontend `13526ee1` | `SupplierBalancePolicyTests.theBalanceIsPerCurrency`; `SupplierBalancesIntegrationTests.theBalanceIsPerCurrencyAndMovesWithEveryPaymentAndNote`; `supplier-profile-view.spec` ("shows Saldo y por pagar under the contacts only to who reads payments in its company") | ✅ |
| PRC-R114 | Un pago con saldo consume el saldo de su moneda, calculado con el proveedor bloqueado, y no lo supera; solo salda facturas de esa moneda, no lleva número de operación ni retención, y no apunta a una nota | D-195, A24 | Dominio: `PaymentMethod`. Servicio: `SupplierPaymentApplications`, candado 4. Base: `supplier_payments_operation_by_method_ck`. Frontend: con «Saldo a favor», «Aplicar a» lista solo las facturas de su moneda (revisión 37 de `DESIGN.md`, ítem 9). Commits: backend `ef015ffd`; frontend `91c322c6`, `4e079f12` | `SupplierPaymentsIntegrationTests.aBalancePaymentOverTheBalanceAnswersBalanceInsufficient`, `aBalancePaymentToAnOrderAnswersBalanceTarget`; `SupplierPaymentLockTests.twoBalancePaymentsAtOnceNeverSpendMoreThanTheBalance`; `supplier-payment-form-view.spec` ("lists only the invoices of its currency in a payment with balance, never an order (revision 37, item 9)"); e2e ("with balance: only the currencies with balance, the balance available and only the invoices of that currency") | ✅ |
| PRC-R115 | La retención se registra si el comprobante la trae, con su comprobante; no se calcula. Cuenta como cubierto y no como dinero entregado | D-196, D-210 | Dominio: `PaymentApplication`. Base: `payment_applications_certificate_ck`. Commits: backend `25eb8f69`, `a86065d6` | `PaymentApplicationTests.aWithholdingCountsAsCoveredAndNotAsDelivered`, `aWithholdingNeedsItsCertificate`; `SupplierPaymentsIntegrationTests.aWithholdingWithoutItsCertificateAnswersWithholdingCertificate` | ✅ |
| PRC-R116 | Imputar es opcional: si se imputa, la suma es lo cubierto por la aplicación, cada unidad aparece una vez, es de una línea vigente de esa factura, y solo se imputa lo aplicado a una factura. Una imputación no cambia nunca | §10.3, D-210, D-211 | Dominio: `PaymentApplication`. Base: `payment_allocations_unit_tg`, `payment_allocations_application_vehicle_uq` y `payment_allocations_data_tg`, y la cuenta de la aplicación sin `UPDATE` sobre la tabla. Commits: backend `25eb8f69`, `fd15e8cf`, `063ce0f2` | `PaymentApplicationTests.theAllocationsAddUpToWhatTheApplicationCovers`; `SupplierPaymentSchemaTests.anAllocationOfAUnitOutsideTheInvoiceIsRefusedByTheDatabase`, que prueba también que no se edita; `SupplierPaymentsIntegrationTests.theAllocationsNameUnitsOfTheInvoiceOnceAndAddUpToWhatItCovers`, `theAllocationsOfAnApplicationWithAWithholdingAddUpToWhatItCovers` (backend `fbd6f291`: con retención, las imputaciones suman monto más retención, y cada unidad muestra su parte de la retención) | ✅ |
| PRC-R117 | Con un proveedor del canal de dealers, toda aplicación a una factura imputa sus unidades, y cada imputación puede declarar la unidad saldada; con otro proveedor, ninguna la declara | D-197, §10.3 | `SupplierPaymentAllocations`. Frontend: el diálogo de imputación, con «Salda la unidad». Commits: backend `063ce0f2`; frontend `91c322c6` | `SupplierPaymentsIntegrationTests.theExampleOfTheDealerChannel`, `aDealerInvoicePaidWithoutUnitsAnswersAllocationRequired`, `aNormalSupplierCannotSettleAUnit`; e2e ("a dealer invoice says its units are missing; the dialog allocates them, settles one and squares") | ✅ |
| PRC-R118 | Una unidad saldada no recibe más pagos; anular el pago que la saldó la libera | D-197 | `SupplierPaymentAllocations`, bajo el candado de su factura. Commits: backend `063ce0f2`, `4ac9730b` | `SupplierPaymentsIntegrationTests.aSettledUnitAnswersUnitSettled`, `annullingThePaymentThatSettledAUnitFreesIt`, que desde backend `fbd6f291` anula por la ruta de la anulación y no por SQL, y comprueba antes el `422 SUPPLIERPAYMENT_UNIT_SETTLED` y después la unidad `SIN_PAGO` en la posición | ✅ |
| PRC-R119 | Lo imputado a una unidad no supera lo que le falta para su valor: su valor menos lo ya imputado, nunca menos de cero, que la posición de la factura dice en `lacking` | D-211 | `SupplierPaymentAllocations`, `UnitCoveragePolicy.lacking`. Frontend: «Le falta» del diálogo de imputación lee `lacking`, como tope y como peso de «Completar» y «Repartir». Commits: backend `063ce0f2`, `de8f9913`; frontend `a214ec87`, `8edfa99b` | `SupplierPaymentsIntegrationTests.anAllocationOverWhatTheUnitLacksAnswersAllocationExceedsUnit`, `eachUnitOfThePositionTellsWhatItLacksAsTheRegistrationCapsIt`; `UnitCoveragePolicyTests.whatAUnitLacksIsItsValueMinusItsAllocationsNeverBelowZero`; `payment-allocations-dialog.spec` ("caps a unit at what is not allocated to it yet, not at its value less what covers it (D-211)", "«Repartir» weighs each unit by what it lacks: none to a unit already allocated in full"); e2e ("a unit covered by money applied without allocations still takes what it lacks: its value less what is allocated to it (D-211)") | ✅ |
| PRC-R120 | Lo aplicado sin imputar se reparte entre las unidades no saldadas de la factura en proporción a lo que le falta a cada una, que sin imputaciones es su valor; los céntimos del redondeo se mueven de a uno desde la primera unidad que recibe parte del reparto, sin pasar lo que le falta ni bajar de cero (C2, 04/11/2026); lo que no cabe es saldo | §10.3, D-199, D-211 | `UnitCoveragePolicy`. Frontend: `supplier-payment-amounts.ts`, con `BigInt` y la mitad hacia arriba. Commits: backend `3a1596b4`, `46f207b1`; frontend `94b411de`, `f8efb41b` | `UnitCoveragePolicyTests.withoutAllocationsTheApplicationGoesInProportionToTheValue` (ejemplo 3), `aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf` (ejemplo 4), `theCentsLeftGoToTheFirstUnit`, `theCentsLeftNeverTakeAUnitBelowZeroNorAboveWhatItLacks`; `supplier-payment-amounts.spec` ("spreads in proportion with the cents on the first unit", "reads the example 3: 48 000,00 over 20 000,00 and 30 000,00 cover 19 200,00 and 28 800,00", "C2: 0,25 over 50 units lacking 100,00 each: the first 25 give their cent back, never below zero", "adds a missing cent to the first unit that still lacks one, never past what it lacks") | ✅ |
| PRC-R121 | Una unidad tiene cinco montos, y solo el valor de factura se guarda: notas, valor del vehículo, cubierto y diferencia se calculan en cada lectura. La diferencia es pendiente de pago, o la inicial del cliente en una unidad saldada. El estado es `SIN_PAGO`, `PARCIAL`, `PAGADA` o `SALDADA`. Se ven con `pagos.ver` en la factura y con `stock.ver_costos` en la unidad, por la ruta del costo de la fase 4; el estado, sin montos, con `compras.ver` en la factura | D-199, D-197, §9.6, D-203, D-212 | `UnitCoveragePolicy`, `PayablePositions`, `InvoicePaymentPositions`; `GET /purchase-invoices/{id}/payment-position` y `GET /vehicles/{id}/purchase-cost`. Frontend: el panel «Pagos» de la ficha de la factura y la sección «Pago» de «Compra y costo». Commits: backend `3a1596b4`, `56f992c8`; frontend `0f96e2a1`, `f2e3b686` | `UnitCoveragePolicyTests.theFiveAmountsOfADealerUnitSettledByItsInitial` (ejemplo 5); `SupplierPaymentsIntegrationTests.thePaymentPositionOfAnInvoiceAfterANoteThatCameAfterPaying`, `theRecordOfAnInvoiceTellsThePaymentOfItsUnitsAndOffersToPayIt`; `VehiclePurchaseCostIntegrationTests.thePurchaseCostOfAUnitCarriesItsPayment`, `aReaderWithoutCostsGetsAccessDenied`; `vehicle-purchase-cost-panel.spec` ("continues with «Pago»: the value less what covers it, what is still to pay, and its payments one a row"); e2e ("the invoice record of who does not read payments shows the status of each line without amounts nor the panel") | ✅ |
| PRC-R122 | El dinero entregado es lo cubierto menos la retención y menos lo pagado con saldo | D-199 | `UnitCoveragePolicy`. Commits: backend `3a1596b4` | `UnitCoveragePolicyTests.aUnitCoveredWithBalanceHasNoDeliveredMoney` (ejemplo 2), `whatTheUnitsDoNotLackIsTheBalanceAndAWithholdingIsNotDelivered` | ✅ |
| PRC-R123 | La fecha base de la brecha de una unidad es la del primer pago no anulado que cubre parte de ella, por su imputación o por su parte de lo que su factura no imputó, también uno que pasó de la orden; mientras ninguno la cubre, la de la factura. Desde el 04/11/2026 es de cada unidad y no de su factura: de 50 motos pagadas en dos bloques de 25, cada bloque cuenta desde su propio pago. La posición de la factura conserva el primer pago aplicado a la factura. `procurement` la da a `fleet` por `UnitPurchases`, y nunca se guarda | D-199, D-27, §10.5; guía, columnas 24 y 43 | `UnitCoveragePolicy` (`UnitCoverage.baseDate`), `PurchaseInvoiceUnitPurchasesAdapter.paymentFacts` y `purchaseCostOf`. Commits: backend `3a1596b4`, `56f992c8`, `c43d1bc1` (merge `1bd767f1`) | `UnitCoveragePolicyTests.eachUnitCountsItsGapFromTheFirstPaymentThatCoversIt`, `aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf` (ejemplo 4, con la fecha de cada bloque), `withoutPaymentsTheBaseDateIsTheDateOfTheInvoice`; `PurchaseInvoiceUnitPurchasesAdapterTests.theBaseDateMovesToTheFirstPaymentWhenItIsRegistered`, `aPaymentFromTheOrderCountsWithItsDate`; `VehiclesIntegrationTests.eachUnitOfAnInvoicePaidInTwoBlocksCountsItsGapFromItsOwnPayment` | ✅ |
| PRC-R124 | La diferencia de cambio de una aplicación de dinero a una factura en moneda extranjera se informa aparte, en la moneda funcional, y nunca toca el costo; con saldo es cero, y en una orden o en un documento en la moneda funcional no hay | D-194, D-213 | `ExchangeDifferencePolicy`, que desde el 04/11/2026 lee el equivalente guardado, con lo que absorbió (§2.4); la ficha y la exportación. Commits: backend `3a1596b4`, `a86065d6`, `b0330d09`, `5c77bc8f` | `ExchangeDifferencePolicyTests.theExampleOfADollarInvoicePaidInSoles` (ejemplo 6), `aPaymentInTheSameForeignCurrencyTakesTheRateOfThePayment`, `withTheBalanceItIsZeroAndWithoutAForeignInvoiceThereIsNone`, `theDifferenceReadsTheEquivalentThatAbsorbedTheResidue`; `SupplierPaymentsIntegrationTests.aPaymentInSolesAbsorbsTheResidueOfTheRounding` (47,89); `SupplierPaymentsIntegrationTests.theExampleOfADollarInvoicePaidInSoles`; `supplier-payment-amounts.spec` ("reads the loss of the example 6: PEN 235,80"); `SupplierPaymentsIntegrationTests.paymentsNeverChangeTheValueOfAUnit` (backend `8c1a8c9f`): una factura en dólares pagada en soles, con imputación y sin ella, deja igual el costo de cada unidad, los valores de la ficha de la factura y los de su posición de pago, porque el costo de la unidad lee solo su línea y sus notas (`UnitPurchaseCostCalculator`) | ✅ |
| PRC-R125 | Una orden recibe pagos solo enviada, y nunca de un proveedor del canal de dealers | D-211, D-197 | `SupplierPaymentApplications`. Desde el 04/11/2026, la ficha del proveedor la sostiene también al revés: no se marca de dealers con pagos en sus órdenes (PRC-R105, `SUPPLIER_DEALER_ORDER_PAYMENTS`). Commits: backend `a86065d6`, `06faa05b` | `SupplierPaymentsIntegrationTests.aDraftOrderAnswersOrderNotPayable`, `anOrderOfADealerAnswersDealerOrder`, `aSupplierWithPaymentsOnItsOrdersIsNotMarkedOfTheDealerChannel` | ✅ |
| PRC-R126 | Cuando una factura se vincula a una orden con pagos aplicados, sus aplicaciones pasan solas a la factura, de la más antigua a la más nueva, hasta lo que se debe; lo que sobra queda en la orden, el pago no se toca y cada paso deja su fila en la bitácora. En otra moneda no pasa nada, y desvincular no devuelve | D-198, D-211 | `OrderPaymentTransfers`, `OrderPaymentTransferPolicy`, desde `PurchaseInvoiceOrderLinks`. Base: `payment_applications_data_tg`. La orden de la que pasó cada aplicación se lee de la bitácora con `ChangeLogHistoryLookup.latestOldValues`, de `shared`. Commits: backend `3a1596b4`, `68528670`, `ad7f9983` | `OrderPaymentTransferPolicyTests.theOldestApplicationsMoveFirstUpToWhatIsOwed` (ejemplo 7); `PaymentApplicationTests.anApplicationToAnOrderGivesPartOfItselfToAnInvoice`; `SupplierPaymentAuditTests.anApplicationToAnOrderGivesPartOfItselfToAnInvoiceWithoutMovingItsPayment`; `PurchaseInvoicesIntegrationTests.registeringAnInvoiceOfAPaidOrderMovesThePaymentToTheInvoice`, `relinkingMovesOnlyWhatTheNewInvoiceOwes`, `unlinkingGivesNothingBackToTheOrder`, `theChangeLogWithholdsTheAmountOfAnApplicationFromWhoDoesNotReadThePayments` (backend `ad87b4d9`: la fila `amount` se ve solo con `pagos.ver`, C6); `ChangeLogHistoryLookupTests.theNewestOldValueOfTheFieldOfEachEntityIsRead`; `ChangeLogAccessPolicyTests.theValuesOfAConfidentialFieldTravelWithItsPermissionInTheCompanyOfTheRow`; `change-logs-view.spec` ("says why each value is withheld by its field: the costs of a unit, a payment, or another reach (C6)") | ✅ |
| PRC-R127 | Una orden con pagos aplicados no se anula ni se cumple, y su `allowedActions`, en la ficha y en cada fila de la lista, no lo ofrece | D-198, D-211 | `PurchaseOrderCommandServiceImpl`: la anulación, después de las facturas vigentes, y el cumplimiento, bajo el candado de la orden; `PurchaseOrderAllowedActions`, `PurchaseOrderViewFactory`. Frontend: la ficha de la orden la vuelve a leer ante `PURCHASEORDER_HAS_PAYMENTS`. Commits: backend `94add284`; frontend `0c10fa14` | `PurchaseOrdersIntegrationTests.anOrderWithAPaymentIsNotAnnulled`, `anOrderWithAPaymentIsNotFulfilled`; `SupplierPaymentLockTests.anAnnulmentAndAPaymentOfTheSameOrderDoNotCross`, `aFulfilmentAndAPaymentOfTheSameOrderDoNotCross` (backend `29db26be`); `purchase-orders.store.spec` ("says an order with payments applied is neither annulled nor fulfilled, and leaves it as it was (D-211)"); e2e de órdenes ("offers neither Anular nor Cumplir on a sent order with payments applied (D-211)", "says an order paid meanwhile is neither annulled nor fulfilled, and offers neither once it reads it again") | ✅ |
| PRC-R128 | Una factura con un pago aplicado no se anula: el puerto de dependientes de la fase 2 lo dice para los pagos | D-161, H3 | `PaymentInvoiceDependentsAdapter`. Commits: backend `ecc963ec` | `PurchaseInvoiceDependentsTests.creditNotesAndPaymentsAreTheDependents`, `anInvoiceWithAPaymentHasDependents`; `SupplierPaymentLockTests.anInvoiceAnnulmentAndAPaymentOfItDoNotCross` | ✅ |
| PRC-R129 | Un pago enviado dos veces se guarda una sola: el mismo `requestId` con el mismo contenido responde el pago que existe, y con otro contenido, `409`. Desde el 04/11/2026, el contenido incluye la retención, su comprobante, las imputaciones y las notas, y un destino coincide también con la orden de la que pasó la aplicación | D-198 | `SupplierPaymentRequests`, candado consultivo, `ChangeLogHistoryLookup` para la orden de origen; `SupplierPayment.sameContentAs`; base: `supplier_payments_request_uq`. Frontend: el formulario conserva su `requestId` en cada reintento. Commits: backend `25eb8f69`, `5ff0c76d`, `a86065d6`, `bb355c8e`; frontend `2f0d3ef5` | `SupplierPaymentTests.theSameRequestSaysTheSameContent`, `theWithholdingItsCertificateAndTheAllocationsArePartOfTheContent`, `anApplicationThatPassedToTheInvoiceStillMatchesTheOrderOfTheRequest`; `SupplierPaymentsIntegrationTests.theSameRequestTwiceAnswersTheSamePayment`, `theSameRequestWithAnotherAmountAnswersRequestConflict`, `theSameRequestWithAnotherWithholdingAllocationOrNotesAnswersRequestConflict`; `PurchaseInvoicesIntegrationTests.aRetryAfterTheApplicationToTheOrderPassedToTheInvoiceAnswersThePayment`; `SupplierPaymentLockTests.twoIdenticalRequestsAtOnceLeaveOnePayment`; `SupplierPaymentAuditTests.aRequestTheDatabaseStopsIsRaisedAsItsNaturalKey`; `supplier-payment-form-view.spec` ("sends the same request again on a retry, so a double click saves one payment (§4.6)"); e2e ("a send whose answer was lost is sent again with the same request, and opens the payment it saved (200)") | ✅ |
| PRC-R130 | El número de operación se guarda normalizado y no se repite en el proveedor entre los pagos no anulados; anular uno libera su número | D-198, §10.1, convenciones §5.3 | `OperationNumber`. Base: `supplier_payments_operation_uq`. Adaptador: `NaturalKey.SUPPLIER_PAYMENT_OPERATION` → `409`. Frontend: `operation-number.ts`; el formulario lo manda normalizado. Commits: backend `25eb8f69`, `5ff0c76d`, `a86065d6`; frontend `94b411de` | `OperationNumberTests.aHyphenAndASpaceAreTheSameNumber`; `SupplierPaymentsIntegrationTests.theSameVoucherOfTheSupplierAnswersOperationConflict`, `anAnnulledPaymentFreesItsVoucher`, que desde backend `fbd6f291` anula por la ruta de la anulación y no por SQL, y comprueba antes el `409`; `SupplierPaymentLockTests.twoPaymentsWithTheSameVoucherLeaveOne`; `SupplierPaymentAuditTests.aNumberOfOperationTheDatabaseStopsIsRaisedAsItsNaturalKey`; `operation-number.spec` ("reads a hyphen, a space and a dot as the same number, in capitals (PRC-R130)") | ✅ |
| PRC-R131 | La transferencia y el cheque llevan número de operación; el efectivo, si lo tiene; el saldo a favor, ninguno | D-212 | Dominio: constructor. Base: `supplier_payments_operation_by_method_ck`. Commits: backend `25eb8f69`, `a86065d6` | `SupplierPaymentTests.aTransferWithoutItsVoucherIsRefused`, `aBalancePaymentWithAVoucherIsRefused`; `SupplierPaymentsIntegrationTests.aTransferWithoutItsVoucherAnswersOperationRequired` | ✅ |
| PRC-R132 | La fecha del pago no es futura en la zona de la empresa, y puede ser anterior a la factura | §10.1, D-212 | `InvoiceDatePolicy.isFuture` (fase 2), con «hoy» de `ProcurementCalendar` desde el 04/11/2026, en lugar del `Clock.systemUTC()` fijo del servicio. Frontend: el calendario, hasta hoy. Commits: backend `a86065d6`, `6165ba5d`; frontend `2f0d3ef5` | `ProcurementDatesInLimaTests.aPaymentDatedTomorrowInLimaIsRefused`, con el reloj fijo en el 2026-11-04T01:00Z, las 20:00 en Lima, ya el día siguiente en UTC (backend `6165ba5d`; antes en `SupplierPaymentsIntegrationTests`, a la hora en que corría); `SupplierPaymentsIntegrationTests.aPaymentBeforeItsInvoiceIsAccepted` | ✅ |
| PRC-R133 | Anular exige `pagos.anular_pago`, un motivo y el `updatedAt` leído; es una sola vez, sus aplicaciones e imputaciones dejan de contar, y deja su fila delicada en la bitácora | D-198, §10.1, §13.1 | Dominio: `annul`. Servicio: `SupplierPaymentAnnulments`. Base: `supplier_payments_annulment_ck`, `supplier_payments_data_tg`. Frontend: «Anular» y «Anular y copiar» de la ficha; desde el 04/11/2026, la copia dice lo que no pudo copiar, revisa cada imputación contra lo que le falta hoy a la unidad, deja «Salda la unidad» solo con un dealer y no copia el número de operación de un pago vigente. Commits: backend `25eb8f69`, `fd15e8cf`, `4ac9730b`; frontend `4c33bc63`, `1e189bf0` | `SupplierPaymentTests.aTargetAppearsOnceAndAnAnnulledPaymentIsAnnulledOnce`; `SupplierPaymentsIntegrationTests.annullingNeedsItsPermissionAReasonAndTheReadMoment`, `theAnnulmentLeavesItsRowInTheChangeLog`; `SupplierPaymentSchemaTests.anAnnulledPaymentNeverChanges`; `supplier-payment-form-view.spec` ("drops a document no longer owed and says which", "caps an allocation past what the unit lacks now, drops one that lacks nothing, and «Salda» outside a dealer", "keeps out the operation number of a payment still in force, and says so", "copies the operation number of an annulled payment"); e2e ("the record annuls with a reason; a negative balance is explained; «Anular y copiar» opens the copy with another request") | ✅ |
| PRC-R134 | Un pago no se anula si, sin él, el saldo del proveedor en alguna moneda quedaría negativo, calculado con el proveedor bloqueado | D-198 | `SupplierPaymentAnnulments`, `SupplierBalancePolicy`, candado 4. Commits: backend `4ac9730b` | `SupplierPaymentsIntegrationTests.annullingThePaymentWhoseOverpaymentWasSpentAnswersBalanceNegative` (ejemplo 2); `SupplierPaymentLockTests.anAnnulmentAndABalancePaymentAtOnceNeverLeaveANegativeBalance` | ✅ |
| PRC-R135 | La anulación de una nota pregunta a los pagos si el saldo quedaría negativo sin ella, con el proveedor bloqueado; los pagos son la única respuesta | D-193, §2.8 | `PaymentSupplierBalanceAdapter`, que implementa `SupplierBalanceAfterAnnulment` (fase 4, su PRC-R86). Commits: backend `64339927` | `PaymentSupplierBalanceAdapterTests.annullingTheNoteWhoseBalanceWasSpentIsRefused` (ejemplo 2); `SupplierBalanceAfterAnnulmentTests.thePaymentsAreTheOnlyGuardOfTheBalance` | ✅ |
| PRC-R136 | Las escrituras de pagos toman los candados en el orden de §2.7, y una carrera que la base detiene responde el código de su llave | Esta especificación, §2.7 | Servicio y adaptadores. Commits: backend `ef015ffd`, y los de cada escritura | `SupplierPaymentLockTests.twoPaymentsOfTheSameInvoiceNeverApplyMoreThanItsOwed`, `twoBalancePaymentsAtOnceNeverSpendMoreThanTheBalance`, `twoIdenticalRequestsAtOnceLeaveOnePayment`, `twoPaymentsWithTheSameVoucherLeaveOne`, `anInvoiceAnnulmentAndAPaymentOfItDoNotCross`, `anAnnulmentAndABalancePaymentAtOnceNeverLeaveANegativeBalance`, `anAnnulmentAndAPaymentOfTheSameOrderDoNotCross`, `aFulfilmentAndAPaymentOfTheSameOrderDoNotCross` | ✅ |
| PRC-R137 | Un pago en una moneda, de cualquier estado, fija sus decimales | §4.1, D-138 | `SupplierPaymentCurrencyUsageAdapter`. Commits: backend `ecc963ec` | `CurrencyDecimalPlacesLockTests.aPaymentFixesTheDecimalPlacesOfItsCurrency` | ✅ |
| PRC-R138 | La exportación lleva una fila por aplicación con las 26 columnas de §4.2.2, la diferencia de cambio rotulada «(calculada)», y tiene el alcance de `pagos.ver` | D-106, D-194 | `SupplierPaymentsController`, `SupplierPaymentSpreadsheetColumns`, `SupplierPaymentViewFactory.toExportRows`, `ProcurementDocumentKinds`. Frontend: «Exportar» de la lista. Commits: backend `b0330d09`; frontend `4c33bc63` | `SupplierPaymentsExportIntegrationTests.thePaymentsAreExportedOneRowPerApplication`, `aWriterWithoutVerInBDoesNotGetBInTheExport`; e2e ("the list filters by company, supplier, method, status and period, and exports with the filters of the screen") | ✅ |
| PRC-R139 | `allowedActions` del pago sale en el orden `annul`, `annulAndCopy`; la factura y el proveedor suman `registerPayment`, y la orden con pagos deja de ofrecer `annul` y `fulfill` | D-115 | `SupplierPaymentAllowedActions`, `PurchaseInvoiceAllowedActions`, `SupplierViewFactory`, `PurchaseOrderAllowedActions`. Commits: backend `b0330d09`, `3a4b7a33`, `56f992c8`, `94add284`; frontend `4c33bc63`, `13526ee1`, `0f96e2a1` | `SupplierPaymentAllowedActionsTests.theActionsComeInTheOrderOfTheSpecification`, `anAnnulmentThatWouldLeaveANegativeBalanceIsNotOffered`, `anAnnulledPaymentOffersNothing`; `SupplierBalancesIntegrationTests.theRecordOfTheSupplierOffersRegisterPaymentOnTheWholeCompany`; `SupplierPaymentsIntegrationTests.theRecordOfAnInvoiceTellsThePaymentOfItsUnitsAndOffersToPayIt`; `PurchaseOrdersIntegrationTests.anOrderWithAPaymentIsNotAnnulled`; `supplier-profile-view.spec` ("offers Registrar pago before Editar when its allowedActions says so, with the supplier and its company") | ✅ |

### 5.2 `fleet`

Siguen en [contextos/fleet.md](contextos/fleet.md) a las de la fase 4, FLT-R94 a FLT-R99, y a las de las cargas de archivos, FLT-R100 a FLT-R126.

| Id | Regla | Fuente | Dónde se aplica | Pruebas | Estado |
|---|---|---|---|---|---|
| FLT-R130 | `fleet` sabe el estado de pago y la fecha base de una unidad solo por `UnitPurchases.paymentFacts`, que responde un solo bean, y no depende de `procurement` | Mapa §5, regla 1.1, FLT-R53 | `fleet/interfaces/acl`, `UnitPurchaseFacts`. Commits: backend `56f992c8`, `1ea73d0d` | `ModularityTests`; `UnitPurchasesWiringTests.procurementIsTheOnlyAnswerOfThePaymentOfAUnit`, en `procurement/infrastructure/fleet` | ✅ |
| FLT-R131 | La ficha de la unidad trae su estado de pago y su brecha de días: desde la fecha base de la unidad —el primer pago que la cubre o, sin ninguno, su factura (PRC-R123)— hasta la primera entrega, el primer paso a Activo, o hasta hoy en la zona de la empresa; se recalcula en cada lectura y no trae montos | D-199, D-27, FLT-R58, convenciones §9.2 | `DayGapPolicy`, `VehicleViewFactory`. Frontend: la insignia del estado de pago después de «Estado», y la brecha, «1 día» o «N días», con el signo menos si es negativa. Commits: backend `1ea73d0d`, `c43d1bc1` (merge `1bd767f1`: la fecha base de cada unidad); frontend `f2e3b686` | `DayGapPolicyTests.aDeliveredUnitCountsUpToItsFirstDelivery`, `anUndeliveredUnitCountsUpToTodayInTheZoneOfTheCompany`, `aUnitPaidAfterItsDeliveryHasANegativeGap`; `VehiclesIntegrationTests.theRecordOfAUnitCarriesItsPaymentStatusAndGapWithoutAmounts`, `eachUnitOfAnInvoicePaidInTwoBlocksCountsItsGapFromItsOwnPayment`; `vehicle-profile-view.spec` ("shows the payment status after Estado and the gap in days with what it counts, without amounts", "shows neither the payment status nor the gap of a unit without a current invoice line"); e2e de flota ("the record of a unit says its payment status and its gap in days, and «Compra y costo» what its payments cover") | ✅ |
| FLT-R132 | El estado y la brecha se ven con `stock.ver`, donde se ve la unidad, sin montos | D-212, D-102 | `VehicleQueryServiceImpl`. Commits: backend `1ea73d0d`; frontend `f2e3b686` | `VehiclesIntegrationTests.anOperatorSeesThePaymentStatusOfAUnitWithoutItsAmounts`; e2e ("the record of a unit without stock.ver_costos says its payment status and its gap, without «Compra y costo»") | ✅ |

**Como se construyó.** Los nombres de las dos tablas son los reales desde la versión 3, con el commit de cada regla. Cambiaron frente a la versión 2:
- `VehiclePurchaseCostIntegrationTests.withoutViewCostsThePurchaseCostAnswersAccessDenied` (PRC-R121) no se escribió: lo prueba la de la fase 4, `aReaderWithoutCostsGetsAccessDenied`, en la misma ruta.
- `SupplierPaymentsIntegrationTests.paymentsNeverChangeTheValueOfAUnit` (PRC-R124) no se escribió con la fase: se agregó después del cierre, en backend `8c1a8c9f`, con `SupplierPaymentLockTests.aFulfilmentAndAPaymentOfTheSameOrderDoNotCross` (PRC-R127, `29db26be`), la carrera del cumplimiento que faltaba junto a la de la anulación. Las dos entraron en `develop` con `059a4457` el 03/11/2026 y no encontraron defectos.
- La posición de la factura y la ficha de la factura con sus pagos se prueban en `SupplierPaymentsIntegrationTests`, y el saldo, en `SupplierBalancesIntegrationTests` (PRC-R101, PRC-R113, PRC-R121, PRC-R139).
- `SupplierPaymentLockTests` vive en `procurement/infrastructure/persistence`, y tiene siete carreras, dos más que la versión 2: el mismo voucher en dos pagos y la anulación de una factura frente a un pago a ella (PRC-R136). `UnitPurchasesWiringTests` vive en `procurement/infrastructure/fleet` (FLT-R130).
- Se sumaron, entre otras, `SupplierPaymentsIntegrationTests.theAllocationsNameUnitsOfTheInvoiceOnceAndAddUpToWhatItCovers`, `eachUnitOfThePositionTellsWhatItLacksAsTheRegistrationCapsIt` y `theRouteAnswersBeforeTheServiceAndTheShapeBeforeTheRequest`, que prueba el orden de §4.1 y §4.2.4; y `ChangeLogHistoryLookupTests.theNewestOldValueOfTheFieldOfEachEntityIsRead`, de `shared` (PRC-R126).

**Como se corrigió (04/11/2026).** Las tablas citan las pruebas de la revisión. Lo que cambió de nombre o de forma:
- **PRC-R132:** `SupplierPaymentsIntegrationTests.aPaymentDatedTomorrowInLimaIsRefused` pasó a `ProcurementDatesInLimaTests.aPaymentDatedTomorrowInLimaIsRefused`, con la factura y la nota del mismo caso, a una hora fija (backend `6165ba5d`).
- **PRC-R118 y PRC-R130:** `annullingThePaymentThatSettledAUnitFreesIt` y `anAnnulledPaymentFreesItsVoucher` anulaban con un `UPDATE` de SQL; ahora anulan por la ruta, y prueban también el rechazo de antes (backend `fbd6f291`).
- **PRC-R110 y PRC-R116** suman una prueba cada una con retención: lo cubierto frente a lo que se debe y la suma de las imputaciones (`fbd6f291`).
- **PRC-R101:** `SupplierBalancesIntegrationTests.aDealerOffersNoOrderAndShowsTheInitialsOfItsSettledUnits` ya existía; ahora paga dos facturas de dealers y comprueba que la que tiene todas sus unidades saldadas no se ofrece (`96c62684`).
- **Nuevas:** `PaymentEquivalentsTests` (PRC-R107, PRC-R109), las de la idempotencia (PRC-R129), la del canal de dealers (PRC-R105, PRC-R125), la de los céntimos (PRC-R120) y la de la bitácora (PRC-R126).
- **Ninguna regla de `fleet`** de esta fase, FLT-R130 a FLT-R132, cambió con la revisión.
- **PRC-R123 y FLT-R131,** después, con R22 (backend `c43d1bc1`, merge `1bd767f1`): la fecha base es de cada unidad (§2.3). PRC-R123 suma `UnitCoveragePolicyTests.eachUnitCountsItsGapFromTheFirstPaymentThatCoversIt` y la fecha de cada bloque en `aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf`; las dos suman `VehiclesIntegrationTests.eachUnitOfAnInvoicePaidInTwoBlocksCountsItsGapFromItsOwnPayment`.

---

## 6. Pantallas

### 6.1 Rutas y menú

| Ruta del navegador | Vista | `data.permissions` | Encabezado |
|---|---|---|---|
| `/procurement/supplier-payments` | `SupplierPaymentsView` | `anyOf: [pagos.ver]` | El del módulo, `modules.supplier-payments` (ya existe) |
| `/procurement/supplier-payments/new`, con `?proveedor=`, `?factura=` o `?copiaDe=` | `SupplierPaymentFormView` | `allOf: [pagos.ver]`, `anyOf: [pagos.registrar_pago]` | `supplier-payments.heading.new-title` y `new-description` |
| `/procurement/supplier-payments/{paymentId}` | `SupplierPaymentProfileView` | `anyOf: [pagos.ver]` | El proveedor y el número de operación, con `overridePageHeading`; sin número, el método: «Pago en efectivo» o «Pago con saldo» (revisión 39 de `DESIGN.md`, frontend `131314dc`) |

**El menú.** En `app-navigation.ts`, la opción `/procurement/supplier-payments` pierde `pending: true` y declara `access: anyOf(PermissionCodes.Payments.VIEW)`.

**Los permisos en el frontend:**
- `PermissionCodes` (`iam/application/permission-codes.ts`) suma el grupo `Payments` (`VIEW`, `REGISTER_PAYMENTS`, `ANNUL_PAYMENTS`).
- `PERMISSION_MODULE_ORDER` pone `'pagos'` después de `'compras'`. Sin eso, `permission-codes.spec.ts` falla, porque el módulo `pagos` no está en esa lista.
- `placePermission` (`permission-matrix-groups.ts`) lleva el módulo `pagos` a Logística › Compras › `pagos-a-proveedores`, la pantalla que `SCREEN_ORDER` ya declara. Hoy lo dejaría en `{ area: 'pagos' }`.
- `es.json` suma `iam.permission-matrix.items.pagos.*`.

### 6.2 Lista

- **Acciones** (`.mm-page-actions`): «Registrar pago», solo con `hasPermission(REGISTER_PAYMENTS)`, y «Exportar».
- **Filtros:**
  - empresa (`mm-company-filter`);
  - proveedor (`mm-search-select`, activos e inactivos; desde el 04/11/2026 busca en el servidor, frontend `3a995e95`);
  - método;
  - estado (Todos, Vigentes, Anulados);
  - moneda;
  - período Desde–Hasta, que incluye el último día elegido.
- **Columnas:**

  | Columna | Contenido | Ordena |
  |---|---|---|
  | Fecha | `dd/mm/aaaa` | Sí |
  | Operación | El número, que abre la ficha; sin número, el método en `text-secondary`: «Efectivo», «Saldo a favor» (revisión 39; antes, todo pago sin número decía «Saldo a favor») | Sí |
  | Proveedor | Razón social y, debajo, el documento; la insignia «Dealer» si lo es | Sí |
  | Método | Traducido | No |
  | Monto | Con su moneda | Sí |
  | Estado | Insignia Vigente o Anulado | No |

- **Menú de la fila:** «Ver ficha».
- **Estados:** esqueleto mientras carga; vacío, «Todavía no hay pagos registrados», con «Registrar pago» si se puede; error, con «Reintentar».

### 6.3 Formulario

Una página con sus paneles, como los dibuja la revisión 37 de `DESIGN.md`, que manda en lo visual. Se registra en `UnsavedWork`: salir con cambios pregunta con `UnsavedChangesQuestion`. Desde la ficha de una factura llega con el proveedor y esa factura marcada; desde la ficha del proveedor, con el proveedor.

**Panel «Datos del pago»:**

| Campo | Comportamiento |
|---|---|
| Empresa | Solo si la sesión paga en más de una. Desde el 04/11/2026, las que lee con `pagos.ver` y donde `pagos.registrar_pago` actúa en toda la empresa o en alguna marca, nunca solo por una sede (C8) |
| Proveedor | `mm-search-select` con los proveedores de la empresa, también los inactivos (D-212), buscados en el servidor desde el 04/11/2026. Elegido, muestra «Canal de dealers» si lo es, y carga lo que se le debe y su saldo |
| Fecha | Calendario hasta hoy en la zona de la empresa (D-212) |
| Método | Transferencia, Cheque, Efectivo, Saldo a favor. Saldo a favor solo se ofrece si el proveedor tiene saldo en alguna moneda; desde el 04/11/2026 se retiene, con su razón, a quien registra pagos solo por una marca (C8) |
| N.° de operación | Obligatorio en transferencia y cheque, opcional en efectivo, oculto con saldo (D-212). Muestra debajo cómo se guardará (`Se guarda como OP99213`) |
| Moneda | Empieza en la funcional. Con saldo a favor, solo las monedas con saldo, y debajo «Saldo disponible: USD 600,00» |
| Tipo de cambio | El campo de la fase 2: «1 · moneda de la empresa» en la funcional; en otra, la tasa con su día y su fuente, el aviso de la tasa anterior y «Registrar tasa» en modo fijado |
| Monto | Con los decimales de la moneda |
| Notas | Texto libre, opcional |

**Panel «Aplicar a»:**
- **Qué lista:** lo que se debe al proveedor (§4.2.7), las facturas primero y después las órdenes, de lo más antiguo a lo más nuevo. Con saldo a favor, solo las facturas de su moneda.
- **Tabla editable**, fuera de `mm-data-manager` (revisión 37, ítem 1). Cada fila tiene:
  - una casilla «Aplicar»;
  - el documento, que abre su ficha en otra pestaña, con su fecha;
  - la moneda, el total, las notas, lo aplicado y lo que se debe;
  - el monto a aplicar;
  - la retención y su comprobante, que se muestran con «Agregar retención»;
  - en otra moneda, la tasa de la aplicación con su día y el equivalente en la moneda del pago;
  - «Unidades», que abre la imputación.
- **Una fila que no se puede aplicar** queda deshabilitada y dice por qué: «Se paga en USD o PEN, no en EUR» (D-194), o «Dealer: se paga contra sus facturas» en una orden.
- **Acciones por fila:** «Aplicar lo que se debe» llena el monto con lo que se debe; desde el 04/11/2026, menos la retención de la fila y, en una factura de dealers, menos `settledInitials`.
- **Acción del panel:** «Repartir el monto» llena las filas marcadas, de la más antigua a la más nueva, hasta agotar el monto. Nunca pasa de lo que se debe de cada factura, con el mismo descuento; desde el 04/11/2026 llena desde el lado del pago, y los equivalentes absorben el redondeo (§2.4).

**El diálogo «Unidades de F001-123»** (revisión 37, ítem 2):
- **Qué lista:** la posición de la factura (§4.2.8): VIN, modelo, valor, cubierto, lo que le falta y el estado. Una unidad saldada está deshabilitada y dice «Saldada: no recibe más pagos».
- **Cada fila:** un monto, que no pasa de lo que le falta (D-211), y, con un proveedor de dealers, la casilla «Salda la unidad», con la pista «Lo que falta es la inicial del cliente».
- **Acciones:**
  - «Repartir en proporción» reparte lo cubierto por la aplicación como §2.3;
  - «Completar lo que falta» llena las elegidas hasta su valor.
- **El pie:** «Imputado 92 000,00 de 92 000,00», con «Cuadra» o la diferencia.
- **Sin imputar:** en un proveedor normal, el pie dice «Sin imputar: se reparte en proporción al valor», y no es obligatorio. En uno de dealers, la fila de la factura dice «Faltan las unidades» hasta que se imputen.

**La barra de totales**, al pie y fija en el teléfono (patrón de la revisión 34):
- «Monto del pago»;
- «Aplicado», la suma de los equivalentes;
- «Falta aplicar», que tiene que llegar a cero (D-210);
- «Retenciones», por moneda;
- «Diferencia de cambio (calculada)», en la moneda de la empresa, con «pérdida» o «ganancia».

Se calculan con `computed` y `BigInt`, como la factura.

**Registrar pago:**
- **No se envía** mientras lo aplicado no sea el monto, una fila supere lo que se debe con su monto más su retención, una imputación no cuadre o falten las unidades de un dealer; el formulario dice por qué.
- **El `requestId`** se genera al abrir el formulario (§4.6).
- **Las respuestas:**
  - un `200` del mismo envío abre la ficha del pago, como un `201`;
  - un `409 …REQUEST_CONFLICT` dice «Ese envío ya registró un pago» con «Ver pago»;
  - un `422 …EXCHANGE_RATE_CHANGED` actualiza las tasas y pide guardar otra vez;
  - un error con `context.applications` marca esas filas.

**Como se corrigió (04/11/2026), en el frontend** (frontend `6386f2fa` a `1ef10642`, en `develop` con `cc782649`; revisión 39 de `DESIGN.md`):
- **La empresa** (`fdf14036`, `18e03774`): solo las que quien registra lee con `pagos.ver` y donde `pagos.registrar_pago` actúa en toda la empresa o en alguna marca, según la sesión (`wholeCompanyPermissions` y `brandPermissions`, convenciones §9.1). Un rol de sede no basta. Si ninguna empresa queda, el formulario lo dice: «Su permiso de registrar pagos no alcanza a toda una empresa: el de una sede no basta. Pida que se lo den sobre la empresa.».
- **El proveedor** (`3a995e95`, sobre `6c779bc3`, que deja a `mm-search-select` buscar en el servidor): se busca en el servidor, y el que nombra `?proveedor=` se lee por su id aunque no esté en la primera página. «Ver pagos» de la ficha del proveedor abre la lista también con su empresa (`6205e8d4`).
- **«Saldo a favor»** (`64581af4`): se retiene, con su razón, a quien registra pagos solo por una marca, porque un pago con saldo es de toda la empresa y de todas sus marcas: «Su permiso de registrar pagos se limita a una marca: un pago con saldo a favor es de toda la empresa.». Una copia que lo trae queda retenida. Sin `wholeCompanyPermissions` en la sesión, el formulario lo ofrece y decide el backend.
- **El monto con su retención** (`c6c96b1e`): la fila juzga monto más retención contra lo que se debe, como el backend: «Con la retención, supera lo que se debe: {owed}». «Aplicar lo que se debe» llena lo que se debe menos la retención de la fila y, en una factura de dealers, menos `settledInitials`, la inicial de sus unidades saldadas; ese es también el tope de «Repartir el monto».
- **Los equivalentes** (`20cebe6f`): absorben el redondeo como el backend (§2.4). «Repartir el monto» llena desde el lado del pago: cada fila toma lo que queda del pago convertido de vuelta a su moneda, topado en lo que se debe, y la fila que toma el resto cuadra al absorber. Los céntimos del reparto entre unidades siguen la regla C2 (`f8efb41b`, §2.3).
- **Los rechazos** (`1e189bf0`, `be45bcfc`, `2b023ba4`): `…_EXCEEDS_OWED` nombra en la fila lo que se debía, «Supera lo que se debe ahora: {owed}», y `…_BALANCE_INSUFFICIENT` el saldo, «El saldo a favor del proveedor es {balance}: no alcanza para este pago.». La primera vuelve a leer lo que se debe, y la segunda, el saldo y lo que se debe; las filas conservan lo escrito, juzgado por lo que deben ahora. La marca de un rechazo se quita en cuanto la fila cambia sus unidades, su retención, su comprobante o su monto.
- **Las tasas** (`b556fa26`): pedir las tasas otra vez mientras se lee una ya no pierde las que esperaban en la cola.
- **Los montos** (`6386f2fa`): aceptan ceros después de los decimales de la moneda, como el backend.
- **Un límite que queda:** lo que se ofrece pagar no trae marcas, así que el formulario no retiene una factura de una marca que el permiso no alcanza; el backend la rechaza con `403` (§4.2.4).

Lo prueban, entre otras, `supplier-payment-form-view.spec` ("searches the suppliers on the server, and keeps the one chosen named whatever the next search brings", "reads what is owed again after a refusal past it, and says so", "says the balance a payment with balance passed, and reads it and what is owed again", "judges a row by the facts read again, never by those it was typed against"), `payment-application-form.spec` ("drops it when its units change", "drops it when its withholding or its certificate change"), `payment-rates.spec` ("reads every currency still pending when a read is asked again while one runs") y las e2e "a refusal marks its row until the row changes: allocating the units again, after one was settled meanwhile, lets the payment be sent" y "a withholding covers what is owed with the amount: past it the row says so, «Aplicar lo que se debe» leaves it out, and the payment is sent". axe pasa sobre la fila con retención de más y sobre «Saldo a favor» retenido (`e2e/accessibility.spec.ts`).

### 6.4 Ficha del pago

- **Panel de datos**, encabezado por el proveedor y el número: empresa, fecha, método, moneda, tipo de cambio con su día, monto, diferencia de cambio (calculada), notas y estado.
- **Línea de estado:** un pago anulado lo dice una vez junto a su insignia: «Anulado el 06/10/2026 15:04 por Tere Ramos: error de registro.».
- **Cabecera**, según `allowedActions`:
  - **«Anular»** abre un diálogo con el motivo en dos opciones, «Error de registro» y «Devuelto» (D-212), y el texto «Lo aplicado deja de contar: lo que se debe de sus N documentos vuelve a subir.». El botón destructivo es «Anular pago».
  - **«Anular y copiar»** usa el mismo diálogo y, al anular, abre `/procurement/supplier-payments/new?copiaDe={id}` con la copia: el proveedor, la fecha, el método, el número de operación, la moneda, el monto, las aplicaciones y las imputaciones. Las tasas se vuelven a proponer, y el `requestId` es nuevo. **Como se corrigió (04/11/2026)** (frontend `1e189bf0`): la copia se revisa contra lo de hoy y dice lo que cambió. No copia un documento que ya no se debe o que este pago no puede pagar, y lo nombra; revisa cada imputación contra lo que le falta hoy a su unidad —quita la de una unidad saldada o a la que no le falta nada, topa la que pasa de lo que le falta— y deja «Salda la unidad» solo con un dealer, con un aviso para revisarlas; y no copia el número de operación de un pago que sigue vigente, porque lo sigue usando: «Su número de operación, {operation}, no se copió: lo sigue usando ese pago.». El de un pago anulado sí se copia.
  - **Un `422 …BALANCE_NEGATIVE`** dice «Sin este pago, el saldo con el proveedor quedaría en PEN −400,00. Anule antes el pago con saldo que lo usó.».
- **Panel «Aplicaciones»:** una fila por aplicación, de solo lectura: documento (enlace), moneda, monto, retención y comprobante, tasa y día, equivalente y diferencia de cambio. Cada factura con imputaciones se despliega en sus unidades: VIN, monto y «Saldada».
- **Estados:** esqueleto, «No encontramos ese pago» con vuelta a la lista, error con «Reintentar».

### 6.5 Ficha de la factura

Suma a la de la fase 2 (su §6.4):
- **La cabecera** ofrece «Registrar pago» con `allowedActions.registerPayment`, que abre el formulario con `?factura=`.
- **El panel «Unidades»** suma la columna «Pago», con la insignia del estado de cada línea (D-212). Con `pagos.ver`, también «Cubierto» y «Diferencia», de la posición de la factura (§4.2.8).
- **Un panel nuevo, «Pagos»,** se muestra solo con `hasPermissionIn(pagos.ver, empresa)` y lee la posición de la factura:
  - **El resumen de lo que se debe,** como una resta: total, menos notas, menos aplicado, igual a lo que se debe. Si es negativo, dice «Pagado de más: PEN 400,00, saldo a favor del proveedor». En un proveedor de dealers con unidades saldadas, una línea más: «De eso, iniciales de clientes de 1 unidad saldada: 8 000,00».
  - **La tabla de aplicaciones:** fecha, operación (enlace al pago), método, monto, retención, equivalente, diferencia de cambio, «Desde la orden OC-…» cuando pasó de ella, y el estado del pago.
  - **Estados:** «Todavía no tiene pagos» o el error con «Reintentar».

### 6.6 Ficha del proveedor

Suma a la de la revisión 31:
- **El panel de datos** muestra «Canal de dealers: Sí/No». El diálogo de edición suma la casilla «Canal de dealers», con la pista «El cliente paga su inicial al concesionario: los pagos a sus facturas detallan sus unidades.».
- **La cabecera** ofrece «Registrar pago» con `allowedActions.registerPayment`.
- **Un panel nuevo, «Saldo y por pagar»,** se muestra con `pagos.ver`. Por moneda: «Por pagar», «Pagado de más», «Usado en pagos con saldo» y «Saldo a favor», con «Ver pagos», que abre la lista filtrada por el proveedor. Sin movimiento, dice «Sin pagos ni saldo con este proveedor».

### 6.7 Ficha de la unidad

Suma a la de las fases 2, 3 y 4:
- **Junto a las insignias de la cabecera,** la del estado de pago (`payment.status`) y, en los datos, «Brecha de días: 123», con la pista «Desde el primer pago, o la factura si no hay pago, hasta la entrega o hoy». Sin línea vigente no se muestra nada: ya está la insignia «Sin factura vigente» de la fase 2. Desde el 04/11/2026 el primer pago es el que cubre la unidad, que responde el backend (§2.3); la pista no cambió.
- **El panel «Compra y costo» de la fase 4** (su §6.6), que ya se muestra con `allowedActions.viewPurchaseCost` y se lee por `GET /vehicles/{id}/purchase-cost`, suma una sección «Pago», con `purchase.payment`:
  - los cinco montos como una resta: los tres que la fase 4 ya muestra —valor de factura, menos notas, igual a valor del vehículo—, menos cubierto, igual a diferencia, rotulada «Pendiente de pago» o «Inicial del cliente»;
  - debajo, el dinero entregado, la retención y lo pagado con saldo;
  - y los pagos, «OP1001 · 03/07/2026 · 50 000,00 / OP1002 · …», como la columna 42 de la guía, sin concatenar en un campo.
- **Sin fachada nueva:** la sección se lee con el store de `fleet` de la fase 4 (`purchaseCost`), así que su carga y su error son los del panel.

### 6.8 La orden

La ficha de la orden deja de ofrecer «Anular» y «Dar por cumplida» cuando tiene pagos aplicados (D-211). Si el backend responde `PURCHASEORDER_HAS_PAYMENTS`, dice «La orden tiene pagos aplicados: anule los pagos o vincúlela a sus facturas.».

### 6.9 A 375 px y accesibilidad

**A 375 px:**
- La lista es la de `mm-data-manager`, que no tiene modo de tarjetas: se desplaza de lado.
- La tabla de «Aplicar a» mantiene fijas a la izquierda la casilla y el documento mientras se desplaza de lado. La barra de totales se pliega en una fila de 48 px con «Ver los totales», como la revisión 34.
- El diálogo de unidades ocupa la pantalla entera, con su pie fijo.
- Los paneles de las fichas se apilan, y las restas de los montos se leen en una columna.
- La página no tiene desplazamiento horizontal.

**Accesibilidad:**
- Cada campo editable de una fila tiene su `aria-label` («Monto a aplicar a F001-123»), y su error queda enlazado con `aria-describedby`.
- El estado de pago y el del pago son texto, no solo color.
- Lo que falta aplicar, la diferencia de la imputación y el resultado de «Repartir» se anuncian en una región `aria-live="polite"`.
- Una fila deshabilitada lleva su razón en `aria-describedby`.
- Los diálogos atrapan el foco y lo devuelven al botón que los abrió.
- axe no encuentra violaciones en las tres anchuras.

### 6.10 Stores y fachadas

| Pieza | Contenido |
|---|---|
| `procurement/domain/model` | `supplier-payment.entity.ts` (`SupplierPayment`, `PaymentApplication`, `PaymentAllocation`), `supplier-balance.ts`, `supplier-payables.ts`, `invoice-payment-position.ts`, `unit-payment-position.ts`, `unit-payment-status.ts` (los cuatro códigos como unión de tipos), los comandos `register-supplier-payment.command.ts` y `annul-supplier-payment.command.ts`, `supplier-payment-amounts.ts` (usa `amountMinor`, `sumMinor` y `fromMinor` de `purchase-order-amounts.ts`; el equivalente y el reparto en proporción con los céntimos a la primera unidad, igual que el backend), `operation-number.ts`. `supplier.entity.ts` suma `isDealer` |
| `procurement/infrastructure/api` | `supplier-payments-api-endpoint.ts` y su respuesta, `register-supplier-payment.request.ts`, `annul-supplier-payment.request.ts`; `suppliers-api-endpoint.ts` suma el saldo y lo que se debe; el de facturas, la posición; `procurement-api.ts` los agrupa |
| `procurement/infrastructure/assemblers` | `supplier-payment-assembler.ts`, `payment-position-assembler.ts` |
| `procurement/application` | `supplier-payments.store.ts`: `paymentPage` (`PagedList`), `payment`, `payables`, `balance`, `invoicePosition`, `loading`, `saving`, `exporting`, `error`, `notice`, y los métodos `register()` (devuelve el id nuevo o `null`), `annul()`, `loadPayables()`, `loadBalance()`, `loadInvoicePosition()` y `exportPayments()`. Elige el texto del error con un `switch` por `ApiError.code` |
| `procurement/presentation` | Vistas `supplier-payments-view`, `supplier-payment-form-view` y `supplier-payment-profile-view`; componentes `payment-applications-table` (edición y lectura), `payment-allocations-dialog`, `annul-supplier-payment-dialog`, `invoice-payments-panel`, `supplier-balance-panel`, `supplier-payment-status-tone.ts` y `unit-payment-status-tone.ts` |
| `fleet` | `vehicle.entity.ts` suma `payment`; `vehicle-purchase-cost.entity.ts` (fase 4) suma `payment`; `vehicle-profile-view` muestra la insignia y la brecha, y `vehicle-purchase-cost-panel` (fase 4), la sección «Pago». Ninguna fachada nueva entre contextos, como en la fase 4 |
| `LocationFacade` | Los miembros de la fase 2: `loadPurchaseRate` y `registerFixedRate`, para la tasa del pago y la de cada aplicación |
| `OrganizationFacade`, `IamFacade` | Los que ya tienen |

**Como se construyó, en el frontend** (`278d6bfe` a `4e079f12`; frontend `e0ed0f68` en `develop`):
- **El formulario:** lee las tasas una a la vez por `LocationFacade.loadPurchaseRate`, y todas otra vez ante `SUPPLIERPAYMENT_EXCHANGE_RATE_CHANGED`; manda el número de operación normalizado; reparte con `splitInProportion`, con la mitad hacia arriba y los céntimos en la primera unidad con peso, como el backend; y no envía un pago que pase los límites de 100 aplicaciones, 100 imputaciones por factura o 500 por pago, con un mensaje propio de cada uno (`f2144160`). Un rechazo que nombra aplicaciones marca las filas enviadas, no las revisadas (`d0040567`).
- **«Le falta»**, en el diálogo de imputación, lee `lacking` de la posición, como tope y como peso de «Completar» y «Repartir» (`a214ec87`, `8edfa99b`): antes usaba el valor menos lo cubierto, que bloqueaba montos que el backend acepta. Fue la rotura que encontró la revisión del contrato del 03/11/2026 (§7.5).
- **Con «Saldo a favor»,** «Aplicar a» lista solo las facturas de esa moneda, sin órdenes, como pide el ítem 9 de la revisión 37 (`4e079f12`, que reemplaza a `ab90f125`), y si no hay ninguna lo dice en esa moneda (`27601b5d`).
- **Las fichas:** el panel «Saldo y por pagar» del proveedor va debajo de Contactos, con «Ver pagos» (`13526ee1`); el desglose de lo que se debe de la factura va en un grupo con nombre, para axe (`1056c3a7`), y `InvoicePaymentsPanel` tiene `breakdownOnly`, para mostrar solo el desglose en el catálogo; los pagos de la sección «Pago» de la unidad abren su ficha en otra pestaña, y `fleet` conserva sus propios tonos y textos (`fleet.payment-status.*`), sin importar nada de compras; la brecha dice «1 día» o «N días», con el signo menos si es negativa (`f2e3b686`). Tras `PURCHASEORDER_HAS_PAYMENTS`, la ficha de la orden la vuelve a leer (`0c10fa14`). `fromOrder` se lee como el número de la orden (`15cde6e9`), y el texto de `PURCHASEINVOICE_HAS_DEPENDENTS` nombra también los pagos (`206d9283`).
- **Base:** el contexto no tiene fachada de pagos, porque ningún otro contexto los consume (§6.10; `806e57a7`, `29246002`).

### 6.11 Revisión 37 de `DESIGN.md`

Esta especificación no edita `DESIGN.md`. La revisión 36 es de la fase 4. La **revisión 37** se escribe antes de construir las pantallas del paso 7.11 y decide:

| # | Qué decide | Por qué |
|---|---|---|
| 1 | La tabla «Aplicar a», editable y fuera de `mm-data-manager`: casilla, documento fijo a la izquierda, celdas de monto, la tasa y el equivalente de otra moneda en una celda de dos líneas, la fila deshabilitada con su razón, y su forma a 375 px | Una tabla nueva fuera de `mm-data-manager` exige una decisión (frontend-feature §3) |
| 2 | El diálogo de imputación de unidades: su lista, «Repartir en proporción», «Salda la unidad», el pie que cuadra y la unidad saldada deshabilitada | Patrón nuevo |
| 3 | Los tonos de los estados: el del pago, Vigente en `info` y Anulado en `danger`, como la factura; el de la unidad, Sin pago en `neutral`, Pago parcial en `warning`, Pagada en `success` y Saldada en `info` | Textos y tonos nuevos de `mm-status-badge` |
| 4 | La resta que explica un monto —lo que se debe de la factura, los cinco montos de la unidad, el saldo del proveedor—: cada término en su fila, el signo a la izquierda y el resultado bajo una línea | Patrón nuevo, que usan tres fichas |
| 5 | La barra de totales del pago, con «Falta aplicar» y la diferencia de cambio con su signo en palabras | Extiende la de la revisión 34 |
| 6 | El panel «Pagos» de la factura y el «Saldo y por pagar» del proveedor, apilados bajo los existentes | Extiende las fichas de las revisiones 31 y 34 |
| 7 | La sección «Pago» del panel «Compra y costo» (revisión 36, ítem 7) y la insignia de pago de la ficha de la unidad | Cambian pantallas de `fleet` |
| 8 | El diálogo de anulación del pago con su motivo y «Anular y copiar», y el texto del saldo negativo | Reutiliza el de la revisión 34 con otro texto |
| 9 | El método «Saldo a favor» en el formulario: las monedas que ofrece y el aviso del saldo disponible, con `.mm-notice--info` | Reutiliza `.mm-notice` |

**Como se construyó** (`c4940b26`, `0929c522`). El catálogo de la revisión 37 está en la vista del sistema de diseño, sin fila de §8: sus piezas se agrupan por `data-catalog`, con las insignias, los controles, la tabla «Aplicar a» con sus muestras, el diálogo de unidades sobre unidades de muestra, que no pregunta nada a la API, y lo que se debe de tres facturas y el saldo de un proveedor en «Desgloses». `0929c522` abre en la prueba visual la Moneda compartida del catálogo antes que la de las muestras de pagos.

**Como se corrigió (04/11/2026): la revisión 39 de `DESIGN.md`**, escrita en el frontend con las correcciones de la revisión, cambia un punto de la 37. La 37 leía «Saldo a favor» en `text-secondary` donde un pago con saldo no tiene número de operación; el efectivo tampoco puede tenerlo. Ahora, sin número, la celda, la etiqueta de su fila, la ficha de la factura y los pagos de una unidad leen el nombre del método, todavía en `text-secondary`: «Efectivo», «Saldo a favor». La ficha del pago lo titula por su método, «Pago en efectivo» o «Pago con saldo», y los pagos de una unidad en «Compra y costo» lo nombran igual, con el enlace «Abrir el pago en efectivo del 05/10/2026 en otra pestaña» (frontend `131314dc`; e2e "a payment without operation number is named by its method: cash is not «Saldo a favor» in the list, its record or the invoice"). El otro punto de la revisión 39, los valores de «Pegar columnas» para todas las líneas, es de la factura (su especificación, §6.3).

---

## 7. Pruebas

### 7.1 Criterios de aceptación

El plan no fija criterios para la fase 5; estos los fija esta especificación, cada uno con su prueba:

| Criterio | Prueba |
|---|---|
| La nota antes del pago baja lo que se debe y no crea saldo | `SupplierPaymentsIntegrationTests.theExampleOfTheNoteBeforeThePayment` |
| La nota después del pago crea saldo, el pago con saldo lo consume, y ni el pago ni la nota se anulan si lo dejarían negativo | `SupplierPaymentsIntegrationTests.theExampleOfTheNoteAfterThePayment`, `annullingThePaymentWhoseOverpaymentWasSpentAnswersBalanceNegative`; `PaymentSupplierBalanceAdapterTests.annullingTheNoteWhoseBalanceWasSpentIsRefused` |
| Una factura en dólares se paga en soles con la tasa del día del pago; el costo de la unidad no cambia y la diferencia de cambio sale aparte | `SupplierPaymentsIntegrationTests.theExampleOfADollarInvoicePaidInSoles`; `ExchangeDifferencePolicyTests.theExampleOfADollarInvoicePaidInSoles`; que el costo no cambia lo da la construcción (PRC-R124) |
| El doble envío guarda un solo pago | `SupplierPaymentLockTests.twoIdenticalRequestsAtOnceLeaveOnePayment`; `SupplierPaymentsIntegrationTests.theSameRequestTwiceAnswersTheSamePayment` |
| Un pago que no se aplica completo no se guarda | `SupplierPaymentTests.aPaymentWithARestToApplyIsRefused`; `SupplierPaymentsIntegrationTests.aPaymentWithARestToApplyAnswersAppliedMismatch` |
| Un pago en soles de una factura en dólares cuadra con el monto del voucher: el equivalente absorbe el redondeo (desde el 04/11/2026) | `PaymentEquivalentsTests`, V1 a V6; `SupplierPaymentsIntegrationTests.aPaymentInSolesAbsorbsTheResidueOfTheRounding`; e2e ("PEN 10,000.00 squares with USD 2,659.57 at 3.760000: the equivalent absorbs the rounding, and the payment is registered (C1)") |
| Un reintento con otra retención, otras imputaciones u otras notas no responde el pago anterior (desde el 04/11/2026) | `SupplierPaymentsIntegrationTests.theSameRequestWithAnotherWithholdingAllocationOrNotesAnswersRequestConflict`; `PurchaseInvoicesIntegrationTests.aRetryAfterTheApplicationToTheOrderPassedToTheInvoiceAnswersThePayment` |
| El mismo voucher del proveedor da `409`; anulado, se libera | `SupplierPaymentsIntegrationTests.theSameVoucherOfTheSupplierAnswersOperationConflict`, `anAnnulledPaymentFreesItsVoucher` |
| Dos pagos a la misma factura a la vez nunca aplican más de lo que se debe | `SupplierPaymentLockTests.twoPaymentsOfTheSameInvoiceNeverApplyMoreThanItsOwed`, con dos transacciones reales |
| El canal de dealers exige unidades, salda una unidad y la diferencia es la inicial | `SupplierPaymentsIntegrationTests.theExampleOfTheDealerChannel`; `UnitCoveragePolicyTests.theFiveAmountsOfADealerUnitSettledByItsInitial` |
| Sin imputar, se reparte en proporción, y pagar la mitad de las motos no cubre dos veces | `UnitCoveragePolicyTests.withoutAllocationsTheApplicationGoesInProportionToTheValue`, `aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf` |
| El pago de una orden pasa a su factura al vincularse, hasta lo que se debe, y la orden con pagos no se anula | `PurchaseInvoicesIntegrationTests.registeringAnInvoiceOfAPaidOrderMovesThePaymentToTheInvoice`; `PurchaseOrdersIntegrationTests.anOrderWithAPaymentIsNotAnnulled` |
| La factura con un pago no se anula | `PurchaseInvoiceDependentsTests.anInvoiceWithAPaymentHasDependents` |
| La anulación exige motivo y el `updatedAt` leído, y deja su fila en la bitácora | `SupplierPaymentsIntegrationTests.annullingNeedsItsPermissionAReasonAndTheReadMoment`, `theAnnulmentLeavesItsRowInTheChangeLog` |
| La ficha de la unidad muestra el estado y la brecha sin montos, y los montos solo con `stock.ver_costos` | `VehiclesIntegrationTests.theRecordOfAUnitCarriesItsPaymentStatusAndGapWithoutAmounts`; `VehiclePurchaseCostIntegrationTests.aReaderWithoutCostsGetsAccessDenied` |
| Los totales del formulario cuadran al céntimo | `supplier-payment-amounts.spec` ("tells what remains to apply to the cent", "spreads in proportion with the cents on the first unit") |
| axe no encuentra violaciones; salir pregunta antes de descartar | la pasada de axe de `e2e/screens.ts` sobre `supplier-payments`, `supplier-payment-form` y `supplier-payment`, en los tres anchos; `supplier-payment-form-view.spec` ("asks before discarding what was typed, and leaves without asking when nothing was"); e2e ("leaving the form with applications asks before discarding them") |
| `ModularityTests`, RLS y filas que no se borran | `ModularityTests`; `ProcurementCompanyIsolationIntegrationTests`; `RowLevelSecurityMigrationTests`; `ProcurementRowsAreNeverDeletedTests`; `NoDeleteEndpointsTests` |

### 7.2 Sentencias y rendimiento

- **Sentencias.** `SupplierPaymentRegistrationStatementsTests`, con `hibernate.generate_statistics`, registra pagos de 1, 10 y 100 aplicaciones, con y sin imputaciones. Comprueba que las sentencias que no son inserciones son el mismo número en los tres tamaños, y ese número queda escrito en la prueba (§2.6).
- **Lecturas.** `PayableFactsReadRepositoryTests` comprueba que el saldo de un proveedor con 200 facturas y la posición de una factura de 100 líneas salen de una consulta por clase.
- **Objetivo provisional:** un pago de 100 aplicaciones en menos de 1 s de mediana sobre `erp_mm_db_verify`, medido primero como línea base, como en la fase 2.

**Como se construyó** (`195d8689`, `ca56b0fa`). `SupplierPaymentRegistrationStatementsTests.theQueriesOfARegistrationDoNotGrowWithItsApplications` fija 38 sentencias además de las inserciones con 1, 10 y 100 aplicaciones sin imputar, y 44 con una imputación por aplicación. `PayableFactsReadRepositoryTests.theBalanceOfASupplierWithTwoHundredInvoicesComesOutOfOneStatementPerKind` y `thePositionOfAnInvoiceOfAHundredLinesComesOutOfOneStatementPerKind` fijan las lecturas. Los pagos suman sentencias a otros registros: el de una factura pasa de 60 a 61 con B11, por los pagos que la retienen, y a 64 con B19, por las tres lecturas del estado de pago de sus líneas (`PurchaseInvoiceRegistrationStatementsTests`); el de una nota, de 41 a 42 con B14, por las notas vigentes del proveedor con que la ficha decide si ofrece anularla (`CreditNoteRegistrationStatementsTests`). El objetivo de 1 s no se midió.

**Como se corrigió (04/11/2026): la línea base** (backend `b44950c8`; hallazgo B19 de la revisión). `SupplierPaymentRegistrationBaselineTests.measureTheRegistrationByItsSize` registra cinco pagos de cada tamaño, 1, 10 y 100 aplicaciones, cada una a una factura propia de una unidad, sin imputar y con una imputación por aplicación, y escribe la mediana y el máximo. Lleva la etiqueta `performance` y corre solo cuando se pide, fuera de la suite normal, con `-Dperformance=true` sobre `erp_mm_db_verify`; deshace sus filas. Medida el 04/11/2026 como `postgres`, en milisegundos:

| Aplicaciones | Sin imputar: mediana / máximo | Con una imputación cada una: mediana / máximo |
|---|---|---|
| 1 | 303 / 441 | 263 / 399 |
| 10 | 500 / 586 | 244 / 390 |
| 100 | 899 / 1 911 | 820 / 1 373 |

La mediana de 100 aplicaciones cumple el objetivo de menos de 1 s; los máximos no. El máximo de cada serie de 100 es su primera corrida, con el equipo cargado. Se midió con la cuenta `postgres`, no con `erp_mm_app`, la de la aplicación, que pasa por RLS (§10). Las sentencias no cambiaron con la revisión: 38 y 44, iguales con 1, 10 y 100 aplicaciones.

### 7.3 Extremo a extremo

**Escenarios de `e2e/supplier-payments.spec.ts`,** con los mocks en `e2e/support/mocks/procurement.ts` (fase 2) y los de la unidad en `mocks/fleet.ts`:
1. La lista filtra por empresa, proveedor, método, estado y período, y exporta con los filtros.
2. Registra un pago en soles a una factura en dólares y a otra en soles: la tasa de la aplicación con su día, el equivalente, la diferencia de cambio y «Falta aplicar: 0,00».
3. Una fila por encima de lo que se debe lo dice y no deja enviar; «Aplicar lo que se debe» la corrige.
4. «Repartir el monto» llena de lo más antiguo a lo más nuevo y nunca pasa de lo que se debe.
5. Un proveedor de dealers: la factura dice «Faltan las unidades»; el diálogo imputa, salda una unidad y cuadra.
6. Saldo a favor: solo las monedas con saldo, el saldo disponible y solo las facturas de esa moneda.
7. Sin tasa del día: «Registrar tasa» en modo fijado, y la tasa nueva en el pago y en sus aplicaciones.
8. El doble clic, o un reintento después de perder la respuesta, abre el mismo pago (`200`).
9. Ficha: anular con motivo; el `422 …BALANCE_NEGATIVE` lo explica; «Anular y copiar» precarga la copia con otro `requestId`.
10. La ficha de la factura: el resumen de lo que se debe, «Pagado de más» y la tabla de pagos; «Registrar pago» abre el formulario con la factura.
11. La ficha del proveedor: «Canal de dealers», «Saldo y por pagar» y «Registrar pago».
12. La ficha de la unidad: la insignia de pago y la brecha; con `stock.ver_costos`, los cinco montos; sin él, el panel no aparece.
13. Salir con aplicaciones pregunta antes de descartar.
14. Sin `pagos.registrar_pago` no aparece «Registrar pago»; sin `pagos.ver` no aparecen los paneles de pagos.

`e2e/screens.ts` suma `supplier-payments`, `supplier-payment-form` (con aplicaciones) y `supplier-payment`, con sus imágenes en las tres anchuras y su pasada de axe. La pantalla «En construcción» deja de cubrir Pagos a proveedores. Las e2e corren en el puerto 4300, con el equipo enchufado.

**Como se construyó** (`328c507f`, `12916b03`, `4d9c326b`, `f8737a0d`, `5999f719`):
- **Los mocks** de los pagos están en su propio archivo, `e2e/support/mocks/supplier-payments.ts`, y no en `procurement.ts`; después de la revisión del contrato rechazan como el backend: las tasas, el orden de las revisiones, el `400` de una aplicación en otra moneda y el alcance de las órdenes por la marca de su plantilla (`f8737a0d`). `pinServerDay` fija el día del servidor, y las pantallas de la ficha de la unidad lo fijan en el 01/10/2026.
- **`e2e/supplier-payments.spec.ts`** tiene 20 escenarios, que cubren los catorce de arriba: "the list filters by company, supplier, method, status and period, and exports with the filters of the screen"; "registers a payment in soles to an invoice in dollars and one in soles: the rate of each, the equivalent, the difference and nothing left to apply"; "a row past what it owes says so and holds the payment back; «Aplicar lo que se debe» corrects it"; "«Repartir el monto» fills the oldest documents first and never passes what each owes"; "a dealer invoice says its units are missing; the dialog allocates them, settles one and squares" y "a unit covered by money applied without allocations still takes what it lacks: its value less what is allocated to it (D-211)"; "with balance: only the currencies with balance, the balance available and only the invoices of that currency"; "without the rate of the day «Registrar tasa» registers it in fixed mode, and the payment and its applications read it"; "a send whose answer was lost is sent again with the same request, and opens the payment it saved (200)"; "the record annuls with a reason; a negative balance is explained; «Anular y copiar» opens the copy with another request"; "the invoice record says what it owes, paid in excess, and its payments; «Registrar pago» opens the form on it" y "the invoice record of who does not read payments shows the status of each line without amounts nor the panel"; "the supplier record says its dealer channel, what is owed to it and its balance, and opens its payments and the form" y "the supplier record of who does not read payments has neither its balance nor «Registrar pago»"; "the record of a unit without stock.ver_costos says its payment status and its gap, without «Compra y costo»", y en `e2e/fleet.spec.ts`, "the record of a unit says its payment status and its gap in days, and «Compra y costo» what its payments cover"; "leaving the form with applications asks before discarding them"; y "without pagos.registrar_pago no screen offers «Registrar pago», and the form is not reached" y "without pagos.ver the menu, the list and the panels of payments are not offered". Suma dos que prueban el orden del backend: "the mock refuses as the backend does: the 403 of the route first, a document in another currency, the rates of a divide" y "a method or a reason the service does not know is its 400, once the route lets the session in".
- **La pantalla «En construcción»** queda en Cargas de archivos (`/imports`), desde `328c507f`, el primer commit de la rama.

**Como se corrigió (04/11/2026).** `e2e/supplier-payments.spec.ts` tiene 27 escenarios, siete más:
- "PEN 10,000.00 squares with USD 2,659.57 at 3.760000: the equivalent absorbs the rounding, and the payment is registered (C1)", que también comprueba que un rechazo informa la suma hacia adelante (`20cebe6f`, `af8a3f16`);
- "a withholding covers what is owed with the amount: past it the row says so, «Aplicar lo que se debe» leaves it out, and the payment is sent" (`c6c96b1e`);
- "a refusal marks its row until the row changes: allocating the units again, after one was settled meanwhile, lets the payment be sent" (`be45bcfc`);
- "a payment without operation number is named by its method: cash is not «Saldo a favor» in the list, its record or the invoice" (`131314dc`);
- "a dealer invoice whose units are all settled is no longer offered: what it owes is its customers' down payments (C5)" (`a52c6a71`);
- "a role limited to a brand pays the documents of its brand, never with balance, which is of every brand" y "a session without the scopes offers the payment with balance, and the backend decides" (`1ef10642`).

Los mocks de `e2e/support/mocks/supplier-payments.ts` absorben el redondeo y reparten los céntimos como el backend (`20cebe6f`, `f8efb41b`). `e2e/accessibility.spec.ts` suma la pasada de axe de una fila con retención de más y la de «Saldo a favor» retenido para un rol de una marca (`8357776b`, `1ef10642`).

### 7.4 Sondas en vivo

`e2e-live/supplier-payments.live.spec.ts`, contra el backend real sobre `erp_mm_db_verify`, con la cuenta del superadministrador de verificación:
1. Registra una factura en dólares con su tasa (fase 2) y un pago en soles a ella, con la tasa del día del pago: la ficha muestra el equivalente y la diferencia de cambio.
2. El mismo envío otra vez responde `200` con el mismo pago, y el mismo voucher en otro envío, `409`.
3. Registra una nota de crédito después del pago (fase 4): el proveedor muestra el saldo a favor; un pago con saldo lo consume.
4. Anular el primer pago responde `422 SUPPLIERPAYMENT_BALANCE_NEGATIVE`; anular antes el pago con saldo, y después el primero, funciona, y la bitácora tiene sus dos filas.
5. Marca un proveedor de dealers; un pago sin unidades responde `422`, y con unidades, salda una, cuya diferencia es la inicial.
6. Paga una orden enviada; una factura que se vincula a ella recibe el pago, y la orden ya no ofrece «Anular».
7. La ficha de la unidad muestra su estado de pago y su brecha.
8. La exportación descarga un Excel.
9. Ningún pedido de la corrida usó `DELETE`, y el registro del backend no tiene ningún `ERROR`.

**Pendiente al 03/11/2026.** La sonda existe desde el frontend `03abb43d`, el commit F14, y `5999f719` la rehízo para que cree sus propios proveedores y lea `lacking`; `febf70b1` hizo que las sondas de flota y de habilitación busquen «Estado» como término entero, aparte de «Estado de pago». No corrió: necesita la contraseña de la cuenta de verificación del responsable. Tiene once pruebas: una que entra, encuentra la empresa de Perú, registra un proveedor de la corrida y prepara las unidades y las tasas ("signs in, finds the company of Peru, registers a supplier of this run, and provisions the units and the rates"), y diez casos numerados: los nueve de arriba, en su orden, con la exportación como caso 8; un caso 9 que lee en la matriz los tres permisos de V105 en SA, AE y CO; y el caso 10, que comprueba que ningún pedido usó `DELETE`. Que el registro del backend no tenga ningún `ERROR` lo mira quien la corre. Las sondas de las fases 2, 3 y 4 tampoco corrieron, por la misma razón.

**Al 04/11/2026 sigue sin correr**, por la misma razón. La revisión no la cambió: sus once pruebas son las mismas (§10).

### 7.5 Lo que corrió al integrar (03/11/2026)

- **Backend:** la suite completa, 3 517 pruebas, sobre `erp_mm_db_verify` con la cuenta `postgres`, en `de8f9913`. Contra la línea base de 75 fallos de `e213c01c`, que se esperan con el superusuario, fallan solo tres nuevas: `ProcurementRowsAreNeverDeletedTests.theApplicationCannotDeleteARowOf`, casos 14 a 16, las tres tablas nuevas, porque un superusuario se salta el `DELETE` revocado. La línea base nueva tiene 78. El CI corre la suite con las dos cuentas.
- **Sentencias:** las de §7.2, las mismas con 1, 10 y 100.
- **Frontend,** en `4e079f12`, con el equipo enchufado: las unitarias, 3 201 pruebas en 273 archivos, todas pasan, y stylelint, limpio; la suite e2e completa en escritorio, sobre una compilación de desarrollo fija en el puerto 4310 y con un solo trabajador, 638 pasan, 3 omitidas y ninguna falla, en 23,9 minutos. Pasa también `screen-dates.spec`, «the year of today is the one of Lima on the night it already changed in Tokyo», que la fase 4 dejó por investigar: no se reprodujo ni sola, ni en una corrida completa, ni cuatro veces bajo carga, así que no tiene commit. Las pasadas de axe y las imágenes en los tres anchos de `supplier-payments`, `supplier-payment-form`, `supplier-payment`, la ficha del proveedor, la de la factura, las de la unidad y el sistema de diseño pasan; los proyectos de tableta y teléfono corrieron solo para las pantallas que cambiaron.
- **La revisión del contrato**, el 03/11/2026, del frontend contra el backend, con un agente de solo lectura: el backend sigue esta especificación, con lo que dice **Como se construyó**. En el frontend, una rotura, «Le falta» (§6), y ocho diferencias menores, todas corregidas: `a214ec87` y `8edfa99b`, «Le falta»; `ab90f125`, reemplazado por `4e079f12`, las facturas de un pago con saldo; `206d9283`, el texto de `PURCHASEINVOICE_HAS_DEPENDENTS`; `f2144160`, los límites; `d0040567`, las filas enviadas; `f8737a0d`, los mocks; `15cde6e9`, `fromOrder`; `27601b5d`, la lista vacía de una moneda; y `5999f719`, la sonda. El backend sumó `lacking` para la pantalla (`de8f9913`).
- **La base de desarrollo**, `erp_mm_db`, estaba en V85: el próximo `spring-boot:run` aplica V86 a V107, y después el responsable corre `scripts/database/hand-over-database.sql`, por las cinco funciones nuevas y el `UPDATE` que se quita a las imputaciones (§3.4).

---

## 8. Secuencia de commits

Ramas `feature/procurement-payments` en los dos repositorios, desde `develop`, en worktrees hermanos. Cada commit compila y pasa las pruebas que alcanza, siempre con `ModularityTests`; con tablas nuevas, también las de RLS y de filas que no se borran.

**Documentación**
1. `docs(conventions): add the contract of supplier payments` — 7.0. Pasa §4 a las convenciones, con D-210 a D-213. Hecho en la documentación `f711675`.

**Backend**

| # | Commit | Paso | Depende de | Hecho en |
|---|---|---|---|---|
| B1 | `feat(iam): seed the permissions of supplier payments` | 7.1 | — | `32ff123d` (V105: los tres permisos y sus nueve marcas, en SA, AE y CO) |
| B2 | `feat(procurement): mark the suppliers of the dealer channel` | 7.2 | — | `3b463a6d` (V106) |
| B3 | `feat(procurement): create the schema of supplier payments, applications and allocations` | 7.3 | Fases 2 y 4 integradas | `fd15e8cf` (V107: las tablas, sus restricciones, sus disparadores, Row Level Security, los permisos de la cuenta y `data_versions`) |
| B4 | `feat(procurement): model what is owed, the supplier balance and the five amounts of a unit` | 7.4 | — | `3a1596b4` |
| B5 | `feat(procurement): model the supplier payment with its applications and allocations` | 7.4 | B4 | `25eb8f69` |
| B6 | `feat(procurement): persist supplier payments and translate their races` | 7.4 | B3, B5 | `5ff0c76d` |
| B7 | `feat(procurement): read the payable facts of invoices, lines, notes and payments in batches` | 7.5 | B3 | `ca56b0fa` |
| B8 | `feat(procurement): register a supplier payment once per request with its rates and applications` | 7.5 | B1, B6, B7 | `a86065d6` |
| B9 | `feat(procurement): allocate payments to units and settle the units of the dealer channel` | 7.5 | B8 | `063ce0f2` |
| B10 | `feat(procurement): pay with the supplier balance under the lock of the supplier` | 7.5 | B8 | `ef015ffd` |
| B11 | `feat(procurement): keep an invoice with payments from annulment and fix the decimals of a paid currency` | 7.5 | B8 | `ecc963ec` |
| B12 | `test(procurement): fix the statements of the registration of a payment` | 7.5 | B9, B10 | `195d8689` |
| B13 | `feat(procurement): annul a supplier payment unless the balance goes negative` | 7.6 | B10 | `4ac9730b` |
| B14 | `feat(procurement): answer the annulment of a credit note with the supplier balance` | 7.6 | B13, puerto de la fase 4 | `64339927` |
| B15 | `feat(procurement): move the payments of an order to its invoice when they link` | 7.7 | B8 | `68528670` |
| B16 | `feat(procurement): keep an order with payments from annulment and fulfilment` | 7.7 | B8 | `94add284` |
| B17 | `feat(procurement): list, read and export supplier payments with their allowed actions` | 7.8 | B13 | `b0330d09` |
| B18 | `feat(procurement): show the balance and the payables of a supplier` | 7.8 | B7 | `3a4b7a33` |
| B19 | `feat(procurement): show the payment position of an invoice and add the payment to the purchase cost of a unit` | 7.8 | B7, B9 | `56f992c8`, que corrige también el costo de compra de una unidad de una factura en la moneda de la empresa (§4.2.11) |
| B20 | `feat(fleet): show the payment status and the day gap on the record of a unit` | 7.9 | B19 | `1ea73d0d` |

**Frontend**

Antes de F1, `328c507f` llevó la pantalla «En construcción» de las e2e a Cargas de archivos, como la fase 4 la había llevado a Pagos a proveedores.

| # | Commit | Paso | Depende de | Hecho en |
|---|---|---|---|---|
| F1 | `feat(iam): place the payment permissions on the matrix` | 7.10 | B1 | `278d6bfe` |
| F2 | `feat(procurement): mark the suppliers of the dealer channel` | 7.2 | B2 | `605b4343` |
| F3 | `feat(procurement): model supplier payments and their amounts in integers` | 7.10 | Documentación 1 | `94b411de` |
| F4 | `feat(procurement): add the API, the store and the facade of supplier payments` | 7.10 | F3 | `806e57a7`, y `29246002`: los identificadores de las muestras de pagos sin llamar al módulo de la API al cargar. Sin fachada: ningún otro contexto consume los pagos (§6.10) |
| F5 | `feat(procurement): add the payment form with its applications and totals` | 7.11 | F4, B8 | `2f0d3ef5` |
| F6 | `feat(procurement): allocate a payment to units and pay with the supplier balance` | 7.11 | F5, B9, B10 | `91c322c6` |
| F7 | `feat(procurement): list, read, annul and copy supplier payments` | 7.11 | F5, B13, B17 | `4c33bc63` |
| F8 | `feat(procurement): show the payments and what is owed on the invoice record` | 7.11 | F4, B19 | `0f96e2a1`, y `1056c3a7`: el desglose de lo que se debe en un grupo con nombre, por la regla `dlitem` de axe |
| F9 | `feat(procurement): show the balance and the payables on the supplier record` | 7.11 | F4, B18 | `13526ee1` |
| F10 | `feat(procurement): refuse annulling or fulfilling an order with payments` | 7.11 | B16 | `0c10fa14` |
| F11 | `feat(fleet): show the payment status, the gap and the payment of a unit in its purchase cost` | 7.11 | B19, B20 | `f2e3b686` |
| F12 | `feat(app-shell): open Logística › Pagos a proveedores` | 7.11 | F7 | `40322418` |
| F13 | `test(e2e): cover supplier payments on three breakpoints with axe` | 7.11 | F12 | `12916b03`, y `4d9c326b`: la casilla que aplica todas las filas, nombrada por su campo, por la regla `label` de axe |
| F14 | `test(live): probe supplier payments against the real backend` | 7.12 | todo | `03abb43d`, rehecha en `5999f719`; y `febf70b1`: las sondas de flota y de habilitación buscan «Estado» aparte de «Estado de pago». No corrió (§7.4) |

**Cierre (7.12), documentación**
2. `docs(procurement): close supplier payments with their rules and commits`:
   - `contextos/procurement.md`, con PRC-R100 a PRC-R139 en ✅ y `/suppliers/{id}/balance` como ruta;
   - `contextos/fleet.md`, con FLT-R130 a FLT-R132;
   - convenciones §14.19, con el pago en la ruta del costo de la unidad de la fase 4.
3. `docs(processes): mark P5 and the screen of supplier payments as built`:
   - procesos P5 y §3.2;
   - el plan;
   - las sondas.

Las filas 2 y 3 se hicieron el 03/11/2026, en varios commits de documentación, uno por documento: esta especificación, la de notas de crédito por el defecto que corrigió B19, las convenciones, los contextos con el mapa, los procesos con el plan, y el orden de implementación.

**Los commits que las tablas no tenían.**
- **Backend, después de la revisión:** `ad7f9983`, que lee la orden de la que pasó una aplicación con `ChangeLogHistoryLookup`, de `shared`, en lugar de nombrar `change_logs` en una consulta de `procurement` (§2.1); y `de8f9913`, que suma `lacking` a cada unidad de la posición de una factura (§4.2.8).
- **Frontend, el catálogo de la revisión 37 de `DESIGN.md`**, sin fila: `c4940b26`, y `0929c522` en la prueba visual (§6.11). Y `db2b9765`, una declaración por línea en las tablas del pago, como pide stylelint.
- **Frontend, lo que dejó la revisión del contrato** (§7.5): `a214ec87`, `8edfa99b`, `ab90f125` —reemplazado por `4e079f12`—, `206d9283`, `f2144160`, `d0040567`, `f8737a0d`, `15cde6e9`, `27601b5d` y `5999f719`.

**Migraciones:** B1, B2 y B3 llevan V(n), V(n+1) y V(n+2), numeradas al integrar en `develop` después de lo que hayan integrado las fases anteriores (plan §4). Son V105, V106 y V107 (§3.1).

**Como se integró (03/11/2026).** La fase 5 se integró después de las fases 2, 3 y 4, así que sus migraciones son V105 a V107 (§3.1). Merges en `develop`: backend `601cc177`, con la rama `feature/procurement-payments` hasta `de8f9913`, 22 commits desde `e213c01c`; y frontend `e0ed0f68`, con la rama hasta `4e079f12`, 33 commits desde `d579211a`, sin ningún commit más después de correr sus suites. Antes de integrar se revisó el contrato del frontend contra el del backend: una rotura del frontend y ocho diferencias menores, corregidas, y el backend sumó `lacking` (§7.5). El responsable decidió ese día diez puntos del contrato —el orden de las revisiones con el `requestId` después de los `400` del servicio y su reintento `200`; el `403` de la ruta justo después del `400` del marco en la anulación y en el costo de la unidad; el comprobante de retención que falta, `422`, y el de más de 30 caracteres, `400`; las formas de `context`; las filas de la lista como una sola vista; `PURCHASEORDER_HAS_PAYMENTS` en la anulación y en el cumplimiento, con su texto; la tasa que no es 1 en la moneda funcional, `EXCHANGE_RATE_CHANGED`; `registerPayment` del proveedor con toda la empresa y todas las marcas; los límites; y la anulación del pago como cambio delicado—, que las convenciones aplican (§2.2, §3, §4.2, §7.1 y §7.2). Lo que el código hizo distinto está en cada sección, con la marca **Como se construyó**: entre otras cosas, el disparador `keep_payment_allocations`; las excepciones y las piezas de lectura; los céntimos a la primera unidad que recibe parte; lo que se puede pagar en una moneda que no existe, vacío; `lacking`; y, en el frontend, «Le falta», el panel del proveedor bajo sus contactos, las facturas de un pago con saldo y los límites del formulario. La documentación de las filas 2 y 3 se hizo el mismo día. Ese día, después del cierre, dos pruebas de guarda —que un pago no cambia el valor de una unidad (PRC-R124) y que el cumplimiento de una orden no se cruza con un pago de ella (PRC-R127)— entraron en `develop` con backend `059a4457` (rama `feature/procurement-payment-guards`, `8c1a8c9f` y `29db26be`), sin cambios de código: pasan con las 120 pruebas de las clases que alcanzan. Falta la sonda en vivo (§7.4).

**Las correcciones de la revisión (04/11/2026)** no tienen fila en estas tablas: sus commits están en §10.1, y entraron con los merges `44136ae1` en el backend y `cc782649` en el frontend. La fecha base de cada unidad (R22), del mismo día, tampoco: entró con el merge `1bd767f1` del backend, de la rama `feature/day-gap-per-unit` con su único commit, `c43d1bc1` (§2.3, §10.1 fila 18).

---

## 9. Preguntas de la versión 1, decididas

Las doce preguntas de la versión 1 (03/11/2026) quedaron decididas el mismo día, por delegación del responsable, y se registran en el diccionario como D-207 y D-210 a D-213. Se aceptaron todas las recomendaciones salvo la P12, la de los números de las reglas de `fleet`. Esta versión las aplica, cada una donde corresponde.

| # | Pregunta de la versión 1 | Decisión |
|---|---|---|
| P1 | ¿§10.1 suma `exchange_rate_date`, `updated_at` y `updated_by`, que D-169 y D-198 ya exigen? | **D-210.** Sí: el cuadro de §10.1 los lleva, y sus dos frases vencidas quedaron corregidas (§3.4) |
| P2 | ¿Con qué tasa se convierte una aplicación cuando el pago está en la moneda funcional? | **D-210.** Cada aplicación guarda su tasa y su día: la de venta del día del pago entre la moneda del pago y la del documento, o 1 si son iguales (§2.4, PRC-R107) |
| P3 | ¿La retención va dentro de `amount` o se suma a él? | **D-210.** Se suma: `amount` es dinero o saldo y cuadra con el voucher, y lo cubierto es `amount` más la retención, que lleva su comprobante (§2.2, PRC-R115) |
| P4 | ¿Un pago puede quedar con una parte sin aplicar? | **D-210.** No: un pago se aplica completo, y uno que no lo hace responde `422 SUPPLIERPAYMENT_APPLIED_MISMATCH` (PRC-R109) |
| P5 | ¿Qué órdenes reciben pagos, y qué pasa al cumplirlas o desvincularlas? | **D-211.** Solo las enviadas, nunca de un proveedor de dealers, y sin tope por su subtotal. Una orden con pagos no se cumple ni se anula; lo aplicado a ella pasa a su factura solo en la misma moneda, y desvincular no devuelve nada (PRC-R125 a PRC-R127) |
| P6 | ¿Cómo se imputa: tope por unidad, reparto con imputación parcial y reasignación? | **D-211.** Lo imputado no pasa de lo que le falta a la unidad; lo no imputado se reparte en proporción a lo que le falta a cada una; reasignar es anular y copiar el pago (§2.3, PRC-R119, PRC-R120) |
| P7 | ¿Fecha no futura, número de operación por método, motivos de anulación y pagos a un proveedor inactivo? | **D-212.** La fecha no es futura; el número es obligatorio en transferencia y cheque y no va con saldo; los motivos son `ERROR_REGISTRO` y `DEVUELTO`; un proveedor inactivo sigue recibiendo pagos (PRC-R131, PRC-R132) |
| P8 | ¿Con qué alcance se registra y se anula un pago? | **D-212.** El de la factura: la empresa y la marca de cada línea que paga; con saldo o a una orden general, todas las marcas. Anular, con el mismo alcance (PRC-R102) |
| P9 | ¿El estado de pago se ve sin `pagos.ver`: en la factura con `compras.ver` y en la unidad con `stock.ver`? | **D-212.** Sí, sin montos, para que Operaciones sepa qué unidades retirar; los montos siguen a `pagos.ver` y a `stock.ver_costos` (PRC-R121, FLT-R132) |
| P10 | ¿Cómo se informa la diferencia de cambio, también la de un pago con saldo? | **D-213.** Por cada aplicación pagada con dinero, en la moneda funcional; con saldo es cero. Nunca toca el costo, y el contador la valida junto con la retención (§2.4, PRC-R124) |
| P11 | ¿La carga inicial trae los pagos de las columnas 42 a 44? | **D-207.** Sí: un pago por bloque de la columna 42, por transferencia y con ese código como número de operación, con la tasa del catálogo o la del Excel; en el canal de dealers, imputado por unidad y saldado (§2.10) |
| P12 | ¿Qué números FLT-R toma esta fase? | **Cambia la recomendación:** las cargas de archivos ya tomaron FLT-R100 a FLT-R126, así que esta fase toma FLT-R130 a FLT-R132 (§5.2) |

**Lo que sigue pendiente fuera de esta especificación:** la respuesta del contador sobre la retención. La primera versión solo la registra; si Millennium es agente de retención, el cálculo se agrega sin cambiar tablas (D-196).

---

## 10. Revisión de punta a punta (03 y 04/11/2026)

El 03/11/2026 el responsable pidió una revisión profunda y crítica del plan v4. La hicieron cuatro revisores de solo lectura, uno por fase, y cada hallazgo clave se verificó en el código. El 04/11/2026 el responsable ordenó resolverlos todos («Resuelve todos los hallazgos como experto»), y con esa orden se fijó el contrato C1 a C8 entre los dos repositorios. Las correcciones entraron en `develop` ese día: backend `44136ae1` (rama `feature/review-findings`, `5c77bc8f` a `df6b7276`) y frontend `cc782649` (rama `feature/review-findings`, `6386f2fa` a `1ef10642`). Ninguna migración nueva en esta fase.

### 10.1 Lo que encontró en esta fase y cómo se cerró

La gravedad mide el efecto sobre el dinero, los permisos o la pantalla: alta, un pago correcto no se podía registrar; media, un dato incorrecto o un permiso que se pasaba; baja, una molestia o una prueba más débil que su regla.

| # | Hallazgo | Gravedad | Cómo se cerró | Commits | Pruebas |
|---|---|---|---|---|---|
| 1 | Un pago en soles de una factura en dólares solo cuadraba con cerca de un cuarto de los montos: el equivalente redondeado hacia adelante no alcanzaba el del voucher (USD 2 659,57 a 3,760000 son PEN 9 999,98, nunca 10 000,00) | Alta | C1: las aplicaciones convertidas absorben el residuo, dentro de lo que al convertirse de vuelta da su monto; la diferencia de cambio lee el equivalente absorbido; el rechazo informa la suma hacia adelante (§2.4) | Backend `5c77bc8f`; frontend `20cebe6f`, `af8a3f16` | `PaymentEquivalentsTests` (V1 a V6); `SupplierPaymentsIntegrationTests.aPaymentInSolesAbsorbsTheResidueOfTheRounding`; e2e C1 (PRC-R107, R109, R124) |
| 2 | Un reintento con otra retención, otro comprobante, otras imputaciones u otras notas respondía `200` con el pago anterior; y uno legítimo, después de que la aplicación a la orden pasara a la factura, respondía `409` | Media | El contenido es todo el pago, y un destino coincide también con la orden de la que pasó, leída de la bitácora (§4.6) | Backend `bb355c8e` | `SupplierPaymentsIntegrationTests.theSameRequestWithAnotherWithholdingAllocationOrNotesAnswersRequestConflict`; `PurchaseInvoicesIntegrationTests.aRetryAfterTheApplicationToTheOrderPassedToTheInvoiceAnswersThePayment` (PRC-R129) |
| 3 | Los céntimos del reparto iban enteros a la primera unidad, que podía quedar bajo cero (−0,24) o por encima de lo que le falta | Media | C2: se mueven de a uno, sin pasar lo que falta ni bajar de cero (§2.3) | Backend `46f207b1`; frontend `f8efb41b` | `UnitCoveragePolicyTests.theCentsLeftNeverTakeAUnitBelowZeroNorAboveWhatItLacks` (PRC-R120) |
| 4 | Un proveedor con pagos en sus órdenes se podía marcar de dealers, y esos pagos pasarían a sus facturas sin las unidades que el canal exige | Media | `422 SUPPLIER_DEALER_ORDER_PAYMENTS`, con el proveedor bloqueado (§4.2.9) | Backend `06faa05b`; frontend `7d5e5dce` | `SupplierPaymentsIntegrationTests.aSupplierWithPaymentsOnItsOrdersIsNotMarkedOfTheDealerChannel` (PRC-R105, R125) |
| 5 | La bitácora mostraba a quien no lee pagos el monto que una aplicación dejó en su orden | Media | C6: el valor se oculta sin `pagos.ver`, con `valuesWithheld` (§2.6) | Backend `ad87b4d9`; frontend `c510e131` | `PurchaseInvoicesIntegrationTests.theChangeLogWithholdsTheAmountOfAnApplicationFromWhoDoesNotReadThePayments` (PRC-R126) |
| 6 | El formulario no contaba la retención contra lo que se debe: dejaba enviar lo que el backend rechaza, y «Aplicar lo que se debe» llenaba de más | Media | La fila juzga monto más retención; «Aplicar lo que se debe» resta la retención y, en un dealer, `settledInitials` (§6.3) | Frontend `c6c96b1e`; backend `fbd6f291` (prueba) | `SupplierPaymentsIntegrationTests.anAmountPlusItsWithholdingOverWhatIsOwedAnswersExceedsOwed`; `payment-application-form.spec`; e2e (PRC-R110) |
| 7 | El formulario ofrecía «Saldo a favor» y empresas donde el backend no acepta el registro: un rol de una marca, o de una sede | Media | C8: la sesión dice el alcance de cada permiso (`wholeCompanyPermissions`, `brandPermissions`), y el formulario ofrece solo lo que el backend acepta (§4.2.4, §6.3) | Backend `df6b7276`; frontend `3742271f`, `fdf14036`, `18e03774`, `64581af4`, `1ef10642` | `supplier-payment-form-view.spec`; `iam.store.spec`; e2e de un rol de una marca (PRC-R102) |
| 8 | «Anular y copiar» copiaba lo de entonces: documentos que ya no se deben, imputaciones por encima de lo que le falta hoy a la unidad, «Salda la unidad» fuera de un dealer y el número de operación de un pago vigente | Media | La copia se revisa contra lo de hoy y dice lo que cambió (§6.4) | Frontend `1e189bf0` | `supplier-payment-form-view.spec` («a copy (`?copiaDe=`)») (PRC-R133) |
| 9 | Pedir las tasas otra vez mientras se leía una perdía las que esperaban | Media | La cola conserva cada tasa pendiente | Frontend `b556fa26` | `payment-rates.spec` |
| 10 | Un rechazo no decía lo que se debía ni el saldo, no volvía a leerlos, y su marca quedaba en la fila aunque esta cambiara | Baja | El rechazo nombra lo que se debía o el saldo, vuelve a leer, y la marca se va cuando la fila cambia (§6.3) | Frontend `1e189bf0`, `be45bcfc`, `2b023ba4` | `supplier-payment-form-view.spec`; `payment-application-form.spec`; e2e |
| 11 | Una factura de dealers con todas sus unidades saldadas se ofrecía para pagar, aunque solo «debe» las iniciales y ningún pago la alcanza | Baja | C5: sale de lo que se ofrece pagar (§4.2.7) | Backend `96c62684`; frontend `a52c6a71` | `SupplierBalancesIntegrationTests.aDealerOffersNoOrderAndShowsTheInitialsOfItsSettledUnits`; e2e C5 (PRC-R101) |
| 12 | Un pago en efectivo sin número de operación se mostraba como «Saldo a favor» | Baja | Revisión 39: sin número, el nombre del método (§6.11) | Frontend `131314dc` | e2e |
| 13 | Los selectores de proveedor leían todos los proveedores de la empresa de una vez | Baja | Buscan en el servidor; el que nombra un enlace se lee por su id | Frontend `6c779bc3`, `3a995e95` | `supplier-payment-form-view.spec` |
| 14 | Un monto con ceros después de los decimales de su moneda se rechazaba en la pantalla y no en el backend; «Ver pagos» del proveedor abría la lista sin su empresa | Baja | Se aceptan; el enlace lleva la empresa | Frontend `6386f2fa`, `6205e8d4` | `purchase-invoice-amounts.spec`; `supplier-profile-view.spec` |
| 15 | Pruebas más débiles que sus reglas: dos anulaciones hechas con SQL, la retención sin prueba frente a lo que se debe ni en las imputaciones, y la fecha de mañana en Lima según la hora en que corría la suite | Baja | Anulan por la ruta; dos pruebas nuevas; `ProcurementCalendar` y `ProcurementDatesInLimaTests` a hora fija | Backend `fbd6f291`, `6165ba5d` | PRC-R110, R116, R118, R130, R132 (§5) |
| 16 | El objetivo de 1 s para 100 aplicaciones no se había medido | Baja | Línea base medida (§7.2) | Backend `b44950c8` | `SupplierPaymentRegistrationBaselineTests` |
| 17 | La tabla de candados decía que la factura toma las marcas de sus unidades nuevas en el candado 2 | Baja, de documentación | Corregida aquí y en la especificación de las facturas (§2.7) | — | — |
| 18 | La brecha de días de cada unidad contaba desde el primer pago aplicado a su factura, y no desde el pago del bloque que la incluye (guía, columnas 24 y 43): de 50 motos pagadas en dos bloques de 25, el segundo bloque contaba desde el pago del primero | Media | R22, aprobada por el responsable el 04/11/2026 con D-199 y §10.5 del diccionario v1.39: cada unidad toma el primer pago que la cubre, o la fecha de la factura; la posición de la factura conserva su primer pago (§2.3) | Backend `c43d1bc1`, merge `1bd767f1` | `UnitCoveragePolicyTests.eachUnitCountsItsGapFromTheFirstPaymentThatCoversIt`, `aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf`; `VehiclesIntegrationTests.eachUnitOfAnInvoicePaidInTwoBlocksCountsItsGapFromItsOwnPayment` (PRC-R123, FLT-R131) |

**Lo que no se cambió, por decisión.** El subtotal de la orden no es tope de lo que se le aplica: D-211 dice que no lo es, porque puede no llevar IGV. Sigue solo para mostrarlo (§4.2.7).

### 10.2 Lo que corrió el 04/11/2026

Backend: la suite completa, 3 570 pruebas, con 78 fallos, todos de la línea base del superusuario (las pruebas de RLS y de filas que nunca se borran, que necesitan las dos cuentas). Frontend: 3 285 pruebas unitarias en 275 archivos, stylelint limpio, la e2e completa de escritorio con 657 que pasan, 3 omitidas y ninguna falla, y axe limpio en los tres anchos de las pantallas que cambiaron; ninguna imagen de referencia se regeneró.

Con R22 (backend `c43d1bc1`, fila 18), pasaron las 117 pruebas que alcanza el cambio: las de `UnitCoveragePolicy`, las del adaptador de `UnitPurchases`, las de la ficha y el costo de compra de la unidad, las de los pagos, las de OpenAPI y `ModularityTests`. El frontend no cambió.

### 10.3 Lo que sigue abierto

- **La sonda en vivo**, `e2e-live/supplier-payments.live.spec.ts`: once pruebas, la que prepara la corrida y diez casos (§7.4). Nunca corrió: necesita la contraseña de la cuenta de verificación del responsable. Las de las fases 2, 3 y 4 tampoco.
- **El rendimiento:** la mediana de 100 aplicaciones, 899 ms, cumple el objetivo de 1 s; los máximos, 1 911 ms sin imputar y 1 373 ms con imputaciones, no (§7.2).
- **El CI con las dos cuentas:** las pruebas de RLS y de filas que nunca se borran de las tablas nuevas no han corrido en verde allí. Una sonda de SQL como `erp_mm_app` sobre `erp_mm_db_verify`, deshecha al terminar (04/11/2026), confirmó en las 11 tablas nuevas con empresa la RLS forzada, ningún permiso de `DELETE`, ningún `UPDATE` sobre `payment_allocations` y el aislamiento entre empresas.
- **El contador:** la validación de D-190, D-192, D-194 y D-196. En esta fase pesan lo que se debe (D-190), las monedas y la diferencia de cambio (D-194, D-213) y la retención (D-196); D-192 es de las notas.
- ~~**El diccionario:** las propuestas de esta fase y de la revisión esperan al responsable y no se aplicaron.~~ Aprobadas por el responsable y aplicadas el 04/11/2026: documentación `7cc29f4`, diccionario v1.39. Las que tocan los pagos son la fila 8 de §3, `suppliers.is_dealer` en §9.1 (el canal de dealers con pagos en sus órdenes), §9.2 (la orden con pagos no se anula ni se cumple), §10.1 (la regla del `request_id`), §10.2 (la aplicación exacta, el paso de la orden a su factura, el comprobante de retención y el equivalente que absorbe el redondeo), §10.3 (el reparto por lo que le falta a cada unidad, los céntimos de a uno, el tope y la imputación que no cambia), §10.5 (la brecha en la zona de la empresa y la fecha base de cada unidad, con D-199: fila 18 de §10.1) y §13.1 (la anulación de un pago, delicada, y los valores que la bitácora oculta).
