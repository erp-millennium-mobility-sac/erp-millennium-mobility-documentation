# Diccionario de datos — ERP Millennium Mobility

**Versión:** 1.25 · **Motor:** PostgreSQL 18 · **Estado:** módulo de stock completo. Bloques 1 a 11 cerrados; ventas y contratos fuera del alcance actual. 51 tablas más `role_permissions`, la tabla de unión de roles y permisos, y las tablas técnicas de §2.7

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
| Los derivados no se persisten | Columnas generadas `VIRTUAL` (por defecto en PG 18) o vistas. Un derivado almacenado se desincroniza. Hay tres excepciones con nombre: el estado actual y el último kilometraje del vehículo, que se guardan además en la unidad (§8.4, §11.1), y `geo_divisions.parent_level`, que es `STORED` porque entra en una llave foránea (D-54) |
| **Todo monto viaja con su moneda** | O la fila tiene `currency_id NOT NULL` junto al monto, o la hereda del encabezado por llave foránea. Un monto sin moneda no se guarda |
| **Toda conversión viaja con su tipo de cambio, y la fecha de ese tipo de cambio es la del documento** | El tipo de cambio de una factura es el de su fecha de emisión; el de un pago, el de su fecha de pago. No hace falta una columna aparte, pero la regla debe estar escrita o alguien usará el tipo de cambio del día en que registra |
| Montos: `NUMERIC`, `NOT NULL DEFAULT 0` | Un `NULL` en un monto se propaga por toda la aritmética y llega a pantalla como celda vacía |
| Vigencias: un rango en la columna `period`, con el inicio incluido y el fin excluido; fin vacío = vigente (§2.6) | Una vigencia se cierra, nunca se borra |
| Intervalos sin solapamiento: `daterange` o `tstzrange` + `WITHOUT OVERLAPS` (PG 18). Cuando hay filas que no deben contar —un bloqueo cancelado— o la regla compara un dato que se deduce de otro —la línea de tiempo de un bloqueo—, una exclusión `EXCLUDE USING gist` con su condición, que necesita `btree_gist` (D-67) | Impide dos períodos cruzados incluso en el pasado. `WITHOUT OVERLAPS` no admite dejar filas afuera ni comparar una expresión; la exclusión sí, y protege lo mismo |
| Los identificadores que fija una norma externa —documentos tributarios y de identidad, placas, códigos de ubigeo— se normalizan al guardar: `TRIM` + `UPPER`, sin guiones, espacios ni puntos. Los códigos propios, como el de una sede o el de una ubicación, solo `TRIM` + `UPPER`, y conservan sus guiones (D-86). Los correos electrónicos, `TRIM` + `LOWER` | `UNIQUE` no distingue `ABC-123` de `ABC123`, ni `Tere@…` de `tere@…`. Un identificador externo se escribe de varias formas según el papel de donde se copia; un código propio lo elige la empresa, y su guion es parte de él: `LIM-CENTRAL` |
| Centinelas (`-`, `NO TIENE`, `NO CORRESPONDE`) no se almacenan | Se generan al exportar. Guardados, obligan a parsear texto en cada cálculo |

La regla de moneda no es teórica. La guía funcional tuvo que enunciar como norma que una nota de crédito debe estar en la misma moneda que su factura (decisión A-06). Esa regla no debería necesitar enunciarse: debería ser imposible violarla.

**Tablas técnicas.** Las que administran las librerías no siguen estas convenciones, y las propias de la plataforma no pertenecen a ninguna empresa: las dos clases están en §2.7.

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
- El `code` de un catálogo propio se puede corregir mientras nada dependa de él; una vez que hay registros que lo usan, o una regla del programa que lo lee, queda fijo; un código que lee el programa queda fijo desde la semilla que lo crea (D-74). Los códigos de normas externas no se editan nunca. La única excepción son los dos códigos ISO del país (D-47): se validan contra la lista de ISO 3166 y se corrigen mientras el país no tenga nada colgado. Extender esa excepción a otro código externo —moneda, sigla, ubigeo— requiere una decisión nueva (D-48).

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
- Los usuarios no pertenecen a una empresa. Quien tiene `seguridad.ver_cuentas` por una asignación de empresa —el administrador de esa empresa— ve las cuentas vinculadas a colaboradores de esa empresa —también las de quienes ya salieron, para comprobar que la salida les quitó el acceso—, con los roles vigentes que tienen en ella y los de alcance sistema; con alcance sistema, las ve todas (D-122). Verlas no es modificarlas: dar acceso, otorgar y retirar roles, bloquear cuentas y restablecer contraseñas exigen sus permisos con alcance sistema, así que ningún rol de una empresa los hace; hoy son del administrador de accesos y del superadministrador (D-57, D-122).
- Los permisos de los colaboradores actúan donde su asignación dice (D-122): `seguridad.gestionar_colaboradores` y `seguridad.registrar_salidas_y_regresos` por un rol de empresa, en toda la empresa; por uno de sede, en esa sede. Un colaborador sin sede es de toda la empresa, y moverlo de sede exige actuar en la de origen y en la de destino.
- El stock es visible para toda la empresa. La sede limita dónde se puede actuar, no qué se ve.
- Un rol de sede actúa sobre las unidades que están en la ubicación propia de su sede; las que están en una ubicación de terceros, o todavía sin ubicación, las tocan los roles de empresa (D-101). La marca de una asignación acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, desde los vehículos (D-102).
- Hay campos que no ve cualquiera aunque vea la fila. Cuando falta el permiso, el campo no sale de la base: ocultarlo en la pantalla no alcanza.
- El acceso total es una excepción explícita evaluada con las mismas reglas, no una cuenta que se saltea el control.
- **La frontera entre empresas la defiende también la base de datos** (D-66). La aplicación filtra cada consulta por las empresas que alcanza quien la hace. Además, en cada tabla que lleva `company_id` —y en `companies`, por su propio `id`—, la base deja ver y escribir solo las filas de esas empresas. Si algún día una consulta olvida su filtro, devuelve menos filas, nunca las de otra empresa. Un rol de alcance sistema ve todas; una operación sin alcance no ve ninguna.
- La tabla de direcciones queda acotada igual que el resto antes de guardar la primera dirección de un proveedor (paso 7): desde ahí guarda direcciones de terceros de cada empresa, y después el domicilio de los clientes, que sí es un dato sensible (D-80). Hasta entonces se lee solo a través de su dueño, que ya está acotado.

**Nota técnica.** Es Row Level Security de PostgreSQL, habilitada y forzada (`ENABLE` y `FORCE ROW LEVEL SECURITY`) para que tampoco el dueño de las tablas la saltee. Hoy cubre `companies`, `commercial_brands`, `branches`, `locations`, `employees`, `user_roles` y `employee_account_link_events`, y cubrirá cada tabla con `company_id` que se cree. Hay dos cuentas de base de datos: la dueña, que solo aplica las migraciones, y la de la aplicación, que no es dueña de nada, no puede saltear la regla y no tiene permiso de borrar. Al comenzar cada transacción —nunca al abrir la conexión—, la aplicación fija desde un solo lugar, con `set_config(…, true)`, quién actúa y qué alcanza: `app.access_scope` (`SYSTEM` o `SCOPED`), `app.company_ids` y `app.user_id`. Un valor que falta no ve nada. El arranque y los procesos automáticos actúan con alcance `SYSTEM` de forma explícita, igual que firman como `SYSTEM` (D-42). `user_roles` le muestra además a cada cuenta sus propias asignaciones y las de alcance sistema, para que pueda calcularse su alcance. `employee_account_link_events` lee más estrecho: sus filas las ven solo una cuenta con `SUPERADMIN` vigente y los procesos con alcance `SYSTEM` sin cuenta, no el administrador de accesos ni los roles de una empresa, porque guardan datos de personas de otras empresas (§5.4). Las pocas comprobaciones que necesitan ver más allá de las empresas de quien opera —si la cuenta tiene un colaborador activo en otra empresa, o cuántos superadministradores quedan (§5.9)— se hacen con una elevación a `SYSTEM` acotada a esa comprobación, que solo devuelve el dato.

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
| Las zonas horarias se guardan con el nombre IANA de una región, de la forma Área/Lugar: `America/Lima`, `America/Bogota`. No se aceptan los desfases fijos (`Etc/GMT+5`, `UTC`) ni los alias sin continente (`US/Eastern`) | Un nombre de región, a diferencia de un desfase fijo como `-05:00` o `Etc/GMT+5`, sigue los cambios de horario que decida cada país. Los alias sin continente son nombres antiguos que IANA conserva solo por compatibilidad |
| "Hoy" y las fechas de operación se calculan en la zona de la sede donde ocurre el hecho. Sin sede, en la del país de la empresa del registro; si el registro no pertenece a ninguna empresa —una cuenta, un bloqueo—, en `America/Lima` | Una factura registrada a las 11 de la noche en Lima es de ese día en Lima, aunque en UTC ya sea el siguiente. De esto dependen la antigüedad de 20 días de una factura, los vencimientos y los cortes del día |
| Una hora local futura o que se repite —un corte diario a las 18:00, un horario de atención— no es un momento: se guarda como hora local junto con la zona de la sede, y se convierte a UTC recién al usarla. Hoy ninguna tabla lo necesita | Si un país cambia su horario, una hora futura guardada en UTC quedaría corrida. Lo que ya ocurrió sí se guarda en UTC, porque ese momento no cambia |
| El formato de pantalla es `dd/mm/aaaa` y `dd/mm/aaaa hh:mm`, solo al mostrar | Nunca se guarda ni se transmite así. Los ejemplos de este documento usan el formato de pantalla para que se lean fácil |

### 2.7 Tablas técnicas

Sostienen la plataforma y no guardan un dato del negocio, así que no figuran en ningún bloque. Son de dos clases.

**Las que administran las librerías** —el historial de migraciones (`flyway_schema_history`), el registro de eventos entre contextos (`event_publications`) y las sesiones (`spring_session` y `spring_session_attributes`, §5.12)— conservan la estructura que exige su librería y no siguen estas convenciones: no llevan `public_id` ni forman parte del modelo del negocio.

**Las propias de la plataforma** son de un módulo de la plataforma, porque las usan todos los contextos y no son de ninguno. Siguen las convenciones de §2.2 —`id`, `public_id`, columnas de registro, nada se borra—, pero no pertenecen a una empresa. Hoy hay una: `generated_documents`, del módulo `documents` (D-125). Hasta D-125 era de `shared`.

#### `generated_documents` — Documentos generados

Cada documento que el sistema genera para que alguien lo descargue —una lista exportada a una hoja de cálculo y, cuando lleguen, los PDF— con su estado y dónde quedó el archivo (D-106). Existe para que un mismo pedido no genere dos veces el mismo archivo: si dos personas con el mismo alcance exportan la misma lista con los mismos criterios, la segunda recibe el archivo de la primera, o espera a que termine.

Es del módulo `documents` (D-125), que genera cada documento, lo guarda, lo vence y lo entrega en `/documents/{id}`. Ningún contexto lee ni escribe esta tabla: los que exportan piden su documento por el contrato del módulo (`documents::acl`), y nunca tocan el agregado.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `document_key` | CHAR(64) | No | La huella del pedido: el SHA-256 de su tipo, su formato, sus criterios ya normalizados, su alcance, el idioma, la zona de los momentos y la versión de los datos que lee | Dos pedidos iguales dan la misma huella, y por ella el segundo encuentra el documento del primero. La página y el tamaño de página no entran: un documento lleva siempre todas las filas |
| `scope_key` | CHAR(64) | No | La huella del alcance efectivo de quien lo pidió: las empresas en que actúa —o `SYSTEM`—, sus restricciones de sede y de marca, y las banderas de visibilidad propias del documento, como si lleva los datos personales de los colaboradores. Nunca la cuenta | Es la guarda de la tabla: cada lectura la vuelve a calcular para quien pregunta, y un documento de otro alcance no existe para él. Que no lleve la cuenta es lo que deja reutilizar el documento entre dos personas con el mismo alcance |
| `kind` | VARCHAR(60) | No | Qué documento es: `currencies`, `vehicle-colors` | Cada tipo declara cómo se genera, su alcance y la versión de sus datos |
| `format` | VARCHAR(10) | No | `xlsx` o `pdf` | — |
| `status` | VARCHAR(12) | No | `generando`, `listo`, `fallido` o `vencido` | Uno `generando` hace esperar al pedido igual; uno `listo` y vigente se reutiliza; uno `vencido` ya no se entrega |
| `file_base_name` | VARCHAR(100) | No | El nombre del archivo, sin fecha ni extensión y sin tildes: `monedas` | La fecha se agrega al descargar, la de hoy en Lima, para que todos los navegadores lo guarden igual |
| `content_type` | VARCHAR(120) | No | El tipo del archivo, el que recibe el navegador | — |
| `storage` | VARCHAR(10) | No | Dónde quedó el archivo: `base`, en la columna `content`, o `spaces`, en DigitalOcean Spaces | Nunca en la memoria ni en el disco del proceso: en App Platform el disco se pierde al reiniciar, y en un despliegue conviven la instancia vieja y la nueva, así que el pedido siguiente puede llegar a la otra (D-106) |
| `storage_key` | VARCHAR(255) | Sí | La llave del objeto en Spaces. Vacía cuando el archivo está en la base | — |
| `content` | BYTEA | Sí | El archivo, cuando se guarda en la base. Se vacía al vencer | — |
| `size_bytes` | BIGINT | Sí | Cuánto pesa el archivo. Vacío mientras se genera | — |
| `generated_at` | TIMESTAMPTZ | Sí | Cuándo quedó listo | — |
| `valid_until` | TIMESTAMPTZ | Sí | Hasta cuándo se reutiliza | Pasado ese momento, un pedido igual genera uno nuevo, aunque los datos no hayan cambiado |
| `lease_until` | TIMESTAMPTZ | Sí | Hasta cuándo la generación en curso se da por viva | Si la instancia que generaba se apagó en un despliegue, otra toma la generación cuando vence, en vez de esperar para siempre |
| `failure_code` | VARCHAR(60) | Sí | El código de error de una generación que falló | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Cuándo se pidió y quién lo pidió primero | Quien lo reutiliza después no queda registrado: el documento es del alcance, no de la persona |
| `updated_at` | TIMESTAMPTZ | No | Último cambio de estado. Sin `updated_by`: el estado lo cambia el sistema —la generación, el vencimiento—, no una persona | — |

**Relaciones.** Apunta a `users`, por sus columnas de registro. No le apunta nadie.

**Reglas de negocio**

- **Un documento se genera una sola vez por huella.** A lo sumo hay un documento `generando` o `listo` por `document_key` (`generated_documents_active_key_uq`, un índice único parcial). Cuando dos pedidos iguales llegan a la vez, la base deja reclamar la huella a uno solo; el otro espera a que termine.
- **La versión de los datos va en la huella.** Cada tipo de documento la calcula con una consulta sobre las tablas que lee —cuántas filas ve, la última transacción que insertó o cambió una de ellas (`xmin`) y, donde la tabla lo tiene, su último `updated_at`; lo que resuelve de otro contexto, con la versión que da su fachada—, con el alcance de quien pregunta. Cada contexto nombra sus propias tablas en su capa de infraestructura, nunca en la de aplicación. La consulta lee cada tabla completa en cada pedido, no solo un índice; su costo se mide contra un umbral de 50 ms por exportación (convenciones de la API, §8.3). <!-- verificar al integrar fase 1: cifras de H3 (P3) y, si pasa el umbral, la decisión D-128 --> `xmin` cubre también las tablas sin `updated_at` y los cambios que no lo tocan, así que no hace falta el último `id` en las tablas que solo reciben filas. Como en este sistema nada se borra, cualquier alta, cambio o desactivación cambia la versión, y el pedido siguiente genera un documento nuevo.
- **No lleva `company_id` ni Row Level Security**, a diferencia de toda tabla que se consulta por sí sola (§2.5). Un documento no es de una empresa: el de un rol de alcance sistema abarca todas, y el de un catálogo general, ninguna. Su guarda es `scope_key`, que la aplicación comprueba en cada lectura: quien pregunta por un documento de otro alcance recibe la misma respuesta que si no existiera. Es una excepción a esa regla de §2.5, por ser una tabla técnica.
- **Las filas no se borran.** La cuenta de la aplicación lee, inserta y actualiza, y no tiene permiso de borrar, como en el resto del sistema (§2.5). Al vencer, la fila pasa a `vencido` y su `content` se vacía: el archivo deja de ocupar espacio, y la fila queda como constancia de qué se generó, para qué alcance y cuándo. Un proceso programado lo hace cada 15 minutos por omisión (`documents.expire-check-interval`), en una sola instancia a la vez; ningún pedido lo espera, porque el que encuentra un documento listo y vencido lo vence él mismo. Los archivos de Spaces los borra la regla de ciclo de vida del bucket, no la aplicación.
- Un documento no se edita: un pedido con otros criterios, otro alcance u otros datos es otro documento.

---

## 3. Orden de construcción

El orden lo fijan las dependencias, no la importancia del negocio. Una tabla no puede escribirse antes que aquellas a las que referencia.

Cada bloque pertenece a un contexto delimitado, que es la unidad de código: `location`, `iam`, `organization`, `vehiclecatalog`, `fleet`, `procurement` y `shared`. Además hay un módulo de la plataforma que no es dueño de ningún bloque: `documents`, con los documentos que genera el sistema y su tabla técnica, `generated_documents` (§2.7, D-125). El mapa de contextos, con quién puede llamar a quién y la estructura de paquetes, se desarrolla en un documento aparte, como propuesta; lo que este diccionario no fija no está decidido. La regla que lo gobierna es una sola: un contexto usa a otro únicamente a través de su fachada, y nunca consulta ni escribe sus tablas; la única excepción son los modelos de lectura declarados como tales, que solo leen.

| Bloque | Contenido | Depende de | Estado |
|---|---|---|---|
| 1 | Geografía y localización: `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses`, `tax_id_types` | — | **Cerrado** |
| 2 | Seguridad: `users`, `user_access_suspensions`, `roles`, `permissions`, `role_permissions` | — (`roles.company_id` necesita el 3; llegó con la matriz de permisos, D-81, D-98) | **Cerrado** |
| 3 | Empresas: `companies`, `commercial_brands`, `branches`, `locations` | 1, 2 | **Cerrado** |
| 3b | Colaboradores y accesos: `employees`, `user_roles`, `employee_account_link_events`, `advisor_assignments` | 2, 3 | **Cerrado** |
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
| 1 | 1 · Geografía y localización | `location` | Monedas, países con sus escalones, divisiones territoriales, direcciones y tipos de documento tributario | Archivo oficial de divisiones territoriales (ubigeo del INEI) | Construido para Perú: V24 carga el ubigeo del INEI, edición 2022, y cada distrito creado después entra con su propia migración cuando tiene código oficial (D-77). Colombia y México se completan antes de su primera empresa (D-52) |
| 2 | 3 · Empresas | `organization` | Empresas, marcas comerciales, sedes y ubicaciones | Razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales | Construido: empresas, marcas comerciales, sedes y ubicaciones. `roles.company_id` no es de este paso: llega con la matriz de permisos, antes del paso 5 (D-81) |
| 3 | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | Colaboradores; asignaciones por empresa, sede y marca; bloqueos de cuenta; permisos; dar acceso | Colaboradores reales | Construido; los permisos llegaron con el paso 3b: colaboradores con su salida y su regreso, dar acceso, asignaciones por empresa, sede y marca con su caducidad, bloqueos en dos líneas de tiempo, la separación de funciones entre quien registra colaboradores y quien da acceso (D-57) y la frontera entre empresas en la base (D-66). El registro público con perfil no es de este paso: sigue apagado y se construye antes de encender el registro público en un entorno (D-83). Lo que falta está en §5.13 |
| 3b | 2 · La matriz de permisos | `iam`, `shared` | `permissions`, `role_permissions`, `roles.company_id` e `is_active`; la pantalla Seguridad › Roles y permisos; las operaciones y las lecturas de los pasos 1 a 4 exigen un permiso. Con la segunda ronda de la auditoría, Seguridad por permisos, `permissions.category`, `permission_incompatibilities`, la copia de un rol y la pestaña Matriz de permisos (D-117 a D-124, D-126) | La matriz de permisos por rol (§5.6, fijada el 19/10/2026 y extendida a Seguridad el 20/10/2026) | Construido el 19/10/2026 e integrado; la segunda ronda, en construcción desde el 20/10/2026 |
| 4 | 4 · Catálogos de flota | `vehiclecatalog` | Tipos de vehículo, marcas y modelos, colores, equipamiento y a qué tipos aplica | Modelos reales, conciliados entre los dos catálogos que advierte §7.3 | Construido, salvo los datos: faltan los modelos reales conciliados (§7.3) |
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
3. **El bloque 2 se termina con la matriz de permisos, en el paso 3b.** El paso 3 se construyó sin `permissions` ni `role_permissions` porque faltaba el dato: qué permisos lleva cada rol. El 19/10/2026 la matriz quedó fijada (§5.6, D-95 a D-105) y el paso 3b la construye, antes del paso 5: desde él, las operaciones de negocio y las lecturas de datos exigen un permiso, un rol global lee lo que su matriz le da (D-68), y existen los roles propios de una empresa (D-81, D-98). Las cuentas y los accesos siguieron por rol hasta que la segunda ronda de la auditoría, el 20/10/2026, los pasó también a permisos (§5.8 regla 7, D-117).
4. **Desde el paso 5 hay dos ramas independientes:** compras y pagos (pasos 7 y 8), y habilitación y fondos (pasos 9 y 10). Se pueden alternar según la prioridad del negocio; fondos usa las cargas de archivos que llegan con la habilitación.

Un paso se puede construir sin sus datos, con datos de ejemplo; los reales se cargan después.

**A qué contexto pertenece cada bloque**

| Bloque | Contexto |
|---|---|
| 1 · Geografía y localización | `location` |
| 2 · Seguridad | `iam` |
| 3 · Empresas | `organization` |
| 3b · Colaboradores y accesos | `organization`, dueño de `employees`, e `iam`, dueño de `user_roles` y `employee_account_link_events` (D-92, D-82) |
| 4 · Catálogos de flota | `vehiclecatalog` |
| 6 · Vehículo y estados | `fleet` |
| 7 · Compras | `procurement` |
| 8 · Pagos | `procurement` |
| 9 · Habilitación | `fleet` |
| 10 · Fondos | `fleet` |
| 11 · Bitácora y cargas de archivos | `shared` |

Los contextos `fleet` y `procurement` comparten el vehículo y hablan de él distinto: para el primero es una unidad física que se mueve y se habilita; para el segundo, una línea de comprobante con un VIN y un monto. Ninguno consulta las tablas del otro.

`employees` se documenta en §5.3 junto con las cuentas, pero se crea después de `companies` porque depende de ella. Vive en `organization`, y pasa a un contexto propio cuando lleguen los contratos laborales, la planilla o la asistencia (D-92).

**No hay dependencia circular entre `users` y `companies`:** el usuario no pertenece a una empresa, así que se crea antes. Lo que depende de ambas es `user_roles`, que va al final. Lo mismo pasa con `roles.company_id`: los roles del sistema lo tienen vacío, así que `roles` se crea con el bloque 2 y esa columna se agrega después de `companies`, con la matriz de permisos (D-81). El único caso de arranque —el primer usuario, que nadie creó— se resuelve con la semilla de §5.9.

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
| `default_timezone` | VARCHAR(50) | No | La zona horaria del país, con el nombre IANA de una región (`America/Lima`, §2.6). Es el punto de partida al crear una sede nueva | La hora real se guarda en cada sede, porque hay países con varias: México tiene cuatro. Si se usara una sola por país, los vencimientos y los cortes del día saldrían corridos varias horas |
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
Cada división concreta del territorio. Se carga desde las listas oficiales de cada país: INEI en Perú, DANE en Colombia, INEGI en México. La de Perú es la edición 2022 del INEI más los distritos creados por ley después, revisada al 17/10/2026 (D-77). La migración V24 cargó la edición 2022 tal como la publica el INEI, con 25 departamentos, 196 provincias y 1 891 distritos. Cada distrito creado después entra con su propia migración en cuanto el INEI o SUNAT publican su código. El primero es Santa Rosa de Loreto (Ley 32403, 03/07/2025): la ley no le asigna código y ni el INEI ni SUNAT lo han publicado todavía, así que hasta entonces no se puede elegir. Las de Colombia y México se cargan antes de registrar la primera empresa de cada país (D-52).

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
- Un distrito que crea una ley entra activo, con su código oficial, y el distrito del que se separó sigue activo. Las leyes nuevas y las ediciones nuevas del INEI se revisan en el cierre de cada paso (D-77).

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

Así el contrato conserva el domicilio con el que se firmó sin necesidad de copiarlo dentro, y la dirección actual del cliente queda siempre disponible. Es la opción de menor costo: no agrega campos ni tablas, solo exige que la pantalla distinga una corrección de una mudanza. Vale para todo dueño con dirección propia: empresas, sedes y ubicaciones de terceros. Al cambiar la dirección de uno ya guardado, la pantalla pregunta cuál de las dos es (D-79).

**La división de una dirección es del mismo país y del escalón más bajo que ese país declaró.** Es lo que permite deducir la provincia y el departamento subiendo por la cadena (§4.3). Una dirección de Perú no puede apuntar a un municipio colombiano ni quedarse en la provincia cuando el país declara distritos. Para una dirección nueva, la división debe estar activa. Que la división sea del mismo país lo impide la base, porque la llave hacia la división incluye el país; que sea del escalón más bajo y que esté activa lo valida la aplicación.

**La división es obligatoria donde el país tiene sus divisiones cargadas (D-53).** Una dirección en Perú no se guarda sin su distrito: sin él, el sistema no puede deducir la provincia ni el departamento. Solo puede faltar en la dirección de un país cuyas divisiones no están cargadas, como la de un proveedor extranjero. Lo valida la aplicación, porque depende de qué países tienen su lista cargada. Otra cosa es si el dueño exige tener dirección. Una ubicación de terceros siempre la tiene, y una propia usa la de su sede. Una empresa o una sede puede registrarse sin dirección mientras se completa su registro, pero una vez que la tiene no la quita: solo la corrige o se muda.

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
| `length` | SMALLINT | Sí | Cuántos caracteres debe tener el número: de 1 a 20, lo que caben las columnas que guardan los números (D-78) | Permite avisar en el momento si alguien escribe un RUC de 10 dígitos en lugar de 11 |
| `has_check_digit` | BOOLEAN | No | Indica si el número trae un dígito de control que permite verificar que esté bien escrito. El cálculo depende de la sigla y lo hace `location` (D-75) | Un RUC mal escrito que llega a una factura se convierte en un problema tributario. Con el dígito de control se detecta al momento de registrarlo |
| `is_active` | BOOLEAN | No | Si el documento se puede elegir en registros nuevos | Permite retirar un documento sin tocar las empresas, proveedores y colaboradores que ya lo usan |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general, como el resto del bloque (D-42) |

**Relaciones.** Apunta a `countries`. Le apuntan `companies`, `suppliers`, `employees` y el emisor de `purchase_order_templates`, que se identifican con la pareja tipo de documento más número. Más adelante también los clientes, que usan DNI en vez de RUC.

**Ejemplo**

| País | Sigla | Nombre | Aplica a | Longitud | Dígito de control |
|---|---|---|---|---|---|
| PE | RUC | Registro Único de Contribuyentes | ambos | 11 | Sí |
| PE | DNI | Documento Nacional de Identidad | persona | 8 | No |
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
- **La longitud se mide cuando cambian el número o el tipo, como el dígito de control (D-91).** Al editar un registro sin cambiar su número ni su tipo, el número no se vuelve a medir, aunque la longitud del tipo se haya corregido después de guardarlo: sigue siendo válido, igual que con un tipo desactivado.
- **Cuando el tipo tiene dígito de control, el número lo cumple (D-75).** El cálculo lo hace `location` según la sigla del tipo dentro de su país. Hoy existe uno solo: el del RUC de Perú, con el módulo 11 de SUNAT, que pondera los diez primeros dígitos con 5, 4, 3, 2, 7, 6, 5, 4, 3 y 2. Un tipo con dígito de control cuyo cálculo todavía no existe —el NIT, el RFC y la CURP— acepta sus números sin verificar hasta que llegue. Se verifica al crear el registro y cada vez que cambian el número o el tipo; los números ya guardados siguen siendo válidos, igual que con un tipo desactivado.
- **El DNI no tiene dígito de control (D-76).** Sus 8 dígitos no traen uno dentro. El carácter de verificación que va impreso en el documento, después del número, es aparte y no se guarda.
- Las longitudes de Colombia y México del ejemplo están por confirmar: el RFC tiene 12 caracteres para empresas y 13 para personas, con un solo tipo que aplica a ambos; falta decidir si el dígito de control del NIT va dentro del número, y la CC no siempre tiene 10 dígitos. También falta el cálculo del dígito de control del NIT, el RFC y la CURP (D-75). Se resuelve antes de registrar la primera empresa de cada país (D-52).

**Nota técnica:** aquí se guardan la longitud y la existencia de dígito de control en vez de un patrón de validación, por el mismo motivo por el que se eliminó el del código postal. La diferencia es que estos dos son datos estables —la longitud del RUC no cambia por decreto— y el cálculo del dígito de control vive en el programa, no en la base. Lo hace `location` según la sigla y lo ofrece por su fachada. `organization` lo aplica en empresas y colaboradores, y `procurement` lo aplicará en proveedores y en el emisor de las plantillas (D-75).

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
- La tasa de cada día la carga un proceso automático desde la fuente oficial del país, a nombre de `SYSTEM`. El superadministrador la corrige a mano cuando hace falta, con `catalogos.corregir_tipo_cambio`, y la corrección queda en la bitácora: la tabla no tiene empresa, así que una tasa sirve a todo el sistema y no se puede acotar a la contabilidad de una empresa (D-103).
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
| 7 | Registra como colaboradores al administrador de cada empresa y al administrador de accesos, que son personas distintas, y les da acceso. Cuando cada uno cambió su contraseña provisional, le asigna su rol: `ADMIN_EMPRESA` sobre su empresa o `ADMIN_ACCESOS`, que alcanza todo el sistema | Usuario |
| 8 | El administrador de cada empresa registra a sus colaboradores, con su puesto y su sede | Usuario |
| 9 | El administrador de accesos da acceso a quienes usan el sistema y les asigna los roles que corresponden a su puesto y su sede | Usuario |

Los pasos 1 a 3 no los puede ejecutar nadie desde la aplicación —no hay con qué iniciar sesión todavía—. Esto no es un rodeo: es la única forma de resolver que el primer usuario no tiene creador.

**Los pasos 8 y 9 los hacen personas distintas, a propósito** (D-57). Quien registra a los colaboradores no les da acceso, y quien da acceso no registra colaboradores: así nadie puede inventar un colaborador y darle acceso. Por eso una misma cuenta nunca junta los permisos de las dos funciones, y en particular nunca tiene a la vez `ADMIN_EMPRESA` y `ADMIN_ACCESOS` (D-119, que reemplaza a D-71). El superadministrador puede hacer ambas cosas. Es el único que otorga los roles reservados —entre ellos `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`—, y el único que actúa sobre las cuentas que tienen alguno (§5.5, D-121).

Los catálogos, los roles y `SYSTEM` van en la migración. El primer superadministrador, en cambio, lo crea la aplicación al arrancar (decisión D-34), y solo mientras no exista ningún superadministrador activo. Toma el correo y la contraseña provisional de las variables de entorno `IAM_BOOTSTRAP_ADMIN_EMAIL` e `IAM_BOOTSTRAP_ADMIN_PASSWORD`, así esa contraseña no queda escrita en el repositorio. La cuenta figura creada por `SYSTEM` y debe cambiar la contraseña en su primer ingreso.

**Es la única persona que nace sin perfil de colaborador**, porque todavía no existe ninguna empresa donde registrarla. Mientras no se vincule a su perfil (paso 6) puede configurar el sistema —empresas, sedes y catálogos— y registrar y modificar colaboradores, porque tiene que crear el suyo, pero no puede dar acceso a nadie, otorgar roles ni registrar salidas o regresos (D-89). Así, desde el primer acceso que se otorga, cada acción queda atribuida a una persona con nombre, documento y puesto. Vincularse a sí mismo es solo de ese paso: lo hace el superadministrador mientras no tiene un colaborador activo. Después ninguna cuenta se vincula a sí misma, tampoco a un colaborador de otra empresa, porque sería darse acceso a sí misma (D-70).

**Casos del arranque.** Si faltan las variables, la aplicación arranca igual, sin superadministrador, y lo advierte en su registro de eventos. Si el correo indicado ya pertenece a otra cuenta, la aplicación no arranca hasta que se corrija. El rol de ese primer superadministrador figura otorgado por `SYSTEM`.

**Registro público, apagado por defecto** (decisión D-32). Además del acceso que da un administrador, una persona puede registrarse desde la pantalla de inicio de sesión. Se enciende o se apaga con la variable de entorno `IAM_PUBLIC_SIGN_UP_ENABLED`, y viene apagado. Hoy no está construido como se describe aquí (§5.13): queda fuera del paso 3, sigue apagado y se construye antes de encenderlo en un entorno (D-83). Como toda cuenta pertenece a un colaborador (D-38), el registro no crea solo la cuenta:

- **Registra el perfil y la cuenta en la misma operación.** La persona elige su empresa —y su sede, si corresponde— y completa los mismos datos obligatorios que exige el registro de un colaborador (§5.3), además de su correo y su contraseña. Si algo falla, no queda ni el colaborador ni la cuenta.
- **Si su documento ya está registrado en esa empresa, el registro se rechaza.** Esa persona ya tiene perfil, y el acceso se lo da el administrador de accesos desde ahí. Si el registro pudiera vincularse a un perfil existente, cualquiera que conozca un número de documento podría apropiarse del perfil de otra persona.
- **Si el correo ya tiene cuenta, el registro se rechaza.** Esa persona ya tiene acceso por otra empresa del grupo: el administrador de esta empresa la registra como colaboradora y el administrador de accesos vincula el perfil a su cuenta (§5.3).
- **Nace sin roles.** Registrarse no da acceso a nada: los roles se los asigna después el administrador de accesos o un superadministrador, con las reglas de §5.5 y §5.7.
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
| `photo_path` | TEXT | Sí | Foto del colaborador, opcional. Su ubicación tiene a lo sumo 500 caracteres, y la base rechaza una más larga | El archivo se guarda fuera de la base, igual que los documentos del vehículo; aquí queda su ubicación. Sirve para mostrarla junto a sus registros |
| `user_id` | BIGINT FK → `users` | Sí | La cuenta con que esta persona entra al sistema | Vacío para quien no usa el sistema, como un mecánico. **La referencia va en este sentido a propósito**: quien trabaja para dos empresas del grupo tiene dos filas de colaborador y una sola cuenta, y ambas pueden apuntar a ella |
| `hire_date` | DATE | No | Fecha de ingreso. No puede ser posterior a hoy (D-88) | — |
| `termination_date` | DATE | Sí | Fecha de salida. Vacío = sigue trabajando | Un colaborador que se va no se borra: sus ventas y sus registros siguen siendo suyos |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | — |
| `created_by` | BIGINT FK → `users` | No | Quién registró al colaborador | `SYSTEM` cuando llegó por el registro público (§5.2); si no, el administrador que lo registró |

**Relaciones.** Apunta a `companies`, `branches`, `tax_id_types` y `users` (dos veces: su cuenta y quién lo registró). Le apuntan las asignaciones de asesora, las instalaciones de equipamiento y los vínculos que decide el superadministrador (`employee_account_link_events`, §5.4).

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
- **Toda cuenta pertenece a un colaborador** (decisión D-38). Las cuentas no se crean solas. Hay dos caminos, y en ambos la cuenta queda vinculada a su colaborador en la misma operación: quien tiene `seguridad.dar_acceso` con alcance sistema —el administrador de accesos o un superadministrador (D-122)— **da acceso** a un colaborador ya registrado, o la persona usa el **registro público**, si está encendido, que crea el colaborador y la cuenta juntos (§5.2). Si la persona ya tiene cuenta porque trabaja para otra empresa del grupo, dar acceso vincula el nuevo colaborador a esa misma cuenta en vez de crear otra. Un colaborador puede existir sin cuenta y recibir acceso cuando lo necesite. Las únicas cuentas sin colaborador son `SYSTEM`, que no es una persona, el primer superadministrador hasta que se vincula (§5.2) y la que quedó sin ninguno al desvincularla, hasta que un superadministrador la vincula de nuevo (más abajo).
- **Dar acceso crea la cuenta o la vincula, y nada más** (D-58). Una cuenta nueva recibe una contraseña provisional que genera el sistema, se muestra una sola vez y debe cambiarse en el primer ingreso (§5.4). Vincular una cuenta que ya existía nunca cambia sus roles ni su contraseña: la persona sigue entrando con la suya y conserva los roles que tiene en otras empresas; los de esta empresa se asignan después (§5.7). El administrador de accesos vincula cuentas de cualquier empresa del grupo, salvo una protegida, la que tiene un rol reservado vigente: esa solo la vincula un superadministrador (§5.5, D-121). `SYSTEM` no se vincula a nadie. La única cuenta que se vincula a sí misma es la del primer superadministrador, y solo mientras no tiene un colaborador activo (§5.2, paso 6; D-70). Si la cuenta estaba cerrada por una salida o por una desvinculación, vincularla cierra el bloqueo `baja`, porque vuelve a tener un colaborador activo (§5.4).
- **Vincular una cuenta que ya existe exige que sea la misma persona** (D-82). Dar acceso vincula el colaborador a una cuenta existente cuando esa cuenta ya pertenece a un colaborador de otra empresa del grupo con la misma sigla de documento y el mismo número. Se compara la sigla del tipo, no el tipo: los tipos de documento son de cada país y cada empresa es de un país (D-5), así que el colaborador de la otra empresa tiene siempre un tipo de otro país. Un pasaporte con la sigla `PAS` en las dos empresas y el mismo número es el mismo documento, aunque cada país tenga su propio tipo. Si no coinciden, solo un superadministrador vincula la cuenta, con `seguridad.confirmar_vinculo`, que tiene solo él (D-117): antes ve el nombre, el documento y la empresa del colaborador que ya la tiene, y confirma que es la misma persona. Lo mismo vale para una cuenta que existe y no pertenece a ningún colaborador, como la que dejó un vínculo equivocado al desvincularse: la vincula solo un superadministrador, que ve que no tiene titulares y confirma. La confirmación vale para los titulares que vio: si cambiaron antes de confirmar —alguno cambió sus datos, apareció o se fue—, la cuenta no se vincula, y la pantalla muestra los de ahora y pide confirmar de nuevo. La confirmación queda registrada en `employee_account_link_events` (§5.4): quién la dio, cuándo y los titulares que confirmó, ninguno si la cuenta no tenía. A cualquier otro se le rechaza. Así un correo mal escrito que resulta ser el de otra persona no le da a esa persona acceso en esta empresa, ni los roles que se otorguen al colaborador, ni le reabre una cuenta cerrada por su salida. La única excepción es la del primer superadministrador, que se vincula a sí mismo: su cuenta no tiene todavía un colaborador con el cual comparar, así que no hay nada que confirmar (§5.2, paso 6). Un vínculo equivocado lo corrige solo un superadministrador, desvinculando la cuenta: la desvinculación funciona como una salida (más abajo).
- **Para tener un rol en una empresa o una sede**, la cuenta debe estar vinculada a un colaborador activo —sin fecha de salida— de esa empresa. Un rol de alcance sistema exige un colaborador activo en alguna empresa.
- **Cuando un colaborador sale**, se registra su fecha de salida, que no puede ser posterior a hoy (D-60). La salida se registra cuando ocurre, no por adelantado, porque retirar los roles antes de tiempo dejaría sin acceso a alguien que todavía trabaja. Hoy se calcula como fija §2.6: en la zona de la sede del colaborador; sin sede, en la del país de su empresa. En la misma operación se retiran los roles de su cuenta en esa empresa, con motivo `baja_colaborador` (§5.7). Quien trabaja para dos empresas y deja una conserva su cuenta y sus roles en la otra.
- **Si a la cuenta no le queda ningún colaborador activo, la salida la cierra** desde el momento en que se registra, aunque la fecha de salida sea anterior, porque un bloqueo no se registra en el pasado (§5.4). La cuenta se bloquea con motivo `baja`, salvo que ya tenga una `baja` en curso. Se retiran sus demás roles con motivo `baja_cuenta` (§5.9) y se cierran sus sesiones. Si la persona estaba de vacaciones o de licencia, ese bloqueo termina en ese momento, y los bloqueos programados de la línea laboral se cancelan. Un bloqueo por `seguridad`, en curso o programado, no se toca: corre en su propia línea de tiempo (§5.4).
- **La salida siempre quita el acceso** (D-60), también el de una cuenta protegida (§5.5): quien se fue no conserva el acceso mientras alguien con más permisos se entera. Una salida equivocada se corrige con un regreso. El único freno es la protección del último superadministrador (§5.9).
- **Desvincular una cuenta funciona como una salida** (D-82). Lo hace solo un superadministrador, con `seguridad.desvincular_cuentas` (D-117, D-122), para corregir un vínculo equivocado, y la operación queda registrada en `employee_account_link_events` (§5.4). El colaborador sigue registrado, ahora sin cuenta, y la cuenta pierde sus roles en esa empresa, retirados con motivo `retiro`: el colaborador no salió, alguien corrigió un error. Si a la cuenta no le queda ningún colaborador activo, se cierra como en una salida (reglas anteriores): `baja`, `baja_cuenta` para sus demás roles y sesiones cerradas, con el mismo freno de la protección del último superadministrador.
- **Registran y modifican colaboradores** quienes tienen `seguridad.gestionar_colaboradores`, y registran sus salidas y regresos quienes tienen `seguridad.registrar_salidas_y_regresos`, cada uno donde su asignación actúa (D-117, D-122): un rol de empresa, en toda la empresa; uno de sede, en su sede. Un colaborador sin sede exige actuar en toda la empresa, y cambiarlo de sede, en la de origen y en la de destino. La semilla da los dos permisos al superadministrador y al administrador de la empresa (D-57). El administrador de accesos no los tiene: quien da acceso no puede también crear al colaborador que lo recibe, y los pares de D-119 impiden que una cuenta junte las dos funciones. Nadie registra su propia salida ni su propio regreso (D-70). Registrar y modificar colaboradores no exige que quien lo hace esté vinculado a uno, así el superadministrador sin vincular crea el suyo; las salidas y los regresos sí lo exigen (D-89, D-122).
- **Leen los colaboradores de una empresa** quienes tienen el permiso (D-93, D-105): la fila sin sus datos personales —nombre, puesto, sede, fechas de ingreso y salida, si está activo y si tiene cuenta— cualquier rol con `configuracion.ver_colaboradores`, para elegir quién instaló un equipo o nombrar a quien registró algo; el documento, el teléfono, la foto y la exportación, solo con `configuracion.ver_datos_personales_colaboradores`, que la matriz da al superadministrador, al administrador de accesos y al administrador de la empresa. Un rol de empresa o de sede que no la administra no ve esos datos personales: quien atiende un local no necesita la planilla de toda la empresa. El campo que no se puede ver no viaja (§2.5).
- **La fecha de ingreso no puede ser posterior a hoy** (D-88), al registrar al colaborador y al modificarlo, como la salida y el regreso. Hoy se calcula como fija §2.6: en la zona de la sede del colaborador; sin sede, en la del país de su empresa. Un colaborador con ingreso futuro podría recibir acceso y roles antes de empezar a trabajar, y no se le podría registrar la salida hasta corregir su ingreso.
- **Un colaborador se registra en una empresa activa y, si tiene sede, en una sede activa** (D-84). El que ya está registrado sigue siendo válido y se sigue modificando aunque su empresa o su sede se hayan desactivado, mientras no cambie de sede; si cambia, la nueva tiene que estar activa.
- **Cuando un colaborador regresa**, se registra su nueva fecha de ingreso, que no puede ser anterior a la de salida ni posterior a hoy, calculado igual que para la salida. Si su cuenta estaba cerrada por la salida, en la misma operación se cierra el bloqueo `baja`, y la persona vuelve a entrar con la misma cuenta y su historial completo. Un bloqueo por `seguridad` en curso sigue hasta que alguien lo levante. Los roles no vuelven solos: se asignan de nuevo (§5.9). Regresar es volver a colgarse de la empresa (D-94): un colaborador regresa solo a una empresa activa y, si tiene sede, a una sede activa, porque el regreso le devuelve la cuenta a la persona.

### 5.4 `users` — Cuentas de acceso

Solo lo necesario para entrar. Quién es la persona vive en `employees`: toda cuenta pertenece a un colaborador, salvo `SYSTEM`, el primer superadministrador hasta que se vincula y la que quedó sin ninguno al desvincularla (§5.3).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `email` | VARCHAR(255) UNIQUE | No | El correo con que entra. No se repite. Se guarda sin espacios y en minúsculas | Una cuenta por persona, no por empresa: quien trabaja para dos empresas del grupo entra una sola vez y su historial queda completo en un solo lugar |
| `password_hash` | VARCHAR(255) | No | La contraseña guardada de forma cifrada e ilegible | Nunca se guarda tal como se escribió. Ni siquiera quien administra el sistema puede verla |
| `must_change_password` | BOOLEAN | No | Obliga a cambiar la contraseña la próxima vez que entra | Cuando se crea una cuenta, o un administrador restablece la contraseña, se entrega una clave provisional. Sin esta marca, esa clave se queda para siempre |
| `password_updated_at` | TIMESTAMPTZ | Sí | Cuándo cambió la contraseña por última vez | Permite exigir el cambio cada cierto tiempo y detectar claves antiguas |
| `mfa_secret` | VARCHAR(255) | Sí | Clave del segundo factor, si lo activó | Un sistema con datos financieros debería poder exigir un segundo paso al entrar |
| `last_login_at` | TIMESTAMPTZ | Sí | Última vez que entró | Con esto se detectan cuentas abandonadas, que son la puerta de entrada más común a un sistema |
| `created_at` | TIMESTAMPTZ | No | Cuándo se creó la cuenta | — |
| `created_by` | BIGINT FK → `users` | Sí | Quién la creó | Primera pregunta de toda revisión de seguridad: quién le dio acceso a quién |
| `updated_at` | TIMESTAMPTZ | No | Última modificación | — |

**Relaciones.** Apunta a sí misma (`created_by`). Le apuntan `employees`, `user_roles`, `user_access_suspensions`, `employee_account_link_events` y prácticamente todas las tablas a través de `created_by`, `updated_by` y campos equivalentes.

**Ejemplo**

| id | email | must_change_password |
|---|---|---|
| 1 | system@millennium.ky | No |
| 2 | tere@millennium.ky | No |
| 3 | alejandro@millennium.ky | Sí |

La primera no corresponde a ninguna persona: existe para figurar como responsable de las cargas masivas y los procesos automáticos. Luis, el mecánico, no aparece aquí porque no usa el sistema.

**Quién es la persona detrás de una cuenta** se responde desde `employees`, que es quien guarda la referencia. Así alguien empleado por dos empresas del grupo tiene dos filas de colaborador —una por empresa, como exige la regla— y ambas apuntan a la misma cuenta. En las cuentas sin colaborador, la pantalla muestra el correo.

**Reglas de negocio**

El usuario **nunca se borra**: su nombre está en las facturas, habilitaciones y cambios de estado que registró. Borrarlo dejaría todo ese historial sin autor.

**Contraseñas.** Tienen entre 8 y 72 caracteres, sin reglas de composición; 72 es el máximo que procesa el cifrado que se usa (BCrypt). La nueva tiene que ser distinta de la actual. Al iniciar sesión, un correo que no existe y una contraseña equivocada reciben la misma respuesta, para no revelar qué cuentas existen. Una cuenta bloqueada se entera de que lo está solo después de que su contraseña coincidió, así la respuesta no le revela nada a quien no la conoce.

**Mientras la contraseña es provisional**, la cuenta solo puede consultar quién es y cambiar su contraseña. Sus roles no valen hasta que la cambie. Tampoco recibe un rol reservado —uno que incluye un permiso privilegiado, como `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA`— hasta que la cambie (§5.7, D-72, D-121): quien le entregó la clave provisional la conoce, y no debe poder terminar con una cuenta privilegiada en sus manos.

**Restablecer la contraseña de otra persona** (D-63). Cuando alguien olvida su contraseña, quien tiene `seguridad.restablecer_contrasenas` con alcance sistema —un superadministrador o el administrador de accesos (D-122)— le restablece una provisional: la genera el sistema, se muestra una sola vez a quien la pidió y la persona debe cambiarla en su siguiente ingreso. En la misma operación se cierran todas las sesiones de esa cuenta, para que nadie siga dentro con la contraseña anterior (§5.12). Nadie restablece así la suya: para eso está el cambio de contraseña. La de `SYSTEM` no se restablece, porque esa cuenta no inicia sesión. Quien no es superadministrador no la restablece a una cuenta protegida (§5.5, D-121), y ningún rol de una empresa restablece contraseñas: el permiso solo cuenta con alcance sistema (D-57, D-122).

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
| `reason` | VARCHAR(20) | No | `vacaciones`, `licencia`, `baja`, `seguridad`. Los tres primeros forman la línea laboral; `seguridad` tiene una línea propia | Distingue una ausencia temporal de una salida de la empresa y de un incidente. Sin esto, en un reporte de personal las vacaciones se cuentan como bajas |
| `notes` | TEXT | Sí | Detalle cuando haga falta | — |
| `created_by` | BIGINT FK → `users` | No | Quién ordenó el bloqueo | Misma exigencia que en los cambios de estado: fecha, usuario y motivo |
| `created_at` | TIMESTAMPTZ | No | Cuándo se registró el bloqueo | — |
| `closed_by` | BIGINT FK → `users` | Sí | Quién levantó el bloqueo, o quién lo canceló antes de que empezara. Vacío mientras nadie lo cierre | Sin este dato se sabría quién quitó el acceso pero no quién lo devolvió, que es la mitad de la pregunta en una revisión de seguridad |
| `closed_at` | TIMESTAMPTZ | Sí | Cuándo se levantó o se canceló. Es siempre el momento en que alguien lo cerró, nunca una fecha elegida | Un bloqueo levantado termina en ese momento. Uno cancelado antes de empezar conserva el período que tenía programado, para que se sepa qué se había planeado, y deja de bloquear |

Cómo queda cada caso:

| Situación | Qué se hace | Al volver |
|---|---|---|
| Vacaciones o licencia | Se crea un período con motivo `vacaciones` o `licencia`, con su fin si ya se conoce | Termina solo al llegar su fin. Si la persona vuelve antes, o el período quedó abierto, se levanta en el momento en que regresa |
| Salida de la empresa | Se crea un período abierto con motivo `baja`. Lo crea solo el sistema, al registrar la salida del último colaborador activo de la cuenta o al desvincular la cuenta de él (§5.3); nadie lo abre a mano | Si la persona regresa, o se le da acceso desde otra empresa del grupo, se cierra ese período y vuelve a entrar con la misma cuenta y su historial completo. Nadie lo cierra a mano |
| Bloqueo por seguridad | Período abierto con motivo `seguridad`, en su propia línea de tiempo | Se cierra cuando se aclara. La salida y el regreso no lo tocan |
| Un bloqueo programado que ya no hace falta | Se cancela antes de que empiece | — |

Un usuario está activo si **no tiene ningún bloqueo en curso**: ninguno que ya haya empezado, no haya terminado y no se haya cancelado. Basta uno, de cualquier línea, para que no pueda entrar. No hay un campo que lo diga, para que no pueda contradecir a la tabla.

**Dos líneas de tiempo** (D-62). Los bloqueos de una persona corren en dos líneas: la **laboral**, con `vacaciones`, `licencia` y `baja`, y la de **seguridad**. Dentro de una misma línea, el sistema impide que dos bloqueos se crucen en el tiempo, incluso en el pasado; los cancelados no cuentan. Entre líneas sí pueden cruzarse: un bloqueo por seguridad puede caer sobre unas vacaciones, una licencia o una baja. Tienen que ser independientes porque responden a hechos distintos. Si alguien investigado por un incidente sale de la empresa, la baja no puede borrar el bloqueo de seguridad. Y si después regresa, cerrar la baja no puede devolverle el acceso mientras el incidente no se aclare. La línea se deduce del motivo; no se guarda.

**Quién bloquea y levanta** (D-57, D-122). Quien tiene `seguridad.bloquear_cuentas` con alcance sistema —el superadministrador y el administrador de accesos—, por vacaciones, licencia o seguridad. Nadie bloquea su propia cuenta ni levanta sus propios bloqueos (D-70), `SYSTEM` no se bloquea, y quien no es superadministrador no bloquea ni levanta los bloqueos de una cuenta protegida (§5.5, D-121). Un bloqueo programado sobre una cuenta que después pasó a ser protegida se conserva: se ve en su historia y lo cancela un superadministrador. La `baja` no la abre ni la cierra nadie a mano (D-69): la abre el sistema con la salida del último colaborador activo de la cuenta, o al desvincular la cuenta de él (§5.3), y la cierran el regreso o un nuevo acceso. El administrador de una empresa no bloquea cuentas: registra la salida, y la `baja` es su consecuencia.

#### Qué queda registrado, exactamente

Cada bloqueo deja una fila que no se borra ni se reutiliza; levantarlo o cancelarlo la completa con quién y cuándo, sin borrar nada. Un ejemplo de la historia de una persona:

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

Con dos líneas, un bloqueo por seguridad puede quedar debajo de uno de la línea laboral. Otra persona, investigada por un incidente, sale de la empresa mientras tanto:

| # | Período | Motivo | Línea | Ordenado por | Reactivado por |
|---|---|---|---|---|---|
| 1 | 03/08/2026 10:00 → *(abierto)* | seguridad | seguridad | Janover | — |
| 2 | 14/08/2026 18:20 → 01/09/2026 09:00 | baja | laboral | Tere | Tere |

La salida del 14/08/2026 abrió la fila 2 y no tocó la 1. El regreso del 01/09/2026 cerró la 2, y la persona sigue sin entrar: la fila 1 sigue abierta hasta que se aclare el incidente.

Lo que **no** cubre esta tabla es la corrección posterior de una de estas filas —si alguien edita el motivo o mueve una fecha—. Eso lo registra la bitácora general de cambios del sistema, que documenta qué campo se modificó, su valor anterior y su valor nuevo (bloque 11). Son dos cosas distintas: esta tabla guarda lo que pasó con el acceso; la bitácora guarda lo que alguien hizo con el registro.

**Relaciones.** Apunta a `users` tres veces: la persona bloqueada, quien ordenó el bloqueo y quien reactivó. No le apunta nadie.


**Se pueden programar por adelantado, nunca hacia atrás** (D-61). Nada impide crear un bloqueo con fecha de inicio futura para unas vacaciones ya aprobadas: la persona conserva el acceso hasta que llegue esa fecha y lo pierde sola. No hace falta que nadie se acuerde el día que empieza. Lo que no se puede es crearlo con inicio en el pasado: esta tabla dice quién podía entrar en cada momento, y un bloqueo que no ocurrió no se registra. Sin fecha de inicio, empieza en el momento en que se registra.

**Levantar y cancelar ocurren siempre ahora.** Cerrar un bloqueo en curso lo termina en ese momento. Cerrar uno programado, antes de que empiece, lo cancela: conserva el período que tenía programado, registra quién lo canceló y cuándo, y deja de contar. Nadie elige la fecha de cierre, porque una fecha elegida podría falsear quién tenía acceso. Un bloqueo que ya terminó, o que ya se levantó o se canceló, no se vuelve a cerrar.

**Una ventaja concreta de bloquear la cuenta en vez de retirarle los roles.** Cuando alguien sale de vacaciones se podrían quitar sus roles y devolvérselos al volver. En la práctica alguien olvida devolver alguno, o devuelve uno de más. Bloqueando la cuenta, los roles quedan intactos: la persona vuelve y encuentra exactamente los mismos accesos que tenía.

**Cuando empieza un bloqueo se cierran también las sesiones abiertas de la cuenta**, o la persona seguiría dentro hasta que vencieran. Como las sesiones viven en el servidor (§5.12), cerrarlas es borrarlas y el efecto es inmediato. Un bloqueo programado surte efecto al llegar su inicio: desde ese momento la sesión deja de servir, sin que nadie tenga que hacer nada.

#### `employee_account_link_events` — Vínculos que decide el superadministrador

Las dos decisiones sobre el vínculo de un colaborador con una cuenta que solo toma un superadministrador —con `seguridad.confirmar_vinculo` y `seguridad.desvincular_cuentas`, que tiene solo él (D-117)— dejan una fila aquí (D-82): vincular la cuenta cuyos titulares tienen otro documento, o que no tiene titulares, después de verlo y confirmar, y desvincular una cuenta para corregir un vínculo equivocado. Es de `iam`, como los bloqueos, y se crea con el bloque 3b porque apunta a `employees` y a `companies`.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `kind` | VARCHAR(20) | No | Qué se decidió: `vinculo_confirmado` (se vinculó una cuenta cuyos titulares tienen otro documento, o que no tiene titulares, confirmándolo tras verlo) o `desvinculo` (se deshizo un vínculo) | Las dos decisiones juntan o separan a una persona de una cuenta, y responden a la misma pregunta de una auditoría |
| `company_id` | BIGINT FK → `companies` | No | La empresa del colaborador | Solo se inserta una fila de las empresas que alcanza quien opera, como en toda tabla con empresa (D-66). Junto con `employee_id` forma la llave hacia el colaborador, así que la fila no puede nombrar a un colaborador de otra empresa |
| `employee_id` | BIGINT FK → `employees` | No | El colaborador vinculado o desvinculado | — |
| `user_id` | BIGINT FK → `users` | No | La cuenta vinculada, o la que tenía el colaborador al desvincularlo | Después de desvincular, el colaborador ya no apunta a la cuenta: solo aquí queda cuál era |
| `holders` | JSONB | Sí | Los titulares que confirmó el superadministrador: la lista de los colaboradores que tenían la cuenta cuando confirmó, cada uno con su identificador público, su nombre, la sigla y el número de su documento, su empresa con su nombre y si estaba activo. Es una lista vacía si la cuenta no tenía ninguno. Solo la tiene un vínculo confirmado; un desvínculo, nunca | Es una foto de ese momento: guarda sobre qué se decidió aunque esos colaboradores cambien después. Son los mismos que vio, porque la confirmación no vale si cambiaron antes de confirmar (§5.3) |
| `created_by` | BIGINT FK → `users` | No | Quién decidió: el superadministrador que confirmó o desvinculó. Lo pone el sistema con la cuenta de la sesión (D-42) | Es la columna de registro de una fila que se escribe una vez, igual que quién ordenó un bloqueo |
| `created_at` | TIMESTAMPTZ | No | Cuándo se decidió: el momento de la operación que hizo el cambio | — |

**Relaciones.** Apunta a `companies`, a `employees` y a `users` (dos veces: la cuenta vinculada o desvinculada y quién decidió). No le apunta nadie.

**Reglas de negocio**

- **Se escribe una vez y nunca se edita ni se borra.** La cuenta de base de datos de la aplicación solo lee e inserta en esta tabla, y la base no tiene regla que permita modificar ni borrar una fila. Una migración que vuelva a otorgar permisos sobre todas las tablas tiene que quitar otra vez el de modificar esta.
- **La fila se escribe en la misma operación que el vínculo o el desvínculo.** Si el cambio no llega a guardarse, la fila tampoco: existe exactamente cuando el cambio ocurrió.
- **Un vínculo confirmado lleva sus titulares, y un desvínculo no los lleva.** Lo impide la base: cuando `kind` es `vinculo_confirmado`, `holders` tiene valor y es una lista, vacía si la cuenta no tenía titulares; cuando es `desvinculo`, `holders` no tiene valor.
- **Los titulares guardados son los que se confirmaron.** Si cambiaron entre que el superadministrador los vio y confirmó, no se vincula ni se escribe la fila: la pantalla muestra los de ahora y pide confirmar de nuevo (§5.3).
- Un vínculo que no necesitó confirmación, porque la sigla y el número del documento coinciden, no deja fila aquí: no hubo una decisión que registrar.
- **Se inserta acotada por empresa** como toda tabla con `company_id` (D-66): solo en las empresas que alcanza quien opera.
- **La leen solo el superadministrador y los procesos automáticos.** La base muestra las filas a una cuenta con `SUPERADMIN` vigente y a un proceso con alcance `SYSTEM` sin cuenta. No las ve el administrador de accesos, aunque también tiene alcance sistema, ni ningún rol de una empresa: `holders` guarda nombres y documentos de personas de otras empresas, y estos vínculos los decide solo el superadministrador.
- **Todavía no hay pantalla que la lea.** Las filas se escriben y se conservan; una lectura, solo para el superadministrador, puede llegar después con su propia operación.

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
| `is_active` | BOOLEAN | No | Si el rol se sigue ofreciendo al otorgar accesos. Un rol propio que ya no se usa se desactiva; los del sistema siempre están activos | Un rol que ya no corresponde no debe seguir ofreciéndose en la pantalla de accesos, y como nada se borra, se desactiva: lo que ya tenía asignado sigue vigente (D-97) |

**Relaciones.** Apunta a `companies` cuando el rol pertenece a una empresa concreta. Le apuntan `role_permissions` (qué permisos incluye) y `user_roles` (a quién se le asignó).

**Ejemplo**

| code | name | empresa | alcance | del sistema |
|---|---|---|---|---|
| `SUPERADMIN` | Superadministrador | *(todas)* | Todo el sistema | Sí |
| `ADMIN_ACCESOS` | Administrador de accesos | *(todas)* | Todo el sistema | Sí |
| `ADMIN_EMPRESA` | Administrador de empresa | *(todas)* | Una empresa | Sí |
| `ASESOR_VENTAS` | Asesor de ventas | *(todas)* | Una sede | Sí |
| `JEFE_PATIO` | Jefe de patio | Millennium Perú | Una sede | No |

El último lo creó el superadministrador para Millennium Perú (D-98): no le aparece a Colombia.

#### Crear roles nuevos

Los siete roles que trae el sistema (cuadro al final de esta sección) son el punto de partida, no la lista definitiva: los puestos cambian y hay que poder agregar roles sin tocar el programa. Crear uno es insertar una fila en `roles` y marcar sus permisos, todo desde la pantalla de administración.

| Quién lo crea | Dónde queda disponible |
|---|---|
| Quien tiene `seguridad.gestionar_roles` con alcance sistema; hoy, solo el superadministrador (D-122, D-123) | En todas las empresas (`company_id` vacío), o solo en una empresa concreta (`company_id` lleno), eligiéndola al crearlo (D-98) |

Un rol propio nace vacío o como copia exacta de otro (D-124, más abajo). Leen los roles quienes tienen `seguridad.ver_roles` (D-122): con alcance sistema, todos; con una empresa, los del sistema, los generales y los de esa empresa. Hoy lo tienen los tres administradores: el administrador de accesos los lee todos, porque es quien los otorga y necesita saber qué incluye cada uno, y el de una empresa, los de la suya (D-98). Ninguno de los dos crea roles: `seguridad.gestionar_roles` solo cuenta con alcance sistema, y la semilla se lo da solo al superadministrador.

**Los roles propios de una empresa existen desde la matriz de permisos** (D-81, D-98): `roles.company_id` apunta a la empresa, o queda vacío en los roles generales y en los del sistema; el código es único dentro de cada empresa y entre los generales, y un rol propio nunca usa el código de un rol del sistema. Un rol de una empresa solo se asigna dentro de esa empresa, y no puede ser de alcance sistema.

Cinco reglas que gobiernan esto:

**1. No se puede dar lo que no se tiene.** Al crear o editar un rol, solo se pueden marcar permisos que quien lo está haciendo ya posee. Sin esta regla, el administrador de una empresa podría crear un rol con acceso total y asignárselo: quedaría convertido en superadministrador sin que nadie lo autorizara. Lo que posee quien edita se cuenta según el rol (D-122): para un rol general, solo lo que tiene por asignaciones de alcance sistema; para un rol de la empresa C, lo que actúa en toda C. Y nadie cambia los permisos de un rol que tiene vigente (D-70, D-122): no se agranda a sí mismo por la puerta del rol.

**2. La misma regla vale al asignar.** Restringir solo la creación no sirve de nada: si el administrador no puede crear un rol con acceso total pero sí puede asignarse uno existente que ya lo tiene, el resultado es idéntico. Nadie puede otorgar un rol cuyos permisos excedan los suyos, contados como en la regla 1. La regla se aplica a los roles no reservados: los reservados los otorga solo el superadministrador, que otorga cualquier rol (D-121). La única excepción es el administrador de accesos, con su permiso propio y los límites que se explican más abajo (D-56, D-120).

**3. Los permisos no tienen niveles, son conjuntos.** No existe "un permiso por encima de otro": la comparación es de contenido. El rol nuevo es válido si todos sus permisos están dentro de los de quien lo crea.

**4. El alcance se fija al crear y no cambia.** Si un rol nace para asignarse a nivel de sede, no puede convertirse después en rol de empresa: las asignaciones que ya existen quedarían con datos incoherentes. Para cambiarlo se crea un rol nuevo.

**5. Lo que alguien autorizó sobrevive a su salida.** Si a un administrador se le retiran permisos o deja la empresa, los roles que creó y los accesos que otorgó **siguen siendo válidos**. La autorización fue legítima cuando se dio, igual que un contrato firmado por un gerente que después renunció. Lo que sí deja de poder hacer es seguir editándolos: de ahí en adelante se le aplica la regla 1 con los permisos que tenga en ese momento, o ninguno si ya salió.

Esto es también la razón de fondo por la que un usuario nunca se borra: si desapareciera, los roles y accesos que autorizó quedarían sin responsable y no habría forma de explicar quién los aprobó.

Como los accesos otorgados no caducan solos —salvo los que se otorgan con caducidad (§5.7, §5.8)—, el control que los mantiene sanos es revisarlos cada cierto tiempo. El modelo lo permite sin campos adicionales: cada asignación guarda quién la otorgó y cuándo, así que se puede listar todo lo vigente y depurar lo que ya no corresponde.

Los roles marcados como propios del sistema no se pueden editar ni borrar, aunque quien lo intente tenga todos los permisos. Tampoco se les cambian los permisos ni se desactivan: la matriz de los siete roles del sistema la fijan las migraciones, y para variarla se crea un rol propio (D-97), también a partir de uno de ellos (D-124).

**Roles reservados por lo que incluyen** (D-121). Un rol es reservado si incluye un permiso privilegiado: un `seguridad.*` de una categoría que no es `ver` (§5.6, D-118). Los del sistema que lo son —`SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`— lo eran antes por su código; desde D-121 lo son por su matriz, y un rol propio que incluya un permiso privilegiado también lo es. Un rol reservado lo otorga y lo retira solo un superadministrador, no se limita a una marca (D-59) y va solo a una cuenta que ya cambió su contraseña provisional (D-72); una cuenta que tiene uno vigente es **protegida**, y solo la maneja un superadministrador. Por eso:

- Agregar o quitar un permiso privilegiado en un rol que tiene titulares vigentes es solo del superadministrador.
- Volver reservado un rol se rechaza (`ROLE_RESERVED_HOLDERS_INVALID`) mientras alguno de sus titulares tenga una asignación acotada a una marca, una contraseña provisional o una asignación que no otorgó un superadministrador: esos titulares no cumplirían lo que un rol reservado exige.

**Dónde se marcan los permisos de accesos y roles** (D-122). Los ocho que solo cuentan con alcance sistema —`dar_acceso`, `asignar_roles`, `asignar_roles_sin_tenerlos`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `gestionar_roles`— solo se marcan en roles de alcance `SYSTEM`, que por eso son generales: marcarlos o copiarlos en otro rol se rechaza (`ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`). `confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos` no se marcan en ningún rol propio (D-117).

**Copiar un rol** (D-124). Un rol propio se crea también como copia exacta de los permisos de otro, del sistema o propio: "Crear rol propio a partir de este". La copia no se recorta: si quien la hace no posee alguno de los permisos, se rechaza nombrando los que faltan (regla 1); si junta un par, se rechaza (D-119); y si el rol nuevo no es de alcance sistema y la copia trae alguno de los ocho de arriba, también. Copiar `SUPERADMIN` o `ADMIN_ACCESOS` se rechaza siempre: el primero junta pares, y el segundo trae `asignar_roles_sin_tenerlos`, que nadie más posee. La pantalla lo dice antes de intentarlo.

**Una excepción a la regla 2, con nombre propio** (D-56, D-120). El administrador de accesos asigna roles cuyos permisos no tiene: es el permiso `seguridad.asignar_roles_sin_tenerlos`, que tiene solo él y no se marca en un rol propio. Su trabajo es dar acceso, no registrar facturas ni dar de alta unidades, y no tendría sentido darle todos los permisos del sistema solo para que pueda asignarlos. La excepción tiene tres límites, que son los que la hacen segura (D-55):

- **Nunca actúa sobre sí mismo.** No se asigna roles ni actúa sobre su propia cuenta, como nadie (D-70).
- **Nunca otorga ni retira un rol reservado** (D-121), entre ellos `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`. Los reservados los otorga y los retira solo un superadministrador.
- **Nunca actúa sobre una cuenta protegida**, la que tiene un rol reservado vigente: no la vincula a un colaborador, no le asigna ni le retira roles, no la bloquea ni le levanta bloqueos y no le restablece la contraseña. Esas cuentas las administra solo un superadministrador.

Con esos límites, la excepción no abre el camino que la regla 2 cierra: el administrador de accesos no puede convertirse en superadministrador ni convertir a otro, ni quitarle el acceso a quien lo controla. Tampoco puede tomar la cuenta de un administrador de empresa —restablecerle la contraseña y entrar como él— para inventar un colaborador y darle acceso. Todo lo que otorga queda con su nombre en `granted_by`. Es la única excepción a la regla 2; extenderla a otro rol requiere una decisión nueva.

**Límite de lo que la base puede garantizar.** La consistencia entre la empresa del rol y la empresa de la asignación no se puede exigir con una llave foránea, porque el campo admite vacío para los roles generales. Se resuelve con una validación programada que rechaza asignar un rol de una empresa dentro de otra. Las reglas 1 a 3, la excepción del administrador de accesos y sus límites, los roles reservados y los pares incompatibles en las cuentas también viven en la aplicación: la base no sabe qué permisos ni qué roles tiene quien está ejecutando la operación. Lo que sí revisa la base, en la migración que los siembra, es que ningún rol vigente salvo `SUPERADMIN` junte un par y que ninguno que no sea de alcance sistema tenga alguno de los ocho de D-122.

**Reglas de negocio**

Roles que trae el sistema:

| code | name | Alcance |
|---|---|---|
| `SUPERADMIN` | Superadministrador | Todo el sistema |
| `ADMIN_ACCESOS` | Administrador de accesos | Todo el sistema |
| `ADMIN_EMPRESA` | Administrador de empresa | Una empresa |
| `OPERACIONES` | Operaciones | Una empresa |
| `CONTABILIDAD` | Contabilidad | Una empresa |
| `ASESOR_VENTAS` | Asesor de ventas | Una sede |
| `TALLER` | Taller | Una sede |

El administrador de accesos da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas, en todas las empresas del grupo (D-55). Es un rol para todo el grupo, no uno por empresa, y por eso su alcance es todo el sistema: ve todas las empresas. Pueden tenerlo varias personas, para que los accesos no se detengan cuando una falta. No registra colaboradores ni sus salidas o regresos (D-57), y sus tres límites están en la excepción a la regla 2. Lo que hace se lo dan sus siete permisos de seguridad (§5.6, D-117). Una misma cuenta no tiene a la vez este rol y el de administrador de empresa, porque juntaría pares incompatibles (D-119).

### 5.6 `permissions`, `role_permissions` y `permission_incompatibilities` — Permisos
Cada acción concreta que se puede autorizar por separado. Un rol es un conjunto de permisos.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(60) UNIQUE | No | Nombre corto de la acción, con el módulo adelante: `compras.registrar_factura` | Al llevar el módulo adelante, se pueden otorgar todos los permisos de un área de una sola vez |
| `module` | VARCHAR(30) | No | A qué parte del sistema pertenece: stock, compras, seguridad | Ordena la pantalla donde se asignan. Con cien permisos sueltos, nadie encuentra el que busca |
| `description` | VARCHAR(150) | No | Qué habilita exactamente este permiso | Es lo que lee quien asigna. Sin explicación se otorgan permisos por el nombre y se termina dando de más |
| `category` | VARCHAR(20) | No | Qué clase de acción es: `ver`, `crear`, `editar`, `gestionar`, `desactivar_anular` o `acciones_especiales` (D-118) | Ordena la matriz dentro de cada módulo: quien asigna encuentra juntos los permisos que solo leen y aparte las excepciones controladas. No hay "eliminar", porque nada se borra |

**Relaciones.** `permissions` no apunta a nadie: es un catálogo fijo que se carga con el sistema; cada paso trae los suyos con una migración de `iam` (D-95). Le apuntan `role_permissions` y `permission_incompatibilities`, dos veces (D-119, más abajo). `role_permissions` es la tabla intermedia que une roles con permisos, con su historia (D-96):

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `role_id` | BIGINT FK → `roles` | No | El rol | |
| `permission_id` | BIGINT FK → `permissions` | No | El permiso que incluye | |
| `granted_at` | TIMESTAMPTZ | No | Desde cuándo lo incluye | |
| `granted_by` | BIGINT FK → `users` | No | Quién lo marcó; `SYSTEM` en la semilla | Lo autorizado tiene responsable, como una asignación (§5.7) |
| `revoked_at` | TIMESTAMPTZ | Sí | Desde cuándo dejó de incluirlo. Vacío = vigente | Desmarcar un permiso no borra la fila: nada se borra, y se puede saber qué permitía un rol en una fecha pasada |
| `revoked_by` | BIGINT FK → `users` | Sí | Quién lo desmarcó | |

Un rol incluye un permiso a lo sumo una vez mientras está vigente.

**El catálogo** (D-95, D-117). Sesenta y dos permisos: cincuenta para los pasos 1 a 11 y doce de seguridad, en `<módulo>.<acción>`, en minúsculas y en español. Cada uno lleva su categoría (D-118): "ver" es una lectura; "crear", "editar" y "desactivar_anular" hacen solo eso; "gestionar" agrupa crear, editar y desactivar un recurso y no se parte; "acciones_especiales" son las excepciones controladas y los hechos con nombre propio, que se dan por separado. Cada paso carga los suyos cuando construye sus operaciones; hoy existen los de los pasos 1 a 5 (V45, V46) y, con la segunda ronda de la auditoría, los doce de seguridad (V54) <!-- verificar al integrar fase 2: V53 a V55 -->. La categoría de los permisos de los pasos 6 a 11 la confirma la migración de su paso.

| code | module | category | description | Paso |
|---|---|---|---|---|
| `catalogos.ver` | catalogos | ver | Leer monedas, tipos de cambio, países, divisiones, documentos tributarios, catálogos de flota, estados y motivos, documentos del vehículo (D-43) | 1, 4, 5 |
| `catalogos.gestionar_generales` | catalogos | gestionar | Monedas, países y sus escalones, divisiones, tipos de documento tributario (D-43) | 1 |
| `catalogos.gestionar_flota` | catalogos | gestionar | Tipos de vehículo, marcas y modelos, colores, equipamiento (§7) | 4 |
| `catalogos.gestionar_estados` | catalogos | gestionar | Estados del vehículo, motivos y caminos con el permiso que exige cada uno, tipos de documento del vehículo (D-100) | 5, 9 |
| `catalogos.corregir_tipo_cambio` | catalogos | acciones_especiales | Corregir a mano una tasa cargada por el proceso automático (§4.8, D-103) | 7 |
| `configuracion.ver_estructura` | configuracion | ver | Empresas, sedes, marcas comerciales y ubicaciones del alcance | 2 |
| `configuracion.gestionar_empresas` | configuracion | gestionar | Crear y editar empresas, corregir su documento, activarlas y desactivarlas (D-85) | 2 |
| `configuracion.gestionar_sedes_y_marcas` | configuracion | gestionar | Sedes y marcas comerciales, con sus direcciones y su estado | 2 |
| `configuracion.gestionar_ubicaciones` | configuracion | gestionar | Ubicaciones, con sus direcciones y su estado (D-79, D-90) | 2 |
| `configuracion.ver_colaboradores` | configuracion | ver | La fila de cada colaborador sin sus datos personales: nombre, puesto, sede, fechas y si tiene cuenta (D-105) | 3 |
| `configuracion.ver_datos_personales_colaboradores` | configuracion | ver | Documento, teléfono y foto, y la exportación de colaboradores (D-93, D-105) | 3 |
| `stock.ver` | stock | ver | Inventario y ficha de la unidad: datos, historial de estados, lecturas, documentos, equipos, llaves y cesiones; vencidas; habilitación. Toda la empresa (D-14), sin costos | 5, 9, 10 |
| `stock.ver_costos` | stock | ver | El costo visto desde la unidad: valor de factura, notas de crédito y valor del vehículo en la ficha, la columna de costo del exportable y los indicadores derivados de costos (§2.5) | 5, 7, 11 |
| `stock.crear_vehiculo` | stock | crear | Dar de alta una unidad | 5 |
| `stock.editar_vehiculo` | stock | editar | Color, motor, año, marca comercial, ubicación actual | 5 |
| `stock.cambiar_estado` | stock | acciones_especiales | Recorrer un camino que no exige un permiso propio (D-100) | 5 |
| `stock.habilitar` | stock | acciones_especiales | El camino En habilitación → Disponible | 5 |
| `stock.vender` | stock | acciones_especiales | El camino Disponible → Vendido | 5 |
| `stock.entregar` | stock | acciones_especiales | El camino Vendido → Activo | 5 |
| `stock.atender_en_taller` | stock | acciones_especiales | Los caminos hacia Mecánica y Puesta a punto, y de vuelta a Disponible | 5 |
| `stock.recuperar` | stock | acciones_especiales | Los caminos por morosidad y resolución de contrato (Custodia) | 5 |
| `stock.registrar_kilometraje` | stock | acciones_especiales | Lecturas de taller, entrega, devolución y manuales (§8.5) | 5 |
| `stock.registrar_sustitucion_odometro` | stock | acciones_especiales | El cambio de tablero, el hecho distinto que admite que la siguiente lectura sea menor (§8.5) | 5 |
| `stock.corregir_placa` | stock | acciones_especiales | Corregir una placa ya registrada, con motivo (§13.1) | 5, 6 |
| `stock.corregir_vin` | stock | acciones_especiales | Corregir un VIN ya registrado, con motivo (§13.1) | 5, 6 |
| `stock.anular_vehiculo` | stock | desactivar_anular | Anular una unidad por acción explícita, con el candado de §9.6 (D-104) | 7 |
| `stock.cargar_migracion_inicial` | stock | crear | La carga del Excel que da de alta unidades (D-18) | 5 |
| `compras.ver` | compras | ver | Proveedores, órdenes, facturas y notas de crédito, con sus importes | 7 |
| `compras.gestionar_proveedores` | compras | gestionar | Crear y editar proveedores, con su dirección y su estado (§9.1) | 7 |
| `compras.gestionar_plantillas` | compras | gestionar | Plantillas de orden de compra y sus tipos (D-35) | 7 |
| `compras.gestionar_ordenes` | compras | gestionar | Crear, editar el borrador, enviar, cerrar a mano y anular órdenes (§9.2) | 7 |
| `compras.registrar_factura` | compras | crear | Registrar una factura de compra con sus líneas | 7 |
| `compras.registrar_factura_antigua` | compras | acciones_especiales | Una factura de más de veinte días (§9.3) | 7 |
| `compras.anular_factura` | compras | desactivar_anular | Anular un comprobante completo (§9.3) | 7 |
| `compras.registrar_nota_credito` | compras | crear | Registrar una nota de crédito y su reparto; con motivo "anulación de unidad", anula la línea y, si el valor llega a cero, la unidad (§9.4 a §9.6) | 7 |
| `pagos.ver` | pagos | ver | Pagos, aplicaciones, imputaciones y el saldo con cada proveedor, con sus importes | 8 |
| `pagos.registrar_pago` | pagos | crear | Registrar, aplicar e imputar un pago (§10.1 a §10.3) | 8 |
| `pagos.anular_pago` | pagos | desactivar_anular | Anular un pago mal registrado (§10.1) | 8 |
| `habilitacion.registrar_hitos` | habilitacion | editar | Ingreso físico, inmatriculación y el primer registro de la placa (§11.1) | 9 |
| `habilitacion.gestionar_documentos` | habilitacion | gestionar | Cargar los documentos del vehículo (§11.3, D-28) | 9 |
| `habilitacion.gestionar_equipos` | habilitacion | gestionar | Instalar y retirar equipamiento (§11.4) | 9 |
| `habilitacion.gestionar_llaves` | habilitacion | gestionar | Registrar las llaves y dónde están (§11.5, D-29) | 9 |
| `habilitacion.cargar_archivo_sat` | habilitacion | acciones_especiales | La declaración del impuesto vehicular (§11.7) | 9 |
| `habilitacion.ver_cargas` | habilitacion | ver | Las cargas de archivos y sus errores (§11.8) | 9 |
| `fondos.ver` | fondos | ver | La pantalla de fondos y las cesiones por fondo (§12) | 10 |
| `fondos.gestionar_fondos` | fondos | gestionar | Crear y editar fondos (§12.1) | 10 |
| `fondos.registrar_cesion` | fondos | acciones_especiales | Ceder una unidad a un fondo o cerrar la cesión a mano (§12.2) | 10 |
| `fondos.cargar_archivo_garantia` | fondos | acciones_especiales | El archivo de garantía mobiliaria (D-30) | 10 |
| `bitacora.ver` | bitacora | ver | La bitácora de cambios de la empresa (§13.1) | 6 |
| `reportes.ver` | reportes | ver | Exportable de stock, brecha de días, diferencia por inicial; los costos siguen a `stock.ver_costos` (§10.5) | 11 |
| `seguridad.ver_cuentas` | seguridad | ver | Leer y exportar las cuentas: todas con alcance sistema; con una empresa, las de sus colaboradores (D-122) | 3b |
| `seguridad.ver_roles` | seguridad | ver | Leer los roles, el catálogo de permisos y la matriz: los del sistema, los generales y los de una empresa donde actúa (D-122) | 3b |
| `seguridad.dar_acceso` | seguridad | crear | Crear la cuenta de un colaborador o vincularlo a la que ya tiene; solo con alcance sistema (D-58, D-122) | 3b |
| `seguridad.gestionar_colaboradores` | seguridad | gestionar | Registrar y modificar colaboradores, en la empresa o la sede de la asignación (D-122) | 3b |
| `seguridad.gestionar_roles` | seguridad | gestionar | Crear, copiar, editar, activar y desactivar roles propios y marcar sus permisos; solo con alcance sistema (D-123, D-124) | 3b |
| `seguridad.bloquear_cuentas` | seguridad | desactivar_anular | Bloquear cuentas, levantar o cancelar sus bloqueos y leerlos; solo con alcance sistema (§5.4) | 3b |
| `seguridad.registrar_salidas_y_regresos` | seguridad | acciones_especiales | Registrar la salida y el regreso de un colaborador, en la empresa o la sede de la asignación (§5.3, D-122) | 3b |
| `seguridad.asignar_roles` | seguridad | acciones_especiales | Otorgar y retirar roles y leer las asignaciones de una cuenta; solo con alcance sistema (§5.7) | 3b |
| `seguridad.restablecer_contrasenas` | seguridad | acciones_especiales | Restablecer la contraseña de otra cuenta con una provisional; solo con alcance sistema (D-63) | 3b |
| `seguridad.confirmar_vinculo` | seguridad | acciones_especiales | Vincular la cuenta de otra persona, o una sin colaborador, después de ver a sus titulares; solo del superadministrador (D-82) | 3b |
| `seguridad.desvincular_cuentas` | seguridad | acciones_especiales | Deshacer el vínculo equivocado de un colaborador con su cuenta; solo del superadministrador (D-82) | 3b |
| `seguridad.asignar_roles_sin_tenerlos` | seguridad | acciones_especiales | Otorgar roles no reservados cuyos permisos no se tienen; solo del administrador de accesos (D-56, D-120) | 3b |

<!-- verificar al integrar fase 2: las descripciones y las categorías que siembran V53 y V54, en especial la de stock.cargar_migracion_inicial y stock.registrar_kilometraje (los dos repartos dan 6/2/1/6/10) -->

En total: catorce de ver, seis de crear, dos de editar, quince de gestionar, cuatro de desactivar o anular y veintiuno de acciones especiales.

Desde D-117, las cuentas, los accesos, los colaboradores y los roles tienen sus permisos en el catálogo: antes los fijaba un rol (D-95). Esperan otra decisión las correcciones delicadas de un monto de factura y de una fecha ya reportada (convenciones de la API, §14.21), que llevarán un permiso cada una cuando tengan su ruta; el bloque 12 traerá los suyos, entre ellos `clientes.ver_cartera_completa`.

**La matriz** (D-95). Lo que incluye cada rol del sistema. SA = `SUPERADMIN` · AA = `ADMIN_ACCESOS` · AE = `ADMIN_EMPRESA` · OP = `OPERACIONES` · CO = `CONTABILIDAD` · AV = `ASESOR_VENTAS` · TA = `TALLER`.

| Permiso | SA | AA | AE | OP | CO | AV | TA |
|---|---|---|---|---|---|---|---|
| `catalogos.ver` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `catalogos.gestionar_generales` | ✔ | | | | | | |
| `catalogos.gestionar_flota` | ✔ | | | | | | |
| `catalogos.gestionar_estados` | ✔ | | | | | | |
| `catalogos.corregir_tipo_cambio` | ✔ | | | | | | |
| `configuracion.ver_estructura` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `configuracion.gestionar_empresas` | ✔ | | | | | | |
| `configuracion.gestionar_sedes_y_marcas` | ✔ | | | | | | |
| `configuracion.gestionar_ubicaciones` | ✔ | | | | | | |
| `configuracion.ver_colaboradores` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `configuracion.ver_datos_personales_colaboradores` | ✔ | ✔ | ✔ | | | | |
| `stock.ver` | ✔ | | ✔ | ✔ | ✔ | ✔ | ✔ |
| `stock.ver_costos` | ✔ | | ✔ | | ✔ | | |
| `stock.crear_vehiculo` | ✔ | | ✔ | ✔ | | | |
| `stock.editar_vehiculo` | ✔ | | ✔ | ✔ | | | |
| `stock.cambiar_estado` | ✔ | | ✔ | ✔ | | | |
| `stock.habilitar` | ✔ | | ✔ | ✔ | | | |
| `stock.vender` | ✔ | | ✔ | | | ✔ | |
| `stock.entregar` | ✔ | | ✔ | | | | |
| `stock.atender_en_taller` | ✔ | | ✔ | ✔ | | | ✔ |
| `stock.recuperar` | ✔ | | ✔ | | | | |
| `stock.registrar_kilometraje` | ✔ | | ✔ | ✔ | | ✔ | ✔ |
| `stock.registrar_sustitucion_odometro` | ✔ | | ✔ | | | | ✔ |
| `stock.corregir_placa` | ✔ | | ✔ | | | | |
| `stock.corregir_vin` | ✔ | | ✔ | | | | |
| `stock.anular_vehiculo` | ✔ | | ✔ | | | | |
| `stock.cargar_migracion_inicial` | ✔ | | ✔ | | | | |
| `compras.ver` | ✔ | | ✔ | ✔ | ✔ | | |
| `compras.gestionar_proveedores` | ✔ | | ✔ | ✔ | | | |
| `compras.gestionar_plantillas` | ✔ | | ✔ | | | | |
| `compras.gestionar_ordenes` | ✔ | | ✔ | ✔ | | | |
| `compras.registrar_factura` | ✔ | | ✔ | ✔ | ✔ | | |
| `compras.registrar_factura_antigua` | ✔ | | ✔ | | ✔ | | |
| `compras.anular_factura` | ✔ | | ✔ | | ✔ | | |
| `compras.registrar_nota_credito` | ✔ | | ✔ | | ✔ | | |
| `pagos.ver` | ✔ | | ✔ | | ✔ | | |
| `pagos.registrar_pago` | ✔ | | ✔ | | ✔ | | |
| `pagos.anular_pago` | ✔ | | ✔ | | ✔ | | |
| `habilitacion.registrar_hitos` | ✔ | | ✔ | ✔ | | | |
| `habilitacion.gestionar_documentos` | ✔ | | ✔ | ✔ | | | |
| `habilitacion.gestionar_equipos` | ✔ | | ✔ | ✔ | | | ✔ |
| `habilitacion.gestionar_llaves` | ✔ | | ✔ | ✔ | | | ✔ |
| `habilitacion.cargar_archivo_sat` | ✔ | | ✔ | | | | |
| `habilitacion.ver_cargas` | ✔ | | ✔ | ✔ | ✔ | | |
| `fondos.ver` | ✔ | | ✔ | | ✔ | | |
| `fondos.gestionar_fondos` | ✔ | | ✔ | | | | |
| `fondos.registrar_cesion` | ✔ | | ✔ | | ✔ | | |
| `fondos.cargar_archivo_garantia` | ✔ | | ✔ | | ✔ | | |
| `bitacora.ver` | ✔ | | ✔ | | | | |
| `reportes.ver` | ✔ | | ✔ | ✔ | ✔ | | |
| `seguridad.ver_cuentas` | ✔ | ✔ | ✔ | | | | |
| `seguridad.ver_roles` | ✔ | ✔ | ✔ | | | | |
| `seguridad.dar_acceso` | ✔ | ✔ | | | | | |
| `seguridad.gestionar_colaboradores` | ✔ | | ✔ | | | | |
| `seguridad.gestionar_roles` | ✔ | | | | | | |
| `seguridad.bloquear_cuentas` | ✔ | ✔ | | | | | |
| `seguridad.registrar_salidas_y_regresos` | ✔ | | ✔ | | | | |
| `seguridad.asignar_roles` | ✔ | ✔ | | | | | |
| `seguridad.restablecer_contrasenas` | ✔ | ✔ | | | | | |
| `seguridad.confirmar_vinculo` | ✔ | | | | | | |
| `seguridad.desvincular_cuentas` | ✔ | | | | | | |
| `seguridad.asignar_roles_sin_tenerlos` | | ✔ | | | | | |

Las doce filas de seguridad suman 22 marcas: once del superadministrador, siete del administrador de accesos y cuatro del administrador de empresa (D-117). Con ellas, las marcas vigentes de los roles del sistema pasan de 76 a 98.

Cómo se lee cada rol: el superadministrador tiene todos los permisos salvo `seguridad.asignar_roles_sin_tenerlos`, porque "el acceso total es una excepción explícita evaluada con las mismas reglas" (§2.5) y porque es lo que le permite crear cualquier rol bajo la regla 1 de §5.5; otorga cualquier rol, también uno que incluya el permiso que no tiene, porque los reservados los otorga solo él y la regla 2 se aplica a los demás (D-121). El administrador de accesos lee la estructura de las empresas, sus colaboradores y los catálogos, porque lo necesita para dar acceso y fijar el alcance de un rol, y nada del negocio: "su trabajo es dar acceso, no registrar facturas" (D-56); de seguridad tiene los siete permisos de las cuentas, los accesos y los roles, y la excepción a la regla 2 (D-120). El administrador de empresa hace todo lo operativo de su empresa, incluidas las excepciones controladas, y no escribe empresas, sedes, marcas, ubicaciones ni catálogos, que son del superadministrador (§5.2, D-43, §7); de seguridad, lee las cuentas y los roles y registra a sus colaboradores, sus salidas y sus regresos. Operaciones compra y recibe unidades y las habilita; ve los importes de los documentos que registra, pero no el costo por unidad, y no registra facturas atrasadas. Contabilidad lleva el dinero: facturas, también las atrasadas, notas de crédito, pagos, fondos y los costos; no da de alta unidades ni las mueve. El asesor de ventas ve el stock de toda la empresa (D-14) y, en su sede, vende y registra las lecturas de entrega y devolución. El taller, en su sede, mueve unidades a taller y de vuelta, registra lecturas y sustituciones de odómetro y gestiona equipos y llaves. Quién registra la entrega al cliente (`stock.entregar`) espera al negocio (procesos y pantallas, pregunta 4): hasta entonces, el administrador de la empresa.

Contabilidad puede registrar facturas atrasadas y Operaciones no: es la excepción que pide la guía funcional, resuelta con un permiso en vez de con una condición escrita en el programa. La separación crear proveedor / pagar, o registrar unidad / anular, se cumple en esta matriz, pero es una propiedad de la semilla y no una regla: el diccionario separa funciones solo con los pares de seguridad de D-119 (abajo).

**Los pares incompatibles** (D-119). `permission_incompatibilities` guarda los pares de permisos que ningún rol propio incluye juntos y que ninguna cuenta junta entre sus roles vigentes, en ninguna empresa. Quien registra a los colaboradores no es quien les da acceso: así nadie inventa un colaborador y le da acceso (D-57).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `permission_a_id` | BIGINT FK → `permissions` | No | Uno de los dos permisos del par | — |
| `permission_b_id` | BIGINT FK → `permissions` | No | El otro. Siempre el de identificador mayor (`CHECK (permission_a_id < permission_b_id)`) | Un par se guarda una sola vez, en un solo orden, y el `UNIQUE` lo detecta repetido |

<!-- verificar al integrar fase 2: los nombres de las columnas y de las restricciones de V55 -->

Es un catálogo del sistema, como `permissions`: lo carga la migración de `iam` y la aplicación solo lo lee; no lleva `public_id`, columnas de registro ni Row Level Security. Los catorce pares, ✕ = no se juntan:

| Permiso | `seguridad.gestionar_colaboradores` | `seguridad.registrar_salidas_y_regresos` |
|---|---|---|
| `seguridad.dar_acceso` | ✕ | ✕ |
| `seguridad.asignar_roles` | ✕ | ✕ |
| `seguridad.bloquear_cuentas` | ✕ | ✕ |
| `seguridad.restablecer_contrasenas` | ✕ | ✕ |
| `seguridad.confirmar_vinculo` | ✕ | ✕ |
| `seguridad.desvincular_cuentas` | ✕ | ✕ |
| `seguridad.asignar_roles_sin_tenerlos` | ✕ | ✕ |

- **En un rol propio**, guardar o copiar uno que junta un par se rechaza (`ROLE_PERMISSIONS_INCOMPATIBLE`).
- **En una cuenta**, se suman los permisos de todos sus roles vigentes, en todas sus empresas, salvo los de sus asignaciones de `SUPERADMIN`: el superadministrador hace las dos cosas (D-57), y su matriz es la única de un rol vigente que junta pares. Otorgar un rol que dejaría a la cuenta con un par se rechaza (`ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`), y también editar un rol de modo que alguno de sus titulares quede con uno (`ROLE_HOLDERS_DUTIES_INCOMPATIBLE`). Así `ADMIN_ACCESOS` y `ADMIN_EMPRESA` siguen sin juntarse, como pedía D-71, y tampoco dos roles propios que los imiten.
- **Lo que lo sostiene en una carrera.** La base no ve dos asignaciones ni dos roles a la vez, así que lo revisa la aplicación, y para que dos operaciones simultáneas no lo burlen, guardar un rol, asignar uno, restablecer una contraseña y otorgar un acceso toman un mismo candado de transacción; guardar un rol bloquea además cada cuenta titular, en orden de identificador.

**Reglas de negocio**

`role_permissions` dice qué permisos incluye cada rol, y conserva los que dejó de incluir (D-96).

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
| `expires_at` | TIMESTAMPTZ | Sí | Momento en que caduca. Vacío significa que no caduca. Cuando la pantalla elige un día, caduca al empezar el día siguiente (D-87) | Para accesos temporales, como el de un auditor externo que necesita entrar un mes. Sin esto, el acceso queda vivo hasta que alguien se acuerde de retirarlo |
| `revoked_at` | TIMESTAMPTZ | Sí | Cuándo se le quitó. Vacío significa que lo tiene todavía | El acceso se retira, no se borra: hay que poder explicar qué permisos tenía una persona el día que hizo una operación |
| `revoked_by` | BIGINT FK → `users` | Sí | Quién se lo quitó: quien lo retiró, quien registró la salida que lo hizo caer, o `SYSTEM` cuando venció. Vacío mientras lo tenga. Obligatorio al retirar | Sin este dato se sabe quién dio el acceso pero no quién lo quitó, que es la mitad de la pregunta en una revisión de accesos, como en los bloqueos (§5.4). Con el administrador de accesos retirando roles en todas las empresas, la pregunta va a aparecer |
| `revocation_reason` | VARCHAR(20) | Sí | Por qué se retiró: `retiro` (alguien lo quitó), `caducidad` (venció), `baja_colaborador` (la persona salió de esa empresa) o `baja_cuenta` (la cuenta se bloqueó con motivo `baja`). Obligatorio al retirar | Distingue un acceso que alguien decidió quitar de uno que venció solo o que cayó por una salida. Sin él, los cuatro casos se ven iguales al revisar |

**Relaciones.** Es la tabla que une casi todo el bloque: apunta a `users` (tres veces: quién recibe el acceso, quién lo otorgó y quién lo retiró), a `roles`, a `companies`, a `branches` y a `commercial_brands`. No le apunta nadie.


#### Un usuario con acceso a dos empresas

El rol define **qué puede hacer** una persona; la asignación define **sobre qué**. Por eso el acceso a dos empresas no se configura en el rol: se otorga con dos filas en esta tabla.

| Persona | Rol | Sobre |
|---|---|---|
| Tere | Contabilidad | Millennium Perú |
| Tere | Contabilidad | Millennium Colombia |

Tere hace lo mismo en ambas, pero es un acceso por empresa. Si mañana deja de llevar la contabilidad de una, se retira esa fila y la otra queda intacta.

Nada obliga a que el rol sea el mismo en las dos:

| Persona | Rol | Sobre |
|---|---|---|
| Alejandro | Operaciones | Millennium Colombia |
| Alejandro | Asesor de ventas | Millennium Perú — sede San Isidro |

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
| El mismo rol no se puede dar dos veces sobre lo mismo —la misma empresa, sede y marca—, mientras siga vigente. Sobre dos empresas distintas, sí | Asignaciones duplicadas que después nadie sabe cuál retirar, sin impedir que Tere tenga Contabilidad en dos empresas |

Lo que valida la aplicación, porque la base no puede verlo en una sola fila: para otorgar un rol de alcance empresa o sede, la cuenta debe estar vinculada a un colaborador activo de esa empresa; para uno de alcance sistema, a un colaborador activo de alguna empresa (§5.3). La única excepción es el rol del primer superadministrador, que se otorga al arrancar (§5.2).

También valida que la empresa, la sede y la marca de una asignación nueva estén activas (D-84): un acceso nuevo no se otorga sobre un local cerrado ni sobre una marca retirada. Las asignaciones que ya existían sobre lo que se desactivó siguen vigentes y explicables.

También valida que un rol reservado nunca se limite a una marca (D-59, D-121), y `ADMIN_EMPRESA` es uno de ellos. Administrar una empresa —registrar sus colaboradores, sus salidas y sus regresos— alcanza a la empresa entera, y un colaborador no pertenece a una marca. La base no puede verlo, porque lo que incluye el rol no está en la fila de la asignación. Los demás roles de empresa y de sede sí pueden limitarse a una marca, como el asesor de Lima Bikes; la API dice de cada rol si admite marca (`limitableToBrand`).

**La caducidad es un momento** (D-87). `expires_at` guarda el instante exacto en que el acceso deja de valer. Cuando la pantalla deja elegir un día, el acceso vale ese día completo y caduca al empezar el siguiente en la zona de la empresa, la de su país (§2.6), igual que el fin excluido de un período `[inicio, fin)` (D-40): "vence el 31/12/2026" en una empresa de Perú se guarda como `2027-01-01T05:00:00Z`. Como es un momento, cambiar después la zona de una sede no mueve una caducidad ya guardada.

#### Quién otorga y retira roles

Otorgar y retirar roles exige `seguridad.asignar_roles` por una asignación de alcance sistema (D-117, D-122); la semilla lo da a dos roles del sistema (D-55, D-57):

| Quién | Qué roles otorga y retira | Sobre qué cuentas |
|---|---|---|
| Superadministrador | Cualquiera, también los reservados, sobre cualquier empresa, sede o marca (D-121) | Cualquiera, salvo `SYSTEM` y la suya |
| Administrador de accesos | Cualquiera que no sea reservado, aunque no tenga sus permisos (`seguridad.asignar_roles_sin_tenerlos`, D-120), sobre cualquier empresa, sede o marca | Cualquiera, salvo `SYSTEM`, la suya y las protegidas (§5.5) |
| Un rol propio general de alcance sistema con `seguridad.asignar_roles` | Los no reservados cuyos permisos tiene por asignaciones de alcance sistema (§5.5 reglas 1 y 2, D-122) | Cualquiera, salvo `SYSTEM`, la suya y las protegidas |
| Cualquier otro rol, también el administrador de empresa | Ninguno: no tiene el permiso, o lo tiene por un rol de empresa, donde no cuenta | — |

- Retirar una asignación exige poder otorgarla: el administrador de accesos no retira un rol reservado.
- Nadie se otorga ni se retira roles a sí mismo, tampoco un superadministrador (D-70).
- Quien otorga o retira tiene que estar vinculado a un colaborador activo (§5.2).
- El administrador de empresa no otorga roles, ni siquiera en su empresa: registra a las personas, y el acceso lo da otro (§5.3).
- **Separación de funciones** (D-119, que reemplaza a D-71). Otorgar un rol que dejaría a la cuenta con las dos partes de un par incompatible, entre todos sus roles vigentes y en cualquier empresa, se rechaza, y el rechazo nombra los pares. Las asignaciones de `SUPERADMIN` no cuentan en esa suma. Así una cuenta sigue sin tener a la vez `ADMIN_ACCESOS` y `ADMIN_EMPRESA`: si una persona pasa de un puesto al otro, primero se le retira el que tenía.
- **Un rol reservado exige contraseña propia** (D-72, D-121). Un rol reservado se otorga solo a una cuenta que ya cambió su contraseña provisional (§5.4). El primer superadministrador no se otorga así: lo crea el arranque (§5.2).
- El mismo rol puede tenerse sobre dos empresas a la vez, como Tere; retirar uno no toca el otro.
- La segunda empresa de una persona la vincula el administrador de accesos: al dar acceso a su colaborador en la otra empresa con el mismo correo, el perfil se vincula a la cuenta que ya tiene, sin crear otra, siempre que la sigla y el número del documento sean los mismos (§5.3, D-82). Un superadministrador solo hace falta si la cuenta es protegida, si el documento no coincide o si la cuenta no tiene colaborador.

Las dos reglas nuevas las valida la aplicación: la base no ve dos asignaciones a la vez ni la contraseña de la cuenta al guardar una asignación.

### 5.8 Cómo se resuelven los permisos

1. **Los permisos se suman.** Con varios roles, la persona puede hacer todo lo que permite cualquiera de ellos. No existen reglas que quiten permisos. Un sistema con permitir y quitar obliga a definir cuál manda, y a partir de ahí nadie puede saber qué puede hacer alguien sin revisar el programa.
2. **El alcance baja, nunca sube.** Un rol sobre una empresa alcanza todas sus sedes. Uno sobre todo el sistema alcanza todas las empresas. Nunca al revés.
3. **Consecuencia práctica:** como los permisos se suman, dar un rol amplio "por un rato" no se compensa dando además uno restringido. Los roles deben ser estrechos y combinables.
4. **Los accesos retirados dejan de valer**, pero se conservan para poder reconstruir quién podía hacer qué en una fecha pasada.
5. **Un acceso que caduca se cierra como si lo hubieran retirado** (D-64): un proceso automático le pone como fecha de retiro la de su caducidad —no la del momento en que pasa el proceso—, con motivo `caducidad`. Tiene que ser así y no quedar abierto con la fecha vencida, porque de lo contrario esa persona no podría recibir de nuevo el mismo acceso —el sistema lo vería como duplicado del que sigue abierto— y el auditor externo que vuelve el año siguiente quedaría fuera. El acceso deja de valer exactamente al caducar, sin esperar al proceso: lo vigente es lo que no está retirado y cuya caducidad no llegó. Si se otorga de nuevo el mismo rol sobre lo mismo antes de que pase el proceso, en ese momento se cierra la asignación vencida con esa misma llave, y solo esa.
6. **El permiso dice qué; la asignación dice dónde, también al leer** (D-99). Un permiso otorgado por un rol de sede actúa solo en esa sede —y lee toda la empresa, D-14—; por uno de empresa, en toda la empresa; por uno de alcance sistema, en todas. Las lecturas de recursos de empresa se filtran por el permiso y el alcance de la asignación que lo trae, no por la suma de las empresas que la cuenta alcanza con cualquier rol: un permiso que se tiene en Perú no abre lo mismo en Colombia. La frontera entre empresas (D-66) queda como piso.
7. **Las cuentas, los accesos, los colaboradores y los roles también van por permiso, con su alcance** (D-117, D-122; hasta entonces iban por rol, D-95). Los habilitan los doce permisos `seguridad.*` de §5.6. Los ocho de accesos y roles solo cuentan desde asignaciones de alcance sistema y solo se marcan en roles de alcance sistema; los dos de los colaboradores actúan en la empresa o la sede de su asignación, como cualquier permiso (regla 6); `ver_cuentas` y `ver_roles` leen según la suya. Lo que la separación de funciones separa no lo puede juntar un rol propio ni una cuenta, por los pares incompatibles (D-119), y lo que solo hace el superadministrador lo cuidan los roles reservados (D-121). Los límites sobre el destino —la propia cuenta, una cuenta protegida, `SYSTEM`, el último superadministrador— quedan fijos en el código, con sus políticas, y responden los mismos códigos de siempre.

### 5.9 Semilla inicial y protección del acceso

El usuario `SYSTEM` no puede iniciar sesión y figura como responsable de toda carga masiva y de todo proceso automático, incluida la actualización por archivo del SAT. Sin él, esos registros quedarían sin autor. Tampoco se vincula a un colaborador, no recibe roles, no se bloquea y no se le restablece la contraseña: no es una persona y nadie lo administra.

**Qué pasa automáticamente al registrar una baja.** El bloqueo con motivo `baja` lo crea solo el sistema, cuando se registra la salida del último colaborador activo de la cuenta o se desvincula la cuenta de él (§5.3, D-69); nadie lo abre ni lo cierra a mano. En la misma operación, el sistema revoca todas las asignaciones activas de esa persona, con motivo `baja_cuenta`. Son dos acciones distintas y las dos hacen falta: bloquear la cuenta corta el acceso en el momento, y retirar los roles evita que, si esa cuenta se reactiva algún día, la persona recupere de golpe todo lo que tenía. Automatizarlo es lo que impide que el segundo paso dependa de que alguien lo recuerde.

Nada se pierde al revocar: las asignaciones quedan con su fecha de retiro y se puede seguir explicando qué accesos tenía esa persona en cualquier fecha pasada.

Las vacaciones y las licencias funcionan al revés a propósito: ahí los roles **no** se tocan. Si se retiraran, al volver habría que devolvérselos y alguien olvidaría uno o daría uno de más. La persona regresa con exactamente los mismos accesos que tenía.

**Protección necesaria:** el sistema impide retirar el último superadministrador activo, ya sea quitándole el rol, bloqueando su cuenta, registrando la salida que la cierra o desvinculándola de su último colaborador activo. Sin ninguno, no hay forma de volver a entrar sin tocar la base a mano.

No basta con que quede otro hoy (D-65). Una operación que le quita, o le quitará, el rol a un superadministrador solo procede si queda otra cuenta que lo sea **de forma duradera**: con el rol `SUPERADMIN` vigente y sin caducidad, y sin ningún bloqueo en curso ni programado. Esas operaciones son retirarle el rol, bloquearlo ahora o con un bloqueo programado, registrar la salida que cierra su cuenta y desvincular la cuenta de su último colaborador activo. Un superadministrador cuyo rol caduca en un mes, o que tiene unas vacaciones programadas, no cuenta como respaldo: al llegar esa fecha el sistema podría quedarse sin ninguno sin que nadie haya hecho nada. Además, esas operaciones se hacen de a una: si dos personas intentan al mismo tiempo retirar a dos superadministradores distintos, la segunda espera a la primera y ve su resultado, en vez de pasar las dos.

**Es el único freno de una salida** (D-60). La salida de un colaborador siempre retira sus roles en esa empresa y, si era el último colaborador activo de la cuenta, la cierra, también si es la de un administrador de accesos o de empresa: quitarle el acceso a quien se fue no espera a nadie. Solo se detiene si dejaría al sistema sin un superadministrador activo.

**El primer superadministrador no puede dar acceso a nadie, otorgar roles ni registrar salidas o regresos** hasta vincularse a su perfil de colaborador (§5.2, D-89). Lo mismo vale para el administrador de accesos y el de empresa: actúan solo mientras su cuenta está vinculada a un colaborador activo (D-57). Vincularse a sí mismo es solo de ese momento: una cuenta que ya tiene un colaborador activo no se vincula a sí misma a otro (D-70).

**No hace falta proteger al último administrador de accesos.** El superadministrador puede hacer todo lo que él hace, así que el sistema nunca se queda sin quien administre los accesos (D-55).

### 5.10 Candidatos del bloque 2

Campos que hoy no entran porque nadie los usa todavía, con la condición que los admitiría. Los que estaban en espera antes se incorporaron: `last_login_at`, `password_updated_at` y `mfa_secret` entraron a `users`; `phone` y `photo_path`, a `employees`; `expires_at`, a las asignaciones; y los roles propios por empresa quedaron resueltos al cerrar D-7.

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `termination_reason` | `employees` | Una pantalla con la historia del colaborador, o un reporte de rotación, que muestre por qué salió. Si el reporte agrupa por motivo, entra como catálogo y no como texto libre |

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
| Cierre por inactividad | 2 horas (D-127) | Una sesión olvidada en una computadora compartida no queda abierta indefinidamente, y una pausa larga en el patio o el taller no hace perder el trabajo |
| Duración máxima | 8 horas, aunque se esté usando | Obliga a volver a entrar en cada jornada |
| Cambio de contraseña | Se cierran las demás sesiones de la persona; sigue abierta la que hizo el cambio | Quien conocía la contraseña anterior pierde el acceso en ese momento |
| Restablecimiento de la contraseña por un administrador | Se cierran todas las sesiones de la persona | Nadie sigue dentro con la contraseña anterior (§5.4) |
| Bloqueo de la cuenta | Se cierran todas sus sesiones cuando el bloqueo empieza, también si estaba programado | Ver §5.4 |
| Cerrar sesión | Se borra la sesión en el servidor, no solo en la pantalla | La cookie que quedara en el navegador ya no sirve para nada |
| Roles, bloqueos y contraseña provisional | Se leen de la base en cada operación, no se guardan en la sesión | Un rol otorgado o retirado vale desde la operación siguiente, y un bloqueo programado corta la sesión apenas empieza, sin esperar a que la persona vuelva a entrar |

**Protección de la cookie.** Viaja marcada como inaccesible para el código de la página (`HttpOnly`), solo hacia el mismo sitio (`SameSite=Strict`) y, en producción, solo por HTTPS (`Secure`). Como el navegador la envía por sí solo, toda operación que modifica datos exige además un token contra solicitudes falsificadas (CSRF) que únicamente la propia aplicación puede leer.

**Detalles de la implementación.** La cookie se llama `MM_SESSION`. Al iniciar sesión se cambia el identificador de la sesión, para que uno obtenido antes no sirva después. Cerrar sesión es `POST /api/v1/authentication/sign-out`. Las 8 horas se cuentan desde que se creó la sesión. El token CSRF llega en la cookie `XSRF-TOKEN` y vuelve en el encabezado `X-XSRF-TOKEN`, también al iniciar sesión y al registrarse.

**Requisito de despliegue.** La aplicación web y la API se sirven desde el mismo origen —mismo dominio y puerto—, detrás de un proxy inverso. En desarrollo lo resuelve el proxy del servidor de Angular.

**Tablas técnicas.** `spring_session` y `spring_session_attributes` las administra la librería de sesiones (§2.2). `spring_session.principal_name` guarda el correo de la persona —es lo que permite cerrar todas sus sesiones— y por eso admite 255 caracteres, igual que `users.email`. Las sesiones vencidas se borran solas cada minuto.

### 5.13 Lo que el backend implementa hoy

El paso 3 completó lo que el bloque 2 había dejado provisional: colaboradores, dar acceso, asignaciones con su empresa, sede y marca, bloqueos, caducidad y la frontera entre empresas en la base (§2.5). También alineó dos columnas con el diseño: V33 agregó `user_roles.revoked_by` (§5.7), y las asignaciones retiradas antes figuran retiradas por `SYSTEM`, porque no se sabe quién lo hizo; V34 quitó `employees.termination_reason`, que sigue como candidato (§5.10). Lo que sigue es lo que todavía no llega al diseño de este capítulo. Nada de esto cambia el diseño: es lo que falta para llegar a él.

| Tema | Hoy | Cuándo se completa |
|---|---|---|
| `permissions` y `role_permissions` | Existen desde el paso 3b (V45), con los permisos de los pasos 1 a 4 y la matriz de los siete roles del sistema para ellos; el paso 5 trajo los suyos (V46), y cada paso siguiente agrega los suyos con una migración de `iam`. Las operaciones de negocio y las lecturas de datos exigen un permiso; las cuentas, los accesos, los colaboradores y los roles, todavía un rol (§5.8, D-95) | Cada paso trae sus permisos (§5.6). Seguridad pasa a permisos con la segunda ronda de la auditoría, en construcción desde el 20/10/2026: la categoría (V53), los doce permisos `seguridad.*` con sus 22 marcas (V54) y los pares incompatibles (V55) (D-117 a D-119) <!-- verificar al integrar fase 2: números de migración --> |
| Separación de funciones | Por rol: una cuenta no tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` a la vez (D-71), y un rol propio podría juntar lo que esos dos separan | Con la segunda ronda: los pares de `permission_incompatibilities`, en los roles propios y en las cuentas (D-119) |
| Roles reservados | Por su código: `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA` | Con la segunda ronda: por lo que incluyen (D-121) |
| Crear un rol a partir de otro | No existe: un rol propio nace sin permisos | Con la segunda ronda: la copia exacta (D-124) |
| Qué lee un rol global | Lo que su matriz le da (D-68): el superadministrador, todo; el administrador de accesos, los catálogos, la estructura de las empresas y sus colaboradores, que necesita para dar acceso, y nada del negocio | Resuelto con la matriz (§5.6) |
| Quién lee los colaboradores | La fila mínima —nombre, puesto, sede, si está activo— cualquier rol con `configuracion.ver_colaboradores`, que la semilla da a los siete, para los selectores y los nombres de autor; el documento, el teléfono, la foto y la exportación, solo con `configuracion.ver_datos_personales_colaboradores`: los dos roles globales y el administrador de la empresa (D-93, D-105) | Resuelto con la matriz |
| La marca de una asignación | Acota dónde se otorga un rol, pero todavía no acota lo que se lee: hoy ningún dato que se lee pertenece a una marca. Decidido (D-102): acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, en el código | Con el paso 5: los vehículos son el primer dato que pertenece a una marca |
| `roles.company_id` | Existe desde el paso 3b (V45): los roles propios de una empresa los crea el superadministrador, con el código único por empresa (D-98) | Resuelto |
| Registro público | Sigue pidiendo solo correo y contraseña y crea una cuenta sin colaborador, contra D-38. Viene apagado (D-32) | Fuera del alcance del paso 3 (D-83): sigue apagado, y el registro con perfil se construye antes de encenderlo en cualquier entorno, con la forma de listar sedes y tipos de documento sin sesión que fijen las convenciones de la API (§14.16) |
| Regreso de un colaborador | Reemplaza la fecha de ingreso y borra la de salida: el período anterior en esa empresa se pierde | Necesita un diseño en este diccionario (§14), antes de cargar los colaboradores reales |
| Restablecer una contraseña | La clave provisional se muestra a quien la restablece, que se la hace llegar a la persona (D-63). Lo ideal es un enlace de un solo uso al correo de la persona, para que nadie más conozca la clave | Cuando exista un servicio de correo. Entonces cambian D-63, §5.4 y §5.12 |

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
| `tax_id` | VARCHAR(20) | No | El número de ese documento, sin guiones, espacios ni puntos | Es el identificador de la empresa ante el Estado. Se guarda normalizado para que `10429...` y `10-429...` no se registren como dos empresas |
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
| 1 | 1 · PE | 1 · RUC | 20512345671 | Millennium Mobility S.A.C. | Millennium | 1 · PEN |
| 2 | 2 · CO | 3 · NIT | 901234567 | Millennium Mobility Colombia S.A.S. | Millennium | 3 · COP |

Una empresa por país, cada una con su propia razón social y su propio documento tributario.

**Reglas de negocio**

- Hay una sola empresa por país, y cada empresa pertenece a un único país: expandirse crea una empresa nueva, no una sede de la existente.
- Una plantilla de orden de compra puede imprimir un emisor editado (§9.2), pero eso no crea otra empresa. Si algún día aparece otra razón social en el mismo país, será otra fila de esta tabla y habrá que revisar D-5.
- El número de documento tributario no se repite dentro del mismo país.
- El número se guarda normalizado, sin guiones, espacios ni puntos, para que la misma empresa no se registre dos veces escrita de formas distintas.
- El tipo de documento es del país de la empresa, está activo y aplica a empresas (§4.7). Si el tipo declara una longitud, el número la respeta, y si tiene dígito de control, el número lo cumple (D-75). Las dos cosas se verifican al registrar la empresa y cuando cambian el número o el tipo (D-91). Que sea del mismo país lo impide la base, porque la llave hacia el tipo incluye el país; lo demás lo valida la aplicación.
- **El país de una empresa no cambia; el tipo y el número de su documento los corrige el superadministrador** (D-85). Una empresa registrada con el documento equivocado se corrige en la misma fila, y la corrección se verifica de nuevo como al registrarla: un tipo del país de la empresa que aplique a empresas, la longitud, el dígito de control y que el número no se repita en el país. El país no se corrige: una empresa constituida en otro país es otra empresa, con sus sedes, su moneda y sus documentos (D-5).
- La razón social es la del registro público y no se ajusta para que se vea mejor: es la que se imprime en comprobantes y contratos.
- Una empresa que deja de operar se desactiva, nunca se borra: su historial de compras y contratos debe seguir siendo consultable.
- **A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo** (D-84). Bajo una empresa inactiva no se registran sedes, marcas, ubicaciones ni colaboradores, ni se otorgan roles; en una sede inactiva no se registran colaboradores ni ubicaciones propias, ni se otorgan roles; sobre una marca inactiva no se otorgan roles. Lo que ya apuntaba a ella sigue siendo válido y se sigue modificando mientras ese vínculo no cambie: un colaborador de una sede que cerró se corrige sin cambiarle la sede. Es el mismo criterio que una moneda o un tipo de documento desactivados (§4.1, §4.7). Reactivar algo es volver a colgarlo (D-94): bajo una empresa inactiva no se reactivan sedes, marcas ni ubicaciones, ni regresa un colaborador; primero se reactiva la empresa. Desactivar no espera a nada.
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

Una marca que se retira se desactiva: no recibe asignaciones de rol nuevas, y las que ya la tienen siguen vigentes y explicables (D-84, §6.2).

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
| `timezone` | VARCHAR(50) | No | La zona horaria de esta sede, con el nombre IANA de una región (`America/Lima`, §2.6) | Aquí ocurren los hechos que se fechan: ingreso del vehículo, instalación de GPS, entrega. En países con varias zonas horarias, tomar la del país haría que los cortes del día salgan corridos |
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
- Una sede que deja de funcionar se desactiva: los vehículos y accesos que la mencionan siguen siendo explicables. No recibe colaboradores, ubicaciones propias ni asignaciones de rol nuevas, y lo que ya la tenía se sigue modificando sin cambiarla (D-84).
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

**Una sede tiene a lo sumo una ubicación propia, activa o inactiva (D-90).** La ubicación `sede_propia` es la sede vista como lugar donde hay vehículos, y es una sola: si una sede tuviera dos, su stock se repartiría entre ellas y "¿dónde está?" dejaría de tener una sola respuesta. Una inactiva también cuenta: si la ubicación propia de una sede se desactivó, se reactiva en vez de crear otra, y conserva el historial de qué unidades estuvieron en ella. Una ubicación propia nueva se enlaza a una sede activa (D-84), y una inactiva no se reactiva mientras su sede siga inactiva: primero se reactiva la sede y después su ubicación.

**La dirección de una ubicación sigue D-8, como la de una empresa o una sede (D-79).** Una ubicación de terceros siempre tiene su dirección; una propia usa la de su sede. La dirección cambia solo por su propia operación, aparte de la que modifica los datos de la ubicación, igual que en una sede. Al cambiarla, la pantalla pregunta si es una corrección o una mudanza. Corregir modifica la misma fila. Mudarse crea una nueva y conserva la anterior, como un patio alquilado que se traslada a otro terreno.

**Una ubicación no pasa de propia a terceros ni de terceros a propia (D-79).** Es otro lugar: la anterior se desactiva y se crea una nueva, salvo que la nueva sea la ubicación propia de una sede que ya tenía una desactivada, que se reactiva (D-90). Así el historial de qué unidades estuvieron en cada una sigue diciendo de quién era el lugar.

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
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general, como los del bloque 1 (D-42, D-73) |

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
- Un tipo inactivo no se elige para un modelo nuevo ni para cambiar el de un modelo existente, y no se registra una unidad de un modelo de ese tipo. El modelo que ya lo tenía lo conserva al editarse, y sus unidades lo siguen referenciando (D-108).

### 7.2 `vehicle_brands` — Marcas de vehículo

La marca del vehículo: Chevrolet, TVS, Bajaj, Hyundai. Es la columna `MARCA` del Excel.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `name` | VARCHAR(80) UNIQUE | No | Nombre oficial de la marca | Sin catálogo, la misma marca aparece como `TVS`, `Tvs` y `T.V.S.`, y contar unidades por marca deja de ser posible |
| `is_active` | BOOLEAN | No | Si se siguen comprando unidades de esta marca | Permite retirarla sin tocar las unidades ya registradas |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general (D-42, D-73) |

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
- Una marca inactiva no recibe modelos nuevos, y no se registra una unidad de un modelo suyo. Los modelos y las unidades que ya tiene la siguen referenciando y se siguen editando (D-108).

### 7.3 `vehicle_models` — Modelos

La combinación de marca y modelo determina el tipo de vehículo y las reglas especiales.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_brand_id` | BIGINT FK → `vehicle_brands` | No | De qué marca es | El nombre del modelo solo identifica junto a la marca: hay modelos con nombres parecidos de marcas distintas |
| `vehicle_type_id` | BIGINT FK → `vehicle_types` | No | Si es auto, moto o trimoto. Se corrige mientras el modelo no tenga unidades; desde la primera queda fijo (D-107) | De aquí sale el tipo de cada unidad, sin que nadie lo elija a mano |
| `name` | VARCHAR(120) | No | Denominación oficial del modelo: `SAIL SEDAN 1.5L LT MT` | Se escribe una vez y todas las unidades la reutilizan |
| `engine_cc` | INTEGER | Sí | Cilindraje en cc, entero. Vacío si no se conoce o no corresponde | Es uno de los datos que hoy se registra por unidad y que en realidad es del modelo: escribirlo una vez evita que la misma moto figure con cilindrajes distintos |
| `fuel_type` | VARCHAR(20) | Sí | Combustible: gasolina, GNV, GLP, diésel, eléctrico, híbrido. Vacío cuando no se conoce: un modelo nuevo no propone ninguno (D-109) | Determina si la unidad necesita certificado de gas. Sin este dato, esa exigencia habría que recordarla a mano modelo por modelo |
| `has_factory_screen` | BOOLEAN | No | Si el modelo ya viene con pantalla instalada de fábrica | Es la regla de la pantalla convertida en dato. Hoy está escrita como una comparación de texto —"Chevrolet más Sail Sedan"— y por eso agregar un modelo obliga a tocar el programa |
| `is_active` | BOOLEAN | No | Si se siguen comprando unidades de este modelo | — |
| `has_units` | BOOLEAN | No | Si el modelo ya tiene al menos una unidad registrada. Nace falso y pasa a verdadero con la primera unidad; no vuelve atrás, porque una unidad no se borra | Desde la primera unidad, el tipo de vehículo del modelo queda fijo: cambiarlo contradiría el tipo de las unidades que ya existen (D-107). Lo escribe `vehiclecatalog` cuando `fleet` le avisa, por su fachada, que registró una unidad del modelo, en la misma operación que el alta; `vehiclecatalog` nunca lee las tablas de `fleet` |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general (D-42, D-73) |

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
- **El tipo de vehículo de un modelo se corrige mientras el modelo no tenga unidades** (D-107). Desde la primera queda fijo, el mismo principio que el código de un catálogo (D-48, D-74): corregirlo después cambiaría el tipo de unidades ya registradas, y con él las reglas que dependen del tipo. La API rechaza la corrección y responde si el tipo todavía se puede corregir, para que la pantalla bloquee el campo. La migración que agrega `has_units` la llena con los modelos que ya tienen unidades.
- Un modelo nuevo es de una marca activa y de un tipo activo; al editarlo, un tipo nuevo tiene que estar activo, y el inactivo que el modelo ya tenía se conserva (D-108, como D-84).
- La exigencia de pantalla sale de este catálogo, no de comparar textos. Agregar un modelo sin pantalla de fábrica es cargar una fila.
- Un modelo que deja de comprarse se desactiva: las unidades ya registradas lo siguen referenciando. No se registra una unidad nueva con un modelo inactivo, ni con uno cuya marca o tipo estén inactivos (D-108, §8.1).
- El combustible de un modelo nuevo no se propone: nace vacío y se elige. Proponer uno haría que un modelo de GLP quedara registrado como de gasolina si nadie cambia el valor, y con él la exigencia del certificado de gas (D-109).
- El cilindraje, cuando se registra, es un número entero de centímetros cúbicos mayor que cero (`vehicle_models_engine_cc_positive_ck`). Se escribe la cifra comercial del modelo, como en los ejemplos —125, 200, 1500—, no la técnica de la ficha (199,26). Un valor con parte decimal (1500,7) se rechaza: no se redondea ni se corta en silencio; 1500,0 es el mismo entero y se acepta. Queda vacío cuando no se conoce o no corresponde, y nunca se escribe 0 para decir «no tiene» (§2.2). No tiene un máximo: nada usa su magnitud (§2.3).
- Un modelo eléctrico no lleva cilindraje: con `fuel_type` eléctrico, `engine_cc` queda vacío (`vehicle_models_electric_engine_cc_ck`).

**Advertencia para la carga inicial.** Los dos catálogos entregados no coinciden entre sí. La guía funcional lista modelos sin marca y el documento de clientes los lista con marca y con otra escritura: `GRAND I10` frente a `I10`, o `GLORY 560 PRO 1.8L MT COMFORT 7 AS` frente a la misma denominación en minúsculas. Cargar ambos sin conciliarlos produce modelos duplicados el primer día, que es exactamente lo que este catálogo existe para evitar.

### 7.4 `colors` — Colores

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `name` | VARCHAR(50) UNIQUE | No | Nombre del color | Es un campo que hoy se escribe libre y termina con `Blanco`, `BLANCO` y `blanco perlado` como tres colores distintos. En un inventario, agrupar por color deja de funcionar |
| `is_active` | BOOLEAN | No | Si se sigue usando | — |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general (D-42, D-73) |

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
| `code` | VARCHAR(20) UNIQUE | No | Código corto: `GPS`, `PANTALLA` | Lo usan las reglas de habilitación, que reconocen el GPS y la pantalla por su código; por eso esos dos no se corrigen. El de otro equipo se corrige mientras ninguna instalación lo use (D-48, D-74) |
| `name` | VARCHAR(60) | No | Nombre del equipo | — |
| `blocks_availability` | BOOLEAN | No | Si sin este equipo la unidad no puede pasar a disponible | El GPS bloquea; la pantalla solo en los modelos que la necesitan. Escrito así, cambiar esa política es marcar una casilla y no modificar el programa |
| `requires_serial` | BOOLEAN | No | Si al instalarlo hay que registrar un número de serie o identificador | El GPS tiene un identificador que después hace falta para ubicar la unidad. Sin este dato, quedaría al criterio de quien carga |
| `is_active` | BOOLEAN | No | Si se sigue instalando | — |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general (D-42, D-73) |

#### `equipment_type_vehicle_types` — A qué vehículos aplica cada equipo

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `equipment_type_id` | BIGINT FK → `equipment_types` | No | El equipo. Parte de la llave | — |
| `vehicle_type_id` | BIGINT FK → `vehicle_types` | No | El tipo de vehículo. Parte de la llave | La pantalla solo tiene sentido en autos; en motos y trimotos no corresponde. Como tabla se puede validar y consultar, cosa que una lista escrita dentro de una columna no permite |
| `is_active` | BOOLEAN | No | Si el equipo sigue aplicando a ese tipo de vehículo | Quitar un tipo de la aplicabilidad desactiva su fila en vez de borrarla, porque nada se borra (§2.5); volver a marcarlo la reactiva. Así se corrige una aplicabilidad mal cargada sin perder que existió (D-73) |

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
- Un equipo no se instala en un tipo de vehículo al que no aplica. Solo cuenta la aplicabilidad activa: una desactivada es como si no existiera, salvo para la historia.
- Un equipo aplica al menos a un tipo de vehículo. Para que deje de instalarse en todos se desactiva el equipo (`is_active`); no se le retiran todos los tipos. Así su retiro se dice en un solo lugar. Cuenta también un tipo de vehículo desactivado: sus unidades ya registradas pueden seguir necesitando el equipo —por ejemplo, un reemplazo de GPS— aunque ya no se registren nuevas (§7.1). Lo valida la aplicación; la base no lo protege.

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
- Una unidad nueva no se registra con un modelo inactivo, ni con uno cuya marca o tipo de vehículo estén inactivos; la que ya lo tiene lo conserva y se sigue editando (D-108, como D-84). Al registrarla, `fleet` le avisa a `vehiclecatalog` que el modelo ya tiene unidades, en la misma operación, y desde entonces el tipo del modelo queda fijo (§7.3, D-107).
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
| `is_automatic` | BOOLEAN | No | Si es el motivo automático reservado de un regreso automático: lo crea el sistema, no se ofrece a las personas y no se edita ni se cierra por la API | Es el motivo con que el regreso automático escribe su fila del historial. Sin él, esa fila tendría que nombrar el motivo de otro camino —el de la venta que venció— y el historial diría que la unidad volvió a Disponible por una Venta (D-110) |

**Relaciones.** Apunta dos veces a `vehicle_statuses`, como origen y como destino, y una tercera cuando hay automatismo. Le apunta `vehicle_status_history`.

**Ejemplo**

| desde | hacia | motivo | tolerancia | al vencerse |
|---|---|---|---|---|
| *(alta)* | En habilitación | Registro de compra | — | — |
| En habilitación | Disponible | Habilitación completa | — | — |
| Disponible | Vendido | Venta | 48 h | Volver a Disponible |
| Vendido | Disponible | Regreso automático por vencimiento *(automático)* | — | — |
| Vendido | Activo | Entrega al cliente | — | — |
| Activo | Mecánica | Mantenimiento preventivo | 72 h | Avisar |
| Activo | Mecánica | Choque | 480 h | Avisar |
| Activo | Custodia | Morosidad | 720 h | Avisar |
| Custodia | Puesta a punto | Resolución de contrato | — | — |
| Puesta a punto | Disponible | Reacondicionada | — | — |

Las dos filas de Mecánica muestran para qué sirve la tabla: mismo camino, distinta expectativa según por qué entró la unidad. La fila de Vendido a Disponible es el motivo automático que el sistema creó para el regreso de la venta: nadie la elige.

**Reglas de negocio**

- Un cambio de estado que no figure aquí se rechaza. No es una advertencia: la unidad no se mueve.
- Todo cambio registra fecha, hora, usuario y motivo. Es la exigencia del flujograma y no admite excepción.
- El tiempo límite se cuenta desde el momento del cambio, y la unidad aparece en la lista de vencidas al pasarse.
- La única transición que ocurre sola es la que tiene configurado un estado de destino al vencerse. Todas las demás las hace una persona.
- **El regreso automático tiene su propio motivo** (D-110). Al guardar un camino de A a B que, al vencerse, mueve sola la unidad a C, el sistema asegura en la misma operación un motivo automático del camino de B a C: código `REGRESO_AUTOMATICO`, nombre «Regreso automático por vencimiento», abierto, sin permiso propio, sin tolerancia y con `is_automatic` verdadero. Si ese camino ya tiene uno, lo reutiliza: hay uno solo por camino. Con el ejemplo de arriba, la Venta, de Disponible a Vendido, crea el motivo automático de Vendido a Disponible.
- Un motivo automático no se ofrece a las personas: no figura entre los caminos que se pueden recorrer al mover una unidad, y elegirlo se rechaza. Tampoco se edita ni se cierra por la API: lo mantiene el sistema.
- Un motivo no se borra: se desactiva, para que el historial siga siendo explicable.

### 8.4 `vehicle_status_history` — Historial de estados

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa de la unidad, copiada de ella al escribir la fila | El historial se consulta por sí solo y crece con cada cambio: con su propia empresa, la base lo separa por empresa sin pasar por la unidad (§2.5, D-111). La empresa de una unidad no cambia, así que nunca discrepa con la de la unidad |
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `from_status_id` | BIGINT FK → `vehicle_statuses` | Sí | Estado anterior. Vacío en el alta | — |
| `to_status_id` | BIGINT FK → `vehicle_statuses` | No | Estado nuevo | — |
| `reason_id` | BIGINT FK → `status_change_reasons` | No | Qué camino y por qué motivo se movió | Sin el motivo, dentro de seis meses nadie puede explicar por qué esa unidad estuvo un mes parada |
| `changed_by` | BIGINT FK → `users` | No | Quién lo hizo. El usuario del sistema cuando fue automático | — |
| `changed_at` | TIMESTAMPTZ | No | Cuándo | — |
| `due_at` | TIMESTAMPTZ | Sí | Cuándo vence la tolerancia de este motivo | Se calcula al registrar el cambio. Se guarda para poder listar las unidades vencidas sin recalcular el plazo de todo el historial en cada consulta |
| `notes` | TEXT | Sí | Detalle libre | — |

**Relaciones.** Apunta a `companies`, a `vehicles`, a `vehicle_statuses` como origen y destino, a `status_change_reasons` y a `users`.

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
- La fila de un regreso automático nombra el motivo automático de su camino (§8.3), la firma el usuario del sistema y no lleva vencimiento, porque ese motivo no tiene tolerancia (D-110). Las filas de regresos escritas antes de D-110, con el motivo del camino que venció, no se reescriben.
- La empresa de cada fila es la de su unidad, y la base separa el historial por esa columna (D-111).
- De aquí sale la fecha en que la unidad pasó a estar lista para entrega, que el Excel guarda como un campo manual y por eso se desincroniza.
- También sale de aquí cuánto tiempo pasó la unidad en cada estado, que es lo que alimenta los indicadores de días promedio y de costo por día paralizado.

**El estado actual se guarda además en la unidad**, aunque se podría deducir de la última fila del historial. Es una excepción deliberada a la regla de no persistir derivados: cada pantalla de stock lista cientos de unidades con su estado, y deducirlo exigiría recorrer el historial de cada una. La condición es que ambos se escriban en la misma operación, de modo que no puedan discrepar.

### 8.5 `vehicle_mileage_readings` — Lecturas de kilometraje

El kilometraje no es un dato que se corrige: es una serie de lecturas. Guardar solo el valor actual serviría para mostrarlo en pantalla y para nada más, y en este negocio hace falta para algo más: el contrato cobra por kilómetro recorrido, y eso exige saber cuánto marcaba al entregar y cuánto marca ahora.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa de la unidad, copiada de ella al registrar la lectura | Las lecturas se consultan por sí solas y crecen con cada registro: con su propia empresa, la base las separa por empresa sin pasar por la unidad (§2.5, D-111) |
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `mileage` | INTEGER | No | Kilómetros que marcaba el odómetro | — |
| `read_at` | TIMESTAMPTZ | No | Cuándo se tomó la lectura | Es la fecha de la lectura, no la del registro: una lectura de taller puede cargarse dos días después |
| `source` | VARCHAR(20) | No | De dónde salió: GPS, taller, entrega, devolución, manual | Una lectura del dispositivo y una que alguien escribió mirando el tablero no merecen la misma confianza |
| `recorded_by` | BIGINT FK → `users` | Sí | Quién la registró. Vacío cuando la trajo el dispositivo | — |
| `notes` | TEXT | Sí | Detalle | Para explicar una lectura que rompe la secuencia |

**Relaciones.** Apunta a `companies`, a `vehicles` y a `users`.

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
| `requires_permission_id` | `status_change_reasons` | Adoptado (D-100): cada camino nombra el permiso que exige; un camino sin permiso propio exige `stock.cambiar_estado`. Entra con la tabla, en el paso 5 |


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
| `tax_id` | VARCHAR(20) | No | Número del documento, sin guiones, espacios ni puntos | Va en la factura y es lo que revisa el auditor |
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
| 2 | 1 · Millennium Perú | 20605412344 | Derco Perú S.A. | Derco |

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
| `issuer_tax_id` | VARCHAR(20) | No | Número de documento del emisor, sin guiones, espacios ni puntos | Se propone el de la empresa |
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
| Lima Bikes · Vehículos | Lima Bikes | Vehículos | `OC-LBV` | Millennium Mobility S.A.C. · RUC 20512345671 | Henry Saldaña, Gerente | Alejandro Suarez |
| Lima Bikes · Taller | Lima Bikes | Taller | `OC-LBT` | Millennium Mobility S.A.C. · RUC 20512345671 | Jefe de taller | Alejandro Suarez |
| Lizzo · Vehículos | Lizzo | Vehículos | `OC-LZV` | Millennium Mobility S.A.C. · RUC 20512345671 | Henry Saldaña, Gerente | Alejandro Suarez |

Las tres toman el emisor de la misma empresa y difieren en lo demás. Cambiar el emisor de una no afecta a las otras.

**Reglas de negocio**

- Los campos son los mismos en todas las plantillas; lo que cambia son sus valores. Una línea de negocio o un tipo de orden nuevos no requieren tocar el programa: se crea otra plantilla.
- Al crear una plantilla, los datos del emisor —razón social, documento, teléfono y domicilio fiscal— se copian de la empresa dueña y **se pueden editar**. Es el mismo criterio que la tasa de impuesto: `countries` la propone y la que manda es la guardada en el comprobante. Editar la empresa después no cambia las plantillas existentes.
- La serie no se repite dentro de la empresa. Se guarda normalizada —mayúsculas, sin espacios— y se puede corregir mientras la plantilla no tenga órdenes; después queda fija, como el código de un catálogo (§2.4).
- La línea de negocio, si se indica, tiene que ser una marca de la misma empresa: el sistema lo impide, igual que en las asignaciones de rol (§5.7).
- Puede haber más de una plantilla activa para la misma línea y tipo —dos talleres con contactos distintos, por ejemplo—: quien emite la orden elige cuál usar.
- Los correos se guardan sin espacios y en minúsculas, y el documento del emisor sin guiones, espacios ni puntos (§2.2).
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

Quedan tres decisiones abiertas, cada una con su momento fijado. Las longitudes y el dígito de control de los documentos tributarios de Colombia y México (§4.7, D-75), antes de registrar la primera empresa de cada país (D-52). Cómo se conserva el período anterior de un colaborador que regresa a la misma empresa (§5.3, §5.13), antes de cargar los colaboradores reales: desde entonces, cada regreso borraría un período que ya no se recupera. Y cómo lista el registro público, sin sesión, las sedes y los tipos de documento (convenciones de la API, §14.16), antes de encenderlo en un entorno (§5.13, D-83). La matriz de permisos por rol quedó fijada el 19/10/2026 (§5.6, D-95 a D-105), y el 20/10/2026 sumó Seguridad, con sus doce permisos, sus categorías y sus pares incompatibles (D-117 a D-124, D-126); una de sus celdas espera al negocio: quién registra la entrega al cliente (`stock.entregar`; procesos y pantallas, pregunta 4). Lo que falta para construir son datos: los datos maestros reales —razón social y documento tributario de la empresa, sedes con su código de establecimiento, marcas, proveedores, colaboradores—. La lista oficial de divisiones de Perú está cargada; los distritos que crean las leyes después entran cuando tienen código oficial (§4.5, D-77).

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
| D-33 | Las sesiones se guardan en el servidor y el navegador solo recibe una cookie `HttpOnly`. Se cierran por 2 horas de inactividad (30 minutos hasta D-127), a las 8 horas en cualquier caso, al cerrar sesión, al cambiar la contraseña (las demás) y al bloquear la cuenta (§5.12) |
| D-34 | El primer superadministrador lo crea la aplicación al arrancar, desde variables de entorno y con contraseña provisional; no la migración (§5.2) |
| D-35 | Las órdenes de compra se emiten con plantillas. Todas tienen los mismos campos; sus valores cambian según la línea de negocio y el tipo de orden (vehículos, taller, otros), que es un catálogo administrable. Los datos del emisor se proponen desde la empresa y se pueden editar; los contactos de atención y soporte son texto (§9.2) |
| D-36 | Cada plantilla tiene su serie y cada serie su correlativo, que se reinicia cada año: `OC-LBT-2026-0001`. El número se asigna al crear la orden, con el año en la hora local del país de la empresa, y no se reutiliza (§9.2) |
| D-37 | Las líneas de una orden de compra admiten conceptos que no son vehículos, igual que las de factura: las órdenes de taller no compran modelos (§9.2) |
| D-38 | Toda cuenta pertenece a un colaborador y nace vinculada a él: un administrador da acceso a un colaborador ya registrado —vinculándolo a su cuenta si la persona ya tiene una por otra empresa—, o el registro público crea el colaborador y la cuenta juntos (D-32). Para tener un rol en una empresa, la cuenta debe estar vinculada a un colaborador activo de esa empresa; al registrar su salida se retiran esos roles, y si no le queda ningún colaborador activo la cuenta se bloquea. Excepciones: `SYSTEM`, que no es una persona, y el primer superadministrador, que hasta vincularse a su perfil configura el sistema pero no puede dar acceso. Una cuenta también queda sin colaborador cuando se la desvincula del único que tenía, hasta que un superadministrador la vincula de nuevo (D-82) (§5.2, §5.3) |
| D-39 | Vocabulario: **rol** es el conjunto de permisos de una cuenta; **puesto**, el trabajo del colaborador, informativo; **perfil**, sus datos personales. El documento no usa "cargo" (§5.1) |
| D-40 | Fechas: los momentos son `TIMESTAMPTZ` en UTC (`_at`), las fechas de calendario `DATE` (`_date`) y los períodos rangos con fin excluido. Todo intercambio usa ISO 8601, con los momentos en UTC; en una hoja de cálculo exportada, los momentos van en una sola zona nombrada en el encabezado (D-49); "hoy" se calcula en la zona de la sede, o del país de la empresa, o `America/Lima`; las zonas son nombres IANA de una región, como `America/Lima`, nunca un desfase fijo ni un alias sin continente; `dd/mm/aaaa` solo en pantalla (§2.6) |
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
| D-52 | Un paso se cierra con el alcance de los países que operan. Hoy opera solo Perú (D-5), así que el bloque 1 se cerró con su ubigeo cargado. Lo que es propio de otro país —su lista oficial de divisiones (D-2) y las longitudes y el cálculo del dígito de control de sus documentos tributarios (§4.7, D-75)— se resuelve antes de registrar su primera empresa |
| D-53 | La división de una dirección es obligatoria cuando el país de la dirección tiene sus divisiones cargadas; solo puede faltar en un país sin ellas, como la dirección de un proveedor extranjero (§4.6) |
| D-54 | `geo_divisions.parent_level` es una columna que calcula la base (el escalón menos uno) y no guarda un dato del negocio: existe solo para que la llave hacia la división superior incluya su escalón (§4.5). Es `STORED` y no `VIRTUAL` porque PostgreSQL 18 no admite una columna virtual en una llave foránea; como la calcula la base, no se desincroniza. Es una de las tres excepciones con nombre a la regla de no guardar lo derivable, junto con el estado actual y el último kilometraje del vehículo (§2.2, §8.4, §11.1), y la única que no es un dato del negocio. La aplicación nunca la escribe |
| D-55 | Existe un administrador de accesos para todo el grupo: el rol `ADMIN_ACCESOS`, del sistema y de alcance sistema, que da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas en todas las empresas, y por eso las ve todas. Pueden tenerlo varias personas. Tiene tres límites: nunca actúa sobre su propia cuenta (D-70); nunca otorga ni retira `SUPERADMIN`, `ADMIN_ACCESOS` ni `ADMIN_EMPRESA`, que otorga y retira solo un superadministrador; y nunca actúa sobre una cuenta protegida —la que tiene `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente—, que solo administra un superadministrador. No se protege al último administrador de accesos, porque el superadministrador puede hacer todo lo que él hace. Enmendada por D-117 y D-121: lo que hace se lo dan sus siete permisos `seguridad.*`, con alcance sistema (D-122); los roles que no otorga ni retira son los reservados por lo que incluyen, entre ellos esos tres, y una cuenta protegida es la que tiene uno de ellos vigente (§5.5, §5.7, §5.9) |
| D-56 | El administrador de accesos asigna roles cuyos permisos no tiene, dentro de los límites de D-55. Es la única excepción a la regla 2 de §5.5; extenderla a otro rol requiere una decisión nueva. Enmendada por D-120: la excepción es el permiso `seguridad.asignar_roles_sin_tenerlos`, solo de `ADMIN_ACCESOS`, que no se marca en un rol propio (§5.5) |
| D-57 | Separación de funciones. El administrador de una empresa registra y modifica sus colaboradores y registra sus salidas y regresos; ve las cuentas de su empresa sin modificarlas, y no da acceso, no otorga ni retira roles, no bloquea cuentas ni restablece contraseñas. El administrador de accesos hace eso y no registra colaboradores, salidas ni regresos, para que nadie pueda inventar un colaborador y darle acceso. El superadministrador hace ambas cosas, además de las empresas y los catálogos. Todos actúan solo mientras su cuenta está vinculada a un colaborador activo —salvo el superadministrador sin vincular, que registra y modifica colaboradores para crear el suyo (D-89)—, y ninguno actúa sobre lo suyo, como fija D-70. Enmendada por D-117, D-119 y D-122: cada función es un permiso `seguridad.*`, que la semilla reparte entre los tres roles como dice esta decisión; la separación la sostienen los pares incompatibles, en los roles propios y en las cuentas; y la excepción del superadministrador sin vincular es la de D-122 (§2.5, §5.2, §5.3, §5.4, §5.7) |
| D-58 | Dar acceso crea la cuenta del colaborador, con una contraseña provisional que genera el sistema y se muestra una sola vez, o la vincula a la cuenta que la persona ya tiene por otra empresa del grupo; nunca cambia los roles ni la contraseña de una cuenta existente. Lo hacen el superadministrador y el administrador de accesos, este en cualquier empresa salvo con una cuenta protegida (D-55). La única cuenta que se vincula a sí misma es la del primer superadministrador, mientras no tiene un colaborador activo (D-70). Vincular una cuenta cerrada por una salida cierra su `baja` (§5.3, §5.4) |
| D-59 | `ADMIN_EMPRESA` nunca se limita a una marca: administrar una empresa alcanza a la empresa entera. Lo valida la aplicación, porque la base no ve el rol en la fila de la asignación. Los demás roles de empresa y de sede sí pueden limitarse a una marca. Enmendada por D-121: ningún rol reservado se limita a una marca, y `ADMIN_EMPRESA` es uno de ellos (§5.7) |
| D-60 | La fecha de salida de un colaborador no puede ser posterior a hoy, calculado en la zona de su sede o, sin sede, en la del país de su empresa (§2.6): la salida se registra cuando ocurre. La salida siempre quita el acceso, también el de una cuenta protegida: retira los roles de esa empresa (`baja_colaborador`) y, si era el último colaborador activo, cierra la cuenta desde ese momento —`baja`, salvo que ya haya una en curso; `baja_cuenta`; sesiones cerradas—, termina las vacaciones o la licencia en curso, cancela los bloqueos programados de la línea laboral y no toca los bloqueos por seguridad. Su único freno es la protección del último superadministrador (§5.9). La fecha del regreso no es anterior a la de salida ni posterior a hoy, calculado igual. El regreso cierra solo la `baja`, y los roles no vuelven solos (§5.3, §5.4, §5.9) |
| D-61 | Un bloqueo de cuenta empieza ahora o en el futuro, nunca en el pasado. Se cierra siempre en el momento en que alguien lo cierra, sin fecha elegida: uno en curso termina ahí; uno programado queda cancelado, conserva su período, registra quién y cuándo, y deja de contar. Uno terminado, levantado o cancelado no se vuelve a cerrar (§5.4) |
| D-62 | Los bloqueos de cuenta corren en dos líneas de tiempo: la laboral —`vacaciones`, `licencia`, `baja`— y la de `seguridad`. Dentro de una línea no se cruzan, incluso en el pasado; entre líneas sí. Basta un bloqueo en curso de cualquier línea para que la cuenta no entre. La salida y el regreso no tocan la línea de seguridad (§5.4) |
| D-63 | Un superadministrador o el administrador de accesos puede restablecer la contraseña de otra cuenta: el sistema genera una provisional, la muestra una sola vez, obliga a cambiarla y cierra todas las sesiones de esa cuenta. Nadie restablece así la suya, la de `SYSTEM` no se restablece, y el administrador de accesos no la restablece a una cuenta protegida (§5.4, §5.12) |
| D-64 | Un proceso automático cierra las asignaciones vencidas con su caducidad como fecha de retiro y motivo `caducidad`. El acceso deja de valer exactamente al caducar, sin esperar al proceso (§5.8) |
| D-65 | La protección del último superadministrador mira hacia adelante: una operación que le quita o le quitará el rol a una cuenta —retirarlo, bloquearla ahora o con un bloqueo programado, registrar la salida que la cierra o desvincularla de su último colaborador activo (D-82)— procede solo si otra cuenta queda como superadministradora duradera: con el rol vigente y sin caducidad, y sin bloqueos en curso ni programados. Esas operaciones se hacen de a una, para que dos solicitudes simultáneas no dejen al sistema sin ninguno (§5.9) |
| D-66 | La frontera entre empresas la defiende también la base: Row Level Security, habilitada y forzada en toda tabla con `company_id` y en `companies`. La aplicación usa una cuenta de base que no es dueña de nada y fija el alcance al comenzar cada transacción; sin alcance no se ve nada. Los roles de alcance sistema, el arranque y los procesos automáticos actúan con alcance `SYSTEM` explícito (§2.5) |
| D-67 | Los bloqueos de cuenta impiden el cruce con una exclusión `EXCLUDE USING gist`, no con `WITHOUT OVERLAPS`: un bloqueo cancelado no cuenta, y la línea de tiempo que se compara se deduce del motivo y no se guarda. Es una excepción con nombre a la regla de §2.2 (§2.2, §5.4) |
| D-68 | La matriz de permisos llega antes del paso 5, el primero con operaciones de negocio, y acota también lo que lee un rol de alcance sistema. Hasta entonces, el superadministrador y el administrador de accesos leen los datos de todas las empresas, que hoy son empresas, marcas, sedes, ubicaciones y colaboradores (§3, §5.13) |
| D-69 | La `baja` es de la salida: la abre solo el sistema, al registrar la salida del último colaborador activo de la cuenta o al desvincular la cuenta de él, que funciona como una salida (D-82), y la cierran solo el regreso o un nuevo acceso que vincula la cuenta. Nadie la abre ni la cierra a mano: los bloqueos que se registran a mano son por vacaciones, licencia o seguridad. Así una cuenta nunca queda dada de baja mientras su colaborador sigue activo (§5.4, §5.9) |
| D-70 | Nadie actúa sobre lo suyo. Ninguna cuenta, tampoco la de un superadministrador, se otorga ni se retira roles, se bloquea, levanta sus bloqueos ni se restablece la contraseña por las operaciones de administración: cambiar la propia contraseña tiene su operación. Nadie registra su propia salida ni su propio regreso. La única cuenta que se vincula a sí misma es la del primer superadministrador, mientras no tiene un colaborador activo. Enmendada por D-122: tampoco se cambian los permisos de un rol que quien edita tiene vigente (§5.2, §5.3, §5.4, §5.5, §5.7) |
| D-71 | Separación estática de funciones: una cuenta nunca tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` vigentes a la vez, para que quien registra los colaboradores de una empresa no sea también quien les da acceso. Lo valida la aplicación al otorgar cualquiera de los dos, porque la base no ve dos asignaciones a la vez. Reemplazada por D-119: la regla son los pares de permisos incompatibles, que siguen impidiendo juntar esos dos roles (§5.5, §5.7) |
| D-72 | Un rol privilegiado —`SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA`, los que hacen protegida una cuenta— se otorga solo a una cuenta que ya cambió su contraseña provisional, para que quien la entregó nunca termine con una cuenta privilegiada en sus manos. El primer superadministrador no se otorga así: lo crea el arranque. Enmendada por D-121: la regla vale para todo rol reservado por lo que incluye, que es el que hace protegida una cuenta (§5.2, §5.4, §5.7) |
| D-73 | Los cinco catálogos del bloque 4 —`vehicle_types`, `vehicle_brands`, `vehicle_models`, `colors` y `equipment_types`— registran fechas pero no autor, como los del bloque 1: son catálogos generales (D-42). La aplicabilidad de un equipo a un tipo de vehículo se retira desactivándola, no borrándola, y se reactiva al volver a marcarla. La última aplicabilidad activa no se retira: un equipo que deja de aplicar a todo se desactiva (§2.5, §7.5) |
| D-74 | Un código que lee una regla del programa cuenta como algo que lo usa (D-48): queda fijo desde la semilla que lo crea. En `equipment_types` son `GPS` y `PANTALLA`, porque las reglas de habilitación reconocen el GPS y la pantalla por su código (§7.5, §11.9). El código de cualquier otro equipo se corrige mientras ninguna instalación lo use, y desde la primera queda fijo. La API rechaza la corrección y responde si el código todavía se puede corregir, para que la pantalla bloquee el campo (§2.4) |
| D-75 | El dígito de control lo calcula `location` según la sigla del tipo dentro de su país y lo ofrece por su fachada. Hoy existe un solo cálculo: el del RUC de Perú, con el módulo 11 de SUNAT (pesos 5, 4, 3, 2, 7, 6, 5, 4, 3, 2). Un tipo con dígito de control cuyo cálculo todavía no existe —el NIT, el RFC y la CURP— acepta sus números sin verificar hasta que llegue su cálculo, que se resuelve con sus longitudes antes de la primera empresa de cada país (D-52). Lo aplica quien guarda el número: `organization` en empresas y colaboradores, y `procurement`, cuando se construya, en proveedores y en el emisor de las plantillas. Se verifica al crear el registro y cuando cambian el número o el tipo; los números ya guardados siguen siendo válidos (§4.7, §6.2) |
| D-76 | El DNI de Perú no lleva dígito de control: sus 8 dígitos no traen uno dentro. El carácter de verificación que va impreso en el documento es aparte y no se guarda (§4.7) |
| D-77 | La lista oficial de divisiones de Perú es la edición 2022 del INEI más los distritos creados por ley después, revisada al 17/10/2026. Cada distrito nuevo entra activo con su propia migración en cuanto el INEI o SUNAT publican su código, y el distrito del que se separó sigue activo; hasta entonces no se puede elegir. El primero es Santa Rosa de Loreto (Ley 32403, 03/07/2025), que todavía no tiene código publicado. La lista se revisa en el cierre de cada paso, contra las leyes nuevas y las ediciones nuevas del INEI (§4.5, D-2) |
| D-78 | La longitud de un tipo de documento tributario, cuando la declara, va de 1 a 20: lo que caben las columnas que guardan los números (`companies.tax_id`, `employees.document_number`, `suppliers.tax_id` y `purchase_order_templates.issuer_tax_id`). Una longitud mayor declararía un tipo que ningún número puede cumplir (§4.7) |
| D-79 | Las ubicaciones siguen D-8 como las empresas y las sedes: corregir modifica la dirección y mudarse crea una nueva. La dirección de una ubicación cambia solo por su propia operación, como la de una sede, y al cambiarla la pantalla pregunta cuál de las dos es. Vale para las ubicaciones de terceros; una propia usa la dirección de su sede. Una ubicación no pasa de propia a terceros ni al revés: es otro lugar, así que la anterior se desactiva y se crea una nueva (§4.6, §6.5) |
| D-80 | La tabla de direcciones queda acotada por empresa antes de guardar la primera dirección de un proveedor (paso 7), no recién con los clientes: desde ahí guarda direcciones de terceros de cada empresa, y la frontera entre empresas la defiende también la base (D-66). Cómo se acota se diseña con ese paso (§2.5) |
| D-81 | `roles.company_id`, los roles propios de una empresa, sale del paso 2 y llega con la matriz de permisos (D-68), antes del paso 5: un rol propio de una empresa solo tiene sentido con permisos que marcarle. Con la matriz se deciden también quién crea esos roles y la unicidad del código por empresa. Hasta entonces todos los roles son generales (§3, §5.5, §5.13) |
| D-82 | Dar acceso vincula un colaborador a una cuenta que ya existe cuando esa cuenta pertenece a un colaborador de otra empresa del grupo con la misma sigla de documento y el mismo número. Se compara la sigla del tipo y no el tipo, porque los tipos son de cada país y cada empresa es de un país (D-5). Si no coinciden, solo un superadministrador vincula la cuenta, después de ver el nombre, el documento y la empresa del colaborador que ya la tiene y de confirmar que es la misma persona; la confirmación queda registrada con quién la dio, cuándo y qué colaborador tenía la cuenta. Una cuenta que existe sin ningún colaborador también la vincula solo un superadministrador, que ve que no tiene titulares y confirma. La confirmación vale para los titulares que vio: si cambiaron antes de confirmar, no se vincula y se le muestran los de ahora para que confirme de nuevo. A cualquier otro se le rechaza. La excepción es el primer superadministrador, que se vincula a sí mismo sin colaborador con el cual comparar (D-70). Un vínculo equivocado lo corrige solo un superadministrador con la desvinculación, que funciona como una salida: retira los roles de la cuenta en esa empresa con motivo `retiro` y, si no le queda ningún colaborador activo, la cierra como D-60, con la protección del último superadministrador (D-65). Cada vínculo confirmado y cada desvinculación dejan una fila en `employee_account_link_events`, escrita en la misma operación que el cambio y que nunca se edita ni se borra; un vínculo confirmado guarda además los titulares que confirmó el superadministrador, una lista vacía si la cuenta no tenía. Esas filas las leen solo el superadministrador y los procesos automáticos, porque guardan datos de personas de otras empresas. Enmendada por D-117 y D-122, solo en el lenguaje: confirmar el vínculo y desvincular son los permisos `seguridad.confirmar_vinculo` y `seguridad.desvincular_cuentas`, que tiene solo el superadministrador y no se marcan en un rol propio; quién lee esas filas no cambia (§2.5, §5.3, §5.4, §5.7, §5.9, D-38, D-58, D-69) |
| D-83 | El registro público con perfil (D-32) queda fuera del alcance del paso 3. Sigue apagado, y se construye antes de encender el registro público en cualquier entorno (§3, §5.2, §5.13) |
| D-84 | A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo: ni sedes, marcas, ubicaciones o colaboradores bajo una empresa inactiva, ni colaboradores o ubicaciones propias en una sede inactiva, ni asignaciones de rol sobre una empresa, una sede o una marca inactivas. Lo que ya apuntaba a ella sigue siendo válido y se sigue modificando mientras ese vínculo no cambie, el mismo criterio que con las monedas y los tipos de documento desactivados (§4.1, §4.7, §5.3, §5.7, §6.2 a §6.5) |
| D-85 | El superadministrador corrige el tipo y el número del documento tributario de una empresa, y la corrección se verifica de nuevo: un tipo del país de la empresa que aplique a empresas, la longitud, el dígito de control y que el número no se repita en el país. El país de una empresa no cambia (§6.2) |
| D-86 | La normalización `TRIM` + `UPPER` sin guiones, espacios ni puntos es de los identificadores que fija una norma externa: documentos tributarios y de identidad, placas, códigos de ubigeo. Los códigos propios, como el de una sede o el de una ubicación, se normalizan solo con `TRIM` + `UPPER` y conservan sus guiones (§2.2) |
| D-87 | La caducidad de una asignación de rol es un momento. Cuando la pantalla elige un día, el acceso caduca al empezar el día siguiente en la zona de la empresa, como el fin excluido de los períodos `[inicio, fin)` de D-40 (§5.7) |
| D-88 | La fecha de ingreso de un colaborador no puede ser posterior a hoy, calculado en la zona de su sede o, sin sede, en la del país de su empresa, igual que la salida y el regreso (D-60). Vale al registrarlo y al modificarlo (§5.3) |
| D-89 | El superadministrador sin colaborador vinculado registra y modifica colaboradores, porque tiene que crear el suyo, pero no registra salidas ni regresos hasta vincularse, como no da acceso ni otorga roles. Así matiza D-57, que pide la cuenta vinculada a un colaborador activo para administrar colaboradores (§5.2, §5.3, §5.9) |
| D-90 | Una sede tiene a lo sumo una ubicación propia (`sede_propia`), activa o inactiva, para que "¿dónde está?" tenga una sola respuesta para lo que está en ella. Si la de una sede se desactivó, se reactiva en vez de crear otra, y conserva el historial de sus vehículos. No se reactiva mientras su sede siga inactiva: primero se reactiva la sede y después su ubicación, como D-84 (§6.5) |
| D-91 | La longitud de un número de documento, como su dígito de control (D-75), se verifica al crear el registro y cuando cambian el número o el tipo. Un número que no cambia no se vuelve a medir al editar el registro, aunque la longitud del tipo se haya corregido después (§4.7, §6.2) |
| D-92 | El colaborador vive en `organization`, y pasa a un contexto propio cuando lleguen los contratos laborales, la planilla o la asistencia. El bloque 3b se reparte así: `employees` es de `organization`; `user_roles` es de `iam`, igual que `user_access_suspensions` y el resto del bloque 2 (§3) |
| D-93 | Leen los colaboradores de una empresa —la lista, cada uno y su exportación— solo una cuenta de alcance global, hoy el superadministrador y el administrador de accesos, y una cuenta que administra esa empresa. Un rol de empresa o de sede que no la administra no los lee: el documento, el teléfono y la foto son datos personales. La matriz de permisos (D-68) decide después qué ve cada rol (§5.3, §5.13) |
| D-94 | Reactivar es volver a colgar (D-84): una sede, una marca o una ubicación se reactivan solo con su empresa activa, y un colaborador regresa solo a una empresa activa y, si tiene sede, a una sede activa, porque el regreso le devuelve la cuenta. Primero se reactiva la empresa y después lo que cuelga de ella, como D-90 con la sede. Desactivar no espera a nada (§5.3, §6.2 a §6.5) |
| D-95 | La matriz de permisos por rol queda fijada en §5.6: cincuenta permisos para los pasos 1 a 11, en `<módulo>.<acción>`, y lo que incluye cada uno de los siete roles del sistema. Las operaciones de negocio y las lecturas de datos exigen un permiso; las cuentas, los accesos, los colaboradores y los roles van por rol (§5.8 regla 7). El ejemplo `seguridad.crear_usuario` no existe: dar acceso lo fija D-57 por rol. Cada paso trae sus permisos con una migración de `iam`; ningún otro contexto escribe `permissions` ni `role_permissions`. Enmendada por D-117: las cuentas, los accesos, los colaboradores y los roles también van por permiso, los doce de `seguridad`, y el catálogo pasa a sesenta y dos (§5.6, §5.8, §5.13) |
| D-96 | `role_permissions` conserva la historia: cada permiso que un rol incluye lleva quién lo marcó y cuándo, y quién lo desmarcó y cuándo; nada se borra, y un rol incluye un permiso a lo sumo una vez mientras está vigente (§5.6) |
| D-97 | Los roles del sistema son inmutables también en su matriz: sus permisos los fijan las migraciones y la pantalla los muestra en solo lectura, también al superadministrador; para variar se crea un rol propio. `roles.is_active` existe: un rol propio inactivo no se otorga y sus asignaciones vigentes siguen válidas; los roles del sistema no se desactivan. Enmendada por D-124: el rol propio puede nacer como copia exacta de uno del sistema, salvo de `SUPERADMIN` y `ADMIN_ACCESOS` (§5.5) |
| D-98 | Los roles propios de una empresa los crea solo el superadministrador, eligiendo la empresa al crearlos; el administrador de empresa y el de accesos los leen. El código es único dentro de cada empresa y entre los generales, nunca el de un rol del sistema; un rol de una empresa no se asigna en otra ni es de alcance sistema. Cierra la decisión que D-81 dejó abierta. Enmendada por D-122 y D-123: los crea quien tiene `seguridad.gestionar_roles` con alcance sistema —hoy solo el superadministrador— y los lee quien tiene `seguridad.ver_roles`, donde actúa (§5.5) |
| D-99 | El permiso dice qué y la asignación dice dónde, también al leer: una lectura de recursos de empresa se filtra por el permiso y el alcance de la asignación que lo trae; la frontera entre empresas de D-66 queda como piso. Un rol de sede lee toda su empresa (D-14) y actúa solo en su sede (§5.8 regla 6) |
| D-100 | Cada camino de estado nombra el permiso que exige (`status_change_reasons.requires_permission_id`, candidato de §8.6 adoptado); un camino sin permiso propio exige `stock.cambiar_estado`. Quién recorre cada camino es un dato, como los caminos (D-19), y lo mantiene solo el superadministrador con `catalogos.gestionar_estados`. Con los caminos de ejemplo de §8.3: habilitar, vender, entregar, atender en taller y recuperar (§5.6, §8.3) |
| D-101 | Un rol de sede actúa sobre las unidades que están en la ubicación propia de su sede; las que están en una ubicación de terceros, o sin ubicación, las tocan los roles de empresa. Si un caso real pide que una sede responda por un patio arrendado o una tienda de dealer, se decide un dato nuevo, distinto de `locations.branch_id`, que sigue siendo solo de las propias (§2.5, §6.5) |
| D-102 | La marca de una asignación acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, aplicada en el código y no en la base; un rol de empresa sin marca ve todas. Los vehículos son el primer dato con marca, en el paso 5 (§2.5, §5.7, §5.13) |
| D-103 | El tipo de cambio lo corrige a mano el superadministrador, con `catalogos.corregir_tipo_cambio`: `exchange_rates` no tiene empresa y una tasa sirve a todo el sistema, así que un rol de empresa no podría acotarse. Si el negocio quiere que la contabilidad lo corrija, será un rol general nuevo o una regla de acotación nueva (§4.8) |
| D-104 | Una unidad se anula por las dos vías que describe el diccionario: como consecuencia de una nota de crédito que deja su valor en cero, gobernada por `compras.registrar_nota_credito`, o por acción explícita, que exige `stock.anular_vehiculo` y el candado de §9.6 (§8.1, §9.6) |
| D-105 | Lo que cada rol ve de los colaboradores, que D-93 dejó a la matriz: la fila sin sus datos personales —nombre, puesto, sede, fechas de ingreso y salida, si está activo y si tiene cuenta— la lee cualquier rol con `configuracion.ver_colaboradores`, que la semilla da a los siete, para los selectores y los nombres de autor; el documento, el teléfono, la foto y la exportación, solo con `configuracion.ver_datos_personales_colaboradores`, que tienen los dos roles globales y el administrador de la empresa. El campo que no se puede ver no viaja (§2.5, §5.3) |
| D-106 | Todo documento que sale del sistema —una hoja de cálculo, un PDF u otro formato— lo genera el backend, y el frontend lo descarga. Un pedido igual —el mismo tipo, formato, criterios, alcance efectivo, idioma, zona y versión de los datos— reutiliza el documento ya generado, o espera al que se está generando: nunca se genera dos veces. El estado de cada documento vive en PostgreSQL (`generated_documents`); sus archivos, en la base o en DigitalOcean Spaces, nunca en la memoria ni en el disco del proceso, porque en App Platform el disco es efímero y en cada despliegue conviven la instancia vieja y la nueva (§2.7; convenciones de la API, §8) |
| D-107 | El tipo de vehículo de un modelo se corrige mientras el modelo no tenga unidades; desde la primera queda fijo, el mismo principio que D-48 y D-74. `fleet` se lo avisa a `vehiclecatalog` por su fachada al registrar la primera unidad del modelo, y `vehiclecatalog` guarda el hecho en `vehicle_models.has_units`: no lee las tablas de `fleet` (§7.3) |
| D-108 | No se crea un modelo de una marca inactiva ni de un tipo de vehículo inactivo, y no se registra una unidad cuyo modelo, marca o tipo estén inactivos. Lo retirado que un registro ya tiene se conserva al editarlo, como D-84 (§7.1 a §7.3, §8.1) |
| D-109 | El combustible de un modelo nuevo no se propone: nace vacío y se elige, y queda vacío cuando no se conoce (§7.3) |
| D-110 | El regreso automático escribe su fila del historial con un motivo del camino que recorre: el motivo automático reservado del camino que va del destino al estado configurado (`status_change_reasons.is_automatic`, código `REGRESO_AUTOMATICO`). Lo crea el sistema, no se ofrece a las personas y no se edita. Reemplaza la propuesta 6 del documento del contexto `fleet` (sección 6 b), que firmaba el regreso con el motivo del camino que venció (§8.3, §8.4) |
| D-111 | `vehicle_status_history` y `vehicle_mileage_readings` llevan su `company_id`, obligatorio, aunque se pueda deducir de la unidad: se consultan por sí solas y acumulan volumen (§2.5). Su Row Level Security lo usa directamente, sin pasar por `vehicles`. La empresa de una unidad no cambia, así que la de sus filas nunca discrepa con la suya (§8.4, §8.5) |
| D-112 | Marcas y modelos es una lista de marcas más la ficha de cada marca, con sus modelos dentro de la ficha, como Empresas (DESIGN.md, revisión 13); no es un maestro-detalle (procesos y pantallas, apartado 3.2) |
| D-113 | `/vehicle-statuses` y `/status-change-reasons` son listas cortas sin página: la tercera excepción a la regla de buscar y paginar en el servidor (convenciones de la API, §6.2). El criterio: un catálogo corto del sistema que se necesita entero para nombrar las referencias. Los caminos se filtran en el servidor con `?fromStatusId={id}` y `?origin=registration` (convenciones §14.23, adoptado) |
| D-114 | Los códigos de error de un agregado llevan un solo prefijo, sin guion bajo interno: `VEHICLESTATUS_*` y `STATUSCHANGEREASON_*`, los que ya produce la fábrica de `*_NOT_FOUND`. Así cada agregado tiene un prefijo y no dos (convenciones de la API, §7.2) |
| D-115 | `allowedActions` de la unidad y `vehicleTypeLocked` del modelo son datos derivados del recurso, que calcula su dueño y nunca se guardan, como `isoCodesLocked` del país y `codeLocked` del tipo de equipamiento (convenciones de la API, §4.2). El kilometraje de la unidad (`currentMileage`) no es uno de ellos: es una de las dos excepciones que se guardan además en la unidad (§2.2, §11.1) |
| D-116 | Todo catálogo y toda lista principal se exporta a una hoja de cálculo desde su propio endpoint: también los cinco catálogos de flota, los modelos de una marca y el inventario de unidades. El exportable de stock del paso 11 es otro documento (D-106; convenciones de la API, §8) |
| D-117 | Seguridad se autoriza por permisos, como el resto del sistema: doce permisos del módulo `seguridad` reemplazan a los roles que exigían las cuentas, los accesos, los colaboradores y los roles —las 18 anotaciones de rol de `iam` y `organization`—. Son `ver_cuentas`, `ver_roles`, `dar_acceso`, `gestionar_colaboradores`, `gestionar_roles`, `bloquear_cuentas`, `registrar_salidas_y_regresos`, `asignar_roles`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos`. La semilla da once al superadministrador —todos salvo `asignar_roles_sin_tenerlos`—; siete al administrador de accesos —`ver_cuentas`, `ver_roles`, `dar_acceso`, `bloquear_cuentas`, `asignar_roles`, `restablecer_contrasenas` y `asignar_roles_sin_tenerlos`—; y cuatro al administrador de empresa —`ver_cuentas`, `ver_roles`, `gestionar_colaboradores` y `registrar_salidas_y_regresos`—: 22 marcas, y las marcas vigentes de los roles del sistema pasan de 76 a 98. `confirmar_vinculo` y `desvincular_cuentas` son solo del superadministrador, y `asignar_roles_sin_tenerlos` solo del administrador de accesos: ninguno de los tres se marca en un rol propio. Los límites sobre el destino —la propia cuenta, una cuenta protegida, `SYSTEM`, el último superadministrador— quedan fijos en el código. No se siembran permisos de pasos que todavía no existen. Enmienda D-95 y la regla 7 de §5.8 (§5.6, §5.8) |
| D-118 | Cada permiso lleva su categoría (`permissions.category`), que ordena la matriz por lo que hace: `ver`, `crear`, `editar`, `gestionar` —crear, editar y desactivar un recurso; un `gestionar_*` no se parte—, `desactivar_anular` o `acciones_especiales`. No hay "eliminar", porque nada se borra. Los 25 permisos sembrados hasta el paso 5 quedan 6 de ver, 2 de crear, 1 de editar, 6 de gestionar y 10 de acciones especiales, y los doce de seguridad llevan la suya (D-117). La categoría es obligatoria y la trae cada permiso en la migración de su paso (§5.6) |
| D-119 | La separación de funciones se escribe como pares de permisos incompatibles, en `permission_incompatibilities`, y reemplaza a D-71. Son catorce pares: cada uno de `seguridad.gestionar_colaboradores` y `seguridad.registrar_salidas_y_regresos` con cada uno de `dar_acceso`, `asignar_roles`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos`, para que quien registra a los colaboradores no sea también quien les da acceso. Un rol propio nunca incluye las dos partes de un par. Una cuenta nunca las junta entre sus roles vigentes, en ninguna empresa; las asignaciones de `SUPERADMIN` quedan fuera de esa suma, porque el superadministrador hace las dos cosas (D-57). Se revisa al guardar un rol —también cuando editarlo haría que un titular junte un par, que se rechaza— y al asignarlo. Para que dos operaciones simultáneas no lo burlen, guardar un rol, asignar uno, restablecer una contraseña y otorgar un acceso toman un mismo candado de transacción, y guardar un rol bloquea además cada cuenta titular, en orden de identificador (§5.5 a §5.7) |
| D-120 | La excepción a la regla 2 de §5.5 que D-56 da al administrador de accesos es un permiso propio, `seguridad.asignar_roles_sin_tenerlos`. Lo tiene solo `ADMIN_ACCESOS` y no se marca en un rol propio, así que extenderla a otro rol sigue exigiendo una decisión nueva. Sus límites siguen siendo los de D-55, sobre los roles reservados de D-121 (§5.5, §5.6) |
| D-121 | Un rol es reservado por lo que incluye: lo es si incluye un permiso privilegiado, un `seguridad.*` de una categoría que no es `ver` (D-118). Solo un superadministrador lo otorga y lo retira, y una cuenta que tiene uno vigente es protegida: solo la maneja un superadministrador. `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, que eran reservados por su código, lo siguen siendo por lo que incluyen, y un rol propio con un permiso privilegiado también lo es. "Rol reservado" reemplaza a los tres códigos en D-59 y D-72. El superadministrador otorga cualquier rol; la regla 2 de §5.5 se aplica a los no reservados. Agregar o quitar un permiso privilegiado en un rol con titulares vigentes es solo del superadministrador. Volver reservado un rol se rechaza (`ROLE_RESERVED_HOLDERS_INVALID`) mientras alguno de sus titulares tenga una asignación acotada a una marca, una contraseña provisional o una asignación que no otorgó un superadministrador: un rol reservado no se limita a una marca, va solo a una cuenta con contraseña propia y lo otorga solo él (§5.5, §5.7) |
| D-122 | Dónde actúan los permisos de seguridad. Los ocho de accesos y roles —`dar_acceso`, `asignar_roles`, `asignar_roles_sin_tenerlos`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `gestionar_roles`— solo cuentan desde asignaciones de alcance sistema, y solo se marcan en roles de alcance `SYSTEM`, que por eso son generales; guardarlos o copiarlos en otro rol se rechaza (`ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`). `gestionar_colaboradores` y `registrar_salidas_y_regresos` actúan en la empresa o la sede de su asignación: un colaborador sin sede exige actuar en toda la empresa, y un cambio de sede, en la de origen y en la de destino. `ver_cuentas` con alcance sistema lee todas las cuentas, y con una empresa, las de esa empresa, también al exportar. `ver_roles` lee los roles del sistema y los generales con cualquier asignación que lo traiga, y los de una empresa donde actúa. Las reglas 1 y 2 de §5.5 comparan, para un rol general, con lo que quien edita tiene por asignaciones de alcance sistema, y para un rol de la empresa C, con lo que actúa en toda C. Nadie cambia los permisos de un rol que tiene vigente (enmienda D-70). Las negativas conservan su código: `ACCESS_DENIED`, `OWN_ACCOUNT_FORBIDDEN`, `ACTOR_EMPLOYEE_REQUIRED` y `ACCOUNT_MANAGED_BY_SUPERADMIN`. La excepción de D-89 queda donde de verdad está: quien tiene `dar_acceso` con alcance sistema —hoy solo el superadministrador— vincula su propia cuenta mientras no tiene un colaborador activo, y registrar colaboradores no exige al actor un colaborador. Desvincular pasa por la misma guardia que las demás operaciones de cuenta (enmienda D-82) (§5.3, §5.5, §5.8) |
| D-123 | Los roles propios los crea, los edita y los copia quien tiene `seguridad.gestionar_roles` con alcance sistema; hoy, solo el superadministrador. Enmienda D-98, que se lo daba al superadministrador por su rol. Lo demás de D-98 sigue: el código es único dentro de cada empresa y entre los generales, nunca el de un rol del sistema, y un rol de una empresa no se asigna en otra ni es de alcance sistema (§5.5) |
| D-124 | Un rol propio se crea también a partir de otro, como copia exacta de sus permisos: "Crear rol propio a partir de este". La copia no se recorta. Si quien la hace no posee alguno de los permisos, se rechaza nombrando los que faltan (`ROLE_PERMISSIONS_NOT_HELD`); si junta un par, `ROLE_PERMISSIONS_INCOMPATIBLE` (D-119); y si el rol nuevo no es de alcance sistema y la copia trae alguno de los ocho de D-122, `ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`. Copiar `SUPERADMIN` o `ADMIN_ACCESOS` siempre se rechaza —el primero junta pares; el segundo trae `asignar_roles_sin_tenerlos`, que nadie más posee—, y la pantalla lo dice antes de intentarlo. Así un rol del sistema se varía sin tocarlo (enmienda D-97) (§5.5) |
| D-125 | La plataforma de documentos es un módulo propio, `documents` (`ky.millennium.mobility.erp.documents`), dueño de `generated_documents` y de lo que la gobierna: la generación única, el almacenamiento en la base o en Spaces, el vencimiento y las rutas `/documents/{id}` y `/documents/{id}/file`. Deja de ser parte de `shared`, porque tiene su tabla, sus reglas y su vocabulario. Su contrato público es un solo paquete, `documents::acl` (`@NamedInterface("acl")`): la fachada `DocumentService`, que devuelve un tipo del contrato y nunca el agregado; `DocumentRequest`, el pedido; `DocumentSeeOther`, la respuesta `303`; y `DocumentFormat`, `DocumentKind`, `DocumentScopes` y `DocumentContent`. No abre su dominio ni su API. No depende de ningún contexto (`allowedDependencies = {}`), y lo declaran los cinco que exportan: `location`, `iam`, `organization`, `vehiclecatalog` y `fleet`. En `shared` quedan los escritores genéricos de hojas de cálculo (`SpreadsheetExporter`, `SpreadsheetColumn`), que no saben de documentos, y las respuestas `410` y `503` del manejador de errores (§2.7, §3; mapa de contextos, §3 y §5.1; convenciones de la API, §8) |
| D-126 | `GET /api/v1/permission-matrix` es la cuarta lista corta sin página: los permisos con su módulo y su categoría, los pares incompatibles y los roles que la sesión lee, cada uno con lo que incluye, para la pestaña Matriz de permisos de Seguridad › Roles y permisos. Es además la única tabla del frontend fuera de `mm-data-manager`: módulos por roles, con su propio contenedor fijo, como lo registra DESIGN.md en su revisión 24 (convenciones de la API, §6.2; procesos y pantallas, apartado 3.2) |
| D-127 | La sesión se cierra por 2 horas de inactividad, antes 30 minutos (enmienda D-33); el tope de 8 horas desde que se abrió se mantiene. Una jornada con pausas largas en el patio o el taller no pierde el trabajo, y el tope de 8 horas sigue protegiendo la computadora compartida (§5.12; decisión del responsable, 20/10/2026) |
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
| D-2 | Se cargan completas las listas oficiales de divisiones territoriales y se mantienen al día con los cambios que hace cada país. En Perú, la edición 2022 del INEI más los distritos creados por ley después (§4.5, D-77) |
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
