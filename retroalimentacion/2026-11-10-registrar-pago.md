# Revisión · Registrar pago visto otra vez por el responsable el 10/11/2026: una página en su orden, los vehículos de la factura, con qué se paga y la constancia como en las facturas

**Fecha:** 10/11/2026 · **Lo revisado:** «Registrar pago» (`/procurement/supplier-payments/new`), el formulario de dos pasos y un resumen que D-245 y la revisión 66 de `DESIGN.md` dejaron el día anterior · **Cierre:** diccionario v1.51, D-246 (documentación `b27c002`), decidida antes de construirla; la [especificación de pagos](../especificacion-pagos-a-proveedores-2026-11-03.md), versión 10 (`d198124`); [contextos/procurement.md](../contextos/procurement.md) (`ca57d0c`), [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (`334945e`) y el [orden de implementación](../orden-de-implementacion-millennium-mobility.md) (`c947def`); frontend, la revisión 69 de `DESIGN.md`, en construcción en la rama `feature/payment-form-one-page`; sin cambio en el backend ni migraciones · **Alcance:** lo que el responsable pidió a lo largo del día, la revisión basada en evidencia y los prototipos, sus respuestas y sus decisiones, lo que se construye y lo que queda.

El día anterior, en la [revisión de los pagos](2026-11-09-revision-de-pagos.md), el responsable aprobó un formulario en dos pasos, «① Qué se paga» y «② Cómo se paga», con un resumen al costado (D-245). Volvió a él: el formulario todavía no lo convencía. Pidió un análisis, vio una primera propuesta en pasos, dio su propio orden del pago y, después de una revisión basada en evidencia y tres prototipos, pidió construirlo de punta a punta.

**Veredicto.** Ningún defecto del producto: el formulario cumplía cada regla, pero no seguía el orden en que la persona piensa el pago. Una decisión del diccionario, D-246, tomada antes de abrir la construcción, sobre las mismas reglas y la misma API; una revisión de `DESIGN.md`, la 69, en construcción.

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

En construcción en la rama `feature/payment-form-one-page` del frontend, con la revisión 69 de `DESIGN.md`. Se completa al integrarla:

- **El frontend, revisión 69:** _por completar con sus commits y su merge en `develop`._
- **Verificado:** _por completar_: `ng build`, las unitarias de lo que se toca, `npm run lint:styles` y las e2e de los pagos en local, por la decisión del responsable de ese día.

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

## Lo que queda abierto, con su disparo

Cada punto está en los pendientes del [orden de implementación](../orden-de-implementacion-millennium-mobility.md) o de [contextos/procurement.md](../contextos/procurement.md):

- **Lo construido:** los commits, las pruebas y lo que fije la construcción se anotan aquí, en el diccionario, en la especificación y en el contexto al integrar la rama `feature/payment-form-one-page` en `develop`.
- **El agente de retención:** si Millennium es agente de retención del IGV lo confirma contabilidad, con la Consulta RUC; hasta entonces la retención sigue opcional y escrita (diccionario §14, D-196).
- **Las palabras fuera del formulario:** D-246 decide solo «Registrar pago». La ficha del pago todavía dice «Cambiar voucher» y «Descargar voucher», y las demás pantallas de Compras dicen «Pendiente» donde el formulario dirá «Por pagar». Si se alinean, lo decide el responsable.
- **La sonda en vivo de los pagos** (`e2e-live/supplier-payments.live.spec.ts`), que nunca corrió, se lleva al formulario de la revisión 69 antes de correrla.
- **Las imágenes de referencia** de `supplier-payment-form`, `-voucher`, `-cash` y `-balance`, que no se regeneraron con la revisión 66, cambian otra vez con la 69: se regeneran juntas.
