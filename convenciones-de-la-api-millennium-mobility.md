# Convenciones de la API REST — ERP Millennium Mobility

**Actualizado:** 06/10/2026 (direcciones anidadas en su dueño, errores elegidos por `code`, fechas del Excel en ISO, rutas sin prefijo de contexto) · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.15 (§2.4 identificadores, §2.6 fechas, §4.6 direcciones, §5.12 sesiones, D-8, D-13, D-38, D-40, D-42, D-43, D-49, D-51), reglas de arquitectura de la skill `backend-module`, [mapa de contextos](mapa-de-contextos-millennium-mobility.md) y el backend construido. Si este documento y el diccionario no coinciden, manda el diccionario.

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
| `iam` | `/authentication/sign-in` · `sign-out` · `me` · `sign-up` · `sign-up/status` · `sign-up/companies` · `change-password` · `/users` · `/users/{id}` · `/access-grants` **(propuesta, §14.15)** · `/users/{id}/role-assignments` · `/role-assignments/{id}` · `/role-assignments/{id}/revocation` · `/users/{id}/suspensions` · `/suspensions/{id}` · `/suspensions/{id}/closure` · `/roles` · `/roles/{id}` · `/permissions` |
| `organization` | `/companies` · `/companies/{id}` · `/companies/{id}/status` · `/companies/{id}/fiscal-address` · `/companies/{id}/branches` · `/branches/{id}` · `/branches/{id}/status` · `/branches/{id}/address` · `/companies/{id}/commercial-brands` · `/commercial-brands/{id}` · `/commercial-brands/{id}/status` · `/companies/{id}/locations` · `/locations/{id}` · `/locations/{id}/status` · `/locations/{id}/address` · `/companies/{id}/employees` · `/employees/{id}` · `/employees/{id}/termination` **(provisional, §14.17)**. `GET /companies?countryId=…` responde si un país opera (D-3) |
| `vehiclecatalog` | `/vehicle-types` · `/vehicle-brands` · `/vehicle-brands/{id}/vehicle-models` · `/vehicle-models/{id}` · `/colors` · `/equipment-types`, cada uno con `/{id}` y `/{id}/status` |
| `fleet` | `/vehicles` · `/vehicles/{id}` · `/vehicles/{id}/plate` · `/vehicles/{id}/vin` · `/vehicles/{id}/annulment` **(propuesta, §14.20)** · `/vehicles/{id}/status-changes` · `/status-changes/{id}` · `/vehicles/{id}/mileage-readings` · `/mileage-readings/{id}` · `/vehicles/{id}/odometer-replacements` **(propuesta, §14.22)** · `/vehicles/{id}/documents` · `/vehicle-documents/{id}` · `/vehicle-documents/{id}/file` · `/vehicles/{id}/equipments` · `/vehicle-equipments/{id}` · `/vehicle-equipments/{id}/removal` · `/vehicles/{id}/keys` · `/vehicle-keys/{id}` · `/vehicles/{id}/fund-assignments` · `/vehicles/{id}/tax-declarations` · `/vehicle-statuses` · `/vehicle-statuses/{id}` · `/vehicle-statuses/{id}/status` · `/status-change-reasons` **(propuesta, §14.23)** · `/status-change-reasons/{id}` · `/status-change-reasons/{id}/status` · `/document-types` · `/document-types/{id}` · `/document-types/{id}/status` · `/funds` · `/funds/{id}` · `/funds/{id}/status` · `/tax-declaration-imports` y `/fund-assignment-imports` **(propuesta, §14.18)** |
| `procurement` | `/suppliers` · `/suppliers/{id}/status` · `/suppliers/{id}/address` · `/suppliers/{id}/balance` · `/purchase-order-types` · `/purchase-order-types/{id}/status` · `/purchase-order-templates` · `/purchase-order-templates/{id}/status` · `/purchase-order-templates/{id}/issuer-address` · `/purchase-orders` · `/purchase-orders/{id}/sending` · `/purchase-orders/{id}/annulment` · `/purchase-orders/{id}/fulfillment` **(propuesta, §14.22)** · `/purchase-orders/{id}/pdf` · `/purchase-invoices` · `/purchase-invoices/{id}/annulment` · `/credit-notes` · `/supplier-payments` · `/supplier-payments/{id}/annulment`, cada colección con `/{id}` |
| Plataforma (`shared`) | `/change-log` · `/imports` · `/imports/{id}` · `/imports/{id}/error-report` |
| Capa de lectura | `/reports/stock-export` · `/reports/day-gaps` · `/reports/down-payment-differences` · `/reports/vehicle-costs?vehicleId=…` **(propuesta, §14.19)** |

- Los recursos de empresa sin anidar (`/vehicles`, `/funds`, los de `procurement`) reciben `companyId` como filtro (§6.1).
- `location` no expone `/addresses`: una dirección se escribe y se lee solo a través de su dueño (§3.3).
- Toda tabla del diccionario con `is_active` expone `PATCH /{id}/status` sobre su ruta de elemento. Es la única forma de retirarla, porque no hay `DELETE` (§3.1). Los escalones territoriales no tienen `is_active` ni se retiran: se declaran en orden y se renombran (D-45).
- `/users` solo lista y lee: no hay alta suelta de cuentas. Las cuentas nacen al dar acceso a un colaborador o por el registro público (D-32, D-38). `SYSTEM` nace en la migración y el primer superadministrador al arrancar (D-34), ninguno por la API.
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
| Cerrar, anular, retirar o enviar | `POST` a un subrecurso con el sustantivo del hecho, en singular | `POST /purchase-invoices/{id}/annulment`, `POST /role-assignments/{id}/revocation`, `POST /purchase-orders/{id}/sending`, `POST /employees/{id}/termination` (provisional, §14.17), `POST /purchase-orders/{id}/fulfillment` (propuesta, §14.22) |
| Una corrección delicada que exige motivo (diccionario §13.1) | `PUT` a un subrecurso propio, con `reason` obligatorio y su propio permiso | `PUT /vehicles/{id}/plate` con `{ "plate": "ABC123", "reason": "Error de digitación" }` |
| Activar o desactivar | `PATCH /{id}/status` | `PATCH /currencies/{id}/status` |

**Correcciones delicadas.** El diccionario fija cuatro (§13.1): el VIN, la placa, el monto de una factura y una fecha ya reportada. El `PUT` del recurso no acepta esos campos ya registrados: la placa y el VIN se corrigen solo por `PUT /vehicles/{id}/plate` y `PUT /vehicles/{id}/vin`, con `reason` obligatorio. El monto de una factura y las fechas ya reportadas todavía no tienen ruta (§14.21).

**Quién y cuándo.** Las columnas de registro (`created_at`, `updated_at`, `created_by` y `updated_by`) nunca viajan desde el cliente. Las llena el sistema con la cuenta de la sesión, o con `SYSTEM` de forma explícita cuando no hay sesión (D-42). Los datos del hecho que el diccionario pide a quien registra sí viajan en el cuerpo:
- el momento de una lectura de kilometraje (`readAt`, que no es la fecha del registro, diccionario §8.5);
- el inicio de un bloqueo programado por adelantado (diccionario §5.4);
- quién instaló un equipo (`installedByEmployeeId`, diccionario §11.4).

Que el sistema llene también los campos de autor de un hecho que nombran a quien opera (`changed_by`, `granted_by`, `recorded_by`) es una propuesta (§14.26): D-42 no los cubre.

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

- `streetName` es obligatorio, hasta 180 caracteres; `streetNumber` llega hasta 20, `unit` hasta 50 y `postalCode` hasta 12. Los demás son opcionales.
- El país no viaja: lo fija el dueño. La dirección de una empresa, una sede o una ubicación está en el país de la empresa, y la base lo exige con una llave foránea compuesta. Es una regla de este documento que el diccionario todavía no registra (§4.6 solo exige que la división sea del país de la dirección). El proveedor, cuando exista, mandará su propio país.
- En el alta del dueño, `fiscalAddress: null` registra al dueño sin dirección.

**Lo que devuelve el servidor.** La dirección embebida en el dueño, sin `id` ni fechas, porque no es un recurso:

```json
"fiscalAddress": {
  "countryId": "019352e1-5b1c-7a3e-9d4f-0a1b2c3d7f3a",
  "countryIsoAlpha2": "PE",
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

Un dueño sin dirección responde `"fiscalAddress": null`. Que el país viaje como `countryId` y no por su código es una deuda registrada (§13 fila 22). La lista de empresas también trae la dirección, y la resuelve en lote: una lectura a `location` por página, nunca una por fila (§8.1).

**Cómo cambia.** El `PUT` del dueño (`PUT /companies/{id}`) no lleva la dirección y no la modifica. La dirección se crea con su dueño o se cambia por su subrecurso:

| Qué pasó | Petición | Efecto | Respuesta |
|---|---|---|---|
| Se registra el dueño con su dirección | `POST /companies` con `fiscalAddress` | La dirección se crea en la misma transacción que la empresa, después de comprobar los conflictos de la empresa. Si algo falla, no queda ninguna de las dos | `201` con la empresa |
| Se escribió mal la dirección | `PUT /companies/{id}/fiscal-address` | Se corrige la misma fila. Si la empresa no tiene dirección, `422` con el código `COMPANY_FISCAL_ADDRESS_MISSING` | `200` con la empresa |
| El dueño se mudó, o registra su primera dirección | `POST /companies/{id}/fiscal-address` | Se crea una fila nueva y el dueño pasa a apuntar a ella; la anterior se conserva para los documentos ya emitidos | `200` con la empresa y su nueva dirección, sin `Location` |

La dirección no tiene ruta propia, así que la mudanza no responde `201`: lo creado no es direccionable y la empresa no es nueva.

Un fallo de la dirección responde por el endpoint del dueño con el código de `location`: `GEODIVISION_NOT_FOUND` si la división no existe, `VALIDATION_ERROR` si el país exige código postal y no llegó, `BUSINESS_RULE_VIOLATION` si la división no sirve para esa dirección.

Las sedes siguen el mismo contrato cuando se construyan: `POST /companies/{companyId}/branches` con `address` anidada, `PUT /branches/{id}` sin dirección, y `PUT` o `POST /branches/{id}/address` para corregir o mudar.

---

## 4. El cuerpo JSON

### 4.1 Nombres

| Regla | Ejemplo |
|---|---|
| Los campos van en camelCase y repiten el nombre de la columna del diccionario | `iso_alpha_2` → `isoAlpha2`, `requires_postal_code` → `requiresPostalCode` |
| Los booleanos conservan su prefijo | `is_system` → `isSystem`, `has_factory_screen` → `hasFactoryScreen` |
| Los sufijos de fecha se conservan | `created_at` → `createdAt`, `invoice_date` → `invoiceDate`, `campaign_month` → `campaignMonth` |
| Los valores de una lista cerrada viajan tal como los fija el diccionario | `"scopeLevel": "COMPANY"`, `"reason": "vacaciones"`, `"revocationReason": "baja_colaborador"` |
| **(propuesta, §14.13)** Los campos opcionales sin valor viajan como `null`, no se omiten | `"taxName": null` |

### 4.2 Identificadores (diccionario §2.4, D-13)

| Qué se referencia | Cómo viaja | Ejemplo |
|---|---|---|
| El propio recurso | `id`, con el `public_id` **(el nombre `id`: propuesta, §14.8)** | `"id": "019352e1-…"` |
| Una entidad del negocio (empresa, sede, proveedor, vehículo, división) | `<entidad>Id` con su `public_id` | `"companyId": "…"`, `"parentId": "…"` |
| Un catálogo de estándar externo | Su código | `"currency": "PEN"`, `"country": "PE"` |

- El `id` interno no aparece nunca en una respuesta, una ruta ni un archivo exportado, tampoco en las referencias (diccionario §2.4).
- Una respuesta puede acompañar la referencia con lo que la pantalla necesita mostrar, para no obligar a otra petición: `"companyId"` con `"companyName"`, `"parentId"` con `"parentName"`.

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

El servidor normaliza lo que recibe, así que el cliente puede enviar cualquier forma: códigos en mayúsculas y sin espacios (`pen` → `PEN`), correos en minúsculas, placas y documentos tributarios sin guiones ni espacios (diccionario §2.2). La respuesta devuelve el valor ya normalizado.

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
| `q` | Texto que contiene alguno de los campos de búsqueda de la lista, sin distinguir mayúsculas. `%` y `_` se buscan tal cual |
| `page` | Índice de la página, desde 0. **Con `page`, la respuesta es el sobre de arriba; sin `page`, es el arreglo de todo lo que coincide**, en el mismo orden y con la misma búsqueda y filtros, como lo pide un select |
| `size` | 50 por omisión y 200 como máximo |
| `sort` | `campo` o `campo,desc`, con el nombre del campo JSON. Cada lista declara por qué campos ordena; con otro campo usa su orden propio. Los empates se ordenan por el `id` interno, para que una fila no salte de página |

Lo que busca y ordena cada lista construida:

| Lista | `q` busca en | Filtros | `sort` acepta | Orden propio |
|---|---|---|---|---|
| `GET /currencies` | `isoCode`, `name`, `symbol` | `isActive` | `isoCode`, `name`, `symbol`, `decimalPlaces`, `isActive` | `isoCode` |
| `GET /countries` | `name`, `isoAlpha2`, `isoAlpha3`, `callingCode` | — | `name`, `isoAlpha2`, `isoAlpha3`, `callingCode`, `defaultTimezone`, `taxRate`, `defaultCurrency` | `name` |
| `GET /countries/{id}/geo-divisions` | `code`, `name` | `level`, `isActive` | `code`, `name`, `level`, `isActive` | `level` y `code` |
| `GET /countries/{id}/tax-id-types` | `code`, `name` | `isActive`, `appliesTo` | `code`, `name`, `appliesTo`, `length`, `hasCheckDigit`, `isActive` | `code` |
| `GET /companies` | `legalName`, `tradeName`, `taxId` | `active`, `countryId` | `legalName`, `tradeName`, `taxId`, `phone`, `isActive` | `tradeName` |
| `GET /users` | `email` | — | `email`, `lastLoginAt`, `mustChangePassword` | `email` |

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
- `message` sale traducido según `Accept-Language` (`es` o `en`), para quien llame a la API sin el frontend.
- El frontend elige su texto de `es.json` por `code` (decidido el 06/10/2026, §14.12), con un `switch` por los códigos que conoce. Un código desconocido cae en el mensaje genérico de la operación, nunca en uno específico: así una causa nueva muestra un texto incompleto, pero nunca uno falso. El estado HTTP solo decide en dos casos: `0`, sin red, y los errores del framework que todavía no tienen código del catálogo (§13 fila 18). El frontend nunca muestra `message` ni lee el cuerpo por su cuenta.
- `details` explica el caso para quien depura. Nunca lleva SQL, nombres de restricciones ni trazas.

### 7.2 Códigos

| `code` | Estado | Cuándo |
|---|---|---|
| `VALIDATION_ERROR` | `400` | El cuerpo, la ruta o un parámetro no cumplen el formato |
| `UNAUTHORIZED` | `401` | No hay sesión, venció, o las credenciales no son válidas |
| `ACCESS_DENIED` | `403` | Hay sesión pero falta el rol o el permiso; también con contraseña provisional y con el registro público apagado |
| `<RECURSO>_NOT_FOUND` | `404` | El recurso, o uno que la petición referencia, no existe |
| `<RECURSO>_CONFLICT` | `409` | Ya existe un valor único: un código ISO, un número de comprobante |
| `<RECURSO>_<CAMPO>_CONFLICT` | `409` | Lo mismo, en un agregado con dos o más reglas de unicidad: cada una tiene su código (`VEHICLE_VIN_CONFLICT`, `VEHICLE_PLATE_CONFLICT`) |
| `METHOD_NOT_ALLOWED` **(propuesta, §14.11)** | `405` | El método no existe para esa ruta; en particular, todo `DELETE` (§3.1) |
| `BUSINESS_RULE_VIOLATION` | `422` | La petición es válida pero rompe una regla del negocio: una lectura de kilometraje menor, un camino de estado no permitido, una nota de crédito en otra moneda |
| `UNEXPECTED_ERROR` | `500` | Un fallo que no debería pasar |

- Un fallo esperado del negocio se devuelve como `Result`, nunca como excepción (regla 3.1), y se traduce aquí.
- Las fábricas de `ApplicationError` (`notFound`, `conflict`, `businessRuleViolation`, `validationError`…) producen los códigos genéricos de la tabla. Por omisión, un duplicado es `ApplicationError.conflict(...)`, que responde `<RECURSO>_CONFLICT`.
- Cuando un agregado tiene dos o más reglas de unicidad, cada una lleva su código propio `<RECURSO>_<CAMPO>_CONFLICT`, que produce `ApplicationError.conflict(recurso, campo, …)`, para que el cliente sepa cuál falló: `COMPANY_COUNTRY_CONFLICT` y `COMPANY_TAX_ID_CONFLICT` en la empresa, que ya los emite; `VEHICLE_VIN_CONFLICT` y `VEHICLE_PLATE_CONFLICT` en la unidad; `BRANCH_CODE_CONFLICT` y `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT` en la sede (diccionario §6.4). Un agregado con una sola regla se queda con el genérico.
- El estado de un código propio sale de su sufijo: `_NOT_FOUND` responde `404`, `_CONFLICT` responde `409` y cualquier otro, `422`. Por eso un código propio de duplicado termina siempre en `_CONFLICT`: con otro sufijo saldría como regla rota. Un código propio que no es un duplicado, como `COMPANY_FISCAL_ADDRESS_MISSING` (§3.3), responde `422`.
- Como todo `Result.failure` es un fallo esperado, un código propio nunca responde `500`. Su texto en el servidor se busca en `error.<código-en-kebab>.message`, por ejemplo `error.vehicle-vin-conflict.message`, y el frontend tiene su propia clave en `es.json` (§7.1).
- `<RECURSO>` es el nombre del agregado en mayúsculas. Hoy las fábricas lo pasan a mayúsculas sin separar palabras: `notFound("GeoDivision", …)` produce `GEODIVISION_NOT_FOUND`. **(propuesta, §14.11)** Separar las palabras con guion bajo (`GEO_DIVISION_NOT_FOUND`, `PURCHASE_INVOICE_CONFLICT`), lo que obliga a cambiar las fábricas.
- **(propuesta, §14.5)** Un `422` dice qué regla falló en `rule`, un identificador estable: `"rule": "mileage-reading.not-lower"`. Así el cliente distingue reglas sin leer el texto.

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
- Cada columna es una clave de `messages.properties` para el encabezado y una función que lee la fila. Encabezados y textos salen en el idioma de la petición (`Accept-Language`), igual que los mensajes de error.
- Los valores conservan su tipo, para que la hoja sume, filtre y ordene: un número es número; una fecha, fecha; un momento, fecha y hora; un booleano, "Sí" o "No". Un texto que se traduce, como un estado, se devuelve como `SpreadsheetColumn.Message`.
- Las fechas y los momentos se escriben en ISO, nunca en el formato de pantalla (diccionario §2.6, D-40, D-49). Una fecha es una celda de fecha con la máscara `AAAA-MM-DD`; un momento, una celda de fecha y hora con la máscara `AAAA-MM-DD hh:mm:ss`. Una celda de Excel no guarda la zona, así que todos los momentos de un archivo van en **una sola zona, nombrada en el encabezado** de cada columna de momentos: «Último acceso (America/Lima)». Nunca se mezclan zonas en una columna.
- Esa zona sale de la regla de "hoy" (diccionario §2.6) aplicada al alcance de la exportación: la sede si la lista es de una sede; si no, el país de la empresa si es de una empresa; si no, `America/Lima`. `SpreadsheetExporter` recibe la zona como parámetro y usa `America/Lima` si no se la pasan, que es lo que corresponde hoy a todas las listas. Desde el paso 3, cada controlador que exporte momentos de una empresa o una sede le pasa la zona de ese alcance.
- Una hoja con el nombre de la lista y la fila de encabezados fija y en negrita. El archivo se llama `<lista>-<fecha>.xlsx`, con la fecha de hoy en Lima y un nombre sin tildes, para que todos los navegadores lo guarden igual. La zona del nombre del archivo es independiente de la zona de los momentos.
- El `id` interno y el `public_id` nunca se exportan (§4.2; diccionario §2.4, D-51). Las columnas muestran lo que lee una persona: nombres y códigos de negocio, no identificadores. La única excepción del diccionario es un archivo pensado para editarse y volver a subirse a actualizar registros, que lleva además el `public_id` como llave estable; hoy no hay ninguno.
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

### 9.2 Quién puede llamar a qué

| Ruta | Acceso |
|---|---|
| `sign-in`, `sign-up`, `sign-up/status`, `sign-up/companies`, documentación de la API, `actuator/health` | Público. `sign-up/companies` responde solo mientras el registro está encendido, y solo nombres (diccionario §5.2). Las listas de sedes y de tipos de documento del registro: pendiente (§14.16) |
| `me`, `change-password` | Con sesión, aunque la contraseña sea provisional |
| Lectura de catálogos generales (monedas, países, divisiones) | Cualquier cuenta con al menos un rol vigente. Cuando existan los permisos, la lectura pasa a un permiso propio (D-43) |
| Escritura de catálogos generales (monedas, países, divisiones) | Solo `SUPERADMIN` (D-43) |
| Escritura de catálogos de flota | `SUPERADMIN` (diccionario §7, D-15). Qué permiso exigirá se define con la matriz de permisos por rol (diccionario §14) |
| Lectura de recursos de empresa (empresas, sedes, marcas comerciales, ubicaciones y todo lo que lleve `company_id`) | Cualquier cuenta con al menos un rol vigente, como piso, **más** el filtro obligatorio por las asignaciones de la cuenta dentro de la consulta (diccionario §5.8). Un rol de alcance sistema ve todo; una cuenta sin asignación en la empresa recibe una lista vacía, y una empresa ajena responde `404`. Cuando existan los permisos, cada lectura pasa a su permiso propio |
| Administración de cuentas y accesos (`iam`) | El rol, con `@PreAuthorize` |
| Toda operación de negocio | Un rol o un permiso. Tener sesión no alcanza (diccionario §5.2) |

- Lecturas de catálogos generales con `@AnyRoleRequired` (D-43). Escrituras de catálogos generales con `@PreAuthorize` y el rol `SUPERADMIN` (D-43). El resto de las escrituras con `@PreAuthorize` y el rol, más el permiso cuando exista.
- Lecturas de recursos de empresa con `@AnyRoleRequired` y el filtro de alcance. No se escriben listas de roles por módulo (`hasAnyRole('ADMIN_EMPRESA', 'OPERACIONES', …)`): chocarían con los roles propios de cada empresa (diccionario §5.5).
- Un recurso de empresa se filtra por las asignaciones de la cuenta: el alcance baja y nunca sube (diccionario §5.8). El filtro se aplica en el listado, en la exportación y en la consulta por id, también en las rutas anidadas (`/companies/{id}/branches`).
- Una empresa ajena responde `404`, no `403`, para no revelar que existe (decidido el 06/10/2026, §14.6).
- **Cuándo.** Hoy solo se otorgan roles de alcance sistema, así que el filtro todavía no recorta nada. El mismo cambio que habilite los roles de empresa y de sede trae el filtro de alcance en `/companies` y sus rutas anidadas, la apertura de `/users` al administrador de empresa y una prueba de integración: el administrador de la empresa A recibe `404` al pedir la empresa B y no la ve en el listado (orden de implementación, pendientes).
- Un campo que la cuenta no puede ver **no viaja**: se omite de la respuesta, no se envía vacío (diccionario §2.5). Ejemplo: sin `stock.ver_costos`, la fila del exportable de stock (`/reports/stock-export`) llega sin `invoicedAmount`. El vehículo de `/vehicles` nunca trae costos: el valor de factura es de `procurement` (diccionario §3; mapa §6.1). Es la única excepción a la regla de enviar `null` (§4.1).

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
| 4 | Las respuestas de escalones, divisiones y tipos de documento ya usan `id`, `countryId` y `parentId`, pero nombran el país con `countryId` (UUID) más `countryIsoAlpha2` | `country: "PE"` (§4.2, D-13) | — |
| 5 | El país referencia su moneda con `defaultCurrencyId` (UUID) y la repite en `defaultCurrencyIsoCode` | `defaultCurrency: "PEN"` (D-13) | — |
| 6 | `RoleResource.system` | `isSystem` (§4.1) | — |
| 7 | `taxRate` viaja como número | Texto decimal (§5.2) | §14.2 |
| 8 | Las altas de monedas, países y divisiones responden `201` sin `Location` | `201` con `Location` (§3) | §14.9 |
| 9 | ~~Filtro `activeOnly` (monedas)~~ | Resuelto el 06/10/2026: las monedas se filtran con `isActive`; `activeOnly` e `isoCode` ya no existen (la búsqueda `q` cubre el código). El filtro `operationalOnly` de países tampoco existe: el país no guarda si opera (D-3). Las empresas siguen filtrando con `active` | §14.10 |
| 10 | ~~`GET /countries/XX/divisions` con un país inexistente responde `200 []`~~ | Resuelto el 05/10/2026: escalones, divisiones y tipos de documento responden `404` si el país no existe | — |
| 11 | ~~Las divisiones territoriales se listan completas, sin página~~ | Resuelto el 06/10/2026: con `page` responden una página (§6.2) | — |
| 12 | ~~`GET /users` devuelve la lista completa, sin página~~ | Resuelto el 06/10/2026: con `page` responde una página (§6.2) | — |
| 13 | Códigos `GEODIVISION_*`, `GEODIVISIONLEVEL_*` y `TAXIDTYPE_*`. El padre inexistente ya responde `GEODIVISION_NOT_FOUND` | Con guion bajo entre palabras si se valida §14.11 | §14.11 |
| 14 | Un error inesperado devuelve en `details` el mensaje de la excepción | `details` nunca lleva el texto interno (§7.1) | — |
| 15 | El `422` no dice qué regla falló | Campo `rule` (§7.2) | §14.5 |
| 16 | `POST /api/v1/users` crea una cuenta sin colaborador (`email`, `temporaryPassword`, `roleIds`) y otorga roles en la misma petición; es la pantalla "Nuevo usuario" | Toda cuenta pertenece a un colaborador (D-38). Desde el paso 3, el acceso se da con `POST /access-grants` y `/users` queda solo de lectura | Ruta: §14.15 |
| 17 | ~~`BaseApiEndpoint.delete` en el frontend envía `DELETE`~~ | Resuelto el 06/10/2026: el método ya no existe; un registro se retira con `PATCH …/status`, una revocación o un cierre (regla 4.2) | — |
| 18 | Los errores del framework (ruta inexistente, método no permitido, tipo de contenido no soportado, parámetro faltante) responden con el nombre del estado como `code` (`NOT_FOUND`, `METHOD_NOT_ALLOWED`, `UNSUPPORTED_MEDIA_TYPE`, `BAD_REQUEST`), y `message` y `details` llegan en inglés, sin traducir (`GlobalExceptionHandler.handleErrorResponse`) | Un código del catálogo de §7.2 y `message` traducido (§7.1) | §14.11 |
| 19 | ~~El frontend elige el texto de error por el estado HTTP (`409` = duplicado, `422` = regla rota), y el store de sedes lee `message`~~ | Resuelto el 06/10/2026: `ApiError` conserva `code` y cada store elige la clave de `es.json` por `code`, con el mensaje genérico de la operación para un código desconocido; el estado solo decide para `0` y para la fila 18 (§7.1) | — |
| 20 | ~~`/api/v1/addresses` con `GET`, `POST`, `PUT` y `GET /{id}`; la empresa recibía `fiscalAddressPublicId` y el frontend creaba la dirección en una petición aparte, antes que la empresa~~ | Resuelto el 06/10/2026: la ruta ya no existe; la dirección viaja anidada en el alta de la empresa, embebida en su respuesta, y se corrige o se muda por `/companies/{id}/fiscal-address` (§3.3) | — |
| 21 | La empresa nombra su propio identificador `publicId` y referencia su país y su moneda funcional por UUID (`countryPublicId`, `functionalCurrencyPublicId`); el tipo de documento llega como `taxIdTypePublicId` | `id` (§14.8); `country: "PE"` y `functionalCurrency: "PEN"` (§4.2, D-13); `taxIdTypeId` (§4.2) | El nombre `id`: §14.8 |
| 22 | La dirección embebida en la empresa nombra el país con `countryId` (UUID) más `countryIsoAlpha2` y `countryName` | `country: "PE"` con `countryName` (§4.2, D-13) | — |

---

## 14. Decisiones pendientes

Todo lo que este documento fija y no remite al diccionario ni a una regla de arquitectura es una propuesta hasta que se valide. Se aplica a lo nuevo, y a lo construido (§13), solo cuando se valide; mientras tanto, cada una aparece marcada en el cuerpo. Las más importantes:

**Forma general**

1. ~~**Prefijo de contexto en las rutas**~~. Rechazado el 06/10/2026: las rutas no llevan el contexto (§2.1). Venía de *Contextos delimitados* v1.8 y de la plantilla de la skill `backend-module`, que ya no lo enseña. Hay un solo despliegue y un solo cliente, ningún recurso repite nombre entre contextos y el cambio se encarecía con cada contexto construido. La autenticación sigue en `/api/v1/authentication/...` (diccionario §5.12).
2. **Formato de los montos**: objeto `{ amount, currency }` con `amount` como texto decimal; los tipos de cambio y las tasas, también como texto decimal (§5.2).
3. **Nombres de los campos de un período**: `periodStart` y `periodEnd` (§5.1). El diccionario fija que viajan como dos campos, pero no cómo se llaman.
4. ~~**Página**~~. Resuelto el 06/10/2026, a pedido: toda lista, también los catálogos chicos, se busca, filtra, ordena y pagina en su propio endpoint, con el sobre `{ items, page, size, totalItems, totalPages }`; sin `page` responde el arreglo completo (§6.2).
5. **Campo `rule`** en los `422` (§7.2), que obliga a ampliar `ErrorResource` y `ApplicationError`.
6. ~~**`404` para una empresa ajena**~~. Adoptado el 06/10/2026 con la lectura de recursos de empresa (§9.2): una empresa ajena responde `404`, no `403`, y una prueba de integración lo verifica en el cambio que habilite los roles de empresa.
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

15. **Dar acceso a un colaborador**: `POST /access-grants` con `employeeId`, `email` y `temporaryPassword` (§2.2). Crea la cuenta, o vincula el colaborador a la que ya tiene ese correo, en una transacción (D-38; mapa §6.2). Los roles no viajan aquí: se otorgan después en `/users/{id}/role-assignments`. D-38 fija la operación, no la ruta. Falta decidir:
    - el nombre (la alternativa es `POST /users` con `employeeId`);
    - la respuesta: `201` si crea la cuenta y `200` si vincula una existente;
    - qué pasa con `temporaryPassword` cuando la cuenta ya existe.
16. **Listas públicas del registro (D-32).** El diccionario (§5.2) exige que el formulario liste sin sesión la empresa y la sede, pero no fija la ruta de las sedes. Tampoco decide cómo obtiene sin sesión los tipos de documento de las personas, aunque `employees.document_type_id` es obligatorio y `/countries/{id}/tax-id-types` exige un rol (D-43).
    - Propuesta: `GET /authentication/sign-up/companies/{companyId}/branches` y `GET /authentication/sign-up/document-types`, públicas, solo con el registro encendido (con el registro apagado, `403` como `sign-up`) y solo con `id` y `name` de los registros activos.
    - Alternativa: que `sign-up/companies` devuelva cada empresa con sus sedes y los tipos de documento de personas de su país.
17. **Ruta de la salida de un colaborador**: `POST /employees/{id}/termination` es provisional. Depende de la decisión pendiente 2 del [mapa de contextos](mapa-de-contextos-millennium-mobility.md): con la opción A el caso de uso vive en `iam` y la ruta cambia de contexto; con la opción B se queda en `organization`.
18. **Cargas que actualizan la flota.** El diccionario pone `import_batches` en `shared` (§3, §11.8). Pero las cargas del impuesto vehicular y de la garantía mobiliaria cruzan por placa y escriben `vehicle_tax_declarations` y `vehicle_fund_assignments`, que son de `fleet`.
    - Opción A (propuesta): la carga entra por el contexto dueño de los datos (`POST /tax-declaration-imports`, `POST /fund-assignment-imports`). `fleet` hace el cruce y usa `shared` solo para registrar la carga y guardar el archivo y el archivo de errores.
    - Opción B: `POST /imports` sigue en `shared`, que define un puerto de procesamiento por `import_type` que `fleet` implementa.

    Las dos evitan el ciclo (mapa §6.5 y decisión 8).
19. **Costos de una unidad para su ficha**: `GET /reports/vehicle-costs?vehicleId=…` en la capa de lectura. Entrega el valor de factura, las notas de crédito y el valor del vehículo calculado, y omite los montos sin `stock.ver_costos` (§9.2; mapa, decisión 7).
20. **Anulación de una unidad.** El diccionario prevé el permiso `stock.anular_vehiculo` (§5.6) y el candado de §9.6 (valor cero y ningún contrato). No fija si la anulación es automática al llegar el valor a cero o una acción explícita con ese permiso, ni su ruta. Propuesta: `POST /vehicles/{id}/annulment`, solo con `stock.anular_vehiculo` y respetando el candado; `procurement` la pide a través de la fachada de `fleet` (mapa §6.1).
21. **Correcciones delicadas** (§3.2). Cada una va por su propio subrecurso con `reason` obligatorio. Falta decidir:
    - qué monto se corrige: el diccionario dice que `invoiced_amount` de una línea no se modifica nunca, y que el total debe coincidir con la suma de las líneas;
    - qué fechas cuentan como "ya reportadas" y por qué ruta se corrigen (por ejemplo, el período de una cesión a fondo);
    - qué permiso exige cada una: el diccionario solo define `stock.corregir_placa`.
22. **Dos hechos que el diccionario exige sin nombrar su ruta.**
    - El cierre manual de una orden de compra a `cumplida` cuando ninguna factura la referencia (D-21) se propone como `POST /purchase-orders/{id}/fulfillment`.
    - La sustitución de odómetro (diccionario §8.5) se propone como `POST /vehicles/{id}/odometer-replacements`. El diccionario no define qué datos guarda (lectura final del tablero anterior, inicial del nuevo, fecha, motivo) ni si tiene tabla propia, así que la forma del cuerpo también queda pendiente.
23. **Motivos de cambio de estado**: una colección filtrable `/status-change-reasons` (listar y crear), con `?fromStatusId={id}` para filtrar por origen y `?origin=registration` para los motivos de alta, que no tienen estado de origen (`from_status_id` vacío, diccionario §8.3). No se usa `fromStatusId` ausente para el alta, porque un filtro ausente no filtra (§6.1).
24. ~~**Subrecurso de dirección**~~. Adoptado el 06/10/2026 (§3.3): la dirección no tiene ruta propia; viaja anidada en el alta del dueño y embebida en su respuesta; `PUT` al subrecurso corrige la misma fila y `POST` crea una fila nueva a la que pasa a apuntar el dueño (D-8); el `PUT` del dueño no la lleva. Construido en empresas; las sedes, las ubicaciones, los proveedores y las plantillas nacen así.
25. ~~**Cómo se retira un escalón territorial.**~~ Resuelto por D-45: un escalón declarado no se retira; se renombra.
26. **Autor de los hechos**: que los campos que nombran a quien opera (`changed_by`, `granted_by`, `recorded_by`) los llene también el sistema con la cuenta de la sesión y no viajen desde el cliente (§3.2). D-42 cubre solo las columnas de registro.
