# Contexto `notifications` — ERP Millennium Mobility

**Fuente:** diccionario de datos v1.61 (§2.5, §2.7, §11.3, §14, D-63, D-125, D-143, D-242, D-243 y D-255 a D-258) y el código de `develop`. Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** El canal de correo está construido, con dos casos de `iam`: el enlace para restablecer la contraseña ([iam.md](iam.md), F8 y F13) y el código para activar una cuenta (F14). El SMS, el push, los webhooks y las pantallas están diseñados y sin código: el documento los marca como previstos. Las columnas de sus tablas están en el diccionario (§2.7, D-242), y la configuración de cada entorno, en el README del backend.

---

## 1. Qué es y qué no es

**Qué es.** Avisarle algo a una persona: el sistema decidió algo y se lo dice, en una sola dirección, por correo, SMS o push. Por ejemplo, el enlace de un solo uso para restablecer una contraseña (D-63, D-243), el código para activar una cuenta nueva (D-255), el aviso de que el SOAT de una unidad está por vencer (§11.3) o, cuando exista la aplicación del cliente, un aviso a su teléfono.

Es un módulo de la plataforma, como `documents` (D-125): lo usan varios contextos y no es de ninguno, y tiene su propio modelo —un pedido de notificación, para un destinatario, por un canal, con una plantilla, un estado y sus intentos—.

**Tablas que posee**

| Tabla | Qué guarda |
|---|---|
| `notifications` | La bandeja de salida: un pedido por fila, con su canal, su tipo, su destinatario, su plantilla, sus parámetros con la lista de los sensibles, su estado (`pendiente`, `enviada`, `fallida` o `vencida`), sus intentos y su vencimiento (§2.7, D-242). Sin empresa: la ve solo el alcance `SYSTEM` (NTF-R10) |
| `notification_attempts` | Cada intento de entrega, con su resultado y la respuesta resumida del proveedor; de solo inserción (§2.7, D-242) |
| `push_devices` | Prevista: los dispositivos registrados para recibir push, con el canal push |

**Por qué un solo módulo, y no uno por proveedor ni uno de "integraciones".**

- SendGrid, Twilio y Firebase Cloud Messaging comparten el mismo modelo. Tres contextos repetirían tres veces a quién se envía, cómo se reintenta, cómo se registra y cómo se evita un envío doble. Y el proveedor es un detalle: cambiarlo cambia un adaptador, no crea ni borra un contexto, como el relay SMTP de Google Workspace junto a SendGrid (F7, D-257).
- Un contexto de "integraciones" agruparía por tecnología —todo lo que habla con afuera— y no por dominio: dependería de todos, o todos de él. El mapa ya descartó ese camino para los archivos externos (§10).

**Por qué `notifications` y no `communications`.** El nombre fija el límite. `notifications` promete avisos en una sola dirección; `communications` atraería respuestas entrantes, conversaciones por WhatsApp, campañas e historial de contacto con clientes, otro modelo que, cuando llegue, merece su propio contexto.

| Parece de `notifications` | Es de | Por qué |
|---|---|---|
| Decidir a quién llega un aviso y qué dice | El contexto que pide | Es una regla de su negocio: a quién llega el aviso de renovación lo decide `fleet` con la decisión abierta de §14; a quién llega el enlace o el código, `iam` |
| El enlace para restablecer la contraseña y el código para activar una cuenta, con su vencimiento | `iam` (D-243, D-255) | `notifications` los recibe como parámetros sensibles y los olvida al enviarlos (NTF-R11) |
| El tipo de cambio que publica la SBS | `location` (§4.8, D-103) | El dato es suyo; el adaptador va en su infraestructura |
| La consulta de un RUC o un DNI | `organization` y `procurement` | Validan sus propios registros |
| El archivo de la SAT, el de la garantía mobiliaria, el Excel de stock, la telemetría del GPS | `fleet` | Mapa §6.5 y §10: son de la unidad |
| Guardar un archivo | `documents` | Ya es suyo (§2.7) |
| Campañas de marketing, conversaciones de ida y vuelta | Otro contexto, cuando lleguen | Otro vocabulario: segmentos, preferencias, bajas, hilos |
| Los textos de atención y soporte de la plantilla de orden de compra | `procurement` (§9.2) | Se imprimen en el PDF; no se envían |

## 2. Ciclo de vida de cada agregado

**Notificación** (`Notification`)

| Momento | Qué pasa |
|---|---|
| Nace | `pendiente`, en la transacción de quien la pide (NTF-R01, NTF-R03): si el hecho se revierte, no existe. Su destinatario va en minúsculas y sin espacios, y la escribe el módulo con el alcance `SYSTEM` (NTF-R10) |
| Se reclama | El despachador la toma y la arrienda antes de llamar a ningún proveedor (NTF-R04) |
| Pendiente → enviada | Se arma con su plantilla (NTF-R09, NTF-R14) y se entrega fuera de toda transacción. Aceptada, queda `enviada`, con su momento y el identificador del proveedor, y olvida sus sensibles (NTF-R11) |
| Sigue pendiente | Un rechazo que puede pasar después espera y se reintenta (NTF-R05) |
| Fallida | Agotó sus intentos, el rechazo es definitivo o no se pudo armar (NTF-R05): queda visible, y olvida sus sensibles |
| Vencida | Llegó su vencimiento sin enviarse (NTF-R12) |
| Enviada → entregada o rebotada | Prevista: el proveedor lo informa por su webhook (NTF-R07) |
| Nunca pasa | Se borra (NTF-R11). Dos corridas envían la misma fila. El envío es **al menos una vez**: una notificación nunca se pierde, y puede salir dos veces (NTF-R04) |

Cada intento queda en `notification_attempts` (NTF-R05).

**Dispositivo para push** (`PushDevice`, previsto): lo registra el frontend para la cuenta de la sesión; se retira cuando la persona cierra la sesión en ese dispositivo o cuando el proveedor rechaza su token. No se borra.

## 3. Flujos funcionales

**F1. Restablecer la contraseña con un enlace** (D-63, D-243). La persona que olvidó su contraseña pide el enlace, o un administrador se lo envía ([iam.md](iam.md), F13 y F8). `iam` emite el token y, en la misma transacción, pide el correo: tipo `iam.password-reset`, plantilla `password-reset`, la dirección de la cuenta, el enlace como parámetro sensible, quién lo pidió y el vencimiento del enlace (NTF-R02, NTF-R12). Después de confirmar, el despachador lo envía (F2). El correo (NTF-R09) dice quién lo pidió, da el enlace, que sirve una vez, cuándo vence, en la hora de Lima, y qué hacer si no se esperaba.

**F2. Despachar la bandeja de salida.** Un proceso programado, en una sola instancia y con el alcance `SYSTEM`: vence las pendientes cuyo momento pasó (NTF-R12); reclama hasta 20 por vez (NTF-R04); envía cada una fuera de toda transacción, así un proveedor lento no retiene candados ni conexiones; y anota cada respuesta (NTF-R05). El registro del servidor nombra una notificación por su identificador público y su tipo, nunca por su dirección ni sus parámetros. Un vigilante avisa cuando los fallos de la última hora pasan su umbral (NTF-R20).

**F3. El aviso de renovación** (paso 9, previsto). Depende de la decisión abierta de §14: a quién llega, con cuánta anticipación y por qué medio. Mientras no haya envío de correos, la forma más simple es una lista de documentos por vencer en Habilitación, que no necesita este módulo.

**F4. Un aviso a la aplicación del cliente** (previsto). Cuando exista esa aplicación, hoy fuera del alcance: push por Firebase Cloud Messaging a los dispositivos registrados de la cuenta del cliente.

**F5. Registrar un dispositivo para push** (previsto). El frontend obtiene el token de Firebase y lo registra para la cuenta de la sesión; al cerrar la sesión lo retira.

**F6. Activar una cuenta con un código** (D-255, D-258). Un administrador da acceso a un colaborador con un correo sin cuenta, le corrige el correo o le reenvía el código, o la persona pide uno nuevo ([iam.md](iam.md), F5, F14, F15 y F16). `iam` genera el código y, en la misma transacción, pide el correo: tipo `iam.account-activation`, plantilla `account-activation`, la dirección de la cuenta, el código como parámetro sensible, el enlace a la pantalla de activación con la dirección ya escrita —que no es sensible, porque no abre nada sin el código— y el vencimiento del código (NTF-R12; [iam.md](iam.md), IAM-R90). Después de confirmar, el despachador lo envía (F2). El correo (NTF-R09) da el código, que sirve una sola vez, y cuándo vence, en la hora de Lima; avisa que hará falta el número del documento que registró la empresa; recuerda que nunca se pedirá el código por teléfono, chat ni correo, y pide ignorarlo si no se esperaba.

**F7. Cambiar de proveedor de correo** (D-257). SendGrid y el relay SMTP de Google Workspace quedan configurados a la vez, con el mismo subdominio remitente, `notifications.millennium.ky`, autenticado en los dos: pasar de uno a otro es un solo valor del entorno (NTF-R18), que vuelve a desplegar en minutos. El subdominio no tiene casillas: el remitente es una dirección suya, y la respuesta va a una casilla real de `millennium.ky` (NTF-R15). Basta un proveedor real; los dos son la reserva recomendada. Los correos pendientes salen por el proveedor nuevo; los que ya quedaron `fallida` no se recuperan, porque sus valores sensibles se borraron (NTF-R11), y la persona pide otro código o enlace. La configuración de cada proveedor, el subdominio y el simulacro están en el README del backend, «Cambiar de proveedor de correo».

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md).

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| NTF-R01 | Un contexto pide una notificación solo por el contrato del módulo; ninguno llama a un proveedor. El pedido se toma solo dentro de la transacción de quien pide y solo con una plantilla que existe | D-143, D-242 | Servicio | `ModularityTests`, `ForeignAclStaysInTranslationLayerTests`, `NotificationDispatchTaskTests` | ✅ |
| NTF-R02 | Quien pide resuelve el destino: una dirección de correo, un teléfono o una cuenta para push. `notifications` no llama a ningún contexto de negocio: si leyera el correo de `iam` mientras `iam` le pide el enlace, habría un ciclo | D-143 | Servicio | `ModularityTests` | ✅ |
| NTF-R03 | El pedido se guarda en la transacción del hecho que lo origina y lo envía después un proceso: un proveedor caído no revierte el hecho ni pierde el aviso, y la persona que pidió un enlace o un código no espera a ningún proveedor | D-143, D-242 | Servicio | `NotificationDispatchTaskTests` | ✅ |
| NTF-R04 | Cada pedido es una fila, y el despachador cuenta y arrienda el intento por 2 minutos en una transacción que confirma antes de llamar al proveedor, así dos corridas nunca envían la misma fila. El envío es al menos una vez: si la instancia se detiene después de que el proveedor tomó el correo y antes de anotarlo, el arriendo vence y puede salir dos veces. Una llave de idempotencia queda candidata, para un caso en que un envío doble importe | D-143, D-242 | Base, Dominio, Servicio | `NotificationTests`, `NotificationDispatchTaskTests` | ✅ |
| NTF-R05 | Un envío fallido que puede pasar después se reintenta con una espera de 1 minuto que se duplica hasta 1 hora, hasta 10 intentos, unas 4 horas de reintentos; un correo vence igual con su código o su enlace. Después del último queda `fallida` y visible. Por SendGrid se reintentan `408`, `429`, `5xx` o la falta de respuesta, y falla en el acto otro `4xx`, también una clave equivocada (`HTTP_401`). Por SMTP se reintentan una respuesta `4xx` (`SMTP_<código>`), un servidor que no se alcanza (`NETWORK_ERROR`) —también un certificado que no nombra el host— y otra falla del envío (`SMTP_ERROR`); fallan en el acto las credenciales rechazadas (`SMTP_AUTH`), una respuesta `5xx` (`SMTP_<código>`) y un mensaje que no se puede armar (`SMTP_MESSAGE_ERROR`). Una plantilla que no se arma falla en el acto. Cada intento queda en `notification_attempts` | D-143, D-242, D-257, D-258 (M9) | Dominio, Servicio | `NotificationTests`, `NotificationDispatchTaskTests`, `SendGridEmailSenderTests`, `SmtpEmailSenderTests`, `SmtpEmailSenderDeliveryTests` | ✅ |
| NTF-R06 | Un puerto por canal —el correo y, previstos, el SMS y el push—, nunca por proveedor; el cliente y los tipos de un proveedor no salen de su adaptador, y el despachador recibe solo un recibo. SendGrid se llama por HTTP, sin su SDK, que traería librerías que chocan con las de la aplicación y no distingue lo que se reintenta de lo final (D-256). `smtp` es otro adaptador del mismo puerto (NTF-R16) | D-143, D-242, D-256, D-257 | Servicio | `LayerDependencyTests`, `SendGridEmailSenderTests` | ✅ |
| NTF-R07 | Los webhooks de los proveedores son las únicas rutas del sistema sin permiso: cada uno verifica la firma de su proveedor antes de leer el cuerpo | D-143; excepción a D-95 que se decide al construirlos | Servicio | — | ❌ |
| NTF-R08 | Las llaves de los proveedores viven en el entorno, nunca en la base ni en el repositorio. Fuera de `dev` la aplicación no arranca sin el proveedor (`capturing`, `sendgrid` o `smtp`) ni el remitente, con `sendgrid` sin su llave, con `smtp` sin lo que necesita (NTF-R18), ni con una dirección de respuesta que no es una dirección (NTF-R15); el mensaje dice qué falta, nunca una llave. La contraseña del relay SMTP nunca llega al registro. En desarrollo y en pruebas, `capturing` guarda los últimos 100 mensajes en memoria y los escribe en el registro, sin llamar a nadie, y de sus imágenes registra solo el nombre; el enlace y el código se ven solo en `dev`, y bajo el perfil `prod` nunca, diga lo que diga la configuración | D-143, D-242, D-256, D-257 | Servicio | `ProvidersOfEmailTests` | ✅ |
| NTF-R09 | Las plantillas están en el código, con sus textos en los archivos de mensajes. Un parámetro terminado en `At` se ofrece también como `<nombre>Local`, en `dd/MM/yyyy HH:mm` y en `America/Lima` (§2.6). Hay dos plantillas, `password-reset` y `account-activation`, con el mismo diseño: tablas con estilos en línea, los colores del frontend y el logo en la cabecera (NTF-R14) | D-143, D-242, D-255, D-256 | Servicio | `NotificationDispatchTaskTests`, `ThymeleafEmailTemplatesTests` | ✅ |
| NTF-R10 | Una notificación de una empresa lleva su empresa y Row Level Security; una de una cuenta, como el enlace o el código de `iam`, va sin empresa y la ve solo el alcance `SYSTEM`. El módulo escribe su fila con ese alcance, dentro de la transacción de quien pide, y el despachador corre con él | §2.5, como la excepción de `generated_documents` (§2.7); D-242 | Base, Servicio | `RowLevelSecurityMigrationTests` | ✅ |
| NTF-R11 | Nada se borra, y el cuerpo del mensaje no se guarda: la fila guarda su plantilla y sus parámetros, y los sensibles —como el enlace o el código— los olvida al enviarse, al fallar o al vencer; una fila que ya no está pendiente no puede tener ninguno. La respuesta del proveedor se guarda resumida, con los valores sensibles ocultos, y la llave nunca llega al registro. La aplicación no borra, y de una notificación solo cambia su estado, sus intentos, su próximo intento, su envío, el identificador del proveedor y sus parámetros. Un SMS que no sea transaccional exigirá el consentimiento de la persona | §2.5, D-242; Ley 29733 | Base, Dominio | `RowLevelSecurityMigrationTests`, `NotificationTests`, `NotificationAuditTests`, `HandOverDatabaseScriptTests`, `SendGridEmailSenderTests` | ✅ |
| NTF-R12 | Una notificación vence en su momento si no salió: queda `vencida`, sin enviarse, y olvida sus sensibles. La del enlace para restablecer la contraseña vence con el enlace, y la del código para activar una cuenta, con el código | D-242, D-243, D-255 | Base, Dominio, Servicio | `NotificationTests`, `NotificationDispatchTaskTests` | ✅ |
| NTF-R13 | El cuerpo que se envía a SendGrid tiene exactamente las llaves que SendGrid documenta: una `personalization` por destinatario, así nadie ve la dirección de otro; `from` con su nombre; `reply_to` solo si el entorno lo nombra (NTF-R15); `subject`; `content` con el texto antes del HTML; `attachments` solo si hay imágenes (NTF-R14); `custom_args.notification_id`, el identificador público de la notificación; y `tracking_settings` con los clics y las aperturas apagados, así el enlace llega tal cual y ninguna imagen avisa que se abrió. El registro del servidor nunca lleva los cuerpos. Por SMTP el mensaje lleva lo mismo (NTF-R17) | D-242, D-256 | Servicio | `SendGridMailBuilderTests`, `SendGridEmailSenderTests` | ✅ |
| NTF-R14 | Ningún correo carga una imagen de la red: las imágenes van dentro del mensaje, en línea, y el HTML las nombra con `cid:`. Todo correo lleva el logo de la marca, leído una vez al arrancar, como `cid:millennium-logo`: SendGrid lo recibe en Base64 como adjunto en línea con su identificador de contenido, y por SMTP va como una parte en línea con ese `Content-ID`. Ocultar los valores sensibles de un mensaje conserva sus imágenes | D-256, D-257 | Servicio | `ThymeleafEmailTemplatesTests`, `SendGridMailBuilderTests`, `ProvidersOfEmailTests` | ✅ |
| NTF-R15 | La dirección de respuesta es opcional: sin ella, el correo no lleva ninguna y las respuestas van al remitente; con una, viaja en cada correo, sin nombre, como `reply_to` en SendGrid y como `Reply-To` por SMTP. Un valor que no es una dirección —sin una arroba con algo antes— no deja arrancar la aplicación, y el mensaje lo nombra | D-256 | Servicio | `ProvidersOfEmailTests`, `SendGridMailBuilderTests`, `SendGridEmailSenderTests` | ✅ |
| NTF-R16 | El proveedor `smtp` envía por un relay SMTP con su propio remitente y no con la configuración automática de correo de Spring, así no aparece otro remitente ni un indicador de salud del correo; sus tipos no salen de su adaptador. La conexión siempre va cifrada: STARTTLS obligatorio en el puerto 587, o TLS desde el inicio en el 465; no hay texto plano, y el certificado del servidor tiene que nombrar su host. Entra al relay con una cuenta si tiene usuario y contraseña, que van juntos; conectar espera 5 segundos, y cada respuesta y cada escritura, 10. Su configuración nunca escribe la contraseña | D-257 | Servicio | `SmtpEmailSenderTests`, `SmtpEmailSenderDeliveryTests` | ✅ |
| NTF-R17 | Por SMTP el mensaje lleva lo mismo que el cuerpo de SendGrid (NTF-R13), en UTF-8, con el texto y el HTML como alternativas, las imágenes en línea (NTF-R14), la dirección de respuesta como `Reply-To` (NTF-R15) y el identificador público de la notificación en la cabecera `X-MM-Notification-Id`, el equivalente de `custom_args`; nada de seguimiento. Un mensaje aceptado guarda su `Message-ID` como identificador del proveedor. Toda respuesta resumida oculta los valores sensibles del mensaje | D-256, D-257 | Servicio | `SmtpEmailSenderTests`, `SmtpEmailSenderDeliveryTests` | ✅ |
| NTF-R18 | El arranque revisa solo lo que necesita el proveedor elegido, así SendGrid y SMTP pueden quedar configurados a la vez y cambiar de uno a otro es un solo valor (F7). Con `smtp`, la aplicación no arranca sin host, con un puerto fuera de 1 a 65535, con una protección que no es `starttls` ni `ssl`, o con solo uno de usuario y contraseña; el mensaje dice qué falta o qué está mal y qué variable lo fija, nunca un valor. Cada proveedor existe solo cuando el entorno lo nombra | D-257 | Servicio | `ProvidersOfEmailTests` | ✅ |
| NTF-R19 | El contrato dice si un correo de este entorno llega a la persona (`deliversEmail`): sí con `sendgrid` y `smtp`; con `capturing`, solo fuera del perfil `prod`, donde el registro del servidor muestra los códigos y los enlaces a quien desarrolla. Quien pide lo pregunta antes de escribir nada, y lo que solo funciona por correo responde `503 EMAIL_DELIVERY_UNAVAILABLE` sin cambiar nada ([iam.md](iam.md), IAM-R103); el pedido mismo no cambia: una notificación se guarda igual con cualquier proveedor | D-258 (M4) | Servicio | `ProvidersOfEmailTests`, `EmailDeliveryUnavailableIntegrationTests` | ✅ |
| NTF-R20 | Cada intento de envío cuenta en la métrica `notifications.delivery.attempts`, por canal, proveedor, resultado y código de error, nunca con una dirección ni un valor del correo. Un vigilante, en una sola instancia, cuenta cada cierto tiempo los intentos fallidos de la última hora y, desde un umbral —con uno menor que 1 la aplicación no arranca—, escribe una sola línea `ERROR` por revisión, con los conteos por código de error y el más reciente, nunca una dirección ni un valor. No toca el chequeo de salud, a propósito: App Platform reinicia la aplicación cuando ese chequeo falla, y un proveedor caído no debe sacarla de servicio. Que la línea llegue como alerta pide reenviar los registros de App Platform a un servicio que avise (sección 6) | D-258 (M10) | Servicio | `MicrometerDeliveryMetricsTests`, `NotificationFailureWatchTaskTests`, `NotificationDispatchTaskTests` | ✅ |

## 5. Relación con otros contextos

**El contrato.** `NotificationService`, la interfaz pública del módulo (`notifications::acl`), con dos miembros: `requestEmail`, que guarda un correo en la transacción de quien pide (NTF-R01, NTF-R03), y `deliversEmail`, que dice si un correo de este entorno llega a la persona (NTF-R19).

**Quién lo usa.** Solo `iam`, para el enlace (F1) y el código de activación (F6), y para saber si un correo llega antes de lo que solo funciona por correo ([iam.md](iam.md), IAM-R103). **Quién lo usará:** `fleet`, para el aviso de renovación si §14 decide un medio (F3); `procurement`, si el negocio pide que la orden enviada viaje por correo al proveedor.

**Lo que necesita de otros.** Solo `documents::acl`, para adjuntar un archivo guardado, como el PDF de una orden enviada; ningún correo de hoy adjunta nada. Del núcleo `shared`, el alcance `SYSTEM` (NTF-R10), la ejecución en una sola instancia y las plantillas HTML. Hacia afuera, un proveedor por su adaptador, que elige el entorno (NTF-R06).

**Por qué no crea ciclos ni escucha eventos.** La flecha va de quien pide hacia `notifications` (mapa §4), que solo llama a `documents`, y el destino lo resuelve quien pide (NTF-R02). Escuchar los eventos de los contextos —la contraseña pedida, el documento por vencer— lo haría depender de todos y llevaría a este módulo las reglas de cada negocio: a quién se avisa y qué se dice.

## 6. Estado y pendientes para cerrar

**Estado.** Está construido el canal de correo, con los proveedores `capturing`, `sendgrid` y `smtp`, las plantillas `password-reset` y `account-activation` y sus dos casos de `iam`. El SMS, el push y los webhooks tienen solo su lugar reservado. El módulo no tiene permisos, rutas ni carpeta en el frontend: llegan con su primera pantalla, como el registro de un dispositivo para push o la lista de notificaciones enviadas.

| Pendiente | Cuándo |
|---|---|
| La cuenta de cada proveedor: el subdominio de envío `notifications.millennium.ky`, autenticado en el proveedor de correo elegido (SPF, DKIM y DMARC); los números de Twilio por país y el proyecto de Firebase | El responsable. El correo, antes del primer envío real; los demás, antes del primer envío de su canal |
| Un proveedor real de correo en el entorno desplegado, SendGrid o el relay SMTP de Google Workspace, con el subdominio autenticado para él. Sin él, ningún correo sale y lo que solo funciona por correo responde `503 EMAIL_DELIVERY_UNAVAILABLE` (NTF-R19; [iam.md](iam.md), sección 6 b, fila 16) | El responsable, en el proveedor elegido, en Cloudflare y en App Platform |
| El segundo proveedor, como reserva recomendada y no como requisito (D-257; F7) | El responsable, en Google Workspace, en Cloudflare y en App Platform |
| Un simulacro del cambio de proveedor, una vez configurados los dos (F7) | Recomendado, una vez configurado (sección 7) |
| Que la línea `ERROR` del vigilante llegue como alerta a alguien: reenviar los registros de App Platform a un servicio que avise (NTF-R20) | El responsable, en App Platform (D-258) |
| Los webhooks de los proveedores, con su firma (NTF-R07), y con ellos los estados entregada y rebotada. Con `smtp` no hay webhook: un rebote llega como un correo a la cuenta que envía (D-257) | Cuando se decida qué hacer con un rebote |
| La llave de idempotencia, si un caso necesita que un correo nunca salga dos veces (NTF-R04) | Candidata: cuando llegue ese caso |
| El SMS y el push, con sus tablas, sus puertos y sus proveedores | Con su primer caso |

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿Un solo remitente para el grupo (`millennium.ky`) o uno por empresa o por marca comercial? | Decisión del responsable, antes del primer correo real. Hoy hay un solo remitente, en el subdominio `notifications.millennium.ky`, una sola dirección de respuesta y un solo logo, el de Millennium (D-257, F7) |
| ¿A quién llega el aviso de renovación, con cuánta anticipación y por qué medio? | Diccionario §14, antes del paso 9 |
| ¿Cómo se exponen los webhooks sin permiso? | Diccionario y convenciones de la API, al construirlos |
| ¿Cuándo se hace el simulacro del cambio de proveedor (F7), y quién lo repite y cada cuánto? | Decisión del responsable, una vez configurados SendGrid y el relay de Google Workspace (D-257) |
| ¿Qué se hace con los rebotes? SendGrid los informaría por su webhook, que no está construido (NTF-R07); por `smtp` llegan como correos a la cuenta que envía, y el sistema no se entera | Decisión del responsable, antes de depender de que los avisos lleguen (D-257); con ella, los webhooks |
