# Revisión · El saldo a favor medido por vehículo: lo que se paga de cada vehículo, el reparto guardado, la inicial congelada y el rastro de cada nota

**Fecha:** 10/11/2026 · **Lo revisado:** «Registrar pago» en `dev`, como lo dejaron D-246 y la revisión 74 de `DESIGN.md`, y la regla del saldo a favor de D-190 frente a una nota sobre un vehículo ya pagado · **Cierre:** diccionario v1.53, D-247 a D-249, decididas antes de construirlas (documentación `3a5ecc3`, `1554357` y `051e731`), y v1.54, construidas; backend, merges `ba5bbb15` (V127) y `4639799f` (V128 y V129); frontend, merges `8aa05753` (revisión 75), `d66ecb84` y `96b1a3f6` (revisión 76); todo en `develop`, que Flyway lleva a la base de `dev` · **Alcance:** lo que pidió el responsable, el plan y sus decisiones, lo construido en cinco fases, lo que se fijó al construir, cómo se verificó y lo que queda.

El responsable revisó «Registrar pago» en `dev` y pidió cuatro cosas: ordenar «Cómo se paga»; pagar una parte de un solo vehículo y montos distintos a varios, también con saldo a favor; que la nota de un vehículo ya pagado pueda pagar otra factura del mismo proveedor, sin cambiar lo facturado y con la factura original debiendo todavía el monto completo de sus otros vehículos, con el registro de por qué el vehículo vale menos y de en qué se usó la nota; y que en el concesionario, con un vehículo saldado por 42 000 de Millennium y 8 000 de inicial, una nota posterior de 10 000 pase entera a saldo y la inicial siga en 8 000.

**Veredicto.** Dos errores nuestros y un vacío de la regla. La revisión 74 llegó a `dev` sin que nadie la mirara y rompió «Cómo se paga» (hallazgo 1). D-246 perdió el monto por vehículo que la pantalla anterior tenía (hallazgo 2). Y la regla del saldo, medida por factura entera, no sabía qué hacer con una nota sobre un vehículo ya pagado mientras otro debía: el exceso no era de nadie, la factura podía quedar pagada con un vehículo en pago parcial, y una nota sobre un saldado del concesionario achicaba la inicial (hallazgo 3). Tres decisiones del diccionario, D-247 a D-249, tomadas antes de construir sobre un plan auditado contra el código, y construidas el mismo día de punta a punta.

---

## Lo que pidió el responsable

- **«Cómo se paga»** mostraba un campo por fila.
- **Pagar una parte de un vehículo,** y montos distintos a varios, también con saldo a favor.
- **La nota de un vehículo ya pagado** paga otra factura del mismo proveedor: lo facturado no cambia, queda registrado por qué el vehículo vale menos, la factura original sigue debiendo el monto completo de los otros vehículos y queda registrado en qué se usó la nota.
- **En el concesionario,** con un vehículo saldado y una nota posterior de 10 000, los 10 000 pasan a saldo de Millennium y la inicial sigue en 8 000.

## El plan y las decisiones

Un plan contrastó lo que había con lo que se quería, verificado en el código, y lo llevó de punta a punta en cinco fases. Lo auditó un revisor independiente contra el código, y las decisiones que surgieron las tomó el responsable:

- **Los pagos sin vehículo guardan su reparto.** «Todos los vehículos pendientes», «Otro monto», el anticipo que pasa de una orden a su factura y la migración se reparten al registrar, y el reparto se guarda como imputaciones marcadas como repartidas, fuera de los límites de 100 y 500.
- **El rastro se fija al registrar.** Un pago con saldo guarda qué notas consumió, empezando por la registrada primero, y no cambia nunca; una nota cuyo saldo se usó no se anula hasta anular ese pago, y ninguna anulación deja un uso sin respaldo.
- **El concesionario saldado** guarda su inicial al saldar, y acepta otro pago solo por lo que le falte además de ella.

Quedaron en el diccionario v1.53 antes de abrir la construcción: D-247, la pantalla, que enmienda D-246; D-248, el saldo medido por vehículo, con la identidad **valor − cubierto − inicial + pasó a saldo = pendiente**, que enmienda D-190, D-197, D-199 y D-211; y D-249, el rastro fijo, que enmienda D-195.

## Lo construido

- **Fase 1, frontend, revisión 75** (merge `8aa05753`: `31dde635`, `f3d7311f`, `0ababba4`, `b5c1aea1` y `2704c7ca`): «Cómo se paga» en dos filas por lo que significan, sobre la cuadrícula de 12 columnas de Registrar factura —el método y la cuenta; la fecha, la operación y la constancia—, y el dinero de los vehículos al final de su fila.
- **Fase 2, frontend, D-247** (merge `d66ecb84`: `e988ff8e`, `4b672db5`, `b41a97c1` y `9b84ffb6`): en «Solo algunos vehículos» cada vehículo marcado lleva su monto, todo lo que debe o una parte, propuesto hasta lo que queda del saldo y nunca bajado por él; el primer monto inválido toma el foco; y la copia de un pago parcial se queda en «Solo algunos vehículos» con sus montos.
- **Fase 3, backend, D-248, V127** (merge `ba5bbb15`: `8ca6f9c0`, `df4a3df2`, `bbe89925`, `c5444af2`, `afc6b76a`, `0f6d7ead` y `678e7b11`): `payment_applications.spread` y `payment_allocations.customer_initial`, con el relleno de las imputaciones que ya saldaban y su restricción; la regla por vehículo en `UnitCoveragePolicy`, `AmountOwedPolicy`, `InvoicePaymentStatusPolicy` y `SupplierBalancePolicy`; el reparto guardado al registrar, en el paso de la orden a la factura y en la migración; `toBalance`, `customerInitial`, `pending` y `current` en la posición, la factura, Por pagar y lo que se le debe al proveedor; el pago de la unidad en `fleet`; y en `reporting`, «Diferencia por inicial» y la columna 45 con la inicial congelada.
- **Fase 4, backend, D-249, V128 y V129** (merge `4639799f`: `0bc4bf39`, `e39fd542`, `cb972403`, `92f22198`, `7803e980` y `59de8fee`): `supplier_balance_uses`, que la aplicación lee e inserta y nunca edita ni borra; `BalanceTracePolicy`; los usos de cada pago con saldo, guardados con el proveedor bloqueado; `CREDITNOTE_BALANCE_USED` y `SUPPLIERPAYMENT_BALANCE_USED`; lo que dicen la nota, el pago, el saldo, lo que se le debe al proveedor y el costo de la unidad; y V129, una migración en Java que da sus usos a los pagos con saldo registrados antes, en el orden en que se hicieron.
- **Fase 5, frontend, revisión 76** (merge `96b1a3f6`; parte A: `1796c27c`, `9025e808`, `a6247590` y `051c421b`; parte B: `ae87e4d7`, `8be18746`, `0272b66e` y `a0ef2f05`): «Pasó a saldo a favor» en la franja de la factura, su desglose, el formulario y el proveedor; lo de cada vehículo en las líneas de la factura; «Saldo a favor» en la ficha de la nota; «De dónde salió el saldo» en un pago con saldo; las notas del saldo en el proveedor y en el aviso del formulario; lo que pasó y dónde se usó cada nota en «Compra y costo»; el aviso de un vehículo ya pagado en una nota nueva; y los rechazos de un saldo ya usado.

## Lo que se fijó al construir

- **Los ejemplos de la especificación de pagos** son el 8 al 11: el 6 y el 7 ya eran del tipo de cambio y del paso de la orden.
- **`differenceKind`** es `PENDIENTE_DE_PAGO` siempre que algo está pendiente, e `INICIAL_DEL_CLIENTE` solo cuando nada está pendiente y la inicial es mayor que cero.
- **Un vehículo saldado sigue `SALDADA`** aunque le falte algo además de su inicial: lo dice su `pending`, y se le puede pagar sin saldar.
- **El reparto** es en proporción a lo pendiente de cada vehículo vigente y no saldado; una aplicación que los vehículos no pueden recibir se guarda como el fondo de antes, con `spread` falso.
- **La exportación de los pagos** cuenta en «Unidades imputadas» también las imputaciones repartidas; el tope de 500 y la comparación de un envío repetido no las cuentan.
- **`…_BALANCE_USED`** se revisa antes que `…_BALANCE_NEGATIVE`, y nombra solo las fuentes que la anulación deja cortas; lo que ninguna fuente puede dar se guarda como «sin nota».
- **`usedIn` de `fleet`** es por nota, repartido en el orden de los usos para mostrarlo, solo en la lectura de una unidad y con `pagos.ver`.
- **Por pagar del proveedor** (`/payables`) trae `balanceSources` arriba, filtradas por la moneda pedida; el saldo (`/balance`), `sources` en cada moneda.
- **La tabla de vehículos de la factura** junta VIN y placa; marca y modelo con año, color y marca comercial; los montos; y estado y pago, para que «Pendiente» entre a 1440 px.

## Después, el mismo día: la revisión 77

El último pedido del responsable, con capturas de `dev` en una pantalla ancha (frontend `cfd96827`, `27fcb725`, `7368994f`, `015d9c25`, `a87e7907` y `c166c27b`, merge `3e53ee53`):

- **«Cómo se paga» en una fila** desde que la tarjeta mide 80rem: el método (3), la cuenta (5), la fecha (2) y la operación (2), y debajo la constancia (6) y el tipo de cambio (6); entre 56rem y 80rem siguen las dos filas de la revisión 75.
- **La caja de la cuenta, más corta:** «BCP Soles PEN ···0045»; la lista conserva el banco.
- **La retención** con sus campos y su desglose en una fila, el desglose a la derecha.
- **Cada monto de un desglose** en la línea de su etiqueta, con un estilo compartido por toda la aplicación.
- **«Quitar la nota»,** que la vacía y devuelve el foco a «Agregar una nota».
- **El saldo nombra su nota** como «la nota NC01-50», y no «NC NC01-50» (frontend `376580dc`, merge `36206d93`).

Se verificó con las unitarias de lo tocado, las e2e a 1920, 1440 y 1024 px, la retención sin violaciones de accesibilidad y las imágenes que el desglose movió, regeneradas y miradas una por una. Al día siguiente, la revisión 78 (merge `0f6d55db`) y la tasa de la factura en cada aplicación (D-213; merges `aaff72d1` y `b22bbb79`) siguieron sobre el mismo formulario y la ficha del pago, y después las revisiones 79 a 85 y la ayuda de la retención (merges `31b51341` a `28da7f89`) ([contextos/procurement.md](../contextos/procurement.md), sección 6). La tasa dejó a Por pagar respondiendo 500 en `dev` hasta el backend `80a4fb23` ([Por pagar respondió 500](2026-11-11-por-pagar.md)).

## Cómo se verificó

- **Backend:** la suite completa en una base local nueva, con las dos cuentas: 3 996 pruebas en la fase 3 y 4 021 en la fase 4, 0 fallos y 2 omitidas, con `ModularityTests` en verde. El OpenAPI publica cada campo nuevo con el nombre que lee el frontend.
- **Frontend:** `ng build` sin advertencias, `npm run lint:styles` y las unitarias de lo tocado —89 en la fase 1, 140 en la 2, 315 en la 5A y 273 en la 5B—; las e2e de cada fase en local, con la aprobación del plan —51 en la fase 1; 57 y 54 de accesibilidad en la 2; 211 y 117 de accesibilidad en la 5A; 66 y 42 en la 5B—; y las imágenes regeneradas, miradas una por una.

## Bien hecho

- **Decidido antes de construir,** sobre un plan auditado contra el código, como D-244 a D-246.
- **El plan midió primero lo que había.** Cada «hoy» del plan se verificó en el código, y así aparecieron la factura pagada con un vehículo en pago parcial y los 8 000 que se perdían en el concesionario, que ni pruebas ni documentos cubrían.
- **Las pruebas antes que el código,** en pares test/feat, con los nombres que el plan había escrito.
- **Ninguna pantalla se integró sin mirarla:** las imágenes de cada fase se miraron una por una, la lección de la revisión 74.
- **Las fases 3 a 5 se integraron juntas,** para que `dev` no mostrara pantallas desfasadas de su backend.

## Hallazgos

### 1. La revisión 74 se publicó sin mirarla (error)

- **Error:** la revisión 74 de `DESIGN.md` llegó a `dev` con «Cómo se paga» roto: `.payment-section`, en columna, iba después de la cuadrícula del `@container` con la misma especificidad y la anulaba, y las filas en `display: contents` apilaban cada campo. Las pruebas no lo vieron: `document-forms.spec.ts` todavía pedía la columna de 60rem, y las imágenes de referencia eran del 05/11. Lo que la revisión 74 de `DESIGN.md` decía no llegó a la pantalla, y nadie la miró antes de integrarla.
- **Corrección:** la revisión 75, con la cuadrícula declarada al final y sin `dense`, la e2e que mide el ancho y la fila de la constancia, y las imágenes regeneradas y miradas (frontend `f3d7311f`, `31dde635`, `b5c1aea1` y `2704c7ca`, merge `8aa05753`).
- **Próxima vez:** nada de diseño se integra sin mirar su pantalla en los tres anchos; la e2e que mide un diseño cambia en el mismo cambio que el diseño, y una imagen de referencia más vieja que la pantalla se regenera antes de integrar.

### 2. D-246 perdió el monto por vehículo (error)

- **Error:** con D-245, «Asignar a unidades» dejaba escribir un monto por unidad, como lo decidió el diccionario. La pantalla de D-246 lo perdió: «Solo algunos vehículos» pagaba todo lo que le faltaba a cada marcado, «Otro monto» repartía entre todos, y «Anular y copiar» convertía una imputación parcial en «Otro monto» (`supplier-payment-form-view.ts`, la copia). El backend lo seguía aceptando (D-211), así que la pantalla nueva hacía menos que la anterior sin que nadie lo hubiera decidido.
- **Corrección:** D-247 y la fase 2: el monto de cada vehículo marcado, también con saldo, y la copia que lo conserva (frontend `4b672db5`, `e988ff8e`, `b41a97c1` y `9b84ffb6`, merge `d66ecb84`).
- **Próxima vez:** antes de rediseñar una pantalla, se hace el inventario de lo que hacía la anterior —cada acción, cada entrada y cada monto que envía— y la nueva se revisa contra esa lista antes de decidirla.

### 3. El saldo medido por factura no sabía de vehículos (aprendizaje)

- **Error:** D-190 midió el saldo por factura entera: solo lo aplicado de más era saldo. Con una nota sobre un vehículo ya pagado mientras otro debía, la factura bajaba lo que debía, el exceso del vehículo no era de nadie y el otro seguía debiendo todo; la factura podía quedar `PAGADA` y salir de Por pagar con un vehículo en `PARCIAL`; pagar ese vehículo respondía `SUPPLIERPAYMENT_EXCEEDS_OWED`; y en el concesionario la inicial era lo que le faltaba al vehículo, así que una nota la achicaba y de 10 000 pasaban 2 000 a saldo. Los pagos sin vehículo eran un fondo que se volvía a repartir en cada lectura, así que una nota sobre un vehículo cambiaba lo pendiente de otro. Ni las pruebas ni los documentos cubrían el caso: la regla no estaba escrita para él.
- **Corrección:** D-248 y D-249: la regla por vehículo, el reparto guardado, la inicial congelada y el rastro de cada nota (backend, merges `ba5bbb15` y `4639799f`; frontend, merge `96b1a3f6`). Lo prueban, entre otras, `UnitLevelBalanceIntegrationTests.theExampleOfTheNoteOnAPaidUnitWhileAnotherStillOwes` y `theExampleOfTheNoteOnASettledDealerUnit`.
- **Próxima vez:** una regla de dinero que vale para un documento entero se prueba también con el documento a medio pagar —una parte pagada y otra no— y con cada hecho que llega después de pagar.

### 4. Los números de los ejemplos chocaban (aprendizaje)

- **Error:** el plan llamó 6 a 9 a los ejemplos nuevos, pero la especificación de pagos ya tenía el 6, del tipo de cambio, y el 7, del paso de la orden, que `contextos/procurement.md` cita en PRC-R124 y PRC-R126.
- **Corrección:** los ejemplos nuevos son el 8 al 11, en la especificación de pagos, versión 12, y las e2e y las pruebas de la fase 5 los citan así.
- **Próxima vez:** un plan que nombra un ejemplo, una regla o una revisión por su número lo busca antes en el documento donde vivirá.

### 5. Los pagos con saldo de antes no tenían rastro (aprendizaje)

- **Error:** la fase 0 dejó abierto qué usos tenían los pagos con saldo registrados antes de V128: sin usos, lo disponible de una nota habría salido mayor de lo que era.
- **Corrección:** V129, una migración en Java que lee los hechos de cada proveedor como el registro y da a cada pago con saldo vigente sus usos, en el orden en que se hicieron, con la misma regla; es idempotente (backend `7803e980`; `BalanceUsesMigrationTests.theMigrationGivesEachPaymentWithTheBalanceItsUsesInTheOrderTheyWereMade`).
- **Próxima vez:** una tabla nueva que guarda un hecho que antes se calculaba se llena para las filas anteriores en la misma entrega, y el plan lo dice.

## Lo que queda abierto, con su disparo

Cada punto está en los pendientes del [orden de implementación](../orden-de-implementacion-millennium-mobility.md) o de [contextos/procurement.md](../contextos/procurement.md):

- ~~**En curso: el último pedido del responsable del 10/11/2026,** la fila de la retención con su desglose a la derecha, «Cómo se paga» en una fila de cuatro en una tarjeta ancha (desde 80rem) con un activador de la cuenta más corto, y «Quitar» para la nota (revisión 77 de `DESIGN.md`, rama `fix/payment-how-wide`). No está construido.~~ Construido el mismo día (merge `3e53ee53`; abajo).
- **La sonda en vivo** (`e2e-live`) está desactualizada más allá de este trabajo: su fuente de tipos de cambio, `Sonda …`, se rechaza y tiene que ser SBS, BCRP u otra admitida; `procurement.live` espera «Desactivar» como botón, que está en el menú ⋮; y el paso 1 de `supplier-payments.live` lee el pedido `multipart` del formulario anterior a D-246. Se pone al día como una tarea propia. Puesta al día y corrida el 11/11/2026 contra un backend local de `develop`: 129 pasan, 1 se omite y ninguna falla, sin ningún defecto del producto (frontend, merge `17a350e0`; [contextos/procurement.md](../contextos/procurement.md), sección 6).
- **Las imágenes de referencia** de la ficha del pago, de Por pagar y de la lista de pagos ya estaban desactualizadas en `develop` antes de este trabajo: se regeneran juntas. Las de Por pagar se regeneraron el 11/11/2026 con la revisión 84.
- ~~**La tabla de unidades de la ficha de la nota** todavía se desplaza de lado a 1440 px.~~ Resuelto el 11/11/2026 por la revisión 86 (merge `565d494f`): «VIN y placa» y el año bajo el modelo.
- ~~**Por pagar a 1024 px** se desplaza de lado y corta «Acciones».~~ Resuelto el 11/11/2026 por la revisión 84 (merge `0326ccb1`): la fecha va bajo la antigüedad y las celdas son compactas.
- ~~**La ficha del proveedor a 375 px:** «Pagar lo pendiente» y «Ver pagos» no ocupan el ancho.~~ Resuelto el 11/11/2026 por la revisión 86 (merge `565d494f`): cada enlace crece a lo ancho de su línea.
- **Dos clases pasan de 300 líneas:** `SupplierPaymentViewFactory` (356) y `SupplierPaymentQueryServiceImpl` (366).
- **El registro de un pago con saldo** no tiene una prueba propia de sus sentencias.
