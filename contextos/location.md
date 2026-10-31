# Contexto `location` — ERP Millennium Mobility

**Actualizado:** 30/10/2026 (diccionario v1.33 y `develop` del 30/10/2026, backend `a54ee29b` y frontend `88484481`, sin publicar todavía: Monedas, Países y la ficha del país se abren solo con `catalogos.ver`, y `countries/new` además con `catalogos.crear_paises`, D-141, que resuelve el hallazgo de quien solo escribe; la búsqueda de monedas, países, divisiones y documentos tributarios no distingue tildes, D-142, V86; la prueba del store de monedas dice que solo una orden de compra fija los decimales, `ec7bd7cc`; la decisión del 29/10/2026 sobre los decimales citada como D-138) · 29/10/2026 (los decimales de una moneda quedan fijos desde el primer monto que la usa, por decisión del responsable del 29/10/2026: `422 CURRENCY_DECIMAL_PLACES_IN_USE`, con el puerto `CurrencyUsage` de `location.interfaces.acl`, que implementa `procurement` con sus órdenes de compra, backend `7b8db0ee`, frontend `f9a23d0b` y `da457b00`; LOC-R05 se aplica; la fachada, con sus dos consumidores, la moneda por su código ISO y leída en modo compartido, y el símbolo en `CurrencySummary` desde el 27/10/2026, `42ecdfa2`; los permisos partidos de D-130 en los flujos, en LOC-R38 y en la sección 5; el grupo Catálogos, por permiso) · 26/10/2026 (cierre de la auditoría del 25/10/2026, diccionario v1.31: `addresses` bajo Row Level Security por el país de la dirección, con rama `SYSTEM`, V78, D-80 y D-134; las preguntas «tiene direcciones» en `SystemAccessScope`; `copyAddress` en la fachada, la copia fiel del domicilio del emisor de una plantilla) · 19/10/2026 (cada escalón de la dirección se elige escribiendo, con `mm-search-select` (DESIGN.md, revisión 21), y la zona horaria del país también; el padre de una división se busca en el servidor mientras se escribe, y ya se alcanzan Puno, San Martín, Tacna, Tumbes y Ucayali; merges `60cdb93` (placeholders), `bced274` (padre) y `dec9ddd` (buscador) en el frontend) · 19/10/2026 (paso 3b, diccionario v1.23: los catálogos se leen con `catalogos.ver` y se escriben con `catalogos.gestionar_generales`, D-43, D-95; LOC-R38; integrado en `develop` el 19/10/2026) · 18/10/2026 (integrado en `develop` dentro de `feature/organization-closure`; el inventario de V39 sale de los pendientes) · 18/10/2026 (validación: las correcciones de la auditoría de `organization` a este documento; LOC-R28 y LOC-R37 con su prueba, LOC-R36 también en la pantalla, el inventario de V39 antes de integrar en `develop`) · 18/10/2026 (cierre del paso 1: dígito de control, edición del ubigeo, longitud de 1 a 20 y direcciones de las ubicaciones, D-75 a D-80; V36 a V39) · 16/10/2026 (LOC-R29: V26 hace única la dirección de una ubicación) · 15/10/2026 (paso 1 construido para Perú) · **Estado:** Validado por el responsable el 18/10/2026 · **Fuente:** diccionario de datos v1.23 (§2, §3, §4.1 a §4.8, §5.6 y las decisiones D-2, D-3, D-5, D-6, D-8, D-13, D-40, D-42 a D-49, D-51 a D-54, D-75 a D-80, D-86 y D-95), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), [orden de implementación](../orden-de-implementacion-millennium-mobility.md), las revisiones del [30/09/2026](../retroalimentacion/2026-09-30-location.md), del [05/10/2026](../retroalimentacion/2026-10-05-location.md), del [15/10/2026](../retroalimentacion/2026-10-15-location.md) y del [18/10/2026](../retroalimentacion/2026-10-18-location.md), la auditoría de `organization` del 18/10/2026 ([revisión](../retroalimentacion/2026-10-18-organization.md)), el código de la rama `feature/location-closure` del backend y del frontend al 18/10/2026, con lo que `feature/organization-closure` agregó para `location`, y el de `feature/permissions-matrix` al 19/10/2026 (backend `9f6c1d8`; [revisión](../retroalimentacion/2026-10-19-permisos.md)); desde el 29/10/2026, la decisión del responsable de ese día sobre los decimales y el código de `develop` al 29/10/2026 (backend `5ec83065`, frontend `bda96499`). Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** Las secciones 1 a 3 están escritas para quien conoce el negocio. La 4 y la 6 nombran archivos del código para el equipo. Lo marcado **(propuesta)** no lo decide nadie todavía.

---

## 1. Qué es y qué no es

`location` guarda los datos que comparten todas las empresas y la localización de cada país: con qué monedas se opera, qué países existen, cómo se divide su territorio, con qué documento se identifican ante su Estado las empresas y las personas, y las direcciones físicas (D-44). Es el paso 1 del orden de implementación y no depende de ningún otro contexto.

**Las seis tablas que posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `currencies` | Las monedas: PEN, USD, COP, MXN | §4.1 |
| `countries` | Los países, con su moneda sugerida, su zona horaria y su impuesto general | §4.2 |
| `geo_division_levels` | Los escalones de cada país: Departamento, Provincia, Distrito | §4.4 |
| `geo_divisions` | Las divisiones concretas, con su código oficial (el ubigeo en Perú) | §4.5 |
| `addresses` | Las direcciones de empresas, sedes, ubicaciones y, más adelante, proveedores | §4.6 |
| `tax_id_types` | Los tipos de documento tributario de cada país: RUC, DNI, NIT, RFC | §4.7 |

Cómo se relacionan las tres tablas de geografía está en §4.3. Las columnas están en el diccionario y no se repiten aquí.

**Lo que parece suyo y no lo es**

| Parece de `location` | Es de | Por qué |
|---|---|---|
| La tabla `locations` (patio, sede, dealer) | `organization` (§6.5, D-6) | "Ubicación" ahí es el lugar donde puede haber vehículos, no un punto del mapa (mapa §1) |
| Si un país opera | `organization` | Se deduce de si tiene una empresa activa; no se guarda en el país (D-3) |
| Los tipos de cambio por día y la moneda de grupo | Nadie todavía | Están diseñados en §4.8 y se construyen justo antes de compras (paso 7) y con los reportes (paso 11) (D-46) |
| La moneda funcional de una empresa | `organization` | La empresa apunta a una moneda de `location`, pero elegirla es una decisión de la empresa (§6.2) |
| El número de RUC de una empresa o el DNI de un colaborador | `organization` | `location` define el tipo de documento; el número lo guarda quien se identifica con él |
| La zona horaria de una sede | `organization` | El país solo propone una zona por defecto; la hora real es la de cada sede (§4.2) |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Escalón | Cada nivel en que un país divide su territorio: Departamento (1), Provincia (2), Distrito (3). Es la palabra del diccionario y de las pantallas | "Nivel", que es solo el nombre de la columna (`level`). En pantalla no se usa |
| División | Un pedazo concreto del territorio: Lima, Chorrillos. Cuelga de una división del escalón inmediato superior | El escalón, que es la categoría |
| Escalón más bajo | El último escalón que declaró el país: el distrito en Perú, el municipio en Colombia. Una dirección apunta a una división de ese escalón | — |
| Corregir una dirección | Se escribió mal: se modifica la misma fila y todo lo que apunta a ella sigue apuntando (D-8) | Mudar |
| Mudar una dirección | El dueño se fue a otro lugar: se crea una fila nueva y la anterior se conserva para los documentos que la citan (D-8) | Corregir |
| Tipo de documento tributario | El documento con que una empresa o una persona se identifica ante el Estado: RUC, DNI, NIT, RFC | "Documento de identidad", que es el dato del colaborador |
| Moneda sugerida | La que el país propone al registrar algo en él. Es solo una sugerencia | Moneda funcional, la de los libros de una empresa |

---

## 2. Ciclo de vida de cada agregado

Ninguna fila de `location` se borra. Lo que deja de usarse se desactiva, salvo los países y los escalones, que no tienen estado. Todas las tablas registran cuándo se creó y cuándo cambió cada fila, pero no quién: son catálogos generales (D-42).

### Moneda

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador la crea en Catálogos › Monedas (D-43). Llegan sembradas PEN, USD, COP y MXN |
| Qué cambia | El nombre, el símbolo y, mientras ningún monto use la moneda, los decimales |
| Qué queda fijo | El código ISO, desde el alta: es de una norma externa y no se edita nunca (§2.4, D-48). Los decimales, desde el primer monto que use la moneda (§4.1): hoy, desde la primera orden de compra en esa moneda, de cualquier empresa y en cualquier estado, también anulada o cumplida, porque sus montos y su PDF enviado se conservan. Los fondos no cuentan: no tienen moneda (§12.1). Se aplica desde el 29/10/2026, por decisión del responsable (LOC-R05); hasta entonces se podían cambiar siempre |
| Retiro | Se desactiva. Una moneda inactiva no se elige en operaciones nuevas, pero el país o la empresa que ya la tenía la conserva (§4.1, LOC-R06) |
| Nunca | Se borra |

### País y su estructura territorial

El país incluye sus escalones: se guardan y se validan con él (D-45).

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador lo registra en su propia ficha, `/location/countries/new`. Llegan sembrados Perú, Colombia y México, con sus escalones |
| Qué cambia | El nombre, el código telefónico, la moneda sugerida, la zona horaria, el impuesto y su tasa, y si exige código postal. Los escalones se declaran uno por uno, siempre debajo del último, y se renombran |
| Qué queda fijo | Los dos códigos ISO, desde que algo cuelga del país: un escalón, una dirección o un tipo de documento. Antes se pueden corregir, por ejemplo un error de tipeo (D-47). El número de cada escalón, desde que se declara |
| Retiro | No tiene. Un país deja de operar cuando se desactivan sus empresas (D-3) |
| Nunca | Se borra un país; se borra o se retira un escalón (D-45); se guarda si el país opera |

### División territorial

| Momento | Qué pasa |
|---|---|
| Alta | La migración V24 carga la lista oficial de Perú (D-2): el ubigeo del INEI, edición 2022, con 25 departamentos, 196 provincias y 1 891 distritos. Cada distrito que una ley crea después entra activo con su propia migración en cuanto el INEI o SUNAT publican su código, y el distrito del que se separó sigue activo (D-77). El primero es Santa Rosa de Loreto (Ley 32403, 03/07/2025), que todavía no tiene código publicado: hasta entonces no se puede elegir. Las de Colombia y México se cargan antes de registrar su primera empresa. El superadministrador también crea divisiones una por una en la ficha del país, pestaña Estructura territorial |
| Qué cambia | Solo el nombre |
| Qué queda fijo | El código oficial, el escalón, la división superior y el país, desde el alta. El código se guarda en mayúsculas y sin guiones, espacios ni puntos (§2.2) |
| Retiro | Se desactiva cuando se fusiona o desaparece. Las direcciones ya registradas siguen apuntando a ella |
| Nunca | Se borra |

### Tipo de documento tributario

| Momento | Qué pasa |
|---|---|
| Alta | El superadministrador lo crea en la ficha del país, pestaña Documentos. Llegan sembrados RUC, DNI, CE y PAS en Perú; NIT y CC en Colombia; RFC y CURP en México |
| Qué cambia | El nombre, a quién aplica (empresa, persona o ambos), la longitud, de 1 a 20 (D-78), y si tiene dígito de control |
| Qué queda fijo | La sigla y el país, desde el alta |
| Retiro | Se desactiva. Quien ya lo usa sigue siendo válido; un registro nuevo no puede elegirlo |
| Nunca | Se borra |

### Dirección

La dirección no tiene pantalla, menú ni ruta propia. Vive dentro de su dueño y se escribe con el permiso del dueño (mapa §6.3, convenciones §3.3).

| Momento | Qué pasa |
|---|---|
| Alta | La crea el dueño al registrarse o al poner su primera dirección: hoy empresas, sedes y ubicaciones de terceros; después proveedores y plantillas de orden. Una ubicación propia usa la dirección de su sede (D-79) |
| Qué cambia | En una corrección, la calle, el número, el interior, la referencia, el código postal y la división, sobre la misma fila (D-8) |
| Qué queda fijo | El país: es el del dueño, y quien escribe la dirección no lo elige (§4.6) |
| Retiro | No tiene. En una mudanza el dueño pasa a apuntar a una fila nueva y la anterior queda para los documentos que la citan |
| Nunca | Se comparte entre dos dueños, aunque el texto coincida; se borra |

---

## 3. Flujos funcionales

Los flujos de escritura exigen el permiso de la operación —`catalogos.crear_monedas`, `editar_monedas`, `desactivar_monedas`, `crear_paises`, `editar_paises`, que también declara y corrige los escalones y crea y corrige las divisiones y los documentos tributarios, o `desactivar_paises`, que activa y desactiva las divisiones y los documentos tributarios—, que la semilla da solo al superadministrador (D-130, V62), salvo F5, que sigue el permiso del dueño; las lecturas exigen `catalogos.ver`, que tienen los siete roles del sistema, porque toda pantalla con un selector los necesita (LOC-R38). `catalogos.gestionar_generales` ya no abre ninguna ruta. Hasta el paso 3b las lecturas las hacía cualquier cuenta con un rol vigente. Desde el 30/10/2026, Monedas, Países y la ficha de un país se abren solo con `catalogos.ver`: escribir exige leer, y un permiso de escritura no abre la pantalla (D-141; procesos, apartado 3.1). La ficha de un país nuevo (`/location/countries/new`) pide `catalogos.ver` y además `catalogos.crear_paises`, porque lee sus opciones y abre la ficha que guardó. Con la semilla los ven los siete roles, y solo el superadministrador escribe. Desde la segunda ronda de la auditoría (20/10/2026) hasta el 30/10/2026 se abrían también con un permiso de escritura de su catálogo, y un rol propio que solo escribía veía la opción pero su lista le respondía `403` (sección 6 b); antes de esa ronda el grupo Catálogos solo lo veía el superadministrador.

### F1 · Mantener las monedas

**Rol:** superadministrador. **Pantalla:** Catálogos › Monedas (`/location/currencies`), construida.

1. Abre la lista, que busca por código, nombre o símbolo, sin distinguir mayúsculas ni tildes (D-142), y filtra por estado.
2. Crea una moneda con su código, nombre, símbolo y decimales. El sistema revisa que el código sea de la norma ISO 4217, que no exista ya y que los decimales estén entre 0 y 4.
3. Edita el nombre, el símbolo o los decimales. El código ya no se puede tocar. Los decimales se corrigen solo mientras ningún monto use la moneda: si una orden de compra ya la usa, la API responde `422 CURRENCY_DECIMAL_PLACES_IN_USE` y nada cambia; el nombre y el símbolo se corrigen siempre (LOC-R05).
4. Exporta la lista a Excel con la misma búsqueda.

**Errores que ve:** "Ya existe una moneda con ese código ISO"; "El código no corresponde a ninguna moneda de la norma ISO 4217"; "Los decimales no se pueden cambiar: una orden de compra ya usa esta moneda con los que tiene." (`CURRENCY_DECIMAL_PLACES_IN_USE`). El diálogo queda abierto con lo escrito.
**Resultado:** la moneda queda disponible en los selectores de países y empresas.

### F2 · Registrar un país y su estructura

**Rol:** superadministrador. **Pantalla:** Catálogos › Países y la ficha del país (`/location/countries/new` y `/location/countries/{id}`), construidas.

1. **Alta.** Escribe los dos códigos ISO, el nombre, el código telefónico, la moneda sugerida, la zona horaria, el impuesto y su tasa, y si exige código postal. El sistema revisa que los dos códigos nombren un mismo país de ISO 3166 (`PE` con `PER`, nunca con `PRI`), que ningún otro país los tenga, que el código telefónico sea `+` y de 1 a 5 dígitos, que la zona horaria sea el nombre IANA de una región (`America/Lima`; no un desfase fijo como `Etc/GMT+5` o `UTC` ni un alias sin continente como `US/Eastern`, §2.6) y que la tasa esté entre 0 y 100 %. Los códigos se aceptan en minúsculas y se guardan en mayúsculas. Al guardar, la ficha abre las pestañas de su estructura territorial y sus documentos tributarios.
2. **Escalones.** Declara el primero por su nombre (Departamento), después el segundo, hasta cuatro. El número lo pone el sistema: siempre el que sigue al último. No hay botón para borrar un escalón; solo se renombra.
3. **Divisiones.** Elige el escalón, busca la división superior por su nombre o su código, en el servidor y desde el 30/10/2026 sin distinguir tildes (solo se ofrecen las del escalón de arriba), y escribe el código oficial y el nombre. El código se guarda en mayúsculas y sin guiones, espacios ni puntos. El sistema revisa que el país declare ese escalón, que el código no se repita en el país, que una división del escalón 1 no tenga superior y que cualquier otra cuelgue de una del mismo país en el escalón inmediato superior.
4. **Documentos tributarios.** Crea cada tipo con su sigla, nombre, a quién aplica, longitud y dígito de control. El sistema revisa que la sigla no se repita en el país y que la longitud, si la declara, vaya de 1 a 20 (D-78). La pestaña filtra los documentos por estado y por a quién identifican.

**Errores que ve:** "Los códigos ISO no son de un mismo país de la norma ISO 3166"; "Otro país ya tiene ese código ISO de dos letras" (o de tres); "El país ya declara cuatro escalones"; "El país ya tiene una división con ese código"; "La división no encaja en la estructura: revise el escalón y la división superior"; "El país ya tiene un documento con esa sigla".
**Resultado:** el país queda listo para registrar empresas en él. Para que una dirección pueda elegir distrito, además hacen falta sus divisiones (F6).

### F3 · Corregir los códigos ISO de un país

**Rol:** superadministrador. **Pantalla:** ficha del país, sección Datos.

1. Mientras nada cuelga del país, los campos de los códigos dicen "Corregible mientras nada dependa del país".
2. Corrige los códigos. El sistema revisa otra vez la norma ISO 3166 y que otro país no los tenga.
3. Si el país ya tiene un escalón, una dirección o un tipo de documento, los campos aparecen bloqueados con "Ya no cambia: hay datos del país que dependen de él", y la API rechaza el cambio igual.

**Errores que ve:** "Los códigos ISO ya no se pueden corregir: el país tiene escalones, direcciones o documentos tributarios".
**Resultado:** los códigos corregidos. Dentro del sistema nada se rompe, porque las demás tablas apuntan al país por su número interno (§4.2).

### F4 · Retirar una moneda, una división o un tipo de documento

**Rol:** superadministrador. **Pantallas:** Monedas, y la ficha del país para divisiones y documentos.

1. Elige "Desactivar" en el menú de la fila. El sistema pregunta antes de hacerlo, como en las pantallas de `organization`.
2. El registro sigue en la lista, marcado como inactivo, y se puede volver a activar.
3. Lo que ya lo usaba sigue válido: una dirección conserva su división, un país o una empresa su moneda y una empresa su tipo de documento.

**Resultado:** el registro deja de ofrecerse en los formularios nuevos, y la API rechaza elegirlo como nuevo. Para la moneda, el error es `CURRENCY_INACTIVE` (LOC-R06).

### F5 · Capturar una dirección desde su dueño

**Rol:** el del dueño; hoy, superadministrador para empresas, sedes y ubicaciones. **Pantalla:** el diálogo del dueño (Empresas, Sedes y Ubicaciones, construidas), con el componente de dirección que presta `location`.

```mermaid
flowchart TD
  A[La persona llena la dirección en el formulario del dueño] --> B{¿El dueño ya tenía dirección?}
  B -- No --> C[Crear: fila nueva en el país del dueño]
  B -- Sí y cambió --> D{El diálogo pregunta: ¿corrección o mudanza?}
  D -- Corrección --> E[Corregir: misma fila]
  D -- Mudanza --> F[Reemplazar: fila nueva; la anterior se conserva]
  C --> G{Validaciones}
  E --> G
  F --> G
  G -- División de otro país, de otro escalón, o nueva e inactiva --> X[Se rechaza: nada se guarda]
  G -- El país exige código postal y falta --> X
  G -- Todo en regla --> H[El dueño guarda la dirección con el resto de su registro]
```

1. El país de la dirección es el del dueño (el de la empresa). La persona no lo elige.
2. La división es obligatoria cuando el país tiene sus divisiones cargadas, como Perú, y opcional en un país sin ellas (D-53). El formulario lo decide con el mismo criterio que la API; si no pudo leer las reglas del país, lo dice, ofrece reintentar y deja guardar los demás datos del dueño mientras la dirección no cambie. Se elige de arriba hacia abajo, un selector por escalón del país: Departamento, después Provincia, después Distrito. Cada selector ofrece solo las divisiones activas que cuelgan de lo elegido arriba, y cambiar un escalón borra los de abajo. Hay que llegar al último escalón; donde la división es opcional, también se puede dejar sin elegir. Una dirección vacía del todo sigue permitida si el dueño no la exige.
3. La calle es obligatoria en cuanto se escribe algo. El código postal aparece y es obligatorio solo en los países que lo exigen (México).
4. Al cambiar la dirección de un dueño ya guardado, los diálogos de empresa, sede y ubicación preguntan si es una corrección o una mudanza (D-8, D-79), y la guardan por el subrecurso de dirección del dueño, aparte de sus demás datos. Una ubicación de terceros que todavía no tiene dirección registra la primera sin preguntar. Quitar la dirección no se permite: solo corregirla o mudarla.
5. Al editar un dueño, el formulario reconstruye la cadena de su división (Lima › Lima › Chorrillos). Una división que ya tenía la dirección se acepta aunque se haya desactivado después, y aparece marcada "(inactiva)"; una que se elige ahora tiene que estar activa (LOC-R26).

**Errores que ve:** "Elija la división hasta Distrito, el último escalón del país" (donde es obligatoria); "Elija también Distrito, o deje la división sin elegir" (donde es opcional); "Elija una división activa del último escalón del país"; "La división que tenía la dirección ya no encaja en los escalones del país: elija una nueva o deje la división sin elegir"; "No se pudieron cargar las divisiones del país"; "No se pudieron leer las reglas de dirección del país: una dirección nueva o cambiada espera a que se lean"; y, desde el backend, un mensaje por cada código `ADDRESS_*` (convenciones §3.3), como "Elija la división de la dirección: su país tiene sus divisiones cargadas" o "La división elegida está inactiva: elija una activa".
**Resultado:** el dueño apunta a su dirección. Al registrar el dueño, si algo falla después, la dirección tampoco se guarda: las dos van en la misma transacción. Al editarlo, la dirección se guarda por su subrecurso antes que los demás datos, en otra petición; si después falla el `PUT`, la dirección ya quedó guardada.

### F6 · Cargar el ubigeo

**Hecho para Perú.** La lista oficial de divisiones de Perú (ubigeo del INEI, edición 2022) se carga completa con la migración `V24__load_peru_ubigeo.sql` (D-2, procesos P1 paso 1). La genera `scripts/ubigeo/` del backend desde el archivo tal como lo publica el INEI, que se guarda byte por byte en `scripts/ubigeo/fuente/` con su fuente y su SHA-256: así se puede volver a generar y comparar con una edición nueva. La migración inserta departamentos, provincias y distritos en ese orden y falla entera si falta un superior; `PeruUbigeoMigrationTests` comprueba los conteos y la cadena. Los nombres quedan como los publica el INEI: en mayúsculas, sin tildes y con la Ñ, y así se muestran en las pantallas (`LIMA › LIMA › CHORRILLOS`), por decisión del responsable del diccionario.

Una edición nueva no reemplaza a V24, porque una migración aplicada no se edita. Cada distrito que una ley crea después entra activo con su propia migración, cuyo número se reserva con el equipo como pide el README de `scripts/ubigeo/`, en cuanto el INEI o SUNAT publican su código; el distrito del que se separó sigue activo, y los que se fusionen se desactivan (D-77). Santa Rosa de Loreto (Ley 32403, 03/07/2025) es el primero y espera ese código. La lista se revisa en el cierre de cada paso, contra las leyes nuevas y las ediciones nuevas del INEI, como la que puede salir del Censo 2025; la última revisión es del 17/10/2026. Volver a cargar la lista desde la aplicación no tiene mecanismo (procesos, apartado 5, pregunta 7).


---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). En la columna **Dónde se aplica**, los nombres de la base son restricciones de las migraciones V4 a V39; los demás, clases del backend o archivos del frontend.

| ID | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| LOC-R01 | El código de una moneda son tres letras de una moneda real de ISO 4217 | §4.1, §2.4 | Base: `currencies_iso_code_format_ck` (solo la forma). Dominio: `IsoCurrencyCode`. Servicio: `CurrencyCommandServiceImpl`; el recurso acepta minúsculas y espacios alrededor y guarda el código en mayúsculas (convenciones §5.3). Frontend: patrón en `currency-form-dialog` | `IsoCurrencyCodeTests`, `CurrencyTest`, `CurrenciesControllerTests` | ✅ |
| LOC-R02 | Dos monedas no tienen el mismo código | §4.1 | Base: `UNIQUE (iso_code)` (V4). Servicio: `CurrencyCommandServiceImpl`; una carrera que la base detiene responde el mismo `CURRENCY_CONFLICT`, traducido en `CurrencyRepositoryImpl` | `CurrencyCommandServiceImplTest`, `CurrenciesControllerTests`, `LocationRacedDuplicatesTests` | ✅ |
| LOC-R03 | El código de una moneda no se edita nunca | §2.4, D-48 | Dominio: `Currency` (el código no tiene cambio). Frontend: campo bloqueado al editar | `CurrencyTest` | ✅ |
| LOC-R04 | Una moneda tiene de 0 a 4 decimales | §4.1 | Base: `currencies_decimal_places_range_ck`. Dominio: `Currency`. Servicio: `@Min`/`@Max` del recurso. Frontend: `currency-form-dialog` | `CurrencyTest` | ✅ |
| LOC-R05 | Los decimales quedan fijos desde el primer monto que usa la moneda; antes se corrigen. Hoy cuenta cualquier orden de compra en esa moneda, de cualquier empresa y en cualquier estado; los fondos no tienen moneda. El nombre y el símbolo se corrigen siempre | §4.1, D-138 | Dominio: `Currency.update` rechaza el cambio con `CurrencyDecimalPlacesInUseException`. Puerto: `CurrencyUsage` (`location.interfaces.acl`), que cada contexto que guarda montos implementa; `location` recibe los que haya sin depender de ninguno, y hoy lo implementa `procurement` con `PurchaseOrderCurrencyUsageAdapter`. Servicio: `CurrencyCommandServiceImpl` bloquea la moneda (`FOR NO KEY UPDATE`) antes de leerla y, solo si cambian los decimales, pregunta a cada `CurrencyUsage` con alcance `SYSTEM`, para contar las órdenes de todas las empresas; responde `422 CURRENCY_DECIMAL_PLACES_IN_USE` y la moneda queda como estaba. Una orden lee su moneda en modo compartido (`fetchCurrencySummaryByIsoCodeForShare`, `914cac1a`) antes de revisar sus precios, así que ningún primer monto se cuela entre la pregunta y el cambio (`7b8db0ee`). Frontend: `currencies.store` dice su propio texto y el diálogo queda abierto con lo escrito (`f9a23d0b`, `da457b00`) | `CurrencyTest.theDecimalPlacesOfACurrencyAnAmountUsesAreFixed` y `anUpdateChangesTheDecimalPlacesOnlyWithAnotherValue`; `CurrencyCommandServiceImplTest.updateRefusesToChangeTheDecimalPlacesOfACurrencyAnAmountUses`, `updateChangesTheDecimalPlacesOfACurrencyNoAmountUses` y `updateChangesTheNameAndTheSymbolOfACurrencyAnAmountUses`; `CurrencyDecimalPlacesIntegrationTests` (`theDecimalPlacesOfACurrencyNoAmountUsesAreCorrected`, `theDecimalPlacesOfACurrencyAnOrderOfAnotherCompanyUsesAreFixed`, `anAnnulledOrderFixesTheDecimalPlacesToo`, `theNameAndTheSymbolOfACurrencyAnOrderUsesStillChange`); en `procurement`, `CurrencyDecimalPlacesLockTests` (`aCorrectionOfTheDecimalPlacesWaitsForAnOpenOrderInTheCurrency`, `anOrderWaitsForAnOpenCorrectionAndChecksItsPricesWithTheNewDecimalPlaces`); `currencies.store.spec` ("says the decimals cannot change once a purchase order uses the currency", así desde `ec7bd7cc`: los fondos no tienen moneda); e2e `currencies` ("says the decimals of a currency an order uses cannot change, and keeps the dialog open") | ✅ |
| LOC-R06 | Una moneda inactiva no se usa en operaciones nuevas; quien ya la tenía la conserva | §4.1 | Servicio: `CountryCommandServiceImpl` para la moneda sugerida del país y, en `organization`, `FunctionalCurrency` para la moneda funcional de la empresa, que recibe el estado por `CurrencySummary.isActive`, y, en `procurement`, `PurchaseOrderReferences` para la moneda de una orden de compra, que el borrador conserva si ya la tenía. Las tres responden `CURRENCY_INACTIVE` (`422`). Frontend: los formularios de país y de empresa ofrecen las activas más la actual, marcada "(inactiva)" | `CountryCommandServiceImplTest`, `CountriesControllerTests`, `FunctionalCurrencyTests`, `CompanyCommandServiceImplTests`, `CompaniesControllerTests`, e2e `country-profile`; en `procurement`, `PurchaseOrdersIntegrationTests.anInactiveCurrencyAnswersCurrencyInactive` y `aDraftKeepsTheCurrencyItHasAfterItIsRetired` | ✅ |
| LOC-R07 | Una moneda, un país, una división o un tipo de documento no se borran | §4.1, §4.5, §4.7 | Servicio: los controladores no tienen `DELETE` y los repositorios del dominio no tienen borrado. Base: la cuenta de la aplicación no tiene permiso de borrar en las seis tablas | `CurrencyAuditTests`, `GeoDivisionCommandServiceImplTests`, `TaxIdTypeAuditTests` (desactivar conserva la fila); `LocationRowsAreNeverDeletedTests` (la base responde `42501` a un `DELETE` de `erp_mm_app` en cada tabla) | ✅ |
| LOC-R08 | Los dos códigos ISO de un país nombran un mismo país de ISO 3166 | §4.2, D-47 | Base: `countries_iso_alpha_2_format_ck` y `countries_iso_alpha_3_format_ck` (solo la forma). Dominio: `IsoCountryCodes`. Servicio: `CountryCommandServiceImpl`. Frontend: patrón en `country-data-form` | `IsoCountryCodesTests`, `CountryTest`, `CountryCommandServiceImplTest`, `CountriesControllerTests` | ✅ |
| LOC-R09 | Dos países no comparten código ISO de dos ni de tres letras | §4.2 | Base: `UNIQUE` de cada código (V5). Servicio: `CountryCommandServiceImpl`, con un error propio para cada código, también en una carrera que la base detiene (`CountryRepositoryImpl` la traduce) | `CountryCommandServiceImplTest`, `CountriesControllerTests`, `LocationRacedDuplicatesTests` | ✅ |
| LOC-R10 | Los códigos ISO se corrigen solo mientras el país no tenga escalones, direcciones ni tipos de documento | §4.2, D-47 | Dominio: `CountryIsoCodesPolicy`, `Country.correctIsoCodes`. Servicio: `CountryCommandServiceImpl`, que bloquea primero la fila del país y después lee sus dependientes, así un escalón que se declara al mismo tiempo espera; responde `COUNTRY_ISO_CODES_LOCKED` (`422`). Frontend: `country-data-form` bloquea los campos con `isoCodesLocked`, que la ficha vuelve a leer al declarar un escalón o crear una división o un tipo de documento | `CountryIsoCodesPolicyTests`, `CountryCommandServiceImplTest`, `CountriesControllerTests`, e2e `country-profile` | ✅ |
| LOC-R11 | El código telefónico es `+` seguido de 1 a 5 dígitos | §4.2 | Base: `countries_calling_code_format_ck`. Dominio: `Country`. Servicio: `@Pattern` del recurso. Frontend: `country-data-form` | `CountryTest` | ✅ |
| LOC-R12 | La zona horaria del país es el nombre IANA de una región, de la forma Área/Lugar: no un desfase fijo (`Etc/GMT+5`, `UTC`) ni un alias sin continente (`US/Eastern`) | §4.2, §2.6, D-40 | Dominio: `Country`, con `RegionTimezone` de `shared`, que también usa la sede en `organization`. Servicio: `@ValidTimezone` (`shared`). Frontend: `country-data-form` ofrece solo esas zonas, y una zona guardada que ya no lo es se marca para elegir otra | `RegionTimezoneTests`, `CountryTest`, `CountriesControllerTests` | ✅ |
| LOC-R13 | La tasa del impuesto va de 0 a 1, con hasta cuatro decimales | §4.2 | Base: `countries_tax_rate_range_ck` y `NUMERIC(5,4)`. Dominio: `Country`. Servicio: `@DecimalMin`, `@DecimalMax` y `@Digits(integer = 1, fraction = 4)`. Frontend: 0 a 100 % | `CountryTest`, `CountriesControllerTests` (un quinto decimal se rechaza) | ✅ |
| LOC-R14 | Un país declara de 1 a 4 escalones | §4.4 | Base: `CHECK (level >= 1 AND level <= 4)` (V8). Dominio: `Country.canDeclareLevel`, `HierarchyTier`. Servicio: `CountryCommandServiceImpl` | `CountryTest`, `GeoDivisionLevelAuditTests`, `GeoDivisionLevelsControllerTests` | ✅ |
| LOC-R15 | Los escalones se declaran en orden, sin saltos: el siguiente es siempre el que sigue al último | §4.4, D-45 | Dominio: `Country.declareLevel` pone el número. Servicio: `CountryCommandServiceImpl` bloquea la fila del país antes de declarar, así dos declaraciones a la vez van una después de la otra. La base solo impide repetirlo (`geo_division_levels_country_level_uq`), no el salto | `CountryTest`, `GeoDivisionLevelsControllerTests`, `CountryLockTests` | ✅ |
| LOC-R16 | Un escalón no se borra ni se retira; solo se renombra | §4.4, D-45 | Dominio: `Country` (solo declara y renombra). Servicio: no hay `DELETE`. Frontend: sin botón | `GeoDivisionLevelsControllerTests` (`levelsCannotBeDeleted`) | ✅ |
| LOC-R17 | Una división va en un escalón que su país declaró | §4.5 | Base: `geo_divisions_country_level_fk`. Servicio: `GeoDivisionCommandServiceImpl` | `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests` | ✅ |
| LOC-R18 | El código oficial de una división no se repite en el país; se guarda en mayúsculas y sin guiones, espacios ni puntos, así `t-2`, `T 2` y `T2` son el mismo código | §4.5, §2.2 | Base: `geo_divisions_country_code_uq`; V39 normalizó los códigos ya guardados y falla si dos quedan iguales. Dominio: `GeoDivisionCode`, con `TextIdentifier` de `shared`. Servicio: `GeoDivisionCommandServiceImpl`, también en una carrera que la base detiene (`GEODIVISION_CONFLICT`). Frontend: el diálogo limpia el código con la misma regla antes de medirlo | `GeoDivisionCodeTests`, `TextIdentifierTest`, `TextIdentifiersMigrationTests`, `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests`, `LocationRacedDuplicatesTests` | ✅ |
| LOC-R19 | El código, el escalón, la división superior y el país de una división no cambian | §2.4, §4.5 | Dominio: `GeoDivision` (campos fijos; solo se renombra y se activa). Servicio: la edición solo recibe el nombre | No hace falta: la API no recibe esos campos al editar | ✅ |
| LOC-R20 | Solo las divisiones del escalón 1 no tienen división superior | §4.5 | Base: `geo_divisions_root_parent_ck`. Dominio: `GeoDivision`. Servicio: `GeoDivisionCommandServiceImpl` | `GeoDivisionTests`, `GeoDivisionCommandServiceImplTests` | ✅ |
| LOC-R21 | La división superior es del mismo país | §4.5 | Base: `geo_divisions_parent_same_country_fk` (V15). Servicio: `GeoDivisionCommandServiceImpl` | `GeoDivisionCommandServiceImplTests`, `SameCountryReferencesMigrationTests.theParentOfADivisionIsOfItsCountry` | ✅ |
| LOC-R22 | La división superior es del escalón inmediato superior: un distrito no cuelga directo del departamento | §4.5, D-54 | Base: `geo_divisions_parent_immediate_level_fk` sobre la columna generada `parent_level` (V25), que protege también las cargas en SQL. Servicio: `GeoDivisionCommandServiceImpl` con `HierarchyTier.isImmediateChildOf`. Frontend: `geo-division-form-dialog` solo ofrece el escalón de arriba | `GeoDivisionParentLevelMigrationTests`, `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests`, `HierarchyTierTests` | ✅ |
| LOC-R23 | Una división retirada se desactiva y sigue existiendo | §4.5 | Dominio: `GeoDivision.deactivate` | `GeoDivisionCommandServiceImplTests`, `GeoDivisionsControllerTests` | ✅ |
| LOC-R24 | Toda dirección tiene calle | §4.6 | Base: `CHECK` de `street_name` (V10). Dominio: `Address`. Frontend: `address-fields` en cuanto se escribe algo | `AddressTests`, `address-fields.spec` | ✅ |
| LOC-R25 | La división de una dirección es del mismo país que la dirección | §4.6 | Base: `addresses_geo_division_same_country_fk` (V15). Dominio: `AddressDivisionPolicy`. Servicio: `AddressCommandServiceImpl` responde `422 ADDRESS_DIVISION_OTHER_COUNTRY` | `AddressDivisionPolicyTests`, `SameCountryReferencesMigrationTests.theDivisionOfAnAddressIsOfItsCountry` | ✅ |
| LOC-R26 | La división de una dirección es del escalón más bajo del país, y está activa si se elige ahora; la que la dirección ya tenía se conserva aunque se haya desactivado | §4.6 | Dominio: `AddressDivisionPolicy`. Servicio: `AddressCommandServiceImpl`, que responde `422 ADDRESS_DIVISION_NOT_DEEPEST_LEVEL` o `422 ADDRESS_DIVISION_INACTIVE`. Frontend: `mm-address-fields` elige en cascada, ofrece solo activas para elegir y conserva, marcada "(inactiva)", la que la dirección ya tenía | `AddressDivisionPolicyTests`, `AddressCommandServiceImplTests`, `address-fields.spec`, `address-options.store.spec`, e2e `companies` | ✅ |
| LOC-R27 | El código postal es obligatorio en los países que lo exigen | §4.2, §4.6 | Dominio: `Address`. Servicio: `AddressCommandServiceImpl`, que responde `422 ADDRESS_POSTAL_CODE_REQUIRED`. Frontend: `address-fields`. La base no lo cubre | `AddressTests`, `AddressCommandServiceImplTests`, `address-fields.spec` | ✅ |
| LOC-R28 | La dirección está en el país de su dueño; quien la escribe no elige el país | §4.6 | Servicio: la fachada recibe el país del dueño (`createAddress`, `replaceAddress`) y `AddressData` no trae país. Base: `companies_fiscal_address_same_country_fk` (V18) y `branches_address_same_country_fk` (V20). En las ubicaciones lo asegura solo la aplicación, como permite §4.6: `locations` no tiene columna de país, y su dirección se crea en el país de la empresa | `LocationContextFacadeImplTest`, `BranchesIntegrationTests`, `SameCountryReferencesMigrationTests.theFiscalAddressOfACompanyIsOfItsCountry`, `CompanyFiscalAddressConstraintTests`; la parte de las ubicaciones, `LocationCommandServiceImplTests.theAddressOfANewLocationIsCreatedInTheCountryOfItsCompany` y `aMoveOfTheAddressCreatesTheNewOneInTheCountryOfItsCompany` (`99f410d`), que comprueban que `createAddress` y `replaceAddress` reciben el país de la empresa | ✅ |
| LOC-R29 | Una dirección no se comparte: cada dueño tiene su propia fila | §4.6 | Base: un `UNIQUE` en cada tabla dueña, que impide que dos filas de la misma tabla apunten a la misma dirección: `companies_fiscal_address_id_uq` (V18), `uq_branches_address_id` (V20) y `locations_address_uq` (V26). Entre tablas distintas —una empresa y una sede con la misma fila— la base no lo impide: lo asegura la aplicación, porque cada dueño solo crea, corrige o reemplaza la suya y la fachada nunca entrega una dirección para que otro la use | `SameCountryReferencesMigrationTests.twoCompaniesCannotShareAFiscalAddress`, `BranchConstraintTests.twoBranchesCannotShareAnAddress`, `OrganizationOwnershipKeysMigrationTests.twoLocationsCannotShareAnAddress` | ✅ |
| LOC-R30 | Corregir modifica la misma fila; mudarse crea una fila nueva y conserva la anterior | §4.6, D-8, D-79 | Servicio: `LocationContextFacadeImpl` (`correctAddress`, `replaceAddress`); empresas, sedes y ubicaciones cambian la dirección solo por su subrecurso, y el `PUT` del dueño no la toca. Frontend: los diálogos de empresa, sede y ubicación preguntan cuál de las dos es | `LocationContextFacadeImplTest`, `BranchesIntegrationTests`, `LocationsIntegrationTests`, `location-form-dialog.spec`, e2e `locations` | ✅ |
| LOC-R31 | La sigla de un tipo de documento no se repite en el país | §4.7 | Base: `tax_id_types_country_code_uq`. Servicio: `TaxIdTypeCommandServiceImpl` | `TaxIdTypeCommandServiceImplTests`, `TaxIdTypesControllerTests` | ✅ |
| LOC-R32 | La sigla y el país de un tipo de documento no se editan | §4.7 | Dominio: `TaxIdType` (campos fijos). Frontend: campos bloqueados al editar | `TaxIdTypeTests`, `TaxIdTypesControllerTests` | ✅ |
| LOC-R33 | Un tipo aplica a empresa, persona o ambos, y solo se usa para lo que aplica: una empresa no se registra con un DNI | §4.7 | Base: `tax_id_types_applies_to_ck`. Dominio: `TaxIdApplicability`. El uso lo revisa `organization` con `TaxIdTypeRules`, que recibe la decisión ya tomada (`coversCompanies`, `coversPeople`) | `TaxIdApplicabilityTests`, `TaxIdTypeRulesTests` | ✅ |
| LOC-R34 | Cuando el tipo declara una longitud, va de 1 a 20 —lo que caben las columnas que guardan los números— y el número, ya normalizado, tiene exactamente esa longitud | §4.7, D-78 | Base: `tax_id_types_length_ck` (V38, antes hasta 30). Dominio: `TaxIdType`. Servicio: `@Max(20)` del recurso, con su propio texto. Frontend: el campo admite de 1 a 20. El uso lo revisa `TaxIdTypeRules` en `organization`. El dato no alcanza para el RFC, el NIT o la CC (sección 6) | `TaxIdTypeTests`, `TaxIdTypeLengthMigrationTests`, `TaxIdTypesControllerTests`, `TaxIdTypeRulesTests` | ✅ |
| LOC-R35 | Cuando el tipo tiene dígito de control, el número lo cumple. Se verifica al crear el registro y cuando cambian el número o el tipo; un número ya guardado sigue siendo válido | §4.7, D-75 | Dominio: `TaxIdCheckDigitPolicy` calcula el dígito según la sigla del tipo dentro de su país; hoy solo el RUC de Perú, con el módulo 11 de SUNAT. Un tipo con dígito de control cuyo cálculo todavía no existe —el NIT, el RFC y la CURP— acepta sus números sin verificar (`NOT_VERIFIED`). Fachada: `verifyTaxIdCheckDigit`. El uso lo aplica `organization` con `TaxIdTypeRules`: `COMPANY_TAX_ID_CHECK_DIGIT_INVALID` y `EMPLOYEE_DOCUMENT_NUMBER_CHECK_DIGIT_INVALID` (`422`). Frontend: los diálogos de empresa y colaborador verifican el RUC antes de enviar, con la misma regla (`location/domain/model/tax-id-check-digit.ts`); la API decide igual. V37 deja el DNI sin dígito de control (D-76) | `TaxIdCheckDigitPolicyTests`, `LocationContextFacadeImplTest`, `PeruTaxIdTypesMigrationTests`, `TaxIdTypeRulesTests`, `CompanyCommandServiceImplTests`, `EmployeeCommandServiceImplTests`, `tax-id-check-digit.spec` | ✅ |
| LOC-R36 | Un tipo de documento inactivo no se elige en registros nuevos; quien ya lo usa sigue válido | §4.7 | Dominio: `TaxIdType.changeStatus`. El uso lo revisa `TaxIdTypeRules` en `organization`, que exige el tipo activo solo cuando es nuevo para el registro: un colaborador cuyo tipo se desactivó se sigue editando. Frontend: el diálogo de colaborador ofrece los tipos activos más el que el colaborador ya tiene, rotulado "(inactivo)" si se desactivó (`9842ea8`, rama `feature/organization-closure`) | `TaxIdTypeRulesTests`, `TaxIdTypesControllerTests`, `EmployeeCommandServiceImplTests.anUnchangedDocumentTypeThatWasDeactivatedStillAllowsTheUpdate` y `changingToAnInactiveDocumentTypeIsRejected`; `employee-form-dialog.spec` ("keeps the document type the employee has, said inactive when it was deactivated") | ✅ |
| LOC-R37 | Quien usa un tipo de documento es de su mismo país | §4.7 | Base: `companies_tax_id_type_same_country_fk` (V15). En los colaboradores lo asegura solo la aplicación: `employees` no tiene columna de país, y el país es el de su empresa. El uso lo revisa `TaxIdTypeRules` | `TaxIdTypeRulesTests`, `SameCountryReferencesMigrationTests.theTaxIdTypeOfACompanyIsOfItsCountry`; el país del documento del colaborador, `TaxIdTypeRulesTests.aTypeOfAnotherCountryIsRejectedForAnEmployee` (`99f410d`), con un colaborador nuevo y con uno que conserva su tipo y su número | ✅ |
| LOC-R38 | Los catálogos los lee quien tiene `catalogos.ver` por un rol vigente, y los crea, corrige o desactiva quien tiene el permiso de esa operación (D-130); la semilla da la lectura a los siete roles del sistema y las escrituras solo al superadministrador (D-43). Un rol propio sin el permiso recibe `403`, también con sesión y rol, y un permiso desmarcado del rol deja de valer en la petición siguiente. Las pantallas también piden la lectura: desde el 30/10/2026 se abren solo con `catalogos.ver`, y el alta de un país además con `catalogos.crear_paises` | §5.6, D-43, D-95, D-130, D-141 | Servicio: `@PermissionRequired("catalogos.ver")` en los ocho controladores y, en cada escritura, el permiso de esa operación desde V62: `catalogos.crear_monedas`, `editar_monedas` y `desactivar_monedas`; `crear_paises` para registrar un país; `editar_paises` para corregirlo y para declarar y corregir sus escalones y crear y corregir sus divisiones y sus documentos tributarios; `desactivar_paises` para activar y desactivar divisiones y documentos tributarios. Hasta el 21/10/2026, `catalogos.gestionar_generales` en cada escritura (`9f6c1d8`), que ya no pide ninguna ruta. Ningún servicio de `location` decide por empresa, porque sus tablas no llevan `company_id`. Frontend: `location.routes.ts` y `app-navigation.ts`, con `catalogos.ver` (`71765c0b`) | `LocationReadAccessTests` (cada lectura y exportación: `401` sin sesión, `403` sin rol, la leen `ADMIN_EMPRESA` y `ADMIN_ACCESOS`, y `aRoleOfItsOwnWithoutThePermissionGets403`); `LocationPermissionIntegrationTests` (`aRoleOfItsOwnReadsTheCatalogsWithThePermissionAndNotWithoutIt`, `aRoleOfItsOwnWithoutThePermissionReadsNoCatalog`, `aPermissionUnmarkedFromTheRoleStopsOnTheNextRequest`, `aRetiredPermissionWritesNothing`, `aRoleOfItsOwnWritesTheCatalogsWithTheWritePermission`); los `*ControllerTests`, con el `403` de cada escritura; `location.routes.spec` ("opens no catalog nor the record of a country to a session that only writes them: every GET asks catalogos.ver (owner, 30/10/2026)", "opens the catalogs and the record of a country with catalogos.ver alone", "opens the form of a new country with the reading and the permission it saves with") | ✅ |
| LOC-R39 | Un archivo exportado lleva códigos y nombres, nunca el número interno ni el `public_id` | §2.4, D-51 | Servicio: las columnas de cada exportación en `CurrenciesController`, `CountriesController`, `CountryGeoDivisionsController` y `CountryTaxIdTypesController`. Las divisiones y los tipos de documento llevan el país, y las divisiones el código de su superior | Las cuatro exportaciones tienen prueba en el backend, y e2e `exports` | ✅ |
| LOC-R40 | Las seis tablas registran cuándo se creó y cambió cada fila, sin autor | D-42 | Base: V6 retira `created_by`. Las entidades extienden `AuditableAbstractPersistenceEntity` | Los seis `*AuditTests` | ✅ |
| LOC-R41 | Una dirección tiene su división cuando su país tiene divisiones cargadas; solo puede faltar en un país sin ellas | §4.6, D-53 | Servicio: `AddressCommandServiceImpl` (crear, corregir y mudar) responde `422 ADDRESS_DIVISION_REQUIRED` cuando el país tiene alguna división, de cualquier escalón y estado. Frontend: `mm-address-fields` usa el mismo criterio, y mientras no conoce las reglas del país no da por válida una dirección nueva o cambiada. La base no lo cubre: depende de qué países tienen su lista cargada | `AddressCommandServiceImplTests`, `CompaniesControllerTests`, `address-fields.spec`, `address-options.store.spec`, e2e `companies` y `locations` | ✅ |

---

## 5. Relación con otros contextos

`location` no depende de ningún contexto de negocio, y `ModularityTests` lo verifica. Desde la primera fase de la segunda ronda de la auditoría, integrada en `develop` (merge `e245186`), declara `allowedDependencies = {"documents::acl"}`, porque sus cuatro exportaciones las pide al módulo `documents` (D-125); hasta entonces, `{}`. Sus 21 `@PermissionRequired` nombran constantes de `PermissionCodes.Catalogs`, de `shared`: `VIEW` en los ocho controladores y, desde D-130 (V62), `CREATE_CURRENCIES`, `EDIT_CURRENCIES`, `DEACTIVATE_CURRENCIES`, `CREATE_COUNTRIES`, `EDIT_COUNTRIES` y `DEACTIVATE_COUNTRIES` en las escrituras; hasta entonces, `MANAGE_GENERAL`. La versión de los datos de sus exportaciones y de su fachada la da `LocationDataVersions`, implementado en `infrastructure` y leído de los contadores de `data_versions` (D-128). Dentro del contexto, los agregados se referencian por su identificador: la división y el tipo de documento guardan el país, y el país guarda su moneda sugerida por identificador; el código ISO de la moneda lo agrega la vista del país. Los demás lo usan por su fachada, nunca por sus tablas (§3).

**El puerto `CurrencyUsage`** (29/10/2026, `7b8db0ee`; LOC-R05). `location` declara en `interfaces.acl` la pregunta «¿algún registro de este contexto guarda un monto en esta moneda, en cualquier empresa?» (`anyAmountIn`), y cada contexto que guarda montos la responde con un bean propio. `location` recibe todos los que haya sin depender de ninguno: la dependencia va de quien implementa hacia `location`, como con la fachada. Hoy la implementa `procurement`, con `PurchaseOrderCurrencyUsageAdapter`, que cuenta las órdenes de compra de cualquier estado; `fleet` no, porque un fondo no tiene moneda (§12.1). Las facturas, las notas de crédito y los pagos la implementarán cuando se construyan.

**Fachada del backend: `LocationContextFacade`.** El detalle de lo que ofrece está en mapa §5.1. Sus consumidores son `organization` y, desde el paso 7, `procurement`, cada uno por su `ExternalLocationService`.

| Miembro | Para qué | Quién lo usa |
|---|---|---|
| `fetchCurrencySummaryByPublicId` | Resolver una moneda por su identificador público, con si está activa | **Nadie en producción** desde el 18/10/2026, cuando la empresa pasó a nombrar su moneda por el código ISO (`d5a84474`); lo prueba `LocationContextFacadeImplTest` |
| `fetchCurrencySummaryByIsoCode` | Resolver una moneda por su código ISO, como la nombra un mensaje (D-13), con si está activa | Empresas: la moneda funcional (`CompanyCommandServiceImpl`, por `FunctionalCurrency`) |
| `fetchCurrencySummaryByIsoCodeForShare` | La moneda de un registro que escribe un monto en ella, leída en modo compartido hasta el fin de la transacción de quien llama: su estado y sus decimales no cambian entre la revisión y la escritura, y una corrección de los decimales espera o ve el monto (LOC-R05, LOC-R06; `914cac1a`) | Órdenes de compra (`PurchaseOrderReferences`), que desde el 29/10/2026 nombran su moneda por el código ISO |
| `fetchCurrencySummariesByIds` | Mostrar la moneda en listas, en lote. `CurrencySummary` lleva el código, el símbolo —desde el 27/10/2026, que la orden de compra imprime antes de un monto (`42ecdfa2`)—, los decimales y si está activa | Vistas de empresas; vistas y PDF de las órdenes de compra |
| `fetchCountrySummaryByPublicId` | Resolver un país por su identificador público | Empresas: el filtro por país de la lista (`CompanyQueryServiceImpl`) |
| `fetchCountrySummaryByIsoAlpha2` | Resolver un país por su código de dos letras, como lo nombra la empresa (D-13) | Empresas: el alta (`CompanyCommandServiceImpl`), desde el 18/10/2026 (`d5a84474`); también las pruebas de integración de `organization`, para resolver los países sembrados |
| `fetchCountrySummariesByIds` | Mostrar el país en listas, en lote, y su zona horaria (`CountrySummary.defaultTimezone`) | Vistas de `organization`; la zona, para el "hoy" de un colaborador sin sede (`EmployeeAffiliationPolicy`) y para la zona de cada empresa que `iam` entrega en `/me` |
| `fetchTaxIdTypeSummaryByPublicId` | Las reglas del tipo de documento de una empresa, un colaborador o un proveedor | Empresas, colaboradores y proveedores (`TaxIdTypeRules`) |
| `fetchTaxIdTypeSummariesByIds` | Mostrar el tipo de documento en listas, en lote | Vistas de empresas, colaboradores, proveedores y plantillas; el PDF de la orden de compra |
| `verifyTaxIdCheckDigit` | Verificar el dígito de control de un número por la sigla de su tipo en su país (LOC-R35, D-75): `VALID`, `INVALID`, `NOT_APPLICABLE` o `NOT_VERIFIED` | Empresas, colaboradores y proveedores, por `TaxIdTypeRules` |
| `createAddress`, `correctAddress`, `replaceAddress` | Crear, corregir y mudar la dirección del dueño, en su transacción (mapa §6.3) | Empresas, sedes, ubicaciones y proveedores |
| `copyAddress` | Copiar tal cual una dirección de otro dueño a una fila nueva: mismo país, misma división aunque esté inactiva y mismas partes; no vuelve a revisar las reglas de una dirección nueva porque nada se elige | Plantillas de orden de compra (domicilio del emisor, copiado del domicilio fiscal de la empresa; D-134) |
| `fetchAddressSummariesByIds` | Mostrar la dirección embebida en el dueño, en lote | Vistas de empresas, sedes, ubicaciones, proveedores y plantillas; el PDF de la orden de compra |
| `fetchDataVersion` | La versión de los datos de `location` que muestra una exportación de quien llama, para no servir un documento viejo (D-128) | Las exportaciones de `organization` y de `procurement` |

Lo que la fachada no ofrece y mapa §5.1 deja como propuesta: la división por su identificador público. Ningún contexto la pide todavía. La zona horaria del país ya viaja en `CountrySummary`, y `organization` la usa.

**Frontend.** `LocationFacade` entrega vistas mínimas (`CountryOption`, `CurrencyOption`, `TaxIdTypeOption`), nunca entidades, y presta `checkDigitHolds`, la misma regla del dígito de control que aplica el backend, para que los diálogos de `organization` avisen antes de enviar. La única pieza de pantalla que `location` presta es `mm-address-fields` (`presentation/public/address-fields`), un control de formulario que produce el valor de la dirección; guardarla le toca al dueño. Lo usan los diálogos de empresa, sede y ubicación de `organization` y el del proveedor de `procurement`. Desde el 19/10/2026 cada escalón se elige escribiendo, con `mm-search-select` de `shared` (DESIGN.md, revisión 21): busca dentro del nombre y del código, sin distinguir mayúsculas ni tildes ("nepena" encuentra NEPEÑA), y solo la opción elegida es el valor; salir sin elegir deja la anterior, y un escalón no se vacía por una tecla suelta. La zona horaria del país usa el mismo campo, y una zona guardada que el navegador no lista se sigue mostrando. En el diálogo de la estructura territorial, el padre de una división se busca en el servidor mientras se escribe (`q`): hasta ese día la búsqueda no corría, solo se ofrecían las primeras 20 divisiones, y Puno, San Martín, Tacna, Tumbes y Ucayali no podían ser padres.

**Contextos que llegarán.** `vehiclecatalog` puede usar `location` y hoy no lo necesita. `fleet` usará la moneda del costo de equipamiento, en el paso 9; sus fondos no tienen moneda (§12.1). `procurement` ya usa las monedas, las direcciones de proveedores y plantillas y el tipo de documento del proveedor desde el paso 7 (arriba), y sus facturas, notas de crédito y pagos implementarán `CurrencyUsage` (mapa §4).

---

## 6. Estado y pendientes para cerrar

**Estado al 18/10/2026: cerrado para Perú**, el único país que opera hoy (D-5). El responsable validó este documento el 18/10/2026, después de las correcciones que pidió la auditoría de `organization`. El cierre se hizo en `feature/location-closure`: 44 commits del backend sobre `ee4df37`, con las migraciones V36 a V39, y 57 del frontend sobre `a8ab115`. `feature/organization-closure`, que sale de ella, corrigió además V39 (`aade406`) y la capa de pantalla de LOC-R36 (`9842ea8`). Partió de la auditoría del 17/10/2026, que encontró 60 hallazgos —ninguno bloqueante, cinco importantes—, y de las decisiones D-75 a D-80 del diccionario v1.20. Lo que hizo:

- **Dígito de control** (LOC-R35, D-75, D-76): `location` lo calcula por la sigla del tipo en su país y lo ofrece por su fachada; hoy, el RUC de Perú con el módulo 11 de SUNAT. `organization` lo aplica en empresas y colaboradores, y los diálogos verifican el RUC antes de enviar. Un tipo cuyo cálculo todavía no existe acepta sus números sin verificar. V37 deja el DNI sin dígito de control.
- **El tipo inactivo en su consumidor** (LOC-R36): editar un colaborador ya no rechaza el tipo de documento que tenía cuando ese tipo se desactivó. El diálogo de colaborador ya no deja el campo en blanco: ofrece ese tipo junto con los activos, rotulado "(inactivo)" (`9842ea8`).
- **Direcciones:** un código propio para cada regla (`ADDRESS_*`, `422`) en lugar de `VALIDATION_ERROR` y `BUSINESS_RULE_VIOLATION`; el mismo criterio de división obligatoria en el frontend y en la API (D-53); y las ubicaciones, como las sedes, corrigen o mudan su dirección solo por `/locations/{id}/address`, con la pregunta de corrección o mudanza en su diálogo, y no pasan de propias a terceros (D-79).
- **Identificadores normalizados** (§2.2, D-86): el código de una división y el número de documento de un colaborador se guardan en mayúsculas y sin guiones, espacios ni puntos, como ya se guardaba el de una empresa, también con los separadores de Unicode. V39 normalizó lo que ya estaba guardado y registra el cambio en las filas que reescribe (`f111262`); en `feature/organization-closure` quita exactamente los separadores que quita la aplicación (`aade406`). El inventario de solo lectura `scripts/database/inventory-text-identifiers.sql` dice qué filas reescribiría y si se detendría (`c94a4bf`). V39 llegó a la base desplegada con la integración del 18/10/2026 y no encontró ninguna colisión: el despliegue terminó sin errores (sección c).
- **Zonas horarias:** el país y la sede aceptan solo nombres IANA de región (§2.6), y una sede guardada con una zona que antes valía se sigue leyendo.
- **Longitud de un tipo de documento** de 1 a 20 (D-78, V38).
- **Carreras:** un duplicado que llega a la base responde el código de su recurso, traducido en el adaptador, y la corrección de los ISO y la declaración de un escalón bloquean primero la fila del país.
- **API:** los errores inesperados ya no exponen el mensaje interno (convenciones §13 fila 14), `size=0` responde una página de una fila, la API responde en español cuando la petición no nombra un idioma que hable, y las exportaciones declaran sus respuestas. V36 quitó tres índices redundantes.
- **Pruebas:** las llaves del mismo país, la dirección fiscal única y la falta de `DELETE` contra PostgreSQL; cada lectura con `401`, `403` y los roles sembrados; `ExternalLocationService` campo por campo; las exportaciones de países y documentos; la accesibilidad de las pestañas y los diálogos; y Ubicaciones con su flujo, su captura y su accesibilidad. El comando del README del backend corre de verdad las pruebas del paquete. Las pruebas de `organization` que leían empresas ya no suponen una base de verificación vacía (`da52323`), y LOC-R28 y LOC-R37 tienen su prueba (`99f410d`).
- **README del backend:** dice que `location` carga el ubigeo con V24 y que `vehiclecatalog` existe desde V35 (`9346134`); antes decía que faltaba el archivo del INEI.
- **Frontend:** textos según el diccionario ("Moneda sugerida", "dígito de control"), decimales y longitudes enteros sin redondeo, la búsqueda de la división superior, los filtros de la pestaña Documentos, los fallos de red que ya no se muestran como datos vacíos, y los selects deshabilitados de un país sin divisiones que dicen por qué.
- **Sonda contra el backend real:** el spec `e2e-live/location.live.spec.ts`, con su propia configuración (`playwright.live.config.ts`), sin la API simulada, y una cuenta de verificación con contraseña permanente creada fuera de Flyway (`scripts/database/create-verification-superadmin.sql`). Su resultado está en la [revisión del 18/10/2026](../retroalimentacion/2026-10-18-location.md).

**Verificación final:** 18/10/2026, sobre `feature/organization-closure` —backend `26bc1e6`, 93 commits sobre `ee4df37`, con las migraciones V36 a V44; frontend `0a40211`, 119 commits sobre `a8ab115`—. La suite completa del backend: 1 576 pruebas sin fallos. Las unitarias del frontend: 1 273 sin fallos. La suite e2e completa: 641 pasan y 22 se omiten por proyecto, en tres corridas seguidas sin fallos. Las sondas contra el backend real: `location`, 8 de 8; `organization`, 8 de 10, con 2 omitidas porque la base tiene una sola empresa (el vínculo entre empresas de D-82 y la frontera del administrador de empresa). Las sondas corrieron sobre `erp_mm_db_verify`, por decisión del responsable, porque `erp_mm_db_live` todavía no existía. Las unitarias y las e2e corrieron sobre `68936ad`; los dos commits siguientes (`e9759d2`, `0a40211`) solo tocan los specs de la sonda.

El detalle de los hallazgos, la sonda y la revisión independiente de las correcciones está en la [revisión del 18/10/2026](../retroalimentacion/2026-10-18-location.md).

### a) Decisiones

Ninguna pendiente para el paso 1. El diccionario v1.20 registró el dueño del dígito de control (D-75), el DNI sin dígito (D-76), la edición del ubigeo (D-77), la longitud de 1 a 20 (D-78), las direcciones de las ubicaciones (D-79), el momento de acotar `addresses` (D-80) y las redacciones de D-53 y D-54. Después, el responsable del diccionario decidió además que los identificadores se guardan sin guiones, espacios ni puntos (§2.2), que el ubigeo se muestra en mayúsculas como lo publica el INEI, que el formulario también verifica el dígito del RUC, que Santa Rosa de Loreto espera su código oficial y que la sonda contra el backend real es un criterio de cierre ([orden de implementación](../orden-de-implementacion-millennium-mobility.md), "Cuándo un paso está terminado"). El diccionario v1.21 precisó además que la normalización sin guiones es de los identificadores que fija una norma externa, como el código de ubigeo, y que los códigos propios conservan sus guiones (D-86). La única decisión abierta del paso 1 con disparo propio son las longitudes y el dígito de control de los documentos de Colombia y México, con su momento fijado (abajo); las demás preguntas abiertas están en la sección 7.

### b) Código

No queda código de `location` pendiente para el paso 1. Lo que sigue abierto tiene su momento asignado:

| Qué | Cuándo |
|---|---|
| ~~Leer los catálogos con un permiso propio y no con cualquier rol vigente (D-43). Toca los ocho controladores y sus pruebas (LOC-R38)~~ | Construido el 19/10/2026 en `feature/permissions-matrix` (paso 3b, `9f6c1d8`): `catalogos.ver` y `catalogos.gestionar_generales`. Llega a `develop` con esa rama |
| ~~Fijar los decimales de una moneda desde su primer monto (LOC-R05)~~ | Resuelto el 29/10/2026, por decisión del responsable (D-138): el puerto `CurrencyUsage`, que implementa `procurement` con sus órdenes de compra, y `422 CURRENCY_DECIMAL_PLACES_IN_USE` (backend `7b8db0ee`; frontend `f9a23d0b` y `da457b00`). Cada contexto que guarde montos después, como las facturas, los pagos o el costo de equipamiento, implementa el puerto con su paso |
| ~~Un rol propio que solo escribe Monedas o Países, sin `catalogos.ver`, ve la opción del menú y abre la pantalla, pero la lista le responde `403`: toda lectura exige `catalogos.ver`~~ | Resuelto el 30/10/2026 por D-141 (frontend `71765c0b`): las dos pantallas y la ficha del país se abren solo con `catalogos.ver`, y `countries/new` además con `catalogos.crear_paises` (LOC-R38) |
| ~~La búsqueda `q` distingue tildes: "peru" no encuentra "Perú" (convenciones §6.2)~~ | Resuelto el 30/10/2026 por D-142 (`145fc104`, `a54ee29b`): las búsquedas de monedas, países, divisiones y documentos tributarios pliegan los dos lados con `fold_search_text`, de V86. Lo prueba `CountriesControllerTests.theSearchFindsACountryWhateverItsAccents` |
| Los tipos de cambio por día y el tipo de tasa de las compras (D-46, §4.8); la moneda de grupo, con los reportes | Justo antes del paso 7; la moneda de grupo, con el paso 11 |
| ~~Acotar `addresses` por empresa, con Row Level Security (D-80, §2.5)~~ | Construido en V78 (26/10/2026, D-134): por el país de la dirección, con rama `SYSTEM` (`access_scope_reaches_country`), tres políticas y ningún `DELETE`; V78 se detiene, nombrando las filas, si la dirección de una empresa, una sede, una ubicación, un proveedor o un emisor de plantilla está en otro país que su empresa; antes de revisar bloquea contra escrituras `addresses` y las tablas de sus dueños, y espera el candado a lo sumo cinco segundos. Las lecturas de «tiene direcciones» —la corrección de los ISO en `CountryCommandServiceImpl` y `isoCodesLocked` en `CountryViewFactory`— van en `SystemAccessScope`. `addresses.company_id` queda como candidato si D-5 o el proveedor local cambian (diccionario §4.8) |
| Colombia y México: las longitudes de sus documentos y el cálculo del dígito de control del NIT, el RFC y la CURP (D-75), y sus listas oficiales (DANE, INEGI). **Riesgo aceptado:** hasta entonces nada en el código impide registrar una empresa de esos países, y sus tipos con dígito de control aceptan cualquier número. No hay que crear a mano divisiones de Colombia ni de México: con una sola división cargada, toda dirección de ese país pasa a exigir la suya (D-53) | Antes de registrar la primera empresa de Colombia o México (D-52) |
| Dar de alta Santa Rosa de Loreto (Ley 32403) con su propia migración | Cuando el INEI o SUNAT publiquen su código (D-77) |
| Revisar la lista de divisiones de Perú contra las leyes nuevas y las ediciones nuevas del INEI | En el cierre de cada paso (D-77); la última revisión es del 17/10/2026 |
| El alta manual de una división acepta cualquier forma de código, que después queda fija: solo se limpia y se pasa a mayúsculas, sin comparar con la forma del código oficial del país (G02-b) | Si el alta manual vuelve a usarse para un país con lista oficial |
| Los nombres de las pruebas: conviven los sufijos `*Test` y `*Tests`, y los `*AuditTests` están repartidos en dos paquetes. Es deuda cosmética de todo el proyecto, no de `location` | Cuando se unifique en todos los contextos |

La fila de las llaves del mismo país en `locations.address_id` y `employees.document_type_id` salió: no se podía hacer como estaba escrita, porque ninguna de las dos tablas tiene columna de país, y el diccionario deja esos dos controles a la aplicación (§4.6, §4.7). Quedan como LOC-R28 y LOC-R37, con su prueba (`99f410d`).

Las propuestas de las convenciones de la API que el contrato de `location` todavía no sigue —`taxRate` como texto decimal (§14.2), el campo `rule` en los `422` (§14.5), `If-Match` para dos ediciones simultáneas (§14.7), `201` con `Location` (§14.9) y el guion bajo en los códigos de error (§14.11)— cambian el contrato cuando el responsable las valide, y §14.2 antes del primer monto (auditoría del 17/10/2026, F62). Las listas públicas del registro (§14.16) esperan a que se encienda el registro (F61). Están en la sección 7.

### c) Integración

`feature/location-closure` sale de `ee4df37` en el backend y de `a8ab115` en el frontend, y trae también los cambios de `organization` que el cierre necesitó: el tipo de documento del colaborador (LOC-R36), el dígito de control, el documento normalizado, la zona de la sede y las direcciones de las ubicaciones (D-79). No se integra sola: `feature/organization-closure` sale de ella, y la última versión de V39 (`aade406`) y la pantalla de LOC-R36 (`9842ea8`) están solo en esa rama. Se integró en `develop` dentro de `feature/organization-closure`, entera, el 18/10/2026, con la verificación final de arriba: el merge es `96b2d04` en el backend y `64e8565` en el frontend. El responsable confirmó que el despliegue terminó sin errores, así que V39 no encontró ninguna colisión. No consta que se corrieran antes del push los dos inventarios de solo lectura (`inventory-text-identifiers.sql` e `inventory-organization-checks.sql`). El registro está en el [orden de implementación](../orden-de-implementacion-millennium-mobility.md), paso 2.

**El buscador de listas largas y el padre de una división** (19/10/2026, solo frontend). `feature/searchable-select` agrega `mm-search-select` a `shared` y lo usa en los escalones de `mm-address-fields` y en la zona horaria del país y de la sede; después de una revisión adversarial —cuatro lentes, un verificador por lente—, cinco defectos menores corregidos. `feature/location-parent-division-search` corrige la búsqueda del padre: con `requireSelection`, Material no pasa lo escrito a `valueChanges`, así que se lee del campo. Verificación del buscador: 1 524 pruebas unitarias y la suite e2e completa, 797 pasan y 22 se omiten por proyecto. Integrados en `develop` el mismo día: merges `60cdb93` (placeholders), `bced274` (padre) y `dec9ddd` (buscador) en el frontend. ~~La búsqueda en el servidor todavía distingue tildes (convenciones de la API, §6.2, pendiente).~~ Desde el 30/10/2026 tampoco las distingue (D-142, V86; sección 6 b).

---

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿El dígito de control del NIT se escribe aparte o forma parte del número? ¿Cómo se modela el RFC de 12 y de 13? | [Revisión del 05/10/2026](../retroalimentacion/2026-10-05-location.md), "Lo que queda abierto" |
| ¿Se puede volver a cargar la lista oficial de divisiones desde la aplicación? | [Procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), apartado 5, pregunta 7 |
| ¿`taxRate` viaja como texto decimal? | [Convenciones](../convenciones-de-la-api-millennium-mobility.md) §14.2 (§13 fila 7) |
| ¿Un `422` dice qué regla falló, en un campo `rule`? | Convenciones §14.5 (§13 fila 15) |
| ¿Dos ediciones simultáneas del mismo registro se detectan con `If-Match`? | Convenciones §14.7 |
| ¿Las altas responden `201` con `Location`? | Convenciones §14.9 (§13 fila 8) |
| ¿Los códigos de error llevan guion bajo entre palabras (`GEO_DIVISION_*`)? | Convenciones §14.11 (§13 fila 13; auditoría del 17/10/2026, F62) |
| ¿Cómo obtiene el registro público, sin sesión, los tipos de documento de las personas? | Convenciones §14.16 (F61); antes de encender el registro (D-83) |
| ~~¿Cómo sabe `location` que un monto ya usa una moneda, para fijar sus decimales?~~ | Resuelto el 29/10/2026: se lo dice cada contexto que guarda montos, por el puerto `CurrencyUsage` (LOC-R05, sección 5) |
| ¿Qué tipo de tasa propone cada país para sus compras? | Diccionario §4.8; lo confirma Contabilidad antes del paso 7 |
