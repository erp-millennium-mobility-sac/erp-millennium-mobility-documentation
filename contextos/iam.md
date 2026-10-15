# Contexto `iam` — ERP Millennium Mobility

**Actualizado:** 14/10/2026 · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.15 (§2.5, §2.6, §3, §5.1 a §5.13, D-32, D-33, D-34, D-38 a D-43), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md) §4, §5.1, §6.2 y §11, [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md) §2.2, §9, §13 y §14, [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) P1 y P2, [orden de implementación](../orden-de-implementacion-millennium-mobility.md) y el código del backend y del frontend en la rama `feature/location`, revisado el 14/10/2026. Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** Lo que cita una sección (§n) o una decisión (D-n) del diccionario está decidido. Lo marcado **(propuesta)** no lo decidió nadie todavía. "Hoy" describe el código revisado, no el diseño.

---

## 1. Qué es y qué no es

`iam` responde a tres preguntas: **con qué cuenta entra una persona**, **qué roles tiene esa cuenta** y **si su sesión sigue abierta**. Es la puerta del sistema: si una regla de este contexto falla, falla la protección de todos los demás.

**Tablas que posee hoy**

| Tabla | Qué guarda | Estado |
|---|---|---|
| `users` | La cuenta: correo, contraseña cifrada, si debe cambiarla, último acceso (§5.4) | Construida (V1) |
| `roles` | Los seis roles del sistema con su alcance (§5.5) | Construida (V1), sin `company_id` |
| `user_roles` | Qué rol tiene cada cuenta, desde cuándo y si se retiró (§5.7) | Construida en versión reducida (V1): sin `company_id`, `branch_id` ni `commercial_brand_id` |
| `spring_session`, `spring_session_attributes` | Las sesiones abiertas, en el servidor (§5.12, D-33) | Construidas (V3). Tablas técnicas de la librería |

**Tablas diseñadas que todavía no existen**

| Tabla o columna | Para qué | Cuándo |
|---|---|---|
| `user_access_suspensions` | Bloqueos de cuenta por vacaciones, licencia, baja o seguridad, como períodos (§5.4) | Paso 3 |
| `permissions`, `role_permissions` | Las acciones concretas y qué rol incluye cada una (§5.6) | Paso 3, con la matriz de permisos |
| `user_roles.company_id`, `branch_id`, `commercial_brand_id` | Sobre qué empresa, sede y marca vale cada rol (§5.7) | Paso 3 |
| `roles.company_id` | Roles propios de una empresa (§5.5, D-7) | Paso 2 según el orden de implementación; no existe todavía |

Que `user_roles` sea de `iam` es una propuesta del mapa (§3 y decisión pendiente 4). El diccionario asigna el bloque 3b a `organization` e `iam` sin separar tablas.

**Lo que parece suyo y no lo es**

- **Los colaboradores** (`employees`) son de `organization` **(propuesta, mapa §6.2)**, incluida la columna `employees.user_id` que une al colaborador con su cuenta. Toda cuenta pertenece a un colaborador (D-38), pero quien guarda la referencia es el colaborador. Ver [organization.md](organization.md).
- **El perfil** (nombre, documento, teléfono, foto, puesto) va en `employees`, no en `users` (D-4, D-10). La cuenta guarda solo lo necesario para entrar.
- **Los clientes** no tendrán cuenta aquí: la aplicación de clientes tendrá su propia tabla de identidades (§5.1).
- **La salida de un colaborador** la registra hoy `organization`. Dónde debe vivir es la decisión pendiente 2 del mapa (§11).

**Vocabulario (D-39)**

| Palabra | Qué es | Dónde vive |
|---|---|---|
| Rol | El conjunto de permisos de una cuenta, con su alcance | `iam` (`roles`, `user_roles`) |
| Puesto | El trabajo del colaborador: asesor, mecánico, contador. Es informativo y no da permisos | `organization` (`employees.position`) |
| Perfil | Los datos personales del colaborador (D-4) | `organization` (`employees`) |

"Cargo" no se usa. El comentario de cabecera de la migración V1 todavía dice "cargos"; una migración aplicada no se edita.

**Cuenta y colaborador.** Son dos cosas distintas. Quien trabaja para dos empresas del grupo tiene dos colaboradores, uno por empresa, y una sola cuenta (§5.3). Un mecánico puede ser colaborador sin cuenta. Las únicas cuentas sin colaborador son `SYSTEM`, que no es una persona, y el primer superadministrador hasta que se vincula a su perfil (§5.2).

---

## 2. Ciclo de vida de cada agregado

### Cuenta (`User`)

| Momento | Qué dice el diseño | Hoy |
|---|---|---|
| Nace | Al dar acceso a un colaborador o por el registro público, siempre vinculada a un colaborador (D-38). `SYSTEM` nace en la migración y el primer superadministrador al arrancar (D-34) | Nace por cuatro caminos: el arranque (F1), `POST /users` desde Usuarios (F5), el registro público (F4) y dar acceso desde `organization` (F6). Los dos primeros y el registro crean cuentas sin colaborador |
| Contraseña provisional | Quien recibe una clave de un administrador debe cambiarla en su primer ingreso (§5.4) | Se cumple. Con `must_change_password` la cuenta solo consulta quién es y cambia su contraseña (F3) |
| Cambia la contraseña | Entre 8 y 72 caracteres, distinta de la actual; cierra las demás sesiones (§5.4, §5.12) | Se cumple |
| Se restablece la contraseña | El diccionario no describe esta operación | Existe en el backend como `POST /employees/{id}/password-reset`, de `organization` (F8) |
| Se bloquea, se da de baja o se reactiva | Por períodos en `user_access_suspensions`; activa es la cuenta sin bloqueo abierto; bloquear cierra sus sesiones (§5.4) | No existe. Ninguna cuenta se puede bloquear |
| Queda fijo | El autor y el momento de creación | Se cumple. Tampoco hay ninguna operación para cambiar el correo |
| Nunca pasa | Se borra (§5.4) | No hay `DELETE`. La cuenta de la aplicación no tiene permiso de borrar en `users` (V19), cuando esa cuenta existe |

### Rol (`Role`)

Los seis roles del sistema (`SUPERADMIN`, `ADMIN_EMPRESA`, `OPERACIONES`, `CONTABILIDAD`, `ASESOR_VENTAS`, `TALLER`) llegan en la migración V1, marcados como propios del sistema, con su alcance: `SYSTEM`, `COMPANY` o `BRANCH` (§5.5). Hoy solo se leen (`GET /roles`). Crear roles desde la pantalla (D-7) espera a `roles.company_id` y a los permisos. El alcance de un rol se fija al crearlo y no cambia (§5.5, regla 4). Un rol del sistema no se edita ni se borra.

### Asignación (`RoleAssignment`)

| Momento | Qué pasa |
|---|---|
| Se otorga | Queda la fecha y quién la otorgó (`granted_by`). El alcance se copia del rol |
| Se retira | Se marca la fecha de retiro y el motivo, uno de cuatro: `retiro`, `caducidad`, `baja_colaborador`, `baja_cuenta` (D-41). Nunca se borra (§5.7, §5.8) |
| Caduca | Si tiene `expires_at`, deja de valer en esa fecha y se cierra con motivo `caducidad` (§5.8) |

Hoy se otorga en el arranque, en `POST /users` y al dar acceso. Se retira solo por dos caminos: la salida de un colaborador (motivo `baja_colaborador`) y la sincronización de roles al dar acceso (motivo `ROLE_SCOPE_CHANGE`, que no es uno de los cuatro). Nada fija `expires_at`, así que nada caduca.

### Sesión

| Momento | Regla (§5.12, D-33) | Hoy |
|---|---|---|
| Se abre | Al iniciar sesión, con un identificador nuevo | Se cumple |
| Cada petición | Roles y contraseña provisional se leen de la base, no de la sesión | Se cumple |
| Se cierra sola | A los 30 minutos sin uso o a las 8 horas desde que se abrió | Se cumple |
| Se cierra por un hecho | Al cerrar sesión; al cambiar la contraseña, las demás; al bloquear la cuenta, todas | Cerrar sesión y cambiar la contraseña, sí. Bloquear no existe. Restablecer la contraseña y la salida del colaborador no cierran sesiones |

---

## 3. Flujos funcionales

### F1 · Puesta en marcha (P1, pasos 1 a 3)

- **Quién:** nadie desde la aplicación. Lo hace la aplicación al arrancar (D-34).
- **Pantalla:** ninguna. El primer ingreso termina en Cambio de contraseña.
- **Pasos:** la migración V1 crea los roles y la cuenta `SYSTEM`. Al arrancar, si ya hay un superadministrador activo, no hace nada. Si faltan `IAM_BOOTSTRAP_ADMIN_EMAIL` o `IAM_BOOTSTRAP_ADMIN_PASSWORD`, arranca sin superadministrador y lo advierte en el registro de eventos. Si no, crea la cuenta con esa contraseña provisional, a nombre de `SYSTEM`, y le otorga `SUPERADMIN` a nombre de `SYSTEM`.
- **Errores:** si el correo ya pertenece a otra cuenta, la aplicación no arranca (§5.2).
- **Resultado:** un superadministrador sin perfil, que debe cambiar su contraseña. El perfil de desarrollo trae un correo y una contraseña por defecto (`application-dev.yaml`); en producción no hay valor por defecto.
- **Diferencia:** §5.2 dice que hasta vincularse a su perfil no puede dar acceso ni otorgar roles. Hoy puede (IAM-R14).

### F2 · Iniciar y cerrar sesión (P2, D-33)

- **Quién:** cualquier persona con cuenta.
- **Pantalla:** Inicio de sesión (`/iam/sign-in`), construida.
- **Pasos:** la persona escribe correo y contraseña. El sistema guarda el correo sin espacios y en minúsculas, compara la contraseña cifrada, registra el último acceso, abre la sesión con un identificador nuevo y devuelve quién es, sus roles y si debe cambiar la contraseña. La pantalla la lleva a Inicio, o a Cambio de contraseña si la clave es provisional. Al recargar la página, el frontend pregunta quién es (`GET /authentication/me`) antes de la primera navegación.
- **Errores:** un correo inexistente y una contraseña equivocada reciben la misma respuesta (`401`). Sin token CSRF, `403`.
- **Cerrar sesión:** `POST /authentication/sign-out` borra la sesión en el servidor y responde `204`. Si cualquier llamada responde `401` porque la sesión venció, el frontend vuelve a Inicio de sesión.

### F3 · Primer ingreso y cambio de contraseña obligatorio (§5.4)

- **Quién:** una cuenta con contraseña provisional.
- **Pantalla:** Cambio de contraseña (`/iam/change-password`), construida.
- **Pasos:** mientras la clave es provisional, el backend reemplaza sus roles por un único permiso de cambiar la contraseña: todo lo demás responde `403`, salvo `me` y `change-password`. El frontend la redirige a esta pantalla desde cualquier ruta. La persona escribe la clave actual y la nueva. El sistema revisa que la actual sea correcta, que la nueva tenga entre 8 y 72 caracteres y que sea distinta de la actual.
- **Errores:** clave actual incorrecta (`400`), longitud fuera de rango (`400`), clave igual a la actual (`422`).
- **Resultado:** la cuenta queda liberada, se registra la fecha del cambio y se cierran sus demás sesiones.

### F4 · Registro público (D-32)

- **Quién:** cualquier persona, solo si `IAM_PUBLIC_SIGN_UP_ENABLED` está encendido. Viene apagado.
- **Pantalla:** Registro (`/iam/sign-up`), construida sin perfil. Solo se abre si `GET /authentication/sign-up/status` responde que está encendido.
- **Lo construido:** correo y contraseña (8 a 72). Crea una cuenta sin roles, a nombre de `SYSTEM`, que no tiene que cambiar la contraseña. No abre sesión: la persona entra después por F2.
- **Lo que exige el diccionario y falta:** elegir empresa y sede sin sesión, completar los datos obligatorios del colaborador, crear el perfil y la cuenta en la misma operación, rechazar un documento ya registrado en esa empresa (§5.2, D-32, D-38). La ruta `sign-up/companies` que nombra convenciones §9.2 no existe; las listas de sedes y documentos están pendientes (convenciones §14.16).
- **Errores:** registro apagado (`403`), correo ya registrado (`409`).
- **Resultado:** una cuenta que puede iniciar sesión y no ve nada, porque toda operación exige un rol (§5.2, D-43).

### F5 · Crear una cuenta desde Usuarios

- **Quién:** `SUPERADMIN`.
- **Pantalla:** Seguridad › Usuarios › Nuevo usuario (`/iam/users/new`), construida.
- **Pasos:** correo, contraseña provisional (8 a 72) y roles. La pantalla ofrece solo roles de alcance sistema. El sistema revisa que el correo no exista, que cada rol exista, que sea de alcance sistema y que quien crea la cuenta tenga ese rol vigente (§5.13).
- **Errores:** correo repetido (`409`), rol inexistente (`404`), rol de empresa o de sede (`422`), rol que quien opera no tiene (`403`).
- **Resultado:** una cuenta que debe cambiar su contraseña, creada por la cuenta de la sesión, con sus roles otorgados por ella. **No tiene colaborador**, contra D-38. Convenciones §13 fila 16 la reemplaza en el paso 3 por dar acceso desde el colaborador, y `/users` queda solo para leer.

### F6 · Dar acceso a un colaborador (D-38; la operación es de `organization`)

El flujo completo está en [organization.md](organization.md). Aquí va solo lo que hace `iam`.

- **Quién y dónde:** `SUPERADMIN` o `ADMIN_EMPRESA` de la empresa del colaborador, desde el diálogo de dar acceso de Colaboradores. El backend es `POST /employees/{id}/access`, que llama a `IamContextFacade.provisionEmployeeUser` con el correo y los códigos de rol.
- **Lo que hace `iam`:** busca una cuenta con ese correo. Si existe, la reutiliza y no entrega clave. Si no, la crea con una clave provisional de 12 caracteres que la respuesta trae una sola vez. Después **sincroniza** los roles: otorga los pedidos que faltan y retira todos los vigentes que no se pidieron, con motivo `ROLE_SCOPE_CHANGE`. Si el colaborador ya tenía cuenta, el mismo camino solo sincroniza.
- **Lo que `iam` no revisa:** si la cuenta encontrada ya pertenece a otra persona o es `SYSTEM`, si quien opera tiene los roles que otorga, si con el retiro queda algún superadministrador activo, ni el alcance de cada rol. Ver §6, punto 1.
- **Resultado:** la cuenta queda vinculada al colaborador en `organization`.

### F7 · Retirar los roles al registrar la salida de un colaborador (D-38; la operación es de `organization`)

- **Quién y dónde:** `POST /employees/{id}/termination`, desde la ficha del colaborador. Ver [organization.md](organization.md).
- **Lo que hace `iam`:** responde si la cuenta es la del último superadministrador activo; si lo es, `organization` rechaza la salida (`422`). Si no, retira **todos** los roles vigentes de la cuenta con motivo `baja_colaborador`.
- **Diferencia con D-38:** se deberían retirar solo los roles de esa empresa y, si a la cuenta no le queda ningún colaborador activo, bloquearla con motivo `baja` y cerrar sus sesiones. Hoy no hay roles por empresa ni bloqueos, y las sesiones siguen abiertas; la persona pierde sus roles en la petición siguiente.

### F8 · Restablecer la contraseña (no figura en el diccionario)

- **Backend:** `POST /employees/{id}/password-reset`, de `organization`. Lo puede pedir `SUPERADMIN` para cualquiera y `ADMIN_EMPRESA` para un colaborador activo de su empresa que no sea superadministrador. `iam` genera una clave provisional de 12 caracteres, obliga a cambiarla y la devuelve en la respuesta.
- **Frontend:** la acción de fila "Restablecer contraseña" de Usuarios llama a `POST /users/{id}/password-reset`, que no existe en el backend, así que falla. Ninguna pantalla llama a la ruta de `organization`.
- **Resultado en el backend:** las sesiones abiertas no se cierran, pero desde la petición siguiente la cuenta solo puede consultar quién es y cambiar su contraseña.

---

## 4. Reglas de negocio

| ID | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| IAM-R01 | La contraseña tiene entre 8 y 72 caracteres, sin reglas de composición | §5.4 | Servicio (validación de los recursos de sign-up, change-password y `POST /users`); Frontend (los tres formularios) | Sin prueba | ⚠️ |
| IAM-R02 | La nueva contraseña es distinta de la actual | §5.4 | Servicio | Sin prueba | ⚠️ |
| IAM-R03 | Un correo inexistente y una contraseña equivocada reciben la misma respuesta | §5.4 | Servicio | `UserCommandServiceImplTest.signInDoesNotRevealWhetherTheAccountExists` | ✅ |
| IAM-R04 | El correo no se repite y se guarda sin espacios y en minúsculas | §5.4 | Base (`UNIQUE`, `users_email_normalized`); Dominio | `UserTest.newAccountHasANormalizedEmail…`, `UserCommandServiceImplTest.anEmailCannotBeRegisteredTwice` | ✅ |
| IAM-R05 | Con contraseña provisional la cuenta solo consulta quién es y cambia su contraseña; sus roles no valen | §5.4, §5.12 | Servicio (cadena de seguridad); Frontend (`iamGuard`) | Frontend: `iam.guard.spec.ts`. Backend: sin prueba | ⚠️ |
| IAM-R06 | La sesión se cierra a los 30 minutos sin uso y a las 8 horas desde que se abrió | §5.12, D-33 | Servicio (configuración y `SessionUserRefreshFilter`) | Sin prueba | ⚠️ |
| IAM-R07 | Roles y contraseña provisional se leen en cada petición: un rol retirado o una clave restablecida valen desde la petición siguiente | §5.12 | Servicio (`SessionUserRefreshFilter`) | Sin prueba | ⚠️ |
| IAM-R08 | Cambiar la contraseña cierra las demás sesiones | §5.12, D-33 | Servicio | `UserCommandServiceImplTest.changingThePasswordClosesTheOtherSessions` (con un doble del servicio de sesiones) | ✅ |
| IAM-R09 | Cerrar sesión borra la sesión en el servidor | §5.12 | Servicio (cierre de sesión de Spring Security) | Sin prueba | ⚠️ |
| IAM-R10 | La cookie `MM_SESSION` es `HttpOnly`, `SameSite=Strict` y `Secure` fuera de desarrollo; al iniciar sesión cambia el identificador | §5.12 | Servicio (configuración, `SessionServiceImpl`) | Sin prueba | ⚠️ |
| IAM-R11 | Toda petición que cambia datos lleva el token CSRF, también iniciar sesión y registrarse; API y frontend en el mismo origen | §5.12, convenciones §9.1 | Servicio (`csrf.spa()`); Frontend (encabezado `X-XSRF-TOKEN`, proxy) | Ninguna prueba verifica el rechazo sin token | ⚠️ |
| IAM-R12 | `SYSTEM` no puede iniciar sesión | §5.9 | Base (la migración V1 le pone un valor que no es un hash) | Sin prueba | ⚠️ Se puede romper por F6 y F8 (§6, punto 1) |
| IAM-R13 | El primer superadministrador se crea al arrancar solo si no hay un superadministrador activo; sin variables arranca sin él; con un correo ya registrado no arranca | §5.2, D-34 | Servicio (`ApplicationReadyEventHandler`, `UserCommandServiceImpl`) | Sin prueba | ⚠️ |
| IAM-R14 | El primer superadministrador no da acceso ni otorga roles hasta vincularse a su perfil | §5.2, §5.9 | — | — | ❌ |
| IAM-R15 | El registro público viene apagado; apagado, la API responde `403` | §5.2, D-32 | Servicio; Frontend (`signUpEnabledGuard`) | `UserCommandServiceImplTest.signUpIsRejectedWhileDisabled`; `iam.guard.spec.ts` | ✅ |
| IAM-R16 | El registro público crea perfil y cuenta juntos, sin roles, a nombre de `SYSTEM`, sin cambio forzado; rechaza correo o documento repetidos | §5.2, D-32 | Servicio, solo la parte de la cuenta | `UserCommandServiceImplTest.signUpCreatesAnAccountWithoutRolesAuthoredBySystem`, `signUpCannotReuseARegisteredEmail` | ⚠️ Falta el perfil |
| IAM-R17 | Toda cuenta pertenece a un colaborador y nace vinculada a él | §5.3, D-38 | Solo en dar acceso (F6) | — | ❌ El registro y `POST /users` crean cuentas sueltas |
| IAM-R18 | Un rol exige que la cuenta esté vinculada a un colaborador activo: de esa empresa para un rol de empresa o de sede, de alguna para uno de sistema | §5.3, §5.7, D-38 | Servicio de `organization`, solo en dar acceso | `EmployeeCommandServiceImplTests.shouldFailGrantingAccessWhenEmployeeTerminated` | ⚠️ `POST /users` no lo revisa |
| IAM-R19 | Nadie otorga un rol que no tiene; hoy, tener ese mismo rol vigente | §5.5 reglas 1 y 2, §5.13 | Dominio (`User.canGrant`) y Servicio, solo en `POST /users` | `UserTest.grantsOnlyTheRolesItHoldsInForce`, `cannotGrantItselfARoleItDoesNotHold`; `UserCommandServiceImplTest.anAccountWithoutTheRoleCannotGrantIt` | ⚠️ Al dar acceso no se revisa |
| IAM-R20 | `POST /users` otorga solo roles de alcance sistema | §5.13 | Servicio; Frontend (Nuevo usuario) | `UserCommandServiceImplTest.companyRolesCannotBeGrantedWithoutTheCompanyTheyApplyTo` | ✅ |
| IAM-R21 | No se puede retirar al último superadministrador activo: ni quitándole el rol, ni bloqueando su cuenta, ni registrando su salida | §5.9 | Servicio de `organization`, solo en la salida (F7) | `EmployeeCommandServiceImplTests.shouldFailTerminationWhenLastActiveSuperadmin` (con `iam` simulado) | ⚠️ Dar acceso puede retirarle el rol |
| IAM-R22 | Un rol se retira, no se borra, y siempre con motivo | §5.7, §5.8 | Base (`user_roles_revocation_reason_ck`; sin permiso de borrar, V19); Dominio | Sin prueba | ⚠️ |
| IAM-R23 | El motivo de retiro es `retiro`, `caducidad`, `baja_colaborador` o `baja_cuenta` | D-41 | — | — | ❌ Se usa `ROLE_SCOPE_CHANGE` y la base acepta cualquier texto |
| IAM-R24 | El mismo rol no se da dos veces mientras siga vigente; el alcance guardado coincide con el del rol | §5.7 | Base (`user_roles_active_uq`, `user_roles_role_scope_fk`) | Sin prueba | ⚠️ |
| IAM-R25 | Un rol vale sobre una empresa, sede o marca; retirarlo en una empresa no toca las otras | §2.5, §5.7, D-38 | — | — | ❌ `user_roles` no tiene esas columnas |
| IAM-R26 | Un acceso vencido se cierra con motivo `caducidad` | §5.8 | — | — | ❌ |
| IAM-R27 | Las cuentas se bloquean por períodos; activa es la que no tiene un bloqueo abierto; bloquear cierra sus sesiones; la salida del último colaborador activo bloquea la cuenta con motivo `baja` | §5.3, §5.4, D-12, D-38 | — | — | ❌ |
| IAM-R28 | El autor lo pone el sistema con la cuenta de la sesión; `SYSTEM` solo de forma explícita y solo sin sesión; nunca se supone | D-42 | Servicio (`CurrentUserAuditorAware`) | `CurrentUserAuditorAwareTest`, `UserAuthorshipTests` | ⚠️ Dar acceso pone `SYSTEM` si no encuentra la cuenta de la sesión |
| IAM-R29 | Ninguna operación se conforma con la sesión: Usuarios exige `SUPERADMIN`; Roles, `SUPERADMIN` o `ADMIN_EMPRESA` | §5.2, convenciones §9.2 | Servicio (`@PreAuthorize`) | `RolesControllerTests`. Usuarios: sin prueba | ⚠️ |
| IAM-R30 | Una cuenta nunca se borra; un rol del sistema no se edita ni se borra | §5.4, §5.5 | No hay operación que lo haga; Base (sin permiso de borrar, V19) | Sin prueba | ⚠️ |

---

## 5. Relación con otros contextos

**Lo que ofrece: `IamContextFacade`** (`interfaces.acl`, interfaz con nombre `acl`). Su único consumidor es `organization`, a través de `ExternalIamService`.

| Miembro | Para qué | Lo usa `organization` |
|---|---|---|
| `fetchCurrentUserId` | Cuenta de la petición actual | Sí: alcance del administrador de empresa |
| `fetchUserPublicIdByUserId`, `fetchUserEmailByUserId` | Mostrar la cuenta de un colaborador | Sí |
| `fetchUsersSummaryByUserIds`, `fetchUsersActiveRoleCodesByUserIds` | Lo mismo, en lote para las listas | Sí |
| `fetchUserActiveRoleCodesByUserId` | Roles vigentes de una cuenta | Sí: saber si el colaborador es superadministrador |
| `provisionEmployeeUser`, `updateEmployeeUserRoles` | Dar acceso y sincronizar roles (F6) | Sí |
| `isLastActiveSuperadmin`, `revokeUserRoles` | Salida del colaborador (F7) | Sí |
| `resetUserPassword` | Restablecer la contraseña (F8) | Sí |
| `fetchUserIdByPublicId`, `fetchUsersPublicIdByUserIds`, `fetchUsersEmailByUserIds` | Traducir identificadores | No |

Además, `iam` aporta a la auditoría común el `CurrentUserAuditorAware`, que llena `created_by` y `updated_by` en todos los contextos (D-42). Spring lo inyecta, sin dependencia de código.

**Lo que necesita de otros.** Hoy nada: `iam` declara `allowedDependencies = {}`. Usa el núcleo `shared` (resultados y errores, clases base de persistencia, exportación a Excel).

**Divergencia abierta: el sentido de la dependencia.** El mapa propone que `iam` llame a `organization` y nunca al revés (mapa §4, §6.2): dar acceso y el registro público vivirían en `iam`, que pediría a `organization` el colaborador. El código hizo lo contrario: `organization` declara `iam::acl` en su `package-info` y llama a `iam` para dar acceso, retirar roles y restablecer claves. Con las dos direcciones habría un ciclo, que Spring Modulith rechaza, así que hay que elegir una. Es la decisión pendiente 2 del mapa (§11), y también la 4 en lo que toca al sentido de las dependencias. Hasta que se decida, el código no cumple la propuesta del mapa.

**Frontend: `IamFacade`.** Es lo único que otros contextos del frontend usan de `iam`.

| Miembro | Quién lo usa |
|---|---|
| `hasRole(code)` | `app-shell` (menú) y las pantallas de `organization` |
| `hasAnyRole` | `app-shell` (opciones que piden cualquier rol) |
| `restoreSession()` | `app.config`, una vez al arrancar |
| `currentCompanyId` | Colaboradores y su formulario, para preseleccionar la empresa del administrador. **Siempre es `null`**: `/authentication/me` no devuelve `companyId` (§6, punto 4) |

---

## 6. Estado y pendientes para cerrar

**Lo construido (paso 0).** Inicio y cierre de sesión con sesiones en el servidor, contraseña provisional obligatoria, cambio de contraseña, registro público sin perfil, arranque del primer superadministrador, lista y exportación de Usuarios, Nuevo usuario y lectura de roles. El paso 3 trae lo que falta (§5.13).

### a) Decisiones

| Qué | Quién la toma |
|---|---|
| Dónde vive la salida del colaborador y cómo se protege al último superadministrador, y con eso el sentido de la dependencia entre `iam` y `organization` (§5) | Mapa §11, decisión 2 |
| Si existe el restablecimiento de contraseña, quién lo pide y por qué ruta. Hoy está en `organization` y no figura en el diccionario ni en convenciones §2.2 | Responsable del diccionario |
| La regla provisional para el administrador de empresa. El código le prohíbe otorgar `SUPERADMIN` y `ADMIN_EMPRESA` con una lista fija; §5.5 y §5.13 piden comparar lo que tiene quien otorga | Responsable del diccionario |
| Qué empresa "actual" tiene una cuenta con colaboradores en varias empresas, si el frontend la necesita (§6, punto 4) | Convenciones §14.10 |
| Ruta y respuesta de dar acceso; qué pasa con la clave cuando la cuenta ya existe | Convenciones §14.15 |

### b) Código

| # | Gravedad | Qué pasa hoy | Qué falta |
|---|---|---|---|
| 1 | Crítica | Dar acceso (F6) reutiliza **cualquier** cuenta con el correo indicado y retira todos sus roles que no se pidieron. No revisa si la cuenta ya es de otra persona, si es `SYSTEM`, si quien opera tiene los roles, ni si queda un superadministrador. Un `ADMIN_EMPRESA` puede dar acceso a un colaborador nuevo de su empresa con el correo de un superadministrador, incluso el último o el del arranque, que no tiene perfil: le retira `SUPERADMIN`, la cuenta queda vinculada a su colaborador y después le restablece la contraseña con `/employees/{id}/password-reset` (F8), que ya no lo ve como superadministrador. Con el correo de `SYSTEM`, esa cuenta recibe una clave y puede iniciar sesión (IAM-R12). Además, un código `" SUPERADMIN"` con espacios pasa el filtro de `organization` (`canGrantRoles` pasa a mayúsculas sin recortar) y `ScopedRoleAssignment` lo recorta después: el administrador de empresa otorga `SUPERADMIN` | Que `iam` revise en `provisionEmployeeUser` y `updateEmployeeUserRoles` lo mismo que en `POST /users`: quien otorga tiene el rol (IAM-R19), nunca `SYSTEM`, nunca el último superadministrador (IAM-R21). Reutilizar una cuenta existente solo para vincularla, sin tocar sus roles de otra empresa (D-38). Validar los códigos de rol en `iam`, no solo en la pantalla y en `organization` |
| 2 | Alta | `user_roles` no tiene empresa, sede ni marca. La salida de un colaborador retira los roles de la cuenta en **todas** las empresas y nunca bloquea la cuenta. Dar acceso en una segunda empresa retira los roles de la primera. Dar acceso escribe roles de empresa y de sede sin alcance (la sede y la marca se validan y se descartan), mientras `POST /users` los rechaza; el bloqueo a roles de sistema que el orden de implementación quería quitar recién con RLS ya está saltado por este camino | Las columnas de alcance de §5.7 con sus restricciones, `user_access_suspensions` y la salida por empresa con bloqueo `baja` (D-38). Paso 3 |
| 3 | Alta | Usuarios llama a `PUT /users/{id}/roles` y a `POST /users/{id}/password-reset`, que no existen. Los commits de "suspensiones" de `iam` en el frontend (`178b67c`, `e2774d9`, `4c7b71f`, `e13605b`) no traen código de suspensión: solo una ruta `/suspensions` sin uso y un texto para un código `ACCOUNT_SUSPENDED` que el backend no envía. El frontend de `organization` manda los bloqueos a `/employees/{id}/suspensions`, sin backend, aunque la tabla es de `iam` | Alinear con convenciones §2.2: `/users/{id}/role-assignments`, `/role-assignments/{id}/revocation`, `/users/{id}/suspensions`, `/suspensions/{id}/closure`. Quitar del frontend las llamadas a rutas inexistentes hasta que existan |
| 4 | Alta | `/authentication/me` y el inicio de sesión no devuelven `companyId`, aunque el frontend lo espera (commit `37cc7a4`). `currentCompanyId` siempre es `null` | Decidir qué significa (§6 a) y alinear backend y frontend |
| 5 | Media | La sincronización retira con motivo `ROLE_SCOPE_CHANGE` (D-41) | Usar los cuatro motivos y agregar el `CHECK` en la base |
| 6 | Media | Dar acceso pone `SYSTEM` como autor si no encuentra la cuenta de la sesión (D-42) | Rechazar la operación sin sesión |
| 7 | Media | El registro público y `POST /users` crean cuentas sin colaborador (D-32, D-38) | Registro con perfil y quitar el alta de `POST /users` (convenciones §13 fila 16). Paso 3 |
| 8 | Media | Sin pruebas: `AuthenticationController`, `UsersController`, dar acceso y la sincronización, el retiro de roles, el restablecimiento, el arranque, el filtro de sesión y el rechazo sin CSRF | Pruebas de cada regla marcada "sin prueba" en §4 |
| 9 | Media | Las pruebas de extremo a extremo de Usuarios fallan en los tres anchos, corridas el 14/10/2026: accesibilidad (WCAG A/AA y orden de encabezados) y captura visual. La lista cambió el 14/10 (`e13605b`) y su captura de referencia es del 06/10 | Corregir la accesibilidad y regenerar la captura |
| 10 | Media | Transversal: un error inesperado devuelve el mensaje de la excepción en `details` (`GlobalExceptionHandler`) | Convenciones §13 fila 14 |
| 11 | Baja | La salida del colaborador y el restablecimiento no cierran sesiones. La persona pierde las operaciones de negocio en la petición siguiente, pero `me` y `change-password` siguen respondiendo | Cerrar las sesiones de la cuenta, como pide §5.4 para el bloqueo |

**Pendientes transversales del orden de implementación que tocan a `iam`**

| Qué | Prioridad |
|---|---|
| La sesión guarda el hash de la contraseña: `UserDetailsImpl` lo lleva y se serializa en `spring_session_attributes` | Media |
| Un correo de más de 255 caracteres responde `500`: los recursos no limitan su largo | Baja |
| Cerrar como `caducidad` las asignaciones vencidas (IAM-R26) | Con la pantalla de roles y usuarios |
| RLS sobre `company_id` y quitar el bloqueo a roles de sistema de `UserCommandServiceImpl` en el mismo cambio | Primera rebanada del paso 3 |
| Filtro de alcance y `/users` abierto al administrador de empresa | Paso 3 |
| Clases base de registro que faltan, para `user_access_suspensions` | Paso 3 |
| Quitar el N+1 de los roles en la lista de usuarios (colección `EAGER`) | Baja |

### c) Datos

- La matriz de permisos por rol (§14; procesos, pregunta 5). Sin ella no hay `permissions` ni la comparación de §5.5.
- Las asignaciones ya retiradas con `ROLE_SCOPE_CHANGE` en las bases donde se dio acceso tendrán que llevarse a uno de los cuatro motivos antes de agregar el `CHECK`.
- El primer superadministrador de cada entorno tendrá que vincularse a su perfil (P1, paso 6). Hoy no hay una operación que solo vincule.

### d) Documentación

- Mapa §5.1 describe `IamContextFacade` con dos capacidades; hoy tiene catorce miembros (§5). Mapa §9 dice que `iam` declarará `organization::acl`; el código declaró `iam::acl` en `organization`.
- Diccionario §5.13 está desactualizado: los colaboradores ya existen (V22) y la salida ya retira roles. Lo actualiza quien es responsable del diccionario.
- Convenciones §2.2 no nombra `/employees/{id}/access`, `/termination`, `/reactivation` ni `/password-reset`, que ya existen.

---

## 7. Preguntas abiertas

1. **¿Dónde se registra la salida de un colaborador y cómo se protege al último superadministrador?** Decide también el sentido de la dependencia entre `iam` y `organization`. [Mapa de contextos](../mapa-de-contextos-millennium-mobility.md), §11, decisiones 2 y 4.
2. **¿Qué permisos tiene cada rol del sistema?** [Procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), apartado 5, pregunta 5; diccionario §14.
3. **¿Cómo se llama la ruta de dar acceso y qué responde cuando la cuenta ya existe?** [Convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), §14.15.
4. **¿Cómo obtiene el registro público, sin sesión, las sedes y los tipos de documento?** Convenciones §14.16.
5. **¿Por qué ruta se registra la salida?** Convenciones §14.17, que depende de la pregunta 1.
6. **¿Lo llena el sistema a `granted_by` con la cuenta de la sesión?** Convenciones §14.26.
7. **¿Cómo se muestran las referencias a cuentas fuera de `iam`?** Mapa §11, decisión 6.
8. **¿Existe el restablecimiento de contraseña y quién puede pedirlo?** Ningún documento lo lleva todavía. **(propuesta)** Registrarlo en convenciones §14 junto con la pregunta 3.
