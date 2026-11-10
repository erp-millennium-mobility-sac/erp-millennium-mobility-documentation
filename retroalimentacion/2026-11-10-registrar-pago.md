# Revisión · Registrar pago visto otra vez por el responsable el 10/11/2026: una página en su orden, los vehículos de la factura, con qué se paga y la constancia como en las facturas

**Fecha:** 10/11/2026 · **Lo revisado:** «Registrar pago» (`/procurement/supplier-payments/new`), el formulario de dos pasos y un resumen que D-245 y la revisión 66 de `DESIGN.md` dejaron el día anterior · **Cierre:** diccionario v1.51, D-246 (documentación `b27c002`), decidida antes de construirla; la [especificación de pagos](../especificacion-pagos-a-proveedores-2026-11-03.md), versión 10 (`d198124`); [contextos/procurement.md](../contextos/procurement.md) (`ca57d0c`), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (`334945e`) y el [orden de implementación](../orden-de-implementacion-millennium-mobility.md) (`c947def`); frontend, la revisión 69 de `DESIGN.md`, merge `62304705`, y el merge `3a928954` y `4796ec7b`, con las revisiones 70 y 71; después, las revisiones 70 a 73 para todo formulario de página, las listas y las fichas, merges `6a02be91`, `2aa8c4c5`, `d59b2f31` y `de2d1c56`; diccionario v1.52; todo en `develop`; sin cambio en el backend ni migraciones · **Alcance:** lo que el responsable pidió a lo largo del día, la revisión basada en evidencia y los prototipos, sus respuestas y sus decisiones, lo que se construye y lo que queda.

El día anterior, en la [revisión de los pagos](2026-11-09-revision-de-pagos.md), el responsable aprobó un formulario en dos pasos, «① Qué se paga» y «② Cómo se paga», con un resumen al costado (D-245). Volvió a él: el formulario todavía no lo convencía. Pidió un análisis, vio una primera propuesta en pasos, dio su propio orden del pago y, después de una revisión basada en evidencia y tres prototipos, pidió construirlo de punta a punta.

**Veredicto.** Ningún defecto del producto: el formulario cumplía cada regla, pero no seguía el orden en que la persona piensa el pago. Una decisión del diccionario, D-246, tomada antes de abrir la construcción, sobre las mismas reglas y la misma API; la revisión 69 de `DESIGN.md`, construida el mismo día; y cuatro revisiones más, la 70 a la 73, que llevaron a todo formulario de página lo que el pago estrenó.

---

## Lo que pidió el responsable

- **El formulario de pagos todavía no lo convencía.** Pidió un análisis de «Registrar pago» como quedó el 09/11/2026.
- **Una primera propuesta, con pasos.** El análisis propuso un asistente con «Continuar» entre pasos, que se corrigió (hallazgo 1).
- **Su propio flujo:** «proveedor → factura → vehículos → cómo se paga, o con notas de crédito que son saldo a favor». Así piensa el pago, y así lo pidió en la pantalla.
- **Listas para el método y la cuenta,** en lugar del control segmentado y de las tarjetas de la revisión 66.
- **La constancia a la izquierda,** como el archivo de una factura.
- **«Implementa end to end».** Después de responder tres preguntas, pidió construirlo de punta a punta.

## La revisión basada en evidencia y el prototipo

La [revisión basada en evidencia](https://claude.ai/artifact/2281pRi3qUewX64cVdWEZf) midió el formulario de la revisión 66 contra lo publicado sobre formularios y pagos: NN/g, sobre los errores hostiles y prematuros, la revelación progresiva y los asistentes por pasos; los sistemas de diseño de GOV.UK y de CMS, sobre el resumen de errores, el botón que envía y nunca se deshabilita, y cuándo una lista y cuándo botones de opción; Baymard, sobre la validación en línea y la elección del medio de pago; y CXL y Speero, sobre la columna única y la lista frente a los botones de opción. La lista de lo que faltaba se leía desde que el formulario se abría, el botón se dejaba pulsar pero se veía deshabilitado, y el pago se pensaba como filas que marcar y no como un documento que se paga. Midió también que los botones de opción se eligen algo más rápido que una lista, y recomendó una acción aparte, «Usar saldo a favor», para pagar con el saldo.

Hubo tres prototipos; el último está [publicado](https://claude.ai/artifact/8nDuTVauqk28n4uta275vz): el proveedor y la factura u orden en una fila, la sección de la factura con sus vehículos, con qué se paga y la cuenta como listas, la constancia a la izquierda de los datos del pago, un resumen de errores arriba y el total al pie. Se revisó en un navegador sin cabeza antes de publicarlo (hallazgo 2).

## Las decisiones del responsable

Respondió las tres preguntas del prototipo:

- **¿Una factura por pago o varias?** Varias, con «Agregar otra factura»: una transferencia y una constancia pueden pagar varias facturas, así que «Pagar N facturas» de Por pagar (`?facturas=`) y la copia de un pago anulado siguen funcionando (hallazgo 3).
- **¿Dónde van las órdenes?** En la misma lista que las facturas, como «Orden … · anticipo».
- **¿Qué pruebas corren?** Solo las e2e de los pagos en local, además de la compilación, las unitarias de lo que se toca y el lint de estilos; ninguna otra e2e.

Y eligió listas para el método y la cuenta sabiendo que los botones de opción se midieron algo más rápidos, y «Saldo a favor» como un método más en lugar de la acción aparte. Se decidió como D-246 en el diccionario antes de abrir la construcción (documentación `b27c002`):

- **Una página, en su orden:** «Proveedor» y «Factura u orden» en una fila, con «Empresa» primero cuando hay varias; cada documento abre su sección, y «Agregar otra factura» suma otro.
- **Los vehículos de la factura** en una tabla, con su estado de pago, lo que tienen por pagar y «Se paga ahora», bajo las cifras de la factura; «¿Qué paga de esta factura?»: todos los vehículos pendientes, sin imputaciones; solo algunos, que se imputan; u otro monto, escrito, que el backend reparte en proporción. Una orden pide su anticipo.
- **El concesionario,** vehículo por vehículo, con «¿La pagó el cliente como inicial?» cuando un vehículo recibe menos de lo que le falta, que salda la unidad.
- **«¿Con qué se paga?»,** una lista, con «Saldo a favor» solo cuando hay saldo en la moneda de los documentos, retenido a quien registra por una marca, y dicho como lo piensa el responsable: el saldo lo dejan las notas de crédito que llegaron cuando su factura ya estaba pagada (D-190). Con saldo, el pago usa a lo sumo lo disponible.
- **La cuenta,** una lista, propuesta cuando es la única activa en la moneda de los documentos (enmienda D-214), nunca la última usada.
- **La constancia** como el archivo de una factura (revisiones 47, 48 y 53), y «Constancia» en lugar de «voucher».
- **Los errores:** ninguno al abrir; cada campo, después de editarlo y dejarlo; y «Registrar pago» siempre habilitado, con un resumen de errores arriba que toma el foco.
- **El total al pie,** en todo ancho; la diferencia de cambio, solo en la ficha del pago; la tasa, solo cuando hay una conversión.
- **Palabras llanas,** con un glosario: «Por pagar», «Cuenta de la que salió el dinero», «Constancia», «N.º de operación», «La diferencia la pagó el cliente», «Retención del IGV» y «Saldo a favor».
- **La retención,** opcional y escrita hasta que contabilidad confirme si Millennium es agente de retención del IGV.

Ninguna regla de D-194 a D-215 cambia, ni la API, ni lo que se envía.

## Lo construido

- **El frontend, revisión 69** (merge `62304705`: `17dae178`, `8d847064`, `aad45bfb`, `df5a873e`, `da44a3d5`, `532180aa` y `83d197a4`): «Registrar pago» en una página, `payment-document-panel` por documento y `payment-drafts.ts`, que deriva lo que se envía de lo que la persona decide. El pedido es el de antes, comprobado contra `UnitCoveragePolicy`: «Todos» manda lo que se debe sin imputaciones; «Solo algunos», a cada vehículo lo que todavía debe, su diferencia sin pasar lo que le falta; «Otro monto», lo escrito sin imputaciones; el concesionario, lo escrito en cada vehículo con `settlesUnit`; la orden, su anticipo; y el saldo, topado, los primeros documentos primero.
- **Lo llevó a las piezas compartidas** (merge `3a928954`: `feed875a`, `9161e8bf`, `db35a7d8`, `8c95df65`, `7367f8c9` y `956cdcff`; y `4796ec7b`): el pie `mm-form-foot`, el resumen `mm-error-summary` y la regla del foco de las revisiones 70 y 71, y el aviso «Se quitó …: el saldo a favor solo paga facturas en PEN.» cuando el saldo deja documentos fuera. «Constancia» reemplazó a «voucher» en los rechazos del formulario y en la ficha del pago.
- **Verificado:** 157 y después 154 pruebas unitarias de lo que se tocó; las dos e2e de los pagos en local, con la aprobación del responsable, dos veces: 54 pasan y 96 se omiten a propósito, en la tableta y el teléfono; `ng build` sin advertencias y `npm run lint:styles` sin errores.

## Después, el mismo día: el pie, lo que falta, el estado y el ícono

- **Revisión 70, «Pie de un formulario de página»** (merge `6a02be91`, `68509826` a `82511e4f`): el responsable pidió que «los botones de acciones que aparecen al último… deben verse como en factura». `mm-form-foot` lo dibuja; la factura y la nota lo toman sin cambiar, y la orden, la plantilla y el país lo adoptan. 314 pruebas unitarias.
- **Revisión 71, «Lo que falta en un formulario de página»** (merge `2aa8c4c5`, `7fdb937a` a `e062439f`): con su «Sí, hazlo», la factura, la nota, la orden, la plantilla y el país se comportan como Registrar pago: nada al abrir, la principal nunca retenida, `mm-error-summary` arriba; y con sus dos pedidos del mismo día, «al guardar se muestra que se están guardando los datos» y «si hay errores, focus en el error de la vista», «Guardando los datos…» con el indicador y el foco en el primer campo con error, no en el resumen. 440 pruebas unitarias; sus e2e se llevaron sin correrlas, porque no tenían la aprobación.
- **Revisión 72** (merge `d59b2f31`: `294f3052`, `8c016f01`): con una captura de Órdenes de compra, el estado de toda lista centrado bajo su encabezado, 31 columnas de 27 listas. 340 pruebas unitarias.
- **Revisión 73** (merge `de2d1c56`: `6e5df87f`, `671933e4`): con una captura de la ficha de una unidad, el ícono de un dato se queda a la altura de su etiqueta y su valor, aunque una celda vecina más alta estire la suya.

## Bien hecho

- **Decidida antes de construirla.** D-246 llegó al diccionario antes que el código, como D-244 y D-245 el día anterior.
- **La evidencia antes de la opinión, y la opinión del responsable después.** Cada recomendación de la revisión llevó su fuente, y donde el responsable eligió otra cosa —las listas, el saldo como método— quedó escrito qué cedía.
- **Ninguna regla cambia de lugar.** La pantalla nueva manda lo mismo que la anterior: el backend no cambia.
- **Preguntar antes de suponer.** Las tres preguntas cerraron lo que el flujo del responsable dejaba abierto: varias facturas, las órdenes y las pruebas.

## Hallazgos

### 1. Un asistente con «Continuar» para una tarea diaria (aprendizaje)

- **Error:** la primera propuesta ordenó el registro en pasos con «Continuar», un asistente. La evidencia lo reserva para tareas largas, raras o nuevas para quien las hace (NN/g; SAP Fiori, que lo recomienda para esas), y registrar un pago es una tarea diaria de Contabilidad. Se propuso sin contrastarlo con las mismas fuentes que la revisión citaba.
- **Corrección:** la revisión basada en evidencia y D-246: una página, en el orden del responsable, donde cada documento abre su sección, la revelación progresiva en lugar de los pasos.
- **Próxima vez:** antes de proponer cómo se recorre una pantalla, se pregunta qué tan seguido se usa y quién la usa: una tarea diaria va en una página, y los pasos se reservan para la que es rara o nueva.

### 2. El prototipo estiraba el documento (aprendizaje)

- **Error:** los controles ocultos a la vista del prototipo se posicionaban fuera del contenedor que se desplaza, así que el documento se alargaba más allá de su contenido y la página tenía un desplazamiento que no correspondía a nada.
- **Corrección:** los controles ocultos se posicionan dentro de su contenedor, y el prototipo se revisó después en un navegador sin cabeza, antes de publicarlo.
- **Próxima vez:** un control oculto a la vista se posiciona dentro de un contenedor propio, dentro del área que se desplaza; y un prototipo se abre en un navegador sin cabeza, midiendo el alto del documento y el desplazamiento de lado en cada ancho, antes de publicarlo.

### 3. Una factura por pago, preguntado antes de suponerlo (aprendizaje)

- **Error:** el orden del responsable —proveedor, factura, vehículos— lleva a pensar un pago como una sola factura. Suponerlo habría roto «Pagar N facturas» de Por pagar (`?facturas=`, D-245, revisión 67), que abre el formulario con varias, y la copia de un pago anulado que pagó varios documentos.
- **Corrección:** se preguntó antes de diseñarlo, y el responsable eligió varias facturas en un pago, con «Agregar otra factura» (D-246).
- **Próxima vez:** el rediseño de una pantalla empieza por listar lo que entra a ella —aquí `?factura=`, `?facturas=`, `?proveedor=&empresa=` y `?copiaDe=`— y pregunta antes de dejar caer alguna entrada.

### 4. Los datos de la e2e copiaban la cuenta de otro pago (aprendizaje)

- **Error:** el pago en efectivo de los datos de la lista clonaba un pago con cuenta y la conservaba, un defecto de los datos desde la revisión 67 que nadie vio porque, por la regla del 09/11/2026, esa e2e se llevaba a las pantallas nuevas sin correrla.
- **Corrección:** `da44a3d5`, al correr las e2e de los pagos con la aprobación del responsable.
- **Próxima vez:** una e2e llevada sin correr queda anotada como deuda, y corre en cuanto su pantalla vuelve a cambiar.

### 5. Un pie dibujado formulario por formulario (aprendizaje)

- **Error:** la factura y la nota cerraban con la barra de las revisiones 47 y 51, y la orden, la plantilla y el país con una fila suelta de botones, Cancelar como texto: la misma parte, hecha dos veces de dos maneras.
- **Corrección:** la revisión 70: `mm-form-foot` en `shared`, en los cinco formularios y en el pago (merge `6a02be91`).
- **Próxima vez:** una parte que se repite en dos formularios va a `shared` antes de hacerse en el tercero.

### 6. El foco de la factura al registrar nunca había funcionado (aprendizaje)

- **Error:** `focusFirstInvalid` buscaba `.mat-mdc-form-field-invalid`, una clase que Angular Material ya no pone en sus campos, así que al pulsar con errores el foco se quedaba en el botón; ninguna prueba miraba adónde iba.
- **Corrección:** la revisión 71: `focusFirstError` lo busca por lo que el campo dice de sí —`aria-invalid`, `.mat-form-field-invalid`— y sus unitarias comprueban el elemento con el foco (merge `2aa8c4c5`).
- **Próxima vez:** un selector de una clase de una librería se prueba contra el componente dibujado, y una regla de foco lleva una prueba de `document.activeElement`.

### 7. El estado de una lista se quedó al inicio de su celda (aprendizaje)

- **Error:** la revisión 64 centró los encabezados y la 68 las fechas, pero una columna de insignias de estado dejó la insignia al inicio, al lado de su encabezado y no debajo.
- **Corrección:** la revisión 72, en 31 columnas de 27 listas (merge `d59b2f31`).
- **Próxima vez:** cuando cambia la regla de un encabezado, se revisan en el mismo cambio todos los tipos de celda que caen bajo él.

### 8. El ícono de un dato se hundía en su celda (aprendizaje)

- **Error:** `mmDataListItem` centraba el ícono en la altura de su celda, y una celda crece con la más alta de su fila, así que junto a una vecina más alta el ícono bajaba mientras el texto seguía arriba.
- **Corrección:** la revisión 73: el ícono, a la altura de la etiqueta y la primera línea del valor (merge `de2d1c56`).
- **Próxima vez:** lo que se alinea a una celda se mira con una vecina más alta que ella.

## Lo que queda abierto, con su disparo

Cada punto está en los pendientes del [orden de implementación](../orden-de-implementacion-millennium-mobility.md) o de [contextos/procurement.md](../contextos/procurement.md):

- **El agente de retención:** si Millennium es agente de retención del IGV lo confirma contabilidad, con la Consulta RUC; hasta entonces la retención sigue opcional, la pantalla la propone en el 3 % y se corrige a mano (diccionario §14, D-196).
- **«Por pagar» o «Pendiente»:** el formulario del pago dice «Por pagar», y las demás pantallas de Compras, «Pendiente». Si se alinean, lo decide el responsable.
- **Las entradas del resumen de errores:** en los cinco formularios de la revisión 71, el campo y su mensaje, «Proveedor: Este campo es obligatorio.»; en el pago, lo que hay que hacer. Cuál queda, lo decide el responsable.
- **Los botones del pie en el teléfono,** apilados o uno junto al otro: lo decide el responsable.
- **Las imágenes de referencia** de toda pantalla tocada ese día, de la revisión 69 a la 73, no se regeneraron, por la regla del responsable: se regeneran juntas.
- ~~**`GET /payables/summary`** sigue en el backend y ninguna pantalla lo lee.~~ Salió del backend el mismo día, en la limpieza de lo que ya no se usa (backend `ec619a1a`, en la rama `chore/remove-payables-summary`, todavía sin merge a `develop`).
- **Las e2e de los cinco formularios** de la revisión 71 no corrieron en local; la sonda en vivo de los pagos se llevó al formulario nuevo (`532180aa`) y tampoco corrió.
