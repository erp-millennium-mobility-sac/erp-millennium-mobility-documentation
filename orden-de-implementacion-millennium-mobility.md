# Orden de implementación — ERP Millennium Mobility

**Actualizado:** 06/10/2026 · **Fuente:** diccionario de datos v1.15, §3. Si este documento y el diccionario no coinciden, manda el diccionario.

El orden lo fijan las dependencias entre tablas: nada se construye antes de lo que referencia. Cada paso entrega su contexto completo, del backend (migraciones, dominio y API) al frontend (pantallas y opciones del menú).

## Resumen


| Paso | Bloque del diccionario                                | Contexto              | Depende de                       | Estado                   |
| ---- | ----------------------------------------------------- | --------------------- | -------------------------------- | ------------------------ |
| 0    | 2 · Seguridad, en parte, y el núcleo común            | `shared`, `iam`       | —                                | Construido               |
| 1    | 1 · Geografía                                         | `location`            | —                                | En curso                 |
| 2    | 3 · Empresas                                          | `organization`        | 1 y 2                            | En curso                 |
| 3    | 3b · Colaboradores y accesos, y el resto del bloque 2 | `organization`, `iam` | 2 y 3                            | Pendiente                |
| 4    | 4 · Catálogos de flota                                | `vehiclecatalog`      | 3                                | Pendiente                |
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
- **Lo que falta del bloque:** se completa en el paso 3. El detalle está en §5.13 del diccionario.



### Paso 1 · Geografía y localización — en curso

- **Estado (06/10/2026):** backend y frontend tienen monedas, países con sus escalones, divisiones, direcciones y tipos de documento tributario, con su API, sus pruebas y la ficha del país. Las direcciones no tienen API propia: las escribe y las lee su dueño, por la fachada de `location` ([convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §3.3). Solo falta la carga del ubigeo.
- **Tablas:** `currencies`, `countries`, `geo_division_levels`, `geo_divisions`, `addresses`, `tax_id_types` (D-44).
- **Pantallas:** Catálogos › Monedas y Países. Cada país abre su ficha, con sus datos, su estructura territorial (escalones y divisiones, D-45) y sus documentos tributarios, una sección bajo otra; la misma ficha registra el país y corrige sus datos. Las direcciones se llenan dentro de cada formulario que las usa; no tienen opción propia.
- **Datos:** el archivo oficial de divisiones territoriales (ubigeo del INEI). Se carga completo (D-2).



### Paso 2 · Empresas — en curso

- **Estado (06/10/2026):** Empresas está construida en el backend y en el frontend, con su dirección fiscal anidada: se registra con la empresa y se corrige o se muda desde su subrecurso. El frontend tiene además las vistas de Sedes y Marcas comerciales, sin ruta y con su opción del menú pendiente hasta que exista su backend. En el backend faltan `commercial_brands`, `branches`, `locations` y `roles.company_id`; Ubicaciones falta en los dos. Las sedes y las ubicaciones nacen con la dirección anidada, igual que la empresa.
- **Tablas:** `companies`, `commercial_brands`, `branches`, `locations`. Además se agrega `roles.company_id`, que esperaba a que existieran las empresas. `tax_id_types` pasó al paso 1 (D-44).
- **Pantallas:** Configuración › Empresas (solo superadministrador), Sedes, Marcas comerciales y Ubicaciones. Qué países operan lo muestra Empresas filtrada por país (D-3).
- **Datos:** razón social y RUC de la empresa, sedes con su código de establecimiento, marcas comerciales.



### Paso 3 · Colaboradores y accesos, y el resto del bloque 2

- **Tablas:** `employees`; `user_roles` con su alcance por empresa, sede y marca; `user_access_suspensions`, `permissions` y `role_permissions`.
- **Pantallas:** Configuración › Colaboradores; Seguridad › Roles y permisos; Seguridad › Usuarios también para el administrador de empresa.
- **Operaciones nuevas:** dar acceso desde el colaborador (reemplaza a "Nuevo usuario"), registro público con perfil completo, bloqueos de cuenta y la salida del colaborador con sus efectos (D-38).
- **Datos:** colaboradores reales y la matriz de permisos por rol.
- **Nota:** `advisor_assignments` figura en el bloque 3b, pero se construye con ventas (bloque 12), porque apunta a los contratos.



### Paso 4 · Catálogos de flota

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
3. **El bloque 2 se termina en el paso 3.** Los permisos hacen falta antes de las operaciones de negocio de los pasos 5 y 7, como dar de alta una unidad o registrar una factura.
4. **Desde el vehículo hay dos ramas independientes**, que se pueden alternar.



## Antes del paso 1

Corregir cómo muestra las fechas el frontend, para que todas las pantallas nuevas nazcan siguiendo la regla (§2.6):

- **Zona:** hoy se muestran en la del navegador. Deben mostrarse en la zona de la sede; sin sede, en la del país de la empresa; sin empresa, en `America/Lima`.
- **Formato:** hoy aparecen como `27/09/26, 9:05 a. m.`. Deben verse como `dd/mm/aaaa hh:mm`.



## Pendientes transversales


| Qué | Prioridad |
| --- | --- |
| La sesión guarda el hash de la contraseña | Media |
| Un correo de más de 255 caracteres devuelve error 500 en vez de 400 | Baja |
| Cerrar como "caducidad" las asignaciones vencidas | Con la pantalla de roles y usuarios |
| Separar la cuenta de Flyway, dueña de las tablas, de la cuenta de la aplicación, que lee, inserta y actualiza, y solo borra en `spring_session*`. El backend ya trae los scripts (`scripts/database/create-roles.sql`, una vez por clúster, y `hand-over-database.sql`, una vez por base ya migrada) y la migración V17 de permisos, que no otorga nada mientras no exista la cuenta `erp_mm_app`; la aplicación arranca como antes. Falta correr los scripts en cada entorno y fijar `FLYWAY_USER`, `FLYWAY_PASSWORD`, `DATABASE_USER` y `DATABASE_PASSWORD` (README del backend, "Cuentas de la base"). En desarrollo se deja de usar `postgres` | Antes de la primera migración con `company_id` (paso 2) |
| Aislamiento entre empresas en la base: políticas RLS sobre `company_id`, un punto central que fija con `set_config` las empresas de la cuenta en cada transacción (`SYSTEM` para el superadministrador, el arranque y los procesos automáticos) y pruebas de integración contra PostgreSQL 18 (Testcontainers), conectadas con la cuenta de la aplicación. Hasta entonces solo se otorgan roles de alcance sistema: el bloqueo de `UserCommandServiceImpl` se quita en este mismo cambio, nunca antes | Primera rebanada del paso 3 |
| Filtro de alcance por asignaciones en el listado, la exportación y la consulta por id de `/companies` y sus rutas anidadas; `/users` abierto al administrador de empresa; una prueba de que una empresa ajena responde `404` ([convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §9.2) | En el mismo cambio que habilite los roles de empresa (paso 3) |
| Zona de los momentos en las exportaciones con alcance de empresa o de sede: cada controlador le pasa a `SpreadsheetExporter` la zona de ese alcance ([convenciones de la API](convenciones-de-la-api-millennium-mobility.md) §8.1) | Paso 3 |
| Clases base de las cinco formas de columnas de registro: faltan la raíz `CreatedAbstractPersistenceEntity` y `CreatedByAbstractPersistenceEntity` (D-42; [patrones de persistencia](patrones-de-persistencia-millennium-mobility.md)) | Paso 3, con `user_access_suspensions`, su primera tabla |
| Fijar los decimales de una moneda cuando un monto ya la usa (§4.1): `location` publica un puerto (SPI) que implementan los contextos con montos, para saber si alguno usa la moneda sin depender de ellos | Antes del paso 7 |
| Listados de usuarios: quitar el N+1 de los roles de cada cuenta (colección `EAGER` en `UserPersistenceEntity`), con una lectura en lote o con `default_batch_fetch_size` | Baja |




## Cuándo un paso está terminado

- Las migraciones de Flyway crean sus tablas, y una migración ya aplicada nunca se edita.
- El backend sigue la estructura de contextos, con sus pruebas, y todo endpoint de negocio exige un rol o un permiso.
- El frontend tiene sus pantallas y las opciones del menú del paso ya no llevan `pending: true`.
- Pasan las pruebas del backend, las unitarias del frontend y las de extremo a extremo, con las capturas de referencia revisadas.
- Si la implementación se apartó del diccionario a propósito, el diccionario lo registra.
- Este documento actualiza el estado del paso.

