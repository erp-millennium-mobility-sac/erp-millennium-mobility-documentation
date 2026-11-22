# Auditoría del plan — áreas, menú y ciclo de compra del GPS — ERP Millennium Mobility

**Estado:** para el responsable, 22/11/2026. Audita la versión 1 del [plan de implementación](plan-de-areas-menu-y-ciclo-del-gps-2026-11-22.md) y su [análisis situacional](analisis-situacional-areas-menu-y-ciclo-del-gps-2026-11-22.md), los dos del mismo día. **Cerrada el mismo día:** el responsable aprobó las recomendaciones (§7 y Q1 a Q27), y la versión 2 del plan y del análisis recoge cada hallazgo. Las líneas «Plan Lnnn», las fases, las migraciones y las decisiones que cita esta auditoría son las de la versión 1; la tabla de equivalencias está en el §12 del plan.

**Fuente:** el código de `develop` del backend (`ed7ccda7`) y del frontend (`ee7736af`), la documentación de `master` (`e71fdb4`), el diccionario 1.79, las skills locales, y el correo al cliente del 16/11/2026. Cinco revisiones de solo lectura: cuatro en paralelo (núcleo de las áreas; compras por área; ciclo del GPS en el backend; frontend y menú) y una propia (inserción por lote, órdenes y plantillas, escaneo, alcance prometido). Ninguna corrió pruebas ni tocó una base.

**Cómo leerla.** Las rutas van abreviadas como en el plan (`B/`, `M/`, `T/`, `F/`, `E/`). La columna **Comprobado** dice si el hallazgo se verificó **a mano** contra el código en esta auditoría o si viene de una **revisión** con su evidencia citada, sin volver a abrirla. «Plan Lnnn» es la línea del plan.

---

## 1. Veredicto

**El plan parte de evidencia real.** Casi todas sus citas son exactas: las líneas, los conteos (`PermissionGrant` 107 usos en 21 archivos de prueba, `RoleScope` 33, `Employee` 44, los comandos del colaborador 39, `RecordChangeLogCommand` 37 en 19 archivos, `permission_incompatibilities` 28), la definición de `user_roles_active_scope_uq`, la última definición de cada disparador que reescribe, los conteos de sentencias (64 al registrar una factura), las dependencias entre módulos, las rutas del menú, la próxima migración libre (V148), la última decisión (D-282), el diccionario 1.79 y `DESIGN.md` en la revisión 98. Los tres frentes caben sobre lo construido, como dice el análisis.

**No está listo para aprobarse.** Tiene **6 vacíos que bloquean** una fase o rompen algo que hoy funciona, **22 importantes** y unos cuarenta menores. Unas pocas afirmaciones no tienen respaldo (§5). Ninguno obliga a rehacer el diseño: casi todos se cierran con una precisión, una decisión del responsable o una pieza que falta.

| Severidad | Cuántos | Dónde se concentran |
|---|---|---|
| Bloqueante | 6 | La sesión (fase 2), las plantillas con órdenes (fase 3), la vinculación de lo instalado (fase 6), el escaneo con varios modelos (fase 5), `isConcept()` (fase 4), el CI del frontend (fase 0) |
| Importante | 22 | El historial prometido al cliente, la factura de 500, «Anular y copiar», la máquina de estados, lecturas de compras sin área, la plantilla común, la cámara, la ficha del equipo |
| Menor | ~40 | Líneas desplazadas, clases sin nombrar, migraciones sin `lock_timeout`, textos |

---

## 2. Bloqueantes

### BL-1 · La sesión de la fase 2 quita al limitado a un área los botones del inventario de equipos

| | |
|---|---|
| **Plan** | L376: «`wholeCompanyPermissions` y `brandPermissions` siguen, y desde esta fase excluyen las asignaciones con área» |
| **Evidencia** | `F/iam/application/iam.store.ts` L400-424: `actsOnEveryBrand` y `actsOnWholeCompany` leen solo esas listas, sin volver a `permissions` cuando existen. `F/equipment/…/equipment-inventory-view.ts` L94, L156, L165 y `equipment-form-dialog.ts` L393 deciden Registrar, Editar y Cambiar estado con `actsOnEveryBrand`; `F/organization/…/company-bank-accounts.ts` L83 con `actsOnWholeCompany`. El backend sigue sin mirar el área: `B/equipment/…/EquipmentCommandServiceImpl.java` L266 (`actsOnWholeCompanyEveryBrand`) |
| **Consecuencia** | Una persona limitada a Monitoreo pierde «Registrar equipo», justo el caso que A-H11 y Q10 quieren proteger. El análisis (A-H5) pide partir en dos «lo que cuenta para otorgar» y «lo que dice la sesión»; el plan hace que las dos excluyan el área, así que la partición no separa nada. Tampoco nombra `User.permissionCodesActingOnBrandsAt` (L401-411), que alimenta `brandPermissions` |
| **Corrección** | La sesión conserva las asignaciones con área en `wholeCompanyPermissions` y `brandPermissions`, que reflejan los controles del backend que no miran el área; solo `RoleDefinitionPolicy.heldFor` (regla 5.5) las excluye; compras pregunta por `assignments`. Pruebas de regresión: los botones del inventario y de las cuentas bancarias con una asignación limitada a un área |
| **Comprobado** | A mano (y por dos revisiones por separado) |

### BL-2 · Las plantillas que ya tienen órdenes no pueden recibir área

| | |
|---|---|
| **Plan** | L669: antes de otorgar la primera asignación con área, «el superadministrador asigna su área a cada plantilla existente». Pero L536 y L600 rechazan cambiar el área de una plantilla con órdenes (`PURCHASEORDERTEMPLATE_AREA_IN_USE`), y L481 impide que cambie el área de una orden |
| **Evidencia** | `B/procurement/…/PurchaseOrderTemplateCommandServiceImpl.java` L231-255: la marca no cambia con órdenes (`brandInUse`), y el plan copia esa regla. `dev` tiene las 2 plantillas reales y 9 órdenes sembradas el 09/11, todas de esas plantillas (no se comprobó cómo se reparten) |
| **Consecuencia** | El paso de operación del que depende la fase 3 es imposible en `dev`, y lo será en producción con cualquier plantilla usada: esas plantillas y sus órdenes quedarían generales, visibles para todas las áreas |
| **Decisiones del responsable (22/11/2026)** | (1) Lo que ya existe también se separa por área: «tiene que haber». (2) La operación única para darle área a lo existente, confirmada, también para las facturas sin orden. (3) Un mismo formato (plantilla) puede servir a todas las áreas, y un proveedor lo usan varias áreas. Q23 queda resuelta |
| **Corrección** | **El área de la plantilla y el de la orden se separan** (IM-22): la plantilla dice quién la usa y la orden es del área de quien la emite. Con eso, el área de una plantilla cambia cuando se quiera, porque solo vale para las órdenes futuras, y sale `PURCHASEORDERTEMPLATE_AREA_IN_USE`. Las plantillas de hoy pueden quedar generales, como formatos comunes. **Para lo ya emitido, una operación única «Asignar área»**, del superadministrador, antes de la primera asignación con área, sobre **una orden general** o sobre **una factura general sin orden**: (1) solo de general a un área, nunca de un área a otra ni de vuelta; (2) en la misma transacción, el área baja de la orden a sus facturas, a las notas de esas facturas y a los pagos de esas facturas y de la orden, en ese orden, para que los disparadores de coherencia de V152 pasen; (3) si una nota o un pago junta esos documentos con otros de otra orden o de otra área, se rechaza y nombra cada documento (Q7); (4) los disparadores de V152 (`enforce_purchase_order_area`, `keep_purchase_invoice_fiscal_data`, `keep_credit_note_data`, `keep_supplier_payment_data`) admiten solo el paso de `area_id` vacío a un valor, una vez; (5) una fila de la bitácora por documento que cambia, y las filas viejas quedan generales (la cuenta de la aplicación solo inserta en `change_logs`); (6) su permiso, su error, su prueba de integración y su prueba de los disparadores. Abre además la puerta a adelantar el GPS (§6) |
| **Comprobado** | A mano |

### BL-3 · La vinculación de los GPS ya instalados no puede registrar el equipo, y deja fuera la carga inicial

| | |
|---|---|
| **Plan** | L1162-1166: `linkLegacyInstalledDevice(empresa, tipo, serial, unidad)` devuelve el equipo con ese serial «o lo registra Instalado sin proveedor ni factura»; corre al arrancar; las instalaciones sin serial se completan con el diálogo que ya existe |
| **Evidencia** | `M/V132` L22 `equipment_model_id NOT NULL`, L19 `location_id NOT NULL` (y el `CHECK` de V155 la sigue exigiendo para INSTALLED), L35 `created_by NOT NULL`; L55-65 serial, IMEI e ICCID únicos en **todo el sistema**, no por empresa. `vehicle_equipments` solo guarda tipo y serial, no modelo. La carga inicial, que corre en la salida a producción, crea instalaciones de GPS **sin serial y sin equipo** (`B/fleet/…/MigratedUnitRegistrations.java` L68, L342-348; D-206). Completar el serial después (`PUT /vehicle-equipments/{id}/serial`, `VehicleEquipmentCommandServiceImpl` L200-223) solo escribe el serial |
| **Consecuencia** | El proceso falla con los `NOT NULL`, o enlaza un equipo de otra empresa, de otro tipo o de baja. Las instalaciones de la carga inicial y las que se completan después del arranque quedan sin equipo y escapan de D-264 |
| **Corrección** | (1) Pasar un modelo (uno «heredado» sembrado por tipo, o el que elija quien completa), una regla de ubicación y la cuenta `SYSTEM`. (2) Definir el resultado de cada caso: el serial existe y está Disponible en la misma empresa y tipo → se instala; existe en otra empresa, otro tipo u otro estado → se informa y no se enlaza. (3) Completar el serial enlaza en la misma transacción. (4) La carga inicial crea o enlaza el equipo por la fachada de `equipment`, o la vinculación se ofrece como proceso que se corre después de ella, no solo al arrancar |
| **Comprobado** | A mano (esquema y carga inicial); revisión (serial completado) |

### BL-4 · El escaneo asigna el IMEI al «siguiente» equipo sin mirar su modelo

| | |
|---|---|
| **Plan** | L1068: cada lectura «lo envía para el siguiente equipo por ingresar de la factura». C3 admite varios modelos en una factura («50 × modelo») |
| **Evidencia** | El texto del plan; el costo del equipo sale del monto de su línea (L827), y la línea guarda su modelo (L729) |
| **Consecuencia** | En una factura con dos modelos, un IMEI del modelo B cae en el lugar de uno del A: el equipo hereda el precio y el modelo equivocados. Con el fabricante, termina en «Revisar» y `CHANGE_MODEL` corrige el equipo pero no la línea; con el proveedor `manual`, nadie lo nota |
| **Corrección** | Si la factura tiene más de un modelo, la pantalla pide el modelo (la caja) antes de escanear, y cada lectura va al siguiente pendiente **de ese modelo**. Con un solo modelo no cambia nada. Dos personas escaneando la misma factura chocan en el mismo equipo: la segunda recibe `EQUIPMENT_STATUS_TRANSITION_INVALID` y la pantalla pasa al siguiente |
| **Comprobado** | A mano |

### BL-5 · `isConcept()` no puede pasar a ser «tipo = concepto»

| | |
|---|---|
| **Plan** | L787: «`isConcept()` deja de ser "sin vehículo" y los nueve archivos que lo usan pasan a preguntar el tipo» |
| **Evidencia** | 13 usos en 9 archivos, y en todos `!isConcept()` significa «línea de vehículo». `B/procurement/domain/model/aggregates/PurchaseInvoice.java` L140 agrega `getVehicleId()` a un `Set` para detectar unidades repetidas: con dos líneas de equipo (vehículo nulo) rechazaría la factura. También `InvoiceUnitCoverages` L97, L112 (`requireNonNull(getVehicleId())`), `PurchaseInvoice` L315, `SupplierPaymentApplications` L330, `CreditNoteLineEffects` L71, `SupplierPaymentAllocations` L220, `CreditNoteAnnulments` L118, L132 y `CreditNoteViewFactory` L228. Al revés, `CreditedLines` L85 (anular la factura entera) sí tiene que contar las líneas de equipo |
| **Corrección** | `isVehicle()` e `isEquipment()` junto a `isConcept()`, y una decisión explícita por cada uno de los 13 usos, con su prueba. El SQL que filtra `vehicle_id IS NOT NULL` ya es correcto y no cambia |
| **Comprobado** | A mano |

### BL-6 · El CI del frontend está en rojo, y por eso el e2e no corre en `develop` desde el 20/11

| | |
|---|---|
| **Plan** | Fase 0, paso 1 (L138): agregar `stock.anular_lectura_kilometraje` al catálogo simulado y mirar `roles.spec.ts` |
| **Evidencia** | El permiso está en `F/iam/application/permission-codes.ts` L103 y falta en `E/support/permission-catalog.ts`. Está en rojo desde `3f47a838` (20/11, D-267), integrado en `f614aa34`. En `.github/workflows/ci.yml` el e2e tiene `needs: build`, y `build` corre `ng test` (L40-41): con la unitaria en rojo, el e2e no corre. Cinco integraciones del frontend llegaron después |
| **Consecuencia** | Las fases 2, 4, 5 y 7 del plan de migración y los arreglos del 22/11 se integraron sin e2e. Si la fase 1 regenera ~120 imágenes encima, se traga cualquier regresión anterior. El arreglo cambia más de lo que dice el plan: `E/roles.spec.ts` L160 («39 de 39» → 40) y L184, `E/permission-matrix.spec.ts` L316 («22/39» → «22/40»), y nueve imágenes (la matriz y la ficha del rol). La matriz simulada no se edita: SUPERADMIN y ADMIN_EMPRESA lo reciben solos |
| **Corrección** | La fase 0 corre el e2e completo en su rama, arregla lo que encuentre y regenera esas nueve imágenes en un commit propio, **antes** de la fase 1. Antes de eso, conviene que el responsable mire la pestaña Actions del frontend: el plan dice rojo y las integraciones de esos días se cerraron con «CI en verde» |
| **Comprobado** | A mano (catálogo, spec y `ci.yml`); revisión (commits y conteos) |

---

## 3. Importantes

### 3.1 Alcance y diseño del GPS

| # | Hallazgo | Evidencia | Corrección | Comprobado |
|---|---|---|---|---|
| IM-1 | **El historial prometido al cliente no está en el plan.** El correo del 16/11 promete «el historial de en qué vehículo estuvo cada equipo y cuándo se retiró o reemplazó». El plan no tiene ruta ni pantalla para eso. `equipment` no guarda historia: no escribe bitácora y `status_reason` (V134) se sobrescribe en cada cambio, así que el motivo de «Devolver al proveedor», de «Revisar» o de la anulación se pierde (regla 4.3). `equipment` no puede leer las instalaciones: `fleet` depende de `equipment`, no al revés | `correos/2026-11-16-avance-del-proyecto.md` L52-55; `B/equipment` sin `ChangeLog`; `M/V134` L8; `package-info` de `equipment` | (1) `fleet` sirve `GET /api/v1/equipments/{id}/installations`, como `procurement` sirve el costo. (2) Una tabla de solo inserción `equipment_status_history` (desde, hasta, motivo, quién, cuándo), como `vehicle_status_history`, que reciben todos los caminos nuevos. (3) La ficha del equipo muestra las dos | A mano |
| IM-2 | **La factura de 500 no se inserta «por lote» con la configuración de hoy, y la meta de tiempo no tiene respaldo.** No hay `hibernate.jdbc.batch_size`, y todo id es `IDENTITY`, que Hibernate nunca agrupa. `bump_data_version` corre por sentencia, así que 500 INSERT son 500 actualizaciones de `data_versions`. El plan cita «PRC-R65 pide 100 líneas en 2 segundos»: PRC-R65 solo exige que las consultas que no son inserciones no crezcan (64 hoy) | `application.yaml` L15-27; `CreatedAbstractPersistenceEntity` L31; `M/V94` L276-281; `contextos/procurement.md` L300; `PurchaseInvoiceRegistrationStatementsTests` L42 | Elegir el mecanismo: un INSERT de varias filas (`unnest … RETURNING id`, o JDBC con `reWriteBatchedInserts`) para equipos y líneas. Medir contra `dev`, por la latencia real a la base, y fijar la meta con ese número. Recordar el corte de 100 s de Cloudflare | A mano |
| IM-3 | **«Anular y copiar» duplica los equipos.** Corregir una factura es anularla y copiarla (D-211, D-214, PRC-R66), o emitir una nota «anulación de la factura». `registerPurchasedDevices` siempre crea equipos nuevos: la copia duplica los por ingresar, y los ya recibidos nunca vuelven a tener línea vigente. Q15 cubre la anulación de la factura, no esa nota. `annulPurchasedDevices` mezcla dos cosas: la nota anula Disponibles y devueltos; la anulación de la factura solo los por ingresar | `contextos/procurement.md` L146, L197; plan L772, L811 | El alta vincula equipos de la misma empresa, proveedor y modelo que no tienen línea vigente ni están anulados (la unidad lo hace por VIN, D-161); Q15 vale también para la nota «anulación de la factura»; la fachada distingue «solo los por ingresar» | Revisión; flujos a mano |
| IM-4 | **La máquina de estados tiene huecos.** `annul()` no admite MAINTENANCE, y la fase 5 no suma PENDING_COMPLETION ni UNDER_REVIEW (una nota por un equipo con `UNKNOWN_IMEI` fallaría). No dice qué hace la nota con un equipo «no anulable». `comeBackRepaired` de un equipo ya reemplazado deja dos equipos vivos en una línea y el costo contado dos veces. Una nota sobre el original reemplazado anula el que está en el proveedor y deja al reemplazo con una raíz de valor cero | Plan L752-758, L995-1001, L1046 | La nota rechaza la operación entera y nombra los equipos; `comeBackRepaired` se rechaza si otro equipo lo reemplazó; «anulación de equipo» apunta al extremo vivo de la cadena; la tabla de caminos cubre los siete estados | A mano (texto del plan) |
| IM-5 | **¿Una orden puede mezclar vehículos y equipos?** El plan decide que una factura es de un solo tipo (C2), no la orden. La plantilla da un área (y una marca) a la orden; una orden mixta mezclaría las compras de dos áreas. Si es mixta, Q14 choca: la orden la escribe quien tiene la marca de la plantilla, y la factura de equipos pide una asignación sin marca. `PurchaseInvoiceOrderLinks.deviations` (L71-83) nombra modelos de vehículo, y `OrderInvoicingPolicy.Deviation` tiene `int year` (L56) | V69 L19 (toda orden tiene plantilla); V68 (la plantilla no tiene líneas) | Una decisión nueva (Q24), con la recomendación «una orden es de un solo tipo»; la factura vinculada tiene el tipo de su orden | A mano (esquema); revisión (desviaciones) |
| IM-6 | **El aviso de antigüedad de los «Por ingresar» se perdió.** Decidido el 22/11 junto a C7: los que faltan quedan por ingresar «con un aviso de antigüedad» antes de la nota. El plan no lo tiene | §1 del plan (C7); notas de la sesión del 22/11 | Sumarlo a C7 y a la fase 4: los días desde la factura en la lista y en la ficha, y un umbral configurable | A mano |
| IM-7 | **Con el proveedor `manual`, 500 equipos se completan uno por uno.** Hasta que exista el adaptador (Q21), cada equipo escaneado queda «Por completar» y pide serial y firmware a mano para estar Disponible (EQP-R14) | Plan L1031, L1004 | Decidir con el responsable: completado en lote por modelo, el serial desde el QR si lo trae, o mantener el ingreso completo de la fase 4 como camino normal mientras no haya fabricante | A mano |
| IM-8 | **`equipment` no puede firmar como `SYSTEM`.** La tarea del fabricante escribe `updated_by` y crea SIM (`created_by NOT NULL`); V159 «firmada por `SYSTEM`» leería `users`, que es de `iam`. `equipment` no depende de `iam::acl`, y el id de la cuenta `SYSTEM` solo lo da `IamContextFacade.fetchSystemAccountId()` | `package-info` de `equipment`; `M/V132` L35; V145 declara un paso de `iam` | Sumar `iam::acl` a `equipment`, o que el llamador pase el id; V159 declara su paso de `iam` | A mano (dependencias y esquema) |
| IM-9 | **La retención de D-264 al pasar a Inactivo está en la clase equivocada.** `VehicleTransfers` solo registra los datos del notario; el paso a Inactivo lo hace `VehicleMovements` (L131, L150, con `TransferExitPolicy`), lo ofrece `VehicleAllowedActions` L124, y también el camino de D-265 | Plan L1160 | Poner el control en `VehicleMovements` y en las acciones permitidas, y probar también «Devolución al proveedor» | A mano |
| IM-10 | **Los resultados viejos del fabricante.** Un completado a mano o una revisión puede llegar con un pedido en curso; un ICCID que ya existe en otra empresa hace fallar «buscar o crear la SIM» por el índice único global | Plan L1034-1035 | Los caminos a mano cancelan el pedido abierto; aplicar un resultado vuelve a mirar el equipo y el pedido bajo candado; un solo orden de candados (equipo → pedido, equipo → SIM); el ICCID ajeno es `SIM_MISMATCH` | Revisión |
| IM-11 | **V159 puede abortar o crear datos falsos.** `imsi` es texto libre (`VARCHAR(30)`): un `CHECK` de 15 dígitos rompe la migración. «No está de baja» incluye los ANULADOS de V155, que recibirían una SIM activa. Un equipo de baja creado y cambiado el mismo día da un rango vacío `[d,d)` | `M/V132` L31; plan L966-969 | Vaciar con un `NOTICE` los IMSI inválidos; tratar ANNULLED como DISCARDED; usar `[alta, greatest(cambio, alta + 1))` | A mano (columna); revisión |
| IM-12 | **El ICCID después de V159 queda sin definir.** `POST /equipments`, `PATCH` y `/arrival` siguen aceptando `iccid`; EQP-R14 lo exige al crear; la búsqueda, `existsByIccid` y el mapeo de la restricción siguen | `EquipmentCommandServiceImpl` L364-369; `EquipmentJpaRepository` L61-66, L78; `EquipmentRepositoryImpl` L35-37 | Decidir: crea y asigna la SIM, se rechaza o se ignora; la fase 7 retira lo que quede | Revisión |
| IM-13 | **La marca vacía deja pasar más que el registro.** Q14 pide una asignación sin marca para registrar una factura de equipos. Pero anular, modificar, la nota y el pago también pasan cuando la factura no tiene marcas | `CreditNoteAccessPolicy.writes` L33-36; `PurchaseInvoiceAccessPolicy` L25-44 | Q14 vale para toda escritura sobre un documento de equipos | Revisión |

### 3.2 Áreas en compras

| # | Hallazgo | Evidencia | Corrección | Comprobado |
|---|---|---|---|---|
| IM-14 | **`GET /suppliers/{id}/payables` no está en el plan**, y es la fuente del formulario de pago. Devuelve las facturas sin filtrar, las órdenes filtradas solo por marca y los orígenes del saldo sin filtrar | `SupplierPaymentQueryServiceImpl` L225, L227, L235-238; `F/…/procurement-api.ts` L394 | Filtrar facturas y órdenes por área; `area` en `PayableInvoiceView` y `PayableOrderView`; los filtros `areaId` y `generalOnly`; los orígenes del saldo solo viajan a una asignación sin área (Q9), porque el origen «sin nota» no tiene área | A mano |
| IM-15 | **Otras lecturas que muestran datos de otra área.** La verificación del VIN devuelve número, proveedor e id de la factura vigente de otra área (`PurchaseInvoiceVinChecks` L134-139); la ficha de un pago nombra notas de otras áreas que consumió el saldo (`SupplierPaymentViewFactory` L329-345); la factura muestra `appliedTotal`/`owed`/`toBalance` con `pagos.ver` de la empresa (`PurchaseInvoiceViewFactory` L196) y ofrece «Registrar pago» igual (`PurchaseInvoiceAllowedActions` L80-81); `SupplierBalanceView.owed` es un total de todas las áreas; Por pagar arma su alcance en `PayableQueryServiceImpl` L92-95 y su acción `mayPay` en L156, no en `PayablesCriteria` | Las citadas | Cada una lee el área en la misma asignación; lo de otra área viaja solo como monto, sin referencia | Revisión |
| IM-16 | **El control de escritura con área tiene que ir en la misma asignación que la marca.** Las marcas de registrar y de la factura atrasada se preguntan en `PurchaseInvoiceUnits` L108 (`registers`) y L114 (`registersOld`), no en L135/L168 como dice el plan. Ponerlo aparte repite en la escritura el error A-H1: «marca X sin área» más «sin marca, área P» registrarían una factura de X en P | Plan L549 | `registers` y `registersOld` preguntan con `anyGrantOf` marca y área a la vez | Revisión |
| IM-17 | **El SQL del filtro, escrito literal, falla en PostgreSQL.** `(:a IS NULL OR area_id IS NULL OR area_id = :a)` con un nulo da «could not determine data type of parameter» en `NamedParameterJdbcTemplate`, y `openInvoices` arma SQL nativo. El código ya lo esquiva con `CAST(:x AS BIGINT) IS NULL` (`PurchaseInvoiceJpaRepository` L58) | Plan L513 | Por cada entrada de `RecordReach`: sin condición de área si la asignación no la tiene; si la tiene, `area_id IS NULL OR area_id = :aN`. Nunca un nulo enlazado | Revisión |
| IM-22 | **Una plantilla para todas las áreas existe, pero sus órdenes nacen generales.** El plan admite la plantilla general (A4), y la orden copia el área de su plantilla: V152 `enforce_purchase_order_area` exige «al insertar, el área es la de la plantilla» y crear la orden «copia el área de la plantilla». Además, una orden sin área solo la escribe una asignación sin área (A-H2). Así, una plantilla común no sirve a un comprador limitado a Monitoreo, y lo que emita alguien sin área lo ven todas las áreas. Contradice la decisión del responsable (un mismo formato para todas las áreas) | Plan L481, L541; análisis A-H2 | El área de la plantilla dice quién la usa (general = todas las áreas); el área de la orden es la de quien la emite: con plantilla de un área, esa área; con plantilla general, el área de quien la emite si está limitado a una, a elegir entre las suyas si tiene varias, o un área o «General» si no está limitado (Q27). `enforce_purchase_order_area` exige la igualdad solo cuando la plantilla tiene área. La orden guarda su área desde que nace, y el formulario suma el selector en el primer panel (plan L622). Proveedores y tipos de orden siguen compartidos (A3); quién los escribe es Q8 | A mano (texto del plan) |
| IM-18 | **La exportación de notas no tiene constante de diseño.** `ProcurementDocumentKinds` L39 es de pagos, L45 de facturas y L51 de Por pagar; las notas no tienen. Al sumar la columna «Área», una exportación generada antes del despliegue se sirve con las columnas viejas | Plan L595 | `CREDIT_NOTES_LAYOUT = "credit-notes-layout=1"` | Revisión |

### 3.3 Frontend y menú

| # | Hallazgo | Evidencia | Corrección | Comprobado |
|---|---|---|---|---|
| IM-19 | **La cámara está bloqueada en el sitio desplegado.** `public/_headers` envía `Permissions-Policy: camera=()` a todo `/*`; en una aplicación de una sola página no sirve un encabezado por ruta. El e2e no lo ve, porque el servidor de desarrollo no aplica `_headers` | `public/_headers` L10 | La fase 5 cambia a `camera=(self)` | A mano |
| IM-20 | **La ficha del equipo es un diálogo.** Abrir desde ella «Asignar SIM», «Registrar ingreso» o «Completar» pone un diálogo de formulario sobre otro, que prohíbe la regla 4.9 | `equipment-inventory-view.ts` L258 (`EquipmentDetailsDialog`); `frontend-rules.md` §4.9 | Una ruta propia para la ficha del equipo y de la SIM (`/equipment/inventory/:id`, `/equipment/sim-cards/:id`), o las acciones en las filas | A mano |
| IM-21 | **La fase 1 rompe más pruebas de las que nombra, y Q3 es una decisión del diccionario.** Fallarán los e2e de catálogos con roles de solo lectura (`E/exchange-rates.spec.ts` L410-413, `E/vehicle-document-types.spec.ts` L26 y L88-90, `E/vehicle-equipment-requirements.spec.ts` L36-41) y más casos de `app-shell.spec.ts` (L91-104, L149-173, L195-215). El e2e «cada uno de los siete roles» no tiene sesiones de OPERACIONES, CONTABILIDAD ni TALLER (`E/support/api.ts` L92-103). El panel «de más filtros» de L634 no existe: el gestor de datos pliega sus filtros solo en el teléfono, así que un filtro de área rompe la barra de una línea de Facturas y Notas. Que los catálogos se abran solo por escritura cambia D-141, §5.6 y DP-18: después, solo el superadministrador abre un catálogo, también por URL (Contabilidad pierde Tipos de cambio) | Las citadas; `data-manager.html` L4-26 | Nombrar esas pruebas; afirmar los siete roles en `app-shell.spec.ts`, que ya lee `SEEDED_MATRIX`; un patrón y una revisión de `DESIGN.md` para el filtro de área; enmendar D-141 y decir explícitamente a quién deja sin qué al decidir Q3 | Revisión |

---

## 4. Menores

Se corrigen al escribir la versión 2 del plan; ninguno cambia una fase.

**Fase 0 y 1**
- `organization-access-scope-options.spec.ts` es de la fase 3, no de la 1 (L245); `app-shell.spec` L451-478 no cambian con Q3, solo la llave del grupo en L448.
- La matriz: los permisos de las cargas (`stock.cargar_migracion_inicial`, `fondos.cargar_archivo_garantia`) siguen en Cargas de archivos (`FILE_LOAD_PERMISSIONS`, L62-67), no en Inventario ni Fondos (L222-237). El renombre olvida `permission-matrix-rows.ts` L13 y L108-140, `permission-matrix.html` L98-102, `role-access-editor.html` L45-76, `permission-matrix.store.ts` L123-125, `PERMISSION_MODULE_ORDER` (`permission-codes.ts` L313-329), `es.json` L339 («Logística») y `DESIGN.md` L416.
- La regla «Catálogos con `catalogos.ver`» está en el SKILL.md de `frontend-feature` §6 L205, no en `frontend-rules`. Seguridad también tiene etiqueta ADMIN (`app-navigation.ts` L390).
- El comando de las imágenes (§4) tiene que nombrar también `e2e/permission-matrix.spec.ts`.

**Fase 2**
- `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` es solo para roles reservados: un rol de alcance sistema con área responde `ROLE_ASSIGNMENT_SCOPE_MISMATCH`, porque `scopeMismatch` corre antes (L139 contra L143), como hoy la marca (`RoleAssignmentsIntegrationTests` L390-396).
- `commercial_brands` solo tiene `CHECK` de no vacío (V16 L20-21); el formato `^[A-Z0-9]+$` de 2 a 20 vive en el dominio. V20 y V39 no lo tocan. Decir si los `CHECK` de `areas` son nuevos.
- Modificar un colaborador bloquea colaborador → sede (→ área), sin la empresa (`EmployeeCommandServiceImpl` L223-227). «Sin índice» tiene su precedente en `employees.branch_id` (V22), no en V42.
- Clases sin nombrar: `ExternalOrganizationService.resolveScope` de `iam` (L187-190, que con el constructor de tres argumentos perdería el área sin error) y un `fetchAreaReferences`; `RoleAssignmentViewFactory`, `UserQueryServiceImpl` L134-163, `AuthenticatedUserResourceFromEntityAssembler`, `RoleViewFactory`, los ensambladores de rol, de usuario y de asignación.
- `keep_area_company()` lleva su línea en `hand-over-database.sql` ya en la fase 2.
- Decir que desactivar un área deja vigentes sus asignaciones (como la marca, D-84) y que el regreso del colaborador no mira el área (una excepción escrita a D-84).
- V150 y V151 abren con alcance `SYSTEM` y `lock_timeout` de 5 s: V150 recrea un índice de `user_roles`, que lee cada pedido.
- Decir si en la fase 2 la API rechaza `areaId` hasta la fase 3 o se acepta sin efecto.
- La lista `assignments` de la sesión: si cada entrada quita los once códigos de accesos, si las asignaciones de sistema aparecen bajo cada empresa, y que las sedes viajan con su identificador público.
- La fachada: `shareAreaByPublicId` rompe la forma `fetch…ForShare`, y falta la lectura simple por identificador público que compras usará en los filtros.
- La etiqueta de la exportación de cuentas nombra hoy solo la empresa: sumar el área sola es incoherente.
- `IamFacade` (L312-334) necesita los ayudantes nuevos, y `organization.facade.spec.ts` L62-76 fija sus miembros. Cada ruta nueva va en `environment.ts` y en `environment.development.ts`. Los nombres cortos de las pestañas ya existen (revisión 53): solo falta «Áreas».

**Fase 3**
- La regla de lectura necesita dos métodos: el de la empresa («alguna asignación en ella», para lo que todavía no tiene registro: L306, L179, `PayableQueryServiceImpl` L99, L131, `CreditNoteReferences` L58, `SupplierPaymentReferences` L130, `PurchaseInvoiceOrderLinks` L53) y el del registro (empresa y área).
- Vincular una orden de un área que no se lee responde 404, no `PURCHASEINVOICE_AREA_MISMATCH`, que revela que existe.
- V153 abre con `SYSTEM` y `lock_timeout`, y crea el `CHECK` y la llave como `NOT VALID` y luego `VALIDATE`.
- Decir si un área inactiva bloquea una factura con orden, una nota o un pago.
- Las filas viejas de la bitácora de una plantilla quedan generales para siempre (la cuenta de la aplicación solo inserta): escribirlo.
- El reporte de brecha de días sigue sin filtrar, con el criterio de A-H10: escribirlo.
- `ExternalOrganizationService` L88 es `checkIssuingScope`; las referencias de marca en lote están en L95. El alcance de la lista de plantillas se arma en `PurchaseOrderTemplateQueryServiceImpl` L147-168.

**Fases 4 a 7**
- La premisa de V154 es falsa: `keep_purchase_invoice_item_fiscal_data` compara una lista cerrada de columnas (V103 L215-221), así que el relleno de `line_kind` pasa sin desactivarlo. Basta con rellenar antes del `CREATE OR REPLACE` que la suma.
- Las imputaciones no hacen falta en ningún lado: la base solo revisa una imputación al insertarla (V107 L207-225), `Spreads.of` y `OrderPaymentTransfers` pasan sin unidades, y `AmountOwedPolicy.toBalance` (L112-117) ya pasa el exceso a saldo por factura. La «política nueva de cobertura» (L820) sobra; lo único que estorba es `allocationRequired` del canal concesionario (`SupplierPaymentAllocations` L77-81), que exime las facturas de equipos. Ninguna migración de pagos.
- `DeadlockRetryTests` no cubre las tareas nuevas «por su paquete»: las lista exactas (L123-126) y hay que sumarlas.
- Retirar, reemplazar y `lockedWithItsUnit` están en `VehicleEquipmentCommandServiceImpl`; `Propagation.MANDATORY` está en `FleetContextFacadeImpl` L74, no en `PurchasedUnitRegistrations`.
- EQP-R13 hoy no exige proveedor (`EquipmentCommandServiceImpl` L96-104; la columna es nula).
- El planificador tiene un solo hilo (`SchedulingConfiguration` sin `pool.size`): una llamada al fabricante de 5 s + 10 s atrasa los correos y los vencimientos. Fijar el tamaño o acotar cada corrida.
- La nota de 500: `CreditNote.MAX_LINES` no conoce el tipo de la factura, y `RegisterCreditNoteResource` L31 tiene `@Size(100)`. El sexto lugar del límite de 100 es `PurchaseInvoiceVinChecks.MAX_VINS` L42.
- V155 conserva los nombres `idx_fleet_equipments_serial_unique` e `idx_fleet_equipments_imei_unique`, que `EquipmentRepositoryImpl` L35-37 traduce a errores; el del ICCID sigue excluyendo solo DISCARDED.
- La ubicación opcional también toca `EquipmentPersistenceEntity` L33 (`nullable = false`), `EquipmentResource` L23 y `EquipmentViewFactory` L67.
- El dígito verificador del IMEI vuelve aleatorias las pruebas que generan `digits(15)` (`EquipmentAuditTests` L57-63 y cinco más), y `updateIdentifiers` (L141-147) revalidaría un IMEI que no cambió.
- `replaced_equipment_id` va como llave compuesta `(id, company_id)`; `CHANGE_MODEL` queda dentro del tipo (`equipment_type_id` es final y V163 lo usa en su llave); devolver a Disponible un GPS heredado sin IMEI choca con EQP-R14 reescrita.
- `pendingArrival=true` cruza contextos: filtrar antes de paginar, y decir cómo.
- La SIM usa `PUT` y el equipo `PATCH /{id}` (`EquipmentsController` L107): elegir una forma. Las variables nuevas van en `.env.example` y `.do/app.yaml`. `sim_card_assignments` va en `COLUMN_UPDATE_TABLES` (`RowLevelSecurityMigrationTests` L90).
- Frontend: `EquipmentRemovalReason` (L5) suma `sin_falla`; la ubicación recordada vive en un store en memoria (la skill prohíbe el almacenamiento del navegador); la fase 5 ubica en la matriz `stock.ingresar_equipamiento` y `stock.*_sims`; los presupuestos de `angular.json` (L46-57) solo miden el paquete inicial, así que una librería perezosa no la vigila el build.

---

## 5. Afirmaciones sin respaldo

| Plan o análisis | Lo que dice | Lo que hay |
|---|---|---|
| Plan L870 | «PRC-R65 pide 100 líneas en 2 segundos» | PRC-R65 no fija tiempo (`contextos/procurement.md` L300); ningún documento dice «2 segundos» de una factura |
| Plan L283 | Los mismos `CHECK` que `commercial_brands` (V16, V20, V39): mayúsculas, sin espacios, de 2 a 20 | Solo no vacío (V16 L20-21) |
| Plan L709 | El disparador fiscal «rechaza toda edición de una línea», hay que desactivarlo | Compara una lista cerrada de columnas |
| Plan L1160 | `VehicleTransfers` no pasa a Inactivo | El paso a Inactivo está en `VehicleMovements` |
| Plan L1084 | `DeadlockRetryTests` cubre las tareas por su paquete | Las lista exactas |
| Plan L1076 | «El build vigila los presupuestos del paquete» | Solo el inicial |
| Plan L412, L634 | La revisión 100 crea los nombres cortos; un «panel de más filtros» | Existen desde la revisión 53; el panel no existe |
| Análisis §5.1 | EQP-R13: ubicación **y proveedor** al registrar | El proveedor es opcional |

---

## 6. El orden de las fases

El plan pone las áreas (fases 2 y 3, de 15 a 21 días) antes del GPS, para que las facturas de equipos nazcan con área, porque el área queda fija con lo fiscal. Es coherente, pero atrasa unas cuatro semanas el módulo de monitoreo, que el correo del 16/11 anunció como la próxima etapa. Con la operación «Asignar área» de BL-2, confirmada por el responsable también para las facturas sin orden, lo registrado antes de las áreas puede recibir su área después, así que las fases 4 a 6 podrían ir antes que la 3. Es una decisión del responsable (Q26).

---

## 7. Lo que necesita el responsable

| # | Pregunta nueva | Recomendación |
|---|---|---|
| Q23 | Las plantillas que ya tienen órdenes (BL-2) | **Resuelta el 22/11/2026:** lo existente también se separa por área, con la operación única «Asignar área» sobre órdenes y facturas sin orden; las plantillas pueden quedar comunes |
| Q27 | Con una plantilla general, ¿de qué área es la orden? (IM-22) | De quien la emite: su área si está limitado a una; a elegir entre las suyas si tiene varias; un área o «General» si no está limitado |
| Q8 | Quién crea, edita y desactiva proveedores, sus contactos y los tipos de orden (sigue abierta) | Solo una asignación sin área. Alternativa: quien tiene área crea, pero no edita ni desactiva |
| Q24 | ¿Una orden puede mezclar vehículos y equipos? (IM-5) | No: una orden es de un solo tipo |
| Q25 | Con el proveedor `manual`, ¿cómo se completan cientos de equipos? (IM-7) | El ingreso completo de la fase 4 sigue como camino normal hasta que exista el adaptador |
| Q26 | ¿Se adelanta el GPS a las áreas? (§6) | Depende de Q23 |
| — | Confirmar el estado del CI del frontend en Actions (BL-6) | Antes de la fase 0 |

Y precisar Q3 con lo que pierde cada rol (IM-21), Q14 para toda escritura (IM-13) y Q15 para la nota «anulación de la factura» (IM-3).

**Esfuerzo.** Lo que suman las correcciones —el historial, la vinculación completa, «Anular y copiar», la sesión, la inserción de varias filas, la ficha del equipo como página— se estima en 5 a 8 días sobre los 45 a 63 del plan. Es una estimación, no una medida.

---

## 8. Siguiente paso

Hecho el 22/11/2026: el responsable aprobó las recomendaciones —Q8 como «cualquier comprador crea proveedores y agrega contactos; editar y desactivar pide acceso a todas las áreas», Q26 como menú → GPS → áreas, Q27 como «la orden es del área de quien la emite», varias áreas por persona con un selector múltiple, y «con acceso a todas las áreas» en lugar de «sin área» para las personas— y la versión 2 del plan y del análisis las recoge. Lo único pendiente del responsable es mirar el CI del frontend en Actions (BL-6) y conseguir la información del fabricante (Q21).
