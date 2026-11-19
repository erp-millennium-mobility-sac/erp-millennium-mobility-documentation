# Mapa de contextos objetivo — ERP Millennium Mobility

**Estado:** propuesta para validar por el responsable (18/11/2026). Hasta que se apruebe, rige el [mapa de contextos](mapa-de-contextos-millennium-mobility.md) vigente; cuando una fase del [plan de migración](plan-de-migracion-a-los-contextos-objetivo-2026-11-18.md) se integra, su parte pasa al mapa vigente y deja de ser propuesta aquí.

**Fuente:** el código de `develop` (backend `ed5adc12` y frontend), el diccionario de datos v1.62, el mapa de contextos vigente y *Domain-Driven Design* de Eric Evans (el «libro azul»). Las referencias «cap. n» son capítulos de ese libro.

Este documento consolida todos los bounded contexts planificados: los que ya existen y se quedan, los que se forman al reordenar lo construido y los que traerá el bloque 12 (clientes, contratos, cobranza y traspasos). Dice qué es cada uno, por qué es un contexto, qué contiene y cómo se relaciona con los demás. Cómo llegar desde lo que hay hoy está en el plan de migración.

---

## 1. Los criterios

Cada frontera de este mapa se decide con seis criterios del libro:

1. **Un bounded context es un modelo con un lenguaje propio** (cap. 2 y 14). Donde las mismas palabras significan otra cosa, o donde se usan palabras que el resto no usa, hay otro modelo.
2. **Las invariantes deciden la frontera** (cap. 6). Lo que debe cumplirse en la misma transacción va en el mismo agregado y en el mismo contexto; lo que solo se consulta puede cruzarla.
3. **El núcleo recibe la inversión** (cap. 15). El *Core Domain* se distingue de los *Generic Subdomains* y de los *Cohesive Mechanisms*, que se aíslan para que no lo contaminen.
4. **Cada relación lleva un patrón explícito** (cap. 14): *Shared Kernel*, *Customer/Supplier*, *Conformist*, *Anticorruption Layer*, *Open Host Service* y *Published Language*.
5. **No se fragmenta sin razón** (cap. 14, *Continuous Integration*). Con un solo equipo, cada frontera cuesta traducción; donde el lenguaje es el mismo, bastan los *Modules* (cap. 5) dentro de un contexto.
6. **Las capas de responsabilidad ordenan las dependencias** (cap. 16, *Responsibility Layers*): un contexto depende de los de su capa o de capas inferiores; cuando uno inferior necesita algo de uno superior, lo pide por un puerto que declara él y que implementa el otro.

En un monolito modular ninguna frontera exige otra base de datos ni otro despliegue: un contexto es un módulo de Spring Modulith con su `package-info.java`, su fachada `::acl` y sus tablas, y `ModularityTests` verifica que nadie cruce por otro lado.

---

## 2. Los contextos

| # | Paquete | Nombre | Tipo (cap. 15) | Responsabilidad | Estado |
|---|---|---|---|---|---|
| 1 | `fleet` | Flota | Núcleo operativo | El ciclo de vida de la unidad y su habilitación | Existe; se reduce |
| 2 | `leasing` | Contratos de leasing | **Núcleo del negocio** | El compromiso con el cliente: contrato, cronograma, entrega, resolución y término | Bloque 12 |
| 3 | `customers` | Clientes | Soporte (núcleo si la evaluación es la ventaja del negocio) | El prospecto, su evaluación y la identidad del cliente | Bloque 12 |
| 4 | `collections` | Cobranza | Núcleo adyacente | Las cuentas por cobrar: cuotas, pagos recibidos, mora y comprobantes de cada cuota | Bloque 12 |
| 5 | `titletransfer` | Traspasos de titularidad | Soporte | El trámite registral del traspaso de una unidad | Bloque 12 |
| 6 | `procurement` | Compras y cuentas por pagar | Soporte | Proveedores, órdenes, facturas, notas, pagos y saldo | Existe; se reordena |
| 7 | `equipment` | Equipamiento | Soporte | Los dispositivos de las unidades (GPS, pantalla) y, después, su monitoreo | Se forma por migración |
| 8 | `financing` | Financiamiento | Soporte | Los fondos y las cesiones de unidades en garantía | Se forma por migración |
| 9 | `organization` | Organización | Soporte | La empresa, su estructura y sus colaboradores | Existe; se ajusta |
| 10 | `iam` | Identidad y acceso | Genérico | Las cuentas de los colaboradores y lo que puede hacer cada una | Existe; se reduce |
| 11 | `reference` (hoy `location`) | Datos de referencia | Genérico | Países, divisiones, direcciones, monedas, tasas y tipos de documento | Existe; renombre opcional |
| 12 | `vehiclecatalog` | Catálogo de vehículos | Genérico | El catálogo del producto: tipos, marcas, modelos y colores | Existe; se reduce |
| 13 | `documents` | Documentos | Mecanismo de plataforma | Los archivos que genera el sistema y los que conserva | Existe |
| 14 | `notifications` | Notificaciones | Mecanismo de plataforma | Los avisos por correo, SMS y push | Existe |
| 15 | `audit` | Bitácora | Genérico | Quién cambió qué y cuándo | Se forma por migración (hoy en `shared`) |
| 16 | `imports` | Cargas de archivos | Genérico | El registro de las cargas y su informe | Se forma por migración (hoy en `shared`) |
| 17 | `reporting` | Reportes | Capa de lectura | Los indicadores y los exportables que cruzan contextos | Existe |
| 18 | `legacyimport` | Migración inicial | *Anticorruption Layer*, temporal | La traducción del libro de stock heredado | Se forma por migración; se borra después de la puesta en marcha |
| — | `shared` | Núcleo compartido | *Shared Kernel* | Lo mínimo que todos comparten | Existe; se reduce |
| — | `customerportal` | Portal del cliente | Aplicación, no contexto | La aplicación del cliente | Bloque 12 |

---

## 3. Cada contexto

### 3.1 `fleet` — Flota

- **Responsabilidad.** La unidad física y su ciclo de vida: alta, estados y caminos con sus motivos, permisos y tolerancias, kilometraje, ubicación, habilitación completa (documentos, instalación de equipos en la unidad, llaves, placas, hitos y la puerta de Disponible), traspaso registrado, anulación e impuesto vehicular.
- **Lenguaje.** Unidad, camino, motivo, tolerancia, puerta, habilitación, hito, Fuera de tienda, Listo para entrega.
- **Agregados y tablas.** `Vehicle` con su historial y sus lecturas (`vehicles`, `vehicle_status_history`, `vehicle_mileage_readings`), `VehicleStatus` y `StatusChangeReason`, `VehicleDocumentType` y `VehicleDocument`, `VehicleEquipment` (la instalación en la unidad), `VehicleKey` y `VehicleTaxDeclaration`.
- **No es suyo.** El inventario de dispositivos (→ `equipment`), los fondos y las cesiones (→ `financing`), el costo y el pago de la unidad (→ `procurement`), la traducción del libro de stock (→ `legacyimport`) y el contrato (→ `leasing`).
- **Ofrece.** `FleetContextFacade`: el alta y el vínculo de las unidades de una factura, las unidades por id, el bloqueo de unidades, la anulación de las devueltas y las transiciones comerciales que llamará `leasing`.
- **Declara puertos.** `UnitPurchases` reducido (valor cero, línea vigente, unidad devuelta), `UnitContracts` (lo implementa `leasing`), `UnitFundAssignments` (lo implementa `financing`) y `UnitTitleTransfers` (lo implementará `titletransfer`).
- **Por qué.** La habilitación es una invariante del estado: Disponible exige la habilitación completa y, mientras la unidad está ahí, no se retira un requisito sin reemplazo (D-164, D-184); eso se cumple en una transacción, así que vive en el mismo contexto que el estado (cap. 6). Los caminos y sus permisos como datos (D-19, D-100) son un *Knowledge Level* (cap. 16).

### 3.2 `leasing` — Contratos de leasing (bloque 12)

- **Responsabilidad.** El contrato con sus condiciones (precio, plazo, inicial, cuota, tasa y frecuencia), la firma, la asignación de la unidad, la entrega, el **cronograma**, la asesora (§5.11 del diccionario), la resolución por incumplimiento y el término.
- **Lenguaje.** Contrato Vigente, Completado o Resuelto (D-253), cuota, cronograma, asesora, resolución. Nunca «venta» al firmar (D-218, D-251): por eso el paquete se llama `leasing` y no `sales`.
- **Agregados previstos.** `Contract` con su cronograma versionado (un refinanciamiento crea una versión nueva sin pisar la anterior) y `AdvisorAssignment` (§5.11).
- **Por qué.** Es el *Core Domain* de un negocio de leasing: el compromiso de donde salen el ingreso y el riesgo (cap. 15). El cronograma es lo que el cliente se comprometió a pagar, así que vive con el contrato; lo que efectivamente pagó es de la cobranza. «El vehículo dura; los contratos pasan» (§5.11): son dos líneas de tiempo, y por eso el contrato no se modela dentro de `fleet`.

### 3.3 `customers` — Clientes (bloque 12)

- **Responsabilidad.** El prospecto, la solicitud, la evaluación y su aprobación; la identidad del cliente (persona o empresa, documento, contacto y dirección por `reference`); el garante; sus documentos; el reparto de prospectos entre asesoras, que no es la asignación del contrato (§5.11).
- **Lenguaje.** Prospecto, solicitud, evaluación, aprobado, garante.
- **Por qué.** Un prospecto tiene su propio ciclo y puede no firmar nunca (cap. 14). Las consultas externas —RENIEC, SUNAT, centrales de riesgo— entran por una *Anticorruption Layer* en su infraestructura. Si la evaluación crediticia llega a ser la ventaja del negocio, pasa a núcleo y puede separarse en un contexto de riesgos.

### 3.4 `collections` — Cobranza (bloque 12)

- **Responsabilidad.** La cuenta corriente del cliente: las cuotas por cobrar que nacen del cronograma, los pagos recibidos y su aplicación, la mora y los días de atraso, la gestión de cobranza, los recordatorios, la conciliación bancaria y los comprobantes electrónicos de cada cuota.
- **Lenguaje.** Recaudo, aplicación del pago, mora, promesa de pago, conciliación.
- **Por qué.** Son **cuentas por cobrar**, el modelo opuesto a las cuentas por pagar de `procurement`, aunque las dos digan «pago»: mezclarlas en un contexto confundiría dos lenguajes (cap. 2). El banco y la facturación electrónica de SUNAT entran por una *Anticorruption Layer*.

### 3.5 `titletransfer` — Traspasos de titularidad (bloque 12)

- **Responsabilidad.** El trámite del traspaso: la minuta, la notaría, la presentación a SUNARP, las observaciones o tachas, la inscripción y la entrega de los documentos al nuevo titular.
- **Por qué.** Tiene su ciclo, su lenguaje y su actor (el área legal y documentaria), y sirve a dos orígenes: el contrato completado y la venta directa (`VENTA_DIRECTA`, D-239). Por eso no cabe dentro de `leasing` ni de `fleet`. `fleet` conserva la transición En traspaso → Inactivo y pregunta por un puerto si el traspaso está inscrito; el titular de la unidad pasa a ser una referencia al cliente, como prevé D-223.

### 3.6 `procurement` — Compras y cuentas por pagar

- **Responsabilidad.** Dos módulos (cap. 5) dentro de un contexto:
  - **Compras:** proveedores con sus contactos, tipos de orden, plantillas con sus documentos y órdenes de compra con su PDF.
  - **Cuentas por pagar:** facturas, notas de crédito, pagos con sus aplicaciones, imputaciones y usos del saldo, Por pagar, y las **cuentas bancarias de la empresa**, que hoy están en `organization` y solo usan los pagos.
- **Ofrece.** Una fachada nueva, `procurement::acl`, con el costo y la posición de pago de cada unidad (*Open Host Service* con *Published Language*), para `reporting`, la ficha de la unidad y, más adelante, la conciliación de la inicial contra el contrato (D-31).
- **Implementa puertos.** `UnitPurchases` de `fleet`, `CurrencyUsage` de `reference` y el de los proveedores de equipos de `equipment`.
- **Por qué un contexto y no dos.** La factura se compara con la orden y el anticipo pagado a una orden pasa a su factura (D-198): son invariantes compartidas (cap. 6). Partirlo en dos contextos agregaría traducción sin dar autonomía a un solo equipo; dos *Modules* hacen explícito el lenguaje de cada parte.

### 3.7 `equipment` — Equipamiento

- **Responsabilidad.** Los dispositivos —GPS y pantalla— con su serie, IMEI, ICCID, sus estados (`AVAILABLE`, `INSTALLED`, `MAINTENANCE`, `DISCARDED`), sus tipos y modelos, su proveedor y, después, el monitoreo del GPS desde su compra hasta su instalación.
- **Agregados y tablas.** `Equipment` (`fleet_equipments`, hoy en `fleet`), `EquipmentType` (`equipment_types`, `equipment_type_vehicle_types`) y `EquipmentModel` (`equipment_models`), hoy en `vehiclecatalog`.
- **Ofrece.** Los tipos y modelos de equipo que lee la habilitación, y la instalación y el retiro de un dispositivo que pide `fleet`.
- **Por qué.** Lenguaje y actores propios (taller, monitoreo). Hoy el inventario todavía no está unido a la instalación (`Equipment#installOnVehicle` no tiene quién lo llame): separarlo antes de construir el monitoreo cuesta mucho menos que después.

### 3.8 `financing` — Financiamiento

- **Responsabilidad.** Los fondos, las cesiones de unidades en garantía y la carga de la lista de garantía mobiliaria.
- **Agregados y tablas.** `Fund` (`funds`) y `VehicleFundAssignment` (`vehicle_fund_assignments`), hoy en `fleet`.
- **Implementa.** El puerto `UnitFundAssignments` de `fleet`, con el que la anulación de una unidad sabe si tiene una cesión abierta (D-182).
- **Por qué.** Otro lenguaje (fondo, cesión, fideicomiso, garantía), otro actor (Contabilidad) y una sola invariante que cruza, resuelta por un puerto. Más adelante puede recibir de la cobranza los flujos que se ceden a un fondo.

### 3.9 `organization` — Organización

- **Responsabilidad.** Empresas, marcas comerciales, sedes, ubicaciones y colaboradores con su historial de ingresos, **incluidas su salida y su regreso**, que hoy ordena `iam`.
- **Declara.** Un puerto para las consecuencias de la salida y el regreso sobre la cuenta del colaborador, que implementa `iam`.
- **Por qué.** Quien es dueño de un agregado es dueño de los comandos que lo cambian (cap. 6). Las cuentas bancarias salen porque solo las usan los pagos: hoy `organization` tiene que preguntarle a `procurement` si una cuenta tiene pagos (`BankAccountPayments`), señal de que el concepto es de cuentas por pagar.

### 3.10 `iam` — Identidad y acceso

- **Responsabilidad.** Las cuentas de los colaboradores, sus credenciales y sesiones, el freno del inicio de sesión, la activación, los roles, los permisos, las asignaciones y los bloqueos.
- **Por qué.** Es un *Generic Subdomain* (cap. 15): se aísla y se congela su alcance. No se estira para los clientes, porque su modelo exige que toda cuenta pertenezca a un colaborador (D-38). Comparte con todos un *Shared Kernel* pequeño: `AccessScope` y `PermissionCodes`.

### 3.11 `reference` — Datos de referencia (hoy `location`)

- **Responsabilidad.** Países con su estructura territorial, divisiones, direcciones dentro de su dueño, monedas, tasas de cambio y tipos de documento tributario.
- **Por qué.** Su *Published Language* son los códigos ISO del país y de la moneda (D-13). El nombre `location` no describe monedas ni tasas y choca con `Location` de `organization`; renombrarlo mejora el lenguaje, pero cambia muchos archivos, así que es opcional.

### 3.12 `vehiclecatalog` — Catálogo de vehículos

- **Responsabilidad.** Tipos de vehículo, marcas, modelos y colores.
- **Cambia.** Entrega los tipos y modelos de equipo a `equipment`, y pregunta por un puerto si un modelo ya tiene unidades, como hace `CurrencyUsage`, en vez de guardar la bandera que hoy le escribe `fleet`.

### 3.13 Plataforma: `documents`, `notifications`, `audit` e `imports`

- **`documents`** y **`notifications`** quedan como están: *Cohesive Mechanisms* (cap. 15) que no conocen ningún contexto.
- **`audit`** es la bitácora (`change_logs`, `GET /change-logs`), hoy en `shared/changelog`.
- **`imports`** es el registro de las cargas (`import_batches`, `/imports`), hoy en `shared/importbatches`.
- **Por qué.** Tienen tabla, endpoint y permiso propios: son *Generic Subdomains*, no núcleo compartido.

### 3.14 `reporting` — Reportes

- **Responsabilidad.** Los indicadores, la brecha de días, la diferencia por inicial y el exportable de stock.
- **Por qué.** Es una capa de lectura que se conforma (*Conformist*) con lo que cada contexto publica. Lee el costo y el pago de las unidades de `procurement::acl`, no de `fleet`, y calcula la brecha de días, que cruza la entrega de `fleet` con el primer pago de `procurement`.

### 3.15 `legacyimport` — Migración inicial (temporal)

- **Responsabilidad.** La traducción del libro de stock de 52 columnas: leerlo, agruparlo y llamar a los servicios de `fleet` y de `procurement`.
- **Por qué.** Es la *Anticorruption Layer* que el libro describe para un sistema heredado (cap. 14): el modelo heredado no contamina el nuevo. Las reglas que sí son de cada contexto —la factura migrada sin la regla de los 20 días (D-177), la unidad registrada en su estado (D-202)— se quedan en el contexto dueño y se exponen por su fachada. El módulo se borra después de la puesta en marcha; quedan los datos y los caminos `MIGRACION_INICIAL`.

### 3.16 `shared` — Núcleo compartido

`Result` y `ApplicationError`, `PageQuery` y `PageResult`, `AccessScope` y `PermissionCodes`, las bases de persistencia, `SingleInstanceRun`, `DeadlockRetry` y `SpreadsheetExporter`. Un *Shared Kernel* tiene que ser pequeño, estable y acordado (cap. 14): por eso salen la bitácora y las cargas.

### 3.17 `customerportal` — Portal del cliente (bloque 12)

La aplicación del cliente: su cronograma, sus pagos, los documentos de su unidad marcados como visibles (`is_visible_to_customer`) y sus trámites. No tiene modelo propio: consume lo que publican los demás contextos, y lleva su propia identidad de cliente, separada de `iam`.

---

## 4. Las relaciones

| Proveedor | Cliente | Patrón (cap. 14) | Qué fluye |
|---|---|---|---|
| `reference` | Todos los de negocio | *Open Host Service* + *Published Language* | País y moneda por su código ISO, tasas, divisiones, tipos de documento y direcciones |
| `organization` | `fleet`, `procurement`, `financing`, `iam`, `leasing` | *Customer/Supplier* | Empresa, sede, marca, ubicación y colaborador (la asesora es un colaborador) |
| `iam` | Todos | *Shared Kernel* | `AccessScope` y `PermissionCodes` |
| `iam` → `organization` | — | Puerto de `organization` | Las consecuencias de la salida y el regreso sobre la cuenta |
| `vehiclecatalog` | `fleet`, `procurement`, `equipment` | *Customer/Supplier* | Modelos, colores y tipos de vehículo |
| `equipment` | `fleet` | *Customer/Supplier* | Tipos de equipo que exige la habilitación; instalar y retirar un dispositivo |
| `fleet` | `procurement` | *Customer/Supplier* | El alta y el vínculo de las unidades de una factura (D-153) |
| `procurement` → `fleet` | — | Puerto `UnitPurchases` de `fleet` | Valor cero, línea vigente, unidad devuelta |
| `procurement` | `reporting`, la ficha de la unidad | *Open Host Service* + *Published Language* | Costo y posición de pago de cada unidad |
| `fleet` | `financing` | *Customer/Supplier* | Las unidades por placa; el bloqueo de la unidad al ceder |
| `financing` → `fleet` | — | Puerto `UnitFundAssignments` | Si la unidad tiene una cesión abierta |
| `customers` | `leasing` | *Customer/Supplier* | Solo un cliente aprobado firma |
| `fleet` | `leasing` | *Customer/Supplier* + *Open Host Service* | Reservar, firmar, entregar, resolver y completar: los caminos comerciales |
| `leasing` → `fleet` | — | Puerto `UnitContracts` | Si la unidad tiene contrato, en vez de deducirlo del historial (D-250) |
| `leasing` | `collections` | *Customer/Supplier* + *Published Language* | El cronograma firmado |
| `collections` → `leasing` | — | Puerto de `leasing` | La mora, para decidir una resolución |
| `leasing` | `titletransfer` | *Customer/Supplier* | Un contrato completado, o una venta directa, abre el trámite |
| `titletransfer` → `fleet` | — | Puerto `UnitTitleTransfers` + *Open Host Service* de `fleet` | En traspaso → Inactivo con el traspaso inscrito |
| `collections` | `financing` | *Customer/Supplier* (futuro) | Los flujos cedidos a un fondo |
| `fleet`, `procurement` | `legacyimport` | *Anticorruption Layer* | Los servicios que llama la traducción del libro heredado |
| Todos | `reporting` | *Conformist* | Lo que cada contexto publica |
| `documents`, `notifications`, `audit`, `imports` | Quien los use | *Cohesive Mechanisms* | Documentos, avisos, bitácora y registro de cargas |

Ninguna relación cierra un ciclo: cuando un contexto de una capa inferior necesita saber algo de uno superior, lo pide por un puerto que declara él y que implementa el otro, el patrón que ya usan `UnitPurchases`, `CurrencyUsage` y `BankAccountPayments`.

---

## 5. Las capas de responsabilidad

```
Soporte a decisiones   reporting
Política               iam
Compromiso             leasing · financing
Operación              fleet · procurement · collections · titletransfer
Potencial              customers · organization · reference · vehiclecatalog · equipment
Plataforma             documents · notifications · audit · imports · shared
Aplicación aparte      customerportal          Temporal: legacyimport
```

Una capa depende de las de abajo (cap. 16): los compromisos se toman sobre lo que se opera, y lo que se opera se apoya en lo que la empresa tiene. Reportes lee de todas.

---

## 6. Lo que no se separa, y por qué

| Tentación | Por qué no |
|---|---|
| La habilitación como contexto propio | Es una invariante del estado de la unidad, en la misma transacción (cap. 6) |
| Compras y cuentas por pagar en dos contextos | Comparten el comprobante y las invariantes entre orden, factura y pago: dos *Modules*, un contexto |
| El impuesto vehicular como contexto | Es pequeño y depende de la placa y la TIVE de la unidad |
| `iam` en autenticación y autorización | Es un solo modelo de cuenta, rol y asignación |
| El cliente antes de tener su modelo | Nace con el bloque 12; separarlo antes sería diseño especulativo |
| La cobranza dentro de `procurement` | Cuentas por cobrar y por pagar son modelos opuestos |

---

## 7. Decisiones abiertas para el responsable

1. **Aprobar este mapa objetivo** y el orden del plan de migración.
2. **Los códigos de los permisos no cambian** con la migración (por ejemplo, `stock.crear_equipamiento` sigue igual aunque su pantalla pase a `equipment`); renombrarlos es una decisión aparte, con su migración de `iam`.
3. **Las cuentas bancarias en la pantalla:** pasan a ser una opción propia de Configuración con `?empresa=`, o siguen como pestaña de la ficha de la empresa prestada por `procurement`, lo que exige una excepción nueva a la regla 1.2 del frontend.
4. **Renombrar `location` a `reference`:** sí o no.
5. **Los requisitos de la habilitación** (`blocks_availability` y a qué tipo de vehículo aplica un equipo): se quedan en el catálogo de equipos o pasan a una configuración de `fleet`, como ya está `vehicle_document_types`.
6. **El bloque 12**, antes de modelarlo:
   - el tipo de contrato: arrendamiento financiero u operativo, o venta a plazos con reserva de dominio (cambia el titular durante el contrato, el IGV y la opción de compra);
   - la evaluación del cliente: propia o de una central de riesgo, y si hay garante;
   - el cronograma: el sistema de cuotas, la mora, los refinanciamientos y los adelantos;
   - quién emite los comprobantes de cada cuota y con qué proveedor de facturación electrónica;
   - cuándo procede la resolución y qué pasa con la deuda y con la unidad;
   - si el regreso automático a Disponible a las 48 horas de la firma es una regla del contrato (la oferta vence) o de la unidad (la reserva vence).
