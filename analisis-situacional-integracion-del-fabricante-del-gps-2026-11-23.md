# Análisis situacional — la integración con el fabricante del GPS (Teltonika FOTA WEB) — ERP Millennium Mobility

**Estado:** versión 1, 23/11/2026. Responde a lo que el responsable definió ese día y es la base de lo que se construye a partir de §4. No decide nada que el responsable no haya decidido: lo que espera una decisión está en §5.

**Fuente:** el código de `develop` del backend (`b8440ac1`) y del frontend (`b198f213`), y la documentación de `master` (`98ae360`); el [análisis situacional del 22/11](analisis-situacional-areas-menu-y-ciclo-del-gps-2026-11-22.md) (frente C) y el [plan de áreas, menú y ciclo del GPS](plan-de-areas-menu-y-ciclo-del-gps-2026-11-22.md) 4.0 (fase 3b, Q21); [equipment.md](contextos/equipment.md); y la referencia oficial de la API de FOTA WEB, la especificación OpenAPI 3.1.1 que publica Teltonika en `https://api.teltonika.lt/swagger/latest/swagger.json` (versión `2026.11.23+ec202948`, leída ese día).

**Objetivo.** Decir qué falta para que el ciclo que el responsable describió funcione de punta a punta, y desde dónde se construye.

**Cómo leerlo.** Las rutas van abreviadas como en el análisis del 22/11: `B/` es `erp-millennium-mobility-backend/src/main/java/ky/millennium/mobility/erp/`, `T/` su carpeta de pruebas y `F/` `erp-millennium-mobility-frontend/src/app/`.

---

## 1. Resumen

### 1.1 Qué definió el responsable

La factura de equipos sigue la lógica de la de vehículos: así como la unidad entra con su VIN y sus datos, el GPS entra con su **IMEI, leído del QR**, y una **integración con el fabricante**, que solo pide el IMEI, trae lo demás: el serial, la marca y el modelo, el firmware y la SIM.

### 1.2 Veredicto

El ciclo ya está construido salvo **una pieza: el adaptador de Teltonika FOTA WEB**.

| Paso | Dónde | Estado |
|---|---|---|
| La factura de equipos crea cada equipo «Por ingresar», por modelo, como su comprobante | `procurement`, fase 2 y revisión 99 de `DESIGN.md` | Construido |
| «Ingreso de equipos» lee el QR —solo el IMEI, con su dígito verificador— con el lector o la cámara | `F/equipment/presentation/views/equipment-arrivals-view`, `POST /equipments/{id}/arrival-scan` | Construido |
| Con un fabricante, el GPS queda «Por completar» y se abre su pedido | `B/equipment`, V167 | Construido |
| Un proceso en segundo plano le pregunta al fabricante por lotes de IMEI y completa el equipo, o lo deja en «Revisar» con su causa | `DeviceCompletionTask`, puerto `DeviceRegistry` | Construido, con el proveedor `manual`, que no pregunta a nadie |
| **La llamada real a FOTA WEB** | `B/equipment/infrastructure/providers/teltonika` | **No existe** (Q21) |

Con el proveedor `manual`, el de todo entorno hoy, el escaneo pide también el serial y el equipo queda Disponible sin pasar por el fabricante (Q25). Por eso, para el responsable, el autocompletado «no existe» todavía.

### 1.3 Lo que cambia respecto de lo que se sabía

El plan dejaba el adaptador esperando la documentación, la cuenta y el token (Q21). La documentación ya no hace falta pedirla: **la especificación oficial es pública** y alcanza para construir el adaptador y probarlo contra respuestas grabadas. Lo que sigue faltando es solo lo que no se puede inventar: la cuenta de FOTA WEB de la empresa y su token, para encenderlo.

### 1.4 Tamaño

Uno a dos días: el adaptador, su prueba contra un servidor HTTP de la prueba, el chequeo de arranque y la documentación. Sin migraciones y sin cambios en el frontend: «Ingreso de equipos» ya cambia sola cuando `GET /equipments/arrival-settings` responde `completesWithManufacturer: true`.

---

## 2. Punto de partida

| Pieza | Dónde | Cómo está |
|---|---|---|
| El puerto | `B/equipment/application/internal/outboundservices/registry/DeviceRegistry.java` | `lookup(Set<String> imeis)` responde, por cada IMEI, `Found(serial, código del modelo, firmware, iccid, imsi)`, `NotFound`, `Retryable(código)` o `Refused(código)`; nunca lanza |
| El proveedor por omisión | `B/equipment/infrastructure/providers/ManualDeviceRegistry.java` | `manual`: no completa ni pregunta |
| El chequeo de arranque | `B/equipment/infrastructure/providers/DeviceRegistryProviderCheck.java` | Solo deja arrancar `manual`; `teltonika` está en `PLANNED` y detiene el arranque diciendo que espera Q21 |
| La configuración | `application.yaml`, `equipment.device-registry` | `provider`, `base-url`, `token`, `connect-timeout` (5 s) y `read-timeout` (10 s), ya cargados |
| El proceso | `DeviceCompletionTask` | Lotes de `EQUIPMENT_DEVICE_COMPLETION_BATCH_SIZE` (20), a lo sumo diez por corrida, cada 30 s; reintentos de 1 minuto a 1 hora, 8 intentos |
| La comparación | `DeviceCompletionCommandServiceImpl` | Disponible si cuadra; «Revisar» con `MODEL_MISMATCH` si el modelo del fabricante no es el «Código del fabricante» del modelo del equipo; `SIM_MISMATCH` si el ICCID difiere del escaneado o lo tiene otro equipo; `UNKNOWN_IMEI` si no conoce el IMEI |
| El precedente | `B/notifications/infrastructure/providers/sendgrid/SendGridEmailSender.java` y su prueba | `RestClient` con los tiempos del JDK, y una prueba contra un `HttpServer` local: el molde del adaptador |
| La guía | README del backend, «El fabricante de los GPS» | Los cinco pasos para encender el adaptador |

---

## 3. Hallazgos

**H1 · Lo único que falta es el adaptador.** Todo lo demás corre hoy con un `DeviceRegistry` de prueba en `DeviceCompletionIntegrationTests`: encontrado, otro modelo, otra SIM, IMEI desconocido, reintentable, agotado y la respuesta que llega después de un completado a mano.

**H2 · La API es pública y está especificada.** La especificación de FOTA WEB dice:

- **Dirección:** `https://api.teltonika.lt`, sin prefijo de versión y sin sandbox.
- **Autenticación:** `Authorization: Bearer <token>`. El token lo crea un usuario de la empresa en `https://fota.teltonika.lt`, y hereda su rol y su empresa. Necesita el permiso `Device.Read`.
- **`User-Agent` obligatorio,** con la empresa, el nombre de la aplicación y su versión. La especificación advierte que omitirlo puede llevar a dar de baja al usuario o a la empresa. El adaptador lo envía siempre, configurable.
- **La lista de equipos:** `GET /devices`, con el filtro `imei`, un arreglo de enteros, y `per_page` de 1 a 100 (25 por omisión). El sobre trae `current_page`, `last_page`, `per_page`, `total` y `data`. Un equipo trae `imei` (siempre), `serial`, `model`, `current_firmware`, `iccid` e `imsi`, todos los demás opcionales.

**H3 · Los IMEI se envían repetidos, no separados por comas.** El README del backend dice `GET /devices?imei=a,b,…`. La especificación declara `imei` como arreglo de enteros sin `style`, así que vale la forma por omisión de OpenAPI: `?imei=a&imei=b`. El servidor es ASP.NET Core, que lee esa forma. La forma con comas sale de un fragmento del wiki que no se pudo leer entero (está detrás de Cloudflare), y de un cliente de terceros. Se usa la forma de la especificación.

**H4 · El IMEI y el serial llegan como números.** `imei` y `serial` son `int64` en JSON. Un IMEI que empieza con 0 perdería ese dígito como número, así que el adaptador lo vuelve a escribir con sus 15 dígitos antes de compararlo con el pedido.

**H5 · El ICCID y el IMSI se pueden escribir a mano en FOTA WEB.** La especificación permite cambiarlos con `PUT`/`PATCH /devices/{imei}` y con `bulkUpdate`: pueden venir tipeados por alguien y no informados por el equipo, y pueden faltar. Es justo lo que ya resuelve `SIM_MISMATCH`, que compara con el ICCID escaneado y deja la decisión a una persona. No cambia nada.

**H6 · Los límites caben.** Teltonika pide no pasar de 100 pedidos por minuto, sin cabeceras de límite. El proceso hace a lo sumo diez llamadas por corrida, cada 30 s: unas 20 por minuto en el peor caso, con lotes de 20 IMEI. Un lote mayor que 100 se parte en llamadas de 100, porque `per_page` no pasa de ahí.

**H7 · Los errores.** Un token ausente o vencido responde `401` sin cuerpo (`application/problem+json`, ASP.NET Core); `400` y `403` están documentados sin forma de cuerpo; un IMEI que FOTA WEB no conoce simplemente no viene en la lista. Se leen como dice el puerto: un IMEI ausente es `NotFound`; 408, 429 y 5xx, `Retryable`; otro 4xx, `Refused` para todo el lote, y un 401 o un 403 lo dicen en el registro del servidor, para revisar el token.

**H8 · La marca y el modelo no se tipean: se confirman.** Para el responsable la integración «trae la marca y el modelo». Lo decidido (Q17, Q40) es que el modelo lo pone la línea de la factura, porque es lo que dice el comprobante y lo que se pagó, y el fabricante lo **confirma**: si dice otro, el equipo va a «Revisar» con `MODEL_MISMATCH`, y el modelo se corrige con «Anular y copiar». La marca es la del modelo. Es la misma idea con un control: nada que escribir, y una diferencia nunca pasa en silencio. No cambia nada, pero **cada modelo de GPS necesita su «Código del fabricante»** (Catálogos › Tipos de equipamiento), el nombre que usa FOTA WEB, como `FMB920` o `FMC130`. Sin él, la comparación no se hace (C-H15).

**H9 · Los GPS heredados guardan un número de IMEI en el serial.** En la lista de Equipos de `dev`, los GPS instalados que trajo la vinculación de la fase 4 («SIN MARCA REGISTRADA», «SIN MODELO REGISTRADO») muestran en «N° serie» números de 15 dígitos que empiezan como un IMEI de Teltonika (`864593…`), y el IMEI vacío. La vinculación (`InstalledGpsLinks`, D-286) toma el serial que se escribió en la instalación de la unidad, y lo guarda como serial. Si en los datos reales lo escrito ahí es el IMEI:

- el fabricante no los puede completar, porque se le pregunta por el IMEI y no lo tienen;
- un escaneo futuro de ese mismo GPS no chocaría con él, porque el índice único mira el IMEI.

Ocho de los diez que muestra esa lista no pasan el dígito verificador del IMEI, así que pueden ser datos de prueba. Hay que mirarlo en los datos reales antes de decidir (§5, P1).

**H10 · No se puede probar contra FOTA WEB sin la cuenta.** El adaptador se prueba contra un servidor HTTP de la prueba con respuestas armadas desde la especificación, como el de SendGrid. La primera llamada real se hace en `dev` con el token de la empresa, antes de encenderlo en producción.

---

## 4. Lo que se construye

| Pieza | Qué |
|---|---|
| `TeltonikaDeviceRegistry` | En `B/equipment/infrastructure/providers/teltonika`, con `@ConditionalOnProperty(provider = teltonika)`: `GET {base-url}/devices?imei=…&imei=…&per_page=…` con `Bearer`, `User-Agent`, `Accept: application/json` y los tiempos de `equipment.device-registry`; una llamada por cada 100 IMEI; el IMEI con sus 15 dígitos; `Found` con el serial, el modelo, el firmware, el ICCID y el IMSI; el IMEI ausente, `NotFound`; 408, 429 y 5xx, `Retryable` (`HTTP_<n>`); sin respuesta, `TIMEOUT` o `NETWORK_ERROR`; una respuesta que no se puede leer, `Retryable` (`INVALID_ANSWER`); otro 4xx, `Refused` (`HTTP_<n>`). Nunca lanza ni escribe un IMEI ni el token en el registro |
| Sus tipos | La página y el equipo de FOTA WEB, privados del paquete: no salen de él |
| `DeviceRegistryProviderCheck` | `teltonika` pasa de `PLANNED` a `PROVIDERS`; sigue pidiendo la dirección HTTPS y el token |
| La configuración | `equipment.device-registry.user-agent` (`EQUIPMENT_DEVICE_REGISTRY_USER_AGENT`), con un valor por omisión con la empresa, la aplicación y su versión |
| La prueba | `TeltonikaDeviceRegistryTests`, contra un `HttpServer` local: lo que envía; un lote con encontrados y desconocidos; el IMEI con un cero adelante; un lote de más de 100; 401, 403, 400, 408, 429 y 503; sin respuesta a tiempo; una respuesta ilegible |
| La documentación | El README del backend (el fabricante, encendido en cinco pasos, ahora tres), `.env.example`, `.do/app.yaml`, [equipment.md](contextos/equipment.md) y el plan (Q21 y la fase 3b) |

Lo que **no** se construye: ningún cambio de flujo, de pantalla ni de base. El proveedor de todo entorno sigue siendo `manual` hasta que el responsable cargue la cuenta.

---

## 5. Lo que espera al responsable

| # | Qué | Por qué |
|---|---|---|
| P1 | Mirar en los datos reales si lo que se escribió como «serial» del GPS de cada unidad es su IMEI (H9) | Si lo es, conviene mover esos números al IMEI de los GPS heredados, para que el fabricante los complete y un escaneo los reconozca. Es una corrección de datos con su propia decisión |
| P2 | Crear en FOTA WEB el token de integración de la empresa, con un usuario que lea sus equipos (`Device.Read`); se muestra una sola vez | Sin él, el adaptador no se enciende |
| P3 | Cargar el «Código del fabricante» de cada modelo de GPS del catálogo, tal como lo nombra FOTA WEB | Sin él, el modelo no se compara (H8) |
| P4 | Encenderlo: en App Platform, `EQUIPMENT_DEVICE_REGISTRY_BASE_URL=https://api.teltonika.lt`, el secreto `EQUIPMENT_DEVICE_REGISTRY_TOKEN` y `EQUIPMENT_DEVICE_REGISTRY_PROVIDER=teltonika`, primero en `dev` | La primera llamada real (H10) |

---

## 6. Registro

| Versión | Fecha | Cambio |
|---|---|---|
| 1 | 23/11/2026 | Primera versión, con la especificación oficial de FOTA WEB leída ese día |
