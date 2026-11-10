# Revisión · Los pagos vistos por el responsable el 09/11/2026: primero el diseño, el monto que ya no se escribe, un solo vocabulario, y las listas y las hojas de compras

**Fecha:** 09/11/2026 · **Lo revisado:** el registro, la lista y la ficha de los pagos a proveedores, Por pagar, y las listas y las hojas de compras, en `dev.erp.millennium.ky`, con los datos de demostración del mismo día · **Cierre:** diccionario v1.50, D-245 (documentación `2bba268`); backend `70dd253b`, `8f84b19c` y `0ca99f14`, en `develop` con el merge `b5308771`, y `600c4859`, con el merge `7f76c5bc`; frontend, la revisión 64 de `DESIGN.md` (`c9bb84f4` a `1491f217` y `01a951be`, merge `52cf8b2b`), la 65 (`3e9d4321` a `e282f746`, merge `69887f1b`), la 66 (`e5741f38` a `1f00e2e3`, merge `5f728fc3`), la 67 (`197d663b` a `9a6c0f0c`, merge `7ae68c2d`) y `a0724b7a`; todo en `develop` y publicado ese día; sin migraciones · **Alcance:** lo que el responsable pidió, la propuesta y sus decisiones, lo que se construyó ese día, la regla de verificación que acotó y lo que queda.

En la [revisión de las facturas](2026-11-09-revision-de-facturas.md) de la mañana, el responsable había pedido a los pagos una pantalla más apropiada, y D-244 dio a la factura su estado de pago. Por la tarde siguió con los pagos mismos y con las listas de compras. Mandó capturas de las listas —en Facturas, a 1440, 1680 y 1920 px, los días nunca llegaban a la línea de la búsqueda y los filtros, y poner un filtro los mandaba a otra línea—, pidió que las hojas de la orden y de la plantilla dejaran el azul primario, el número de una orden como enlace con el RUC del proveedor y Comprobante centrado. De los pagos dijo «primero diseña»: un diseño antes de tocar el código.

**Veredicto.** Ningún defecto del producto: el registro de un pago funcionaba y cumplía cada regla, pero se leía como una hoja de cálculo. Una decisión del diccionario, D-245, y cuatro revisiones de `DESIGN.md`, construidas e integradas en `develop` el mismo día.

---

## Lo que pidió el responsable

- **Las listas,** con capturas: los encabezados centrados y los días en la línea de los filtros, también con un filtro puesto, para toda lista de `mm-data-manager`, y el cambio en el componente compartido, no lista por lista.
- **Las hojas de la orden y de la plantilla:** no le gustó el azul primario de las bandas.
- **Órdenes de compra:** el número de cada fila como enlace a su ficha, como el de una factura, y el RUC del proveedor bajo su razón social, como en Facturas.
- **Facturas:** Comprobante centrado, con el tipo bajo el número.
- **Los pagos:** «primero diseña». La pantalla se construye después de ver y aprobar una propuesta.

## La propuesta

La [propuesta](https://claude.ai/artifact/AWriMyMNW1MTtsuePGfdGq) del mismo día midió el formulario: una tabla de unos 73rem que se desplazaba de lado casi en cualquier ancho, un monto escrito aparte que tenía que cuadrar con ella, y una sola razón a la vez de lo que retenía el registro. Propuso seis principios sobre las mismas reglas —los principios 3, 4 y 5 para el formulario, que serían la revisión 66, y los 1, 2 y 6 para lo que lo rodea, la revisión 67— y tres preguntas, cada una con su opción recomendada.

## Las decisiones del responsable

Aprobó la propuesta con las tres opciones recomendadas:

- **El monto calculado.** El monto del pago no se escribe: es la suma de lo marcado en «Qué se paga», en la moneda del pago, así que el pago se aplica completo por construcción (D-210). El backend sigue recibiendo `amount` y comparándolo. Enmienda la regla de la revisión 52, «el monto nunca se llena solo».
- **Un solo vocabulario en toda la aplicación:** «Pendiente» por lo que se debe, «Pagado» por lo aplicado más la retención, «Asignar a unidades» por imputar y «Concesionario» por dealer, en Compras y en la ficha de la unidad; «Saldo a favor» se queda para el crédito del proveedor.
- **Todo junto:** el formulario y lo que lo rodea a la vez —Por pagar, la ficha del proveedor, la lista y la ficha del pago—, y no por etapas.

Se decidió como D-245 en el diccionario antes de abrir las ramas (documentación `2bba268`). Dejó abierta una elección, qué hace Por pagar cuando se marca una factura de otro proveedor, y se tomó la que no pierde lo elegido: decirlo y retener el pago, sin reemplazar la selección. Y acotó ese día la regla de verificación (hallazgo 5).

## Lo construido

- **El backend de los pagos** (`develop`, merge `b5308771`): cada pago, en las filas de `GET /api/v1/supplier-payments` y en su ficha, nombra los documentos que pagó, `documents: [{type, id, number}]`, con `type` `INVOICE` u `ORDER`, en el orden en que se aplicaron y cada uno una vez (`8f84b19c`); `q` encuentra además un pago por el número de toda factura u orden que pagó, sin mayúsculas ni tildes, la factura también como la guarda su registro, así que `F002-00007741` encuentra `F002-7741` (`70dd253b`); la expresión del número de la orden pasa a `PurchaseOrderRepositoryImpl.numberOf`, que comparten las dos búsquedas; la exportación sube su versión a `supplier-payments-layout=3`; y una página prepara 17 sentencias con 1, 10 o 40 filas (`0ca99f14`). [contextos/procurement.md](../contextos/procurement.md), PRC-R171.
- **El backend de las órdenes** (merge `7f76c5bc`): toda respuesta de una orden nombra el documento tributario de su proveedor, `supplierTaxId` (`600c4859`).
- **El frontend, revisión 64** (merge `52cf8b2b`): los encabezados centrados en toda lista de `mm-data-manager`; `centered` y `fullHeader` de `mm-data-column`, con «UND» en Facturas y Notas de crédito; la barra con los días tras los filtros y las acciones al final, «Limpiar filtros» primero, y la búsqueda que parte del espacio de su texto de ayuda; Facturas con Proveedor en `.mm-filter--medium` (12,5rem) y Estado en `--short`, así que sus días llegan a la línea de los filtros desde 1680 px; y el aviso «Todo facturado» de la orden, que ocupaba unos 300 px, con la altura de sus líneas (`01a951be`).
- **El frontend, revisión 65** (merge `69887f1b`): el número de la orden abre su ficha y la lista lee el RUC bajo la razón social; las hojas de la orden y de la plantilla toman `table-header` en sus bandas y en la cabecera de sus líneas, TOTAL su peso sin relleno y el título `text-primary`, mientras el PDF conserva sus colores; y Comprobante, centrada entera, con el tipo de todo documento bajo el número, como texto.
- **El frontend, revisión 66** (merge `5f728fc3`, 7 commits): «Registrar pago» en dos pasos —«Qué se paga», los documentos como filas, tarjetas en el teléfono; «Cómo se paga», el método como control segmentado y la cuenta como tarjetas— y un «Resumen» fijo con el total, las líneas, la lista de todas las razones que retienen el pago y «Registrar pago de USD X». El monto se calcula de lo marcado (`PaymentFigures`) y ya no hay campo; las unidades se asignan dentro de la fila (`payment-unit-allocations`) y el diálogo se va; un concesionario llega con el reparto en proporción propuesto; `?facturas=` marca varias facturas; y el campo de la tasa suma `significantDecimals`.
- **El frontend, revisión 67** (merge `7ae68c2d`, 11 commits): Por pagar elige facturas de un proveedor y una empresa y «Pagar N facturas» abre el formulario con `?facturas=`, desde una barra bajo la lista (`selectionAt="end"` de `mm-data-manager`); la ficha del proveedor, «Pendiente y saldo a favor» con «Pagar lo pendiente»; la lista de pagos, Fecha, Proveedor, «Pagó», «Cómo», Monto y Estado, con la búsqueda nueva; la ficha del pago, una franja con el monto, cómo, la fecha, el estado y «Pagado a», el proveedor como enlace con su RUC y «Concesionario»; y el vocabulario en la factura, el proveedor, Por pagar y «Compra y costo» de la unidad, con el código ISO también ahí.
- **Los textos** (`a0724b7a`): los rechazos de un registro y el nombre del permiso, «Registrar pagos y asignarlos a unidades».
- **Verificado:** en el backend, `SupplierPaymentsIntegrationTests`, 50 de 50; `SupplierPaymentListStatementsTests`; `SupplierPaymentsExportIntegrationTests`, 5 de 5; `PurchaseOrderSearchIntegrationTests`, 9 de 9; y `ModularityTests`. En el frontend, `ng build`, las unitarias de lo que se tocó y `npm run lint:styles`; con la revisión 64, además, `toolbar-rows.spec.ts` y las imágenes de toda lista (`1491f217`), y desde la 65 ninguna e2e (hallazgo 5).

## Bien hecho

- **Primero el diseño, después la decisión, después el código.** La propuesta, la aprobación del responsable y D-245 en el diccionario llegaron antes de la primera rama, como D-244 en la mañana.
- **El monto calculado como el backend.** La pantalla suma los equivalentes de las filas marcadas, cada uno redondeado mitad hacia arriba como el backend, así que el pago cuadra sin que una fila absorba un céntimo, y el backend sigue comparando `amount`: ninguna regla cambió de lugar.
- **Una página cuesta lo mismo con cualquier número de filas.** `documents` se arma con las referencias que la página ya lee, y `SupplierPaymentListStatementsTests` lo fija con las estadísticas de Hibernate, como `PurchaseInvoiceListStatementsTests` con D-244.
- **Una sola expresión para el número de una orden.** `PurchaseOrderRepositoryImpl.numberOf` sirve a la búsqueda de las órdenes y a la de los pagos, en lugar de una copia.
- **La exportación no sirve un archivo viejo.** Su versión sube aunque las columnas no cambien, porque cambia lo que la búsqueda encuentra.
- **Otro proveedor se dice, no se reemplaza.** Quien eligió cuatro facturas no las pierde de un clic.
- **El vocabulario, solo valores.** Ninguna clave de `es.json` cambió.
- **El cambio de las listas en su lugar.** La barra y los encabezados cambiaron en `mm-data-manager`, y Facturas solo ajustó sus anchos.

## Hallazgos

### 1. El registro de un pago se leía como una hoja de cálculo (aprendizaje)

- **Error:** el formulario cumplía cada regla (D-194 a D-198, D-210 a D-215), pero «Aplicar a» era una tabla de unos 73rem que se desplazaba de lado casi en cualquier ancho, el monto se escribía aparte y tenía que cuadrar con ella, y el formulario decía una sola razón a la vez de lo que lo retenía (`DESIGN.md`, revisión 66). Se armó por capas —las revisiones 37, 51, 52 y 53—, cada una correcta, sin volver a mirar el recorrido entero.
- **Corrección:** D-245 y la revisión 66: dos pasos y un resumen, el monto calculado, filas en lugar de la tabla, las unidades en la fila y la lista completa de lo que falta (frontend `e5741f38` a `1f00e2e3`, merge `5f728fc3`).
- **Próxima vez:** un formulario que crece por capas se mide de punta a punta —lo que pide escribir dos veces, lo que se desplaza de lado, lo que calla— antes de sumarle otra; y lo que se puede calcular de lo ya marcado no se pide escribir.

### 2. Varias palabras para lo mismo (aprendizaje)

- **Error:** lo que se debe se leía «Se debe», «Saldo», «Por pagar» o «Pendiente de pago», según la pantalla; lo pagado, «Aplicado» o «Cubierto»; y el pago a unidades, «imputar», con «canal de dealers» o «Dealer».
- **Corrección:** D-245, la revisión 67 (frontend `dc442309` y `e10d7873`) y `a0724b7a`: «Pendiente», «Pagado», «Asignar a unidades» y «Concesionario», en Compras y en la ficha de la unidad.
- **Próxima vez:** una palabra para un monto se busca primero en `es.json`; si ya hay una, se usa esa, también en otro contexto.

### 3. Los días de la barra se movían al poner un filtro (aprendizaje)

- **Error:** la revisión 46 ponía los días y «Limpiar filtros» en un bloque que no se partía. Al aparecer «Limpiar filtros», el bloque dejaba de caber y bajaba de línea; en Facturas, a 1440, 1680 y 1920 px, los días nunca llegaban a la línea de la búsqueda (`DESIGN.md`, revisión 64). La barra se había medido sin filtros puestos.
- **Corrección:** la revisión 64 (frontend `c9bb84f4`, `6dcb303b` y `0d51b2b1`, merge `52cf8b2b`): dos bloques, los días tras los filtros y las acciones al final, «Limpiar filtros» primero; Proveedor y Estado más angostos en Facturas; y `toolbar-rows.spec.ts` mide los días en su línea con un filtro puesto.
- **Próxima vez:** una barra se mide también con un filtro puesto, porque lo que aparece al filtrar cambia su ancho.

### 4. El azul de las acciones en los encabezados de una hoja (aprendizaje)

- **Error:** las hojas de la orden y de la plantilla encabezaban sus recuadros, sus líneas y TOTAL en `primary` (revisiones 28 y 32), el color de las acciones, y el responsable no lo quería.
- **Corrección:** la revisión 65 (frontend `1efc4085`): `table-header` en las bandas y en la cabecera de las líneas, TOTAL por su peso y el título en `text-primary`; `primary` queda en los botones que agregan una línea. PRC-R29 lo dice desde ese día.
- **Próxima vez:** un encabezado toma el tono de un encabezado; `primary`, solo lo que se pulsa.

### 5. Las pruebas, acotadas otra vez (aprendizaje)

- **Error:** la regla de la mañana, de la revisión de las facturas, todavía corría en local las e2e y las imágenes de un cambio que solo se ve: con la revisión 64 corrieron `toolbar-rows.spec.ts` y se tomaron otra vez 105 imágenes (`1491f217`).
- **Corrección:** la regla del responsable, acotada ese día: un cambio acotado no corre en local ninguna e2e, tampoco las suyas; solo la compilación, las pruebas unitarias de los archivos que toca y el lint de estilos. GitHub Actions corre las suites completas en cada push. Desde la revisión 65, las e2e que el cambio alcanzaba se llevaron a las pantallas nuevas sin correrlas, y cada revisión de `DESIGN.md` anota qué imágenes quedaron viejas.
- **Próxima vez:** un cambio acotado corre la compilación, sus unitarias y el lint de estilos; las imágenes de las pantallas que cambió se regeneran juntas, en una tanda, y hasta entonces quedan anotadas.

## Lo que queda abierto, con su disparo

Cada punto está en los pendientes transversales del [orden de implementación](../orden-de-implementacion-millennium-mobility.md):

- **Las imágenes de referencia sin regenerar,** la consecuencia conocida de la regla: las de `purchase-orders`, `purchase-order`, `purchase-order-new`, `purchase-order-template`, `purchase-order-template-new` y `purchase-invoices` (revisión 65); las del formulario del pago en sus cuatro métodos (revisión 66); las de `payables`, `supplier-payments`, `supplier-payment`, `supplier`, `purchase-invoice` y la ficha de la unidad (revisión 67); y el catálogo. Las pruebas visuales de GitHub Actions pueden fallar hasta regenerarlas juntas.
- **`toolbar-rows.spec.ts`** sigue esperando la barra de Pagos a proveedores en una línea a 1440 px, que el texto de ayuda más largo de su búsqueda puede no dejar.
- **El nombre del permiso:** `DESIGN.md`, revisión 67, todavía dice que el nombre de `pagos.registrar_pago` se queda, «Registrar, aplicar e imputar pagos», por la imagen de la matriz de permisos; `a0724b7a` lo cambió después a «Registrar pagos y asignarlos a unidades». La revisión se corrige con el próximo cambio de `DESIGN.md`, y la imagen de la matriz entra en la tanda.
- **La tasa de «Compra y costo»** de la unidad conserva seis decimales, porque `fleet` no lee ayudas de compras.
- **La sonda en vivo de los pagos** (`e2e-live/supplier-payments.live.spec.ts`), que nunca corrió, maneja el formulario anterior —el campo Monto, «Aplicar lo que se debe», el método y la cuenta como listas—: se lleva al de la revisión 66 antes de correrla.
- **El catálogo** no tiene muestras de la franja de un pago ni de la barra bajo una lista.
- **Lo que dejó la revisión 63:** la columna «Monto en USD» de las líneas de la nota de crédito, sin moneda, y la lista de facturas, sin sus filtros en la dirección. Los desgloses de la unidad y del proveedor del catálogo, con su símbolo, se cerraron con la revisión 67.
