# Plan de cierre de la revisión del 31/10/2026

**Alcance:** los hallazgos de la revisión completa de los tres repositorios del 31/10/2026, validados ese día contra el código:
- el inicio de sesión;
- el largo de la contraseña;
- los decimales de los montos;
- la traducción de las fachadas en `procurement`;
- dónde vive la regla del camino de estado;
- el tamaño de los servicios de comandos;
- la navegación del store de `iam`;
- los textos desactualizados de la documentación y de las skills.

Incluye un refuerzo: que al backend solo se llegue por Cloudflare.

**Base:** backend `1e207eb7` (`develop`, V1 a V86), frontend `6645341c` (`develop`) y documentación `853ca24` (`master`).

**Estado:** versión 3.1, **aprobada por el responsable el 31/10/2026** («Apliquemos lo recomendado»). Suma a la versión 3 los cuatro ajustes de la sección 14. **Construido e integrado el 31/10/2026**: la sección 15 dice los commits de cada slice, las pruebas, los desvíos y lo que queda.
- **Versión 2:** incorporó la auditoría adversarial del plan (sección 13).
- **Versión 3:** fija las decisiones recomendadas tras revisarlas con criterio crítico (sección 14). El freno cuenta por cuenta y por dispositivo (opción B de DP-1), y el refuerzo del secreto de origen pasa a ser parte del plan, fuera de la ruta crítica (DP-13).

El responsable aprueba la sección 3 antes de empezar cada fase que depende de ella. Las propuestas para el diccionario (DP-11) no se escriben en él hasta que el responsable las apruebe.

---

## 1. Criterios con los que se armó el plan

1. **Primero lo que deja entrar a alguien.** El inicio de sesión es el único hallazgo de seguridad. Su primera mitad, la comparación de relleno (S1), no cambia ningún contrato ni necesita decisión: sale sola y antes que todo.
2. **Un freno que no se pueda usar para cerrar cuentas ajenas, y que frene también un ataque repartido entre muchas IP.** Por eso cuenta por cuenta para quien llega sin historia, y por dispositivo para quien ya entró (OWASP, *Slow Down Online Guessing Attacks with Device Cookies*). Así no depende de ninguna IP.
3. **Defensa en profundidad sin trabar el cierre.** El refuerzo del secreto de origen hace efectivas las protecciones de Cloudflare y oculta el backend, pero el freno no depende de él: van en paralelo.
4. **El origen del defecto se corrige donde está.** El largo de la contraseña (§5.4) y la escala de los montos (§4.1) nacen en el diccionario.
5. **Cada arreglo de comportamiento trae la prueba que fallaba antes** (H1 a H6). Lo estructural y lo de texto (H7 a H9) se verifica con medición y revisión, y así lo dice su slice.
6. **La deuda de arquitectura se cierra con una guarda que impide que vuelva**, como `LayerDependencyTests.theKnownDebtOnlyShrinks`.
7. **Lo más barato que cumple la regla.** Y el momento correcto: lo que la próxima funcionalidad copiaría se corrige antes de ella (DP-7 y DP-8).
8. **El alcance se dice entero.** Lo que no se hace y lo no validado queda nombrado en la sección 9.

---

## 2. Hallazgos y evidencia

**Gravedad:**
- **media:** alguien de afuera puede aprovecharlo hoy;
- **baja:** un dato o una regla equivocada que hoy no ocurre o no rompe nada;
- **deuda:** no cambia lo que el sistema hace.

### H1 · El inicio de sesión no tiene freno y deja saber qué cuentas existen — media

- **Evidencia:**
  - `UserCommandServiceImpl.handle(SignInCommand)` (`iam/application/internal/commandservices/UserCommandServiceImpl.java:96-113`) responde sin llamar a `hashingService.matches` si el correo no existe o es `SYSTEM`.
  - Su propio Javadoc (líneas 90-95) promete lo contrario: «Unknown emails and wrong passwords get the same answer».
- **Medición:** una comparación BCrypt de costo 10 tardó en promedio 86,5 ms (`jshell`, `spring-security-crypto` 7.1.1). Un correo inexistente responde unos 86 ms antes.
- **Sin límite de intentos:** ninguna capa cuenta los fallos, ni el backend ni la función de Pages.
- **La misma debilidad, en otros dos caminos:**
  - **Cambio de contraseña:** compara la clave actual sin límite (`UserCommandServiceImpl.java:126-128`).
  - **Registro público:** responde `409` si el correo ya existe (líneas 148-150). Hoy viene apagado (DP-12).
- **El backend es alcanzable directo:**
  - La URL de DigitalOcean es pública (README del frontend: `BACKEND_ORIGIN`).
  - App Platform no permite restringir la entrada por IP: no tiene cortafuegos para apps, y figura como pedido en el tablero de ideas de DigitalOcean.
  - Con `forward-headers-strategy: native` (`application-prod.yaml:19`), la IP que ve el backend sale de `X-Forwarded-For`, que quien llama directo escribe a su gusto.
  - Además, Swagger y `/v3/api-docs` están activos en todos los perfiles (`application.yaml:73-76`) y se leen en esa URL, porque la función de Pages solo reenvía `/api/`.
- **Capacidad:** una sola instancia de 1 vCPU (`.do/app.yaml`, `apps-s-1vcpu-1gb`), con el pool de Hikari por omisión (10), y BCrypt corre dentro de la transacción del inicio de sesión.
- **Sin decisión que lo cubra:** ningún documento lo decide, y la revisión de seguridad del 28/10/2026 no lo cubrió.

### H2 · La contraseña se limita en caracteres, pero BCrypt corta en bytes — baja

- **Evidencia en el código:**
  - `ChangePasswordResource.java:18` y `SignUpResource.java:25` validan `@Size(min = 8, max = 72)`.
  - El frontend usa `Validators.maxLength(72)` (`sign-up-form.ts:60`, `change-password-form.ts:60`).
- **Medición:**
  - 40 «ñ» son 80 bytes, y `encode` lanza `IllegalArgumentException: password cannot be more than 72 bytes`.
  - 36 «ñ» son 72 bytes y pasan.
  - No existe ninguna clave guardada de más de 72 bytes, porque `encode` la rechaza: no hace falta migrar datos.
- **Qué recibe el usuario:** `400 VALIDATION_ERROR`, con el texto de BCrypt en inglés en `details` (`GlobalExceptionHandler.java:112-119`).
- **Otro camino:** `IAM_BOOTSTRAP_ADMIN_PASSWORD` llega a `encode` sin validar (`ApplicationReadyEventHandler`).
- **Origen:** el diccionario, §5.4 (línea 753). Lo repiten `contextos/iam.md:82`, `:198` y `:318` (IAM-R01), y `es.json:216`.

### H3 · Una moneda admite cuatro decimales y los montos se guardan con dos — baja hoy, sube con las facturas

- **Evidencia en el código:**
  - Admiten de 0 a 4 decimales: V4:23-24, `Currency.java:33`, `@Max(4)` en `CreateCurrencyResource.java:34` y `UpdateCurrencyResource.java:28`, `currency-form-dialog.ts:85`, y «entre 0 y 4» en `messages*.properties:57`.
  - El precio es `NUMERIC(14, 2)` (V69:74; `PurchaseOrderItemPersistenceEntity.java:52`).
  - `PurchaseOrderLines.java:135-137` acepta los decimales que permita la moneda, y PostgreSQL redondea a 2 sin error.
- **Origen:** el diccionario. §4.1 (línea 349) admite de 0 a 4 decimales, pero todos sus montos son `NUMERIC(14,2)`: líneas 2258, 2433, 2468, 2501, 2515, 2607, 2641 y 2668.
- **Por qué hoy no ocurre:** las semillas son PEN (2), USD (2) y COP (0), y los países previstos (D-52) usan monedas de 2 decimales o ninguno.

### H4 · `procurement` usa los records de otras fachadas sin traducirlos — deuda

- **La regla:** `references/module-template.md:306` de la skill `backend-module`: el consumidor traduce a «un objeto de valor de `procurement` con su propia regla, como `TaxIdTypeRules`», que vive en `procurement/domain/model/valueobjects`.
- **Evidencia:** 26 archivos, fuera de `outboundservices/acl`, importan records de otras fachadas:
  - `AddressSummary` en 11; `CompanySummary` en 9; `OrganizationReference` y `CurrencySummary` en 7 cada uno; `VehicleModelSummary` en 6; `EmployeeAttention` y `TaxIdTypeSummary` en 3 cada uno;
  - `ScopeReference`, `CompanyIssuer` y `CommercialBrandReference` en 1 cada uno.
  - El archivo 27, `PurchaseOrderCurrencyUsageAdapter`, implementa el puerto `CurrencyUsage` y es correcto.
- **Hasta dónde llegan:** `AddressSummary` llega a vistas públicas y a tres ensambladores REST.
- **Comparación:** ningún otro contexto de negocio lo hace, si no se cuenta el contrato de la plataforma `documents::acl`, que importan 31 archivos y es legítimo.
- **Impacto real:** es un acoplamiento que rompe la compilación, no un cambio silencioso del JSON.

### H5 · La regla del camino de estado vive en el servicio y no en el dominio — deuda

- **Evidencia:** `Vehicle.transitionTo(Long)` (`Vehicle.java:137-139`) es público y solo asigna el estado. Las revisiones están en `VehicleCommandServiceImpl.java:277-294`.
- **Lo que sí se cumple:** la regla se aplica y tiene prueba (`VehicleStatusTransitionTests.java:161`).
- **El riesgo:** que las cargas del paso 9 escriban estados sin pasar por ella.

### H6 · El store de `iam` navega — deuda

- **La regla:** skill `frontend-feature`, `SKILL.md:124`: «El store nunca navega».
- **Evidencia:** `iam.store.ts` navega en `signIn` (470), `signUp` (499) y `changePassword` (518).
- **La cuarta llamada (1016) se justifica:** es el fin de la sesión que dispara el interceptor de un `401`, sin vista. La regla no la nombra.

### H7 · Servicios de comandos por encima de las ~300 líneas — deuda

| Servicio | Código | Qué escribe |
|---|---|---|
| `fleet/.../VehicleCommandServiceImpl` | 527 | La unidad y su historial (184, 258 y 378) y las lecturas de kilometraje (300) |
| `procurement/.../PurchaseOrderTemplateCommandServiceImpl` | 477 | Uno, ya partido en parte |
| `organization/.../EmployeeCommandServiceImpl` | 399 | Uno |
| `iam/.../RoleCommandServiceImpl`, `procurement/.../PurchaseOrderCommandServiceImpl` y `organization/.../LocationCommandServiceImpl` | 338, 320 y 309 | En el borde |
| `iam/.../UserCommandServiceImpl` | 280 | Tres agregados; pasaría de 300 con el freno, y por eso S4 saca de ahí el inicio de sesión |

### H8 · Textos desactualizados — baja

| Dónde | Línea | Qué dice | Qué es hoy |
|---|---|---|---|
| README del backend | 60 | Un rol de empresa lo ve «la cuenta que lo tiene asignado» | La que lo tiene **o lo tuvo** (sección 9) |
| README del backend | 70 | Las cuentas y los roles «siguen por rol» | Por los permisos `seguridad.*` (D-117) |
| README del backend | 382 y 388 | «Los cinco contextos que exportan» y «`procurement` todavía no existe» | Seis exportan; faltan `procurement` y `notifications` en el árbol |
| README del frontend | 79 a 87 | Seis sondas | Ocho: faltan `procurement` y `funds` |
| README del frontend | 111 | «iam, location y organization hoy» | Están todos, además de `change-log` |
| `app.routes.ts` | 40, 41 y 52 | `data.roles` | `data.permissions` |
| `playwright.live.config.ts` | 4 | Tres sondas | Ocho |
| Convenciones | 52 | `/change-log` | `/change-logs` |
| Convenciones | 66 y 898 | «los ocho de accesos y roles» | Once (`PermissionCodes.java:175-177`) |
| Convenciones | 708 | Cinco contextos que exportan | Seis |
| Convenciones | 968 | `@PermissionRequired("catalogos.ver")` como texto | `PermissionCodes.Catalogs.VIEW` |
| `contextos/iam.md` | 291 | «los ocho» | Once |
| `retroalimentacion/2026-10-31-sondas.md` | 3 | «sin publicar todavía» | Prohibido por `contextos/README.md:56` |
| Skill `backend-module` (las dos copias) | `SKILL.md:178` y `architecture-rules.md:294` | «Los ocho» | Once |
| Skill `frontend-feature` (las dos copias) | `SKILL.md:192` | «los ocho» | Once |
| Diccionario | 1402, 3234 y 3236 | «doce permisos `seguridad.*`» y «los ocho» | Dieciséis no retirados y once. **Lo cambia solo el responsable** |

### H9 · La regla 6.2 de las skills choca con las convenciones — baja

- **La regla:** `architecture-rules.md:335-349` prohíbe responder un archivo en el mismo pedido, y su `grep` marca el `Content-Disposition`.
- **El código:** dos endpoints responden los bytes de `kept_files` en el mismo pedido, como dicen las convenciones (L221 y L828):
  - `PurchaseOrderTemplatesController.java:210-226`, con `Content-Disposition: inline`;
  - `PurchaseOrdersController.java:227-245`, sin esa cabecera.

---

## 3. Decisiones del responsable

Recomendaciones de la versión 3: la columna de la derecha dice qué cambió respecto de la versión 2.

| # | Decisión recomendada | Por qué | Antes de | Cambio |
|---|---|---|---|---|
| DP-1 | **El freno cuenta por cuenta y por dispositivo** (opción B). Quien entró con éxito desde un navegador recibe una cookie firmada, `MM_DEVICE`, y desde entonces sus intentos en ese navegador tienen su propio contador. Todo intento sin esa cookie comparte un único contador por cuenta, venga de la IP que venga, también para los correos que no existen | Frena un ataque repartido entre muchas IP, que la llave por IP no frenaba. Nadie puede bloquear a una persona en su navegador habitual, incluido el único superadministrador, que no puede restablecer su propia clave (D-70). No depende de ninguna IP, en un App Platform que no permite filtrarlas. Es el patrón de OWASP | S4 | Era correo más IP (opción A) |
| DP-2 | **Política de cada contador:** cinco fallos libres; después, una espera de 1 minuto que se duplica hasta 15. Un acierto vuelve a cero solo el contador usado, y un fallo de hace más de 24 horas deja de contar. Mientras dura la espera responde `429 SIGN_IN_THROTTLED`, con `Retry-After` y `context.retryAfterSeconds`, sin revisar la clave. Nunca hay bloqueo definitivo. Un aviso en el registro cuando una cuenta suma más de 100 fallos en 24 horas | Unos 100 intentos por día por cuenta desde navegadores sin historia, sin delatar qué cuentas existen. Quien llega desde un navegador nuevo justo durante un ataque a su cuenta espera o usa su equipo habitual: es el costo aceptado | S4 | Sin cambio, salvo la llave |
| DP-3 | **Privacidad:** el HMAC-SHA256 con la clave `IAM_SIGN_IN_KEY` para las llaves de la tabla, y la misma clave firma `MM_DEVICE`. La cookie lleva un identificador al azar y su firma, nunca el correo. Una tarea purga los contadores sin actividad en 24 horas y los dispositivos de confianza sin uso en 180 días, con `DELETE` solo sobre esa tabla, como en `spring_session*` | Un SHA-256 sin clave de un correo de dominio conocido se revierte (Ley 29733). La regla de no borrar es de los datos del negocio, no de esta tabla de control | S4 | La clave también firma la cookie |
| DP-4 | **Dos** comparaciones BCrypt simultáneas por instancia, con 3 segundos de espera; después, `503 SIGN_IN_BUSY`. El valor es configurable | En 1 vCPU, más de dos tareas de CPU a la vez solo se reparten el procesador y vuelven lentas las demás pantallas | S4 | Eran cuatro, con 2 s |
| DP-5 | El largo de la contraseña, de 8 caracteres a 72 bytes en UTF-8. El diccionario lo dice en bytes; la pantalla, en lo que la persona entiende: «De 8 a 72 caracteres; las letras con tilde y la ñ cuentan doble» | El diccionario coincide con el cifrado, sin migrar claves. Queda como paso siguiente, fuera de este cierre, rechazar las claves ya filtradas, como recomienda NIST | S6 | — |
| DP-6 | **Opción A:** las monedas, de 0 a 2 decimales | Coincide con las semillas y con los países previstos. Los tipos de cambio son otra columna. Una moneda de 3 decimales reabriría la decisión | S7, antes de las facturas | — |
| DP-7 | **La guarda de H4, ahora. La traducción de `procurement`, antes de empezar las facturas.** Lo que llega a una vista o a REST se traduce a un objeto de valor del dominio; lo que usa solo un servicio de comandos, a un record de `outboundservices/acl`. Quedan fuera `documents::acl` y los puertos que el contexto implementa | Las facturas viven en `procurement`: escritas antes, copiarían el acoplamiento. La guarda es barata y frena que crezca desde hoy | S11 | Fija el momento |
| DP-8 | **En `fleet`, antes del paso 9:** `StatusPathPolicy` dentro del agregado y las lecturas en su propio servicio. La unidad y su historial, en piezas con nombre. `UserCommandServiceImpl` pierde el inicio de sesión con S4. Los demás, cuando se trabaje en ellos | Las cargas del paso 9 serán las primeras en escribir estados y lecturas fuera del servicio actual. Partir por partir arriesga una zona estable y probada | S12 y S4 | Fija el momento |
| DP-9 | El store deja de navegar, con la excepción nombrada del fin de sesión | Bajo costo, y deja la regla coherente | S13 | — |
| DP-10 | La regla 6.2 nombra la excepción de `kept_files`, y su `grep` se ajusta | El código y las convenciones ya lo hacen | S9 | — |
| DP-11 | **Diccionario v1.34:** D-144 (el freno: DP-1 a DP-4, su tabla y la cookie), D-145 (DP-5), D-146 (DP-6), D-147 (el refuerzo, DP-13), y «once» y «dieciséis» en §5.8 regla 7, D-122 y D-124. **La tabla, en el bloque 2 de `iam`**, como tabla de control | El mapa de contextos reparte las tablas por su dueño, que es `iam`. §2.7 es para las tablas de la plataforma | S3, S4, S6 y S7 | La tabla, en el bloque 2 y no en §2.7 |
| DP-12 | El registro público sigue apagado. Antes de encenderlo, junto con el registro con perfil (D-32, D-83), tiene que responder igual exista o no el correo y pasar por el mismo freno | Hoy responde `403` si está apagado: no se puede aprovechar | — | — |
| DP-13 | **El refuerzo: al backend solo se llega por Cloudflare.** La función de Pages manda un secreto (`ORIGIN_SECRET`), y en `prod` el backend responde `404` a todo pedido sin él, salvo `/actuator/health`. Se aceptan dos secretos a la vez para rotarlo sin cortar el servicio, y se comparan en tiempo constante. La función manda también la IP real (`CF-Connecting-IP`), que el backend usa solo para el registro. Swagger y `/v3/api-docs` se apagan en `prod`. Una regla de limitación de Cloudflare sobre el inicio de sesión, holgada: 30 pedidos por minuto por IP con 10 minutos de bloqueo, o lo más cercano que permita el plan de la zona, porque la fuerza bruta la frena la aplicación y una sede entra a la misma hora detrás de una sola IP | App Platform no permite filtrar la entrada por IP, y filtrar las IP de Cloudflare en la aplicación dependería de cabeceras falsificables: un secreto compartido es la forma práctica. Hace efectivas la regla de limitación y las protecciones de Cloudflare ante una ráfaga, que hoy se saltan llamando directo. Deja de exponer la documentación de la API | S3 | Nueva: de opcional pasa a ser parte del plan, fuera de la ruta crítica |

**Acciones tuyas en el entorno** (nadie más puede hacerlas):

1. **Antes de S7, en las tres bases** —producción, tu `erp_mm_db` y `erp_mm_db_verify`—: `SELECT iso_code, decimal_places FROM currencies WHERE decimal_places > 2;`.
2. **Antes de S4:** generar `IAM_SIGN_IN_KEY` (`openssl rand -base64 32`) y cargarla como secreto en App Platform.
3. **Durante S3**, en el orden de la sección 7:
   - generar `ORIGIN_SECRET` y cargarlo con el mismo valor en *Variables and Secrets* de Pages y como secreto en App Platform;
   - encender `IAM_ORIGIN_CHECK_ENABLED` en App Platform cuando la función ya mande el secreto;
   - crear la regla de limitación de Cloudflare sobre `POST /api/v1/authentication/sign-in`, con los valores que permita el plan de la zona.
4. **Bases:** las pruebas y las sondas siguen sobre `erp_mm_db_verify` mientras `erp_mm_db_live` no exista.

Los dos secretos nunca van al repositorio ni a un `.env` que se suba. En desarrollo no hacen falta: el filtro del origen solo se enciende en `prod`, y la clave del freno tendrá un valor de desarrollo en `application-dev.yaml`, que S4 agrega.

---

## 4. Reglas de ejecución

Las del plan de cierre del 25/10/2026 ([sección 3](plan-de-cierre-de-la-auditoria-2026-10-25.md)), con lo que aplica aquí:

- **Ramas y lugar de trabajo:**
  - Cada rama `feature/<contexto>` vive en un worktree hermano (`mm-<tema>-be`, `mm-<tema>-fe`).
  - Nunca se compila en los repositorios principales.
  - El e2e corre en el 4300, y las sondas en el 8090 contra `erp_mm_db_verify`.
- **Pruebas:**
  - Se corren las que alcanza cada cambio.
  - La suite completa del backend, antes de integrar S1, S3, S4, S6, S7 y S11.
  - El e2e completo, con el equipo enchufado, antes de integrar S2, S6, S7 y S13.
- **Pruebas sobre una base que no se vacía:** las que necesitan ver una confirmación real no son `@Transactional`: usan `TransactionTemplate`, como las `*LockTests`, y correos y dispositivos propios de cada corrida.
- **Migraciones:**
  - Solo una rama con migraciones sin integrar a la vez.
  - Cada una abre con `set_config('app.access_scope', 'SYSTEM', true)` y `SET LOCAL lock_timeout = '5s'`, como V84 y V85.
  - El número se fija al integrar.
- **Commits:** Conventional Commits en inglés, uno por slice, y cada commit compila.
- **Integración y publicación:** integras tú, o con tu visto bueno, y el push lo pides tú, en el orden de la sección 7.
- **Documentación:** cada slice actualiza en el mismo cambio su documento de contexto y su regla. Los documentos citan commits y fechas, nunca el estado de publicación.
- **Agentes:** el trabajo largo va en agentes en segundo plano, y cada slice se commitea apenas está en verde.

---

## 5. Fases

### F1 · El inicio de sesión (H1)

#### S1 · La misma respuesta y el mismo tiempo — backend, sin decisión

- **Qué:**
  - Al construirse, el servicio genera un hash de relleno con `hashingService.encode`. El puerto no cambia.
  - `handle(SignInCommand)` compara siempre: contra el hash de la cuenta, o contra el de relleno si el correo no existe o es `SYSTEM`. La respuesta no cambia.
- **Prueba que falla antes:** `UserCommandServiceImplTest` (líneas 102-214). Su `PlainTextHashingService` (línea 476) cuenta las comparaciones: una con un correo inexistente, una con `SYSTEM` y una con la clave equivocada.
- **Contrato:** ninguno. Se publica solo.
- **Documentación:** una regla IAM- nueva en `contextos/iam.md`.
- **Tamaño:** S.

#### S2 · El frontend entiende el freno — frontend, antes de S4

- **Simulador:** `e2e/support/api.ts` no simula `POST /authentication/sign-in` (3514-3522). Se suma, con escenarios de `429` y `503`.
- **Store:**
  - Su error, hoy solo una clave (`iam.store.ts:112`; `sign-in-form.html`, `errorKey | translate`), pasa a llevar parámetros, para decir los minutos con `context.retryAfterSeconds`.
  - `signInErrorKey` (1078) y `changePasswordErrorKey` suman `SIGN_IN_THROTTLED` y `SIGN_IN_BUSY`, con sus claves en `es.json`.
- **La cookie:** no hay nada que hacer en el frontend. `MM_DEVICE` es `HttpOnly` y del mismo origen, así que viaja sola.
- **Pruebas que fallan antes:** `iam.store.spec.ts`, `sign-in-form.spec.ts` y un e2e del formulario junto a los de `shell.spec.ts`.
- **Contrato:** expande antes que el backend.
- **Tamaño:** S.

#### S3 · El refuerzo: al backend solo se llega por Cloudflare — backend, frontend y entorno (DP-13)

Independiente del freno: se construye y se publica en paralelo con S2 y S4.

- **Función de Pages** (`functions/api/[[path]].js`):
  - si `env.ORIGIN_SECRET` existe, agrega `X-MM-Origin` con ese valor y `X-MM-Client-IP` con `CF-Connecting-IP`;
  - si falta, reenvía como hoy, para que configurar los dos lados no dependa de un orden exacto;
  - nunca reenvía un `X-MM-Origin` o un `X-MM-Client-IP` que haya llegado del navegador: los borra antes de poner los suyos.
- **Backend:**
  - Un filtro de servlet en `iam/infrastructure/authorization`, antes de la cadena de Spring Security, activo solo con `iam.origin-check.enabled=true` (`IAM_ORIGIN_CHECK_ENABLED`, apagado por omisión).
  - Compara `X-MM-Origin` en tiempo constante (`MessageDigest.isEqual`) contra `ORIGIN_SECRET` y, si existe, `ORIGIN_SECRET_PREVIOUS`, que sirve para rotar.
  - Sin secreto válido responde `404`, sin cuerpo que explique el motivo, salvo `/actuator/health`, que usa el chequeo de `.do/app.yaml`.
  - Con secreto válido, guarda la IP de `X-MM-Client-IP` como atributo del pedido, solo para el registro (el aviso de DP-2). Ninguna regla la usa para decidir.
- **Swagger en producción:** `springdoc.api-docs.enabled` y `springdoc.swagger-ui.enabled` en `false` en `application-prod.yaml`. Sigue en `dev`.
- **Pruebas:**
  - `OriginCheckFilterTests`. Con el filtro encendido:
    - sin cabecera, `404`;
    - con una cabecera equivocada, `404`;
    - con la correcta, pasa y expone la IP;
    - con el secreto anterior, pasa;
    - la ruta de salud pasa sin cabecera.

    Apagado, todo pasa.
  - Una prueba del perfil `prod` que comprueba que `/v3/api-docs` no responde.
  - La función no tiene pruebas automáticas en el repositorio. Se verifica con `npx wrangler pages dev` y `curl`: que las cabeceras lleguen y que las del navegador se reemplacen.
- **Publicación:** la sección 7, paso 5.
  1. El backend con el filtro apagado.
  2. El secreto en Pages y en App Platform, y la función con las cabeceras.
  3. Se enciende el filtro.
  4. Se comprueba:
     - `curl` directo a la URL de DigitalOcean responde `404`;
     - `dev.erp.millennium.ky` funciona;
     - el chequeo de salud sigue en verde.
  5. La regla de Cloudflare.
- **Rotación**, documentada en el README de Producción:
  1. el secreto nuevo en `ORIGIN_SECRET` y el viejo en `ORIGIN_SECRET_PREVIOUS`, en App Platform;
  2. el nuevo en Pages;
  3. al terminar, se vacía `ORIGIN_SECRET_PREVIOUS`.
- **Documentación:**
  - README del backend: Producción y variables;
  - README del frontend: Pages y `ORIGIN_SECRET`;
  - `.do/app.yaml`;
  - convenciones §9;
  - `contextos/iam.md`.
- **Tamaño:** M.

#### S4 · El freno por cuenta y por dispositivo — backend, con DP-1 a DP-4, DP-8 y DP-11

- **Migración V87: `sign_in_throttles`**, de `iam`.
  - Columnas:
    - `id`;
    - `throttle_key CHAR(64) UNIQUE`: el HMAC del correo normalizado, para el contador de la cuenta, o del correo con el identificador del dispositivo, para el de un dispositivo;
    - `kind VARCHAR(12)`, con `CHECK` en `cuenta` o `dispositivo`, en minúsculas como las demás listas cerradas;
    - `email_key CHAR(64)`, para el aviso de DP-2;
    - `consecutive_failures SMALLINT NOT NULL DEFAULT 0`, `last_failure_at`, `blocked_until`;
    - `trusted_until TIMESTAMPTZ`: hasta cuándo un dispositivo es de confianza;
    - `created_at`, `updated_at`.
  - Sin `public_id`, sin `company_id` ni Row Level Security, como `users`.
  - `GRANT DELETE` a `erp_mm_app` solo sobre esta tabla, con su línea en `hand-over-database.sql` (`HandOverDatabaseScriptTests`).
  - Sin `data_versions`.
- **La cookie `MM_DEVICE`:**
  - un identificador al azar de 128 bits y su firma HMAC;
  - `HttpOnly`, `Secure` fuera de `dev`, `SameSite=Strict`, `Path=/api/v1/authentication` y 180 días.
  - Se emite, o se renueva, solo con un acierto.
  - Una cookie con la firma equivocada se trata como si no existiera.
  - Un dispositivo es de confianza para una cuenta solo si tiene una fila `dispositivo` con `trusted_until` en el futuro, y esa fila nace con un acierto. Por eso la cookie no delata nada.
- **Dominio:** el agregado `SignInThrottle`, en Java puro, con la regla de DP-2:
  - `reserve(now)`: si está bloqueado devuelve cuánto falta; si no, cuenta el intento como fallo de antemano;
  - `settleSuccess(now)`;
  - el decaimiento;
  - `trustUntil`.

  Su repositorio va en `iam/domain/repositories`, con `findByKeyForUpdate`. Su entidad usa la clase base de sus columnas (`AuditableAbstractPersistenceEntity`, D-42), sin SQL que escriba fechas a mano.
- **El flujo**, sin candados ni conexión mientras corre BCrypt:
  1. **Elegir el contador.** Si la cookie es válida y la fila `dispositivo` de esa cuenta es de confianza, el del dispositivo; si no, el de la cuenta.
  2. **Transacción corta 1:**
     - asegura la fila (`INSERT … ON CONFLICT DO NOTHING`);
     - la bloquea con `FOR NO KEY UPDATE` y la refresca;
     - `reserve(now)` decide si sigue o responde `429`;
     - lee la cuenta, si existe;
     - confirma.

     Diez pedidos simultáneos sobre el mismo contador se ordenan aquí, y solo cinco pasan.
  3. **Fuera de toda transacción:** la comparación BCrypt —contra el hash de la cuenta o el de relleno— bajo el tope de DP-4. Sin permiso en 3 segundos, `503`.
  4. **Transacción corta 2, solo si acertó:**
     - `settleSuccess` sobre el contador usado, nunca sobre el de la cuenta si se usó el del dispositivo: el de la cuenta se enfría solo;
     - la fila `dispositivo` con `trusted_until` en 180 días;
     - la revisión de suspensión;
     - `recordSignIn`;
     - la sesión;
     - la cookie.

     Una cuenta suspendida con la clave correcta también vuelve a cero, y responde `ACCOUNT_SUSPENDED`, como hoy.
- **El cambio de contraseña** pasa por el mismo flujo antes de comparar la clave actual.
- **Dónde vive** (DP-8):
  - `SignInCommandServiceImpl`, nuevo, con el inicio de sesión y el cambio de contraseña, que salen de `UserCommandServiceImpl`.
  - La reserva y el cierre, en `SignInThrottleCommandServiceImpl`, cada uno en su transacción y envuelto por `DeadlockRetry`. Una repetición por interbloqueo cuenta, a lo sumo, un intento de más.
- **Purga:** `SignInThrottlePurgeTask`, dentro de `SingleInstanceRun.runAlone` (`iam.sign-in-throttle-purge`), con las reglas de DP-3.
- **Respuesta:**
  - `ErrorResponseAssembler.toStatusFromErrorCode` suma `SIGN_IN_THROTTLED` → `429` y `SIGN_IN_BUSY` → `503`.
  - El controlador arma el `context` con `toErrorResponseFromApplicationError(error, context)`, porque `ResponseEntityAssembler` lo descarta, y agrega `Retry-After`.
  - Las anotaciones de OpenAPI suman el `429` y el `503`.
  - Las claves van en los dos `messages*.properties` (`OwnErrorCodesHaveMessagesTests`).
- **Configuración:** `iam.sign-in.throttle.*`, `iam.sign-in.max-concurrent-verifications` e `IAM_SIGN_IN_KEY`, con sus variables en el README y en `.do/app.yaml`.
- **Pruebas que fallan antes:**
  - `SignInThrottleTests`, del dominio, sin Spring.
  - `SignInThrottleIntegrationTests`, no transaccional, con correos y dispositivos de la corrida:
    1. sin cookie, cinco fallos y el sexto responde `429`, aunque traiga la clave correcta;
    2. un correo inexistente recibe la misma secuencia;
    3. con la cuenta frenada, un dispositivo de confianza entra con la clave correcta;
    4. una cookie con la firma equivocada se trata como sin cookie;
    5. un dispositivo se vuelve de confianza solo después de un acierto;
    6. un acierto vuelve a cero;
    7. un fallo de hace más de 24 horas no cuenta;
    8. el cambio de contraseña con la clave actual equivocada cuenta;
    9. la purga borra solo lo vencido.
  - `SignInThrottleConcurrencyTests`: diez pedidos simultáneos equivocados dan exactamente cinco `401` y cinco `429`.
  - `SignInVerificationLimitTests`: el tope responde `503`.
  - `ErrorResponseAssemblerTest`: los dos estados.
  - `UserCommandServiceImplTest` pierde los casos del inicio de sesión, que pasan a `SignInCommandServiceImplTest`.
- **Documentación:**
  - convenciones §7 (`429` y `503`) y §9 (`MM_DEVICE` junto a `MM_SESSION`);
  - `contextos/iam.md`;
  - el diccionario (DP-11);
  - el README.
- **Tamaño:** L. Es la ruta crítica.

#### S5 · La sonda lo confirma — frontend

- **Qué:** `e2e-live/security.live.spec.ts` suma dos casos.
  1. Con una cuenta creada por la corrida y con un correo que no existe, seis intentos cada uno desde un contexto de navegador nuevo: las dos secuencias son idénticas, cinco `401` y un `429`, con la pantalla y su mensaje.
  2. La cuenta de la corrida, ya frenada, entra desde el contexto de navegador donde había entrado antes.
- **Tamaño:** S.

### F2 · Lo que el diccionario corrige (H2 y H3)

#### S6 · El largo de la contraseña en bytes — backend y frontend, con DP-5 y DP-11

- **Backend:**
  - Una anotación de Bean Validation en `iam/interfaces/rest/validation` (al menos 8 caracteres y como máximo 72 bytes) reemplaza a `@Size` en `ChangePasswordResource.java:18` y `SignUpResource.java:25`.
  - La clave `validation.password-length` cambia en los dos `ValidationMessages*.properties`.
  - `ApplicationReadyEventHandler` valida `IAM_BOOTSTRAP_ADMIN_PASSWORD`, y si no cumple no arranca, con un mensaje claro.
- **Frontend:**
  - Un validador que cuenta bytes con `TextEncoder` reemplaza a `Validators.maxLength(72)`.
  - Su error se suma a `KNOWN_ERRORS` (`base-form.ts:9`), con su clave en `es.json`.
  - La pista `es.json:216` toma el texto de DP-5.
- **Pruebas que fallan antes:**
  - `AccountPasswordLengthIntegrationTests`:
    - 36 «ñ» se aceptan;
    - 37 responden `400` nombrando el campo y sin el texto de BCrypt;
    - 72 caracteres ASCII se aceptan.
  - `sign-up-form.spec.ts` y un `change-password-form.spec.ts` nuevo.
  - Una prueba del arranque con una clave de 80 bytes.
- **Capturas:** cambian `sign-up-{desktop,mobile,tablet}-win32.png`; se regeneran y se revisan.
- **Documentación:** `contextos/iam.md:82`, `:198` y `:318`, y §5.4 del diccionario.
- **Contrato:** el frontend primero.
- **Tamaño:** S.

#### S7 · Monedas de hasta dos decimales — backend y frontend, con DP-6, DP-11 y la acción 1

- **V88:**
  - bloquea `currencies` (`SHARE ROW EXCLUSIVE`, como V76);
  - se detiene si alguna moneda tiene más de 2 decimales;
  - cambia el `CHECK` a `BETWEEN 0 AND 2`.
- **Código:**
  - `Currency.MAX_DECIMAL_PLACES = 2`, y su Javadoc (`Currency.java:59`);
  - `CreateCurrencyCommand` y `CurrencySummary`;
  - `@Max(2)` en los dos recursos;
  - `messages*.properties:57`;
  - `currency-form-dialog.ts:85`.
- **Pruebas que fallan antes:**
  - `CurrencyTest`;
  - `CurrenciesControllerTests`;
  - `currency-form-dialog.spec.ts`, línea 119;
  - `CurrencyDecimalPlacesMigrationTests`: después de migrar, insertar 3 decimales viola el `CHECK`.
  - La detención de V88 la cubre la acción 1, en las tres bases.
- **Documentación:**
  - el diccionario §4.1 (349) y la columna `decimal_places` (327);
  - LOC-R04 (`location.md:206`) y `location.md:120`;
  - la ficha de Monedas en procesos y pantallas.
- **Contrato:** el frontend primero, después la migración y el backend.
- **Tamaño:** S. Va antes de las facturas.

### F3 · Documentación y skills (H8 y H9)

Se verifica por revisión y `grep`. Va en paralelo con todo lo demás.

- **S8 · Repositorio de documentación**, sin el diccionario:
  - las filas de convenciones de H8;
  - `contextos/iam.md:291`;
  - la línea 3 de `retroalimentacion/2026-10-31-sondas.md`;
  - con S3 y S4, convenciones §7 y §9.
- **S8b · READMEs y comentarios:**
  - README del backend: 60, 70, 382 y 388, y el árbol;
  - README del frontend: 79 a 87 y 111;
  - `app.routes.ts`: 40, 41 y 52;
  - `playwright.live.config.ts`: 4.
- **S9 · Skills, en sus dos copias:**
  - «los once»;
  - la excepción de DP-10 en la regla 6.2;
  - la de DP-9 en `frontend-feature` §3;
  - la guarda de S11 en la regla 1.1;
  - DP-3 como excepción nombrada a la regla 4.2;
  - el filtro del origen de DP-13 en la sección de seguridad.
- **S10 · Diccionario v1.34:** el texto exacto de DP-11, para que lo apruebes.
- **Tamaño:** S en total.

### F4 · Deuda de arquitectura, sin cambio de comportamiento (H4, H5 y H7)

Cada slice cierra con las pruebas del contexto, `ModularityTests` y su sonda en vivo.

#### S11 · `procurement` traduce sus fachadas — backend, con DP-7

1. **La guarda, ahora.** `ForeignAclStaysInTranslationLayerTests`, con el estilo de `LayerDependencyTests`, la regla de DP-7 y los 26 archivos como deuda que solo baja. Se mide con la regla exacta antes de commitear.
2. **La traducción, antes de las facturas**, un slice por record:

| Slice | Record de otro contexto | Destino propuesto | Archivos |
|---|---|---|---|
| S11a | `AddressSummary` (y `AddressData`, solo dentro del servicio de salida) | `PostalAddress`, objeto de valor del dominio | 11 |
| S11b | `CompanySummary`, `CompanyIssuer` | `OwningCompany`, `IssuerData` | 9 y 1 |
| S11c | `OrganizationReference`, `ScopeReference`, `CommercialBrandReference` | `NamedReference`, `IssuingScope`, `BrandReference` | 7, 1 y 1 |
| S11d | `CurrencySummary` | `OrderCurrency` | 7 |
| S11e | `VehicleModelSummary` | `LineModel` | 6 |
| S11f | `EmployeeAttention`, `TaxIdTypeSummary` | `AttentionContact`; `TaxIdTypeRules` ya existe | 3 y 3 |

- **Pruebas:** las 52 clases de `procurement`, `ModularityTests` y `procurement.live.spec.ts`.
- **Tamaño:** M.

#### S12 · `fleet`: el camino en el agregado y el servicio partido — backend, antes del paso 9 (DP-8)

- **Política:** `StatusPathPolicy`, en `fleet/domain/services`. Devuelve un `Verdict`, como `MileageSeriesPolicy`, nunca códigos de error.
- **El agregado:** `Vehicle.transitionTo` recibe el camino declarado como objeto de valor y se niega si la política no lo permite. El servicio traduce el rechazo a los códigos de hoy. `transitionAlone` sigue con `AutomaticReturnPolicy`.
- **La partición:**
  - `VehicleMileageReadingCommandServiceImpl`, nuevo, con las lecturas (300);
  - `VehicleCommandServiceImpl`, con las piezas `VehicleMovements` (184, 258 y 378) y `VehicleIdentifierCorrections` (385 y 448). Meta: unas 300 líneas de código.
- **Pruebas:**
  - `StatusPathPolicyTests` y el caso de `Vehicle` que se niega;
  - `VehicleStatusTransitionTests` y `VehicleRacedIdentifierTests`;
  - `DeadlockRetryTests`, que ya los acepta (línea 119);
  - `fleet.live.spec.ts` y `changelog.live.spec.ts`.
- **Documentación:** FLT-R17 pasa a «Dominio».
- **Tamaño:** M.

#### S13 · El store de `iam` deja de navegar — frontend (DP-9)

- **Qué:**
  - `signIn` devuelve `Promise<Session | null>`; `signUp` y `changePassword`, `Promise<boolean>`.
  - Navegan las tres vistas.
  - El fin de la sesión (1016) se queda, con la excepción nombrada.
- **Pruebas:** las 8 referencias de `iam.store.spec.ts` que no son del fin de sesión, los specs de las vistas y los e2e del formulario.
- **Tamaño:** S.

#### S14 · Las demás particiones — cuando se trabaje en ellas (DP-8)

`EmployeeCommandServiceImpl`, `PurchaseOrderTemplateCommandServiceImpl` y, si crecen, `RoleCommandServiceImpl`, `PurchaseOrderCommandServiceImpl` y `LocationCommandServiceImpl`.

---

## 6. Registro de migraciones

| Número | Slice | Qué hace | Condición |
|---|---|---|---|
| V87 | S4 | Crea `sign_in_throttles`, con `DELETE` solo para esa tabla | DP-1 a DP-3 y DP-11 aprobadas |
| V88 | S7 | El `CHECK` de los decimales a 0..2; se detiene si alguna moneda tiene más | DP-6 y la acción 1 sin filas en las tres bases |

Si S7 se integra antes que S4, los números se intercambian.

---

## 7. Contratos y orden de publicación

`develop` despliega en cada push, y durante un despliegue conviven dos instancias.

| Paso | Backend | Frontend y entorno | Por qué ese orden |
|---|---|---|---|
| 1 | S1 | — | Sin contrato |
| 2 | — | S2, S6 (frontend) y S7 (frontend) | El frontend conoce los códigos nuevos, valida bytes y limita a 2 decimales antes de que el backend lo exija |
| 3 | S4 (V87) y S6 (backend) | La clave del freno en App Platform, antes | Durante el despliegue, la instancia vieja no frena y la nueva sí. La tabla nueva no rompe a la vieja |
| 4 | — | S5 | La sonda confirma el freno |
| 5 | S3: el backend con el filtro apagado y Swagger apagado en `prod` | El secreto en los dos lados y la función → el filtro encendido → la comprobación con `curl` → la regla de Cloudflare | Para no dejar la API sin servicio. **Independiente de los pasos 2 a 4:** puede ir antes o después |
| 6 | S7 (V88) | — | Tras la acción 1 en las tres bases |
| 7 | La guarda de S11 | — | Cuanto antes |
| 8 | La traducción de S11 | — | Antes de las facturas |
| 9 | S12 | — | Antes del paso 9 |
| 10 | — | S13 | Sin contrato |

F3 se publica con el código que describe; S8b y S9 pueden ir antes.

---

## 8. Cobertura

| Hallazgo | Slice | Prueba que lo cierra | Documento que cambia |
|---|---|---|---|
| H1, tiempo | S1 | `UserCommandServiceImplTest` | `contextos/iam.md` |
| H1, fuerza bruta y enumeración | S2, S4 y S5 | `SignInThrottleTests`, `SignInThrottleIntegrationTests`, `SignInThrottleConcurrencyTests`, `SignInVerificationLimitTests`, `iam.store.spec.ts`, `sign-in-form.spec.ts`, e2e del formulario, `security.live.spec.ts` | Diccionario (D-144, bloque 2), convenciones §7 y §9, `contextos/iam.md`, README del backend, `.do/app.yaml` |
| H1, cambio de contraseña | S4 | `SignInThrottleIntegrationTests`, caso 8 | `contextos/iam.md` |
| H1, acceso directo y Swagger expuesto | S3 | `OriginCheckFilterTests`, la prueba de `/v3/api-docs` en `prod`, la comprobación con `curl` | Diccionario (D-147), convenciones §9, los dos README, `.do/app.yaml` |
| H1, registro público | — | — | Fuera, con condición (DP-12) |
| H2 | S6 | `AccountPasswordLengthIntegrationTests`, los dos specs de formulario, la prueba del arranque | Diccionario §5.4 (D-145), `contextos/iam.md` |
| H3 | S7 | `CurrencyTest`, `CurrenciesControllerTests`, `CurrencyDecimalPlacesMigrationTests`, `currency-form-dialog.spec.ts`, la acción 1 | Diccionario §4.1 (D-146), LOC-R04 |
| H4 | S11 | `ForeignAclStaysInTranslationLayerTests`, en cero | Skill `backend-module` |
| H5 | S12 | `StatusPathPolicyTests`, el caso de `Vehicle` | FLT-R17 |
| H6 | S13 | `iam.store.spec.ts` y los specs de las vistas | Skill `frontend-feature` |
| H7 | S4, S12 (y S14) | Medición de líneas de código | — |
| H8 | S8, S8b, S9 y S10 | Revisión y `grep` | Los de la tabla de H8 |
| H9 | S9 | Revisión y `grep` | Regla 6.2 |

---

## 9. Lo que no se hace, y por qué

| Tema | Por qué no |
|---|---|
| `roles_select` (V45:93-99) deja ver los roles de asignaciones revocadas | No es defecto: `User` carga todas sus asignaciones con su rol, y escondiendo uno, cargar la cuenta fallaría. Solo cambia la redacción del README |
| Códigos `NOT_FOUND` y otros de `GlobalExceptionHandler.handleErrorResponse` | Ya registrado en convenciones §13 (L1056), con su decisión pendiente en §14 (L1082) |
| `HtmlTemplates` y `HtmlPdf` en `shared` con un solo usuario | D-143 y NTF-R09 los destinan a `notifications` |
| `strict` y `strictTemplates` en el frontend | Ya están activos por omisión en TypeScript 6.0.3 y Angular 22.2.3 |
| ESLint en el frontend | Ninguna regla lo exige; sería una propuesta aparte |
| El registro público y su `409` | DP-12 |
| Un freno por IP | El freno por cuenta y dispositivo ya frena los ataques repartidos, y la IP solo sirve para el registro (DP-13) |
| Rechazar claves filtradas o pedir composición | Paso siguiente de DP-5; no es un hallazgo de esta revisión |
| Reducir la clave con SHA-256 antes de BCrypt | DP-5 |
| Partir ahora los demás servicios grandes | DP-8 |

**Reportado por los exploradores el 31/10/2026 y no validado** (fuera de esta revisión; una segunda revisión los valida si lo pides):

- **Backend:**
  - `ChangeLogsController` documenta un `403` y responde una página vacía.
  - `@Setter id` en las clases base, que los ensambladores de `procurement` llaman en `toEntity`.
  - `procurement` declara `iam::acl` sin usarlo.
  - El Javadoc de `Vehicle.correctVin` está desactualizado.
  - `NaturalKeyTakenException` está repetida, y `TaxIdTypeRules` duplicada.
  - `PermissionCodes` lleva conjuntos con reglas.
  - El registro de eventos de Modulith no se usa.
  - `ProtectedAccounts` es una dependencia que Modulith no ve.
  - El alcance `SYSTEM` en la base sale de cualquier rol global.
  - Las pruebas con MockMvc no recorren el `doBegin` real.
- **Frontend:**
  - 12 stores no exponen `loading`.
  - Dos desvíos de la skill: `downloadDocument` y la fachada de `location` en la bitácora.
  - `domain/model/commands` y los espacios de `es.json` están mezclados.
  - Hay comentarios fuera del tono de la casa.
  - `reuseExistingServer` está en el 4200.
  - El simulador tiene 7 434 líneas.
  - Material y el núcleo de Angular van en versiones distintas.
  - No hay CSP ni HSTS en `_headers`.
  - Las medidas del cuerpo de DESIGN.md no coinciden con sus revisiones.
- **Diccionario y convenciones:**
  - las listas cerradas en mayúsculas frente a D-133;
  - el formato de los montos y los nombres del período;
  - la cita de D-39 para el idioma;
  - §14.7, la concurrencia, como pendiente aunque ya está decidida;
  - el «hoy» del nombre del archivo.

---

## 10. Riesgos y cómo se cubren

| Riesgo | Cómo se cubre |
|---|---|
| Alguien cierra una cuenta ajena | Solo frena el contador de los navegadores sin historia. La persona entra desde su navegador habitual, que tiene su propio contador (DP-1) |
| Una persona entra desde un navegador nuevo justo durante un ataque a su cuenta | Espera, como máximo 15 minutos por intento, o usa su equipo habitual. Es el costo aceptado de DP-2 |
| Un ataque repartido entre muchas IP | Comparte el contador de la cuenta: unos 100 intentos por día como máximo. El aviso lo hace visible |
| Se roba una cookie `MM_DEVICE` | No es una credencial: solo da el contador de ese dispositivo, también frenado. Sin la clave no se fabrica |
| Una ráfaga satura la instancia | El tope de DP-4 responde `503` sin ocupar la CPU ni el pool, y con S3 la regla de Cloudflare frena antes |
| Los pedidos simultáneos pasan el freno | La reserva va antes de comparar, bajo `FOR NO KEY UPDATE` (`SignInThrottleConcurrencyTests`) |
| El fallo se deshace y no cuenta | Se cuenta en la transacción 1, que confirma antes de comparar |
| La tabla guarda datos personales para siempre | HMAC con clave y purga (DP-3) |
| S3 deja la API sin servicio | El filtro nace apagado, la función manda el secreto antes de encenderlo, y se aceptan dos secretos para rotar |
| Alguien descubre el secreto de origen | Solo saltaría a Cloudflare; el freno y los permisos siguen. Se rota con el procedimiento de S3 |
| El chequeo de salud de DigitalOcean queda bloqueado | `/actuator/health` queda fuera del filtro, y la prueba lo fija |
| V88 falla en producción o en tu base | La acción 1 en las tres bases |
| S11 rompe una lectura de compras | Es un refactor sin cambio de JSON: lo cubren las 52 clases de prueba, la sonda y una revisión por slice |

---

## 11. Tamaño y ruta crítica

| Fase | Slices | Tamaño |
|---|---|---|
| F1 | S1 (S), S2 (S), S3 (M), S4 (L), S5 (S) | L |
| F2 | S6 (S), S7 (S) | S |
| F3 | S8, S8b, S9, S10 | S |
| F4 | S11 (M), S12 (M), S13 (S); S14 sin fecha | M |

**Ruta crítica:** DP-1 a DP-4 y DP-11 → la clave del freno → S2 → S4 → S5. S1 no espera a nada, y S3 corre en paralelo.

**Orden recomendado:**
1. S1;
2. tus aprobaciones;
3. S2, S4 y S5, con S3 en paralelo;
4. S6 y S7;
5. la guarda de S11 y F3;
6. la traducción de S11, antes de las facturas;
7. S12, antes del paso 9;
8. S13.

---

## 12. Cuándo la revisión queda cerrada

- H1 a H9 están cerrados, o decididos fuera con tu visto bueno (sección 9).
- El diccionario v1.34 tiene D-144 a D-147 y las correcciones de DP-11, y los documentos de H8 dicen lo que hace el código.
- Pasan:
  - la suite completa del backend sobre `erp_mm_db_verify`, con las dos cuentas;
  - las unitarias del frontend;
  - el e2e completo, con las capturas de S6 revisadas.
- Las ocho sondas pasan en dos corridas seguidas, con los casos de S5, sin omitidas.
- El backend responde `404` a un pedido directo a su URL de DigitalOcean, y la aplicación funciona por `dev.erp.millennium.ky`.
- `ForeignAclStaysInTranslationLayerTests` está en cero, y S11 y S12 están antes de las facturas y del paso 9, respectivamente.
- `retroalimentacion/` tiene la revisión de este cierre, con cada hallazgo como **error** o **aprendizaje**.

---

## 13. Auditoría del plan (31/10/2026)

Una auditoría adversarial leyó la versión 1 contra el código, y se verificó cada punto antes de aceptarlo.

**Lo que cambió:**

| # | Punto de la auditoría | Veredicto | Dónde quedó |
|---|---|---|---|
| A1 | Con la llave solo por correo y sin decaimiento, cualquiera cerraba una cuenta para siempre, incluida la del único superadministrador (D-70) | Aceptado, bloqueante | DP-1 y DP-2. La versión 2 lo resolvió con correo más IP; la versión 3, con cuenta y dispositivo (sección 14) |
| A2 | Leer «no bloqueado» antes de comparar y contar después deja pasar las ráfagas, y la regla vivía en dos lugares | Aceptado, bloqueante | S4: reserva bajo `FOR NO KEY UPDATE` antes de comparar |
| A3 | BCrypt dentro de la transacción satura 1 vCPU | Aceptado | DP-4 y el flujo de S4 |
| A4 | Las pruebas `@Transactional` no prueban que el fallo se confirme | Aceptado | Sección 4 y S4 |
| A5 | `UserCommandServiceImplTest` construye el servicio y su doble de `HashingService` | Aceptado | S1 y S4 |
| A6 | El simulador no tenía el inicio de sesión, el error no lleva parámetros y `ResponseEntityAssembler` descarta el `context` | Aceptado | S2 y S4 |
| A7 | La guarda fallaba el primer día por `documents::acl`, y los records internos no pueden llegar a vistas públicas | Aceptado | DP-7 y S11 |
| A8 | `transitionTo` seguía público; una política del dominio no devuelve códigos; la partición no queda por agregado | Aceptado | S12 |
| A9 | El PDF enviado no lleva `Content-Disposition` | Aceptado | H9 y DP-10 |
| A10 | Un SHA-256 sin clave se revierte, y sin purga se guarda para siempre | Aceptado | DP-3 |
| A11 | El `UPSERT` saltaba D-42, y el orden de bloqueo quedaba al revés del restablecimiento | Aceptado | S4 |
| A12 | `UserCommandServiceImpl` pasaría de 300 líneas | Aceptado | H7 y DP-8 |
| A13 | El cambio de contraseña y el registro seguían abiertos, y con `native` Tomcat toma la IP de la cabecera | Aceptado | S4, DP-12 y S3 |
| A14 | S6 incompleto | Aceptado | S6 |
| A15 | S7 incompleto, con una prueba de migración imposible | Aceptado | S7 y la acción 1 |
| — | Los datos menores: conteos, archivos de Mockito, sondas del README, `iam.md:291`, «doce permisos», `DeadlockRetryTests`, cabeceras de V79 y V83, números de las decisiones, OpenAPI y §9 | Aceptados | En sus secciones |

**Lo que se rechazó:**

| Punto | Por qué |
|---|---|
| «La línea 388 del README del backend no dice que `procurement` no existe» | Sí lo dice: «`procurement` todavía no existe y se construye en el orden que fija la §3 del diccionario» |

---

## 14. Decisiones recomendadas tras la revisión crítica (versión 3)

| Cambio | Antes (versión 2) | Ahora (versión 3) | Por qué |
|---|---|---|---|
| La llave del freno (DP-1) | Correo más IP, con S3 como requisito | Cuenta y dispositivo, con la cookie `MM_DEVICE` | La llave por IP no frenaba un ataque repartido entre muchas IP, el más común contra un inicio de sesión expuesto, y dependía de una IP confiable. App Platform no permite filtrar la entrada por IP ([tablero de ideas de DigitalOcean](https://ideas.digitalocean.com/app-platform/p/ip-whitelisting-for-apps), [comunidad](https://www.digitalocean.com/community/questions/firewall-for-app-platform)) |
| El secreto de origen (DP-13) | Requisito del freno | Refuerzo propio, en paralelo y fuera de la ruta crítica, con rotación, comparación en tiempo constante y la IP solo para el registro | Hace efectivas las protecciones de Cloudflare y oculta el backend, sin trabar el cierre de H1 |
| Swagger en producción | No se trataba | Apagado en `prod` (S3) | Hoy se lee la documentación completa de la API en la URL pública de DigitalOcean |
| El tope de BCrypt (DP-4) | Cuatro, con 2 s | Dos, con 3 s | En 1 vCPU, más de dos solo reparten el procesador |
| El momento de S11 y S12 (DP-7, DP-8) | Cuando se pudiera | La guarda ahora, la traducción antes de las facturas y `fleet` antes del paso 9 | Se corrige lo que la próxima funcionalidad copiaría, antes de ella |
| Dónde va la tabla (DP-11) | §2.7 | El bloque 2 de `iam` | El dueño es `iam`, y el mapa reparte las tablas por dueño |
| Los secretos | Dos, los dos en la ruta crítica | `IAM_SIGN_IN_KEY` para el freno; `ORIGIN_SECRET` para el refuerzo | Cada uno cuando lo necesita su slice |

**Ajustes de la versión 3.1** (aprobados con el plan):

1. **Un acierto vuelve a cero solo el contador usado.** Si también volviera a cero el de la cuenta, cada entrada diaria del dueño desde su equipo regalaría cinco intentos a quien prueba claves contra esa cuenta.
2. **La clave se llama `IAM_SIGN_IN_KEY`.** Firma la cookie y protege la tabla. Rotarla vuelve todos los navegadores a «sin historia» y pone los contadores en cero, y así se documenta.
3. **La regla de Cloudflare es holgada** (30 por minuto por IP): es para las ráfagas grandes, no para la fuerza bruta.
4. **La pantalla habla de caracteres; el diccionario, de bytes.**

---

## 15. Estado de la implementación (31/10/2026)

Construida el 31/10/2026, cada slice en su rama `feature/<contexto>` y en su worktree hermano, e integrada ese día en la rama `integration/review-2026-10-31` de los dos repositorios, que sale de la base de este plan (backend `1e207eb7`, frontend `6645341c`):

- **Backend**, cabeza `7f8e122e`: merges `55ede253` (`feature/iam-sign-in-throttle`), `5355138d` (`feature/procurement-acl-translation`), `403a7903` (`feature/fleet-status-path`) y `7f8e122e` (`feature/location-currency-decimals`). V87 es de S4 y V88 de S7, como en la sección 6.
- **Frontend**, cabeza `308d9ae9`: merges `a3e89e30` (`feature/iam-sign-in-throttle`) y `308d9ae9` (`feature/location-currency-decimals`).

### 15.1 Commits por slice

| Slice | Backend | Frontend y otros | Estado |
|---|---|---|---|
| S1 | `13be744f`: el hash de relleno, la misma respuesta y el mismo tiempo para un correo inexistente y `SYSTEM` | — | Construido |
| S2 | — | `a1b01aa4`: los mensajes con los minutos y `SIGN_IN_BUSY`; `da347a6c`: el simulador del inicio de sesión y `e2e/sign-in.spec.ts` | Construido |
| S3 | `ea805b2c`: `OriginCheckFilter`, `ORIGIN_SECRET` y `ORIGIN_SECRET_PREVIOUS`, `/actuator/health` fuera del filtro, `X-MM-Client-IP` solo para el registro y Swagger apagado en `prod` | `c4176ee3`: la función de Pages manda el secreto y la IP | Construido. Encenderlo es la acción 3 de la sección 3, en el orden de la sección 7, paso 5 |
| S4 | `9f88a6b3`: V87, `sign_in_throttles`, el agregado, su persistencia y `hand-over-database.sql`; `8aff79fb`: `SIGN_IN_THROTTLED` `429` y `SIGN_IN_BUSY` `503`; `7b8583cc`: el freno por cuenta y por navegador, `MM_DEVICE`, la purga, `SignInCommandServiceImpl` y `SignInThrottleCommandServiceImpl` | — | Construido. `UserCommandServiceImpl` queda en 241 líneas de código |
| S5 | — | `9df6208d`: los dos casos de `e2e-live/security.live.spec.ts` | Escrito, sin correr (15.4) |
| S6 | `17072d0a`: de 8 caracteres a 72 bytes, la anotación `@PasswordLength` y la contraseña del arranque validada | `11d31aa5`: `atMostUtf8Bytes(72)` y la pista; las tres capturas de Registro, regeneradas | Construido |
| S7 | `1917101f`: `Currency` hasta 2, `@Max(2)` y los textos; `e6aa2094`: V88, que se detiene si una moneda tiene más de 2 y deja el `CHECK` en 0..2; `8799bc96` y `4e2379bd` (15.3) | `c86fb25d`: el diálogo, hasta 2; `7c8a4fa5`: sin el texto de `PURCHASEORDER_CURRENCY_SCALE` | Construido, con dos desvíos (15.3) |
| S8 | — | Documentación: `3c75e96`, `9308f09`, `fcf73cb` y `8b5a77f`; esta pasada, de `af73275` en adelante | Hecho |
| S8b | `d34cba7c`: las líneas viejas del README | `3a31e2ef`: README, `app.routes.ts` y `playwright.live.config.ts` | Hecho |
| S9 | — | Las dos copias de las skills, fuera de git e idénticas: los once, la excepción de `kept_files` en la regla 6.2, la del fin de la sesión en `frontend-feature`, la guarda de S11, `sign_in_throttles` como excepción de la regla 4.2, el filtro del origen y, el 31/10/2026, los veintiocho pares | Hecho |
| S10 | — | Diccionario v1.34: `3802749` (D-144 a D-147) y `12ddc57` (los permisos de seguridad como los tiene el código), aprobados por el responsable | Hecho |
| S11 | `2173a1ad`: la guarda `ForeignAclStaysInTranslationLayerTests`; `d77e82ea` (`PostalAddress`), `405e7278` (`OwningCompany`, `IssuerData`), `71610dcf` (`NamedReference`, `IssuingScope`, `BrandReference`), `933f0026` (`OrderCurrency`), `ab54d29e` (`LineModel`) y `2c8ec038` (`AttentionContact`, `TaxIdTypeReference`) | — | Construido; la guarda, en cero (`KNOWN_DEBT` vacía), antes de las facturas |
| S12 | `44c41c1f`: `StatusPathPolicy` con su `Verdict`, y `Vehicle.transitionTo(Long, StatusPath)` se niega ante un camino no declarado; `9694c24a`: `VehicleMileageReadingCommandServiceImpl`; `9bf56bf9`: las piezas `VehicleMovements` y `VehicleIdentifierCorrections` | — | Construido, antes del paso 9. `VehicleCommandServiceImpl` baja de 527 a 312 líneas de código |
| S13 | — | `0aae457b`: navegan las vistas; en el store, solo `endLocalSession` | Construido |
| S14 | — | — | Sin fecha (DP-8) |

### 15.2 Pruebas

| Qué | Dónde | Resultado |
|---|---|---|
| Suite completa del backend | La integración (`7f8e122e`), sobre `erp_mm_db_verify`, conectada como `postgres`, el superusuario, por decisión del responsable | 2 900 pruebas; 63 fallos en 39 métodos, todos de pruebas que solo pasan con la cuenta de la aplicación: Row Level Security y permisos que necesitan `erp_mm_app`. Es el mismo conjunto que la base, `develop` `1e207eb7`, corrida igual: 2 807 pruebas, 63 fallos y 2 errores. **Ningún fallo nuevo** |
| Unitarias del frontend | La integración (`308d9ae9`) | 202 archivos, 2 648 pruebas, 0 fallan |
| Build del frontend | La integración | Compila. Su único aviso, el presupuesto de estilos de `purchase-order-form.scss` (6,33 kB sobre 5 kB), es de un archivo que este cierre no cambió |
| e2e simulado completo | La integración, en el puerto 4310 | 1 418 pasan, 73 se omiten por proyecto, 0 fallan |

Antes de integrar, en la rama del frontend, las unitarias dieron 2 647 en verde y el e2e completo, 1 418 pasan, 73 omitidas y ninguna falla.

**No corrió todavía:**

- la suite del backend con las dos cuentas, `erp_mm_app` para la aplicación y `erp_mm_owner` para Flyway, la que exige la sección 12 y la que esas 63 pruebas necesitan;
- las ocho sondas en vivo, con los dos casos de S5: necesitan la contraseña de la cuenta de verificación.

### 15.3 Desvíos del plan

**La corrección de H3.** Al construir S7 se vio que `procurement` ya rechazaba con `422 PURCHASEORDER_CURRENCY_SCALE` una orden en una moneda de más de 2 decimales (`PurchaseOrderReferences.java:120`, en `1e207eb7`), desde que existen las órdenes (22/10/2026, `55edee69`). No había, entonces, el redondeo silencioso que dice H3 en la sección 2: PostgreSQL nunca recibía esos montos. El defecto real era que una moneda así se podía crear y no se podía usar en una orden. D-146 sigue valiendo, porque alinea el catálogo con la escala de los montos y corta el problema donde nace. La lección, para la revisión misma, está en [retroalimentacion/2026-10-31-revision.md](retroalimentacion/2026-10-31-revision.md).

**S7, dos cambios a lo que la sección 5 pedía:**

1. **La prueba de la detención de V88 salió** (`8799bc96`). Volver a correr V88 sobre una moneda de 3 decimales exigía quitar antes `currencies_decimal_places_range_ck`, y eso solo lo puede el dueño de la tabla: las pruebas corren como `erp_mm_app` y fallaba con «must be owner of table currencies». La detención la cubre la acción 1 en las tres bases, que, según `e6aa2094`, se hizo antes de integrar. `CurrencyDecimalPlacesMigrationTests` conserva la definición del `CHECK`, el alcance `SYSTEM` primero, y 3 decimales rechazados y 2 aceptados.
2. **La revisión de `procurement` salió** (`4e2379bd`, y en el frontend `7c8a4fa5`). Con V88, el dominio y la API en 0..2, `PURCHASEORDER_CURRENCY_SCALE` ya no se alcanza. Salieron la revisión, su fábrica de errores, sus textos y su prueba, que por la misma razón de arriba no podía correr como la cuenta de la aplicación. `PURCHASEORDER_PRICE_SCALE` sigue.

**Decisiones tomadas al construir**, sin cambiar lo que el responsable aprobó:

| Slice | Decisión | Por qué |
|---|---|---|
| S11 | `notifications::acl` queda fuera de la guarda, junto con `documents::acl` | Es también un contrato de un módulo de la plataforma (D-143), que todos los contextos usan tal cual |
| S11 | El tipo de documento que una vista o la hoja imprimen pasa a `TaxIdTypeReference`, un record de la capa de traducción, y no a `TaxIdTypeRules`, como decía S11f | Lo que se imprime es el identificador, el código y el nombre. `TaxIdTypeRules` sigue siendo lo que la regla del proveedor revisa de un tipo |
| S12 | El `422` del camino sigue antes del `403` de su permiso | `VehicleMovements.move` revisa el camino en memoria, y `VehicleCommandServiceImpl` decide después el permiso del camino (D-100): las respuestas conservan el orden que siempre tuvieron |
| S4 | La cookie `MM_DEVICE` se manda después de que confirma la transacción del acierto | Un rollback no deshace una cabecera ya escrita |
| S4 | Una entrada con contraseña provisional confía en el navegador | Es un acierto como cualquier otro: la contraseña coincidió. La sonda de S5 entra así con su cuenta de la corrida antes de frenarla |

### 15.4 Lo que queda

1. **La suite del backend con las dos cuentas** sobre `erp_mm_db_verify`, sin fallos (sección 12).
2. **Las ocho sondas en vivo, en dos corridas seguidas, con los dos casos de S5**, contra el backend con V87 y V88, cuando haya la contraseña de la cuenta de verificación.
3. **Las acciones del responsable en el entorno** (sección 3): `IAM_SIGN_IN_KEY` como secreto en App Platform antes de desplegar S4, porque fuera de `dev` la aplicación no arranca sin ella; y, durante S3, `ORIGIN_SECRET` en Pages y en App Platform, `IAM_ORIGIN_CHECK_ENABLED` encendido cuando la función ya lo mande, la comprobación con `curl` —la URL de DigitalOcean responde `404` y `dev.erp.millennium.ky` funciona— y la regla de limitación de Cloudflare. **Verificado el 01/11/2026 desde fuera**, sin sesión: la dirección propia del backend en App Platform responde `404` a una ruta de la API (`/api/v1/authentication/sign-up/status`), a `/v3/api-docs` y a `/swagger-ui.html`, y `200` a `/actuator/health`; la misma ruta de la API responde `200` por `dev.erp.millennium.ky`, y la pantalla de inicio de sesión carga. El filtro del origen está encendido y la función de Pages manda el secreto correcto; como el filtro y Swagger apagado en `prod` llegan con `ea805b2c`, el entorno corre el código del cierre, que no arranca sin `IAM_SIGN_IN_KEY`. La regla de limitación de Cloudflare no se comprueba desde fuera sin forzar el inicio de sesión: la confirma el responsable.
4. **Los push, que pide el responsable.** Hechos: el 01/11/2026 el entorno corre ese código (punto 3). El 31/10/2026 las integraciones pasaron a `develop` por avance directo: backend `7f8e122e` y frontend `308d9ae9`, los mismos commits que se verificaron arriba. Los worktrees y las ramas de los slices se borraron. Como todo está ya en `develop`, el orden de la sección 7 se reduce a este:
   1. **El frontend** (`308d9ae9`). Conoce los códigos nuevos, valida los bytes y los 2 decimales, y su función solo manda cabeceras cuando existe `ORIGIN_SECRET`, así que no depende del backend.
   2. **Antes del backend:** `IAM_SIGN_IN_KEY` en App Platform; la consulta de monedas en producción (V88); y, si se puede, una corrida del CI con las dos cuentas sobre una rama subida, a pedido desde *Actions*, antes de publicar `develop`, que despliega en cada push.
   3. **El backend** (`7f8e122e`). El filtro de origen nace apagado y Swagger queda apagado en producción.
   4. **El refuerzo:** `ORIGIN_SECRET` en App Platform y en Pages, la función desplegada con él, `IAM_ORIGIN_CHECK_ENABLED` encendido, la comprobación con `curl` y la regla de Cloudflare.
   5. **La documentación.**
5. **La revisión de este cierre** en `retroalimentacion/`, con cada hallazgo como error o aprendizaje: [2026-10-31-revision.md](retroalimentacion/2026-10-31-revision.md).

Con los puntos 1 a 3 en verde, la revisión cumple la sección 12.

### 15.5 Cierre (01/11/2026)

La revisión cumple la sección 12:

- **La suite del backend con las dos cuentas:** la corrida del CI del push de `7c5c6066` a `develop`, que se conecta como `erp_mm_app` y migra como `erp_mm_owner`, terminó en verde; lo confirmó el responsable.
- **Las ocho sondas, en dos corridas seguidas, con los casos de S5:** 74 pasan, 0 fallan y 0 se omiten en cada una, contra el backend `e5f8bcd3` y el frontend `24b0074f`, sobre `erp_mm_db_verify` ([sondas del 01/11/2026](retroalimentacion/2026-11-01-sondas.md)). Antes encontraron dos defectos, los dos corregidos: el patrón del mensaje del freno en la sonda (`24b0074f`) y el orden de las líneas de una orden de compra (`e5f8bcd3`, PRC-R30).
- **Al backend solo por Cloudflare:** verificado el 01/11/2026 (15.4, punto 3).
- **H8:** las dos líneas de `contextos/iam.md` que quedaban, corregidas (`0be488e`).
- Lo demás de la sección 12 ya se cumplía el 31/10/2026 (15.1 a 15.3).

Fuera de la sección 12 y con el responsable queda la regla de limitación de Cloudflare sobre el inicio de sesión (sección 3, acción 3).
