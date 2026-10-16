# Diccionario de datos — ERP Millennium Mobility

**Versión:** 1.17 · **Motor:** PostgreSQL 18 · **Estado:** módulo de stock completo. Bloques 1 a 11 cerrados; ventas y contratos fuera del alcance actual. 49 tablas más `role_permissions`, la tabla de unión de roles y permisos

> **Cómo leer los cuadros.** En cada tabla, las columnas *Descripción* y *Por qué importa* están escritas para que las entienda cualquier persona, sin conocimientos técnicos: sirven para que quien conoce el negocio pueda revisar si el dato es el correcto. Las columnas *Tipo* y *Nulo*, las restricciones y el código SQL son para el equipo de desarrollo.

---

## 1. Propósito y alcance

Este documento define la estructura de datos del ERP que reemplazará la gestión actual en Excel y el sistema Sigma. Es la fuente única del diseño: cada tabla se documenta con su propósito, sus campos, el porqué de cada campo y las reglas que la gobiernan.

No documenta pantallas ni procesos. Cuando una regla de negocio determina una restricción de base de datos, se indica; cuando la regla vive solo en la aplicación, también se dice, para que nadie asuma que la base la protege.

**Lo que este documento no hace:** copiar las 52 columnas del exportable actual. El objetivo declarado del proyecto es sistematizar el proceso, no trasladar la hoja de cálculo a una pantalla web. Varias reglas del exportable son reglas de celda —montos repetidos en cada fila, códigos concatenados con `/`— que no pueden ser reglas de almacenamiento.

---

## 2. Convenciones

### 2.1 Nombres

| Regla | Ejemplo correcto | Incorrecto |
|---|---|---|
| `snake_case`; dígito solo si es parte del nombre del estándar | `iso_alpha_2` | `iso2`, `isoCode2` |
| Llaves foráneas terminan en `_id` | `default_currency_id` | `currency` |
| Marcas de tiempo (`TIMESTAMPTZ`) terminan en `_at`, y solo ellas | `created_at` | `creation_date`, `hired_at` para una fecha sin hora |
| Fechas sin hora (`DATE`) terminan en `_date` | `invoice_date`, `hire_date` | `invoice`, `hired_at` |
| Un mes se guarda como `DATE` con día 1 y termina en `_month` | `campaign_month` | `campaign_period` |
| Un período se guarda como rango en una columna `period` | `period` | `valid_from` + `valid_to` |
| Quien hizo algo se nombra con el participio y `_by`, y apunta a `users` | `created_by`, `granted_by` | `created_by_id`, `creator` |
| Booleanos de estado o atributo empiezan con `is_` o `has_` | `is_system`, `has_factory_screen` | `system`, `factory_screen` |
| Booleanos que activan una regla empiezan con el verbo de la regla | `requires_serial`, `blocks_availability`, `must_change_password` | `serial_required` |
| Tabla en plural, columna en singular | `countries.name` | `country.names` |
| Sin prefijo de tabla en la columna | `countries.name` | `countries.country_name` |

### 2.2 Estructura

| Regla | Fundamento |
|---|---|
| Llave primaria subrogada `BIGINT` con `GENERATED ALWAYS AS IDENTITY`; toda llave foránea que la referencia también es `BIGINT` | `SERIAL` no es estándar SQL y permite insertar valores que desincronizan la secuencia. `BIGINT` corresponde al `Long` con que el código maneja los identificadores |
| Llaves naturales (ISO, TIN, VIN, placa) protegidas con `UNIQUE`, nunca como PK | Una PK no debe depender de un organismo externo |
| Toda tabla lleva `id` interno y `public_id`. No se repiten en los cuadros | El `id` es el número con que el sistema la reconoce; el `public_id` es el que sale al exterior (§2.4) |
| Las columnas de registro —`created_at`, `updated_at`, `created_by`, `updated_by`— van donde el cuadro de cada tabla las pide, y las llena el sistema, nunca la pantalla | Las fechas se ponen al guardar y el autor sale de la sesión. Un proceso sin sesión, como el registro público o el arranque, registra a `SYSTEM` de forma explícita (D-42) |
| **Cada tabla con identidad propia lleva `public_id UUID DEFAULT uuidv7()`** | Es el identificador que sale al exterior. El `id` interno nunca aparece en una URL, una respuesta de API ni un archivo exportado |
| El `code` de un catálogo queda fijo desde que algo lo usa (§2.4) | El nombre visible se corrige cuando haga falta; el código no, porque hay mensajes, archivos y sistemas externos que ya lo usan |
| Es tabla lo que administra el usuario; es lista cerrada lo que interpreta el programa | La prueba: ¿puede el cliente agregar un valor un martes y que funcione solo? Un color, sí; un nivel de alcance, no, porque habría que programar cómo filtra |
| Los derivados no se persisten | Columnas generadas `VIRTUAL` (por defecto en PG 18) o vistas. Un derivado almacenado se desincroniza |
| **Todo monto viaja con su moneda** | O la fila tiene `currency_id NOT NULL` junto al monto, o la hereda del encabezado por llave foránea. Un monto sin moneda no se guarda |
| **Toda conversión viaja con su tipo de cambio, y la fecha de ese tipo de cambio es la del documento** | El tipo de cambio de una factura es el de su fecha de emisión; el de un pago, el de su fecha de pago. No hace falta una columna aparte, pero la regla debe estar escrita o alguien usará el tipo de cambio del día en que registra |
| Montos: `NUMERIC`, `NOT NULL DEFAULT 0` | Un `NULL` en un monto se propaga por toda la aritmética y llega a pantalla como celda vacía |
| Vigencias: un rango en la columna `period`, con el inicio incluido y el fin excluido; fin vacío = vigente (§2.6) | Una vigencia se cierra, nunca se borra |
| Intervalos sin solapamiento: `daterange` o `tstzrange` + `WITHOUT OVERLAPS` (PG 18) | Impide dos períodos cruzados incluso en el pasado |
| Identificadores de texto normalizados al guardar: `TRIM` + `UPPER`, sin guiones. Los correos electrónicos, `TRIM` + `LOWER` | `UNIQUE` no distingue `ABC-123` de `ABC123`, ni `Tere@…` de `tere@…` |
| Centinelas (`-`, `NO TIENE`, `NO CORRESPONDE`) no se almacenan | Se generan al exportar. Guardados, obligan a parsear texto en cada cálculo |

La regla de moneda no es teórica. La guía funcional tuvo que enunciar como norma que una nota de crédito debe estar en la misma moneda que su factura (decisión A-06). Esa regla no debería necesitar enunciarse: debería ser imposible violarla.

**Tablas técnicas.** Las que administran las librerías —el historial de migraciones (`flyway_schema_history`), el registro de eventos entre contextos (`event_publications`) y las sesiones (`spring_session` y `spring_session_attributes`, §5.12)— conservan la estructura que exige su librería y no siguen estas convenciones: no llevan `public_id` ni forman parte del modelo del negocio.

### 2.3 Criterio de admisión de campos

> Un campo entra si existe **alguien que lo consume** o **una regla de negocio que lo necesita**.
> "Puede servir después" y "por consistencia" no son razones.

Los campos que no pasan el filtro quedan listados como **candidatos**, con el criterio concreto que los admitiría. Agregar una columna nullable después es un `ALTER TABLE` sin migración; quitar una que ya tiene datos, no.

### 2.4 Los tres identificadores

| Identificador | Ejemplo | Para qué sirve | Quién lo ve |
|---|---|---|---|
| `id` interno | `312` | Unir tablas entre sí | Nadie fuera de la base |
| `public_id` | `019352e1-…-7f3a` | Direcciones web e integraciones por API | Los sistemas externos |
| Código de negocio | VIN, placa, RUC | Identificar la cosa real | Las personas |

**Reglas**

- Toda tabla con identidad propia lleva `public_id`, generado como UUID versión 7. No lo llevan las tablas que solo unen otras dos.
- El `id` interno no aparece nunca en una dirección web, una respuesta de API ni un archivo exportado. Tampoco en las llaves foráneas que se devuelven: un vehículo devuelve el `public_id` de su empresa, no su número interno.
- La llave primaria sigue siendo el entero. El UUID es una columna aparte con valor único.
- Las direcciones web usan siempre el UUID. Dentro de los mensajes, los catálogos que siguen un estándar externo viajan por su código —`"moneda": "PEN"`— y las entidades del negocio por UUID.
- Los archivos y documentos que salen del sistema llevan códigos de negocio y nombres —VIN, placa, RUC—, nunca el `id` interno ni el `public_id`. Excepción: un archivo diseñado para editarse y volver a subirse con el fin de actualizar registros existentes lleva además la columna `public_id` como llave estable, porque un código o un nombre pueden corregirse entre la descarga y la subida. El criterio es el uso de ida y vuelta, no quién recibe el archivo (D-51).
- El `code` de un catálogo propio se puede corregir mientras nada dependa de él; una vez que hay registros que lo usan, queda fijo. Los códigos de normas externas no se editan nunca. La única excepción son los dos códigos ISO del país (D-47): se validan contra la lista de ISO 3166 y se corrigen mientras el país no tenga nada colgado. Extender esa excepción a otro código externo —moneda, sigla, ubigeo— requiere una decisión nueva (D-48).

### 2.5 Separación de datos por empresa

**Qué alcanza cada asignación**

| Alcance | Qué datos alcanza |
|---|---|
| Todo el sistema | Todas las empresas, incluidas las que se creen después |
| Una empresa | Todos los datos de esa empresa |
| Una empresa y una marca | Solo lo de esa marca dentro de esa empresa |
| Una sede | Solo lo de esa sede |

**Dos clases de dato, dos reglas**

| Clase | Ejemplos | Quién lo ve | Qué se restringe |
|---|---|---|---|
| Recurso compartido | Stock, catálogos, proveedores | Toda la empresa | Dónde se puede actuar sobre él |
| Responsabilidad asignada | Cartera de clientes, contratos | Solo quien la tiene asignada | Se amplía con un permiso |

**Reglas**

- Toda tabla que se consulte por sí sola o que acumule volumen lleva su `company_id`, aunque se pudiera deducir siguiendo la cadena de relaciones. Las filas que solo se leen a través de su padre se apoyan en él.
- Los catálogos generales —monedas, países, divisiones territoriales, tipos de documento tributario, permisos— no se filtran: son iguales para todos.
- Los usuarios no pertenecen a una empresa. El administrador de una empresa ve a quienes tienen alguna asignación vigente en ella.
- El stock es visible para toda la empresa. La sede limita dónde se puede actuar, no qué se ve.
- Hay campos que no ve cualquiera aunque vea la fila. Cuando falta el permiso, el campo no sale de la base: ocultarlo en la pantalla no alcanza.
- El acceso total es una excepción explícita evaluada con las mismas reglas, no una cuenta que se saltea el control.
- Cuando existan clientes, la tabla de direcciones tendrá que quedar acotada igual que el resto: el domicilio de un cliente sí es un dato sensible.

### 2.6 Fechas y horas

Decisiones D-40 y D-49. Las reglas valen para la base, la API, los archivos exportados y la bitácora de cambios.

| Regla | Fundamento |
|---|---|
| Un momento exacto —cuándo se registró, se aprobó o se bloqueó algo— es `TIMESTAMPTZ` y termina en `_at` | PostgreSQL lo guarda en UTC y lo convierte a cualquier zona al leerlo. Dos registros hechos en Lima y en Bogotá se ordenan bien sin cálculos, y la zona del servidor no importa. Nunca `TIMESTAMP` sin zona: guardaría la hora sin saber de qué país es |
| Una fecha de calendario —la emisión de una factura, el ingreso de un colaborador— es `DATE` y termina en `_date` | No tiene hora ni zona: el 15 de marzo es el 15 de marzo en cualquier país. Guardada con hora, cambiaría de día al convertirla de zona |
| Un período es un rango (`daterange` o `tstzrange`) en la columna `period`, con el inicio incluido y el fin excluido: `[inicio, fin)`. Fin vacío = sigue vigente | Un período termina exactamente donde empieza el siguiente, sin huecos ni cruces, y `WITHOUT OVERLAPS` lo valida. El GPS que se cambia el 04/07/2026 cierra el anterior en esa fecha y abre el nuevo en la misma |
| Un mes es un `DATE` con día 1 y termina en `_month` | Septiembre de 2026 se guarda como `2026-09-01`, y así se compara y se ordena como cualquier fecha |
| **Todo intercambio usa ISO 8601** —la API, los archivos para otros sistemas y los valores de fecha de la bitácora—: fechas `AAAA-MM-DD`, momentos `AAAA-MM-DDThh:mm:ssZ` en UTC | Es el único formato que no se presta a confusión: `01/03/2025` es 1 de marzo en Perú y 3 de enero en Estados Unidos. Además se ordena bien como texto |
| **En una hoja de cálculo exportada**, la fecha es una celda de fecha con formato `AAAA-MM-DD` y el momento una celda de fecha y hora con formato `AAAA-MM-DD hh:mm:ss`, en una sola zona por archivo, nombrada en el encabezado de la columna: `Último acceso (America/Lima)`. La zona sale de la regla de "hoy" aplicada al alcance de la exportación: la de la sede si el listado es de una sede; si no, la del país de la empresa si es de una empresa; si no, `America/Lima` (D-49) | Una celda de hoja de cálculo guarda el valor sin zona: en UTC no coincidiría con la hora que la persona ve en pantalla, y con zonas mezcladas en una columna dejaría de ordenarse por el momento real. Las celdas son de fecha y no texto, para que la hoja sume, filtre y ordene; quien prefiera otra máscara la cambia en la hoja sin alterar el valor |
| En la API, un período viaja como dos campos, desde y hasta; hasta vacío = abierto | Cualquier cliente lo entiende sin conocer la notación de rangos de PostgreSQL |
| Las zonas horarias se guardan con su nombre IANA: `America/Lima`, `America/Bogota` | Un nombre, a diferencia de un desfase fijo como `-05:00`, sigue los cambios de horario que decida cada país |
| "Hoy" y las fechas de operación se calculan en la zona de la sede donde ocurre el hecho. Sin sede, en la del país de la empresa del registro; si el registro no pertenece a ninguna empresa —una cuenta, un bloqueo—, en `America/Lima` | Una factura registrada a las 11 de la noche en Lima es de ese día en Lima, aunque en UTC ya sea el siguiente. De esto dependen la antigüedad de 20 días de una factura, los vencimientos y los cortes del día |
| Una hora local futura o que se repite —un corte diario a las 18:00, un horario de atención— no es un momento: se guarda como hora local junto con la zona de la sede, y se convierte a UTC recién al usarla. Hoy ninguna tabla lo necesita | Si un país cambia su horario, una hora futura guardada en UTC quedaría corrida. Lo que ya ocurrió sí se guarda en UTC, porque ese momento no cambia |
| El formato de pantalla es `dd/mm/aaaa` y `dd/mm/aaaa hh:mm`, solo al mostrar | Nunca se guarda ni se transmite así. Los ejemplos de este documento usan el formato de pantalla para que se lean fácil |

---

## 3. Orden de construcción

El orden lo fijan las dependencias, no la importancia del negocio. Una tabla no puede escribirse antes que aquellas a las que referencia.

Cada bloque pertenece a un contexto delimitado, que es la unidad de código: `location`, `iam`, `organization`, `vehiclecatalog`, `fleet`, `procurement` y `shared`. El mapa de contextos, con quién puede llamar a quién y la estructura de paquetes, se desarrolla en un documento aparte, como propuesta; lo que este diccionario no fija no está decidido. La regla que lo gobierna es una sola: un contexto usa a otro únicamente a través de su fachada, y nunca consulta ni escribe sus tablas; la única excepción son los modelos de lectura declarados como tales, que solo leen.

| Bloque | Contenido | Depende de | Estado |
|---|---|---|---|
| 1 | Geografía y localización: `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses`, `tax_id_types` | — | **Cerrado** |
| 2 | Seguridad: `users`, `user_access_suspensions`, `roles`, `permissions`, `role_permissions` | — (`roles.company_id` espera al 3) | **Cerrado** |
| 3 | Empresas: `companies`, `commercial_brands`, `branches`, `locations` | 1, 2 | **Cerrado** |
| 3b | Colaboradores y accesos: `employees`, `user_roles`, `advisor_assignments` | 2, 3 | **Cerrado** |
| 4 | Catálogos de flota: `vehicle_types`, `vehicle_brands`, `vehicle_models`, `colors`, `equipment_types` | 3 | **Cerrado** |
| 5 | Terceros: `suppliers` | 3 | **Cerrado** — se absorbió en el bloque 7 |
| 6 | Vehículo, estados y transiciones | 4 | **Cerrado** |
| 7 | Compras: proveedores, tipos y plantillas de orden, órdenes, facturas, líneas por VIN, notas de crédito | 6 | **Cerrado** |
| 8 | Pagos y saldo con proveedores | 7 | **Cerrado** |
| 9 | Habilitación: documentos, equipamiento, llaves, seguros | 6 | **Cerrado** |
| 10 | Fondos y garantía mobiliaria | 6 | **Cerrado** |
| 11 | Bitácora general de cambios | 6 | **Cerrado** |
| 12 | Ventas y contratos | 6, 11 | Fuera del alcance actual |

La columna *Estado* se refiere al diseño, no al código.

**Orden de implementación**

El código se construye en este orden. Respeta las dependencias del cuadro anterior, con cuatro ajustes que se explican debajo.

| Paso | Bloque | Contexto | Qué incluye | Datos que hacen falta | Código |
|---|---|---|---|---|---|
| 0 | 2 · Seguridad, en parte, y el núcleo común | `shared`, `iam` | Cuentas, roles, sesiones y asignaciones reducidas | — | Construido; lo que falta está en §5.13 |
| 1 | 1 · Geografía y localización | `location` | Monedas, países con sus escalones, divisiones territoriales, direcciones y tipos de documento tributario | Archivo oficial de divisiones territoriales (ubigeo del INEI) | Construido para Perú: V24 carga el ubigeo del INEI, edición 2022. Colombia y México se completan antes de su primera empresa (D-52) |
| 2 | 3 · Empresas | `organization` | Empresas, marcas comerciales, sedes y ubicaciones; `roles.company_id` | Razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales | En curso: empresas, marcas comerciales, sedes y ubicaciones construidas; falta `roles.company_id` |
| 3 | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | Colaboradores; asignaciones por empresa, sede y marca; bloqueos de cuenta; permisos; dar acceso; registro público con perfil | Colaboradores reales y la matriz de permisos por rol | Pendiente |
| 4 | 4 · Catálogos de flota | `vehiclecatalog` | Tipos de vehículo, marcas y modelos, colores, equipamiento y a qué tipos aplica | Modelos reales, conciliados entre los dos catálogos que advierte §7.3 | Pendiente |
| 5 | 6 · Vehículo y estados | `fleet` | Estados, caminos y motivos, vehículos, historial de estados, kilometraje | — | Pendiente |
| 6 | 11 · Bitácora de cambios | `shared` | `change_log` | — | Pendiente |
| 7 | 7 · Compras, que incluye el 5 | `procurement` | Proveedores, tipos y plantillas de orden, órdenes, facturas y notas de crédito | Proveedores | Pendiente |
| 8 | 8 · Pagos | `procurement` | Pagos, a qué se aplican y qué unidades cubren, saldo a favor | — | Pendiente |
| 9 | 9 · Habilitación | `fleet`, `shared` | Documentos y sus tipos, equipamiento instalado, llaves, impuesto vehicular, cargas de archivos | — | Pendiente |
| 10 | 10 · Fondos | `fleet` | Fondos y cesiones | — | Pendiente |
| 11 | Reportes | Modelo de lectura | Indicadores calculados (§10.5) y el exportable de stock | — | Pendiente |
| — | 12 · Ventas y contratos | — | Contratos y asesoras | — | Fuera del alcance actual |

Los cuatro ajustes:

1. **La bitácora va apenas después del vehículo**, no al final. Depende solo del bloque 6, y desde ahí ya hay cambios delicados —corregir una placa, un VIN o el monto de una factura— que tienen que quedar registrados.
2. **`advisor_assignments` se construye con el bloque 12**, aunque figure en el 3b: apunta a los contratos, que son de ventas.
3. **El bloque 2 se termina en el paso 3.** Los permisos hacen falta antes de las operaciones de negocio de los bloques 6 y 7, como registrar una factura o dar de alta una unidad.
4. **Desde el paso 5 hay dos ramas independientes:** compras y pagos (pasos 7 y 8), y habilitación y fondos (pasos 9 y 10). Se pueden alternar según la prioridad del negocio; fondos usa las cargas de archivos que llegan con la habilitación.

Un paso se puede construir sin sus datos, con datos de ejemplo; los reales se cargan después.

**A qué contexto pertenece cada bloque**

| Bloque | Contexto |
|---|---|
| 1 · Geografía y localización | `location` |
| 2 · Seguridad | `iam` |
| 3 · Empresas | `organization` |
| 3b · Colaboradores y accesos | `organization` e `iam` |
| 4 · Catálogos de flota | `vehiclecatalog` |
| 6 · Vehículo y estados | `fleet` |
| 7 · Compras | `procurement` |
| 8 · Pagos | `procurement` |
| 9 · Habilitación | `fleet` |
| 10 · Fondos | `fleet` |
| 11 · Bitácora y cargas de archivos | `shared` |

Los contextos `fleet` y `procurement` comparten el vehículo y hablan de él distinto: para el primero es una unidad física que se mueve y se habilita; para el segundo, una línea de comprobante con un VIN y un monto. Ninguno consulta las tablas del otro.

`employees` se documenta en §5.3 junto con las cuentas, pero se crea después de `companies` porque depende de ella.

**No hay dependencia circular entre `users` y `companies`:** el usuario no pertenece a una empresa, así que se crea antes. Lo que depende de ambas es `user_roles`, que va al final. Lo mismo pasa con `roles.company_id`: los roles del sistema lo tienen vacío, así que `roles` se crea con el bloque 2 y esa columna se agrega con el bloque 3, cuando ya existe `companies`. El único caso de arranque —el primer usuario, que nadie creó— se resuelve con la semilla de §5.9.

---

## 4. Bloque 1 · Geografía y localización

El bloque reúne los datos maestros que comparten todas las empresas y la localización de cada país: cómo se divide su territorio y con qué documentos se identifican ante su Estado las empresas y las personas (D-44).

### 4.1 `currencies` — Monedas
Las monedas con las que la empresa compra, factura o cobra.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `iso_code` | CHAR(3) UNIQUE | No | Las tres letras con que se conoce la moneda en todo el mundo: `PEN` soles, `USD` dólares, `COP` pesos colombianos | Si no hay un código único, la misma moneda termina escrita de cinco formas —`S/`, `SOLES`, `Soles`— y los reportes dejan de cuadrar |
| `name` | VARCHAR(50) | No | El nombre completo de la moneda: Sol peruano | Es lo que la persona lee cuando elige la moneda en pantalla |
| `symbol` | VARCHAR(5) | No | El signo con que se escribe el monto: `S/`, `$` | Aparece impreso en las órdenes de compra que se envían al proveedor y en los reportes |
| `decimal_places` | SMALLINT | No | Cuántos decimales usa esa moneda. Normalmente dos | El peso colombiano no usa céntimos. Si el sistema le pone dos decimales a todo, las compras en Colombia salen descuadradas por centavos y encontrar de dónde viene la diferencia cuesta horas |
| `is_active` | BOOLEAN | No | Indica si la moneda se puede seguir usando en operaciones nuevas | Permite dejar de usar una moneda sin borrar las facturas antiguas emitidas en ella |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | Las monedas no llevan autor: son un catálogo general, igual que las marcas o las sedes (D-42) |

**Relaciones.** Nadie le apunta desde arriba; ella es referenciada por `countries` (moneda sugerida del país) y por `companies` (moneda de sus libros). Más adelante la usarán facturas, pagos y notas de crédito, porque ningún monto se guarda sin su moneda.

**Ejemplo**

| id | iso_code | name | symbol | decimal_places | is_active |
|---|---|---|---|---|---|
| 1 | PEN | Sol peruano | S/ | 2 | Sí |
| 2 | USD | Dólar estadounidense | $ | 2 | Sí |
| 3 | COP | Peso colombiano | $ | 0 | Sí |
| 4 | MXN | Peso mexicano | $ | 2 | Sí |

El peso colombiano con cero decimales es el caso que obliga a tener esa columna.

**Reglas de negocio**

- El código de tres letras es el que manda: ningún monto se guarda con la moneda escrita a mano.
- Una moneda no se borra nunca. Si deja de usarse se desactiva, para no romper las facturas antiguas emitidas en ella.
- La cantidad de decimales la fija la moneda, no el sistema: todos los cálculos y reportes la respetan.
- Una moneda tiene de 0 a 4 decimales.
- Los decimales se pueden corregir mientras ningún monto use la moneda. Desde el primer documento que la usa quedan fijos: cambiarlos redondearía de otra forma los montos ya registrados.

### 4.2 `countries` — Países
Los países donde la empresa opera o puede operar. Casi toda la información del sistema se separa por país.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `iso_alpha_2` | CHAR(2) UNIQUE | No | Las dos letras con que se identifica al país en el mundo: `PE`, `CO`, `MX` | Es el código que piden los bancos, las pasarelas de pago y cualquier sistema externo. Si no se guarda, cada integración inventa el suyo y hay que traducir a mano |
| `iso_alpha_3` | CHAR(3) UNIQUE | No | Las tres letras del mismo país: `PER`, `COL`, `MEX` | Es el formato que exigen aduanas y bancos en los papeles de importación, que es justo por donde entran los vehículos |
| `name` | VARCHAR(100) | No | El nombre del país como lo lee una persona: Perú | Va separado del código para que nadie busque por el texto escrito, que cambia según quién lo escriba |
| `calling_code` | VARCHAR(6) | No | El código telefónico del país: `+51` para Perú | Sirve para completar los teléfonos que alguien escribe sin el código. No sirve para lo contrario —adivinar el país por el número—, porque el `+1` lo comparten más de veinte países |
| `default_currency_id` | BIGINT FK → `currencies` | No | La moneda que el sistema propone cuando se registra algo en ese país | Ahorra elegirla en cada registro. Es solo una sugerencia: en Perú también se compra en dólares con normalidad |
| `default_timezone` | VARCHAR(50) | No | La zona horaria del país, con su nombre IANA (`America/Lima`). Es el punto de partida al crear una sede nueva | La hora real se guarda en cada sede, porque hay países con varias: México tiene cuatro. Si se usara una sola por país, los vencimientos y los cortes del día saldrían corridos varias horas |
| `tax_name` | VARCHAR(20) | Sí | Cómo se llama el impuesto general en ese país: IGV, IVA | Se imprime en comprobantes y reportes. En Colombia nadie entiende "IGV" |
| `tax_rate` | NUMERIC(5,4) | Sí | Tasa vigente del impuesto general | Es solo el valor que se propone al registrar una factura. La tasa que manda es la que queda guardada en cada comprobante |
| `requires_postal_code` | BOOLEAN | No | Indica si en ese país el código postal es obligatorio en una dirección | En México es un dato clave del domicilio fiscal; en Perú casi nadie lo usa. El formulario tiene que pedirlo o no según el país |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | Los países no llevan autor: son un catálogo general (D-42) |

**Relaciones.** Apunta a `currencies`. Le apuntan `geo_division_levels` (cómo se divide), `geo_divisions` (sus divisiones), `addresses`, `tax_id_types` (qué documentos emite) y `companies` (qué empresa está constituida ahí).

**Ejemplo**

| id | iso_alpha_2 | iso_alpha_3 | name | calling_code | default_currency_id | default_timezone | requires_postal_code |
|---|---|---|---|---|---|---|---|
| 1 | PE | PER | Perú | +51 | 1 · PEN | America/Lima | No |
| 2 | CO | COL | Colombia | +57 | 3 · COP | America/Bogota | No |
| 3 | MX | MEX | México | +52 | 4 · MXN | America/Mexico_City | Sí |

**Reglas de negocio**

**No hay un campo que diga si el país está operativo.** Se responde preguntando si tiene alguna empresa activa. Guardarlo además como casilla crearía dos lugares para apagar lo mismo, con la posibilidad de que se contradigan: la casilla en "no operativo" y tres empresas facturando. Dejar de operar en un país se hace desactivando sus empresas.

**Los dos códigos ISO nombran un mismo país de la norma ISO 3166** (D-47). `PE` va con `PER`, nunca con `PRI`, que es Puerto Rico; un código que no nombra ningún país, como `PU`, se rechaza al escribirlo. Es el error que más cuesta: el código llega a los archivos del banco y a los papeles de aduana, donde nadie lo corrige a mano.

**Los códigos ISO se corrigen mientras nada cuelga del país** (D-47). Un error de tipeo que se nota después de registrar el país se corrige mientras el país no tenga escalones, direcciones ni tipos de documento tributario. Tampoco puede tener divisiones ni empresas: una división cuelga de un escalón y una empresa necesita un tipo de documento de su país. Desde entonces los códigos pueden haber salido en un archivo o una integración, y quedan fijos. Es el mismo criterio que los decimales de una moneda (§4.1). Dentro del sistema nada se rompe al corregirlos: las demás tablas apuntan al país por su `id`, no por su código.

### 4.3 Cómo se relacionan las tres tablas de geografía

`countries` dice qué países existen, `geo_division_levels` cómo se divide cada uno, y `geo_divisions` guarda las divisiones concretas.

**Paso 1 — Perú declara sus escalones**

| País | Escalón | Se llama |
|---|---|---|
| PE | 1 | Departamento |
| PE | 2 | Provincia |
| PE | 3 | Distrito |

**Paso 2 — Se cargan las divisiones**

| id | País | Pertenece a | Escalón | Código oficial | Nombre |
|---|---|---|---|---|---|
| 10 | PE | *(nada)* | 1 | 15 | Lima |
| 45 | PE | 10 | 2 | 1501 | Lima |
| 312 | PE | 45 | 3 | 150108 | Chorrillos |

**Paso 3 — La dirección guarda solo el escalón más bajo**

Una dirección en Chorrillos guarda la división 312. El departamento y la provincia se obtienen subiendo por la cadena: 312 → 45 → 10.

Colombia declara solo dos escalones —Departamento y Municipio— y México tres con otros nombres. Con columnas fijas, en Colombia la tercera quedaría vacía y en México las tres tendrían el nombre equivocado.

### 4.4 `geo_division_levels` — Cómo se divide cada país
Cada país divide su territorio de forma distinta y con nombres distintos. Perú usa Departamento → Provincia → Distrito; Colombia, Departamento → Municipio; México, Estado → Municipio → Colonia. Esta tabla guarda esa estructura.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | El país al que corresponde esta forma de dividirse | — |
| `level` | SMALLINT | No | El escalón de la división: 1 es el más grande, 2 el que sigue, y así hasta 4 | Permite que cada país tenga la cantidad de escalones que realmente usa, en vez de obligar a todos a tener tres |
| `name` | VARCHAR(50) | No | Cómo se llama ese escalón en ese país: Departamento, Provincia, Distrito | Un usuario en Colombia debe leer "Municipio" donde uno en Perú lee "Provincia". Con nombres fijos, el formulario pediría datos que allá no existen |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | Los escalones no llevan autor: son un catálogo general (D-42) |

**Relaciones.** Apunta a `countries`. Le apunta `geo_divisions`, que no puede registrar un escalón que el país no haya declarado aquí.

**Ejemplo**

| país | escalón | se llama |
|---|---|---|
| PE | 1 | Departamento |
| PE | 2 | Provincia |
| PE | 3 | Distrito |
| CO | 1 | Departamento |
| CO | 2 | Municipio |

**Reglas de negocio**

- La combinación de país y número de escalón identifica cada fila: un país no puede declarar dos veces el mismo escalón.
- Se admiten hasta cuatro escalones. Perú usa tres y Colombia dos.
- Cada país declara sus escalones antes de cargar sus divisiones. Sin esta tabla, la siguiente no se puede llenar.
- **Los escalones se declaran en orden, sin saltos** (D-45). El siguiente escalón es siempre el que sigue al último declarado. Un escalón 3 sin escalón 2 no podría tener divisiones, porque cada división cuelga de una del escalón inmediato superior.
- **Un escalón declarado no se borra ni se retira** (D-45). Se corrige su nombre. Las divisiones y las direcciones dependen de él, y retirarlo dejaría divisiones colgando de un escalón que ya no existe.
- Los escalones forman parte del país: se guardan y se validan junto con él (D-45).
- Que los escalones vayan en orden lo protege solo la aplicación: la base impide declarar dos veces el mismo escalón (`UNIQUE` de país y escalón), pero no un salto.

### 4.5 `geo_divisions` — Departamentos, provincias y distritos
Cada división concreta del territorio. Se carga desde las listas oficiales de cada país: INEI en Perú, DANE en Colombia, INEGI en México. La de Perú es la edición 2022 del INEI, con 25 departamentos, 196 provincias y 1 891 distritos, y la cargó la migración V24 tal como la publica el INEI. Las de Colombia y México se cargan antes de registrar la primera empresa de cada país (D-52).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | El país al que pertenece | Impide, por ejemplo, que un distrito peruano quede colgando de un departamento colombiano |
| `parent_id` | BIGINT FK → `geo_divisions` | Sí | A qué división mayor pertenece: el distrito apunta a su provincia, la provincia a su departamento | Al guardar solo el escalón inmediato superior, el sistema puede agrupar por departamento, por provincia o por distrito sin repetir el dato tres veces |
| `level` | SMALLINT | No | En qué escalón está: departamento, provincia o distrito | Impide registrar un cuarto escalón en un país que solo tiene tres |
| `code` | VARCHAR(20) | No | El código oficial que el Estado le asigna a cada división. En Perú se llama UBIGEO: Chorrillos es `150108` | Hoy el distrito se escribe a mano y aparece de varias formas. Con el código oficial se ordena de una vez, y en ese cruce se descubre cuántas versiones del mismo nombre hay en los archivos actuales |
| `name` | VARCHAR(120) | No | El nombre de la división | — |
| `is_active` | BOOLEAN | No | Indica si la división sigue vigente | En Perú se crean y se fusionan distritos. Permite retirar uno sin dañar las direcciones ya registradas que lo mencionan |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | Muestra cuándo se cargó o se desactivó una división. No lleva autor: es un catálogo general (D-42) |

**Relaciones.** Apunta a `countries` y a sí misma (`parent_id`, la división del escalón superior). Le apunta `addresses`. El ejemplo de carga está en §4.3.

**Reglas de negocio**

- El código oficial no se repite dentro del mismo país.
- Una división no puede pertenecer a otra de un país distinto: un distrito peruano nunca cuelga de un departamento colombiano.
- No se puede registrar un escalón que el país no haya declarado en la tabla anterior.
- Solo las divisiones del primer escalón no pertenecen a ninguna otra.
- Una división que se fusiona o desaparece se desactiva, nunca se borra: las direcciones ya registradas siguen apuntando a ella.

**Qué protege la base y qué la aplicación.** La base impide que una división cuelgue de otra de un país distinto: la llave hacia la división superior incluye el país. También impide usar un escalón que el país no declaró. Desde V25 impide también que la división superior sea de otro escalón que el inmediato: la llave hacia la superior incluye su escalón, que la base calcula como el de la división menos uno en la columna generada `parent_level` (D-54). Un distrito no puede colgar directo del departamento, ni desde la aplicación ni desde una carga en SQL de una lista oficial. La base impide además que una división del primer escalón tenga superior y que una de otro escalón no lo tenga (`geo_divisions_root_parent_ck`).

### 4.6 `addresses` — Direcciones
Una dirección física, separada en sus partes. La usan empresas, sedes, proveedores y clientes.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | El país de la dirección | Es el único dato geográfico que siempre se conoce. El distrito puede faltar, por ejemplo en la dirección de un proveedor extranjero |
| `geo_division_id` | BIGINT FK → `geo_divisions` | Sí | El distrito, municipio o colonia donde está la dirección. Obligatorio cuando el país de la dirección tiene sus divisiones cargadas (D-53) | Guardando solo el más específico, el sistema deduce solo la provincia y el departamento, sin que nadie tenga que escribirlos |
| `street_name` | VARCHAR(180) | No | La calle o avenida escrita completa: Av. Defensores del Morro | Se guarda junto con el "Av." o "Jr." porque separarlos genera tres formas de escribir lo mismo —Av., Avenida, AV— sin ninguna utilidad |
| `street_number` | VARCHAR(20) | Sí | El número de la puerta | Se guarda como texto y no como número porque existen casos reales como "S/N" o "Km 12.5". Si fuera numérico, habría que inventar un valor para poder grabar |
| `unit` | VARCHAR(50) | Sí | Interior, departamento, piso, oficina, manzana o lote | En Lima buena parte de las direcciones tiene interior. Sin un campo propio termina pegado al número de puerta y ya no se puede separar |
| `reference` | TEXT | Sí | Cómo llegar: frente a qué queda, de qué color es el portón | Sin referencia, quien va a entregar un vehículo o a cobrar no encuentra el lugar. En la práctica es tan necesario como la calle |
| `postal_code` | VARCHAR(12) | Sí | El código postal, obligatorio solo en los países que lo exigen | En México forma parte del domicilio fiscal |
| `created_at` | TIMESTAMPTZ | No | Cuándo se registró la dirección | — |
| `updated_at` | TIMESTAMPTZ | No | Cuándo se modificó por última vez | — |

**Relaciones.** Apunta a `countries` y a `geo_divisions`. Le apuntan `companies` (domicilio fiscal), `branches`, `locations` y `purchase_order_templates` (el domicilio que imprime cada plantilla); más adelante, proveedores y clientes.

**Ejemplo**

| id | country_id | geo_division_id | street_name | street_number | unit | reference |
|---|---|---|---|---|---|---|
| 1 | 1 · PE | 312 · Chorrillos | Av. Defensores del Morro | 1234 | Of. 302 | Frente al grifo, portón azul |
| 2 | 1 · PE | 318 · Ate | Av. Nicolás Ayllón | Km 12.5 | — | Al costado del almacén de repuestos |

La segunda fila muestra por qué el número de puerta se guarda como texto.

**Reglas de negocio**

**Las direcciones no se comparten entre entidades.** Cada empresa, sede o cliente tiene su propia fila aunque el texto sea idéntico. Si dos compartieran la misma y una se mudara, al corregir la dirección se estaría mudando también a la otra sin que nadie se entere.

**Una mudanza crea una fila nueva; una corrección modifica la existente (D-8).** La diferencia importa porque un contrato firmado apunta a una dirección concreta:

| Situación | Qué se hace | Efecto en los documentos ya emitidos |
|---|---|---|
| Se escribió mal el número de puerta | Se corrige la fila | Ninguno: siempre fue el mismo lugar |
| El cliente se mudó | Se crea una fila nueva y el cliente pasa a apuntar a ella | El contrato sigue apuntando a la dirección anterior, que es la que tenía el día de la firma |

Así el contrato conserva el domicilio con el que se firmó sin necesidad de copiarlo dentro, y la dirección actual del cliente queda siempre disponible. Es la opción de menor costo: no agrega campos ni tablas, solo exige que la pantalla distinga una corrección de una mudanza.

**La división de una dirección es del mismo país y del escalón más bajo que ese país declaró.** Es lo que permite deducir la provincia y el departamento subiendo por la cadena (§4.3). Una dirección de Perú no puede apuntar a un municipio colombiano ni quedarse en la provincia cuando el país declara distritos. Para una dirección nueva, la división debe estar activa. Que la división sea del mismo país lo impide la base, porque la llave hacia la división incluye el país; que sea del escalón más bajo y que esté activa lo valida la aplicación.

**La división es obligatoria donde el país tiene sus divisiones cargadas (D-53).** Una empresa, una sede o una ubicación en Perú no se registra sin su distrito: sin él, el sistema no puede deducir la provincia ni el departamento. Solo puede faltar en la dirección de un país cuyas divisiones no están cargadas, como la de un proveedor extranjero. Lo valida la aplicación, porque depende de qué países tienen su lista cargada.

**La dirección está en el país de su dueño.** Quien la escribe no elige el país: lo fija el dueño, que es una empresa, una sede o una ubicación. Una empresa peruana no puede tener su domicilio fiscal en Colombia. La base lo exige en empresas y sedes, porque la llave hacia `addresses` incluye el país; en las demás, lo valida la aplicación.

### 4.7 `tax_id_types` — Tipos de documento tributario
Con qué documento se identifica una empresa o una persona ante el Estado, en cada país. Es localización del país, igual que sus escalones territoriales, y por eso pertenece a este bloque (D-44).

**Por qué es una tabla y no un dato fijo:** no existe un documento tributario universal. El nombre neutro que se usa internacionalmente es **TIN** (*Tax Identification Number*, el término de la OCDE); "RUT" es simplemente como se le dice en Chile y Colombia, igual que RUC en Perú y Ecuador, NIT en Colombia, RFC en México, CUIT en Argentina o CNPJ en Brasil. La forma correcta de identificar a una empresa en un sistema que opera en varios países es la combinación **país + tipo de documento + número**.

El único identificador realmente mundial de empresas es el **LEI** (una clave de 20 caracteres que emite un organismo internacional). No todas lo tienen: se paga y se renueva cada año. Va como dato opcional de la empresa, no aquí.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | El país que emite ese documento | Un mismo nombre puede existir en dos países con reglas distintas |
| `code` | VARCHAR(20) | No | La sigla con que se conoce: RUC en Perú, NIT en Colombia, RFC en México | Es lo que se imprime en las facturas y lo que la persona reconoce en pantalla |
| `name` | VARCHAR(100) | No | El nombre completo: Registro Único de Contribuyentes | Aparece en formularios y documentos legales |
| `applies_to` | VARCHAR(10) | No | Si ese documento lo tienen empresas, personas o ambos: `empresa`, `persona`, `ambos` | En Perú el RUC lo tienen ambos, el DNI solo las personas. Una empresa no se registra con un DNI |
| `length` | SMALLINT | Sí | Cuántos caracteres debe tener el número | Permite avisar en el momento si alguien escribe un RUC de 10 dígitos en lugar de 11 |
| `has_check_digit` | BOOLEAN | No | Indica si el número trae un dígito de control que permite verificar que esté bien escrito | Un RUC mal escrito que llega a una factura se convierte en un problema tributario. Con el dígito de control se detecta al momento de registrarlo |
| `is_active` | BOOLEAN | No | Si el documento se puede elegir en registros nuevos | Permite retirar un documento sin tocar las empresas, proveedores y colaboradores que ya lo usan |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general, como el resto del bloque (D-42) |

**Relaciones.** Apunta a `countries`. Le apuntan `companies`, `suppliers`, `employees` y el emisor de `purchase_order_templates`, que se identifican con la pareja tipo de documento más número. Más adelante también los clientes, que usan DNI en vez de RUC.

**Ejemplo**

| País | Sigla | Nombre | Aplica a | Longitud | Dígito de control |
|---|---|---|---|---|---|
| PE | RUC | Registro Único de Contribuyentes | ambos | 11 | Sí |
| PE | DNI | Documento Nacional de Identidad | persona | 8 | Sí |
| PE | CE | Carné de Extranjería | persona | — | No |
| PE | PAS | Pasaporte | persona | — | No |
| CO | NIT | Número de Identificación Tributaria | ambos | 9 | Sí |
| CO | CC | Cédula de Ciudadanía | persona | 10 | No |
| MX | RFC | Registro Federal de Contribuyentes | ambos | 12 | Sí |
| MX | CURP | Clave Única de Registro de Población | persona | 18 | Sí |

**Reglas de negocio**

- La sigla no se repite dentro del mismo país y no se edita: se corrige el nombre, el alcance, la longitud y el dígito de control.
- Un tipo de documento no se borra: se desactiva. Los registros que ya lo usan siguen siendo válidos.
- Quien registra una empresa, un proveedor o un colaborador elige un tipo activo, de su mismo país y que aplique a lo que registra. Cuando el tipo declara una longitud, el número la respeta.
- Las longitudes de Colombia y México del ejemplo están por confirmar: el RFC tiene 12 caracteres para empresas y 13 para personas, con un solo tipo que aplica a ambos; falta decidir si el dígito de control del NIT va dentro del número, y la CC no siempre tiene 10 dígitos. Se resuelve antes de registrar la primera empresa de cada país (D-52).

**Nota técnica:** aquí se guardan la longitud y la existencia de dígito de control en vez de un patrón de validación, por el mismo motivo por el que se eliminó el del código postal. La diferencia es que estos dos son datos estables —la longitud del RUC no cambia por decreto— y el cálculo del dígito de control vive en el programa, no en la base.

### 4.8 Candidatos del bloque 1

Campos que hoy no entran porque nadie los usa todavía, con la condición que los admitiría.

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `iso_numeric` | `countries` | Que los papeles de importación pidan el código numérico del país |
| `locale` | `countries` | Que se defina el formato de los reportes según quién los recibe, y resulte que el país es el criterio correcto |
| `iso_subdivision_code` | `geo_divisions` | Un reporte o una integración que use códigos internacionales de región |
| `latitude` / `longitude` | `addresses` | Que se implemente ruteo de entregas o gestión de cobranza en campo |
| `street_type` | `addresses` | Que se defina como lista cerrada por país y alguien realmente lo use para filtrar |
| `exchange_rates` (tabla) | — | El primer documento que convierte montos: la factura de compra. Se construye justo antes del paso 7, con el diseño de abajo (D-46) |
| `purchase_exchange_rate_type` | `countries` | La misma condición: qué tipo de tasa propone cada país para sus compras |
| `is_group_currency` | `currencies` | El primer reporte consolidado de varias empresas. Se construye con el paso 11, con el diseño de abajo (D-46) |

#### Diseño previsto: tipos de cambio

Hoy cada documento guarda el tipo de cambio que usó (§2.2), pero nada lo propone, así que alguien lo escribiría a mano en cada factura. Los ERP que operan en varios países guardan la tasa de cada día y la proponen según la fecha del documento.

`exchange_rates`, en el bloque 1:

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `from_currency_id` | BIGINT FK → `currencies` | No | La moneda que se convierte: USD | — |
| `to_currency_id` | BIGINT FK → `currencies` | No | La moneda a la que se convierte: PEN | — |
| `rate_date` | DATE | No | El día al que corresponde la tasa | Una tasa vale por un día de calendario, no por un momento (§2.6) |
| `rate_type` | VARCHAR(10) | No | `compra`, `venta` u `oficial` | La SBS de Perú publica compra y venta; la TRM de Colombia y el FIX de México publican una sola tasa oficial |
| `rate` | NUMERIC(12,6) | No | Cuántas unidades de la moneda de destino vale una de la de origen | La misma precisión que el `exchange_rate` de facturas, notas y pagos |
| `source` | VARCHAR(20) | No | De dónde salió: `SBS`, `BCRP`, `BANREP`, `BANXICO`, `manual` | Es lo primero que pregunta un auditor sobre una tasa |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la cargó y cuándo | `SYSTEM` cuando la trajo la carga automática |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la corrigió por última vez | — |

- Las dos monedas, la fecha y el tipo no se repiten. Las dos monedas son distintas y la tasa es mayor que cero.
- La tasa de cada día la carga un proceso automático desde la fuente oficial del país, a nombre de `SYSTEM`. Contabilidad la corrige a mano cuando hace falta, y la corrección queda en la bitácora.
- El documento propone la tasa más reciente de su fecha o anterior, porque las fuentes no publican los fines de semana, y la copia en su propia columna. Corregir una tasa después no cambia los documentos que ya la copiaron.
- Qué tipo de tasa propone cada país para sus compras lo dice `countries.purchase_exchange_rate_type`. En Perú la práctica es la de venta de la SBS a la fecha de emisión del comprobante; lo confirma Contabilidad antes del paso 7.

#### Diseño previsto: moneda de grupo

- `currencies.is_group_currency`: BOOLEAN, `NOT NULL DEFAULT FALSE`, con un índice único parcial para que solo una moneda la tenga. Es USD (D-46).
- Los reportes consolidados convierten cada monto desde la moneda funcional de su empresa a la de grupo, con la tasa de la fecha de su documento. Sirve para reportes de gestión; una consolidación contable con tasas de cierre y promedio queda fuera, porque este sistema no lleva libros contables.

---

## 5. Bloque 2 · Seguridad: usuarios, roles y permisos

### 5.1 Cinco conceptos que conviene no mezclar

| Concepto | Qué es | Tabla |
|---|---|---|
| **Colaborador** | La persona que trabaja para la empresa, use o no el sistema | `employees` |
| **Usuario** | La cuenta con que un colaborador inicia sesión | `users` |
| **Rol** | Un conjunto de permisos que se asigna a una cuenta: *Administrador de empresa*, *Asesor de ventas*, *Contabilidad* | `roles` |
| **Permiso** | Una acción concreta que se puede autorizar: registrar una factura, anular una unidad | `permissions` |
| **Asignación** | Qué rol tiene un usuario **y sobre qué alcance**: empresa, sede o todo el sistema | `user_roles` |

**Sobre la palabra "perfil"** (decisión D-4, cerrada): se refiere a los **datos personales** de la persona —nombre, documento, teléfono, foto, puesto, fecha de ingreso—, no al rol. Esos datos van en `employees` (§5.3), no en `users`: la cuenta guarda solo lo necesario para entrar (D-10).

**Vocabulario** (decisión D-39). **Rol** es el conjunto de permisos que se asigna a una cuenta, con su alcance; en la base y en el código se llama igual, `role`. **Puesto** es el trabajo del colaborador —asesor de ventas, mecánico, contador—: es informativo y no da permisos. **Perfil** son los datos personales del colaborador (D-4). Este documento no usa la palabra "cargo", porque se confundía con los dos primeros.

**Los clientes no van en esta tabla.** Cuando exista la aplicación para clientes, quienes entren por ahí tendrán su propia tabla de identidades, separada de la del personal interno. Tres razones: un error asignando un rol podría darle acceso interno a un cliente; la tabla pasaría de decenas de filas a miles, con reglas de contraseña y recuperación distintas; y todo el modelo de roles y alcances no le aplica a alguien que solo ve lo suyo.

Y sobre distinguir al que ya es cliente del que solo descargó la aplicación: **no se guarda como un campo**. Es cliente quien tiene o tuvo un contrato, de modo que se deduce. Guardarlo como estado obligaría a acordarse de cambiarlo al firmar, y el primer olvido deja a alguien con contrato marcado como prospecto. Se define en el bloque 12.

**Un usuario no pertenece a una empresa.** Es una identidad global y su acceso se expresa como asignaciones de rol. Al revés —`users.company_id`— el primer usuario sería imposible, porque tendría que pertenecer a una empresa que todavía no existe, y habría que duplicar la cuenta de quien trabaje para dos empresas del grupo.

### 5.2 La secuencia real de arranque

| Paso | Qué ocurre | Quién lo hace |
|---|---|---|
| 1 | Se cargan los catálogos: monedas, países, divisiones geográficas | Migración inicial |
| 2 | Se crean los permisos y los roles del sistema | Migración inicial |
| 3 | Se crea el usuario `SYSTEM` y el primer superadministrador, con una contraseña provisional | Migración inicial (`SYSTEM`) y la aplicación al arrancar (superadministrador) |
| 4 | El superadministrador crea las empresas, una por país | Usuario |
| 5 | Crea las sedes de cada empresa | Usuario |
| 6 | Se registra como colaborador de la empresa para la que trabaja y vincula su cuenta a ese perfil | Usuario |
| 7 | Registra al administrador de cada empresa como colaborador, le da acceso y le asigna el rol `ADMIN_EMPRESA` sobre esa empresa | Usuario |
| 8 | El administrador de cada empresa registra a sus colaboradores y da acceso a quienes usan el sistema, con los roles que corresponden a su puesto y su sede | Usuario |

Los pasos 1 a 3 no los puede ejecutar nadie desde la aplicación —no hay con qué iniciar sesión todavía—. Esto no es un rodeo: es la única forma de resolver que el primer usuario no tiene creador.

Los catálogos, los roles y `SYSTEM` van en la migración. El primer superadministrador, en cambio, lo crea la aplicación al arrancar (decisión D-34), y solo mientras no exista ningún superadministrador activo. Toma el correo y la contraseña provisional de las variables de entorno `IAM_BOOTSTRAP_ADMIN_EMAIL` e `IAM_BOOTSTRAP_ADMIN_PASSWORD`, así esa contraseña no queda escrita en el repositorio. La cuenta figura creada por `SYSTEM` y debe cambiar la contraseña en su primer ingreso.

**Es la única persona que nace sin perfil de colaborador**, porque todavía no existe ninguna empresa donde registrarla. Mientras no se vincule a su perfil (paso 6) puede configurar el sistema —empresas, sedes, catálogos y su propio perfil—, pero no puede dar acceso a nadie ni otorgar roles. Así, desde el primer acceso que se otorga, cada acción queda atribuida a una persona con nombre, documento y puesto.

**Casos del arranque.** Si faltan las variables, la aplicación arranca igual, sin superadministrador, y lo advierte en su registro de eventos. Si el correo indicado ya pertenece a otra cuenta, la aplicación no arranca hasta que se corrija. El rol de ese primer superadministrador figura otorgado por `SYSTEM`.

**Registro público, apagado por defecto** (decisión D-32). Además del acceso que da un administrador, una persona puede registrarse desde la pantalla de inicio de sesión. Se enciende o se apaga con la variable de entorno `IAM_PUBLIC_SIGN_UP_ENABLED`, y viene apagado. Como toda cuenta pertenece a un colaborador (D-38), el registro no crea solo la cuenta:

- **Registra el perfil y la cuenta en la misma operación.** La persona elige su empresa —y su sede, si corresponde— y completa los mismos datos obligatorios que exige el registro de un colaborador (§5.3), además de su correo y su contraseña. Si algo falla, no queda ni el colaborador ni la cuenta.
- **Si su documento ya está registrado en esa empresa, el registro se rechaza.** Esa persona ya tiene perfil, y el acceso se lo da un administrador desde ahí. Si el registro pudiera vincularse a un perfil existente, cualquiera que conozca un número de documento podría apropiarse del perfil de otra persona.
- **Si el correo ya tiene cuenta, el registro se rechaza.** Esa persona ya tiene acceso por otra empresa del grupo: un administrador de esta empresa la registra como colaboradora y vincula el perfil a su cuenta.
- **Nace sin roles.** Registrarse no da acceso a nada: los roles se los asigna un administrador después, con las mismas reglas de §5.5.
- **La cuenta y el perfil figuran creados por `SYSTEM`**, igual que cualquier otro proceso automático, para que `created_by` siga respondiendo quién los originó.
- **No tiene que cambiar su contraseña**, porque la eligió la propia persona; la obligación de §5.4 es para las claves provisionales que entrega un administrador.
- **Para elegir empresa y sede**, el formulario las lista sin sesión iniciada, solo con su nombre, y solo mientras el registro está encendido.
- **La pantalla de inicio de sesión sabe si el registro está encendido** por una ruta pública que solo responde eso. Si alguien intenta registrarse con el registro apagado, la API rechaza la operación como prohibida.

La consecuencia que hay que respetar al programar: con el registro encendido, cualquiera puede tener una sesión iniciada, y también puede haber sesiones sin ningún rol por otras razones, como la de alguien a quien se le retiraron todos. Por eso ninguna operación de negocio puede conformarse con que el usuario haya iniciado sesión: tiene que exigir un rol o un permiso.

### 5.3 `employees` — Colaboradores

Las personas que trabajan para la empresa, usen o no el sistema. Un mecánico del taller es colaborador y puede no tener cuenta; una asesora es colaboradora y sí la tiene.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa para la que trabaja | — |
| `branch_id` | BIGINT FK → `branches` | Sí | Sede donde trabaja | Vacío para quien no está asignado a una sede fija |
| `first_name` | VARCHAR(100) | No | Nombres | — |
| `last_name_paternal` | VARCHAR(80) | No | Apellido paterno | Se guarda separado porque los documentos formales lo exigen así |
| `last_name_maternal` | VARCHAR(80) | Sí | Apellido materno | Opcional: un colaborador extranjero puede no tenerlo |
| `document_type_id` | BIGINT FK → `tax_id_types` | No | Tipo de documento de identidad | Usa el mismo catálogo de documentos, filtrado a los que aplican a personas |
| `document_number` | VARCHAR(20) | No | Número del documento | Es como se identifica a la persona en actas y contratos |
| `position` | VARCHAR(80) | Sí | Puesto: asesor de ventas, mecánico, contador | Es informativo y distinto del rol que da permisos: dos personas con el mismo puesto pueden tener accesos distintos |
| `phone` | VARCHAR(30) | Sí | Teléfono | — |
| `photo_path` | TEXT | Sí | Foto del colaborador, opcional | El archivo se guarda fuera de la base, igual que los documentos del vehículo; aquí queda su ubicación. Sirve para mostrarla junto a sus registros |
| `user_id` | BIGINT FK → `users` | Sí | La cuenta con que esta persona entra al sistema | Vacío para quien no usa el sistema, como un mecánico. **La referencia va en este sentido a propósito**: quien trabaja para dos empresas del grupo tiene dos filas de colaborador y una sola cuenta, y ambas pueden apuntar a ella |
| `hire_date` | DATE | No | Fecha de ingreso | — |
| `termination_date` | DATE | Sí | Fecha de salida. Vacío = sigue trabajando | Un colaborador que se va no se borra: sus ventas y sus registros siguen siendo suyos |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | — |
| `created_by` | BIGINT FK → `users` | No | Quién registró al colaborador | `SYSTEM` cuando llegó por el registro público (§5.2); si no, el administrador que lo registró |

**Relaciones.** Apunta a `companies`, `branches`, `tax_id_types` y `users` (dos veces: su cuenta y quién lo registró). Le apuntan las asignaciones de asesora y las instalaciones de equipamiento.

**Ejemplo**

| id | company_id | nombres | apellidos | puesto | ingreso | salida |
|---|---|---|---|---|---|---|
| 1 | 1 | Tere | Ramírez Quispe | Contabilidad | 02/01/2024 | — |
| 2 | 1 | Alejandro | Vargas Ríos | Operaciones | 15/03/2025 | — |
| 3 | 1 | Luis | Chávez | Mecánico | 01/06/2025 | — |

El tercero no tiene cuenta en el sistema y existe igual.

**Reglas de negocio**

- El documento de identidad no se repite dentro de la misma empresa.
- Un colaborador no se borra: se registra su fecha de salida. Sus ventas, sus registros y los roles que otorgó siguen siendo explicables.
- El puesto es un texto informativo y **no da permisos**. Los permisos vienen del rol asignado en §5.7.
- La misma persona trabajando para dos empresas del grupo son dos colaboradores, uno por empresa, con una sola cuenta de acceso.
- Una cuenta se vincula a lo sumo a un colaborador por empresa.
- **Toda cuenta pertenece a un colaborador** (decisión D-38). Las cuentas no se crean solas. Hay dos caminos, y en ambos la cuenta queda vinculada a su colaborador en la misma operación: un administrador **da acceso** a un colaborador ya registrado, o la persona usa el **registro público**, si está encendido, que crea el colaborador y la cuenta juntos (§5.2). Si la persona ya tiene cuenta porque trabaja para otra empresa del grupo, dar acceso vincula el nuevo colaborador a esa misma cuenta en vez de crear otra. Un colaborador puede existir sin cuenta y recibir acceso cuando lo necesite. Las únicas cuentas sin colaborador son `SYSTEM`, que no es una persona, y el primer superadministrador hasta que se vincula (§5.2).
- **Para tener un rol en una empresa o una sede**, la cuenta debe estar vinculada a un colaborador activo —sin fecha de salida— de esa empresa. Un rol de alcance sistema exige un colaborador activo en alguna empresa.
- **Cuando un colaborador sale**, se registra su fecha de salida y en la misma operación se retiran los roles de su cuenta en esa empresa, con motivo `baja_colaborador` (§5.7). Si a la cuenta no le queda ningún colaborador activo, se bloquea con motivo `baja` (§5.4), lo que cierra sus sesiones. Quien trabaja para dos empresas y deja una conserva su cuenta y sus roles en la otra. El sistema impide registrar la salida si con ella no quedaría ningún superadministrador activo (§5.9).

### 5.4 `users` — Cuentas de acceso

Solo lo necesario para entrar. Quién es la persona vive en `employees`: toda cuenta pertenece a un colaborador, salvo `SYSTEM` y el primer superadministrador hasta que se vincula (§5.3).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `email` | VARCHAR(255) UNIQUE | No | El correo con que entra. No se repite. Se guarda sin espacios y en minúsculas | Una cuenta por persona, no por empresa: quien trabaja para dos empresas del grupo entra una sola vez y su historial queda completo en un solo lugar |
| `password_hash` | VARCHAR(255) | No | La contraseña guardada de forma cifrada e ilegible | Nunca se guarda tal como se escribió. Ni siquiera quien administra el sistema puede verla |
| `must_change_password` | BOOLEAN | No | Obliga a cambiar la contraseña la primera vez que entra | Cuando se crea una cuenta se entrega una clave provisional. Sin esta marca, esa clave se queda para siempre |
| `password_updated_at` | TIMESTAMPTZ | Sí | Cuándo cambió la contraseña por última vez | Permite exigir el cambio cada cierto tiempo y detectar claves antiguas |
| `mfa_secret` | VARCHAR(255) | Sí | Clave del segundo factor, si lo activó | Un sistema con datos financieros debería poder exigir un segundo paso al entrar |
| `last_login_at` | TIMESTAMPTZ | Sí | Última vez que entró | Con esto se detectan cuentas abandonadas, que son la puerta de entrada más común a un sistema |
| `created_at` | TIMESTAMPTZ | No | Cuándo se creó la cuenta | — |
| `created_by` | BIGINT FK → `users` | Sí | Quién la creó | Primera pregunta de toda revisión de seguridad: quién le dio acceso a quién |
| `updated_at` | TIMESTAMPTZ | No | Última modificación | — |

**Relaciones.** Apunta a sí misma (`created_by`). Le apuntan `employees`, `user_roles`, `user_access_suspensions` y prácticamente todas las tablas a través de `created_by`, `updated_by` y campos equivalentes.

**Ejemplo**

| id | email | must_change_password |
|---|---|---|
| 1 | system@millennium.ky | No |
| 2 | tere@millennium.ky | No |
| 3 | alejandro@millennium.ky | Sí |

La primera no corresponde a ninguna persona: existe para figurar como responsable de las cargas masivas y los procesos automáticos. Luis, el mecánico, no aparece aquí porque no usa el sistema.

**Quién es la persona detrás de una cuenta** se responde desde `employees`, que es quien guarda la referencia. Así alguien empleado por dos empresas del grupo tiene dos filas de colaborador —una por empresa, como exige la regla— y ambas apuntan a la misma cuenta. En las dos cuentas sin colaborador, la pantalla muestra el correo.

**Reglas de negocio**

El usuario **nunca se borra**: su nombre está en las facturas, habilitaciones y cambios de estado que registró. Borrarlo dejaría todo ese historial sin autor.

**Contraseñas.** Tienen entre 8 y 72 caracteres, sin reglas de composición; 72 es el máximo que procesa el cifrado que se usa (BCrypt). La nueva tiene que ser distinta de la actual. Al iniciar sesión, un correo que no existe y una contraseña equivocada reciben la misma respuesta, para no revelar qué cuentas existen.

**Mientras la contraseña es provisional**, la cuenta solo puede consultar quién es y cambiar su contraseña. Sus roles no valen hasta que la cambie.

#### Suspender, dar de baja y volver a activar

Una sola fecha de baja no alcanza. Pasan tres cosas distintas y las tres tienen que poder deshacerse:

- Alguien sale de **vacaciones o licencia** y se le bloquea la cuenta mientras no está.
- Alguien **se va de la empresa** y se le retira el acceso.
- Alguien que se había ido **vuelve**, o su cuenta se bloqueó por un incidente y se aclara.

Con un único campo de baja, reactivar significa borrarlo, y con eso desaparece el rastro de que estuvo bloqueado. Eso choca con la regla que ya seguimos en los accesos: el permiso se retira, no se borra, porque hay que poder explicar quién podía entrar al sistema el día que ocurrió una operación.

Por eso los bloqueos se guardan como **períodos** en una tabla aparte, igual que el GPS de un vehículo o la asesora de un cliente:

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `user_id` | BIGINT FK → `users` | No | La persona a la que se le bloqueó el acceso | — |
| `period` | TSTZRANGE | No | Desde cuándo hasta cuándo estuvo bloqueada, con fecha y hora. Abierto = sigue bloqueada | Permite responder quién tenía acceso en un momento pasado. Se guarda con hora, no solo fecha, porque un bloqueo y una reactivación pueden ocurrir el mismo día y hay que saber si la persona tenía acceso cuando se registró una operación a media tarde |
| `reason` | VARCHAR(20) | No | `vacaciones`, `licencia`, `baja`, `seguridad` | Distingue una ausencia temporal de una salida de la empresa. Sin esto, en un reporte de personal las vacaciones se cuentan como bajas |
| `notes` | TEXT | Sí | Detalle cuando haga falta | — |
| `created_by` | BIGINT FK → `users` | No | Quién ordenó el bloqueo | Misma exigencia que en los cambios de estado: fecha, usuario y motivo |
| `created_at` | TIMESTAMPTZ | No | Cuándo se registró el bloqueo | — |
| `closed_by` | BIGINT FK → `users` | Sí | Quién reactivó la cuenta. Vacío mientras siga bloqueada | Sin este dato se sabría quién quitó el acceso pero no quién lo devolvió, que es la mitad de la pregunta en una revisión de seguridad |
| `closed_at` | TIMESTAMPTZ | Sí | Cuándo se reactivó | — |

Cómo queda cada caso:

| Situación | Qué se hace | Al volver |
|---|---|---|
| Vacaciones | Se crea un período con motivo `vacaciones` | Se cierra el período con la fecha de regreso |
| Salida de la empresa | Se crea un período abierto con motivo `baja` | Si la persona regresa, se cierra ese período y vuelve a entrar con la misma cuenta y su historial completo |
| Bloqueo por seguridad | Período abierto con motivo `seguridad` | Se cierra cuando se aclara |

Un usuario está activo si **no tiene ningún período de bloqueo abierto**. No hay un campo que lo diga, para que no pueda contradecir a la tabla.

El sistema impide que dos bloqueos de la misma persona se crucen en el tiempo, incluso en el pasado.

#### Qué queda registrado, exactamente

Cada bloqueo y cada reactivación deja una fila que no se borra ni se reutiliza. Un ejemplo de la historia de una persona:

| # | Período | Motivo | Ordenado por | Reactivado por |
|---|---|---|---|---|
| 1 | 10 feb 2026 → 24 feb 2026 | vacaciones | Tere | Tere |
| 2 | 03 may 2026 09:15 → 03 may 2026 16:40 | seguridad | Tere | Janover |
| 3 | 01 jul 2026 → *(abierto)* | baja | Tere | — |

Con eso se responden las preguntas que hace una auditoría:

- *¿Tenía acceso el 15 de febrero?* No: la fila 1 cubre esa fecha.
- *¿Y el 3 de mayo a las 11 de la mañana, cuando se registró esa factura?* No: la fila 2 cubre ese momento. Por eso el período lleva hora.
- *¿Quién le devolvió el acceso ese día?* Janover, a las 16:40.
- *¿Está activo hoy?* No: la fila 3 sigue abierta.

Lo que **no** cubre esta tabla es la corrección posterior de una de estas filas —si alguien edita el motivo o mueve una fecha—. Eso lo registra la bitácora general de cambios del sistema, que documenta qué campo se modificó, su valor anterior y su valor nuevo (bloque 11). Son dos cosas distintas: esta tabla guarda lo que pasó con el acceso; la bitácora guarda lo que alguien hizo con el registro.

**Relaciones.** Apunta a `users` tres veces: la persona bloqueada, quien ordenó el bloqueo y quien reactivó. No le apunta nadie.


**Se pueden programar por adelantado.** Nada impide crear un bloqueo con fecha de inicio futura para unas vacaciones ya aprobadas: la persona conserva el acceso hasta que llegue esa fecha y lo pierde sola. No hace falta que nadie se acuerde el día que empieza.

**Una ventaja concreta de bloquear la cuenta en vez de retirarle los roles.** Cuando alguien sale de vacaciones se podrían quitar sus roles y devolvérselos al volver. En la práctica alguien olvida devolver alguno, o devuelve uno de más. Bloqueando la cuenta, los roles quedan intactos: la persona vuelve y encuentra exactamente los mismos accesos que tenía.

**Al bloquear una cuenta se cierran también sus sesiones abiertas**, o la persona seguiría dentro hasta que vencieran. Como las sesiones viven en el servidor (§5.12), cerrarlas es borrarlas y el efecto es inmediato.

#### Los datos personales van en `employees`, no aquí

Guardar el nombre, el documento o el teléfono dentro de la cuenta parece lo más simple, pero no encaja con cómo trabaja la empresa (D-10):

- Hay personas con datos y sin cuenta: el mecánico del taller es colaborador y no entra al sistema.
- Hay cuentas sin persona: `SYSTEM` firma las cargas masivas y los procesos automáticos.
- Quien trabaja para dos empresas del grupo tiene una sola cuenta y dos registros de colaborador, uno por empresa, cada uno con su puesto y su fecha de ingreso.

Por eso los datos personales viven en `employees` (§5.3) y `users` guarda solo lo necesario para entrar.

### 5.5 `roles` — Roles
Un rol con su nivel de acceso. Define qué puede hacer alguien, no sobre qué empresa.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | Sí | La empresa que creó el rol. **Vacío = disponible para todas** | Los roles que trae el sistema y los que crea el superadministrador sirven en cualquier país. Uno creado por el administrador de Perú no debe aparecerle a Colombia |
| `code` | VARCHAR(40) | No | Nombre corto e invariable del rol: `ADMIN_EMPRESA` | El sistema lo usa internamente, así que se puede cambiar cómo se llama el rol en pantalla sin que nada deje de funcionar |
| `name` | VARCHAR(80) | No | El nombre del rol como se lee: Administrador de empresa | Es lo que ve quien asigna un acceso |
| `description` | TEXT | Sí | Explicación de qué hace este rol | Evita que dentro de un año nadie sepa en qué se diferencian dos roles parecidos |
| `scope_level` | VARCHAR(10) | No | Hasta dónde llega el rol: `SYSTEM` (todo el sistema), `COMPANY` (una empresa) o `BRANCH` (una sede) | Es lo que impide darle a un asesor de una tienda el mismo alcance que al administrador general |
| `is_system` | BOOLEAN | No | Marca los roles que trae el sistema y no se pueden borrar ni modificar | Protege al administrador general: si alguien lo borrara por error, nadie podría volver a administrar el sistema |

**Relaciones.** Apunta a `companies` cuando el rol pertenece a una empresa concreta. Le apuntan `role_permissions` (qué permisos incluye) y `user_roles` (a quién se le asignó).

**Ejemplo**

| code | name | empresa | alcance | del sistema |
|---|---|---|---|---|
| `SUPERADMIN` | Superadministrador | *(todas)* | Todo el sistema | Sí |
| `ADMIN_EMPRESA` | Administrador de empresa | *(todas)* | Una empresa | Sí |
| `ASESOR_VENTAS` | Asesor de ventas | *(todas)* | Una sede | Sí |
| `JEFE_PATIO` | Jefe de patio | Millennium Perú | Una sede | No |

El último lo creó el administrador de Perú: no le aparece a Colombia.

#### Crear roles nuevos

Los seis roles que trae el sistema (cuadro al final de esta sección) son el punto de partida, no la lista definitiva: los puestos cambian y hay que poder agregar roles sin tocar el programa. Crear uno es insertar una fila en `roles` y marcar sus permisos, todo desde la pantalla de administración.

| Quién lo crea | Dónde queda disponible |
|---|---|
| Superadministrador | En todas las empresas (`company_id` vacío) |
| Administrador de una empresa | Solo en esa empresa |

Cinco reglas que gobiernan esto:

**1. No se puede dar lo que no se tiene.** Al crear o editar un rol, solo se pueden marcar permisos que quien lo está haciendo ya posee. Sin esta regla, el administrador de una empresa podría crear un rol con acceso total y asignárselo: quedaría convertido en superadministrador sin que nadie lo autorizara.

**2. La misma regla vale al asignar.** Restringir solo la creación no sirve de nada: si el administrador no puede crear un rol con acceso total pero sí puede asignarse uno existente que ya lo tiene, el resultado es idéntico. Nadie puede otorgar un rol cuyos permisos excedan los suyos.

**3. Los permisos no tienen niveles, son conjuntos.** No existe "un permiso por encima de otro": la comparación es de contenido. El rol nuevo es válido si todos sus permisos están dentro de los de quien lo crea.

**4. El alcance se fija al crear y no cambia.** Si un rol nace para asignarse a nivel de sede, no puede convertirse después en rol de empresa: las asignaciones que ya existen quedarían con datos incoherentes. Para cambiarlo se crea un rol nuevo.

**5. Lo que alguien autorizó sobrevive a su salida.** Si a un administrador se le retiran permisos o deja la empresa, los roles que creó y los accesos que otorgó **siguen siendo válidos**. La autorización fue legítima cuando se dio, igual que un contrato firmado por un gerente que después renunció. Lo que sí deja de poder hacer es seguir editándolos: de ahí en adelante se le aplica la regla 1 con los permisos que tenga en ese momento, o ninguno si ya salió.

Esto es también la razón de fondo por la que un usuario nunca se borra: si desapareciera, los roles y accesos que autorizó quedarían sin responsable y no habría forma de explicar quién los aprobó.

Como los accesos otorgados no caducan solos, el control que los mantiene sanos es revisarlos cada cierto tiempo. El modelo lo permite sin campos adicionales: cada asignación guarda quién la otorgó y cuándo, así que se puede listar todo lo vigente y depurar lo que ya no corresponde.

Los roles marcados como propios del sistema no se pueden editar ni borrar, aunque quien lo intente tenga todos los permisos.

**Límite de lo que la base puede garantizar.** La consistencia entre la empresa del rol y la empresa de la asignación no se puede exigir con una llave foránea, porque el campo admite vacío para los roles generales. Se resuelve con una validación programada que rechaza asignar un rol de una empresa dentro de otra. Las reglas 1 a 3 también viven en la aplicación: la base no sabe qué permisos tiene quien está ejecutando la operación.

**Reglas de negocio**

Roles que trae el sistema:

| code | name | Alcance |
|---|---|---|
| `SUPERADMIN` | Superadministrador | Todo el sistema |
| `ADMIN_EMPRESA` | Administrador de empresa | Una empresa |
| `OPERACIONES` | Operaciones | Una empresa |
| `CONTABILIDAD` | Contabilidad | Una empresa |
| `ASESOR_VENTAS` | Asesor de ventas | Una sede |
| `TALLER` | Taller | Una sede |

### 5.6 `permissions` y `role_permissions` — Permisos
Cada acción concreta que se puede autorizar por separado. Un rol es un conjunto de permisos.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(60) UNIQUE | No | Nombre corto de la acción, con el módulo adelante: `compras.registrar_factura` | Al llevar el módulo adelante, se pueden otorgar todos los permisos de un área de una sola vez |
| `module` | VARCHAR(30) | No | A qué parte del sistema pertenece: stock, compras, seguridad | Ordena la pantalla donde se asignan. Con cien permisos sueltos, nadie encuentra el que busca |
| `description` | VARCHAR(150) | No | Qué habilita exactamente este permiso | Es lo que lee quien asigna. Sin explicación se otorgan permisos por el nombre y se termina dando de más |

**Relaciones.** `permissions` no apunta a nadie: es un catálogo fijo que se carga con el sistema. `role_permissions` es la tabla intermedia que une roles con permisos.

**Ejemplo de `permissions`**

| id | code | module | description |
|---|---|---|---|
| 1 | `compras.registrar_factura` | compras | Registrar una factura de compra |
| 2 | `compras.registrar_factura_antigua` | compras | Registrar una factura de más de 20 días |
| 3 | `stock.crear_vehiculo` | stock | Dar de alta una unidad |
| 4 | `stock.corregir_placa` | stock | Corregir una placa ya registrada |
| 5 | `stock.anular_vehiculo` | stock | Anular una unidad |
| 6 | `seguridad.crear_usuario` | seguridad | Crear cuentas de acceso |
| 7 | `stock.ver_costos` | stock | Ver el costo de compra y los descuentos del proveedor |
| 8 | `clientes.ver_cartera_completa` | clientes | Ver la cartera de todos los asesores, no solo la propia |
| 9 | `compras.gestionar_plantillas` | compras | Crear y editar las plantillas de orden de compra |

**Ejemplo de `role_permissions`**

| Rol | Permisos que incluye |
|---|---|
| Operaciones | 1, 3 |
| Contabilidad | 1, 2 |
| Administrador de empresa | 1, 2, 3, 4, 5, 6 |

Contabilidad puede registrar facturas atrasadas y Operaciones no: es la excepción que pide la guía funcional, resuelta con un permiso en vez de con una condición escrita en el programa.

**Reglas de negocio**

`role_permissions` solo une las dos tablas: indica qué permisos tiene cada rol.

**Por qué los permisos van uno por uno y no se resuelven con el rol.** La guía funcional exige que una factura con más de veinte días de antigüedad solo pueda registrarla un administrador. Si eso se programa como "solo si el rol es administrador", el día que contabilidad necesite hacerlo hay que modificar el programa y volver a instalarlo. Como permiso separado, es marcar una casilla. Lo mismo aplica a corregir una placa ya registrada, anular una unidad o cargar el archivo del SAT: son excepciones controladas y cada una debe poder darse por separado.

### 5.7 `user_roles` — Quién tiene qué rol y sobre qué
Aquí se responde la pregunta real: *esta persona, con este rol, sobre qué empresa o sede*.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `user_id` | BIGINT FK → `users` | No | A qué persona se le da el rol | — |
| `role_id` | BIGINT FK → `roles` | No | Qué rol se le da | Una persona puede tener varias filas aquí: uno o muchos roles a la vez |
| `scope_level` | VARCHAR(10) | No | Copia del alcance del rol: `SYSTEM`, `COMPANY` o `BRANCH` | Guardarlo aquí permite que el sistema verifique solo que los datos sean coherentes, sin depender de que el programa lo recuerde |
| `company_id` | BIGINT FK → `companies` | Sí | Sobre qué empresa aplica. Vacío si el rol alcanza todo el sistema | Es lo que hace que "Administrador" signifique administrador de una empresa y no de todas |
| `branch_id` | BIGINT FK → `branches` | Sí | Sobre qué sede aplica. Vacío salvo que el rol sea de sede | Permite que un asesor vea solo el local donde trabaja |
| `commercial_brand_id` | BIGINT FK → `commercial_brands` | Sí | Sobre qué marca aplica. **Vacío significa todas las marcas de esa empresa** | Permite que un asesor vea solo Lima Bikes y no Lizzo. Para darle dos marcas de tres, se crean dos filas |
| `granted_at` | TIMESTAMPTZ | No | Cuándo se le dio el acceso | — |
| `granted_by` | BIGINT FK → `users` | Sí | Quién se lo dio | Primera pregunta de toda revisión de accesos: quién autorizó esto |
| `expires_at` | TIMESTAMPTZ | Sí | Fecha prevista de caducidad. Vacío significa que no caduca | Para accesos temporales, como el de un auditor externo que necesita entrar un mes. Sin esto, el acceso queda vivo hasta que alguien se acuerde de retirarlo |
| `revoked_at` | TIMESTAMPTZ | Sí | Cuándo se le quitó. Vacío significa que lo tiene todavía | El acceso se retira, no se borra: hay que poder explicar qué permisos tenía una persona el día que hizo una operación |
| `revocation_reason` | VARCHAR(20) | Sí | Por qué se retiró: `retiro` (alguien lo quitó), `caducidad` (venció), `baja_colaborador` (la persona salió de esa empresa) o `baja_cuenta` (la cuenta se bloqueó con motivo `baja`). Obligatorio al retirar | Distingue un acceso que alguien decidió quitar de uno que venció solo o que cayó por una salida. Sin él, los cuatro casos se ven iguales al revisar |

**Relaciones.** Es la tabla que une casi todo el bloque: apunta a `users` (dos veces: quién recibe el acceso y quién lo otorgó), a `roles`, a `companies`, a `branches` y a `commercial_brands`. No le apunta nadie.


#### Un usuario con acceso a dos empresas

El rol define **qué puede hacer** una persona; la asignación define **sobre qué**. Por eso el acceso a dos empresas no se configura en el rol: se otorga con dos filas en esta tabla.

| Persona | Rol | Sobre |
|---|---|---|
| Tere | Contabilidad | Lizzo |
| Tere | Contabilidad | Lima Bikes |

Tere hace lo mismo en ambas, pero es un acceso por empresa. Si mañana deja de llevar la contabilidad de una, se retira esa fila y la otra queda intacta.

Nada obliga a que el rol sea el mismo en las dos:

| Persona | Rol | Sobre |
|---|---|---|
| Alejandro | Operaciones | Lima Bikes |
| Alejandro | Asesor de ventas | Lizzo — sede San Isidro |

Hay entonces tres formas de dar acceso, y conviene no confundirlas:

| Se necesita | Cómo se hace | Qué pasa si mañana se crea una empresa nueva |
|---|---|---|
| Una empresa | Una fila | No la ve |
| Algunas empresas | Una fila por cada una | No la ve hasta que se le agregue una fila |
| Todas las empresas | Un rol de alcance global, una sola fila | La ve automáticamente |

Esa última columna es la diferencia real entre "todas las empresas de hoy" y "alcance global": el alcance global incluye las empresas que todavía no existen. Para un superadministrador eso es lo correcto; para un contador externo, casi nunca.

#### Limitar el acceso a una marca

Lizzo y Lima Bikes son marcas de la misma empresa, no empresas distintas (§6.3). Aun así, el acceso se puede limitar a una de ellas: la asignación lleva una marca opcional.

| Persona | Rol | Empresa | Marca |
|---|---|---|---|
| Asesor de motos | Asesor de ventas | Millennium Perú | Lima Bikes |
| Jefe comercial | Operaciones | Millennium Perú | *(vacío = todas)* |
| Coordinadora | Operaciones | Millennium Perú | Lizzo |
| Coordinadora | Operaciones | Millennium Perú | Lima Bikes |

Dejar la marca vacía significa **todas las marcas**, incluidas las que se creen después. Para dar dos marcas de tres, se crean dos filas, igual que con las empresas. Y vale la misma advertencia: dos filas explícitas no incluyen automáticamente una marca nueva; el campo vacío sí.

La marca es una dimensión aparte del alcance, no un escalón más. Por eso se puede combinar: un asesor puede estar limitado a la vez a una sede y a una marca.

**Lo que no se agregó y por qué.** Podría crearse un "grupo de empresas" para asignar varias de una sola vez. Con dos o tres empresas no compensa: agrega una tabla, una pantalla para mantenerla y una pregunta más en cada revisión de accesos. Vale la pena recién cuando sean muchas y se repita siempre el mismo conjunto.

**Reglas de negocio**

Lo que el sistema impide por sí solo, sin depender del programa:

| Regla | Qué evita |
|---|---|
| El alcance guardado debe coincidir con el del rol | Que alguien invente un alcance distinto al que el rol permite |
| Rol de todo el sistema: sin empresa ni sede. De empresa: con empresa y sin sede. De sede: con ambas | Un superadministrador limitado a una tienda, o un asesor sin tienda asignada |
| La sede debe pertenecer a la empresa asignada | Dar acceso a la sede de otra empresa |
| La marca debe pertenecer a la empresa asignada | Dar acceso a una marca que no es de esa empresa |
| Un rol de alcance global no puede limitarse a una marca | Un superadministrador que en realidad solo ve una parte del sistema |
| El mismo rol no se puede dar dos veces sobre lo mismo, mientras siga vigente | Asignaciones duplicadas que después nadie sabe cuál retirar |

Lo que valida la aplicación, porque la base no puede verlo en una sola fila: para otorgar un rol de alcance empresa o sede, la cuenta debe estar vinculada a un colaborador activo de esa empresa; para uno de alcance sistema, a un colaborador activo de alguna empresa (§5.3). La única excepción es el rol del primer superadministrador, que se otorga al arrancar (§5.2).

### 5.8 Cómo se resuelven los permisos

1. **Los permisos se suman.** Con varios roles, la persona puede hacer todo lo que permite cualquiera de ellos. No existen reglas que quiten permisos. Un sistema con permitir y quitar obliga a definir cuál manda, y a partir de ahí nadie puede saber qué puede hacer alguien sin revisar el programa.
2. **El alcance baja, nunca sube.** Un rol sobre una empresa alcanza todas sus sedes. Uno sobre todo el sistema alcanza todas las empresas. Nunca al revés.
3. **Consecuencia práctica:** como los permisos se suman, dar un rol amplio "por un rato" no se compensa dando además uno restringido. Los roles deben ser estrechos y combinables.
4. **Los accesos retirados dejan de valer**, pero se conservan para poder reconstruir quién podía hacer qué en una fecha pasada.
5. **Un acceso que caduca se cierra como si lo hubieran retirado**: el sistema le pone la fecha de retiro con motivo `caducidad`. Tiene que ser así y no quedar abierto con la fecha vencida, porque de lo contrario esa persona no podría recibir de nuevo el mismo acceso —el sistema lo vería como duplicado del que sigue abierto— y el auditor externo que vuelve el año siguiente quedaría fuera.

### 5.9 Semilla inicial y protección del acceso

El usuario `SYSTEM` no puede iniciar sesión y figura como responsable de toda carga masiva y de todo proceso automático, incluida la actualización por archivo del SAT. Sin él, esos registros quedarían sin autor.

**Qué pasa automáticamente al registrar una baja.** Cuando se crea un bloqueo con motivo `baja`, el sistema revoca en la misma operación todas las asignaciones activas de esa persona, con motivo `baja_cuenta`. Son dos acciones distintas y las dos hacen falta: bloquear la cuenta corta el acceso en el momento, y retirar los roles evita que, si esa cuenta se reactiva algún día, la persona recupere de golpe todo lo que tenía. Automatizarlo es lo que impide que el segundo paso dependa de que alguien lo recuerde.

Nada se pierde al revocar: las asignaciones quedan con su fecha de retiro y se puede seguir explicando qué accesos tenía esa persona en cualquier fecha pasada.

Las vacaciones y las licencias funcionan al revés a propósito: ahí los roles **no** se tocan. Si se retiraran, al volver habría que devolvérselos y alguien olvidaría uno o daría uno de más. La persona regresa con exactamente los mismos accesos que tenía.

**Protección necesaria:** el sistema impide retirar el último superadministrador activo, ya sea quitándole el rol, bloqueando su cuenta o registrando la salida de su colaborador. Sin ninguno, no hay forma de volver a entrar sin tocar la base a mano.

**El primer superadministrador no puede dar acceso a nadie ni otorgar roles** hasta vincularse a su perfil de colaborador (§5.2).

### 5.10 Candidatos del bloque 2

Ninguno: todos los campos que estaban en espera se incorporaron. `last_login_at`, `password_updated_at` y `mfa_secret` entraron a `users`; `phone` y `photo_path`, a `employees`; `expires_at`, a las asignaciones; y los roles propios por empresa quedaron resueltos al cerrar D-7.

Los datos personales de las personas viven en `employees`, no aquí.

### 5.11 Asesores: cómo se conecta el usuario con el cliente

Confirmado: cada cliente tiene una asesora asignada y esa asignación puede cambiar. Eso trae dos consecuencias para el modelo, y conviene dejarlas escritas ahora aunque el bloque de ventas se construya después.

**La asesora es un usuario, no un texto.** El contrato apunta al usuario, no guarda el nombre escrito. Si se guardara como texto, al mes aparecerían tres formas de escribir a la misma persona y sería imposible contar cuántas unidades colocó. Tampoco hace falta un campo nuevo para marcar quién es asesora: son los usuarios que tienen el rol `ASESOR_VENTAS` activo en esa empresa o sede. Cuando alguien deja de serlo, se le retira el rol; sus asignaciones pasadas siguen apuntando a su usuario, porque los usuarios nunca se borran.

**Reasignar significa que esto es una historia, no una columna.** Si la asesora fuera un solo campo que se sobrescribe, al reasignar se perdería quién atendía antes. Y con eso se pierde la respuesta a preguntas que el negocio sí hace: cuántas unidades colocó alguien el mes pasado, o quién atendía a este cliente cuando dejó de pagar.

Por eso son **dos datos distintos**, no uno:

| Dato | Cambia | Para qué sirve |
|---|---|---|
| Quién originó la venta | Nunca | Comisiones, medición de desempeño, historia de la operación |
| Quién atiende hoy | Cada vez que se reasigna | Saber a quién reclamar, a quién dirigir una consulta, cómo repartir la cartera |

Mezclarlos en un solo campo es el error clásico: al reasignar por vacaciones o por renuncia, la venta cambia retroactivamente de dueño y los reportes de meses cerrados dejan de cuadrar.

#### `advisor_assignments` — anticipo del bloque de ventas

Sigue el mismo patrón que el GPS o los contratos de un vehículo: varias filas en el tiempo, una sola vigente.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `contract_id` | BIGINT FK → `contracts` | No | El contrato que se está atendiendo | Cuelga del contrato y no del cliente porque el contrato es el único que tiene fechas propias. Un cliente con dos unidades tomadas con seis meses de diferencia puede tener dos asesoras, y eso es información correcta, no un problema |
| `advisor_employee_id` | BIGINT FK → `employees` | No | La asesora asignada | Apunta al colaborador y no a su cuenta de acceso: la comisión es de la persona, y sigue siéndolo aunque pierda el acceso al sistema |
| `period` | DATERANGE | No | Desde cuándo hasta cuándo atendió. Abierto = sigue atendiendo | Permite responder quién atendía en una fecha concreta del pasado |
| `reason` | VARCHAR(50) | Sí | Por qué se reasignó: renuncia, vacaciones, redistribución de cartera | Sin el motivo, un cambio de asesora a mitad de mes no se puede explicar después |
| `assigned_by` | BIGINT FK → `users` | No | Quién hizo la reasignación | Es la misma exigencia que el flujograma pone para los cambios de estado: fecha, hora, usuario y motivo |
| `assigned_at` | TIMESTAMPTZ | No | Cuándo se registró | — |

**Relaciones.** Apunta a `employees` (la asesora), a `users` (quien hizo la reasignación) y a `contracts`.

**Reglas de negocio**

- El sistema impide que dos asesoras figuren atendiendo el mismo contrato en fechas que se crucen, incluso si el cruce ocurre en el pasado. Es la misma garantía que evita que un vehículo tenga dos GPS instalados a la vez.
- Quien originó la venta queda fijo en el contrato y no cambia nunca: es lo que alimenta las comisiones y la medición de desempeño.
- Quien atiende hoy cambia con cada reasignación, y cada cambio deja su fila con motivo y autor.
- Una reasignación no altera ventas ya cerradas. Si la asignación colgara del cliente, cambiar de asesora reescribiría el origen de operaciones de meses atrás.
- Cuando un contrato se anula y la unidad vuelve al stock, nace un contrato nuevo con su propia asesora. El historial del anterior queda intacto.

#### `contracts` — Contratos (esbozo)

Se define completo en el bloque 12. Aquí va el mínimo para que las asignaciones de asesora tengan a qué apuntar.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad arrendada | Un vehículo tiene varios contratos a lo largo de su vida, pero nunca dos a la vez |
| `customer_id` | BIGINT FK | No | El cliente | Se define en el bloque 12 |
| `legacy_cod` | VARCHAR(20) | Sí | El COD que tenía en Sigma | Solo referencia histórica. **Sin restricción de unicidad**: las tres reglas del documento de clientes no pueden cumplirse a la vez si el mismo cliente retoma la misma placa |
| `start_date` / `end_date` | DATE | No / Sí | Vigencia del contrato | — |
| `status` | VARCHAR(20) | No | Estado del contrato | Solo uno puede estar activo por unidad |

**Reglas de negocio**

- Dos contratos de la misma unidad no pueden superponerse; sucederse, todos los que hagan falta.
- Solo un contrato activo por unidad. El sistema lo impide, de modo que la condición que el documento de clientes quiere detectar con una alerta no se puede crear.
- Un incumplimiento cierra el contrato con su fecha y motivo. La deuda queda con el cliente; la unidad vuelve al stock sin arrastrar nada.

**Un caso que no cubre la tabla de asignaciones.** Un prospecto que descargó la aplicación y todavía no firmó no tiene contrato, así que no puede tener una asignación aquí. Si el negocio quiere repartir los prospectos entre asesoras para el seguimiento comercial, eso es otra cosa y va en el bloque 12: una mide comisiones, la otra reparte trabajo de venta. Conviene no darles el mismo nombre.


#### El vehículo dura; los contratos pasan

El negocio gira alrededor del vehículo, y el vehículo sobrevive a sus clientes. Una misma unidad pasa por varias manos a lo largo de su vida, y por eso el contrato es una tabla aparte con su propio período.

| Qué ocurre | Qué hace el sistema |
|---|---|
| Un cliente incumple y se resuelve el contrato | El contrato se cierra con su fecha y su motivo. No se borra |
| La unidad vuelve al ciclo comercial | Recorre sus propios estados —recuperación, puesta a punto, disponible— sin arrastrar nada del contrato anterior |
| Aparece un cliente nuevo | Se crea un contrato nuevo, con su propia asesora y su propio período |

**La deuda se queda con la persona; el vehículo se va limpio.** Lo que quedó impago pertenece al contrato cerrado y a su cliente. Si colgara del vehículo, el próximo cliente heredaría el problema del anterior.

Son dos líneas de tiempo sobre la misma placa —la del vehículo y la de sus contratos— y por eso viven en tablas separadas. El sistema solo impide que dos contratos de la misma unidad se superpongan; sucederse, todos los que hagan falta.

**Un dato que sale gratis.** Si la unidad ya tuvo contratos, es semi-nueva. Eso que el documento de Registro de Clientes guarda como "NUEVO / SEMIPREMIUM" no hace falta almacenarlo ni acordarse de cambiarlo: se deduce de si existe algún contrato anterior.

**Dos reglas más**

- Un contrato **no puede quedarse sin asesora**. Al retirar a alguien, el sistema exige el reemplazo en la misma operación: no se admite un período en blanco.
- Todos los asesores usan el sistema, así que todos tienen cuenta. Aun así la asignación apunta al colaborador y no a la cuenta, porque la comisión pertenece a la persona.

### 5.12 Sesiones

Cómo se mantiene iniciada la sesión de alguien (decisión D-33). No es un dato del negocio, pero de esto depende que un bloqueo o un cambio de contraseña surtan efecto en el momento.

**La sesión vive en el servidor, en PostgreSQL.** El navegador solo recibe un identificador opaco dentro de una cookie que el código de la página no puede leer. No hay ninguna credencial guardada en el navegador: un script malicioso inyectado en la página no puede llevarse la sesión, y cerrarla es borrar su fila, sin copias que sigan sirviendo en otro lado.

| Regla | Valor | Por qué importa |
|---|---|---|
| Cierre por inactividad | 30 minutos | Una sesión olvidada en una computadora compartida no queda abierta indefinidamente |
| Duración máxima | 8 horas, aunque se esté usando | Obliga a volver a entrar en cada jornada |
| Cambio de contraseña | Se cierran las demás sesiones de la persona; sigue abierta la que hizo el cambio | Quien conocía la contraseña anterior pierde el acceso en ese momento |
| Bloqueo de la cuenta | Se cierran todas sus sesiones | Ver §5.4 |
| Cerrar sesión | Se borra la sesión en el servidor, no solo en la pantalla | La cookie que quedara en el navegador ya no sirve para nada |
| Roles y contraseña provisional | Se leen de la base en cada operación, no se guardan en la sesión | Un rol otorgado o retirado vale desde la operación siguiente, sin esperar a que la persona vuelva a entrar |

**Protección de la cookie.** Viaja marcada como inaccesible para el código de la página (`HttpOnly`), solo hacia el mismo sitio (`SameSite=Strict`) y, en producción, solo por HTTPS (`Secure`). Como el navegador la envía por sí solo, toda operación que modifica datos exige además un token contra solicitudes falsificadas (CSRF) que únicamente la propia aplicación puede leer.

**Detalles de la implementación.** La cookie se llama `MM_SESSION`. Al iniciar sesión se cambia el identificador de la sesión, para que uno obtenido antes no sirva después. Cerrar sesión es `POST /api/v1/authentication/sign-out`. Las 8 horas se cuentan desde que se creó la sesión. El token CSRF llega en la cookie `XSRF-TOKEN` y vuelve en el encabezado `X-XSRF-TOKEN`, también al iniciar sesión y al registrarse.

**Requisito de despliegue.** La aplicación web y la API se sirven desde el mismo origen —mismo dominio y puerto—, detrás de un proxy inverso. En desarrollo lo resuelve el proxy del servidor de Angular.

**Tablas técnicas.** `spring_session` y `spring_session_attributes` las administra la librería de sesiones (§2.2). `spring_session.principal_name` guarda el correo de la persona —es lo que permite cerrar todas sus sesiones— y por eso admite 255 caracteres, igual que `users.email`. Las sesiones vencidas se borran solas cada minuto.

### 5.13 Lo que el backend implementa hoy

El bloque 2 se construyó antes que `organization`, así que lo que depende de empresas, sedes y colaboradores quedó en una versión provisional. Nada de esto cambia el diseño de este capítulo: es lo que falta para llegar a él.

| Tema | Hoy | Cuándo se completa |
|---|---|---|
| `user_roles` | Se creó antes de su bloque (3b), sin `company_id`, `branch_id` ni `commercial_brand_id`. Solo se asignan roles de alcance sistema | Con el bloque 3 |
| `roles.company_id` | No existe todavía: todos los roles son generales, y `code` es único en todo el sistema | Con el bloque 3, que define también la unicidad por empresa |
| Otorgar un rol | Se exige tener ese mismo rol vigente, porque todavía no hay permisos que comparar (§5.5, reglas 1 a 3) | Cuando existan `permissions` y `role_permissions` |
| Gestión de usuarios | Solo el superadministrador, porque todavía no se pueden acotar por empresa (§2.5) | Con el bloque 3 |
| Colaboradores | No existen: las cuentas se crean solas y el registro público pide solo correo y contraseña, así que D-38 todavía no se aplica | Con el bloque 3b |
| `user_access_suspensions`, `permissions`, `role_permissions` | No existen todavía | Los bloqueos, con el bloque 3b; los permisos, cuando se cargue la matriz de permisos por rol (§14) |
| Retirar un rol y caducidad | No hay operación para retirar un rol ni nada que fije una caducidad, así que las protecciones de §5.5 y §5.9 y el cierre por caducidad de §5.8 todavía no tienen qué vigilar | Con la pantalla de roles y usuarios |

---

## 6. Bloque 3 · Empresas

### 6.1 `tax_id_types` — se trasladó a §4.7

Los tipos de documento tributario son localización del país y pasaron al bloque 1 (D-44). La sección conserva su número para que las referencias a §6.2 y siguientes no cambien.

### 6.2 `companies` — Empresas
La razón social constituida en un país. Es quien compra, factura, tributa y figura como propietaria de los vehículos.

**Decisión cerrada (D-5):** hay **una sola empresa por país**. Lizzo y Lima Bikes no son empresas: son marcas comerciales de esa empresa, y pueden aparecer más con el tiempo (§6.3). Por eso el exportable de stock dice `PROPIETARIO = MILLENNIUM`: el dueño legal es la empresa, la marca es solo cómo se comercializa la unidad.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | País donde está constituida | Una empresa pertenece a un solo país. Expandirse a Colombia crea una empresa nueva, no una sede de la peruana |
| `tax_id_type_id` | BIGINT FK → `tax_id_types` | No | Con qué documento se identifica: RUC, NIT, RFC | El tipo varía por país, así que no se puede asumir "RUC" en el modelo |
| `tax_id` | VARCHAR(20) | No | El número de ese documento, sin guiones ni espacios | Es el identificador de la empresa ante el Estado. Se guarda normalizado para que `10429...` y `10-429...` no se registren como dos empresas |
| `legal_name` | VARCHAR(255) | No | La razón social exacta, como figura en registros públicos | Es la que debe aparecer impresa en comprobantes y contratos. No se puede "arreglar" para que se vea mejor |
| `trade_name` | VARCHAR(150) | No | El nombre con que se la conoce a diario | Es el que se muestra en pantalla, más corto que la razón social |
| `fiscal_address_id` | BIGINT FK → `addresses` | Sí | Domicilio fiscal declarado | Va en facturas y en trámites. Puede faltar mientras la empresa se está registrando |
| `phone` | VARCHAR(30) | Sí | Teléfono principal de la empresa | Es el que se propone como teléfono del emisor en las plantillas de orden de compra (§9.2) |
| `functional_currency_id` | BIGINT FK → `currencies` | No | La moneda en que lleva su contabilidad | Normalmente la del país, pero puede diferir. Determina a qué moneda se convierten las compras en dólares |
| `is_active` | BOOLEAN | No | Si la empresa sigue operando. Por defecto `TRUE` | Permite cerrar una operación sin borrar su historial de compras y contratos |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Cuándo y quién la registró | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Cuándo y quién la modificó por última vez | — |

**Relaciones.** Apunta a `countries`, a `tax_id_types`, a `addresses` (domicilio fiscal) y a `currencies` (moneda de sus libros). Le apuntan `commercial_brands`, `branches`, `locations`, `roles` (los roles propios de esa empresa), `user_roles` (los accesos otorgados sobre ella), y `purchase_order_types` y `purchase_order_templates` (sus tipos y plantillas de orden de compra). Más adelante le apuntarán los vehículos y las facturas.

**Ejemplo**

| id | country_id | tax_id_type_id | tax_id | legal_name | trade_name | functional_currency_id |
|---|---|---|---|---|---|---|
| 1 | 1 · PE | 1 · RUC | 20512345678 | Millennium Mobility S.A.C. | Millennium | 1 · PEN |
| 2 | 2 · CO | 3 · NIT | 901234567 | Millennium Mobility Colombia S.A.S. | Millennium | 3 · COP |

Una empresa por país, cada una con su propia razón social y su propio documento tributario.

**Reglas de negocio**

- Hay una sola empresa por país, y cada empresa pertenece a un único país: expandirse crea una empresa nueva, no una sede de la existente.
- Una plantilla de orden de compra puede imprimir un emisor editado (§9.2), pero eso no crea otra empresa. Si algún día aparece otra razón social en el mismo país, será otra fila de esta tabla y habrá que revisar D-5.
- El número de documento tributario no se repite dentro del mismo país.
- El número se guarda normalizado, sin guiones ni espacios, para que la misma empresa no se registre dos veces escrita de formas distintas.
- El tipo de documento es del país de la empresa, está activo y aplica a empresas (§4.7). Si el tipo declara una longitud, el número la respeta. Que sea del mismo país lo impide la base, porque la llave hacia el tipo incluye el país; lo demás lo valida la aplicación.
- La razón social es la del registro público y no se ajusta para que se vea mejor: es la que se imprime en comprobantes y contratos.
- Una empresa que deja de operar se desactiva, nunca se borra: su historial de compras y contratos debe seguir siendo consultable.
- El país de la empresa se propaga a sus sedes y vehículos sin que puedan contradecirlo.

### 6.3 `commercial_brands` — Marcas comerciales
El nombre bajo el cual la empresa coloca sus unidades en el mercado: Lizzo para autos, Lima Bikes para motos y trimotos. Una empresa puede tener varias y se espera que aparezcan más.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa dueña de la marca | Una marca no existe por sí sola: pertenece a la razón social que factura |
| `code` | VARCHAR(20) | No | Código corto e invariable: `LIZZO`, `LIMABIKES` | Es lo que usan los reportes y las integraciones. El nombre visible puede cambiar; este no |
| `name` | VARCHAR(100) | No | El nombre como se muestra | — |
| `is_active` | BOOLEAN | No | Si se siguen colocando unidades bajo esta marca | Permite retirar una marca sin tocar los vehículos ya vendidos con ella |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | — |

**Relaciones.** Apunta a `companies`. Le apuntan `user_roles`, cuando un acceso se limita a una marca, y `purchase_order_templates`, cuando una plantilla es de una línea de negocio. Más adelante le apuntarán los vehículos, para registrar bajo qué marca se coloca cada unidad.

**Ejemplo**

| id | company_id | code | name | is_active |
|---|---|---|---|---|
| 1 | 1 · Millennium Perú | `LIZZO` | Lizzo | Sí |
| 2 | 1 · Millennium Perú | `LIMABIKES` | Lima Bikes | Sí |

Las dos cuelgan de la misma empresa: son marcas, no razones sociales. Agregar una tercera es insertar una fila.

**Reglas de negocio**

**Cuidado con la palabra "marca", que significa dos cosas en este proyecto:**

| Concepto | Ejemplos | Tabla |
|---|---|---|
| Marca comercial de la empresa | Lizzo, Lima Bikes | `commercial_brands` |
| Marca del vehículo | Chevrolet, TVS, Bajaj | `vehicle_brands` (bloque 4) |

Son tablas distintas y nunca deben mezclarse. La primera dice bajo qué nombre vendemos; la segunda, quién fabricó la unidad.

En la práctica cada marca se asocia a ciertos tipos de vehículo —Lizzo a autos, Lima Bikes a motos y trimotos—, pero esa relación se define en el bloque 4, cuando exista el catálogo de tipos.

### 6.4 `branches` — Sedes
Los locales de la empresa donde trabaja personal: oficinas, el patio propio, el taller. Es la unidad a la que se pueden asignar roles.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa a la que pertenece | — |
| `country_id` | BIGINT | No | País, heredado de la empresa | No puede contradecir al de la empresa: el sistema lo impide con una llave compuesta |
| `code` | VARCHAR(20) | No | Código corto de la sede: `LIM-CENTRAL` | Se usa en reportes y en la asignación de accesos |
| `name` | VARCHAR(100) | No | Nombre de la sede | — |
| `address_id` | BIGINT FK → `addresses` | Sí | Dónde queda | La dirección registrada aquí debe coincidir con la declarada ante la autoridad tributaria, porque es la que se imprime en los comprobantes |
| `tax_establishment_code` | VARCHAR(10) | Sí | Código con que la autoridad tributaria identifica este local. En Perú es el código de establecimiento anexo de SUNAT: `0000` es el domicilio fiscal y `0001` en adelante los anexos | Va impreso en cada comprobante que emite la sede. Sin él no se puede facturar desde ese local. Se deja opcional porque no todos los países usan este concepto |
| `timezone` | VARCHAR(50) | No | La zona horaria de esta sede, con su nombre IANA (`America/Lima`) | Aquí ocurren los hechos que se fechan: ingreso del vehículo, instalación de GPS, entrega. En países con varias zonas horarias, tomar la del país haría que los cortes del día salgan corridos |
| `is_active` | BOOLEAN | No | Si la sede sigue funcionando | — |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | — |

**Relaciones.** Apunta a `companies` y a `addresses`. Le apuntan `user_roles` (los accesos limitados a una sede) y `locations` (cuando la ubicación es un local propio).

**Ejemplo**

| id | company_id | code | name | tax_establishment_code | timezone | is_active |
|---|---|---|---|---|---|---|
| 1 | 1 · Millennium Perú | `LIM-CENTRAL` | Sede central Lima | 0000 | America/Lima | Sí |
| 2 | 1 · Millennium Perú | `LIM-TALLER` | Taller Ate | 0001 | America/Lima | Sí |

**Reglas de negocio**

- El código de la sede no se repite dentro de la misma empresa.
- El código de establecimiento tributario tampoco: es el que identifica al local ante la autoridad y va impreso en cada comprobante que emite.
- La dirección registrada debe coincidir con la declarada ante esa autoridad.
- Una sede pertenece siempre al país de su empresa; el sistema impide que apunte a otro.
- Una sede que deja de funcionar se desactiva: los vehículos y accesos que la mencionan siguen siendo explicables.
- La sede no lleva tipo (D-50). Hoy ninguna regla ni ningún reporte depende de la función del local, y la clase de lugar ya la dice `locations.location_type`. Lo que admitiría el campo está en §6.6.

### 6.5 `locations` — Ubicaciones de vehículos
**Decisión cerrada (D-6):** los patios arrendados y las tiendas de los dealers donde se entregan las trimotos **no son sedes de la empresa**: son lugares donde puede haber vehículos. Por eso van en una tabla aparte.

La diferencia importa por una razón concreta: a una sede se le asignan roles y personas, y aparece en la pantalla de accesos. Si cada cochera alquilada se registrara como sede, esa pantalla se llenaría de lugares a los que nadie va a asignar a nadie, y la tienda de un tercero figuraría como local propio.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa que usa esta ubicación | — |
| `code` | VARCHAR(20) | No | Código corto: `PATIO-ATE`, `DEALER-SJL` | Se usa en los reportes de stock por ubicación |
| `name` | VARCHAR(100) | No | Nombre del lugar | — |
| `location_type` | VARCHAR(20) | No | Qué clase de lugar es: `sede_propia`, `patio_arrendado`, `tienda_dealer`, `taller_tercero`, `otro` | Permite separar en los reportes lo que está en casa de lo que está en manos de terceros, que es una diferencia de riesgo |
| `branch_id` | BIGINT FK → `branches` | Sí | La sede, cuando la ubicación es un local propio | El patio principal es a la vez sede y ubicación. Este campo los enlaza en vez de duplicar el lugar |
| `address_id` | BIGINT FK → `addresses` | Sí | Dónde queda | Necesario para los lugares de terceros, que no tienen sede asociada |
| `is_active` | BOOLEAN | No | Si se siguen dejando vehículos ahí | Al terminar el alquiler de un patio se desactiva, y el historial de qué unidades estuvieron ahí se conserva |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | — |

**Relaciones.** Apunta a `companies`, a `addresses` y a `branches` (solo cuando el lugar es una sede propia). Más adelante le apuntarán los vehículos, para decir dónde está físicamente cada unidad.

**Ejemplo**

| id | company_id | code | name | location_type | branch_id |
|---|---|---|---|---|---|
| 1 | 1 | `PATIO-CENTRAL` | Patio sede central | sede_propia | 1 · Sede central Lima |
| 2 | 1 | `PATIO-ATE` | Patio alquilado Ate | patio_arrendado | — |
| 3 | 1 | `DEALER-SJL` | Tienda dealer San Juan | tienda_dealer | — |

La primera fila es la sede propia vista como lugar donde hay vehículos; las otras dos son de terceros y por eso no tienen sede asociada.

**Reglas de negocio**

**El vehículo apunta siempre a una ubicación, nunca a una sede.** Aunque esté en el patio principal, se registra en la ubicación correspondiente a esa sede. Así la pregunta "¿dónde está esta unidad?" se responde siempre igual, sin casos especiales según de quién sea el lugar.

Cuando el patio arrendado tenga un costo asociado —el flujograma menciona el indicador de costo por día paralizado—, ese dato colgará de esta tabla. Hoy no se incluye porque todavía no hay quién lo calcule.

### 6.6 Candidatos del bloque 3

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `lei_code` | `companies` | Que un fondo internacional exija el identificador global de entidades legales. Se paga y se renueva cada año |
| `parent_company_id` | `companies` | Que aparezca una holding con documento propio que consolide a las demás |
| `logo_path` | `commercial_brands` | Que la marca se muestre en documentos o en la aplicación del cliente |
| `branch_type` | `branches` | Que una regla dependa de la función del local —por ejemplo, que el rol `TALLER` solo se asigne a sedes que sean taller— o que un reporte agrupe sedes por función. No debe repetir `locations.location_type` (D-6): la sede principal se deriva de `tax_establishment_code = '0000'` y los patios y almacenes son ubicaciones. Si se admite, sus valores van en un solo idioma y no se solapan entre sí (D-50) |
| `rental_cost` y período | `locations` | Que se implemente el indicador de costo por día paralizado |

---

## 7. Bloque 4 · Catálogos de flota

Todo lo que describe un vehículo se parametriza antes de poder registrar una unidad. La guía funcional lo dice sin rodeos: si se compra un modelo nuevo, se crea en el catálogo **antes** del primer registro. El objetivo es que el mismo modelo no termine escrito de cinco formas distintas, que es lo que pasa hoy.

**Estos catálogos son del sistema, no de cada empresa.** Un Chevrolet Sail es el mismo en Perú y en Colombia, y mantener una copia por país garantiza que se escriban distinto y que ningún reporte consolidado cuadre. A cambio, un administrador de una empresa no los crea: los crea el superadministrador, con el permiso correspondiente.

### 7.1 `vehicle_types` — Tipos de vehículo

Clasifica la unidad y activa las reglas que dependen del tipo.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(20) UNIQUE | No | Código corto e invariable: `AUTO`, `MOTO`, `TRIMOTO` | Es el que usan las reglas del sistema. No cambia aunque cambie el nombre comercial |
| `name` | VARCHAR(50) | No | Cómo se llama el tipo: Automóvil, Motocicleta, Trimoto | — |
| `commercial_name` | VARCHAR(50) | No | El nombre con que la operación lo conoce: Leasing Car, Leasing Bike, Leasing Tuk Tuk | Es el que exige el exportable de stock y el que usa el fondo al que se reporta. Va aparte porque puede cambiar sin que cambie el tipo de vehículo |
| `is_active` | BOOLEAN | No | Si se siguen registrando unidades de este tipo | — |

**Relaciones.** Le apuntan `vehicle_models` y la tabla que dice a qué tipos de vehículo aplica cada equipamiento.

**Ejemplo**

| id | code | name | commercial_name |
|---|---|---|---|
| 1 | `AUTO` | Automóvil | Leasing Car |
| 2 | `MOTO` | Motocicleta | Leasing Bike |
| 3 | `TRIMOTO` | Trimoto | Leasing Tuk Tuk |

**Reglas de negocio**

- El tipo de una unidad no se escribe a mano: sale del modelo, y el modelo lo trae del catálogo.
- Se guardan los dos nombres a propósito. Hoy coinciden uno a uno, pero son conceptos distintos: si algún día se financia un auto bajo un producto que no sea leasing, el nombre comercial deja de servir como tipo y separarlo entonces obligaría a revisar todas las unidades.
- Cuidado con el nombre: en el documento de Registro de Clientes, "tipo de vehículo" significa `NUEVO / SEMIPREMIUM`, que es otra cosa y se deduce de si la unidad ya tuvo contratos.

### 7.2 `vehicle_brands` — Marcas de vehículo

La marca del vehículo: Chevrolet, TVS, Bajaj, Hyundai. Es la columna `MARCA` del Excel.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `name` | VARCHAR(80) UNIQUE | No | Nombre oficial de la marca | Sin catálogo, la misma marca aparece como `TVS`, `Tvs` y `T.V.S.`, y contar unidades por marca deja de ser posible |
| `is_active` | BOOLEAN | No | Si se siguen comprando unidades de esta marca | Permite retirarla sin tocar las unidades ya registradas |

**Relaciones.** Le apunta `vehicle_models`.

**Ejemplo**

| id | name | is_active |
|---|---|---|
| 1 | CHEVROLET | Sí |
| 2 | TVS | Sí |
| 3 | BAJAJ | Sí |
| 4 | HYUNDAI | Sí |

**Reglas de negocio**

- No confundir con las marcas comerciales de §6.3: aquí está la marca del vehículo; allá, bajo qué nombre lo coloca Millennium. Una moto **TVS King** vendida bajo la marca **Lima Bikes** usa las dos tablas.
- Una marca nueva se crea antes de poder registrar un modelo suyo.

### 7.3 `vehicle_models` — Modelos

La combinación de marca y modelo determina el tipo de vehículo y las reglas especiales.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_brand_id` | BIGINT FK → `vehicle_brands` | No | De qué marca es | El nombre del modelo solo identifica junto a la marca: hay modelos con nombres parecidos de marcas distintas |
| `vehicle_type_id` | BIGINT FK → `vehicle_types` | No | Si es auto, moto o trimoto | De aquí sale el tipo de cada unidad, sin que nadie lo elija a mano |
| `name` | VARCHAR(120) | No | Denominación oficial del modelo: `SAIL SEDAN 1.5L LT MT` | Se escribe una vez y todas las unidades la reutilizan |
| `engine_cc` | INTEGER | Sí | Cilindraje | Es uno de los datos que hoy se registra por unidad y que en realidad es del modelo: escribirlo una vez evita que la misma moto figure con cilindrajes distintos |
| `fuel_type` | VARCHAR(20) | Sí | Combustible: gasolina, GNV, GLP, diésel, eléctrico, híbrido | Determina si la unidad necesita certificado de gas. Sin este dato, esa exigencia habría que recordarla a mano modelo por modelo |
| `has_factory_screen` | BOOLEAN | No | Si el modelo ya viene con pantalla instalada de fábrica | Es la regla de la pantalla convertida en dato. Hoy está escrita como una comparación de texto —"Chevrolet más Sail Sedan"— y por eso agregar un modelo obliga a tocar el programa |
| `is_active` | BOOLEAN | No | Si se siguen comprando unidades de este modelo | — |

**Relaciones.** Apunta a `vehicle_brands` y a `vehicle_types`. Le apuntarán los vehículos y las tarifas de costo por kilómetro del bloque de ventas.

**Ejemplo**

| id | vehicle_brand_id | vehicle_type_id | name | engine_cc | fuel_type | has_factory_screen |
|---|---|---|---|---|---|---|
| 1 | 1 · CHEVROLET | 1 · AUTO | SAIL SEDAN 1.5L LT MT | 1500 | gasolina | No |
| 2 | 4 · HYUNDAI | 1 · AUTO | GRAND I10 | 1200 | gasolina | Sí |
| 3 | 2 · TVS | 3 · TRIMOTO | KING LS+ DP IT | 200 | GLP | No corresponde |
| 4 | 3 · BAJAJ | 2 · MOTO | DISCOVER 125-ST | 125 | gasolina | No corresponde |

La primera fila es el único modelo de auto que hoy requiere instalar pantalla; la segunda, uno que ya la trae.

**Reglas de negocio**

- La misma marca no puede tener dos modelos con el mismo nombre.
- Un modelo se crea antes de poder registrar una unidad suya. No se crea al vuelo durante una carga.
- El tipo de vehículo de una unidad sale de su modelo y no se puede contradecir desde la unidad.
- La exigencia de pantalla sale de este catálogo, no de comparar textos. Agregar un modelo sin pantalla de fábrica es cargar una fila.
- Un modelo que deja de comprarse se desactiva: las unidades ya registradas lo siguen referenciando.

**Advertencia para la carga inicial.** Los dos catálogos entregados no coinciden entre sí. La guía funcional lista modelos sin marca y el documento de clientes los lista con marca y con otra escritura: `GRAND I10` frente a `I10`, o `GLORY 560 PRO 1.8L MT COMFORT 7 AS` frente a la misma denominación en minúsculas. Cargar ambos sin conciliarlos produce modelos duplicados el primer día, que es exactamente lo que este catálogo existe para evitar.

### 7.4 `colors` — Colores

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `name` | VARCHAR(50) UNIQUE | No | Nombre del color | Es un campo que hoy se escribe libre y termina con `Blanco`, `BLANCO` y `blanco perlado` como tres colores distintos. En un inventario, agrupar por color deja de funcionar |
| `is_active` | BOOLEAN | No | Si se sigue usando | — |

**Relaciones.** Le apuntarán los vehículos.

**Ejemplo**

| id | name |
|---|---|
| 1 | BLANCO |
| 2 | NEGRO |
| 3 | GRIS |
| 4 | ROJO |

**Reglas de negocio**

- Es lista cerrada: la unidad elige de aquí, no escribe el color.
- Un color nuevo se crea antes de poder usarlo, igual que un modelo.

### 7.5 `equipment_types` — Tipos de equipamiento

Lo que se instala en la unidad: GPS, pantalla multimedia y lo que venga.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(20) UNIQUE | No | Código corto: `GPS`, `PANTALLA` | Lo usan las reglas de habilitación |
| `name` | VARCHAR(60) | No | Nombre del equipo | — |
| `blocks_availability` | BOOLEAN | No | Si sin este equipo la unidad no puede pasar a disponible | El GPS bloquea; la pantalla solo en los modelos que la necesitan. Escrito así, cambiar esa política es marcar una casilla y no modificar el programa |
| `requires_serial` | BOOLEAN | No | Si al instalarlo hay que registrar un número de serie o identificador | El GPS tiene un identificador que después hace falta para ubicar la unidad. Sin este dato, quedaría al criterio de quien carga |
| `is_active` | BOOLEAN | No | Si se sigue instalando | — |

#### `equipment_type_vehicle_types` — A qué vehículos aplica cada equipo

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `equipment_type_id` | BIGINT FK → `equipment_types` | No | El equipo. Parte de la llave | — |
| `vehicle_type_id` | BIGINT FK → `vehicle_types` | No | El tipo de vehículo. Parte de la llave | La pantalla solo tiene sentido en autos; en motos y trimotos no corresponde. Como tabla se puede validar y consultar, cosa que una lista escrita dentro de una columna no permite |

**Ejemplo**

| equipo | aplica a |
|---|---|
| GPS | Auto, Moto, Trimoto |
| Pantalla | Auto |

**Relaciones.** Se relaciona con `vehicle_types`. Le apuntarán los equipos instalados en cada unidad, en el bloque 9.

**Ejemplo**

| id | code | name | blocks_availability | requires_serial | Aplica a |
|---|---|---|---|---|---|
| 1 | `GPS` | Dispositivo GPS | Sí | Sí | Auto, moto, trimoto |
| 2 | `PANTALLA` | Pantalla multimedia | Solo si el modelo la requiere | No | Auto |

**Reglas de negocio**

- Sin GPS instalado, la unidad no pasa a disponible. Es una de las tres condiciones que el cliente enumeró.
- La pantalla solo bloquea en los modelos que no la traen de fábrica. En motos y trimotos no corresponde.
- Un equipo no se instala en un tipo de vehículo al que no aplica.

**La pantalla usa dos datos que responden preguntas distintas y conviene no confundir.** La aplicabilidad dice si el equipo *tiene sentido* en ese tipo de vehículo; `has_factory_screen` dice si ese modelo *ya la trae puesta*. Una moto no es un auto sin pantalla: es un vehículo donde la pregunta no corresponde. La regla, escrita una sola vez:

> Una unidad requiere instalación de pantalla si el equipo aplica a su tipo de vehículo **y** su modelo no la trae de fábrica. En cualquier otro caso, no corresponde.

Si esa regla se implementa en dos lugares —uno mirando el tipo y otro mirando el modelo— tarde o temprano dirán cosas distintas.

### 7.6 La placa no tiene tabla propia

La guía funcional exige 6 caracteres y el documento de Registro de Clientes admite 6 o 7: en Perú la placa de un auto y la de una moto no tienen la misma estructura. Se evaluó una tabla de formatos por país y tipo de vehículo y se descartó, porque la validación estricta que permitiría es indeseable —una placa legítima bloqueada obliga al operador a inventar una que pase el filtro— y sin ella la tabla no tendría qué contener.

La placa es una columna del vehículo, con tres reglas:

- Se guarda normalizada: mayúsculas, sin guiones ni espacios. Es lo único que previene duplicados, porque la restricción de unicidad no distingue `ABC-123` de `ABC123`.
- No se repite dentro de un mismo país.
- Solo se controla que sean letras y números dentro de un rango de longitud.

Corregir una placa es un cambio delicado y debe quedar registrado quién lo hizo: los cruces con el SAT y con garantía mobiliaria dependen de ella.

### 7.7 Candidatos del bloque 4

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `plate_formats` (tabla) | — | Placas provisionales con estructura propia, o un país que las defina por estado |
| `hex_code` | `colors` | Que la aplicación del cliente muestre el color de su unidad |
| `weight`, `passengers` | `vehicle_models` | Que haga falta para un trámite o un reporte. Hoy nadie los usa |
| `warranty_months` | `vehicle_models` | Que se gestione la garantía de fábrica, que hoy no se registra |

---

## 8. Bloque 6 · El vehículo y sus estados

Es el centro del modelo: la empresa financia vehículos, y todo lo demás —compras, contratos, documentos— cuelga de aquí.

**Lo que esta tabla no guarda.** El proveedor no es un atributo de la unidad sino de la compra: en el Excel aparece en cada fila porque una hoja de cálculo es plana, pero en el modelo sale de la factura. Los costos llegan con el bloque 7 y los datos de habilitación con el bloque 9.

### 8.1 `vehicles` — Vehículos

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa propietaria | Es quien compró la unidad y figura como dueña mientras no se traspase |
| `country_id` | BIGINT | No | País, heredado de la empresa | Permite exigir que la placa no se repita dentro del país sin recorrer relaciones en cada consulta |
| `commercial_brand_id` | BIGINT FK → `commercial_brands` | No | La marca comercial bajo la que se coloca | Lizzo o Lima Bikes. No es la marca del vehículo |
| `model_id` | BIGINT FK → `vehicle_models` | No | El modelo, que trae consigo la marca y el tipo de vehículo | De aquí sale si es auto, moto o trimoto, sin que nadie lo elija a mano |
| `color_id` | BIGINT FK → `colors` | No | Color de la unidad | Sirve para identificarla físicamente en el patio y en una recuperación |
| `vin` | VARCHAR(17) UNIQUE | No | Número de chasis, 17 caracteres | Es el identificador que no cambia nunca. La placa aparece después y puede corregirse; el VIN acompaña a la unidad toda su vida |
| `engine_number` | VARCHAR(50) | Sí | Número de motor | Es el segundo identificador físico. Va opcional porque en las facturas de lote de motos no siempre viene, y exigirlo en el alta obligaría al operador a inventarlo |
| `plate` | VARCHAR(10) | Sí | Placa, en mayúsculas y sin guiones | Vacía hasta la inmatriculación. Es la llave con que se cruzan los archivos del SAT y de garantía mobiliaria |
| `model_year` | SMALLINT | No | Año del modelo | Se valida entre 2017 y el año siguiente al actual |
| `current_location_id` | BIGINT FK → `locations` | Sí | Dónde está físicamente la unidad | Vacío mientras no ha ingresado. Apunta siempre a una ubicación, aunque sea la sede propia, para que "¿dónde está?" se responda igual en todos los casos |
| `current_status_id` | BIGINT FK → `vehicle_statuses` | No | En qué estado del ciclo de vida está | Ver §8.2 |
| `legacy_number` | INTEGER | Sí | El número correlativo que la unidad tenía en el Excel | Solo para las unidades migradas, como referencia a reportes antiguos. No se generan números nuevos: ver abajo |
| `transfer_date` | DATE | Sí | Fecha en que dejó de ser de la empresa | Es cuando el activo sale de los libros. Vacío significa que sigue siendo de la empresa |
| `transfer_reason` | VARCHAR(30) | Sí | Por qué se traspasó: fin de contrato, venta directa, otro | Un traspaso por fin de contrato y una venta directa tienen tratamientos contables distintos |
| `annulled_at` | TIMESTAMPTZ | Sí | Cuándo se anuló la unidad | Se marca cuando una nota de crédito deja su valor en cero: la compra se deshizo. La fila no se borra porque su historial debe seguir siendo explicable |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Cuándo y quién registró la unidad | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Última modificación | — |

**Relaciones.** Apunta a `companies`, `commercial_brands`, `vehicle_models`, `colors`, `locations` y `vehicle_statuses`. Le apuntan los contratos, los equipos instalados, los documentos, los seguros, las líneas de factura y el historial de estados.

**Ejemplo**

| id | company_id | commercial_brand_id | model_id | vin | plate | model_year | current_location_id | current_status_id |
|---|---|---|---|---|---|---|---|---|
| 1 | 1 · Millennium Perú | 1 · Lizzo | 1 · CHEVROLET SAIL SEDAN | 9BGKS69X0LG123456 | ABC123 | 2025 | 1 · Patio central | 2 · Disponible |
| 2 | 1 · Millennium Perú | 2 · Lima Bikes | 4 · BAJAJ DISCOVER 125-ST | MD2A11CY6LWA12345 | *(vacía)* | 2026 | 1 · Patio central | 1 · En habilitación |

La segunda unidad todavía no tiene placa: llega con la inmatriculación.

**Reglas de negocio**

- El VIN no se repite en todo el sistema y tiene 17 caracteres.
- La placa no se repite dentro de un mismo país, y se guarda normalizada —mayúsculas, sin guiones ni espacios— porque la restricción de unicidad no distingue `ABC-123` de `ABC123`.
- Corregir un VIN o una placa ya registrados es un cambio delicado: los cruces con el SAT y con garantía mobiliaria dependen de la placa, así que debe quedar registrado quién lo hizo y cuándo.
- El tipo de vehículo sale del modelo y no se puede contradecir desde la unidad.
- La unidad no se borra nunca. Si una nota de crédito anula la compra, se marca como anulada; si se traspasa, se registra la fecha.
- **El propietario vigente se deduce**: si no hay fecha de traspaso, es la empresa; si la hay, dejó de serlo. No hay un campo que lo diga, para que no pueda contradecir a la fecha.

**No se generan correlativos nuevos.** El `N°` del Excel se conserva solo en las unidades migradas, como referencia. El orden de registro lo da la fecha de alta y la identificación, el VIN y la placa.

### 8.2 `vehicle_statuses` — Estados del vehículo

Los nueve estados del flujograma, y los que se agreguen mientras se refinan los procesos.

**Por qué es una tabla y no una lista fija.** Porque la lista va a crecer. Si estuviera escrita dentro del programa, agregar un estado obligaría a modificar el sistema y volver a instalarlo; siendo una tabla, es cargar una fila.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(30) UNIQUE | No | Código corto: `DISPONIBLE`, `ACTIVO`, `CUSTODIA` | Es el que usan las reglas del sistema |
| `name` | VARCHAR(60) | No | Nombre visible | — |
| `category` | VARCHAR(20) | No | Familia a la que pertenece: comercial, productivo, recuperación, terminal | El flujograma agrupa los estados por familia y así se calculan los indicadores sin enumerar estados uno por uno |
| `generates_revenue` | BOOLEAN | No | Si la unidad genera ingresos estando en este estado | Es lo que permite calcular el porcentaje de activación de flota, que es el indicador principal del tablero |
| `is_initial` | BOOLEAN | No | Si una unidad recién registrada nace en este estado | Hoy es "En habilitación". Marcarlo evita que el estado inicial quede escrito en el programa |
| `is_active` | BOOLEAN | No | Si se sigue usando | Permite retirar un estado sin tocar el historial de las unidades que pasaron por él |

**Relaciones.** Le apuntan `vehicles` (estado actual), `status_change_reasons` como origen y destino, y `vehicle_status_history`.

**Ejemplo**

| id | code | name | category | genera ingresos |
|---|---|---|---|---|
| 1 | `HABILITACION` | En habilitación | comercial | No |
| 2 | `DISPONIBLE` | Disponible | comercial | No |
| 3 | `VENDIDO` | Vendido | comercial | No |
| 4 | `ACTIVO` | Activo | productivo | Sí |
| 5 | `MECANICA` | Mecánica | recuperación | No |
| 6 | `SINIESTRO` | Siniestro | recuperación | No |
| 7 | `CUSTODIA` | Custodia | recuperación | No |
| 8 | `PAP` | Puesta a punto | recuperación | No |
| 9 | `INACTIVO` | Inactivo | terminal | No |

### 8.3 `status_change_reasons` — Caminos permitidos, motivos y tolerancias

Una sola tabla en lugar de dos. Como **todo cambio de estado exige motivo**, el motivo puede llevar consigo de dónde a dónde va: los caminos permitidos son los pares distintos que aparecen aquí. Separarlos en dos tablas agregaba un salto más a la cadena sin permitir expresar nada nuevo — un camino sin ningún motivo cargado sería inusable.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `from_status_id` | BIGINT FK → `vehicle_statuses` | Sí | Estado de origen. Vacío significa el alta de la unidad | — |
| `to_status_id` | BIGINT FK → `vehicle_statuses` | No | Estado de destino | Solo existen los cambios que figuran aquí. De Inactivo no se vuelve porque no hay fila con ese origen |
| `code` | VARCHAR(30) | No | Código del motivo: `MANT_PREVENTIVO`, `CHOQUE`, `MOROSIDAD` | — |
| `name` | VARCHAR(80) | No | Nombre visible | Es lo que elige el usuario al mover la unidad |
| `sla_hours` | INTEGER | Sí | Cuántas horas se tolera la unidad en ese estado por ese motivo | Es lo que dispara la alerta. Vacío significa que ese caso no se mide |
| `on_breach` | VARCHAR(20) | Sí | Qué hacer al vencerse: avisar o mover la unidad sola | El flujograma pide que una unidad Vendida vuelva a Disponible a las 48 horas. Escrito así, ese automatismo se configura en vez de programarse |
| `auto_transition_to_status_id` | BIGINT FK → `vehicle_statuses` | Sí | A qué estado mover la unidad si el automatismo aplica | — |
| `is_active` | BOOLEAN | No | Si el motivo sigue disponible | Desactivar todos los motivos de un par cierra ese camino sin borrar el historial de quienes ya lo recorrieron |

**Relaciones.** Apunta dos veces a `vehicle_statuses`, como origen y como destino, y una tercera cuando hay automatismo. Le apunta `vehicle_status_history`.

**Ejemplo**

| desde | hacia | motivo | tolerancia | al vencerse |
|---|---|---|---|---|
| *(alta)* | En habilitación | Registro de compra | — | — |
| En habilitación | Disponible | Habilitación completa | — | — |
| Disponible | Vendido | Venta | 48 h | Volver a Disponible |
| Vendido | Activo | Entrega al cliente | — | — |
| Activo | Mecánica | Mantenimiento preventivo | 72 h | Avisar |
| Activo | Mecánica | Choque | 480 h | Avisar |
| Activo | Custodia | Morosidad | 720 h | Avisar |
| Custodia | Puesta a punto | Resolución de contrato | — | — |
| Puesta a punto | Disponible | Reacondicionada | — | — |

Las dos filas de Mecánica muestran para qué sirve la tabla: mismo camino, distinta expectativa según por qué entró la unidad.

**Reglas de negocio**

- Un cambio de estado que no figure aquí se rechaza. No es una advertencia: la unidad no se mueve.
- Todo cambio registra fecha, hora, usuario y motivo. Es la exigencia del flujograma y no admite excepción.
- El tiempo límite se cuenta desde el momento del cambio, y la unidad aparece en la lista de vencidas al pasarse.
- La única transición que ocurre sola es la que tiene configurado un estado de destino al vencerse. Todas las demás las hace una persona.
- Un motivo no se borra: se desactiva, para que el historial siga siendo explicable.

### 8.4 `vehicle_status_history` — Historial de estados

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `from_status_id` | BIGINT FK → `vehicle_statuses` | Sí | Estado anterior. Vacío en el alta | — |
| `to_status_id` | BIGINT FK → `vehicle_statuses` | No | Estado nuevo | — |
| `reason_id` | BIGINT FK → `status_change_reasons` | No | Qué camino y por qué motivo se movió | Sin el motivo, dentro de seis meses nadie puede explicar por qué esa unidad estuvo un mes parada |
| `changed_by` | BIGINT FK → `users` | No | Quién lo hizo. El usuario del sistema cuando fue automático | — |
| `changed_at` | TIMESTAMPTZ | No | Cuándo | — |
| `due_at` | TIMESTAMPTZ | Sí | Cuándo vence la tolerancia de este motivo | Se calcula al registrar el cambio. Se guarda para poder listar las unidades vencidas sin recalcular el plazo de todo el historial en cada consulta |
| `notes` | TEXT | Sí | Detalle libre | — |

**Relaciones.** Apunta a `vehicles`, a `vehicle_statuses` como origen y destino, a `status_change_reasons` y a `users`.

**Ejemplo** — la vida de una unidad en cuatro movimientos:

| desde | hacia | motivo | cuándo | vence |
|---|---|---|---|---|
| *(alta)* | En habilitación | Registro de compra | 03/02/2026 | — |
| En habilitación | Disponible | Habilitación completa | 18/02/2026 | — |
| Disponible | Vendido | Venta | 02/03/2026 09:10 | 04/03/2026 09:10 |
| Vendido | Activo | Entrega al cliente | 03/03/2026 16:20 | — |

La cuarta fila es la fecha de entrega: no hay un campo aparte para ella.

**Reglas de negocio**

- El historial no se edita ni se borra. Un error se corrige con un cambio nuevo, no reescribiendo el anterior.
- De aquí sale la fecha en que la unidad pasó a estar lista para entrega, que el Excel guarda como un campo manual y por eso se desincroniza.
- También sale de aquí cuánto tiempo pasó la unidad en cada estado, que es lo que alimenta los indicadores de días promedio y de costo por día paralizado.

**El estado actual se guarda además en la unidad**, aunque se podría deducir de la última fila del historial. Es una excepción deliberada a la regla de no persistir derivados: cada pantalla de stock lista cientos de unidades con su estado, y deducirlo exigiría recorrer el historial de cada una. La condición es que ambos se escriban en la misma operación, de modo que no puedan discrepar.

### 8.5 `vehicle_mileage_readings` — Lecturas de kilometraje

El kilometraje no es un dato que se corrige: es una serie de lecturas. Guardar solo el valor actual serviría para mostrarlo en pantalla y para nada más, y en este negocio hace falta para algo más: el contrato cobra por kilómetro recorrido, y eso exige saber cuánto marcaba al entregar y cuánto marca ahora.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `mileage` | INTEGER | No | Kilómetros que marcaba el odómetro | — |
| `read_at` | TIMESTAMPTZ | No | Cuándo se tomó la lectura | Es la fecha de la lectura, no la del registro: una lectura de taller puede cargarse dos días después |
| `source` | VARCHAR(20) | No | De dónde salió: GPS, taller, entrega, devolución, manual | Una lectura del dispositivo y una que alguien escribió mirando el tablero no merecen la misma confianza |
| `recorded_by` | BIGINT FK → `users` | Sí | Quién la registró. Vacío cuando la trajo el dispositivo | — |
| `notes` | TEXT | Sí | Detalle | Para explicar una lectura que rompe la secuencia |

**Relaciones.** Apunta a `vehicles` y a `users`.

**Ejemplo**

| unidad | kilometraje | cuándo | origen |
|---|---|---|---|
| ABC123 | 12 | 03/03/2026 | Entrega |
| ABC123 | 4 180 | 01/04/2026 | GPS |
| ABC123 | 8 350 | 01/05/2026 | GPS |
| ABC123 | 8 410 | 12/05/2026 | Taller |

La primera es la lectura de entrega: contra ella se mide todo lo que el cliente recorra.

**Reglas de negocio**

- **Una lectura no puede ser menor que la anterior.** Un odómetro no retrocede: si pasa, o es un error de digitación o le cambiaron el tablero. El sistema lo rechaza y exige registrarlo como sustitución de odómetro, que es un hecho distinto. Aceptarlo en silencio corrompería el cobro por kilómetro sin que nadie lo note.
- Tampoco puede ser una lectura futura.
- Las lecturas no se editan ni se borran. Una equivocada se corrige registrando la correcta con su motivo.
- El kilometraje de la unidad es el de la última lectura. Ambos se escriben en la misma operación para que no puedan discrepar.

### 8.6 Candidatos del bloque 6

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `transferred_to_customer_id` | `vehicles` | El bloque 12, cuando exista el cliente al que se traspasa la unidad |
| `requires_permission_id` | `status_change_reasons` | Que algún cambio de estado deba estar reservado a ciertos roles |


---

## 9. Bloque 7 · Compras

Es el bloque donde la auditoría encontró los problemas más graves, y casi todos vienen del mismo origen: en el Excel la factura es una columna de la fila del vehículo, cuando en la práctica una factura puede cubrir ochenta motos y recibir notas de crédito durante meses.

Tres reglas del negocio que el modelo tiene que poder expresar y hoy no:

- En autos, una factura es un vehículo. En motos, una factura puede traer ochenta, y una nota de crédito puede anular dos de ellas.
- Un vehículo puede recibir **más de una** nota de crédito: una por la campaña del mes y otra por una campaña adicional.
- Una nota de crédito emitida hoy puede corresponder a una compra de hace dos meses, porque el bono depende de cuándo se entregó la unidad y no de cuándo se compró.

**El proveedor entra aquí.** Se había pospuesto porque no es un atributo del vehículo —en el Excel figura en cada fila solo porque una hoja es plana—, pero una factura sin proveedor no existe.

### 9.1 `suppliers` — Proveedores

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa que le compra | El proveedor se registra con el documento tributario del país donde opera. La misma empresa vendiendo en Colombia es otro proveedor, con otro número |
| `tax_id_type_id` | BIGINT FK → `tax_id_types` | No | Con qué documento se identifica | — |
| `tax_id` | VARCHAR(20) | No | Número del documento, sin guiones ni espacios | Va en la factura y es lo que revisa el auditor |
| `legal_name` | VARCHAR(255) | No | Razón social | Es la que debe figurar en la orden de compra que se le envía |
| `trade_name` | VARCHAR(150) | Sí | Nombre comercial | El nombre corto con que lo conoce la operación |
| `address_id` | BIGINT FK → `addresses` | Sí | Domicilio | Se imprime en la orden de compra |
| `is_active` | BOOLEAN | No | Si se le sigue comprando | Permite retirarlo sin tocar las facturas antiguas |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | — |

**Relaciones.** Apunta a `companies`, `tax_id_types` y `addresses`. Le apuntan las órdenes de compra, las facturas y las notas de crédito.

**Ejemplo**

| id | company_id | tax_id | legal_name | trade_name |
|---|---|---|---|---|
| 1 | 1 · Millennium Perú | 20100128056 | Crosland Automotriz S.A.C. | Crossland |
| 2 | 1 · Millennium Perú | 20605412345 | Derco Perú S.A. | Derco |

**Reglas de negocio**

- El documento tributario no se repite dentro de la misma empresa.
- Un proveedor nuevo se crea antes de poder registrarle una compra. No se crea al vuelo durante una carga.
- No confundir con la marca del vehículo: Crossland vende motos Bajaj. El proveedor es a quién se le compra; la marca, qué se compró.

### 9.2 `purchase_orders` — Órdenes de compra

Lo que se le pide al proveedor. No bloquea nada: una unidad puede registrarse sin que exista una orden.

Cada orden se emite con una **plantilla**, que decide lo que se imprime —quién emite, a quién va dirigida, a quién consultar, qué instrucciones y qué firma lleva— y la serie con que se numera. Las plantillas y los tipos de orden se describen al final de esta sección.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que compra | Es la empresa dueña de la plantilla con que se emite |
| `supplier_id` | BIGINT FK → `suppliers` | No | A quién se le pide | — |
| `purchase_order_template_id` | BIGINT FK → `purchase_order_templates` | No | Con qué plantilla se emite | De ella salen los datos que se imprimen y la serie de numeración |
| `series` | VARCHAR(20) | No | Serie de la plantilla al crear la orden: `OC-LBT` | Se copia y no se vuelve a leer de la plantilla: si la serie se corrigiera, las órdenes ya numeradas no cambiarían de número |
| `sequence_year` | SMALLINT | No | Año del número: el de la fecha de creación, en la hora local del país de la empresa | Se fija al asignar el número y no se recalcula. Un borrador creado el 30 de diciembre y enviado en enero conserva el número del año en que se creó |
| `sequence_number` | INTEGER | No | Correlativo dentro de la serie y el año: 1, 2, 3… | Vuelve a 1 cada año en cada serie |
| `order_number` | TEXT, columna generada | No | Número completo: serie, año y correlativo con cuatro dígitos como mínimo. `OC-LBT-2026-0001` | Es el que aparece en el PDF que se envía al proveedor y con el que él responde. Se calcula de las tres columnas anteriores, no se escribe |
| `issue_date` | DATE | No | Fecha de emisión | — |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda del pedido | — |
| `payment_terms` | VARCHAR(50) | Sí | Condición de pago acordada: contado, crédito 30 | Se imprime en la orden y es lo que después se reclama |
| `purpose` | VARCHAR(150) | Sí | Concepto del pedido: requerimiento de stock | — |
| `status` | VARCHAR(20) | No | En qué va: borrador, enviada, cumplida, anulada | Es el seguimiento que pidió el cliente |
| `sent_at` | TIMESTAMPTZ | Sí | Cuándo se envió al proveedor | Distingue una orden preparada de una ya comprometida |
| `pdf_path` | TEXT | Sí | Archivo generado | Es el documento que se le envía; conviene conservar el que efectivamente se mandó y no volver a generarlo |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la emitió y cuándo | — |

#### `purchase_order_items` — Líneas de la orden

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `purchase_order_id` | BIGINT FK → `purchase_orders` | No | A qué orden pertenece | — |
| `model_id` | BIGINT FK → `vehicle_models` | Sí | Qué modelo se pide, cuando la línea es de vehículos | Al ordenar todavía no se sabe qué VIN van a llegar: se pide por modelo y cantidad. Vacío en las líneas que no son vehículos |
| `concept` | VARCHAR(150) | Sí | Qué se pide, cuando la línea no es un vehículo: un mantenimiento, repuestos, un servicio | Las órdenes de taller no compran modelos. Es la misma solución que las líneas de factura (D-22) |
| `quantity` | INTEGER | No | Cuántas unidades | Es contra esto que se calcula lo pendiente de llegar |
| `unit_price` | NUMERIC(14,2) | No | Precio acordado por unidad | Es el precio comprometido, que puede diferir del que después facture el proveedor |

Cada línea lleva un modelo o un concepto, nunca los dos ni ninguno. El total de la línea se calcula, no se escribe.

**Relaciones.** Apunta a `companies`, `suppliers`, `currencies` y `purchase_order_templates`; sus líneas apuntan a `vehicle_models` cuando son de vehículos. Le apuntan opcionalmente las facturas.

**Ejemplo**

| orden | plantilla | proveedor | fecha | estado | línea | cantidad | precio unitario |
|---|---|---|---|---|---|---|---|
| OC-LBV-2026-0014 | Lima Bikes · Vehículos | Crossland | 03/02/2026 | Enviada | BAJAJ DISCOVER 125-ST | 80 | S/ 5 900 |
| OC-LZV-2026-0003 | Lizzo · Vehículos | Derco | 10/02/2026 | Borrador | CHEVROLET SAIL SEDAN | 12 | S/ 48 500 |
| OC-LBT-2026-0001 | Lima Bikes · Taller | Mecánica Andina | 12/02/2026 | Enviada | Mantenimiento preventivo de 10 000 km | 10 | S/ 120 |

Cada serie lleva su propio correlativo: la orden de taller es la primera de su serie en el año, aunque ya existan órdenes de vehículos.

**Reglas de negocio**

- La orden **no bloquea el registro de stock**. Si llega una unidad sin orden previa, se registra igual.
- Las líneas de vehículos se piden por cantidad y modelo, no por unidad: al pedirlas todavía no se sabe qué VIN van a llegar.
- Una orden enviada no se edita: se anula y se emite otra. El proveedor ya tiene el documento anterior en la mano.
- **El pendiente se calcula, no se escribe**: es lo pedido menos lo que ya llegó facturado. Ver la regla siguiente.
- El vínculo de la factura con la orden es **opcional**. Con él, el pendiente se calcula solo; sin él, la orden se cierra a mano.
- Toda orden se emite con una plantilla de su misma empresa; el sistema impide usar la plantilla de otra. Lo que se imprime sale de la plantilla al generar el PDF: mientras la orden es borrador, refleja los cambios que se le hagan a la plantilla.
- **Una vez enviada, su PDF es el documento de la orden.** Se conserva en `pdf_path`, y editar la plantilla después no altera las órdenes ya enviadas.

#### Cómo se numera una orden

- Cada plantilla tiene su serie, y cada serie su correlativo, que vuelve a 1 cada año: `OC-LBT-2026-0001`, `OC-LBT-2026-0002`… y en enero, `OC-LBT-2027-0001`. El año en el número dice de un vistazo de cuándo es la orden y lo mantiene corto.
- El número se asigna al crear la orden, aunque sea un borrador, y **no se reutiliza nunca**. Como nada se borra —un borrador que no sigue se anula—, cada número corresponde a una orden que se puede consultar, vigente o anulada: no quedan huecos sin explicación.
- El año es el de la fecha de creación **en la hora local del país de la empresa** (`countries.default_timezone`), no en la hora universal. Una orden creada en Lima el 31 de diciembre a las 20:00 es del año que termina.
- El número siguiente es el mayor de esa serie en ese año más uno. Se calcula dentro de la operación que crea la orden, con la plantilla bloqueada, para que dos órdenes simultáneas no tomen el mismo número. No se guarda un contador: sería un derivado que puede desincronizarse.
- La combinación de empresa, serie, año y correlativo no se repite: es la garantía final de que no haya dos órdenes con el mismo número.
- La orden de compra no es un comprobante de pago: su serie y su correlativo no los regula la autoridad tributaria, así que siguen la forma que ya usa la operación.

#### `purchase_order_types` — Tipos de orden

Clasifica la orden según lo que se compra. Junto con la línea de negocio, decide qué plantilla corresponde.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa que usa el tipo | Cada empresa define los suyos |
| `code` | VARCHAR(20) | No | Código corto e invariable: `VEHICULOS`, `TALLER`, `OTROS` | Lo usan los reportes y los filtros. El nombre visible puede cambiar; este no |
| `name` | VARCHAR(60) | No | Nombre visible: Vehículos, Taller, Otros | — |
| `is_active` | BOOLEAN | No | Si se siguen emitiendo órdenes de este tipo | Permite retirarlo sin tocar las órdenes ya emitidas |

**Relaciones.** Apunta a `companies`. Le apunta `purchase_order_templates`.

**Ejemplo**

| code | name | activo |
|---|---|---|
| `VEHICULOS` | Vehículos | Sí |
| `TALLER` | Taller | Sí |
| `OTROS` | Otros | Sí |

**Reglas de negocio**

- Es tabla y no lista cerrada: agregar un tipo —repuestos, servicios— es cargar una fila, sin tocar el programa, porque el tipo no cambia cómo funciona la orden, solo qué plantilla le corresponde.
- El código no se repite dentro de la empresa y queda fijo en cuanto una plantilla lo usa.
- Un tipo no se borra: se desactiva.

#### `purchase_order_templates` — Plantillas de orden de compra

El formato con que se emite una orden. Los campos son siempre los mismos; lo que cambia de una plantilla a otra son sus valores, según la línea de negocio y el tipo de orden (D-35).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa dueña de la plantilla | Es la que compra con las órdenes emitidas con ella, y la que propone los datos del emisor |
| `commercial_brand_id` | BIGINT FK → `commercial_brands` | Sí | Línea de negocio: Lizzo, Lima Bikes. **Vacío = plantilla general** | Cada línea puede llevar su propio formato. Tiene que ser una marca de la misma empresa |
| `purchase_order_type_id` | BIGINT FK → `purchase_order_types` | No | Tipo de orden: vehículos, taller, otros | — |
| `name` | VARCHAR(100) | No | Nombre con que se elige al emitir: Lima Bikes · Taller | — |
| `series` | VARCHAR(20) | No | Prefijo de numeración: `OC-LBT`. En mayúsculas y sin espacios | Separa la numeración de cada plantilla: cada serie lleva su propio correlativo |
| `issuer_legal_name` | VARCHAR(255) | No | Razón social que se imprime como emisora | Se propone la de la empresa y se puede editar |
| `issuer_tax_id_type_id` | BIGINT FK → `tax_id_types` | No | Tipo de documento del emisor: RUC | Se propone el de la empresa |
| `issuer_tax_id` | VARCHAR(20) | No | Número de documento del emisor, sin guiones ni espacios | Se propone el de la empresa |
| `issuer_phone` | VARCHAR(30) | Sí | Teléfono del emisor | Se propone el de la empresa (`companies.phone`) |
| `issuer_address_id` | BIGINT FK → `addresses` | Sí | Dirección que se imprime; el distrito o ciudad sale de su división territorial | Fila propia, copiada del domicilio fiscal de la empresa al crear la plantilla. Las direcciones no se comparten entre entidades (§4.6) |
| `attention_name` | VARCHAR(150) | Sí | Sección "Atención": quién recibe la orden dentro de la empresa | Es texto y no una referencia a un colaborador: puede ser un área o un buzón compartido |
| `attention_position` | VARCHAR(80) | Sí | Puesto de quien recibe | — |
| `attention_email` | VARCHAR(255) | Sí | Correo de quien recibe | — |
| `attention_phone` | VARCHAR(30) | Sí | Teléfono de quien recibe | — |
| `support_name` | VARCHAR(150) | Sí | Contacto de soporte: aparece al pie como "Si usted tiene alguna pregunta, póngase en contacto con…" | — |
| `support_phone` | VARCHAR(30) | Sí | Teléfono del contacto de soporte | — |
| `support_email` | VARCHAR(255) | Sí | Correo del contacto de soporte | — |
| `special_instructions` | TEXT | Sí | Comentarios o instrucciones que van al pie de cada orden: a dónde enviar las facturas | Es lo que el proveedor necesita para facturar bien. Sin un campo propio, se escribe a mano en cada orden |
| `signature_path` | TEXT | Sí | Imagen de la firma, en la esquina inferior derecha. Vacío = sin firma | El archivo se guarda fuera de la base, igual que los documentos del vehículo (§11.3) |
| `is_active` | BOOLEAN | No | Si se siguen emitiendo órdenes con ella | Permite retirarla sin tocar las órdenes ya emitidas |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la creó y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Última modificación | — |

**Relaciones.** Apunta a `companies`, `commercial_brands`, `purchase_order_types`, `tax_id_types` y `addresses`. Le apunta `purchase_orders`.

**Ejemplo**

| nombre | línea | tipo | serie | emisor | atención | soporte |
|---|---|---|---|---|---|---|
| Lima Bikes · Vehículos | Lima Bikes | Vehículos | `OC-LBV` | Millennium Mobility S.A.C. · RUC 20512345678 | Henry Saldaña, Gerente | Alejandro Suarez |
| Lima Bikes · Taller | Lima Bikes | Taller | `OC-LBT` | Millennium Mobility S.A.C. · RUC 20512345678 | Jefe de taller | Alejandro Suarez |
| Lizzo · Vehículos | Lizzo | Vehículos | `OC-LZV` | Millennium Mobility S.A.C. · RUC 20512345678 | Henry Saldaña, Gerente | Alejandro Suarez |

Las tres toman el emisor de la misma empresa y difieren en lo demás. Cambiar el emisor de una no afecta a las otras.

**Reglas de negocio**

- Los campos son los mismos en todas las plantillas; lo que cambia son sus valores. Una línea de negocio o un tipo de orden nuevos no requieren tocar el programa: se crea otra plantilla.
- Al crear una plantilla, los datos del emisor —razón social, documento, teléfono y domicilio fiscal— se copian de la empresa dueña y **se pueden editar**. Es el mismo criterio que la tasa de impuesto: `countries` la propone y la que manda es la guardada en el comprobante. Editar la empresa después no cambia las plantillas existentes.
- La serie no se repite dentro de la empresa. Se guarda normalizada —mayúsculas, sin espacios— y se puede corregir mientras la plantilla no tenga órdenes; después queda fija, como el código de un catálogo (§2.4).
- La línea de negocio, si se indica, tiene que ser una marca de la misma empresa: el sistema lo impide, igual que en las asignaciones de rol (§5.7).
- Puede haber más de una plantilla activa para la misma línea y tipo —dos talleres con contactos distintos, por ejemplo—: quien emite la orden elige cuál usar.
- Los correos se guardan sin espacios y en minúsculas, y el documento del emisor sin guiones ni espacios (§2.2).
- Una plantilla no se borra: se desactiva. Las órdenes emitidas con ella la siguen referenciando.
- Cada cambio queda en la bitácora (§13), con el valor anterior y el nuevo. El documento de cada orden enviada ya lo conserva su PDF; la bitácora explica qué tenía la plantilla en una fecha pasada.
- Crear o editar plantillas exige el permiso `compras.gestionar_plantillas`.

### 9.3 `purchase_invoices` — Facturas de compra

El comprobante del proveedor. Es el encabezado: quién emitió, cuándo, en qué moneda y por cuánto.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que compra | — |
| `supplier_id` | BIGINT FK → `suppliers` | No | Quién emitió el comprobante | — |
| `purchase_order_id` | BIGINT FK → `purchase_orders` | Sí | A qué orden corresponde | Opcional. Si está, permite calcular lo pendiente de la orden |
| `document_type` | VARCHAR(20) | No | Tipo de comprobante: factura, boleta, otro | Se prellena como factura porque es el caso prácticamente universal, y se puede cambiar |
| `document_number` | VARCHAR(30) | No | Serie y número del comprobante | Es lo que se le muestra al auditor |
| `invoice_date` | DATE | No | Fecha del comprobante | Es el límite inferior de varias fechas posteriores y determina el tipo de cambio aplicable |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda del comprobante | — |
| `exchange_rate` | NUMERIC(12,6) | No | Tipo de cambio aplicado | Se guarda el que se usó, no el de hoy: un comprobante de 2024 no puede recalcularse solo cuando cambie el dólar |
| `tax_rate` | NUMERIC(5,4) | No | Tasa de impuesto aplicada a esta factura | Ver la nota sobre el valor contable |
| `gross_total` | NUMERIC(14,2) | No | Total del comprobante, impuestos incluidos | Debe coincidir con la suma de sus líneas |
| `file_path` | TEXT | Sí | Archivo del comprobante | Se envía a los auditores. El cliente lo pidió expresamente |
| `xml_path` | TEXT | Sí | Archivo electrónico del comprobante | Es el que permitirá extraer los datos automáticamente más adelante: el formato es el mismo para todos los proveedores, a diferencia del PDF |
| `annulled_at` | TIMESTAMPTZ | Sí | Si el comprobante completo se anuló | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | — |

**Relaciones.** Apunta a `companies`, `suppliers`, `purchase_orders` (opcional) y `currencies`. Le apuntan sus líneas y las notas de crédito.

**Ejemplo**

| comprobante | proveedor | fecha | moneda | TC | tasa | total |
|---|---|---|---|---|---|---|
| F001-00123 | Crossland | 15/03/2026 | PEN | 1,000000 | 0,1800 | 472 000,00 |
| F002-00981 | Derco | 22/03/2026 | USD | 3,742000 | 0,1800 | 13 100,00 |

La primera cubre ochenta motos; la segunda, un auto. El tipo de cambio es el de la fecha del comprobante, no el de hoy.

**Reglas de negocio**

- El número de comprobante no se repite dentro del mismo proveedor.
- La suma de las líneas debe coincidir con el total del comprobante. Es la validación que detecta una línea olvidada al cargar ochenta motos.
- Una factura con más de veinte días de antigüedad solo la registra quien tenga el permiso de excepción. No es un bloqueo absoluto: es un permiso que se puede otorgar.
- La factura no se borra. Si se anula por completo, se marca.

**Sobre el impuesto.** La tasa se guarda en cada factura, no como constante: cambia por ley y difiere entre países. `countries` propone la del país al registrar y el valor es editable, porque una unidad importada directamente llega sin impuesto del proveedor. El ISC va incluido en el precio y no se recupera, así que forma parte del costo; se guarda aparte solo como información.

### 9.4 `purchase_invoice_items` — Líneas de factura

**Es la tabla que el modelo anterior no tenía, y la que hace posible todo lo demás.** Sin ella, una factura de ochenta motos es un solo monto y no hay forma de anular dos unidades, ni de saber cuánto costó cada una.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `purchase_invoice_id` | BIGINT FK → `purchase_invoices` | No | A qué comprobante pertenece | — |
| `vehicle_id` | BIGINT FK → `vehicles` | Sí | Qué unidad se está comprando | Vacío cuando la línea no es un vehículo: un flete, un accesorio, un trámite. Hoy solo se usan líneas de vehículo, pero la estructura ya lo admite |
| `concept` | VARCHAR(150) | Sí | Descripción de la línea cuando no es un vehículo | — |
| `invoiced_amount` | NUMERIC(14,2) | No | **Valor de factura**: lo que el comprobante dice que costó | No se modifica nunca. Es lo que se reporta al fondo y lo que ve el auditor |
| `isc_amount` | NUMERIC(14,2) | No | Impuesto selectivo incluido en el precio, si aplica | Solo en motos. Informativo: ya está dentro del valor de factura |
| `status` | VARCHAR(20) | No | Vigente o anulada | Una nota de crédito puede anular esta línea sin tocar el resto del comprobante |
| `annulled_at` | TIMESTAMPTZ | Sí | Cuándo se anuló | — |

**Relaciones.** Apunta a `purchase_invoices` y opcionalmente a `vehicles`. Le apuntan las líneas de nota de crédito.

**Ejemplo** — una factura de motos con ochenta líneas, de las que se muestran tres:

| comprobante | VIN | monto facturado | ISC | estado |
|---|---|---|---|---|
| F001-00123 | MD2A11CY6LWA12345 | 5 900,00 | 420,00 | Vigente |
| F001-00123 | MD2A11CY6LWA12346 | 5 900,00 | 420,00 | Vigente |
| F001-00123 | MD2A11CY6LWA12347 | 5 900,00 | 420,00 | Anulada |

**Reglas de negocio**

- Una misma unidad no puede aparecer dos veces en el mismo comprobante.
- El valor de factura **no se modifica nunca**. Los descuentos posteriores no lo tocan: se registran como notas de crédito y el valor del vehículo se calcula.
- Anular una línea no altera las demás. Es exactamente el caso del proveedor que anula dos motos de una factura de ochenta.

### 9.5 `credit_notes` — Notas de crédito

Solo de crédito: el proveedor no emite notas de débito.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que la recibe | — |
| `supplier_id` | BIGINT FK → `suppliers` | No | Quién la emitió | Una nota de crédito solo puede descontar compras del mismo proveedor |
| `document_number` | VARCHAR(30) | No | Serie y número del documento | — |
| `issue_date` | DATE | No | Fecha de emisión | No puede ser anterior a la factura que descuenta |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda | Debe ser la misma de la factura que descuenta |
| `exchange_rate` | NUMERIC(12,6) | No | Tipo de cambio aplicado | — |
| `total_amount` | NUMERIC(14,2) | No | Monto total del documento | Debe coincidir con la suma de lo que reparte entre unidades |
| `reason` | VARCHAR(30) | No | Por qué se emitió: descuento de campaña, anulación de unidad, ajuste de precio | Un descuento y una anulación tienen efectos distintos sobre la unidad |
| `campaign_month` | DATE | Sí | Mes de la campaña que la origina, guardado con día 1 | Es lo que permite responder qué notas corresponden a las entregas de septiembre, que es como el proveedor las liquida |
| `file_path` | TEXT | Sí | Archivo del documento | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | — |

#### `credit_note_items` — Reparto de la nota entre unidades

Es donde la nota se vuelve concreta: qué unidad recibe cuánto.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `credit_note_id` | BIGINT FK → `credit_notes` | No | La nota que se está repartiendo | — |
| `purchase_invoice_item_id` | BIGINT FK → `purchase_invoice_items` | No | A qué unidad de qué factura afecta | Apunta a la línea de factura y no a "la compra actual": por eso una nota de hoy puede descontar una compra de hace dos meses |
| `amount` | NUMERIC(14,2) | No | Cuánto le corresponde a esa unidad | La suma de las líneas debe coincidir con el total del documento |

**Relaciones.** Apunta a `companies`, `suppliers` y `currencies`; sus líneas apuntan a `purchase_invoice_items`.

**Ejemplo** — una nota de 10 000 repartida entre cuatro de las diez unidades de una factura:

| nota | fecha | motivo | unidad afectada | monto |
|---|---|---|---|---|
| NC001-0045 | 15/09/2026 | Descuento de campaña | MD2A11CY6LWA12345 | 2 500,00 |
| NC001-0045 | 15/09/2026 | Descuento de campaña | MD2A11CY6LWA12346 | 2 500,00 |
| NC001-0045 | 15/09/2026 | Descuento de campaña | MD2A11CY6LWA12347 | 2 500,00 |
| NC001-0045 | 15/09/2026 | Descuento de campaña | MD2A11CY6LWA12348 | 2 500,00 |

**Reglas de negocio**

- **Una nota de crédito nunca reemplaza a otra.** Un vehículo puede recibir varias —la campaña del mes y una adicional— y todas se conservan. Este es el punto donde el modelo anterior perdía datos: al ser un solo campo, la segunda nota borraba a la primera sin dejar rastro.
- **Debe detallar a qué unidades afecta.** El sistema no reparte por su cuenta entre todas las unidades del comprobante. Si el documento menciona una unidad que no está en esa factura, se rechaza.
- **Puede descontar una compra anterior.** El bono de campaña se emite cuando la unidad se entrega, no cuando se compró, así que una nota de hoy puede corresponder a una factura de hace dos meses. Por eso apunta a la línea de factura y no a "la compra actual".
- La moneda debe ser la misma de la factura que descuenta.
- La suma de lo repartido debe coincidir con el total del documento.

### 9.6 Valor de factura y valor del vehículo

Son dos datos distintos y ambos hacen falta.

| Concepto | Dónde vive | Cambia | Para qué se usa |
|---|---|---|---|
| **Valor de factura** | `purchase_invoice_items.invoiced_amount` | Nunca | Reportes externos y auditoría |
| **Notas de crédito de la unidad** | `credit_note_items`, una fila por nota | Se agregan filas | Historia de los descuentos |
| **Valor del vehículo** | No se guarda: se calcula | Con cada nota nueva | Contabilidad, inventario y análisis interno |

**Ejemplo** — una moto comprada en marzo que recibe dos descuentos:

| Momento | Documento | Monto | Valor de factura | Valor del vehículo |
|---|---|---|---|---|
| Marzo | Factura F001-00123 | 5 900,00 | 5 900,00 | 5 900,00 |
| Agosto | Nota de crédito NC001-0045 | −310,00 | 5 900,00 | 5 590,00 |
| Octubre | Nota de crédito NC001-0061 | −150,00 | 5 900,00 | 5 440,00 |

El valor de factura sigue siendo 5 900 en las tres filas: es lo que dice el comprobante y lo que ve el auditor. El valor del vehículo baja con cada nota y es el que usa contabilidad, porque la unidad vale menos.

**Reglas de negocio**

- El valor del vehículo **no es un campo**. Guardarlo lo desactualizaría en cuanto llegue una nota nueva, que es exactamente lo que ocurre hoy.
- Una unidad puede recibir varias notas y todas se conservan. La segunda no reemplaza a la primera.
- Cuando el valor del vehículo llega a cero, la compra se deshizo y la unidad queda anulada.
- **La unidad solo se puede anular si su valor llegó a cero y no tiene ningún contrato.** Es el candado para que una nota de crédito no borre información de un cliente.

### 9.7 Lo que queda para el bloque de pagos

Una nota de crédito tiene dos efectos independientes y aquí está resuelto solo el primero:

| Efecto | Dónde vive |
|---|---|
| Reduce el valor de una unidad concreta | Este bloque |
| Genera un saldo a favor con el proveedor, que sirve para pagar otras facturas | Bloque 8 |

El valor de las unidades no cambia porque una compra se haya cubierto con saldo: siguen costando lo que costaron, pero no salió efectivo. Son dos reportes distintos.

### 9.8 Candidatos del bloque 7

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| Extracción automática desde el archivo electrónico | `purchase_invoices` | Que se implemente la lectura del comprobante para evitar digitar ochenta líneas |
| `withholding_amount` | `purchase_invoices` | Que aparezcan retenciones o detracciones, que hoy no se registran |
| `expected_delivery_date` | `purchase_order_items` | Que haga falta seguir la fecha comprometida de llegada, no solo la cantidad |
| Una sola plantilla activa por línea y tipo | `purchase_order_templates` | Que se pida impedir dos plantillas vigentes para la misma línea de negocio y el mismo tipo de orden |


---

## 10. Bloque 8 · Pagos a proveedores

Todo este bloque es dinero que **sale** hacia el proveedor. Lo que un cliente paga por su unidad es un cobro y pertenece al bloque de ventas: mezclarlos es lo que hace que el flujo de caja no cuadre.

Tres cosas que el Excel no puede expresar y aquí sí:

- Un pago cubre **varias facturas a la vez**: se paga por bloques, no factura por factura.
- Se paga **antes de que exista la factura**, contra la orden de compra. No hay restricción de fechas.
- Se paga **con saldo de notas de crédito** en vez de con dinero.

### 10.1 `supplier_payments` — Pagos

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que paga | — |
| `supplier_id` | BIGINT FK → `suppliers` | No | A quién se le paga | Un pago no puede repartirse entre proveedores distintos |
| `payment_date` | DATE | No | Fecha del pago | Puede ser anterior a la factura. No se valida contra ella |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda | — |
| `exchange_rate` | NUMERIC(12,6) | No | Tipo de cambio aplicado | El del día del pago, que puede diferir del de la factura |
| `amount` | NUMERIC(14,2) | No | Monto total del pago | — |
| `method` | VARCHAR(30) | No | Cómo se pagó: transferencia, cheque, efectivo, saldo a favor | Un pago con saldo a favor no mueve dinero, y esa diferencia es la que separa el flujo de caja del valor del inventario |
| `operation_number` | VARCHAR(100) | Sí | Número de operación del voucher | Es lo que se concilia con el banco |
| `credit_note_id` | BIGINT FK → `credit_notes` | Sí | Qué nota de crédito se consumió, cuando se paga con saldo | Vacío en los pagos con dinero |
| `annulled_at` | TIMESTAMPTZ | Sí | Cuándo se anuló el pago | Un pago mal registrado no se borra: se anula y se vuelve a registrar. Las aplicaciones de un pago anulado dejan de contar |
| `notes` | TEXT | Sí | Detalle | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién lo registró y cuándo | — |

**Relaciones.** Apunta a `companies`, `suppliers`, `currencies` y opcionalmente a `credit_notes`. Le apuntan sus aplicaciones.

**Ejemplo**

| operación | fecha | monto | método | nota consumida |
|---|---|---|---|---|
| OP-99213 | 12/03/2026 | 300 000,00 | Transferencia | — |
| — | 04/07/2026 | 306 000,00 | Saldo a favor | NC001-0045 y otras |

El segundo no movió dinero: se cubrió con notas de crédito acumuladas. Las unidades siguen valiendo lo mismo.

**Reglas de negocio**

- El número de operación no se repite dentro del mismo proveedor: un mismo voucher no puede registrarse dos veces.
- Un pago con método *saldo a favor* debe indicar qué nota de crédito consume, y no puede superar lo que queda disponible de ella.
- El pago no se borra. Si se registró mal, se anula y se vuelve a registrar.

### 10.2 `payment_applications` — A qué se aplica cada pago

Un pago se reparte entre facturas, órdenes de compra, o ambas.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `payment_id` | BIGINT FK → `supplier_payments` | No | Qué pago se está aplicando | — |
| `purchase_invoice_id` | BIGINT FK → `purchase_invoices` | Sí | A qué factura | — |
| `purchase_order_id` | BIGINT FK → `purchase_orders` | Sí | A qué orden, cuando todavía no hay factura | Es el caso normal: el pago suele ir antes que el comprobante |
| `amount` | NUMERIC(14,2) | No | Cuánto de ese pago se aplica aquí | Permite que un pago cubra tres facturas con montos distintos |

**Relaciones.** Apunta a `supplier_payments` y a `purchase_invoices` u `purchase_orders`.

**Ejemplo** — un pago de 300 000 repartido entre dos facturas y una orden aún sin facturar:

| pago | fecha | aplicado a | monto |
|---|---|---|---|
| OP-99213 | 12/03/2026 | Factura F001-00123 | 177 000 |
| OP-99213 | 12/03/2026 | Factura F001-00124 | 88 500 |
| OP-99213 | 12/03/2026 | Orden OC-LZV-2026-0003 | 34 500 |

**Reglas de negocio**

- Cada aplicación apunta a una factura **o** a una orden, nunca a las dos.
- La suma de las aplicaciones no puede superar el monto del pago.
- Lo aplicado a una factura no puede superar su total.
- Cuando la factura de una orden ya pagada aparece, la aplicación se traslada de la orden a la factura. El pago no se toca.

### 10.3 `payment_allocations` — Qué unidades cubre un pago

Opcional. Solo hace falta cuando un pago parcial libera unidades concretas.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `payment_application_id` | BIGINT FK → `payment_applications` | No | Qué aplicación se está detallando | — |
| `vehicle_id` | BIGINT FK → `vehicles` | No | Qué unidad queda cubierta | — |
| `amount` | NUMERIC(14,2) | No | Cuánto le corresponde | — |

**Reglas de negocio**

- **Si no se detalla, el pago se reparte proporcionalmente** entre las unidades de la factura. Es el comportamiento por defecto y cubre el caso normal.
- **En compras al canal de dealers el detalle es obligatorio**, porque la diferencia entre el valor y lo pagado corresponde a la inicial de un cliente concreto. Repartirla entre todas las unidades atribuiría iniciales a clientes que no las pagaron.
- Si se detalla, esas unidades quedan pagadas y el resto no. Es el caso de pagar media factura para llevarse 25 de las 50 motos: se llevan motos enteras, no media moto cada una.
- Una unidad está pagada cuando lo aplicado a ella cubre su valor de vehículo.
- Los céntimos que no se reparten exactos se asignan a la primera unidad, para que la suma coincida con el pago.

### 10.4 El saldo a favor con el proveedor

Una nota de crédito tiene dos efectos y este es el segundo: además de bajar el valor de una unidad, deja un saldo disponible con ese proveedor.

| Concepto | Cómo se obtiene |
|---|---|
| Saldo disponible de una nota | Su monto total menos lo ya consumido en pagos |
| Saldo del proveedor | La suma de los saldos disponibles de sus notas |

**No se guarda como campo**: se calcula. Un saldo almacenado se desincroniza al primer pago que nadie registre bien.

**Reglas de negocio**

- El saldo solo sirve para pagar facturas **del mismo proveedor** que emitió la nota.
- Un pago con saldo a favor **no mueve dinero**, pero cancela la factura igual. Las unidades siguen valiendo lo que valen: el saldo no baja el inventario, baja la caja que iba a salir.
- La nota que genera el saldo puede haber descontado una compra anterior. Son dos relaciones distintas: a qué unidad le bajó el valor, y qué factura pagó.

### 10.5 Indicadores que se calculan

| Indicador | Cómo sale |
|---|---|
| Diferencia por inicial | Valor de las unidades menos lo que Millennium pagó por ellas |
| Brecha de días | Días entre la fecha base y la entrega; si no se entregó, hasta hoy. La fecha base es la del pago y, si no hay pago registrado, la de la factura |

**Qué es la diferencia por inicial.** Mide lo que Millennium no desembolsó porque el cliente pagó su inicial directamente al concesionario: la factura dice 50 000, el concesionario cobra 42 000 y la diferencia son los 8 000 que puso el cliente. Sigue ocurriendo en el canal de dealers y sirve para validar contra ellos las iniciales cobradas.

Por eso **en ese canal la diferencia debe imputarse a las unidades concretas y no repartirse**. Si un bloque de pago cubre diez autos y solo tres clientes pagaron inicial, el prorrateo le atribuye a las otras siete una inicial que nadie pagó y la validación con el concesionario deja de cuadrar. La imputación por unidad, que en el caso general es opcional, aquí es obligatoria.

La validación en sí cruza dos mundos: esta diferencia, que sale de compras, contra la inicial registrada en el contrato del cliente, que vive en ventas. El reporte se puede construir cuando exista ese bloque.

Ninguno de los dos indicadores se guarda. La brecha **se recalcula siempre con la mejor información disponible**: si el pago se registra dos meses después, la brecha de esa unidad cambia. No es un error sino un dato que faltaba, y congelarla significaría fijar para siempre un número calculado sobre información incompleta. Si hace falta que un reporte mensual sea reproducible, lo que se conserva es el reporte, no el campo.

### 10.6 Candidatos del bloque 8

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `bank_account_id` | `supplier_payments` | Que se lleve control de desde qué cuenta salió cada pago |
| `scheduled_date` | `payment_applications` | Que se programen pagos futuros y haga falta un calendario de vencimientos |

---

## 11. Bloque 9 · Habilitación

Lo que hay que completar para que una unidad pueda entregarse: que llegue físicamente, se inmatricule, obtenga sus documentos, se le instale el equipamiento y se le asignen las llaves.

### 11.1 Campos que se agregan a `vehicles`

Son atributos de la unidad, con un solo valor cada uno, así que van en la misma tabla del bloque 6 y no en una tabla aparte.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `arrival_date` | DATE | Sí | Cuándo ingresó físicamente a la empresa | Es el "en planta" del Excel. Se conserva como hito aunque la unidad salga después: sin ingreso no se puede instalar el GPS |
| `registration_start_date` | DATE | Sí | Cuándo empezó la inmatriculación | — |
| `title_number` | VARCHAR(50) | Sí | Número de título que entrega el concesionario | Sirve para seguir el trámite. No es único: puede repetirse entre unidades |
| `title_date` | DATE | Sí | Fecha del título | Solo existe si hay título. Es la fecha de primera inscripción que se reporta |
| `plates_received_date` | DATE | Sí | Cuándo llegaron las placas físicas | Distinto de tener el número de placa asignado |
| `current_mileage` | INTEGER | Sí | Último kilometraje conocido | Es el valor de la última lectura registrada. Se guarda además en la unidad porque toda pantalla de stock lo muestra y deducirlo exigiría buscar la última lectura de cada una |

**Reglas de negocio**

- La placa solo se puede registrar si la unidad tiene el título y el documento de identificación vehicular cargado.
- La fecha del título no puede existir sin título.
- El ingreso físico se conserva como hito histórico: volver a marcarlo como no ingresado solo sirve para corregir un error.

### 11.2 `document_types` — Tipos de documento del vehículo

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(30) UNIQUE | No | Código corto: `TIVE`, `SOAT`, `CERT_GAS`, `REV_TECNICA`, `FACTURA` | — |
| `name` | VARCHAR(80) | No | Nombre del documento | — |
| `has_expiry` | BOOLEAN | No | Si vence | Determina si el sistema debe avisar cuando se acerque la fecha |
| `blocks_availability` | BOOLEAN | No | Si sin él la unidad no puede pasar a disponible | Hoy solo el documento de identificación vehicular bloquea. Los demás son obligatorios pero no detienen la venta: el cliente fue explícito en que pueden demorarse |
| `is_visible_to_customer` | BOOLEAN | No | Si el cliente puede descargarlo desde su aplicación | El cliente pidió que pueda ver los documentos de su unidad. No todos corresponde mostrarlos |
| `requires_gas_conversion` | BOOLEAN | No | Si solo se exige a unidades convertidas a gas. Por defecto `FALSE` | El certificado de gas aplica a GNV y a GLP y a ninguna otra unidad. Un booleano cubre ambos casos sin agregar una tabla |
| `is_active` | BOOLEAN | No | Si se sigue pidiendo | — |

**Relaciones.** Le apunta `vehicle_documents`.

**Reglas de negocio**

- Un tipo de documento no se borra: se desactiva. Los documentos ya cargados lo siguen referenciando.
- Solo los marcados como bloqueantes impiden que la unidad pase a disponible. Hoy únicamente el documento de identificación vehicular.

**Ejemplo**

| code | name | vence | bloquea | visible al cliente |
|---|---|---|---|---|
| `TIVE` | Tarjeta de identificación vehicular | No | Sí | Sí |
| `SOAT` | Seguro obligatorio | Sí | No | Sí |
| `CERT_GAS` | Certificado de conversión a gas | Sí | No | Sí |
| `REV_TECNICA` | Revisión técnica | Sí | No | Sí |
| `FACTURA` | Factura de compra | No | No | No |

La factura se guarda para los auditores y no se le muestra al cliente. Un auto nuevo no tiene revisión técnica y por eso ese documento no bloquea nada.

### 11.3 `vehicle_documents` — Documentos cargados

Cada archivo que se sube para una unidad.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `document_type_id` | BIGINT FK → `document_types` | No | Qué documento es | — |
| `file_path` | TEXT | No | Dónde está guardado el archivo | El archivo no se guarda dentro de la base: se guarda aparte y aquí queda su ubicación. Meter archivos en la base la hace lenta de copiar y de restaurar |
| `file_name` | VARCHAR(255) | No | Nombre original del archivo | Es el que ve el usuario al descargarlo |
| `mime_type` | VARCHAR(100) | No | Tipo de archivo: PDF, imagen | Determina si se puede mostrar en pantalla o solo descargar |
| `file_size` | BIGINT | Sí | Tamaño en bytes | — |
| `issue_date` | DATE | Sí | Fecha de emisión del documento | — |
| `expiry_date` | DATE | Sí | Fecha de vencimiento | Obligatoria si el tipo vence. Es lo que dispara el aviso de renovación |
| `is_current` | BOOLEAN | No | Si es la versión vigente | Un SOAT renovado no borra al anterior: el nuevo queda vigente y el viejo se conserva |
| `uploaded_by` | BIGINT FK → `users` | No | Quién lo cargó | — |
| `uploaded_at` | TIMESTAMPTZ | No | Cuándo | — |

**Relaciones.** Apunta a `vehicles`, `document_types` y `users`.

**Ejemplo** — una unidad cuyo SOAT se renovó:

| documento | archivo | emitido | vence | vigente |
|---|---|---|---|---|
| SOAT | soat_abc123_2025.pdf | 01/03/2025 | 28/02/2026 | No |
| SOAT | soat_abc123_2026.pdf | 01/03/2026 | 28/02/2027 | Sí |
| Tarjeta de identificación | tive_abc123.pdf | 20/02/2025 | — | Sí |

El SOAT anterior se conserva: permite explicar con qué documento circulaba la unidad el año pasado.

**Reglas de negocio**

- Solo puede haber **un documento vigente de cada tipo por unidad**. Al cargar uno nuevo, el anterior deja de ser vigente y se conserva.
- Un documento no se borra: se reemplaza. El historial permite explicar con qué documento se entregó la unidad.
- Si el tipo vence, la fecha de vencimiento es obligatoria.
- Los documentos no impiden vender, salvo los marcados como bloqueantes. Un auto nuevo sin revisión técnica se entrega igual.
- La aplicación del cliente solo muestra los tipos marcados como visibles.

### 11.4 `vehicle_equipments` — Equipamiento instalado

Cada instalación, con su período. Es el patrón de intervalos: varias filas en el tiempo, una sola vigente por tipo.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `equipment_type_id` | BIGINT FK → `equipment_types` | No | Qué equipo | — |
| `serial` | VARCHAR(100) | Sí | Identificador del equipo | Obligatorio para los tipos que lo exigen, como el GPS |
| `period` | DATERANGE | No | Desde cuándo hasta cuándo estuvo instalado. Abierto = sigue puesto | Permite responder qué GPS tenía la unidad en una fecha pasada |
| `removal_reason` | VARCHAR(30) | Sí | Por qué se retiró: falla, robo, reemplazo | — |
| `installed_by_employee_id` | BIGINT FK → `employees` | Sí | Quién lo instaló | — |
| `cost` | NUMERIC(12,2) | No | Costo de la instalación. Por defecto 0 | Entra en el costo de habilitación de la unidad |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda del costo | Ningún monto se guarda sin su moneda |
| `created_at` | TIMESTAMPTZ | No | Cuándo se registró | — |

**Relaciones.** Apunta a `vehicles`, `equipment_types`, `employees` y `currencies`.

**Ejemplo** — una unidad a la que se le reemplazó el GPS:

| equipo | serial | desde | hasta | motivo del retiro |
|---|---|---|---|---|
| GPS | 8641-AA12 | 15/02/2026 | 04/07/2026 | Falla |
| GPS | 8712-BC55 | 04/07/2026 | — | — |

**Reglas de negocio**

- Dos equipos del mismo tipo no pueden estar instalados a la vez en la misma unidad, ni siquiera con períodos que se crucen en el pasado.
- Un equipo no se instala en un tipo de vehículo al que no aplica.
- No se puede instalar en una unidad que no ha ingresado físicamente.
- Reemplazar un equipo cierra el período del anterior y abre uno nuevo. No se sobrescribe ni se borra la fecha, como hace el Excel cuando se desmarca la casilla.

### 11.5 `vehicle_keys` — Llaves

Se registran todas: la que se entrega con la unidad y el duplicado que queda en la empresa.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `company_id` | BIGINT | No | Empresa, heredada del vehículo | Repetido a propósito: la etiqueta no se repite dentro de la empresa, y sin esta columna esa regla exigiría recorrer la relación con el vehículo en cada alta |
| `key_type` | VARCHAR(20) | No | Entregada, duplicado, otra | — |
| `tag_code` | VARCHAR(30) | No | Etiqueta que asigna la empresa. Única por empresa | Es la que permite responder "¿cuál es la llave de tal carro?", que es para lo que se pidió |
| `serial` | VARCHAR(50) | Sí | Serial que trae la llave de fábrica | Puede repetirse entre vehículos: lo pone el fabricante y no lo controlamos. Sirve para pedir una copia, no para buscar |
| `storage_group` | VARCHAR(10) | Sí | Caja donde se guarda: `G01` | Se repite entre unidades a propósito: es una ubicación, no una identidad. Vacío para las llaves que no están en la empresa |
| `holder` | VARCHAR(20) | No | Dónde está: empresa, cliente, perdida | Sin este dato, la caja quedaría vacía en la mitad de las filas sin explicar por qué |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | — |

**Relaciones.** Apunta a `vehicles`.

**Ejemplo**

| unidad | tipo | etiqueta | caja | dónde está |
|---|---|---|---|---|
| ABC123 | Entregada | LL-0450 | — | Con el cliente |
| ABC123 | Duplicado | LL-0451 | G01 | En la empresa |

**Reglas de negocio**

- La etiqueta no se repite dentro de la misma empresa. Es lo que hace posible la búsqueda.
- El serial de fábrica sí puede repetirse y no identifica nada.
- Solo las llaves que están en la empresa tienen caja asignada.

### 11.6 Los seguros son documentos, no una tabla aparte

El SOAT es un archivo con fecha de emisión y de vencimiento, que es exactamente lo que `vehicle_documents` guarda. Modelarlo además como póliza duplicaría las mismas fechas y el mismo archivo en dos lugares, con la garantía de que algún día dirán cosas distintas.

Se evaluó una tabla de pólizas con su cobertura por unidad y se dejó fuera, por dos razones: el seguro vehicular **hoy no se contrata** —ni en motos ni en autos, según la reunión— y el único seguro que sí existe ya está cubierto como documento.

Las reglas del SOAT viven entonces en el catálogo de tipos de documento: vence, no bloquea la disponibilidad, y es visible para el cliente en su aplicación. El aviso de renovación sale de la fecha de vencimiento, igual que para el certificado de gas o la revisión técnica.

**Qué traería las pólizas de vuelta:** que se contrate seguro vehicular de flota, donde una sola póliza cubre muchas unidades que entran y salen en fechas distintas. Ahí sí hace falta una tabla propia con su cobertura por unidad, porque la vigencia de la póliza y la de cada unidad dejan de coincidir. Queda como candidato del bloque.

### 11.7 `vehicle_tax_declarations` — Declaración del impuesto vehicular

Se completa desde el archivo que exporta la plataforma estatal. El archivo **solo actualiza unidades que ya existen**: nunca crea vehículos.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `status` | VARCHAR(30) | No | Estado que informa el archivo | — |
| `declaration_date` | DATE | Sí | Fecha de inscripción de la declaración | — |
| `import_batch_id` | BIGINT FK → `import_batches` | No | De qué carga salió | Permite saber con qué archivo llegó cada dato y rehacer una carga equivocada |

**Relaciones.** Apunta a `vehicles` e `import_batches`.

**Ejemplo**

| unidad | estado | declarado | carga |
|---|---|---|---|
| ABC123 | Inscrito | 12/03/2026 | SAT 2026-03 |
| GHI789 | Pendiente | — | SAT 2026-03 |

**Reglas de negocio**

- Solo aplica a autos. En motos y trimotos no corresponde.
- Una unidad sin el documento de identificación vehicular no puede tener declaración: el trámite no existe todavía.
- El cruce se hace por placa. Las filas del archivo que no encuentran unidad quedan registradas como no vinculadas, no se descartan en silencio.

### 11.8 `import_batches` — Cargas de archivos

Toda carga masiva —declaración del impuesto, garantía mobiliaria, migración inicial— queda registrada.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa | — |
| `import_type` | VARCHAR(30) | No | Qué se cargó | — |
| `file_name` | VARCHAR(255) | No | Archivo procesado | — |
| `file_path` | TEXT | No | Dónde quedó guardado | Permite rehacer la carga o auditarla sin pedir el archivo de nuevo |
| `rows_total` / `rows_ok` / `rows_failed` | INTEGER | No | Cuántas filas trajo, cuántas entraron y cuántas fallaron | — |
| `error_report_path` | TEXT | Sí | Archivo con las filas que no se pudieron vincular | Es el patrón que el cliente ya conoce: se devuelve un archivo con lo que falló para corregirlo y volver a subirlo |
| `imported_by` | BIGINT FK → `users` | No | Quién la ejecutó | — |
| `imported_at` | TIMESTAMPTZ | No | Cuándo | — |

**Relaciones.** Apunta a `companies` y `users`. Le apuntan las declaraciones del impuesto vehicular y las cesiones a fondos.

**Ejemplo**

| tipo | archivo | filas | entraron | fallaron |
|---|---|---|---|---|
| Declaración del impuesto | sat_2026_03.xlsx | 412 | 408 | 4 |
| Garantía mobiliaria | gm_2026_03.xlsx | 1 250 | 1 250 | 0 |

**Reglas de negocio**

- Ningún archivo auxiliar crea vehículos: solo actualiza los que ya existen.
- Las filas que no encuentran unidad quedan en el archivo de errores, no se descartan en silencio.

### 11.9 El estado de habilitación se calcula

No es un campo: se evalúa en orden y devuelve el primer estado que aplique.

| Orden | Si… | Estado |
|---|---|---|
| 1 | No ha ingresado físicamente | Fuera de tienda |
| 2 | Falta el documento de identificación vehicular | Sin documento vehicular |
| 3 | Es un auto cuyo modelo requiere pantalla y no la tiene instalada | Sin pantalla |
| 4 | No tiene GPS instalado | Sin GPS |
| 5 | No tiene placa | Sin placa |
| 6 | Cumple todo y no se ha entregado | Listo para entrega |
| 7 | Cumple todo y se entregó | Entregado |

**Se corrigió el orden de la guía funcional.** Ahí el documento vehicular se evaluaba después de las placas, pero registrar una placa exige tenerlo, de modo que esa rama nunca se ejecutaba; y el resultado que devolvía no era un nombre de estado sino una frase, así que la columna quedaba vacía. Evaluarlo antes y darle nombre resuelve las dos cosas.

**Reglas de negocio**

- Se recalcula en cuanto cambia cualquiera de sus condiciones. No se guarda ni se edita a mano.
- Las tres condiciones que impiden vender son las que el cliente enumeró: estar en la empresa, tener placa y tener GPS. Los documentos no bloquean, salvo el vehicular.
- La fecha en que la unidad llegó a *listo para entrega* sale del historial de estados, no de un campo que alguien escribe.
- **La fecha de entrega también sale del historial**: es el momento en que la unidad pasó a Activo, que es la transición que el flujograma llama entrega al cliente. De ahí la toman el estado *Entregado* y el cálculo de la brecha de días. No hay un campo de fecha de entrega y no debe haberlo, porque sería un segundo lugar donde el dato puede discrepar.

### 11.10 Candidatos del bloque 9

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `checksum` | `vehicle_documents` | Que haga falta detectar si un archivo fue reemplazado por otro distinto |
| `reminder_days` | `document_types` | Que se configure con cuánta anticipación avisar de cada vencimiento |
| `insurance_policies` y su cobertura | — | Que se contrate seguro vehicular de flota, donde una póliza cubre muchas unidades con vigencias distintas |

---

## 12. Bloque 10 · Fondos y garantía mobiliaria

Los vehículos se ceden en garantía a un fondo, y lo que interesa es **a cuál y desde cuándo**. La garantía no se retira hasta que termina el crédito.

### 12.1 `funds` — Fondos

Catálogo administrable. Hoy hay uno; se esperan más.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que cede | — |
| `code` | VARCHAR(30) | No | Código corto: `PF_MILLENNIUM`, `BID` | — |
| `name` | VARCHAR(150) | No | Nombre del fondo | — |
| `fund_type` | VARCHAR(30) | No | Fideicomiso, fondo propio, banco | — |
| `is_active` | BOOLEAN | No | Si se siguen cediendo unidades a él | — |

**Relaciones.** Apunta a `companies`. Le apunta `vehicle_fund_assignments`.

**Ejemplo**

| code | name | tipo | activo |
|---|---|---|---|
| `PF_MILLENNIUM` | PF Millennium | Fideicomiso | Sí |
| `BID` | Banco Interamericano de Desarrollo | Banco | Sí |

**Reglas de negocio**

- El fondo es una fila, no un valor escrito en el código. Cuando entre otro, se carga y ya.
- Un fondo no se borra: se desactiva. Las unidades que estuvieron cedidas siguen siendo explicables.

### 12.2 `vehicle_fund_assignments` — Cesiones

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad cedida | — |
| `fund_id` | BIGINT FK → `funds` | No | A qué fondo | — |
| `period` | DATERANGE | No | Desde cuándo hasta cuándo estuvo cedida. Abierto = sigue cedida | Responde a qué fondo respaldaba una unidad en una fecha pasada, que es lo que preguntaría ese fondo o su auditor |
| `import_batch_id` | BIGINT FK → `import_batches` | Sí | De qué carga salió | Vacío si se asignó a mano |
| `notes` | TEXT | Sí | Detalle | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | — |

**Ejemplo**

| unidad | fondo | desde | hasta |
|---|---|---|---|
| ABC123 | PF Millennium | 01/03/2025 | — |
| DEF456 | PF Millennium | 01/03/2025 | 01/07/2026 |
| DEF456 | BID | 01/07/2026 | — |

**Reglas de negocio**

- Una unidad no puede estar cedida a dos fondos a la vez, ni con períodos que se crucen en el pasado.
- La cesión no se retira hasta que termina el crédito. Que un cliente incumpla no la levanta.
- **El archivo que se carga actualiza la historia, no la reemplaza.** Que una placa desaparezca del último archivo cierra su cesión con esa fecha; no borra que estuvo cedida. La guía funcional lo trata como una fotografía que sobrescribe, y así se pierde desde cuándo estaba cedida cada unidad, que es justamente lo que se pidió conservar.
- El archivo solo actualiza unidades existentes: nunca crea vehículos.

### 12.3 Candidatos del bloque 10

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `contract_reference` | `vehicle_fund_assignments` | Que haga falta referenciar el contrato de cesión con el fondo |
| Acceso de terceros | — | Que un fondo o su auditor entre al sistema. La asignación de accesos ya admite dimensiones opcionales: se agregaría el fondo como una más |


---

## 13. Bloque 11 · Bitácora de cambios

Las tablas de historial que ya existen —estados del vehículo, cesiones a fondos, equipamiento, accesos, bloqueos de cuenta— responden **qué pasó**. Ninguna responde **qué hizo alguien con un registro**: si corrigió el motivo de un bloqueo, movió una fecha de cesión o cambió una placa mal digitada.

Esta tabla cubre eso. Son dos auditorías distintas y ninguna reemplaza a la otra.

### 13.1 `change_log` — Registro de modificaciones

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `entity` | VARCHAR(50) | No | Qué tabla se modificó | — |
| `entity_id` | BIGINT | No | Qué registro | No es una llave foránea: la tabla apunta a cualquier entidad, y una llave a todas sería imposible de declarar |
| `company_id` | BIGINT FK → `companies` | Sí | A qué empresa pertenece el registro modificado | Permite que cada empresa vea solo su propia bitácora. Vacío para cambios sobre catálogos generales |
| `field_name` | VARCHAR(60) | No | Qué campo cambió | Un cambio que toca tres campos deja tres filas: así se puede buscar quién tocó una placa sin leer el resto |
| `old_value` | TEXT | Sí | Valor anterior | Vacío cuando el campo estaba sin llenar |
| `new_value` | TEXT | Sí | Valor nuevo | Vacío cuando se borró el contenido |
| `reason` | TEXT | Sí | Por qué se hizo el cambio | Obligatorio en los cambios marcados como delicados: corregir un VIN, una placa o el monto de una factura |
| `changed_by` | BIGINT FK → `users` | No | Quién lo hizo. El usuario del sistema cuando fue automático | — |
| `changed_at` | TIMESTAMPTZ | No | Cuándo | — |

**Relaciones.** Apunta a `users` y opcionalmente a `companies`. No le apunta nadie.

**Ejemplo**

| entidad | registro | campo | valor anterior | valor nuevo | motivo | quién | cuándo |
|---|---|---|---|---|---|---|---|
| `vehicles` | 1 | `plate` | ABC124 | ABC123 | Error de digitación | Alejandro | 14/03/2026 09:12 |
| `vehicle_fund_assignments` | 87 | `period` | `[2025-03-01,)` | `[2025-04-01,)` | Fecha real de cesión | Tere | 20/03/2026 16:40 |
| `user_access_suspensions` | 12 | `reason` | licencia | baja | Corrección | Tere | 02/04/2026 11:05 |

Los valores de fecha se guardan en ISO 8601 (§2.6): un período abierto que empieza el 1 de marzo de 2025 es `[2025-03-01,)`. La columna *cuándo* está en formato de pantalla solo para leer el ejemplo.

La primera fila es el caso que más importa: la placa es la llave con que se cruzan los archivos del impuesto vehicular y de garantía mobiliaria, y corregirla sin dejar rastro haría imposible explicar por qué un cruce dejó de funcionar.

**Reglas de negocio**

- La bitácora **no se edita ni se borra**. Un registro equivocado se corrige con un cambio nuevo, que deja su propia fila.
- Se registra el cambio de valores, no la consulta. Quién vio qué es otra cosa y hoy no se pide.
- Los cambios marcados como delicados exigen motivo: corregir un VIN, una placa, el monto de una factura o una fecha ya reportada.
- No guarda contraseñas ni el segundo factor, aunque cambien.
- Cada empresa ve solo la bitácora de sus propios registros.

**Qué no cubre.** Las altas y las bajas no dejan filas aquí: una fila nueva ya registra quién la creó, y nada se borra en este sistema. La bitácora es para las modificaciones.

### 13.2 Qué se audita en cada lugar

| Pregunta | Dónde se responde |
|---|---|
| ¿Por qué esta unidad estuvo un mes parada? | Historial de estados |
| ¿A qué fondo respaldaba en marzo? | Cesiones a fondos |
| ¿Qué GPS tenía instalado el año pasado? | Equipamiento |
| ¿Qué accesos tenía esta persona cuando registró esa factura? | Asignaciones de rol y bloqueos de cuenta |
| ¿Quién cambió esta placa y por qué? | Bitácora de cambios |

### 13.3 Candidatos del bloque 11

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `ip_address` y dispositivo | `change_log` | Que se exija trazabilidad de seguridad y no solo de negocio |
| Registro de consultas | — | Que haya que auditar quién vio datos sensibles, no solo quién los cambió |


## 14. Decisiones

Queda una decisión abierta, con su momento fijado: las longitudes de los documentos tributarios de Colombia y México (§4.7), antes de registrar la primera empresa de cada país (D-52). Lo que falta para construir son datos, no definiciones: la matriz de permisos por rol, los datos maestros reales —razón social y documento tributario de la empresa, sedes con su código de establecimiento, marcas, proveedores, colaboradores—. El archivo oficial de divisiones territoriales de Perú ya está cargado (§4.5).

### Decisiones ya cerradas

| # | Decisión tomada |
|---|---|
| D-1 | El país se identifica internamente con un número; el código internacional se guarda aparte como valor único |
| D-9 | El VIN tiene 17 caracteres exactos |
| D-15 | Los catálogos de flota son del sistema, no de cada empresa: un modelo se carga una vez y sirve para todos los países |
| D-16 | La placa es una columna del vehículo, sin tabla de formatos: se normaliza, no se repite por país y se valida solo por longitud y caracteres |
| D-27 | La brecha de días se recalcula siempre con la mejor información disponible; congelarla fijaría un número calculado sobre datos incompletos |
| D-28 | Los documentos del vehículo son una tabla con su archivo adjunto. Solo uno vigente por tipo; el anterior se conserva |
| D-29 | Las llaves llevan dos códigos: la etiqueta que asigna la empresa, única, y el serial de fábrica, que puede repetirse. El grupo es la caja donde se guarda |
| D-30 | El archivo de garantía mobiliaria actualiza la historia de cesiones, no la reemplaza |
| D-31 | La diferencia por inicial mide lo que Millennium no desembolsó porque el cliente pagó al concesionario. Se calcula del lado de compras y se valida contra la inicial registrada en el contrato |
| D-32 | Existe un registro público de cuentas, habilitable por la variable de entorno `IAM_PUBLIC_SIGN_UP_ENABLED` y apagado por defecto. Registra el perfil de colaborador y la cuenta en la misma operación, sin paso de validación, y se rechaza si el documento ya existe en esa empresa o si el correo ya tiene cuenta. La cuenta nace sin roles, y la cuenta y el perfil figuran creados por `SYSTEM` (§5.2) |
| — | Los identificadores internos y sus llaves foráneas son `BIGINT`, el `Long` del código (§2.2) |
| D-33 | Las sesiones se guardan en el servidor y el navegador solo recibe una cookie `HttpOnly`. Se cierran por 30 minutos de inactividad, a las 8 horas en cualquier caso, al cerrar sesión, al cambiar la contraseña (las demás) y al bloquear la cuenta (§5.12) |
| D-34 | El primer superadministrador lo crea la aplicación al arrancar, desde variables de entorno y con contraseña provisional; no la migración (§5.2) |
| D-35 | Las órdenes de compra se emiten con plantillas. Todas tienen los mismos campos; sus valores cambian según la línea de negocio y el tipo de orden (vehículos, taller, otros), que es un catálogo administrable. Los datos del emisor se proponen desde la empresa y se pueden editar; los contactos de atención y soporte son texto (§9.2) |
| D-36 | Cada plantilla tiene su serie y cada serie su correlativo, que se reinicia cada año: `OC-LBT-2026-0001`. El número se asigna al crear la orden, con el año en la hora local del país de la empresa, y no se reutiliza (§9.2) |
| D-37 | Las líneas de una orden de compra admiten conceptos que no son vehículos, igual que las de factura: las órdenes de taller no compran modelos (§9.2) |
| D-38 | Toda cuenta pertenece a un colaborador y nace vinculada a él: un administrador da acceso a un colaborador ya registrado —vinculándolo a su cuenta si la persona ya tiene una por otra empresa—, o el registro público crea el colaborador y la cuenta juntos (D-32). Para tener un rol en una empresa, la cuenta debe estar vinculada a un colaborador activo de esa empresa; al registrar su salida se retiran esos roles, y si no le queda ningún colaborador activo la cuenta se bloquea. Excepciones: `SYSTEM`, que no es una persona, y el primer superadministrador, que hasta vincularse a su perfil configura el sistema pero no puede dar acceso (§5.2, §5.3) |
| D-39 | Vocabulario: **rol** es el conjunto de permisos de una cuenta; **puesto**, el trabajo del colaborador, informativo; **perfil**, sus datos personales. El documento no usa "cargo" (§5.1) |
| D-40 | Fechas: los momentos son `TIMESTAMPTZ` en UTC (`_at`), las fechas de calendario `DATE` (`_date`) y los períodos rangos con fin excluido. Todo intercambio usa ISO 8601, con los momentos en UTC; en una hoja de cálculo exportada, los momentos van en una sola zona nombrada en el encabezado (D-49); "hoy" se calcula en la zona de la sede, o del país de la empresa, o `America/Lima`; `dd/mm/aaaa` solo en pantalla (§2.6) |
| D-41 | Un rol se retira con uno de cuatro motivos: `retiro`, `caducidad`, `baja_colaborador` o `baja_cuenta` (§5.7) |
| D-42 | Las columnas de registro las llena el sistema: `created_at` y `updated_at` al guardar, y `created_by` y `updated_by` con la cuenta de la sesión. Cuando no hay sesión —el registro público, el arranque— el proceso registra a `SYSTEM` de forma explícita; el sistema nunca lo supone, para que un cambio que perdió su sesión por error no quede a nombre de nadie. Solo las llevan las tablas cuyo cuadro las pide: las seis tablas del bloque 1 —`currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses` y `tax_id_types`— registran fechas pero no autor (§2.2) |
| D-43 | Los catálogos generales —monedas, países, divisiones territoriales, tipos de documento tributario— los lee cualquier cuenta con al menos un rol vigente, porque toda pantalla con un selector los necesita; solo el superadministrador los crea o modifica. Tener sesión sin rol no alcanza (§5.2). Cuando existan los permisos, la lectura pasa a un permiso propio |
| D-44 | Los tipos de documento tributario son localización del país: pertenecen al bloque 1 y al contexto `location`, junto con los escalones y las divisiones territoriales. Los ERP que operan en varios países los tratan igual. Llevan `is_active` y fechas, sin autor (§4.7) |
| D-45 | Los escalones territoriales forman parte del país: se guardan y se validan con él. Se declaran en orden, sin saltos, y no se borran ni se retiran; solo se corrige su nombre (§4.4) |
| D-46 | Los tipos de cambio por día y la moneda de grupo quedan diseñados en §4.8 y se construyen cuando aparece quien los usa: los tipos de cambio justo antes de compras (paso 7) y la moneda de grupo con los reportes (paso 11). La moneda de grupo es USD |
| D-47 | Los dos códigos ISO de un país nombran un mismo país de ISO 3166. Se corrigen mientras el país no tenga escalones, direcciones ni tipos de documento tributario, lo que también excluye divisiones y empresas; desde entonces quedan fijos. `location` lo decide sin consultar a `organization`, para no volver al ciclo que D-3 eliminó. Es la única excepción con nombre a la regla de que los códigos de normas externas no se editan; extenderla requiere una decisión nueva (§2.4, §4.2, D-48) |
| D-48 | El `code` de un catálogo queda fijo desde que algo lo usa: uno propio se corrige mientras nada dependa de él, y los de normas externas no se editan nunca. La única excepción son los dos códigos ISO del país (D-47), que se validan contra ISO 3166 y se corrigen mientras el país no tenga nada colgado. Extenderla a otro código externo —moneda, sigla, ubigeo— requiere una decisión nueva: la moneda, por ejemplo, la usa la empresa como moneda funcional aunque sea de otro país, y `location` no puede saberlo sin consultar a otro contexto (§2.2, §2.4) |
| D-49 | En una hoja de cálculo exportada, la fecha es una celda de fecha con formato `AAAA-MM-DD` y el momento una celda de fecha y hora con formato `AAAA-MM-DD hh:mm:ss`, en una sola zona por archivo, nombrada en el encabezado de la columna. La zona sale de la regla de "hoy" aplicada al alcance de la exportación: la sede, si no el país de la empresa, si no `America/Lima`. Los archivos para otros sistemas usan ISO 8601 con los momentos en UTC y `Z` (§2.6) |
| D-50 | La sede no lleva tipo: ninguna regla ni reporte depende hoy de la función del local, y la clase de lugar ya la dice `locations.location_type` (D-6). `branch_type` queda como candidato, con el criterio que lo admitiría (§6.4, §6.6) |
| D-51 | El `public_id` sale por las direcciones web y las integraciones por API. Los archivos que genera el sistema llevan códigos de negocio y nombres, nunca el `id` interno ni el `public_id`. Excepción: un archivo diseñado para editarse y volver a subirse con el fin de actualizar registros existentes lleva además el `public_id` como llave estable. El criterio es el uso de ida y vuelta, no quién recibe el archivo (§2.4) |
| D-52 | Un paso se cierra con el alcance de los países que operan. Hoy opera solo Perú (D-5), así que el bloque 1 se cerró con su ubigeo cargado. Lo que es propio de otro país —su lista oficial de divisiones (D-2) y las longitudes de sus documentos tributarios (§4.7)— se resuelve antes de registrar su primera empresa |
| D-53 | La división de una dirección es obligatoria cuando el país de la dirección tiene sus divisiones cargadas; solo puede faltar en un país sin ellas, como la dirección de un proveedor extranjero (§4.6) |
| D-54 | `geo_divisions.parent_level` es una columna que calcula la base (el escalón menos uno) y no guarda un dato del negocio: existe solo para que la llave hacia la división superior incluya su escalón (§4.5). Es la única excepción a la regla de no guardar lo derivable, y la aplicación nunca la escribe |
| D-10 | Los datos personales van en `employees`, el registro de colaboradores. `users` guarda solo lo necesario para entrar |
| D-25 | Colaboradores y cuentas son tablas distintas: hay personal sin acceso al sistema, y el usuario del sistema no es una persona |
| D-26 | Un contrato no puede quedarse sin asesora: al retirar a alguien se exige el reemplazo en la misma operación |
| D-21 | La factura puede indicar su orden de compra, de forma opcional: con el vínculo el pendiente se calcula, sin él la orden se cierra a mano |
| D-22 | La línea de factura admite conceptos que no son vehículos, aunque hoy solo se usen líneas de vehículo |
| D-23 | Solo existen notas de crédito. No se modelan notas de débito |
| D-24 | La tasa de impuesto se guarda en cada factura, no como constante: cambia por ley y difiere entre países |
| D-18 | No se generan correlativos nuevos: el número del Excel se conserva solo como referencia de las unidades migradas |
| D-19 | Los estados, las transiciones permitidas y los motivos con su tolerancia son tablas, no listas fijas en el programa |
| D-20 | Al terminar el contrato la unidad se traspasa al cliente: se registra la fecha y el motivo, y el propietario vigente se deduce |
| D-17 | Los catálogos de flota se mantienen aunque hoy tengan pocas filas: habrá más tipos de equipamiento, más tipos de vehículo, y operación en otro país dentro de un año |
| D-11 | La asesora se asigna al contrato, no al cliente. Quien originó la venta queda fijo; quien atiende puede cambiar |
| D-14 | El alcance de sede limita dónde se actúa, no qué se ve: el stock es visible para toda la empresa |
| D-2 | Se cargan completas las listas oficiales de divisiones territoriales |
| D-3 | No se guarda si un país está operativo: se deduce de si tiene alguna empresa activa. Apagar la operación se hace en `companies.is_active` |
| D-8 | Una mudanza crea una dirección nueva; una corrección modifica la existente. El contrato conserva el domicilio de la firma |
| — | Las sedes emiten comprobantes: `branches` lleva el código de establecimiento tributario |
| D-4 | "Perfil" son los datos personales de la persona; van en `employees`, no en `users` |
| D-5 | Una sola empresa por país. Lizzo y Lima Bikes son marcas, no empresas |
| D-6 | Los patios arrendados y las tiendas de dealers son ubicaciones, no sedes |
| D-7 | Los roles se crean desde la pantalla; `company_id` los acota a una empresa |
| D-12 | Registrar una baja revoca automáticamente todos los accesos |
| D-13 | Las direcciones web usan siempre UUID; los catálogos de estándar externo viajan por código dentro de los mensajes |
| — | No existe la entidad `organizations`: el agrupador de países son las empresas |
