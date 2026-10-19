# Orden de implementación — ERP Millennium Mobility

**Actualizado:** 19/10/2026 (paso 3b, la matriz de permisos, construido en `feature/permissions-matrix` sobre `feature/organization-followup`, diccionario v1.23, D-95 a D-105; el paso 3 cerrado; las condiciones de entrada del paso 5; el ajuste 3 y los pendientes transversales al día) · 18/10/2026 (pasos 2 y 3 integrados en `develop`, con sus merges y el despliegue sin errores; el seguimiento en `feature/organization-followup`, con D-93 y D-94; los pendientes transversales al día) · 18/10/2026 (`contextos/location.md` validado; paso 2 cerrado, a falta de la verificación final, con `roles.company_id` en la matriz de permisos, D-81; paso 3 cerrado salvo la matriz, con el registro público con perfil fuera del paso, D-83; cómo se integran los cierres; las condiciones para empezar el paso 5) · 18/10/2026 (paso 1 cerrado, a falta de validar `contextos/location.md`; la sonda contra el backend real como criterio de cierre; el dígito de control, construido) · 16/10/2026 (paso 3 construido, salvo los permisos, el registro público con perfil y el dígito de control) · 15/10/2026 · **Fuente:** diccionario de datos v1.23, §3, §5.6 y D-81 a D-105. Si este documento y el diccionario no coinciden, manda el diccionario.

El orden lo fijan las dependencias entre tablas: nada se construye antes de lo que referencia. Cada paso entrega su contexto completo, del backend (migraciones, dominio y API) al frontend (pantallas y opciones del menú).

## Resumen


| Paso | Bloque del diccionario                                | Contexto              | Depende de                       | Estado                   |
| ---- | ----------------------------------------------------- | --------------------- | -------------------------------- | ------------------------ |
| 0    | 2 · Seguridad, en parte, y el núcleo común            | `shared`, `iam`       | —                                | Construido               |
| 1    | 1 · Geografía                                         | `location`            | —                                | Cerrado (Perú)           |
| 2    | 3 · Empresas                                          | `organization`        | 1 y 2                            | Cerrado e integrado en `develop` |
| 3    | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | 2 y 3                            | Cerrado e integrado; su seguimiento, en `feature/organization-followup`, pendiente de integrar |
| 3b   | 2 · La matriz de permisos                             | `iam`, `shared`       | 3                                | Construido en `feature/permissions-matrix`, pendiente de integrar |
| 4    | 4 · Catálogos de flota                                | `vehiclecatalog`      | 3                                | En curso                 |
| 5    | 6 · Vehículo y estados                                | `fleet`               | 4                                | Pendiente                |
| 6    | 11 · Bitácora de cambios                              | `shared`              | 6                                | Pendiente                |
| 7    | 7 · Compras, que incluye el 5                         | `procurement`         | 6                                | Pendiente                |
| 8    | 8 · Pagos                                             | `procurement`         | 7                                | Pendiente                |
| 9    | 9 · Habilitación                                      | `fleet`, `shared`     | 6                                | Pendiente                |
| 10   | 10 · Fondos                                           | `fleet`               | 6 y las cargas de archivos del 9 | Pendiente                |
| 11   | Reportes                                              | Modelo de lectura     | 8, 9 y 10                        | Pendiente                |
| —    | 12 · Ventas y contratos                               | —                     | 6 y 11                           | Fuera del alcance actual |


La columna *Depende de* usa los números de bloque del diccionario.

## Dependencias

```
1 Geografía ─┐
             ├─► 3 Empresas ─► 3b Colaboradores ─► 4 Catálogos de flota ─► 6 Vehículo ─┬─► 11 Bitácora
2 Seguridad ─┘                                                                         ├─► 7 Compras ─► 8 Pagos ──┐
                                                                                       ├─► 9 Habilitación ────────┼─► Reportes
                                                                                       └─► 10 Fondos ─────────────┘
```

Desde el vehículo hay dos ramas independientes: compras y pagos (pasos 7 y 8), y habilitación y fondos (pasos 9 y 10). Se pueden alternar según la prioridad del negocio.

## Los pasos



### Paso 0 · Seguridad, en parte, y el núcleo común — construido

- **Tablas:** `users`, `roles` y una versión reducida de `user_roles`, más las sesiones (`spring_session`).
- **Pantallas:** inicio de sesión, registro público (se enciende con una variable de entorno), cambio de contraseña y Seguridad › Usuarios.
- **Lo que falta del bloque:** el paso 3 completó las asignaciones con su alcance, los bloqueos, la caducidad y dar acceso desde el colaborador, que reemplazó a "Nuevo usuario"; el paso 3b trajo los permisos, la matriz y los roles propios. Queda el registro público con perfil, fuera del paso 3 (D-83, pendientes transversales).



### Paso 1 · Geografía y localización — cerrado para Perú

- **Estado (18/10/2026):** cerrado para Perú, el único país que opera (D-5). El responsable validó [contextos/location.md](contextos/location.md) el 18/10/2026, después de las correcciones que pidió la auditoría de `organization`, y se integró en `develop` dentro de `feature/organization-closure` ese mismo día (paso 2, "Integración"). Backend y frontend tienen monedas, países con sus escalones, divisiones, direcciones y tipos de documento tributario, con su API, sus pruebas y la ficha del país; la dirección elige su división en cascada. V24 carga el ubigeo del INEI, edición 2022, con 1 891 distritos, y cada distrito creado por ley después entra con su propia migración cuando tiene código oficial (D-77). El cierre, en `feature/location-closure` (backend sobre `ee4df37`, con V36 a V39; frontend sobre `a8ab115`), atendió los 60 hallazgos de la auditoría del 17/10/2026 con las decisiones D-75 a D-80 del diccionario v1.20:
  - el dígito de control lo calcula `location` y lo aplica `organization` en empresas y colaboradores; hoy, el del RUC (D-75), y el DNI no lo tiene (D-76);
  - un colaborador cuyo tipo de documento se desactivó se sigue editando (LOC-R36);
  - cada regla de una dirección responde su propio código, y las ubicaciones corrigen o mudan su dirección por su subrecurso, como las sedes (D-79);
  - los identificadores se guardan sin guiones, espacios ni puntos (§2.2, V39); las zonas horarias son nombres IANA de región; un tipo de documento declara una longitud de 1 a 20 (D-78, V38);
  - un duplicado en carrera responde el código de su recurso y ya no `RESOURCE_CONFLICT`; un error inesperado ya no expone su mensaje (convenciones §13 fila 14); la API responde en español por omisión;
  - las pruebas que faltaban contra la base, de lectura con los roles sembrados y de extremo a extremo de Ubicaciones, y la primera sonda contra el backend real.

  Verificación final: 18/10/2026, sobre `feature/organization-closure` —backend `26bc1e6`, 93 commits sobre `ee4df37`, con las migraciones V36 a V44; frontend `0a40211`, 119 commits sobre `a8ab115`—. La suite completa del backend: 1 576 pruebas sin fallos. Las unitarias del frontend: 1 273 sin fallos. La suite e2e completa: 641 pasan y 22 se omiten por proyecto, en tres corridas seguidas sin fallos. Las sondas contra el backend real: `location`, 8 de 8; `organization`, 8 de 10, con 2 omitidas porque la base tiene una sola empresa (el vínculo entre empresas de D-82 y la frontera del administrador de empresa). Las sondas corrieron sobre `erp_mm_db_verify`, por decisión del responsable, porque `erp_mm_db_live` todavía no existía. Las unitarias y las e2e corrieron sobre `68936ad`; los dos commits siguientes (`e9759d2`, `0a40211`) solo tocan los specs de la sonda. El detalle está en [contextos/location.md](contextos/location.md), sección 6, y en la [revisión del 18/10/2026](retroalimentacion/2026-10-18-location.md). Lo que es de Colombia y México y los demás pendientes con disparo quedan en los pendientes transversales.
- **Tablas:** `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses`, `tax_id_types` (D-44).
- **Pantallas:** Catálogos › Monedas y Países. Cada país abre su ficha, con sus datos arriba y, debajo, en pestañas, su estructura territorial (escalones y divisiones, D-45) y sus documentos tributarios; la misma ficha registra el país y corrige sus datos. Las direcciones se llenan dentro de cada formulario que las usa; no tienen opción propia.
- **Datos:** el archivo oficial de divisiones territoriales (ubigeo del INEI), edición 2022, cargado por V24 desde `scripts/ubigeo/` del backend; los distritos creados por ley después entran cuando el INEI o SUNAT publican su código, y la lista se revisó al 17/10/2026 (D-2, D-77). Falta Santa Rosa de Loreto, que espera el suyo.



### Paso 2 · Empresas — cerrado

- **Estado (18/10/2026):** cerrado e integrado en `develop` (abajo, "Integración"). Empresas, Marcas comerciales, Sedes y Ubicaciones están construidas en el backend y en el frontend, con su opción del menú, y la empresa, la sede y la ubicación llevan su dirección anidada. `roles.company_id` ya no es de este paso: llega con la matriz de permisos, antes del paso 5, porque un rol propio de una empresa solo tiene sentido con permisos que marcarle (D-81). El cierre, en `feature/organization-closure` (49 commits del backend y 55 del frontend sobre `feature/location-closure`, con V40 a V44), partió de los 82 hallazgos confirmados de la auditoría del 18/10/2026 y de las decisiones D-81 a D-92 del diccionario v1.21; lo que no se corrigió quedó con su paso:
  - la empresa nombra su país y su moneda por su código (D-13), y el tipo de una ubicación viaja en minúsculas;
  - a una empresa, una sede o una marca inactiva no se le cuelga nada nuevo (D-84), y el superadministrador corrige el documento tributario de una empresa (D-85);
  - los códigos propios conservan sus guiones (D-86), una sede tiene a lo sumo una ubicación propia, que se reactiva después de su sede (D-90), y la longitud de un número sin cambios no se vuelve a medir (D-91);
  - empresas, sedes y marcas se bloquean antes de cambiarlas, y lo que se les cuelga lee el padre con `FOR SHARE`;
  - las pruebas de autorización de cada escritura, las claves naturales en una carrera, que ya no suponen una base vacía, y la sonda contra el backend real de las pantallas de `organization`.

  Verificación final: 18/10/2026, sobre `feature/organization-closure` —backend `26bc1e6`, 93 commits sobre `ee4df37`, con las migraciones V36 a V44; frontend `0a40211`, 119 commits sobre `a8ab115`—. La suite completa del backend: 1 576 pruebas sin fallos. Las unitarias del frontend: 1 273 sin fallos. La suite e2e completa: 641 pasan y 22 se omiten por proyecto, en tres corridas seguidas sin fallos. Las sondas contra el backend real: `location`, 8 de 8; `organization`, 8 de 10, con 2 omitidas porque la base tiene una sola empresa (el vínculo entre empresas de D-82 y la frontera del administrador de empresa). Las sondas corrieron sobre `erp_mm_db_verify`, por decisión del responsable, porque `erp_mm_db_live` todavía no existía. Las unitarias y las e2e corrieron sobre `68936ad`; los dos commits siguientes (`e9759d2`, `0a40211`) solo tocan los specs de la sonda. El detalle está en [contextos/organization.md](contextos/organization.md), sección 6, y en la [revisión del 18/10/2026](retroalimentacion/2026-10-18-organization.md).
- **Integración (18/10/2026):** `feature/organization-closure` se integró entera en `develop` y se subió, en el backend y en el frontend, con `feature/location-closure` dentro: la última versión de V39 (`aade406`) y la pantalla de LOC-R36 (`9842ea8`) estaban solo en ella. En el backend, el merge es `96b2d04`, con `840e446` (documentación de la sonda) después de la cabeza verificada `26bc1e6`; en el frontend, `64e8565`, con `aef5067` (documentación de la sonda) después de `0a40211`. El responsable confirmó que el despliegue de `develop` terminó sin errores. Como V40, V41 y V43 se detienen si una fila guardada rompe su regla, ninguna la rompió, y V39 no encontró ninguna colisión. No consta que se corrieran los inventarios antes del push (`scripts/database/inventory-text-identifiers.sql` e `inventory-organization-checks.sql`, README del backend).
- **Seguimiento (18/10/2026):** la auditoría posterior a la integración, de solo lectura, no encontró ningún bloqueante. Sus hallazgos confirmados y las decisiones D-93 y D-94 del diccionario v1.22 se corrigieron en `feature/organization-followup`: 16 commits del backend sobre `96b2d04` y 13 del frontend sobre `64e8565`, sin migraciones, todavía sin integrar. Cierra:
  - quién lee los colaboradores: solo un rol global y quien administra la empresa (D-93, que resuelve O19);
  - nada se reactiva ni regresa bajo una empresa o una sede inactiva (D-94), con "Activar" y "Reincorporar colaborador" deshabilitados y su motivo;
  - la fecha de ingreso después de la salida y los números hechos solo de separadores, cada uno con su código en su campo; el código de la sede en las exportaciones;
  - las vistas de empresa, sede y marca con campos propios (O46); `hand-over-database.sql` entrega también las funciones de Row Level Security; se fueron lecturas que nadie llamaba;
  - en el frontend, el diálogo de nueva ubicación que se quedaba en la primera empresa, los avisos cuando no cargan los países o los tipos de documento, y el simulador e2e que ahora rechaza lo que rechaza el backend.

  **Verificación final de `feature/organization-followup`:** 18/10/2026, sobre el backend `e460819` (16 commits sobre `96b2d04`) y el frontend `8df0abc` (13 commits sobre `64e8565`), sin migraciones. La suite completa del backend: 1 623 pruebas sin fallos. Las unitarias del frontend: 1 313 sin fallos, con `tsc` limpio. La suite e2e completa: 662 pasan y 22 se omiten por proyecto. Después, una revisión adversarial de solo lectura de las dos ramas —cinco lentes y dos verificadores por hallazgo— no encontró bloqueantes ni importantes, y sus siete menores se corrigieron. En el backend: un nombre hecho solo de caracteres de control responde `400 VALIDATION_ERROR` en su campo, igual en el alta y en la edición (`98fdd55`); el README dice que una base entregada antes de `c41fecd` corre el traspaso otra vez (`3c2bf88`); y `CompanySharedLockTests` prueba en PostgreSQL que el bloqueo compartido de una empresa por su identificador frena un cambio de su estado (`e460819`). En el frontend: un Excel rechazado lee el cuerpo de su error, así el `403` de los colaboradores dice su propio texto (`755f3e1`); el aviso de los tipos de documento se olvida cuando otra empresa lee los suyos (`51bae7d`); y el simulador e2e valida el cuerpo antes del rol, como Spring, y corrige un colaborador en el orden del backend (`8df0abc`). Un commit intermedio del frontend dejaba en rojo tres specs de `location`; se fusionó con el que los arreglaba, el árbol final es idéntico y `755f3e1` pasa solo. La sonda contra el backend real no se volvió a correr: corre con un superadministrador, al que D-93 no le cambia nada. Va a `develop` con ella; como no trae migraciones, no necesita inventarios. El detalle está en [contextos/organization.md](contextos/organization.md), sección 6, y en la [revisión de seguimiento](retroalimentacion/2026-10-18-organization-seguimiento.md).
- **Tablas:** `companies`, `commercial_brands`, `branches`, `locations`. `roles.company_id` pasó a la matriz de permisos (D-81), y `tax_id_types` al paso 1 (D-44).
- **Pantallas:** Configuración › Empresas (solo superadministrador), Sedes, Marcas comerciales y Ubicaciones. Qué países operan lo muestra Empresas filtrada por país (D-3).
- **Datos:** razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales.



### Paso 3 · Colaboradores y accesos, y el resto del bloque 2 — cerrado

- **Estado (19/10/2026):** cerrado. La matriz de permisos (D-68), lo único que le faltaba, llegó con el paso 3b (abajo), construido en `feature/permissions-matrix` sobre `feature/organization-followup`; las dos ramas se integran juntas, en ese orden.
- **Estado (18/10/2026):** cerrado salvo la matriz de permisos (D-68), que llega antes del paso 5. El registro público con perfil (D-32) no es de este paso: sigue apagado y se construye antes de encender el registro público en cualquier entorno (D-83). El cierre de `organization` (paso 2) agregó al paso 3 la cuenta existente solo de la misma persona, con la confirmación del superadministrador atada a los titulares que vio, y la desvinculación, que funciona como una salida (D-82); el registro de esas decisiones (`employee_account_link_events`, V44); la fecha de ingreso no posterior a hoy (D-88); el superadministrador sin vincular, que registra colaboradores pero no salidas, regresos ni desvínculos (D-89); la caducidad de un rol como el día completo en la zona de la empresa (D-87); y en qué empresas trabaja cada cuenta, en la sesión y en la lista de cuentas. Su verificación y su integración son las del paso 2. El seguimiento (paso 2) le agrega quién lee los colaboradores (D-93) y el regreso solo a una empresa y una sede activas, con la cuenta cerrada si no regresa (D-94).
- **Estado (16/10/2026):** construido en `feature/access-model`, en el backend y en el frontend, con las seis rondas del modelo de acceso. Lo construido:
  - colaboradores, con su salida y su regreso, que nunca tienen fecha posterior a hoy en la zona del colaborador;
  - dar acceso desde el colaborador (`POST /access-grants`), que crea la cuenta o vincula la que la persona ya tiene, sin tocar sus roles ni su contraseña;
  - asignaciones por empresa, sede y marca, con su caducidad y un proceso que cierra las vencidas;
  - bloqueos de cuenta en dos líneas de tiempo, la laboral y la de seguridad, y el restablecimiento de contraseña;
  - la salida que retira los roles de esa empresa y cierra la cuenta cuando era el último colaborador activo, siempre, con el último superadministrador como único freno;
  - el administrador de accesos (`ADMIN_ACCESOS`) y la separación de funciones: el administrador de empresa registra a su gente y no maneja cuentas, el de accesos maneja cuentas y no registra colaboradores, y una misma cuenta no tiene los dos roles;
  - la frontera entre empresas en la aplicación (`404` fuera del alcance) y en la base (Row Level Security, con la aplicación conectada con una cuenta que no es dueña de nada);
  - `iam` depende de `organization` y nunca al revés (mapa de contextos, decisiones 2 y 4).
  
  Migraciones V26 a V32. El detalle está en [contextos/iam.md](contextos/iam.md), [contextos/organization.md](contextos/organization.md) y la [revisión del 16/10/2026](retroalimentacion/2026-10-16-iam.md). Las decisiones que se tomaron en el camino están en el diccionario v1.18 (D-55 a D-72), validado el 17/10/2026.
- **Verificación** (sobre el backend `6a7d9b1` y el frontend `1a341ab`): 926 pruebas del backend, sin fallos, errores ni omitidas, conectadas como `erp_mm_app` sobre una `erp_mm_db_verify` recreada y migrada por `erp_mm_owner` de V1 a V32; 877 pruebas unitarias del frontend en 88 archivos; build inicial de 489,11 kB, bajo el presupuesto de 500 kB; `lint:styles` sin observaciones; 425 e2e en verde y 22 omitidas a propósito, sin fallos. La primera sonda contra el backend real (ronda 4) coincidió en 210 de 211 comprobaciones HTTP —la restante fue el orden de la propia sonda, no el backend— y en las 40 de la frontera entre empresas, y el recorrido con los tres roles no dio errores de página. Sonda final (rondas 5 y 6): 129 de 129 comprobaciones de la política coincidieron, y el navegador sin simulaciones se comportó como dice el contrato; quedan cuatro diferencias menores, anotadas en la [revisión del 16/10/2026](retroalimentacion/2026-10-16-iam.md).
- **Lo que falta del paso:** nada. `permissions` y `role_permissions`, con `roles.company_id` (D-81), que esperaban la matriz de permisos (D-68, ajuste 3), llegaron con el paso 3b el 19/10/2026.

  El registro público con perfil, que figuraba aquí, salió del paso (D-83): queda en los pendientes transversales.
  
  El dígito de control de los documentos tributarios, que figuraba aquí, se construyó con el cierre del paso 1 (D-75; [location.md](contextos/location.md), LOC-R35, y [organization.md](contextos/organization.md), ORG-R04).
- **Tablas:** `employees`; `user_roles` con su alcance por empresa, sede y marca; `user_access_suspensions` y `employee_account_link_events`. `employees` es de `organization`, y las demás de `iam` (D-92). `permissions` y `role_permissions` son del paso 3b.
- **Pantallas:** Configuración › Colaboradores, con las acciones de cuenta según el rol de la sesión; Seguridad › Usuarios, para el superadministrador y el administrador de accesos. Seguridad › Roles y permisos es del paso 3b.
- **Datos:** colaboradores reales.
- **Nota:** `advisor_assignments` figura en el bloque 3b, pero se construye con ventas (bloque 12), porque apunta a los contratos.



### Paso 3b · La matriz de permisos — construido, pendiente de integrar

- **Estado (19/10/2026):** construido en `feature/permissions-matrix`, en el backend y en el frontend, sobre `feature/organization-followup`, el mismo día en que el diccionario v1.23 fijó la matriz (§5.6, D-95 a D-105). Lo construido:
  - el catálogo de permisos y lo que incluye cada rol, con su historia (V45): los nueve permisos de los pasos 1 a 4 y las 29 marcas de los siete roles del sistema, firmadas por `SYSTEM`; cada paso siguiente agrega los suyos con una migración de `iam` (D-95, D-96);
  - los roles propios, generales o de una empresa, que crea solo el superadministrador y a los que marca solo los permisos que posee; un rol del sistema no se renombra, no se desactiva ni cambia su matriz (D-97, D-98); `roles.company_id` e `is_active`, con Row Level Security en `roles` —un rol de una empresa lo ve quien la alcanza y, siempre, la cuenta que lo tiene— y en `role_permissions` a través del rol;
  - toda operación de negocio y toda lectura de datos exigen un permiso (`@PermissionRequired`, autoridades `PERM_`), y el servicio decide dónde actúa con la asignación que lo trae (D-99): los catálogos generales y de flota, la estructura de las empresas y los colaboradores de los pasos 1 a 4, con la fila mínima para todos y los datos personales y la exportación solo con su permiso (D-105); las cuentas, los accesos, las escrituras de colaboradores y los roles siguen por rol (§5.8 regla 7);
  - `permissions` en la sesión, y la cuenta leída una sola vez por petición;
  - en el frontend, Seguridad › Roles y permisos de vuelta en el menú, con la lista por empresa, la ficha con la matriz por módulo y "n de m", los roles del sistema de solo lectura, el diálogo de roles con solo los roles activos de la empresa, y Colaboradores sin los datos personales que la sesión no ve.

  Backend: 15 commits sobre `e460819` (`f0ee470` V45, `a0f6cd7` `shared`, `7851f6d`, `3dc4dc8` y `039797b` `iam`, `9f6c1d8` `location`, `a39e111` `vehiclecatalog`, `cef8bc8` `organization`, `74714bd` el README; después de la revisión, `01903ba`, `77bd0c3`, `968289c`, `2dc2fb3`, `bc765d6` y `2cd0314`). Frontend: 12 commits sobre `8df0abc` (`91ecc32`, `f7c08fe`, `aabc5e9`, `ac9fbb6`, `56041fe`, `45154fa`, `b621d74`; después de la construcción `8e76964`, y después de la revisión `621558a`, `6fdbb90`, `a872459` y `0041574`). Al terminar la construcción: la suite completa del backend, 1 718 pruebas sin fallos; las unitarias del frontend, 1 394; la suite e2e, 719 pasan y 22 se omiten por proyecto. El detalle está en [contextos/iam.md](contextos/iam.md), sección 6, y en la [revisión del 19/10/2026](retroalimentacion/2026-10-19-permisos.md).

  **Verificación final de `feature/permissions-matrix`:** 19/10/2026, sobre el backend `2cd0314` (15 commits sobre `e460819`, con la migración V45) y el frontend `0041574` (12 commits sobre `8df0abc`). Dos revisiones adversariales de solo lectura del código construido —cinco lentes en el backend y cuatro en el frontend, con dos verificadores por hallazgo— no encontraron bloqueantes; confirmaron en el backend 1 importante y 3 menores, y en el frontend 2 importantes y 8 menores, todos corregidos. En el backend: `01903ba` y `77bd0c3` (una concesión de sede ya no escribe empresas, sedes, marcas ni ubicaciones: `AccessScope.actsOnWholeCompany`, D-99), `bc765d6` (un código de permiso nulo o en blanco responde `400 VALIDATION_ERROR`), `968289c` y `2cd0314` (el esquema del error nombra los dos errores que llevan contexto) y `2dc2fb3` (las pruebas de roles usan códigos propios y leen por identificador público). En el frontend: `621558a` (las marcas sin guardar sobreviven a un renombrado o a un cambio de estado desde la ficha, y "Descartar" pregunta antes), `6fdbb90` (el diálogo de otorgar suelta la empresa que el rol elegido ya no admite y ofrece un rol propio solo donde la cuenta trabaja), `a872459` y `0041574` (el simulador comprueba un rol nuevo en el orden del backend, siembra roles propios para el administrador de empresa, y los specs hacen lo que su nombre dice). La suite completa del backend, sobre `bc765d6`: 1 743 pruebas sin fallos; `2cd0314` solo cambia un Javadoc. Las unitarias del frontend: 1 402 sin fallos, con `tsc` limpio. La suite e2e completa: 719 pasan y 22 se omiten por proyecto. La sonda contra el backend real no tiene casos de roles todavía (pendiente, abajo).
- **Integración:** con `feature/organization-followup`, en ese orden, porque sale de ella; la verificación final de la matriz está hecha (19/10/2026, arriba) y las dos esperan la validación del responsable. No necesita inventarios: V45 solo crea tablas y columnas y siembra el catálogo y la matriz, y no reescribe ninguna fila guardada.
- **Tablas:** `permissions`, `role_permissions`, `roles.company_id` e `is_active` (V45). Todas de `iam`.
- **Pantallas:** Seguridad › Roles y permisos (`/iam/roles` y la ficha `/iam/roles/{id}`), para `SUPERADMIN`, que escribe, y `ADMIN_ACCESOS` y `ADMIN_EMPRESA`, que leen (D-98).
- **Datos:** la matriz de permisos por rol, fijada el 19/10/2026 (§5.6, D-95). La única celda que espera al negocio es quién registra la entrega al cliente (`stock.entregar`; procesos y pantallas, pregunta 4), que llega con el paso 5 y hasta entonces es del administrador de la empresa.



### Paso 4 · Catálogos de flota — en curso

- **Estado (17/10/2026):** construido en `develop`, backend y frontend, rama `feature/vehicle-catalog`. El backend tomó el catálogo de `feature/vehiclecatalog` (johanncamiloaga, coautor de esos commits) y lo adaptó al diccionario v1.19 (D-73, D-74, reglas de §7.3 y §7.5): migración V35 con las semillas `AUTO`, `MOTO`, `TRIMOTO`, `GPS` y `PANTALLA`, rutas sin prefijo de contexto, aplicabilidad que se desactiva en vez de borrarse. El resto de esa rama —bloqueos, roles y permisos— no se integró: lo reemplaza el paso 3, y los permisos llegaron con el paso 3b: los catálogos de flota se leen con `catalogos.ver` y se escriben con `catalogos.gestionar_flota` (`a39e111`, en `feature/permissions-matrix`). Falta para cerrar el paso: los modelos reales conciliados (§7.3) y `contextos/vehiclecatalog.md` con sus reglas. Queda abierta la tensión entre §7.5 y §11.9 sobre si la pantalla bloquea la disponibilidad; la semilla sigue §7.5.
- **Tablas:** `vehicle_types`, `vehicle_brands`, `vehicle_models`, `colors`, `equipment_types` y `equipment_type_vehicle_types`.
- **Pantallas:** Catálogos › Tipos de vehículo, Marcas y modelos (una sola pantalla: marcas a la izquierda, los modelos de la marca elegida a la derecha), Colores y Equipamiento.
- **Datos:** los modelos reales, conciliados entre los dos catálogos entregados, que no coinciden entre sí (§7.3).



### Paso 5 · Vehículo y estados

- **Para empezarlo** (condiciones de entrada, auditoría de `organization` del 18/10/2026):
  - ~~la matriz de permisos (D-68), con `permissions`, `role_permissions` y `roles.company_id` (D-81)~~: cumplida con el paso 3b (19/10/2026, D-95 a D-105); las operaciones de negocio exigen un permiso, no un rol, y el paso 5 trae con su migración de `iam` `catalogos.gestionar_estados` y los permisos de `stock`, con sus marcas en la matriz;
  - ~~qué lee cada rol~~: resuelto por la matriz: un rol global lee lo que ella le da (D-68), y cada rol ve de los colaboradores lo que D-105 fija. Lo que sí es de este paso: la marca de una asignación acota lo que se ve y se toca de los datos que pertenecen a una marca, en el código y no en la base (D-102), y los vehículos son el primer dato con marca; cada camino de estado nombra el permiso que exige, y un rol de sede actúa sobre las unidades de la ubicación propia de su sede (D-100, D-101); quién registra la entrega al cliente (`stock.entregar`) espera al negocio (procesos, pregunta 4);
  - que la marca y la ubicación de un vehículo sean de su empresa: la regla de §8.1 del diccionario y la llave compuesta del vehículo hacia su ubicación, que puede apuntar a `locations_id_company_uq (id, company_id)`, ya agregada por V41 (G3.1);
  - la lectura en lote de las ubicaciones de una empresa por la fachada de `organization`, que `fleet` va a necesitar (G3.2);
  - los casos de la sonda de `organization` que no corrieron —el vínculo entre empresas de D-82, la frontera del administrador de empresa, y la salida y el regreso—, sobre `erp_mm_db_live` con una segunda empresa y la cuenta de verificación vinculada a un colaborador.
- **Tablas:** `vehicle_statuses`, `status_change_reasons`, `vehicles`, `vehicle_status_history`, `vehicle_mileage_readings`.
- **Pantallas:** Logística › Inventario (la ficha del vehículo reúne su historial de estados y su kilometraje); Catálogos › Estados y motivos.



### Paso 6 · Bitácora de cambios

- **Tabla:** `change_log`.
- **Pantalla:** Seguridad › Bitácora de cambios; cada empresa ve la suya.
- **Por qué va aquí y no al final:** depende solo del vehículo, y desde el paso 5 ya hay cambios delicados —corregir una placa, un VIN o el monto de una factura— que tienen que quedar registrados.



### Paso 7 · Compras

- **Tablas:** `suppliers`, `purchase_order_types`, `purchase_order_templates`, `purchase_orders` y sus líneas, `purchase_invoices` y sus líneas, `credit_notes` y su reparto entre unidades.
- **Pantallas:** Configuración › Proveedores y Plantillas de OC (los tipos de orden se gestionan dentro); Logística › Órdenes de compra, Facturas y Notas de crédito.
- **Datos:** proveedores.



### Paso 8 · Pagos

- **Tablas:** `supplier_payments`, `payment_applications`, `payment_allocations`. El saldo a favor con cada proveedor se calcula; no se guarda.
- **Pantalla:** Logística › Pagos a proveedores.



### Paso 9 · Habilitación

- **Tablas:** los campos de habilitación de `vehicles`, `document_types`, `vehicle_documents`, `vehicle_equipments`, `vehicle_keys`, `vehicle_tax_declarations` e `import_batches` (del núcleo común).
- **Pantallas:** Logística › Habilitación y Cargas de archivos; Catálogos › Tipos de documento.
- **Nota:** el estado de habilitación se calcula (§11.9). Las cargas de archivos registran toda carga masiva: la declaración del impuesto, la garantía mobiliaria y la migración inicial.



### Paso 10 · Fondos

- **Tablas:** `funds` y `vehicle_fund_assignments`.
- **Pantalla:** Configuración › Fondos. Las cesiones se alimentan con el archivo de garantía mobiliaria, que llega por las cargas del paso 9, y se ven en la ficha del vehículo.



### Paso 11 · Reportes

- **Qué incluye:** los indicadores calculados (§10.5: diferencia por inicial y brecha de días) y el exportable de stock, armados en una capa de lectura que no pertenece a ningún contexto y nunca escribe.
- **Pantalla:** Logística › Reportes.
- **Nota:** la validación de la diferencia por inicial contra el contrato del cliente espera a ventas (bloque 12).



### Fuera del alcance · Ventas y contratos (bloque 12)

Contratos y asignación de asesoras (`advisor_assignments`).

## Ajustes respecto del cuadro de diseño

El diccionario ordena los bloques por sus dependencias. La implementación lo sigue, con cuatro ajustes:

1. **La bitácora va apenas después del vehículo**, no al final.
2. `advisor_assignments` **se construye con ventas**, aunque figure en el bloque 3b.
3. **El bloque 2 se terminó con la matriz de permisos, en el paso 3b.** El paso 3 se construyó sin `permissions` ni `role_permissions`, porque faltaba el dato: qué permisos lleva cada rol. El 19/10/2026 la matriz quedó fijada (§5.6, D-95 a D-105) y el paso 3b la construyó el mismo día, antes del paso 5: desde él, las operaciones de negocio y las lecturas de datos exigen un permiso, las cuentas y los accesos siguen por rol (§5.8 regla 7), un rol global lee lo que su matriz le da (D-68) y existen los roles propios de una empresa, que crea solo el superadministrador (D-81, D-98). Cada paso siguiente trae sus permisos y sus marcas con una migración de `iam`.
4. **Desde el vehículo hay dos ramas independientes**, que se pueden alternar.






## Pendientes transversales


| Qué | Prioridad |
| --- | --- |
| Historia de los reingresos: hoy el regreso de un colaborador reemplaza su fecha de ingreso y borra la de salida, y el período anterior se pierde. Necesita un diseño en el diccionario (§5.3, §5.13 y §14) | Cuando el diccionario lo diseñe, antes de cargar los colaboradores reales: desde entonces, cada regreso borraría un período que ya no se recupera |
| La marca de una asignación acota dónde se otorga un rol, pero todavía no lo que se lee: hoy ningún dato que se lee pertenece a una marca. Decidido el 19/10/2026 (D-102): acota lo que se ve y lo que se toca de los datos que pertenecen a una marca, aplicada en el código y no en la base; un rol de empresa sin marca ve todas | Con el paso 5: los vehículos son el primer dato que pertenece a una marca |
| Restablecer la contraseña con un enlace de un solo uso al correo de la persona, en vez de mostrarle la provisional al administrador | Cuando exista el envío de correos |
| Registro público con perfil (D-32): elegir empresa y sede sin sesión y crear el colaborador y la cuenta juntos; hoy crea solo la cuenta y viene apagado. Quedó fuera del paso 3 (D-83) | Antes de encender el registro público en cualquier entorno, con la forma que fijen las [convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §14.16 |
| La sesión guarda el hash de la contraseña ([iam.md](contextos/iam.md), §6 b, O58) | Media. **(propuesta)** Antes del paso 5, y a más tardar antes del primer entorno con usuarios reales |
| Un correo de más de 255 caracteres devuelve error 500 en vez de 400 (O58) | Baja. **(propuesta)** Con el próximo cambio de los recursos de entrada de `iam`, y a más tardar antes del paso 5 |
| Los casos de la sonda de `organization` que no corrieron en su corrida final: el vínculo entre empresas de D-82, la frontera del administrador de empresa, y la salida y el regreso ([contextos/organization.md](contextos/organization.md), §6 b) | Antes del paso 5, sobre `erp_mm_db_live` con una segunda empresa y la cuenta de verificación vinculada a un colaborador. La base la crea el responsable |
| Integrar `feature/organization-followup` y, sobre ella, `feature/permissions-matrix` en `develop`, juntas y en ese orden (paso 2, "Seguimiento"; paso 3b). Sin inventarios: el seguimiento no trae migraciones y V45 solo crea y siembra | La verificación final del seguimiento está hecha (18/10/2026, paso 2); la de la matriz, pendiente (paso 3b). Cuando esté hecha y el responsable valide `contextos/organization.md`, `contextos/iam.md` y el texto de D-93 a D-105 (diccionario v1.22 y v1.23) |
| Una pantalla de solo lectura de `employee_account_link_events` para el superadministrador; hoy no tiene ruta de lectura (§5.4) | Cuando el responsable la pida |
| El colaborador en un contexto propio (D-92) | Cuando lleguen los contratos laborales, la planilla o la asistencia |
| Separar en cada entorno la cuenta de Flyway, dueña de las tablas, de la cuenta de la aplicación, que lee, inserta y actualiza, y solo borra en `spring_session*`. El código y las pruebas ya funcionan así: las pruebas corren conectadas como `erp_mm_app` (README del backend, "Cuentas de la base" y "Pruebas"). Falta correr los scripts (`scripts/database/create-roles.sql`, una vez por clúster, y `hand-over-database.sql`, una vez por base ya migrada) y fijar `FLYWAY_USER`, `FLYWAY_PASSWORD`, `DATABASE_USER` y `DATABASE_PASSWORD`. Con `postgres`, que es superusuario, la base no aísla las empresas (V29) | En cada entorno, antes de usar roles de empresa o de sede en él |
| Zona de los momentos en las exportaciones con alcance de empresa o de sede: cada controlador le pasa a `SpreadsheetExporter` la zona de ese alcance ([convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §8.1). Hoy ninguna exportación de empresa o de sede lleva momentos; la de cuentas los da en `America/Lima`, porque una cuenta no es de una empresa | Con la primera exportación de empresa o de sede que lleve momentos |
| Fijar los decimales de una moneda cuando un monto ya la usa (§4.1): `location` publica un puerto (SPI) que implementan los contextos con montos, para saber si alguno usa la moneda sin depender de ellos | Antes del paso 7 o del 9, el que vaya primero: los dos traen los primeros montos |
| Los tipos de cambio por día (`exchange_rates`) y el tipo de tasa que propone cada país para sus compras (§4.8, D-46); la moneda de grupo, con los reportes | Justo antes del paso 7; la moneda de grupo, con el paso 11 |
| Acotar la tabla de direcciones por empresa, con Row Level Security, como las demás tablas de empresa (§2.5, D-66, D-80). Cómo se acota se diseña con el paso | En el paso 7, antes de guardar la primera dirección de un proveedor |
| Fechas en pantalla (§2.6): deben verse en la zona de la sede; sin sede, en la del país de la empresa; sin empresa, en `America/Lima`, y como `dd/mm/aaaa hh:mm`. Las pantallas de cuentas ya lo hacen, en `America/Lima`, y las de `organization` también; las demás todavía usan la zona del navegador (O70) | Media. Antes de la próxima pantalla que muestre fechas (pasos 4 y 5) |
| Colombia y México (§4.7, D-52, D-75): las longitudes de sus documentos —el RFC tiene 12 caracteres para empresas y 13 para personas con un solo tipo; el NIT, decidir si el dígito de control va dentro del número; la CC exige exactamente 10—, el cálculo del dígito de control del NIT, el RFC y la CURP, y la lista oficial de divisiones del país (DANE, INEGI). **(propuesta)** Partir el RFC en dos tipos, dejar la CC sin longitud fija y ajustar el NIT. **Riesgo aceptado:** hasta entonces nada en el código impide registrar una empresa de esos países, y sus tipos con dígito de control aceptan cualquier número sin verificar. No hay que crear a mano divisiones de Colombia ni de México: con una sola cargada, toda dirección de ese país pasa a exigir la suya (D-53) | Antes de registrar la primera empresa de Colombia o México |
| Dar de alta Santa Rosa de Loreto (Ley 32403, 03/07/2025) con su propia migración; hasta entonces no se puede elegir (D-77) | Cuando el INEI o SUNAT publiquen su código |




## Cuándo un paso está terminado

- Las migraciones de Flyway crean sus tablas, y una migración ya aplicada nunca se edita.
- El backend sigue la estructura de contextos, con sus pruebas, y todo endpoint de negocio exige un rol o un permiso.
- El frontend tiene sus pantallas y las opciones del menú del paso ya no llevan `pending: true`.
- Pasan las pruebas del backend, las unitarias del frontend y las de extremo a extremo, con las capturas de referencia revisadas.
- Si el paso tiene pantallas, se cierra con una corrida del spec en vivo contra el backend real, sin la API simulada, y su resultado queda anotado en [retroalimentacion/](retroalimentacion/README.md).
- La lista oficial de divisiones de Perú se revisa contra las leyes nuevas y las ediciones nuevas del INEI, y cada distrito nuevo con código oficial entra con su propia migración (D-77).
- Si la implementación se apartó del diccionario a propósito, el diccionario lo registra.
- El documento de su contexto en [contextos/](contextos/README.md) tiene sus reglas al día, y su sección 6 queda sin pendientes de código o con cada pendiente asignado a un paso posterior.
- Este documento actualiza el estado del paso.

