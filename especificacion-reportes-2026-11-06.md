# Especificación: Reportes (paso 11)

**Versión 3 · 10/11/2026** (la inicial del cliente congelada al saldar, D-248 del diccionario v1.53, decidida ese día y por construir: §6 y la columna 45 de §7). **Versión 2 · 09/11/2026** (En traspaso, D-239: un estado más de la familia terminal, en §4 y en la columna 50; la moneda de grupo, en §1; y la sonda en vivo que falta, en §9). Versión 1 · 06/11/2026. Decisiones delegadas por el responsable: D-232 a D-237 del diccionario. Lee de §10.5, §11.9, P11 de procesos, la guía funcional y la plantilla `STOCK_MILLENNIUM_2026_EXPORTABLE FINAL.xlsx`.

## 1. Alcance

| Qué | Dónde | Permiso |
|---|---|---|
| Indicadores de la flota | Logística › Reportes, pestaña Indicadores | `reportes.ver` |
| Brecha de días por unidad | Pestaña Brecha de días | `reportes.ver` |
| Diferencia por inicial por unidad | Pestaña Diferencia por inicial | `reportes.ver` y `stock.ver_costos` |
| Exportable de stock, 52 columnas | Botón «Exportar stock» | `reportes.ver`; los montos, con `stock.ver_costos` |

**Queda fuera, y por qué:**
- **Costo por día paralizada:** el sistema no guarda ningún costo diario de una unidad detenida. Calcularlo sería inventar el dato.
- **Validación de la inicial contra el contrato:** cruza compras con ventas, y el bloque 12 no existe (§10.5).
- **Pólizas (columnas 28 a 30):** son candidato (§11.6). Las columnas salen vacías para conservar la plantilla.
- **Excel de la brecha y de la diferencia:** el exportable de stock ya las trae, columnas 24 y 45.
- **Moneda de grupo** (09/11/2026): D-46 y §4.8 la mandaban con este paso. Un reporte es de una empresa y se lee en su moneda funcional, y dos monedas no se suman (D-233), así que ningún reporte la usa. Queda decidida fuera del alcance de los reportes, y pendiente para cuando haya dos monedas funcionales que reunir en un reporte.

## 2. Dónde vive (D-232)

Un módulo `reporting`, capa de lectura: no escribe y no tiene pantallas de escritura. Resuelve la decisión pendiente 3 con un ajuste. Lee por SQL los datos, como proponía el mapa, y pide a `fleet::acl` las reglas que tienen dueño, porque escribirlas de nuevo en SQL daría otro número:

| Regla | Dueño | Por dónde llega |
|---|---|---|
| Estado de habilitación, entrega vigente, primera entrega y brecha de días | `fleet` | `FleetContextFacade.fetchReportFacts` |
| Valor del vehículo, valores en moneda funcional, pagos de la unidad y diferencia | `procurement` | `FleetContextFacade.fetchPurchaseCosts`, por el puerto `UnitPurchases` |

`reporting` depende solo de `shared`, `documents::acl` y `fleet::acl`. Lo que llega de `fleet` lo traduce su `ExternalFleetService` a registros propios, y ninguna otra clase nombra esos tipos. Cada consulta suya corre contra el esquema de Flyway en una prueba de integración.

## 3. Alcance de cada pedido (D-233)

Un reporte es de **una empresa**: `companyId` es obligatorio, porque de ella salen la zona y la moneda funcional, y dos monedas no se suman. `commercialBrandId` es opcional.

- La empresa se lee con `reportes.ver`; fuera de ese alcance responde `404 COMPANY_NOT_FOUND`.
- Quien tiene `reportes.ver` acotado a marcas lee solo esas marcas.
- Las unidades anuladas no cuentan.
- Los montos siguen a `stock.ver_costos`, por empresa y marca de cada unidad. Sin ese permiso las columnas de montos salen vacías, y no salen de la base hacia la respuesta.

## 4. Indicadores (D-235)

`GET /api/v1/reports/indicators?companyId=&commercialBrandId=`

| Indicador | Cómo sale |
|---|---|
| **% de activación** | Activos / (Activos + Paralizadas) × 100, con un decimal. Activos: familia `productivo`. Paralizadas: familia `recuperacion`, es decir Mecánica, Siniestro, Custodia y Puesta a punto. Sin unidades en ninguna de las dos, no hay porcentaje |
| **Unidades por estado** | Cuenta por estado actual, con todos los estados en uso |
| **Días promedio por estado** | Promedio de cada estadía en el estado, desde que la unidad entró hasta que salió, o hasta ahora si sigue ahí |
| **Tolerancia vencida** | Unidades cuya fila actual ya pasó su `due_at` |
| **Próximas a tolerancia vencida** | Unidades cuya fila actual vence dentro de 72 horas |
| **Próximas a resolución** | Unidades en Custodia con tolerancia vencida o por vencer dentro de 72 horas. Custodia es el único estado que lleva a «Resolución de contrato» |
| **En riesgo de pérdida** | Unidades en Custodia por Incautación, Comisaría o Depósito: una autoridad las tiene y Millennium no decide cuándo vuelven |

**Robado, Pérdida total, En traspaso e Inactivo no son paralizadas:** son de familia `terminal`, y no entran en el porcentaje (D-219, D-220, D-239). Sí se cuentan en Unidades por estado. En traspaso existe desde V123 (09/11/2026, backend `9f9bd093`), y `ReportsIntegrationTests.theRateCountsTheActiveOverTheActiveAndTheStopped` lo cuenta así.

## 5. Brecha de días

`GET /api/v1/reports/day-gaps?companyId=&commercialBrandId=&q=&delivered=&page=&size=&sort=`

Unidades con línea vigente. La brecha es la de D-199: de la fecha base a la primera entrega o, sin entrega, a hoy en la zona de la empresa. Ordena por `dayGap`, por defecto de mayor a menor, `baseDate` o `plate`. `q` busca en placa y VIN. `delivered` deja las entregadas alguna vez o las que nunca se entregaron.

## 6. Diferencia por inicial (D-236)

`GET /api/v1/reports/initial-differences?companyId=&commercialBrandId=&q=&page=&size=`
`GET /api/v1/reports/initial-differences/totals?companyId=&commercialBrandId=&q=`

Solo unidades saldadas del canal de dealers cuya diferencia es `INICIAL_DEL_CLIENTE` (D-197), de mayor a menor. Exige `stock.ver_costos` en la empresa: si no, `403 ACCESS_DENIED`. Las unidades de una marca que el permiso no lee no aparecen. Los totales van por moneda.

**Como cambia (10/11/2026, D-248)** (decidida en el diccionario v1.53; se construye en el backend con `feature/unit-level-balance`, con V127). Hasta ahora la inicial era lo que le faltaba a la unidad saldada, así que una nota posterior la achicaba: con un auto de 50 000,00 saldado con 42 000,00 de Millennium y 8 000,00 de inicial, una nota de 10 000,00 la dejaba en 0,00 y la unidad salía del reporte. Desde D-248 la inicial se congela al saldar, en la imputación que salda (`payment_allocations.customer_initial`), y una nota posterior no la cambia: lo que la nota baja pasa a saldo a favor del proveedor. El reporte lista las unidades saldadas cuya inicial congelada es mayor que cero, con esa inicial (`ReportQueryServiceImpl`), y sus totales suman esas iniciales. Las imputaciones que ya saldaban la toman de V127, que la rellena con el valor de hoy; en `dev`, si hubo una nota después de saldar, la inicial rellenada ya viene reducida. Lo prueba `ReportsIntegrationTests`, con un saldado y una nota posterior que conserva su inicial.

## 7. Exportable de stock (D-234)

`GET /api/v1/reports/stock?companyId=&commercialBrandId=` con `Accept` de Excel: un documento generado (D-106), reutilizado si es idéntico. Encabezados y orden exactos de la plantilla, hoja `STOCK`. El día de la empresa entra en el pedido, porque la brecha cambia de un día a otro sin que nada más cambie, y el alcance incluye `stock.ver_costos`: quien no ve montos nunca recibe el archivo de quien sí.

| # | Columna | Sale de |
|---|---|---|
| 1 | N° | `legacy_number`; vacío en una unidad que no vino del Excel |
| 2 | LLAVE | Grupo de la llave duplicada; `NO TIENE` sin duplicado |
| 3, 4, 6 | PLACA, VIN, MOTOR | La unidad |
| 5 | ESTADO DE HABILITACIÓN | §11.9, con lo que falta entre paréntesis |
| 7 a 10 | MARCA, MODELO, COLOR, AÑO | El catálogo y la unidad |
| 11, 35, 36 | PROVEEDOR, FACTURA, FECHA DE FACTURA | La línea vigente |
| 12 | DUPLICADO DE LLAVES | Su serial; `SI` sin serial; `NO` sin duplicado |
| 13 | EN PLANTA | `SI` con fecha de ingreso |
| 14, 15 | INMATRICULACIÓN / TÍTULO, FECHA DE TÍTULO | La unidad |
| 16, 17 | TIVE, PLACAS | `SI` con documento vigente y con placas físicas |
| 18, 19 | GPS, FECHA | Instalación abierta |
| 20, 21 | PANTALLA, FECHA | `NO CORRESPONDE` si el tipo no la lleva; `SI` y `DE FÁBRICA` si el modelo la trae; si no, su instalación abierta |
| 22 | ENTREGADO | `SI` mientras la entrega sigue vigente (D-216) |
| 23 | FECHA DE PRIMERA ENTREGA | La primera, también en una unidad recolocada |
| 24 | BRECHA DE DÍAS DE HABILITACIÓN | D-199 |
| 25 a 27 | SOAT y sus fechas | Documento vigente |
| 28 a 30 | PÓLIZA y sus fechas | Vacías |
| 31, 32 | SAT, FECHA DECLARACIÓN SAT | La última declaración cargada |
| 33 | GARANTÍA MOBILIARIA | `SI` con cesión abierta |
| 34 | ORDEN DE COMPRA | La orden de la factura |
| 37 | PRECIO DE FACTURA | Monto |
| 38 a 40 | NC / ND, FECHA, VALOR | Notas vigentes: números unidos por `/`, la primera fecha, el total (monto) |
| 41 | VALOR DEL VEHÍCULO | §9.6 (monto) |
| 42, 43 | OPERACIÓN DE PAGO, FECHA DE PAGO | Pagos vigentes que alcanzan la unidad: unidos por `/`, y la fecha del primero (guía) |
| 44 | MONTO DE PAGO | Suma de esos pagos completos, repetida en las filas del bloque (guía) (monto) |
| 45 | DIFERENCIA POR INICIAL | Solo si es `INICIAL_DEL_CLIENTE` (monto). Desde D-248 (decidida el 10/11/2026; se construye con V127), la inicial congelada al saldar, que una nota posterior no achica (§6); el exportable sube a la versión 2 de su diseño (`STOCK_LAYOUT`), para que no se sirva un archivo generado con la regla anterior |
| 46, 47 | MONEDA, TC | La factura |
| 48, 49 | PRECIO DE COMPRA PEN, VALOR CONTABLE PEN | En la moneda funcional de la empresa (D-159) (monto) |
| 50 | PROPIETARIO | A quién se traspasó (D-223); sin traspaso, la razón social de la empresa. Una unidad En traspaso dice la empresa hasta que se registra su traspaso (D-239) |
| 51 | DECLARACIÓN DE STOCK | Última entrada a Disponible, así una unidad recolocada dice cuándo volvió a estar lista |
| 52 | TIPO DE VEHÍCULO | El catálogo |

Una unidad sin línea vigente deja vacías las columnas de compra. Las filas van por N° y VIN.

**Versión del documento (D-237).** V121 versiona `vehicle_keys`, `vehicle_tax_declarations` y `vehicle_fund_assignments`, que el exportable muestra (regla 6.4). V120 siembra `reportes.ver`, que la matriz listaba sin sembrar.

## 8. Pantalla

Ruta `/reports`, carpeta `reporting`. Permiso `reportes.ver` en ruta y menú. Filtro de empresa arriba, con marca comercial, y «Exportar stock» a la derecha. Tres pestañas. Diferencia por inicial solo se ofrece a quien tiene `stock.ver_costos` en la empresa elegida.

## 9. Cómo se construyó

Backend `e5bcc028`, `c69f097d`, `ff52ea1e`, `486428cd`, `85f23e2e`, `bd37d618`, `4f62a26e` y `d68e3b24`, frontend `90a0a7ee`, `ab142f03`, `2896de23` y `85520b76`, migraciones V120 y V121. Pruebas: `ReportsIntegrationTests` corre cada consulta contra el esquema de Flyway y compara los encabezados con la plantilla real; `e2e/reports.spec.ts` recorre la pantalla en los tres anchos. No hay sonda en vivo: `e2e-live/` no tiene `reports.live.spec.ts` (09/11/2026), y queda pendiente con las demás que nunca corrieron (orden de implementación, pendientes transversales).
