# Contexto `notifications` — ERP Millennium Mobility

**Actualizado:** 30/10/2026 (creado: decisión del responsable del 30/10/2026 de llevar el correo, el SMS y el push a un solo módulo de la plataforma, `notifications`, con un adaptador por proveedor; existe solo su arquitectura, en `develop` del backend, `2cf3f91d`, sin publicar todavía; la decisión espera su texto en el diccionario como D-143) · **Fuente:** diccionario de datos v1.33 (§2.5, §2.7, §11.3, §14, D-63, D-125) y el análisis del 30/10/2026.

**Cómo leerlo.** Este módulo está diseñado y no construido. Las secciones 1 a 3 dicen qué hará y qué no; la 4 lista sus reglas, todas ❌ hasta su primer caso de uso; la 6 dice qué existe hoy, que es solo la organización de sus paquetes. Sus tablas no están en el diccionario: entran con su primer caso, con sus columnas y su migración.

---

## 1. Qué es y qué no es

**Qué es.** Avisarle algo a una persona: el sistema decidió algo y se lo dice, en una sola dirección, por correo, SMS o push. Por ejemplo, el enlace de un solo uso para restablecer una contraseña (D-63), el aviso de que el SOAT de una unidad está por vencer (§11.3) o, cuando exista la aplicación del cliente, un aviso a su teléfono.

Es un módulo de la plataforma, como `documents` (D-125): lo usan varios contextos y no es de ninguno, y tiene su propio modelo —un pedido de notificación, para un destinatario, por un canal, con una plantilla, un estado y sus intentos—.

**Tablas que poseerá** (nombres previstos; entran al diccionario con el primer caso): `notifications`, el pedido con su estado; `notification_attempts`, cada intento de entrega con lo que respondió el proveedor; y `push_devices`, los dispositivos registrados para recibir push.

**Por qué un solo módulo, y no uno por proveedor ni uno de "integraciones".**

- SendGrid, Twilio y Firebase Cloud Messaging comparten el mismo modelo. Tres contextos repetirían tres veces a quién se envía, cómo se reintenta, cómo se registra y cómo se evita un envío doble. Y el proveedor es un detalle: cambiar SendGrid por otro servicio de correo cambia un adaptador, no crea ni borra un contexto.
- Un contexto de "integraciones" agruparía por tecnología —todo lo que habla con afuera— y no por dominio: dependería de todos, o todos de él. El mapa ya descartó ese camino para los archivos externos (§10: los adaptadores de archivos van en `fleet`, no en `shared.dataimport`).

**Por qué `notifications` y no `communications`.** El nombre fija el límite. `notifications` promete avisos en una sola dirección; `communications` atraería respuestas entrantes, conversaciones por WhatsApp, campañas e historial de contacto con clientes, otro modelo que, cuando llegue, merece su propio contexto.

| Parece de `notifications` | Es de | Por qué |
|---|---|---|
| Decidir a quién llega un aviso | El contexto que pide | Es una regla de su negocio: a quién llega el aviso de renovación lo decide `fleet` con la decisión abierta de §14 |
| El tipo de cambio que publica la SBS | `location` (§4.8, D-103) | El dato es suyo; el adaptador va en su infraestructura |
| La consulta de un RUC o un DNI | `organization` y `procurement` | Validan sus propios registros |
| El archivo de la SAT, el de la garantía mobiliaria, el Excel de stock | `fleet`, con `import_batches` | Mapa §6.5 y §10 |
| La telemetría del GPS | `fleet` | Es el kilometraje y el equipo de la unidad |
| Guardar un archivo | `documents` | Ya es suyo (§2.7) |
| Campañas de marketing, conversaciones de ida y vuelta | Otro contexto, cuando lleguen | Otro vocabulario: segmentos, preferencias, bajas, hilos |
| Los textos de atención y soporte de la plantilla de orden de compra | `procurement` (§9.2) | Se imprimen en el PDF; no se envían |

## 2. Ciclo de vida de cada agregado (previsto)

**Notificación** (`Notification`)

| Momento | Qué pasa |
|---|---|
| Nace | Un contexto la pide por `notifications::acl`, en la misma transacción del hecho que la origina. Si el hecho se revierte, la notificación no existe |
| Pendiente → enviada | Un proceso programado la envía por su canal. La respuesta del proveedor queda como un intento |
| Enviada → entregada o rebotada | El proveedor lo informa por su webhook, cuando lo tiene |
| Fallida | Agotó sus reintentos. Queda visible, no se pierde |
| Nunca pasa | Se borra, o se envía dos veces el mismo pedido |

**Dispositivo para push** (`PushDevice`): lo registra el frontend para la cuenta de la sesión; se retira cuando la persona cierra la sesión en ese dispositivo o cuando el proveedor rechaza su token. No se borra.

## 3. Flujos funcionales (previstos)

**F1. Restablecer la contraseña con un enlace** (D-63, pendiente desde el paso 3). Quien restablece pide el enlace; `iam` crea el token de un solo uso y pide una notificación de correo con la dirección de la cuenta y el enlace; `notifications` la envía por SendGrid. La persona abre el enlace y elige su contraseña: nadie más la conoce. Cambia D-63, §5.4 y §5.12 del diccionario.

**F2. El aviso de renovación** (paso 9). Depende de la decisión abierta de §14: a quién llega, con cuánta anticipación y por qué medio. Mientras no haya envío de correos, la forma más simple es una lista de documentos por vencer en Habilitación, que no necesita este módulo.

**F3. Un aviso a la aplicación del cliente.** Cuando exista esa aplicación, hoy fuera del alcance: push por Firebase Cloud Messaging a los dispositivos registrados de la cuenta del cliente.

**F4. Registrar un dispositivo para push.** El frontend obtiene el token de Firebase y lo registra para la cuenta de la sesión; al cerrar la sesión lo retira.

## 4. Reglas de negocio (previstas)

| ID | Regla | Fuente | Dónde se aplicará | Estado |
|---|---|---|---|---|
| NTF-R01 | Un contexto pide una notificación solo por `notifications::acl`; ninguno llama a un proveedor | D-143 (propuesta) | Spring Modulith | ❌ |
| NTF-R02 | Quien pide resuelve el destino: una dirección de correo, un teléfono o una cuenta para push. `notifications` no llama a ningún contexto de negocio | D-143; si leyera el correo de `iam` mientras `iam` le pide el enlace, habría un ciclo | Contrato | ❌ |
| NTF-R03 | El pedido se guarda en la transacción del hecho que lo origina y lo envía después un proceso: un proveedor caído no revierte el hecho ni pierde el aviso | D-143 | Servicio | ❌ |
| NTF-R04 | Un pedido no se envía dos veces: cada uno lleva una llave de idempotencia única | D-143 | Base | ❌ |
| NTF-R05 | Un envío fallido se reintenta con una espera creciente y un tope; después del último queda fallido y visible | D-143 | Servicio | ❌ |
| NTF-R06 | Un puerto por canal (`EmailSender`, `SmsSender`, `PushSender`), nunca por proveedor; el cliente y los tipos de un proveedor no salen de su paquete de `infrastructure/providers` | D-143 | Arquitectura | ❌ |
| NTF-R07 | Los webhooks de los proveedores son las únicas rutas del sistema sin permiso: cada uno verifica la firma de su proveedor antes de leer el cuerpo | D-143; excepción a D-95 que se decide al construirlos | Servicio | ❌ |
| NTF-R08 | Las llaves de los proveedores viven en variables de entorno, nunca en la base ni en el repositorio. En desarrollo y en pruebas, un adaptador registra lo que se enviaría y no llama a nadie | D-143 | Configuración | ❌ |
| NTF-R09 | Las plantillas están en el código, con los `HtmlTemplates` de `shared`, y sus textos en los bundles | D-143 | Infraestructura | ❌ |
| NTF-R10 | Una notificación de una empresa lleva su `company_id` y Row Level Security; una de una cuenta, como el enlace para restablecer la contraseña, va sin empresa y la ve solo el alcance `SYSTEM` | §2.5, como la excepción de `generated_documents` (§2.7) | Base | ❌ |
| NTF-R11 | Nada se borra. El cuerpo del mensaje se conserva por un plazo acotado, porque lleva datos personales, y un SMS que no sea transaccional exige el consentimiento de la persona | §2.5; Ley 29733 | Servicio | ❌ |

## 5. Relación con otros contextos

**Lo que necesita de otros.** Solo `documents::acl`, para adjuntar un archivo guardado —por ejemplo, el PDF de una orden enviada—. Su `package-info.java` lo declara: `allowedDependencies = {"documents::acl"}`.

**Quién lo usará.** `iam`, para el enlace de restablecimiento (F1); `fleet`, para el aviso de renovación si §14 decide un medio (F2); `procurement`, si el negocio pide que la orden enviada viaje por correo al proveedor.

**Por qué no crea ciclos.** La flecha va de quien pide hacia `notifications` (mapa §4), y `notifications` solo llama a `documents`, que no llama a nadie. El destino lo resuelve quien pide, y los dispositivos de push son de este módulo, que los registra por su propia ruta.

**Por qué no escucha eventos.** Se evaluó que `notifications` escuchara los eventos de los contextos —la contraseña pedida, el documento por vencer—. Lo haría depender de todos y llevaría a este módulo las reglas de cada negocio: a quién se avisa y qué se dice. Pedir por su contrato deja esas reglas donde están.

## 6. Estado y pendientes para cerrar

**Construido el 30/10/2026: solo la arquitectura** (backend `2cf3f91d`, en `develop`, sin publicar). El módulo `ky.millennium.mobility.erp.notifications` tiene un `package-info.java` en cada paquete, con lo que va en él, siguiendo la estructura de la skill `backend-module`: el raíz declara el módulo; `interfaces/acl` es su contrato, con `@NamedInterface("acl")`; los puertos por canal en `application/internal/outboundservices/channels`, el despachador en `application/internal/scheduledtasks` y un paquete por proveedor en `infrastructure/providers` —`sendgrid`, `twilio`, `fcm` y `capturing`, el de desarrollo y pruebas—. No tiene clases, tablas, rutas ni beans. Con él pasan `ModularityTests`, `LayerDependencyTests`, `NoDeleteEndpointsTests` y el arranque de la aplicación.

**El frontend no tiene carpeta todavía.** Angular no tiene un equivalente de `package-info.java`, y una carpeta vacía no queda en git. Llegará como `src/app/notifications`, con sus cuatro capas (skill `frontend-feature`), cuando haya una pantalla que la necesite: el registro de un dispositivo para push o la lista de notificaciones enviadas.

| Pendiente | Cuándo |
|---|---|
| Las tablas en el diccionario, su migración con Row Level Security, los permisos `notificaciones.*` y las rutas en las convenciones de la API | Con el primer caso |
| La cuenta de cada proveedor: el dominio `millennium.ky` autenticado en SendGrid (SPF y DKIM), los números de Twilio por país y el proyecto de Firebase | Antes del primer envío real de cada canal |
| Las variables de entorno de cada proveedor en App Platform | Con el primer caso |

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿Cuál es el primer caso: el enlace para restablecer la contraseña (D-63) o el aviso de renovación? | Decisión del responsable; orden de implementación, pendientes transversales |
| ¿Un solo remitente para el grupo (`millennium.ky`) o uno por empresa o por marca comercial? | Decisión del responsable, antes del primer correo |
| ¿A quién llega el aviso de renovación, con cuánta anticipación y por qué medio? | Diccionario §14, antes del paso 9 |
| ¿Cuánto tiempo se conserva el cuerpo de un mensaje? | Diccionario, con las tablas del módulo |
| ¿Cómo se exponen los webhooks sin permiso? | Diccionario y convenciones de la API, al construirlos |
