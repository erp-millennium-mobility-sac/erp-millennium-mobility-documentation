# Contexto `organization` — ERP Millennium Mobility

**Actualizado:** 17/10/2026 (diccionario v1.18; sin motivo de salida, V34) · 16/10/2026 (paso 3: colaboradores y el modelo de acceso) · 15/10/2026 · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.18 (§2.5, §2.6, §3, §5.2, §5.3, §5.7, §5.9, §5.10, §6, §14 y D-55 a D-72), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), [orden de implementación](../orden-de-implementacion-millennium-mobility.md), [iam.md](iam.md) y el código de la rama `feature/access-model` del backend y del frontend al 16/10/2026. Si este documento y el diccionario no coinciden, manda el diccionario. La salida ya no guarda un motivo: `termination_reason` es un candidato del diccionario (§5.10) y V34 lo quitó.

Este documento junta lo que hace falta para entender `organization` sin recorrer los cuatro documentos por tema. Las secciones 1 a 3 están escritas para quien conoce el negocio; la 4 y la 6 son para el equipo de desarrollo.

## 1. Qué es y qué no es

`organization` responde quién es la empresa, con qué marcas vende, en qué locales trabaja su gente, dónde puede haber vehículos y quiénes son sus colaboradores. Casi todo lo demás del sistema cuelga de estas tablas: un vehículo es de una empresa y está en una ubicación; una factura la recibe una empresa; un rol se otorga sobre una empresa o una sede.

**Tablas que posee** (diccionario §3; el reparto del bloque 3b es propuesta del mapa §3)

| Tabla | Qué guarda | Diccionario | Paso |
|---|---|---|---|
| `companies` | La razón social constituida en un país: compra, factura y es dueña de los vehículos | §6.2 | 2 |
| `commercial_brands` | Los nombres bajo los que la empresa vende: Lizzo, Lima Bikes | §6.3 | 2 |
| `branches` | Los locales donde trabaja personal y desde donde se emiten comprobantes | §6.4 | 2 |
| `locations` | Los lugares donde puede haber vehículos, propios o de terceros | §6.5 | 2 |
| `employees` | Las personas que trabajan para la empresa, usen o no el sistema | §5.3 | 3 |
| `advisor_assignments` | Qué asesora atiende a qué cliente. Diseñada, se construye con ventas (diccionario §3, ajuste 2) | §5.11 | — |

**Lo que parece suyo y no lo es**

- **Las cuentas y los roles son de `iam`.** El colaborador es de `organization`; la cuenta con que entra, sus roles y sus bloqueos son de `iam` (diccionario §5.1, mapa §6.2). La columna que los une, `employees.user_id`, es de `organization`, y la escribe solo cuando `iam` le pide, por la fachada, vincular un colaborador a una cuenta. Dar acceso, restablecer la contraseña, los bloqueos y lo que la salida y el regreso le hacen a la cuenta los orquesta `iam` ([iam.md](iam.md)).
- **El alcance de quien llama tampoco es suyo.** `organization` no depende de `iam`: lee qué empresas ve y administra la cuenta de la petición del puerto `CurrentAccessScope` del núcleo `shared`, que implementa `iam` (mapa §5).
- **Las direcciones son de `location`**, pero cada dueño tiene la suya (diccionario §4.6). La empresa, la sede y la ubicación piden a `location` crear, corregir o reemplazar su dirección, y nunca la comparten (mapa §6.3, convenciones §3.3).
- **"Ubicación" aquí es un lugar físico donde hay vehículos**, no geografía. Los países, las divisiones territoriales y las direcciones son de `location` (D-6, diccionario §6.5).
- **Los tipos de documento tributario son de `location`** (D-44). La empresa y el colaborador solo eligen uno.

**Vocabulario**

| Término | Qué es | Qué no es |
|---|---|---|
| Empresa | Una razón social en un país. Hay **una sola por país** (D-5) | Lizzo y Lima Bikes no son empresas |
| Marca comercial | Bajo qué nombre vendemos: Lizzo, Lima Bikes (§6.3) | La marca del vehículo (Chevrolet, TVS), que es de `vehiclecatalog` (§7.2) |
| Sede | Un local de la empresa con personal; se le asignan roles y emite comprobantes (§6.4) | Un patio alquilado o la tienda de un dealer |
| Ubicación | Un lugar donde puede estar un vehículo: la sede propia vista como patio, un patio arrendado, un dealer, un taller (§6.5, D-6) | Un local al que se asignan personas |
| Puesto | El trabajo del colaborador: asesor, mecánico. Informativo (D-39) | Un rol: no da permisos |
| Rol | Conjunto de permisos de una cuenta, con su alcance. Es de `iam` (D-39) | Un puesto |
| Perfil | Los datos personales del colaborador (D-4) | La cuenta ni el rol |

## 2. Ciclo de vida de cada agregado

Nada de este contexto se borra. Las empresas, marcas, sedes y ubicaciones se desactivan; los colaboradores reciben una fecha de salida. La API no tiene `DELETE` (convenciones §3.1).

### Empresa

- **Quién la crea:** el superadministrador (diccionario §5.2, paso 4). Nace activa, con su país, su tipo y número de documento tributario, su razón social y nombre comercial, su teléfono, su moneda funcional y, si la tiene, su dirección fiscal.
- **Qué se cambia:** la razón social, el nombre comercial, el teléfono y la moneda funcional. La moneda funcional puede ser de otro país (§6.2).
- **Qué queda fijo desde el alta:** el país, el tipo de documento y el número. El código no los deja cambiar; el diccionario no lo dice (pregunta 6).
- **La dirección fiscal** se cambia por su propio camino (D-8): corregir modifica la misma fila; mudarse crea una fila nueva y la anterior queda para los documentos ya emitidos. Una empresa sin dirección registra la primera como una mudanza.
- **Cómo se retira:** se desactiva y se puede reactivar. Desactivar no revisa ni afecta sus sedes, marcas, ubicaciones ni colaboradores, y nada impide seguir creándolos bajo una empresa inactiva.
- **Qué deduce:** un país opera si tiene una empresa activa; el país no lo guarda (D-3).

### Marca comercial

- La crea el superadministrador bajo una empresa, con un código corto y un nombre.
- Se cambia solo el nombre. El código queda fijo desde el alta, como pide §6.3 ("código invariable").
- Se desactiva y se reactiva. Desactivarla no revisa nada que la use.

### Sede

- La crea el superadministrador (§5.2, paso 5), con código, nombre, código de establecimiento tributario opcional, zona horaria obligatoria y dirección opcional. Hereda el país de su empresa.
- Se cambian el nombre, el código de establecimiento y la zona. El código de la sede queda fijo desde el alta.
- La dirección sigue la misma regla que la de la empresa: corregir o mudarse (D-8).
- La sede no tiene tipo (D-50).
- Se desactiva y se reactiva. Desactivarla no revisa las ubicaciones ni los colaboradores que la mencionan; solo impide elegirla para una ubicación propia o un colaborador nuevo.

### Ubicación

- La crea el superadministrador bajo una empresa, con código, nombre y tipo: `sede_propia`, `patio_arrendado`, `tienda_dealer`, `taller_tercero` u `otro` (§6.5).
- Una `sede_propia` apunta a una sede activa de la misma empresa y no tiene dirección propia: su dirección es la de la sede. Las demás tienen dirección propia y no apuntan a una sede. Esta segunda parte es del código; el diccionario solo dice que la sede se enlaza "solo cuando el lugar es una sede propia" y que la dirección es "necesaria para los lugares de terceros".
- Se cambian el nombre, el tipo, la sede y la dirección. El código queda fijo.
- Se desactiva al terminar el alquiler; el historial se conserva (§6.5).

### Colaborador

- **Quién lo registra:** el superadministrador en cualquier empresa, o el administrador de la empresa en las que administra (§5.2, pasos 6 a 8; D-57). El administrador de accesos ve los colaboradores de todas las empresas y no los registra ni los modifica: quien da acceso no crea al colaborador que lo recibe. Nace activo y sin cuenta.
- **Qué se cambia:** los nombres, el documento, la sede, el puesto, el teléfono, la foto y la fecha de ingreso. La empresa queda fija.
- **Activo** significa sin fecha de salida (§5.3). La fecha de salida no puede ser posterior a hoy, así que un colaborador nunca queda inactivo antes de irse.
- **Sus fechas** se leen en la zona de su sede; sin sede, en la del país de su empresa; sin ninguna, en `America/Lima` (§2.6, D-40). La API entrega esa zona con cada colaborador (`timeZone`), y es la misma que usan la salida y el regreso para saber qué día es hoy.
- **Dar acceso** lo hace `iam` ([iam.md](iam.md), F5): crea la cuenta o vincula la que la persona ya tiene, y `organization` guarda el vínculo. Una vez vinculada, la cuenta no se cambia, y una cuenta se vincula a lo sumo a un colaborador por empresa.
- **Salida:** se registra la fecha y un motivo opcional; en la misma transacción `iam` retira los roles de la cuenta en esa empresa y, si era su último colaborador activo, la cierra (F7).
- **Reingreso:** se reactiva al colaborador con una nueva fecha de ingreso, no anterior a la salida ni posterior a hoy. Sobrescribe la fecha de ingreso y borra la fecha y el motivo de salida: el período anterior se pierde. El diccionario no prevé el reingreso y dice que un colaborador que se va conserva sus registros (§5.3); el reingreso no borra registros de otras tablas, pero sí la historia del propio colaborador (pregunta 3).

## 3. Flujos funcionales

En cada flujo, "el sistema revisa" es lo que el backend comprueba en ese orden. Los códigos de error son los de convenciones §7.2.

### F1 · Dar de alta una empresa

- **Rol:** `SUPERADMIN`. La pantalla solo se abre con ese rol y la API rechaza a cualquier otro con `403`.
- **Pantalla:** Configuración › Empresas (`/organization/companies`), diálogo de alta.
- **Pasos:**
  1. Elige el país. La pantalla carga solo los tipos de documento de ese país que aplican a empresas.
  2. Escribe el número, la razón social exacta, el nombre comercial y el teléfono, y elige la moneda funcional. La pantalla ofrece solo monedas activas y, al editar, también la que la empresa ya tiene.
  3. Llena la dirección fiscal con el componente de direcciones de `location`. El país no se elige: es el de la empresa.
  4. Guarda (`POST /companies`). El sistema revisa, en orden: que el país exista; que el país no tenga ya una empresa; que el número no esté usado en el país; que el tipo exista, sea del país, esté activo, aplique a empresas y que el número tenga su longitud; que la moneda exista y, si es nueva para la empresa, esté activa; que los textos respeten sus largos. La dirección se crea al final, cuando todo lo anterior pasó.
- **Errores visibles:** `COMPANY_COUNTRY_CONFLICT` y `COMPANY_TAX_ID_CONFLICT` (`409`); `CURRENCY_INACTIVE` (`422`) si la moneda elegida está inactiva; un tipo que no aplica o una longitud equivocada (`422`); país, tipo o moneda inexistentes (`404`); los de la dirección, que vienen de `location` (`GEODIVISION_NOT_FOUND`, código postal faltante, división que no sirve).
- **Resultado:** la empresa queda activa con su propia dirección fiscal. Si algo falla, no queda ni la empresa ni la dirección.
- **Después:** editar nombres, teléfono y moneda; corregir la dirección (si no tiene, `422 COMPANY_FISCAL_ADDRESS_MISSING`) o mudarla; el diálogo pregunta cuál de las dos es (D-8). Activar o desactivar.

### F2 · Marcas comerciales

- **Rol:** escribe `SUPERADMIN`. `ADMIN_EMPRESA` abre la pantalla y solo lee.
- **Pantalla:** Configuración › Marcas comerciales (`/organization/commercial-brands`).
- **Pasos:** elige la empresa, escribe el código (letras y números, de 2 a 20, se guarda en mayúsculas) y el nombre. El sistema revisa que la empresa exista y que el código no esté usado en ella.
- **Errores visibles:** `COMMERCIALBRAND_CONFLICT` (`409`), datos inválidos (`400`).
- **Resultado:** la marca queda activa. Después solo se cambia el nombre, o se desactiva.

### F3 · Sedes con su dirección y su zona

- **Rol:** escribe `SUPERADMIN`; `ADMIN_EMPRESA` solo lee.
- **Pantalla:** Configuración › Sedes (`/organization/branches`).
- **Pasos:** elige la empresa; escribe código (letras, números y guiones, de 2 a 20), nombre, código de establecimiento tributario si lo tiene, y elige la zona horaria por su nombre IANA (`America/Lima`); llena la dirección si la tiene. El sistema revisa que la empresa exista, que el código y el código de establecimiento no estén usados en ella y que la zona sea un nombre IANA válido; la dirección se crea al final, en el país de la empresa.
- **Errores visibles:** `BRANCH_CODE_CONFLICT` y `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT` (`409`); zona inválida (`400`); los de la dirección; `BRANCH_ADDRESS_MISSING` (`422`) al corregir una dirección que no existe.
- **Resultado:** la sede queda activa en el país de su empresa. Después se cambian nombre, código de establecimiento y zona; la dirección se corrige o se muda, y el diálogo pregunta cuál de las dos es (D-8).

### F4 · Ubicaciones

- **Rol:** escribe `SUPERADMIN`, y la pantalla ofrece el alta y la edición solo a él (D-43). La lista la lee cualquier cuenta que alcance la empresa: los roles de sistema, todas.
- **Pantalla:** Configuración › Ubicaciones (`/organization/locations`).
- **Pasos:** elige la empresa, escribe código y nombre y elige el tipo. Si es `sede_propia`, elige la sede y no escribe dirección; si es otro tipo, escribe la dirección y no elige sede. El sistema revisa que la empresa exista, que el código no esté usado en ella, y según el tipo: que la sede exista, sea de la misma empresa y esté activa, o que llegue una dirección y ninguna sede.
- **Errores visibles:** `LOCATION_CODE_CONFLICT` (`409`); sede de otra empresa, sede inactiva, sede o dirección que faltan o sobran (`400`); sede inexistente (`404`); `LOCATION_ADDRESS_NOT_ALLOWED` al corregir o mudar la dirección de una `sede_propia` y `LOCATION_ADDRESS_MISSING` al corregir una que no existe (`422`).
- **Resultado:** la ubicación queda activa. Al editarla, la dirección que viaja en el formulario corrige la fila existente o crea la primera; la pantalla no pregunta si es una mudanza. La API tiene además `PUT` y `POST /locations/{id}/address` para corregir o mudar.

### F5 · Registrar un colaborador

- **Rol:** `SUPERADMIN` en cualquier empresa; `ADMIN_EMPRESA` en las empresas que administra (D-57). Una empresa fuera del alcance de la cuenta responde `404 COMPANY_NOT_FOUND`; una que la cuenta ve pero no administra, `403 ACCESS_DENIED`. El administrador de accesos ve los colaboradores de todas las empresas, de solo lectura.
- **Pantalla:** Configuración › Colaboradores (`/organization/employees`). El selector de empresa, que aparece cuando la cuenta alcanza más de una, lista las empresas activas de su alcance (`GET /companies`, que ya filtra por él) por su nombre comercial. Donde la sesión administra la empresa, la pantalla ofrece crear, editar, registrar la salida y el regreso; donde solo la ve, muestra los colaboradores sin esas acciones y sin avisos de error. La columna de cuenta dice si el colaborador tiene una (`hasAccount`) y qué roles tiene en la empresa; a quien maneja cuentas le ofrece además las acciones de cuenta de `iam` (F6 a F9).
- **Pasos:** escribe nombres y apellidos; elige el tipo de documento entre los de personas del país de la empresa y escribe el número; elige la sede si tiene una fija; escribe puesto, teléfono, foto y fecha de ingreso. El diálogo lee las sedes activas de la empresa elegida en su propio estado, así cambiar de empresa dentro del diálogo no cambia las sedes que ofrece el filtro de la lista, y avisa si no las pudo leer; la lista avisa lo mismo de las sedes de su filtro. El sistema revisa: que la empresa exista y esté en el alcance; que quien opera la administre; que el tipo de documento exista, sea del país, esté activo, aplique a personas y que el número tenga su longitud; que la sede exista, sea de la empresa y esté activa; que el documento no esté registrado en la empresa.
- **Errores visibles:** documento repetido, `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` (`409`); tipo o sede que no sirven (`422`); empresa fuera del alcance, tipo o sede inexistentes (`404`); empresa que no administra (`403`). Si la lista de tipos de documento no carga, el formulario no lo avisa y el selector queda vacío.
- **Resultado:** el colaborador queda activo y sin cuenta, registrado a nombre de quien lo dio de alta. Editarlo después (`PUT /employees/{id}`) repite las mismas revisiones.

### F6 · Dar acceso a un colaborador

Es una operación de `iam` ([iam.md](iam.md), F5): la hacen el superadministrador y el administrador de accesos, desde la acción "Dar acceso al sistema" de esta pantalla, con `POST /access-grants`. El administrador de empresa no da acceso (D-57).

Lo que pone `organization`, por su fachada:

- **El colaborador, bloqueado.** `iam` lo lee con su fila bloqueada hasta que termina la operación, así un acceso y una salida del mismo colaborador corren uno después del otro.
- **El vínculo.** `linkUserToEmployee` revisa que el colaborador esté activo y sin cuenta, y que la cuenta no esté vinculada a otro colaborador de la misma empresa, y escribe `employees.user_id`. La base impide lo mismo con `employees_company_user_uq` (V26). Si el vínculo falla, `iam` deshace la cuenta que acababa de crear.

El diálogo pide solo el correo: los roles se otorgan después, en el diálogo Roles ([iam.md](iam.md), F6).

### F7 · Salida y regreso

Las rutas las sirve `iam`, que decide quién puede y qué le pasa a la cuenta ([iam.md](iam.md), F9); `organization` registra el hecho y sus fechas, en la misma transacción.

- **Rol:** `SUPERADMIN`, o el `ADMIN_EMPRESA` de la empresa del colaborador. El administrador de accesos no registra salidas ni regresos (D-57). Nadie registra su propia salida ni su propio regreso (`403 OWN_ACCOUNT_FORBIDDEN`, D-70).
- **Pantalla:** Configuración › Colaboradores, acciones "Dar de baja" y "Reincorporar colaborador", solo donde la sesión administra la empresa. Los dos diálogos calculan qué día es hoy con la zona que trae el colaborador (`timeZone`), la misma que usa el backend.
- **Salida** (`POST /employees/{id}/termination`): fecha de salida y un motivo opcional de hasta 500 caracteres. `organization` revisa que el colaborador esté activo, que la fecha no sea anterior al ingreso y que no sea posterior a hoy en la zona del colaborador. La salida se registra cuando ocurre: una fecha futura retiraría los roles antes de que la persona se vaya. Lo que le pasa a la cuenta —retiro de los roles en esa empresa y, si era su último colaborador activo, el cierre— lo hace `iam` en la misma transacción.
- **Reingreso** (`POST /employees/{id}/reactivation`): el diálogo pide la nueva fecha de ingreso. `organization` revisa que el colaborador haya salido, que la fecha no sea anterior a la salida y que no sea posterior a hoy en la zona del colaborador. Sobrescribe la fecha de ingreso y borra la salida y su motivo (pregunta 3). Si la salida había cerrado la cuenta, `iam` cierra la `baja`; los roles no vuelven solos.
- **Errores visibles:** cada regla con su código, todos `422`: `EMPLOYEE_ALREADY_TERMINATED`, `EMPLOYEE_TERMINATION_BEFORE_HIRE`, `EMPLOYEE_TERMINATION_DATE_IN_FUTURE`, `EMPLOYEE_ALREADY_ACTIVE`, `EMPLOYEE_REHIRE_BEFORE_TERMINATION` y `EMPLOYEE_REHIRE_DATE_IN_FUTURE`; además `LAST_ACTIVE_SUPERADMIN`, `404 EMPLOYEE_NOT_FOUND` para un colaborador de una empresa fuera del alcance y los `403` de quién puede. `EMPLOYEE_ALREADY_TERMINATED` y `EMPLOYEE_ALREADY_ACTIVE` dicen que la lista quedó vieja, y la pantalla la vuelve a leer.
- **Resultado:** la respuesta es `{id, isActive, hireDate, terminationDate, terminationReason}`, y la pantalla vuelve a leer la lista.

### F8 · Restablecer la contraseña de un colaborador

Es una operación de `iam` sobre la cuenta ([iam.md](iam.md), F8), para el superadministrador y el administrador de accesos, desde la acción "Restablecer contraseña" de esta pantalla o de Seguridad › Usuarios. La ruta `/employees/{id}/password-reset` ya no existe.

### F9 · Bloquear la cuenta de un colaborador

El bloqueo es de la cuenta, no del colaborador (§5.4): es de `iam` ([iam.md](iam.md), F7). La acción "Bloqueos" de esta pantalla abre el mismo diálogo que Seguridad › Usuarios, para el superadministrador y el administrador de accesos, donde se bloquea, se levanta o se cancela cada período. Las rutas `/employees/{id}/suspensions` ya no existen en el frontend.

## 4. Reglas de negocio

| ID | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| ORG-R01 | Una sola empresa por país | D-5, §6.2 | Base: `UNIQUE (country_id)` (V12). Servicio: `COMPANY_COUNTRY_CONFLICT` | `CompaniesControllerTests.creationForSameCountryReturns409ConflictDecisionD5`; `CompanyCommandServiceImplTests.creatingASecondCompanyForTheCountryFailsWithItsOwnConflict` | ✅ |
| ORG-R02 | El documento tributario no se repite en el país y se guarda sin espacios, guiones ni puntos, en mayúsculas | §6.2 | Base: `UNIQUE (country_id, tax_id)` (V12). Dominio: normaliza. Servicio: `COMPANY_TAX_ID_CONFLICT` | `CompanyCommandServiceImplTests.creatingWithATaxIdAlreadyUsedInTheCountryFailsWithItsOwnConflict`; `CompanyTests.theTaxIdIsStoredWithoutSeparatorsAndInUpperCase` | ✅ |
| ORG-R03 | El tipo de documento de la empresa es de su país, está activo, aplica a empresas, y el número tiene su longitud | §6.2, §4.7 | Base: el país, con llave compuesta hacia `tax_id_types` (V15). Servicio: el resto, con `TaxIdTypeRules` | `TaxIdTypeRulesTests`; `CompaniesControllerTests.creationWithATypeThatDoesNotApplyToCompaniesReturns422` y `creationWithANumberOfTheWrongLengthReturns422` | ✅ |
| ORG-R04 | Un número cuyo tipo declara dígito de control se verifica al registrarlo | §4.7 | En ningún lado: ni la empresa ni el colaborador lo calculan | Sin prueba | ❌ |
| ORG-R05 | La dirección fiscal es solo de esa empresa y está en su país | §4.6; mapa §6.3 (el país, propuesta) | Base: `UNIQUE (fiscal_address_id)` y llave compuesta hacia `addresses` (V18). Servicio: la crea en el país de la empresa | `CompanyFiscalAddressConstraintTests` (el país); la unicidad, sin prueba | ⚠️ |
| ORG-R06 | Corregir una dirección modifica su fila; mudarse crea una nueva; el `PUT` del dueño no toca la dirección | D-8; convenciones §3.3 | Servicio | `theFirstAddressACorrectionAndAMoveFollowD8` en `CompaniesControllerTests` y `BranchesIntegrationTests`. Ubicaciones: el `PUT /locations/{id}` sí corrige la dirección | ⚠️ |
| ORG-R07 | La sede está en el país de su empresa, y su dirección es solo suya y de ese país | §6.4, §4.6 | Base: llaves compuestas hacia `companies` y `addresses` y `UNIQUE (address_id)` (V20). Servicio | `BranchConstraintTests` (tres casos) | ✅ |
| ORG-R08 | El código de la sede y su código de establecimiento no se repiten en la empresa; el código no cambia | §6.4; convenciones §7.2 | Base: dos `UNIQUE` (V17, V20). Servicio: un código de conflicto por campo. Dominio: el código es fijo | `BranchCommandServiceImplTests`; `BranchConstraintTests.aTaxEstablishmentCodeDoesNotRepeatWithinTheCompany`; `BranchesIntegrationTests.theBranchUpdateNeverTouchesTheAddressNorTheCode` | ✅ |
| ORG-R09 | El nombre de una sede o de una marca puede repetirse en la empresa | §6.3, §6.4 | Base: V20 quitó las dos `UNIQUE` | `theNameOfABranchMayRepeatWithinItsCompany`; `theNameOfABrandMayRepeatWithinItsCompany` | ✅ |
| ORG-R10 | La zona horaria de la sede es obligatoria y es un nombre IANA, sin valor por defecto | §6.4, §2.6, D-40 | Dominio. Base: `NOT NULL` sin default (V20) | `BranchTests.invalidTimezoneThrowsException`; `BranchesIntegrationTests.theZoneIsRequiredAndIsAnIanaName` | ✅ |
| ORG-R11 | La sede no lleva tipo | D-50 | Base: V20 quitó la columna | No hace falta: la columna no existe | ✅ |
| ORG-R12 | El código de la marca no se repite en la empresa, se guarda en mayúsculas y no cambia | §6.3 | Base: `UNIQUE (company_id, code)` (V16). Servicio: `COMMERCIALBRAND_CONFLICT`. Dominio | `CommercialBrandCommandServiceImplTests.returnsConflictWhenBrandCodeAlreadyExistsUnderCompany`; `CommercialBrandTests` | ✅ |
| ORG-R13 | El código de la ubicación no se repite en la empresa y no cambia; el tipo es uno de los cinco | §6.5 | Base: `UNIQUE (company_id, code)` y `CHECK` del tipo (V21). Servicio: `LOCATION_CODE_CONFLICT` | `LocationCommandServiceImplTests.shouldRejectDuplicateCode`; `LocationsIntegrationTests` | ✅ |
| ORG-R14 | Una `sede_propia` apunta a una sede y no tiene dirección propia; las demás tienen dirección y no apuntan a una sede | §6.5 (la exclusión de la dirección es del código) | Dominio y servicio. Base en parte: el `CHECK` de V21 no prohíbe la dirección en una `sede_propia` | `LocationTests` (seis casos); `LocationCommandServiceImplTests.shouldRejectAddressCorrectionForOwnedFacility` | ✅ |
| ORG-R15 | La sede de una ubicación propia es de la misma empresa y está activa | §6.5 | Servicio (`LocationFacilityPolicy`). Base: la empresa, con la llave compuesta `locations_branch_same_company_fk` (V26) | `LocationCommandServiceImplTests.shouldRejectOwnedLocationFromDifferentCompanyBranch`, `OrganizationOwnershipKeysMigrationTests.theBranchOfALocationIsOfItsCompany`; la sede inactiva, sin prueba | ⚠️ |
| ORG-R16 | La dirección de una ubicación es solo suya y está en el país de la empresa | §4.6; mapa §6.3 | Servicio: la crea en el país de la empresa. Base: `locations_address_uq` (V26); falta la llave compuesta del país | `OrganizationOwnershipKeysMigrationTests.twoLocationsCannotShareAnAddress`; el país, sin prueba | ⚠️ |
| ORG-R17 | El tipo de documento del colaborador es del país de la empresa, está activo, aplica a personas, y el número tiene su longitud | §5.3, §4.7 | Servicio (`TaxIdTypeRules.rejectionForIndividual`). Base: llave simple, sin el país | `EmployeeCommandServiceImplTests.shouldFailWhenDocumentTypeDoesNotApplyToIndividuals`; el país y la longitud, sin prueba | ⚠️ |
| ORG-R18 | El documento de identidad no se repite en la empresa | §5.3 | Base: `UNIQUE (company_id, document_type_id, document_number)` (V22), que deja pasar el mismo número con otro tipo. Servicio: `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` | `EmployeeCommandServiceImplTests.shouldFailWhenDocumentAlreadyExistsInCompany`; `EmployeesIntegrationTests` | ⚠️ |
| ORG-R19 | La sede del colaborador es de su empresa y está activa | §5.3 | Servicio (`EmployeeAffiliationPolicy`). Base: la empresa, con la llave compuesta `employees_branch_same_company_fk` (V26) | `EmployeeCommandServiceImplTests.shouldFailWhenBranchBelongsToDifferentCompany`, `OrganizationOwnershipKeysMigrationTests.theBranchOfAnEmployeeIsOfItsCompany` | ✅ |
| ORG-R20 | Una cuenta se vincula a lo sumo a un colaborador por empresa | §5.3 | Base: `employees_company_user_uq` (V26). Servicio: `linkUserToEmployee` | `OrganizationOwnershipKeysMigrationTests.anAccountLinksToOneEmployeePerCompany`, `theSameAccountLinksToEmployeesOfTwoCompanies`; `OrganizationContextFacadeImplTests.anAccountIsLinkedToOneEmployeePerCompany` | ✅ |
| ORG-R21 | Activo es sin fecha de salida; la salida no es anterior al ingreso, no es posterior a hoy en la zona del colaborador y no se registra dos veces; cada regla responde su propio código | §5.3, D-60 | Dominio y servicio (`EmployeeCommandServiceImpl`). Base: sin `CHECK` | `EmployeeTests.terminationDateBeforeHireDateThrows` y `terminatingAlreadyTerminatedThrows`; `EmployeeTenureIntegrationTests.anExitIsRecordedWhenItHappensNotAhead`, `eachRuleOfTheExitAnswersItsOwnCode` | ✅ |
| ORG-R22 | Al registrar la salida se retiran los roles de la cuenta **en esa empresa**, con motivo `baja_colaborador` | §5.3, D-38, D-41 | Servicio de `iam` (`EmployeeTenureCommandServiceImpl`), en la misma transacción que la salida ([iam.md](iam.md), IAM-R40) | `EmployeeTenureIntegrationTests.theSameRoleOnTwoCompaniesSurvivesLeavingOneOfThem` | ✅ |
| ORG-R23 | Si a la cuenta no le queda colaborador activo, la salida la cierra con un bloqueo `baja` | §5.3, D-38, D-12, D-60 | Servicio de `iam`, con el dato de las otras empresas leído con alcance `SYSTEM` ([iam.md](iam.md), IAM-R40, R41) | `EmployeeTenureIntegrationTests.leavingTheLastCompanyTakesEveryRoleAway`, `CrossCompanyAccountAdministrationIntegrationTests.theExitFromItsCompanyDoesNotCloseAnAccountThatStillWorksForAnotherOne` | ✅ |
| ORG-R24 | No se registra la salida del último superadministrador activo | §5.9, D-65 | Servicio de `iam` (`LastSuperadminGuard`) | `EmployeeTenureIntegrationTests.theLastSuperadminCannotLeave`, `aSuperadminCannotLeaveWhileTheOnlyOtherOneHasALeaveScheduled` | ✅ |
| ORG-R25 | Se da acceso solo a un colaborador activo, y la cuenta queda vinculada en la misma operación; si el correo ya tiene cuenta, se vincula a ella | D-38, D-58, §5.3 | Servicio de `iam` (`UserCommandServiceImpl`) y la fachada (`linkUserToEmployee`) ([iam.md](iam.md), IAM-R36) | `AccessGrantsIntegrationTests.aTerminatedEmployeeIsNotGivenAccess`, `OrganizationContextFacadeImplTests.aTerminatedEmployeeIsNotLinked`, `linkingAnAccountRecordsItOnTheEmployee` | ✅ |
| ORG-R26 | La sede y la marca de una asignación son de la empresa de la asignación | §5.7 | Base: llaves compuestas de `user_roles` (V27). Servicio: la fachada las resuelve dentro de la empresa (`resolveScope`) | `OrganizationContextFacadeImplTests.aBranchOrABrandOfAnotherCompanyIsNotFoundInThisOne`, `RoleAssignmentScopeMigrationTests.theBranchIsOfTheCompanyOfTheAssignment`, `theBrandIsOfTheCompanyOfTheAssignment` | ✅ |
| ORG-R27 | El primer superadministrador no da acceso ni otorga roles hasta vincularse a su perfil | §5.2, §5.9, D-38 | Servicio de `iam` ([iam.md](iam.md), IAM-R14) | `RoleAssignmentsIntegrationTests.theFirstSuperadminGrantsNothingUntilItLinksItsOwnAccount` | ✅ |
| ORG-R28 | ~~`ADMIN_EMPRESA` no otorga `SUPERADMIN` ni `ADMIN_EMPRESA`~~ | §5.5 (reglas 1 y 2); procesos P2 | Retirada el 16/10/2026: el administrador de empresa no otorga ningún rol (D-57; [iam.md](iam.md), IAM-R31) | — | — |
| ORG-R29 | Registran y modifican colaboradores el superadministrador, en cualquier empresa, y el administrador de la empresa, en las que administra; el administrador de accesos no. Una empresa fuera del alcance responde `404`; una que se ve sin administrarla, `403` | §2.5, D-57 | Servicio (`EmployeeCommandServiceImpl`, con `CurrentAccessScope.administers`); Frontend (crear y editar solo donde la sesión administra la empresa) | `EmployeeAdministrationIntegrationTests` (los cinco casos), `AccessAdministratorIntegrationTests.theAccessAdministratorNeitherWritesEmployeesNorRecordsTheirExitsAndReturns` | ✅ |
| ORG-R30 | Solo `SUPERADMIN` escribe empresas, marcas, sedes y ubicaciones | §5.2 (pasos 4 y 5), D-43; procesos §1 y §3.2 | Servicio: `@PreAuthorize`. Frontend: las pantallas ofrecen las escrituras solo a `SUPERADMIN` | `CompaniesControllerTests.creationWithoutSuperadminReturns403`; `BranchesIntegrationTests.changingTheAddressOrTheStatusWithoutSuperadminReturns403`; marcas y ubicaciones, sin prueba | ⚠️ |
| ORG-R31 | La lectura de empresas, marcas, sedes, ubicaciones y colaboradores se filtra por el alcance de la cuenta —en la lista, la exportación, por identificador y en las rutas anidadas—, y una empresa ajena, o lo que le pertenece, responde `404` | §2.5, §5.8; convenciones §9.2 | Servicio: cada consulta filtra con `CurrentAccessScope`. Base: Row Level Security (V29) | `CompanyIsolationIntegrationTests` (diez casos), `RowLevelSecurityMigrationTests` | ✅ |
| ORG-R32 | Un país opera si tiene una empresa activa; no se guarda | D-3 | Servicio: `GET /companies?countryId=…&active=true`. La fachada (`existsCompanyByCountryId`) no mira si la empresa está activa | `CompanyQueryServiceImplTests.searchingPassesTheTextTheStatusTheInternalCountryAndThePageToTheRepository` | ⚠️ |
| ORG-R33 | Nada se borra: se desactiva o se registra la salida | §5.3, §6.2, §6.4, §6.5; convenciones §3.1 | Servicio: no hay `DELETE` | `405` en `BranchesIntegrationTests`, `CommercialBrandsIntegrationTests` y `EmployeesIntegrationTests`; empresas y ubicaciones, sin prueba | ⚠️ |
| ORG-R34 | Columnas de registro: la empresa guarda quién la creó y quién la cambió; el colaborador, quién lo registró; marcas, sedes y ubicaciones, solo fechas | D-42; §5.3, §6.2 a §6.5 | Base y servicio: clases base compartidas y el auditor de `iam` (`Revised` en empresas, `Authored` en colaboradores, `Auditable` en el resto) | `EmployeeAuditTests` (autor); `CompanyAuditTests`, `BranchAuditTests`, `CommercialBrandAuditTests`, `LocationAuditTests` (fechas); el autor de la empresa, sin prueba | ⚠️ |
| ORG-R35 | La moneda funcional nueva está activa; la que la empresa ya tiene se conserva aunque se haya desactivado | §4.1, §6.2 | Dominio: `FunctionalCurrency`, que recibe el estado de `location` por `CurrencySummary.isActive`. Servicio: `CompanyCommandServiceImpl`, que responde `CURRENCY_INACTIVE` (`422`). Frontend: el diálogo ofrece las activas más la actual (rama `feature/location-closure`) | `FunctionalCurrencyTests`, `CompanyCommandServiceImplTests`, `CompaniesControllerTests` | ✅ |
| ORG-R36 | El filtro de sede de los colaboradores de una empresa nunca se descarta: una sede desconocida o de otra empresa da una lista vacía | convenciones §6.1 | Servicio (`EmployeeQueryServiceImpl`) | `CompanyIsolationIntegrationTests.aBranchFilterOutsideTheCompanyListsNoEmployee` | ✅ |
| ORG-R37 | El regreso tiene una fecha no anterior a la salida ni posterior a hoy en la zona del colaborador; cada regla responde su propio código | §5.3, D-60 | Servicio (`EmployeeCommandServiceImpl`) | `EmployeeTenureIntegrationTests.eachRuleOfTheReturnAnswersItsOwnCode`, `aReturnIsRecordedWhenItHappensNotAhead` (un regreso con fecha de mañana); `EmployeeCommandServiceImplTests.shouldRejectAReturnInTheFuture` | ✅ |
| ORG-R38 | Las fechas de un colaborador se leen en la zona de su sede; sin sede, en la del país de su empresa; sin ninguna, en `America/Lima`. La API la entrega con cada colaborador (`timeZone`), resuelta en lote | §2.6, D-40 | Dominio (`EmployeeAffiliationPolicy.timeZoneOf`, la única regla); Servicio (`EmployeeCommandServiceImpl` para la salida y el regreso; la vista del colaborador para `timeZone`); Frontend (los diálogos de salida y regreso) | `EmployeeTimeZoneIntegrationTests.theZoneOfAnEmployeeIsTheOneOfItsBranchElseTheOneOfTheCountryOfItsCompany` (una sede en otra zona), `aNewEmployeeIsAnsweredWithTheZoneOfItsBranch`; `EmployeeAffiliationPolicyTests.theZoneOfAnEmployeeIsTheOneOfItsBranchElseTheOneOfItsCountryElseLima` | ✅ |
| ORG-R39 | Una cuenta con colaboradores y roles en dos empresas usa las pantallas de colaboradores de las dos; una empresa donde no tiene rol responde `404` aunque tenga un colaborador ahí | §5.3, §5.7 | Servicio: el alcance sale de las asignaciones (`CurrentAccessScope`), no de "el colaborador de la cuenta" | `MultiCompanyEmployeeAccessIntegrationTests` (tres casos), `EmployeeAdministrationIntegrationTests.theAdministratorOfTwoCompaniesRegistersAndChangesTheEmployeesOfBoth` | ✅ |

## 5. Relación con otros contextos

**Las dependencias.** `organization` declara `allowedDependencies = {"location::acl"}` y no llama a `iam`: es `iam` quien llama a `organization` (mapa §4, decisión 4 resuelta en lo que toca al sentido). El alcance de quien llama lo lee del puerto `CurrentAccessScope` del núcleo `shared`, que implementa `iam`, igual que el autor de sus registros sale del auditor que aporta `iam` (D-42). `ExternalIamService` ya no existe. `ModularityTests` lo verifica.

**Lo que ofrece: `OrganizationContextFacade`.** Su consumidor es `iam` (`ExternalOrganizationService`). Lo que le da:

| Para qué | Miembros |
|---|---|
| El colaborador que se da de acceso, sale o regresa, con su fila bloqueada hasta que termina la transacción | `fetchEmployeeSummaryForUpdate` |
| En qué empresas tiene colaboradores una cuenta | `fetchEmployeeSummariesByUserId`, `fetchActiveEmployeeCompanyIdsByUserId` |
| Vincular un colaborador a una cuenta | `linkUserToEmployee` |
| Registrar la salida y el regreso, con sus reglas de fechas y "hoy" en la zona del colaborador | `terminateEmployee`, `reactivateEmployee` |
| Resolver el alcance de una asignación: la empresa, y la sede y la marca de esa empresa | `resolveScope` |
| Nombres en lote para las listas de `iam`, las empresas activas y la zona de cada empresa para `/me`, y las cuentas con colaborador en una empresa | `fetchCompanyReferencesByIds`, `fetchBranchReferencesByIds`, `fetchCommercialBrandReferencesByIds`, `fetchActiveCompanyReferences`, `fetchCompanyTimeZonesByIds`, `fetchLinkedEmployeePublicIdsByCompanyId` |

Las lecturas de la fachada no aplican el filtro de alcance de la API: `iam` decide quién puede actuar sobre lo que pide, y con alcance `SYSTEM` solo lee datos sueltos (si la cuenta trabaja para otra empresa), nunca agregados. Siguen además los resúmenes de empresa, marca, sede y ubicación, y si una marca, sede o ubicación es de una empresa, que ningún contexto consume todavía. Lo que el mapa §5.1 proponía para el registro público (registrar el colaborador a nombre de `SYSTEM`) espera a D-32.

**Lo que usa de `location`** (`ExternalLocationService`): el identificador interno de un país y, en lote, sus nombres e identificadores públicos; el identificador de una moneda y, en lote, sus códigos ISO; las reglas de un tipo de documento (`TaxIdTypeRules`) y, en lote, sus códigos, nombres e identificadores públicos; crear, corregir y reemplazar una dirección en el país que fija el dueño; y leer direcciones en lote.

**Frontend.** `OrganizationFacade` devuelve vistas mínimas de ubicaciones y colaboradores (`LocationOption`, `EmployeeOption`) y ningún otro contexto la usa; ya no ofrece `CompanyOption`, que figura en el mapa §9. El contexto usa `LocationFacade` para países, monedas y tipos de documento; `mm-address-fields` en los formularios de empresa, sede y ubicación; e `IamFacade` para los roles de la sesión, si la sesión maneja cuentas (`managesAccess`), si administra una empresa (`administers`), las cuentas de los colaboradores de la empresa y los diálogos de cuenta ([iam.md](iam.md), §5). `currentCompanyId` ya no existe: la sesión trae todas sus empresas.

## 6. Estado y pendientes para cerrar

**Construido.** Los cuatro agregados del paso 2 y los colaboradores del paso 3, con el modelo de acceso: el filtro de alcance en cada lectura, la frontera entre empresas en la base y las llaves que mantienen cada registro bajo su empresa (V26). Del paso 2 falta `roles.company_id` (diccionario §3).

| Agregado | Backend | Frontend | Pruebas de extremo a extremo |
|---|---|---|---|
| Empresa | API, dirección fiscal anidada, migraciones V12, V15 y V18 | Configuración › Empresas, solo `SUPERADMIN` | Flujo y captura |
| Marca comercial | API, migraciones V16 y V20 | Configuración › Marcas comerciales | Flujo y captura |
| Sede | API, dirección anidada, migraciones V17 y V20 | Configuración › Sedes | Flujo y captura |
| Ubicación | API, dirección anidada, migraciones V21 y V26 | Configuración › Ubicaciones | Ninguna |
| Colaborador | API con alta, edición, indicadores y la salida y el regreso que orquesta `iam`; migraciones V22, V23 y V26 | Configuración › Colaboradores, con las acciones de cuenta de `iam` según el rol de la sesión | Flujos (`access.spec.ts`), captura y accesibilidad en los tres anchos; todas pasan en la verificación final |

Cada agregado tiene pruebas de dominio, de servicio, de persistencia y de integración en el backend, y pruebas unitarias en el frontend. Las del backend corren como `erp_mm_app`, la cuenta de la aplicación, que no puede saltarse Row Level Security. La verificación final está en [iam.md](iam.md), sección 6.

**La rama.** `feature/access-model` lleva el paso 3 de `organization` e `iam`. Los hallazgos críticos y altos de la revisión del 14/10/2026 están resueltos: la toma de cuentas al dar acceso, los roles sin empresa, la salida que no cerraba la cuenta, la búsqueda del colaborador de una cuenta con un solo resultado, el filtro de alcance y las suspensiones sin backend. El detalle está en la [revisión del 16/10/2026](../retroalimentacion/2026-10-16-iam.md).

### a) Decisiones

| Qué | Quién |
|---|---|
| Si existe el reingreso y cómo conserva el período anterior (pregunta 3) | Responsable del diccionario |
| Si `termination_reason` entra al diccionario (V23 lo agregó sin respaldo) | Responsable del diccionario |

### b) Código

**Medio**

- El reingreso borra el período anterior (pregunta 3).
- V21 sigue más débil que V18 y V20 en un punto: `locations.address_id` no tiene la llave compuesta del país (ORG-R16, LOC-R28).
- Desactivar una empresa o una sede no revisa lo que depende de ella, y se pueden crear sedes, marcas, ubicaciones y colaboradores bajo una empresa inactiva.
- El formulario de colaborador no avisa cuando no cargan los tipos de documento.
- `/companies/{id}/employees/metrics` no lo usa ninguna pantalla desde que el frontend quitó las tarjetas de indicadores de Colaboradores.
- Ubicaciones no tiene prueba de extremo a extremo ni simulación de su API.

**Bajo**

- `PUT /locations/{id}` corrige la dirección, a diferencia de empresas y sedes (ORG-R06).
- La fachada responde que un país tiene empresa aunque esté inactiva (ORG-R32).
- No se verifica el dígito de control (ORG-R04, LOC-R35): sigue como pendiente del paso 3 ([orden de implementación](../orden-de-implementacion-millennium-mobility.md)).
- `employees.document_type_id` no tiene la llave del mismo país: necesita `employees.country_id` (ORG-R17, LOC-R37).

### c) Datos

- Razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales y colaboradores reales (diccionario §14).
- La matriz de permisos por rol, que decide qué hace cada rol más allá de los que manejan cuentas y administran empresas, y qué lee un rol global; llega antes del paso 5 (procesos, pregunta 5; orden de implementación, ajuste 3).

### d) Documentación

- El diccionario (§3, columna *Código*, y §5.13) todavía describe el paso 3 como pendiente. La propuesta de la v1.18 lo actualiza; la valida el responsable del diccionario.
- El README del backend dice que `organization` tiene solo empresas.

## 7. Preguntas abiertas

1. ~~**¿Dónde se registra la salida y cómo se protege al último superadministrador?**~~ Resuelto el 16/10/2026: la orquesta `iam`, que valida y le pide a `organization` registrar el hecho (mapa §11, decisión 2, opción A); la ruta se mantiene ([convenciones](../convenciones-de-la-api-millennium-mobility.md) §14.17).
2. ~~**¿En qué sentido va la dependencia entre `iam` y `organization`?**~~ Resuelto el 16/10/2026: `iam` depende de `organization`, y `organization` lee el alcance por el puerto de `shared` (mapa §11, decisión 4).
3. **¿Existe el reingreso?** Si existe, ¿conserva el período anterior en otra fila o en un historial, dado que el documento no puede repetirse en la empresa? Pregunta nueva, a registrar en el [diccionario](../diccionario-de-dtos-millennium-mobility.md) §5.3; la propuesta v1.18 la lleva a §5.13 y §14.
4. **¿Qué permisos tiene cada rol?** Decide quién ve y escribe cada pantalla de Configuración. [Procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), apartado 5, pregunta 5, y diccionario §14 (dato pendiente).
5. ~~**¿Cómo se llama y qué responde dar acceso?**~~ Resuelto el 16/10/2026: `POST /access-grants` con `employeeId` y `email`; `201` si crea la cuenta, con la contraseña provisional, y `200` si vincula una existente, sin contraseña ([convenciones](../convenciones-de-la-api-millennium-mobility.md) §14.15).
6. **¿Se puede corregir el país, el tipo o el número de documento de una empresa mal registrada?** El código no lo permite y el diccionario no lo dice. Pregunta nueva, a registrar en el [diccionario](../diccionario-de-dtos-millennium-mobility.md) §6.2.
7. **¿Cómo lista el registro público, sin sesión, las sedes y los tipos de documento?** [Convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), §14.16.
