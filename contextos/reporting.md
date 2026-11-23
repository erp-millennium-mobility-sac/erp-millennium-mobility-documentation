# Contexto `reporting` — ERP Millennium Mobility

**Fuente:** diccionario de datos, D-232 a D-237, con D-239, D-248, D-280 y D-288; §10.5 y §11.9 del diccionario, P11 de [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), la guía funcional y la plantilla del cliente `STOCK_MILLENNIUM_2026_EXPORTABLE FINAL.xlsx`. Si este documento y el diccionario no coinciden, manda el diccionario.

La capa de lectura: los reportes que cruzan contextos. Lee por SQL y nunca escribe.

## 1. Qué es y qué no es

| Qué | Dónde | Permiso |
|---|---|---|
| Indicadores de la flota | Herramientas › Reportes, pestaña Indicadores | `reportes.ver` |
| Brecha de días por unidad | Pestaña Brecha de días | `reportes.ver` |
| Diferencia por inicial por unidad | Pestaña Diferencia por inicial | `reportes.ver` y `stock.ver_costos` |
| Exportable de stock, 52 columnas | Botón «Exportar stock» | `reportes.ver`; los montos, con `stock.ver_costos` |

`reportes.ver` lo siembra V120 en `SUPERADMIN`, `ADMIN_EMPRESA`, `OPERACIONES` y `CONTABILIDAD` (D-232).

**Dónde vive (D-232).** Un módulo `reporting`, capa de lectura: no escribe, no tiene tablas propias ni pantallas de escritura. Lee por SQL los datos de los contextos y pide a sus dueños, por sus fachadas, las reglas que tienen dueño, porque escribirlas de nuevo en SQL daría otro número y un reporte nunca debe decir otra cosa que la ficha de la unidad:

| Regla | Dueño | Por dónde llega |
|---|---|---|
| Estado de habilitación con lo que falta, entrega vigente, primera entrega, fecha base y brecha de días | `fleet` | `FleetContextFacade.fetchReportFacts` |
| Proveedor, factura, notas, valor del vehículo, valores en moneda funcional, pagos que alcanzan la unidad y la inicial congelada | `procurement`, que lo calcula | `ProcurementContextFacade.fetchUnitPurchaseCosts`, la fachada de `procurement` (D-280; antes de la fase 5 del plan de migración, por `fleet`) |

`reporting` depende solo de `shared`, `documents::acl`, `fleet::acl` y `procurement::acl`. Lo que llega de `fleet` lo traduce su `ExternalFleetService` a `ReportUnitFacts`, y lo que llega de `procurement`, su `ExternalProcurementService` a `ReportUnitPurchase`; ninguna otra clase nombra esos tipos. Cada consulta suya corre contra el esquema de Flyway en una prueba de integración (`ReportsIntegrationTests`).

**Queda fuera, y por qué:**
- **Costo por día paralizada:** el sistema no guarda ningún costo diario de una unidad detenida. Calcularlo sería inventar el dato (D-235).
- **Validación de la inicial contra el contrato:** cruza compras con ventas, y espera al bloque 12 (D-236, §10.5).
- **Pólizas (columnas 28 a 30 del exportable):** son candidato (§11.6). Las columnas salen vacías para conservar la plantilla.
- **Excel de la brecha y de la diferencia:** el exportable de stock ya las trae, columnas 24 y 45.
- **Moneda de grupo:** un reporte es de una empresa y se lee en su moneda funcional, y dos monedas no se suman (D-233), así que ningún reporte la usa. Queda pendiente para cuando haya dos monedas funcionales que reunir en un mismo reporte (D-46, §4.8).

## 2. Alcance de cada pedido (D-233)

Un reporte es de **una empresa**: `companyId` es obligatorio, porque de ella salen la zona y la moneda funcional, y dos monedas no se suman. `commercialBrandId` es opcional.

- La empresa se lee con `reportes.ver`; sin el permiso, `403 ACCESS_DENIED`, y una empresa que no existe o fuera de ese alcance, `404 COMPANY_NOT_FOUND`.
- Quien tiene `reportes.ver` acotado a marcas lee solo esas marcas.
- Las unidades anuladas no cuentan.
- Los montos siguen a `stock.ver_costos`, por empresa y marca de cada unidad y por el área de su factura, en una misma asignación (D-288). Sin ese permiso las columnas de montos salen vacías, y no salen de la base hacia la respuesta.
- **El área de la factura oculta la compra** (A6, D-288). Quien no lee con `reportes.ver` la empresa, la marca de la unidad y el área de su factura vigente, en una misma asignación, recibe vacías todas sus columnas de compra, también el número de la orden. Una unidad sin factura vigente se muestra como hoy. Fuera de eso, el reporte no mira el área: las unidades son de todas las áreas (A8).

## 3. Los reportes

### Indicadores (D-235)

`GET /api/v1/reports/indicators?companyId=&commercialBrandId=`

| Indicador | Cómo sale |
|---|---|
| **% de activación** | Activos / (Activos + Paralizadas) × 100, con un decimal. Activos: familia `productivo`. Paralizadas: familia `recuperacion`, es decir Mecánica, Siniestro, Custodia y Puesta a punto. Sin unidades en ninguna de las dos, no hay porcentaje (`FleetIndicatorsPolicy.activationRate`) |
| **Unidades por estado** | Cuenta por estado actual: cada estado activo, y uno inactivo que todavía tiene unidades |
| **Días promedio por estado** | Promedio de cada estadía en el estado, desde que la unidad entró hasta que salió, o hasta ahora si sigue ahí |
| **Tolerancia vencida** | Unidades cuya fila actual del historial ya pasó su `due_at` |
| **Próximas a tolerancia vencida** | Unidades cuya fila actual vence dentro de 72 horas (`FleetIndicatorsPolicy.WINDOW`) |
| **Próximas a resolución** | Unidades en Custodia con tolerancia vencida o por vencer dentro de 72 horas. Custodia es el único estado que lleva a «Resolución de contrato» |
| **En riesgo de pérdida** | Unidades en Custodia por Incautación, Comisaría o Depósito: una autoridad las tiene y Millennium no decide cuándo vuelven |

**Qué cuenta como paralizada.** Solo la familia `recuperacion`. Robado, Pérdida total, En traspaso e Inactivo no son paralizadas: son de familia `terminal`, no entran en el porcentaje y no bajan la activación (D-219, D-220, D-235, D-239). Sí se cuentan en Unidades por estado.

### Brecha de días

`GET /api/v1/reports/day-gaps?companyId=&commercialBrandId=&q=&delivered=&page=&size=&sort=`

Unidades con línea vigente. La brecha es la de D-199, que calcula `fleet`: de la fecha base —el primer pago que cubre parte de la unidad o, si ninguno la cubre, la fecha de la factura— a la primera entrega o, sin entrega, a hoy en la zona de la empresa. Ordena por `dayGap`, por defecto de mayor a menor, `baseDate` o `plate`. `q` busca en placa y VIN. `delivered` deja las entregadas alguna vez (`true`) o las que nunca se entregaron (`false`). La página se corta en memoria, porque lo que la ordena sale de reglas que no se escriben dos veces (D-236).

### Diferencia por inicial (D-236, D-248)

`GET /api/v1/reports/initial-differences?companyId=&commercialBrandId=&q=&page=&size=`
`GET /api/v1/reports/initial-differences/totals?companyId=&commercialBrandId=&q=`

Las unidades saldadas del canal de dealers (D-197), con su inicial del cliente mayor que cero, de mayor a menor. La inicial es la congelada al saldar, en la imputación que salda (`payment_allocations.customer_initial`), y una nota posterior no la achica: lo que la nota baja pasa a saldo a favor del proveedor (D-248). Así, un auto de 50 000,00 saldado con 42 000,00 de Millennium y 8 000,00 de inicial sigue con 8 000,00 de inicial aunque después llegue una nota de 10 000,00, que pasa entera a saldo.

Exige `stock.ver_costos` en la empresa: si no, `403 ACCESS_DENIED`. Las unidades de una marca que ese permiso no lee no aparecen, ni las de una factura de un área que no lee en la misma asignación que la marca (D-288). Los totales van por moneda, con cuántas unidades y cuánto suma cada una. Como la brecha, corta su página en memoria (D-236).

### Exportable de stock (D-234)

`GET /api/v1/reports/stock?companyId=&commercialBrandId=` con `Accept` de Excel: un documento generado (D-106), en `303` hacia `/api/v1/documents/{id}`, reutilizado si es idéntico; con demasiados documentos en curso, `503 DOCUMENTS_BUSY`. Una fila por unidad no anulada de la empresa, con los encabezados y el orden exactos de la plantilla, hoja `STOCK`, en cualquier idioma. El día de la empresa entra en el pedido, porque la brecha cambia de un día a otro sin que nada más cambie, y el alcance incluye `stock.ver_costos`: quien no ve montos nunca recibe el archivo de quien sí.

| # | Columna | Sale de |
|---|---|---|
| 1 | N° | `legacy_number`; vacío en una unidad que no vino del Excel |
| 2 | LLAVE | Grupo de la llave duplicada; `NO TIENE` sin duplicado |
| 3, 4, 6 | PLACA, VIN, MOTOR | La unidad |
| 5 | ESTADO DE HABILITACIÓN | §11.9, con lo que falta entre paréntesis: «Sin equipo (GPS)» |
| 7 a 10 | MARCA, MODELO, COLOR, AÑO | El catálogo y la unidad |
| 11, 35, 36 | PROVEEDOR, FACTURA, FECHA DE FACTURA | La línea vigente |
| 12 | DUPLICADO DE LLAVES | Su serial; `SI` sin serial; `NO` sin duplicado |
| 13 | EN PLANTA | `SI` con fecha de ingreso |
| 14, 15 | INMATRICULACIÓN / TÍTULO, FECHA DE TÍTULO | La unidad |
| 16, 17 | TIVE, PLACAS | `SI` con documento vigente y con placas físicas |
| 18, 19 | GPS, FECHA | Instalación abierta |
| 20, 21 | PANTALLA, FECHA | `NO CORRESPONDE` si el tipo no la lleva; `SI` y `DE FÁBRICA` si el modelo la trae; si no, su instalación abierta. Lo decide el catálogo, nunca un nombre |
| 22 | ENTREGADO | `SI` mientras la entrega sigue vigente (D-216) |
| 23 | FECHA DE PRIMERA ENTREGA | La primera, también en una unidad recolocada |
| 24 | BRECHA DE DÍAS DE HABILITACIÓN | D-199 |
| 25 a 27 | SOAT y sus fechas | Documento vigente |
| 28 a 30 | PÓLIZA y sus fechas | Siempre vacías (§11.6) |
| 31, 32 | SAT, FECHA DECLARACIÓN SAT | La última declaración cargada |
| 33 | GARANTÍA MOBILIARIA | `SI` con cesión abierta, leída de las cesiones de `financing` como modelo de lectura |
| 34 | ORDEN DE COMPRA | La orden de la factura |
| 37 | PRECIO DE FACTURA | Monto |
| 38 a 40 | NC / ND, FECHA, VALOR | Notas vigentes: números unidos por `/`, la primera fecha, el total (monto) |
| 41 | VALOR DEL VEHÍCULO | §9.6 (monto) |
| 42, 43 | OPERACIÓN DE PAGO, FECHA DE PAGO | Pagos vigentes que alcanzan la unidad: sus operaciones unidas por `/`, y la fecha del primero (guía) |
| 44 | MONTO DE PAGO | Suma de esos pagos completos, una vez cada uno, repetida en las filas del bloque (guía); si están en dos monedas, cada una por separado (monto) |
| 45 | DIFERENCIA POR INICIAL | La inicial del cliente congelada al saldar, que una nota posterior no achica (D-248); vacía si la unidad no está saldada con inicial (monto) |
| 46, 47 | MONEDA, TC | La factura |
| 48, 49 | PRECIO DE COMPRA PEN, VALOR CONTABLE PEN | En la moneda funcional de la empresa (D-159): el valor del vehículo y su costo sin impuesto (monto) |
| 50 | PROPIETARIO | A quién se traspasó (D-223); sin traspaso registrado, la razón social de la empresa, que es quien figura en el registro. Una unidad En traspaso dice la empresa hasta que se registra su traspaso (D-239) |
| 51 | DECLARACIÓN DE STOCK | Última entrada a Disponible, así una unidad recolocada dice cuándo volvió a estar lista |
| 52 | TIPO DE VEHÍCULO | El catálogo |

**Qué sale vacío.** Las columnas 28 a 30, siempre. Las de compra (11 y 35 a 49), en una unidad sin línea vigente y, desde D-288, en una cuya factura es de un área que quien exporta no lee con `reportes.ver` en la marca de la unidad. Los montos (37, 40, 41, 44, 45, 48 y 49), para quien no tiene `stock.ver_costos` en la empresa, la marca de la unidad y el área de su factura, en una misma asignación. El diseño de las columnas no cambió, y el alcance del documento ya distingue la asignación con área (D-287): quien lee un área nunca recibe el archivo de quien lee otra. Las filas van por N° y VIN.

**Versión del documento (D-237).** V121 versiona en `data_versions` `vehicle_keys`, `vehicle_tax_declarations` y `vehicle_fund_assignments`, porque el exportable las muestra y un cambio en ellas debe cambiar la versión del documento (regla 6.4). El diseño de las columnas tiene además su propia versión (`STOCK_LAYOUT`, hoy `stock-columns=2`, en `ReportingDocumentKinds`): cambiar cómo se lee una columna la sube, para que no se sirva un archivo generado con la regla anterior.

### Pantalla

Ruta `/reports`, carpeta `reporting`, opción Herramientas › Reportes. Permiso `reportes.ver` en la ruta y en el menú. Arriba, el filtro de empresa (`mm-company-filter`, que ofrece solo las empresas donde la sesión tiene `reportes.ver`), el de marca comercial y «Exportar stock» a la derecha. Tres pestañas: Indicadores, Brecha de días y Diferencia por inicial, que se ofrece solo a quien tiene `stock.ver_costos` en la empresa elegida.

**La unidad desde su VIN.** En las listas de la brecha de días y de la diferencia por inicial, el VIN de cada fila abre la ficha de la unidad, que una unidad sin placa también tiene. El enlace aparece solo a quien tiene `stock.ver` en la empresa del reporte; a quien lee los reportes sin leer las unidades, el VIN es texto, porque un enlace que no podría abrir sería un callejón (`DESIGN.md`, revisión 87). Los cuatro roles del sistema con `reportes.ver` tienen `stock.ver`, así que la condición cuenta solo para un rol propio.

## 4. Reglas

Las pruebas del backend son del módulo `reporting`, salvo `ModularityTests`, que es de la raíz; las de la pantalla son `reports-view.spec.ts` y `e2e/reports.spec.ts`.

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| RPT-R01 | `reporting` no escribe, y depende solo de `shared`, `documents::acl`, `fleet::acl` y `procurement::acl` | D-232, D-280 | Servicio | `ModularityTests` | ✅ |
| RPT-R02 | Las reglas con dueño se piden a su dueño: a `fleet::acl`, la habilitación, la entrega vigente y la brecha; a `procurement::acl`, el costo de compra, sus notas y sus pagos | D-232, D-280 | Servicio | `ReportsIntegrationTests` | ✅ |
| RPT-R03 | Un reporte es de una empresa que el llamante lee con `reportes.ver`; si no, `404 COMPANY_NOT_FOUND`. Una asignación acotada a marcas lee solo esas | D-233 | Servicio | `ReportsIntegrationTests` | ✅ |
| RPT-R04 | Las unidades anuladas no cuentan | D-233 | Servicio | `ReportsIntegrationTests` | ✅ |
| RPT-R05 | El exportable conserva los 52 encabezados y el orden de la plantilla, en cualquier idioma | D-234 | Servicio | `ReportsIntegrationTests` | ✅ |
| RPT-R06 | Los montos siguen a `stock.ver_costos` por empresa, marca y área de la factura, en una misma asignación, y ese permiso entra en el alcance del documento | D-234, D-288 | Servicio | `ReportsIntegrationTests`, `StockRowsTests`, `PurchaseCostBrandCharacterizationTests` | ✅ |
| RPT-R07 | Robado, Pérdida total, En traspaso e Inactivo no cuentan como paralizadas: son de familia terminal. Sí se cuentan en Unidades por estado | D-235, D-239 | Dominio, Servicio | `FleetIndicatorsPolicyTests`, `ReportsIntegrationTests` | ✅ |
| RPT-R08 | La diferencia por inicial exige `stock.ver_costos` en la empresa y totaliza por moneda | D-236 | Servicio, Frontend | `ReportsIntegrationTests`, `reports.spec.ts` | ✅ |
| RPT-R09 | Cada tabla que muestra un documento tiene sus casillas en `data_versions` | D-237 | Base | Ninguna nombra las tres tablas que se versionan para el exportable | ⚠️ |
| RPT-R10 | La diferencia por inicial es la inicial del cliente congelada al saldar, la de la imputación que salda, que una nota posterior no achica: «Diferencia por inicial» lista los saldados con inicial mayor que cero, y la columna 45 del exportable la lee. El diseño del exportable va en su versión 2. La inicial llega de `procurement::acl`, que la responde congelada | D-248, D-236, D-280 | Servicio | `ReportsIntegrationTests` | ✅ |
| RPT-R11 | El área de la factura de una unidad oculta su compra (A6): sin `reportes.ver` en su empresa, su marca y esa área en una misma asignación, el stock deja vacías sus columnas de compra y el número de su orden (`StockRows.of` con `seesPurchase`), y la diferencia por inicial no la muestra sin `stock.ver_costos` con la misma regla. La compra llega con su área de `procurement::acl` (`UnitPurchaseCost.areaId`, en `ReportUnitPurchase`). La brecha de días no mira el área: no muestra montos ni números de compra | D-288, A6 | Servicio | Sin prueba propia del área en `reporting`: la cubren `ProcurementAreaIsolationIntegrationTests` para el costo y `PurchaseCostBrandCharacterizationTests` para el stock sin área | ⚠️ |

## 5. Relación con otros contextos

- **`fleet::acl`:** `FleetContextFacade.fetchReportFacts(companyId)`, los hechos de cada unidad de la empresa (sección 1).
- **`procurement::acl`:** `ProcurementContextFacade.fetchUnitPurchaseCosts(vehicleIds)`, la compra de cada unidad con línea vigente, por su `ExternalProcurementService` (D-280, fase 5 del plan de migración; antes la leía por `fleet`, que la obtenía de `procurement` por su puerto `UnitPurchases`). `ReportsIntegrationTests` fija las columnas de compra y de pago del stock, la diferencia por inicial y la brecha, celda por celda, con `stock.ver_costos` en todas las marcas y en una (`thePurchaseColumnsOfTheReportsComeFromTheCostOfEachUnit`). `ReportingDataVersions` no cambió: ya nombraba las tablas de compras. Desde D-288 cada compra trae el área de su factura (`ReportUnitPurchase.areaId`), con la que el stock y la diferencia por inicial deciden qué se ve.
- **Las cesiones de `financing`:** la columna 33 lee `vehicle_fund_assignments` por SQL, como modelo de lectura (*Conformist*), porque `financing` no tiene fachada (D-279). `ReportsIntegrationTests` la prueba: `SI` con una cesión abierta, `NO` sin cesión o con la cesión cerrada.
- **`documents::acl`:** el exportable de stock es un documento generado (D-106), con su tipo `stock` en `ReportingDocumentKinds` y la versión de sus datos por `ReportingDataVersions`.
- **`shared`:** el alcance de quien llama (`CurrentAccessScope`) y los permisos de `PermissionCodes`.
- **Frontend:** la pantalla usa `mm-company-filter` de `organization`, las marcas de la empresa por `OrganizationFacade` y los permisos de la sesión por `IamFacade` (`hasPermissionIn`).

Ningún contexto depende de `reporting`.

## 6. Estado y pendientes

- **RPT-R09**, sin prueba propia.
- **RPT-R11**, sin prueba propia en `reporting` de las columnas que vacía el área: `ReportsIntegrationTests` y `StockRowsTests` no cambiaron con la fase 7 del plan de áreas.
- **La sonda en vivo:** `e2e-live/` no tiene `reports.live.spec.ts`. La pantalla tiene su prueba contra el simulador del backend (`e2e/reports.spec.ts`), y las consultas, su prueba de integración contra el esquema de Flyway.
- **La validación de la inicial contra el contrato:** espera al bloque 12 (D-236).
- **La moneda de grupo:** para cuando haya dos monedas funcionales que reunir en un mismo reporte (sección 1).

## 7. Preguntas abiertas

Ninguna.
