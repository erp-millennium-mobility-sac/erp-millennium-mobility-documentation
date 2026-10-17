# Contexto `iam` — ERP Millennium Mobility

**Actualizado:** 16/10/2026 (paso 3: modelo de acceso) · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.17 (§2.5, §2.6, §3, §5.1 a §5.13, D-32 a D-34, D-38 a D-43), la propuesta de diccionario v1.18 (D-55 a D-72), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md) §4, §5, §6.2 y §11, [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md) §2.2, §7.2, §9 y §14, [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) P1 y P2, [orden de implementación](../orden-de-implementacion-millennium-mobility.md), los contratos del modelo de acceso (rondas 1 a 5, del 15/10/2026 y el 16/10/2026) y el código de la rama `feature/access-model` del backend (`6a7d9b1`) y del frontend (`1a341ab`) al 16/10/2026, con la ronda 6. Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** Lo que cita una sección (§n) o una decisión (D-n) del diccionario está decidido. Las decisiones del modelo de acceso que el usuario aprobó el 15/10/2026 y el 16/10/2026 se citan con el número que les da la propuesta del diccionario v1.18 (D-55 a D-72); hasta que el diccionario las registre, la fuente es esa propuesta. "Hoy" describe el código de `feature/access-model`.

---

## 1. Qué es y qué no es

`iam` responde cuatro preguntas: **con qué cuenta entra una persona**, **qué roles tiene esa cuenta y sobre qué empresa, sede y marca**, **si la cuenta está bloqueada** y **si su sesión sigue abierta**. También es quien da acceso a los colaboradores, otorga y retira roles, bloquea cuentas, restablece contraseñas y orquesta lo que la salida y el regreso de un colaborador le hacen a su cuenta. Es la puerta del sistema: si una regla de este contexto falla, falla la protección de todos los demás.

**Tablas que posee**

| Tabla | Qué guarda | Estado |
|---|---|---|
| `users` | La cuenta: correo, contraseña cifrada, si debe cambiarla, último acceso (§5.4) | Construida (V1) |
| `roles` | Los siete roles del sistema con su alcance (§5.5, D-55) | Construida (V1; `ADMIN_ACCESOS` en V30), sin `company_id` |
| `user_roles` | Qué rol tiene cada cuenta, sobre qué empresa, sede y marca, desde cuándo, hasta cuándo y, si se retiró, cuándo y por qué (§5.7) | Construida (V1; alcance en V27) |
| `user_access_suspensions` | Los períodos en que una cuenta no puede entrar, en dos líneas de tiempo (§5.4, D-62) | Construida (V28; las dos líneas en V32) |
| `spring_session`, `spring_session_attributes` | Las sesiones abiertas, en el servidor (§5.12, D-33) | Construidas (V3). Tablas técnicas de la librería |

**Tablas diseñadas que todavía no existen**

| Tabla o columna | Para qué | Cuándo |
|---|---|---|
| `permissions`, `role_permissions` | Las acciones concretas y qué rol incluye cada una (§5.6) | Con la matriz de permisos, antes del paso 5 (orden, ajuste 3) |
| `roles.company_id` | Roles propios de una empresa (§5.5, D-7) | Paso 2 según el orden de implementación; no existe todavía |

Que `user_roles` y `user_access_suspensions` sean de `iam`, y `employees` de `organization`, es el reparto del bloque 3b que propone el mapa (§3, decisión pendiente 4). El diccionario asigna el bloque 3b a `organization` e `iam` sin separar tablas; el código sigue la propuesta.

**Lo que parece suyo y no lo es**

- **Los colaboradores** (`employees`) son de `organization`, incluida la columna `employees.user_id` que une al colaborador con su cuenta (§5.3). `iam` la escribe solo pidiéndole a `organization`, por su fachada, que vincule un colaborador a una cuenta. Ver [organization.md](organization.md).
- **El perfil** (nombre, documento, teléfono, foto, puesto) va en `employees`, no en `users` (D-4, D-10). La cuenta guarda solo lo necesario para entrar.
- **La fecha de salida y la de regreso** son datos del colaborador, de `organization`, que valida sus reglas de fechas. `iam` decide quién puede registrarlas, le pide a `organization` que las registre y cambia la cuenta en la misma transacción (mapa §11, decisión 2, opción A).
- **La frontera entre empresas en la base** (Row Level Security) es de `shared`: las políticas cubren tablas de `organization` y de `iam`. `iam` aporta el alcance de la cuenta que la alimenta.
- **Los clientes** no tendrán cuenta aquí: la aplicación de clientes tendrá su propia tabla de identidades (§5.1).

**Vocabulario (D-39)**

| Palabra | Qué es | Dónde vive |
|---|---|---|
| Rol | El conjunto de permisos de una cuenta, con su alcance | `iam` (`roles`, `user_roles`) |
| Puesto | El trabajo del colaborador: asesor, mecánico, contador. Es informativo y no da permisos | `organization` (`employees.position`) |
| Perfil | Los datos personales del colaborador (D-4) | `organization` (`employees`) |

"Cargo" no se usa. El comentario de cabecera de la migración V1 todavía dice "cargos"; una migración aplicada no se edita.

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Vigente | Una asignación que no se retiró y cuya caducidad no llegó. El acceso se corta exactamente al caducar, aunque el proceso que la cierra todavía no haya pasado (D-64) | "Abierta", que no mira la caducidad |
| Ver una empresa | Leer sus datos. Un rol de alcance sistema ve todas, también las que se creen después | Administrarla |
| Administrar una empresa | Registrar y modificar sus colaboradores, y registrar sus salidas y regresos. Lo hacen el superadministrador, en todas, y el administrador de esa empresa. El administrador de accesos ve todas y no administra ninguna (D-57) | Ver una empresa |
| Manejar cuentas | Dar acceso, otorgar y retirar roles, bloquear y levantar bloqueos, restablecer contraseñas. Lo hacen el superadministrador y el administrador de accesos (D-55, D-57) | Administrar una empresa |
| Rol reservado | `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA`: solo un superadministrador los otorga y los retira (D-55) | — |
| Cuenta protegida | Una cuenta con un rol reservado vigente, en cualquier empresa. Solo la maneja un superadministrador (D-55). La lista de cuentas lo dice en `managedBySuperadmin` (IAM-R51) | — |
| Bloqueo | Un período en que la cuenta no puede entrar (`user_access_suspensions`). Tiene un motivo, y el motivo dice su línea de tiempo: laboral o de seguridad (D-62) | La salida del colaborador, que es un dato de `organization` |
| Dar acceso | Crear la cuenta de un colaborador o vincularlo a la que la persona ya tiene. No otorga roles (D-58) | Otorgar un rol |

**Cuenta y colaborador.** Son dos cosas distintas. Quien trabaja para dos empresas del grupo tiene dos colaboradores, uno por empresa, y una sola cuenta (§5.3). Un mecánico puede ser colaborador sin cuenta. Las únicas cuentas sin colaborador son `SYSTEM`, que no es una persona, y el primer superadministrador hasta que se vincula a su perfil (§5.2).

---

## 2. Ciclo de vida de cada agregado

`iam` tiene dos agregados: la cuenta (`User`), que lleva dentro sus asignaciones de rol, y el bloqueo (`AccessSuspension`), con su propio repositorio. El rol es un catálogo que se lee.

### Cuenta (`User`)

| Momento | Qué pasa |
|---|---|
| Nace | Al dar acceso a un colaborador (F5), con una contraseña provisional que genera el sistema y se muestra una sola vez, o por el registro público, hoy sin perfil (F4). `SYSTEM` nace en la migración y el primer superadministrador al arrancar (F1, D-34). Ya no hay alta suelta de cuentas: `POST /users` se quitó (D-38) |
| Contraseña provisional | Debe cambiarse en el primer ingreso (§5.4). Mientras tanto la cuenta solo consulta quién es y cambia su contraseña (F3), y no recibe un rol reservado (D-72) |
| Cambia la contraseña | Entre 8 y 72 caracteres, distinta de la actual; cierra las demás sesiones (§5.4, §5.12) |
| Se restablece la contraseña | Por un superadministrador o el administrador de accesos: una provisional nueva, que debe cambiarse, y todas las sesiones cerradas (F8, D-63) |
| Se vincula a un colaborador | Al dar acceso, a lo sumo a un colaborador por empresa (§5.3) |
| Se bloquea | Por períodos, con el agregado `AccessSuspension` (abajo) |
| Queda fijo | El correo (no hay operación para cambiarlo), el autor y el momento de creación |
| Nunca pasa | Se borra (§5.4). No hay `DELETE`, y la cuenta de la aplicación no tiene permiso de borrar en `users` (V19). `SYSTEM` no inicia sesión, no se vincula, no recibe roles, no se bloquea y no se le restablece la contraseña (§5.9) |

### Rol (`Role`)

Los siete roles del sistema llegan en las migraciones, marcados como propios del sistema, con su alcance (§5.5): `SUPERADMIN` y `ADMIN_ACCESOS` de alcance `SYSTEM`; `ADMIN_EMPRESA`, `OPERACIONES` y `CONTABILIDAD` de alcance `COMPANY`; `ASESOR_VENTAS` y `TALLER` de alcance `BRANCH`. `ADMIN_ACCESOS` llegó en V30 (D-55): da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas en todas las empresas, y no registra colaboradores.

`SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA` son **reservados al superadministrador**: solo él los otorga y los retira (D-55).

Hoy los roles solo se leen (`GET /roles`, el arreglo completo). Crear roles desde la pantalla (D-7) espera a `roles.company_id` y a la matriz de permisos; quién crea los de una empresa se decide con esa matriz. El frontend ya tiene las pantallas de roles propios y permisos (Seguridad › Roles y permisos), que llaman a `/iam/roles` y `/iam/permissions`; quedan fuera del menú hasta que el backend sirva la matriz, y esas rutas tendrán que seguir las convenciones (§2.2) cuando se construyan. El alcance de un rol se fija al crearlo y no cambia (§5.5, regla 4). Un rol del sistema no se edita ni se borra.

### Asignación (`RoleAssignment`)

| Momento | Qué pasa |
|---|---|
| Se otorga | La otorga un superadministrador o el administrador de accesos (F6). El alcance es el que pide el nivel del rol: uno de sistema, sin empresa; uno de empresa, con su empresa; uno de sede, con su empresa y su sede. Cualquiera que no sea de sistema puede limitarse a una marca de su empresa, salvo `ADMIN_EMPRESA`, que nunca se limita a una marca (D-59). Puede llevar caducidad, siempre posterior al momento en que se otorga. Guarda quién la otorgó (`granted_by`, que pone el sistema con la cuenta de la sesión) y cuándo |
| Se retira | Con uno de cuatro motivos (D-41): `retiro`, cuando un administrador la quita; `baja_colaborador`, cuando la persona sale de esa empresa; `baja_cuenta`, cuando la salida cierra la cuenta; `caducidad`, cuando vence. Nunca se borra (§5.7, §5.8) |
| Caduca | Deja de valer exactamente en su `expires_at`. Un proceso programado, que corre con alcance `SYSTEM` cada `IAM_ROLE_EXPIRY_CHECK_INTERVAL` (5 minutos por omisión), la cierra con fecha de retiro igual a su caducidad y motivo `caducidad`. Si antes se otorga otra vez el mismo rol sobre el mismo alcance, en ese momento se cierra la vencida con esa misma llave, y solo esa (§5.8, D-64) |
| Nunca pasa | Dos asignaciones vigentes del mismo rol sobre el mismo alcance; un rol de sistema con empresa, sede o marca; un rol de sede sin sede; una cuenta con `ADMIN_ACCESOS` y `ADMIN_EMPRESA` vigentes a la vez (D-71) |

### Bloqueo (`AccessSuspension`)

Un bloqueo es un período `[inicio, fin)`; sin fin, sigue abierto. Su motivo es uno de cuatro (§5.4) y decide su línea de tiempo (D-62):

| Línea | Motivos | Quién lo abre |
|---|---|---|
| Laboral | `vacaciones`, `licencia` | Un superadministrador o el administrador de accesos, a mano (F7) |
| Laboral | `baja` | Solo el sistema, al registrar la salida del último colaborador activo de la cuenta (F9, D-69) |
| Seguridad | `seguridad` | Un superadministrador o el administrador de accesos, a mano (F7) |

Dentro de una línea, dos bloqueos no se cruzan, ni siquiera en el pasado; entre líneas sí. Basta un bloqueo en curso, de cualquier línea, para que la cuenta no entre.

Dónde está cada bloqueo no se guarda: se calcula al leerlo, de sus fechas y de su cierre.

| Estado | Cuándo | Qué se puede hacer |
|---|---|---|
| `SCHEDULED` | Empieza después | Cancelarlo: conserva su período, registra quién y cuándo, y deja de contar |
| `IN_FORCE` | Ya empezó, no terminó y no se canceló | Levantarlo: termina en ese momento. Una `baja` no se levanta a mano |
| `ENDED` | Llegó su fin, el planeado o el del momento en que se levantó | Nada |
| `CANCELLED` | Se cerró antes de empezar | Nada |

| Momento | Qué pasa |
|---|---|
| Se abre | Empieza ahora o después, nunca antes de la petición (D-61). El fin, si lo tiene, es posterior al inicio. Si ya está en curso, se cierran en el acto las sesiones de la cuenta |
| Empieza solo | Un bloqueo programado surte efecto al llegar su inicio: el filtro de sesión vuelve a leer en cada petición si la cuenta está bloqueada, y desde ese momento la trata como desconectada |
| Se cierra | Siempre en el momento en que alguien lo cierra, sin fecha elegida: uno en curso se levanta; uno programado se cancela (D-61) |
| La `baja` se cierra | Solo con el regreso del colaborador o con un nuevo acceso que vincula la cuenta (F5, F9). Un bloqueo por seguridad en curso sigue bloqueando la cuenta |
| Queda | Un bloqueo programado sobre una cuenta que después pasó a ser protegida se conserva: se ve en su historia y lo cancela un superadministrador |
| Nunca pasa | Se borra; se cierra dos veces; se cruza con otro de su línea, porque la base lo impide (V32) |

### Sesión

| Momento | Regla (§5.12, D-33) | Hoy |
|---|---|---|
| Se abre | Al iniciar sesión, con un identificador nuevo | Se cumple. Una cuenta bloqueada no inicia sesión |
| Cada petición | Roles, bloqueos y contraseña provisional se leen de la base, no de la sesión | Se cumple: el filtro de sesión vuelve a leerlos y fija el alcance de la cuenta para la petición |
| Se cierra sola | A los 30 minutos sin uso o a las 8 horas desde que se abrió | Se cumple |
| Se cierra por un hecho | Al cerrar sesión; al cambiar la contraseña, las demás; al restablecerla, todas; cuando empieza un bloqueo, todas | Se cumple. La salida que cierra la cuenta también las cierra todas |

La cookie se llama `MM_SESSION` también con Spring Session JDBC en el servidor en marcha: lo prueba `SessionCookieTests`, que resolvió la duda de la ronda 2.

---

## 3. Flujos funcionales

Las pantallas de cuentas son dos: Seguridad › Usuarios (`/iam/users`), para el superadministrador y el administrador de accesos, y las acciones de cuenta de cada fila de Configuración › Colaboradores (`/organization/employees`). Las dos abren los mismos diálogos de `iam`: dar acceso, roles, bloqueos y restablecer contraseña. Ninguna ofrece acciones sobre la propia cuenta, sobre `SYSTEM` ni, al administrador de accesos, sobre una cuenta protegida, que la pantalla reconoce solo por `managedBySuperadmin` (IAM-R51); la API las rechaza igual. Un rechazo que significa que la pantalla quedó vieja —entre otros `ACCESS_SUSPENSION_CLOSED`, `ROLE_ASSIGNMENT_INACTIVE`, `ACCOUNT_MANAGED_BY_SUPERADMIN`, `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`, `EMPLOYEE_ALREADY_TERMINATED` y `EMPLOYEE_ALREADY_ACTIVE`— vuelve a leer la lista o el diálogo afectado, para que los botones digan el estado real.

### F1 · Puesta en marcha y autovinculación (P1, pasos 1 a 7)

- **Quién:** la aplicación al arrancar (D-34) y después el primer superadministrador.
- **Pasos:**
  1. La migración V1 crea los roles y la cuenta `SYSTEM`; V30 agrega `ADMIN_ACCESOS`.
  2. Al arrancar, si ya hay un superadministrador vigente, no hace nada. Si faltan `IAM_BOOTSTRAP_ADMIN_EMAIL` o `IAM_BOOTSTRAP_ADMIN_PASSWORD`, arranca sin superadministrador y lo advierte en el registro de eventos. Si no, crea la cuenta con esa contraseña provisional y le otorga `SUPERADMIN`, todo a nombre de `SYSTEM` y con alcance `SYSTEM` explícito. No pasa por la regla de la contraseña propia (D-72): se crea, no se otorga.
  3. El superadministrador cambia su contraseña (F3), crea la empresa y sus sedes y se registra como colaborador ([organization.md](organization.md), F5).
  4. **Se vincula a sí mismo:** da acceso a su propio colaborador con su propio correo (F5). Es la única autovinculación permitida, y solo mientras no tiene un colaborador activo (D-70): ya vinculado, no se vincula a sí mismo a un colaborador de otra empresa.
  5. Desde ahí puede dar acceso y otorgar roles. Registra al administrador de cada empresa y al administrador de accesos, que son personas distintas (D-71), les da acceso y, cuando cada uno cambió su contraseña provisional (D-72), les otorga `ADMIN_EMPRESA` sobre su empresa o `ADMIN_ACCESOS`.
- **Errores:** si el correo del arranque ya pertenece a otra cuenta, la aplicación no arranca (§5.2). Mientras no está vinculado, dar acceso a otro u otorgar un rol responde `403 ACTOR_EMPLOYEE_REQUIRED`.
- **Resultado:** cada acceso que se otorga desde entonces queda atribuido a una persona con perfil (§5.2).

### F2 · Iniciar y cerrar sesión (P2, D-33)

- **Quién:** cualquier persona con cuenta.
- **Pantalla:** Inicio de sesión (`/iam/sign-in`).
- **Pasos:** la persona escribe correo y contraseña. El sistema guarda el correo sin espacios y en minúsculas y compara la contraseña cifrada. Si coincide y la cuenta está bloqueada, lo dice; si no, registra el último acceso, abre la sesión con un identificador nuevo y devuelve quién es: `id`, `email`, `mustChangePassword`, `roles` (los códigos vigentes en cualquier alcance), `global` y `companies`, cada una con `id`, `name` (el nombre comercial), `timeZone` (la zona de su país) y `administered` (convenciones §9.1). La pantalla la lleva a Inicio, o a Cambio de contraseña si la clave es provisional. Al recargar la página, el frontend pregunta quién es (`GET /authentication/me`) antes de la primera navegación; el `401` de esa pregunta sin sesión es el esperado.
- **Errores:** un correo inexistente, una contraseña equivocada y la cuenta `SYSTEM` reciben la misma respuesta (`401`). Una cuenta bloqueada recibe `403 ACCOUNT_SUSPENDED`, y solo después de que su contraseña coincidió, así la respuesta no le revela nada a quien no la conoce. Sin token CSRF, `403`.
- **Cerrar sesión:** `POST /authentication/sign-out` borra la sesión en el servidor y responde `204`. Si cualquier llamada responde `401` porque la sesión venció, o porque un bloqueo empezó, el frontend vuelve a Inicio de sesión.

### F3 · Primer ingreso y cambio de contraseña obligatorio (§5.4)

- **Quién:** una cuenta con contraseña provisional: la del arranque, una recién creada al dar acceso o una restablecida.
- **Pantalla:** Cambio de contraseña (`/iam/change-password`).
- **Pasos:** mientras la clave es provisional, el backend reemplaza sus roles por un único permiso de cambiar la contraseña: todo lo demás responde `403`, salvo `me` y `change-password`. El frontend la redirige a esta pantalla desde cualquier ruta. La persona escribe la clave actual y la nueva. El sistema revisa que la actual sea correcta, que la nueva tenga entre 8 y 72 caracteres y que sea distinta de la actual.
- **Errores:** clave actual incorrecta (`400`), longitud fuera de rango (`400`), clave igual a la actual (`422`).
- **Resultado:** la cuenta queda liberada, se registra la fecha del cambio y se cierran sus demás sesiones. Desde ahí puede recibir un rol reservado (D-72).

### F4 · Registro público (D-32)

- **Quién:** cualquier persona, solo si `IAM_PUBLIC_SIGN_UP_ENABLED` está encendido. Viene apagado.
- **Pantalla:** Registro (`/iam/sign-up`), construida sin perfil. Solo se abre si `GET /authentication/sign-up/status` responde que está encendido.
- **Lo construido:** correo y contraseña (8 a 72). Crea una cuenta sin roles, a nombre de `SYSTEM`, que no tiene que cambiar la contraseña. No abre sesión.
- **Lo que exige el diccionario y falta:** elegir empresa y sede sin sesión, completar los datos obligatorios del colaborador, crear el perfil y la cuenta en la misma operación, rechazar un documento ya registrado en esa empresa (§5.2, D-32, D-38). La ruta `sign-up/companies` que nombra convenciones §9.2 no existe; las listas de sedes y documentos están pendientes (convenciones §14.16). Mientras falte, es el único camino que crea una cuenta sin colaborador, y por eso sigue apagado.
- **Errores:** registro apagado (`403`), correo ya registrado (`409`).

### F5 · Dar acceso a un colaborador (D-38, D-58)

- **Quién:** un superadministrador o el administrador de accesos, en cualquier empresa. El administrador de empresa no da acceso (D-57).
- **Pantalla:** Configuración › Colaboradores, acción "Dar acceso al sistema" de la fila de un colaborador activo sin cuenta. El diálogo pide solo el correo.
- **Pasos** (`POST /access-grants` con `employeeId` y `email`). El correo se recorta y se pasa a minúsculas antes de validarlo, así `"  ana@x.ky "` se acepta como `ana@x.ky`. El sistema bloquea primero la fila del colaborador y después la de la cuenta, así dos accesos al mismo colaborador, o un acceso durante su salida, corren uno después del otro. Revisa, en este orden:
  1. Que el colaborador exista (`404 EMPLOYEE_NOT_FOUND`).
  2. Que quien opera maneje cuentas (`403 ACCESS_DENIED`).
  3. Que el colaborador esté activo (`422 EMPLOYEE_INACTIVE`) y no tenga ya una cuenta (`409 EMPLOYEE_ACCOUNT_CONFLICT`).
  4. Si el correo es el de quien opera: solo un superadministrador sin colaborador activo se vincula a sí mismo; si no, `403 OWN_ACCOUNT_FORBIDDEN`.
  5. Si no es su propio correo, que quien opera esté vinculado a un colaborador activo (`403 ACTOR_EMPLOYEE_REQUIRED`).
  6. **Si el correo no tiene cuenta**, crea una con una contraseña provisional que genera el sistema, a nombre de quien opera, y responde `201` con `created: true` y la contraseña, que se muestra una sola vez.
  7. **Si el correo ya tiene cuenta**: `SYSTEM` se rechaza (`422 BUSINESS_RULE_VIOLATION`); una cuenta ya vinculada a otro colaborador de esa empresa, `409 EMPLOYEE_ACCOUNT_CONFLICT`; una cuenta protegida, si quien opera no es superadministrador, `403 ACCOUNT_MANAGED_BY_SUPERADMIN`. Si nada de eso, la vincula y responde `200` con `created: false` y sin contraseña. **Nunca cambia sus roles ni su contraseña.** Si la cuenta estaba cerrada por una salida, cierra en la misma operación su `baja`; un bloqueo de otro motivo no se toca.
- **Resultado:** el colaborador tiene cuenta y la cuenta no ganó ningún rol. El diálogo ofrece "Asignar roles" en la empresa del colaborador, salvo cuando quien opera acaba de vincular su propia cuenta: sobre ella no se otorga nada.

### F6 · Otorgar y retirar roles (§5.5 a §5.8)

- **Quién:** un superadministrador, cualquier rol, sobre cualquier cuenta salvo la suya y `SYSTEM`; el administrador de accesos, cualquier rol salvo los reservados, y nunca sobre su propia cuenta ni sobre una protegida (D-55). Nadie más.
- **Pantalla:** el diálogo Roles, desde Seguridad › Usuarios o desde Configuración › Colaboradores. Lista las asignaciones vigentes y pasadas de la cuenta, la más nueva primero (`GET /users/{id}/role-assignments`), y otorga en una empresa. Al administrador de accesos no le ofrece los roles reservados; no ofrece un rol incompatible con uno que la cuenta ya tiene; nunca ofrece marca para `ADMIN_EMPRESA`. Mientras la cuenta tiene su contraseña provisional, no ofrece `SUPERADMIN`, `ADMIN_ACCESOS` ni `ADMIN_EMPRESA`, y una nota del formulario dice que se podrán otorgar cuando la persona inicie sesión y elija la suya; el administrador de accesos, a quien nunca se le ofrecen, no ve la nota. El diálogo lo sabe por `mustChangePassword`, que le pasan Seguridad › Usuarios, Colaboradores y dar acceso, este último solo para la cuenta que acaba de crear. De una cuenta existente recién vinculada no lo sabe: si el otorgamiento responde `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`, lo dice y vuelve a leer la cuenta. Nunca ofrece otorgar ni retirar en la propia cuenta. La caducidad se muestra en la zona de la sede para un rol de sede y, si no, en la de la empresa.
- **Otorgar** (`POST /users/{id}/role-assignments` con `roleId`, `companyId`, `branchId`, `commercialBrandId` y `expiresAt`, los cuatro últimos opcionales). El sistema bloquea la fila de la cuenta y revisa:
  - Que la cuenta y el rol existan (`404 USER_NOT_FOUND`, `404 ROLE_NOT_FOUND`). Un código de rol no sirve como identificador.
  - Que la cuenta no sea `SYSTEM` (`422 BUSINESS_RULE_VIOLATION`).
  - Que el alcance encaje en el nivel del rol (`422 ROLE_ASSIGNMENT_SCOPE_MISMATCH`) y que `ADMIN_EMPRESA` no traiga marca (`422 ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`).
  - Que la empresa exista y que la sede y la marca sean de ella (`404 COMPANY_NOT_FOUND`, `BRANCH_NOT_FOUND`, `COMMERCIALBRAND_NOT_FOUND`).
  - Que quien opera pueda otorgar ese rol (`403 ACCESS_DENIED`, también para un rol reservado) y actuar sobre esa cuenta: no la propia (`403 OWN_ACCOUNT_FORBIDDEN`), estando vinculado a un colaborador activo (`403 ACTOR_EMPLOYEE_REQUIRED`) y no una protegida si no es superadministrador (`403 ACCOUNT_MANAGED_BY_SUPERADMIN`).
  - Que la cuenta tenga un colaborador activo en la empresa del rol, o en alguna para un rol de sistema (`422 ROLE_ASSIGNMENT_EMPLOYEE_REQUIRED`).
  - Que la caducidad sea posterior al momento (`422 BUSINESS_RULE_VIOLATION`) y que la cuenta no tenga ya ese rol vigente sobre ese alcance (`409 ROLE_ASSIGNMENT_CONFLICT`).
  - Que no quede con `ADMIN_ACCESOS` y `ADMIN_EMPRESA` a la vez (`422 ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`), y que un rol reservado vaya a una cuenta que ya cambió su contraseña provisional (`422 ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`).
- **Retirar** (`POST /role-assignments/{id}/revocation`, sin cuerpo). El motivo es siempre `retiro`. Retirar exige poder otorgar ese rol y actuar sobre esa cuenta. Errores: `404 ROLE_ASSIGNMENT_NOT_FOUND`, los `403` de otorgar, `422 ROLE_ASSIGNMENT_INACTIVE` si ya estaba retirada o vencida, `422 LAST_ACTIVE_SUPERADMIN`.
- **Resultado:** la asignación vale desde la petición siguiente, sin que la persona vuelva a entrar (§5.12).

### F7 · Bloquear, levantar y cancelar (§5.4)

- **Quién:** un superadministrador o el administrador de accesos, sobre una cuenta que pueden manejar (F6). El administrador de empresa no bloquea: la `baja` es consecuencia de la salida que registra (F9).
- **Pantalla:** el diálogo Bloqueos, desde Seguridad › Usuarios o desde Configuración › Colaboradores. Lista todos los períodos de la cuenta, el más nuevo primero, con su estado: puede haber dos en curso, uno por línea. Ofrece "Cancelar" para un `SCHEDULED` y "Levantar" para un `IN_FORCE`, salvo en una `baja`, de la que dice que la cierra el regreso. Las confirmaciones nombran el motivo del bloqueo y, si otro período sigue en curso, avisan que la cuenta sigue bloqueada por él. Las fechas de una cuenta y de un bloqueo se muestran en `America/Lima`, porque no son de una empresa (§2.6).
- **Bloquear** (`POST /users/{id}/suspensions` con `reason`, `startsAt`, `endsAt` y `notes`, los tres últimos opcionales). El motivo es `vacaciones`, `licencia` o `seguridad`; `baja` responde `400 VALIDATION_ERROR`, como un motivo desconocido. Sin `startsAt` empieza ahora: "desde hoy" en la pantalla no lo manda, y un día futuro manda el inicio de ese día en Lima. El sistema bloquea la fila de la cuenta y revisa: que exista (`404`), que no sea `SYSTEM` (`422`), que quien opera pueda manejarla (los `403` de F6), que no empiece antes de la petición (`422 ACCESS_SUSPENSION_START_IN_PAST`), que el fin sea posterior al inicio (`422`), que no se cruce con otro de su línea (`409 ACCESS_SUSPENSION_CONFLICT`, también si la base lo detecta en una carrera) y que no deje al sistema sin superadministrador (`422 LAST_ACTIVE_SUPERADMIN`). Si empieza ahora, cierra todas las sesiones de la cuenta.
- **Levantar o cancelar** (`POST /suspensions/{id}/closure`, sin cuerpo). El sistema bloquea la fila de la cuenta antes de leer el período. Uno en curso termina ahora; uno programado queda cancelado. Errores: `404 ACCESS_SUSPENSION_NOT_FOUND`, los `403` de F6, `422 ACCESS_SUSPENSION_BAJA_RESERVED` para una `baja`, `422 ACCESS_SUSPENSION_CLOSED` si ya terminó o ya se cerró.
- **Resultado:** las vacaciones y las licencias no tocan los roles: la persona vuelve con los mismos accesos (§5.9).

### F8 · Restablecer la contraseña (D-63)

- **Quién:** un superadministrador o el administrador de accesos, sobre una cuenta que pueden manejar (F6). Nadie restablece así la suya: para eso está el cambio de contraseña (F3).
- **Pantalla:** acción "Restablecer contraseña", en Seguridad › Usuarios y en Configuración › Colaboradores. Pregunta antes.
- **Pasos** (`POST /users/{id}/password-reset`). El sistema bloquea la fila de la cuenta, revisa que exista (`404`), que no sea `SYSTEM` (`422`) y que quien opera pueda manejarla (los `403` de F6), genera una contraseña provisional, obliga a cambiarla y cierra todas las sesiones de la cuenta.
- **Resultado:** la respuesta trae la contraseña una sola vez, y el diálogo la muestra con un botón para copiarla. No se guarda legible en ningún lado. Lo ideal sería un enlace de un solo uso al correo de la persona; necesita un servicio de correo y queda pendiente (sección 6).

### F9 · Salida y regreso de un colaborador: lo que le pasa a la cuenta (§5.3, §5.4, §5.9)

Registrar las fechas es de `organization` ([organization.md](organization.md), F7). Las rutas `POST /employees/{id}/termination` y `POST /employees/{id}/reactivation` las sirve `iam`, que valida quién lo hace, le pide a `organization` que registre el hecho y cambia la cuenta en la misma transacción.

- **Quién:** un superadministrador, o el administrador de la empresa del colaborador vinculado a un colaborador activo. El administrador de accesos no registra salidas ni regresos (D-57). Nadie registra su propia salida ni su propio regreso: `403 OWN_ACCOUNT_FORBIDDEN` (D-70).
- **Salida.** El sistema bloquea la fila del colaborador y después la de su cuenta, si tiene. Si la cuenta no tiene un colaborador activo en otra empresa —un dato que se lee con alcance `SYSTEM`, porque desde una empresa todas las cuentas parecerían dejar la última—, la salida va a cerrarla, y antes revisa que no deje al sistema sin superadministrador (`422 LAST_ACTIVE_SUPERADMIN`). `organization` registra la salida con sus reglas de fechas. Después, en la misma transacción:
  1. Retira los roles vigentes de la cuenta en esa empresa con motivo `baja_colaborador`. Los de otras empresas y los de sistema quedan.
  2. Si era su último colaborador activo, cierra la cuenta desde ese momento: termina las vacaciones o la licencia en curso, cancela los bloqueos programados de la línea laboral, abre una `baja` sin fin salvo que ya haya una en curso, retira los demás roles con motivo `baja_cuenta` y cierra todas sus sesiones. Un bloqueo por seguridad, en curso o programado, no se toca (D-62). Los bloqueos que termina o cancela quedan cerrados a nombre de quien registró la salida.
- **La salida siempre quita el acceso**, también el de una cuenta protegida: el único freno es el último superadministrador (D-60). Cuando un administrador de empresa cierra una cuenta que tenía un rol de sistema, esos retiros se escriben con la elevación a `SYSTEM`, porque la base solo deja cambiar a ese alcance las filas sin empresa (V29).
- **Regreso.** `organization` registra la nueva fecha de ingreso con sus reglas, entre ellas que no sea posterior a hoy. Si la cuenta estaba cerrada por la salida, el sistema cierra en ese momento su `baja`, a nombre de quien registró el regreso. Los roles no vuelven solos: se otorgan de nuevo (F6). Un bloqueo por seguridad en curso sigue bloqueando la cuenta hasta que alguien lo levante.
- **Errores visibles:** los de `organization` para las fechas (su F7); `404 EMPLOYEE_NOT_FOUND` si el colaborador es de una empresa fuera del alcance de quien opera, que la base ni siquiera le muestra; `403 ACCESS_DENIED` si la ve pero no la administra; `403 OWN_ACCOUNT_FORBIDDEN`; `403 ACTOR_EMPLOYEE_REQUIRED` si es un administrador de empresa sin colaborador activo; `422 LAST_ACTIVE_SUPERADMIN`; y `409 ACCESS_SUSPENSION_CONFLICT` si la base detecta un cruce en una carrera.

### F10 · Caducidad automática (§5.8, D-64)

- **Quién:** nadie. Un proceso programado, el primero del proyecto (`RoleAssignmentExpiryTask`), corre con alcance `SYSTEM` cada `IAM_ROLE_EXPIRY_CHECK_INTERVAL` después de terminar la vuelta anterior.
- **Pasos:** busca las cuentas con asignaciones vencidas, bloquea la fila de cada una y cierra cada asignación vencida con su caducidad como fecha de retiro y motivo `caducidad`.
- **Resultado:** el registro dice la verdad. El acceso ya se había cortado al vencer, porque todo cálculo de lo vigente mira la caducidad: los roles de la sesión, el alcance, `/me`, las listas, las guardias y la revisión del arranque.

---

## 4. Reglas de negocio

Capas y marcas según el [README de esta carpeta](README.md). En la columna **Dónde se aplica**, los nombres de la base son restricciones de las migraciones; los demás, clases del backend o archivos del frontend. Las pruebas del backend corren conectadas con la cuenta de la aplicación, `erp_mm_app`, como en producción.

| ID | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| IAM-R01 | La contraseña tiene entre 8 y 72 caracteres, sin reglas de composición | §5.4 | Servicio (validación de los recursos de sign-up y change-password); Frontend (los dos formularios) | Sin prueba | ⚠️ |
| IAM-R02 | La nueva contraseña es distinta de la actual | §5.4 | Servicio | Sin prueba | ⚠️ |
| IAM-R03 | Un correo inexistente, una contraseña equivocada y `SYSTEM` reciben la misma respuesta | §5.4, §5.9 | Servicio (`UserCommandServiceImpl`) | `UserCommandServiceImplTest.signInDoesNotRevealWhetherTheAccountExists`, `theSystemAccountCannotSignInAndGetsTheAnswerOfWrongCredentials` | ✅ |
| IAM-R04 | El correo no se repite y se guarda sin espacios y en minúsculas; dar acceso, el inicio de sesión y el registro lo recortan antes de validarlo | §5.4 | Base (`UNIQUE`, `users_email_normalized`); Dominio (`User.normalizeEmail`); Servicio (los recursos de entrada) | `UserTest.newAccountHasANormalizedEmailAndMustChangeItsTemporaryPassword`, `UserCommandServiceImplTest.signUpCannotReuseARegisteredEmail`, `AccessGrantsIntegrationTests.anEmailWithSurroundingSpacesIsAccepted`, `AuthenticatedUserIntegrationTests.theSignInIgnoresTheSpacesAroundTheEmail` | ✅ |
| IAM-R05 | Con contraseña provisional la cuenta solo consulta quién es y cambia su contraseña; sus roles no valen | §5.4, §5.12 | Servicio (cadena de seguridad, `PASSWORD_CHANGE_REQUIRED`); Frontend (`iamGuard`) | Frontend: `iam.guard.spec.ts`. Backend: sin prueba | ⚠️ |
| IAM-R06 | La sesión se cierra a los 30 minutos sin uso y a las 8 horas desde que se abrió | §5.12, D-33 | Servicio (configuración y `SessionUserRefreshFilter`) | Sin prueba | ⚠️ |
| IAM-R07 | Roles, bloqueos y contraseña provisional se leen en cada petición: un rol retirado, una clave restablecida o un bloqueo que empieza valen desde la petición siguiente | §5.12 | Servicio (`SessionUserRefreshFilter`) | `SuspendedAccountIntegrationTests.theSessionOfASuspendedAccountStopsWorking`, `aSuspensionScheduledAheadTakesEffectByItself`; `SessionClosingIntegrationTests.aSuspensionScheduledAheadLeavesTheSessionOpenUntilItStarts` | ✅ |
| IAM-R08 | Cambiar la contraseña cierra las demás sesiones | §5.12, D-33 | Servicio | `UserCommandServiceImplTest.changingThePasswordClosesTheOtherSessions` | ✅ |
| IAM-R09 | Cerrar sesión borra la sesión en el servidor | §5.12 | Servicio (cierre de sesión de Spring Security) | Sin prueba | ⚠️ |
| IAM-R10 | La cookie `MM_SESSION` es `HttpOnly`, `SameSite=Strict` y `Secure` fuera de desarrollo; al iniciar sesión cambia el identificador | §5.12 | Servicio (configuración, `SessionServiceImpl`) | `SessionCookieTests.theRunningServerWritesTheSessionCookieAsConfigured` (el nombre); los atributos, sin prueba | ⚠️ |
| IAM-R11 | Toda petición que cambia datos lleva el token CSRF, también iniciar sesión y registrarse; API y frontend en el mismo origen | §5.12, convenciones §9.1 | Servicio (`csrf.spa()`); Frontend (encabezado `X-XSRF-TOKEN`, proxy) | Ninguna prueba verifica el rechazo sin token | ⚠️ |
| IAM-R12 | `SYSTEM` no inicia sesión, no se vincula a un colaborador, no recibe roles, no se bloquea y no se le restablece la contraseña | §5.9 | Base (V1 le pone un valor que no es un hash); Servicio (cada operación lo rechaza) | `UserCommandServiceImplTest.theSystemAccountCannotSignIn…`, `AccessGrantsIntegrationTests.theSystemAccountIsNeverLinkedToAnEmployee`, `RoleAssignmentsIntegrationTests.theSystemAccountReceivesNoRoles`, `UserPasswordResetIntegrationTests.theSystemAccountHasNoPasswordToReset`; el bloqueo, sin prueba propia | ⚠️ |
| IAM-R13 | El primer superadministrador se crea al arrancar solo si no hay uno vigente; sin variables arranca sin él; con un correo ya registrado no arranca | §5.2, D-34 | Servicio (`ApplicationReadyEventHandler`, `UserCommandServiceImpl`), con alcance `SYSTEM` | `UserCommandServiceImplTest.theFirstSuperAdministratorIsSeededOnlyWhileThereIsNone`, `RoleAssignmentExpiryTaskTests.anExpiredSuperadminIsNoActiveSuperadminForTheStartUpSeed`; los otros dos casos, sin prueba | ⚠️ |
| IAM-R14 | El primer superadministrador no da acceso ni otorga roles hasta vincularse a su perfil | §5.2, §5.9 | Servicio (`AccountAdministrationGuard`, `UserCommandServiceImpl`) | `RoleAssignmentsIntegrationTests.theFirstSuperadminGrantsNothingUntilItLinksItsOwnAccount`, `AccessGrantsIntegrationTests.theFirstSuperadminLinksItsOwnAccountBeforeGivingAccess` | ✅ |
| IAM-R15 | El registro público viene apagado; apagado, la API responde `403` | §5.2, D-32 | Servicio; Frontend (`signUpEnabledGuard`) | `UserCommandServiceImplTest.signUpIsRejectedWhileDisabled`; `iam.guard.spec.ts` | ✅ |
| IAM-R16 | El registro público crea perfil y cuenta juntos, sin roles, a nombre de `SYSTEM`, sin cambio forzado; rechaza correo o documento repetidos | §5.2, D-32 | Servicio, solo la parte de la cuenta | `UserCommandServiceImplTest.signUpCreatesAnAccountWithoutRolesAuthoredBySystem`, `signUpCannotReuseARegisteredEmail` | ⚠️ Falta el perfil |
| IAM-R17 | Toda cuenta pertenece a un colaborador y nace vinculada a él | §5.3, D-38 | Servicio: dar acceso crea y vincula en la misma transacción, y deshace la cuenta si el vínculo falla. `POST /users` ya no existe | `AccessGrantsIntegrationTests.aSuperadminCreatesTheAccountOfAnEmployeeWithATemporaryPassword`, `UserPasswordResetIntegrationTests.accountsAreNoLongerCreatedWithoutAnEmployee` | ⚠️ El registro público todavía crea cuentas sueltas (IAM-R16) |
| IAM-R18 | Un rol exige que la cuenta esté vinculada a un colaborador activo: de esa empresa para un rol de empresa o de sede, de alguna para uno de sistema | §5.3, §5.7, D-38 | Servicio (`RoleAssignmentCommandServiceImpl`) | `RoleAssignmentsIntegrationTests.anAccountWithoutAnActiveEmployeeInTheCompanyReceivesNoRoleThere` | ✅ |
| IAM-R19 | ~~Nadie otorga un rol que no tiene; hoy, tener ese mismo rol vigente~~ | §5.5, §5.13 | Retirada el 16/10/2026: la reemplaza IAM-R31 | — | — |
| IAM-R20 | ~~`POST /users` otorga solo roles de alcance sistema~~ | §5.13 | Retirada el 16/10/2026: `POST /users` ya no existe (D-38) | — | — |
| IAM-R21 | Ninguna operación deja al sistema sin superadministrador: retirar el rol, bloquear ahora o con un bloqueo programado, ni la salida que cierra la cuenta. Procede solo si otro queda de forma duradera —rol sin caducidad y sin bloqueos en curso ni programados—, y esas operaciones van de a una | §5.9, D-65 | Servicio (`LastSuperadminGuard`: bloqueo consultivo de la transacción y conteo con alcance `SYSTEM`) | `LastSuperadminGuardTests`, `SuperadministratorsLockTests`, `RoleAssignmentsIntegrationTests.aSuperadminIsNotRevokedWhileTheOnlyOtherOneExpires`, `…HasABlockScheduled`, `SuspensionsIntegrationTests.noSuspensionLeavesTheSystemWithoutAnActiveSuperadmin`, `aBlockScheduledOnTheOnlyOtherSuperadminStopsTheNextOne`, `EmployeeTenureIntegrationTests.theLastSuperadminCannotLeave`, `aSuperadminCannotLeaveWhileTheOnlyOtherOneHasALeaveScheduled` | ✅ |
| IAM-R22 | Un rol se retira, no se borra, y siempre con motivo | §5.7, §5.8 | Base (`user_roles_revocation_reason_ck`; sin permiso de borrar, V19); Dominio | `RoleAssignmentScopeMigrationTests.aRevocationAlwaysSaysWhy`, `UserTest.revokingRecordsTheReasonAndOnlyAnAssignmentInForceIsRevoked` | ✅ |
| IAM-R23 | El motivo de retiro es `retiro`, `caducidad`, `baja_colaborador` o `baja_cuenta` | D-41 | Base (`CHECK`, V27); Dominio (`RevocationReason`) | `RoleAssignmentScopeMigrationTests.aRevocationReasonIsOneOfTheFour` | ✅ |
| IAM-R24 | El mismo rol no se da dos veces sobre el mismo alcance mientras siga vigente; sobre dos empresas, sí. El alcance guardado coincide con el del rol | §5.7 | Base (índice único con `NULLS NOT DISTINCT` sobre todo el alcance, V27; `user_roles_role_scope_fk`); Servicio (`409 ROLE_ASSIGNMENT_CONFLICT`) | `RoleAssignmentScopeMigrationTests.theSameRoleIsHeldOnTwoCompanies`, `theSameRoleIsNeverHeldTwiceOnTheSameScope`; `RoleAssignmentsIntegrationTests` (los mismos dos casos) | ✅ |
| IAM-R25 | Un rol vale sobre una empresa, sede o marca, coherentes con su nivel y entre sí; retirarlo en una empresa no toca las otras | §2.5, §5.7, D-38 | Base (`user_roles_scope_coherence_ck` y llaves compuestas hacia sedes y marcas de la misma empresa, V26 y V27); Dominio (`RoleScope`) | `RoleAssignmentScopeMigrationTests` (alcance, sede y marca), `UserTest.leavingACompanyRevokesOnlyTheRolesOnThatCompany`, `EmployeeTenureIntegrationTests.theSameRoleOnTwoCompaniesSurvivesLeavingOneOfThem` | ✅ |
| IAM-R26 | Un acceso deja de valer al caducar; un proceso lo cierra con su caducidad como fecha y motivo `caducidad`; otorgar de nuevo cierra solo la vencida con la misma llave | §5.8, D-64 | Dominio (`RoleAssignment.isActiveAt`, `User.grant`); Servicio (`RoleAssignmentExpiryTask`, alcance `SYSTEM`) | `RoleAssignmentExpiryTaskTests.theProcessClosesEveryExpiredAssignmentAtItsExpiryWithCaducidad`, `UserTest.grantingClosesOnlyTheExpiredAssignmentOfTheSameRoleOnTheSameScope`, `RoleAssignmentsIntegrationTests.aGrantOnOneScopeLeavesTheExpiredAssignmentsOfOtherScopesAlone` | ✅ |
| IAM-R27 | Las cuentas se bloquean por períodos; activa es la que no tiene ninguno en curso, de cualquier línea; cuando un bloqueo empieza se cierran sus sesiones; una cuenta bloqueada recibe `403 ACCOUNT_SUSPENDED` solo después de que su contraseña coincidió | §5.4, §5.12, D-62 | Base (V28); Dominio (`AccessSuspension`); Servicio (inicio de sesión, `SessionUserRefreshFilter`) | `AccessSuspensionTests`, `SuspendedAccountIntegrationTests`, `SessionClosingIntegrationTests.theSessionWorksUntilTheAccountIsSuspendedAndThenItIsGone`, `UserCommandServiceImplTest.aWrongPasswordOfASuspendedAccountRevealsNothing` | ✅ |
| IAM-R28 | El autor lo pone el sistema con la cuenta de la sesión; `SYSTEM` solo de forma explícita y solo sin sesión; nunca se supone. `granted_by` y `closed_by` también salen de la sesión | D-42; convenciones §14.26 | Servicio (`CurrentUserAuditorAware`; los servicios de comandos pasan la cuenta de quien opera) | `CurrentUserAuditorAwareTest`, `UserAuthorshipTests`, `AccessSuspensionAuditTests.aNewSuspensionGetsItsPublicIdItsMomentAndTheAccountOfTheSessionAsAuthor` | ✅ |
| IAM-R29 | Ninguna operación se conforma con la sesión: cada ruta exige un rol, y el servicio decide además sobre qué cuenta o empresa (convenciones §9.2) | §5.2, convenciones §9.2 | Servicio (`@PreAuthorize` en cada controlador; políticas en los servicios) | `RolesControllerTests` y los `403` de cada prueba de integración de `iam` | ✅ |
| IAM-R30 | Una cuenta nunca se borra; un rol del sistema no se edita ni se borra | §5.4, §5.5 | No hay operación que lo haga; Base (sin permiso de borrar, V19) | Sin prueba | ⚠️ |
| IAM-R31 | Otorgan roles el superadministrador, cualquiera, y el administrador de accesos, cualquiera salvo los reservados (`SUPERADMIN`, `ADMIN_ACCESOS`, `ADMIN_EMPRESA`); nadie más. Retirar exige poder otorgar | §5.5 regla 2 con su excepción, D-55, D-56, D-57 | Dominio (`GrantPolicy`); Servicio; Frontend (el diálogo no ofrece los reservados) | `GrantPolicyTests`, `RoleAssignmentsIntegrationTests.theAccessAdministratorDoesNotGrantTheRolesReservedToTheSuperadmin`, `theAccessAdministratorDoesNotRevokeTheCompanyAdministratorRole`, `aCompanyAdministratorGrantsNoRole`, `whoCannotGrantAnAssignmentCannotRevokeIt` | ✅ |
| IAM-R32 | Manejan cuentas el superadministrador y el administrador de accesos, solo mientras están vinculados a un colaborador activo (`403 ACTOR_EMPLOYEE_REQUIRED`); el administrador de accesos nunca actúa sobre una cuenta protegida (`403 ACCOUNT_MANAGED_BY_SUPERADMIN`) | D-55, D-57 | Dominio (`AccountAdministrationPolicy`); Servicio (`AccountAdministrationGuard`, en dar acceso, otorgar, retirar, bloquear, cerrar y restablecer); Frontend (`IamStore.managesAccount`, que lee `managedBySuperadmin`) | `AccountAdministrationPolicyTests`, `AccessGrantsIntegrationTests.theAccessAdministratorCannotTakeOverTheAccountOfASuperadmin`, `AccessAdministratorIntegrationTests.theAccessAdministratorDoesNotActOnTheAccountOfACompanyAdministrator`, `RoleAssignmentsIntegrationTests.theAccessAdministratorGrantsNothingToAProtectedAccount`, `anAccessAdministratorNotLinkedToAnEmployeeGrantsNothing`, `UserPasswordResetIntegrationTests.theAccessAdministratorCannotResetThePasswordOfAProtectedAccount` | ✅ |
| IAM-R33 | Nadie actúa sobre su propia cuenta, tampoco un superadministrador (`403 OWN_ACCOUNT_FORBIDDEN`) | D-70 | Dominio (`AccountAdministrationPolicy`); Servicio; Frontend (ninguna acción en la propia fila ni en el diálogo de roles de la propia cuenta) | `AccountAdministrationPolicyTests.nobodyActsOnTheirOwnAccountNotEvenASuperadmin`, `RoleAssignmentsIntegrationTests.nobodyGrantsRolesToTheirOwnAccount`, `nobodyRevokesTheirOwnRoles`, `UserPasswordResetIntegrationTests.nobodyResetsTheirOwnPasswordThroughTheAdministratorOperation`, `SuspensionsIntegrationTests.onlyWhoAdministersAnAccountSuspendsItAndNeverItsOwn` | ✅ |
| IAM-R34 | Nadie registra su propia salida ni su propio regreso (`403 OWN_ACCOUNT_FORBIDDEN`) | D-70 | Servicio (`EmployeeTenureCommandServiceImpl`); Frontend (ninguna salida ni regreso en el propio colaborador) | `EmployeeTenureIntegrationTests.nobodyRecordsTheirOwnExitOrReturn` | ✅ |
| IAM-R35 | La única cuenta que se vincula a sí misma es la del primer superadministrador, mientras no tiene un colaborador activo | §5.2 paso 6, D-70 | Servicio (`UserCommandServiceImpl`) | `AccessGrantsIntegrationTests.theFirstSuperadminLinksItsOwnAccountBeforeGivingAccess`, `theAccessAdministratorDoesNotLinkItsOwnAccount`, `aLinkedSuperadminDoesNotLinkItsOwnAccountToAnotherCompany` | ✅ |
| IAM-R36 | Dar acceso crea la cuenta o la vincula y nunca cambia los roles ni la contraseña de una cuenta existente; una cuenta protegida solo la vincula un superadministrador; vincular una cuenta cerrada por una salida cierra su `baja` | D-38, D-58 | Servicio (`UserCommandServiceImpl`); Base (`employees_company_user_uq`, V26) | `AccessGrantsIntegrationTests.linkingAnExistingAccountNeverChangesItsRolesOrItsPassword`, `onlyASuperadminLinksTheAccountOfAnotherAccessAdministrator`, `linkingAnAccountClosedByAnExitEndsThatExit`, `linkingAnAccountLeavesItsOtherBlocksAsTheyAre`, `anAccountIsLinkedToOneEmployeePerCompany` | ✅ |
| IAM-R37 | Un bloqueo empieza ahora o después; se cierra siempre ahora; cerrar uno programado lo cancela y conserva su período; uno terminado o cerrado no se cierra de nuevo | §5.4, D-61 | Dominio (`AccessSuspension`); Servicio; Base (el `CHECK` de V28 sobre el cierre) | `AccessSuspensionTests`, `SuspensionsIntegrationTests.aSuspensionDoesNotStartBeforeTheRequest`, `closingASuspensionScheduledAheadCancelsIt`, `aSuspensionThatAlreadyEndedIsNotClosed`, `aSuspensionIsAlwaysClosedNow`, `AccessSuspensionMigrationTests.aPeriodCancelledBeforeItStartsKeepsItsSchedule` | ✅ |
| IAM-R38 | Dos bloqueos de la misma línea no se cruzan, ni en el pasado; los cancelados no cuentan; entre líneas sí se cruzan | §5.4, D-62, D-67 | Base (`EXCLUDE USING gist` con la línea, V32); Servicio (`409`, también cuando la base lo detecta en una carrera) | `AccessSuspensionMigrationTests.twoPeriodsOfTheSameTrackNeverOverlap`, `aSecurityBlockMayOverlapAnEmploymentPeriod`, `aCancelledPeriodNoLongerCountsForOverlaps`, `SuspensionsIntegrationTests.vacationsMayOverlapASecurityBlockButNotALeave`, `AccountLockTests.anOverlapRefusedByTheDatabaseIsReportedAsAnOverlap` | ✅ |
| IAM-R39 | La `baja` es de la salida: no se abre a mano (`400`) ni se cierra a mano (`422 ACCESS_SUSPENSION_BAJA_RESERVED`) | D-69 | Servicio (el recurso acepta tres motivos; `AccessSuspensionCommandServiceImpl`); Frontend (el diálogo no ofrece la `baja` ni su cierre) | `SuspensionsIntegrationTests.aBajaIsNeverOpenedByHand`, `aBajaIsNeverClosedByHand` | ✅ |
| IAM-R40 | La salida retira los roles de esa empresa (`baja_colaborador`); si era el último colaborador activo, cierra la cuenta (`baja` salvo una en curso, `baja_cuenta`, sesiones), termina las vacaciones o la licencia en curso, cancela lo programado de la línea laboral y no toca la seguridad | §5.3, §5.9, D-60 | Servicio (`EmployeeTenureCommandServiceImpl`, con el dato de las otras empresas leído con alcance `SYSTEM`) | `EmployeeTenureIntegrationTests.leavingTheLastCompanyTakesEveryRoleAway`, `leavingOneOfTwoCompaniesKeepsTheAccountOpen`, `theExitEndsTheVacationsInForceAndOpensTheExitPeriodWhereTheyEnd`, `theExitCancelsTheScheduledEmploymentPeriodsAndLeavesTheSecurityBlocksAlone`, `CrossCompanyAccountAdministrationIntegrationTests.theExitFromItsCompanyDoesNotCloseAnAccountThatStillWorksForAnotherOne` | ✅ |
| IAM-R41 | La salida siempre quita el acceso, también el de una cuenta protegida; solo la detiene el último superadministrador | D-60 | Servicio (`EmployeeTenureCommandServiceImpl`; los retiros de roles de sistema, con la elevación a `SYSTEM`) | `CrossCompanyAccountAdministrationIntegrationTests.theExitOfTheLastEmployeeClosesAProtectedAccountToo`, `AccessAdministratorIntegrationTests.theCompanyAdministratorsExitOfAnAccessAdministratorClosesItsAccount`, `EmployeeTenureIntegrationTests.aCompanyAdministratorClosesTheAccountOfASuperadminThatLeaves` | ✅ |
| IAM-R42 | Registran salidas y regresos el superadministrador y el administrador de la empresa del colaborador, este solo mientras está vinculado a un colaborador activo (`403 ACTOR_EMPLOYEE_REQUIRED`); el administrador de accesos no | D-57 | Servicio (`EmployeeTenureCommandServiceImpl`); `@PreAuthorize`; Frontend (las acciones solo donde la sesión administra la empresa) | `EmployeeTenureIntegrationTests.aCompanyAdministratorRecordsTheExitOfAnEmployeeOfItsCompany`, `aCompanyAdministratorDoesNotRecordExitsInAnotherCompany`, `aCompanyAdministratorNotLinkedToAnEmployeeRecordsNoExit`, `AccessAdministratorIntegrationTests.theAccessAdministratorNeitherWritesEmployeesNorRecordsTheirExitsAndReturns` | ✅ |
| IAM-R43 | El regreso cierra solo la `baja`; los roles no vuelven; un bloqueo por seguridad sigue | §5.4, §5.9, D-60 | Servicio | `EmployeeTenureIntegrationTests.theReturnOfAnEmployeeDoesNotRestoreItsRoles`, `aSecurityBlockSurvivesTheExitAndTheReturn` | ✅ |
| IAM-R44 | `ADMIN_EMPRESA` nunca se limita a una marca | D-59 | Servicio (`422 ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`); V31 cerró las que había; Frontend (sin campo de marca) | `UserTest.theCompanyAdministratorIsNeverLimitedToABrand`, `RoleAssignmentsIntegrationTests.theCompanyAdministratorRoleIsNeverLimitedToABrand` | ✅ |
| IAM-R45 | Una cuenta nunca tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` vigentes a la vez (`422 ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`) | D-71 | Dominio (`User`); Servicio (la base no ve dos asignaciones a la vez); Frontend (no ofrece el rol incompatible) | `UserTest.theAccessAdministratorAndTheCompanyAdministratorAreNeverHeldTogether`, `RoleAssignmentsIntegrationTests.theAccessAdministratorAndTheCompanyAdministratorAreNeverGrantedToTheSameAccount` (los dos sentidos) | ✅ |
| IAM-R46 | Un rol reservado se otorga solo a una cuenta que ya cambió su contraseña provisional (`422 ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`) | D-72 | Dominio (`User`); Servicio; Frontend (el diálogo de roles no ofrece los reservados a una cuenta con contraseña provisional y dice por qué) | `UserTest.aPrivilegedRoleGoesOnlyToAnAccountWithItsOwnPassword`, `RoleAssignmentsIntegrationTests.aPrivilegedRoleGoesOnlyToAnAccountThatChangedItsTemporaryPassword` (los tres reservados rechazados, `CONTABILIDAD` aceptado y `201` después del cambio) | ✅ |
| IAM-R47 | El alcance de una cuenta sale de sus asignaciones vigentes: un rol de sistema ve todas las empresas; administra todas solo el superadministrador, y una, el administrador de esa empresa; sin sesión o bloqueada, nada | §5.7, §5.8, D-57 | Servicio (`CurrentUserAccessScope`, que implementa `CurrentAccessScope`) | `AccessScopeTests`, `AuthenticatedUserIntegrationTests.theAccessAdministratorReachesEveryCompanyAndAdministersNone`, `aCompanyAdministratorAdministersOnlyTheCompaniesOfThatRole` | ✅ |
| IAM-R48 | La frontera entre empresas la defiende la base: Row Level Security forzada; el alcance se fija con `set_config` al comenzar cada transacción, desde un solo punto; sin alcance no se ve nada; `SYSTEM` explícito para el arranque, los procesos y las comprobaciones acotadas | D-66; reglas 5.1 a 5.3 y 5.7 | Base (V29); Servicio (`AccessScopeJpaTransactionManager`, `DatabaseAccessScope`, `SystemAccessScope`) | `RowLevelSecurityMigrationTests`, `DatabaseAccessScopeTests`; toda la suite corre como `erp_mm_app` | ✅ |
| IAM-R49 | Las operaciones que leen y después cambian una cuenta bloquean primero su fila de `users`, y las que cambian un colaborador, la de `employees`; lo que leen es la fila de después del bloqueo, no una copia anterior | Modelo de acceso (r3 B11, r5 T6); [patrones de persistencia](../patrones-de-persistencia-millennium-mobility.md) | Servicio (`findBy…ForUpdate`) | `AccountLockTests.aSecondReadForUpdateOfTheSameAccountWaitsUntilTheFirstTransactionEnds`, `EmployeeLockTests.readingAnEmployeeForUpdateLocksItsRow`, `AccountLockTests.anAccountReadForUpdateIsItsRowsAsTheLockFindsThemNotTheCopyReadBefore`, `EmployeeLockTests.anEmployeeReadForUpdateIsItsRowAsTheLockFindsItNotTheCopyReadBefore` | ✅ |
| IAM-R50 | Restablecer una contraseña genera una provisional que se muestra una sola vez, obliga a cambiarla y cierra todas las sesiones de la cuenta | §5.4, §5.12, D-63 | Servicio | `UserPasswordResetIntegrationTests.aSuperadminResetsAPasswordThatMustBeChangedOnTheNextSignIn`, `SessionClosingIntegrationTests.resettingThePasswordClosesEverySessionOfTheAccount` | ✅ |
| IAM-R51 | La lista de cuentas dice en cada fila si solo un superadministrador la maneja (`managedBySuperadmin`), con la misma regla que responde `ACCOUNT_MANAGED_BY_SUPERADMIN` y mirando sus asignaciones vigentes en todas las empresas, también las que la fila no muestra o quien llama no ve. El frontend toma "protegida" solo de ese campo | D-55 | Dominio (`AccountAdministrationPolicy.isManagedBySuperadmin`); Servicio (`ProtectedAccountsReader`: para quien no ve todas las empresas, vuelve a leer las cuentas con la elevación a `SYSTEM` y solo saca los identificadores de las protegidas); Frontend (`IamStore.managesAccount`, `CompanyAccount.manageable`) | `AccountAdministrationPolicyTests.anAccountIsManagedBySuperadminWhileItHoldsAReservedRoleInForce`, `ProtectedAccountsReaderTests.forACallerThatSeesEveryCompanyTheAccountsAsReadAreJudged`, `forACompanyScopedCallerTheAccountsAreReadAgainWithTheSystemScope`, `UsersIntegrationTests.anAccountThatAdministersAnotherCompanyIsManagedBySuperadminInTheListOfEachCompany`, `theListOfEveryAccountSaysWhichOnesOnlyASuperadminManages`; frontend: `iam.store.spec.ts` | ✅ |

---

## 5. Relación con otros contextos

**Lo que ofrece.** Ninguna fachada: ningún contexto llama a `iam` (mapa §4), y `IamContextFacade` ya no existe. Lo que los demás necesitan de la cuenta lo reciben por puertos del núcleo `shared`, que `iam` implementa sin que nadie dependa de su código:

| Puerto | Lo implementa | Para qué |
|---|---|---|
| `AuditorAware` de Spring | `CurrentUserAuditorAware` | Llenar `created_by` y `updated_by` en todos los contextos (D-42) |
| `CurrentAccessScope` | `CurrentUserAccessScope` | Decir qué empresas ve y administra la cuenta de la petición: `AccessScope` con `global`, `administersEveryCompany`, `companyIds` y `administeredCompanyIds`. `organization` filtra con él cada lectura y cada escritura de colaboradores |
| `SystemAccessScope` | `DatabaseAccessScope`, de `shared` | Ejecutar con alcance `SYSTEM` un proceso automático o una comprobación que necesita ver más allá de las empresas de quien opera, y volver al alcance de quien llama |

Las elevaciones a `SYSTEM` son pocas y explícitas: el arranque, el proceso de caducidad, el conteo de superadministradores; en la salida que registra un administrador de empresa, si la cuenta tiene un colaborador activo en otra empresa y los retiros de los roles de sistema de la cuenta que cierra; y, en la lista de cuentas que pide quien no ve todas las empresas, cuáles son protegidas (`ProtectedAccountsReader`, IAM-R51). Cada una devuelve solo el dato que necesita, nunca un agregado: de la última salen solo los identificadores de las cuentas protegidas, y la lista sigue mostrando solo los roles que quien llama ve.

**Lo que usa de `organization`** (`ExternalOrganizationService`, sobre `OrganizationContextFacade`; `iam` declara `allowedDependencies = {"organization::acl"}`):

| Para qué | Miembros de la fachada |
|---|---|
| Leer y bloquear el colaborador al dar acceso, en la salida y en el regreso | `fetchEmployeeSummaryForUpdate` |
| Saber en qué empresas tiene colaboradores una cuenta | `fetchEmployeeSummariesByUserId`, `fetchActiveEmployeeCompanyIdsByUserId` |
| Vincular un colaborador a una cuenta | `linkUserToEmployee` |
| Registrar la salida y el regreso, con sus reglas de fechas | `terminateEmployee`, `reactivateEmployee` |
| Resolver el alcance de una asignación: la empresa, y la sede y la marca de esa empresa | `resolveScope` |
| Nombrar empresas, sedes y marcas en lote en las listas | `fetchCompanyReferencesByIds`, `fetchBranchReferencesByIds`, `fetchCommercialBrandReferencesByIds` |
| Las empresas y sus zonas para `/me` | `fetchActiveCompanyReferences`, `fetchCompanyTimeZonesByIds` |
| Las cuentas con colaborador en una empresa, para `/users?companyId` | `fetchLinkedEmployeePublicIdsByCompanyId` |

Las lecturas que `organization` le sirve a `iam` por la fachada no aplican el filtro de alcance de la API: `iam` decide quién puede actuar sobre lo que pide.

**Frontend: `IamFacade`.** Es lo único que otros contextos del frontend usan de `iam`.

| Miembro | Quién lo usa |
|---|---|
| `hasRole(code)`, `hasAnyRole` | `app-shell` (menú) y las pantallas de `organization` |
| `managesAccess` | Colaboradores: muestra las acciones de cuenta a quien maneja cuentas. Se deriva de los roles de la sesión |
| `administers(companyId)`, `readsAccountsOf(companyId)` | Colaboradores: crear, editar, salida y regreso donde la sesión administra la empresa; leer las cuentas de la empresa |
| `loadCompanyAccounts`, `companyAccounts` | Colaboradores: la columna de cuenta y roles, con si la cuenta tiene contraseña provisional (`mustChangePassword`) y si la sesión puede manejarla (`manageable`, que sale de `managedBySuperadmin`) |
| `grantAccess`, `manageRoles`, `resetPassword`, `manageSuspensions` | Colaboradores: abren los diálogos de `iam`, que se cargan al primer uso |
| `restoreSession()` | `app.config`, una vez al arrancar |

`currentCompanyId` ya no existe: una cuenta puede tener varias empresas, y la sesión trae la lista (`companies`). La zona de las fechas de un colaborador la trae el propio colaborador (`EmployeeResource.timeZone`), y los diálogos de salida y regreso la usan.

---

## 6. Estado y pendientes para cerrar

**Lo construido (paso 3).** En `feature/access-model` están dar acceso, las asignaciones con su empresa, sede y marca, su caducidad y su cierre automático, los bloqueos en dos líneas de tiempo, el restablecimiento de contraseña, la salida y el regreso con sus consecuencias en la cuenta, el administrador de accesos con la separación de funciones completa y la frontera entre empresas en la base (V26 a V32), con sus pantallas. `organization` ya no depende de `iam` (mapa §4).

**Verificación** (16/10/2026, sobre el backend `6a7d9b1` y el frontend `1a341ab`):

- **Backend:** 926 pruebas, sin fallos, errores ni omitidas. Corren conectadas como `erp_mm_app`, con Row Level Security en vigor, sobre una `erp_mm_db_verify` recreada y migrada por `erp_mm_owner` de V1 a V32.
- **Frontend:** 877 pruebas unitarias en 88 archivos; el build inicial pesa 489,11 kB, bajo el presupuesto de 500 kB; `lint:styles` sin observaciones; 425 e2e pasan en los tres anchos y 22 se omiten a propósito, en los anchos donde repetirlas no agrega nada; ninguna falla.
- **Primera sonda contra el backend real** (ronda 4): 210 de 211 comprobaciones HTTP coincidieron con lo esperado; la restante fue el orden de la propia sonda, que volvió a leer el `/me` de una cuenta después de que su salida le había retirado los roles, y el backend respondió bien. Las 40 comprobaciones de la frontera entre empresas pasaron, y el recorrido del navegador con los tres roles no dio errores de página.
- **Sonda final** (rondas 5 y 6): 129 de 129 comprobaciones de la política coincidieron con el backend real: las reglas de la ronda 5, los códigos nuevos (`OWN_ACCOUNT_FORBIDDEN`, `ACTOR_EMPLOYEE_REQUIRED`, `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`, `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`, `EMPLOYEE_REHIRE_DATE_IN_FUTURE`), `managedBySuperadmin` —también en una administradora de otra empresa listada desde la primera, cuyo rol allá no se ve—, `timeZone` con una sede en Tokio y el nombre comercial de las empresas. El navegador, sin simulaciones, ofreció cada acción según el rol y no dio más error que el 401 esperado al cargar sin sesión. Quedaron cuatro diferencias menores, anotadas en lo abierto en la [revisión del 16/10/2026](../retroalimentacion/2026-10-16-iam.md).

### a) Decisiones

| Qué | Quién la toma |
|---|---|
| Validar las decisiones D-55 a D-72 en el diccionario v1.18 | Responsable del diccionario |
| La matriz de permisos por rol, que llega antes del paso 5 y acota lo que leen los roles globales; con ella, quién crea los roles de una empresa | Responsable del diccionario, con el negocio (orden de implementación, ajuste 3) |
| Cuántas personas tienen `ADMIN_ACCESOS`: el código no lo limita | Responsable del diccionario |
| Si `user_roles` registra quién retiró un rol (`revoked_by`) | Responsable del diccionario |
| Restablecer la contraseña con un enlace de un solo uso al correo, en vez de mostrar la provisional al administrador | Responsable del diccionario, cuando haya un servicio de correo |

### b) Código

| # | Prioridad | Qué falta | Cuándo |
|---|---|---|---|
| 1 | Media | El registro público con perfil (IAM-R16, R17) | Cuando se decida convenciones §14.16; mientras tanto sigue apagado |
| 2 | Media | La sesión guarda el hash de la contraseña: `UserDetailsImpl` lo lleva y se serializa en `spring_session_attributes` | Pendiente transversal del orden |
| 3 | Media | Un error inesperado devuelve el mensaje de la excepción en `details` (`GlobalExceptionHandler`) | Convenciones §13 fila 14 |
| 4 | Media | Pruebas de las reglas marcadas "sin prueba" en §4 | Con cada regla que se toque |
| 5 | Baja | Un correo de más de 255 caracteres responde `500`: los recursos no limitan su largo | Pendiente transversal del orden |
| 6 | Baja | `RoleResource.system` debería llamarse `isSystem` | Convenciones §13 fila 6 |

### c) Datos

- La matriz de permisos por rol (§14; procesos, pregunta 5). Sin ella no hay `permissions` ni la comparación de §5.5.
- El primer superadministrador de cada entorno tiene que registrarse como colaborador y vincularse a su perfil (P1, paso 6) antes de dar acceso. Ya hay con qué: dar acceso con su propio correo (F1).

### d) Documentación

- La propuesta del diccionario v1.18 registra D-55 a D-72 y actualiza §3 y §5.13; la valida el responsable del diccionario.

---

## 7. Preguntas abiertas

1. **¿Qué permisos tiene cada rol del sistema, y quién crea los roles de una empresa?** [Procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), apartado 5, pregunta 5; diccionario §14; [orden de implementación](../orden-de-implementacion-millennium-mobility.md), ajuste 3.
2. **¿Cómo obtiene el registro público, sin sesión, las sedes y los tipos de documento?** [Convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), §14.16.
3. **¿Cómo se muestran las referencias a cuentas fuera de `iam`?** [Mapa de contextos](../mapa-de-contextos-millennium-mobility.md), §11, decisión 6.
4. **¿Cuántas personas pueden tener `ADMIN_ACCESOS`, y se registra quién retiró un rol?** Propuesta del diccionario v1.18, preguntas abiertas.
5. **¿El restablecimiento pasa a un enlace de un solo uso al correo?** Propuesta del diccionario v1.18, preguntas abiertas; [orden de implementación](../orden-de-implementacion-millennium-mobility.md), pendientes transversales.
