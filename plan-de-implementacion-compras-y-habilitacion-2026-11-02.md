# Auditoría y plan de implementación: de la factura de compra a la habilitación, las notas, los pagos y las cargas

**Versión 4 · 03/11/2026.** Cierra de punta a punta todo lo que planifica: las fases 1 a 5 y el paquete de cargas de archivos, cada uno con su especificación.

**Cómo se llegó aquí:**
- **Versión 1:** auditó tres puntos de partida, el backend `b1264ed5` (hasta V91), el frontend `88312e13` y la documentación `6ebb900`, junto con una investigación de ERP comparables y de la norma de SUNAT.
- **Versión 2:** sumó una auditoría estricta del plan, la revisión de las fuentes del cliente (la guía funcional v2.0 y el flujograma maestro de estados) y las decisiones D-157 a D-168.
- **Versión 3:**
  - cierra los hallazgos de la auditoría de la versión 2 con D-169 a D-177, en el [diccionario](diccionario-de-dtos-millennium-mobility.md) v1.38;
  - agrega las especificaciones listas para implementar: [facturas de compra](especificacion-facturas-de-compra-2026-11-02.md) y [habilitación](especificacion-habilitacion-2026-11-02.md);
  - el 03/11/2026 cierra, con D-178 a D-189, las preguntas abiertas que dejaron esas especificaciones.
- **Versión 4:**
  - audita las fases 4 y 5 y el paquete de cargas, y los cierra con D-190 a D-202 (§2.8);
  - agrega sus especificaciones: [notas de crédito](especificacion-notas-de-credito-2026-11-03.md), [pagos a proveedores](especificacion-pagos-a-proveedores-2026-11-03.md) y [cargas de archivos](especificacion-cargas-de-archivos-2026-11-03.md).

**Cómo leerlo.**
- **§1** fija el alcance.
- **§2** resume los hallazgos de las tres auditorías y cómo quedó cada uno.
- **§3** lista las decisiones y lo que sigue abierto.
- **§4** fija el orden.
- **§5** es el plan por fases, con su esfuerzo.
- **§6** trata los riesgos.
- **§7** explica cómo se construye, se verifica y se despliega.
- **El detalle de cada fase está en su especificación:** el contrato de la API, el esquema, las reglas con sus pruebas, las pantallas y la secuencia de commits.

---

## 1. Alcance

**El flujo cubierto** empieza en la orden de compra, que ya está construida, y sigue así:
1. la factura de compra;
2. cada unidad, con su VIN;
3. su ingreso físico;
4. su habilitación, hasta que la unidad queda **Disponible**.

Con autos llega una factura por auto; con motos, una factura por lote de hasta unas 80 unidades.

**Puntos del [orden de implementación](orden-de-implementacion-millennium-mobility.md):**
- **En detalle:** el 4 (fechas en pantalla), el 5 (facturas) y el 8 (habilitación).
- **En resumen:** el 6 (notas de crédito), el 7 (pagos) y el paquete de cargas de archivos, que D-168 separa del punto 8.

**Línea de negocio.** Millennium financia vehículos a sus clientes: autos, motos y trimotos (Leasing Car, Leasing Bike y Leasing Tuk Tuk). Compra las unidades nuevas a distribuidores locales que importan: no importa directamente, así que el ISC viene dentro del precio del proveedor.

Queda fuera lo que no ocurre en este negocio:
- la importación directa, con sus costos de aduana;
- la recompra de un VIN mediante una factura de proveedor;
- la conversión de unidades a gas después de comprarlas.

**Punto de partida**

| Pieza | Estado |
|---|---|
| Proveedores, tipos y plantillas de orden, órdenes (`procurement`) | Construido, con la columna de descuento por orden (`has_discount_column`), su PDF y 52 clases de prueba |
| Tipos de cambio por día (`location`, V89 a V91) | Construido: registro, corrección con motivo, aviso de salto grande y candado consultivo |
| Unidades, estados, caminos, historial, kilometraje y fondos con cesiones (`fleet`) | Construido |
| Equipamiento con su aplicabilidad y la pantalla de fábrica del modelo (`vehiclecatalog`, V35) | Construido |
| Facturas, notas de crédito, pagos | Sin construir. Están diseñados en el diccionario §9 y §10, y las facturas, especificadas |
| Habilitación (bloque 9, §11) | Sin construir. Está especificada |
| Fachada de `fleet` para otros contextos | No existe |

---

## 2. Hallazgos y cómo quedaron

**Severidades:**
- **Crítico o Alto:** bloqueaba construir o rompía datos, seguridad o rendimiento.
- **Medio:** generaba deuda o inconsistencia.
- **Bajo:** limpieza.

Las referencias a líneas de código son del punto de partida.

### 2.1 Veredicto

El código existente no tiene defectos críticos. Las políticas de RLS, las carreras de llaves naturales, los candados y las pruebas siguen un patrón sólido y se reutilizan. Lo que faltaba definir quedó cerrado por D-157 a D-177 y por las dos especificaciones.

**Puertas:**
- **Fase 1:** empieza ya.
- **Fases 2 y 3:** empiezan cuando el responsable valide el texto de la v1.38 y las especificaciones.
- **Fases 4 y 5:** tienen sus decisiones (D-190 a D-213) y sus especificaciones. El contador solo valida, sin detenerlas (§3.2).
- **Paquete de cargas:** especificado en tres entregas (D-208). Su primer paso, el perfilado, espera los Excel reales.

### 2.2 Primera auditoría: críticos

| ID | Hallazgo | Cómo quedó |
|---|---|---|
| A1 | El diccionario no podía guardar la especificación de la factura | Diccionario v1.38 |
| A2 | El costo no tenía base definida | D-159: el precio se guarda con impuestos, y el costo sin IGV es valor ÷ (1 + tasa), como la columna 49 de la guía |
| A3 | El VIN es único en todo el sistema, incluso entre unidades anuladas, así que anular las unidades con su factura impedía volver a usarlas | D-161: anular la factura no anula sus unidades |
| C1 | Faltaba la carga inicial del Excel | D-168 y D-177, en un paquete propio |
| C3 | La puerta de Disponible cubría un solo camino (V47:224-232) | D-164 y D-170: `requires_enablement`, fijo en Disponible |

### 2.3 Primera auditoría: altos

| ID | Hallazgo | Cómo quedó |
|---|---|---|
| A4 | **El alta unitaria no sirve para un lote** (`VehicleCommandServiceImpl.java:132,346`) | Un comando de lote (paso 5.3) |
| A5 | **Un fallo puede dejar medio lote** (`VehicleCommandServiceImpl.java:155-160,357-361`) | Validar todo antes de escribir, con `MANDATORY` |
| A6 | **El alta exige `stock.crear_vehiculo` dos veces** | Un comando aparte |
| A7 | **No hay fachada de `fleet`** | `fleet/interfaces/acl`, y puertos que compras implementa |
| A8 | **La anulación de una unidad no existe** (`Vehicle.java:53`) | La vía explícita de D-104, en la fase 2 |
| A9 | **Un VIN puede quedar en dos facturas vigentes** | Un único parcial en las líneas vigentes (D-160) |
| A10 | **Faltan permisos**, y el texto de Contabilidad estaba vencido | Siembra por fase y en dos tiempos (D-176). El texto de Contabilidad, corregido (D-163) |
| A11 | **`location` no da la tasa vigente a una fecha** | Paso 5.2 |
| A12 | **El diálogo de tasa no se puede abrir desde otro contexto** | Paso 5.9 |
| A13 | **«Corregir» en dos pasos** | Desaparece: se anula y se registra otra con «Anular y copiar» (D-161) |
| A14 | **La habilitación no gobierna Disponible** | D-164, D-165 y D-170 |
| A15 | **`vehiclecatalog` no entrega las reglas de habilitación** | Paso 8.3 |

### 2.4 Primera auditoría: medios y bajos

| ID | Hallazgo | Cómo quedó |
|---|---|---|
| A16 | La línea no guarda modelo, año ni marca | D-160 |
| A17 | La orden de compra | D-162 y D-172 |
| A18 | El alcance por marca | D-163 |
| A19 | La tasa cambia antes de guardar | Se envía el día de la tasa vista; responde 422 si cambió |
| A20 | El candado de decimales (D-146) solo está en las órdenes | `CurrencyUsage` por cada tabla con montos |
| A21 | El formulario no escala a 80 líneas | Paso 5.10, con tope de 100 líneas (D-160) |
| A22 | Una tabla editable fuera de `mm-data-manager` | Revisión de DESIGN (pasos 5.10 y 8.8) |
| A23 | Las pruebas «En construcción» y los mocks e2e | Paso 5.11 |
| A24 | Las contradicciones en pagos | Fase 5 |
| A25 | El canal de dealers y la migración | La migración, por D-168 y D-177; el canal de dealers, en la fase 5 |
| A26 | El alcance del punto 4 | Fase 1, reducida a sus tres tareas oficiales |
| A27 | Texto vencido | Corregido. Los contextos de compras y de flota se cierran en el paso 5.12 |
| A28 | Clases grandes | Lo nuevo va en piezas separadas |
| A29 | `Year.now()` usa la zona de la JVM | `Clock` y la zona de la empresa |

### 2.5 Segunda auditoría (versión 1): estricta

| ID | Hallazgo | Cómo quedó |
|---|---|---|
| E1 | «Corregir» daba a Operaciones una forma de anular | Desaparece (D-161) |
| E2 | Con el ISC apagado, la base y el IGV impresos no cuadraban | D-159: no se guardan |
| E3 | Reglas de SUNAT escritas como globales | D-160: el formato se verifica solo en Perú |
| E4 | No había regla para una tasa corregida después de usarse | §4.8 ya la tenía. Además, D-160 guarda el día de la tasa, y una factura con tasa equivocada se arregla con «Anular y copiar» |
| E5 | Faltaba el contrato de la API | En las especificaciones |
| E6 | Los bonos de campaña quedaban sin reflejar | D-176 |
| E7 | La versión 1 decía que el fondo recibía el valor del vehículo | D-159: el fondo recibe el valor de factura |
| E8 | La aceptación no se podía medir | §5 y las especificaciones |
| E9 | El alcance por sede | D-163 |
| E10 | Desajustes de modelo, año o marca | D-160 |
| E11 | Avisos de vencimiento | D-167 |
| E12 | La boleta | Sigue como opción; el costo es el valor completo (D-159) |
| E13 | Las tasas de las notas | D-23 y D-152. Queda el reparto entre varias facturas (§3.2) |
| E14 | Las placas | D-165: cuentan las placas físicas |
| E15 | El archivo huérfano si se revierte la transacción | Se sube después de validar y se borra si la transacción se revierte |
| E16 | Permisos asignados al paso equivocado | Fases 2 y 3, y el paquete de cargas |
| E17 | No había plan de salida a producción | §7 |

### 2.6 Tercera auditoría (versión 2)

| ID | Hallazgo | Cómo quedó |
|---|---|---|
| H1 | «No usar facturas reales hasta tener notas de crédito» no tenía un mecanismo | **D-176:** los permisos de facturas se siembran solo al superadministrador en la fase 2, y al resto de la matriz en la fase 4 |
| H2 | Una factura en la moneda de la empresa no tenía regla de tasa | **D-169:** tasa 1, con la fecha de la factura como día de la tasa |
| H3 | Había criterios de aceptación de la fase 2 imposibles de probar | El puerto «dependientes de la factura» responde «ninguno» hasta las fases 4 y 5, y esas pruebas pasan a ellas |
| H4 | La puerta de Disponible se podía desmarcar desde el catálogo de estados | **D-170:** fija en Disponible. El dato llega a la API y a la pantalla de estados (paso 8.6) |
| H5 | Faltaban el contrato de la fase 3 y las especificaciones de pantallas | Las dos especificaciones y la revisión de DESIGN que listan |
| H6 | D-168 se decidió sin datos reales | **D-177:** excepciones de la carga inicial, más un primer paso del paquete que perfila los Excel reales |
| M1 | El permiso para cambiar lo no fiscal | D-172: `compras.registrar_factura` |
| M2 | Cambiar la orden después de registrar | D-172 |
| M3 | Unidades sin factura vigente | D-175: un indicador y un filtro en Inventario |
| M4 | La bitácora de la anulación | D-175 |
| M5 | Dependencias mal declaradas | Corregidas en §5 |
| M6 | Códigos estables del estado de habilitación | D-171 |
| M7 | Tipos y tamaño de los archivos | D-174 |
| M8 | El costo del equipo con su moneda | Paso 8.4 |
| M9 | Las líneas de concepto | D-173: solo líneas de vehículo en la primera versión |
| M10 | Las tablas de reglas con sus pruebas | En las especificaciones, con estado ❌ hasta construirse |
| M11 | La base y el IGV derivados pueden diferir en céntimos | Riesgo aceptado: la exportación los rotula «calculados», y si el contador necesita el IGV exacto, se guarda el impreso |
| B1 | El objetivo de rendimiento no tenía una medición previa | Primero se mide la línea base (§5) |
| B2 | La guarda de la fase 1 sin lugares definidos | Paso 1.3, con la lista de lugares permitidos |
| B3 | Las columnas de la exportación | En la especificación de facturas |
| B4 | Faltaba una estimación | §5 |
| B5 | Los pasos de los permisos de cargas en §5.6 | Sin cambio: el paso 9 del diccionario se entrega en dos partes, la fase 3 y el paquete de cargas |
| B6 | «Anular y copiar» de una factura de más de 20 días | Exige `compras.registrar_factura_antigua`, que tiene quien puede anular |

### 2.8 Cuarta auditoría: fases 4 y 5 y cargas (03/11/2026)

| ID | Hallazgo | Cómo quedó |
|---|---|---|
| D1 | **El saldo a favor se contaba dos veces.** Cada nota era saldo por sí misma, y la de una factura todavía impaga reducía lo que se pagaba y además quedaba como saldo | **D-190:** la nota reduce lo que se debe de su factura. Solo lo pagado de más es saldo, que es del proveedor y por moneda (D-195) |
| D2 | **Una factura anulada por nota de crédito y re-emitida quedaba trabada.** La nota anulaba las unidades, y su VIN ya no se podía vincular ni volver a dar de alta | **D-191:** solo «anulación de unidad» anula la unidad. Con otro motivo, la unidad queda sin factura vigente |
| D3 | **Pagar en soles una factura en dólares no tenía regla** | **D-194:** cada aplicación guarda los dos montos. El costo no cambia; la diferencia de cambio va aparte |
| D4 | La nota que modifica varias facturas hasta 2026 esperaba al contador | **D-192:** cada línea copia la tasa de su factura, y desde el 01/01/2027 una nota descuenta una sola factura |
| D5 | La retención esperaba saber si Millennium es agente | **D-196:** se registra en la aplicación si el comprobante la trae; el cálculo, si hace falta, se agrega sin cambiar tablas |
| D6 | El canal de dealers esperaba al negocio | Ya estaba respondido: «sigue ocurriendo» (§10.5). **D-197:** `suppliers.is_dealer`, con detalle obligatorio y unidad saldada |
| D7 | A24: un pago consumía una sola nota; las tablas hijas no tenían empresa; la retención estaba en la factura | D-195, D-198 y D-196 |
| D8 | Los valores de una unidad se confundían: el valor de factura, el valor después de las notas y lo pagado | **D-199:** cinco montos, más el dinero entregado y la brecha de días |
| D9 | No estaba definido dónde viven las cargas (decisión 8 del mapa), ni si una carga con errores entra parcial (A-12 de la guía) | D-200 y D-201 |
| D10 | No estaba definido en qué estado entran las unidades migradas | **D-202:** el camino «Migración inicial», hacia Activo, Disponible o En habilitación |

### 2.9 Lo que se reutiliza

- **La columna de descuento de la orden**, de punta a punta: dominio, disparadores V69 y V70, y formulario. Es el patrón del interruptor del ISC.
- **Montos en enteros con `BigInt`** en el frontend.
- **Infraestructura del backend:**
  - `KeptFileStore` y la verificación de archivos de las plantillas;
  - `DocumentService`;
  - `ChangeLogCommandService`;
  - el patrón de RLS de V83 y las clases base de D-42;
  - `DeadlockRetry` y `UniqueViolations`;
  - `AccessScope.actsOn…`;
  - la siembra de V90.
- **En `fleet`:**
  - el VIN como objeto de valor y `stock.corregir_vin`;
  - las cesiones con `EXCLUDE gist`;
  - el motivo automático (`is_automatic`, D-110), que distingue el regreso de 48 h.
- **En `vehiclecatalog`:** GPS y PANTALLA con código fijo, y `has_factory_screen`.
- **En `location`:** las tasas con candado consultivo y la confirmación del salto grande.
- **En el frontend:**
  - el diálogo de tasa;
  - `ConfirmationDialog` y `UnsavedWork`;
  - `mm-data-manager`;
  - `mm-image-slot`;
  - las pipes de fecha.

---

## 3. Decisiones

### 3.1 Tomadas por el responsable el 02/11/2026

El texto completo de cada una está en el diccionario v1.38.

| N.º | Decisión |
|---|---|
| D-157 | Carga manual primero; el lector de XML vuelve a candidato |
| D-158 | ISC por factura (`has_isc_column`), en autos y motos, informativo |
| D-159 | Precio con impuestos por línea. El costo sin IGV se calcula. Al fondo se le reporta el valor de factura |
| D-160 | Reglas del registro: <ul><li>el número normalizado, único por proveedor y tipo entre las facturas no anuladas;</li><li>la fecha no es futura, y se guarda el día de la tasa;</li><li>hasta 100 líneas, que copian modelo, año, marca y empresa;</li><li>una sola línea vigente por unidad;</li><li>la ubicación llega con el ingreso;</li><li>el VIN, sin I, O ni Q.</li></ul> |
| D-161 | Lo fiscal no se edita: se anula con motivo y se usa «Anular y copiar». Anular no anula las unidades. La vía explícita de D-104 llega con este paso |
| D-162 | Una orden enviada a lo sumo. Lo pendiente se cuenta por modelo y año. Los avisos se confirman. La orden se cumple a mano |
| D-163 | Sin permisos nuevos. Exige la empresa y la marca de cada unidad. Excel en la primera versión |
| D-164 | `requires_enablement`: todo camino manual hacia Disponible exige *Listo para entrega* |
| D-165 | `company_id` en las tablas de habilitación; el bloqueo se lee de los datos; cuentan las placas físicas; el ingreso fija la ubicación; `current_mileage` se deduce |
| D-166 | El certificado de gas, según el combustible del modelo |
| D-167 | Los vencimientos se ven en Habilitación hasta el módulo de notificaciones |
| D-168 | La carga inicial trae las facturas, las notas y los datos de habilitación |
| D-169 | En la moneda funcional, tasa 1 con la fecha del documento |
| D-170 | `requires_enablement` queda fijo en Disponible |
| D-171 | Códigos estables del estado de habilitación |
| D-172 | Archivo y orden de una factura vigente: se cambian con `compras.registrar_factura`; vincular, solo a una orden enviada; desvincular, siempre |
| D-173 | Solo líneas de vehículo en la primera versión |
| D-174 | Archivos PDF, PNG, JPEG o WebP de hasta 5 MB, verificados por su contenido; el XML, de hasta 2 MB |
| D-175 | La anulación queda en la bitácora; Inventario filtra las unidades sin factura vigente |
| D-176 | Los permisos de facturas se siembran en dos tiempos |
| D-177 | Excepciones de la carga inicial, y perfilar antes los Excel reales |
| D-178 | La factura lleva `updated_at` y `updated_by`, con su candado. Cambiar y anular exigen el alcance de registrar. El `PUT` cambia el archivo, el XML y la orden |
| D-179 | El valor de factura es mayor que cero. Una unidad existente conserva su color y su motor, y el formulario avisa si difieren |
| D-180 | La exportación muestra la base y el IGV calculados; en una boleta, base igual al total e IGV en cero |
| D-181 | Tipo de tasa de compra: `venta` en Perú y `oficial` por omisión en los demás países |
| D-182 | Una unidad con una cesión abierta a un fondo no se anula |
| D-183 | La puerta exige que no falte nada: una unidad reacondicionada vuelve a Disponible |
| D-184 | La puerta se sostiene: en un estado marcado no se retira un requisito sin reemplazo. Agrega el camino «Pierde un requisito» (Disponible → En habilitación) y marca Vendido, que se puede desmarcar |
| D-185 | `equipment_types.has_installations` fija el código de un equipo en uso |
| D-186 | Ninguna fecha de la habilitación es futura, y una instalación no es anterior al ingreso |
| D-187 | El costo de una instalación lo escribe quien ve los costos |
| D-188 | Un documento que bloquea cuenta aunque haya vencido. Una unidad que ingresó no queda sin ubicación. Un país sin documento de identificación no registra placas |
| D-189 | Los archivos de facturas y de habilitación van a `kept_files` |
| D-190 | La nota reduce lo que se debe de su factura; solo lo pagado de más es saldo |
| D-191 | Solo «anulación de unidad» anula la unidad; con otro motivo, la unidad queda sin factura vigente |
| D-192 | Tasa por línea de la nota, copiada de su factura; desde 2027, una nota por factura |
| D-193 | La nota se registra a mano y se anula como la factura |
| D-194 | Pagos en cualquier moneda, con su equivalente; el costo no cambia |
| D-195 | Saldo a favor del proveedor, por moneda; un pago con saldo no apunta a una nota |
| D-196 | La retención se registra al pagar; no se calcula en la primera versión |
| D-197 | Canal de dealers: proveedor marcado, detalle obligatorio, unidad saldada |
| D-198 | Aplicaciones con empresa; `request_id`; traslado de la orden a la factura; anulación del pago |
| D-199 | Los cinco montos de una unidad |
| D-200 | Cada carga en el contexto dueño de lo que actualiza, registrada en `shared` |
| D-201 | El impuesto y la garantía se cargan por fila; la carga inicial, por factura |
| D-202 | La unidad migrada entra en su estado por «Migración inicial» |

### 3.2 Lo que sigue abierto

| A quién | Pregunta | Bloquea |
|---|---|---|
| Contador | Validar tres tratamientos: <ul><li>la nota reduce lo que se debe de su factura (D-190);</li><li>la tasa por línea de una nota de varias facturas (D-192);</li><li>la diferencia de cambio fuera del costo (D-194).</li></ul> Son prácticas estándar y ya están decididos | Nada; se valida en paralelo |
| Responsable | Si Millennium es agente de retención del IGV: la Consulta RUC de SUNAT lo dice | Nada: la primera versión solo registra la retención (D-196) |
| Negocio | Dos o tres facturas reales: una de auto y una de un lote de motos. Confirman el precio con IGV por línea, el VIN de 17 caracteres en motos y si traen cargos que no son vehículos (D-173) | El formulario (paso 5.10) |
| Negocio | La lista de modelos conciliada (diccionario §7.3) | Registrar facturas reales |
| Negocio | Los Excel reales de autos y de motos (Flota Lizzo New y Flota Lima Bikes), y un archivo real del impuesto vehicular y otro de la garantía mobiliaria, para perfilarlos (D-177) | El primer paso del paquete de cargas |
| Negocio | Qué proveedores son del canal de dealers, para marcarlos (D-197) | La carga de proveedores reales |

Las preguntas abiertas de las cinco especificaciones quedaron decididas el 03/11/2026:
- facturas y habilitación, con D-178 a D-189;
- notas de crédito, con D-203 a D-205;
- cargas de archivos, con D-206 a D-209;
- pagos, con D-210 a D-213.

---

## 4. Orden

**Decidido el 02/11/2026 y completado el 03/11/2026 con las cargas (D-208):**
1. fase 1 (punto 4);
2. fase 2 (punto 5);
3. fase 3 (punto 8);
4. cargas, primera entrega: el registro de cargas, el impuesto vehicular y la garantía mobiliaria;
5. fase 4 (punto 6), que habilita las facturas para uso real (D-176);
6. cargas, segunda entrega: la carga inicial;
7. fase 5 (punto 7);
8. cargas, tercera entrega: los pagos de la carga inicial;
9. la corrida de la carga inicial en producción, una sola vez y con la base respaldada.

**Por qué la habilitación va antes de las notas de crédito y los pagos:**
- cierra el flujo operativo de la compra a Disponible;
- no espera respuestas de afuera;
- las unidades que nacen con la factura necesitan pronto su ingreso físico.

**Las facturas no se usan con datos reales hasta tener las notas de crédito.** El mecanismo es D-176: hasta la fase 4, solo el superadministrador tiene los permisos de facturas.

**Qué puede ir en paralelo:**
- **Fases 2 y 3**, con dos excepciones:
  - el botón «Registrar ingreso de sus unidades» de la ficha de la factura (paso 8.7) espera la ficha (paso 5.11);
  - las dos fases tocan `fleet`, así que se integran con cuidado.
- **Fase 1**, junto a cualquiera.
- **La primera entrega de cargas, junto a la fase 4;** la segunda, junto a la fase 5. Cada una solo espera la fase que la habilita.
- **Numeración de reglas:** cada especificación tiene su rango, sin cruces:
  - **facturas:** PRC-R31 a R67 y FLT-R48 a R54;
  - **habilitación:** FLT-R55 a R93 y VCT-R28 a R31;
  - **notas:** PRC-R68 a R99 y FLT-R94 a R99;
  - **pagos:** PRC-R100 a R139 y FLT-R130 a R132;
  - **cargas:** PRC-R140 a R154, FLT-R100 a R127 y SHR-R01 a R13.
- **Numeración de migraciones:** se asigna al integrar en `develop`, después de V91, en el orden en que se integren.

---

## 5. Plan por fases

**Tamaños:**
- **S:** hasta un día.
- **M:** de dos a tres días.
- **L:** una semana o más.

Cada paso termina en uno o varios commits que compilan, con sus pruebas. **La secuencia de commits, el contrato, el esquema, las reglas y las pantallas de cada paso están en su especificación.**

### Fase 0 · Decisiones y especificaciones

**Hecho el 02/11/2026:**
- el diccionario v1.38, con D-157 a D-177;
- procesos P4 y P6;
- el orden de implementación;
- las convenciones de las correcciones delicadas;
- el contexto de flota;
- las dos especificaciones;
- el 03/11/2026, D-178 a D-189 y las revisiones 34 (facturas) y 35 (habilitación) de `DESIGN.md`, en el frontend;
- el mismo día, D-190 a D-213, las especificaciones de notas de crédito, pagos y cargas de archivos, y las revisiones 36 a 38 de `DESIGN.md`;
- el mapa de contextos, con sus decisiones 7 y 8 cerradas (D-203, D-200).

**Falta:** que el responsable valide los textos.

### Fase 1 · Punto 4: fechas en pantalla (frontend)

| Paso | Qué | Tamaño |
|---|---|---|
| 1.1 | Quitar `Date` de los valores de celda de `mm-data-manager` (`data-manager.ts:343`), para que no compile una columna que devuelva `Date` | S |
| 1.2 | Revisar, pantalla por pantalla, que cada momento se lea en la zona de la sede; si no hay sede, en la del país de la empresa; si no, en `America/Lima`. Incluye: <ul><li>el año por omisión del alta de unidad (`vehicle-form-dialog.ts:79`);</li><li>la zona de `correctedAt` en tipos de cambio.</li></ul> | M |
| 1.3 | Pruebas de guarda: <ul><li>**Escaneo estático.** Prohíbe `toLocaleDateString` y `toLocaleTimeString` en todo `src`. Prohíbe `DatePipe`, `formatDate(` y `provideNativeDateAdapter` fuera de `shared/presentation/pipes/` y de la vista del sistema de diseño. `toLocaleString('es-PE')` sobre números sigue permitido.</li><li>**Prueba e2e** con zona `Asia/Tokyo` y `en-US`, que compare con Lima.</li></ul> | M |
| 1.4 | Actualizar O70 y la fila «Fechas en pantalla» de procesos §4 | S |

**Aceptación:**
- el escaneo falla si se reintroduce `toLocaleDateString`;
- la e2e de Tokio muestra los mismos días que Lima.

**Fuera de la fase:** cambiar los 12 campos de fecha nativos por un selector propio es de formato, no de zona. Iría como un punto aparte.

**Hecho el 03/11/2026** (frontend `62b86d3b`), con la aceptación cumplida: 2 742 pruebas unitarias y la e2e de Tokio. El escaneo también prohíbe `| date` en las plantillas. La corrección de una tasa se lee en la zona de su país, la que fecha su día (D-156). Las tres diferencias menores que quedan están en los pendientes transversales del [orden de implementación](orden-de-implementacion-millennium-mobility.md).

### Fase 2 · Punto 5: facturas de compra

**Puerta:** el responsable valida la v1.38 y la [especificación de facturas](especificacion-facturas-de-compra-2026-11-02.md).

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 5.0 | Pasar el contrato de la especificación a las convenciones: rutas, cuerpos, códigos `PURCHASEINVOICE_*` y avisos que se confirman | S | 0 |
| 5.1 | **Permisos:** `compras.registrar_factura`, `registrar_factura_antigua`, `anular_factura` y `stock.anular_vehiculo`. Los de facturas, **solo al superadministrador** (D-176) | S | 0 |
| 5.2 | **`location`:** <ul><li>`fetchEffectiveRate` con «menor o igual»;</li><li>`countries.purchase_exchange_rate_type`;</li><li>la tasa de impuesto en `CountrySummary`;</li><li>el 409 de la tasa repetida.</li></ul> | M | 0 |
| 5.3 | **`fleet/interfaces/acl`:** <ul><li>alta y vínculo por lote;</li><li>`fetchUnitsByIds`;</li><li>`VehicleRegistrations` extraído;</li><li>los puertos que compras implementa: línea vigente de una unidad, para el bloqueo de modelo y año; valor de la unidad, para §9.6; unidades sin factura vigente, para D-175.</li></ul> **El alta, el vínculo y `fetchUnitsByIds` no esperan al esquema; las implementaciones de los puertos, del lado de compras, esperan el paso 5.5** | L | 5.1; 5.5 para los puertos |
| 5.4 | **Vía explícita de anulación de la unidad (D-104, D-161):** <ul><li>`Vehicle.annul`;</li><li>la guarda `422 VEHICLE_ANNULLED`;</li><li>`allowedActions` vacío;</li><li>`UnitContracts`, que responde «sin contratos» mientras ventas no exista;</li><li>una unidad con una cesión abierta no se anula (D-182).</li></ul> | M | 5.3 |
| 5.5 | **Esquema de compras:** <ul><li>`purchase_invoices` y `purchase_invoice_items` (D-157 a D-174);</li><li>los únicos parciales;</li><li>los disparadores del ISC;</li><li>RLS sin `DELETE`;</li><li>`CurrencyUsage`.</li></ul> | M | 0 |
| 5.6 | **Dominio y registro:** <ul><li>la suma exacta, el ISC, el formato del número en Perú, la fecha, los 20 días y las 100 líneas;</li><li>la tasa 1 en la moneda funcional (D-169);</li><li>el archivo verificado (D-174), subido después de validar y borrado si se revierte la transacción;</li><li>los avisos de la orden;</li><li>el puerto «dependientes de la factura», que responde «ninguno» hasta las fases 4 y 5.</li></ul> | L | 5.2, 5.3, 5.5 |
| 5.7 | **Anular y cambiar lo no fiscal:** <ul><li>la anulación, con motivo y su fila en la bitácora (D-175);</li><li>el `PUT` del archivo, del XML y de la orden, con `compras.registrar_factura` (D-172);</li><li>los dos llevan el `updated_at` leído y exigen el alcance de registrar (D-178).</li></ul> | M | 5.6 |
| 5.8 | **Consultas:** <ul><li>lista, ficha y `allowedActions`;</li><li>lo pendiente de la orden por modelo y año, con «todo facturado»;</li><li>la orden con facturas vigentes no se anula;</li><li>la exportación con las columnas de la especificación;</li><li>el filtro de Inventario «sin factura vigente».</li></ul> | M | 5.6 |
| 5.9 | **Frontend, base:** dominio, store, montos, el analizador del texto pegado y las fachadas | M | 5.0 |
| 5.10 | **Formulario**, según la revisión 34 de DESIGN, ya escrita. **Antes de empezar llegan las facturas reales (§3.2)** | L | 5.9 |
| 5.11 | **Lista, ficha, anulación y «Anular y copiar»;** las e2e y los mocks divididos por contexto | M | 5.10 |
| 5.12 | **Cierre:** <ul><li>los contextos de compras y de flota, con la decisión 5 del mapa cerrada y las reglas PRC y FLT en ✅;</li><li>P4 y este plan;</li><li>las sondas en vivo.</li></ul> | S | todo |

**Aceptación** (cada criterio tiene su prueba en la especificación):
- **Registro:**
  - una factura de 100 líneas se guarda en una transacción;
  - un fallo en la línea 100 no deja filas ni archivo;
  - el doble envío da 409;
  - el número de sentencias se mide con las estadísticas de Hibernate y la prueba lo fija.
- **Rendimiento:**
  - se mide primero la línea base;
  - 100 líneas en menos de 2 s en la base de verificación es el objetivo provisional.
- **Moneda funcional:** una factura en soles guarda tasa 1 y su fecha como día de la tasa.
- **Concurrencia y seguridad:**
  - si dos facturas registran el mismo VIN nuevo, una gana y no queda ninguna unidad huérfana;
  - un VIN de otra empresa da 409 sin revelar sus datos;
  - sin `stock.crear_vehiculo` funciona;
  - sin `compras.registrar_factura`, o fuera de la marca, da 403;
  - AE, OP y CO todavía no tienen el permiso (D-176).
- **Anular:**
  - anular deja la unidad sin factura vigente, y la factura nueva la vincula por su VIN;
  - la anulación queda en la bitácora;
  - cambiar el modelo de una unidad con línea vigente da 422;
  - el puerto de dependientes responde «ninguno».
- **Archivos:** un PDF renombrado `.png`, o uno de más de 5 MB, da 422.
- **Pantalla:**
  - el total cuadra al céntimo con 100 líneas pegadas;
  - axe no encuentra violaciones;
  - salir pregunta antes de descartar.
- **Integridad:** `ModularityTests`, RLS y filas que no se borran.

**Hecho el 03/11/2026** (backend `16f69af3`, con V92 a V94; frontend `2dc6505c`), con la aceptación cumplida salvo las sondas en vivo:
- **Pasos 5.0 a 5.11:** el contrato en las convenciones; los permisos (V92), el tipo de tasa de compra de cada país (V93) y las tablas de facturas (V94); la fachada de `fleet` y sus puertos; la anulación de la unidad; el registro, el cambio de archivo u orden, la anulación, las consultas y la exportación; y el frontend, con la lista, el formulario y la ficha. Las reglas PRC-R31 a PRC-R67, FLT-R48 a FLT-R54, LOC-R53 y LOC-R54 están en ✅, con el nombre real de cada prueba en la [especificación de facturas](especificacion-facturas-de-compra-2026-11-02.md) §5 y en los contextos.
- **Pruebas:** la suite completa del backend, 3 090 pruebas, con solo los fallos que se esperan con el superusuario sobre `erp_mm_db_verify`; las unitarias del frontend, y las e2e de facturas, órdenes de compra, flota y tipos de cambio.
- **Rendimiento:** la línea base, medida con la cuenta `postgres` y no con la de la aplicación, da medianas de 582 ms con 1 línea nueva, 916 ms con 10, 1 150 ms con 50 y 1 731 ms con 100 (máximo 2 063 ms), y 1 025 ms con 100 líneas que vinculan unidades existentes: la mediana de 100 líneas cumple el objetivo provisional de 2 s. Un registro prepara 57 sentencias además de sus inserciones, con 1, 10 y 100 líneas.
- **Lo que el código hizo distinto** queda en la versión 3 de la especificación: entre otras cosas, las filas de la lista son la ficha entera, y un `PUT` que manda la orden que la factura ya tiene no la toca, aunque esté cumplida.
- **Falta** (paso 5.12): las sondas en vivo, que necesitan la contraseña de verificación del responsable (especificación §7.4), y el texto de §2.7 del diccionario sobre el descarte de un archivo, que cambia solo con la aprobación del responsable.

### Fase 3 · Punto 8: habilitación

**Puerta:** el responsable valida la v1.38 y la [especificación de habilitación](especificacion-habilitacion-2026-11-02.md).

| Paso | Qué | Tamaño | Depende de |
|---|---|---|---|
| 8.0 | Pasar el contrato de la especificación a las convenciones | S | 0 |
| 8.1 | **Permisos:** `habilitacion.registrar_hitos`, `gestionar_documentos`, `gestionar_equipos` y `gestionar_llaves`, y los tres de los tipos de documento | S | 0 |
| 8.2 | **Esquema:** <ul><li>las cinco columnas de §11.1;</li><li>`requires_enablement`, fijo en DISPONIBLE (D-170) y marcado también en VENDIDO, donde se puede desmarcar (D-184);</li><li>el camino «Pierde un requisito», de Disponible a En habilitación (D-184);</li><li>`equipment_types.has_installations`, en `vehiclecatalog` (D-185);</li><li>`vehicle_document_types`, con la semilla de Perú;</li><li>`vehicle_documents`, `vehicle_equipments` (con su costo y moneda) y `vehicle_keys`, con `company_id`, RLS y sin `DELETE`.</li></ul> | M | 8.1 |
| 8.3 | **`vehiclecatalog`:** <ul><li>una fachada por lote con la pantalla requerida, los equipos que aplican y el combustible;</li><li>`markEquipmentTypeInUse`.</li></ul> | S–M | 8.2 |
| 8.4 | **Comandos:** <ul><li>el ingreso por unidad o en lote, que fija la ubicación;</li><li>los hitos, la placa y las placas físicas;</li><li>los documentos verificados (D-174), con el certificado de gas solo para GNV o GLP;</li><li>los equipos, con sus períodos, su costo y su moneda (`CurrencyUsage`; el costo se ve con `stock.ver_costos`);</li><li>las llaves.</li></ul> | L | 8.2, 8.3 |
| 8.5 | **`EnablementPolicy`:** <ul><li>seis filas con sus códigos (D-171);</li><li>se aplica en todo camino manual hacia un estado con `requires_enablement`, pero no en el automático, y deja pasar una unidad a la que no le falta nada, esté Lista o Entregada (D-183);</li><li>en un estado marcado no deja retirar un requisito sin reemplazo (D-184);</li><li>se refleja en `allowedActions` y en una proyección SQL para listar y filtrar.</li></ul> | M | 8.4 |
| 8.6 | **Frontend:** <ul><li>Catálogos › Tipos de documento;</li><li>la casilla `requires_enablement` en Catálogos › Estados, desactivada en Disponible.</li></ul> | M + S | 8.2 |
| 8.7 | **Logística › Habilitación:** <ul><li>la lista con su código y su detalle;</li><li>el filtro «vence en 30 días»;</li><li>la exportación;</li><li>el ingreso en lote.</li></ul> **El botón en la ficha de la factura espera el paso 5.11** | M | 8.5 |
| 8.8 | La página de habilitación de la unidad, según la revisión 35 de DESIGN, ya escrita | L | 8.5 |
| 8.9 | **Cierre:** <ul><li>el contexto de flota, con las reglas en ✅;</li><li>P6 y este plan;</li><li>las sondas en vivo.</li></ul> | S | todo |

**Paso 8.0 hecho el 03/11/2026:** el contrato de la [especificación de habilitación](especificacion-habilitacion-2026-11-02.md) §4 está en las convenciones (§2.2, §3, §3.2, §4.1, §4.2, §6.2, §7.1, §7.2, §8.1 y §9.2), y F8 a F12, FLT-R55 a FLT-R93 y VCT-R28 a VCT-R31 están en los contextos de flota y de catálogos de flota, en ❌ hasta que se construyan.

**Aceptación** (cada criterio tiene su prueba en la especificación):
- **Estado:**
  - una prueba pura fija el orden de las seis filas y sus códigos;
  - la entrada manual a Disponible se rechaza, por «Habilitación completa» y por «Reacondicionada», si falta la TIVE, la pantalla cuando corresponde, el GPS o las placas físicas;
  - el regreso automático de Vendido no se bloquea.
- **La puerta:**
  - desmarcar `requires_enablement` en Disponible da 422;
  - retirar el GPS de una unidad Disponible sin reemplazo da 422, y la unidad puede salir por «Pierde un requisito»;
  - una unidad entregada y reacondicionada vuelve a Disponible.
- **Concurrencia:** retirar el GPS mientras otra persona habilita no deja una unidad Disponible sin GPS.
- **Operación:**
  - no se instala nada antes del ingreso;
  - 80 motos ingresan en una operación, con su ubicación;
  - el filtro de vencimientos muestra exactamente los de los próximos 30 días, en la zona de la empresa;
  - el certificado de gas solo se pide a modelos a GNV o GLP.

### Cargas, primera entrega · Registro, impuesto vehicular y garantía mobiliaria

**Puerta:**
- la fase 3 integrada;
- la [especificación de cargas](especificacion-cargas-de-archivos-2026-11-03.md) validada;
- un archivo real del impuesto vehicular y otro de la garantía, perfilados.

| Paso | Qué | Tamaño |
|---|---|---|
| C1.0 | Perfilar los archivos reales del impuesto vehicular y de la garantía con su lista de comprobaciones. Lo que encuentre ajusta el formato de §5 de la especificación antes de construir | M |
| C1.1 | `shared`: `import_batches`, su fachada, el informe de errores en `kept_files` y la pantalla Logística › Cargas de archivos, con `habilitacion.ver_cargas` (D-200) | M |
| C1.2 | `fleet`: la carga del impuesto vehicular por placa, fila por fila, solo autos, sin crear unidades (§11.7, D-201), con `habilitacion.cargar_archivo_sat` | M |
| C1.3 | `fleet`: la carga de la garantía, que actualiza la historia de cesiones y cierra las que no figuran, también las registradas a mano (D-30, D-209), con `fondos.cargar_archivo_garantia` | M |
| C1.4 | Cierre: contextos, P6, este plan y sondas en vivo | S |

**Aceptación:**
- una fila sin unidad va al archivo de errores y las demás entran;
- volver a subir el mismo archivo no repite nada;
- una placa cedida a otro fondo falla sin tocar nada;
- el informe de errores se descarga con su permiso.

### Fase 4 · Punto 6: notas de crédito

**Puerta:**
- la fase 2 integrada;
- la [especificación de notas](especificacion-notas-de-credito-2026-11-03.md) validada.

| Paso | Qué | Tamaño |
|---|---|---|
| 6.0 | El contrato pasa a las convenciones | S |
| 6.1 | El permiso `compras.registrar_nota_credito` | S |
| 6.2 | **Esquema:** <ul><li>las notas y sus líneas, con la tasa por línea (D-192), la anulación y `updated_at` (D-193);</li><li>RLS sin `DELETE`;</li><li>el disparador que deja volver una línea de factura a vigente.</li></ul> | M |
| 6.3 | **El agregado:** <ul><li>motivos (D-204);</li><li>la línea que llega a cero se anula;</li><li>el corte del 01/01/2027, con la fecha configurable (D-192).</li></ul> | M |
| 6.4 | **`FleetContextFacade`:** <ul><li>anula la unidad devuelta (D-191);</li><li>marca «Devuelta al proveedor» la unidad con contratos o con una cesión abierta (D-204).</li></ul> | S |
| 6.5 | **El registro en una transacción:** <ul><li>reduce lo que se debe (D-190);</li><li>el puerto de dependientes, para que una factura con notas no se anule;</li><li>`valueIsZero`.</li></ul> | L |
| 6.6 | **Anulación y archivo:** <ul><li>la anulación de la nota, con la guarda del saldo que implementa la fase 5;</li><li>el cambio de su archivo (D-204).</li></ul> | M |
| 6.7 | **Consultas:** lista, ficha, exportación, y el panel de notas en la ficha de la factura | M |
| 6.8 | **El costo de compra en la ficha de la unidad** (`GET /vehicles/{id}/purchase-cost`, D-203) | M |
| 6.9 a 6.11 | **Frontend:** formulario con «repartir en partes iguales», lista, ficha y los paneles de la factura y de la unidad (DESIGN, revisión 36) | M + L |
| 6.12 | **Los permisos de facturas en AE, OP y CO:** desde aquí, las facturas se usan con datos reales (D-176) | S |
| 6.13 | **Cierre:** contextos, P4, este plan y sondas | S |

**Aceptación:**
- **Valor:**
  - una nota reduce el valor de cada unidad que nombra, y nunca baja de cero;
  - una nota de varias facturas guarda la tasa de cada una;
  - una nota con fecha desde el 01/01/2027 que nombra dos facturas da 422.
- **Motivos:**
  - con «anulación de unidad», la unidad se anula, o queda «Devuelta al proveedor» si tiene contratos o una cesión abierta;
  - con «anulación de la factura», la unidad queda sin factura vigente, y la factura re-emitida la vincula.
- **Anulación:**
  - anular una factura con notas da 422;
  - anular una nota devuelve sus líneas a vigente, salvo que su unidad esté anulada.
- **Costo:** el costo en la ficha de la unidad se ve solo con `stock.ver_costos`.

### Cargas, segunda entrega · La carga inicial

**Puerta:**
- la fase 4 integrada;
- los Excel reales de autos y de motos, perfilados.

| Paso | Qué | Tamaño |
|---|---|---|
| C2.0 | **Perfilar los Excel reales** con las comprobaciones de la especificación: <ul><li>columnas;</li><li>filas sin proveedor o sin modelo;</li><li>notas de débito;</li><li>tasas distintas para un mismo día;</li><li>propietarios distintos de la empresa;</li><li>fechas que habrá que deducir.</li></ul> | M |
| C2.1 | **`procurement` orquesta, por factura** (D-200, D-201): <ul><li>la factura y sus unidades;</li><li>las notas, como «descuento de campaña» (D-207);</li><li>la habilitación, con la TIVE y el SOAT sin archivo y el GPS sin serial, rotulados (D-206);</li><li>el estado de cada unidad por «Migración inicial» (D-202).</li></ul> | L |
| C2.2 | **Las fechas deducidas, las llaves con etiqueta provisional y el perfil de lo deducido** (D-206) | M |
| C2.3 | **Pantallas:** el diálogo de carga, la ficha del lote y el rótulo «Migrado, sin archivo» (DESIGN, revisión 38) | M |
| C2.4 | **Ensayo** con los Excel reales sobre la base de verificación, aprobado por el responsable | S |

**Aceptación:**
- las filas de una misma factura entran o fallan juntas;
- un VIN ya migrado se informa y no se repite;
- una unidad sin requisitos no entra a Disponible aunque el Excel diga «listo» (D-183);
- el informe cuenta cada dato deducido.

### Fase 5 · Punto 7: pagos a proveedores

**Puerta:**
- la fase 4 integrada;
- la [especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md) validada.

| Paso | Qué | Tamaño |
|---|---|---|
| 7.0 | El contrato pasa a las convenciones | S |
| 7.1 | Los permisos `pagos.*` | S |
| 7.2 | `suppliers.is_dealer`, en el esquema, la API y la pantalla (D-197) | S |
| 7.3 | **Esquema:** <ul><li>los pagos, con `request_id`, `exchange_rate_date` y `updated_at`;</li><li>las aplicaciones, con su tasa, su equivalente y la retención;</li><li>las imputaciones, con `settles_unit`;</li><li>todo con `company_id` y RLS.</li></ul> | M |
| 7.4 | **El dominio:** <ul><li>lo que se debe y el saldo del proveedor por moneda (D-190, D-195);</li><li>el pago aplicado completo (D-210);</li><li>el reparto y el tope por unidad (D-211).</li></ul> | M |
| 7.5 | **El registro:** <ul><li>la idempotencia;</li><li>las tasas por aplicación;</li><li>el pago con saldo, con el proveedor bloqueado;</li><li>los puertos de dependientes y de la guarda de la fase 4.</li></ul> | L |
| 7.6 | **La anulación del pago** (D-198, D-212) | M |
| 7.7 | **La orden:** el paso de la orden a la factura, y la orden con pagos, que no se cumple ni se anula (D-211) | M |
| 7.8 | **Consultas:** <ul><li>lista, ficha y exportación;</li><li>el saldo del proveedor y lo que se debe de cada factura;</li><li>la diferencia de cambio (D-213).</li></ul> | M |
| 7.9 | **`fleet`:** el estado de pago, visible sin montos con `stock.ver` (D-212), y la brecha de días en la ficha de la unidad | S |
| 7.10 y 7.11 | **Frontend:** base, lista, formulario con aplicaciones e imputaciones, ficha, y los paneles de la factura, el proveedor y la unidad (DESIGN, revisión 37) | M + L |
| 7.12 | **Cierre:** contextos, P5, este plan y sondas | S |

**Aceptación:**
- **Saldo (D-190):**
  - un bono antes del pago deja lo que se debe en el neto y no crea saldo;
  - un bono después del pago crea saldo, que paga la factura siguiente sin mover dinero.
- **Moneda:** una factura en dólares pagada en soles guarda la tasa de la aplicación, y su costo no cambia.
- **Aplicación:**
  - un pago que no se aplica completo da 422;
  - el doble envío se guarda una sola vez.
- **Canal de dealers:** sin detalle por unidad da 422, y una unidad saldada no recibe más pagos.
- **Anulación:** anular un pago que dejaría negativo el saldo da 422.
- **Unidad:** la unidad muestra sus cinco montos, el dinero entregado y la brecha de días desde el primer pago.

### Cargas, tercera entrega y corrida en producción

| Paso | Qué | Tamaño |
|---|---|---|
| C3.1 | **Los pagos de la carga inicial** (D-207): <ul><li>uno por bloque de la columna 42, por transferencia;</li><li>con tasa 1 en la moneda funcional o, en otra, la del catálogo o la del Excel;</li><li>en el canal de dealers, imputados por unidad y saldados.</li></ul> | M |
| C3.2 | **Ensayo completo** de las tres entregas con los Excel reales, aprobado por el responsable | S |
| C3.3 | **La corrida en producción.** No es un commit. Una sola vez, con la base respaldada, por orden del responsable (D-208) | S |

### Esfuerzo estimado

La estimación es gruesa: S = 1 día, M = 2,5 y L = 5, y los tamaños no se han calibrado.

| Tramo | Días de una persona |
|---|---|
| Fase 1 | 7 |
| Fase 2 | 35 |
| Fase 3 | 26 |
| Cargas, primera entrega | 12 |
| Fase 4 | 30 |
| Cargas, segunda entrega | 15 |
| Fase 5 | 30 |
| Cargas, tercera entrega y corrida | 4 |
| **Total** | **unos 160 días: 32 semanas con una persona, o unas 19 con dos** (con las fases 2 y 3 en paralelo, y cada entrega de cargas junto a la fase siguiente) |

---

## 6. Riesgos

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Las facturas reales no traen lo supuesto: el precio por línea, el VIN de las motos, otros cargos | Media | Alto | Pedirlas antes del paso 5.10. Si traen cargos, se habilitan las líneas de concepto (D-173) |
| Faltan los modelos reales conciliados | Alta | Alto | Pedir la lista y cargarla antes de cerrar la fase 2 |
| Los Excel reales tienen datos que la carga no admite | Alta | Alto | Perfilarlos primero; lo que no entra va al archivo de errores (D-177) |
| El lote de 100 unidades rinde mal o deja escrituras a medias | Media | Alto | Línea base, estadísticas de Hibernate y una prueba de fallo en la última línea |
| El atajo del permiso de alta abre una escalada | Baja | Alto | Un comando aparte, con pruebas de 403 |
| Carreras del VIN y del número | Media | Medio | Únicos parciales, orden de candados y pruebas de carrera |
| Las facturas se usan antes de tener las notas de crédito | Baja | Medio | D-176 |
| Las fases 2 y 3 chocan en `fleet` | Media | Medio | Integrar seguido; las migraciones se numeran al integrar |
| El contador objeta un tratamiento ya decidido (D-190, D-192, D-194) | Baja | Medio | Son prácticas estándar; validarlos antes de la fase 4 |
| Un archivo grande pasa el límite de tiempo de Cloudflare | Media | Medio | Un tope de filas medido en la línea base; un archivo más grande se parte (D-208) |
| SUNAT vuelve a mover la fecha de una nota por comprobante | Media | Bajo | El corte en una política con la fecha configurable |
| Las imágenes de referencia e2e cambian mucho | Alta | Bajo | Mover primero las pruebas «En construcción» |

---

## 7. Cómo se construye, se verifica y se despliega

- **Ramas y entornos:**
  - una rama por fase desde `develop`: `feature/screen-dates`, `feature/procurement-invoices` y `feature/fleet-enablement`;
  - el trabajo se hace en worktrees hermanos, porque el responsable corre la aplicación desde los repositorios principales;
  - commits pequeños con Conventional Commits, cada uno compilando;
  - se publica solo por orden del responsable, porque `develop` despliega.
- **Pruebas:**
  - las que alcanza cada cambio, y siempre `ModularityTests`;
  - con tablas nuevas, también las de RLS y de filas que no se borran;
  - las suites completas antes de integrar en `develop`;
  - las de integración y las sondas en vivo sobre la base de verificación, que crea el responsable;
  - las e2e en el puerto 4300, con el equipo enchufado.
- **Despliegue:**
  - las migraciones solo agregan;
  - se respalda la base antes de desplegar cada fase;
  - un error se corrige hacia adelante, nunca editando una migración aplicada;
  - las facturas quedan solo para el superadministrador hasta la fase 4 (D-176).
- **Documentación:**
  - cada fase cierra actualizando su contexto, sus procesos, su especificación y este plan, citando commits y fechas;
  - las reglas pasan de ❌ a ✅ con su prueba.
- **Siguiente paso:** el responsable valida el texto de la v1.38, las cinco especificaciones y las revisiones 34 a 38 de `DESIGN.md`. Con eso empiezan la fase 1 y la fase 2.
