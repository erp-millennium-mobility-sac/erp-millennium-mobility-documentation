# Contexto `reporting` — ERP Millennium Mobility

**Actualizado:** 06/11/2026 (backend `e5bcc028`, `c69f097d`, `ff52ea1e`, `486428cd`, `85f23e2e`, `bd37d618`, `4f62a26e` y `d68e3b24`, frontend `90a0a7ee`, `ab142f03`, `2896de23` y `85520b76`, migraciones V120 y V121).

La capa de lectura: los reportes que cruzan contextos. Lee por SQL y nunca escribe. Lo que hace y por qué está en la [especificación](../especificacion-reportes-2026-11-06.md); las decisiones son D-232 a D-237.

## Reglas

| ID | Regla | Fuente |
|---|---|---|
| RPT-R01 | `reporting` no escribe, y depende solo de `shared`, `documents::acl` y `fleet::acl` | D-232 |
| RPT-R02 | Las reglas con dueño se piden a `fleet::acl`: habilitación, entrega vigente, brecha y montos | D-232 |
| RPT-R03 | Un reporte es de una empresa que el llamante lee con `reportes.ver`; si no, `404 COMPANY_NOT_FOUND`. Una asignación acotada a marcas lee solo esas | D-233 |
| RPT-R04 | Las unidades anuladas no cuentan | D-233 |
| RPT-R05 | El exportable conserva los 52 encabezados y el orden de la plantilla, en cualquier idioma | D-234 |
| RPT-R06 | Los montos siguen a `stock.ver_costos` por empresa y marca, y ese permiso entra en el alcance del documento | D-234 |
| RPT-R07 | Robado, Pérdida total e Inactivo no cuentan como paralizadas | D-235 |
| RPT-R08 | La diferencia por inicial exige `stock.ver_costos` en la empresa y totaliza por moneda | D-236 |
| RPT-R09 | Cada tabla que muestra un documento tiene sus casillas en `data_versions` | D-237 |

## Fuera de alcance

- El costo por día paralizada: no hay ese dato.
- La validación de la inicial contra el contrato: espera al bloque 12.
