# Contexto `financing` — ERP Millennium Mobility

**Estado:** borrador para validar · **Fuente:** diccionario de datos (§2.4 a §2.6, §5.6, §11.8, §12, §13.1 y las decisiones que cita cada regla, en especial D-30, D-133, D-134, D-182, D-200, D-209 y D-279), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (P9 y el apartado 3), los [patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md), el [plan de migración](../plan-de-migracion-a-los-contextos-objetivo-2026-11-18.md) (fase 4) y el código. Si este documento y el diccionario no coinciden, manda el diccionario.

---

## 1. Qué es y qué no es

`financing` responde a qué fondo respalda cada unidad y desde cuándo (§12): los fondos de cada empresa —un fideicomiso, un fondo propio, un banco— y las cesiones de sus unidades a ellos, que se registran a mano o las abre y las cierra la lista de garantía mobiliaria de un fondo. La lista es una fotografía a una fecha, y la carga la traduce en cesiones que se abren y se cierran sin reescribir la historia (D-30). Su actor es Contabilidad. Nació en la fase 4 del plan de migración a los contextos objetivo (D-279): sus tablas y la carga de la garantía eran de `fleet`.

**Las dos tablas que posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `funds` | Los fondos de cada empresa, con su código, su nombre, su tipo y si se siguen cediendo unidades a él | §12.1 |
| `vehicle_fund_assignments` | Las cesiones de cada unidad a un fondo, con su período, quién la registró y la carga que la abrió, si fue una | §12.2 |

**Lo que parece suyo y no lo es**

| Parece de `financing` | Es de | Por qué |
|---|---|---|
| La unidad que se cede, su placa, si está anulada y quién la ve | `fleet` (§8.1) | `financing` la lee y la bloquea por la fachada de `fleet`, con el candado de toda escritura de la unidad; quién la ve —`stock.ver` en su empresa y su marca— lo decide `fleet` (D-102; FIN-R12) |
| Que una unidad con una cesión abierta no se anule | `fleet` (D-182; FLT-R54, en [fleet.md](fleet.md)) | La anulación es de la unidad, y pregunta por el puerto `UnitFundAssignments`, que `financing` responde (FIN-R21) |
| Si la ficha de la unidad ofrece cederla | `fleet`, que decide qué ofrece su ficha (`allowedActions.registerFundAssignment`) | `financing` responde por el mismo puerto si la sesión puede cederla ahora (FIN-R21); el recurso de la unidad no cambió (opción A de la decisión 5 de la fase 0 del plan) |
| La pestaña de cesiones de la ficha de la unidad | `fleet`, que la dibuja con lo que le da `FinancingFacade` | Opción A1 de la decisión 6 de la fase 0 del plan (sección 5) |
| El registro de la carga de la garantía, su archivo, su informe y la pantalla Logística › Cargas de archivos | `imports` (D-200, D-262) | La carga registra su lote por `imports::acl` (F3) |
| La empresa de un fondo, su zona y si opera | `organization` (§6) | La lee por su fachada |
| El nombre de quien registró una cesión | `iam` y `organization` | `iam` entrega el identificador público de la cuenta y el nombre de su colaborador (mapa, decisión 6) |
| La columna «GARANTÍA MOBILIARIA» del exportable de stock | `reporting` (D-232, D-234) | La lee de `vehicle_fund_assignments` por SQL, como modelo de lectura: `financing` no tiene fachada |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Fondo | Quien respalda el crédito con que una empresa compra sus unidades: un fideicomiso, un fondo propio o un banco (§12.1). Es de una empresa y de todas sus marcas | El fondo de un pago a proveedor, la parte de las aplicaciones sin imputar, que es de `procurement` |
| Ceder | Registrar que una unidad respalda a un fondo desde un día, a mano en su ficha o por la lista de garantía | Vender la unidad |
| Cesión | Que una unidad está cedida en garantía a un fondo durante un período `[inicio, fin)`; abierta, sin fin, mientras sigue cedida (§12.2) | — |
| Cerrar una cesión | Ponerle fin con la fecha que la termina, excluida, o con la fecha de la lista que ya no la nombra. La cesión no se borra | Desactivar el fondo |
| Lista de garantía mobiliaria | La lista de las placas que un fondo tiene en garantía a una fecha: una fotografía (guía, §8.2) | La historia de las cesiones, que la carga actualiza sin reemplazar |

---

## 2. Ciclo de vida de cada agregado

Nada de `financing` se borra: ni los fondos ni las cesiones (FIN-R06).

### Fondo

| Momento | Qué pasa |
|---|---|
| Alta | Lo crea quien escribe los fondos en toda la empresa y los lee (FIN-R01), con su código, su nombre y su tipo (FIN-R02, FIN-R03); no bajo una empresa inactiva (FIN-R05) |
| Qué cambia | El nombre, el tipo y el código, este mientras ninguna cesión usa el fondo (FIN-R04) |
| Retiro | Se desactiva y se reactiva; no se reactiva bajo una empresa inactiva (FIN-R05). Un fondo inactivo no recibe cesiones (FIN-R13) |
| Nunca | Se borra (FIN-R06) |

### Cesión de una unidad a un fondo

| Momento | Qué pasa |
|---|---|
| Alta | La registra quien cede en toda la empresa o en la marca de la unidad (FIN-R11), desde la ficha de la unidad, o la abre la lista de garantía, que la nombra (FLT-R111, FLT-R118). La unidad se ve y el fondo también (FIN-R12), son de la misma empresa y el fondo está activo (FIN-R13). Abre un período desde un día conocido que no se cruza con otra cesión de la unidad (FIN-R14). La ficha ofrece ceder cuando la sesión puede hacerlo ahora (FIN-R21) |
| Qué cambia | Solo su cierre (FIN-R15), que deja su fila en la bitácora (FIN-R16). Su autor y su creación quedan fijos (FIN-R19) |
| Quién la ve | Quien ve la unidad (FIN-R17) |
| Nunca | Se borra (FIN-R06); se abre o se cierra en una unidad anulada, ni deja anular una unidad mientras está abierta (FIN-R20) |

---

## 3. Flujos funcionales

En cada flujo, «el sistema revisa» es lo que el backend comprueba. Toda ruta exige un permiso del catálogo (§5.6, D-95), y el servicio decide dónde actúa con la asignación que lo trae (D-99). Escribir exige leer. El orden de las revisiones y sus códigos están en las convenciones (§7.2).

**Las pantallas** están en procesos y pantallas: Configuración › Fondos (`/financing/funds`, con `fondos.ver`), la pestaña de cesiones de la ficha de la unidad, de `fleet` (con `stock.ver`), y el diálogo de la garantía en Logística › Cargas de archivos, de `imports` (con `habilitacion.ver_cargas`).

**Los candados.** El orden de los candados de los fondos, de las cesiones y de la carga está en los [patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md), sección `financing`.

### F1 · Mantener los fondos

**Quién:** quien tiene `fondos.ver` los lee; `fondos.crear_fondos`, `fondos.editar_fondos` y `fondos.desactivar_fondos` los escriben (FIN-R01). En la semilla, el superadministrador y el administrador de empresa los escriben, y Contabilidad los lee. **Pantalla:** Configuración › Fondos (`/financing/funds`).

1. **Buscar y filtrar** (`GET /funds`): la empresa se elige en la lista, que trae los fondos de las empresas donde `fondos.ver` actúa; busca por código y nombre, filtra por estado, ordena y pagina en el servidor (FIN-R07), y exporta a Excel (FIN-R09).
2. **Registrar** (`POST /funds`): el código, el nombre y el tipo (FIN-R02); el código no se repite en la empresa (FIN-R03).
3. **Editar** (`PUT /funds/{id}`): el nombre, el tipo y, mientras ninguna cesión usa el fondo, el código (FIN-R04).
4. **Desactivar y activar** (`PATCH /funds/{id}/status`): desactivar pregunta antes; reactivar exige la empresa activa (FIN-R05).

Cada fila ofrece solo lo que el fondo dice que la sesión puede hacer (FIN-R08).

**Si algo falla:** `404 FUND_NOT_FOUND` o `COMPANY_NOT_FOUND`, `403`, `409 FUND_CONFLICT`, `422 FUND_CODE_LOCKED` o `COMPANY_INACTIVE`.

### F2 · Ceder una unidad y cerrar su cesión

**Quién:** `fondos.registrar_cesion` sobre toda la empresa o sobre la marca de la unidad (FIN-R11): en la semilla, el superadministrador, el administrador de empresa y Contabilidad. **Pantalla:** la pestaña de cesiones de la ficha de la unidad (`/fleet/vehicles/{id}?seccion=cesiones`), de `fleet`, que la dibuja con `FinancingFacade` (sección 5).

1. **Ceder** (`POST /vehicles/{id}/fund-assignments`). La ficha ofrece «Ceder» cuando la sesión puede ceder la unidad ahora (FIN-R21). El diálogo, de `financing`, ofrece los fondos activos de la empresa de la unidad y pide el fondo, el día de inicio en la zona de la empresa y, si se quiere, una nota.
2. El sistema bloquea la unidad por la fachada de `fleet` y revisa que la sesión la vea (FIN-R12), el permiso (FIN-R11), que no esté anulada (FIN-R20), el fondo, que se lee en modo compartido (FIN-R12, FIN-R13), y que el período no se cruce con otra cesión de la unidad (FIN-R14).
3. **Cerrar** (`POST /fund-assignments/{id}/closure`). La cesión dice su unidad, que se bloquea primero; después se revisan el permiso y la unidad anulada, y la cesión se vuelve a leer bajo el candado y se cierra en el día elegido (FIN-R15), con su fila en la bitácora (FIN-R16).
4. **Leer** (`GET /vehicles/{id}/fund-assignments`): la pestaña lista las cesiones de la unidad (FIN-R17) y ofrece cerrar la abierta a quien cede la unidad (FIN-R18).

**Si algo falla:** `404 VEHICLE_NOT_FOUND`, `FUND_NOT_FOUND` o `VEHICLEFUNDASSIGNMENT_NOT_FOUND`; `403`; `422 VEHICLE_ANNULLED`, `FUND_INACTIVE`, `VEHICLEFUNDASSIGNMENT_COMPANY_MISMATCH`, `VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START` o `VEHICLEFUNDASSIGNMENT_ALREADY_CLOSED`; `409 VEHICLEFUNDASSIGNMENT_CONFLICT`. **Resultado:** la ficha vuelve a leer la unidad, porque si se la puede ceder es un estado de la unidad.

### F3 · Cargar la lista de garantía mobiliaria

**Quién:** `fondos.cargar_archivo_garantia` y `habilitacion.ver_cargas` en toda la empresa y en todas sus marcas, y `fondos.ver` en la empresa (FLT-R116): en la semilla, superadministrador, administrador de empresa y Contabilidad. Desde Logística › Cargas de archivos (`POST /fund-assignment-imports`), que abre el diálogo de `financing` por `FinancingFacade`.

**Qué es el archivo.** La lista de las placas que un fondo tiene en garantía a una fecha: una fotografía (guía, §8.2). La fotografía no reemplaza la historia (D-30, §12.2): la carga la traduce en cesiones que se abren y se cierran, y solo actualiza unidades que existen (FLT-R110). Lo que comparte con las demás cargas —una a la vez de cada tipo por empresa, el tope de 5 000 filas, los conteos y el informe— está en el diccionario, §11.8, y en [fleet.md](fleet.md), F13 a F15.

1. Elige el fondo, entre los activos de la empresa, y la fecha de la lista, hoy en la zona de la empresa por omisión, nunca futura (FLT-R115).
2. El sistema revisa la empresa, los permisos, el fondo y la fecha, y toma el candado de la carga. La placa de cada fila se normaliza con la regla de la placa de una unidad, que da la fachada de `fleet`: una que no es placa falla su fila. Bloquea, por la fachada de `fleet`, las unidades de la empresa con las placas del archivo y las que tienen una cesión abierta al fondo: una cesión a mano de esas unidades espera (FLT-R117). Lee las cesiones de esas unidades, de cualquier fondo, y decide:

| Caso | Efecto |
|---|---|
| Placa de la lista sin cesión abierta | Abre una cesión al fondo desde la fecha de la lista, con su carga y la cuenta de la sesión como autor (FLT-R111, FLT-R118) |
| Placa de la lista con la cesión abierta a este fondo | Sin cambios (FLT-R111) |
| Placa de la lista con una cesión abierta a otro fondo, o cuya cesión nueva se cruzaría con una que ya existe | Falla la fila y no cambia nada de esa unidad: una unidad no está en dos fondos a la vez, y la lista de un fondo no decide sobre otro (FLT-R113) |
| Placa que no es de una unidad de la empresa, o unidad anulada | Falla la fila (FLT-R103, en [fleet.md](fleet.md)) |
| Placa repetida en el archivo | Fallan todas sus filas, y su cesión no se cierra (FLT-R105, en [fleet.md](fleet.md)) |
| Cesión abierta al fondo de una unidad con placa que no está en la lista, también una registrada a mano (D-209) | Se cierra con la fecha de la lista, excluida como todo fin de período (§2.6), con su fila en la bitácora y el motivo de la carga (FLT-R112) |
| Ese cierre no queda posterior al inicio de la cesión | No se cierra, y se informa (FLT-R113) |
| Cesión abierta al fondo de una unidad sin placa | No se cierra: la lista se cruza por placa y esa unidad no puede figurar. Se informa como «sin placa» (FLT-R112) |

3. **Los cierres se confirman** (FLT-R114). Si la carga cerraría al menos una cesión y el pedido no lo confirma, responde lo que haría —cuántas abre, cierra, deja igual, fallan y quedan sin placa— y no escribe nada; la pantalla lo pregunta y reenvía el mismo archivo confirmado. Así una lista equivocada no cierra cesiones sin que nadie lo vea.
4. Registra el lote, abre las cesiones nuevas con su carga y cierra las que corresponden. El informe se escribe siempre: lleva el fondo, la fecha de la lista y cada cesión que la lista cerró o no pudo cerrar (FLT-R113).

**Resultado:** las cesiones de la ficha de la unidad dicen qué carga abrió cada una, enlazada a su ficha para quien tiene `habilitacion.ver_cargas`.

Hasta perfilar una lista real, sus encabezados son provisionales (sección 6).

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). Las pruebas del backend están en `src/test/java/.../financing`, salvo las que dicen otro contexto y las de la raíz del paquete: `FundAssignmentAnnulmentRaceTests`, `GuaranteeLoadRaceTests` y `FundAssignmentAllowedActionsTests`. Las del frontend están junto a cada archivo de `src/app/financing`, y las de extremo a extremo, en `e2e/funds.spec.ts` y `e2e/exports.spec.ts`. Las reglas de la carga de la garantía conservan su identificador `FLT-`, de cuando eran de `fleet`; las de los fondos y las cesiones, que en `fleet` no tenían identificador, llevan `FIN-`. FLT-R103 y FLT-R105 valen también para el archivo del SAT, y están en [fleet.md](fleet.md).

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| FIN-R01 | Un fondo es de una empresa y de todas sus marcas: lo crean, lo editan y lo activan o desactivan solo las asignaciones de toda la empresa y sin marca, con `fondos.crear_fondos`, `fondos.editar_fondos` o `fondos.desactivar_fondos`, y quien lo escribe tiene que leerlo con `fondos.ver`: crear sin leer se rechaza sin guardar nada, y editar o cambiar el estado de un fondo que no se lee responde `404` | §12.1, §5.6, D-134 | Servicio, Frontend | `FundsIntegrationTests`, `e2e/funds.spec.ts` | ✅ |
| FIN-R02 | Un fondo lleva código, nombre y tipo. El código se guarda en mayúsculas; el tipo es uno de la lista cerrada —`fideicomiso`, `fondo_propio` o `banco`—, en minúsculas, y uno desconocido es un error de validación, que se responde antes de mirar el código | §12.1, D-133, DP-1 | Base, Dominio, Servicio, Frontend | `FundsIntegrationTests`, `fund-form-dialog.spec.ts` | ✅ |
| FIN-R03 | El código no se repite en la empresa; otra empresa puede usarlo. Si dos pedidos pasan la revisión a la vez, la base detiene el segundo, que responde el mismo `409 FUND_CONFLICT` sin guardar nada | §12.1 | Base, Servicio | `FundsIntegrationTests`, `FundRacedCodeTests`, `FundLockTests` | ✅ |
| FIN-R04 | El código se corrige mientras ninguna cesión usa el fondo; desde la primera queda fijo (`422 FUND_CODE_LOCKED`), y el fondo lo dice (`codeLocked`). Toda escritura de cesiones lee el fondo en modo compartido, así que la corrección, que lo bloquea para actualizar, ve la cesión que se confirma a la vez | §12.1, §2.4, D-134, DP-12 | Servicio, Frontend | `FundsIntegrationTests`, `FundLockTests`, `e2e/funds.spec.ts` | ✅ |
| FIN-R05 | Un fondo no se crea bajo una empresa inactiva ni se reactiva mientras su empresa lo está (`422 COMPANY_INACTIVE`): la empresa se lee en modo compartido hasta escribir, después del fondo cuando el fondo existe | §12.1, D-84, D-94 | Servicio | `FundsIntegrationTests`, `FundLockTests` | ✅ |
| FIN-R06 | Un fondo no se borra: se desactiva y se reactiva, y las unidades que estuvieron cedidas siguen explicables. Una cesión tampoco: se cierra. La API no tiene `DELETE`, y la cuenta de la aplicación no puede borrar un fondo ni una cesión | §12.1, §12.2, regla 4.2 | Base, Servicio | `FundsIntegrationTests`, `FinancingRowsAreNeverDeletedTests`, `NoDeleteEndpointsTests` | ✅ |
| FIN-R07 | Los fondos se leen con `fondos.ver` en las empresas donde actúa, y la pantalla se abre con él: un fondo de otra empresa responde `404` al leerlo, editarlo o cambiar su estado. La lista busca por código y nombre sin distinguir tildes, filtra por empresa y estado, y ordena y pagina en el servidor | §12.1, D-99, D-134, D-142 | Servicio, Frontend | `FundsIntegrationTests`, `funds.store.spec.ts`, `funds-view.spec.ts`, `financing.routes.spec.ts`, `e2e/funds.spec.ts` | ✅ |
| FIN-R08 | Cada fondo dice qué puede hacer la sesión sobre él, con la regla de los comandos: `edit`, y después `deactivate` o `activate`, cada uno si su permiso actúa en toda la empresa y sin marca | §12.1, D-134 | Servicio, Frontend | `FundsIntegrationTests`, `e2e/funds.spec.ts` | ✅ |
| FIN-R09 | La lista de fondos se exporta a Excel desde su propio endpoint, con la búsqueda, los filtros y el orden de la pantalla, el nombre de su empresa y su tipo traducido. Su alcance es exactamente `fondos.ver`, con la empresa de cada asignación: dos cuentas que leen los mismos fondos comparten el archivo, y una empresa donde no se lee queda fuera | §12.1, D-106, D-116, D-134 | Servicio, Frontend | `FundsExportIntegrationTests`, `e2e/exports.spec.ts` | ✅ |
| FIN-R10 | Un fondo guarda cuándo se creó y cuándo cambió, sin autor; guardarlo conserva sus identificadores, su empresa, su código y su creación | §12.1, D-133 | Base, Servicio | `FundAuditTests` | ✅ |
| FIN-R11 | Ceder una unidad y cerrar su cesión exigen `fondos.registrar_cesion` sobre toda la empresa o sobre la marca de la unidad, nunca sobre su sede: ceder es un acto financiero de la empresa, y un rol de sede ve la unidad y sus cesiones pero no la cede | §12.2, D-134 (excepción a D-101), DP-11 | Servicio, Frontend | `VehicleFundAssignmentsIntegrationTests`, `e2e/funds.spec.ts` | ✅ |
| FIN-R12 | La unidad se lee y se bloquea por la fachada de `fleet`: una que la sesión no ve con `stock.ver`, marca incluida, responde `404` antes de cualquier `403`, también al cerrar una de sus cesiones. El fondo se lee con `fondos.ver` o `fondos.registrar_cesion` en su empresa, y uno que no se ve responde `404` antes de comparar las empresas | §12.2, D-102, D-134 | Servicio | `VehicleFundAssignmentsIntegrationTests`, `FleetContextFacadeFinancedUnitsTests` (`fleet`) | ✅ |
| FIN-R13 | La unidad y el fondo son de la misma empresa (`422 VEHICLEFUNDASSIGNMENT_COMPANY_MISMATCH`), y la base lo exige también; el fondo está activo (`422 FUND_INACTIVE`) y se lee en modo compartido hasta escribir, así que una desactivación simultánea espera a la cesión o la hace esperar | §12.2, D-133 | Base, Servicio | `VehicleFundAssignmentsIntegrationTests`, `FinancingRowLevelSecurityTests`, `FundLockTests` | ✅ |
| FIN-R14 | Una cesión abre un período `[inicio, fin)` desde un día conocido, y una unidad no tiene dos cesiones que se crucen, tampoco en una carrera, que la base detiene (`409 VEHICLEFUNDASSIGNMENT_CONFLICT`); una cesión puede empezar el día en que terminó la anterior | §12.2, §2.6 | Base, Servicio, Frontend | `VehicleFundAssignmentsIntegrationTests`, `FinancingRowLevelSecurityTests`, `assign-fund-dialog.spec.ts` | ✅ |
| FIN-R15 | Una cesión solo cambia por su cierre: con la fecha de fin que elige quien la cierra, excluida y posterior al inicio (`422 VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START`), o con la fecha de la lista que ya no la nombra (FLT-R112). Una cerrada no se vuelve a cerrar (`422 VEHICLEFUNDASSIGNMENT_ALREADY_CLOSED`), tampoco si otro cierre se confirmó mientras este esperaba el candado de la unidad | §12.2, §2.6 | Dominio, Servicio, Frontend | `VehicleFundAssignmentsIntegrationTests`, `FundLockTests`, `close-fund-assignment-dialog.spec.ts` | ✅ |
| FIN-R16 | El cierre de una cesión, a mano o por la lista, deja su fila en la bitácora: el período de antes y el de después, con la marca de la unidad (D-102) y, el de la lista, su motivo. Ningún campo de la cesión es confidencial | §13.1, D-129, D-209 | Servicio | `FundAssignmentImportsIntegrationTests`, el de la lista; el cierre a mano, sin prueba propia | ⚠️ |
| FIN-R17 | Las cesiones de una unidad se leen con `stock.ver` donde se ve la unidad, marca incluida; `fondos.ver` solo no basta. La lista pone primero la abierta y después las demás, de la de inicio más nuevo a la más vieja, y dice qué carga abrió cada una (FLT-R118) | §12.2, D-102 | Servicio, Frontend | `VehicleFundAssignmentsIntegrationTests`, `vehicle-fund-assignments.store.spec.ts`, `vehicle-fund-assignments.spec.ts` (`fleet`) | ✅ |
| FIN-R18 | Una cesión abierta ofrece cerrarla (`allowedActions.close`) solo a quien cede la unidad | §12.2, DP-11 | Servicio, Frontend | `VehicleFundAssignmentsIntegrationTests`, `financing.facade.spec.ts`, `vehicle-fund-assignments.spec.ts` (`fleet`) | ✅ |
| FIN-R19 | Una cesión guarda quién la registró —la cuenta de la sesión, también en la que abre la lista— y cuándo, y su cierre conserva eso, sus identificadores y su empresa; sin sesión, queda el autor que pone quien la escribe | D-42, D-133 | Base, Servicio | `VehicleFundAssignmentAuditTests` | ✅ |
| FIN-R20 | Una unidad anulada no se cede ni se le cierra una cesión (`422 VEHICLE_ANNULLED`), y una con una cesión abierta no se anula (FLT-R54, en [fleet.md](fleet.md)). Ceder, cerrar y anular toman primero el candado de la unidad, así que no se cruzan en ningún orden: la segunda espera y responde según lo que dejó la primera | D-104, D-182 | Servicio | `FundAssignmentAnnulmentRaceTests`, `VehicleAnnulmentsIntegrationTests` (`fleet`) | ✅ |
| FIN-R21 | `financing` responde el puerto `UnitFundAssignments` de `fleet`, y un solo bean lo implementa: si una unidad tiene una cesión abierta, para su anulación (FLT-R54), y si la sesión puede cederla ahora —`fondos.registrar_cesion` en toda la empresa o en la marca de la unidad, `fondos.ver` en su empresa, porque el fondo se elige de su lista, y ninguna cesión abierta ni que termine después de hoy en la zona de la empresa (D-40)—, para lo que ofrece la ficha (`allowedActions.registerFundAssignment`) | D-182, D-40, DP-11; opción A de la decisión 5 de la fase 0 del plan | Servicio, Frontend | `UnitFundAssignmentsWiringTests`, `UnitFundAssignmentsTodayTests`, `FundAssignmentAllowedActionsTests`, `VehicleAllowedActionsTests` (`fleet`), `vehicle-profile-view.spec.ts` (`fleet`) | ✅ |
| FIN-R22 | Las tablas de los fondos son de empresa, con Row Level Security: el alcance de una empresa no ve los fondos ni las cesiones de otra, ni agrega o cambia una cesión de sus unidades | §2.5, D-133 | Base | `FinancingRowLevelSecurityTests`, `RowLevelSecurityMigrationTests` | ✅ |
| FLT-R110 | La lista de garantía es la de un fondo a una fecha; solo actualiza unidades existentes y nunca crea una | §12.2, §11.8; guía, §8.2 | Servicio | `GuaranteeListPolicyTests`, `FundAssignmentImportsIntegrationTests` | ✅ |
| FLT-R111 | Una placa sin cesión abierta abre una al fondo desde la fecha de la lista, con su carga; con la cesión abierta a ese fondo, queda sin cambios | §12.2, D-30 | Dominio | `GuaranteeListPolicyTests` | ✅ |
| FLT-R112 | Una cesión abierta al fondo cuya unidad tiene placa y no figura en la lista se cierra en la fecha de la lista, con su fila en la bitácora; la historia se conserva. Vale también para las cesiones registradas a mano. La de una unidad sin placa no se cierra y se informa | §12.2, D-30, D-209, §13.1 | Dominio, Servicio | `GuaranteeListPolicyTests`, `FundAssignmentImportsIntegrationTests` | ✅ |
| FLT-R113 | Una placa cedida a otro fondo, o cuya cesión nueva se cruzaría con otra, falla y no cambia nada de esa unidad; un cierre que no queda posterior al inicio no se hace y se informa. El fondo y la fecha de la lista quedan en el informe | §12.2, D-133, D-209 | Base, Dominio | `GuaranteeListPolicyTests`, `FundAssignmentImportsIntegrationTests` | ✅ |
| FLT-R114 | Si la carga cerraría cesiones, exige la confirmación; sin ella responde los efectos y no escribe nada | D-30 | Servicio, Frontend | `FundAssignmentImportsIntegrationTests` | ✅ |
| FLT-R115 | El fondo es de la empresa, activo y se lee en modo compartido, como toda escritura de cesiones; la fecha de la lista no es futura en la zona de la empresa | §12.1, §2.6 | Servicio | `FundAssignmentImportsIntegrationTests`, la fecha; el fondo inactivo y su lectura compartida, sin prueba propia | ⚠️ |
| FLT-R116 | Cargar la garantía exige `fondos.cargar_archivo_garantia`, `fondos.ver` y `habilitacion.ver_cargas` en toda la empresa y en todas sus marcas | §5.6, D-134 | Servicio | Sin prueba propia (`FleetImportsIntegrationTests`, de `fleet`, prueba la del SAT) | ⚠️ |
| FLT-R117 | Las unidades se bloquean antes de decidir, por la fachada de `fleet`, en una sentencia y en orden de `id`: las de las placas de la lista y las que tienen una cesión abierta al fondo. Una cesión a mano en paralelo espera y ve lo que la carga escribió | Patrones de persistencia | Servicio | `GuaranteeLoadRaceTests`, `VehicleFundAssignmentsOpenToFundTests`, `FleetContextFacadeFinancedUnitsTests` (`fleet`) | ✅ |
| FLT-R118 | Una cesión nombra la carga que la abrió, de su misma empresa; la de una cesión a mano queda vacía | §12.2, D-133 | Base, Servicio | `FundAssignmentImportsIntegrationTests`, la carga que la abrió; que sea de su misma empresa y que la de una cesión a mano quede vacía, sin prueba propia | ⚠️ |

---

## 5. Relación con otros contextos

**Las dependencias.** `financing` usa a `fleet`, `organization` e `iam` por sus fachadas; a `audit`, `imports` y `documents`, por sus contratos de la plataforma; y a `shared`. `ModularityTests` lo verifica. Ninguna forma un ciclo: ninguno de ellos llama a `financing`, y lo que `fleet` sabe de las cesiones lo pregunta por un puerto suyo que `financing` implementa (D-279).

**Lo que ofrece.** Ninguna fachada: nadie la consume. `reporting` lee `vehicle_fund_assignments` por SQL, como modelo de lectura (*Conformist*), para la columna «GARANTÍA MOBILIARIA» del exportable de stock.

**Los puertos que implementa.** `UnitFundAssignments`, de `fleet` (FIN-R21): si una unidad tiene una cesión abierta y si la sesión puede cederla ahora. Lo responde con las cesiones, el alcance de la sesión y el «hoy» de la empresa, sin pasar por la fachada de `fleet`, así que no forma un ciclo de beans con ella.

**Lo que usa de otros contextos**, por sus fachadas:

| Contexto | Para qué |
|---|---|
| `fleet` | La unidad que se cede, cuya cesión se cierra o cuyas cesiones se listan, con su empresa, su marca, su placa, su VIN y si está anulada, leída o bloqueada con el candado de toda escritura de la unidad y solo si la sesión la ve (FIN-R12); las unidades de la empresa con las placas de una lista o con una cesión abierta al fondo, bloqueadas en una sentencia y por `id` (FLT-R117); y la placa de una fila, normalizada con la regla de la placa de una unidad (F3) |
| `organization` | La empresa de un fondo, si existe y quien llama la alcanza, leída en modo compartido al crear o reactivar un fondo (FIN-R05); los nombres de las empresas de una lista; la zona de una empresa, para el «hoy» de la lista y de lo que la ficha ofrece (FLT-R115, FIN-R21); y la versión de sus nombres, para la exportación (FIN-R09) |
| `iam` | El nombre de quien registró una cesión (mapa, decisión 6, opción B) |
| `audit` | La bitácora del cierre de una cesión (FIN-R16) |
| `imports` | El registro de la carga de la garantía, con su lector y su escritor de libros (F3; D-200, D-262) |
| `documents` | La exportación de los fondos (FIN-R09) |
| `shared` | El alcance de quien llama (D-99, D-102), las clases base de la persistencia y el reintento de un interbloqueo (D-128) |

**Frontend.** `FinancingFacade` ofrece a `fleet` la pestaña de cesiones de la ficha de la unidad —la página, su aviso, su error y las negativas porque la unidad o sus cesiones cambiaron mientras tanto—, con vistas propias y su propia carga, apartadas de la pantalla de Fondos (opción A1); y los diálogos de ceder y de cerrar, que responden si se guardó para que la ficha vuelva a leer la unidad. A Logística › Cargas de archivos le ofrece el diálogo de la garantía. `financing` lee las empresas de `OrganizationFacade` y los permisos de la sesión de `IamFacade`.

---

## 6. Estado y pendientes

**Estado:** construido, con FIN-R16, FLT-R115, FLT-R116 y FLT-R118 en ⚠️. La carga de la garantía está construida y no se usa todavía (§11.8). Falta la validación de este documento por el responsable. No se conocen diferencias con las convenciones de la API.

| Qué | Quién y cuándo |
|---|---|
| Perfilar una lista de garantía real: sus encabezados son provisionales, y el perfil dice si trae el fondo o una fecha por placa | El negocio entrega el archivo |
| Ensayar la carga con un archivo real sobre una base de verificación, como las demás cargas (§11.8) | El equipo, antes de usarla |
| Las pruebas que faltan: el cierre a mano en la bitácora (FIN-R16); el fondo inactivo y su lectura compartida en la carga (FLT-R115); los permisos de la carga (FLT-R116); la carga de la misma empresa y la cesión a mano sin carga (FLT-R118); y la unidad anulada en la lista (FLT-R103, en [fleet.md](fleet.md)) | El equipo, antes de usar la carga |
| Corregir el período de una cesión con un motivo (`PUT /fund-assignments/{id}/period`) | Propuesta de las convenciones (§14.21), sin decidir |
| Que un fondo o su auditor entre al sistema, y la referencia al contrato de cesión con el fondo | Candidatos del diccionario (§12.3) |
| Recibir de la cobranza los flujos que se ceden a un fondo | Con el bloque 12 (mapa objetivo, §3.8) |
| Validar este documento | El responsable |

---

## 7. Preguntas abiertas

Ninguna.
