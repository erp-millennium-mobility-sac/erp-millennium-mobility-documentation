# Contexto `organization` — ERP Millennium Mobility

**Estado:** vigente, validado por el responsable · **Fuente:** diccionario de datos (§2.2, §2.5, §2.6, §3, §4.6, §4.7, §5.2, §5.3, §5.4, §5.6 a §5.10, §5.13, §6, §14 y las decisiones que cita cada regla), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), [iam.md](iam.md) y el código. Si este documento y el diccionario no coinciden, manda el diccionario.

## 1. Qué es y qué no es

`organization` responde quién es la empresa, con qué marcas vende, en qué locales trabaja su gente, dónde puede haber vehículos, desde qué cuentas paga y quiénes son sus colaboradores. Casi todo lo demás del sistema cuelga de estas tablas: un vehículo es de una empresa y está en una ubicación; una factura la recibe una empresa; un pago sale de una de sus cuentas; un rol se otorga sobre una empresa o una sede.

**Tablas que posee** (diccionario §3; el reparto del bloque 3b lo fija D-92)

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `companies` | La razón social constituida en un país: compra, factura y es dueña de los vehículos | §6.2 |
| `commercial_brands` | Los nombres bajo los que la empresa vende: Lizzo, Lima Bikes | §6.3 |
| `company_bank_accounts` | Las cuentas desde las que la empresa paga a sus proveedores; no es tesorería: no guarda saldos ni movimientos (D-214) | §6.2.1 |
| `branches` | Los locales donde trabaja personal y desde donde se emiten comprobantes | §6.4 |
| `locations` | Los lugares donde puede haber vehículos, propios o de terceros | §6.5 |
| `employees` | Las personas que trabajan para la empresa, usen o no el sistema, con su período vigente | §5.3 |
| `employee_tenure_periods` | Los períodos que un colaborador ya cerró en su empresa: el ingreso y la salida que cada regreso deja atrás (D-241) | §5.3 |
| `advisor_assignments` | Qué asesora atiende a qué cliente. Diseñada, se construye con ventas (diccionario §3, ajuste 2) | §5.11 |

El colaborador vive aquí hasta que lleguen los contratos laborales, la planilla o la asistencia; entonces pasa a un contexto propio (D-92).

**Lo que parece suyo y no lo es**

- **Las cuentas y los roles son de `iam`.** El colaborador es de `organization`; la cuenta con que entra, sus roles y sus bloqueos son de `iam` (diccionario §5.1, D-92). El vínculo entre los dos lo guarda `organization`, y lo escribe solo cuando `iam` se lo pide por la fachada. Dar acceso, desvincular, el enlace para restablecer la contraseña, el código de activación, los bloqueos y lo que la salida y el regreso le hacen a la cuenta los orquesta `iam` ([iam.md](iam.md)).
- **El registro de los vínculos confirmados y de los desvínculos** (§5.4) es de `iam`, aunque nombra un colaborador y su empresa (D-82).
- **El alcance de quien llama tampoco es suyo.** `organization` no depende de `iam`: lee qué empresas alcanza la cuenta de la petición, y con qué permiso actúa en cada una, de un puerto de `shared` que implementa `iam` (mapa §5; sección 5). Con eso decide cada lectura y cada escritura (ORG-R29 a ORG-R31, ORG-R57).
- **Las direcciones son de `location`**, pero cada dueño tiene la suya (diccionario §4.6). La empresa, la sede y la ubicación piden a `location` crear, corregir o reemplazar su dirección, y nunca la comparten (mapa §6.3, convenciones §3.3).
- **"Ubicación" aquí es un lugar físico donde hay vehículos**, no geografía. Los países, las divisiones territoriales y las direcciones son de `location` (D-6, diccionario §6.5).
- **Los tipos de documento tributario son de `location`** (D-44). La empresa y el colaborador solo eligen uno.
- **Los pagos que salen de una cuenta bancaria son de `procurement`.** `organization` guarda la cuenta y sabe si un registro de otro contexto la nombra —hoy, un pago— por un puerto suyo que implementa `procurement` (sección 5).

**Vocabulario**

| Término | Qué es | Qué no es |
|---|---|---|
| Empresa | Una razón social en un país. Hay **una sola por país** (D-5) | Lizzo y Lima Bikes no son empresas |
| Marca comercial | Bajo qué nombre vendemos: Lizzo, Lima Bikes (§6.3) | La marca del vehículo (Chevrolet, TVS), que es de `vehiclecatalog` (§7.2) |
| Sede | Un local de la empresa con personal; se le asignan roles y emite comprobantes (§6.4) | Un patio alquilado o la tienda de un dealer |
| Ubicación | Un lugar donde puede estar un vehículo: la sede propia vista como patio, un patio arrendado, un dealer, un taller (§6.5, D-6) | Un local al que se asignan personas |
| Cuenta bancaria | Una cuenta de la empresa desde la que paga a sus proveedores (§6.2.1, D-214) | Tesorería: no lleva saldos ni movimientos |
| Puesto | El trabajo del colaborador: asesor, mecánico. Informativo (D-39) | Un rol: no da permisos |
| Rol | Conjunto de permisos de una cuenta, con su alcance. Es de `iam` (D-39) | Un puesto |
| Perfil | Los datos personales del colaborador (D-4) | La cuenta ni el rol |
| Titular de una cuenta | Cada colaborador, de cualquier empresa, que ya tiene vinculada la cuenta de un correo (D-82) | La persona que recibe el acceso ahora |
| Desvincular | Deshacer el vínculo equivocado entre un colaborador y una cuenta. Funciona como una salida para la cuenta, no para el colaborador (D-82) | La salida, que registra que la persona se fue |

## 2. Ciclo de vida de cada agregado

Nada de este contexto se borra (ORG-R33): las empresas, marcas, sedes, ubicaciones y cuentas bancarias se desactivan; los colaboradores reciben una fecha de salida. A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo, y lo que ya apuntaba a ella se sigue modificando mientras ese vínculo no cambie (ORG-R45), con el mismo criterio que una moneda o un tipo de documento desactivados. Reactivar algo es volver a colgarlo (ORG-R58): primero se reactiva la empresa. Desactivar no espera a nada.

### Empresa

- **Quién la crea:** quien tiene `configuracion.crear_empresas` en todas las empresas; con la semilla, el superadministrador (§5.2, paso 4; ORG-R30). Nace activa, con su país, su tipo y número de documento tributario, su razón social y nombre comercial, su teléfono, su moneda funcional y, si la tiene, su dirección fiscal (ORG-R54).
- **Qué se cambia:** la razón social, el nombre comercial, el teléfono y la moneda funcional, que puede ser de otro país (§6.2; ORG-R35). El documento tributario se corrige en la misma fila, por su propio camino, si se registró equivocado (ORG-R46).
- **Qué queda fijo:** el país. Una empresa constituida en otro país es otra empresa, con sus sedes, su moneda y sus documentos (D-5, D-85).
- **La dirección fiscal** se corrige o se muda por su propio camino (ORG-R05, ORG-R06). Una empresa sin dirección registra la primera como una mudanza.
- **Cómo se retira:** se desactiva y se puede reactivar. Lo que ya tiene sigue válido y se sigue corrigiendo; mientras siga inactiva no se le cuelga nada nuevo (ORG-R45) ni se reactiva nada de lo suyo (ORG-R58). Quien la lee la sigue eligiendo en las listas, para consultar y corregir.
- **Qué deduce:** un país opera si tiene una empresa activa (ORG-R32).

### Marca comercial

- La crea quien tiene `configuracion.crear_marcas_comerciales` en toda la empresa (ORG-R30), bajo una empresa activa, con un código corto y un nombre.
- Se cambia solo el nombre; el código queda fijo desde el alta (ORG-R12).
- Se desactiva y se reactiva (ORG-R58). Una marca inactiva no recibe roles nuevos; los que ya la tienen siguen vigentes (ORG-R45).

### Sede

- La crea quien tiene `configuracion.crear_sedes` en toda la empresa (§5.2, paso 5; ORG-R30), bajo una empresa activa, con código (ORG-R08, ORG-R47), nombre, código de establecimiento tributario opcional, zona horaria obligatoria (ORG-R10) y dirección opcional. Hereda el país de su empresa (ORG-R07). No tiene tipo (ORG-R11).
- Se cambian el nombre, el código de establecimiento y la zona; el código queda fijo (ORG-R08). La zona se valida al guardarla; una sede ya guardada no la vuelve a validar. La dirección se corrige o se muda (ORG-R06).
- Se desactiva y se reactiva (ORG-R58). Una sede inactiva no recibe colaboradores, ubicaciones propias ni roles nuevos, y lo que ya la tiene la conserva (ORG-R45). Su ubicación propia se reactiva solo después de ella (ORG-R51).

### Ubicación

- La crea quien tiene `configuracion.crear_ubicaciones` en toda la empresa (ORG-R30), bajo una empresa activa, con código (ORG-R13, ORG-R47), nombre y tipo: `sede_propia`, `patio_arrendado`, `tienda_dealer`, `taller_tercero` u `otro` (§6.5; ORG-R55).
- Una `sede_propia` apunta a una sede activa de la misma empresa y su dirección es la de la sede; las demás tienen dirección propia (ORG-R14 a ORG-R16). Una sede tiene a lo sumo una ubicación propia (ORG-R51).
- Se cambian el nombre, el tipo dentro de su grupo (ORG-R40) y la sede, que si es nueva tiene que estar activa y no tener ya la suya (ORG-R15, ORG-R51). El código queda fijo. La dirección de una ubicación de terceros se corrige o se muda por su propio camino (ORG-R06).
- Se desactiva al terminar el alquiler; el historial se conserva (§6.5). La propia de una sede se reactiva en vez de crear otra (ORG-R51), y ninguna se reactiva bajo una empresa inactiva (ORG-R58).

### Cuenta bancaria

- **Quién la crea:** quien tiene `configuracion.crear_cuentas_bancarias` en toda la empresa; con la semilla, `SUPERADMIN` y `ADMIN_EMPRESA` (ORG-R63). Nace activa bajo una empresa activa, con su nombre corto («BBVA Dólares»), su banco, su moneda y su número (ORG-R64). Una empresa tiene varias, también varias por moneda.
- **Qué se cambia:** el nombre y el banco siempre; el número y la moneda, solo mientras ningún pago salió de ella (ORG-R65). Un pago anulado también salió de su cuenta y conserva el número y la moneda con que se registró.
- **Qué queda fijo:** su empresa (ORG-R63).
- **Cómo se retira:** se desactiva y se reactiva, solo bajo una empresa activa (ORG-R58). Desactivarla no toca los pagos que salieron de ella. Nunca se borra (ORG-R63).
- **Quién la lee:** quien tiene `configuracion.ver_estructura` donde actúa (ORG-R31); el número completo, solo quien gestiona las cuentas de esa empresa (ORG-R66).

### Colaborador

- **Quién lo registra y lo modifica:** ORG-R29. El administrador de accesos lo lee y no lo registra: quien da acceso no crea al colaborador que lo recibe. Se registra en una empresa activa y, si tiene sede, en una sede activa (ORG-R45, ORG-R19). Nace activo y sin cuenta.
- **Quién lo lee:** la fila mínima, cualquier rol con `configuracion.ver_colaboradores`; sus datos personales, solo con `configuracion.ver_datos_personales_colaboradores` (ORG-R57). Un rol de sede lee los colaboradores de toda su empresa (D-14).
- **Qué se cambia:** los nombres, el documento (ORG-R17, ORG-R18, ORG-R52), la sede (ORG-R19), el puesto, el teléfono, la foto (ORG-R53) y la fecha de ingreso (ORG-R49, ORG-R21, ORG-R61). La empresa queda fija. También se corrige a un colaborador que ya salió, sin que regrese (ORG-R21): por ejemplo, el documento que lo identifica si vuelve o si lo contrata otra empresa (D-82).
- **Activo** significa sin fecha de salida (ORG-R21). Sus fechas se leen en su zona (ORG-R38).
- **Dar acceso y desvincular** los hace `iam` (F6, F11): el colaborador recibe una cuenta nueva o la que ya tiene la misma persona (ORG-R25, ORG-R41, ORG-R42), una por empresa (ORG-R20), y desvincular deja al colaborador sin cuenta (ORG-R43).
- **Salida:** se registra solo la fecha, sin motivo: `termination_reason` es un candidato del diccionario (§5.10, pregunta 8). Sigue ORG-R21, y en la misma transacción `iam` retira los roles de la cuenta en esa empresa y, si era su último colaborador activo, la cierra (ORG-R22 a ORG-R24; F7).
- **Regreso** (§5.3, D-60): una nueva fecha de ingreso (ORG-R37), solo a una empresa y una sede activas (ORG-R58). Reemplaza la fecha de ingreso, borra la de salida y guarda el período que cierra (ORG-R60). Si la salida había cerrado la cuenta, la persona vuelve a entrar con la misma; los roles no vuelven solos. Los regresos registrados antes de que existiera el historial no guardaron su período, y no se recupera.

### Período de un colaborador

- **Quién lo crea:** solo el regreso, con su autor y su momento (ORG-R60).
- **Qué guarda:** el colaborador y su empresa, el ingreso y la salida.
- **Qué se cambia:** nada; es de solo inserción (ORG-R60).
- **Quién lo lee:** quien lee la fila mínima del colaborador (ORG-R62; F13).
- **Nunca pasa:** se edita, se borra o se recupera uno anterior al historial.

## 3. Flujos funcionales

En cada flujo, "el sistema revisa" es lo que el backend comprueba, en ese orden; los códigos de error están en convenciones §7.2. Cada ruta exige un permiso del catálogo (§5.6, D-95), y el servicio decide dónde actúa con la asignación que lo trae (D-99): las escrituras de la estructura, en toda la empresa (ORG-R30, ORG-R63); las lecturas, donde actúa el permiso de leer (ORG-R31). Una empresa que la cuenta alcanza con un rol que no trae el permiso responde `403`; una fuera de su alcance, `404`. Las listas buscan sin distinguir mayúsculas ni tildes (D-142; convenciones §6.2).

### F1 · Dar de alta una empresa

- **Quién:** crea quien tiene `configuracion.crear_empresas` en todas las empresas; edita, corrige el documento y cambia la dirección fiscal quien tiene `configuracion.editar_empresas`, y activa o desactiva quien tiene `configuracion.desactivar_empresas`, en toda la empresa (D-130; ORG-R30). La semilla da los tres solo a `SUPERADMIN`. Lee quien tiene `configuracion.ver_estructura` (ORG-R31), que tienen los siete roles del sistema: las Empresas, la ficha de cada una y Ubicaciones se abren solo con él, y un rol que solo escribe la estructura no las abre (D-141).
- **Pantalla:** Configuración › Empresas y la ficha de cada empresa, que reúne sus sedes, sus marcas comerciales y sus cuentas bancarias. Quien no puede crear empresas y alcanza una sola entra directo a su ficha.
- **Pasos:**
  1. Elige el país; se le ofrecen los tipos de documento de ese país que aplican a empresas.
  2. Escribe el número, la razón social, el nombre comercial y el teléfono, y elige la moneda funcional entre las activas (ORG-R35).
  3. Llena la dirección fiscal, en el país de la empresa (ORG-R05).
  4. Guarda (`POST /companies`). El sistema revisa: que el país exista; el tipo y el número (ORG-R03, ORG-R02), con su dígito de control, que calcula `location` (ORG-R04; un tipo cuyo cálculo todavía no existe acepta el número); que el país no tenga ya una empresa (ORG-R01); que el número no esté usado en el país (ORG-R02); la moneda (ORG-R35); y los largos. El número se revisa antes que los duplicados, así un RUC mal escrito dice qué tiene mal y no que la empresa ya existe. La dirección se crea al final.
- **Si falla:** no queda ni la empresa ni la dirección, y cada regla responde su código, como `COMPANY_COUNTRY_CONFLICT`, `COMPANY_TAX_ID_CONFLICT` o `COMPANY_TAX_ID_CHECK_DIGIT_INVALID`; los de la dirección, en convenciones §3.3.
- **Después:** se editan los nombres, el teléfono y la moneda (`PUT /companies/{id}`); se corrige o se muda la dirección (ORG-R06; sin dirección que corregir, `COMPANY_FISCAL_ADDRESS_MISSING`); se corrige el documento (F10); y se desactiva o se reactiva.

### F2 · Marcas comerciales

- **Quién:** escriben `configuracion.crear_marcas_comerciales`, `editar_marcas_comerciales` y `desactivar_marcas_comerciales`, en toda la empresa (ORG-R30); lee `configuracion.ver_estructura`.
- **Pantalla:** la ficha de la empresa, en sus marcas comerciales.
- **Pasos:** escribe el código y el nombre. El sistema revisa que la empresa exista y esté activa (ORG-R45) y que el código no esté usado en ella (ORG-R12).
- **Si falla:** `COMMERCIALBRAND_CONFLICT` o `COMPANY_INACTIVE`.
- **Después:** se cambia el nombre; se desactiva o se reactiva (`PATCH /commercial-brands/{id}/status`), solo bajo una empresa activa (ORG-R58).

### F3 · Sedes con su dirección y su zona

- **Quién:** escriben `configuracion.crear_sedes`, `editar_sedes` —también la dirección— y `desactivar_sedes`, en toda la empresa (ORG-R30); lee `configuracion.ver_estructura`.
- **Pantalla:** la ficha de la empresa, en sus sedes.
- **Pasos:** escribe el código, el nombre y el código de establecimiento si lo tiene, elige la zona horaria y llena la dirección si la tiene. El sistema revisa que la empresa exista y esté activa (ORG-R45), que el código y el código de establecimiento no estén usados en ella (ORG-R08, ORG-R47) y la zona (ORG-R10); la dirección se crea al final, en el país de la empresa (ORG-R07).
- **Si falla:** `BRANCH_CODE_CONFLICT`, `BRANCH_TAX_ESTABLISHMENT_CODE_CONFLICT` o `COMPANY_INACTIVE`; al corregir una dirección que no existe, `BRANCH_ADDRESS_MISSING`.
- **Después:** se cambian el nombre, el código de establecimiento y la zona; la dirección se corrige o se muda (ORG-R06); se desactiva o se reactiva (`PATCH /branches/{id}/status`), solo bajo una empresa activa (ORG-R58).

### F4 · Ubicaciones

- **Quién:** escriben `configuracion.crear_ubicaciones`, `editar_ubicaciones` —también la dirección— y `desactivar_ubicaciones`, en toda la empresa (ORG-R30); lee `configuracion.ver_estructura` (ORG-R31).
- **Pantalla:** Configuración › Ubicaciones, con la empresa elegida.
- **Pasos:** escribe el código y el nombre y elige el tipo; si es `sede_propia`, elige la sede y no escribe dirección, y si no, escribe la dirección (ORG-R14). El sistema revisa que la empresa exista y esté activa (ORG-R45), que el código no esté usado en ella (ORG-R13), y según el tipo: que la sede sea de la misma empresa, esté activa (ORG-R15) y no tenga ya su ubicación propia (ORG-R51), o la dirección (ORG-R16).
- **Si falla:** `LOCATION_CODE_CONFLICT`, `LOCATION_BRANCH_CONFLICT`, `BRANCH_OTHER_COMPANY`, `BRANCH_INACTIVE` o `COMPANY_INACTIVE`; una sede o una dirección que falta o sobra, `400 VALIDATION_ERROR`.
- **Después:** se cambian el nombre, el tipo dentro de su grupo (ORG-R40, `LOCATION_OWNERSHIP_CHANGE_NOT_ALLOWED`) y la sede. La dirección se cambia aparte (`PUT` o `POST /locations/{id}/address`): la de una ubicación de terceros se corrige o se muda (ORG-R06), y una sin dirección registra la primera; la de una `sede_propia` no (`LOCATION_ADDRESS_NOT_ALLOWED`). Se reactiva (`PATCH /locations/{id}/status`) solo bajo una empresa activa y, si es propia, con su sede activa (ORG-R58, ORG-R51; `LOCATION_BRANCH_INACTIVE`). La lista se exporta (convenciones §8.1; ORG-R59).

### F5 · Registrar un colaborador

- **Quién:** registra quien tiene `seguridad.registrar_colaboradores` y modifica quien tiene `seguridad.modificar_colaboradores`, donde actúa (ORG-R29): con la semilla, `SUPERADMIN` en cualquier empresa, también sin estar vinculado (D-89), y `ADMIN_EMPRESA` en la suya. Los lee quien tiene `configuracion.ver_colaboradores`, y sus datos personales, quien tiene `configuracion.ver_datos_personales_colaboradores` (ORG-R57).
- **Pantalla:** Configuración › Colaboradores, con la empresa elegida. Se abre con `configuracion.ver_datos_personales_colaboradores`, en las empresas donde actúa ([procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md), 3.1): un rol de sede no la ve, aunque la API le sirva la fila mínima. Cada fila ofrece solo las acciones que el backend calcula con las reglas de cada operación (convenciones §4.2; D-122). Las de cuenta de `iam` (F6, F8, F9 y F11) exigen un colaborador activo; a uno que salió se le ofrece a lo sumo corregirlo (ORG-R21) y reincorporarlo. La salida, el regreso y las acciones de cuenta exigen además a quien opera vinculado a un colaborador activo (ORG-R50), nunca son sobre lo propio (D-70), y una cuenta protegida solo la maneja quien actúa sobre lo reservado (D-121). La cuenta de cada colaborador se muestra donde la sesión lee las cuentas (`seguridad.ver_cuentas`, D-122).
- **Pasos:** escribe los nombres, elige el tipo de documento entre los de personas del país de la empresa y escribe el número; elige la sede si tiene una fija; escribe el puesto, el teléfono, la foto y la fecha de ingreso. El sistema revisa, al registrarlo: que la empresa exista y esté en el alcance; que el permiso actúe donde va el colaborador (ORG-R29); que la empresa esté activa (ORG-R45); el tipo y el número (ORG-R17, ORG-R04); la sede (ORG-R19); la fecha de ingreso (ORG-R49); y que el documento no esté registrado en la empresa (ORG-R18). Al editarlo repite esas revisiones salvo la de la empresa activa, con el tipo que ya tenía conservado aunque se haya desactivado (LOC-R36), la longitud y el dígito de control medidos solo si cambian (ORG-R52) y la sede activa solo si es nueva (ORG-R19); en un colaborador que ya salió, ORG-R21, y en uno que ya regresó, ORG-R61.
- **Si falla:** cada regla responde su código, como `EMPLOYEE_DOCUMENT_NUMBER_CONFLICT`, `EMPLOYEE_DOCUMENT_NUMBER_CHECK_DIGIT_INVALID`, `EMPLOYEE_HIRE_DATE_IN_FUTURE`, `EMPLOYEE_TERMINATION_BEFORE_HIRE` o `EMPLOYEE_HIRE_DATE_BEFORE_PREVIOUS_TERMINATION`.
- **Resultado:** el colaborador queda activo y sin cuenta, registrado a nombre de quien lo dio de alta (ORG-R34). Se edita con `PUT /employees/{id}`. La lista se exporta solo con el permiso de los datos personales, porque lleva el documento de cada uno (ORG-R57, ORG-R59; convenciones §8.1).

### F6 · Dar acceso a un colaborador

Es una operación de `iam` ([iam.md](iam.md), F5): la hace quien tiene `seguridad.dar_acceso` con alcance sistema —con la semilla, el superadministrador y el administrador de accesos; el administrador de empresa no (D-57)—, con `POST /access-grants`. Crea la cuenta y le envía el código para activarla, o vincula la que la persona ya tiene; ninguna contraseña viaja de vuelta (D-255, D-258). Los roles se otorgan después ([iam.md](iam.md), F6).

Lo que pone `organization`, por su fachada:

- **El colaborador, bloqueado** hasta que termina la operación, así un acceso y una salida del mismo colaborador corren uno después del otro.
- **El vínculo con una cuenta nueva** (ORG-R25, ORG-R20). Si el vínculo falla, `iam` deshace la cuenta que acababa de crear.
- **El vínculo con una cuenta que ya existía**, solo si es de la misma persona (ORG-R41, ORG-R42). Compara la sigla del tipo y no el tipo, porque los tipos son de cada país: un pasaporte `PAS` con el mismo número en Perú y en Colombia es el mismo documento. Una cuenta cerrada por la salida de otra persona sigue cerrada.
- **Quién tiene la cuenta:** los titulares, que `iam` muestra solo a un superadministrador para que confirme que es la misma persona (ORG-R41).

### F7 · Salida y regreso

Las rutas las sirve `iam`, que decide quién puede y qué le pasa a la cuenta ([iam.md](iam.md), F9); `organization` registra el hecho y sus fechas, en la misma transacción.

- **Quién:** quien tiene `seguridad.registrar_salidas_y_regresos` donde trabaja el colaborador, con las mismas formas que las escrituras de F5 (D-117, D-122); con la semilla, `SUPERADMIN` o el `ADMIN_EMPRESA` de su empresa. Tiene que estar vinculado a un colaborador activo (ORG-R50) y nunca registra su propia salida ni su propio regreso (`OWN_ACCOUNT_FORBIDDEN`, D-70). El administrador de accesos no registra salidas ni regresos (D-57).
- **Pantalla:** Configuración › Colaboradores, en las filas que ofrecen la baja o la reincorporación (F5). Los dos diálogos calculan el día de hoy en la zona del colaborador (ORG-R38).
- **Salida** (`POST /employees/{id}/termination`): solo la fecha. El sistema revisa ORG-R21. La salida se registra cuando ocurre: una fecha futura retiraría los roles antes de que la persona se vaya. Lo que le pasa a la cuenta lo hace `iam` en la misma transacción (ORG-R22 a ORG-R24).
- **Regreso** (`POST /employees/{id}/reactivation`): la nueva fecha de ingreso. El sistema revisa, en este orden: que el colaborador exista en el alcance; que haya salido; que la fecha no sea anterior a la salida (ORG-R37); que su empresa y su sede estén activas (ORG-R58); y que la fecha no sea posterior a hoy (ORG-R37). Guarda el período que cierra (ORG-R60). Si la salida había cerrado la cuenta, `iam` la reabre; los roles no vuelven solos.
- **Si falla:** no sale ni regresa, una cuenta cerrada por la salida sigue cerrada, y cada regla responde su código; `EMPLOYEE_ALREADY_TERMINATED` y `EMPLOYEE_ALREADY_ACTIVE` dicen que la lista quedó vieja, y la pantalla la vuelve a leer.

### F8 · Enviar el enlace para restablecer la contraseña de un colaborador

Es una operación de `iam` sobre la cuenta ([iam.md](iam.md), F8), para el superadministrador y el administrador de accesos, desde esta pantalla o desde Seguridad › Usuarios: el administrador no entrega una contraseña, le envía a la persona un enlace de un solo uso a su correo (D-243). Una cuenta que espera su activación todavía no tiene contraseña: su fila ofrece en cambio reenviar el código de activación y corregir el correo, con los mismos diálogos de `iam` ([iam.md](iam.md), F15 y F16; D-259).

### F9 · Bloquear la cuenta de un colaborador

El bloqueo es de la cuenta, no del colaborador (§5.4): es de `iam` ([iam.md](iam.md), F7). Esta pantalla abre el mismo diálogo que Seguridad › Usuarios, para el superadministrador y el administrador de accesos.

### F10 · Corregir el documento tributario de una empresa (D-85)

- **Quién:** quien tiene `configuracion.editar_empresas` en toda la empresa; con la semilla, solo `SUPERADMIN` (ORG-R46).
- **Pantalla:** Configuración › Empresas, en la edición de la empresa, desde la lista o desde su ficha. Un documento cambiado se corrige primero, por su propio camino, y después la dirección y los demás datos, solo si cambiaron.
- **Pasos** (`PUT /companies/{id}/tax-id`): el sistema bloquea la empresa y revisa la corrección como el alta (ORG-R46, con ORG-R02 a ORG-R04 y ORG-R52). Una corrección que no cambia nada no guarda nada.
- **Si falla:** `COMPANY_TAX_ID_CONFLICT` o `COMPANY_TAX_ID_CHECK_DIGIT_INVALID`, entre otros.
- **Resultado:** la empresa conserva su fila, sus sedes y su historia con el documento corregido. El país no se corrige nunca.

### F11 · Desvincular la cuenta de un colaborador (D-82)

Es una operación de `iam` ([iam.md](iam.md), F11), con `POST /employees/{id}/account-unlinking`. Corrige un vínculo equivocado, por ejemplo uno hecho con un documento mal escrito en las dos empresas.

- **Quién:** quien tiene `seguridad.desvincular_cuentas` con alcance sistema, que solo tiene `SUPERADMIN` y ningún rol propio incluye (D-117, D-122), con las condiciones de ORG-R43 y ORG-R50 y, sobre una cuenta protegida, solo si actúa sobre lo reservado (D-121).
- **Pantalla:** Configuración › Colaboradores, en la fila de un colaborador con cuenta que lo ofrece. Pregunta antes, diciendo que la cuenta perderá sus roles en esa empresa y que, sin otro colaborador activo, quedará cerrada.
- **Lo que pone `organization`:** borra el vínculo del colaborador bloqueado, o responde `EMPLOYEE_ACCOUNT_MISSING` si ya no tiene cuenta. El colaborador sigue registrado y activo.
- **Lo que hace `iam`, en la misma transacción:** ORG-R43, con el freno del último superadministrador (ORG-R24), y el registro del desvínculo (ORG-R44).
- **Resultado:** la pantalla dice si la cuenta quedó cerrada y vuelve a leer la lista. El colaborador puede recibir su propio acceso (F6).

### F12 · Las cuentas bancarias de una empresa (D-214)

- **Quién:** registra, corrige y activa o desactiva quien tiene el permiso de cada acción en toda la empresa (ORG-R63); lee quien tiene `configuracion.ver_estructura` donde actúa, y el número completo solo quien gestiona las cuentas (ORG-R66).
- **Pantalla:** la ficha de la empresa, en sus cuentas bancarias.
- **Pasos:** registra (`POST /companies/{companyId}/bank-accounts`), corrige (`PUT /bank-accounts/{id}`, con el momento en que la abrió) y activa o desactiva (`PATCH /bank-accounts/{id}/status`). El sistema revisa que la empresa esté activa (ORG-R45, ORG-R58), el nombre, el número y la moneda (ORG-R64) y, al corregir, si salieron pagos de la cuenta y si cambió desde que se abrió (ORG-R65).
- **Si falla:** cada regla responde su código, como `BANKACCOUNT_NAME_CONFLICT`, `BANKACCOUNT_NUMBER_CONFLICT`, `BANKACCOUNT_IN_USE` o `BANKACCOUNT_OUTDATED`.
- **Resultado:** la cuenta dice si tiene pagos, y la pantalla no ofrece cambiar su número ni su moneda. El registro de un pago la elige entre las de la empresa en `GET /supplier-payment-accounts`, con `pagos.ver` y nunca con el número completo ([procurement.md](procurement.md)).

### F13 · El historial de ingresos de un colaborador (D-241)

- **Quién:** quien lee la fila mínima de los colaboradores de la empresa (ORG-R62). No hace falta el permiso de los datos personales.
- **Pantalla:** Configuración › Colaboradores, en cada fila de una empresa donde la sesión tiene ese permiso, en un diálogo de solo lectura con cada período, el vigente señalado. Si no puede leerlo, lo dice, nunca con una lista vacía.
- **Pasos:** `GET /employees/{id}/tenure-periods` (ORG-R62).
- **Si falla:** `403 ACCESS_DENIED` o `404 EMPLOYEE_NOT_FOUND`.

## 4. Reglas de negocio

| Id | Regla | Fuente | Capa | Prueba | Estado |
|---|---|---|---|---|---|
| ORG-R01 | Una sola empresa por país | D-5, §6.2 | Base, Servicio | `CompaniesControllerTests`, `CompanyCommandServiceImplTests`, `OrganizationNaturalKeysTests` | ✅ |
| ORG-R02 | El documento tributario no se repite en el país y se guarda sin espacios, guiones ni puntos, en mayúsculas. Un número hecho solo de separadores responde `400` en su campo, con cualquier tipo, también el RUC | §6.2, §2.2 | Base, Dominio, Servicio | `CompanyCommandServiceImplTests`, `CompanyTests`, `CompaniesControllerTests`, `TextIdentifierSizeValidatorTests`, `OrganizationOpenApiTests` | ✅ |
| ORG-R03 | El tipo de documento de la empresa es de su país, está activo, aplica a empresas, y el número tiene su longitud | §6.2, §4.7 | Base, Servicio | `TaxIdTypeRulesTests`, `CompaniesControllerTests` | ✅ |
| ORG-R04 | Un número cuyo tipo declara dígito de control lo cumple. Se verifica al registrarlo y cuando cambian el número o el tipo; uno ya guardado sigue siendo válido | §4.7, §6.2, D-75 | Dominio, Servicio, Frontend | `TaxIdTypeRulesTests`, `CompanyCommandServiceImplTests`, `CompaniesControllerTests`, `EmployeeCommandServiceImplTests`, `EmployeesIntegrationTests`, `company-form-dialog.spec.ts`, `employee-form-dialog.spec.ts` | ✅ |
| ORG-R05 | La dirección fiscal es solo de esa empresa y está en su país | §4.6 | Base, Servicio | `CompanyFiscalAddressConstraintTests`, `SameCountryReferencesMigrationTests` | ✅ |
| ORG-R06 | Corregir una dirección modifica su fila; mudarse crea una nueva; el `PUT` del dueño no toca la dirección. Vale para empresas, sedes y ubicaciones de terceros, y al cambiarla la pantalla pregunta cuál de las dos es | D-8, D-79; convenciones §3.3 | Servicio, Frontend | `CompaniesControllerTests`, `BranchesIntegrationTests`, `LocationsIntegrationTests`, `location-form-dialog.spec.ts`, `locations.spec.ts` | ✅ |
| ORG-R07 | La sede está en el país de su empresa, y su dirección es solo suya y de ese país | §6.4, §4.6 | Base, Servicio | `BranchConstraintTests` | ✅ |
| ORG-R08 | El código de la sede y su código de establecimiento no se repiten en la empresa; el código no cambia | §6.4; convenciones §7.2 | Base, Dominio, Servicio | `BranchCommandServiceImplTests`, `BranchConstraintTests`, `BranchesIntegrationTests`, `OrganizationNaturalKeysTests` | ✅ |
| ORG-R09 | El nombre de una sede o de una marca puede repetirse en la empresa | §6.3, §6.4 | Base | `BranchCommandServiceImplTests`, `CommercialBrandCommandServiceImplTests` | ✅ |
| ORG-R10 | La zona horaria de la sede es obligatoria y es el nombre IANA de una región, sin valor por defecto | §6.4, §2.6, D-40 | Base, Dominio, Servicio | `BranchTests`, `BranchesIntegrationTests`, `BranchReconstitutionTests` | ✅ |
| ORG-R11 | La sede no lleva tipo | D-50 | Base | Ninguna: la columna no existe | ✅ |
| ORG-R12 | El código de la marca no se repite en la empresa, se guarda en mayúsculas y no cambia | §6.3 | Base, Dominio, Servicio | `CommercialBrandCommandServiceImplTests`, `CommercialBrandTests`, `CommercialBrandsIntegrationTests`, `OrganizationNaturalKeysTests` | ✅ |
| ORG-R13 | El código de la ubicación no se repite en la empresa y no cambia; el tipo es uno de los cinco | §6.5 | Base, Servicio | `LocationCommandServiceImplTests`, `LocationsIntegrationTests`, `OrganizationNaturalKeysTests` | ✅ |
| ORG-R14 | Una `sede_propia` apunta a una sede y no tiene dirección propia; las demás tienen dirección y no apuntan a una sede | §6.5, D-79 | Base, Dominio, Servicio | `LocationTests`, `LocationCommandServiceImplTests`, `LocationsIntegrationTests`, `LocationAndEmployeeChecksMigrationTests` | ✅ |
| ORG-R15 | La sede de una ubicación propia es de la misma empresa y está activa si es nueva para la ubicación; la que ya tiene se conserva aunque se haya desactivado | §6.5, D-84 | Base, Dominio, Frontend | `LocationCommandServiceImplTests`, `BranchAffiliationPolicyTests`, `OrganizationOwnershipKeysMigrationTests` | ✅ |
| ORG-R16 | La dirección de una ubicación es solo suya y está en el país de la empresa | §4.6; mapa §6.3 | Base, Servicio | `OrganizationOwnershipKeysMigrationTests`, `LocationCommandServiceImplTests` | ✅ |
| ORG-R17 | El tipo de documento del colaborador es del país de la empresa, aplica a personas, está activo si es nuevo para el colaborador, y el número tiene su longitud | §5.3, §4.7 | Servicio, Frontend | `EmployeeCommandServiceImplTests`, `TaxIdTypeRulesTests`, `employee-form-dialog.spec.ts` | ✅ |
| ORG-R18 | El documento de identidad no se repite en la empresa —la base lo compara por su huella (ORG-R67)—, y se guarda en mayúsculas y sin guiones, espacios ni puntos. El documento es el tipo y el número: el mismo número con otro tipo es otro documento, como compara D-82. Un número hecho solo de separadores responde `400` en su campo | §5.3, §2.2, D-82 | Base, Dominio, Servicio, Frontend | `EmployeeCommandServiceImplTests`, `EmployeeTests`, `EmployeesIntegrationTests`, `TextIdentifierSizeValidatorTests`, `OrganizationOpenApiTests`, `OrganizationNaturalKeysTests`, `TextIdentifiersMigrationTests` | ✅ |
| ORG-R19 | La sede del colaborador es de su empresa, y está activa si es nueva para el colaborador; la que ya tiene se conserva aunque se haya desactivado | §5.3, D-84 | Base, Dominio, Frontend | `EmployeeCommandServiceImplTests`, `BranchAffiliationPolicyTests`, `OrganizationOwnershipKeysMigrationTests`, `employee-form-dialog.spec.ts` | ✅ |
| ORG-R20 | Una cuenta se vincula a lo sumo a un colaborador por empresa | §5.3 | Base, Servicio | `OrganizationOwnershipKeysMigrationTests`, `OrganizationContextFacadeImplTests`, `OrganizationNaturalKeysTests` | ✅ |
| ORG-R21 | Activo es sin fecha de salida; la salida no es anterior al ingreso, no es posterior a hoy en la zona del colaborador y no se registra dos veces; cada regla responde su propio código. Al editar un colaborador que salió, su fecha de ingreso no pasa después de su salida (`422 EMPLOYEE_TERMINATION_BEFORE_HIRE` sobre `hireDate`) | §5.3, D-60 | Base, Dominio, Servicio | `EmployeeTests`, `EmployeeCommandServiceImplTests`, `EmployeesIntegrationTests`, `EmployeeTenureIntegrationTests`, `LocationAndEmployeeChecksMigrationTests` | ✅ |
| ORG-R22 | Al registrar la salida se retiran los roles de la cuenta **en esa empresa**, con motivo `baja_colaborador`. Lo hace `iam`, en la misma transacción que la salida ([iam.md](iam.md), IAM-R40) | §5.3, D-38, D-41 | Servicio | `EmployeeTenureIntegrationTests` | ✅ |
| ORG-R23 | Si a la cuenta no le queda colaborador activo, la salida la cierra con un bloqueo `baja`. Lo hace `iam` ([iam.md](iam.md), IAM-R40, R41) | §5.3, D-38, D-12, D-60 | Servicio | `EmployeeTenureIntegrationTests`, `CrossCompanyAccountAdministrationIntegrationTests` | ✅ |
| ORG-R24 | No se registra la salida del último superadministrador activo, ni el desvínculo que lo dejaría sin colaborador activo. Lo frena `iam` | §5.9, D-65 | Servicio | `EmployeeTenureIntegrationTests`, `EmployeeAccountUnlinkingIntegrationTests` | ✅ |
| ORG-R25 | Se da acceso solo a un colaborador activo, y la cuenta queda vinculada en la misma operación; si el correo ya tiene cuenta, se vincula a ella solo si es de la misma persona (ORG-R41). Lo hacen `iam` y la fachada ([iam.md](iam.md), IAM-R36) | D-38, D-58, D-82, D-255, §5.3 | Servicio | `AccessGrantsIntegrationTests`, `OrganizationContextFacadeImplTests` | ✅ |
| ORG-R26 | La sede y la marca de una asignación son de la empresa de la asignación | §5.7 | Base, Servicio | `OrganizationContextFacadeImplTests`, `RoleAssignmentScopeMigrationTests` | ✅ |
| ORG-R27 | El primer superadministrador no da acceso ni otorga roles hasta vincularse a su perfil. Lo hace `iam` ([iam.md](iam.md), IAM-R14); lo que tampoco registra hasta vincularse está en ORG-R50 | §5.2, §5.9, D-38 | Servicio | `RoleAssignmentsIntegrationTests` | ✅ |
| ORG-R29 | Registran colaboradores quienes tienen `seguridad.registrar_colaboradores` donde actúa, y los modifican quienes tienen `seguridad.modificar_colaboradores`: un colaborador sin sede, en toda la empresa; un cambio de sede, en la de origen y en la de destino; un rol de sede, en su sede. Con la semilla, el superadministrador, en cualquier empresa y también sin estar vinculado (D-89), y el administrador de la empresa, en la suya; el administrador de accesos no. Una empresa fuera del alcance responde `404`; una donde el permiso no actúa, `403` | §2.5, D-57, D-89, D-117, D-122, D-130 | Servicio, Frontend | `EmployeeAdministrationIntegrationTests`, `AccessAdministratorIntegrationTests`, `EmployeeCommandServiceImplTests`, `EmployeeAllowedActionsIntegrationTests`, `SecurityOperationsBySystemRoleIntegrationTests`, `security.live.spec.ts` | ✅ |
| ORG-R30 | Las empresas se escriben con `configuracion.crear_empresas`, `editar_empresas` y `desactivar_empresas`, las sedes con `crear_sedes`, `editar_sedes` y `desactivar_sedes`, las marcas con `crear_marcas_comerciales`, `editar_marcas_comerciales` y `desactivar_marcas_comerciales` y las ubicaciones con `crear_ubicaciones`, `editar_ubicaciones` y `desactivar_ubicaciones`, cada uno solo donde la asignación que lo trae actúa en toda la empresa: una asignación de sede que trae uno de ellos nunca escribe estos recursos, que son de toda la empresa. En una empresa que la cuenta alcanza con un rol que no trae el permiso, `403 ACCESS_DENIED`; fuera del alcance, `404` (D-99). Registrar una empresa exige el permiso en todas. La semilla los da solo a `SUPERADMIN` | §5.2 (pasos 4 y 5), §5.6, D-43, D-95, D-99, D-130; procesos §1 y §3.2 | Servicio, Frontend | `OrganizationAccessTests`, `CompaniesControllerTests`, `BranchesIntegrationTests`, `OrganizationOpenApiTests` | ✅ |
| ORG-R31 | Las empresas, las marcas, las sedes, las ubicaciones y las cuentas bancarias se leen con `configuracion.ver_estructura` —en la lista, la exportación, por identificador y en las rutas anidadas—, solo en las empresas donde la asignación que lo trae actúa: una lista filtra por las empresas que alcanza con él, y una empresa ajena, o alcanzada con un rol que no trae el permiso, o lo que le pertenece, responde `404` (D-99). Los colaboradores siguen D-105 (ORG-R57). Alcanzar la empresa y Row Level Security quedan como piso | §2.5, §5.8 regla 6, D-66, D-99; convenciones §9.2 | Base, Servicio, Frontend | `CompanyIsolationIntegrationTests`, `RowLevelSecurityMigrationTests`, `OrganizationAccessTests`, `CompanyQueryServiceImplTests`, `shell.spec.ts` | ✅ |
| ORG-R32 | Un país opera si tiene una empresa activa; no se guarda | D-3 | Servicio | `CompanyQueryServiceImplTests` | ✅ |
| ORG-R33 | Nada se borra: se desactiva o se registra la salida | §5.3, §6.2, §6.4, §6.5; convenciones §3.1 | Base, Servicio | `OrganizationAccessTests`, `OrganizationRowsAreNeverDeletedTests` | ✅ |
| ORG-R34 | Columnas de registro: la empresa guarda quién la creó y quién la cambió; el colaborador, quién lo registró; marcas, sedes y ubicaciones, solo fechas | D-42; §5.3, §6.2 a §6.5 | Base, Servicio | `EmployeeAuditTests`, `CompanyAuditTests`, `BranchAuditTests`, `CommercialBrandAuditTests`, `LocationAuditTests` | ✅ |
| ORG-R35 | La moneda funcional nueva está activa; la que la empresa ya tiene se conserva aunque se haya desactivado | §4.1, §6.2 | Dominio, Servicio, Frontend | `FunctionalCurrencyTests`, `CompanyCommandServiceImplTests`, `CompaniesControllerTests` | ✅ |
| ORG-R36 | El filtro de sede de los colaboradores de una empresa nunca se descarta: una sede desconocida o de otra empresa da una lista vacía | convenciones §6.1 | Servicio | `CompanyIsolationIntegrationTests` | ✅ |
| ORG-R37 | El regreso tiene una fecha no anterior a la salida ni posterior a hoy en la zona del colaborador; cada regla responde su propio código | §5.3, D-60 | Servicio | `EmployeeTenureIntegrationTests`, `EmployeeCommandServiceImplTests` | ✅ |
| ORG-R38 | Las fechas de un colaborador se leen en la zona de su sede; sin sede, en la del país de su empresa; sin ninguna, en `America/Lima`. La API la entrega con cada colaborador (`timeZone`), resuelta en lote | §2.6, D-40 | Dominio, Servicio, Frontend | `EmployeeTimeZoneIntegrationTests`, `EmployeeAffiliationPolicyTests` | ✅ |
| ORG-R39 | Una cuenta con colaboradores y roles en dos empresas usa las pantallas de colaboradores de las dos; una empresa donde no tiene rol responde `404` aunque tenga un colaborador ahí, porque el alcance sale de las asignaciones y no del colaborador de la cuenta | §5.3, §5.7 | Servicio | `MultiCompanyEmployeeAccessIntegrationTests`, `EmployeeAdministrationIntegrationTests` | ✅ |
| ORG-R40 | Una ubicación no pasa de sede propia a ubicación de terceros ni al revés: es otro lugar, así que se desactiva y se registra una nueva. Entre los tipos de terceros sí cambia | §6.5, D-79 | Dominio, Servicio, Frontend | `LocationTests`, `LocationsIntegrationTests`, `LocationLockTests`, `locations.spec.ts` | ✅ |
| ORG-R41 | Una cuenta que ya existe se vincula a un colaborador solo si es de la misma persona: ya pertenece a un colaborador con la misma sigla de tipo de documento y el mismo número. Si no, `422 EMPLOYEE_ACCOUNT_IDENTITY_MISMATCH`, sin vincular nada. Solo un superadministrador la vincula igual, después de ver a los titulares, que ningún otro rol recibe, y confirmar con su huella; si los titulares cambiaron, vuelve a mostrarlos | §5.3, §5.7, D-82 | Servicio, Frontend | `EmployeeCommandServiceImplTests`, `AccessGrantsIntegrationTests`, `ErrorResponseAssemblerTest`, `OrganizationOpenApiTests` | ✅ |
| ORG-R42 | Una cuenta que existe sin ningún colaborador —por ejemplo, la que dejó un desvínculo— se vincula solo si un superadministrador confirma, al ver que no tiene titulares. La excepción es el primer superadministrador, que vincula su propia cuenta sin confirmar: todavía no tiene un colaborador con el cual compararla | D-82, D-38; §5.2, paso 6 | Servicio | `AccessGrantsIntegrationTests` | ✅ |
| ORG-R43 | Desvincular una cuenta funciona como una salida: solo un superadministrador vinculado lo hace, nunca sobre su propia cuenta; la cuenta pierde sus roles en esa empresa (`retiro`) y, sin colaborador activo, se cierra (`baja`, `baja_cuenta`, sesiones cerradas), con el freno del último superadministrador. El colaborador sigue registrado, sin cuenta | §5.3, D-82, D-60, D-65, D-69 | Servicio, Frontend | `EmployeeCommandServiceImplTests`, `EmployeeAccountUnlinkingIntegrationTests`, `EmployeeAccountUnlinkingRollbackTests`, `SessionClosingIntegrationTests` | ✅ |
| ORG-R44 | Cada vínculo confirmado de la cuenta de otra persona y cada desvínculo dejan una fila en `employee_account_link_events`, en la misma transacción que el cambio, que nunca se edita ni se borra; el vínculo confirmado guarda los titulares que vio el superadministrador, y el desvínculo ninguno. La leen solo un superadministrador vigente y los procesos del sistema | §5.4, D-82 | Base | `EmployeeAccountLinkEventsMigrationTests`, `AccessGrantsIntegrationTests`, `HandOverDatabaseScriptTests` | ✅ |
| ORG-R45 | A una empresa, una sede o una marca inactiva no se le cuelga nada nuevo: ni sedes, marcas, ubicaciones, cuentas bancarias o colaboradores bajo una empresa inactiva, ni colaboradores o ubicaciones propias en una sede inactiva, ni roles sobre ninguna de las tres. Lo que ya apuntaba a ella se sigue modificando mientras ese vínculo no cambie. Reactivar y regresar siguen la misma regla (ORG-R58) | §6.2 a §6.5, §5.3, §5.7, D-84 | Servicio, Frontend | `BranchCommandServiceImplTests`, `CommercialBrandCommandServiceImplTests`, `LocationCommandServiceImplTests`, `EmployeeCommandServiceImplTests`, `RoleAssignmentsIntegrationTests`, `BranchAffiliationPolicyTests`, `BankAccountsIntegrationTests` | ✅ |
| ORG-R46 | Quien tiene `configuracion.editar_empresas` en toda la empresa —con la semilla, solo el superadministrador— corrige el tipo y el número del documento tributario de una empresa, y la corrección se verifica como el alta: tipo del país que aplique a empresas, activo si es nuevo, longitud, dígito de control y número no repetido en el país. El país no cambia | §6.2, D-85, D-130 | Servicio, Frontend | `CompanyCommandServiceImplTests`, `TaxIdTypeRulesTests`, `CompaniesControllerTests`, `OrganizationAccessTests` | ✅ |
| ORG-R47 | Los códigos propios —el de una sede y el de una ubicación— se guardan recortados y en mayúsculas y conservan sus guiones: `LIM-1` y `LIM1` son dos códigos | §2.2, D-86 | Dominio, Frontend | `BranchesIntegrationTests`, `LocationsIntegrationTests`, `location-form-dialog.spec.ts` | ✅ |
| ORG-R48 | La caducidad de un rol es un momento. Cuando la pantalla elige un día, el rol vale ese día completo y caduca al empezar el día siguiente en la zona de la empresa, también un rol de sede; "vence el 31/12/2026" en Perú es `2027-01-01T05:00:00Z`. Lo aplica el diálogo de roles de `iam` ([iam.md](iam.md), F6) | §5.7, D-87 | Servicio, Frontend | `role-assignments-dialog.spec.ts` | ✅ |
| ORG-R49 | La fecha de ingreso no es posterior a hoy en la zona de la sede del colaborador o, sin sede, del país de su empresa, al registrarlo y al modificarlo | §5.3, D-88 | Servicio, Frontend | `EmployeeCommandServiceImplTests`, `employee-form-dialog.spec.ts` | ✅ |
| ORG-R50 | El superadministrador sin colaborador vinculado registra y modifica colaboradores, pero no registra salidas, regresos ni desvínculos hasta vincularse (`403 ACTOR_EMPLOYEE_REQUIRED`). D-89 cubre las salidas y los regresos; el desvínculo queda fuera porque D-82 lo hace funcionar como una salida y es una operación de cuenta, que exige a quien opera vinculado a un colaborador activo (D-57; F11) | §5.2, §5.3, D-89; D-82, D-57 | Servicio, Frontend | `EmployeeTenureIntegrationTests`, `EmployeeAccountUnlinkingIntegrationTests`, `EmployeeAdministrationIntegrationTests` | ✅ |
| ORG-R51 | Una sede tiene a lo sumo una ubicación propia, activa o inactiva; la inactiva se reactiva en vez de crear otra, y no se reactiva mientras su sede siga inactiva | §6.5, D-90 | Base, Servicio, Frontend | `LocationCommandServiceImplTests`, `LocationsIntegrationTests`, `OrganizationNaturalKeysTests` | ✅ |
| ORG-R52 | La longitud de un número de documento, como su dígito de control, se verifica al crear el registro y cuando cambian el número o el tipo; un número que no cambia no se vuelve a medir | §4.7, §6.2, D-91 | Dominio | `TaxIdTypeRulesTests` | ✅ |
| ORG-R53 | La ubicación de la foto de un colaborador tiene a lo sumo 500 caracteres | §5.3 | Base, Servicio, Frontend | `EmployeesIntegrationTests`, `LocationAndEmployeeChecksMigrationTests` | ✅ |
| ORG-R54 | La empresa nombra su país y su moneda funcional por su código (`country: "PE"`, `functionalCurrency: "PEN"`), y el tipo de documento por `taxIdTypeId`; la dirección embebida de empresas, sedes y ubicaciones, su país por `country` | D-13; convenciones §4.2, §13 filas 21 y 22 | Servicio, Frontend | `CompaniesControllerTests`, `BranchesIntegrationTests` | ✅ |
| ORG-R55 | El tipo de una ubicación viaja en minúsculas, como lo escribe el diccionario (`sede_propia`…); al leer se aceptan las dos formas, en el cuerpo y en el filtro. La exportación lo nombra en el idioma de la petición | §6.5; convenciones §4.1 | Servicio, Frontend | `LocationsIntegrationTests`, `OrganizationOpenApiTests` | ✅ |
| ORG-R56 | Lo que lee y después cambia una fila la bloquea antes: la edición, el cambio de estado y el cambio de dirección de una empresa o una sede, y la edición y el cambio de estado de una marca, con `FOR NO KEY UPDATE`; las altas, el cambio de sede, la reactivación de una sede, una marca o una ubicación, el regreso de un colaborador y el alcance de un rol nuevo leen su empresa, sede y marca con `FOR SHARE`, siempre en ese orden, así esperan a un cambio de estado simultáneo | Patrones de persistencia; D-84, D-90, D-94 | Servicio | `CompanyLockTests`, `BranchLockTests`, `ParentRowLocksTests`, `EmployeeCommandServiceImplTests`, `LocationCommandServiceImplTests`, `OrganizationContextFacadeImplLockOrderTests`, `CompanySharedLockTests` | ✅ |
| ORG-R57 | Los colaboradores de una empresa —la lista, su página, el filtro de sede y cada uno— los lee cualquier rol con `configuracion.ver_colaboradores` donde ese permiso actúa, con la fila mínima: nombre, puesto, sede, fechas, si está activo y si tiene cuenta; la semilla se lo da a los siete roles del sistema. El documento, el teléfono y la foto viajan solo con `configuracion.ver_datos_personales_colaboradores` en esa empresa —`SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`— y si no, se omiten; la exportación exige ese permiso en su ruta (convenciones §7.2). Una empresa alcanzada con un rol que no trae `ver_colaboradores`, o fuera del alcance, responde `404` (D-99) | §2.5, §5.3, §5.13, D-93, D-99, D-105 | Servicio, Frontend | `OrganizationAccessTests`, `EmployeeQueryServiceImplTests`, `OrganizationOpenApiTests`, `employees-view.spec.ts`, `employee-assembler.spec.ts`, `employee-form-dialog.spec.ts`, `app-shell.spec.ts`, `access.spec.ts` | ✅ |
| ORG-R58 | Reactivar es volver a colgar: una sede, una marca, una ubicación o una cuenta bancaria inactivas se reactivan solo con su empresa activa (`422 COMPANY_INACTIVE`), y en una ubicación propia la empresa se revisa antes que la sede; un colaborador regresa solo a una empresa activa y, si tiene sede, a una sede activa (`COMPANY_INACTIVE`, `BRANCH_INACTIVE`), y si no regresa su cuenta sigue cerrada. Lo que ya está activo no se revisa, y desactivar no espera a nada | §5.3, §6.2 a §6.5, D-94 | Servicio, Frontend | `BranchCommandServiceImplTests`, `CommercialBrandCommandServiceImplTests`, `LocationCommandServiceImplTests`, `EmployeeCommandServiceImplTests`, `BranchesIntegrationTests`, `CommercialBrandsIntegrationTests`, `LocationsIntegrationTests`, `EmployeeTenureIntegrationTests`, `BankAccountsIntegrationTests`, `OrganizationOpenApiTests`, `company-profile.spec.ts` | ✅ |
| ORG-R59 | Las exportaciones nombran la sede por su código además de su nombre, porque dos sedes de una empresa pueden llamarse igual (ORG-R09): la de colaboradores, y la de ubicaciones para una ubicación propia | §6.4; convenciones §8.1 | Servicio | `EmployeesIntegrationTests`, `LocationsIntegrationTests` | ✅ |
| ORG-R60 | El regreso de un colaborador guarda el período que cierra —su ingreso y su salida anteriores, su empresa, quién registró el regreso y cuándo— en `employee_tenure_periods`, en la misma transacción en que reemplaza la fecha de ingreso y borra la de salida; un regreso rechazado no guarda nada. Un período es de solo inserción: no se edita ni se borra, su salida no es anterior a su ingreso y nunca deja su colaborador ni su empresa. El período vigente sigue en `employees` | §5.3, D-241 | Base, Dominio, Servicio, Frontend | `EmployeeTests`, `EmployeeTenurePeriodTests`, `EmployeeTenurePeriodAuditTests`, `EmployeeTenureIntegrationTests`, `employees.spec.ts` | ✅ |
| ORG-R61 | Al corregir un colaborador, la fecha de ingreso no queda antes de la salida de su último período cerrado, que pisaría: `422 EMPLOYEE_HIRE_DATE_BEFORE_PREVIOUS_TERMINATION` en `hireDate`. El mismo día se acepta, y un colaborador sin períodos cerrados toma cualquier fecha anterior que las demás reglas acepten | §5.3, D-241 | Dominio, Servicio, Frontend | `EmployeeAffiliationPolicyTests`, `EmployeeCommandServiceImplTests`, `EmployeeTenurePeriodsIntegrationTests`, `employees.spec.ts` | ✅ |
| ORG-R62 | Los períodos de un colaborador los lee quien lee su fila mínima, con `configuracion.ver_colaboradores` donde actúa (D-105), en `GET /employees/{id}/tenure-periods`: primero el período que el colaborador tiene, vigente o cerrado por su última salida, y después los cerrados, el más nuevo primero. Sin el permiso, `403`; un colaborador que no existe o de una empresa donde no actúa, `404 EMPLOYEE_NOT_FOUND` | §5.3, D-105, D-241 | Servicio, Frontend | `EmployeeTenurePeriodsIntegrationTests`, `EmployeeTenurePeriodAuditTests`, `employee-tenure-history-dialog.spec.ts`, `employees.spec.ts` | ✅ |
| ORG-R63 | Una cuenta bancaria la registra, la corrige y la activa o desactiva quien tiene `configuracion.crear_cuentas_bancarias`, `editar_cuentas_bancarias` o `desactivar_cuentas_bancarias` en toda la empresa; con la semilla, `SUPERADMIN` y `ADMIN_EMPRESA`. Nace bajo una empresa activa y se reactiva solo bajo ella (D-84). Nunca deja su empresa y nunca se borra | §6.2.1, D-84, D-130, D-214 | Base, Servicio, Frontend | `BankAccountsIntegrationTests`, `BankAccountRowLevelSecurityTests` | ✅ |
| ORG-R64 | El nombre —de hasta 60 caracteres— y el número —de hasta 34 letras, dígitos, espacios o guiones— de una cuenta no se repiten en su empresa, y sí en otra; el banco tiene hasta 100. La moneda existe y está activa, salvo la que la cuenta ya tiene, que se conserva aunque se haya desactivado | §6.2.1, D-84 | Base, Dominio, Servicio | `BankAccountsIntegrationTests`, `BankAccountTests`, `BankAccountRowLevelSecurityTests` | ✅ |
| ORG-R65 | Una cuenta que nombra un registro de otro contexto, vigente o anulado —hoy un pago que salió de ella—, ya no cambia su número ni su moneda (`422 BANKACCOUNT_IN_USE`); el nombre y el banco sí. Lo sabe por el puerto que declara, que hoy implementa `procurement` (sección 5), y la corrección espera al pago que se está registrando desde la cuenta. Una corrección lleva el `updatedAt` con que se abrió la cuenta, y si cambió después responde `422 BANKACCOUNT_OUTDATED` sin guardar | §6.2.1, D-214 | Dominio, Servicio, Frontend | `BankAccountsIntegrationTests`, `SupplierPaymentAccountsIntegrationTests`, `BankAccountUsageWiringTests`, `BankAccountUsageFactsTests`, `BankAccountLockTests`, `BankAccountTests`, `bank-account-form-dialog.spec.ts` | ✅ |
| ORG-R66 | El número completo de una cuenta lo lee solo quien gestiona las cuentas de esa empresa —uno de sus tres permisos en toda ella—; los demás, y el registro de un pago, leen sus últimos cuatro dígitos o letras, sin espacios ni guiones, y no buscan ni ordenan por el número, que lo diría dígito a dígito | §6.2.1, D-214 | Dominio, Servicio, Frontend | `BankAccountsIntegrationTests`, `BankAccountTests`, `SupplierPaymentAccountsIntegrationTests` | ✅ |
| ORG-R67 | El documento y el teléfono de un colaborador se guardan cifrados, con una clave que la base no ve: cada valor con un nonce propio y autenticado para su columna, así que un mismo número no repite sus bytes y los de una columna no se leen en otra. El número lleva su huella, el HMAC de su forma normalizada, que es lo que la base compara. Las columnas en claro quedan vacías y la base rechaza un valor en ellas; sin la clave la aplicación no arranca | §5.3, D-93, D-261 | Base, Servicio | `PersonalDataCipherTests`, `EmployeePersonalDataTests`, `EmployeePersonalDataStorageTests`, `EmployeesIntegrationTests` | ✅ |
| ORG-R68 | La búsqueda de los colaboradores encuentra el documento solo por su número completo, escrito con o sin separadores; los nombres y el puesto, por cualquier parte. La lista no se ordena por documento: un orden que lo pide cae en el de por omisión | §5.3, D-261; convenciones §6.2 | Servicio, Frontend | `EmployeesIntegrationTests` | ✅ |

## 5. Relación con otros contextos

**Lo que necesita.** `organization` depende solo de `location::acl`, de `documents::acl`, este para sus cinco exportaciones (D-125) con la versión de sus datos (D-128), y de `audit::acl`, para implementar `AuditCompanyLookupPort`: la empresa de cada fila de la bitácora y de las cargas, con su zona (D-129, D-262). No llama a `iam` ni a `procurement`: es `iam` quien llama a `organization` (mapa §4, decisión 4). De `shared` usa tres puertos que implementa `iam`: `CurrentAccessScope`, el alcance de quien llama —qué empresas alcanza y con qué permiso actúa en cada una (D-99)—; `ProtectedAccounts`, qué cuentas son protegidas y si la sesión actúa sobre lo reservado (D-121); y `PendingAccountActivations`, qué cuentas esperan su activación (D-259). El autor de sus registros sale del auditor que aporta `iam` (D-42).

De `location` usa: el país y la moneda por su código, con sus nombres y zonas en lote; las reglas de un tipo de documento, con el dígito de control que calcula `location` (D-75); y crear, corregir, reemplazar y leer en lote las direcciones, en el país que fija el dueño.

**Lo que ofrece: `OrganizationContextFacade`**, a `iam`, `fleet` y `procurement`:

| Para qué | Miembros | Quién los usa |
|---|---|---|
| Saber si existe una empresa: la de las cuentas de una empresa, la de un rol propio, que tiene que estar activa (D-98), y la de una unidad o de un documento de compra | `fetchCompanySummaryByPublicId` | `iam`, `fleet`, `procurement` |
| El colaborador que recibe acceso, sale, regresa o se desvincula, bloqueado hasta que termina la operación | `fetchEmployeeSummaryForUpdate` | `iam` |
| En qué empresas tiene colaboradores una cuenta, y dónde tiene uno activo, para una cuenta o en lote | `fetchEmployeeSummariesByUserId`, `fetchActiveEmployeeCompanyIdsByUserId`, `fetchActiveEmployeeCompanyIdsByUserIds` | `iam` |
| Los titulares de la cuenta de un correo (D-82) | `fetchAccountHoldersByUserId` | `iam` |
| Vincular un colaborador a una cuenta nueva o a la de la misma persona, y deshacer el vínculo (D-82) | `linkUserToEmployee`, `linkExistingUserToEmployee`, `unlinkUserFromEmployee` | `iam` |
| Registrar la salida y el regreso con sus reglas (F7) | `terminateEmployee`, `reactivateEmployee` | `iam` |
| Resolver el alcance de una asignación, la marca de una unidad o de un fondo, o la empresa y la marca de una orden: la empresa, y la sede y la marca de esa empresa, activas (D-84; ORG-R56) | `resolveScope` | `iam`, `fleet`, `procurement` |
| Nombres en lote para las listas, las empresas activas, la zona de cada empresa y las cuentas con colaborador en una empresa | `fetchCompanyReferencesByIds`, `fetchCommercialBrandReferencesByIds` y `fetchCompanyTimeZonesByIds` (los tres); `fetchBranchReferencesByIds`, `fetchActiveCompanyReferences`, `fetchLinkedEmployeePublicIdsByCompanyId` (`iam`) | `iam`, `fleet`, `procurement` |
| El nombre del colaborador de cada cuenta de una lista, el de uno activo si lo hay (mapa, decisión 6) | `fetchEmployeeNamesByUserIds` | `iam` |
| La marca comercial de un filtro, dentro de las empresas que quien llama lee (D-99) | `fetchCommercialBrandReferenceByPublicId` | `fleet`, `procurement` |
| La ubicación donde se pone una unidad, una o en lote, dentro de las empresas que quien llama lee (D-99) | `fetchLocationReferenceByPublicId`, `fetchLocationReferencesByIds` | `fleet` |
| El emisor de una plantilla, que se copia de la empresa (D-134) | `fetchCompanyIssuerByPublicId` | `procurement` |
| El colaborador de Atención de una orden, o quien instala un equipo, uno o en lote | `fetchEmployeeAttentionByPublicId`, `fetchEmployeeAttentionsByIds` | `procurement`, `fleet` |
| La empresa que compra y las marcas de una empresa, una o en lote, y en modo compartido mientras se registra el documento | `fetchPurchasingCompanyForShare`, `fetchPurchasingCompaniesByIds`, `fetchCommercialBrandReferencesByPublicIds`, `fetchCommercialBrandReferencesByIdsForShare` | `procurement`, `fleet` |
| La cuenta de origen de un pago, en modo compartido hasta que el pago confirma, y las cuentas en lote o de una empresa, siempre sin el número completo (D-214) | `fetchBankAccountForShare`, `fetchBankAccountsByIds`, `fetchBankAccountsByPublicIds`, `fetchBankAccountsOfCompany` | `procurement` |

Las lecturas que usa `iam` no aplican el filtro de alcance de la API: `iam` decide quién puede actuar sobre lo que pide, y con alcance de sistema solo lee datos sueltos, nunca agregados. Lo que usa `fleet` está también en [fleet.md](fleet.md), sección 5.

**El puerto que declara: `BankAccountUsage`.** Dice cuáles de unas cuentas nombra un registro del contexto que lo implementa, vigente o anulado (ORG-R65). `organization` no depende de quien lo implementa: pregunta a todos los que haya y une lo que responden, y sin ninguno ninguna cuenta está en uso. Un contexto cuyos registros nombren cuentas lo implementa sin tocar `organization`; hoy lo implementa solo `procurement`, que cuenta los pagos vigentes y los anulados. Quien lo implementa:

- responde con el alcance de quien llama: una cuenta y los registros que la nombran son de la misma empresa;
- cuenta también lo anulado, que salió igual de la cuenta y conserva el número y la moneda con que se registró;
- lee una sola vez por llamada, para toda una página de cuentas;
- lee la cuenta con `fetchBankAccountForShare` en cada escritura que la nombra, así que la corrección del número o de la moneda, que bloquea la cuenta, ve esa escritura o la espera.

**Frontend.** `OrganizationFacade` devuelve vistas mínimas propias:

| Miembro | Qué da | Quién lo usa |
|---|---|---|
| `loadCompanyOptions` | Las empresas, activas e inactivas, con su país y su moneda funcional | `iam` (el alcance y las fechas de un rol, y la empresa de un rol propio, D-98), `mm-company-filter`, `fleet`, `procurement`, las cargas y el registro de cambios |
| `loadCompanyUnits` | Las sedes, con su zona, y las marcas activas de una empresa | `iam`, `fleet` |
| `loadBranchTimeZones` | La zona de cada sede de una empresa | `iam` (los días de un rol de sede) |
| `loadCommercialBrands` | Todas las marcas de una empresa, para nombrar la que un registro ya tiene y ofrecer las activas (D-84) | `fleet`, `procurement`, `reporting` |
| `loadCompanyLocations` | Todas las ubicaciones de una empresa, para decir dónde está una unidad y ofrecer las activas | `fleet`, `procurement` |
| `loadCompanyIssuer` | La identidad fiscal de una empresa, como emisora de una plantilla nueva | `procurement` |
| `loadActiveCollaborators` | Los colaboradores activos de una empresa, para nombrar a uno en un documento | `procurement`, `fleet` |

`organization` presta además `mm-company-filter` (DP-9), el filtro de empresa de las listas de los demás contextos: ofrece las empresas inactivas solo a quien lee la estructura (DP-14; D-84), cada lista puede acotarlo a las empresas donde la sesión la lee, y sin ninguna empresa que ofrecer la lista se muestra vacía en lugar de quedarse cargando.

El contexto usa de `location` los países, las monedas, los tipos de documento y el componente de direcciones, y de `iam` lo que la sesión puede (sus permisos y dónde actúan, si lee las cuentas, si está vinculada a un colaborador activo) y los diálogos de cuenta de los colaboradores ([iam.md](iam.md), §5).

## 6. Estado y pendientes

| Qué | Cuándo |
|---|---|
| Que la cuenta de la aplicación respete Row Level Security en cada entorno. Fuera del perfil `dev`, el arranque lo comprueba: la aplicación no arranca si su cuenta es superusuario o tiene `BYPASSRLS`, ni si Flyway migra con la misma cuenta. En el perfil `dev` la aplicación puede usar una sola cuenta, y nada lo comprueba | En cada entorno, antes de usar roles de empresa o de sede en él |
| El servicio de comandos de los colaboradores pasa de 300 líneas (582): atiende al colaborador y a su período. La skill `backend-module` pide partirlo en piezas con nombre de lo que hacen | Con el próximo cambio que lo toque |
| `termination_reason` sigue candidato (§5.10): la condición que lo admitiría, una pantalla con la historia del colaborador, ya se cumple con el historial de ingresos (F13), que no muestra ningún motivo | El responsable (D-241; pregunta 8) |
| La sonda `e2e-live/organization.live.spec.ts` no cubre las cuentas bancarias ni el historial de ingresos | Con la próxima corrida de las sondas |
| El registro público con perfil (D-32): sigue apagado | Antes de encender el registro público en cualquier entorno (D-83) |
| Una pantalla de solo lectura del registro de vínculos y desvínculos, para el superadministrador; no tiene ruta de lectura | Cuando el responsable la pida; la ruta propuesta es `GET /employees/{id}/account-link-events` |
| El colaborador en un contexto propio (D-92) | Cuando lleguen los contratos laborales, la planilla o la asistencia |
| Colombia y México: las longitudes de sus documentos y el dígito de control del NIT, el RFC y la CURP ([location.md](location.md), sección 6) | Antes de registrar la primera empresa de esos países (D-52) |
| La empresa nombra su propio identificador `publicId`, donde las convenciones proponen `id` (convenciones §13, fila 21) | Cuando el responsable valide la propuesta de convenciones §14.8 |
| Los datos reales: la razón social y el RUC de la empresa, las sedes con su código de establecimiento, las marcas comerciales y los colaboradores | Los entrega el negocio (diccionario §14) |
| `ORGANIZATION_PERSONAL_DATA_KEY` en App Platform, guardada donde se pueda recuperar (ORG-R67) | Antes de desplegar la entrega que trae V138 a V140: sin ella la aplicación no arranca |
| Borrar `employees.document_number` y `employees.phone`, vacías desde V139, con su revisión de V140 | En una entrega posterior, cuando la de D-261 ya esté desplegada (condición de instancia única del plan de migración) |
| Las copias del teléfono y del documento que siguen en claro fuera de `employees`, y cómo se rota la clave | El responsable (diccionario §14, D-261; pregunta 9) |

## 7. Preguntas abiertas

7. **¿Cómo lista el registro público, sin sesión, las sedes y los tipos de documento?** [Convenciones de la API](../convenciones-de-la-api-millennium-mobility.md), §14.16. Fuera del alcance mientras el registro público siga apagado (D-83): se decide antes de encenderlo en un entorno.
8. **¿Se registra el motivo de la salida (`termination_reason`)?** Diccionario §5.10, D-241: la condición que lo admitiría ya se cumple; lo decide el responsable.
9. **¿Qué más se cifra de los datos personales, y quién busca por ellos?** Diccionario §14, D-261: si se cifran también las copias que siguen en claro (el teléfono de «Atención» de las plantillas y sus valores en la bitácora, los documentos de la huella de vínculos), cómo se rota la clave, y si quien no ve los datos personales (D-105) puede buscar un colaborador por su documento.
