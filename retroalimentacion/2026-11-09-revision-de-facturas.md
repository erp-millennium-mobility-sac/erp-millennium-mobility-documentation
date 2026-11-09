# Revisión · Las facturas vistas por el responsable el 09/11/2026: un botón principal y un menú en toda ficha, el pago de la factura sobre sus datos y su estado en la lista

**Fecha:** 09/11/2026 · **Lo revisado:** la lista y la ficha de las facturas de compra en `dev.erp.millennium.ky`, con los datos de demostración del mismo día · **Cierre:** diccionario v1.49, D-244 (documentación `077a5c1`); backend `edd12f9a` a `6230346b`, en `develop` con el merge `4d15256d`; frontend, la revisión 62 de `DESIGN.md` (`83aff699` a `e6b48de8`, merge `2fcbecbe`) y la 63 (`8fa29b6e` a `6cbf7243`, merge `cf862569`); sin migraciones · **Alcance:** lo que el responsable vio, lo que preguntó y decidió, lo que se construyó ese día y la regla de verificación que fijó.

El mismo día, `dev.erp.millennium.ky` se llenó con datos ficticios por la API: 3 sedes, 8 ubicaciones, 4 cuentas bancarias, 18 colaboradores, 3 fondos, las tasas de USD a PEN del 01/07 al 10/11/2026, 9 órdenes, 7 facturas con 45 unidades, 2 notas de crédito y 6 pagos, y las unidades llevadas a lo largo de su vida. Con esos datos el responsable abrió la lista y la ficha de las facturas, y pidió tres cosas: ordenar las acciones de la cabecera —hasta ocho botones en dos filas, con «Anular» junto a «Registrar pago»—, dar a los pagos una pantalla más apropiada y saber por qué la lista dice «Factura» en cada fila.

**Veredicto.** Ningún defecto del producto: tres pedidos de pantalla y una pregunta, que llevaron a una decisión del diccionario, D-244, y a dos revisiones de `DESIGN.md`, construidas e integradas en `develop` el mismo día.

---

## Las decisiones del responsable

- **Un botón principal y un menú agrupado, en toda ficha.** No los ocho botones en dos filas que pidió primero: a lo sumo dos en la cabecera, el principal relleno y uno secundario delineado, y el resto en el menú ⋮ «Más acciones de la …», agrupado, con lo que retira la ficha al final y en rojo. Reemplaza, para dónde van los botones, su decisión del 19/10/2026 de las revisiones 13 y 15 de `DESIGN.md`.
- **El resumen del pago sobre los datos de la factura, y solo la tabla de los pagos en su panel.**
- **El estado de pago en la lista, con el cambio del backend:** D-244, decidido en el diccionario v1.49 antes de construirlo (`077a5c1`): el estado se calcula y no se guarda, con el vocabulario del de sus unidades —`SIN_PAGO`, `PARCIAL`, `PAGADA`, `SALDADA` o `ANULADA_POR_NOTAS`—, lo ve quien lee la factura con `compras.ver`, y sus montos solo quien tiene `pagos.ver` en su empresa.
- **La respuesta a la pregunta:** hay tres tipos de comprobante, Factura, Boleta y Otro (D-159), y solo la factura da crédito fiscal. Como la serie «F…» ya lo dice, la lista muestra solo el número y marca la excepción, «Boleta» u «Otro».

## Lo construido

- **El backend** (`develop`, merge `4d15256d`): `InvoicePaymentStatus` e `InvoicePaymentStatusPolicy` (`edd12f9a`); `bankAccountName` en cada aplicación de la posición de pago (`e59fb619`); `paymentStatus`, `appliedTotal` y `owed` en la lista y la ficha, y las `allowedActions` de cada fila con la regla de la ficha (`b250636d`); la columna «Estado de pago» de la exportación, sin montos, con la versión del archivo que sigue también las notas y los pagos (`8fc04028`); el filtro `paymentStatus` en la lista y la exportación, con la página cortada en memoria (`eb80ced8`); y la prueba de las sentencias de una página (`6230346b`). [contextos/procurement.md](../contextos/procurement.md), PRC-R59 y PRC-R167 a PRC-R170.
- **El frontend, revisión 62** (merge `2fcbecbe`, 14 commits): el componente compartido `mm-record-actions` (`shared/presentation/components/record-actions`) y la clase global `.mm-menu-item-destructive`, en las fichas de la factura, del pago —donde «Registrar otro pago» pasa a ser la acción principal—, de la nota de crédito, del proveedor, de la unidad y de la empresa —cuyo Desactivar pasa al menú, en rojo—, y en el pie de la orden: Cancelar, Enviar o Cumplir, Guardar y el menú con «Descargar PDF» y «Anular»; «Descargar PDF» sigue como botón cuando no se ofrece ni Enviar ni Cumplir. Un elemento deshabilitado del menú dice por qué debajo de su nombre.
- **El frontend, revisión 63** (merge `cf862569`): el resumen del pago sobre los datos de la factura —la insignia, Total, Notas de crédito, Pagado y Saldo o «Pagado de más», una barra quieta «N % pagado» y la línea de las iniciales de un dealer bajo el saldo; sin `pagos.ver`, solo Total y la insignia; ninguno en una factura anulada—; el panel de datos sin «Notas de crédito» ni «Total después de notas»; «Pagos», solo su tabla, con «Cuenta de origen», y «Equivalente» y «Diferencia de cambio» solo cuando un pago está en otra moneda, y vacía, «Todavía no tiene pagos.» con «Registrar pago»; los montos de la factura, de sus paneles y de la nota con el código ISO, «USD 7,734.00», y las tasas con sus decimales significativos, al menos tres; y la lista con solo el número y la insignia «Boleta» u «Otro», la orden como enlace, la columna «Pago», los filtros Tipo (`.mm-filter--short`) y Pago, los números y los montos en una línea, y el menú de la fila con «Ver detalles», «Registrar pago», «Registrar nota de crédito» y «Descargar comprobante», según sus `allowedActions`.
- **Verificado:** la suite completa del backend, antes del merge, sobre una base local recién creada con las dos cuentas: 3 960 pruebas, 0 fallos, 0 errores y 2 omitidas. En `dev`, después del despliegue, las siete facturas de demostración leen `PAGADA` (3), `PARCIAL` (2) y `SIN_PAGO` (2), el filtro `PARCIAL` devuelve esas dos, y cada pago nombra su cuenta.

## Bien hecho

- **La decisión antes que el código.** D-244 entró al diccionario v1.49 antes de abrir la rama, con quién ve los montos, el filtro en memoria y su límite —unos miles de facturas por empresa—, como D-238 a D-243 en la auditoría del mismo día.
- **El estado se deriva de lo que ya existía.** `InvoicePaymentStatusPolicy` lee lo que se debe de `AmountOwedPolicy` y lo que se puede pagar de un dealer, sin guardar nada ni escribir dos veces la cobertura de las unidades; y usa las palabras que la persona ya lee en las líneas.
- **Una página cuesta lo mismo con cualquier número de filas.** Las filas leen una sola vez lo que su pago y sus acciones necesitan —`PurchaseInvoiceDependents.invoicesWithAny(Set<Long>)` y `ExternalOrganizationService.fetchCompanyTimeZones`, en lote—, y `PurchaseInvoiceListStatementsTests` lo fija con las estadísticas de Hibernate: 17 sentencias una página de 2, 20 u 80 filas, y 20 una filtrada.
- **La exportación no filtra montos.** Exportar solo pide `compras.ver`, así que «Estado de pago» va sin montos, como D-212 deja ver el estado de cada línea.
- **Una pieza compartida en lugar de seis fichas a mano.** `mm-record-actions` decide el lugar y el grupo de cada acción, y `.mm-menu-item-destructive` reemplaza las tres copias que tenían el administrador de datos, Estados y motivos y la página de la habilitación.
- **Comprobado donde el responsable lo vio.** Después del despliegue, los estados de las siete facturas de demostración y el filtro se leyeron en `dev`, no solo en las pruebas.

## Hallazgos

### 1. La ficha ofrecía cada acción como un botón (aprendizaje)

- **Error:** la regla de la revisión 13 (decisión del responsable del 19/10/2026) ponía en la cabecera de los datos, como botones con etiqueta, todo lo que se hace con la ficha. Con las notas, los pagos, el ingreso y las descargas, la factura llegó a seis botones delineados, que en el teléfono se apilaban en una columna y apretaban el título a su lado (`DESIGN.md`, revisión 62). El responsable lo vio con datos de demostración y pidió ordenarlos.
- **Corrección:** su decisión, un botón principal y un menú agrupado en toda ficha: revisión 62, frontend `83aff699` a `e6b48de8`, merge `2fcbecbe`. Las preguntas, los diálogos, el foco después de una anulación y los `aria-describedby` de cada ficha no cambiaron.
- **Próxima vez:** una ficha nueva declara sus acciones como `RecordAction`, con su lugar —principal, secundaria o menú— y su grupo; una acción nueva entra al grupo del menú que le toca, no como un botón más en la cabecera.

### 2. La factura no decía cómo estaba pagada (aprendizaje)

- **Error:** el estado de pago existía por línea (D-212), no por factura. La lista no lo decía, y la ficha lo explicaba solo dentro del panel «Pagos», que abría con la resta de lo que se debe y seguía con la tabla de las aplicaciones (revisión 37, ítem 6). El responsable pidió a los pagos una pantalla más apropiada.
- **Corrección:** D-244 y la revisión 63: el estado de la factura en su lista, su ficha y su exportación, con su filtro (backend `edd12f9a`, `b250636d`, `8fc04028`, `eb80ced8`); el resumen arriba, y «Pagos» con un solo trabajo, la tabla, que ahora nombra la cuenta de cada pago (`e59fb619`; frontend `8fa29b6e`, `3dfca05d`, `0c59fd66`).
- **Próxima vez:** lo que resume un documento va antes de sus datos, y cada panel tiene un solo trabajo. Un estado que la persona va a buscar en una lista se decide en el diccionario, con quién lo ve y cómo se filtra, antes de dibujarlo.

### 3. «Factura» en cada fila (aprendizaje)

- **Error:** la columna Comprobante leía el tipo y el número, así que la lista repetía «Factura» en casi todas sus filas, y el responsable preguntó por qué. La respuesta estaba en el diccionario —tres tipos, D-159, y solo la factura da crédito fiscal— y en el número: la serie «F…» ya lo dice. Además, la revisión 44 había quitado Tipo de los filtros de Facturas, porque la factura es el caso casi universal.
- **Corrección:** revisión 63 (frontend `0c59fd66`, `5700020a`): solo el número, con la insignia «Boleta» u «Otro» cuando no es una factura, y Tipo de vuelta como un filtro corto, `.mm-filter--short`, con Pago a su lado.
- **Próxima vez:** una columna no repite lo que casi siempre dice igual: marca la excepción.

### 4. Los montos con su símbolo (aprendizaje)

- **Error:** la ficha de la factura, sus paneles y la ficha de la nota de crédito escribían cada monto con el símbolo de su moneda, que cambia de una a otra («US$», «S/», «€»), mientras la lista y la ficha del pago usaban su código; y las tasas, con seis decimales.
- **Corrección:** revisión 63 (frontend `3dfca05d`): el código ISO, «USD 7,734.00», y las tasas con sus decimales significativos, al menos tres («3.423», «3.7521», «1.000»). Los formularios conservan el símbolo en sus campos. Quedan dos rincones, abajo.
- **Próxima vez:** un monto que se lee en una ficha o una lista lleva el código de su moneda; el símbolo, solo el campo de un formulario.

### 5. Las pruebas se escalan al cambio (aprendizaje)

- **Error:** hasta ese día, cada verificación corría en local las suites completas, también para un cambio que solo se ve: la e2e completa del cierre de la auditoría, por ejemplo, 2 157 pruebas, con 186 que vencieron su tiempo por la concurrencia y pasaron al repetirlas ([orden de implementación](../orden-de-implementacion-millennium-mobility.md), «Cierre de la auditoría del 09/11/2026»). GitHub Actions ya corre la compilación, el lint, las unitarias, la e2e completa de Playwright y la suite completa del backend en cada push a `develop`.
- **Corrección:** la regla del responsable del 09/11/2026, también guardada como memoria del agente: un cambio que solo se ve corre sus propias pruebas unitarias y e2e y sus imágenes de referencia, y nunca las suites completas en local. El backend de D-244, que no era solo de pantalla, corrió su suite completa antes del merge: 3 960 pruebas, 0 fallos, 0 errores y 2 omitidas.
- **Próxima vez:** antes de verificar, se mide el cambio: lo que solo se ve, sus especificaciones y sus imágenes; lo que cambia el contrato o los datos, la suite de su repositorio; y lo demás lo dice GitHub Actions después del push.

## Lo que queda abierto, con su disparo

Cada punto está en los pendientes transversales del [orden de implementación](../orden-de-implementacion-millennium-mobility.md):

- **Lo que dejó la revisión 63**, sin fecha: en el catálogo, «Desgloses» escribe con su símbolo los de la unidad y del proveedor, mientras los de la factura usan el código; la columna «Monto en USD» de las líneas de la nota de crédito sigue sin moneda en sus montos; y la lista de facturas no guarda sus filtros en la dirección.
- **Las cuentas de acceso con roles para los datos de demostración**: crearlas lo bloqueó el clasificador de seguridad del agente, y las crea el responsable.
- **SendGrid.** La integración de la auditoría (`integration/audit-2026-11-09`, D-238 a D-243) entró en `develop` ese día y se publicó (backend `f4e025f7`, frontend `0f797b69`), después de que el responsable fijara en App Platform `APP_PUBLIC_URL`, `NOTIFICATIONS_EMAIL_PROVIDER=capturing` y `NOTIFICATIONS_EMAIL_FROM`; `dev` desplegó V122 a V126. Con `capturing` ningún correo sale: el enlace para restablecer la contraseña no llega, y un restablecimiento del administrador deja a la persona fuera, hasta que exista SendGrid (D-242, D-243).
