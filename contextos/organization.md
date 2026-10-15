# Contexto `organization` — ERP Millennium Mobility

**Actualizado:** 14/10/2026 · **Estado:** borrador para validar · **Fuente:** diccionario de datos v1.15 (§2.5, §3, §5.2, §5.3, §5.7, §5.9, §6 y §14), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), [orden de implementación](../orden-de-implementacion-millennium-mobility.md) y el código de la rama `feature/location` del backend y del frontend al 14/10/2026. Si este documento y el diccionario no coinciden, manda el diccionario.

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

- **Las cuentas y los roles son de `iam`.** El colaborador es de `organization`; la cuenta con que entra, sus roles y sus bloqueos son de `iam` (diccionario §5.1, mapa §6.2). La columna que los une, `employees.user_id`, es de `organization`.
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

- **Quién lo registra:** el superadministrador en cualquier empresa, o el administrador de la empresa en la suya (§5.2, pasos 6 a 8). Nace activo y sin cuenta.
- **Qué se cambia:** los nombres, el documento, la sede, el puesto, el teléfono, la foto y la fecha de ingreso. La empresa queda fija.
- **Activo** significa sin fecha de salida (§5.3). Una fecha de salida futura lo deja inactivo desde el momento en que se registra.
- **Dar acceso** crea o reutiliza una cuenta y la vincula (D-38, flujo F6). Una vez vinculada, la cuenta no se cambia.
- **Salida:** se registra la fecha y un motivo opcional, y se retiran los roles de la cuenta (F7).
- **Reingreso:** el código permite reactivar a un colaborador con una nueva fecha de ingreso. Sobrescribe la fecha de ingreso y borra la fecha y el motivo de salida: el período anterior se pierde. El diccionario no prevé el reingreso y dice que un colaborador que se va conserva sus registros (§5.3); el reingreso no borra registros de otras tablas, pero sí la historia del propio colaborador (pregunta 3).

## 3. Flujos funcionales

En cada flujo, "el sistema revisa" es lo que el backend comprueba en ese orden. Los códigos de error son los de convenciones §7.2.

### F1 · Dar de alta una empresa

- **Rol:** `SUPERADMIN`. La pantalla solo se abre con ese rol y la API rechaza a cualquier otro con `403`.
- **Pantalla:** Configuración › Empresas (`/organization/companies`), diálogo de alta.
- **Pasos:**
  1. Elige el país. La pantalla carga solo los tipos de documento de ese país que aplican a empresas.
  2. Escribe el número, la razón social exacta, el nombre comercial y el teléfono, y elige la moneda funcional.
  3. Llena la dirección fiscal con el componente de direcciones de `location`. El país no se elige: es el de la empresa.
  4. Guarda (`POST /companies`). El sistema revisa, en orden: que el país exista; que el país no tenga ya una empresa; que el número no esté usado en el país; que el tipo exista, sea del país, esté activo, aplique a empresas y que el número tenga su longitud; que la moneda exista; que los textos respeten sus largos. La dirección se crea al final, cuando todo lo anterior pasó.
- **Errores visibles:** `COMPANY_COUNTRY_CONFLICT` y `COMPANY_TAX_ID_CONFLICT` (`409`); un tipo que no aplica o una longitud equivocada (`422`); país, tipo o moneda inexistentes (`404`); los de la dirección, que vienen de `location` (`GEODIVISION_NOT_FOUND`, código postal faltante, división que no sirve).
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

- **Rol:** la API solo deja escribir a `SUPERADMIN`. La pantalla muestra los botones de alta y edición también a `ADMIN_EMPRESA`, que recibe `403` al guardar. La lista la leen `SUPERADMIN` y el `ADMIN_EMPRESA` de esa empresa.
- **Pantalla:** Configuración › Ubicaciones (`/organization/locations`).
- **Pasos:** elige la empresa, escribe código y nombre y elige el tipo. Si es `sede_propia`, elige la sede y no escribe dirección; si es otro tipo, escribe la dirección y no elige sede. El sistema revisa que la empresa exista, que el código no esté usado en ella, y según el tipo: que la sede exista, sea de la misma empresa y esté activa, o que llegue una dirección y ninguna sede.
- **Errores visibles:** `LOCATION_CODE_CONFLICT` (`409`); sede de otra empresa, sede inactiva, sede o dirección que faltan o sobran (`400`); sede inexistente (`404`); `LOCATION_ADDRESS_NOT_ALLOWED` al corregir o mudar la dirección de una `sede_propia` y `LOCATION_ADDRESS_MISSING` al corregir una que no existe (`422`).
- **Resultado:** la ubicación queda activa. Al editarla, la dirección que viaja en el formulario corrige la fila existente o crea la primera; la pantalla no pregunta si es una mudanza. La API tiene además `PUT` y `POST /locations/{id}/address` para corregir o mudar.

### F5 · Registrar un colaborador

- **Rol:** `SUPERADMIN` en cualquier empresa; `ADMIN_EMPRESA` solo en la empresa de su propio perfil de colaborador. Otra empresa responde `403`.
- **Pantalla:** Configuración › Colaboradores (`/organization/employees`). `SUPERADMIN` elige la empresa; `ADMIN_EMPRESA` no puede elegirla, y hoy la pantalla le abre la primera empresa activa de la lista, que puede no ser la suya (§6, código).
- **Pasos:** escribe nombres y apellidos; elige el tipo de documento entre los de personas del país de la empresa y escribe el número; elige la sede si tiene una fija; escribe puesto, teléfono, foto y fecha de ingreso. El sistema revisa: que la empresa exista; que el tipo de documento exista, sea del país, esté activo, aplique a personas y que el número tenga su longitud; que la sede exista, sea de la empresa y esté activa; que el documento no esté registrado en la empresa.
- **Errores visibles:** documento repetido, `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` (`409`); tipo o sede que no sirven (`422`); empresa, tipo o sede inexistentes (`404`). Si la lista de tipos de documento no carga, el formulario no lo avisa y el selector queda vacío.
- **Resultado:** el colaborador queda activo y sin cuenta, registrado a nombre de quien lo dio de alta. Editarlo después (`PUT /employees/{id}`) repite las mismas revisiones.

### F6 · Dar acceso a un colaborador

Cruza `organization` e `iam`. El diccionario fija la operación (D-38) y el mapa propone que viva en `iam` (mapa §6.2); el código la puso en `organization`, que llama a `iam` por su fachada.

- **Rol:** `SUPERADMIN`, o el `ADMIN_EMPRESA` de la empresa del colaborador siempre que el colaborador no tenga el rol `SUPERADMIN`. `ADMIN_EMPRESA` no puede otorgar `SUPERADMIN` ni `ADMIN_EMPRESA`.
- **Pantalla:** Configuración › Colaboradores, acción "Dar acceso" de la fila. El diálogo pide el correo y uno o más roles de una lista fija de seis; a `ADMIN_EMPRESA` no le muestra los dos roles de administración. Si el colaborador ya tiene cuenta, el correo aparece bloqueado y el diálogo sirve para cambiar sus roles.
- **Pasos** (`POST /employees/{id}/access`):
  1. El sistema revisa que el colaborador exista y esté activo.
  2. Para cada rol, si trae sede o marca, revisa que existan y sean de la empresa del colaborador. La pantalla manda siempre la sede del colaborador y nunca una marca.
  3. **Si el colaborador ya tiene cuenta**, los roles enviados reemplazan todos los roles vigentes de esa cuenta: los que no vienen se retiran y los nuevos se otorgan. El correo enviado se ignora.
  4. **Si no tiene cuenta**, `iam` busca una por el correo, sin espacios y en minúsculas. Si existe, la reutiliza, le reemplaza los roles igual que en el paso 3 y no genera contraseña. Si no existe, crea la cuenta con una contraseña provisional generada por el sistema, marcada para cambiarse en el primer ingreso, y registra como autor a quien dio el acceso.
  5. Vincula el colaborador a la cuenta en la misma transacción.
- **La contraseña provisional se muestra una sola vez.** Solo viaja en la respuesta que creó la cuenta; el diálogo la muestra con un botón para copiarla y avisa que debe cambiarse al entrar. No se guarda legible en ningún lado y no se puede volver a consultar: si se pierde, se restablece (F8). Cuando la cuenta se reutilizó, no hay contraseña y el diálogo se cierra sin decir que la cuenta ya existía.
- **Errores visibles:** colaborador cesado (`422`); sede o marca de otra empresa (`422`) o inexistentes (`404`); un código de rol que no existe (`404`); sin permiso (`403`).
- **Resultado:** el colaborador tiene cuenta y la cuenta tiene exactamente los roles enviados, todos sin empresa, sede ni marca, porque `user_roles` todavía no tiene esas columnas (diccionario §5.13).

### F7 · Cesar y reingresar

- **Rol:** `SUPERADMIN`, o el `ADMIN_EMPRESA` de la empresa si el colaborador no es `SUPERADMIN`.
- **Pantalla:** Configuración › Colaboradores, acciones "Registrar salida" y "Reingresar".
- **Salida** (`POST /employees/{id}/termination`): fecha de salida y un motivo opcional de hasta 500 caracteres. El sistema revisa que el colaborador esté activo, que la fecha no sea anterior al ingreso y que, si tiene cuenta, no sea el último superadministrador activo. Registra la salida y, si tiene cuenta, retira en la misma transacción todos sus roles vigentes con motivo `baja_colaborador`. No bloquea la cuenta ni cierra sus sesiones; la sesión queda sin roles y no puede operar.
- **Reingreso** (`POST /employees/{id}/reactivation`): el diálogo pide la fecha de ingreso y el puesto, pero la API solo recibe la fecha: el puesto se descarta sin aviso. El sistema revisa que el colaborador esté cesado y que la nueva fecha no sea anterior a la salida. Sobrescribe la fecha de ingreso y borra la salida y su motivo. No devuelve los roles: hay que dar acceso de nuevo.
- **Errores visibles:** ya cesado o ya activo, fecha fuera de orden, último superadministrador (`422`).

### F8 · Restablecer la contraseña de un colaborador

- **Rol:** el mismo que F7.
- **Pantalla:** ninguna. La acción existía en Colaboradores y se quitó; hoy el endpoint no lo usa ninguna pantalla. Seguridad › Usuarios tiene un diálogo que llama a `/users/{id}/password-reset`, una ruta que el backend no tiene.
- **Pasos** (`POST /employees/{id}/password-reset`): el sistema revisa que el colaborador esté activo y tenga cuenta, genera una contraseña provisional nueva, obliga a cambiarla y la devuelve una sola vez. No cierra las sesiones abiertas de esa cuenta.
- **Errores visibles:** colaborador cesado o sin cuenta (`422`).

### F9 · Suspender a un colaborador

Solo existe en el frontend. La pantalla de Colaboradores ofrece "Suspender", "Cerrar suspensión" e "Historial de suspensiones", que llaman a `/employees/{id}/suspensions`. El backend no tiene esas rutas, así que las tres acciones fallan. La marca de colaborador suspendido nunca aparece, porque el backend no envía ese dato. En el diccionario el bloqueo es de la cuenta, no del colaborador (`user_access_suspensions`, §5.4), y su ruta propuesta es `/users/{id}/suspensions` en `iam` (convenciones §2.2).

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
| ORG-R15 | La sede de una ubicación propia es de la misma empresa y está activa | §6.5 | Servicio (`LocationFacilityPolicy`). Base: sin llave compuesta | `LocationCommandServiceImplTests.shouldRejectOwnedLocationFromDifferentCompanyBranch`; la sede inactiva, sin prueba | ⚠️ |
| ORG-R16 | La dirección de una ubicación es solo suya y está en el país de la empresa | §4.6; mapa §6.3 | Servicio: la crea en el país de la empresa. Base: falta `UNIQUE (address_id)` y la llave compuesta (V21) | Sin prueba | ⚠️ |
| ORG-R17 | El tipo de documento del colaborador es del país de la empresa, está activo, aplica a personas, y el número tiene su longitud | §5.3, §4.7 | Servicio (`TaxIdTypeRules.rejectionForIndividual`). Base: llave simple, sin el país | `EmployeeCommandServiceImplTests.shouldFailWhenDocumentTypeDoesNotApplyToIndividuals`; el país y la longitud, sin prueba | ⚠️ |
| ORG-R18 | El documento de identidad no se repite en la empresa | §5.3 | Base: `UNIQUE (company_id, document_type_id, document_number)` (V22), que deja pasar el mismo número con otro tipo. Servicio: `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT` | `EmployeeCommandServiceImplTests.shouldFailWhenDocumentAlreadyExistsInCompany`; `EmployeesIntegrationTests` | ⚠️ |
| ORG-R19 | La sede del colaborador es de su empresa y está activa | §5.3 | Servicio (`EmployeeAffiliationPolicy`). Base: sin llave compuesta (V22) | `EmployeeCommandServiceImplTests.shouldFailWhenBranchBelongsToDifferentCompany` | ⚠️ |
| ORG-R20 | Una cuenta se vincula a lo sumo a un colaborador por empresa | §5.3 | En ningún lado: no hay `UNIQUE (company_id, user_id)` ni revisión | Sin prueba | ❌ |
| ORG-R21 | Activo es sin fecha de salida; la salida no es anterior al ingreso y no se registra dos veces | §5.3 | Dominio y servicio. Base: sin `CHECK` | `EmployeeTests.terminationDateBeforeHireDateThrows` y `terminatingAlreadyTerminatedThrows`; `EmployeesIntegrationTests` | ✅ |
| ORG-R22 | Al registrar la salida se retiran los roles de la cuenta **en esa empresa**, con motivo `baja_colaborador` | §5.3, D-38, D-41 | Servicio, en la misma transacción. Retira los roles de todas las empresas, porque `user_roles` no tiene empresa | `EmployeeCommandServiceImplTests.shouldTerminateEmployeeSuccessfully` | ⚠️ |
| ORG-R23 | Si a la cuenta no le queda colaborador activo, se bloquea con motivo `baja` | §5.3, D-38, D-12 | En ningún lado: no existe `user_access_suspensions` | Sin prueba | ❌ |
| ORG-R24 | No se registra la salida del último superadministrador activo | §5.9 | Servicio, preguntando a `iam` | `EmployeeCommandServiceImplTests.shouldFailTerminationWhenLastActiveSuperadmin` | ✅ |
| ORG-R25 | Se da acceso solo a un colaborador activo, y la cuenta queda vinculada en la misma operación; si el correo ya tiene cuenta, se vincula a ella | D-38, §5.3 | Servicio | `EmployeeCommandServiceImplTests.shouldGrantAccessSuccessfully` y `shouldFailGrantingAccessWhenEmployeeTerminated` | ✅ |
| ORG-R26 | La sede y la marca de una asignación son de la empresa del colaborador | §5.7 | Servicio: las valida y después las descarta; la asignación se guarda sin sede ni marca | `EmployeeCommandServiceImplTests.shouldFailGrantingAccessWhenCommercialBrandMismatch` | ⚠️ |
| ORG-R27 | El primer superadministrador no da acceso ni otorga roles hasta vincularse a su perfil | §5.2, §5.9, D-38 | En ningún lado | Sin prueba | ❌ |
| ORG-R28 | `ADMIN_EMPRESA` no otorga `SUPERADMIN` ni `ADMIN_EMPRESA` | §5.5 (reglas 1 y 2); procesos P2 | Servicio: seguridad del endpoint (`canGrantRoles`). Frontend: oculta esos roles. Se salta con un código con espacios, `" SUPERADMIN"` | `OrganizationSecurityServiceTests.adminEmpresaCannotGrantForbiddenRoles`, sin el caso con espacios | ⚠️ |
| ORG-R29 | `ADMIN_EMPRESA` solo actúa sobre los colaboradores de su empresa y no modifica a un `SUPERADMIN` | §2.5; procesos §1 | Servicio: seguridad del endpoint. Se salta dando acceso con el correo de un superadministrador (§6, crítico). Responde `403`, no `404` | `OrganizationSecurityServiceTests` (los casos de acceso y modificación) | ⚠️ |
| ORG-R30 | Solo `SUPERADMIN` escribe empresas, marcas, sedes y ubicaciones | §5.2 (pasos 4 y 5); procesos §1 y §3.2 | Servicio: `@PreAuthorize`. Frontend: Ubicaciones deja escribir a `ADMIN_EMPRESA` | `CompaniesControllerTests.creationWithoutSuperadminReturns403`; `BranchesIntegrationTests.changingTheAddressOrTheStatusWithoutSuperadminReturns403`; marcas y ubicaciones, sin prueba | ⚠️ |
| ORG-R31 | La lectura de recursos de empresa se filtra por las asignaciones de la cuenta, y una empresa ajena responde `404` | §2.5, §5.8; convenciones §9.2 | Solo en colaboradores y en la lista de ubicaciones de una empresa, con `403`. Empresas, sedes, marcas y `GET /locations/{id}` los lee cualquier cuenta con un rol | Sin prueba | ❌ |
| ORG-R32 | Un país opera si tiene una empresa activa; no se guarda | D-3 | Servicio: `GET /companies?countryId=…&active=true`. La fachada (`existsCompanyByCountryId`) no mira si la empresa está activa | `CompanyQueryServiceImplTests.searchingPassesTheTextTheStatusTheInternalCountryAndThePageToTheRepository` | ⚠️ |
| ORG-R33 | Nada se borra: se desactiva o se registra la salida | §5.3, §6.2, §6.4, §6.5; convenciones §3.1 | Servicio: no hay `DELETE` | `405` en `BranchesIntegrationTests`, `CommercialBrandsIntegrationTests` y `EmployeesIntegrationTests`; empresas y ubicaciones, sin prueba | ⚠️ |
| ORG-R34 | Columnas de registro: la empresa guarda quién la creó y quién la cambió; el colaborador, quién lo registró; marcas, sedes y ubicaciones, solo fechas | D-42; §5.3, §6.2 a §6.5 | Base y servicio: clases base compartidas y el auditor de `iam` (`Revised` en empresas, `Authored` en colaboradores, `Auditable` en el resto) | `EmployeeAuditTests` (autor); `CompanyAuditTests`, `BranchAuditTests`, `CommercialBrandAuditTests`, `LocationAuditTests` (fechas); el autor de la empresa, sin prueba | ⚠️ |

## 5. Relación con otros contextos

**Lo que ofrece: `OrganizationContextFacade`.** Resumen de empresa por identificador público o por país, si un país tiene empresa, resumen de marca, sede, ubicación y colaborador, si una marca, sede o ubicación es de una empresa, y el identificador público de una empresa. Está construida y **ningún contexto la consume todavía**. Ofrece más que lo que lista el mapa §5.1; lo que el mapa propone para el paso 3 (colaborador activo en una empresa, cuenta con algún colaborador activo, registrar el colaborador del registro público, zona de la sede) no está.

**Lo que usa de `location`** (`ExternalLocationService`): el identificador interno de un país y, en lote, sus nombres e identificadores públicos; el identificador de una moneda y, en lote, sus códigos ISO; las reglas de un tipo de documento (`TaxIdTypeRules`) y, en lote, sus códigos, nombres e identificadores públicos; crear, corregir y reemplazar una dirección en el país que fija el dueño; y leer direcciones en lote.

**Lo que usa de `iam`** (`ExternalIamService`): la cuenta de la petición actual; el identificador público, el correo y, en lote, el resumen de una cuenta; los roles vigentes de una cuenta, uno a uno y en lote; crear o reutilizar la cuenta de un colaborador con sus roles; reemplazar los roles de una cuenta; retirar todos sus roles con un motivo; restablecer su contraseña; y si es el último superadministrador activo.

**El sentido de la dependencia está al revés del mapa.** `organization` declara `allowedDependencies = {"location::acl", "iam::acl"}` y `iam` no depende de nadie. El mapa propone lo contrario: `iam` llama a `organization` y `organization` no llama a `iam` (mapa §4, §6.2, §10). Dar acceso, la salida y el restablecimiento viven hoy en `organization`. Cuál de los dos sentidos queda es la decisión pendiente 4 del mapa, y dónde vive la salida es la pendiente 2 (§11).

**Frontend.** `OrganizationFacade` devuelve vistas mínimas de ubicaciones y colaboradores (`LocationOption`, `EmployeeOption`) y ningún otro contexto la usa; ya no ofrece `CompanyOption`, que figura en el mapa §9. El contexto usa `LocationFacade` para países, monedas y tipos de documento; `mm-address-fields` en los formularios de empresa, sede y ubicación; e `IamFacade` para los roles de la sesión y `currentCompanyId`. Ese último siempre vale `null`, porque `/authentication/me` no devuelve la empresa de la cuenta.

## 6. Estado y pendientes para cerrar

**Construido.** Los cuatro agregados del paso 2 y los colaboradores del paso 3. Del paso 2 falta `roles.company_id` (diccionario §3).

| Agregado | Backend | Frontend | Pruebas de extremo a extremo |
|---|---|---|---|
| Empresa | API, dirección fiscal anidada, migraciones V12, V15 y V18 | Configuración › Empresas, solo `SUPERADMIN` | Flujo y captura |
| Marca comercial | API, migraciones V16 y V20 | Configuración › Marcas comerciales | Flujo y captura |
| Sede | API, dirección anidada, migraciones V17 y V20 | Configuración › Sedes | Flujo y captura |
| Ubicación | API, dirección anidada, migración V21 | Configuración › Ubicaciones | Ninguna |
| Colaborador | API con dar acceso, salida, reingreso, restablecimiento e indicadores; migraciones V22 y V23 | Configuración › Colaboradores, con dar acceso, salida, reingreso y suspensiones | Solo captura y accesibilidad; las dos fallan |

Cada agregado tiene pruebas de dominio, de servicio, de persistencia y de integración en el backend, y pruebas unitarias en el frontend.

**La rama.** `feature/location` mezcla trabajo de `location`, `organization` e `iam`. Antes de integrarla a `develop` hay que revisar este contexto y resolver al menos los pendientes críticos y altos.

### a) Decisiones

| Qué | Quién |
|---|---|
| Dónde vive la salida del colaborador y cómo se protege al último superadministrador (mapa §11, pendiente 2) | Responsable del diccionario |
| Sentido de la dependencia entre `iam` y `organization` (mapa §11, pendiente 4) | Responsable del diccionario |
| Si existe el reingreso y cómo conserva el período anterior (pregunta 3) | Responsable del diccionario |
| Si `termination_reason` entra al diccionario (V23 lo agregó sin respaldo) | Responsable del diccionario |
| Ruta de dar acceso: el código usa `POST /employees/{id}/access` con los roles; convenciones §14.15 propone `POST /access-grants`, sin roles | Responsable del diccionario |

### b) Código

**Crítico**

- **Toma de una cuenta de superadministrador.** Dar acceso reutiliza cualquier cuenta que ya tenga ese correo y le reemplaza los roles. Un `ADMIN_EMPRESA` puede dar acceso a un colaborador sin cuenta de su empresa con el correo de un superadministrador y el rol `OPERACIONES`: la cuenta pierde `SUPERADMIN`, aunque sea el último, y queda vinculada a ese colaborador. Como ya no es superadministrador, el mismo `ADMIN_EMPRESA` le restablece la contraseña (`/employees/{id}/password-reset`) y recibe una provisional. Además, un código de rol con espacios (`" SUPERADMIN"`) pasa `canGrantRoles` y se otorga.

**Alto**

- `user_roles` no tiene empresa (diccionario §5.13). Por eso la salida retira los roles de la cuenta en todas las empresas; dar acceso en una segunda empresa borra los roles de la primera; y la sede y la marca de cada rol se validan y se descartan.
- La salida no bloquea la cuenta cuando no le queda colaborador activo (ORG-R23).
- `findByUserId` espera un solo colaborador por cuenta. Con una cuenta vinculada a colaboradores de dos empresas, que el diccionario permite, la seguridad de colaboradores falla con error interno o niega el acceso.
- Falta el filtro por empresa en la lectura de empresas, sedes, marcas y ubicaciones, incluido `GET /locations/{id}`, y la prueba de que una empresa ajena responde `404` (ORG-R31; orden de implementación, pendientes).
- El frontend nunca recibe la empresa de la cuenta: Colaboradores abre a `ADMIN_EMPRESA` la primera empresa activa, que puede no ser la suya.
- Las suspensiones del frontend llaman a rutas que no existen y la marca de suspendido nunca llega (F9).

**Medio**

- `ADMIN_EMPRESA` puede modificar, cesar y restablecer la contraseña de otro `ADMIN_EMPRESA` de su empresa. El diccionario no lo fija.
- Ubicaciones muestra a `ADMIN_EMPRESA` acciones que la API rechaza con `403`.
- El reingreso borra el período anterior y el diálogo descarta el puesto sin aviso.
- Los roles que se quitan al reemplazar los de una cuenta se retiran con motivo `ROLE_SCOPE_CHANGE`, que no es uno de los cuatro de D-41; la base no lo impide.
- V21 y V22 quedaron más débiles que V18 y V20: `locations.address_id` sin `UNIQUE` ni llave compuesta de país, y sin llave que obligue a que `locations.branch_id` y `employees.branch_id` sean de la misma empresa (ORG-R15, ORG-R16, ORG-R19).
- No se aplican ORG-R20 (una cuenta por empresa) ni ORG-R27 (el primer superadministrador sin perfil no da acceso).
- Desactivar una empresa o una sede no revisa lo que depende de ella, y se pueden crear sedes, marcas, ubicaciones y colaboradores bajo una empresa inactiva.
- El formulario de colaborador no avisa cuando no cargan los tipos de documento.
- El frontend quitó las tarjetas de indicadores y el restablecimiento de contraseña de Colaboradores: `/companies/{id}/employees/metrics` y `/employees/{id}/password-reset` no los usa ninguna pantalla.
- Pruebas de extremo a extremo, corridas el 14/10/2026: Colaboradores falla en accesibilidad (WCAG A/AA, en escritorio y tableta) y en la captura del teléfono, que es del 08/10/2026, anterior a tres cambios de la pantalla. Ubicaciones no tiene prueba ni simulación de su API, y faltan las simulaciones de suspensiones.

**Bajo**

- El restablecimiento de contraseña no cierra las sesiones abiertas de la cuenta (D-33 las cierra al cambiar la contraseña).
- Una fecha de salida futura deja al colaborador inactivo y sin roles desde el momento en que se registra.
- `PUT /locations/{id}` corrige la dirección, a diferencia de empresas y sedes (ORG-R06).
- La fachada responde que un país tiene empresa aunque esté inactiva (ORG-R32).
- No se verifica el dígito de control (ORG-R04).

### c) Datos

- Razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales y colaboradores reales (diccionario §14).
- La matriz de permisos por rol, que decide qué hace cada rol más allá de `SUPERADMIN` y `ADMIN_EMPRESA` (procesos, pregunta 5).

### d) Documentación

- El orden de implementación (paso 2 y paso 3), procesos y pantallas (§3.2) y el diccionario (§3, columna *Código*) dicen que el paso 2 tiene solo empresas, que Sedes y Marcas no tienen ruta, que Ubicaciones falta y que Colaboradores está pendiente. Los cinco están construidos.
- El README del backend dice que `organization` tiene solo empresas.
- El mapa §5.1 y §9 describen una fachada y un `OrganizationFacade` distintos de los actuales (§5).
- El orden de implementación cita "la migración V17 de permisos"; los permisos de la cuenta de la aplicación están en V19 y V17 crea las sedes.
- Convenciones §2.2 no lista `/employees/{id}/access`, `/reactivation`, `/password-reset` ni `/companies/{id}/employees/metrics`.

## 7. Preguntas abiertas

1. **¿Dónde se registra la salida y cómo se protege al último superadministrador?** Hoy vive en `organization` con una llamada directa a `iam`. [Mapa de contextos](../mapa-de-contextos-millennium-mobility.md), §11, pendiente 2, y convenciones §14.17.
2. **¿En qué sentido va la dependencia entre `iam` y `organization`?** [Mapa de contextos](../mapa-de-contextos-millennium-mobility.md), §11, pendiente 4.
3. **¿Existe el reingreso?** Si existe, ¿conserva el período anterior en otra fila o en un historial, dado que el documento no puede repetirse en la empresa? Pregunta nueva, a registrar en el [diccionario](../diccionario-de-dtos-millennium-mobility.md) §5.3.
4. **¿Qué permisos tiene cada rol?** Decide quién ve y escribe cada pantalla de Configuración. [Procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), apartado 5, pregunta 5, y diccionario §14 (dato pendiente).
5. **¿Cómo se llama y qué responde dar acceso?** Nombre de la ruta, `201` o `200` según cree o vincule la cuenta, y qué pasa con la contraseña cuando la cuenta ya existe. [Convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), §14.15.
6. **¿Se puede corregir el país, el tipo o el número de documento de una empresa mal registrada?** El código no lo permite y el diccionario no lo dice. Pregunta nueva, a registrar en el [diccionario](../diccionario-de-dtos-millennium-mobility.md) §6.2.
7. **¿Cómo lista el registro público, sin sesión, las sedes y los tipos de documento?** [Convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), §14.16.
