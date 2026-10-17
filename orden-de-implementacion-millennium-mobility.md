# Orden de implementación — ERP Millennium Mobility

**Actualizado:** 16/10/2026 (paso 3 construido, salvo los permisos, el registro público con perfil y el dígito de control) · 15/10/2026 · **Fuente:** diccionario de datos v1.17, §3. Si este documento y el diccionario no coinciden, manda el diccionario.

El orden lo fijan las dependencias entre tablas: nada se construye antes de lo que referencia. Cada paso entrega su contexto completo, del backend (migraciones, dominio y API) al frontend (pantallas y opciones del menú).

## Resumen


| Paso | Bloque del diccionario                                | Contexto              | Depende de                       | Estado                   |
| ---- | ----------------------------------------------------- | --------------------- | -------------------------------- | ------------------------ |
| 0    | 2 · Seguridad, en parte, y el núcleo común            | `shared`, `iam`       | —                                | Construido               |
| 1    | 1 · Geografía                                         | `location`            | —                                | Construido (Perú)        |
| 2    | 3 · Empresas                                          | `organization`        | 1 y 2                            | En curso                 |
| 3    | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | 2 y 3                            | Construido, salvo permisos, registro con perfil y dígito de control |
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
- **Lo que falta del bloque:** el paso 3 completó las asignaciones con su alcance, los bloqueos, la caducidad y dar acceso desde el colaborador, que reemplazó a "Nuevo usuario". Quedan los permisos y el registro público con perfil (paso 3, abajo).



### Paso 1 · Geografía y localización — construido para Perú

- **Estado (15/10/2026):** construido para Perú, el único país que opera (D-5). Backend y frontend tienen monedas, países con sus escalones, divisiones, direcciones y tipos de documento tributario, con su API, sus pruebas y la ficha del país; la dirección elige su división en cascada. V24 carga el ubigeo del INEI, edición 2022, con 1 891 distritos. El diccionario v1.16 registra sus reglas y el alcance del cierre (D-52); el detalle está en la [revisión de cierre](retroalimentacion/2026-10-15-location.md). Las longitudes de los documentos de Colombia y México y sus listas oficiales quedan en los pendientes transversales, para antes de registrar su primera empresa.
- **Tablas:** `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses`, `tax_id_types` (D-44).
- **Pantallas:** Catálogos › Monedas y Países. Cada país abre su ficha, con sus datos arriba y, debajo, en pestañas, su estructura territorial (escalones y divisiones, D-45) y sus documentos tributarios; la misma ficha registra el país y corrige sus datos. Las direcciones se llenan dentro de cada formulario que las usa; no tienen opción propia.
- **Datos:** el archivo oficial de divisiones territoriales (ubigeo del INEI), cargado completo (D-2) por V24 desde `scripts/ubigeo/` del backend.



### Paso 2 · Empresas — en curso

- **Estado (16/10/2026):** Empresas, Marcas comerciales, Sedes y Ubicaciones están construidas en el backend y en el frontend, con su opción del menú. La empresa, la sede y la ubicación llevan su dirección anidada. Falta `roles.company_id`. Los hallazgos de seguridad de la revisión del 14/10/2026 quedaron resueltos con el paso 3; lo que falta para cerrar el paso está en [contextos/organization.md](contextos/organization.md), sección 6.
- **Tablas:** `companies`, `commercial_brands`, `branches`, `locations`. Además se agrega `roles.company_id`, que esperaba a que existieran las empresas. `tax_id_types` pasó al paso 1 (D-44).
- **Pantallas:** Configuración › Empresas (solo superadministrador), Sedes, Marcas comerciales y Ubicaciones. Qué países operan lo muestra Empresas filtrada por país (D-3).
- **Datos:** razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales.



### Paso 3 · Colaboradores y accesos, y el resto del bloque 2 — construido, salvo los permisos, el registro público con perfil y el dígito de control

- **Estado (16/10/2026):** construido en `feature/access-model`, en el backend y en el frontend, con las seis rondas del modelo de acceso. Lo construido:
  - colaboradores, con su salida y su regreso, que nunca tienen fecha posterior a hoy en la zona del colaborador;
  - dar acceso desde el colaborador (`POST /access-grants`), que crea la cuenta o vincula la que la persona ya tiene, sin tocar sus roles ni su contraseña;
  - asignaciones por empresa, sede y marca, con su caducidad y un proceso que cierra las vencidas;
  - bloqueos de cuenta en dos líneas de tiempo, la laboral y la de seguridad, y el restablecimiento de contraseña;
  - la salida que retira los roles de esa empresa y cierra la cuenta cuando era el último colaborador activo, siempre, con el último superadministrador como único freno;
  - el administrador de accesos (`ADMIN_ACCESOS`) y la separación de funciones: el administrador de empresa registra a su gente y no maneja cuentas, el de accesos maneja cuentas y no registra colaboradores, y una misma cuenta no tiene los dos roles;
  - la frontera entre empresas en la aplicación (`404` fuera del alcance) y en la base (Row Level Security, con la aplicación conectada con una cuenta que no es dueña de nada);
  - `iam` depende de `organization` y nunca al revés (mapa de contextos, decisiones 2 y 4).
  
  Migraciones V26 a V32. El detalle está en [contextos/iam.md](contextos/iam.md), [contextos/organization.md](contextos/organization.md) y la [revisión del 16/10/2026](retroalimentacion/2026-10-16-iam.md). Las decisiones que se tomaron en el camino están en la propuesta del diccionario v1.18 (D-55 a D-72), pendiente de validar.
- **Verificación** (sobre el backend `6a7d9b1` y el frontend `1a341ab`): 926 pruebas del backend, sin fallos, errores ni omitidas, conectadas como `erp_mm_app` sobre una `erp_mm_db_verify` recreada y migrada por `erp_mm_owner` de V1 a V32; 877 pruebas unitarias del frontend en 88 archivos; build inicial de 489,11 kB, bajo el presupuesto de 500 kB; `lint:styles` sin observaciones; 425 e2e en verde y 22 omitidas a propósito, sin fallos. La primera sonda contra el backend real (ronda 4) coincidió en 210 de 211 comprobaciones HTTP —la restante fue el orden de la propia sonda, no el backend— y en las 40 de la frontera entre empresas, y el recorrido con los tres roles no dio errores de página. Sonda final (rondas 5 y 6): 129 de 129 comprobaciones de la política coincidieron, y el navegador sin simulaciones se comportó como dice el contrato; quedan cuatro diferencias menores, anotadas en la [revisión del 16/10/2026](retroalimentacion/2026-10-16-iam.md).
- **Lo que falta del paso:**
  - `permissions` y `role_permissions`: esperan la matriz de permisos, que llega antes del paso 5 (ajuste 3). Hasta entonces, los roles globales —el superadministrador y el administrador de accesos— leen los datos de todas las empresas.
  - El registro público con perfil (D-32): espera a que convenciones §14.16 defina cómo lista sedes y tipos de documento sin sesión. Mientras tanto sigue apagado.
  - Verificar el dígito de control de los documentos tributarios (§4.7; [location.md](contextos/location.md), LOC-R35, y [organization.md](contextos/organization.md), ORG-R04): decidir qué contexto lo calcula. Hoy `has_check_digit` se guarda y nadie lo verifica (pendientes transversales).
- **Tablas:** `employees`; `user_roles` con su alcance por empresa, sede y marca; `user_access_suspensions`, `permissions` y `role_permissions`.
- **Pantallas:** Configuración › Colaboradores, con las acciones de cuenta según el rol de la sesión; Seguridad › Usuarios, para el superadministrador y el administrador de accesos; Seguridad › Roles y permisos: sus pantallas de roles propios y permisos existen en el frontend, pero no figuran en el menú hasta que el backend sirva la matriz.
- **Datos:** colaboradores reales y la matriz de permisos por rol.
- **Nota:** `advisor_assignments` figura en el bloque 3b, pero se construye con ventas (bloque 12), porque apunta a los contratos.



### Paso 4 · Catálogos de flota — en curso

- **Estado (16/10/2026):** el frontend de Tipos de vehículo, Marcas y modelos, Colores y Equipamiento está en `feature/location`, con su opción del menú; su backend está en la rama `feature/vehiclecatalog`, que se pone al día con `feature/location` antes de integrarse. Hasta entonces esas pantallas no tienen servidor que les responda.
- **Tablas:** `vehicle_types`, `vehicle_brands`, `vehicle_models`, `colors`, `equipment_types` y `equipment_type_vehicle_types`.
- **Pantallas:** Catálogos › Tipos de vehículo, Marcas y modelos (una sola pantalla: marcas a la izquierda, los modelos de la marca elegida a la derecha), Colores y Equipamiento.
- **Datos:** los modelos reales, conciliados entre los dos catálogos entregados, que no coinciden entre sí (§7.3).



### Paso 5 · Vehículo y estados

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
3. **El bloque 2 se termina con la matriz de permisos, antes del paso 5.** El paso 3 se construyó sin `permissions` ni `role_permissions`, porque falta el dato: qué permisos lleva cada rol. Los permisos hacen falta antes de las operaciones de negocio de los pasos 5 y 7, como dar de alta una unidad o registrar una factura, y con ellos se acota también lo que lee un rol global: hasta entonces, el superadministrador y el administrador de accesos leen los datos de todas las empresas. Con la matriz se decide además quién crea los roles de una empresa.
4. **Desde el vehículo hay dos ramas independientes**, que se pueden alternar.






## Pendientes transversales


| Qué | Prioridad |
| --- | --- |
| La matriz de permisos por rol: `permissions` y `role_permissions` (§5.6), qué lee un rol global y quién crea los roles de una empresa. Hasta entonces, el superadministrador y el administrador de accesos leen los datos de todas las empresas, y la opción Seguridad › Roles y permisos queda fuera del menú: sus pantallas existen en el frontend y llaman a rutas que el backend todavía no sirve | Antes del paso 5 (ajuste 3) |
| Historia de los reingresos: hoy el regreso de un colaborador reemplaza su fecha de ingreso y borra la de salida, y el período anterior se pierde. Necesita un diseño en el diccionario (§5.3; propuesta v1.18, §5.13) | Cuando el diccionario lo diseñe, antes del primer reingreso real |
| Restablecer la contraseña con un enlace de un solo uso al correo de la persona, en vez de mostrarle la provisional al administrador | Cuando exista el envío de correos |
| Registro público con perfil (D-32): elegir empresa y sede sin sesión y crear el colaborador y la cuenta juntos; hoy crea solo la cuenta y viene apagado | Cuando se decida [convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §14.16, antes de encender el registro |
| La sesión guarda el hash de la contraseña | Media |
| Un correo de más de 255 caracteres devuelve error 500 en vez de 400 | Baja |
| Separar en cada entorno la cuenta de Flyway, dueña de las tablas, de la cuenta de la aplicación, que lee, inserta y actualiza, y solo borra en `spring_session*`. El código y las pruebas ya funcionan así: las pruebas corren conectadas como `erp_mm_app` (README del backend, "Cuentas de la base" y "Pruebas"). Falta correr los scripts (`scripts/database/create-roles.sql`, una vez por clúster, y `hand-over-database.sql`, una vez por base ya migrada) y fijar `FLYWAY_USER`, `FLYWAY_PASSWORD`, `DATABASE_USER` y `DATABASE_PASSWORD`. Con `postgres`, que es superusuario, la base no aísla las empresas (V29) | En cada entorno, antes de usar roles de empresa o de sede en él |
| Zona de los momentos en las exportaciones con alcance de empresa o de sede: cada controlador le pasa a `SpreadsheetExporter` la zona de ese alcance ([convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §8.1). Hoy ninguna exportación de empresa o de sede lleva momentos; la de cuentas los da en `America/Lima`, porque una cuenta no es de una empresa | Con la primera exportación de empresa o de sede que lleve momentos |
| Fijar los decimales de una moneda cuando un monto ya la usa (§4.1): `location` publica un puerto (SPI) que implementan los contextos con montos, para saber si alguno usa la moneda sin depender de ellos | Antes del paso 7 |
| Fechas en pantalla (§2.6): deben verse en la zona de la sede; sin sede, en la del país de la empresa; sin empresa, en `America/Lima`, y como `dd/mm/aaaa hh:mm`. Las pantallas de cuentas ya lo hacen, en `America/Lima`; las demás todavía usan la zona del navegador | Media. Antes de la próxima pantalla que muestre fechas |
| Verificar el dígito de control de los documentos tributarios (§4.7): decidir qué contexto lo calcula. Hoy `has_check_digit` se guarda y nadie lo verifica | Paso 3, con los colaboradores |
| Longitudes de los documentos de Colombia y México (§4.7): el RFC tiene 12 caracteres para empresas y 13 para personas con un solo tipo; el NIT y el DNI, decidir si el dígito de control va dentro del número; la CC exige exactamente 10. **(propuesta)** Partir el RFC en dos tipos, dejar la CC sin longitud fija y ajustar el NIT. Además, cargar la lista oficial de divisiones del país (DANE, INEGI) | Antes de registrar la primera empresa de Colombia o México |




## Cuándo un paso está terminado

- Las migraciones de Flyway crean sus tablas, y una migración ya aplicada nunca se edita.
- El backend sigue la estructura de contextos, con sus pruebas, y todo endpoint de negocio exige un rol o un permiso.
- El frontend tiene sus pantallas y las opciones del menú del paso ya no llevan `pending: true`.
- Pasan las pruebas del backend, las unitarias del frontend y las de extremo a extremo, con las capturas de referencia revisadas.
- Si la implementación se apartó del diccionario a propósito, el diccionario lo registra.
- El documento de su contexto en [contextos/](contextos/README.md) tiene sus reglas al día, y su sección 6 queda sin pendientes de código o con cada pendiente asignado a un paso posterior.
- Este documento actualiza el estado del paso.

