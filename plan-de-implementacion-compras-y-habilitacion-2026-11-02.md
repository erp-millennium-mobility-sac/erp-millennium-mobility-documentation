# Auditoría y plan de implementación: de la factura de compra a la habilitación, las notas, los pagos y las cargas

**Versión 4 · 03/11/2026.** Cierra de punta a punta todo lo que planifica: las fases 1 a 5 y el paquete de cargas de archivos, cada uno con su especificación. **Avance al 03/11/2026:** las fases 1 a 5 están construidas e integradas en `develop` (§5, el «Hecho» de cada una): lo que esta versión implementa está construido de punta a punta, y la fase 5, los pagos a proveedores, cerró el 03/11/2026 (backend `601cc177`, frontend `e0ed0f68`). El 03 y el 04/11/2026, una revisión de punta a punta encontró lo que el código hacía distinto de las reglas, y sus correcciones se integraron en `develop` el 04/11/2026: backend `44136ae1` y frontend `cc782649` (§2.10). El mismo día el responsable aprobó las propuestas para el diccionario de los cierres de las fases 4 y 5 y de la revisión, que la documentación `7cc29f4` aplica en el diccionario v1.39, y entre ellas R22, la fecha base de la brecha de cada unidad, integrada con el backend `1bd767f1` (§2.10). También el 04/11/2026, una segunda revisión de punta a punta, con cuatro revisores independientes, y tres pedidos del responsable —la barra de las listas con varios filtros, la importación de las líneas de una factura desde Excel y la tarjeta de datos en grupos— se integraron en `develop`: backend `851b4d09` y frontend `3a49f141` (§2.11). Faltan las sondas en vivo de las fases 2 a 5, que necesitan la contraseña de verificación del responsable, y las propuestas R23 a R25 para el diccionario, que decide el responsable. El paquete de cargas sigue pospuesto por decisión del responsable del 03/11/2026 («Aún no haremos cargas iniciales, solo implementación», §4).

**Cómo se llegó aquí:**
- **Versión 1:** auditó tres puntos de partida, el backend `b1264ed5` (hasta V91), el frontend `88312e13` y la documentación `6ebb900`, junto con una investigación de ERP comparables y de la norma de SUNAT.
- **Versión 2:** sumó una auditoría estricta del plan, la revisión de las fuentes del cliente (la guía funcional v2.0 y el flujograma maestro de estados) y las decisiones D-157 a D-168.
- **Versión 3:**
  - cierra los hallazgos de la auditoría de la versión 2 con D-169 a D-177, en el [diccionario](diccionario-de-dtos-millennium-mobility.md) v1.38;
  - agrega las especificaciones listas para implementar: [facturas de compra](especificacion-facturas-de-compra-2026-11-02.md) y [habilitación](especificacion-habilitacion-2026-11-02.md);
  - el 03/11/2026 cierra, con D-178 a D-189, las preguntas abiertas que dejaron esas especificaciones.
- **Versión 4:**
  - audita las fases 4 y 5 y el paquete de cargas, y los cierra con D-190 a D-202 (§2.8);
  - agrega sus especificaciones: [notas de crédito](especificacion-notas-de-credito-2026-11-03.md), [pagos a proveedores](especificacion-pagos-a-proveedores-2026-11-03.md) y [cargas de archivos](especificacion-cargas-de-archivos-2026-11-03.md);
  - el 04/11/2026 registra la revisión de punta a punta de las fases construidas y sus correcciones (§2.10);
  - el mismo día registra las propuestas para el diccionario que aprobó el responsable, aplicadas en la v1.39 (documentación `7cc29f4`), y R22, la fecha base de cada unidad (backend `c43d1bc1`, merge `1bd767f1`; §2.10, §3.2);
  - y registra la segunda revisión de punta a punta y sus correcciones, con los tres pedidos del responsable y el contrato C10 (backend `851b4d09`, frontend `3a49f141`; §2.11).

**Cómo leerlo.**
- **§1** fija el alcance.
- **§2** resume los hallazgos de las auditorías y de las dos revisiones de punta a punta (§2.10 y §2.11), y cómo quedó cada uno.
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

### 2.10 Revisión de punta a punta y correcciones (03 y 04/11/2026)

Con las fases 1 a 5 integradas, el responsable pidió una revisión crítica de punta a punta de esta versión, y el 04/11/2026 ordenó «Resuelve todos los hallazgos como experto». Cuatro revisores leyeron el código, uno por fase (2 a 5), y cada hallazgo clave se verificó en el código antes de corregirlo. Las correcciones se integraron en `develop` el 04/11/2026, sin migraciones:
- **backend** `44136ae1`, la rama `feature/review-findings`, 23 commits de `5c77bc8f` a `df6b7276` sobre `059a4457`;
- **frontend** `cc782649`, la rama del mismo nombre, 35 commits de `6386f2fa` a `1ef10642` sobre `e0ed0f68`, con la revisión 39 de `DESIGN.md`;
- después, el mismo día, dos textos: el backend cuenta en `UserQueryServiceImpl` y `SessionCompanyResource` los once permisos de accesos y roles que la sesión deja fuera, y no ocho (`728dbc52`, merge `63a56051`), y el rechazo del canal de dealers pide pasar los pagos de las órdenes a sus facturas o anularlos, nunca anular las órdenes (frontend `89402f13`, merge `a1956861`);
- y R22, la fecha base de cada unidad, con la aprobación del responsable: backend `c43d1bc1`, merge `1bd767f1`.

Cada especificación tiene su versión 4, con la marca **Como se corrigió (04/11/2026)** donde cambió el comportamiento y su §10 con el detalle de su fase: [facturas](especificacion-facturas-de-compra-2026-11-02.md), [habilitación](especificacion-habilitacion-2026-11-02.md), [notas](especificacion-notas-de-credito-2026-11-03.md) y [pagos](especificacion-pagos-a-proveedores-2026-11-03.md).

**Lo que se comprobó de punta a punta y estaba bien.**
- Row Level Security sobre `erp_mm_db_verify`, con `SET ROLE erp_mm_app` en una transacción revertida (04/11/2026): las 11 tablas nuevas de empresa la tienen forzada, ninguna tiene `DELETE`, `payment_allocations` no tiene `UPDATE`, y una empresa no ve las filas de otra.
- La matriz de permisos es la del diccionario §5.6.
- El menú y las rutas, la fase 1, la configuración y el almacén de archivos que no vencen.

**Severidades.** La revisión clasificó como alto el redondeo de la conversión de los pagos. El resto lo clasifica este cierre por su efecto, con la escala de §2.

**Las decisiones.** C1 a C8 las tomó la coordinación con la orden del responsable, y las dos ramas las siguieron. R22 la aprobó el responsable el 04/11/2026, con las demás propuestas para el diccionario (abajo):

| N.º | Decisión |
|---|---|
| C1 | El equivalente de una aplicación absorbe el redondeo, para que un pago en otra moneda cuadre |
| C2 | Los céntimos del reparto entre unidades van de a uno desde la primera, sin bajar de cero ni pasar lo que a una unidad le falta |
| C3 | La habilitación de una unidad y su lista dicen el código del estado actual y si tiene puerta (`currentStatusCode`, `currentStatusRequiresEnablement`) |
| C4 | Cuatro códigos `422` nuevos: `STATUSCHANGEREASON_AUTO_TARGET_GATED`, `VEHICLESTATUS_ENABLEMENT_AUTO_TARGET`, `VEHICLESTATUS_ENABLEMENT_INITIAL` y `SUPPLIER_DEALER_ORDER_PAYMENTS`; y `PURCHASEINVOICE_TOO_OLD` responde también la revisión por marca |
| C5 | Lo que se puede pagar deja fuera una factura de dealers con todas sus unidades vigentes saldadas |
| C6 | La bitácora oculta el costo y la moneda de una instalación sin `stock.ver_costos` en la marca de la unidad, y el monto de una aplicación sin `pagos.ver` |
| C7 | La búsqueda de facturas y notas encuentra el número escrito como un número de comprobante, normalizado |
| C8 | La sesión dice, por empresa, qué permisos actúan en toda la empresa (`wholeCompanyPermissions`) y cuáles en una marca (`brandPermissions`) |
| R22 | La brecha de días de cada unidad cuenta desde el primer pago que la cubre —por su imputación, o por su parte de lo que su factura no imputó— o, sin ninguno, desde su factura, y no desde el primer pago aplicado a la factura (guía, columnas 24 y 43): de 50 motos pagadas en dos bloques de 25, cada bloque cuenta desde su propio pago. La posición de la factura conserva su primer pago. D-199 y §10.5 del diccionario v1.39 lo dicen. Backend `c43d1bc1`, merge `1bd767f1`; el frontend no cambió |

**Lo que no cambió, por decisión.**
- **El subtotal de la orden como tope de un pago:** D-211 dice que no lo es, porque puede no llevar el IGV.
- **El agrupamiento global de JDBC:** el ingreso de 80 unidades cumple su meta de 2 s. Se corrigió el texto, que decía otra cosa: las lecturas no crecen con las unidades, y cada actualización es su propia sentencia (fase 3, abajo).

**Fase 2 · Facturas de compra**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Alto | Un `PUT` revertido que mandaba los mismos bytes o un archivo anterior descartaba de Spaces el archivo de una factura confirmada. Valía también para el archivo de una nota y el de un documento de una unidad (B10) | `KeptFileStore.put` dice si la transacción escribió la llave primero (`RETURNING (xmax = 0)`), y solo entonces se registra el descarte. Queda la carrera de los mismos bytes a la vez, documentada en cada especificación | Backend `7ad20a4a` |
| Medio | Los formularios ofrecían las empresas donde un rol de sede tenía el permiso, y el backend rechazaba el registro; no podían retener las líneas de una marca que el permiso no alcanza (B21) | C8; los tres formularios ofrecen solo las empresas donde se registra sin sede, lo dicen cuando no hay ninguna y retienen esas líneas | Backend `df6b7276`; frontend `3742271f`, `fdf14036`, `18e03774`, `5fad3ceb`, `c8b1d359`, `1ef10642` |
| Medio | «Orden de compra» leía solo las 100 órdenes enviadas más recientes, y los selectores de proveedor, una página | Se buscan en el servidor; lo que una copia o un enlace nombra se lee por su id | Frontend `6c779bc3`, `3a995e95`, `9f011f06` |
| Medio | Sin los países leídos, el formulario tomaba `oficial` como tipo de tasa | No propone ninguna tasa: lo dice y ofrece «Reintentar» | Frontend `add2c336` |
| Medio | «Cambiar archivo u orden» seguía, después de `PURCHASEINVOICE_OUTDATED`, con la factura con que se abrió | Toma la leída otra vez y conserva los archivos elegidos | Frontend `ea714437` |
| Bajo | El permiso de facturas antiguas que no alcanzaba la marca de una línea respondía `403` | `422 PURCHASEINVOICE_TOO_OLD`, como la fecha (C4) | Backend `5f0a01b1` |
| Bajo | Los años que una factura trae a una línea de la orden sin año se comparaban uno por uno (B12) | Se suman antes de comparar | Backend `10850e34` |
| Bajo | Las líneas que vinculan unidades existentes las leían una por una: las sentencias crecían con ellas (B15) | Se leen y se bloquean en una sentencia (`VehicleRepositoryImpl.readAllLocked`) | Backend `61ebde79` |
| Bajo | Una línea de concepto, que la base admite, respondía `500` (B17) | Se lee y las reglas de las unidades la pasan por alto; construirla sigue sin soporte (D-173) | Backend `a7afefd8` |
| Bajo | La búsqueda no encontraba el número escrito con ceros o separadores | C7 | Backend `53f11cd3` |
| Bajo | La pantalla rechazaba ceros más allá de los decimales de la moneda; «Pegar columnas» dejaba cada línea sin marca, modelo ni año | Los acepta; «Para todas las líneas (opcional)», revisión 39 de `DESIGN.md` | Frontend `6386f2fa`, `8be386af` |
| Bajo | La especificación ponía las marcas de las unidades nuevas en el segundo candado y la orden en `FOR UPDATE` | Solo el texto: las marcas van al final, `FOR SHARE`, después de la orden (`FOR NO KEY UPDATE`) y de las unidades existentes; anular una marca bloquea solo la marca, así que no hay ciclo | — |

**Fase 3 · Habilitación**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Alto | La puerta de Disponible se podía apagar desde el catálogo: un camino que mueve solas las unidades a un estado marcado, o un estado inicial marcado | C4: `VEHICLESTATUS_ENABLEMENT_INITIAL`, `VEHICLESTATUS_ENABLEMENT_AUTO_TARGET` y `STATUSCHANGEREASON_AUTO_TARGET_GATED`; solo el regreso desde Vendido mueve sola una unidad a un estado marcado (`AutomaticReturnGate`), y el trabajo deja donde está, con un aviso `WARN`, una unidad incompleta en un camino guardado antes | Backend `345c1f75`; frontend `7d5e5dce`, `660d15dc`, `891f3fa0`, `bdcf9e5e` |
| Alto | V96 se detiene si alguna unidad tiene placa, y no se sabía qué tiene la base desplegada (B18) | `scripts/database/inventory-plated-units.sql` y la sección «Inventario antes de V96» del README del backend, que el responsable corre antes de que `develop` despliegue V96 | Backend `7bc29d52` |
| Medio | «Editar hitos» y el ingreso de una unidad seguían, después de `VEHICLE_OUTDATED`, con lo que se abrió: un reintento podía pisar el cambio de otra persona | Toman la unidad leída otra vez | Frontend `ea714437` |
| Medio | La página y la lista de la habilitación leían `GET /vehicle-statuses`, que pide `catalogos.ver`, y fallaban a un rol de `stock.ver` | C3; el filtro de estado de la lista se muestra solo con `catalogos.ver` | Backend `4aa250b9`; frontend `ab67dce4` |
| Medio | El costo de una instalación se leía y se escribía con `stock.ver_costos` en la empresa, no en la marca de la unidad (D-187), y la bitácora lo mostraba a quien tenía `bitacora.ver` | Por la marca (`VehicleAccessPolicy.seesCosts`), y C6 | Backend `f5eacfe7`, `ad87b4d9`; frontend `ab67dce4`, `c510e131` |
| Bajo | El descarte del documento de una unidad bajaba el objeto entero para decidir | Lo decide `KeptFileStore.put` (B10) | Backend `7ad20a4a` |
| Bajo | El diálogo del equipo perdía un costo escrito sin moneda; la nota de un equipo retenido ofrecía «Pierde un requisito» fuera de Disponible | Pide la moneda; lo ofrece solo en Disponible | Frontend `311dc136`, `ab67dce4` |
| Bajo | La especificación y este plan decían «lotes de JDBC» y «las mismas sentencias con 1, 10 y 80» | Solo el texto: el backend no configura lotes | — |

**Fase 4 · Notas de crédito**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Alto | El cambio del archivo de una nota, revertido, podía descartar un archivo confirmado (B10, arriba) | La misma corrección | Backend `7ad20a4a` |
| Medio | «Agregar unidades» leía solo las 100 primeras facturas del proveedor | Las busca en el servidor por su número, las más recientes primero | Frontend `103cf5cf` |
| Bajo | «Devuelta al proveedor» seguía en una unidad que una factura vigente volvió a vincular (B16) | Se deja fuera | Backend `ab8ec0ff` |
| Bajo | Sin la regla del corte leída, el formulario seguía como si no hubiera corte | Espera, con «Reintentar» | Frontend `8c7c88ff` |
| Bajo | La especificación citaba clases que no existen y la prueba que V104 retiró | Solo el texto | — |

**Fase 5 · Pagos a proveedores**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Alto | Un pago en soles de una factura en dólares cuadraba solo con cerca del 27 % de los montos en soles: los demás no daban la suma exacta que exige D-210 (B1) | C1: el equivalente absorbe el redondeo (`PaymentEquivalents`). Con el ejemplo del voucher, S/ 10 000,01 se rechaza y S/ 10 000,00 se registra, con diferencia de cambio 47,89 | Backend `5c77bc8f`; frontend `20cebe6f`, `af8a3f16` |
| Medio | Un reintento no comparaba la retención, el comprobante, las imputaciones ni el detalle, y uno legítimo respondía `409` después de que la aplicación a la orden pasara a la factura | Compara todo el contenido, y la orden de donde vino cada aplicación | Backend `bb355c8e` |
| Medio | Los céntimos del reparto podían dejar una unidad bajo cero | C2 | Backend `46f207b1`; frontend `f8efb41b` |
| Medio | Un proveedor con pagos en sus órdenes se podía marcar del canal de dealers | `422 SUPPLIER_DEALER_ORDER_PAYMENTS` (C4) | Backend `06faa05b`; frontend `7d5e5dce` |
| Medio | La bitácora mostraba el monto que una aplicación deja en la orden sin `pagos.ver` | C6 | Backend `ad87b4d9`; frontend `c510e131` |
| Medio | El formulario no contaba la retención contra lo que se debe, ofrecía «Saldo a favor» a quien registra solo en una marca, copiaba datos viejos con «Anular y copiar» y perdía tasas pendientes | La fila juzga monto más retención; «Saldo a favor» se retiene (C8); la copia dice lo que dejó y revisa lo que copia; la cola de tasas conserva las pendientes | Frontend `c6c96b1e`, `64581af4`, `1e189bf0`, `b556fa26` |
| Bajo | Una factura de dealers con todas sus unidades saldadas se seguía ofreciendo | C5 | Backend `96c62684`; frontend `a52c6a71` |
| Bajo | Los rechazos no nombraban lo que se debía, y su marca quedaba; un pago sin número de operación se mostraba como de saldo a favor; «Ver pagos» abría la lista sin su empresa | Corregidos en la pantalla | Frontend `2b023ba4`, `be45bcfc`, `131314dc`, `6205e8d4` |
| Bajo | El objetivo de 1 s de un pago de 100 aplicaciones no se había medido (B19) | Se midió (fase 5, abajo) | Backend `b44950c8` |
| Medio | La brecha de días de cada unidad contaba desde el primer pago aplicado a su factura, no desde el pago del bloque que la incluye | R22: `UnitCoverage.baseDate`, que `PurchaseInvoiceUnitPurchasesAdapter` pasa a `fleet` en la ficha y en el costo de compra de la unidad. Lo prueban `UnitCoveragePolicyTests.eachUnitCountsItsGapFromTheFirstPaymentThatCoversIt`, el ejemplo 4 (`aPaymentWithoutUnitsAfterPayingHalfTheMotorcyclesGoesToTheOtherHalf`), que ahora comprueba la fecha de cada bloque, y `VehiclesIntegrationTests.eachUnitOfAnInvoicePaidInTwoBlocksCountsItsGapFromItsOwnPayment` | Backend `c43d1bc1`, merge `1bd767f1` |

**Pruebas más débiles que su regla** (B20; todas las fases): las de mañana en Lima dependían de la hora en que corría la suite, y `ProcurementCalendar` reemplaza el `Clock.systemUTC()` fijo (`ProcurementDatesInLimaTests`, `EnablementDatesInLimaTests`); el fallo en la línea 100 nunca escribía nada antes de fallar; y la línea de cero, la tasa 1, la fila delicada, el redondeo de la exportación, el corte de las notas, la retención contra lo que se debe y las anulaciones por la ruta se leen ahora donde la regla los pone (backend `6165ba5d`, `7984202f`, `7b92a7de`, `fbd6f291`). Las citas de pruebas de las especificaciones y de los contextos quedaron al día.

**Lo que corrió el 04/11/2026.**
- **Backend:** la suite completa, 3 570 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`: fallan 78, todas de la línea base del superusuario, las de Row Level Security y de filas que no se borran, que necesitan las dos cuentas.
- **Frontend:** 3 285 pruebas unitarias en 275 archivos; stylelint limpio; la e2e completa de escritorio, 657 pasan, 3 se saltan y ninguna falla; axe limpio en las tres anchuras en las pantallas que cambiaron. No se regeneró ninguna imagen de referencia.
- **R22** (backend `c43d1bc1`): las 117 pruebas que alcanza el cambio —la política, el adaptador, la ficha y el costo de compra de la unidad, los pagos, las de OpenAPI y `ModularityTests`— pasan. El frontend no cambió.

**Lo que queda.**
- **Las sondas en vivo** de las fases 2 a 5, `e2e-live/purchase-invoices`, `enablement`, `credit-notes` y `supplier-payments`: 45 casos (12, 10, 12 y 11) que nunca corrieron. Necesitan la contraseña de verificación del responsable.
- **El CI con las dos cuentas:** las pruebas de Row Level Security y de filas que no se borran de las tablas nuevas no han pasado en verde allí. La sonda de SQL de arriba confirmó lo que comprueban, pero no las reemplaza.
- **El inventario antes de V96**, contra la base desplegada, antes de que `develop` despliegue V96. Lo corre el responsable.
- **El rendimiento:** la línea base de una factura de 100 líneas se midió con `postgres` (mediana 1 731 ms, máximo 2 063 ms), no con la cuenta de la aplicación; y el máximo de un pago de 100 aplicaciones, 1 911 ms, supera el objetivo de 1 s, aunque la mediana, 899 ms, lo cumple.
- **La validación del contador** de D-190, D-192, D-194 y D-196, y la Consulta RUC, que dice si Millennium es agente de retención (§3.2).
- ~~**Las propuestas para el diccionario**, que decide el responsable y no se aplicaron:~~ Aprobadas por el responsable el 04/11/2026 y aplicadas ese día: documentación `7cc29f4`, diccionario v1.39.
  - del cierre de la fase 4 (Q1 a Q6): §2.7 (los archivos de las notas), §3 fila 7 y §13.1 (la anulación de un pago, delicada);
  - del cierre de la fase 5 (R8 a R16): §3 filas 8 y 9; §10.2 (la aplicación exacta, el paso de una orden a su factura, el comprobante de retención); §10.3 (el reparto por lo que le falta a cada unidad, los céntimos, el tope, la imputación que no cambia); §9.2 (la orden con pagos); §10.1 (la regla del `request_id`); y §10.5 (la brecha en la zona de la empresa);
  - de esta revisión (R17 a R22): §10.2 (el equivalente que absorbe el redondeo), §9.1 (`suppliers.is_dealer` no se marca con pagos en sus órdenes), §8.2 (ningún camino salvo el regreso desde Vendido mueve sola una unidad a un estado con puerta, y el estado inicial no se marca), §13.1 (los campos cuyos valores solo muestra un permiso), §2.7 (una sola regla de descarte, en el párrafo y en la fila de `storage_key`, que reemplaza la frase «Es seguro sin leer la tabla») y D-199 con §10.5 (la fecha base de cada unidad, R22, arriba);
  - el texto del permiso `compras.registrar_nota_credito` (Q7) no se propuso aplicar y queda como está.
- **Un límite conocido del formulario de pagos:** lo que se puede pagar no trae marcas, así que el formulario no retiene una aplicación a una factura de una marca que el permiso no alcanza; la rechaza el backend, con `403`.

### 2.11 Segunda revisión de punta a punta y correcciones (04/11/2026)

Con las correcciones de §2.10 integradas, cuatro revisores independientes volvieron a leer el plan v4 de punta a punta el 04/11/2026, y el responsable pidió tres cambios de pantalla. Las correcciones y los pedidos se integraron en `develop` el mismo día, sin migraciones:
- **backend** `851b4d09`, la rama `feature/review-2-findings`, 20 commits de `a126216f` a `607e6756` sobre `1bd767f1`;
- **frontend** `3a49f141`, la rama del mismo nombre, 50 commits de `a8ccc252` a `c64888e9` sobre `2908405f`, con las revisiones 40 y 41 de `DESIGN.md`.

Cada especificación registra lo suyo con la marca **Como se corrigió (04/11/2026, segunda revisión)** y su §10.4: [facturas](especificacion-facturas-de-compra-2026-11-02.md), versión 5, con el contrato C10 en su §11; [habilitación](especificacion-habilitacion-2026-11-02.md) y [notas](especificacion-notas-de-credito-2026-11-03.md), versión 5; y [pagos](especificacion-pagos-a-proveedores-2026-11-03.md), versión 6.

**Lo que se comprobó y estaba bien:** los contratos entre el frontend y el backend; el cálculo de la absorción del redondeo, idéntico en los dos repositorios con siete vectores hechos a mano; los candados; Row Level Security; los permisos; y que existen las 250 pruebas que citan las especificaciones.

**Los pedidos del responsable (04/11/2026):**

| Pedido | Cómo quedó | Commits |
|---|---|---|
| La barra de una lista con varios filtros | Revisión 40 de `DESIGN.md`: una lista con tres filtros o más lleva la búsqueda y las acciones en una fila, y los filtros en otra, alineados al inicio, cada uno con un valor —«Todos …» o «Todas …»; los días, con la etiqueta flotando—, con «Limpiar filtros» mientras uno está puesto y, en el teléfono, «Filtros» con la cuenta de los activos. La toman Facturas, Notas de crédito, Pagos, Inventario, Habilitación y Vencimientos, Tipos de cambio, Bitácora, Roles, Marcas y modelos, Plantillas de OC, Ubicaciones y Colaboradores; Órdenes de compra y las listas de una ficha, de dos filtros o menos, conservan su barra | Frontend `89f9f6c5`, `83a83cce`, `68f17cbc`, `71ccb43c`, `0036e50c`, `1c06b9c5`, `9b09c97b`, `977636db`, `61b08bf4`, `0d51eec3`, `48235558`, `bebbab4f`, `9ed75604`, `f5425b24` |
| Importar las líneas de una factura desde Excel, solo en las facturas de compra | El contrato C10: `GET /purchase-invoices/line-template`, la plantilla como documento generado (D-106), y `POST /purchase-invoices/line-import`, que lee el libro y responde sus líneas y los errores de sus filas sin guardar nada, con cuatro `422 PURCHASEINVOICE_IMPORT_*`; en la pantalla, «Importar desde Excel» reemplaza «Pegar columnas» ([especificación de facturas](especificacion-facturas-de-compra-2026-11-02.md) §11) | Backend `6eee6878`, `d3b1e6ae`, `23fb6bba`; frontend `1184be6f`, `6a25fabe`, `4e7d440b`, `3d5387b9`, `1a1775d7` |
| La tarjeta de datos de los formularios en grupos | Revisión 41: la factura en Comprobante, Importes, Orden de compra y Archivos; la nota en Comprobante, Motivo, Importe y Archivo; el pago en Pago, Importe y Notas; cada grupo nombrado, sobre una grilla de 12 columnas, cada campo con la etiqueta flotando y la tasa o la moneda leídas como un dato | Frontend `f6510432`, `94c1c6a3`, `c288a00d`, `2da6c23f` |

**Las decisiones**, tomadas por la coordinación con la orden del responsable, como C1 a C8 (§2.10):

| N.º | Decisión |
|---|---|
| C10 | La importación de las líneas de una factura: la plantilla es un documento generado, del tipo `purchase-invoice-line-template`, con el alcance de tener `compras.registrar_factura` y la versión de los datos de `vehiclecatalog`; la importación no guarda nada y no es una carga de archivos (D-200, D-208); los nombres se resuelven sin mayúsculas ni tildes contra los modelos que una unidad nueva puede tomar (D-108); hasta 100 filas y 2 MB. No necesita cambiar el diccionario |
| C5, completada | Lo que la empresa puede pagar de una factura de dealers es lo que se debe menos las iniciales de sus unidades saldadas (`AmountOwedPolicy.payable`): si no es mayor que cero, la factura sale de lo que se ofrece pagar y su ficha no ofrece `registerPayment` |
| — | El regreso automático desde Vendido salta la puerta solo mientras Vendido la exige; el catálogo mantiene la exención por el código. Propuestas R23 (§8.2) y R24 (D-164) |
| — | La diferencia de cambio del formulario suma lo que absorbió el equivalente, como el backend. Propuesta R25 (D-213) |
| — | Una prueba que confirma datos reusa la escena de las carreras, nunca un país ni una empresa nuevos (§7) |

**Fase 2 · Facturas de compra**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Medio | El formulario no retenía una línea que vincula una unidad de una marca que el permiso no alcanza: leía el valor del formulario, que deja fuera los campos deshabilitados de esa línea. «Marca comercial por omisión» ofrecía marcas donde la sesión no registra | Cuenta la marca de la unidad; la marca por omisión ofrece solo las de la sesión (C8) | Frontend `cbf35bf6` |
| Bajo | Una línea de concepto respondía `500` al anular la factura o cambiar su orden: B17 no alcanzó `brandsOf` | Deja fuera las marcas nulas | Backend `76fa8f0d` |
| Bajo | Las marcas de un rechazo seguían la posición de la línea, en la factura y en la nota | Se marcan por la línea misma | Frontend `10b31b28`, `4aef618a` |
| Bajo | Una revisión del VIN que respondía después de cambiar de empresa se aplicaba igual | Se descarta | Frontend `36b4d052` |
| Bajo | El selector buscado en el servidor no volvía a preguntar un texto escrito después de elegir, y lo leído para opciones que se borraron quedaba entre las siguientes (también en notas y pagos) | Corregidos en la pieza compartida y en los servicios de búsqueda | Frontend `842c98b5`, `21cf2277` |
| Bajo | El ingreso desde la ficha de la factura confundía una unidad anulada mientras tanto con una ingresada | La anulada ya no se lista, y lo dice | Frontend `cb2b7efd` |

**Fase 3 · Habilitación**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Alto | La puerta de Disponible se saltaba con el regreso de las 48 h una vez desmarcado Vendido (D-184): a una unidad vendida se le podía retirar el GPS y volvía sola a Disponible sin él, lo que rompe D-170 | El proceso exime el regreso desde Vendido solo mientras Vendido exige la habilitación; desmarcado, deja en Vendido a la unidad incompleta (`AutomaticReturnGate.appliesToTheReturn`) | Backend `a126216f` |
| Bajo | El inventario antes de V96 no decía si el estado inicial es Disponible o Vendido, ni los caminos `mover_sola` hacia ellos desde otro estado que Vendido | El inventario y su README los listan, con qué hacer antes del merge | Backend `ea5f342c` |
| Bajo | El proceso escribía el mismo `WARN` en cada corrida por una unidad retenida | Solo en la primera corrida después del vencimiento; las demás, en `DEBUG` | Backend `62708a60` |
| Bajo | Marcar un estado y guardar un camino hacia él revisaban sin candado: dos pedidos a la vez dejaban juntas las dos filas que la puerta prohíbe | El camino lee su destino `FOR SHARE`; marcar o retirar un estado lo lee `FOR NO KEY UPDATE` antes de sus caminos ([patrones de persistencia](patrones-de-persistencia-millennium-mobility.md)) | Backend `5bcd796b` |
| Bajo | Los diálogos: «Registrar costo» por el acceso de la página y un costo vacío; un `{{ status }}` literal en el aviso de un camino reabierto; el texto de la lista dentro del diálogo del camino; el ingreso en lote que obligaba a quitar a mano las unidades ya ingresadas; un destino que la puerta rechazó, todavía elegido; una ubicación desactivada mientras tanto; el mock de `PUT /vehicle-statuses` en otro orden que el backend | Cada uno en su pantalla | Frontend `7cd9f40f`, `46d36540`, `4b24c845`, `d184bceb`, `b698c392`, `e18a6754`, `5d6d34c5`, `1d278ad7` |
| Bajo | Los filtros Modelo y Documento de Habilitación leen catálogos que piden `catalogos.ver`, y quedaban vacíos sin decirlo | Solo con `catalogos.ver`, como Estado | Frontend `d2ae9682` |
| Bajo | Comentarios y descripciones de la API viejos: ocho permisos de accesos y roles en lugar de once; el costo de una instalación por la empresa sola; el descarte de `VehicleDocumentFiles` por `open` | Corregidos | Backend `6e71a19d`; frontend `a8ccc252` |

**Fase 4 · Notas de crédito**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Medio | Con montos fijos, `CREDITNOTE_EXCEEDS_VALUE` dejaba el formulario sin salida | Toma lo que le queda a la línea, lo dice y deja volver a registrar; con montos escritos, actualiza el valor actual | Frontend `01b667c2`, `4aef618a` |
| Bajo | `CREDITNOTE_INVOICE_ANNULMENT_PARTIAL` no nombraba las facturas que quedarían con valor | Las nombra | Frontend `01b667c2`, `4aef618a` |
| Bajo | Decir que no a «Cambiar empresa» perdía el proveedor | La empresa y su proveedor se quedan | Frontend `b5511c54` |
| Bajo | El aviso de las unidades devueltas mandaba anular una unidad que otra factura ya volvió a comprar | Dice qué hacer mientras siga sin factura vigente | Frontend `a3f532b1` |

**Fase 5 · Pagos a proveedores**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Medio | La diferencia de cambio del formulario no sumaba lo que absorbió el equivalente: con V1 mostraba PEN 47,87, y el backend guardaba 47,89 | Lo suma, como `ExchangeDifferencePolicy` | Frontend `fd6d0ca8` |
| Bajo | Una factura de dealers que solo debe las iniciales de sus unidades saldadas, sin estarlo todas, seguía entre lo que se puede pagar, y su ficha ofrecía «Registrar pago»: C5 estaba incompleto | C5, completada (arriba) | Backend `5c6f5134`, `74397269`; frontend `cc9379ab` |
| Bajo | «Anular y copiar» de un pago con dos aplicaciones a un mismo documento perdía el monto de una; `?factura=` de una factura que no se debe abría sin fila y sin decirlo | La copia las junta; el formulario lo dice | Frontend `4ea36a0d` |

**En todas las fases**

| Severidad | Hallazgo | Cómo quedó | Commits |
|---|---|---|---|
| Bajo | En Spaces, `KeptFileStore.put` escribía el objeto antes que su fila: una fila que fallaba dejaba un objeto sin fila y sin descarte | Primero la fila, después el objeto ([patrones de persistencia](patrones-de-persistencia-millennium-mobility.md)) | Backend `5df515b8` |
| Bajo | Pruebas más débiles que su regla: PRC-R101 con un identificador al azar; PRC-R102 sin la marca; PRC-R98 que fallaba antes de escribir; el intervalo del equivalente con seis vectores; FLT-R95 y `readAllLocked` sin transacciones reales; FLT-R131 y FLT-R63 sin su caso; el `put` repetido dentro de la misma transacción | Pruebas nuevas, sin cambiar el código: `SupplierPaymentsIntegrationTests` (dos casos), `CreditNoteRollbackTests`, la prueba de propiedades y los vectores V7 a V10 de `PaymentEquivalentsTests` (y V9 en el frontend), `FleetContextFacadeLockTests`, `VehiclesIntegrationTests.theRecordOfADeliveredUnitCountsItsGapUpToItsFirstMoveIntoActivo`, `EnablementDatesInLimaTests.aCorrectedArrivalOfTomorrowInLimaIsRefused`, `KeptFileStoreUpsertTests.aKeyAnEarlierTransactionCommittedIsNotWrittenFirstAgain` | Backend `e1c0400f`, `fb84d6c8`, `d19b16ba`, `eed0c204`, `22b16143`, `5f967ae3`; frontend `be081bc8` |
| Bajo | Las pruebas confirmadas gastaban datos: `PurchaseInvoiceRollbackTests`, `CreditNoteRollbackTests` y `VehicleLockTests.setUp` confirmaban una empresa en un país de prueba nuevo en cada corrida. Al 04/11/2026 había 26 en `erp_mm_db_verify`, todos los de `XA` a `XZ`, y `AccessFixtures.countryWithoutCompany` fallaba sin código libre | Las dos de reversión registran sobre la escena de las carreras (`InvoiceRaces.root`); `VehicleLockTests` arma su empresa solo dentro de la transacción de la prueba; `countryWithoutCompany` busca entre todos los códigos que ISO 3166 deja a sus usuarios (AA, QM a QZ, XA a XZ y ZZ). La regla queda en §7 | Backend `dd225f1a`, `607e6756` |
| — | La e2e del armazón, `e2e/shell.spec.ts`, fallaba a veces: medía la barra lateral y la superior a mitad de su transición. Ya fallaba así en `develop` | Las mide cuando las dos se asientan | Frontend `c64888e9` |

**Lo que corrió el 04/11/2026** (la verificación de la coordinación):
- **Backend:** la suite completa dos veces seguidas, 3 619 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`: fallan 78, todas de la línea base del superusuario, y ninguna corrida gastó un país de prueba.
- **Frontend:** 3 327 pruebas unitarias en 275 archivos; stylelint limpio; la compilación, bien. La e2e completa de escritorio: 659 pasan, 3 se saltan y una falla de forma intermitente, la del armazón; con `c64888e9`, 116 de 116 en cuatro repeticiones, en escritorio y en tableta. Las imágenes de referencia de las 16 pantallas que cambiaron se tomaron otra vez en las tres anchuras, 48 en total, y cada una se miró (`abb8a556`).
- **Las capturas** de las barras nuevas y de las tarjetas en grupos, a 1 440 y a 375 px, se revisaron a ojo.

**Lo que queda.**
- **Las sondas en vivo** de las fases 2 a 5: 45 casos que nunca corrieron (§2.10). Necesitan la contraseña de verificación del responsable. Ninguna cubre la importación desde Excel.
- **El CI con las dos cuentas** (§2.10).
- **El despliegue de lo integrado desde el 03/11/2026** (hecho el 04/11/2026, §2.12), las migraciones V92 a V107 (la base desplegada llegó a V91 con backend `b1264ed5`, el 02/11/2026), con V96: antes, el responsable corre el inventario contra la base desplegada, que desde `ea5f342c` lista también el estado inicial y los caminos `mover_sola` que V96 no detiene (§3.2).
- **La validación del contador** de D-190, D-192, D-194 y D-196 (§3.2).
- **Las propuestas para el diccionario** de esta revisión, redactadas y sin aplicar, que decide el responsable:
  - R23 (§8.2) y R24 (D-164): el regreso desde Vendido salta la puerta solo mientras Vendido la exige; desmarcado, deja en Vendido a una unidad incompleta (D-170, D-184);
  - R25 (D-213): la diferencia de cambio suma los céntimos que el equivalente tomó del redondeo (§10.2), para que diga lo que de verdad salió.
  - La importación desde Excel no necesita cambiar el diccionario: no guarda nada, no es una carga de archivos y sigue D-106 como documento generado.

### 2.12 La barra de las listas y las tarjetas iguales (04/11/2026)

**Por qué.** Después de la segunda revisión, el responsable pidió dos cambios de pantalla: en todas las listas, los filtros arriba y las fechas abajo, seguidas de importar, exportar o crear, aprovechando el espacio; y uniformidad en las tarjetas de datos, con placeholders pertinentes y un «Descargar plantilla» más visible.

**Commits:** frontend `863817c3`, el merge de `feature/toolbar-rows-2` (de `58d55bd8` a `d90e045e`). Sin migraciones y sin cambios en el backend.

- **La barra (revisión 42 de `DESIGN.md`; `58d55bd8`, `ba0959dd`, `3da41678`):** en las listas de varios filtros, la búsqueda y los selects llenan el ancho arriba; abajo van las fechas y «Limpiar filtros» al inicio, y las acciones al final. Una lista sin fechas conserva esa fila, con las acciones al final. En el teléfono, «Filtros» pliega los selects y las fechas, y las acciones comparten el ancho.
- **Las tarjetas (revisión 43; `5678d636`, `ce9d1aee`, `70199bfb`, `01f667ab`, `075576ee`, `f93038a2`):** una sola zona de archivo, con sus tipos y su límite reales, en las tres tarjetas, en «Importar desde Excel», en «Cambiar archivo u orden» y en «Cargar documento»; ya no queda ningún «Elegir archivo». Una sola voz, de usted. Cada campo vacío dice algo: un ejemplo real en los de texto, una indicación en los selects. Campos de cuatro columnas de doce, y los datos que se leen, con la altura de un campo. «Descargar plantilla» es un botón delineado en el color primario.
- **Un límite que estaba mal:** el diálogo de importación decía 5 MB y el backend rechaza más de 2 MB (`PURCHASEINVOICE_IMPORT_FILE_SIZE`, C10). Ahora dice 2 MB y rechaza antes de enviar (`075576ee`).
- **La enmienda:** la revisión 43 enmienda la revisión 18, que no pedía textos de ayuda, por decisión del responsable del 04/11/2026.

**Un hallazgo de esta verificación, y una falla de la anterior.** El CI corre la e2e en los tres anchos, y la verificación de §2.11 solo corrió escritorio. Por eso no se vio que 23 pruebas de teléfono fallaban en `develop` desde `3a49f141`: buscaban filtros que la revisión 40 pliega detrás de «Filtros». Las pantallas estaban bien; las pruebas, no. Las corrige un solo ayudante, `unfoldFilters` (`8353b38b`, `d90e045e`), que decide por el proyecto y no por si el botón se ve, porque la barra puede no haberse dibujado. Un cambio de pantalla se verifica en los tres anchos, no solo en escritorio.

**Lo que ya no se usa, retirado:** `isCreditNoteReason`, que solo nombraba su prueba; el tipo `LineImportCode`; dos importaciones de `SearchSelectOption` (`c8958263`); `formatFileSize` y los 12 textos que dejaron los dos diálogos y «Todas» en Habilitación (`f93038a2`); y los comentarios que seguían hablando de líneas pegadas (`2f611910`). Se buscó también en el backend: ninguna clase quedó sin uso, y el resto de lo señalado se usa por inyección de Spring o por claves de texto construidas al vuelo.

**Lo que corrió el 04/11/2026**, sobre `d8197f3a`, con `develop` en `8d41eb99` ya integrado:
- 3 336 pruebas unitarias en 275 archivos; stylelint limpio; las compilaciones de desarrollo y de producción, bien.
- La e2e completa en los tres anchos, 2 055 pruebas: las 685 de escritorio, sin fallas; en tableta y teléfono, 1 143 pasan, 221 se saltan y 6 de teléfono fallaron por la misma causa de arriba. Esas 6 se corrigieron en `d90e045e`, y `access.spec` y `company-profile.spec` pasan completas en los tres anchos, 231 pruebas.
- Una muestra de las listas, las tres tarjetas y los tres diálogos, a 1 440, 1 024 y 375 px, se revisó a ojo.

**El despliegue de V92 a V107.** El backend `851b4d09` terminó de desplegarse bien el 04/11/2026 a las 15:02 (captura del responsable). En `prod`, Flyway corre al arrancar y Hibernate valida el esquema, así que las migraciones quedaron aplicadas y V96 no se detuvo. Se deduce del despliegue; nadie leyó la base desplegada.

**Lo que queda.**
- **El job e2e del CI:** con este merge debería pasar; `8d41eb99` debió fallar por las 23 pruebas. Desde aquí no se puede leer el CI.
- **Las sondas en vivo, el CI con las dos cuentas, la validación del contador y R23 a R25** (§2.11).
- **La decisión sobre D-184:** si se conserva la opción de desmarcar Vendido, que también permite entregar unidades incompletas, contra la columna 22 de la guía.

### 2.13 Un solo flujo, las fechas a la derecha y solo los filtros necesarios (04/11/2026)

**Por qué.** El responsable abrió las listas y no vio el espacio bien aprovechado. Lo que veía era la barra de la revisión 40, en una pestaña anterior al despliegue de `863817c3`, pero al reproducirlo a su ancho apareció un fallo de la revisión 42: solo se había medido a 1 440, 1 024 y 375 px. A 1 280 px un filtro ocupaba los 966 px de una línea, a 1 536 px ocupaba 1 222 px, y las listas sin fechas gastaban una fila en sus botones. Después de ver cada lista antes y después, decidió las tres cosas de abajo.

**Commits:** frontend `4593514f`, el merge de `feature/toolbar-fit` (de `6721267e` a `b1b60754`), con la revisión 44 de `DESIGN.md`. Sin migraciones y sin cambios en el backend, que sigue aceptando los filtros retirados.

- **Un solo flujo (`6721267e`):** la búsqueda, los filtros y, al final, un bloque indivisible con las fechas y los botones, que se queda en la línea si cabe entero y baja a la siguiente si no. Un filtro crece hasta 25rem.
- **Las fechas a la derecha:** Desde y Hasta van pegados a los botones, al final de la fila.
- **Filtros retirados (`916e20e4`):** Tipo en Facturas; Moneda en Facturas, Notas y Pagos; Estado en Habilitación; Moneda de destino en Tipos de cambio; Alcance y Tipo de rol en Roles. Marca comercial, en Inventario, aparece al elegir una empresa. El diccionario no exigía ninguno. La consulta de facturas conserva `currency`, porque el formulario de la nota pide las facturas del proveedor en su moneda.
- **Las búsquedas** dicen por qué campos buscan, nunca un ejemplo: «Buscar por placa, VIN o motor» en Inventario y Habilitación, y «Buscar por permiso, módulo o categoría» en la matriz de permisos (`916e20e4`, `5424a030`).
- **Las fechas de la Bitácora (`eb4f1dfc`)** usan el calendario de las demás listas, en la zona en que se leen.
- **Todos los campos de calendario (`5424a030`)** muestran «dd/mm/aaaa», y Mes de campaña «mm/aaaa». Cuatro mostraban un día de ejemplo y los otros 22, nada.
- **Debajo de las pestañas (`f4a8716d`):** en Habilitación y Roles y permisos, el contenido empieza a 16 px de la barra de pestañas, no a 40.

| Medido en 12 listas a 8 anchos, de 1 024 a 2 560 px | Antes | Después |
|---|---|---|
| Barras de una fila | 0 | 47 de 96 |
| Barras de dos filas | 71 | 49 |
| Barras de tres filas | 25 | 0 |
| Filtro más ancho | 1 366 px | 400 px |

**Lo que corrió el 04/11/2026**, sobre `b1b60754`: 3 336 pruebas unitarias en 275 archivos; stylelint limpio; las compilaciones de desarrollo y de producción, bien; la e2e completa en los tres anchos, 1 843 pasan, 248 se saltan y ninguna falla. `e2e/toolbar-rows.spec.ts` mide además 1 280, 1 366, 1 536, 1 680, 1 920 y 2 560 px: una barra con un filtro estirado ya no pasa.

### 2.14 La regla de Desde y Hasta y el período de las órdenes (04/11/2026)

**Por qué.** El responsable pidió dos cosas: que Órdenes de compra, la única lista de compras sin fechas, filtre por fecha o por intervalo, y que un Desde sin Hasta muestre solo ese día, no todo lo posterior.

**Commits:** frontend `bcfb4c93`, backend `7c8cd891`. Sin migraciones.

- **La regla:** Desde solo es ese día; Desde y Hasta, el intervalo completo, con los dos días; Hasta solo, todo hasta ese día. Vive en un solo lugar, `dayPeriod` de `calendar-day.ts`, y la usan Facturas, Notas de crédito, Pagos a proveedores, Tipos de cambio, Bitácora de cambios y Órdenes de compra. La exportación usa los mismos filtros, así que dice lo mismo que la pantalla. La API no cambia: sigue leyendo `[from, to)`, y es la pantalla la que manda `to` como el día siguiente. El caso de Hasta solo no lo pidió el responsable: conserva lo que ya hacía.
- **Las órdenes:** `GET /purchase-orders` acepta `from` y `to` sobre la fecha de emisión, y la lista toma la barra de las demás (DESIGN.md, revisión 45).

**Lo que corrió el 04/11/2026:** 3 341 pruebas unitarias en 275 archivos, y las compilaciones y stylelint, bien; la e2e de lo que el cambio alcanza en los tres anchos, 259 que pasan; el proyecto de teléfono completo, 576 que pasan y 124 que se saltan; y accesibilidad y fechas en escritorio y tableta, 325 que pasan y una inestable por carga, que pasa 16 de 16 repetida sola y con cinco workers. En el backend, 312 pruebas de compras y de módulos, con los 16 fallos de la línea base del superusuario.

---

## 3. Decisiones

### 3.1 Tomadas por el responsable el 02/11/2026

El texto completo de cada una está en el diccionario v1.38. La v1.39 (04/11/2026, documentación `7cc29f4`) corrige D-199: la brecha de cada unidad cuenta desde el primer pago que la cubre (R22, §2.10).

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
| Responsable | **Cerrado el 04/11/2026:** el despliegue de `851b4d09` pasó V96 (§2.12). Correr el inventario antes de V96 contra la base desplegada (§2.10). Desde el 04/11/2026 (`ea5f342c`, §2.11) lista también si el estado inicial es Disponible o Vendido y los caminos `mover_sola` hacia ellos desde otro estado que Vendido, que se corrigen antes del merge | El despliegue de V96 en `develop` |
| Responsable | Validar las propuestas para el diccionario de la segunda revisión (§2.11): R23 (§8.2) y R24 (D-164), el regreso desde Vendido con la puerta según su marca; R25 (D-213), la diferencia de cambio con lo que absorbió el equivalente | Nada: el código ya las sigue desde el 04/11/2026 (backend `a126216f`; frontend `fd6d0ca8`) |
| Responsable | La contraseña de verificación, para las sondas en vivo de las fases 2 a 5 | El cierre de los pasos 5.12, 6.13, 7.12 y 8.9 |
| ~~Responsable~~ | ~~Validar las propuestas para el diccionario de las fases 4 y 5 y de la revisión de punta a punta (§2.10)~~ | Hecho el 04/11/2026: el responsable las aprobó y la documentación `7cc29f4` las aplica en el diccionario v1.39, con R22, la fecha base de cada unidad (§2.10) |

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

**Pospuesto el 03/11/2026 por el responsable:** los pasos 4, 6, 8 y 9, es decir, las tres entregas de cargas y la corrida de la carga inicial («Aún no haremos cargas iniciales, solo implementación»). El orden sigue con la fase 4 y la fase 5, construidas las dos el 03/11/2026. Las cargas esperan una nueva orden del responsable y sus archivos reales, que se perfilan antes de construir (§1.3 de la [especificación de cargas](especificacion-cargas-de-archivos-2026-11-03.md)). Mientras tanto, las unidades que ya existen se registran a mano desde sus facturas, y su habilitación, en su página. El diccionario no cambia: D-208 fija que la carga inicial corre una sola vez y después de la fase 5, no cuándo.

**Por qué la habilitación va antes de las notas de crédito y los pagos:**
- cierra el flujo operativo de la compra a Disponible;
- no espera respuestas de afuera;
- las unidades que nacen con la factura necesitan pronto su ingreso físico.

**Las facturas no se usan con datos reales hasta tener las notas de crédito.** El mecanismo es D-176: hasta la fase 4, solo el superadministrador tenía los permisos de facturas. Desde V104, que se integró con la fase 4 el 03/11/2026, los tienen los roles de la matriz, y las facturas se usan con datos reales.

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
- **Falta** (paso 5.12): las sondas en vivo, que existen desde el frontend `10696eed` y `d126d8a2` (`e2e-live/purchase-invoices.live.spec.ts`, el commit F17) y no corrieron, porque necesitan la contraseña de verificación del responsable (especificación §7.4). El texto de §2.7 del diccionario sobre el descarte de un archivo quedó escrito el 03/11/2026 con su aprobación (documentación `caaada3`); la revisión del 04/11/2026 encontró falsa, para un `PUT`, su frase «Es seguro sin leer la tabla», y el código ya registra el descarte solo de una llave que la transacción escribió primero (§2.10).

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

**Hecho el 03/11/2026** (backend `8bfdb938`, con V95 a V101; frontend `2659126a`), con la aceptación cumplida salvo las sondas en vivo:
- **Pasos 8.0 a 8.8:** el contrato en las convenciones (documentación `462388f`); los permisos (V95) y el esquema (V96 a V101), con la marca de Disponible y de Vendido y el camino «Pierde un requisito»; la fachada de `vehiclecatalog`; el ingreso, los hitos, la placa, los documentos, los equipos y las llaves; `EnablementPolicy`, la puerta que se sostiene, `allowedActions` y la lista con su proyección; y el frontend: Catálogos › Tipos de documento, la marca de Estados y motivos, Logística › Habilitación con los vencimientos y el ingreso en lote, la página de la habilitación de una unidad, el cambio de estado con los caminos retenidos y el ingreso desde la ficha de la factura. Las reglas FLT-R55 a FLT-R93 y VCT-R28 a VCT-R31 están en ✅, con el nombre real de cada prueba y sus commits en la [especificación de habilitación](especificacion-habilitacion-2026-11-02.md) §5 y en los contextos; el commit de cada paso, en su §8.
- **Aceptación:** el orden de las seis filas lo fija `EnablementPolicyTests`; la entrada manual a Disponible se rechaza por los dos caminos si falta la TIVE, la pantalla, el GPS o las placas físicas, y una unidad entregada y reacondicionada vuelve (`EnablementGateIntegrationTests`); el regreso automático no se bloquea; desmarcar Disponible da 422; el GPS de una unidad Disponible no se retira sin reemplazo, y la unidad sale por «Pierde un requisito»; la carrera del GPS no deja una unidad Disponible sin GPS en ninguno de los dos órdenes (`EnablementConcurrencyTests`); nada se instala antes del ingreso; 80 unidades ingresan en una operación, en menos de 2 s, y las sentencias que no son inserciones ni actualizaciones son las mismas con 1, 10 y 80 (corregido el 04/11/2026: este texto decía «las mismas sentencias», pero cada actualización y cada fila de la bitácora es su propia sentencia, porque el backend no configura lotes de JDBC; §2.10); el filtro de vencimientos cuenta los 30 días en la zona de la empresa; y el certificado de gas solo se pide a modelos a GNV o GLP.
- **Pruebas:** la suite completa del backend, 3 254 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`, con solo seis fallos nuevos frente a la línea base de 67 de `16f69af3`, todos porque el superusuario se salta Row Level Security y los permisos; las unitarias del frontend, 3 007 de 3 007; las e2e en escritorio de habilitación, flota, facturas de compra y tipos de documento, 63 de 63; las de accesibilidad de la habilitación en los tres anchos, 18 de 18; y todas las capturas en los tres anchos.
- **La revisión del contrato** del frontend contra el backend, antes de integrar: el backend respondía lo especificado, y las diferencias del frontend —una rotura, cuatro textos, diez respuestas del simulador y los campos que pueden llegar en `null`— quedaron corregidas.
- **Lo que el código hizo distinto** queda en la versión 3 de la especificación, con la marca «Como se construyó»: entre otras cosas, los cuatro textos de `VEHICLE_ENABLEMENT_HELD_BY_STATUS`, que reemplazan al único que proponía; `location::acl` desde el paso de los tipos de documento; y la fachada del ingreso de una factura, que recibe la empresa.
- **Falta** (paso 8.9): las sondas en vivo, que existen desde el frontend `4ca16d02` y necesitan la contraseña de verificación del responsable (especificación §7.6). ~~Y el texto de §2.7 del diccionario sobre la llave, la empresa y el descarte de los archivos de un documento, que es una propuesta para el responsable.~~ Aprobado por el responsable y aplicado el 04/11/2026 (documentación `7cc29f4`, diccionario v1.39).

### Cargas, primera entrega · Registro, impuesto vehicular y garantía mobiliaria

**Pospuesta el 03/11/2026 por decisión del responsable (§4).**

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

**Hecho el 03/11/2026** (backend `e213c01c`, con V102 a V104; frontend `d579211a`), con la aceptación cumplida salvo la sonda en vivo. La puerta no esperó al contador: valida D-190, D-192 y D-194 en paralelo (§3.2).
- **Pasos 6.0 a 6.12:** el contrato en las convenciones (documentación `ec972c8`, con `be4a004` y `0e3dc40`); el permiso (V102) y las tablas, con el disparador que deja volver una línea a vigente (V103); el agregado con sus motivos y el corte configurable; la fachada de `fleet`, que anula la unidad devuelta o la deja «Devuelta al proveedor»; el registro, los dependientes de la factura y `valueIsZero`; la anulación, con la guarda del saldo que implementa la fase 5, y el cambio del archivo; la lista, la ficha, la exportación y el panel de notas de la ficha de la factura; el costo de compra de la unidad; el frontend, con el formulario, la lista, la ficha y los paneles de la factura y de la unidad (DESIGN, revisión 36); y los permisos de facturas en AE, OP y CO (V104), que se integraron con la fase por orden del responsable del 03/11/2026: desde entonces las facturas se usan con datos reales. Las reglas PRC-R68 a PRC-R99 y FLT-R94 a FLT-R99 están en ✅, con el nombre real de cada prueba en la [especificación de notas](especificacion-notas-de-credito-2026-11-03.md) §5 y en los contextos; el commit de cada paso, en su §8.
- **Aceptación:** una nota reduce el valor de cada unidad que nombra y nunca la baja de cero (`CreditNotesIntegrationTests`, `CreditNoteLockTests.twoNotesOnTheSameLineDoNotTakeItBelowZero`); una nota de dos facturas guarda la tasa de cada una (`eachLineCopiesTheRateOfItsInvoiceAcrossTwoInvoices`); desde el 01/01/2027 una nota de dos facturas da 422 (`SingleInvoiceCutoffTests`, `CreditNoteCutoffIntegrationTests`); «anulación de unidad» anula la unidad o la deja «Devuelta al proveedor», y «anulación de la factura» la deja sin factura vigente para que la vincule la que se re-emite; anular una factura con notas da 422 (`PurchaseInvoiceAnnulmentsIntegrationTests.anInvoiceWithACurrentNoteAnswersHasDependents`); anular una nota devuelve sus líneas, salvo con una unidad anulada (`CreditNotesIntegrationTests.aNoteWhoseUnitWasAnnulledIsNotAnnulled`); y el costo de la unidad se ve solo con `stock.ver_costos` (`VehiclePurchaseCostIntegrationTests`).
- **Pruebas:** la suite completa del backend, 3 388 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`, con solo dos fallos nuevos frente a la línea base de 73 de `8bfdb938`, las dos tablas nuevas en `ProcurementRowsAreNeverDeletedTests`, porque el superusuario puede borrar; la base de verificación se volvió a crear ese día, con la autorización del responsable, después de que una prueba de la fase 2 dejara 25 países de prueba. Las unitarias del frontend, 3 131; las e2e de notas de crédito, facturas de compra, roles, matriz de permisos y flota, 203 en los tres anchos, sin fallos; las de accesibilidad, 441 de 441. La suite completa en escritorio tiene un fallo, `screen-dates.spec`, «the year of today is the one of Lima on the night it already changed in Tokyo», que falla igual en `2659126a`: es anterior a la fase y se investiga con la fase 5.
- **Sentencias:** un registro de una nota prepara 41 sentencias además de sus inserciones y actualizaciones, con 1, 10 y 100 líneas; el de una factura pasa de 57 a 60, con la lectura de sus dependientes y las dos lecturas agrupadas de lo que le restan sus notas.
- **La revisión del contrato** del frontend contra el backend, antes de integrar: ninguna rotura; doce respuestas del simulador no tenían la forma o el orden del backend, y quedaron corregidas.
- **Lo que el código hizo distinto** queda en la versión 3 de la especificación, con la marca «Como se construyó»: entre otras cosas, el `403` de la ruta antes que las revisiones del servicio, que las convenciones fijan desde ese día para toda ruta (§7.2); el `400` de un `PUT` sin archivo, que responde el servicio; la lista y la ficha como un solo recurso; y los records del costo anidados en `UnitPurchases`.
- **Falta** (paso 6.13): la sonda en vivo, que existe desde el frontend `b681e083` (`e2e-live/credit-notes.live.spec.ts`, el commit F13) y no corrió, porque necesita la contraseña de verificación del responsable (especificación §7.4); correr `scripts/database/hand-over-database.sql` donde se entregue la base, por sus cuatro funciones nuevas; y la línea base de una factura de 100 líneas con la cuenta de la aplicación: el máximo, 2 063 ms, se midió con `postgres`. La base de desarrollo, `erp_mm_db`, estaba en V85, sin unidades con placa: el próximo `spring-boot:run` aplica V86 a V104.

### Cargas, segunda entrega · La carga inicial

**Pospuesta el 03/11/2026 por decisión del responsable (§4).**

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

**Hecho el 03/11/2026** (backend `601cc177`, con V105 a V107; frontend `e0ed0f68`), con la aceptación cumplida salvo la sonda en vivo. La puerta no esperó al contador: valida D-190, D-192, D-194 y D-196 en paralelo (§3.2).
- **Pasos 7.0 a 7.11:** el contrato en las convenciones (documentación `f711675`); los permisos (V105) y el canal de dealers (V106); las tablas, con sus disparadores y Row Level Security (V107); las políticas puras y el agregado; el registro, una sola vez por envío, con las tasas de cada aplicación, las imputaciones, el canal de dealers y el pago con saldo, y los puertos de dependientes y de la guarda de la fase 4; la anulación; el paso de la orden a la factura y la orden con pagos; la lista, la ficha, la exportación, el saldo, lo que se puede pagar y la posición de la factura; el estado de pago y la brecha en la ficha de la unidad; y el frontend, con el formulario, la lista, la ficha y los paneles de la factura, el proveedor y la unidad (DESIGN, revisión 37). Las reglas PRC-R100 a PRC-R139 y FLT-R130 a FLT-R132 están en ✅, con el nombre real de cada prueba en la [especificación de pagos](especificacion-pagos-a-proveedores-2026-11-03.md) §5 y en los contextos; el commit de cada paso, en su §8.
- **Aceptación:** un bono antes del pago deja lo que se debe en el neto y no crea saldo (`SupplierPaymentsIntegrationTests.theExampleOfTheNoteBeforeThePayment`), y uno después crea saldo, que paga la factura siguiente sin mover dinero (`theExampleOfTheNoteAfterThePayment`); una factura en dólares pagada en soles guarda la tasa de su aplicación (`theExampleOfADollarInvoicePaidInSoles`), y su costo no cambia, porque el costo lee solo la línea y sus notas; un pago que no se aplica completo da 422 (`aPaymentWithARestToApplyAnswersAppliedMismatch`), y el doble envío se guarda una vez (`SupplierPaymentLockTests.twoIdenticalRequestsAtOnceLeaveOnePayment`); un dealer sin detalle por unidad da 422 y una unidad saldada no recibe más pagos (`aDealerInvoicePaidWithoutUnitsAnswersAllocationRequired`, `aSettledUnitAnswersUnitSettled`); anular un pago que dejaría negativo el saldo da 422 (`annullingThePaymentWhoseOverpaymentWasSpentAnswersBalanceNegative`); y la unidad muestra sus cinco montos, el dinero entregado y la brecha (`VehiclePurchaseCostIntegrationTests.thePurchaseCostOfAUnitCarriesItsPayment`, `VehiclesIntegrationTests.theRecordOfAUnitCarriesItsPaymentStatusAndGapWithoutAmounts`).
- **Pruebas:** la suite completa del backend, 3 517 pruebas sobre `erp_mm_db_verify` con la cuenta `postgres`, con solo tres fallos nuevos frente a la línea base de 75 de `e213c01c`, las tres tablas nuevas en `ProcurementRowsAreNeverDeletedTests`, porque el superusuario puede borrar; la línea base pasa a 78. Las unitarias del frontend, 3 201 en 273 archivos, y stylelint, limpio; la suite e2e completa en escritorio, 638 pasan, 3 omitidas y ninguna falla, también `screen-dates.spec` en Tokio, que la fase 4 dejó por investigar y no se reprodujo; y la pasada de axe y las imágenes en los tres anchos de las pantallas que cambiaron.
- **Sentencias:** un registro de un pago prepara 38 sentencias además de sus inserciones, con 1, 10 y 100 aplicaciones, y 44 con una imputación por aplicación; el de una factura pasa de 60 a 64, y el de una nota, de 41 a 42.
- **La revisión del contrato** del frontend contra el backend, antes de integrar: una rotura del frontend, «Le falta» de la imputación, y ocho diferencias menores, corregidas; el backend sumó `lacking` a la posición de una factura. Las diez decisiones del responsable del 03/11/2026 sobre el contrato están en las convenciones.
- **Un defecto de la fase 4 corregido:** el costo de compra de una unidad de una factura en la moneda de la empresa respondía `400` (backend `56f992c8`; [especificación de notas](especificacion-notas-de-credito-2026-11-03.md) §8).
- **Lo que el código hizo distinto** queda en la versión 3 de la especificación, con la marca «Como se construyó»: entre otras cosas, las filas de la lista como la ficha sin sus aplicaciones, el `requestId` después de los `400` del servicio, el disparador que no deja cambiar una imputación, y `lacking`.
- **Falta** (paso 7.12): la sonda en vivo, que existe desde el frontend `03abb43d`, rehecha en `5999f719` (`e2e-live/supplier-payments.live.spec.ts`, el commit F14, con once pruebas), y no corrió, porque necesita la contraseña de verificación del responsable (especificación §7.4); correr `scripts/database/hand-over-database.sql` donde se entregue la base, por sus cinco funciones nuevas y el `UPDATE` que quita a las imputaciones; y ~~medir el objetivo de un pago de 100 aplicaciones en menos de 1 s, que no se midió~~, medido el 04/11/2026 (abajo). La base de desarrollo, `erp_mm_db`, estaba en V85: el próximo `spring-boot:run` aplica V86 a V107.
- **Rendimiento** (medido el 04/11/2026, B19 de la revisión de punta a punta, §2.10; `SupplierPaymentRegistrationBaselineTests`, backend `b44950c8`, que corre con `-Dperformance=true`). Sobre `erp_mm_db_verify` con la cuenta `postgres`, cinco corridas por caso, mediana y máximo del registro de un pago:

  | Aplicaciones | Sin imputaciones | Con una imputación por aplicación |
  |---|---|---|
  | 1 | 303 ms y 441 ms | 263 ms y 399 ms |
  | 10 | 500 ms y 586 ms | 244 ms y 390 ms |
  | 100 | 899 ms y 1 911 ms | 820 ms y 1 373 ms |

  La mediana de 100 aplicaciones cumple el objetivo de 1 s; los máximos no, con el equipo cargado. Falta medirlo con la cuenta de la aplicación, que pasa por Row Level Security.

### Cargas, tercera entrega y corrida en producción

**Pospuesta el 03/11/2026 por decisión del responsable (§4).**

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
| El contador objeta un tratamiento ya decidido (D-190, D-192, D-194) | Baja | Medio | Son prácticas estándar; se validan en paralelo, sin detener nada (§3.2). La fase 4 se integró el 03/11/2026 sin esperar su respuesta |
| Un archivo grande pasa el límite de tiempo de Cloudflare | Media | Medio | Un tope de filas medido en la línea base; un archivo más grande se parte (D-208) |
| SUNAT vuelve a mover la fecha de una nota por comprobante | Media | Bajo | El corte en una política con la fecha configurable |
| Las imágenes de referencia e2e cambian mucho | Alta | Bajo | Mover primero las pruebas «En construcción» |
| V96 se detiene en una base desplegada con unidades con placa | Media | Alto | El inventario antes de V96, que corre el responsable antes de desplegarla (§2.10) |

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
  - las e2e en el puerto 4300, con el equipo enchufado;
  - **una prueba que confirma datos reusa la escena de las carreras, nunca un país ni una empresa nuevos** (desde el 04/11/2026, §2.11). Una prueba sin la transacción de la prueba —una carrera con dos transacciones reales, una reversión que tiene que escribir antes de fallar— confirma lo que escribe, y lo que confirma queda en `erp_mm_db_verify` para siempre, porque la cuenta de la aplicación no borra. Por eso registra sobre la escena que las carreras ya guardan y reusan (`InvoiceRaces.scene`, `InvoiceRaces.root`, la empresa de Perú de las pruebas de candados), y comprueba solo lo suyo —su número de factura, sus VIN, su número de nota, su llave—, nuevo en cada corrida. Lo que arma para una prueba con transacción se arma dentro de ella, que lo revierte (`VehicleLockTests.setUp`). `AccessFixtures.countryWithoutCompany`, para las pruebas que sí necesitan un país propio, busca entre todos los códigos que ISO 3166 deja a sus usuarios (AA, QM a QZ, XA a XZ y ZZ), y sin ninguno libre falla diciendo que alguna prueba confirma su país. Dos corridas seguidas de la suite completa no gastaron ninguno (backend `dd225f1a`, `607e6756`).
- **Despliegue:**
  - las migraciones solo agregan;
  - se respalda la base antes de desplegar cada fase;
  - un error se corrige hacia adelante, nunca editando una migración aplicada;
  - las facturas quedaron solo para el superadministrador hasta la fase 4 (D-176); desde V104, integrada con ella el 03/11/2026, siguen la matriz.
- **Documentación:**
  - cada fase cierra actualizando su contexto, sus procesos, su especificación y este plan, citando commits y fechas;
  - las reglas pasan de ❌ a ✅ con su prueba.
- **Siguiente paso** (04/11/2026, después de §2.12): lo que queda de §2.10 a §2.14, que es del responsable: las sondas en vivo, el CI con las dos cuentas, la validación del contador y las propuestas R23 a R25 para el diccionario. Las propuestas anteriores para el diccionario, que también estaban aquí, las aprobó el responsable el 04/11/2026, y la documentación `7cc29f4` las aplica (diccionario v1.39). Hasta el 03/11/2026 decía que el responsable validara la v1.38, las especificaciones y las revisiones 34 a 38 de `DESIGN.md` para empezar las fases 1 y 2, que ya se construyeron.
