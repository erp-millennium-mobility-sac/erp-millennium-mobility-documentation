# Mapa de contextos objetivo — ERP Millennium Mobility

**Estado:** aprobado por el responsable el 19/11/2026 (propuesto el 18/11/2026 y revisado el 19/11/2026 con la auditoría del plan). El mismo día resolvió sus decisiones de §7, salvo las del bloque 12: `location` no se renombra y `legacyimport` no se forma. Mientras las fases no se integren, rige el [mapa de contextos](mapa-de-contextos-millennium-mobility.md) vigente; cuando una fase del [plan de migración](plan-de-migracion-a-los-contextos-objetivo-2026-11-18.md) se integra, su parte pasa al mapa vigente y aquí se queda, con la marca «Integrada en la fase n (fecha)».

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
| 9 | `organization` | Organización | Soporte | La empresa, su estructura, sus cuentas bancarias y sus colaboradores | Existe; se ajusta |
| 10 | `iam` | Identidad y acceso | Genérico | Las cuentas de los colaboradores y lo que puede hacer cada una | Existe; se reduce |
| 11 | `location` | Datos de referencia | Genérico | Países, divisiones, direcciones, monedas, tasas y tipos de documento | Existe; no se renombra (§7, decisión 4) |
| 12 | `vehiclecatalog` | Catálogo de vehículos | Genérico | El catálogo del producto: tipos, marcas, modelos y colores | Existe; se reduce |
| 13 | `documents` | Documentos | Mecanismo de plataforma | Los archivos que genera el sistema y los que conserva | Existe |
| 14 | `notifications` | Notificaciones | Mecanismo de plataforma | Los avisos por correo, SMS y push | Existe |
| 15 | `audit` | Bitácora | Genérico | Quién cambió qué y cuándo | Existe: integrada en la fase 1 (19/11/2026) |
| 16 | `imports` | Cargas de archivos | Genérico | El registro de las cargas y su informe | Existe: integrada en la fase 1 (19/11/2026) |
| 17 | `reporting` | Reportes | Capa de lectura | Los indicadores y los exportables que cruzan contextos | Existe |
| 18 | `legacyimport` | Migración inicial | — | La traducción del libro de stock heredado | No se forma: la carga inicial corre antes de la fase 3, y su código se borra de `procurement` y `fleet` después de la puesta en marcha (§3.15; §7, decisión 9) |
| — | `shared` | Núcleo compartido | *Shared Kernel* | Lo mínimo que todos comparten | Existe; se reduce |
| — | `customerportal` | Portal del cliente | Aplicación, no contexto | La aplicación del cliente | Bloque 12 |

---

## 3. Cada contexto

### 3.1 `fleet` — Flota

- **Responsabilidad.** La unidad física y su ciclo de vida: alta, estados y caminos con sus motivos, permisos y tolerancias, kilometraje, ubicación, habilitación completa (documentos, instalación de equipos en la unidad, llaves, placas, hitos y la puerta de Disponible), traspaso registrado, anulación e impuesto vehicular.
- **Lenguaje.** Unidad, camino, motivo, tolerancia, puerta, habilitación, hito, Fuera de tienda, Listo para entrega.
- **Agregados y tablas.** `Vehicle` con su historial y sus lecturas (`vehicles`, `vehicle_status_history`, `vehicle_mileage_readings`), `VehicleStatus` y `StatusChangeReason`, `VehicleDocumentType` y `VehicleDocument`, `VehicleEquipment` (la instalación en la unidad), `VehicleKey`, `VehicleTaxDeclaration` y `VehicleEquipmentRequirement` (`vehicle_equipment_requirements`), la configuración de qué equipo es requisito de la unidad y cuál es de seguridad (D-260).
- **No es suyo.** El inventario de dispositivos (→ `equipment`), los fondos y las cesiones (→ `financing`), el costo y el pago de la unidad (→ `procurement`) y el contrato (→ `leasing`). Su parte de la carga inicial —el alta de las unidades migradas— se borra después de la puesta en marcha (§3.15).
- **Ofrece.** `FleetContextFacade`: el alta y el vínculo de las unidades de una factura, las unidades por id, el bloqueo de unidades, la anulación de las devueltas, la unidad que la sesión ve (`fetchSeenUnit`, para el costo que sirve `procurement`), las unidades que pide `financing` —por id, por identificador público y por placas, con y sin candado—, `fetchReportFacts` (lo que las reglas de `fleet` dicen de las unidades, para `reporting`) y las transiciones comerciales que llamará `leasing`.
- **Declara puertos.** `UnitPurchases` reducido (valor cero, línea vigente, unidad devuelta, y el estado de pago con su fecha base, sin montos, para la brecha de días), `UnitContracts` (lo implementa `leasing`), `UnitFundAssignments` (la cesión abierta y si la sesión puede ceder la unidad, por la opción A de la fase 4; lo implementa `financing`) y `UnitTitleTransfers` (lo implementará `titletransfer`).
- **Implementa puertos.** `EquipmentUnits` de `equipment` (la placa de la unidad donde está un dispositivo), `CurrencyUsage` de `location` (el costo de las instalaciones), `ConfidentialChangeLogFieldsPort` de `audit` y, desde la fase 8 del plan, `VehicleModelUsage` de `vehiclecatalog` y `EquipmentTypeUsage` de `equipment`. Integrada en la fase 8 (21/11/2026, entrega N): `VehicleModelUsageAdapter` y `VehicleEquipmentTypeUsageAdapter` cuentan las unidades y las instalaciones de todas las empresas, y `fleet` ya no avisa a los catálogos al registrar una unidad ni al instalar un equipo (D-282).
- **Por qué.** La habilitación es una invariante del estado: Disponible exige la habilitación completa, y perder un requisito saca a la unidad de Disponible en la misma operación (D-164, D-260); eso se cumple en una transacción, así que vive en el mismo contexto que el estado (cap. 6). Por lo mismo, qué equipo es requisito es política de la habilitación y es de `fleet`, no del catálogo de equipos (D-260). Los caminos y sus permisos como datos (D-19, D-100) son un *Knowledge Level* (cap. 16).

### 3.2 `leasing` — Contratos de leasing (bloque 12)

- **Responsabilidad.** El contrato con sus condiciones (precio, plazo, inicial, cuota, tasa y frecuencia), la firma, la asignación de la unidad, la entrega, el **cronograma**, la asesora (§5.11 del diccionario), la resolución por incumplimiento y el término.
- **Lenguaje.** Contrato Vigente, Completado o Resuelto (D-253), cuota, cronograma, asesora, resolución. Nunca «venta» al firmar (D-218, D-251): por eso el paquete se llama `leasing` y no `sales`.
- **Agregados previstos.** `Contract` con su cronograma versionado (un refinanciamiento crea una versión nueva sin pisar la anterior) y `AdvisorAssignment` (§5.11). El dueño de `advisor_assignments` está por decidir (§7, punto 8): el diccionario la pone en el bloque 3b, de `organization` e `iam`, aunque se construya con el bloque 12 (§3, ajuste 2). Hasta esa decisión manda el diccionario.
- **Por qué.** Es el *Core Domain* de un negocio de leasing: el compromiso de donde salen el ingreso y el riesgo (cap. 15). El cronograma es lo que el cliente se comprometió a pagar, así que vive con el contrato; lo que efectivamente pagó es de la cobranza. «El vehículo dura; los contratos pasan» (§5.11): son dos líneas de tiempo, y por eso el contrato no se modela dentro de `fleet`.

### 3.3 `customers` — Clientes (bloque 12)

- **Responsabilidad.** El prospecto, la solicitud, la evaluación y su aprobación; la identidad del cliente (persona o empresa, documento, contacto y dirección por `location`); el garante; sus documentos; el reparto de prospectos entre asesoras, que no es la asignación del contrato (§5.11).
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

- **Responsabilidad.** Dos lenguajes dentro de un contexto, que hoy comparten paquetes; separarlos en dos *Modules* (cap. 5) no es parte del plan de migración:
  - **Compras:** proveedores con sus contactos, tipos de orden, plantillas con sus documentos y órdenes de compra con su PDF.
  - **Cuentas por pagar:** facturas, notas de crédito, pagos con sus aplicaciones, imputaciones y usos del saldo, y Por pagar.
- **Ofrece.** Una fachada nueva, `procurement::acl`, con el costo de compra de cada unidad por lote, para `reporting` y, más adelante, la conciliación de la inicial contra el contrato (D-31); y la ruta `GET /vehicles/{id}/purchase-cost` para la ficha de la unidad. Es su *Open Host Service* con *Published Language* (cap. 14), pequeño: sin el detalle de los usos del saldo. La carga inicial no pasa por esa fachada: sigue en `procurement` hasta que se borra después de la puesta en marcha (§3.15). Integrada en la fase 5 (20/11/2026): `ProcurementContextFacade` publica `fetchUnitPurchaseCosts`, que lee `reporting`, y `procurement` sirve la ruta con la misma respuesta y el mismo orden de negativas, con la unidad que le dice `fleet` (`fetchSeenUnit`); el puerto `UnitPurchases` quedó reducido (D-280).
- **Implementa puertos.** `UnitPurchases` de `fleet`, `CurrencyUsage` de `location`, `BankAccountUsage` de `organization`, `EquipmentSuppliers` de `equipment` y `ConfidentialChangeLogFieldsPort` de `audit`.
- **Por qué un contexto y no dos.** La factura se compara con la orden y el anticipo pagado a una orden pasa a su factura (D-198): son invariantes compartidas (cap. 6). Partirlo en dos contextos agregaría traducción sin dar autonomía a un solo equipo; dos *Modules* hacen explícito el lenguaje de cada parte.

### 3.7 `equipment` — Equipamiento

- **Responsabilidad.** Los dispositivos —GPS y pantalla— con su serie, IMEI, ICCID, sus estados (`AVAILABLE`, `INSTALLED`, `MAINTENANCE`, `DISCARDED`), sus tipos y modelos, su proveedor y, después, el monitoreo del GPS desde su compra hasta su instalación.
- **Agregados y tablas.** `Equipment` (`fleet_equipments`, hoy en `fleet`), `EquipmentType` (`equipment_types`, `equipment_type_vehicle_types`), `EquipmentBrand` (`equipment_brands`) y `EquipmentModel` (`equipment_models`), hoy en `vehiclecatalog`. Las marcas de equipo llegan con la rama `refactor/remove-equipment-type-flags` (su V139, que es V141 en `feature/d-260`), que se integra con D-260 antes de la fase 2; `EquipmentModel` las lee por su repositorio, así que pasan juntas. Integrada en la fase 2 (20/11/2026): los cuatro agregados y sus cinco tablas son de `equipment`, con la fachada `EquipmentContextFacade` y los puertos `EquipmentSuppliers` y `EquipmentUnits`; `fleet_equipments` conserva su nombre (D-278).
- **Depende de.** `organization` (la empresa y la ubicación de cada dispositivo), `vehiclecatalog` (los tipos de vehículo y lo que la habilitación necesita de cada modelo) y `documents` (las exportaciones del inventario y de los catálogos).
- **Ofrece.** Los tipos de equipo, a qué tipo de vehículo aplica cada uno y si un modelo ya lo trae de fábrica, que lee la habilitación —qué equipo se exige lo decide `fleet` (D-260)— y, cuando llegue el monitoreo, la instalación y el retiro de un dispositivo que pide `fleet`, con la regla de la fase 2 §2.6 del plan, que el responsable aprobó el 19/11/2026 (D-263, D-264): la instalación de `fleet` es la fuente de verdad, solo `fleet` instala, reemplaza y retira, y el motivo del retiro decide adónde vuelve el dispositivo.
- **Declara puertos.** `EquipmentSuppliers` (lo implementa `procurement`), `EquipmentUnits` y, desde la fase 8 del plan, `EquipmentTypeUsage` (los dos los implementa `fleet`), así `equipment` no depende de `fleet`. Integrada en la fase 8 (21/11/2026, entrega N): la corrección del código de un tipo lo bloquea `FOR UPDATE` y pregunta a `EquipmentTypeUsage` si ya se instaló, y la lista calcula `codeLocked` con una pregunta por página. Preguntan solo sus pantallas: su fachada nunca, porque la lee `fleet` dentro de sus transacciones. La entrega N+1 (21/11/2026) borró `equipment_types.has_installations`, que ya nadie escribía (V147, D-282).
- **Por qué.** Lenguaje y actores propios (taller, monitoreo). Hoy el inventario todavía no está unido a la instalación (`Equipment#installOnVehicle` no tiene quién lo llame): separarlo antes de construir el monitoreo cuesta mucho menos que después.

### 3.8 `financing` — Financiamiento

- **Responsabilidad.** Los fondos, las cesiones de unidades en garantía y la carga de la lista de garantía mobiliaria.
- **Agregados y tablas.** `Fund` (`funds`) y `VehicleFundAssignment` (`vehicle_fund_assignments`), hoy en `fleet`. Integrada en la fase 4 (20/11/2026): los dos agregados y sus tablas son de `financing`, con la carga de la garantía, sin fachada propia; `financing` lee y bloquea las unidades por la fachada de `fleet` e implementa su puerto `UnitFundAssignments` (D-279).
- **Implementa.** El puerto `UnitFundAssignments` de `fleet`, con el que la anulación de una unidad sabe si tiene una cesión abierta (D-182) y la ficha sabe si la sesión puede cederla (opción A de la fase 4).
- **Depende de.** `fleet` (las unidades y su candado al ceder y al cerrar, en la misma transacción), `organization`, `iam` (quién registró una cesión), `audit` (el cierre y la carga escriben la bitácora), `imports` (el lote de la carga de garantía) y `documents` (la exportación de los fondos).
- **Por qué.** Otro lenguaje (fondo, cesión, fideicomiso, garantía), otro actor (Contabilidad) y una sola invariante que cruza, resuelta por un puerto. Más adelante puede recibir de la cobranza los flujos que se ceden a un fondo.

### 3.9 `organization` — Organización

- **Responsabilidad.** Empresas, marcas comerciales, sedes, ubicaciones, las cuentas bancarias de la empresa y los colaboradores con su historial de ingresos, **incluidas su salida y su regreso**, que hoy ordena `iam`. Integrada en la fase 7 (21/11/2026): `organization` sirve la salida y el regreso (`EmployeeTenureCommandService`), con sus negativas, y declara `EmployeeAccountConsequences`, que implementa `iam`; las rutas, el permiso, los cuerpos y las negativas no cambiaron (D-281).
- **Declara puertos.** `EmployeeAccountConsequences`, que implementa `iam`: bloquear la cuenta, la guardia del último superadministrador y las consecuencias de la salida y el regreso sobre la cuenta. Las negativas de la operación son de `organization`. Y `BankAccountUsage`, que implementan `procurement` y, después, `collections`: si una cuenta está en uso, para que no cambie su número ni su moneda (ORG-R65). Integrada en la fase 6 (19/11/2026): el puerto se llamaba `BankAccountPayments`, y las cuentas no cambiaron de contexto.
- **Implementa puertos.** `AuditCompanyLookupPort` de `audit`: la empresa y su zona, para la bitácora y las cargas.
- **Por qué.** El caso de uso pertenece al contexto dueño del término (cap. 4 y 14), y el cambio entra por la raíz `Employee` (cap. 6). Las cuentas bancarias se quedan: son de lo que la empresa tiene, y las van a usar los pagos y los cobros, dos contextos hermanos y opuestos; cada uno responde por un puerto, como `CurrencyUsage` (cap. 14 y 16).

### 3.10 `iam` — Identidad y acceso

- **Responsabilidad.** Las cuentas de los colaboradores, sus credenciales y sesiones, el freno del inicio de sesión, la activación, los roles, los permisos, las asignaciones y los bloqueos.
- **Implementa puertos.** `EmployeeAccountConsequences` de `organization` (fase 7 del plan), `AuditActorLookupPort` de `audit` (el nombre de una cuenta) y los puertos del alcance de `shared`: `CurrentAccessScope`, `ProtectedAccounts` y `PendingAccountActivations`. Integrada en la fase 7 (21/11/2026): `EmployeeAccountConsequencesService`, que corre solo dentro de la transacción de `organization`, y `iam` ya no sirve las rutas de la salida y el regreso (D-281).
- **Por qué.** Es un *Generic Subdomain* (cap. 15): se aísla y se congela su alcance. No se estira para los clientes, porque su modelo exige que toda cuenta pertenezca a un colaborador (D-38). Su vocabulario del alcance, `AccessScope` y `PermissionCodes`, vive en `shared`, el *Shared Kernel* de todos.

### 3.11 `location` — Datos de referencia

- **Responsabilidad.** Países con su estructura territorial, divisiones, direcciones dentro de su dueño, monedas, tasas de cambio y tipos de documento tributario.
- **Declara puertos.** `CurrencyUsage`, que implementan `procurement` y `fleet` y, con el bloque 12, `leasing` y `collections`: si algún monto usa una moneda, antes de cambiar sus decimales (LOC-R05).
- **Por qué.** Su *Published Language* son los códigos ISO del país y de la moneda (D-13). El nombre `location` no describe monedas ni tasas y choca con `Location` de `organization`; renombrarlo a `reference` mejoraría el lenguaje, pero cambia muchos archivos, y el responsable decidió el 19/11/2026 no renombrarlo (§7, decisión 4).

### 3.12 `vehiclecatalog` — Catálogo de vehículos

- **Responsabilidad.** Tipos de vehículo, marcas, modelos y colores.
- **Cambia.** Entrega los tipos, las marcas y los modelos de equipo a `equipment`, y pregunta por un puerto si un modelo ya tiene unidades, como hace `CurrencyUsage`, en vez de guardar la bandera que hoy le escribe `fleet`. Integrada en la fase 2 (20/11/2026) la entrega del equipamiento: su fachada responde ahora los tipos de vehículo en lote y, de cada modelo, su tipo, si es a gas y si trae la pantalla de fábrica (D-278). Integrado en la fase 8 (21/11/2026, entrega N) el puerto de las unidades: `vehiclecatalog` declara `VehicleModelUsage`, que implementa `fleet`, la corrección del tipo bloquea el modelo `FOR UPDATE` antes de preguntar, y la lista calcula `vehicleTypeLocked` con una pregunta por página. Preguntan solo sus pantallas: su fachada nunca, porque la leen otros contextos dentro de sus transacciones. La entrega N+1 (21/11/2026) borró `vehicle_models.has_units`, que ya nadie escribía (V147, D-282).

### 3.13 Plataforma: `documents`, `notifications`, `audit` e `imports`

- **`documents`** y **`notifications`** quedan como están: *Cohesive Mechanisms* (cap. 15) que no conocen ningún contexto.
- **`audit`** es la bitácora (`change_logs`, `GET /change-logs`). Integrada en la fase 1 (19/11/2026): salió de `shared/changelog` (D-262).
- **`imports`** es el registro de las cargas (`import_batches`, `/imports`), con su lector y su escritor de libros. Integrada en la fase 1 (19/11/2026): salió de `shared/importbatches` (D-262). Depende de `audit` (los nombres de quien cargó y de la empresa) y de `documents` (donde guarda el archivo y el informe).
- **Por qué.** Tienen tabla, endpoint y permiso propios: son *Generic Subdomains*, no núcleo compartido.

### 3.14 `reporting` — Reportes

- **Responsabilidad.** Los indicadores, la brecha de días, la diferencia por inicial y el exportable de stock.
- **Por qué.** Es una capa de lectura que se conforma (*Conformist*) con lo que cada contexto publica. Lee el costo de las unidades de `procurement::acl`, no de `fleet`, y la brecha de días de `fleet::acl`, que la calcula con la misma regla que la ficha (D-232; opción A de la fase 5). Además lee tablas de los demás por SQL, una integración por base compartida que sostienen sus pruebas. Integrada en la fase 5 (20/11/2026): `reporting` lee el costo por su `ExternalProcurementService`, y sus reportes no cambiaron (D-280).

### 3.15 `legacyimport` — No se forma

Iba a ser la *Anticorruption Layer* que el libro describe para un sistema heredado (cap. 14): la traducción del libro de stock de 52 columnas, fuera de `procurement` y de `fleet`. El responsable decidió el 19/11/2026 que la carga inicial corre antes de la fase 3 del plan (§7, decisión 9), así que esa capa no se construye: el libro se lee una sola vez con el código de hoy, en `procurement` y en `fleet`, y ese código se borra después de la puesta en marcha (borrado 3b del plan). Quedan los datos y los caminos `MIGRACION_INICIAL`.

### 3.16 `shared` — Núcleo compartido

`Result` y `ApplicationError`, `PageQuery`, `PageResult` y `TextIdentifier`, `AccessScope` con sus alcances y `PermissionCodes`, los puertos del alcance (`CurrentAccessScope`, `ProtectedAccounts` y `PendingAccountActivations`, que implementa `iam`, y `SystemAccessScope`, que implementa el propio `shared`), `@PermissionRequired`, las bases de persistencia, `JdbcDataVersions`, `SingleInstanceRun`, `DeadlockRetry` y `SpreadsheetExporter`. `ProtectedAccounts` y `PendingAccountActivations` son vocabulario de `iam`: se quedan, porque `organization` sigue calculando las acciones de cuenta de cada fila de Colaboradores (decisión 14 de la fase 0 del plan, 19/11/2026). Un *Shared Kernel* tiene que ser pequeño, estable y acordado (cap. 14): por eso salen la bitácora y las cargas.

### 3.17 `customerportal` — Portal del cliente (bloque 12)

La aplicación del cliente: su cronograma, sus pagos, los documentos de su unidad marcados como visibles (`is_visible_to_customer`) y sus trámites. No tiene modelo propio: consume lo que publican los demás contextos. La identidad de acceso del cliente —credenciales, sesiones y activación— va separada de `iam`, pero es un modelo con tablas y no cabe aquí: su dueño, `customers` o un contexto genérico nuevo, se decide con el bloque 12 (§7, punto 8), y entonces se suma a §2, §3 y §4.

---

## 4. Las relaciones

| Proveedor | Cliente | Patrón (cap. 14) | Qué fluye |
|---|---|---|---|
| `location` | Todos los de negocio | *Open Host Service* + *Published Language* | País y moneda por su código ISO, tasas, divisiones, tipos de documento y direcciones |
| `organization` | `fleet`, `procurement`, `financing`, `equipment`, `iam`, `leasing`, `customers`, `collections` | *Customer/Supplier* | Empresa, sede, marca, ubicación, cuenta bancaria y colaborador (la asesora es un colaborador) |
| `shared` | Todos | *Shared Kernel* | `AccessScope` y `PermissionCodes`, vocabulario de `iam` que vive en `shared` (regla 6.1) |
| `iam` | `fleet`, `financing` | *Customer/Supplier* | El nombre de quien creó o cambió una unidad o registró una cesión; para `fleet`, además, la cuenta `SYSTEM` del regreso automático y el permiso de cada camino (D-42, D-100). Es la opción B de la decisión 6 del mapa vigente |
| `iam` → `organization` | — | Puerto `EmployeeAccountConsequences` de `organization` | Bloquear la cuenta, la guardia del último superadministrador y las consecuencias de la salida y el regreso |
| `procurement` → `organization` | — | Puerto `BankAccountUsage` de `organization` | Si una cuenta tiene pagos |
| `collections` → `organization` | — | Puerto `BankAccountUsage` (futuro) | Si una cuenta tiene cobros |
| `fleet`, `procurement` → `location` | — | Puerto `CurrencyUsage` de `location` | Si algún monto usa una moneda, antes de cambiar sus decimales (LOC-R05) |
| `audit` | `imports` y los contextos que escriben la bitácora | *Generic Subdomain*, contrato de plataforma | La bitácora y los nombres de quien actuó y de la empresa |
| `iam`, `organization` → `audit` | — | Puertos `AuditActorLookupPort` y `AuditCompanyLookupPort` de `audit` | El nombre de una cuenta; la empresa y su zona |
| `fleet`, `procurement` → `audit` | — | Puerto `ConfidentialChangeLogFieldsPort` de `audit` | Los campos de la bitácora que solo ve quien tiene su permiso |
| `vehiclecatalog` | `fleet`, `procurement`, `equipment` | *Customer/Supplier* | Modelos, colores y tipos de vehículo |
| `equipment` | `fleet` | *Customer/Supplier* | Tipos de equipo, su aplicabilidad y la pantalla de fábrica, que lee la habilitación; más adelante, instalar y retirar un dispositivo |
| `fleet` → `vehiclecatalog` | — | Puerto `VehicleModelUsage` de `vehiclecatalog` (fase 8 del plan) | Si un modelo tiene unidades |
| `fleet` → `equipment` | — | Puerto `EquipmentUnits` de `equipment` | La placa de la unidad donde está un dispositivo |
| `fleet` → `equipment` | — | Puerto `EquipmentTypeUsage` de `equipment` (fase 8 del plan) | Si un tipo de equipo tiene instalaciones |
| `procurement` → `equipment` | — | Puerto `EquipmentSuppliers` de `equipment` | El proveedor de un dispositivo |
| `fleet` | `procurement` | *Customer/Supplier* | El alta y el vínculo de las unidades de una factura (D-153); la unidad que la sesión ve, para el costo |
| `procurement` → `fleet` | — | Puerto `UnitPurchases` de `fleet` | Valor cero, línea vigente, unidad devuelta, y el estado de pago con su fecha base, sin montos |
| `procurement` | `reporting`, la ficha de la unidad | *Open Host Service* + *Published Language* | El costo de compra de cada unidad |
| `fleet` | `financing` | *Customer/Supplier* | Las unidades por id, por identificador público y por placas; su bloqueo al ceder y al cerrar |
| `financing` → `fleet` | — | Puerto `UnitFundAssignments` | Si la unidad tiene una cesión abierta y si la sesión puede cederla (opción A de la fase 4) |
| `customers` | `leasing` | *Customer/Supplier* | Solo un cliente aprobado firma |
| `fleet` | `leasing` | *Customer/Supplier* + *Open Host Service* | Reservar, firmar, entregar, resolver y completar: los caminos comerciales |
| `leasing` → `fleet` | — | Puerto `UnitContracts` | Si la unidad tiene contrato, que se suma a lo que dice su historial (D-250) |
| `leasing` | `collections` | *Customer/Supplier* + *Published Language* | El cronograma firmado |
| `collections` → `leasing` | — | Puerto de `leasing` | La mora, para decidir una resolución |
| `leasing` | `titletransfer` | *Customer/Supplier* | Un contrato completado, o una venta directa, abre el trámite |
| `titletransfer` → `fleet` | — | Puerto `UnitTitleTransfers` + *Open Host Service* de `fleet` | En traspaso → Inactivo con el traspaso inscrito |
| `collections` | `financing` | *Customer/Supplier* (futuro) | Los flujos cedidos a un fondo |
| Todos | `reporting` | *Conformist* | Lo que cada contexto publica |
| `documents`, `notifications` | Quien los use | *Cohesive Mechanisms* | Documentos y avisos |
| `audit`, `imports` | Quien los use | *Generic Subdomains*, contratos de plataforma | Bitácora y registro de cargas |

Ninguna relación cierra un ciclo: cuando un contexto de una capa inferior necesita saber algo de uno superior, lo pide por un puerto que declara él y que implementa el otro, el patrón que ya usan `UnitPurchases`, `CurrencyUsage` y `BankAccountUsage`. La consistencia sincrónica entre agregados de dos contextos —la cesión y la anulación, la salida y la cuenta, el dispositivo y su instalación— es una excepción deliberada al consejo del cap. 6: la justifica una invariante que no admite una ventana, y la permite una sola base con un orden fijo de candados.

---

## 5. Las capas de responsabilidad

```
Soporte a decisiones   reporting
Compromiso             leasing · collections · titletransfer · financing
Operación              fleet · procurement
Potencial              customers · organization · iam · location · vehiclecatalog · equipment
Plataforma             documents · notifications · audit · imports · shared
Aplicación aparte      customerportal
```

Un contexto depende de los de su capa o de las de abajo (cap. 16, criterio 6): los compromisos se toman sobre lo que se opera, y lo que se opera se apoya en lo que la empresa tiene. Reportes lee de todas. `iam` va en la capa de potencial: la operación y el compromiso lo leen (el nombre de una cuenta, la cuenta `SYSTEM`, el permiso de cada camino), así que no puede estar encima de ellos. `collections` y `titletransfer` van con `leasing`, porque dependen del contrato: el cronograma firmado, la mora y el contrato completado (§4).

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

1. **Aprobar este mapa objetivo** y el orden del plan de migración. **Resuelta el 19/11/2026:** aprobados.
2. **Los códigos de los permisos no cambian** con la migración (por ejemplo, `stock.crear_equipamiento` sigue igual aunque su pantalla pase a `equipment`); renombrarlos es una decisión aparte, con su migración de `iam`. **Resuelta el 19/11/2026:** no cambian.
3. **Las cuentas bancarias:** cerrada por la auditoría del plan. Se quedan en `organization`, con su pantalla en la ficha de la empresa, y cada consumidor responde por `BankAccountUsage`.
4. **Renombrar `location` a `reference`:** sí o no. **Resuelta el 19/11/2026: no.** El contexto se llama `location` en todas las secciones de este mapa, y la fase 9 sale del plan (decisiones 2 y 16 de su fase 0).
5. **Los requisitos de la habilitación:** cerrada (D-260). Qué equipo es requisito pasa a una configuración de `fleet`, como `vehicle_document_types`, con dos marcas: si se exige para pasar a Disponible y si es un equipo de seguridad, que una unidad con un cliente no pierde. La aplicabilidad y la pantalla de fábrica se quedan con el catálogo de equipos, en `equipment`. Construida antes de la fase 2 del plan, en dos entregas: la N, y la N+1, que borró `equipment_types.blocks_availability` (V143).
6. **Las opciones de las fases 4 y 5 del plan:** quién calcula si la sesión puede ceder una unidad, quién dibuja la pestaña de cesiones de la ficha y quién es dueño de la brecha de días. **Resuelta el 19/11/2026 con las recomendadas, A, A1 y A:** `fleet` lo pregunta a `financing` por su puerto, `fleet` dibuja la pestaña con `FinancingFacade` y la brecha sigue en `fleet`. El recurso de la unidad no cambia. Las dos de la fase 4 se construyeron con ella (20/11/2026), y la de la fase 5 el mismo día: la brecha la calcula `fleet`, y `reporting` se la pide por `fetchReportFacts`.
7. **El destino de un dispositivo al retirarlo de una unidad,** para cuando el monitoreo una el inventario a la instalación, y qué pasa con un dispositivo instalado cuando la unidad se anula o pasa a Inactivo. **Resuelta el 19/11/2026:** `falla` lo lleva a mantenimiento, `robo` a baja, y `reemplazo` y `sin_falla`, un cuarto motivo para todo retiro de un dispositivo sano, a disponible (D-263); y una unidad con dispositivos instalados no se anula ni pasa a Inactivo hasta que se retiran (D-264), lo que alcanza a todo GPS porque los ya instalados se enlazan a su dispositivo al poner en marcha el monitoreo. Se construyen con el monitoreo.
8. **El bloque 12**, antes de modelarlo:
   - el tipo de contrato: arrendamiento financiero u operativo, o venta a plazos con reserva de dominio (cambia el titular durante el contrato, el IGV y la opción de compra);
   - la evaluación del cliente: propia o de una central de riesgo, y si hay garante;
   - el cronograma: el sistema de cuotas, la mora, los refinanciamientos y los adelantos;
   - quién emite los comprobantes de cada cuota y con qué proveedor de facturación electrónica;
   - cuándo procede la resolución y qué pasa con la deuda y con la unidad;
   - si el regreso automático a Disponible a las 48 horas de la firma es una regla del contrato (la oferta vence) o de la unidad (la reserva vence);
   - el GPS al terminar el contrato: **resuelto el 19/11/2026:** al traspasar la unidad al cliente, el GPS se retira antes de cerrar el traspaso y vuelve al inventario, y el cliente pasa por el taller (D-264);
   - el dueño de `advisor_assignments`: o una decisión la pasa al bloque 12 y a `leasing`, o este mapa la devuelve a `organization` (§3.2);
   - el dueño de la identidad de acceso del cliente: `customers` o un contexto genérico nuevo (§3.17).
9. **La carga inicial:** si se ejecuta antes de la fase 3 del plan, la fase 3 se reemplaza por borrar su código después de la puesta en marcha. **Resuelta el 19/11/2026: antes de la fase 3.** La fase 3 se reemplaza por el borrado 3b, y `legacyimport` no se forma (§3.15).
