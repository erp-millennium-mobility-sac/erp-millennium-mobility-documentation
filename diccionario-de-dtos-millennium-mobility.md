# Diccionario de datos — ERP Millennium Mobility

**Versión:** 1.57 · **Actualizado:** 13/11/2026 (D-250 a D-253, decididas por el responsable ese día después del análisis crítico de la logística de punta a punta, y construidas en las ramas `feature/contract-guard-and-code` del backend, con V130, y `feature/contract-code-and-brand-words` del frontend, en `develop` desde el 14/11/2026 (merges backend `d7781187` y frontend `1a366868`): una unidad que se firmó o se entregó a un cliente no se anula, también mientras ventas no existe, en §9.6; los códigos del contrato firmado, `CONTRATO_FIRMADO` y `FIRMA_CONTRATO`, en §8.2; «Marca» y «Marca comercial» en toda pantalla; y las palabras de ventas, reservadas antes de diseñar el bloque 12. En §14, lo que necesitaría calcular la retención si Millennium es agente). Antes, 11/11/2026, versión 1.56 (las revisiones 79, 80 y 81 de `DESIGN.md` en la fila de D-247 —frontend, merges `31b51341`, `fa53171a` y `2c18a520`—; la 85, decisión del responsable de ese día, y la ayuda de la retención, en la de D-246 —frontend, merges `28da7f89` y `df4b57cd`—; y en la de D-213, que Por pagar respondió 500 en `dev` hasta el backend `80a4fb23`, merge `23d66dcf`; sin migraciones). Antes, el mismo día, versión 1.55: la revisión 77 de `DESIGN.md`, el último pedido del responsable del 10/11/2026, construida ese día —frontend, merge `3e53ee53`—, en la fila de D-247; y D-213 precisada: cada aplicación de un pago dice la tasa de su factura y su día, contra la que se mide su diferencia de cambio, y la ficha del pago dice de dónde sale la diferencia —backend, merge `aaff72d1`; frontend, merge `b22bbb79`, con la revisión 78—; sin migraciones). Antes, 10/11/2026, versión 1.54: D-247 a D-249 construidas el mismo día e integradas en `develop`, que Flyway lleva a la base de `dev`: backend, merges `ba5bbb15`, con V127, y `4639799f`, con V128 y V129, una migración en Java que da sus usos a los pagos con saldo registrados antes; frontend, merges `8aa05753` y `d66ecb84`, con la revisión 75 de `DESIGN.md`, y `96b1a3f6`, con la 76. Las filas de D-247 a D-249 dicen lo que se fijó al construirlas, entre ello que un vehículo saldado sigue saldado aunque le falte algo además de su inicial, que `…_BALANCE_USED` se revisa antes que `…_BALANCE_NEGATIVE` y que lo que ninguna nota explica se guarda como «sin nota»; §10.4 dice cómo se llenan los usos de los pagos de antes; y este encabezado cuenta las migraciones hasta V129. El último pedido del responsable del mismo día, la revisión 77, ~~está en curso~~ se construyó ese día (versión 1.55). Antes, el mismo día, versión 1.53: el pago por vehículo, el saldo a favor medido por vehículo y el rastro de cada nota, D-247 a D-249, que el responsable decidió ese día después de revisar «Registrar pago» en `dev` y de aprobar un plan auditado contra el código. Decididas antes de construirlas: D-247, la pantalla —el monto de cada vehículo en «Solo algunos vehículos», también con saldo a favor, la copia que conserva los montos y «Cómo se paga» en dos filas—, que el frontend construye con la revisión 75 de `DESIGN.md` y que enmienda D-246; D-248, el saldo medido por vehículo, con el reparto de los pagos sin vehículo guardado al registrar y la inicial del cliente congelada al saldar, con V127, que enmienda D-190, D-197, D-199 y D-211; y D-249, el rastro fijo de las notas que usa cada pago con saldo, con V128 y la tabla `supplier_balance_uses` (§10.4), que enmienda D-195. Cambian §9.5, §9.7, §10.1 a §10.5, la fila de D-212, que dice quién lee cada campo nuevo, y el número de tablas de este encabezado. Antes, el mismo día, versión 1.52: Registrar pago en una página, D-246, construido el mismo día, sin cambio en el backend ni migración: frontend, merges `62304705` y `3a928954`, con las revisiones 69, 70 y 71 de `DESIGN.md`, y `4796ec7b`; la fila de D-246 dice lo que se fijó al construirla, entre ello que el foco va al primer campo con error y no al resumen, como el responsable pidió ese día para todo formulario de página, que el pie es el de Registrar factura y que «constancia» se lee también en los rechazos y en la ficha del pago; y §14 suma si «Por pagar» reemplaza a «Pendiente» fuera de ese formulario. Antes, el mismo día, versión 1.51: el registro de un pago en una página, D-246, que el responsable aprobó ese día después de revisar otra vez «Registrar pago», con una revisión basada en evidencia y tres prototipos: el proveedor y la factura u orden en una fila, los vehículos de la factura con lo que se paga de ella, con qué se paga —también con el saldo a favor que dejan las notas de crédito— y la cuenta como listas, la cuenta propuesta cuando es la única activa en la moneda de los documentos, la constancia como el archivo de una factura, los errores en un resumen arriba y el total al pie. Decidida antes de construirla (documentación `b27c002`), sobre las mismas reglas y sin cambio en el backend ni migración: la construye el frontend con la revisión 69 de `DESIGN.md`. Enmienda D-214 en la cuenta que se propone y la pantalla que decidió D-245, y deja abierto en §14 si Millennium es agente de retención del IGV. Antes, 09/11/2026, versión 1.50: la experiencia de los pagos a proveedores, D-245, rediseñada sobre las mismas reglas con la propuesta que el responsable aprobó ese día, después de revisar los pagos: el monto calculado de lo marcado, un solo vocabulario, el pago de varias facturas y los documentos de cada pago en su lista. Decidida antes de construirla (documentación `2bba268`) y construida el mismo día, sin migración: backend `70dd253b`, `8f84b19c` y `0ca99f14`, en `develop` con el merge `b5308771`, y frontend, merges `5f728fc3` y `7ae68c2d`, con las revisiones 66 y 67 de `DESIGN.md`, y `a0724b7a`; la fila de D-245 dice lo que se fijó al construirla. Antes, el mismo día, versión 1.49: el estado de pago de una factura, D-244, decidido antes de construirlo con la revisión de la ficha y la lista de facturas que pidió el responsable ese día: el vocabulario del de sus unidades, quién lo ve, el filtro de la lista y la cuenta de origen de cada pago de la factura. Construido el mismo día, sin migración: backend `edd12f9a` a `6230346b`, en `develop` con el merge `4d15256d`, y frontend, merge `cf862569`, con la revisión 63 de `DESIGN.md`; la fila de D-244 dice lo que se fijó al construirlo. Antes, el mismo día, versión 1.48: lo construido ese día con D-238 a D-243, integrado en `integration/audit-2026-11-09`: V122, el asesor de ventas registra la entrega, backend `b7d846f3`; V123, En traspaso y su traspaso, backend `9f9bd093` y frontend `9dd5c398`, con la decisión abierta de §14 sobre la carga inicial cerrada por la enmienda de D-225; V124, los períodos de un colaborador, con la regla de que una corrección no adelanta el ingreso antes de la salida del período anterior, backend `dff65113`, `6395606c`, `1136c4f3` y `897b8ef6`, frontend `f07df1aa`, `478047ea` y `f75d6470`; V125, `notifications`, backend `f82caa6c`; V126, el enlace para restablecer la contraseña, backend `af2e9494` y `267ca405`, frontend `06ba1c4d`. Los cuadros de §2.7, §5.3 y §5.4 dicen ahora las columnas, las restricciones y los privilegios que crearon esas migraciones. Y lo que la auditoría de punta a punta del mismo día encontró desactualizado: los ciento veintiún permisos de §5.6, que el texto contaba como ciento dieciocho; las cuarenta y nueve tablas versionadas de §2.7, que eran treinta y cuatro hasta V86; y el número de tablas de este encabezado, que decía 54. Antes, 09/11/2026: fase 0 del plan de cierre de la auditoría, aprobado ese día por el responsable: la entrega al cliente también del asesor de ventas, D-238; el estado En traspaso, D-239; las tolerancias de los caminos por migración, D-240; la historia de los reingresos, D-241; el módulo `notifications` con su primer caso, D-242; y el enlace para restablecer la contraseña, D-243; decidido antes de construirlo, versión 1.47) · **Motor:** PostgreSQL 18 · **Estado:** módulo de stock completo. Bloques 1 a 11 cerrados; ventas y contratos fuera del alcance actual. Proveedores con su ficha y sus contactos (D-137), tipos, plantillas y órdenes de compra construidos (D-132), con el formato impreso de la empresa, el año del modelo en cada línea de vehículo (D-135) y el borrador que no pisa un cambio más nuevo (D-138); facturas, notas de crédito y pagos construidos el 03/11/2026 y corregidos tras su revisión de punta a punta el 04/11/2026, con las propuestas de sus cierres aprobadas ese día por el responsable. Fondos y cesiones a mano construidos antes de las cargas del bloque 9 (D-133). El cierre de la auditoría del 25/10/2026 queda en D-133 y D-134, y el de la revisión del 31/10/2026, en D-144 a D-147: el freno del inicio de sesión, la contraseña medida en bytes, las monedas de hasta dos decimales y el backend que se alcanza solo por Cloudflare. Los tipos de cambio quedan decididos antes de las facturas: por país, cargados a mano y corregidos con motivo (D-148, D-149), y la escritura de los catálogos generales, solo desde un rol de alcance sistema (D-150). La factura, la nota de crédito y el pago proponen la tasa de su fecha y la registran desde su formulario cuando falta (D-151). En Perú las compras y los pagos usan la tasa de venta de su fecha, y la nota de crédito, la de su factura (D-152); la unidad nace con la factura o se vincula por su VIN (D-153). Las tasas fijan los códigos ISO de su país (D-155) y su fecha no pasa de mañana (D-156). La factura se registra a mano primero (D-157), con el ISC por factura (D-158), el precio con impuestos y el costo sin IGV calculado (D-159), su número, sus líneas y sus vínculos (D-160), la anulación que no anula unidades (D-161), su orden (D-162) y sus permisos (D-163). La habilitación gobierna la entrada a Disponible (D-164, D-165), el certificado de gas sigue al modelo (D-166), el aviso de renovación espera a las notificaciones (D-167) y la migración trae las facturas de sus unidades (D-168). La auditoría del plan cierra lo que faltaba para construir: la tasa en la moneda de la empresa (D-169), la puerta fija en Disponible (D-170), los códigos del estado de habilitación (D-171), lo que cambia en una factura registrada (D-172), solo líneas de vehículo en la primera versión (D-173), los archivos admitidos (D-174), el rastro de la anulación (D-175), los permisos de facturas en dos tiempos (D-176) y las excepciones de la carga inicial (D-177). Las especificaciones de facturas y de habilitación cierran sus preguntas: la factura registrada con su candado y su alcance (D-178), el vínculo con una unidad existente y el monto mayor que cero (D-179), la exportación de la base y el IGV (D-180), el tipo de tasa de los demás países (D-181), la cesión abierta que impide anular (D-182), la puerta que exige que no falte nada (D-183) y que se sostiene en el estado (D-184), el código fijo de un equipo en uso (D-185), las fechas de la habilitación (D-186), el costo de una instalación (D-187), tres reglas menores (D-188) y los archivos en `kept_files` (D-189). Las notas de crédito y los pagos quedan cerrados: la nota reduce lo que se debe y solo lo pagado de más es saldo (D-190), medido por vehículo desde D-248, anula unidades solo cuando vuelven al proveedor (D-191), copia la tasa de cada factura (D-192) y se registra y anula como la factura (D-193); el pago se aplica en cualquier moneda (D-194), consume el saldo del proveedor por moneda (D-195) y, desde D-249, guarda de qué notas salió, registra la retención (D-196), detalla unidades en el canal de dealers (D-197), y sus aplicaciones siguen reglas de traslado, unicidad y anulación (D-198); la unidad tiene cinco montos (D-199). Las cargas de archivos viven en el contexto dueño de lo que actualizan (D-200), entran por fila o por factura (D-201) y la carga inicial da de alta cada unidad en su estado (D-202). La especificación de notas cierra el costo en la ficha de la unidad (D-203), las reglas de la nota (D-204) y los 20 días, que no aplican a la nota (D-205). La de cargas cierra los datos de habilitación de las unidades migradas (D-206), sus notas y pagos (D-207), las tres entregas del paquete (D-208) y la lista de garantía, que cierra cesiones (D-209). La de pagos cierra el registro del pago (D-210), sus órdenes e imputaciones (D-211), sus reglas, su alcance y lo que se ve sin `pagos.ver` (D-212), y la diferencia de cambio (D-213). La salida de la flota pasa por En traspaso (D-239), los regresos de un colaborador conservan su período anterior (D-241) y la contraseña se restablece con un enlace que envía `notifications` (D-242, D-243), construidos el 09/11/2026 con V122 a V126. ~~54 tablas~~ ~~58 tablas~~ 59 tablas más `role_permissions`, la tabla de unión de roles y permisos, y las cinco tablas técnicas de §2.7 (`generated_documents`, `kept_files`, `data_versions`, `notifications` y `notification_attempts`): ~~las 64 que crean V1 a V126~~ las 65 que crean V1 a V129 —V127 suma columnas (D-248), V128 crea `supplier_balance_uses` (§10.4, D-249) y V129, una migración en Java, no crea tablas: da sus usos a los pagos con saldo registrados antes; las tres, construidas el 10/11/2026—, sin las que administran las librerías (§2.7). Las 59 incluyen `user_access_suspensions` (§5.4) y no los esbozos del bloque de ventas (`advisor_assignments` y `contracts`, §5.11), que no tienen migración

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

**Tablas técnicas.** Las que administran las librerías no siguen estas convenciones, y las propias de la plataforma no pertenecen a ninguna empresa, salvo `kept_files`, que desde V84 nombra la empresa de cada archivo: las dos clases están en §2.7.

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
- Los catálogos generales —monedas, tipos de cambio, países, divisiones territoriales, tipos de documento tributario, permisos— no se filtran: son iguales para todos.
- Los usuarios no pertenecen a una empresa. Quien tiene `seguridad.ver_cuentas` por una asignación de empresa —el administrador de esa empresa— ve las cuentas vinculadas a colaboradores de esa empresa —también las de quienes ya salieron, para comprobar que la salida les quitó el acceso—, con los roles vigentes que tienen en ella y los de alcance sistema; con alcance sistema, las ve todas (D-122). Verlas no es modificarlas: dar acceso, otorgar y retirar roles, bloquear cuentas y restablecer contraseñas exigen sus permisos con alcance sistema, así que ningún rol de una empresa los hace; hoy son del administrador de accesos y del superadministrador (D-57, D-122).
- Los permisos de los colaboradores actúan donde su asignación dice (D-122, D-130): `seguridad.registrar_colaboradores`, `seguridad.modificar_colaboradores` y `seguridad.registrar_salidas_y_regresos` por un rol de empresa, en toda la empresa; por uno de sede, en esa sede. Un colaborador sin sede es de toda la empresa, y moverlo de sede exige actuar en la de origen y en la de destino.
- El stock es visible para toda la empresa. La sede limita dónde se puede actuar, no qué se ve.
- Un rol de sede actúa sobre las unidades que están en la ubicación propia de su sede; las que están en una ubicación de terceros, o todavía sin ubicación, las tocan los roles de empresa (D-101). La marca de una asignación acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, desde los vehículos (D-102).
- Lo que es de todas las marcas de una empresa —un proveedor, un fondo, un tipo de orden, una plantilla general— lo escriben solo las asignaciones de toda la empresa y sin marca. Una de sede o de marca lo lee, pero no lo escribe: un comprador limitado a una marca no desactiva el proveedor que usan las otras (D-134). Ceder una unidad a un fondo y cerrar la cesión son actos financieros de la empresa: los hace una asignación de toda la empresa o de la marca de la unidad, nunca la de la sede donde está la unidad. Es una excepción a D-101 (§12.2, D-134).
- Hay campos que no ve cualquiera aunque vea la fila. Cuando falta el permiso, el campo no sale de la base: ocultarlo en la pantalla no alcanza.
- El acceso total es una excepción explícita evaluada con las mismas reglas, no una cuenta que se saltea el control.
- **La frontera entre empresas la defiende también la base de datos** (D-66). La aplicación filtra cada consulta por las empresas que alcanza quien la hace. Además, en cada tabla que lleva `company_id` —y en `companies`, por su propio `id`—, la base deja ver y escribir solo las filas de esas empresas. Si algún día una consulta olvida su filtro, devuelve menos filas, nunca las de otra empresa. Un rol de alcance sistema ve todas; una operación sin alcance no ve ninguna.
- **La tabla de direcciones se acota por el país de la dirección** (D-80, D-134). Desde que guarda las direcciones de los proveedores, una dirección se ve y se escribe donde se alcanza una empresa de su país, o con alcance `SYSTEM`. No lleva `company_id`: su dueño —una empresa, una sede, una ubicación, un proveedor o el emisor de una plantilla— es de una empresa, y la dirección está en el país de esa empresa (§4.6). Con una empresa por país (D-5) y solo proveedores locales (D-133), el país nombra a la empresa del dueño, así que equivale a acotar por empresa. Es la excepción a la regla de que la frontera va sobre `company_id`. La rama `SYSTEM` hace falta porque el alta de una empresa crea su domicilio antes de que la empresa exista. Las lecturas que necesitan todas las empresas —si un país tiene direcciones (D-47)— se hacen con una elevación a `SYSTEM` que solo devuelve el dato. Si D-5 o el proveedor local cambian, `addresses` pasa a llevar su `company_id` (candidato, §4.8). El domicilio de los clientes, que sí es un dato sensible, llega ya acotado.

**Nota técnica.** Es Row Level Security de PostgreSQL, habilitada y forzada (`ENABLE` y `FORCE ROW LEVEL SECURITY`) para que tampoco el dueño de las tablas la saltee. Hoy cubre `companies`, `commercial_brands`, `branches`, `locations`, `employees`, `employee_tenure_periods` (desde V124, D-241), `user_roles`, `employee_account_link_events`, las tablas de empresa de flota y de compras y `kept_files`, y cubrirá cada tabla con `company_id` que se cree. `notifications` y `notification_attempts` la tienen sin `company_id`: sus políticas dejan pasar solo una transacción con alcance `SYSTEM` (V125, NTF-R10, §2.7). Cada tabla tiene una política por operación —leer, insertar y actualizar donde alcanza la transacción— y ninguna para borrar, que la cuenta de la aplicación no puede hacer. Las de compras la siguen desde V83: hasta entonces tenían una sola política para todo, que dejaba pasar una fila sin empresa. `kept_files` lleva su empresa y su política desde V84 (§2.7). `addresses` se acota por su país (`access_scope_reaches_country`, V78). Hay dos cuentas de base de datos: la dueña, que solo aplica las migraciones, y la de la aplicación, que no es dueña de nada, no puede saltear la regla y no tiene permiso de borrar, salvo en tres tablas de control que se purgan y no guardan datos del negocio: las sesiones (§5.12), los contadores del freno del inicio de sesión (`sign_in_throttles`, §5.4, D-144) y los enlaces para restablecer la contraseña (`password_reset_tokens`, §5.4, D-243). Fuera de desarrollo, la aplicación no arranca si su cuenta es la de las migraciones, un superusuario o una con `BYPASSRLS`, que se saltan la regla aunque esté forzada: lo pregunta a PostgreSQL antes de atender el primer pedido. Al comenzar cada transacción —nunca al abrir la conexión—, la aplicación fija desde un solo lugar, con `set_config(…, true)`, quién actúa y qué alcanza: `app.access_scope` (`SYSTEM` o `SCOPED`), `app.company_ids` y `app.user_id`. Un valor que falta no ve nada. El arranque y los procesos automáticos actúan con alcance `SYSTEM` de forma explícita, igual que firman como `SYSTEM` (D-42). `user_roles` le muestra además a cada cuenta sus propias asignaciones y las de alcance sistema, para que pueda calcularse su alcance. `employee_account_link_events` lee más estrecho: sus filas las ven solo una cuenta con `SUPERADMIN` vigente y los procesos con alcance `SYSTEM` sin cuenta, no el administrador de accesos ni los roles de una empresa, porque guardan datos de personas de otras empresas (§5.4). Las pocas comprobaciones que necesitan ver más allá de las empresas de quien opera —si la cuenta tiene un colaborador activo en otra empresa, o cuántos superadministradores quedan (§5.9)— se hacen con una elevación a `SYSTEM` acotada a esa comprobación, que solo devuelve el dato.

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

**Las propias de la plataforma** son de un módulo de la plataforma, porque las usan todos los contextos y no son de ninguno. Salvo `kept_files`, cuyos archivos son de la empresa de su plantilla, de su orden, de su factura o de la unidad de su documento de habilitación (V84, D-189), no pertenecen a una empresa. Hoy hay cinco. `generated_documents` y `kept_files`, del módulo `documents` (D-125), siguen las convenciones de §2.2 —`id`, `public_id`, y en la primera las columnas de registro—; `generated_documents` era de `shared` hasta D-125. `data_versions`, de `shared` (D-128), no las sigue: sus filas no son registros que alguien consulte o edite, sino contadores que sube un disparador, así que no llevan `id`, `public_id` ni columnas de registro. `notifications` y `notification_attempts`, del módulo `notifications` (D-143), llegaron con su primer caso, el enlace para restablecer la contraseña (D-242, D-243): las crea V125, construida el 09/11/2026 (backend `f82caa6c`). Tienen Row Level Security sin `company_id`, solo para el alcance `SYSTEM` (§2.5). Van aquí y no en un bloque porque son de un módulo de la plataforma, como las de `documents`: las usan todos los contextos y no son de ninguno.

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
- **La versión de los datos va en la huella.** Cada tipo de documento la lee de los contadores de las tablas que muestra, en `data_versions` (D-128): la versión de una tabla es la suma de sus 64 casillas, y lo que el tipo resuelve de otro contexto llega con la versión que da su fachada. Cada contexto nombra sus propias tablas en su capa de infraestructura, nunca en la de aplicación. La versión es la misma para todos los alcances: un cambio en una empresa cambia la que leen todas, así que el documento se reutiliza un poco menos, nunca de más. Como en este sistema nada se borra, cualquier alta, cambio o desactivación cambia la versión, y el pedido siguiente genera un documento nuevo. Hasta D-128 se leía de las propias tablas —cuántas filas veía quien preguntaba, su `xmin` y su último `updated_at`—, y su costo crecía con ellas: de 150 a 200 ms para el Inventario, sobre el umbral de 50 ms por exportación (convenciones de la API, §8.3).
- **No lleva `company_id` ni Row Level Security**, a diferencia de toda tabla que se consulta por sí sola (§2.5). Un documento no es de una empresa: el de un rol de alcance sistema abarca todas, y el de un catálogo general, ninguna. Su guarda es `scope_key`, que la aplicación comprueba en cada lectura: quien pregunta por un documento de otro alcance recibe la misma respuesta que si no existiera. Es una excepción a esa regla de §2.5, por ser una tabla técnica.
- **Las filas no se borran.** La cuenta de la aplicación lee, inserta y actualiza, y no tiene permiso de borrar, como en el resto del sistema (§2.5). Al vencer, la fila pasa a `vencido` y su `content` se vacía: el archivo deja de ocupar espacio, y la fila queda como constancia de qué se generó, para qué alcance y cuándo. Un proceso programado lo hace cada 15 minutos por omisión (`documents.expire-check-interval`), en una sola instancia a la vez; ningún pedido lo espera, porque el que encuentra un documento listo y vencido lo vence él mismo. Los archivos de Spaces los borra la regla de ciclo de vida del bucket, no la aplicación.
- Un documento no se edita: un pedido con otros criterios, otro alcance u otros datos es otro documento.

#### `kept_files` — Archivos que no vencen

El almacén de bytes que se conservan: el logo y la firma de una plantilla de orden de compra, el PDF de una orden ya enviada (D-125, §9.2), el comprobante y el XML de una factura de compra, el archivo de una nota de crédito (D-189, §9.5) y el archivo de un documento de la habilitación de una unidad (D-189, §11.3). No es `generated_documents`: ese vence y se reutiliza por la huella del pedido; este no vence y se sigue abriendo después. Un archivo de la plantilla se guarda bajo una llave de su contenido, y reemplazarlo escribe una llave nueva (D-134).

Es del módulo `documents`. Ningún contexto lee la tabla. Quien guarda o abre un archivo lo hace por `KeptFileStore`, del contrato `documents::acl`. La fila de negocio —`purchase_order_template_documents.file_path`, `purchase_orders.pdf_path`, `purchase_invoices.file_path` y `xml_path`, `supplier_payments.file_path`, `vehicle_documents.file_path`— guarda el texto de `storage_key` y no una clave foránea.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa de la plantilla, de la orden, de la factura o de la nota de crédito a la que pertenece el archivo, o la de la unidad de un documento de la habilitación | La base deja leer y escribir el archivo solo donde la transacción alcanza esa empresa (§2.5). Quien lo guarda la nombra (`KeptFileStore.put`); en las filas anteriores, V84 la tomó de la plantilla o de la orden que nombra la llave |
| `storage_key` | TEXT | No | La llave del archivo. Uno de la plantilla, por su contenido: `purchase-order-templates/{publicId}/{tipo}/{sha256}`, con el tipo en minúsculas (`logo`, `signature`). El PDF de una orden enviada, fija: `purchase-orders/{publicId}/sent.pdf`. El comprobante y el XML de una factura, por su contenido: `purchase-invoices/{publicId}/document/{sha256}` y `purchase-invoices/{publicId}/xml/{sha256}`. El de una nota de crédito, por su contenido: `credit-notes/{publicId}/document/{sha256}`. El voucher de un pago, por su contenido: `supplier-payments/{publicId}/voucher/{sha256}`. El de un documento de la habilitación, por su contenido: `vehicle-documents/{publicId de la unidad}/{código del tipo, en minúsculas}/{sha256}` | Reemplazar un archivo de la plantilla escribe una llave nueva y la fila del documento pasa a apuntarla; la llave anterior y sus bytes se quedan. Un reintento deja, a lo sumo, un objeto sin referencia, nunca bytes nuevos bajo una fila confirmada. Un archivo de una factura, de una nota de crédito o de un documento de la habilitación no queda así cuando la transacción se revierte: quien lo guardó lo descarta (`KeptFileStore.discardRolledBack`), pero solo si su transacción escribió la llave primero, lo que dice `KeptFileStore.put`. La llave lleva el contenido, así que los mismos bytes —un `PUT` que vuelve a subir el mismo archivo, o uno anterior— dan una llave que un registro confirmado puede estar apuntando |
| `media_type` | VARCHAR(100) | No | El tipo que recibe el navegador | — |
| `storage` | VARCHAR(10) | No | `base`, con los bytes en `content`, o `spaces`, con el objeto en DigitalOcean Spaces | El disco del proceso no guarda archivos |
| `content` | BYTEA | Sí | Los bytes, cuando `storage` es `base`. Vacío cuando están en Spaces | — |
| `size_bytes` | BIGINT | Sí | Cuánto pesa. En las filas anteriores a V74 se llenó con el largo de `content`; en Spaces llega con cada escritura | — |
| `checksum_sha256` | VARCHAR(64) | Sí | SHA-256 en hexadecimal, en minúsculas. Vacío en las filas anteriores a V74 | Distingue un reemplazo de los mismos bytes |
| `created_at` | TIMESTAMPTZ | No | Cuándo se guardó por primera vez esa llave. Sin `created_by`: el autor está en la fila de negocio | — |

**Relaciones.** Apunta a `companies`. Nadie le apunta. La aplicación lee, inserta y actualiza, y no borra.

**Reglas de negocio**

- **Una llave, una fila.** `storage_key` es único. Un archivo de la plantilla nunca se reescribe: los mismos bytes dan la misma llave, y otros bytes, otra. Solo el PDF enviado tiene llave fija, y lo escribe una sola vez el envío que tiene el candado de la orden: es la excepción a la llave por contenido (D-134).
- **Se escribe antes de confirmar y se apunta al confirmar.** El objeto nuevo se guarda dentro de la operación que cambia la plantilla, y la fila del documento lo apunta en la misma transacción. Si la operación se revierte, el objeto queda sin referencia y nadie lo abre. Desde las facturas de compra (plan E15), quien guarda un archivo de una factura, de una nota de crédito o de un documento de una unidad (D-189) registra su descarte, pero solo para una llave que su transacción escribió primero: `KeptFileStore.put` dice si insertó la fila de `kept_files`. Si la transacción se revierte, `KeptFileStore.discardRolledBack` borra el objeto de Spaces; en la base no hace nada, porque la reversión ya quitó la fila. Una llave que una transacción confirmada ya apunta nunca se descarta: la llave lleva el contenido, así que un `PUT` que vuelve a subir los mismos bytes, o un archivo anterior, da una llave que ya existe. Queda un caso: los mismos bytes subidos a la vez para el mismo registro, cuando uno se revierte y el otro confirma; el descarte del que se revirtió corre después de soltar el candado y puede borrar de Spaces el objeto que el otro acaba de volver a escribir. En la base no pasa, porque la reversión ya quitó su fila. Un archivo de la plantilla todavía puede quedar sin referencia.
- **Quitar el archivo de la plantilla no borra esta fila.** La fila de `purchase_order_template_documents` vacía `file_path`; los bytes siguen aquí.
- **Lleva su `company_id` y Row Level Security desde V84**, como toda tabla de empresa (§2.5). Quien lee pasa antes por la plantilla, la orden, la factura, la nota de crédito o la unidad, que están acotadas a su empresa, y además la base no le entrega el archivo de otra empresa aunque tenga su llave: para él, la llave no existe. Hasta V84 la fila no decía de quién era, así que la base no podía atribuirla ni negarla por sí sola. V84 tomó la empresa de la llave —la plantilla o la orden que nombra— y se detiene sin cambiar nada si una fila no nombra ninguna: de quién es se decide, no se limpia.
- **No tiene columnas de autor.** El autor del logo, del PDF enviado o del comprobante queda en la fila de compras, y el de un documento de la habilitación en `vehicle_documents.uploaded_by`.

#### `data_versions` — Versiones de los datos

Un contador por cada tabla que muestra un documento generado, para saber si sus datos cambiaron desde el último documento igual sin leer la tabla entera (D-128). Es de `shared`, porque lo escriben las tablas de todos los contextos. Cada tabla versionada tiene 64 filas, sus casillas, y su versión es la suma de todas: dos personas que escriben a la vez la misma tabla suben, casi siempre, casillas distintas, y no se esperan una a otra.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `table_name` | VARCHAR(63) | No | La tabla que cuenta, como la nombra el esquema: `vehicles` | Una tabla del esquema, nunca un texto que llegó en un pedido |
| `slot` | SMALLINT | No | La casilla, de 0 a 63 | Repartir la cuenta en 64 casillas reparte la espera entre quienes escriben la misma tabla a la vez, como las lecturas de kilometraje |
| `version` | BIGINT | No | Cuántas veces subió esta casilla. Empieza en 0 | Solo se compara: si la suma de una tabla no cambió, sus datos tampoco |

**Restricciones.** Llave primaria (`table_name`, `slot`), `data_versions_pk`; `data_versions_slot_ck`, la casilla entre 0 y 63. Sin `company_id` ni Row Level Security: es una tabla técnica, y la versión es de la tabla entera.

**`bump_data_version()`** es la función que la escribe. Cada tabla versionada tiene dos disparadores que la llaman una vez por sentencia, uno después de insertar y otro después de actualizar (`<tabla>_data_version_insert_tg` y `<tabla>_data_version_update_tg`, `REFERENCING NEW TABLE AS changed FOR EACH STATEMENT`), porque una tabla de transición no admite dos eventos en un mismo disparador. La función:

- no hace nada si la sentencia no cambió ninguna fila: lo lee de su tabla de transición, que no filtra Row Level Security;
- sortea la casilla una sola vez por transacción y por tabla, y la guarda en el ajuste de la transacción `mm_dv.<tabla>`, que reutilizan las sentencias siguientes: una transacción sube y bloquea a lo sumo una casilla de cada tabla, así que dos que escriben la misma tabla no se interbloquean en los contadores;
- suma uno a esa casilla en la misma transacción del cambio: una transacción que confirma tarde cambia la versión cuando confirma, sea cual sea el orden de sus identificadores;
- corre con los derechos de su dueño, la cuenta de Flyway (`SECURITY DEFINER`, con el `search_path` fijo), y falla con un mensaje que lo dice si la tabla no tiene sus casillas.

**Reglas de negocio**

- **La aplicación solo lee esta tabla.** La cuenta de la aplicación tiene `SELECT` y nada más; la escribe solo la función. `hand-over-database.sql` le quita `INSERT` y `UPDATE` al entregar una base.
- **Las filas no se borran**, como en el resto del sistema. Las casillas se siembran al versionar la tabla y solo suben.
- **Una tabla nueva que muestra un documento se versiona en su migración:** siembra sus 64 casillas (`generate_series(0, 63)`) y crea sus dos disparadores. Sin ellas, su primera escritura y su versión fallan diciéndolo.
- **Las tablas versionadas** son ~~las 34~~ las 49 que leen los documentos de hoy. Treinta y cuatro hasta V86: `vehicles`, `vehicle_statuses`, `vehicle_status_history`, `vehicle_mileage_readings` y `funds` (`fleet`); `users`, `user_roles`, `roles` y `user_access_suspensions` (`iam`); `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `tax_id_types` y `addresses` (`location`); `companies`, `branches`, `commercial_brands`, `locations` y `employees` (`organization`); `suppliers`, `supplier_contacts`, `purchase_order_types`, `purchase_order_templates`, `purchase_order_template_document_types`, `purchase_order_template_documents`, `purchase_orders` y `purchase_order_items` (`procurement`); y `vehicle_types`, `vehicle_brands`, `vehicle_models`, `colors`, `equipment_types` y `equipment_type_vehicle_types` (`vehiclecatalog`). Las veinticinco primeras las crean V53, V54 y V55; las de compras, V67, V68, V69 y V74; `funds`, V75. Después, quince más, cada una en su migración: `exchange_rates` (V91); `purchase_invoices` y `purchase_invoice_items` (V94); `vehicle_document_types` (V97); `vehicle_documents` (V98); `vehicle_equipments` (V100); `credit_notes` y `credit_note_items` (V103); `supplier_payments` y `payment_applications` (V107; `payment_allocations` no, porque ningún documento la muestra); `company_bank_accounts` (V108); `vehicle_keys`, `vehicle_tax_declarations` y `vehicle_fund_assignments` (V121, D-237); y `employee_tenure_periods` (V124, D-241). ~~Hasta V86 ninguna otra migración versiona una tabla.~~ La cuenta de 34 se quedó en V86 hasta la versión 1.48 de este diccionario. `status_change_reasons`, `import_batches`, `sign_in_throttles`, `password_reset_tokens`, `notifications`, `notification_attempts` y, desde V128, `supplier_balance_uses` no se versionan.
- **Un riesgo que queda, acotado.** Dos transacciones que escriben las mismas dos tablas en orden opuesto se interbloquean en los contadores cuando cada una sorteó la casilla que tiene la otra en la otra tabla: alrededor de una vez en 4 096. PostgreSQL aborta a una de las dos (`40P01`), y la aplicación la reintenta una sola vez (D-128).

#### `notifications` — La bandeja de salida

Cada aviso que el sistema tiene que enviarle a una persona, con su estado (D-242). Es del módulo `notifications` (D-143). Un contexto lo pide por `notifications::acl` y le pasa el destino que ya resolvió (NTF-R02); ninguno llama a un proveedor. **Ningún correo se envía dentro de la transacción del hecho:** la fila se escribe en ella, y un proceso programado la despacha después, con reintentos. Si el hecho se revierte, la fila no existe; si el proveedor está caído, el hecho no se revierte ni el aviso se pierde. El primer caso es el enlace para restablecer la contraseña (§5.4, D-243). La crea V125.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `channel` | VARCHAR(10) | No | Por dónde sale: `correo`. El SMS y el push llegan con su primer caso | Cada canal tiene su puerto, y cada proveedor es un adaptador de ese puerto (NTF-R06) |
| `kind` | VARCHAR(60) | No | Qué aviso es, en minúsculas con puntos y guiones: `iam.password-reset`, el enlace para restablecer la contraseña | Distingue los avisos sin leer su contenido |
| `recipient` | VARCHAR(255) | No | El destino que resolvió quien la pidió: una dirección de correo, sin espacios y en minúsculas | `notifications` no consulta a ningún contexto para saberlo (NTF-R02) |
| `template` | VARCHAR(60) | No | La plantilla con que se arma el mensaje: un nombre, nunca una ruta. `password-reset` | Las plantillas están en el código (`templates/email`) y sus textos en los bundles (NTF-R09) |
| `parameters` | JSONB | No | Los valores que la plantilla pone en el mensaje, como un objeto. Por omisión, vacío | — |
| `sensitive_parameters` | TEXT[] | No | Cuáles de esos valores son sensibles, como el enlace. Por omisión, ninguno | Son los que se vacían al enviar, al fallar o al vencer (NTF-R11) |
| `status` | VARCHAR(10) | No | `pendiente`, `enviada`, `fallida` o `vencida`. Nace `pendiente` | Una pendiente espera su turno; una fallida agotó sus reintentos o el proveedor la rechazó, y queda visible; una vencida ya no se envía |
| `attempts` | SMALLINT | No | Cuántos intentos lleva. Empieza en 0, y sube cuando el despachador reserva el intento, antes de llamar al proveedor | Con el tope de reintentos, dice cuándo queda fallida (NTF-R05) |
| `next_attempt_at` | TIMESTAMPTZ | Sí | Cuándo toca el próximo intento. Lleno mientras está pendiente y vacío en cualquier otro estado. Al reservar un intento pasa a dos minutos después: si la instancia se detiene mientras envía, otra la toma cuando vence ese plazo | El proceso despacha las pendientes cuyo momento llegó; la espera crece con cada intento fallido (NTF-R04, NTF-R05) |
| `expires_at` | TIMESTAMPTZ | No | Hasta cuándo sirve enviarla. Posterior a `created_at`. El enlace para restablecer la contraseña vence cuando vence el enlace | Un enlace que ya venció no se envía: la notificación pasa a `vencida` |
| `sent_at` | TIMESTAMPTZ | Sí | Cuándo la aceptó el proveedor. Lleno solo en una `enviada` | — |
| `provider_message_id` | VARCHAR(255) | Sí | El identificador que le dio el proveedor | Con él se busca el mensaje en el proveedor |
| `created_at` | TIMESTAMPTZ | No | Cuándo se pidió: el momento del hecho que la origina | Sin autor: la pide un contexto en nombre de su hecho, que ya tiene el suyo |

**Relaciones.** No apunta a ninguna tabla: el destino es un texto que resolvió quien pidió. Le apunta `notification_attempts`.

**Restricciones.** `public_id` único (§2.2). `notifications_channel_ck`: solo `correo`; el SMS y el push llegan con una migración que amplía la lista. `notifications_kind_ck` y `notifications_template_ck`: minúsculas, números y guiones, y puntos en el tipo. `notifications_recipient_ck`: sin espacios, en minúsculas y con algo antes de la arroba. `notifications_parameters_ck`: `parameters` es un objeto. `notifications_status_ck`: los cuatro estados. `notifications_attempts_ck`: no negativo. `notifications_next_attempt_ck`: `next_attempt_at` lleno si y solo si está `pendiente`. `notifications_sent_at_ck`: `sent_at` lleno si y solo si está `enviada`. `notifications_sensitive_parameters_ck`: fuera de `pendiente`, `parameters` no conserva ninguna de las claves de `sensitive_parameters` (NTF-R11). `notifications_expires_at_ck`: `expires_at` posterior a `created_at`. Dos índices parciales sobre las pendientes, por `next_attempt_at` y por `expires_at`, para el despachador.

**Reglas de negocio**

- **Sin `company_id`; la ve, la inserta y la actualiza solo el alcance `SYSTEM`** (NTF-R10). Row Level Security habilitada y forzada, con una política por operación que pide `app.access_scope = 'SYSTEM'` (V125). El módulo escribe la fila con ese alcance dentro de la transacción de quien la pide, y el despachador corre con él. El primer caso es de una cuenta, que no pertenece a una empresa (§2.5). Un aviso de una empresa, como el de renovación (D-167), traerá su `company_id` y las políticas de V29 en su propia migración.
- **No se borra, y se actualiza solo en lo que cambia al enviarla.** La cuenta de la aplicación tiene `SELECT` e `INSERT`, y `UPDATE` solo de `status`, `attempts`, `next_attempt_at`, `sent_at`, `provider_message_id` y `parameters`; no tiene `DELETE`, como en el resto del sistema (§2.5). V125 le quita todo primero, porque los privilegios por omisión de V19 le darían `UPDATE` de toda la fila. `hand-over-database.sql` hace lo mismo en una base que se entrega después.
- **Los parámetros sensibles se vacían al enviar, al fallar o al vencer** (NTF-R11), y la base lo exige (`notifications_sensitive_parameters_ck`). El enlace para restablecer la contraseña no queda guardado después de servir. La fila queda como constancia de qué se envió, a quién y cuándo. El cuerpo del mensaje no se guarda: se arma con la plantilla en cada intento.
- **Se despacha después, con reintentos** (NTF-R03, NTF-R05). Un proceso programado, en una sola instancia a la vez, corre cada 15 segundos por omisión (`NOTIFICATIONS_DISPATCH_INTERVAL`). Primero vence las pendientes cuyo `expires_at` pasó. Después, en una transacción corta, reserva hasta 20 pendientes cuyo `next_attempt_at` llegó (`FOR UPDATE SKIP LOCKED`): cuenta el intento y corre `next_attempt_at` dos minutos. Fuera de toda transacción llama al proveedor, y en otra transacción corta deja el intento en `notification_attempts` y el resultado en la notificación. Un `2xx` la envía. Un `408`, un `429`, un `5xx` o la falta de respuesta programan el siguiente intento, con una espera de 1 minuto que se duplica con cada intento hasta 1 hora; agotados los 5 intentos (`NOTIFICATIONS_DISPATCH_MAX_ATTEMPTS`), queda `fallida`. Otro `4xx`, o una plantilla que no se puede armar, la dejan `fallida` en el acto, porque otro intento no lo cambia.
- **Cada notificación sale al menos una vez** (NTF-R04). Dos corridas nunca toman la misma pendiente, porque el intento se reserva antes de llamar al proveedor. Si la instancia se detiene después de que el proveedor aceptó y antes de anotarlo, el plazo de dos minutos vence y otra corrida la envía de nuevo: el correo puede llegar dos veces, y ninguno se pierde. No hay `idempotency_key` ni número de intento en la tabla; la llave de idempotencia queda candidata, para cuando un proveedor la acepte y un segundo correo importe.
- **Dos proveedores para el correo.** `capturing`, en desarrollo y en pruebas, guarda en memoria los últimos 100 mensajes y los escribe en el registro del servidor, sin llamar a nadie (NTF-R08); los valores sensibles, como el enlace, solo se ven en `dev`, nunca bajo el perfil `prod`. `sendgrid` usa la API v3 de SendGrid por HTTP (`RestClient`), sin su SDK, con la llave en `SENDGRID_API_KEY`, el seguimiento de clics y de aperturas apagado, para que el enlace llegue tal cual, y el `public_id` de la notificación en `custom_args.notification_id`. Fuera de `dev` la aplicación no arranca sin `NOTIFICATIONS_EMAIL_PROVIDER` (`capturing` o `sendgrid`) ni sin `NOTIFICATIONS_EMAIL_FROM`, y con `sendgrid`, sin `SENDGRID_API_KEY`. Mientras el responsable no defina el remitente, el SPF y el DKIM de `millennium.ky` y la cuenta de SendGrid, un entorno desplegado con `capturing` no envía ningún correo: un enlace que envía un administrador deja a la persona sin contraseña hasta que reciba uno real (D-242, §5.4).

#### `notification_attempts` — Los intentos de envío

Cada intento de entregar una notificación al proveedor, con lo que respondió (D-242). Explica por qué una notificación quedó fallida. La crea V125.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `notification_id` | BIGINT FK → `notifications` | No | La notificación que se intentó enviar | — |
| `attempted_at` | TIMESTAMPTZ | No | Cuándo se intentó | — |
| `outcome` | VARCHAR(10) | No | El resultado: `aceptado` si el proveedor la recibió, `fallido` si no | — |
| `error_code` | VARCHAR(60) | Sí | El código del error. Lleno si y solo si falló | Si otro intento puede pasar se decide al anotarlo, no se guarda |
| `provider_response` | VARCHAR(500) | Sí | La respuesta del proveedor, resumida | Explica un fallo sin guardar el mensaje ni sus parámetros sensibles |

**Relaciones.** Apunta a `notifications`. No le apunta nadie.

**Restricciones.** `notification_attempts_outcome_ck`: `aceptado` o `fallido`. `notification_attempts_error_code_ck`: un intento aceptado no tiene `error_code`, y uno fallido lo tiene. Índice por `notification_id`. Sin `public_id` ni número de intento: la API no la expone, y el orden lo da `attempted_at`.

**Reglas de negocio**

- **Solo se inserta.** La cuenta de la aplicación tiene `SELECT` e `INSERT`, y no modifica ni borra: cada intento es un hecho, también el que falla. Tampoco hay política de `UPDATE`.
- **Sin `company_id`, como su notificación;** la ve y la inserta solo el alcance `SYSTEM`, con Row Level Security habilitada y forzada (V125, NTF-R10).
- **Ninguna de las dos se versiona** en `data_versions`: ningún documento las muestra.

---

## 3. Orden de construcción

El orden lo fijan las dependencias, no la importancia del negocio. Una tabla no puede escribirse antes que aquellas a las que referencia.

Cada bloque pertenece a un contexto delimitado, que es la unidad de código: `location`, `iam`, `organization`, `vehiclecatalog`, `fleet`, `procurement` y `shared`. Además hay dos módulos de la plataforma que no son dueños de ningún bloque: `documents`, con los documentos que genera el sistema y su tabla técnica, `generated_documents` (§2.7, D-125), y `notifications`, con los avisos que envía el sistema y sus tablas técnicas, `notifications` y `notification_attempts` (§2.7, D-143, D-242). El mapa de contextos, con quién puede llamar a quién y la estructura de paquetes, se desarrolla en un documento aparte, como propuesta; lo que este diccionario no fija no está decidido. La regla que lo gobierna es una sola: un contexto usa a otro únicamente a través de su fachada, y nunca consulta ni escribe sus tablas; la única excepción son los modelos de lectura declarados como tales, que solo leen.

| Bloque | Contenido | Depende de | Estado |
|---|---|---|---|
| 1 | Geografía y localización: `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses`, `tax_id_types` | — | **Cerrado** |
| 2 | Seguridad: `users`, `user_access_suspensions`, `sign_in_throttles`, `password_reset_tokens`, `roles`, `permissions`, `role_permissions` | — (`roles.company_id` necesita el 3; llegó con la matriz de permisos, D-81, D-98) | **Cerrado** |
| 3 | Empresas: `companies`, `commercial_brands`, `branches`, `locations` | 1, 2 | **Cerrado** |
| 3b | Colaboradores y accesos: `employees`, `employee_tenure_periods`, `user_roles`, `employee_account_link_events`, `advisor_assignments` | 2, 3 | **Cerrado** |
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
| 3b | 2 · La matriz de permisos | `iam`, `shared` | `permissions`, `role_permissions`, `roles.company_id` e `is_active`; la pantalla Seguridad › Roles y permisos; las operaciones y las lecturas de los pasos 1 a 4 exigen un permiso. Con la segunda ronda de la auditoría, Seguridad por permisos, `permissions.category`, `permission_incompatibilities`, la copia de un rol y la pestaña Matriz de permisos (D-117 a D-124, D-126) | La matriz de permisos por rol (§5.6, fijada el 19/10/2026 y extendida a Seguridad el 20/10/2026) | Construido el 19/10/2026 e integrado; la segunda ronda, construida e integrada en `develop` el 20/10/2026 |
| 4 | 4 · Catálogos de flota | `vehiclecatalog` | Tipos de vehículo, marcas y modelos, colores, equipamiento y a qué tipos aplica | Modelos reales, conciliados entre los dos catálogos que advierte §7.3 | Construido, salvo los datos: faltan los modelos reales conciliados (§7.3). Marcas y modelos lista todos los modelos desde el 29/10/2026 (D-136) |
| 5 | 6 · Vehículo y estados | `fleet` | Estados, caminos y motivos, vehículos, historial de estados, kilometraje | — | Construido e integrado en `develop` el 20/10/2026 (V46 a V48) |
| 6 | 11 · Bitácora de cambios | `shared` | `change_logs` | — | Construido e integrado en `develop` el 21/10/2026 (V59 a V61, D-129) |
| 7 | 7 · Compras, que incluye el 5 | `procurement` | Proveedores, tipos y plantillas de orden, órdenes, facturas y notas de crédito | Proveedores | Construido: proveedores con su ficha y sus contactos, tipos, plantillas y órdenes (D-132 a D-138), y desde el 03/11/2026 las facturas y las notas de crédito (D-157 a D-182, D-190 a D-193, D-203 a D-205). Los tipos de cambio que usan están construidos e integrados desde el 02/11/2026 (§4.8, D-148 a D-150, D-155, D-156) |
| 8 | 8 · Pagos | `procurement` | Pagos, a qué se aplican y qué unidades cubren, saldo a favor | — | Construido e integrado en `develop` el 03/11/2026, con V105 a V107 (D-190, D-194 a D-199, D-210 a D-213); los pagos de la carga inicial (D-207) esperan las cargas de archivos, pospuestas |
| 9 | 9 · Habilitación | `fleet`, `shared` | Documentos y sus tipos, equipamiento instalado, llaves, impuesto vehicular, cargas de archivos | — | En curso: la habilitación, construida e integrada en `develop` el 03/11/2026 (D-164 a D-189); faltan el impuesto vehicular y las cargas de archivos |
| 10 | 10 · Fondos | `fleet` | Fondos y cesiones | — | En curso: fondos y cesiones a mano construidos antes de las cargas del paso 9 (V75, D-133); falta la carga del archivo de garantía mobiliaria |
| 11 | Reportes | Modelo de lectura | Indicadores calculados (§10.5) y el exportable de stock | — | Construido: módulo `reporting` (D-232) |
| — | 12 · Ventas y contratos | — | Contratos y asesoras | — | Fuera del alcance actual |

Los cuatro ajustes:

1. **La bitácora va apenas después del vehículo**, no al final. Depende solo del bloque 6, y desde ahí ya hay cambios delicados —corregir una placa o un VIN, o anular una factura— que tienen que quedar registrados.
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
| `decimal_places` | SMALLINT | No | Cuántos decimales usa esa moneda: de 0 a 2 (D-146). Normalmente dos | El peso colombiano no usa céntimos. Si el sistema le pone dos decimales a todo, las compras en Colombia salen descuadradas por centavos y encontrar de dónde viene la diferencia cuesta horas |
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
- Una moneda tiene de 0 a 2 decimales (D-146), la escala con que se guardan todos los montos, `NUMERIC(14,2)`: con más, un monto en esa moneda se redondearía en silencio al guardarse. Una moneda de 3 decimales reabre la decisión.
- Los decimales se pueden corregir mientras ningún monto use la moneda. Desde el primer documento que la usa quedan fijos: cambiarlos redondearía de otra forma los montos ya registrados. Cuenta todo documento con montos en esa moneda, de cualquier estado y de cualquier empresa; hoy, las órdenes de compra (D-138).

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

**Los códigos ISO se corrigen mientras nada cuelga del país** (D-47). Un error de tipeo que se nota después de registrar el país se corrige mientras el país no tenga escalones, direcciones, tipos de documento tributario ni tasas de cambio (D-155). Tampoco puede tener divisiones ni empresas: una división cuelga de un escalón y una empresa necesita un tipo de documento de su país. Desde entonces los códigos pueden haber salido en un archivo o una integración, y quedan fijos. Es el mismo criterio que los decimales de una moneda (§4.1). Dentro del sistema nada se rompe al corregirlos: las demás tablas apuntan al país por su `id`, no por su código.

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
| `country_id` | BIGINT FK → `countries` | No | El país de la dirección | Es el único dato geográfico que siempre se conoce. El distrito puede faltar en la dirección de un país cuyas divisiones no están cargadas |
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

**La división es obligatoria donde el país tiene sus divisiones cargadas (D-53).** Una dirección en Perú no se guarda sin su distrito: sin él, el sistema no puede deducir la provincia ni el departamento. Solo puede faltar en la dirección de un país cuyas divisiones no están cargadas, como sería la de un proveedor del exterior, que hoy es candidato (§9.8, D-133). Lo valida la aplicación, porque depende de qué países tienen su lista cargada. Otra cosa es si el dueño exige tener dirección. Una ubicación de terceros siempre la tiene, y una propia usa la de su sede. Una empresa o una sede puede registrarse sin dirección mientras se completa su registro, pero una vez que la tiene no la quita: solo la corrige o se muda.

**La dirección está en el país de su dueño.** Quien la escribe no elige el país: lo fija el dueño, que es una empresa, una sede, una ubicación, un proveedor o el emisor de una plantilla, y es siempre el de su empresa. Una empresa peruana no puede tener su domicilio fiscal en Colombia. La base lo exige en empresas y sedes, porque la llave hacia `addresses` incluye el país; en las demás, lo valida la aplicación. La Row Level Security de la tabla se apoya en esta regla (§2.5): V78 detiene el despliegue si encuentra una dirección de un proveedor, una ubicación o un emisor en otro país que el de su empresa, porque esa fila quedaría oculta para la empresa que la usa (D-134).

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
| `company_id` | `addresses` | Que una empresa llegue a tener más de un país (D-5) o que se registren proveedores del exterior (§9.8): la dirección se acotaría por la empresa de su dueño y no por su país (D-80, D-134) |
| `exchange_rates` (tabla) | — | ~~El primer documento que convierte montos: la factura de compra~~ Admitida por D-148 (01/11/2026), con el diseño de abajo, y construida e integrada el 02/11/2026, antes de las facturas del paso 7. Las órdenes, ya construidas, no convierten montos |
| `purchase_exchange_rate_type` | `countries` | La misma condición: qué tipo de tasa propone cada país para sus compras; en Perú, `venta` (D-152) |
| `is_group_currency` | `currencies` | El primer reporte consolidado de varias empresas. Se construye con el paso 11, con el diseño de abajo (D-46) |

#### `exchange_rates` — Tipos de cambio (D-46, D-148)

Hoy cada documento guarda el tipo de cambio que usó (§2.2), pero nada lo propone, así que alguien lo escribiría a mano en cada factura. Los ERP que operan en varios países guardan la tasa de cada día y la proponen según la fecha del documento. Cada tasa es de un país: la publica la autoridad de ese país, y la usan todas sus empresas (D-148).

`exchange_rates`, en el bloque 1:

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | El país cuya autoridad publica la tasa: Perú, para la SBS | Una factura usa la tasa del país de su empresa (D-148). Sin el país, el mismo par de monedas tendría una sola tasa por día, y dos países pueden necesitarlo de dos autoridades distintas: una empresa que lleva su contabilidad en una moneda que no es la de su país (§6.2), o dos países con la misma moneda |
| `from_currency_id` | BIGINT FK → `currencies` | No | La moneda que se convierte: USD | — |
| `to_currency_id` | BIGINT FK → `currencies` | No | La moneda a la que se convierte: PEN | — |
| `rate_date` | DATE | No | El día cuyos documentos usan la tasa: la que corresponde a una factura emitida ese día, no el día en que cerró el mercado | Una tasa vale por un día de calendario, no por un momento (§2.6). Qué tasa corresponde a un día lo sabe quien la registra: en Perú, la que SUNAT muestra para esa fecha (D-152); el sistema no interpreta las reglas de publicación de cada fuente (D-148) |
| `rate_type` | VARCHAR(10) | No | `compra`, `venta` u `oficial` | La SBS de Perú publica compra y venta; la TRM de Colombia y el FIX de México publican una sola tasa oficial |
| `rate` | NUMERIC(12,6) | No | Cuántas unidades de la moneda de destino vale una de la de origen | La misma precisión que el `exchange_rate` de facturas, notas y pagos |
| `source` | VARCHAR(20) | No | La autoridad que publicó la tasa: `SBS`, `BCRP`, `BANREP` o `BANXICO`. No admite `manual`: quién la registró lo dice `created_by` (D-148) | Es lo primero que pregunta un auditor sobre una tasa. Una tasa escrita a mano sigue siendo la de su autoridad |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | La persona que la cargó, o `SYSTEM` cuando exista la carga automática (D-148) |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la corrigió por última vez | La pantalla de tasas lo muestra junto a cada tasa; el historial, con el valor anterior y el motivo, está en la bitácora (§13.1, D-149) |

- El país, las dos monedas, la fecha y el tipo no se repiten (D-148). Las dos monedas son distintas y la tasa es mayor que cero. La fecha no pasa de mañana en la zona horaria del país (D-156).
- **La tasa se registra a mano, cuando hace falta** (D-148): no hay que cargar una cada día, sino la que pide un documento. La registra quien tiene `catalogos.registrar_tipo_cambio`, y una tasa que se aparta de la anterior del mismo país, par y tipo —la de un día anterior— más que el umbral pide confirmación antes de guardarse, porque un dígito equivocado queda copiado en los documentos que la usan. El umbral es 3 % por omisión y se configura (`LOCATION_EXCHANGE_RATE_CONFIRMATION_THRESHOLD`).
- **Corregirla es un cambio delicado** (D-149): exige `catalogos.corregir_tipo_cambio` y un motivo, y queda en la bitácora (§13.1), porque una tasa que ya se usó movió montos. Si la corrección cambia el valor de la tasa, pide la misma confirmación de un salto grande que el registro.
- **Quién la carga** (D-149, D-150): la semilla da los dos permisos al superadministrador. La persona de Contabilidad que carga las tasas los recibe por un rol propio general, de alcance sistema, que crea el superadministrador. Un rol de empresa no puede llevarlos: la tabla no tiene empresa, y una tasa sirve a todas las empresas de su país (D-103).
- **La carga automática espera una fuente con acceso permitido** (D-148). Cargar la tasa del día desde la fuente oficial del país, a nombre de `SYSTEM`, era el diseño de D-46; la API del BCRP, que publica la tasa de la SBS, respondió el 01/11/2026 a una consulta automatizada con una página contra robots, y saltarla no es una opción. Llega cuando haya una fuente con acceso permitido para un sistema —un acceso que dé el BCRP o un proveedor con contrato—, sin cambiar la tabla.
- El documento propone la tasa del país de su empresa más reciente de su fecha o anterior, porque las fuentes no publican los fines de semana ni los feriados, y dice de qué día es; la copia en su propia columna. Corregir una tasa después no cambia los documentos que ya la copiaron. Si en un día hábil, de lunes a viernes, la tasa más reciente es anterior a la fecha del documento, el formulario lo avisa y ofrece registrarla ahí mismo, con el permiso y las reglas de este apartado; el aviso no impide seguir con la anterior, porque un feriado tampoco tiene tasa y el sistema no guarda un calendario de feriados (D-151). La nota de crédito no la propone: copia la de la factura que descuenta (D-152). La orden de compra no usa tasa: no convierte montos.
- Qué tipo de tasa propone cada país para sus compras lo dice `countries.purchase_exchange_rate_type`; en los países distintos de Perú y en los que se creen después es `oficial` por omisión, la tasa única que publican la TRM y el FIX (D-181). En Perú es la de venta: la que SUNAT muestra para la fecha de emisión del comprobante, que sale de la SBS, y el documento no la cambia a mano (D-152).

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
- **La respuesta no dice si el correo ya tiene cuenta, y el registro pasa por el freno del inicio de sesión** (D-144). Son condiciones para encenderlo, fijadas con el cierre de la revisión del 31/10/2026: el registro de hoy responde `409` cuando el correo existe, y encendido diría qué cuentas existen a quien pruebe correos.
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

**Relaciones.** Apunta a `companies`, `branches`, `tax_id_types` y `users` (dos veces: su cuenta y quién lo registró). Le apuntan las asignaciones de asesora, las instalaciones de equipamiento, los vínculos que decide el superadministrador (`employee_account_link_events`, §5.4) y sus períodos anteriores (`employee_tenure_periods`, más abajo).

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
- **Registran colaboradores** quienes tienen `seguridad.registrar_colaboradores` y **los modifican** quienes tienen `seguridad.modificar_colaboradores`; registran sus salidas y regresos quienes tienen `seguridad.registrar_salidas_y_regresos`, cada uno donde su asignación actúa (D-117, D-122, D-130): un rol de empresa, en toda la empresa; uno de sede, en su sede. Un colaborador sin sede exige actuar en toda la empresa, y cambiarlo de sede, en la de origen y en la de destino. La semilla da esos permisos al superadministrador y al administrador de la empresa (D-57, D-130). El administrador de accesos no los tiene: quien da acceso no puede también crear al colaborador que lo recibe, y los pares de D-119 impiden que una cuenta junte las dos funciones. Nadie registra su propia salida ni su propio regreso (D-70). Registrar y modificar colaboradores no exige que quien lo hace esté vinculado a uno, así el superadministrador sin vincular crea el suyo; las salidas y los regresos sí lo exigen (D-89, D-122).
- **Leen los colaboradores de una empresa** quienes tienen el permiso (D-93, D-105): la fila sin sus datos personales —nombre, puesto, sede, fechas de ingreso y salida, si está activo y si tiene cuenta— cualquier rol con `configuracion.ver_colaboradores`, para elegir quién instaló un equipo o nombrar a quien registró algo; el documento, el teléfono, la foto y la exportación, solo con `configuracion.ver_datos_personales_colaboradores`, que la matriz da al superadministrador, al administrador de accesos y al administrador de la empresa. Un rol de empresa o de sede que no la administra no ve esos datos personales: quien atiende un local no necesita la planilla de toda la empresa. El campo que no se puede ver no viaja (§2.5).
- **La fecha de ingreso no puede ser posterior a hoy** (D-88), al registrar al colaborador y al modificarlo, como la salida y el regreso. Hoy se calcula como fija §2.6: en la zona de la sede del colaborador; sin sede, en la del país de su empresa. Un colaborador con ingreso futuro podría recibir acceso y roles antes de empezar a trabajar, y no se le podría registrar la salida hasta corregir su ingreso.
- **Un colaborador se registra en una empresa activa y, si tiene sede, en una sede activa** (D-84). El que ya está registrado sigue siendo válido y se sigue modificando aunque su empresa o su sede se hayan desactivado, mientras no cambie de sede; si cambia, la nueva tiene que estar activa.
- **Cuando un colaborador regresa**, se registra su nueva fecha de ingreso, que no puede ser anterior a la de salida ni posterior a hoy, calculado igual que para la salida. El período que termina —su ingreso y su salida de antes— queda en `employee_tenure_periods`, en la misma operación (más abajo, D-241). Si su cuenta estaba cerrada por la salida, en la misma operación se cierra el bloqueo `baja`, y la persona vuelve a entrar con la misma cuenta y su historial completo. Un bloqueo por `seguridad` en curso sigue hasta que alguien lo levante. Los roles no vuelven solos: se asignan de nuevo (§5.9). Regresar es volver a colgarse de la empresa (D-94): un colaborador regresa solo a una empresa activa y, si tiene sede, a una sede activa, porque el regreso le devuelve la cuenta a la persona.
- **Los períodos no se cruzan** (D-241). Al corregir al colaborador (`PUT /api/v1/employees/{id}`), su fecha de ingreso no puede quedar antes de la salida del último período que cerró: se rechaza con `422 EMPLOYEE_HIRE_DATE_BEFORE_PREVIOUS_TERMINATION`, en el campo `hireDate`. El mismo día se acepta, como en el regreso. La regla la impone el servicio (`EmployeeAffiliationPolicy.validateHireDateCorrection`), con el colaborador bloqueado, el mismo candado que toma el regreso antes de guardar su período. Construida el 09/11/2026 (backend `897b8ef6`, frontend `f75d6470`).

#### `employee_tenure_periods` — Los períodos anteriores de un colaborador

Cada vez que un colaborador regresa a la misma empresa, el período que cerró con su última salida queda aquí (D-241). El período vigente sigue en `employees` —`hire_date` y `termination_date`—, y el regreso reemplaza esas fechas: sin esta tabla, cada regreso borraba un período que ya no se recupera. Es de `organization`, como el colaborador. La crea V124.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `employee_id` | BIGINT FK → `employees` | No | El colaborador que regresó | — |
| `company_id` | BIGINT FK → `companies` | No | La empresa del colaborador. Junto con `employee_id` forma la llave hacia `employees (id, company_id)` | La fila no puede nombrar a un colaborador de otra empresa, y la base la separa por empresa (§2.5) |
| `hire_date` | DATE | No | El ingreso del período que cerró | — |
| `termination_date` | DATE | No | La salida que lo cerró. No es anterior al ingreso | Un período de esta tabla siempre terminó: el abierto está en `employees` |
| `created_at` | TIMESTAMPTZ | No | Cuándo se registró el regreso que guardó el período | — |
| `created_by` | BIGINT FK → `users` | No | Quién registró ese regreso | — |

**Relaciones.** Apunta a `employees` y a `companies`, por la llave compuesta, y a `users`. No le apunta nadie.

**Restricciones.** `public_id` único (§2.2). `employee_tenure_periods_employee_company_fk`, hacia `employees (id, company_id)`. `employee_tenure_periods_termination_not_before_hire_ck`, la misma regla que `employees` (V41). Índices por (`employee_id`, `hire_date` descendente), para leer los períodos de un colaborador del más nuevo al más viejo, y por `company_id`.

**Ejemplo**

| colaborador | ingreso | salida | registrado |
|---|---|---|---|
| María Torres | 01/02/2025 | 31/12/2025 | 15/03/2026 |

María salió el 31/12/2025 y regresó el 15/03/2026. Su período anterior queda aquí, y `employees` dice el vigente: ingreso 15/03/2026, sin salida.

**Reglas de negocio**

- **Se escribe en el regreso, en la misma transacción.** Al registrar el regreso (§5.3), antes de reemplazar las fechas del colaborador, se inserta el período que se cierra. Si el regreso no se guarda, el período tampoco.
- **La salida no es anterior al ingreso.** Lo impide un `CHECK`, como en `employees`.
- **Solo se inserta.** Row Level Security por `company_id`, con las políticas de V29 para leer e insertar (§2.5). La cuenta de la aplicación no tiene `UPDATE` ni `DELETE`: un período cerrado no se corrige ni se borra.
- **Se versiona en `data_versions`** (§2.7), con sus 64 casillas y sus dos disparadores, desde V124.
- **Se lee con el colaborador.** `GET /api/v1/employees/{id}/tenure-periods`, con el permiso y el alcance con que hoy se lee su fecha de ingreso: `configuracion.ver_colaboradores`, donde quien pregunta lo lee (D-105). Las fechas de ingreso y de salida están en la fila mínima del colaborador, la que se lee sin sus datos personales. Responde una lista plana de períodos, cada uno con `hireDate` y `terminationDate`: primero el de `employees`, con `terminationDate` vacío si sigue vigente, y después los de esta tabla, del ingreso más nuevo al más viejo. Un colaborador fuera del alcance responde `404 EMPLOYEE_NOT_FOUND`. En la pantalla es «Ver historial de ingresos», en la fila del colaborador, un diálogo de solo lectura (frontend `478047ea`).
- **Los períodos no se cruzan.** Corregir el ingreso del colaborador no lo deja antes de la salida del último período de esta tabla (§5.3).
- **Lo anterior no se recupera.** Los regresos registrados antes de V124 reemplazaron el período anterior, y esos períodos no se reconstruyen.
- **El motivo de la salida no se guarda.** `termination_reason` sigue candidato (§5.10).
- **Construida el 09/11/2026** (backend `dff65113`, `6395606c` y `1136c4f3`; frontend `f07df1aa` y `478047ea`): V124, el período que guarda el regreso en la misma transacción y la lectura.

### 5.4 `users` — Cuentas de acceso

Solo lo necesario para entrar. Quién es la persona vive en `employees`: toda cuenta pertenece a un colaborador, salvo `SYSTEM`, el primer superadministrador hasta que se vincula y la que quedó sin ninguno al desvincularla (§5.3).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `email` | VARCHAR(255) UNIQUE | No | El correo con que entra. No se repite. Se guarda sin espacios y en minúsculas | Una cuenta por persona, no por empresa: quien trabaja para dos empresas del grupo entra una sola vez y su historial queda completo en un solo lugar |
| `password_hash` | VARCHAR(255) | No | La contraseña guardada de forma cifrada e ilegible | Nunca se guarda tal como se escribió. Ni siquiera quien administra el sistema puede verla |
| `must_change_password` | BOOLEAN | No | Obliga a cambiar la contraseña la próxima vez que entra | Cuando se da acceso y se crea una cuenta, se entrega una clave provisional (D-58). Sin esta marca, esa clave se queda para siempre. Restablecer la contraseña ya no entrega una provisional: envía un enlace (D-243). Cuando lo envía un administrador, la marca queda encendida hasta que la persona elige su contraseña con el enlace |
| `password_updated_at` | TIMESTAMPTZ | Sí | Cuándo cambió la contraseña por última vez | Permite exigir el cambio cada cierto tiempo y detectar claves antiguas |
| `mfa_secret` | VARCHAR(255) | Sí | Clave del segundo factor, si lo activó | Un sistema con datos financieros debería poder exigir un segundo paso al entrar |
| `last_login_at` | TIMESTAMPTZ | Sí | Última vez que entró | Con esto se detectan cuentas abandonadas, que son la puerta de entrada más común a un sistema |
| `created_at` | TIMESTAMPTZ | No | Cuándo se creó la cuenta | — |
| `created_by` | BIGINT FK → `users` | Sí | Quién la creó | Primera pregunta de toda revisión de seguridad: quién le dio acceso a quién |
| `updated_at` | TIMESTAMPTZ | No | Última modificación | — |

**Relaciones.** Apunta a sí misma (`created_by`). Le apuntan `employees`, `user_roles`, `user_access_suspensions`, `employee_account_link_events`, `password_reset_tokens` y prácticamente todas las tablas a través de `created_by`, `updated_by` y campos equivalentes.

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

**Contraseñas.** Tienen de 8 caracteres a 72 bytes en UTF-8, sin reglas de composición (D-145). 72 bytes es el máximo que procesa el cifrado que se usa (BCrypt), que cuenta bytes y no caracteres: una letra con tilde o la ñ ocupa dos. La pantalla lo dice en caracteres: «De 8 a 72 caracteres; las letras con tilde y la ñ cuentan doble». La nueva tiene que ser distinta de la actual. Al iniciar sesión, un correo que no existe y una contraseña equivocada reciben la misma respuesta, para no revelar qué cuentas existen. Una cuenta bloqueada se entera de que lo está solo después de que su contraseña coincidió, así la respuesta no le revela nada a quien no la conoce. Los intentos fallidos del inicio de sesión y del cambio de contraseña los frena `sign_in_throttles` (más abajo, D-144).

**Mientras la contraseña es provisional**, la cuenta solo puede consultar quién es y cambiar su contraseña. Sus roles no valen hasta que la cambie. Tampoco recibe un rol reservado —uno que incluye un permiso privilegiado, como `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA`— hasta que la cambie (§5.7, D-72, D-121): quien le entregó la clave provisional la conoce, y no debe poder terminar con una cuenta privilegiada en sus manos.

**Restablecer la contraseña con un enlace** (D-63, enmendada por D-243). Nadie conoce la contraseña de otra persona: el sistema le envía a su correo un enlace de un solo uso (`password_reset_tokens`, más abajo) y ella elige la nueva. El correo lo envía `notifications` (§2.7, D-242): `iam` escribe el pedido en la misma transacción, y un proceso lo despacha después. Hay dos caminos para pedir el enlace:

- **La persona lo pide** desde la pantalla de inicio de sesión, con «¿Olvidó su contraseña?», que abre `/iam/forgot-password`, y `POST /api/v1/authentication/password-reset-requests` con su correo. Responde siempre `202`, exista o no la cuenta, para no revelar cuáles existen; solo un correo mal escrito responde `400`. Cada pedido corre los mismos pasos, sea cual sea el correo: se cuenta en el freno, se genera un token y se lee la cuenta bajo su candado. Pasa por el freno del navegador, con sus propios contadores (`sign_in_throttles`, `kind` `recuperacion`, más abajo), y por una espera de 5 minutos por cuenta, desde el `created_at` de su último enlace. Un pedido frenado también responde `202`. Solo emite a una cuenta sin bloqueo vigente que no sea `SYSTEM`. El enlace vence a los 30 minutos.
- **Un administrador lo envía**: quien tiene `seguridad.restablecer_contrasenas` con alcance sistema —un superadministrador o el administrador de accesos (D-122)—, con `POST /api/v1/users/{id}/password-reset`. Ya no muestra una contraseña provisional: la contraseña anterior deja de valer —la reemplaza el BCrypt de un secreto al azar de 256 bits que nadie conserva—, se cierran todas las sesiones de esa cuenta, como antes (§5.12), y se envía el enlace, que vence a las 24 horas. `must_change_password` queda encendido hasta que la persona use el enlace: mientras tanto la cuenta no tiene una contraseña propia, y un rol reservado no llega a ella (D-121). Responde `202` con el correo enmascarado —`{"sentTo": "j***@millennium.ky"}`, su primera letra, tres asteriscos y su dominio—, para que quien lo envió sepa adónde llegó. Las guardas son las de siempre: nadie lo hace con su propia cuenta, para eso está el cambio de contraseña; la de `SYSTEM` no se restablece, porque esa cuenta no inicia sesión; quien no es superadministrador no lo hace con una cuenta protegida (§5.5, D-121), y ningún rol de una empresa restablece contraseñas: el permiso solo cuenta con alcance sistema (D-57, D-122).

**Usar el enlace.** El enlace es `<URL pública>/iam/reset-password#<token>`, con la URL pública en `APP_PUBLIC_URL`: el token va en el fragmento, que el navegador no envía al servidor ni en el `Referer`. Las pantallas de acceso viven bajo `/iam`. La pantalla `/iam/reset-password` se abre con sesión o sin ella, lee el token, lo quita de la dirección (`history.replaceState`) y lo envía con `POST /api/v1/authentication/password-resets`, con `token` y `newPassword`; además, Cloudflare la sirve con `Referrer-Policy: no-referrer`. La contraseña nueva sigue las reglas de longitud (D-145), o responde `400 VALIDATION_ERROR` en `newPassword`, y es distinta de la actual, o responde `422 BUSINESS_RULE_VIOLATION`. Se usa en dos transacciones cortas, como el cambio de contraseña, para no tener un candado mientras corre BCrypt: la primera lee el enlace y su cuenta; fuera de toda transacción se compara y se cifra la contraseña nueva, con el tope de comparaciones a la vez (`503 SIGN_IN_BUSY`); la segunda vuelve a leer el enlace y la cuenta bajo su candado, fija la contraseña, borra todos los enlaces de la cuenta y cierra sus sesiones. Un enlace que no existe, venció, se usó o fue reemplazado, o cuya cuenta quedó bloqueada o cambió su contraseña entre los dos pasos, responde siempre `422 PASSWORD_RESET_LINK_INVALID`, sin decir cuál. Usar el enlace no vuelve a cero el contador del inicio de sesión de la cuenta (`sign_in_throttles`): si debe hacerlo queda abierto (§14).

**Dar acceso a un colaborador sigue mostrando la provisional** (§5.3, D-58). Enviar también ahí un enlace queda como seguimiento posible.

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

#### `sign_in_throttles` — El freno del inicio de sesión

Los contadores de los intentos fallidos al iniciar sesión y al cambiar la contraseña, y los navegadores de confianza de cada cuenta (D-144); desde V126, también los de los pedidos de un enlace para restablecer la contraseña (D-243). Frena a quien prueba contraseñas sin que nadie pueda cerrar una cuenta ajena: los intentos desde un navegador sin historia comparten un contador por cuenta, vengan de la IP que vengan, y el navegador desde donde la persona ya entró tiene el suyo. Ninguna IP decide nada. Es una tabla de control de `iam`, no un dato del negocio, y por eso sus filas se purgan (más abajo).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `throttle_key` | CHAR(64) UNIQUE | No | La llave del contador: el HMAC-SHA256, en hexadecimal, del correo normalizado, para el contador de la cuenta, o del correo con el identificador del dispositivo, para el de un dispositivo. Lo calcula la aplicación con la clave `IAM_SIGN_IN_KEY` | La tabla nunca guarda el correo ni el identificador de la cookie. Un SHA-256 sin clave de un correo de un dominio conocido se revierte probando correos; con la clave, quien lea la tabla no sabe de quién es cada fila (Ley 29733) |
| `kind` | VARCHAR(12) | No | `cuenta`, `dispositivo` o, desde V126, `recuperacion` | El de la cuenta lo comparten todos los intentos que llegan sin un dispositivo de confianza, también los de un correo que no existe; el de un dispositivo, solo los de ese navegador para esa cuenta. Los de `recuperacion` cuentan los pedidos de un enlace para restablecer la contraseña (D-243), aparte de los del inicio de sesión |
| `email_key` | CHAR(64) | No | El HMAC del correo normalizado, el mismo en el contador de la cuenta y en los de sus dispositivos | Junta los fallos de una cuenta en todos sus contadores, para el aviso de más de 100 en 24 horas |
| `consecutive_failures` | SMALLINT | No | Cuántos fallos seguidos lleva el contador. Empieza en 0 | Los cinco primeros no tienen espera; desde el quinto, cada fallo pone el contador en espera |
| `last_failure_at` | TIMESTAMPTZ | Sí | Cuándo fue el último fallo. Vacío si el contador no tiene ninguno | Un fallo de hace más de 24 horas deja de contar: el contador se enfría solo, sin que nadie lo levante |
| `blocked_until` | TIMESTAMPTZ | Sí | Hasta cuándo el contador está en espera. Vacío = no lo está | Mientras dura, el intento responde `429 SIGN_IN_THROTTLED` sin revisar la contraseña, con los segundos que faltan. Nunca pasa de 15 minutos |
| `trusted_until` | TIMESTAMPTZ | Sí | Hasta cuándo el dispositivo es de confianza para esa cuenta: 180 días desde su último acierto. Vacío en el contador de la cuenta | Sin esta fecha en el futuro, una cookie `MM_DEVICE` no da un contador propio. La fila nace con un acierto, así que la cookie sola no delata ni abre nada |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Cuándo nació el contador y su último cambio | Sin autor: lo escribe el sistema en cada intento, no una persona |

**Relaciones.** No apunta a ninguna tabla ni le apunta nadie. La cuenta se reconoce por la llave y no por una llave foránea a `users`: un correo que no existe tiene su contador igual que uno que existe, y así la respuesta no delata cuál es cuál.

**Ejemplo**

| kind | consecutive_failures | blocked_until | trusted_until |
|---|---|---|---|
| cuenta | 9 | 31/10/2026 10:24 | — |
| dispositivo | 0 | — | 29/04/2027 08:10 |

Las dos filas son de la misma cuenta. Alguien prueba claves contra ella desde navegadores sin historia, y el contador de la cuenta está en espera; su dueña entra igual desde su equipo, que tiene su propio contador. Su acierto no vuelve a cero el de la cuenta.

**Reglas de negocio**

- **Cinco fallos libres; después, una espera que se duplica.** Los cinco primeros fallos seguidos de un contador no tienen espera. El quinto lo pone en espera 1 minuto, y cada fallo siguiente la duplica, hasta 15 minutos. Un fallo de hace más de 24 horas deja de contar. Nunca hay bloqueo definitivo: nadie cierra una cuenta ajena probando claves, tampoco la del único superadministrador, a quien nadie más puede restablecerle la contraseña (D-70). Quien llega desde un navegador nuevo justo durante un ataque a su cuenta espera, o entra desde su equipo habitual: es el costo aceptado.
- **El intento se cuenta antes de comparar la contraseña.** Con la fila del contador bloqueada, el intento se reserva como fallo y se confirma; después se compara la contraseña, fuera de toda transacción, y un acierto vuelve el contador a cero. Así varios pedidos simultáneos sobre el mismo contador no pasan juntos el freno, y un fallo cuenta aunque el resto de la operación no llegue a guardarse.
- **Un acierto vuelve a cero solo el contador que se usó.** Si se entró desde un dispositivo de confianza, el contador de la cuenta no cambia y se enfría solo: si volviera a cero, cada entrada de su dueño desde su equipo le regalaría cinco intentos a quien prueba claves contra esa cuenta. Una cuenta bloqueada (más arriba) cuya contraseña coincidió también vuelve su contador a cero, y la respuesta dice que está bloqueada, como siempre.
- **El dispositivo y su cookie.** Un acierto emite o renueva la cookie `MM_DEVICE` —un identificador al azar de 128 bits y su firma HMAC con `IAM_SIGN_IN_KEY`, nunca el correo— y deja la fila `dispositivo` de esa cuenta con `trusted_until` a 180 días. La cookie es `HttpOnly` y `SameSite=Strict`, `Secure` fuera de desarrollo, viaja solo hacia `/api/v1/authentication` y dura 180 días. Una cookie con la firma equivocada se trata como si no existiera. No es una credencial: sin la contraseña no sirve, y robada solo da el contador de ese dispositivo, que también se frena.
- **El cambio de contraseña pasa por el mismo freno** antes de comparar la contraseña actual.
- **Los pedidos de un enlace para restablecer la contraseña tienen sus propios contadores** (`recuperacion`, V126, D-243), con la misma regla: cinco libres y después una espera de 1 a 15 minutos. Uno es del correo y lo comparten los pedidos que llegan sin un navegador de confianza para esa cuenta; el otro, del correo con ese navegador, cuando su cookie `MM_DEVICE` es de confianza para la cuenta. Cada pedido cuenta, y ninguno vuelve a cero un contador ni hace de confianza un navegador. Sus llaves son el HMAC con `IAM_SIGN_IN_KEY` de un texto propio, distinto del de los contadores del inicio de sesión, y su `email_key` es su misma llave: quien no logra entrar recibe igual su enlace, pedir enlaces no hace esperar a nadie para entrar, y los pedidos no suman al aviso de los fallos de la cuenta. Un pedido frenado responde `202`, como cualquier otro (§5.4).
- **Usar un enlace no vuelve a cero el contador del inicio de sesión de la cuenta.** Quien restablece su contraseña porque la olvidó puede encontrar ese contador en espera: si debe volver a cero queda abierto (§14).
- **A lo sumo dos comparaciones a la vez por instancia.** Sin lugar en 3 segundos, el intento responde `503 SIGN_IN_BUSY`: en una instancia de un solo procesador, más de dos comparaciones a la vez solo se reparten la CPU y vuelven lentas las demás pantallas. El tope es configurable.
- **El aviso.** Una cuenta que suma más de 100 fallos en 24 horas, entre todos sus contadores, deja un aviso en el registro del servidor. Hace visible un ataque repartido entre muchas IP, que el contador de la cuenta deja en unos 100 intentos por día.
- **Se purga, con `DELETE`.** Un proceso programado, en una sola instancia a la vez, borra los contadores sin actividad en 24 horas y los dispositivos de confianza sin uso en 180 días. Es la excepción con nombre a la regla de no borrar, que es de los datos del negocio: un contador vencido no explica nada, y conservarlo guardaría para siempre a qué cuentas se intentó entrar. Con las sesiones (§5.12) y los enlaces para restablecer la contraseña (`password_reset_tokens`, más abajo, D-243), es una de las tres tablas en que la cuenta de la aplicación puede borrar: `hand-over-database.sql` le da `DELETE` sobre ella.
- **Rotar la clave pone todo en cero.** Cambiar `IAM_SIGN_IN_KEY` vuelve todos los navegadores a «sin historia», porque sus cookies dejan de validar, y deja sin uso los contadores anteriores hasta que la purga los borra.

**Restricciones.** `throttle_key` es único. `kind` admite solo `cuenta`, `dispositivo` o `recuperacion` (`sign_in_throttles_kind_ck`, ampliado por V126), en minúsculas como las demás listas cerradas. `consecutive_failures` empieza en 0. Sin `public_id`: la API nunca direcciona una fila suya. Sin `company_id` ni Row Level Security, como `users`: un contador no es de una empresa, y el de un correo que no existe no tendría cuál. Sin columnas de autor ni `data_versions`: lo escribe el sistema y ningún documento lo lee.

#### `password_reset_tokens` — Los enlaces para restablecer la contraseña

Los enlaces de un solo uso con que una persona elige una contraseña nueva (D-243, que enmienda D-63). Cada enlace lleva un token al azar de 256 bits, y la base guarda solo su SHA-256: quien lea la tabla no puede armar un enlace. Es una tabla de control de `iam`, como `sign_in_throttles`, no un dato del negocio, y por eso sus filas se borran. La crea V126.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `user_id` | BIGINT FK → `users` | No | La cuenta cuya contraseña se restablece | — |
| `token_hash` | CHAR(64) | No | El SHA-256 del token, en hexadecimal | El token viaja solo en el enlace. Sin él, la fila no sirve para entrar |
| `created_at` | TIMESTAMPTZ | No | Cuándo se emitió el enlace | — |
| `expires_at` | TIMESTAMPTZ | No | Hasta cuándo vale: 30 minutos si lo pidió la persona, 24 horas si lo envió un administrador | Un enlace que se lee tarde, o que alguien encuentra después en el correo, no sirve para siempre |
| `requested_by` | BIGINT FK → `users` | Sí | El administrador que lo envió. Vacío cuando lo pidió la propia persona | Responde quién pidió restablecer la contraseña de otra cuenta |

**Relaciones.** Apunta a `users` dos veces: la cuenta y quien envió el enlace. No le apunta nadie.

**Reglas de negocio**

- **Un solo uso.** Usar el enlace fija la contraseña, borra todos los enlaces de la cuenta y cierra sus sesiones, en la misma transacción (§5.12). Un enlace nuevo reemplaza al anterior: emitirlo borra los de la cuenta, en la transacción que pide su correo. Un enlace que no existe, venció, se usó o fue reemplazado, o cuya cuenta quedó bloqueada o cambió su contraseña, responde siempre `422 PASSWORD_RESET_LINK_INVALID`.
- **El token va en el fragmento.** El enlace es `<URL pública>/iam/reset-password#<token>`: el fragmento no llega al servidor ni sale en el `Referer`, así que no queda en ningún registro de acceso. Llega solo en el cuerpo de `POST /api/v1/authentication/password-resets`. El token son 32 bytes al azar en Base64 para URL, 43 caracteres; nunca se guarda ni se escribe en el registro del servidor. La aplicación no arranca sin `APP_PUBLIC_URL`, salvo en `dev`.
- **Se borra, con `DELETE`.** Es la excepción con nombre a la regla de no borrar, como `sign_in_throttles` (§2.5): usar un enlace borra los de su cuenta, emitir uno borra el anterior y un proceso programado, en una sola instancia a la vez, borra los vencidos cada hora por omisión (`IAM_PASSWORD_RESET_PURGE_INTERVAL`). V126 le da `DELETE` sobre esta tabla a la cuenta de la aplicación, y `hand-over-database.sql`, en una base que se entrega después. Un enlace usado, reemplazado o vencido no explica nada, y a quién se envió y cuándo queda en `notifications` (§2.7).
- **Los plazos se configuran.** 30 minutos el enlace que pide la persona (`IAM_PASSWORD_RESET_SELF_SERVICE_VALIDITY`), 24 horas el que envía un administrador (`IAM_PASSWORD_RESET_ADMINISTRATOR_VALIDITY`) y 5 minutos entre dos enlaces de una cuenta (`IAM_PASSWORD_RESET_REQUEST_WAIT`).

**Restricciones.** `password_reset_tokens_token_hash_uq`: el SHA-256 no se repite. `password_reset_tokens_token_hash_ck`: 64 caracteres hexadecimales en minúsculas. `password_reset_tokens_expires_at_ck`: `expires_at` posterior a `created_at`. Índices por `user_id` y por `expires_at`, para borrar los de una cuenta y purgar los vencidos. Sin `public_id`: la API nunca direcciona una fila suya; el enlace la encuentra por su token. Sin `company_id` ni Row Level Security, como `users`: una cuenta no es de una empresa. Sin `data_versions`: ningún documento la lee.

**Construida el 09/11/2026** con V126 (backend `af2e9494` y `267ca405`; frontend `06ba1c4d`).

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
| Quien tiene `seguridad.crear_roles` con alcance sistema; hoy, solo el superadministrador (D-122, D-130) | En todas las empresas (`company_id` vacío), o solo en una empresa concreta (`company_id` lleno), eligiéndola al crearlo (D-98) |

Un rol propio nace vacío o como copia exacta de otro (D-124, más abajo). Leen los roles quienes tienen `seguridad.ver_roles` (D-122): con alcance sistema, todos; con una empresa, los del sistema, los generales y los de esa empresa. Hoy lo tienen los tres administradores: el administrador de accesos los lee todos, porque es quien los otorga y necesita saber qué incluye cada uno, y el de una empresa, los de la suya (D-98). Ninguno de los dos crea roles: `seguridad.crear_roles`, `editar_roles`, `desactivar_roles` y `marcar_permisos_de_roles` solo cuentan con alcance sistema, y la semilla se los da solo al superadministrador (D-130).

**Los roles propios de una empresa existen desde la matriz de permisos** (D-81, D-98): `roles.company_id` apunta a la empresa, o queda vacío en los roles generales y en los del sistema; el código es único dentro de cada empresa y entre los generales, y un rol propio nunca usa el código de un rol del sistema. Un rol de una empresa solo se asigna dentro de esa empresa, y no puede ser de alcance sistema.

Cinco reglas que gobiernan esto:

**1. No se puede dar lo que no se tiene.** Al crear o editar un rol, solo se pueden marcar permisos que quien lo está haciendo ya posee. Sin esta regla, el administrador de una empresa podría crear un rol con acceso total y asignárselo: quedaría convertido en superadministrador sin que nadie lo autorizara. Lo que posee quien edita se cuenta según el rol (D-122): para un rol general, solo lo que tiene por asignaciones de alcance sistema; para un rol de la empresa C, lo que actúa en toda C. Y nadie cambia los permisos de un rol que tiene vigente (D-70, D-122): no se agranda a sí mismo por la puerta del rol.

**2. La misma regla vale al asignar.** Restringir solo la creación no sirve de nada: si el administrador no puede crear un rol con acceso total pero sí puede asignarse uno existente que ya lo tiene, el resultado es idéntico. Nadie puede otorgar un rol cuyos permisos excedan los suyos, contados como en la regla 1. La regla se aplica a los roles no reservados: los reservados los otorga solo el superadministrador, que otorga cualquier rol (D-121). La única excepción es el administrador de accesos, con su permiso propio y los límites que se explican más abajo (D-56, D-120).

**3. Los permisos no tienen niveles, son conjuntos.** No existe "un permiso por encima de otro": la comparación es de contenido. El rol nuevo es válido si todos sus permisos están dentro de los de quien lo crea.

**4. El alcance se fija al crear y no cambia.** Si un rol nace para asignarse a nivel de sede, no puede convertirse después en rol de empresa: las asignaciones que ya existen quedarían con datos incoherentes. Para cambiarlo se crea un rol nuevo.

**5. Lo que alguien autorizó sobrevive a su salida.** Si a un administrador se le retiran permisos o deja la empresa, los roles que creó y los accesos que otorgó **siguen siendo válidos**. La autorización fue legítima cuando se dio, igual que un contrato firmado por un gerente que después renunció. Lo que sí deja de poder hacer es seguir editándolos: de ahí en adelante se le aplica la regla 1 con los permisos que tenga en ese momento, o ninguno si ya salió.

Esto es también la razón de fondo por la que un usuario nunca se borra: si desapareciera, los roles y accesos que autorizó quedarían sin responsable y no habría forma de explicar quién los aprobó.

Como los accesos otorgados no caducan solos —salvo los que se otorgan con caducidad (§5.7, §5.8)—, el control que los mantiene sanos es revisarlos cada cierto tiempo. El modelo lo permite sin campos adicionales: cada asignación guarda quién la otorgó y cuándo, así que se puede listar todo lo vigente y depurar lo que ya no corresponde.

Los roles del sistema no se renombran ni se desactivan. Sus permisos los cambia solo el superadministrador, salvo el rol que él mismo tiene vigente (D-70, D-131). Cualquier otra persona los ve en solo lectura. Un rol propio sigue siendo la otra forma de variar lo que hace un rol del sistema (D-97, D-124).

**Roles reservados por lo que incluyen** (D-121). Un rol es reservado si incluye un permiso privilegiado: un `seguridad.*` de una categoría que no es `ver` (§5.6, D-118). Los del sistema que lo son —`SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`— lo eran antes por su código; desde D-121 lo son por su matriz, y un rol propio que incluya un permiso privilegiado también lo es. Un rol reservado lo otorga y lo retira solo un superadministrador, no se limita a una marca (D-59) y va solo a una cuenta que ya cambió su contraseña provisional (D-72); una cuenta que tiene uno vigente es **protegida**, y solo la maneja un superadministrador. Por eso:

- Agregar o quitar un permiso privilegiado en un rol que tiene titulares vigentes es solo del superadministrador.
- Volver reservado un rol se rechaza (`ROLE_RESERVED_HOLDERS_INVALID`) mientras alguno de sus titulares tenga una asignación acotada a una marca, una contraseña provisional o una asignación que no otorgó un superadministrador: esos titulares no cumplirían lo que un rol reservado exige.

**Dónde se marcan los permisos de accesos y roles, y los que escriben los catálogos generales** (D-122, D-130, D-150). Los once que solo cuentan con alcance sistema —`dar_acceso`, `asignar_roles`, `asignar_roles_sin_tenerlos`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas`, `crear_roles`, `editar_roles`, `desactivar_roles` y `marcar_permisos_de_roles`— solo se marcan en roles de alcance `SYSTEM`, que por eso son generales: marcarlos o copiarlos en otro rol se rechaza (`ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`). `confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos` no se marcan en ningún rol propio (D-117). Lo mismo vale para los permisos del módulo `catalogos` que no son de la categoría `ver` (D-150): crear, editar y desactivar monedas, países, tipos de vehículo, marcas y modelos, colores, equipamiento y estados, los de los tipos de documento del vehículo cuando lleguen, y registrar y corregir el tipo de cambio. Un catálogo general sirve a todas las empresas, y un rol de una empresa o de una sede que lo escribiera lo cambiaría para todas.

**Copiar un rol** (D-124). Un rol propio se crea también como copia exacta de los permisos de otro, del sistema o propio: "Crear rol propio a partir de este". La copia no se recorta: si quien la hace no posee alguno de los permisos, se rechaza nombrando los que faltan (regla 1); si junta un par, se rechaza (D-119); y si el rol nuevo no es de alcance sistema y la copia trae alguno de los once de arriba o un permiso que escribe un catálogo general (D-150), también. Copiar `SUPERADMIN` o `ADMIN_ACCESOS` se rechaza siempre: el primero junta pares, y el segundo trae `asignar_roles_sin_tenerlos`, que nadie más posee. La pantalla lo dice antes de intentarlo.

**Una excepción a la regla 2, con nombre propio** (D-56, D-120). El administrador de accesos asigna roles cuyos permisos no tiene: es el permiso `seguridad.asignar_roles_sin_tenerlos`, que tiene solo él y no se marca en un rol propio. Su trabajo es dar acceso, no registrar facturas ni dar de alta unidades, y no tendría sentido darle todos los permisos del sistema solo para que pueda asignarlos. La excepción tiene tres límites, que son los que la hacen segura (D-55):

- **Nunca actúa sobre sí mismo.** No se asigna roles ni actúa sobre su propia cuenta, como nadie (D-70).
- **Nunca otorga ni retira un rol reservado** (D-121), entre ellos `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`. Los reservados los otorga y los retira solo un superadministrador.
- **Nunca actúa sobre una cuenta protegida**, la que tiene un rol reservado vigente: no la vincula a un colaborador, no le asigna ni le retira roles, no la bloquea ni le levanta bloqueos y no le restablece la contraseña. Esas cuentas las administra solo un superadministrador.

Con esos límites, la excepción no abre el camino que la regla 2 cierra: el administrador de accesos no puede convertirse en superadministrador ni convertir a otro, ni quitarle el acceso a quien lo controla. Tampoco puede tomar la cuenta de un administrador de empresa —restablecerle la contraseña y entrar como él— para inventar un colaborador y darle acceso. Todo lo que otorga queda con su nombre en `granted_by`. Es la única excepción a la regla 2; extenderla a otro rol requiere una decisión nueva.

**Límite de lo que la base puede garantizar.** La consistencia entre la empresa del rol y la empresa de la asignación no se puede exigir con una llave foránea, porque el campo admite vacío para los roles generales. Se resuelve con una validación programada que rechaza asignar un rol de una empresa dentro de otra. Las reglas 1 a 3, la excepción del administrador de accesos y sus límites, los roles reservados y los pares incompatibles en las cuentas también viven en la aplicación: la base no sabe qué permisos ni qué roles tiene quien está ejecutando la operación. Lo que sí revisa la base, en la migración que los siembra, es que ningún rol vigente salvo `SUPERADMIN` junte un par y que ninguno que no sea de alcance sistema tenga alguno de los once de D-122 y D-130.

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

**El catálogo** (D-95, D-117, D-130, D-139, D-149). Ciento veintiún permisos, uno por fila de la tabla siguiente, los mismos que siembran las migraciones: nueve V45, dieciséis V46, doce V57, uno V59 (`bitacora.ver`), treinta y nueve V62, dos V63 y tres V64 (proveedores), uno V67 y uno V69 (los gruesos de plantillas y de órdenes), cinco V75 (fondos), ocho V79, dos V90 (tipos de cambio), cuatro V92 (facturas), siete V95 (la habilitación, con los tres de los tipos de documento del vehículo), uno V102 (notas de crédito), tres V105 (pagos), tres V109 (cuentas bancarias), tres V114 (cargas de archivos) y uno V120 (`reportes.ver`). De V121 a V126 ninguna suma uno: V122 marca `stock.entregar` al asesor de ventas (D-238), y V123 y V126 cambian la descripción de `stock.recuperar` y de `seguridad.restablecer_contrasenas`. Hasta la versión 1.47 este párrafo decía ~~ciento dieciocho~~, una cuenta que no siguió a la tabla y que se armaba así: los sesenta y dos de antes más treinta y nueve que parten, en ver, crear, editar y desactivar, las ocho acciones `gestionar_*` de las pantallas ya construidas, los seis que nacieron partidos con su pantalla —crear, editar y desactivar proveedores (V64) y fondos (V75, D-133)—, menos `fondos.gestionar_fondos`, que nunca se sembró, y los ocho que parten `compras.gestionar_plantillas` y `compras.gestionar_ordenes` (V79, D-130), más los tres de los tipos de documento del vehículo, que nacerán partidos con el paso 9 (D-139), y `catalogos.registrar_tipo_cambio`, que nace con los tipos de cambio (D-149). Esos once códigos gruesos —los ocho de V62, `compras.gestionar_proveedores` y los dos de plantillas y órdenes— siguen en la tabla, para la historia de lo que un rol llegó a incluir, y ninguna pantalla los pide ni la matriz los ofrece; V62, V64 y V80 les quitaron la marca a los roles del sistema, y V80 también a los roles propios, firmado por `SYSTEM` (D-130). Una pantalla que todavía no existe, como Facturas, no tiene casilla hasta el paso que la construye, y entonces nace ya partida, no como un `gestionar_*` nuevo. Cada uno lleva su categoría (D-118): "ver" es una lectura; "crear", "editar" y "desactivar_anular" hacen solo eso; "gestionar" queda solo en lo que aún no tiene pantalla; "acciones_especiales" son las excepciones controladas y los hechos con nombre propio, que se dan por separado. Cada paso carga los suyos cuando construye sus operaciones.

| code | module | category | description | Paso |
|---|---|---|---|---|
| `catalogos.ver` | catalogos | ver | Leer monedas, tipos de cambio, países, divisiones, documentos tributarios, catálogos de flota, estados y motivos, documentos del vehículo (D-43) | 1, 4, 5 |
| `catalogos.gestionar_generales` | catalogos | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130). Lo partieron crear, editar y desactivar monedas y países | 1 |
| `catalogos.crear_monedas` | catalogos | crear | Registrar monedas (D-130) | 1 |
| `catalogos.editar_monedas` | catalogos | editar | Corregir monedas (D-130) | 1 |
| `catalogos.desactivar_monedas` | catalogos | desactivar_anular | Activar y desactivar monedas (D-130) | 1 |
| `catalogos.crear_paises` | catalogos | crear | Registrar países (D-130) | 1 |
| `catalogos.editar_paises` | catalogos | editar | Corregir un país y, en uno que ya existe, sus escalones, divisiones y documentos tributarios (D-130) | 1 |
| `catalogos.desactivar_paises` | catalogos | desactivar_anular | Activar y desactivar divisiones y documentos tributarios. Un escalón no se desactiva (D-45, D-130) | 1 |
| `catalogos.gestionar_flota` | catalogos | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130) | 4 |
| `catalogos.crear_tipos_vehiculo` | catalogos | crear | Registrar tipos de vehículo (D-130) | 4 |
| `catalogos.editar_tipos_vehiculo` | catalogos | editar | Corregir tipos de vehículo (D-130) | 4 |
| `catalogos.desactivar_tipos_vehiculo` | catalogos | desactivar_anular | Activar y desactivar tipos de vehículo (D-130) | 4 |
| `catalogos.crear_marcas_y_modelos` | catalogos | crear | Registrar marcas y modelos (D-130) | 4 |
| `catalogos.editar_marcas_y_modelos` | catalogos | editar | Corregir marcas y modelos (D-130) | 4 |
| `catalogos.desactivar_marcas_y_modelos` | catalogos | desactivar_anular | Activar y desactivar marcas y modelos (D-130) | 4 |
| `catalogos.crear_colores` | catalogos | crear | Registrar colores (D-130) | 4 |
| `catalogos.editar_colores` | catalogos | editar | Corregir colores (D-130) | 4 |
| `catalogos.desactivar_colores` | catalogos | desactivar_anular | Activar y desactivar colores (D-130) | 4 |
| `catalogos.crear_equipamiento` | catalogos | crear | Registrar equipamiento (D-130) | 4 |
| `catalogos.editar_equipamiento` | catalogos | editar | Corregir equipamiento (D-130) | 4 |
| `catalogos.desactivar_equipamiento` | catalogos | desactivar_anular | Activar y desactivar equipamiento (D-130) | 4 |
| `catalogos.gestionar_estados` | catalogos | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130). Los caminos siguen pidiendo el permiso que cada uno nombra (D-100). Los tipos de documento del vehículo, que D-100 le daba, tienen los suyos (D-139) | 5 |
| `catalogos.crear_estados` | catalogos | crear | Registrar estados y motivos (D-130) | 5 |
| `catalogos.editar_estados` | catalogos | editar | Corregir estados y motivos (D-130) | 5 |
| `catalogos.desactivar_estados` | catalogos | desactivar_anular | Activar y desactivar estados y motivos (D-130) | 5 |
| `catalogos.crear_tipos_documento` | catalogos | crear | Registrar tipos de documento del vehículo (§11.2, D-139) | 9 |
| `catalogos.editar_tipos_documento` | catalogos | editar | Corregir tipos de documento del vehículo; el código `TIVE` no se corrige (§11.2, D-74, D-139) | 9 |
| `catalogos.desactivar_tipos_documento` | catalogos | desactivar_anular | Activar y desactivar tipos de documento del vehículo (§11.2, D-139) | 9 |
| `catalogos.registrar_tipo_cambio` | catalogos | crear | Registrar la tasa de un día que todavía no la tiene (§4.8, D-149) | 7 |
| `catalogos.corregir_tipo_cambio` | catalogos | acciones_especiales | Corregir una tasa ya registrada, con motivo (§4.8, D-149) | 7 |
| `configuracion.ver_estructura` | configuracion | ver | Empresas, sedes, marcas comerciales y ubicaciones del alcance | 2 |
| `configuracion.gestionar_empresas` | configuracion | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130) | 2 |
| `configuracion.crear_empresas` | configuracion | crear | Registrar empresas. Crear una empresa sigue exigiendo alcance de todo el sistema (D-130) | 2 |
| `configuracion.editar_empresas` | configuracion | editar | Corregir una empresa, su documento y su dirección fiscal (D-85, D-130) | 2 |
| `configuracion.desactivar_empresas` | configuracion | desactivar_anular | Activar y desactivar empresas (D-130) | 2 |
| `configuracion.gestionar_sedes_y_marcas` | configuracion | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130) | 2 |
| `configuracion.crear_sedes` | configuracion | crear | Registrar sedes (D-130) | 2 |
| `configuracion.editar_sedes` | configuracion | editar | Corregir sedes y sus direcciones (D-130) | 2 |
| `configuracion.desactivar_sedes` | configuracion | desactivar_anular | Activar y desactivar sedes (D-130) | 2 |
| `configuracion.crear_cuentas_bancarias` | configuracion | crear | Registrar las cuentas bancarias de la empresa (D-214) | 8 |
| `configuracion.editar_cuentas_bancarias` | configuracion | editar | Corregir las cuentas bancarias de la empresa (D-214) | 8 |
| `configuracion.desactivar_cuentas_bancarias` | configuracion | desactivar_anular | Activar y desactivar las cuentas bancarias de la empresa (D-214) | 8 |
| `configuracion.crear_marcas_comerciales` | configuracion | crear | Registrar marcas comerciales (D-130) | 2 |
| `configuracion.editar_marcas_comerciales` | configuracion | editar | Corregir marcas comerciales (D-130) | 2 |
| `configuracion.desactivar_marcas_comerciales` | configuracion | desactivar_anular | Activar y desactivar marcas comerciales (D-130) | 2 |
| `configuracion.gestionar_ubicaciones` | configuracion | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130) | 2 |
| `configuracion.crear_ubicaciones` | configuracion | crear | Registrar ubicaciones (D-130) | 2 |
| `configuracion.editar_ubicaciones` | configuracion | editar | Corregir ubicaciones y sus direcciones (D-79, D-90, D-130) | 2 |
| `configuracion.desactivar_ubicaciones` | configuracion | desactivar_anular | Activar y desactivar ubicaciones (D-130) | 2 |
| `configuracion.ver_colaboradores` | configuracion | ver | La fila de cada colaborador sin sus datos personales: nombre, puesto, sede, fechas y si tiene cuenta (D-105) | 3 |
| `configuracion.ver_datos_personales_colaboradores` | configuracion | ver | Documento, teléfono y foto, y la exportación de colaboradores (D-93, D-105) | 3 |
| `stock.ver` | stock | ver | Inventario y ficha de la unidad: datos, historial de estados, lecturas, documentos, equipos, llaves y cesiones; vencidas; habilitación. Toda la empresa (D-14), sin costos | 5, 9, 10 |
| `stock.ver_costos` | stock | ver | El costo visto desde la unidad: valor de factura, notas de crédito y valor del vehículo en la ficha, la columna de costo del exportable y los indicadores derivados de costos (§2.5) | 5, 7, 11 |
| `stock.crear_vehiculo` | stock | crear | Dar de alta una unidad | 5 |
| `stock.editar_vehiculo` | stock | editar | Color, motor, año, marca comercial, ubicación actual | 5 |
| `stock.cambiar_estado` | stock | acciones_especiales | Recorrer un camino que no exige un permiso propio (D-100) | 5 |
| `stock.habilitar` | stock | acciones_especiales | El camino En habilitación → Disponible | 5 |
| `stock.vender` | stock | acciones_especiales | El camino Disponible → Contrato firmado | 5 |
| `stock.entregar` | stock | acciones_especiales | El camino Contrato firmado → Activo | 5 |
| `stock.atender_en_taller` | stock | acciones_especiales | Los caminos de taller: Mecánica, Siniestro y Puesta a punto | 5 |
| `stock.recuperar` | stock | acciones_especiales | Los caminos de recuperación y de salida de la flota: Custodia, Robado, Pérdida total, En traspaso e Inactivo; y el traspaso de la unidad que salió (D-223, D-239; la siembra V123) | 5 |
| `stock.registrar_kilometraje` | stock | crear | Lecturas de taller, entrega, devolución y manuales (§8.5) | 5 |
| `stock.registrar_sustitucion_odometro` | stock | acciones_especiales | El cambio de tablero, el hecho distinto que admite que la siguiente lectura sea menor (§8.5) | 5 |
| `stock.corregir_placa` | stock | acciones_especiales | Corregir una placa ya registrada, con motivo (§13.1) | 5, 6 |
| `stock.corregir_vin` | stock | acciones_especiales | Corregir un VIN ya registrado, con motivo (§13.1) | 5, 6 |
| `stock.anular_vehiculo` | stock | desactivar_anular | Anular una unidad por acción explícita, con el candado de §9.6 (D-104) | 7 |
| `stock.cargar_migracion_inicial` | stock | acciones_especiales | La carga del Excel que da de alta unidades (D-18) | 5 |
| `compras.ver` | compras | ver | Proveedores, órdenes, facturas y notas de crédito, con sus importes | 7 |
| `compras.gestionar_proveedores` | compras | gestionar | Quedó en el catálogo. La pantalla usa los tres de abajo (V64) | 7 |
| `compras.crear_proveedores` | compras | crear | Registrar el proveedor y su domicilio (§9.1). Sus contactos se agregan después, en su ficha, con `editar_proveedores` (§9.1.1, D-137) | 7 |
| `compras.editar_proveedores` | compras | editar | Corregir los datos del proveedor y corregir o mudar su domicilio; agregar, corregir, desactivar y reactivar sus contactos, uno por uno, en su ficha (§9.1, §9.1.1, D-137) | 7 |
| `compras.desactivar_proveedores` | compras | desactivar_anular | Activar o desactivar al proveedor (§9.1) | 7 |
| `compras.gestionar_plantillas` | compras | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (V79/V80, D-130) | 7 |
| `compras.gestionar_ordenes` | compras | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (V79/V80, D-130) | 7 |
| `compras.crear_plantillas` | compras | crear | Registrar plantillas de orden de compra y tipos de orden, con su logo y su firma (§9.2) | 7 |
| `compras.editar_plantillas` | compras | editar | Corregir una plantilla, su atención y sus archivos, y renombrar un tipo de orden (§9.2) | 7 |
| `compras.desactivar_plantillas` | compras | desactivar_anular | Activar o desactivar una plantilla o un tipo de orden (§9.2) | 7 |
| `compras.crear_ordenes` | compras | crear | Crear una orden de compra en borrador, que recibe su número (§9.2) | 7 |
| `compras.editar_ordenes` | compras | editar | Corregir el borrador de una orden: proveedor, contacto, moneda, condiciones, líneas y descuento (§9.2) | 7 |
| `compras.enviar_ordenes` | compras | acciones_especiales | Enviar la orden al proveedor; su PDF queda fijo como documento de la orden (§9.2) | 7 |
| `compras.cumplir_ordenes` | compras | acciones_especiales | Dar por cumplida a mano una orden enviada (§9.2) | 7 |
| `compras.anular_ordenes` | compras | desactivar_anular | Anular una orden en borrador o enviada (§9.2) | 7 |
| `compras.registrar_factura` | compras | crear | Registrar una factura de compra con sus líneas | 7 |
| `compras.registrar_factura_antigua` | compras | acciones_especiales | Una factura de más de veinte días (§9.3) | 7 |
| `compras.anular_factura` | compras | desactivar_anular | Anular un comprobante completo (§9.3) | 7 |
| `compras.registrar_nota_credito` | compras | crear | Registrar y anular una nota de crédito y su reparto; la línea que queda en cero se anula y, con «anulación de unidad», también la unidad (§9.4 a §9.6, D-191, D-193) | 7 |
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
| `fondos.crear_fondos` | fondos | crear | Registrar fondos (§12.1, D-133) | 10 |
| `fondos.editar_fondos` | fondos | editar | Corregir el nombre, el tipo y, mientras ninguna cesión lo use, el código de un fondo (§12.1, D-133) | 10 |
| `fondos.desactivar_fondos` | fondos | desactivar_anular | Activar y desactivar fondos (§12.1, D-133) | 10 |
| `fondos.registrar_cesion` | fondos | acciones_especiales | Ceder una unidad a un fondo o cerrar la cesión a mano (§12.2) | 10 |
| `fondos.cargar_archivo_garantia` | fondos | acciones_especiales | El archivo de garantía mobiliaria (D-30) | 10 |
| `bitacora.ver` | bitacora | ver | La bitácora de cambios de la empresa (§13.1) | 6 |
| `reportes.ver` | reportes | ver | Exportable de stock, indicadores de la flota, brecha de días, diferencia por inicial; los costos siguen a `stock.ver_costos` (§10.5) | 11 |
| `seguridad.ver_cuentas` | seguridad | ver | Leer y exportar las cuentas: todas con alcance sistema; con una empresa, las de sus colaboradores (D-122) | 3b |
| `seguridad.ver_roles` | seguridad | ver | Leer los roles, el catálogo de permisos y la matriz: los del sistema, los generales y los de una empresa donde actúa (D-122) | 3b |
| `seguridad.dar_acceso` | seguridad | crear | Crear la cuenta de un colaborador o vincularlo a la que ya tiene; solo con alcance sistema (D-58, D-122) | 3b |
| `seguridad.gestionar_colaboradores` | seguridad | gestionar | Quedó en el catálogo; ninguna pantalla lo pide. Sigue en los pares de D-119 (D-130) | 3b |
| `seguridad.registrar_colaboradores` | seguridad | crear | Registrar colaboradores, en la empresa o la sede de la asignación (D-130) | 3b |
| `seguridad.modificar_colaboradores` | seguridad | editar | Modificar colaboradores, en el origen y en el destino (D-130) | 3b |
| `seguridad.gestionar_roles` | seguridad | gestionar | Quedó en el catálogo; ninguna pantalla lo pide (D-130) | 3b |
| `seguridad.crear_roles` | seguridad | crear | Crear y copiar roles propios; solo con alcance sistema (D-130) | 3b |
| `seguridad.editar_roles` | seguridad | editar | Renombrar roles propios; solo con alcance sistema (D-130) | 3b |
| `seguridad.desactivar_roles` | seguridad | desactivar_anular | Activar y desactivar roles propios; solo con alcance sistema (D-130) | 3b |
| `seguridad.marcar_permisos_de_roles` | seguridad | acciones_especiales | Marcar los permisos de un rol propio; solo con alcance sistema (D-130) | 3b |
| `seguridad.bloquear_cuentas` | seguridad | desactivar_anular | Bloquear cuentas, levantar o cancelar sus bloqueos y leerlos; solo con alcance sistema (§5.4) | 3b |
| `seguridad.registrar_salidas_y_regresos` | seguridad | acciones_especiales | Registrar la salida y el regreso de un colaborador, en la empresa o la sede de la asignación (§5.3, D-122) | 3b |
| `seguridad.asignar_roles` | seguridad | acciones_especiales | Otorgar y retirar roles y leer las asignaciones de una cuenta; solo con alcance sistema (§5.7) | 3b |
| `seguridad.restablecer_contrasenas` | seguridad | acciones_especiales | Enviar a otra cuenta el enlace para restablecer su contraseña; solo con alcance sistema (D-63, D-243) | 3b |
| `seguridad.confirmar_vinculo` | seguridad | acciones_especiales | Vincular la cuenta de otra persona, o una sin colaborador, después de ver a sus titulares; solo del superadministrador (D-82) | 3b |
| `seguridad.desvincular_cuentas` | seguridad | acciones_especiales | Deshacer el vínculo equivocado de un colaborador con su cuenta; solo del superadministrador (D-82) | 3b |
| `seguridad.asignar_roles_sin_tenerlos` | seguridad | acciones_especiales | Otorgar roles no reservados cuyos permisos no se tienen; solo del administrador de accesos (D-56, D-120) | 3b |

Las descripciones y las categorías de esta tabla son las que siembran V56 (la categoría de los veinticinco de los pasos 1 a 5) y V57 (los doce de seguridad, con su descripción y su categoría).

Los ocho de compras que siembra V79 llevan la descripción de esta tabla sin la referencia entre paréntesis.

En total, ciento veintiuno: catorce de ver, ~~veinticuatro~~ veintiséis de crear, ~~veinte~~ veintiuno de editar, catorce de gestionar, ~~veintiuno~~ veintidós de desactivar o anular y veinticuatro de acciones especiales. Hasta la versión 1.47 estos totales sumaban ciento diecisiete; los corrige la versión 1.48, contando la columna `category` de la tabla. De gestionar quedan los once gruesos, que ninguna pantalla pide, y los tres de la habilitación (`habilitacion.gestionar_documentos`, `habilitacion.gestionar_equipos` y `habilitacion.gestionar_llaves`).

Desde D-117, las cuentas, los accesos, los colaboradores y los roles tienen sus permisos en el catálogo: antes los fijaba un rol (D-95). Esperan otra decisión las correcciones delicadas de un monto de factura y de una fecha ya reportada (convenciones de la API, §14.21), que llevarán un permiso cada una cuando tengan su ruta; el bloque 12 traerá los suyos, entre ellos `clientes.ver_cartera_completa`.

**La matriz** (D-95). Lo que incluye cada rol del sistema. SA = `SUPERADMIN` · AA = `ADMIN_ACCESOS` · AE = `ADMIN_EMPRESA` · OP = `OPERACIONES` · CO = `CONTABILIDAD` · AV = `ASESOR_VENTAS` · TA = `TALLER`.

| Permiso | SA | AA | AE | OP | CO | AV | TA |
|---|---|---|---|---|---|---|---|
| `catalogos.ver` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `catalogos.gestionar_generales` | | | | | | | |
| `catalogos.crear_monedas` | ✔ | | | | | | |
| `catalogos.editar_monedas` | ✔ | | | | | | |
| `catalogos.desactivar_monedas` | ✔ | | | | | | |
| `catalogos.crear_paises` | ✔ | | | | | | |
| `catalogos.editar_paises` | ✔ | | | | | | |
| `catalogos.desactivar_paises` | ✔ | | | | | | |
| `catalogos.gestionar_flota` | | | | | | | |
| `catalogos.crear_tipos_vehiculo` | ✔ | | | | | | |
| `catalogos.editar_tipos_vehiculo` | ✔ | | | | | | |
| `catalogos.desactivar_tipos_vehiculo` | ✔ | | | | | | |
| `catalogos.crear_marcas_y_modelos` | ✔ | | | | | | |
| `catalogos.editar_marcas_y_modelos` | ✔ | | | | | | |
| `catalogos.desactivar_marcas_y_modelos` | ✔ | | | | | | |
| `catalogos.crear_colores` | ✔ | | | | | | |
| `catalogos.editar_colores` | ✔ | | | | | | |
| `catalogos.desactivar_colores` | ✔ | | | | | | |
| `catalogos.crear_equipamiento` | ✔ | | | | | | |
| `catalogos.editar_equipamiento` | ✔ | | | | | | |
| `catalogos.desactivar_equipamiento` | ✔ | | | | | | |
| `catalogos.gestionar_estados` | | | | | | | |
| `catalogos.crear_estados` | ✔ | | | | | | |
| `catalogos.editar_estados` | ✔ | | | | | | |
| `catalogos.desactivar_estados` | ✔ | | | | | | |
| `catalogos.crear_tipos_documento` | ✔ | | | | | | |
| `catalogos.editar_tipos_documento` | ✔ | | | | | | |
| `catalogos.desactivar_tipos_documento` | ✔ | | | | | | |
| `catalogos.registrar_tipo_cambio` | ✔ | | | | | | |
| `catalogos.corregir_tipo_cambio` | ✔ | | | | | | |
| `configuracion.ver_estructura` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `configuracion.gestionar_empresas` | | | | | | | |
| `configuracion.crear_empresas` | ✔ | | | | | | |
| `configuracion.editar_empresas` | ✔ | | | | | | |
| `configuracion.desactivar_empresas` | ✔ | | | | | | |
| `configuracion.gestionar_sedes_y_marcas` | | | | | | | |
| `configuracion.crear_sedes` | ✔ | | | | | | |
| `configuracion.editar_sedes` | ✔ | | | | | | |
| `configuracion.desactivar_sedes` | ✔ | | | | | | |
| `configuracion.crear_cuentas_bancarias` | ✔ | | ✔ | | | | |
| `configuracion.editar_cuentas_bancarias` | ✔ | | ✔ | | | | |
| `configuracion.desactivar_cuentas_bancarias` | ✔ | | ✔ | | | | |
| `configuracion.crear_marcas_comerciales` | ✔ | | | | | | |
| `configuracion.editar_marcas_comerciales` | ✔ | | | | | | |
| `configuracion.desactivar_marcas_comerciales` | ✔ | | | | | | |
| `configuracion.gestionar_ubicaciones` | | | | | | | |
| `configuracion.crear_ubicaciones` | ✔ | | | | | | |
| `configuracion.editar_ubicaciones` | ✔ | | | | | | |
| `configuracion.desactivar_ubicaciones` | ✔ | | | | | | |
| `configuracion.ver_colaboradores` | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| `configuracion.ver_datos_personales_colaboradores` | ✔ | ✔ | ✔ | | | | |
| `stock.ver` | ✔ | | ✔ | ✔ | ✔ | ✔ | ✔ |
| `stock.ver_costos` | ✔ | | ✔ | | ✔ | | |
| `stock.crear_vehiculo` | ✔ | | ✔ | ✔ | | | |
| `stock.editar_vehiculo` | ✔ | | ✔ | ✔ | | | |
| `stock.cambiar_estado` | ✔ | | ✔ | ✔ | | | |
| `stock.habilitar` | ✔ | | ✔ | ✔ | | | |
| `stock.vender` | ✔ | | ✔ | | | ✔ | |
| `stock.entregar` | ✔ | | ✔ | | | ✔ | |
| `stock.atender_en_taller` | ✔ | | ✔ | ✔ | | | ✔ |
| `stock.recuperar` | ✔ | | ✔ | | | | |
| `stock.registrar_kilometraje` | ✔ | | ✔ | ✔ | | ✔ | ✔ |
| `stock.registrar_sustitucion_odometro` | ✔ | | ✔ | | | | ✔ |
| `stock.corregir_placa` | ✔ | | ✔ | | | | |
| `stock.corregir_vin` | ✔ | | ✔ | | | | |
| `stock.anular_vehiculo` | ✔ | | ✔ | | | | |
| `stock.cargar_migracion_inicial` | ✔ | | ✔ | | | | |
| `compras.ver` | ✔ | | ✔ | ✔ | ✔ | | |
| `compras.gestionar_proveedores` | | | | | | | |
| `compras.crear_proveedores` | ✔ | | ✔ | ✔ | | | |
| `compras.editar_proveedores` | ✔ | | ✔ | ✔ | | | |
| `compras.desactivar_proveedores` | ✔ | | ✔ | ✔ | | | |
| `compras.gestionar_plantillas` | | | | | | | |
| `compras.gestionar_ordenes` | | | | | | | |
| `compras.crear_plantillas` | ✔ | | ✔ | | | | |
| `compras.editar_plantillas` | ✔ | | ✔ | | | | |
| `compras.desactivar_plantillas` | ✔ | | ✔ | | | | |
| `compras.crear_ordenes` | ✔ | | ✔ | ✔ | | | |
| `compras.editar_ordenes` | ✔ | | ✔ | ✔ | | | |
| `compras.enviar_ordenes` | ✔ | | ✔ | ✔ | | | |
| `compras.cumplir_ordenes` | ✔ | | ✔ | ✔ | | | |
| `compras.anular_ordenes` | ✔ | | ✔ | ✔ | | | |
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
| `fondos.crear_fondos` | ✔ | | ✔ | | | | |
| `fondos.editar_fondos` | ✔ | | ✔ | | | | |
| `fondos.desactivar_fondos` | ✔ | | ✔ | | | | |
| `fondos.registrar_cesion` | ✔ | | ✔ | | ✔ | | |
| `fondos.cargar_archivo_garantia` | ✔ | | ✔ | | ✔ | | |
| `bitacora.ver` | ✔ | | ✔ | | | | |
| `reportes.ver` | ✔ | | ✔ | ✔ | ✔ | | |
| `seguridad.ver_cuentas` | ✔ | ✔ | ✔ | | | | |
| `seguridad.ver_roles` | ✔ | ✔ | ✔ | | | | |
| `seguridad.dar_acceso` | ✔ | ✔ | | | | | |
| `seguridad.gestionar_colaboradores` | | | | | | | |
| `seguridad.registrar_colaboradores` | ✔ | | ✔ | | | | |
| `seguridad.modificar_colaboradores` | ✔ | | ✔ | | | | |
| `seguridad.gestionar_roles` | | | | | | | |
| `seguridad.crear_roles` | ✔ | | | | | | |
| `seguridad.editar_roles` | ✔ | | | | | | |
| `seguridad.desactivar_roles` | ✔ | | | | | | |
| `seguridad.marcar_permisos_de_roles` | ✔ | | | | | | |
| `seguridad.bloquear_cuentas` | ✔ | ✔ | | | | | |
| `seguridad.registrar_salidas_y_regresos` | ✔ | | ✔ | | | | |
| `seguridad.asignar_roles` | ✔ | ✔ | | | | | |
| `seguridad.restablecer_contrasenas` | ✔ | ✔ | | | | | |
| `seguridad.confirmar_vinculo` | ✔ | | | | | | |
| `seguridad.desvincular_cuentas` | ✔ | | | | | | |
| `seguridad.asignar_roles_sin_tenerlos` | | ✔ | | | | | |

Las doce filas de seguridad suman 22 marcas: once del superadministrador, siete del administrador de accesos y cuatro del administrador de empresa (D-117). Con ellas, las marcas vigentes de los roles del sistema pasan de 76 a 98.

La fila de `bitacora.ver` suma dos marcas, del superadministrador y del administrador de empresa, que se siembran con el paso 6 (V59): las marcas vigentes de los roles del sistema pasan de 98 a 100.

V62 desmarca las ocho acciones gruesas en los roles del sistema (nueve marcas: las ocho del superadministrador y `gestionar_colaboradores` del administrador de empresa) y marca las treinta y nueve partidas, todas en el superadministrador y `registrar_colaboradores` y `modificar_colaboradores` también en el administrador de empresa: 100 − 9 + 41 = 132 marcas vigentes.

V79 marca los ocho de compras en los roles del sistema sin quitarle a nadie lo que hacía: los ocho al superadministrador y al administrador de empresa, que tenían `gestionar_plantillas` y `gestionar_ordenes`, y los cinco de órdenes a Operaciones, que tenía `gestionar_ordenes`; Contabilidad, ninguno (veintiuna marcas). V80 desmarca los dos gruesos (cinco marcas). Las dos migraciones también remarcan los roles propios (D-130).

V122 marca `stock.entregar` al asesor de ventas, que registra la entrega en su sede (D-101, D-238): una marca más. El administrador de empresa la conserva.

Cómo se lee cada rol: el superadministrador tiene todos los permisos salvo `seguridad.asignar_roles_sin_tenerlos`, porque "el acceso total es una excepción explícita evaluada con las mismas reglas" (§2.5) y porque es lo que le permite crear cualquier rol bajo la regla 1 de §5.5; otorga cualquier rol, también uno que incluya el permiso que no tiene, porque los reservados los otorga solo él y la regla 2 se aplica a los demás (D-121). El administrador de accesos lee la estructura de las empresas, sus colaboradores y los catálogos, porque lo necesita para dar acceso y fijar el alcance de un rol, y nada del negocio: "su trabajo es dar acceso, no registrar facturas" (D-56); de seguridad tiene los siete permisos de las cuentas, los accesos y los roles, y la excepción a la regla 2 (D-120). El administrador de empresa hace todo lo operativo de su empresa, incluidas las excepciones controladas, y no escribe empresas, sedes, marcas, ubicaciones ni catálogos, que son del superadministrador (§5.2, D-43, §7); de seguridad, lee las cuentas y los roles y registra a sus colaboradores, sus salidas y sus regresos. Operaciones compra y recibe unidades y las habilita; ve los importes de los documentos que registra, pero no el costo por unidad, y no registra facturas atrasadas. Contabilidad lleva el dinero: facturas, también las atrasadas, notas de crédito, pagos, fondos y los costos; no da de alta unidades a mano ni las mueve, aunque las que nacen con las facturas que registra quedan dadas de alta (D-153, D-163). El asesor de ventas ve el stock de toda la empresa (D-14) y, en su sede, firma contratos, entrega la unidad al cliente y registra las lecturas de entrega y devolución. El taller, en su sede, mueve unidades a taller y de vuelta, registra lecturas y sustituciones de odómetro y gestiona equipos y llaves. La entrega al cliente (`stock.entregar`) la registran el asesor de ventas, sobre las unidades de su sede (D-101), y el administrador de la empresa (D-238, que cierra la pregunta 4 de procesos y pantallas).

Los permisos de facturas se siembran en dos tiempos (D-176): con el paso de facturas, solo al superadministrador, que las prueba en desarrollo; con las notas de crédito, al administrador de empresa, a Operaciones y a Contabilidad, como marca esta matriz. Así ninguna factura real se registra antes de que existan las notas que reflejan los bonos de campaña.

Contabilidad puede registrar facturas atrasadas y Operaciones no: es la excepción que pide la guía funcional, resuelta con un permiso en vez de con una condición escrita en el programa. La separación crear proveedor / pagar, o registrar unidad / anular, se cumple en esta matriz, pero es una propiedad de la semilla y no una regla: el diccionario separa funciones solo con los pares de seguridad de D-119 (abajo).

**Escribir exige leer** (D-134). Las pantallas de compras y sus lecturas —proveedores, tipos, plantillas y órdenes, con el PDF y la exportación— exigen `compras.ver` donde actúa quien pregunta: un permiso de escritura no las abre. Escribir sobre un registro que quien llama no lee con `compras.ver` responde como si no existiera, y emitir una orden exige leer su plantilla y su proveedor. La respuesta de una escritura es lo que el comando guardó, sin volver a filtrar. Los fondos siguen la misma regla con `fondos.ver`, y los catálogos con `catalogos.ver` desde el 30/10/2026: sus pantallas no se abren con un permiso de escritura (D-141). No es una regla de la matriz: los permisos de una cuenta son la suma de sus roles y asignaciones, así que un rol que marca una escritura sin `ver` se guarda igual, y quien solo tiene la escritura recibe la negativa de lo que no lee.

**Los pares incompatibles** (D-119). `permission_incompatibilities` guarda los pares de permisos que ningún rol propio incluye juntos y que ninguna cuenta junta entre sus roles vigentes, en ninguna empresa. Quien registra a los colaboradores no es quien les da acceso: así nadie inventa un colaborador y le da acceso (D-57).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `permission_a_id` | BIGINT FK → `permissions` | No | Uno de los dos permisos del par | — |
| `permission_b_id` | BIGINT FK → `permissions` | No | El otro. Siempre el de identificador mayor (`CHECK (permission_a_id < permission_b_id)`) | Un par se guarda una sola vez, en un solo orden, y el `UNIQUE` lo detecta repetido |

La tabla la crea V58, con un identificador interno (`id`) que esta tabla no lista, como no lo listan las demás; el `UNIQUE` es `permission_incompatibilities_pair_uq` y el `CHECK`, `permission_incompatibilities_order_ck`.

Es un catálogo del sistema, como `permissions`: lo carga la migración de `iam` y la aplicación solo lo lee; no lleva `public_id`, columnas de registro ni Row Level Security. Los veintiocho pares (D-119 y D-130), ✕ = no se juntan. Los siete de `gestionar_colaboradores` siguen en la tabla aunque la matriz ya no ofrece ese código. Como un código retirado no autoriza nada (D-130), la aplicación no lo cuenta en los pares: un rol propio que todavía lo tiene puede recibir `dar_acceso`, y sus titulares no juntan un par por él. Sí sigue contando para la reserva del rol (D-121) y para el alcance sistema de los once de D-122 (D-134):

| Permiso | `seguridad.gestionar_colaboradores` | `seguridad.registrar_salidas_y_regresos` | `seguridad.registrar_colaboradores` | `seguridad.modificar_colaboradores` |
|---|---|---|---|---|
| `seguridad.dar_acceso` | ✕ | ✕ | ✕ | ✕ |
| `seguridad.asignar_roles` | ✕ | ✕ | ✕ | ✕ |
| `seguridad.bloquear_cuentas` | ✕ | ✕ | ✕ | ✕ |
| `seguridad.restablecer_contrasenas` | ✕ | ✕ | ✕ | ✕ |
| `seguridad.confirmar_vinculo` | ✕ | ✕ | ✕ | ✕ |
| `seguridad.desvincular_cuentas` | ✕ | ✕ | ✕ | ✕ |
| `seguridad.asignar_roles_sin_tenerlos` | ✕ | ✕ | ✕ | ✕ |

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
7. **Las cuentas, los accesos, los colaboradores y los roles también van por permiso, con su alcance** (D-117, D-122; hasta entonces iban por rol, D-95). Los habilitan los dieciséis permisos `seguridad.*` no retirados de §5.6 (D-130). Los once de accesos y roles solo cuentan desde asignaciones de alcance sistema y solo se marcan en roles de alcance sistema; los tres de los colaboradores actúan en la empresa o la sede de su asignación, como cualquier permiso (regla 6); `ver_cuentas` y `ver_roles` leen según la suya. Lo que la separación de funciones separa no lo puede juntar un rol propio ni una cuenta, por los pares incompatibles (D-119), y lo que solo hace el superadministrador lo cuidan los roles reservados (D-121). Los límites sobre el destino —la propia cuenta, una cuenta protegida, `SYSTEM`, el último superadministrador— quedan fijos en el código, con sus políticas, y responden los mismos códigos de siempre.

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
| `termination_reason` | `employees` | ~~Una pantalla con la historia del colaborador, o un reporte de rotación, que muestre por qué salió.~~ La historia existe desde el 09/11/2026: «Ver historial de ingresos», en la fila del colaborador, muestra cada período con su ingreso y su salida (`employee_tenure_periods`, §5.3, D-241; frontend `478047ea`), y no el motivo, porque no hay dónde guardarlo. Lo admitiría ahora que el negocio pida ver en esa historia por qué salió cada vez, o un reporte de rotación. Si el reporte agrupa por motivo, entra como catálogo y no como texto libre, y el motivo de cada salida iría también en el período que se cierra. Si entra lo decide el responsable (§14) |

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
| Restablecimiento de la contraseña | Se cierran todas las sesiones de la persona cuando un administrador le envía el enlace y cuando se usa cualquier enlace (D-243). Después del envío de un administrador, la cuenta sigue obligada a elegir su contraseña hasta usar el enlace (§5.4) | Nadie sigue dentro con la contraseña anterior (§5.4) |
| Bloqueo de la cuenta | Se cierran todas sus sesiones cuando el bloqueo empieza, también si estaba programado | Ver §5.4 |
| Cerrar sesión | Se borra la sesión en el servidor, no solo en la pantalla | La cookie que quedara en el navegador ya no sirve para nada |
| Roles, bloqueos y contraseña provisional | Se leen de la base en cada operación, no se guardan en la sesión | Un rol otorgado o retirado vale desde la operación siguiente, y un bloqueo programado corta la sesión apenas empieza, sin esperar a que la persona vuelva a entrar |

**Protección de la cookie.** Viaja marcada como inaccesible para el código de la página (`HttpOnly`), solo hacia el mismo sitio (`SameSite=Strict`) y, en producción, solo por HTTPS (`Secure`). Como el navegador la envía por sí solo, toda operación que modifica datos exige además un token contra solicitudes falsificadas (CSRF) que únicamente la propia aplicación puede leer.

**Detalles de la implementación.** La cookie se llama `MM_SESSION`. Al iniciar sesión se cambia el identificador de la sesión, para que uno obtenido antes no sirva después. Cerrar sesión es `POST /api/v1/authentication/sign-out`. Las 8 horas se cuentan desde que se creó la sesión. El token CSRF llega en la cookie `XSRF-TOKEN` y vuelve en el encabezado `X-XSRF-TOKEN`, también al iniciar sesión y al registrarse. La cookie del dispositivo, `MM_DEVICE`, no es una sesión: solo elige el contador del freno del inicio de sesión (§5.4, D-144).

**Requisito de despliegue.** La aplicación web y la API se sirven desde el mismo origen —mismo dominio y puerto—, detrás de un proxy inverso. En desarrollo lo resuelve el proxy del servidor de Angular. En producción, ese proxy es una función de Cloudflare, y al backend solo se llega por ella (D-147).

**Tablas técnicas.** `spring_session` y `spring_session_attributes` las administra la librería de sesiones (§2.2). `spring_session.principal_name` guarda el correo de la persona —es lo que permite cerrar todas sus sesiones— y por eso admite 255 caracteres, igual que `users.email`. Las sesiones vencidas se borran solas cada minuto.

### 5.13 Lo que el backend implementa hoy

El paso 3 completó lo que el bloque 2 había dejado provisional: colaboradores, dar acceso, asignaciones con su empresa, sede y marca, bloqueos, caducidad y la frontera entre empresas en la base (§2.5). También alineó dos columnas con el diseño: V33 agregó `user_roles.revoked_by` (§5.7), y las asignaciones retiradas antes figuran retiradas por `SYSTEM`, porque no se sabe quién lo hizo; V34 quitó `employees.termination_reason`, que sigue como candidato (§5.10). Lo que sigue es lo que todavía no llega al diseño de este capítulo. Nada de esto cambia el diseño: es lo que falta para llegar a él.

| Tema | Hoy | Cuándo se completa |
|---|---|---|
| `permissions` y `role_permissions` | Existen desde el paso 3b (V45), con los permisos de los pasos 1 a 4 y la matriz de los siete roles del sistema para ellos; el paso 5 trajo los suyos (V46), y cada paso siguiente agrega los suyos con una migración de `iam`. Las operaciones de negocio y las lecturas de datos exigen un permiso; desde la segunda ronda de la auditoría, integrada en `develop` el 20/10/2026, también las cuentas, los accesos, los colaboradores y los roles, con sus permisos `seguridad.*` y ninguna anotación de rol (§5.8, D-95, D-117, D-122): la categoría (V56), los doce permisos `seguridad.*` con sus 22 marcas (V57) y los pares incompatibles (V58) (D-117 a D-119) | Cada paso trae sus permisos (§5.6). Seguridad, resuelto con la segunda ronda |
| Separación de funciones | Por los pares de `permission_incompatibilities`, en los roles propios y en las cuentas (D-119), desde la segunda ronda; hasta ella, por rol: una cuenta no tenía `ADMIN_ACCESOS` y `ADMIN_EMPRESA` a la vez (D-71) | Resuelto con la segunda ronda |
| Roles reservados | Por lo que incluyen (D-121), desde la segunda ronda; hasta ella, por su código: `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA` | Resuelto con la segunda ronda |
| Crear un rol a partir de otro | La copia exacta (D-124), desde la segunda ronda; hasta ella, un rol propio nacía sin permisos | Resuelto con la segunda ronda |
| Qué lee un rol global | Lo que su matriz le da (D-68): el superadministrador, todo; el administrador de accesos, los catálogos, la estructura de las empresas y sus colaboradores, que necesita para dar acceso, y nada del negocio | Resuelto con la matriz (§5.6) |
| Quién lee los colaboradores | La fila mínima —nombre, puesto, sede, si está activo— cualquier rol con `configuracion.ver_colaboradores`, que la semilla da a los siete, para los selectores y los nombres de autor; el documento, el teléfono, la foto y la exportación, solo con `configuracion.ver_datos_personales_colaboradores`: los dos roles globales y el administrador de la empresa (D-93, D-105) | Resuelto con la matriz |
| La marca de una asignación | Acota dónde se otorga un rol, pero todavía no acota lo que se lee: hoy ningún dato que se lee pertenece a una marca. Decidido (D-102): acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, en el código | Con el paso 5: los vehículos son el primer dato que pertenece a una marca |
| `roles.company_id` | Existe desde el paso 3b (V45): los roles propios de una empresa los crea el superadministrador, con el código único por empresa (D-98) | Resuelto |
| Registro público | Sigue pidiendo solo correo y contraseña y crea una cuenta sin colaborador, contra D-38. Viene apagado (D-32) | Fuera del alcance del paso 3 (D-83): sigue apagado, y el registro con perfil se construye antes de encenderlo en cualquier entorno, con la forma de listar sedes y tipos de documento sin sesión que fijen las convenciones de la API (§14.16), con la misma respuesta exista o no el correo y con el freno de D-144 (§5.2) |
| Regreso de un colaborador | ~~Reemplaza la fecha de ingreso y borra la de salida: el período anterior en esa empresa se pierde.~~ Guarda el período que cierra en `employee_tenure_periods` (§5.3), en la misma transacción, y la fila del colaborador ofrece «Ver historial de ingresos». Una corrección no deja el ingreso antes de la salida del período anterior (`EMPLOYEE_HIRE_DATE_BEFORE_PREVIOUS_TERMINATION`). Construido el 09/11/2026 en `feature/employee-tenure-history`: V124, backend `dff65113`, `6395606c`, `1136c4f3` y `897b8ef6`; frontend `f07df1aa`, `478047ea` y `f75d6470` | Decidido el 09/11/2026 (D-241) y construido el mismo día. Los períodos perdidos antes no se recuperan |
| Restablecer una contraseña | ~~La clave provisional se muestra a quien la restablece, que se la hace llegar a la persona (D-63).~~ Un enlace de un solo uso al correo de la persona, que pide ella misma en `/iam/forgot-password` o le envía un administrador, y que ella usa en `/iam/reset-password` (§5.4, §5.12), con `password_reset_tokens` (V126) y el módulo `notifications` (V125), que despacha el correo después de la transacción. Construido el 09/11/2026 en `feature/password-reset-link`: backend `d96fd798`, `f82caa6c`, `af2e9494`, `eda758d5` y `267ca405`; frontend `06ba1c4d` y `0e9dc7f7`. Dar acceso sigue mostrando la provisional | Decidido el 09/11/2026 (D-242, D-243) y construido el mismo día. Antes del primer correo real, el remitente, el SPF y el DKIM de `millennium.ky` y la cuenta de SendGrid (§2.7, §14) |
| Freno del inicio de sesión | Construido con el cierre de la revisión del 31/10/2026: `sign_in_throttles` (V87, `9f88a6b3`) y el freno por cuenta y por navegador, con la cookie `MM_DEVICE`, del inicio de sesión y del cambio de contraseña (`7b8583cc`), en `develop` (D-144) | Resuelto. Sus casos de la sonda en vivo pasaron el 01/11/2026 ([sondas del 01/11/2026](retroalimentacion/2026-11-01-sondas.md)) |
| Largo de la contraseña | Se valida de 8 caracteres a 72 bytes en UTF-8, como lo cuenta BCrypt, también la contraseña provisional del primer superadministrador (`17072d0a`, en `develop`; D-145) | Resuelto |
| Acceso solo por Cloudflare | `OriginCheckFilter` responde `404` a lo que llega sin el secreto de origen, salvo `/actuator/health`, y la documentación de la API está apagada en `prod` (`ea805b2c`, en `develop`; D-147). El filtro nace apagado | Construido y encendido en el entorno, verificado desde fuera el 01/11/2026 ([plan de cierre](plan-de-cierre-de-la-revision-2026-10-31.md), 15.4) |
| Escritura de los catálogos generales | Construida con los tipos de cambio el 02/11/2026: fuera de un rol de alcance sistema también se rechazan los permisos de `catalogos` que no son de la categoría `ver`, al marcarlos y al copiar un rol (`RoleDefinitionPolicy`, `a6ce58aa`), y V89 comprueba al migrar que ningún rol de empresa o de sede tenga uno marcado (D-150) | Resuelto |

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
| `phone` | VARCHAR(30) | Sí | Teléfono principal de la empresa | Es el que se copia como teléfono del emisor al crear una plantilla de orden de compra (§9.2, D-134) |
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
- Una plantilla de orden de compra imprime el emisor que copió de su empresa al crearse, y no se edita (§9.2, D-134): la orden no inventa otra razón social. Si algún día aparece otra razón social en el mismo país, será otra fila de esta tabla y habrá que revisar D-5.
- El número de documento tributario no se repite dentro del mismo país.
- El número se guarda normalizado, sin guiones, espacios ni puntos, para que la misma empresa no se registre dos veces escrita de formas distintas.
- El tipo de documento es del país de la empresa, está activo y aplica a empresas (§4.7). Si el tipo declara una longitud, el número la respeta, y si tiene dígito de control, el número lo cumple (D-75). Las dos cosas se verifican al registrar la empresa y cuando cambian el número o el tipo (D-91). Que sea del mismo país lo impide la base, porque la llave hacia el tipo incluye el país; lo demás lo valida la aplicación.
- **El país de una empresa no cambia; el tipo y el número de su documento los corrige el superadministrador** (D-85). Una empresa registrada con el documento equivocado se corrige en la misma fila, y la corrección se verifica de nuevo como al registrarla: un tipo del país de la empresa que aplique a empresas, la longitud, el dígito de control y que el número no se repita en el país. El país no se corrige: una empresa constituida en otro país es otra empresa, con sus sedes, su moneda y sus documentos (D-5).
- La razón social es la del registro público y no se ajusta para que se vea mejor: es la que se imprime en comprobantes y contratos.
- Una empresa que deja de operar se desactiva, nunca se borra: su historial de compras y contratos debe seguir siendo consultable.
- **A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo** (D-84). Bajo una empresa inactiva no se registran sedes, marcas, ubicaciones ni colaboradores, ni se otorgan roles; en una sede inactiva no se registran colaboradores ni ubicaciones propias, ni se otorgan roles; sobre una marca inactiva no se otorgan roles. Lo que ya apuntaba a ella sigue siendo válido y se sigue modificando mientras ese vínculo no cambie: un colaborador de una sede que cerró se corrige sin cambiarle la sede. Es el mismo criterio que una moneda o un tipo de documento desactivados (§4.1, §4.7). Reactivar algo es volver a colgarlo (D-94): bajo una empresa inactiva no se reactivan sedes, marcas ni ubicaciones, ni regresa un colaborador; primero se reactiva la empresa. Desactivar no espera a nada.
- El país de la empresa se propaga a sus sedes y vehículos sin que puedan contradecirlo.

### 6.2.1 `company_bank_accounts` — Cuentas bancarias

Las cuentas desde las que la empresa paga a sus proveedores. No son siempre la misma, y sin saber de cuál salió un pago no se puede conciliar con el extracto de esa cuenta (D-214). No es tesorería: no guarda saldos ni movimientos.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Dueña de la cuenta | No cambia de empresa |
| `name` | VARCHAR(60) | No | Nombre corto con que se reconoce: BBVA Dólares | Es lo que se elige al registrar un pago. Único en la empresa |
| `bank_name` | VARCHAR(100) | No | El banco | Dice a qué extracto mirar |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda de la cuenta | Fija la moneda de cada pago que sale de ella |
| `account_number` | VARCHAR(34) | No | Número de la cuenta | Único en la empresa. Solo quien gestiona las cuentas lee el número completo y busca u ordena por él; los demás, y el registro de un pago, ven sus últimos dígitos (D-214) |
| `is_active` | BOOLEAN | No | Si sigue en uso | Desactivarla no borra la fila ni toca los pagos que salieron de ella |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Último cambio | Corregirla exige el `updated_at` que se leyó: uno anterior se rechaza sin guardar (D-137) |

**Relaciones.** Apunta a `companies` y `currencies`. Le apuntan los pagos que salieron de ella (§10.1).

**Ejemplo**

| name | bank_name | moneda | account_number | is_active |
|---|---|---|---|---|
| BBVA Dólares | BBVA | USD | 0011-0000-0000007788 | sí |
| BCP Soles | BCP | PEN | 191-0000000-0-45 | sí |
| Interbank Soles | Interbank | PEN | 200-0000000000 | no |

**Reglas de negocio**

- Varias por empresa, y por moneda.
- No se borran: la que ya no se usa queda inactiva, y deja de ofrecerse al registrar un pago.
- Con pagos registrados, su número y su moneda ya no cambian: la cuenta equivocada se desactiva y se registra otra. El nombre y el banco se pueden corregir.
- Las gestionan quienes tienen `configuracion.crear_cuentas_bancarias`, `configuracion.editar_cuentas_bancarias` y `configuracion.desactivar_cuentas_bancarias`, en una pestaña de la ficha de la empresa. Verlas pide `configuracion.ver_estructura`, y el número completo solo llega a quien las gestiona.

### 6.3 `commercial_brands` — Marcas comerciales
El nombre bajo el cual la empresa coloca sus unidades en el mercado: Lizzo para autos, Lima Bikes para motos y trimotos. Una empresa puede tener varias y se espera que aparezcan más.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa dueña de la marca | Una marca no existe por sí sola: pertenece a la razón social que factura |
| `code` | VARCHAR(20) | No | Código corto e invariable: `LIZZO`, `LIMABIKES` | Es lo que usan los reportes y las integraciones. El nombre visible puede cambiar; este no |
| `name` | VARCHAR(100) | No | El nombre como se muestra | — |
| `is_active` | BOOLEAN | No | Si se siguen colocando unidades bajo esta marca | Permite retirar una marca sin tocar los vehículos ya vendidos con ella |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | — |

**Relaciones.** Apunta a `companies`. Le apuntan `user_roles`, cuando un acceso se limita a una marca, y `purchase_order_templates`, cuando una plantilla es de una marca comercial. Más adelante le apuntarán los vehículos, para registrar bajo qué marca se coloca cada unidad.

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
| `has_installations` | BOOLEAN | No | Si alguna unidad ya lo tuvo instalado. Por defecto `FALSE` | Lo marca `fleet` en la primera instalación, como `vehicle_models.has_units` (D-107), y desde entonces su código queda fijo (D-48, D-185): así `vehiclecatalog` no cuenta las filas de una tabla de otro contexto |
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
| `transfer_reason` | VARCHAR(30) | Sí | Por qué se traspasó: `fin_contrato`, `venta_directa`, `otro` | Un traspaso por fin de contrato y una venta directa tienen tratamientos contables distintos |
| `transfer_owner_name` | VARCHAR(150) | Sí | A quién se traspasó, como lo dice el traspaso | El exportable muestra el propietario vigente (guía, columna 50), y todavía no hay clientes a quienes apuntar. Las tres columnas van juntas: las tres llenas o las tres vacías (D-223) |
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
- Una unidad con una línea de factura vigente no cambia de modelo ni de año: lo que dice el comprobante y lo que dice la unidad no pueden separarse (D-160).
- Una unidad nueva no se registra con un modelo inactivo, ni con uno cuya marca o tipo de vehículo estén inactivos; la que ya lo tiene lo conserva y se sigue editando (D-108, como D-84). Al registrarla, `fleet` le avisa a `vehiclecatalog` que el modelo ya tiene unidades, en la misma operación, y desde entonces el tipo del modelo queda fijo (§7.3, D-107).
- La unidad no se borra nunca. Si una nota de crédito anula la compra, se marca como anulada; si se traspasa, se registra la fecha.
- **El propietario vigente se deduce**: si no hay fecha de traspaso, es la empresa; si la hay, es `transfer_owner_name`. No hay un campo que diga si sigue siendo de la empresa, para que no pueda contradecir a la fecha.
- **Solo se traspasa una unidad en En traspaso o en Inactivo** (D-223, enmendada por D-239): una unidad que sigue en la flota no puede ser de otro. El camino a En traspaso dice que el contrato terminó o que la unidad se vendió directo; el traspaso, el día en que el notario lo firmó, que puede ser después. La unidad pasa a Inactivo por «Traspaso inscrito» solo con el traspaso registrado (§8.3). Una que llegó a Inactivo por otro camino lo registra ahí. Lo registrado se corrige con justificación (§13.1). Registrarlo en otro estado responde `422 VEHICLE_TRANSFER_NOT_ALLOWED`, que la pantalla dice «Solo se traspasa una unidad que salió de la flota: En traspaso o Inactivo». La ficha ofrece «Registrar traspaso» donde `stock.recuperar` actúa sobre la unidad y el estado lo admite (V123, backend `9f9bd093`, frontend `9dd5c398`).

**No se generan correlativos nuevos.** El `N°` del Excel se conserva solo en las unidades migradas, como referencia. El orden de registro lo da la fecha de alta y la identificación, el VIN y la placa.

### 8.2 `vehicle_statuses` — Estados del vehículo

Los nueve estados del flujograma, los dos que sumó el leasing (D-219, D-220), En traspaso (D-239) y los que se agreguen mientras se refinan los procesos.

**Millennium coloca sus unidades en leasing.** Firmar un contrato no vende la unidad: sigue siendo de la empresa hasta su traspaso (D-20), puede volver por incumplimiento y entregarse a otra persona. Por eso el estado se llama «Contrato firmado» (D-218) y, desde V130, también su código: `CONTRATO_FIRMADO`, que hasta entonces era `VENDIDO`, y el de su camino, `FIRMA_CONTRATO`, que era `VENTA` (D-251). La venta real es «Venta directa» (`VENTA_DIRECTA`), que lleva a En traspaso (D-239). Un estado existe si la unidad tiene salidas distintas o se cuenta distinto; si solo cambia el porqué, es un motivo (§8.3).

**Por qué es una tabla y no una lista fija.** Porque la lista va a crecer. Si estuviera escrita dentro del programa, agregar un estado obligaría a modificar el sistema y volver a instalarlo; siendo una tabla, es cargar una fila.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(30) UNIQUE | No | Código corto: `DISPONIBLE`, `ACTIVO`, `CUSTODIA` | Es el que usan las reglas del sistema |
| `name` | VARCHAR(60) | No | Nombre visible | — |
| `category` | VARCHAR(20) | No | Familia a la que pertenece: comercial, productivo, recuperación, terminal | El flujograma agrupa los estados por familia y así se calculan los indicadores sin enumerar estados uno por uno |
| `generates_revenue` | BOOLEAN | No | Si la unidad genera ingresos estando en este estado | Es lo que permite calcular el porcentaje de activación de flota, que es el indicador principal del tablero |
| `is_initial` | BOOLEAN | No | Si una unidad recién registrada nace en este estado | Hoy es "En habilitación". Marcarlo evita que el estado inicial quede escrito en el programa |
| `requires_enablement` | BOOLEAN | No | Si entrar a mano a este estado exige la habilitación completa. Por defecto `FALSE` | La semilla lo marca en Disponible, que el flujograma define como «unidades listas para entrega a cliente». Escrito como dato, la puerta no depende del camino por el que se llega (D-164). En Disponible queda fijo: no se desmarca (D-170). La semilla también lo marca en Vendido, donde se puede desmarcar (D-184) |
| `is_active` | BOOLEAN | No | Si se sigue usando | Permite retirar un estado sin tocar el historial de las unidades que pasaron por él |

**Relaciones.** Le apuntan `vehicles` (estado actual), `status_change_reasons` como origen y destino, y `vehicle_status_history`.

**Ejemplo**

| id | code | name | category | genera ingresos |
|---|---|---|---|---|
| 1 | `HABILITACION` | En habilitación | comercial | No |
| 2 | `DISPONIBLE` | Disponible | comercial | No |
| 3 | `CONTRATO_FIRMADO` (~~`VENDIDO`~~ hasta V130, D-251) | Contrato firmado | comercial | No |
| 4 | `ACTIVO` | Activo | productivo | Sí |
| 5 | `MECANICA` | Mecánica | recuperación | No |
| 6 | `SINIESTRO` | Siniestro | recuperación | No |
| 7 | `CUSTODIA` | Custodia | recuperación | No |
| 8 | `PAP` | Puesta a punto | recuperación | No |
| 9 | `INACTIVO` | Inactivo | terminal | No |
| 10 | `ROBADO` | Robado | terminal | No |
| 11 | `PERDIDA_TOTAL` | Pérdida total | terminal | No |
| 12 | `EN_TRASPASO` | En traspaso | terminal | No |

**La puerta de Disponible** (D-164). Un cambio de estado a mano hacia un estado con `requires_enablement` —por «Habilitación completa» o por «Reacondicionada»— exige que el estado de habilitación sea *Listo para entrega* (§11.9); si no lo es, se rechaza nombrando lo que falta. El regreso automático de Contrato firmado a las 48 h no la aplica mientras ese estado la exige: deshace una entrega que no ocurrió, y la unidad no pudo perder un requisito. Si se desmarca (D-184), el regreso deja ahí la unidad a la que le falta algo. Salir de Puesta a punto hacia un estado marcado exige además la revisión física de la unidad (D-217). Ningún otro camino mueve sola una unidad a un estado marcado, y el estado inicial no se marca: así la puerta no se apaga desde el catálogo (D-170). Pasa la puerta la unidad a la que no le falta nada —sin ninguna de las filas 1 a 4 de §11.9—, esté *Listo para entrega* o *Entregado*, así que una unidad reacondicionada vuelve a Disponible (D-183). **La puerta se sostiene** (D-184): en un estado marcado no se retira un requisito sin reemplazo —el GPS, la pantalla que corresponde, las placas físicas o el ingreso—; un equipo se reemplaza, y si no hay repuesto, la unidad sale primero de Disponible por el camino «Pierde un requisito», hacia En habilitación, que trae la semilla con `stock.habilitar`.

**En traspaso** (D-239). Es el tiempo en que la notaría inscribe el traspaso de una unidad cuyo contrato terminó o que se vendió directo. Es de familia `terminal` y sin ingresos: no cuenta en el porcentaje de activación ni como paralizada (§10.5). Su código queda fijo desde la semilla, como el de `INACTIVO`, porque lo leen las reglas del traspaso. Ni En traspaso ni Inactivo se desactivan: por ellos una unidad sale de la flota y se traspasa, y desactivarlos responde `422 VEHICLESTATUS_TRANSFER_RETIREMENT_NOT_ALLOWED` (`VehicleStatus.changeStatus`). Hasta V123, Inactivo sí se podía desactivar. Se entra desde Activo, por «Contrato completado» o «Venta directa», y desde Disponible, por «Venta directa»; se sale solo a Inactivo, por «Traspaso inscrito», con el traspaso ya registrado (§8.1, §8.3). Su tolerancia es lo que tarda la notaría, con «avisar» (D-240); V123 siembra sus caminos sin tolerancia, hasta que el negocio la fije. Lo siembra V123, de `fleet` (backend `9f9bd093`): activo, no inicial y sin la puerta de la habilitación. Si un estado `EN_TRASPASO` ya existía, V123 lo conserva solo si es lo que leen las reglas, y si no, se detiene nombrándolo; también se detiene si Inactivo estaba desactivado.

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
| *(alta)* | En habilitación, Disponible o Activo | Migración inicial (D-202) | — | — |
| En habilitación | Disponible | Habilitación completa | — | — |
| Disponible | En habilitación | Pierde un requisito (D-184) | — | — |
| Disponible | Contrato firmado | Firma de contrato | 48 h | Volver a Disponible |
| Disponible | En traspaso | Venta directa (D-239) | Por fijar (D-240) | Avisar, cuando se fije |
| Contrato firmado | Disponible | Regreso automático por vencimiento *(automático)* | — | — |
| Contrato firmado | Activo | Entrega al cliente | — | — |
| Activo | Mecánica | Mantenimiento preventivo | 72 h | Avisar |
| Activo | Mecánica | Falla · Alerta de sistema · Kilometraje · Tiempo | — | — |
| Activo | Siniestro | Choque | 480 h | Avisar |
| Activo | Siniestro | Accidente · Robo frustrado · Daños a terceros | — | — |
| Activo | Custodia | Morosidad | 720 h | Avisar |
| Activo | Custodia | Incautación · Comisaría · Depósito · Devolución voluntaria · Otros incumplimientos | — | — |
| Activo | Robado | Robo · Autorrobo · Fraude | — | — |
| Activo | En traspaso | Contrato completado · Venta directa (D-239) | Por fijar (D-240) | Avisar, cuando se fije |
| En traspaso | Inactivo | Traspaso inscrito (D-239) | — | — |
| Mecánica | Activo | Mantenimiento finalizado | — | — |
| Mecánica | Puesta a punto | No asumida | — | — |
| Siniestro | Activo | Reparación finalizada | — | — |
| Siniestro | Puesta a punto | No asumida | — | — |
| Siniestro | Pérdida total | Pérdida total | — | — |
| Custodia | Activo | Cliente regulariza | — | — |
| Custodia | Puesta a punto | Resolución de contrato | — | — |
| Custodia | Robado | Robo · Autorrobo · Fraude | — | — |
| Robado | Puesta a punto | Recuperado | — | — |
| Robado | Inactivo | Baja definitiva | — | — |
| Pérdida total | Puesta a punto | Recupera valor | — | — |
| Pérdida total | Inactivo | Baja definitiva | — | — |
| Puesta a punto | Disponible | Reacondicionada | — | — |

Las filas de Mecánica muestran para qué sirve la tabla: mismo camino, distinta expectativa según por qué entró la unidad. La fila de Contrato firmado a Disponible es el motivo automático que el sistema creó para el regreso de la venta: nadie la elige. Los caminos de Activo y de Disponible a Inactivo por «Contrato completado» y «Venta directa» quedaron cerrados por D-239: pasan por En traspaso.

**Reglas de negocio**

- Un cambio de estado que no figure aquí se rechaza. No es una advertencia: la unidad no se mueve.
- Todo cambio registra fecha, hora, usuario y motivo. Es la exigencia del flujograma y no admite excepción.
- El tiempo límite se cuenta desde el momento del cambio, y la unidad aparece en la lista de vencidas al pasarse.
- La única transición que ocurre sola es la que tiene configurado un estado de destino al vencerse. Todas las demás las hace una persona.
- **El regreso automático tiene su propio motivo** (D-110). Al guardar un camino de A a B que, al vencerse, mueve sola la unidad a C, el sistema asegura en la misma operación un motivo automático del camino de B a C: código `REGRESO_AUTOMATICO`, nombre «Regreso automático por vencimiento», abierto, sin permiso propio, sin tolerancia y con `is_automatic` verdadero. Si ese camino ya tiene uno, lo reutiliza: hay uno solo por camino. Con el ejemplo de arriba, la Venta, de Disponible a Vendido, crea el motivo automático de Vendido a Disponible.
- Un motivo automático no se ofrece a las personas: no figura entre los caminos que se pueden recorrer al mover una unidad, y elegirlo se rechaza. Tampoco se edita ni se cierra por la API: lo mantiene el sistema.
- Un motivo no se borra: se desactiva, para que el historial siga siendo explicable.
- **Salir de Puesta a punto hacia un estado con `requires_enablement` pide la revisión física** (D-217): quien mueve la unidad confirma que revisó el GPS, la pantalla si corresponde, las placas y el documento vehicular, porque lo que la puerta lee se registró antes de que la unidad saliera. Sin esa confirmación el cambio se rechaza, y la regla la decide el sistema por el estado de origen, no un motivo del catálogo: no se puede apagar.
- **Salir de la flota pasa por En traspaso** (D-239). V123 cierra Activo → Inactivo por «Contrato completado» y «Venta directa» y Disponible → Inactivo por «Venta directa», y abre Activo → En traspaso por los mismos dos motivos, Disponible → En traspaso por «Venta directa» y En traspaso → Inactivo por «Traspaso inscrito» (`TRASPASO_INSCRITO`), todos con `stock.recuperar`. Los motivos cerrados se desactivan, no se borran: el historial que los recorrió sigue explicado. Los cuatro caminos nuevos nacen sin tolerancia (D-240). El último camino exige el traspaso registrado (§8.1): sin él responde `422 VEHICLE_TRANSFER_REQUIRED`. La regla es del sistema, como la revisión física: la deciden el estado de origen y el de destino (`TransferExitPolicy`), no un motivo del catálogo, así que no se apaga. Un cambio de estado se revisa en este orden: el camino, el permiso, la unidad anulada, la revisión física, el traspaso y la puerta de la habilitación; el traspaso y la puerta los revisa la propia unidad al moverse (`Vehicle.transitionTo`). La ficha muestra deshabilitado, mientras falta el traspaso, el camino que lo exige.
- **Las tolerancias las fija una migración** (D-240), con los valores que entrega el negocio, para que toda base nueva nazca con ellas; se piden en la especificación de tolerancias del 09/11/2026. Después las ajusta el superadministrador en Catálogos › Estados y motivos. Un cambio vale para los movimientos siguientes: el vencimiento se calcula al registrar cada movimiento (`due_at`, §8.4), y el historial solo se inserta.

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
| `physically_verified` | BOOLEAN | No | Si quien movió la unidad confirmó que la revisó físicamente. Por defecto `FALSE` | Solo lo pide salir de Puesta a punto hacia un estado con `requires_enablement` (D-217). Quién y cuándo ya están en la fila |

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
| `tax_id` | VARCHAR(20) | No | Número del documento, normalizado como todo identificador externo (§2.2): sin guiones, espacios ni puntos, tampoco los separadores Unicode | Va en la factura y es lo que revisa el auditor. Normalizado, el mismo proveedor escrito de dos formas no se registra dos veces; V76 normaliza los ya guardados y detiene el despliegue si dos de una empresa coinciden (D-134) |
| `legal_name` | VARCHAR(255) | No | Razón social | Es la que debe figurar en la orden de compra que se le envía |
| `trade_name` | VARCHAR(150) | Sí | Nombre comercial | El nombre corto con que lo conoce la operación |
| `address_id` | BIGINT FK → `addresses` | Sí | Domicilio | Se imprime en la orden de compra. Es una fila propia: dos proveedores no apuntan a la misma (`UNIQUE`, V77; §4.6) |
| `is_active` | BOOLEAN | No | Si se le sigue comprando | Permite retirarlo sin tocar las facturas antiguas |
| `is_dealer` | BOOLEAN | No | Si es del canal de dealers, donde el cliente paga su inicial al concesionario. Por defecto `FALSE` | Los pagos a sus facturas detallan unidades, y lo que falta de una unidad saldada es la inicial del cliente (§10.3, §10.5, D-197). No se marca mientras alguna de sus órdenes tenga pagos aplicados, porque la orden de un dealer no recibe pagos (D-211) |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | — |

**Relaciones.** Apunta a `companies`, `tax_id_types` y `addresses`. Le apuntan `supplier_contacts`, las órdenes de compra, las facturas y las notas de crédito.

**Ejemplo**

| id | company_id | tax_id | legal_name | trade_name |
|---|---|---|---|---|
| 1 | 1 · Millennium Perú | 20100128056 | Crosland Automotriz S.A.C. | Crossland |
| 2 | 1 · Millennium Perú | 20605412344 | Derco Perú S.A. | Derco |

**Reglas de negocio**

- El documento tributario no se repite dentro de la misma empresa.
- Un proveedor nuevo se crea antes de poder registrarle una compra. No se crea al vuelo durante una carga.
- No confundir con la marca del vehículo: Crossland vende motos Bajaj. El proveedor es a quién se le compra; la marca, qué se compró.
- El domicilio puede faltar al registrarlo y se agrega después. Si ya está, se corrige en su fila o se muda a una fila nueva; la anterior se conserva. El documento tributario no se corrige en esta entrega.
- Hoy solo se registran proveedores locales: su domicilio está en el país de la empresa que le compra, como el de la empresa y sus sedes, y su documento tributario es un tipo de ese país (§4.6, §4.7, D-133). Un proveedor del exterior es candidato (§9.8).
- Un proveedor es de todas las marcas de su empresa: lo crean, lo editan y lo activan o desactivan solo las asignaciones de toda la empresa y sin marca (§2.5, D-134). Una de sede o de marca lo lee con `compras.ver` y lo elige al emitir.
- La lista de proveedores se exporta a una hoja de cálculo con el alcance de su lectura, `compras.ver`, sin los contactos (D-116, D-134).

### 9.1.1 `supplier_contacts` — Contactos del proveedor

La persona a la que la empresa llama o escribe. Un proveedor tiene varias, o ninguna. No son los textos de atención y soporte de la plantilla de orden de compra: esos son de la empresa que compra, y hay uno de cada uno por plantilla.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `supplier_id` | BIGINT FK → `suppliers` | No | A qué proveedor pertenece | No cambia de proveedor |
| `company_id` | BIGINT FK → `companies` | No | La empresa del proveedor, copiada | Junto con `supplier_id` apunta a `suppliers (id, company_id)`, para que no pueda decir otra empresa |
| `name` | VARCHAR(150) | No | La persona | Sin esto no hay a quién llamar |
| `position` | VARCHAR(80) | Sí | El puesto | Puede faltar |
| `phone` | VARCHAR(30) | Sí | Teléfono | Hace falta este o el correo |
| `email` | VARCHAR(255) | Sí | Correo, en minúsculas y sin espacios | Hace falta este o el teléfono |
| `is_active` | BOOLEAN | No | Si sigue siendo contacto | Retirarlo no borra la fila |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | Corregir un contacto exige el `updated_at` que se leyó: uno anterior se rechaza sin guardar (D-137) |

**Relaciones.** Apunta a `suppliers` por la pareja `(supplier_id, company_id)`.

**Ejemplo**

| supplier | name | position | phone | email | is_active |
|---|---|---|---|---|---|
| Crosland | Ana Ruiz | Ventas | 999111222 | — | sí |
| Crosland | Mesa de facturación | Facturación | — | facturas@crosland.example | sí |
| Crosland | Pedro Solís | — | 999000111 | — | no |

**Reglas de negocio**

- Varios por proveedor, incluidos cero.
- Hace falta teléfono o correo.
- No se borran: quien ya no es contacto queda inactivo.
- No son los textos de atención y soporte de la plantilla (§9.2).
- Se mantienen uno por uno en la ficha del proveedor, por su propia ruta, con `compras.editar_proveedores`: agregarlos, corregirlos, desactivarlos y reactivarlos no reescribe al proveedor ni a sus otros contactos. El alta del proveedor no los lleva (D-137).
- A un proveedor inactivo no se le agrega un contacto ni se le reactiva uno retirado (D-108); los que tiene se siguen corrigiendo y desactivando (D-94).
- La lista de contactos de un proveedor no se exporta.

### 9.2 `purchase_orders` — Órdenes de compra

Lo que se le pide al proveedor. No bloquea nada: una unidad puede registrarse sin que exista una orden.

Cada orden se emite con una **plantilla**, que decide lo que se imprime —quién emite, a quién va dirigida, a quién consultar, qué instrucciones y qué firma lleva— y la serie con que se numera. Las plantillas y los tipos de orden se describen al final de esta sección.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que compra | Es la empresa dueña de la plantilla con que se emite |
| `supplier_id` | BIGINT FK → `suppliers` | No | A quién se le pide | — |
| `supplier_contact_id` | BIGINT FK → `supplier_contacts` | Sí | Persona del recuadro Proveedor | Opcional. Tiene que ser un contacto de ese proveedor; uno nuevo tiene que estar activo (D-132, D-84) |
| `purchase_order_template_id` | BIGINT FK → `purchase_order_templates` | No | Con qué plantilla se emite | De ella salen los datos que se imprimen y la serie de numeración |
| `has_discount_column` | BOOLEAN | No | Si esta orden incluye la columna de porcentaje de descuento | Se decide al crear o editar el borrador; no vive en la plantilla (D-132) |
| `series` | VARCHAR(20) | No | Serie de la plantilla al crear la orden: `OC-LBT` | Se copia y no se vuelve a leer de la plantilla: si la serie se corrigiera, las órdenes ya numeradas no cambiarían de número |
| `sequence_year` | SMALLINT | No | Año del número: el de la fecha de creación, en la hora local del país de la empresa | Se fija al asignar el número y no se recalcula. Un borrador creado el 30 de diciembre y enviado en enero conserva el número del año en que se creó |
| `sequence_number` | INTEGER | No | Correlativo dentro de la serie y el año: 1, 2, 3… | Vuelve a 1 cada año en cada serie |
| `order_number` | TEXT, columna generada | No | Número completo: serie, año y correlativo con cuatro dígitos como mínimo. `OC-LBT-2026-0001` | Es el que aparece en el PDF que se envía al proveedor y con el que él responde. Se calcula de las tres columnas anteriores, no se escribe |
| `issue_date` | DATE | No | Fecha de emisión | — |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda del pedido | En la API viaja por su código ISO (`PEN`), como la moneda de la empresa (D-13, D-138) |
| `payment_terms` | VARCHAR(50) | Sí | Condición de pago acordada: contado, crédito 30 | Se imprime en la orden, bajo la fecha y el número, y es lo que después se reclama |
| `purpose` | VARCHAR(150) | Sí | Concepto del pedido: requerimiento de stock | — |
| `status` | VARCHAR(20) | No | En qué va: `BORRADOR`, `ENVIADA`, `CUMPLIDA`, `ANULADA` | Códigos guardados; la pantalla los traduce (D-132) |
| `sent_at` | TIMESTAMPTZ | Sí | Cuándo se envió al proveedor | Distingue una orden preparada de una ya comprometida |
| `pdf_path` | TEXT | Sí | Archivo generado | Es el documento que se le envía; conviene conservar el que efectivamente se mandó y no volver a generarlo |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la emitió y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Cuándo cambió la orden por última vez y quién: una corrección del borrador, el envío, el cumplimiento o la anulación | Guardar el borrador exige el `updated_at` con que se abrió: si alguien lo cambió después, se rechaza sin pisar sus cambios (D-138). V85 lo agregó; las órdenes anteriores tomaron su creación como último cambio |

#### `purchase_order_items` — Líneas de la orden

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `purchase_order_id` | BIGINT FK → `purchase_orders` | No | A qué orden pertenece | — |
| `company_id` | BIGINT FK → `companies` | No | Empresa de la orden, copiada | La línea acumula volumen y la frontera entre empresas la lee directo, igual que D-111 (D-132) |
| `model_id` | BIGINT FK → `vehicle_models` | Sí | Qué modelo se pide, cuando la línea es de vehículos | Al ordenar todavía no se sabe qué VIN van a llegar: se pide por modelo y cantidad. Vacío en las líneas que no son vehículos |
| `model_year` | SMALLINT | Sí | Año del modelo que se pide, cuando la línea es de vehículos | Se elige con la marca y el modelo y se imprime con ellos. Se valida entre 2017 y el año siguiente al actual, como el de la unidad (§11.1); una línea de concepto no lo tiene. Las líneas guardadas antes de pedirlo quedan vacías hasta que su borrador se guarde otra vez (D-135) |
| `concept` | VARCHAR(150) | Sí | Qué se pide, cuando la línea no es un vehículo: un mantenimiento, repuestos, un servicio | Las órdenes de taller no compran modelos. Es la misma solución que las líneas de factura (D-22) |
| `quantity` | INTEGER | No | Cuántas unidades | Es contra esto que se calcula lo pendiente de llegar |
| `unit_price` | NUMERIC(14,2) | No | Precio acordado por unidad | Es el precio comprometido, que puede diferir del que después facture el proveedor |
| `discount_rate` | NUMERIC(5,4) | Sí | Porcentaje de descuento de la línea, de 0 a 1 | Obligatoria en una línea activa si la orden incluye la columna; nula si no la incluye (D-132) |
| `is_active` | BOOLEAN | No | Si la línea sigue en la orden | Una línea retirada del borrador queda inactiva y no entra al PDF ni a los totales; no se borra (D-132) |

Cada línea lleva un modelo con su año o un concepto, nunca los dos ni ninguno. El total de la línea se calcula, no se escribe: cantidad × precio × (1 − tasa), con tasa 0 si la columna de descuento está apagada, redondeado a los decimales de la moneda de la orden, mitad hacia arriba; el subtotal es la suma de las líneas así redondeadas, y la pantalla, la API y el PDF muestran los mismos montos (D-135). En el PDF, una línea de vehículo imprime la marca, el nombre del modelo y su año, sin otra palabra delante —«JETOUR DASHING 1.5T 6DCT CONFORT ACP, AÑO 2026»—; una de concepto imprime `concept`. La API devuelve ese mismo texto en cada línea.

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
- **Una vez enviada, su PDF es el documento de la orden.** Se conserva en `pdf_path`, y editar la plantilla después no altera las órdenes ya enviadas. Es una sola copia fiel, con el teléfono de Atención, para todo el que lee la orden: es la que recibió el proveedor, sea quien sea quien pulsó «Enviar». El borrador lleva ese teléfono solo para quien ve los datos personales de los colaboradores (D-105, D-134).
- **El PDF tiene el formato con que la empresa emite sus órdenes**: el logo y el título, el emisor con la fecha, el número y la condición de pago, los recuadros PROVEEDOR y ATENCION, la tabla de líneas con sus filas vacías y la columna TOTAL sombreada, los totales, los comentarios junto a la firma y a quién consultar. De una orden a otra solo cambian los datos y la columna de descuento. Se imprimen todas las líneas, en las páginas A4 que hagan falta: la cabecera de la tabla se repite en cada una y el pie de cada página nombra la orden y la página (D-135).
- Una orden se emite solo con una plantilla activa cuya empresa, marca —si la tiene— y tipo de orden están activos (D-84).
- Un borrador conserva el proveedor, el contacto, la moneda y el modelo de cada línea que ya tenía aunque se hayan desactivado después; uno nuevo debe estar activo (D-84, D-108).
- Emitir una orden exige leer su plantilla y su proveedor con `compras.ver` (§5.6). La lista de órdenes y la de plantillas todavía no se exportan: llegan con el paso de reportes (D-116, D-134).
- La columna de descuento se decide en cada orden (`has_discount_column`), no en la plantilla. Si está apagada, ninguna línea activa lleva tasa; si está encendida, cada línea activa exige `discount_rate` (D-132).
- El contacto del recuadro Proveedor es opcional y de ese proveedor. Cero contactos es válido (D-132).
- Una línea que se quita del borrador queda inactiva y no entra al PDF ni a los totales; no se borra (D-132).
- Dos ediciones del mismo borrador no se pisan: el guardado lleva el `updated_at` con que se abrió la orden, y uno que ya no es el de la orden se rechaza sin guardar nada (D-138).
- El estado guardado es `BORRADOR`, `ENVIADA`, `CUMPLIDA` o `ANULADA`. Cumplir solo sale de enviada; anular no sale de cumplida (D-132).
- Una orden con facturas vigentes no se anula, y cumplirla es manual aunque tenga facturas vinculadas: la orden muestra si ya está todo facturado (D-162).
- Una orden con pagos aplicados no se anula ni se cumple: sus pagos se anulan o pasan a las facturas que se vinculan a ella (D-198, D-211).

#### Cómo se numera una orden

- Cada plantilla tiene su serie, y cada serie su correlativo, que vuelve a 1 cada año: `OC-LBT-2026-0001`, `OC-LBT-2026-0002`… y en enero, `OC-LBT-2027-0001`. El año en el número dice de un vistazo de cuándo es la orden y lo mantiene corto.
- El número se asigna al crear la orden, aunque sea un borrador, y **no se reutiliza nunca**. Como nada se borra —un borrador que no sigue se anula—, cada número corresponde a una orden que se puede consultar, vigente o anulada: no quedan huecos sin explicación.
- El año es el de la fecha de creación **en la hora local del país de la empresa** (`countries.default_timezone`), no en la hora universal. Una orden creada en Lima el 31 de diciembre a las 20:00 es del año que termina.
- El número siguiente es el mayor de esa serie en ese año más uno. Se calcula dentro de la operación que crea la orden, con la plantilla bloqueada, para que dos órdenes simultáneas no tomen el mismo número. No se guarda un contador: sería un derivado que puede desincronizarse.
- La combinación de empresa, serie, año y correlativo no se repite: es la garantía final de que no haya dos órdenes con el mismo número.
- La orden de compra no es un comprobante de pago: su serie y su correlativo no los regula la autoridad tributaria, así que siguen la forma que ya usa la operación.
- **La forma se mantiene** frente a la de la muestra de la empresa (`2026-03-0001`): la serie distingue la marca y la línea de negocio, que la muestra no dice; el mes no agrega nada que la fecha impresa no diga; y una forma distinta de la del sistema anterior evita que un número nuevo coincida con uno ya emitido allí (D-140).

#### `purchase_order_types` — Tipos de orden

Clasifica la orden según lo que se compra. Junto con la marca comercial, decide qué plantilla corresponde.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | La empresa que usa el tipo | Cada empresa define los suyos |
| `code` | VARCHAR(20) | No | Código corto: `VEHICULOS`, `TALLER`, `OTROS`. Se corrige mientras ninguna plantilla usa el tipo y queda fijo en cuanto una lo usa | Lo usan los reportes y los filtros. El nombre visible puede cambiar siempre; el código, solo para corregirlo antes de que una plantilla lo tome, como el código del fondo (§12.1) |
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
- El código no se repite dentro de la empresa. Se corrige mientras ninguna plantilla usa el tipo y queda fijo en cuanto una lo usa.
- Un tipo no se borra: se desactiva.

#### `purchase_order_templates` — Plantillas de orden de compra

El formato con que se emite una orden. Los campos son siempre los mismos; lo que cambia de una plantilla a otra son sus valores, según la marca comercial y el tipo de orden (D-35).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa dueña de la plantilla | Es la que compra con las órdenes emitidas con ella, y de la que se copian los datos del emisor al crearla |
| `commercial_brand_id` | BIGINT FK → `commercial_brands` | Sí | Marca comercial: Lizzo, Lima Bikes. **Vacío = plantilla general** | Cada línea puede llevar su propio formato. Tiene que ser una marca de la misma empresa |
| `purchase_order_type_id` | BIGINT FK → `purchase_order_types` | No | Tipo de orden: vehículos, taller, otros | — |
| `name` | VARCHAR(100) | No | Nombre con que se elige al emitir: Lima Bikes · Taller | — |
| `series` | VARCHAR(20) | No | Prefijo de numeración: `OC-LBT`. En mayúsculas y sin espacios | Separa la numeración de cada plantilla: cada serie lleva su propio correlativo |
| `issuer_legal_name` | VARCHAR(255) | No | Razón social que se imprime como emisora | Se copia la de la empresa al crear la plantilla y no se edita (D-134) |
| `issuer_tax_id_type_id` | BIGINT FK → `tax_id_types` | No | Tipo de documento del emisor: RUC | Se copia el de la empresa al crear la plantilla |
| `issuer_tax_id` | VARCHAR(20) | No | Número de documento del emisor, sin guiones, espacios ni puntos | Se copia el de la empresa al crear la plantilla. Ya se verificó al registrar la empresa, así que la plantilla no lo vuelve a verificar |
| `issuer_phone` | VARCHAR(30) | Sí | Teléfono del emisor | Se copia el de la empresa (`companies.phone`) al crear la plantilla |
| `issuer_address_id` | BIGINT FK → `addresses` | Sí | Dirección que se imprime; el distrito o ciudad sale de su división territorial | Fila propia, copia fiel del domicilio fiscal de la empresa al crear la plantilla: la misma división, aunque ya esté inactiva, porque es el mismo lugar y no una mudanza. Las direcciones no se comparten entre entidades (§4.6) |
| `attention_name` | VARCHAR(150) | Sí | Sección "Atención": quién recibe la orden dentro de la empresa | Se copia del colaborador elegido (`attention_employee_id`). No se escribe a mano |
| `attention_employee_id` | BIGINT FK → `employees` | Sí | Colaborador de la misma empresa que figura en Atención | Tiene que trabajar en la empresa de la plantilla. Uno nuevo tiene que estar activo |
| `attention_position` | VARCHAR(80) | Sí | Puesto de quien recibe | Se copia del colaborador elegido |
| `attention_email` | VARCHAR(255) | Sí | Correo de quien recibe | Hoy queda vacío: el colaborador no tiene correo y la hoja no tiene ese campo. Cuando los colaboradores tengan correo de trabajo, saldrá del colaborador, como el teléfono, y no se escribirá a mano en la plantilla. Es candidato (§9.8, D-134, D-140) |
| `attention_phone` | VARCHAR(30) | Sí | Teléfono de quien recibe | Se copia del colaborador elegido. Es un dato personal: la ficha y el borrador lo muestran solo a quien ve los datos personales de los colaboradores; el PDF de una orden enviada lo lleva para todos (D-105, D-134) |
| `support_name` | VARCHAR(150) | Sí | Contacto de soporte: aparece al pie como "Si usted tiene alguna pregunta, póngase en contacto con…" | — |
| `support_phone` | VARCHAR(30) | Sí | Teléfono del contacto de soporte | — |
| `support_email` | VARCHAR(255) | Sí | Correo del contacto de soporte | — |
| `observation` | TEXT | Sí | Texto de «Obs:» al pie de la tabla | Cambia con el tipo porque cada plantilla es de un tipo. Vacío no imprime la línea. No es `special_instructions` (D-132) |
| `special_instructions` | TEXT | Sí | Comentarios o instrucciones que van al pie de cada orden: a dónde enviar las facturas | Es lo que el proveedor necesita para facturar bien. Sin un campo propio, se escribe a mano en cada orden |
| `is_active` | BOOLEAN | No | Si se siguen emitiendo órdenes con ella | Permite retirarla sin tocar las órdenes ya emitidas |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la creó y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Última modificación | — |

**Relaciones.** Apunta a `companies`, `commercial_brands`, `purchase_order_types`, `tax_id_types` y `addresses`. Le apuntan `purchase_orders` y `purchase_order_template_documents`. El logo y la firma no son columnas: son filas de esa tabla, una por tipo.

**Ejemplo**

| nombre | línea | tipo | serie | emisor | atención | soporte |
|---|---|---|---|---|---|---|
| Lima Bikes · Vehículos | Lima Bikes | Vehículos | `OC-LBV` | Millennium Mobility S.A.C. · RUC 20512345671 | Henry Saldaña, Gerente | Alejandro Suarez |
| Lima Bikes · Taller | Lima Bikes | Taller | `OC-LBT` | Millennium Mobility S.A.C. · RUC 20512345671 | Jefe de taller | Alejandro Suarez |
| Lizzo · Vehículos | Lizzo | Vehículos | `OC-LZV` | Millennium Mobility S.A.C. · RUC 20512345671 | Henry Saldaña, Gerente | Alejandro Suarez |

Las tres copiaron el emisor de la misma empresa al crearse y difieren en lo demás. El emisor no se edita en la plantilla: corregir la empresa después no cambia las plantillas ya guardadas.

**Reglas de negocio**

- Los campos son los mismos en todas las plantillas; lo que cambia son sus valores. Una marca comercial o un tipo de orden nuevos no requieren tocar el programa: se crea otra plantilla.
- Al crear una plantilla, los datos del emisor y su domicilio se copian de la empresa en el servidor, y la hoja los muestra sin editarlos. El alta y la edición ignoran un emisor que llegue en el pedido; la respuesta devuelve la copia, que es la que imprime el PDF. Editar la empresa después no cambia las plantillas ya guardadas (D-134).
- El proveedor y su contacto se eligen al emitir la orden, no en la plantilla. La columna Atención sigue siendo el colaborador de la empresa.
- La serie no se repite dentro de la empresa. Se guarda normalizada —mayúsculas, sin espacios— y se puede corregir mientras la plantilla no tenga órdenes; después queda fija, como el código de un catálogo (§2.4).
- La marca comercial, si se indica, tiene que ser una marca de la misma empresa: el sistema lo impide, igual que en las asignaciones de rol (§5.7).
- Puede haber más de una plantilla activa para la misma línea y tipo —dos talleres con contactos distintos, por ejemplo—: quien emite la orden elige cuál usar.
- Los correos se guardan sin espacios y en minúsculas, y el documento del emisor sin guiones, espacios ni puntos (§2.2).
- Una plantilla no se borra: se desactiva. Las órdenes emitidas con ella la siguen referenciando.
- Reactivar una plantilla exige su empresa y, si la tiene, su marca activas (D-94).
- Cada cambio queda en la bitácora (§13), con el valor anterior y el nuevo. El documento de cada orden enviada ya lo conserva su PDF; la bitácora explica qué tenía la plantilla en una fecha pasada.
- Crear plantillas y tipos de orden exige `compras.crear_plantillas`; corregirlos, y cargar o quitar un archivo de la plantilla, `compras.editar_plantillas`; activarlos o desactivarlos, `compras.desactivar_plantillas`. Crear una orden, corregir su borrador, enviarla, darla por cumplida y anularla exigen `compras.crear_ordenes`, `compras.editar_ordenes`, `compras.enviar_ordenes`, `compras.cumplir_ordenes` y `compras.anular_ordenes`. Enviar y cumplir son acciones especiales: comprometen a la empresa ante un tercero y fijan el documento (D-130). Leerlos exige `compras.ver` (§5.6). Una plantilla general, sin marca, es de todas las marcas: la escriben solo las asignaciones de toda la empresa y sin marca (§2.5).
- La observación es de la plantilla: cada tipo o marca comercial lleva la suya. No se vuelve a escribir al emitir (D-132).
- El logo, la firma y cualquier archivo posterior son filas de `purchase_order_template_documents`, no columnas. Un tipo nuevo es una fila de `purchase_order_template_document_types`. Los bytes quedan en `kept_files`; la fila guarda la llave. Quitar el archivo vacía la llave y conserva la fila. Hay como máximo uno de cada tipo por plantilla. Cada archivo es PNG, JPEG, WebP o PDF, de a lo sumo 5 MB; una imagen, de a lo sumo 4096 píxeles por lado y 16 megapíxeles, que se leen de su encabezado antes de decodificarla (D-134). `LOGO` y `SIGNATURE` se dibujan en la orden cuando son imagen; un PDF se guarda y se muestra en la ficha, y la orden impresa lo omite. Un tipo que el PDF no conoce se guarda y se devuelve, y no se imprime hasta que el generador lo reconozca.

#### `purchase_order_template_document_types` — Tipos de archivo de la plantilla

Catálogo global. No es el de los documentos del vehículo.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `code` | VARCHAR(30) | No | `LOGO`, `SIGNATURE`, y el que se agregue | Fijo desde que una fila lo usa. Lo lee el PDF |
| `name` | VARCHAR(80) | No | Nombre visible | — |
| `is_active` | BOOLEAN | No | Si se sigue ofreciendo | — |

**Relaciones.** Le apunta `purchase_order_template_documents`. No tiene empresa: los códigos son del sistema.

#### `purchase_order_template_documents` — Archivos de la plantilla

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa de la plantilla | La frontera entre empresas se lee directo |
| `purchase_order_template_id` | BIGINT FK → `purchase_order_templates` | No | La plantilla | Junto con el tipo no se repite |
| `document_type_id` | BIGINT FK → `purchase_order_template_document_types` | No | Logo, firma u otro | — |
| `file_path` | TEXT | Sí | Llave en `kept_files`. Vacío = el archivo se quitó | No es clave foránea. La fila de `kept_files` no se borra |
| `file_name` | VARCHAR(255) | No | Nombre original | El que ve quien descarga |
| `mime_type` | VARCHAR(100) | No | PNG, JPEG, WebP o PDF | A lo sumo 5 MB; una imagen, a lo sumo 4096 píxeles por lado y 16 megapíxeles |
| `file_size` | BIGINT | Sí | Peso en bytes | Copia para la lista, sin abrir el archivo |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién lo cargó | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Último reemplazo | — |

**Relaciones.** Apunta a `companies`, `purchase_order_templates` y `purchase_order_template_document_types`. La llave apunta por texto a `kept_files.storage_key`.

### 9.3 `purchase_invoices` — Facturas de compra

El comprobante del proveedor. Es el encabezado: quién emitió, cuándo, en qué moneda y por cuánto.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que compra | — |
| `supplier_id` | BIGINT FK → `suppliers` | No | Quién emitió el comprobante | — |
| `purchase_order_id` | BIGINT FK → `purchase_orders` | Sí | A qué orden corresponde | Opcional. Si está, permite calcular lo pendiente de la orden |
| `document_type` | VARCHAR(20) | No | Tipo de comprobante: factura, boleta, otro | Se prellena como factura porque es el caso prácticamente universal, y se puede cambiar |
| `document_number` | VARCHAR(30) | No | Serie y correlativo del comprobante, normalizados: `F001-123` | Es lo que se le muestra al auditor. Normalizado, `F001-00000123` y `F001-123` son el mismo comprobante y la regla de unicidad lo detecta (D-160) |
| `invoice_date` | DATE | No | Fecha del comprobante | Es el límite inferior de varias fechas posteriores y determina el tipo de cambio aplicable |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda del comprobante | — |
| `exchange_rate_date` | DATE | No | Día de la tasa copiada | Un sábado copia la del viernes: sin este dato no se sabría de qué publicación salió la tasa (D-151, D-160) |
| `exchange_rate` | NUMERIC(12,6) | No | Tipo de cambio aplicado | Se guarda el que se usó, no el de hoy: un comprobante de 2024 no puede recalcularse solo cuando cambie el dólar. En Perú es la de venta de la fecha del comprobante, propuesta y nunca cambiada a mano en la factura (D-151, D-152) |
| `tax_rate` | NUMERIC(5,4) | No | Tasa de impuesto aplicada a esta factura | Ver la nota sobre el valor contable |
| `gross_total` | NUMERIC(14,2) | No | Total del comprobante, impuestos incluidos | Debe coincidir con la suma de sus líneas |
| `has_isc_column` | BOOLEAN | No | Si sus líneas llevan el ISC. Por defecto `FALSE` | Se marca cuando el comprobante muestra el ISC, como la columna de descuento de la orden (D-132, D-158) |
| `file_path` | TEXT | Sí | Archivo del comprobante: PDF, PNG, JPEG o WebP, de a lo sumo 5 MB, verificado por su contenido (D-174) | Se envía a los auditores. El cliente lo pidió expresamente |
| `xml_path` | TEXT | Sí | Archivo electrónico del comprobante, si se adjunta | Se guarda para los auditores. Leerlo para llenar la factura es candidato (§9.8): la factura se registra a mano (D-157) |
| `annulled_at` / `annulled_by` | TIMESTAMPTZ / FK `users` | Sí | Cuándo y quién anuló el comprobante completo | — |
| `annulment_reason` | VARCHAR(30) | Sí | Por qué: error de registro o anulada por el emisor. Obligatorio si se anuló | Una factura mal digitada y una que el proveedor dio de baja se explican distinto ante el auditor (D-161) |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Último cambio: del archivo, del XML, de la orden o la anulación | Cada cambio lleva el `updated_at` que se leyó, y uno viejo se rechaza sin guardar, como el borrador de la orden (D-138, D-178) |

**Relaciones.** Apunta a `companies`, `suppliers`, `purchase_orders` (opcional) y `currencies`. Le apuntan sus líneas y las notas de crédito.

**Ejemplo**

| comprobante | proveedor | fecha | moneda | TC | tasa | total |
|---|---|---|---|---|---|---|
| F001-00123 | Crossland | 15/03/2026 | PEN | 1,000000 | 0,1800 | 472 000,00 |
| F002-00981 | Derco | 22/03/2026 | USD | 3,742000 | 0,1800 | 13 100,00 |

La primera cubre ochenta motos; la segunda, un auto. El tipo de cambio es el de la fecha del comprobante, no el de hoy.

**Reglas de negocio**

- El número de comprobante no se repite dentro del mismo proveedor y tipo entre las facturas no anuladas: anular una libera su número para registrarla bien. En Perú la serie tiene cuatro caracteres y el correlativo hasta ocho dígitos; en otro país se acepta sin verificar el formato (D-160).
- La fecha no es futura en la zona horaria de la empresa (§2.6), y una factura tiene hasta 100 líneas (D-160).
- La suma de las líneas debe coincidir con el total del comprobante. Es la validación que detecta una línea olvidada al cargar ochenta motos.
- Una factura con más de veinte días de antigüedad solo la registra quien tenga el permiso de excepción. No es un bloqueo absoluto: es un permiso que se puede otorgar.
- La factura no se borra. Si se anula por completo, se marca con quién, cuándo y por qué. Anular exige `compras.anular_factura` y procede solo si ninguna de sus líneas tiene notas de crédito ni pagos aplicados; sus unidades no se anulan con ella (D-161).
- **Lo fiscal no se edita** (D-161): el proveedor, el tipo, el número, la fecha, la moneda, la tasa, los montos y las líneas. Un error se corrige anulando y registrando otra, que la pantalla precarga con «Anular y copiar». Sí cambian el archivo y la orden vinculada.
- **Se registra a mano** desde el comprobante (D-157). Leer su XML para llenarla llega después, sin cambiar las tablas (§9.8).
- Una factura en la moneda funcional de su empresa lleva tasa 1 y, como día de la tasa, su propia fecha: no consulta el catálogo (D-169).
- Cambiar el archivo o la orden de una factura vigente exige `compras.registrar_factura`. Solo se vincula a una orden enviada del mismo proveedor; desvincularla se puede siempre, y una orden ya cumplida sigue cumplida. Una factura anulada ya no cambia (D-172). Cambiar el archivo, el XML o la orden, y anular, exigen el mismo alcance que registrar: la empresa y la marca comercial de cada línea (D-178).
- Anular una factura deja su fila en la bitácora con el motivo (§13.1), e Inventario muestra y filtra las unidades que quedaron sin factura vigente, para que una anulada sin reemplazo no se pierda (D-175).

**Sobre el impuesto.** La tasa se guarda en cada factura, no como constante: cambia por ley y difiere entre países. `countries` propone la del país al registrar y el valor es editable, porque la tasa que manda es la del comprobante. El ISC va incluido en el precio y no se recupera, así que forma parte del costo; se guarda aparte solo como información, en las facturas que lo muestran (D-158). **El costo sin IGV se calcula** (D-159): en una factura, el valor del vehículo ÷ (1 + `tax_rate`), como el valor contable de la guía funcional, que divide entre 1,18; en una boleta u otro comprobante, que no dan crédito fiscal, el valor completo. La exportación de facturas muestra la base y el IGV calculados así —en una boleta, la base igual al total y el IGV en cero—, rotulados «calculado» porque pueden diferir en céntimos del comprobante (D-180).

### 9.4 `purchase_invoice_items` — Líneas de factura

**Es la tabla que el modelo anterior no tenía, y la que hace posible todo lo demás.** Sin ella, una factura de ochenta motos es un solo monto y no hay forma de anular dos unidades, ni de saber cuánto costó cada una.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `purchase_invoice_id` | BIGINT FK → `purchase_invoices` | No | A qué comprobante pertenece | — |
| `company_id` | BIGINT | No | Empresa, heredada de la factura | Para su política de RLS (§2.5, D-160) |
| `vehicle_id` | BIGINT FK → `vehicles` | Sí | Qué unidad se está comprando | Vacío cuando la línea no es un vehículo: un flete, un accesorio, un trámite. Hoy solo se usan líneas de vehículo, pero la estructura ya lo admite. La primera versión solo registra líneas de vehículo (D-173) |
| `concept` | VARCHAR(150) | Sí | Descripción de la línea cuando no es un vehículo | — |
| `model_id` / `model_year` | BIGINT FK → `vehicle_models` / SMALLINT | Sí | Modelo y año de la unidad, copiados del comprobante | Lo pendiente de la orden se cuenta por modelo y año sin consultar a `fleet`. Vacíos en una línea de concepto (D-160, D-162) |
| `commercial_brand_id` | BIGINT FK → `commercial_brands` | Sí | Marca comercial de la unidad | Registrar exige actuar sobre ella (D-163). Vacía en una línea de concepto |
| `invoiced_amount` | NUMERIC(14,2) | No | **Valor de factura**: lo que el comprobante dice que costó la unidad, con impuestos. Mayor que cero (D-179) | No se modifica nunca. Es lo que se reporta al fondo y lo que ve el auditor (D-159) |
| `isc_amount` | NUMERIC(14,2) | Sí | Impuesto selectivo incluido en el precio | Obligatorio, de cero hasta el monto de la línea, cuando la factura tiene `has_isc_column`; vacío si no. Autos y motos. Informativo: ya está dentro del valor de factura (D-158) |
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
- **La unidad nace con la factura** (D-153). Si el VIN de una línea no existe, el registro de la factura le pide a `fleet` darla de alta, con el motivo "Registro de compra" (§8.3); si ya está en Inventario, la vincula y no la duplica. La factura no pide ubicación: la unidad nace sin ella, Fuera de tienda, y la recibe al registrar su ingreso físico (§11.1, D-160).
- **Una unidad está en una sola línea vigente** (D-160). Si su VIN ya existe, se vincula solo si es de la misma empresa, no está anulada y no tiene otra línea vigente; su modelo, su año y su marca comercial mandan, y una línea que diga otros se rechaza. Su color y su motor tampoco se toman de la línea: el formulario los muestra fijos y avisa, sin impedir guardar, si lo escrito difiere o si la unidad no tiene motor, para que se corrija en Inventario (D-179). Un VIN mal escrito se corrige en la unidad con `stock.corregir_vin`, sin tocar la factura (D-161).
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
| `total_amount` | NUMERIC(14,2) | No | Monto total del documento | Debe coincidir con la suma de lo que reparte entre unidades |
| `reason` | VARCHAR(30) | No | Por qué se emitió: descuento de campaña, anulación de unidad, anulación de la factura, ajuste de precio | Un descuento y una anulación tienen efectos distintos sobre la unidad. Solo «anulación de unidad» anula la unidad; «anulación de la factura», que el proveedor emite para re-emitirla, la deja sin factura vigente (D-191) |
| `campaign_month` | DATE | Sí | Mes de la campaña que la origina, guardado con día 1 | Es lo que permite responder qué notas corresponden a las entregas de septiembre, que es como el proveedor las liquida |
| `file_path` | TEXT | Sí | Archivo del documento: PDF o imagen, como la factura (D-174) | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién la registró y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Último cambio, como la anulación | El cambio lleva el `updated_at` que se leyó (D-178, D-193) |
| `annulled_at` / `annulled_by` | TIMESTAMPTZ / FK `users` | Sí | Cuándo y quién la anuló | — |
| `annulment_reason` | VARCHAR(30) | Sí | Error de registro o anulada por el emisor | Obligatorio si se anuló (D-193) |

#### `credit_note_items` — Reparto de la nota entre unidades

Es donde la nota se vuelve concreta: qué unidad recibe cuánto.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `credit_note_id` | BIGINT FK → `credit_notes` | No | La nota que se está repartiendo | — |
| `company_id` | BIGINT | No | Empresa, heredada de la nota | Para su política de RLS (§2.5) |
| `exchange_rate` | NUMERIC(12,6) | No | Tasa copiada de la factura que descuenta esta línea; 1 en la moneda funcional | Una nota de varias facturas vale en soles lo que redujo cada una (D-152, D-169, D-192) |
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
- **El tipo de cambio es el de la factura que descuenta** (D-152), no el de su fecha: la nota reduce el impuesto que esa factura registró, y con otra tasa la reducción no coincidiría. Cada línea copia el de su factura (D-192).
- Todas las facturas de una nota son del mismo proveedor, la misma empresa y la misma moneda. Desde el 01/01/2027, por su fecha de emisión, una nota descuenta una sola factura (D-192).
- **Reduce lo que se debe de su factura** (D-190): ~~solo si esa factura ya estaba pagada, lo pagado de más es saldo a favor del proveedor.~~ desde D-248 (decidida y construida el 10/11/2026, con V127), por vehículo: lo que la nota baja del valor de un vehículo que todavía debe, baja lo que se debe; lo que baja de un vehículo ya cubierto pasa a saldo a favor del proveedor, aunque la factura todavía deba por otros vehículos (§10.4). En qué pagos se usó ese saldo lo guarda `supplier_balance_uses` (D-249).
- Se registra a mano y se anula como la factura (D-193): número normalizado y único por proveedor entre las no anuladas, fecha no futura, reparto que puede hacerse en partes iguales entre las unidades elegidas, y anulación con motivo que no deja negativo el saldo del proveedor. Desde D-249, la anulación tampoco deja sin respaldo un uso de su saldo: una nota cuyo saldo pagó otra factura no se anula hasta anular ese pago (`CREDITNOTE_BALANCE_USED`, §10.4).
- La suma de lo repartido debe coincidir con el total del documento.

### 9.6 Valor de factura y valor del vehículo

Son dos datos distintos y ambos hacen falta.

| Concepto | Dónde vive | Cambia | Para qué se usa |
|---|---|---|---|
| **Valor de factura** | `purchase_invoice_items.invoiced_amount` | Nunca | Reportes externos y auditoría |
| **Notas de crédito de la unidad** | `credit_note_items`, una fila por nota | Se agregan filas | Historia de los descuentos |
| **Valor del vehículo** | No se guarda: se calcula | Con cada nota nueva | Contabilidad, inventario y análisis interno |
| **Costo sin IGV** | No se guarda: se calcula | Con cada nota nueva | Valor contable: el valor del vehículo ÷ (1 + tasa) en una factura, el valor completo en una boleta (D-159) |

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
- Cuando las notas de crédito dejan en cero el valor de una línea, la línea se anula. Solo con el motivo «anulación de unidad», que la devuelve al proveedor, la unidad también queda anulada (D-104), salvo que tenga contratos o una cesión abierta, en cuyo caso queda marcada para revisar; con otro motivo, queda sin factura vigente (D-191). Anular su factura no la anula: la unidad queda sin factura vigente, se vincula por su VIN a la factura que la reemplaza y, si la compra no existió, se anula por la vía explícita (D-161).
- **La unidad solo se puede anular si su valor llegó a cero y no tiene ningún contrato.** Tampoco con una cesión abierta a un fondo: primero se cierra la cesión (D-182). Es el candado para que una nota de crédito no borre información de un cliente. Desde el 13/11/2026 (D-250), mientras ventas no existe, el contrato se lee del historial de la unidad: lo tiene la que espera en Contrato firmado o la que entró alguna vez a Activo, aunque haya vuelto; una firma que deshizo el regreso automático no cuenta. Hasta entonces ninguna unidad tenía contratos, y una unidad con su cliente se anulaba en cuanto su valor llegaba a cero.

### 9.7 Lo que queda para el bloque de pagos

Una nota de crédito tiene dos efectos independientes y aquí está resuelto solo el primero:

| Efecto | Dónde vive |
|---|---|
| Reduce el valor de una unidad concreta | Este bloque |
| Genera un saldo a favor con el proveedor, que sirve para pagar otras facturas | Bloque 8 |

El valor de las unidades no cambia porque una compra se haya cubierto con saldo: siguen costando lo que costaron, pero no salió efectivo. Son dos reportes distintos.

**Desde el 10/11/2026** (D-248 y D-249, decididas y construidas ese día, con V127 y V128): el segundo efecto se mide por vehículo. Una nota sobre un vehículo que todavía debe solo baja lo que se debe. Sobre uno ya cubierto, lo que baja de su valor pasa a saldo, aunque su factura todavía deba por otros vehículos; en un vehículo saldado del canal de dealers pasa entero, y la inicial del cliente no cambia. Lo que pasó a saldo de cada nota, y en qué pagos se usó, queda registrado (§10.4). El primer efecto no cambia: el valor del vehículo es lo facturado menos sus notas, con el número, la fecha y el motivo de cada una (§9.6).

### 9.8 Candidatos del bloque 7

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| Extracción automática desde el archivo electrónico | `purchase_invoices` | Que se implemente la lectura del comprobante para evitar digitar ochenta líneas. D-154 la admitió el 01/11/2026 y D-157 la devolvió a candidato el 02/11/2026: primero se registra a mano, y el lector llega con XML reales de cada proveedor, sin cambiar las tablas |
| ~~`withholding_amount`~~ | ~~`purchase_invoices`~~ | ~~Que aparezcan retenciones o detracciones, que hoy no se registran~~ La retención se registra al pagar, en cada aplicación (§10.2, D-196) |
| `expected_delivery_date` | `purchase_order_items` | Que haga falta seguir la fecha comprometida de llegada, no solo la cantidad |
| Una sola plantilla activa por línea y tipo | `purchase_order_templates` | Que se pida impedir dos plantillas vigentes para la misma marca comercial y el mismo tipo de orden |
| Proveedor del exterior | `suppliers` | Que se compre a proveedores de otro país: su domicilio y su documento tributario serían de ese país, con un `country_id` propio del proveedor (D-133). La dirección dejaría de estar en el país de la empresa, así que `addresses` pasaría a acotarse por empresa (§4.8) |
| `attention_email` con fuente | `purchase_order_templates` | Que el colaborador tenga un correo de trabajo: de él saldrá el de Atención, no de la hoja. Hoy la columna queda vacía (D-134, D-140) |


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
| `payment_date` | DATE | No | Fecha del pago | Puede ser anterior a la factura. No se valida contra ella, pero no es futura (D-212) |
| `exchange_rate_date` | DATE | No | Día de la tasa del pago | Como la factura (D-160); en la moneda funcional, la fecha del pago (D-169, D-210) |
| `currency_id` | BIGINT FK → `currencies` | No | Moneda | — |
| `exchange_rate` | NUMERIC(12,6) | No | Tipo de cambio aplicado | El del día del pago, que puede diferir del de la factura |
| `amount` | NUMERIC(14,2) | No | Monto total del pago | — |
| `method` | VARCHAR(30) | No | Cómo se pagó: transferencia, cheque, efectivo, saldo a favor | Un pago con saldo a favor no mueve dinero, y esa diferencia es la que separa el flujo de caja del valor del inventario |
| `operation_number` | VARCHAR(100) | Sí | Número de operación del voucher | Es lo que se concilia con el banco |
| `bank_account_id` | BIGINT FK → `company_bank_accounts` | Sí | Cuenta de la que salió el dinero (D-214) | Dice contra qué extracto se concilia el pago. Vacía en efectivo, con saldo a favor y en los pagos anteriores a esta regla |
| `file_path` | TEXT | Sí | Archivo del voucher: PDF o imagen, como la factura (D-174, D-215) | Es el respaldo del pago ante el auditor. Vacío con saldo a favor y en los pagos anteriores a esta regla |
| `request_id` | UUID | No | Identificador que manda la pantalla con cada envío | Un pago enviado dos veces se guarda una sola (D-198) |
| `annulment_reason` / `annulled_by` | VARCHAR(30) / FK `users` | Sí | Por qué y quién lo anuló | Obligatorios si se anuló (D-198) |
| `annulled_at` | TIMESTAMPTZ | Sí | Cuándo se anuló el pago | Un pago mal registrado no se borra: se anula y se vuelve a registrar. Las aplicaciones de un pago anulado dejan de contar |
| `notes` | TEXT | Sí | Detalle | — |
| `created_at` / `created_by` | TIMESTAMPTZ / FK `users` | No / Sí | Quién lo registró y cuándo | — |
| `updated_at` / `updated_by` | TIMESTAMPTZ / FK `users` | No / Sí | Último cambio: la anulación o el voucher | La anulación y el cambio del voucher llevan el `updated_at` que se leyó (D-198, D-210, D-215) |

**Relaciones.** Apunta a `companies`, `suppliers` y `currencies`, y a `company_bank_accounts` por la pareja `(bank_account_id, company_id)`, para que no pueda salir de la cuenta de otra empresa. Le apuntan sus aplicaciones. Un pago con saldo no apunta a una nota: consume el saldo del proveedor (D-195). Desde D-249 (decidida y construida el 10/11/2026, con V128) sigue sin apuntar a una nota en su fila, pero sus usos —de qué notas salió el saldo y cuánto de cada una— se guardan aparte, en `supplier_balance_uses` (§10.4).

**Ejemplo**

| operación | fecha | monto | método | nota consumida |
|---|---|---|---|---|
| OP-99213 | 12/03/2026 | 300 000,00 | Transferencia | — |
| — | 04/07/2026 | 306 000,00 | Saldo a favor | NC001-0045 y otras |

El segundo no movió dinero: se cubrió con notas de crédito acumuladas. Las unidades siguen valiendo lo mismo. Hasta D-249, «nota consumida» no salía de ningún dato: el pago no apuntaba a una nota. Desde D-249 es real: la dicen los usos que el pago guarda al registrarse (§10.4), que la ficha del pago muestra como «De dónde salió el saldo».

**Reglas de negocio**

- El número de operación no se repite dentro del mismo proveedor entre los pagos no anulados: un mismo voucher no puede registrarse dos veces (D-198). Es obligatorio en transferencia y cheque, y no lleva con saldo a favor (D-212).
- Un pago se aplica completo: la suma de los equivalentes de sus aplicaciones es igual a su monto, porque lo sin aplicar no sería deuda ni saldo y desaparecería de toda cuenta (D-210).
- El equivalente de una aplicación es su monto a su tasa, redondeado a los decimales de la moneda del pago. Si los redondeos dejan la suma a unos céntimos del monto, las aplicaciones en otra moneda, de la última a la primera, toman la diferencia mientras su monto siga siendo el redondeo de su equivalente convertido de vuelta; se guarda ese equivalente. Así un pago en soles de una factura en dólares cuadra con el voucher: a 3,76, USD 2 659,57 se aplica con S/ 10 000,00, que de otro modo no se alcanzaba. Si la diferencia no cabe, el pago no se registra (D-210).
- Un pago con método *saldo a favor* consume el saldo del proveedor en su moneda, calculado con el proveedor bloqueado, y no lo supera; no apunta a una nota (D-195). Desde D-249 guarda, con el proveedor bloqueado, qué notas consumió y cuánto de cada una: primero el saldo que no explica ninguna nota y después las notas en el orden en que se registraron (§10.4).
- Transferencia y cheque llevan cuenta de origen y voucher, porque el dinero sale de una cuenta y el número de operación ya sale del voucher. Efectivo no lleva cuenta, y su voucher es opcional. Con saldo a favor no se lleva ninguno de los dos, porque no sale dinero (D-214, D-215).
- La moneda de un pago con cuenta es la de su cuenta, y la cuenta tiene que estar activa y ser de la empresa del pago (D-214).
- Lo que se registró no se edita, y la cuenta tampoco: una equivocada se corrige anulando y copiando el pago. El voucher sí se cambia sin anular un pago vigente, con `pagos.registrar_pago` y el alcance del pago, y el cambio queda en la bitácora (D-215).
- Los pagos registrados antes de esta regla y los migrados quedan sin cuenta. A esos se les puede adjuntar el voucher después (D-215).
- El `request_id` impide guardar dos veces el mismo envío: el mismo contenido responde el pago que ya existe, y otro contenido se rechaza (D-198).
- El pago no se borra. Si se registró mal, se anula con `pagos.anular_pago` y su motivo, y se vuelve a registrar. No se anula si, sin él, el saldo del proveedor quedaría negativo. Desde D-249, tampoco si su anulación deja sin respaldo un uso de saldo ya registrado (`SUPPLIERPAYMENT_BALANCE_USED`): primero se anula el pago con saldo que lo usó. La anulación deja su fila en la bitácora (D-198).

### 10.2 `payment_applications` — A qué se aplica cada pago

Un pago se reparte entre facturas, órdenes de compra, o ambas.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `payment_id` | BIGINT FK → `supplier_payments` | No | Qué pago se está aplicando | — |
| `purchase_invoice_id` | BIGINT FK → `purchase_invoices` | Sí | A qué factura | — |
| `purchase_order_id` | BIGINT FK → `purchase_orders` | Sí | A qué orden, cuando todavía no hay factura | Es el caso normal: el pago suele ir antes que el comprobante |
| `company_id` | BIGINT | No | Empresa, heredada del pago | Para su política de RLS (§2.5, D-198) |
| `amount` | NUMERIC(14,2) | No | Cuánto de ese pago se aplica aquí, en la moneda de la factura o la orden | Permite que un pago cubra tres facturas con montos distintos |
| `exchange_rate` / `exchange_rate_date` | NUMERIC(12,6) / DATE | No | Tasa y día con que se convierte esta aplicación: la de venta del día del pago entre las dos monedas, o 1 si son iguales | Un pago en soles tiene tasa 1 y no sirve para convertir una factura en dólares; guardada, la diferencia de cambio se puede reproducir (D-210) |
| `payment_amount` | NUMERIC(14,2) | No | El mismo monto en la moneda del pago, convertido con la tasa de la aplicación | Una factura en dólares se paga en soles; el costo de la unidad no cambia (D-194) |
| `withheld_amount` | NUMERIC(14,2) | No | Retención del IGV practicada, en la moneda de la aplicación. Por defecto 0 | Se suma a `amount`, que es dinero o saldo y cuadra con el voucher: lo cubierto es `amount` más lo retenido. Cuenta como cubierto, no como dinero entregado al proveedor (D-196, D-210) |
| `withholding_certificate` | VARCHAR(30) | Sí | Número del comprobante de retención; obligatorio si hay retención | Una retención sin su comprobante no se registra (D-210) |
| `spread` | BOOLEAN | No | Si sus imputaciones son el reparto que hizo el sistema al registrar, porque el pago no eligió vehículos. Por defecto `FALSE`, también en los pagos registrados antes, que V127 no rellena | Desde D-248 (decidida y construida el 10/11/2026, con V127): «Todos los vehículos pendientes», «Otro monto», el anticipo que pasa de una orden a su factura y el bloque migrado de un proveedor normal guardan cuánto le tocó a cada vehículo, y una nota posterior afecta solo a su vehículo. La ficha del pago lo dice «Repartido entre los vehículos» |

**Relaciones.** Apunta a `supplier_payments` y a `purchase_invoices` u `purchase_orders`.

**Ejemplo** — un pago de 300 000 repartido entre dos facturas y una orden aún sin facturar:

| pago | fecha | aplicado a | monto |
|---|---|---|---|
| OP-99213 | 12/03/2026 | Factura F001-00123 | 177 000 |
| OP-99213 | 12/03/2026 | Factura F001-00124 | 88 500 |
| OP-99213 | 12/03/2026 | Orden OC-LZV-2026-0003 | 34 500 |

**Reglas de negocio**

- Cada aplicación apunta a una factura **o** a una orden, nunca a las dos.
- La suma de los equivalentes de las aplicaciones, en la moneda del pago, es exactamente su monto: un pago se aplica completo (D-210). Una de las dos monedas, la del pago o la de la factura, es la funcional de la empresa, o las dos son iguales (D-194).
- Lo aplicado a una factura no supera lo que se debe de ella al aplicarlo: su total, menos sus notas vigentes, menos lo ya aplicado (D-190).
- Cuando una factura se vincula a una orden con pagos aplicados, la aplicación pasa sola de la orden a la factura, hasta lo que se debe de ella; lo que sobra queda en la orden. El pago no se toca. Solo pasa en la misma moneda, y desvincular la factura no devuelve nada a la orden. Solo una orden enviada recibe pagos, y nunca la de un proveedor del canal de dealers. Una orden con pagos aplicados no se anula ni se cumple (D-198, D-211). Desde D-248, la parte que pasa a la factura se guarda repartida entre sus vehículos (`spread`), como un pago que no eligió vehículos.

### 10.3 `payment_allocations` — Qué unidades cubre un pago

Opcional. Solo hace falta cuando un pago parcial libera unidades concretas. Desde D-248 (decidida y construida el 10/11/2026, con V127) guarda también el reparto de un pago que no eligió vehículos (`payment_applications.spread`, §10.2).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `payment_application_id` | BIGINT FK → `payment_applications` | No | Qué aplicación se está detallando | — |
| `vehicle_id` | BIGINT FK → `vehicles` | No | Qué unidad queda cubierta | — |
| `company_id` | BIGINT | No | Empresa, heredada de la aplicación | Para su política de RLS (§2.5, D-198) |
| `amount` | NUMERIC(14,2) | No | Cuánto le corresponde | — |
| `settles_unit` | BOOLEAN | No | Si deja saldada la unidad. Por defecto `FALSE` | En el canal de dealers, lo que le falta a una unidad saldada es la inicial del cliente, y desde entonces no recibe más pagos (D-197) |
| `customer_initial` | NUMERIC(14,2) | Sí | La inicial del cliente, congelada al saldar: lo que le faltaba a la unidad después de esta imputación. Solo en la imputación que salda, y mayor o igual que cero; vacía en las demás | Desde D-248 (decidida y construida el 10/11/2026, con V127): una nota posterior no la achica, y lo que la nota baja pasa a saldo. V127 la rellena en las imputaciones que ya saldan, con el valor de hoy, y después fija su restricción: la imputación que salda la lleva y ninguna otra |

**Reglas de negocio**

- **Si no se detalla, lo aplicado se reparte** entre las unidades no saldadas de la factura en proporción a lo que le falta a cada una, que sin detalle es su valor (D-211). Es el comportamiento por defecto y cubre el caso normal. Desde D-248 el reparto se hace al registrar, con la misma proporción y los mismos céntimos, y se guarda como imputaciones de una aplicación repartida (`spread`), que no cuentan para los límites de 100 por documento y 500 por pago. Los pagos registrados antes conservan su reparto al leer.
- **En compras al canal de dealers el detalle es obligatorio**, porque la diferencia entre el valor y lo pagado corresponde a la inicial de un cliente concreto. Repartirla entre todas las unidades atribuiría iniciales a clientes que no las pagaron.
- Si se detalla, esas unidades quedan pagadas y el resto no. Es el caso de pagar media factura para llevarse 25 de las 50 motos: se llevan motos enteras, no media moto cada una.
- Una unidad está pagada cuando lo aplicado a ella cubre su valor de vehículo. Sin detalle, lo aplicado a su factura se reparte en proporción a lo que le falta a cada unidad (D-199, D-211).
- Los céntimos que el redondeo deja de más o de menos se mueven de a uno desde la primera unidad, sin dejar ninguna bajo cero ni por encima de lo que le falta, para que la suma coincida con lo aplicado; una unidad cuya parte queda en cero no recibe imputación (D-211).
- Lo imputado a una unidad no supera lo que le falta para su valor, y una unidad saldada no recibe más imputaciones (D-197, D-211). Desde D-248, una unidad saldada recibe una imputación sin saldar solo por lo que le falte además de su inicial congelada, que ocurre cuando se anula una nota registrada antes de saldarla; si no le falta nada, se rechaza como hasta ahora.
- Desde D-248, la imputación que salda guarda la inicial del cliente (`customer_initial`): lo que le faltaba a la unidad después de ella. Una nota posterior no la cambia.
- Una imputación no cambia nunca: para reasignarla se anula el pago y se registra otro (D-211). La base solo la inserta.

### 10.4 El saldo a favor con el proveedor

Una nota de crédito tiene dos efectos y este es el segundo: además de bajar el valor de una unidad, deja un saldo disponible con ese proveedor.

| Concepto | Cómo se obtiene |
|---|---|
| Lo que se debe de una factura | Su total, menos sus notas vigentes, menos lo aplicado de pagos. Desde D-248, más lo que sus vehículos pasaron a saldo (abajo) |
| Saldo del proveedor, por moneda | ~~Lo aplicado de más en sus facturas de esa moneda —porque la nota llegó después de pagarlas—,~~ Desde D-248, lo que pasó a saldo en sus facturas de esa moneda (`toBalance`, abajo), menos lo consumido en pagos con saldo (D-190, D-195) |

**No se guarda como campo**: se calcula. Un saldo almacenado se desincroniza al primer pago que nadie registre bien.

**Desde el 10/11/2026, el saldo se mide por vehículo** (D-248, decidida y construida ese día, con V127). Lo aplicado de más a una factura entera no alcanzaba: una nota sobre un vehículo ya pagado, en una factura que todavía debía por otro, bajaba lo que se debía de la factura, el exceso de ese vehículo no era de nadie y el otro seguía debiendo todo; la factura podía quedar pagada con un vehículo en pago parcial. Ahora se mide cada línea de vehículo de la factura, vigente o anulada porque una nota la dejó en cero:

| Concepto | Cómo se obtiene |
|---|---|
| Valor | Lo facturado menos sus notas vigentes (§9.6) |
| Cubierto | Sus imputaciones de pagos vigentes, elegidas o repartidas (§10.3); en los pagos registrados antes de este cambio, también su parte del fondo sin imputar |
| Inicial | La congelada al saldar (`customer_initial`, §10.3), si está saldado; si no, cero |
| Pendiente | Valor − cubierto − inicial, nunca menos de cero |
| Pasó a saldo | Cubierto + inicial − valor, nunca menos de cero |

Toda pantalla muestra la identidad **valor − cubierto − inicial + pasó a saldo = pendiente**. La «diferencia» de la API es pendiente + inicial, nunca negativa (§10.5).

Por factura, con E la suma de lo que pasó a saldo de sus vehículos:

| Concepto | Cómo se obtiene |
|---|---|
| Lo que se debe (`owed`) | Total − notas − aplicado + E, con su signo, como hasta ahora |
| Lo que pasa a saldo (`toBalance`) | El mayor entre E y aplicado + notas − total |
| Lo que se puede pagar (`payable`) | Lo que se debe menos las iniciales de sus vehículos saldados (§10.5) |
| Saldo del proveedor, por moneda | La suma de `toBalance` de sus facturas de esa moneda, menos lo consumido en pagos con saldo |

**Ejemplos:**

| Caso | Por pagar | Pasa a saldo |
|---|---|---|
| V1 y V2 de 10 000,00 cada uno. V1 pagado con una imputación de 10 000,00. Nota de 1 000,00 sobre V1 | 10 000,00, todo de V2 | 1 000,00 |
| Lo mismo, pero pagado con «Otro monto» de 10 000,00, que guarda 5 000,00 en cada uno. Nota de 1 000,00 sobre V1 | 9 000,00: 4 000,00 de V1 y 5 000,00 de V2 | 0,00 |
| Concesionario, 50 000,00: Millennium 42 000,00 + inicial 8 000,00, saldado. Nota de 10 000,00 | Los 8 000,00 de la inicial, que Millennium no paga: lo que se puede pagar es 0,00 y la factura sigue saldada | 10 000,00 |
| Concesionario: nota de 10 000,00 antes de saldar (Millennium 32 000,00 + inicial 8 000,00). Después se anula la nota | 10 000,00, que se pueden pagar al vehículo saldado | 0,00 |

En el tercero, el vehículo vale 40 000,00, está cubierto por 42 000,00 y su inicial sigue en 8 000,00: 40 000,00 − 42 000,00 − 8 000,00 + 10 000,00 = 0,00. Hasta D-248 la inicial era lo que le faltaba al vehículo, así que la nota la achicaba: de los 10 000,00 pasaban 2 000,00 a saldo y 8 000,00 se perdían. En el cuarto, al anular la nota el vehículo vuelve a valer 50 000,00: 50 000,00 − 32 000,00 − 8 000,00 + 0,00 = 10 000,00 pendientes. La inicial sigue en 8 000,00, y un pago sin saldar puede cubrir esos 10 000,00 (§10.3).

**Lo que se sostiene:**
- un pago de dinero nunca crea saldo: lo topan lo que se debe de la factura y lo que le falta a cada vehículo;
- una nota nueva nunca baja el saldo, ni lo que aportaron las notas anteriores, porque el aporte se cuenta en el orden de registro (D-249);
- al desplegar, ningún saldo baja;
- el filtro previo de Por pagar en la base sigue siendo seguro, porque E nunca supera las notas;
- las anulaciones de un pago y de una nota ya bloquean al proveedor.

**Los pagos registrados antes** conservan su fondo: su parte sin imputar se sigue repartiendo al leer, como hasta ahora, y para ellos una nota posterior se reparte igual. En `dev` solo hay datos de prueba.

**Reglas de negocio**

- El saldo solo sirve para pagar facturas **del mismo proveedor** y de la misma moneda.
- ~~**Una nota de una factura todavía impaga no genera saldo:** reduce lo que se debe de ella.~~ **Desde D-248 se mide por vehículo:** una nota sobre un vehículo que todavía debe no genera saldo, porque reduce lo que se debe; sobre un vehículo ya cubierto, lo que baja de su valor pasa a saldo, aunque la factura todavía deba por otros vehículos. Contarla también como saldo la descontaría dos veces (D-190).
- Un pago con saldo a favor **no mueve dinero**, pero cancela la factura igual. Las unidades siguen valiendo lo que valen: el saldo no baja el inventario, baja la caja que iba a salir.
- La nota que genera el saldo puede haber descontado una compra anterior. Son dos relaciones distintas: a qué unidad le bajó el valor, y qué factura pagó. Desde D-249 las dos quedan guardadas: la primera en `credit_note_items` (§9.5), la segunda en `supplier_balance_uses` (abajo).

#### `supplier_balance_uses` — De qué notas salió el saldo de cada pago

Decidida y construida el 10/11/2026 (D-249); la crea V128 (backend `0bc4bf39`). Un pago con saldo a favor consume el saldo del proveedor, que no es de una nota (D-195). Esta tabla guarda, al registrar el pago, de qué notas salió ese saldo y cuánto de cada una. Es el rastro que pidió el responsable: en qué se usó la nota de un vehículo.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `id` | BIGINT | No | Llave interna | — |
| `public_id` | UUID | No | Identificador que viaja por la API | — |
| `company_id` | BIGINT FK → `companies` | No | Empresa, la del pago | Para su política de RLS (§2.5) y para que sus llaves no crucen empresas |
| `supplier_payment_id` | BIGINT FK → `supplier_payments` | No | El pago con saldo que lo consumió | — |
| `credit_note_id` | BIGINT FK → `credit_notes` | Sí | La nota de la que salió esta parte. Vacía es «sin nota»: el saldo que no explica ninguna nota | Dice en qué se usó cada nota, que muestran la ficha de la nota, la del pago y la del vehículo |
| `amount` | NUMERIC(14,2) | No | Cuánto salió de esa nota, mayor que cero | Lo disponible de una nota es lo que aportó al saldo menos sus usos en pagos vigentes |

**Relaciones.** Apunta a `supplier_payments` y a `credit_notes` por la pareja con `company_id`, como sus vecinas.

**Ejemplo** — el primero de arriba: la nota NC001-0045 de 1 000,00 sobre V1, ya pagado, pasa 1 000,00 a saldo, y un pago con saldo de 1 000,00 salda parte de otra factura del mismo proveedor:

| pago | nota | monto |
|---|---|---|
| Saldo a favor, 1 000,00 | NC001-0045 | 1 000,00 |

La ficha de la nota dice que pasaron 1 000,00 a saldo, que se usaron en ese pago y que no queda nada disponible; la del pago, de qué nota salió; y la de V1, bajo esa nota, en qué factura se usó.

**Reglas de negocio**

- **Se escribe una vez y no cambia nunca.** Al registrar el pago con saldo, con el proveedor bloqueado, se guardan sus usos: primero «sin nota» y después las notas en el orden en que se registraron, hasta cubrir lo que el pago consume. La cuenta de la aplicación solo tiene `SELECT` e `INSERT` sobre la tabla: es inmutable por sus permisos, sin función nueva. Tiene Row Level Security por empresa, como sus vecinas.
- **Lo que aporta una nota** se cuenta por factura: cada línea de la nota, en el orden de registro de la nota y de la línea, aporta lo que sube lo que su factura pasa a saldo (`toBalance`). Lo que no explica ninguna nota es «sin nota». Una nota registrada después nunca cambia lo que aportaron las anteriores.
- **Lo disponible de una nota** es su aporte menos sus usos en pagos vigentes. Anular el pago con saldo libera sus usos.
- **Ningún uso queda sin respaldo.** Una nota cuyo saldo se usó no se anula hasta anular ese pago, tampoco cuando anularla cambia el aporte de otra nota ya usada (`CREDITNOTE_BALANCE_USED`, que nombra los pagos). Un pago no se anula si su anulación deja sin respaldo un uso, como el que pagó el vehículo cuyo exceso ya se usó (`SUPPLIERPAYMENT_BALANCE_USED`).
- Un pago nombra cada fuente una vez, también «sin nota» (`supplier_balance_uses_payment_note_uq`, única con `NULLS NOT DISTINCT`). Lo que ninguna fuente puede dar se guarda como «sin nota».
- ~~Qué usos se atribuyen a los pagos con saldo registrados antes de V128, que no los guardaron, se fija al construir.~~ Se fijó al construir: V129, una migración en Java, da sus usos a los pagos con saldo vigentes registrados antes, por su fecha y su identificador, con la misma regla del registro; un pago que ya tiene usos no se vuelve a tomar, así que correrla otra vez no escribe nada.
- No se versiona en `data_versions`: ningún documento la muestra (§2.7, D-128).

### 10.5 Indicadores que se calculan

| Indicador | Cómo sale |
|---|---|
| Diferencia | Valor del vehículo menos lo cubierto. En el canal normal es lo pendiente de pago; en una unidad saldada del canal de dealers, la inicial del cliente (D-197, D-199). Desde D-248: lo pendiente más la inicial congelada, nunca menos de cero; lo que el vehículo cubre por encima de su valor es lo que pasó a saldo (§10.4) |
| Dinero entregado | Lo cubierto menos la retención y menos lo pagado con saldo: es el dato de caja (D-199) |
| Por pagar | Lo que se debe de una factura vigente menos, en el canal de dealers, las iniciales de sus unidades saldadas: lo que la empresa todavía puede pagar (D-190, D-197, D-224). Se ordena por antigüedad, los días desde la fecha de la factura, porque las facturas no tienen vencimiento |
| Brecha de días | Días entre la fecha base y la entrega; si no se entregó, hasta hoy en la zona de la empresa. Es negativa si la unidad se entregó antes de su primer pago. La fecha base es la del primer pago que cubre parte de la unidad —por su imputación, o por el reparto de lo no imputado de su factura— y, si ninguno la cubre, la de la factura (guía, columnas 24 y 43; D-199). Así, de una factura de 50 motos pagada en dos bloques de 25, cada bloque cuenta desde su propio pago |

**Qué es la diferencia por inicial.** Mide lo que Millennium no desembolsó porque el cliente pagó su inicial directamente al concesionario: la factura dice 50 000, el concesionario cobra 42 000 y la diferencia son los 8 000 que puso el cliente. Sigue ocurriendo en el canal de dealers y sirve para validar contra ellos las iniciales cobradas. Desde D-248 la inicial se congela al saldar (§10.3): una nota posterior no la achica, y el reporte la lee de la imputación que salda.

Por eso **en ese canal la diferencia debe imputarse a las unidades concretas y no repartirse**. Si un bloque de pago cubre diez autos y solo tres clientes pagaron inicial, el prorrateo le atribuye a las otras siete una inicial que nadie pagó y la validación con el concesionario deja de cuadrar. La imputación por unidad, que en el caso general es opcional, aquí es obligatoria.

La validación en sí cruza dos mundos: esta diferencia, que sale de compras, contra la inicial registrada en el contrato del cliente, que vive en ventas. El reporte se puede construir cuando exista ese bloque.

Ninguno de los dos indicadores se guarda. La brecha **se recalcula siempre con la mejor información disponible**: si el pago se registra dos meses después, la brecha de esa unidad cambia. No es un error sino un dato que faltaba, y congelarla significaría fijar para siempre un número calculado sobre información incompleta. Si hace falta que un reporte mensual sea reproducible, lo que se conserva es el reporte, no el campo.

**Los indicadores de la flota** (D-235) se calculan al leerlos, por empresa, y no se guardan.

| Indicador | Cómo sale |
|---|---|
| % de activación | Activos / (Activos + Paralizadas). Activos es la familia `productivo`; Paralizadas, la familia `recuperacion`: Mecánica, Siniestro, Custodia y Puesta a punto. Robado, Pérdida total, En traspaso (D-239) e Inactivo no entran |
| Unidades por estado | Las unidades no anuladas en cada estado |
| Días promedio por estado | El promedio de cada estadía, y hasta ahora en las que siguen en curso |
| Tolerancia vencida y por vencer | Por el `due_at` de la fila actual de cada unidad, con una ventana de 72 horas |
| Próximas a resolución | Unidades en Custodia con tolerancia vencida o por vencer en esa ventana |
| En riesgo de pérdida | Unidades en Custodia por Incautación, Comisaría o Depósito |

**El costo por día paralizada no se calcula:** el sistema no guarda ningún costo diario de una unidad detenida.

### 10.6 Candidatos del bloque 8

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
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

**Reglas de negocio**

- La placa solo se puede registrar si la unidad tiene el título y el documento de identificación vehicular cargado y vigente. En un país sin ese tipo de documento no se registra hasta decidirlo con su primera empresa, como sus documentos tributarios (D-52, D-188).
- La fecha del título no puede existir sin título.
- Las placas físicas (`plates_received_date`) solo se registran si la unidad ya tiene su placa (guía funcional, columna 17; D-165).
- Registrar el ingreso físico fija la ubicación de la unidad, una por una o en lote, como las unidades de una factura (D-165). Una unidad que ingresó no queda sin ubicación, ni al editarla en Inventario (D-188).
- Ninguna fecha de la habilitación es futura en la zona de la empresa: el ingreso, los hitos, la emisión de un documento ni una instalación (D-186).
- El kilometraje actual no es una columna: es la última lectura (§8.5), como ya lo deduce el código (D-165).
- El ingreso físico se conserva como hito histórico: volver a marcarlo como no ingresado solo sirve para corregir un error.

### 11.2 `vehicle_document_types` — Tipos de documento del vehículo

Qué documentos se piden a cada unidad, con sus reglas: cuál vence, cuál impide venderla, cuál ve el cliente y cuál se pide solo a las unidades a gas. Es por país, como los documentos tributarios (§4.7), porque cada país tiene los suyos. Se llama `vehicle_document_types` para no confundirla con `tax_id_types` ni con `purchase_order_template_document_types` (D-139).

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `country_id` | BIGINT FK → `countries` | No | País cuyo documento es | `TIVE`, `SOAT` y `REV_TECNICA` son documentos de Perú; Colombia y México tienen los suyos. La unidad usa los del país de su empresa (D-139) |
| `code` | VARCHAR(30) | No | Código corto: `TIVE`, `SOAT`, `CERT_GAS`, `REV_TECNICA`. Único dentro de su país | `TIVE` queda fijo desde la semilla: lo leen las reglas de la placa, del impuesto vehicular y de la habilitación (D-74, D-139) |
| `name` | VARCHAR(80) | No | Nombre del documento | — |
| `has_expiry` | BOOLEAN | No | Si vence | Determina si el sistema debe avisar cuando se acerque la fecha |
| `blocks_availability` | BOOLEAN | No | Si sin él la unidad no puede pasar a disponible | Hoy solo el documento de identificación vehicular bloquea. Los demás son obligatorios pero no detienen la venta: el cliente fue explícito en que pueden demorarse |
| `is_visible_to_customer` | BOOLEAN | No | Si el cliente puede descargarlo desde su aplicación | El cliente pidió que pueda ver los documentos de su unidad. No todos corresponde mostrarlos |
| `requires_gas_conversion` | BOOLEAN | No | Si solo se exige a las unidades a gas. Por defecto `FALSE` | El certificado de gas aplica a GNV y a GLP y a ninguna otra unidad. Un booleano cubre ambos casos sin agregar una tabla. Una unidad es a gas si su modelo usa GNV o GLP (`fuel_type`, §7): ninguna fuente del cliente habla de conversiones después de la compra (D-166) |
| `is_active` | BOOLEAN | No | Si se sigue pidiendo | — |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y última modificación | No lleva autor: es un catálogo general, como el resto (D-42, D-73, D-139) |

**Relaciones.** Apunta a `countries`. Le apunta `vehicle_documents`.

**Reglas de negocio**

- Un tipo de documento no se borra: se desactiva. Los documentos ya cargados lo siguen referenciando.
- Solo los marcados como bloqueantes impiden que la unidad pase a disponible. Hoy únicamente el documento de identificación vehicular.
- El código no se repite dentro de su país, y `TIVE` no se corrige (D-74).
- Una unidad usa los tipos del país de su empresa.
- Los crean, corrigen y desactivan quienes tienen `catalogos.crear_tipos_documento`, `editar_tipos_documento` y `desactivar_tipos_documento` (§5.6, D-139).

**Ejemplo** — los de Perú

| code | name | vence | bloquea | visible al cliente |
|---|---|---|---|---|
| `TIVE` | Tarjeta de identificación vehicular | No | Sí | Sí |
| `SOAT` | Seguro obligatorio | Sí | No | Sí |
| `CERT_GAS` | Certificado de conversión a gas | Sí | No | Sí |
| `REV_TECNICA` | Revisión técnica | Sí | No | Sí |

La factura de compra no es un documento del vehículo: la unidad llega a la suya por su línea de factura (§9.4), y el archivo lo guarda `purchase_invoices` (§9.3). Guardarla también aquí repetiría el mismo archivo en dos lugares, lo que §11.6 evita (D-139). Un auto nuevo no tiene revisión técnica y por eso ese documento no bloquea nada.

### 11.3 `vehicle_documents` — Documentos cargados

Cada archivo que se sube para una unidad.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad | — |
| `company_id` | BIGINT | No | Empresa, heredada del vehículo | Para su política de RLS (§2.5, D-165) |
| `document_type_id` | BIGINT FK → `vehicle_document_types` | No | Qué documento es | Un tipo del país de la empresa de la unidad (D-139) |
| `file_path` | TEXT | No | Dónde está guardado el archivo: en `kept_files`, con la empresa de su unidad (D-189) | El archivo no se guarda dentro de la base: se guarda aparte y aquí queda su ubicación. Meter archivos en la base la hace lenta de copiar y de restaurar |
| `file_name` | VARCHAR(255) | No | Nombre original del archivo, recortado. Si llega sin nombre, el código del tipo en minúsculas con la extensión de su contenido: `tive.pdf` | Es el que ve el usuario al descargarlo |
| `mime_type` | VARCHAR(100) | No | Tipo de archivo: PDF, PNG, JPEG o WebP, de a lo sumo 5 MB, verificado por su contenido (D-174) | Determina si se puede mostrar en pantalla o solo descargar |
| `file_size` | BIGINT | Sí | Tamaño en bytes | — |
| `issue_date` | DATE | Sí | Fecha de emisión del documento | — |
| `expiry_date` | DATE | Sí | Fecha de vencimiento | Obligatoria si el tipo vence. Es lo que dispara el aviso de renovación |
| `is_current` | BOOLEAN | No | Si es la versión vigente | Un SOAT renovado no borra al anterior: el nuevo queda vigente y el viejo se conserva |
| `uploaded_by` | BIGINT FK → `users` | No | Quién lo cargó | — |
| `uploaded_at` | TIMESTAMPTZ | No | Cuándo | — |

**Relaciones.** Apunta a `vehicles`, `vehicle_document_types` y `users`.

**Ejemplo** — una unidad cuyo SOAT se renovó:

| documento | archivo | emitido | vence | vigente |
|---|---|---|---|---|
| SOAT | soat_abc123_2025.pdf | 01/03/2025 | 28/02/2026 | No |
| SOAT | soat_abc123_2026.pdf | 01/03/2026 | 28/02/2027 | Sí |
| Tarjeta de identificación | tive_abc123.pdf | 20/02/2025 | — | Sí |

El SOAT anterior se conserva: permite explicar con qué documento circulaba la unidad el año pasado.

**Reglas de negocio**

- Solo la carga inicial guarda un documento sin archivo —la TIVE o el SOAT que el Excel marca sin adjuntarlo—, rotulado «Migrado, sin archivo»; el archivo se sube después como reemplazo (D-28, D-206).
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
| `company_id` | BIGINT | No | Empresa, heredada del vehículo | Para su política de RLS (§2.5, D-165) |
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

- Solo la carga inicial instala un GPS sin serial; el serial se completa una sola vez después (D-206). Esa vez la defiende solo el servicio (`VEHICLEEQUIPMENT_SERIAL_ALREADY_SET`): desde V119 la cuenta de la aplicación puede actualizar la columna `serial` (D-229), y la base no mira si estaba vacía. Un disparador que lo exija queda candidato (§11.10).
- Dos equipos del mismo tipo no pueden estar instalados a la vez en la misma unidad, ni siquiera con períodos que se crucen en el pasado.
- Un equipo no se instala en un tipo de vehículo al que no aplica.
- No se puede instalar en una unidad que no ha ingresado físicamente, ni con una fecha anterior a su ingreso (D-186).
- **El costo lo escribe quien lo ve** (D-187): quien instala sin `stock.ver_costos` lo guarda en cero, en la moneda funcional de la empresa, y quien lo ve lo registra o lo corrige después, con una fila por campo que cambia en la bitácora: `cost` y `currency_id`, este con los códigos ISO de las monedas.
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
| `company_id` | BIGINT | No | Empresa, heredada del vehículo | Para su política de RLS (§2.5, D-165) |
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
- La carga inicial tiene sus propias excepciones (D-177): copia la tasa de cada fila del Excel, no aplica los 20 días de la factura antigua, agrupa las notas de crédito por su código y manda al archivo de errores las notas de débito, que el sistema no modela (D-23). Antes de construirla se perfilan los Excel reales.
- **Cada carga vive en el contexto dueño de lo que actualiza** (D-200): el impuesto vehicular y la garantía mobiliaria, en `fleet`; la carga inicial la orquesta `procurement`, porque las unidades nacen con su factura, y da de alta las unidades y su habilitación por la fachada de `fleet`. Todas registran su lote aquí, por la fachada de `shared`, que sirve la pantalla Cargas de archivos.
- **Entran por fila o por factura** (D-201): el archivo es .xlsx con encabezados exactos. El impuesto vehicular y la garantía, fila por fila. La carga inicial, por factura: las filas de una misma factura entran o fallan juntas, y un VIN ya migrado se informa sin repetirse, así que volver a subir el archivo corregido es seguro.
- **Lo que el Excel no fecha se deduce de su fila** (D-206): el ingreso, de la fecha más temprana entre el GPS, la pantalla, la primera entrega y la declaración de stock, o la de la factura; las placas físicas, de la primera entrega o la declaración de stock, o la del título; la entrada a Disponible, de la declaración de stock (columna 51), a las 00:00 en la zona de la empresa. La ubicación la elige quien carga. Las llaves entran con una etiqueta provisional —el código de la marca comercial y el `N°`—, y ~~una unidad cuyo propietario ya no es la empresa entra como suya, para registrar su traspaso después (D-20)~~ lo del propietario lo reemplazó D-225, enmendada por D-239 (más abajo).
- **Notas y pagos migrados** (D-207): las notas entran como «descuento de campaña», sin mes, y las que dejarían una línea en cero se registran a mano; los pagos, uno por bloque de la columna 42, por transferencia, con tasa 1 en la moneda funcional o, en otra, la del catálogo o la del Excel; en el canal de dealers, imputados por unidad y saldados.
- **El paquete se entrega en tres partes** (D-208): el registro de cargas, el impuesto vehicular y la garantía, después de la habilitación; la carga inicial, después de las notas de crédito; y sus pagos, después de los pagos. En producción la carga inicial corre una sola vez, con los pagos ya construidos. Un archivo es un .xlsx de a lo sumo 10 MB, el límite del servidor, con un tope de filas medido en su línea base, y uno más grande se parte, porque volver a subirlo no repite nada.
- **La unidad migrada entra en su estado** (D-202), por el camino «Migración inicial»: a Activo si se entregó, a Disponible si cumple la habilitación y no se entregó, y si no, a En habilitación.
- **Una unidad con otro propietario terminó su contrato** (D-225, enmendada por D-239): entra en Activo con su fecha de primera entrega y, en la misma carga, pasa ~~a Inactivo~~ a En traspaso por «Contrato completado», con la nota «Propietario en el Excel: … Traspaso por registrar.» en esa fila del historial. Los campos del traspaso quedan vacíos, porque exigen la fecha que el Excel no trae: queda en En traspaso hasta que alguien registra el traspaso (D-223) y la mueve a Inactivo por «Traspaso inscrito» (§8.3). La lista de precarga y el informe la cuentan entre las unidades En traspaso. Sin el camino Activo → En traspaso por «Contrato completado» abierto, la carga falla antes de escribir, nombrando el estado que falta (V123, backend `9f9bd093`).
- **La carga inicial se revisa antes de cargar** (D-226): la lista de precarga dice qué dejaría el archivo sin escribir nada.

### 11.9 El estado de habilitación se calcula

No es un campo: se evalúa en orden y devuelve el primer estado que aplique.

| Orden | Si… | Estado | Código |
|---|---|---|---|
| 1 | No ha ingresado físicamente | Fuera de tienda | `FUERA_DE_TIENDA` |
| 2 | Le falta un documento vigente de un tipo que bloquea la disponibilidad; hoy, solo el documento de identificación vehicular | Sin documento vehicular, con el nombre del que falta | `SIN_DOCUMENTO` |
| 3 | Le falta instalado un equipo de un tipo que bloquea y aplica a su tipo de vehículo; la pantalla, solo si su modelo no la trae de fábrica. Hoy, la pantalla y el GPS | Sin pantalla, Sin GPS: con el nombre de los que faltan | `SIN_EQUIPO` |
| 4 | No tiene sus placas físicas (`plates_received_date`), que exigen la placa registrada | Sin placas | `SIN_PLACAS` |
| 5 | Cumple todo y su entrega no sigue vigente: nunca se entregó, o entró a Puesta a punto después (D-216) | Listo para entrega | `LISTO_PARA_ENTREGA` |
| 6 | Cumple todo y su entrega sigue vigente: su último paso a Activo es posterior a su último paso a Puesta a punto | Entregado | `ENTREGADO` |

**Se corrigió el orden de la guía funcional.** Ahí el documento vehicular se evaluaba después de las placas, pero registrar una placa exige tenerlo, de modo que esa rama nunca se ejecutaba; y el resultado que devolvía no era un nombre de estado sino una frase, así que la columna quedaba vacía. Evaluarlo antes y darle nombre resuelve las dos cosas.

**Reglas de negocio**

- Se recalcula en cuanto cambia cualquiera de sus condiciones. No se guarda ni se edita a mano.
- Las condiciones que impiden vender son las que el cliente enumeró: estar en la empresa, tener las placas puestas y tener GPS, más la pantalla en los autos que la requieren y el documento vehicular (guía funcional, §6). Los documentos no bloquean, salvo el vehicular. Marcar otro tipo de documento o de equipo como bloqueante lo suma a la fila 2 o a la 3 sin programar (D-139, D-165).
- Un documento vigente de un tipo que bloquea cuenta aunque haya vencido: el vencimiento se ve en la pantalla de Habilitación (D-167, D-188). Hoy no cambia nada, porque la TIVE no vence.
- **Cada estado tiene un código estable** (D-171), con el que se filtra y se exporta; el detalle dice qué falta: el nombre del documento o de los equipos.
- **El estado gobierna la entrada a Disponible** (D-164, D-183): un cambio a mano hacia un estado con `requires_enablement` exige que no falte nada —ninguna de las filas 1 a 4—, esté la unidad *Listo para entrega* o *Entregado*.
- La fecha en que la unidad llegó a *listo para entrega* sale del historial de estados, no de un campo que alguien escribe.
- **La fecha de entrega también sale del historial**: es el momento en que la unidad pasó a Activo, que es la transición que el flujograma llama entrega al cliente. La brecha de días mide hasta la primera entrega, también en una unidad recolocada, porque compara el pago de su compra.
- **Un mismo nombre, dos cosas:** el *Entregado* de esta sección dice que la unidad está con un cliente. En las reglas del registro de clientes, ENTREGADO es un contrato pagado y traspasado, e INACTIVO un contrato terminado. El bloque 12 tiene que fijar esa equivalencia.
- **Puesta a punto cierra la entrega** (D-216): Millennium coloca sus unidades en leasing, y una unidad recuperada puede entregarse a otro cliente. Al entrar a Puesta a punto deja de leerse *Entregado*, y la siguiente entrega abre una nueva. La puerta no cambia: acepta las dos filas. No hay un campo de fecha de entrega y no debe haberlo, porque sería un segundo lugar donde el dato puede discrepar.

### 11.10 Candidatos del bloque 9

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `checksum` | `vehicle_documents` | Ya no hace falta: lo dicen `kept_files.checksum_sha256` y la llave del archivo, que lleva su SHA-256 (§2.7) |
| `reminder_days` | `vehicle_document_types` | Que se configure con cuánta anticipación avisar de cada vencimiento |
| `insurance_policies` y su cobertura | — | Que se contrate seguro vehicular de flota, donde una póliza cubre muchas unidades con vigencias distintas |
| Un disparador `BEFORE UPDATE OF serial` | `vehicle_equipments` | Que la base, y no solo el servicio, exija que el serial se completa una sola vez y solo sobre uno vacío (D-206, D-229). Desde V119 la cuenta de la aplicación puede escribir esa columna. Lo admitiría un segundo camino de escritura del serial, o la auditoría que lo pida (09/11/2026) |

---

## 12. Bloque 10 · Fondos y garantía mobiliaria

Los vehículos se ceden en garantía a un fondo, y lo que interesa es **a cuál y desde cuándo**. La garantía no se retira hasta que termina el crédito.

### 12.1 `funds` — Fondos

Catálogo administrable. Hoy hay uno; se esperan más.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `company_id` | BIGINT FK → `companies` | No | Empresa que cede | — |
| `code` | VARCHAR(30) | No | Código corto: `PF_MILLENNIUM`, `BID` | Se corrige mientras ninguna cesión lo usa; después queda fijo (§2.4, D-134) |
| `name` | VARCHAR(150) | No | Nombre del fondo | — |
| `fund_type` | VARCHAR(30) | No | Tipo del fondo, de una lista cerrada en minúsculas: `fideicomiso`, `fondo_propio` o `banco` | Agrupa los fondos y se traduce en pantalla y en la hoja de cálculo. Un tipo nuevo llega con una migración que amplía el `CHECK` (D-133) |
| `is_active` | BOOLEAN | No | Si se siguen cediendo unidades a él | — |
| `created_at` / `updated_at` | TIMESTAMPTZ | No | Fechas de registro y modificación | Responden desde cuándo existe el fondo y cuándo cambió por última vez (D-133) |

**Relaciones.** Apunta a `companies`. Le apunta `vehicle_fund_assignments`.

**Ejemplo**

| code | name | tipo | activo |
|---|---|---|---|
| `PF_MILLENNIUM` | PF Millennium | Fideicomiso | Sí |
| `BID` | Banco Interamericano de Desarrollo | Banco | Sí |

**Reglas de negocio**

- El fondo es una fila, no un valor escrito en el código. Cuando entre otro, se carga y ya.
- Un fondo no se borra: se desactiva. Las unidades que estuvieron cedidas siguen siendo explicables.
- El código no se repite dentro de la empresa. Se corrige mientras ninguna cesión lo usa, como el de cualquier catálogo propio (§2.4): un `PF_MILLENIUM` mal escrito se arregla antes de ceder la primera unidad (D-134).
- Un fondo es de todas las marcas de su empresa, como un proveedor: lo crean, lo editan y lo activan o desactivan solo las asignaciones de toda la empresa y sin marca, y quien lo escribe tiene que poder leerlo con `fondos.ver` (§2.5, §5.6, D-134).
- A una empresa inactiva no se le crea un fondo ni se le reactiva uno (D-84, D-94).
- La lista de fondos se exporta a una hoja de cálculo con el alcance de su lectura, `fondos.ver` (D-116, D-134).

### 12.2 `vehicle_fund_assignments` — Cesiones

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `vehicle_id` | BIGINT FK → `vehicles` | No | La unidad cedida | — |
| `company_id` | BIGINT FK → `companies` | No | La empresa de la unidad y del fondo, copiada de la unidad al ceder | Las cesiones se consultan por sí solas —por fondo, para su auditor— y crecen con cada archivo: con su propia empresa, la base las separa por empresa sin pasar por la unidad (§2.5, D-111, D-133) |
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
- La unidad y el fondo son de la misma empresa: la base lo exige con llaves compuestas hacia `vehicles (id, company_id)` y `funds (id, company_id)`, así que una cesión a un fondo de otra empresa no se puede guardar (D-133).
- La llave de `import_batch_id` llega con las cargas del bloque 9. Hasta entonces las cesiones se registran a mano y la columna queda vacía (D-133).
- **Quién cede una unidad** (D-134, excepción a D-101). Ceder una unidad a un fondo y cerrar la cesión son actos financieros de la empresa: `fondos.registrar_cesion` actúa sobre toda la empresa o sobre la marca de la unidad, nunca sobre la sede donde está la unidad. Un rol de sede ve la unidad y sus cesiones, pero no la cede.
- Las cesiones de una unidad son parte de su ficha: se leen con `stock.ver` donde se ve la unidad, marca incluida (D-102).
- El fin de una cesión es un hecho del negocio —el fin del crédito—, no el momento en que se registra: quien la cierra elige la fecha, que excluye ese día y es posterior al inicio (§2.6).
- La cesión no se retira hasta que termina el crédito. Que un cliente incumpla no la levanta.
- **El archivo que se carga actualiza la historia, no la reemplaza.** Que una placa desaparezca del último archivo cierra su cesión con esa fecha; no borra que estuvo cedida. La guía funcional lo trata como una fotografía que sobrescribe, y así se pierde desde cuándo estaba cedida cada unidad, que es justamente lo que se pidió conservar.
- El archivo solo actualiza unidades existentes: nunca crea vehículos.
- La lista cierra también las cesiones registradas a mano: si una unidad con placa no figura, su cesión se cierra con la fecha de la lista; la de una unidad sin placa no se cierra y se informa, y una placa cedida a otro fondo falla sin tocar nada (D-209).

### 12.3 Candidatos del bloque 10

| Campo | Tabla | Qué lo admitiría |
|---|---|---|
| `contract_reference` | `vehicle_fund_assignments` | Que haga falta referenciar el contrato de cesión con el fondo |
| Acceso de terceros | — | Que un fondo o su auditor entre al sistema. La asignación de accesos ya admite dimensiones opcionales: se agregaría el fondo como una más |


---

## 13. Bloque 11 · Bitácora de cambios

Las tablas de historial que ya existen —estados del vehículo, cesiones a fondos, equipamiento, accesos, bloqueos de cuenta— responden **qué pasó**. Ninguna responde **qué hizo alguien con un registro**: si corrigió el motivo de un bloqueo, movió una fecha de cesión o cambió una placa mal digitada.

Esta tabla cubre eso. Son dos auditorías distintas y ninguna reemplaza a la otra.

### 13.1 `change_logs` — Registro de modificaciones

La tabla se llama `change_logs`, en plural, como la nombra la estrategia de nombres del backend para toda tabla de un agregado (`vehicles`, `permissions`, `role_permissions`). Hasta la v1.25 el diccionario la escribía `change_log`.

| Variable | Tipo | Nulo | Descripción | Por qué importa |
|---|---|---|---|---|
| `public_id` | UUID | No | Identificador público de la fila | Es el que sale por la API (regla 5.6); el `id` interno no sale del servidor |
| `entity` | VARCHAR(50) | No | Qué tabla se modificó | — |
| `entity_id` | BIGINT | No | Qué registro | No es una llave foránea: la tabla apunta a cualquier entidad, y una llave a todas sería imposible de declarar. No sale por la API |
| `entity_public_id` | UUID | No | El identificador público del registro modificado | Es por el que la API filtra la bitácora: la ficha de una unidad enlaza a su historia con `entity=vehicles&entityPublicId=…`. Como `entity_id` es interno y no sale del servidor, sin esta columna la bitácora de un registro no se podría pedir. Todo el que escribe en la bitácora la llena |
| `company_id` | BIGINT FK → `companies` | Sí | A qué empresa pertenece el registro modificado | Permite que cada empresa vea solo su propia bitácora. Vacío para cambios sobre catálogos generales |
| `commercial_brand_id` | BIGINT FK → `commercial_brands` | Sí | La marca del registro modificado, cuando pertenece a una | D-102: la marca de una asignación acota también la bitácora; vacía para lo que no es de una marca. Es de la misma empresa de la fila (llave compuesta sobre `commercial_brands (id, company_id)`), y una marca sin empresa no se admite |
| `field_name` | VARCHAR(60) | No | Qué campo cambió | Un cambio que toca tres campos deja tres filas: así se puede buscar quién tocó una placa sin leer el resto |
| `old_value` | TEXT | Sí | Valor anterior | Vacío cuando el campo estaba sin llenar |
| `new_value` | TEXT | Sí | Valor nuevo | Vacío cuando se borró el contenido |
| `reason` | TEXT | Sí | Por qué se hizo el cambio | Obligatorio en los cambios marcados como delicados: corregir un VIN, una placa o una tasa de cambio, o anular una factura, una nota de crédito o un pago (D-161, D-193, D-198) |
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
- Los cambios marcados como delicados exigen motivo: corregir un VIN, una placa, una tasa de cambio (D-149) o una fecha ya reportada, y anular una factura (D-161) o una nota de crédito (D-193).
- No guarda contraseñas ni el segundo factor, aunque cambien.
- Cada empresa ve solo la bitácora de sus propios registros.
- Un campo cuyo valor solo se ve con un permiso —un dato personal, un costo, un pago— lo marca el contexto dueño, no `shared`: declara el campo y el permiso que muestra sus valores por un puerto de la bitácora. Quien lee la bitácora sin ese permiso donde actúa en la empresa de la fila, y en su marca cuando el permiso va por marca, ve que el campo cambió, sin el valor anterior ni el nuevo, y la fila trae `valuesWithheld: true`. La marca se lee al consultar, así que cubre también las filas ya escritas. Las marcas son el teléfono de Atención de una plantilla (`purchase_order_templates.attentionPhone`), con `configuracion.ver_datos_personales_colaboradores` (D-105); el costo y la moneda de una instalación (`vehicle_equipments.cost` y `currency_id`), con `stock.ver_costos` en la marca de la unidad (D-102, D-187); y el monto de una aplicación de pago (`payment_applications.amount`), con `pagos.ver`.
- Qué cambio es delicado lo dice el contexto que lo hace, que es dueño de esa regla —la flota marca la corrección de una placa o de un VIN; compras, la anulación de una factura o de una nota de crédito; localización, la de una tasa de cambio—, no la bitácora: la bitácora solo exige el motivo de un cambio marcado.
- Una fila sin empresa, un cambio sobre un catálogo general, la ve solo quien tiene `bitacora.ver` con alcance sobre todas las empresas. Quien lo tiene en algunas empresas ve solo las filas de esas empresas (D-99).
- Quien tiene `bitacora.ver` en una empresa acotado a una marca ve las filas de esa marca y las que no son de ninguna (D-102). La marca es la del registro cuando se hizo el cambio.
- La bitácora se filtra por un período `[desde, hasta)`, con el fin excluido, como todo período (§2.6).

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
| `ip_address` y dispositivo | `change_logs` | Que se exija trazabilidad de seguridad y no solo de negocio |
| Registro de consultas | — | Que haya que auditar quién vio datos sensibles, no solo quién los cambió |


## 14. Decisiones

Quedan ~~tres~~ dos decisiones abiertas, cada una con su momento fijado. Las longitudes y el dígito de control de los documentos tributarios de Colombia y México (§4.7, D-75), antes de registrar la primera empresa de cada país (D-52). Cómo lista el registro público, sin sesión, las sedes y los tipos de documento (convenciones de la API, §14.16), antes de encenderlo en un entorno (§5.13, D-83). ~~Cómo entra por la carga inicial una unidad con otro propietario (D-225, §11.8), que pasaba a Inactivo por «Contrato completado», un camino que D-239 cierra, antes de integrar V123.~~ Cerrada el 09/11/2026 con V123 (backend `9f9bd093`), por una enmienda de D-225: la unidad pasa a En traspaso por «Contrato completado», con la nota «Propietario en el Excel: … Traspaso por registrar.», y espera ahí su traspaso (§11.8, D-239). El período anterior de un colaborador que regresa quedó decidido y construido el 09/11/2026 (D-241). Quedan con el responsable si entra `termination_reason`: la historia de los períodos existe y no muestra el motivo (§5.10); si el contador del inicio de sesión de la cuenta vuelve a cero cuando la persona restablece su contraseña con un enlace, que hoy no lo hace (§5.4); y, antes del primer correo real, el remitente, el SPF y el DKIM de `millennium.ky` y la cuenta de SendGrid (D-242): hasta entonces, un entorno desplegado con `capturing` no envía ningún correo (§2.7). Queda con contabilidad, con la Consulta RUC, si Millennium es agente de retención del IGV: hasta que lo confirme, la retención de un pago sigue opcional, la pantalla la propone en el 3 % de lo que se paga y se corrige a mano, y el backend no la calcula (§10.2; D-196, D-246). Si lo es, calcularla pide más de lo que D-196 suponía (revisado el 13/11/2026): el monto ya tiene su columna, pero no se retiene a un proveedor que es buen contribuyente o agente de retención, y `suppliers` no guarda esa condición, así que haría falta un dato del proveedor o la consulta de su RUC, además del umbral de S/ 700 de la operación. Si no lo es, la pantalla deja de ofrecer «+ Retención del IGV» y la columna queda para lo ya registrado. Queda con el responsable si «Por pagar», la palabra del formulario del pago desde D-246, reemplaza a «Pendiente» de D-245 en las demás pantallas de Compras y en la ficha de la unidad. Quedan como candidatos, sin fecha: la llave de idempotencia de `notifications`, que hoy cumple NTF-R04 con la reserva del intento (§2.7), y un disparador que defienda en la base que el serial de un equipo se completa una sola vez (§11.4, §11.10; D-206, D-229). El certificado de gas y el aviso de renovación, que esperaban al paso 9, quedaron decididos el 02/11/2026 (D-166, D-167). La matriz de permisos por rol quedó fijada el 19/10/2026 (§5.6, D-95 a D-105), y el 20/10/2026 sumó Seguridad, con sus doce permisos, sus categorías y sus pares incompatibles (D-117 a D-124, D-126); su celda pendiente, quién registra la entrega al cliente, quedó decidida el 09/11/2026: también el asesor de ventas, en su sede (`stock.entregar`, D-238). Lo que falta para construir son datos: los datos maestros reales —razón social y documento tributario de la empresa, sedes con su código de establecimiento, marcas, proveedores, colaboradores— y las tolerancias de los caminos de estado, que pide al negocio la especificación del 09/11/2026 (D-240). La lista oficial de divisiones de Perú está cargada; los distritos que crean las leyes después entran cuando tienen código oficial (§4.5, D-77).

### Decisiones ya cerradas

| # | Decisión tomada |
|---|---|
| D-1 | El país se identifica internamente con un número; el código internacional se guarda aparte como valor único |
| D-9 | El VIN tiene 17 caracteres exactos. Enmendada por D-160: sin I, O ni Q (ISO 3779), como ya valida el código |
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
| D-35 | Las órdenes de compra se emiten con plantillas. Todas tienen los mismos campos; sus valores cambian según la línea de negocio y el tipo de orden (vehículos, taller, otros), que es un catálogo administrable. Los datos del emisor se copian de la empresa al crear la plantilla y no se editan (D-134); los contactos de atención y soporte son texto. La observación, el descuento por orden, el contacto del proveedor, las líneas que se retiran y los códigos de estado los fija D-132 (§9.2) |
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
| D-46 | Los tipos de cambio por día y la moneda de grupo quedan diseñados en §4.8 y se construyen cuando aparece quien los usa: los tipos de cambio justo antes de compras (paso 7) y la moneda de grupo con los reportes (paso 11). La moneda de grupo es USD. Desde D-148, los tipos de cambio son por país y se cargan a mano, y la carga automática espera una fuente con acceso permitido |
| D-47 | Los dos códigos ISO de un país nombran un mismo país de ISO 3166. Se corrigen mientras el país no tenga escalones, direcciones ni tipos de documento tributario, lo que también excluye divisiones y empresas; desde entonces quedan fijos. `location` lo decide sin consultar a `organization`, para no volver al ciclo que D-3 eliminó. Es la única excepción con nombre a la regla de que los códigos de normas externas no se editan; extenderla requiere una decisión nueva (§2.4, §4.2, D-48). Enmendada por D-155: las tasas de cambio también fijan los códigos (§4.8) |
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
| D-63 | Un superadministrador o el administrador de accesos puede restablecer la contraseña de otra cuenta: el sistema genera una provisional, la muestra una sola vez, obliga a cambiarla y cierra todas las sesiones de esa cuenta. Nadie restablece así la suya, la de `SYSTEM` no se restablece, y el administrador de accesos no la restablece a una cuenta protegida (§5.4, §5.12). Enmendada por D-243: ya no se muestra una provisional; se envía un enlace de un solo uso al correo de la cuenta, la contraseña anterior deja de valer y la cuenta sigue obligada a elegir su contraseña hasta usar el enlace (§5.4) |
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
| D-74 | Un código que lee una regla del programa cuenta como algo que lo usa (D-48): queda fijo desde la semilla que lo crea. En `equipment_types` son `GPS` y `PANTALLA`, porque las reglas de habilitación reconocen el GPS y la pantalla por su código (§7.5, §11.9). El código de cualquier otro equipo se corrige mientras ninguna instalación lo use, y desde la primera queda fijo. La API rechaza la corrección y responde si el código todavía se puede corregir, para que la pantalla bloquee el campo (§2.4). Desde D-139 también queda fijo `TIVE` en `vehicle_document_types`, que leen las reglas de la placa, del impuesto vehicular y de la habilitación (§11.1, §11.7, §11.9) |
| D-75 | El dígito de control lo calcula `location` según la sigla del tipo dentro de su país y lo ofrece por su fachada. Hoy existe un solo cálculo: el del RUC de Perú, con el módulo 11 de SUNAT (pesos 5, 4, 3, 2, 7, 6, 5, 4, 3, 2). Un tipo con dígito de control cuyo cálculo todavía no existe —el NIT, el RFC y la CURP— acepta sus números sin verificar hasta que llegue su cálculo, que se resuelve con sus longitudes antes de la primera empresa de cada país (D-52). Lo aplica quien guarda el número: `organization` en empresas y colaboradores, y `procurement` en proveedores. El emisor de una plantilla es una copia del documento de su empresa, ya verificado, y no se verifica de nuevo (D-134). Se verifica al crear el registro y cuando cambian el número o el tipo; los números ya guardados siguen siendo válidos (§4.7, §6.2) |
| D-76 | El DNI de Perú no lleva dígito de control: sus 8 dígitos no traen uno dentro. El carácter de verificación que va impreso en el documento es aparte y no se guarda (§4.7) |
| D-77 | La lista oficial de divisiones de Perú es la edición 2022 del INEI más los distritos creados por ley después, revisada al 17/10/2026. Cada distrito nuevo entra activo con su propia migración en cuanto el INEI o SUNAT publican su código, y el distrito del que se separó sigue activo; hasta entonces no se puede elegir. El primero es Santa Rosa de Loreto (Ley 32403, 03/07/2025), que todavía no tiene código publicado. La lista se revisa en el cierre de cada paso, contra las leyes nuevas y las ediciones nuevas del INEI (§4.5, D-2) |
| D-78 | La longitud de un tipo de documento tributario, cuando la declara, va de 1 a 20: lo que caben las columnas que guardan los números (`companies.tax_id`, `employees.document_number`, `suppliers.tax_id` y `purchase_order_templates.issuer_tax_id`). Una longitud mayor declararía un tipo que ningún número puede cumplir (§4.7) |
| D-79 | Las ubicaciones siguen D-8 como las empresas y las sedes: corregir modifica la dirección y mudarse crea una nueva. La dirección de una ubicación cambia solo por su propia operación, como la de una sede, y al cambiarla la pantalla pregunta cuál de las dos es. Vale para las ubicaciones de terceros; una propia usa la dirección de su sede. Una ubicación no pasa de propia a terceros ni al revés: es otro lugar, así que la anterior se desactiva y se crea una nueva (§4.6, §6.5) |
| D-80 | La tabla de direcciones queda acotada antes de guardar la primera dirección de un proveedor (paso 7), no recién con los clientes: desde ahí guarda direcciones de terceros de cada empresa, y la frontera entre empresas la defiende también la base (D-66). Se acota por el país de la dirección, con una rama `SYSTEM` (V78, D-134): con una empresa por país (D-5) y proveedores locales (D-133) equivale a acotar por empresa, sin columna nueva. Es la excepción anotada a la regla de `company_id` (§2.5). Si D-5 o el proveedor local cambian, `addresses.company_id` (candidato, §4.8) |
| D-81 | `roles.company_id`, los roles propios de una empresa, sale del paso 2 y llega con la matriz de permisos (D-68), antes del paso 5: un rol propio de una empresa solo tiene sentido con permisos que marcarle. Con la matriz se deciden también quién crea esos roles y la unicidad del código por empresa. Hasta entonces todos los roles son generales (§3, §5.5, §5.13) |
| D-82 | Dar acceso vincula un colaborador a una cuenta que ya existe cuando esa cuenta pertenece a un colaborador de otra empresa del grupo con la misma sigla de documento y el mismo número. Se compara la sigla del tipo y no el tipo, porque los tipos son de cada país y cada empresa es de un país (D-5). Si no coinciden, solo un superadministrador vincula la cuenta, después de ver el nombre, el documento y la empresa del colaborador que ya la tiene y de confirmar que es la misma persona; la confirmación queda registrada con quién la dio, cuándo y qué colaborador tenía la cuenta. Una cuenta que existe sin ningún colaborador también la vincula solo un superadministrador, que ve que no tiene titulares y confirma. La confirmación vale para los titulares que vio: si cambiaron antes de confirmar, no se vincula y se le muestran los de ahora para que confirme de nuevo. A cualquier otro se le rechaza. La excepción es el primer superadministrador, que se vincula a sí mismo sin colaborador con el cual comparar (D-70). Un vínculo equivocado lo corrige solo un superadministrador con la desvinculación, que funciona como una salida: retira los roles de la cuenta en esa empresa con motivo `retiro` y, si no le queda ningún colaborador activo, la cierra como D-60, con la protección del último superadministrador (D-65). Cada vínculo confirmado y cada desvinculación dejan una fila en `employee_account_link_events`, escrita en la misma operación que el cambio y que nunca se edita ni se borra; un vínculo confirmado guarda además los titulares que confirmó el superadministrador, una lista vacía si la cuenta no tenía. Esas filas las leen solo el superadministrador y los procesos automáticos, porque guardan datos de personas de otras empresas. Enmendada por D-117 y D-122, solo en el lenguaje: confirmar el vínculo y desvincular son los permisos `seguridad.confirmar_vinculo` y `seguridad.desvincular_cuentas`, que tiene solo el superadministrador y no se marcan en un rol propio; quién lee esas filas no cambia (§2.5, §5.3, §5.4, §5.7, §5.9, D-38, D-58, D-69) |
| D-83 | El registro público con perfil (D-32) queda fuera del alcance del paso 3. Sigue apagado, y se construye antes de encender el registro público en cualquier entorno (§3, §5.2, §5.13). Desde el 31/10/2026, encenderlo exige además que responda lo mismo exista o no el correo y que pase por el freno del inicio de sesión (D-144) |
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
| D-97 | Los roles del sistema no se renombran ni se desactivan: su nombre y su estado los fijan las migraciones. `roles.is_active` existe: un rol propio inactivo no se otorga y sus asignaciones vigentes siguen válidas. Enmendada por D-124: el rol propio puede nacer como copia exacta de uno del sistema, salvo de `SUPERADMIN` y `ADMIN_ACCESOS`. Enmendada por D-131: el superadministrador sí cambia los permisos de un rol del sistema que él no tiene vigente (§5.5) |
| D-98 | Los roles propios de una empresa los crea solo el superadministrador, eligiendo la empresa al crearlos; el administrador de empresa y el de accesos los leen. El código es único dentro de cada empresa y entre los generales, nunca el de un rol del sistema; un rol de una empresa no se asigna en otra ni es de alcance sistema. Cierra la decisión que D-81 dejó abierta. Enmendada por D-122 y D-123: los crea quien tiene `seguridad.gestionar_roles` con alcance sistema —hoy solo el superadministrador— y los lee quien tiene `seguridad.ver_roles`, donde actúa (§5.5) |
| D-99 | El permiso dice qué y la asignación dice dónde, también al leer: una lectura de recursos de empresa se filtra por el permiso y el alcance de la asignación que lo trae; la frontera entre empresas de D-66 queda como piso. Un rol de sede lee toda su empresa (D-14) y actúa solo en su sede (§5.8 regla 6) |
| D-100 | Cada camino de estado nombra el permiso que exige (`status_change_reasons.requires_permission_id`, candidato de §8.6 adoptado); un camino sin permiso propio exige `stock.cambiar_estado`. Quién recorre cada camino es un dato, como los caminos (D-19), y lo mantiene solo el superadministrador con `catalogos.gestionar_estados`. Con los caminos de ejemplo de §8.3: habilitar, vender, entregar, atender en taller y recuperar (§5.6, §8.3). D-130 partió `gestionar_estados` en `crear_estados`, `editar_estados` y `desactivar_estados`; los tipos de documento del vehículo tienen los suyos (D-139) |
| D-101 | Un rol de sede actúa sobre las unidades que están en la ubicación propia de su sede; las que están en una ubicación de terceros, o sin ubicación, las tocan los roles de empresa. Si un caso real pide que una sede responda por un patio arrendado o una tienda de dealer, se decide un dato nuevo, distinto de `locations.branch_id`, que sigue siendo solo de las propias (§2.5, §6.5). **Excepción:** ceder una unidad a un fondo y cerrar la cesión no siguen la sede: los hace una asignación de toda la empresa o de la marca de la unidad, porque son actos financieros de la empresa (§12.2, D-134) |
| D-102 | La marca de una asignación acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, aplicada en el código y no en la base; un rol de empresa sin marca ve todas. Los vehículos son el primer dato con marca, en el paso 5 (§2.5, §5.7, §5.13) |
| D-103 | El tipo de cambio lo corrige a mano el superadministrador, con `catalogos.corregir_tipo_cambio`: `exchange_rates` no tiene empresa y una tasa sirve a todo el sistema, así que un rol de empresa no podría acotarse. Si el negocio quiere que la contabilidad lo corrija, será un rol general nuevo o una regla de acotación nueva (§4.8). D-149 toma el primer camino: la persona de Contabilidad que carga las tasas recibe sus dos permisos por un rol propio general, de alcance sistema, y D-150 impide marcarlos en un rol de empresa |
| D-104 | Una unidad se anula por las dos vías que describe el diccionario: como consecuencia de una nota de crédito que deja su valor en cero, gobernada por `compras.registrar_nota_credito`, o por acción explícita, que exige `stock.anular_vehiculo` y el candado de §9.6 (§8.1, §9.6). Precisada por D-161: anular su factura no anula la unidad, la deja sin factura vigente. Precisada por D-191: la primera vía solo con el motivo «anulación de unidad» |
| D-105 | Lo que cada rol ve de los colaboradores, que D-93 dejó a la matriz: la fila sin sus datos personales —nombre, puesto, sede, fechas de ingreso y salida, si está activo y si tiene cuenta— la lee cualquier rol con `configuracion.ver_colaboradores`, que la semilla da a los siete, para los selectores y los nombres de autor; el documento, el teléfono, la foto y la exportación, solo con `configuracion.ver_datos_personales_colaboradores`, que tienen los dos roles globales y el administrador de la empresa. El campo que no se puede ver no viaja (§2.5, §5.3). **Excepción:** el PDF de una orden de compra enviada es una sola copia fiel, con el teléfono de Atención, para todo el que lee la orden: el documento ya salió hacia el proveedor con ese dato, así que ocultarlo adentro no protege nada, y lo que recibe el proveedor no depende de quién pulsó «Enviar» (§9.2, D-134). El borrador sigue la regla: lleva el teléfono solo para quien ve los datos personales. La bitácora también: el cambio del teléfono de Atención de una plantilla se registra para todo el que lee la bitácora, y su valor solo viaja a quien ve los datos personales de los colaboradores en la empresa de la plantilla (§13.1) |
| D-106 | Todo documento que sale del sistema —una hoja de cálculo, un PDF u otro formato— lo genera el backend, y el frontend lo descarga. Un pedido igual —el mismo tipo, formato, criterios, alcance efectivo, idioma, zona y versión de los datos— reutiliza el documento ya generado, o espera al que se está generando: nunca se genera dos veces. El estado de cada documento vive en PostgreSQL (`generated_documents`); sus archivos, en la base o en DigitalOcean Spaces, nunca en la memoria ni en el disco del proceso, porque en App Platform el disco es efímero y en cada despliegue conviven la instancia vieja y la nueva (§2.7; convenciones de la API, §8) |
| D-107 | El tipo de vehículo de un modelo se corrige mientras el modelo no tenga unidades; desde la primera queda fijo, el mismo principio que D-48 y D-74. `fleet` se lo avisa a `vehiclecatalog` por su fachada al registrar la primera unidad del modelo, y `vehiclecatalog` guarda el hecho en `vehicle_models.has_units`: no lee las tablas de `fleet` (§7.3) |
| D-108 | No se crea un modelo de una marca inactiva ni de un tipo de vehículo inactivo, y no se registra una unidad cuyo modelo, marca o tipo estén inactivos. Lo retirado que un registro ya tiene se conserva al editarlo, como D-84 (§7.1 a §7.3, §8.1) |
| D-109 | El combustible de un modelo nuevo no se propone: nace vacío y se elige, y queda vacío cuando no se conoce (§7.3) |
| D-110 | El regreso automático escribe su fila del historial con un motivo del camino que recorre: el motivo automático reservado del camino que va del destino al estado configurado (`status_change_reasons.is_automatic`, código `REGRESO_AUTOMATICO`). Lo crea el sistema, no se ofrece a las personas y no se edita. Reemplaza la propuesta 6 del documento del contexto `fleet` (sección 6 b), que firmaba el regreso con el motivo del camino que venció (§8.3, §8.4) |
| D-111 | `vehicle_status_history` y `vehicle_mileage_readings` llevan su `company_id`, obligatorio, aunque se pueda deducir de la unidad: se consultan por sí solas y acumulan volumen (§2.5). Su Row Level Security lo usa directamente, sin pasar por `vehicles`. La empresa de una unidad no cambia, así que la de sus filas nunca discrepa con la suya (§8.4, §8.5) |
| D-112 | ~~Marcas y modelos es una lista de marcas más la ficha de cada marca, con sus modelos dentro de la ficha, como Empresas (DESIGN.md, revisión 13); no es un maestro-detalle (procesos y pantallas, apartado 3.2)~~ Reemplazada por D-136 el 29/10/2026 |
| D-113 | `/vehicle-statuses` y `/status-change-reasons` son listas cortas sin página: la tercera excepción a la regla de buscar y paginar en el servidor (convenciones de la API, §6.2). El criterio: un catálogo corto del sistema que se necesita entero para nombrar las referencias. Los caminos se filtran en el servidor con `?fromStatusId={id}` y `?origin=registration` (convenciones §14.23, adoptado) |
| D-114 | Los códigos de error de un agregado llevan un solo prefijo, sin guion bajo interno: `VEHICLESTATUS_*` y `STATUSCHANGEREASON_*`, los que ya produce la fábrica de `*_NOT_FOUND`. Así cada agregado tiene un prefijo y no dos (convenciones de la API, §7.2) |
| D-115 | `allowedActions` de la unidad y `vehicleTypeLocked` del modelo son datos derivados del recurso, que calcula su dueño y nunca se guardan, como `isoCodesLocked` del país y `codeLocked` del tipo de equipamiento (convenciones de la API, §4.2). El kilometraje de la unidad (`currentMileage`) no es uno de ellos: es una de las dos excepciones que se guardan además en la unidad (§2.2, §11.1) |
| D-116 | Todo catálogo y toda lista principal se exporta a una hoja de cálculo desde su propio endpoint: también los cinco catálogos de flota, los modelos de una marca y el inventario de unidades. El exportable de stock del paso 11 es otro documento (D-106; convenciones de la API, §8). Los proveedores y los fondos se exportan desde el 26/10/2026, con el alcance de su permiso de lectura. Pendiente con fecha: la lista de órdenes de compra y la de plantillas se exportan en el paso de reportes (D-134). Desde D-136 las marcas ya no se exportan por separado: la exportación de los modelos lleva su marca |
| D-117 | Seguridad se autoriza por permisos, como el resto del sistema: doce permisos del módulo `seguridad` reemplazan a los roles que exigían las cuentas, los accesos, los colaboradores y los roles —las 18 anotaciones de rol de `iam` y `organization`—. Son `ver_cuentas`, `ver_roles`, `dar_acceso`, `gestionar_colaboradores`, `gestionar_roles`, `bloquear_cuentas`, `registrar_salidas_y_regresos`, `asignar_roles`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos`. La semilla da once al superadministrador —todos salvo `asignar_roles_sin_tenerlos`—; siete al administrador de accesos —`ver_cuentas`, `ver_roles`, `dar_acceso`, `bloquear_cuentas`, `asignar_roles`, `restablecer_contrasenas` y `asignar_roles_sin_tenerlos`—; y cuatro al administrador de empresa —`ver_cuentas`, `ver_roles`, `gestionar_colaboradores` y `registrar_salidas_y_regresos`—: 22 marcas, y las marcas vigentes de los roles del sistema pasan de 76 a 98. `confirmar_vinculo` y `desvincular_cuentas` son solo del superadministrador, y `asignar_roles_sin_tenerlos` solo del administrador de accesos: ninguno de los tres se marca en un rol propio. Los límites sobre el destino —la propia cuenta, una cuenta protegida, `SYSTEM`, el último superadministrador— quedan fijos en el código. No se siembran permisos de pasos que todavía no existen. Enmienda D-95 y la regla 7 de §5.8 (§5.6, §5.8) |
| D-118 | Cada permiso lleva su categoría (`permissions.category`), que ordena la matriz por lo que hace: `ver`, `crear`, `editar`, `gestionar` —crear, editar y desactivar un recurso; un `gestionar_*` no se parte—, `desactivar_anular` o `acciones_especiales`. No hay "eliminar", porque nada se borra. Los 25 permisos sembrados hasta el paso 5 quedan 6 de ver, 2 de crear, 1 de editar, 6 de gestionar y 10 de acciones especiales, y los doce de seguridad llevan la suya (D-117). La categoría es obligatoria y la trae cada permiso en la migración de su paso (§5.6) |
| D-119 | La separación de funciones se escribe como pares de permisos incompatibles, en `permission_incompatibilities`, y reemplaza a D-71. Son catorce pares: cada uno de `seguridad.gestionar_colaboradores` y `seguridad.registrar_salidas_y_regresos` con cada uno de `dar_acceso`, `asignar_roles`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos`, para que quien registra a los colaboradores no sea también quien les da acceso. Un rol propio nunca incluye las dos partes de un par. Una cuenta nunca las junta entre sus roles vigentes, en ninguna empresa; las asignaciones de `SUPERADMIN` quedan fuera de esa suma, porque el superadministrador hace las dos cosas (D-57). Se revisa al guardar un rol —también cuando editarlo haría que un titular junte un par, que se rechaza— y al asignarlo. Para que dos operaciones simultáneas no lo burlen, guardar un rol, asignar uno, restablecer una contraseña y otorgar un acceso toman un mismo candado de transacción, y guardar un rol bloquea además cada cuenta titular, en orden de identificador (§5.5 a §5.7) |
| D-120 | La excepción a la regla 2 de §5.5 que D-56 da al administrador de accesos es un permiso propio, `seguridad.asignar_roles_sin_tenerlos`. Lo tiene solo `ADMIN_ACCESOS` y no se marca en un rol propio, así que extenderla a otro rol sigue exigiendo una decisión nueva. Sus límites siguen siendo los de D-55, sobre los roles reservados de D-121 (§5.5, §5.6) |
| D-121 | Un rol es reservado por lo que incluye: lo es si incluye un permiso privilegiado, un `seguridad.*` de una categoría que no es `ver` (D-118). Solo un superadministrador lo otorga y lo retira, y una cuenta que tiene uno vigente es protegida: solo la maneja un superadministrador. `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, que eran reservados por su código, lo siguen siendo por lo que incluyen, y un rol propio con un permiso privilegiado también lo es. "Rol reservado" reemplaza a los tres códigos en D-59 y D-72. El superadministrador otorga cualquier rol; la regla 2 de §5.5 se aplica a los no reservados. Agregar o quitar un permiso privilegiado en un rol con titulares vigentes es solo del superadministrador. Volver reservado un rol se rechaza (`ROLE_RESERVED_HOLDERS_INVALID`) mientras alguno de sus titulares tenga una asignación acotada a una marca, una contraseña provisional o una asignación que no otorgó un superadministrador: un rol reservado no se limita a una marca, va solo a una cuenta con contraseña propia y lo otorga solo él (§5.5, §5.7) |
| D-122 | Dónde actúan los permisos de seguridad. Los once de accesos y roles —`dar_acceso`, `asignar_roles`, `asignar_roles_sin_tenerlos`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas`, `crear_roles`, `editar_roles`, `desactivar_roles` y `marcar_permisos_de_roles`, los cuatro últimos en lugar de `gestionar_roles` desde D-130— solo cuentan desde asignaciones de alcance sistema, y solo se marcan en roles de alcance `SYSTEM`, que por eso son generales; guardarlos o copiarlos en otro rol se rechaza (`ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`). `registrar_colaboradores`, `modificar_colaboradores` —los dos en lugar de `gestionar_colaboradores` desde D-130— y `registrar_salidas_y_regresos` actúan en la empresa o la sede de su asignación: un colaborador sin sede exige actuar en toda la empresa, y un cambio de sede, en la de origen y en la de destino. `ver_cuentas` con alcance sistema lee todas las cuentas, y con una empresa, las de esa empresa, también al exportar. `ver_roles` lee los roles del sistema y los generales con cualquier asignación que lo traiga, y los de una empresa donde actúa. Las reglas 1 y 2 de §5.5 comparan, para un rol general, con lo que quien edita tiene por asignaciones de alcance sistema, y para un rol de la empresa C, con lo que actúa en toda C. Nadie cambia los permisos de un rol que tiene vigente (enmienda D-70). Las negativas conservan su código: `ACCESS_DENIED`, `OWN_ACCOUNT_FORBIDDEN`, `ACTOR_EMPLOYEE_REQUIRED` y `ACCOUNT_MANAGED_BY_SUPERADMIN`. La excepción de D-89 queda donde de verdad está: quien tiene `dar_acceso` con alcance sistema —hoy solo el superadministrador— vincula su propia cuenta mientras no tiene un colaborador activo, y registrar colaboradores no exige al actor un colaborador. Desvincular pasa por la misma guardia que las demás operaciones de cuenta (enmienda D-82) (§5.3, §5.5, §5.8) |
| D-123 | Los roles propios los crea, los edita y los copia quien tiene `seguridad.gestionar_roles` con alcance sistema; hoy, solo el superadministrador. Enmienda D-98, que se lo daba al superadministrador por su rol. Lo demás de D-98 sigue: el código es único dentro de cada empresa y entre los generales, nunca el de un rol del sistema, y un rol de una empresa no se asigna en otra ni es de alcance sistema (§5.5) |
| D-124 | Un rol propio se crea también a partir de otro, como copia exacta de sus permisos: "Crear rol propio a partir de este". La copia no se recorta. Si quien la hace no posee alguno de los permisos, se rechaza nombrando los que faltan (`ROLE_PERMISSIONS_NOT_HELD`); si junta un par, `ROLE_PERMISSIONS_INCOMPATIBLE` (D-119); y si el rol nuevo no es de alcance sistema y la copia trae alguno de los once de D-122, `ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`. Copiar `SUPERADMIN` o `ADMIN_ACCESOS` siempre se rechaza —el primero junta pares; el segundo trae `asignar_roles_sin_tenerlos`, que nadie más posee—, y la pantalla lo dice antes de intentarlo. Así un rol del sistema se varía sin tocarlo (enmienda D-97) (§5.5). Enmendada por D-150: la copia a un rol que no es de alcance sistema también se rechaza si trae un permiso que escribe un catálogo general |
| D-125 | La plataforma de documentos es un módulo propio, `documents` (`ky.millennium.mobility.erp.documents`), dueño de `generated_documents` y de lo que la gobierna: la generación única, el almacenamiento en la base o en Spaces, el vencimiento y las rutas `/documents/{id}` y `/documents/{id}/file`. Deja de ser parte de `shared`, porque tiene su tabla, sus reglas y su vocabulario. Su contrato público es un solo paquete, `documents::acl` (`@NamedInterface("acl")`): la fachada `DocumentService`, que devuelve un tipo del contrato y nunca el agregado; `DocumentRequest`, el pedido; `DocumentSeeOther`, la respuesta `303`; y `DocumentFormat`, `DocumentKind`, `DocumentScopes` y `DocumentContent`. No abre su dominio ni su API. No depende de ningún contexto (`allowedDependencies = {}`), y lo declaran los seis que exportan: `location`, `iam`, `organization`, `vehiclecatalog`, `fleet` y, desde que exporta los proveedores, `procurement` (D-134). Desde el 23/10/2026 el mismo módulo guarda los archivos que no vencen en `kept_files` (§2.7) y el contrato publica también `KeptFileStore`: lo declara `procurement`, que guarda ahí el logo, la firma y el PDF de la orden enviada. En `shared` quedan los escritores genéricos de hojas de cálculo (`SpreadsheetExporter`, `SpreadsheetColumn`), que no saben de documentos, y las respuestas `410` y `503` del manejador de errores (§2.7, §3; mapa de contextos, §3 y §5.1; convenciones de la API, §8) |
| D-126 | `GET /api/v1/permission-matrix` es la cuarta lista corta sin página: los permisos con su módulo y su categoría, los pares incompatibles y los roles que la sesión lee, cada uno con lo que incluye, para la pestaña Matriz de permisos de Seguridad › Roles y permisos. Es además la única tabla del frontend fuera de `mm-data-manager`: módulos por roles, con su propio contenedor fijo, como lo registra DESIGN.md en su revisión 24 (convenciones de la API, §6.2; procesos y pantallas, apartado 3.2) |
| D-127 | La sesión se cierra por 2 horas de inactividad, antes 30 minutos (enmienda D-33); el tope de 8 horas desde que se abrió se mantiene. Una jornada con pausas largas en el patio o el taller no pierde el trabajo, y el tope de 8 horas sigue protegiendo la computadora compartida (§5.12; decisión del responsable, 20/10/2026) |
| D-128 | La versión de los datos de cada tabla que exporta un documento es la suma de sus 64 contadores en `data_versions`, una tabla técnica de `shared`. Los suben dos disparadores por sentencia en cada tabla versionada, uno después de insertar y otro después de actualizar, con su tabla de transición y solo si la sentencia cambió alguna fila; la casilla se sortea una vez por transacción y por tabla, y las sentencias siguientes de esa transacción la reutilizan. El aumento va en la transacción del cambio, así que una transacción que confirma tarde cambia la versión igual. La versión es global por tabla, no del alcance de quien pregunta: un cambio en una empresa cambia la que leen todas; es más conservadora, el documento se reutiliza un poco menos, y es correcta. Reemplaza la lectura de las propias tablas (cuántas filas, `xmin` y `updated_at`, bajo Row Level Security), que costaba de 150 a 200 ms para el Inventario con 100 000 movimientos y 100 000 lecturas, sobre el umbral de 50 ms por exportación; ahora cuesta alrededor de 1,6 ms, con cualquier volumen. Queda un riesgo: dos transacciones que escriben dos tablas en orden opuesto se interbloquean en los contadores cuando cada una sorteó la casilla que tiene la otra, alrededor de una vez en 4 096. Lo mitiga `DeadlockRetry`, que reintenta una sola vez, en una transacción nueva, el servicio de comandos o el proceso programado que perdió un interbloqueo (`40P01`); por eso un servicio de comandos debe poder repetirse entero (§2.7; convenciones de la API, §8.3; decisión del responsable, 20/10/2026) |
| D-129 | La bitácora de cambios (`change_logs`, §13.1) vive en `shared`, que la escribe y también la sirve: `GET /api/v1/change-logs`, con `bitacora.ver` y su alcance, y la pantalla Seguridad › Bitácora de cambios (`/change-log`). Resuelve sus referencias a otros contextos por dos puertos de `shared` que implementa cada dueño, como `ProtectedAccounts` (D-121): `AuditActorLookupPort`, que implementa `iam`, devuelve de cada cuenta de `changed_by` su identificador público y su nombre —el de su colaborador, que `iam` lee por la fachada de `organization` (`organization::acl`), nunca por SQL sobre `employees`; `SYSTEM` para la cuenta de los procesos automáticos; ninguno para una cuenta sin colaborador—; y `AuditCompanyLookupPort`, que implementa `organization`, traduce la empresa de cada fila entre su identificador público y el interno. Los dos leen con el alcance de quien consulta. `shared` no lee tablas de otro contexto (regla 1.2) ni depende de su código. Enmienda la decisión pendiente 6 del mapa de contextos solo para la bitácora: deja la opción A (la capa de lectura) para `import_batches`, y la B sigue para `fleet` y `procurement`. La decisión pendiente 3 queda para `/imports` y `/reports` (§13.1; mapa de contextos §11) |
| D-131 | El superadministrador cambia los permisos de un rol del sistema que él no tiene vigente, en la misma matriz y con las mismas reglas que un rol propio: solo marca lo que tiene, no junta un par incompatible y no se agranda por un rol que ya tiene (D-70). El nombre y el estado de ese rol siguen fijos (D-97). Quien no es superadministrador los ve en solo lectura (§5.5, §5.6) |
| D-132 | Plantillas y órdenes de compra: la observación (`observation`) es texto de la plantilla, distinta de `special_instructions`; `has_discount_column` se decide en cada orden al crear o editar el borrador; una línea activa lleva `discount_rate` (0 a 1) solo cuando la columna está encendida y el total de la línea no se guarda; `supplier_contact_id` es opcional y de ese proveedor; una línea retirada del borrador queda `is_active = false` y no se borra; `purchase_order_items.company_id` se copia de la orden (como D-111); en el PDF, un vehículo imprime marca y nombre del modelo y un concepto imprime `concept`; Envío y Otro se imprimen en blanco y no se guardan; el estado guardado es `BORRADOR`, `ENVIADA`, `CUMPLIDA` o `ANULADA` (§9.2; enmienda D-35) |
| D-133 | Cierre de la auditoría del 25/10/2026 sobre proveedores, plantillas y fondos. El fondo lleva `created_at` y `updated_at` (§12.1). La cesión lleva su `company_id`, copiado de la unidad, con llaves compuestas hacia la unidad y el fondo para que la base no admita un fondo de otra empresa, y su Row Level Security lo usa directo, como D-111 (§12.2). El catálogo de fondos y las cesiones a mano se construyen antes de las cargas del bloque 9; la llave de `import_batch_id` y el archivo de garantía mobiliaria llegan con ellas. Hoy solo se registran proveedores locales, con el domicilio en el país de la empresa; el del exterior queda como candidato (§9.1, §9.8). El proveedor y su contacto no se guardan en la plantilla, como ya decía §9.2: se retiran las columnas que agregó V73. Los permisos de fondos nacen partidos como su pantalla (D-130): `fondos.crear_fondos`, `fondos.editar_fondos` y `fondos.desactivar_fondos` para el superadministrador y el administrador de la empresa; `fondos.gestionar_fondos` no se siembra. `fund_type` es una lista cerrada en minúsculas, como toda lista cerrada del sistema |
| D-134 | El resto del cierre de la auditoría del 25/10/2026, que D-133 no recoge. **Emisor:** se copia de la empresa al crear la plantilla, con su domicilio como copia fiel —la misma división, aunque esté inactiva—, y no se edita; el alta y la edición ignoran el emisor que llegue en el pedido y la respuesta devuelve la copia (§9.2; enmienda D-35 y D-75). **Atención:** se copia del colaborador; `attention_email` queda vacío y es candidato (§9.8). **PDF enviado:** una sola copia fiel con el teléfono de Atención para todo el que lee la orden, excepción a D-105 (§9.2). **Archivos de la plantilla:** se guardan bajo una llave de su contenido (SHA-256), escrita antes de confirmar y apuntada al confirmar; `sent.pdf` es la única llave fija (§2.7). Una imagen admite a lo sumo 4096 píxeles por lado y 16 megapíxeles, leídos de su encabezado antes de decodificarla (§9.2). **Direcciones:** se acotan por el país de la dirección, con rama `SYSTEM`, como excepción a la regla de `company_id`; V78 detiene el despliegue si una dirección de una empresa, una sede, una ubicación, un proveedor o un emisor está fuera del país de su empresa, y `addresses.company_id` queda como candidato (§2.5, §4.6, §4.8; enmienda D-80). El domicilio de un proveedor es una fila propia (`UNIQUE`, V77). **Quién escribe:** un proveedor, un fondo, un tipo de orden y una plantilla general los escribe solo una asignación de toda la empresa y sin marca; ceder una unidad y cerrar la cesión, una de toda la empresa o de la marca de la unidad, nunca la de la sede, como excepción a D-101 (§2.5, §12.2). **Escribir exige leer:** las pantallas y lecturas de compras piden `compras.ver`, y las de fondos `fondos.ver`; escribir sobre lo que no se lee responde como si no existiera, y la respuesta de una escritura no vuelve a filtrar (§5.6). **Permisos retirados:** no cuentan en los pares de D-119; sí en la reserva del rol y en el alcance sistema (§5.6). **Código del fondo:** se corrige mientras ninguna cesión lo usa (§12.1). **Documento del proveedor:** se normaliza como todo identificador externo, también los ya guardados (V76, §9.1). **Exportaciones:** proveedores y fondos se exportan con el alcance de su lectura; la lista de órdenes y la de plantillas quedan pendientes para el paso de reportes (D-116). **Errores:** un código `_OUTDATED` responde `422`, como todo código propio por su sufijo (convenciones de la API, §7.2). **Permisos de compras:** plantillas, tipos y órdenes piden un permiso por acción; V79 siembra los ocho finos y los marca sin quitarle a ningún rol lo que hacía, también en los roles propios, y V80 desmarca los gruesos en todos los roles, con su propia ventana de remarcado —solo para el rol propio que nunca tuvo ningún permiso fino del grupo, ni vigente ni revocado: un desmarcado deliberado deja filas revocadas y se respeta— y una revisión al final (§5.6, §9.2; enmienda D-130) |
| D-135 | La orden de compra se imprime con el formato con que la empresa la emite, una plantilla HTML que solo varía en los datos y en la columna de descuento, y en las páginas A4 que hagan falta, sin dejar líneas fuera de la hoja ni de sus totales. Una línea de vehículo guarda el año del modelo (`purchase_order_items.model_year`), entre 2017 y el año siguiente al actual como el de la unidad, y se imprime `{marca} {modelo}, AÑO {año}`, sin la palabra VEHÍCULO delante (decisión del usuario del 27/10/2026); la API devuelve ese mismo texto en cada línea, y la pantalla la arma con marca, modelo y año, sin escribirla a mano. La condición de pago se imprime bajo la fecha y el número. El total de cada línea se redondea a los decimales de la moneda, mitad hacia arriba, y el subtotal suma las líneas así redondeadas (§9.2; enmienda D-132) |
| D-136 | Marcas y modelos es la lista de todos los modelos del catálogo, cada uno con el nombre de su marca, con búsqueda, filtros por marca, tipo de vehículo y estado, y su exportación (`GET /vehicle-models`). Las marcas se registran, se corrigen, se activan y se desactivan en su lugar, en el diálogo «Gestionar marcas», que busca y pagina en el servidor; no tienen ficha propia, y un enlace a la antigua ficha abre la lista filtrada por esa marca. Un modelo nuevo elige su marca, activa, en su diálogo y se sigue creando bajo ella (`POST /vehicle-brands/{id}/vehicle-models`). Las marcas ya no se exportan por separado: la exportación de los modelos lleva su marca (enmienda D-116 para este catálogo). Reemplaza a D-112 (procesos y pantallas, apartado 3.2; DESIGN.md, revisión 30) |
| D-137 | Los contactos de un proveedor se mantienen en su ficha, uno por uno y por su propia ruta (`/suppliers/{id}/contacts`, `/supplier-contacts/{id}`), en una tabla con búsqueda, filtro de estado y páginas: se agregan, se corrigen, se desactivan y se reactivan con `compras.editar_proveedores`, y no se borran ni se exportan. Corregir uno exige el `updatedAt` que se leyó (`422 SUPPLIER_CONTACT_OUTDATED`). El diálogo del proveedor ya no los lleva: ni el alta (`POST /suppliers`) ni la corrección (`PUT /suppliers/{id}`) los leen, y crear un proveedor abre su ficha. `GET /suppliers/{id}` sigue trayendo los activos, que usa la orden (§9.1.1, §5.6) |
| D-138 | La orden de compra nombra su moneda por su código ISO (`"currency": "PEN"`), como la empresa y el país (D-13). Registra su último cambio (`updated_at`, `updated_by`, V85), y guardar un borrador exige el `updatedAt` que se leyó: uno anterior responde `422 PURCHASEORDER_OUTDATED` sin guardar nada. Los decimales de una moneda quedan fijos desde su primera orden, de cualquier estado y de cualquier empresa: `location` lo pregunta por el puerto `CurrencyUsage`, que implementan los contextos con montos, sin depender de ellos (§4.1, §9.2) |
| D-139 | Los tipos de documento del vehículo (§11.2) se diseñan así antes del paso 9. La tabla se llama `vehicle_document_types`, para no confundirla con los documentos tributarios ni con los tipos de archivo de la plantilla. Es por país, con el código único dentro de él, como los documentos tributarios (D-44), porque `TIVE`, `SOAT` y `REV_TECNICA` son de Perú. Lleva `created_at` y `updated_at`, como los demás catálogos (D-42, D-73). `TIVE` queda fijo desde la semilla, porque lo leen las reglas de la placa, del impuesto vehicular y de la habilitación (extiende D-74). El estado de habilitación nombra el documento bloqueante que falta, así que marcar otro tipo como bloqueante no exige programar (§11.9). La factura de compra deja de ser un tipo: la unidad llega a la suya por su línea de factura (§9.3, §9.4), y guardarla dos veces es lo que §11.6 evita. Lo mantienen `catalogos.crear_tipos_documento`, `editar_tipos_documento` y `desactivar_tipos_documento`, no los permisos de estados (enmienda D-100). Su ruta en la API es `/vehicle-document-types`. Quedan abiertas la exigencia del certificado de gas a las unidades convertidas y a quién llega el aviso de renovación (§14) |
| D-140 | Se cierran las dos preguntas abiertas de compras. El número de la orden mantiene su forma, serie, año y correlativo (D-36), y no toma la de la muestra de la empresa (`2026-03-0001`). La Atención no lleva correo mientras los colaboradores no tengan correo de trabajo: cuando lo tengan, saldrá del colaborador, como el teléfono (D-134), y no se escribirá a mano en la plantilla (§9.2, §9.8). La hoja de la plantilla toma las bandas de la hoja de la orden (DESIGN.md, revisión 32) |
| D-141 | Las pantallas de Catálogos siguen «Escribir exige leer» (D-134), como compras y fondos: se abren solo con `catalogos.ver`, y un permiso de escritura no las abre. Hasta el 30/10/2026, un rol con solo una escritura veía la opción del menú y la lista le respondía `403` (§5.6) |
| D-142 | La búsqueda `q` de las listas del servidor no distingue tildes ni mayúsculas: los dos lados de la comparación pasan por `fold_search_text`, una función inmutable de la base que, con `translate()`, quita a las vocales sus acentos (agudo, grave, circunflejo, diéresis, tilde y anillo), a la ñ su tilde y a la ç su cedilla, y pasa a minúsculas (V86). No usa la extensión `unaccent`, que pide permisos sobre la base y cuya función no es inmutable, ni un índice: una búsqueda que contiene un texto no lo aprovecharía, y si una lista crece se agrega `pg_trgm` sobre el texto plegado. Los filtros exactos, como el puesto de un colaborador o el campo de la bitácora, siguen comparando sin plegar (convenciones de la API, §6.2) |
| D-143 | Avisarle algo a una persona —por correo, SMS o push— es un módulo de la plataforma, `notifications` (`ky.millennium.mobility.erp.notifications`), como `documents` (D-125): será dueño de los pedidos de notificación con su estado y sus intentos, de las plantillas de cada tipo y de los dispositivos registrados para push. Cada proveedor —SendGrid, Twilio, Firebase Cloud Messaging— es un adaptador de un puerto por canal, no un contexto; una integración que trae datos de un dominio va con el dueño del dato (el tipo de cambio en `location`, los archivos en `fleet`). Los contextos piden por `notifications::acl` y le pasan el destino que resolvieron, así que `notifications` solo depende de `documents::acl`, para los adjuntos. El pedido se guarda en la misma transacción del hecho que lo origina y lo envía después un proceso con reintentos, sin enviar dos veces el mismo. Desde el 30/10/2026 existe solo su arquitectura; sus tablas entran a este diccionario con su primer caso: el enlace para restablecer la contraseña (D-63) o el aviso de renovación del paso 9 (§14). Precisada por D-242: el primer caso es el enlace (D-243). Al construirlo, el 09/11/2026, «sin enviar dos veces el mismo» quedó como al menos una vez: dos corridas nunca toman la misma notificación, pero una instancia que se detiene entre la aceptación del proveedor y su registro deja que el correo salga de nuevo (§2.7) |
| D-144 | El inicio de sesión tiene un freno que cuenta los intentos fallidos por cuenta y por dispositivo, nunca por IP, en `sign_in_throttles` (§5.4). Quien entró con éxito desde un navegador recibe la cookie firmada `MM_DEVICE`, y desde entonces sus intentos en ese navegador tienen su propio contador; todo intento sin esa cookie, o con una firma que no coincide, comparte un único contador por cuenta, venga de la IP que venga, también el de un correo que no existe. Cada contador deja cinco fallos sin espera; después, una espera de 1 minuto que se duplica hasta 15, y un fallo de hace más de 24 horas deja de contar. Mientras dura la espera responde `429 SIGN_IN_THROTTLED`, con `Retry-After` y `context.retryAfterSeconds`, sin revisar la contraseña. El intento se cuenta antes de comparar, con la fila del contador bloqueada, y un acierto vuelve a cero solo el contador que se usó, nunca el de la cuenta si se entró desde un dispositivo de confianza. Nunca hay bloqueo definitivo: nadie cierra una cuenta ajena, tampoco la del único superadministrador, que no puede restablecer su propia contraseña (D-70). Una cuenta que suma más de 100 fallos en 24 horas deja un aviso en el registro del servidor. Las llaves de la tabla son el HMAC-SHA256 del correo, o del correo con el identificador del dispositivo, con la clave `IAM_SIGN_IN_KEY`, que firma también la cookie; la cookie lleva un identificador al azar y su firma, nunca el correo, y rotar la clave vuelve todos los navegadores a «sin historia». Un proceso purga los contadores sin actividad en 24 horas y los dispositivos de confianza sin uso en 180 días, con `DELETE` solo sobre esa tabla de control: es la excepción con nombre a la regla de no borrar, que es de los datos del negocio, como las sesiones (§2.5, §5.12). La comparación de la contraseña corre fuera de toda transacción, con a lo sumo dos a la vez por instancia; sin lugar en 3 segundos responde `503 SIGN_IN_BUSY`, y el tope es configurable. El cambio de contraseña pasa por el mismo freno antes de comparar la actual, y el registro público tendrá que pasar por él antes de encenderse (D-83) (§5.2, §5.4, §5.12; convenciones de la API, §7.2 y §9.1; decisión del responsable, 31/10/2026) |
| D-145 | La contraseña tiene de 8 caracteres a 72 bytes en UTF-8, sin reglas de composición. 72 bytes es lo que procesa BCrypt, que corta en bytes y no en caracteres: una letra con tilde o la ñ ocupa dos, y otros signos hasta cuatro. Las pantallas lo dicen en lo que la persona entiende: «De 8 a 72 caracteres; las letras con tilde y la ñ cuentan doble». Una contraseña más larga responde `400 VALIDATION_ERROR` en su campo, sin el texto del cifrado, y la provisional del primer superadministrador (`IAM_BOOTSTRAP_ADMIN_PASSWORD`) cumple la misma regla, o la aplicación no arranca. No hay claves que migrar: el cifrado nunca guardó una de más de 72 bytes. Rechazar las contraseñas ya filtradas, como recomienda NIST, queda como paso siguiente, fuera de esta decisión. Enmienda §5.4, que la limitaba a 72 caracteres (§5.2, §5.4; decisión del responsable, 31/10/2026) |
| D-146 | Una moneda tiene de 0 a 2 decimales, la escala con que se guardan todos los montos, `NUMERIC(14,2)`: así ningún monto en una moneda se redondea en silencio al guardarse. Coincide con las monedas sembradas —PEN y USD con 2, COP con 0— y con las de los países previstos (D-52). Los tipos de cambio son otra columna, con seis decimales (§4.8). La migración que cambia la restricción se detiene si alguna moneda tiene más de 2. Una moneda de 3 decimales reabre la decisión, junto con la escala de los montos. Enmienda §4.1, que admitía de 0 a 4 (§4.1; decisión del responsable, 31/10/2026) |
| D-147 | Al backend solo se llega por Cloudflare. La función de Cloudflare Pages que sirve la API desde el mismo origen que la aplicación (§5.12) manda un secreto de origen (`ORIGIN_SECRET`), y en producción el backend responde `404`, sin cuerpo que explique el motivo, a todo pedido que no lo trae, salvo el chequeo de salud (`/actuator/health`) de App Platform. Acepta dos secretos a la vez, el vigente y el anterior (`ORIGIN_SECRET_PREVIOUS`), para rotarlo sin cortar el servicio, y los compara en tiempo constante. La función borra las cabeceras de origen y de IP que lleguen del navegador antes de poner las suyas, y manda la IP real del cliente (`CF-Connecting-IP`), que el backend usa solo para su registro, nunca para decidir. Fuera de producción el filtro está apagado. La documentación de la API (Swagger y `/v3/api-docs`) se apaga en producción. Una regla de limitación de Cloudflare sobre `POST /api/v1/authentication/sign-in`, holgada —30 pedidos por minuto por IP con 10 minutos de bloqueo, o lo más cercano que permita el plan de la zona—, frena las ráfagas grandes: la fuerza bruta la frena la aplicación (D-144), y una sede entra a la misma hora detrás de una sola IP. Hace falta porque App Platform no permite filtrar la entrada por IP, y filtrar las IP de Cloudflare en la aplicación dependería de cabeceras que quien llama directo escribe a su gusto; sin el secreto, las protecciones de Cloudflare se saltan llamando directo a la dirección de DigitalOcean (§5.12; convenciones de la API, §9.1; decisión del responsable, 31/10/2026) |
| D-148 | Los tipos de cambio son por país. `exchange_rates` lleva `country_id`, el país cuya autoridad publica la tasa, y su llave única pasa a ser el país, las dos monedas, el día y el tipo: un documento usa la tasa del país de su empresa, la más reciente de su fecha o anterior, y la copia. `rate_date` es el día cuyos documentos usan la tasa, no el día en que cerró el mercado, y qué tasa le corresponde lo sabe quien la registra, con la práctica que confirma Contabilidad. `source` nombra la autoridad que la publicó —`SBS`, `BCRP`, `BANREP` o `BANXICO`— y deja de admitir `manual`, porque quién la registró ya lo dice `created_by`. La carga es manual y cuando hace falta, sin obligación diaria, y una tasa que se aparta mucho de la anterior del mismo país, par y tipo pide confirmación. La carga automática de D-46 espera una fuente con acceso permitido para un sistema: la API del BCRP respondió el 01/11/2026 a una consulta automatizada con una página contra robots, y saltarla no es una opción. Las tasas se ven y se registran en Catálogos › Tipos de cambio, filtradas por país, y cada una muestra su última corrección (§4.8; decisión del responsable, 01/11/2026) |
| D-149 | Registrar y corregir una tasa son dos permisos, como los de D-130: `catalogos.registrar_tipo_cambio`, para la tasa de un día que no la tiene, y `catalogos.corregir_tipo_cambio`, para una ya registrada. Corregir es un cambio delicado: exige motivo y queda en la bitácora, porque una tasa que ya se usó movió montos. La semilla da los dos al superadministrador, y la persona de Contabilidad que carga las tasas los recibe por un rol propio general, de alcance sistema, el camino que D-103 dejaba abierto. Una fila de la bitácora sin empresa la sigue viendo solo quien tiene `bitacora.ver` sobre todas las empresas (§13.1); quién corrigió cada tasa por última vez y cuándo lo muestra su pantalla, con `updated_by` y `updated_at` (§4.8; decisión del responsable, 01/11/2026) |
| D-150 | Los permisos del módulo `catalogos` que no son de la categoría `ver` —crear, editar y desactivar monedas, países, tipos de vehículo, marcas y modelos, colores, equipamiento y estados, los de los tipos de documento del vehículo cuando lleguen, y registrar y corregir el tipo de cambio— solo se marcan en roles de alcance `SYSTEM`, como los once de accesos y roles (D-122): marcarlos o copiarlos en un rol de empresa o de sede se rechaza (`ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`). Un catálogo general sirve a todas las empresas, y un rol de una empresa que lo escribiera lo cambiaría para todas. Hasta esta decisión lo sostenía solo la semilla: el código rechazaba fuera de un rol de alcance sistema únicamente los once de accesos y roles (§5.5, §5.13; decisión del responsable, 01/11/2026) |
| D-151 | El documento que convierte montos —la factura de compra, la nota de crédito y el pago— propone su tasa: la del país de su empresa, del tipo que ese país usa para sus compras, la más reciente de su fecha o anterior, y la muestra con el día del que viene; no se elige de una lista, porque la tasa la fija la fecha del documento. Si en un día hábil la tasa más reciente es anterior a la fecha del documento, el formulario lo avisa y ofrece registrarla ahí mismo, con `catalogos.registrar_tipo_cambio` y las reglas de §4.8, incluida la confirmación de un salto grande; si otra persona la registró a la vez, toma la que ya existe. El aviso no impide seguir con la anterior: un feriado tampoco tiene tasa, y el sistema no guarda un calendario de feriados. Quien registra documentos y no tiene el permiso ve el aviso para pedirla; si debe registrarla, recibe además el rol general de D-149, que una cuenta puede tener junto a su rol de empresa. La orden de compra no usa tasa. Se construye con las facturas (§4.8; decisión del responsable, 01/11/2026). Enmendada por D-152: la nota de crédito no propone la tasa de su fecha, copia la de la factura que descuenta |
| D-152 | La factura de compra y el pago usan en Perú la tasa de venta: la que SUNAT muestra para la fecha de emisión del comprobante, o para la del pago, que sale de la SBS y se registra con la fuente `SBS`; un fin de semana o un feriado toma la última publicada (D-151). Es la práctica tributaria peruana para las compras y los egresos, y la de los ERP, que fijan el tipo de tasa por clase de documento. El documento no cambia la tasa a mano: si está mal, se corrige en el catálogo con su motivo (D-149), para que haya una sola fuente y quede rastro; para efectos tributarios vale la tasa oficial, no una pactada con el proveedor. La nota de crédito no propone la tasa de su fecha: copia la de la factura que descuenta, porque reduce el impuesto que esa factura registró (criterio de SUNAT, Oficio 024-2000). `countries.purchase_exchange_rate_type` de Perú es `venta`. Enmienda D-151 en lo de la nota de crédito y cierra lo que §4.8 dejaba a Contabilidad (§4.8, §9.3, §9.5, §10.1; decisión del responsable, 01/11/2026) |
| D-153 | La unidad nace al registrar la factura de compra. Cada línea de vehículo trae su VIN: si la unidad no existe, el registro de la factura le pide a `fleet` darla de alta, en En habilitación y con el motivo "Registro de compra" (§8.3); si ya está en Inventario, porque llegó antes que la factura, la vincula por su VIN y no la duplica. Es lo que hacen los sistemas para concesionarios, y en Perú el comprobante de un bien identificable lleva su número de serie o de motor, así que el VIN se escribe una sola vez y sirve llegue la unidad antes o después que la factura. Lo que el comprobante no trae, como la ubicación, se completa por unidad al registrar la factura. El permiso de registrar la factura basta para el alta; el alta a mano en Inventario sigue exigiendo `stock.crear_vehiculo` (§5.6). `procurement` lo pide por `FleetContextFacade` y nunca escribe en `vehicles`. Cierra la decisión 5 del mapa de contextos y la pregunta 1 de procesos y pantallas (§9.4; decisión del responsable, 01/11/2026). Enmendada por D-160: la factura no pide ubicación; la unidad la recibe con su ingreso físico (§11.1) |
| D-154 | La factura de compra se carga desde el XML del comprobante electrónico (UBL 2.1, el formato de SUNAT): el sistema lee el proveedor, la serie y el número, la fecha, la moneda, los totales, el IGV y cada línea, y todo se revisa en pantalla antes de guardar; la carga a mano queda para lo que el XML no resuelve. En Perú todas las facturas son electrónicas y con el mismo formato, así que leerlas es exacto y evita digitar ochenta líneas. Dónde pone el VIN cada proveedor —en la descripción o en un campo adicional— varía, así que el lector se construye con XML reales de facturas de vehículos, que se piden al negocio. El archivo queda en `xml_path`. Admite el candidato de §9.8 y cierra la pregunta 3 de procesos y pantallas (§9.3; decisión del responsable, 01/11/2026). Enmendada por D-157: la factura se registra a mano primero y el lector vuelve a candidato |
| D-155 | Las tasas de cambio también fijan los códigos ISO de su país: un país con tasas no los corrige, como uno con escalones, direcciones o tipos de documento tributario. Una tasa es de la autoridad de su país (D-148) y su exportación nombra el país por su código, así que corregirlo cambiaría de qué país dice ser una tasa que ya movió montos. `location` lo decide sin salir de su contexto. Enmienda D-47 (§4.2, §4.8; decisión del responsable, 01/11/2026) |
| D-156 | La fecha de una tasa no pasa de mañana en la zona horaria de su país (`countries.default_timezone`): la tasa de un día se conoce desde la tarde anterior, cuando la SBS publica su cierre, y ningún documento lleva fecha futura. Así un año mal escrito, como 2062, no queda registrado. La corrección no cambia la fecha (§4.8; decisión del responsable, 01/11/2026) |
| D-157 | La factura de compra se registra a mano desde su comprobante, y leer su XML vuelve a ser candidato (§9.8): llega después, con XML reales de cada proveedor, sin cambiar las tablas. `xml_path` guarda el XML cuando se adjunta. Registrar a mano no espera a saber dónde pone el VIN cada proveedor, y es lo que el negocio hace hoy. Enmienda D-154 (§9.3; decisión del responsable, 02/11/2026) |
| D-158 | El ISC se decide en cada factura con `has_isc_column`, como la columna de descuento de la orden (D-132): se marca cuando el comprobante lo muestra, porque a veces el proveedor que importa lo detalla y a veces no. Marcada, cada línea de vehículo exige `isc_amount`, de cero hasta el monto de la línea; sin marcar, queda vacío. Aplica a autos y a motos. Es informativo: el ISC ya está dentro del valor de factura y es costo, así que no cambia ningún valor. Enmienda §9.4, que lo hacía obligatorio y solo de motos (§9.3, §9.4; decisión del responsable, 02/11/2026) |
| D-159 | Cada línea guarda el precio de la unidad con impuestos, como lo dice el comprobante y como lo registra hoy el negocio (guía funcional, columna 37), y el total es la suma exacta de las líneas. La base y el IGV impresos no se guardan: se deducen de la tasa. El costo sin IGV se calcula: en una factura, el valor del vehículo ÷ (1 + `tax_rate`), que es el valor contable de la guía (columna 49, que divide entre 1,18); en una boleta u otro comprobante, que no dan crédito fiscal, el valor completo. Al fondo se le sigue reportando el valor de factura (§9.4), y el valor del vehículo sigue siendo el de factura menos sus notas de crédito (§9.6). Cierra la base del costo, que el plan del 02/11/2026 dejaba al contador (§9.3, §9.6; decisión del responsable, 02/11/2026) |
| D-160 | El número de la factura se guarda normalizado, serie y correlativo sin ceros a la izquierda (`F001-123`); en Perú la serie tiene cuatro caracteres y el correlativo hasta ocho dígitos, y en otro país se acepta sin verificar el formato, como un documento tributario sin regla (D-75). No se repite dentro del mismo proveedor y tipo entre las facturas no anuladas. La fecha no es futura en la zona de la empresa (§2.6), y la factura guarda el día de la tasa que copió (`exchange_rate_date`, D-151). Una factura tiene hasta 100 líneas. Cada línea de vehículo trae VIN, motor si viene, modelo, año, color y marca comercial, y copia el modelo, el año, la marca y la empresa. Una unidad está en una sola línea vigente; si su VIN ya existe, se vincula solo si es de la misma empresa, no está anulada y no tiene otra línea vigente, y su modelo, su año y su marca mandan sobre lo que diga la línea; desde entonces `fleet` no le cambia el modelo ni el año. La factura no pide ubicación: la unidad nace sin ella, Fuera de tienda, y la recibe con su ingreso físico (§11.1). El VIN no lleva I, O ni Q (ISO 3779), como ya valida el código. Enmienda D-9 y, en la ubicación, D-153 (§8.1, §9.3, §9.4; decisión del responsable, 02/11/2026) |
| D-161 | Una factura registrada solo cambia en lo que no es fiscal: su archivo y su orden de compra. Lo fiscal —proveedor, tipo, número, fecha, moneda, tasa, montos y líneas— no se edita: se anula y se registra otra, y la pantalla ofrece «Anular y copiar», que precarga la nueva. Un VIN mal escrito se corrige en la unidad con `stock.corregir_vin` (§8.1), sin tocar la factura. Anular exige `compras.anular_factura` y un motivo —error de registro o anulada por el emisor—, guarda quién y cuándo, es de toda la factura y procede solo si ninguna línea tiene notas de crédito ni pagos aplicados. Anular la factura no anula sus unidades: quedan sin factura vigente, se vinculan por su VIN a la que la reemplaza y, si la compra no existió, se anulan por la vía explícita de D-104, con `stock.anular_vehiculo`, que llega con este paso. Así no hacen falta una tercera vía de anulación ni reactivar unidades, que el VIN único de todo el sistema impediría. Reemplaza «corregir el monto de una factura» como cambio delicado de §13.1 (§9.3, §9.6, §13.1; decisión del responsable, 02/11/2026) |
| D-162 | Una factura se vincula a lo sumo a una orden enviada de su misma empresa y su mismo proveedor, y el vínculo se puede poner o quitar después (D-161). Lo pendiente se cuenta por modelo y año: lo pedido menos las unidades de las facturas vigentes vinculadas. Facturar un modelo o un año que la orden no pide, o más de lo pendiente, da un aviso que se confirma, no un error. El precio no se compara, porque no está dicho si el de la orden incluye IGV. Cumplir la orden sigue siendo manual también con el vínculo, con un indicador de «todo facturado», porque anular una factura la reabriría; y una orden con facturas vigentes no se anula. Precisa D-21 (§9.2, §9.3; decisión del responsable, 02/11/2026) |
| D-163 | Las facturas usan los permisos de compras de §5.6, sin agregar ninguno. Registrar exige actuar sobre la empresa y sobre la marca comercial de cada unidad, como el alta a mano (D-101, D-102), sin sede, porque la unidad todavía no tiene ubicación. Una factura se ve por empresa, y un rol de sede ve sus unidades desde que ingresan a su sede. La lista de facturas se exporta a Excel desde la primera versión. Contabilidad no da de alta unidades a mano, pero las que nacen con las facturas que registra quedan dadas de alta (D-153): se corrige su descripción en §5.6 (§5.6, §9.3; decisión del responsable, 02/11/2026) |
| D-164 | `vehicle_statuses.requires_enablement` marca los estados a los que solo se entra con la habilitación completa, y la semilla lo marca en Disponible, que el flujograma define como «unidades listas para entrega a cliente». Un cambio de estado a mano hacia un estado marcado exige que el estado de habilitación sea *Listo para entrega* (§11.9), llegue por «Habilitación completa» o por «Reacondicionada», y si no lo es se rechaza nombrando lo que falta. El regreso automático de Vendido a las 48 h no la aplica mientras Vendido la exige, porque deshace una entrega que no ocurrió y la unidad no pudo perder un requisito; si se desmarca Vendido, el regreso deja en Vendido la unidad a la que le falta algo (D-170, D-184). Antes solo el nombre del camino sugería la regla, y una unidad sin GPS podía volver a Disponible desde Puesta a punto (§8.2, §11.9; decisión del responsable, 02/11/2026) |
| D-165 | `vehicle_documents`, `vehicle_equipments` y `vehicle_tax_declarations` llevan `company_id`, para su política de RLS (§2.5). El estado de habilitación lee de los datos qué bloquea —los documentos y los equipos con `blocks_availability` que aplican a la unidad, la pantalla solo si su modelo no la trae de fábrica— y nombra lo que falta, como D-139 hizo con los documentos. Tener placas son las placas físicas (`plates_received_date`), que exigen la placa registrada, como la columna PLACAS y la decisión A-03 de la guía funcional. El ingreso físico se registra por unidad o en lote, como las unidades de una factura, y fija la ubicación, que el flujograma pide para Disponible. `current_mileage` no es columna: es la última lectura (§8.5), como ya la deduce el código (§11.1, §11.3, §11.4, §11.7, §11.9; decisión del responsable, 02/11/2026) |
| D-166 | El certificado de gas se pide a las unidades cuyo modelo usa GNV o GLP (`vehicle_models.fuel_type`, §7), y la unidad no lleva un dato propio de conversión. Ninguna fuente del cliente habla de unidades convertidas después de comprarlas, y las de gas de su catálogo lo son de fábrica, como el JS4 1.6 MT COMFORT GNV o los RE AUTORIKSHA TORITO 4T LPG (guía funcional, §12). Si un día se convierten, se agrega el dato en la unidad. Cierra la decisión abierta de §14 sobre el gas (§11.2; decisión del responsable, 02/11/2026) |
| D-167 | Hasta el módulo de notificaciones (D-143), el aviso de renovación es la pantalla Logística › Habilitación: muestra el vencimiento de cada documento y filtra los que vencen en los próximos 30 días. A quién se envía, con cuánta anticipación y por qué medio se decide con ese módulo, y ya no detiene el paso 9. Cierra, para el paso 9, la decisión abierta de §14 sobre el aviso (§11.3, §11.10; decisión del responsable, 02/11/2026) |
| D-168 | La carga inicial del Excel trae, con cada unidad, su factura de compra —proveedor, número, fecha, moneda, tasa y precio, agrupando las filas por proveedor y número— y sus notas de crédito, que el Excel ya tiene (guía funcional, columnas 11, 35 a 40, 46 y 47), además de sus datos de habilitación. Sin factura, una unidad migrada valdría cero y quedaría al alcance de la anulación de §9.6; sin sus hitos, quedaría Fuera de tienda y la puerta de D-164 le impediría volver a Disponible. La carga llega en un paso propio, con las del impuesto vehicular y la garantía mobiliaria, después de la habilitación (§9.3, §11.8; decisión del responsable, 02/11/2026) |
| D-169 | Una factura, una nota de crédito o un pago en la moneda funcional de su empresa lleva tasa 1 y, como día de la tasa, su propia fecha: no consulta el catálogo, que no guarda la tasa de una moneda consigo misma, como manda la guía funcional (columna 47: «si MONEDA=PEN, TC debe ser 1») (§9.3, §9.5, §10.1; decisión del responsable, 02/11/2026) |
| D-170 | `requires_enablement` queda fijo en Disponible: la semilla lo marca y no se desmarca, como los códigos fijos de D-74 y D-139. En los demás estados se marca o desmarca con `catalogos.editar_estados`. Es el control que impide dejar Disponible una unidad sin GPS o sin placas, y no puede apagarse desde un catálogo. Precisa D-164 (§8.2; decisión del responsable, 02/11/2026) |
| D-171 | El estado de habilitación tiene un código estable por fila —`FUERA_DE_TIENDA`, `SIN_DOCUMENTO`, `SIN_EQUIPO`, `SIN_PLACAS`, `LISTO_PARA_ENTREGA` y `ENTREGADO`—, con el que se filtra y se exporta, y un detalle con lo que falta: el nombre del documento o de los equipos. La pantalla muestra «Sin pantalla» o «Sin GPS» desde ese detalle. Precisa D-165 (§11.9; decisión del responsable, 02/11/2026) |
| D-172 | Cambiar el archivo o la orden de una factura vigente exige `compras.registrar_factura`. Solo se vincula a una orden enviada del mismo proveedor y la misma empresa; desvincularla se puede siempre, y una orden ya cumplida sigue cumplida, porque cumplir es manual (D-162). Una factura anulada ya no cambia. Precisa D-161 y D-162 (§9.3; decisión del responsable, 02/11/2026) |
| D-173 | La primera versión de la factura solo registra líneas de vehículo, como hace hoy el negocio. Las de concepto —un flete, un accesorio, un trámite— quedan en la estructura (D-22) y se habilitan si las facturas reales traen esos cargos, antes de construir el formulario. Precisa D-22 (§9.4; decisión del responsable, 02/11/2026) |
| D-174 | El archivo de la factura y cada documento del vehículo es PDF, PNG, JPEG o WebP, de a lo sumo 5 MB, y su tipo se verifica por su contenido, no por su nombre, como los archivos de las plantillas (§9.2). El XML de la factura, si se adjunta, es XML bien formado de a lo sumo 2 MB (§9.3, §11.3; decisión del responsable, 02/11/2026) |
| D-175 | Anular una factura deja su fila en la bitácora con el motivo, como todo cambio delicado (§13.1), e Inventario muestra y filtra las unidades sin factura vigente, para que una anulada sin reemplazo no se pierda. Precisa D-161 (§9.3, §13.1; decisión del responsable, 02/11/2026) |
| D-176 | Los permisos de facturas (`compras.registrar_factura`, `registrar_factura_antigua` y `anular_factura`) se siembran en dos tiempos: con el paso de facturas, solo al superadministrador, que las prueba en desarrollo; con las notas de crédito, al administrador de empresa, a Operaciones y a Contabilidad, como marca la matriz. Así ninguna factura real se registra antes de que existan las notas de crédito, que reflejan los bonos de campaña de cada mes. La matriz final no cambia (§5.6; decisión del responsable, 02/11/2026) |
| D-177 | La carga inicial tiene sus propias excepciones: copia la tasa de cada fila del Excel en vez de proponerla del catálogo, no aplica los 20 días de la factura antigua, agrupa las notas de crédito por su código y su proveedor, y manda al archivo de errores las notas de débito, que el sistema no modela (D-23), y las filas cuyo proveedor o modelo no está en el catálogo. Antes de construirla se perfilan los Excel reales de autos y de motos. Precisa D-168 (§11.8; decisión del responsable, 02/11/2026) |
| D-178 | `purchase_invoices` lleva `updated_at` y `updated_by` (D-42). Cambiar el archivo, el XML o la orden de una factura vigente, y anularla, lleva el `updated_at` que se leyó, y uno viejo se rechaza sin guardar, como el borrador de la orden (D-138). Esos cambios y la anulación exigen el mismo alcance que registrar: la empresa y la marca comercial de cada línea, porque deshacer un registro con menos alcance del que hizo falta para crearlo abriría una vía a quien no registra esa marca. El `PUT` cambia el archivo, el XML y la orden, nada fiscal. Precisa D-161 y D-172 (§9.3; decisión del responsable, 03/11/2026) |
| D-179 | El valor de factura de una línea es mayor que cero: un valor cero significa que la compra se deshizo (§9.6). Una línea que vincula una unidad existente no toma de ella el color ni el motor: la unidad conserva los suyos, el formulario los muestra fijos y avisa sin impedir guardar si lo escrito difiere o si la unidad no tiene motor, para que se corrija en Inventario. Precisa D-160 (§9.4; decisión del responsable, 03/11/2026) |
| D-180 | La exportación de facturas muestra la base y el IGV calculados con la tasa de cada factura —en una boleta u otro comprobante sin crédito fiscal, la base igual al total y el IGV en cero—, rotulados «calculado», porque pueden diferir en céntimos del comprobante impreso. Si Contabilidad necesita el IGV exacto, se guarda el impreso. Precisa D-159 (§9.3; decisión del responsable, 03/11/2026) |
| D-181 | `countries.purchase_exchange_rate_type` es `venta` en Perú y `oficial` por omisión en los demás países y en los que se creen después: la TRM de Colombia y el FIX de México son una tasa única. No se edita por la API del país hasta que haga falta. Precisa D-152 (§4.8; decisión del responsable, 03/11/2026) |
| D-182 | Una unidad con una cesión abierta a un fondo no se anula: primero se cierra la cesión, con su motivo. Así ninguna unidad anulada tiene una cesión abierta, y la guarda de la unidad anulada vale para toda escritura, sin excepciones. Precisa D-104 (§9.6, §12; decisión del responsable, 03/11/2026) |
| D-183 | La puerta de Disponible exige que a la unidad no le falte nada —ninguna de las filas 1 a 4 de §11.9—, esté *Listo para entrega* o *Entregado*. Así una unidad reacondicionada, que ya se entregó alguna vez, vuelve a Disponible por «Reacondicionada», como dice el flujograma. Precisa D-164 (§8.2, §11.9; decisión del responsable, 03/11/2026) |
| D-184 | La puerta se sostiene: en un estado marcado con `requires_enablement` no se retira un requisito sin reemplazo —el GPS, la pantalla que corresponde, las placas físicas o el ingreso— (`VEHICLE_ENABLEMENT_HELD_BY_STATUS`). Un equipo se reemplaza; si no hay repuesto, la unidad sale primero por el camino «Pierde un requisito», de Disponible a En habilitación, que trae la semilla con `stock.habilitar`. La semilla también marca Vendido, que se puede desmarcar, porque la guía exige todos los requisitos para entregar (columna 22). Precisa D-164 y D-170 (§8.2, §8.3; decisión del responsable, 03/11/2026) |
| D-185 | `equipment_types.has_installations` dice si alguna unidad ya tuvo instalado ese equipo; lo marca `fleet` en la primera instalación con `markEquipmentTypeInUse`, como `vehicle_models.has_units` (D-107), y desde entonces su código queda fijo (D-48). Así `vehiclecatalog` no cuenta filas de una tabla de otro contexto (§7.5; decisión del responsable, 03/11/2026) |
| D-186 | Ninguna fecha de la habilitación es futura en la zona de la empresa —el ingreso, los hitos, la emisión de un documento ni una instalación—, como la lectura de kilometraje y la tasa (D-156); una instalación tampoco es anterior al ingreso. La ventana de ±20 días y el «no antes de la factura» de la guía (columnas 19 y 21) no se adoptan: el sistema registra hechos, y `fleet` no lee facturas (§11.1, §11.4; decisión del responsable, 03/11/2026) |
| D-187 | Quien instala un equipo sin `stock.ver_costos` guarda su costo en cero, en la moneda funcional de la empresa; quien ve los costos lo registra o lo corrige después, con `habilitacion.gestionar_equipos` y su fila en la bitácora. Así nadie escribe un costo que no puede ver, y el costo de habilitación no queda en cero para siempre (§11.4; decisión del responsable, 03/11/2026) |
| D-188 | Tres reglas de la habilitación: un documento vigente de un tipo que bloquea cuenta aunque haya vencido, porque el vencimiento se sigue en la pantalla de Habilitación (D-167); una unidad que ingresó no queda sin ubicación, tampoco al editarla en Inventario; y en un país sin documento de identificación vehicular la placa no se registra hasta decidirlo con su primera empresa, como sus documentos tributarios (D-52) (§11.1, §11.9; decisión del responsable, 03/11/2026) |
| D-189 | Los archivos de las facturas y de la habilitación se guardan en `kept_files`, con la empresa de su factura o de su unidad, como los de las plantillas y las órdenes (V84): los bytes van a Spaces, y a la base solo donde no hay bucket (§2.7, §9.3, §11.3; decisión del responsable, 03/11/2026) |
| D-190 | Una nota de crédito reduce lo que se debe de la factura que descuenta: lo que se debe de una factura es su total, menos sus notas vigentes, menos lo aplicado de pagos, y lo aplicado no lo supera al aplicarse. Solo lo aplicado de más —porque la nota llegó después de pagarla, como el bono de campaña— es saldo a favor, y el saldo es del proveedor, no de cada nota. Antes cada nota era saldo por sí misma, y la de una factura todavía impaga se descontaba dos veces: al bajar lo que se pagaba y otra vez como saldo. Es la regla de cuentas por pagar, y la guía la sigue: compara lo pagado con el valor después de las notas (columna 45). Enmienda §10.2 y §10.4 (§9.7, §10.2, §10.4; decisión del responsable, 03/11/2026). Enmendada por D-248: el saldo se mide por vehículo. Lo que cubre un vehículo por encima de su valor después de notas pasa a saldo a favor, aunque la factura todavía deba por otros vehículos, y lo que se debe de la factura es lo que les falta a sus vehículos: total − notas − aplicado + lo que pasó a saldo. Una nota sobre un vehículo que todavía debe sigue bajando lo que se debe y no es saldo (§10.4) |
| D-191 | Una nota que deja en cero el valor de una línea la anula. Solo con el motivo «anulación de unidad», cuando la unidad vuelve al proveedor, anula también la unidad, si no tiene contratos ni una cesión abierta; si los tiene, la nota se registra igual y la unidad queda marcada para revisar. Con otro motivo —«anulación de la factura», que el proveedor emite para re-emitirla pasado el plazo de baja, o un ajuste—, la unidad queda sin factura vigente y la factura que la reemplaza la vincula por su VIN: anularla la dejaría trabada, porque su VIN no se puede volver a dar de alta (D-160). Enmienda la primera vía de D-104 (§9.5, §9.6; decisión del responsable, 03/11/2026) |
| D-192 | Cada línea de una nota copia la tasa de la factura que descuenta (D-152), y la tasa pasa de `credit_notes` a `credit_note_items`: una nota que descuenta varias facturas, como permite SUNAT hasta el 31/12/2026, vale en soles exactamente lo que redujo cada una. Todas sus facturas son del mismo proveedor, la misma empresa y la misma moneda. Desde el 01/01/2027, por la fecha de emisión de la nota, todas sus líneas son de una sola factura (RS 000048-2026/SUNAT, aplazada por la RS 000143-2026); el corte es configurable. En la moneda funcional la tasa es 1 (D-169) (§9.5; decisión del responsable, 03/11/2026) |
| D-193 | La nota de crédito se registra a mano, como la factura (D-157): número normalizado y único por proveedor entre las no anuladas (D-160), fecha no futura ni anterior a las facturas que descuenta, archivo como el de la factura (D-174), y su reparto por unidad, con la ayuda de repartir en partes iguales entre las unidades elegidas y los céntimos sobrantes en la primera, como la guía (columna 40). Una línea no baja de cero ni descuenta una línea o una factura anulada. Registrar y anular exigen `compras.registrar_nota_credito` y el alcance de la empresa y de la marca de cada unidad (D-163, D-178). Anular exige un motivo —error de registro o anulada por el emisor— y el `updated_at` leído, deja su fila en la bitácora, y no procede si, sin ella, el saldo del proveedor quedaría negativo, si anuló una unidad o si una línea que anuló ya está en otra factura vigente (§9.5; decisión del responsable, 03/11/2026) |
| D-194 | Un pago se aplica a facturas u órdenes de su proveedor aunque estén en otra moneda: cada aplicación guarda el monto en la moneda de la factura o la orden y su equivalente en la del pago, con la tasa del pago (D-152; 1 en la moneda funcional, D-169), y la suma de los equivalentes no supera el pago. Una de las dos monedas es la funcional de la empresa, o las dos son iguales. El costo de la unidad no cambia: la diferencia con la tasa de la factura es diferencia de cambio, que se informa aparte y nunca toca el costo (§10.1, §10.2; decisión del responsable, 03/11/2026) |
| D-195 | El saldo a favor es del proveedor y por moneda (D-190). Un pago con saldo lo consume, calculado con el proveedor bloqueado, y solo salda facturas de esa moneda. No apunta a una nota: `supplier_payments.credit_note_id` sale. Así un pago puede usar el saldo de varias notas, como el ejemplo de §10.1, que el modelo anterior no admitía (A24) (§10.1, §10.4; decisión del responsable, 03/11/2026). Enmendada por D-249: el pago sigue sin apuntar a una nota en su fila, pero al registrarlo se guardan sus usos en `supplier_balance_uses`, qué notas consumió y cuánto de cada una, primero «sin nota» y después las notas en el orden en que se registraron; esos usos no cambian, y una nota cuyo saldo se usó no se anula hasta anular ese pago (§10.4) |
| D-196 | Cada aplicación registra, si el comprobante de pago la trae, la retención del IGV (`withheld_amount`, en la moneda de la aplicación) y el número de su comprobante de retención. Lo retenido cuenta como cubierto, porque Millennium lo paga a SUNAT por cuenta del proveedor, pero no como dinero entregado al proveedor. La primera versión no la calcula; si Millennium es agente de retención (3 % sobre operaciones de más de S/ 700, con sus excepciones), el cálculo se agrega sin cambiar tablas. La retención sale de los candidatos de la factura (§9.8): se practica al pagar (§10.2; decisión del responsable, 03/11/2026) |
| D-197 | `suppliers.is_dealer` marca al proveedor del canal de dealers, que sigue vigente (§10.5): ahí el cliente paga su inicial al concesionario. Una aplicación a una factura de ese proveedor detalla sus unidades (§10.3), y cada detalle puede declarar la unidad saldada (`settles_unit`): desde entonces no recibe más pagos, y lo que le falta para su valor es la inicial del cliente, que se valida contra el contrato cuando exista ventas (D-31). En el canal normal no hay inicial: lo que falta es pendiente de pago (§9.1, §10.3, §10.5; decisión del responsable, 03/11/2026). Enmendada por D-248: la inicial se congela al saldar, en la imputación que salda (`customer_initial`), y una nota posterior no la achica: lo que la nota baja pasa a saldo. Una unidad saldada recibe otro pago, sin saldar, solo por lo que le falte además de su inicial (§10.3) |
| D-198 | Las aplicaciones y sus detalles llevan `company_id`. Un pago lleva `request_id`, que impide guardarlo dos veces, y su número de operación no se repite en el proveedor entre los no anulados. Lo aplicado a una orden pasa solo a su factura cuando se vinculan, hasta lo que se debe de ella, y una orden con pagos aplicados no se anula. Anular un pago exige `pagos.anular_pago`, su motivo y el `updated_at` leído; no procede si, sin él, el saldo del proveedor quedaría negativo, y deja su fila en la bitácora (§10.1 a §10.3; decisión del responsable, 03/11/2026) |
| D-199 | Una unidad tiene cinco montos, y solo el primero se guarda: valor de factura (fijo; al fondo y al auditor); sus notas de crédito; valor del vehículo (factura menos notas; el costo); cubierto (lo que le toca de lo aplicado a su factura —dinero, retención y saldo—, por detalle o, sin él, en proporción a su valor); y la diferencia (valor menos cubierto), que es pendiente de pago, o la inicial del cliente en una unidad saldada del canal de dealers. Está pagada cuando lo cubierto alcanza su valor. El dinero entregado al proveedor es lo cubierto menos la retención y lo pagado con saldo. La brecha de días cuenta desde el primer pago que cubre parte de la unidad —por su imputación o por el reparto de lo no imputado de su factura— y, si ninguno la cubre, desde la fecha de la factura: cada unidad toma la fecha del bloque de pago que la incluye (guía, columnas 24 y 43). Precisa §9.6 y §10.5 (decisión del responsable, 03/11/2026). Enmendada por D-248: cubierto es lo imputado, elegido o repartido al registrar, y la parte del fondo queda solo para los pagos registrados antes; se suman la inicial congelada y lo que pasó a saldo, con la identidad valor − cubierto − inicial + pasó a saldo = pendiente; y la diferencia es lo pendiente más la inicial, nunca negativa (§10.4) |
| D-200 | Cada carga de archivos vive en el contexto dueño de lo que actualiza y registra su lote en `import_batches` por la fachada de `shared`: el impuesto vehicular y la garantía mobiliaria, en `fleet`; la carga inicial la orquesta `procurement`, porque las unidades nacen con su factura (D-153, D-168), y da de alta las unidades y su habilitación por la fachada de `fleet`. La pantalla Logística › Cargas de archivos la sirve `shared`, dueño de `import_batches`; el resto de la capa de lectura (decisión 3 del mapa) queda para los reportes. Cierra la decisión 8 del mapa de contextos (§11.8; decisión del responsable, 03/11/2026) |
| D-201 | Una carga es un archivo .xlsx con encabezados exactos; la inicial, los 52 de la guía funcional. El impuesto vehicular y la garantía mobiliaria se cargan fila por fila: lo que no encuentra unidad va al archivo de errores. La carga inicial, por factura: las filas de una misma factura entran o fallan juntas, y un VIN ya migrado se informa sin repetirse, así que volver a subir el archivo corregido es seguro. Cierra el pendiente A-12 de la guía (§11.8; decisión del responsable, 03/11/2026) |
| D-202 | La carga inicial da de alta cada unidad por el camino «Migración inicial», con `stock.cargar_migracion_inicial`: a Activo si se entregó (con la fecha de primera entrega como el momento de su paso a Activo), a Disponible si cumple la habilitación (D-183) y no se entregó, y si no, a En habilitación. Conserva su `legacy_number` (D-18). El Excel de stock no trae los demás estados del ciclo, como Mecánica o Custodia: se corrigen después por sus caminos (§8.3, §11.8; decisión del responsable, 03/11/2026) |
| D-203 | El costo de la unidad en su ficha lo lee `fleet` de compras por el puerto `UnitPurchases` (`GET /vehicles/{id}/purchase-cost`), solo con `stock.ver_costos`: el valor de factura, sus notas, el valor del vehículo y el costo sin IGV en su moneda y en la moneda funcional de la empresa —en Perú, soles— con la tasa de la factura (D-169). No espera a la capa de lectura, que no existe. Cierra la decisión 7 del mapa de contextos (§6.1 del mapa, D-199; decisión del responsable, 03/11/2026) |
| D-204 | Reglas de la nota de crédito: <ul><li>«anulación de unidad» deja en cero cada línea que descuenta, y «anulación de la factura» todas las líneas vigentes de sus facturas; un monto parcial con esos motivos se rechaza;</li><li>la unidad devuelta que no se pudo anular, por contratos o por una cesión abierta, se marca «Devuelta al proveedor» como dato calculado, sin columna, y se resuelve por la vía explícita;</li><li>los motivos se guardan como `DESCUENTO_CAMPANA`, `ANULACION_UNIDAD`, `ANULACION_FACTURA` y `AJUSTE_PRECIO`;</li><li>una nota tiene hasta 100 líneas, y el mes de campaña solo con «descuento de campaña»;</li><li>el número sigue el formato de Perú (D-160);</li><li>se registra aunque su proveedor o su moneda se hayan desactivado después de sus facturas, pero con la empresa activa (D-84);</li><li>su archivo se cambia después con un `PUT`, con su `updatedAt` y el alcance de registrar, como el de la factura (D-172, D-178);</li><li>no se anula si una unidad de sus líneas quedó anulada por cualquier vía.</li></ul> Precisa D-191 y D-193 (§9.5; decisión del responsable, 03/11/2026) |
| D-205 | La excepción de los 20 días de la guía (columna 39) no se adopta para la nota de crédito: con la matriz, todos los que registran notas —SA, AE y CO— tienen también `compras.registrar_factura_antigua`, y una nota de campaña suele llegar semanas después de la entrega (§9.5; decisión del responsable, 03/11/2026) |
| D-206 | En la carga inicial, la TIVE y el SOAT que el Excel marca se guardan sin archivo, rotulados «Migrado, sin archivo», y el GPS sin serial; los archivos se suben después como reemplazo (D-28) y el serial se completa una sola vez. Sin eso ninguna unidad migrada tendría TIVE: no registraría su placa, el impuesto vehicular y la garantía no la encontrarían y no entraría a Disponible. Lo que el Excel no fecha se deduce de la misma fila: el ingreso, de la más temprana entre el GPS, la pantalla, la primera entrega y la declaración de stock, o de la factura; las placas físicas, de la primera entrega o la declaración de stock, o del título; la entrada a Disponible, de la declaración de stock (columna 51), a las 00:00 en la zona de la empresa. La ubicación la elige quien carga. Las llaves entran con una etiqueta provisional —el código de la marca comercial y el `N°`—, y una unidad cuyo propietario ya no es la empresa entra como suya y su traspaso se registra después (D-20). El perfil cuenta cada dato deducido (§11.3, §11.4, §11.8; decisión del responsable, 03/11/2026) |
| D-207 | Las notas de crédito migradas entran con el motivo «descuento de campaña», sin mes de campaña, que el Excel no trae; las que dejarían una línea en cero no se migran: se registran a mano con su motivo. Los pagos migrados son uno por bloque de la columna 42, por transferencia y con ese código como número de operación; la tasa es 1 en la moneda funcional y, en otra, la de venta del catálogo para el día del pago o, si falta, la del Excel (D-177). En el canal de dealers, cada unidad se imputa por su valor menos la diferencia que calcula el Excel (columna 45) y queda saldada (D-197); el perfil señala que esa diferencia viene prorrateada (§10, §11.8; decisión del responsable, 03/11/2026) |
| D-208 | El paquete de cargas se entrega en tres partes: el registro de cargas, el impuesto vehicular y la garantía mobiliaria, después de la habilitación; la carga inicial, después de las notas de crédito, porque registra facturas reales (D-176); y su paso de pagos, después de los pagos. En producción la carga inicial corre una sola vez, con los pagos ya construidos, para que los datos entren completos. La carga es síncrona: un .xlsx de a lo sumo 10 MB, el límite del servidor por archivo, con un tope de filas medido en su línea base; un archivo más grande se parte, porque volver a subirlo no repite nada (D-201) (§11.8; decisión del responsable, 03/11/2026) |
| D-209 | La lista de garantía mobiliaria cierra también las cesiones registradas a mano: si una unidad con placa no figura, su cesión se cierra con la fecha de la lista, como manda la guía (§8.2) y sin borrar la historia (D-30); la de una unidad sin placa no se cierra y se informa, y una placa cedida a otro fondo falla sin tocar nada. El fondo y la fecha de la lista quedan en el informe de la carga (§12.2; decisión del responsable, 03/11/2026) |
| D-210 | El registro del pago: `supplier_payments` lleva `exchange_rate_date`, `updated_at` y `updated_by`; cada aplicación guarda su propia tasa y su día —la de venta del día del pago entre la moneda del pago y la de la factura o la orden, o 1 si son iguales—, porque un pago en soles tiene tasa 1 y no sirve para convertir una factura en dólares; la retención se suma a `amount`, que es dinero o saldo y cuadra con el voucher; y un pago se aplica completo, porque lo sin aplicar no sería deuda ni saldo. Precisa D-194, D-196 y D-198 (§10.1, §10.2; decisión del responsable, 03/11/2026) |
| D-211 | Órdenes e imputaciones: un pago se aplica solo a órdenes enviadas, nunca a una de un proveedor dealer, que no tiene unidades que imputar, y sin tope por su subtotal, que puede no llevar IGV (D-162); una orden con pagos no se cumple ni se anula, y lo aplicado a ella pasa a su factura solo en la misma moneda; desvincular no devuelve nada a la orden. Lo imputado a una unidad no pasa de lo que le falta; lo no imputado se reparte en proporción a lo que le falta a cada unidad; reasignar es anular y copiar el pago (§10.2, §10.3; decisión del responsable, 03/11/2026). Enmendada por D-248: lo no imputado ya no se reparte al leer, sino al registrar, con la misma proporción y los mismos céntimos, y se guarda como imputaciones de una aplicación repartida (`spread`), que no cuentan para los límites de 100 por documento y 500 por pago; también la parte del anticipo de una orden que pasa a su factura. Los pagos registrados antes conservan su reparto al leer (§10.2, §10.3) |
| D-212 | El pago no tiene fecha futura; su número de operación es obligatorio en transferencia y cheque y no lleva con saldo; sus motivos de anulación son `ERROR_REGISTRO` y `DEVUELTO`; un proveedor inactivo sigue recibiendo pagos, porque retirarlo es dejar de comprarle, no dejar de pagarle. Registrar y anular exigen el alcance de la factura: la empresa y la marca de cada línea que paga; con saldo o a una orden general, todas las marcas (D-134, D-163, D-178). Quien ve una factura con `compras.ver` o una unidad con `stock.ver` ve si está pagada, sin montos, para que Operaciones sepa qué unidades retirar; los montos siguen a `pagos.ver` y a `stock.ver_costos` (§10; decisión del responsable, 03/11/2026). Los campos de D-248 y D-249 (10/11/2026) siguen esta regla: <ul><li>con `pagos.ver` en la empresa: `toBalance` de la posición de pago de la factura, de cada vehículo, de la factura, de Por pagar y de lo que se le debe al proveedor; `customerInitial`, `pending` y `current` de cada vehículo de la posición; `spread` y `customerInitial` de las aplicaciones y las imputaciones de un pago; el rastro de la nota (`balance` de `GET /credit-notes/{id}`); el del pago (`balanceSources`); y las notas con saldo del proveedor (`sources` de su saldo y, desde que se construyó, `balanceSources` de lo que se le debe);</li><li>con `stock.ver_costos`, como el resto de «Compra y costo» del vehículo: su `toBalance`, `customerInitial` y `pending`, y lo que pasó a saldo de cada nota (`passedToBalance`); y además con `pagos.ver`, en qué pagos se usó cada nota (`usedIn`);</li><li>sin montos, con `compras.ver`: el estado de pago de cada línea, con el que el formulario de la nota avisa que lo que acredite a un vehículo ya pagado pasará a saldo.</li></ul> |
| D-213 | La diferencia de cambio se informa por cada aplicación pagada con dinero: lo cubierto por la tasa de la aplicación, más los céntimos que su equivalente tomó del redondeo para cuadrar con el pago (§10.2), menos lo cubierto por la tasa de la factura, en la moneda funcional: así refleja lo que de verdad salió; con saldo es cero, porque no se compró moneda. Nunca toca el costo de la unidad (D-194). El contador la valida junto con la retención (§10.5; decisión del responsable, 03/11/2026). Precisada el 11/11/2026 y construida ese día (backend `a91b3611` y `4d292ca0`, merge `aaff72d1`; frontend `29110beb`, `9beabc23` y `0e5dc35c`, merge `b22bbb79`, con la revisión 78 de `DESIGN.md`): cada aplicación de un pago dice la tasa de su factura y su día (`invoiceExchangeRate`, con seis decimales, e `invoiceExchangeRateDate`), contra la que se mide su diferencia, nulas en una orden y en un documento en la moneda de la empresa; la tasa del día es la de la aplicación o, si el pago está en la moneda de la factura, la del pago. La ficha del pago dice de dónde sale la diferencia: con una factura, «Factura a 3.742 · pago a 3.760» debajo de ella; con varias, «De N facturas · el detalle en Aplicaciones», y cada fila de «Aplicaciones», su línea. Sin migración: la tasa ya estaba en la factura. Al integrarla, Por pagar respondió 500 en `dev`: la fila que comparten las lecturas de una factura pedía el día de su tasa, y la consulta propia de Por pagar no lo seleccionaba; lo corrigió el mismo día el backend `80a4fb23` (merge `23d66dcf`), y `PayablesIntegrationTests`, que falla sin él, lo prueba |
| D-214 | La cuenta de origen: `company_bank_accounts` guarda las cuentas con que cada empresa paga, y `supplier_payments.bank_account_id` dice de cuál salió el dinero, porque los pagos no salen siempre de la misma. Transferencia y cheque la llevan; efectivo y saldo a favor, no. La cuenta fija la moneda del pago. La pantalla la propone solo si la empresa tiene una sola activa: recordar la última sería un error que recién aparece al conciliar. No es tesorería: no guarda saldos ni movimientos, y la lista y el Excel de pagos filtran y muestran la cuenta para conciliar. El número de operación sigue único por proveedor (D-198). Las gestionan SA y AE, y solo quien las gestiona lee su número completo (§5.6, §6.2.1, §10.1; decisión del responsable, 05/11/2026). Enmendada por D-246: la pantalla propone la cuenta cuando es la única activa de la empresa en la moneda de los documentos del pago, aunque la empresa tenga otras en otra moneda; nunca propone la última usada |
| D-215 | El voucher: `supplier_payments.file_path` guarda el archivo del pago, PDF o imagen como el de la factura (D-174), en `kept_files`. Es obligatorio en transferencia y cheque, porque su número de operación ya es obligatorio y sale de él; opcional en efectivo; y no lo lleva un pago con saldo a favor. Se cambia sin anular el pago, y el cambio queda en la bitácora. Los pagos anteriores y los migrados quedan sin cuenta, y se les puede adjuntar el voucher después (§2.7, §10.1; decisión del responsable, 05/11/2026) |
| D-216 | La entrega de una unidad sigue vigente hasta que entra a Puesta a punto, y la siguiente la abre de nuevo: *Entregado* pide que su último paso a Activo sea posterior a su último paso a Puesta a punto. El negocio es de leasing y una unidad recuperada por incumplimiento se entrega a otro cliente; leerla *Entregado* para siempre la escondía del filtro *Listo para entrega*. La puerta no cambia, porque acepta ambos (D-183), y la brecha de días sigue midiendo hasta la primera entrega (D-199). Precisa §11.9 (decisión del responsable, 05/11/2026) |
| D-217 | Sacar una unidad de Puesta a punto hacia un estado con `requires_enablement` exige que quien la mueve confirme que la revisó físicamente: el GPS, la pantalla si corresponde, las placas y el documento vehicular. Lo que la puerta lee se registró antes de que la unidad saliera y nada lo borra al recuperarla, de modo que una unidad devuelta sin GPS pasaba. Sin la confirmación el cambio se rechaza (`VEHICLE_PHYSICAL_CHECK_REQUIRED`), y queda en `vehicle_status_history.physically_verified`, con el quién y el cuándo de su fila. La decide el sistema por el estado de origen, sin marca en el catálogo, para que no se pueda apagar, como D-170. La unidad no vuelve a En habilitación: Puesta a punto es su habilitación, como dibuja el flujograma. Precisa D-183 (§8.2, §8.3, §8.4; decisión del responsable, 05/11/2026) |
| D-218 | El estado de código `VENDIDO` se llama «Contrato firmado», el motivo `VENTA` «Firma de contrato» y el permiso `stock.vender` se lee «Firmar contrato»: en un leasing no se vende nada al firmar, y la venta real es la directa o el traspaso al terminar el contrato (D-20). Activo no cambia: es la unidad que está en leasing con el cliente. Los códigos se conservan (FLT-R14), y las decisiones anteriores el nombre con que se escribieron. Sigue marcado con `requires_enablement`: ninguna unidad se coloca incompleta (D-184) (§8.2; decisión del responsable, 05/11/2026). Enmendada por D-251: desde V130 los códigos son `CONTRATO_FIRMADO` y `FIRMA_CONTRATO` |
| D-219 | Robado es un estado propio, de familia terminal y sin ingresos, con los motivos robo, autorrobo y fraude, como en las reglas del registro de clientes. Un robo puede volver y un contrato terminado no: en Inactivo, «Recuperado» se ofrecería también a una unidad traspasada. Sale a Puesta a punto por «Recuperado» o a Inactivo por «Baja definitiva». Inactivo queda final: de ahí no se vuelve. Reemplaza el camino Inactivo → Puesta a punto que dibuja el flujograma (§8.2, §8.3; decisión del responsable, 05/11/2026) |
| D-220 | Pérdida total es un estado propio, de familia terminal y sin ingresos: la reparación supera entre 50 % y 75 % del valor comercial, o hay partes críticas destruidas, según las reglas del registro de clientes. Entra desde Siniestro y sale a Puesta a punto por «Recupera valor» o a Inactivo por «Baja definitiva». Resuelve que el flujograma la lista en dos destinos (§8.2, §8.3; decisión del responsable, 05/11/2026) |
| D-221 | Se cargan los caminos del flujograma (§8.3), cada uno con su permiso (D-100): los de taller con `stock.atender_en_taller`, los de recuperación y salida con `stock.recuperar`. Sin tolerancia los que el flujograma no fija. Choque va a Siniestro, y su camino a Mecánica se cierra sin borrarse. Se agrega Disponible → Inactivo por «Venta directa», que el flujograma no ubica, para vender una unidad de stock sin contrato. No se carga Puesta a punto → Activo, una flecha sin nombre, porque una unidad para otro cliente pasa por Disponible. Reemplaza la pregunta abierta 2 de procesos (§8.3; decisión del responsable, 05/11/2026) |
| D-222 | Inventario y su Excel filtran por «Motivo», el código del motivo del último cambio, para que un motivo que llega a un estado desde dos sea uno solo, y por «Pasó por», un estado en que la unidad estuvo alguna vez. Sin ellos, las situaciones que siguen como motivo no se podían listar, y una unidad robada y recuperada solo se veía en su propio historial (§8.4; decisión del responsable, 05/11/2026) |
| D-223 | El traspaso de una unidad se registra en su ficha, con `stock.recuperar`, que ya es el permiso de salir de la flota, y solo en Inactivo: el día, sin pasar de hoy en la zona de la empresa; el motivo; y a quién, en `vehicles.transfer_owner_name`, porque la guía pide mostrar el propietario vigente (columna 50) y todavía no hay clientes. Cuando existan, ese texto se vinculará al cliente. Lo registrado se corrige con justificación, y cada campo deja su fila en la bitácora; no se deshace, igual que Inactivo es final. Inventario y su Excel dicen la propietaria en vigor. Registrar el traspaso no mueve el estado: el camino a Inactivo va primero, y la diferencia entre los dos momentos es la que dejaría ver una unidad en notaría. Precisa D-20 (§8.1; decisión delegada por el responsable, 06/11/2026). Enmendada por D-239: el traspaso se registra en En traspaso o en Inactivo, y el camino de En traspaso a Inactivo lo exige |
| D-224 | Por pagar es una opción propia de Compras, con `pagos.ver`: las facturas vigentes que la empresa todavía puede pagar, de la más antigua a la más nueva, con la antigüedad en días y cuatro tramos: hasta 30, de 31 a 60, de 61 a 90 y más de 90. Encima, un resumen por moneda y tramo, porque dos monedas no se suman; es una tabla fuera de `mm-data-manager`, leída de una vez, sin filas por página. Se calcula en cada lectura con las mismas reglas de la ficha de la factura y del formulario del pago (D-190, D-197, D-199); las páginas se cortan en el servidor sobre las facturas abiertas, no en la base, porque lo que un dealer todavía puede pagar sale de una regla que no se escribe dos veces. Cada fila abre su factura y, donde `pagos.registrar_pago` actúa sobre su empresa y sus marcas, el pago con esa factura (D-212). No incluye órdenes ni anticipos: por pagar es lo facturado. Excel por `documents`, con el día en sus criterios, porque la antigüedad cambia aunque nada más cambie (§10.5; decisión delegada por el responsable, 06/11/2026). Enmendada el 09/11/2026 por decisión del responsable (DESIGN.md, revisión 68): la pantalla ya no muestra el resumen encima de la lista; ~~`GET /payables/summary` sigue en el backend, sin pantalla que lo lea~~. Enmendada el 10/11/2026: el endpoint `GET /payables/summary` se quitó del backend porque ninguna pantalla lo leía (revisión 68) |
| D-225 | Una unidad del Excel cuyo `PROPIETARIO` no es la empresa terminó su contrato: entra en Activo por «Migración inicial», con su fecha de primera entrega, y en la misma carga pasa a Inactivo por «Contrato completado», dos filas de historial. Conserva su entrega y su brecha de días, el filtro por motivo la encuentra (D-222), y no hace falta un camino de alta a Inactivo. No se escriben los campos del traspaso: `vehicles_transfer_whole_ck` exige fecha, motivo y propietario juntos, y el Excel no trae la fecha, así que inventarla sería un dato falso. El nombre queda en la nota de la fila del historial, la lista de precarga y el informe lo dicen en «Traspasos por registrar», y «Registrar traspaso» (D-223) lo completa. `ENTREGADO=NO` con otro propietario falla, porque no se sabe qué contrato terminó. Es «Contrato completado» por omisión, porque el Excel no distingue una venta directa. Reemplaza a D-206 en lo del propietario. Decisión delegada por el responsable, 06/11/2026. Enmendada por D-239: el camino de Activo a Inactivo por «Contrato completado» se cerró, así que la unidad pasa a En traspaso por ese motivo, con la nota «Propietario en el Excel: … Traspaso por registrar.», y queda ahí hasta que se registra su traspaso y pasa a Inactivo por «Traspaso inscrito». La lista de precarga y el informe la cuentan en En traspaso. Construida el 09/11/2026 con V123 (backend `9f9bd093`; §11.8) |
| D-226 | La carga inicial tiene una lista de precarga: `POST /initial-migration-imports/preview`, con `stock.cargar_migracion_inicial`, corre los mismos pasos con cada factura en una transacción que se revierte, sin lote ni archivo guardado. Dice las filas que entrarían, las unidades por estado, cuántas dicen `ENTREGADO=SI`, para contrastarlas con el registro de clientes, los traspasos por registrar y las filas que fallarían. Revierte en vez de calcular aparte para que la lista no se desvíe de la carga; las notas y los pagos solo se cuentan, porque necesitan sus facturas confirmadas. Decisión delegada por el responsable, 06/11/2026 |
| D-227 | Una unidad recuperada entra en Activo como cualquier entregada, porque el Excel de stock no la distingue, y se corrige por sus caminos: Activo → Custodia → Puesta a punto → Disponible (D-216). Decisión delegada por el responsable, 06/11/2026 |
| D-228 | `PROPIETARIO` es texto libre, así que la empresa se reconoce sin mayúsculas, tildes, espacios ni puntuación, y también por una parte de su nombre: «MILLENNIUM» y «Millennium Mobility S.A.C.» son ella. Vacío o un centinela también. Lo demás es otro dueño, y un nombre mal escrito se ve en la lista de precarga antes de cargar. Decisión delegada por el responsable, 06/11/2026 |
| D-229 | Quedan decididos los puntos que la especificación de cargas dejaba en propuesta: el código reservado `MIGRACION_INICIAL` para los tres caminos de alta, que no cuentan para el camino único ni para el estado inicial y que la API no deja usar (V117); `PUT /vehicle-equipments/{id}/serial`, con V119 para que la aplicación pueda escribir esa columna; y el `N°` único por empresa y marca. Decisión delegada por el responsable, 06/11/2026 |
| D-230 | Ajustes de la carga inicial al construirla: la tasa de un pago migrado usa el tipo de compra del país, el mismo de los pagos a mano, `venta` en Perú; la aplicación va con tasa 1, la que exige la misma moneda; un nombre inactivo del catálogo se lee como inexistente, porque solo se listan los que una unidad puede tomar; y un paso que falla sin que nadie lo esperara cae en el informe (`INITIALMIGRATION_UNEXPECTED`) y la carga sigue, porque lo que ya entró solo lo cuenta su lote. Decisión delegada por el responsable, 06/11/2026 |
| D-231 | La carga inicial toma hasta 400 filas por archivo (`imports.max-rows.migracion-inicial`), medido el 06/11/2026 con facturas de 10 filas: 500 filas en frío: revisar 75 s y cargar 62 s; 1 000 filas en caliente: 83 s y 79 s; volver a subir lo ya cargado: 6 a 7 s, con la máquina ocupada. Es una medición sintética, en un portátil: el ensayo con los archivos reales la confirma (§8.7 de la especificación). Un archivo más grande se parte sin separar una factura, una nota ni un bloque de pago. Decisión delegada por el responsable, 06/11/2026 |
| D-232 | La capa de lectura es el módulo `reporting`, que lee por SQL y nunca escribe. Resuelve la decisión pendiente 3 con un ajuste: las reglas que tienen dueño no se reescriben en SQL. El estado de habilitación, la entrega vigente, la brecha de días y los montos de una unidad se piden a `fleet::acl`, para que un reporte nunca diga otra cosa que la ficha. `reporting` depende de `shared`, `documents::acl` y `fleet::acl`, y cada consulta suya corre contra el esquema en una prueba de integración. `reportes.ver` estaba en la matriz sin sembrarse: lo siembra V120 con sus cuatro marcas (decisión delegada por el responsable, 06/11/2026) |
| D-233 | Un reporte es de una empresa: `companyId` es obligatorio, porque de ella salen la zona y la moneda funcional, y dos monedas no se suman. La marca es opcional, y una asignación acotada a marcas lee solo esas. Las unidades anuladas no cuentan (decisión delegada por el responsable, 06/11/2026) |
| D-234 | El exportable de stock conserva los 52 encabezados y el orden de la plantilla del cliente, en cualquier idioma. ENTREGADO dice si la entrega sigue vigente (D-216) y FECHA DE PRIMERA ENTREGA es la primera; PROPIETARIO es a quién se traspasó (D-223) o la razón social de la empresa, que es quien figura en el registro; DECLARACIÓN DE STOCK es la última entrada a Disponible, para que una unidad recolocada diga cuándo volvió a estar lista; MONTO DE PAGO suma los pagos completos que alcanzan la unidad, y si están en dos monedas los muestra separados. Las columnas de póliza salen vacías (§11.6). Los montos siguen a `stock.ver_costos` por empresa y marca, y ese permiso entra en el alcance del documento: quien no ve montos nunca recibe el archivo de quien sí. El día de la empresa entra en el pedido, porque la brecha cambia sin que nada más cambie (decisión delegada por el responsable, 06/11/2026) |
| D-235 | Los indicadores del flujograma se calculan como dice §10.5. Robado y Pérdida total no son paralizadas: son de familia terminal y no bajan la activación. La ventana de «por vencer» es de 72 horas. «Próximas a resolución» son las unidades en Custodia con tolerancia vencida o por vencer, porque Custodia es el único estado que lleva a «Resolución de contrato». «En riesgo de pérdida» son las unidades en Custodia por Incautación, Comisaría o Depósito: una autoridad las tiene. El costo por día paralizada no se calcula, porque no hay ese dato (decisión delegada por el responsable, 06/11/2026). Precisada por D-239: En traspaso, de familia terminal, tampoco es paralizada ni baja la activación (§10.5) |
| D-236 | La diferencia por inicial se lista solo en unidades saldadas de un concesionario cuya diferencia es `INICIAL_DEL_CLIENTE` (D-197), exige `stock.ver_costos` en la empresa y totaliza por moneda. Validarla contra la inicial del contrato espera al bloque 12. Las dos listas de Reportes cortan sus páginas en memoria, como Por pagar (D-224), porque lo que las ordena sale de reglas que no se escriben dos veces (decisión delegada por el responsable, 06/11/2026) |
| D-237 | V121 versiona `vehicle_keys`, `vehicle_tax_declarations` y `vehicle_fund_assignments` en `data_versions`, porque el exportable las muestra y un cambio en ellas debe cambiar la versión del documento (regla 6.4; decisión delegada por el responsable, 06/11/2026) |
| D-238 | `stock.entregar` se marca también a `ASESOR_VENTAS`, que registra la entrega al cliente en su sede: sobre las unidades que están en la ubicación propia de su sede (D-101). El administrador de empresa lo conserva. Llega con V122, una migración de `iam`: una marca más en la matriz (§5.6). Cierra la pregunta 4 de procesos y pantallas (decisión del responsable, 09/11/2026). Construida el mismo día (backend `b7d846f3`): V122 marca el permiso firmado por `SYSTEM` y se detiene si al final al asesor de ventas o al administrador de empresa les falta la marca |
| D-239 | Estado `EN_TRASPASO`, «En traspaso»: el tiempo en que la notaría inscribe el traspaso de una unidad cuyo contrato terminó o que se vendió directo. Es de familia `terminal`, sin ingresos: no cuenta en el porcentaje de activación ni como paralizada. Su código queda fijo desde la semilla, como el de `INACTIVO`, y no se desactiva. V123, de `fleet`, cierra Activo → Inactivo («Contrato completado», «Venta directa») y Disponible → Inactivo («Venta directa»), y abre Activo → En traspaso por los mismos dos motivos, Disponible → En traspaso por «Venta directa» y En traspaso → Inactivo por «Traspaso inscrito» (`TRASPASO_INSCRITO`), todos con `stock.recuperar`. El traspaso —`transfer_date`, `transfer_reason` y `transfer_owner_name` (D-223)— se registra en En traspaso o en Inactivo, y el camino de En traspaso a Inactivo exige que esté registrado: sin él, `422 VEHICLE_TRANSFER_REQUIRED`. Su tolerancia es lo que tarda la notaría, con «avisar», y llega con D-240. Enmienda D-223, que lo registraba solo en Inactivo (§8.1 a §8.3, §10.5; decisión del responsable, 09/11/2026). Construida el mismo día (backend `9f9bd093`, frontend `9dd5c398`). Al construirla se fijó además: ni En traspaso ni Inactivo se desactivan (`422 VEHICLESTATUS_TRANSFER_RETIREMENT_NOT_ALLOWED`; Inactivo no estaba protegido); V123 siembra los cuatro caminos sin tolerancia; `stock.recuperar` dice «Los caminos de recuperación y de salida de la flota: Custodia, Robado, Pérdida total, En traspaso e Inactivo; y el traspaso de la unidad que salió»; registrar el traspaso en otro estado responde `422 VEHICLE_TRANSFER_NOT_ALLOWED`; y la carga inicial deja En traspaso la unidad con otro propietario, una enmienda de D-225 (§8.2, §11.8) |
| D-240 | La tolerancia de cada camino —`sla_hours` y `on_breach`— se fija por migración, con los valores que entrega el negocio, para que toda base nueva nazca con ellas; después el superadministrador las ajusta en Catálogos › Estados y motivos. Cambiar una tolerancia vale para los movimientos siguientes: el vencimiento se calcula al registrar cada movimiento, y el historial solo se inserta (§8.4). Los valores se piden en la especificación de tolerancias de los caminos del 09/11/2026 (§8.3; decisión del responsable, 09/11/2026). Sin construir: espera la tabla del negocio, y hasta entonces los caminos de V123 no tienen tolerancia |
| D-241 | La historia de los reingresos: `employee_tenure_periods`, de `organization`, de solo inserción (§5.3). Cada regreso guarda, en la misma transacción, el período que se cierra: el ingreso y la salida, que no es anterior al ingreso, con la empresa en la llave compuesta hacia `employees (id, company_id)`, quién registró el regreso y cuándo. Row Level Security por `company_id`, con las políticas de V29 para leer e insertar; la cuenta de la aplicación no la modifica ni la borra. Se versiona en `data_versions`. El período vigente sigue en `employees`. Se lee en `GET /api/v1/employees/{id}/tenure-periods`, con el permiso y el alcance con que hoy se lee la fecha de ingreso (`configuracion.ver_colaboradores`, D-105). Los períodos perdidos antes de la tabla no se recuperan. `termination_reason` sigue candidato (§5.10): la condición que lo admitiría, una pantalla con la historia del colaborador, ya se cumple, y lo decide el responsable. Llega con V124, de `organization`, y cierra la decisión abierta de §14 (§5.13; decisión del responsable, 09/11/2026). Construida el mismo día (backend `dff65113`, `6395606c`, `1136c4f3` y `897b8ef6`; frontend `f07df1aa`, `478047ea` y `f75d6470`). Al construirla se sumó una regla: corregir al colaborador no deja su ingreso antes de la salida del último período que cerró, el mismo día sí, o responde `422 EMPLOYEE_HIRE_DATE_BEFORE_PREVIOUS_TERMINATION` en `hireDate`, para que los períodos no se crucen (§5.3). La fila del colaborador ofrece «Ver historial de ingresos», de solo lectura, sin el motivo de cada salida |
| D-242 | El módulo `notifications` (D-143) tiene su primer caso, el enlace para restablecer la contraseña (D-243), y sus dos tablas (§2.7). `notifications` es la bandeja de salida: canal, tipo, destinatario, plantilla, parámetros en `jsonb` con la lista de los sensibles, estado (`pendiente`, `enviada`, `fallida` o `vencida`), intentos, próximo intento, vencimiento, momento del envío e identificador del proveedor. No lleva `company_id`, la lee solo el alcance `SYSTEM` (NTF-R10) y no se borra; sus parámetros sensibles se vacían al enviar o al vencer (NTF-R11). `notification_attempts`, de solo inserción, guarda cada intento: su notificación, su momento, su resultado, su código de error y la respuesta resumida del proveedor. Canal correo, con dos proveedores: `capturing`, para desarrollo y pruebas, y `sendgrid`, la API v3 por HTTP, sin SDK. Ningún correo se envía dentro de la transacción del hecho: la fila se escribe en ella y un proceso programado la despacha, con reintentos. Llega con V125. Quedan con el responsable el remitente, el SPF y el DKIM de `millennium.ky` y la cuenta de SendGrid; mientras tanto, `develop` usa `capturing` (decisión del responsable, 09/11/2026). Construida el mismo día (backend `d96fd798` y `f82caa6c`), con las columnas que decía, salvo cuatro detalles que §2.7 ya corrige: `status` y `outcome` de 10 caracteres, `expires_at` obligatorio y `provider_response` de hasta 500. Al construirla se fijó: Row Level Security habilitada y forzada en las dos tablas, con políticas solo para el alcance `SYSTEM`; la cuenta de la aplicación con `SELECT` e `INSERT`, y `UPDATE` solo de las columnas de `notifications` que cambian al enviarla; `CHECK` que atan `next_attempt_at` a `pendiente`, `sent_at` a `enviada` y prohíben un parámetro sensible fuera de `pendiente`; los sensibles se vacían también al fallar; sin `idempotency_key` ni número de intento, porque NTF-R04 se cumple reservando el intento con un plazo de dos minutos antes de llamar al proveedor, y la llave queda candidata; el envío es al menos una vez; reintentos de 1 minuto que se duplican hasta 1 hora, con 5 intentos; y fuera de `dev` la aplicación no arranca sin `NOTIFICATIONS_EMAIL_PROVIDER` ni `NOTIFICATIONS_EMAIL_FROM`. Con `capturing` en un entorno desplegado ningún correo sale (§2.7) |
| D-243 | Restablecer la contraseña con un enlace. Un token al azar de 256 bits, del que la base guarda solo el SHA-256, en `password_reset_tokens`: una tabla de control de `iam`, como `sign_in_throttles`, sin `company_id` ni Row Level Security, con `DELETE` para purgar. Vence a los 30 minutos si lo pide la persona y a las 24 horas si lo envía un administrador. Es de un solo uso: usarlo fija la contraseña, borra todos los enlaces de la cuenta y cierra sus sesiones, en la misma transacción; un enlace nuevo reemplaza al anterior; uno inválido, vencido o usado responde siempre `422 PASSWORD_RESET_LINK_INVALID`. La persona lo pide en `POST /api/v1/authentication/password-reset-requests`, que responde siempre `202`, exista o no la cuenta, con el freno del navegador y una espera de 5 minutos por cuenta, y solo emite a una cuenta activa, sin bloqueo vigente, que no sea `SYSTEM`. El administrador lo envía en `POST /api/v1/users/{id}/password-reset`, con el mismo permiso, `seguridad.restablecer_contrasenas`, y las mismas guardas, y ya no muestra una provisional: la contraseña anterior deja de valer y se envía el enlace; responde `202` con el correo enmascarado. El enlace se usa en `POST /api/v1/authentication/password-resets`, con `token` y `newPassword`, que sigue las reglas de longitud (D-145) y es distinta de la actual. El enlace es ~~`<URL pública>/reset-password#<token>`~~ `<URL pública>/iam/reset-password#<token>`: el fragmento no viaja al servidor ni en el `Referer`. Dar acceso a un colaborador sigue mostrando la provisional; enviar ahí un enlace queda como seguimiento posible. Llega con V126. Enmienda D-63 (§5.4, §5.12; decisión del responsable, 09/11/2026). Construida el mismo día (backend `af2e9494` y `267ca405`; frontend `06ba1c4d`). Al construirla se fijó: las pantallas de acceso viven bajo `/iam`, así que el enlace abre `/iam/reset-password`, y la persona lo pide en `/iam/forgot-password`; el freno de los pedidos usa contadores propios en `sign_in_throttles`, de `kind` `recuperacion`, por correo y por correo y navegador de confianza, y un pedido frenado también responde `202`; la espera de 5 minutos se cuenta desde el último enlace de la cuenta; el envío de un administrador reemplaza la contraseña por el BCrypt de un secreto que nadie conserva y deja `must_change_password` encendido hasta que la persona usa el enlace; el `422 PASSWORD_RESET_LINK_INVALID` cubre también el enlace reemplazado y la cuenta que se bloqueó o cambió su contraseña entre los dos pasos de su uso; y un proceso programado borra los vencidos cada hora. Queda abierto si usar el enlace vuelve a cero el contador del inicio de sesión de la cuenta (§14) |
| D-253 | Las palabras de ventas, reservadas antes de diseñar el bloque 12. «Activo» y «Entregado» son de la unidad: Activo es el estado en que la unidad genera ingresos con su cliente (§8.2), y entregada es la unidad que salió físicamente hacia él (§11.9). Un contrato no usa ninguna de las dos: está «Vigente», «Completado» o «Resuelto», y la unidad cuyo contrato terminó y pasó a su cliente está «Traspasada» (D-20, D-223). Así una pantalla de ventas nunca muestra dos «Activo» con significados distintos (mapa de contextos, §1; decisión del responsable, 13/11/2026) |
| D-252 | «Marca» y «Marca comercial» no se confunden en ninguna pantalla ni documento. «Marca», sola, es siempre la de fábrica, la del catálogo de flota (Chevrolet, TVS; §7.2), como la columna MARCA de la plantilla de stock del cliente. Lizzo y Lima Bikes son siempre «Marca comercial» (§6.3), también en los avisos y los rechazos que hablan del alcance de un permiso o de un rol limitado a una marca comercial (D-102), y en las opciones «Todas las marcas comerciales» de sus filtros. «Línea de negocio» deja de usarse como su nombre, también en §9.2; queda solo en las decisiones que ya la usaban y en la frase de la serie de una orden, «la serie distingue la marca y la línea de negocio» (D-140), donde no está claro si nombra la marca comercial o el tipo de orden, para que el responsable lo diga. Quedan dos rótulos cortos dentro de pantallas que ya son de la marca comercial: la pestaña «Marcas» de la ficha de la empresa y «Código de marca» en su diálogo (frontend `feature/contract-code-and-brand-words`, 25 textos; decisión del responsable, 13/11/2026) |
| D-251 | Los códigos del contrato firmado dicen lo que es: el estado `VENDIDO` pasa a `CONTRATO_FIRMADO` y el motivo `VENTA` de su camino, a `FIRMA_CONTRATO` (V130). D-218 cambió los nombres y conservó los códigos, que seguían diciendo una venta; en el leasing la venta es «Venta directa» (`VENTA_DIRECTA`, D-239), y un reporte que filtrara `VENDIDO` para contar ventas contaría contratos. Es la única excepción a que un código no cambia (FLT-R14), decidida mientras ningún entorno guarda unidades reales: cada regla que leía `VENDIDO` lee `CONTRATO_FIRMADO` en el mismo cambio, y la migración se detiene si alguien ya creó esos códigos a mano. `REGRESO_AUTOMATICO` y el permiso `stock.vender`, que se lee «Firmar contrato», conservan su código. Las decisiones anteriores conservan el nombre con que se escribieron (§8.2, §8.3; decisión del responsable, 13/11/2026) |
| D-250 | Una unidad que se firmó o se entregó a un cliente no se anula, tampoco mientras ventas no existe. Hasta ese día la regla de §9.6 —sin contratos— dependía solo de ventas, que no existe, así que ninguna unidad tenía contratos: una unidad en Activo con su cliente se anulaba en cuanto su valor llegaba a cero, con «Anular unidad» o, sin preguntar, con una nota de crédito por «anulación de unidad», y una unidad anulada no admite ninguna escritura (D-182). Desde D-250, `fleet` lee el contrato también del historial de la unidad: lo tiene la que espera en Contrato firmado y la que entró alguna vez a Activo, aunque haya vuelto a Puesta a punto; no lo tiene la que solo pasó por Contrato firmado y volvió sola a Disponible, porque ese regreso deshace un contrato que no se entregó (D-164). Cuando ventas exista, sus contratos se suman a lo que dice el historial. La nota que devuelve una unidad así la deja «Devuelta al proveedor», sin anular, como con una cesión abierta (D-191). Enmienda D-104 (§9.6; FLT-R48; decisión del responsable, 13/11/2026) |
| D-249 | El rastro de cada nota: qué notas usó cada pago con saldo. El responsable pidió el 10/11/2026 que quede registrado en qué se usó la nota de un vehículo ya pagado. Hasta entonces un pago con saldo consumía el saldo del proveedor sin apuntar a una nota (D-195), y nada lo mostraba. Desde esta decisión: <ul><li>**Al registrar un pago con saldo** se guarda qué notas consumió y cuánto de cada una, en `supplier_balance_uses` (§10.4): primero el saldo que no explica ninguna nota, «sin nota», y después las notas en el orden en que se registraron. Se guarda con el proveedor ya bloqueado, en el paso del saldo del registro, y no cambia nunca.</li><li>**Lo que aporta cada nota** se cuenta por factura: cada línea de una nota, en el orden de registro de su nota y de la línea, aporta lo que sube lo que su factura pasa a saldo (`toBalance`, D-248); lo que no explica ninguna nota es «sin nota». Lo disponible de una nota es su aporte menos sus usos en pagos vigentes. Una nota registrada después nunca cambia lo que aportaron las anteriores.</li><li>**Ninguna anulación deja un uso sin respaldo.** Una nota cuyo saldo se usó no se anula hasta anular ese pago: anular una nota se rechaza si a ella, o a otra cuyo aporte cambia sin ella, le quedaría un uso sin respaldo, con `422 CREDITNOTE_BALANCE_USED`, cuyo contexto nombra los pagos; `CREDITNOTE_SUPPLIER_BALANCE_NEGATIVE` se mantiene. Un pago no se anula si deja un uso sin respaldo, con `422 SUPPLIERPAYMENT_BALANCE_USED`; `SUPPLIERPAYMENT_BALANCE_NEGATIVE` se mantiene. Anular el pago con saldo libera sus usos, porque solo cuentan los de pagos vigentes. Los dos chequeos van en las guardas que ya existen, bajo el candado del proveedor.</li><li>**Lo que se ve:** la ficha de la nota, con `pagos.ver`, un panel «Saldo a favor» con lo que pasó a saldo, dónde se usó —fecha, facturas y monto, con enlace al pago— y lo disponible (`GET /credit-notes/{id}`, `balance`); la ficha de un pago con saldo, «De dónde salió el saldo», cada nota con su monto y su enlace (`balanceSources`); el formulario del pago, al elegir «Saldo a favor», de qué notas sale, «USD 1,000.00 de NC F001-45»; el proveedor, las notas con saldo disponible (`sources`, por moneda, en su saldo y en lo que se le debe); y el vehículo, en «Compra y costo», bajo cada nota, lo que pasó a saldo y en qué factura se usó, con el enlace al pago solo para quien lee los pagos (`passedToBalance` y `usedIn`). El formulario de la nota avisa, en cada línea cuyo vehículo ya está pagado, que lo que acredite pasará a saldo a favor del proveedor: lo dice por el estado de pago de la línea, sin montos.</li></ul> V128 crea la tabla, con sus llaves y su Row Level Security por empresa, como sus vecinas, y solo `SELECT` e `INSERT`: es inmutable por sus permisos, sin función nueva. Enmienda D-195: el pago sigue sin apuntar a una nota en su fila, pero sus usos se guardan aparte (§10.1, §10.4; decisión del responsable, 10/11/2026). Decidida el 10/11/2026, antes de construirla; se construye en el backend con V128 (`feature/balance-trace`) y en el frontend con la revisión 76 de `DESIGN.md` (`feature/balance-trace-screens`), integradas a `develop` junto con D-248. Construida el mismo día: backend, V128 y V129, merge `4639799f` (`0bc4bf39`, `e39fd542`, `cb972403`, `92f22198`, `7803e980` y `59de8fee`); frontend, revisión 76 de `DESIGN.md`, merge `96b1a3f6` (`ae87e4d7`, `8be18746`, `0272b66e` y `a0ef2f05`). Al construirla se fijó: `…_BALANCE_USED` se revisa antes que `…_BALANCE_NEGATIVE` y nombra en `context.sources` solo las fuentes que la anulación deja cortas, cada una con lo que pasaría, lo usado y los pagos que la usaron; lo que ninguna fuente puede dar se guarda como «sin nota»; un pago nombra cada fuente una vez, también «sin nota», y la tabla no se versiona, porque ningún documento la muestra; V129, una migración en Java, da sus usos a los pagos con saldo vigentes registrados antes, en el orden en que se hicieron y con la misma regla, y no escribe nada si corre otra vez; la guarda de la anulación de una nota responde su efecto, los usos que dejaría sin respaldo y si el saldo quedaría negativo (`SupplierBalanceAfterAnnulment.effectOfAnnulling`); lo que se le debe al proveedor trae `balanceSources` arriba, filtradas por la moneda pedida, y su saldo, `sources` en cada moneda; `usedIn` de la unidad es por nota, repartido en el orden de los usos para mostrarlo, solo en la lectura de una unidad y con `pagos.ver`; y las fichas preparan sentencias fijas: la de la nota 28, antes 27; la del pago con saldo 32, antes 29; y el saldo 13, antes 12. Se verificó con la suite completa del backend en una base local nueva, con las dos cuentas: 4 021 pruebas, 0 fallos y 2 omitidas, con `ModularityTests` y las pruebas de V128 y V129; y en el frontend con 273 pruebas unitarias de lo que se tocó, 66 e2e y 42 de accesibilidad en local, y las imágenes miradas una por una. Queda sin prueba propia de sus sentencias el registro de un pago con saldo |
| D-248 | El saldo a favor se mide por vehículo. El responsable pidió el 10/11/2026 que la nota de un vehículo ya pagado pueda pagar otra factura del mismo proveedor mientras la factura original sigue debiendo el monto completo de sus otros vehículos, sin cambiar lo facturado y con el porqué de que el vehículo valga menos; y que en el concesionario, con un vehículo saldado por 50 000,00 —42 000,00 de Millennium y 8 000,00 de inicial—, una nota posterior de 10 000,00 pase entera a saldo y la inicial siga en 8 000,00. Hasta entonces el saldo era lo aplicado de más a una factura entera (D-190): una nota sobre un vehículo pagado bajaba lo que se debía de la factura, el exceso no era de nadie, el otro vehículo seguía debiendo todo, la factura podía quedar pagada y salir de Por pagar con un vehículo en pago parcial, y pagar ese vehículo se rechazaba con `SUPPLIERPAYMENT_EXCEEDS_OWED`; y la inicial de un saldado era lo que le faltaba, así que la nota la achicaba: pasaban 2 000,00 y se perdían 8 000,00. Desde esta decisión: <ul><li>**La regla, por vehículo** —cada línea de vehículo de la factura, vigente o anulada porque una nota la dejó en cero—: *valor* es lo facturado menos sus notas vigentes; *cubierto*, sus imputaciones de pagos vigentes, elegidas o repartidas, y, solo en los pagos registrados antes de este cambio, su parte del fondo; *inicial*, la congelada si está saldado, y si no cero; *pendiente*, valor − cubierto − inicial, nunca menos de cero; y *pasó a saldo*, cubierto + inicial − valor, nunca menos de cero. Toda pantalla muestra la identidad **valor − cubierto − inicial + pasó a saldo = pendiente**. La «diferencia» de la API es pendiente + inicial, nunca negativa.</li><li>**Por factura,** con E la suma de lo que pasó a saldo de sus vehículos: lo que se debe (`owed`) es total − notas − aplicado + E, con su signo, como hasta ahora; lo que pasa a saldo (`toBalance`), el mayor entre E y aplicado + notas − total; y lo que se puede pagar (`payable`), lo que se debe menos las iniciales. **El saldo** del proveedor, por moneda, es la suma de `toBalance` de sus facturas menos lo consumido (D-195).</li><li>**El reparto se guarda.** Los pagos sin vehículo —«Todos los vehículos pendientes», «Otro monto», el anticipo que pasa de una orden a su factura y el bloque migrado de un proveedor normal— se reparten al registrarlos, en proporción a lo que le falta a cada vehículo y con la regla de céntimos de hoy (§10.3), y el reparto se guarda como imputaciones de una aplicación marcada repartida (`payment_applications.spread`), que no cuentan para los límites de 100 por documento y 500 por pago. Así, una nota posterior afecta solo a su vehículo.</li><li>**La inicial se congela al saldar.** La imputación que salda guarda la inicial del cliente (`payment_allocations.customer_initial`), lo que le faltaba al vehículo después de ella. Una nota posterior no la achica: lo que la nota baja pasa a saldo. Un vehículo saldado acepta otro pago, sin saldar, solo por lo que le falte además de su inicial —normalmente nada, y entonces se rechaza como hasta ahora, con `SUPPLIERPAYMENT_UNIT_SETTLED`—, lo que ocurre cuando se anula una nota registrada antes de saldarlo.</li><li>**Los ejemplos** (§10.4): V1 y V2 de 10 000,00, V1 pagado con una imputación de 10 000,00, y una nota de 1 000,00 sobre V1: se deben 10 000,00, todo de V2, y pasan 1 000,00 a saldo. Lo mismo pagado con «Otro monto» de 10 000,00, que guarda 5 000,00 en cada uno: se deben 9 000,00 —4 000,00 de V1 y 5 000,00 de V2— y nada pasa a saldo. El concesionario saldado y la nota de 10 000,00: la factura debe los 8 000,00 de la inicial, que Millennium no paga, sigue saldada, y pasan 10 000,00 a saldo. Y si esa nota llegó antes de saldar —Millennium 32 000,00 más 8 000,00 de inicial— y después se anula: al vehículo saldado se le pueden pagar 10 000,00, y nada pasa a saldo.</li><li>**Lo que se sostiene:** un pago de dinero nunca crea saldo, porque lo topan lo que se debe de la factura y lo que le falta a cada vehículo; una nota nueva nunca baja el saldo, ni lo que aportaron las notas anteriores; al desplegar, ningún saldo baja; el filtro previo de Por pagar en la base sigue siendo seguro, porque E nunca supera las notas; y las anulaciones de un pago y de una nota ya bloquean al proveedor.</li><li>**Los pagos registrados antes** conservan su fondo, que la regla sigue leyendo: su parte sin imputar se reparte al leer, como hasta ahora, y para ellos una nota posterior se reparte igual. V127 suma `payment_applications.spread`, falso y sin relleno, y `payment_allocations.customer_initial`, que rellena en las imputaciones que ya saldan con el valor de hoy —si en `dev` hubo una nota después de saldar, la inicial rellenada ya viene reducida—, y recién entonces fija su restricción: la imputación que salda lleva su inicial, mayor o igual que cero, y ninguna otra la lleva. No crea funciones.</li><li>**Lo que se ve:** la posición de pago de la factura, cada vehículo, la factura, Por pagar y lo que se le debe a un proveedor suman `toBalance`; cada vehículo, además, `customerInitial`, `pending` y `current`; las aplicaciones y las imputaciones de un pago, `spread` y `customerInitial`; y la unidad, en su costo de compra, `toBalance`, `customerInitial` y `pending`. La ficha de la factura dice «Pasó a saldo a favor» donde decía «Pagado de más», con Total − Notas − Pagado + Pasó a saldo = Pendiente, y su panel de líneas da por vehículo lo cubierto, la inicial, lo que pasó a saldo y lo pendiente; el formulario del pago suma la cifra «Pasó a saldo» de un documento cuando es mayor que cero; el proveedor dice «Pasó a saldo a favor» donde decía «Pagado de más»; y «Compra y costo» del vehículo, valor − cubierto − inicial + pasó a saldo = pendiente. El reporte «Diferencia por inicial» lista los saldados con inicial mayor que cero, y la columna 45 del exportable de stock lee la inicial congelada. Por pagar exporta lo que pasó a saldo en una columna nueva, y suben las versiones de las exportaciones de facturas y de stock para no servir archivos viejos (D-106).</li></ul> Enmienda D-190, D-197, D-199 y D-211 (§9.5, §9.7, §10.2 a §10.5; decisión del responsable, 10/11/2026). Decidida el 10/11/2026, antes de construirla; se construye en el backend con V127 (`feature/unit-level-balance`) y en el frontend con la revisión 76 de `DESIGN.md` (`feature/balance-trace-screens`), integradas a `develop` junto con D-249. Construida el mismo día: backend, V127, merge `ba5bbb15` (`8ca6f9c0`, `df4a3df2`, `bbe89925`, `c5444af2`, `afc6b76a`, `0f6d7ead` y `678e7b11`); frontend, revisión 76 de `DESIGN.md`, merge `96b1a3f6` (`1796c27c`, `9025e808`, `a6247590` y `051c421b`). Al construirla se fijó: `differenceKind` es `PENDIENTE_DE_PAGO` siempre que algo está pendiente, e `INICIAL_DEL_CLIENTE` solo cuando nada lo está y la inicial es mayor que cero; un vehículo saldado sigue `SALDADA` aunque le falte algo además de su inicial, lo dice su `pending` y se le puede pagar sin saldar; el reparto es en proporción a lo pendiente de cada vehículo vigente y no saldado, y una aplicación que los vehículos no pueden recibir se guarda como el fondo de antes, con `spread` falso; «Unidades imputadas» de la exportación de los pagos cuenta las imputaciones repartidas, y el tope de 500 y la comparación de un envío repetido no las cuentan; `customerInitial` es cero en una unidad no saldada y nulo en una imputación que no salda; `overpaid` del saldo del proveedor conserva su nombre y dice ahora lo que pasó a saldo; la tabla de vehículos de la factura junta VIN y placa, marca y modelo con año, color y marca comercial, los montos, y estado y pago, para que «Pendiente» entre a 1440 px; y el registro de un pago prepara 43 sentencias además de sus inserciones. Se verificó con la suite completa del backend en una base local nueva, con las dos cuentas: 3 996 pruebas, 0 fallos y 2 omitidas, con `ModularityTests` y la prueba de la migración V127 en verde; y en el frontend con 315 pruebas unitarias de lo que se tocó, 211 e2e y 117 de accesibilidad en local, `ng build` sin advertencias, `npm run lint:styles` y las imágenes miradas una por una |
| D-247 | Registrar pago: lo que se paga de cada vehículo. El responsable revisó «Registrar pago» en `dev` el 10/11/2026 y pidió ordenar «Cómo se paga», que la revisión 74 de `DESIGN.md` dejó con un campo por fila, y pagar una parte de un solo vehículo y montos distintos a varios, también con saldo a favor. El backend ya lo admite (D-211): una imputación por vehículo, hasta lo que le falta; cambia la pantalla: <ul><li>**«Solo algunos vehículos»** lleva un monto en cada vehículo marcado: de partida, lo que debe, y se corrige a mano, nunca más de lo que le falta. Lo que se paga de la factura es la suma de esos montos, y el pago envía una imputación por vehículo, sin saldarlo. La pantalla lo explica, «Marque cuáles y cuánto se paga de cada uno», y un monto inválido detiene el registro y recibe el foco. Recupera el monto por vehículo que tenía «Asignar a unidades» (D-245) y que la pantalla de D-246 dejó de tener.</li><li>**Con saldo a favor,** el monto que se propone al marcar un vehículo no pasa de lo que queda del saldo; si lo escrito lo supera, la pantalla pide bajar lo que se paga de los vehículos, como ya lo hacía con el concesionario. «Todos los vehículos pendientes» y «Otro monto» siguen bajando solos hasta el saldo.</li><li>**La copia** de un pago anulado («Anular y copiar») de un proveedor normal con imputaciones elegidas queda en «Solo algunos vehículos» con sus montos, topados en lo que cada vehículo debe hoy y en el saldo, y ya no se convierte en «Otro monto». Una aplicación repartida (D-248) se copia como «Todos los vehículos pendientes» o como «Otro monto», según haya cubierto o no todo lo que se debía.</li><li>**«Otro monto»** queda para el pago que no elige vehículos: el sistema lo reparte al registrarlo y guarda el reparto (D-248). Con el monto vacío, el pie de la tabla dice «Escriba el monto para ver cómo se reparte», en lugar de «Ningún vehículo marcado».</li><li>**«Cómo se paga» en dos filas** por lo que significan, sobre la cuadrícula de 12 columnas de Registrar factura: el método (4) y la cuenta (8); la fecha (3), el número de operación (3) y la constancia (6); debajo, el tipo de cambio (6). El orden de la pantalla es el del documento. Por debajo de 56rem, dos por fila; en el teléfono, uno.</li><li>**La tabla de vehículos:** el dinero a la derecha, el vehículo a la izquierda y el estado al centro, y sus columnas cuentan la del monto también fuera del concesionario.</li></ul> Sin cambio en la API: lo que se envía es el contrato de D-246. Enmienda D-246 en «Solo algunos vehículos», en el saldo, en la copia y en «Cómo se paga» (§10.2, §10.3; decisión del responsable, 10/11/2026). Decidida el 10/11/2026, antes de construirla; se construye en el frontend con la revisión 75 de `DESIGN.md`, en `fix/payment-how-layout`, primero, porque `dev` está roto, y en `feature/payment-vehicle-amounts`. Construida el mismo día: frontend, revisión 75 de `DESIGN.md`, merge `8aa05753` (`31dde635`, `f3d7311f`, `0ababba4`, `b5c1aea1` y `2704c7ca`), con «Cómo se paga» en dos filas y el dinero de los vehículos al final de su fila; y merge `d66ecb84` (`e988ff8e`, `4b672db5`, `b41a97c1` y `9b84ffb6`), con el monto de cada vehículo en «Solo algunos vehículos», propuesto hasta lo que queda del saldo y nunca bajado por él, el primer monto inválido con el foco y la copia de un pago parcial en «Solo algunos vehículos». Se verificó con 89 y 140 pruebas unitarias de lo que se tocó; las e2e de cada fase en local, que la aprobación del plan autorizó, 51 en la primera y 57 más 54 de accesibilidad en la segunda; `ng build` sin advertencias, `npm run lint:styles` y las imágenes regeneradas, miradas una por una. El último pedido del responsable del mismo día —la retención con su desglose a la derecha, «Cómo se paga» en una fila de cuatro en una tarjeta ancha, desde 80rem, con un activador de la cuenta más corto, y «Quitar» para la nota, revisión 77 de `DESIGN.md`, rama `fix/payment-how-wide`— ~~está en curso~~ se construyó el mismo día (frontend `cfd96827`, `27fcb725`, `7368994f`, `015d9c25`, `a87e7907` y `c166c27b`, merge `3e53ee53`): desde 80rem, el método, la cuenta, la fecha y la operación en una fila, y la constancia junto al tipo de cambio; la caja de la cuenta, «BCP Soles PEN ···0045»; la retención con su desglose a la derecha; cada monto de un desglose en la línea de su etiqueta, en toda la aplicación; y «Quitar la nota», que devuelve el foco a «Agregar una nota». Con el merge `36206d93`, el saldo nombra su nota como «la nota NC01-50». El 11/11/2026, la revisión 78 (merge `0f6d55db`) puso lo que se paga de un documento sobre las columnas de sus cifras y la constancia plegada en dos filas junto a la fecha y la operación. El mismo día, la revisión 79 (merge `31b51341`) puso el tipo de cambio y su aviso uno al lado del otro, a lo ancho de «Cómo se paga» o en ocho columnas junto a la constancia en una tarjeta ancha; la 80 (merge `fa53171a`) plegó la constancia en la caja de la altura de un campo, en su celda, en lugar de las dos filas de la 78, y dio al aviso de la tasa la altura de su dato; y la 81 (merge `2c18a520`) hizo de esa caja la caja compacta de un archivo del sistema de diseño, con su peso tras el nombre y sus acciones como botones de ícono nombrados |
| D-246 | Registrar pago en una página: el documento, sus vehículos y con qué se paga. El responsable revisó otra vez «Registrar pago» (`/procurement/supplier-payments/new`) el 10/11/2026 y, después de una [revisión basada en evidencia](https://claude.ai/artifact/2281pRi3qUewX64cVdWEZf) y tres prototipos, el último [publicado](https://claude.ai/artifact/8nDuTVauqk28n4uta275vz), aprobó una pantalla nueva sobre las mismas reglas: no cambia ninguna de D-194 a D-215, ni la API, ni lo que se envía —`amount`, las aplicaciones con su retención y sus imputaciones, `settlesUnit` y `requestId`—, ni los límites de 100 aplicaciones, 100 unidades por documento y 500 por pago. <ul><li>**Una página, en el orden en que la persona lo piensa.** Una fila con «Proveedor», que se busca, y «Factura u orden», una lista de lo que se le debe: sus facturas y sus órdenes enviadas, estas como «Orden … · anticipo». Con más de una empresa, «Empresa» va primero en esa fila. Elegir un documento abre su sección, y «Agregar otra factura» suma otro documento al mismo pago —una transferencia, una constancia—, así que «Pagar N facturas» de Por pagar (`?facturas=`) y la copia de un pago anulado siguen abriendo el formulario con todos sus documentos. El monto del pago sigue siendo la suma de lo que paga cada documento, en la moneda del pago (D-245).</li><li>**Los vehículos de la factura.** Su sección dice arriba sus cifras —total, notas de crédito, ya pagado y por pagar— y debajo, en una tabla, cada vehículo con su estado de pago (Pagada, Pago parcial o Sin pago), lo que tiene por pagar y «Se paga ahora». «¿Qué paga de esta factura?» es una lista de tres: «Todos los vehículos pendientes», la de partida, que paga lo que se debe de la factura y no envía imputaciones; «Solo algunos vehículos», que se marcan, paga lo que les falta y envía sus imputaciones; y «Otro monto», que se escribe y no envía imputaciones, así que el backend lo reparte en proporción a lo que le falta a cada vehículo, como hoy (D-211). Una orden no tiene vehículos: su sección pide el monto del anticipo.</li><li>**El concesionario** sigue pagando vehículo por vehículo (D-197): cada vehículo lleva su monto y, cuando es menor que lo que le falta, la fila pregunta «¿La pagó el cliente como inicial?»; con un sí, la imputación salda la unidad (`settlesUnit`, la «Saldada» de hoy), y la fila lo dice como «La diferencia la pagó el cliente como inicial».</li><li>**«¿Con qué se paga?»** es una lista: Transferencia, Cheque, Efectivo y «Saldo a favor». «Saldo a favor» se ofrece solo cuando el proveedor tiene saldo en la moneda de los documentos, y se retiene, con su razón, cuando el permiso de quien registra se limita a una marca (D-195, D-212). El responsable piensa el saldo como «las notas de crédito que son saldo a favor», y la pantalla lo dice así de claro: el saldo lo dejan las notas que llegaron cuando su factura ya estaba pagada (D-190), mientras que una nota de una factura todavía impaga baja lo que se debe de ella y no es saldo. Con saldo, el pago usa a lo sumo lo disponible; si no alcanza, la factura queda con el resto por pagar. Enmienda el control segmentado de la revisión 66 de `DESIGN.md`, que retenía «Saldo a favor» entre los cuatro métodos, y deja sin adoptar la acción aparte «Usar saldo a favor» que recomendaba la revisión basada en evidencia: el responsable lo prefirió como un método más.</li><li>**La cuenta** es una lista: su nombre y, debajo, el banco, la moneda y sus últimos dígitos («BCP · USD · ···7788»). Se propone cuando es la única cuenta activa de la empresa en la moneda de los documentos, y nunca se propone la última usada; sigue fijando la moneda del pago. Enmienda D-214, que la proponía solo cuando la empresa tenía una sola cuenta activa.</li><li>**La constancia se comporta como el archivo de una factura** (`DocumentFile`, revisiones 47, 48 y 53 de `DESIGN.md`): una zona para soltarla, de la altura de un campo, hasta elegirla; elegida, se abre a la izquierda de los datos del pago, con Minimizar, Cambiar y Quitar; plegada, es una tarjeta con Expandir; en el teléfono, solo se nombra. La pantalla dice «Constancia» en todas partes y ya no «voucher». Cuándo se exige sigue D-215.</li><li>**Los errores y el botón.** Al abrir, el formulario no dice ningún error; un campo dice el suyo una vez que se editó y se dejó. «Registrar pago» está siempre habilitado: si falta algo, un resumen de errores arriba de la página lo lista, con un enlace a cada campo, y toma el foco. Reemplaza la lista de lo que falta del resumen al costado y el botón `disabledInteractive` de la revisión 66. El botón se deshabilita solo mientras guarda; `requestId` ya impide registrar dos veces el mismo pago (D-198).</li><li>**El total al pie.** Una barra al pie de la página, en todo ancho, dice el total, lo que paga y lo que falta; el resumen al costado de la revisión 66 se va. La diferencia de cambio deja el formulario y se queda en la ficha del pago (D-213), y la tasa se muestra solo cuando hay una conversión.</li><li>**Palabras llanas,** con un glosario en la pantalla: «Por pagar» por lo que se debe, en lugar del «Pendiente» que D-245 dio al formulario; «Cuenta de la que salió el dinero» por la cuenta de origen; «Constancia»; «N.º de operación»; «La diferencia la pagó el cliente»; «Retención del IGV», el 3 % que Millennium retiene y paga a SUNAT por cuenta del proveedor si es agente de retención, y que cuenta como pagado al proveedor (D-196); y «Saldo a favor».</li><li>**La retención** sigue opcional y escrita (D-196) hasta que contabilidad confirme si Millennium es agente de retención del IGV, una pregunta que queda abierta (§14).</li></ul> La sostienen NN/g (los errores hostiles y prematuros, la revelación progresiva y los asistentes por pasos, que sirven a una tarea rara y no a una diaria), los sistemas de diseño de GOV.UK y de CMS (el resumen de errores, nunca deshabilitar el botón que envía, y cuándo una lista y cuándo botones de opción), Baymard (la validación en línea y la elección del medio de pago) y CXL y Speero (una sola columna, la lista frente a los botones de opción). El responsable eligió listas para el método y la cuenta sabiendo que los botones de opción se midieron algo más rápidos. Enmienda D-214 en la cuenta que se propone, y de D-245 y de las revisiones 66 y 67 de `DESIGN.md` solo lo que toca a la pantalla del registro; la construye el frontend con la revisión 69 de `DESIGN.md`. Sin cambio en el backend ni migración (§10.1, §10.2; decisión del responsable, 10/11/2026). Decidida antes de construirla (documentación `b27c002`) y construida el mismo día (frontend `17dae178`, `8d847064`, `aad45bfb`, `df5a873e`, `da44a3d5`, `532180aa` y `83d197a4`, en `develop` con el merge `62304705`, con la revisión 69 de `DESIGN.md`; y `feed875a`, `9161e8bf`, `db35a7d8`, `8c95df65`, `7367f8c9` y `956cdcff`, con el merge `3a928954`, y `4796ec7b`, que la llevan al pie, al resumen de errores y a la regla del foco de las revisiones 70 y 71). Al construirla se fijó: el resumen de errores (`mm-error-summary`, «Faltan 3 datos para registrar el pago») no toma el foco, que el responsable pidió ese día para todo formulario de página en el primer campo con error, en el orden de la pantalla, y un rechazo del backend en lo que nombra o en la línea de error del pie (revisión 71); el pie es el de Registrar factura (`mm-form-foot`, revisión 70), con «Total a pagar» o «Se paga con saldo a favor», «Paga» y la retención, una línea que dice lo que falta en el orden de la pantalla, «Falta: constancia y n.º de operación», y «Guardando los datos…» mientras registra; en el teléfono, la barra fija de la revisión 34; «Factura u orden» espera al proveedor, se abre sola cuando se lee lo que se le debe y elige el único documento cuando hay uno, y un documento que este pago no puede pagar —otro par de monedas (D-194) o una orden de un concesionario (D-197)— se ofrece retenido con su razón, nunca se deja fuera; los documentos se envían en el orden de lo que se debe, de lo más antiguo a lo más nuevo, sea cual sea el orden en que se eligieron, a lo sumo 100; lo que se envía es el contrato de antes, comprobado contra `UnitCoveragePolicy`: «Todos los vehículos pendientes», lo que se debe y ninguna imputación; «Solo algunos vehículos», a cada vehículo marcado lo que todavía debe, su diferencia, nunca más de lo que le falta; «Otro monto», lo escrito y ninguna imputación; el concesionario, lo escrito en cada vehículo, con `settlesUnit` solo en el que recibe menos y cuya diferencia pagó el cliente; la orden, su anticipo; y cada aplicación, lo que se paga menos su retención (D-210), todo derivado de lo que la persona decide por `payment-drafts.ts`, mientras `PaymentFigures` no cambia; el concesionario no tiene «¿Qué paga de esta factura?»: cada vehículo por pagar es una casilla marcada con todo lo que debe, y pagado de menos pregunta «¿Los pagó el cliente como inicial?», con «Sí, la pagó el cliente», que se deshace; «+ Retención del IGV» abre la retención de un documento y la propone en el 3 % de lo que se paga de él, redondeado mitad hacia arriba, que se corrige a mano: sigue opcional, no se ofrece con saldo y el backend no la calcula (D-196); la cuenta se propone solo cuando todos los documentos son de una moneda y la empresa tiene una sola activa en ella, nunca con documentos de dos monedas ni en una copia, y una propuesta sigue a los documentos mientras la persona la conserva; con saldo, el pago usa a lo sumo lo disponible, los primeros documentos elegidos primero —«Todos los vehículos pendientes» y «Otro monto» bajan sin imputaciones, «Solo algunos vehículos» lo reparte entre los marcados, y a un concesionario la pantalla no lo baja: pide bajar lo de sus vehículos—, pasar a «Saldo a favor» quita las órdenes y las facturas de otra moneda y lo dice, «Se quitó OC-GEN-2026-0001: el saldo a favor solo paga facturas en PEN.», hasta que cambien el método o los documentos, y un método que mueve dinero recuerda el saldo disponible; «Constancia» se lee en el formulario, en los rechazos del registro (`SUPPLIERPAYMENT_VOUCHER_*`, `SUPPLIERPAYMENT_FILE_*`) y en la ficha del pago («Cambiar constancia», «Adjuntar constancia», «Descargar constancia»), cambiando solo valores de `es.json`; «Por pagar» es la palabra de este formulario, y «Pendiente» sigue en las demás pantallas mientras el responsable no decida otra cosa (§14); y se van `payment-documents`, `payment-unit-allocations` y `payment-summary`, que reemplazan `payment-document-panel` y `payment-drafts.ts`. Se verificó con 157 y después 154 pruebas unitarias de lo que se tocó, las dos e2e de los pagos (`supplier-payments.spec.ts` y `payment-account-voucher.spec.ts`) en local, con la aprobación del responsable, dos veces: 54 pasan y 96 se omiten a propósito, en la tableta y el teléfono; `ng build` sin advertencias y `npm run lint:styles` sin errores. Enmendada por D-247: «Solo algunos vehículos» lleva el monto de cada vehículo marcado, de partida lo que debe y editable; con saldo, el monto propuesto no pasa de lo que queda del saldo y la pantalla pide bajar lo de los vehículos, en lugar de repartirlo entre los marcados; la copia de un pago con imputaciones elegidas conserva sus montos en «Solo algunos vehículos»; «Otro monto» queda para el pago que no elige vehículos; y «Cómo se paga» va en dos filas (revisión 75 de `DESIGN.md`). Enmendada por D-248: el saldo lo dejan las notas que llegaron cuando su vehículo, no toda su factura, ya estaba cubierto, y el backend guarda el reparto de «Todos los vehículos pendientes» y de «Otro monto» al registrarlos. Enmendada el 11/11/2026 por la revisión 85 de `DESIGN.md` (decisión del responsable de ese día; frontend `dff4a839`, `449e6007`, `2a59f4fc`, `abc4d39b`, `4b50003c` y `55c97ba6`, merge `28da7f89`): «Cómo se paga» aparece solo después de elegir una factura u orden, o con «Saldo a favor», y antes el pie pide solo «Falta: proveedor» o «Falta: factura u orden»; un proveedor al que no se le debe nada es un estado vacío, con su saldo a favor, «Elegir otro proveedor» y «Ver el proveedor»; una transferencia o un cheque, antes de su cuenta, se toman en la moneda de sus documentos cuando la comparten, y no en la funcional, y la cuenta sigue fijando la moneda (D-214); la conversión se dice en palabras, «Se paga en soles: USD 53,800.00 son PEN 184,211.20 al tipo de cambio 3.424 del 10/11/2026.», con «(no hay del 11/11/2026)» cuando la tasa es de un día hábil anterior; y una orden dice por qué no tiene vehículos (D-211). El mismo día, la ayuda de la retención dice de qué sale el 3 %, «3 % de PEN 5,900.00. Si el comprobante dice otro monto, escríbalo.», y escrita a mano, «Monto escrito a mano.» con «Volver al 3 %», que devuelve la propuesta (frontend `48a7a99e` y `f0ff6d4b`, merge `df4b57cd`) |
| D-245 | La experiencia de los pagos a proveedores, rediseñada sobre las mismas reglas (D-194 a D-198, D-210 a D-215), aprobada por el responsable el 09/11/2026 sobre la [propuesta](https://claude.ai/artifact/AWriMyMNW1MTtsuePGfdGq). El monto del pago ya no se escribe: es la suma de lo marcado en «Qué se paga», en la moneda del pago, así que el pago se aplica completo por construcción (D-210); el backend sigue recibiendo `amount` y comparándolo, sin cambio. Un solo vocabulario en Compras y en la ficha de la unidad: «Pendiente» para lo que se debe, «Pagado» para lo aplicado más la retención, «Asignar a unidades» por imputar, «Concesionario» por dealer; «Saldo a favor» se queda para el crédito del proveedor. Un pago puede empezar con varias facturas del mismo proveedor y empresa, desde «Por pagar» o desde la ficha del proveedor. La lista de pagos se busca también por el número de los documentos que pagó (`q`), y cada fila trae sus documentos (`documents`: tipo, identificador público y número, en el orden en que se aplicaron), para decir qué pagó. Sin migración. Enmienda la regla de diseño de «el monto nunca se llena solo» (DESIGN, revisión 52) y la de «sin tarjetas en el teléfono» (revisión 37) (decisión del responsable, 09/11/2026). Construida el mismo día (backend `70dd253b`, `8f84b19c` y `0ca99f14`, en `develop` con el merge `b5308771`; frontend `e5741f38`, `78485bcd`, `dd080bef`, `0eb878ad`, `b410c2fb`, `342c22eb` y `1f00e2e3`, en `develop` con el merge `5f728fc3`, con la revisión 66 de `DESIGN.md`, el formulario; `197d663b` a `9a6c0f0c`, once commits, con el merge `7ae68c2d` y la revisión 67, lo que lo rodea; y `a0724b7a`, los textos de los rechazos y del permiso). Al construirla se fijó: `documents` nombra cada documento una sola vez, en el orden de su primera aplicación, aunque el pago lo pague dos veces —por su aplicación propia y por otra que pasó de su orden (D-198)—, con `type` `INVOICE` u `ORDER`, su identificador público y su número como se muestra, el de la factura o el de la orden (`SERIE-AAAA-0001`); viaja también en la ficha, y se arma con las aplicaciones que la página ya lee, así que una página prepara las mismas sentencias con 1, 10 o 40 filas, también buscada por el número de un documento: 17, contadas con las estadísticas de Hibernate (`SupplierPaymentListStatementsTests`); `q` encuentra la factura por su número como se muestra y como lo guarda su registro (D-160), así que `F002-00007741` encuentra `F002-7741`, y la orden por su número como se muestra, sin mayúsculas ni tildes (D-142), con una sola expresión del número de la orden, `PurchaseOrderRepositoryImpl.numberOf`, para la búsqueda de las órdenes y la de los pagos; la exportación no cambia de columnas, pero su versión sube a `supplier-payments-layout=3`, porque su búsqueda encuentra otras filas y un archivo generado antes no se sirve otra vez; la pantalla suma los equivalentes de las filas marcadas, cada uno redondeado mitad hacia arriba como el backend, así que ninguna fila absorbe un céntimo, y una fila marcada toma todo lo pendiente, sin tope; las unidades se asignan dentro de la fila del documento y el diálogo de imputación desaparece, y la factura de un concesionario llega con el reparto en proporción ya propuesto; `?facturas=` lleva hasta 100 facturas, y Por pagar no reemplaza lo elegido cuando se marca una de otro proveedor, sino que lo dice y retiene el pago; desde la ficha del proveedor, «Pagar lo pendiente» abre el formulario con el proveedor y su empresa, que lista lo pendiente sin marcarlo; la pantalla dice todas a la vez las razones que retienen un pago, con una nueva, el total que pasa el saldo a favor, que el backend ya rechazaba (D-195); y el vocabulario cambia solo valores de `es.json`, nunca una clave, también en los rechazos del registro y en el nombre de `pagos.registrar_pago`, «Registrar pagos y asignarlos a unidades». La lista de pagos lee seis columnas, con «Pagó» y «Cómo», y la ficha del pago abre con una franja de quién se pagó, cuánto, cómo, cuándo y su estado. «Compra y costo» de la unidad conserva seis decimales en su tasa, porque `fleet` no lee ayudas de compras. Enmendada por D-246, solo en la pantalla del registro: ya no va en dos pasos con un resumen al costado ni marca documentos como filas, sino en una página, con el proveedor y la factura u orden en una fila, los vehículos de cada factura con lo que se paga de ella, el método y la cuenta como listas, la constancia como el archivo de una factura, un resumen de errores arriba y el total al pie (revisión 69 de `DESIGN.md`, que enmienda la 66); y en el formulario lo que se debe se lee «Por pagar». El monto calculado, `?facturas=`, `documents`, la búsqueda y lo demás de D-245 siguen |
| D-244 | El estado de pago de una factura, derivado y nunca guardado, con el vocabulario del de sus unidades (D-199, D-212). Se calcula con `owed = total − notas − aplicado` (`AmountOwedPolicy`; lo aplicado suma el monto y la retención de cada aplicación de un pago vigente) y, en el canal de dealers, `payable = owed − iniciales de clientes de sus unidades saldadas` (D-197). Una factura anulada no tiene estado. Una vigente está en `ANULADA_POR_NOTAS` si sus notas vigentes cubren todo su total —el texto que ya lee una línea cuyas notas la llevan a cero—; en `SALDADA` si `payable ≤ 0` y hay iniciales saldadas; en `PAGADA` si `payable ≤ 0`, también pagada de más, que el desglose dice aparte; en `SIN_PAGO` si nada se le aplicó; y en `PARCIAL` en cualquier otro caso. La lista de facturas (`GET /api/v1/purchase-invoices`) y su ficha devuelven `paymentStatus` a quien las lee con `compras.ver`, como D-212 deja ver el estado de pago de cada línea; lo aplicado (`appliedTotal`) y lo que se debe (`owed`) solo a quien tiene `pagos.ver` en la empresa de la factura, nulos para cualquier otro. La lista se filtra por `paymentStatus`; con ese filtro se busca sin paginar, se calcula el estado y se pagina en memoria, como «Por pagar» y los reportes (D-236), porque lo que se debe no es una columna y el de un dealer depende de la cobertura de sus unidades, que no se escribe dos veces: sirve mientras una empresa tenga unos miles de facturas, y pasarlo a SQL es el seguimiento si crece. La exportación suma la columna «Estado de pago», sin montos, porque exportar solo pide `compras.ver`, y respeta el filtro. Las filas de la lista llevan además sus `allowedActions`, con la regla de la ficha, para que su menú ofrezca registrar un pago o una nota y descargar el comprobante. Y cada aplicación de la posición de pago de una factura nombra la cuenta de la que salió el pago (`bankAccountName`, D-214), nula sin cuenta. Sin migración (§9, §10; decisión del responsable, 09/11/2026). Construida el mismo día (backend `edd12f9a`, `e59fb619`, `b250636d`, `8fc04028`, `eb80ced8` y `6230346b`, en `develop` con el merge `4d15256d`; frontend `8fa29b6e`, `3dfca05d`, `0c59fd66`, `93da593f`, `5700020a`, `bee5c95b`, `ac5569e1`, `258d3766` y `6cbf7243`, en `develop` con el merge `cf862569`, con la revisión 63 de `DESIGN.md`). Al construirla se fijó: `InvoicePaymentStatusPolicy` decide el estado e `InvoicePaymentStatus` lo nombra; `appliedTotal` y `owed` son nulos también en una factura anulada, aunque quien lee tenga `pagos.ver`; el `owed` de un dealer sigue contando las iniciales de sus unidades saldadas, así que una factura `SALDADA` puede deber más que cero, y lo explica `settledInitials` de su posición; el filtro acepta los cinco códigos en cualquier mayúscula, uno fuera de ellos responde `400 VALIDATION_ERROR` en `paymentStatus`, y una factura anulada no entra nunca en él; «Estado de pago» va en la exportación después de «Estado», vacía en una anulada, con las claves `export.purchase-invoices.payment-status.sin-pago`, `parcial`, `pagada`, `saldada` y `anulada-por-notas`, y la versión del archivo sigue también la de las notas y la de los pagos, que cambian el estado; y las filas de una página leen una sola vez lo que su pago y sus acciones necesitan —sus dependientes, por `PurchaseInvoiceDependents.invoicesWithAny(Set<Long>)`, y la zona de cada empresa, por `ExternalOrganizationService.fetchCompanyTimeZones`—, así que una página prepara las mismas sentencias con 2, 20 u 80 filas: 17, y 20 filtrada por el estado de pago, contadas con las estadísticas de Hibernate (`PurchaseInvoiceListStatementsTests`). La ficha de la factura dice su pago en un resumen sobre sus datos, «Pagos» queda solo con la tabla de sus pagos, con la cuenta de cada uno, y la lista suma la columna «Pago» y los filtros Tipo y Pago |
| D-130 | Las ocho acciones `gestionar_*` de las pantallas ya construidas se parten en crear, editar y desactivar (y, en roles, marcar permisos). Enmienda D-118 solo para esos ocho: `catalogos.gestionar_generales`, `catalogos.gestionar_flota`, `catalogos.gestionar_estados`, `configuracion.gestionar_empresas`, `configuracion.gestionar_sedes_y_marcas`, `configuracion.gestionar_ubicaciones`, `seguridad.gestionar_colaboradores` y `seguridad.gestionar_roles`. Los treinta y nueve códigos nuevos los siembra V62. Los ocho gruesos siguen en `permissions`, para no reescribir la historia de `role_permissions`, y ninguna ruta los pide; `GET /permissions` y `GET /permission-matrix` no los ofrecen. La semilla desmarca esos ocho en los roles del sistema y marca los treinta y nueve en el superadministrador; el administrador de empresa recibe `registrar_colaboradores` y `modificar_colaboradores` en lugar de `gestionar_colaboradores`. Un rol propio que todavía tenga un código grueso no se reescribe: se vuelve a marcar a mano. Crear, copiar, renombrar, activar o desactivar y marcar permisos de un rol son cuatro permisos de alcance sistema (`crear_roles`, `editar_roles`, `desactivar_roles`, `marcar_permisos_de_roles`), que reemplazan a `gestionar_roles` en D-122 y D-123. `registrar_colaboradores` y `modificar_colaboradores` entran en los mismos pares que `gestionar_colaboradores` (D-119): veintiocho pares. Una pantalla que todavía no existe no tiene casilla hasta el paso que la construye, y entonces nace ya partida. Exportar sigue pidiendo el permiso de lectura. Los escalones de un país no se desactivan (D-45): declararlos y corregirlos es `editar_paises`; activar o desactivar una división o un documento tributario es `desactivar_paises` (§5.6). `compras.gestionar_plantillas` y `compras.gestionar_ordenes`, que nacieron gruesos con sus pantallas en el paso 7, se parten igual en V79: tres de plantillas y tipos (`crear_plantillas`, `editar_plantillas` y `desactivar_plantillas`) y cinco de órdenes (`crear_ordenes`, `editar_ordenes`, `enviar_ordenes`, `cumplir_ordenes` y `anular_ordenes`). Ningún rol pierde lo que hacía: el superadministrador y el administrador de empresa reciben los ocho, Operaciones los cinco de órdenes y Contabilidad ninguno. Desde V79 ninguna ruta pide los dos gruesos, y V80 los desmarca en todos los roles, también en los propios. **Excepción en los roles propios:** esos dos gruesos estaban vigentes desde el 23/10/2026, así que V79 marca, firmado por `SYSTEM`, los finos que reemplazan al grueso vigente de cada rol propio, para que no pierda la escritura en silencio. V80, antes de desmarcar, vuelve a marcarlos en el rol propio que tiene un grueso vigente y ninguno de sus finos —uno que guardó, mientras se desplegaba V79, una instancia con el código anterior— y deja como está al que ya tiene alguno, porque los demás los desmarcó un administrador; después de desmarcar, se detiene si queda un grueso vigente. No hay pares incompatibles en compras |
| D-10 | Los datos personales van en `employees`, el registro de colaboradores. `users` guarda solo lo necesario para entrar |
| D-25 | Colaboradores y cuentas son tablas distintas: hay personal sin acceso al sistema, y el usuario del sistema no es una persona |
| D-26 | Un contrato no puede quedarse sin asesora: al retirar a alguien se exige el reemplazo en la misma operación |
| D-21 | La factura puede indicar su orden de compra, de forma opcional: con el vínculo el pendiente se calcula, sin él la orden se cierra a mano. Precisada por D-162: cumplirla es manual también con el vínculo |
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
