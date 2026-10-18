# Revisión · `organization`: seguimiento del cierre, después de integrarlo en `develop`

**Fecha:** 18/10/2026 · **Ramas:** `develop` después de integrar `feature/organization-closure` (backend: merge `96b2d04`; frontend: merge `64e8565`) y `feature/organization-followup`, creada sobre esos merges (backend: 13 commits sobre `96b2d04`, el último `0c3434a`; frontend: 10 commits sobre `64e8565`, el último `b7b1f71`; sin migraciones) · **Alcance:** la auditoría del 18/10/2026 posterior a la integración, sobre todo `organization` y lo que `iam` hace con sus colaboradores, en el backend, el frontend y [contextos/organization.md](../contextos/organization.md), y las correcciones de sus hallazgos.

La auditoría fue de solo lectura: cuatro revisores, cada uno con su lente, y un panel adversarial de 17 verificadores que intentó refutar cada hallazgo antes de confirmarlo. Revisó `develop` después del merge contra el diccionario v1.21, [contextos/organization.md](../contextos/organization.md), la [revisión del cierre](2026-10-18-organization.md) y las skills `backend-module`, `frontend-feature` y `architecture-review`. Las dos decisiones que pedía las tomó el responsable el mismo día: quién lee los colaboradores (D-93) y que reactivar es volver a colgar (D-94), en el diccionario v1.22.

**Veredicto.** Ningún bloqueante. El cierre de los pasos 2 y 3 se sostiene: el paso 2 queda **cerrado e integrado**, y el paso 3 **cerrado salvo la matriz de permisos** (D-68), que llega antes del paso 5. Los hallazgos confirmados se corrigieron en `feature/organization-followup`, que no trae migraciones, así que su integración no necesita inventarios: va a `develop` con su verificación final, que está pendiente (abajo).

---

## Bien hecho

- **El núcleo está sólido.** Ningún verificador encontró un bloqueante, y ningún hallazgo del cierre volvió a aparecer.
- **El esquema es el del diccionario**, columna por columna, en §5.3 y §6.2 a §6.5, con sus llaves de misma empresa y mismo país.
- **Row Level Security está habilitada y forzada en las siete tablas que la llevan**, y la aplicación fija el alcance en un solo lugar, al comenzar cada transacción.
- **Nada se borra.** No hay `DELETE` en la API, y la cuenta de la aplicación no puede borrar en las tablas de `organization`.
- **Los bloqueos no forman ciclos.** Toda operación que lee un padre lo hace en el mismo orden —empresa, sede, marca—, así que dos operaciones simultáneas se esperan y no se traban.
- **Las 56 reglas de [organization.md](../contextos/organization.md) hacen lo que dicen**, y las 144 pruebas que el documento citaba existen.
- **Los inventarios de solo lectura repiten las migraciones carácter por carácter:** `inventory-text-identifiers.sql` quita los mismos separadores que V39, e `inventory-organization-checks.sql` busca las mismas filas que detendrían V40, V41 y V43.

## Hallazgos

### 1. La integración no dejó constancia de los inventarios ni de los commits del merge (aprendizaje)

- **Error:** `feature/organization-closure` se integró en `develop` y se subió, y el plan no anotó los commits del merge, como pedía su paso 2 ("Después del merge se anotan los commits integrados"), ni si se corrieron antes los dos inventarios de solo lectura que el plan ponía como condición. Los documentos seguían diciendo "antes de integrar" y "a falta de la verificación final" cuando la rama ya estaba en `develop`. Lo que protegió la base desplegada fue el diseño de V40, V41 y V43, que se detienen sin cambiar nada si una fila guardada rompe su regla: el despliegue terminó sin errores, así que ninguna la rompió, y V39 no encontró ninguna colisión.
- **Corrección:** esta actualización registra los merges `96b2d04` en el backend y `64e8565` en el frontend, los commits que llegaron después de la cabeza verificada —`840e446` después de `26bc1e6`, `aef5067` después de `0a40211`, los dos de documentación— y el despliegue sin errores que confirmó el responsable. No consta que se corrieran los inventarios antes del push; el responsable lo corrige si los corrió.
- **Próxima vez:** la integración se anota en el mismo momento: los commits del merge, el resultado del despliegue y lo que se corrió antes. Un paso previo que el plan exige deja constancia también cuando todo sale bien, porque la próxima vez puede no salir.

### 2. `organization.md` seguía en borrador, con seis afirmaciones que no coincidían (aprendizaje)

- **Error:** el documento del contexto decía:
  - que la sonda registraba, editaba y daba de baja un colaborador y confirmaba la cuenta de otra persona con la huella, cuando su corrida final no registró la salida ni el regreso, porque la cuenta de verificación no está vinculada, ni el vínculo entre empresas, porque la base tiene una sola empresa ([revisión del cierre](2026-10-18-organization.md), "Sonda contra el backend real");
  - que el paso 2 estaba "a falta de la verificación final", que ya figuraba unas líneas más abajo;
  - que D-89 impide al superadministrador sin vincular desvincular una cuenta, cuando D-89 habla de salidas y regresos, y el desvínculo queda fuera porque D-82 lo hace funcionar como una salida;
  - que toda cuenta existente sin colaborador se vincula solo con la confirmación de un superadministrador, sin la excepción de D-82: el primer superadministrador vincula su propia cuenta sin confirmar;
  - ORG-R49, ORG-R51 y ORG-R56 en ✅ sin la prueba de una de sus partes: la zona del país de la empresa, el cambio de sede de una edición y el orden de los bloqueos;
  - y que `CompanyOption` "figura en el mapa §9", cuando el mapa ya dice que no existe.
- **Corrección:** esta actualización corrige las seis y agrega las pruebas que faltaban (`650690c`, con el reloj fijo de `68cfa4e`). El documento sigue en borrador: lo valida el responsable.
- **Próxima vez:** el documento del contexto se escribe desde la corrida final y el código, no desde el plan. Una regla en ✅ nombra la prueba de cada una de sus partes, y la revisión que cierra un paso compara cada afirmación del documento con lo que la revisión registró.

### 3. El diálogo de nueva ubicación se quedaba en la primera empresa (aprendizaje)

- **Error:** el diálogo calculaba la empresa elegida con un `computed()` que leía el valor de un `FormControl`. Ese valor no es una señal, así que el `computed` no se enteraba de los cambios y se quedaba con la primera empresa: al elegir otra, las sedes que ofrecía, el aviso de empresa inactiva y el país de la dirección seguían siendo los de la primera, y el backend respondía `ADDRESS_DIVISION_OTHER_COUNTRY` a una dirección armada con la división de otro país. La prueba miraba solo el valor inicial.
- **Corrección:** el diálogo lee la empresa de los cambios del control, y el disparador, la marca y el país de la dirección siguen cada cambio; los campos de dirección reciben el país de la empresa nueva y sueltan la división de la anterior. El simulador e2e puede sumar la empresa de Chile a la sesión de un superadministrador para probar el cambio de país (frontend `a9d28ca`).
- **Próxima vez:** un `computed` lee solo señales. El valor de un formulario entra como señal con `toSignal(valueChanges)`, como ya lo hacía el diálogo de colaborador. Y la prueba cambia el valor, no se queda en el inicial.

### 4. O19 se aplazó con un disparo equivocado (aprendizaje)

- **Error:** la revisión del cierre dejó O19 —cualquier rol con alcance en la empresa, también uno de sede o de marca, lee y exporta los colaboradores de toda la empresa, con su documento y su teléfono— "con la matriz, antes del paso 5". Pero los roles de empresa y de sede se otorgan desde el paso 3: desde el día en que se le otorgara su rol, un asesor de ventas de una sede podía leer los documentos y los teléfonos de toda la empresa. El riesgo no esperaba al paso 5.
- **Corrección:** D-93 (diccionario v1.22): los colaboradores los leen solo una cuenta de alcance global y una que administra la empresa; la matriz decide después qué ve cada rol. La lista, su página, su exportación y cada colaborador responden `403 ACCESS_DENIED` a quien ve la empresa sin administrarla, y `404` fuera del alcance (backend `a0e3743`); Colaboradores ofrece solo las empresas que la sesión puede leer, y el `403` tiene su texto (frontend `11d371f`). Es ORG-R57.
- **Próxima vez:** el disparo de un aplazamiento es el hecho que vuelve real el riesgo —aquí, otorgar el rol—, no el siguiente paso del plan. Si ese hecho ya puede ocurrir, el aplazamiento no vale.

### 5. D-84 no decía si reactivar es volver a colgar (aprendizaje)

- **Error:** D-84 fija qué no se cuelga de una empresa, una sede o una marca inactiva, pero no decía nada de reactivar. El código lo dejaba pasar: una sede, una marca o una ubicación de una empresa inactiva se reactivaban, y un colaborador regresaba a una empresa o una sede inactiva, con su cuenta reabierta. Solo estaba decidida la ubicación propia frente a su sede (D-90).
- **Corrección:** D-94 (diccionario v1.22): reactivar es volver a colgar; primero se reactiva la empresa, y desactivar no espera a nada. Las reactivaciones responden `422 COMPANY_INACTIVE`, y el regreso `COMPANY_INACTIVE` o `BRANCH_INACTIVE` antes de que `iam` reabra la cuenta, que sigue cerrada; la empresa y la sede se leen con `FOR SHARE` (backend `6d0d44c`). "Activar" y "Reincorporar colaborador" salen deshabilitados bajo una empresa inactiva, con el motivo (frontend `58f3011`). Es ORG-R58.
- **Próxima vez:** una regla sobre colgar algo de un padre se escribe también para reactivarlo: volver a activar es una forma de crear, y un regreso que devuelve una cuenta es un acceso nuevo.

### 6. Fallas de carga que nadie avisaba, peores al editar (aprendizaje)

- **Error:** si no cargaban los tipos de documento o los países, el formulario de colaborador dejaba el selector vacío sin decir nada. Al editar era peor: rotulaba "(inactivo)" el tipo que el colaborador ya tenía, solo porque no lo encontraba en la lista que había fallado. En Sedes y Ubicaciones, la dirección pedía "Seleccione primero el país." aunque la empresa ya estaba elegida, y el diálogo de empresa mostraba el error que había dejado otro formulario.
- **Corrección:** los formularios dicen qué falló, con "Reintentar"; un tipo se rotula "(inactivo)" solo cuando se sabe que lo está; la dirección dice por qué espera a su país, y una dirección cambiada no se guarda sin sus reglas; y un formulario ya no hereda el error de otro (frontend `22ab06c`).
- **Próxima vez:** una lectura que puede fallar dice que falló y ofrece reintentar. Un formulario de edición no deduce nada de una lista que no cargó.

### 7. Códigos genéricos en la API (error)

- **Error:** una edición que dejaba la fecha de ingreso de un colaborador que salió después de su salida la rechazaba el dominio, con una regla genérica y sin campo. Un número de documento hecho solo de separadores (`-.-`) respondía `422 BUSINESS_RULE_VIOLATION` en el colaborador, y en la empresa la respuesta dependía del tipo: `422` o un `400` sin campo, distinta con el RUC.
- **Corrección:** `422 EMPLOYEE_TERMINATION_BEFORE_HIRE` sobre `hireDate`, revisado antes de tocar el agregado (backend `0e8f79e`); `400 VALIDATION_ERROR` sobre `documentNumber` y sobre `taxId`, medido sobre el número limpio con cualquier tipo, en el alta, la edición y la corrección (backend `befe8de`, `0c3434a`); y su texto en la pantalla (frontend `58f3011`). Las convenciones lo registran en §7.2.
- **Próxima vez:** ya está en la [revisión del cierre](2026-10-18-organization.md), hallazgo 11, y en las de `location`: una causa, un código y su campo. Un identificador se valida sobre el valor normalizado, igual para todos los tipos.

### 8. El simulador e2e aceptaba lo que el backend rechaza, y una prueba usaba un rol que no existe (aprendizaje)

- **Error:** el simulador de las pruebas de extremo a extremo dejaba escribir empresas, sedes, marcas y ubicaciones a cualquier sesión, conservaba en el `PUT` de un colaborador los campos que no llegaban, aceptaba una foto de más de 500 caracteres y un documento hecho solo de separadores, servía los colaboradores a cualquier rol y no seguía el orden del regreso. Una prueba e2e podía pasar contra un comportamiento que el backend no tiene. Además, una prueba de la vista de ubicaciones usaba el rol `OPERADOR`, que no existe; la revisión del cierre ya había encontrado `ROLE_OPERATOR` en el backend (hallazgo 12).
- **Corrección:** el simulador responde como el backend: `403` a las escrituras de `organization` sin `SUPERADMIN`, el `PUT` que reemplaza los campos, el límite de la foto, los separadores, las lecturas de D-93 y el orden del regreso (frontend `b447d62`, `c5253e5`, `b7b1f71`). La prueba usa `OPERACIONES` (frontend `a8502c8`).
- **Próxima vez:** el simulador repite los rechazos del backend, no solo sus respuestas felices: cada regla nueva del backend llega al simulador en la misma rama. Un rol en una prueba es uno de los roles sembrados.

### 9. Menores (aprendizaje)

- **Error y corrección, uno por uno:**
  - las exportaciones de colaboradores y ubicaciones nombraban la sede solo por su nombre, que puede repetirse en la empresa (V20): ahora llevan también "Código de sede" (backend `f449512`, ORG-R59);
  - `GetCompanyByCountryIdQuery` leía una empresa sin el filtro de alcance, y nadie la llamaba, como otras tres lecturas del repositorio: se fueron (backend `fe0f263`);
  - `hand-over-database.sql` no le entregaba a `erp_mm_owner` las funciones de Row Level Security (`access_scope_reaches(bigint)`, `access_scope_user_id()` y `access_scope_is_set()`), que quedaban a nombre de quien corrió las migraciones: ahora las entrega, y su comprobación final rechaza una función del esquema que sea de otra cuenta (backend `c41fecd`);
  - la vista de ubicaciones leía las sedes también para el administrador de empresa, que solo lee la lista, y le avisaba sus fallas por un "Activar" que no tiene (frontend `1c2db70`);
  - el código de una ubicación escrito en minúsculas frenaba "Crear" mientras el campo tenía el foco; ahora se acepta y se normaliza al guardar (frontend `0fe6efe`);
  - faltaban las pruebas de ORG-R49 en la zona del país de la empresa, de ORG-R51 en el cambio de sede y de ORG-R56 en el orden de los bloqueos (backend `650690c`, con el reloj que las pruebas fijan, `68cfa4e`);
  - la mitad de O46 que había quedado abierta: las vistas de empresa, sede y marca envolvían su agregado; ahora tienen campos propios, cada una con su fábrica —`CommercialBrandViewFactory` es nueva—, y el JSON no cambió (backend `9b3b0c1`, `0758640`, `200d56b`).
- **Próxima vez:** cada uno ya tenía su regla en las skills o en las revisiones anteriores: una referencia se exporta por algo que no se repite, una lectura sin alcance no existe aunque nadie la llame, el traspaso de la base entrega todo lo que crearon las migraciones, y una pantalla lee solo lo que su usuario puede usar.

## Un error de esta auditoría

- **Error:** el coordinador corrió tres pruebas de Spring sin las variables de la base de verificación, y se conectaron a la base de desarrollo, `erp_mm_db`. Eran `@Transactional` y se deshicieron al terminar; Flyway no aplicó nada, y solo se consumieron valores de las secuencias.
- **Corrección:** no hay nada que deshacer en la base: las transacciones se deshicieron, y los valores de secuencia consumidos solo dejan huecos en la numeración.
- **Próxima vez:** `DATABASE_NAME=erp_mm_db_verify` antes de cualquier prueba que arranque Spring, también las que parecen puras.

## Propuestas para el diccionario

Registradas en el diccionario v1.22 el 18/10/2026 (`03aa892`, `44919cb`), con su texto pendiente de la validación del responsable: quién lee los colaboradores, la lista, cada uno y su exportación (D-93, §5.3, §5.13), y que reactivar es volver a colgar, también el regreso (D-94, §5.3, §6.2). Los códigos nuevos —`403 ACCESS_DENIED` en las lecturas de colaboradores, `COMPANY_INACTIVE` en las reactivaciones, el orden del regreso, `EMPLOYEE_TERMINATION_BEFORE_HIRE` en la edición y el `400` de los números hechos solo de separadores— son convenciones de la API y quedan en sus §7.2.

## Lo que queda abierto, con su disparo

| Qué | Cuándo |
|---|---|
| `roles.company_id`: los roles propios de una empresa, con la unicidad de su código y quién los crea (D-81). Llega con `permissions` y `role_permissions` | Con la matriz de permisos (D-68), antes del paso 5 |
| La matriz de permisos (D-68): qué hace cada rol más allá de los que manejan cuentas y administran empresas, qué lee un rol global y la marca en la lectura | Antes del paso 5; es condición para empezarlo |
| Qué ve de los colaboradores cada rol, por ejemplo los de su sede. O19 quedó resuelta por D-93 (ORG-R57) | Con la matriz, antes del paso 5 (D-68, §5.13) |
| Que la cuenta de la aplicación respete Row Level Security en cada entorno: `dev` usa `postgres` por omisión (O16) | En cada entorno, antes de usar roles de empresa o de sede en él |
| El regreso reemplaza la fecha de ingreso y pierde el período anterior (O08; diccionario §14) | Antes de cargar los colaboradores reales |
| La regla de §8.1 para la marca y la ubicación de un vehículo, con `locations_id_company_uq` (V41), y la lectura en lote de las ubicaciones de una empresa por la fachada (G3.1, G3.2) | Con el paso 5 |
| Las fechas en pantalla según §2.6 (O70) | Antes de la próxima pantalla con fechas (pasos 4 y 5) |
| El registro público con perfil (D-32) | Fuera del paso 3 (D-83): antes de encenderlo en un entorno |
| Una pantalla que lea `employee_account_link_events`, solo para el superadministrador | Cuando el responsable la pida; hoy no tiene ruta de lectura |
| El colaborador en un contexto propio (D-92) | Cuando lleguen los contratos laborales, la planilla o la asistencia |
| Los casos de la sonda de `organization` que no corrieron: el vínculo entre empresas de D-82, la frontera del administrador de empresa, y la salida y el regreso | Antes del paso 5, sobre `erp_mm_db_live` con una segunda empresa y la cuenta de verificación vinculada a un colaborador. La base la crea el responsable |
| Colombia y México: las longitudes de sus documentos y el dígito de control del NIT, el RFC y la CURP (O68) | Antes de registrar la primera empresa de esos países (D-52) |
| Acotar `addresses` por empresa con Row Level Security (O69) | En el paso 7 (D-80) |
| La empresa todavía nombra su propio identificador `publicId`, donde las convenciones proponen `id` (convenciones §13, fila 21) | Cuando el responsable valide la propuesta de convenciones §14.8 |
| `CompanySummary` lleva campos que `iam` no lee, y el `code` de `CompanyUnitOption` no tiene consumidor | **(propuesta)** Con el próximo cambio de cada fachada |

Es la tabla de [organization.md](../contextos/organization.md), §6 b. Lo abierto de `iam` sigue en [iam.md](../contextos/iam.md), §6 b.

## Verificación

Hasta ahora, sobre `feature/organization-followup`: la suite completa del backend, 1 616 pruebas sin fallos, antes de `0c3434a`; después de `0c3434a`, las de `organization`, `iam` y `ModularityTests`, 913 sin fallos. Las unitarias del frontend, 1 309, todas en verde. La suite e2e, 656 pasan y 22 se omiten por proyecto; después de `b7b1f71`, 276 pasan en los specs afectados.

**Verificación final de `feature/organization-followup`: pendiente.**
