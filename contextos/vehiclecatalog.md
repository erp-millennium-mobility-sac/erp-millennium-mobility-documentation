# Contexto `vehiclecatalog` — ERP Millennium Mobility

**Estado:** borrador para validar · **Fuente:** diccionario de datos (§2.2, §2.4, §2.5, §7.1 a §7.7, §8.1 y las decisiones que cita cada regla), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (P3 y apartado 3.2) y el código. Si este documento y el diccionario no coinciden, manda el diccionario.

---

## 1. Qué es y qué no es

`vehiclecatalog` responde con qué se describe una unidad: de qué tipo es, de qué marca y de qué modelo y de qué color. El equipamiento que se le instala —sus tipos, sus marcas y sus modelos— es del contexto `equipment` desde la fase 2 del plan de migración ([equipment.md](equipment.md), D-278), y qué equipo exige la habilitación lo dice `fleet` (D-260). Sus catálogos son del sistema, no de cada empresa (D-15): un Chevrolet Sail es el mismo en Perú y en Colombia (VCT-R01). Los escribe quien tiene crear, editar o desactivar de su familia —con la semilla, solo el superadministrador— y los leen los siete roles del sistema (VCT-R02). No llama a ningún contexto; `fleet`, `procurement` y `equipment` lo usan por su fachada, y si un modelo ya tiene unidades se lo pregunta a `fleet` por un puerto suyo, `VehicleModelUsage` (VCT-R38).

**Las cuatro tablas que posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `vehicle_types` | Los tipos de vehículo —`AUTO`, `MOTO`, `TRIMOTO`—, con su nombre y el nombre comercial con que los conoce la operación | §7.1 |
| `vehicle_brands` | Las marcas del vehículo: CHEVROLET, TVS, BAJAJ, HYUNDAI | §7.2 |
| `vehicle_models` | Los modelos de cada marca, con su tipo, su cilindraje, su combustible y si traen pantalla de fábrica. Si ya tienen unidades lo responde `fleet` (VCT-R38), y la columna que lo guardaba ya no se escribe | §7.3, D-282 |
| `colors` | Los colores | §7.4 |

**Lo que parece suyo y no lo es**

| Parece de `vehiclecatalog` | Es de | Por qué |
|---|---|---|
| La marca comercial: Lizzo, Lima Bikes | `organization` (§6.3) | Es bajo qué nombre coloca Millennium la unidad, no quién la fabrica. Una moto TVS King vendida bajo Lima Bikes usa las dos tablas |
| Las unidades de un modelo | `fleet` (§8.1) | `vehiclecatalog` solo pregunta si un modelo tiene alguna, por su puerto `VehicleModelUsage`, que `fleet` implementa (VCT-R38, D-282). Nunca lee las tablas de `fleet` |
| Los estados del vehículo, los motivos y los tipos de documento | `fleet` (mapa §6.4) | Son de los bloques 6 y 9 |
| Los tipos, las marcas y los modelos de equipamiento, a qué tipos de vehículo aplica cada tipo, y el equipo físico en el almacén de una empresa | `equipment` (§7.5, §11.4; D-254, D-278) | Salieron de este catálogo en la fase 2 del plan de migración. `equipment` lee de aquí, por la fachada, los tipos de vehículo y, de cada modelo, su tipo y su pantalla de fábrica |
| El equipamiento instalado en una unidad | `fleet` (§11.4) | La instalación apunta al tipo de equipamiento de `equipment` |
| Qué equipo exige una unidad, también si requiere instalar pantalla | `fleet` (§11.4, D-260) | `vehiclecatalog` responde de cada modelo, en lote, su tipo, si es a gas y si trae la pantalla de fábrica (VCT-R17, VCT-R29); `equipment` dice qué equipo aplica a cada modelo (VCT-R28, en [equipment.md](equipment.md)); `fleet` lo cruza con su configuración de lo que se exige y de lo que es de seguridad |
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
| Pantalla de fábrica | Que el modelo ya trae la pantalla instalada | Que el equipo pantalla aplique a su tipo, que lo dice `equipment` (§7.5) |
| Modelo con unidades | Un modelo del que `fleet` ya registró al menos una unidad, aunque esté anulada. Desde entonces su tipo de vehículo queda fijo (D-282) | Un modelo activo, que solo dice si se sigue comprando |
| Desactivar | Dejar de ofrecer un registro para lo nuevo; lo que ya lo usa lo conserva | Borrar, que no existe |

Las palabras del equipamiento —marca y modelo de equipamiento, telemático, aplicabilidad, el código que lee el programa y el equipamiento con instalaciones— están en [equipment.md](equipment.md).

---

## 2. Ciclo de vida de cada agregado

Nada de `vehiclecatalog` se borra: ni los tipos, ni las marcas, ni los modelos, ni los colores (VCT-R03). Registran cuándo se crearon y cuándo cambiaron, sin autor (VCT-R04). Los códigos y los nombres se guardan recortados y en mayúsculas (VCT-R05).

### Tipo de vehículo

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador, en Catálogos › Tipos de vehículo, con su código, su nombre y su nombre comercial. La semilla trae `AUTO`, `MOTO` y `TRIMOTO`, que son los códigos que interpreta el programa (VCT-R22) |
| Qué cambia | El nombre y el nombre comercial |
| Qué queda fijo | El código, desde el alta: lo leen las reglas del sistema (VCT-R06) |
| Retiro | Se desactiva y se puede reactivar. Un tipo inactivo no se elige para un modelo nuevo ni para cambiar el de uno existente (VCT-R15), y no se registra una unidad de un modelo de ese tipo (VCT-R16). Los modelos que ya lo tienen lo conservan, y los equipamientos de `equipment` que aplican a él siguen aplicando: sus unidades pueden seguir necesitándolos (§7.5) |
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

El tipo, la marca y el modelo de equipamiento, con sus ciclos de vida, están en [equipment.md](equipment.md).

---

## 3. Flujos funcionales

Las lecturas piden `catalogos.ver` y las escrituras, crear, editar o desactivar de su familia; con la semilla escribe solo el superadministrador y los demás roles leen, aquí y en los selectores de las pantallas que usan los catálogos, como el alta de una unidad (VCT-R02). Las pantallas son tres, en el grupo Catálogos: Tipos de vehículo, Marcas y modelos y Colores; Equipamiento es de `equipment` ([equipment.md](equipment.md), F1). Cada una se abre solo con `catalogos.ver` (D-141), y cada acción se ofrece solo con su permiso. Las listas se buscan sin distinguir mayúsculas ni tildes, se filtran por estado, se ordenan y se paginan en el servidor (VCT-R24). Desactivar un registro pregunta antes (VCT-R23).

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

### F6 · Exportar un catálogo

**Rol:** quien lee los catálogos, con `catalogos.ver`. **Pantalla:** las tres listas (D-116).

1. Busca y filtra la lista y la exporta a Excel con la misma búsqueda, los mismos filtros y el mismo orden, por la plataforma de documentos (VCT-R25; convenciones §8).
2. Marcas y modelos exporta los modelos de la lista, cada uno con su marca; las marcas no se exportan por separado (D-136). Los tipos de equipamiento los exporta `equipment` ([equipment.md](equipment.md), F2).

Los archivos llevan códigos y nombres, nunca un identificador (D-51); sus columnas están en convenciones §8.1.

**Si algo falla:** la pantalla dice que no pudo descargar el archivo.

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). Las pruebas del backend están en `src/test/java/.../vehiclecatalog`; las del frontend, junto a cada archivo de `src/app/vehiclecatalog`, y las de extremo a extremo en `e2e/vehicle-catalog.spec.ts`.

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| VCT-R01 | Los catálogos de flota son del sistema: no llevan empresa ni Row Level Security. El de equipos también, en `equipment` (EQP-R01) | §7, D-15 | Base | `VehicleCatalogMigrationTests` | ✅ |
| VCT-R02 | Los lee quien tiene `catalogos.ver` (los siete roles) y los escribe quien tiene crear, editar o desactivar de esa familia (en la semilla, solo el superadministrador, D-130); el permiso de los catálogos generales no escribe estos | §5.6, D-95, D-130, D-141 | Servicio, Frontend | `VehicleCatalogPermissionIntegrationTests`, los `*ControllerTests`, `VehicleModelsListIntegrationTests`, `brands-and-models-view.spec.ts`, `vehicle-brands-dialog.spec.ts`, `app-shell.spec.ts`, `e2e/shell.spec.ts` | ✅ |
| VCT-R03 | Nada se borra: un registro se desactiva y se reactiva | §2.5, D-73 | Base, Servicio | Los `*AuditTests`, `VehicleTypesControllerTests`, los `*ControllerTests` | ✅ |
| VCT-R04 | Registran cuándo se crearon y cambiaron, sin autor | D-42, D-73 | Base, Servicio | `ColorAuditTests`, `VehicleBrandAuditTests`, `VehicleModelAuditTests`, `VehicleTypeAuditTests` | ✅ |
| VCT-R05 | Los códigos y los nombres de tipos, marcas, modelos y colores se guardan recortados y en mayúsculas, y la unicidad se mide sobre lo guardado | §2.2 | Base, Dominio, Frontend | `ColorCommandServiceImplTests`, `VehicleBrandCommandServiceImplTests`, `VehicleTypeCommandServiceImplTests`, `color-form-dialog.spec.ts` | ✅ |
| VCT-R06 | El código de un tipo de vehículo no se repite y no cambia | §7.1 | Base, Dominio, Frontend | `VehicleTypeTests`, `VehicleTypesControllerTests`, `vehicle-type-form-dialog.spec.ts` | ✅ |
| VCT-R07 | Una marca no repite nombre | §7.2 | Base, Servicio | `VehicleBrandsControllerTests`, `brands-and-models.store.spec.ts`, `vehicle-brands-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R08 | Una marca no tiene dos modelos con el mismo nombre; otra marca puede usarlo | §7.3 | Base, Servicio | `VehicleModelConstraintTests`, `VehicleModelsControllerTests` | ✅ |
| VCT-R09 | El cilindraje es un entero mayor que cero, o vacío; no tiene máximo salvo el que cabe en su columna, y un decimal se rechaza en vez de cortarse | §7.3 | Base, Servicio, Frontend | `VehicleModelConstraintTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts` | ✅ |
| VCT-R10 | Un modelo eléctrico no lleva cilindraje | §7.3 | Base, Dominio, Frontend | `VehicleModelConstraintTests`, `VehicleModelTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts` | ✅ |
| VCT-R11 | El combustible es uno de la lista cerrada, guardado en minúsculas y sin tildes, o vacío | §7.3 | Base, Dominio, Servicio | `VehicleModelConstraintTests`, `VehicleModelCommandServiceImplTests`, `FuelTypeTests` | ✅ |
| VCT-R12 | El combustible de un modelo nuevo no se propone: nace vacío y se elige | D-109 | Servicio, Frontend | `vehicle-model-form-dialog.spec.ts`, `vehicle-model-assembler.spec.ts`, `VehicleModelCommandServiceImplTests`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R13 | La marca de un modelo no cambia | §7.3 | Dominio, Servicio, Frontend | `VehicleModelCommandServiceImplTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts` | ✅ |
| VCT-R14 | El tipo de vehículo de un modelo se corrige mientras el modelo no tenga unidades; desde la primera queda fijo. Si las tiene se lo pregunta a `fleet` (VCT-R38): la corrección bloquea el modelo `FOR UPDATE` antes de preguntar, así una unidad en curso la hace esperar, y una que llega durante la corrección la espera; solo pregunta si el tipo cambia. La lista calcula `vehicleTypeLocked` con una pregunta por página; la fachada no lo calcula ni pregunta | D-282, §7.3 | Base, Dominio, Servicio, Frontend | `VehicleModelTests`, `VehicleModelCommandServiceImplTests`, `VehicleModelQueryServiceImplTests`, `VehicleModelsControllerTests`, `VehicleModelTypeLockTests`, `VehicleRetiredReferencesTests` (`fleet`), `vehicle-model-form-dialog.spec.ts`, `brands-and-models.store.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R15 | Un modelo nuevo es de una marca activa y de un tipo activo; al editarlo, un tipo nuevo tiene que estar activo, y el inactivo que ya tenía se conserva. El diálogo de un modelo nuevo ofrece solo las marcas activas | D-108, D-84 | Servicio, Frontend | `VehicleModelCommandServiceImplTests`, `VehicleModelsControllerTests`, `vehicle-model-form-dialog.spec.ts`, `brands-and-models.store.spec.ts`, `brands-and-models-view.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R16 | Una unidad nueva no se registra con un modelo inactivo, ni con uno cuya marca o tipo estén inactivos | D-108, §8.1 | Servicio, en `fleet` (FLT-R07), con los estados que da la fachada | `VehicleCatalogContextFacadeTests`, `VehicleRetiredReferencesTests` (`fleet`) | ✅ |
| VCT-R17 | Que un modelo traiga la pantalla de fábrica sale de su dato, nunca de comparar textos, y la fachada lo responde con su tipo de vehículo. Si el equipo pantalla aplica a ese tipo lo dice `equipment` (VCT-R28), y si se exige, `fleet` con su configuración (D-260) | §7.3, §7.5, D-165 | Base, Servicio | `VehicleModelTests`, `VehicleCatalogContextFacadeTests` | ✅ |
| VCT-R21 | Un duplicado que la base detiene en una carrera responde el `*_CONFLICT` de su recurso, nunca un `500` | Convenciones §7.2 | Base, Servicio | `VehicleCatalogRacedDuplicatesTests`, `VehicleModelCommandServiceImplTests` | ✅ |
| VCT-R22 | La semilla trae los tres tipos con sus dos nombres; el GPS y la pantalla los trae la semilla del catálogo de equipos (EQP-R06) | §7.1 | Base | `VehicleCatalogMigrationTests` | ✅ |
| VCT-R23 | Desactivar un registro pregunta antes, con su nombre | DESIGN.md; como Estados y motivos (FLT-R25) | Frontend | `deactivation-question.spec.ts`, `vehicle-types-view.spec.ts`, `colors-view.spec.ts`, `brands-and-models-view.spec.ts`, `vehicle-brands-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R24 | Las listas se buscan, se filtran, se ordenan y se paginan en el servidor; la búsqueda no distingue mayúsculas ni tildes; un filtro por un tipo de vehículo que no existe no coincide con ninguna fila | Convenciones §6.2, D-142 | Servicio, Frontend | `VehicleTypesControllerTests`, `ColorsControllerTests`, `VehicleModelQueryServiceImplTests`, `VehicleModelsListIntegrationTests`, `vehicle-brands-dialog.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R25 | Los tipos, los colores, los modelos de una marca y los de todas se exportan a Excel desde su propio endpoint; la pantalla exporta los tipos, los modelos de la lista y los colores. Los tipos de equipamiento los exporta `equipment` (EQP-R09). Las marcas no se exportan por separado: la exportación de los modelos lleva su marca, y `GET /vehicle-brands` pedido solo como hoja de cálculo responde `406` | D-116, D-136 | Servicio, Frontend | `VehicleCatalogExportsIntegrationTests`, `VehicleCatalogOpenApiTests`, los `*.store.spec.ts` y `*-view.spec.ts` de las tres listas, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R26 | Marcas y modelos lista todos los modelos, cada uno con el nombre de su marca, y se filtra por marca, por tipo de vehículo y por estado; ordena por la marca y después por el modelo. Una marca o un tipo que no existe no coincide con ningún modelo. La marca del filtro queda en la dirección (`?marca=`), y un enlace a `/vehicle-catalog/brands-and-models/{id}` lleva a la lista filtrada por ella | D-136; §7.3 | Servicio, Frontend | `VehicleModelsListIntegrationTests`, `VehicleModelsControllerTests`, `VehicleModelQueryServiceImplTests`, `VehicleCatalogOpenApiTests`, `brands-and-models-view.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R27 | Las marcas se registran, se renombran, se activan y se desactivan en su lugar, en el diálogo «Gestionar marcas», que se busca y se pagina en el servidor; un formulario a la vez, y un diálogo nunca abre otro de formulario encima | D-136; DESIGN.md, revisiones 29 y 30 | Servicio, Frontend | `vehicle-brands-dialog.spec.ts`, `vehicle-brand-form.spec.ts`, `brands-and-models-view.spec.ts`, `e2e/vehicle-catalog.spec.ts` | ✅ |
| VCT-R29 | Un modelo es a gas si su combustible es `gnv` o `glp`; vacío no lo es. La fachada lo responde, en lote, con el tipo de vehículo y la pantalla de fábrica del modelo | §7.3, D-166, D-109 | Servicio | `VehicleCatalogContextFacadeTests` | ✅ |
| VCT-R31 | La versión de lo que la habilitación nombra del catálogo —los modelos y sus tipos de vehículo— la da la fachada; la del equipamiento y su aplicabilidad, `equipment` (EQP-R10) | D-128 | Servicio | `VehicleCatalogContextFacadeTests` | ✅ |
| VCT-R32 | La fachada lista, en una consulta por clase de dato, los modelos que una unidad nueva puede tomar —activos, de una marca activa y de un tipo de vehículo activo— y los colores activos. Es lo que la plantilla de las líneas de una factura nombra y lo que su importación resuelve, por nombre | D-108; decisión del responsable del 04/11/2026 | Servicio | `PurchaseInvoiceLinesIntegrationTests` (`procurement`; PRC-R155, PRC-R157 de [procurement.md](procurement.md)) | ✅ |
| VCT-R38 | Si un modelo ya tiene unidades lo responde `fleet` por el puerto `VehicleModelUsage`, por lote: cuenta toda unidad, de cualquier empresa y también anulada. `vehiclecatalog` pregunta con alcance `SYSTEM`, solo salen identificadores, y el puerto tiene un solo bean, obligatorio. Preguntan solo las pantallas del catálogo —sus listas y su exportación, la lectura de un modelo y la respuesta de sus comandos—; la fachada nunca, porque la leen otros contextos dentro de sus transacciones, y entrar ahí al alcance `SYSTEM` escribiría y vaciaría lo que leyeron: responde el modelo con su marca y su tipo (`VehicleModelReference`) | D-282 | Servicio | `VehicleModelUsageAdapterTests` (`fleet`), `VehicleModelUsageWiringTests` (`fleet`), `FleetContextFacadeImplTests` (`fleet`: una sola elevación por llamada) | ✅ |

VCT-R18 a VCT-R20, VCT-R28, VCT-R30 y VCT-R33 a VCT-R37 son del catálogo de equipos: están en [equipment.md](equipment.md), con su número, desde la fase 2 del plan de migración (D-278).

---

## 5. Relación con otros contextos

`vehiclecatalog` no llama a ningún otro contexto: sus exportaciones las pide al módulo `documents` (D-125), y la versión de sus datos sale de los contadores de `data_versions` (D-128). El mapa le permite usar `location` (mapa §4), y no lo usa. Si un modelo ya tiene unidades lo pregunta por `VehicleModelUsage`, un puerto que declara en su `acl` y que implementa `fleet`, como `location` con `CurrencyUsage`: `vehiclecatalog` no llama a `fleet` ni lee sus tablas, y `fleet` ya dependía de él (mapa §6.4, D-282). Tampoco llama a `equipment`, que lo lee por la fachada. La única lectura de una tabla ajena es la de la migración que marcó, al crear la columna, los modelos que ya tenían unidades; desde la fase 8 del plan esa columna ya no se escribe, y sale en la entrega siguiente.

**La fachada `VehicleCatalogContextFacade`** (detalle en mapa §5.1). La usan `fleet`, `procurement` y `equipment`.

| Miembro | Para qué | Lo usa |
|---|---|---|
| `fetchVehicleModelSummaryByPublicId` | El modelo del alta de una unidad: si existe, si está activo y si lo están su marca y su tipo (VCT-R16) | `fleet`, `procurement` |
| `fetchColorSummaryByPublicId` | El color del alta y de la edición de una unidad, con si está activo | `fleet` |
| `fetchVehicleModelSummariesByIds`, `fetchColorSummariesByIds` | Los modelos y los colores de una lista de unidades, en una lectura, activos o no | `fleet` |
| `fetchVehicleModelSummariesByPublicIds`, `fetchColorSummariesByPublicIds` | Los modelos y los colores que nombra un pedido, como las unidades nuevas de una factura | `procurement` |
| `fetchActiveVehicleModelSummaries()`, `fetchActiveColorSummaries()` | Los modelos que una unidad nueva puede tomar y los colores activos, para la plantilla de las líneas de una factura y su importación (VCT-R32) | `procurement` |
| `fetchDataVersion()` | La versión de los tipos, las marcas, los modelos y los colores, para no reutilizar un documento generado cuando uno cambia (D-106, D-128) | `fleet`, `procurement`, `equipment` |
| `fetchVehicleTypeSummariesByPublicIds`, `fetchVehicleTypeSummariesByIds` | Los tipos de vehículo que nombra un pedido o una lista, en una lectura, activos o no, con su código y su nombre: los tipos a los que aplica un tipo de equipamiento | `equipment` |
| `fetchModelEnablementFactsByIds(…)` | Lo que la habilitación necesita de este catálogo para cada modelo de un lote: su tipo de vehículo, si es a gas y si trae la pantalla de fábrica (VCT-R17, VCT-R29). Qué equipo aplica a ese tipo lo dice `equipment`, y qué se exige, `fleet` con su configuración (D-260). Un modelo cuyo tipo no existe queda fuera | `fleet` (si es a gas), `equipment` (el tipo y la pantalla de fábrica) |
| `fetchEnablementDataVersion()` | La versión de los modelos y sus tipos que nombra la lista de habilitación de `fleet` (VCT-R31) | `fleet` |

La fachada ya no recibe que un modelo tiene unidades, y tampoco lo pregunta: lee los modelos como referencias —el modelo con su marca y su tipo (`VehicleModelReference`), sin bloqueo— y responde sin entrar al alcance `SYSTEM` dentro de la transacción de quien la llama (D-282).

**El puerto que declara: `VehicleModelUsage`**, en `vehiclecatalog::acl`, que implementa `fleet` (`VehicleModelUsageAdapter`): cuáles de un lote de modelos tienen al menos una unidad, de cualquier empresa y también anulada (`modelIdsWithUnits`). `vehiclecatalog` inyecta un solo bean, nunca una lista: sin él la aplicación no arranca, y `VehicleModelUsageWiringTests` lo vigila. Pregunta con alcance `SYSTEM`, porque las unidades tienen Row Level Security, y solo desde sus pantallas: una vez por página para `vehicleTypeLocked` —en la lista, en la exportación, en la lectura de un modelo y en la respuesta de sus comandos—, y al corregir un modelo solo si su tipo cambia, con el modelo ya bloqueado `FOR UPDATE` (VCT-R14, VCT-R38).

**Frontend.** `VehicleCatalogFacade` ofrece vistas mínimas de los catálogos: a `fleet`, los tipos de vehículo activos, las marcas, los colores y los modelos —los activos de una marca y todos, para el filtro de Logística › Habilitación—; y a `equipment`, todos los tipos de vehículo, activos o no, con su propia carga, porque el diálogo de un tipo de equipamiento ofrece también los inactivos que ya tiene. Cada catálogo que ofrece tiene su propia carga y su propio error, apartados de los de las pantallas de catálogos: una lectura de otro contexto nunca deja su error ni sus filas en ellas.

---

## 6. Estado y pendientes

**Estado:** construido, con el paso abierto: faltan el catálogo maestro de modelos, validado por el negocio (§7.3, D-273), y la validación de este documento por el responsable. No se conocen diferencias con las convenciones de la API.

| Qué | Quién y cuándo |
|---|---|
| El catálogo maestro de modelos: los dos catálogos entregados no coinciden, así que el equipo prepara la lista de sus diferencias, el negocio elige el nombre de cada modelo y valida el catálogo, y solo ese se carga (§7.3, D-273) | Antes de la carga inicial; criterio de cierre del paso |
| Validar este documento | El responsable |
| Una prueba propia de los dos miembros de la fachada de VCT-R32, que hoy prueba su consumidor | Sin fecha |
| Retirar `vehicle_models.has_units`, que ya nadie escribe ni lee desde que `vehiclecatalog` pregunta por `VehicleModelUsage` (D-282) | La entrega N+1 de la fase 8 del plan, cuando la entrega N esté desplegada en `dev` con su CI en verde |

---

## 7. Preguntas abiertas

No quedan preguntas abiertas. Las respondió el responsable el 19/11/2026 (D-273): se carga un solo catálogo maestro de modelos, validado por el negocio antes de la carga inicial (sección 6), y los catálogos de flota no suman datos por ahora: el color en hexadecimal, el peso, los pasajeros y la garantía siguen candidatos hasta que una pantalla o un reporte los pida (diccionario §7.7).
