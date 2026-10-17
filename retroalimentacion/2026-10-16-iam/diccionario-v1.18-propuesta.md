# Diccionario de datos v1.18 — propuesta de cambios

**Preparada:** 16/10/2026 · **Revisada:** 16/10/2026, con las aclaraciones C4-1 y C4-2 de r4 y la ronda 5 · **Estado:** para tu validación. Este archivo no modifica el diccionario: solo tú lo cambias.
**Fuentes:** diccionario v1.17; los cinco contratos del modelo de acceso (`access-model-spec.md`, `-r2`, `-r3`, `-r4` con sus aclaraciones C4-1 y C4-2, y `-r5` con las decisiones A a G que aprobaste el 16/10/2026; manda el posterior); el código de `mm-acc-be` en `feature/access-model`, leído solo para que §3 y §5.13 digan la verdad.

**Cómo leerlo.** Las entradas siguen el orden del diccionario. Cada una trae el texto **actual** copiado tal cual, el texto **propuesto** listo para pegar y el **porqué** (la decisión del contrato o el hallazgo). Las marcadas *(opcional)* no las exigen los contratos: corrigen algo vecino que quedaría desactualizado. Las decisiones nuevas van de D-55 a D-72 (§3 de este archivo). La revisión agregó dos entradas, C-37 y C-38: van en su lugar del diccionario, con el número que sigue, para no cambiar el de las demás.

---

## 1. Qué cambia y por qué

- **Nace un rol nuevo: el administrador de accesos** (`ADMIN_ACCESOS`, pensado para TI). Es uno para todo el grupo, no uno por empresa. Da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas en todas las empresas. Tiene tres límites: nunca actúa sobre su propia cuenta, nunca otorga ni retira los tres roles reservados al superadministrador (`SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`) y nunca toca una cuenta que tenga alguno de los tres, una **cuenta protegida**. Esas cuentas las administra solo el superadministrador.
- **Separación de funciones.** El administrador de empresa se queda con su gente: registra colaboradores y registra sus salidas y regresos. Deja de manejar cuentas. El administrador de accesos maneja cuentas y no registra colaboradores. Así nadie puede inventar un colaborador y darle acceso. Para que la separación no se pueda saltar, una misma cuenta nunca tiene a la vez los dos roles, y el administrador de accesos no puede tomar la cuenta de un administrador de empresa, porque es protegida. El superadministrador conserva todo.
- **El diccionario gana una excepción con nombre** a la regla "nadie otorga un rol con más permisos que los suyos": el administrador de accesos asigna roles que no tiene, siempre dentro de sus tres límites.
- **Nadie actúa sobre lo suyo.** Ninguna cuenta, tampoco la de un superadministrador, se otorga roles, se bloquea ni se restablece la contraseña a sí misma, y nadie registra su propia salida ni su propio regreso. La única cuenta que se vincula a sí misma es la del primer superadministrador, al arrancar.
- **Un rol privilegiado exige contraseña propia.** `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA` se otorgan solo a una cuenta que ya cambió su contraseña provisional: quien la entregó nunca termina con una cuenta privilegiada en sus manos.
- **Los bloqueos de cuenta tienen reglas nuevas.** Corren en dos líneas de tiempo: la laboral (vacaciones, licencia, baja) y la de seguridad. Un bloqueo de seguridad puede cruzarse con los otros, y la salida y el regreso nunca lo tocan. Un bloqueo nunca empieza en el pasado, se cierra siempre "ahora", y cerrar uno programado lo cancela. La `baja` es de la salida: nadie la abre ni la cierra a mano.
- **La salida de un colaborador** no puede tener fecha posterior a hoy, y siempre quita el acceso, también el de una cuenta protegida: el único freno es el último superadministrador. El diccionario dice ahora exactamente qué le hace a la cuenta: retira roles, cierra la cuenta si era el último colaborador, termina o cancela las vacaciones y licencias, y no toca la seguridad. También dice qué hace el regreso, que tampoco puede tener fecha posterior a hoy.
- **Restablecer la contraseña de otra persona** entra al diccionario: quién puede hacerlo, que genera una clave provisional y que cierra las sesiones.
- **La caducidad pasa sola.** Un proceso automático cierra los accesos vencidos, y el acceso deja de valer exactamente al vencer.
- **La protección del último superadministrador mira hacia adelante.** Un superadministrador con caducidad o con vacaciones programadas no cuenta como respaldo. Además, dos solicitudes simultáneas no pueden dejar al sistema sin ninguno.
- **La frontera entre empresas la defiende también la base de datos** (Row Level Security), no solo el programa.
- **La matriz de permisos tiene fecha.** Debe llegar antes del paso 5, el primero con operaciones de negocio. Hasta entonces, los dos roles globales leen todas las empresas.
- **§3 y §5.13 se actualizan** a lo que construyó el paso 3 y a lo que todavía falta.

**Un aviso de tiempo.** Al 16/10/2026 la ronda 4, con sus aclaraciones C4-1 y C4-2, ya está en el código (V30 a V32). La ronda 5 se está construyendo en paralelo. Los textos de §3 y §5.13 describen el estado una vez integrada la ronda 5. Conviene aplicarlos cuando eso ocurra.

---

## 2. Cambios, en el orden del diccionario

### C-1 · Encabezado (línea 3)

**Actual**
```md
**Versión:** 1.17 · **Motor:** PostgreSQL 18 · **Estado:** módulo de stock completo. Bloques 1 a 11 cerrados; ventas y contratos fuera del alcance actual. 49 tablas más `role_permissions`, la tabla de unión de roles y permisos
```

**Propuesto**
```md
**Versión:** 1.18 · **Motor:** PostgreSQL 18 · **Estado:** módulo de stock completo. Bloques 1 a 11 cerrados; ventas y contratos fuera del alcance actual. 49 tablas más `role_permissions`, la tabla de unión de roles y permisos
```

**Por qué.** Así se registra cada versión (v1.16 y v1.17 cambiaron solo esta línea y agregaron sus decisiones). No hay tablas nuevas: `user_access_suspensions` ya estaba contada.

---

### C-2 · §2.2 Estructura, fila "Intervalos sin solapamiento"

**Actual**
```md
| Intervalos sin solapamiento: `daterange` o `tstzrange` + `WITHOUT OVERLAPS` (PG 18) | Impide dos períodos cruzados incluso en el pasado |
```

**Propuesto**
```md
| Intervalos sin solapamiento: `daterange` o `tstzrange` + `WITHOUT OVERLAPS` (PG 18). Cuando hay filas que no deben contar —un bloqueo cancelado— o la regla compara un dato que se deduce de otro —la línea de tiempo de un bloqueo—, una exclusión `EXCLUDE USING gist` con su condición, que necesita `btree_gist` (D-67) | Impide dos períodos cruzados incluso en el pasado. `WITHOUT OVERLAPS` no admite dejar filas afuera ni comparar una expresión; la exclusión sí, y protege lo mismo |
```

**Por qué.** r2 R2 reemplazó `WITHOUT OVERLAPS` por una exclusión que deja afuera los bloqueos cancelados. r4 D4-9 hace que la exclusión compare la línea de tiempo, que se deduce del motivo y no se guarda (§2.2: "Los derivados no se persisten"). V28 ya usa `EXCLUDE`. Es una divergencia deliberada de la regla, así que va escrita.

---

### C-3 · §2.5 Reglas, viñeta de los usuarios (línea 107)

**Actual**
```md
- Los usuarios no pertenecen a una empresa. El administrador de una empresa ve a quienes tienen alguna asignación vigente en ella.
```

**Propuesto**
```md
- Los usuarios no pertenecen a una empresa. El administrador de una empresa ve las cuentas vinculadas a colaboradores de esa empresa, con los roles vigentes que tienen en ella y los de alcance sistema, pero no las modifica: dar acceso, otorgar y retirar roles, bloquear cuentas y restablecer contraseñas es trabajo del administrador de accesos (D-57).
```

**Por qué.** r2 R3: `GET /users?companyId` devuelve las cuentas que tienen un colaborador en esa empresa, con los roles de esa empresa y los de alcance sistema. La tabla de r4 lo deja de solo lectura para el administrador de empresa (D4-4). El código incluye también a los colaboradores que ya salieron (`findAllByCompanyIdAndUserIdIsNotNull`): ver §5 de este archivo, punto 6.

---

### C-4 · §2.5 Reglas, viñeta nueva sobre Row Level Security (insertar después de "El acceso total es una excepción explícita…", línea 110)

**Actual** (la viñeta después de la cual se inserta; no cambia)
```md
- El acceso total es una excepción explícita evaluada con las mismas reglas, no una cuenta que se saltea el control.
```

**Propuesto** (insertar a continuación)
```md
- **La frontera entre empresas la defiende también la base de datos** (D-66). La aplicación filtra cada consulta por las empresas que alcanza quien la hace. Además, en cada tabla que lleva `company_id` —y en `companies`, por su propio `id`—, la base deja ver y escribir solo las filas de esas empresas. Si algún día una consulta olvida su filtro, devuelve menos filas, nunca las de otra empresa. Un rol de alcance sistema ve todas; una operación sin alcance no ve ninguna.

**Nota técnica.** Es Row Level Security de PostgreSQL, habilitada y forzada (`ENABLE` y `FORCE ROW LEVEL SECURITY`) para que tampoco el dueño de las tablas la saltee. Hoy cubre `companies`, `commercial_brands`, `branches`, `locations`, `employees` y `user_roles`, y cubrirá cada tabla con `company_id` que se cree. Hay dos cuentas de base de datos: la dueña, que solo aplica las migraciones, y la de la aplicación, que no es dueña de nada, no puede saltear la regla y no tiene permiso de borrar. Al comenzar cada transacción —nunca al abrir la conexión—, la aplicación fija desde un solo lugar, con `set_config(…, true)`, quién actúa y qué alcanza: `app.access_scope` (`SYSTEM` o `SCOPED`), `app.company_ids` y `app.user_id`. Un valor que falta no ve nada. El arranque y los procesos automáticos actúan con alcance `SYSTEM` de forma explícita, igual que firman como `SYSTEM` (D-42). `user_roles` le muestra además a cada cuenta sus propias asignaciones y las de alcance sistema, para que pueda calcularse su alcance. Las pocas comprobaciones que necesitan ver más allá de las empresas de quien opera —si la cuenta tiene un colaborador activo en otra empresa, o cuántos superadministradores quedan (§5.9)— se hacen con una elevación a `SYSTEM` acotada a esa comprobación, que solo devuelve el dato.
```

**Por qué.** r2 R1 puso la Row Level Security en el alcance del paso 3 (reglas 5.1 a 5.3 y 5.7 de la skill del backend), y V29 la construyó. §2.5 es el lugar del diccionario para la separación por empresa, y §1 pide que se indique cuando una regla de negocio determina una restricción de base de datos. La viñeta anterior ("El acceso total es una excepción explícita…") es exactamente el alcance `SYSTEM` de la política. Las elevaciones quedan reducidas a las que conservan r4 y r5: si la cuenta trabaja para otra empresa, cuando la salida la registra un administrador de empresa, y el conteo de superadministradores. r5 C quita la de la cuenta protegida, porque la salida ya no pregunta por ella.

---

### C-5 · §3 Orden de implementación, fila del paso 3

**Actual**
```md
| 3 | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | Colaboradores; asignaciones por empresa, sede y marca; bloqueos de cuenta; permisos; dar acceso; registro público con perfil | Colaboradores reales y la matriz de permisos por rol | Pendiente |
```

**Propuesto**
```md
| 3 | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | Colaboradores; asignaciones por empresa, sede y marca; bloqueos de cuenta; permisos; dar acceso; registro público con perfil | Colaboradores reales y la matriz de permisos por rol | Construido, salvo los permisos y el registro público con perfil: colaboradores con su salida y su regreso, dar acceso, asignaciones por empresa, sede y marca con su caducidad, bloqueos en dos líneas de tiempo, la separación de funciones entre quien registra colaboradores y quien da acceso (D-57) y la frontera entre empresas en la base (D-66). Lo que falta está en §5.13 |
```

**Por qué.** Refleja lo construido en `feature/access-model` (V26 a V32) más la ronda 5. La fila del paso 2 ("falta `roles.company_id`") sigue siendo cierta y no cambia.

---

### C-6 · §3, cuarto párrafo de "Los cuatro ajustes" (ajuste 3)

**Actual**
```md
3. **El bloque 2 se termina en el paso 3.** Los permisos hacen falta antes de las operaciones de negocio de los bloques 6 y 7, como registrar una factura o dar de alta una unidad.
```

**Propuesto**
```md
3. **El bloque 2 se termina cuando llegue la matriz de permisos.** El paso 3 se construyó sin `permissions` ni `role_permissions` porque falta el dato: qué permisos lleva cada rol. Mientras tanto, quién otorga qué lo fija una regla provisional (§5.7, §5.13). Un rol de alcance sistema —el superadministrador o el administrador de accesos— lee los datos de todas las empresas: hoy empresas, marcas, sedes, ubicaciones y colaboradores, que el administrador de accesos necesita para dar acceso. La matriz llega antes del paso 5, el primero con operaciones de negocio, como dar de alta una unidad, y con ella se acota también lo que lee un rol global (D-68).
```

**Por qué.** r4 D4-7: la matriz se debe antes del paso 7, y hasta entonces un rol global lee todas las empresas. El ajuste actual dice que el bloque 2 se termina en el paso 3, lo que ya no es cierto. r5 fija la matriz antes del paso 5, según este mismo ajuste ("Not changed, recorded", sobre la opción del menú Roles y permisos), lo que cumple también D4-7: ver §5 de este archivo, punto 3.

---

### C-7 · §5.2 La secuencia real de arranque, pasos 7 y 8

**Actual**
```md
| 7 | Registra al administrador de cada empresa como colaborador, le da acceso y le asigna el rol `ADMIN_EMPRESA` sobre esa empresa | Usuario |
| 8 | El administrador de cada empresa registra a sus colaboradores y da acceso a quienes usan el sistema, con los roles que corresponden a su puesto y su sede | Usuario |
```

**Propuesto**
```md
| 7 | Registra como colaboradores al administrador de cada empresa y al administrador de accesos, que son personas distintas, y les da acceso. Cuando cada uno cambió su contraseña provisional, le asigna su rol: `ADMIN_EMPRESA` sobre su empresa o `ADMIN_ACCESOS`, que alcanza todo el sistema | Usuario |
| 8 | El administrador de cada empresa registra a sus colaboradores, con su puesto y su sede | Usuario |
| 9 | El administrador de accesos da acceso a quienes usan el sistema y les asigna los roles que corresponden a su puesto y su sede | Usuario |
```

**Por qué.** r4 D4-1 (el rol nuevo), D4-4 (el administrador de empresa registra a su gente y no da acceso) y D4-6 (todo actor es colaborador: el administrador de accesos necesita un colaborador activo en alguna empresa para tener un rol de alcance sistema, §5.7). r4 D4-2 y r5 A: `ADMIN_ACCESOS` y `ADMIN_EMPRESA` los otorga solo un superadministrador. r5 D: una cuenta nunca tiene los dos a la vez (D-71). r5 E: un rol privilegiado se otorga solo a una cuenta que ya cambió su contraseña provisional (D-72), por eso el paso 7 espera a que cada uno entre por primera vez.

---

### C-8 · §5.2, párrafo nuevo (insertar después de "Los pasos 1 a 3 no los puede ejecutar nadie desde la aplicación…")

**Actual** (el párrafo después del cual se inserta; no cambia)
```md
Los pasos 1 a 3 no los puede ejecutar nadie desde la aplicación —no hay con qué iniciar sesión todavía—. Esto no es un rodeo: es la única forma de resolver que el primer usuario no tiene creador.
```

**Propuesto** (insertar a continuación)
```md
**Los pasos 8 y 9 los hacen personas distintas, a propósito** (D-57). Quien registra a los colaboradores no les da acceso, y quien da acceso no registra colaboradores: así nadie puede inventar un colaborador y darle acceso. Por eso una misma cuenta nunca tiene a la vez `ADMIN_EMPRESA` y `ADMIN_ACCESOS` (D-71). El superadministrador puede hacer ambas cosas. Es el único que otorga `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, y el único que actúa sobre las cuentas que tienen alguno de esos tres roles (§5.5).
```

**Por qué.** r4 D4-4 y D4-5, con la razón que da el propio contrato: "One person must not be able to invent an employee and give it access". r5 A (`ADMIN_EMPRESA` es un rol reservado), r5 B (la cuenta de un administrador de empresa también es protegida) y r5 D (separación estática), con la razón de r5 B: sin ella, el administrador de accesos restablece la contraseña de un administrador de empresa, entra como él, inventa un colaborador y le da acceso.

---

### C-37 · §5.2, párrafo "Es la única persona que nace sin perfil de colaborador" *(agregada en la revisión)*

**Actual**
```md
**Es la única persona que nace sin perfil de colaborador**, porque todavía no existe ninguna empresa donde registrarla. Mientras no se vincule a su perfil (paso 6) puede configurar el sistema —empresas, sedes, catálogos y su propio perfil—, pero no puede dar acceso a nadie ni otorgar roles. Así, desde el primer acceso que se otorga, cada acción queda atribuida a una persona con nombre, documento y puesto.
```

**Propuesto**
```md
**Es la única persona que nace sin perfil de colaborador**, porque todavía no existe ninguna empresa donde registrarla. Mientras no se vincule a su perfil (paso 6) puede configurar el sistema —empresas, sedes, catálogos y su propio perfil—, pero no puede dar acceso a nadie ni otorgar roles. Así, desde el primer acceso que se otorga, cada acción queda atribuida a una persona con nombre, documento y puesto. Vincularse a sí mismo es solo de ese paso: lo hace el superadministrador mientras no tiene un colaborador activo. Después ninguna cuenta se vincula a sí misma, tampoco a un colaborador de otra empresa, porque sería darse acceso a sí misma (D-70).
```

**Por qué.** r5 T8: un superadministrador vincula su propia cuenta solo mientras no tiene un colaborador activo; si no, `403`. Es la única excepción a C4-1 (nadie actúa sobre su propia cuenta), y el paso 6 es su razón.

---

### C-9 · §5.2 Registro público, tres viñetas

**Actual**
```md
- **Si su documento ya está registrado en esa empresa, el registro se rechaza.** Esa persona ya tiene perfil, y el acceso se lo da un administrador desde ahí. Si el registro pudiera vincularse a un perfil existente, cualquiera que conozca un número de documento podría apropiarse del perfil de otra persona.
- **Si el correo ya tiene cuenta, el registro se rechaza.** Esa persona ya tiene acceso por otra empresa del grupo: un administrador de esta empresa la registra como colaboradora y vincula el perfil a su cuenta.
- **Nace sin roles.** Registrarse no da acceso a nada: los roles se los asigna un administrador después, con las mismas reglas de §5.5.
```

**Propuesto**
```md
- **Si su documento ya está registrado en esa empresa, el registro se rechaza.** Esa persona ya tiene perfil, y el acceso se lo da el administrador de accesos desde ahí. Si el registro pudiera vincularse a un perfil existente, cualquiera que conozca un número de documento podría apropiarse del perfil de otra persona.
- **Si el correo ya tiene cuenta, el registro se rechaza.** Esa persona ya tiene acceso por otra empresa del grupo: el administrador de esta empresa la registra como colaboradora y el administrador de accesos vincula el perfil a su cuenta (§5.3).
- **Nace sin roles.** Registrarse no da acceso a nada: los roles se los asigna después el administrador de accesos o un superadministrador, con las reglas de §5.5 y §5.7.
```

**Por qué.** r4 D4-4: el administrador de empresa ya no da acceso ni vincula cuentas. La tabla de r4 deja `POST /access-grants` y los roles al superadministrador y al administrador de accesos.

---

### C-10 · §5.3 Reglas de negocio, viñeta "Toda cuenta pertenece a un colaborador" y viñeta nueva "Dar acceso"

**Actual**
```md
- **Toda cuenta pertenece a un colaborador** (decisión D-38). Las cuentas no se crean solas. Hay dos caminos, y en ambos la cuenta queda vinculada a su colaborador en la misma operación: un administrador **da acceso** a un colaborador ya registrado, o la persona usa el **registro público**, si está encendido, que crea el colaborador y la cuenta juntos (§5.2). Si la persona ya tiene cuenta porque trabaja para otra empresa del grupo, dar acceso vincula el nuevo colaborador a esa misma cuenta en vez de crear otra. Un colaborador puede existir sin cuenta y recibir acceso cuando lo necesite. Las únicas cuentas sin colaborador son `SYSTEM`, que no es una persona, y el primer superadministrador hasta que se vincula (§5.2).
```

**Propuesto**
```md
- **Toda cuenta pertenece a un colaborador** (decisión D-38). Las cuentas no se crean solas. Hay dos caminos, y en ambos la cuenta queda vinculada a su colaborador en la misma operación: el administrador de accesos o un superadministrador **da acceso** a un colaborador ya registrado, o la persona usa el **registro público**, si está encendido, que crea el colaborador y la cuenta juntos (§5.2). Si la persona ya tiene cuenta porque trabaja para otra empresa del grupo, dar acceso vincula el nuevo colaborador a esa misma cuenta en vez de crear otra. Un colaborador puede existir sin cuenta y recibir acceso cuando lo necesite. Las únicas cuentas sin colaborador son `SYSTEM`, que no es una persona, y el primer superadministrador hasta que se vincula (§5.2).
- **Dar acceso crea la cuenta o la vincula, y nada más** (D-58). Una cuenta nueva recibe una contraseña provisional que genera el sistema, se muestra una sola vez y debe cambiarse en el primer ingreso (§5.4). Vincular una cuenta que ya existía nunca cambia sus roles ni su contraseña: la persona sigue entrando con la suya y conserva los roles que tiene en otras empresas; los de esta empresa se asignan después (§5.7). El administrador de accesos vincula cuentas de cualquier empresa del grupo, salvo una que tenga `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente: esa solo la vincula un superadministrador (§5.5). `SYSTEM` no se vincula a nadie. La única cuenta que se vincula a sí misma es la del primer superadministrador, y solo mientras no tiene un colaborador activo (§5.2, paso 6; D-70). Si la cuenta estaba cerrada por una salida, vincularla cierra el bloqueo `baja`, porque vuelve a tener un colaborador activo (§5.4).
```

**Por qué.** r1 D2: dar acceso nunca cambia roles ni contraseña. Era la causa A de la auditoría del 14/10/2026: dar acceso reemplazaba los roles de una cuenta ajena. r4 D4-2 y su tabla: el administrador de accesos vincula en cualquier empresa, salvo cuentas protegidas, y desaparece la regla de r3 B3 que exigía un superadministrador para vincular una cuenta ajena. r5 B: la cuenta de un administrador de empresa también es protegida. r1 §3: `SYSTEM` no se vincula. r1 §4 y r5 T8: el primer superadministrador se vincula a sí mismo, y solo mientras no tiene colaborador. r3 B1 y r4 D4-9: vincular cierra solo la `baja`.

---

### C-11 · §5.3 Reglas de negocio, viñeta "Cuando un colaborador sale" (se reemplaza por cuatro viñetas)

**Actual**
```md
- **Cuando un colaborador sale**, se registra su fecha de salida y en la misma operación se retiran los roles de su cuenta en esa empresa, con motivo `baja_colaborador` (§5.7). Si a la cuenta no le queda ningún colaborador activo, se bloquea con motivo `baja` (§5.4), lo que cierra sus sesiones. Quien trabaja para dos empresas y deja una conserva su cuenta y sus roles en la otra. El sistema impide registrar la salida si con ella no quedaría ningún superadministrador activo (§5.9).
```

**Propuesto**
```md
- **Cuando un colaborador sale**, se registra su fecha de salida, que no puede ser posterior a hoy (D-60). La salida se registra cuando ocurre, no por adelantado, porque retirar los roles antes de tiempo dejaría sin acceso a alguien que todavía trabaja. Hoy se calcula como fija §2.6: en la zona de la sede del colaborador; sin sede, en la del país de su empresa. En la misma operación se retiran los roles de su cuenta en esa empresa, con motivo `baja_colaborador` (§5.7). Quien trabaja para dos empresas y deja una conserva su cuenta y sus roles en la otra.
- **Si a la cuenta no le queda ningún colaborador activo, la salida la cierra** desde el momento en que se registra, aunque la fecha de salida sea anterior, porque un bloqueo no se registra en el pasado (§5.4). La cuenta se bloquea con motivo `baja`, salvo que ya tenga una `baja` en curso. Se retiran sus demás roles con motivo `baja_cuenta` (§5.9) y se cierran sus sesiones. Si la persona estaba de vacaciones o de licencia, ese bloqueo termina en ese momento, y los bloqueos programados de la línea laboral se cancelan. Un bloqueo por `seguridad`, en curso o programado, no se toca: corre en su propia línea de tiempo (§5.4).
- **La salida siempre quita el acceso** (D-60), también el de una cuenta protegida (§5.5): quien se fue no conserva el acceso mientras alguien con más permisos se entera. Una salida equivocada se corrige con un regreso. El único freno es la protección del último superadministrador (§5.9).
- **Registran y modifican colaboradores**, y registran sus salidas y regresos, el superadministrador y el administrador de la empresa del colaborador (D-57). El administrador de accesos no lo hace: quien da acceso no puede también crear al colaborador que lo recibe. Nadie registra su propia salida ni su propio regreso (D-70).
- **Cuando un colaborador regresa**, se registra su nueva fecha de ingreso, que no puede ser anterior a la de salida ni posterior a hoy, calculado igual que para la salida. Si su cuenta estaba cerrada por la salida, en la misma operación se cierra el bloqueo `baja`, y la persona vuelve a entrar con la misma cuenta y su historial completo. Un bloqueo por `seguridad` en curso sigue hasta que alguien lo levante. Los roles no vuelven solos: se asignan de nuevo (§5.9).
```

**Por qué.**
- La fecha no posterior a hoy es una **[DECISION]** de r1 §5, con su razón.
- Que la `baja` empiece "ahora" sale de r1 §5 ("open a `baja` suspension now") y de r3 B9.
- La cancelación de lo programado sale de r2 R2, acotada por r4 D4-9 y por C4-2: se cancela todo lo programado de la línea laboral —vacaciones, licencia y una `baja` que hubiera quedado de antes—, seguridad nunca, y no se abre una segunda `baja`.
- Quién registra salidas y regresos sale de r4 D4-4 y su tabla. Nadie registra los suyos: r5 F, la regla de dos personas de NIST AC-5 aplicada al ciclo de la propia persona.
- r5 C quita la regla de r4 D4-4 que exigía un superadministrador para la salida que cierra una cuenta protegida, con su razón: NIST PS-4 e ISO 27001 A.5.18 piden quitar el acceso apenas la persona se va.
- El regreso cierra solo la `baja` (r1 §5, r4 D4-9). Los roles no se restauran (r1 §5, §5.9). La fecha del regreso no puede ser anterior a la salida (r3 B12) ni posterior a hoy, en la misma zona que la salida (r5 G).
- La zona de hoy: para un colaborador nunca se llega a `America/Lima`, porque siempre tiene empresa y `countries.default_timezone` es obligatorio. Por eso cito §2.6 en vez de escribir un tercer escalón. Que la API entregue esa zona con cada colaborador (r5 T4, `EmployeeResource.timeZone`) es una forma de mostrar §2.6, no una regla nueva: no cambia el diccionario.
- Lo que el regreso hace con el período anterior queda abierto: ver §5 de este archivo, punto 4.

---

### C-12 · §5.4 `users`, fila `must_change_password`

**Actual**
```md
| `must_change_password` | BOOLEAN | No | Obliga a cambiar la contraseña la primera vez que entra | Cuando se crea una cuenta se entrega una clave provisional. Sin esta marca, esa clave se queda para siempre |
```

**Propuesto**
```md
| `must_change_password` | BOOLEAN | No | Obliga a cambiar la contraseña la próxima vez que entra | Cuando se crea una cuenta, o un administrador restablece la contraseña, se entrega una clave provisional. Sin esta marca, esa clave se queda para siempre |
```

**Por qué.** r1 §5: el restablecimiento entrega una clave provisional que debe cambiarse.

---

### C-13 · §5.4, párrafo "Contraseñas" y párrafo nuevo "Restablecer la contraseña de otra persona"

**Actual**
```md
**Contraseñas.** Tienen entre 8 y 72 caracteres, sin reglas de composición; 72 es el máximo que procesa el cifrado que se usa (BCrypt). La nueva tiene que ser distinta de la actual. Al iniciar sesión, un correo que no existe y una contraseña equivocada reciben la misma respuesta, para no revelar qué cuentas existen.

**Mientras la contraseña es provisional**, la cuenta solo puede consultar quién es y cambiar su contraseña. Sus roles no valen hasta que la cambie.
```

**Propuesto**
```md
**Contraseñas.** Tienen entre 8 y 72 caracteres, sin reglas de composición; 72 es el máximo que procesa el cifrado que se usa (BCrypt). La nueva tiene que ser distinta de la actual. Al iniciar sesión, un correo que no existe y una contraseña equivocada reciben la misma respuesta, para no revelar qué cuentas existen. Una cuenta bloqueada se entera de que lo está solo después de que su contraseña coincidió, así la respuesta no le revela nada a quien no la conoce.

**Mientras la contraseña es provisional**, la cuenta solo puede consultar quién es y cambiar su contraseña. Sus roles no valen hasta que la cambie. Tampoco recibe un rol privilegiado —`SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA`— hasta que la cambie (§5.7, D-72): quien le entregó la clave provisional la conoce, y no debe poder terminar con una cuenta privilegiada en sus manos.

**Restablecer la contraseña de otra persona** (D-63). Cuando alguien olvida su contraseña, un superadministrador o el administrador de accesos le restablece una provisional: la genera el sistema, se muestra una sola vez a quien la pidió y la persona debe cambiarla en su siguiente ingreso. En la misma operación se cierran todas las sesiones de esa cuenta, para que nadie siga dentro con la contraseña anterior (§5.12). Nadie restablece así la suya: para eso está el cambio de contraseña. La de `SYSTEM` no se restablece, porque esa cuenta no inicia sesión. El administrador de accesos no la restablece a una cuenta protegida (§5.5), y el administrador de una empresa no restablece contraseñas (D-57).
```

**Por qué.** r1 §5: el restablecimiento es una **[DECISION]** que hoy no está en el diccionario y que el contrato dice que se registra en la v1.18. Cierra todas las sesiones. Las limitaciones salen de r1 §3 (`SYSTEM`, nunca la propia), r4 D4-2 y r5 B (cuentas protegidas) y r4 D4-4 (el administrador de empresa no restablece). La respuesta a una cuenta bloqueada sale de r1 §5 (`403 ACCOUNT_SUSPENDED`), y el código la da solo después de que la contraseña coincidió. El rol privilegiado que espera a la contraseña propia sale de r5 E, con su razón. Que la clave provisional se muestre al administrador es lo construido; lo ideal sería un enlace de un solo uso al correo de la persona, que r5 deja pendiente: ver §5 de este archivo, punto 17.

---

### C-14 · §5.4, cuadro de `user_access_suspensions`: filas `reason`, `closed_by` y `closed_at`

**Actual**
```md
| `reason` | VARCHAR(20) | No | `vacaciones`, `licencia`, `baja`, `seguridad` | Distingue una ausencia temporal de una salida de la empresa. Sin esto, en un reporte de personal las vacaciones se cuentan como bajas |
```
```md
| `closed_by` | BIGINT FK → `users` | Sí | Quién reactivó la cuenta. Vacío mientras siga bloqueada | Sin este dato se sabría quién quitó el acceso pero no quién lo devolvió, que es la mitad de la pregunta en una revisión de seguridad |
| `closed_at` | TIMESTAMPTZ | Sí | Cuándo se reactivó | — |
```

**Propuesto**
```md
| `reason` | VARCHAR(20) | No | `vacaciones`, `licencia`, `baja`, `seguridad`. Los tres primeros forman la línea laboral; `seguridad` tiene una línea propia | Distingue una ausencia temporal de una salida de la empresa y de un incidente. Sin esto, en un reporte de personal las vacaciones se cuentan como bajas |
```
```md
| `closed_by` | BIGINT FK → `users` | Sí | Quién levantó el bloqueo, o quién lo canceló antes de que empezara. Vacío mientras nadie lo cierre | Sin este dato se sabría quién quitó el acceso pero no quién lo devolvió, que es la mitad de la pregunta en una revisión de seguridad |
| `closed_at` | TIMESTAMPTZ | Sí | Cuándo se levantó o se canceló. Es siempre el momento en que alguien lo cerró, nunca una fecha elegida | Un bloqueo levantado termina en ese momento. Uno cancelado antes de empezar conserva el período que tenía programado, para que se sepa qué se había planeado, y deja de bloquear |
```

**Por qué.** r2 R2 (cancelar un bloqueo programado: conserva su período y registra quién y cuándo), r3 B2 (el cierre ocurre siempre ahora) y r4 D4-9 (las dos líneas). V28 ya guarda la cancelación así (`closed_at <= lower(period)`).

---

### C-15 · §5.4, cuadro "Cómo queda cada caso", regla de cuenta activa y regla de cruce

**Actual**
```md
Cómo queda cada caso:

| Situación | Qué se hace | Al volver |
|---|---|---|
| Vacaciones | Se crea un período con motivo `vacaciones` | Se cierra el período con la fecha de regreso |
| Salida de la empresa | Se crea un período abierto con motivo `baja` | Si la persona regresa, se cierra ese período y vuelve a entrar con la misma cuenta y su historial completo |
| Bloqueo por seguridad | Período abierto con motivo `seguridad` | Se cierra cuando se aclara |

Un usuario está activo si **no tiene ningún período de bloqueo abierto**. No hay un campo que lo diga, para que no pueda contradecir a la tabla.

El sistema impide que dos bloqueos de la misma persona se crucen en el tiempo, incluso en el pasado.
```

**Propuesto**
```md
Cómo queda cada caso:

| Situación | Qué se hace | Al volver |
|---|---|---|
| Vacaciones o licencia | Se crea un período con motivo `vacaciones` o `licencia`, con su fin si ya se conoce | Termina solo al llegar su fin. Si la persona vuelve antes, o el período quedó abierto, se levanta en el momento en que regresa |
| Salida de la empresa | Se crea un período abierto con motivo `baja`. Lo crea solo el sistema, al registrar la salida del último colaborador activo de la cuenta (§5.3); nadie lo abre a mano | Si la persona regresa, o se le da acceso desde otra empresa del grupo, se cierra ese período y vuelve a entrar con la misma cuenta y su historial completo. Nadie lo cierra a mano |
| Bloqueo por seguridad | Período abierto con motivo `seguridad`, en su propia línea de tiempo | Se cierra cuando se aclara. La salida y el regreso no lo tocan |
| Un bloqueo programado que ya no hace falta | Se cancela antes de que empiece | — |

Un usuario está activo si **no tiene ningún bloqueo en curso**: ninguno que ya haya empezado, no haya terminado y no se haya cancelado. Basta uno, de cualquier línea, para que no pueda entrar. No hay un campo que lo diga, para que no pueda contradecir a la tabla.

**Dos líneas de tiempo** (D-62). Los bloqueos de una persona corren en dos líneas: la **laboral**, con `vacaciones`, `licencia` y `baja`, y la de **seguridad**. Dentro de una misma línea, el sistema impide que dos bloqueos se crucen en el tiempo, incluso en el pasado; los cancelados no cuentan. Entre líneas sí pueden cruzarse: un bloqueo por seguridad puede caer sobre unas vacaciones, una licencia o una baja. Tienen que ser independientes porque responden a hechos distintos. Si alguien investigado por un incidente sale de la empresa, la baja no puede borrar el bloqueo de seguridad. Y si después regresa, cerrar la baja no puede devolverle el acceso mientras el incidente no se aclare. La línea se deduce del motivo; no se guarda.
```

**Por qué.**
- Las dos líneas y su cruce salen de r4 D4-9. Los cancelados no cuentan (r2 R2).
- Basta un bloqueo en curso de cualquier línea (r4 D4-9, "What stays the same").
- "Abierto" deja de alcanzar: un bloqueo con fin futuro también está en curso, uno programado todavía no lo está y uno cancelado nunca lo estuvo (r2 R2, el estado `SCHEDULED | IN_FORCE | ENDED | CANCELLED`).
- "Se cierra con la fecha de regreso" contradecía r3 B2: el cierre es siempre ahora.
- La `baja` se cierra también al vincular desde otra empresa (r3 B1).
- La `baja` la abre y la cierra solo el sistema (C4-2, D-69).

---

### C-16 · §5.4, párrafo nuevo "Quién bloquea y levanta" (insertar después de "Dos líneas de tiempo")

**Actual:** no existe.

**Propuesto**
```md
**Quién bloquea y levanta** (D-57). El superadministrador y el administrador de accesos, por vacaciones, licencia o seguridad. Nadie bloquea su propia cuenta ni levanta sus propios bloqueos (D-70), `SYSTEM` no se bloquea, y el administrador de accesos no bloquea ni levanta los bloqueos de una cuenta protegida (§5.5). Un bloqueo programado sobre una cuenta que después pasó a ser protegida se conserva: se ve en su historia y lo cancela un superadministrador. La `baja` no la abre ni la cierra nadie a mano (D-69): la abre el sistema con la salida del último colaborador activo de la cuenta (§5.3) y la cierran el regreso o un nuevo acceso. El administrador de una empresa no bloquea cuentas: registra la salida, y la `baja` es su consecuencia.
```

**Por qué.** La tabla de r4 ("Open or close a suspension"), con r1 §3 (`SYSTEM` no se bloquea), C4-1 (nadie actúa sobre su propia cuenta, tampoco el superadministrador), r4 D4-2 y r5 B (cuentas protegidas) y r4 D4-4. C4-2: los motivos que se registran a mano son tres, y la `baja` es de la salida. El bloqueo programado antes de una promoción lo deja r5 sin cambio, a propósito ("A block scheduled before a promotion stays").

---

### C-17 · §5.4 "Qué queda registrado, exactamente": primera frase y ejemplo de las dos líneas

**Actual** (primera frase)
```md
Cada bloqueo y cada reactivación deja una fila que no se borra ni se reutiliza. Un ejemplo de la historia de una persona:
```

**Propuesto** (primera frase)
```md
Cada bloqueo deja una fila que no se borra ni se reutiliza; levantarlo o cancelarlo la completa con quién y cuándo, sin borrar nada. Un ejemplo de la historia de una persona:
```

**Actual** (la última pregunta de auditoría, después de la cual se inserta; no cambia)
```md
- *¿Está activo hoy?* No: la fila 3 sigue abierta.
```

**Propuesto** (insertar a continuación)
```md
Con dos líneas, un bloqueo por seguridad puede quedar debajo de uno de la línea laboral. Otra persona, investigada por un incidente, sale de la empresa mientras tanto:

| # | Período | Motivo | Línea | Ordenado por | Reactivado por |
|---|---|---|---|---|---|
| 1 | 03/08/2026 10:00 → *(abierto)* | seguridad | seguridad | Janover | — |
| 2 | 14/08/2026 18:20 → 01/09/2026 09:00 | baja | laboral | Tere | Tere |

La salida del 14/08/2026 abrió la fila 2 y no tocó la 1. El regreso del 01/09/2026 cerró la 2, y la persona sigue sin entrar: la fila 1 sigue abierta hasta que se aclare el incidente.
```

**Por qué.** r2 R2 (la cancelación completa la fila) y r4 D4-9. El ejemplo es la secuencia que r4 pide probar ("The security track, in order", pasos 1 a 3). Usé fechas `dd/mm/aaaa`. El ejemplo existente usa "10 feb 2026" y no lo toqué.

---

### C-18 · §5.4, párrafo "Se pueden programar por adelantado" y párrafo nuevo "Levantar y cancelar"

**Actual**
```md
**Se pueden programar por adelantado.** Nada impide crear un bloqueo con fecha de inicio futura para unas vacaciones ya aprobadas: la persona conserva el acceso hasta que llegue esa fecha y lo pierde sola. No hace falta que nadie se acuerde el día que empieza.
```

**Propuesto**
```md
**Se pueden programar por adelantado, nunca hacia atrás** (D-61). Nada impide crear un bloqueo con fecha de inicio futura para unas vacaciones ya aprobadas: la persona conserva el acceso hasta que llegue esa fecha y lo pierde sola. No hace falta que nadie se acuerde el día que empieza. Lo que no se puede es crearlo con inicio en el pasado: esta tabla dice quién podía entrar en cada momento, y un bloqueo que no ocurrió no se registra. Sin fecha de inicio, empieza en el momento en que se registra.

**Levantar y cancelar ocurren siempre ahora.** Cerrar un bloqueo en curso lo termina en ese momento. Cerrar uno programado, antes de que empiece, lo cancela: conserva el período que tenía programado, registra quién lo canceló y cuándo, y deja de contar. Nadie elige la fecha de cierre, porque una fecha elegida podría falsear quién tenía acceso. Un bloqueo que ya terminó, o que ya se levantó o se canceló, no se vuelve a cerrar.
```

**Por qué.** r3 B9 (no empieza en el pasado; sin inicio, empieza ahora), r3 B2 (el cierre es siempre ahora y sin fecha del cliente, con la razón del contrato) y r2 R2 (cancelar, y no volver a cerrar uno terminado o cerrado).

---

### C-19 · §5.4, párrafo de las sesiones al bloquear

**Actual**
```md
**Al bloquear una cuenta se cierran también sus sesiones abiertas**, o la persona seguiría dentro hasta que vencieran. Como las sesiones viven en el servidor (§5.12), cerrarlas es borrarlas y el efecto es inmediato.
```

**Propuesto**
```md
**Cuando empieza un bloqueo se cierran también las sesiones abiertas de la cuenta**, o la persona seguiría dentro hasta que vencieran. Como las sesiones viven en el servidor (§5.12), cerrarlas es borrarlas y el efecto es inmediato. Un bloqueo programado surte efecto al llegar su inicio: desde ese momento la sesión deja de servir, sin que nadie tenga que hacer nada.
```

**Por qué.** r1 §5: un bloqueo que empieza ahora cierra las sesiones, y el filtro de sesión trata como desconectada a una cuenta bloqueada, así que un bloqueo programado surte efecto solo. r4 D4-9: las sesiones se cierran cuando empieza cualquier período.

---

### C-20 · §5.5 `roles`, cuadro "Ejemplo"

**Actual**
```md
| code | name | empresa | alcance | del sistema |
|---|---|---|---|---|
| `SUPERADMIN` | Superadministrador | *(todas)* | Todo el sistema | Sí |
| `ADMIN_EMPRESA` | Administrador de empresa | *(todas)* | Una empresa | Sí |
| `ASESOR_VENTAS` | Asesor de ventas | *(todas)* | Una sede | Sí |
| `JEFE_PATIO` | Jefe de patio | Millennium Perú | Una sede | No |
```

**Propuesto**
```md
| code | name | empresa | alcance | del sistema |
|---|---|---|---|---|
| `SUPERADMIN` | Superadministrador | *(todas)* | Todo el sistema | Sí |
| `ADMIN_ACCESOS` | Administrador de accesos | *(todas)* | Todo el sistema | Sí |
| `ADMIN_EMPRESA` | Administrador de empresa | *(todas)* | Una empresa | Sí |
| `ASESOR_VENTAS` | Asesor de ventas | *(todas)* | Una sede | Sí |
| `JEFE_PATIO` | Jefe de patio | Millennium Perú | Una sede | No |
```

**Por qué.** r4 D4-1: `scope_level = SYSTEM`, `is_system = true`, `company_id` vacío.

---

### C-21 · §5.5 "Crear roles nuevos", primera frase

**Actual**
```md
Los seis roles que trae el sistema (cuadro al final de esta sección) son el punto de partida, no la lista definitiva: los puestos cambian y hay que poder agregar roles sin tocar el programa. Crear uno es insertar una fila en `roles` y marcar sus permisos, todo desde la pantalla de administración.
```

**Propuesto**
```md
Los siete roles que trae el sistema (cuadro al final de esta sección) son el punto de partida, no la lista definitiva: los puestos cambian y hay que poder agregar roles sin tocar el programa. Crear uno es insertar una fila en `roles` y marcar sus permisos, todo desde la pantalla de administración.
```

**Por qué.** r4 D4-1 agrega un séptimo rol del sistema.

---

### C-22 · §5.5, regla 2

**Actual**
```md
**2. La misma regla vale al asignar.** Restringir solo la creación no sirve de nada: si el administrador no puede crear un rol con acceso total pero sí puede asignarse uno existente que ya lo tiene, el resultado es idéntico. Nadie puede otorgar un rol cuyos permisos excedan los suyos.
```

**Propuesto**
```md
**2. La misma regla vale al asignar.** Restringir solo la creación no sirve de nada: si el administrador no puede crear un rol con acceso total pero sí puede asignarse uno existente que ya lo tiene, el resultado es idéntico. Nadie puede otorgar un rol cuyos permisos excedan los suyos. La única excepción es el administrador de accesos, con los límites que se explican más abajo (D-56).
```

**Por qué.** r4 D4-3: una excepción explícita a la regla 2, cuyo texto va en esta propuesta.

---

### C-23 · §5.5, párrafo de la revisión periódica *(opcional)*

**Actual**
```md
Como los accesos otorgados no caducan solos, el control que los mantiene sanos es revisarlos cada cierto tiempo. El modelo lo permite sin campos adicionales: cada asignación guarda quién la otorgó y cuándo, así que se puede listar todo lo vigente y depurar lo que ya no corresponde.
```

**Propuesto**
```md
Como los accesos otorgados no caducan solos —salvo los que se otorgan con caducidad (§5.7, §5.8)—, el control que los mantiene sanos es revisarlos cada cierto tiempo. El modelo lo permite sin campos adicionales: cada asignación guarda quién la otorgó y cuándo, así que se puede listar todo lo vigente y depurar lo que ya no corresponde.
```

**Por qué.** Con el cierre automático de r3 B8, "no caducan solos" se lee como si `expires_at` no hiciera nada.

---

### C-24 · §5.5, párrafo nuevo "Una excepción a la regla 2" y párrafo "Límite de lo que la base puede garantizar"

**Actual** (el párrafo después del cual se inserta; no cambia)
```md
Los roles marcados como propios del sistema no se pueden editar ni borrar, aunque quien lo intente tenga todos los permisos.
```

**Propuesto** (insertar a continuación)
```md
**Una excepción a la regla 2, con nombre propio** (D-56). El administrador de accesos asigna roles cuyos permisos no tiene. Su trabajo es dar acceso, no registrar facturas ni dar de alta unidades, y no tendría sentido darle todos los permisos del sistema solo para que pueda asignarlos. La excepción tiene tres límites, que son los que la hacen segura (D-55):

- **Nunca actúa sobre sí mismo.** No se asigna roles ni actúa sobre su propia cuenta, como nadie (D-70).
- **Nunca otorga ni retira `SUPERADMIN`, `ADMIN_ACCESOS` ni `ADMIN_EMPRESA`.** Esos tres los otorga y los retira solo un superadministrador.
- **Nunca actúa sobre una cuenta protegida**, la que tiene `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente: no la vincula a un colaborador, no le asigna ni le retira roles, no la bloquea ni le levanta bloqueos y no le restablece la contraseña. Esas cuentas las administra solo un superadministrador.

Con esos límites, la excepción no abre el camino que la regla 2 cierra: el administrador de accesos no puede convertirse en superadministrador ni convertir a otro, ni quitarle el acceso a quien lo controla. Tampoco puede tomar la cuenta de un administrador de empresa —restablecerle la contraseña y entrar como él— para inventar un colaborador y darle acceso. Todo lo que otorga queda con su nombre en `granted_by`. Es la única excepción a la regla 2; extenderla a otro rol requiere una decisión nueva.
```

**Actual** (párrafo siguiente)
```md
**Límite de lo que la base puede garantizar.** La consistencia entre la empresa del rol y la empresa de la asignación no se puede exigir con una llave foránea, porque el campo admite vacío para los roles generales. Se resuelve con una validación programada que rechaza asignar un rol de una empresa dentro de otra. Las reglas 1 a 3 también viven en la aplicación: la base no sabe qué permisos tiene quien está ejecutando la operación.
```

**Propuesto**
```md
**Límite de lo que la base puede garantizar.** La consistencia entre la empresa del rol y la empresa de la asignación no se puede exigir con una llave foránea, porque el campo admite vacío para los roles generales. Se resuelve con una validación programada que rechaza asignar un rol de una empresa dentro de otra. Las reglas 1 a 3, la excepción del administrador de accesos y sus límites también viven en la aplicación: la base no sabe qué permisos ni qué roles tiene quien está ejecutando la operación.
```

**Por qué.** r4 D4-2 (los tres límites, con la lista exacta de lo que es "actuar") y r4 D4-3 (la excepción). r5 A agrega `ADMIN_EMPRESA` a los roles reservados al superadministrador, y r5 B a los que hacen protegida una cuenta, con la razón que da el contrato: sin eso, una sola persona derrota la separación de funciones.

---

### C-25 · §5.5 Reglas de negocio, cuadro "Roles que trae el sistema"

**Actual**
```md
| code | name | Alcance |
|---|---|---|
| `SUPERADMIN` | Superadministrador | Todo el sistema |
| `ADMIN_EMPRESA` | Administrador de empresa | Una empresa |
| `OPERACIONES` | Operaciones | Una empresa |
| `CONTABILIDAD` | Contabilidad | Una empresa |
| `ASESOR_VENTAS` | Asesor de ventas | Una sede |
| `TALLER` | Taller | Una sede |
```

**Propuesto**
```md
| code | name | Alcance |
|---|---|---|
| `SUPERADMIN` | Superadministrador | Todo el sistema |
| `ADMIN_ACCESOS` | Administrador de accesos | Todo el sistema |
| `ADMIN_EMPRESA` | Administrador de empresa | Una empresa |
| `OPERACIONES` | Operaciones | Una empresa |
| `CONTABILIDAD` | Contabilidad | Una empresa |
| `ASESOR_VENTAS` | Asesor de ventas | Una sede |
| `TALLER` | Taller | Una sede |

El administrador de accesos da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas, en todas las empresas del grupo (D-55). Es uno para todo el grupo, no uno por empresa, y por eso su alcance es todo el sistema: ve todas las empresas. No registra colaboradores ni sus salidas o regresos (D-57), y sus tres límites están en la excepción a la regla 2. Una misma cuenta no tiene a la vez este rol y el de administrador de empresa (D-71).
```

**Por qué.** r4 D4-1. La descripción sigue la que el contrato fija para la fila del rol. Que vea todas las empresas también sale de D4-1 ("like SUPERADMIN, it sees every company"). La incompatibilidad de los dos roles sale de r5 D.

---

### C-26 · §5.6, cuadro "Ejemplo de `role_permissions`"

**Actual**
```md
| Rol | Permisos que incluye |
|---|---|
| Operaciones | 1, 3 |
| Contabilidad | 1, 2 |
| Administrador de empresa | 1, 2, 3, 4, 5, 6 |
```

**Propuesto**
```md
| Rol | Permisos que incluye |
|---|---|
| Operaciones | 1, 3 |
| Contabilidad | 1, 2 |
| Administrador de empresa | 1, 2, 3, 4, 5 |
| Administrador de accesos | 6 |

El administrador de empresa no lleva `seguridad.crear_usuario`: las cuentas las crea el administrador de accesos (D-57).
```

**Por qué.** Con r4 D4-4, el ejemplo actual le da al administrador de empresa el permiso de crear cuentas, que es justo lo que pierde.

---

### C-27 · §5.7 Reglas de negocio, fila de unicidad

**Actual**
```md
| El mismo rol no se puede dar dos veces sobre lo mismo, mientras siga vigente | Asignaciones duplicadas que después nadie sabe cuál retirar |
```

**Propuesto**
```md
| El mismo rol no se puede dar dos veces sobre lo mismo —la misma empresa, sede y marca—, mientras siga vigente. Sobre dos empresas distintas, sí | Asignaciones duplicadas que después nadie sabe cuál retirar, sin impedir que Tere tenga Contabilidad en dos empresas |
```

**Por qué.** Era la causa B de la auditoría: el índice anterior prohibía el mismo rol en dos empresas, contra el ejemplo de Tere. V27 lo reemplazó por un índice sobre todo el alcance con `NULLS NOT DISTINCT`. La regla ya era esa; la fila lo dice explícito.

---

### C-28 · §5.7, párrafo "Lo que valida la aplicación"

**Actual**
```md
Lo que valida la aplicación, porque la base no puede verlo en una sola fila: para otorgar un rol de alcance empresa o sede, la cuenta debe estar vinculada a un colaborador activo de esa empresa; para uno de alcance sistema, a un colaborador activo de alguna empresa (§5.3). La única excepción es el rol del primer superadministrador, que se otorga al arrancar (§5.2).
```

**Propuesto**
```md
Lo que valida la aplicación, porque la base no puede verlo en una sola fila: para otorgar un rol de alcance empresa o sede, la cuenta debe estar vinculada a un colaborador activo de esa empresa; para uno de alcance sistema, a un colaborador activo de alguna empresa (§5.3). La única excepción es el rol del primer superadministrador, que se otorga al arrancar (§5.2).

También valida que `ADMIN_EMPRESA` nunca se limite a una marca (D-59). Administrar una empresa —registrar sus colaboradores, sus salidas y sus regresos— alcanza a la empresa entera, y un colaborador no pertenece a una marca. La base no puede verlo, porque el código del rol no está en la fila de la asignación. Los demás roles de empresa y de sede sí pueden limitarse a una marca, como el asesor de Lima Bikes.
```

**Por qué.** r4 D4-8, incluida su nota de que la regla vive en la aplicación porque la base no ve el rol en una fila. Reemplaza el caso especial de r3 B10 (el administrador con marca no administraba), que deja de existir.

---

### C-29 · §5.7, subsección nueva "Quién otorga y retira roles" (insertar después del párrafo de C-28, antes de §5.8)

**Actual:** no existe. La regla provisional estaba en §5.13 ("Se exige tener ese mismo rol vigente").

**Propuesto**
```md
#### Quién otorga y retira roles

Otorgar y retirar roles es trabajo de dos roles del sistema (D-55, D-57):

| Quién | Qué roles otorga y retira | Sobre qué cuentas |
|---|---|---|
| Superadministrador | Cualquiera, sobre cualquier empresa, sede o marca | Cualquiera, salvo `SYSTEM` y la suya |
| Administrador de accesos | Cualquiera, salvo `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, sobre cualquier empresa, sede o marca | Cualquiera, salvo `SYSTEM`, la suya y las protegidas (§5.5) |
| Administrador de empresa | Ninguno | — |
| Cualquier otro rol | Ninguno | — |

- Retirar una asignación exige poder otorgarla: el administrador de accesos no retira un `SUPERADMIN`, un `ADMIN_ACCESOS` ni un `ADMIN_EMPRESA`.
- Nadie se otorga ni se retira roles a sí mismo, tampoco un superadministrador (D-70).
- Quien otorga o retira tiene que estar vinculado a un colaborador activo (§5.2).
- El administrador de empresa no otorga roles, ni siquiera en su empresa: registra a las personas, y el acceso lo da otro (§5.3).
- **Separación estática** (D-71). Una cuenta nunca tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` vigentes a la vez: otorgar uno a quien tiene el otro se rechaza. Si una persona pasa de un puesto al otro, primero se le retira el que tenía.
- **Un rol privilegiado exige contraseña propia** (D-72). `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA` se otorgan solo a una cuenta que ya cambió su contraseña provisional (§5.4). El primer superadministrador no se otorga así: lo crea el arranque (§5.2).
- El mismo rol puede tenerse sobre dos empresas a la vez, como Tere; retirar uno no toca el otro.
- La segunda empresa de una persona la vincula el administrador de accesos: al dar acceso a su colaborador en la otra empresa con el mismo correo, el perfil se vincula a la cuenta que ya tiene, sin crear otra (§5.3). Un superadministrador solo hace falta si la cuenta es protegida.

Las dos reglas nuevas las valida la aplicación: la base no ve dos asignaciones a la vez ni la contraseña de la cuenta al guardar una asignación.
```

**Por qué.**
- La política sale de la tabla de r4 ("Grant a role", "Revoke an assignment"), con r5 A: `ADMIN_EMPRESA` también es un rol reservado al superadministrador.
- C4-1: nadie actúa sobre su propia cuenta; D4-5 gana sobre la fila de la tabla de r4 que decía "to anyone but SYS". Resuelve §5 de este archivo, punto 1.
- r1 D3: retirar exige poder otorgar, y quien otorga está vinculado a un colaborador.
- r4 D4-6: todo actor es colaborador.
- r4 D4-4: el administrador de empresa no otorga nada.
- r5 D: separación estática de funciones (ANSI INCITS 359), validada en la aplicación.
- r5 E: el rol privilegiado espera a la contraseña propia. El texto que aprobaste nombraba los roles que hacen protegida una cuenta; r5 lo extiende a `ADMIN_EMPRESA` por coherencia con r5 B: ver §5 de este archivo, punto 16.
- r4 regresión: el administrador de accesos vincula una cuenta de otra empresa → 200.
- Reemplaza la regla provisional de §5.13, que r1 D3 ya señalaba como contraria a §5.2.

---

### C-30 · §5.8, punto 5

**Actual**
```md
5. **Un acceso que caduca se cierra como si lo hubieran retirado**: el sistema le pone la fecha de retiro con motivo `caducidad`. Tiene que ser así y no quedar abierto con la fecha vencida, porque de lo contrario esa persona no podría recibir de nuevo el mismo acceso —el sistema lo vería como duplicado del que sigue abierto— y el auditor externo que vuelve el año siguiente quedaría fuera.
```

**Propuesto**
```md
5. **Un acceso que caduca se cierra como si lo hubieran retirado** (D-64): un proceso automático le pone como fecha de retiro la de su caducidad —no la del momento en que pasa el proceso—, con motivo `caducidad`. Tiene que ser así y no quedar abierto con la fecha vencida, porque de lo contrario esa persona no podría recibir de nuevo el mismo acceso —el sistema lo vería como duplicado del que sigue abierto— y el auditor externo que vuelve el año siguiente quedaría fuera. El acceso deja de valer exactamente al caducar, sin esperar al proceso: lo vigente es lo que no está retirado y cuya caducidad no llegó. Si se otorga de nuevo el mismo rol sobre lo mismo antes de que pase el proceso, en ese momento se cierra la asignación vencida con esa misma llave, y solo esa.
```

**Por qué.** r3 B8: el proceso programado cierra con `revoked_at = expires_at`. Todo cálculo de "vigente" usa `revoked_at IS NULL AND (expires_at IS NULL OR expires_at > now)`, y otorgar de nuevo cierra solo la vencida con la misma llave.

---

### C-31 · §5.9, párrafo de `SYSTEM`

**Actual**
```md
El usuario `SYSTEM` no puede iniciar sesión y figura como responsable de toda carga masiva y de todo proceso automático, incluida la actualización por archivo del SAT. Sin él, esos registros quedarían sin autor.
```

**Propuesto**
```md
El usuario `SYSTEM` no puede iniciar sesión y figura como responsable de toda carga masiva y de todo proceso automático, incluida la actualización por archivo del SAT. Sin él, esos registros quedarían sin autor. Tampoco se vincula a un colaborador, no recibe roles, no se bloquea y no se le restablece la contraseña: no es una persona y nadie lo administra.
```

**Por qué.** r1 §3 ("SYSTEM account").

---

### C-38 · §5.9, párrafo "Qué pasa automáticamente al registrar una baja" *(agregada en la revisión)*

**Actual**
```md
**Qué pasa automáticamente al registrar una baja.** Cuando se crea un bloqueo con motivo `baja`, el sistema revoca en la misma operación todas las asignaciones activas de esa persona, con motivo `baja_cuenta`. Son dos acciones distintas y las dos hacen falta: bloquear la cuenta corta el acceso en el momento, y retirar los roles evita que, si esa cuenta se reactiva algún día, la persona recupere de golpe todo lo que tenía. Automatizarlo es lo que impide que el segundo paso dependa de que alguien lo recuerde.
```

**Propuesto**
```md
**Qué pasa automáticamente al registrar una baja.** El bloqueo con motivo `baja` lo crea solo el sistema, cuando se registra la salida del último colaborador activo de la cuenta (§5.3, D-69); nadie lo abre ni lo cierra a mano. En la misma operación, el sistema revoca todas las asignaciones activas de esa persona, con motivo `baja_cuenta`. Son dos acciones distintas y las dos hacen falta: bloquear la cuenta corta el acceso en el momento, y retirar los roles evita que, si esa cuenta se reactiva algún día, la persona recupere de golpe todo lo que tenía. Automatizarlo es lo que impide que el segundo paso dependa de que alguien lo recuerde.
```

**Por qué.** C4-2: la `baja` es de la salida. Abrirla a mano responde como un motivo desconocido, y cerrarla a mano se rechaza: la cierran el regreso o un nuevo acceso (r3 B1). Resuelve §5 de este archivo, punto 8: una cuenta ya no puede quedar dada de baja mientras su colaborador sigue activo.

---

### C-32 · §5.9, párrafo "Protección necesaria"

**Actual**
```md
**Protección necesaria:** el sistema impide retirar el último superadministrador activo, ya sea quitándole el rol, bloqueando su cuenta o registrando la salida de su colaborador. Sin ninguno, no hay forma de volver a entrar sin tocar la base a mano.
```

**Propuesto**
```md
**Protección necesaria:** el sistema impide retirar el último superadministrador activo, ya sea quitándole el rol, bloqueando su cuenta o registrando la salida que la cierra. Sin ninguno, no hay forma de volver a entrar sin tocar la base a mano.

No basta con que quede otro hoy (D-65). Una operación que le quita, o le quitará, el rol a un superadministrador solo procede si queda otra cuenta que lo sea **de forma duradera**: con el rol `SUPERADMIN` vigente y sin caducidad, y sin ningún bloqueo en curso ni programado. Esas operaciones son retirarle el rol, bloquearlo ahora o con un bloqueo programado, y registrar la salida que cierra su cuenta. Un superadministrador cuyo rol caduca en un mes, o que tiene unas vacaciones programadas, no cuenta como respaldo: al llegar esa fecha el sistema podría quedarse sin ninguno sin que nadie haya hecho nada. Además, esas operaciones se hacen de a una: si dos personas intentan al mismo tiempo retirar a dos superadministradores distintos, la segunda espera a la primera y ve su resultado, en vez de pasar las dos.

**Es el único freno de una salida** (D-60). La salida de un colaborador siempre retira sus roles en esa empresa y, si era el último colaborador activo de la cuenta, la cierra, también si es la de un administrador de accesos o de empresa: quitarle el acceso a quien se fue no espera a nadie. Solo se detiene si dejaría al sistema sin un superadministrador activo.
```

**Por qué.** r3 B7: la protección en el tiempo (superadministrador duradero) y con concurrencia (operaciones serializadas). r4 D4-9: la guardia cuenta los bloqueos de las dos líneas. r5 C reemplaza la regla de r4 D4-4 que exigía un superadministrador para la salida que cierra una cuenta protegida: la salida siempre quita el acceso, y el último superadministrador es lo único que la detiene.

---

### C-33 · §5.9, párrafo del primer superadministrador y párrafo nuevo sobre el administrador de accesos

**Actual**
```md
**El primer superadministrador no puede dar acceso a nadie ni otorgar roles** hasta vincularse a su perfil de colaborador (§5.2).
```

**Propuesto**
```md
**El primer superadministrador no puede dar acceso a nadie ni otorgar roles** hasta vincularse a su perfil de colaborador (§5.2). Lo mismo vale para el administrador de accesos y el de empresa: actúan solo mientras su cuenta está vinculada a un colaborador activo (D-57). Vincularse a sí mismo es solo de ese momento: una cuenta que ya tiene un colaborador activo no se vincula a sí misma a otro (D-70).

**No hace falta proteger al último administrador de accesos.** El superadministrador puede hacer todo lo que él hace, así que el sistema nunca se queda sin quien administre los accesos (D-55).
```

**Por qué.** r4 D4-6 (todo actor es colaborador), r5 T8 (la autovinculación es solo del arranque) y r4 D4-5 (no hay guardia del último administrador de accesos, con su razón).

---

### C-34 · §5.12 Sesiones, filas del cuadro

**Actual**
```md
| Cambio de contraseña | Se cierran las demás sesiones de la persona; sigue abierta la que hizo el cambio | Quien conocía la contraseña anterior pierde el acceso en ese momento |
| Bloqueo de la cuenta | Se cierran todas sus sesiones | Ver §5.4 |
| Cerrar sesión | Se borra la sesión en el servidor, no solo en la pantalla | La cookie que quedara en el navegador ya no sirve para nada |
| Roles y contraseña provisional | Se leen de la base en cada operación, no se guardan en la sesión | Un rol otorgado o retirado vale desde la operación siguiente, sin esperar a que la persona vuelva a entrar |
```

**Propuesto**
```md
| Cambio de contraseña | Se cierran las demás sesiones de la persona; sigue abierta la que hizo el cambio | Quien conocía la contraseña anterior pierde el acceso en ese momento |
| Restablecimiento de la contraseña por un administrador | Se cierran todas las sesiones de la persona | Nadie sigue dentro con la contraseña anterior (§5.4) |
| Bloqueo de la cuenta | Se cierran todas sus sesiones cuando el bloqueo empieza, también si estaba programado | Ver §5.4 |
| Cerrar sesión | Se borra la sesión en el servidor, no solo en la pantalla | La cookie que quedara en el navegador ya no sirve para nada |
| Roles, bloqueos y contraseña provisional | Se leen de la base en cada operación, no se guardan en la sesión | Un rol otorgado o retirado vale desde la operación siguiente, y un bloqueo programado corta la sesión apenas empieza, sin esperar a que la persona vuelva a entrar |
```

**Por qué.** r1 §5: el restablecimiento cierra todas las sesiones, y el filtro de sesión vuelve a leer en cada operación si la cuenta está bloqueada. r4 D4-9: las sesiones se cierran al empezar cualquier período. La cookie `MM_SESSION` no cambia: la duda de r2 R1 la resolvió una prueba que confirma el nombre configurado.

---

### C-35 · §5.13 Lo que el backend implementa hoy (se reemplaza completa)

**Actual**
```md
### 5.13 Lo que el backend implementa hoy

El bloque 2 se construyó antes que `organization`, así que lo que depende de empresas, sedes y colaboradores quedó en una versión provisional. Nada de esto cambia el diseño de este capítulo: es lo que falta para llegar a él.

| Tema | Hoy | Cuándo se completa |
|---|---|---|
| `user_roles` | Se creó antes de su bloque (3b), sin `company_id`, `branch_id` ni `commercial_brand_id`. Solo se asignan roles de alcance sistema | Con el bloque 3 |
| `roles.company_id` | No existe todavía: todos los roles son generales, y `code` es único en todo el sistema | Con el bloque 3, que define también la unicidad por empresa |
| Otorgar un rol | Se exige tener ese mismo rol vigente, porque todavía no hay permisos que comparar (§5.5, reglas 1 a 3) | Cuando existan `permissions` y `role_permissions` |
| Gestión de usuarios | Solo el superadministrador, porque todavía no se pueden acotar por empresa (§2.5) | Con el bloque 3 |
| Colaboradores | No existen: las cuentas se crean solas y el registro público pide solo correo y contraseña, así que D-38 todavía no se aplica | Con el bloque 3b |
| `user_access_suspensions`, `permissions`, `role_permissions` | No existen todavía | Los bloqueos, con el bloque 3b; los permisos, cuando se cargue la matriz de permisos por rol (§14) |
| Retirar un rol y caducidad | No hay operación para retirar un rol ni nada que fije una caducidad, así que las protecciones de §5.5 y §5.9 y el cierre por caducidad de §5.8 todavía no tienen qué vigilar | Con la pantalla de roles y usuarios |
```

**Propuesto**
```md
### 5.13 Lo que el backend implementa hoy

El paso 3 completó lo que el bloque 2 había dejado provisional: colaboradores, dar acceso, asignaciones con su empresa, sede y marca, bloqueos, caducidad y la frontera entre empresas en la base (§2.5). Lo que sigue es lo que todavía no llega al diseño de este capítulo. Nada de esto cambia el diseño: es lo que falta para llegar a él.

| Tema | Hoy | Cuándo se completa |
|---|---|---|
| `permissions` y `role_permissions` | No existen. Quién otorga qué lo fija §5.7: el superadministrador otorga cualquier rol y el administrador de accesos cualquiera salvo `SUPERADMIN`, `ADMIN_ACCESOS` y `ADMIN_EMPRESA`. Las reglas 1 a 3 de §5.5 todavía no tienen permisos que comparar, y las operaciones exigen un rol, no un permiso | Cuando se cargue la matriz de permisos por rol, antes del paso 5 (§3, ajuste 3) |
| Qué lee un rol global | El superadministrador y el administrador de accesos leen los datos de todas las empresas: hoy empresas, marcas, sedes, ubicaciones y colaboradores | Con la matriz de permisos, antes del paso 5 (D-68) |
| La marca de una asignación | Acota dónde se otorga un rol, pero todavía no acota lo que se lee: la frontera que aplican la aplicación y la base es la empresa. Hoy ningún dato que se lee pertenece a una marca | Con el primer dato que pertenezca a una marca |
| `roles.company_id` | No existe todavía: todos los roles son generales, y `code` es único en todo el sistema. Tampoco está decidido quién crea los roles de una empresa, ahora que el administrador de empresa no otorga roles (D-57) | Con el paso 2, que define también la unicidad por empresa. Quién los crea se decide con la matriz de permisos |
| Registro público | Sigue pidiendo solo correo y contraseña y crea una cuenta sin colaborador, contra D-38. Viene apagado (D-32) | Cuando las convenciones de la API definan cómo lista sedes y tipos de documento sin sesión (convenciones, §14.16) |
| Regreso de un colaborador | Reemplaza la fecha de ingreso y borra la de salida: el período anterior en esa empresa se pierde | Necesita un diseño en este diccionario (§14) |
```

**Por qué.**
- Las filas viejas de `user_roles`, gestión de usuarios, colaboradores, bloqueos y retiro con caducidad quedaron construidas (V26 a V29, r1 a r3).
- La regla "tener el mismo rol" la reemplazó r1 D3, y después r4 y r5 A.
- Las filas nuevas salen de r1 §8 (fuera de alcance: registro público con perfil, historia de reingresos, matriz de permisos), de r4 D4-7 y de lo que r5 registra sin cambio: la matriz llega antes del paso 5 y decide quién crea los roles de una empresa.
- El registro público lo verifiqué en el código: `SignUpCommand` lleva solo correo y contraseña.
- La marca no acota las lecturas porque `AccessScope` y la base filtran por empresa: ver §5 de este archivo, punto 7.
- Si decides sobre `employees.termination_reason` (punto 5), quizás convenga una fila más.

---

### C-36 · §14 Decisiones, párrafo inicial *(opcional; depende de §5 de este archivo, punto 4)*

**Actual**
```md
Queda una decisión abierta, con su momento fijado: las longitudes de los documentos tributarios de Colombia y México (§4.7), antes de registrar la primera empresa de cada país (D-52). Lo que falta para construir son datos, no definiciones: la matriz de permisos por rol, los datos maestros reales —razón social y documento tributario de la empresa, sedes con su código de establecimiento, marcas, proveedores, colaboradores—. El archivo oficial de divisiones territoriales de Perú ya está cargado (§4.5).
```

**Propuesto**
```md
Quedan dos decisiones abiertas. Una tiene su momento fijado: las longitudes de los documentos tributarios de Colombia y México (§4.7), antes de registrar la primera empresa de cada país (D-52). La otra es cómo se conserva el período anterior de un colaborador que regresa a la misma empresa (§5.3, §5.13). Lo que falta para construir son datos, no definiciones: la matriz de permisos por rol, que tiene que llegar antes del paso 5 (D-68), y los datos maestros reales —razón social y documento tributario de la empresa, sedes con su código de establecimiento, marcas, proveedores, colaboradores—. El archivo oficial de divisiones territoriales de Perú ya está cargado (§4.5).
```

**Por qué.** r1 §8: "Rehire history (reactivation erases the previous period): needs a dictionary design". r4 D4-7 y r5 dan fecha a la matriz.

---

## 3. Decisiones nuevas (insertar en "Decisiones ya cerradas", después de D-54)

```md
| D-55 | Existe un administrador de accesos para todo el grupo: el rol `ADMIN_ACCESOS`, del sistema y de alcance sistema, que da acceso a los colaboradores, asigna y retira roles, bloquea cuentas y restablece contraseñas en todas las empresas, y por eso las ve todas. Tiene tres límites: nunca actúa sobre su propia cuenta (D-70); nunca otorga ni retira `SUPERADMIN`, `ADMIN_ACCESOS` ni `ADMIN_EMPRESA`, que otorga y retira solo un superadministrador; y nunca actúa sobre una cuenta protegida —la que tiene `SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA` vigente—, que solo administra un superadministrador. No se protege al último administrador de accesos, porque el superadministrador puede hacer todo lo que él hace (§5.5, §5.7, §5.9) |
| D-56 | El administrador de accesos asigna roles cuyos permisos no tiene, dentro de los límites de D-55. Es la única excepción a la regla 2 de §5.5; extenderla a otro rol requiere una decisión nueva (§5.5) |
| D-57 | Separación de funciones. El administrador de una empresa registra y modifica sus colaboradores y registra sus salidas y regresos; ve las cuentas de su empresa sin modificarlas, y no da acceso, no otorga ni retira roles, no bloquea cuentas ni restablece contraseñas. El administrador de accesos hace eso y no registra colaboradores, salidas ni regresos, para que nadie pueda inventar un colaborador y darle acceso. El superadministrador hace ambas cosas, además de las empresas y los catálogos. Todos actúan solo mientras su cuenta está vinculada a un colaborador activo, y ninguno actúa sobre lo suyo, como fija D-70 (§2.5, §5.2, §5.3, §5.4, §5.7) |
| D-58 | Dar acceso crea la cuenta del colaborador, con una contraseña provisional que genera el sistema y se muestra una sola vez, o la vincula a la cuenta que la persona ya tiene por otra empresa del grupo; nunca cambia los roles ni la contraseña de una cuenta existente. Lo hacen el superadministrador y el administrador de accesos, este en cualquier empresa salvo con una cuenta protegida (D-55). La única cuenta que se vincula a sí misma es la del primer superadministrador, mientras no tiene un colaborador activo (D-70). Vincular una cuenta cerrada por una salida cierra su `baja` (§5.3, §5.4) |
| D-59 | `ADMIN_EMPRESA` nunca se limita a una marca: administrar una empresa alcanza a la empresa entera. Lo valida la aplicación, porque la base no ve el rol en la fila de la asignación. Los demás roles de empresa y de sede sí pueden limitarse a una marca (§5.7) |
| D-60 | La fecha de salida de un colaborador no puede ser posterior a hoy, calculado en la zona de su sede o, sin sede, en la del país de su empresa (§2.6): la salida se registra cuando ocurre. La salida siempre quita el acceso, también el de una cuenta protegida: retira los roles de esa empresa (`baja_colaborador`) y, si era el último colaborador activo, cierra la cuenta desde ese momento —`baja`, salvo que ya haya una en curso; `baja_cuenta`; sesiones cerradas—, termina las vacaciones o la licencia en curso, cancela los bloqueos programados de la línea laboral y no toca los bloqueos por seguridad. Su único freno es la protección del último superadministrador (§5.9). La fecha del regreso no es anterior a la de salida ni posterior a hoy, calculado igual. El regreso cierra solo la `baja`, y los roles no vuelven solos (§5.3, §5.4, §5.9) |
| D-61 | Un bloqueo de cuenta empieza ahora o en el futuro, nunca en el pasado. Se cierra siempre en el momento en que alguien lo cierra, sin fecha elegida: uno en curso termina ahí; uno programado queda cancelado, conserva su período, registra quién y cuándo, y deja de contar. Uno terminado, levantado o cancelado no se vuelve a cerrar (§5.4) |
| D-62 | Los bloqueos de cuenta corren en dos líneas de tiempo: la laboral —`vacaciones`, `licencia`, `baja`— y la de `seguridad`. Dentro de una línea no se cruzan, incluso en el pasado; entre líneas sí. Basta un bloqueo en curso de cualquier línea para que la cuenta no entre. La salida y el regreso no tocan la línea de seguridad (§5.4) |
| D-63 | Un superadministrador o el administrador de accesos puede restablecer la contraseña de otra cuenta: el sistema genera una provisional, la muestra una sola vez, obliga a cambiarla y cierra todas las sesiones de esa cuenta. Nadie restablece así la suya, la de `SYSTEM` no se restablece, y el administrador de accesos no la restablece a una cuenta protegida (§5.4, §5.12) |
| D-64 | Un proceso automático cierra las asignaciones vencidas con su caducidad como fecha de retiro y motivo `caducidad`. El acceso deja de valer exactamente al caducar, sin esperar al proceso (§5.8) |
| D-65 | La protección del último superadministrador mira hacia adelante: una operación que le quita o le quitará el rol a una cuenta —retirarlo, bloquearla ahora o con un bloqueo programado, registrar la salida que la cierra— procede solo si otra cuenta queda como superadministradora duradera: con el rol vigente y sin caducidad, y sin bloqueos en curso ni programados. Esas operaciones se hacen de a una, para que dos solicitudes simultáneas no dejen al sistema sin ninguno (§5.9) |
| D-66 | La frontera entre empresas la defiende también la base: Row Level Security, habilitada y forzada en toda tabla con `company_id` y en `companies`. La aplicación usa una cuenta de base que no es dueña de nada y fija el alcance al comenzar cada transacción; sin alcance no se ve nada. Los roles de alcance sistema, el arranque y los procesos automáticos actúan con alcance `SYSTEM` explícito (§2.5) |
| D-67 | Los bloqueos de cuenta impiden el cruce con una exclusión `EXCLUDE USING gist`, no con `WITHOUT OVERLAPS`: un bloqueo cancelado no cuenta, y la línea de tiempo que se compara se deduce del motivo y no se guarda. Es una excepción con nombre a la regla de §2.2 (§2.2, §5.4) |
| D-68 | La matriz de permisos llega antes del paso 5, el primero con operaciones de negocio, y acota también lo que lee un rol de alcance sistema. Hasta entonces, el superadministrador y el administrador de accesos leen los datos de todas las empresas, que hoy son empresas, marcas, sedes, ubicaciones y colaboradores (§3, §5.13) |
| D-69 | La `baja` es de la salida: la abre solo el sistema, al registrar la salida del último colaborador activo de la cuenta, y la cierran solo el regreso o un nuevo acceso que vincula la cuenta. Nadie la abre ni la cierra a mano: los bloqueos que se registran a mano son por vacaciones, licencia o seguridad. Así una cuenta nunca queda dada de baja mientras su colaborador sigue activo (§5.4, §5.9) |
| D-70 | Nadie actúa sobre lo suyo. Ninguna cuenta, tampoco la de un superadministrador, se otorga ni se retira roles, se bloquea, levanta sus bloqueos ni se restablece la contraseña por las operaciones de administración: cambiar la propia contraseña tiene su operación. Nadie registra su propia salida ni su propio regreso. La única cuenta que se vincula a sí misma es la del primer superadministrador, mientras no tiene un colaborador activo (§5.2, §5.3, §5.4, §5.7) |
| D-71 | Separación estática de funciones: una cuenta nunca tiene `ADMIN_ACCESOS` y `ADMIN_EMPRESA` vigentes a la vez, para que quien registra los colaboradores de una empresa no sea también quien les da acceso. Lo valida la aplicación al otorgar cualquiera de los dos, porque la base no ve dos asignaciones a la vez (§5.5, §5.7) |
| D-72 | Un rol privilegiado —`SUPERADMIN`, `ADMIN_ACCESOS` o `ADMIN_EMPRESA`, los que hacen protegida una cuenta— se otorga solo a una cuenta que ya cambió su contraseña provisional, para que quien la entregó nunca termine con una cuenta privilegiada en sus manos. El primer superadministrador no se otorga así: lo crea el arranque (§5.2, §5.4, §5.7) |
```

**De dónde sale cada una:**

| Decisión | Contrato |
|---|---|
| D-55 | r4 D4-1, D4-2, D4-5; r5 A y B |
| D-56 | r4 D4-3 |
| D-57 | r4 D4-4, D4-5, D4-6 |
| D-58 | r1 D2, r3 B1, r4 D4-2 y su tabla (elimina r3 B3), r5 B y T8 |
| D-59 | r4 D4-8 (reemplaza r3 B10) |
| D-60 | r1 §5 [DECISION], r2 R2, r3 B12, r4 D4-4, D4-9, C4-2; r5 C (quita la salida reservada al superadministrador de r4 D4-4) y G |
| D-61 | r2 R2, r3 B2, r3 B9 |
| D-62 | r4 D4-9 |
| D-63 | r1 §5 [DECISION], r4 tabla, r5 B |
| D-64 | r3 B8 |
| D-65 | r3 B7, r4 D4-9 |
| D-66 | r2 R1 |
| D-67 | r2 R2, r4 D4-9 |
| D-68 | r4 D4-7; r5, "Not changed, recorded" (la matriz antes del paso 5, según el ajuste 3) |
| D-69 | r4 C4-2 |
| D-70 | r4 C4-1; r5 F y T8 |
| D-71 | r5 D |
| D-72 | r5 E |

D-38, D-41 y D-12 siguen siendo ciertas tal como están: no propongo editarlas.

---

## 4. Historia de la versión

El diccionario no tiene una sección de historia. Cada versión se registra en dos lugares: la línea **Versión** del encabezado (C-1) y el mensaje del commit, en inglés, con una viñeta por decisión, como el de la v1.15 (`1dbd077`). Para la v1.18 propongo:

```text
docs(dictionary): add the access administrator, two suspension tracks and row level security (v1.18)

- D-55: one system role, ADMIN_ACCESOS, gives access, grants and revokes roles, suspends
  accounts and resets passwords in every company; it never acts on its own account, never
  grants or revokes SUPERADMIN, ADMIN_ACCESOS or ADMIN_EMPRESA, and never acts on an account
  that holds any of them (5.5, 5.7)
- D-56: the access administrator assigns roles whose permissions it does not hold, within
  D-55: the only exception to rule 2 of 5.5
- D-57: a company administrator keeps its people (employees, exits, returns) and manages no
  account; the access administrator registers no employee; every actor is linked to an
  active employee (2.5, 5.2, 5.3, 5.4, 5.7)
- D-58: giving access creates the account or links an existing one and never changes its
  roles or password; linking an account closed by an exit closes its baja (5.3, 5.4)
- D-59: ADMIN_EMPRESA is never limited to a brand (5.7)
- D-60: an exit date is never after today in the employee's zone; an exit always removes
  access, protected accounts included, stopped only by the last-superadmin guard; what an
  exit and a return do to the account; a return is never dated after today (5.3, 5.9)
- D-61: a suspension starts now or later and always closes now; closing a scheduled one
  cancels it (5.4)
- D-62: suspensions run on two tracks, employment and security, which overlap only across
  tracks (5.4)
- D-63: a SUPERADMIN or the access administrator resets another account's password to a
  temporary one and closes its sessions (5.4, 5.12)
- D-64: a scheduled process closes expired assignments at their expiry with caducidad;
  access stops exactly at expiry (5.8)
- D-65: the last-superadmin guard requires another durable SUPERADMIN and serializes the
  operations that could remove one (5.9)
- D-66: row level security defends the boundary between companies in the database (2.5)
- D-67: suspensions use an exclusion constraint instead of WITHOUT OVERLAPS (2.2)
- D-68: the permission matrix arrives before step 5 and restricts what a global role
  reads (3, 5.13)
- D-69: the baja belongs to the exit: only the system opens it and only a return or a new
  link closes it (5.4, 5.9)
- D-70: nobody acts on what is their own: their account, their exit or their return; the
  first SUPERADMIN links itself only while it has no active employee (5.2, 5.3, 5.4, 5.7)
- D-71: one account never holds ADMIN_ACCESOS and ADMIN_EMPRESA at the same time (5.5, 5.7)
- D-72: SUPERADMIN, ADMIN_ACCESOS and ADMIN_EMPRESA go only to an account that changed its
  temporary password (5.4, 5.7)
- 3 and 5.13 state what step 3 built and what is still missing
```

---

## 5. Contradicciones que no cubren los cambios de arriba, y lo que no pude ubicar

**Revisado el 16/10/2026** con las aclaraciones C4-1 y C4-2 de r4 y con r5. Los puntos resueltos quedan tachados con su número, para que las referencias sigan valiendo, y dicen cómo se resolvieron. Los puntos 16 a 18 son nuevos.

1. ~~**r4 se contradice sobre la cuenta propia del superadministrador.**~~ Resuelto por C4-1: nadie actúa sobre su propia cuenta, tampoco un superadministrador, y D4-5 gana sobre la fila de la tabla que decía "to anyone but SYS". Queda en la tabla de C-29 y en D-70.
2. ~~**Una `baja` programada al registrar la salida.**~~ Resuelto por C4-2, como se proponía: la salida cancela todo lo programado de la línea laboral, `baja` incluida, y ya nadie programa una `baja` a mano. Queda en C-11, D-60 y D-69.
3. ~~**Plazo de la matriz de permisos.**~~ Resuelto por r5, que lo registra sin cambio a propósito de la opción del menú Roles y permisos: la matriz se debe antes del paso 5, según el ajuste 3, y con ella se acota también lo que lee un rol global, lo que cumple D4-7 con margen. Ajusté C-6, C-35, C-36 y D-68. Confírmalo: D4-7, que aprobaste, decía "antes del paso 7", y la fecha del paso 5 sale de leer el ajuste 3, no de una decisión tuya nueva.
4. **El regreso borra el período anterior.** `Employee.rehire` reemplaza `hire_date` y borra `termination_date` y `termination_reason`. Eso va contra §2.2 ("Una vigencia se cierra, nunca se borra") y contra el espíritu de §5.3 ("Un colaborador no se borra: se registra su fecha de salida"). r1 §8 lo deja fuera de alcance porque "needs a dictionary design". Lo puse como pendiente en §5.13 (C-35) y, si lo validas, como decisión abierta en §14 (C-36).
5. **`employees.termination_reason` existe en el código y no en el diccionario.** Es un `VARCHAR(500)` agregado en V23, y la salida acepta `terminationReason`. §5.3 no lo lista. Por §2.3 hay que decidir quién lo consume: si alguien, entra al cuadro de §5.3 con su porqué; si nadie, se quita o queda como candidato.
6. **Qué cuentas ve el administrador de una empresa.** §2.5 decía "quienes tienen alguna asignación vigente en ella". El código lista las cuentas con cualquier colaborador en la empresa, también los que ya salieron, y lo prueba a propósito (`theAccountOfAnEmployeeWhoLeftStaysInTheListOfItsCompany`). Escribí "las cuentas vinculadas a colaboradores de esa empresa" (C-3). Confirma si deben incluir a los que salieron.
7. **La marca todavía no acota lo que se lee.** §2.5 ("Una empresa y una marca: solo lo de esa marca") y §5.7 ("un asesor vea solo Lima Bikes") lo prometen, pero `AccessScope` y la base filtran solo por empresa. Hoy no hay datos de marca que leer, así que lo ubiqué como pendiente en §5.13. Conviene resolverlo antes del primer dato con marca (vehículos, paso 5).
8. ~~**Una `baja` manual con colaboradores activos.**~~ Resuelto por C4-2: la `baja` no se abre a mano, y abrirla responde como un motivo desconocido. Un incidente sobre una cuenta con colaboradores activos es `seguridad`. Queda en C-16, C-38 y D-69.
9. **Quién crea roles por empresa.** §5.5 ("Administrador de una empresa — Solo en esa empresa") y D-7 dejan al administrador de empresa crear roles de su empresa, pero con D-57 ya no otorga ninguno, y con r5 A ni siquiera `ADMIN_EMPRESA` lo otorga otro que no sea el superadministrador. Los contratos no lo deciden, pero r5 le fija el momento: se decide con la matriz de permisos, antes del paso 5. Mientras tanto, la opción del menú Roles y permisos sigue sin construir y se muestra al administrador de empresa. Está como pendiente en C-35.
10. **Cuántas personas pueden tener `ADMIN_ACCESOS`.** "A single access administrator for the whole group" lo leí como un solo rol para el grupo, no uno por empresa, y no como una sola persona: D4-5 descarta la guardia del último, lo que supone que puede haber varios. Confirma.
11. **No se registra quién retiró un rol.** `user_roles` no tiene `revoked_by`: se sabe quién otorgó (`granted_by`), pero no quién retiró. §5.4 dice de los bloqueos que eso es "la mitad de la pregunta en una revisión de seguridad". No viene de los contratos, pero con el administrador de accesos retirando roles en todas las empresas la pregunta aparecerá. Podría ir como candidato en §5.10.
12. **Un ejemplo previo de §5.7 contradice D-5.** "Un usuario con acceso a dos empresas" usa Lizzo y Lima Bikes como empresas, pero §6.3 y D-5 dicen que son marcas de una misma empresa. Corrección sugerida: Tere con Contabilidad en Millennium Perú y en Millennium Colombia; Alejandro con Operaciones en Millennium Colombia y Asesor de ventas en Millennium Perú, sede San Isidro.
13. **No ubicado: el país del documento del colaborador.** r1 §8 dice que la llave que impediría un tipo de documento de otro país necesita `employees.country_id`. §4.7 ya exige que el tipo sea del mismo país. No verifiqué si hoy lo valida la aplicación, así que no lo puse en §5.13.
14. **No es materia del diccionario** y va a otros documentos (r1 §0 dice que el mapa y las convenciones también registran estas decisiones). Quedan registrados en el mismo cambio que esta revisión, en el repositorio de documentación:
    - D1: `iam` depende de `organization`, e `iam` orquesta la salida y el regreso. Mapa de contextos §4, §5, §6.2 y decisiones 2 y 4.
    - Las rutas y los códigos de error (r2 R4, r3 B12, r4, r5 T2): convenciones §2.2, §7.2, §9.2, §13 fila 16, §14.15 y §14.17. Los códigos nuevos de r5 son `OWN_ACCOUNT_FORBIDDEN`, `ACTOR_EMPLOYEE_REQUIRED`, `ROLE_ASSIGNMENT_DUTIES_INCOMPATIBLE`, `ROLE_ASSIGNMENT_TEMPORARY_PASSWORD` y `EMPLOYEE_REHIRE_DATE_IN_FUTURE`; el diccionario no lista códigos, así que solo entra la regla de cada uno.
    - `EmployeeResource.timeZone` (r5 T4): la API entrega con cada colaborador la zona de §2.6. No cambia el diccionario.
    - Los puertos `CurrentAccessScope` y `SystemAccessScope`, y la separación entre "ve todas las empresas" y "administra una empresa": mapa §5 y `contextos/iam.md`.
    - El bloqueo consultivo de B7, el bloqueo de la fila al leer y cambiar (r3 B11, r5 T6) y el intervalo del proceso de caducidad: patrones de persistencia y `contextos/iam.md`.
    - Todo el frontend.
    - ~~`contextos/iam.md` todavía dice "RLS … primera rebanada del paso 3".~~ Corregido en el mismo cambio.
15. **Tiempo.** Al 16/10/2026, `mm-acc-be` tiene la ronda 4 con C4-1 y C4-2 (V30 a V32, `ACCOUNT_MANAGED_BY_SUPERADMIN`, `ACCESS_SUSPENSION_BAJA_RESERVED`), y la ronda 5 se está construyendo: el código todavía exige un superadministrador para la salida que cierra una cuenta protegida, protege solo las cuentas con `SUPERADMIN` o `ADMIN_ACCESOS` y responde `ACCESS_DENIED` donde r5 pide `OWN_ACCOUNT_FORBIDDEN` y `ACTOR_EMPLOYEE_REQUIRED`. C-5 y C-35 describen el estado una vez integrada la ronda 5.
16. **El rol privilegiado que espera a la contraseña propia alcanza a `ADMIN_EMPRESA`.** r5 E dice que el texto que aprobaste nombraba los roles que hacen protegida una cuenta, y lo extiende a `ADMIN_EMPRESA` "consistently with B". Lo escribí así en C-13, C-29 y D-72. Confirma la extensión.
17. **El restablecimiento entrega la clave provisional al administrador.** Es lo construido y lo que dice D-63. r5 lo registra sin cambio y deja pendiente lo ideal: un enlace de un solo uso al correo de la persona, que necesita un servicio de correo. Cuando llegue, cambian D-63, C-13 y la fila del restablecimiento en §5.12 (C-34). Si quieres dejarlo escrito como pendiente del diccionario, iría como una fila más de §5.13.
18. **La separación estática solo en la aplicación.** r5 D pide validarla en la aplicación y deja abierta una restricción de la base (un *trigger*) si se justifica. D-71 dice "lo valida la aplicación". Si la ronda 5 agrega el *trigger*, esa frase cambia. Lo dirá el informe de la ronda 5.
