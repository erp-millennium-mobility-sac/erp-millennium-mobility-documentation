# Convenciones de la API REST — ERP Millennium Mobility

**Actualizado:** 19/10/2026 (paso 3b, construido en `feature/permissions-matrix`, diccionario v1.23: las rutas de los roles y del catálogo de permisos, con `RoleDetailResponse`; `permissions` en `/me`; los códigos de los roles y de la matriz; `ACCESS_DENIED` también por el permiso que falta o no actúa, D-99; la política de acceso por permiso en §9.2; los tres campos del colaborador que se omiten, D-105; `role_permissions` sin `public_id`; §14.20 y §14.21 con D-104 y el catálogo) · 18/10/2026 (seguimiento de `organization`, construido en `feature/organization-followup`, diccionario v1.22: el `403` de las lecturas de colaboradores, D-93; `COMPANY_INACTIVE` al reactivar y el orden del regreso, D-94; `EMPLOYEE_TERMINATION_BEFORE_HIRE` en la edición; el `400` de un número hecho solo de separadores; las columnas de las exportaciones de colaboradores y ubicaciones) · 18/10/2026 (cierre de `organization`, diccionario v1.21: la empresa nombra su país y su moneda por su código, D-13, y §13 filas 21 y 22; `locationType` en minúsculas; el campo `context` del error; vincular la cuenta de otra persona con confirmación, D-82; `PUT /companies/{id}/tax-id`, D-85; `POST /employees/{id}/account-unlinking`, D-82; los códigos de lo que se cuelga de algo inactivo, D-84, de la fecha de ingreso, D-88, y de la ubicación propia de una sede, D-90; `employeeCompanyIds`; `sort` por el nombre del campo en ubicaciones y colaboradores) · 18/10/2026 (cierre de `location`: códigos propios de la dirección, del dígito de control y de la ubicación; la ubicación corrige o muda su dirección por su subrecurso, D-79; identificadores sin guiones, espacios ni puntos; un duplicado en carrera responde el código de su recurso; página de 1 a 200; español por omisión; §13 fila 14 resuelta) · 16/10/2026 (rutas, códigos y política de acceso de `iam` del paso 3; `/me`; `/users?companyId` y `managedBySuperadmin`; filtro de alcance construido) · 14/10/2026 (`location` nombra el país y la moneda por su código; escalones e hijos de una división como listas cortas; campos derivados del dueño) · 06/10/2026 (direcciones anidadas en su dueño, errores elegidos por `code`, fechas del Excel en ISO, rutas sin prefijo de contexto) · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.23 (§2.2 normalización, §2.4 identificadores, §2.5 separación por empresa, §2.6 fechas, §4.6 direcciones, §4.7 tipos de documento, §5.3 colaboradores, §5.5 y §5.6 roles y permisos, §5.8 cómo se resuelven, §5.12 sesiones, §6.2 a §6.5, D-8, D-13, D-38, D-40, D-42, D-43, D-49, D-51, D-53, D-75, D-78, D-79, D-82, D-84 a D-91 y D-95 a D-105), reglas de arquitectura de la skill `backend-module`, [mapa de contextos](mapa-de-contextos-millennium-mobility.md) y el backend construido. Si este documento y el diccionario no coinciden, manda el diccionario.

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
| `iam` | `/authentication/sign-in` · `sign-out` · `me` · `sign-up` · `sign-up/status` · `sign-up/companies` · `change-password` · `/users` · `/users/{id}/password-reset` · `/access-grants` · `/users/{id}/role-assignments` · `/role-assignments/{id}/revocation` · `/users/{id}/suspensions` · `/suspensions/{id}/closure` · `/employees/{id}/termination` · `/employees/{id}/reactivation` · `/employees/{id}/account-unlinking` · `/roles` · `/roles/{id}` · `/roles/{id}/status` · `/roles/{id}/permissions` · `/permissions` |
| `organization` | `/companies` · `/companies/{id}` · `/companies/{id}/status` · `/companies/{id}/fiscal-address` · `/companies/{id}/tax-id` · `/companies/{id}/branches` · `/branches/{id}` · `/branches/{id}/status` · `/branches/{id}/address` · `/companies/{id}/commercial-brands` · `/commercial-brands/{id}` · `/commercial-brands/{id}/status` · `/companies/{id}/locations` · `/locations/{id}` · `/locations/{id}/status` · `/locations/{id}/address` · `/companies/{id}/employees` · `/employees/{id}`. `GET /companies?countryId=…` responde si un país opera (D-3) |
| `vehiclecatalog` | `/vehicle-types` · `/vehicle-brands` · `/vehicle-brands/{id}/vehicle-models` · `/vehicle-models/{id}` · `/colors` · `/equipment-types`, cada uno con `/{id}` y `/{id}/status` |
| `fleet` | `/vehicles` · `/vehicles/{id}` · `/vehicles/{id}/plate` · `/vehicles/{id}/vin` · `/vehicles/{id}/annulment` **(propuesta, §14.20)** · `/vehicles/{id}/status-changes` · `/status-changes/{id}` · `/vehicles/{id}/mileage-readings` · `/mileage-readings/{id}` · `/vehicles/{id}/odometer-replacements` **(propuesta, §14.22)** · `/vehicles/{id}/documents` · `/vehicle-documents/{id}` · `/vehicle-documents/{id}/file` · `/vehicles/{id}/equipments` · `/vehicle-equipments/{id}` · `/vehicle-equipments/{id}/removal` · `/vehicles/{id}/keys` · `/vehicle-keys/{id}` · `/vehicles/{id}/fund-assignments` · `/vehicles/{id}/tax-declarations` · `/vehicle-statuses` · `/vehicle-statuses/{id}` · `/vehicle-statuses/{id}/status` · `/status-change-reasons` **(propuesta, §14.23)** · `/status-change-reasons/{id}` · `/status-change-reasons/{id}/status` · `/document-types` · `/document-types/{id}` · `/document-types/{id}/status` · `/funds` · `/funds/{id}` · `/funds/{id}/status` · `/tax-declaration-imports` y `/fund-assignment-imports` **(propuesta, §14.18)** |
| `procurement` | `/suppliers` · `/suppliers/{id}/status` · `/suppliers/{id}/address` · `/suppliers/{id}/balance` · `/purchase-order-types` · `/purchase-order-types/{id}/status` · `/purchase-order-templates` · `/purchase-order-templates/{id}/status` · `/purchase-order-templates/{id}/issuer-address` · `/purchase-orders` · `/purchase-orders/{id}/sending` · `/purchase-orders/{id}/annulment` · `/purchase-orders/{id}/fulfillment` **(propuesta, §14.22)** · `/purchase-orders/{id}/pdf` · `/purchase-invoices` · `/purchase-invoices/{id}/annulment` · `/credit-notes` · `/supplier-payments` · `/supplier-payments/{id}/annulment`, cada colección con `/{id}` |
| Plataforma (`shared`) | `/change-log` · `/imports` · `/imports/{id}` · `/imports/{id}/error-report` |
| Capa de lectura | `/reports/stock-export` · `/reports/day-gaps` · `/reports/down-payment-differences` · `/reports/vehicle-costs?vehicleId=…` **(propuesta, §14.19)** |

- Los recursos de empresa sin anidar (`/vehicles`, `/funds`, los de `procurement`) reciben `companyId` como filtro (§6.1).
- `location` no expone `/addresses`: una dirección se escribe y se lee solo a través de su dueño (§3.3).
- Toda tabla del diccionario con `is_active` expone `PATCH /{id}/status` sobre su ruta de elemento. Es la única forma de retirarla, porque no hay `DELETE` (§3.1). Los escalones territoriales no tienen `is_active` ni se retiran: se declaran en orden y se renombran (D-45).
- `/users` solo lista y exporta: no hay alta suelta de cuentas ni lectura de una cuenta por su identificador. Las cuentas nacen al dar acceso a un colaborador (`POST /access-grants`, §14.15) o por el registro público (D-32, D-38). `SYSTEM` nace en la migración y el primer superadministrador al arrancar (D-34), ninguno por la API.
- Una asignación de rol y un bloqueo no tienen ruta de elemento para leerlos: se listan desde su cuenta (`/users/{id}/role-assignments`, `/users/{id}/suspensions`) y solo tienen el subrecurso de su hecho (`revocation`, `closure`).
- La salida y el regreso de un colaborador cuelgan de `/employees/{id}`, pero los sirve `iam`, que los orquesta (§14.17; mapa de contextos, decisión 2). Como las rutas no llevan contexto (§2.1), el cambio de dueño no cambió la ruta. Lo mismo `POST /employees/{id}/account-unlinking`, que deshace el vínculo equivocado de un colaborador con una cuenta (D-82): la sirve `iam`, porque decide qué pierde la cuenta.
- `GET /companies/{id}/employees/metrics` ya no existe desde el 18/10/2026: ninguna pantalla lo leía.
- Los roles y el catálogo de permisos (construido el 19/10/2026, diccionario §5.5, §5.6, D-95 a D-98): `GET /roles` responde el arreglo completo, sin página y sin las matrices, con los roles que quien llama ve (D-98) ordenados por código; `GET /roles/{id}` trae además `permissionCodes`, los permisos que el rol incluye ahora. Un rol propio se crea con `POST /roles` (`201` con `Location`), se renombra con `PUT /roles/{id}`, se deja de ofrecer o se ofrece otra vez con `PATCH /roles/{id}/status`, y su matriz se reemplaza entera con `PUT /roles/{id}/permissions`; las cuatro escrituras responden el mismo `RoleDetailResponse`, el rol con sus permisos vigentes. `GET /permissions`, entero o por `?module=`, solo lista el catálogo: ningún permiso se crea ni se edita por la API, los cargan las migraciones (D-95). Las matrices de los siete roles del sistema no se escriben (D-97).
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

### 3.1 Por qué no hay `DELETE`

Nada se borra en este sistema: los catálogos se desactivan, los accesos se retiran, los documentos se reemplazan, los vehículos y comprobantes se anulan y los períodos se cierran (regla 4.2). Los repositorios del dominio no exponen `delete`, y la API tampoco. **(propuesta, §14.11)** Un `DELETE` sobre cualquier ruta existente responde `405` con el código `METHOD_NOT_ALLOWED` (§7.2).

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

**Deshacer el vínculo de una cuenta** (D-82, construido el 18/10/2026). `POST /employees/{id}/account-unlinking`, sin cuerpo, solo para un superadministrador vinculado a un colaborador activo. Funciona como una salida: retira los roles de la cuenta en la empresa del colaborador con motivo `retiro` y, si a la cuenta no le queda ningún colaborador activo, la cierra como la salida de su último colaborador (`baja`, `baja_cuenta` para sus demás roles y sesiones cerradas), con la protección del último superadministrador. Responde `200` con `{employeeId, userId, email, accountClosed}`; `accountClosed` dice si la cuenta quedó cerrada. Después el colaborador puede recibir su propio acceso con `POST /access-grants`. Los códigos están en §7.2.

**Quién y cuándo.** Las columnas de registro (`created_at`, `updated_at`, `created_by` y `updated_by`) nunca viajan desde el cliente. Las llena el sistema con la cuenta de la sesión, o con `SYSTEM` de forma explícita cuando no hay sesión (D-42). Los datos del hecho que el diccionario pide a quien registra sí viajan en el cuerpo:
- el momento de una lectura de kilometraje (`readAt`, que no es la fecha del registro, diccionario §8.5);
- el inicio de un bloqueo programado por adelantado (diccionario §5.4);
- quién instaló un equipo (`installedByEmployeeId`, diccionario §11.4).

Que el sistema llene también los campos de autor de un hecho que nombran a quien opera (`changed_by`, `granted_by`, `recorded_by`) es una propuesta (§14.26): D-42 no los cubre. `iam` ya la sigue: `granted_by` de una asignación y `closed_by` de un bloqueo salen de la cuenta de la sesión, y ningún cuerpo los trae. Desde el 18/10/2026 la asignación responde a sus autores por su `public_id` y por su correo, para mostrarlos: `grantedBy` con `grantedByEmail`, y `revokedBy` con `revokedByEmail`, `null` mientras sigue vigente; lo que venció lo firma `system@millennium.ky`. Se muestra el correo y no el nombre, porque el nombre es de `organization` (mapa de contextos §11, decisión 6).

Un hecho que se cierra lo hace en el momento de la petición: `POST /suspensions/{id}/closure` y `POST /role-assignments/{id}/revocation` no llevan cuerpo. Una fecha elegida por el cliente podría falsear quién tenía acceso (diccionario §5.4).

**La autenticación es la excepción.** `sign-in`, `sign-out`, `sign-up` y `change-password` son operaciones, no recursos, y conservan sus nombres (diccionario §5.12).

### 3.3 Las direcciones

Las direcciones no tienen rutas propias: no existe `/addresses`. Viajan anidadas en el `POST` que crea a su dueño (empresa, sede, ubicación, proveedor, plantilla) y embebidas en sus respuestas, y quien puede escribir o leer al dueño es quien puede escribir o leer su dirección. Para distinguir corregir de mudarse (D-8), cada dueño ofrece un subrecurso de dirección (decidido el 06/10/2026, §14.24):
- `/companies/{id}/fiscal-address`;
- `/branches/{id}/address`;
- `/locations/{id}/address`;
- `/suppliers/{id}/address`;
- `/purchase-order-templates/{id}/issuer-address`.

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
- El país no viaja: lo fija el dueño. La dirección de una empresa, una sede o una ubicación está en el país de la empresa (diccionario §4.6). La base lo exige en empresas y sedes con una llave foránea compuesta hacia `addresses (id, country_id)`; en una ubicación lo asegura la aplicación, porque `locations` no tiene columna de país. El proveedor, cuando exista, mandará su propio país.
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
- Una respuesta puede acompañar la referencia con lo que la pantalla necesita mostrar, para no obligar a otra petición: `"companyId"` con `"companyName"`, `"parentId"` con `"parentName"`.
- Una respuesta también puede traer un dato derivado que la pantalla necesita para decidir, calculado por el dueño del recurso y nunca guardado. Hoy hay dos: `isoCodesLocked` del país dice si sus códigos ISO todavía se pueden corregir (diccionario §4.2, D-47), y `codeLocked` del tipo de equipamiento dice si su código todavía se puede corregir (diccionario §7.5, D-74). La pantalla lo usa para bloquear los campos; la API rechaza la corrección igual.

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
| `q` | Texto que contiene alguno de los campos de búsqueda de la lista, sin distinguir mayúsculas. `%` y `_` se buscan tal cual. Todavía distingue tildes: "peru" no encuentra "Perú" **(pendiente)**: plegarlas antes de la primera lista que se busque en el servidor mientras se escribe (proveedores, paso 7), como ya lo hace el buscador del frontend con las listas cargadas |
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

**Dos listas cortas, como excepción** (registrada el 14/10/2026). `GET /countries/{id}/geo-division-levels`, que tiene a lo sumo cuatro escalones, y `GET /geo-divisions/{id}/children`, que trae las divisiones hijas de una para el selector en cascada de la dirección, responden siempre el arreglo completo, sin `q`, `sort` ni `page`. Son cortas por naturaleza y solo pueblan un select. Los escalones llegan por número y los hijos por `code`; los hijos aceptan el filtro `isActive`.

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
- `context` es opcional (registrado el 18/10/2026): lleva los datos que el cliente necesita para actuar sobre el error, por ejemplo lo que tiene que mostrar antes de pedir una confirmación. Solo lo traen los errores que lo documentan; en todos los demás no viaja, ni siquiera como `null`. Hoy lo trae uno solo, `422 EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH` de `POST /access-grants`, y solo cuando quien llama es superadministrador: cualquier otro rol, también el administrador de accesos, recibe el mismo `422` sin `context`, porque los titulares son personas de otras empresas y solo un superadministrador decide sobre ese vínculo (D-82).

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
| `ACCESS_DENIED` | `403` | Hay sesión pero falta el rol o el permiso de la ruta, o, desde el paso 3b, el permiso no actúa en la empresa alcanzada: se tiene por un rol de otra empresa (D-99); también con contraseña provisional y con el registro público apagado. En las operaciones de cuenta, es "tu rol no puede hacer esto", incluido otorgar un rol reservado al superadministrador |
| `ACCOUNT_SUSPENDED` | `403` | La contraseña coincidió, pero la cuenta tiene un bloqueo en curso (diccionario §5.4) |
| `ACCOUNT_MANAGED_BY_SUPERADMIN` | `403` | Quien opera maneja cuentas, pero la cuenta es protegida —tiene `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente— y solo la maneja un superadministrador |
| `OWN_ACCOUNT_FORBIDDEN` | `403` | Quien opera actúa sobre su propia cuenta —otorgar, retirar, bloquear, cerrar un bloqueo, restablecer, vincularse a sí mismo fuera del arranque— o registra su propia salida o regreso |
| `ACTOR_EMPLOYEE_REQUIRED` | `403` | La cuenta de quien opera no está vinculada a un colaborador activo, así que no puede actuar sobre cuentas ni registrar salidas o regresos; tampoco un superadministrador sin vincular (D-89) |
| `<RECURSO>_NOT_FOUND` | `404` | El recurso, o uno que la petición referencia, no existe |
| `<RECURSO>_CONFLICT` | `409` | Ya existe un valor único: un código ISO, un número de comprobante |
| `<RECURSO>_<CAMPO>_CONFLICT` | `409` | Lo mismo, en un agregado con dos o más reglas de unicidad: cada una tiene su código (`VEHICLE_VIN_CONFLICT`, `VEHICLE_PLATE_CONFLICT`) |
| `METHOD_NOT_ALLOWED` **(propuesta, §14.11)** | `405` | El método no existe para esa ruta; en particular, todo `DELETE` (§3.1) |
| `BUSINESS_RULE_VIOLATION` | `422` | La petición es válida pero rompe una regla del negocio: una lectura de kilometraje menor, un camino de estado no permitido, una nota de crédito en otra moneda |
| `UNEXPECTED_ERROR` | `500` | Un fallo que no debería pasar |

- Un fallo esperado del negocio se devuelve como `Result`, nunca como excepción (regla 3.1), y se traduce aquí.
- Las fábricas de `ApplicationError` (`notFound`, `conflict`, `businessRuleViolation`, `validationError`…) producen los códigos genéricos de la tabla. Por omisión, un duplicado es `ApplicationError.conflict(...)`, que responde `<RECURSO>_CONFLICT`.
- **Un duplicado que llega a la base en una carrera responde el código de su recurso**, el mismo que el servicio da sin carrera. Dos peticiones simultáneas pueden pasar las dos la comprobación del servicio; la restricción única de la base detiene a la segunda, y el adaptador del repositorio traduce esa violación en el fallo de su clave natural ([patrones de persistencia](patrones-de-persistencia-millennium-mobility.md)). En `location` responden así `CURRENCY_CONFLICT`, `COUNTRY_ISO_ALPHA2_CONFLICT`, `COUNTRY_ISO_ALPHA3_CONFLICT`, `GEODIVISION_CONFLICT` y `TAXIDTYPE_CONFLICT` (`409`). En `organization`, desde el 18/10/2026, `COMPANY_COUNTRY_CONFLICT`, `COMPANY_TAX_ID_CONFLICT`, `BRANCH_CODE_CONFLICT`, `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT`, `COMMERCIALBRAND_CONFLICT`, `LOCATION_CODE_CONFLICT`, `LOCATION_BRANCH_CONFLICT`, `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` y `EMPLOYEE_ACCOUNT_IN_COMPANY_CONFLICT`. Solo un duplicado que su adaptador todavía no traduce llega al manejador global y responde el genérico `RESOURCE_CONFLICT` (`409`), sin el texto de la base.
- Cuando un agregado tiene dos o más reglas de unicidad, cada una lleva su código propio `<RECURSO>_<CAMPO>_CONFLICT`, que produce `ApplicationError.conflict(recurso, campo, …)`, para que el cliente sepa cuál falló: `COMPANY_COUNTRY_CONFLICT` y `COMPANY_TAX_ID_CONFLICT` en la empresa, que ya los emite; `VEHICLE_VIN_CONFLICT` y `VEHICLE_PLATE_CONFLICT` en la unidad; `BRANCH_CODE_CONFLICT` y `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT` en la sede (diccionario §6.4). Un agregado con una sola regla se queda con el genérico.
- El estado de un código propio sale de su sufijo: `_NOT_FOUND` responde `404`, `_CONFLICT` responde `409` y cualquier otro, `422`. Por eso un código propio de duplicado termina siempre en `_CONFLICT`: con otro sufijo saldría como regla rota. Un código propio que no es un duplicado, como `COMPANY_FISCAL_ADDRESS_MISSING` (§3.3), responde `422`. Los cuatro `403` propios de la tabla (`ACCOUNT_SUSPENDED`, `ACCOUNT_MANAGED_BY_SUPERADMIN`, `OWN_ACCOUNT_FORBIDDEN` y `ACTOR_EMPLOYEE_REQUIRED`) son la excepción: no tienen un sufijo que dé su estado, así que `ErrorResponseAssembler` los nombra uno por uno.
- Un `403` dice por qué cuando la pantalla tiene que explicarlo: cada causa tiene su código, y `ACCESS_DENIED` queda para la falta de rol o de permiso, y para el permiso que no actúa en la empresa (D-99). El rol o el permiso de la ruta se revisa antes que la existencia: quien no lo tiene en ninguna empresa recibe `403` aunque el recurso no exista; la exportación de colaboradores lo exige en su método, así que quien no tiene `configuracion.ver_datos_personales_colaboradores` en ninguna empresa recibe `403` antes de que se busque la empresa. Quien lo tiene en otra empresa recibe `404 COMPANY_NOT_FOUND` por una fuera de su alcance, y `403` por una alcanzada donde el permiso no actúa (D-99).
- Como todo `Result.failure` es un fallo esperado, un código propio nunca responde `500`. Su texto en el servidor se busca en `error.<código-en-kebab>.message`, por ejemplo `error.vehicle-vin-conflict.message`, y el frontend tiene su propia clave en `es.json` (§7.1).
- `<RECURSO>` es el nombre del agregado en mayúsculas. Hoy las fábricas lo pasan a mayúsculas sin separar palabras: `notFound("GeoDivision", …)` produce `GEODIVISION_NOT_FOUND`. **(propuesta, §14.11)** Separar las palabras con guion bajo (`GEO_DIVISION_NOT_FOUND`, `PURCHASE_INVOICE_CONFLICT`), lo que obliga a cambiar las fábricas.
- **(propuesta, §14.5)** Un `422` dice qué regla falló en `rule`, un identificador estable: `"rule": "mileage-reading.not-lower"`. Así el cliente distingue reglas sin leer el texto.

**Los códigos del documento tributario y de la ubicación** (registrado el 18/10/2026). Todos responden `422`. Los de la dirección están en §3.3.

| `code` | Operación | Campo | Cuándo |
|---|---|---|---|
| `COMPANY_TAX_ID_CHECK_DIGIT_INVALID` | `POST /companies`, `PUT /companies/{id}/tax-id` | `taxId` | El tipo tiene dígito de control y el número no lo cumple (D-75): el RUC `20512345678`, cuyo dígito correcto es `1`. Un tipo con dígito de control cuyo cálculo `location` todavía no tiene —el NIT, el RFC y la CURP— acepta el número sin verificarlo |
| `EMPLOYEE_DOCUMENT_NUMBER_CHECK_DIGIT_INVALID` | `POST /companies/{id}/employees`, `PUT /employees/{id}` | `documentNumber` | Lo mismo, para el documento del colaborador. En la edición se verifica solo si cambian el tipo o el número: un documento ya guardado sigue siendo válido |
| `EMPLOYEE_TERMINATION_BEFORE_HIRE` | `PUT /employees/{id}` | `hireDate` | La edición deja la fecha de ingreso de un colaborador que ya salió después de su salida (registrado el 18/10/2026). Es el mismo código que responde una salida anterior al ingreso (más abajo) |
| `LOCATION_OWNERSHIP_CHANGE_NOT_ALLOWED` | `PUT /locations/{id}` | `locationType` | La edición pasa una sede propia a ubicación de terceros o al revés (D-79). Entre los tipos de terceros sí se cambia |
| `LOCATION_ADDRESS_MISSING` | `PUT /locations/{id}/address` | — | La ubicación no tiene dirección que corregir; la primera se registra con `POST` |
| `LOCATION_ADDRESS_NOT_ALLOWED` | `PUT` y `POST /locations/{id}/address` | — | Es una sede propia: usa la dirección de su sede, que se corrige o se muda en `/branches/{id}/address` |

- El número se revisa antes que los duplicados: un RUC mal escrito responde `422` (`COMPANY_TAX_ID_CHECK_DIGIT_INVALID`, o `BUSINESS_RULE_VIOLATION` por el tipo o la longitud) aunque el país ya tenga su empresa, en vez de `409`. Lo mismo el documento de un colaborador.
- Los demás rechazos del tipo de documento —de otro país, inactivo, que no aplica a empresas o a personas, de otra longitud— siguen siendo `BUSINESS_RULE_VIOLATION`. Un tipo inactivo se rechaza solo cuando es nuevo para el registro: el colaborador que ya lo tenía se sigue editando (LOC-R36).
- La longitud que declara un tipo de documento va de 1 a 20 (D-78): con 21 o más, `400 VALIDATION_ERROR`.
- Un número hecho solo de separadores, como `-.-`, queda vacío al limpiarlo y responde `400 VALIDATION_ERROR` en su campo, con cualquier tipo, también el RUC (registrado el 18/10/2026): `taxId` en `POST /companies` y `PUT /companies/{id}/tax-id`, y `documentNumber` en `POST /companies/{companyId}/employees` y `PUT /employees/{id}`. Antes respondía `422 BUSINESS_RULE_VIOLATION` en el colaborador, y en la empresa un `422` o un `400` sin campo según el tipo.
- La longitud del número, como su dígito de control, se mide sin separadores y solo al crear el registro o cuando cambian el número o el tipo (D-91): editar un colaborador cuyo documento no cambia no lo vuelve a medir.

**Los códigos de lo que se cuelga de una empresa, una sede o una marca** (registrado el 18/10/2026). A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo (D-84); lo que ya la tenía se sigue editando mientras no cambie ese vínculo. Todos responden `422` salvo que se indique otro estado.

| `code` | Operación | Cuándo |
|---|---|---|
| `COMPANY_INACTIVE` | `POST /companies/{id}/branches`, `/commercial-brands`, `/locations` y `/employees`; `POST /users/{id}/role-assignments` | La empresa está inactiva |
| `COMPANY_INACTIVE` | `PATCH /branches/{id}/status`, `/commercial-brands/{id}/status` y `/locations/{id}/status` con `{"isActive": true}` | Se reactiva una sede, una marca o una ubicación inactiva cuya empresa está inactiva: reactivar es volver a colgar (D-94), y primero se reactiva la empresa. En una ubicación propia, la empresa se revisa antes que la sede, así que responde este código antes que `LOCATION_BRANCH_INACTIVE`. Una que ya está activa no se revisa, y desactivar no espera a nada (registrado el 18/10/2026) |
| `COMPANY_INACTIVE`, `BRANCH_INACTIVE` | `POST /employees/{id}/reactivation` | El colaborador regresaría a una empresa inactiva o, si tiene sede, a una sede inactiva (D-94). No regresa, y su cuenta sigue cerrada. El orden del regreso está en la tabla de la salida y el regreso (registrado el 18/10/2026) |
| `BRANCH_INACTIVE` | El alta de un colaborador o de una sede propia; su edición cuando cambia de sede; `POST /users/{id}/role-assignments` | La sede es nueva para el registro y está inactiva. La sede inactiva que ya tenía se conserva |
| `BRANCH_OTHER_COMPANY` | El alta y la edición de un colaborador o de una sede propia | La sede es de otra empresa. Antes respondía `422 BUSINESS_RULE_VIOLATION` en el colaborador y `400 VALIDATION_ERROR` en la ubicación |
| `COMMERCIALBRAND_INACTIVE` | `POST /users/{id}/role-assignments` | La marca está inactiva |
| `EMPLOYEE_HIRE_DATE_IN_FUTURE` | `POST /companies/{id}/employees`, `PUT /employees/{id}` | La fecha de ingreso es posterior a hoy en la zona de la sede del colaborador o, sin sede, del país de su empresa (D-88) |
| `LOCATION_BRANCH_CONFLICT` (`409`) | `POST /companies/{id}/locations`, `PUT /locations/{id}` con `sede_propia` | La sede ya tiene su ubicación propia, activa o inactiva (D-90): la inactiva se reactiva, no se crea otra |
| `LOCATION_BRANCH_INACTIVE` | `PATCH /locations/{id}/status` con `{"isActive": true}` | Se reactiva una sede propia cuya sede está inactiva (D-90): primero se reactiva la sede. Una ubicación de terceros no tiene sede y no se frena |

- `LOCATION_CODE_CONFLICT` y `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` (`409`) tienen su propio mensaje desde el 18/10/2026; antes salían con el genérico «Conflicto de recurso».
- `photoPath` llega hasta 500 caracteres (V43): con más, `400 VALIDATION_ERROR` sobre ese campo, en el alta y la edición del colaborador. Antes respondía `422 BUSINESS_RULE_VIOLATION` sin campo.
- Los códigos de una sede o una ubicación conservan sus guiones: solo se recortan y se pasan a mayúsculas, así `LIM-1` y `LIM1` son dos códigos distintos (D-86, diccionario §2.2). Los separadores se quitan solo de los identificadores de una norma externa (§5.3).

**Los códigos de los roles y de la matriz de permisos** (registrado el 19/10/2026; diccionario §5.5, §5.6, D-96 a D-98). Los escribe `iam`; el único que escribe roles es el superadministrador (D-98).

| `code` | Estado | Operación | Cuándo |
|---|---|---|---|
| `ROLE_CONFLICT` | `409` | `POST /roles` | La empresa, o los roles generales, ya tiene un rol con ese código; también cuando dos altas chocan en la base |
| `ROLE_CODE_RESERVED` | `422` | `POST /roles` | El código es el de uno de los siete roles del sistema |
| `ROLE_SCOPE_INVALID` | `422` | `POST /roles` | Un rol de una empresa con alcance `SYSTEM`: su alcance es `COMPANY` o `BRANCH` |
| `COMPANY_INACTIVE` | `422` | `POST /roles` | La empresa del rol está inactiva (D-84) |
| `ROLE_SYSTEM_IMMUTABLE` | `422` | `PUT /roles/{id}`, `PATCH /roles/{id}/status`, `PUT /roles/{id}/permissions` | El rol es del sistema: su nombre, su estado y su matriz los fijan las migraciones (D-97) |
| `ROLE_PERMISSIONS_NOT_HELD` | `422` | `PUT /roles/{id}/permissions` | Quien marca no posee alguno de los permisos; los que faltan viajan en `context.permissionCodes`, y no se marca nada (§5.5 regla 1). El cuerpo está documentado como `RolePermissionsNotHeldError` |
| `PERMISSION_NOT_FOUND` | `404` | `PUT /roles/{id}/permissions` | Un código que el catálogo no tiene; no se marca nada |
| `ROLE_NOT_FOUND` | `404` | `GET /roles/{id}` y las tres escrituras | No existe, o es de una empresa que quien llama no administra (D-98) |
| `ROLE_INACTIVE` | `422` | `POST /users/{id}/role-assignments` | El rol propio está desactivado: ya no se otorga, y lo que tenía asignado sigue vigente (D-97) |
| `ROLE_ASSIGNMENT_COMPANY_MISMATCH` | `422` | `POST /users/{id}/role-assignments` | Un rol de una empresa se otorga en otra (D-98). Sin empresa, el nivel del rol responde antes `ROLE_ASSIGNMENT_SCOPE_MISMATCH` |

- `POST /roles` revisa en este orden: el rol de quien llama (`403`), `ROLE_CODE_RESERVED`, la empresa (`404 COMPANY_NOT_FOUND`, `COMPANY_INACTIVE`, `ROLE_SCOPE_INVALID`), `ROLE_CONFLICT` y la forma del código y el nombre (`400 VALIDATION_ERROR`: de 2 a 40 mayúsculas, dígitos o guiones bajos; el nombre hasta 80). `PUT /roles/{id}/permissions` exige `permissionCodes`, también vacío (`400`); la ronda de correcciones de la revisión adversarial, pendiente de llegar a la rama al 19/10/2026, valida además cada elemento, y un código nulo o en blanco responde `400 VALIDATION_ERROR` en vez de buscarse en el catálogo, y hace que la documentación de `context` en `ErrorResource` (§7.1) nombre también a `ROLE_PERMISSIONS_NOT_HELD`.
- Los tres administradores leen `GET /roles`, `GET /roles/{id}` y `GET /permissions`; cualquier otro rol recibe `403 ACCESS_DENIED`, y también el administrador de accesos y el de empresa en cada escritura.

**Los códigos de las cuentas y de la salida y el regreso** (registrado el 16/10/2026). Cada causa que la pantalla tiene que distinguir lleva su código, y el frontend tiene un mensaje para cada uno. Todos responden `422` salvo que se indique otro estado. "Los cuatro `403`" son `ACCESS_DENIED`, `OWN_ACCOUNT_FORBIDDEN`, `ACTOR_EMPLOYEE_REQUIRED` y `ACCOUNT_MANAGED_BY_SUPERADMIN`.

| Operación | Códigos |
|---|---|
| `POST /authentication/sign-in` | `UNAUTHORIZED` `401` (también para `SYSTEM`), `ACCOUNT_SUSPENDED` `403`, `VALIDATION_ERROR` `400` |
| `POST /access-grants` | `EMPLOYEE_NOT_FOUND` `404`; los cuatro `403`; `EMPLOYEE_INACTIVE`; `EMPLOYEE_ACCOUNT_CONFLICT` `409` (el colaborador ya tiene cuenta); `EMPLOYEE_ACCOUNT_IN_COMPANY_CONFLICT` `409` (la cuenta ya está vinculada a otro colaborador de esa empresa); `EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH` (la cuenta existente no pertenece a ningún colaborador con la misma sigla y el mismo número de documento, o a ninguno: D-82, §14.15); `BUSINESS_RULE_VIOLATION` (la cuenta `SYSTEM`); `VALIDATION_ERROR` `400` |
| `POST /employees/{id}/account-unlinking` | `EMPLOYEE_NOT_FOUND` `404`; `ACCESS_DENIED` (no es superadministrador), `ACTOR_EMPLOYEE_REQUIRED` y `OWN_ACCOUNT_FORBIDDEN` (la cuenta es la propia) `403`; `EMPLOYEE_ACCOUNT_MISSING` (el colaborador no tiene cuenta que desvincular); `LAST_ACTIVE_SUPERADMIN`; `ACCESS_SUSPENSION_CONFLICT` `409` (un bloqueo simultáneo tomó el lugar de la `baja`) |
| `POST /users/{id}/role-assignments` | `USER_NOT_FOUND`, `ROLE_NOT_FOUND`, `COMPANY_NOT_FOUND`, `BRANCH_NOT_FOUND`, `COMMERCIALBRAND_NOT_FOUND` `404`; los cuatro `403`; `ROLE_ASSIGNMENT_CONFLICT` `409`; `ROLE_INACTIVE` (D-97); `ROLE_ASSIGNMENT_SCOPE_MISMATCH`; `ROLE_ASSIGNMENT_COMPANY_MISMATCH` (un rol de una empresa en otra, D-98); `ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`; `ROLE_ASSIGNMENT_EMPLOYEE_REQUIRED`; `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`; `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD`; `COMPANY_INACTIVE`, `BRANCH_INACTIVE` y `COMMERCIALBRAND_INACTIVE` (el alcance está inactivo, D-84); `BUSINESS_RULE_VIOLATION` (la cuenta `SYSTEM`, una caducidad que no es posterior); `VALIDATION_ERROR` `400` |
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

## 8. Archivos

| Caso | Petición | Respuesta |
|---|---|---|
| Subir un documento o una carga | `POST` en `multipart/form-data`: el archivo en `file` y los datos en `data`, como JSON. La ruta de las cargas de flota depende de §14.18 | `201` con el recurso |
| Descargar | `GET .../file`, `.../pdf` o `.../error-report` | El archivo con su `Content-Type` y `Content-Disposition: attachment; filename="…"` |
| Exportar una lista | El mismo `GET` de la lista con `Accept: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (§8.1) | El `.xlsx` con `Content-Disposition: attachment; filename="monedas-2026-10-06.xlsx"` |
| Exportar un reporte | `GET /reports/...` con el mismo `Accept` | El archivo |

Los archivos se guardan fuera de la base; la API nunca devuelve `file_path`.

### 8.1 Exportar a Excel

El archivo lo arma siempre el backend, con fastexcel (`org.dhatim:fastexcel`: escribe en streaming, pesa poco y no arrastra Apache POI). El frontend nunca arma un archivo: lo pide y el navegador lo guarda con el nombre que manda el backend.

**Cómo funciona**

- Una lista se exporta desde **su propio endpoint**. El `GET` que devuelve el JSON, pedido con el `Accept` de Excel, devuelve el archivo con la misma búsqueda (`q`), los mismos filtros y el mismo orden (`sort`), y **todas** las filas que coinciden: ignora `page` y `size` (§6.2). No hay rutas `/export` aparte.
- El controlador declara un segundo método en la misma ruta con `produces = SpreadsheetExporter.MEDIA_TYPE`, que llama a la misma consulta y pasa las filas a `SpreadsheetExporter` (`shared.interfaces.rest.export`) con sus columnas (`SpreadsheetColumn`).
- Cada columna es una clave de `messages.properties` para el encabezado y una función que lee la fila. Encabezados y textos salen en el idioma de la petición (`Accept-Language`), igual que los mensajes de error, y en español si la petición no nombra un idioma que la API hable (§7.1).
- Los valores conservan su tipo, para que la hoja sume, filtre y ordene: un número es número; una fecha, fecha; un momento, fecha y hora; un booleano, "Sí" o "No". Un texto que se traduce, como un estado o el tipo de una ubicación («Sede propia», «Own site»), se devuelve como `SpreadsheetColumn.Message`.
- Las fechas y los momentos se escriben en ISO, nunca en el formato de pantalla (diccionario §2.6, D-40, D-49). Una fecha es una celda de fecha con la máscara `AAAA-MM-DD`; un momento, una celda de fecha y hora con la máscara `AAAA-MM-DD hh:mm:ss`. Una celda de Excel no guarda la zona, así que todos los momentos de un archivo van en **una sola zona, nombrada en el encabezado** de cada columna de momentos: «Último acceso (America/Lima)». Nunca se mezclan zonas en una columna.
- Esa zona sale de la regla de "hoy" (diccionario §2.6) aplicada al alcance de la exportación: la sede si la lista es de una sede; si no, el país de la empresa si es de una empresa; si no, `America/Lima`. `SpreadsheetExporter` recibe la zona como parámetro y usa `America/Lima` si no se la pasan, que es lo que corresponde hoy a todas las listas. Desde el paso 3, cada controlador que exporte momentos de una empresa o una sede le pasa la zona de ese alcance.
- Una hoja con el nombre de la lista y la fila de encabezados fija y en negrita. El archivo se llama `<lista>-<fecha>.xlsx`, con la fecha de hoy en Lima y un nombre sin tildes, para que todos los navegadores lo guarden igual. La zona del nombre del archivo es independiente de la zona de los momentos.
- El `id` interno y el `public_id` nunca se exportan (§4.2; diccionario §2.4, D-51). Las columnas muestran lo que lee una persona: nombres y códigos de negocio, no identificadores. La única excepción del diccionario es un archivo pensado para editarse y volver a subirse a actualizar registros, que lleva además el `public_id` como llave estable; hoy no hay ninguno.
- Una referencia cuyo nombre puede repetirse se exporta también por su código. Dos sedes de una empresa pueden llamarse igual (diccionario §6.4), así que desde el 18/10/2026 la exportación de colaboradores lleva Nombre completo, Documento, Puesto, Código de sede, Sede, Fecha de ingreso y Estado, y la de ubicaciones, Código, Nombre de ubicación, Tipo de ubicación, Código de sede (de una ubicación propia), Instalación / Dirección y Estado.
- Un error responde JSON aunque se haya pedido Excel (§7): el país que no existe es `404` y la falta de permiso, `403`.

**Casos**

| Caso | Ejemplo | Qué se exporta |
|---|---|---|
| Catálogo completo | `GET /currencies`, `/countries`, `/users` | Lo que coincide con la búsqueda y los filtros de la pantalla (`isActive` en monedas), en su orden |
| Lista de un registro padre | `GET /countries/{countryId}/tax-id-types` | Los del padre; si el padre no existe, `404` |
| Lista filtrada en el servidor | `GET /countries/{countryId}/geo-divisions?q=lima&level=3&isActive=true&sort=name,asc`, `/companies?countryId=…` | Lo mismo que la lista con esos parámetros, sin página: el frontend manda los de la pantalla |
| Columnas de otro contexto | Empresas: país, documento tributario y moneda | Los nombres los resuelve la consulta por la fachada del dueño (`ExternalLocationService`), en lote: una llamada por tipo de referencia con todos los identificadores, nunca una por fila ni con un `join` |
| Reporte que cruza contextos | El exportable de stock de 52 columnas | Lo arma la capa de lectura en `/reports/...` (mapa de contextos §7), con el mismo `SpreadsheetExporter` |

**Lo que no hace**

- No exporta solo la página visible: siempre todo lo que coincide con la búsqueda y los filtros.
- No genera CSV ni otros formatos. Si hace falta uno, se agrega otro `produces` al mismo endpoint.

**Cómo se agrega a una lista**

1. Backend: en el controlador, el método `@GetMapping(produces = SpreadsheetExporter.MEDIA_TYPE)` con los mismos `@RequestParam` de la lista y `PageParams`, que lee todas las filas con `pageParams.toAllRowsQuery()`, y las claves `export.<lista>.*` en `messages.properties` y `messages_es.properties`. Una prueba lee el archivo de vuelta con `fastexcel-reader`.
2. Frontend: el endpoint llama a `exportSpreadsheet()` de `BaseApiEndpoint` con los filtros y `listQueryParams()` de la búsqueda y el orden; la API del módulo lo expone; el store tiene `exporting` y `export<Lista>()`, que lee la consulta de su `PagedList` y en un fallo deja `data-manager.export-error`. En la vista, `<mm-data-manager exportable [exporting]="store.exporting()" (exportRequested)="store.exportX()">`.

---

## 9. Seguridad

### 9.1 Sesión (diccionario §5.12, D-33)

- La sesión vive en el servidor. El navegador recibe la cookie `MM_SESSION` (`HttpOnly`, `SameSite=Strict`, `Secure` en producción) y la envía solo.
- Toda petición que cambia datos (`POST`, `PUT`, `PATCH`) lleva el token CSRF: llega en la cookie `XSRF-TOKEN` y vuelve en el encabezado `X-XSRF-TOKEN`. Vale también para iniciar sesión y registrarse.
- La API y el frontend se sirven desde el mismo origen. No hay CORS.
- `GET /authentication/me` y el inicio de sesión responden quién es la cuenta: `{id, email, mustChangePassword, roles, permissions, global, companies, employeeCompanyIds}`. `roles` son los códigos de sus roles vigentes en cualquier alcance; `permissions` (desde el 19/10/2026), los códigos de los permisos que esos roles incluyen, sumados, sin repetir y en orden, cualquiera sea la empresa en que actúa cada uno (diccionario §5.8 regla 1, D-99): la pantalla sabe por ellos qué ofrecer y qué puede marcar en un rol propio, y dónde actúa cada uno lo decide el backend; `global` dice si tiene un rol de alcance sistema, que ve todas las empresas. `companies` son las empresas que la cuenta alcanza —todas las activas si es global; si no, las de sus asignaciones vigentes—, cada una como `{id, name, timeZone, administered}`: `name` es el nombre comercial, `timeZone` la zona de su país (diccionario §2.6) y `administered` si la cuenta la administra (el superadministrador, todas; el administrador de empresa, la suya; el administrador de accesos, ninguna). No hay una empresa "actual": una cuenta puede trabajar para varias (diccionario §5.3). `employeeCompanyIds` (desde el 18/10/2026) son los `public_id` de las empresas donde la cuenta tiene un colaborador activo, tenga o no un rol ahí y aunque `companies` no las liste; un superadministrador que todavía no se vinculó lo recibe vacío (D-89), y la pantalla de colaboradores lo usa para deshabilitarle, diciendo por qué, la salida, el regreso y deshacer un vínculo.

### 9.2 Quién puede llamar a qué

| Ruta | Acceso |
|---|---|
| `sign-in`, `sign-up`, `sign-up/status`, `sign-up/companies`, documentación de la API, `actuator/health` | Público. `sign-up/companies` responde solo mientras el registro está encendido, y solo nombres (diccionario §5.2). Las listas de sedes y de tipos de documento del registro: pendiente (§14.16) |
| `me`, `change-password` | Con sesión, aunque la contraseña sea provisional |
| Lectura de catálogos generales (monedas, países, escalones, divisiones y tipos de documento tributario) y de catálogos de flota | `catalogos.ver`, que la semilla da a los siete roles del sistema (diccionario §5.6, D-43). Hasta el paso 3b, cualquier cuenta con un rol vigente |
| Escritura de catálogos generales | `catalogos.gestionar_generales`, que la semilla da solo a `SUPERADMIN` (D-43) |
| Escritura de catálogos de flota | `catalogos.gestionar_flota`, que la semilla da solo a `SUPERADMIN` (diccionario §7, D-15) |
| Lectura de recursos de empresa (empresas, sedes, marcas comerciales y ubicaciones, y todo lo que lleve `company_id` en los pasos siguientes) | El permiso de lectura del recurso —hoy `configuracion.ver_estructura`, que tienen los siete roles— **y** el alcance de la asignación que lo trae (diccionario §5.8 regla 6, D-99): una lista filtra por las empresas donde ese permiso actúa (`companiesActingOn`), y una empresa ajena, o alcanzada solo con un rol que no trae el permiso, o algo que le pertenece, responde `404`. Un rol de alcance sistema con el permiso ve todo; uno de sede lee toda su empresa (D-14). Alcanzar la empresa, y Row Level Security debajo, siguen como piso (D-66) |
| Escritura de recursos de empresa | El permiso de escritura del recurso —`configuracion.gestionar_empresas`, `configuracion.gestionar_sedes_y_marcas`, `configuracion.gestionar_ubicaciones`, que la semilla da solo a `SUPERADMIN`— **y** que su asignación actúe en esa empresa (`actsOn`): donde no actúa, `403 ACCESS_DENIED`; fuera del alcance, `404` (D-99). Las empresas, las sedes, las marcas y las ubicaciones son recursos de toda la empresa, así que una asignación de sede que trae el permiso no los escribe: la ronda de correcciones de la revisión adversarial, pendiente de llegar a la rama al 19/10/2026, lo decide con `actsOnWholeCompany(permiso, empresa)`; `actsOn(permiso, empresa, sede)` queda para lo que es de una sede en los pasos siguientes |
| Colaboradores | La fila mínima con `configuracion.ver_colaboradores`, que tienen los siete roles; `documentNumber`, `phone`, `photoPath` y la exportación con `configuracion.ver_datos_personales_colaboradores`, que tienen `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, cada uno donde actúa (D-105, tabla de abajo). Registrarlos, modificarlos y sus salidas y regresos siguen por rol |
| Cuentas, accesos, colaboradores y roles | Según la tabla de abajo: el rol con `@PreAuthorize` en la ruta, y el servicio decide además sobre qué cuenta o empresa (diccionario §5.8 regla 7) |
| Toda operación de negocio | Un permiso del catálogo, y para las cuentas, los accesos, los colaboradores y los roles, un rol (D-95). Tener sesión no alcanza (diccionario §5.2) |

**Cuentas, accesos y colaboradores** (decidido el 15/10/2026 y el 16/10/2026; diccionario v1.18, D-55 a D-72). Una cuenta **protegida** es la que tiene `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente, en cualquier empresa; la lista de cuentas lo dice en `managedBySuperadmin` (§6.2). Esos tres son los roles **reservados**, que solo otorga y retira un superadministrador. `ADMIN_EMPRESA` (C) es el administrador de la empresa C.

| Operación | `SUPERADMIN` | `ADMIN_ACCESOS` | `ADMIN_EMPRESA` (C) | Otro rol |
|---|---|---|---|---|
| `POST /access-grants` (crear o vincular la cuenta) | Sí, también una cuenta protegida. A sí mismo, solo mientras no tiene un colaborador activo. Una cuenta de otra persona, o sin colaborador, solo después de ver sus titulares y confirmar (D-82) | Sí, en cualquier empresa, salvo vincular una cuenta protegida (`ACCOUNT_MANAGED_BY_SUPERADMIN`) o una de otra persona (`EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH`, sin `context`) | `403` | `403` |
| `POST /employees/{id}/account-unlinking` | Sí, vinculado a un colaborador activo, salvo la propia cuenta y con la guardia del último superadministrador | `403` | `403` | `403` |
| Otorgar un rol | Cualquier rol, a cualquier cuenta salvo la propia y `SYSTEM` | Cualquiera salvo los reservados (`ACCESS_DENIED`), nunca a la propia ni a una protegida | `403` | `403` |
| Retirar una asignación | Cualquiera salvo las propias, con la guardia del último superadministrador | Las que puede otorgar, nunca de la propia ni de una protegida | `403` | `403` |
| Abrir o cerrar un bloqueo | Salvo en la propia y en `SYSTEM`, con la guardia | Salvo en la propia, en `SYSTEM` y en una protegida | `403` | `403` |
| Restablecer la contraseña | Salvo la propia y la de `SYSTEM` | Salvo la propia, la de `SYSTEM` y la de una protegida | `403` | `403` |
| `GET /users` sin `companyId` y su exportación; `GET /users/{id}/role-assignments`; `GET /users/{id}/suspensions` | Sí | Sí | `403` | `403` |
| `GET /users?companyId` y su exportación | Sí | Sí | Sí, de solo lectura, para C. Una empresa que ve sin administrarla, `403`; una fuera de su alcance, `404` | `403` |
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
- Una cuenta nunca tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` a la vez, y un rol reservado solo se otorga a una cuenta que ya cambió su contraseña provisional (§7.2).
- No hace falta proteger al último administrador de accesos: el superadministrador hace todo lo que él hace. Al último superadministrador sí: ninguna operación lo deja sin uno (§7.2, `LAST_ACTIVE_SUPERADMIN`).
- Un rol global lee lo que su matriz le da (D-68): el superadministrador, todo; el administrador de accesos, los catálogos, la estructura y los colaboradores con sus datos personales, nada del negocio (diccionario §5.6).

- Toda operación de negocio y toda lectura de datos exigen un permiso con `@PermissionRequired("catalogos.ver")`, de `shared`, en la clase o en el método del controlador: es `@PreAuthorize("hasAuthority('PERM_{value}')")`, y `iam` convierte en cada petición los permisos de los roles vigentes en autoridades `PERM_<código>` (construido el 19/10/2026, D-95). El servicio decide dónde actúa el permiso con `AccessScope.actsOn(permiso, empresa[, sede])` para una lectura o una escritura y con `companiesActingOn(permiso)` para una lista (D-99): el permiso dice qué y la asignación que lo trae dice dónde, también al leer. Las cuentas, los accesos, los colaboradores (sus escrituras, la salida y el regreso) y los roles siguen con `@PreAuthorize` y el rol (diccionario §5.8 regla 7). No se escriben listas de roles por módulo (`hasAnyRole('ADMIN_EMPRESA', 'OPERACIONES', …)`): chocarían con los roles propios de cada empresa (diccionario §5.5).
- Un recurso de empresa se filtra por las asignaciones de la cuenta que traen el permiso: el alcance baja y nunca sube (diccionario §5.8). El filtro se aplica en el listado, en la exportación y en la consulta por id, también en las rutas anidadas (`/companies/{id}/branches`). Alcanzar la empresa con cualquier rol queda como piso (D-66): no basta para leerla ni escribirla.
- Una empresa ajena responde `404`, no `403`, para no revelar que existe (decidido el 06/10/2026, §14.6).
- **Construido el 16/10/2026**, en el mismo cambio que habilitó los roles de empresa y de sede: el filtro de alcance en empresas, marcas, sedes, ubicaciones y colaboradores, también en la exportación, por identificador y en las rutas anidadas, más `/users?companyId` para el administrador de empresa. `CompanyIsolationIntegrationTests` prueba que el administrador de la empresa A recibe `404` al pedir la empresa B o algo suyo y no la ve en el listado. Debajo, la base aplica la misma frontera con Row Level Security (diccionario §2.5; mapa de contextos §9).
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
| 13 | Códigos `GEODIVISION_*`, `GEODIVISIONLEVEL_*` y `TAXIDTYPE_*`. El padre inexistente ya responde `GEODIVISION_NOT_FOUND` | Con guion bajo entre palabras si se valida §14.11 | §14.11 |
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
7. **Concurrencia**: si dos personas editan el mismo registro, hoy gana la última. Una opción es exigir `If-Match` con la versión del recurso; no hace falta mientras los catálogos los edite solo el superadministrador.
8. **Nombre del campo propio: `id` y no `publicId`** (§4.2). D-13 fija que viaja el `public_id`, no cómo se llama el campo. Desde el 05/10/2026 todos los recursos construidos usan `id`, también escalones, divisiones y tipos de documento. Las empresas todavía usan `publicId` (§13 fila 21). La dirección embebida no lleva identificador propio (§3.3).
9. **Códigos de éxito**: `201` con `Location` para un alta y para un hecho que se suma a una colección; `200` con el recurso afectado para un hecho en singular y para la mudanza de una dirección (§3, §3.3).
10. **Filtros**: nombres iguales al campo (`isActive`, `countryId`); `404` cuando el dueño de la colección no existe; `companyId` obligatorio cuando la cuenta tiene acceso a más de una empresa (§6.1).
11. **Códigos de error.**
    - `<RECURSO>` con guion bajo entre palabras (`GEO_DIVISION_*`), que exige cambiar las fábricas de `ApplicationError` o pasarles el nombre ya separado (§7.2).
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
    - El cierre manual de una orden de compra a `cumplida` cuando ninguna factura la referencia (D-21) se propone como `POST /purchase-orders/{id}/fulfillment`.
    - La sustitución de odómetro (diccionario §8.5) se propone como `POST /vehicles/{id}/odometer-replacements`. El diccionario no define qué datos guarda (lectura final del tablero anterior, inicial del nuevo, fecha, motivo) ni si tiene tabla propia, así que la forma del cuerpo también queda pendiente.
23. **Motivos de cambio de estado**: una colección filtrable `/status-change-reasons` (listar y crear), con `?fromStatusId={id}` para filtrar por origen y `?origin=registration` para los motivos de alta, que no tienen estado de origen (`from_status_id` vacío, diccionario §8.3). No se usa `fromStatusId` ausente para el alta, porque un filtro ausente no filtra (§6.1).
24. ~~**Subrecurso de dirección**~~. Adoptado el 06/10/2026 (§3.3): la dirección no tiene ruta propia; viaja anidada en el alta del dueño y embebida en su respuesta; `PUT` al subrecurso corrige la misma fila y `POST` crea una fila nueva a la que pasa a apuntar el dueño (D-8); el `PUT` del dueño no la lleva. Construido en empresas, sedes y ubicaciones de terceros (D-79); los proveedores y las plantillas nacen así.
25. ~~**Cómo se retira un escalón territorial.**~~ Resuelto por D-45: un escalón declarado no se retira; se renombra.
26. **Autor de los hechos**: que los campos que nombran a quien opera (`changed_by`, `granted_by`, `recorded_by`) los llene también el sistema con la cuenta de la sesión y no viajen desde el cliente (§3.2). D-42 cubre solo las columnas de registro. `iam` ya lo hace desde el 16/10/2026 con `granted_by` y con `closed_by` de los bloqueos; falta validarlo como regla para todos los contextos.
