# Contexto `vehiclecatalog` — ERP Millennium Mobility

**Actualizado:** 20/10/2026 (lo que construyó la auditoría, verificado contra el código integrado: backend `audit/be-integration`, merge `81bb8c1`, con `040ad1e` y `f11e4f6`; frontend `audit/fe-integration`, merge `36210f6`, con `139e513` a `100485d`; la segunda fase, la exportación, D-116, con `be7bdb9` en el backend y `f73a656` en el frontend. Falta llevarlo a `develop`) · 20/10/2026 (primera versión: el paso 4 tal como está en `develop`, más lo que decidió la auditoría del 20/10/2026 —D-107, D-108, D-109, D-112 y D-116—) · **Estado:** borrador para validar; el paso 4 sigue abierto · **Fuente:** diccionario de datos v1.24 (§2.2, §2.4, §2.5, §7.1 a §7.7, §8.1 y las decisiones D-15, D-17, D-42, D-48, D-73, D-74, D-84 y D-106 a D-116), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (P3 y apartado 3.2), [orden de implementación](../orden-de-implementacion-millennium-mobility.md) (paso 4), el plan de la auditoría del 20/10/2026 y el código de `develop` al 20/10/2026: backend `32d4996` y frontend `23d07ad`, con la carpeta del contexto renombrada `src/app/vehiclecatalog` en `8d4fd27` (rama `audit/fe-base`). Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** Las secciones 1 a 3 están escritas para quien conoce el negocio. La 4 y la 6 nombran archivos del código para el equipo. Lo que aprobó el responsable el 20/10/2026 con la auditoría está construido e integrado en las ramas de integración de la auditoría, y se describe como está, también su segunda fase, la exportación.

---

## 1. Qué es y qué no es

`vehiclecatalog` responde con qué se describe una unidad: de qué tipo es, de qué marca y de qué modelo, de qué color, y qué equipamiento se le instala. Sus catálogos son del sistema, no de cada empresa (D-15): un Chevrolet Sail es el mismo en Perú y en Colombia, así que sus tablas no llevan `company_id` ni Row Level Security. Los escribe solo el superadministrador, con `catalogos.gestionar_flota`, y los leen los siete roles del sistema, con `catalogos.ver` (§5.6). Es el paso 4 del orden de implementación. No llama a ningún contexto; `fleet` lo usa por su fachada.

**Las seis tablas que posee** (diccionario §7, migración V35, más `has_units` en V49)

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `vehicle_types` | Los tipos de vehículo —`AUTO`, `MOTO`, `TRIMOTO`—, con su nombre y el nombre comercial con que los conoce la operación | §7.1 |
| `vehicle_brands` | Las marcas del vehículo: CHEVROLET, TVS, BAJAJ, HYUNDAI | §7.2 |
| `vehicle_models` | Los modelos de cada marca, con su tipo, su cilindraje, su combustible, si traen pantalla de fábrica y si ya tienen unidades | §7.3, D-107 |
| `colors` | Los colores | §7.4 |
| `equipment_types` | Lo que se instala en una unidad: `GPS`, `PANTALLA` | §7.5 |
| `equipment_type_vehicle_types` | A qué tipos de vehículo aplica cada equipamiento | §7.5, D-73 |

Las columnas están en el diccionario y no se repiten aquí.

**Lo que parece suyo y no lo es**

| Parece de `vehiclecatalog` | Es de | Por qué |
|---|---|---|
| La marca comercial: Lizzo, Lima Bikes | `organization` (§6.3) | Es bajo qué nombre coloca Millennium la unidad, no quién la fabrica. Una moto TVS King vendida bajo Lima Bikes usa las dos tablas |
| Las unidades de un modelo | `fleet` (§8.1) | `vehiclecatalog` solo sabe si un modelo tiene alguna (`has_units`), porque `fleet` se lo avisa (D-107). Nunca lee las tablas de `fleet` |
| Los estados del vehículo, los motivos y los tipos de documento | `fleet` (mapa §6.4) | Son de los bloques 6 y 9, no del 4 |
| El equipamiento instalado en una unidad | `fleet`, en el paso 9 (§11.4) | Aquí está solo el tipo de equipamiento |
| Si una unidad requiere instalar pantalla | La regla combina dos datos de aquí (§7.5) | La responderá un miembro de la fachada, con el paso 9 (mapa §5.1) |
| La placa | `fleet` (§7.6) | No tiene tabla: es una columna de la unidad |
| Las tarifas por kilómetro de un modelo | Ventas, fuera del alcance (§7.3) | Le apuntarán al modelo cuando lleguen |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Tipo de vehículo | Auto, moto o trimoto. Clasifica la unidad y activa las reglas que dependen del tipo (§7.1) | El "tipo de vehículo" del Registro de Clientes, `NUEVO / SEMIPREMIUM`, que se deduce de si la unidad tuvo contratos |
| Nombre comercial | Cómo conoce la operación un tipo: Leasing Car, Leasing Bike, Leasing Tuk Tuk | El nombre del tipo, que puede no coincidir algún día |
| Marca | Quien fabrica el vehículo | La marca comercial (§6.3) |
| Modelo | La denominación oficial dentro de una marca: `SAIL SEDAN 1.5L LT MT` de CHEVROLET. El nombre solo identifica junto a la marca | — |
| Cilindraje | La cifra comercial del modelo en cc, entera: 125, 200, 1500 | La cifra técnica de la ficha (199,26) |
| Pantalla de fábrica | Que el modelo ya trae la pantalla instalada | Que el equipo pantalla aplique a su tipo (§7.5) |
| Aplicabilidad | Que un equipamiento tenga sentido en un tipo de vehículo | Una instalación, que es de `fleet` |
| Código que lee el programa | `GPS` y `PANTALLA`: las reglas de habilitación los reconocen por su código, así que quedan fijos desde la semilla (D-74) | Un código propio, que se corrige mientras nada lo use |
| Modelo con unidades | Un modelo del que `fleet` ya registró al menos una unidad (`has_units`). Desde entonces su tipo de vehículo queda fijo (D-107) | Un modelo activo, que solo dice si se sigue comprando |
| Desactivar | Dejar de ofrecer un registro para lo nuevo; lo que ya lo usa lo conserva | Borrar, que no existe |

---

## 2. Ciclo de vida de cada agregado

Nada de `vehiclecatalog` se borra: ni los tipos, ni las marcas, ni los modelos, ni los colores, ni los equipamientos, ni sus aplicabilidades. La API no tiene `DELETE` (convenciones §3.1), y la cuenta de la aplicación lee, inserta y actualiza estas tablas, pero no borra (V35). Registran cuándo se crearon y cuándo cambiaron, sin autor (D-42, D-73).

### Tipo de vehículo

| Momento | Qué pasa |
|---|---|
| Alta | La hace el superadministrador en Catálogos › Tipos de vehículo, con su código, su nombre y su nombre comercial. V35 siembra `AUTO`, `MOTO` y `TRIMOTO`, que son los códigos que interpreta el programa |
| Qué cambia | El nombre y el nombre comercial |
| Qué queda fijo | El código, desde el alta: lo leen las reglas del sistema (§7.1) |
| Retiro | Se desactiva y se puede reactivar. Un tipo inactivo no se elige para un modelo nuevo ni para cambiar el de uno existente, y no se registra una unidad de un modelo de ese tipo (D-108). Los modelos que ya lo tienen lo conservan, y los equipamientos que aplican a él siguen aplicando: sus unidades pueden seguir necesitándolos (§7.5) |
| Nunca | Se borra; repite código |

### Marca

| Momento | Qué pasa |
|---|---|
| Alta | La hace el superadministrador en Catálogos › Marcas y modelos, con su nombre, que se guarda recortado y en mayúsculas: `Tvs` y `TVS` son la misma marca |
| Qué cambia | El nombre |
| Retiro | Se desactiva y se reactiva desde su ficha o desde su fila, con una confirmación al desactivarla. Una marca inactiva no recibe modelos nuevos, y no se registra una unidad de un modelo suyo (D-108). Sus modelos y sus unidades la siguen referenciando y se siguen editando |
| Nunca | Se borra; repite nombre |

### Modelo

| Momento | Qué pasa |
|---|---|
| Alta | Lo hace el superadministrador desde la ficha de su marca, con su nombre, su tipo de vehículo, su cilindraje, su combustible y si trae pantalla de fábrica. La marca y el tipo tienen que estar activos (D-108). El combustible nace vacío y se elige (D-109). Nace sin unidades |
| Qué cambia | El nombre, el cilindraje, el combustible, la pantalla de fábrica y, mientras el modelo no tenga unidades, el tipo de vehículo (D-107). Un tipo nuevo tiene que estar activo; el inactivo que el modelo ya tenía se conserva |
| Qué queda fijo | La marca, desde el alta. El tipo de vehículo, desde la primera unidad que `fleet` registra con el modelo (D-107): cambiarlo contradiría el tipo de las unidades que ya existen |
| Retiro | Se desactiva y se reactiva. Las unidades que ya lo tienen lo conservan; ninguna nueva lo elige |
| Nunca | Se borra; repite nombre dentro de su marca; lleva cilindraje si es eléctrico; vuelve a no tener unidades |

### Color

| Momento | Qué pasa |
|---|---|
| Alta | La hace el superadministrador en Catálogos › Colores, con su nombre, recortado y en mayúsculas |
| Qué cambia | El nombre |
| Retiro | Se desactiva y se reactiva. Una unidad nueva no elige un color inactivo, y la que ya lo tiene lo conserva (`fleet`, FLT-R07) |
| Nunca | Se borra; repite nombre |

### Tipo de equipamiento

| Momento | Qué pasa |
|---|---|
| Alta | La hace el superadministrador en Catálogos › Equipamiento, con su código, su nombre, si bloquea la disponibilidad, si requiere número de serie y los tipos de vehículo a los que aplica, al menos uno. V35 siembra `GPS` (bloquea, con serie, aplica a los tres tipos) y `PANTALLA` (bloquea, sin serie, solo autos) |
| Qué cambia | El nombre, las dos banderas, los tipos a los que aplica y el código, salvo el de `GPS` y el de `PANTALLA`, que lee el programa (D-74). El código de otro equipamiento quedará fijo desde su primera instalación, cuando existan, en el paso 9 |
| Retiro | Se desactiva y se reactiva. Quitarle un tipo de vehículo desactiva esa aplicabilidad, y volver a marcarlo reactiva la misma; solo cuenta la activa (D-73). Para que deje de instalarse en todo se desactiva el equipamiento, no se le quitan todos los tipos |
| Nunca | Se borra; queda sin ningún tipo de vehículo; repite código |

---

## 3. Flujos funcionales

En cada flujo, "el sistema revisa" es lo que el backend comprueba. Toda ruta exige un permiso: las lecturas, `catalogos.ver`; las escrituras, `catalogos.gestionar_flota` (§5.6). Sin sesión responde `401`, y con un rol que no trae el permiso, `403 ACCESS_DENIED`.

Las pantallas son cinco, en el grupo Catálogos del menú: Tipos de vehículo (`/vehicle-catalog/vehicle-types`), Marcas y modelos (`/vehicle-catalog/brands-and-models`) con la ficha de cada marca (`/vehicle-catalog/brands-and-models/{id}`), Colores (`/vehicle-catalog/colors`) y Equipamiento (`/vehicle-catalog/equipment-types`). Las abre solo el superadministrador. Los demás roles no las ven: leen los catálogos desde los selectores de las pantallas que los usan, como el alta de una unidad en Inventario.

Las cuatro listas se buscan, se filtran por estado ("Todos", "Solo activos" o "Solo inactivos"), se ordenan y se paginan en el servidor (convenciones §6.2). Una lista sin resultados lo dice con su propio texto: "Sin tipos de vehículo", "Sin marcas registradas", "Sin colores registrados", "Sin tipos de equipamiento".

### F1 · Mantener los tipos de vehículo

**Rol:** superadministrador. **Pantalla:** Catálogos › Tipos de vehículo, lista con diálogo.

1. **Buscar.** Escribe parte del código, del nombre o del nombre comercial. La lista muestra código, nombre, nombre comercial y estado.
2. **Nuevo tipo de vehículo.** Escribe el código ("Letras, números, guiones y espacios."; se guarda en mayúsculas), el nombre formal y el nombre comercial.
3. **Editar.** El código se muestra bloqueado: no cambia después del alta.
4. **Desactivar o activar** desde el menú de la fila. Desactivar pregunta antes, con el nombre y el código del tipo: "¿Desactivar el tipo de vehículo?", "El tipo {nombre} ({código}) ya no podrá elegirse en modelos nuevos. Los modelos, el equipamiento y las unidades que ya lo tienen lo conservan.", con "Desactivar" como acción destructiva.

El sistema revisa la forma de cada campo (`400 VALIDATION_ERROR`) y que el código no se repita, una vez en mayúsculas (`409 VEHICLETYPE_CONFLICT`), también si dos altas llegan a la vez: la base detiene la segunda y el adaptador la responde con el mismo código.

**Errores que ve:** "Ya existe un tipo de vehículo con ese código." (`VEHICLETYPE_CONFLICT`); "No se pudo guardar el tipo de vehículo." para cualquier otro código.
**Resultado:** "Tipo de vehículo registrado exitosamente.", "Tipo de vehículo actualizado exitosamente.", "Tipo de vehículo desactivado exitosamente."

### F2 · Registrar y buscar las marcas

**Rol:** superadministrador. **Pantalla:** Catálogos › Marcas y modelos: la lista de las marcas oficiales, y la ficha de cada una con sus modelos (D-112). No es un maestro-detalle: es la lista con ficha de Empresas (DESIGN.md, revisión 13).

1. **Buscar.** Escribe parte del nombre ("Buscar marcas..."). El nombre de cada marca enlaza a su ficha; también "Ver ficha de marca", en el menú de la fila.
2. **Nueva marca.** Escribe el nombre oficial del fabricante ("CHEVROLET"), que se guarda en mayúsculas.
3. **Editar marca**, desde la fila o desde la ficha.
4. **Desactivar o activar** desde el menú de la fila o desde la ficha. Desactivar pregunta antes: "¿Desactivar la marca?", "La marca {nombre} no recibirá modelos nuevos, y sus modelos ya no podrán elegirse en unidades nuevas. Las unidades que ya la tienen la conservan."

El sistema revisa la forma del nombre y que no se repita, una vez en mayúsculas (`409 VEHICLEBRAND_CONFLICT`), también en una carrera.

**Errores que ve:** "Ya existe una marca con ese nombre." (`VEHICLEBRAND_CONFLICT`).
**Resultado:** "Marca oficial registrada exitosamente."

### F3 · La ficha de una marca

**Rol:** superadministrador. **Pantalla:** `/vehicle-catalog/brands-and-models/{id}`, que no es opción del menú.

1. La ficha empieza en sus datos: el panel "Datos de la marca", con el nombre y el estado, y "Editar marca" en su cabecera. Activar y Desactivar van también en esa cabecera; desactivar pregunta antes, y una marca inactiva lo dice en una línea corta junto a su insignia: "No se le registran modelos ni unidades nuevas.".
2. Debajo, el panel "Modelos de la marca": la lista de sus modelos, que se busca por nombre, se filtra por estado y se ordena, con el modelo, su tipo de vehículo, su cilindraje ("1500 cc", o "N/A"), su combustible ("Sin especificar" si no tiene), si trae pantalla de fábrica y su estado. Un tipo de vehículo inactivo se nombra igual; si los tipos no se pudieron leer, la columna lo dice en vez de mostrar identificadores.
3. "Nuevo modelo" abre el diálogo de F4. Una marca inactiva no lo ofrece (D-108).
4. "Desactivar" en el menú de un modelo pregunta antes: "¿Desactivar el modelo?", "El modelo {marca} {nombre} ya no podrá elegirse en unidades nuevas. Las unidades que ya lo tienen lo conservan."

El identificador de la marca se acepta en mayúsculas o minúsculas. **Errores que ve:** "No se encontró la marca solicitada." (`404 VEHICLEBRAND_NOT_FOUND`, también con un identificador mal formado); "No se pudo cargar la marca. Revise su conexión y vuelva a abrir la ficha."

### F4 · Registrar y corregir un modelo

**Rol:** superadministrador. **Pantalla:** la ficha de la marca, botón "Nuevo modelo" y "Editar" en el menú de cada modelo.

1. Escribe el nombre del modelo ("SAIL SEDAN 1.5L LT MT"), que se guarda en mayúsculas.
2. Elige el tipo de vehículo, entre los activos; al editar, también el inactivo que el modelo ya tiene. Si el modelo ya tiene unidades, el tipo se muestra bloqueado, con la pista "Ya no cambia: el modelo tiene unidades registradas.", como el código de un equipamiento (`vehicleTypeLocked`, D-107). Si el modelo tomó su primera unidad mientras el diálogo estaba abierto, el rechazo vuelve a poner el tipo que tiene y lo bloquea.
3. Elige el combustible. Al registrar un modelo nuevo nace vacío, sin proponer ninguno, y puede quedar vacío si no se conoce (D-109).
4. Escribe el cilindraje ("Cifra comercial en cc, sin decimales: 125, 200, 1500. Vacío en los eléctricos."). Un modelo eléctrico no lo lleva: el diálogo vacía el campo al elegir eléctrico.
5. Marca si trae pantalla de fábrica.
6. Guarda: `POST /vehicle-brands/{id}/vehicle-models` o `PUT /vehicle-models/{id}`.

El sistema revisa, en el alta: que la marca exista (`404 VEHICLEBRAND_NOT_FOUND`) y esté activa (`422 VEHICLEBRAND_INACTIVE`); que el tipo exista (`404 VEHICLETYPE_NOT_FOUND`) y esté activo (`422 VEHICLETYPE_INACTIVE`); que el combustible sea uno de la lista; que la marca no tenga otro modelo con ese nombre (`409 VEHICLEMODEL_CONFLICT`); y la forma de cada campo: el cilindraje es un entero mayor que cero —1500,0 es 1500, y 1500,7 se rechaza en vez de cortarse—, sin máximo salvo el que cabe en su columna, y vacío en un eléctrico (`400 VALIDATION_ERROR`). En la edición, además, que un tipo nuevo esté activo y que no cambie si el modelo ya tiene unidades (`422 VEHICLEMODEL_TYPE_LOCKED`); el modelo se lee con `FOR NO KEY UPDATE`, así que la marca de una primera unidad espera a la edición o la edición la ve. La marca de un modelo no cambia nunca.

**Errores que ve:** "Ya existe un modelo con ese nombre para esta marca." (`VEHICLEMODEL_CONFLICT`); "No se pudo guardar el modelo: revise sus datos. Un modelo eléctrico no lleva cilindraje, y el cilindraje es un número entero de cc." (`VALIDATION_ERROR`); "El tipo de vehículo de este modelo ya no cambia: el modelo tiene unidades registradas." (`VEHICLEMODEL_TYPE_LOCKED`), y la ficha vuelve a leer su página de modelos; "La marca está inactiva: no recibe modelos nuevos." (`VEHICLEBRAND_INACTIVE`), y la ficha vuelve a leer la marca; "El tipo de vehículo elegido está inactivo: elija uno activo." (`VEHICLETYPE_INACTIVE`), y el diálogo vuelve a leer los tipos. El diálogo queda abierto con lo escrito.
**Resultado:** "Modelo registrado exitosamente." y la lista de la ficha vuelve a leer su página.

### F5 · Mantener los colores

**Rol:** superadministrador. **Pantalla:** Catálogos › Colores, lista con diálogo.

1. Busca por nombre; la lista muestra el nombre y el estado.
2. "Nuevo color": el nombre ("BLANCO"), que se guarda en mayúsculas, para que `Blanco`, `BLANCO` y `blanco` no sean tres colores.
3. Desactivar o activar desde la fila; desactivar pregunta antes: "¿Desactivar el color?", "El color {nombre} ya no podrá elegirse en unidades nuevas. Las unidades que ya lo tienen lo conservan."

**Errores que ve:** "Ya existe un color registrado con ese nombre." (`409 COLOR_CONFLICT`, también en una carrera, en el alta y al renombrar).
**Resultado:** "Color registrado exitosamente."

### F6 · Mantener los tipos de equipamiento

**Rol:** superadministrador. **Pantalla:** Catálogos › Equipamiento, lista con diálogo.

1. Busca por código o nombre, y la API filtra también por el tipo de vehículo al que aplica hoy. La lista muestra código, nombre, si bloquea la disponibilidad, si requiere serie, a qué tipos aplica —un tipo inactivo se nombra igual— y el estado.
2. "Nuevo tipo de equipamiento": código, nombre, "Bloquea disponibilidad operativa", "Requiere registro de número de serie" y los tipos de vehículo aplicables, entre los activos. Sin ninguno no se guarda.
3. Al editar, el código de `GPS` y de `PANTALLA` se muestra bloqueado ("Ya no cambia: el sistema usa este código en sus reglas."); el de otro equipamiento se corrige ("Corregible mientras nada dependa del equipamiento."). El diálogo ofrece también los tipos inactivos a los que el equipamiento ya aplica, para poder desmarcarlos.
4. Desactivar o activar desde la fila; desactivar pregunta antes: "¿Desactivar el tipo de equipamiento?", "El equipamiento {nombre} ({código}) ya no podrá instalarse en las unidades. Lo que ya está instalado lo conserva."

El sistema revisa que los tipos existan (`404 VEHICLETYPE_NOT_FOUND`, con los que faltan), que haya al menos uno (`400 VALIDATION_ERROR`), que el código no se repita (`409 EQUIPMENTTYPE_CONFLICT`, también en una carrera) y que no se corrija uno que lee el programa (`422 EQUIPMENTTYPE_CODE_LOCKED`). Bloquea el equipamiento mientras decide un cambio, para que dos ediciones simultáneas no se pisen.

**Errores que ve:** "Ya existe un tipo de equipamiento con ese código."; "El código de este equipamiento ya no cambia: el sistema lo usa en sus reglas.", y el diálogo vuelve a poner el código guardado y lo bloquea.
**Resultado:** "Tipo de equipamiento registrado exitosamente."

### F7 · Exportar un catálogo

**Rol:** superadministrador. **Pantalla:** las cuatro listas y la lista de modelos de la ficha de una marca, botón "Exportar" (D-116; segunda fase de la auditoría).

1. Busca y filtra la lista como siempre, y pulsa "Exportar".
2. El archivo sale de su propio endpoint (`GET /vehicle-types`, `/vehicle-brands`, `/vehicle-brands/{id}/vehicle-models`, `/colors` y `/equipment-types`, con el `Accept` de Excel), con la misma búsqueda, los mismos filtros y el mismo orden, y todas las filas que coinciden, por la plataforma de documentos (convenciones §8): `303` al documento, que el frontend consulta hasta que está listo y descarga. Un pedido igual reutiliza el documento mientras no cambien los datos.
3. Cada lista es un tipo de documento de `VehicleCatalogDocumentKinds` —`vehicle-types`, `vehicle-brands`, `vehicle-models`, `vehicle-colors` y `equipment-types`—, con el alcance de un catálogo general, tener `catalogos.ver`, y la versión de las tablas que lee su exportación.

**Qué lleva cada archivo** (`tipos-de-vehiculo`, `marcas-de-vehiculo`, `modelos-de-vehiculo`, `colores`, `tipos-de-equipamiento`, con la fecha de hoy en Lima): los tipos, Código, Tipo de vehículo, Nombre comercial y Estado; las marcas, Marca y Estado; los modelos de una marca, Marca, Modelo, Código de tipo de vehículo, Tipo de vehículo, Cilindraje (cc), Combustible —traducido—, Pantalla de fábrica y Estado; los colores, Color y Estado; los equipamientos, Código, Equipamiento, Bloquea la disponibilidad, Exige número de serie, Aplica a («Nombre (CÓDIGO)», …) y Estado. Nunca un identificador (D-51); el tipo de vehículo va también por su código, porque su nombre puede repetirse.

**Errores que ve:** una marca que no existe responde `404 VEHICLEBRAND_NOT_FOUND` a la exportación de sus modelos; ante cualquier fallo, la pantalla dice "No se pudo descargar el archivo. Inténtelo de nuevo." (`data-manager.export-error`).

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). En la columna **Dónde se aplica**, los nombres de la base son restricciones de V35 (y la columna de V49); los demás, clases del backend o archivos del frontend. Las pruebas del backend están en `src/test/java/.../vehiclecatalog`; las del frontend, junto a cada archivo de `src/app/vehiclecatalog`, y las de extremo a extremo en `e2e/vehicle-catalog.spec.ts`.

| ID | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| VCT-R01 | Los catálogos de flota son del sistema: no llevan empresa ni Row Level Security | §7, D-15 | Base: V35, sin `company_id` | `VehicleCatalogMigrationTests.theCatalogsAreGeneralAndCarryNoCompany` | ✅ |
| VCT-R02 | Los lee quien tiene `catalogos.ver` (los siete roles) y los escribe quien tiene `catalogos.gestionar_flota` (en la semilla, solo el superadministrador); el permiso de los catálogos generales no escribe estos | §5.6, D-95 | Servicio: `@PermissionRequired` en los cinco controladores (`a39e111`). Frontend: las rutas las abre solo `SUPERADMIN` | `VehicleCatalogPermissionIntegrationTests` (sus cuatro casos); `*ControllerTests.onlySuperAdministratorsWrite` y `withoutSession…NotReadable` | ✅ |
| VCT-R03 | Nada se borra: un registro se desactiva y se reactiva, y una aplicabilidad también | §2.5, D-73 | Base: la cuenta de la aplicación no tiene `DELETE` (V35). Servicio: sin rutas `DELETE` | `*AuditTests.deactivatingKeepsTheRowAndTheCreationMoment`; `EquipmentTypeApplicabilityConstraintTests.theApplicationCannotDeleteIt`; `VehicleTypesControllerTests.typesCannotBeDeleted`; `*ControllerTests.a…IsDeactivatedAndActivatedAgainButNeverDeleted` | ✅ |
| VCT-R04 | Registran cuándo se crearon y cambiaron, sin autor | D-42, D-73 | Base: `created_at` y `updated_at`. Persistencia: la edición guarda sobre la misma fila | `ColorAuditTests`, `VehicleBrandAuditTests`, `VehicleModelAuditTests`, `VehicleTypeAuditTests`, `EquipmentTypeAuditTests` (también `changingOnlyTheVehicleTypesItAppliesToMovesTheModificationMoment`) | ✅ |
| VCT-R05 | Los códigos y los nombres de tipos, marcas, modelos y colores se guardan recortados y en mayúsculas, y la unicidad se mide sobre lo guardado | §2.2 | Dominio: `normalizeCode` y `normalizeName` de cada agregado. Base: los `*_uq` de V35. Frontend: los diálogos muestran el valor en mayúsculas | `ColorCommandServiceImplTests.aNameThatExistsOnceNormalizedIsAConflict`; `VehicleBrandCommandServiceImplTests.aNameThatExistsOnceNormalizedIsAConflict`; `VehicleTypeCommandServiceImplTests.aCodeThatExistsOnceNormalizedIsAConflict`; `color-form-dialog.spec` ("normalizes color name to uppercase…") | ✅ |
| VCT-R06 | El código de un tipo de vehículo no se repite y no cambia | §7.1 | Base: `vehicle_types_code_uq`. Dominio: `VehicleType.update` no recibe el código. Frontend: el código queda bloqueado al editar | `VehicleTypeTests.shouldUpdateDetailsAndKeepTheCode`; `VehicleTypesControllerTests.anEditChangesTheNamesAndNeverTheCode`; `vehicle-type-form-dialog.spec` ("locks the code control in edit mode") | ✅ |
| VCT-R07 | Una marca no repite nombre | §7.2 | Base: `vehicle_brands_name_uq`. Servicio: `409 VEHICLEBRAND_CONFLICT` | `VehicleBrandsControllerTests.aNameThatExistsOnceNormalizedIsAConflictAlsoOnARename`; `brands-and-models.store.spec` ("says the brand name is taken…") | ✅ |
| VCT-R08 | Una marca no tiene dos modelos con el mismo nombre; otra marca puede usarlo | §7.3 | Base: `vehicle_models_brand_name_uq`. Servicio: `409 VEHICLEMODEL_CONFLICT` | `VehicleModelConstraintTests.aBrandCannotHaveTwoModelsWithTheSameName` y `anotherBrandMayUseTheSameName`; `VehicleModelsControllerTests.theSameNameUnderTheBrandInAnotherCaseIsAConflictButNotUnderAnotherBrand` | ✅ |
| VCT-R09 | El cilindraje es un entero mayor que cero, o vacío; no tiene máximo salvo el que cabe en su columna, y un decimal se rechaza en vez de cortarse | §7.3 | Base: `vehicle_models_engine_cc_positive_ck`. Recurso: `@Positive`, `@WholeNumber` y `@DecimalMax` sobre un decimal. Frontend: `vehicle-model-form-dialog` | `VehicleModelConstraintTests.theDisplacementIsPositiveOrMissing`; `VehicleModelsControllerTests.theDisplacementIsAWholeNumberHoweverItIsWrittenAndIsNeverCut` y `theDisplacementHasNoMaximumButMustFitWhereItIsStored`; `vehicle-model-form-dialog.spec` ("takes only a whole number…", "takes a whole number written with a zero decimal…") | ✅ |
| VCT-R10 | Un modelo eléctrico no lleva cilindraje | §7.3 | Base: `vehicle_models_electric_engine_cc_ck`. Dominio: `VehicleModel`. Frontend: el diálogo vacía el campo | `VehicleModelConstraintTests.anElectricModelHasNoDisplacement`; `VehicleModelTests.anElectricModelHasNoDisplacement`; `VehicleModelsControllerTests.anElectricModelWithADisplacementIsAValidationError`; `vehicle-model-form-dialog.spec` ("clears and ignores engineCc when fuelType is electrico") | ✅ |
| VCT-R11 | El combustible es uno de la lista cerrada, guardado en minúsculas y sin tildes, o vacío | §7.3 | Base: `vehicle_models_fuel_type_ck`. Dominio: `FuelType`. Recurso: `@Pattern` | `VehicleModelConstraintTests.theFuelIsOneOfTheClosedListOrMissing`; `VehicleModelCommandServiceImplTests.aFuelOutsideTheListIsAValidationError` y `aModelWithoutFuelIsAccepted`; `FuelTypeTests` | ✅ |
| VCT-R12 | El combustible de un modelo nuevo no se propone: nace vacío y se elige | D-109 | Frontend: `vehicle-model-form-dialog` no propone combustible a un modelo nuevo (`23a7e9a`). Recurso: el combustible es opcional | `vehicle-model-form-dialog.spec` ("starts a new model without a fuel, proposes none and saves it empty"); `vehicle-model-assembler.spec` ("…keeps a model without fuel without one"); `VehicleModelCommandServiceImplTests.aModelWithoutFuelIsAccepted`; e2e ("registers a model of a brand without proposing a fuel…") | ✅ |
| VCT-R13 | La marca de un modelo no cambia | §7.3 | Dominio: `VehicleModel.vehicleBrandId` es fijo. Recurso: la edición no la recibe | `VehicleModelCommandServiceImplTests.updatingCorrectsTheVehicleTypeAndKeepsTheBrand`; `VehicleModelsControllerTests.anEditCorrectsTheVehicleTypeAndTheSpecificationsAndKeepsTheBrand` | ✅ |
| VCT-R14 | El tipo de vehículo de un modelo se corrige mientras el modelo no tenga unidades; desde la primera queda fijo. `fleet` avisa la primera unidad por la fachada, en su misma transacción | D-107, §7.3 | Base: `vehicle_models.has_units` (V49, rellenada con los modelos que ya están en `vehicles`). Fachada: `markModelInUse`, idempotente, con `MarkVehicleModelInUseCommand` y un `UPDATE … SET has_units = TRUE` que guardar el modelo nunca pisa. Dominio: `VehicleModel` (el tipo queda fijo con unidades). Servicio: `422 VEHICLEMODEL_TYPE_LOCKED`, con el modelo leído con `FOR NO KEY UPDATE`. Recurso: `vehicleTypeLocked`. Frontend: `vehicle-model-form-dialog` bloquea el tipo con su pista (`040ad1e`, `a448cb2`) | `VehicleModelTests.aNewModelHasItsVehicleTypeOpen` y `aModelWithUnitsHasItsVehicleTypeLocked`; `VehicleModelCommandServiceImplTests.theVehicleTypeOfAModelWithUnitsIsLocked`, `aModelWithUnitsIsStillCorrectedWhenItKeepsItsVehicleType` y `markingAModelInUseSaysWhetherThisCallMarkedIt`; `VehicleModelsControllerTests.aModelTravelsWithWhetherItsVehicleTypeIsLocked` y `theVehicleTypeOfAModelWithUnitsIsLockedButTheRestIsCorrected`; `VehicleCatalogContextFacadeTests.theFirstUnitMarksTheModelAndMarkingAgainChangesNothing`; `VehicleCatalogMigrationTests.aModelRecordsWhetherItHasUnitsAndANewOneHasNone`; en `fleet`, `VehicleRetiredReferencesTests.aRegistrationMarksItsModelAsHavingUnits` y `aRefusedRegistrationLeavesTheModelUnmarked`; `vehicle-model-form-dialog.spec` ("locks the type of a model with units, says why and keeps it on save", "locks the type when the model took its first unit meanwhile…"); `brands-and-models.store.spec` ("says the type of a model with units no longer changes…"); e2e ("keeps the type of a model that has units…") | ✅ |
| VCT-R15 | Un modelo nuevo es de una marca activa y de un tipo activo; al editarlo, un tipo nuevo tiene que estar activo, y el inactivo que ya tenía se conserva. La ficha de una marca inactiva no ofrece "Nuevo modelo" | D-108, D-84 | Servicio: `VehicleModelCommandServiceImpl`, `422 VEHICLEBRAND_INACTIVE` y `VEHICLETYPE_INACTIVE` (`040ad1e`). Frontend: el diálogo ofrece los tipos activos más el inactivo que el modelo tiene; la ficha de una marca inactiva no ofrece "Nuevo modelo" (`9c254bf`) | `VehicleModelCommandServiceImplTests.aModelIsNotCreatedUnderAnInactiveBrand`, `aModelIsNotCreatedOfAnInactiveVehicleType`, `aModelIsNotMovedToAnInactiveVehicleType` y `theInactiveVehicleTypeAModelAlreadyHasIsKeptOnEdit`; `VehicleModelsControllerTests.noModelIsCreatedUnderAnInactiveBrandNorOfAnInactiveVehicleType` y `aModelKeepsTheInactiveVehicleTypeItHasButIsNotMovedToOne`; `vehicle-model-form-dialog.spec` ("keeps the vehicle type editable and offers the inactive one the model has", "offers only the active types to a new model", "reads the types again when the one chosen was deactivated meanwhile"); `brands-and-models.store.spec` ("says the type chosen is inactive on VEHICLETYPE_INACTIVE", "says the brand is inactive on VEHICLEBRAND_INACTIVE…"); `brand-profile-view.spec` ("says once beside the status what an inactive brand means, offers Activar and no new model"); e2e ("deactivates a brand from its record once confirmed, and takes no model while inactive") | ✅ |
| VCT-R16 | Una unidad nueva no se registra con un modelo inactivo, ni con uno cuya marca o tipo estén inactivos | D-108, §8.1 | Fachada: `VehicleModelSummary` con `isActive`, `brandIsActive` y `vehicleTypeIsActive`, resueltos en una consulta por clase para un lote. La aplica `fleet`: `ExternalVehicleCatalogService.resolveModelForNewUnit` (FLT-R07) | `VehicleCatalogContextFacadeTests.aModelTravelsWithTheStateOfItsBrandAndOfItsVehicleType` y `aBatchAnswersEveryModelThatExistsWithItsStates`; en `fleet`, `VehicleRetiredReferencesTests.aRetiredModelIsNotGivenToANewUnit`, `aModelOfARetiredBrandIsNotGivenToANewUnit` y `aModelOfARetiredVehicleTypeIsNotGivenToANewUnit` | ✅ |
| VCT-R17 | La exigencia de pantalla sale del catálogo —si el equipo aplica al tipo y si el modelo la trae de fábrica—, nunca de comparar textos | §7.3, §7.5 | Base: `has_factory_screen` y la aplicabilidad. La regla que las combina llega con el paso 9 | `VehicleModelTests.shouldUpdateSpecifications`; `VehicleCatalogMigrationTests.theGpsAppliesToEveryVehicleTypeAndTheScreenOnlyToCars` | ⚠️ |
| VCT-R18 | Un equipamiento aplica al menos a un tipo de vehículo; quitarle uno desactiva esa aplicabilidad y volver a marcarlo la reactiva; solo cuenta la activa | §7.5, D-73 | Base: `equipment_type_vehicle_types_pk` e `is_active`. Dominio: `EquipmentType`. Frontend: `equipment-type-form-dialog` | `EquipmentTypeTests.itAppliesToAtLeastOneVehicleType`, `withdrawingATypeDeactivatesItsApplicabilityInsteadOfRemovingIt` y `checkingAWithdrawnTypeAgainReactivatesTheSameApplicabilityAndANewTypeIsAdded`; `EquipmentTypeApplicabilityPersistenceTests`; `EquipmentTypesControllerTests.anEditCannotLeaveItApplyingToNoVehicleType`; `equipment-type-form-dialog.spec` ("keeps an equipment type without any vehicle type from being saved") | ✅ |
| VCT-R19 | Los códigos `GPS` y `PANTALLA` quedan fijos desde la semilla; el de otro equipamiento se corrige mientras ninguna instalación lo use | D-74, §2.4 | Dominio: `EquipmentType.isCodeLocked`. Servicio: `422 EQUIPMENTTYPE_CODE_LOCKED`. Recurso: `codeLocked`. Frontend: el diálogo bloquea el código. Que una instalación lo fije espera a `vehicle_equipments`, en el paso 9 | `EquipmentTypeTests.theCodesTheProgramReadsAreLocked` y `anOwnCodeIsCorrectedAndStoredNormalized`; `EquipmentTypesControllerTests.anOwnCodeIsCorrectedWhileTheCodesTheProgramReadsAreLocked`; `equipment-type-form-dialog.spec`; e2e ("keeps the code of an equipment type the program reads") | ⚠️ |
| VCT-R20 | Dos cambios de un mismo tipo de equipamiento se deciden de a uno | [Patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md) | Persistencia: `EquipmentTypeRepositoryImpl.findByPublicIdForUpdate`, con `FOR NO KEY UPDATE` | `EquipmentTypeLockTests` | ✅ |
| VCT-R21 | Un duplicado que la base detiene en una carrera responde el `*_CONFLICT` de su recurso, nunca un `500` | Convenciones §7.2 | Persistencia: los cinco adaptadores traducen la violación de `vehicle_types_code_uq`, `vehicle_brands_name_uq`, `vehicle_models_brand_name_uq`, `colors_name_uq` y `equipment_types_code_uq` con `UniqueViolations` en `NaturalKeyTakenException`, y los servicios responden el `*_CONFLICT` de cada recurso (`040ad1e`) | `VehicleCatalogRacedDuplicatesTests` (sus seis casos, uno por catálogo y el color renombrado sobre un nombre tomado mientras tanto); `VehicleModelCommandServiceImplTests.aNameTakenByARacingRequestIsTheSameConflict` | ✅ |
| VCT-R22 | La semilla trae los tres tipos con sus dos nombres, y el GPS y la pantalla con sus banderas y sus tipos | §7.1, §7.5 | Base: V35 | `VehicleCatalogMigrationTests.theThreeVehicleTypesComeWithTheirTwoNames`, `theGpsAndTheScreenBlockAvailabilityAndOnlyTheGpsRecordsASerial` y `theGpsAppliesToEveryVehicleTypeAndTheScreenOnlyToCars` | ✅ |
| VCT-R23 | Desactivar un registro pregunta antes, con su nombre | DESIGN.md; como Estados y motivos (FLT-R25) | Frontend: `deactivationQuestion` (`vehiclecatalog/presentation/components`), con las claves `vehicle-catalog.confirm.*` y `destructive: true`, en las cuatro listas, en la ficha de la marca y en sus modelos (`929443a`, `9c254bf`) | `deactivation-question.spec`; `vehicle-types-view.spec`, `brands-and-models-view.spec`, `colors-view.spec` y `equipment-types-view.spec` ("asks before deactivating…, as a destructive action"); `brand-profile-view.spec` ("asks before deactivating the brand, then deactivates it", "asks before deactivating a model of the brand…"); e2e (alta, edición, duplicado y desactivación en cada catálogo) | ✅ |
| VCT-R24 | Las listas se buscan, se filtran, se ordenan y se paginan en el servidor; un filtro por un tipo de vehículo que no existe no coincide con ninguna fila | Convenciones §6.2 | Persistencia: los `*RepositoryImpl`, con su `SORTABLE`. Frontend: `PagedList` en cada store | `VehicleTypesControllerTests.withAPageTheListAnswersOnePageInOrderAndHowManyMatch` y `theSearchReadsTheNamesAndTakesItsWildcardsLiterally`; `EquipmentTypeQueryServiceImplTests.anUnknownVehicleTypeMatchesNoEquipmentType`; `VehicleModelQueryServiceImplTests.anUnknownVehicleTypeMatchesNoModel`; e2e ("asks the server for the vehicle types of the status filter") | ✅ |
| VCT-R25 | Los cuatro catálogos y los modelos de una marca se exportan a Excel desde su propio endpoint | D-116 | Servicio: `VehicleCatalogDocumentKinds` y el `@GetMapping(produces = SpreadsheetExporter.MEDIA_TYPE)` de los cinco controladores, sobre la plataforma de documentos (`be7bdb9`). Frontend: `exportVehicleTypes`, `exportBrands`, `exportModels`, `exportColors` y `exportEquipmentTypes` en los stores, y las cinco listas `exportable` (`f73a656`) | `VehicleCatalogExportsIntegrationTests` (`theVehicleTypesExportTheirCodeNamesAndState`, `theBrandsExportTheirNameAndStateInTheOrderOfTheList`, `theModelsOfABrandExportTheBrandTheVehicleTypeAndTheFuelWithTheirFilters`, `theModelsOfAnUnknownBrandAnswerItsNotFound`, `theColorsExportTheirNameAndState`, `theEquipmentTypesExportTheVehicleTypesTheyApplyTo`, `inEnglishTheHeadersAndTheFuelAreInEnglish`); los specs de los cinco stores ("downloads the … on screen as the Excel file the backend generates, and says when it fails") y de las vistas ("offers the … as an Excel file…"); e2e ("downloads the vehicle types with the status filter of the list (D-116)" y los demás) | ✅ |

---

## 5. Relación con otros contextos

**Las dependencias.** `vehiclecatalog` declara `allowedDependencies = {}`, y `ModularityTests` lo verifica: no llama a ningún contexto. El mapa le permite `shared` y `location` (mapa §4), y hoy no usa `location`. La única lectura de una tabla ajena es la de la migración V49, que llena `has_units` con los modelos que ya están en `vehicles.model_id`; es parte de la migración, no del código del contexto, y la migración lo dice.

**Lo que ofrece: `VehicleCatalogContextFacade`**, que consume solo `fleet`, por su `ExternalVehicleCatalogService`.

| Miembro | Para qué |
|---|---|
| `fetchVehicleModelSummaryByPublicId` | El modelo del alta de una unidad: si existe, si está activo y si lo están su marca y su tipo (D-108) |
| `fetchColorSummaryByPublicId` | El color del alta y de la edición, con si está activo |
| `fetchVehicleModelSummariesByIds`, `fetchColorSummariesByIds` | Los modelos y los colores de una lista de unidades, en una lectura, para las vistas |
| `fetchDataVersion()` | La versión de los datos que muestra —tipos, marcas, modelos y colores—, para que el Inventario de `fleet` no se sirva desde un documento viejo cuando uno cambia (D-106; segunda fase, `be7bdb9`) |
| `markModelInUse(Long modelId)` | La única escritura: pone `has_units` en verdadero cuando `fleet` registra una unidad del modelo. Es idempotente y corre en la transacción de quien llama, así que si el alta falla, el modelo tampoco queda marcado (D-107) |

`VehicleModelSummary` lleva `id`, `publicId`, `name`, `isActive`, `brandIsActive` y `vehicleTypeIsActive`; `brandId`, `vehicleTypeId`, `engineCc`, `fuelType` y `hasFactoryScreen` no tenían consumidor y salieron, con el `toLowerCase` del combustible (`040ad1e`, cambio incompatible de la fachada). La segunda fase le sumó `brandName`, `vehicleTypeCode` y `vehicleTypeName`, que nombran la marca y el tipo en la exportación del Inventario (`be7bdb9`). `ColorSummary` lleva `id`, `publicId`, `name` e `isActive`.

**Sin ciclo.** Que `fleet` escriba por la fachada no invierte el sentido de la dependencia: `fleet` llama a `vehiclecatalog`, como para leer, y `vehiclecatalog` no llama a `fleet` ni lee sus tablas (mapa §6.4).

**Los puertos de `shared`.** Los controladores exigen su permiso con `@PermissionRequired`, y las listas usan `PageParams`, `PageQuery` y `PageRequests`. No lee `CurrentAccessScope`: un catálogo general no se acota por empresa. Las exportaciones usan la plataforma de documentos de `shared` (D-106, D-116): `DocumentKind`, `DocumentScopes.holding`, `DataVersions` y `DocumentCommandService`.

**Frontend.** `VehicleCatalogFacade` (`vehiclecatalog.facade.ts`) ofrece a `fleet` vistas mínimas: los tipos de vehículo activos, las marcas activas y todas, los colores, los modelos activos de una marca (`loadActiveModels`) y un modelo por su identificador (`loadModelOption`), leídos por los stores del contexto. Desde la auditoría (`a2ef8f2`), cada catálogo que ofrece tiene su propio estado, apartado del de las pantallas: los stores dueños guardan para la fachada sus filas, su carga y su error externos (`loadingExternal`, `externalError`, `clearExternalError()`), y la fachada los expone como `vehicleTypesLoading` y `vehicleTypesError`, `brandsLoading` y `brandsError`, `colorsLoading` y `colorsError`, cada error con su `clear…Error()`, como `LocationFacade`. Una lectura de `fleet` nunca deja su error, su carga ni sus filas en las pantallas de catálogos. Los miembros que nadie usaba salieron (`139e513`), y la fachada ya no lee el equipamiento.

---

## 6. Estado y pendientes para cerrar

**Estado al 20/10/2026: construido en `develop`, abierto.** Falta lo que el orden de implementación pide para cerrar el paso: los modelos reales conciliados (§7.3), y la validación de este documento por el responsable. Lo que aprobó la auditoría del 20/10/2026 está construido e integrado en `audit/be-integration` (merge `81bb8c1`) y `audit/fe-integration` (merge `36210f6`), con su segunda fase, la exportación (D-116), y falta llevarlo a `develop`.

**Cómo se llegó.** El 17/10/2026 el backend tomó el catálogo de `feature/vehiclecatalog` (johanncamiloaga, coautor de esos commits) y lo adaptó al diccionario v1.19: el esquema y sus semillas en V35 (`368619c`), las rutas sin prefijo de contexto (`d8af95a`), la búsqueda en el servidor (`88f4dcd`), la aplicabilidad que se desactiva en vez de borrarse, con su cambio registrado como una modificación del equipamiento (`bebc940`), el cilindraje entero y sin máximo (`d31ed73`, `3769edd`), el eléctrico sin cilindraje (`0f2e46f`) y el código corregible salvo el que lee el programa (`4f80289`, `9f1b443`). El paso 3b le puso los permisos (`a39e111`), y el paso 5 la fachada que usa `fleet` (`7ff2949`, `d54b8d4`). El frontend se construyó el mismo 17/10/2026, con los modelos anidados bajo su marca (`d0a9440`) y las listas en el servidor (`d035363`); el 20/10/2026 la ficha de la marca empezó en sus datos (`71907ad`) y la carpeta pasó a llamarse como el contexto (`8d4fd27`, en `audit/fe-base`).

### a) Decisiones

Las de origen: los catálogos son del sistema (D-15) y se mantienen aunque tengan pocas filas (D-17); registran fechas sin autor y la aplicabilidad se desactiva (D-73); `GPS` y `PANTALLA` quedan fijos desde la semilla (D-74). El responsable aprobó el 20/10/2026, con la auditoría:

1. **D-107.** El tipo de un modelo queda fijo desde su primera unidad, que `fleet` avisa por la fachada (`markModelInUse`, `has_units`).
2. **D-108.** Ni modelos bajo una marca o un tipo inactivos, ni unidades con un modelo, una marca o un tipo inactivos; lo que un registro ya tiene se conserva.
3. **D-109.** El combustible de un modelo nuevo nace vacío.
4. **D-112.** Marcas y modelos es una lista con la ficha de cada marca, no un maestro-detalle.
5. **D-116.** Los catálogos y los modelos de una marca se exportan a Excel, construido en la segunda fase.

### b) Lo que construyó la auditoría

Los hallazgos son del plan de la auditoría del 20/10/2026 (B1 a B17). Están construidos e integrados, salvo B10, hecho antes (`8d4fd27`), y B14, que queda fuera de esta ronda. La exportación llegó en la segunda fase.

| Hallazgo | Qué cambió | Commits |
|---|---|---|
| B2, D-107 | V49 `has_units`; `markModelInUse`; `422 VEHICLEMODEL_TYPE_LOCKED`; `vehicleTypeLocked` en el recurso del modelo; el diálogo bloquea el tipo (VCT-R14) | Backend `040ad1e`, `f11e4f6`; frontend `a448cb2` |
| B3, D-108 | `422 VEHICLEBRAND_INACTIVE` al crear un modelo, y `VEHICLETYPE_INACTIVE` al crearlo o al cambiarle el tipo; `brandIsActive` y `vehicleTypeIsActive` en la fachada (VCT-R15, VCT-R16) | Backend `040ad1e` |
| B4 | La carrera de un duplicado responde el `*_CONFLICT` de cada recurso (VCT-R21) | Backend `040ad1e` |
| B5, B6 | El combustible ya no pasa por la fachada: salió con los demás campos sin consumidor, y con él el `toLowerCase` | Backend `040ad1e` |
| B7 | "Desactivar" pregunta antes en las cuatro listas y en los modelos de la ficha (VCT-R23) | Frontend `929443a` |
| B8, B11 | `VehicleCatalogFacade` con su propio estado de carga y de error por catálogo; fuera los miembros sin uso y la lectura del equipamiento | Frontend `a2ef8f2`, `139e513` |
| B12 | Cada `*.resource.ts` pasó a su `*-response.ts`, como en `location` | Frontend `191e502` |
| B13 | La ficha de la marca con Activar y Desactivar, con su línea junto a la insignia, y sin "Nuevo modelo" cuando está inactiva | Frontend `9c254bf`, `51e8cf4` |
| D-109 | El combustible de un modelo nuevo nace vacío (VCT-R12) | Frontend `23a7e9a` |
| B15 a B17 | Las pruebas de la fachada, del bloqueo del tipo, de la marca y el tipo inactivos, de la carrera, y las de extremo a extremo del alta, la edición, el duplicado y la desactivación en los cinco catálogos | Backend `040ad1e`; frontend `100485d` |
| B1, B9, D-116 | La exportación de los cinco catálogos y de los modelos (VCT-R25), en la segunda fase | Backend `be7bdb9`; frontend `f73a656` |

### c) Pendientes, cada uno con su momento

| Qué | Quién y cuándo |
|---|---|
| Los modelos reales, conciliados entre los dos catálogos entregados, que no coinciden (§7.3, advertencia para la carga inicial) | El negocio entrega los datos; criterio de cierre del paso 4 |
| Validar este documento | El responsable |
| Llevar a `develop` lo que construyó la auditoría, ya integrado y verificado en este documento el 20/10/2026 | El integrador, al cerrar la ronda || La ficha de la unidad descarga los catálogos completos para nombrar la marca y el color (B14) | Fuera de esta ronda; seguimiento del frontend |
| Las pantallas se abren por el rol `SUPERADMIN` (`RouteAccess` por roles), no por `catalogos.gestionar_flota`: un rol propio con ese permiso (D-98) no ve las opciones, aunque la API le responda | Seguimiento del frontend, para toda pantalla que exige un permiso (procesos, apartado 3.1) |
| El código de un equipamiento que una instalación usa queda fijo (D-74), y si una unidad requiere pantalla lo responde la fachada | Paso 9, con `vehicle_equipments` |
| `q` todavía distingue tildes (convenciones §6.2) | Antes del paso 7 |
| La sonda contra el backend real (`e2e-live`) | El responsable crea `erp_mm_db_live`; criterio de cierre del paso |

### d) Diferencias con las convenciones de la API

Ninguna conocida. Las altas responden `201` con `Location`, las listas siguen §6.2, los duplicados responden el código de su recurso, también en una carrera (B4: antes llegaba al manejador global como el genérico `RESOURCE_CONFLICT`), y nada tiene `DELETE`.

---

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿La pantalla bloquea la disponibilidad en todos los autos o solo en los modelos que no la traen de fábrica? La semilla sigue §7.5 | [Orden de implementación](../orden-de-implementacion-millennium-mobility.md), paso 4; diccionario §7.5 y §11.9 |
| ¿Cómo se concilian los dos catálogos de modelos entregados, que escriben el mismo modelo de formas distintas? | Diccionario §7.3, advertencia para la carga inicial |
| ¿Qué dato nuevo admitiría un catálogo de flota: el color en hexadecimal, el peso, los pasajeros, la garantía? | Diccionario §7.7, candidatos del bloque 4 |
