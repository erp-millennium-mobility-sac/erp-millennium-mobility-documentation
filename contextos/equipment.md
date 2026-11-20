# Contexto `equipment` — ERP Millennium Mobility

**Estado:** borrador para validar · **Fuente:** diccionario de datos (§2.2, §2.5, §5.6, §7.5, §11.4 y las decisiones que cita cada regla, en especial D-254, D-260, D-263, D-264 y D-278), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (apartado 3.2), el [plan de migración](../plan-de-migracion-a-los-contextos-objetivo-2026-11-18.md) (fase 2) y el código. Si este documento y el diccionario no coinciden, manda el diccionario.

---

## 1. Qué es y qué no es

`equipment` responde qué equipos se instalan en una unidad —el GPS y la pantalla—, de qué marca y de qué modelo es cada uno, a qué tipos de vehículo aplica cada tipo, y qué equipos físicos tiene cada empresa en su almacén, con sus identificadores y su estado (§7.5, §11.4). Nació en la fase 2 del plan de migración a los contextos objetivo (D-278): su catálogo era de `vehiclecatalog` y su inventario, de `fleet`. Qué equipo exige la habilitación y qué equipo está instalado en qué unidad no lo dice `equipment`: lo dice `fleet` (D-260; FLT-R148). Usa a `organization`, `vehiclecatalog` y `documents` por sus fachadas; `fleet` lo usa por la suya, y `procurement` y `fleet` le responden por los puertos que declara (sección 5).

**Las cinco tablas que posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `equipment_types` | Lo que se instala en una unidad —`GPS`, `PANTALLA`—, con si exige número de serie, si su equipo lleva IMEI y SIM y si ya tuvo una instalación. Si se exige para pasar a Disponible no lo dice aquí: lo dice la configuración de `fleet` (D-260) | §7.5, D-185, D-254 |
| `equipment_type_vehicle_types` | A qué tipos de vehículo aplica cada equipamiento | §7.5, D-73 |
| `equipment_brands` | Las marcas de equipamiento —TELTONIKA, QUECLINK—, que agrupan sus modelos | §7.5 |
| `equipment_models` | El modelo comercial de cada tipo de equipamiento, de una marca —«FMB920» del GPS—, con nombre único en su tipo y su marca | §7.5, D-254 |
| `fleet_equipments` | El inventario: cada equipo físico en el almacén de una empresa, con su ubicación, su proveedor, su tipo, su modelo, sus identificadores y su estado. Conserva el nombre que tenía en `fleet`: ninguna fase renombra una tabla (plan, §1) | §11.4, D-254 |

**Lo que parece suyo y no lo es**

| Parece de `equipment` | Es de | Por qué |
|---|---|---|
| La instalación de un equipo en una unidad, con su período, quién instaló y su costo | `fleet` (§11.4) | La puerta de Disponible la lee en la misma transacción que el estado de la unidad (plan, fase 2). Hoy la instalación guarda un serial escrito a mano y no toca el inventario; con el módulo de monitoreo elegirá un equipo del inventario (FLT-R152) |
| Qué equipo exige una unidad y cuál es de seguridad | `fleet` (§11.4, D-260) | `equipment` responde, una sola vez y en lote, qué equipo aplica a cada modelo y cuál trae de fábrica (VCT-R28); `fleet` lo cruza con su configuración de lo que se exige y de lo que es de seguridad |
| Los tipos de vehículo, los modelos y si un modelo trae la pantalla de fábrica | `vehiclecatalog` (§7.1, §7.3) | `equipment` los lee por su fachada, en lote: los tipos de vehículo de un equipamiento y, de cada modelo, su tipo y su pantalla de fábrica |
| El proveedor de un equipo | `procurement` (§9.1) | Lo responde por el puerto `EquipmentSuppliers` (EQP-R18) |
| La unidad en que está un equipo y su placa | `fleet` (§8.1) | La responde por el puerto `EquipmentUnits` (EQP-R18) |
| La empresa y la ubicación de un equipo | `organization` (§6) | Las lee por su fachada |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Tipo de equipamiento | Lo que se instala en una unidad: el GPS, la pantalla (§7.5) | Un equipo físico del inventario |
| Aplicabilidad | Que un equipamiento tenga sentido en un tipo de vehículo | Una instalación, que es de `fleet`; y que se exija, que lo dice la configuración de `fleet` |
| Marca de equipamiento | Quien fabrica un equipo: TELTONIKA. Agrupa sus modelos | La marca de un vehículo y la marca comercial (§6.3) |
| Modelo de equipamiento | El modelo comercial de un tipo de equipamiento, de una marca: «FMB920» del GPS. El nombre solo identifica junto a su tipo y su marca (D-254) | El modelo de un vehículo |
| Telemático | Un tipo de equipamiento cuyo equipo lleva IMEI y tarjeta SIM, como el GPS (D-254) | Que exija número de serie |
| Código que lee el programa | `GPS` y `PANTALLA`: la pantalla de fábrica, la telemetría del GPS, la carga inicial y los reportes los reconocen por su código, así que quedan fijos desde la semilla (D-74) | Un código propio, que se corrige mientras ninguna instalación lo use; y lo que exige la habilitación, que no sale del código sino de la configuración de `fleet` (D-260) |
| Equipamiento con instalaciones | Un tipo de equipamiento que `fleet` ya instaló en alguna unidad. Desde entonces su código queda fijo (D-185) | Un equipamiento activo, que solo dice si se sigue instalando |
| Equipo del inventario | Un equipo físico de una empresa —un GPS, una pantalla— con su serial, su IMEI, su ICCID y su estado (§11.4) | La instalación en una unidad, que es de `fleet` y hoy no apunta a él |
| De baja | El estado final de un equipo que salió del inventario: perdido, robado o inservible. No se edita ni vuelve | Borrarlo, que no existe |
| Desactivar | Dejar de ofrecer un tipo, una marca o un modelo para lo nuevo; lo que ya lo usa lo conserva | Borrar, que no existe |

---

## 2. Ciclo de vida de cada agregado

Nada de `equipment` se borra: ni los tipos, ni sus aplicabilidades, ni las marcas, ni los modelos, ni los equipos del inventario (EQP-R03). El catálogo registra cuándo se creó y cuándo cambió, sin autor; el inventario, además, quién lo registró y quién lo cambió (EQP-R04).

### Tipo de equipamiento

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en el diálogo de los tipos de Catálogos › Equipamiento, con su código, su nombre, si requiere número de serie y los tipos de vehículo a los que aplica, al menos uno (VCT-R18). La semilla trae `GPS` y `PANTALLA` con sus banderas y sus tipos (EQP-R06), y el GPS es telemático. Un tipo con código `GPS` es siempre telemático; los demás nacen sin serlo, salvo que la API reciba `requiresTelematics` (VCT-R35). Un tipo nuevo nace sin requisito: si se exige para Disponible y si es de seguridad lo marca `fleet`, en Catálogos › Requisitos de equipos (D-260) |
| Qué cambia | El nombre, si requiere número de serie, los tipos a los que aplica y el código mientras no quede fijo. Si es telemático cambia solo por la API: una edición que no lo manda conserva lo guardado, y la pantalla nunca lo manda (D-254) |
| Qué queda fijo | El código de `GPS` y el de `PANTALLA`, desde la semilla (VCT-R19); el de otro equipamiento, desde su primera instalación (VCT-R30) |
| Retiro | Se desactiva y se reactiva. Quitarle un tipo de vehículo desactiva esa aplicabilidad, y volver a marcarlo reactiva la misma (VCT-R18). Para que deje de instalarse en todo se desactiva el equipamiento, no se le quitan todos los tipos. Un tipo inactivo no recibe modelos nuevos ni reactiva los suyos (VCT-R34), y un equipo nuevo del inventario no lo elige (EQP-R13). Un tipo de vehículo que `vehiclecatalog` desactiva sigue en los equipamientos que ya aplican a él: sus unidades pueden seguir necesitándolos (§7.5) |
| Nunca | Se borra; queda sin ningún tipo de vehículo; repite código; vuelve a no tener instalaciones |

### Marca de equipamiento

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en «Gestionar marcas» de Catálogos › Equipamiento, con su nombre: `Teltonika` y `TELTONIKA` son la misma marca (VCT-R36) |
| Qué cambia | El nombre |
| Retiro | Se desactiva y se reactiva en el mismo diálogo. Una marca inactiva no recibe modelos nuevos ni reactiva uno, y un equipo nuevo del inventario no elige un modelo suyo (VCT-R37). Sus modelos y los equipos que los tienen la siguen referenciando |
| Nunca | Se borra; repite nombre (VCT-R36) |

### Modelo de equipamiento

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en Catálogos › Equipamiento, eligiendo su tipo y su marca de equipamiento entre los activos, con su nombre comercial (VCT-R33, VCT-R34, VCT-R37) |
| Qué cambia | El nombre |
| Qué queda fijo | Su tipo y su marca, desde el alta (VCT-R33) |
| Retiro | Se desactiva y se reactiva; se reactiva solo mientras su tipo y su marca están activos (VCT-R34, VCT-R37). Los equipos del inventario que ya lo tienen lo conservan; ninguno nuevo lo elige (EQP-R13) |
| Nunca | Se borra; repite nombre dentro de su tipo y su marca (VCT-R33) |

### Equipo del inventario

Cada equipo físico —un GPS, una pantalla— en el almacén de una empresa, con su ubicación, su proveedor, su tipo, su modelo, sus identificadores (serial, IMEI, ICCID, IMSI) y su estado (§11.4, D-254).

| Momento | Qué pasa |
|---|---|
| Alta | Con `stock.crear_equipamiento`, sobre toda la empresa (EQP-R12; en la semilla, superadministrador, administrador de empresa y Operaciones). Su ubicación fija su empresa; el proveedor es de esa empresa y está activo; el tipo y el modelo están activos, el modelo es del tipo y su marca está activa (EQP-R13, VCT-R37). El serial, el IMEI y el ICCID son únicos en todo el sistema, y no por empresa, mientras el equipo no está de baja, y un tipo telemático exige IMEI e ICCID (EQP-R14). Nace Disponible. Registrarlo no fija el código de su tipo, que se fija con su primera instalación (EQP-R17) |
| Estados | Disponible → Mantenimiento o De baja; Mantenimiento → Disponible o De baja; Instalado → Mantenimiento; De baja es final. Cambian con `stock.cambiar_estado_equipamiento` (EQP-R15). Ningún camino lo pasa todavía a Instalado: con el módulo de monitoreo, solo la instalación de `fleet` lo pasa a Instalado y lo saca de ahí, y el inventario ya no lo manda de Instalado a Mantenimiento (FLT-R152, FLT-R153) |
| Qué cambia | Con `stock.editar_equipamiento`: los datos técnicos, salvo de baja; el serial, el IMEI, la ubicación y el proveedor, solo mientras está Disponible (EQP-R16) |
| Quién lo ve | Quien tiene `stock.ver` en su empresa: un equipo de otra empresa responde `404` aunque la cuenta tenga ahí otro rol (EQP-R11) |
| Retiro | Sale del inventario de baja; no se borra (EQP-R03) |

---

## 3. Flujos funcionales

Las lecturas del catálogo piden `catalogos.ver` y sus escrituras, crear, editar o desactivar equipamiento (EQP-R02); con la semilla escribe solo el superadministrador y los demás roles leen. El inventario se lee con `stock.ver` y se escribe con sus tres permisos `stock.*_equipamiento`, sobre toda la empresa del equipo (EQP-R11, EQP-R12). Las pantallas son dos: Catálogos › Equipamiento (`/equipment/types`) y Logística › Inventario de equipamiento (`/equipment/inventory`); cada una se abre solo con su permiso de lectura (D-141), y cada acción se ofrece solo con el suyo. Las rutas que tenían antes, `/vehicle-catalog/equipment-types` y `/fleet/equipments`, llevan a ellas con lo que pedían, como `?empresa=` o `?tipo=`. Las listas se buscan sin distinguir mayúsculas ni tildes, se filtran, se ordenan y se paginan en el servidor (EQP-R08). Desactivar un tipo, una marca o un modelo pregunta antes (EQP-R07).

### F1 · Mantener el equipamiento: sus modelos, sus marcas y sus tipos

**Rol:** superadministrador. **Pantalla:** Catálogos › Equipamiento: la lista de los modelos de equipamiento, con los diálogos de sus marcas y de sus tipos (D-254).

**Los modelos de equipamiento** (`/equipment-models`).

1. La lista muestra los modelos con su marca y su tipo de equipamiento, y se filtra por tipo, por marca y por estado.
2. Registra un modelo bajo un tipo y una marca activos, con su nombre comercial (VCT-R33, VCT-R34, VCT-R37).
3. Edita solo el nombre; el tipo y la marca no cambian (VCT-R33).
4. Lo desactiva o lo activa; reactivarlo exige que su tipo y su marca estén activos (VCT-R34, VCT-R37).

**Si algo falla:** el diálogo lo dice (`EQUIPMENTMODEL_CONFLICT`, `EQUIPMENTTYPE_INACTIVE`, `EQUIPMENTBRAND_INACTIVE`, `EQUIPMENTMODEL_NOT_FOUND`).

**Las marcas de equipamiento** (`/equipment-brands`). «Gestionar marcas» abre un diálogo que lista las marcas, las busca y las pagina en el servidor, y las registra, renombra, activa y desactiva en su lugar, un formulario a la vez, sin abrir otro diálogo encima (VCT-R36). Desactivar una pregunta antes (EQP-R07). Al cerrar el diálogo después de un cambio, la lista vuelve a leer sus modelos.

**Si algo falla:** el diálogo lo dice y conserva lo escrito (`EQUIPMENTBRAND_CONFLICT`, también en una carrera; EQP-R05).

**Los tipos de equipamiento** (`/equipment-types`). El diálogo los lista, los busca y los pagina en el servidor; la API busca por código o nombre y filtra también por el tipo de vehículo al que aplica hoy.

1. Registra un tipo con su código, su nombre, si requiere serie y los tipos de vehículo activos a los que aplica, al menos uno (VCT-R18). La pantalla no muestra ni envía si es telemático: lo dice el backend (VCT-R35). Si el tipo se exige para Disponible o es de seguridad no se marca aquí: lo marca `fleet` en Catálogos › Requisitos de equipos (D-260).
2. Lo edita: el código de `GPS`, de `PANTALLA` y de un equipamiento que ya tuvo una instalación no cambia (VCT-R19, VCT-R30); el formulario ofrece también los tipos de vehículo inactivos a los que ya aplica, para poder desmarcarlos (VCT-R18). Los tipos de vehículo los lee de `vehiclecatalog`, por su fachada.
3. Lo desactiva o lo activa.

El sistema bloquea el equipamiento mientras decide un cambio, para que dos ediciones simultáneas no se pisen (VCT-R20).

**Si algo falla:** el formulario lo dice y conserva lo escrito (`EQUIPMENTTYPE_CONFLICT`, `VEHICLETYPE_NOT_FOUND`, `EQUIPMENTTYPE_CODE_LOCKED`); si el código quedó fijo mientras tanto, vuelve al guardado.

### F2 · Exportar los tipos de equipamiento

**Rol:** quien lee los catálogos, con `catalogos.ver`. **Pantalla:** Catálogos › Equipamiento (D-116).

1. Exporta a Excel todos los tipos de equipamiento, sin los filtros de la lista de modelos, por la plataforma de documentos; la API exporta con la misma búsqueda y los mismos filtros de su lista (EQP-R09; convenciones §8).
2. Cada tipo lleva los tipos de vehículo a los que aplica, por su nombre y su código. Los modelos, las marcas y el inventario no tienen exportación.

Los archivos llevan códigos y nombres, nunca un identificador (D-51); sus columnas están en convenciones §8.1.

**Si algo falla:** la pantalla dice que no pudo descargar el archivo.

### F3 · Registrar y mantener un equipo del inventario

**Quién:** `stock.crear_equipamiento`, `stock.editar_equipamiento` y `stock.cambiar_estado_equipamiento` (en la semilla, superadministrador, administrador de empresa y Operaciones), sobre toda la empresa del equipo (EQP-R12). **Pantalla:** Logística › Inventario de equipamiento (`/equipment/inventory`; la API, `/equipments`), que se abre con `stock.ver`.

1. Elige la empresa; la lista filtra por ubicación, tipo, modelo y estado, y busca por serial, IMEI, ICCID o código de producto. Lee solo las empresas donde actúa `stock.ver` (EQP-R11). Cada equipo muestra su tipo, su marca, su modelo, su proveedor y, si está instalado, la unidad con su placa (EQP-R18).
2. **Registra un equipo** (`POST /equipments`) con su ubicación, su proveedor, su tipo, su modelo, su serial y, para un tipo telemático, su IMEI y su ICCID. El sistema revisa, en este orden: la ubicación, que se vea, y que el permiso actúe en toda su empresa; la empresa, si la nombra, que sea la de la ubicación; el proveedor, que exista, sea de esa empresa y esté activo (EQP-R13); el tipo y el modelo, que existan y estén activos, con la marca del modelo activa (VCT-R37), y que el modelo sea del tipo; los identificadores que exige el tipo y su formato (EQP-R14); y que el serial, el IMEI y el ICCID no los tenga otro equipo, también en una carrera (EQP-R05). Nace Disponible.
3. **Edita** (`PATCH /equipments/{id}`): solo los campos que trae. El serial, el IMEI, la ubicación y el proveedor cambian solo mientras está Disponible, y un equipo de baja no cambia (EQP-R16).
4. **Cambia el estado** (`PATCH /equipments/{id}/status`), con su motivo, por los caminos de EQP-R15. Instalado no se pide por aquí.

El sistema bloquea el equipo mientras decide un cambio ([patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md)).

**Si algo falla:** `404 LOCATION_NOT_FOUND`, `COMPANY_NOT_FOUND`, `SUPPLIER_NOT_FOUND`, `EQUIPMENTTYPE_NOT_FOUND`, `EQUIPMENTMODEL_NOT_FOUND` o `EQUIPMENT_NOT_FOUND`; `403` si el permiso no actúa en la empresa; `422 SUPPLIER_INACTIVE`, `EQUIPMENT_SUPPLIER_OTHER_COMPANY`, `EQUIPMENT_TYPE_INACTIVE`, `EQUIPMENT_MODEL_INACTIVE`, `EQUIPMENTBRAND_INACTIVE`, `EQUIPMENT_MODEL_TYPE_MISMATCH`, `EQUIPMENT_TELEMATICS_REQUIRED`, `EQUIPMENT_SERIAL_REQUIRED`, `EQUIPMENT_IMEI_INVALID`, `EQUIPMENT_ICCID_INVALID`, `EQUIPMENT_IDENTIFIERS_NOT_MODIFIABLE`, `EQUIPMENT_DISCARDED_IMMUTABLE` o `EQUIPMENT_STATUS_TRANSITION_INVALID`; `409 EQUIPMENT_SERIAL_NUMBER_CONFLICT`, `EQUIPMENT_IMEI_CONFLICT` o `EQUIPMENT_ICCID_CONFLICT`. Son los mismos códigos que tenía en `fleet`.

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). Las pruebas del backend están en `src/test/java/.../equipment`; las del frontend, junto a cada archivo de `src/app/equipment`, y las de extremo a extremo en `e2e/vehicle-catalog.spec.ts` y `e2e/fleet.spec.ts`. Las reglas que vinieron de `vehiclecatalog` conservan su identificador `VCT-`; las que vinieron de una regla común a los dos catálogos, y las del inventario, que en `fleet` no tenían identificador, llevan `EQP-`.

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| EQP-R01 | El catálogo de equipos es del sistema: los tipos, sus aplicabilidades, las marcas y los modelos no llevan empresa ni Row Level Security. El inventario sí: cada equipo es de una empresa, con Row Level Security | §2.5, §7.5, §11.4, D-15, D-254 | Base | `EquipmentMigrationTests`, `RowLevelSecurityMigrationTests` | ✅ |
| EQP-R02 | El catálogo lo lee quien tiene `catalogos.ver` (los siete roles) y lo escribe quien tiene `catalogos.crear_equipamiento`, `editar_equipamiento` o `desactivar_equipamiento` (en la semilla, solo el superadministrador, D-130) | §5.6, D-95, D-130, D-141 | Servicio, Frontend | `EquipmentTypesControllerTests`, `EquipmentModelsControllerTests`, `EquipmentBrandsControllerTests`, `equipment.routes.spec.ts` | ✅ |
| EQP-R03 | Nada se borra: un tipo, una aplicabilidad, una marca y un modelo se desactivan y se reactivan, y un equipo sale del inventario de baja. La API no tiene `DELETE`, y la cuenta de la aplicación no puede borrar un equipo | §2.5, D-73, D-254 | Base, Servicio | `EquipmentTypeApplicabilityConstraintTests`, `EquipmentModelsControllerTests`, `EquipmentsControllerTests`, `EquipmentRowsAreNeverDeletedTests` | ✅ |
| EQP-R04 | El catálogo registra cuándo se creó y cuándo cambió, sin autor; el inventario, además, quién registró y quién cambió cada equipo (D-42). Guardar un registro que ya existe conserva su creación y su identificador público | D-42, D-73 | Base, Servicio | `EquipmentTypeAuditTests`, `EquipmentBrandAuditTests`, `EquipmentAuditTests` | ✅ |
| EQP-R05 | Un duplicado que la base detiene en una carrera responde el `*_CONFLICT` de su recurso, nunca un `500`: el código de un tipo, el nombre de una marca o de un modelo, y el serial, el IMEI y el ICCID de un equipo | Convenciones §7.2 | Base, Servicio | `EquipmentRacedDuplicatesTests` (el tipo), `EquipmentCommandServiceImplTests` (el equipo). La marca y el modelo, sin prueba propia de la carrera | ⚠️ |
| EQP-R06 | La semilla trae el GPS y la pantalla, activos, con sus banderas: solo el GPS exige serie y es telemático. El GPS aplica a los tres tipos de vehículo y la pantalla, a los autos | §7.5 | Base | `EquipmentMigrationTests` | ✅ |
| EQP-R07 | Desactivar un tipo, una marca o un modelo pregunta antes, con su nombre | DESIGN.md; como Estados y motivos (FLT-R25) | Frontend | `deactivation-question.spec.ts`, `equipment-types-view.spec.ts`, `equipment-types-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| EQP-R08 | Las listas se buscan, se filtran, se ordenan y se paginan en el servidor; la búsqueda no distingue mayúsculas ni tildes. Un tipo de vehículo que no existe no coincide con ningún tipo de equipamiento, y un filtro del inventario por una ubicación, un tipo o un modelo que no existe no lee nada | Convenciones §6.2, D-142 | Servicio, Frontend | `EquipmentTypeQueryServiceImplTests`, `EquipmentModelsControllerTests`, `EquipmentQueryServiceImplTests`, `equipment-brands-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| EQP-R09 | Los tipos de equipamiento se exportan a Excel desde su propio endpoint, con la búsqueda y los filtros de su lista; cada tipo nombra los tipos de vehículo a los que aplica, por su nombre y su código, y el archivo no se reutiliza cuando cambia uno de ellos. Los modelos, las marcas y el inventario no tienen exportación | D-106, D-116, D-128 | Servicio, Frontend | `EquipmentExportsIntegrationTests`, `equipment-catalog.store.spec.ts`, `equipment-types-view.spec.ts` | ✅ |
| EQP-R10 | La versión de lo que la habilitación nombra del catálogo de equipos —el equipamiento y su aplicabilidad— la da la fachada. La de los modelos y sus tipos la da `vehiclecatalog` (VCT-R31), y la exportación de la lista de la habilitación de `fleet` compone las dos | D-128, D-278 | Servicio | `EquipmentContextFacadeTests`, `VehicleCatalogContextFacadeTests` | ✅ |
| EQP-R11 | El inventario se lee con `stock.ver` en las empresas donde actúa: un equipo de otra empresa responde `404` aunque la cuenta tenga ahí otro rol, y un filtro por una empresa fuera de ese alcance no lee nada | D-99, D-254 | Servicio | `EquipmentQueryServiceImplTests`, `EquipmentsIntegrationTests` | ✅ |
| EQP-R12 | Registrar, editar y cambiar el estado de un equipo exigen `stock.crear_equipamiento`, `stock.editar_equipamiento` y `stock.cambiar_estado_equipamiento` sobre toda la empresa del equipo y todas sus marcas; sin él, `403` | §5.6, D-254 | Servicio, Frontend | `EquipmentCommandServiceImplTests`, `EquipmentsIntegrationTests` | ✅ |
| EQP-R13 | Un equipo es de una ubicación de su empresa, cualquiera, que fija su empresa, y de un proveedor de esa misma empresa y activo; su tipo y su modelo están activos, el modelo es del tipo y su marca está activa (VCT-R37) | §11.4, D-254; decisión 10 de la fase 0 del plan | Base, Servicio | `EquipmentCommandServiceImplTests`, `EquipmentsIntegrationTests`, `equipment-form-dialog.spec.ts` | ✅ |
| EQP-R14 | El serial, el IMEI y el ICCID son únicos en todo el sistema, y no por empresa, mientras el equipo no está de baja. Un tipo telemático exige el serial, un IMEI de 15 dígitos y un ICCID de 19 o 20; uno que exige serie, el serial; un IMEI o un ICCID con otro formato se rechaza en cualquier tipo | §11.4, D-254, D-263 | Base, Dominio, Servicio, Frontend | `EquipmentTests`, `EquipmentCommandServiceImplTests`, `EquipmentsIntegrationTests`, `equipment-form-dialog.spec.ts` | ✅ |
| EQP-R15 | Un equipo pasa de Disponible a Mantenimiento o De baja, de Mantenimiento a Disponible o De baja, y de Instalado a Mantenimiento; De baja es final. El inventario no lo pasa a Instalado: lo hará la instalación de `fleet` con el módulo de monitoreo, que le quitará al inventario el paso de Instalado a Mantenimiento (FLT-R152, FLT-R153; plan §2.6) | §11.4, D-254, D-263 | Dominio, Servicio | `EquipmentTests`, `EquipmentCommandServiceImplTests`, `EquipmentsControllerTests` | ✅ |
| EQP-R16 | El serial, el IMEI, la ubicación y el proveedor cambian solo mientras el equipo está Disponible; los datos técnicos, salvo de baja. Una edición cambia solo los campos que trae | §11.4, D-254 | Dominio, Servicio, Frontend | `EquipmentTests`, `EquipmentCommandServiceImplTests`, `equipment-form-dialog.spec.ts` | ✅ |
| EQP-R17 | Registrar un equipo no fija el código de su tipo: lo fija su primera instalación (VCT-R30) | D-185, D-254 | Servicio | `EquipmentCommandServiceImplTests` | ✅ |
| EQP-R18 | Un equipo nombra la unidad en que está instalado, con su placa, que responde `fleet` por el puerto `EquipmentUnits`, y su proveedor, que responde `procurement` por el puerto `EquipmentSuppliers`: `equipment` no depende de ninguno de los dos, y cada puerto tiene una sola respuesta | D-278 | Servicio | `VehicleEquipmentUnitsAdapterTests` (`fleet`), `EquipmentSuppliersWiringTests` (`procurement`) | ✅ |
| VCT-R18 | Un equipamiento aplica al menos a un tipo de vehículo; quitarle uno desactiva esa aplicabilidad y volver a marcarlo la reactiva; solo cuenta la activa | §7.5, D-73 | Base, Dominio, Frontend | `EquipmentTypeTests`, `EquipmentTypeApplicabilityPersistenceTests`, `EquipmentTypesControllerTests`, `equipment-types-dialog.spec.ts` | ✅ |
| VCT-R19 | Los códigos `GPS` y `PANTALLA` quedan fijos desde la semilla; el de otro equipamiento se corrige mientras ninguna instalación lo use | D-74, §2.4 | Dominio, Servicio, Frontend | `EquipmentTypeTests`, `EquipmentTypesControllerTests`, `equipment-types-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R20 | Dos cambios de un mismo tipo de equipamiento se deciden de a uno | [Patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md) | Servicio | `EquipmentTypeLockTests` | ✅ |
| VCT-R28 | Lo que la habilitación necesita del catálogo de equipos para cada modelo se responde una sola vez y en lote: los equipos activos que aplican a su tipo, con su aplicabilidad activa, y cuál trae de fábrica: la pantalla, si el modelo la trae. El tipo de cada modelo y su pantalla de fábrica los da `vehiclecatalog`, también en una lectura por lote. El catálogo no calcula qué se exige: lo decide `fleet`, que lo cruza con su configuración (D-260, FLT-R148) | §7.5, §11.9, D-165, D-260, D-278 | Servicio | `EquipmentContextFacadeTests`, `ModelRequirementsTests` (`fleet`), `EnablementCharacterizationTests` (`fleet`), `vehicle-enablement-view.spec.ts` (`fleet`), `e2e/enablement.spec.ts` | ✅ |
| VCT-R30 | El código de un equipamiento queda fijo desde su primera instalación, que `fleet` avisa por la fachada en su transacción y `equipment` guarda en `has_installations`; `GPS` y `PANTALLA`, desde la semilla | D-74, D-185, §2.4, §7.5 | Base, Dominio, Servicio, Frontend | `EquipmentTypeTests`, `EquipmentContextFacadeTests`, `EquipmentTypesControllerTests`, `VehicleEquipmentIntegrationTests` (`fleet`) | ✅ |
| VCT-R33 | Un modelo de equipamiento es de un tipo y de una marca de equipamiento, que no cambian. Su nombre, de 1 a 60 caracteres, se guarda recortado y en mayúsculas y no se repite en su tipo y su marca, sin distinguir mayúsculas ni los espacios de los bordes; otro tipo u otra marca pueden usarlo | §7.5, D-254 | Base, Dominio, Servicio, Frontend: la edición recibe solo el nombre, y el diálogo muestra el tipo y la marca de un modelo editado como texto fijo | `EquipmentModelTests`, `EquipmentModelCommandServiceImplTests`, `EquipmentModelsControllerTests`, `equipment-model-form-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R34 | Un modelo de equipamiento nuevo es de un tipo activo, y uno inactivo se reactiva solo mientras su tipo está activo (`422 EQUIPMENTTYPE_INACTIVE`); desactivarlo no espera a nada. El diálogo de un modelo nuevo ofrece solo los tipos activos. Lo mismo vale para su marca (VCT-R37) | §7.5, D-254, D-84 | Servicio, Frontend | `EquipmentModelCommandServiceImplTests`, `EquipmentModelsControllerTests`, `equipment-model-form-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R35 | Un tipo de equipamiento dice si su equipo lleva IMEI y SIM (`requires_telematics`). Un tipo con código `GPS` siempre lo lleva; una edición que no manda el dato conserva lo guardado. Lo que exige al equipo del inventario —su serial, un IMEI de 15 dígitos y un ICCID de 19 o 20— lo aplica el inventario (EQP-R14) | §7.5, D-254 | Base, Dominio, Servicio, Frontend: el formulario del tipo no lo muestra ni lo envía, y el inventario lo lee del catálogo de su propio contexto | Ninguna prueba del backend lo nombra: las de `EquipmentType` pasan el valor guardado; `equipment-catalog.store.spec.ts` lo lee | ⚠️ |
| VCT-R36 | Una marca de equipamiento no repite nombre; el nombre, de 1 a 80 caracteres, se guarda recortado y en mayúsculas, y la unicidad se mide sobre lo guardado. Se registra, se renombra, se activa y se desactiva en su lugar, en «Gestionar marcas» de Catálogos › Equipamiento, que se busca y se pagina en el servidor, un formulario a la vez; no se borra | §7.5 | Base, Dominio, Servicio, Frontend | `EquipmentBrandTests`, `EquipmentBrandCommandServiceImplTests`, `EquipmentBrandsControllerTests`, `EquipmentBrandAuditTests`, `equipment-brands-dialog.spec.ts`, `equipment-brand-form.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R37 | Una marca de equipamiento inactiva no recibe modelos nuevos ni reactiva uno (`422 EQUIPMENTBRAND_INACTIVE`), y el inventario no registra un equipo de un modelo suyo; lo que ya la tiene la conserva. El diálogo de un modelo nuevo ofrece solo las marcas activas | §7.5, D-84 | Servicio, Frontend | `EquipmentModelCommandServiceImplTests`, `EquipmentCommandServiceImplTests`, `equipment-model-form-dialog.spec.ts` | ✅ |

---

## 5. Relación con otros contextos

**Las dependencias.** `equipment` usa a `organization`, `vehiclecatalog` y `documents` por sus fachadas, y a `shared`; `ModularityTests` lo verifica. Ninguna forma un ciclo: ninguno de los tres llama a `equipment`. `fleet` y `procurement` dependen de `equipment::acl`, y `equipment` no depende de ellos: lo que necesita de cada uno lo pregunta por un puerto suyo que el otro implementa (D-278).

**Lo que ofrece: `EquipmentContextFacade`**, solo con lo que usa `fleet` (plan, fase 2 §2.2). La llama `fleet`, por su `ExternalEquipmentService`.

| Miembro | Para qué |
|---|---|
| `fetchEquipmentTypeReferenceByPublicId`, `fetchEquipmentTypeReferencesByIds` | El tipo de equipamiento que se instala o que se marca en la configuración de lo que se exige, activo o no, con si exige serie y si es telemático, y los de una lista |
| `fetchEquipmentTypeReferences()` | Todos los tipos, activos o no, por su código y en una lectura: la lista de Requisitos de equipos de `fleet` (D-260) |
| `fetchEquipmentApplicabilityByModelIds(…)` | Lo que admite cada modelo de un lote (`ModelEquipmentApplicability`): los equipos activos que aplican a su tipo, cada uno con si el modelo lo trae de fábrica (VCT-R28). No dice qué se exige: lo decide `fleet` con su configuración (D-260). Un modelo que `vehiclecatalog` no tiene queda fuera |
| `markEquipmentTypeInUse(…)` | Marca la primera instalación de un tipo, idempotente y en la transacción de quien llama; desde entonces su código queda fijo (VCT-R30) |
| `fetchEnablementDataVersion()` | La versión del equipamiento y su aplicabilidad, para la exportación de la lista de la habilitación (EQP-R10) |

Sin miembros de los modelos ni de las marcas: ningún otro contexto los lee.

**Los puertos que declara.** `equipment` no depende de `procurement` ni de `fleet`: lo que necesita de ellos lo pregunta por puertos propios, en `equipment::acl`, que ellos implementan (EQP-R18).
- **`EquipmentSuppliers`**, que implementa `procurement` (`SupplierEquipmentSuppliersAdapter`): el proveedor de un equipo, por su identificador público y en lote, con su empresa, su razón social y si está activo. `equipment` pregunta a todos los que haya: sin ninguno, todo equipo con proveedor respondería `SUPPLIER_NOT_FOUND` sin que nada falle al arrancar, y `EquipmentSuppliersWiringTests` lo vigila.
- **`EquipmentUnits`**, que implementa `fleet` (`VehicleEquipmentUnitsAdapter`): la unidad en que está un equipo, en lote, con su identificador público y su placa, bajo el alcance de quien llama.

**Lo que usa de otros contextos**, por sus fachadas:

| Contexto | Para qué |
|---|---|
| `organization` | La empresa de un equipo, si existe y quien llama la alcanza; las ubicaciones de un equipo y de una lista, con su empresa; y los nombres de las empresas de una lista |
| `vehiclecatalog` | Los tipos de vehículo a los que aplica un tipo de equipamiento, activos o no, por su identificador público y por su identificador, en lote; de cada modelo de un lote, su tipo de vehículo y si trae la pantalla de fábrica (VCT-R28); y la versión de lo que muestra de los tipos de vehículo, para la exportación (EQP-R09) |
| `documents` | La exportación de los tipos de equipamiento (D-125) |
| `shared` | El alcance de quien llama (D-99), las clases base de la persistencia y el reintento de un interbloqueo (D-128) |

**Frontend.** `EquipmentFacade` ofrece a otros contextos solo los tipos de equipamiento, activos o no, con su código y su nombre, con su propia carga y su propio error, apartados de los de la pantalla de Equipamiento: su único consumidor es el historial de la unidad de `fleet`, que nombra el equipo que perdió una unidad (D-260). El inventario, del mismo contexto, lee los tipos, las marcas y los modelos de su propio store. `equipment` lee los tipos de vehículo de `VehicleCatalogFacade` y las empresas y las ubicaciones de `OrganizationFacade`.

---

## 6. Estado y pendientes

**Estado:** construido, con dos reglas en ⚠️ (EQP-R05 y VCT-R35) y el enlace del inventario con la instalación decidido y por construir con el módulo de monitoreo. Falta la validación de este documento por el responsable. No se conocen diferencias con las convenciones de la API.

| Qué | Quién y cuándo |
|---|---|
| El inventario unido a la instalación: instalar elige un equipo Disponible de la empresa y lo pasa a Instalado, el motivo del retiro decide adónde vuelve, y una unidad con equipos instalados no se anula ni pasa a Inactivo. Solo `fleet` instala y retira, y `equipment` escribe sus filas por su fachada (FLT-R152 a FLT-R154; plan §2.6; D-263, D-264) | El equipo, con el módulo de monitoreo |
| `fleet` deja de escribir la marca de la primera instalación: `equipment` preguntará por un puerto, `EquipmentTypeUsage`, si un tipo ya tuvo instalaciones, como hace `CurrencyUsage` (mapa objetivo §3.7) | La fase 8 del plan, antes o después del monitoreo, nunca durante |
| Las pruebas que faltan: la carrera del nombre de una marca y de un modelo (EQP-R05) y una prueba del backend que nombre `requires_telematics` (VCT-R35) | Sin fecha |
| Validar este documento | El responsable |

---

## 7. Preguntas abiertas

No quedan preguntas abiertas. Las del inventario las respondió el responsable el 19/11/2026: cómo se enlaza con la instalación (D-263, D-264, sección 6) y que los identificadores sean únicos en todo el sistema (D-263).
