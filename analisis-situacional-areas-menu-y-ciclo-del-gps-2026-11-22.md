# Análisis situacional — áreas, menú y ciclo de compra del GPS — ERP Millennium Mobility

**Estado:** versión 3, 22/11/2026 (el primer tramo, §7). Es la base del [plan de implementación](plan-de-areas-menu-y-ciclo-del-gps-2026-11-22.md) y no decide nada por sí solo. Recoge la [auditoría del mismo día](auditoria-del-plan-de-areas-menu-y-ciclo-del-gps-2026-11-22.md) y las decisiones que el responsable aprobó ese día (Q1 a Q27, en el plan, §2).

**Fuente:** el código de `develop` del backend (`ed7ccda7`) y del frontend (`ee7736af`) y la documentación de `master` (`e71fdb4`), los tres sin cambios pendientes; el [diccionario de datos](diccionario-de-dtos-millennium-mobility.md) 1.79; las reglas de arquitectura de las skills `backend-module` y `frontend-feature`; el [plan de migración a los contextos objetivo](plan-de-migracion-a-los-contextos-objetivo-2026-11-18.md), por su forma de trabajo; cuatro revisiones de solo lectura hechas el 22/11/2026 —el núcleo de las áreas (`shared`, `iam`, `organization`, `documents`), las áreas en compras, bitácora y reportes, el ciclo del GPS y el frontend—, con sus puntos críticos verificados a mano; y la auditoría de la versión 1, con cinco revisiones más y el correo al cliente del 16/11/2026. Las líneas citadas (L) son las de esos commits: se comprueban otra vez al abrir cada fase, porque el código sigue cambiando.

**Objetivo.** Decir, antes del plan, qué existe, qué se reutiliza, qué cambia y desde dónde se construye cada uno de los tres frentes que el responsable pidió el 22/11/2026.

**Cómo leerlo.** Las rutas van abreviadas:

| Prefijo | Carpeta |
|---|---|
| `B/` | `erp-millennium-mobility-backend/src/main/java/ky/millennium/mobility/erp/` |
| `M/` | `erp-millennium-mobility-backend/src/main/resources/db/migration/` |
| `T/` | `erp-millennium-mobility-backend/src/test/java/ky/millennium/mobility/erp/` |
| `F/` | `erp-millennium-mobility-frontend/src/app/` |
| `E/` | `erp-millennium-mobility-frontend/e2e/` |

Una referencia **§n** o **D-n** sin más señala una sección o una decisión del diccionario; las reglas de arquitectura se citan como «regla n.n».

---

## 1. Resumen

### 1.1 Qué se pidió

| Frente | Qué es |
|---|---|
| **A · Áreas** | Una dimensión nueva del aislamiento de datos, configurable por empresa (Monitoreo, Habilitación…), que funciona como la marca comercial de la asignación de rol y separa todo compras: plantillas, órdenes, facturas, notas de crédito, pagos y lo que se debe. Un documento sin área es general y lo leen todas las áreas. Una plantilla general sirve a todas las áreas, y la orden es del área de quien la emite. Una persona puede tener una, varias o todas las áreas. Lo que ya existe también recibe su área («Asignar área»). La defiende el código, no la base |
| **B · Menú** | Reordenar el menú lateral por procesos —Flota, Monitoreo, Compras, Finanzas, Herramientas, Administración, Catálogos, Seguridad—, sin las etiquetas ADMIN y SUPER, con Catálogos visible solo para quien escribe |
| **C · Ciclo de compra del GPS** | El equipo nace en la factura como el vehículo: orden de equipos, factura de equipos que crea cada equipo «Por ingresar», ingreso por escaneo del QR (solo el IMEI) por modelo, la API del fabricante completa el resto, la SIM como registro propio con su historia, faltantes y fallados con nota de crédito o reemplazo, la instalación tomada del inventario (D-263, D-264) y el historial de cada equipo —sus estados y en qué vehículo estuvo—, que el correo al cliente del 16/11 promete |

### 1.2 Veredicto

Los tres frentes caben sobre lo construido, sin un contexto nuevo y sin tocar la frontera entre empresas que defiende la base. Cada pieza tiene un molde que ya funciona:

| Lo que se necesita | Se construye sobre | Por qué sirve |
|---|---|---|
| El área como dimensión del alcance | La marca de la asignación: `user_roles.commercial_brand_id` (V27), `PermissionGrant` y los métodos de marca de `AccessScope` (D-102) | Es exactamente la misma forma: opcional, vacía significa todas, acotada a la empresa de la asignación y defendida en el código |
| El catálogo de áreas | `CommercialBrand` de `organization` de punta a punta, con V108 como plantilla moderna de tabla y V109 como plantilla de semilla de permisos | Mismo dueño, mismo ciclo (crear, renombrar, desactivar, nunca borrar), mismas pruebas |
| Compras por área | Las políticas de compras por marca y la regla de D-134 para lo que es de todas las marcas | Cada política ya pasa la marca; el área entra al lado |
| El equipo que nace en la factura | La unidad que nace en la factura (D-153): `PurchaseInvoiceUnits` → `FleetContextFacade.registerOrLinkPurchasedUnits` → `PurchasedUnitRegistrations` | Mismo flujo: la factura le pide al dueño que cree lo que compra, dentro de su transacción |
| La nota que anula un equipo | La nota «anulación de unidad» (D-191, D-193) | Mismo mecanismo: la línea llega a cero y el dueño anula lo que nombra |
| El costo de compra del equipo | El costo de la unidad que sirve y publica `procurement` (D-280) | Misma ruta de lectura: el dueño del costo lee lo que la sesión ve en el otro contexto |
| La API del fabricante | El envío de correos de `notifications`: pedido guardado en la transacción del hecho, tarea programada en una sola instancia, llamada fuera de toda transacción, reintentos, intentos anotados y vigilante | Resuelve exactamente lo que pide la regla 6.4: ninguna llamada externa dentro de una transacción |
| La SIM con su historia | La instalación de `fleet`, con su período `[inicio, fin)` y su `EXCLUDE` (V100) | Una asignación con período que no se cruza, en los dos sentidos |
| La instalación unida al inventario | D-263, D-264 y la fase 2 §2.6 del plan de migración | Ya está decidida y escrita; falta construirla |
| El historial del equipo | `vehicle_status_history` (§8.4), de solo inserción, y la ruta que un contexto sirve sobre el recurso de otro (`GET /vehicles/{id}/purchase-cost`, de `procurement`) | La historia de estados es una fila por hecho; las instalaciones de un equipo las sirve `fleet`, que es su dueño |
| El menú | `app-navigation.ts`, con acceso por permiso (D-117) | Cambia la estructura, no el mecanismo |

### 1.3 Lo que encontró el análisis y cambia el diseño

| #   | Hallazgo                                                                                                                                                                                                                                      | Evidencia                                                                                                                                                                                                                                       | Consecuencia                                                                                                                                       |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Una sola asignación tiene que cumplir todo a la vez.** Las lecturas de plantillas, órdenes y bitácora suman las marcas de todas las asignaciones de una empresa; un conjunto de áreas sumado igual al lado mezclaría asignaciones distintas | `PurchaseOrderAccessPolicy.reach` (`B/procurement/application/internal/policies/PurchaseOrderAccessPolicy.java` L28-41), `ChangeLogAccessPolicy.reach` (`B/audit/…/ChangeLogAccessPolicy.java` L35-48), verificado                              | El filtro de una lista pasa a ser la lista de asignaciones —empresa, marca y área de cada una—, no conjuntos separados; lo mismo en las escrituras (§3.4, A-H1, A-H17) |
| 2   | **«Sin área» no significa lo mismo al leer que al escribir**, y hoy todas las plantillas son generales                                                                                                                                        | Decisión del 22/11; ninguna fila tiene área                                                                                                                                                                                                     | Leer lo general lo hace todo el mundo; escribirlo, solo quien tiene acceso a todas las áreas. Una plantilla general la usa cualquier área, y la orden toma el área de quien la emite (A-H2) |
| 3   | **Un documento general puede mostrar datos de un área** si una nota o un pago de esa área lo incluye                                                                                                                                          | `PurchaseInvoiceView.creditNotes`, `InvoicePaymentPositionView.applications`, `UnitPurchaseCostView`, `CreditNoteView.balance.uses`                                                                                                             | Una nota o un pago incluye documentos de una sola área, y lo general aparte (A-H3; decidido, Q7)                                                   |
| 4   | **Una línea de factura sin vehículo es un concepto**, para todo compras                                                                                                                                                                       | `PurchaseInvoiceItem.isConcept()` es `vehicleId == null` (L84-86), verificado; 54 archivos de compras nombran `vehicle_id`; 13 usos de `isConcept()` en 9 archivos                                                                               | La línea necesita un tipo explícito —vehículo, equipo, concepto— antes de cualquier otra cosa (C-H1, C-H21)                                        |
| 5   | **Lo fiscal de una línea de factura nunca se edita**, tampoco el registro al que apunta                                                                                                                                                       | Disparador `keep_purchase_invoice_item_fiscal_data` (V103 L209-221): compara una lista cerrada de columnas, entre ellas `vehicle_id`, verificado                                                                                                                                                                 | El reemplazo de un GPS fallado no mueve la línea: el equipo nuevo apunta al que reemplaza, y el costo y la factura se siguen por esa cadena (C-H2) |
| 6   | **Las imputaciones de un pago son por unidad**                                                                                                                                                                                                | `payment_allocations.vehicle_id NOT NULL` y `UNIQUE (payment_application_id, vehicle_id)` (V107), verificado; `JdbcPayableFactsReadRepository` filtra `vehicle_id IS NOT NULL` y lee un nulo como 0                                             | Una factura de equipos se paga a nivel de factura, sin imputación por equipo (decidido, Q12); nada exige imputaciones salvo el canal concesionario, así que no hay migración de pagos (C-H3) |
| 7   | **Mostrar Catálogos, Empresas y Ubicaciones solo a quien escribe** dejaría al administrador de empresa sin Ubicaciones y al de accesos sin Empresas                                                                                           | Matriz sembrada: los siete roles leen catálogos y estructura (`E/support/permission-catalog.ts` L267)                                                                                                                                           | Solo Catálogos por escritura (decidido, Q3); Administración sigue con la lectura. Hoy solo el superadministrador escribe catálogos, así que solo él los abre, y D-141 se enmienda (B-H1, B-H5) |
| 8   | **La vinculación de los GPS ya instalados no puede ser una migración SQL**                                                                                                                                                                    | Reglas 1.2 y 1.3: ni siquiera una migración de datos escribe ni consulta las tablas de otro contexto                                                                                                                                            | Es un proceso idempotente de `fleet` que pasa por la fachada de `equipment` (C-H13, C-H19)                                                         |
| 9   | **Una prueba del frontend falla en** `develop` **desde el 20/11, y por eso el e2e no corre**                                                                                                                                                  | `stock.anular_lectura_kilometraje` (V145) falta en `E/support/permission-catalog.ts`; rojo desde `3f47a838`; el e2e de `ci.yml` tiene `needs: build`, que corre `ng test`                                                                        | La fase 0 lo corrige y corre el e2e completo antes de tocar el menú (§2.3)                                                                        |

Los hallazgos que siguen vienen de la auditoría de la versión 1:

| # | Hallazgo | Evidencia | Consecuencia |
|---|---|---|---|
| 10 | **La sesión no puede excluir las asignaciones con área de lo que no mira el área** | `F/iam/application/iam.store.ts` L400-424; `equipment-inventory-view.ts` L94, L156, L165; `EquipmentCommandServiceImpl` L266 | Las listas de la sesión conservan esas asignaciones; solo la regla 5.5 las excluye (A-H5) |
| 11 | **El área de la plantilla y el de la orden son dos cosas.** Si la orden copiara el área de su plantilla, una plantilla general daría órdenes generales, que solo emite alguien con acceso a todas las áreas | Decisión del 22/11: un mismo formato sirve a todas las áreas | La plantilla dice quién la usa; la orden es del área de quien la emite (A-H2) |
| 12 | **Lo ya emitido no puede recibir área** con las reglas copiadas de la marca | `PurchaseOrderTemplateCommandServiceImpl` L231-255 (`brandInUse`); `dev` tiene 2 plantillas y 9 órdenes | La operación única «Asignar área» sobre órdenes generales y facturas generales sin orden (A-H14) |
| 13 | **Una lectura de compras devuelve todo** | `GET /suppliers/{id}/payables` (`SupplierPaymentQueryServiceImpl` L225, L227, L235-238) | Se filtra por área, con las demás lecturas que nombra A-H16 |
| 14 | **El historial del equipo que se prometió al cliente no existe** | Correo del 16/11, L52-55; `equipment` no escribe bitácora y `status_reason` (V134) se sobrescribe | Una tabla de historia de estados y las instalaciones de un equipo servidas por `fleet` (C-H18) |
| 15 | **La vinculación de lo instalado choca con el esquema y con la carga inicial** | `V132` L19, L22, L35; `MigratedUnitRegistrations` L68, L342-348 (D-206) | Un modelo heredado por tipo, la ubicación, la cuenta `SYSTEM`, y enlazar también desde la carga inicial y al completar el serial (C-H19) |
| 16 | **El escaneo tiene que saber el modelo** | Una factura admite varios modelos; el costo sale de la línea | Se escanea por modelo (C-H20) |
| 17 | **`isConcept()` no puede pasar a ser «tipo = concepto»** | 13 usos en 9 archivos; `PurchaseInvoice` L140 | `isVehicle()` e `isEquipment()`, y una decisión por uso (C-H21) |
| 18 | **500 líneas no se insertan por lote con la configuración de hoy** | Sin `hibernate.jdbc.batch_size`; ids `IDENTITY`; `bump_data_version` por sentencia (V94 L276-281); PRC-R65 no fija tiempo | Un INSERT de varias filas y una meta medida contra `dev` (C-H22) |
| 19 | **«Anular y copiar» duplicaría los equipos** | D-211, D-214, PRC-R66 | El alta vincula los equipos sin línea vigente (C-H23) |
| 20 | **`equipment` no puede firmar como `SYSTEM`** | `created_by NOT NULL` (V132 L35); `equipment` no depende de `iam::acl` | `equipment` suma `iam::acl` (C-H25) |
| 21 | **La cámara está bloqueada y la ficha del equipo es un diálogo** | `public/_headers` L10 (`camera=()`); `EquipmentDetailsDialog` y la regla 4.9 | `camera=(self)`; la ficha del equipo y la de la SIM pasan a páginas (C-H26, C-H27) |
| 22 | **Una orden mixta mezclaría áreas, y sin fabricante cientos de equipos se completan uno por uno** | La plantilla da área y marca a la orden; EQP-R14 pide serial para Disponible | Una orden es de un solo tipo (Q24); el ingreso completo a mano sigue como camino normal hasta que llegue la API (Q25) (C-H28, C-H30) |

### 1.4 Tamaño

| Qué | Cuánto |
|---|---|
| Migraciones | V148 a V166, diecinueve, repartidas en `procurement`, `equipment`, `iam`, `fleet`, `organization` y `audit` (§6.1) |
| Contextos del backend | `shared`, `iam`, `organization`, `documents`, `procurement`, `audit`, `reporting`, `equipment` y `fleet` |
| Contextos del frontend | `app-shell`, `shared`, `iam`, `organization`, `procurement`, `equipment`, `fleet` y `change-log` |
| Esfuerzo | De 53 a 72 días de trabajo, en ocho fases (plan, §9) |
| Bloqueos externos | La documentación y las credenciales de la API del fabricante: sin ellas se construye todo menos su adaptador (C-H14) |

---

## 2. Punto de partida

### 2.1 Los repositorios

| Repositorio | Rama | Commit | Cambios pendientes |
|---|---|---|---|
| Backend | `develop` | `ed7ccda7` | Ninguno |
| Frontend | `develop` | `ee7736af` | Ninguno |
| Documentación | `master` | `e71fdb4` | Ninguno |

La próxima migración libre es V148, la última decisión es D-282, el diccionario está en la versión 1.79 y `DESIGN.md` termina en la revisión 98. Hay dos migraciones Java: V129 (`procurement`) y V139 (`organization`), componentes de Spring que extienden `BaseJavaMigration`.

### 2.2 Lo que ya está decidido y por construir

| Qué | Dónde | Estado |
|---|---|---|
| La instalación toma un equipo Disponible del inventario y lo pasa a Instalado; el motivo del retiro decide adónde vuelve (falla → Mantenimiento, robo → De baja, reemplazo o «sin falla» → Disponible); solo `fleet` instala y retira | D-263; fase 2 §2.6 del plan de migración | FLT-R152 a FLT-R154 en ❌ (`contextos/fleet.md`) |
| Una unidad con dispositivos del inventario instalados no se anula ni pasa a Inactivo; al traspasar, el GPS se retira antes | D-264 | Por construir |
| Al registrar una nota de «anulación de unidad», la respuesta nombra las unidades retenidas | D-266 | Por construir: hoy `procurement` descarta lo que responde `fleet` |

### 2.3 Problemas que existen hoy

1. **El CI del frontend está en rojo, y el e2e no corre en `develop`.** `stock.anular_lectura_kilometraje` llegó con V145 (D-267) y está en `F/iam/application/permission-codes.ts` L103, pero no en el catálogo simulado `E/support/permission-catalog.ts`. `permission-codes.spec.ts` lo detecta (1 de 10 falla, corrido el 22/11/2026). Está en rojo desde `3f47a838` (20/11), integrado en `f614aa34`; ni la spec ni el catálogo cambiaron después. El paso «Unit tests» de `.github/workflows/ci.yml` (L40-41) corre `npx ng test --watch=false` dentro del job `build`, y el e2e tiene `needs: build`: con la unitaria en rojo, el e2e no corre. Cinco integraciones del frontend llegaron después sin e2e. El arreglo cambia además `E/roles.spec.ts` L160 y L184 («39» → «40»), `E/permission-matrix.spec.ts` L316 («22/39» → «22/40») y nueve imágenes (la matriz y la ficha del rol). La matriz simulada no se edita: SUPERADMIN y ADMIN_EMPRESA reciben el permiso solos.
2. **La matriz de permisos repite el menú a mano y ya se desfasó.** `F/iam/presentation/components/permission-matrix/permission-matrix-groups.ts` no tiene Inventario de equipamiento (sus tres permisos caen en Inventario por `module === 'stock'`, L114-116), ni Requisitos de equipos, ni Por pagar.
3. `E/screens.ts` **no tiene la pantalla del inventario de equipamiento**, así que no tiene imagen de referencia.
4. **El inventario todavía deja pasar un equipo de Instalado a Mantenimiento** (`B/equipment/domain/model/aggregates/Equipment.java` `moveToMaintenance` L193), un camino que D-263 retira.
5. **D-266 sin construir:** la nota de «anulación de unidad» descarta las unidades retenidas.
6. **El equipo no guarda historia.** `equipment` no escribe la bitácora, y `fleet_equipments.status_reason` (V134 L8) se sobrescribe en cada cambio de estado, así que el motivo anterior se pierde. El correo al cliente del 16/11 promete «el historial de en qué vehículo estuvo cada equipo y cuándo se retiró o reemplazó».
7. **La cámara está bloqueada en el sitio desplegado.** `public/_headers` L10 envía `Permissions-Policy: camera=()` a todo `/*`.

### 2.4 Reglas que gobiernan todo el trabajo

| Regla | Qué exige aquí |
|---|---|
| 1.1 a 1.4 | Cada contexto usa a otro solo por su fachada `::acl`, sin agregados; `ModularityTests` y `ForeignAclStaysInTranslationLayerTests` lo vigilan |
| 1.2 y 1.3 | Ningún código, tampoco una migración de datos, consulta o escribe las tablas de otro contexto. Las llaves foráneas entre contextos sí |
| 3.3 | Un duplicado termina en `_CONFLICT` (409); cualquier otro código propio responde 422. Cada código nuevo lleva su clave `error.<kebab>.message` en los dos bundles (`OwnErrorCodesHaveMessagesTests`) |
| 4.1 | No se guarda lo que se calcula: el faltante de una orden, el valor de un equipo y su estado de pago se calculan |
| 4.2 y 4.3 | Nada se borra; un hecho nuevo es una fila nueva: la SIM de un GPS, el reemplazo de un equipo |
| 4.8 y 4.10 | Cada agregado nuevo tiene su `<Agregado>AuditTests`; la clase base sale de las columnas de su tabla |
| 5.1, 5.7 | Toda tabla con `company_id` lleva RLS con las políticas de V29 y su prueba; `RowLevelSecurityMigrationTests` exige la lista exacta |
| 5.4 | El campo que no se puede ver no viaja: el costo de un equipo, las columnas de compras de otra área |
| 5.8 | Toda ruta pide un permiso del catálogo, con su constante en `PermissionCodes` (`PermissionCatalogConsistencyTests`) |
| 6.2 | Toda exportación es un documento generado, con su alcance escrito por `DocumentScopes` |
| 6.3 | Todo `@Scheduled` empieza con `SingleInstanceRun.runAlone` |
| 6.4 | Un servicio de comandos se repite entero ante un interbloqueo (`DeadlockRetry`): ninguna llamada externa dentro de su transacción |

---

## 3. Frente A — Áreas

### 3.1 Cómo funciona hoy la marca, que es el molde

| Pieza | Dónde | Qué hace |
|---|---|---|
| La columna | `user_roles.commercial_brand_id` (V27): llave compuesta `(commercial_brand_id, company_id)`, y el índice único `user_roles_active_scope_uq` sobre `(user_id, role_id, company_id, branch_id, commercial_brand_id) NULLS NOT DISTINCT WHERE revoked_at IS NULL` | Una asignación puede limitarse a una marca de su empresa; vacía son todas, también las futuras |
| Quién no se limita | V31 y `ROLE_ASSIGNMENT_BRAND_NOT_ALLOWED`; `Role.isLimitableToBrand()`; `limitableToBrand` en la API | Un rol global o reservado no se limita a una marca |
| El alcance de la asignación | `B/iam/domain/model/valueobjects/RoleScope.java` (empresa, sede, marca) | El dónde de cada asignación |
| El permiso con su dónde | `B/shared/application/security/PermissionGrant.java` (código, empresa, sede, marca) | Lo que lee cada contexto |
| Las preguntas | `AccessScope`: `actsOnBrand`, `actsOnWholeCompanyBrand`, `actsOnBranchBrand`, `actsOnWholeCompanyEveryBrand`, `brandReach` (L215-289) | Una lectura o una escritura de algo que tiene marca |
| Dónde aplica | Solo donde el código llama a un método con marca (D-102): `actsOn`, `actsOnWholeCompany`, `actsWhere` y `companyReach` la ignoran | La marca acota los vehículos, la bitácora de lo que tiene marca, las órdenes por la marca de su plantilla y las escrituras de facturas y notas por la marca de cada línea |
| La sesión | `SessionCompanyResource`: `permissions`, `wholeCompanyPermissions` y `brandPermissions` | El frontend repite las preguntas para ofrecer acciones (`F/iam/application/iam.store.ts` L400-425) |
| La exportación | `DocumentScopes.where` (`B/documents/interfaces/acl/DocumentScopes.java` L65-68): `empresa/sede/marca` de cada asignación | La llave de reutilización de un documento generado |

**Consecuencia que conviene tener presente.** Un espejo exacto de la marca no limita nada mientras ningún registro tenga área. Por eso el área aísla solo lo que lleva la columna —compras y su bitácora— y nada más (A-H11).

### 3.2 Qué se reutiliza tal cual

- La frontera entre empresas: RLS, `DatabaseAccessScope`, `AccessScopeJpaTransactionManager` y las funciones de V29 no cambian. Ningún área entra a la base como política.
- `CurrentUserAccessScope` sigue calculando el alcance una vez por pedido; solo pasa el área de cada asignación a su `PermissionGrant`.
- Las pruebas de arquitectura detectan solas casi todo olvido: `ModularityTests`, `ForeignAclStaysInTranslationLayerTests`, `LayerDependencyTests`, `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`, `DeadlockRetryTests`, `JdbcDataVersionsTests`, `RowLevelSecurityMigrationTests`, `HandOverDatabaseScriptTests` y `OwnErrorCodesHaveMessagesTests`.
- No hace falta ningún par incompatible nuevo: la estructura de una empresa no tiene pares (V109 L9), y `permission_incompatibilities` sigue en 28.
- Ninguna dependencia nueva entre módulos: `iam`, `procurement`, `fleet` y `financing` ya dependen de `organization::acl`.

### 3.3 Qué cambia, capa por capa

`organization` **— el catálogo.** Todo se copia de la marca comercial:

| Pieza | Molde | Nuevo |
|---|---|---|
| Agregado | `CommercialBrand` (código `^[A-Z0-9]+$` de 2 a 20, en mayúsculas; nombre de 2 a 100). El formato vive en el dominio (`CommercialBrand` L29, L116-119); la tabla solo exige código y nombre no vacíos (V16 L20-21) | `Area` |
| Comandos y consultas | `Create/Update/ChangeCommercialBrandStatusCommand`; `GetCommercialBrandByPublicIdQuery`, `…AggregateByPublicIdQuery`, `…AggregateForShareQuery`, `…ByIdsQuery`, `SearchCommercialBrandsByCompanyQuery`, `ExportCommercialBrandsByCompanyQuery` | Sus pares de `Area` |
| Servicios | `CommercialBrandCommandServiceImpl` (empresa `FOR SHARE`, `COMPANY_INACTIVE`, duplicado también en carrera, `deniedOn`), `CommercialBrandQueryServiceImpl` (lecturas con `actsOn(VIEW_STRUCTURE)`, `FOR SHARE` con `Propagation.MANDATORY`) | `AreaCommandServiceImpl`, `AreaQueryServiceImpl` |
| Persistencia | `CommercialBrandPersistenceEntity` (`AuditableAbstractPersistenceEntity`), su ensamblador, `CommercialBrandRepositoryImpl` (`NATURAL_KEYS`, relectura tras el candado, `lockByPublicId`, `shareByPublicId`, `shareByIds`, búsqueda con `fold_search_text`) | Sus pares de `Area` |
| REST | `CompanyCommercialBrandsController` (`/api/v1/companies/{companyId}/commercial-brands`) y `CommercialBrandsController` (`/api/v1/commercial-brands/{id}`) | `CompanyAreasController` (`/api/v1/companies/{companyId}/areas`) y `AreasController` (`/api/v1/areas/{id}`) |
| Fachada | `OrganizationContextFacade.resolveScope` (L135-148), referencias en lote (L162-166) | `resolveScope` de cuatro argumentos (el de tres queda como `default` que pasa nulo: lo llaman también `fleet`, `financing` y `procurement`), referencias de áreas en lote, área por identificador público `FOR SHARE` |
| Exportación y versión | `OrganizationDocumentKinds` (L22, L42-47), `OrganizationDataVersions` (L24-27), `JdbcOrganizationDataVersions` (L32-50) | Tipo `company-areas`, `areas()`, y `areas` en `employees()` y en lo que resuelve la fachada |
| Errores | `NaturalKeyTakenException` (L27-28), `AffiliationErrors` (L20-43) | `AREA_CONFLICT`, `AREA_NOT_FOUND`, `AREA_INACTIVE`, `AREA_OTHER_COMPANY` |
| Colaborador | `branch_id`: `Employee` (L24), `BranchAffiliationPolicy` (L52-60), `EmployeeCommandServiceImpl` (L170-181, L253-265), recursos, vista y exportación | `area_id` informativo con `AreaAffiliationPolicy`; no decide dónde actúa un permiso ni la zona horaria |

`iam` **— la asignación y la sesión.**

| Pieza | Hoy | Cambio |
|---|---|---|
| Recurso y comando | `GrantRoleResource` (L31-32), `GrantRoleCommand` (L24-25) | `areaId` |
| Validación | `RoleAssignmentCommandServiceImpl`: `scopeMismatch` (L299-314), el control de marca (L143-146), `resolveScope` (L147-149) | Un rol de alcance sistema con área responde `ROLE_ASSIGNMENT_SCOPE_MISMATCH`, porque `scopeMismatch` corre antes (L139 contra L143), como hoy la marca; `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` es solo para un rol reservado; `resolveScope` con el área. El duplicado (`holds`, L243) ya compara `RoleScope` completo, así que el mismo rol en dos áreas son dos asignaciones válidas |
| Traducción | `ExternalOrganizationService.resolveScope` de `iam` (L187-190) arma `new RoleScope(c, b, brand)`; las referencias en L205-214 | Pasa el área: con el constructor de tres argumentos compilaría y perdería el área sin error. Suma `fetchAreaReferences` |
| Dominio | `RoleScope` (L20-26), `RoleAssignment` (L46-55, L121-123), `Role.isLimitableToBrand()` (L169-171) | `areaId` («un área siempre es de una empresa»), `admitsArea`, `isLimitableToArea()` con la misma regla que la marca |
| Regla 5.5 | `User.permissionCodesActingOnWholeCompanyAt` (L382-390), usado por `RoleDefinitionPolicy.heldFor` y por la sesión | Se parte en dos (A-H5) |
| Persistencia | `UserRoleAssignmentPersistenceEntity` (L63-64), `UserPersistenceAssembler` (L104, L134) | `areaId`, `updatable = false` |
| Lectura | `RoleAssignmentView`/`Resource`, `UserRoleResource`, `RoleView`/`RoleResource`/`RoleDetailResource`, titulares reservados (`AccountDutiesReader`, `ReservedHoldersView`) | El área (identificador público y nombre), `limitableToArea` y la causa `assignmentLimitedToArea` |
| Alcance de cada pedido | `CurrentUserAccessScope` (L115-118) | Pasa `scope.areaId()` al `PermissionGrant` |
| Sesión | `SessionCompanyView`/`Resource`, armada en `UserQueryServiceImpl` L134-163 con `User.permissionCodesActingOnWholeCompanyAt` (L382-390) y `permissionCodesActingOnBrandsAt` (L401-411) | Una lista de asignaciones por empresa —sede, marca, área y permisos de cada una— para que compras pregunte exactamente lo que pregunta el backend. `wholeCompanyPermissions` y `brandPermissions` **conservan** las asignaciones con área, porque reflejan los controles que no miran el área (A-H5) |

`shared` **y** `documents`**.**

| Pieza | Cambio |
|---|---|
| `PermissionGrant` | `@Nullable Long areaId`, con un constructor de cuatro argumentos que se conserva (107 usos en 21 archivos de prueba); `readsArea` y `writesArea` |
| `AccessScope` | `anyGrantOf(permiso, condición)`: una misma asignación del permiso cumple a la vez empresa, sede, marca y área. Es la forma de preguntar con área sin multiplicar los métodos de la marca |
| `RecordReach` (nuevo) | La lista de asignaciones de un permiso —empresa, marca y área de cada una— que filtra una lista (A-H1) |
| `DocumentScopes.where` | Agrega `/área` solo cuando la asignación tiene área (A-H7) |

`procurement`**,** `audit` **y** `reporting`**.** Es la parte grande del frente: cada política, cada lectura, cada vista y cada comando de plantillas, órdenes, facturas, notas, pagos y Por pagar recibe el área; la orden guarda la suya desde que nace; la bitácora suma `change_logs.area_id`; el costo de una unidad y el de un equipo, y el reporte de stock, ocultan lo de un área que no se alcanza; la operación «Asignar área» da su área a lo ya emitido. El inventario completo, con sus líneas, está en el plan, fase 7.

**Frontend.** La pestaña Áreas de la ficha de la empresa, el área del colaborador, el área al otorgar un rol —un selector de varias áreas que crea una asignación por cada una—, la sesión con las asignaciones, y en compras: el área de la plantilla (quién la usa), la de la orden (de quien la emite), la de la factura, los filtros, «Asignar área» y la bitácora. El inventario completo está en el plan, fases 6 y 7.

### 3.4 Hallazgos

**A-H1 · El filtro de una lista es la lista de asignaciones.** Hoy `PurchaseOrderAccessPolicy.reach`, `PurchaseOrderTemplateQueryServiceImpl.reach` (L147-168) y `ChangeLogAccessPolicy.reach` guardan, por empresa, la unión de las marcas de todas las asignaciones. Si el área se agregara como otra unión, dos asignaciones «marca X, todas las áreas» y «todas las marcas, área P» darían «todas las marcas y todas las áreas», y una fila de marca Y y área Q se vería sin que ninguna asignación la alcance. El filtro correcto es la lista de asignaciones del permiso: una fila se ve si alguna asignación es de su empresa y, en esa misma asignación, la marca está vacía o coincide, y el área está vacía o coincide. Para facturas, notas, pagos y Por pagar, que se leen sin marca, la misma lista funciona ignorando la marca.

**A-H2 · Leer y escribir con área, y la plantilla común.** Leer: la asignación tiene acceso a todas las áreas, o el registro es general, o coinciden. Escribir: la asignación tiene acceso a todas las áreas, o el registro tiene exactamente el área de la asignación; un documento general solo lo escribe quien tiene acceso a todas las áreas. Difiere de la marca en un punto: una orden de una plantilla sin marca la escribe cualquier asignación de toda la empresa (`PurchaseOrderAccessPolicy.mayAct` L57-61), y una orden general solo la escribe quien tiene acceso a todas las áreas.

Por eso **el área de la plantilla y el de la orden no pueden ser el mismo dato** (auditoría, IM-22). Si la orden copiara el área de su plantilla, como la marca, una plantilla general daría órdenes generales: un comprador limitado a Monitoreo no podría usar el formato común, y lo que emitiera alguien con acceso a todas las áreas lo verían todas. El responsable decidió el 22/11 que un mismo formato sirve a todas las áreas. Entonces:

- **El área de la plantilla dice quién la usa.** Una plantilla general la usan todas las áreas; una de Monitoreo, solo Monitoreo y quien tiene acceso a todas.
- **La orden es del área de quien la emite (Q27).** Con una plantilla de un área, esa área. Con una plantilla general: el área de quien la emite si está limitado a una; una de las suyas, a elegir, si tiene varias; un área o «General», a elegir, si tiene acceso a todas.
- **La orden guarda su área desde que nace**, y sus facturas, notas y pagos la heredan. El área de una plantilla puede cambiar cuando se quiera, porque solo vale para las órdenes futuras: no hace falta el equivalente de `PURCHASEORDERTEMPLATE_BRAND_IN_USE`.

**Hoy todas las plantillas son generales**, y pueden quedar así, como formatos comunes. Lo ya emitido recibe su área con «Asignar área» (A-H14).

**A-H3 · Un documento general puede mostrar datos de un área.** Si una nota o un pago del área P incluyera facturas generales, una factura general —que lee todo el mundo— mostraría la nota o el pago de P en sus notas (`PurchaseInvoiceView.creditNotes`), en su posición de pago (`InvoicePaymentPositionView.applications`), en el costo de la unidad (`UnitPurchaseCostView`) y en los usos del saldo (`CreditNoteView.balance.uses`, `BalanceUseContexts`). Hay dos salidas: filtrar cada lista anidada, o que una nota o un pago incluya documentos de una sola área, contando lo general como un valor propio. La segunda cierra el problema en su origen, y es la que decidió el responsable (Q7).

**A-H4 · Los cálculos por proveedor leen todo.** `JdbcPayableFactsReadRepository` lee las facturas, notas, usos del saldo y consumos de un proveedor (L104, L110, L127, L304), y `PayablePositions`/`SupplierFacts` calculan con ellos lo que se debe por unidad y el saldo. Filtrar esas lecturas por área daría montos equivocados: el área filtra solo lo que se devuelve.

**A-H5 · La regla 5.5 y la sesión preguntan distinto.** `User.permissionCodesActingOnWholeCompanyAt` (L382-390) sirve a la vez para «no otorgar lo que no se tiene» (`RoleDefinitionPolicy.heldFor`) y para la sesión, y `permissionCodesActingOnBrandsAt` (L401-411) alimenta `brandPermissions`. La regla 5.5 tiene que excluir lo que se tiene limitado a un área —como ya excluye lo limitado a una marca—. La sesión, en cambio, tiene que decir exactamente lo que permite el backend, y fuera de compras el backend no mira el área: el inventario de equipos pregunta `actsOnWholeCompanyEveryBrand` (`EquipmentCommandServiceImpl` L266) y las cuentas bancarias `actsOnWholeCompany`. El frontend decide esos botones con `actsOnEveryBrand` y `actsOnWholeCompany` (`F/iam/application/iam.store.ts` L400-424), que leen solo `wholeCompanyPermissions` y `brandPermissions` (`equipment-inventory-view.ts` L94, L156, L165; `equipment-form-dialog.ts` L393; `company-bank-accounts.ts` L83). Si esas listas excluyeran las asignaciones con área, una persona limitada a Monitoreo perdería «Registrar equipo», justo lo que A-H11 protege. Entonces: los métodos se parten en dos; el de la regla 5.5 excluye el área; las listas de la sesión **conservan** las asignaciones con área; compras pregunta por la lista `assignments`, con una misma asignación para empresa, marca y área. Los botones del inventario y de las cuentas bancarias llevan su prueba de regresión.

**A-H6 · El orden de los candados.** `resolveScope` bloquea `FOR SHARE` empresa → sede → marca (ORG-R56); el área va después: empresa → sede → marca → área. Registrar un colaborador bloquea empresa → sede → área; modificarlo bloquea el colaborador `FOR UPDATE`, lee la empresa sin candado (`EmployeeCommandServiceImpl` L223-227) y bloquea sede → área, sin sumar un candado de empresa. Lo prueban `ParentRowLocksTests` y `OrganizationContextFacadeImplLockOrderTests`. El `default` de tres argumentos de `resolveScope` no bloquea área, así que los llamadores de `fleet`, `financing` y `procurement` no cambian su orden.

**A-H7 · Las llaves de los documentos generados.** La llave guardada es un SHA-256 del alcance, recalculado en cada lectura (`DocumentQueryServiceImpl` L79-85). Cambiar el formato para todos haría que los documentos generados antes del despliegue respondan 404. El área se agrega a la llave solo cuando existe.

**A-H8 · Constructores con muchos usos.** Agregar un campo rompe `PermissionGrant` (107 construcciones en pruebas), `RoleScope` (33), `Employee` (44), los comandos del colaborador (39) y `ScopeReference`. Se agregan constructores secundarios y el `default` de `resolveScope`.

**A-H9 · El índice único va en la misma migración que la columna.** Si `user_roles_active_scope_uq` no incluye `area_id`, la segunda asignación del mismo rol en otra área pasa la validación de la aplicación y choca con el índice como un error crudo, no como `ROLE_ASSIGNMENT_CONFLICT`. Conserva su nombre: `RoleAssignmentScopeMigrationTests` L110 lo afirma.

**A-H10 · Lo que el área no oculta, y se acepta.** El número de una factura, una nota o una operación sigue siendo único por empresa, así que un `409` dice que el documento existe en otra área, sin mostrar nada de él. El inventario sigue mostrando hechos de compras sin montos ni números —si la unidad está pagada, si no tiene factura vigente (`PurchaseInvoiceUnitPurchasesAdapter` L62, L77-100)—, porque el inventario es compartido.

**A-H11 · Fuera de compras ninguna regla mira el área.** Los vehículos, los fondos, el inventario de equipos, la carga del impuesto vehicular y la de la garantía mobiliaria no llevan área. Sus métodos `…EveryBrand` siguen siendo solo de marca: si se leyeran como «de todas las áreas», un rol limitado a Monitoreo no podría registrar un equipo del inventario, que es justo su trabajo.

**A-H12 · Lo compartido de compras.** Proveedores y tipos de orden no llevan área: un proveedor lo usan varias áreas. D-134 ya dice que lo que es de todas las marcas lo escribe solo una asignación de toda la empresa y sin marca. El responsable decidió el 22/11 (Q8) un punto medio para no frenar a un área: **cualquier comprador crea un proveedor y le agrega contactos**; **editar o desactivar un proveedor, sus contactos o los tipos de orden lo hace solo quien tiene acceso a todas las áreas**. Así Monitoreo da de alta a su proveedor de GPS sin esperar, pero no le cambia nada al de Habilitación.

**A-H13 · El saldo del proveedor suma todas las áreas.** Pagar con el saldo exige alcanzar todas las áreas (decisión del 22/11). Mostrarlo a quien está limitado a un área revelaría un total de las demás: lo ve y lo usa solo quien tiene acceso a todas las áreas (Q9). Con él sale `SupplierBalanceView.owed`, que también es un total de todas las áreas.

**A-H14 · Lo ya emitido y «Asignar área».** Con las reglas copiadas de la marca, el área de una plantilla no cambia con órdenes (`PurchaseOrderTemplateCommandServiceImpl` L231-255, `brandInUse`) y el de una orden nunca cambia. `dev` tiene las 2 plantillas reales y 9 órdenes sembradas el 09/11: nada de eso podría recibir área, y quedaría visible para todas. El responsable decidió el 22/11 que lo existente también se separa. La salida es una operación única **«Asignar área»**, del superadministrador (permiso nuevo `compras.asignar_area`), sobre **una orden general** o sobre **una factura general sin orden**:

- solo de general a un área, nunca de un área a otra ni de vuelta;
- en la misma transacción, el área baja de la orden a sus facturas, a las notas de esas facturas y a los pagos de esas facturas y de la orden, en ese orden, para que los disparadores de coherencia pasen;
- si una nota o un pago junta esos documentos con otros de otra orden o de otra área, se rechaza y nombra cada documento (Q7);
- los disparadores que guardan lo fiscal (`keep_purchase_invoice_fiscal_data`, `keep_credit_note_data`, `keep_supplier_payment_data`) y el de la orden admiten solo el paso de `area_id` vacío a un valor, una vez;
- deja una fila de la bitácora por documento que cambia; las filas viejas quedan generales, porque la cuenta de la aplicación solo inserta en `change_logs`.

La misma operación da su área a las facturas de equipos registradas antes de que existan las áreas, y por eso el GPS puede ir antes (§7).

**A-H15 · Varias áreas.** Una persona puede tener acceso a una, a varias o a todas las áreas. Para varias, una asignación del mismo rol por cada área (A2): el índice único incluye el área (A-H9) y la lista de asignaciones las suma bien (A-H1). Tener todas las áreas de hoy, una por una, no es tener acceso a todas: esa asignación incluye las áreas futuras y es la única que escribe lo general, el saldo y lo compartido. El diálogo de otorgar un rol ofrece un selector de varias áreas que crea una asignación por cada una; el backend no cambia.

**A-H16 · Lecturas de compras que muestran otra área.** Además de las listas y las lecturas por identificador, el área tiene que filtrar:

| Lectura | Hoy | Evidencia |
|---|---|---|
| `GET /suppliers/{id}/payables`, la fuente del formulario de pago | Facturas sin filtrar, órdenes solo por marca, orígenes del saldo sin filtrar | `SupplierPaymentQueryServiceImpl` L225, L227, L235-238 |
| La verificación del VIN | Devuelve número, proveedor e id de la factura vigente de otra área | `PurchaseInvoiceVinChecks` L134-139 |
| La ficha de un pago con saldo | Nombra las notas de otras áreas que consumió | `SupplierPaymentViewFactory` L329-345 |
| Los montos de pago en la factura y su acción «Registrar pago» | Se muestran con `pagos.ver` de la empresa | `PurchaseInvoiceViewFactory` L196; `PurchaseInvoiceAllowedActions` L80-81 |
| Por pagar | El alcance se arma en el servicio, no en el criterio; la acción `mayPay` no mira el área | `PayableQueryServiceImpl` L92-95, L156 |

Los orígenes del saldo viajan solo a quien tiene acceso a todas las áreas, porque el origen «sin nota» no tiene área. Lo de otra área viaja solo como monto, sin referencia. Vincular a una factura una orden de un área que no se lee responde 404, no un 422 que revela que existe.

**A-H17 · La escritura también es de una sola asignación.** Las marcas de registrar una factura y de la factura atrasada se preguntan en `PurchaseInvoiceUnits` L108 (`registers`) y L114 (`registersOld`). Si el área se preguntara aparte, «marca X, todas las áreas» más «todas las marcas, área P» registrarían una factura de X en P: el mismo error de A-H1, en la escritura. Marca y área se preguntan con una misma asignación.

**A-H18 · El filtro en SQL nativo.** `(:a IS NULL OR area_id IS NULL OR area_id = :a)` con un nulo falla en PostgreSQL por `NamedParameterJdbcTemplate` («could not determine data type of parameter»), y `JdbcPayableFactsReadRepository.openInvoices` arma SQL nativo. El código ya lo esquiva con `CAST(:x AS BIGINT) IS NULL` (`PurchaseInvoiceJpaRepository` L58). Por cada entrada de la lista de asignaciones: ninguna condición de área si tiene acceso a todas; si no, `area_id IS NULL OR area_id = :aN`. Nunca un nulo enlazado.

**A-H19 · La exportación de notas no tiene constante de diseño.** `ProcurementDocumentKinds` L39 es de pagos, L45 de facturas y L51 de Por pagar. Al sumar la columna «Área», una exportación de notas generada antes del despliegue se serviría con las columnas viejas: hace falta `CREDIT_NOTES_LAYOUT`.

### 3.5 Desde dónde se construye

El frente A va después del GPS (§7). Dentro de él:

1. El catálogo de áreas en `organization`, con su tabla, su RLS, su versión, su exportación y sus permisos (fase 6).
2. El área en la asignación y en la sesión, sin exponerla todavía en la pantalla de otorgar un rol (fase 6).
3. Compras por área —también las facturas, notas y pagos de equipos, que ya existirán—, con la bitácora, el costo de la unidad y del equipo, el reporte y «Asignar área»; recién entonces la pantalla deja otorgar un rol con una o varias áreas (fase 7).

---

## 4. Frente B — Menú

### 4.1 Hoy

`F/app-shell/app-navigation.ts` tiene Inicio y cuatro grupos con 30 opciones: Logística (L109-210, secciones Compras y Flota), Configuración (L211-279, etiqueta ADMIN), Catálogos (L280-386, etiqueta SUPER) y Seguridad (L387-437, con la Bitácora). Cada opción pide los permisos de su pantalla y su ruta pide lo mismo (D-117). `CATALOG_READERS` es `catalogos.ver` (L74) y `STRUCTURE_READERS` es `configuracion.ver_estructura` (L81), y los siete roles sembrados tienen los dos: por eso un asesor ve «Catálogos SUPER» con diez opciones y «Configuración ADMIN» con Empresas y Ubicaciones.

### 4.2 Qué cambia

| Pieza | Cambio |
|---|---|
| `app-navigation.ts` | Los grupos nuevos; las rutas no cambian |
| Etiquetas | Sale `badge` de `AppNavigationGroup` (L59) y de `NavigationGroup` (`F/shared/presentation/components/layout/navigation.ts` L37-38), de `layout.html` (L84-86), `layout.scss` (L200-225) y `layout.spec.ts` (L52, L248-264), y las claves `navigation.badge.*`. Reemplaza la regla 7 «Tag» de `DESIGN.md` (L329) |
| Textos | Claves nuevas `navigation.monitoring`, `finance`, `tools`, `administration`; salen `navigation.logistics`, `configuration`, `organization` y `purchasing-and-fleet` si nadie más las usa; «Equipamiento» del catálogo pasa a «Tipos de equipamiento» y el inventario a «Equipos» (con `F/equipment/presentation/equipment.routes.ts` L50, L57 y su spec L69-71) |
| Catálogos por escritura | Cada opción y su ruta piden `{ allOf: [catalogos.ver], anyOf: [<escrituras de la pantalla>] }`: se ajustan `location.routes.ts`, `vehiclecatalog.routes.ts`, `equipment.routes.ts` y `fleet.routes.ts` |
| Matriz de permisos | Se reescribe `permission-matrix-groups.ts` (`AREA_ORDER` L11, `SECTION_ORDER` L17-22, `SCREEN_ORDER` L28-43, `placePermission` y sus ayudantes L109-242), `iam.permission-matrix` en `es.json` (L765-822), `role-access-editor.ts` (L20-23, L159-171) y `permission-matrix.ts` (L313-338) |
| Pruebas | `app-shell.spec.ts`, `layout.spec.ts`, las specs de rutas de cada catálogo, `permission-matrix-groups.spec.ts`, `permission-matrix-rows.spec.ts`; e2e `shell.spec.ts` (el botón «Logística» en L240-257, L335, L363, L453), `permission-matrix.spec.ts`, `roles.spec.ts`, `accessibility.spec.ts` |

### 4.3 Hallazgos

**B-H1 · Mostrar solo a quien escribe, y a quién deja sin qué.** La forma limpia es que la opción y su ruta pidan lo mismo: la lectura y alguna escritura de la pantalla. Romper la igualdad entre menú y ruta (pedir más solo en el menú) obliga a dar un encabezado propio a cada ruta oculta, porque `layout.ts` arma el título y la flecha atrás solo con las opciones visibles (L142-184), y rompe a propósito las specs que lo afirman. Con la matriz sembrada, aplicado a Catálogos, Empresas y Ubicaciones:

| Rol | Pierde |
|---|---|
| Superadministrador | Nada |
| Administrador de accesos | Catálogos, Empresas y Ubicaciones |
| Administrador de empresa | Catálogos y Ubicaciones; conserva Empresas solo por las cuentas bancarias |
| Operaciones, Contabilidad, Asesor, Taller | Catálogos, Empresas y Ubicaciones |

Que el administrador de una empresa no vea las ubicaciones de su empresa es una pérdida real. Aplicado solo a Catálogos —datos del sistema, diez opciones, el ruido mayor—, nadie pierde la vista de los datos de su propia empresa.

**B-H2 · La palabra «área» ya está tomada en la matriz.** `PermissionPlace.area`, `AreaRow` y `iam.permission-matrix.areas` llaman «área» a un grupo del menú. Con el área como concepto del negocio, conviene renombrarlos a «grupo» en la misma fase que reescribe la matriz.

**B-H3 · Las imágenes de referencia.** Las capturas de escritorio y tableta muestran el menú (`E/visual.spec.ts`, `fullPage`): unas 60 pantallas por dos anchos, cerca de 120 imágenes, cambian solo por el menú. Las de teléfono no, porque el cajón está cerrado. Aquí el cambio es intencional, así que se regeneran en un commit propio, después de revisar una muestra; las de la matriz y la ficha del rol también.

**B-H4 · Cuatro pestañas a 375 px.** Las áreas llegan como cuarta pestaña de la ficha de la empresa (Sedes, Marcas comerciales, Cuentas bancarias, Áreas). La skill dice que a 375 px caben unas dos etiquetas cortas, y `E/company-profile.spec.ts` L75-83 afirma tres pestañas sin paginación. Los nombres cortos ya existen desde la revisión 53 (`company-profile-view.html` L109-126, `es.json` L1456-1458): solo falta «Áreas». Es del frente A (fase 6), con su revisión de `DESIGN.md`.

**B-H5 · Catálogos solo para quien escribe deja los catálogos al superadministrador.** Ninguna migración da una escritura de catálogo a otro rol (D-150), así que con Q3 solo el superadministrador abre un catálogo, también por URL: Contabilidad pierde Tipos de cambio, y el administrador de empresa y Operaciones pierden Estados y motivos y Requisitos de equipos. Es lo decidido; cambia D-141, §5.6 y DP-18 del diccionario, que dicen que un catálogo se abre con `catalogos.ver`. Rompe además los e2e de catálogos con roles de solo lectura (`E/exchange-rates.spec.ts` L410-413, `E/vehicle-document-types.spec.ts` L26 y L88-90, `E/vehicle-equipment-requirements.spec.ts` L36-41) y más casos de `F/app-shell/app-shell.spec.ts` (L91-104, L149-173, L195-215). El e2e no tiene sesiones de OPERACIONES, CONTABILIDAD ni TALLER (`E/support/api.ts` L92-103): los siete roles se afirman en `app-shell.spec.ts`, que ya lee `SEEDED_MATRIX`.

**B-H6 · El filtro de área no cabe en la barra de Facturas y Notas.** No existe un panel «de más filtros»: el gestor de datos pliega sus filtros solo en el teléfono (`data-manager.html` L4-26), y `E/toolbar-rows.spec.ts` L21-22 afirma una sola línea. El filtro de área necesita un patrón propio y su revisión de `DESIGN.md`.

### 4.4 Desde dónde se construye

Es solo frontend y documentación, no cambia una ruta ni el contrato de la API, y no depende de los otros frentes. Va primero, después de la fase 0 con el e2e en verde: es el cambio más visible, el más barato, y deja la matriz y los grupos listos para recibir las pantallas de Monitoreo y la pestaña de Áreas.

---

## 5. Frente C — Ciclo de compra del GPS y SIM

### 5.1 Hoy

| Pieza | Dónde | Cómo está |
|---|---|---|
| El equipo | `B/equipment/domain/model/aggregates/Equipment.java` | Nace siempre Disponible, con ubicación obligatoria (L56, L93); el modelo es `final` (L26); IMEI `^\d{15}$` e ICCID `^\d{19,20}$` sin dígito verificador (L17-18); estados `AVAILABLE`, `INSTALLED`, `MAINTENANCE`, `DISCARDED` |
| Su tabla | `fleet_equipments` (V132, V134) | `location_id NOT NULL` (L19), `equipment_model_id NOT NULL` (L22), `created_by NOT NULL` (L35); `supplier_id` opcional, con llave compuesta al proveedor de la empresa; `imsi` texto libre `VARCHAR(30)` (L31); INSTALLED exige `current_vehicle_id` (L49-51); índices únicos de serial, IMEI e ICCID **en todo el sistema**, que excluyen `DISCARDED` (L55-65); `status_reason` (V134) se sobrescribe; RLS; sin `data_versions` (no se exporta) |
| Sus reglas | `EquipmentCommandServiceImpl` | EQP-R13 (ubicación al registrar; el proveedor es opcional, L96-104), EQP-R14 (un tipo telemático exige serial, IMEI e ICCID, L364-369), EQP-R16 (identificadores solo en Disponible) |
| Su lectura | `EquipmentViewFactory` L92-93 | `requireReference(location)`: un equipo sin ubicación hace fallar la vista |
| Su fachada | `EquipmentContextFacade` (L16-52) | Solo tipos, aplicabilidad y la versión de la habilitación: nada de equipos ni de modelos |
| Línea de orden | `PurchaseOrderItem.parts()` (L112-146); V69 L86-87 | Modelo de vehículo con año, o concepto |
| Línea de factura | `PurchaseInvoiceItem` | Vehículo, modelo, año, marca; `isConcept()` es `vehicleId == null`. Límite de 100 líneas en `PurchaseInvoice.MAX_LINES` (L39), `@Size(max = 100)` de `RegisterPurchaseInvoiceResource` (L38), `InvoiceLineImportPolicy.MAX_ROWS` (L32), `StockRowGroups` (L60) y `CheckPurchaseInvoiceVinsQuery` |
| El alta desde la factura | `PurchaseInvoiceCommandServiceImpl` L106-244 → `PurchaseInvoiceUnits` (resolve L97-138, match L145-197, register L204-220) → `FleetContextFacade.registerOrLinkPurchasedUnits` → `PurchasedUnitRegistrations` (`Propagation.MANDATORY`) | El molde del alta de equipos |
| La nota | `CreditNoteReason` (L14-18, `returnsUnits()` L33-35), `CreditNoteUnits` → `FleetContextFacade.annulReturnedUnits` | Solo de crédito (D-23); límite de 100 líneas (`CreditNote.MAX_LINES` L37) |
| El pago | `payment_allocations` (V107 L130-150), `SupplierPaymentAllocations`, `InvoiceUnitCoverages`, `UnitCoveragePolicy`, `JdbcPayableFactsReadRepository` | Por unidad; 500 imputaciones por pago (`SupplierPayment.MAX_ALLOCATIONS` L46). Nada exige imputaciones salvo el canal concesionario (`allocationRequired`, `SupplierPaymentAllocations` L77-81): la base revisa una imputación solo al insertarla (V107 L207-225), y `AmountOwedPolicy.toBalance` (L112-117) ya pasa el exceso a saldo por factura |
| El costo de la unidad | `UnitPurchaseCostsController` (`GET /vehicles/{id}/purchase-cost`), `UnitPurchaseCostQueryServiceImpl`, `UnitPurchaseCostCalculator`, `ProcurementContextFacade.fetchUnitPurchaseCosts` | El molde del costo del equipo |
| Cumplir la orden | `PurchaseOrder.fulfill()` (L201-206), `POST /purchase-orders/{id}/fulfillment` sin cuerpo | Manual, sin motivo (D-162) |
| La instalación | `VehicleEquipmentsController`, `VehicleEquipmentInstallations.check` (L70-110); retirar, reemplazar, completar el serial y `lockedWithItsUnit` (L238-249) en `VehicleEquipmentCommandServiceImpl` | El serial se escribe a mano; retiro solo por falla o robo; ni la anulación ni el paso a Inactivo (`VehicleMovements` L131, L150, con `TransferExitPolicy`) miran las instalaciones. `Equipment.installOnVehicle` existe y nadie la llama |
| La carga inicial | `MigratedUnitRegistrations` (L68, L342-348; D-206) | Crea instalaciones de GPS sin serial y sin equipo del inventario. Correrá en la salida a producción |
| Los correos | `notifications` (V125; `NotificationDispatchTask` L78-94; `RetryPolicy`; `NotificationFailureWatchTask`; proveedor elegido por `@ConditionalOnProperty`, con `EmailProviderCheck` al arrancar; métricas Micrometer; reclamo con `FOR UPDATE SKIP LOCKED`, donde `next_attempt_at` hace de *lease*) | El molde de la API del fabricante. El planificador tiene un solo hilo (`SchedulingConfiguration` sin `pool.size`) |
| La firma de lo automático | `IamContextFacade.fetchSystemAccountId()`, que hoy solo usa `fleet` (`ExternalIamService` L37) | `equipment` no depende de `iam::acl` |
| El sitio | `public/_headers` L10 | `Permissions-Policy: camera=()` para todo `/*` |
| La ficha del equipo | `EquipmentDetailsDialog` (`equipment-inventory-view.ts` L258) | Un diálogo; la regla 4.9 del frontend prohíbe un diálogo de formulario sobre otro |
| Las cargas | `imports` (`ImportLocks`, `ImportWorkbookReader`, `ImportReportWriter`) | Disponibles si algún día se recibe por Excel |

### 5.2 Qué se reutiliza

- El flujo completo del alta de unidades, con su propagación obligatoria y su componente que no se llama `*CommandServiceImpl` (así `DeadlockRetry` repite solo el servicio de afuera).
- El mecanismo de la nota que lleva una línea a cero y le pide al dueño que anule.
- El cálculo del costo de compra y su lectura de lo que la sesión ve en el otro contexto.
- El patrón completo de los correos para la API del fabricante.
- La forma de la instalación (período `[inicio, fin)` con `EXCLUDE`) para la asignación SIM ↔ GPS.
- `EQUIPMENT_IMEI_INVALID` y `EQUIPMENT_STATUS_TRANSITION_INVALID`, que ya existen, para el dígito verificador y los caminos de estado nuevos.

### 5.3 Qué cambia, por contexto

| Contexto | Cambio |
|---|---|
| `procurement` | Tipo explícito de línea en órdenes y facturas, con `isVehicle()` e `isEquipment()` junto a `isConcept()`; tipo de orden y de factura (vehículos o equipos, nunca los dos); línea de equipo en órdenes (modelo de equipamiento y cantidad) y en facturas (un equipo por línea); lo pendiente por modelo con el tipo en la llave; alta de equipos por la fachada de `equipment`, que vincula los equipos sin línea vigente al «Anular y copiar»; nota «anulación de equipo»; pago de una factura de equipos a nivel de factura; costo del equipo; motivo al cumplir con faltante; límites de 500 para lo de equipos, con un INSERT de varias filas |
| `equipment` | Estados nuevos; ubicación e identificadores opcionales hasta el ingreso; la historia de estados, de solo inserción; alta por la fachada; ingreso; devolución al proveedor; reemplazo en cadena; anulación; la SIM y su asignación; el pedido de datos al fabricante con su tarea, sus intentos y su vigilante; el código del fabricante en el modelo; un modelo heredado por tipo; la instalación por la fachada; la firma `SYSTEM` por `iam::acl` |
| `fleet` | La instalación toma un equipo del inventario, exige SIM en un tipo telemático, retira con destino; la unidad con equipos instalados no se anula ni pasa a Inactivo (en `VehicleMovements`); la vinculación de lo ya instalado, también desde la carga inicial y al completar el serial; las instalaciones de un equipo, que sirve en una ruta propia |
| Frontend | Línea de equipo en la orden; tipo y líneas de equipo en la factura; nota sobre equipos; motivo al cumplir; estados, acciones y aviso de antigüedad del inventario; la ficha del equipo como página, con su historial; pantalla de Ingreso de equipos con escaneo por modelo; pantalla de SIM, con su ficha como página; diálogo de instalación; `camera=(self)` en `_headers` |

### 5.4 Hallazgos

**C-H1 · La línea necesita su tipo.** Hoy toda regla que cuenta unidades ignora una línea sin `vehicle_id`, porque `isConcept()` es `vehicleId == null`. Una línea de equipo sería un concepto para facturas, notas y pagos. El tipo explícito (`VEHICULO`, `EQUIPO`, `CONCEPTO`) va primero, con las filas actuales llenadas desde lo que tienen.

**C-H2 · El reemplazo no puede mover la línea.** `keep_purchase_invoice_item_fiscal_data` (V103 L215-221) compara una lista cerrada de columnas fiscales, entre ellas `vehicle_id`, y la fase 2 suma `equipment_id`: no se edita a qué apunta una línea. Una columna nueva fuera de esa lista sí se puede llenar, así que el relleno del tipo de línea no necesita desactivar el disparador; basta con rellenar antes del `CREATE OR REPLACE` que la suma. El reemplazo se registra en el equipo nuevo, que apunta al que reemplaza (`replaced_equipment_id`, único: un equipo se reemplaza una vez); la línea queda en el original. El costo y la factura del equipo nuevo se siguen por esa cadena hasta su raíz. Así se cumple lo que decidió el responsable —el nuevo toma la línea del fallado— sin tocar un dato fiscal.

**C-H3 · Los pagos de equipos, a nivel de factura.** Las imputaciones por unidad existen por reglas de los vehículos: la brecha de días, la inicial del cliente y los fondos. Ninguna aplica a un GPS. Generalizar las imputaciones a líneas toca el motor de pagos completo (`SupplierPaymentAllocations`, `InvoiceUnitCoverages`, `UnitCoveragePolicy`, el SQL de `JdbcPayableFactsReadRepository` y el disparador `enforce_payment_allocation_unit`). Pagar una factura de equipos como un todo, con la nota que baja primero lo que se debe y pasa el exceso a saldo, da el mismo resultado de negocio con una fracción del riesgo (decidido, Q12). Y ya funciona casi solo: la base revisa una imputación solo al insertarla (V107 L207-225), `Spreads.of` devuelve vacío sin unidades (`InvoiceUnitCoverages` L67-78), `OrderPaymentTransfers` pasa sin partes (L117-126) y `AmountOwedPolicy.toBalance` (L112-117) ya pasa el exceso a saldo por factura. Lo único que estorba es `allocationRequired` del canal concesionario (`SupplierPaymentAllocations` L77-81), que exime las facturas de equipos. No hace falta una migración de pagos ni una política de cobertura nueva.

**C-H4 · El equipo sin ubicación.** La ubicación es obligatoria en el agregado, en la tabla y en la vista. Un equipo «Por ingresar» no la tiene, y uno instalado está en la unidad: el constructor de reconstitución, `EquipmentPersistenceEntity` (L33, `nullable = false`), `EquipmentView.locationPublicId`, `EquipmentResource` (L23) y `EquipmentViewFactory` (L67, L92-93) aceptan el nulo, y la tabla cambia `NOT NULL` por un `CHECK` atado al estado que libera PENDING_ARRIVAL, ANNULLED e INSTALLED.

**C-H5 · El modelo del equipo.** Es `final`. Un equipo en «Revisar» porque el fabricante dice otro modelo necesita poder corregirse; la línea de la factura conserva el modelo que dice el comprobante, como pasa con la unidad (D-160).

**C-H6 · El límite de 100 está en seis lugares**, y la nota y las imputaciones tienen el suyo. Los seis de la factura: `PurchaseInvoice.MAX_LINES` (L39), `RegisterPurchaseInvoiceResource` (L38), `InvoiceLineImportPolicy.MAX_ROWS` (L32), `StockRowGroups` (L60), `PurchaseInvoiceVinChecks.MAX_VINS` (L42) y `CheckPurchaseInvoiceVinsQuery`. La nota: `CreditNote.MAX_LINES` (L37), que no conoce el tipo de la factura, y `RegisterCreditNoteResource` L31 (`@Size(100)`). Una factura de 500 GPS anulada entera necesita una nota de 500 líneas.

**C-H7 · El equipo no tiene marca comercial.** El conjunto de marcas de una factura de equipos queda vacío y el control de marca pasa sin preguntar (`PurchaseInvoiceAccessPolicy.actsOnLines` L25-28): un comprador limitado a una marca podría registrarla. Pasa lo mismo al anular y modificar la factura (L25-44), en la nota (`CreditNoteAccessPolicy.writes` L33-36) y en el pago. Un equipo es de todas las marcas, así que lo coherente con D-134 es pedir una asignación sin marca para toda escritura sobre un documento de equipos (Q14).

**C-H8 · Los códigos de estado.** Los del equipo están en inglés y mayúsculas (`AVAILABLE`); los nuevos siguen esa forma en su tabla. Las listas cerradas de compras están en español y mayúsculas (`VIGENTE`, `ANULACION_UNIDAD`), y lo nuevo de compras sigue esa.

**C-H9 · El dígito verificador.** No existe hoy. Se valida al escribir (escaneo, edición, alta de SIM), nunca al leer ni en un `CHECK` de la base, porque los IMEI e ICCID ya guardados pueden no cumplirlo.

**C-H10 · La SIM sale del equipo.** El ICCID y el IMSI solo existen en `equipment` y en V132: ningún código de `fleet` ni de `procurement` los toca. El índice viejo excluía `DISCARDED`, así que dos equipos de baja pueden compartir ICCID; la migración lo informa y conserva las columnas viejas una entrega más, como V142 → V143.

**C-H11 · Anular una factura de equipos.** D-161 dice que las unidades no se anulan con su factura, porque existen. Un equipo «Por ingresar» todavía no existe físicamente: si la factura se anuló por un error de registro, sus equipos por ingresar quedarían colgados. Lo coherente es anular con la factura los equipos que siguen por ingresar; los ya recibidos quedan, sin factura vigente, como la unidad. Vale igual para la nota «anulación de la factura» (Q15), y la fachada distingue «solo los por ingresar» de lo que anula la nota «anulación de equipo».

**C-H12 · D-264 y la nota.** Una unidad devuelta por una nota que tiene un GPS instalado tiene que quedar retenida, igual que la que tiene un contrato (D-191, D-250), y entrar en lo que D-266 informa.

**C-H13 · La vinculación de lo ya instalado.** D-263 pide enlazar cada instalación abierta de GPS a su equipo por el serial, y registrar el que falte. Una migración SQL de `fleet` no puede escribir ni leer `fleet_equipments` (reglas 1.2 y 1.3). Lo correcto es un proceso idempotente de `fleet`, en una sola instancia y con alcance `SYSTEM`, que pregunta y registra por la fachada de `equipment`. Las instalaciones sin serial no se enlazan y se listan para completarlas. Lo que le falta para funcionar está en C-H19.

**C-H14 · El fabricante.** El responsable dijo que la API es del fabricante, sin nombrarlo. Se construye la interfaz, el proveedor «manual» (sin llamadas: el equipo queda por completar y se completa a mano), la tarea, los intentos y el vigilante; el adaptador real espera la documentación y las credenciales.

**C-H15 · Comparar el modelo.** Para decir «el fabricante dice otro modelo» hay que saber cómo nombra el fabricante cada modelo del catálogo. `equipment_models` no lo guarda: se le agrega un código del fabricante, opcional; sin él, la comparación no se hace y la ficha lo dice.

**C-H16 · Leer el QR.** El frontend no tiene ninguna librería de lectura. Un lector USB o Bluetooth escribe como un teclado y funciona en cualquier navegador; es la forma principal (Q22). La cámara necesita `BarcodeDetector`, que no existe en todos (en iPhone no), o una librería cargada solo en esa pantalla, para no pesar en el resto; los presupuestos de `angular.json` (L46-57) solo miden el paquete inicial, así que esa librería se vigila aparte. Y hoy el sitio bloquea la cámara (C-H26).

**C-H17 · El ingreso no necesita que** `equipment` **conozca las facturas.** La factura de equipos, que es de `procurement`, ya conoce sus equipos y lee sus estados por la fachada de `equipment`. La pantalla de ingreso toma de la factura los equipos por ingresar y escanea cada uno contra `equipment`: ninguna dependencia nueva. Cuando llegue la fase 7, el área de la factura decide quién la ve; antes, la ve quien lee compras en su empresa.

Los hallazgos que siguen vienen de la auditoría de la versión 1.

**C-H18 · El historial del equipo.** El correo al cliente del 16/11 (L52-55) promete «el historial de en qué vehículo estuvo cada equipo y cuándo se retiró o reemplazó». Hoy no existe: `equipment` no escribe la bitácora, y `status_reason` (V134 L8) se sobrescribe en cada cambio, así que el motivo de «Devolver al proveedor», de «Revisar» o de la anulación se perdería (regla 4.3). Hacen falta dos piezas:

- una tabla de solo inserción `equipment_status_history` —desde, hasta, motivo, quién y cuándo—, como `vehicle_status_history` (§8.4), que reciben todos los caminos; cada equipo de hoy recibe una fila de partida firmada por su propio autor, sin leer `iam`;
- las instalaciones de un equipo, que sirve `fleet` en `GET /api/v1/equipments/{id}/installations`, como `procurement` sirve `GET /vehicles/{id}/purchase-cost`: `equipment` no puede leer `vehicle_equipments`, porque `fleet` depende de `equipment` y no al revés.

La ficha del equipo muestra las dos (C-H27).

**C-H19 · La vinculación de lo instalado, completa.** C-H13 no alcanza tal como está:

- registrar un equipo instalado choca con `equipment_model_id NOT NULL` (V132 L22), con la ubicación (L19) y con `created_by NOT NULL` (L35), y `vehicle_equipments` solo guarda tipo y serial, no modelo. Hace falta un modelo heredado por tipo de equipamiento, sembrado; el equipo instalado no lleva ubicación (C-H4); la firma es la cuenta `SYSTEM` (C-H25);
- el serial es único en todo el sistema (L55-65): si el equipo con ese serial está Disponible en la misma empresa y el mismo tipo, se instala; si es de otra empresa, de otro tipo o de otro estado, se informa y no se enlaza;
- la carga inicial (`MigratedUnitRegistrations` L68, L342-348; D-206) crea instalaciones de GPS sin serial y sin equipo, y correrá en la salida a producción, después de este trabajo. Tiene que crear o enlazar el equipo por la fachada de `equipment`, o la vinculación se ofrece como proceso que se corre después de ella, no solo al arrancar;
- completar el serial después (`PUT /vehicle-equipments/{id}/serial`, `VehicleEquipmentCommandServiceImpl` L200-223) hoy solo escribe el serial: tiene que enlazar en la misma transacción, o esa instalación escapa de D-264.

**C-H20 · El escaneo por modelo.** Si cada lectura va «al siguiente equipo por ingresar de la factura», en una factura con dos modelos un IMEI del modelo B cae en el lugar de uno del A y hereda el precio y el modelo de la línea equivocada. Con el fabricante termina en «Revisar»; con el proveedor `manual`, nadie lo nota. Si la factura tiene más de un modelo, la pantalla pide el modelo (la caja) antes de escanear, y cada lectura va al siguiente pendiente de ese modelo. Dos personas escaneando la misma factura chocan en el mismo equipo: la segunda recibe `EQUIPMENT_STATUS_TRANSITION_INVALID` y la pantalla pasa al siguiente.

**C-H21 · `isConcept()` no puede pasar a ser «tipo = concepto».** Tiene 13 usos en 9 archivos, y en todos `!isConcept()` significa «línea de vehículo». `PurchaseInvoice` L140 agrega `getVehicleId()` a un `Set` para detectar unidades repetidas: con dos líneas de equipo rechazaría la factura. También `InvoiceUnitCoverages` L97 y L112 (`requireNonNull(getVehicleId())`), `PurchaseInvoice` L315, `SupplierPaymentApplications` L330, `CreditNoteLineEffects` L71, `SupplierPaymentAllocations` L220, `CreditNoteAnnulments` L118 y L132, y `CreditNoteViewFactory` L228. Al revés, `CreditedLines` L85 (anular la factura entera) sí tiene que contar las líneas de equipo. Entran `isVehicle()` e `isEquipment()`, con una decisión explícita y una prueba por uso. El SQL que filtra `vehicle_id IS NOT NULL` ya es correcto.

**C-H22 · Insertar 500 líneas.** No hay `hibernate.jdbc.batch_size` (`application.yaml` L15-27) y todo id es `IDENTITY` (`CreatedAbstractPersistenceEntity` L31), que Hibernate nunca agrupa. `bump_data_version` corre por sentencia (V94 L276-281): 500 INSERT son 500 actualizaciones de `data_versions`. PRC-R65 no fija un tiempo: solo exige que las consultas que no son inserciones no crezcan (64 hoy, `PurchaseInvoiceRegistrationStatementsTests` L42). Los equipos y las líneas se insertan con un INSERT de varias filas (`unnest … RETURNING id`, o JDBC con `reWriteBatchedInserts`), y la meta de tiempo se fija midiendo contra `dev`, por la latencia real a la base. El corte de Cloudflare es de 100 s.

**C-H23 · «Anular y copiar».** Corregir una factura es anularla y copiarla (D-211, D-214, PRC-R66), o emitir una nota «anulación de la factura». Si el alta siempre creara equipos nuevos, la copia duplicaría los por ingresar, y los ya recibidos nunca volverían a tener línea vigente. La unidad se vincula por VIN (D-161); el equipo por ingresar no tiene identificadores. El alta vincula primero los equipos de la misma empresa, proveedor y modelo que no tienen línea vigente ni están anulados.

**C-H24 · La máquina de estados, completa.** Anular tiene que admitir MAINTENANCE, PENDING_COMPLETION y UNDER_REVIEW (una nota por un equipo con IMEI desconocido no podría registrarse); una nota que nombra un equipo instalado o no anulable se rechaza entera y nombra los equipos; «Volvió reparado» se rechaza si otro equipo ya lo reemplazó (si no, dos equipos vivos en una línea y el costo contado dos veces); «anulación de equipo» sobre un original reemplazado anula el extremo vivo de la cadena; corregir el modelo queda dentro del tipo (`equipment_type_id` es final y la llave de la instalación lo usa); `replaced_equipment_id` va con llave compuesta `(id, company_id)`.

**C-H25 · La firma `SYSTEM` en `equipment`.** La tarea del fabricante escribe `updated_by` y crea SIM (`created_by NOT NULL`), y la vinculación registra equipos. El id de la cuenta `SYSTEM` solo lo da `IamContextFacade.fetchSystemAccountId()`, que hoy usa `fleet` (`ExternalIamService` L37), y `equipment` no depende de `iam::acl`. `equipment` suma `iam::acl`. Una migración que firma como `SYSTEM` declara su paso de `iam`, como V145; la de la SIM copia el autor de cada equipo y no lo necesita.

**C-H26 · La cámara está bloqueada.** `public/_headers` L10 envía `Permissions-Policy: camera=()` a todo `/*`, y en una aplicación de una sola página no sirve un encabezado por ruta. Pasa a `camera=(self)`. El e2e no lo ve, porque el servidor de desarrollo no aplica `_headers`.

**C-H27 · La ficha del equipo, como página.** Hoy es `EquipmentDetailsDialog` (`equipment-inventory-view.ts` L258). Abrir desde ella «Asignar SIM», «Registrar ingreso» o «Completar» pondría un diálogo de formulario sobre otro, que prohíbe la regla 4.9 del frontend. La ficha del equipo y la de la SIM pasan a rutas propias (`/equipment/inventory/:id`, `/equipment/sim-cards/:id`), con el historial (C-H18).

**C-H28 · Una orden, un solo tipo.** La plantilla da área y marca a la orden. Una orden mixta mezclaría las compras de dos áreas, y chocaría con C-H7: la orden la escribe quien tiene la marca de la plantilla y la factura de equipos pide una asignación sin marca. `PurchaseInvoiceOrderLinks.deviations` (L71-83) nombra modelos de vehículo y `OrderInvoicingPolicy.Deviation` tiene `int year` (L56). Una orden es de vehículos o de equipos, y la factura vinculada tiene el tipo de su orden (Q24).

**C-H29 · El aviso de antigüedad.** Decidido el 22/11 junto a C7: los que faltan quedan por ingresar con un aviso de antigüedad antes de la nota. La lista y la ficha muestran los días desde la factura, con un umbral configurable.

**C-H30 · Sin fabricante, el ingreso a mano.** Con el proveedor `manual`, cada equipo escaneado queda «Por completar» y pide serial y firmware para estar Disponible (EQP-R14): cientos de completados uno por uno. Hasta que exista el adaptador, el ingreso completo a mano de la fase 2 sigue como camino normal (Q25).

**C-H31 · El traslado a la SIM.** `imsi` es texto libre (`VARCHAR(30)`, V132 L31): un `CHECK` de 15 dígitos rompería la migración, así que los inválidos se vacían con un `NOTICE`. Los equipos ANNULLED se tratan como DISCARDED. Un equipo de baja creado y cambiado el mismo día daría un rango vacío: se usa `[alta, greatest(cambio, alta + 1))`. Después del traslado, `POST /equipments`, `PATCH` y `/arrival` siguen aceptando `iccid`, EQP-R14 lo exige al crear, y la búsqueda, `existsByIccid` y el mapeo de la restricción lo nombran (`EquipmentCommandServiceImpl` L364-369; `EquipmentJpaRepository` L61-66, L78; `EquipmentRepositoryImpl` L35-37). Lo coherente con C-H10 es que un ICCID recibido cree y asigne su SIM; la limpieza retira lo que quede.

**C-H32 · Los resultados viejos del fabricante.** Un completado a mano o una revisión puede llegar con un pedido en curso; un ICCID que ya existe en otra empresa hace fallar «buscar o crear la SIM» por el índice único global. Los caminos a mano cancelan el pedido abierto; aplicar un resultado vuelve a mirar el equipo y el pedido bajo candado; un solo orden de candados (equipo → pedido, equipo → SIM); el ICCID ajeno es `SIM_MISMATCH`. El planificador tiene un solo hilo: una llamada de 5 s + 10 s atrasa los correos, así que se fija su tamaño o se acota cada corrida. `DeadlockRetryTests` lista las tareas exactas (L123-126) y suma las nuevas.

### 5.5 Desde dónde se construye

El frente C va antes que las áreas (§7):

1. La compra (fase 2): el tipo de línea y de orden, la factura de equipos, los estados de compra del equipo y su historia, la nota, el pago, el costo, el faltante y el aviso de antigüedad, con el ingreso completo a mano para que la fase quede usable sola.
2. El ingreso por escaneo por modelo, la API del fabricante y la SIM (fase 3).
3. La instalación unida al inventario, las instalaciones de cada equipo y la vinculación de lo ya instalado, también desde la carga inicial (fase 4).
4. La limpieza de las columnas viejas de la SIM (fase 5).

---

## 6. Lo transversal

### 6.1 Las migraciones

| Versión | Contexto | Fase | Qué |
|---|---|---|---|
| V148 | `procurement` | 2 | El tipo de línea, el tipo de factura y el tipo de orden |
| V149 | `equipment` | 2 | Los estados de compra del equipo y la ubicación opcional (libre en PENDING_ARRIVAL, ANNULLED e INSTALLED) |
| V150 | `equipment` | 2 | `equipment_status_history`, de solo inserción, con una fila de partida por equipo firmada por su propio autor |
| V151 | `procurement` | 2 | Las líneas de equipo y la nota «anulación de equipo» |
| V152 | `procurement` | 2 | El motivo al cumplir con faltante |
| V153 | `iam` | 2 | `stock.ingresar_equipamiento` |
| V154 | `equipment` | 3 | La SIM y su asignación, con el traslado de lo existente |
| V155 | `equipment` | 3 | Los estados de completado, el pedido al fabricante y sus intentos, el código del fabricante en el modelo, `replaced_equipment_id` |
| V156 | `iam` | 3 | Los permisos de la SIM |
| V157 | `equipment` | 4 | Las llaves del equipo que pide la instalación (empresa y tipo; la unidad actual) y un modelo heredado por tipo de equipamiento |
| V158 | `fleet` | 4 | La instalación unida al inventario |
| V159 | `equipment` | 5 | Retira el ICCID y el IMSI del equipo, y el disparador de compatibilidad de V148 (con un paso de `procurement`) |
| V160 | `organization` | 6 | La tabla `areas` |
| V161 | `iam` | 6 | Los permisos de las áreas y sus marcas |
| V162 | `iam` | 6 | `user_roles.area_id` y el índice único |
| V163 | `organization` | 6 | `employees.area_id` |
| V164 | `procurement` | 7 | El área de plantillas, órdenes (la suya propia), facturas, notas y pagos, con disparadores que admiten el paso de vacío a un área una vez |
| V165 | `audit` | 7 | `change_logs.area_id` |
| V166 | `iam` | 7 | `compras.asignar_area`, marcado en `SUPERADMIN` |

Si el orden de integración cambia, se renumeran al abrir la rama: Flyway exige que la versión siga a la última aplicada.

### 6.2 Lo que cada migración nueva arrastra

- Una tabla con `company_id`: sus tres políticas de V29, su lugar en `COMPANY_TABLES` de `RowLevelSecurityMigrationTests` (afirmada exacta), su fila en el accesorio de esa prueba y su prueba de aislamiento con la cuenta de la aplicación.
- Una tabla que se exporta: sus 64 casillas y sus dos disparadores de `data_versions` (V108 es la plantilla).
- Una función nueva: su línea `ALTER FUNCTION … OWNER TO erp_mm_owner` en `scripts/database/hand-over-database.sql`; una tabla de solo inserción, su línea de permisos. `HandOverDatabaseScriptTests` falla mientras falte.
- Toda migración abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos; las que tocan `user_roles`, `employees` o `change_logs`, también, y las restricciones nuevas sobre tablas grandes van `NOT VALID` y luego `VALIDATE`.
- Una tabla con columnas que la aplicación actualiza por separado: su lugar en `COLUMN_UPDATE_TABLES` de `RowLevelSecurityMigrationTests` (L90).

### 6.3 La API

Agregar un campo a una respuesta o un campo opcional a un pedido es compatible. Cambiar el cuerpo de la instalación (de un serial escrito a un equipo del inventario) no lo es: va en tres pasos, como manda el §1 de las convenciones de la API (el backend acepta las dos formas, el frontend pasa a la nueva, el backend retira la vieja). Las rutas nuevas son `GET /api/v1/equipments/{id}/installations` (de `fleet`), `GET /api/v1/equipments/{id}/purchase-cost` (de `procurement`), las del equipo, la SIM y las áreas, y la de «Asignar área».

### 6.4 La documentación que cambia

El diccionario: §2.5, §5.3, §5.6, §5.7, §6, §7.5, §8.4 (la historia del equipo como la de la unidad), §9.2 a §9.6, §10, §11.4, §13; la enmienda de D-141, §5.6 y DP-18 (Catálogos solo para quien escribe); y las decisiones nuevas:

| Decisión | Qué recoge |
|---|---|
| D-283 | La compra de equipos |
| D-284 | Cumplir una orden con faltante pide su motivo |
| D-285 | El ingreso por escaneo, la API del fabricante y la SIM |
| D-286 | El historial del equipo y la vinculación de lo instalado; precisa D-263, D-264 y D-206 |
| D-287 | Las áreas |
| D-288 | Compras por área y «Asignar área» |

También `contextos/organization.md`, `iam.md`, `procurement.md`, `audit.md`, `reporting.md`, `equipment.md` y `fleet.md`; las convenciones de la API (rutas nuevas); `procesos-y-pantallas` (§3.1 y §3.2, y los procesos P4, P5 y P6); `DESIGN.md` (desde la revisión 99, con una revisión para el patrón del filtro de área); el README del backend, `.env.example` y `.do/app.yaml` (las variables del fabricante); y las cuatro copias locales de las skills.

---

## 7. Conclusión: el orden

El responsable decidió el 22/11 (Q26) el orden **menú → GPS → áreas**, en vez de áreas primero:

- el módulo de monitoreo es la próxima etapa que se le anunció al cliente el 16/11, y con las áreas primero se atrasaba unas cuatro semanas;
- no hay producción todavía, así que el aislamiento por área no protege datos reales mientras tanto;
- «Asignar área» (A-H14) da después su área a las facturas de equipos registradas antes.

El costo es que compras por área cubre también las facturas, notas y pagos de equipos, que ya existirán.

| Orden | Fase | Por qué en ese lugar |
|---|---|---|
| 0 | Preparación | El CI del frontend tiene que estar en verde, con el e2e completo, antes de integrar nada |
| 1 | Menú | No depende de nada, es lo más visible y deja la matriz lista para lo que sigue |
| 2 | Compra de equipos | El GPS nace en la factura, con su historia de estados desde el primer día |
| 3 | Ingreso, fabricante y SIM | Sobre equipos que ya nacen por ingresar |
| 4 | Instalación unida al inventario e historial del equipo | Necesita la SIM para exigirla al instalar; cierra lo que el correo del 16/11 prometió |
| 5 | Limpieza | Una entrega después de la 3, cuando nada lee las columnas viejas, y con la 4 integrada |
| 6 | Áreas: catálogo, asignación y sesión | Es la base de la fase 7 |
| 7 | Compras por área y «Asignar área» | Aísla a Monitoreo de Habilitación en todo compras, también en lo de equipos y en lo ya emitido |

**El primer tramo (Q28, Q29).** Después, el mismo día, el responsable precisó la prioridad: primero el flujo operativo hasta el ingreso de los IMEIs —la fase 0 reducida, la compra de equipos (fase 2) y el ingreso por escaneo sin fabricante con el reemplazo de un fallado (fase 3a)—; la SIM, la API del fabricante, la instalación, el menú y las áreas van después, en el orden que él diga. La columna del reemplazo pasa a V149, así que ninguna migración cambia de número (plan, §5 y §9).

---

## 8. Registro

| Versión | Fecha | Qué |
|---|---|---|
| 1 | 22/11/2026 | Primera versión, con cuatro revisiones de solo lectura |
| 2 | 22/11/2026 | Recoge la auditoría y las decisiones del responsable del mismo día: la plantilla común con la orden del área de quien la emite, varias áreas por persona, «Asignar área», la sesión que conserva las asignaciones con área, el historial del equipo, la vinculación completa de lo instalado, el escaneo por modelo y los demás hallazgos A-H14 a A-H19 y C-H18 a C-H32; corrige los hechos que la auditoría desmintió; el orden pasa a menú → GPS → áreas, con V148 a V166 |
| 3 | 22/11/2026 | El primer tramo del responsable: fase 0 reducida, fase 2 y fase 3a (§7) |
