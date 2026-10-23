# Contexto `procurement` — ERP Millennium Mobility

**Actualizado:** 23/10/2026 (proveedores, tipos de orden, plantillas con sus archivos y órdenes de compra, en `develop`; facturas, notas de crédito y pagos siguen sin código) · **Estado:** borrador para validar; el paso 7 sigue abierto · **Fuente:** diccionario de datos v1.28 (§9.1, §9.1.1, §9.2, §2.7 `kept_files`, D-35, D-36, D-37, D-102, D-105, D-125 y D-132), [mapa de contextos](../mapa-de-contextos-millennium-mobility.md), [convenciones de la API](../convenciones-de-la-api-millennium-mobility.md) §2.2 y §3.4, [procesos y pantallas](../procesos-y-pantallas-millennium-mobility.md) (P4) y [orden de implementación](../orden-de-implementacion-millennium-mobility.md) (paso 7). Si este documento y el diccionario no coinciden, manda el diccionario.

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
| El domicilio del emisor y el del proveedor | `location`, `addresses` (§4.6) | La plantilla y el proveedor lo crean, lo corrigen o lo mudan por la fachada. No hay ruta de dirección |
| La empresa, la marca comercial y el colaborador de Atención | `organization` | La plantilla apunta a ellos. El teléfono de Atención sale de la vista del colaborador: sin `configuracion.ver_datos_personales_colaboradores` no se guarda (D-105) |
| El tipo de documento del proveedor y del emisor | `location` | `procurement` lo verifica por la fachada |
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

Nada de lo construido se borra. El `DELETE` de un documento de plantilla vacía la llave y deja la fila (convenciones §3.1 y §3.4).

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
| Alta | Dentro de Plantillas de OC, con `compras.gestionar_plantillas` |
| Qué cambia | El nombre, con `PUT /purchase-order-types/{id}` |
| Qué queda fijo | El código, en cuanto una plantilla lo usa |
| Retiro | `PATCH /purchase-order-types/{id}/status` |
| Nunca | Se borra. Agregar un tipo no cambia cómo funciona la orden |

### Plantilla

| Momento | Qué pasa |
|---|---|
| Alta | Un solo `POST /purchase-order-templates`, multipart. La parte `template` es la ficha. `logo` y `signature` pueden ir, las dos, una o ninguna |
| Qué cambia | `PATCH /purchase-order-templates/{id}`. Un campo que el JSON no trae se queda; uno que trae en `null` se vacía si admite vacío. `updatedAt` tiene que ser el de la fila (`422 PURCHASEORDERTEMPLATE_OUTDATED`). El estado es `isActive` en ese mismo `PATCH`, también desde la lista, con solo `{ updatedAt, isActive }` |
| Qué queda fijo | La serie, la marca comercial y el tipo, cuando la plantilla ya tiene órdenes |
| Archivos | `PUT /{id}/documents/{type}` reemplaza uno. `DELETE /{id}/documents/{documentId}` vacía la llave. `GET /{id}/documents/{documentId}/file` devuelve los bytes. La respuesta de la plantilla lista solo los documentos que tienen archivo |
| Retiro | `isActive: false` en el `PATCH`. Las órdenes ya emitidas la siguen usando |
| Nunca | `PUT` de la ficha entera, `PATCH /{id}/status`, `POST /{id}/signature` ni `POST /{id}/signature-removal`. Esas rutas ya no existen |

El proveedor no se elige en la plantilla. Se elige al emitir la orden. La columna Atención es un colaborador de la empresa.

### Orden

| Momento | Qué pasa |
|---|---|
| Alta | Logística › Órdenes de compra. El número se asigna al crear el borrador y no se reutiliza (D-36). Ahí se elige la plantilla, el proveedor y si esa orden lleva columna de descuento |
| Qué cambia | El borrador, con `PUT /purchase-orders/{id}`. Una línea que se quita queda inactiva y no entra al PDF ni a los totales |
| Envío | `POST /{id}/sending`. El PDF de ese momento queda en `kept_files` (`purchase-orders/{publicId}/sent.pdf`) y ya no sigue a la plantilla |
| Cumplida | `POST /{id}/fulfillment`, solo desde enviada |
| Anulada | `POST /{id}/annulment`. No sale de cumplida |
| PDF | `GET /{id}/pdf`. En borrador es el documento generado, que refleja la plantilla de ahora. Enviada, cumplida o anulada entrega el PDF congelado |
| Nunca | Se edita después de enviada: se anula y se emite otra. Se borra |

---

## 3. Flujos funcionales

Quien lee proveedores, plantillas y órdenes entra con `compras.ver`. Crear, editar o desactivar un proveedor pide el permiso de esa operación. Plantillas y tipos piden `compras.gestionar_plantillas`. Crear, editar el borrador, enviar, cumplir y anular una orden piden `compras.gestionar_ordenes`. El menú de Plantillas de OC también se muestra con `compras.gestionar_ordenes`, porque quien emite elige una plantilla; el `GET` de la ruta pide `compras.ver` antes que el servicio.

### F1 · Registrar un proveedor

1. Abre Configuración › Proveedores (`/procurement/suppliers`).
2. Completa la ficha, el domicilio y los contactos, y guarda (`POST /suppliers`).
3. Corregir o mudar el domicilio, o mantener los contactos, es otro guardado de la ficha (`PUT /suppliers/{id}`).
4. Activar o desactivar es la acción de la fila (`PATCH /suppliers/{id}/status`).

### F2 · Dar de alta una plantilla, con o sin imágenes

1. Abre Configuración › Plantillas de OC (`/procurement/purchase-order-templates`) y entra al formulario de página.
2. Elige empresa, línea de negocio si la plantilla no es general, y tipo. El tipo se crea en esa misma pantalla cuando falta.
3. La hoja propone el emisor desde la empresa. La observación y las instrucciones son de esta plantilla.
4. Puede elegir un logo, una firma, los dos o ninguno. Al guardar, el formulario manda la ficha y los archivos elegidos en un solo `POST`.
5. Si un archivo no es PNG, JPEG, WebP o PDF, o pasa de 5 MB, no se crea la plantilla (`422`).

### F3 · Corregir la plantilla o su estado

1. Guardar el formulario manda un `PATCH` multipart con la ficha y, si se eligió una imagen nueva, esa parte. Una imagen que no se tocó no viaja y se conserva.
2. Quitar el logo o la firma es `DELETE` del documento. La fila y los bytes se quedan; la hoja deja de mostrarlo.
3. Activar o desactivar desde la lista manda `{ updatedAt, isActive }` y no reescribe la ficha.
4. Reemplazar un solo tipo, sin abrir el resto de la ficha, es `PUT /{id}/documents/LOGO` o `…/SIGNATURE`.

### F4 · Emitir una orden

1. Abre Logística › Órdenes de compra (`/procurement/purchase-orders`).
2. Elige la plantilla. La hoja muestra en solo lectura lo que imprime, incluido el logo y la firma cuando la plantilla los tiene.
3. Elige el proveedor y, si quiere, un contacto de ese proveedor. Decide si esta orden lleva columna de descuento y carga las líneas.
4. El borrador se guarda con `PUT`. Enviar, cumplir y anular son los tres `POST` de hecho.
5. El PDF del borrador sigue a la plantilla. El de la orden enviada, no.

---

## 4. Reglas de negocio

| Id | Regla | Fuente | Dónde se aplica | Prueba | Estado |
|---|---|---|---|---|---|
| PRC-R01 | El proveedor y su contacto se eligen al emitir la orden. La plantilla no los fija. Atención es un colaborador de la empresa | §9.2 | Servicio: `PurchaseOrderTemplateCommandServiceImpl`. Frontend: el formulario de la plantilla no ofrece proveedor | `PurchaseOrderTemplateAttentionPolicyTests` | ✅ |
| PRC-R02 | Sin `configuracion.ver_datos_personales_colaboradores`, el teléfono de Atención que se guarda es nulo. El de un colaborador que la sesión no puede ver no se copia | D-105 | `PurchaseOrderTemplateViewFactory` y el servicio de comandos | `PurchaseOrderTemplateAttentionPolicyTests` | ✅ |
| PRC-R03 | La serie, la marca y el tipo quedan fijos cuando la plantilla ya tiene órdenes | §9.2, §2.4 | Dominio y servicio (`SERIES_IN_USE`, `BRAND_IN_USE`, `TYPE_IN_USE`) | — | ⚠️ |
| PRC-R04 | El alta de la plantilla acepta el logo y la firma como partes opcionales. Un archivo inválido no deja la plantilla creada | §9.2, convenciones §3.4 | `PurchaseOrderTemplatesController.create` y `attachImages`, en la misma transacción | — | ⚠️ |
| PRC-R05 | El `PATCH` cambia solo los campos presentes. Un archivo que no viaja no se quita. `isActive` va en ese `PATCH` | convenciones §3.4 | `toCommand(id, JsonNode)` y `PurchaseOrderTemplateCommandServiceImpl.update` | — | ⚠️ |
| PRC-R06 | Hay a lo sumo un documento de cada tipo. Quitar el archivo vacía `file_path` y conserva la fila y `kept_files` | §9.2 | Base: único `(template_id, document_type_id)`. Servicio: `clearDocument` | — | ⚠️ |
| PRC-R07 | `LOGO` y `SIGNATURE` se dibujan en el PDF. Otro código se guarda y se devuelve, y no se imprime | §9.2 | `PurchaseOrderPdfGenerator` | — | ⚠️ |
| PRC-R08 | La columna de descuento es de la orden. Una línea activa lleva tasa de 0 a 1 solo si la columna está encendida. Una línea retirada queda inactiva | D-132 | Dominio de la orden y base (V70, el disparador del descuento es diferible) | `PurchaseOrdersIntegrationTests` | ✅ |
| PRC-R09 | El PDF del borrador refleja la plantilla. El de la orden enviada queda en `kept_files` y no vuelve a generarse | §9.2, §2.7 | `PurchaseOrdersController.pdf` | `PurchaseOrdersIntegrationTests` | ✅ |
| PRC-R10 | Cumplir solo sale de enviada. Anular no sale de cumplida. Una orden enviada no se edita | D-132 | Dominio de la orden | `PurchaseOrderTests`, `PurchaseOrdersIntegrationTests` | ✅ |

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
- Plantillas: alta única con imágenes opcionales, `PATCH` parcial, estado en ese `PATCH`, documentos por tipo.
- Órdenes: borrador, envío, cumplimiento, anulación y PDF vivo o congelado.
- Diccionario §9.2 y §2.7, convenciones §3.4, y las pantallas de procesos y pantallas.

**Sigue abierto en el paso 7**

- Facturas y notas de crédito, con sus pantallas. De ellas depende la pregunta de dónde nace la unidad (mapa, decisión 5).
- La fachada de `fleet`, que este contexto todavía no declara.
- Una prueba de integración del `PATCH` y de los documentos de la plantilla (PRC-R03 a PRC-R07).
- El saldo del proveedor (`/suppliers/{id}/balance`), con el paso 8.

---

## 7. Preguntas abiertas

| Pregunta | Dónde se lleva |
|---|---|
| ¿La unidad nace en Inventario o al registrar la factura? | Procesos y pantallas, pregunta 1; mapa, decisión 5 |
| ¿Quién crea un tipo de archivo de plantilla que no sea logo ni firma? Hoy solo los siembra la migración | Diccionario §9.2. No hay ruta de alta del catálogo |
| Facturas de más de veinte días, anulación y notas de crédito | Diccionario §9.3 a §9.6, cuando se construyan |
