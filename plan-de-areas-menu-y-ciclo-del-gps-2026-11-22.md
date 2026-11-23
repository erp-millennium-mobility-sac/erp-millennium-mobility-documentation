# Plan de implementación — áreas, menú y ciclo de compra del GPS — ERP Millennium Mobility

**Estado:** versión 3.9, 23/11/2026. Recoge la [auditoría del mismo día](auditoria-del-plan-de-areas-menu-y-ciclo-del-gps-2026-11-22.md) y las decisiones que el responsable aprobó ese día (§1 y §2). La versión 2 cambió el orden de la versión 1 (el menú, el ciclo del GPS y al final las áreas); la versión 3 adelanta un **primer tramo** —la fase 0 reducida, la compra de equipos y su ingreso con el IMEI, con el reemplazo de un fallado— y deja para después la SIM, la API del fabricante y lo demás (Q28, Q29; §5). La 3.1 recoge Q30 y Q31: la SIM es un dato de su GPS, y la fase 3b queda solo con la API del fabricante (§12). La 3.3 cierra la fase 1, el menú por procesos. La 3.4 cierra la fase 4, la instalación unida al inventario —backend `e45c93af`, integrado en `develop` con `0dbd3695`; frontend `e36ea865`, integrado en `develop` con `2507b3ed`—, y abre Q33 a Q36. La 3.5 cierra la fase 6, áreas: catálogo, asignación y sesión —backend `1c7a2e51`, integrado en `develop` con `ad7a61c7`; frontend `dfe3ae92`, integrado en `develop` con `60b79b96`—, con V157 a V160 después de las de la fase 4. La 3.6 cierra las fases 5 y 7. La 5, la limpieza —backend `e17729ec`, integrado en `develop` con `f387c252`; frontend `5fb0e41a`, integrado en `develop` con `d3b92cb3`—, sin migraciones. La 7, compras por área y «Asignar área» —backend `43d6482b`, integrado en `develop` con `90b18bc6`; frontend `35a2e58d`, integrado en `develop` con `10869523`—, con V161 a V163 en lugar de V164 a V166, y abre Q37. Con las dos, el diccionario pasa a la 1.88. La 3.7 recoge Q33 y Q34, que el responsable respondió y que se construyeron el mismo día, sin migraciones (D-289, D-290) —backend `01dfee10` y `49b0963b`, integrados en `develop` con `b4eb60cc`; frontend `3fb495bf`, `893fbcfd` y `bbc5a817`, integrados en `develop` con `d416be6f`—, y abre Q38 a Q41; el diccionario pasa a la 1.89. La 3.8 recoge Q35 a Q41, que el responsable respondió el mismo día como se le recomendaron (D-291 a D-296); el diccionario pasa a la 1.90, y solo queda pendiente la información del fabricante (Q21). La 3.9 las cierra, construidas el mismo día, con V165 y V166 —V164 no se usó—: D-291 a D-294, backend `4a3754cc`, `3b6d2b80`, `21b078b1` y `1c7dcbc7`, integrados en `develop` con `3ed4dee9`, y frontend `f0bba89e`, `6bd3bf5e`, `1436166b` y `33d404d4`, integrados en `develop` con `ab045232`; D-295, backend `8fdd4112`, integrado en `develop` con `cfae200c`, y frontend `ad636530`, integrado con `d2c151e9`; y D-296, backend `83de71a2` y `5edc4166`, integrados con `cda2e9c1` y `f022e488`, y frontend `284b7b39`, integrado con `38bb8a59`; y el texto nuevo de `EQUIPMENT_MODEL_FROM_INVOICE`, que manda a «Anular y copiar», backend `3cccdf2c` y frontend `bb4364fd`. El diccionario pasa a la 1.91. Cada fase se abre comprobando otra vez las rutas y las líneas que cita.

**Fuente:** el [análisis situacional del 22/11/2026](analisis-situacional-areas-menu-y-ciclo-del-gps-2026-11-22.md), con el inventario de `develop` (backend `ed7ccda7`, frontend `ee7736af`) y de la documentación (`e71fdb4`); la auditoría del plan, del mismo día; las decisiones del responsable del 22/11/2026; el [diccionario de datos](diccionario-de-dtos-millennium-mobility.md) 1.79; las reglas de arquitectura de las skills `backend-module`, `frontend-feature` y `architecture-review`; y la forma de trabajo del [plan de migración a los contextos objetivo](plan-de-migracion-a-los-contextos-objetivo-2026-11-18.md), que este plan sigue.

**Objetivo.** Construir de punta a punta tres frentes —el menú por procesos, el ciclo de compra del GPS con su SIM, su instalación y su historial, y las áreas como dimensión del aislamiento de datos— en ocho fases pequeñas que dejan el sistema funcionando y en verde al final de cada una.

**Cómo leerlo.** Las rutas van abreviadas como en el análisis: `B/` es `erp-millennium-mobility-backend/src/main/java/ky/millennium/mobility/erp/`, `M/` sus migraciones, `T/` sus pruebas, `F/` es `erp-millennium-mobility-frontend/src/app/` y `E/` su carpeta `e2e/`. Cada fase dice, en este orden: su objetivo, lo que la rige, sus migraciones, el backend, la API y los errores, el frontend, las pruebas, la documentación, el orden de los commits y cuándo está terminada. Las fases, las migraciones y las decisiones de la versión 1 cambiaron de número: la equivalencia está en el §12. **«Con acceso a todas las áreas»** dice de una asignación que no está limitada a un área (en la pantalla, «Todas las áreas»); **«general»** dice de un documento que no tiene área y que leen todas.

---

## 1. Lo que decidió el responsable el 22/11/2026

### Áreas

| # | Decisión |
|---|---|
| A1 | Cada empresa tiene sus propias áreas, configurables: se agregan, se renombran y se desactivan; nunca se borran |
| A2 | La asignación de un rol lleva un área opcional; vacía significa todas, también las que se creen después |
| A3 | El área separa todo compras: plantillas, órdenes, facturas, notas de crédito, pagos y lo que se debe. Proveedores y tipos de orden siguen compartidos: un proveedor lo usan varias áreas |
| A4 | Un documento sin área es general: lo leen todas las áreas y solo lo escribe quien tiene acceso a todas las áreas |
| A5 | La frontera entre áreas la defiende el código, como la marca (D-102); la base sigue defendiendo solo la empresa |
| A6 | El área también oculta los datos de las facturas de otra área en el costo de compra de la unidad y del equipo y en el reporte de stock |
| A7 | Un pago es de un área; mezclar áreas se rechaza; pagar con el saldo del proveedor exige acceso a todas las áreas |
| A8 | El inventario de vehículos y el de equipos siguen compartidos entre áreas |
| A9 | Una persona puede tener acceso a algunas áreas: se le otorga el mismo rol una vez por área, y el diálogo de otorgar acepta varias áreas a la vez. Tener todas las áreas de hoy no es tener acceso a todas las áreas: eso lo da solo la asignación sin límite, que incluye las futuras |
| A10 | Una plantilla puede servir a todas las áreas. El área de la plantilla dice quién la usa (general = todas); el área de la orden es la de quien la emite |
| A11 | Lo que ya existe también se separa: una operación única, «Asignar área», le da su área a una orden general, con sus facturas, notas y pagos, y a una factura general sin orden |

### Menú

| # | Decisión |
|---|---|
| B1 | El menú se ordena por procesos: Flota, Monitoreo, Compras, Finanzas, Herramientas, Administración, Catálogos y Seguridad |
| B2 | Sin las etiquetas ADMIN y SUPER |
| B3 | Monitoreo es un grupo propio; la compra del GPS sigue en Compras, filtrada por área, y la instalación en Habilitación. El menú es el mismo para todas las áreas: el área filtra lo que hay dentro de cada pantalla |

### Ciclo de compra del GPS

| # | Decisión |
|---|---|
| C1 | La orden de compra tiene líneas de equipo: un modelo del catálogo de equipamiento, la cantidad y el precio |
| C2 | Una factura es de vehículos o de equipos, nunca de los dos; una orden también (Q24) |
| C3 | Una línea de factura por cada equipo físico; el formulario acepta «50 × modelo» y lo abre en líneas; una factura de equipos admite hasta 500 |
| C4 | Cada línea crea su equipo «Por ingresar», sin ubicación ni identificadores; el proveedor sale de la factura. La factura se registra sin los datos de los GPS y se completan después |
| C5 | Al llegar el producto se escanea el QR, que llena solo el IMEI; la API del fabricante completa el resto (serial, modelo, firmware, SIM). Si algo no cuadra, el equipo queda en «Revisar»; si la API no responde, se completa a mano con permiso |
| C6 | El costo de compra del GPS queda en el equipo y nunca se suma a una unidad; la instalación conserva solo su costo de mano de obra |
| C7 | Faltantes: si la factura trae menos, la orden queda con pendiente; si la factura trae todo y llega menos, los que faltan siguen por ingresar, con un aviso de antigüedad, y una nota de crédito «anulación de equipo» los anula |
| C8 | Cumplir una orden con faltante pide un motivo, y la orden muestra «Cumplida con faltante: N». Vale para toda orden |
| C9 | Solo hay notas de crédito: D-23 no cambia |
| C10 | Fallados: el proveedor a veces los reemplaza (otro IMEI, sin factura nueva) y a veces emite una nota de crédito; el sistema tiene los dos caminos |
| C11 | La SIM es un dato de su GPS: viene con él, se reemplaza con él y su línea la maneja el proveedor; se exige para instalarlo —el GPS tiene su ICCID—, no para que esté Disponible. Enmendada por Q30 y Q31: decía que la SIM era un registro propio con la historia de su asignación al GPS |
| C12 | La API para completar los datos la expone el fabricante del GPS |
| C13 | Cada equipo tiene su historial: sus estados, con quién y por qué, y en qué unidad estuvo y cuándo se retiró o reemplazó. Es lo que anunció al cliente el correo del 16/11/2026 |

---

## 2. Las preguntas, resueltas el 22/11/2026

El responsable aprobó las recomendaciones de la auditoría. El plan está escrito con estas respuestas.

| # | Pregunta | Decisión | La usa |
|---|---|---|---|
| Q1 | ¿Cómo se llama el grupo de compras? | «Compras» | Fase 1 |
| Q2 | ¿Dónde van Por pagar, Pagos a proveedores y Fondos? | En un grupo «Finanzas» | Fase 1 |
| Q3 | ¿Qué pantallas de mantenimiento se muestran solo a quien escribe? | Solo Catálogos; Empresas y Ubicaciones siguen con la lectura. Como hoy solo el superadministrador escribe catálogos (D-150), solo él los ve, también por dirección: Contabilidad deja de ver Tipos de cambio, y Operaciones y el administrador de empresa, Estados y motivos y Requisitos de equipos. Se enmiendan D-141, §5.6 y DP-18 | Fase 1 |
| Q4 | ¿Se muestran ya las opciones de Monitoreo que no existen? | No: cada opción aparece cuando su pantalla existe | Fase 1 |
| Q5 | ¿El acceso sale del área del colaborador o de la asignación? | De la asignación. El área del colaborador es informativa y se propone al otorgarle un rol | Fase 6 |
| Q6 | ¿Quién crea y cambia las áreas? | El superadministrador y el administrador de cada empresa, con una asignación de toda la empresa y con acceso a todas las áreas | Fase 6 |
| Q7 | ¿Una nota o un pago puede juntar áreas? | No: incluye documentos de una sola área, y lo general cuenta como un valor propio | Fase 7 |
| Q8 | ¿Quién escribe lo compartido de compras? | Cualquier comprador crea proveedores y les agrega contactos; editar o desactivar un proveedor, sus contactos o un tipo de orden pide acceso a todas las áreas | Fase 7 |
| Q9 | ¿Quién ve el saldo a favor de un proveedor? | Quien puede usarlo: una asignación de `pagos.ver` con acceso a todas las áreas. Para los demás, el saldo y el total que se le debe al proveedor no viajan | Fase 7 |
| Q10 | ¿Alguna regla fuera de compras mira el área? | No | Fase 7 |
| Q11 | ¿Las exportaciones de compras llevan el área? | Sí: una columna «Área» | Fase 7 |
| Q12 | ¿Cómo se paga una factura de equipos? | Como un todo, sin imputación por equipo. La base no exige imputaciones, así que no hay migración de pagos | Fase 2 |
| Q13 | ¿Cómo se registra el reemplazo de un GPS fallado? | El equipo nuevo apunta al que reemplaza; la línea de la factura no se toca | Fase 3 |
| Q14 | ¿Quién escribe un documento de equipos, que no tiene marca? | Una asignación de todas las marcas, en toda escritura: registrar, modificar, anular, la nota y el pago | Fase 2 |
| Q15 | ¿Qué pasa con los equipos al anular una factura de equipos? | Se anulan los que siguen por ingresar, al anular la factura y con la nota «anulación de la factura»; los ya recibidos quedan, sin factura vigente | Fase 2 |
| Q16 | ¿Un equipo devuelto al proveedor puede volver? | Sí, reparado, a Disponible; salvo que otro equipo ya lo haya reemplazado | Fases 2 y 3 |
| Q17 | En «Revisar» por otro modelo, ¿qué se corrige? | El modelo del equipo, dentro del mismo tipo; la línea conserva el modelo del comprobante | Fase 3 |
| Q18 | ¿Qué se guarda al cumplir una orden con faltante? | Solo el motivo; el faltante se calcula | Fase 2 |
| Q19 | ¿Qué permisos nuevos y quién los tiene? | `stock.ingresar_equipamiento` (fase 2), para el superadministrador, el administrador de empresa y Operaciones. Los tres de la SIM de la fase 3 (`stock.crear_sims`, `stock.editar_sims`, `stock.cambiar_estado_sims`) salieron con Q30 y Q31 | Fase 2 |
| Q20 | ¿La factura cobra la SIM aparte? | No: viene incluida en el precio del GPS | Fase 3 |
| Q21 | ¿Qué fabricante, con qué API? | **En parte respondida (22/11/2026):** Teltonika (en la foto del responsable, un FMC130). Su QR lleva solo el IMEI, y el proveedor no entrega la lista de IMEIs: se escanean al llegar o al registrar la factura, también con la cámara del celular desde la misma web. La API sería la de FOTA WEB de Teltonika, con un token de integración que se crea en su configuración y que se muestra una sola vez; su lista de equipos se filtra por varios IMEIs a la vez y trae el serial. **Falta:** la cuenta de FOTA WEB de la empresa, confirmar que los equipos comprados aparecen en ella, el token, los límites de la API y su referencia completa (api.teltonika.lt/documents). Sin ellos se construye todo menos el adaptador | Fase 3 |
| Q22 | ¿Cómo se lee el QR? | Lector USB o Bluetooth como forma principal; la cámara como apoyo, con una librería cargada solo en esa pantalla, y el sitio la habilita (`camera=(self)`) | Fase 3 |
| Q23 | ¿Qué pasa con lo que ya existe? | Se separa por área con la operación «Asignar área» (A11). Las plantillas de hoy pueden quedar generales, como formatos comunes | Fase 7 |
| Q24 | ¿Una orden puede mezclar vehículos y equipos? | No: una orden es de un solo tipo, y la factura vinculada tiene el tipo de su orden | Fase 2 |
| Q25 | Sin la API del fabricante, ¿cómo se completan cientos de equipos? | Con el proveedor `manual`, la pantalla de ingreso toma el IMEI, el serial y el ICCID de cada equipo y lo deja Disponible; con un fabricante, solo el IMEI, y el resto lo completa la API | Fase 3 |
| Q26 | ¿En qué orden van los frentes? | Menú → GPS → áreas. El monitoreo es la próxima etapa anunciada al cliente; no hay producción, así que el aislamiento no protege todavía datos reales; y «Asignar área» le da su área después a lo que se registre antes | Todas |
| Q27 | Con una plantilla general, ¿de qué área es la orden? | De quien la emite: su área si está limitado a una; a elegir entre las suyas si tiene varias; un área o «General» si tiene acceso a todas las áreas | Fase 7 |
| Q28 | ¿Qué va primero? (precisa Q26) | El flujo operativo hasta el ingreso de los IMEIs: la fase 0 reducida, la fase 2 y la fase 3a. La SIM como registro propio —que Q30 y Q31 retiraron—, la API del fabricante, la instalación, el menú y las áreas van después, en el orden que diga el responsable | Todas |
| Q29 | ¿El reemplazo de un fallado entra en el primer tramo? | Sí, en la fase 3a; su columna va en V149 para no renumerar las migraciones | Fases 2 y 3a |

**Respondidas después, el mismo 22/11/2026:**

| # | Pregunta | Decisión | La usa |
|---|---|---|---|
| Q30 | ¿Una SIM se cambia alguna vez sola, en el mismo GPS? | No: «La 1 no pasa». Ya había dicho: «La SIM existe para con el GPS; si el GPS se cambia, se cambia con todo y SIM», que el reemplazo de un fallado de la fase 3a respeta. La SIM es un dato de su GPS toda su vida: no hay historia de sus asignaciones a los GPS ni rutas para asignarla o retirarla. Enmienda C11 y D-285 | Fases 3b, 4 y 5 |
| Q31 | ¿Un GPS que vuelve al proveedor pide dar de baja su línea con el operador? | No: «En la 2 también se encarga el proveedor, el proveedor me da los GPS + los SIM». El proveedor entrega cada GPS con su SIM y se encarga de su línea; el ERP no da de baja nada con un operador ni lleva el ciclo de vida de la SIM (activa, suspendida, cancelada). Enmienda D-285 | Fase 3b |
| Q32 | ¿Los equipos que no llegan se pueden dar de baja sin la nota de crédito del proveedor? | No, como una unidad: un equipo que la factura cobra y no llegó solo se da de baja con la nota «anulación de equipo» del proveedor, o con la anulación de la factura, como una unidad con «anulación de unidad»; el aviso de antigüedad sirve para reclamarlo. Lo común no pide nota: si llegan 45 de 50 y la factura es por 45, solo nacen 45 equipos y la orden se cierra con «Cumplir» y su motivo (D-284), en equipos y en vehículos. Lo que se debe sigue siempre a los comprobantes. Si los casos sin nota se vuelven frecuentes, se puede sumar después un estado «En reclamo» que los saque de «Ingreso de equipos» sin tocar la factura | — (nada que construir) |

**Respondidas el 23/11/2026, y construidas el mismo día** (backend y pantallas, sin migraciones; sus hashes, en la línea de Estado):

| # | Pregunta | Decisión | La usa |
|---|---|---|---|
| Q33 | ¿Cómo sale a Inactivo una unidad Robada con un GPS del inventario? | **La baja definitiva retira sola sus equipos** (D-289). El paso de Robado a Inactivo —hoy, «Baja definitiva»— retira en el mismo cambio cada equipo del inventario instalado: cierra su instalación con el motivo `robo` en el día del cambio, en la zona de la empresa, y lleva el equipo a Baja en la ubicación de la unidad, con «Retirado de la unidad {VIN} por robo, en la baja definitiva de la unidad» en su historia. Lo decide el par de estados, no el motivo (`StolenUnitExitPolicy`). Los candados: la unidad, después sus instalaciones con equipo y después sus equipos, por identificador. Un equipo instalado ese mismo día responde `VEHICLEEQUIPMENT_END_NOT_AFTER_START`, sin escribir. Las instalaciones sin equipo siguen abiertas, y la anulación de una unidad Robada sigue retenida. Pérdida total no congela el equipamiento (V142 marca `blocks_equipment_changes` solo en `ROBADO`), así que su GPS se retira antes a mano. `allowedActions.devicesRetiredReasonIds` (UUID[]) lista el camino, que sale de `devicesHeldReasonIds`, y «Cambiar estado», al elegirlo, avisa «Los equipos del inventario instalados se retiran con el motivo «Robo» y pasan a Baja.» | Construida (`fleet`, FLT-R161) |
| Q34 | ¿Cómo se corrige el modelo de un equipo? | **Desde su ficha, dentro de su tipo, salvo el de una factura** (D-290). `PATCH /equipments/{id}` acepta `equipmentModelId` y revisa, en orden: que el modelo exista (`404 EQUIPMENTMODEL_NOT_FOUND`), que sea del mismo tipo (`EQUIPMENT_MODEL_TYPE_MISMATCH`), que el modelo y su marca estén activos (`EQUIPMENT_MODEL_INACTIVE`, `EQUIPMENTBRAND_INACTIVE`) y que el equipo no haya nacido de una línea de factura (`422 EQUIPMENT_MODEL_FROM_INVOICE`, nuevo: «El modelo de un equipo comprado en una factura es el de su factura: se corrige con «Anular y copiar» de su factura»); el modelo que ya tiene no cambia nada. `allowedActions` suma `correctModel` después de `edit`, y la corrección queda en `updated_by` y `updated_at`. En «Editar equipamiento», con `correctModel`, «Modelo» es un selector con los modelos activos de marcas activas del mismo tipo, de cualquier marca, que conserva el suyo, y la marca sigue al modelo. La precisaron el mismo día D-292, que retiró la excepción de un `PATCH` con solo el modelo, porque la edición revisa solo los identificadores que cambia; D-294, que deja corregirlo al que reemplazó a uno comprado; y D-296, que corrige el de una factura con «Anular y copiar» | Construida (`equipment`, EQP-R29); Q17, para la fase 3b, queda con Q40 |

**Respondidas después, el mismo 23/11/2026, y construidas ese día** (el responsable aprobó las recomendaciones; Q35 y Q36 las abrió la fase 4, Q37 la fase 7, y Q38 a Q41 las respuestas a Q33 y Q34; backend y pantallas, con V165 y V166; sus hashes, en la línea de Estado):

| # | Pregunta | Decisión | La usa |
|---|---|---|---|
| Q35 | ¿La instalación guarda quién la retiró? | **Sí: «Retirado por»** (D-291). El retiro y el reemplazo reciben `removedByEmployeeId`, opcional: un colaborador activo de la empresa de la unidad, como «Instalado por» (`404 EMPLOYEE_NOT_FOUND`, `422 VEHICLEEQUIPMENT_REMOVER_OTHER_COMPANY`, nuevo, y `EMPLOYEE_INACTIVE`). V166 agrega `vehicle_equipments.removed_by_employee_id`, con su llave a la empresa de la unidad y `vehicle_equipments_remover_ck`, y «Instalaciones», en la ficha del equipo, dice «Retiró: {nombre}» o «sin indicar». El retiro automático de la baja definitiva de una unidad Robada (D-289) lo deja vacío | Construida (`fleet`, V166, FLT-R162); la tabla de equipamiento de la habilitación todavía no lo muestra |
| Q36 | ¿Se edita un equipo instalado? | **Sí, con límites** (D-292, junto con Q41). Sobre un equipo instalado, `PATCH /equipments/{id}` rechaza otro valor en `locationId`, `supplierId`, `serialNumber`, `productCode` o `imei`, en ese orden (`422 EQUIPMENT_INSTALLED_FIELD_LOCKED`, nuevo, con el campo en `details` y en `context.field`); se editan la descripción, el firmware (`firmware`, `dtb`, `firmwareBle`, `bleDtb`), la SIM (ICCID e IMSI) y el modelo (D-290). Mandar un campo bloqueado con el mismo valor no cuenta como cambio, y la pantalla los muestra deshabilitados, cada uno con su pista | Construida (`equipment`, EQP-R30) |
| Q37 | ¿Cómo se corrige el área de un documento que la recibió mal? | **Con «Cambiar área»** (D-295): `POST /purchase-area-changes`, con `compras.asignar_area` y acceso a todas las áreas, mueve a otra área, o de vuelta a General con `areaId` en `null`, la cadena completa —la orden con sus facturas, notas y pagos, o una factura sin orden con sus notas y pagos, también los anulados—, con las revisiones de «Asignar área» y tres rechazos propios (`PURCHASEAREACHANGE_SAME_AREA`, `PURCHASEAREACHANGE_INVOICE_HAS_ORDER` y `PURCHASEAREACHANGE_DOCUMENTS_MIXED`), y deja una fila de la bitácora por documento con el área vieja y la nueva. V165 permite el cambio en la base solo dentro de esa operación, y cuatro disparadores diferidos revisan al confirmar que cada cadena quede en una sola área. No pide motivo | Construida (`procurement`, V165, PRC-R212) |
| Q38 | ¿Puede otro estado congelar el equipamiento? | **No: la marca la fija el sistema** (D-293). `PUT /vehicle-statuses/{id}` acepta `blocksEquipmentChanges` ausente o igual al que el estado tiene, y otro valor responde `422 VEHICLESTATUS_EQUIPMENT_FREEZE_LOCKED`, nuevo, antes que cualquier otra regla; la casilla se lee deshabilitada. Solo Robado congela el equipamiento, y su baja definitiva retira sola los equipos (D-289) | Construida (`fleet`, FLT-R151) |
| Q39 | ¿Corrige su modelo el equipo que reemplazó a uno comprado? | **Sí, dentro de su tipo** (D-294). Solo el equipo que nació de la línea de la factura conserva el modelo de la factura: `PurchaseOrigins` ya no sigue la cadena de reemplazos, y el que lo reemplazó (D-285) recibe `correctModel` y lo corrige desde su ficha con las reglas de D-290 | Construida (`equipment`, EQP-R29) |
| Q40 | ¿Cómo se corrige el modelo de un equipo ingresado de una factura, si llegó otro? | **Con «Anular y copiar»** (D-296). La copia de una factura de equipos acepta una línea que vuelve a tomar su equipo con otro modelo del mismo tipo, activo y de una marca activa (otro tipo: `422 PURCHASEINVOICE_EQUIPMENT_MODEL_TYPE_MISMATCH`, nuevo); esos equipos, y los que los reemplazaron mientras conservan el modelo de la línea anulada, pasan al nuevo en la misma transacción, por la fachada de `equipment` (`correctPurchasedDeviceModels`), porque son las mismas unidades físicas, sin fila en su historia. `EQUIPMENT_MODEL_FROM_INVOICE` manda ahora a «Anular y copiar» | Construida (`procurement`, `equipment`, PRC-R213); «Revisar», de la fase 3b, lo concilia con Q17 al abrirse |
| Q41 | ¿Se editan los demás datos de un GPS enlazado sin IMEI ni ICCID mientras está instalado? | **Sí** (D-292, junto con Q36). La edición valida solo los identificadores que cambian: uno que ya faltaba y no se toca no la frena, y dejar vacío uno que el tipo exige se sigue rechazando (`EQUIPMENT_TELEMATICS_REQUIRED`, `EQUIPMENT_SERIAL_REQUIRED`). Reemplaza la excepción de Q34 para un `PATCH` con solo el modelo, que se retiró, y el formulario pide solo los identificadores que el equipo ya tiene | Construida (`equipment`, EQP-R30) |

**Pendiente del responsable:** la información del fabricante (Q21), para la fase 3b.

---

## 3. Lo que no cambia

| Qué | Por qué se mantiene |
|---|---|
| **La frontera entre empresas** | RLS, `DatabaseAccessScope`, `AccessScopeJpaTransactionManager` y las funciones de V29 no cambian. Ninguna política de la base mira el área |
| **Las rutas existentes** | El menú cambia de grupo, no de dirección; ninguna ruta de la API ni del frontend se renombra. Las rutas nuevas se agregan |
| **Los códigos de permisos, de errores y las claves de mensajes existentes** | Son contratos por nombre con el catálogo, el frontend y las pruebas. Solo se agregan; las únicas claves que salen son las del menú viejo que nadie más usa |
| **Los datos existentes** | Ninguna migración le da un área a una fila: lo que existe queda general hasta que alguien con acceso a todas las áreas use «Asignar área» (fase 7). Solo dos migraciones mueven datos: V148, que llena el tipo de cada línea y de cada orden desde lo que ya tienen; y V150, que abre la historia de cada equipo con su estado de hoy. El traslado del ICCID y el IMSI de cada equipo a una SIM salió con Q30 y Q31: siguen en el equipo |
| **El aislamiento fuera de compras** | Vehículos, fondos, inventario de equipos, cargas, catálogos y organización no miran el área (Q10) |
| **Las reglas de las facturas de vehículos** | D-153, D-160, D-161, D-191, D-193, D-248, las imputaciones por unidad y la regla del concesionario siguen iguales para las facturas de vehículos |
| **El orden de los candados existente** | Cada candado nuevo va al final de su cadena (empresa → sede → marca → área; la unidad y su instalación antes que el equipo; el equipo antes que su pedido al fabricante) y se prueba |

---

## 4. Cómo se trabaja

- **Una fase, una rama por repositorio**, en un worktree bajo `.wt/`, y una base de verificación propia, `erp_mm_<fase>_verify`, creada con su dueña (`erp_mm_owner`). Al integrar se borran el worktree, la rama y la base.
- **Cada push a `develop` despliega `dev`:** el backend en App Platform y el frontend en Cloudflare Pages, por separado. Por eso cada fase integra primero el backend, que acepta lo viejo y lo nuevo, y después el frontend. La rama de la documentación se integra a `master` el mismo día que la del backend.
- **Un cambio incompatible de la API va en tres pasos** (convenciones de la API, §1): el backend acepta las dos formas, el frontend pasa a la nueva y el backend retira la vieja. En este plan solo lo es el cuerpo de la instalación (fase 4, con su tercer paso en la 5).
- **El CI corre en la rama antes de integrar** (*Run workflow*). `gh` no está autenticado en la máquina de trabajo: el resultado lo mira el responsable en GitHub. El e2e del frontend solo corre si pasan el build y las unitarias (`needs: build`): una unitaria en rojo esconde el e2e entero.
- **Las pruebas, a la medida del cambio.** En local, en una base nueva de `erp_mm_owner`, con las dos cuentas y nunca con el `.env` remoto: las pruebas del contexto que cambia, las de sus consumidores, las de arquitectura y `ErpMillenniumMobilityBackendApplicationTests`. Las fases 2 a 7 cambian el esquema o la seguridad, así que corren además la suite completa del backend. En el frontend: el build, las specs de lo que cambia y `lint:styles`. El e2e lo corre el CI.
- **La excepción de las imágenes de referencia.** El CI del e2e corre en `windows-latest` y compara imágenes `*-win32.png`. Una fase que cambia una pantalla a propósito regenera sus imágenes en la máquina de trabajo (Windows) con `npm run e2e:update -- e2e/visual.spec.ts e2e/permission-matrix.spec.ts`, en un commit propio y después de revisar una muestra. La fase 1 lo hace con todas las pantallas del menú; las demás, solo con las pantallas que cambian. Una imagen que cambia sin que la fase lo diga es una regresión.
- **Primero se caracteriza.** Las pruebas de caracterización de compras se escriben antes de la fase 7, en `develop`, y la fase 7 termina con ellas sin cambios; la fase 4 escribe las suyas de la instalación antes de tocarla. Las fases 2 y 3a se apoyan en las pruebas de compras que ya existen y en la suite completa.
- **Cada fachada nueva trae su prueba de contrato**, en el módulo que la ofrece.
- **Toda tarea programada nueva** empieza con `SingleInstanceRun.runAlone` y entra en la lista exacta de `DeadlockRetryTests` (L123-126). **Toda variable nueva** va en `.env.example`, en `.do/app.yaml` y en el README.
- **El diccionario manda.** Cada fase actualiza en el mismo cambio el diccionario, `contextos/`, las convenciones y `procesos-y-pantallas`, y pasa su decisión de «Decidido y por construir» a «cerradas».
- **Las cuatro copias locales de las skills** (`.claude` y `.cursor` del backend y del frontend) se actualizan a mano el día que integra cada fase; nunca se versionan (decisión 15 del plan de migración).

---

## 5. Las fases

| Fase | Qué | Contextos | Migraciones | Costo | Riesgo | Depende de |
|---|---|---|---|---|---|---|
| 0 | Preparación | Frontend (catálogo simulado, e2e), documentación | — | 1 a 2 días | Bajo | — |
| 1 | Menú por procesos | Frontend: `app-shell`, `shared`, `iam` (matriz) y las rutas de los catálogos; diccionario (D-141) | — | 2 a 4 días | Bajo | 0 |
| 2 | Compra de equipos e historia del equipo | `procurement`, `equipment`, `iam`; frontend `procurement`, `equipment` | V148 a V153 | 14 a 18 días | **Alto** | 0 |
| 3a | Ingreso de equipos por escaneo, sin fabricante, y reemplazo de un fallado | `equipment`, `procurement`; frontend `equipment`, `app-shell` | — (la columna del reemplazo va en V149) | 5 a 7 días | Medio | 2 |
| 3b | API del fabricante; la SIM queda como dato del GPS (Q30, Q31) | `equipment`, `procurement`; frontend `equipment` | Una: el completado, que se numera al abrir su rama (V155 la tomó la fase 4) | Menos que los 5 a 7 días de la versión 3, que contaban la SIM, y 2 a 4 más para el adaptador | Medio | 3a; Q21 para el adaptador |
| 4 | Instalación unida al inventario e historial del equipo (D-263, D-264) | `fleet`, `equipment`; frontend `fleet`, `equipment` | V155, V156 (eran V157 y V158) | 8 a 11 días | Medio | Hecha el 23/11/2026, antes de la 3b y sin «Por completar» (fase 4, precisión 2) |
| 5 | Limpieza | `fleet`; frontend `fleet` | Ninguna | Menos de 1 día | Bajo | Hecha e integrada el 23/11/2026 (fase 5, «Hecha») |
| 6 | Áreas: catálogo, asignación y sesión | `shared`, `iam`, `organization`, `documents`; frontend `organization`, `iam` | V157 a V160 | 5 a 7 días | Medio | 0 |
| 7 | Compras por área y «Asignar área» | `procurement`, `audit`, `reporting`, `iam`; frontend `procurement`, `iam`, `change-log` | V161 a V163 (eran V164 a V166) | 12 a 16 días | **Alto** | Hecha e integrada el 23/11/2026, después de la 6 (fase 7, «Hecha») |

**El primer tramo (Q28)** es la fase 0 reducida, la 2 y la 3a: el flujo operativo hasta el ingreso de los IMEIs, con el reemplazo de un fallado. Lo demás —3b, 4, 5, 1, 6 y 7— va después, en el orden que diga el responsable y con las dependencias de la tabla. Las migraciones conservan su número: si el orden de integración cambia, se renumeran al abrir la rama. Con lo integrado hasta hoy (la fase 2 usó V148 a V154) y sin la SIM (Q30, Q31), si las fases van en el orden de la tabla, la 3b usa V155; la 4, V156 y V157; la 5, ninguna; la 6, V158 a V161; y la 7, V162 a V164.

---

## 6. Fase por fase

### Fase 0 — Preparación

**Objetivo.** Empezar con el CI del frontend en verde de verdad —unitarias y e2e—, las decisiones escritas y las pruebas que fijan el comportamiento de hoy.

**Pasos:**

1. **El CI del frontend, en verde.** `stock.anular_lectura_kilometraje` llegó con V145 (D-267) y está en `F/iam/application/permission-codes.ts` L103, pero no en `seedPermissions` de `E/support/permission-catalog.ts`; `F/iam/application/permission-codes.spec.ts` «names only permissions of the catalog» falla desde `3f47a838` (20/11), integrado en `f614aa34`. Como el e2e de `.github/workflows/ci.yml` tiene `needs: build` y `build` corre `ng test` (L40-41), el e2e no corre en `develop` desde entonces: cinco integraciones del frontend llegaron sin él. Rama `fix/permission-catalog-mileage-annulment`:
   - agregar el permiso a `seedPermissions`, con su categoría (`desactivar_anular`). La matriz simulada no se toca: el superadministrador recibe todos los permisos y el administrador de empresa los de inventario por `STOCK_OF_STEP_5` (L269);
   - ajustar lo que cambia con un permiso más: `E/roles.spec.ts` L160 («39 de 39» → «40 de 40») y L184 («6 de 39» → «6 de 40»), `E/permission-matrix.spec.ts` L316 («22/39» → «22/40»);
   - correr el **e2e completo** en la rama (*Run workflow*), arreglar lo que esté en rojo por las integraciones que llegaron sin e2e —por ejemplo `vehicle-profile-mileage`, que D-267 pudo mover con la columna de acciones— y regenerar, en un commit propio y después de mirarlas, las nueve imágenes que cambian por el conteo (`permission-matrix-expanded`, `permission-matrix` y `role-profile`, en sus tres anchos) y las que el e2e señale;
   - se integra antes de la fase 1, para que la regeneración del menú no esconda una regresión anterior.
2. **El fabricante (Q21):** nombre, documentación de la API, credenciales de prueba, límites de uso y si la consulta es por equipo o por lote.
3. **Las decisiones en el diccionario**, en «Decidido y por construir» de §14, versión 1.80:

   | Decisión | Qué recoge | Se cierra en |
   |---|---|---|
   | D-283 | La compra de equipos: tipo de línea y de orden, factura de equipos, equipo por ingresar, el aviso de antigüedad, la nota «anulación de equipo», la anulación de la factura y «Anular y copiar», pago a nivel de factura, costo del equipo, la historia de estados; enmienda D-173 y precisa D-160, D-187 y D-254 | Fase 2 |
   | D-284 | Cumplir una orden con faltante pide su motivo | Fase 2 |
   | D-285 | El ingreso por escaneo, la API del fabricante, el reemplazo y la SIM | Fase 3 |
   | D-286 | El historial del equipo en sus unidades y la vinculación de lo ya instalado, también lo de la carga inicial; precisa D-206, D-263 y D-264 | Fase 4 |
   | D-287 | Las áreas: catálogo por empresa, área en la asignación (una asignación por área; «todas las áreas»), regla de lectura y escritura, el área del colaborador, la sesión | Fase 6 |
   | D-288 | Compras por área: la plantilla dice quién la usa y la orden es del área de quien la emite, facturas, notas, pagos, Por pagar, bitácora, costo, reporte y «Asignar área» | Fase 7 |

   El menú no es una decisión nueva: va en `DESIGN.md` y en `procesos-y-pantallas`, y la fase 1 enmienda D-141.
4. **Pruebas de caracterización de compras** —desde la versión 3, se escriben antes de la fase 7, no en la fase 0 reducida—, en `develop`, contra el código de ese momento, con asignaciones limitadas a una marca:
   - las listas, la lectura por identificador y la exportación de plantillas, órdenes, facturas, notas, pagos y Por pagar, y `GET /suppliers/{id}/payables`;
   - el costo de una unidad y el reporte de stock;
   - la bitácora de un registro con marca;
   - dos asignaciones de marcas distintas en la misma empresa (la unión de hoy);
   - la forma de la llave de `DocumentScopes` (`DocumentScopesTests`).

   Casi todas existen como `*AccessPolicyTests` y pruebas de integración de cada lista; se agregan las que falten. Pasan sin cambios al terminar la fase 7.

**Terminada cuando:** el CI del frontend está en verde en `develop`, con el e2e corrido entero, y las decisiones están escritas en el diccionario. Las pruebas de caracterización quedan para antes de la fase 7.

---

### Fase 1 — Menú por procesos

**Objetivo.** El menú ordenado por procesos, sin etiquetas, con Catálogos para quien puede escribir en él, y la matriz de permisos agrupada igual.

**Lo que la rige.** B1 a B3; Q1 a Q4; D-117 (el menú decide por permiso); la regla de que una opción y su ruta piden lo mismo.

**Migraciones y backend.** Ninguno.

**Frontend.**

1. **`F/app-shell/app-navigation.ts`.** El menú nuevo, con las rutas de hoy:

   | Grupo | Icono | Opción | Ruta | Se ve con |
   |---|---|---|---|---|
   | — | `home` | Inicio | `/home` | Cualquier sesión |
   | Flota | `local_shipping` | Inventario | `/fleet/vehicles` | `stock.ver` |
   | | | Habilitación | `/fleet/enablement` | `stock.ver` |
   | Monitoreo | `sensors` | Equipos | `/equipment/inventory` | `stock.ver` |
   | Compras | `shopping_cart` | Proveedores | `/procurement/suppliers` | `compras.ver` |
   | | | Plantillas de OC | `/procurement/purchase-order-templates` | `compras.ver` |
   | | | Órdenes de compra | `/procurement/purchase-orders` | `compras.ver` |
   | | | Facturas | `/procurement/invoices` | `compras.ver` |
   | | | Notas de crédito | `/procurement/credit-notes` | `compras.ver` |
   | Finanzas | `account_balance` | Por pagar | `/procurement/payables` | `pagos.ver` |
   | | | Pagos a proveedores | `/procurement/supplier-payments` | `pagos.ver` |
   | | | Fondos | `/financing/funds` | `fondos.ver` |
   | Herramientas | `build` | Reportes | `/reports` | `reportes.ver` |
   | | | Cargas de archivos | `/imports` | `habilitacion.ver_cargas` |
   | | | Bitácora de cambios | `/change-log` | `bitacora.ver` |
   | Administración | `settings` | Empresas | `/organization/companies` | `configuracion.ver_estructura` |
   | | | Ubicaciones | `/organization/locations` | `configuracion.ver_estructura` |
   | | | Colaboradores | `/organization/employees` | `configuracion.ver_datos_personales_colaboradores` |
   | Catálogos › Generales | `category` | Monedas | `/location/currencies` | `catalogos.ver` y alguno de `crear_monedas`, `editar_monedas`, `desactivar_monedas` |
   | | | Tipos de cambio | `/location/exchange-rates` | `catalogos.ver` y alguno de `registrar_tipo_cambio`, `corregir_tipo_cambio` |
   | | | Países | `/location/countries` | `catalogos.ver` y alguno de `crear_paises`, `editar_paises`, `desactivar_paises` |
   | Catálogos › Flota | | Tipos de vehículo | `/vehicle-catalog/vehicle-types` | `catalogos.ver` y alguno de `crear_tipos_vehiculo`, `editar_tipos_vehiculo`, `desactivar_tipos_vehiculo` |
   | | | Marcas y modelos | `/vehicle-catalog/brands-and-models` | `catalogos.ver` y alguno de `crear_marcas_y_modelos`, `editar_marcas_y_modelos`, `desactivar_marcas_y_modelos` |
   | | | Colores | `/vehicle-catalog/colors` | `catalogos.ver` y alguno de `crear_colores`, `editar_colores`, `desactivar_colores` |
   | | | Tipos de equipamiento | `/equipment/types` | `catalogos.ver` y alguno de `crear_equipamiento`, `editar_equipamiento`, `desactivar_equipamiento` |
   | | | Estados y motivos | `/fleet/vehicle-statuses` | `catalogos.ver` y alguno de `crear_estados`, `editar_estados`, `desactivar_estados` |
   | | | Tipos de documento | `/fleet/document-types` | `catalogos.ver` y alguno de `crear_tipos_documento`, `editar_tipos_documento`, `desactivar_tipos_documento` |
   | | | Requisitos de equipos | `/fleet/equipment-requirements` | `catalogos.ver` y `editar_tipos_documento` |
   | Seguridad | `shield` | Usuarios | `/iam/users` | Como hoy: `ver_cuentas` con alcance sistema y alguno de `asignar_roles`, `bloquear_cuentas`, `restablecer_contrasenas` |
   | | | Roles y permisos | `/iam/roles` | `seguridad.ver_roles` |

   - Todos los códigos de Catálogos son `catalogos.*` y salen de `PermissionCodes.Catalogs`; los retirados (`gestionar_generales`, `gestionar_flota`, `gestionar_estados`) no cuentan.
   - Un ayudante nuevo junto a `anyOf` (L66): `writersOf(read, ...writes)` = `{ permissions: { allOf: [read], anyOf: writes } }`.
   - El comentario de cabecera pasa de cuatro grupos a ocho, cada uno con la pregunta que responde.
   - Siguen siendo 30 opciones.
2. **Las rutas de Catálogos piden lo mismo que su opción**, también sus páginas hijas (la ficha de un país nuevo sigue pidiendo además `crear_paises`, D-141): `F/location/presentation/location.routes.ts` (L22-24, L47-91), `F/vehiclecatalog/presentation/vehiclecatalog.routes.ts` (L22-24, L56-69), `F/equipment/presentation/equipment.routes.ts` (L30-32) y `F/fleet/presentation/fleet.routes.ts` (L58-94). El guard, la precarga (`screen-preloading.ts` L72) y la paleta de comandos (`layout.ts` `paletteOptions` L99) las siguen solos.
3. **Sin etiquetas.** Sale `badge` de `AppNavigationGroup` (L59) y de los tres grupos que la usan (Configuración L214, Catálogos L283 y Seguridad L390), de `NavigationGroup` (`F/shared/presentation/components/layout/navigation.ts` L37-38), su dibujo en `layout.html` (L84-86), su estilo en `layout.scss` (L200-225) y las claves `navigation.badge.*`.
4. **Textos (`public/i18n/es.json`).**
   - Nuevas: `navigation.monitoring` «Monitoreo», `navigation.finance` «Finanzas», `navigation.tools` «Herramientas», `navigation.administration` «Administración».
   - Cambian: `navigation.equipment-inventory` «Equipos»; `navigation.equipment-types` «Tipos de equipamiento»; `modules.equipment-inventory.title` «Equipos», con una descripción de 90 caracteres o menos; los títulos de las rutas de `equipment.routes.ts` (L50, L57).
   - Salen `navigation.logistics`, `configuration`, `organization` y `purchasing-and-fleet` si un `grep` no encuentra otro uso.
5. **La matriz de permisos**, agrupada como el menú, y sin la palabra «área» para los grupos:
   - `permission-matrix-groups.ts`: `AREA_ORDER` pasa a `GROUP_ORDER` = Flota, Monitoreo, Compras, Finanzas, Herramientas, Administración, Catálogos, Seguridad; solo Catálogos conserva secciones (Generales, Flota); las pantallas de cada grupo siguen el orden del menú.
   - Dónde cae cada permiso (`placePermission`):

     | Permiso | Grupo › pantalla |
     |---|---|
     | Los de las cargas (`FILE_LOAD_PERMISSIONS`, L62-67, que se pregunta primero, como hoy) | Herramientas › Cargas de archivos, también `stock.cargar_migracion_inicial` y `fondos.cargar_archivo_garantia` |
     | `stock.*` | Flota › Inventario, salvo `stock.*_equipamiento`, que va a Monitoreo › Equipos (una pantalla con clave propia: `equipamiento` ya es «Tipos de equipamiento») |
     | `habilitacion.*` | Flota › Habilitación |
     | `compras.*` | Compras › su pantalla (proveedores y plantillas, ahora en Compras) |
     | `pagos.*` | Finanzas › Pagos a proveedores |
     | `fondos.*` | Finanzas › Fondos |
     | `reportes.*` | Herramientas › Reportes |
     | `bitacora.*` | Herramientas › Bitácora de cambios |
     | `configuracion.*` | Administración › su pantalla |
     | Los de colaboradores de `seguridad.*` | Administración › Colaboradores |
     | Los de roles de `seguridad.*` | Seguridad › Roles y permisos |
     | El resto de `seguridad.*` | Seguridad › Usuarios |
     | `catalogos.*` | Catálogos, como hoy, con Requisitos de equipos en Flota |

   - Corrige lo desfasado: Equipos en Monitoreo; Requisitos de equipos y Por pagar como pantallas sin permiso propio (no se dibujan hasta que tengan uno).
   - Renombra `PermissionPlace.area` → `group`, `AreaRow` → `GroupRow` (definido en `permission-matrix-rows.ts` L13, que usa `AREA_ORDER`, `SECTION_ORDER` y `SCREEN_ORDER` en L108-140) y las claves `iam.permission-matrix.areas` → `groups`, y ajusta `role-access-editor.ts` (L20-23, L159-171) y su html (L45-76), `permission-matrix.ts` (L313-338) y su html (L98-102), `permission-matrix.store.ts` (L123-125) y los textos de la matriz en `es.json` (L765-822).
   - `PERMISSION_MODULE_ORDER` (`permission-codes.ts` L313-329) deja de decir «seguridad al final, seguida de su bitácora»: la Bitácora pasa a Herramientas. `iam.roles-view.modules.stock` (`es.json` L339, «Logística») pasa a «Flota».
6. **`DESIGN.md`, revisión 99 «El menú por procesos»:** los grupos, su orden y sus iconos; sin etiquetas (reemplaza la regla 7 «Tag», L329); Catálogos solo para quien escribe; y L416, que todavía nombra los cuatro grupos viejos.
7. **El diccionario.** Q3 cambia quién abre un catálogo: se enmiendan D-141, §5.6 y DP-18, que dicen que una pantalla de catálogo se abre con `catalogos.ver`. Desde esta fase, la abre quien tiene la lectura y alguna escritura de esa pantalla; con la matriz sembrada, solo el superadministrador.

**Pruebas.**

- Unitarias que cambian: `F/app-shell/app-shell.spec.ts` (un rol con solo `catalogos.ver` ya no ve Catálogos, L91-104; las listas de grupos L106-147 y L175-193; «Organización tiene tres enlaces», L149-173; el grupo de Logística de L195-215, que pasa a Flota, Monitoreo y Finanzas; la Bitácora ahora en Herramientas L217-223; las pruebas «se ofrece con…» L357-361; la igualdad de rutas L384-417; la llave del grupo en L448), `layout.spec.ts` (sin etiqueta, L52 y L248-264), las specs de rutas de `location`, `vehiclecatalog`, `equipment` y `fleet`, `permission-matrix-groups.spec.ts` y `permission-matrix-rows.spec.ts`. `app-shell.spec.ts` afirma además, para cada uno de los siete roles sembrados, los grupos que ve: `renderSignedInAs` (L39-41) ya lee `SEEDED_MATRIX`, y el e2e no tiene sesiones de Operaciones, Contabilidad ni Taller (`E/support/api.ts` L92-103).
- e2e que cambian: `E/shell.spec.ts` (siguen 30 enlaces; el botón «Logística» de L240-257, L335, L363 y L453 pasa a «Flota» o «Compras»; un rol de solo lectura no ve Catálogos; la prueba de L293, que nombra «Configuración», se renombra), `E/permission-matrix.spec.ts`, `E/roles.spec.ts` (los conteos por grupo), `E/accessibility.spec.ts` L463, y los e2e que abren un catálogo con un rol que solo lee, que pasan al superadministrador o afirman la vuelta a Inicio: `E/exchange-rates.spec.ts` L410-413, `E/vehicle-document-types.spec.ts` L26 y L88-90, `E/vehicle-equipment-requirements.spec.ts` L36-41. `app.spec.ts` L128 (`navigation.logistics`) se renombra.
- Imágenes de referencia: se regeneran las de escritorio y tableta de todas las pantallas con menú (unas 120: 69 de escritorio y 63 de tableta, menos los seis diálogos y las seis pantallas de acceso), y las de la matriz y la ficha del rol, después de revisar una por grupo.

**Documentación.** `procesos-y-pantallas` §3.1 (la tabla del menú, entera) y su nota de quién ve Catálogos; el diccionario (D-141, §5.6 y DP-18); las cuatro copias de la skill `frontend-feature` (SKILL.md §6, los grupos, y L205: un catálogo ya no se abre solo con `catalogos.ver`).

**Orden de los commits.** (1) El menú, los textos y las etiquetas, con sus specs. (2) El acceso de las rutas de Catálogos, con sus specs. (3) La matriz y su renombre, con sus specs. (4) Los e2e. (5) Las imágenes de referencia, solas. (6) `DESIGN.md`. La documentación, con D-141, en su rama.

**Terminada cuando:** el CI de la rama está en verde (unitarias y e2e) y cada rol sembrado ve exactamente los grupos que le corresponden.

**Hecha el 22/11/2026, solo frontend** (integrada en `develop` con `a3c9a609`; documentación en `master` con esta versión del plan). Se aparta del plan en tres cosas:
- **31 opciones, no 30.** «Ingreso de equipos» llegó con la fase 3a y va en Monitoreo, después de Equipos; en la matriz, `stock.ingresar_equipamiento` cae en Monitoreo › Equipos, por la regla de `stock.*_equipamiento`.
- **`DESIGN.md`, revisión 100**, no 99: la 99 la tomó, el mismo día, la factura de equipos leída como su comprobante (frontend `8fb49548`).
- **Las pruebas.** Por la regla de verificación del responsable, se corrieron en local el build, las unitarias de los archivos tocados y `lint:styles`; los e2e se corrigieron leyéndolos, sin correrlos, y los corre el CI. Las imágenes de referencia no se regeneraron: las pruebas visuales del CI siguen en rojo hasta el lote que pida el responsable.
Lo ven así los roles sembrados: el administrador de empresa, siete grupos, todos menos Catálogos; Operaciones, Flota, Monitoreo, Compras, Herramientas y Administración; Contabilidad, los mismos y Finanzas, sin Ingreso de equipos; el asesor de ventas y el taller, Flota, Monitoreo (Equipos) y Administración (Empresas y Ubicaciones); el administrador de accesos, Administración y Seguridad; y solo el superadministrador, Catálogos.

---

### Fase 2 — Compra de equipos e historia del equipo

**Hecho el backend el 22/11/2026, con la parte de la fase 3a que no es pantalla** (rama `feature/phase-2-equipment-purchase`, commits `7519933f` a `2974ab18`, más `521ff5e4` y `c6318f0e`, integrados en `develop` con `6b4c6668`; documentación en `docs/phase-2-gps-purchase`, diccionario 1.81): los estados de compra y la historia del equipo, su ingreso a mano, su devolución y su regreso, y el aviso de antigüedad; la orden y la factura de equipos, con sus equipos por ingresar; su anulación y «Anular y copiar»; la nota «anulación de equipo»; el pago; cumplir con faltante; el reemplazo de un fallado; y las facturas con equipos por ingresar. Lo que quedaba —el costo de un equipo, sus acciones, su cadena de reemplazos y el frontend— se hizo el mismo día (precisiones 12 a 13 y el párrafo siguiente a la lista). D-283 y D-284 quedan construidas, y D-285 en todo lo que no es la SIM ni la API del fabricante. Se precisaron trece cosas que el plan no fijaba o que el código hizo distinto:

1. **Las migraciones cambiaron de número.** V148, el tipo de la orden y de la factura; V149, los estados de compra, la ubicación opcional y `replaced_equipment_id`, como estaba escrita; V150, la historia, como estaba escrita, con dos `CHECK` más —una fila nunca deja y alcanza el mismo estado, y su motivo nunca está en blanco— y `changed_by` con su llave a `users`, como `vehicle_status_history`; V151, `stock.ingresar_equipamiento` (era V153); V152, las líneas de equipo de las órdenes, y V153, las de las facturas, los disparadores del tipo y `ANULACION_EQUIPO` (las dos eran V151); y V154, `purchase_orders.fulfilment_reason` (era V152). La próxima libre es V155: las de las fases siguientes se renumeran al abrir cada rama (§5). Sin la SIM (Q30, Q31), la 3b tiene una sola, el completado.
2. **El tipo de la línea no se guarda** (regla 4.1: no se guarda lo que se calcula). V148 no crea `purchase_order_items.line_kind` ni `purchase_invoice_items.line_kind`, ni el disparador de compatibilidad `fill_purchase_line_kind()`, así que la fase 5 no tiene que retirarlo: una línea de vehículo nombra su unidad o su modelo, una de equipo su equipo o su modelo de equipamiento, y un concepto ninguno, y los `CHECK` por tipo se escriben sobre esas columnas (`purchase_order_items_what_ck`, `num_nonnulls(model_id, equipment_model_id, concept) = 1`; `purchase_invoice_items_what_ck`). `PurchaseLineKind` lo deriva cada línea (`getKind()`), con `isVehicle()`, `isEquipment()` e `isConcept()`; los 13 usos se decidieron como dice la tabla del backend, y solo `CreditedLines` cuenta lo que no es concepto. V148 suma `kind` a `keep_purchase_invoice_fiscal_data`, y V153, `equipment_id` y `equipment_model_id` a `keep_purchase_invoice_item_fiscal_data`.
3. **Los disparadores del tipo** son las tres funciones del plan, en dos migraciones: `enforce_purchase_order_item_kind()` en V152, al insertar una línea o cambiar su orden o lo que nombra; y en V153, `enforce_purchase_invoice_item_kind()`, solo al insertar, porque lo que nombra una línea de factura no cambia, y `enforce_purchase_invoice_order_kind()`, al insertar la factura o cambiar su orden, sin `kind`, que es fiscal. Las dos de V153 tienen su línea en `hand-over-database.sql`, como la de V152.
4. **Las facturas con equipos por ingresar tienen su propia lectura,** `GET /api/v1/purchase-invoices/pending-arrivals` (`PurchaseInvoicePendingArrivalsController`), con `stock.ingresar_equipamiento`, en vez de los filtros `kind=EQUIPOS` y `pendingArrival=true` de `GET /purchase-invoices`: quien ingresa puede no tener `compras.ver`, y la pantalla necesita los identificadores de los equipos por ingresar de cada modelo, que la lista de facturas no trae. Responde, sin página, las vigentes de equipos de las empresas donde actúa el permiso, de la más antigua a la más nueva; `equipment` responde sus equipos por ingresar (`fetchPendingArrivalDevices`, los equipos y no solo sus identificadores), y sus líneas y facturas se leen en una consulta (convenciones §13, fila 26). La lista de facturas no filtra por `kind`, aunque cada fila lo trae.
5. **Las inserciones de varias filas** son las del plan —los equipos, su historia y las líneas de una factura, con `INSERT … SELECT … FROM unnest(…)`—, y también los cambios de estado de un lote: la anulación de los equipos de una factura o de una nota escribe sus estados con un `UPDATE … FROM unnest(…)` y su historia con otro `INSERT`, después de bloquearlos en una sentencia. Las líneas de una nota de 500 líneas siguen por JPA, una a una: solo las líneas de factura, los equipos y su historia van por lote. `PurchaseInvoiceRegistrationStatementsTests` comprueba que Hibernate prepara las mismas sentencias con 1, 10 y 500 líneas; la prueba de tiempo con su meta medida contra `dev` no se escribió.
6. **El dígito verificador del IMEI** (`ImeiCheckDigit`) lo pasa todo IMEI nuevo o cambiado: en el ingreso y el reemplazo y, desde `c6318f0e`, en el alta y la edición a mano (`EQUIPMENT_IMEI_INVALID`). A diferencia de lo que pedía el plan para `updateIdentifiers`, la regla no está en el dominio sino en el servicio, y un IMEI guardado antes de la regla se conserva aunque no la pase, así esos equipos se siguen editando; nunca se revisa al leer (D-285).
7. **`equipment` depende de `iam::acl` desde esta fase,** no desde la 3b: la historia nombra a quien hizo cada cambio (`IamContextFacade.fetchAccountReferencesByIds`, por un `ExternalIamService` de `equipment`). Ya expone `fetchSystemAccountId`, que nadie usa hasta la tarea del fabricante. `iam` no depende de `equipment`, y `ModularityTests` lo comprueba; la lista de control (§7) decía que llegaba con la fase 3.
8. **La fachada de `equipment`** tiene los miembros que usó `procurement`: `fetchEquipmentModelReferenceByPublicId`, `…ByIds` y `…ByPublicIds`; `registerPurchasedDevices`; `fetchPurchasedDevicesByIds`, `fetchPurchasedDevicesByPublicIdsForUpdate` y `fetchPurchasedDevicesByIdsForUpdate`, que reemplazan a `fetchDevicesByIds` y `lockDevicesByIds`; `annulPendingDevices`, que dice cuántos anuló; `fetchCreditedRefusals`, nuevo, que revisa la nota antes de guardarla; `annulCreditedDevices`; y `fetchPendingArrivalDevices`. Los componentes que corren con `Propagation.MANDATORY` son `PurchasedDeviceRegistrations`, `PurchasedDeviceAnnulments` y `EquipmentSaving`, por donde guarda todo servicio que cambia un equipo; todos con su prueba de contrato en `EquipmentContextFacadePurchaseTests`. `fetchSeenDevice` y `fetchPurchaseRoots` llegaron con el costo (precisión 12), y `EquipmentModelReference` suma `requiresSerial` y `requiresTelematics` para las facturas con equipos por ingresar (`8672dc2d`).
9. **Lo pendiente de una orden de equipos** se cuenta con `OrderInvoicingPolicy.orderedEquipmentOf` y `equipmentDeviations`, una forma propia (`EquipmentDeviation`, sin año), en vez de sumar el tipo a la llave de `Group`; el aviso es el mismo `PURCHASEINVOICE_ORDER_DEVIATION_UNCONFIRMED`.
10. **La nota sobre equipos.** El tope de 500 vale cuando todas sus líneas son de equipo, lo que el servicio lee antes de bloquear y le pasa a `CreditNote`; con cualquier otra línea, 100. Anular una nota se rechaza con `CREDITNOTE_EQUIPMENT_ANNULLED` cuando devolvería una línea cuyo equipo está anulado, por la nota que sea —también la anulación de la factura que anuló sus equipos por ingresar—, y con `CREDITNOTE_LINE_RELINKED` cuando el equipo ya tiene otra línea vigente.
11. **Los códigos nuevos** son los siete de la tabla de la fase y `EQUIPMENT_REPLACED` de la 3a, más dos que el plan no nombraba: `EQUIPMENT_ARRIVAL_IDENTIFIERS_REQUIRED` —un tipo telemático que ingresa sin su serial o su IMEI— y `EQUIPMENT_ANNULLED_IMMUTABLE` —editar un equipo anulado—. Cada uno con sus mensajes en los dos bundles.
12. **Lo que se construyó después, el mismo día** (backend `77921dcd`, integrado en `develop` con `0afa5adf`): el costo de un equipo (`EquipmentPurchaseCostsController`, `GET /api/v1/equipments/{id}/purchase-cost`, con `stock.ver_costos`, `fetchSeenDevice` y `fetchPurchaseRoots`), calculado como el de la unidad (`UnitPurchaseCostCalculator`), con el estado de pago de la factura entera en lugar de la posición de pago de la unidad, y sin marca en el permiso; las acciones del equipo (`allowedActions`, `EquipmentAllowedActions`, en el orden `edit`, `changeStatus`, `returnToSupplier`, `comeBackRepaired`, `registerArrival`, `registerReplacement` y `viewPurchaseCost`), con `replacesEquipmentId` y `replacedByEquipmentId`; y la lista sin los reemplazados por omisión, con el filtro `RETURNED_TO_SUPPLIER` para verlos. Antes, en `563230d0` y `59bb4be1`, lo que las pantallas necesitaban: `kind` y `equipment` en las líneas de una nota, `kind` en las filas de Por pagar y de lo que se le debe a un proveedor, `GET /purchase-orders?kind=` y, de cada modelo de equipamiento, `brandIsActive` y `typeIsActive`. No se construyó el filtro de la lista de facturas por tipo. **El aviso de antigüedad se construyó antes** (`521ff5e4`): el recurso del equipo responde `daysPendingArrival` —los días enteros que un equipo por ingresar lleva esperando desde que se creó, nulo en otro estado— y `pendingArrivalOverdue` —si pasó de `equipment.pending-arrival-warning-days`, `EQUIPMENT_PENDING_ARRIVAL_WARNING_DAYS`, 30 por omisión—, calculados en cada lectura; pero la lista los marca y no filtra por ellos, y la variable está solo en la tabla del README y en `application.yaml`, no en `.env.example` ni en `.do/app.yaml` como pide §4 para toda variable nueva, porque tiene valor por omisión.
13. **La parte de la fase 3a** que no es pantalla: el reemplazo (`POST /api/v1/equipments/{id}/replacement`), con `EQUIPMENT_REPLACED` también en «volvió reparado» y la nota «anulación de equipo» que anula el extremo vivo de la cadena; el ingreso sin fabricante es el ingreso a mano completo de esta fase (`/arrival`), que usa la pantalla de escaneo. El costo que sigue la cadena (`fetchPurchaseRoots`) y el reemplazado fuera de la lista por omisión llegaron con `77921dcd` (precisión 12), y la pantalla «Ingreso de equipos», con su cámara y `_headers`, con el frontend (abajo).

**Hecho el frontend el 22/11/2026** (integrado en `develop` con `7619826e` y, el cierre del tramo —`d84cbcce`, `f50f6f44`, `c2774678` y `1f30d2ac`—, con `3067073c`; el backend del cierre, con `0afa5adf`): la orden de equipos y cumplir con su motivo; la factura de equipos, su ficha y «Anular y copiar»; la nota «Anulación de equipo»; el pago sin imputaciones; el inventario con los estados nuevos, la devolución, «Volvió reparado», el reemplazo, el historial, el aviso de antigüedad, el costo y la cadena; y «Ingreso de equipos». Se precisaron cinco cosas:

1. **La ficha del equipo sigue siendo un diálogo**, no la página `/equipment/inventory/:id` que pedía el plan, ni con una pestaña «Historia»: el diálogo muestra los datos técnicos, el historial de estados, «Reemplaza a» y «Reemplazado por» —que abren ese equipo en el mismo diálogo— y el costo de compra, con sus acciones en el encabezado. Elegir una acción cierra el diálogo y la página abre la suya, así ningún formulario se abre sobre otro diálogo (regla 4.9 de `frontend-rules`), que era la razón del plan para la página. No hubo revisión de `DESIGN.md` para una ficha como página.
2. **Las acciones de un equipo salen de su `allowedActions`**, en el orden del backend: la vista ya no las decide por el estado ni por el alcance en todas las marcas, y «Nuevo equipamiento» conserva su propia revisión. «Registrar ingreso» es un enlace a «Ingreso de equipos», porque el equipo no trae su factura.
3. **«Ingreso de equipos» va en Logística**, justo después de Inventario de equipamiento, y no en Monitoreo, que espera la fase 1: el menú tiene 31 opciones. La ubicación se recuerda por empresa durante la sesión, en memoria. Lee lo que pide el tipo de cada modelo (`requiresSerial`, `requiresTelematics`): de un telemático, el IMEI con su dígito verificador, el serial y el ICCID opcional; de uno con serie, el serial solo; de uno sin identificadores, solo «Ingresar». Manda las lecturas de a una y, si otra persona tomó el equipo, pasa una vez al siguiente.
4. **La cámara.** «Usar cámara» (`mm-barcode-camera`) abre la cámara trasera y lee con `BarcodeDetector` cuando el navegador lo tiene para los formatos de una etiqueta; si no, carga ZXing —`@zxing/browser` 0.2.1 con `@zxing/library` 0.23.0, con su versión fija— solo al abrirla: QR, Data Matrix y los códigos de una dimensión, como Code 128. El trozo perezoso `zxing-barcode-decoder` pesa 484 kB sin comprimir y 92 kB transferidos, con su presupuesto de tipo `bundle` en `angular.json` —aviso a 520 kB, error a 600 kB—, y el paquete inicial sigue en 485 kB. `public/_headers` pasa a `Permissions-Policy: camera=(self), microphone=(), geolocation=()`, una sola cabecera para la aplicación de una página; el e2e no la comprueba, porque el servidor de desarrollo no aplica `_headers`. La cámara deja de leer al cerrarse, al irse la zona de lectura y al salir de la página, y dice en su lugar un permiso negado, la falta de cámara, una cámara ocupada o un lector que no cargó, con «Reintentar».
5. **Las pruebas**: las specs de cada pantalla, del store de lecturas, de `imei.ts`, de la cámara y de la lectura de varios códigos en un cuadro; y el e2e `equipment-arrivals.spec.ts`, con el lector simulado y el simulacro de la API que aplica las reglas del backend. La cámara no se prueba en e2e. Se regeneraron las imágenes de la matriz de permisos, la ficha del rol, el formulario y la ficha de la factura, sus notas y la orden en el teléfono; ninguna más.

**Objetivo.** Una orden de equipos; una factura de equipos que crea cada equipo «Por ingresar»; la nota «anulación de equipo» que lo anula; anular la factura o «Anular y copiar» sin duplicar equipos; el pago de la factura como un todo; el costo de cada equipo; cumplir con faltante con su motivo; y la historia de estados de cada equipo, en su propia ficha. Un ingreso a mano completo deja la fase usable sola, antes del escaneo.

**Lo que la rige.** C1 a C4, C6 a C10 y C13; Q12, Q14 a Q16, Q18, Q19 y Q24; D-23, D-153, D-160, D-161, D-173 (enmienda), D-187 (precisión), D-191, D-193, D-211, D-214, D-248, D-254, D-280; reglas 1.4, 4.1, 4.3 y 6.4. Todavía no hay áreas: lo que se registra en esta fase nace general, y la fase 7 le da su área con «Asignar área».

**Migraciones.**

**V148__give_purchase_lines_and_orders_their_kind.sql** (`Context: procurement`). Abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos.

- `purchase_order_items.line_kind VARCHAR(10)`: se llena con `VEHICULO` donde hay `model_id` y `CONCEPTO` en el resto, y queda `NOT NULL` con `CHECK (line_kind IN ('VEHICULO', 'EQUIPO', 'CONCEPTO'))`.
- `purchase_orders.kind VARCHAR(10) NOT NULL DEFAULT 'VEHICULOS'`, con `CHECK (kind IN ('VEHICULOS', 'EQUIPOS'))` (Q24). Las órdenes de hoy son todas de vehículos.
- `purchase_invoice_items.line_kind`: se llena con `VEHICULO` donde hay `vehicle_id` y `CONCEPTO` en el resto, y queda igual. `keep_purchase_invoice_item_fiscal_data` compara una lista cerrada de columnas (V103 L215-221), así que el relleno de una columna nueva no la dispara: basta con rellenar antes del `CREATE OR REPLACE` que suma `line_kind` a la lista, sin desactivar nada.
- `purchase_invoices.kind VARCHAR(10) NOT NULL DEFAULT 'VEHICULOS'`, con `CHECK (kind IN ('VEHICULOS', 'EQUIPOS'))`.
- `CREATE OR REPLACE` de `keep_purchase_invoice_fiscal_data` (desde V94 L183-200, suma `kind`) y de `keep_purchase_invoice_item_fiscal_data` (desde V103 L209-221, suma `line_kind`).
- **Compatibilidad hacia atrás:** un disparador `fill_purchase_line_kind()`, `BEFORE INSERT` en las dos tablas de líneas, llena `line_kind` desde la línea cuando llega nulo, para que el código anterior siga insertando durante el despliegue y ante una vuelta atrás. Lo retira la fase 5.
- Las funciones nuevas, con su línea en `scripts/database/hand-over-database.sql`.

**V149__add_the_purchase_statuses_of_the_equipments.sql** (`Context: equipment`).

- El `CHECK` de `status` suma `PENDING_ARRIVAL`, `RETURNED_TO_SUPPLIER` y `ANNULLED`.
- `location_id` deja de ser `NOT NULL`, con `fleet_equipments_location_ck CHECK (location_id IS NOT NULL OR status IN ('PENDING_ARRIVAL', 'ANNULLED', 'INSTALLED'))`. Un equipo instalado está en su unidad: la vinculación de lo ya instalado (fase 4) registra equipos que nunca tuvieron una ubicación, y el retiro le da una (fase 4).
- `fleet_equipments_id_company_uq UNIQUE (id, company_id)`, destino de las llaves de las líneas de factura, de la historia y del reemplazo.
- `fleet_equipments.replaced_equipment_id BIGINT`, con la llave compuesta `(replaced_equipment_id, company_id) → fleet_equipments (id, company_id)`, único (un equipo se reemplaza una sola vez) y distinto de sí mismo (Q13). Lo usa la fase 3a; va aquí para que el primer tramo no renumere las migraciones (Q29).
- Los índices únicos del serial y del IMEI se vuelven a crear excluyendo `DISCARDED` y `ANNULLED`, **con los mismos nombres** (`idx_fleet_equipments_serial_unique`, `idx_fleet_equipments_imei_unique`), porque `EquipmentRepositoryImpl` L35-37 los traduce a errores. El del ICCID no cambia: el ICCID se queda en el equipo (Q30, Q31). `RETURNED_TO_SUPPLIER` conserva sus identificadores, porque el equipo puede volver (Q16).

**V150__create_the_equipment_status_history.sql** (`Context: equipment`).

```
equipment_status_history
  id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
  public_id    UUID        NOT NULL DEFAULT uuidv7()
  company_id   BIGINT      NOT NULL REFERENCES companies (id)
  equipment_id BIGINT      NOT NULL                  -- (equipment_id, company_id) → fleet_equipments (id, company_id)
  from_status  VARCHAR(20)                           -- vacío en la fila de apertura
  to_status    VARCHAR(20) NOT NULL
  reason       TEXT                                  -- el motivo, si el camino lo pide
  changed_at   TIMESTAMPTZ NOT NULL DEFAULT now()
  changed_by   BIGINT      NOT NULL
  índice (equipment_id, changed_at)
```

- De solo inserción: la cuenta de la aplicación tiene `SELECT` e `INSERT`, con su línea en `hand-over-database.sql`. RLS de empresa con las políticas de V29. No se exporta.
- Cada equipo que ya existe recibe una fila de apertura con su estado y su `status_reason` de hoy, firmada con su propio autor (`coalesce(updated_by, created_by)`) y fechada en `updated_at`: así la migración no lee `users`, que es de `iam`.
- `fleet_equipments.status_reason` sigue siendo el motivo vigente; la historia guarda todos (regla 4.3).

**V151__add_equipment_lines_to_purchases.sql** (`Context: procurement`).

- `purchase_order_items.equipment_model_id BIGINT REFERENCES equipment_models (id)`. El `CHECK` de modelo o concepto (V69 L86-87) pasa a uno por tipo:
  - `VEHICULO`: con modelo de vehículo, sin modelo de equipo ni concepto;
  - `EQUIPO`: con modelo de equipo, sin modelo de vehículo, sin año ni concepto;
  - `CONCEPTO`: con concepto y sin modelos.

  El `CHECK` del año (V82) queda solo para `VEHICULO`. Función nueva `enforce_purchase_order_item_kind()`: una línea `VEHICULO` va solo en una orden `VEHICULOS`, una `EQUIPO` solo en una `EQUIPOS`; un concepto, en las dos.
- `purchase_invoice_items.equipment_id BIGINT`, con `purchase_invoice_items_equipment_company_fk (equipment_id, company_id) → fleet_equipments (id, company_id)`, y `equipment_model_id BIGINT REFERENCES equipment_models (id)`.
  - El `CHECK` de vehículo o concepto (V94 L110-114) pasa a uno por tipo: una línea `EQUIPO` tiene equipo y modelo de equipo, sin vehículo, modelo de vehículo, año, marca ni ISC.
  - Una línea vigente por equipo: índice único parcial sobre `equipment_id` `WHERE status = 'VIGENTE' AND equipment_id IS NOT NULL`.
  - `keep_purchase_invoice_item_fiscal_data` suma `equipment_id` y `equipment_model_id`.
- Función nueva `enforce_purchase_invoice_item_kind()`: una línea `VEHICULO` va solo en una factura `VEHICULOS` y una `EQUIPO` solo en una `EQUIPOS` (C2). Y `enforce_purchase_invoice_order_kind()`, `BEFORE INSERT OR UPDATE OF purchase_order_id, kind`: una factura con orden tiene el tipo de su orden. Los disparadores del ISC ya saltan las líneas sin vehículo.
- El `CHECK` de `credit_notes.reason` (V103 L67) suma `ANULACION_EQUIPO`.
- Las funciones nuevas suman su línea en `hand-over-database.sql`.

**V152__record_why_an_order_was_fulfilled_short.sql** (`Context: procurement`).

- `purchase_orders.fulfilment_reason VARCHAR(200)`, con `CHECK (fulfilment_reason IS NULL OR (status = 'CUMPLIDA' AND length(btrim(fulfilment_reason)) > 0))`. `CUMPLIDA` es final, así que el `CHECK` no estorba a ningún camino.

**V153__seed_iam_permission_of_the_equipment_arrival.sql** (`Context: iam`). Plantilla: V133.

- `stock.ingresar_equipamiento`, categoría `editar`, marcado en `SUPERADMIN`, `ADMIN_EMPRESA` y `OPERACIONES` (Q19).

**Backend.**

*`equipment`*

- **Los estados:** `EquipmentStatus` suma `PENDING_ARRIVAL`, `RETURNED_TO_SUPPLIER` y `ANNULLED`. Los `switch` no tienen `default`, así que el compilador señala cada uno que falta.
- **El agregado** (`Equipment`). Cada camino deja su fila en la historia, en la misma transacción: el agregado acumula sus cambios de estado y el repositorio los inserta, como `vehicle_status_history` en `fleet`.

  | Operación | Desde | Hasta | Qué valida |
  |---|---|---|---|
  | `purchased(empresa, proveedor, tipo, modelo)` | — | `PENDING_ARRIVAL` | Sin ubicación ni identificadores |
  | `registerArrival(ubicación, identificadores)` | `PENDING_ARRIVAL` | `AVAILABLE` | EQP-R14 con el dígito verificador del IMEI; el ICCID, en el equipo |
  | `returnToSupplier(motivo)` | `AVAILABLE`, `MAINTENANCE` | `RETURNED_TO_SUPPLIER` | El motivo es obligatorio |
  | `comeBackRepaired()` | `RETURNED_TO_SUPPLIER` | `AVAILABLE` | Q16; la fase 3 lo rechaza si otro equipo ya lo reemplazó |
  | `annulPending()` | `PENDING_ARRIVAL` | `ANNULLED` | La pide la anulación de su factura o la nota «anulación de la factura» (Q15) |
  | `annulCredited()` | `PENDING_ARRIVAL`, `AVAILABLE`, `MAINTENANCE`, `RETURNED_TO_SUPPLIER` | `ANNULLED` | La pide la nota «anulación de equipo»; la fase 3 suma `PENDING_COMPLETION` y `UNDER_REVIEW` |

  El constructor de reconstitución acepta la ubicación nula, y `EquipmentAuditTests` sigue con el constructor de quince argumentos más uno secundario.
- **El dígito verificador:** `ImeiCheckDigit`, en el dominio, valida en los caminos que escriben un IMEI (alta, ingreso, edición), nunca en el constructor ni al leer: las pruebas que generan `digits(15)` al azar (`EquipmentAuditTests` L57-63 y cinco más) se volverían aleatorias. `updateIdentifiers` (L141-147) valida el IMEI solo si cambia. El rechazo es `EQUIPMENT_IMEI_INVALID`, el código que ya existe.
- **Las reglas que se reescriben:**
  - EQP-R13: el alta a mano sigue pidiendo ubicación, y el proveedor sigue siendo opcional (`EquipmentCommandServiceImpl` L96-104); el equipo comprado recibe la ubicación al ingresar.
  - EQP-R14: lo que exige un tipo telemático para estar Disponible.
  - EQP-R15: los caminos nuevos.
  - EQP-R16: qué cambia en cada estado.
  - Una regla nueva: todo cambio de estado deja su fila en la historia.
- **El aviso de antigüedad (C7):** el recurso suma `daysPendingArrival` (los días desde que nació, solo por ingresar), y la lista marca y filtra los que pasan de `EQUIPMENT_PENDING_ARRIVAL_WARNING_DAYS` (30 por omisión, a confirmar con el responsable).
- **La lectura:** `EquipmentView.locationPublicId` acepta el nulo, y también `EquipmentPersistenceEntity` (L33, hoy `nullable = false`), `EquipmentResource` (L23) y `EquipmentViewFactory`, que deja de exigir la ubicación (L92-93) y no mete un nulo en el conjunto de ids (L67). El recurso suma `allowedActions` (`registerArrival`, `edit`, `changeStatus`, `returnToSupplier`, `comeBackRepaired`) y la lista filtra por los estados nuevos.
- **La fachada** (`EquipmentContextFacade`). Las operaciones nuevas corren con `Propagation.MANDATORY`, en componentes de `application/internal/commandservices` que no se llaman `*CommandServiceImpl`, como `PurchasedUnitRegistrations`:

  | Operación | Para | Qué |
  |---|---|---|
  | `registerPurchasedDevices(empresa, proveedor, líneas)` | `procurement` | Por cada línea, crea un equipo por ingresar de su modelo, o vuelve a usar el equipo que la línea nombra (`relinkEquipmentId`) si es de la misma empresa, proveedor y modelo y no está anulado; `procurement` comprueba que ese equipo no tenga otra línea vigente. Devuelve los ids. Exige el modelo y su tipo activos |
  | `annulPendingDevices(ids)` | `procurement` | Anula los que siguen por ingresar y deja los demás como están; dice qué hizo con cada uno |
  | `annulCreditedDevices(ids)` | `procurement` | Anula todos, o ninguno: si alguno no se puede anular (instalado o en otro estado), devuelve la lista |
  | `lockDevicesByIds(ids)` | `procurement` | `FOR NO KEY UPDATE`, en orden de id |
  | `fetchDevicesByIds(ids)` | `procurement` | Estado, modelo, tipo, serial, IMEI y ubicación |
  | `fetchSeenDevice(publicId)` | `procurement` | El equipo si la sesión lo ve con `stock.ver`, con su empresa |
  | `fetchEquipmentModelsByIds` / `ByPublicIds` | `procurement` | Nombre, marca, tipo y si está activo |

  Cada una trae su prueba de contrato. Ninguna necesita la cuenta `SYSTEM`: todas corren dentro del pedido de una persona.
- **Las rutas nuevas:**
  - `POST /api/v1/equipments/{id}/arrival` (`stock.ingresar_equipamiento`): ubicación, serial, IMEI e ICCID; el ingreso a mano completo.
  - `POST /api/v1/equipments/{id}/return-to-supplier` (`stock.cambiar_estado_equipamiento`): con el motivo.
  - `PATCH /api/v1/equipments/{id}/status` admite el regreso del reparado.
  - `GET /api/v1/equipments/{id}/status-history` (`stock.ver`): la historia, de la más nueva a la más vieja, con quién y por qué.

*`procurement`*

- **El tipo de la línea:** `LineKind` en `PurchaseOrderItem` y en `PurchaseInvoiceItem`, con `isVehicle()` e `isEquipment()` junto a `isConcept()`, que pasa a ser «tipo concepto». Hoy `!isConcept()` significa «línea de vehículo» en 13 usos de 9 archivos, y cada uno se decide:

  | Uso | Con equipos pregunta |
  |---|---|
  | `PurchaseInvoice.checkUnits` L140 (una unidad no se repite) | `isVehicle()`: con `!isConcept()`, dos equipos (vehículo nulo) se tomarían por la misma unidad |
  | `PurchaseInvoice` L315; `InvoiceUnitCoverages` L97, L112; `SupplierPaymentAllocations` L220; `CreditNoteViewFactory` L228 | `isVehicle()` |
  | `SupplierPaymentApplications` L330 (las marcas del pago) | `isVehicle()`: un equipo no tiene marca |
  | `CreditNoteLineEffects` L71 (lo que se le pide a `fleet`) | `isVehicle()`; las líneas de equipo van a la fachada de `equipment` |
  | `CreditNoteAnnulments` L118, L132 | `isVehicle()`; anular una nota que anuló equipos se rechaza |
  | `CreditedLines` L85 (lo que falta para anular la factura entera) | `!isConcept()`: vehículo **y** equipo |

  El SQL que filtra `vehicle_id IS NOT NULL` (`JdbcPayableFactsReadRepository` L218, L227, L253; `PurchaseInvoiceJpaRepository` L45, L58) ya es correcto y no cambia.
- **Las órdenes:**
  - el tipo se elige al crear y queda fijo; una línea de otro tipo responde `PURCHASEORDER_KIND_MISMATCH` (Q24);
  - un `ExternalEquipmentService` nuevo en `application/internal/outboundservices/acl`, con nombre de bean explícito, traduce los modelos de la fachada de `equipment`;
  - `PurchaseOrderLines` resuelve el modelo de equipo; la descripción del PDF (`PurchaseOrderLineDescription` L21-33) dice «Equipo: marca y modelo»;
  - `OrderInvoicingPolicy.Group` (L31) suma el tipo a la llave, porque los ids de modelo de vehículo y de equipo vienen de tablas distintas y pueden coincidir; `Deviation` (L56, con `int year`) tiene su forma de equipo, sin año;
  - una consulta nueva cuenta las líneas vigentes de equipo de una orden por modelo, junto a `countCurrentUnitsOfOrder` (`PurchaseInvoiceJpaRepository` L58-60); `PurchaseOrderInvoicingReader` (L33-50) responde también por una orden de equipos;
  - vincular una orden después de registrar (`PurchaseInvoiceOrderLinks.deviations`, L71-83) compara también modelos de equipo, y exige el mismo tipo.
- **Cumplir con faltante (D-284):**
  - `FulfillPurchaseOrderCommand(orderId, motivo)`; `POST /purchase-orders/{id}/fulfillment` acepta un cuerpo opcional `{reason}` (sin cuerpo sigue valiendo);
  - el pendiente se calcula bajo el candado de la orden; si hay pendiente y el motivo está vacío, `PURCHASEORDER_SHORTFALL_REASON_REQUIRED`;
  - en una orden solo de conceptos el pendiente no se calcula y el motivo es opcional;
  - la vista suma `shortfall` (el pendiente, nulo si no se calcula) y `fulfilmentReason` (Q18).
- **La factura de equipos (D-283):**
  - el comando suma `kind`; con orden, el tipo es el de la orden (`PURCHASEINVOICE_KIND_MISMATCH`). Las líneas de equipo llegan una por equipo, con su modelo, su monto y, opcional, el equipo que vuelven a usar; el formulario abre la cantidad;
  - `PurchaseInvoice.MAX_LINES` pasa a depender del tipo (100 de vehículos, 500 de equipos), igual que la validación del recurso (L38), que deja de ser un `@Size` fijo. `InvoiceLineImportPolicy.MAX_ROWS`, `StockRowGroups` (L60) y `PurchaseInvoiceVinChecks.MAX_VINS` (L42) siguen en 100: son solo de vehículos;
  - el flujo copia el de las unidades:
    1. los controles;
    2. los modelos activos;
    3. las desviaciones de la orden, por modelo de equipo;
    4. el permiso: toda la empresa y todas las marcas (`actsOnWholeCompanyEveryBrand`, Q14); el área llega en la fase 7;
    5. `PurchaseInvoiceDevices.register`, que llama a `registerPurchasedDevices` con el proveedor de la factura;
    6. las líneas con su `equipment_id`;
    7. guardar;
    8. los pagos de la orden pasan a la factura (`OrderPaymentTransfers` ya pasa sin unidades, L117-126).
  - **La inserción de 500.** No hay `hibernate.jdbc.batch_size` y todo id es `IDENTITY`, que Hibernate nunca agrupa; y `bump_data_version` corre una vez por sentencia (V94 L276-281). Los equipos y las líneas se insertan con una sola sentencia de varias filas cada uno (`INSERT … SELECT … FROM unnest(…) RETURNING id`, con `JdbcTemplate`), dentro de los adaptadores de sus repositorios y sin cambiar los agregados; la factura y lo demás siguen por JPA. PRC-R65 conserva su regla —las consultas que no son inserciones no crecen con las líneas— y la meta de tiempo se fija al abrir la fase, midiendo contra `dev` (la latencia real de App Platform a su base), con el corte de 100 segundos de Cloudflare como límite duro;
  - la ficha de la factura lee el estado y los identificadores de sus equipos por la fachada;
  - **anular** una factura de equipos sigue las reglas de hoy y además anula los equipos que siguen por ingresar (`annulPendingDevices`, Q15); la nota «anulación de la factura» sobre una factura de equipos hace lo mismo;
  - **«Anular y copiar»** (D-211, D-214): la copia lleva, en cada línea, el equipo de la línea anulada. Los recibidos vuelven a tener una línea vigente; los por ingresar, que la anulación anuló, nacen otra vez. Ningún equipo se duplica. Un equipo que no se puede volver a usar responde `PURCHASEINVOICE_EQUIPMENT_NOT_RELINKABLE`;
  - modificar y anular una factura de equipos exigen una asignación de todas las marcas (`PurchaseInvoiceAccessPolicy` L25-44, Q14).
- **La nota «anulación de equipo»** (`CreditNoteReason` L14-18):
  - el motivo nuevo exige llevar sus líneas a cero, como la anulación de unidad, y luego llama a `annulCreditedDevices`;
  - si un equipo no se puede anular (instalado, o en un estado que no lo admite), la nota entera se rechaza y nombra cada equipo con su estado (`CREDITNOTE_EQUIPMENT_NOT_ANNULLABLE`): un instalado se retira antes, distinto de la unidad, que queda retenida;
  - el motivo tiene que corresponder al tipo de la factura (`CREDITNOTE_REASON_KIND_MISMATCH`);
  - anular una nota que anuló equipos se rechaza (`CREDITNOTE_EQUIPMENT_ANNULLED`), como D-193;
  - una nota sobre una factura de equipos admite hasta 500 líneas: `CreditNote.MAX_LINES` no conoce el tipo de la factura, así que el servicio le pasa el límite, y `RegisterCreditNoteResource` (L31, hoy `@Size(max = 100)`) sube a 500 con el control por tipo en el servicio;
  - escribirla exige una asignación de todas las marcas (`CreditNoteAccessPolicy.writes` L33-36, Q14).
- **El pago de una factura de equipos (Q12).** No hay migración ni política nueva: la base solo revisa una imputación cuando se inserta (V107 L207-225); `Spreads.of` no devuelve nada sin unidades (`InvoiceUnitCoverages` L67-78); y `AmountOwedPolicy.toBalance` (L112-117) ya pasa el exceso a saldo por factura. Solo cambia:
  - la regla del canal concesionario `allocationRequired` (`SupplierPaymentAllocations` L77-81) exime las facturas de equipos;
  - las líneas de equipo quedan fuera de los caminos de unidades (`isVehicle()`, arriba);
  - un pago que incluye una factura de equipos exige una asignación de todas las marcas (Q14).
- **El costo de un equipo:**
  - lo sirve un controlador nuevo de `procurement`, `EquipmentPurchaseCostsController`, en `GET /api/v1/equipments/{id}/purchase-cost` con `stock.ver_costos`. No choca con `EquipmentsController`, que mapea `""`, `/{id}`, `/{id}/status` y las rutas nuevas de esta fase;
  - lee el equipo con `fetchSeenDevice` (404 si la sesión no lo ve) y exige `stock.ver_costos` en su empresa (403). El área llega en la fase 7;
  - el costo es el monto de la línea menos sus notas, con la factura, las notas y los pagos de la factura;
  - el cálculo es el de la unidad (`UnitPurchaseCostCalculator`).

**API y errores.**

| Código | Estado | Cuándo |
|---|---|---|
| `PURCHASEORDER_SHORTFALL_REASON_REQUIRED` | 422 | Cumplir con pendiente sin motivo |
| `PURCHASEORDER_KIND_MISMATCH` | 422 | Una línea de otro tipo que su orden |
| `PURCHASEINVOICE_KIND_MISMATCH` | 422 | Una línea de otro tipo que su factura, o una orden de otro tipo |
| `PURCHASEINVOICE_EQUIPMENT_NOT_RELINKABLE` | 422 | El equipo que nombra una línea es de otra empresa, proveedor o modelo, está anulado o ya tiene una línea vigente |
| `CREDITNOTE_REASON_KIND_MISMATCH` | 422 | «Anulación de unidad» sobre equipos, o «anulación de equipo» sobre vehículos |
| `CREDITNOTE_EQUIPMENT_NOT_ANNULLABLE` | 422 | La nota anularía un equipo instalado o en un estado que no lo admite; nombra cada uno |
| `CREDITNOTE_EQUIPMENT_ANNULLED` | 422 | Anular una nota que anuló equipos |

Se reutilizan `EQUIPMENT_IMEI_INVALID`, `EQUIPMENT_STATUS_TRANSITION_INVALID`, `EQUIPMENT_MODEL_INACTIVE` y `EQUIPMENT_TYPE_INACTIVE`. Cada código con sus mensajes en los dos bundles y su lugar en la OpenAPI.

**Frontend.**

- **La orden** (`purchase-order-form.ts`):
  - el tipo (Vehículos o Equipos) se elige al empezar y queda fijo con la primera línea;
  - `LineKind` (L117) suma `'equipment'`, con modelo de equipo, cantidad y precio (`LineControls` L119-130, `createLineGroup` L1482-1520, envío L1222-1233, descripción del html L869-949, botones L1010-1027);
  - `EquipmentFacade` expone los modelos con su propio estado de carga, no el del catálogo (`equipment-inventory-view.ts` L75-81 ya lee esas señales).
- **Cumplir** (`fulfill()` L1342-1347): con pendiente, un diálogo pide el motivo, copiado de `annul-purchase-invoice-dialog.ts`; la orden muestra «Cumplida con faltante: N» y el motivo.
- **La factura:**
  - el tipo se elige al empezar y queda fijo con la primera línea; con orden, es el de la orden;
  - las líneas de equipo se cargan por modelo, cantidad y monto unitario, se muestran agrupadas por modelo con su cuenta y se envían una por equipo;
  - el límite es de 500 (`purchase-invoice-lines.ts` `MAX_INVOICE_LINES` L81 pasa a depender del tipo); la importación pegada sigue solo para vehículos;
  - «Anular y copiar» lleva los equipos de la factura anulada;
  - la ficha muestra el estado y el IMEI de cada equipo, y la lista filtra por tipo.
- **La nota:** elegir equipos de una factura de equipos, con el motivo «Anulación de equipo»; hasta 500 líneas (`credit-note-lines.ts` L40).
- **El pago:** una factura de equipos no muestra el panel de unidades (`payment-document-panel`) y se aplica por monto.
- **El inventario de equipos:**
  - los estados nuevos, con sus tonos y textos (`equipment-status.ts` L4-11, `equipment-status-tone.ts`, `equipment-status-dialog.ts` L61-65, `mocks/equipments.ts`);
  - las acciones de cada fila salen de `allowedActions` del backend (hoy se calculan en `equipment-inventory-view.ts` L147-167);
  - la columna y el filtro de los días por ingresar, con la marca del aviso;
  - **la ficha del equipo pasa a ser una página**, `/equipment/inventory/:id`. Hoy es un diálogo (`EquipmentDetailsDialog`, `equipment-inventory-view.ts` L258), y abrir desde él un diálogo de formulario rompe la regla 4.9 de `frontend-rules`. La página muestra sus datos, su factura si se lee y su costo con `stock.ver_costos`, y una pestaña «Historia» con sus estados; desde ella se abren «Registrar ingreso», «Devolver al proveedor» y «Volvió reparado»;
  - `F/equipment/domain/model/imei.ts`: el dígito verificador, que usa también la fase 3;
  - `E/screens.ts` suma el inventario y la ficha.
- **`DESIGN.md`:** una revisión para la ficha del equipo como página.

**Pruebas.**

- Backend, nuevas:
  - el dominio: los caminos de `Equipment`, su fila de historia en cada uno, y el dígito verificador;
  - el tipo de la orden y de la factura, y sus límites;
  - la migración del tipo de línea y de orden (el relleno sin desactivar el disparador fiscal, y el disparador de compatibilidad), la de la historia (la fila de apertura) y los disparadores nuevos;
  - una línea vigente por equipo;
  - los 13 usos de `isVehicle()`/`isEquipment()`/`isConcept()`, con una factura de varias líneas de equipo;
  - **la factura de 500 equipos:** su conteo de sentencias (las de varias filas cuentan una) y una prueba de tiempo con la meta medida al abrir la fase;
  - «Anular y copiar» sin equipos duplicados, y la nota «anulación de la factura» que anula los por ingresar;
  - la nota sobre equipos; el pago a nivel de factura, con un proveedor del canal concesionario; el costo de un equipo; cumplir con faltante; el aviso de antigüedad;
  - el contrato de cada operación nueva de la fachada.
- Backend, que cambian:
  - `EquipmentTests`, `EquipmentCommandServiceImplTests`, `EquipmentsIntegrationTests`, `EquipmentsControllerTests`, `EquipmentQueryServiceImplTests`, `EquipmentAuditTests`, `EquipmentMigrationTests`, `EquipmentRowsAreNeverDeletedTests`;
  - `RowLevelSecurityMigrationTests` (la historia en `COMPANY_TABLES` y su fila en el accesorio; el accesorio de L257, con la ubicación ahora opcional);
  - las de órdenes, `OrderInvoicingPolicy`, notas y pagos, `PayablesIntegrationTests`;
  - `PermissionsMigrationTests`, `PermissionCatalogConsistencyTests` y `HandOverDatabaseScriptTests`.
- Frontend: las specs de los formularios, del inventario, de la ficha, del diálogo de ingreso y de `imei.ts`; e2e `purchase-orders`, `invoice-form`, `credit-notes`, `supplier-payments` y el inventario de equipos; las imágenes de lo que cambia.
- La suite completa del backend, en local.

**Documentación.**

- Diccionario:
  - §7.5: los modelos de equipamiento en las órdenes;
  - §9.2: el tipo de la orden y de la línea, la línea de equipo, `fulfilment_reason` y su regla;
  - §9.3: el tipo de factura, el límite de 500, la anulación y «Anular y copiar»;
  - §9.4: el tipo de línea y `equipment_id`;
  - §9.5: «anulación de equipo» y la nota «anulación de la factura» sobre equipos;
  - §9.6: el valor de un equipo;
  - §10: el pago de una factura de equipos;
  - §11.4: los estados del equipo, la ubicación opcional, el aviso de antigüedad, la tabla `equipment_status_history` y el costo de la instalación, que es solo de mano de obra;
  - D-283 y D-284 pasan a cerradas, con las precisiones de D-160, D-173, D-187 y D-254.
- `contextos/procurement.md` y `equipment.md` (EQP-R13 a R16 reescritas, la regla de la historia), las convenciones de la API, `procesos-y-pantallas` (P4, P5 y un proceso nuevo, «Compra e ingreso de equipos»), y la variable del aviso en el README, `.env.example` y `.do/app.yaml`.

**Orden de los commits.**

- Backend:
  1. V148 y el tipo de línea en todo `procurement` (`isVehicle()`, `isEquipment()` y los 13 usos), sin cambiar comportamiento.
  2. V149, V150, los estados de compra, la historia y la fachada de `equipment`.
  3. V151 y las líneas de equipo en órdenes, de un solo tipo.
  4. La factura de equipos, su inserción de varias filas y «Anular y copiar».
  5. La nota.
  6. El pago.
  7. El costo del equipo.
  8. V152 y cumplir con faltante.
  9. V153, el ingreso a mano, la devolución y la ruta de la historia.
- Frontend:
  1. Los estados, el inventario y la ficha como página.
  2. La orden.
  3. Cumplir.
  4. La factura.
  5. La nota.
  6. El pago.
  7. Los e2e y las imágenes.

**Terminada cuando:** en `dev` se recorre de punta a punta:
1. una orden de 100 GPS;
2. una factura de 98, con 98 equipos por ingresar y la orden con 2 pendientes;
3. su ingreso a mano;
4. una factura de 100 con 98 recibidos, el aviso de antigüedad en los 2 que faltan y una nota que los anula;
5. «Anular y copiar» de una factura con equipos recibidos, sin duplicarlos;
6. cumplir con motivo;
7. el pago;
8. el costo de un equipo;
9. la historia de cada equipo con todos sus cambios.

La suite completa y el CI están en verde.

---

### Fase 3 — Ingreso por escaneo, reemplazo y API del fabricante (3a y 3b)

**Hecha la fase 3a el 22/11/2026, con la fase 2:** el reemplazo de un fallado y las facturas con equipos por ingresar, en una lectura propia y no como filtros de la lista de facturas (fase 2, precisiones 4 y 13); el costo que sigue la cadena del reemplazo (precisión 12); y la pantalla «Ingreso de equipos», con el lector, la cámara y `_headers` (fase 2, el párrafo del frontend, precisiones 3 y 4). La 3b tiene una sola migración, el completado, que se numera al abrir su rama (§5), y su adaptador espera Q21.

**La SIM, desde Q30 y Q31 (versión 3.1).** La SIM es un dato de su GPS toda su vida: llega con él, se reemplaza con él y su línea la maneja el proveedor (D-285). Salen de esta fase `sim_cards` y `sim_card_assignments` con sus `EXCLUDE`, el traslado de los ICCID, las rutas `/sim-cards` y `/equipments/{id}/sim-assignment`, los permisos `stock.*_sims`, la pantalla «SIM» con su opción del menú y los códigos `SIMCARD_*` y `EQUIPMENT_SIM_ALREADY_ASSIGNED`. El ICCID y el IMSI siguen en `fleet_equipments`, donde los escribe el ingreso y donde los completa la API del fabricante, con su índice único del ICCID. Se define en la 3b si el número de la línea (MSISDN) y el operador se guardan como datos del GPS.

**Objetivo.** Ingresar los equipos escaneando su QR, modelo por modelo; completar sus datos con la API del fabricante en segundo plano o, sin fabricante, en el mismo escaneo; resolver lo que no cuadra; y registrar el reemplazo de un fallado. La SIM es un dato del GPS (Q30, Q31).

**Lo que la rige.** C5, C10, C11, C12; Q13, Q16, Q17, Q19 a Q22, Q25, Q28 a Q31; D-42 (lo automático firma como `SYSTEM`); reglas 4.3, 6.3 y 6.4.

**Se parte en dos (Q28).** La fase 3a cierra el primer tramo del responsable —el flujo operativo hasta el ingreso de los IMEIs— y la 3b llega después, cuando él lo diga:

| Pieza (de esta sección) | 3a, en el primer tramo | 3b, después |
|---|---|---|
| Migraciones | Ninguna: los estados de compra, `replaced_equipment_id` (V149) y `stock.ingresar_equipamiento` (V151) llegaron con la fase 2 | Una: el completado |
| El escaneo | Solo sin fabricante: IMEI, serial y, si la SIM viene, su ICCID, que se guarda en `fleet_equipments.iccid`, como hoy; el equipo queda Disponible. No hay `arrival-settings`: en 3a siempre se ingresa así | El escaneo con fabricante («Por completar») y `arrival-settings`; el ICCID que reporta el fabricante, también en el equipo |
| El escaneo por modelo, la pantalla «Ingreso de equipos», su contador, la cámara (`_headers` y la librería perezosa) y las facturas con equipos por ingresar (`pendingArrival`) | Sí | — |
| El reemplazo de un fallado (`POST /equipments/{id}/replacement`), el rechazo de «volvió reparado» de un reemplazado (`EQUIPMENT_REPLACED`), la nota sobre la cadena y el costo que sigue la cadena (`fetchPurchaseRoots`) | Sí: el equipo nuevo queda Disponible, con su IMEI, serial e ICCID escaneados | — |
| La SIM, sus rutas, su pantalla y sus permisos | — | No: la SIM es un dato del GPS (Q30, Q31) |
| El pedido al fabricante, la tarea, el vigilante, la métrica, el planificador, los estados «Por completar» y «Revisar», el código del fabricante y la firma `SYSTEM` (`iam::acl`) | — | Sí |
| El menú | «Ingreso de equipos» va en el grupo de hoy, junto a Inventario de equipamiento, hasta que la fase 1 lo lleve a Monitoreo | Nada: no hay opción «SIM» |

La fase 4 ya no necesita la 3b por la SIM: «la SIM se exige para instalar» (C11) lee el ICCID del equipo, que ya existe (Q30). La sigue necesitando por el estado «Por completar», adonde vuelve un GPS heredado sin IMEI al retirarlo (fase 4).

**Migraciones.** Una sola, el completado (V155 si se abre en el orden del §5). La de la SIM (`sim_cards`, `sim_card_assignments` con sus `EXCLUDE` y el traslado de los ICCID) y la de sus permisos salieron con Q30 y Q31: el ICCID y el IMSI siguen en `fleet_equipments` y la aplicación los sigue escribiendo ahí.

**V155__add_the_completion_of_the_equipments.sql** (`Context: equipment`). Abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos.

- El `CHECK` de `status` suma `PENDING_COMPLETION` y `UNDER_REVIEW`.
- `fleet_equipments.review_reason VARCHAR(20)`, con `CHECK (review_reason IN ('MODEL_MISMATCH', 'UNKNOWN_IMEI', 'SIM_MISMATCH'))`, y presente solo en `UNDER_REVIEW`. `SIM_MISMATCH` dice que el ICCID que reporta el fabricante difiere del escaneado, o que ese ICCID ya lo tiene otro equipo.
- Si el número de la línea (MSISDN) y el operador se guardan en `fleet_equipments` se define al abrir la fase; si se guardan, van en esta migración, como datos del GPS.
- `equipment_models.manufacturer_model_code VARCHAR(60)`, opcional: cómo nombra el fabricante el modelo (análisis, C-H15).
- `equipment_completion_requests`:
  - columnas: `id`, `public_id`, `company_id`, `equipment_id` (llave compuesta), `imei`, `status` (`PENDING`, `IN_PROGRESS`, `COMPLETED`, `FAILED`, `CANCELLED`), `attempts`, `next_attempt_at`, `last_error_code`, `created_at`, `updated_at`. Como en los correos, `next_attempt_at` hace de reserva: reclamar un pedido lo corre hacia adelante (`Notification.claim` L182-187), sin otra columna;
  - un pedido abierto por equipo (índice único parcial sobre `PENDING` e `IN_PROGRESS`) y un índice `(status, next_attempt_at)`;
  - `UNIQUE (id, company_id)`.
- `equipment_completion_attempts`:
  - columnas: `id`, `public_id`, `company_id`, `request_id` (llave compuesta), `attempted_at`, `provider`, `outcome` (`COMPLETED`, `NOT_FOUND`, `RETRYABLE`, `REFUSED`), `http_status`, `error_code`, `duration_ms`;
  - de solo inserción: `SELECT` e `INSERT`, y su línea en `hand-over-database.sql`.
- Las dos tablas tienen RLS de empresa; la tarea corre con alcance `SYSTEM`.

**Backend.**

*La firma de lo automático.* `equipment` suma `iam::acl` a sus `allowedDependencies` para leer la cuenta `SYSTEM` (`IamContextFacade.fetchSystemAccountId()`, hoy solo la usa `fleet` en `ExternalIamService` L37). La tarea del fabricante firma con ella lo que escribe —`updated_by` del equipo y la fila de su historia, que no admite nulo— porque fuera de una sesión `CurrentUserAuditorAware` no devuelve a nadie y una actualización conservaría en silencio al autor anterior. `iam` no depende de `equipment`, así que no hay ciclo; `ModularityTests` lo comprueba.

*El ingreso y lo que sigue, en `equipment`*

| Operación | Desde | Hasta | Ruta y permiso |
|---|---|---|---|
| Escaneo, con fabricante | `PENDING_ARRIVAL` | `PENDING_COMPLETION` | `POST /api/v1/equipments/{id}/arrival-scan` `{imei, locationId}`, `stock.ingresar_equipamiento`. Valida el dígito verificador y la unicidad, y crea el pedido al fabricante en la misma transacción |
| Escaneo, sin fabricante (Q25) | `PENDING_ARRIVAL` | `AVAILABLE` | La misma ruta con `{imei, serialNumber, iccid?, locationId}`: con el proveedor `manual`, el escaneo trae también el serial —y el ICCID si la SIM viene— y el equipo queda Disponible, sin pasar por «Por completar». Con un fabricante configurado, esos campos se ignoran |
| Completar a mano | `PENDING_COMPLETION`, `UNDER_REVIEW` | `AVAILABLE` | `POST /api/v1/equipments/{id}/completion` `{serialNumber, firmware, …}`, `stock.editar_equipamiento`. Cancela el pedido abierto |
| Resolver la revisión | `UNDER_REVIEW` | `AVAILABLE` o `RETURNED_TO_SUPPLIER` | `POST /api/v1/equipments/{id}/review-resolution` `{outcome: CONFIRM \| CHANGE_MODEL \| RETURN, equipmentModelId}`, `stock.editar_equipamiento`. El modelo se corrige aquí dentro del mismo tipo (Q17) —desde D-290 también desde la ficha, salvo el de una factura, que es el caso de «Revisar», y el de una factura lo corrige «Anular y copiar» con sus equipos (D-296, Q40), que esta fase concilia con Q17 al abrirse—: `equipment_type_id` no cambia, y la llave de la instalación de la fase 4 lo usa. Cancela el pedido abierto |
| Reintentar con el fabricante | `PENDING_COMPLETION` | — | `POST /api/v1/equipments/{id}/completion-retry`, `stock.editar_equipamiento` |
| Reemplazo | `RETURNED_TO_SUPPLIER` | Un equipo nuevo en `PENDING_COMPLETION` (o `AVAILABLE` sin fabricante) | `POST /api/v1/equipments/{id}/replacement` `{imei, locationId, …}`, `stock.ingresar_equipamiento`. El nuevo copia empresa, proveedor, tipo y modelo, y apunta al reemplazado; el reemplazado queda devuelto y sale de las listas por omisión |
| Saber cómo se ingresa | — | — | `GET /api/v1/equipments/arrival-settings`, `stock.ingresar_equipamiento`: `{completesWithManufacturer}`, para que la pantalla pida o no el serial |

- El ingreso completo a mano de la fase 2 (`/arrival`) sigue.
- EQP-R14 queda: un tipo telemático exige el serial y el IMEI para estar Disponible; el ICCID sigue en el equipo, y no se exige para estar Disponible, sí para instalarlo (C11).
- **La máquina de estados, completa:**
  - `annulCredited()` suma `PENDING_COMPLETION` y `UNDER_REVIEW`: una nota por un equipo desconocido para el fabricante tiene que poder anularlo;
  - `comeBackRepaired()` se rechaza si otro equipo apunta a este como reemplazado (`EQUIPMENT_REPLACED`): si no, una línea tendría dos equipos vivos y su costo se contaría dos veces;
  - la nota «anulación de equipo» sobre la línea de un equipo reemplazado anula el extremo vivo de la cadena —el reemplazo—, no el que quedó en el proveedor;
  - cada camino deja su fila en la historia de la fase 2.
- **El escaneo va por modelo.** Una factura puede traer varios modelos (C3), y el costo del equipo sale de su línea. La pantalla elige el siguiente equipo por ingresar **del modelo que se está escaneando**; con un solo modelo no se pregunta. Si dos personas escanean la misma factura y chocan en el mismo equipo, la segunda recibe `EQUIPMENT_STATUS_TRANSITION_INVALID` y la pantalla pasa al siguiente.

*La SIM, dato del GPS (Q30, Q31)*

- No hay agregados ni rutas de la SIM: ni `SimCard` ni `SimCardAssignment`, ni `/sim-cards`, ni asignar o retirar una SIM de un GPS. Una SIM nunca se cambia sin su GPS, y el reemplazo de un fallado ya trae el GPS nuevo con su SIM (3a).
- El ICCID y el IMSI siguen en `fleet_equipments`, con el índice único del ICCID y sus códigos de siempre (`EQUIPMENT_ICCID_INVALID`, `EQUIPMENT_ICCID_CONFLICT`). Los escriben el alta y la edición a mano, el ingreso, el reemplazo y la API del fabricante; `POST /equipments`, `PATCH /equipments/{id}` y `/arrival` siguen aceptando `iccid`, y ninguna fase lo retira.
- El ERP no lleva el ciclo de vida de la SIM —activa, suspendida, cancelada— ni da de baja su línea: la maneja el proveedor (Q31).
- El recurso del equipo suma el estado de su completado y `reviewReason`; a quién reemplazó y quién lo reemplazó ya lo dice desde la fase 2.

*El pedido al fabricante, en `equipment`*, con el patrón de los correos:

| Pieza | Qué es |
|---|---|
| Puerto `DeviceRegistry` (`application/internal/outboundservices/registry`) | `lookup(imei)` responde `Found(serial, código del modelo, firmware, iccid, imsi)`, `NotFound`, `Retryable(código)` o `Refused(código)` |
| `ManualDeviceRegistry` (`infrastructure/providers`) | El proveedor por omisión (`equipment.device-registry.provider=manual`): no llama a nadie ni crea pedidos; el escaneo trae el serial (Q25) |
| El adaptador del fabricante | `RestClient` con timeouts del JDK (5 s para conectar, 10 s para leer); 408, 429 y 5xx se reintentan, otro 4xx se rechaza, 404 es `NotFound`. Espera Q21 |
| `DeviceRegistryProviderCheck` | Al arrancar, como `EmailProviderCheck`: un proveedor que no es `manual` exige su dirección y sus credenciales. El proveedor se elige con `@ConditionalOnProperty`, como los tres de correo |
| `DeviceCompletionTask` (`application/internal/scheduledtasks`, proceso `equipment.device-completion`) | `runAlone` → alcance `SYSTEM` → reclama los pedidos debidos con `FOR UPDATE SKIP LOCKED`, corriendo su `next_attempt_at`, en una transacción corta → llama al fabricante **fuera de toda transacción** → anota el intento y aplica el resultado en otra transacción corta, firmado por `SYSTEM` |
| La aplicación del resultado | Primero vuelve a leer, bajo candado, el equipo y luego el pedido (siempre en ese orden): si el equipo ya no está por completar o el pedido fue cancelado, el resultado se descarta. `Found`: guarda serial y firmware; si el modelo tiene código del fabricante y no coincide → `UNDER_REVIEW` `MODEL_MISMATCH`; guarda en el equipo el ICCID y el IMSI si no tiene ICCID; si el escaneado difiere del que reporta el fabricante, o si ese ICCID ya lo tiene otro equipo, de cualquier empresa (el índice es único en todo el sistema) → `UNDER_REVIEW` `SIM_MISMATCH`; si todo cuadra → `AVAILABLE`. `NotFound` → `UNDER_REVIEW` `UNKNOWN_IMEI` |
| Reintentos | Una espera que se duplica de 1 minuto a 1 hora, copiada de `RetryPolicy`; agotados, el pedido queda `FAILED` y el equipo por completar, con «el fabricante no respondió» y el completado a mano disponible |
| `DeviceCompletionFailureWatchTask` (proceso `equipment.device-completion-watch`) | Una línea `ERROR` por revisión cuando los fallos de la última hora llegan al umbral; no toca `/actuator/health` |
| Métrica | Contador `equipment.device_registry.lookups` con `provider`, `outcome` y `error_code` |
| El planificador | Hoy tiene un solo hilo (`SchedulingConfiguration` no fija `spring.task.scheduling.pool.size`): una llamada de 5 s + 10 s atrasaría los correos y los vencimientos. Se fija el tamaño del grupo y cada corrida reclama a lo sumo un lote acotado |
| Variables | `EQUIPMENT_DEVICE_REGISTRY_PROVIDER` (`manual` por omisión), su dirección y sus credenciales (los nombres finales con el adaptador), `EQUIPMENT_DEVICE_COMPLETION_INTERVAL` (`PT30S`), `EQUIPMENT_DEVICE_COMPLETION_MAX_ATTEMPTS` (`8`), `EQUIPMENT_DEVICE_COMPLETION_ALERT_FAILURES_PER_HOUR` (`3`), en el README, `.env.example` y `.do/app.yaml` |

- **El catálogo:** el diálogo del modelo de equipamiento (Catálogos › Tipos de equipamiento) suma «Código del fabricante», con `catalogos.editar_equipamiento`.

*`procurement`*

- La ficha de la factura muestra los estados nuevos.
- El costo de un equipo de reemplazo sigue la cadena hasta el equipo original: una operación nueva de la fachada de `equipment`, `fetchPurchaseRoots(ids)`.
- **Las facturas con equipos por ingresar,** para la pantalla de ingreso: `GET /api/v1/purchase-invoices` acepta `kind=EQUIPOS` y `pendingArrival=true`, y responde, por factura, cuántos equipos de cada modelo siguen por ingresar. Como el filtro cruza contextos, se aplica antes de paginar: `procurement` pide a la fachada de `equipment` los ids de los equipos por ingresar de las empresas que lee (`fetchPendingArrivalDeviceIds`, que son a lo sumo unos cientos), filtra en SQL las facturas cuyas líneas vigentes los nombran y recién entonces pagina.

**API y errores.**

| Código | Estado | Cuándo |
|---|---|---|
| `EQUIPMENT_REPLACED` | 422 | «Volvió reparado» de un equipo que otro ya reemplazó (construido con la 3a) |

Se reutilizan `EQUIPMENT_IMEI_INVALID`, `EQUIPMENT_ICCID_INVALID`, `EQUIPMENT_ICCID_CONFLICT` y `EQUIPMENT_STATUS_TRANSITION_INVALID`: el ICCID sigue siendo del equipo. Los códigos de la SIM que pensaba la versión 3 —`SIMCARD_NOT_FOUND`, `SIMCARD_CONFLICT`, `SIMCARD_ICCID_INVALID`, `SIMCARD_NOT_ACTIVE`, `SIMCARD_ALREADY_ASSIGNED` y `EQUIPMENT_SIM_ALREADY_ASSIGNED`— no se crean (Q30, Q31).

**Frontend.**

- **«Ingreso de equipos»** (`F/equipment/presentation/views/equipment-arrivals-view`, ruta `/equipment/arrivals`, con `stock.ingresar_equipamiento`):
  1. Elegir una factura con equipos por ingresar (de `ProcurementFacade`).
  2. Si la factura tiene más de un modelo, elegir el modelo que se va a escanear (la caja); el contador es por modelo.
  3. Elegir la ubicación, que se recuerda durante la sesión en un store en memoria: la skill prohíbe guardarla en el navegador (§3, L87).
  4. Escanear: un campo con el foco puesto para el lector (cada lectura termina con Enter; también se escribe a mano), y «Usar cámara», que carga la librería de lectura solo al pulsarlo (`BarcodeDetector` donde existe).
  - Cada lectura busca en el contenido el número de 15 dígitos que pasa el dígito verificador (`imei.ts`), y lo envía para el siguiente equipo por ingresar de ese modelo.
  - Sin fabricante (`arrival-settings`), la misma fila pide después el serial y, si la SIM viene, su ICCID, también por escaneo; el equipo queda Disponible.
  - Un contador («37 de 100») y, por cada lectura, una fila con su resultado (ingresado, repetido, inválido, ya registrado), en una lista y no en una tabla, para no abrir otra excepción a `mm-data-manager`.
  - Mientras haya lecturas sin enviar, la pantalla se registra como trabajo sin guardar.
  - Al terminar, un resumen.
- **La cámara en el sitio desplegado.** `public/_headers` (L10) envía `Permissions-Policy: camera=()` a todo `/*`, y en una aplicación de una sola página no sirve un encabezado por ruta: pasa a `camera=(self)`. El e2e no lo detecta, porque el servidor de desarrollo no aplica `_headers`; se comprueba en `dev`.
- **La librería de la cámara**, con su versión exacta, cargada perezosamente. Los presupuestos de `angular.json` (L46-57) solo miden el paquete inicial: se suma un presupuesto de tipo `bundle` para su trozo perezoso, y `npm audit` sigue en cero.
- **Sin pantalla «SIM»** (Q30, Q31): la SIM se ve en la ficha del equipo, su ICCID y su IMSI, como hoy.
- **El inventario y la ficha del equipo:** los estados «Por completar» y «Revisar» (con su causa), sus filtros, y las acciones que dice el backend: completar datos, resolver la revisión, reintentar con el fabricante, registrar el reemplazo, volvió reparado.
- **El menú:** Monitoreo suma «Ingreso de equipos» (`stock.ingresar_equipamiento`) cuando la fase 1 lo cree; no hay opción «SIM» ni permisos `stock.*_sims` en la matriz.
- **`DESIGN.md`:** una revisión para la pantalla de escaneo, su lista y la cámara.
- Cada ruta nueva va en `environment.ts` y en `environment.development.ts`.

**Pruebas.**

- Backend:
  - la tarea con un `DeviceRegistry` de prueba: encontrado, otro modelo, un ICCID distinto del escaneado, un ICCID que ya tiene otro equipo, de otra empresa, desconocido, reintentable, agotado, y un resultado que llega después de un completado a mano;
  - que corre en una sola instancia y fuera de la transacción del escaneo: las dos tareas nuevas entran en la lista exacta de `DeadlockRetryTests` (L123-126);
  - el vigilante y la métrica; la comprobación de arranque;
  - el escaneo con y sin fabricante (dígito verificador, repetido, estado equivocado, permiso, modelo), el reemplazo y su costo, «volvió reparado» de un reemplazado, la nota sobre la cadena, la revisión dentro del tipo;
  - `ModularityTests` con `iam::acl` en `equipment`;
  - `EquipmentRowsAreNeverDeletedTests` con las tablas nuevas, `RowLevelSecurityMigrationTests` con las dos del completado, `HandOverDatabaseScriptTests`;
  - la suite completa, en local.
- Frontend: las specs de la pantalla de ingreso (la extracción del IMEI, el Enter del lector, el modelo, el contador, el modo sin fabricante, los errores), de `imei.ts` y del inventario; e2e del ingreso (el lector simulado escribiendo y pulsando Enter; la cámara no se prueba en e2e) y del inventario; `E/screens.ts` y las imágenes de las pantallas que cambian.

**Documentación.**

- Diccionario:
  - §11.4: los estados de completado y la revisión, con `SIM_MISMATCH`, y, si se guardan, el número de la línea y el operador del GPS;
  - §2.7: los pedidos al fabricante y sus intentos;
  - §7.5: el código del fabricante;
  - D-285 pasa a cerradas.
- `contextos/equipment.md`: las reglas, el puerto del fabricante, la integración y la firma `SYSTEM`.
- El README del backend: las variables, el proveedor, el vigilante y cómo encenderlo.
- `procesos-y-pantallas`: el proceso «Ingreso de equipos» y las pantallas nuevas. Las convenciones de la API: las rutas.

**El adaptador real,** cuando llegue Q21: el adaptador del fabricante, sus pruebas contra respuestas grabadas (`MockRestServiceServer`), su sección «El fabricante» en el README, y el cambio de `EQUIPMENT_DEVICE_REGISTRY_PROVIDER` en App Platform.

**Orden de los commits.**

- Backend (3b; lo de la 3a ya se integró):
  1. La migración del completado, sus estados y la máquina de estados completa.
  2. El escaneo con fabricante y el completado a mano.
  3. El puerto, el proveedor manual, la tarea, el vigilante, la métrica y el planificador.
- Frontend (3b):
  1. El escaneo con fabricante en «Ingreso de equipos».
  2. El inventario y la ficha.
  3. Los e2e y las imágenes.

**Terminada cuando:**
- **3a:** en `dev`, se escanea una factura de dos modelos, modelo por modelo, y sus equipos quedan Disponibles con su IMEI y su serial; la cámara abre en un teléfono; un fallado devuelto al proveedor se reemplaza por otro equipo, que toma su costo, y el devuelto ya no puede «volver reparado»; la suite completa y el CI están en verde.
- **3b:** con un `DeviceRegistry` de prueba, la suite recorre todos los resultados, también el ICCID que no cuadra; la suite completa y el CI están en verde.

---

### Fase 4 — Instalación unida al inventario e historial del equipo (D-263, D-264)

**Objetivo.** Instalar es elegir un equipo Disponible del inventario y, si es telemático, con su SIM, es decir, con su ICCID (C11); retirar lo devuelve al estado que dice su motivo; una unidad con equipos instalados no se anula ni sale de la flota; los GPS ya instalados —también los de la carga inicial— quedan enlazados a su equipo; y cada equipo muestra en qué unidades estuvo y cuándo se retiró o reemplazó (C13).

**Lo que la rige.** D-263, D-264, D-206, FLT-R152 a FLT-R154, la fase 2 §2.6 del plan de migración; C11, C13; reglas 1.2 y 1.3.

**Antes de tocar nada:** las pruebas de caracterización de la instalación de hoy, en `develop`.

**Migraciones.**

**V157__key_the_equipments_for_their_installation.sql** (`Context: equipment`).

- `fleet_equipments_id_company_type_uq UNIQUE (id, company_id, equipment_type_id)`, destino de la llave de la instalación.
- `fleet_equipments_current_vehicle_fk (current_vehicle_id, company_id) → vehicles (id, company_id)`, que reemplaza la llave simple de V132 L34. El `CHECK` que ata Instalado a `current_vehicle_id` (V132 L49-51) no cambia.
- **Un modelo «Sin modelo registrado» por cada tipo telemático** de `equipment_types`, inactivo para las compras: `equipment_model_id` no admite nulo (V132 L22), y una instalación vieja solo sabe su tipo y su serial. Lo usa solo la vinculación de lo ya instalado; el modelo se corrige después desde la ficha del equipo.

**V158__link_the_installations_to_the_inventory.sql** (`Context: fleet`).

- `vehicle_equipments.equipment_id BIGINT`, con `vehicle_equipments_equipment_fk (equipment_id, company_id, equipment_type_id) → fleet_equipments (id, company_id, equipment_type_id)`. `vehicle_equipments` ya tiene `equipment_type_id` (V100). Queda vacío solo en lo de fábrica.
- Una instalación abierta por equipo: índice único parcial sobre `equipment_id` `WHERE equipment_id IS NOT NULL AND upper_inf(period)`.
- El `CHECK` de `removal_reason` (V100 L47-48) suma `sin_falla`.
- `UPDATE (equipment_id)` para la cuenta de la aplicación, como V119 con el serial, para enlazar lo ya instalado.

**Backend.**

*`fleet`*

- **Instalar** (`VehicleEquipmentInstallations.check` L70-110):
  - el pedido suma `equipmentId`. Es el **primer paso** del cambio incompatible: el backend acepta el equipo y el serial;
  - el equipo tiene que verse, ser de la empresa de la unidad, estar Disponible y ser de un tipo que aplica;
  - un tipo telemático exige que el equipo tenga su ICCID (`EQUIPMENT_SIM_REQUIRED`): la SIM es un dato del GPS, sin estado ni asignación propios (C11, Q30);
  - después de los candados de la unidad y de la instalación (`lockedWithItsUnit`, `VehicleEquipmentCommandServiceImpl` L238-249), `installDevice` de `equipment` bloquea el equipo y lo pasa a Instalado, con su unidad.
- **Retirar y reemplazar** (`VehicleEquipmentCommandServiceImpl`): los motivos falla, robo, reemplazo y sin falla. `releaseDevice` lo devuelve a Mantenimiento (falla), De baja (robo) o Disponible (reemplazo, sin falla), en la ubicación actual de la unidad; si la unidad no tiene ubicación, el retiro la pide. Un GPS heredado sin serial o sin IMEI que volvería a Disponible queda «Por completar» (EQP-R14). Reemplazar elige otro equipo.
- **D-264:**
  - `VehicleAnnulments` suma la retención `DEVICES_INSTALLED`;
  - **el paso a Inactivo** con equipos instalados se rechaza (`VEHICLE_DEVICES_INSTALLED`) donde ocurre: en `VehicleMovements` (L131, L150, con `TransferExitPolicy`), y `VehicleAllowedActions` (L124) deja de ofrecerlo. Cubre también el camino «Devolución al proveedor» de D-265 (`StatusChangeReasonCommandServiceImpl` L83). `VehicleTransfers` solo registra los datos del notario; cerrar un traspaso sale a Inactivo por `VehicleMovements`, así que la misma regla exige retirar antes el GPS;
  - `ReturnedUnits` retiene la unidad que una nota devuelve si tiene equipos instalados (análisis, C-H12), con la causa que informará D-266.
- **El historial del equipo en sus unidades (C13).** `fleet` es dueño de las instalaciones y `equipment` no puede leerlas (`fleet` depende de `equipment`, no al revés). Así que `fleet` sirve `GET /api/v1/equipments/{id}/installations` (`stock.ver`), en un controlador propio, como `procurement` sirve el costo del equipo: lee el equipo con `fetchSeenDevice` de la fachada de `equipment` (404 si la sesión no lo ve) y lista cada instalación con su unidad (VIN y placa), su inicio, su fin, el motivo del retiro y quién instaló y retiró.
- **La vinculación de lo ya instalado** (análisis, C-H13; D-286):
  - `linkLegacyInstalledDevice(empresa, tipo, serial, unidad, cuenta SYSTEM)` de la fachada de `equipment`, con un resultado por caso. El serial es único en todo el sistema (V132 L55-57):

    | El serial | Resultado |
    |---|---|
    | No existe | Registra el equipo Instalado en esa unidad, con el modelo «Sin modelo registrado» de su tipo, sin ubicación ni proveedor, firmado por `SYSTEM`, y abre su historia |
    | Existe en la misma empresa y tipo, Disponible | Lo instala en esa unidad |
    | Existe en la misma empresa y tipo, ya instalado en esa unidad | Lo enlaza: es idempotente |
    | Existe en otra empresa, en otro tipo o en otro estado | No enlaza; se informa |

  - `InstalledGpsInventoryLinking`, de `fleet`: un proceso único e idempotente que corre después del arranque, sin demorarlo (al estar lista la aplicación, en segundo plano), en una sola instancia (`fleet.installed-gps-linking`) y con alcance `SYSTEM`. Para cada instalación abierta de GPS sin equipo y con serial, pide a `equipment` el enlace y escribe `equipment_id`. Las que no tienen serial y las que no se enlazan se listan en el registro, con su causa. En cada arranque posterior no encuentra nada que hacer;
  - **completar el serial después** (`PUT /vehicle-equipments/{id}/serial`, `VehicleEquipmentCommandServiceImpl` L200-223, que hoy solo escribe el serial) enlaza en la misma transacción, con los mismos casos;
  - **la carga inicial** crea las instalaciones de GPS solo con su tipo y su fecha, sin serial (`MigratedUnitRegistrations` L68, L226, L342-348; D-206): quedan sin equipo hasta que alguien completa su serial, y entonces se enlazan. Así ningún GPS escapa de D-264 por haber llegado después del arranque.

*`equipment`*

- `installDevice`, `releaseDevice(destino, ubicación)` y `linkLegacyInstalledDevice` en la fachada, con `Propagation.MANDATORY` y el equipo bloqueado `FOR NO KEY UPDATE`; cada uno deja su fila en la historia.
- El cambio de estado a mano rechaza un equipo Instalado (`EQUIPMENT_INSTALLED`) y deja de ofrecer el paso de Instalado a Mantenimiento (`Equipment.moveToMaintenance`, L192-199).

**API y errores.**

| Código | Estado | Cuándo |
|---|---|---|
| `EQUIPMENT_NOT_AVAILABLE` | 422 | El equipo elegido no está Disponible |
| `EQUIPMENT_SIM_REQUIRED` | 422 | Un equipo telemático sin ICCID |
| `EQUIPMENT_INSTALLED` | 422 | Cambiar a mano el estado de un equipo instalado |
| `VEHICLE_DEVICES_INSTALLED` | 422 | Anular, pasar a Inactivo o cerrar el traspaso de una unidad con equipos instalados; nombra los equipos |

**Frontend.**

- **El diálogo de instalación** (`vehicle-equipment-dialog.ts`, formulario L113-131, envío L257-288): una búsqueda (`mm-search-select`) de los equipos Disponibles de la empresa de la unidad y del tipo elegido, que muestra su ICCID. Es el **segundo paso**: el frontend envía el equipo.
- `REMOVAL_REASONS` (`vehicle-equipment.entity.ts` L8) y `EquipmentRemovalReason` (L5) suman `sin_falla`; el retiro pide la ubicación cuando la unidad no tiene una.
- Los errores nuevos, y `complete-equipment-serial-dialog`, que queda para lo instalado sin serial y ahora también enlaza.
- **La ficha del equipo** suma la pestaña «Instalaciones», con lo que responde `GET /equipments/{id}/installations`: junto a la pestaña «Historia» de la fase 2, es el historial que anunció el correo del 16/11.

**Pruebas.**

- La caracterización, sin cambios.
- FLT-R152 a FLT-R154.
- Las retenciones de anulación, Inactivo (también por «Devolución al proveedor»), traspaso y nota.
- La vinculación: cada caso de la tabla, idempotente, el serial completado después y una instalación de la carga inicial que se enlaza al completar su serial.
- La ruta de las instalaciones de un equipo (404 si no se ve).
- El orden de los candados.
- `VehicleEquipmentIntegrationTests`, `VehicleEquipmentAuditTests`, `COLUMN_UPDATE_TABLES` de `RowLevelSecurityMigrationTests`.
- e2e `enablement.spec.ts` (L234-460) y la ficha del equipo.
- La suite completa, en local.

**Documentación.** D-263 y D-264 pasan a construidas y D-286 a cerradas, con la precisión de D-206; FLT-R152 a FLT-R154 en ✅ en `contextos/fleet.md`; el diccionario §11.4 y §8.3 (la retención); `procesos-y-pantallas` P6 y la ficha del equipo.

**Orden de los commits.**
1. La caracterización.
2. V157 y la fachada de `equipment`.
3. V158, instalar, retirar y reemplazar.
4. Las retenciones.
5. La vinculación y el serial completado.
6. Las instalaciones de un equipo.
7. El frontend.

**Terminada cuando:** se instala un GPS del inventario con su SIM, se retira con cada motivo, una unidad con GPS no se anula ni pasa a Inactivo, los GPS ya instalados de `dev` quedan enlazados o listados con su causa, completar un serial enlaza, la ficha de cada equipo muestra su historia y sus instalaciones, y la suite y el CI están en verde.

**Hecha el 23/11/2026** (backend y frontend en `feature/phase-4-installation-from-inventory`, integrados en `develop`, con los hashes de la línea de Estado; documentación en `docs/phase-4-installation`, diccionario 1.86). Se hizo antes de la 3b, que espera a Q21. D-263 queda construida en sus dos primeros pasos —el tercero es la fase 5—, D-264 construida y D-286 cerrada, con la precisión de D-206: completar el serial de un GPS lo enlaza. Se aparta del plan en doce cosas:
1. **Las migraciones** son V155 (la V157 de arriba) y V156 (la V158). V155 crea también la marca «SIN MARCA REGISTRADA», inactiva como los modelos, que el plan no nombraba; V156 suma `vehicle_equipments_equipment_idx`, para leer las instalaciones de un equipo, y la columna en `scripts/database/hand-over-database.sql`. La 3b, que esperaba V155, toma su número al abrir su rama, después de las de la fase 6 (V157 a V160) y de las de la 7.
2. **No hay «Por completar».** La fase dependía de la 3b por ese estado; en su lugar, un equipo que volvería a Disponible sin los identificadores que exige su tipo —el serial y el IMEI de un telemático; el serial de uno que lo exige— queda en Mantenimiento, y su historia lo dice («Le faltan los identificadores que su tipo exige para estar Disponible»): se completan desde su ficha y vuelve a Disponible a mano.
3. **El selector usa la lista que ya existía**, `GET /equipments?companyId&equipmentTypeId&status=AVAILABLE&q`, de a 50 y buscada en el servidor (`EquipmentFacade.searchAvailableDevices`), sin ruta nueva. Cada opción dice el serial o el IMEI, el modelo y el ICCID o «Sin SIM (ICCID)». `SearchedOptions` pasó de `procurement` a `shared`, en un commit propio, para usarlo también aquí.
4. **Completar un serial rechaza lo que no enlaza**, en vez de informarlo: no escribe nada y responde `VEHICLEEQUIPMENT_DEVICE_OTHER_COMPANY`, `VEHICLEEQUIPMENT_DEVICE_TYPE_MISMATCH`, `EQUIPMENT_NOT_AVAILABLE`, el nuevo `VEHICLEEQUIPMENT_UNREGISTERED_MODEL_MISSING` o `409 EQUIPMENT_SERIAL_NUMBER_CONFLICT`, que el diálogo dice con sus propios textos. Solo el proceso del arranque informa, en el registro. El serial de un tipo que no es telemático se completa como antes, sin enlazar.
5. **Un GPS heredado se enlaza sin ICCID**, el que se registra y el Disponible que se instala: la SIM solo se exige al instalar desde el inventario (C11).
6. **La tabla del enlace tiene ocho casos**, no cuatro: los tres que enlazan (`REGISTERED`, `INSTALLED`, `ALREADY_INSTALLED`) y cinco que no —`OTHER_COMPANY`, `OTHER_TYPE`, `OTHER_STATUS`, `MODEL_MISSING`, un tipo telemático sin el modelo «SIN MODELO REGISTRADO», y `SERIAL_TAKEN`, un serial de un equipo que la transacción no ve, que el índice único rechaza—. Un equipo de baja o anulado responde por el serial solo si ningún equipo vivo lo tiene. El proceso se llama `InstalledGpsInventoryLinkingTask`, y en lugar de correr «al estar lista la aplicación» corre una vez en el programador, `fleet.installed-gps-linking.initial-delay` después del arranque (un minuto), cada instalación en su transacción; las pruebas lo apagan (`P3650D`) y lo corren a mano. Enlaza también, en el arranque siguiente, un GPS que se instaló por su serial escrito durante el primer paso del cambio.
7. **La variable nueva**, `FLEET_INSTALLED_GPS_LINKING_INITIAL_DELAY`, está solo en `application.yaml`: falta en `.env.example`, en `.do/app.yaml` y en la tabla del README, que §4 pide para toda variable nueva; la de la fase 2, que también tiene valor por omisión, al menos está en el README. **La cerró la fase 5**, que la documenta en los tres (abajo).
8. **Las retenciones** van donde dice el plan, con dos precisiones: el paso a Inactivo se revisa después de la devolución al proveedor y antes del traspaso y la puerta, y la anulación, al final; y la ficha no deja de ofrecer los caminos a Inactivo, sino que los muestra deshabilitados por una lista nueva, `allowedActions.devicesHeldReasonIds`, como `transferHeldReasonIds`. `VEHICLE_DEVICES_INSTALLED` nombra los equipos en `context.devices`, y la anulación ahora responde su `context`. La nota que devuelve la unidad la retiene con `ReturnedUnitHold.DEVICES_INSTALLED`, que `procurement` sigue descartando hasta D-266.
9. **Los códigos nuevos son ocho**, no cuatro: los del plan y `VEHICLEEQUIPMENT_DEVICE_OTHER_COMPANY`, `VEHICLEEQUIPMENT_DEVICE_TYPE_MISMATCH`, `VEHICLEEQUIPMENT_LOCATION_REQUIRED` y `VEHICLEEQUIPMENT_UNREGISTERED_MODEL_MISSING`, cada uno con su mensaje en los dos bundles. Se reutilizan `EQUIPMENT_NOT_FOUND`, `VALIDATION_ERROR` —un serial escrito que no es el del equipo elegido—, `LOCATION_NOT_FOUND`, `LOCATION_OTHER_COMPANY`, `LOCATION_INACTIVE` y `EQUIPMENT_SERIAL_NUMBER_CONFLICT`.
10. **Las instalaciones de un equipo no dicen quién retiró**: `vehicle_equipments` no lo guarda, y lo dice la historia del equipo, en el cambio que lo sacó de Instalado (Q35). Cada fila trae también su identificador, su tipo y su serial, y la ruta, en `EquipmentInstallationsController` de `fleet`, responde el arreglo sin página (convenciones, §13 fila 27).
11. **La ficha del equipo sigue siendo un diálogo** (fase 2, precisión 1 del frontend): «Instalaciones» es su última sección, después de «Historial de estados», y no una pestaña. A un equipo instalado se le quitó «Cambiar estado», pero se le sigue ofreciendo «Editar», y sus datos técnicos se corrigen (Q36).
12. **Las pruebas.** Del backend, la caracterización, sin cambios, y las de cada paso: `VehicleEquipmentInventoryIntegrationTests`, `InstalledDevicesHoldsIntegrationTests`, `InstalledGpsInventoryLinkingTaskTests`, `SerialCompletionLinkIntegrationTests`, `EquipmentInstallationsIntegrationTests`, `EquipmentContextFacadeInstallationTests` y `DeviceInstallationsTests`, además de las que cambiaron. `RowLevelSecurityMigrationTests` no cambió: su `COLUMN_UPDATE_TABLES` revisa una sola columna por tabla, `cost` en `vehicle_equipments`. Del frontend, por la regla de verificación del responsable, el build, las specs de los archivos tocados y `lint:styles`; los e2e (`enablement.spec.ts` y la ficha del equipo) no se tocaron, y el simulacro de la API del e2e no sirve todavía `GET /equipments/{id}/installations`: los corre el CI.

Quedan del responsable cuatro hallazgos, como preguntas (§2): una unidad Robada con un GPS del inventario no sale a Inactivo (Q33), el modelo de un equipo no se corrige (Q34), la instalación no guarda quién la retiró (Q35) y la edición se ofrece a un equipo instalado (Q36). Q33 y Q34 las respondió el mismo día (D-289, D-290; versión 3.7 del plan), y Q35 y Q36 también (D-291, D-292; versión 3.8).

---

### Fase 5 — Limpieza

**Objetivo.** Retirar lo que quedó para la compatibilidad del ciclo del GPS.

**Antes de empezar:** la fase 4 está integrada, con su frontend (el segundo paso de la instalación).

**Migraciones.** Ninguna. La versión 3 retiraba aquí `fleet_equipments.iccid` e `imsi`, con su índice único, y el disparador `fill_purchase_line_kind()`: el ICCID y el IMSI se quedan, porque la SIM es un dato del GPS (Q30, Q31), y el disparador nunca se creó (fase 2, precisión 2). Con ellos salen las condiciones que pedía antes de borrar columnas.

**Backend.** El **tercer paso** de la instalación: sale el camino del serial escrito para los tipos del inventario. `complete-equipment-serial-dialog` y su ruta siguen, solo para lo instalado sin serial, que al completarse se enlaza (fase 4). `POST /equipments`, `PATCH /equipments/{id}` y `/arrival` siguen aceptando `iccid`.

**Frontend.** Sale el serial escrito del diálogo de instalación; el ICCID se queda en el formulario del equipo.

**Documentación.** El tercer paso de la instalación, en D-263.

**Terminada cuando:** el CI está en verde y la instalación de un tipo del inventario ya no acepta el serial escrito.

**Hecha el 23/11/2026 e integrada** (backend y frontend juntos, después de la fase 7 del backend, con los hashes de la línea de Estado; documentación con la de la fase 7, diccionario 1.88). Sobre `develop` con las fases 6 y 7 pasaron en local 30 clases y 150 pruebas dirigidas de la instalación y la habilitación; el CI de GitHub no corrió, porque la cuenta no arranca los jobs. D-263 queda construida en sus tres pasos. Sin migraciones, como decía el plan. Se aparta de él, o lo precisa, en seis cosas:
1. **Qué es un tipo del inventario**, que el plan no definía: un tipo telemático que el modelo de la unidad no trae de fábrica (`EquipmentTypeReference.requiresDevice`: `requiresTelematics && !factoryFitted`), leído en el tipo como lo admite el modelo de la unidad; hoy, solo el GPS. Es el criterio con que la fase 4 enlaza un GPS ya instalado y el serial completado (D-286). La pantalla, que no es telemática, y lo que el modelo trae de fábrica se siguen instalando con su serial escrito o con un equipo.
2. **Un código nuevo, `VEHICLEEQUIPMENT_DEVICE_REQUIRED`** (`422`, campo `equipmentId`), con su texto en los dos bundles: instalar o reemplazar un tipo del inventario sin `equipmentId`, revisado en `VehicleEquipmentInstallations.check` después de `EQUIPMENTTYPE_INACTIVE` y `VEHICLEEQUIPMENT_TYPE_NOT_APPLICABLE` y antes del serial, también para reemplazar un GPS que se instaló a mano por su serial. Con `equipmentId`, un `serial` escrito responde `400 VALIDATION_ERROR` sobre `serial` **aunque coincida**: en la fase 4 se aceptaba si era el del equipo. `InstallVehicleEquipmentCommand` perdió su constructor sin equipo.
3. **La habilitación suma `requiresDevice`** en cada fila de `equipmentRequirements` (`GET /vehicles/{id}/enablement`): un campo nuevo, compatible con los clientes de hoy, que la pantalla lee para dejar de pedir el serial de esos tipos.
4. **Completar el serial** sigue, solo para lo instalado sin él, como los GPS de la carga inicial, y enlaza un GPS (D-286); retirar un GPS instalado por su serial no cambia. `POST /equipments`, `PATCH /equipments/{id}` y `/arrival` siguen aceptando `iccid`, como decía el plan.
5. **La variable de la fase 4** (precisión 7): `FLEET_INSTALLED_GPS_LINKING_INITIAL_DELAY` queda en `.env.example`, comentada como las demás opcionales; en `.do/app.yaml`, entre los valores por omisión de los procesos programados; y en la tabla del README, que dice además que `InstalledGpsInventoryLinkingTask` corre una vez después del arranque, en segundo plano, en una sola instancia y con el alcance `SYSTEM`. Las pruebas la siguen apagando (`P3650D`).
6. **Los commits del backend son tres:** la variable, `requiresDevice` en la habilitación y el tercer paso. Las pruebas que instalaban o reemplazaban un GPS por su serial escrito eligen un GPS Disponible del inventario (`EquipmentFixtures.availableGps`); `VehicleEquipmentIntegrationTests` prueba las reglas de cada tipo con un tipo propio que pide su serial; y la caracterización de la instalación de la fase 4 (`InstallationCharacterizationTests`) y `VehicleEquipmentInventoryIntegrationTests` dicen el tercer paso. La caracterización cambia a propósito: el comportamiento que fijaba es el que la fase retira.

**El frontend, en su rama** (`feature/phase-5-cleanup`, con el hash de la línea de Estado, sin integrar). La habilitación lee `requiresDevice`, falso si el backend no lo dice, así que con un backend anterior el diálogo sigue pidiendo el serial. Para un tipo del inventario, el diálogo de instalar y reemplazar exige el equipo, sin opción vacía, no muestra ni envía el serial, dice que el serial es el del equipo y, si el backend responde `VEHICLEEQUIPMENT_DEVICE_REQUIRED`, lo pide en el campo del equipo; los demás tipos siguen como en la fase 4, y el ICCID se queda en el formulario del equipo. Los e2e de la habilitación instalan y reemplazan el GPS eligiendo `SN-GPS-0001` y `SN-GPS-0002` en «Equipo del inventario», con el simulacro de la habilitación que responde como el backend; no se corrieron en local, los corre el CI. `e2e-live` elige en `dev` un GPS Disponible con su ICCID de la empresa de Perú para cada instalación de sus pasos 4 y 5, y los salta diciéndolo si no hay dos. Si el backend se integra antes, como pide §4, la pantalla de `develop` sigue ofreciendo «Ninguno del inventario» con el serial escrito para un GPS, que el backend rechaza con su mensaje: mientras tanto, el GPS se instala eligiendo su equipo, que esa pantalla ya permite.

---

### Fase 6 — Áreas: catálogo, asignación y sesión

**Objetivo.** Cada empresa administra sus áreas; un colaborador puede tener la suya; la asignación de un rol puede llevar un área —una persona con varias áreas tiene una asignación por área— y la sesión la conoce. **Nada se filtra todavía**: ningún registro tiene área. La pantalla de otorgar un rol ofrece el área recién en la fase 7, para que no exista una asignación con área antes de que compras la respete. La API ya la acepta, sin efecto hasta la fase 7: el resultado es el mismo rol sin área, nunca más acceso.

**Lo que la rige.** A1, A2, A5, A9; Q5, Q6; D-84 (a lo inactivo no se le cuelga nada nuevo); D-102 como molde; reglas 5.5, 5.7 y 5.9; V27, V31.

**Migraciones.**

**V160__create_organization_areas_schema.sql** (`Context: organization`). Plantilla: V108, con V16, V20 y V26.

```
areas
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
  public_id   UUID         NOT NULL DEFAULT uuidv7()
  company_id  BIGINT       NOT NULL REFERENCES companies (id)
  code        VARCHAR(20)  NOT NULL
  name        VARCHAR(100) NOT NULL
  is_active   BOOLEAN      NOT NULL DEFAULT TRUE
  created_at  TIMESTAMPTZ  NOT NULL DEFAULT now()
  updated_at  TIMESTAMPTZ  NOT NULL DEFAULT now()

  areas_public_id_uq     UNIQUE (public_id)
  areas_company_code_uq  UNIQUE (company_id, code)
  areas_id_company_uq    UNIQUE (id, company_id)     -- destino de las llaves compuestas
```

- Las mismas columnas de registro y los mismos `CHECK` que `commercial_brands` (V16 L20-21): código y nombre no vacíos; el nombre no es único. El formato del código —mayúsculas y dígitos, de 2 a 20— vive en el dominio, como el de la marca (`CommercialBrand` L29, L116-119).
- Un disparador que impide cambiar la empresa de un área, como V108 L47-60: `keep_area_company()`, con su línea en `hand-over-database.sql` en esta misma fase (`HandOverDatabaseScriptTests` L159).
- RLS habilitada y forzada, con `areas_select`, `areas_insert` y `areas_update` sobre `access_scope_reaches(company_id)`; los permisos de la cuenta de la aplicación como V108 L73-86.
- Sus 64 casillas en `data_versions` y sus dos disparadores (`areas_data_version_insert_tg`, `areas_data_version_update_tg`), porque se exporta.
- Abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos.

**V161__seed_iam_permissions_of_areas.sql** (`Context: iam`). Plantilla: V109.

| Código | Categoría | Se marca en |
|---|---|---|
| `configuracion.crear_areas` | `crear` | `SUPERADMIN`, `ADMIN_EMPRESA` |
| `configuracion.editar_areas` | `editar` | `SUPERADMIN`, `ADMIN_EMPRESA` |
| `configuracion.desactivar_areas` | `desactivar_anular` | `SUPERADMIN`, `ADMIN_EMPRESA` |

Con `ON CONFLICT (code) DO NOTHING` y las marcas firmadas por `system@millennium.ky`. Leer las áreas es `configuracion.ver_estructura`. No hay pares incompatibles nuevos.

**V162__scope_iam_role_assignments_by_area.sql** (`Context: iam`). Plantilla: V27 L31-57 y V61 L22-29.

- Abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos: vuelve a crear un índice de `user_roles`, que lee cada pedido.
- `user_roles.area_id BIGINT`.
- `user_roles_area_same_company_fk FOREIGN KEY (area_id, company_id) REFERENCES areas (id, company_id)`.
- `user_roles_area_requires_company_ck CHECK (area_id IS NULL OR company_id IS NOT NULL)`: la llave compuesta no lo impide sola, porque una empresa nula la salta.
- `user_roles_active_scope_uq` se vuelve a crear con el mismo nombre sobre `(user_id, role_id, company_id, branch_id, commercial_brand_id, area_id) NULLS NOT DISTINCT WHERE revoked_at IS NULL`, en esta misma migración (análisis, A-H9).

**V163__add_the_area_of_the_employees.sql** (`Context: organization`).

- Abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos.
- `employees.area_id BIGINT` y `employees_area_same_company_fk FOREIGN KEY (area_id, company_id) REFERENCES areas (id, company_id)`. Sin índice, como `employees.branch_id` desde V22.

**Backend.**

*`shared`*

- `PermissionGrant`: `@Nullable Long areaId` («un área siempre es de una empresa»), con el constructor de cuatro argumentos conservado. Predicados nuevos:

  | Predicado | Verdadero cuando |
  |---|---|
  | `readsBrand(@Nullable brandId)` | La asignación no tiene marca, o el registro no la tiene, o coinciden |
  | `writesEveryBrand()` | La asignación no tiene marca |
  | `readsArea(@Nullable areaId)` | La asignación no tiene área, o el registro no la tiene, o coinciden |
  | `writesArea(@Nullable areaId)` | La asignación no tiene área, o el registro tiene exactamente su área |
  | `writesEveryArea()` | La asignación no tiene área |

- `AccessScope`:
  - `anyGrantOf(String permission, Predicate<PermissionGrant> where)`: verdadero con el alcance `SYSTEM`, o cuando **una misma** asignación del permiso cumple `where`. Es la forma de exigir empresa, sede, marca y área a la vez sin sumar asignaciones.
  - `recordReach(String permission)`: devuelve `RecordReach.Every` con el alcance `SYSTEM` o con una asignación global del permiso; si no, `RecordReach.Some` con una entrada por asignación —empresa, marca y área—, quizá ninguna.
  - Los métodos de hoy no cambian; el Javadoc (L23-34) suma el área.
- `RecordReach` (nuevo, en `shared/application/security`): `Every` o `Some(List<Entry(companyId, @Nullable commercialBrandId, @Nullable areaId)>)`. Una fila se alcanza cuando alguna entrada es de su empresa y, en esa entrada, la marca está vacía o coincide (o la fila no tiene marca) y el área está vacía o coincide (o la fila no tiene área).

*`documents`*

- `DocumentScopes.where` (L65-68): `empresa/sede/marca` y, solo si la asignación tiene área, `/área`. Las llaves de hoy no cambian, así que ningún documento generado antes del despliegue responde 404.

*`organization`*

- **El agregado `Area`**, copiado de `CommercialBrand`, con todas sus piezas:

  | Capa | Archivos |
  |---|---|
  | Dominio | `Area`; `CreateAreaCommand`, `UpdateAreaCommand`, `ChangeAreaStatusCommand`; `GetAreaByPublicIdQuery`, `GetAreaAggregateByPublicIdQuery`, `GetAreaAggregateForShareQuery`, `GetAreasByIdsQuery`, `SearchAreasByCompanyQuery`, `ExportAreasByCompanyQuery`; `AreaRepository` |
  | Aplicación | `AreaCommandService` y `AreaCommandServiceImpl`; `AreaQueryService` y `AreaQueryServiceImpl`; `AreaView`, `AreaExportRow`, `AreaViewFactory` |
  | Infraestructura | `AreaPersistenceEntity` (`AuditableAbstractPersistenceEntity`, como la marca), su ensamblador, `AreaRepositoryImpl`, `AreaJpaRepository` (`lockByPublicId` `FOR NO KEY UPDATE`, `shareByPublicId` `FOR SHARE`, `shareByIds` en orden de id, búsqueda con `fold_search_text`) |
  | Interfaces | `CompanyAreasController`, `AreasController`, sus recursos y transformadores |

- **Quién escribe un área (Q6):** `anyGrantOf(permiso, g -> g.actsOnWholeCompany(empresa) && g.writesEveryArea())`. Una asignación de sede lee las áreas y no las escribe.
- **Crear y reactivar** bloquean la empresa `FOR SHARE` y rechazan una empresa inactiva (`COMPANY_INACTIVE`), como la marca.
- **La fachada** (`OrganizationContextFacade`):
  - `resolveScope(companyId, branchId, brandId, areaId)`, con el de tres argumentos como `default` que pasa nulo. Bloquea `FOR SHARE` empresa → sede → marca → área; un área de otra empresa responde `AREA_NOT_FOUND`, una inactiva `AREA_INACTIVE`.
  - `fetchAreaReferencesByIds(Set<Long>)` → nombres en lote (`OrganizationReference`).
  - `fetchAreaByPublicId(UUID)`, la lectura simple que compras usará en los filtros de sus listas (la pareja de L176 y L259 de la marca).
  - `fetchAreaByPublicIdForShare(companyId, UUID)` → `Optional<AreaReference>` (id, empresa, si está activa), con `Propagation.MANDATORY`, para las escrituras de compras de la fase 7; el nombre sigue la forma `fetch…ForShare` de la fachada (L241, L268, L278).
  - Su prueba de contrato y la del orden de los candados (`OrganizationContextFacadeImplLockOrderTests`).
- **Desactivar un área** deja vigentes las asignaciones limitadas a ella, con su límite, como la marca (facade L235-237, D-84; `CommercialBrandCommandServiceImpl` L117-137 no las toca): a lo inactivo no se le cuelga nada nuevo, pero lo que ya tenía sigue.
- **El colaborador (Q5):**
  - `AreaAffiliationPolicy`, copiada de `BranchAffiliationPolicy` (L52-60): un área de otra empresa responde `AREA_OTHER_COMPANY`; una inactiva, `AREA_INACTIVE`, salvo que sea la que ya tiene (D-84).
  - Registrar bloquea `FOR SHARE` empresa → sede → área. Modificar bloquea antes al colaborador y lee la empresa sin candado (`EmployeeCommandServiceImpl` L223-227), así que su orden es colaborador → sede → área: no se le agrega el candado de la empresa.
  - El área no decide dónde actúa un permiso (`actsWhere`, L150 y L229-230) ni la zona horaria (L182, L266).
  - El regreso no mira el área (`EmployeeTenureRules` L131-160), aunque sí vuelve a mirar la sede (L156-161): es una excepción a D-84, escrita en D-287, porque el área es informativa.
  - La vista, el recurso y la exportación suman el área (código y nombre). No es un dato personal: no se cifra ni se oculta.
- **Exportación y versión.** El tipo de documento `company-areas` en `OrganizationDocumentKinds` (L22, L42-47), con `DocumentScopes.grantsOf(scope, VIEW_STRUCTURE)`. `OrganizationDataVersions.areas()`, y `areas` en `employees()` y en lo que resuelve la fachada (`JdbcOrganizationDataVersions` L32-50).

*`iam`*

- **Otorgar con área, una o varias (A9):**
  - `GrantRoleResource` (L31-32) suma `areaIds`, una lista: vacía es «todas las áreas»; con varias, crea una asignación por área en la misma transacción, y si una falla no se crea ninguna. `GrantRoleCommand` (L24-25) lleva un `areaId` por asignación.
  - `RoleAssignmentCommandServiceImpl`: `scopeMismatch` (L299-314) exige área vacía a un rol de alcance sistema, y como corre antes que el control de la marca (L139 contra L143), ese caso responde `ROLE_ASSIGNMENT_SCOPE_MISMATCH`, como hoy la marca (`RoleAssignmentsIntegrationTests` L390-396); junto al control de la marca (L143-146), `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` para un rol reservado; `resolveScope` con el área (L147-149). El duplicado (`holds`, L243) ya compara el `RoleScope` completo.
  - `RoleScope` suma `areaId`; su constructor de tres argumentos queda solo para las pruebas, para que ningún código de producción pierda el área sin error. `RoleAssignment` suma `admitsArea` y `Role` suma `isLimitableToArea()`: un rol reservado no se limita a un área.
  - `ExternalOrganizationService.resolveScope` de `iam` (L187-190), que hoy arma `new RoleScope(c, b, brand)`, pasa el área; y suma `fetchAreaReferences`, junto a L205-214.
- **La regla 5.5 y la sesión (análisis, A-H5).** `User.permissionCodesActingOnWholeCompanyAt` (L382-390) se parte en:
  - el que cuenta para otorgar (`RoleDefinitionPolicy.heldFor`), que excluye las asignaciones con sede, con marca o con área: no se otorga lo que solo se tiene en un área;
  - el que lee la sesión, que **sí** cuenta las asignaciones con área, porque fuera de compras ningún control del backend mira el área (A-H11): el inventario de equipos pregunta `actsOnWholeCompanyEveryBrand` (`EquipmentCommandServiceImpl` L266) y las cuentas bancarias `actsOnWholeCompany`. Igual `User.permissionCodesActingOnBrandsAt` (L401-411), que alimenta `brandPermissions`. Si la sesión las excluyera, una persona limitada a Monitoreo perdería «Registrar equipo».
- **Persistencia:** `UserRoleAssignmentPersistenceEntity` (L63-64) suma `areaId` con `updatable = false`; `UserPersistenceAssembler` (L104, L134) lo mapea.
- **Lectura:**
  - `RoleAssignmentView`/`Resource` y `UserRoleResource` suman `areaId` y `areaName` (`RoleAssignmentViewFactory` L49-51 y L69, `RoleAssignmentResourceFromViewAssembler`, `UserResourceFromViewAssembler` L41-42).
  - `RoleView`, `RoleResource` y `RoleDetailResource` suman `limitableToArea` (`RoleViewFactory` L41, `RoleResourceFromViewAssembler` L29 y L50).
  - Los titulares reservados suman la causa `assignmentLimitedToArea` (`AccountDutiesReader`, `RoleCommandServiceImpl` L291-317, `ReservedHoldersView`, `ReservedHoldersResource`): las causas pasan de tres a cuatro.
  - La etiqueta del rol en la exportación de cuentas no cambia: hoy nombra solo la empresa, no la sede ni la marca.
- **El alcance de cada pedido:** `CurrentUserAccessScope` (L115-118) pasa `scope.areaId()` al `PermissionGrant`.
- **La sesión** (`UserQueryServiceImpl` L134-163, `AuthenticatedUserResourceFromEntityAssembler`): `SessionCompanyView` y `SessionCompanyResource` suman `assignments`, una entrada por asignación vigente en esa empresa con `branchId`, `commercialBrandId` y `areaId` (identificadores públicos o nulos; `iam` ya tiene `fetchBranchReferences`) y sus `permissions`, sin los once códigos de accesos y roles (`withoutTheSystemScopeOnes`). Las asignaciones de alcance sistema no aparecen: las cubre `hasSystemPermission`. `wholeCompanyPermissions` y `brandPermissions` siguen como hoy y cuentan también las asignaciones con área (arriba); solo compras pregunta por `assignments`, desde la fase 7.

**API y errores.**

| Método | Ruta | Permiso | Qué |
|---|---|---|---|
| `POST` | `/api/v1/companies/{companyId}/areas` | `configuracion.crear_areas` | `{code, name}` → `201` con el área |
| `GET` | `/api/v1/companies/{companyId}/areas` | `configuracion.ver_estructura` | Lista paginada; busca por código y nombre; filtra por activa |
| `GET` | `/api/v1/companies/{companyId}/areas` (`.xlsx`) | `configuracion.ver_estructura` | `303` al documento `company-areas` |
| `GET` | `/api/v1/areas/{id}` | `configuracion.ver_estructura` | Un área |
| `PUT` | `/api/v1/areas/{id}` | `configuracion.editar_areas` | `{code, name}` |
| `PATCH` | `/api/v1/areas/{id}/status` | `configuracion.desactivar_areas` | `{isActive}` |

- El recurso del área: `id`, `companyId`, `code`, `name`, `isActive`, `createdAt`, `updatedAt`.
- Otorgar un rol suma `areaIds` (opcional; vacío es «todas las áreas») al cuerpo de la ruta de `UserRoleAssignmentsController`; registrar y modificar un colaborador suman `areaId`; las respuestas suman el área donde corresponde.

| Código | Estado | Cuándo |
|---|---|---|
| `AREA_NOT_FOUND` | 404 | El área no existe, o es de una empresa que no se lee, o de otra empresa en una asignación |
| `AREA_CONFLICT` | 409 | El código ya existe en la empresa, también en carrera |
| `AREA_INACTIVE` | 422 | Se le cuelga algo nuevo a un área inactiva |
| `AREA_OTHER_COMPANY` | 422 | El área de un colaborador es de otra empresa |
| `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` | 422 | El rol es reservado (un rol de alcance sistema con área responde `ROLE_ASSIGNMENT_SCOPE_MISMATCH`, como hoy) |

Cada código con su clave `error.<kebab>.message` en `messages.properties` y `messages_es.properties`, y las de validación `validation.area.*` y de exportación `export.areas.*`.

**Frontend.**

- **La pestaña Áreas** de la ficha de la empresa, copiada de las marcas comerciales:
  - `F/organization/domain/model/area.entity.ts` y sus comandos;
  - `infrastructure/api/areas-api-endpoint.ts`, su respuesta, sus pedidos y `assemblers/area-assembler.ts`;
  - `application/areas.store.ts` (copia de `commercial-brands.store.ts`, con `findAreaChoices`);
  - `presentation/components/company-areas/*` y `area-form-dialog/*`;
  - `company-profile-view.ts` suma `'areas'` a `TABBED_SECTIONS` (L52);
  - `organizationAreasEndpointPath` en `src/environments/environment.ts` y en `environment.development.ts`;
  - `OrganizationFacade.loadAreaChoices(companyId)` y `CompanyUnits.areas`, y `organization.facade.spec.ts` (L62-76), que fija los miembros de la fachada.
- **Cuatro pestañas a 375 px** (análisis, B-H4). Los nombres cortos en pantallas angostas ya existen desde la revisión 53 (`company-profile-view.html` L109-126, `es.json` L1456-1458): se suma «Áreas». Una revisión de `DESIGN.md` dice que caben cuatro sin paginación; si no caben, permite las flechas de paginación de Material, y el e2e lo afirma.
- **El área del colaborador:** un campo opcional «Área» en `employee-form-dialog.ts`, copiado del de la sede (L139-150, L163-165, L431-440, L488, L506; html L58-76), que ofrece las áreas activas y conserva la inactiva que ya tiene. La lista no suma columna en esta fase; el área sale en la ficha y en la exportación.
- **`iam`, solo lectura en esta fase:** `UserRole` (`user.entity.ts` L8-33), `users-response.ts`, `user-assembler.ts`, `role-assignment.entity.ts`, `role-assignments-response.ts`, `role-assignment-assembler.ts` y `role.entity.ts` suman el área y `limitableToArea`; la sesión lee `assignments` (`authenticated-user-response.ts`, `sign-in-assembler.ts` L30-31, `iam.store.ts` `toSession` L1240-1259). Las listas de asignaciones muestran el área cuando la hay. El diálogo de otorgar no cambia todavía.
- **Permisos:** `Configuration.CREATE_AREAS`, `EDIT_AREAS`, `DEACTIVATE_AREAS` en `permission-codes.ts` (L71-77) y en el catálogo simulado con sus marcas; en la matriz, la pantalla «Áreas» en Administración, después de Cuentas bancarias.
- **Textos:** los mensajes de los errores nuevos, en `es.json`.

**Pruebas.**

- Backend, nuevas:
  - `AreaTests`, `AreaCommandServiceImplTests`, `AreaQueryServiceImplTests`, `AreaAuditTests` (regla 4.8).
  - `AreaRowLevelSecurityTests`, copiada de `BankAccountRowLevelSecurityTests`.
  - `AreasIntegrationTests`: el ciclo completo, la exportación que sigue el `303`, una empresa que no ve las áreas de otra, una asignación de sede que no escribe.
  - `AreaPermissionsSeedMigrationTests`, copiada de `BankAccountPermissionsSeedMigrationTests`.
  - `PermissionGrantTests` y casos nuevos en `AccessScopeTests` (los de la marca de L175-245 con área, y el caso de dos asignaciones que no se suman).
  - Un caso con área en `DocumentScopesTests`, con el formato de hoy intacto.
- Backend, que cambian:
  - `RowLevelSecurityMigrationTests` (`COMPANY_TABLES` L40-49 con `areas`, su fila en el accesorio, el mapa de inserción, `COMPANY_KEPT_BY_TRIGGER_TABLES` L99-100).
  - `OrganizationNaturalKeysTests`, `OrganizationRowsAreNeverDeletedTests`, `ParentRowLocksTests` (L95-135), `OrganizationOwnershipKeysMigrationTests`, `OrganizationAccessTests` (el administrador de empresa sí escribe áreas), `CompanyIsolationIntegrationTests`, `OrganizationOpenApiTests`.
  - `EmployeeTests`, `EmployeeCommandServiceImplTests`, `EmployeeAuditTests`, `EmployeesIntegrationTests` (otra empresa, inactiva, conserva la inactiva, el regreso no la mira), `EmployeeTenureRulesTests`.
  - `RoleAssignmentsIntegrationTests` (con área, `AREA_NOT_FOUND`, `AREA_INACTIVE`, `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` para un rol reservado, `ROLE_ASSIGNMENT_SCOPE_MISMATCH` para uno de alcance sistema, el mismo rol en dos áreas en un solo pedido y todo o nada si una falla, el duplicado como `ROLE_ASSIGNMENT_CONFLICT`, y una asignación que sigue vigente después de desactivar su área), `RoleAssignmentScopeMigrationTests`, `RoleDefinitionPolicyTests` (lo que se tiene limitado a un área no cuenta), `GrantPolicyTests`, `UserTest`, `OwnSecurityRolesIntegrationTests`, `AuthenticatedUserIntegrationTests` (L423-484: `assignments` en la sesión, y una asignación con área que conserva `stock.crear_equipamiento` en `wholeCompanyPermissions`), `UsersIntegrationTests`, `RoleCommandServiceImplTests`.
  - `PermissionsMigrationTests` (`FINE`, `CATEGORIES` y las matrices de `SUPERADMIN` y `ADMIN_EMPRESA`), `PermissionCatalogConsistencyTests`, `HandOverDatabaseScriptTests`, y `T/AccessFixtures.java` (un ayudante `area(...)` y una asignación con área).
- Frontend: las specs del store, el endpoint, el ensamblador y los componentes nuevos; `company-profile-view.spec.ts`, `employee-form-dialog.spec.ts`, las specs de los ensambladores de `iam`, `iam.store.spec.ts` (la sesión), `permission-codes.spec.ts` y las de la matriz; y una regresión: con una sesión limitada a un área, `equipment-inventory-view` sigue ofreciendo Registrar, Editar y Cambiar estado, y `company-bank-accounts` sus acciones. e2e: `company-profile.spec.ts` (cuatro pestañas), el formulario del colaborador; las imágenes de la ficha de la empresa y la nueva `company-profile-areas`.
- La suite completa del backend, en local.

**Documentación.**

- Diccionario:
  - nueva §6.2.2 `areas`;
  - §2.5: la fila «Una empresa y un área» en «Qué alcanza cada asignación»;
  - §5.3: `employees.area_id`;
  - §5.6: los tres permisos y su lugar en la matriz;
  - §5.7: `user_roles.area_id`, una sección «Limitar el acceso a un área» y sus reglas;
  - §5.8 regla 6: el área;
  - la nota técnica de §2.5 suma `areas` a la lista de tablas con RLS;
  - D-287 pasa a cerradas.
- El Javadoc de `RoleDefinitionPolicy` (L19-23), `AccessScope`, `RoleScope`, `GrantRoleCommand` y `ReservedHoldersView`.
- `contextos/organization.md` (el agregado, sus reglas y la fachada) e `iam.md` (la asignación con área, varias áreas, la sesión); las convenciones de la API (las rutas de las áreas); `procesos-y-pantallas` (P1: crear las áreas de cada empresa; §3.2: la pestaña); la skill `backend-module` (`RecordReach`, `anyGrantOf`, el agregado `Area`) y `frontend-feature` (la pestaña), en sus cuatro copias.

**Orden de los commits.**

- Backend:
  1. V160 y el agregado `Area`, con sus pruebas.
  2. V161, las constantes y su prueba.
  3. `PermissionGrant`, `AccessScope`, `RecordReach` y `DocumentScopes`, con sus pruebas.
  4. V162, la asignación con área y la sesión.
  5. V163 y el área del colaborador.
  6. OpenAPI y mensajes.
- Frontend:
  1. Códigos de permisos y catálogo simulado.
  2. La pestaña Áreas.
  3. El área del colaborador.
  4. La lectura en `iam`.
  5. Los e2e y las imágenes.

**Terminada cuando:** las áreas se crean y se mantienen por empresa, con el aislamiento probado; una asignación con área se otorga y se valida por la API; la sesión trae las asignaciones; nada de compras cambió (caracterización en verde); la suite completa y el CI están en verde.

**Hecha el 23/11/2026 e integrada después de la fase 4** (los hashes, en la línea de Estado; diccionario 1.87), por el orden de las migraciones: la 4 usa V155 y V156, y esta, V157 a V160. Integrada sobre `develop`, que ya tenía las caracterizaciones de compras de la fase 0 (paso 4, `bea8f3ef`), pasaron en local 49 clases y 634 pruebas dirigidas —las de la fase, las cinco caracterizaciones de compras, las de la instalación de la fase 4, `ModularityTests`, `HandOverDatabaseScriptTests` y `RowLevelSecurityMigrationTests`—: nada de compras cambió. El CI de GitHub no corrió ese día: la cuenta no arranca los jobs por un pago fallido o el límite de gasto. D-287 queda construida, y filtra desde la fase 7. Se aparta del plan en once cosas:
1. **Las migraciones**, renumeradas: V157 `areas` (la V160 de arriba), V158 los tres permisos (V161), V159 `user_roles.area_id` (V162) y V160 `employees.area_id` (V163). La fase 7 se renumera al abrir su rama, desde V161.
2. **Otorgar con `areaIds`.** `GrantRoleCommand` lleva un `areaId`, y un comando nuevo, `GrantRoleOnAreasCommand`, reúne las asignaciones de un pedido, que solo difieren en el área. La respuesta sigue al pedido, que el plan no fijaba: sin `areaIds`, la asignación, como antes; con `areaIds`, aunque vacío, el arreglo de las otorgadas en el orden de la lista, y vacío es una asignación con acceso a todas las áreas. Cada asignación se revisa como si fuera sola antes de otorgar ninguna, y la misma área dos veces responde `409 ROLE_ASSIGNMENT_CONFLICT`.
3. **La regla 5.5 y la sesión.** `User.permissionCodesActingOnWholeCompanyAt` se quedó con la sesión y sigue contando las asignaciones con área; lo que cuenta para otorgar es un método nuevo, `permissionCodesActingOnWholeCompanyEveryAreaAt`, que lee `RoleDefinitionPolicy.heldFor`.
4. **La sesión** lista en `assignments` también las asignaciones de las sedes de la empresa; deja fuera una cuya sede, marca o área no se puede leer, o que se queda sin códigos, y las ordena por sede, marca, área y códigos.
5. **El colaborador.** Su recurso trae también el identificador público del área (`areaId`, `areaCode`, `areaName`); la exportación suma «Código de área» y «Área» después de la sede; y la corrección vuelve a mandar el área cada vez: sin ella, el colaborador queda sin área, como con la sede. Los constructores y los comandos de antes, sin área, se conservan.
6. **La fachada** suma `AreaReference` (id, empresa, si está activa) y conserva el constructor de tres argumentos de `ScopeReference`; `fetchAreaByPublicId` lee solo dentro de las empresas donde quien llama lee la estructura (D-99); `AreaRepository` suma `findByIdForShare`.
7. **`OrganizationFacade.loadAreaChoices` y `CompanyUnits.areas` pasan a la fase 7**, con `organization.facade.spec.ts`: en esta fase no tenían consumidor, porque el diálogo de otorgar no ofrece el área y el formulario del colaborador, que es de `organization`, lee las suyas con `AreasStore.findAreaChoices`.
8. **Quién escribe en la pestaña.** No lo decide `actsOnWholeCompany`, que lee `wholeCompanyPermissions` y cuenta las asignaciones con área, sino `actsOnEveryArea`, nuevo en `IamStore` e `IamFacade`: desde el sistema, o por una entrada de `assignments` sin sede y sin área que traiga el código, de cualquier marca; una sesión sin `assignments` escribe las áreas solo desde el sistema. El diálogo deja corregir el código, a diferencia del de la marca, con la ayuda «Letras y números, único en la empresa.».
9. **Las cuatro pestañas a 375 px** no cabían con el ancho mínimo de Material: el grupo de la ficha usa la clase global nueva `mm-tabs-narrow`, que en el teléfono deja cada pestaña en 12 px a cada lado y sin mínimo (DESIGN.md, revisión 101, que enmienda la 13 y la 53). «Áreas» no necesita nombre corto. No se escribieron los e2e (`company-profile.spec.ts`, las cuatro pestañas sin flechas, y el formulario del colaborador) ni las imágenes de la ficha y de `company-profile-areas`: quedan para el lote de imágenes que pida el responsable.
10. **Las pruebas.** `RoleCommandServiceImplTests`, `PermissionCatalogConsistencyTests` y `HandOverDatabaseScriptTests` no cambiaron: las dos últimas revisan el catálogo y el guion de forma general, y la cuarta causa de los titulares reservados la prueban `OwnSecurityRolesIntegrationTests` e `IamOpenApiTests`. Se sumaron `RoleScopeTests`, que falla si el código de producción arma un alcance sin su área, `AreaAffiliationPolicyTests` e `IamOpenApiTests`.
11. **El orden de los commits.** El backend hizo primero V157 y V158 con sus constantes, después `shared` y `documents`, y recién entonces el agregado `Area`, su API y su fachada; el último commit es solo la prueba de OpenAPI, porque los textos y los mensajes llegaron con cada cambio. El frontend no hizo su quinto commit (precisión 9) y sumó dos: uno que corrige dos specs de `iam` y otro que alinea la pestaña y los mensajes con el backend final (`actsOnEveryArea`, `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` y la causa `assignmentLimitedToArea`).

---

### Fase 7 — Compras por área y «Asignar área»

**Objetivo.** Cada documento de compras tiene su área y solo lo ve y lo toca quien la alcanza; una plantilla puede servir a todas las áreas y la orden es del área de quien la emite; lo que ya existe recibe su área con «Asignar área»; la bitácora, las exportaciones, el costo de la unidad y del equipo y el reporte de stock respetan el área; y la pantalla de otorgar un rol ofrece una o varias áreas.

**Lo que la rige.** A3, A4, A6, A7, A9, A10, A11; Q7 a Q11, Q23 y Q27; D-102, D-134, D-224, D-248, D-249, D-280; reglas 4.1, 5.4 y 6.2.

**Migraciones.**

**V164__give_procurement_its_areas.sql** (`Context: procurement`). Abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos. No rellena ninguna fila: `ADD COLUMN` sin valor por omisión no escribe filas, así que no dispara `bump_data_version` ni los `keep_*`.

| Tabla | Cambio |
|---|---|
| `purchase_order_templates` | `area_id BIGINT`, `purchase_order_templates_area_company_fk (area_id, company_id) → areas (id, company_id)`. Dice quién usa la plantilla (vacía: todas las áreas); puede cambiar cuando se quiera, porque solo vale para las órdenes futuras |
| `purchase_orders` | `area_id` con su llave compuesta. Función nueva `enforce_purchase_order_area()`, `BEFORE INSERT OR UPDATE OF area_id, purchase_order_template_id`: al insertar, si la plantilla tiene área, la orden tiene esa; si es general, la orden tiene cualquier área de su empresa o ninguna. Al actualizar, el área solo pasa de vacía a un valor, una vez («Asignar área») |
| `purchase_invoices` | `area_id` con su llave compuesta. Función nueva `enforce_purchase_invoice_area()`, `BEFORE INSERT OR UPDATE OF purchase_order_id, area_id`: con orden, el área es la de la orden. `CREATE OR REPLACE keep_purchase_invoice_fiscal_data` (desde V148) suma `area_id` a lo que no cambia, salvo el paso de vacía a un valor |
| `credit_notes` | `area_id` con su llave compuesta. `CREATE OR REPLACE enforce_credit_note_item_invoice` (V103 L111-144) suma: la factura de cada línea tiene el área de la nota (Q7: lo general cuenta como un valor propio). `CREATE OR REPLACE keep_credit_note_data` (V103 L176-191) suma `area_id`, salvo el paso de vacía a un valor |
| `supplier_payments` | `area_id` con su llave compuesta. `CREATE OR REPLACE keep_supplier_payment_data` desde V110 (L38-56), con `area_id` y la misma salvedad. `CREATE OR REPLACE enforce_payment_application_supplier` (V107 L153-170, `INSERT OR UPDATE OF payment_id, purchase_invoice_id, purchase_order_id`) suma: la factura o la orden de cada aplicación tiene el área del pago; cubre también el paso de un pago de la orden a su factura |
| Líneas, aplicaciones, imputaciones y usos del saldo | Sin columna: el área va en el encabezado |

Las dos funciones nuevas suman su línea `ALTER FUNCTION … OWNER TO erp_mm_owner` en `scripts/database/hand-over-database.sql`. No hay índices nuevos: las listas ya filtran por empresa, y el área es una condición más sobre pocas filas; si una prueba de conteo o un plan de consulta lo pide, se agrega en la misma fase.

**V165__limit_the_change_log_by_the_area_of_its_record.sql** (`Context: audit`). Copia de V61, pero abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos, que V61 no tiene: `change_logs.area_id`, `change_logs_area_requires_company_ck CHECK (area_id IS NULL OR company_id IS NOT NULL)` y la llave compuesta a `areas`, las dos `NOT VALID` y luego `VALIDATE`, para no bloquear la tabla mientras se revisa; y el índice `(company_id, area_id, changed_at DESC)`. La cuenta de la aplicación sigue con `SELECT` e `INSERT`.

**V166__seed_iam_permission_of_the_area_assignment.sql** (`Context: iam`). Plantilla: V109.

- `compras.asignar_area`, categoría `editar`, marcado en `SUPERADMIN`. Además del permiso, «Asignar área» exige una asignación con acceso a todas las áreas de la empresa.

**Backend.**

*Las políticas de `procurement`* (`B/procurement/application/internal/policies`), reescritas con `anyGrantOf`, de modo que una sola asignación cumpla todo. Cada una distingue dos lecturas: la de la empresa («alguna asignación en ella», para lo que todavía no tiene registro: `payableSupplier` L306, las cuentas de pago L179, `PayableQueryServiceImpl` L99, el registro de una factura L131, `CreditNoteReferences` L58, `SupplierPaymentReferences` L130, `PurchaseInvoiceOrderLinks` L53) y la del registro (empresa, marca si la tiene, y área).

| Política | Leer (`compras.ver` o `pagos.ver`) | Escribir |
|---|---|---|
| `PurchaseOrderTemplateAccessPolicy` (L22, L40) | Empresa, `readsBrand(marca)` y `readsArea(área)`. **Usar** una plantilla para emitir es leerla: una general la usan todas las áreas | Toda la empresa; una plantilla sin marca exige `writesEveryBrand()`, una con marca `actsOnBrand`; una con área, `writesArea(área)`; una general, `writesEveryArea()`. Cambiar su área exige escribir en la vieja y en la nueva |
| `PurchaseOrderAccessPolicy` (L28, L47, L57) | `recordReach` para la lista; la marca de la plantilla y el área **de la orden** para una | Toda la empresa, la regla de marca de hoy (L57-61) y `writesArea(área de la orden)` |
| `PurchaseInvoiceAccessPolicy` (L18-44) | Empresa y `readsArea`, sin marca, como hoy | Toda la empresa, la marca de cada línea (hoy; todas las marcas en una factura de equipos) y `writesArea`, en la misma asignación; también la factura atrasada |
| `CreditNoteAccessPolicy` (L20-36) | Empresa y `readsArea` | Toda la empresa, la marca de cada línea y `writesArea` |
| `SupplierPaymentAccessPolicy` (L21-61) | Empresa y `readsArea` | Toda la empresa, las marcas y `writesArea`. Con el saldo: `writesEveryBrand()` y `writesEveryArea()`. Anular exige lo que exigió registrar |
| Lo compartido (Q8) | Todos los que leen compras | Crear un proveedor y agregarle un contacto: cualquier asignación del permiso, con o sin área. Editar o desactivar un proveedor, sus contactos o un tipo de orden: `writesEveryBrand()` y `writesEveryArea()` (`SupplierCommandServiceImpl` L315, `SupplierContactCommandServiceImpl`, los tipos de orden) |

*Las lecturas.*

- **El filtro de cada lista pasa a `RecordReach`:**
  - el de plantillas, que arma `PurchaseOrderTemplateQueryServiceImpl` (L147-168) en `PurchaseOrderTemplateSearchFilter.Reach` (L32), aplicado en `PurchaseOrderTemplateRepositoryImpl` (L135-147);
  - `PurchaseOrderSearchFilter.Reach` (L37), en `PurchaseOrderRepositoryImpl.matching` (L78-100), con `order.area_id` directo;
  - `PurchaseInvoiceSearchFilter.companyIds` (L24), en `PurchaseInvoiceRepositoryImpl` (L148, L251-252);
  - `CreditNoteSearchFilter` (L23), en `CreditNoteRepositoryImpl` (L147, L170-171);
  - `SupplierPaymentSearchFilter` (L27), en `SupplierPaymentRepositoryImpl` (L176, L229-230);
  - Por pagar: el alcance que arma `PayableQueryServiceImpl` (L92-95), que hoy pasa un `Set<Long>` de empresas al puerto `PayableFactsReadRepository.openInvoices` (`JdbcPayableFactsReadRepository` L166-210), y su acción `mayPay` (L156).

  Cada entrada se vuelve `company_id = :cN`, con la marca si la tiene, y con `area_id IS NULL OR area_id = :aN` **solo si la asignación tiene área**; sin área no se agrega condición. Las entradas se unen por `OR`, y `Every` no filtra. Nunca se enlaza un nulo: en SQL nativo, `(:a IS NULL OR …)` falla en PostgreSQL con `NamedParameterJdbcTemplate` («could not determine data type of parameter»), y el código ya lo esquiva así (`PurchaseInvoiceJpaRepository` L58). Los repositorios con Criteria ya arman un predicado por entrada con valores literales.
- **Dos filtros nuevos en cada lista:** `areaId` (un área) y `generalOnly` (solo lo general). `withPaymentStatus` (`PurchaseInvoiceQueryServiceImpl` L125), que copia el filtro campo por campo, copia los nuevos. Las exportaciones pasan los mismos criterios (`PurchaseInvoicesController` L194-198, `PayablesController` L124-126 y los demás).
- **Las lecturas de un registro** responden 404 si su área no se lee:
  - la factura por identificador, con su archivo y su XML (`PurchaseInvoiceQueryServiceImpl` L194-200);
  - la posición de pago de una factura (`SupplierPaymentQueryServiceImpl` L281, L284);
  - el pago y su archivo (L153, L161);
  - la nota y su archivo (`CreditNoteQueryServiceImpl` L137);
  - la plantilla y su documento (L139);
  - la orden y su PDF (`readOrder` L156-162).
- **Los documentos por pagar de un proveedor** (`GET /suppliers/{id}/payables`, `SupplierPaymentQueryServiceImpl` L211-274), la fuente del formulario de pago (`F/…/procurement-api.ts` L394). Hoy devuelve las facturas sin filtrar (L225), las órdenes filtradas solo por marca (L235-238) y los orígenes del saldo sin filtrar (L227). Filtra facturas y órdenes por área; `PayableInvoiceView` y `PayableOrderView` suman `area`; acepta `areaId` y `generalOnly`, para que el formulario se quede en una sola área; y los orígenes del saldo solo viajan a una asignación con acceso a todas las áreas (Q9), porque el origen «sin nota» (`creditNoteId` nulo) no tiene área.
- **La verificación del VIN** (`PurchaseInvoiceVinChecks`): sigue respondiendo `PURCHASEINVOICE_UNIT_CONFLICT`, pero `CurrentInvoiceView` (L134-139) —número, proveedor e identificador de la factura vigente— viaja solo si se lee el área de esa factura.
- **Los cálculos por proveedor leen todo** (análisis, A-H4): `supplierFactsOf`, `PayablePositions` y `SupplierFacts` no filtran; el área filtra solo las facturas, órdenes y orígenes del saldo que se devuelven.
- **El saldo a favor (Q9)** y el total que se le debe al proveedor (`SupplierBalanceView.owed`, que suma todas las áreas) viajan solo si `anyGrantOf(pagos.ver, g -> g.actsOn(empresa) && g.writesEveryArea())`; si no, `null`.

*Las vistas y las acciones.*

- Toda vista de plantilla, orden, factura, nota y pago suma el área (identificador público y nombre), con las referencias en lote de `ExternalOrganizationService` (las de la marca están en L95, `fetchCommercialBrandReferencesByIds`; se suma la de las áreas).
- Las acciones (`PurchaseOrderTemplateViewFactory` L178-209, `PurchaseOrderAllowedActions` L31-43, `PurchaseInvoiceAllowedActions` L37-85, `CreditNoteAllowedActions` L32-43, `SupplierPaymentAllowedActions` L34-45, `SupplierPaymentScopes` L42-70, `SupplierViewFactory` L116) preguntan a las políticas nuevas. «Registrar pago» desde una factura (`PurchaseInvoiceAllowedActions` L80-81) y desde el proveedor (`SupplierViewFactory` L116) exige `pagos.registrar` que escriba en el área del documento.
- **Los montos de pago en la factura** (`appliedTotal`, `owed`, `toBalance`, `PurchaseInvoiceViewFactory` L196) exigen `pagos.ver` que lea el área de la factura, en la misma asignación.
- Con Q7, las notas, pagos y aplicaciones anidados comparten el área de su documento. El único cruce que queda es el saldo, que es del proveedor: los rastros de un uso del saldo (`CreditNoteViewFactory.balanceView` L256-265, `BalanceUseContexts`) y las notas que consumió un pago con saldo (`SupplierPaymentViewFactory.balanceSources` L329-345) muestran el documento de origen solo si su área se lee, y si no, solo el monto.

*Los comandos.*

- **Plantillas** (`PurchaseOrderTemplateCommandServiceImpl`):
  - crear (L82-195) resuelve el área como la marca (L89-103), con `fetchAreaByPublicIdForShare` después de la marca, y rechaza un área inactiva; vacía es «la usan todas las áreas», y solo la crea quien tiene acceso a todas las áreas;
  - modificar (L216-373) cambia el área **aunque la plantilla tenga órdenes**: cada orden guarda su propia área, así que el cambio solo vale para las futuras. No hay `PURCHASEORDERTEMPLATE_AREA_IN_USE`;
  - activar (L466-488) exige el área activa;
  - el agregado suma `changeArea` (junto a `changeBrand`, L167);
  - `PurchaseOrderTemplateChanges` suma el área a su `Snapshot` (L58-93) y anota el cambio de área como el de la marca (L112-116), con la fila marcada con el área nueva.
- **Órdenes** (`PurchaseOrderCommandServiceImpl`):
  - **crear (L177) toma el área de quien emite (Q27):** con una plantilla de un área, esa (otra responde `PURCHASEORDER_AREA_MISMATCH`); con una plantilla general, el `areaId` del pedido, que tiene que ser un área donde quien emite escribe; si no lo manda, el área de quien emite cuando está limitado a una sola; si tiene varias, es obligatorio (`PURCHASEORDER_AREA_REQUIRED`); y vacía, «General», solo para quien tiene acceso a todas las áreas;
  - emitir comprueba el área activa con candado compartido, junto a `checkIssuingScope` (`PurchaseOrderIssuing` L61-87);
  - `writeRefusal` (L380-390), que usan modificar, enviar, cumplir y anular, recibe el área de la orden.
- **Facturas** (`PurchaseInvoiceCommandServiceImpl`):
  - **registrar (L106-244):**
    - con orden, lee el área de la orden sin candado antes de `PurchaseInvoiceUnits.resolve` (L177), porque no cambia, y deja el candado de la orden donde está (L185);
    - un `areaId` en el pedido distinto del de la orden responde `PURCHASEINVOICE_AREA_MISMATCH`;
    - sin orden, toma el `areaId` del pedido con la regla de la orden, con el área activa;
    - **la marca y el área se preguntan en la misma asignación**, donde hoy se pregunta la marca: `PurchaseInvoiceUnits` L108 (`registers`) y L114 (`registersOld`). Preguntarlas por separado repetiría en la escritura el error A-H1: «marca X sin área» más «sin marca, área P» registrarían una factura de X en P. Una factura de equipos, con todas las marcas (Q14).
  - **modificar (L285-365):** vincula solo una orden con la misma área (L312-332 y `PurchaseInvoiceOrderLinks.order` L51-63); una orden de un área que no se lee responde 404, para no revelar que existe; una legible de otra área, `PURCHASEINVOICE_AREA_MISMATCH`. Desvincular conserva el área.
  - **anular (L247-273):** con el área.
- **Notas** (`CreditNoteCommandServiceImpl`, `CreditedLines`):
  - el área es la común de las facturas bloqueadas (`lock` L95, `refusal` L143-182); dos áreas distintas responden `CREDITNOTE_AREAS_MIXED`;
  - el control de marca (L168-169) lleva el área;
  - la nota nueva lleva el área (L139).
- **Pagos:**
  - el área es la común de sus facturas y órdenes (`SupplierPaymentApplications.reach` L324-341, `targets` L198-214); dos áreas responden `SUPPLIERPAYMENT_AREAS_MIXED`;
  - el saldo exige una asignación con acceso a todas las áreas;
  - un pedido repetido compara el área (`SupplierPaymentRequests.repeated` L167-180);
  - el paso de un pago de la orden a su factura (`OrderPaymentTransfers` L81) exige la misma área, que el disparador también impone;
  - anular (`SupplierPaymentAnnulments` L80-92) y reemplazar el comprobante (`SupplierPaymentVoucherReplacements` L51-66) llevan el área.
- **Un área inactiva** no bloquea lo que hereda su área de un documento que ya la tiene —la factura de una orden, la nota y el pago de sus facturas—, para que los ciclos abiertos se cierren; sí bloquea lo que nace eligiéndola: una plantilla, una orden y una factura sin orden.
- **La carga inicial** (`InitialMigrationImportCommandServiceImpl.scope`, L159-192): exige una asignación con acceso a todas las áreas y escribe todo general (`MigratedInvoiceWriter` L131, `MigratedCreditNoteWriter` L135, `MigratedPaymentWriter` L181).

*«Asignar área» (A11, Q23).*

- `POST /api/v1/purchase-area-assignments` `{areaId, purchaseOrderIds, purchaseInvoiceIds}`, con `compras.asignar_area` y una asignación con acceso a todas las áreas de la empresa. Varias órdenes y facturas van juntas, porque una nota o un pago puede abarcar documentos de más de una.
- Solo pasa documentos **generales** a un área: un documento que ya tiene área responde `PURCHASEAREAASSIGNMENT_ALREADY_SET`; una factura con orden no se nombra sola, porque toma el área de su orden (`PURCHASEAREAASSIGNMENT_INVOICE_HAS_ORDER`).
- En una transacción, con candados en orden de id —órdenes, después facturas, notas y pagos—, le pone el área a cada orden nombrada, a sus facturas, a las facturas nombradas, a las notas de todas ellas y a los pagos de esas facturas y órdenes, en ese orden, para que los disparadores de coherencia de V164 pasen.
- Si una nota o un pago abarca un documento que no entra en el conjunto, o uno de otra área, se rechaza todo y nombra cada documento (`PURCHASEAREAASSIGNMENT_DOCUMENTS_MIXED`).
- Una fila de la bitácora por documento, con el área nueva. Las filas viejas quedan generales: la cuenta de la aplicación solo inserta en `change_logs`, así que la historia anterior de esos documentos la siguen viendo todas las áreas. Es un límite aceptado.
- Las plantillas no entran: pueden quedar generales, como formatos comunes, o recibir un área que diga quién las usa, cuando se quiera.

*La bitácora (`audit`).*

- `RecordChangeLogCommand` (L25-36) suma `areaId`, conservando el constructor de once argumentos (37 usos en 19 archivos); `ChangeLogEntry` («el área exige una empresa»), su entidad y su ensamblador.
- Los escritores de compras pasan el área:
  - `PurchaseOrderTemplateChanges` (L106-107, L195);
  - `PurchaseInvoiceCommandServiceImpl` (L353) y `PurchaseInvoiceAnnulments` (L57);
  - `CreditNoteCommandServiceImpl` (L238) y `CreditNoteAnnulments` (L181);
  - `SupplierPaymentAnnulments` (L148) y `SupplierPaymentVoucherReplacements` (L85);
  - `OrderPaymentTransfers` (L158, L161);
  - «Asignar área».

  Los proveedores no llevan área (`SupplierCommandServiceImpl` L197 queda como está).
- `ChangeLogReach.Some` (L37) pasa a las entradas de `RecordReach`; `ChangeLogAccessPolicy.reach` (L35-48) y `ChangeLogRepositoryImpl.reached` (L150-165) la siguen. La búsqueda suma el filtro `areaId`. `seesValuesOf` (L62-69) sigue solo con la marca, y `ChangeLogHistoryLookup` no cambia: solo lo usan `SupplierPaymentRequests` e `InvoicePaymentPositions`, sobre registros que ya autorizaron.

*El costo y el reporte.*

- **El costo de una unidad** (`UnitPurchaseCostQueryServiceImpl` L39-51): después de `seesCosts` de `fleet` (empresa y marca), exige `anyGrantOf(stock.ver_costos, g -> g.actsOn(empresa) && g.actsOnBrand(marca) && g.readsArea(área de la factura))`; si no, `403 ACCESS_DENIED`, nunca un costo vacío, que se leería como «sin factura». Las notas, pagos y usos anidados de otra área no viajan (`UnitPurchaseCosts.costs` L98-158, L205 en adelante).
- **El costo de un equipo** (`EquipmentPurchaseCostsController`, fase 2) exige además `readsArea` del área de su factura, con la misma regla.
- **La fachada de compras:** `ProcurementContextFacade.UnitPurchaseCost` (`interfaces/acl/UnitPurchaseCost.java` L15-18) suma `areaId`, y lo anidado si se filtra. `reporting` lo recibe por `ExternalProcurementService.toPurchase` (L34-43) en `ReportUnitPurchase` (L19).
- **El reporte de stock** (`StockRows.of` L45-104):
  - una bandera `seesPurchase`: una asignación de `reportes.ver` que lee la empresa, la marca de la unidad y el área de su factura. Si es falsa, quedan vacías todas las columnas de compras, también el número de la orden (`JdbcReportsReadRepository.findStockUnits` L174-177);
  - `seesCosts` pide `stock.ver_costos`, la marca y el área en la misma asignación;
  - el reporte de diferencias por inicial (L149-169) filtra igual;
  - la versión de los datos no cambia.
- **El reporte de brecha de días** (`ReportQueryServiceImpl`, `gapOf`) sigue sin filtrar: muestra la fecha base de unidades de otras áreas, sin montos ni números, con el criterio de A-H10.
- **`fleet` no cambia:** el inventario sigue mostrando si una unidad está pagada y si no tiene factura vigente, sin montos (análisis, A-H10).

**API y errores.**

- Las plantillas reciben `areaId` al crear y modificar; las órdenes y las facturas sin orden, al registrar.
- Plantillas, órdenes, facturas, notas, pagos, Por pagar y los documentos por pagar de un proveedor responden el área (`area: {id, name}` o nulo) y aceptan los filtros `areaId` y `generalOnly`.
- El saldo del proveedor y su total adeudado pueden venir nulos, y la OpenAPI lo dice.
- Las exportaciones de facturas, notas, pagos y Por pagar suman la columna «Área» (Q11) y suben su constante de diseño (`ProcurementDocumentKinds` L39 pagos, L45 facturas, L51 Por pagar). Las notas no tienen constante (L114-127): se crea `CREDIT_NOTES_LAYOUT = "credit-notes-layout=1"`, para que una exportación generada antes del despliegue no se sirva con las columnas viejas.
- La bitácora acepta `areaId` y responde el área de cada fila.

| Código | Estado | Cuándo |
|---|---|---|
| `PURCHASEORDER_AREA_MISMATCH` | 422 | El área pedida no es la de la plantilla, o no es un área donde quien emite escribe |
| `PURCHASEORDER_AREA_REQUIRED` | 422 | Con una plantilla general, quien tiene varias áreas no eligió una |
| `PURCHASEINVOICE_AREA_MISMATCH` | 422 | El área pedida no es la de la orden, o se vincula una orden legible de otra área |
| `CREDITNOTE_AREAS_MIXED` | 422 | Una nota junta facturas de áreas distintas |
| `SUPPLIERPAYMENT_AREAS_MIXED` | 422 | Un pago junta documentos de áreas distintas |
| `PURCHASEAREAASSIGNMENT_ALREADY_SET` | 422 | «Asignar área» a un documento que ya tiene área |
| `PURCHASEAREAASSIGNMENT_INVOICE_HAS_ORDER` | 422 | «Asignar área» a una factura con orden, que toma la de su orden |
| `PURCHASEAREAASSIGNMENT_DOCUMENTS_MIXED` | 422 | Una nota o un pago abarca documentos fuera del conjunto o de otra área; los nombra |

Se reutilizan `AREA_NOT_FOUND`, `AREA_INACTIVE` y `ACCESS_DENIED`. Ningún código nuevo termina en `_CONFLICT` (regla 3.3).

**Frontend.**

- **Lo que dejó la fase 6** (fase 6, precisión 7): `OrganizationFacade.loadAreaChoices(companyId)` y `CompanyUnits.areas`, con `organization.facade.spec.ts`, que fija los miembros de la fachada; `AreasStore.findAreaChoices` ya da las áreas activas y la inactiva que un registro conserva. Ya están `scopeOf` con el área, la traducción de `ROLE_ASSIGNMENT_AREA_NOT_ALLOWED` en `grantRoleErrorKey` e `IamFacade.actsOnEveryArea`, que lee `assignments`.
- **`iam`:**
  - el diálogo de otorgar un rol (`role-assignments-dialog.ts`: formulario L149-155, `brandAllowed` L202, `scopeOf` L302-309, `grant` L435-447, `fitScopeToRole` L489-522, `loadUnits` L525-539; el campo de marca del html L182-193 es el molde) suma «Áreas», una selección **múltiple** con «Todas las áreas» por omisión (A9). Se habilita con una empresa elegida y un rol que admite área, y propone el área del colaborador cuando se abre desde él. Envía `areaIds`;
  - `access-scope-options.ts` y `organization-access-scope-options.ts` (con su spec) suman las áreas;
  - los errores nuevos van en `grantRoleErrorKey` (`iam.store.ts` L1555-1605).
- **Los ayudantes de la sesión** (`iam.store.ts` L378-425), que leen `assignments`, también en `IamFacade` (`iam.facade.ts` L312-334), porque `procurement-writes.ts` solo ve la fachada:
  - `readsArea(código, empresa, área)` y `writesOnArea(código, empresa, área, marcas)` preguntan, en una misma asignación, la empresa, la marca y el área;
  - `writableAreas(código, empresa)` da las áreas donde se escribe, para los selectores;
  - `actsOnEveryBrandAndArea(código, empresa)` sirve para el saldo, lo compartido y «Asignar área»;
  - `actsOnEveryBrand` y `actsOnWholeCompany` siguen como hoy, para lo que no mira el área.
- **`procurement-writes.ts`** (L40-80) usa los ayudantes nuevos.
- **Plantillas:**
  - el formulario suma «Área» —quién la usa; vacío es «Todas las áreas»—, copiado del selector de la marca (L153, L176, L320-341, L849-910); ofrece las áreas donde se escribe, y «Todas las áreas» solo a quien tiene acceso a todas;
  - el área se puede cambiar siempre, sin el candado de `brandLocked`;
  - la lista suma la columna «Área» y su filtro.
- **Órdenes:** el primer panel (html L77-81) muestra el área de la plantilla si la tiene; con una plantilla general, un selector con las áreas de quien emite (fijo si tiene una sola, con «General» solo para quien tiene acceso a todas). La lista filtra por área.
- **Facturas:**
  - con orden, el área se muestra fija; `searched-sent-orders.ts` trae el área de cada orden;
  - sin orden, un selector «Área», con la regla de la orden;
  - el diálogo de modificar busca solo órdenes de la misma área;
  - la ficha muestra el área y la lista filtra.
- **Notas:** el área se muestra, derivada de las facturas, y la búsqueda de facturas se acota a la misma área.
- **Pagos:**
  - el área sale de los documentos, y la búsqueda (`GET /suppliers/{id}/payables`) se acota a una;
  - la opción del saldo aparece solo con `actsOnEveryBrandAndArea` (`supplier-payment-form-view.ts` L550-553).
- **Por pagar y la ficha del proveedor:** el filtro de área; el saldo y el total adeudado no se muestran si no viajan. Crear un proveedor y agregar un contacto se ofrece a todo comprador; editar y desactivar, solo con acceso a todas las áreas (Q8).
- **«Asignar área»:** en las listas de Órdenes y Facturas, para quien tiene `compras.asignar_area` y acceso a todas las áreas: elegir filas generales y un área; el diálogo muestra las notas y los pagos que entran y, si se rechaza, los documentos que lo impiden.
- **Bitácora** (`change-logs-view.ts`): el filtro de área, habilitado después de elegir la empresa (L229-250); `FIELD_OPTIONS` suma `area` para las plantillas (L74-97); `ChangeLogFilters` suma `areaId` (`change-logs-api-endpoint.ts` L17-24, L44-54).
- **Los filtros de área** se arman en cada lista con `OrganizationFacade.loadAreaChoices`, sin un componente compartido nuevo: la lista cerrada de piezas prestadas (frontend-rules 1.2) no cambia. En Plantillas, Órdenes, Pagos y Por pagar van en la barra. Facturas y Notas tienen su barra en una sola línea (`E/toolbar-rows.spec.ts` L21-22) y no existe un panel de más filtros: el gestor de datos pliega sus filtros solo en el teléfono (`data-manager.html` L4-26). Una revisión de `DESIGN.md` define ese patrón —«Más filtros»— antes de usarlo.

**Pruebas.**

- Backend, nuevas:
  - **`ProcurementAreaIsolationIntegrationTests`**, copiada de `ProcurementCompanyIsolationIntegrationTests`. Para cada documento, también los de equipos: quien está limitado a un área lee lo suyo y lo general; escribe solo lo suyo; no escribe lo general; no ve el saldo ni el total adeudado; la exportación respeta el área; los documentos por pagar de un proveedor, la verificación del VIN y los montos de pago de la factura no muestran otra área.
  - **La regresión de una sola asignación**, en plantillas, órdenes, bitácora y en el registro de una factura: «marca X sin área» y «sin marca, área P» no leen ni escriben «marca Y, área Q».
  - **El área de la orden (Q27):** plantilla de un área, plantilla general con una, varias y todas las áreas.
  - **«Asignar área»:** la cascada completa, el rechazo por mezcla con sus documentos, una orden con nota sobre dos órdenes nombradas juntas, el documento que ya tiene área, la factura con orden y la fila de la bitácora.
  - **Los disparadores:** el área de la orden es la de su plantilla cuando la tiene y no cambia salvo de vacía a un valor; la de la factura es la de su orden; las facturas de una nota tienen su área; los documentos de un pago tienen su área; lo fiscal incluye el área con su única salvedad.
  - **La migración** (estilo `BalanceUsesMigrationTests`).
  - **El SQL de Por pagar** con entradas con y sin área, sin enlazar un nulo.
  - **El costo de una unidad y de un equipo** (`403` por área y lo anidado), **el reporte de stock** (columnas vacías por área) y **Por pagar** por área.
  - **Lo compartido (Q8):** quien tiene área crea un proveedor y un contacto, y no edita ni desactiva.
- Backend, que cambian:
  - los cinco `*AccessPolicyTests`, los cuatro `*AllowedActionsTests`, los cinco `*AuditTests` de compras;
  - `PurchaseInvoiceFiscalTriggerTests`, `CreditNoteTriggerTests`, `PurchaseInvoiceSchemaTests`, `SupplierPaymentSchemaTests`;
  - los conteos de sentencias: `PurchaseInvoiceRegistrationStatementsTests` (hoy 64), `SupplierPaymentRegistrationStatementsTests`, `CreditNoteRegistrationStatementsTests`, `PurchaseInvoiceListStatementsTests`, `SupplierPaymentListStatementsTests`, `BalanceTraceStatementsTests`. Cada sentencia nueva se justifica: la lectura compartida del área suma una;
  - las de integración y exportación de cada lista, `PayablesIntegrationTests`, `SupplierBalancesIntegrationTests`, `VehiclePurchaseCostIntegrationTests`, `UnitPurchaseCostsTests`, `ProcurementContextFacadeTests`, `InitialMigrationIntegrationTests`, `ProcurementOpenApiTests`;
  - en `audit`: `ChangeLogAccessPolicyTests`, `ChangeLogsSearchFilteringTests`, `ChangeLogsControllerTenantTests`, `ChangeLogEntryTests`, `ChangeLogCommandServiceTests`;
  - en `reporting`: `ReportsIntegrationTests`, `StockRowsTests`;
  - `PermissionsMigrationTests`, `PermissionCatalogConsistencyTests` y `HandOverDatabaseScriptTests`.
- Las pruebas de caracterización de la fase 0 pasan sin cambios.
- Frontend: las specs de los formularios y las vistas, `procurement-writes.spec.ts`, `iam.store.spec.ts`, `iam.facade.spec.ts`, `role-assignments-dialog.spec.ts` (varias áreas), `organization-access-scope-options.spec.ts`, `change-logs-view.spec.ts` y el diálogo de «Asignar área»; e2e `purchase-order-templates`, `purchase-orders`, `invoice-form`, `purchase-invoices`, `credit-notes`, `supplier-payments`, `suppliers`, `change-log` y `access`, con sus mocks; las imágenes de las pantallas que suman un campo.
- La suite completa del backend, en local.

**Documentación.**

- Diccionario:
  - §2.5: las reglas del área en compras;
  - §9.1 a §9.5: las columnas `area_id` y sus reglas, la plantilla que dice quién la usa y el área de la orden;
  - §10.1 a §10.4: el área del pago y el saldo;
  - §13.1: `change_logs.area_id`;
  - «Asignar área» y su permiso (§5.6);
  - D-288 pasa a cerradas.
- `contextos/procurement.md`, `audit.md` y `reporting.md`, con sus reglas nuevas.
- Las convenciones de la API: los filtros, el área en los recursos y la ruta de «Asignar área».
- `procesos-y-pantallas`: P2 (una o varias áreas al otorgar un rol), P4, P5, P10, P11, §3.4 y un proceso nuevo, «Darle área a lo que ya existe».
- `DESIGN.md`: el patrón «Más filtros».
- Las skills `backend-module` (`RecordReach` en compras, el SQL sin nulos enlazados) y `architecture-review` (la regla de la lista de asignaciones), en sus cuatro copias.

**Antes de otorgar la primera asignación con área:** quien tiene acceso a todas las áreas da su área, con «Asignar área», a las órdenes y facturas sin orden que ya existen y son de un área; lo que es de todas queda general. Las plantillas pueden quedar generales: la orden toma el área de quien la emite.

**Orden de los commits.**

- Backend:
  1. V164, V165 y V166, con las pruebas de sus disparadores.
  2. El área en la bitácora.
  3. Las políticas, `RecordReach` y las lecturas.
  4. Las vistas y las acciones.
  5. Plantillas y órdenes, con el área de quien emite.
  6. Facturas.
  7. Notas.
  8. Pagos, saldo, Por pagar y los documentos por pagar del proveedor.
  9. Lo compartido.
  10. «Asignar área».
  11. El costo, la fachada y el reporte.
  12. Exportaciones, OpenAPI y mensajes.
- Frontend:
  1. La sesión, los ayudantes, la fachada y el diálogo de otorgar con varias áreas.
  2. Plantillas.
  3. Órdenes.
  4. Facturas.
  5. Notas.
  6. Pagos, Por pagar y proveedor.
  7. «Asignar área».
  8. La bitácora.
  9. El patrón «Más filtros» y su revisión de `DESIGN.md`.
  10. Los e2e y las imágenes.

**Terminada cuando:** las pruebas de aislamiento y de caracterización están en verde; la suite completa y el CI también; y una prueba de punta a punta en `dev` muestra que una persona limitada a Monitoreo ve solo lo de Monitoreo y lo general, emite con la plantilla común una orden de Monitoreo, y que las órdenes existentes recibieron su área con «Asignar área».

**Hecha el 23/11/2026 e integrada** (backend y frontend en `feature/phase-7-procurement-by-area`, integrados en `develop` y subidos, primero el backend, con los hashes de la línea de Estado; documentación en `docs/phase-7-procurement-by-area`, diccionario 1.88). En el backend, integrado después de la fase 6 y sobre las de las fases 4 y 6, pasaron en local 31 clases y 276 pruebas dirigidas; el CI de GitHub no corrió, porque la cuenta no arranca los jobs por un pago o un límite de gasto. D-288 queda construida, backend y pantallas, y D-287 completa: compras respeta el área de la asignación, y el diálogo de otorgar la ofrece. El backend se aparta del plan en catorce cosas, y el frontend lo dice su párrafo (abajo):
1. **Las migraciones**, renumeradas: V161 `give_procurement_its_areas` (la V164 de arriba), V162 `limit_the_change_log_by_the_area_of_its_record` (V165) y V163 `seed_iam_permission_of_the_area_assignment` (V166), que se detiene si falta la marca del superadministrador. En V161, los `keep_*` de facturas, notas y pagos dejan pasar el paso de vacía a un área **también en un documento anulado**, cuando no cambia nada más que el área y su revisión: así «Asignar área» alcanza toda la historia de una orden. `enforce_purchase_order_area()` no vuelve a preguntar la plantilla cuando una actualización la conserva, y `enforce_purchase_invoice_area()` corre también al vincular la orden después. La siguiente migración libre es V164.
2. **Los campos son planos:** `areaId` y `areaName` en cada recurso —plantilla, orden, factura, nota, pago, fila de Por pagar, factura y orden de `/suppliers/{id}/payables` y fila de la bitácora—, no `area: {id, name}`, como la marca.
3. **Un código nuevo, `PURCHASEINVOICE_AREA_REQUIRED`:** una factura sin orden elige su área con la regla de la orden de una plantilla general (Q27), y quien escribe en varias áreas sin nombrar una lo recibe, como `PURCHASEORDER_AREA_REQUIRED`. `PURCHASEINVOICE_AREA_MISMATCH` cubre también un área que quien registra no escribe. Los códigos nuevos son nueve, con sus textos en `messages.properties` y `messages_es.properties`; se reutilizan `AREA_NOT_FOUND`, `AREA_INACTIVE`, `ACCESS_DENIED` y `VALIDATION_ERROR`.
4. **El filtro de área exige leer la estructura:** `areaId` lo resuelve `organization` (`fetchAreaByPublicId`) para quien tiene `configuracion.ver_estructura` en su empresa, como se listan las áreas; uno que no existe o no se lee así no encuentra nada, y `areaId` con `generalOnly` tampoco. Los siete roles de la semilla lo tienen; un rol propio sin él no filtra por área. La bitácora acepta solo `areaId`, con la misma resolución (`AuditCompanyLookupPort.resolveAreaId`).
5. **«Asignar área»** exige que los documentos sean de **una sola empresa** (`400 VALIDATION_ERROR`), admite **de 1 a 500**, órdenes y facturas juntas, y alcanza **también a los anulados**. Un documento que no se lee responde su `404`; los rechazos propios nombran los documentos en `context.documents` y, el de la mezcla, cada nota o pago con lo que abarca fuera en `context.spans`. Responde `{areaId, areaName, purchaseOrderIds, purchaseInvoiceIds, creditNoteIds, supplierPaymentIds}`. El área se escribe por JDBC (`PurchaseAreaAssignmentStore`), solo donde sigue vacía, en el orden órdenes, facturas, notas y pagos; cada documento deja una fila de la bitácora con el campo `area` y el nombre del área.
6. **`registerPayment` del proveedor no cambió:** «Registrar pago» desde su ficha sigue pidiendo `pagos.registrar_pago` en toda la empresa y todas las marcas, con área o sin ella; el formulario acota los documentos a un área con `/suppliers/{id}/payables?areaId=`. Desde una factura, en cambio, pide el área de la factura, como decía el plan.
7. **La caracterización por marca de la fase 0** pasa sin cambios, y tres cosas que dejó a la vista quedan así: un pago aplicado a una orden de otra marca sigue mostrando el número de esa orden a quien lee los pagos, porque los pagos se leen por empresa y área, no por marca; el costo de una unidad exige `stock.ver_costos` con su marca y el área de su factura en una misma asignación, o responde `403`, y dónde se usó una nota se dice por el área de la factura, no por su marca; y la bitácora de una plantilla guarda el área que la plantilla tiene después del cambio, como la marca, mientras las filas viejas siguen generales.
8. **El área de la orden** se bloquea al final de la cadena empresa → marca → área (`checkIssuingScope`, con `resolveScope` de cuatro argumentos), y una plantilla de un área inactiva tampoco emite, porque la orden es nueva. La factura con una orden de un área que no se lee responde `404 PURCHASEORDER_NOT_FOUND` al registrarla, no solo al cambiar su orden.
9. **Un envío repetido de un pago** no compara el área: responde el pago ya registrado solo a quien lee su empresa y su área. Lo aplicado a una orden pasa a su factura solo en la misma área, lo que ya garantiza la base.
10. **Lo compartido (Q8)** pide todas las áreas también para reactivar un contacto, que usa la ruta de su estado, y para escribir un tipo de orden; la ficha del proveedor ofrece «Editar» y «Desactivar» con esa misma regla (`SharedProcurementPolicy`).
11. **La bitácora** conserva el constructor de once argumentos de `RecordChangeLogCommand`, sin área, para todos los contextos menos compras (Q10). `ChangeLogReach.Some` pasó de «empresas con todas las marcas» y «marcas por empresa» a una entrada por asignación, como `RecordReach`, y cada fila responde su área resuelta en lote (`resolveAreas`, `AuditArea`).
12. **El reporte de stock** vacía las columnas de compra con una bandera propia (`seesPurchase`), que pide `reportes.ver` con la marca de la unidad y el área de su factura en una misma asignación; sus montos siguen a `stock.ver_costos` con la misma regla, y la diferencia por inicial deja fuera la unidad que no se lee así. El diseño del stock no cambió (`stock-columns=2`). Las exportaciones de compras van en pagos 4, facturas 4, Por pagar 2 y `credit-notes-layout=1`, con «Área» después de «Empresa».
13. **Los commits del backend son cinco, no doce:** `53f3213f`, las caracterizaciones de compras por marca de la fase 0, paso 4, traídas a la rama; `494d116b`, V161 a V163 con sus pruebas; `9074a887`, el área en la bitácora; `e46077c1`, compras por área —políticas, lecturas, escrituras, vistas, acciones, filtros, saldo, exportaciones, el costo y el reporte, los pasos 3 a 9, 11 y 12 del plan—; y `43d6482b`, «Asignar área».
14. **Las pruebas.** Nuevas: `ProcurementAreaTriggerTests`, `ProcurementAreasMigrationTests`, `AreaAssignmentPermissionSeedMigrationTests`, `ChangeLogsAreaReachTests`, `ProcurementAreaPoliciesTests`, `ProcurementAreaIsolationIntegrationTests` y `PurchaseAreaAssignmentIntegrationTests`, y un caso de Por pagar por entrada en `PayableFactsReadRepositoryTests`. Cambiaron los cinco `*AccessPolicyTests` de compras, `ChangeLogEntryTests`, `ChangeLogAccessPolicyTests`, las tres exportaciones, `PayablesIntegrationTests` y `PermissionsMigrationTests`. No cambiaron —aunque el plan los nombraba— los `*AllowedActionsTests`, los `*AuditTests`, los conteos de sentencias, `ReportsIntegrationTests`, `StockRowsTests`, `UnitPurchaseCostsTests`, `ProcurementOpenApiTests` ni `ChangeLogsSearchFilteringTests`: el reporte de stock por área no tiene prueba propia en `reporting` (RPT-R11). La suite completa del backend no corrió en local, y el CI tampoco.

**El frontend** (rama `feature/phase-7-procurement-by-area`, apilada sobre la fase 6 e integrada en `develop` después del backend, con los hashes de la línea de Estado). Son diez commits, uno por paso, y el último lo concilia con el backend final: los campos planos `areaId` y `areaName`, el saldo retenido y los textos de los rechazos del área como los dice `messages_es.properties`. Lo construido:
- **La sesión** pregunta la empresa, la marca y el área a una misma entrada de `assignments`, con los ayudantes nuevos de `IamStore` e `IamFacade`: `readsArea`, `writesOnArea`, `writableAreas` y `actsOnEveryBrandAndArea`. Una sesión sin `assignments` cae en `hasPermissionIn`, `actsOnWholeCompany` y `actsOnEveryBrand`, y decide el backend. `procurement-writes.ts` los usa en toda escritura de compras. Llegan `OrganizationFacade.loadAreaChoices` y `CompanyUnits.areas`, como pedía la precisión 7 de la fase 6.
- **El diálogo de otorgar un rol** ofrece «Áreas», una selección múltiple con «Todas las áreas» mientras no se elige ninguna, para un rol que admite área en una empresa con áreas; envía `areaIds` y, abierto desde un colaborador, propone el área del colaborador.
- **Las pantallas de compras.** La plantilla tiene su campo «Área», su columna y su filtro. La orden sigue la regla Q27: el área de su plantilla o, con una general, la de quien la emite, fija si escribe en una sola, a elegir si escribe en varias y «General» solo con acceso a todas. La factura toma el área de su orden o, sin orden, elige la suya con la misma regla, y cambia de orden solo entre las de su área. La nota toma el área de sus facturas y busca solo facturas de esa área. El pago queda en el área de su primer documento y ofrece solo los de esa área, con «Saldo a favor» solo para quien escribe en todas las marcas y todas las áreas. Por pagar, la ficha del proveedor y la bitácora muestran el área en cada una, y la bitácora filtra por ella después de elegir la empresa.
- **El saldo que no viaja** (Q9). A quien no tiene todas las áreas, cada monto del saldo de un proveedor se le lee «—», nunca un cero, y el formulario del pago no toma saldo de él; un uso del saldo de una nota por un pago de un área que no lee dice «un pago de un área que usted no ve», sin enlace.
- **«Asignar área»**, en las listas de Órdenes de compra y de Facturas —en Facturas, solo las que no tienen orden—, para quien tiene `compras.asignar_area` en toda la empresa y todas las áreas (`actsOnEveryBrandAndArea`): elige filas generales, también de varias páginas, y abre su diálogo, cargado al primer uso, que pide un área activa, dice cuántas órdenes, facturas, notas y pagos la tomaron y, si se rechaza, nombra cada documento que lo impide. `compras.asignar_area` va en el grupo Compras de la matriz, fuera de sus pantallas.
- **«Más filtros»** en `mm-data-manager` (`DESIGN.md`, revisión 102, que define también el área de una lista, de un formulario y de una ficha, y «Asignar área»): Facturas y Notas de crédito, que mantienen su barra en una línea, llevan ahí el filtro de área, con su columna.
- **Cuándo no se dibujan** el filtro, la columna ni «Asignar área»: sin `configuracion.ver_estructura`, con que se leen las áreas de la empresa, o en una empresa sin áreas.

**Queda pendiente:** los e2e de la fase, con los simulacros de `/companies/{id}/areas` y de `assignments` con área; las imágenes de referencia de las pantallas que suman el área; la prueba de punta a punta en `dev` que pide «Terminada cuando»; y el CI, que no corre porque la cuenta de GitHub tiene un problema de pago o de límite de gasto. Antes de otorgar la primera asignación con área en `dev`, lo que ya existe recibe su área con «Asignar área» (arriba).

---

## 7. Lista de control de cada fase

**Backend**
- [ ] `ModularityTests` pasa: ninguna dependencia nueva fuera de `allowedDependencies` (la única nueva del plan es `iam::acl` en `equipment`, fase 3); las fachadas exponen lo que usan sus consumidores; ninguna clase importa el interior de otro contexto (`ForeignAclStaysInTranslationLayerTests`).
- [ ] Cada `External<Contexto>Service` nuevo lleva nombre de bean explícito, y ninguna ruta queda mapeada dos veces (`ErpMillenniumMobilityBackendApplicationTests` arranca el contexto).
- [ ] Toda ruta nueva pide su permiso con una constante de `PermissionCodes` (`NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`).
- [ ] Los servicios de comandos siguen el patrón `*CommandServiceImpl`; lo que corre dentro de la transacción de otro contexto no se llama así; todo `@Scheduled` empieza con `runAlone` y entra en la lista exacta de `DeadlockRetryTests`.
- [ ] Ninguna llamada externa ocurre dentro de una transacción (regla 6.4).
- [ ] Toda tabla nueva con `company_id` tiene sus políticas, su lugar en `RowLevelSecurityMigrationTests` (y en `COLUMN_UPDATE_TABLES` si actualiza por columnas) y su prueba de aislamiento; toda tabla que se exporta, sus casillas en `data_versions` (`JdbcDataVersionsTests`).
- [ ] Toda migración abre con alcance `SYSTEM` y `lock_timeout` de 5 segundos; ninguna lee ni escribe las tablas de otro contexto (reglas 1.2 y 1.3).
- [ ] Toda función o tabla de solo inserción nueva tiene su línea en `hand-over-database.sql` (`HandOverDatabaseScriptTests`).
- [ ] Todo código de error nuevo tiene sus mensajes en los dos bundles (`OwnErrorCodesHaveMessagesTests`) y su lugar en la OpenAPI (`*OpenApiTests`).
- [ ] Todo agregado nuevo tiene su `<Agregado>AuditTests`; todo servicio de consultas que lee tablas de empresa corre `readOnly`.
- [ ] Toda sentencia nueva en un conteo de sentencias está justificada en la prueba.
- [ ] Ningún SQL nativo enlaza un nulo en una comparación opcional.
- [ ] Toda variable nueva está en el README, `.env.example` y `.do/app.yaml`.

**Frontend**
- [ ] Las cuatro capas en cada contexto; las fachadas solo con lo que alguien usa, y su spec de miembros al día.
- [ ] Cada ruta pide lo mismo que su opción del menú; cada ruta nueva de la API está en los dos archivos de `environments`.
- [ ] Ningún diálogo de formulario se abre sobre otro diálogo (regla 4.9).
- [ ] Cada cambio visual tiene su revisión de `DESIGN.md`, y solo cambian las imágenes de referencia que la fase nombra.
- [ ] Las claves nuevas de `es.json` siguen sus convenciones (descripciones de 90 caracteres o menos).

**Documentación**
- [ ] El diccionario, `contextos/`, las convenciones y `procesos-y-pantallas`, en el mismo cambio; la decisión de la fase, de «Decidido y por construir» a «cerradas».
- [ ] El README del backend si cambió una variable; las cuatro copias de las skills.

**Integración**
- [ ] El CI de la rama, en verde —unitarias **y** e2e—, mirado por el responsable.
- [ ] Primero el backend, después el frontend; la documentación a `master` el mismo día.
- [ ] Al cerrar: el despliegue en `dev` activo, y el worktree, la rama y la base de verificación borrados.

---

## 8. Vuelta atrás

| Fase | Cómo se deshace |
|---|---|
| 0 | Se revierte su commit en el frontend. No toca datos |
| 1 | Se revierte su commit de integración en el frontend, y la enmienda de D-141. No toca datos |
| 2 | Gracias al disparador de compatibilidad, el código anterior sigue insertando líneas. Pero en cuanto exista un equipo en un estado nuevo o una orden de equipos, el código anterior no los lee: después de la primera factura de equipos, la fase se corrige hacia adelante. Durante el despliegue no se registra una factura de equipos |
| 3 | La 3a se revierte con su código: no tiene migración propia. Sin la SIM (Q30, Q31), la 3b no mueve datos: el ICCID nunca deja el equipo. Pero el código anterior no lee un equipo «Por completar» o «Revisar», así que, después del primer escaneo con fabricante, se corrige hacia adelante |
| 4 | El cambio de la instalación va en tres pasos: el segundo (el frontend) se revierte solo. Los enlaces escritos por la vinculación no se deshacen |
| 5 | Se revierte su commit: no toca datos |
| 6 | Revertir el código deja la tabla de áreas y las columnas nuevas sin usar, que no molestan; lo demás se corrige hacia adelante. Una asignación con área creada por la API quedaría sin efecto en compras: por eso la pantalla no la ofrece hasta la fase 7 |
| 7 | Revertir el código deja las columnas de área en lo nuevo y los disparadores activos, que con el área vacía dejan pasar todo. Se corrige hacia adelante si ya hay documentos con área o se usó «Asignar área» |

---

## 9. Calendario orientativo

**El primer tramo (Q28):**

| Fase | Días de trabajo | Acumulado |
|---|---|---|
| 0, reducida | 1 | 1 |
| 2 | 14 a 18 | 15 a 19 |
| 3a | 5 a 7 | 20 a 26 |

El flujo operativo hasta el ingreso de los IMEIs, con el reemplazo de un fallado, queda listo en `dev` entre los días 20 y 26.

**Lo demás, en el orden que diga el responsable:**

| Fase | Días de trabajo |
|---|---|
| 3b | Menos de 5 a 7, que contaban la SIM (Q30, Q31), y 2 a 4 para el adaptador |
| 4 | 8 a 11 (después de la 3b) |
| 5 | Menos de 1 (después de la 4) |
| 1 | 2 a 4 |
| 6 | 5 a 7 |
| Las pruebas de caracterización de compras | 1 (antes de la 7) |
| 7 | 12 a 16 (después de la 6) |

En total, de 53 a 73 días, algo menos sin la SIM. La fase 6 toca archivos distintos de las del GPS y podría ir en paralelo con dos personas; con una, va en orden. El adaptador del fabricante se agrega cuando llegue Q21.

---

## 10. Riesgos

| Riesgo | Probabilidad | Impacto | Cómo se cubre |
|---|---|---|---|
| El e2e del frontend no corre desde el 20/11 y esconde regresiones | Alta | Medio | La fase 0 lo corre entero y lo arregla antes de la regeneración de imágenes de la fase 1 |
| Una consulta de compras olvida el área y muestra datos de otra (la defensa es solo del código) | Media | Alto | La prueba de aislamiento por documento, la regresión de una sola asignación, la caracterización, la lista de lecturas de la fase 7 y la revisión de arquitectura de cada lectura |
| El motor de pagos cambia de comportamiento para los vehículos | Baja | Alto | Las facturas de equipos no tocan las imputaciones; las pruebas de pagos de vehículos no cambian |
| `isConcept()` cambia de sentido en un uso que nadie miró | Media | Alto | Los 13 usos decididos uno por uno, con una factura de varias líneas de equipo en las pruebas |
| Una factura de 500 equipos es lenta o choca con el corte de Cloudflare | Media | Medio | La inserción de varias filas, el conteo de sentencias y la meta medida contra `dev` |
| Un equipo hereda el precio o el modelo de otro al escanear | Baja | Medio | El escaneo por modelo |
| La API del fabricante no llega o es inestable | Media | Medio | El proveedor `manual`, que ingresa con el serial en el escaneo; los reintentos, el vigilante y el completado a mano |
| Un GPS instalado queda sin equipo y escapa de D-264 | Media | Medio | La vinculación al arrancar, la del serial completado y la de las instalaciones de la carga inicial |
| Regenerar 120 imágenes esconde una regresión real | Media | Medio | El e2e en verde de la fase 0; se revisa una por grupo y se mira la diferencia antes de aceptar |
| Las personas limitadas a un área no pueden emitir órdenes | Baja | Medio | Una plantilla general la usan todas las áreas, y la orden toma el área de quien emite |
| «Asignar área» no puede separar documentos que una nota o un pago mezcla | Media | Bajo | Se asignan juntos en un solo pedido; si son de áreas distintas, quedan generales y el rechazo lo dice |
| Conviven dos instancias durante el despliegue de la fase 2 | Baja | Bajo | No se registra una factura de equipos mientras se despliega |
| Los constructores con muchos usos rompen las pruebas | Alta | Bajo | Constructores secundarios y el `default` de `resolveScope`; el de tres argumentos de `RoleScope`, solo para pruebas |
| El alcance crece (D-266 completo, ingreso por Excel, mapa) | Media | Medio | El §11 los deja fuera; cada uno es su propio cambio |

---

## 11. Fuera del alcance

- El monitoreo en vivo: el mapa y las alertas de Monitoreo.
- La línea de cada SIM y su costo: los maneja el proveedor, que entrega el GPS con su SIM (Q31).
- El ingreso de equipos por Excel, con la plataforma de cargas: posible después, sobre el mismo ingreso.
- Las áreas fuera de compras (Q10).
- D-266 completo, salvo la retención por equipos instalados de la fase 4.
- Las notas de débito (D-23).
- El bloque 12: clientes, contratos, cobranza y traspasos.

---

## 12. Registro del plan

| Versión | Fecha | Qué |
|---|---|---|
| 1 | 22/11/2026 | Primera versión, desde el análisis situacional del mismo día y las decisiones del responsable |
| 2 | 22/11/2026 | Recoge la auditoría del mismo día (6 bloqueantes, 22 importantes) y las decisiones Q1 a Q27. Cambia el orden a menú → GPS → áreas; suma la historia y el historial del equipo, la vinculación de la carga inicial, el escaneo por modelo, «Anular y copiar» sin duplicados, la orden de un solo tipo, la plantilla común con la orden del área de quien emite, varias áreas por persona y «Asignar área»; corrige la sesión de las áreas, `isConcept()`, la inserción de 500 líneas, los pagos sin migración y las afirmaciones sin respaldo |
| 3 | 22/11/2026 | El primer tramo del responsable (Q28, Q29): la fase 0 reducida —las pruebas de caracterización de compras pasan a antes de la fase 7—, la fase 2 y la fase 3a, el ingreso sin fabricante con el reemplazo de un fallado, cuya columna pasa de V155 a V149. La fase 3 se parte en 3a y 3b; ninguna migración cambia de número |
| 3.1 | 22/11/2026 | Las respuestas Q30 y Q31: la SIM es un dato de su GPS toda su vida, sin registro propio, y su línea la maneja el proveedor (C11 y D-285 enmendadas). La 3b pierde `sim_cards`, `sim_card_assignments`, sus rutas, sus permisos, su pantalla y sus códigos, y queda con una migración, el completado; `SIM_MISMATCH` compara el ICCID del fabricante con el escaneado; la fase 4 exige el ICCID del equipo para instalar; la fase 5 ya no retira el ICCID y queda sin migración. Las migraciones esperadas se recuentan en el §5. Q32 sigue abierta |
| 3.2 | 22/11/2026 | La respuesta Q32: un equipo cobrado que no llegó solo se da de baja con la nota de crédito del proveedor, como una unidad; no se construye nada. Ninguna pregunta del ciclo del GPS queda abierta; falta la información del fabricante (Q21) |
| 3.3 | 22/11/2026 | La fase 1, el menú por procesos, hecha (frontend `a3c9a609`), con 31 opciones y la revisión 100 de `DESIGN.md`; las imágenes de referencia quedan para un lote. La factura de equipos se lee como su comprobante —líneas por modelo, cantidad y precio unitario, la ficha por modelo y la nota de crédito por cantidad— (frontend `8fb49548`). Q21, en parte: el fabricante es Teltonika, su QR lleva solo el IMEI y la API sería la de FOTA WEB |
| 3.4 | 23/11/2026 | La fase 4, la instalación unida al inventario y el historial del equipo, hecha (sus hashes, en la línea de Estado) con V155 y V156, antes de la 3b y sin «Por completar». Se aparta del plan en doce cosas, entre ellas los ocho casos del enlace, completar un serial que rechaza lo que no enlaza, `allowedActions.devicesHeldReasonIds` y las instalaciones sin quién retiró. Abre Q33 a Q36 |
| 3.5 | 23/11/2026 | La fase 6, áreas: catálogo, asignación y sesión, hecha e integrada después de la fase 4 (sus hashes, en la línea de Estado), con sus migraciones renumeradas V157 a V160. Se aparta del plan en once cosas, entre ellas la respuesta de otorgar con `areaIds`, `actsOnEveryArea` en la pestaña, y `OrganizationFacade.loadAreaChoices` y `CompanyUnits.areas`, que pasan a la fase 7; sus e2e y sus imágenes quedan para un lote. |
| 3.6 | 23/11/2026 | Las fases 5 y 7, con el diccionario 1.88. La fase 5, la limpieza, hecha e integrada (sus hashes, en la línea de Estado): un tipo del inventario —telemático y que el modelo no trae de fábrica, hoy el GPS— se instala y se reemplaza solo con su equipo, con el código nuevo `VEHICLEEQUIPMENT_DEVICE_REQUIRED` y `requiresDevice` en la habilitación, y la variable de la fase 4 queda documentada; D-263, construida en sus tres pasos. La fase 7, compras por área y «Asignar área», hecha en el backend e integrada después de la fase 6 (sus hashes, en la línea de Estado), con sus migraciones renumeradas V161 a V163, y su frontend integrado después, con el área en cada pantalla de compras y en la bitácora, el diálogo de otorgar con áreas, «Asignar área» y «Más filtros» (`DESIGN.md`, revisión 102). Se aparta del plan en catorce cosas, entre ellas los campos planos `areaId` y `areaName`, el código nuevo `PURCHASEINVOICE_AREA_REQUIRED`, el filtro que exige `configuracion.ver_estructura`, «Asignar área» de una sola empresa, hasta 500 documentos y también sobre los anulados, y cinco commits en lugar de doce. Abre Q37, cómo se corrige un área mal asignada |
| 3.7 | 23/11/2026 | Las respuestas Q33 y Q34, construidas el mismo día, backend y pantallas, sin migraciones (sus hashes, en la línea de Estado), con el diccionario 1.89: la baja definitiva de una unidad Robada retira sola sus equipos del inventario, por robo, con `allowedActions.devicesRetiredReasonIds` y el aviso de «Cambiar estado» (D-289); y el modelo de un equipo se corrige desde su ficha, dentro de su tipo, salvo el de una factura, con `correctModel` y el código nuevo `EQUIPMENT_MODEL_FROM_INVOICE` (D-290). Abre Q38 a Q41: la marca que congela el equipamiento, editable en el catálogo; el modelo del reemplazo de un equipo comprado; Q17 frente a la regla de la factura; y la edición de un GPS enlazado sin IMEI ni ICCID mientras está instalado |
| 3.8 | 23/11/2026 | Las respuestas Q35 a Q41, que el responsable aprobó como se le recomendaron, decididas y en construcción, con el diccionario 1.90: «Retirado por» en el retiro y el reemplazo, con V164 (D-291); la edición de un equipo instalado, con límites, que valida solo los identificadores que cambian (D-292, Q36 y Q41); la marca que congela el equipamiento, fija (D-293); el modelo del equipo que reemplazó a uno comprado, que se corrige (D-294); «Cambiar área», con V165 (D-295); y «Anular y copiar», que cambia el modelo de una línea cuyos equipos ya llegaron y el de esos equipos (D-296, Q40). Sus contratos de la API se documentan al terminar cada frente. Solo queda pendiente la información del fabricante (Q21) |
| 3.9 | 23/11/2026 | Q35 a Q41 construidas el mismo día, backend y pantallas (sus hashes, en la línea de Estado), con V165 y V166 —V164 no se usó— y el diccionario 1.91: «Retirado por», con `removedByEmployeeId` y el código nuevo `VEHICLEEQUIPMENT_REMOVER_OTHER_COMPANY` (D-291); la edición de un equipo instalado, con `EQUIPMENT_INSTALLED_FIELD_LOCKED`, que revisa solo los identificadores que cambia y retira la excepción de un `PATCH` con solo el modelo (D-292); `VEHICLESTATUS_EQUIPMENT_FREEZE_LOCKED` (D-293); `correctModel` para el reemplazo de un equipo comprado (D-294); «Cambiar área», `POST /purchase-area-changes`, con tres códigos nuevos y los disparadores diferidos de V165 (D-295); y el modelo de una línea cambiado con «Anular y copiar», con `PURCHASEINVOICE_EQUIPMENT_MODEL_TYPE_MISMATCH` (D-296). `EQUIPMENT_MODEL_FROM_INVOICE` manda ahora a «Anular y copiar». Queda, menor, que la tabla de equipamiento de la habilitación no muestra «Retirado por» |

**Equivalencias con la versión 1.** La auditoría cita las fases, las migraciones y las decisiones de la versión 1. La versión 3 conserva la numeración de la 2, salvo que la fase 3 se parte en 3a y 3b.

| Versión 1 | Versión 2 |
|---|---|
| Fase 0, 1 | Fase 0, 1 |
| Fase 2 (áreas: catálogo, asignación y sesión) | Fase 6 |
| Fase 3 (compras por área) | Fase 7 |
| Fase 4 (compra de equipos) | Fase 2 |
| Fase 5 (ingreso, fabricante y SIM) | Fase 3 |
| Fase 6 (instalación) | Fase 4 |
| Fase 7 (limpieza) | Fase 5 |
| V148, V149, V150, V151 | V160, V161, V162, V163 |
| V152, V153 | V164, V165 |
| V154, V155 | V148, V149 |
| — | V150 (la historia de estados del equipo) |
| V156, V157, V158 | V151, V152, V153 |
| V159, V160, V161 | V154, V155, V156 |
| V162, V163 | V157, V158 |
| V164 | V159 |
| — | V166 (`compras.asignar_area`) |
| D-283 (áreas), D-284 (compras por área) | D-287, D-288 |
| D-285 (compra de equipos), D-287 (cumplir con faltante) | D-283, D-284 |
| D-286 (ingreso, fabricante y SIM) | D-285 |
| — | D-286 (historial del equipo y vinculación) |

Si el orden de integración cambia otra vez, las migraciones se renumeran al abrir la rama: Flyway exige que la versión siga a la última aplicada.

