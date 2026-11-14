# Auditoría y plan de implementación: la unidad que llega antes que su factura

**Versión 1 · 13/11/2026.** Propuesta para validar: nada de este plan está decidido ni construido. Audita cómo nace una unidad que llega antes que su factura —el botón «Nueva unidad» de Inventario— y cómo la factura la vincula después, y propone cinco fases. Las decisiones que pide van como propuestas, P-1 a P-5 (§3.1): al aprobarlas el responsable toman su número en el diccionario, desde D-254, y sus reglas desde FLT-R148 y PRC-R183.

**Cómo se llegó aquí:** las preguntas del responsable del 13/11/2026 —por qué Inventario tiene «Nueva unidad» si la unidad nace con la factura, si el alta puede ir en una vista propia, qué pasa con muchas unidades de una sola factura y cómo se sincronizan las que llegan antes que ella— y una auditoría de los tres repositorios ese día, todavía sin commit: el backend en `feature/contract-guard-and-code` (hasta V130), el frontend en `feature/contract-code-and-brand-words` (con la revisión 97 de DESIGN.md) y la documentación en `docs/decisions-d250-to-d253`. Las rutas del backend se citan desde `src/main/java/ky/millennium/mobility/erp/`; las del frontend, desde `src/app/`.

---

## 1. Alcance

| Implementa, si se aprueba | Qué |
|---|---|
| P-1 | La factura avisa cuando el modelo o el año del comprobante difieren de la unidad que vincula |
| P-2 | El alta a mano dice que es a mano, cuándo usarla y qué pasa si se anula |
| P-3 | La factura corrige la unidad que nació sin factura |
| P-4 | Un reporte de las unidades sin factura |
| P-5 | El alta en lote, solo si el negocio la necesita |

Sirve a los procesos P4 (compras) y P6 (habilitación) de [procesos y pantallas](procesos-y-pantallas-millennium-mobility.md).

**Queda fuera:** el lector del XML (D-157, candidato); las facturas de más de 100 unidades (pregunta 3 de §3.2); la marca comercial que cambia en una unidad ya facturada (hallazgo H7, pregunta 5); la carga inicial de una unidad dada de alta a mano (PRC-R142, riesgo R6).

---

## 2. Hallazgos

### 2.1 Veredicto

«Nueva unidad» tiene razón de ser: D-153 la dejó para la unidad que llega antes que su factura, que la factura vincula después por su VIN. El problema no es el botón, sino lo que pasa después: **la factura copia en silencio el modelo que alguien escribió a mano, y ninguna vía corrige el modelo de una unidad**. Lo demás —el origen que no se distingue, la falta de seguimiento, el VIN que se pierde al anular— agrava ese problema.

**Puertas:** P-3 necesita su especificación antes del código (Fase 3); P-5 necesita que el negocio confirme el caso (pregunta 1 y 2 de §3.2).

### 2.2 Lo que se encontró

| N.º | Hallazgo | Evidencia | Gravedad |
|---|---|---|---|
| H1 | **La factura toma en silencio el modelo, el año y la marca de la unidad.** Una línea que vincula una unidad que la sesión ve envía los datos de la unidad, no los del comprobante, y el aviso solo compara el motor y el color. Por la API, una diferencia se rechaza (`422 PURCHASEINVOICE_UNIT_MISMATCH`); por la pantalla nunca llega al backend. Un modelo mal escrito a mano termina en la línea de la factura, en el costo de compra, en las notas y en los reportes | Frontend: `invoice-line-form.ts:156-172` (`lineDraftOf`), `vin-check.ts:96-117` (`linkedUnitDifferences`), `purchase-invoice-lines.ts:635-648` (`noticeOf`). Backend: `procurement/.../PurchaseInvoiceUnits.java:161-189`. Especificación de facturas, §11.3: la línea «toma la marca, el modelo y el año de su unidad» | Alta |
| H2 | **El modelo de una unidad no se corrige por ninguna vía.** La edición no lo recibe y la persistencia no lo escribe aunque cambiara | FLT-R08 («el modelo no cambia después del alta»); `fleet/domain/model/commands/UpdateVehicleGeneralDataCommand.java`; `fleet/infrastructure/persistence/jpa/entities/VehiclePersistenceEntity.java:42-43` (`updatable = false`) | Alta |
| H3 | **Anular por error una unidad sin factura pierde su VIN.** Sin factura su valor es cero, así que se anula; anulada, ninguna factura la vincula y su VIN no vuelve a darse de alta. D-191 ya lo reconoce para las notas de crédito («la dejaría trabada») | FLT-R48, PRC-R44 (`VEHICLE_ANNULLED`), FLT-R01 | Media |
| H4 | **El historial no distingue el origen del alta.** El alta a mano y el de la factura escriben la misma fila, «Alta de la unidad · Registro de compra», sin nota: una unidad tecleada a mano dice «Registro de compra» sin compra registrada | `fleet/application/internal/commandservices/VehicleMovements.java:91-96` (nota `null`); los dos llaman a `VehicleRegistrations.register` (`:118`) con el mismo camino (`:68-79`) | Media |
| H5 | **Nada sigue a las unidades sin factura.** Solo están la insignia y el filtro «Sin factura vigente» de Inventario, pensados para la factura anulada (D-175). No hay reporte, aviso ni proceso programado, y «Brecha de días» las excluye. La puerta de Disponible no pide factura (D-186: `fleet` no lee facturas), así que una unidad a mano puede habilitarse y alquilarse sin que nadie lo note | FLT-R51; `reporting/.../ReportsController.java:96-97`; `es.json`, `reports.gaps.empty-hint` | Media |
| H6 | **Nadie sabe si una unidad tuvo factura alguna vez.** Todas las consultas de líneas por unidad filtran las vigentes, y el puerto `UnitPurchases` solo responde por la línea vigente. Las líneas anuladas conservan su unidad, así que el dato existe, pero no hay consulta ni índice que lo lea | `procurement/.../PurchaseInvoiceJpaRepository.java:35-46, 114-123`; `fleet/interfaces/acl/UnitPurchases.java:28-40`; V94:124-130, :205-223 | Media |
| H7 | **La marca comercial de una unidad facturada cambia y su línea no.** La edición la deja cambiar con una línea vigente; la línea la copió al registrarse y no cambia. La factura, las notas y los pagos deciden el alcance por la marca de la línea, e Inventario y los reportes por la de la unidad. Anular una nota repone su línea sin volver a comparar | `fleet/.../VehicleCommandServiceImpl.java:186-196`; `PurchaseInvoiceUnits.java:106-110`; `CreditedLines.java:167`; `SupplierPaymentScopes.java:54`; `CreditNoteAnnulments.java:126-148` | Media (fuera de este plan) |
| H8 | **Cambiar el modelo arrastra la habilitación.** Del modelo salen el tipo de vehículo (qué equipos aplican), la pantalla de fábrica y el gas (qué documentos aplican). La puerta solo se mira al entrar a un estado, así que una unidad ya en Disponible no se vuelve a revisar | `vehiclecatalog/.../ModelEnablementProfiles.java:46-77`; `fleet/.../VehicleEnablementFactsReader.java:63-68`; `VehicleMovements.java:182` | Media, para P-3 |
| H9 | **La edición de una unidad no deja rastro en la bitácora.** Solo la placa, el VIN y el traspaso lo dejan | `VehicleCommandServiceImpl.java:157-232`, sin `ChangeLogCommandService`; `Vehicle.java:34-45` | Baja, para P-3 |
| H10 | **«Nueva unidad» no dice cuándo usarse,** y a una sesión limitada a una marca le ofrece todas | `es.json`, `fleet.vehicles.dialog.create-description`; [fleet](contextos/fleet.md), sección 6 | Baja |
| H11 | **Una factura de más de 100 unidades no entra.** El negocio habló de lotes de hasta unas 80 motos | PRC-R36, PRC-R156; [plan de compras](plan-de-implementacion-compras-y-habilitacion-2026-11-02.md), §1 | Baja (pregunta) |

### 2.3 Lo que ya funciona y se reutiliza

- **El autocompletado del VIN** (D-179): `GET /purchase-invoice-vin-checks` responde, por cada VIN, si es nuevo, se vincula o se rechaza, con los datos de la unidad para quien la ve con `stock.ver` en su marca (`procurement/.../PurchaseInvoiceVinChecks.java:98-153`). La línea que vincula muestra esos datos fijos y un enlace a la ficha.
- **Las líneas importadas conservan lo que trajo el Excel** aunque se bloqueen: el modelo, el año, el color y el motor siguen en sus campos (`purchase-invoice-lines.ts:497-540`, `:413-427`), así que se puede comparar sin cambiar el backend. La marca comercial no: la importación pone la marca por omisión del formulario, porque el comprobante no la trae.
- **Las fechas pasadas.** El ingreso solo rechaza una fecha futura (FLT-R62, D-186) y una instalación, una anterior al ingreso o futura (FLT-R76). Por eso nada obliga a crear la unidad antes de su factura: se registra la factura cuando llega, y después el ingreso y las instalaciones con sus fechas reales.
- **El ingreso en lote,** hasta 100 unidades, todo o nada (FLT-R62).
- **El permiso de la factura** da de alta sus unidades sin `stock.crear_vehiculo` (D-153, D-163), y la factura bloquea sus unidades `FOR NO KEY UPDATE` en su transacción (especificación de facturas, §2.4).
- **La pregunta con tabla** de `ConfirmationDialog` (DESIGN.md, revisión 36), para confirmar varias correcciones de una vez.

### 2.4 ¿Una vista propia para «Nueva unidad»?

**No para una sola unidad.** DESIGN.md crea un registro de unos diez campos en un diálogo (revisión 11) y deja la vista propia para los documentos con líneas: órdenes, facturas, notas y pagos (revisiones 34, 36 y 37). Lo que faltaba —decir cuándo usarla y llevar a la factura— cabe en el diálogo (Fase 2). **Sí para el lote** (Fase 5): ahí el alta es un documento con líneas, como la factura.

---

## 3. Decisiones

### 3.1 Propuestas al responsable

| N.º | Propuesta | Qué enmienda | Fase |
|---|---|---|---|
| P-1 | La línea que vincula una unidad avisa también cuando el modelo o el año que trae difieren de los de la unidad, sin impedir guardar, como ya avisa por el motor y el color. No compara la marca comercial, que el comprobante no trae | Precisa D-179 y PRC-R67 | 1 |
| P-2 | El alta a mano escribe en su fila del historial «Alta a mano en Inventario, sin factura.», y el alta de una factura, su número. El diálogo dice cuándo usarla y lleva a Facturas. «Anular unidad», en una unidad sin factura vigente, avisa que su VIN no vuelve a darse de alta ni a vincularse | Precisa D-153 | 2 |
| P-3 | Una factura corrige la unidad que vincula si esa unidad nunca tuvo una línea de factura, en ningún estado, no está anulada y nunca tuvo un contrato (D-250): toma del comprobante el modelo, siempre del mismo tipo de vehículo, el año, el color y el motor. La marca comercial sigue siendo la de la unidad. Quien registra lo confirma antes de guardar, y cada campo corregido queda en la bitácora con el número de la factura. Una unidad que ya tuvo factura sigue con D-160: su modelo y su año mandan | Enmienda D-160 y D-179; excepción a FLT-R08 | 3 |
| P-4 | Reportes lista las unidades sin factura vigente, si nunca la tuvieron o la perdieron, su estado y cuántos días llevan así | Precisa D-175 | 4 |
| P-5 | *(Condicional)* «Nuevas unidades»: el alta de hasta 100 unidades en una vista, con la plantilla de Excel de la factura, todo o nada | Precisa D-153 | 5 |

**Alternativa para P-2:** que el alta a mano vaya por su propio camino con código reservado, «Alta sin factura», como «Migración inicial» (FLT-R120), para que el motivo mismo cambie. Cuesta una migración y la regla del camino de alta único y del estado inicial. Se recomienda la nota: dice lo mismo a quien lee el historial, y el reporte de P-4 da la lista sin depender de ella.

### 3.2 Preguntas abiertas

| N.º | A quién | Pregunta | Bloquea |
|---|---|---|---|
| 1 | Negocio | ¿Llegan unidades antes que su factura? ¿Con qué frecuencia y en qué cantidad? | Fase 5 |
| 2 | Negocio | ¿Necesitan verlas en Inventario antes de tener la factura, o pueden registrarlas cuando llegue? | Fase 5 |
| 3 | Negocio | ¿Algún proveedor factura más de 100 unidades en un comprobante? | Nada; si la respuesta es sí, abre una decisión sobre PRC-R36 |
| 4 | Negocio | ¿Desde cuántos días una unidad sin factura es una alerta? | El aviso de la Fase 4, no su reporte |
| 5 | Responsable | ¿La marca comercial de una unidad facturada puede cambiar? Si puede, ¿su línea de factura debe seguirla? (H7) | Nada de este plan |
| 6 | Responsable | ¿Hay hoy en producción unidades dadas de alta a mano? Si las hay, sus facturas se registran a mano antes de la carga inicial (PRC-R142; [especificación de cargas](especificacion-cargas-de-archivos-2026-11-03.md), §1.3, fila 6) | La carga inicial, no este plan |
| 7 | Responsable | Si se anula una factura que corrigió una unidad, ¿la unidad recupera la posibilidad de corregirse con la factura que la reemplace? | La especificación de la Fase 3 |

---

## 4. Orden

**Fase 0 → 1 → 2 → 3 → 4,** y la 5 solo si el negocio la confirma.

- **Fase 0 primero:** las ramas sin commit tocan los mismos archivos que este plan (`es.json`, DESIGN.md, [fleet](contextos/fleet.md), el diccionario).
- **Fase 1 enseguida:** es solo frontend y saca a la luz el error silencioso de H1 mientras llega la corrección.
- **Fase 2 antes de la 3:** es chica, y conviene que las unidades que se den de alta a mano desde ahora ya digan su origen.
- **Fase 3 con su especificación:** es la única grande y toca la frontera entre `procurement` y `fleet`.
- **Fase 4 después de la 3:** usa la consulta «tuvo factura alguna vez» que construye la 3.

---

## 5. Plan por fases

### Fase 0 · Integrar lo pendiente

**Puerta:** el responsable autoriza los commits.

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 0.1 | Un commit por decisión, D-250 a D-253, en las tres ramas, y la revisión 97 aparte en el frontend | S | — |
| 0.2 | La suite completa del backend con las cuentas `erp_mm_app` y `erp_mm_owner` sobre `erp_mm_db_verify`, no con `postgres`, que se salta Row Level Security | M | 0.1 y las claves locales |
| 0.3 | Merge de las tres ramas a `develop` | S | 0.2 |

**Aceptación:** `develop` tiene V130 y las suites del backend y del frontend pasan.

### Fase 1 · La factura avisa (P-1, solo frontend)

**Puerta:** P-1 aprobada.

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 1.1 | `linkedUnitDifferences` compara también el modelo y el año que la línea trae, escritos o importados, con los de la unidad; nunca la marca comercial | S | 0.3 |
| 1.2 | `noticeOf` dice «El comprobante dice {modelo} {año} y la unidad es {modelo} {año}. La línea conserva los de la unidad.» | S | 1.1 |
| 1.3 | Pruebas: `vin-check.spec.ts` (modelo, año, ningún aviso por la marca), `purchase-invoice-lines.spec.ts`, y el e2e de `purchase-invoices.spec.ts` con un Excel que trae otro modelo para un VIN existente | S | 1.2 |
| 1.4 | Especificación de facturas (§6.3 y §11.3), PRC-R67 en [procurement](contextos/procurement.md) y una revisión de DESIGN.md | S | 1.3 |

**Aceptación:** una línea que vincula una unidad cuyo modelo o año difieren de lo importado muestra el aviso; guardar sigue funcionando como hoy; la marca comercial no da aviso.

### Fase 2 · El alta a mano dice lo que es (P-2)

**Puerta:** P-2 aprobada.

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 2.1 | Backend: `VehicleMovements.recordRegistration` recibe la nota; `VehicleCommandServiceImpl` escribe «Alta a mano en Inventario, sin factura.»; `PurchasedUnitRegistrations` escribe el número de la factura, que `RegisterPurchasedUnitsRequest` suma | S | 0.3 |
| 2.2 | Frontend: el diálogo «Nueva unidad» abre con el aviso «La unidad nace con su factura de compra. Regístrela aquí solo si llegó antes que su factura: al registrarla, la factura la vincula por su VIN.» y, para quien tiene `compras.registrar_factura`, «Ir a Facturas»; el texto de la lista vacía deja de invitar a registrar a mano | S | — |
| 2.3 | Frontend: «Anular unidad», en una unidad sin factura vigente, suma «Su VIN no volverá a darse de alta ni a vincularse a una factura. Si el VIN está mal, use «Corregir VIN».» | S | — |
| 2.4 | Pruebas: `VehiclesIntegrationTests` (la nota del alta a mano), `PurchaseInvoicesIntegrationTests` (la nota con el número), los specs del diálogo y del aviso, el e2e de `fleet.spec.ts` | S | 2.1 a 2.3 |
| 2.5 | F1 de [fleet](contextos/fleet.md), su regla nueva y una revisión de DESIGN.md | S | 2.4 |

**Aceptación:** el historial de una unidad dada de alta a mano dice que fue a mano y sin factura; el de una nacida con la factura, su número; el diálogo dice cuándo usarse; anular una unidad sin factura vigente avisa que su VIN se pierde.

### Fase 3 · La factura corrige la unidad que nació sin factura (P-3)

**Puerta:** P-3 aprobada, su especificación escrita y la Fase 1 integrada.

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 3.1 | La especificación: reglas, API, pantallas, pruebas y las preguntas que deja (entre ellas, la 7 de §3.2) | M | — |
| 3.2 | `procurement`: la consulta «tuvo una línea alguna vez» sobre `purchase_invoice_items` en cualquier estado, con su índice por unidad (V131); la revisión del VIN dice por cada unidad que vincula si se puede corregir, y si no, por qué | M | 3.1 |
| 3.3 | `procurement`: cada línea del registro lleva `correctUnit`. Con él y una unidad elegible, una diferencia de modelo o año no es un rechazo, y la línea copia el modelo y el año del comprobante y la marca comercial de la unidad. Sin él, PRC-R44 no cambia | M | 3.2 |
| 3.4 | `fleet`: `PurchasedUnitLine` suma la corrección; `PurchasedUnitRegistrations` revisa lo que solo `fleet` sabe: que no esté anulada, que no tenga contratos (`UnitContractFacts`, D-250), que el modelo nuevo esté activo con su marca y su tipo (D-108) y sea del mismo tipo de vehículo, y que el color esté activo. `Vehicle.correctFromPurchaseInvoice(…)` es la única ruta que cambia el modelo; `modelId` pasa a escribirse; `markModelsInUse` marca los modelos nuevos en la misma llamada que los de las unidades nuevas (D-107); cada campo corregido escribe su fila en la bitácora | L | 3.3 |
| 3.5 | Frontend: en cada línea con diferencias y una unidad que se puede corregir, el aviso de la Fase 1 y la casilla «Corregir la unidad con el comprobante»; al guardar, una pregunta con la tabla VIN, Campo, Unidad, Comprobante; `lineDraftOf` envía lo del comprobante y la corrección | M | 3.3 |
| 3.6 | Pruebas: corrige los cuatro campos y escribe la bitácora; lee la unidad de la base para probar que el modelo se guardó (H2); rechaza otro tipo de vehículo, una unidad que tuvo factura y una con contrato; sin `correctUnit` sigue el rechazo; el bloqueo de una factura que corrige contra la edición de la misma unidad; las sentencias no crecen con las líneas (PRC-R65); specs y e2e del formulario | M | 3.4, 3.5 |
| 3.7 | La decisión en el diccionario, sus reglas, la especificación de facturas, [fleet](contextos/fleet.md), [procurement](contextos/procurement.md), procesos P4 y DESIGN.md | S | 3.6 |

**Aceptación:** una unidad escrita a mano como modelo A 2025 se vincula a una factura cuyo Excel dice modelo B 2026, del mismo tipo: tras confirmar, la unidad y la línea son B 2026, y la bitácora tiene dos filas con el número de la factura. Una unidad que ya tuvo factura, que tuvo un contrato o cuyo modelo nuevo es de otro tipo no se corrige, y la línea dice por qué.

### Fase 4 · Seguimiento de las unidades sin factura (P-4)

**Puerta:** la consulta de 3.2 integrada.

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 4.1 | `reporting`: «Sin factura», la lista de las unidades sin línea vigente, con si nunca la tuvieron o la perdieron (factura anulada, nota de crédito), su estado y los días desde el alta o desde que la perdieron, en el alcance de la marca comercial | M | 3.2 |
| 4.2 | Un indicador con la cuenta en «Indicadores» | S | 4.1 |
| 4.3 | El aviso a los N días, cuando el negocio fije N y `notifications` tenga ese caso | — | Pregunta 4 |

**Aceptación:** el reporte separa las unidades nacidas a mano de las que esperan la factura que reemplaza a una anulada, y cuenta sus días.

### Fase 5 · Alta en lote (P-5, condicional)

**Puerta:** el negocio confirma que llegan lotes antes de su factura **y** que necesita verlos en Inventario antes de tenerla (preguntas 1 y 2).

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 5.1 | Backend: `POST /vehicle-registrations`, de 1 a 100 unidades, todo o nada, sobre `VehicleRegistrations`, con las revisiones de F1 por lote, como `PurchasedUnitRegistrations` hace con las unidades nuevas | M | Fase 2 |
| 5.2 | Frontend: la vista «Nuevas unidades» con su tabla de líneas y la plantilla de Excel de la factura sin Valor ni ISC, para que un mismo archivo sirva a las dos | L | 5.1 |
| 5.3 | Pruebas, DESIGN.md y documentación | M | 5.2 |

---

## 6. Riesgos

| N.º | Riesgo | Cómo se evita |
|---|---|---|
| R1 | El modelo no se guarda y nadie lo nota (H2: `updatable = false`) | La prueba lee la unidad de la base, no del agregado |
| R2 | Un modelo nuevo cambia lo que pide la habilitación: gas, pantalla de fábrica (H8) | Solo el mismo tipo de vehículo; los requisitos se calculan al leer y Habilitación muestra lo que falte; la especificación decide si solo se corrige en el estado inicial |
| R3 | Un Excel equivocado pisa datos buenos | La confirmación con la tabla y la bitácora; la pregunta 7 |
| R4 | El registro de 100 líneas se vuelve lento | Las lecturas en lote y la prueba de sentencias (PRC-R65); medir con la cuenta de la aplicación |
| R5 | Un interbloqueo entre la factura y la edición de la unidad | La corrección va bajo el bloqueo que la factura ya toma (especificación de facturas, §2.4) y `markModelsInUse` en una sola llamada |
| R6 | Las unidades dadas de alta a mano hacen fallar su factura en la carga inicial (PRC-R142) | Registrar a mano sus facturas antes de migrar (pregunta 6), o extender después la carga con la regla de P-3 |
| R7 | Las pruebas con `postgres` esconden fallas de Row Level Security | Correr con `erp_mm_app` (Fase 0) |

---

## 7. Cómo se construye

- Una rama por fase desde `develop` en cada repositorio que toque.
- La especificación antes del código en las Fases 3 y 5.
- Las pruebas de cada regla primero. El backend se prueba con `erp_mm_app` sobre `erp_mm_db_verify`, nunca con el `.env`, que apunta a la base desplegada.
- Cada cambio de pantalla, con su revisión de DESIGN.md y su «Checked by».
- La documentación cambia en el mismo cambio que la regla ([contextos](contextos/README.md)).
- Commits por paso, con mensajes convencionales en inglés.

---

## 8. Lo que no se hace, y por qué

- **Corregir el modelo en Inventario.** Con P-3 no hace falta, y abriría una segunda vía para un dato que decide el comprobante.
- **Una vista propia para una sola unidad** (§2.4).
- **Tomar la marca comercial de la factura.** El comprobante no la trae: es una decisión interna de la empresa.
- **Pedir la factura en la puerta de Disponible.** Bloquearía la habilitación temprana, que es legítima; P-4 la hace visible.
