# Contexto `procurement` — ERP Millennium Mobility

**Actualizado:** 27/10/2026, tercera revisión (el PDF desde la plantilla HTML con el formato de la empresa; el año del modelo en cada línea de vehículo y la descripción que imprime el PDF en la API; la pantalla de la orden rediseñada) · 27/10/2026, segunda revisión (el PDF imprime todas las líneas en las páginas que hagan falta, la condición de pago y el tipo de documento del emisor; los montos redondeados a los decimales de la moneda en la API, el PDF y la hoja; el teléfono de Atención copiado sea quien sea quien guarda; un archivo de la plantilla mueve su `updatedAt`; la marca y el tipo retirados que la plantilla ya tiene se ofrecen en su formulario) · 27/10/2026 (los hallazgos de órdenes y plantillas: el descuento y una línea nueva en el mismo guardado; ninguna orden bajo una empresa, una marca o un tipo inactivos, y `issueOrder` en la plantilla; el borrador conserva lo retirado que ya tiene; la empresa y la moneda del pedido; el orden por número; la reactivación de una plantilla con su marca activa; el frontend lee la orden como la responde la API) · 26/10/2026 (cierre de la auditoría del 25/10/2026, diccionario v1.31, D-130, D-133 y D-134: los ocho permisos de plantillas, tipos y órdenes, V79 y V80; el estado de la plantilla en `PATCH /{id}/status`; sin ninguna ruta `DELETE`; el emisor fuera de los pedidos; las columnas de V73 borradas en V81; el emisor copiado de la empresa y no editable; la Atención copiada del colaborador; el PDF enviado como una sola copia fiel con el teléfono de Atención; los archivos de la plantilla bajo una llave de su contenido y `removedDocuments` en el `PATCH`; las lecturas solo con `compras.ver`, escribir exige leer; un rol de sede no escribe compras, y lo que es de todas las marcas lo escribe la empresa entera; los códigos de compras de convenciones §7.2; la exportación de proveedores) · 23/10/2026 (proveedores, tipos de orden, plantillas con sus archivos y órdenes de compra, en `develop`; facturas, notas de crédito y pagos siguen sin código) · **Estado:** borrador para validar; el paso 7 sigue abierto · **Fuente:** diccionario de datos v1.28 (§9.1, §9.1.1, §9.2, §2.7 `kept_files`, D-35, D-36, D-37, D-102, D-105, D-125 y D-132), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md) §2.2 y §3.4, [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (P4) y [orden de implementación](../orden-de-implementacion-millennium-mobility.md) (paso 7). Si este documento y el diccionario no coinciden, manda el diccionario.

**Cómo leerlo.** Las secciones 1 a 3 están escritas para quien conoce el negocio. La 4 y la 6 nombran el código. Lo que todavía no tiene pantalla —facturas, notas de crédito y pagos— se nombra solo para decir que este paso no lo cierra.

---

## 1. Qué es y qué no es

`procurement` responde quién le vende a la empresa, con qué formato se emite una orden de compra y qué orden quedó emitida. Es el paso 7 del orden de implementación, a medias: proveedores, tipos, plantillas y órdenes están construidos. Las facturas, las notas de crédito y los pagos del paso 8 siguen en el diccionario y no tienen ruta.

**Las tablas que ya posee**

| Tabla | Qué guarda | Diccionario |
|---|---|---|
| `suppliers` | El proveedor de una empresa, con su documento y su domicilio | §9.1 |
| `supplier_contacts` | Las personas de ese proveedor | §9.1.1 |
| `purchase_order_types` | Vehículos, taller, otros: un catálogo de la empresa | §9.2 |
| `purchase_order_templates` | El formato con que se imprime y se numera una orden | §9.2, D-35, D-132 |
| `purchase_order_template_document_types` | Los códigos de archivo de la plantilla. Hoy `LOGO` y `SIGNATURE`. No es de una empresa | §9.2 |
| `purchase_order_template_documents` | Un archivo de una plantilla, a lo sumo uno por tipo | §9.2 |
| `purchase_orders` | La orden, con su estado y, si ya se envió, la llave de su PDF | §9.2, D-36 |
| `purchase_order_items` | Las líneas del borrador: modelo y cantidad, o un concepto | §9.2, D-37, D-132 |

Las columnas están en el diccionario y no se repiten aquí.

**Lo que el diccionario le asigna y todavía no tiene código:** `purchase_invoices`, `purchase_invoice_items`, `credit_notes`, `credit_note_items` (§9.3 a §9.6) y, en el paso 8, `supplier_payments`, `payment_applications` y `payment_allocations`.

**Lo que parece suyo y no lo es**

| Parece de `procurement` | Es de | Por qué |
|---|---|---|
| Los bytes del logo, de la firma y del PDF enviado | `documents`, en `kept_files` (§2.7, D-125) | Aquí se guarda la llave (`file_path`, `pdf_path`). El contexto abre el archivo por `KeptFileStore` |
| El PDF del borrador, que cambia si cambia la plantilla | `documents`, en `generated_documents` | Se pide por `DocumentService` y vence. Al enviar, el PDF que se mandó pasa a `kept_files` y ya no cambia |
| El domicilio del emisor y el del proveedor | `location`, `addresses` (§4.6) | El proveedor lo crea, lo corrige o lo muda por la fachada; la plantilla copia el de la empresa al crearse y no lo cambia (D-134). No hay ruta de dirección. La base acota las direcciones por su país (§2.5, V78) |
| La empresa, la marca comercial y el colaborador de Atención | `organization` | La plantilla apunta a ellos. El emisor sale de la empresa y la Atención —nombre, puesto y teléfono— del colaborador, copiados en el servidor (D-134) |
| El tipo de documento del proveedor | `location` | `procurement` lo verifica por la fachada: del país de la empresa, activo, que aplica a empresas y con su longitud (D-133). El del emisor es una copia del de la empresa y no se verifica de nuevo |
| El modelo de una línea de vehículos | `vehiclecatalog` | La línea guarda el identificador; el PDF imprime la marca y el nombre que devuelve la fachada |
| La unidad y su VIN | `fleet` | Ninguna orden de hoy crea una unidad. La factura, cuando se construya, preguntará a `fleet` (mapa §6.1). Este contexto todavía no declara esa dependencia |
| Los tipos de documento del vehículo | `fleet` (§11.2) | El catálogo `purchase_order_template_document_types` es solo de la plantilla. Un SOAT no cabe ahí |

**Palabras con un significado preciso aquí**

| Palabra | Significa | No confundir con |
|---|---|---|
| Tipo de orden | Vehículos, taller u otro, de una empresa | El tipo de vehículo (auto, moto) ni el tipo de documento del vehículo |
| Plantilla | Quién emite, qué serie numera, qué texto de «Obs:» y qué imágenes se imprimen | La orden, que elige proveedor y líneas |
| Serie | El prefijo de numeración, `OC-LBT`. No se repite en la empresa y queda fija cuando la plantilla ya tiene órdenes | El número de la orden, que sale de la serie y del año |
| Observación | El texto de «Obs:» al pie de la tabla. Vive en la plantilla | `special_instructions`, las instrucciones de facturación, también de la plantilla |
| Documento de la plantilla | Una fila de logo, firma u otro tipo, con la llave del archivo | Un documento del vehículo, y un documento generado que vence |
| Quitar el archivo | Vaciar la llave de esa fila | Borrar la fila o los bytes |
| Descuento | Una columna que se enciende en cada orden. La tasa de la línea va de 0 a 1 | Un descuento de la plantilla: no existe |

---

## 2. Ciclo de vida de cada agregado

Nada de lo construido se borra, y no hay rutas `DELETE`: quitar un archivo de la plantilla es `removedDocuments` en su `PATCH`, que vacía la llave y deja la fila (convenciones §3.1 y §3.4).

### Proveedor

| Momento | Qué pasa |
|---|---|
| Alta | Configuración › Proveedores. Documento, razón social y domicilio en el mismo diálogo (`POST /suppliers`) |
| Qué cambia | La ficha completa, con `PUT /suppliers/{id}`. El domicilio se corrige o se muda ahí (`address` + `addressChange`). Los contactos se agregan, se corrigen o se retiran en esa misma ficha |
| Qué queda fijo | La empresa y el documento |
| Retiro | `PATCH /suppliers/{id}/status`. Un proveedor inactivo no se elige en una orden nueva |
| Nunca | Se borra. `/suppliers/{id}/balance` no existe: el saldo se calculará con los pagos |

### Tipo de orden

| Momento | Qué pasa |
|---|---|
| Alta | Dentro de Plantillas de OC, con `compras.crear_plantillas` |
| Qué cambia | El nombre, con `PUT /purchase-order-types/{id}` y `compras.editar_plantillas`; en el mismo pedido, el código (`code`, opcional), mientras ninguna plantilla usa el tipo |
| Qué queda fijo | El código, en cuanto una plantilla lo usa: otro código responde `422 PURCHASEORDERTYPE_CODE_LOCKED`, y `codeLocked` lo anuncia |
| Retiro | `PATCH /purchase-order-types/{id}/status`, con `compras.desactivar_plantillas` |
| Nunca | Se borra. Agregar un tipo no cambia cómo funciona la orden |

### Plantilla

| Momento | Qué pasa |
|---|---|
| Alta | Un solo `POST /purchase-order-templates`, multipart, con `compras.crear_plantillas`. La parte `template` es la ficha, sin emisor. `logo` y `signature` pueden ir, las dos, una o ninguna |
| Qué cambia | `PATCH /purchase-order-templates/{id}`, con `compras.editar_plantillas`. Un campo que el JSON no trae se queda; uno que trae en `null` se vacía si admite vacío. `updatedAt` tiene que ser el de la fila (`422 PURCHASEORDERTEMPLATE_OUTDATED`). El emisor no es del pedido: si llega, se ignora. `isActive` tampoco: si llega, `400 VALIDATION_ERROR` y no cambia nada |
| Estado | Solo por `PATCH /{id}/status` con `{ isActive, updatedAt }`, con `compras.desactivar_plantillas`. Un `updatedAt` viejo responde `422 PURCHASEORDERTEMPLATE_OUTDATED`; un cuerpo sin uno de los dos, `400`. Reactivarla es volver a colgarla (D-94): solo con su empresa activa (`COMPANY_INACTIVE`) y, si es de una marca, con esa marca activa (`COMMERCIALBRAND_INACTIVE`) |
| Qué queda fijo | La serie, la marca comercial y el tipo, cuando la plantilla ya tiene órdenes |
| Archivos | Van en el mismo `PATCH`: una parte `logo` o `signature` reemplaza el archivo, y `removedDocuments: ["LOGO"]` lo quita, todo en una transacción. `PUT /{id}/documents/{type}` reemplaza uno solo, sin `updatedAt`; `removedDocuments` en el `PATCH` es la única forma de vaciar la llave. Guardar o quitar un archivo mueve el `updatedAt` de la plantilla: un `PATCH` o un cambio de estado abiertos antes responden `PURCHASEORDERTEMPLATE_OUTDATED` (PRC-R19). `GET /{id}/documents/{documentId}/file` devuelve los bytes. Cada archivo se guarda bajo una llave de su contenido, y reemplazarlo escribe una llave nueva (§2.7, D-134). La respuesta de la plantilla lista solo los documentos que tienen archivo |
| Retiro | `isActive: false` en `PATCH /{id}/status`. Las órdenes ya emitidas la siguen usando |
| Nunca | `PUT` de la ficha entera, `DELETE /{id}/documents/{documentId}`, `POST /{id}/signature` ni `POST /{id}/signature-removal`. Esas rutas ya no existen |

El proveedor no se elige en la plantilla. Se elige al emitir la orden. La columna Atención es un colaborador de la empresa. El emisor no se edita: es la copia de la empresa que hizo el alta (D-134).

### Orden

| Momento | Qué pasa |
|---|---|
| Alta | Logística › Órdenes de compra, con `compras.crear_ordenes`. El número se asigna al crear el borrador y no se reutiliza (D-36). Ahí se elige la plantilla, el proveedor y si esa orden lleva columna de descuento. Se emite solo con una plantilla activa cuya empresa, marca —si la tiene— y tipo están activos (D-84, §9.2); la pantalla ofrece solo las plantillas con `issueOrder` en su `allowedActions`. La empresa del pedido tiene que ser la de la plantilla, y cada precio viaja en la moneda de la orden |
| Qué cambia | El borrador, con `PUT /purchase-orders/{id}` y `compras.editar_ordenes`. Una línea que se quita queda inactiva y no entra al PDF ni a los totales. El borrador conserva el proveedor, el contacto, la moneda y el modelo de cada línea que ya tenía aunque se hayan desactivado después; uno nuevo se revisa como en el alta (D-84, D-108). Encender o apagar el descuento y agregar una línea van en el mismo guardado |
| Envío | `POST /{id}/sending`, con `compras.enviar_ordenes`. El PDF de ese momento queda en `kept_files` (`purchase-orders/{publicId}/sent.pdf`) y ya no sigue a la plantilla. Es una sola copia fiel con el teléfono de Atención, la que recibió el proveedor, para todo el que lee la orden, pulse quien pulse «Enviar» (D-105, D-134) |
| Cumplida | `POST /{id}/fulfillment`, con `compras.cumplir_ordenes`, solo desde enviada |
| Anulada | `POST /{id}/annulment`, con `compras.anular_ordenes`. No sale de cumplida |
| PDF | `GET /{id}/pdf`. En borrador es el documento generado, que refleja la plantilla de ahora. Enviada, cumplida o anulada entrega el PDF congelado. La hoja de la pantalla muestra la plantilla y el proveedor de ahora: en una orden con PDF enviado, un aviso dice que el documento que recibió el proveedor es ese PDF, con «Descargar PDF» |
| Nunca | Se edita después de enviada: se anula y se emite otra. Se borra |

---

## 3. Flujos funcionales

Quien lee proveedores, tipos, plantillas y órdenes entra con `compras.ver`, y solo con él: desde el 26/10/2026 el servicio tampoco acepta un permiso de escritura en su lugar, y las tres entradas del menú de compras piden `compras.ver` (diccionario §5.6, D-134). Escribir exige leer: una escritura sobre lo que quien llama no lee con `compras.ver` responde `404`, y la respuesta de una escritura es lo que el comando guardó. Crear, editar o desactivar un proveedor pide el permiso de esa operación. Plantillas y tipos piden `compras.crear_plantillas`, `compras.editar_plantillas` o `compras.desactivar_plantillas`, según la operación. Crear, editar el borrador, enviar, cumplir y anular una orden piden `compras.crear_ordenes`, `compras.editar_ordenes`, `compras.enviar_ordenes`, `compras.cumplir_ordenes` y `compras.anular_ordenes` (V79, D-130). Los dos `gestionar_*` a los que reemplazan ya no abren ninguna ruta, y V80 los desmarcó en todos los roles. Ningún permiso de compras actúa por sede: un proveedor, un tipo y una plantilla general los escribe solo una asignación de toda la empresa y sin marca; una plantilla con marca, y las órdenes que salen de ella, también una de esa marca (PRC-R11, PRC-R12).

### F1 · Registrar un proveedor

1. Abre Configuración › Proveedores (`/procurement/suppliers`).
2. Completa la ficha, el domicilio y los contactos, y guarda (`POST /suppliers`).
3. Corregir o mudar el domicilio, o mantener los contactos, es otro guardado de la ficha (`PUT /suppliers/{id}`).
4. Activar o desactivar es la acción de la fila (`PATCH /suppliers/{id}/status`).

### F2 · Dar de alta una plantilla, con o sin imágenes

1. Abre Configuración › Plantillas de OC (`/procurement/purchase-order-templates`) y entra al formulario de página.
2. Elige empresa, línea de negocio si la plantilla no es general, y tipo. El tipo se crea en esa misma pantalla cuando falta.
3. La hoja muestra el emisor de la empresa, sin editarlo: el servidor lo copia al guardar. La Atención se elige entre los colaboradores de la empresa y se copia del elegido. La observación y las instrucciones son de esta plantilla.
4. Puede elegir un logo, una firma, los dos o ninguno. Al guardar, el formulario manda la ficha y los archivos elegidos en un solo `POST`.
5. Si un archivo no es PNG, JPEG, WebP o PDF —leído de sus bytes—, pasa de 5 MB o es una imagen de más de 4096 píxeles por lado o 16 megapíxeles, no se crea la plantilla (`422`). Por encima de 10 MB el servidor corta el pedido (`413`).

### F3 · Corregir la plantilla o su estado

1. Guardar el formulario manda un `PATCH` multipart con la ficha y, si se eligió una imagen nueva, esa parte. Una imagen que no se tocó no viaja y se conserva.
2. Quitar el logo o la firma viaja en el mismo `PATCH`, con `removedDocuments`, junto con los campos y las imágenes nuevas. La fila y los bytes se quedan; la hoja deja de mostrarlo.
3. Activar o desactivar desde la lista manda `{ isActive, updatedAt }` a `PATCH /{id}/status` y no reescribe la ficha.
4. Reemplazar un solo tipo, sin abrir el resto de la ficha, es `PUT /{id}/documents/LOGO` o `…/SIGNATURE`.

### F4 · Emitir una orden

1. Abre Logística › Órdenes de compra (`/procurement/purchase-orders`).
2. Elige la plantilla, la primera decisión de la pantalla; si la empresa tiene una sola con la que se puede emitir, ya viene elegida. Un encabezado compacto muestra el número que tendrá, la fecha y el emisor. El logo y la firma no se muestran en la pantalla: salen en el PDF. Lo demás que imprime la plantilla —Atención, observación, instrucciones y soporte— está en un bloque plegable, cerrado.
3. Elige el proveedor y, si quiere, un contacto de ese proveedor, la moneda, la condición de pago y el concepto del pedido.
4. Carga las líneas con «Agregar vehículo» —marca, modelo y año, y la pantalla muestra cómo se imprime— o «Agregar línea libre», un texto que se imprime tal como se escribe. Decide si esta orden lleva columna de descuento.
5. El borrador se guarda con `PUT`. Enviar, cumplir y anular son los tres `POST` de hecho. Una orden en solo lectura muestra cada línea con la descripción que imprime el PDF.
6. El PDF del borrador sigue a la plantilla. El de la orden enviada, no.

---

## 4. Reglas de negocio

| Id | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| PRC-R01 | El proveedor y su contacto se eligen al emitir la orden. La plantilla no los guarda: el código dejó de leerlos y escribirlos, y V81 quitó `purchase_order_templates.supplier_id` y `supplier_contact_id` con sus llaves (las de V73), después de comprobar que ninguna fila las usaba (D-133). Atención es un colaborador de la empresa. El emisor y su domicilio se copian de la empresa al crear la plantilla y no se editan: el alta y el `PATCH` ya no los tienen, un cliente que los manda no falla porque se ignoran, y la respuesta devuelve la copia (D-134) | §9.2, D-133, D-134 | Servicio: `PurchaseOrderTemplateCommandServiceImpl`, con `CompanyIssuer`; el domicilio se copia tal cual con `copyAddress` de `location`, de modo que una división desactivada después no impide crear la plantilla. Base: V81. Frontend: el formulario no ofrece proveedor ni edita el emisor | `PurchaseOrderTemplateAttentionPolicyTests`; `PurchaseOrderTemplatesIntegrationTests` (`createWithoutIssuerCopiesTheCompany`, `theFiscalAddressIsCopiedAsItIsEvenWithItsDivisionInactive`, `createWithADifferentIssuerIgnoresIt`, `patchWithMalformedIssuerFieldsIsIgnored`); `AddressCommandServiceImplTests.aCopyIsANewRowWithTheSamePartsEvenWithAnInactiveDivisionOrWithoutAPostalCodeNowRequired`; `PurchaseOrderTemplateSupplierDropMigrationTests` | ✅ |
| PRC-R02 | La Atención se copia solo del colaborador: nombre, puesto y teléfono; el correo queda vacío (D-134). El teléfono se copia tal como lo tiene el colaborador, sea quien sea quien guarda la plantilla: la fachada de `organization` lo entrega para copiarlo, y antes lo leía por la vista de quien llamaba, así que un editor sin datos personales guardaba la Atención sin teléfono y la orden enviada salía sin él. Sin `configuracion.ver_datos_personales_colaboradores`, la ficha y el PDF del borrador no muestran el teléfono de Atención; el PDF de la orden enviada sí, para todos: es la copia fiel que recibió el proveedor (excepción a D-105) | §9.2, D-105, D-134 | `PurchaseOrderTemplateAttentions`, con `OrganizationContextFacade.fetchEmployeeAttentionByPublicId`; `PurchaseOrderTemplateViewFactory`, `PurchaseOrderQueryServiceImpl.renderDraftPdf` y `PurchaseOrderCommandServiceImpl` (envío) | `PurchaseOrderTemplateAttentionPolicyTests`; `PurchaseOrdersIntegrationTests` (`aReaderWithoutPersonalDataDoesNotGetTheDraftSheetGeneratedForOneWithIt`, `aSentOrderPrintsThePhoneWhoeverSentIt`); `PurchaseOrderTemplatesIntegrationTests` (`theAttentionIsCopiedFromTheEmployee`, `aWriterWithoutThePersonalDataCopiesThePhoneOfTheAttentionWithoutSeeingIt`) | ✅ |
| PRC-R03 | La serie, la marca y el tipo quedan fijos cuando la plantilla ya tiene órdenes | §9.2, §2.4 | Dominio y servicio (`SERIES_IN_USE`, `BRAND_IN_USE`, `TYPE_IN_USE`) | — | ⚠️ |
| PRC-R04 | El alta de la plantilla acepta el logo y la firma como partes opcionales. Un archivo inválido no deja la plantilla creada | §9.2, convenciones §3.4 | `PurchaseOrderTemplateCommandServiceImpl.create`, que revisa los dos archivos con `PurchaseOrderTemplateDocuments.checkAll` antes de crear la plantilla y los guarda con `keepAll` en la misma transacción | — | ⚠️ |
| PRC-R05 | El `PATCH` cambia solo los campos presentes. Un archivo que no viaja no se quita; `removedDocuments` quita los que nombra, en la misma transacción que los campos y las subidas, después de comparar `updatedAt` bajo el candado. Un tipo desconocido responde `404 PURCHASEORDERTEMPLATEDOCUMENTTYPE_NOT_FOUND`, y quitar y subir el mismo tipo, `400`. `name`, `series` y el tipo en `null` responden `400`. `isActive` no va en ese `PATCH` (`400`): el estado cambia solo por `PATCH /{id}/status`. No hay `DELETE` de un documento | convenciones §3.4, D-134 | `PurchaseOrderTemplateCommandFromResourceAssembler` y `PurchaseOrderTemplateCommandServiceImpl.update`, con `PurchaseOrderTemplateDocuments.checkRemovals` | `PurchaseOrderTemplatesIntegrationTests` (`removesTheLogoInTheSamePatch`, `savesFieldsUploadsSignatureAndRemovesLogoAtomically`, `removingAFileTheTemplateDoesNotHaveChangesNothing`, `aTemplateChangesItsNameWithAPatchAndNotItsStatus`, `theStatusRouteRefusesAnIncompleteBodyAndATemplateNotRead`, `theDeleteOfADocumentIsNoLongerARoute`); `NoDeleteEndpointsTests`; `PurchaseOrderTemplateLockTests.aChangeOfStatusOverAnEditThatCommittedMeanwhileIsOutdated` | ✅ |
| PRC-R06 | Hay a lo sumo un documento de cada tipo. Quitar el archivo vacía `file_path` y conserva la fila y `kept_files`. Cada archivo se guarda bajo una llave de su contenido (`purchase-order-templates/{publicId}/{tipo}/{sha256}`), escrita antes de confirmar y apuntada al confirmar: reemplazarlo escribe una llave nueva, y la anterior y sus bytes se quedan (§2.7, D-134) | §9.2, §2.7 | Base: único `(template_id, document_type_id)`. Servicio: `PurchaseOrderTemplateDocuments` y `KeptFileStore` | `PurchaseOrderTemplatesIntegrationTests.replacingALogoPointsToANewKeyAndKeepsTheOldBytes`, `PurchaseOrderTemplateDocumentAuditTests` | ✅ |
| PRC-R07 | `LOGO` y `SIGNATURE` se dibujan en el PDF. Otro código se guarda y se devuelve, y no se imprime | §9.2 | `PurchaseOrderPdfGenerator` | — | ⚠️ |
| PRC-R08 | La columna de descuento es de la orden. Una línea activa lleva tasa de 0 a 1 solo si la columna está encendida. Una línea retirada queda inactiva. Encender o apagar la columna y agregar una línea van en el mismo guardado: la orden escribe su cabecera antes que sus líneas, porque el disparador de cada línea, inmediato, lee la columna de su cabecera (antes, la línea nueva se revisaba contra la columna anterior y el guardado fallaba con `500`) | D-132 | Dominio de la orden y base (V69 por línea, inmediato; V70 por cabecera, diferible). `PurchaseOrderRepositoryImpl.save` | `PurchaseOrdersIntegrationTests` (`discountRules`, `turningTheDiscountColumnOnWhileAddingALineKeepsTheRatesOfBothLines`, `turningTheDiscountColumnOffWhileAddingALineLeavesBothLinesWithoutARate`) | ✅ |
| PRC-R09 | El PDF del borrador refleja la plantilla y sus archivos: se dibuja solo cuando `DocumentService` genera el documento, y su versión cuenta `purchase_order_template_documents`. El de la orden enviada es una sola copia fiel con el teléfono de Atención, queda en `kept_files` (`purchase-orders/{publicId}/sent.pdf`) y no vuelve a generarse | §9.2, §2.7, D-134 | `PurchaseOrderQueryServiceImpl.pdfOf` y `renderDraftPdf`, `PurchaseOrderCommandServiceImpl` (envío) | `PurchaseOrdersIntegrationTests` (`sentPdfIsFrozen`, `theDraftIsDrawnOnlyWhenItsDocumentIsGenerated`, `aSentOrderPrintsThePhoneWhoeverSentIt`), `PurchaseOrderTemplatesIntegrationTests.theDraftSheetFollowsANewLogo` | ✅ |
| PRC-R10 | Cumplir solo sale de enviada. Anular no sale de cumplida. Una orden enviada no se edita | D-132 | Dominio de la orden | `PurchaseOrderTests`, `PurchaseOrdersIntegrationTests` | ✅ |
| PRC-R11 | Ningún permiso de compras actúa por sede: un rol de sede lee, pero no escribe proveedores, tipos, plantillas ni órdenes. Una orden de una plantilla general la escribe una asignación de toda la empresa; una de una plantilla con marca, también una de esa marca. Emitir decide el acceso con la plantilla que encuentra el candado (hallazgo NUEVO-A del plan de cierre) | §2.5, D-134 | `PurchaseOrderAccessPolicy`, `PurchaseOrderTemplateAccessPolicy` y `SupplierCommandServiceImpl` | `PurchaseOrderAccessPolicyTests` (`aBranchGrantManagesNoOrder`, `aBrandGrantManagesOrdersOfItsBrandAndOfGeneralTemplates`), `PurchaseOrderTemplateAccessPolicyTests` (`aBranchGrantWritesNoGeneralTemplate`, `aBrandGrantWritesNoGeneralTemplateNorType`), `PurchaseOrdersIntegrationTests` (`aRoleOfABranchDoesNotIssueNorSendOrders`, `issuingAnOrderDecidesAccessOnTheBrandTheLockFinds`), `SuppliersAccessIntegrationTests` (`aRoleOfABranchReadsSuppliersButDoesNotWriteThem`, `aGrantLimitedToABrandReadsSuppliersButDoesNotWriteThem`) | ✅ |
| PRC-R12 | Las lecturas piden solo `compras.ver` donde actúa. Escribir exige leer: una escritura, o la emisión de una orden con una plantilla o un proveedor, que quien llama no lee responde `404`. La respuesta de una escritura es lo que el comando guardó, sin volver a filtrar. La exportación de proveedores tiene el alcance de `compras.ver` | §5.6, D-134 | `ProcurementReadPermissions`, los servicios de consultas y de comandos | `SuppliersAccessIntegrationTests` (`aWriterWithoutVerGetsWhatItSaved`, `aWriterWithoutVerCannotEditNorChangeTheStatusOfASupplierItDoesNotRead`), `PurchaseOrdersIntegrationTests` (`aWriterWithoutVerInACompanyDoesNotReadItsOrders`, `aWriterWithoutVerCannotIssueAnOrderFromATemplateItDoesNotRead`), `PurchaseOrderTemplatesIntegrationTests.aWriterWithoutVerInACompanyDoesNotReadItsTemplatesNorTheirDocuments`, `SuppliersExportIntegrationTests.aWriterWithoutVerInBDoesNotGetBInTheExport` | ✅ |
| PRC-R13 | Cada escritura de plantillas, tipos y órdenes pide su propio permiso: crear, editar o desactivar plantillas; crear, editar, enviar, cumplir o anular órdenes. `allowedActions` ofrece cada acción por ese mismo permiso. V79 los siembra y los marca sin quitarle a ningún rol lo que hacía, también a los roles propios que tenían un grueso; V80 desmarca los gruesos en todos los roles, después de remarcar al rol propio que tiene un grueso y ninguno de sus finos, y se detiene si queda una marca gruesa | §5.6, §9.2, D-130 | Rutas (`@PermissionRequired`), `PurchaseOrderCommandServiceImpl`, `PurchaseOrderTemplateCommandServiceImpl`, `PurchaseOrderTypeCommandServiceImpl`, `PurchaseOrderAllowedActions`, `PurchaseOrderTemplateViewFactory` y `PurchaseOrderTypeViewFactory`. Base: V79 y V80 | `PurchaseOrderTemplatesIntegrationTests.theStatusRouteAsksForDeactivatingAndThePatchForEditing`, `PurchaseOrdersIntegrationTests.aRoleOfItsOwnWithTheCoarseCodeWritesOrdersThroughTheMarksOfV79`, `ProcurementPermissionsSplitMigrationTests`, `ProcurementCoarsePermissionsRevocationMigrationTests` | ✅ |
| PRC-R14 | Una orden se emite solo con una plantilla activa cuya empresa, marca —si la tiene— y tipo están activos: a una empresa o una marca inactiva no se le cuelga nada nuevo, y el tipo decide si se siguen emitiendo órdenes suyas. Responden `COMPANY_INACTIVE`, `COMMERCIALBRAND_INACTIVE` y `PURCHASEORDERTYPE_INACTIVE`, después del permiso. El tipo se lee con candado compartido, así que su desactivación espera a la orden o se confirma antes. La plantilla ofrece `issueOrder` en su `allowedActions` solo cuando el alta lo aceptaría, y la pantalla ofrece solo esas plantillas. Reactivar una plantilla pide su empresa y su marca activas (D-94) | D-84, D-94, D-115, §9.2 | `PurchaseOrderCommandServiceImpl` (`issuingRefusal`), `PurchaseOrderTypeRepository.findByIdForShare`, `PurchaseOrderTemplateViewFactory`, `PurchaseOrderTemplateCommandServiceImpl.activate`. Frontend: el formulario de la orden filtra por `issueOrder` | `PurchaseOrdersIntegrationTests` (`aTemplateOfAnInactiveCompanyAnswersCompanyInactive`, `aTemplateOfAnInactiveBrandAnswersCommercialBrandInactive`, `aTemplateOfAnInactiveTypeAnswersPurchaseOrderTypeInactive`), `PurchaseOrderTemplatesIntegrationTests` (`issuingAnOrderIsOfferedOnlyOnATemplateThatCanIssueOne`, `issuingAnOrderIsNotOfferedWithoutCreatingOrders`, `aTemplateOfAnInactiveBrandIsNotReactivated`), `ProcurementOpenApiTests` | ✅ |
| PRC-R15 | El borrador conserva lo retirado que ya tiene: el proveedor, el contacto de ese proveedor, la moneda y el modelo de cada línea se siguen guardando aunque se hayan desactivado después, mientras ese vínculo no cambie. Uno nuevo, o una línea que cambia de modelo, se revisa como en el alta. La pantalla ofrece lo que el borrador ya tiene, marcado como inactivo | D-84, D-108 | `PurchaseOrderReferences` (`supplier`, `contact`, `currency`) y `PurchaseOrderLines.resolveUpdateDrafts`, con `RetiredReferencePolicy`, que aplica la misma regla al tipo y a la Atención de la plantilla. Frontend: `PurchaseOrderForm`, y `PurchaseOrderTemplateForm` para la marca y el tipo que la plantilla ya tiene | `PurchaseOrdersIntegrationTests` (`aDraftKeepsTheSupplierItHasAfterItIsRetired`, `aDraftKeepsTheContactItHasAfterItIsRetired`, `aDraftKeepsTheCurrencyItHasAfterItIsRetired`, `aDraftLineKeepsTheModelItHasAfterItIsRetired`) | ✅ |
| PRC-R16 | La empresa que nombra el alta de una orden es la de su plantilla: otra, o una que no existe, responde `404 PURCHASEORDERTEMPLATE_NOT_FOUND`. Cada `unitPrice` viaja con el código de la moneda de la orden; otro responde `400 VALIDATION_ERROR` sobre `unitPrice.currency` | §9.2, convenciones §5.2 | `PurchaseOrderCommandServiceImpl` | `PurchaseOrdersIntegrationTests` (`aCompanyThatIsNotTheOneOfTheTemplateAnswersTemplateNotFound`, `aUnitPriceInAnotherCurrencyAnswersValidationError`), `ProcurementOpenApiTests.writingAnOrderNamesTheCurrencyOfItsUnitPrices` | ✅ |
| PRC-R17 | El PDF tiene el formato con que la empresa emite sus órdenes: una plantilla HTML (`resources/templates/documents/purchase-order.html`) que `HtmlTemplates` llena con Thymeleaf y `HtmlPdf` arma como PDF (openhtmltopdf sobre PDFBox). De una orden a otra solo cambian los datos y la columna de descuento. Imprime todas las líneas activas, en las páginas A4 que hagan falta: la cabecera de la tabla se repite en cada página y el pie de cada una nombra la orden y la página cuando hay más de una. El subtotal y el total suman todas las líneas (antes, las que no cabían en la primera página quedaban fuera de la hoja y de sus totales, también en el PDF congelado al enviar). Imprime la condición de pago bajo la fecha y el número (`COND. PAGO`), los montos con el símbolo de la moneda y el código del tipo de documento del emisor, nunca `RUC` fijo. Un carácter que la fuente no escribe sale como `?` en lugar de hacer fallar la hoja. El concepto del pedido (`purpose`) sigue sin imprimirse | §9.2, D-132, D-135 | `PurchaseOrderPdfGenerator`, `PurchaseOrderSheets` (resuelve el tipo de documento del emisor), `shared/infrastructure/templates`; `CurrencySummary.symbol` en `location` | `PurchaseOrderPdfGeneratorTests` (`theSheetFollowsThePrintedFormat`, `everyLineIsPrintedAndAddedUpOnAsManyPagesAsItTakes`, `theIssuerPrintsTheCodeOfItsTaxDocument`, `thePaymentTermsArePrinted`, `aLongTextWrapsInsteadOfRunningOffThePage`, `aCharacterTheFontCannotWriteIsPrintedAsAQuestionMark`) | ✅ |
| PRC-R20 | Una línea de vehículo nombra el año de su modelo (`model_year`, V82), entre 2017 y el año siguiente al actual como el de la unidad; una de concepto no lo tiene, y una línea guardada antes de pedirlo queda sin año hasta que su borrador se guarde otra vez. Lo contrario responde `400 VALIDATION_ERROR`. Cada línea de la respuesta trae `modelYear` y `description`, el texto que imprime el PDF: `VEHÍCULO {marca} {modelo}, AÑO {año}`, o el concepto tal como se escribió. La pantalla arma la línea de vehículo con marca, modelo y año, y la libre con su texto, y en solo lectura muestra `description` | §9.2, D-135 | `PurchaseOrderItem`, `PurchaseOrderLineDescription` (una sola regla para la API y el PDF). Frontend: `PurchaseOrderForm` | `PurchaseOrderItemTests` (`aLineOfAModelNeedsItsYear`, `aLineSavedWithoutYearIsReadAndCorrectedWithOne`), `PurchaseOrderLineDescriptionTests`, `PurchaseOrdersIntegrationTests.aVehicleLineCarriesItsYearAndReadsAsTheSheetPrintsIt` | ✅ |
| PRC-R18 | El total de una línea se redondea a los decimales de la moneda de la orden, mitad hacia arriba, y el subtotal suma las líneas así redondeadas. La API envía el precio unitario, el total de cada línea y el subtotal con exactamente los decimales de la moneda (convenciones §5.2), y el PDF y la hoja de la pantalla muestran los mismos montos; la hoja calcula en enteros sobre el texto decimal, sin coma flotante | §9.2, convenciones §5.2 | `PurchaseOrderItem.lineTotal(decimalPlaces)`, `PurchaseOrder.subtotal(decimalPlaces)`, `PurchaseOrderViewFactory`. Frontend: `purchase-order-amounts`, con `decimalPlaces` en `CurrencyOption` | `PurchaseOrderItemTests.lineTotalIsRoundedHalfUp`, `PurchaseOrderTests.subtotalAddsTheRoundedLines`, `PurchaseOrdersIntegrationTests.theAmountsOfAnOrderCarryTheDecimalsOfItsCurrency`; `purchase-order-amounts.spec.ts`, `purchase-order-form.spec.ts` | ✅ |
| PRC-R19 | Guardar o quitar un archivo de la plantilla —en el `PATCH` multipart o con `PUT /{id}/documents/{type}`— mueve su `updatedAt` y `updatedBy`, aunque ninguna columna propia cambie: un `PATCH` o un cambio de estado abiertos antes responden `422 PURCHASEORDERTEMPLATE_OUTDATED` en lugar de quitar un logo que no vieron. `PUT /{id}/documents/{type}` sigue sin pedir `updatedAt`: reemplaza el archivo entero | D9, convenciones §3.4 | `PurchaseOrderTemplateRepository.recordRevision`, `PurchaseOrderTemplateCommandServiceImpl` | `PurchaseOrderTemplatesIntegrationTests.aFileReplacedMeanwhileMakesWhatWasOpenedBeforeOutdated` | ✅ |

---

## 5. Relación con otros contextos

`package-info.java` declara `shared`, `organization::acl`, `location::acl`, `vehiclecatalog::acl`, `documents::acl` e `iam::acl`. El código llama a las cuatro primeras fachadas desde su `External<Contexto>Service`, y a `documents` por `KeptFileStore` y `DocumentService`. Declara `iam::acl` y todavía no llama a `IamContextFacade`: el autor de los registros llega por el auditor de `shared`. No declara `fleet`.

| Necesita de | Para qué |
|---|---|
| `organization` | Empresa, marca comercial y colaborador de Atención |
| `location` | Domicilio, tipo de documento y moneda |
| `vehiclecatalog` | Marca y nombre del modelo que imprime una línea |
| `documents` | Bytes que no vencen, y el PDF del borrador que sí vence |
| `fleet` | Cuando existan las facturas: preguntar el VIN y, si el valor llega a cero, pedir la anulación (mapa §6.1). Hoy no |

Ningún contexto llama a `procurement`. Los costos de la unidad los leerá la capa de lectura.

En el frontend, las pantallas viven en `src/app/procurement`. El menú las muestra desde `app-navigation.ts` (procesos y pantallas, apartado 3.1).

---

## 6. Estado y pendientes para cerrar

**Construido e integrado en `develop` el 23/10/2026**

- Proveedores, con domicilio y contactos en la misma ficha.
- Tipos de orden, dentro de Plantillas de OC.
- Plantillas: alta única con imágenes opcionales, `PATCH` parcial, documentos por tipo. El estado, que viajaba en ese `PATCH`, va desde el 26/10/2026 por `PATCH /{id}/status` (PRC-R05).
- Órdenes: borrador, envío, cumplimiento, anulación y PDF vivo o congelado.
- Diccionario §9.2 y §2.7, convenciones §3.4, y las pantallas de procesos y pantallas.

**Cerrado con la auditoría del 25/10/2026** (diccionario v1.31, D-130, D-133 y D-134; [plan de cierre](../plan-de-cierre-de-la-auditoria-2026-10-25.md))

- Lo que expone datos: los proveedores de otra empresa y el PDF del borrador con el teléfono de Atención para quien no lo ve (PRC-R02, PRC-R12).
- Quién escribe y quién lee (PRC-R11, PRC-R12), el emisor copiado (PRC-R01), los archivos en el mismo `PATCH` y bajo su llave de contenido (PRC-R05, PRC-R06), y el PDF enviado como copia fiel (PRC-R09).
- Los códigos de compras —las referencias inactivas, un solo prefijo, el dígito de control del proveedor, las carreras traducidas— están en convenciones §7.2, y no se copian aquí.
- La exportación de proveedores (convenciones §8.1). La lista de órdenes y la de plantillas se exportan con el paso de reportes (D-116).
- Los ocho permisos de plantillas, tipos y órdenes, que reemplazan a `compras.gestionar_plantillas` y `compras.gestionar_ordenes` (V79 y V80, PRC-R13); el estado de la plantilla en su propia ruta y el retiro del `DELETE` de un documento (PRC-R05); el emisor fuera de los pedidos y el `DROP` de las columnas de V73 (V81, PRC-R01).

**Cerrado el 27/10/2026** (hallazgos de órdenes y plantillas; `feature/procurement` en los dos repositorios)

- La orden se lee en el frontend como la responde la API: `supplierLegalName`, `templateName`, `hasPdf`, `subtotal` y, en cada línea, `modelName`, `brandName` y `lineTotal`. La lista, la ficha y la hoja muestran el proveedor, y una línea de vehículo su marca y su modelo. La prueba contra el backend real lo verifica.
- Las opciones de marca y modelo de cada línea van con la línea, no con su posición: quitar una línea ya no cambia las de las siguientes.
- El descuento y una línea nueva en el mismo guardado (PRC-R08); lo que se emite y lo que se reactiva (PRC-R14); lo retirado que conserva el borrador (PRC-R15); la empresa y la moneda del pedido (PRC-R16); el número de orden ordena por serie, año y correlativo.
- La hoja de una orden enviada avisa que el documento del proveedor es su PDF.

**Cerrado el 27/10/2026, segunda revisión** (`feature/procurement` en los dos repositorios, sin migraciones)

- El PDF imprime todas las líneas en las páginas que hagan falta y suma todas en sus totales, con la condición de pago y el tipo de documento del emisor (PRC-R17).
- Los montos de la orden se redondean por línea a los decimales de su moneda, igual en la API, el PDF y la hoja (PRC-R18).
- El teléfono de Atención se copia del colaborador sea quien sea quien guarda la plantilla (PRC-R02).
- Un archivo de la plantilla mueve su `updatedAt` (PRC-R19).
- El formulario de la plantilla ofrece la marca y el tipo retirados que ya tiene, marcados como inactivos (PRC-R15).
- La prueba en vivo de compras pasa contra el backend de la rama.

**Cerrado el 27/10/2026, tercera revisión** (`feature/procurement` en los dos repositorios; migración V82; diccionario v1.32, D-135)

- El PDF sale de la plantilla HTML con el formato de la empresa (PRC-R17), en `resources/templates/documents`; `resources/templates/email` queda lista para los correos, que llegan después.
- Una línea de vehículo guarda el año del modelo y la API devuelve la descripción que imprime el PDF (PRC-R20).
- La pantalla de la orden deja de ser la hoja impresa: la plantilla primero —elegida sola cuando es la única—, un encabezado compacto, proveedor, condiciones, líneas de vehículo o libres con dos botones y un bloque plegable con lo que imprime la plantilla. El logo y la firma ya no se muestran en la pantalla; salen en el PDF.

**Sigue abierto en el paso 7**

- Facturas y notas de crédito, con sus pantallas. De ellas depende la pregunta de dónde nace la unidad (mapa, decisión 5).
- La fachada de `fleet`, que este contexto todavía no declara.
- Una prueba de integración de lo que queda fijo, del alta con imágenes y del dibujo de los tipos (PRC-R03, PRC-R04 y PRC-R07). El `PATCH` y los documentos ya la tienen (PRC-R05, PRC-R06).
- El saldo del proveedor (`/suppliers/{id}/balance`), con el paso 8.

---

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿La unidad nace en Inventario o al registrar la factura? | Procesos y pantallas, pregunta 1; mapa, decisión 5 |
| ¿Quién crea un tipo de archivo de plantilla que no sea logo ni firma? Hoy solo los siembra la migración | Diccionario §9.2. No hay ruta de alta del catálogo |
| Facturas de más de veinte días, anulación y notas de crédito | Diccionario §9.3 a §9.6, cuando se construyan |
