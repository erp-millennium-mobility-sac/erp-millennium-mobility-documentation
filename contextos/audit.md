# Contexto `audit` — ERP Millennium Mobility

**Fuente:** diccionario de datos (versión vigente; §13, D-102, D-105, D-129, D-149, D-161, D-193, D-198 y D-262), el [mapa de contextos](../mapa-de-contextos-millennium-mobility.md) (§5.1, §5.3 y §6.5) y el código de `develop`. Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** `audit` es la bitácora de cambios, un módulo de la plataforma desde la fase 1 del plan de migración (D-262); antes vivía en `shared`. Las columnas de `change_logs` están en el diccionario (§13.1), la ruta y sus errores en las convenciones (§2.2 y §7.2), y la pantalla Herramientas › Bitácora de cambios en procesos y pantallas.

---

## 1. Qué es y qué no es

**Qué es.** El registro de **qué hizo alguien con un registro**: quién cambió qué campo, de qué valor a cuál, cuándo y, si el cambio es delicado, por qué. Por ejemplo, la corrección de una placa mal digitada, de una tasa de cambio o de la fecha de una cesión, o la anulación de una factura (§13).

**Tabla que posee:** `change_logs` (§13.1, V59 a V61).

**Lo que parece suyo y no lo es:**

- **Lo que pasó con una unidad, una cesión, un equipo o una cuenta** lo responden sus propias historias —los estados, las cesiones, las instalaciones, las asignaciones y los bloqueos—, de sus contextos. La bitácora responde otra pregunta y no reemplaza a ninguna (§13.2).
- **Qué cambio es delicado y qué campo es confidencial** lo decide el contexto que hace el cambio, dueño de esa regla, no la bitácora (AUD-R05).
- **El nombre de una cuenta y la empresa de una fila** son de `iam` y de `organization`: la bitácora los pide por sus puertos, sin leer sus tablas (AUD-R11).
- **Las altas y las bajas** no dejan filas: una fila nueva ya registra quién la creó, y nada se borra en este sistema (§13.1).

## 2. Ciclo de vida del agregado

**`ChangeLogEntry`** es una fila por campo cambiado. Nace en la misma transacción que el cambio que registra, por `ChangeLogCommandService`, y desde entonces no cambia ni se borra (AUD-R01): un registro equivocado se corrige con un cambio nuevo, que deja su propia fila. No tiene estados.

## 3. Flujos funcionales

### F1 · Registrar un cambio

- **Quién:** el servicio de un contexto que modifica un registro cuyo cambio la bitácora guarda: `location` (la corrección de una tasa de cambio), `fleet` (placas, VIN, hitos, traspasos), `financing` (el cierre de una cesión, a mano o por la carga de la garantía) y `procurement` (proveedores, plantillas, anulaciones, aplicaciones de pago).
- **Pasos:** el contexto pide `recordChange` o `recordChanges` por `audit::acl`, con la tabla, el identificador interno y el público del registro, su empresa y su marca, el campo, los dos valores, si el cambio es delicado y el motivo.
- **Qué revisa el sistema:** que la entidad y el campo no vayan vacíos y que la marca sea de la empresa (AUD-R03, AUD-R07); que un cambio delicado traiga su motivo (AUD-R04).
- **Si falla:** la anotación vuelve como error y el contexto descarta sus escrituras: el cambio y su fila se guardan juntos o no se guarda ninguno.

### F2 · Leer la bitácora

- **Quién:** quien tiene `bitacora.ver`, en Herramientas › Bitácora de cambios (`GET /api/v1/change-logs`), o desde la ficha de un registro, que enlaza a su historia con `entity` y `entityPublicId`.
- **Qué revisa el sistema:** las filas que alcanza con el permiso, por empresa y por marca (AUD-R06); los valores que puede ver de un campo confidencial (AUD-R08); los filtros y el período (AUD-R09).
- **Resultado:** cada fila nombra a quien hizo el cambio por su colaborador (AUD-R11).

### F3 · Leer la historia de las filas propias

- **Quién:** el dueño de una tabla, por `ChangeLogHistoryLookup`, sin el alcance de `bitacora.ver`: `procurement` dice de qué orden pasó la aplicación de un pago (AUD-R12).

## 4. Reglas de negocio

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| AUD-R01 | La bitácora no se edita ni se borra, ni siquiera con el alcance `SYSTEM`: un error se corrige con un cambio nuevo | §13.1 | Base | `ChangeLogRowLevelSecurityTests`, `HandOverDatabaseScriptTests` | ✅ |
| AUD-R02 | Un cambio deja una fila por campo, con su valor anterior y el nuevo; registra modificaciones, no altas, bajas ni consultas | §13.1 | Servicio | `ChangeLogCommandServiceTests`, `VehicleChangeLogIntegrationTests` | ✅ |
| AUD-R03 | Cada fila nombra la entidad, el campo y el identificador público de su registro, por el que la API la filtra | §13.1 | Base, Dominio | `ChangeLogEntryTests`, `ChangeLogRowLevelSecurityTests` | ✅ |
| AUD-R04 | Un cambio marcado como delicado exige motivo; sin él no se registra nada | §13.1, D-149, D-161, D-193, D-198 | Dominio, Servicio | `ChangeLogEntryTests`, `ChangeLogCommandServiceTests` | ✅ |
| AUD-R05 | Qué cambio es delicado y qué campo es confidencial lo dice el contexto que escribe: la bitácora solo exige el motivo de un cambio marcado y lee las marcas por `ConfidentialChangeLogFieldsPort` | §13.1, D-105 | Servicio | `ChangeLogEntryTests`, `ChangeLogAccessPolicyTests` | ✅ |
| AUD-R06 | Se lee con `bitacora.ver` donde actúa: cada empresa ve sus filas; una fila sin empresa, solo quien lo tiene en todas; una asignación acotada a una marca ve esa marca y lo que no es de ninguna | §13.1, D-99, D-102 | Base, Servicio | `ChangeLogAccessPolicyTests`, `ChangeLogsControllerTenantTests`, `ChangeLogRowLevelSecurityTests` | ✅ |
| AUD-R07 | La marca de una fila es de la empresa de la fila, y una marca sin empresa no se admite | §13.1, D-102 | Base, Dominio | `ChangeLogEntryTests`, `ChangeLogRowLevelSecurityTests` | ✅ |
| AUD-R08 | De un campo confidencial, quien no tiene su permiso donde actúa en la empresa de la fila —y en su marca, cuando el permiso va por marca— ve que cambió, sin sus valores (`valuesWithheld`). La marca se lee al consultar, así que cubre las filas ya escritas | §13.1, D-105, D-187 | Servicio | `ChangeLogAccessPolicyTests`, `PurchaseOrderTemplatesIntegrationTests`, `VehicleEquipmentIntegrationTests` | ✅ |
| AUD-R09 | Se filtra por entidad y registro, por campo y por un período `[desde, hasta)`, con el fin excluido; un período invertido se rechaza | §13.1, §2.6 | Servicio | `ChangeLogsSearchFilteringTests` | ✅ |
| AUD-R10 | La ruta solo lee: la bitácora no tiene `POST`, `PUT`, `PATCH` ni `DELETE` | §13.1 | Servicio | `ChangeLogsPolicyTests` | ✅ |
| AUD-R11 | Quién hizo cada cambio sale por su identificador público y el nombre de su colaborador, que resuelve `iam` en lote por `AuditActorLookupPort`; `SYSTEM` para lo automático. La empresa la traduce `organization` por `AuditCompanyLookupPort`. La bitácora no lee sus tablas | D-129 | Servicio | `VehicleChangeLogIntegrationTests` | ✅ |
| AUD-R12 | El dueño de una tabla lee la historia de sus propias filas sin el alcance de `bitacora.ver`, por `ChangeLogHistoryLookup`: el valor anterior más reciente y el más antiguo de un campo | D-129 | Servicio | `ChangeLogHistoryLookupTests` | ✅ |
| AUD-R13 | No guarda contraseñas ni el segundo factor | §13.1 | Servicio | Sin prueba propia: `iam` no escribe en la bitácora | ⚠️ |

## 5. Relación con otros contextos

**`audit::acl`** (`audit/interfaces/acl`) es su puerta, y `audit` no depende de nadie (`allowedDependencies = {}`; mapa de contextos §4):

| Miembro | Para qué | Quién lo usa o lo implementa |
|---|---|---|
| `ChangeLogCommandService` con `RecordChangeLogCommand` | Anotar un cambio en la transacción del contexto (F1) | `location`, `fleet` y `procurement` |
| `ChangeLogHistoryLookup` | La historia de las filas propias (F3) | `procurement` |
| `AuditActorLookupPort` con `AuditActor` | El nombre de una cuenta | Lo implementa `iam`; lo leen también `imports` y `location` |
| `AuditCompanyLookupPort` | La empresa de una fila y su zona | Lo implementa `organization`; lo lee también `imports` |
| `ConfidentialChangeLogFieldsPort` con `ConfidentialChangeLogField` | Los campos confidenciales | Lo implementan `fleet` y `procurement` |

## 6. Estado y pendientes para cerrar

| Qué | Cuándo |
|---|---|
| Una prueba de que ninguna anotación guarda una contraseña ni un secreto (AUD-R13) | Cuando un contexto que maneja secretos empiece a escribir en la bitácora |
| Que ningún contexto nombre `change_logs` en una consulta suya lo revisa la skill `architecture-review`; no hay una prueba que lo exija | Con la próxima revisión de arquitectura |

## 7. Preguntas abiertas

Ninguna propia. Los candidatos del bloque 11 —la dirección IP y el dispositivo, el registro de consultas— están en el diccionario, §13.3.
