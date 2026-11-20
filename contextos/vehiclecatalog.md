# Contexto `vehiclecatalog` — ERP Millennium Mobility

**Estado:** borrador para validar · **Fuente:** diccionario de datos (§2.2, §2.4, §2.5, §7.1 a §7.7, §8.1 y las decisiones que cita cada regla), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (P3 y apartado 3.2) y el código. Si este documento y el diccionario no coinciden, manda el diccionario.

---

## 1. Qué es y qué no es

`vehiclecatalog` responde con qué se describe una unidad: de qué tipo es, de qué marca y de qué modelo, de qué color, y qué equipamiento se le instala, con la marca y el modelo comercial de cada equipo. Qué equipo exige la habilitación no lo dice el catálogo: lo dice `fleet` (D-260). Sus catálogos son del sistema, no de cada empresa (D-15): un Chevrolet Sail es el mismo en Perú y en Colombia (VCT-R01). Los escribe quien tiene crear, editar o desactivar de su familia —con la semilla, solo el superadministrador— y los leen los siete roles del sistema (VCT-R02). No llama a ningún contexto; `fleet` y `procurement` lo usan por su fachada.

**Las ocho tablas que posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `vehicle_types` | Los tipos de vehículo —`AUTO`, `MOTO`, `TRIMOTO`—, con su nombre y el nombre comercial con que los conoce la operación | §7.1 |
| `vehicle_brands` | Las marcas del vehículo: CHEVROLET, TVS, BAJAJ, HYUNDAI | §7.2 |
| `vehicle_models` | Los modelos de cada marca, con su tipo, su cilindraje, su combustible, si traen pantalla de fábrica y si ya tienen unidades | §7.3, D-107 |
| `colors` | Los colores | §7.4 |
| `equipment_types` | Lo que se instala en una unidad —`GPS`, `PANTALLA`—, con si exige número de serie, si su equipo lleva IMEI y SIM y si ya tuvo una instalación. Si se exige para pasar a Disponible ya no lo dice aquí: lo dice la configuración de `fleet` (D-260) | §7.5, D-185, D-254 |
| `equipment_type_vehicle_types` | A qué tipos de vehículo aplica cada equipamiento | §7.5, D-73 |
| `equipment_brands` | Las marcas de equipamiento —TELTONIKA, QUECLINK—, que agrupan sus modelos | §7.5 |
| `equipment_models` | El modelo comercial de cada tipo de equipamiento, de una marca —«Fleet 790» del GPS—, con nombre único en su tipo y su marca | §7.5, D-254 |

**Lo que parece suyo y no lo es**

| Parece de `vehiclecatalog` | Es de | Por qué |
|---|---|---|
| La marca comercial: Lizzo, Lima Bikes | `organization` (§6.3) | Es bajo qué nombre coloca Millennium la unidad, no quién la fabrica. Una moto TVS King vendida bajo Lima Bikes usa las dos tablas |
| Las unidades de un modelo | `fleet` (§8.1) | `vehiclecatalog` solo sabe si un modelo tiene alguna, porque `fleet` se lo avisa (D-107). Nunca lee las tablas de `fleet` |
| Los estados del vehículo, los motivos y los tipos de documento | `fleet` (mapa §6.4) | Son de los bloques 6 y 9 |
| El equipamiento instalado en una unidad, y el equipo físico en el almacén de una empresa | `fleet` (§11.4, D-254) | Aquí están solo el tipo y el modelo de equipamiento; `fleet` los lee por la fachada. Que un equipo de un tipo telemático lleve su serial, su IMEI y su ICCID, o que no elija un modelo inactivo, lo exige `fleet` |
| Qué equipo exige una unidad, también si requiere instalar pantalla | `fleet` (§11.4, D-260) | `vehiclecatalog` responde, una sola vez y en lote, qué equipo aplica a cada modelo y cuál trae de fábrica (VCT-R17, VCT-R28); `fleet` lo cruza con su configuración de lo que se exige y de lo que es de seguridad |
| La placa | `fleet` (§7.6) | No tiene tabla: es una columna de la unidad |
| Las tarifas por kilómetro de un modelo | Ventas, fuera del alcance (§7.3) | Le apuntarán al modelo cuando lleguen |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Tipo de vehículo | Auto, moto o trimoto. Clasifica la unidad y activa las reglas que dependen del tipo (§7.1) | El "tipo de vehículo" del Registro de Clientes, `NUEVO / SEMIPREMIUM`, que se deduce de si la unidad tuvo contratos |
| Nombre comercial | Cómo conoce la operación un tipo: Leasing Car, Leasing Bike, Leasing Tuk Tuk | El nombre del tipo, que puede no coincidir algún día |
| Marca | Quien fabrica el vehículo | La marca comercial (§6.3) |
| Modelo | La denominación oficial dentro de una marca: `SAIL SEDAN 1.5L LT MT` de CHEVROLET. El nombre solo identifica junto a la marca | El modelo de equipamiento |
| Cilindraje | La cifra comercial del modelo en cc, entera: 125, 200, 1500 | La cifra técnica de la ficha (199,26) |
| Pantalla de fábrica | Que el modelo ya trae la pantalla instalada | Que el equipo pantalla aplique a su tipo (§7.5) |
| Aplicabilidad | Que un equipamiento tenga sentido en un tipo de vehículo | Una instalación, que es de `fleet` |
| Marca de equipamiento | Quien fabrica un equipo: TELTONIKA. Agrupa sus modelos | La marca de un vehículo y la marca comercial (§6.3) |
| Modelo de equipamiento | El modelo comercial de un tipo de equipamiento, de una marca: «FMB920» del GPS. El nombre solo identifica junto a su tipo y su marca (D-254) | El modelo de un vehículo |
| Telemático | Un tipo de equipamiento cuyo equipo lleva IMEI y tarjeta SIM, como el GPS (D-254) | Que exija número de serie |
| Código que lee el programa | `GPS` y `PANTALLA`: la pantalla de fábrica, la telemetría del GPS, la carga inicial y los reportes los reconocen por su código, así que quedan fijos desde la semilla (D-74) | Un código propio, que se corrige mientras ninguna instalación lo use; y lo que exige la habilitación, que no sale del código sino de la configuración de `fleet` (D-260) |
| Modelo con unidades | Un modelo del que `fleet` ya registró al menos una unidad. Desde entonces su tipo de vehículo queda fijo (D-107) | Un modelo activo, que solo dice si se sigue comprando |
| Equipamiento con instalaciones | Un tipo de equipamiento que `fleet` ya instaló en alguna unidad. Desde entonces su código queda fijo (D-185) | Un equipamiento activo, que solo dice si se sigue instalando |
| Desactivar | Dejar de ofrecer un registro para lo nuevo; lo que ya lo usa lo conserva | Borrar, que no existe |

---

## 2. Ciclo de vida de cada agregado

Nada de `vehiclecatalog` se borra: ni los tipos, ni las marcas, ni los modelos, ni los colores, ni los equipamientos, ni sus aplicabilidades, ni sus marcas, ni sus modelos (VCT-R03). Registran cuándo se crearon y cuándo cambiaron, sin autor (VCT-R04). Los códigos y los nombres se guardan recortados y en mayúsculas (VCT-R05).

### Tipo de vehículo

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en Catálogos › Tipos de vehículo, con su código, su nombre y su nombre comercial. La semilla trae `AUTO`, `MOTO` y `TRIMOTO`, que son los códigos que interpreta el programa (VCT-R22) |
| Qué cambia | El nombre y el nombre comercial |
| Qué queda fijo | El código, desde el alta: lo leen las reglas del sistema (VCT-R06) |
| Retiro | Se desactiva y se puede reactivar. Un tipo inactivo no se elige para un modelo nuevo ni para cambiar el de uno existente (VCT-R15), y no se registra una unidad de un modelo de ese tipo (VCT-R16). Los modelos que ya lo tienen lo conservan, y los equipamientos que aplican a él siguen aplicando: sus unidades pueden seguir necesitándolos (§7.5) |
| Nunca | Se borra; repite código (VCT-R06) |

### Marca

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en el diálogo de las marcas de Catálogos › Marcas y modelos (VCT-R27), con su nombre: `Tvs` y `TVS` son la misma marca |
| Qué cambia | El nombre |
| Retiro | Se desactiva y se reactiva en el mismo diálogo. Una marca inactiva no recibe modelos nuevos (VCT-R15), y no se registra una unidad de un modelo suyo (VCT-R16). Sus modelos y sus unidades la siguen referenciando y se siguen editando |
| Nunca | Se borra; repite nombre (VCT-R07) |

### Modelo

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en Catálogos › Marcas y modelos, eligiendo su marca, con su nombre, su tipo de vehículo, su cilindraje, su combustible y si trae pantalla de fábrica. La marca y el tipo tienen que estar activos (VCT-R15). El combustible nace vacío y se elige (VCT-R12). Nace sin unidades |
| Qué cambia | El nombre, el cilindraje, el combustible, la pantalla de fábrica y, mientras el modelo no tenga unidades, el tipo de vehículo (VCT-R14). Un tipo nuevo tiene que estar activo; el inactivo que el modelo ya tenía se conserva (VCT-R15) |
| Qué queda fijo | La marca, desde el alta (VCT-R13). El tipo de vehículo, desde la primera unidad que `fleet` registra con el modelo (VCT-R14): cambiarlo contradiría el tipo de las unidades que ya existen |
| Retiro | Se desactiva y se reactiva. Las unidades que ya lo tienen lo conservan; ninguna nueva lo elige (VCT-R16) |
| Nunca | Se borra; repite nombre dentro de su marca (VCT-R08); lleva cilindraje si es eléctrico (VCT-R10); vuelve a no tener unidades |

### Color

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en Catálogos › Colores, con su nombre |
| Qué cambia | El nombre |
| Retiro | Se desactiva y se reactiva. Una unidad nueva no elige un color inactivo, y la que ya lo tiene lo conserva (`fleet`, FLT-R07) |
| Nunca | Se borra; repite nombre |

### Tipo de equipamiento

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en el diálogo de los tipos de Catálogos › Equipamiento, con su código, su nombre, si requiere número de serie y los tipos de vehículo a los que aplica, al menos uno (VCT-R18). La semilla trae `GPS` y `PANTALLA` con sus banderas y sus tipos (VCT-R22), y el GPS es telemático. Un tipo con código `GPS` es siempre telemático; los demás nacen sin serlo, salvo que la API reciba `requiresTelematics` (VCT-R35). Un tipo nuevo nace sin requisito: si se exige para Disponible y si es de seguridad lo marca `fleet`, en Catálogos › Requisitos de equipos (D-260) |
| Qué cambia | El nombre, si requiere número de serie, los tipos a los que aplica y el código mientras no quede fijo. Si es telemático cambia solo por la API: una edición que no lo manda conserva lo guardado, y la pantalla nunca lo manda (D-254) |
| Qué queda fijo | El código de `GPS` y el de `PANTALLA`, desde la semilla (VCT-R19); el de otro equipamiento, desde su primera instalación (VCT-R30) |
| Retiro | Se desactiva y se reactiva. Quitarle un tipo de vehículo desactiva esa aplicabilidad, y volver a marcarlo reactiva la misma (VCT-R18). Para que deje de instalarse en todo se desactiva el equipamiento, no se le quitan todos los tipos. Un tipo inactivo no recibe modelos nuevos ni reactiva los suyos (VCT-R34) |
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
| Retiro | Se desactiva y se reactiva; se reactiva solo mientras su tipo y su marca están activos (VCT-R34, VCT-R37). Los equipos del inventario que ya lo tienen lo conservan; ninguno nuevo lo elige (`fleet`, §11.4) |
| Nunca | Se borra; repite nombre dentro de su tipo y su marca (VCT-R33) |

---

## 3. Flujos funcionales

Las lecturas piden `catalogos.ver` y las escrituras, crear, editar o desactivar de su familia; con la semilla escribe solo el superadministrador y los demás roles leen, aquí y en los selectores de las pantallas que usan los catálogos, como el alta de una unidad (VCT-R02). Las pantallas son cuatro, en el grupo Catálogos: Tipos de vehículo, Marcas y modelos, Colores y Equipamiento; cada una se abre solo con `catalogos.ver` (D-141), y cada acción se ofrece solo con su permiso. Las listas se buscan sin distinguir mayúsculas ni tildes, se filtran por estado, se ordenan y se paginan en el servidor (VCT-R24). Desactivar un registro pregunta antes (VCT-R23).

### F1 · Mantener los tipos de vehículo

**Rol:** superadministrador. **Pantalla:** Catálogos › Tipos de vehículo (`/vehicle-types`).

1. Busca por código, nombre o nombre comercial.
2. Registra un tipo con su código, su nombre formal y su nombre comercial. El sistema revisa que el código no se repita, una vez en mayúsculas (VCT-R05, VCT-R06), también si dos altas llegan a la vez (VCT-R21).
3. Edita los nombres; el código no cambia (VCT-R06).
4. Desactiva o activa el tipo.

**Si algo falla:** el diálogo lo dice (`VEHICLETYPE_CONFLICT`).

### F2 · Gestionar las marcas

**Rol:** quien tiene crear, editar o desactivar marcas y modelos; con la semilla, el superadministrador. **Pantalla:** Catálogos › Marcas y modelos, diálogo de las marcas (`/vehicle-brands`; D-136).

1. Busca las marcas por nombre, en el servidor (VCT-R24).
2. Registra una marca o la renombra en el mismo diálogo, un formulario a la vez (VCT-R27). El sistema revisa que el nombre no se repita, una vez en mayúsculas (VCT-R05, VCT-R07), también en una carrera (VCT-R21).
3. Desactiva o activa una marca.
4. Al cerrar el diálogo después de un cambio, la lista vuelve a leer las marcas de su filtro y sus modelos.

**Si algo falla:** el diálogo lo dice en la misma fila, que conserva lo escrito (`VEHICLEBRAND_CONFLICT`).

### F3 · La lista de todos los modelos

**Rol:** quien lee los catálogos, con `catalogos.ver`. **Pantalla:** Catálogos › Marcas y modelos (`GET /vehicle-models`), la lista de todos los modelos del catálogo (D-136).

1. La lista muestra cada modelo con su marca, porque dos marcas pueden tener modelos con el mismo nombre (VCT-R26).
2. Busca por el nombre del modelo y ordena; por omisión, por la marca y después por el modelo.
3. Filtra por marca, por tipo de vehículo y por estado. El filtro de marca ofrece también las inactivas, para encontrar los modelos de una marca retirada. Una marca o un tipo que no existe no coincide con ningún modelo (VCT-R26).
4. La marca del filtro queda en la dirección: una recarga o un enlace abren la misma marca, y el enlace a una marca lleva a la lista filtrada por ella (VCT-R26).
5. Desde la lista se registra un modelo (F4), y se edita, se activa o se desactiva cada uno, cada acción con su permiso.

**Si algo falla:** la lista lo dice.

### F4 · Registrar y corregir un modelo

**Rol:** superadministrador. **Pantalla:** Catálogos › Marcas y modelos.

1. **La marca.** Un modelo nuevo la elige primero, entre las activas (VCT-R15); empieza en la del filtro de la lista si está activa. Al editar, la marca no cambia (VCT-R13).
2. **El nombre**, que no se repite en su marca (VCT-R08).
3. **El tipo de vehículo**, entre los activos; al editar, también el inactivo que el modelo ya tiene (VCT-R15). Si el modelo ya tiene unidades, el tipo no cambia (VCT-R14).
4. **El combustible**, que en un modelo nuevo nace vacío y puede quedar vacío si no se conoce (VCT-R11, VCT-R12).
5. **El cilindraje**, la cifra comercial en cc, entera, vacía en un eléctrico (VCT-R09, VCT-R10).
6. **Si trae pantalla de fábrica** (VCT-R17).
7. Guarda: `POST /vehicle-brands/{id}/vehicle-models`, bajo la marca elegida, o `PUT /vehicle-models/{id}`.

**Si algo falla:** el diálogo lo dice y queda abierto con lo escrito (`VEHICLEMODEL_CONFLICT`, `VALIDATION_ERROR`, `VEHICLEMODEL_TYPE_LOCKED`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE`). Ante un tipo bloqueado vuelve a poner el que el modelo tiene; ante una marca o un tipo inactivos, vuelve a leerlos.
**Resultado:** la lista de todos los modelos vuelve a leer su página.

### F5 · Mantener los colores

**Rol:** superadministrador. **Pantalla:** Catálogos › Colores (`/colors`).

1. Busca por nombre.
2. Registra un color por su nombre, en mayúsculas, para que `Blanco`, `BLANCO` y `blanco` no sean tres colores (VCT-R05).
3. Lo renombra, lo desactiva o lo activa.

**Si algo falla:** el diálogo lo dice (`COLOR_CONFLICT`, también en una carrera, en el alta y al renombrar; VCT-R21).

### F6 · Mantener el equipamiento: sus modelos, sus marcas y sus tipos

**Rol:** superadministrador. **Pantalla:** Catálogos › Equipamiento: la lista de los modelos de equipamiento, con los diálogos de sus marcas y de sus tipos (D-254).

**Los modelos de equipamiento** (`/equipment-models`).

1. La lista muestra los modelos con su marca y su tipo de equipamiento, y se filtra por tipo, por marca y por estado.
2. Registra un modelo bajo un tipo y una marca activos, con su nombre comercial (VCT-R33, VCT-R34, VCT-R37).
3. Edita solo el nombre; el tipo y la marca no cambian (VCT-R33).
4. Lo desactiva o lo activa; reactivarlo exige que su tipo y su marca estén activos (VCT-R34, VCT-R37).

**Si algo falla:** el diálogo lo dice (`EQUIPMENTMODEL_CONFLICT`, `EQUIPMENTTYPE_INACTIVE`, `EQUIPMENTBRAND_INACTIVE`, `EQUIPMENTMODEL_NOT_FOUND`).

**Las marcas de equipamiento** (`/equipment-brands`). «Gestionar marcas» abre un diálogo que lista las marcas, las busca y las pagina en el servidor, y las registra, renombra, activa y desactiva en su lugar, un formulario a la vez, sin abrir otro diálogo encima (VCT-R36). Desactivar una pregunta antes (VCT-R23). Al cerrar el diálogo después de un cambio, la lista vuelve a leer sus modelos.

**Si algo falla:** el diálogo lo dice y conserva lo escrito (`EQUIPMENTBRAND_CONFLICT`, también en una carrera; VCT-R21).

**Los tipos de equipamiento** (`/equipment-types`). El diálogo los lista, los busca y los pagina en el servidor; la API busca por código o nombre y filtra también por el tipo de vehículo al que aplica hoy.

1. Registra un tipo con su código, su nombre, si requiere serie y los tipos de vehículo activos a los que aplica, al menos uno (VCT-R18). La pantalla no muestra ni envía si es telemático: lo dice el backend (VCT-R35). Si el tipo se exige para Disponible o es de seguridad no se marca aquí: lo marca `fleet` en Catálogos › Requisitos de equipos (D-260).
2. Lo edita: el código de `GPS`, de `PANTALLA` y de un equipamiento que ya tuvo una instalación no cambia (VCT-R19, VCT-R30); el formulario ofrece también los tipos de vehículo inactivos a los que ya aplica, para poder desmarcarlos (VCT-R18).
3. Lo desactiva o lo activa.

El sistema bloquea el equipamiento mientras decide un cambio, para que dos ediciones simultáneas no se pisen (VCT-R20).

**Si algo falla:** el formulario lo dice y conserva lo escrito (`EQUIPMENTTYPE_CONFLICT`, `VEHICLETYPE_NOT_FOUND`, `EQUIPMENTTYPE_CODE_LOCKED`); si el código quedó fijo mientras tanto, vuelve al guardado.

### F7 · Exportar un catálogo

**Rol:** quien lee los catálogos, con `catalogos.ver`. **Pantalla:** las cuatro listas (D-116).

1. Busca y filtra la lista y la exporta a Excel con la misma búsqueda, los mismos filtros y el mismo orden, por la plataforma de documentos (VCT-R25; convenciones §8).
2. Marcas y modelos exporta los modelos de la lista, cada uno con su marca; las marcas no se exportan por separado (D-136). Equipamiento exporta todos los tipos de equipamiento, sin los filtros de la lista de modelos; los modelos y las marcas de equipamiento no tienen exportación.

Los archivos llevan códigos y nombres, nunca un identificador (D-51); sus columnas están en convenciones §8.1.

**Si algo falla:** la pantalla dice que no pudo descargar el archivo.

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). Las pruebas del backend están en `src/test/java/.../vehiclecatalog`; las del frontend, junto a cada archivo de `src/app/vehiclecatalog`, y las de extremo a extremo en `e2e/vehicle-catalog.spec.ts`.

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| VCT-R01 | Los catálogos de flota son del sistema: no llevan empresa ni Row Level Security | §7, D-15 | Base | `VehicleCatalogMigrationTests` | ✅ |
| VCT-R02 | Los lee quien tiene `catalogos.ver` (los siete roles) y los escribe quien tiene crear, editar o desactivar de esa familia (en la semilla, solo el superadministrador, D-130); el permiso de los catálogos generales no escribe estos | §5.6, D-95, D-130, D-141 | Servicio, Frontend | `VehicleCatalogPermissionIntegrationTests`, los `*ControllerTests`, `VehicleModelsListIntegrationTests`, `brands-and-models-view.spec.ts`, `vehicle-brands-dialog.spec.ts`, `app-shell.spec.ts`, `e2e/shell.spec.ts` | ✅ |
| VCT-R03 | Nada se borra: un registro se desactiva y se reactiva, y una aplicabilidad también | §2.5, D-73 | Base, Servicio | Los `*AuditTests`, `EquipmentTypeApplicabilityConstraintTests`, `VehicleTypesControllerTests`, los `*ControllerTests`, `EquipmentModelsControllerTests` | ✅ |
| VCT-R04 | Registran cuándo se crearon y cambiaron, sin autor | D-42, D-73 | Base, Servicio | `ColorAuditTests`, `VehicleBrandAuditTests`, `VehicleModelAuditTests`, `VehicleTypeAuditTests`, `EquipmentTypeAuditTests`, `EquipmentBrandAuditTests` | ✅ |
| VCT-R05 | Los códigos y los nombres de tipos, marcas, modelos y colores se guardan recortados y en mayúsculas, y la unicidad se mide sobre lo guardado | §2.2 | Base, Dominio, Frontend | `ColorCommandServiceImplTests`, `VehicleBrandCommandServiceImplTests`, `VehicleTypeCommandServiceImplTests`, `color-form-dialog.spec.ts` | ✅ |
| VCT-R06 | El código de un tipo de vehículo no se repite y no cambia | §7.1 | Base, Dominio, Frontend | `VehicleTypeTests`, `VehicleTypesControllerTests`, `vehicle-type-form-dialog.spec.ts` | ✅ |
| VCT-R07 | Una marca no repite nombre | §7.2 | Base, Servicio | `VehicleBrandsControllerTests`, `brands-and-models.store.spec.ts`, `vehicle-brands-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R08 | Una marca no tiene dos modelos con el mismo nombre; otra marca puede usarlo | §7.3 | Base, Servicio | `VehicleModelConstraintTests`, `VehicleModelsControllerTests` | ✅ |
| VCT-R09 | El cilindraje es un entero mayor que cero, o vacío; no tiene máximo salvo el que cabe en su columna, y un decimal se rechaza en vez de cortarse | §7.3 | Base, Servicio, Frontend | `VehicleModelConstraintTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts` | ✅ |
| VCT-R10 | Un modelo eléctrico no lleva cilindraje | §7.3 | Base, Dominio, Frontend | `VehicleModelConstraintTests`, `VehicleModelTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts` | ✅ |
| VCT-R11 | El combustible es uno de la lista cerrada, guardado en minúsculas y sin tildes, o vacío | §7.3 | Base, Dominio, Servicio | `VehicleModelConstraintTests`, `VehicleModelCommandServiceImplTests`, `FuelTypeTests` | ✅ |
| VCT-R12 | El combustible de un modelo nuevo no se propone: nace vacío y se elige | D-109 | Servicio, Frontend | `vehicle-model-form-dialog.spec.ts`, `vehicle-model-assembler.spec.ts`, `VehicleModelCommandServiceImplTests`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R13 | La marca de un modelo no cambia | §7.3 | Dominio, Servicio, Frontend | `VehicleModelCommandServiceImplTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts` | ✅ |
| VCT-R14 | El tipo de vehículo de un modelo se corrige mientras el modelo no tenga unidades; desde la primera queda fijo. `fleet` avisa la primera unidad por la fachada, en su misma transacción | D-107, §7.3 | Base, Dominio, Servicio, Frontend | `VehicleModelTests`, `VehicleModelCommandServiceImplTests`, `VehicleModelsControllerTests`, `VehicleCatalogContextFacadeTests`, `VehicleCatalogMigrationTests`, `VehicleRetiredReferencesTests` (`fleet`), `vehicle-model-form-dialog.spec.ts`, `brands-and-models.store.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R15 | Un modelo nuevo es de una marca activa y de un tipo activo; al editarlo, un tipo nuevo tiene que estar activo, y el inactivo que ya tenía se conserva. El diálogo de un modelo nuevo ofrece solo las marcas activas | D-108, D-84 | Servicio, Frontend | `VehicleModelCommandServiceImplTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts`, `brands-and-models.store.spec.ts`, `brands-and-models-view.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R16 | Una unidad nueva no se registra con un modelo inactivo, ni con uno cuya marca o tipo estén inactivos | D-108, §8.1 | Servicio, en `fleet` (FLT-R07), con los estados que da la fachada | `VehicleCatalogContextFacadeTests`, `VehicleRetiredReferencesTests` (`fleet`) | ✅ |
| VCT-R17 | Lo que el catálogo dice de la pantalla —si el equipo aplica al tipo y si el modelo la trae de fábrica— sale de sus datos, nunca de comparar textos; si se exige lo decide `fleet` con su configuración (D-260) | §7.3, §7.5, D-165 | Base, Servicio | `VehicleModelTests`, `VehicleCatalogMigrationTests`, `VehicleCatalogContextFacadeTests` | ✅ |
| VCT-R18 | Un equipamiento aplica al menos a un tipo de vehículo; quitarle uno desactiva esa aplicabilidad y volver a marcarlo la reactiva; solo cuenta la activa | §7.5, D-73 | Base, Dominio, Frontend | `EquipmentTypeTests`, `EquipmentTypeApplicabilityPersistenceTests`, `EquipmentTypesControllerTests`, `equipment-types-dialog.spec.ts` | ✅ |
| VCT-R19 | Los códigos `GPS` y `PANTALLA` quedan fijos desde la semilla; el de otro equipamiento se corrige mientras ninguna instalación lo use | D-74, §2.4 | Dominio, Servicio, Frontend | `EquipmentTypeTests`, `EquipmentTypesControllerTests`, `equipment-types-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R20 | Dos cambios de un mismo tipo de equipamiento se deciden de a uno | [Patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md) | Servicio | `EquipmentTypeLockTests` | ✅ |
| VCT-R21 | Un duplicado que la base detiene en una carrera responde el `*_CONFLICT` de su recurso, nunca un `500` | Convenciones §7.2 | Base, Servicio | `VehicleCatalogRacedDuplicatesTests`, `VehicleModelCommandServiceImplTests` | ✅ |
| VCT-R22 | La semilla trae los tres tipos con sus dos nombres, y el GPS y la pantalla con sus banderas y sus tipos | §7.1, §7.5 | Base | `VehicleCatalogMigrationTests` | ✅ |
| VCT-R23 | Desactivar un registro pregunta antes, con su nombre | DESIGN.md; como Estados y motivos (FLT-R25) | Frontend | `deactivation-question.spec.ts`, `vehicle-types-view.spec.ts`, `colors-view.spec.ts`, `equipment-types-view.spec.ts`, `brands-and-models-view.spec.ts`, `vehicle-brands-dialog.spec.ts`, `equipment-types-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R24 | Las listas se buscan, se filtran, se ordenan y se paginan en el servidor; la búsqueda no distingue mayúsculas ni tildes; un filtro por un tipo de vehículo que no existe no coincide con ninguna fila | Convenciones §6.2, D-142 | Servicio, Frontend | `VehicleTypesControllerTests`, `ColorsControllerTests`, `EquipmentTypeQueryServiceImplTests`, `VehicleModelQueryServiceImplTests`, `VehicleModelsListIntegrationTests`, `EquipmentModelsControllerTests`, `vehicle-brands-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R25 | Los tipos, los colores, el equipamiento, los modelos de una marca y los de todas se exportan a Excel desde su propio endpoint; la pantalla exporta los tipos, los modelos de la lista, los colores y los tipos de equipamiento. Las marcas no se exportan por separado: la exportación de los modelos lleva su marca, y `GET /vehicle-brands` pedido solo como hoja de cálculo responde `406` | D-116, D-136 | Servicio, Frontend | `VehicleCatalogExportsIntegrationTests`, `VehicleCatalogOpenApiTests`, los `*.store.spec.ts` y `*-view.spec.ts` de las cuatro listas, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R26 | Marcas y modelos lista todos los modelos, cada uno con el nombre de su marca, y se filtra por marca, por tipo de vehículo y por estado; ordena por la marca y después por el modelo. Una marca o un tipo que no existe no coincide con ningún modelo. La marca del filtro queda en la dirección (`?marca=`), y un enlace a `/vehicle-catalog/brands-and-models/{id}` lleva a la lista filtrada por ella | D-136; §7.3 | Servicio, Frontend | `VehicleModelsListIntegrationTests`, `VehicleModelsControllerTests`, `VehicleModelQueryServiceImplTests`, `VehicleCatalogOpenApiTests`, `brands-and-models-view.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R27 | Las marcas se registran, se renombran, se activan y se desactivan en su lugar, en el diálogo «Gestionar marcas», que se busca y se pagina en el servidor; un formulario a la vez, y un diálogo nunca abre otro de formulario encima | D-136; DESIGN.md, revisiones 29 y 30 | Servicio, Frontend | `vehicle-brands-dialog.spec.ts`, `vehicle-brand-form.spec.ts`, `brands-and-models-view.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R28 | Lo que la habilitación necesita del catálogo para cada modelo se responde una sola vez y en lote: los equipos activos que aplican a su tipo, con su aplicabilidad activa, y cuál trae de fábrica: la pantalla, si el modelo la trae. El catálogo ya no calcula qué se exige: lo decide `fleet`, que lo cruza con su configuración (D-260, FLT-R148) | §7.5, §11.9, D-165, D-260 | Servicio | `VehicleCatalogContextFacadeTests`, `ModelRequirementsTests` (`fleet`), `vehicle-enablement-view.spec.ts` (`fleet`), `e2e/enablement.spec.ts` | ✅ |
| VCT-R29 | Un modelo es a gas si su combustible es `gnv` o `glp`; vacío no lo es | §7.3, D-166, D-109 | Servicio | `VehicleCatalogContextFacadeTests` | ✅ |
| VCT-R30 | El código de un equipamiento queda fijo desde su primera instalación, que `fleet` avisa por la fachada en su transacción y `vehiclecatalog` guarda en `has_installations`; `GPS` y `PANTALLA`, desde la semilla | D-74, D-185, §2.4, §7.5 | Base, Dominio, Servicio, Frontend | `EquipmentTypeTests`, `VehicleCatalogContextFacadeTests`, `EquipmentTypesControllerTests`, `VehicleEquipmentIntegrationTests` (`fleet`) | ✅ |
| VCT-R31 | La versión de lo que la habilitación nombra del catálogo —modelos, tipos, equipamiento y aplicabilidad— la da la fachada | D-128 | Servicio | `VehicleCatalogContextFacadeTests` | ✅ |
| VCT-R32 | La fachada lista, en una consulta por clase de dato, los modelos que una unidad nueva puede tomar —activos, de una marca activa y de un tipo de vehículo activo— y los colores activos. Es lo que la plantilla de las líneas de una factura nombra y lo que su importación resuelve, por nombre | D-108; decisión del responsable del 04/11/2026 | Servicio | `PurchaseInvoiceLinesIntegrationTests` (`procurement`; PRC-R155, PRC-R157 de [procurement.md](procurement.md)) | ✅ |
| VCT-R33 | Un modelo de equipamiento es de un tipo y de una marca de equipamiento, que no cambian. Su nombre, de 1 a 60 caracteres, se guarda recortado y en mayúsculas y no se repite en su tipo y su marca, sin distinguir mayúsculas ni los espacios de los bordes; otro tipo u otra marca pueden usarlo | §7.5, D-254 | Base: `uq_equipment_models_type_brand_name_lower` (V141) y `equipment_models_name_not_empty_ck` (V131). Dominio: `EquipmentModel`. Servicio: `409 EQUIPMENTMODEL_CONFLICT`, también en una carrera (VCT-R21). Recurso: la edición recibe solo el nombre. Frontend: el diálogo muestra el tipo y la marca de un modelo editado como texto fijo | `EquipmentModelTests.aNewEquipmentModelIsActiveAndNormalizedToUpperCase`, `blankOrExcessivelyLongNamesAreRejected` y `updateNormalizesAndReplacesName`; `EquipmentModelCommandServiceImplTests.createSucceedsWithNormalizedUppercaseName`, `createFailsWhenDuplicateNameUnderSameTypeAndBrand` y `updateFailsWhenNameExistsOnAnotherModel`; `EquipmentModelsControllerTests.createDuplicateReturns409Conflict`, `createSameNameUnderDifferentBrandSucceeds` y `updateModifiesName`; `equipment-model-form-dialog.spec`; e2e ("registers an equipment model under an active type and brand, refuses a taken name in its dialog, renames it and deactivates it") | ✅ |
| VCT-R34 | Un modelo de equipamiento nuevo es de un tipo activo, y uno inactivo se reactiva solo mientras su tipo está activo (`422 EQUIPMENTTYPE_INACTIVE`); desactivarlo no espera a nada. El diálogo de un modelo nuevo ofrece solo los tipos activos. Lo mismo vale para su marca (VCT-R37) | §7.5, D-254, D-84 | Servicio: `EquipmentModelCommandServiceImpl`, con el modelo leído con `FOR NO KEY UPDATE` al editarlo y al cambiar su estado. Frontend: `equipment-model-form-dialog` | `EquipmentModelCommandServiceImplTests.createFailsWhenEquipmentTypeNotFound`, `createFailsWhenEquipmentTypeInactive`, `changeStatusDeactivatesModel` y `changeStatusReactivationFailsIfParentTypeIsInactive`; `EquipmentModelsControllerTests.changeStatusDeactivatesAndReactivates`; e2e ("registers an equipment model under an active type and brand…") | ✅ |
| VCT-R36 | Una marca de equipamiento no repite nombre; el nombre, de 1 a 80 caracteres, se guarda recortado y en mayúsculas, y la unicidad se mide sobre lo guardado. Se registra, se renombra, se activa y se desactiva en su lugar, en «Gestionar marcas» de Catálogos › Equipamiento, que se busca y se pagina en el servidor, un formulario a la vez; no se borra | §7.5 | Base, Dominio, Servicio, Frontend | `EquipmentBrandTests`, `EquipmentBrandCommandServiceImplTests`, `EquipmentBrandsControllerTests`, `EquipmentBrandAuditTests`, `equipment-brands-dialog.spec.ts`, `equipment-brand-form.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R37 | Una marca de equipamiento inactiva no recibe modelos nuevos ni reactiva uno (`422 EQUIPMENTBRAND_INACTIVE`), y `fleet` no registra en el inventario un equipo de un modelo suyo; lo que ya la tiene la conserva. El diálogo de un modelo nuevo ofrece solo las marcas activas | §7.5, D-84 | Servicio, Frontend | `EquipmentModelCommandServiceImplTests`, `EquipmentCommandServiceImplTests` (`fleet`), `equipment-model-form-dialog.spec.ts` | ✅ |
| VCT-R35 | Un tipo de equipamiento dice si su equipo lleva IMEI y SIM (`requires_telematics`). Un tipo con código `GPS` siempre lo lleva; una edición que no manda el dato conserva lo guardado. Lo que exige al equipo del inventario —su serial, un IMEI de 15 dígitos y un ICCID de 19 o 20— lo aplica `fleet` (§11.4) | §7.5, D-254 | Base: `equipment_types.requires_telematics` (V132, que lo marca en el GPS). Dominio: `EquipmentType`, que lo fija para el código `GPS` al crear y al editar. Servicio: `EquipmentTypeCommandServiceImpl`, que conserva lo guardado cuando la edición no lo trae. Fachada: `EquipmentTypeReference.requiresTelematics`. Frontend: el formulario del tipo no lo muestra ni lo envía; la fachada lo entrega a `fleet` en `EquipmentTypeOption` | Ninguna prueba lo nombra: las pruebas de `EquipmentType` pasan el valor guardado; `equipment-catalog.store.spec` y `vehiclecatalog.facade.spec` lo leen | ⚠️ |

---

## 5. Relación con otros contextos

`vehiclecatalog` no llama a ningún otro contexto: sus exportaciones las pide al módulo `documents` (D-125), y la versión de sus datos sale de los contadores de `data_versions` (D-128). El mapa le permite usar `location` (mapa §4), y no lo usa. Las dos escrituras de la fachada, `markModelsInUse` y `markEquipmentTypeInUse`, van en el mismo sentido que las lecturas: `fleet` llama a `vehiclecatalog`, y `vehiclecatalog` no llama a `fleet` ni lee sus tablas (mapa §6.4). La única lectura de una tabla ajena es la de la migración que marcó, al crear la columna, los modelos que ya tenían unidades.

**La fachada `VehicleCatalogContextFacade`** (detalle en mapa §5.1). La usan `fleet` y `procurement`.

| Miembro | Para qué | Lo usa |
|---|---|---|
| `fetchVehicleModelSummaryByPublicId` | El modelo del alta de una unidad: si existe, si está activo y si lo están su marca y su tipo (VCT-R16) | `fleet`, `procurement` |
| `fetchColorSummaryByPublicId` | El color del alta y de la edición de una unidad, con si está activo | `fleet` |
| `fetchVehicleModelSummariesByIds`, `fetchColorSummariesByIds` | Los modelos y los colores de una lista de unidades, en una lectura, activos o no | `fleet` |
| `fetchVehicleModelSummariesByPublicIds`, `fetchColorSummariesByPublicIds` | Los modelos y los colores que nombra un pedido, como las unidades nuevas de una factura | `procurement` |
| `fetchActiveVehicleModelSummaries()`, `fetchActiveColorSummaries()` | Los modelos que una unidad nueva puede tomar y los colores activos, para la plantilla de las líneas de una factura y su importación (VCT-R32) | `procurement` |
| `fetchDataVersion()` | La versión de los tipos, las marcas, los modelos y los colores, para no reutilizar un documento generado cuando uno cambia (D-106, D-128) | `fleet`, `procurement` |
| `markModelsInUse(…)` | Marca que los modelos de las unidades que `fleet` registra ya tienen unidades, sea una sola o un lote, de una vez. Es idempotente y corre en la transacción de quien llama: si el alta falla, ningún modelo queda marcado (VCT-R14) | `fleet` |
| `fetchEnablementProfilesByModelIds(…)` | Lo que la habilitación necesita del catálogo para cada modelo de un lote: si es a gas y los equipos que le aplican, cada uno con si el modelo lo trae de fábrica (VCT-R28, VCT-R29). No dice qué se exige: lo decide `fleet` con su configuración (D-260). Un modelo cuyo tipo no existe queda fuera | `fleet` |
| `fetchEquipmentTypeReferenceByPublicId`, `fetchEquipmentTypeReferencesByIds` | El tipo de equipamiento que se instala o que tiene un equipo del inventario, activo o no, con si es telemático, y los de una lista | `fleet` |
| `fetchEquipmentTypeReferences()` | Todos los tipos de equipamiento, activos o no, por su código y en una lectura: la lista de Requisitos de equipamiento, que así no necesita la fachada del catálogo en el frontend (D-260) | `fleet` |
| `fetchEquipmentModelReferenceByPublicId`, `fetchEquipmentModelReferencesByIds` | El modelo de equipamiento de un equipo del inventario, activo o no, con su marca y si está activa, y los de una lista (D-254, VCT-R37) | `fleet` |
| `markEquipmentTypeInUse(…)` | Marca la primera instalación de un equipamiento, idempotente y en la transacción de quien llama; desde entonces su código queda fijo (VCT-R30) | `fleet` |
| `fetchEnablementDataVersion()` | La versión de lo que nombra la lista de habilitación de `fleet` (VCT-R31) | `fleet` |

**Frontend.** `VehicleCatalogFacade` ofrece a `fleet` vistas mínimas de los catálogos: los tipos de vehículo activos, las marcas, los colores, los modelos —los activos de una marca y todos, para el filtro de Logística › Habilitación— y los tipos, los modelos y las marcas de equipamiento: los tipos con si su equipo lleva serie y si es telemático, y las marcas activas o no, para que el inventario nombre la que conserva un equipo. Cada catálogo que ofrece tiene su propia carga y su propio error, apartados de los de las pantallas de catálogos: una lectura de `fleet` nunca deja su error ni sus filas en ellas.

---

## 6. Estado y pendientes

**Estado:** construido, con el paso abierto: faltan el catálogo maestro de modelos, validado por el negocio (§7.3, D-273), y la validación de este documento por el responsable. No se conocen diferencias con las convenciones de la API.

| Qué | Quién y cuándo |
|---|---|
| El catálogo maestro de modelos: los dos catálogos entregados no coinciden, así que el equipo prepara la lista de sus diferencias, el negocio elige el nombre de cada modelo y valida el catálogo, y solo ese se carga (§7.3, D-273) | Antes de la carga inicial; criterio de cierre del paso |
| Validar este documento | El responsable |
| Las pruebas que faltan: la carrera de un modelo y de una marca de equipamiento (VCT-R21) y una prueba propia de los dos miembros de la fachada de VCT-R32, que hoy prueba su consumidor | Sin fecha |

---

## 7. Preguntas abiertas

No quedan preguntas abiertas. Las respondió el responsable el 19/11/2026 (D-273): se carga un solo catálogo maestro de modelos, validado por el negocio antes de la carga inicial (sección 6), y los catálogos de flota no suman datos por ahora: el color en hexadecimal, el peso, los pasajeros y la garantía siguen candidatos hasta que una pantalla o un reporte los pida (diccionario §7.7).
