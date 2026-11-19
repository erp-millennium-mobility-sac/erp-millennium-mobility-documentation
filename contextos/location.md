# Contexto `location` — ERP Millennium Mobility

**Estado:** vigente, validado por el responsable · **Fuente:** diccionario de datos (§2, §3, §4.1 a §4.8, §5.6 y las decisiones que cita cada regla), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) y el código de `develop`. Si este documento y el diccionario no coinciden, manda el diccionario.

---

## 1. Qué es y qué no es

`location` guarda los datos que comparten todas las empresas y la localización de cada país: con qué monedas se opera, qué países existen, cómo se divide su territorio, con qué documento se identifican ante su Estado las empresas y las personas, y las direcciones físicas (D-44). Guarda también los tipos de cambio: la tasa de cada día que publica la autoridad de un país, con la que un documento de ese país convierte una moneda en otra (§4.8, D-148). No depende de ningún otro contexto.

**Las siete tablas que posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `currencies` | Las monedas: PEN, USD, COP, MXN | §4.1 |
| `countries` | Los países, con su moneda sugerida, su zona horaria, su impuesto general y el tipo de tasa que proponen sus compras | §4.2 |
| `geo_division_levels` | Los escalones de cada país: Departamento, Provincia, Distrito | §4.4 |
| `geo_divisions` | Las divisiones concretas, con su código oficial (el ubigeo en Perú) | §4.5 |
| `addresses` | Las direcciones de empresas, sedes, ubicaciones, proveedores y emisores de plantillas de orden | §4.6 |
| `tax_id_types` | Los tipos de documento tributario de cada país: RUC, DNI, NIT, RFC | §4.7 |
| `exchange_rates` | Los tipos de cambio de cada país por día, con su tipo y su fuente: el dólar en soles, de venta, que publica la SBS para Perú | §4.8 |

Cómo se relacionan las tres tablas de geografía está en §4.3.

**Lo que parece suyo y no lo es**

| Parece de `location` | Es de | Por qué |
|---|---|---|
| La tabla `locations` (patio, sede, dealer) | `organization` (§6.5, D-6) | "Ubicación" ahí es el lugar donde puede haber vehículos, no un punto del mapa (mapa §1) |
| Si un país opera | `organization` | Se deduce de si tiene una empresa activa; no se guarda en el país (D-3) |
| La moneda de grupo | Nadie todavía | Está diseñada en §4.8 (D-46); se construye cuando haya dos monedas funcionales que reunir en un mismo reporte (sección 6) |
| La moneda funcional de una empresa | `organization` | La empresa apunta a una moneda de `location`, pero elegirla es una decisión de la empresa (§6.2) |
| El número de RUC de una empresa o el DNI de un colaborador | `organization` | `location` define el tipo de documento; el número lo guarda quien se identifica con él |
| La zona horaria de una sede | `organization` | El país solo propone una zona por defecto; la hora real es la de cada sede (§4.2) |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Escalón | Cada nivel en que un país divide su territorio: Departamento (1), Provincia (2), Distrito (3). Es la palabra del diccionario y de las pantallas | "Nivel", que es solo el nombre de la columna. En pantalla no se usa |
| División | Un pedazo concreto del territorio: Lima, Chorrillos. Cuelga de una división del escalón inmediato superior | El escalón, que es la categoría |
| Escalón más bajo | El último escalón que declaró el país: el distrito en Perú, el municipio en Colombia. Una dirección apunta a una división de ese escalón | — |
| Corregir una dirección | Se escribió mal: se modifica la misma fila (D-8) | Mudarla: el dueño se fue a otro lugar, y la fila nueva deja la anterior para los documentos que la citan |
| Tipo de documento tributario | El documento con que una empresa o una persona se identifica ante el Estado: RUC, DNI, NIT, RFC | "Documento de identidad", que es el dato del colaborador |
| Moneda sugerida | La que el país propone al registrar algo en él. Es solo una sugerencia | Moneda funcional, la de los libros de una empresa |
| Tasa | Cuántas unidades de la moneda de destino vale una de la de origen, un día, en un país: 3.752 soles por dólar. Lleva su tipo y la autoridad que la publicó (§4.8) | Un monto: la tasa no lleva moneda propia, la nombra su par |
| Serie | Las tasas de un mismo país, par de monedas y tipo: el dólar en soles de venta de Perú. La tasa anterior de una tasa es la más reciente de su serie de un día anterior al suyo | La llave de una tasa, que suma el día |

---

## 2. Ciclo de vida de cada agregado

Ninguna fila de `location` se borra (LOC-R07). Lo que deja de usarse se desactiva, salvo los países, los escalones y las tasas de cambio, que no tienen estado. Las tablas registran cuándo se creó y cuándo cambió cada fila, sin autor, porque son catálogos generales; las tasas guardan además quién las registró y quién las corrigió por última vez (LOC-R40).

### Moneda

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador la crea en Catálogos › Monedas (D-43). Llegan sembradas PEN, USD, COP y MXN |
| Qué cambia | El nombre, el símbolo y, mientras ningún monto use la moneda, los decimales (LOC-R04, LOC-R05) |
| Qué queda fijo | El código ISO, desde el alta (LOC-R03). Los decimales, desde el primer monto que use la moneda (LOC-R05) |
| Retiro | Se desactiva: no se elige en operaciones nuevas, y el país o la empresa que ya la tenía la conserva (LOC-R06) |
| Nunca | Se borra |

### País y su estructura territorial

El país incluye sus escalones: se guardan y se validan con él (D-45).

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador lo registra en su propia ficha. Llegan sembrados Perú, Colombia y México, con sus escalones. Un país nuevo propone para sus compras la tasa `oficial` (LOC-R53) |
| Qué cambia | El nombre, el código telefónico, la moneda sugerida, la zona horaria, el impuesto y su tasa, y si exige código postal. Los escalones se declaran uno por uno, siempre debajo del último, y se renombran (LOC-R15) |
| Qué queda fijo | Los dos códigos ISO, desde que algo cuelga del país (LOC-R10). El número de cada escalón, desde que se declara. El tipo de tasa de sus compras, que la API no edita (LOC-R53) |
| Retiro | No tiene. Un país deja de operar cuando se desactivan sus empresas (D-3) |
| Nunca | Se borra un país; se borra o se retira un escalón (LOC-R16); se guarda si el país opera |

### División territorial

| Momento | Qué pasa |
|---|---|
| Alta | La lista oficial de Perú llega con la carga inicial, y cada distrito que una ley crea después, con su propia migración (F6). Las de Colombia y México se cargan antes de registrar su primera empresa. El superadministrador también crea divisiones una por una en la ficha del país |
| Qué cambia | Solo el nombre |
| Qué queda fijo | El código oficial, el escalón, la división superior y el país, desde el alta (LOC-R18, LOC-R19) |
| Retiro | Se desactiva cuando se fusiona o desaparece; las direcciones ya registradas siguen apuntando a ella (LOC-R23) |
| Nunca | Se borra |

### Tipo de documento tributario

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador lo crea en la ficha del país. Llegan sembrados RUC, DNI, CE y PAS en Perú; NIT y CC en Colombia; RFC y CURP en México |
| Qué cambia | El nombre, a quién aplica (LOC-R33), la longitud (LOC-R34) y si tiene dígito de control |
| Qué queda fijo | La sigla y el país, desde el alta (LOC-R32) |
| Retiro | Se desactiva. Quien ya lo usa sigue siendo válido; un registro nuevo no puede elegirlo (LOC-R36) |
| Nunca | Se borra |

El dígito de control lo calcula `location` según la sigla del tipo en su país, para el contexto que guarda el número (LOC-R35): hoy, solo el del RUC de Perú, con el módulo 11 de SUNAT; el DNI no lo tiene (D-76). El NIT, el RFC y la CURP aceptan sus números sin verificar hasta la primera empresa de su país (D-52; sección 6).

### Dirección

La dirección no tiene pantalla, menú ni ruta propia: vive dentro de su dueño y se escribe con el permiso del dueño (mapa §6.3, convenciones §3.3). Se ve y se escribe por el país de la dirección (diccionario §2.5, D-80, D-134).

| Momento | Qué pasa |
|---|---|
| Alta | La crea el dueño al registrarse o al poner su primera dirección: empresas, sedes, ubicaciones de terceros y proveedores. Una ubicación propia usa la dirección de su sede (D-79). El emisor de una plantilla de orden de compra recibe una copia fiel del domicilio fiscal de su empresa, en una fila propia (D-134) |
| Qué cambia | En una corrección, la calle, el número, el interior, la referencia, el código postal y la división, sobre la misma fila (LOC-R30) |
| Qué queda fijo | El país: es el del dueño, y quien escribe la dirección no lo elige (LOC-R28) |
| Retiro | No tiene. En una mudanza el dueño pasa a apuntar a una fila nueva y la anterior queda para los documentos que la citan (LOC-R30) |
| Nunca | Se comparte entre dos dueños, aunque el texto coincida (LOC-R29); se borra |

### Tasa de cambio

Una tasa es de un país, el de la autoridad que la publica, y la usan todas sus empresas (LOC-R50). La identifican su país, su par de monedas, su tipo y su día (LOC-R42).

| Momento | Qué pasa |
|---|---|
| Alta | La registra a mano, cuando un documento la necesita, en Catálogos › Tipos de cambio o desde el formulario del documento (LOC-R45, D-151). No llega ninguna sembrada, y la carga automática espera una fuente (sección 6) |
| Qué cambia | La tasa y su fuente, en una corrección con motivo que queda en la bitácora (LOC-R47) |
| Qué queda fijo | El país, las dos monedas, el tipo y el día, desde el alta: una tasa del día equivocado es otra tasa. Corregir una tasa no cambia los documentos que ya la copiaron (§4.8) |
| Retiro | No tiene: una tasa no tiene estado (LOC-R49) |
| Nunca | Se borra; se registra para un día posterior a mañana en su país (LOC-R48); lleva la fuente `manual` (LOC-R43) |

---

## 3. Flujos funcionales

Las lecturas piden `catalogos.ver`, que tienen los siete roles, porque toda pantalla con un selector los necesita; cada escritura, el permiso de su operación, que la semilla da solo al superadministrador (LOC-R38; cuál cubre cada operación, diccionario §5.6). La dirección (F5) sigue el permiso de su dueño. Un permiso de escritura no abre ninguna pantalla (D-141), y uno retirado, como `catalogos.gestionar_generales`, no abre ni escribe nada.

### F1 · Mantener las monedas

**Rol:** superadministrador. **Pantalla:** Catálogos › Monedas.

1. Busca por código, nombre o símbolo, sin distinguir mayúsculas ni tildes (D-142), y filtra por estado.
2. Crea una moneda con su código, nombre, símbolo y decimales. El sistema revisa el código (LOC-R01, LOC-R02) y los decimales (LOC-R04).
3. Edita el nombre, el símbolo o los decimales; el código no (LOC-R03). Los decimales cambian solo mientras ningún monto use la moneda (LOC-R05).
4. Exporta la lista a Excel con la misma búsqueda (LOC-R39).

**Si algo falla:** el diálogo lo dice y queda abierto con lo escrito (`CURRENCY_CONFLICT`, `CURRENCY_DECIMAL_PLACES_IN_USE`).
**Resultado:** la moneda queda disponible en los selectores de países y empresas.

### F2 · Registrar un país y su estructura

**Rol:** superadministrador. **Pantalla:** Catálogos › Países y la ficha del país (`/location/countries/new` y `/location/countries/{id}`).

1. **Alta** (`POST /countries`). Escribe los dos códigos ISO, el nombre, el código telefónico, la moneda sugerida, la zona horaria, el impuesto y su tasa, y si exige código postal. El sistema revisa los códigos (LOC-R08, LOC-R09), el código telefónico (LOC-R11), la zona horaria (LOC-R12), la tasa del impuesto (LOC-R13) y que la moneda esté activa (LOC-R06).
2. **Escalones.** Declara el primero por su nombre, después el siguiente, hasta cuatro; el número lo pone el sistema (LOC-R14, LOC-R15). Un escalón solo se renombra (LOC-R16).
3. **Divisiones.** Elige el escalón, busca la división superior entre las del escalón de arriba y escribe el código oficial y el nombre. El sistema revisa el escalón (LOC-R17), el código (LOC-R18) y la división superior (LOC-R20, LOC-R21, LOC-R22).
4. **Documentos tributarios.** Crea cada tipo con su sigla, nombre, a quién aplica, longitud y dígito de control. El sistema revisa la sigla (LOC-R31) y la longitud (LOC-R34).

**Si algo falla:** el diálogo dice qué regla no se cumplió y nada se guarda.
**Resultado:** el país queda listo para registrar empresas en él. Para que una dirección pueda elegir distrito, además hacen falta sus divisiones (F6).

### F3 · Corregir los códigos ISO de un país

**Rol:** superadministrador. **Pantalla:** ficha del país (`PUT /countries/{id}`).

1. Mientras nada cuelga del país, corrige los códigos; el sistema revisa otra vez LOC-R08 y LOC-R09.
2. Con un escalón, una dirección, un tipo de documento o una tasa de cambio, los códigos quedan fijos (LOC-R10), y la ficha los muestra bloqueados. Una tasa cuenta porque su exportación nombra el país por su código: corregirlo cambiaría de qué país dice ser una tasa que ya movió montos.

**Si algo falla:** `COUNTRY_ISO_CODES_LOCKED`; como la ficha ya bloquea los campos, solo se ve en una carrera.
**Resultado:** los códigos corregidos. Dentro del sistema nada se rompe, porque las demás tablas apuntan al país por su número interno (§4.2).

### F4 · Retirar una moneda, una división o un tipo de documento

**Rol:** superadministrador. **Pantallas:** Monedas, y la ficha del país para divisiones y documentos.

1. Desactiva el registro; el sistema pregunta antes. Sigue en la lista, inactivo, y se puede volver a activar.
2. Lo que ya lo usaba sigue válido: una dirección conserva su división, un país o una empresa su moneda y una empresa su tipo de documento (LOC-R06, LOC-R23, LOC-R36).

**Resultado:** el registro deja de ofrecerse en los formularios nuevos, y la API rechaza elegirlo como nuevo; para la moneda, con `CURRENCY_INACTIVE` (LOC-R06).

### F5 · Capturar una dirección desde su dueño

**Rol:** el que escribe al dueño. **Pantalla:** el diálogo del dueño —empresa, sede y ubicación en `organization`, proveedor en `procurement`—, con el campo de dirección que presta `location`.

1. El país de la dirección es el del dueño; la persona no lo elige (LOC-R28).
2. La división es obligatoria cuando el país tiene sus divisiones cargadas, como Perú, y opcional en uno sin ellas (LOC-R41). Se elige de arriba hacia abajo, entre las activas que cuelgan de lo elegido arriba, hasta el último escalón (LOC-R26). Si el formulario no pudo leer las reglas del país, deja guardar los demás datos del dueño mientras la dirección no cambie. Una dirección vacía del todo sigue permitida si el dueño no la exige.
3. La calle es obligatoria en cuanto se escribe algo (LOC-R24); el código postal, solo en los países que lo exigen, como México (LOC-R27).
4. Al cambiar la dirección de un dueño ya guardado, el diálogo pregunta si es una corrección o una mudanza (LOC-R30) y la guarda por el subrecurso de dirección del dueño, aparte de sus demás datos (convenciones §3.3). La primera dirección se registra sin preguntar. Quitar la dirección no se permite: solo corregirla o mudarla.
5. Al editar un dueño, el formulario reconstruye la cadena de su división (Lima › Lima › Chorrillos). La división que ya tenía la dirección se acepta aunque se haya desactivado después; una que se elige ahora tiene que estar activa (LOC-R26).

**Si algo falla:** nada se guarda, y el formulario dice qué falta o, desde el backend, el código `ADDRESS_*` de la regla rota (convenciones §3.3).
**Resultado:** el dueño apunta a su dirección. Al registrar el dueño, la dirección va en su misma transacción: si algo falla después, tampoco se guarda. Al editarlo, la dirección se guarda por su subrecurso antes que los demás datos, en otra petición; si después falla la edición del dueño, la dirección ya quedó guardada.

### F6 · Cargar el ubigeo

La lista oficial de Perú (ubigeo del INEI, edición 2022) se carga completa con una migración (§4.5, D-2; procesos, P1 paso 1), generada desde el archivo tal como lo publica el INEI, que se conserva para compararlo con una edición nueva. Los nombres quedan como los publica el INEI —en mayúsculas, sin tildes y con la Ñ— y así se muestran (`LIMA › LIMA › CHORRILLOS`), por decisión del responsable del diccionario.

Una edición nueva no reemplaza esa carga. Cada distrito que una ley crea después entra activo con su propia migración en cuanto el INEI o SUNAT publican su código; el distrito del que se separó sigue activo, y los que se fusionan se desactivan (D-77). Santa Rosa de Loreto (Ley 32403) es el primero y, mientras espera su código, no se puede elegir (sección 6). Volver a cargar la lista desde la aplicación no tiene mecanismo (sección 7).

### F7 · Registrar y corregir un tipo de cambio

**Rol:** registra quien tiene `catalogos.registrar_tipo_cambio` y corrige quien tiene `catalogos.corregir_tipo_cambio`: con la semilla, el superadministrador, y la persona de Contabilidad que carga las tasas, por un rol propio general de alcance sistema (D-149, D-150; LOC-R38). **Pantalla:** Catálogos › Tipos de cambio.

1. **La lista.** Empieza en Perú, con sus tasas del día más reciente al más antiguo, y se filtra por país, moneda de origen, tipo y período. Cada tasa muestra su última corrección: cuándo y por quién (LOC-R47). Se exporta a Excel sin quien corrigió (LOC-R52).
2. **Registrar** (`POST /exchange-rates`) el país, las dos monedas, la fecha, el tipo, la tasa y la fuente. El sistema revisa, en este orden, la forma de cada campo (LOC-R43, LOC-R44), que el país y las monedas existan, que las monedas sean distintas y estén activas (LOC-R43, LOC-R06), la fecha (LOC-R48) y, con la serie de la tasa bloqueada (LOC-R51), que esa tasa no exista ya (LOC-R42) y cuánto se aparta de la anterior (LOC-R46).
3. **Un salto grande.** Si la tasa se aparta de la anterior de su serie más que el umbral, el backend responde `EXCHANGERATE_LARGE_CHANGE_UNCONFIRMED` con la anterior y la pantalla pregunta si es correcta: revisarla vuelve al diálogo con lo escrito, sin guardar nada; confirmarla manda el mismo pedido otra vez, confirmado (LOC-R46).
4. **Corregir** (`PUT /exchange-rates/{id}`) la tasa, la fuente o las dos, con un motivo; una tasa del día equivocado es otra tasa, que se registra aparte (LOC-R47). Si otra corrección guardó la tasa después de abrirla, nada se guarda y la lista se vuelve a leer. Un salto grande pregunta igual que en el alta.
5. **La bitácora.** Cada campo corregido queda en Seguridad › Bitácora de cambios como un cambio delicado y sin empresa (§13.1, LOC-R47, LOC-R50).

**Si algo falla:** el diálogo lo dice y queda abierto con lo escrito (`EXCHANGERATE_CONFLICT`, `EXCHANGERATE_SAME_CURRENCIES`, `CURRENCY_INACTIVE`, `EXCHANGERATE_DATE_TOO_LATE`, `EXCHANGERATE_OUTDATED`, o `403` sin el permiso).
**Resultado:** la tasa queda en la lista de su país, para los documentos de su día (sección 5).

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md).

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| LOC-R01 | El código de una moneda son tres letras de una moneda real de ISO 4217 | §4.1, §2.4 | Base (solo la forma), Dominio, Servicio, Frontend | `IsoCurrencyCodeTests`, `CurrencyTest`, `CurrenciesControllerTests` | ✅ |
| LOC-R02 | Dos monedas no tienen el mismo código | §4.1 | Base, Servicio | `CurrencyCommandServiceImplTest`, `CurrenciesControllerTests`, `LocationRacedDuplicatesTests` | ✅ |
| LOC-R03 | El código de una moneda no se edita nunca | §2.4, D-48 | Dominio, Frontend | `CurrencyTest` | ✅ |
| LOC-R04 | Una moneda tiene de 0 a 2 decimales, la escala con que se guardan todos los montos: así ningún monto en una moneda se redondea en silencio al guardarse. Una moneda de 3 decimales reabre la decisión, junto con la escala de los montos | §4.1, D-146 | Base, Dominio, Servicio, Frontend | `CurrencyTest`, `CurrenciesControllerTests`, `CurrencyDecimalPlacesMigrationTests`, `currency-form-dialog.spec.ts` | ✅ |
| LOC-R05 | Los decimales quedan fijos desde el primer monto que usa la moneda; antes se corrigen. Cuenta cualquier orden de compra, factura o pago a proveedor en esa moneda, de cualquier empresa y en cualquier estado, también anulado, y el costo de un equipo de una unidad; los fondos no tienen moneda. El nombre y el símbolo se corrigen siempre | §4.1, D-138 | Dominio, Servicio, Frontend | `CurrencyTest`, `CurrencyCommandServiceImplTest`, `CurrencyDecimalPlacesIntegrationTests`, `CurrencyDecimalPlacesLockTests` (`procurement`), `CurrencyUsageWiringTests`, `currencies.store.spec.ts`, `e2e/currencies.spec.ts` | ✅ |
| LOC-R06 | Una moneda inactiva no se usa en operaciones nuevas; quien ya la tenía la conserva | §4.1 | Dominio, Servicio, Frontend | `CountryCommandServiceImplTest`, `CountriesControllerTests`, `FunctionalCurrencyTests`, `CompanyCommandServiceImplTests`, `CompaniesControllerTests`, `PurchaseOrdersIntegrationTests`, `ExchangeRateTests`, `ExchangeRatesIntegrationTests`, `ExchangeRateLockTests`, `register-exchange-rate-dialog.spec.ts`, `e2e/country-profile.spec.ts` | ✅ |
| LOC-R07 | Una moneda, un país, una división, un tipo de documento o una tasa de cambio no se borran (LOC-R49) | §4.1, §4.5, §4.7, §4.8 | Base, Servicio | `CurrencyAuditTests`, `GeoDivisionCommandServiceImplTests`, `TaxIdTypeAuditTests`, `LocationRowsAreNeverDeletedTests` | ✅ |
| LOC-R08 | Los dos códigos ISO de un país nombran un mismo país de ISO 3166 | §4.2, D-47 | Base (solo la forma), Dominio, Servicio, Frontend | `IsoCountryCodesTests`, `CountryTest`, `CountryCommandServiceImplTest`, `CountriesControllerTests` | ✅ |
| LOC-R09 | Dos países no comparten código ISO de dos ni de tres letras | §4.2 | Base, Servicio | `CountryCommandServiceImplTest`, `CountriesControllerTests`, `LocationRacedDuplicatesTests` | ✅ |
| LOC-R10 | Los códigos ISO se corrigen solo mientras el país no tenga escalones, direcciones, tipos de documento ni tasas de cambio (D-155) | §4.2, §4.8, D-47, D-155 | Dominio, Servicio, Frontend | `CountryIsoCodesPolicyTests`, `CountryCommandServiceImplTest`, `CountriesControllerTests`, `ExchangeRatesIntegrationTests`, `e2e/country-profile.spec.ts` | ✅ |
| LOC-R11 | El código telefónico es `+` seguido de 1 a 5 dígitos | §4.2 | Base, Dominio, Servicio, Frontend | `CountryTest` | ✅ |
| LOC-R12 | La zona horaria del país es el nombre IANA de una región, de la forma Área/Lugar: no un desfase fijo (`Etc/GMT+5`, `UTC`) ni un alias sin continente (`US/Eastern`) | §4.2, §2.6, D-40 | Dominio, Servicio, Frontend | `RegionTimezoneTests`, `CountryTest`, `CountriesControllerTests` | ✅ |
| LOC-R13 | La tasa del impuesto va de 0 a 1, con hasta cuatro decimales | §4.2 | Base, Dominio, Servicio, Frontend | `CountryTest`, `CountriesControllerTests` | ✅ |
| LOC-R14 | Un país declara de 1 a 4 escalones | §4.4 | Base, Dominio, Servicio | `CountryTest`, `GeoDivisionLevelAuditTests`, `GeoDivisionLevelsControllerTests` | ✅ |
| LOC-R15 | Los escalones se declaran en orden, sin saltos: el siguiente es siempre el que sigue al último | §4.4, D-45 | Dominio, Servicio | `CountryTest`, `GeoDivisionLevelsControllerTests`, `CountryLockTests` | ✅ |
| LOC-R16 | Un escalón no se borra ni se retira; solo se renombra | §4.4, D-45 | Dominio, Servicio, Frontend | `GeoDivisionLevelsControllerTests` | ✅ |
| LOC-R17 | Una división va en un escalón que su país declaró | §4.5 | Base, Servicio | `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests` | ✅ |
| LOC-R18 | El código oficial de una división no se repite en el país; se guarda en mayúsculas y sin guiones, espacios ni puntos, así `t-2`, `T 2` y `T2` son el mismo código | §4.5, §2.2 | Base, Dominio, Servicio, Frontend | `GeoDivisionCodeTests`, `TextIdentifierTest`, `TextIdentifiersMigrationTests`, `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests`, `LocationRacedDuplicatesTests` | ✅ |
| LOC-R19 | El código, el escalón, la división superior y el país de una división no cambian | §2.4, §4.5 | Dominio, Servicio | No hace falta: la API no recibe esos campos al editar | ✅ |
| LOC-R20 | Solo las divisiones del escalón 1 no tienen división superior | §4.5 | Base, Dominio, Servicio | `GeoDivisionTests`, `GeoDivisionCommandServiceImplTests` | ✅ |
| LOC-R21 | La división superior es del mismo país | §4.5 | Base, Servicio | `GeoDivisionCommandServiceImplTests`, `SameCountryReferencesMigrationTests` | ✅ |
| LOC-R22 | La división superior es del escalón inmediato superior: un distrito no cuelga directo del departamento | §4.5, D-54 | Base, Servicio, Frontend | `GeoDivisionParentLevelMigrationTests`, `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests`, `HierarchyTierTests` | ✅ |
| LOC-R23 | Una división retirada se desactiva y sigue existiendo | §4.5 | Dominio | `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests` | ✅ |
| LOC-R24 | Toda dirección tiene calle | §4.6 | Base, Dominio, Frontend | `AddressTests`, `address-fields.spec.ts` | ✅ |
| LOC-R25 | La división de una dirección es del mismo país que la dirección | §4.6 | Base, Dominio, Servicio | `AddressDivisionPolicyTests`, `SameCountryReferencesMigrationTests` | ✅ |
| LOC-R26 | La división de una dirección es del escalón más bajo del país, y está activa si se elige ahora; la que la dirección ya tenía se conserva aunque se haya desactivado | §4.6 | Dominio, Servicio, Frontend | `AddressDivisionPolicyTests`, `AddressCommandServiceImplTests`, `address-fields.spec.ts`, `address-options.store.spec.ts`, `e2e/companies.spec.ts` | ✅ |
| LOC-R27 | El código postal es obligatorio en los países que lo exigen | §4.2, §4.6 | Dominio, Servicio, Frontend | `AddressTests`, `AddressCommandServiceImplTests`, `address-fields.spec.ts` | ✅ |
| LOC-R28 | La dirección está en el país de su dueño; quien la escribe no elige el país | §4.6 | Base (empresas y sedes), Servicio | `LocationContextFacadeImplTest`, `BranchesIntegrationTests`, `SameCountryReferencesMigrationTests`, `CompanyFiscalAddressConstraintTests`, `LocationCommandServiceImplTests` | ✅ |
| LOC-R29 | Una dirección no se comparte: cada dueño tiene su propia fila | §4.6 | Base (dentro de cada tabla dueña), Servicio | `SameCountryReferencesMigrationTests`, `BranchConstraintTests`, `OrganizationOwnershipKeysMigrationTests` | ✅ |
| LOC-R30 | Corregir modifica la misma fila; mudarse crea una fila nueva y conserva la anterior | §4.6, D-8, D-79 | Servicio, Frontend | `LocationContextFacadeImplTest`, `BranchesIntegrationTests`, `LocationsIntegrationTests`, `location-form-dialog.spec.ts`, `e2e/locations.spec.ts` | ✅ |
| LOC-R31 | La sigla de un tipo de documento no se repite en el país | §4.7 | Base, Servicio | `TaxIdTypeCommandServiceImplTests`, `TaxIdTypesControllerTests` | ✅ |
| LOC-R32 | La sigla y el país de un tipo de documento no se editan | §4.7 | Dominio, Frontend | `TaxIdTypeTests`, `TaxIdTypesControllerTests` | ✅ |
| LOC-R33 | Un tipo aplica a empresa, persona o ambos, y solo se usa para lo que aplica: una empresa no se registra con un DNI | §4.7 | Base, Dominio; el uso, Servicio de `organization` | `TaxIdApplicabilityTests`, `TaxIdTypeRulesTests` | ✅ |
| LOC-R34 | Cuando el tipo declara una longitud, va de 1 a 20 —lo que caben las columnas que guardan los números— y el número, ya normalizado, tiene exactamente esa longitud | §4.7, D-78 | Base, Dominio, Servicio, Frontend | `TaxIdTypeTests`, `TaxIdTypeLengthMigrationTests`, `TaxIdTypesControllerTests`, `TaxIdTypeRulesTests` | ✅ |
| LOC-R35 | Cuando el tipo tiene dígito de control, el número lo cumple. Se verifica al crear el registro y cuando cambian el número o el tipo; un número ya guardado sigue siendo válido | §4.7, D-75 | Dominio; el uso, Servicio de `organization`; Frontend | `TaxIdCheckDigitPolicyTests`, `LocationContextFacadeImplTest`, `PeruTaxIdTypesMigrationTests`, `TaxIdTypeRulesTests`, `CompanyCommandServiceImplTests`, `EmployeeCommandServiceImplTests`, `tax-id-check-digit.spec.ts` | ✅ |
| LOC-R36 | Un tipo de documento inactivo no se elige en registros nuevos; quien ya lo usa sigue válido | §4.7 | Dominio; el uso, Servicio de `organization`; Frontend | `TaxIdTypeRulesTests`, `TaxIdTypesControllerTests`, `EmployeeCommandServiceImplTests`, `employee-form-dialog.spec.ts` | ✅ |
| LOC-R37 | Quien usa un tipo de documento es de su mismo país | §4.7 | Base (empresas), Servicio de `organization` | `TaxIdTypeRulesTests`, `SameCountryReferencesMigrationTests` | ✅ |
| LOC-R38 | Los catálogos los lee quien tiene `catalogos.ver` por un rol vigente, y los crea, corrige o desactiva quien tiene el permiso de esa operación (D-130); la semilla da la lectura a los siete roles del sistema y las escrituras solo al superadministrador (D-43). Las tasas de cambio se registran con `catalogos.registrar_tipo_cambio` y se corrigen con `catalogos.corregir_tipo_cambio` (D-149); como toda escritura de un catálogo general, esos permisos solo se marcan en roles de alcance sistema (D-150; [iam.md](iam.md), IAM-R80). Un rol propio sin el permiso recibe `403`, también con sesión y rol, y un permiso desmarcado del rol deja de valer en la petición siguiente. Las pantallas también piden la lectura: se abren solo con `catalogos.ver`, y el alta de un país además con `catalogos.crear_paises` | §5.6, D-43, D-95, D-130, D-141, D-149, D-150 | Servicio, Frontend | `LocationReadAccessTests`, `LocationPermissionIntegrationTests`, los `*ControllerTests`, `ExchangeRatesIntegrationTests`, `ExchangeRateCorrectionsIntegrationTests`, `location.routes.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R39 | Un archivo exportado lleva códigos y nombres, nunca el número interno ni el `public_id` | §2.4, D-51 | Servicio | `CurrenciesControllerTests`, `CountriesControllerTests`, `GeoDivisionsControllerTests`, `TaxIdTypesControllerTests`, `ExchangeRatesIntegrationTests`, `e2e/exports.spec.ts` | ✅ |
| LOC-R40 | Las seis tablas del paso 1 registran cuándo se creó y cambió cada fila, sin autor. `exchange_rates` registra además quién registró cada tasa y quién la corrigió por última vez, que su pantalla muestra (§4.8, D-149) | D-42, §4.8 | Base, Servicio | Los seis `*AuditTests`, `ExchangeRateAuditTests`, `ExchangeRatesIntegrationTests` | ✅ |
| LOC-R41 | Una dirección tiene su división cuando su país tiene divisiones cargadas; solo puede faltar en un país sin ellas | §4.6, D-53 | Servicio, Frontend | `AddressCommandServiceImplTests`, `CompaniesControllerTests`, `address-fields.spec.ts`, `address-options.store.spec.ts`, `e2e/companies.spec.ts`, `e2e/locations.spec.ts` | ✅ |
| LOC-R42 | El país, las dos monedas, el tipo y el día de una tasa no se repiten: una tasa de compra y una de venta del mismo día son dos tasas | §4.8, D-148 | Base, Dominio, Servicio, Frontend | `ExchangeRateTests`, `ExchangeRatesIntegrationTests`, `ExchangeRateRacedDuplicateTests`, `register-exchange-rate-dialog.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R43 | Una tasa convierte una moneda en otra distinta, es mayor que cero, y su tipo y su fuente son de listas cerradas: `compra`, `venta` u `oficial`; `SBS`, `BCRP`, `BANREP` o `BANXICO`. No hay fuente `manual`: quién registró la tasa lo dice su autor (LOC-R40) | §4.8, D-148 | Base, Dominio, Servicio, Frontend | `ExchangeRateTests`, `ExchangeRatesIntegrationTests`, `ExchangeRateCorrectionsIntegrationTests`, `register-exchange-rate-dialog.spec.ts` | ✅ |
| LOC-R44 | Una tasa tiene a lo sumo seis decimales y es menor que 1 000 000, lo que guarda su columna, la misma precisión que la tasa de facturas, notas y pagos. No es un monto: no lleva moneda propia, y `3.750000` y `3.75` son la misma tasa | §4.8, D-146 | Base, Dominio, Servicio, Frontend | `ExchangeRateTests`, `ExchangeRatesIntegrationTests`, `register-exchange-rate-dialog.spec.ts`, `exchange-rates-view.spec.ts`, `change-logs-view.spec.ts` | ✅ |
| LOC-R45 | Una tasa la registra a mano, cuando un documento la necesita, quien tiene `catalogos.registrar_tipo_cambio`; no hay que cargar una cada día. Su autor es la cuenta de la sesión | §4.8, D-42, D-148, D-149 | Servicio, Frontend | `ExchangeRatesIntegrationTests`, `PermissionsMigrationTests`, `exchange-rates-view.spec.ts`, `register-exchange-rate-dialog.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R46 | Una tasa que se aparta de la anterior de su serie —el mismo país, par y tipo— más que el umbral pide confirmación antes de guardarse, al registrarla y al corregirla. La anterior es la más reciente de un día anterior al suyo, así que una tasa nunca se compara consigo misma, y sin anterior no hay nada que confirmar. El umbral es configuración, 3 % por omisión, y un cambio de exactamente el umbral no pregunta | §4.8, D-148 | Dominio, Servicio, Frontend | `ExchangeRateTests`, `ExchangeRatesIntegrationTests`, `ExchangeRateCorrectionsIntegrationTests`, `large-rate-change-question.spec.ts`, `exchange-rates.store.spec.ts`, `register-exchange-rate-dialog.spec.ts`, `correct-exchange-rate-dialog.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R47 | Corregir una tasa es un cambio delicado: lo hace quien tiene `catalogos.corregir_tipo_cambio`, con un motivo de 1 a 500 caracteres, y cambia solo la tasa y su fuente, nunca el país, el par, el tipo ni el día. Cada campo que cambia deja su fila en la bitácora, con el valor anterior, el nuevo y el motivo, en la misma transacción; sin sus filas la corrección no se guarda. Una corrección sobre un guardado posterior al que se abrió se rechaza, y una que no cambia nada no escribe nada. La lista muestra la última corrección de cada tasa: cuándo y por quién | §4.8, §13.1, D-149 | Dominio, Servicio, Frontend | `ExchangeRateTests`, `ExchangeRateCorrectionsIntegrationTests`, `ExchangeRateAuditTests`, `ExchangeRateLockTests`, `ExchangeRateQueryServiceImplTests`, `correct-exchange-rate-dialog.spec.ts`, `exchange-rates.store.spec.ts`, `change-logs-view.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R48 | La fecha de una tasa no pasa de mañana en la zona horaria de su país: la tasa de un día se conoce desde la tarde anterior, y un año mal escrito, como 2062, no queda registrado. La corrección no cambia la fecha | §4.8, D-156 | Dominio, Servicio, Frontend | `ExchangeRateTests`, `ExchangeRatesIntegrationTests`, `register-exchange-rate-dialog.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R49 | Una tasa no se borra ni se desactiva: no tiene estado, y lo que se escribió mal se corrige en su fila, con su motivo (LOC-R47) | §4.8, D-149 | Base, Servicio, Frontend | `LocationRowsAreNeverDeletedTests`, `NoDeleteEndpointsTests` | ✅ |
| LOC-R50 | Las tasas son un catálogo general: no llevan empresa ni Row Level Security, cada tasa sirve a todas las empresas de su país, y la lee cualquiera con `catalogos.ver`, con cualquier alcance. Sus filas de la bitácora no llevan empresa, así que solo las ve quien tiene `bitacora.ver` sobre todas las empresas | §2.5, §4.8, §13.1, D-148, D-149 | Base, Servicio, Frontend | `ExchangeRatesIntegrationTests`, `ExchangeRateCorrectionsIntegrationTests` | ✅ |
| LOC-R51 | Las escrituras de una misma serie van de a una: un registro de un día y la corrección del día anterior, o dos registros de días seguidos, no se cruzan, así que ninguno se salta la confirmación comparando con una tasa que otro está cambiando. Un registro lee sus dos monedas en modo compartido, así que una moneda que se desactiva a la vez espera o se ve | §4.8, D-148 | Servicio | `ExchangeRateLockTests` | ✅ |
| LOC-R52 | La exportación de las tasas lleva el país, las dos monedas, la fecha, el tipo, la tasa y la fuente, con sus códigos y nunca el número interno ni el `public_id`, y nunca quién corrigió una tasa: el archivo lo comparten todos los que tienen `catalogos.ver`, y el nombre de quien corrigió se resuelve entre los colaboradores que ve cada uno | §2.4, §4.8, D-51, D-106 | Servicio, Frontend | `ExchangeRatesIntegrationTests`, `ExchangeRateQueryServiceImplTests`, `exchange-rates-api-endpoint.spec.ts`, `exchange-rates-view.spec.ts`, `e2e/exchange-rates.spec.ts` | ✅ |
| LOC-R53 | Cada país dice qué tipo de tasa propone para sus compras: Perú, `venta`; los demás, también uno nuevo, `oficial`. La API no lo edita | D-152, D-181 | Base, Dominio, Servicio | `CountriesControllerTests` | ✅ |
| LOC-R54 | La fachada da la tasa vigente de un día: la más reciente de ese día o anterior | D-151 | Servicio | `LocationContextFacadeImplTest` | ✅ |

---

## 5. Relación con otros contextos

`location` no depende de ningún contexto de negocio: sus cinco exportaciones —monedas, países, divisiones, tipos de documento y tasas— las pide al módulo `documents` (D-125), la corrección de una tasa la anota en la bitácora del módulo `audit` (D-262), y la versión de sus datos sale de los contadores de `data_versions` (D-128). Los demás lo usan por su fachada, nunca por sus tablas (§3).

**La fachada `LocationContextFacade`** (detalle en mapa §5.1). La usan `organization`, `procurement` y `fleet`.

| Miembro | Para qué | Quién lo usa |
|---|---|---|
| `fetchCurrencySummaryByIsoCode` | Una moneda por su código ISO (D-13), con si está activa | `organization` (la moneda funcional), `procurement` (filtros y la moneda de una nota de crédito) |
| `fetchCurrencySummaryByIsoCodeForShare` | La moneda de un registro que escribe un monto en ella, leída en modo compartido hasta el fin de la transacción de quien llama: su estado y sus decimales no cambian entre la revisión y la escritura (LOC-R05, LOC-R06) | `procurement` (órdenes, facturas, pagos, carga inicial), `fleet` (el costo de un equipo), `organization` (una cuenta bancaria) |
| `fetchCurrencySummariesByIds` | Monedas en lote, con código, símbolo, decimales y estado | `organization`, `procurement`, `fleet` |
| `fetchEffectiveRate` | La tasa vigente de un país, un par y un tipo en un día, con su día (LOC-R54) | `procurement`: facturas y pagos |
| `fetchCountrySummaryByPublicId`, `fetchCountrySummaryByIsoAlpha2` | Un país por su identificador público o por su código de dos letras (D-13) | `organization` (el alta y el filtro de empresas), `fleet` (el país de un tipo de documento del vehículo) |
| `fetchCountrySummariesByIds` | Países en lote, con su zona horaria, la tasa de su impuesto y el tipo de tasa de sus compras (LOC-R53) | `organization` (también el «hoy» de un colaborador sin sede y la zona de cada empresa en `/me`), `procurement`, `fleet` |
| `fetchTaxIdTypeSummaryByPublicId`, `fetchTaxIdTypeSummariesByIds` | El tipo de documento de un registro, con sus reglas (LOC-R33 a LOC-R37), y los de una lista | `organization` (empresas, colaboradores), `procurement` (proveedores, plantillas) |
| `verifyTaxIdCheckDigit` | El dígito de control de un número: válido, inválido, no aplica o no verificado (LOC-R35) | `organization`, `procurement` |
| `createAddress`, `correctAddress`, `replaceAddress`, `fetchAddressSummariesByIds` | Crear, corregir y mudar la dirección del dueño, en su transacción, y mostrarlas en lote (LOC-R28, LOC-R30) | `organization` (empresas, sedes, ubicaciones), `procurement` (proveedores) |
| `copyAddress` | Copiar tal cual una dirección a una fila nueva de otro dueño, sin revisar otra vez las reglas de una dirección nueva, porque nada se elige (D-134) | `procurement`: el emisor de una plantilla de orden de compra |
| `fetchDataVersion` | La versión de los datos de `location` que muestra una exportación de quien llama (D-128) | `organization`, `procurement`, `fleet` |

La división por su identificador público, que mapa §5.1 deja como propuesta, no la ofrece la fachada: ningún contexto la pide.

**El puerto `CurrencyUsage`** (LOC-R05). `location` pregunta a cada contexto que guarda montos si alguno de sus registros usa una moneda, en cualquier empresa, y lo recibe sin depender de él: la dependencia va de quien implementa hacia `location` (mapa §6.6). Lo implementan `procurement`, con sus órdenes, sus facturas y sus pagos, de cualquier estado, también anulados, y `fleet`, con el costo de una instalación. La nota de crédito no lo necesita: descuenta solo facturas de su moneda (`CREDITNOTE_INVOICE_MISMATCH`), que ya la fijan. Un contexto nuevo que guarde montos lo implementa con su paso (mapa §4).

**La tasa de un documento** (D-151, D-152). La factura de compra y el pago a proveedor proponen la tasa del país de su empresa, del tipo que ese país usa para sus compras (LOC-R53), la más reciente de la fecha del documento o anterior (LOC-R54), y la copian; la nota de crédito copia la de la factura que descuenta. Si en un día hábil falta la tasa de la fecha del documento, el formulario lo avisa y abre ahí el registro de la tasa, con las reglas de F7.

**La bitácora.** La corrección de una tasa escribe en la bitácora de `audit` (`audit::acl`, D-262), y la lista nombra a quien corrigió por el puerto que implementa `iam`, entre los colaboradores que ve quien lee, sin que `location` dependa de `iam` ni de `organization`.

**Frontend.** `LocationFacade` ofrece a los demás contextos opciones mínimas de países, monedas y tipos de documento, la tasa propuesta de un documento y la regla del dígito de control del backend, para avisar antes de enviar (LOC-R35). Presta una sola pieza de pantalla, el campo de dirección, que produce su valor; guardarla le toca al dueño (empresa, sede, ubicación y proveedor).

**Sin uso.** `vehiclecatalog` puede usar `location` y no lo necesita. Los fondos de `fleet` no tienen moneda (§12.1).

---

## 6. Estado y pendientes

`location` está construido para Perú, el único país que opera (D-5). Las tasas de cambio se cargan a mano. La única decisión abierta con disparo propio son las longitudes y el dígito de control de los documentos de Colombia y México (§4.7, D-52), con su momento abajo.

| Qué | Cuándo |
|---|---|
| La carga automática de las tasas desde la fuente oficial, a nombre de `SYSTEM` (D-46, D-148) | Cuando haya una fuente con acceso permitido para un sistema: la API del BCRP responde a una consulta automatizada con una página contra robots |
| La moneda de grupo (D-46, §4.8). Los reportes no la usan: un reporte es de una empresa y se lee en su moneda funcional (D-233; [reporting.md](reporting.md)) | Cuando haya dos monedas funcionales que reunir en un mismo reporte |
| Colombia y México: las longitudes de sus documentos y el cálculo del dígito de control del NIT, el RFC y la CURP (D-75), y sus listas oficiales (DANE, INEGI). **Riesgo aceptado:** hasta entonces nada en el código impide registrar una empresa de esos países, y sus tipos con dígito de control aceptan cualquier número. No hay que crear a mano divisiones de Colombia ni de México: con una sola división cargada, toda dirección de ese país pasa a exigir la suya (D-53) | Antes de registrar la primera empresa de Colombia o México (D-52) |
| Dar de alta Santa Rosa de Loreto (Ley 32403) con su propia migración | Cuando el INEI o SUNAT publiquen su código (D-77) |
| Revisar la lista de divisiones de Perú contra las leyes nuevas y las ediciones nuevas del INEI, como la que puede salir del Censo 2025 | En el cierre de cada paso (D-77); la última revisión es del 17/10/2026 |
| El alta manual de una división acepta cualquier forma de código, que después queda fija: solo se limpia y se pasa a mayúsculas, sin comparar con la forma del código oficial del país | Si el alta manual vuelve a usarse para un país con lista oficial |
| Una prueba de que una tasa registrada a la vez espera a la corrección de los códigos ISO de su país (LOC-R10) | Sin fecha |

---

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿El dígito de control del NIT se escribe aparte o forma parte del número? ¿Cómo se modela el RFC de 12 y de 13? | Diccionario §4.7; se resuelve antes de registrar la primera empresa de cada país (D-52) |
| ¿Se puede volver a cargar la lista oficial de divisiones desde la aplicación? | [Procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), apartado 5, pregunta 7 |
| ¿`taxRate` y la `rate` de un tipo de cambio viajan como texto decimal? | [Convenciones](../convenciones-de-la-api-millennium-mobility.md) §14.2 (§13 fila 7) |
| ¿Un `422` dice qué regla falló, en un campo `rule`? | Convenciones §14.5 (§13 fila 15) |
| ¿Dos ediciones simultáneas del mismo registro se detectan con `If-Match`? | Convenciones §14.7 |
| ¿Las altas, también la de una tasa, responden `201` con `Location`? | Convenciones §14.9 (§13 fila 8) |
| ¿Los códigos de error llevan guion bajo entre palabras (`GEO_DIVISION_*`)? | Convenciones §14.11 (§13 fila 13) |
| ¿Cómo obtiene el registro público, sin sesión, los tipos de documento de las personas? | Convenciones §14.16; antes de encender el registro (D-83) |
