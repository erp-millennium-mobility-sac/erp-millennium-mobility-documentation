# Contexto `reporting` — ERP Millennium Mobility

**Actualizado:** 10/11/2026 (la inicial del cliente congelada al saldar, D-248 del diccionario v1.53, decidida ese día y por construir: RPT-R10, en ❌ hasta construirla, y lo que queda). Antes, 09/11/2026 (En traspaso, D-239, que los indicadores cuentan como terminal y el exportable dice de quién es, con RPT-R07 al día: backend `9f9bd093`, en `feature/fleet-transfer-and-delivery`, integrada en `integration/audit-2026-11-09`, merge `9627e4b8`; las pruebas de cada regla; la moneda de grupo, decidida fuera de los reportes; y la sonda en vivo, que no existe). Antes, 06/11/2026 (backend `e5bcc028`, `c69f097d`, `ff52ea1e`, `486428cd`, `85f23e2e`, `bd37d618`, `4f62a26e` y `d68e3b24`, frontend `90a0a7ee`, `ab142f03`, `2896de23` y `85520b76`, migraciones V120 y V121).

La capa de lectura: los reportes que cruzan contextos. Lee por SQL y nunca escribe. Lo que hace y por qué está en la [especificación](../especificacion-reportes-2026-11-06.md); las decisiones son D-232 a D-237.

## Reglas

Las pruebas del backend están en `src/test/java/.../reporting` salvo que se diga otro paquete; la de la pantalla, en `e2e/reports.spec.ts`.

| ID | Regla | Fuente | Prueba | Estado |
|---|---|---|---|---|
| RPT-R01 | `reporting` no escribe, y depende solo de `shared`, `documents::acl` y `fleet::acl` | D-232 | `ModularityTests` (la raíz), con los `allowedDependencies` del `package-info.java` del módulo | ✅ |
| RPT-R02 | Las reglas con dueño se piden a `fleet::acl`: habilitación, entrega vigente, brecha y montos | D-232 | `ReportsIntegrationTests.theGapOfEachUnitComesFromItsBaseDate` y `theExportSaysWhatTheRecordOfTheUnitSays` | ✅ |
| RPT-R03 | Un reporte es de una empresa que el llamante lee con `reportes.ver`; si no, `404 COMPANY_NOT_FOUND`. Una asignación acotada a marcas lee solo esas | D-233 | `ReportsIntegrationTests.aReportReadsOnlyTheCompanyAndTheBrandsOfTheCaller`, `withoutThePermissionOfTheReportsNothingIsRead` y `theExportOfACompanyTheCallerDoesNotReadIsNotFound` | ✅ |
| RPT-R04 | Las unidades anuladas no cuentan | D-233 | `ReportsIntegrationTests.theRateCountsTheActiveOverTheActiveAndTheStopped`, con una unidad Activo anulada que no suma | ✅ |
| RPT-R05 | El exportable conserva los 52 encabezados y el orden de la plantilla, en cualquier idioma | D-234 | `ReportsIntegrationTests.theExportKeepsTheHeadersOfTheTemplateInItsOrder` | ✅ |
| RPT-R06 | Los montos siguen a `stock.ver_costos` por empresa y marca, y ese permiso entra en el alcance del documento | D-234 | `ReportsIntegrationTests.theExportSaysWhatTheRecordOfTheUnitSays`, con un lector sin costos; `StockRowsTests` | ✅ |
| RPT-R07 | Robado, Pérdida total, En traspaso e Inactivo no cuentan como paralizadas: son de familia terminal. Sí se cuentan en Unidades por estado. En traspaso se suma desde el 09/11/2026, con V123 (D-239) | D-235, D-239 | `FleetIndicatorsPolicyTests.theRateIsTheActiveOverTheActiveAndTheStoppedToOneDecimal`; `ReportsIntegrationTests.theRateCountsTheActiveOverTheActiveAndTheStopped`, que desde `9f9bd093` pone una unidad En traspaso y la cuenta en su estado, de familia `terminal`, fuera del porcentaje | ✅ |
| RPT-R08 | La diferencia por inicial exige `stock.ver_costos` en la empresa y totaliza por moneda | D-236 | `ReportsIntegrationTests.theDifferenceByDownPaymentAsksForTheCosts` | ✅ |
| RPT-R09 | Cada tabla que muestra un documento tiene sus casillas en `data_versions` | D-237 | Ninguna prueba nombra todavía las tres tablas que V121 versiona | ⚠️ |
| RPT-R10 | La diferencia por inicial es la inicial del cliente congelada al saldar, la de la imputación que salda (`customer_initial`), que una nota posterior no achica: «Diferencia por inicial» lista los saldados con inicial mayor que cero, y la columna 45 del exportable la lee. El exportable sube a la versión 2 de su diseño (`STOCK_LAYOUT`). Decidida el 10/11/2026, por construir | D-248, D-236 | Por escribir: `ReportsIntegrationTests`, con un saldado y una nota posterior que conserva su inicial | ❌ |

**El propietario en el exportable** (columna 50, D-223). Desde el 09/11/2026 vale también para una unidad En traspaso: dice a quién se traspasó cuando su traspaso está registrado, y la razón social de la empresa mientras falta (`ReportsIntegrationTests.theExportSaysWhatTheRecordOfTheUnitSays`).

## Fuera de alcance

- El costo por día paralizada: no hay ese dato.
- La validación de la inicial contra el contrato: espera al bloque 12.
- **La moneda de grupo** (09/11/2026). D-46 y §4.8 del diccionario la mandaban construir con los reportes, el paso 11. D-233 la deja fuera en la práctica: un reporte es de una empresa, se lee en su moneda funcional y dos monedas no se suman. Queda decidida fuera del alcance de los reportes, y pendiente para cuando haya dos monedas funcionales que reunir en un mismo reporte.

## Estado y pendientes

- **La sonda en vivo** (09/11/2026). No existe: `e2e-live/` no tiene `reports.live.spec.ts`. La pantalla tiene su prueba contra el simulador del backend (`e2e/reports.spec.ts`), y las consultas, su prueba de integración contra el esquema de Flyway. Escribirla y correrla contra el backend real queda pendiente, con las sondas que nunca corrieron ([orden de implementación](../orden-de-implementacion-millennium-mobility.md), pendientes transversales).
- RPT-R09, sin prueba propia (arriba).
- **RPT-R10** (10/11/2026, D-248), decidida y por construir con `feature/unit-level-balance` (backend, V127): `StockRows` toma la columna 45 de la inicial congelada, y «Diferencia por inicial» (`ReportQueryServiceImpl`) lista los saldados con inicial mayor que cero. Hasta entonces, la inicial es lo que le falta a la unidad saldada, y una nota posterior la achica ([especificación](../especificacion-reportes-2026-11-06.md), versión 3, §6).
