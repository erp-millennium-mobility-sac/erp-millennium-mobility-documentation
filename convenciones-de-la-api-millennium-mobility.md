# Convenciones de la API REST — ERP Millennium Mobility

**Actualizado:** 29/10/2026 (decisiones del responsable del 29/10/2026: la orden de compra nombra su moneda por su código, `currency`, sin `currencyId` ni `currencyCode`; el candado de `updatedAt` en un solo lugar (§3), con el borrador de la orden, V85, y `PURCHASEORDER_OUTDATED`; los contactos de un proveedor por sus propias rutas, con `SUPPLIER_CONTACT_OUTDATED`, y el `PUT /suppliers/{id}` sin contactos; `CURRENCY_DECIMAL_PLACES_IN_USE` desde la primera orden en una moneda; `GET /vehicle-models`, `vehicleBrandName` en todo modelo y la exportación `all-vehicle-models`. Además: `SUPPLIER_CONTACT_NOT_FOUND` y `SUPPLIER_CONTACT_INACTIVE` en lugar de `PURCHASEORDER_CONTACT`; el `400` de una línea nula y de una tasa o un precio fuera de su columna; lo que la orden revisa activo, leído con candado compartido; las acciones de la orden en un orden fijo; las listas de compras en la tabla de §6.2; la advertencia en el registro de un duplicado sin traducir; la revisión de seguridad del 28/10/2026: V83, V84 y el arranque que rechaza una cuenta de base que se salta Row Level Security) · 26/10/2026 (cierre de la auditoría del 25/10/2026, diccionario v1.31, D-130, D-133 y D-134: las rutas de los fondos y el cierre de una cesión con su fecha en el cuerpo, `POST /fund-assignments/{id}/closure`; `allowedActions` y `codeLocked` del fondo, `allowedActions` de la cesión, del proveedor, de la plantilla, del tipo y de la orden, y `registerFundAssignment` de la unidad; los códigos de los fondos y de las cesiones, y los de compras —las referencias inactivas, un solo prefijo, el dígito de control del proveedor, las carreras traducidas, `_OUTDATED` con `422`, `PURCHASEORDERTEMPLATE_DOCUMENT_DIMENSIONS`—; el `413 CONTENT_TOO_LARGE` de una subida; `removedDocuments` en el `PATCH` de la plantilla como la única forma de quitar un archivo, sin ninguna ruta `DELETE`; el estado de la plantilla en `PATCH /{id}/status`, y el `PATCH /{id}` que responde `400` si trae `isActive`; el emisor fuera de los pedidos de la plantilla; los ocho permisos de plantillas, tipos y órdenes que parten los dos `gestionar_*` de compras (V79, D-130); las lecturas de compras solo con `compras.ver`; las exportaciones de proveedores y fondos) · 23/10/2026 (paso 7, en curso, diccionario v1.28: proveedores, tipos, plantillas y órdenes construidos; la plantilla se crea en un `POST` multipart y se corrige con `PATCH`, con el logo y la firma opcionales; su estado viaja en ese `PATCH` y ya no hay `/{id}/status`, `/{id}/signature` ni `/{id}/signature-removal`; los archivos son `/{id}/documents`; [contextos/procurement.md](contextos/procurement.md)) · 20/10/2026 (segunda ronda de la auditoría, segunda fase, diccionario v1.25, integrada en `develop`, merges `007151d` en el backend y `11235e0` en el frontend, y verificada contra el código: las cuentas, los accesos, los colaboradores y los roles por sus doce permisos `seguridad.*`, D-117 y D-122, con la tabla por permiso de §9.2 y `NoRoleAnnotationTests`; `/permission-matrix` como cuarta lista corta, con `grantableForGeneralRoles` y `grantableForCompanyRoles`, D-126; el orden de las revisiones de `PUT /roles/{id}/permissions` y de la copia; `administered` fuera de `/me`; la copia de un rol, D-124; `reserved`, `limitableToBrand`, `allowedActions` del colaborador, `systemPermissions` y `actsOnReserved`; los cuatro códigos nuevos de los roles y el `context` de `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`, D-119 y D-121) · 20/10/2026 (segunda ronda de la auditoría, primera fase, diccionario v1.25: el módulo `documents` y su contrato `documents::acl`, D-125; la versión de los datos de cada contexto sin nombrar tablas en `application`, su costo medido y la versión leída de los contadores de `data_versions`, D-128, con `DeadlockRetry`; la espera acotada con `documents.max-waiting` y el servicio de documentos partido; integrada en `develop`, merge `e245186`, y verificada contra el código) · 20/10/2026 (lo que la auditoría construyó, verificado contra el código integrado en `audit/be-integration` y `audit/fe-integration`: `VALIDATION_ERROR` responde `400`, `general=false` y `sort=system` en `/roles`, el texto de `COMMERCIALBRAND_INACTIVE`, el `409` de un `REGRESO_AUTOMATICO` que declaró una persona, el cuerpo de `/documents/{id}`, el lease y su dueño, la versión de los datos con `xmin`, el alcance vacío `NONE` y la depuración de los documentos) · 20/10/2026 (auditoría del 20/10/2026, diccionario v1.24: §8 reescrita como Documentos, generados una vez y reutilizados, D-106 y D-116; `/documents/{id}` y su archivo; `vehicleTypeLocked` y `allowedActions` como datos derivados, D-115; lo que buscan y ordenan los catálogos de flota, `/vehicles`, el historial, las lecturas y `/roles`, que pagina en el servidor; los estados y los caminos como tercera lista corta, D-113, y §14.23 adoptada; los códigos de los catálogos de flota, D-107 y D-108, de `fleet`, D-110, y de los documentos; un solo prefijo por agregado, D-114, que rechaza la primera parte de §14.11 y resuelve §13 fila 13) · 19/10/2026 (paso 3b, construido en `feature/permissions-matrix`, diccionario v1.23: las rutas de los roles y del catálogo de permisos, con `RoleDetailResponse`; `permissions` en `/me`; los códigos de los roles y de la matriz; `ACCESS_DENIED` también por el permiso que falta o no actúa, D-99; la política de acceso por permiso en §9.2; los tres campos del colaborador que se omiten, D-105; `role_permissions` sin `public_id`; §14.20 y §14.21 con D-104 y el catálogo) · 18/10/2026 (seguimiento de `organization`, construido en `feature/organization-followup`, diccionario v1.22: el `403` de las lecturas de colaboradores, D-93; `COMPANY_INACTIVE` al reactivar y el orden del regreso, D-94; `EMPLOYEE_TERMINATION_BEFORE_HIRE` en la edición; el `400` de un número hecho solo de separadores; las columnas de las exportaciones de colaboradores y ubicaciones) · 18/10/2026 (cierre de `organization`, diccionario v1.21: la empresa nombra su país y su moneda por su código, D-13, y §13 filas 21 y 22; `locationType` en minúsculas; el campo `context` del error; vincular la cuenta de otra persona con confirmación, D-82; `PUT /companies/{id}/tax-id`, D-85; `POST /employees/{id}/account-unlinking`, D-82; los códigos de lo que se cuelga de algo inactivo, D-84, de la fecha de ingreso, D-88, y de la ubicación propia de una sede, D-90; `employeeCompanyIds`; `sort` por el nombre del campo en ubicaciones y colaboradores) · 18/10/2026 (cierre de `location`: códigos propios de la dirección, del dígito de control y de la ubicación; la ubicación corrige o muda su dirección por su subrecurso, D-79; identificadores sin guiones, espacios ni puntos; un duplicado en carrera responde el código de su recurso; página de 1 a 200; español por omisión; §13 fila 14 resuelta) · 16/10/2026 (rutas, códigos y política de acceso de `iam` del paso 3; `/me`; `/users?companyId` y `managedBySuperadmin`; filtro de alcance construido) · 14/10/2026 (`location` nombra el país y la moneda por su código; escalones e hijos de una división como listas cortas; campos derivados del dueño) · 06/10/2026 (direcciones anidadas en su dueño, errores elegidos por `code`, fechas del Excel en ISO, rutas sin prefijo de contexto) · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.32 (§2.2 normalización, §2.4 identificadores, §2.5 separación por empresa, §2.6 fechas, §4.1 monedas, §4.6 direcciones, §4.7 tipos de documento, §5.3 colaboradores, §5.5 y §5.6 roles y permisos, §5.8 cómo se resuelven, §5.12 sesiones, §6.2 a §6.5, §9.1 a §9.2 compras, D-8, D-13, D-38, D-40, D-42, D-43, D-49, D-51, D-53, D-75, D-78, D-79, D-82, D-84 a D-91 y D-95 a D-135), reglas de arquitectura de la skill `backend-module`, [mapa de contextos](mapa-de-contextos-millennium-mobility.md) y el backend construido. Si este documento y el diccionario no coinciden, manda el diccionario.

La API es REST: expone **recursos** con sustantivos, los manipula con los **métodos de HTTP** según su semántica y responde con **códigos de estado** que dicen qué pasó. El único cliente hoy es el frontend del ERP, que se sirve desde el mismo origen.

**Cómo leerlo.**
- "diccionario §n" remite al diccionario de datos; "regla n" a las reglas de arquitectura de la skill `backend-module`; "§n" sin más, a este documento.
- Lo que cita el diccionario o una regla de arquitectura está decidido. Todo lo demás es propuesta de este documento hasta que se valide, y las propuestas con más peso están marcadas **(propuesta, §14.n)** y listadas en §14.
- Lo construido que no sigue estas convenciones está en §13. Se ajusta solo cuando se valide la propuesta de la que depende.

---

## 1. Base y versión

- Toda la API vive bajo **`/api/v1`**.
- **(propuesta, §14.14)** Mientras el único cliente sea el frontend propio, un cambio incompatible se hace en los dos repositorios a la vez y no abre una `v2`. La `v2` llega cuando exista un cliente externo (la aplicación del cliente, una integración).
- Cuerpos y respuestas en `application/json` con UTF-8, salvo archivos (§8).

---

## 2. Rutas

### 2.1 Forma de una ruta

```
/api/v1/{recurso}[/{id}[/{subrecurso}]]
```

| Regla | Correcto | Incorrecto |
|---|---|---|
| Los recursos son sustantivos en plural y en kebab-case | `/purchase-invoices`, `/geo-divisions` | `/getInvoices`, `/purchaseInvoice` |
| El identificador en la ruta es siempre el `public_id` (D-13) | `/vehicles/019352e1-…-7f3a` | `/vehicles/312`, `/countries/PE` |
| Se anida una colección bajo su dueño **solo si el dueño es del mismo contexto** | `/countries/{countryId}/geo-divisions` | `/companies/{companyId}/vehicles` |
| Un elemento se direcciona por su propio identificador, sin repetir al dueño | `/geo-divisions/{divisionId}` | `/countries/{countryId}/geo-divisions/{divisionId}` |
| Si el dueño es de otro contexto, va como filtro en la consulta y como campo en el cuerpo | `GET /vehicles?companyId=…` | — |

**Sin prefijo de contexto (decidido el 06/10/2026, §14.1).** El primer segmento es el recurso, no el contexto dueño: `/api/v1/vehicles` y `/api/v1/colors`, no `/api/v1/fleet/vehicles` ni `/api/v1/vehicle-catalog/colors`. Hay un solo despliegue y un solo cliente, y el mapa de §2.2 no tiene dos recursos con el mismo nombre. Que las URL del navegador sí lleven el contexto (`/vehicle-catalog/...`, mapa §8) es deliberado: allí ordenan las pantallas, y en la API nombran el recurso. La autenticación conserva `/api/v1/authentication/...`, que es la ruta del diccionario (§5.12: `POST /api/v1/authentication/sign-out`).

### 2.2 Recursos de cada contexto

Mapa de rutas objetivo, agrupado por contexto dueño; las rutas no llevan el contexto (§2.1). `/api/v1` se omite. Las que ya existen con otra forma están en §13. Lo marcado **(propuesta)** es un nombre que ninguna decisión fija y que figura en §14.

| Contexto | Rutas |
|---|---|
| `location` | `/currencies` · `/currencies/{id}` · `/currencies/{id}/status` · `/countries` · `/countries/{id}` · `/countries/{id}/geo-division-levels` · `/geo-division-levels/{id}` · `/countries/{id}/geo-divisions` · `/geo-divisions/{id}` · `/geo-divisions/{id}/status` · `/geo-divisions/{id}/children` · `/countries/{id}/tax-id-types` · `/tax-id-types/{id}` · `/tax-id-types/{id}/status` (D-44) |
| `iam` | `/authentication/sign-in` · `sign-out` · `me` · `sign-up` · `sign-up/status` · `sign-up/companies` · `change-password` · `/users` · `/users/{id}/password-reset` · `/access-grants` · `/users/{id}/role-assignments` · `/role-assignments/{id}/revocation` · `/users/{id}/suspensions` · `/suspensions/{id}/closure` · `/employees/{id}/termination` · `/employees/{id}/reactivation` · `/employees/{id}/account-unlinking` · `/roles` · `/roles/{id}` · `/roles/{id}/status` · `/roles/{id}/permissions` · `/permissions` · `/permission-matrix` (D-126) |
| `organization` | `/companies` · `/companies/{id}` · `/companies/{id}/status` · `/companies/{id}/fiscal-address` · `/companies/{id}/tax-id` · `/companies/{id}/branches` · `/branches/{id}` · `/branches/{id}/status` · `/branches/{id}/address` · `/companies/{id}/commercial-brands` · `/commercial-brands/{id}` · `/commercial-brands/{id}/status` · `/companies/{id}/locations` · `/locations/{id}` · `/locations/{id}/status` · `/locations/{id}/address` · `/companies/{id}/employees` · `/employees/{id}`. `GET /companies?countryId=…` responde si un país opera (D-3) |
| `vehiclecatalog` | `/vehicle-types` · `/vehicle-brands` · `/vehicle-brands/{id}/vehicle-models` · `/vehicle-models/{id}` · `/colors` · `/equipment-types`, cada uno con `/{id}` y `/{id}/status`. Además, desde el 29/10/2026, `GET /vehicle-models`, la lista de los modelos de todas las marcas, que solo lista y exporta: un modelo se sigue creando bajo su marca, con `POST /vehicle-brands/{id}/vehicle-models` |
| `fleet` | `/vehicles` · `/vehicles/{id}` · `/vehicles/{id}/plate` · `/vehicles/{id}/vin` · `/vehicles/{id}/annulment` **(propuesta, §14.20)** · `/vehicles/{id}/status-changes` · `/status-changes/{id}` · `/vehicles/{id}/mileage-readings` · `/mileage-readings/{id}` · `/vehicles/{id}/odometer-replacements` **(propuesta, §14.22)** · `/vehicles/{id}/documents` · `/vehicle-documents/{id}` · `/vehicle-documents/{id}/file` · `/vehicles/{id}/equipments` · `/vehicle-equipments/{id}` · `/vehicle-equipments/{id}/removal` · `/vehicles/{id}/keys` · `/vehicle-keys/{id}` · `/vehicles/{id}/fund-assignments` · `/fund-assignments/{id}/closure` · `/vehicles/{id}/tax-declarations` · `/vehicle-statuses` · `/vehicle-statuses/{id}` · `/vehicle-statuses/{id}/status` · `/status-change-reasons` (D-113) · `/status-change-reasons/{id}` · `/status-change-reasons/{id}/status` · `/document-types` · `/document-types/{id}` · `/document-types/{id}/status` · `/funds` · `/funds/{id}` · `/funds/{id}/status` · `/tax-declaration-imports` y `/fund-assignment-imports` **(propuesta, §14.18)** |
| `procurement` | `/suppliers` · `/suppliers/{id}/status` · `/suppliers/{id}/contacts` · `/supplier-contacts/{id}` · `/supplier-contacts/{id}/status` · `/suppliers/{id}/balance` **(propuesta, todavía sin ruta)** · `/purchase-order-types` · `/purchase-order-types/{id}/status` · `/purchase-order-templates` · `/purchase-order-templates/{id}/status` · `/purchase-order-templates/{id}/documents/{type}` · `/purchase-order-templates/{id}/documents/{documentId}/file` · `/purchase-orders` · `/purchase-orders/{id}/sending` · `/purchase-orders/{id}/annulment` · `/purchase-orders/{id}/fulfillment` · `/purchase-orders/{id}/pdf` · `/purchase-invoices` · `/purchase-invoices/{id}/annulment` · `/credit-notes` · `/supplier-payments` · `/supplier-payments/{id}/annulment`, cada colección con `/{id}`, salvo los contactos, que se direccionan por `/supplier-contacts/{id}` (§2.1). Facturas, notas y pagos siguen en diseño. El domicilio del proveedor viaja en el alta y en el `PUT` de su ficha (§3.3); sus contactos, desde el 29/10/2026, por sus propias rutas; el de la plantilla no viaja: se copia de la empresa al crearla (§3.4, D-134). No hay `/suppliers/{id}/address` ni `/purchase-order-templates/{id}/issuer-address`, ni `/{id}/signature` ni `/{id}/signature-removal` de la plantilla |
| Plataforma (`shared`) | `/change-log` · `/imports` · `/imports/{id}` · `/imports/{id}/error-report` |
| Plataforma (`documents`) | `/documents/{id}` · `/documents/{id}/file` (D-106, D-125, §8). Hasta D-125 las servía `shared`; la ruta no cambió, porque no lleva el contexto (§2.1) |
| Capa de lectura | `/reports/stock-export` · `/reports/day-gaps` · `/reports/down-payment-differences` · `/reports/vehicle-costs?vehicleId=…` **(propuesta, §14.19)** |

- Los recursos de empresa sin anidar (`/vehicles`, `/funds`, los de `procurement`) reciben `companyId` como filtro (§6.1).
- `location` no expone `/addresses`: una dirección se escribe y se lee solo a través de su dueño (§3.3).
- Toda tabla del diccionario con `is_active` expone `PATCH /{id}/status` sobre su ruta de elemento. Es la única forma de retirarla, porque no hay `DELETE` (§3.1). Los escalones territoriales no tienen `is_active` ni se retiran: se declaran en orden y se renombran (D-45).
- `/users` solo lista y exporta: no hay alta suelta de cuentas ni lectura de una cuenta por su identificador. Las cuentas nacen al dar acceso a un colaborador (`POST /access-grants`, §14.15) o por el registro público (D-32, D-38). `SYSTEM` nace en la migración y el primer superadministrador al arrancar (D-34), ninguno por la API.
- Una asignación de rol y un bloqueo no tienen ruta de elemento para leerlos: se listan desde su cuenta (`/users/{id}/role-assignments`, `/users/{id}/suspensions`) y solo tienen el subrecurso de su hecho (`revocation`, `closure`).
- Una cesión de una unidad a un fondo tampoco tiene ruta de elemento para leerla: se lista desde su unidad (`GET /vehicles/{id}/fund-assignments`, con `page` opcional) y su único subrecurso es el de su hecho, `POST /fund-assignments/{id}/closure` (construido el 26/10/2026, paso 10 parcial, D-133).
- Los contactos de un proveedor tienen sus propias rutas desde el 29/10/2026 (decisión del responsable de esa fecha; diccionario §9.1.1), como las sedes de una empresa: la colección cuelga de su proveedor y cada contacto se direcciona por su identificador. `GET /suppliers/{id}/contacts` lista, busca y pagina (§6.2); `POST /suppliers/{id}/contacts`, con `{name, position, phone, email}`, responde `201` con `Location: /api/v1/supplier-contacts/{id}`; `GET /supplier-contacts/{id}` lee uno, activo o retirado; `PUT /supplier-contacts/{id}`, con `{updatedAt, name, position, phone, email}`, lo corrige y conserva su proveedor y su estado (§3); `PATCH /supplier-contacts/{id}/status`, con `{isActive}`, lo retira o lo reactiva. Las cuatro escrituras responden el contacto: `{id, supplierId, name, position, phone, email, isActive, createdAt, updatedAt, allowedActions}`. El nombre es obligatorio, hasta 150; el puesto llega hasta 80, el teléfono hasta 30 y el correo hasta 255, recortado, en minúsculas y sin espacios dentro; hace falta el teléfono o el correo (`400 VALIDATION_ERROR`). Se leen con `compras.ver` donde está el proveedor y se escriben con `compras.editar_proveedores` en toda su empresa y sin marca (§9.2). Un proveedor inactivo no recibe un contacto nuevo ni reactiva uno (`422 SUPPLIER_INACTIVE`, D-94, D-108); pedir el estado que el contacto ya tiene no cambia nada. Cada escritura bloquea su proveedor, y la orden lo lee en modo compartido: un retiro y una orden que nombra ese contacto no se cruzan ([contextos/procurement.md](contextos/procurement.md), PRC-R24). `PUT /suppliers/{id}` ya no lleva `contacts`: si llegan, se ignoran y no retiran ninguno (cambio incompatible, §1). `POST /suppliers` todavía acepta los primeros, sin `id`, y la pantalla ya no los manda. `GET /suppliers/{id}` sigue trayendo los activos, como resumen `{id, name, position, phone, email}`, porque la orden los ofrece; la lista de proveedores los trae en `null`.
- La salida y el regreso de un colaborador cuelgan de `/employees/{id}`, pero los sirve `iam`, que los orquesta (§14.17; mapa de contextos, decisión 2). Como las rutas no llevan contexto (§2.1), el cambio de dueño no cambió la ruta. Lo mismo `POST /employees/{id}/account-unlinking`, que deshace el vínculo equivocado de un colaborador con una cuenta (D-82): la sirve `iam`, porque decide qué pierde la cuenta.
- `GET /companies/{id}/employees/metrics` ya no existe desde el 18/10/2026: ninguna pantalla lo leía.
- Los roles y el catálogo de permisos (construido el 19/10/2026, diccionario §5.5, §5.6, D-95 a D-98): `GET /roles` sigue la regla de §6.2 desde el 20/10/2026: con `page` responde el sobre paginado, y sin `page` el arreglo, como lo piden los selectores; nunca trae las matrices, y lista solo los roles que quien llama ve (D-98). Lo que busca, filtra y ordena está en la tabla de §6.2. Hasta entonces respondía siempre el arreglo completo, ordenado por código, y Roles y permisos lo filtraba en el navegador. `GET /roles/{id}` trae además `permissionCodes`, los permisos que el rol incluye ahora. Un rol propio se crea con `POST /roles` (`201` con `Location`), se renombra con `PUT /roles/{id}`, se deja de ofrecer o se ofrece otra vez con `PATCH /roles/{id}/status`, y su matriz se reemplaza entera con `PUT /roles/{id}/permissions`; las cuatro escrituras responden el mismo `RoleDetailResponse`, el rol con sus permisos vigentes. `GET /permissions`, entero o por `?module=`, solo lista el catálogo: ningún permiso se crea ni se edita por la API, los cargan las migraciones (D-95). Las matrices de los siete roles del sistema no se escriben (D-97).
- Con la segunda ronda de la auditoría (construida e integrada en `develop` el 20/10/2026; D-122 a D-124, D-126): `POST /roles` acepta además `copyPermissionsFromRoleId`, el rol del que se copian exactamente los permisos (D-124), y `GET /permission-matrix?companyId=` responde de una vez la matriz de la pestaña Matriz de permisos (`PermissionMatrixResponse`): `{permissions: [{code, module, category, description}], incompatibilities: [{permissionCode, incompatiblePermissionCode}], roles: [{id, code, name, scopeLevel, isSystem, isActive, companyId, editable, reserved, systemScopeOnly, permissionCodes}], grantableForGeneralRoles: [...], grantableForCompanyRoles: [...] | null}`, con los roles que la sesión lee por `seguridad.ver_roles` y, con `companyId`, los de esa empresa además de los del sistema y los generales (§6.2, D-126). Cada par de `incompatibilities` lleva sus dos códigos en orden alfabético. `editable` dice si la sesión puede cambiar lo que el rol incluye: un rol propio, para quien tiene `seguridad.gestionar_roles` con alcance sistema y no tiene ese rol vigente (D-70, D-123); `reserved`, si lo que incluye lo reserva al superadministrador (D-121); `systemScopeOnly`, si el rol es de alcance sistema, el único donde se marcan los ocho de accesos y roles (D-122). `grantableForGeneralRoles` y `grantableForCompanyRoles` son los códigos que la sesión puede marcar, contados como los cuenta el guardado (§5.5 regla 1, D-122): en un rol general, lo que tiene por asignaciones de alcance sistema; en uno de la empresa pedida, lo que actúa en toda ella, sin sede ni marca, o `null` sin `companyId`. No aplican los otros rechazos del guardado —los tres permisos que ningún rol propio lleva, los de alcance sistema y los pares—, que la pantalla deduce del catálogo, de `systemScopeOnly` y de `incompatibilities`. Se guarda por columna, con el mismo `PUT /roles/{id}/permissions` de siempre.
- Las escrituras de los roles las hace quien tiene, con alcance sistema, `seguridad.crear_roles`, `editar_roles`, `desactivar_roles` o `marcar_permisos_de_roles`, según la operación (D-130), y las lecturas quien tiene `seguridad.ver_roles` (D-122); hoy, el superadministrador escribe y los tres administradores leen.
- `role_permissions` es la excepción a los tres identificadores del diccionario (§2.4, §4.2): no tiene `public_id`, porque la API nunca direcciona una fila suya. Lo que un rol incluye se lee y se reemplaza entero por `/roles/{id}/permissions`, y la historia de cada marca —quién la puso y la quitó, y cuándo— queda en la tabla sin ruta de lectura (D-96).
- Una línea de factura no tiene operación propia de anulación: queda anulada al registrar una nota de crédito con motivo «anulación de unidad» (`POST /credit-notes`; diccionario §9.4, §9.5).
- Las cargas que actualizan la flota y su ruta de subida dependen de §14.18. `/imports` lista, consulta y descarga el archivo de errores de todas las cargas.

---

## 3. Métodos y respuestas

| Método | Para qué | Éxito | Cuerpo de la respuesta |
|---|---|---|---|
| `GET` colección | Listar, con filtros, orden y página | `200` | La lista (§6) |
| `GET` elemento | Leer uno | `200` | El recurso |
| `POST` colección | Crear | `201` con `Location: /api/v1/...` del recurso creado **(propuesta, §14.9)** | El recurso creado |
| `PUT` elemento | Reemplazar los campos editables | `200` | El recurso actualizado |
| `PATCH /{id}/status` | Activar o desactivar: `{ "isActive": false }` | `200` | El recurso actualizado |
| `POST /{id}/{colección-de-hechos}` | Registrar un hecho que queda en la historia (§3.2): `status-changes`, `mileage-readings`, `suspensions`, `role-assignments` | `201` con `Location` del hecho creado **(propuesta, §14.9)** | El hecho creado |
| `POST /{id}/{sustantivo-del-hecho}` | Cerrar, anular, retirar o enviar (§3.2): `annulment`, `revocation`, `termination`, `sending`, `closure`, `removal` | `200` **(propuesta, §14.9)** | El recurso afectado |
| `DELETE` | **No se usa** (regla 4.2) | — | — |

**Excepción de las cesiones** (26/10/2026). `POST /vehicles/{id}/fund-assignments` responde `201` con `Location: /api/v1/vehicles/{id}/fund-assignments`, la colección de la unidad, como `POST /vehicles/{id}/mileage-readings`, porque la cesión no tiene ruta de elemento para leerla (§2.2). El cuerpo es la cesión creada.

**El candado de `updatedAt`.** Una ficha que dos personas pueden editar a la vez lleva en su escritura el `updatedAt` con que se abrió, el que respondió su lectura: el proveedor (`PUT /suppliers/{id}`), la plantilla (`PATCH /purchase-order-templates/{id}` y su `/status`, §3.4) y, desde el 29/10/2026, el borrador de una orden (`PUT /purchase-orders/{id}`, V85) y el contacto de un proveedor (`PUT /supplier-contacts/{id}`), por decisión del responsable de esa fecha. Bajo el candado de la fila, un `updatedAt` que no es el de la fila, comparado al milisegundo, responde `422 <RECURSO>_OUTDATED` y no guarda nada; uno que falta, `400 VALIDATION_ERROR` (§7.2). En la orden se compara después del permiso y antes de `PURCHASEORDER_NOT_EDITABLE`, y lo mueven también el envío, el cumplimiento y la anulación. Escribir un contacto no mueve el `updatedAt` de su proveedor: cada contacto tiene el suyo. Los cambios de estado del proveedor, del tipo de orden y del contacto no lo llevan. La pantalla, ante ese código, vuelve a leer la ficha, la muestra como está y lo dice.

### 3.1 Por qué no hay `DELETE`

Nada se borra en este sistema: los catálogos se desactivan, los accesos se retiran, los documentos se reemplazan, los vehículos y comprobantes se anulan y los períodos se cierran (regla 4.2). Los repositorios del dominio no exponen `delete`. **(propuesta, §14.11)** Un `DELETE` sobre una ruta que no lo declara responde `405` con el código `METHOD_NOT_ALLOWED` (§7.2).

No hay ninguna ruta `DELETE`; lo comprueba `NoDeleteEndpointsTests`. El `DELETE` con que se quitaba el archivo de una plantilla desde el 23/10/2026 se retiró el 26/10/2026: quitarlo es `removedDocuments` en su `PATCH`, que vacía `file_path` y deja la fila del documento y la de `kept_files` (§3.4, diccionario §9.2).

### 3.2 Hechos de negocio

Lo que no es "editar un campo" se modela como un recurso, nunca como un verbo en la ruta:

| Situación | Cómo se expresa | Ejemplo |
|---|---|---|
| Un hecho nuevo que queda en la historia | `POST` a la colección de hechos del recurso | `POST /vehicles/{id}/status-changes`, `POST /vehicles/{id}/mileage-readings`, `POST /users/{id}/suspensions` |
| Cerrar, anular, retirar o enviar | `POST` a un subrecurso con el sustantivo del hecho, en singular | `POST /purchase-invoices/{id}/annulment`, `POST /role-assignments/{id}/revocation`, `POST /purchase-orders/{id}/sending`, `POST /employees/{id}/termination`, `POST /purchase-orders/{id}/fulfillment` (propuesta, §14.22) |
| Una corrección delicada que exige motivo (diccionario §13.1) | `PUT` a un subrecurso propio, con `reason` obligatorio y su propio permiso | `PUT /vehicles/{id}/plate` con `{ "plate": "ABC123", "reason": "Error de digitación" }` |
| Activar o desactivar | `PATCH /{id}/status` | `PATCH /currencies/{id}/status` |

**Correcciones delicadas.** El diccionario fija cuatro (§13.1): el VIN, la placa, el monto de una factura y una fecha ya reportada. El `PUT` del recurso no acepta esos campos ya registrados: la placa y el VIN se corrigen solo por `PUT /vehicles/{id}/plate` y `PUT /vehicles/{id}/vin`, con `reason` obligatorio. El monto de una factura y las fechas ya reportadas todavía no tienen ruta (§14.21).

**El documento tributario de una empresa** (D-85, construido el 18/10/2026). El tipo y el número no viajan en el `PUT /companies/{id}`: los corrige solo el superadministrador, en la misma fila, con `PUT /companies/{id}/tax-id` y `{ "taxIdTypeId": "<UUID>", "taxId": "20-131312955" }`. Responde `200` con la empresa. La corrección se verifica como el alta: un tipo del país de la empresa que aplique a empresas, la longitud y el dígito de control (`422`, §7.2) y que el número no lo tenga otra empresa del país (`409 COMPANY_TAX_ID_CONFLICT`). Un tipo inactivo se rechaza solo si es nuevo: el que la empresa ya tiene se conserva. El país no se corrige nunca: una empresa de otro país es otra empresa. Cualquier otro rol recibe `403`. No lleva `reason`: no es una de las correcciones de §13.1 del diccionario.

**Deshacer el vínculo de una cuenta** (D-82, construido el 18/10/2026). `POST /employees/{id}/account-unlinking`, sin cuerpo, para quien tiene `seguridad.desvincular_cuentas` con alcance sistema —solo el superadministrador, porque el permiso no se marca en un rol propio (D-117, D-122)— y está vinculado a un colaborador activo. Desde la segunda ronda de la auditoría pasa por la misma guardia que las demás operaciones de cuenta: quien no es superadministrador no desvincula una cuenta protegida (`403 ACCOUNT_MANAGED_BY_SUPERADMIN`). Funciona como una salida: retira los roles de la cuenta en la empresa del colaborador con motivo `retiro` y, si a la cuenta no le queda ningún colaborador activo, la cierra como la salida de su último colaborador (`baja`, `baja_cuenta` para sus demás roles y sesiones cerradas), con la protección del último superadministrador. Responde `200` con `{employeeId, userId, email, accountClosed}`; `accountClosed` dice si la cuenta quedó cerrada. Después el colaborador puede recibir su propio acceso con `POST /access-grants`. Los códigos están en §7.2.

**Quién y cuándo.** Las columnas de registro (`created_at`, `updated_at`, `created_by` y `updated_by`) nunca viajan desde el cliente. Las llena el sistema con la cuenta de la sesión, o con `SYSTEM` de forma explícita cuando no hay sesión (D-42). Los datos del hecho que el diccionario pide a quien registra sí viajan en el cuerpo:
- el momento de una lectura de kilometraje (`readAt`, que no es la fecha del registro, diccionario §8.5);
- el inicio de un bloqueo programado por adelantado (diccionario §5.4);
- quién instaló un equipo (`installedByEmployeeId`, diccionario §11.4).

Que el sistema llene también los campos de autor de un hecho que nombran a quien opera (`changed_by`, `granted_by`, `recorded_by`) es una propuesta (§14.26): D-42 no los cubre. `iam` ya la sigue: `granted_by` de una asignación y `closed_by` de un bloqueo salen de la cuenta de la sesión, y ningún cuerpo los trae. Desde el 18/10/2026 la asignación responde a sus autores por su `public_id` y por su correo, para mostrarlos: `grantedBy` con `grantedByEmail`, y `revokedBy` con `revokedByEmail`, `null` mientras sigue vigente; lo que venció lo firma `system@millennium.ky`. Se muestra el correo y no el nombre, porque el nombre es de `organization` (mapa de contextos §11, decisión 6).

Un hecho que se cierra lo hace en el momento de la petición: `POST /suspensions/{id}/closure` y `POST /role-assignments/{id}/revocation` no llevan cuerpo. Una fecha elegida por el cliente podría falsear quién tenía acceso (diccionario §5.4).

**La excepción es el cierre de una cesión** (26/10/2026). `POST /fund-assignments/{id}/closure` lleva `{ "endDate": "2026-07-01" }` y responde `200` con la cesión cerrada: el fin de una cesión es un hecho del negocio —el fin del crédito o de la garantía—, no el momento de la petición. La fecha es excluida y posterior al inicio (`[inicio, fin)`, diccionario §2.6, §12.2). El cierre bloquea primero la unidad y después vuelve a leer la cesión, así que dos cierres de la misma cesión no se pisan: el segundo responde `422 VEHICLEFUNDASSIGNMENT_ALREADY_CLOSED`.

**La autenticación es la excepción.** `sign-in`, `sign-out`, `sign-up` y `change-password` son operaciones, no recursos, y conservan sus nombres (diccionario §5.12).

### 3.3 Las direcciones

Las direcciones no tienen rutas propias: no existe `/addresses`. Viajan anidadas en el `POST` que crea a su dueño (empresa, sede, ubicación, proveedor) y embebidas en sus respuestas —también en la plantilla de orden de compra, que desde el 26/10/2026 la copia de su empresa y no la recibe (abajo)—, y quien puede escribir o leer al dueño es quien puede escribir o leer su dirección. Para distinguir corregir de mudarse (D-8), empresas, sedes y ubicaciones de terceros ofrecen un subrecurso de dirección (decidido el 06/10/2026, §14.24):
- `/companies/{id}/fiscal-address`;
- `/branches/{id}/address`;
- `/locations/{id}/address`.

El proveedor nace con el domicilio en el alta y lo corrige o lo muda en el `PUT` de la ficha (`address` + `addressChange = correction|move`), sin subrecurso. La plantilla no recibe domicilio: desde el 26/10/2026 copia el domicilio fiscal de su empresa al crearse, como el resto del emisor, y su alta y su `PATCH` ya no tienen `address` ni `addressChange`: si un cliente todavía los manda, se ignoran como cualquier propiedad que el pedido no tiene (§3.4, D-134). La respuesta conserva el domicilio del emisor, la copia que imprime el PDF.

**Lo que envía el cliente.** El mismo objeto en el alta del dueño y en su subrecurso:

```json
"fiscalAddress": {
  "geoDivisionId": "019352e1-5b1c-7a3e-9d4f-0a1b2c3d4e5f",
  "streetName": "Av. Javier Prado Este",
  "streetNumber": "4200",
  "unit": "Piso 3",
  "reference": "Frente al óvalo",
  "postalCode": null
}
```

- `streetName` es obligatorio, hasta 180 caracteres; `streetNumber` llega hasta 20, `unit` hasta 50 y `postalCode` hasta 12.
- `geoDivisionId` es obligatorio cuando el país de la dirección tiene sus divisiones cargadas, como Perú, y solo puede faltar en un país sin ellas (D-53). `postalCode` es obligatorio en los países que lo exigen (`requiresPostalCode`). Los demás son opcionales.
- El país no viaja: lo fija el dueño. La dirección de una empresa, una sede o una ubicación está en el país de la empresa (diccionario §4.6). La base lo exige en empresas y sedes con una llave foránea compuesta hacia `addresses (id, country_id)`; en una ubicación lo asegura la aplicación, porque `locations` no tiene columna de país. El proveedor tampoco manda país: hoy es solo local, y su domicilio está en el país de su empresa (diccionario §9.1, D-133). La Row Level Security de las direcciones se apoya en esta regla (diccionario §2.5, D-134).
- En el alta del dueño, `fiscalAddress: null` registra al dueño sin dirección. Una ubicación de terceros siempre la lleva (`address`); una sede propia nunca, porque usa la de su sede (`address: null`, D-79). El alta de una sede propia que trae `address` responde `400 VALIDATION_ERROR` sobre ese campo, y no crea ninguna dirección.

**Lo que devuelve el servidor.** La dirección embebida en el dueño, sin `id` ni fechas, porque no es un recurso:

```json
"fiscalAddress": {
  "country": "PE",
  "countryName": "Perú",
  "geoDivisionId": "019352e1-5b1c-7a3e-9d4f-0a1b2c3d4e5f",
  "geoDivisionName": "San Isidro",
  "streetName": "Av. Javier Prado Este",
  "streetNumber": "4200",
  "unit": "Piso 3",
  "reference": "Frente al óvalo",
  "postalCode": null
}
```

Un dueño sin dirección responde `"fiscalAddress": null`. El país viaja por su código (D-13) desde el 18/10/2026, también en la dirección embebida en sedes y ubicaciones (`address`), y `countryName` lo acompaña para mostrarlo (§13 fila 22). La lista de empresas también trae la dirección, y la resuelve en lote: una lectura a `location` por página, nunca una por fila (§8.1).

**Cómo cambia.** El `PUT` del dueño (`PUT /companies/{id}`) no lleva la dirección y no la modifica. La dirección se crea con su dueño o se cambia por su subrecurso:

| Qué pasó | Petición | Efecto | Respuesta |
|---|---|---|---|
| Se registra el dueño con su dirección | `POST /companies` con `fiscalAddress` | La dirección se crea en la misma transacción que la empresa, después de comprobar los conflictos de la empresa. Si algo falla, no queda ninguna de las dos | `201` con la empresa |
| Se escribió mal la dirección | `PUT /companies/{id}/fiscal-address` | Se corrige la misma fila. Si la empresa no tiene dirección, `422` con el código `COMPANY_FISCAL_ADDRESS_MISSING` | `200` con la empresa |
| El dueño se mudó, o registra su primera dirección | `POST /companies/{id}/fiscal-address` | Se crea una fila nueva y el dueño pasa a apuntar a ella; la anterior se conserva para los documentos ya emitidos | `200` con la empresa y su nueva dirección, sin `Location` |

La dirección no tiene ruta propia, así que la mudanza no responde `201`: lo creado no es direccionable y la empresa no es nueva.

**Las sedes y las ubicaciones siguen el mismo contrato** (D-79): `POST /companies/{companyId}/branches` y `POST /companies/{companyId}/locations` con `address` anidada; `PUT /branches/{id}` y `PUT /locations/{id}` sin dirección, y `PUT` o `POST /branches/{id}/address` y `/locations/{id}/address` para corregir o mudar. El `PUT /locations/{id}` recibe `{ name, locationType, branchId }`: si un cliente viejo todavía manda `address`, el servidor la ignora y la dirección no cambia. Corregir la dirección de una ubicación que no tiene responde `422 LOCATION_ADDRESS_MISSING`, y corregir o mudar la de una sede propia, `422 LOCATION_ADDRESS_NOT_ALLOWED`: usa la de su sede. Una ubicación tampoco pasa de sede propia a ubicación de terceros ni al revés, porque es otro lugar: el `PUT` que lo intenta responde `422 LOCATION_OWNERSHIP_CHANGE_NOT_ALLOWED`, y se desactiva la ubicación y se registra una nueva.

**Los fallos de la dirección** salen por el endpoint del dueño que la lleva —el alta de la empresa, la sede o la ubicación, y su subrecurso de dirección— con un código propio de cada regla. Todos responden `422`, y el campo es el objeto de dirección anidado (`fiscalAddress` o `address`):

| `code` | Cuándo |
|---|---|
| `ADDRESS_DIVISION_REQUIRED` | Falta la división y el país de la dirección tiene sus divisiones cargadas (D-53) |
| `ADDRESS_POSTAL_CODE_REQUIRED` | Falta el código postal y el país lo exige |
| `ADDRESS_DIVISION_OTHER_COUNTRY` | La división es de otro país que la dirección |
| `ADDRESS_DIVISION_NOT_DEEPEST_LEVEL` | La división no es del último escalón que declara el país: una provincia y no un distrito |
| `ADDRESS_DIVISION_INACTIVE` | La división está inactiva y es nueva para la dirección; la que la dirección ya tenía se conserva |

Siguen además `GEODIVISION_NOT_FOUND` (`404`) si la división no existe y `VALIDATION_ERROR` (`400`) para el formato del cuerpo: una calle vacía, una longitud.

### 3.4 Plantillas de orden de compra

Construido el 23/10/2026 (diccionario §9.2, D-132; [contextos/procurement.md](contextos/procurement.md)). El `GET` de la colección, de la ficha y del archivo pide `compras.ver`. Desde el 26/10/2026 cada escritura pide su propio permiso (diccionario §5.6, D-130): crear pide `compras.crear_plantillas`; corregir, y cargar o quitar un archivo, `compras.editar_plantillas`; activar o desactivar, `compras.desactivar_plantillas`. Hasta entonces las cuatro pedían `compras.gestionar_plantillas`, que ya no abre ninguna ruta. Desde la misma fecha el servicio de lectura pide también solo `compras.ver` donde actúa la plantilla: escribir exige leer, así que corregir una plantilla que quien llama no lee responde `404 PURCHASEORDERTEMPLATE_NOT_FOUND`, y la respuesta de cada escritura es la plantilla guardada, sin volver a filtrar (diccionario §5.6, D-134). Hasta entonces aceptaba además `compras.gestionar_ordenes` y `compras.gestionar_plantillas`, y esa unión dejaba leer las órdenes y el PDF de otra empresa. Una plantilla general la escribe solo una asignación de toda la empresa y sin marca; una con marca, también una de esa marca (diccionario §2.5).

**El alta es un solo `POST` multipart.** La parte `template` es el JSON de la ficha, con tipo `application/json`. Las partes `logo` y `signature` son opcionales: PNG, JPEG, WebP o PDF, de a lo sumo 5 MB. Un PDF se guarda y se ve en la ficha; la orden impresa solo dibuja una imagen. Falta de las dos, o de una, deja la plantilla sin ese archivo. Responde `201` con `Location` y la plantilla. Un archivo que no cumple el tipo, el peso o las dimensiones rechaza el alta entero: no queda la plantilla (`422 PURCHASEORDERTEMPLATE_DOCUMENT_TYPE`, `PURCHASEORDERTEMPLATE_DOCUMENT_SIZE` o `PURCHASEORDERTEMPLATE_DOCUMENT_DIMENSIONS`). El tipo se lee de los bytes, no del nombre ni del tipo que declara el navegador. Una imagen de más de 4096 píxeles por lado o de más de 16 megapíxeles se rechaza por lo que declara su encabezado, sin decodificarla. Un archivo de más de 10 MB, o un pedido de más de 21 MB, no llega al servicio: el servidor lo corta con `413 CONTENT_TOO_LARGE` (§7.2).

**El emisor no viaja** (26/10/2026, D-134). El alta copia de la empresa, en el servidor, la razón social, el tipo y el número de documento, el teléfono y el domicilio fiscal del emisor (diccionario §9.2). El alta y el `PATCH` ya no tienen `issuerLegalName`, `issuerTaxIdTypeId`, `issuerTaxId`, `issuerPhone`, `address` ni `addressChange`: un cliente que todavía los manda no falla, se ignoran como cualquier propiedad que el pedido no tiene, sin validarlos ni contarlos como cambio. La respuesta conserva el emisor, porque es la copia que imprime el PDF.

**La corrección es un `PATCH` de verdad.** Hay dos contenidos sobre la misma ruta:

| Contenido | Qué manda | Qué cambia |
|---|---|---|
| `application/json` | Solo los campos que cambian, y siempre `updatedAt`. `removedDocuments`, si viene, nombra los tipos de archivo que se quitan: `["LOGO"]` | Un campo ausente se queda. Uno presente en `null` vacía un campo que admite vacío. `name`, `series` y `purchaseOrderTypeId`, si vienen, no pueden ser `null`: `400 VALIDATION_ERROR`. `isActive` no es del `PATCH`: si viene, con cualquier valor, responde `400 VALIDATION_ERROR` y no cambia nada. Un campo del emisor se ignora (arriba) |
| `multipart/form-data` | La parte `template` con ese mismo JSON —también `removedDocuments`—, y `logo` o `signature` si se reemplazan | Una parte de archivo ausente no quita el archivo. Una presente lo reemplaza |

`updatedAt` es el candado: si no es el de la fila, al milisegundo, responde `422 PURCHASEORDERTEMPLATE_OUTDATED`.

**El estado va aparte** (26/10/2026). Activar o desactivar es `PATCH /purchase-order-templates/{id}/status` con `{ "isActive", "updatedAt" }`, con `compras.desactivar_plantillas`. Un `updatedAt` que no es el de la plantilla responde `422 PURCHASEORDERTEMPLATE_OUTDATED`, y un cuerpo sin uno de los dos, `400 VALIDATION_ERROR`. Responde `200` con la plantilla. Entre el 23/10/2026 y el 26/10/2026 el estado viajaba en el `PATCH /{id}`.

**Los archivos son documentos de la plantilla**, no columnas. La respuesta trae `documents`, y solo los que tienen archivo: cada uno con `id`, `type` (`LOGO`, `SIGNATURE`), `typeName`, `fileName`, `mimeType` y `sizeBytes`.

| Acción | Ruta | Respuesta |
|---|---|---|
| Reemplazar uno, también uno que el alta no mandó | `PUT /{id}/documents/{type}`, parte `file` | `200` con la plantilla. El tipo es el código, `LOGO` o `SIGNATURE`. Uno que el catálogo no tiene, `404 PURCHASEORDERTEMPLATEDOCUMENTTYPE_NOT_FOUND`. Se conserva para los tipos que lleguen después |
| Quitar la referencia | `removedDocuments` en el `PATCH` (arriba), desde el 26/10/2026 | `200` con la plantilla. Quita esos archivos en la misma transacción que los campos y las subidas, después de comparar `updatedAt` bajo el candado. Un tipo que el catálogo no tiene responde `404 PURCHASEORDERTEMPLATEDOCUMENTTYPE_NOT_FOUND`; quitar y subir el mismo tipo en un pedido, `400 VALIDATION_ERROR`; quitar un archivo que la plantilla no tiene no cambia nada. Vacía la llave; la fila y los bytes se quedan. Es la única forma de quitar un archivo: no hay `DELETE` (§3.1) |
| Ver el archivo | `GET /{id}/documents/{documentId}/file` | Los bytes, `Content-Disposition: inline`. Lo abre quien puede leer la plantilla |

El PDF de la orden dibuja `LOGO` y `SIGNATURE`. Un tipo que el generador no conoce se guarda y se devuelve, y no se imprime.

---

## 4. El cuerpo JSON

### 4.1 Nombres

| Regla | Ejemplo |
|---|---|
| Los campos van en camelCase y repiten el nombre de la columna del diccionario | `iso_alpha_2` → `isoAlpha2`, `requires_postal_code` → `requiresPostalCode` |
| Los booleanos conservan su prefijo | `is_system` → `isSystem`, `has_factory_screen` → `hasFactoryScreen` |
| Los sufijos de fecha se conservan | `created_at` → `createdAt`, `invoice_date` → `invoiceDate`, `campaign_month` → `campaignMonth` |
| Los valores de una lista cerrada viajan tal como los fija el diccionario | `"scopeLevel": "COMPANY"`, `"reason": "vacaciones"`, `"revocationReason": "baja_colaborador"`, `"locationType": "sede_propia"` |
| **(propuesta, §14.13)** Los campos opcionales sin valor viajan como `null`, no se omiten | `"taxName": null` |

El tipo de una ubicación viaja en minúsculas desde el 18/10/2026, como lo escribe el diccionario (§6.5): `sede_propia`, `patio_arrendado`, `tienda_dealer`, `taller_tercero` y `otro`. Antes viajaba como el nombre de la constante (`SEDE_PROPIA`). Al leer, el servidor acepta las dos formas, sin distinguir mayúsculas y con espacios alrededor, en el cuerpo (`POST /companies/{id}/locations`, `PUT /locations/{id}`) y en el filtro `?locationType=`; responde siempre la forma en minúsculas.

**La excepción a `null`** (construida el 19/10/2026, D-105). `documentNumber`, `phone` y `photoPath` del colaborador se omiten de la respuesta —no viajan en `null`— cuando la cuenta no tiene `configuracion.ver_datos_personales_colaboradores` donde el colaborador trabaja (§9.2): un campo que la cuenta no puede ver no viaja (diccionario §2.5). Y un teléfono o una foto legítimamente vacíos también se omiten, con o sin el permiso, porque la fila no distingue "no lo ve" de "no tiene"; la pantalla muestra un guion en los dos casos. Es la única excepción a la regla de enviar `null` (§14.13), y vale solo para esos tres campos.

### 4.2 Identificadores (diccionario §2.4, D-13)

| Qué se referencia | Cómo viaja | Ejemplo |
|---|---|---|
| El propio recurso | `id`, con el `public_id` **(el nombre `id`: propuesta, §14.8)** | `"id": "019352e1-…"` |
| Una entidad del negocio (empresa, sede, proveedor, vehículo, división) | `<entidad>Id` con su `public_id` | `"companyId": "…"`, `"parentId": "…"` |
| Un catálogo de estándar externo | Su código | `"currency": "PEN"`, `"country": "PE"` |

- El `id` interno no aparece nunca en una respuesta, una ruta ni un archivo exportado, tampoco en las referencias (diccionario §2.4).
- La empresa sigue esta tabla desde el 18/10/2026: recibe y responde `country: "PE"` y `functionalCurrency: "PEN"`, y el tipo de documento tributario, que no es de una norma externa sino de cada país (diccionario §4.7), como `taxIdTypeId` con su `public_id`. Un país mal formado responde `400 VALIDATION_ERROR` y uno inexistente `404 COUNTRY_NOT_FOUND`; una moneda inexistente, `404 CURRENCY_NOT_FOUND`. Se aceptan en minúsculas y con espacios alrededor. Su propio identificador sigue llamándose `publicId` (§14.8) y su filtro de estado `active` (§14.10).
- La orden de compra la sigue desde el 29/10/2026 (decisión del responsable de esa fecha): `POST /purchase-orders` y `PUT /purchase-orders/{id}` reciben `"currency": "PEN"` en lugar de `currencyId`, y la orden responde `currency` en su detalle, en su lista y en cada escritura, sin `currencyId` ni `currencyCode`. Se acepta en minúsculas y con espacios alrededor y se responde en mayúsculas; un texto que no son tres letras responde `400 VALIDATION_ERROR`, y un código que ninguna moneda tiene, `404 CURRENCY_NOT_FOUND`. Cada `unitPrice` de sus líneas viaja con ese mismo código (§5.2, §7.2). Fue un cambio incompatible, hecho en los dos repositorios a la vez (§1).
- Una respuesta puede acompañar la referencia con lo que la pantalla necesita mostrar, para no obligar a otra petición: `"companyId"` con `"companyName"`, `"parentId"` con `"parentName"`.
- Una respuesta también puede traer un dato derivado que la pantalla necesita para decidir, calculado por el dueño del recurso y nunca guardado. Hoy hay cuatro (D-115):
  - `isoCodesLocked` del país dice si sus códigos ISO todavía se pueden corregir (diccionario §4.2, D-47);
  - `codeLocked` del tipo de equipamiento dice si su código todavía se puede corregir (diccionario §7.5, D-74);
  - `vehicleTypeLocked` del modelo dice si su tipo de vehículo todavía se puede corregir: es verdadero desde que el modelo tiene su primera unidad (diccionario §7.3, D-107);
  - `allowedActions` de la unidad dice qué puede hacer la sesión sobre esa unidad —editarla, los caminos que puede recorrer, registrar una lectura o una sustitución de odómetro, cederla a un fondo (`registerFundAssignment`)—, calculado con las mismas reglas que cada operación ([contextos/fleet.md](contextos/fleet.md), FLT-R36). `registerFundAssignment` es verdadero si `fondos.registrar_cesion` actúa en toda la empresa o en la marca de la unidad (D-134), `fondos.ver` lee su empresa —el diálogo lista sus fondos— y la unidad no tiene una cesión abierta ni una que termine después de hoy, en la zona del país de su empresa (D-40). Solo la ficha y las respuestas de las escrituras de la unidad lo traen; las filas de la lista llevan `allowedActions` nulo.

  La segunda ronda de la auditoría suma estos, construidos e integrados en `develop` el 20/10/2026 (D-119 a D-122):
  - `reserved` del rol (`RoleResponse` y `RoleDetailResponse`) y de la asignación (`/role-assignments` y los roles de una cuenta) dice si el rol es reservado por lo que incluye ahora (diccionario §5.5, D-121): lo otorga solo un superadministrador y hace protegida la cuenta que lo tiene;
  - `limitableToBrand` del rol dice si una asignación suya puede limitarse a una marca: un rol de alcance sistema o reservado, nunca (D-59, D-121);
  - `allowedActions` del colaborador dice qué puede hacer la sesión en esa fila, con las mismas reglas que cada operación, también dónde actúa cada permiso y las reglas sobre la cuenta de destino (D-122). Sus valores, siempre en este orden: `edit` (editarlo), `terminate` y `reactivate` (registrar su salida o su regreso), `grantAccess` (darle acceso), `manageRoles` (sus roles), `suspend` (bloquear su cuenta), `resetPassword` (restablecer su contraseña) y `unlink` (desvincularla). Las cuatro de la cuenta exigen que el colaborador esté activo; a uno que salió se le ofrece a lo sumo `edit`, para corregir sus datos (ORG-R21), y `reactivate`. Lo calcula `EmployeeAllowedActions`, de `organization`, que pregunta a `iam` qué cuentas son protegidas por el puerto `ProtectedAccounts`, de `shared`;
  - en la sesión (`/me`, §9.1), `systemPermissions`, los permisos que la cuenta tiene por asignaciones de alcance sistema, y `actsOnReserved`, si puede actuar sobre lo reservado —hoy, si es superadministrador—, para que la pantalla aplique del lado de quien opera las reglas sobre el destino.

  El cierre de la auditoría del 25/10/2026 suma estos (26/10/2026, D-133, D-134):
  - `allowedActions` del fondo, en este orden: `edit` y después `deactivate` o `activate`, cada uno si su permiso (`fondos.editar_fondos`, `fondos.desactivar_fondos`) actúa en toda la empresa y sin marca: un fondo es de todas las marcas de su empresa. No repite las reglas que responden `422`, como la empresa inactiva;
  - `allowedActions` de la cesión: `close` mientras está abierta, si `fondos.registrar_cesion` actúa en toda la empresa o en la marca de su unidad;
  - `codeLocked` del fondo dice si su código todavía se puede corregir: es verdadero desde que una cesión lo usa (diccionario §2.4, §12.1);
  - `codeLocked` del tipo de orden dice lo mismo de su código: es verdadero desde que una plantilla lo usa (diccionario §9.2);
  - `valuesWithheld` de una fila de la bitácora dice que `oldValue` y `newValue` se omitieron porque quien llama no ve ese dato personal; con `false`, un valor nulo es un campo vacío (diccionario §13.1);
  - `allowedActions` del proveedor, en el mismo orden que el fondo: `edit` y `deactivate` o `activate`, cada uno si su permiso actúa en toda la empresa y sin marca, y solo para quien lee el proveedor con `compras.ver`;
  - `allowedActions` del contacto de un proveedor (registrado el 29/10/2026), con la misma forma: `edit` y después `deactivate` o `activate` según su estado, los tres con `compras.editar_proveedores` en toda la empresa del proveedor y sin marca, para quien lo lee con `compras.ver`. No repite los `422`: a un contacto retirado de un proveedor inactivo se le ofrece `activate`, y la API responde `SUPPLIER_INACTIVE`;
  - `allowedActions` del tipo de orden y de la plantilla, con la misma forma: `edit` con `compras.editar_plantillas` y `deactivate` o `activate` con `compras.desactivar_plantillas`; una plantilla general, como un tipo, con un permiso de toda la empresa y sin marca; una con marca, también con uno de esa marca. La plantilla agrega `issueOrder` cuando el alta de una orden la aceptaría: la plantilla, su empresa, su marca —si la tiene— y su tipo están activos, quien llama la lee con `compras.ver` y `compras.crear_ordenes` actúa donde está (registrado el 27/10/2026);
  - `allowedActions` de la orden, según su estado: en un borrador, `edit`, `send`, `annul` y `downloadPdf`; en una enviada, `fulfill`, `annul` y `downloadPdf`; en una cumplida o anulada, `downloadPdf`. Fuera del borrador, `downloadPdf` sale solo si la orden tiene su hoja guardada. Cada acción se ofrece por su propio permiso: `edit` con `compras.editar_ordenes`, `send` con `compras.enviar_ordenes`, `fulfill` con `compras.cumplir_ordenes`, `annul` con `compras.anular_ordenes` y `downloadPdf` con `compras.ver`. Cada uno exige además leer la orden y sigue las mismas reglas que su operación ([contextos/procurement.md](contextos/procurement.md)). Desde el 29/10/2026 llegan siempre en el orden `edit`, `send`, `fulfill`, `annul` y `downloadPdf`; antes salían de un conjunto, en un orden que podía cambiar de un arranque a otro.

  `category` del permiso no es uno de ellos: es una columna de `permissions` (diccionario §5.6, D-118), y viaja como la escribe el diccionario: `ver`, `crear`, `editar`, `gestionar`, `desactivar_anular` o `acciones_especiales`.

  La pantalla los usa para bloquear un campo u ofrecer solo lo que se puede; la API rechaza igual lo que no se puede. `currentMileage` de la unidad no es un dato derivado de esta lista: es el último kilometraje, una de las dos excepciones que el diccionario guarda además en la unidad (diccionario §2.2, §11.1). Hasta que llegue su columna, con el paso 9, se calcula de la lectura más reciente cada vez que se lee.

---

## 5. Fechas, montos y números

### 5.1 Fechas (diccionario §2.6, D-40)

| Dato | Tipo Java | JSON | Ejemplo |
|---|---|---|---|
| Momento (`_at`) | `Instant` | ISO 8601 en UTC, con `Z` | `"2026-09-27T14:05:00Z"` |
| Fecha de calendario (`_date`) | `LocalDate` | `AAAA-MM-DD` | `"2026-09-15"` |
| Mes (`_month`) | `LocalDate`, día 1 | `AAAA-MM-DD` con día 1 | `"2026-09-01"` |
| Período (`period`) | Rango | Dos campos; el fin es excluido y `null` significa abierto. **(propuesta, §14.3)** Se llaman `periodStart` y `periodEnd` | `"periodStart": "2026-07-04", "periodEnd": null` |
| Zona horaria | `String` | Nombre IANA | `"America/Lima"` |

Nunca `dd/mm/aaaa` en la API: ese formato existe solo en la pantalla (D-40).

### 5.2 Montos (diccionario §2.2, reglas 4.4 y 4.5)

Todo monto viaja con su moneda (regla 4.4) y nunca es `null`: el diccionario los define `NOT NULL DEFAULT 0` (regla 4.5).

**(propuesta, §14.2)** El monto viaja en un objeto:

```json
"invoicedAmount": { "amount": "5900.00", "currency": "PEN" }
```

- `amount` es texto decimal con exactamente los decimales de la moneda: `"5900.00"` en soles, `"5900"` en pesos colombianos. Como texto no pierde precisión en el navegador.
- Los tipos de cambio (`NUMERIC(12,6)`) y las tasas (`NUMERIC(5,4)`) también viajan como texto decimal: `"exchangeRate": "3.742000"`, `"taxRate": "0.1800"`.

### 5.3 Normalización

El servidor normaliza lo que recibe, así que el cliente puede enviar cualquier forma (diccionario §2.2). La respuesta devuelve el valor ya normalizado.

- Los códigos de norma externa, recortados y en mayúsculas: `" pen "` → `PEN`, en el código de una moneda nueva y en `defaultCurrency`, `isoAlpha2` e `isoAlpha3` del país.
- Los identificadores de texto, con `TRIM` + `UPPER` y sin guiones, espacios ni puntos en ninguna posición: el código de una división (`" t-2 "` → `T2`), el número de documento de una empresa y el de un colaborador (`" 74-839.201 "` → `74839201`), y las placas cuando existan. También caen los espacios y los guiones de Unicode que llegan al pegar un texto: el espacio duro, el guion no separable, la raya, el signo menos y los caracteres invisibles. Dos formas que quedan iguales son el mismo valor y chocan con el `409` de su recurso; un código de división hecho solo de separadores responde `400 VALIDATION_ERROR`. La migración V39 llevó a esta forma los que ya estaban guardados.
- Los correos, recortados y en minúsculas.
- Un valor de lista cerrada se acepta en mayúsculas y con espacios alrededor, y se guarda como lo fija el diccionario: `" Persona "` → `persona` en `appliesTo`.

---

## 6. Colecciones

### 6.1 Filtros

| Regla | Ejemplo |
|---|---|
| **(propuesta, §14.10)** Los filtros son parámetros de consulta en camelCase, con el nombre del campo que filtran | `?isActive=true`, `?level=2`, `?companyId=…` |
| Un filtro ausente no filtra | Sin `isActive`, vienen activas e inactivas |
| La búsqueda libre usa `q` | `?q=lima` |
| **(propuesta, §14.10)** Un recurso de empresa exige `companyId` cuando la cuenta tiene acceso a más de una empresa | `GET /vehicles?companyId=…` |
| **(propuesta, §14.10)** Si el dueño de la colección no existe, responde `404`, no una lista vacía | `GET /countries/{id}/geo-divisions` con un país inexistente |

### 6.2 Búsqueda, orden y página

Desde el 06/10/2026, toda lista se busca, se filtra, se ordena y se pagina en el servidor, y **en un solo endpoint**: el mismo `GET` de la colección recibe `q`, los filtros (§6.1), `sort`, `page` y `size`. También los catálogos chicos: una tabla nunca trae la colección entera para paginarla en el navegador.

```
GET /api/v1/countries/{id}/geo-divisions?q=lima&level=2&isActive=true&page=0&size=50&sort=name,asc
```

```json
{
  "items": [ … ],
  "page": 0,
  "size": 50,
  "totalItems": 1874,
  "totalPages": 38
}
```

| Parámetro | Regla |
|---|---|
| `q` | Texto que contiene alguno de los campos de búsqueda de la lista, sin distinguir mayúsculas. `%` y `_` se buscan tal cual. Todavía distingue tildes: "peru" no encuentra "Perú" **(pendiente)**: plegarlas antes de la primera lista que se busque en el servidor mientras se escribe (proveedores, paso 7), como ya lo hace el buscador del frontend con las listas cargadas. Esa lista existe desde el 23/10/2026 y al 29/10/2026 sigue sin plegarlas: vencido, espera la decisión del responsable ([orden de implementación](orden-de-implementacion-millennium-mobility.md), pendientes transversales) |
| `page` | Índice de la página, desde 0. **Con `page`, la respuesta es el sobre de arriba; sin `page`, es el arreglo de todo lo que coincide**, en el mismo orden y con la misma búsqueda y filtros, como lo pide un select |
| `size` | 50 por omisión, de 1 a 200. Un valor fuera de ese rango se lleva a su borde: más de 200 responde una página de 200, y 0 o un negativo, una página de una fila, nunca todas las filas |
| `sort` | `campo` o `campo,desc`, con el nombre del campo JSON. Cada lista declara por qué campos ordena; con otro campo usa su orden propio. Los empates se ordenan por el `id` interno, para que una fila no salte de página |

Lo que busca y ordena cada lista construida:

| Lista | `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|---|
| `GET /currencies` | `isoCode`, `name`, `symbol` | `isActive` | `isoCode`, `name`, `symbol`, `decimalPlaces`, `isActive` | `isoCode` |
| `GET /countries` | `name`, `isoAlpha2`, `isoAlpha3`, `callingCode` | — | `name`, `isoAlpha2`, `isoAlpha3`, `callingCode`, `defaultTimezone`, `taxRate`, `defaultCurrency` | `name` |
| `GET /countries/{id}/geo-divisions` | `code`, `name` | `level`, `isActive` | `code`, `name`, `level`, `isActive` | `level` y `code` |
| `GET /countries/{id}/tax-id-types` | `code`, `name` | `isActive`, `appliesTo` | `code`, `name`, `appliesTo`, `length`, `hasCheckDigit`, `isActive` | `code` |
| `GET /companies` | `legalName`, `tradeName`, `taxId` | `active`, `countryId` | `legalName`, `tradeName`, `taxId`, `phone`, `isActive` | `tradeName` |
| `GET /companies/{id}/locations` | `code`, `name` | `isActive`, `locationType` | `code`, `name`, `locationType`, `isActive`, `createdAt`, `updatedAt` | `name` |
| `GET /companies/{id}/employees` | `firstName`, `lastNamePaternal`, `lastNameMaternal`, `documentNumber`, `position` | `branchId`, `position`, `isActive` | `fullName` (nombre, apellido paterno y apellido materno, como lo muestra la columna), `firstName`, `lastNamePaternal`, `documentNumber`, `hireDate`, `position`, `createdAt`, `updatedAt` | `lastNamePaternal` |
| `GET /users` | `email` | `companyId` (ver abajo) | `email`, `lastLoginAt`, `mustChangePassword` | `email` |
| `GET /roles` | `code`, `name`, `description` | `scopeLevel`, `isSystem`, `isActive`, `companyId` (los roles de esa empresa) y `general`: `true`, los roles para todas las empresas, sin empresa; `false`, los de alguna empresa. `companyId` y `general` juntos responden `400 VALIDATION_ERROR`, como un `scopeLevel` fuera de `SYSTEM`, `COMPANY` y `BRANCH` | `code`, `name`, `scopeLevel`, `isSystem` (también `system`, el nombre del campo en el JSON), `isActive` | `name` |
| `GET /vehicle-types` | `code`, `name`, `commercialName` | `isActive` | `code`, `name`, `commercialName`, `isActive` | `code` |
| `GET /vehicle-brands` | `name` | `isActive` | `name`, `isActive` | `name` |
| `GET /vehicle-brands/{id}/vehicle-models` | `name` | `isActive`, `vehicleTypeId` (uno que no existe no coincide con ninguno) | `name`, `engineCc`, `fuelType`, `hasFactoryScreen`, `isActive` | `name` |
| `GET /vehicle-models` (29/10/2026) | `name` del modelo | `vehicleBrandId`, `vehicleTypeId` e `isActive`; una marca o un tipo que no existen no coinciden con ningún modelo: son filtros, no el dueño de la lista | `vehicleBrandName` (el nombre de la marca y después el del modelo, los dos en la dirección pedida), `name`, `engineCc`, `fuelType`, `hasFactoryScreen`, `isActive` | `vehicleBrandName` |
| `GET /colors` | `name` | `isActive` | `name`, `isActive` | `name` |
| `GET /equipment-types` | `code`, `name` | `isActive`, `vehicleTypeId` (los que aplican hoy a ese tipo: solo cuenta la aplicabilidad activa; uno que no existe no coincide con ninguno) | `code`, `name`, `blocksAvailability`, `requiresSerial`, `isActive` | `code` |
| `GET /vehicles` | `vin`, `plate`, `engineNumber` | `companyId`, `commercialBrandId` (con `companyId`), `currentStatusId`, `currentLocationId`, `modelId`, `overdue` (solo las unidades cuya tolerancia del último cambio ya pasó) | `vin`, `plate`, `engineNumber`, `modelYear`, `createdAt`, `updatedAt` | `vin` |
| `GET /vehicles/{id}/status-changes` | — (descarta `q`) | — | — (descarta `sort`) | De la más nueva a la más antigua |
| `GET /vehicles/{id}/mileage-readings` | — (descarta `q`) | — | — (descarta `sort`) | De la más nueva a la más antigua, por el momento de la lectura |
| `GET /funds` | `code`, `name` (texto literal: `%` y `_` no son comodines) | `companyId`, `isActive` | `code`, `name`, `fundType`, `createdAt`, `updatedAt` | `name` |
| `GET /vehicles/{id}/fund-assignments` | — (descarta `q`) | — | — (descarta `sort`) | La abierta primero, después de la de inicio más nuevo a la más vieja |
| `GET /suppliers` | `legalName`, `tradeName`, `taxId` | `companyId`, `isActive` | `legalName`, `tradeName`, `taxId`, `createdAt`, `updatedAt` | `legalName` |
| `GET /suppliers/{id}/contacts` | `name`, `position`, `email` | `isActive` (sin él, los activos y los retirados) | `name`, `position`, `isActive` | `name` |
| `GET /purchase-order-types` | `code`, `name` | `companyId`, `isActive` | `code`, `name` | `name` |
| `GET /purchase-order-templates` | `name` | `companyId`, `isActive`, `purchaseOrderTypeId` | `name`, `series`, `createdAt`, `updatedAt` | `name` |
| `GET /purchase-orders` | El número como se muestra (`OC-LBT-2026-0001`) y la razón social del proveedor | `companyId`, `status` (`BORRADOR`, `ENVIADA`, `CUMPLIDA` o `ANULADA`, en cualquier mayúscula) | `orderNumber` (por serie, año y correlativo), `issueDate`, `status`, `createdAt` | `createdAt` |

`GET /roles` filtra como lo hacía Roles y permisos en el navegador, ahora en el servidor (20/10/2026): con `null`, o sin el parámetro, ese filtro no filtra, y el alcance de quien llama se aplica como antes (D-98). `/vehicles` no ordena por las columnas que nombran una referencia —la empresa, la marca comercial, el modelo, el estado—. El historial y las lecturas de una unidad se paginan, pero no se buscan ni se ordenan: los pide la ficha de la unidad, siempre en el mismo orden.

`GET /vehicle-models` lista los modelos de todas las marcas, como la pantalla Marcas y modelos los muestra desde el 29/10/2026 (decisión del responsable de esa fecha), con `catalogos.ver`, como la lista de una marca. Puede ordenar por el nombre de la marca porque la marca es del mismo contexto: no es una de las columnas de otro contexto que no se ordenan (abajo). Desde la misma fecha todo modelo viaja con `vehicleBrandName` junto a `vehicleBrandId`: en la lista de una marca, en la de todas, en su alta y en la lectura, la edición y el cambio de estado por su propia ruta; la marca se lee una vez por página.

Las listas de compras se suman a la tabla el 29/10/2026: las de proveedores, tipos, plantillas y órdenes existen desde el 23/10/2026, y la de los contactos de un proveedor desde ese día. Cada una lista solo lo que quien llama lee con `compras.ver`, donde actúa (§9.2). `GET /suppliers/{id}/contacts` responde `404 SUPPLIER_NOT_FOUND` si el proveedor no existe o quien llama no lo lee (§6.1), y no se exporta (decisión del responsable del 29/10/2026).

**Las listas cortas, como excepción.** Hay cuatro, y responden siempre el arreglo o el objeto completo, sin `q`, `sort` ni `page`:

1. **Los escalones y los hijos de una división** (registrada el 14/10/2026). `GET /countries/{id}/geo-division-levels`, que tiene a lo sumo cuatro escalones, y `GET /geo-divisions/{id}/children`, que trae las divisiones hijas de una para el selector en cascada de la dirección. Son cortas por naturaleza y solo pueblan un select. Los escalones llegan por número y los hijos por `code`; los hijos aceptan el filtro `isActive`.
2. **Las cuentas de una empresa** (`GET /users?companyId=…`, abajo).
3. **Los estados del vehículo y los caminos** (D-113, 20/10/2026). `GET /vehicle-statuses` y `GET /status-change-reasons`. El criterio que las admite: un catálogo corto del sistema que se necesita entero para nombrar las referencias —el estado de cada unidad, el camino de cada fila del historial—. Los estados aceptan `isActive` y llegan por `code`. Los caminos aceptan `isActive` y se filtran en el servidor por su origen: `?fromStatusId={id}`, los que salen de ese estado, o `?origin=registration`, los caminos de alta, que no tienen estado de origen (§14.23, adoptado). Los dos juntos responden `400 VALIDATION_ERROR`, y también un `origin` que no sea `registration`. Llegan ordenados por su origen —los de alta primero—, su destino y su código. Un motivo automático viaja con `isAutomatic: true` (diccionario §8.3, D-110), para que Estados y motivos lo muestre sin acciones.
4. **La matriz de permisos** (D-126, construida e integrada en `develop` el 20/10/2026). `GET /permission-matrix?companyId=`, con `seguridad.ver_roles`, responde un objeto con los permisos del catálogo, los pares incompatibles, los roles que la sesión lee, cada uno con lo que incluye, y lo que la sesión puede marcar en un rol general y en uno de la empresa pedida (§2.2). Se necesita entera porque es una tabla de módulos por roles: la pantalla la contrae, la expande y la busca en el navegador, y un permiso o un rol que faltara dejaría una celda sin sentido. Sin `companyId` llegan los roles del sistema y los generales, y `grantableForCompanyRoles` viaja en `null`; con él, también los de esa empresa. Una empresa que no existe, o en la que `seguridad.ver_roles` de la sesión no actúa, responde `404 COMPANY_NOT_FOUND`, como un rol de esa empresa (D-98, D-122); sin `seguridad.ver_roles`, `403 ACCESS_DENIED`. Los roles llegan primero los del sistema, en el orden en que los sembró la migración, y después los demás por nombre. Es también la única tabla del frontend fuera de `mm-data-manager` (D-126).

Una lista que no cumple ese criterio no se suma a esta excepción: se busca y se pagina en el servidor.

**Las cuentas de una empresa, como arreglo** (registrado el 16/10/2026). `GET /users?companyId=…` responde siempre el arreglo de las cuentas que tienen un colaborador en esa empresa, sin página: lo usa la columna de cuenta de Colaboradores, que muestra la empresa entera. Cada fila es `{id, email, mustChangePassword, lastLoginAt, isSuspended, managedBySuperadmin, employeeId, roles, employeeCompanyIds}`, donde `employeeCompanyIds` son las empresas donde la cuenta tiene un colaborador activo, entre las que ve quien llama —el diálogo de roles ofrece un rol de empresa o de sede solo en ellas—, y `isSuspended` dice si un bloqueo la tiene fuera ahora; `managedBySuperadmin` dice si solo un superadministrador la maneja, porque tiene `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente en cualquier empresa, también una que `roles` no muestra o que quien llama no ve, con la misma regla que responde `ACCOUNT_MANAGED_BY_SUPERADMIN` (§9.2), y el frontend no le ofrece al administrador de accesos ninguna acción sobre ella; `employeeId` es su colaborador en esa empresa (sin `companyId`, `null`) y `roles` son sus asignaciones vigentes en esa empresa más las de alcance sistema, cada una con `assignmentId`, `roleId`, `roleCode`, `roleName`, `scopeLevel`, la empresa, la sede y la marca con sus nombres —el de la empresa es su nombre comercial— y `expiresAt`. Sin `companyId`, la lista es de todas las cuentas, con todas sus asignaciones vigentes, y se pagina como las demás. Todo se resuelve en lote, sin una consulta por fila, y la exportación sigue las mismas reglas y las mismas filas.

Ubicaciones y Colaboradores ordenan por el nombre del campo JSON desde el 18/10/2026: antes Ubicaciones entendía solo `locationtype`, `type`, `createdat` y `updatedat`, en minúsculas, y Colaboradores pasaba por alto `sort=fullName`.

Una columna que nombra un registro de otro contexto —el país y la moneda de una empresa, los roles de una cuenta— no se ordena: el dueño de la lista no lee las tablas ajenas (mapa de contextos).

**Cómo se construye.** El controlador recibe `PageParams` (`shared.interfaces.rest.resources`) y responde con `pageParams.toResponse(...)`, que arma el sobre `PageResource` o el arreglo. La consulta lleva un `PageQuery` y el servicio devuelve un `PageResult` (`shared.domain.model.valueobjects`). El adaptador del repositorio traduce el `PageQuery` con `PageRequests` (`shared.infrastructure.persistence.jpa`): los campos que acepta `sort` y el patrón de `q`. Una lista cuyas filas traen colecciones, como los países con sus escalones, pagina primero los identificadores y después lee esas filas, para que la base pagine y no la memoria.

En el frontend, `mm-data-manager` emite la búsqueda, el orden y la página en `queryChange`; el store guarda la página en un `PagedList` (`shared/application`) y el endpoint la pide con `getPage()` de `BaseApiEndpoint`.

---

## 7. Errores

### 7.1 El cuerpo

Todo error responde el mismo recurso, también los que ocurren antes del controlador (sin sesión, sin permiso):

```json
{
  "code": "CURRENCY_CONFLICT",
  "message": "Ya existe una moneda con este código ISO",
  "details": "A currency with ISO code 'PEN' already exists"
}
```

- `code` es estable y es lo que el cliente usa para decidir qué mostrar.
- `message` sale traducido según `Accept-Language` (`es` o `en`), para quien llame a la API sin el frontend. Sin `Accept-Language`, o con un idioma que la API no habla, responde en español, el idioma de las pantallas (D-39). El frontend manda `Accept-Language: es` en cada petición, para que el idioma del navegador no cambie los mensajes ni las exportaciones.
- El frontend elige su texto de `es.json` por `code` (decidido el 06/10/2026, §14.12), con un `switch` por los códigos que conoce. Un código desconocido cae en el mensaje genérico de la operación, nunca en uno específico: así una causa nueva muestra un texto incompleto, pero nunca uno falso. El estado HTTP solo decide en dos casos: `0`, sin red, y los errores del framework que todavía no tienen código del catálogo (§13 fila 18). El frontend nunca muestra `message` ni lee el cuerpo por su cuenta.
- `details` explica el caso para quien depura. Nunca lleva SQL, nombres de restricciones ni trazas. En un `500 UNEXPECTED_ERROR` es un texto fijo y traducido («Ocurrió un error inesperado. Quedó registrado para su revisión»), nunca el mensaje de la excepción, que queda solo en el registro del servidor.
- `context` es opcional (registrado el 18/10/2026): lleva los datos que el cliente necesita para actuar sobre el error, por ejemplo lo que tiene que mostrar antes de pedir una confirmación. Solo lo traen los errores que lo documentan; en todos los demás no viaja, ni siquiera como `null`. Lo traen:
  - `422 EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH` de `POST /access-grants`, solo cuando quien llama tiene `seguridad.confirmar_vinculo` —el superadministrador—: cualquier otro, también el administrador de accesos, recibe el mismo `422` sin `context`, porque los titulares son personas de otras empresas y solo un superadministrador decide sobre ese vínculo (D-82). Su forma, abajo;
  - `422 ROLE_PERMISSIONS_NOT_HELD`, con los permisos que faltan en `context.permissionCodes` (§7.2);
  - desde la segunda ronda de la auditoría (D-119, D-121, D-122), los cinco códigos de los roles y de las asignaciones que nombran lo que choca: `ROLE_PERMISSIONS_INCOMPATIBLE`, `ROLE_HOLDERS_DUTIES_INCOMPATIBLE`, `ROLE_RESERVED_HOLDERS_INVALID`, `ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE` y `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE` (§7.2). La pantalla de la matriz marca con ese `context` las celdas que lo causan.

```json
{
  "code": "EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH",
  "message": "La cuenta es de otra persona: ninguno de sus colaboradores tiene el mismo documento de identidad",
  "details": "…",
  "context": {
    "hasHolders": true,
    "holders": [
      { "employeeId": "0199…", "fullName": "Ana Pérez Rojas", "documentTypeCode": "PAS", "documentNumber": "PA7200050",
        "companyId": "0199…", "companyName": "Millennium Colombia", "isActive": true }
    ],
    "holdersFingerprint": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"
  }
}
```

  - `holders` son los colaboradores que ya tienen la cuenta del correo, activos o no, ordenados por el nombre de su empresa, solo con identificadores públicos. Puede haber varios: una persona con colaboradores en dos empresas.
  - `hasHolders` es `false`, con `holders: []`, cuando la cuenta existe pero no pertenece a ningún colaborador, por ejemplo la que dejó un vínculo equivocado al deshacerse. También esa la vincula solo un superadministrador que confirma.
  - `holdersFingerprint` es la huella de exactamente esos titulares: el SHA-256, en hexadecimal en minúsculas, de todos los campos de cada titular, ordenados por su `employeeId`. La lista vacía también tiene su huella. La confirmación la devuelve (§14.15).

### 7.2 Códigos

| `code` | Estado | Cuándo |
|---|---|---|
| `VALIDATION_ERROR` | `400` | El cuerpo, la ruta o un parámetro no cumplen el formato |
| `UNAUTHORIZED` | `401` | No hay sesión, venció, o las credenciales no son válidas |
| `ACCESS_DENIED` | `403` | Hay sesión pero falta el rol o el permiso de la ruta, o, desde el paso 3b, el permiso no actúa en la empresa alcanzada: se tiene por un rol de otra empresa (D-99); también con contraseña provisional y con el registro público apagado. En las operaciones de cuenta, es "su permiso no alcanza para esto": falta el permiso de seguridad, o se tiene por una asignación de empresa cuando la operación pide alcance sistema (D-122), incluido otorgar un rol reservado sin ser superadministrador (D-121) |
| `ACCOUNT_SUSPENDED` | `403` | La contraseña coincidió, pero la cuenta tiene un bloqueo en curso (diccionario §5.4) |
| `ACCOUNT_MANAGED_BY_SUPERADMIN` | `403` | Quien opera maneja cuentas, pero la cuenta es protegida —tiene un rol reservado vigente, como `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` (D-121)— y solo la maneja un superadministrador |
| `OWN_ACCOUNT_FORBIDDEN` | `403` | Quien opera actúa sobre su propia cuenta —otorgar, retirar, bloquear, cerrar un bloqueo, restablecer, vincularse a sí mismo fuera del arranque— o registra su propia salida o regreso |
| `ACTOR_EMPLOYEE_REQUIRED` | `403` | La cuenta de quien opera no está vinculada a un colaborador activo, así que no puede actuar sobre cuentas ni registrar salidas o regresos; tampoco un superadministrador sin vincular (D-89) |
| `<RECURSO>_NOT_FOUND` | `404` | El recurso, o uno que la petición referencia, no existe |
| `<RECURSO>_CONFLICT` | `409` | Ya existe un valor único: un código ISO, un número de comprobante |
| `<RECURSO>_<CAMPO>_CONFLICT` | `409` | Lo mismo, en un agregado con dos o más reglas de unicidad: cada una tiene su código (`VEHICLE_VIN_CONFLICT`, `VEHICLE_PLATE_CONFLICT`) |
| `METHOD_NOT_ALLOWED` **(propuesta, §14.11)** | `405` | El método no existe para esa ruta; en particular, todo `DELETE` (§3.1) |
| `CONTENT_TOO_LARGE` | `413` | Una subida pasa el límite del servidor: un archivo de más de 10 MB o un pedido de más de 21 MB. Lo responde el manejador global con el nombre del estado, sin campo, antes de que el pedido llegue al servicio; entre 5 y 10 MB responde el `422` de peso de su recurso, como `PURCHASEORDERTEMPLATE_DOCUMENT_SIZE` (registrado el 26/10/2026). El frontend lleva los dos nombres del estado, `CONTENT_TOO_LARGE` y `PAYLOAD_TOO_LARGE`, al texto del peso |
| `BUSINESS_RULE_VIOLATION` | `422` | La petición es válida pero rompe una regla del negocio: una lectura de kilometraje menor, un camino de estado no permitido, una nota de crédito en otra moneda |
| `UNEXPECTED_ERROR` | `500` | Un fallo que no debería pasar |

- Un fallo esperado del negocio se devuelve como `Result`, nunca como excepción (regla 3.1), y se traduce aquí.
- Las fábricas de `ApplicationError` (`notFound`, `conflict`, `businessRuleViolation`, `validationError`…) producen los códigos genéricos de la tabla. Por omisión, un duplicado es `ApplicationError.conflict(...)`, que responde `<RECURSO>_CONFLICT`.
- **Un duplicado que llega a la base en una carrera responde el código de su recurso**, el mismo que el servicio da sin carrera. Dos peticiones simultáneas pueden pasar las dos la comprobación del servicio; la restricción única de la base detiene a la segunda, y el adaptador del repositorio traduce esa violación en el fallo de su clave natural ([patrones de persistencia](patrones-de-persistencia-millennium-mobility.md)). En `location` responden así `CURRENCY_CONFLICT`, `COUNTRY_ISO_ALPHA2_CONFLICT`, `COUNTRY_ISO_ALPHA3_CONFLICT`, `GEODIVISION_CONFLICT` y `TAXIDTYPE_CONFLICT` (`409`). En `organization`, desde el 18/10/2026, `COMPANY_COUNTRY_CONFLICT`, `COMPANY_TAX_ID_CONFLICT`, `BRANCH_CODE_CONFLICT`, `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT`, `COMMERCIALBRAND_CONFLICT`, `LOCATION_CODE_CONFLICT`, `LOCATION_BRANCH_CONFLICT`, `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` y `EMPLOYEE_ACCOUNT_IN_COMPANY_CONFLICT`. En `vehiclecatalog`, desde el 20/10/2026, `VEHICLETYPE_CONFLICT`, `VEHICLEBRAND_CONFLICT`, `VEHICLEMODEL_CONFLICT`, `COLOR_CONFLICT` y `EQUIPMENTTYPE_CONFLICT`, sobre `vehicle_types_code_uq`, `vehicle_brands_name_uq`, `vehicle_models_brand_name_uq`, `colors_name_uq` y `equipment_types_code_uq` (V35). En `fleet`, `VEHICLE_VIN_CONFLICT`, `VEHICLE_PLATE_CONFLICT`, `STATUSCHANGEREASON_REGISTRATION_CONFLICT` y, desde el 26/10/2026, `FUND_CONFLICT`, sobre el código del fondo en su empresa; el cruce de dos cesiones que detiene la exclusión de la base (`23P01`) responde `VEHICLEFUNDASSIGNMENT_CONFLICT`. En `procurement`, desde el 26/10/2026, `SUPPLIER_TAX_ID_CONFLICT`, `PURCHASEORDERTYPE_CODE_CONFLICT` y `PURCHASEORDERTEMPLATE_SERIES_CONFLICT`, sobre `suppliers_company_tax_id_uq` (V63), `purchase_order_types_company_code_uq` (V67) y `purchase_order_templates_company_series_uq` (V68), en el alta y, para la serie, también en la edición. La numeración de las órdenes (`purchase_orders_number_uq`) no se traduce a propósito: corre bajo el candado de su plantilla, así que una violación ahí es un defecto que tiene que verse. Solo un duplicado que su adaptador todavía no traduce llega al manejador global y responde el genérico `RESOURCE_CONFLICT` (`409`), sin el texto de la base. Desde el 29/10/2026 el manejador deja además en el registro del servidor una advertencia con el nombre de la restricción, leído del mensaje de PostgreSQL en inglés o en español, sin los valores de la llave; quien llama sigue recibiendo `RESOURCE_CONFLICT` sin él (`GlobalExceptionHandler`).
- Cuando un agregado tiene dos o más reglas de unicidad, cada una lleva su código propio `<RECURSO>_<CAMPO>_CONFLICT`, que produce `ApplicationError.conflict(recurso, campo, …)`, para que el cliente sepa cuál falló: `COMPANY_COUNTRY_CONFLICT` y `COMPANY_TAX_ID_CONFLICT` en la empresa, que ya los emite; `VEHICLE_VIN_CONFLICT` y `VEHICLE_PLATE_CONFLICT` en la unidad; `BRANCH_CODE_CONFLICT` y `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT` en la sede (diccionario §6.4). Un agregado con una sola regla se queda con el genérico, como `FUND_CONFLICT` y `VEHICLEFUNDASSIGNMENT_CONFLICT`, sin campo. Tres excepciones, ya publicadas y mapeadas por el frontend, llevan su campo aunque su agregado tenga una sola regla única: `SUPPLIER_TAX_ID_CONFLICT`, `PURCHASEORDERTYPE_CODE_CONFLICT` y `PURCHASEORDERTEMPLATE_SERIES_CONFLICT` (registrado el 26/10/2026). Responden igual con o sin carrera.
- El estado de un código propio sale de su sufijo: `_NOT_FOUND` responde `404`, `_CONFLICT` responde `409` y cualquier otro, `422`. Por eso un código propio de duplicado termina siempre en `_CONFLICT`: con otro sufijo saldría como regla rota. Un código propio que no es un duplicado, como `COMPANY_FISCAL_ADDRESS_MISSING` (§3.3), responde `422`. Los cuatro `403` propios de la tabla (`ACCOUNT_SUSPENDED`, `ACCOUNT_MANAGED_BY_SUPERADMIN`, `OWN_ACCOUNT_FORBIDDEN` y `ACTOR_EMPLOYEE_REQUIRED`) son la excepción: no tienen un sufijo que dé su estado, así que `ErrorResponseAssembler` los nombra uno por uno. Lo mismo `DOCUMENT_EXPIRED` (`410`) y `DOCUMENTS_BUSY` (`503`), desde el 20/10/2026. Un código `_OUTDATED` —el candado de `updatedAt` de una ficha (§3): `SUPPLIER_OUTDATED`, `PURCHASEORDERTEMPLATE_OUTDATED` y, desde el 29/10/2026, `PURCHASEORDER_OUTDATED` y `SUPPLIER_CONTACT_OUTDATED`— no es un duplicado ni una excepción: responde `422` por su sufijo, como todo código propio (D-134). Hasta el 26/10/2026 `SUPPLIER_OUTDATED` era un caso con nombre que respondía `409`; ya no lo es, y el frontend elige su texto por el código, no por el estado.
- Un `403` dice por qué cuando la pantalla tiene que explicarlo: cada causa tiene su código, y `ACCESS_DENIED` queda para la falta de rol o de permiso, y para el permiso que no actúa en la empresa (D-99). El rol o el permiso de la ruta se revisa antes que la existencia: quien no lo tiene en ninguna empresa recibe `403` aunque el recurso no exista; la exportación de colaboradores lo exige en su método, así que quien no tiene `configuracion.ver_datos_personales_colaboradores` en ninguna empresa recibe `403` antes de que se busque la empresa. Quien lo tiene en otra empresa recibe `404 COMPANY_NOT_FOUND` por una fuera de su alcance, y `403` por una alcanzada donde el permiso no actúa (D-99).
- Como todo `Result.failure` es un fallo esperado, un código propio nunca responde `500`. Su texto en el servidor se busca en `error.<código-en-kebab>.message`, por ejemplo `error.vehicle-vin-conflict.message`, y el frontend tiene su propia clave en `es.json` (§7.1).
- `<RECURSO>` es el nombre del agregado en mayúsculas y sin separar sus palabras, como lo producen las fábricas: `notFound("GeoDivision", …)` produce `GEODIVISION_NOT_FOUND`. **Cada agregado tiene un solo prefijo** (D-114, decidido el 20/10/2026): sus códigos propios usan el mismo que su `_NOT_FOUND`, sin guion bajo interno. Así `fleet` pasó de `VEHICLE_STATUS_*` a `VEHICLESTATUS_*` y de `STATUS_CHANGE_REASON_*` a `STATUSCHANGEREASON_*` (abajo), y la propuesta de separar las palabras con guion bajo (§14.11) queda rechazada.
- **(propuesta, §14.5)** Un `422` dice qué regla falló en `rule`, un identificador estable: `"rule": "mileage-reading.not-lower"`. Así el cliente distingue reglas sin leer el texto.

**Los códigos del documento tributario y de la ubicación** (registrado el 18/10/2026). Todos responden `422`. Los de la dirección están en §3.3.

| `code` | Operación | Campo | Cuándo |
|---|---|---|---|
| `COMPANY_TAX_ID_CHECK_DIGIT_INVALID` | `POST /companies`, `PUT /companies/{id}/tax-id` | `taxId` | El tipo tiene dígito de control y el número no lo cumple (D-75): el RUC `20512345678`, cuyo dígito correcto es `1`. Un tipo con dígito de control cuyo cálculo `location` todavía no tiene —el NIT, el RFC y la CURP— acepta el número sin verificarlo |
| `EMPLOYEE_DOCUMENT_NUMBER_CHECK_DIGIT_INVALID` | `POST /companies/{id}/employees`, `PUT /employees/{id}` | `documentNumber` | Lo mismo, para el documento del colaborador. En la edición se verifica solo si cambian el tipo o el número: un documento ya guardado sigue siendo válido |
| `SUPPLIER_TAX_ID_CHECK_DIGIT_INVALID` | `POST /suppliers` | `taxId` | Lo mismo, para el documento del proveedor (registrado el 26/10/2026). Antes respondía `400 VALIDATION_ERROR`. El documento de un proveedor no se corrige después del alta (diccionario §9.1), así que solo el alta lo verifica |
| `EMPLOYEE_TERMINATION_BEFORE_HIRE` | `PUT /employees/{id}` | `hireDate` | La edición deja la fecha de ingreso de un colaborador que ya salió después de su salida (registrado el 18/10/2026). Es el mismo código que responde una salida anterior al ingreso (más abajo) |
| `LOCATION_OWNERSHIP_CHANGE_NOT_ALLOWED` | `PUT /locations/{id}` | `locationType` | La edición pasa una sede propia a ubicación de terceros o al revés (D-79). Entre los tipos de terceros sí se cambia |
| `LOCATION_ADDRESS_MISSING` | `PUT /locations/{id}/address` | — | La ubicación no tiene dirección que corregir; la primera se registra con `POST` |
| `LOCATION_ADDRESS_NOT_ALLOWED` | `PUT` y `POST /locations/{id}/address` | — | Es una sede propia: usa la dirección de su sede, que se corrige o se muda en `/branches/{id}/address` |

- El número se revisa antes que los duplicados: un RUC mal escrito responde `422` (`COMPANY_TAX_ID_CHECK_DIGIT_INVALID`, o `BUSINESS_RULE_VIOLATION` por el tipo o la longitud) aunque el país ya tenga su empresa, en vez de `409`. Lo mismo el documento de un colaborador y el de un proveedor, que responde `422` antes que `409 SUPPLIER_TAX_ID_CONFLICT`.
- Los demás rechazos del tipo de documento —de otro país, inactivo, que no aplica a empresas o a personas, de otra longitud— siguen siendo `BUSINESS_RULE_VIOLATION`. Un tipo inactivo se rechaza solo cuando es nuevo para el registro: el colaborador que ya lo tenía se sigue editando (LOC-R36). El proveedor sigue la regla de la empresa, porque es solo local (D-133): un tipo del país de la empresa que le compra, activo, que aplica a empresas y con su longitud. Antes solo se revisaba que estuviera activo, y uno inactivo respondía `409 TAX_ID_TYPE_CONFLICT`. Un tipo que no existe responde `404 TAXIDTYPE_NOT_FOUND`.
- El número del proveedor se normaliza como el de la empresa (§5.3), también los separadores Unicode; V76 normalizó los ya guardados (diccionario §9.1, D-134).
- La longitud que declara un tipo de documento va de 1 a 20 (D-78): con 21 o más, `400 VALIDATION_ERROR`.
- Un número hecho solo de separadores, como `-.-`, queda vacío al limpiarlo y responde `400 VALIDATION_ERROR` en su campo, con cualquier tipo, también el RUC (registrado el 18/10/2026): `taxId` en `POST /companies` y `PUT /companies/{id}/tax-id`, y `documentNumber` en `POST /companies/{companyId}/employees` y `PUT /employees/{id}`. Antes respondía `422 BUSINESS_RULE_VIOLATION` en el colaborador, y en la empresa un `422` o un `400` sin campo según el tipo.
- La longitud del número, como su dígito de control, se mide sin separadores y solo al crear el registro o cuando cambian el número o el tipo (D-91): editar un colaborador cuyo documento no cambia no lo vuelve a medir.

**El código de los decimales de una moneda** (registrado el 29/10/2026, decisión del responsable de esa fecha; diccionario §4.1).

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `CURRENCY_DECIMAL_PLACES_IN_USE` | `422` | `PUT /currencies/{id}` | El pedido cambia los decimales de una moneda que ya usa una orden de compra, de cualquier empresa y en cualquier estado, también anulada. No cambia nada: un `PUT` que conserva los decimales sigue cambiando el nombre y el símbolo |

- `location` bloquea la moneda antes de leerla y, solo si cambian los decimales, pregunta a cada contexto que guarda montos por el puerto `CurrencyUsage`, de su contrato `location::acl`, con el alcance `SYSTEM`: cuenta las órdenes de todas las empresas, que quien corrige la moneda quizá no lee. `location` no depende de quien lo implementa. Hoy lo implementa `procurement` con sus órdenes; los fondos no tienen moneda (diccionario §12.1), y las facturas, las notas de crédito y los pagos lo implementarán cuando se construyan.
- La orden lee su moneda en modo compartido antes de revisar sus precios: la corrección espera a una orden abierta, y una orden espera a la corrección y revisa sus precios con los decimales nuevos ([contextos/procurement.md](contextos/procurement.md), PRC-R26).

**Los códigos de lo que se cuelga de una empresa, una sede o una marca** (registrado el 18/10/2026). A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo (D-84); lo que ya la tenía se sigue editando mientras no cambie ese vínculo. Todos responden `422` salvo que se indique otro estado.

| `code` | Operación | Cuándo |
|---|---|---|
| `COMPANY_INACTIVE` | `POST /companies/{id}/branches`, `/commercial-brands`, `/locations` y `/employees`; `POST /users/{id}/role-assignments` | La empresa está inactiva |
| `COMPANY_INACTIVE` | `PATCH /branches/{id}/status`, `/commercial-brands/{id}/status` y `/locations/{id}/status` con `{"isActive": true}` | Se reactiva una sede, una marca o una ubicación inactiva cuya empresa está inactiva: reactivar es volver a colgar (D-94), y primero se reactiva la empresa. En una ubicación propia, la empresa se revisa antes que la sede, así que responde este código antes que `LOCATION_BRANCH_INACTIVE`. Una que ya está activa no se revisa, y desactivar no espera a nada (registrado el 18/10/2026) |
| `COMPANY_INACTIVE` | `POST /funds`; `PATCH /funds/{id}/status` con `{"isActive": true}` | La empresa del fondo está inactiva: no se crea un fondo bajo ella ni se reactiva uno (D-84, D-94). La empresa se lee en modo compartido después del fondo, así que no se desactiva en el medio. Desactivar no espera a nada (registrado el 26/10/2026) |
| `COMPANY_INACTIVE`, `BRANCH_INACTIVE` | `POST /employees/{id}/reactivation` | El colaborador regresaría a una empresa inactiva o, si tiene sede, a una sede inactiva (D-94). No regresa, y su cuenta sigue cerrada. El orden del regreso está en la tabla de la salida y el regreso (registrado el 18/10/2026) |
| `BRANCH_INACTIVE` | El alta de un colaborador o de una sede propia; su edición cuando cambia de sede; `POST /users/{id}/role-assignments` | La sede es nueva para el registro y está inactiva. La sede inactiva que ya tenía se conserva |
| `BRANCH_OTHER_COMPANY` | El alta y la edición de un colaborador o de una sede propia | La sede es de otra empresa. Antes respondía `422 BUSINESS_RULE_VIOLATION` en el colaborador y `400 VALIDATION_ERROR` en la ubicación |
| `COMMERCIALBRAND_INACTIVE` | `POST /users/{id}/role-assignments`; `POST /vehicles` y `PUT /vehicles/{id}` cuando cambia la marca | La marca está inactiva. Desde el 20/10/2026 su texto sirve a las dos operaciones: «La marca comercial está inactiva: no se le asigna nada nuevo» («The commercial brand is inactive: nothing new is assigned to it») |
| `EMPLOYEE_HIRE_DATE_IN_FUTURE` | `POST /companies/{id}/employees`, `PUT /employees/{id}` | La fecha de ingreso es posterior a hoy en la zona de la sede del colaborador o, sin sede, del país de su empresa (D-88) |
| `LOCATION_BRANCH_CONFLICT` (`409`) | `POST /companies/{id}/locations`, `PUT /locations/{id}` con `sede_propia` | La sede ya tiene su ubicación propia, activa o inactiva (D-90): la inactiva se reactiva, no se crea otra |
| `LOCATION_BRANCH_INACTIVE` | `PATCH /locations/{id}/status` con `{"isActive": true}` | Se reactiva una sede propia cuya sede está inactiva (D-90): primero se reactiva la sede. Una ubicación de terceros no tiene sede y no se frena |

- `LOCATION_CODE_CONFLICT` y `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` (`409`) tienen su propio mensaje desde el 18/10/2026; antes salían con el genérico «Conflicto de recurso».
- `photoPath` llega hasta 500 caracteres (V43): con más, `400 VALIDATION_ERROR` sobre ese campo, en el alta y la edición del colaborador. Antes respondía `422 BUSINESS_RULE_VIOLATION` sin campo.
- Los códigos de una sede o una ubicación conservan sus guiones: solo se recortan y se pasan a mayúsculas, así `LIM-1` y `LIM1` son dos códigos distintos (D-86, diccionario §2.2). Los separadores se quitan solo de los identificadores de una norma externa (§5.3).

**Los códigos de compras** (registrado el 26/10/2026, D-134). Una referencia inactiva ya no responde `409 <RECURSO>_CONFLICT` ni `404`: responde `422` con el código del agregado que está inactivo, y solo a quien escribe donde actúa, porque el permiso se revisa antes. Cada agregado tiene un solo prefijo (D-114).

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `COMPANY_INACTIVE` | `422` | `POST /suppliers`, `POST /purchase-order-types`, el alta de una plantilla; activar un tipo o una plantilla; `POST /purchase-orders` | La empresa está inactiva. En `POST /purchase-orders`, la empresa de la plantilla, revisada después del permiso y de la plantilla inactiva, antes de la marca y el tipo (D-84, registrado el 27/10/2026) |
| `COMMERCIALBRAND_INACTIVE` | `422` | El alta de una plantilla y su `PATCH` cuando cambia la marca; activar una plantilla de una marca; `POST /purchase-orders` | La marca es nueva para la plantilla y está inactiva. Desde el 27/10/2026 también reactivar una plantilla cuya marca está inactiva (D-94) y emitir una orden con ella (D-84) |
| `PURCHASEORDERTYPE_INACTIVE` | `422` | El alta de una plantilla y su `PATCH` cuando cambia el tipo; `POST /purchase-orders` | El tipo es nuevo para la plantilla y está inactivo. Desde el 27/10/2026 también emitir una orden con una plantilla cuyo tipo está inactivo (diccionario §9.2): el tipo se lee con candado compartido. Desde el 29/10/2026 su texto sirve a las dos operaciones: con ese tipo no se emite nada nuevo. La empresa, la marca, el proveedor con sus contactos y la moneda de la orden también se leen desde esa fecha con candado compartido, así que su desactivación espera a la orden o se confirma antes y la orden la ve |
| `PURCHASEORDERTYPE_CODE_LOCKED` | `422` | `PUT /purchase-order-types/{id}` con `code` | Una plantilla ya usa el tipo: su código queda fijo (diccionario §9.2). El mismo código, escrito como sea, no es corrección, y el resto de la edición pasa. El recurso lo anuncia con `codeLocked` |
| `PURCHASEORDERTEMPLATE_DOCUMENT_UNREADABLE` | `422` | `POST /purchase-orders/{id}/sending` | El logo o la firma de la plantilla falta en el almacén, no se lee, no se decodifica, supera el tope o no es PNG, JPEG ni WebP: la hoja enviada es la copia fiel que se guarda para siempre, así que no se envía sin ella. `details` dice `LOGO` o `SIGNATURE`. La orden sigue en borrador y no se guarda ninguna hoja; el borrador sí se imprime, sin esa imagen |
| `PURCHASEORDERTEMPLATE_INACTIVE` | `422` | `POST /purchase-orders` | La plantilla existe y quien emite la lee, pero está inactiva. Antes respondía `404 PURCHASEORDERTEMPLATE_NOT_FOUND` |
| `SUPPLIER_INACTIVE` | `422` | `POST /purchase-orders`, `PUT /purchase-orders/{id}`; `POST /suppliers/{id}/contacts`; `PATCH /supplier-contacts/{id}/status` con `{"isActive": true}` | El proveedor es de la empresa y quien emite lo lee, pero está inactivo. Antes respondía `404 SUPPLIER_NOT_FOUND`. Desde el 27/10/2026 el `PUT` conserva el proveedor que el borrador ya tiene aunque se haya desactivado después, y también su contacto mientras el proveedor no cambia (D-84, D-108). Desde el 29/10/2026 un proveedor inactivo tampoco recibe un contacto nuevo ni reactiva uno retirado (D-94, D-108), y quien no escribe sus contactos recibe antes `403` |
| `CURRENCY_INACTIVE` | `422` | `POST /purchase-orders`, `PUT /purchase-orders/{id}` | La moneda está inactiva. Antes respondía `404 CURRENCY_NOT_FOUND`. Desde el 27/10/2026 el `PUT` conserva la moneda que el borrador ya tiene (D-84) |
| `CURRENCY_NOT_FOUND` | `404` | `POST /purchase-orders`, `PUT /purchase-orders/{id}` | Ninguna moneda tiene el código de `currency`. Desde el 29/10/2026 la orden nombra su moneda por su código (§4.2); antes, por `currencyId` |
| `PURCHASEORDER_OUTDATED` (nuevo) | `422` | `PUT /purchase-orders/{id}` | El borrador se abrió y la orden cambió después —otra edición, un envío, un cumplimiento o una anulación—: el `updatedAt` del cuerpo no es el de la fila (V85, §3). No se guarda nada. Registrado el 29/10/2026, cuando el `PUT` empezó a exigir `updatedAt` |
| `SUPPLIER_CONTACT_INACTIVE` (nuevo) | `422` | `POST /purchase-orders`, `PUT /purchase-orders/{id}` | El contacto es de ese proveedor, pero está retirado y no es el que el borrador ya tiene (D-108). Registrado el 29/10/2026: junto con `SUPPLIER_CONTACT_NOT_FOUND` reemplaza a `PURCHASEORDER_CONTACT` (`422`), que respondía igual a un contacto de otro proveedor y a uno retirado, y ya no existe |
| `VEHICLEMODEL_INACTIVE`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE` | `422` | Las líneas de `POST /purchase-orders` y `PUT /purchase-orders/{id}` | El modelo de la línea, su marca o su tipo de vehículo están inactivos (D-108). Antes respondían `409 VEHICLEMODEL_CONFLICT`; un modelo que no existe responde `404 VEHICLEMODEL_NOT_FOUND`. Desde el 27/10/2026 una línea del `PUT` que conserva su modelo no se rechaza por su retiro; una nueva, o una que cambia de modelo, sí |
| `PURCHASEORDERTEMPLATE_NOT_FOUND` | `404` | `POST /purchase-orders` con un `companyId` que no es el de la plantilla | Esa empresa no tiene esa plantilla. Antes el `companyId` del alta se ignoraba (registrado el 27/10/2026) |
| `VALIDATION_ERROR` | `400` | Las líneas de `POST /purchase-orders` y `PUT /purchase-orders/{id}` | `unitPrice.currency` no es el código de la moneda de la orden. Antes se ignoraba (registrado el 27/10/2026). Desde el 29/10/2026 también, en su lugar: `currency` que no son tres letras; un `PUT` sin `updatedAt`; una línea enviada como `null` (`items[n]`), que antes llegaba al servicio y respondía `500`; una tasa de descuento de más de cuatro decimales, la escala de `NUMERIC(5,4)`, que antes la base redondeaba sin decirlo; un precio unitario mayor que `999999999999.99`, el tope de `NUMERIC(14,2)` (`items[n].unitPrice.amount`), que antes fallaba en la base. Un precio negativo también responde `400`. Los decimales del precio siguen respondiendo `422 PURCHASEORDER_PRICE_SCALE`, que conoce la moneda |
| `SUPPLIER_OUTDATED` | `422` | `PUT /suppliers/{id}` | La ficha se abrió y la fila cambió después. Antes respondía `409`. Desde el 29/10/2026 escribir un contacto ya no mueve el `updatedAt` del proveedor |
| `SUPPLIER_CONTACT_OUTDATED` (nuevo) | `422` | `PUT /supplier-contacts/{id}` | El contacto se abrió y su fila cambió después (§3). No se guarda nada. Registrado el 29/10/2026, con el prefijo de los demás códigos del contacto (D-114) |
| `SUPPLIER_CONTACT_NOT_FOUND` | `404` | `POST /purchase-orders`, `PUT /purchase-orders/{id}`; `GET` y `PUT /supplier-contacts/{id}`, `PATCH /supplier-contacts/{id}/status` | En una orden, el contacto no es de ese proveedor (desde el 29/10/2026; antes `PURCHASEORDER_CONTACT`, arriba). En las rutas del contacto, no existe o quien llama no lee su proveedor con `compras.ver`. Es un código del agregado proveedor, con su prefijo. Hasta el 29/10/2026 lo respondía `PUT /suppliers/{id}` por un contacto de la ficha que no era de ese proveedor; ese `PUT` ya no lleva contactos |
| `PURCHASEORDER_NOT_FOUND` | `404` | El detalle, el PDF y las escrituras de una orden | Un solo prefijo: el detalle respondía `PURCHASE_ORDER_NOT_FOUND` |
| `PURCHASEORDER_PDF_NOT_FOUND` | `404` | `GET /purchase-orders/{id}/pdf` | La orden ya no es borrador y no tiene hoja guardada, o su archivo falta. Antes `PURCHASEORDERPDF_NOT_FOUND` |
| `TAXIDTYPE_NOT_FOUND` | `404` | `POST /suppliers` | El tipo de documento no existe. Antes `TAX_ID_TYPE_NOT_FOUND` |
| `PURCHASEORDERTEMPLATEDOCUMENT_NOT_FOUND`, `PURCHASEORDERTEMPLATEDOCUMENTTYPE_NOT_FOUND` | `404` | Los archivos de una plantilla, también `removedDocuments` | El archivo, o su tipo, no existe; ahora con su texto |
| `PURCHASEORDERTEMPLATE_DOCUMENT_DIMENSIONS` | `422` | El alta de una plantilla, su `PATCH` multipart y `PUT /purchase-order-templates/{id}/documents/{type}` | La imagen pasa de 4096 píxeles por lado o de 16 megapíxeles, según su encabezado (diccionario §9.2). Se rechaza antes de decodificarla: una imagen chica en bytes podría agotar la memoria al dibujarla |

- Escribir exige leer (diccionario §5.6, D-134): una escritura sobre un proveedor, una plantilla, un tipo o una orden que quien llama no lee con `compras.ver` responde su `404`, como si no existiera, y emitir una orden con una plantilla o un proveedor que no lee, también. La respuesta de cada escritura es lo que el comando guardó, sin volver a filtrar.
- El emisor de la plantilla ya no responde `404 TAX_ID_TYPE_NOT_FOUND` ni `409 TAX_ID_TYPE_CONFLICT` por su tipo de documento: se copia de la empresa y no se valida (§3.4).

**Los códigos de los roles y de la matriz de permisos** (registrado el 19/10/2026; diccionario §5.5, §5.6, D-96 a D-98; la segunda ronda de la auditoría, construida e integrada en `develop` el 20/10/2026, D-119 a D-124). Los escribe `iam`; crea, renombra, activa o desactiva y marca roles quien tiene, con alcance sistema, `seguridad.crear_roles`, `editar_roles`, `desactivar_roles` o `marcar_permisos_de_roles`, hoy solo el superadministrador (D-130).

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `ROLE_CONFLICT` | `409` | `POST /roles` | La empresa, o los roles generales, ya tiene un rol con ese código; también cuando dos altas chocan en la base |
| `ROLE_CODE_RESERVED` | `422` | `POST /roles` | El código es el de uno de los siete roles del sistema |
| `ROLE_SCOPE_INVALID` | `422` | `POST /roles` | Un rol de una empresa con alcance `SYSTEM`: su alcance es `COMPANY` o `BRANCH` |
| `COMPANY_INACTIVE` | `422` | `POST /roles` | La empresa del rol está inactiva (D-84) |
| `ROLE_SYSTEM_IMMUTABLE` | `422` | `PUT /roles/{id}`, `PATCH /roles/{id}/status`, `PUT /roles/{id}/permissions` | El rol es del sistema: su nombre, su estado y su matriz los fijan las migraciones (D-97) |
| `ROLE_PERMISSIONS_NOT_HELD` | `422` | `PUT /roles/{id}/permissions`; `POST /roles` con `copyPermissionsFromRoleId` | Quien marca, o quien copia, no posee alguno de los permisos, contados como dice D-122; también uno de los tres que ningún rol propio incluye (`seguridad.confirmar_vinculo`, `desvincular_cuentas` y `asignar_roles_sin_tenerlos`, D-117, D-120), que nadie posee para marcarlo. Los que faltan viajan en `context.permissionCodes`, en orden, y no se marca nada (§5.5 regla 1, D-124). El cuerpo está documentado como `RolePermissionsNotHeldError` |
| `ROLE_PERMISSIONS_INCOMPATIBLE` (nuevo) | `422` | `PUT /roles/{id}/permissions`; `POST /roles` con copia | El rol juntaría las dos partes de un par incompatible (D-119); los pares viajan en `context.incompatiblePairs`, cada uno como `{permissionCode, incompatiblePermissionCode}` y en el orden de sus códigos, y no se marca nada |
| `ROLE_HOLDERS_DUTIES_INCOMPATIBLE` (nuevo) | `422` | `PUT /roles/{id}/permissions` | El rol no junta un par, pero alguno de sus titulares lo juntaría con sus otros roles vigentes (D-119); los pares viajan en `context.incompatiblePairs`, con la misma forma |
| `ROLE_RESERVED_HOLDERS_INVALID` (nuevo) | `422` | `PUT /roles/{id}/permissions` | Los permisos nuevos vuelven reservado el rol, y alguno de sus titulares tiene una asignación acotada a una marca, una contraseña provisional o una asignación que no otorgó un superadministrador (D-121); `context.causes` dice cuáles, en este orden y sin repetir: `assignmentLimitedToBrand`, `temporaryPassword` y `notGrantedBySuperadmin` |
| `ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE` (nuevo) | `422` | `PUT /roles/{id}/permissions`; `POST /roles` con copia | El rol no es de alcance `SYSTEM` y trae alguno de los once permisos de accesos y roles (D-122, D-130); los que sobran viajan en `context.permissionCodes` |
| `PERMISSION_NOT_FOUND` | `404` | `PUT /roles/{id}/permissions` | Un código que el catálogo no tiene; no se marca nada |
| `ROLE_NOT_FOUND` | `404` | `GET /roles/{id}`, las tres escrituras y el rol de una copia | No existe, o es de una empresa donde `seguridad.ver_roles` de quien llama no actúa (D-98, D-122) |
| `ROLE_INACTIVE` | `422` | `POST /users/{id}/role-assignments` | El rol propio está desactivado: ya no se otorga, y lo que tenía asignado sigue vigente (D-97) |
| `ROLE_ASSIGNMENT_COMPANY_MISMATCH` | `422` | `POST /users/{id}/role-assignments` | Un rol de una empresa se otorga en otra (D-98). Sin empresa, el nivel del rol responde antes `ROLE_ASSIGNMENT_SCOPE_MISMATCH` |

- `PUT /roles/{id}/permissions` revisa en este orden (`RoleCommandServiceImpl`), y responde el primero que falla: `seguridad.marcar_permisos_de_roles` con alcance sistema (`403 ACCESS_DENIED`); el rol (`404 ROLE_NOT_FOUND`); que no sea del sistema (`422 ROLE_SYSTEM_IMMUTABLE`); cada código en el catálogo (`404 PERMISSION_NOT_FOUND`); que quien edita no tenga vigente el rol (`403 ACCESS_DENIED`, D-70); los pares (`422 ROLE_PERMISSIONS_INCOMPATIBLE`); los once de accesos y roles en un rol que no es de alcance sistema (`422 ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE`, D-130); lo que quien edita posee donde el rol actúa, con los tres que ningún rol propio incluye (`422 ROLE_PERMISSIONS_NOT_HELD`); agregar o quitar un permiso privilegiado en un rol con titulares vigentes sin ser superadministrador (`403 ACCESS_DENIED`, D-121); los titulares que juntarían un par (`422 ROLE_HOLDERS_DUTIES_INCOMPATIBLE`); y los titulares de un rol que se vuelve reservado (`422 ROLE_RESERVED_HOLDERS_INVALID`). Todo corre bajo el candado de la separación de funciones, que toma antes de leer el rol, y los titulares se bloquean en el orden de su identificador (D-119).
- `POST /roles` revisa en este orden: el permiso de quien llama (`403`), `ROLE_CODE_RESERVED`, la empresa (`404 COMPANY_NOT_FOUND`, `COMPANY_INACTIVE`, `ROLE_SCOPE_INVALID`), `ROLE_CONFLICT` y la forma del código y el nombre (`400 VALIDATION_ERROR`: de 2 a 40 mayúsculas, dígitos o guiones bajos; el nombre hasta 80). Con `copyPermissionsFromRoleId` sigue, bajo el mismo candado: el rol que se copia (`404 ROLE_NOT_FOUND` si no existe o es de una empresa donde `seguridad.ver_roles` no actúa), y después `ROLE_PERMISSIONS_INCOMPATIBLE`, `ROLE_PERMISSION_REQUIRES_SYSTEM_SCOPE` y `ROLE_PERMISSIONS_NOT_HELD`, con el alcance y la empresa del rol nuevo (D-124).
- Copiar `SUPERADMIN` responde `ROLE_PERMISSIONS_INCOMPATIBLE`, porque junta doce pares, y copiar `ADMIN_ACCESOS`, `ROLE_PERMISSIONS_NOT_HELD`, porque `seguridad.asignar_roles_sin_tenerlos` no lo lleva ningún rol propio (D-120, D-124). La pantalla no lo ofrece, y dice por qué.
- Quien edita un rol que tiene vigente no le cambia los permisos (D-70, D-122): responde `403 ACCESS_DENIED`. Solo cubre los permisos: renombrar un rol o cambiar su estado no la exigen.
- `PUT /roles/{id}/permissions` exige `permissionCodes`, también vacío (`400`), y valida cada elemento: un código nulo o en blanco responde `400 VALIDATION_ERROR` en vez de buscarse en el catálogo. La documentación de `context` en `ErrorResource` (§7.1) nombra `ROLE_PERMISSIONS_NOT_HELD`, junto a `EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH`; las formas de los cuatro nuevos (`IncompatiblePairs`, `SystemScopePermissions` y `ReservedHolders`) las documentan las respuestas `422` de `POST /roles`, `PUT /roles/{id}/permissions` y `POST /users/{id}/role-assignments`.
- Leen `GET /roles`, `GET /roles/{id}`, `GET /permissions` y `GET /permission-matrix` quienes tienen `seguridad.ver_roles`, que la semilla da a los tres administradores; sin él, `403 ACCESS_DENIED`. `GET /permissions` y la matriz no ofrecen los ocho códigos gruesos (D-130). Las escrituras exigen, con alcance sistema, `seguridad.crear_roles`, `editar_roles`, `desactivar_roles` o `marcar_permisos_de_roles`, según la operación: el administrador de accesos y el de empresa reciben `403` en cada una (D-130).

**Los códigos de los catálogos de flota** (registrado el 20/10/2026; diccionario §7, D-73, D-74, D-107, D-108; D-130). Los escribe `vehiclecatalog` con el permiso de la familia y de la operación —crear, editar o desactivar tipos, marcas y modelos, colores o equipamiento—; los lee cualquiera con `catalogos.ver` (§9.2). `catalogos.gestionar_flota` ya no abre ninguna ruta.

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `VEHICLETYPE_NOT_FOUND` | `404` | `/vehicle-types/{id}`; el alta y la edición de un modelo; el alta y la edición de un tipo de equipamiento | El tipo no existe. En un tipo de equipamiento, `details` nombra los que faltan; en el filtro `vehicleTypeId` de una lista, uno inexistente no responde `404`: no coincide con ninguna fila |
| `VEHICLETYPE_CONFLICT` | `409` | `POST /vehicle-types` | Otro tipo tiene ese código, una vez recortado y en mayúsculas; también en una carrera que la base detiene. El código de un tipo no cambia, así que la edición no lo revisa |
| `VEHICLETYPE_INACTIVE` (nuevo) | `422` | `POST /vehicle-brands/{id}/vehicle-models`, `PUT /vehicle-models/{id}` | El tipo elegido está inactivo y es nuevo para el modelo. El inactivo que el modelo ya tenía se conserva al editarlo (D-108) |
| `VEHICLEBRAND_NOT_FOUND` | `404` | `/vehicle-brands/{id}`; la lista y el alta de sus modelos | La marca no existe. En el filtro `vehicleBrandId` de `GET /vehicle-models` una marca que no existe no responde `404`: no coincide con ningún modelo (29/10/2026) |
| `VEHICLEBRAND_CONFLICT` | `409` | `POST /vehicle-brands`, `PUT /vehicle-brands/{id}` | Otra marca tiene ese nombre, una vez recortado y en mayúsculas; también en una carrera |
| `VEHICLEBRAND_INACTIVE` (nuevo) | `422` | `POST /vehicle-brands/{id}/vehicle-models` | La marca está inactiva: no recibe modelos nuevos (D-108). Sus modelos se siguen editando |
| `VEHICLEMODEL_NOT_FOUND` | `404` | `/vehicle-models/{id}` | El modelo no existe |
| `VEHICLEMODEL_CONFLICT` | `409` | `POST /vehicle-brands/{id}/vehicle-models`, `PUT /vehicle-models/{id}` | La marca ya tiene un modelo con ese nombre, en cualquier mayúscula; también en una carrera. Otra marca puede usarlo |
| `VEHICLEMODEL_TYPE_LOCKED` (nuevo) | `422` | `PUT /vehicle-models/{id}` | El modelo ya tiene unidades y la edición cambia su tipo de vehículo (D-107). El recurso del modelo lo anuncia con `vehicleTypeLocked` (§4.2). Su texto está en `error.vehiclemodel-type-locked.message` |
| `COLOR_NOT_FOUND` | `404` | `/colors/{id}` | El color no existe |
| `COLOR_CONFLICT` | `409` | `POST /colors`, `PUT /colors/{id}` | Otro color tiene ese nombre, una vez recortado y en mayúsculas; también en una carrera |
| `EQUIPMENTTYPE_NOT_FOUND` | `404` | `/equipment-types/{id}` | El tipo de equipamiento no existe |
| `EQUIPMENTTYPE_CONFLICT` | `409` | `POST /equipment-types`, `PUT /equipment-types/{id}` | Otro tipo de equipamiento tiene ese código; también en una carrera |
| `EQUIPMENTTYPE_CODE_LOCKED` | `422` | `PUT /equipment-types/{id}` | Se corrige el código de `GPS` o de `PANTALLA`, que leen las reglas de habilitación y quedan fijos desde la semilla (D-74). El recurso lo anuncia con `codeLocked` (§4.2) |
| `VALIDATION_ERROR` | `400` | Las altas y las ediciones | El formato: un nombre o un código vacío o más largo que su columna, un combustible fuera de la lista (en minúsculas: `gasolina`, `gnv`, `glp`, `diesel`, `electrico`, `hibrido`), un cilindraje que no es un entero mayor que cero o que lleva un modelo eléctrico, un tipo de equipamiento que no aplica a ningún tipo de vehículo |

- Un combustible vacío se acepta: el modelo nuevo no lo propone (D-109).
- `fleet` responde además `VEHICLEMODEL_INACTIVE`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE` y `COLOR_INACTIVE` al registrar una unidad (abajo); los códigos son los de `vehiclecatalog`, porque nombran sus agregados.

**Los códigos de `fleet` que cambian con la auditoría** (registrado el 20/10/2026; diccionario D-108, D-110, D-113, D-114). Los demás códigos de `fleet` están en los flujos de [contextos/fleet.md](contextos/fleet.md), con el texto que ve cada persona.

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `VEHICLEMODEL_INACTIVE`, `VEHICLEBRAND_INACTIVE`, `VEHICLETYPE_INACTIVE` | `422` | `POST /vehicles` | El modelo de la unidad, su marca o su tipo de vehículo están inactivos (D-108). Solo el alta los revisa: el modelo de una unidad no cambia después del alta, y `PUT /vehicles/{id}` no lo recibe (D-84, FLT-R08). `ExternalVehicleCatalogService` los traduce de lo que responde la fachada de `vehiclecatalog`. `VEHICLEBRAND_INACTIVE` y `VEHICLETYPE_INACTIVE` son nuevos |
| `STATUSCHANGEREASON_AUTOMATIC_NOT_SELECTABLE` (nuevo) | `422` | `POST /vehicles/{id}/status-changes` | El motivo elegido es un motivo automático: lo recorre solo el regreso automático (diccionario §8.3, D-110). Tampoco figura en `allowedActions.walkableReasonIds` |
| `STATUSCHANGEREASON_AUTOMATIC_LOCKED` (nuevo) | `422` | `PUT /status-change-reasons/{id}`, `PATCH /status-change-reasons/{id}/status` | El motivo es automático: lo mantiene el sistema, y no se edita ni se cierra por la API (D-110) |
| `STATUSCHANGEREASON_CODE_CONFLICT` | `409` | `POST /status-change-reasons`, `PUT /status-change-reasons/{id}` y `PATCH /status-change-reasons/{id}/status` de un camino abierto que mueve sola la unidad (`mover_sola`) | Además de su caso de siempre, un código repetido en su camino: el camino de regreso ya tiene un motivo `REGRESO_AUTOMATICO` que declaró una persona, no el sistema. El camino no se guarda, y nada de la operación queda escrito (D-110). Responde lo mismo cuando esa persona lo declara mientras tanto, en una carrera; dos guardados que crean a la vez el mismo motivo automático no fallan: el segundo reutiliza el del primero |
| `VALIDATION_ERROR` | `400` | `GET /status-change-reasons` | `fromStatusId` y `origin=registration` juntos, o un `origin` que no sea `registration` (D-113, §6.2) |

**Los códigos de los fondos y las cesiones** (registrado el 26/10/2026, paso 10 parcial; diccionario §12, D-133, D-134).

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `FUND_NOT_FOUND` | `404` | `/funds/{id}`; `POST /vehicles/{id}/fund-assignments` | El fondo no existe o quien llama no tiene `fondos.ver` en su empresa (al ceder, tampoco `fondos.registrar_cesion`) |
| `FUND_CONFLICT` | `409` | `POST /funds`, `PUT /funds/{id}` con `code` | Otro fondo de la empresa ya tiene ese código; también si dos pedidos corren a la vez y la base frena al segundo. Sin campo: el fondo tiene una sola regla de unicidad |
| `FUND_CODE_LOCKED` | `422` | `PUT /funds/{id}` con `code` | Ya se cedió una unidad al fondo: su código queda fijo (diccionario §2.4, §12.1). El mismo código no es corrección, y el resto de la edición pasa |
| `FUND_INACTIVE` | `422` | `POST /vehicles/{id}/fund-assignments` | El fondo elegido está inactivo |
| `VEHICLEFUNDASSIGNMENT_NOT_FOUND` | `404` | `POST /fund-assignments/{id}/closure` | La cesión no existe o quien llama no ve su unidad con `stock.ver` (marca incluida, D-102): la negativa no dice que existe |
| `VEHICLEFUNDASSIGNMENT_CONFLICT` | `409` | El alta y el cierre de una cesión | El período se cruza con otra cesión de la unidad, también en una carrera (`23P01`). Sin campo |
| `VEHICLEFUNDASSIGNMENT_COMPANY_MISMATCH` | `422` | `POST /vehicles/{id}/fund-assignments` | Quien llama ve la unidad y el fondo, pero son de empresas distintas |
| `VEHICLEFUNDASSIGNMENT_ALREADY_CLOSED` | `422` | `POST /fund-assignments/{id}/closure` | La cesión ya estaba cerrada, también por otro cierre que confirmó mientras este esperaba la unidad |
| `VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START` | `422` | `POST /fund-assignments/{id}/closure` | La fecha de fin no es posterior a la de inicio |

El orden de las revisiones:
- `POST /funds`: la empresa (`404 COMPANY_NOT_FOUND` si no existe o no se alcanza), `fondos.crear_fondos` en toda la empresa y sin marca (`403`), `fondos.ver` en esa empresa (`403`: quien crea un fondo tiene que poder verlo), `COMPANY_INACTIVE`, el código y el tipo (`400`) y `FUND_CONFLICT`.
- `PUT /funds/{id}`: el fondo (`404`), `fondos.editar_fondos` en toda la empresa y sin marca (`403`), el formato (`400`), y `FUND_CODE_LOCKED` y `FUND_CONFLICT` si trae otro `code`.
- `PUT /purchase-order-types/{id}`: el tipo (`404 PURCHASEORDERTYPE_NOT_FOUND` si quien llama no lo lee con `compras.ver`), `compras.editar_plantillas` en toda la empresa y sin marca (`403`), el formato del nombre y del código (`400`), y `PURCHASEORDERTYPE_CODE_LOCKED` y `PURCHASEORDERTYPE_CODE_CONFLICT` (`409`) si trae otro `code`. El tipo se bloquea antes de revisar si una plantilla lo usa.
- `PATCH /funds/{id}/status`: el fondo (`404`), `fondos.desactivar_fondos` en toda la empresa y sin marca (`403`) y `COMPANY_INACTIVE` al reactivar.
- Escribir exige leer, como en compras (diccionario §5.6): editar o cambiar el estado de un fondo que quien llama no lee con `fondos.ver` responde `404 FUND_NOT_FOUND` antes de escribir, y crear uno sin `fondos.ver` en la empresa responde `403`. La respuesta de cada escritura es el fondo guardado, sin volver a filtrar por `fondos.ver`.
- `POST /vehicles/{id}/fund-assignments`: la unidad (`404 VEHICLE_NOT_FOUND` si `stock.ver` no la ve), `fondos.registrar_cesion` en toda la empresa o en la marca de la unidad (`403`), el fondo (`404 FUND_NOT_FOUND` si quien llama no tiene `fondos.ver` ni `fondos.registrar_cesion` en su empresa), `VEHICLEFUNDASSIGNMENT_COMPANY_MISMATCH`, `FUND_INACTIVE` y `VEHICLEFUNDASSIGNMENT_CONFLICT`. La unidad se bloquea para actualizar y el fondo en modo compartido.
- `POST /fund-assignments/{id}/closure`: la cesión y su unidad (`404 VEHICLEFUNDASSIGNMENT_NOT_FOUND`), `fondos.registrar_cesion` como al ceder (`403`), y después `VEHICLEFUNDASSIGNMENT_ALREADY_CLOSED`, `VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START` y `VEHICLEFUNDASSIGNMENT_CONFLICT`.
- `GET /vehicles/{id}/fund-assignments` pide `stock.ver` donde se ve la unidad: `fondos.ver` solo no basta (`403`), y una unidad que no se ve responde `404` antes de cualquier `403`. Así un `403` o un `422` no dicen que existe algo que quien llama no ve.

`FUND_CODE_TAKEN`, `VEHICLE_FUND_ASSIGNMENT_OVERLAP`, `VEHICLE_FUND_COMPANY_MISMATCH`, `ASSIGNMENT_ALREADY_CLOSED` e `INVALID_ASSIGNMENT_PERIOD` nunca se publicaron; los reemplazan los de esta tabla (D-114).

**Un solo prefijo por agregado** (D-114). Desde el 20/10/2026 los códigos de los estados y de los caminos usan el prefijo de su `_NOT_FOUND`, que ya eran `VEHICLESTATUS_NOT_FOUND` y `STATUSCHANGEREASON_NOT_FOUND`. Cambian estos trece, con su clave `error.<código-en-kebab>.message` en los dos idiomas; ningún otro código cambia, tampoco `STATUS_TRANSITION_NOT_ALLOWED`:

| Antes | Desde el 20/10/2026 |
|---|---|
| `VEHICLE_STATUS_CODE_CONFLICT` | `VEHICLESTATUS_CODE_CONFLICT` |
| `VEHICLE_STATUS_INITIAL_CONFLICT` | `VEHICLESTATUS_INITIAL_CONFLICT` |
| `VEHICLE_STATUS_INITIAL_MISSING` | `VEHICLESTATUS_INITIAL_MISSING` |
| `VEHICLE_STATUS_INACTIVE` | `VEHICLESTATUS_INACTIVE` |
| `VEHICLE_STATUS_INITIAL_RETIREMENT_NOT_ALLOWED` | `VEHICLESTATUS_INITIAL_RETIREMENT_NOT_ALLOWED` |
| `VEHICLE_STATUS_AUTO_TARGET_RETIREMENT_NOT_ALLOWED` | `VEHICLESTATUS_AUTO_TARGET_RETIREMENT_NOT_ALLOWED` |
| `STATUS_CHANGE_REASON_CODE_CONFLICT` | `STATUSCHANGEREASON_CODE_CONFLICT` |
| `STATUS_CHANGE_REASON_TARGET_MISMATCH` | `STATUSCHANGEREASON_TARGET_MISMATCH` |
| `STATUS_CHANGE_REASON_REGISTRATION_MISSING` | `STATUSCHANGEREASON_REGISTRATION_MISSING` |
| `STATUS_CHANGE_REASON_REGISTRATION_CONFLICT` | `STATUSCHANGEREASON_REGISTRATION_CONFLICT` |
| `STATUS_CHANGE_REASON_REGISTRATION_PERMISSION_NOT_ALLOWED` | `STATUSCHANGEREASON_REGISTRATION_PERMISSION_NOT_ALLOWED` |
| `STATUS_CHANGE_REASON_REGISTRATION_TARGET_NOT_INITIAL` | `STATUSCHANGEREASON_REGISTRATION_TARGET_NOT_INITIAL` |
| `STATUS_CHANGE_REASON_AUTO_TARGET_INACTIVE` | `STATUSCHANGEREASON_AUTO_TARGET_INACTIVE` |

El cambio es incompatible y se hizo en los dos repositorios a la vez (§1): el frontend elige sus textos por los códigos nuevos.

**Los códigos de los documentos** (registrado el 20/10/2026; D-106, D-125, §8). Los responde el módulo `documents`, con su clave en los dos idiomas; el `410` y el `503`, que no tienen un sufijo que dé su estado, los sigue nombrando `ErrorResponseAssembler`, de `shared`.

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `DOCUMENT_NOT_FOUND` | `404` | `GET /documents/{id}`, `GET /documents/{id}/file` | El documento no existe, o es de otro alcance que el de quien pregunta: las dos respuestas son iguales. `/file` responde lo mismo mientras el documento no está listo |
| `DOCUMENT_EXPIRED` | `410` | `GET /documents/{id}`, `GET /documents/{id}/file` | El documento venció y su archivo ya no se entrega. El frontend repite una vez el pedido original, que genera otro |
| `DOCUMENTS_BUSY` | `503` | El `GET` de una lista con el `Accept` de Excel | No hubo un lugar libre para generar dentro de `documents.permit-wait` (§8.3). El documento queda fallido, y el pedido se puede repetir |

**Los códigos de las cuentas y de la salida y el regreso** (registrado el 16/10/2026). Cada causa que la pantalla tiene que distinguir lleva su código, y el frontend tiene un mensaje para cada uno. Todos responden `422` salvo que se indique otro estado. "Los cuatro `403`" son `ACCESS_DENIED`, `OWN_ACCOUNT_FORBIDDEN`, `ACTOR_EMPLOYEE_REQUIRED` y `ACCOUNT_MANAGED_BY_SUPERADMIN`.

| Operación | Códigos |
|---|---|
| `POST /authentication/sign-in` | `UNAUTHORIZED` `401` (también para `SYSTEM`), `ACCOUNT_SUSPENDED` `403`, `VALIDATION_ERROR` `400` |
| `POST /access-grants` | `EMPLOYEE_NOT_FOUND` `404`; los cuatro `403`; `EMPLOYEE_INACTIVE`; `EMPLOYEE_ACCOUNT_CONFLICT` `409` (el colaborador ya tiene cuenta); `EMPLOYEE_ACCOUNT_IN_COMPANY_CONFLICT` `409` (la cuenta ya está vinculada a otro colaborador de esa empresa); `EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH` (la cuenta existente no pertenece a ningún colaborador con la misma sigla y el mismo número de documento, o a ninguno: D-82, §14.15); `BUSINESS_RULE_VIOLATION` (la cuenta `SYSTEM`); `VALIDATION_ERROR` `400` |
| `POST /employees/{id}/account-unlinking` | `EMPLOYEE_NOT_FOUND` `404`; `ACCESS_DENIED` (sin `seguridad.desvincular_cuentas` de alcance sistema: no es superadministrador), `ACTOR_EMPLOYEE_REQUIRED`, `OWN_ACCOUNT_FORBIDDEN` (la cuenta es la propia) y, desde la segunda ronda, `ACCOUNT_MANAGED_BY_SUPERADMIN` `403`; `EMPLOYEE_ACCOUNT_MISSING` (el colaborador no tiene cuenta que desvincular); `LAST_ACTIVE_SUPERADMIN`; `ACCESS_SUSPENSION_CONFLICT` `409` (un bloqueo simultáneo tomó el lugar de la `baja`) |
| `POST /users/{id}/role-assignments` | `USER_NOT_FOUND`, `ROLE_NOT_FOUND`, `COMPANY_NOT_FOUND`, `BRANCH_NOT_FOUND`, `COMMERCIALBRAND_NOT_FOUND` `404`; los cuatro `403` —quien no puede otorgar ese rol, como el administrador de accesos con un rol reservado o cualquiera con uno cuyos permisos no tiene donde el rol actúa, recibe `ACCESS_DENIED`; `ACCOUNT_MANAGED_BY_SUPERADMIN` es solo por una cuenta de destino protegida (D-121, D-122)—; `ROLE_ASSIGNMENT_CONFLICT` `409`; `ROLE_INACTIVE` (D-97); `ROLE_ASSIGNMENT_SCOPE_MISMATCH`; `ROLE_ASSIGNMENT_COMPANY_MISMATCH` (un rol de una empresa en otra, D-98); `ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`; `ROLE_ASSIGNMENT_EMPLOYEE_REQUIRED`; `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE` (la cuenta juntaría un par incompatible entre sus roles vigentes, D-119, con los pares en `context.incompatiblePairs`); `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD` (un rol reservado a una cuenta con contraseña provisional, D-121); `COMPANY_INACTIVE`, `BRANCH_INACTIVE` y `COMMERCIALBRAND_INACTIVE` (el alcance está inactivo, D-84); `BUSINESS_RULE_VIOLATION` (la cuenta `SYSTEM`, una caducidad que no es posterior); `VALIDATION_ERROR` `400` |
| `POST /role-assignments/{id}/revocation` | `ROLE_ASSIGNMENT_NOT_FOUND` `404`; los cuatro `403`; `ROLE_ASSIGNMENT_INACTIVE`; `LAST_ACTIVE_SUPERADMIN` |
| `POST /users/{id}/suspensions` | `USER_NOT_FOUND` `404`; los cuatro `403`; `ACCESS_SUSPENSION_CONFLICT` `409`; `ACCESS_SUSPENSION_START_IN_PAST`; `LAST_ACTIVE_SUPERADMIN`; `BUSINESS_RULE_VIOLATION` (fin antes del inicio, la cuenta `SYSTEM`); `VALIDATION_ERROR` `400`, también para el motivo `baja` |
| `POST /suspensions/{id}/closure` | `ACCESS_SUSPENSION_NOT_FOUND` `404`; los cuatro `403`; `ACCESS_SUSPENSION_BAJA_RESERVED`; `ACCESS_SUSPENSION_CLOSED` |
| `POST /users/{id}/password-reset` | `USER_NOT_FOUND` `404`; los cuatro `403`; `BUSINESS_RULE_VIOLATION` (la cuenta `SYSTEM`) |
| `POST /employees/{id}/termination` | `EMPLOYEE_NOT_FOUND` `404`; `ACCESS_DENIED`, `OWN_ACCOUNT_FORBIDDEN` y `ACTOR_EMPLOYEE_REQUIRED` (quien opera no está vinculado a un colaborador activo, también un superadministrador, D-89) `403`; `EMPLOYEE_ALREADY_TERMINATED`; `EMPLOYEE_TERMINATION_BEFORE_HIRE`; `EMPLOYEE_TERMINATION_DATE_IN_FUTURE`; `LAST_ACTIVE_SUPERADMIN`; `ACCESS_SUSPENSION_CONFLICT` `409` (un cruce que la base detecta en una carrera); `BUSINESS_RULE_VIOLATION` (otra regla de fechas); `VALIDATION_ERROR` `400` |
| `POST /employees/{id}/reactivation` | `EMPLOYEE_NOT_FOUND` `404`; `ACCESS_DENIED`, `OWN_ACCOUNT_FORBIDDEN` y `ACTOR_EMPLOYEE_REQUIRED` (como en la salida, D-89) `403`; `EMPLOYEE_ALREADY_ACTIVE`; `EMPLOYEE_REHIRE_BEFORE_TERMINATION`; `COMPANY_INACTIVE` y `BRANCH_INACTIVE` (la empresa o la sede del colaborador están inactivas, D-94); `EMPLOYEE_REHIRE_DATE_IN_FUTURE`; `BUSINESS_RULE_VIOLATION` (otra regla de fechas); `VALIDATION_ERROR` `400`. `organization` revisa en ese orden: el `404`, `EMPLOYEE_ALREADY_ACTIVE`, `EMPLOYEE_REHIRE_BEFORE_TERMINATION`, `COMPANY_INACTIVE`, `BRANCH_INACTIVE` y `EMPLOYEE_REHIRE_DATE_IN_FUTURE`; si alguna falla, la cuenta sigue cerrada |
| `GET /companies/{companyId}/employees` —la página, la lista sin página y el filtro de sede— y `GET /employees/{id}` | `ACCESS_DENIED` `403` sin `configuracion.ver_colaboradores`; `COMPANY_NOT_FOUND` o `EMPLOYEE_NOT_FOUND` `404` fuera del alcance, y también en una empresa que la cuenta alcanza con un rol que no trae ese permiso (D-99). Con el permiso, la fila llega sin `documentNumber`, `phone` ni `photoPath` donde la cuenta no tiene `configuracion.ver_datos_personales_colaboradores` (D-105, §4.1). Hasta el paso 3b respondía `403` a quien veía la empresa sin administrarla (D-93, registrado el 18/10/2026) |
| La exportación de `GET /companies/{companyId}/employees` | `ACCESS_DENIED` `403` sin `configuracion.ver_datos_personales_colaboradores`, porque el archivo lleva el documento de cada colaborador (D-105): la ruta lo exige antes de buscar la empresa, así que quien no lo tiene en ninguna recibe `403` aunque la empresa no exista; quien lo tiene en otra recibe `COMPANY_NOT_FOUND` `404` por una fuera de su alcance y `403` por una alcanzada donde no actúa (D-99) |
| `GET /roles`, `GET /roles/{id}`, `GET /permissions` y las escrituras de `/roles` | Arriba, "Los códigos de los roles y de la matriz de permisos" |
| `GET /users`, con o sin `companyId`, y su exportación | `ACCESS_DENIED` `403`; `COMPANY_NOT_FOUND` `404` si `companyId` no existe o está fuera del alcance de quien llama |

Los códigos que dicen que la pantalla quedó vieja —entre otros `ACCESS_SUSPENSION_CLOSED`, `ROLE_ASSIGNMENT_INACTIVE`, `ACCOUNT_MANAGED_BY_SUPERADMIN`, `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`, `EMPLOYEE_ALREADY_TERMINATED` y `EMPLOYEE_ALREADY_ACTIVE`— hacen que el frontend vuelva a leer la lista o el diálogo afectado. La salida y el regreso responden `{id, isActive, hireDate, terminationDate}`.

---

## 8. Documentos

Un documento es un archivo que entra o sale del sistema: el que se sube —un documento del vehículo, una carga— y el que el sistema genera para que alguien lo descargue —una lista exportada a Excel, el PDF de una orden de compra—. Todo documento que sale lo genera el backend, y el frontend solo lo descarga (D-106).

**El módulo `documents`** (D-125, construido en la primera fase de la segunda ronda de la auditoría e integrado en `develop`, merge `e245186`). Los documentos que genera el sistema son de un módulo propio, `ky.millennium.mobility.erp.documents`, dueño de `generated_documents` y de las rutas `/documents/{id}` y `/documents/{id}/file`; hasta D-125 eran parte de `shared`. Los contextos que exportan —`location`, `iam`, `organization`, `vehiclecatalog` y `fleet`— lo usan solo por su contrato público, `documents::acl`: el paquete `ky.millennium.mobility.erp.documents.interfaces.acl`, con `@NamedInterface("acl")`, que publica estos siete tipos y ningún otro:

| Tipo | Qué es | Antes, en `shared` |
|---|---|---|
| `DocumentService` | La fachada: `request(DocumentRequest, DocumentContent)` devuelve un `Result<DocumentSeeOther, ApplicationError>`, nunca el agregado. La implementa `DocumentServiceImpl`, en `documents/application/acl`, que traduce el pedido al comando del dominio, `RequestDocumentCommand`, con `RequestDocumentCommandFromRequestAssembler` | `DocumentCommandService`, que sigue dentro del módulo y ya no recibe el pedido del contrato |
| `DocumentRequest` | El pedido: el tipo, el formato y los criterios, armado como `DocumentRequest.spreadsheet(kind).text("q", q).with("isActive", isActive).sortedAs(rows)` | `RequestDocumentCommand` y `DocumentCriteria` |
| `DocumentSeeOther` | Dónde lee el navegador el documento: su identificador público y su `location()`. El controlador responde `DocumentSeeOther.toResponse(result)`: el `303 See Other` a `/documents/{id}`, o el error como JSON | `DocumentResourceFromEntityAssembler.toSeeOtherFromResult` |
| `DocumentFormat`, `DocumentKind`, `DocumentScopes`, `DocumentContent` | El formato, la declaración de cada tipo de documento, el alcance efectivo y el contenido que escribe el generador | Los mismos nombres. `DocumentFormat` del contrato es solo `XLSX` o `PDF`: el dominio tiene su gemelo, en `domain/model/valueobjects`, con el tipo de contenido y la extensión, y la fachada traduce uno al otro, para que el dominio no importe nada de `interfaces` |

El módulo no abre su dominio ni su API, y no depende de ningún contexto (`allowedDependencies = {}`). Se quedan en `shared` lo genérico: `SpreadsheetExporter` y `SpreadsheetColumn`, que escriben una hoja de cálculo sin saber de documentos, y el `410` y el `503` de `ErrorResponseAssembler` (§7.2).

| Caso | Petición | Respuesta |
|---|---|---|
| Subir un documento o una carga | `POST` en `multipart/form-data`: el archivo en `file` y los datos en `data`, como JSON. La ruta de las cargas de flota depende de §14.18 | `201` con el recurso |
| Descargar un archivo subido | `GET .../file` o `.../error-report` | El archivo con su `Content-Type` y `Content-Disposition: attachment; filename="…"` |
| Exportar una lista | El mismo `GET` de la lista con el `Accept` de Excel (§8.1) | `303 See Other` con `Location: /api/v1/documents/{id}`; el archivo se descarga de `/documents/{id}/file` |
| Generar un PDF | El `GET .../pdf` del recurso, con `Accept: application/pdf` (§8.2) | El mismo flujo. Construido en la orden de compra: el borrador sigue este flujo, y una orden enviada responde `200` con su hoja guardada (§8.2) |
| Exportar un reporte | `GET /reports/...` con el `Accept` de su formato | El mismo flujo |

Los archivos que se suben se guardan fuera de la base. Los documentos que genera el sistema, en la base o en DigitalOcean Spaces, nunca en la memoria ni en el disco del proceso (§8.3). La API nunca devuelve `file_path` ni la llave de un objeto en Spaces.

### 8.1 Exportar a Excel

El archivo lo arma siempre el backend, con fastexcel (`org.dhatim:fastexcel`: escribe en streaming, pesa poco y no arrastra Apache POI). El frontend nunca arma un archivo: lo pide, espera a que esté listo y lo descarga con el nombre que manda el backend.

**Qué lleva el archivo**

- Una lista se exporta desde **su propio endpoint**. El `GET` que devuelve el JSON, pedido con el `Accept` de Excel, genera el archivo con la misma búsqueda (`q`), los mismos filtros y el mismo orden (`sort`), y **todas** las filas que coinciden: ignora `page` y `size` (§6.2). No hay rutas `/export` aparte. Todo catálogo y toda lista principal se exporta así (D-116).
- Cada columna es una clave de `messages.properties` para el encabezado y una función que lee la fila (`SpreadsheetColumn`). Encabezados y textos salen en el idioma de la petición (`Accept-Language`), igual que los mensajes de error, y en español si la petición no nombra un idioma que la API hable (§7.1).
- Los valores conservan su tipo, para que la hoja sume, filtre y ordene: un número es número; una fecha, fecha; un momento, fecha y hora; un booleano, "Sí" o "No". Un texto que se traduce, como un estado o el tipo de una ubicación («Sede propia», «Own site»), se devuelve como `SpreadsheetColumn.Message`.
- Las fechas y los momentos se escriben en ISO, nunca en el formato de pantalla (diccionario §2.6, D-40, D-49). Una fecha es una celda de fecha con la máscara `AAAA-MM-DD`; un momento, una celda de fecha y hora con la máscara `AAAA-MM-DD hh:mm:ss`. Una celda de Excel no guarda la zona, así que todos los momentos de un archivo van en **una sola zona, nombrada en el encabezado** de cada columna de momentos: «Último acceso (America/Lima)». Nunca se mezclan zonas en una columna.
- Esa zona sale de la regla de "hoy" (diccionario §2.6) aplicada al alcance de la exportación: la sede si la lista es de una sede; si no, el país de la empresa si es de una empresa; si no, `America/Lima`. `SpreadsheetExporter` recibe la zona como parámetro y usa `America/Lima` si no se la pasan. La zona entra en la llave del documento (§8.3).
- Una hoja con el nombre de la lista y la fila de encabezados fija y en negrita. El archivo se llama `<lista>-<fecha>.xlsx`: el nombre base lo declara el tipo de documento, sin tildes, para que todos los navegadores lo guarden igual, y la fecha es la de hoy en Lima al descargarlo. La zona del nombre del archivo es independiente de la zona de los momentos.
- El `id` interno y el `public_id` nunca se exportan (§4.2; diccionario §2.4, D-51). Las columnas muestran lo que lee una persona: nombres y códigos de negocio, no identificadores. La única excepción del diccionario es un archivo pensado para editarse y volver a subirse a actualizar registros, que lleva además el `public_id` como llave estable; hoy no hay ninguno.
- Una referencia cuyo nombre puede repetirse se exporta también por su código. Dos sedes de una empresa pueden llamarse igual (diccionario §6.4), así que desde el 18/10/2026 la exportación de colaboradores lleva Nombre completo, Documento, Puesto, Código de sede, Sede, Fecha de ingreso y Estado, y la de ubicaciones, Código, Nombre de ubicación, Tipo de ubicación, Código de sede (de una ubicación propia), Instalación / Dirección y Estado.

**El flujo** (desde el 20/10/2026, D-106)

1. El frontend pide la lista con `Accept: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/json;q=0.9`. Spring elige el método de Excel del controlador (`@GetMapping(produces = SpreadsheetExporter.MEDIA_TYPE)`), que recibe los mismos parámetros que la lista y delega en `DocumentService`, del módulo `documents` (D-125).
2. El servicio calcula la llave del pedido y la reclama (§8.3). Si la gana, genera el archivo en el mismo hilo de la petición, para que conserve la sesión, el alcance y Row Level Security: `SpreadsheetExporter` lo escribe a un archivo temporal, el servicio lo guarda en el almacenamiento y marca el documento listo. Si no la gana, reutiliza el documento listo o espera al que se está generando.
3. Responde `303 See Other` con `Location: /api/v1/documents/{id}`. El navegador sigue la redirección solo, con el mismo `Accept`, y `GET /documents/{id}` responde JSON (`DocumentResource`):

   ```json
   {
     "id": "019a0b3c-5b1c-7a3e-9d4f-0a1b2c3d4e5f",
     "status": "ready",
     "fileName": "monedas-2026-10-20.xlsx",
     "downloadUrl": "/api/v1/documents/019a0b3c-5b1c-7a3e-9d4f-0a1b2c3d4e5f/file",
     "expiresAt": "2026-10-20T15:40:00Z"
   }
   ```

   - `status` es `generating`, `ready` o `failed`.
   - `fileName` es `<nombre base>-<fecha de hoy en Lima>.<extensión>`.
   - `downloadUrl` es `/api/v1/documents/{id}/file` cuando el documento está listo, y `null` mientras no lo está.
   - `expiresAt` es hasta cuándo se reutiliza el documento (`valid_until`), también solo cuando está listo; si no, `null`.
   - Un documento vencido no responde este cuerpo: responde `410 DOCUMENT_EXPIRED`.
4. Mientras `generating`, el frontend vuelve a consultar `GET /documents/{id}` cada segundo, hasta 120 segundos. En `ready`, descarga navegando a `downloadUrl` con un enlace oculto, sin pasar el archivo por la memoria de la página. Un `410` repite una vez el pedido original.
5. `GET /documents/{id}/file` vuelve a comprobar el alcance y entrega el archivo: desde la base, con `Content-Disposition: attachment; filename="…"`; desde Spaces, con un `302` a una URL firmada que vale 5 minutos y lleva `response-content-disposition=attachment`.

- `GET /documents/{id}` y `GET /documents/{id}/file` exigen sesión y nada más: la guarda es el alcance (§9.2). Recalculan el alcance efectivo del tipo de documento para quien pregunta y, si no coincide con el del documento, responden `404 DOCUMENT_NOT_FOUND`, como si no existiera. Uno vencido responde `410 DOCUMENT_EXPIRED`.
- Un error responde JSON aunque se haya pedido Excel (§7): el país que no existe es `404` y la falta de permiso, `403`. Si la generación falla, el documento queda fallido y el error sale como cualquier otro. Sin un lugar libre para generar, `503 DOCUMENTS_BUSY`.
- Una prueba demuestra que Spring elige el método de Excel con ese `Accept`, y que `/documents/{id}` responde JSON con el mismo `Accept` después de la redirección.
- Del lado del frontend, el contrato de los stores no cambia: `exportX()`, `exporting` y, en un fallo, `data-manager.export-error`. `exportSpreadsheet()` de `BaseApiEndpoint` manda el `Accept`, lee el `DocumentResource` y descarga.

**Casos**

| Caso | Ejemplo | Qué se exporta |
|---|---|---|
| Catálogo completo | `GET /currencies`, `/countries`, `/users` | Lo que coincide con la búsqueda y los filtros de la pantalla (`isActive` en monedas), en su orden |
| Lista de un registro padre | `GET /countries/{countryId}/tax-id-types` | Los del padre; si el padre no existe, `404` |
| Lista filtrada en el servidor | `GET /countries/{countryId}/geo-divisions?q=lima&level=3&isActive=true&sort=name,asc`, `/companies?countryId=…` | Lo mismo que la lista con esos parámetros, sin página: el frontend manda los de la pantalla |
| Columnas de otro contexto | Empresas: país, documento tributario y moneda | Los nombres los resuelve la consulta por la fachada del dueño (`ExternalLocationService`), en lote: una llamada por tipo de referencia con todos los identificadores, nunca una por fila ni con un `join` |
| Reporte que cruza contextos | El exportable de stock de 52 columnas | Lo arma la capa de lectura en `/reports/...` (mapa de contextos §7), con el mismo `SpreadsheetExporter` y el mismo flujo. Es otro documento que el Inventario (D-116) |

Pasan al flujo del 20/10/2026 las exportaciones que ya existían: monedas, países, divisiones y tipos de documento (`location`); empresas, sedes, marcas comerciales, ubicaciones y colaboradores (`organization`); y cuentas (`iam`). Sus pruebas siguen la redirección y descargan de `/documents/{id}/file`. En la segunda fase de la auditoría llegaron, sobre la plataforma ya integrada (D-116): los cinco catálogos de flota (`GET /vehicle-types`, `/vehicle-brands`, `/colors` y `/equipment-types`), los modelos de una marca (`GET /vehicle-brands/{id}/vehicle-models`) y el Inventario (`GET /vehicles`), con los nombres de sus referencias resueltos en lote por las fachadas, en una lectura por clase de referencia. Sus columnas:

| Lista | `kind` | Columnas |
|---|---|---|
| Tipos de vehículo | `vehicle-types` | Código, Tipo de vehículo, Nombre comercial, Estado |
| Marcas de vehículo | `vehicle-brands` | Marca, Estado |
| Modelos de una marca | `vehicle-models` | Marca, Modelo, Código de tipo de vehículo, Tipo de vehículo, Cilindraje (cc), Combustible (traducido), Pantalla de fábrica, Estado |
| Colores | `vehicle-colors` | Color, Estado |
| Tipos de equipamiento | `equipment-types` | Código, Equipamiento, Bloquea la disponibilidad, Exige número de serie, Aplica a («Nombre (CÓDIGO)», …), Estado |
| Inventario | `vehicles` | Placa, VIN, Número de motor, Año del modelo, Marca, Modelo, Código de tipo de vehículo, Tipo de vehículo, Color, Empresa (nombre comercial), Marca comercial, Código de ubicación, Ubicación, Código de estado, Estado, Kilometraje (km), N.° anterior, Registrada (un momento, con su zona en el encabezado) |

El tipo de vehículo, la ubicación y el estado van también por su código, porque sus nombres pueden repetirse. Los cinco catálogos tienen el alcance de un catálogo general, tener `catalogos.ver`; el Inventario, cada asignación de `stock.ver` con su empresa, su sede y su marca. La zona de los momentos del Inventario es la del país de la empresa por la que filtra la exportación o, si no filtra, la de la única empresa que ve quien pide; si ve varias, `America/Lima`.

Con el cierre de la auditoría llegaron los proveedores (`GET /suppliers`) y los fondos (`GET /funds`), el 26/10/2026 (D-116, D-134):

| Lista | `kind` | Columnas |
|---|---|---|
| Proveedores | `suppliers` | Empresa, Tipo de documento, Documento tributario, Razón social, Nombre comercial, Domicilio (en una línea), Estado. Sin los contactos, cuya lista (`GET /suppliers/{id}/contacts`) tampoco se exporta (decisión del responsable del 29/10/2026) |
| Fondos | `funds` | Código, Nombre, Tipo (traducido: Fideicomiso, Fondo propio, Banco), Empresa (nombre comercial), Estado |

Cada una tiene el alcance de cada asignación de su permiso de lectura con su empresa —`compras.ver` en los proveedores, `fondos.ver` en los fondos—, exactamente el permiso de su lista: dos cuentas que leen las mismas filas comparten el archivo, escriban lo que escriban, y una empresa donde quien pide solo escribe no entra. No tienen momentos, así que no nombran zona. La versión de los proveedores es la de `suppliers` más la de lo que `organization` y `location` resuelven para ellos; la de los fondos, la de `funds` (`FleetDataVersions.funds()`) más la de `organization`. La lista de órdenes de compra y la de plantillas todavía no se exportan: quedan para el paso de reportes (D-116, D-134).

El 29/10/2026 llegó la lista de los modelos de todas las marcas (`GET /vehicle-models`, §6.2), que la pantalla Marcas y modelos exporta desde entonces:

| Lista | `kind` | Columnas |
|---|---|---|
| Modelos de todas las marcas | `all-vehicle-models` | Las de los modelos de una marca, en el mismo orden (`VehicleModelSpreadsheetColumns`); el archivo se llama igual, `modelos-de-vehiculo-<fecha>.xlsx` |

Es un tipo propio, con el alcance (`catalogos.ver`) y la versión de los datos de los modelos de una marca: las dos listas reciben los mismos criterios, pero solo esta ordena por `vehicleBrandName`, así que un pedido igual a cada una nunca recibe el archivo de la otra. Una marca o un tipo que no existen dan un archivo con solo sus encabezados. La exportación de las marcas (`GET /vehicle-brands` con el `Accept` de Excel, `vehicle-brands`) sigue en la API, aunque la pantalla ya no la ofrece: las marcas se mantienen en el diálogo Gestionar marcas.

**Lo que no hace**

- No exporta solo la página visible: siempre todo lo que coincide con la búsqueda y los filtros.
- No responde el archivo en el mismo pedido: lo entrega siempre `/documents/{id}/file`, para que un documento ya generado se reutilice.
- No genera CSV ni otros formatos. Si hace falta uno, se agrega otro `produces` al mismo endpoint, con el mismo flujo.

**Cómo se agrega a una lista**

1. Backend: en el controlador, el método `@GetMapping(produces = SpreadsheetExporter.MEDIA_TYPE)` con los mismos `@RequestParam` de la lista y `PageParams`, que arma un `DocumentRequest`, lo pide a `DocumentService` y responde su `DocumentSeeOther` (D-125). El contexto declara `"documents::acl"` en sus `allowedDependencies`. El tipo de documento se declara en el registro de su contexto (`<Contexto>DocumentKinds`): su `kind`, su nombre de archivo base, su función de alcance y de visibilidad, su versión de los datos —que pide a su `<Contexto>DataVersions`, sin nombrar tablas en `application`— y su generador, que lee todas las filas con `pageParams.toAllRowsQuery()` y escribe el archivo con los mismos criterios. Las claves `export.<lista>.*` van en `messages.properties` y `messages_es.properties`. Una prueba sigue la redirección, descarga de `/documents/{id}/file` y lee el archivo con `fastexcel-reader`.
2. Frontend: el endpoint llama a `exportSpreadsheet()` de `BaseApiEndpoint` con los filtros y `listQueryParams()` de la búsqueda y el orden; la API del módulo lo expone; el store tiene `exporting` y `export<Lista>()`, que lee la consulta de su `PagedList` y en un fallo deja `data-manager.export-error`. En la vista, `<mm-data-manager exportable [exporting]="store.exporting()" (exportRequested)="store.exportX()">`.

### 8.2 PDF

El PDF de un registro se pide con `Accept: application/pdf, application/json;q=0.9`: la ruta produce PDF, responde `406` a un `Accept` que no lo admite, y sus errores salen en JSON con su código. El de una orden de compra en borrador sigue el flujo de las listas: `GET /purchase-orders/{id}/pdf` (§2.2) delega en el servicio de documentos con su tipo, responde `303` a `/documents/{id}`, y el archivo se descarga de `/documents/{id}/file`; la llave, la reutilización y el almacenamiento son los de §8.3, con el formato `pdf`. El de una orden enviada es la hoja fija que se guardó al enviarla (`purchase-orders/{publicId}/sent.pdf`) y responde `200` con sus bytes; el frontend la guarda con el nombre de su `Content-Disposition` o, sin él, con el número de la orden.

### 8.3 Reutilización

Un pedido igual nunca genera dos veces el mismo documento: lo reutiliza, o espera al que se está generando (D-106). El estado de cada documento está en `generated_documents` (diccionario §2.7).

**La llave.** El SHA-256 del JSON canónico de:
- el tipo (`kind`) y el formato;
- los criterios normalizados: `q` sin espacios sobrantes y en minúsculas; los filtros en un orden fijo, sin los nulos; `sort`; nunca `page` ni `size`;
- la llave del alcance;
- el idioma de la petición y la zona de los momentos;
- la versión de los datos.

**La llave del alcance** (`scopeKey`) es el SHA-256 del alcance efectivo de quien pide: las empresas en que actúa —o `SYSTEM`—, sus restricciones de sede y de marca, y las banderas de visibilidad propias del tipo, como los datos personales de los colaboradores (D-105). Nunca la cuenta: dos personas con el mismo alcance comparten el documento, y una con otro alcance nunca lo ve. Quien no lee nada de ese tipo tiene también un alcance, el alcance vacío, que se escribe `NONE`: la lista le responde lo que le responde, y sus documentos los leen solo quienes tampoco leen nada.

**La versión de los datos** la da cada tipo, con una sola consulta a los contadores de `data_versions`, en la transacción de quien pide (diccionario §2.7, D-128): la versión de una tabla es la suma de sus 64 casillas, y la de varias, la de cada una en un orden fijo. La consulta genérica es `JdbcDataVersions`, de `shared/infrastructure/persistence/jdbc`, que no conoce ninguna tabla. Cada contexto la usa desde su propio puerto, declarado en `application/internal/outboundservices` e implementado en `infrastructure/persistence/jdbc`, con un método por cada cosa que versiona, así ninguna tabla queda nombrada en la capa de aplicación (H2, primera fase de la segunda ronda): `LocationDataVersions`, `IamDataVersions`, `OrganizationDataVersions`, `VehicleCatalogDataVersions` y `FleetDataVersions`, con sus adaptadores `Jdbc<Contexto>DataVersions`. Lo que un tipo resuelve por la fachada de otro contexto llega con la versión que da esa fachada (`fetchDataVersion()` de `LocationContextFacade`, `OrganizationContextFacade` y `VehicleCatalogContextFacade`, que la piden a `whatTheFacadeResolves()` de su puerto): un contexto nunca nombra las tablas de otro. El Inventario, por ejemplo, lee `vehicles`, `vehicle_statuses`, `vehicle_status_history` y `vehicle_mileage_readings` (`FleetDataVersions.vehicles()`), y suma las versiones de `vehiclecatalog` y de `organization`.

Los contadores los sube `bump_data_version()`, con dos disparadores por sentencia en cada tabla versionada, uno después de insertar y otro después de actualizar, y solo si la sentencia cambió alguna fila. La casilla se sortea una vez por transacción y por tabla, y las sentencias siguientes de esa transacción la reutilizan (V55). El aumento va en la transacción del cambio, así que una transacción que confirma tarde, después de que alguien leyó la versión, la cambia igual, sea cual sea el orden de sus identificadores: lo comprueba `JdbcDataVersionsTests.theVersionChangesWhenTheTransactionWithTheLowerIdCommitsLast`. Como nada se borra (§3.1), cualquier alta, cambio o desactivación la mueve, y el pedido siguiente genera un documento nuevo. **La versión es global por tabla**, no del alcance de quien pide: un cambio en una empresa cambia la versión que leen todas. Es más conservadora —el documento se reutiliza un poco menos— y nunca reutiliza uno con datos viejos. Una tabla nueva que muestra un documento siembra sus 64 casillas y crea sus dos disparadores en la migración que la versiona; sin ellos, su primera escritura y su versión fallan diciéndolo.

**Lo que cuesta la versión** (H3, medido el 20/10/2026). Hasta D-128 la versión se leía de las propias tablas —`count(*)`, la suma de `xmin` y `max(updated_at)`, bajo Row Level Security forzada—, y esa consulta leía cada tabla completa en cada pedido. Se midió con `EXPLAIN ANALYZE`, en una transacción que se deshizo sobre `erp_mm_db_verify`, con 5 000 unidades, 100 000 movimientos de estado, 100 000 lecturas de kilometraje y 5 000 cuentas y colaboradores, contra un umbral de **50 ms por exportación**:

| Exportación | Antes, alcance `SYSTEM` | Antes, una empresa | Con `data_versions` |
|---|---|---|---|
| Inventario (`vehicles`, 13 tablas con las de sus fachadas) | 160 ms | 199 ms | 1,6 ms (`SYSTEM`) y 1,3 ms (una empresa) |
| Cuentas (`users`) | 21 ms | 22 ms | 1,0 ms y 1,1 ms |
| Colaboradores de una empresa | 16 ms | 19 ms | 1,2 ms |
| Ubicaciones de una empresa | 11 ms | 9 ms | 1,1 ms |
| Divisiones de un país, tipos de equipamiento | 2 a 5 ms | 2 a 4 ms | 0,2 a 0,3 ms |

Las cifras son la suma de las medianas de cada consulta. Por tabla, antes, `vehicle_status_history` y `vehicle_mileage_readings` costaban de 65 a 85 ms cada una, y `vehicles`, de 12 a 14 ms; ahora cada consulta lee 64 filas por tabla por la llave primaria y cuesta menos de 1 ms, con cualquier volumen. El Inventario pasaba el umbral; por eso D-128.

**El riesgo que queda: el interbloqueo.** Una transacción bloquea a lo sumo una casilla de cada tabla que escribe, así que dos que escriben la misma tabla no se interbloquean en los contadores. Dos que escriben las mismas dos tablas en orden opuesto sí pueden, cuando cada una sorteó la casilla que tiene la otra en la otra tabla: alrededor de una vez en 4 096. Lo mitiga `DeadlockRetry` (`shared/infrastructure/persistence/transactions`), un aspecto que envuelve cada método público de los `*CommandServiceImpl` de `application.internal.commandservices` y cada `@Scheduled` de `application.internal.scheduledtasks`, en todos los contextos, por fuera de su transacción. Si la transacción más externa pierde un interbloqueo —una excepción con SQLSTATE `40P01` en su cadena de causas, la traduzca Spring o no—, registra una advertencia con el nombre de la operación, nunca sus valores, y la repite entera una sola vez, en una transacción nueva. Una operación llamada dentro de otra, o dentro de una transacción ya abierta, deja pasar el fallo; un segundo interbloqueo pasa como está. Por eso un servicio de comandos tiene que poder repetirse entero: todo lo que hace va en su transacción.

**El vencimiento.** Un documento listo se reutiliza durante `documents.reuse-for` (por omisión `PT10M`). Pasado ese plazo, el pedido igual lo da por vencido y genera otro. Cada `documents.expire-check-interval` (por omisión `PT15M`) después de terminar la corrida anterior, `GeneratedDocumentExpiryTask`, del módulo `documents`, vence lo listo que ya pasó su plazo y vacía su contenido; ningún pedido la espera. Corre en una sola instancia a la vez, como toda tarea programada: el puerto `SingleInstanceRun` (`shared.application.scheduling`) y su adaptador `PostgresAdvisoryLockSingleInstanceRun` (`shared.infrastructure.scheduling`), que toma un candado de PostgreSQL (`pg_try_advisory_lock`, con una llave derivada del nombre del proceso) en una conexión propia durante toda la corrida; si otra instancia lo tiene, la corrida se salta. Los objetos de Spaces los borra la regla de ciclo de vida del bucket: los de `documents/`, al día de creados. Las filas no se borran nunca.

**Un límite conocido: lo que depende del reloj.** La versión de los datos solo ve los cambios en las filas. Un filtro que depende de la hora, como `overdue` del Inventario ("Solo vencidas"), cambia de resultado sin que cambie ninguna fila. Así, un documento pedido con `overdue=true` se reutiliza hasta `documents.reuse-for` (diez minutos por omisión) sin las unidades que se vencieron en ese lapso.

**La generación única.**
1. El servicio reclama la llave con un `INSERT … ON CONFLICT DO NOTHING` sobre `generated_documents_active_key_uq`, el índice único parcial que admite a lo sumo un documento `generando` o `listo` por llave.
2. Si gana, genera. Antes toma un lugar de un semáforo de `documents.max-concurrent-generations` lugares (por omisión 2), que es **por instancia**: dos instancias generan hasta el doble (`GenerationPermits`). Lo espera hasta `documents.permit-wait` (por omisión `PT30S`); si no lo consigue, marca el documento fallido y responde `503 DOCUMENTS_BUSY`. Mientras genera, el documento lleva su `lease_until`: el reclamo lo fija a `documents.lease-for` (por omisión `PT2M`) de ese momento, y lo renueva por el mismo plazo al conseguir el lugar.
   - **El dueño del lease.** El último `lease_until` que fijó una petición, exacto al microsegundo, hace de dueño: cada escritura de esa generación —renovar el lease, guardar el archivo en la fila, marcarlo listo, marcarlo fallido— lo exige junto con `status = 'generando'`. No hace falta una migración.
   - **Una generación que dura más que su lease.** Si otra petición la toma (paso 3), fija un `lease_until` propio, y desde ese momento ninguna escritura de la original toca la fila. La original no responde `500`: relee el documento y responde `303` a él, en el estado en que esté. Si ya está listo, es como si lo hubiera reutilizado; si no, responde lo mismo que quien encuentra una generación en curso.
   - **Un fallo tardío de la original** tampoco toca la fila: no marca fallido el documento que la otra petición sigue generando.
   - **Lo que no cambia.** Un `DOCUMENTS_BUSY` y el error que la lista le responde a quien pide se responden igual, aunque otra petición haya tomado el documento.
   - **En Spaces.** El objeto se llama como el documento, así que la original sube un archivo igual con el mismo nombre. Solo quien tiene el lease marca el documento listo.
3. Si no gana, lee el documento activo de esa llave:
   - listo y vigente: lo reutiliza;
   - listo y vencido: lo pasa a vencido con un `UPDATE` condicionado y vuelve a reclamar la llave;
   - generándose, con su `lease_until` vigente: lo espera consultando la base cada 200 ms, hasta `documents.wait-for` (por omisión `PT10S`). Si sigue generándose, responde igual, con el documento en `generating`, y el frontend sigue consultando `/documents/{id}`. La espera está acotada (H5): si ya esperan `documents.max-waiting` pedidos en la instancia (por omisión 20), el siguiente no espera y responde enseguida el `303`, porque el frontend consulta igual (`document-download.ts`);
   - generándose, con su `lease_until` vencido —la instancia que lo generaba se apagó en un despliegue, o la generación duró más que su lease—: lo toma con un `UPDATE` condicionado, que fija su propio `lease_until`, y lo genera.
4. En todos los casos responde `303` a `/documents/{id}`.

El servicio de comandos de los documentos está partido (H4, primera fase de la segunda ronda). El reclamo, la toma, el vencimiento, el lease y su dueño viven en `DocumentClaims`, cada paso en su propia transacción; el semáforo, en `GenerationPermits`; los dos en `documents/application/internal/commandservices`, junto a `DocumentCommandServiceImpl`, que lleva la espera. El nombre del archivo lo da el puerto `DocumentFileBaseNames`, de `documents/application/internal/outboundservices/names`, que implementa `MessageSourceDocumentFileBaseNames`, de `documents/infrastructure/i18n`, con el `MessageSource` de Spring: la aplicación no lee mensajes. `documents.max-waiting` acota solo la espera del documento que genera otro pedido (paso 3), con un semáforo propio; la espera de un lugar para generar la acota `documents.permit-wait` (paso 2). Un pedido que esperó y termina generando él mismo libera su lugar entre los que esperan antes de generar. Lo prueba `DocumentsMaxWaitingIntegrationTests.pastTheRequestsThatWaitTheNextOneAnswersAtOnce`.

**El almacenamiento.** Un puerto `DocumentStorage` con dos adaptadores:
- `base`, por omisión: el archivo en la columna `content`.
- `spaces`: DigitalOcean Spaces, con el cliente `s3` de AWS SDK v2 y su firmador de URL. Se enciende solo si `documents.storage.spaces.bucket` no está vacío. Sus variables son `DOCUMENTS_SPACES_BUCKET`, `DOCUMENTS_SPACES_REGION`, `DOCUMENTS_SPACES_ENDPOINT`, `DOCUMENTS_SPACES_ACCESS_KEY` y `DOCUMENTS_SPACES_SECRET_KEY`: están en `.env.example` y, comentadas, en `.do/app.yaml`, para no exigir valores que todavía no existen. Sus pruebas usan un cliente S3 simulado; todavía no hay bucket.

**Por qué así en App Platform.** El backend corre en App Platform, donde el disco de una instancia se pierde al reiniciarla y, durante cada despliegue, la instancia vieja y la nueva atienden a la vez. Un documento guardado en la memoria o en el disco del proceso desaparecería con el reinicio, o lo pediría la otra instancia sin encontrarlo. Por eso el estado vive en PostgreSQL, que ven las dos instancias, y el archivo en la base o en Spaces; `lease_until` cubre la instancia que se apaga a mitad de una generación, y el semáforo evita que varias exportaciones grandes a la vez agoten la memoria de una instancia.

---

## 9. Seguridad

### 9.1 Sesión (diccionario §5.12, D-33)

- La sesión vive en el servidor. El navegador recibe la cookie `MM_SESSION` (`HttpOnly`, `SameSite=Strict`, `Secure` en producción) y la envía solo.
- Toda petición que cambia datos (`POST`, `PUT`, `PATCH`) lleva el token CSRF: llega en la cookie `XSRF-TOKEN` y vuelve en el encabezado `X-XSRF-TOKEN`. Vale también para iniciar sesión y registrarse.
- La API y el frontend se sirven desde el mismo origen. No hay CORS.
- `GET /authentication/me` y el inicio de sesión responden quién es la cuenta: `{id, email, mustChangePassword, roles, permissions, global, companies, employeeCompanyIds}`. `roles` son los códigos de sus roles vigentes en cualquier alcance; `permissions` (desde el 19/10/2026), los códigos de los permisos que esos roles incluyen, sumados, sin repetir y en orden, cualquiera sea la empresa en que actúa cada uno (diccionario §5.8 regla 1, D-99): la pantalla sabe por ellos qué ofrecer y qué puede marcar en un rol propio, y dónde actúa cada uno lo decide el backend; `global` dice si tiene un rol de alcance sistema, que ve todas las empresas. `companies` son las empresas que la cuenta alcanza —todas las activas si es global; si no, las de sus asignaciones vigentes—, cada una como `{id, name, timeZone}`, más sus `permissions` (abajo): `name` es el nombre comercial y `timeZone` la zona de su país (diccionario §2.6). Hasta la segunda ronda de la auditoría llevaba además `administered`, si la cuenta la administraba (el superadministrador, todas; el administrador de empresa, la suya; el administrador de accesos, ninguna). No hay una empresa "actual": una cuenta puede trabajar para varias (diccionario §5.3). `employeeCompanyIds` (desde el 18/10/2026) son los `public_id` de las empresas donde la cuenta tiene un colaborador activo, tenga o no un rol ahí y aunque `companies` no las liste; un superadministrador que todavía no se vinculó lo recibe vacío (D-89), y la pantalla de colaboradores lo usa para deshabilitarle, diciendo por qué, la salida, el regreso y deshacer un vínculo.
- **Lo que suma la segunda ronda de la auditoría** (construida e integrada en `develop` el 20/10/2026; D-117, D-122). La respuesta queda `{id, email, mustChangePassword, roles, permissions, global, companies, employeeCompanyIds, systemPermissions, actsOnReserved}`, y cada empresa, `{id, name, timeZone, permissions}`. `companies[].permissions` son los permisos que la cuenta tiene en cada empresa, por las asignaciones que actúan en ella; `systemPermissions`, los que tiene por asignaciones de alcance sistema, y son los únicos por los que viajan los ocho de accesos y roles (`seguridad.dar_acceso`, `asignar_roles`, `asignar_roles_sin_tenerlos`, `bloquear_cuentas`, `restablecer_contrasenas`, `confirmar_vinculo`, `desvincular_cuentas` y `gestionar_roles`), que solo cuentan con ese alcance; y `actsOnReserved`, un dato derivado que dice si la cuenta actúa sobre lo reservado —hoy, si es superadministrador (D-121)—, para que la pantalla aplique del lado de quien opera las reglas sobre la cuenta de destino. Con ellos el superadministrador sin vincular se vincula y la pantalla sabe qué le queda habilitado sin refrescar nada más (D-89). `administered` ya no viaja: lo quitó B14, una vez el frontend de la segunda ronda en `develop`, que ya no lo lee; su commit llega al cerrar la ronda. Los permisos de una empresa, en orden, son los de las asignaciones de alcance sistema y los de las de esa empresa o una de sus sedes, sin los ocho de accesos y roles.

### 9.2 Quién puede llamar a qué

| Ruta | Acceso |
|---|---|
| `sign-in`, `sign-up`, `sign-up/status`, `sign-up/companies`, documentación de la API, `actuator/health` | Público. `sign-up/companies` responde solo mientras el registro está encendido, y solo nombres (diccionario §5.2). Las listas de sedes y de tipos de documento del registro: pendiente (§14.16) |
| `me`, `change-password` | Con sesión, aunque la contraseña sea provisional |
| `/documents/{id}` y `/documents/{id}/file` | Con sesión. No exigen un permiso propio: la guarda es el alcance del documento, que se vuelve a calcular para quien pregunta; uno de otro alcance responde `404 DOCUMENT_NOT_FOUND` (§8.1, D-106). El permiso lo exigió la lista al pedirlo |
| Lectura de catálogos generales (monedas, países, escalones, divisiones y tipos de documento tributario) y de catálogos de flota | `catalogos.ver`, que la semilla da a los siete roles del sistema (diccionario §5.6, D-43). Hasta el paso 3b, cualquier cuenta con un rol vigente |
| Escritura de catálogos generales | El permiso de la operación (D-130): `catalogos.crear_monedas`, `editar_monedas` o `desactivar_monedas`, y en países `crear_paises` para el alta, `editar_paises` para corregir el país o agregar escalones, divisiones y documentos, y `desactivar_paises` para activar o desactivar una división o un documento. La semilla se los da solo a `SUPERADMIN`. `catalogos.gestionar_generales` ya no abre ninguna ruta |
| Escritura de catálogos de flota | El permiso de la familia y de la operación (D-130): crear, editar o desactivar tipos de vehículo, marcas y modelos, colores, equipamiento o estados. La semilla se los da solo a `SUPERADMIN`. `catalogos.gestionar_flota` y `catalogos.gestionar_estados` ya no abren ninguna ruta |
| Lectura de recursos de empresa (empresas, sedes, marcas comerciales y ubicaciones, y todo lo que lleve `company_id` en los pasos siguientes) | El permiso de lectura del recurso —hoy `configuracion.ver_estructura`, que tienen los siete roles— **y** el alcance de la asignación que lo trae (diccionario §5.8 regla 6, D-99): una lista filtra por las empresas donde ese permiso actúa (`companyReach`, que responde `CompanyReach.Every` o `Some` con sus empresas, quizá ninguna), y una empresa ajena, o alcanzada solo con un rol que no trae el permiso, o algo que le pertenece, responde `404`. Un rol de alcance sistema con el permiso ve todo; uno de sede lee toda su empresa (D-14). Alcanzar la empresa, y Row Level Security debajo, siguen como piso (D-66) |
| Escritura de recursos de empresa | El permiso de la operación (D-130): crear, editar o desactivar empresas, sedes, marcas comerciales o ubicaciones, que la semilla da solo a `SUPERADMIN`, **y** que su asignación actúe en toda esa empresa (`actsOnWholeCompany`). Crear una empresa exige además alcance de todo el sistema. Donde no actúa, `403 ACCESS_DENIED`; fuera del alcance, `404` (D-99). Los tres `gestionar_*` de configuración ya no abren ninguna ruta |
| Compras y fondos (26/10/2026, D-134) | Las pantallas y las lecturas piden el permiso de lectura —`compras.ver`, `fondos.ver`— donde actúa, también en el servicio: un permiso de escritura no abre una lectura, y escribir sobre lo que no se lee responde `404`. Cada escritura de compras pide el permiso de su operación (D-130): crear, editar o desactivar proveedores, y, desde el 29/10/2026, `compras.editar_proveedores` para agregar, corregir, retirar o reactivar uno de sus contactos, que lo mantiene (diccionario §5.6); `compras.crear_plantillas`, `editar_plantillas` o `desactivar_plantillas` en tipos y plantillas; `compras.crear_ordenes`, `editar_ordenes`, `enviar_ordenes`, `cumplir_ordenes` o `anular_ordenes` en las órdenes. Los tres `gestionar_*` de compras ya no abren ninguna ruta. Un proveedor, un fondo, un tipo de orden y una plantilla general los escribe solo una asignación de toda la empresa y sin marca (`actsOnWholeCompanyEveryBrand`); una plantilla con marca, también una de esa marca. Ceder una unidad a un fondo y cerrar la cesión, una de toda la empresa o de la marca de la unidad, nunca la de la sede (excepción a D-101). Las cesiones de una unidad se leen con `stock.ver`, como su ficha (diccionario §2.5, §5.6, §12.2) |
| Colaboradores | La fila mínima con `configuracion.ver_colaboradores`, que tienen los siete roles; `documentNumber`, `phone`, `photoPath` y la exportación con `configuracion.ver_datos_personales_colaboradores`, que tienen `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, cada uno donde actúa (D-105, tabla de abajo). Registrarlos exige `seguridad.registrar_colaboradores` y modificarlos `seguridad.modificar_colaboradores`; sus salidas y regresos, `seguridad.registrar_salidas_y_regresos`, donde actúan (D-122, D-130) |
| Cuentas, accesos, colaboradores y roles | Según la tabla de abajo: un permiso `seguridad.*` con `@PermissionRequired` en la ruta, y el servicio decide además dónde actúa y sobre qué cuenta (diccionario §5.8 regla 7, D-117, D-122). Hasta la segunda ronda de la auditoría, el rol con `@PreAuthorize` |
| Toda operación de negocio | Un permiso del catálogo, también para las cuentas, los accesos, los colaboradores y los roles (D-95, D-117). Tener sesión no alcanza (diccionario §5.2) |

**Cuentas, accesos, colaboradores y roles, por permiso** (D-117 a D-124; construido en la segunda ronda de la auditoría e integrado en `develop` el 20/10/2026, merges `007151d` en el backend y `11235e0` en el frontend). Cada fila es la del controlador: `@PermissionRequired` en la clase, y en el método cuando la escritura pide otro permiso que la lectura —`POST /users/{id}/password-reset` sobre `seguridad.ver_cuentas` de `UsersController`, las escrituras de `/roles` sobre `seguridad.ver_roles`, y el alta y la edición de colaboradores sobre `configuracion.ver_colaboradores`—, porque el del método reemplaza al de la clase. Ninguna ruta lleva ya un `@PreAuthorize` con un rol: lo comprueba `NoRoleAnnotationTests`, que además exige un permiso en todo controlador salvo el inicio de sesión y `/documents` (D-106). `@AnyRoleRequired`, que solo pedía tener algún rol, ya no existe; su commit llega al cerrar la ronda. La red `SecurityOperationsBySystemRoleIntegrationTests`, escrita antes de cambiar nada, comprueba que los siete roles del sistema obtienen con la semilla lo mismo que con sus códigos: siete roles por veinticuatro operaciones, más cinco casos de actor y destino —el superadministrador sin vincular, la cuenta propia, una cuenta protegida, una con contraseña provisional y el administrador de una empresa sobre otra—. Una cuenta **protegida** es la que tiene un rol **reservado** vigente, en cualquier empresa: uno que incluye un permiso `seguridad.*` que no es de ver, como `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA` (D-121). La lista de cuentas lo dice en `managedBySuperadmin` (§6.2). Un rol reservado lo otorga y lo retira solo un superadministrador.

| Operación | Permiso | Dónde actúa | Reglas sobre la cuenta de destino | Con la semilla |
|---|---|---|---|---|
| `POST /access-grants` (crear o vincular la cuenta) | `seguridad.dar_acceso`; vincular la cuenta de otra persona, o una sin colaborador, después de ver sus titulares, además `seguridad.confirmar_vinculo` (D-82) | Alcance sistema | Una protegida, solo un superadministrador (`ACCOUNT_MANAGED_BY_SUPERADMIN`). La propia, solo mientras quien opera no tiene un colaborador activo (D-89, D-122); fuera de eso, `OWN_ACCOUNT_FORBIDDEN`, y quien opera tiene que estar vinculado (`ACTOR_EMPLOYEE_REQUIRED`) | `SUPERADMIN` y `ADMIN_ACCESOS`; confirmar, solo `SUPERADMIN` |
| `POST /employees/{id}/account-unlinking` | `seguridad.desvincular_cuentas` | Alcance sistema | Nunca la propia; una protegida, solo un superadministrador; la guardia del último superadministrador | `SUPERADMIN` |
| `POST /users/{id}/role-assignments`, `POST /role-assignments/{id}/revocation` y `GET /users/{id}/role-assignments` | `seguridad.asignar_roles`; un rol no reservado cuyos permisos quien opera no tiene, además `seguridad.asignar_roles_sin_tenerlos` (D-120) | Alcance sistema | Un rol reservado, solo un superadministrador (`ACCESS_DENIED`), y solo a una cuenta con contraseña propia; nunca a la propia ni a `SYSTEM`; una protegida, solo un superadministrador; nunca deja a la cuenta con un par incompatible (D-119); retirar exige poder otorgar, y la guardia del último superadministrador | `SUPERADMIN` y `ADMIN_ACCESOS`; la excepción, solo `ADMIN_ACCESOS` |
| `POST /users/{id}/suspensions`, `POST /suspensions/{id}/closure` y `GET /users/{id}/suspensions` | `seguridad.bloquear_cuentas` | Alcance sistema | Nunca la propia ni `SYSTEM`; una protegida, solo un superadministrador; la guardia | `SUPERADMIN` y `ADMIN_ACCESOS` |
| `POST /users/{id}/password-reset` | `seguridad.restablecer_contrasenas` | Alcance sistema | Nunca la propia ni la de `SYSTEM`; una protegida, solo un superadministrador | `SUPERADMIN` y `ADMIN_ACCESOS` |
| `GET /users`, con o sin `companyId`, y su exportación | `seguridad.ver_cuentas` | Con alcance sistema, todas las cuentas; con una empresa, las de `?companyId=` de esa empresa, de solo lectura. Una empresa fuera del alcance, `404`; una alcanzada donde el permiso no actúa, `403` | — | Los tres administradores; `ADMIN_EMPRESA`, las de su empresa |
| `GET /roles`, `GET /roles/{id}`, `GET /permissions` y `GET /permission-matrix` | `seguridad.ver_roles` | Los del sistema y los generales, con cualquier asignación que lo traiga; los de una empresa, donde actúa, y `404` en otra (D-98, D-122) | — | Los tres administradores |
| `POST /roles` (también la copia) | `seguridad.crear_roles` | Alcance sistema | Nunca el código de un rol del sistema (D-97); la copia responde las mismas negativas que marcar sus permisos (D-124, D-130) | `SUPERADMIN` |
| `PUT /roles/{id}` | `seguridad.editar_roles` | Alcance sistema | Nunca un rol del sistema (D-97) | `SUPERADMIN` |
| `PATCH /roles/{id}/status` | `seguridad.desactivar_roles` | Alcance sistema | Nunca un rol del sistema (D-97) | `SUPERADMIN` |
| `PUT /roles/{id}/permissions` | `seguridad.marcar_permisos_de_roles` | Alcance sistema | Nunca un rol del sistema (D-97) ni los permisos de uno que quien edita tiene vigente (D-70); los permisos privilegiados de un rol con titulares, solo un superadministrador (D-121); las reglas 1 y 2 de §5.5, contadas según el alcance del rol (D-122); los once de accesos y roles, solo en un rol de alcance `SYSTEM` (D-130) | `SUPERADMIN` |
| `POST /companies/{id}/employees` | `seguridad.registrar_colaboradores` | La empresa o la sede de la asignación: un colaborador sin sede exige toda la empresa | No exige que quien opera esté vinculado: el superadministrador sin vincular crea el suyo (D-89, D-130) | `SUPERADMIN` y `ADMIN_EMPRESA` |
| `PUT /employees/{id}` | `seguridad.modificar_colaboradores` | La empresa o la sede de origen y, si cambia de sede, también la de destino | Como el alta (D-130) | `SUPERADMIN` y `ADMIN_EMPRESA` |
| `POST /employees/{id}/termination` y `/reactivation` | `seguridad.registrar_salidas_y_regresos` | Como la anterior | Nunca la propia salida ni el propio regreso; quien opera, vinculado a un colaborador activo (D-89); la guardia del último superadministrador | `SUPERADMIN` y `ADMIN_EMPRESA` |

- Las acciones de cada fila de Colaboradores las decide el backend en `allowedActions` (§4.2), con estas mismas reglas.
- Las reglas 1 y 2 de §5.5 cuentan lo que quien edita o quien otorga tiene donde el rol actúa (D-122): para un rol general, solo sus asignaciones de alcance sistema; para uno de la empresa C, lo que actúa en toda C, sin sede y sin marca. Una asignación de empresa limitada a una marca no cuenta para toda la empresa.
- Nadie actúa sobre lo propio (`OWN_ACCOUNT_FORBIDDEN`), con la única excepción de la autovinculación de la primera fila. En los hechos es solo del superadministrador (D-89): quien tiene `seguridad.dar_acceso` con alcance sistema tiene un rol reservado, así que su cuenta es protegida, y vincularla le responde `ACCOUNT_MANAGED_BY_SUPERADMIN` a quien no actúa sobre lo reservado. Una negativa por permiso responde `ACCESS_DENIED`, y las reglas sobre el destino conservan sus códigos (D-122).

**Lo que hace cada rol del sistema con la semilla** (decidido el 15/10/2026 y el 16/10/2026; diccionario v1.18, D-55 a D-72; desde la segunda ronda, es lo que resulta de la tabla de arriba). `ADMIN_EMPRESA` (C) es el administrador de la empresa C.

| Operación | `SUPERADMIN` | `ADMIN_ACCESOS` | `ADMIN_EMPRESA` (C) | Otro rol |
|---|---|---|---|---|
| `POST /access-grants` (crear o vincular la cuenta) | Sí, también una cuenta protegida. A sí mismo, solo mientras no tiene un colaborador activo. Una cuenta de otra persona, o sin colaborador, solo después de ver sus titulares y confirmar (D-82) | Sí, en cualquier empresa, salvo vincular una cuenta protegida (`ACCOUNT_MANAGED_BY_SUPERADMIN`) o una de otra persona (`EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH`, sin `context`) | `403` | `403` |
| `POST /employees/{id}/account-unlinking` | Sí, vinculado a un colaborador activo, salvo la propia cuenta y con la guardia del último superadministrador | `403` | `403` | `403` |
| Otorgar un rol | Cualquier rol, a cualquier cuenta salvo la propia y `SYSTEM` | Cualquiera salvo los reservados (`ACCESS_DENIED`), nunca a la propia ni a una protegida | `403` | `403` |
| Retirar una asignación | Cualquiera salvo las propias, con la guardia del último superadministrador | Las que puede otorgar, nunca de la propia ni de una protegida | `403` | `403` |
| Abrir o cerrar un bloqueo | Salvo en la propia y en `SYSTEM`, con la guardia | Salvo en la propia, en `SYSTEM` y en una protegida | `403` | `403` |
| Restablecer la contraseña | Salvo la propia y la de `SYSTEM` | Salvo la propia, la de `SYSTEM` y la de una protegida | `403` | `403` |
| `GET /users` sin `companyId` y su exportación; `GET /users/{id}/role-assignments`; `GET /users/{id}/suspensions` | Sí | Sí | `403` | `403` |
| `GET /users?companyId` y su exportación | Sí | Sí | Sí, de solo lectura, para C. Una empresa que alcanza sin que `seguridad.ver_cuentas` actúe en ella, `403`; una fuera de su alcance, `404` | `403` |
| `GET /roles` y `GET /roles/{id}` | Sí, todos | Sí, todos | Los del sistema, los generales y los de C; uno de otra empresa, `404` (D-98) | `403` |
| `GET /permissions` | Sí | Sí | Sí | `403` |
| `POST /roles`, `PUT /roles/{id}`, `PATCH /roles/{id}/status`, `PUT /roles/{id}/permissions` | Sí, general o de una empresa activa, y marca solo los permisos que posee (§5.5 regla 1); nunca un rol del sistema (D-97) | `403` | `403` | `403` |
| Registrar o modificar colaboradores | Sí, también sin vincular, porque tiene que crear el suyo (D-89) | `403` | Sí, en C | `403` |
| Salida y regreso | Sí, salvo los propios, y solo vinculado a un colaborador activo (D-89) | `403` | Sí, en C, salvo los propios | `403` |
| Escribir empresas, sedes, marcas y ubicaciones, y corregir el documento tributario de una empresa (`PUT /companies/{id}/tax-id`, D-85) | Sí | `403` | `403` | `403` |
| Leer datos de empresa | Todas, por `configuracion.ver_estructura` de alcance sistema | Todas, igual | Las empresas donde su rol, que trae el permiso, actúa: C | Las empresas donde el rol que trae el permiso actúa; los siete roles del sistema lo traen |
| Leer colaboradores: `GET /companies/{companyId}/employees`, su página, el filtro de sede y `GET /employees/{id}` (D-105) | Todas, con los datos personales | Todas, con los datos personales | Los de C, con los datos personales | La fila mínima, sin `documentNumber`, `phone` ni `photoPath`, en las empresas donde su rol actúa; los siete roles del sistema traen `configuracion.ver_colaboradores`. Fuera de ellas, `404` |
| Exportar los colaboradores de una empresa (D-105) | Sí | Sí | Los de C | `403`: el archivo lleva el documento de cada uno |

- Todas las operaciones de cuenta exigen que quien opera esté vinculado a un colaborador activo (`ACTOR_EMPLOYEE_REQUIRED`), salvo la autovinculación del primer superadministrador (diccionario §5.2, paso 6). El superadministrador y el administrador de empresa también lo necesitan para la salida y el regreso (D-89).
- Nadie actúa sobre lo propio (`OWN_ACCOUNT_FORBIDDEN`): ni sobre su cuenta ni sobre su propia salida o regreso. Cambiar la propia contraseña tiene su ruta (`change-password`).
- Una cuenta nunca junta un par de permisos incompatibles entre sus roles vigentes, y por eso nunca tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` a la vez (D-119); un rol reservado solo se otorga a una cuenta que ya cambió su contraseña provisional (§7.2, D-121).
- No hace falta proteger al último administrador de accesos: el superadministrador hace todo lo que él hace. Al último superadministrador sí: ninguna operación lo deja sin uno (§7.2, `LAST_ACTIVE_SUPERADMIN`).
- Un rol global lee lo que su matriz le da (D-68): el superadministrador, todo; el administrador de accesos, los catálogos, la estructura y los colaboradores con sus datos personales, nada del negocio (diccionario §5.6).

- Toda operación de negocio y toda lectura de datos exigen un permiso con `@PermissionRequired("catalogos.ver")`, de `shared`, en la clase o en el método del controlador: es `@PreAuthorize("hasAuthority('PERM_{value}')")`, y `iam` convierte en cada petición los permisos de los roles vigentes en autoridades `PERM_<código>` (construido el 19/10/2026, D-95). El servicio decide dónde actúa el permiso con `AccessScope.actsOn(permiso, empresa[, sede])` para una lectura o una escritura y con `companyReach(permiso)` para una lista, que responde todas las empresas (`CompanyReach.Every`) o algunas, quizá ninguna (`CompanyReach.Some`), para que un alcance vacío no se lea como «todas» (D-99, F3): el permiso dice qué y la asignación que lo trae dice dónde, también al leer. Desde la segunda ronda de la auditoría, también las cuentas, los accesos, los colaboradores y los roles, con sus doce permisos `seguridad.*`: no queda ninguna anotación de rol (`hasRole`, `hasAnyRole`, una autoridad `ROLE_` ni `@Secured`) en un `@PreAuthorize`, y `NoRoleAnnotationTests` lo comprueba (diccionario §5.8 regla 7, D-117). Los códigos no se escriben sueltos: cada uno de los 101 `@PermissionRequired` —82 hasta la primera fase y 19 de seguridad— usa una constante de `PermissionCodes`, de `shared/application/security` —un grupo por módulo: `Catalogs` (`VIEW`, `MANAGE_GENERAL`, `MANAGE_FLEET`, `MANAGE_STATUSES`), `Configuration` (`VIEW_STRUCTURE`, `MANAGE_COMPANIES`, `MANAGE_BRANCHES_AND_BRANDS`, `MANAGE_LOCATIONS`, `VIEW_EMPLOYEES`, `VIEW_EMPLOYEE_PERSONAL_DATA`) y `Stock` (`VIEW`, `VIEW_COSTS`, `CREATE_VEHICLE`, `EDIT_VEHICLE`, `CHANGE_STATUS`, `REGISTER_MILEAGE`, `REGISTER_ODOMETER_SUBSTITUTION`, `CORRECT_PLATE`, `CORRECT_VIN`, `LOAD_INITIAL_MIGRATION`), y `Security` (`VIEW_ACCOUNTS`, `VIEW_ROLES`, `GRANT_ACCESS`, `MANAGE_EMPLOYEES`, `MANAGE_ROLES`, `BLOCK_ACCOUNTS`, `REGISTER_EXITS_AND_RETURNS`, `ASSIGN_ROLES`, `RESET_PASSWORDS`, `CONFIRM_LINK`, `UNLINK_ACCOUNTS` y `ASSIGN_ROLES_WITHOUT_HOLDING`, con los conjuntos `SYSTEM_SCOPE_ONLY`, los ocho de accesos y roles, y `NOT_MARKED_ON_OWN_ROLES`, los tres que ningún rol propio incluye)—, como `@PermissionRequired(PermissionCodes.Catalogs.VIEW)`. Los cinco permisos que nombra un camino de estado (`stock.atender_en_taller`, `stock.entregar`, `stock.habilitar`, `stock.recuperar` y `stock.vender`, D-100) no tienen constante: el camino los lee del catálogo como dato. `PermissionCatalogConsistencyTests` comprueba que cada valor pedido y cada constante existen en `permissions`, que cada permiso sembrado tiene su constante salvo esos cinco, y que ningún `@PermissionRequired` lleva un texto suelto (mapa de contextos §5.3). No se escriben listas de roles por módulo (`hasAnyRole('ADMIN_EMPRESA', 'OPERACIONES', …)`): chocarían con los roles propios de cada empresa (diccionario §5.5).
- Un recurso de empresa se filtra por las asignaciones de la cuenta que traen el permiso: el alcance baja y nunca sube (diccionario §5.8). El filtro se aplica en el listado, en la exportación y en la consulta por id, también en las rutas anidadas (`/companies/{id}/branches`). Alcanzar la empresa con cualquier rol queda como piso (D-66): no basta para leerla ni escribirla.
- Una empresa ajena responde `404`, no `403`, para no revelar que existe (decidido el 06/10/2026, §14.6).
- **Construido el 16/10/2026**, en el mismo cambio que habilitó los roles de empresa y de sede: el filtro de alcance en empresas, marcas, sedes, ubicaciones y colaboradores, también en la exportación, por identificador y en las rutas anidadas, más `/users?companyId` para el administrador de empresa. `CompanyIsolationIntegrationTests` prueba que el administrador de la empresa A recibe `404` al pedir la empresa B o algo suyo y no la ve en el listado. Debajo, la base aplica la misma frontera con Row Level Security (diccionario §2.5; mapa de contextos §9).
- **Revisión de seguridad del 28/10/2026.** Fuera de desarrollo, el backend no arranca si la cuenta de base de la aplicación se salta Row Level Security —un superusuario o una con `BYPASSRLS`—: lo pregunta a PostgreSQL antes de atender un pedido, además de exigir, como antes, que no sea la cuenta de Flyway (`DatabaseAccountsConfiguration`). V83 da a las siete tablas de compras las políticas de V29 —leer, insertar y actualizar solo donde llega el alcance, sin política para borrar—, en lugar de una sola que dejaba pasar una fila sin empresa, y V84 da a `kept_files` la empresa de la plantilla o de la orden que nombra su llave y la aísla igual. `RowLevelSecurityMigrationTests` exige esa forma en toda tabla de una empresa. Las rutas de compras de otra empresa, también las de los contactos de un proveedor, responden `404` (`ProcurementCompanyIsolationIntegrationTests`).
- El alcance lo da el puerto `CurrentAccessScope` (mapa de contextos §5.3), nunca "el colaborador de la cuenta": una cuenta con colaboradores en dos empresas usa las dos.
- Un campo que la cuenta no puede ver **no viaja**: se omite de la respuesta, no se envía vacío (diccionario §2.5). Construido el 19/10/2026 con el colaborador: sin `configuracion.ver_datos_personales_colaboradores` en su empresa, la fila llega sin `documentNumber`, `phone` ni `photoPath` (D-105, §4.1). Después, sin `stock.ver_costos`, la fila del exportable de stock (`/reports/stock-export`) llegará sin `invoicedAmount`. El vehículo de `/vehicles` nunca trae costos: el valor de factura es de `procurement` (diccionario §3; mapa §6.1). Es la única excepción a la regla de enviar `null` (§4.1).

---

## 10. Validación

- Los recursos de entrada validan formato con Bean Validation, con mensajes en `ValidationMessages*.properties` y `messages*.properties`.
- Los límites de la API repiten los `CHECK` de la base: longitudes, rangos, patrones.
- Un `400` de validación lista cada campo con su problema en `details`.
- Las reglas que necesitan datos (unicidad, pertenencia a la empresa, caminos de estado) no son validación de formato: las resuelve el servicio de comandos y responden `404`, `409` o `422`.

---

## 11. Documentación de la API

- Cada controlador declara su `@Tag` y cada operación su `@Operation` y todas sus respuestas: el éxito, `400`, `401`, `403`, `404`, `409` y `422`, las que correspondan.
- Swagger UI queda en `/swagger-ui.html` y envía el token CSRF por sí solo.
- Los recursos llevan `@Schema` con un ejemplo realista del dominio: un RUC, un ubigeo, `PEN`.

---

## 12. Plantilla de un recurso

```
GET    /api/v1/countries?page=0              200  { "items": [ … ], "page": 0, … }
GET    /api/v1/countries                     200  [ … ]  (sin page: todo, para un select)
POST   /api/v1/countries                     201  Location: /api/v1/countries/{id}
GET    /api/v1/countries/{id}                200
PUT    /api/v1/countries/{id}                200
GET    /api/v1/countries/{id}/geo-divisions?page=0  200  { "items": [ … ], "page": 0, … }
POST   /api/v1/countries/{id}/geo-divisions  201  Location: /api/v1/geo-divisions/{id}
GET    /api/v1/geo-divisions/{id}            200
PUT    /api/v1/geo-divisions/{id}            200
PATCH  /api/v1/geo-divisions/{id}/status     200
```

```json
{
  "id": "019352e1-5b1c-7a3e-9d4f-0a1b2c3d7f3a",
  "isoAlpha2": "PE",
  "isoAlpha3": "PER",
  "name": "Perú",
  "callingCode": "+51",
  "defaultCurrency": "PEN",
  "defaultTimezone": "America/Lima",
  "taxName": "IGV",
  "taxRate": "0.1800",
  "requiresPostalCode": false,
  "createdAt": "2026-09-27T14:05:00Z",
  "updatedAt": "2026-09-27T14:05:00Z"
}
```

La plantilla aplica propuestas pendientes: el `Location` (§14.9) y `taxRate` como texto (§14.2). El sobre paginado ya rige (§6.2).

---

## 13. Lo construido que no sigue estas convenciones (06/10/2026)

Las filas resueltas se conservan con su número, para que las referencias sigan valiendo.

La columna "Pendiente de" indica la propuesta de §14 que debe validarse antes del cambio. Sin ella, el cambio tiene respaldo en el diccionario o en una regla de arquitectura.

| # | Hoy | Según este documento | Pendiente de |
|---|---|---|---|
| 1 | ~~Rutas sin prefijo de contexto: `/api/v1/currencies`, `/countries`, `/users`, `/roles`, `/geo-division-levels`, `/geo-divisions`~~ | Resuelto el 06/10/2026: se rechazó el prefijo (§2.1, §14.1), así que las rutas construidas ya cumplen | — |
| 2 | ~~`/countries/{countryId}/levels` y `/divisions` aceptan el código ISO además del UUID~~ | Resuelto el 05/10/2026: solo el UUID, en `/countries/{id}/geo-division-levels`, `/geo-divisions` y `/tax-id-types` | — |
| 3 | ~~`DELETE /geo-division-levels/{id}`~~ | Resuelto el 05/10/2026: no hay `DELETE` y los escalones no se retiran (D-45) | — |
| 4 | ~~Las respuestas de escalones, divisiones y tipos de documento ya usan `id`, `countryId` y `parentId`, pero nombran el país con `countryId` (UUID) más `countryIsoAlpha2`~~ | Resuelto el 14/10/2026 en la rama `feature/location-closure`: nombran el país con `country: "PE"` (§4.2, D-13) | — |
| 5 | ~~El país referencia su moneda con `defaultCurrencyId` (UUID) y la repite en `defaultCurrencyIsoCode`~~ | Resuelto el 14/10/2026 en la rama `feature/location-closure`: el país responde y recibe `defaultCurrency: "PEN"` (D-13); un código inexistente responde `404 CURRENCY_NOT_FOUND` | — |
| 6 | `RoleResource.system`, y desde el 19/10/2026 también `RoleDetailResponse.system`, junto a `isActive`, que sí lleva su prefijo | `isSystem` (§4.1) | — |
| 7 | `taxRate` viaja como número | Texto decimal (§5.2) | §14.2 |
| 8 | Las altas de monedas, países y divisiones responden `201` sin `Location` | `201` con `Location` (§3) | §14.9 |
| 9 | ~~Filtro `activeOnly` (monedas)~~ | Resuelto el 06/10/2026: las monedas se filtran con `isActive`; `activeOnly` e `isoCode` ya no existen (la búsqueda `q` cubre el código). El filtro `operationalOnly` de países tampoco existe: el país no guarda si opera (D-3). Las empresas siguen filtrando con `active` | §14.10 |
| 10 | ~~`GET /countries/XX/divisions` con un país inexistente responde `200 []`~~ | Resuelto el 05/10/2026: escalones, divisiones y tipos de documento responden `404` si el país no existe | — |
| 11 | ~~Las divisiones territoriales se listan completas, sin página~~ | Resuelto el 06/10/2026: con `page` responden una página (§6.2) | — |
| 12 | ~~`GET /users` devuelve la lista completa, sin página~~ | Resuelto el 06/10/2026: con `page` responde una página (§6.2) | — |
| 13 | ~~Códigos `GEODIVISION_*`, `GEODIVISIONLEVEL_*` y `TAXIDTYPE_*`. El padre inexistente ya responde `GEODIVISION_NOT_FOUND`~~ | Resuelto el 20/10/2026 por D-114: un solo prefijo por agregado, sin guion bajo interno, así que estos códigos ya cumplen (§7.2) | — |
| 14 | ~~Un error inesperado devuelve en `details` el mensaje de la excepción~~ | Resuelto el 17/10/2026 en la rama `feature/location-closure`: `UNEXPECTED_ERROR` lleva en `details` un texto fijo y traducido, y la excepción queda solo en el registro del servidor (§7.1) | — |
| 15 | El `422` no dice qué regla falló | Campo `rule` (§7.2) | §14.5 |
| 16 | ~~`POST /api/v1/users` crea una cuenta sin colaborador (`email`, `temporaryPassword`, `roleIds`) y otorga roles en la misma petición; es la pantalla "Nuevo usuario"~~ | Resuelto el 16/10/2026: `POST /users` y la pantalla "Nuevo usuario" ya no existen. El acceso se da con `POST /access-grants` (§14.15), los roles se otorgan aparte en `/users/{id}/role-assignments`, y `/users` solo lista y exporta | — |
| 17 | ~~`BaseApiEndpoint.delete` en el frontend envía `DELETE`~~ | Resuelto el 06/10/2026: el método ya no existe; un registro se retira con `PATCH …/status`, una revocación o un cierre (regla 4.2) | — |
| 18 | Los errores del framework (ruta inexistente, método no permitido, tipo de contenido no soportado, parámetro faltante) responden con el nombre del estado como `code` (`NOT_FOUND`, `METHOD_NOT_ALLOWED`, `UNSUPPORTED_MEDIA_TYPE`, `BAD_REQUEST`), y `message` y `details` llegan en inglés, sin traducir (`GlobalExceptionHandler.handleErrorResponse`) | Un código del catálogo de §7.2 y `message` traducido (§7.1) | §14.11 |
| 19 | ~~El frontend elige el texto de error por el estado HTTP (`409` = duplicado, `422` = regla rota), y el store de sedes lee `message`~~ | Resuelto el 06/10/2026: `ApiError` conserva `code` y cada store elige la clave de `es.json` por `code`, con el mensaje genérico de la operación para un código desconocido; el estado solo decide para `0` y para la fila 18 (§7.1) | — |
| 20 | ~~`/api/v1/addresses` con `GET`, `POST`, `PUT` y `GET /{id}`; la empresa recibía `fiscalAddressPublicId` y el frontend creaba la dirección en una petición aparte, antes que la empresa~~ | Resuelto el 06/10/2026: la ruta ya no existe; la dirección viaja anidada en el alta de la empresa, embebida en su respuesta, y se corrige o se muda por `/companies/{id}/fiscal-address` (§3.3) | — |
| 21 | La empresa nombra su propio identificador `publicId`. ~~Referencia su país y su moneda funcional por UUID (`countryPublicId`, `functionalCurrencyPublicId`); el tipo de documento llega como `taxIdTypePublicId`~~ | `id` (§14.8). Lo demás, resuelto el 18/10/2026 en la rama `feature/organization-closure`: la empresa recibe y responde `country: "PE"`, `functionalCurrency: "PEN"` y `taxIdTypeId` (§4.2, D-13) | El nombre `id`: §14.8 |
| 22 | ~~La dirección embebida en la empresa nombra el país con `countryId` (UUID) más `countryIsoAlpha2` y `countryName`~~ | Resuelto el 18/10/2026 en la rama `feature/organization-closure`: la dirección embebida en empresas, sedes y ubicaciones nombra el país con `country: "PE"` y `countryName` (§3.3, D-13) | — |

---

## 14. Decisiones pendientes

Todo lo que este documento fija y no remite al diccionario ni a una regla de arquitectura es una propuesta hasta que se valide. Se aplica a lo nuevo, y a lo construido (§13), solo cuando se valide; mientras tanto, cada una aparece marcada en el cuerpo. Las más importantes:

**Forma general**

1. ~~**Prefijo de contexto en las rutas**~~. Rechazado el 06/10/2026: las rutas no llevan el contexto (§2.1). Venía de *Contextos delimitados* v1.8 y de la plantilla de la skill `backend-module`, que ya no lo enseña. Hay un solo despliegue y un solo cliente, ningún recurso repite nombre entre contextos y el cambio se encarecía con cada contexto construido. La autenticación sigue en `/api/v1/authentication/...` (diccionario §5.12).
2. **Formato de los montos**: objeto `{ amount, currency }` con `amount` como texto decimal; los tipos de cambio y las tasas, también como texto decimal (§5.2).
3. **Nombres de los campos de un período**: `periodStart` y `periodEnd` (§5.1). El diccionario fija que viajan como dos campos, pero no cómo se llaman.
4. ~~**Página**~~. Resuelto el 06/10/2026, a pedido: toda lista, también los catálogos chicos, se busca, filtra, ordena y pagina en su propio endpoint, con el sobre `{ items, page, size, totalItems, totalPages }`; sin `page` responde el arreglo completo (§6.2).
5. **Campo `rule`** en los `422` (§7.2), que obliga a ampliar `ErrorResource` y `ApplicationError`.
6. ~~**`404` para una empresa ajena**~~. Adoptado el 06/10/2026 con la lectura de recursos de empresa (§9.2): una empresa ajena responde `404`, no `403`. Construido el 16/10/2026 en empresas, marcas, sedes, ubicaciones, colaboradores y `/users?companyId`, con su prueba de integración (`CompanyIsolationIntegrationTests`).
7. **Concurrencia**: si dos personas editan el mismo registro, en la mayoría de los recursos gana la última. Una opción es exigir `If-Match` con la versión del recurso; no hace falta mientras los catálogos los edite solo el superadministrador. Las fichas que más se editan a la vez ya no la necesitan: el proveedor, la plantilla, el borrador de una orden y el contacto de un proveedor llevan `updatedAt` en el cuerpo y responden `422 <RECURSO>_OUTDATED` (§3, §7.2; la orden y el contacto, desde el 29/10/2026).
8. **Nombre del campo propio: `id` y no `publicId`** (§4.2). D-13 fija que viaja el `public_id`, no cómo se llama el campo. Desde el 05/10/2026 todos los recursos construidos usan `id`, también escalones, divisiones y tipos de documento. Las empresas todavía usan `publicId` (§13 fila 21). La dirección embebida no lleva identificador propio (§3.3).
9. **Códigos de éxito**: `201` con `Location` para un alta y para un hecho que se suma a una colección; `200` con el recurso afectado para un hecho en singular y para la mudanza de una dirección (§3, §3.3).
10. **Filtros**: nombres iguales al campo (`isActive`, `countryId`); `404` cuando el dueño de la colección no existe; `companyId` obligatorio cuando la cuenta tiene acceso a más de una empresa (§6.1).
11. **Códigos de error.**
    - ~~`<RECURSO>` con guion bajo entre palabras (`GEO_DIVISION_*`)~~. Rechazado el 20/10/2026 por D-114: cada agregado lleva un solo prefijo, el que producen las fábricas, sin guion bajo interno (§7.2).
    - Los errores del framework, con dos opciones. Opción a: ampliar la tabla de §7.2 con `NOT_FOUND` (`404`, ruta inexistente), `METHOD_NOT_ALLOWED` (`405`) y `UNSUPPORTED_MEDIA_TYPE` (`415`), y traducir su `message`. Opción b: llevarlos a los códigos existentes (`VALIDATION_ERROR` para el parámetro faltante, etc.). Mientras no se decida, el `405` de §3.1 lleva `METHOD_NOT_ALLOWED`.
    - Que el `405` lleve el encabezado `Allow`; hoy el manejador global no lo envía.
12. ~~**Textos de error por `code`**~~. Adoptado el 06/10/2026: el frontend traduce cada `code` a una clave de `es.json`, y un código desconocido cae en el mensaje genérico de la operación (§7.1). El backend acuña `<RECURSO>_<CAMPO>_CONFLICT` cuando un agregado tiene dos o más reglas de unicidad (§7.2). Si se adopta `rule` (§14.5), los `422` se traducen igual.
13. **`null` en lugar de omitir** un campo opcional (§4.1).
14. **Política de versiones**: cuándo se abre una `v2` (§1).

**Rutas puntuales**

15. ~~**Dar acceso a un colaborador**~~. Decidido el 15/10/2026 y construido: `POST /access-grants` con `employeeId` y `email`. Crea la cuenta, o vincula el colaborador a la que ya tiene ese correo, en una transacción (D-38; mapa §6.2). La contraseña provisional la genera el sistema: no viaja en la petición. Si crea la cuenta responde `201` con `{userId, email, created: true, temporaryPassword}`, y la contraseña se muestra una sola vez; si vincula una existente responde `200` con `created: false` y `temporaryPassword: null`, y nunca cambia sus roles ni su contraseña. Los roles no viajan aquí: se otorgan después en `/users/{id}/role-assignments`. Los códigos de error están en §7.2.
    - **La cuenta de otra persona** (D-82, construido el 18/10/2026). Una cuenta existente se vincula sola solo si ya pertenece a un colaborador con la misma sigla de tipo y el mismo número de documento. Si no, responde `422 EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH`, y al superadministrador con `context`: sus titulares y su huella (§7.1). El superadministrador la vincula reenviando el mismo `POST` con dos campos opcionales más: `{employeeId, email, identityMismatchConfirmed: true, holdersFingerprint: "<la del 422>"}`. Vincula solo si los titulares leídos ahora, con la cuenta bloqueada, dan la misma huella, y responde `200` con `created: false`. Sin huella, o si los titulares cambiaron mientras tanto, no vincula y responde otra vez el `422` con los titulares y la huella de ahora, para que la pantalla los muestre y pida confirmar de nuevo. Cualquier otro rol recibe el `422` sin `context`, mande lo que mande. Cada vínculo confirmado, con los titulares que vio el superadministrador, y cada desvínculo (§3.2) quedan registrados en `employee_account_link_events` (diccionario §5.4); no hay ruta para leer ese registro.
16. **Listas públicas del registro (D-32).** El diccionario (§5.2) exige que el formulario liste sin sesión la empresa y la sede, pero no fija la ruta de las sedes. Tampoco decide cómo obtiene sin sesión los tipos de documento de las personas, aunque `employees.document_type_id` es obligatorio y `/countries/{id}/tax-id-types` exige un rol (D-43).
    - Propuesta: `GET /authentication/sign-up/companies/{companyId}/branches` y `GET /authentication/sign-up/document-types`, públicas, solo con el registro encendido (con el registro apagado, `403` como `sign-up`) y solo con `id` y `name` de los registros activos.
    - Alternativa: que `sign-up/companies` devuelva cada empresa con sus sedes y los tipos de documento de personas de su país.
17. ~~**Ruta de la salida de un colaborador**~~. Resuelto el 15/10/2026 con la opción A de la decisión 2 del [mapa de contextos](mapa-de-contextos-millennium-mobility.md): el caso de uso vive en `iam`, y la ruta se queda en `POST /employees/{id}/termination`, igual que `POST /employees/{id}/reactivation`, porque las rutas no llevan contexto (§2.1). La salida recibe `{terminationDate}`, el regreso `{hireDate}`, y los dos responden `{id, isActive, hireDate, terminationDate}`. El motivo de la salida salió el 17/10/2026: es un candidato del diccionario (§5.10).
18. **Cargas que actualizan la flota.** El diccionario pone `import_batches` en `shared` (§3, §11.8). Pero las cargas del impuesto vehicular y de la garantía mobiliaria cruzan por placa y escriben `vehicle_tax_declarations` y `vehicle_fund_assignments`, que son de `fleet`.
    - Opción A (propuesta): la carga entra por el contexto dueño de los datos (`POST /tax-declaration-imports`, `POST /fund-assignment-imports`). `fleet` hace el cruce y usa `shared` solo para registrar la carga y guardar el archivo y el archivo de errores.
    - Opción B: `POST /imports` sigue en `shared`, que define un puerto de procesamiento por `import_type` que `fleet` implementa.

    Las dos evitan el ciclo (mapa §6.5 y decisión 8).
19. **Costos de una unidad para su ficha**: `GET /reports/vehicle-costs?vehicleId=…` en la capa de lectura. Entrega el valor de factura, las notas de crédito y el valor del vehículo calculado, y omite los montos sin `stock.ver_costos` (§9.2; mapa, decisión 7).
20. **Anulación de una unidad.** Decidido el 19/10/2026 (D-104): una unidad se anula por las dos vías, como consecuencia de una nota de crédito que deja su valor en cero, gobernada por `compras.registrar_nota_credito`, o por acción explícita, que exige `stock.anular_vehiculo` y el candado de §9.6 (valor cero y ningún contrato). Queda como propuesta solo la ruta: `POST /vehicles/{id}/annulment`, con `stock.anular_vehiculo` y respetando el candado; `procurement` la pide a través de la fachada de `fleet` (mapa §6.1).
21. **Correcciones delicadas** (§3.2). Cada una va por su propio subrecurso con `reason` obligatorio. Falta decidir:
    - qué monto se corrige: el diccionario dice que `invoiced_amount` de una línea no se modifica nunca, y que el total debe coincidir con la suma de las líneas;
    - qué fechas cuentan como "ya reportadas" y por qué ruta se corrigen (por ejemplo, el período de una cesión a fondo);
    - qué permiso exige cada una: el catálogo (diccionario §5.6, D-95) define `stock.corregir_placa` y `stock.corregir_vin`; las correcciones de un monto de factura y de una fecha ya reportada llevarán un permiso cada una cuando tengan su ruta, y el diccionario las deja aquí a propósito.
22. **Dos hechos que el diccionario exige sin nombrar su ruta.**
    - El cierre manual de una orden de compra a `cumplida` cuando ninguna factura la referencia (D-21) es `POST /purchase-orders/{id}/fulfillment`, ya construida y exigida por `compras.cumplir_ordenes`.
    - La sustitución de odómetro (diccionario §8.5) se propone como `POST /vehicles/{id}/odometer-replacements`. El diccionario no define qué datos guarda (lectura final del tablero anterior, inicial del nuevo, fecha, motivo) ni si tiene tabla propia, así que la forma del cuerpo también queda pendiente.
23. ~~**Motivos de cambio de estado**~~. Adoptado el 20/10/2026 (D-113): `/status-change-reasons` es una colección filtrable (listar y crear), con `?fromStatusId={id}` para filtrar por origen y `?origin=registration` para los motivos de alta, que no tienen estado de origen (`from_status_id` vacío, diccionario §8.3). No se usa `fromStatusId` ausente para el alta, porque un filtro ausente no filtra (§6.1). Los dos filtros juntos responden `400 VALIDATION_ERROR`, y un `origin` que no sea `registration` también. Acepta además `isActive`. Responde siempre el arreglo completo, sin página: es una de las listas cortas de §6.2.
24. ~~**Subrecurso de dirección**~~. Adoptado el 06/10/2026 (§3.3): la dirección no tiene ruta propia; viaja anidada en el alta del dueño y embebida en su respuesta; `PUT` al subrecurso corrige la misma fila y `POST` crea una fila nueva a la que pasa a apuntar el dueño (D-8); el `PUT` del dueño no la lleva. Construido en empresas, sedes y ubicaciones de terceros (D-79); los proveedores nacen así y corrigen o mudan su domicilio en el `PUT` de su ficha. Las plantillas nacieron así hasta el 26/10/2026; desde entonces copian el domicilio de su empresa (§3.3, D-134).
25. ~~**Cómo se retira un escalón territorial.**~~ Resuelto por D-45: un escalón declarado no se retira; se renombra.
26. **Autor de los hechos**: que los campos que nombran a quien opera (`changed_by`, `granted_by`, `recorded_by`) los llene también el sistema con la cuenta de la sesión y no viajen desde el cliente (§3.2). D-42 cubre solo las columnas de registro. `iam` ya lo hace desde el 16/10/2026 con `granted_by` y con `closed_by` de los bloqueos; falta validarlo como regla para todos los contextos.
