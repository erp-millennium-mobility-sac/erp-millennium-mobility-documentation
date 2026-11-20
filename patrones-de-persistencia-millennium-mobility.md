# Patrones de persistencia — ERP Millennium Mobility

**Fuente:** skill `backend-module` (§1 y pasos de construcción; `module-template.md`), `architecture-rules.md` (reglas 2.1, 3.1 y 5.1 a 5.7), el código de `shared/infrastructure/persistence` y de los adaptadores de cada contexto, y el diccionario de datos (D-42). Si este documento y el diccionario no coinciden, manda el diccionario.

Todos los agregados del backend se guardan de la misma manera: el modelo de negocio en `domain`, la tabla en `infrastructure`. Este documento explica qué hace cada pieza, cómo se guarda una edición sin perder datos y por qué no hay excepciones para los catálogos.

## La regla en una frase

> El dominio es solo dominio. JPA y Spring Data son tecnología de persistencia y viven en `infrastructure`, en todos los contextos, catálogos incluidos.

## Las piezas de cada agregado

| Pieza | Capa | Para qué existe |
|---|---|---|
| **Agregado** | `domain/model` | Contiene las reglas del negocio y nada más. Sin anotaciones de JPA, sin constructor vacío para Hibernate, sin setters para el mapeo. Tiene dos constructores: el que crea uno nuevo, validando, y el que lo reconstituye desde la base |
| **Interfaz de repositorio** | `domain/repositories` | Dice qué necesita el dominio para guardar y leer, en sus términos y solo eso. No tiene `delete`: las filas no se borran |
| **Entidad de persistencia** | `infrastructure/persistence/jpa/entities` | Tiene la forma de la tabla: columnas, llaves foráneas y relaciones. Extiende la clase base que corresponde a sus columnas de registro, o ninguna (ver "Lo que vale siempre") |
| **Ensamblador** | `infrastructure/persistence/jpa/assemblers` | Reconstituye el agregado desde la fila (`toDomainFromPersistence`) y copia los valores del agregado sobre una fila que recibe (`copyToPersistence`). Nunca arma la fila |
| **Repositorio de Spring Data** | `infrastructure/persistence/jpa/repositories` | Las consultas a la tabla. Solo lo usa el adaptador |
| **Adaptador** | `infrastructure/persistence/jpa/adapters` | Implementa la interfaz del dominio con el repositorio de Spring Data |

Lo demás no cambia: comandos y consultas como `record`, servicios de comandos que devuelven `Result`, recursos REST con sus transformadores y controladores sin lógica. Los servicios solo conocen la interfaz del dominio.

## Ejemplo: la moneda

**Dominio.** Una clase simple con sus reglas.

```java
public class Currency {                       // sin @Entity
    private Long id;                          // null hasta guardarla
    private UUID publicId;
    private final String isoCode;             // "PEN", no cambia nunca
    private String name;
    private short decimalPlaces;              // de 0 a 2
    private Instant createdAt;                // los registra el sistema

    public Currency(String isoCode, String name, String symbol, Short decimalPlaces) { /* valida */ }
    public Currency(Long id, UUID publicId, /* … */ Instant createdAt, Instant updatedAt) { /* reconstituye */ }
    public void update(String name, String symbol, Short decimalPlaces, boolean amountsUseIt) { /* valida y cambia */ }
}

public interface CurrencyRepository {         // en domain/repositories
    Currency save(Currency currency);
    Optional<Currency> findByIsoCode(String isoCode);
    Optional<Currency> findByPublicIdForUpdate(UUID publicId);
}
```

**Infraestructura.** La tabla y cómo se guarda.

```java
@Entity @Table(name = "currencies")
public class CurrencyPersistenceEntity extends AuditableAbstractPersistenceEntity {
    private UUID publicId;
    private String isoCode;
    private String name;
    // … columnas
}

@Repository
public class CurrencyRepositoryImpl implements CurrencyRepository {
    public Currency save(Currency currency) {
        var row = currency.getId() == null
                ? new CurrencyPersistenceEntity()                     // nueva
                : jpa.findById(currency.getId()).orElseThrow();      // la que ya está en la base
        CurrencyPersistenceAssembler.copyToPersistence(currency, row);
        try {
            return CurrencyPersistenceAssembler.toDomainFromPersistence(jpa.saveAndFlush(row));
        } catch (DataIntegrityViolationException e) {
            throw UniqueViolations.translated(e, NATURAL_KEYS);       // el código ISO repetido, en términos del dominio
        }
    }
}
```

## Cómo se guarda una edición

**El adaptador nunca arma una fila nueva para un agregado que ya existe.** Carga la fila de la base y le copia los cambios.

La razón: si armara la fila desde cero en cada guardado, con el id pero sin la fecha de creación —que la registra el sistema y el dominio no asigna—, la base conservaría el valor, porque la columna no se actualiza, pero la respuesta de `PUT` y `PATCH` saldría con `createdAt: null`. Cargando la fila, los valores que registró el sistema (`created_at` y, donde exista, `created_by`) siguen ahí.

La regla, en concreto:

- El adaptador decide la fila: una nueva si el agregado no tiene `id`, o la que carga de la base si lo tiene. Después le pide al ensamblador que copie y guarda esa fila.
- El ensamblador no tiene un método que construya la fila desde el agregado ni llama a `setId`. Copia solo los valores que el dominio posee; el identificador, las fechas de registro y el autor quedan como los tiene la fila.
- Los hijos se sincronizan por su `id`: al que existe se le copian sus cambios sobre su propia fila y solo un hijo nuevo recibe una fila nueva. Ninguno se quita de la colección, porque las filas no se borran.
- Una referencia a una fila que el agregado no posee, como el rol de una asignación, apunta a la fila que la base ya tiene (`findById`), nunca a una reconstruida desde el dominio.

`CurrencyAuditTests`, `CountryAuditTests` y `UserAuditTests` prueban que una edición conserva la fecha de creación (y el autor, en la cuenta). Cada agregado nuevo necesita una prueba igual.

## Una entidad dentro del agregado

Los escalones de un país viven dentro de `Country` (D-45). El país decide si admite otro, en qué posición va y cómo se renombra, y se guardan con él:

- `CountryPersistenceEntity` mapea sus escalones con `@OneToMany` y `cascade` de `PERSIST` y `MERGE`. Nunca `REMOVE` ni `orphanRemoval`: las filas no se borran.
- El ensamblador sincroniza los hijos por su `id`. Al que existe le copia el nombre y al nuevo le crea su fila, con el país y la posición.
- Las lecturas del país cargan sus escalones en la misma consulta (`@EntityGraph`), para no hacer una consulta por país.
- Para leer o renombrar un escalón suelto, el repositorio busca el país que lo declara.

`GeoDivisionLevelAuditTests` prueba que un escalón se guarda a través de su país y conserva su fecha de creación al renombrarse.

## Leer y después cambiar: primero se bloquea

Una operación que lee un estado y decide con él, y después lo cambia, puede correr al mismo tiempo que otra que decide con el mismo estado. Las dos pasan sus revisiones y la segunda pisa a la primera: dos bloqueos de la misma cuenta que se cruzan, un acceso que se otorga durante la salida del colaborador, dos salidas de la misma persona en dos empresas que no cierran la cuenta.

La regla:

- **Se bloquea la fila antes de leer.** El repositorio del dominio ofrece la lectura que bloquea, con el sufijo `ForUpdate` (`findByIdForUpdate`, `findByPublicIdForUpdate`): primero un `SELECT … FOR NO KEY UPDATE` sobre la fila, que dura hasta que termina la transacción, y después la lectura del agregado. `NO KEY` deja pasar a quien solo inserta filas que apuntan a ella, como un período nuevo de otra cuenta. La segunda operación espera a la primera y decide con lo que la primera dejó.
- **Se bloquea la fila que representa lo que se protege, aunque se vaya a cambiar otra tabla.** Los bloqueos de una cuenta viven en `user_access_suspensions`, pero se bloquea la fila de la cuenta en `users` y después se leen sus períodos. Lo mismo para otorgar y retirar roles, dar acceso, restablecer la contraseña y la salida y el regreso. En compras, un contacto cambia solo por su proveedor: agregarlo, corregirlo, activarlo o desactivarlo bloquea la fila del proveedor con `FOR NO KEY UPDATE`, y emitir una orden lee el proveedor con sus contactos en modo compartido (abajo). La llave foránea de la orden toma sobre el contacto solo `FOR KEY SHARE`, que la actualización del contacto no espera: sin el candado del proveedor, una orden saldría con un contacto retirado al mismo tiempo.
- **Siempre en el mismo orden.** Una operación que toca un colaborador y su cuenta bloquea primero `employees` y después `users`; varias unidades se bloquean por `id` ascendente. Con un orden fijo, dos operaciones no se esperan la una a la otra para siempre.
- **Lo que se referencia y se revisa activo se lee en modo compartido.** Una operación que escribe un registro que apunta a otro, y antes revisa que el otro siga activo, lo lee con `FOR SHARE` (`findByPublicIdForShare`): espera a una edición o una desactivación en curso, y la hace esperar hasta terminar, sin frenar a otra operación que también solo lo referencia. Sin él, una desactivación confirmada entre la revisión y el `INSERT` dejaría pasar el registro, porque sus llaves foráneas toman solo `FOR KEY SHARE`. Así lo hace `organization` con la empresa, la sede y la marca que referencia lo que se cuelga de ellas (D-84), y `fleet` con el fondo al ceder una unidad (D-133): una desactivación simultánea del fondo espera a la cesión o la hace esperar. La corrección del código del fondo, que bloquea el fondo para actualizar, sabe así si alguna cesión lo usa; todo escritor de cesiones toma el mismo bloqueo. `procurement` hace lo mismo con el tipo de orden al crear una plantilla o cambiarle el tipo: el `FOR KEY SHARE` de la llave foránea no choca con el `FOR NO KEY UPDATE` de la edición del tipo, así que sin el `FOR SHARE` la corrección del código no vería una plantilla que se confirma a la vez. Emitir una orden de compra lee así todo lo que revisa activo: el proveedor con sus contactos (`SupplierRepository.findByPublicIdForShare`), la empresa y la marca por `resolveScope` de la fachada de `organization`, como `fleet` las resuelve para un fondo o una unidad, y la moneda por `fetchCurrencySummaryByIsoCodeForShare` de la de `location`. Lo mismo hacen la factura con su empresa, el pago con su cuenta bancaria y las unidades nuevas de un lote con sus marcas, por los miembros `…ForShare` de la fachada de `organization`, y el costo de un equipo con su moneda. Toda escritura que nombra una cuenta bancaria la lee así, con `fetchBankAccountForShare`, también la que traiga `collections`: lo pide el contrato de `BankAccountUsage` a quien lo implementa, y así la corrección del número o de la moneda, que bloquea la cuenta para actualizar, ve esa escritura o la espera. El modelo de cada línea sigue en una lectura simple: `vehiclecatalog` no ofrece una lectura con bloqueo, y `fleet` lee igual el modelo de una unidad.
- **Lo que se lee es la fila de después del bloqueo.** Si la transacción ya había leído esa fila antes, Hibernate la devuelve de su memoria, con los valores de antes. La lectura que bloquea refresca la entidad (`entityManager.refresh`), o la lee sin pasar por la memoria de la transacción, para que la decisión se tome con lo que hay en la base. Por eso la consulta nativa que bloquea devuelve solo el identificador, y la entidad se vuelve a leer y se refresca después: es el mismo patrón con `FOR NO KEY UPDATE` y con `FOR SHARE`. Se prueba con una operación en la que quien pide es también el afectado, que es cuando la transacción ya tenía la fila.
- **Cuando lo protegido no es una fila, se usa un bloqueo consultivo**, de la transacción (`pg_advisory_xact_lock`) salvo donde se dice, que se suelta solo al terminar la transacción. La protección del último superadministrador cuenta cuentas: no hay una fila que bloquear para que dos retiros de dos superadministradores distintos no pasen a la vez, y una llave fija hace que esas operaciones vayan de a una. La serie de un tipo de cambio —país, par de monedas y tipo— se bloquea por su nombre, porque la tasa del día anterior puede no existir todavía; y el pedido de un pago, por su identificador, para que un reintento del mismo envío no registre dos pagos. Una carga de archivos lo intenta por su tipo y su empresa, sin esperar: una segunda carga del mismo tipo en la misma empresa responde en el acto en vez de esperar, porque una espera larga terminaría en el corte de Cloudflare (`ImportLocks`, SHR-R10); la que corre en varias transacciones lo toma en una conexión propia, por toda la carga. Un proceso programado corre en una sola instancia con un bloqueo consultivo de la sesión, que tampoco espera (`SingleInstanceRun`).
- **Una migración que revisa y después cambia también bloquea primero.** Si revisa unas filas y decide con ellas —que normalizar los documentos no repita uno, que ninguna dirección esté fuera del país de su dueño—, bloquea antes las tablas contra escrituras (`LOCK TABLE … IN SHARE ROW EXCLUSIVE MODE`), así nadie cambia una fila entre la revisión y el cambio; las lecturas siguen. V76 bloquea `suppliers`; V78, `addresses` y las tablas de sus dueños. Y no deja la aplicación esperando detrás de ella: una migración abre con `SET LOCAL lock_timeout = '5s'`, y una que espera un candado más de cinco segundos falla, sin cambiar nada, en vez de encolar los pedidos.
- **La base sigue siendo la última línea.** Si una restricción de la base rechaza igual lo que una carrera dejó pasar, el adaptador traduce la violación a una excepción del dominio —`UniqueViolations.keyOf`, de `shared`, encuentra la restricción por su nombre, que PostgreSQL escribe igual en cualquier idioma— y el servicio responde el mismo fallo esperado que su propia revisión (por ejemplo, `AccessSuspensionOverlapException` y `409 ACCESS_SUSPENSION_CONFLICT`), nunca un `500`. Un duplicado que ningún adaptador traduce responde el genérico `409 RESOURCE_CONFLICT`, y el registro nombra la restricción, sin los valores de la llave, para que un defecto como dos órdenes con el mismo número no pase sin rastro (`GlobalExceptionHandlerTest`).

`AccountLockTests`, `EmployeeLockTests` y `SuperadministratorsLockTests` prueban que una segunda transacción espera a la primera. En compras, `SupplierLockTests`, `PurchaseOrderTypeLockTests`, `PurchaseOrderTemplateLockTests` y `PurchaseOrderLockTests` prueban que la lectura que bloquea trae la fila como la encuentra el candado (también la serie que numera una orden); el tipo de orden se lee con `FOR NO KEY UPDATE` y se refresca antes de renombrarlo y antes de cambiar su estado, así un cambio no pisa el que otra petición confirmó en el medio; y una anulación y un envío de la misma orden en paralelo la dejan anulada. `FundLockTests` prueba el modo compartido del fondo al ceder, `CompanySharedLockTests` el de la empresa y `BankAccountLockTests` el de la cuenta de un pago.

**En compras**, además:

- El proveedor se lee en modo compartido, o se bloquea por uno de sus contactos, y llega como lo encuentra el candado (`SupplierLockTests`).
- Emitir una orden retiene en modo compartido lo que revisa, y un proveedor, una moneda o una marca desactivados después de una primera lectura rechazan la orden sin escribir nada (`PurchaseOrderIssuingLockTests`). En la carrera entre el retiro de un contacto y la orden que lo nombra, la orden que llega durante el retiro lo espera, ve el contacto retirado y responde `SUPPLIER_CONTACT_INACTIVE` sin guardar nada, y el retiro que llega durante la orden espera a que termine.
- La corrección de los decimales de una moneda y una orden en esa moneda no se cruzan: la corrección espera a la orden abierta, que la hace contar, y la orden espera a la corrección abierta y revisa sus precios con los decimales nuevos (`CurrencyDecimalPlacesLockTests`; [mapa de contextos](mapa-de-contextos-millennium-mobility.md), §6.6).

**En `fleet`**, además:

- **Una regla que se revisa en los dos lados se serializa en los dos.** La puerta de la habilitación junta dos filas del catálogo: un estado que la exige y un camino que mueve solas las unidades hacia él, que no pueden estar juntos salvo el regreso desde Contrato firmado (D-164, D-170). **Guardar o reabrir un camino lee su destino automático `FOR SHARE`** (`VehicleStatusRepository.findByIdForShare`, `findByPublicIdForShare`) hasta guardar el camino, como lo que se referencia y se revisa activo; y **marcar un estado, o cambiar su actividad, lo lee `FOR NO KEY UPDATE`** (`findByPublicIdForUpdate`) antes de leer sus caminos. `FOR NO KEY UPDATE` choca con `FOR SHARE` como `FOR UPDATE`, así que el segundo pedido espera al primero y decide con lo que confirmó; y deja pasar los movimientos de unidades hacia el estado, que solo toman `FOR KEY SHARE` por su llave foránea. `StatusCatalogLockTests` juega los dos órdenes con dos transacciones reales.
- **Los candados de las unidades.** La fachada de `fleet` bloquea las unidades de una nota por `id` ascendente con `FOR NO KEY UPDATE`, que esperan por una unidad que otra transacción tiene, rechazan `FOR SHARE NOWAIT`, dejan pasar `FOR KEY SHARE` y siguen bloqueadas hasta que termina la transacción de la fachada; y una unidad que se leyó antes del candado y otra transacción cambió mientras tanto se vuelve a leer una vez bloqueada (`FleetContextFacadeLockTests`): es la regla de la fila de después del bloqueo.

## El orden de los candados de cada contexto

Cada escritura toma sus candados en un orden fijo, el mismo en todo el sistema, para que dos escrituras —del mismo contexto o de dos contextos— no se esperen la una a la otra para siempre. Lo que no cubre un candado lo cubre un único de la base, que se traduce al fallo esperado de la revisión que dejó pasar, casi siempre un `409` (arriba, «La base sigue siendo la última línea»).

### `organization`

Lo que lee y después cambia una empresa, una sede, una marca o una cuenta bancaria la bloquea antes con `FOR NO KEY UPDATE`. Lo que se cuelga de ellas —las altas, el cambio de sede, la reactivación, el regreso de un colaborador y el alcance de un rol nuevo— lee su empresa, su sede y su marca con `FOR SHARE`, siempre en ese orden, y así espera a un cambio de estado simultáneo (ORG-R56, D-84). Una operación que toca un colaborador y su cuenta bloquea primero `employees` y después `users`.

### `fleet`

**La regla de `fleet`** (FLT-R84). Toda escritura de una unidad —la habilitación, el cambio de estado, la edición, la anulación, ceder y cerrar una cesión— bloquea primero la unidad (`FOR NO KEY UPDATE`) y después, si hace falta, su fila hija, siempre en ese orden, y lee lo que decide después del candado. Varias unidades se bloquean por `id` ascendente, en una sentencia.

| Operación | Candados, en orden | Lo que lee después del candado |
|---|---|---|
| El cambio de estado (F3) | La unidad | El estado actual, la anulación, el traspaso; solo por «Devolución al proveedor», si una nota vigente la devolvió, que responde compras por `UnitPurchases` y no se cruza con la nota, porque registrarla y anularla también bloquean la unidad (D-265); y, solo si el destino tiene puerta o exige los equipos de seguridad, los hechos de la habilitación, leídos una sola vez para las dos revisiones: lo que falta y los equipos abiertos de la unidad |
| La edición de la unidad (F2) | La unidad | Si tiene una línea de factura vigente (el año queda fijo) y si ingresó (no queda sin ubicación) |
| El ingreso en lote | Todas las unidades del lote, en una sentencia, por `id` ascendente | Las unidades, sus ubicaciones actuales y la ubicación de destino |
| El ingreso de una unidad y los hitos | La unidad | Su `updatedAt`, su estado, su placa y el inicio más antiguo de sus equipos. Si la escritura vacía el ingreso o las placas físicas en Disponible, además el camino «Pierde un requisito» y En habilitación, antes de escribir; y escribe el paso de la unidad —ella y su fila del historial— en la misma transacción, sin otro candado (D-260) |
| La primera placa | La unidad | Su título y su TIVE vigente; la unicidad en el país, con una elevación que solo responde sí o no |
| Un documento | La unidad; después el documento vigente de ese tipo | El tipo y si el modelo es a gas |
| Instalar un equipo | La unidad | Su estado, primero, por si congela el equipamiento (D-260); después su ingreso, sus equipos abiertos del tipo, el tipo de equipo y la moneda del costo, esta en modo compartido (`CurrencyUsage`) |
| Reemplazar, retirar, registrar el costo o completar el serial de un equipo | La unidad del equipo; después el equipo | El período del equipo. Al reemplazar, el estado de la unidad, que lo detiene solo si congela el equipamiento. Al retirar, el estado con sus tres marcas y, solo si exige la habilitación o los equipos de seguridad, lo que el modelo exige y lo que es de seguridad; en Disponible, además el camino «Pierde un requisito» y En habilitación, antes de escribir, y escribe el paso de la unidad en la misma transacción, sin otro candado (D-260). Registrar el costo y completar el serial no leen el estado |
| Una llave | La unidad; en la edición, después la llave | — |
| Anular una unidad | La unidad | Su valor, sus contratos y su cesión abierta: la cesión se lee bajo el mismo candado que toman ceder y cerrar |
| Crear o reactivar un fondo | El fondo; la empresa en modo compartido, después del fondo | Si la empresa está activa (D-84, D-94) |
| Ceder una unidad a un fondo | La unidad para actualizar; el fondo en modo compartido (`FOR SHARE`) | Las cesiones de la unidad |
| Cerrar una cesión | La unidad | La cesión, que vuelve a leer |
| La carga del SAT (F13) | El candado de la carga, consultivo, por tipo y empresa, sin esperar; las unidades de la empresa con las placas del archivo, por `id` | El tipo de vehículo de sus modelos, si tienen la TIVE vigente y su última declaración, en lote |
| La carga de garantía (F14) | El fondo en modo compartido; el candado de la carga; en una sentencia, por `id`, las unidades de la empresa con las placas del archivo y las que tienen una cesión abierta al fondo | Las cesiones de esas unidades, de cualquier fondo |
| El alta y el vínculo por lote de una factura (fachada, FLT-R52) | Las unidades existentes por `id` ascendente | Las unidades bloqueadas, leídas en una sola sentencia, reutilizando la copia que la transacción ya leyó bajo el candado |
| La nota de crédito que devuelve o se anula (fachada, FLT-R95) | Las unidades de sus líneas por `id` ascendente, con el candado de toda escritura de la unidad | La unidad que se leyó antes del candado y otra transacción cambió se vuelve a leer |
| El regreso automático (F4) | Una sola instancia del proceso, con un bloqueo consultivo de la sesión que no espera (`SingleInstanceRun`, `fleet.status-tolerance-breach`); después, cada unidad | La unidad, vuelta a leer: si alguien la movió, si está anulada, el camino, el destino y, si aplica, la habilitación |
| Marcar un tipo de equipo en la configuración de lo que se exige (D-260) | La fila del tipo en `vehicle_equipment_requirements`, por su `equipment_type_id`, con `FOR NO KEY UPDATE`; es su único candado y no se ordena con ningún otro | La fila, vuelta a leer después del candado, y su `updatedAt`. Un tipo sin fila no tiene qué bloquear: dos primeras marcas a la vez las detiene el `UNIQUE` del tipo, y la segunda responde `422 VEHICLEEQUIPMENTREQUIREMENT_OUTDATED` sin escribir nada |

**Lo que resulta de ese orden**

- Si una persona mueve la unidad a Disponible mientras otra retira su GPS, una espera a la otra: si el retiro confirma primero, el cambio recibe `VEHICLE_ENABLEMENT_INCOMPLETE`; si el cambio confirma primero, el retiro confirma después y lleva la unidad a En habilitación por «Pierde un requisito», en su misma transacción (D-260). Ningún orden deja una unidad Disponible sin GPS (`EnablementConcurrencyTests`).
- Las escrituras de la habilitación que se suman o se cierran —subir un documento, instalar, reemplazar y retirar un equipo, registrar la placa y el ingreso en lote— no llevan `updatedAt`: las serializa el candado de la unidad, y `_ALREADY_REMOVED`, `_ALREADY_ARRIVED` y `_PLATE_ALREADY_REGISTERED` dicen que otro llegó antes. Las ediciones —el ingreso de una unidad, los hitos y una llave— llevan además el `updatedAt` leído (`VEHICLE_OUTDATED`, `VEHICLEKEY_OUTDATED`).
- Una cesión a mano de las unidades de una carga de garantía espera a la carga, porque bloquea la unidad primero, y ve lo que la carga escribió (FLT-R117).
- Toda escritura de cesiones lee su fondo en modo compartido, también la carga de garantía: así la corrección del código del fondo, que lo bloquea para actualizar, sabe si alguna cesión lo usa.
- El proceso del regreso automático, si pierde un interbloqueo, repite una vez la corrida entera (`DeadlockRetry`, D-128); si otra instancia tiene su candado, salta la corrida, y la siguiente recoge lo que quede.
- Dos altas con el mismo VIN, o dos placas iguales en el país, que pasan a la vez: la base detiene la segunda, y el servicio responde el mismo `409` que su propia revisión (FLT-R01, FLT-R02).

### `equipment`

Lo que lee y después cambia un equipo del inventario, un tipo o un modelo de equipamiento lo bloquea antes con `FOR NO KEY UPDATE`, y es su único candado: editar un equipo o cambiar su estado bloquea el equipo (`fleet_equipments`); editar un tipo o cambiar su estado, el tipo (`equipment_types`, VCT-R20), y editar un modelo o cambiar su estado, el modelo (`equipment_models`, VCT-R34). Registrar un equipo, un tipo, una marca o un modelo no bloquea nada: el único de la base detiene el segundo de dos iguales, y el servicio responde el mismo `409` que su propia revisión (EQP-R05, en [contextos/equipment.md](contextos/equipment.md)). Ninguno de esos candados cambió al pasar el inventario de `fleet` y el catálogo de `vehiclecatalog` a `equipment` (fase 2 del plan de migración, D-278).

- **La primera instalación de un tipo** la avisa `fleet` por la fachada de `equipment`, con la unidad ya bloqueada: el orden es la unidad y después la fila del tipo, que la escritura de `has_installations` toma solo la primera vez. Una edición del tipo bloquea solo el tipo, nunca una unidad, así que los dos órdenes no se cruzan.
- **Con el módulo de monitoreo** la instalación tomará la unidad, después la instalación y después los equipos del inventario, por `id` cuando un reemplazo toma dos; `equipment` nunca bloquea una unidad ni una instalación (plan, fase 2 §2.6).

### `procurement`

Las notas de crédito extienden la secuencia de las facturas, y los pagos usan los mismos números (PRC-R14, PRC-R21, PRC-R24, PRC-R26, PRC-R60, PRC-R88 y PRC-R136, en [contextos/procurement.md](contextos/procurement.md)). Los proveedores, sus contactos, las plantillas y la emisión de una orden siguen lo dicho arriba; además, el número de una orden se asigna bajo el candado de su plantilla, y la anulación y el cumplimiento de una orden bloquean solo la orden y leen sus facturas y sus pagos sin candado.

#### Facturas

| # | Qué | Modo | Quién más lo toma |
|---|---|---|---|
| 1 | La factura, en su `PUT` y en su anulación | `FOR UPDATE` | Las escrituras de facturas; el registro y la anulación de una nota; el registro de un pago |
| 2 | La empresa | `FOR SHARE`, por `organization` | Su desactivación |
| 3 | El proveedor | `FOR SHARE` | Su cambio de estado y sus contactos |
| 4 | La moneda | `FOR SHARE` | La corrección de sus decimales |
| 5 | La orden vinculada | `FOR NO KEY UPDATE` | El envío, el cumplimiento y la anulación de la orden; un pago a la orden |
| 6 | Las unidades existentes de la empresa, por `id` ascendente | `FOR NO KEY UPDATE`: `procurement` las bloquea al cruzar los VIN, de una vez, y `fleet` otra vez al vincularlas, en la misma transacción | Toda escritura de la unidad: edición, caminos, VIN, anulación |
| 7 | Las marcas comerciales de las unidades nuevas, por `id` ascendente | `FOR SHARE`, por `organization`, desde `fleet`, justo antes de escribir | Su desactivación y su reactivación, que bloquean la marca en `FOR NO KEY UPDATE` |

Ninguna transacción los toma en otro orden: la anulación de una orden bloquea solo la orden y lee las facturas sin candado; la edición de una unidad bloquea la unidad y pregunta a compras sin candado; desactivar una marca bloquea solo la marca, y reactivarla lee después la empresa en modo compartido, que no choca con el de la factura. Lo que no cubre un candado lo cubre un único: dos facturas con el mismo VIN nuevo chocan en `vehicles_vin_uq`, y dos que vinculan la misma unidad, en `purchase_invoice_items_vehicle_current_uq`; la carrera responde el `409` de su llave.

#### Notas de crédito

La nota va primero, y las facturas que descuenta van detrás de la empresa. Ninguna escritura de facturas toma a la vez la factura y la empresa, así que ninguna cambia su orden.

| # | Qué | Modo | Quién lo toma |
|---|---|---|---|
| 1 | La nota, en su anulación y en el cambio de su archivo | `FOR UPDATE` | Solo esas dos escrituras |
| 2 | La empresa | `FOR SHARE`, por `organization` | El registro de la nota y el de la factura |
| 3 | Las facturas que la nota descuenta, por `id` ascendente, en una sentencia | `FOR UPDATE` | El registro y la anulación de la nota; el `PUT` y la anulación de una factura (su candado 1) |
| 4 | El proveedor | `FOR SHARE` al registrar una factura o una orden; `FOR NO KEY UPDATE`, el candado del saldo (D-195), al anular una nota | Su cambio de estado y sus contactos; los pagos con saldo y su anulación |
| 5 | La moneda | `FOR SHARE` | El registro de la factura. La nota no la toma: sus decimales los fija su factura |
| 6 | La orden vinculada | `FOR NO KEY UPDATE` | Las escrituras de la factura y de la orden |
| 7 | Las unidades, por `id` ascendente | `FOR NO KEY UPDATE` | La anulación de las unidades devueltas y el candado de las unidades de una nota que se anula; el registro de una factura; toda escritura de la unidad |
| 8 | Las marcas de las unidades nuevas de una factura | `FOR SHARE`, por `organization`, desde `fleet` | Solo el registro de la factura, después de la orden y de sus unidades existentes |

**Por qué no hay ciclos.** El registro de una nota toma 2, 3 y 7; su anulación, 1, 3, 4 y 7; el cambio de su archivo, solo 1; el registro de una factura, 2, 4, 5, 6, 7 y 8; su `PUT`, 3 y 6; su anulación, 3. Nadie que tenga una unidad espera una factura ni el proveedor, y nadie que tenga el proveedor en modo exclusivo espera algo que tenga el registro de una factura. La anulación de una unidad y la edición de su año bloquean solo la unidad y preguntan a compras sin candado, así que ven las líneas que confirmó la transacción que tenía la unidad antes. Un pago toma las facturas antes que el proveedor. La nota se anula y se guarda antes de devolver sus líneas, porque el disparador de la línea cuenta las notas vigentes y tiene que verla anulada.

**Las carreras que resuelve.** Dos notas sobre la misma línea se ordenan en el candado de su factura: la segunda ve lo que restó la primera y, si se pasa, responde `422 CREDITNOTE_EXCEEDS_VALUE`. El doble envío del mismo alta espera al primero en el candado de las facturas y responde `409 CREDITNOTE_CONFLICT` (único `credit_notes_number_uq`). La anulación de una nota y el registro de una factura que vincula la misma unidad se ordenan en el candado de la unidad: gana la primera, y la otra responde `422 CREDITNOTE_LINE_RELINKED` (el único `purchase_invoice_items_vehicle_current_uq`, traducido) o `409 PURCHASEINVOICE_UNIT_CONFLICT`.

#### Pagos

Siguen la secuencia común —la empresa antes que las facturas, las facturas antes que el proveedor—, con los mismos números, y suman el `requestId` y la cuenta de origen.

| # | Qué | Modo | Quién lo toma |
|---|---|---|---|
| 0 | El `requestId` | `pg_advisory_xact_lock`, antes de leer nada | El registro de un pago |
| 1 | El pago | `FOR NO KEY UPDATE` | Su anulación y el cambio de su constancia |
| 2 | La empresa | `FOR SHARE`, por `organization` | El registro. Un pago no toma marcas |
| 3 | Las facturas que nombra, por `id` ascendente, después de leer sus `id` sin candado | `FOR UPDATE` | El registro |
| 4 | El proveedor | `FOR SHARE` en un pago de dinero; `FOR NO KEY UPDATE`, el candado del saldo (D-195), en un pago con saldo, en la anulación de un pago y en la de una nota | Las anteriores |
| 5 | La moneda del pago | `FOR SHARE` | El registro |
| 6 | Las órdenes que nombra, por `id` ascendente | `FOR NO KEY UPDATE` | El registro de un pago con órdenes; el vínculo de una factura, que lleva el paso de la orden a la factura; la anulación y el cumplimiento de la orden |
| — | La cuenta de origen, después de las órdenes | `FOR SHARE`, por `organization` | El registro de una transferencia o un cheque. La llave foránea sola no espera a un `FOR NO KEY UPDATE`: sin el candado, el registro podría cruzarse con la desactivación de la cuenta o con la corrección de su número (D-214) |
| — | Las unidades | — | Un pago no las toma: su unidad la nombra una línea vigente de una factura ya bloqueada |

Lo que no existe o no se ve queda sin candado, y lo responde su revisión. La anulación de un pago bloquea el pago, comprueba el permiso con su alcance, el `updatedAt` y que no esté anulado; bloquea al proveedor y vuelve a leer las aplicaciones del pago, para ver un paso de la orden a la factura confirmado mientras tanto; y calcula sin él el saldo y los usos. El cambio de la constancia bloquea el pago y revisa, en este orden, el permiso con su alcance, el `updatedAt`, el estado y el método. El reparto, la inicial y los usos del saldo se escriben con las facturas y el proveedor ya bloqueados: no suman candados.

**Por qué el pago se bloquea con `FOR NO KEY UPDATE` y no con `FOR UPDATE`, como la nota:** la parte de una aplicación que pasa de la orden a la factura inserta una fila que apunta al pago, y su llave foránea toma `FOR KEY SHARE` sobre él. Con `FOR UPDATE`, un vínculo que ya tiene al proveedor en modo compartido esperaría al pago, y la anulación del pago, que espera al proveedor en modo exclusivo, lo esperaría a él.

**Lo que cubre:** dos pagos a la misma factura se esperan en el candado 3, y el segundo no aplica de más; la anulación de una factura y un pago a ella, también en el 3, y el segundo ve el resultado del primero; un pago con saldo, la anulación de un pago y la de una nota, en el 4, y cada uno calcula el saldo que dejó el anterior; una orden que se anula o se cumple y un pago a ella, en el 6; una nota que anula una línea y una imputación a esa unidad, en el 3, que la nota toma antes de anular la línea; y una transferencia o un cheque y la corrección o la desactivación de su cuenta, en el candado de la cuenta: la corrección que llega durante el pago lo espera y responde `422 BANKACCOUNT_IN_USE`, y el pago que llega durante la desactivación la espera y responde `422 SUPPLIERPAYMENT_BANK_ACCOUNT_INACTIVE` sin guardar nada (`BankAccountLockTests`). Lo que no cubre un candado lo cubre un único, que se traduce a su `409`: el mismo `requestId` choca en `supplier_payments_request_uq`, y el mismo número de operación, en `supplier_payments_operation_uq`. Lo que solo se muestra —lo disponible de una nota, leído en varias sentencias— puede quedar un instante desfasado; lo que se registra va bajo el candado.

#### La carga inicial

El orquestador no es transaccional: cada factura, cada nota, cada pago y el lote corren en su propia transacción (`PROPAGATION_REQUIRES_NEW`), y un paso que pierde un interbloqueo se repite una vez. Después de revisar el alcance y antes de escribir nada, la carga toma un candado consultivo de sesión por el tipo de carga y la empresa, en una conexión propia, y lo suelta al terminar: otra carga inicial de la misma empresa responde `409 IMPORTBATCH_CONFLICT` sin esperar, porque una espera larga terminaría en el corte de Cloudflare; dos empresas distintas no se esperan.

| Paso | Candados, en orden |
|---|---|
| Cada factura | La empresa, el proveedor y la moneda en modo compartido; las unidades nuevas no existen todavía, y el único del VIN las protege |
| Cada nota | La empresa, el proveedor y la moneda en modo compartido; las facturas de la nota, por `id`, como el registro a mano |
| Cada pago | El `requestId`; la empresa; las facturas del bloque, por `id`; el proveedor y la moneda en modo compartido |
| El lote | — |

## Una operación que pierde un interbloqueo se repite una vez

Aun con el orden fijo, dos transacciones pueden bloquearse en órdenes opuestos —por ejemplo, sobre los contadores de `data_versions` que comparten dos escritores de tablas distintas—, y PostgreSQL aborta una (`40P01`). `DeadlockRetry` (`shared/infrastructure/persistence/transactions`) la repite entera una sola vez, para que no responda un `500` que un segundo intento no habría dado:

- envuelve cada método público de un `*CommandServiceImpl` de `application.internal.commandservices`, en todos los contextos, y cada `@Scheduled` de `application.internal.scheduledtasks`;
- solo la operación más externa y solo cuando no había una transacción abierta al empezar: repetir dentro de otra transacción repetiría media transacción que PostgreSQL ya abortó;
- va por fuera de la transacción, así el segundo intento abre una nueva;
- reconoce el interbloqueo por el `SQLException` con `40P01` en la cadena de causas, lo envuelva quien lo envuelva; una espera de candado vencida (`55P03`), una falla de serialización (`40001`) o cualquier otra falla pasan tal cual, y un segundo interbloqueo también.

Un servicio que no abre una transacción alrededor de sus pasos, como el inicio de sesión, cuenta con que la operación entera se puede repetir. Lo prueban `DeadlockRetryTests` y `DeadlockRetryRecognitionTests`.

## Un archivo y su fila: primero la fila

Los archivos que no vencen —el de una factura, el de una nota de crédito, la constancia de un pago, el de un documento de una unidad, los de una plantilla de orden, el PDF enviado de una orden y los de las cargas— van a `kept_files` por `KeptFileStore` (`documents`, D-189, diccionario §2.7). Con almacenamiento en la base, los bytes viajan en la fila, y la transacción los guarda o los revierte juntos. En DigitalOcean Spaces son dos escrituras, la fila y el objeto, y el objeto no se revierte con la transacción. La regla:

- **Primero la fila, después el objeto.** `SpacesKeptFileStore.put` guarda la fila con `INSERT … ON CONFLICT … DO UPDATE … RETURNING (xmax = 0)`, que dice si la insertó, y solo después escribe el objeto. Si la fila falla, no hay objeto. Si el objeto falla, el `put` falla y la fila se revierte con la transacción.
- **Al revertir, se descarta solo lo que la transacción escribió primero.** Quien llama —la factura, la nota, el pago, el documento de una unidad y las cargas— registra un `TransactionSynchronization` que, si la transacción se revierte, llama a `discardRolledBack` para cada llave cuyo `put` respondió que la insertó. Una llave que ya existía —los mismos bytes enviados otra vez, o un archivo anterior que vuelve— nunca se descarta, porque un registro confirmado la apunta. La plantilla de orden y el PDF enviado de una orden no lo registran: un objeto suyo que escribió una transacción revertida queda en el bucket sin fila.
- **Queda un caso documentado:** dos escrituras de la misma llave a la vez, cuando una se revierte y la otra confirma. Las dos esperan el candado de su registro, pero el descarte de la que se revirtió corre después de soltarlo, y puede borrar de Spaces el objeto que la otra volvió a escribir. En la base no pasa.

Lo prueban `SpacesKeptFileStoreTests` y `KeptFileStoreUpsertTests`, en los dos almacenes.

## Dos ediciones de lo mismo: el `updatedAt` que se leyó

El bloqueo de una fila ordena dos operaciones que corren al mismo tiempo; no protege una edición que alguien abrió hace diez minutos, mientras otra persona guardaba la suya. Para eso, lo que se edita en un formulario llega con su `updatedAt` y vuelve con él al guardar:

- **Se compara bajo el candado.** El servicio lee la fila con `ForUpdate`, hace las revisiones de acceso y, antes de las reglas de su estado, compara el `updatedAt` que llegó con el de la fila, en milisegundos, la precisión de un momento en el navegador. Si no coinciden, responde `422 <RECURSO>_OUTDATED` y no guarda nada.
- **Lo hacen** el proveedor (`SUPPLIER_OUTDATED`) y cada uno de sus contactos (`SUPPLIER_CONTACT_OUTDATED`, D-137), la plantilla (`PURCHASEORDERTEMPLATE_OUTDATED`), el borrador de una orden de compra (`PURCHASEORDER_OUTDATED`, D-138), la factura (`PURCHASEINVOICE_OUTDATED`), la nota de crédito (`CREDITNOTE_OUTDATED`), el pago (`SUPPLIERPAYMENT_OUTDATED`), la cuenta bancaria (`BANKACCOUNT_OUTDATED`), el tipo de cambio (`EXCHANGERATE_OUTDATED`), la unidad (`VEHICLE_OUTDATED`), sus llaves (`VEHICLEKEY_OUTDATED`) y lo que la habilitación exige de un tipo de equipo (`VEHICLEEQUIPMENTREQUIREMENT_OUTDATED`, D-260), que llega con un `updatedAt` nulo mientras el tipo no tiene fila.
- **La revisión la llena el sistema** (D-42): `updated_at` y, donde existe, `updated_by` los pone la auditoría de la clase base.
- **Lo que no toca la cabecera también mueve la revisión.** Un guardado que cambia solo las líneas de una orden deja la entidad sin cambios, y la auditoría no escribiría nada: el repositorio mueve la revisión con una sentencia propia (`PurchaseOrderJpaRepository.recordRevision`), como la plantilla cuando cambia uno de sus archivos. Enviar, cumplir y anular la mueven también.

`PurchaseOrdersIntegrationTests` y `SupplierContactsIntegrationTests` lo prueban por la API; en PostgreSQL, con dos transacciones, `SupplierLockTests` y `PurchaseOrderTemplateLockTests`.

## Las tablas de empresa y Row Level Security

Toda tabla con `company_id`, y `companies` por su propio `id`, tiene Row Level Security activada y forzada: la base deja ver y escribir solo las filas de las empresas que alcanza quien actúa. Cada tabla de empresa lleva las políticas de V29 —una para leer, una para insertar y, salvo en las que solo se insertan, como la bitácora, el historial de estados o las líneas de una nota de crédito, una para actualizar, todas donde alcanza la transacción y ninguna para borrar—. `addresses`, sin `company_id`, la tiene por el país de la dirección (V78, diccionario §2.5, D-134): se ve donde se alcanza una empresa de ese país, o con `SYSTEM`. `kept_files`, los bytes que no vencen, lleva la empresa de su registro, con la misma Row Level Security (V84). `RowLevelSecurityMigrationTests` exige esa forma en cada tabla con `company_id`. Para la persistencia eso significa:

- **El alcance se fija al comenzar cada transacción, desde un solo punto.** `AccessScopeJpaTransactionManager`, el administrador de transacciones de la aplicación, llama a `set_config(…, true)` sobre la conexión de la transacción con lo que `DatabaseAccessScope` tiene para el hilo: `app.access_scope` (`SYSTEM`, `SCOPED` o nada), `app.company_ids` y `app.user_id`. Nunca al abrir la conexión. Si no puede fijarlo, revierte y suelta la transacción, que nunca corre sin su alcance. Pasan por él todas las transacciones: las de los servicios y los repositorios, de solo lectura o no, las que abre `REQUIRES_NEW` y las de Spring Session. Un `JdbcTemplate` dentro de la misma transacción comparte la conexión y sus valores.
- **Sin alcance no se ve nada.** Una lectura fuera de toda transacción, o de un hilo que nadie preparó, devuelve cero filas de empresa. Por eso los servicios de consultas leen en una transacción de solo lectura.
- **`SYSTEM` siempre es explícito.** El arranque, los procesos programados y las migraciones de datos lo fijan para su propio trabajo. Una comprobación que necesita ver más allá de las empresas de quien llama lo pide con `SystemAccessScope`, que devuelve solo datos sueltos y vuelve al alcance anterior; antes de cambiar de alcance escribe lo pendiente y no deja en memoria nada leído con el otro.
- **Una pregunta sobre todas las empresas va en `SYSTEM`.** Si un país tiene direcciones lo cuentan las de todas las empresas, que quien corrige el país puede no ver: `CountryCommandServiceImpl` bloquea el país, entra en `SYSTEM`, pregunta, sale y vuelve a leer el país, porque cambiar de alcance vacía lo leído; `CountryViewFactory` pregunta igual para «tiene direcciones». `CurrencyCommandServiceImpl` hace lo mismo para saber si una moneda ya tiene montos (§4.1): bloquea la moneda, pregunta en `SYSTEM` a los `CurrencyUsage` de los contextos con montos y vuelve a leerla ([mapa de contextos](mapa-de-contextos-millennium-mobility.md), §6.6).
- **La aplicación no es dueña de las tablas.** Se conecta como `erp_mm_app`, que no puede saltarse la regla ni borrar filas de negocio; Flyway migra como `erp_mm_owner`. Las pruebas se conectan igual, así que un camino que olvide fijar el alcance falla en las pruebas y no en producción. Fuera de `dev`, la aplicación no arranca si Flyway no tiene su propia cuenta, distinta de la de la aplicación, ni si PostgreSQL dice que la cuenta de la aplicación es superusuario o tiene `BYPASSRLS`, que se saltan Row Level Security aunque esté forzada: `DatabaseAccountsConfiguration` lo pregunta antes de que el servidor atienda una petición (`DatabaseAccountsConfigurationTests`, `ApplicationAccountQueryTests`). En `dev`, sin `FLYWAY_USER`, sigue la excepción de una sola cuenta (README del backend, «Cuentas de la base»).
- **Un script local que borra filas forzadas** levanta `FORCE` solo en su transacción (`NO FORCE ROW LEVEL SECURITY`, borrar, `FORCE`), también en `addresses` y en `role_permissions`: sin política de `DELETE`, un borrado no alcanza ninguna fila, ni siquiera para la dueña.
- **Una tabla de solo inserción tiene tres barreras, una detrás de otra, y cada una se prueba por separado.** La cuenta de la aplicación no tiene `UPDATE` sobre ella y recibe `42501`. La dueña tiene todos los privilegios, pero con Row Level Security forzada y sin política de `UPDATE` su `UPDATE` cambia 0 filas sin avisar: no hay error que esperar, sino un conteo de cero. El disparador que rechaza la edición («never edited») solo lo alcanza quien se salta las políticas: un superusuario, o la dueña de una tabla sin `FORCE`. En `payment_allocations` (PRC-R116) lo prueban `SupplierPaymentSchemaTests` y `PaymentAllocationsOwnerTriggerTests`, que se conecta como la dueña y levanta `FORCE` solo en su transacción revertida para llegar al disparador. En `credit_note_items` (V137, D-259), `CreditNoteTriggerTests` espera `42501` con la cuenta de la aplicación, y `CreditNoteItemsOwnerTriggerTests` prueba las otras dos barreras con la dueña. Insertar una línea dispara los disparadores de restricción diferidos de la nota, y PostgreSQL no altera una tabla con eventos de disparadores pendientes: la prueba corre `SET CONSTRAINTS ALL IMMEDIATE` antes de levantar `FORCE`.

**En las pruebas, el alcance también se fija al comenzar una transacción.** `SystemAccessScopeTestExecutionListener` entra en `SYSTEM` antes de que empiece la transacción de cada prueba, así que las filas que una prueba escribe directo en la base, y las que lee para comprobar, se ven en todas las empresas, como en una migración; lo que pide a la API corre con el alcance de quien llama. Una prueba sin transacción propia —una carrera, o una que confirma sus filas para que otra transacción las vea— no tiene alcance en sus sentencias directas: cada una va en una transacción, con `InvoiceRaces.committed` o un `TransactionTemplate` equivalente. Fuera de transacción, la cuenta de la aplicación no ve ninguna fila de empresa, así que un conteo de cero no prueba nada: «la carga no escribió nada» pasaría aunque hubiera escrito. Las filas que una carrera necesita confirmadas se crean una vez y se reutilizan entre corridas, como la cuenta de la carga inicial con su rol propio en Perú y la ubicación de la empresa de Colombia, para que ninguna corrida gaste un país de prueba (README del backend, «Pruebas»).

El filtro de la aplicación sigue siendo obligatorio: cada consulta filtra por el alcance de quien llama. La base es la defensa de abajo, para que una consulta que olvidó su filtro devuelva menos filas, nunca las de otra empresa (reglas 5.1 a 5.3 y 5.7 de la skill `backend-module`). Las instrucciones para correr la aplicación y las pruebas con las dos cuentas están en el README del backend.

## Una columna con un dato personal: cifrada en la aplicación

El documento y el teléfono de un colaborador (D-261) se guardan cifrados, con una clave que la base nunca ve. Es el modelo para el próximo dato personal que haga falta proteger:

- **El dominio no se entera.** El agregado guarda el valor como texto. Un `AttributeConverter` de `infrastructure`, que Spring crea con la clave (`SpringBeanContainer`), cifra al escribir y descifra al leer; la columna es `BYTEA` y se llama `<campo>_ciphertext`.
- **Cifrado autenticado y propio de cada valor.** AES-256-GCM con un nonce al azar por valor y el propósito de la columna como dato asociado (`employees.document_number`): el mismo valor nunca repite sus bytes, y los de una columna no se descifran en otra. Los bytes empiezan con una versión, para poder cambiar de clave o de algoritmo.
- **Lo que la base compara va en una huella.** Un valor cifrado no se compara ni se ordena en SQL. Si hace falta unicidad o búsqueda, se guarda además un HMAC-SHA256 de su forma normalizada (`<campo>_digest`), con otra clave derivada del mismo secreto, y la restricción `UNIQUE` y la búsqueda van sobre la huella: solo encuentra el valor completo. El adaptador del repositorio la escribe en cada guardado y la calcula igual al buscar. No se ordena por la columna.
- **Nunca `pgcrypto`.** Llevaría la clave al servidor de la base en cada sentencia, que es justo de quien se protege el dato.
- **Pasar una columna existente a cifrada** son tres migraciones: una que agrega las columnas nuevas, una de Java que cifra las filas existentes con el bean de la clave y vacía la columna en claro, bajo `LOCK TABLE … SHARE ROW EXCLUSIVE`, y una que exige las nuevas y prohíbe volver a escribir en claro con un `CHECK`. La columna vieja se borra en una entrega posterior (condición de instancia única del plan de migración).
- **La clave no se rota reemplazándola**: sin la clave con que se cifró, el dato es ilegible. Rotarla pide volver a cifrar cada fila y recalcular su huella.
- **Las pruebas** escriben y leen esas columnas con la misma clave de desarrollo (`EmployeePersonalDataForTests`).

## Por qué también los catálogos

Un catálogo no tiene reglas que proteger, y el patrón completo duplica campos: un camino corto —el modelo como entidad de JPA y el repositorio que extiende `JpaRepository`— ahorraría una entidad y un ensamblador por tabla. No se usa, porque el dominio quedaría atado a la persistencia: el modelo llevaría anotaciones de JPA y la interfaz del repositorio dependería de Spring Data, con métodos como `delete` que el dominio no debe tener. Separar las capas no causa el riesgo de perder `created_at`; lo causa reconstruir la fila al guardar, y eso se evita cargándola (arriba). Con esa regla, el costo de separar es una entidad y un ensamblador por tabla, a cambio de un dominio que se prueba sin nada alrededor y de una regla sin excepciones.

## Ejemplo de por qué importa: el vehículo

El negocio dice que **el odómetro no retrocede**: si hoy marca 15 000 km, nadie puede registrar 12 000 como una lectura más. Si le cambiaron el tablero, se registra como sustitución de odómetro, que es un hecho distinto (diccionario §8.5).

**Con una clase de JPA, la regla se puede saltar:** JPA necesita un constructor vacío y suele terminar con setters.

```java
@Entity
public class Vehicle {
    public void setMileage(Integer mileage) { this.mileage = mileage; }   // cualquiera puede llamarlo
}

vehicle.setMileage(12000);   // nadie lo impide: la regla quedó rota
```

**Con la regla en el dominio, siempre se cumple:** vive en `MileageSeriesPolicy` (`fleet/domain/services`), sin nada alrededor, y una prueba la comprueba sin base de datos.

```java
public final class MileageSeriesPolicy {        // sin @Entity, sin Spring

    public enum Verdict { ACCEPTED, LOWER_THAN_PREVIOUS, HIGHER_THAN_NEXT }

    public static Verdict check(int mileage, Integer previous, Integer next, boolean isOdometerSubstitution) {
        if (previous != null && mileage < previous && !isOdometerSubstitution) return Verdict.LOWER_THAN_PREVIOUS;
        if (next != null && mileage > next) return Verdict.HIGHER_THAN_NEXT;   // una lectura de taller cargada días después
        return Verdict.ACCEPTED;
    }
}
```

El servicio lee la lectura anterior y la siguiente, pregunta a la política y, si la lectura no cabe, devuelve `Result.failure` (`mileageBelowPrevious`, `mileageAboveNext`): un fallo esperado del negocio no se lanza como excepción (regla 3.1). Las exigencias de JPA se van a `VehicleMileageReadingPersistenceEntity`, que solo sabe de columnas, y el código del negocio nunca la toca.

## Objetos de valor y rangos

Un objeto de valor que solo valida su formato, como `Vin`, `LicensePlate` o un código ISO, vive en el dominio y se convierte a columna en la entidad de persistencia, con un conversor (`infrastructure/persistence/jpa/converters`) o en el ensamblador. Lo mismo vale para un monto con su moneda (dos columnas) o un período `[desde, hasta)`, que en la base es un `daterange` —una cesión a un fondo, una instalación de un equipo— o un `tstzrange` —un bloqueo de una cuenta—.

## Lo que vale siempre

- **Las columnas de registro las llena el sistema (D-42).**
  - Las fechas las pone la auditoría de Spring Data (`@CreatedDate`, `@LastModifiedDate`). Nadie las calcula con `Instant.now()`.
  - `created_by` y `updated_by` los toma de la cuenta de la sesión el auditor que aporta `iam` (`CurrentUserAuditorAware`).
  - Un proceso sin sesión asigna `SYSTEM` de forma explícita.
- **La clase base se elige por las columnas que pide el cuadro del diccionario, nunca por el nombre.** Todas viven en `shared/infrastructure/persistence/jpa/entities` y forman una sola cadena, con `id` y `created_at` en la raíz:

| Columnas de registro del cuadro | Clase base | Ejemplos |
|---|---|---|
| `created_at` | `CreatedAbstractPersistenceEntity` | `vehicle_equipments`, `password_reset_tokens`, `account_activation_codes`, `notifications` |
| `created_at`, `updated_at` | `AuditableAbstractPersistenceEntity` | Las tablas del bloque 1 salvo `exchange_rates`; `suppliers`, `supplier_contacts`, `branches`, los catálogos de flota |
| `created_at`, `created_by` | `CreatedByAbstractPersistenceEntity` | `user_access_suspensions`, `employee_account_link_events`, `employee_tenure_periods`, `vehicle_fund_assignments` |
| `created_at`, `updated_at`, `created_by` | `AuthoredAbstractPersistenceEntity` | `users`, `employees`, `generated_documents` |
| `created_at`, `updated_at`, `created_by`, `updated_by` | `RevisedAbstractPersistenceEntity` | `companies`, `company_bank_accounts`, `exchange_rates`, `vehicles`, `fleet_equipments`, `purchase_orders`, `purchase_order_templates`, `purchase_invoices`, `credit_notes`, `supplier_payments` |
| Ninguna | Ninguna: la entidad declara su propio `id` | `roles`, `user_roles`, `permissions`, `change_logs`, las líneas de las facturas, las notas y los pagos |

  - `CreatedAbstractPersistenceEntity` es la raíz: `id`, `created_at` y el oyente de auditoría. `Auditable` la extiende y agrega `updated_at`; `Authored` agrega `created_by` y `Revised`, `updated_by`. `CreatedBy` extiende la raíz y agrega `created_by`, con las mismas anotaciones que `Authored`: es la única duplicación aceptada.
  - Extender una clase que trae una columna que el cuadro no pide es una violación: con `ddl-auto: validate`, la aplicación no arranca.
- **Las filas no se borran**: se desactivan, se cierran o se anulan. Por eso las interfaces de repositorio de negocio no tienen `delete`. La cuenta de la aplicación tiene `DELETE` solo sobre las sesiones y sobre las tablas de control que se purgan —los frenos del inicio de sesión, los enlaces para restablecer la contraseña y los códigos de activación—, y `hand-over-database.sql` comprueba que no tenga más.
- **Una búsqueda pliega los dos lados en la base** (D-142, V86). El patrón de `q` lo arma `PageRequests.containsPattern`, en minúsculas y con los comodines escapados, y la consulta compara `fold_search_text(x) like fold_search_text(cast(:pattern as String)) escape '\'` en una `@Query`, o usa `PageRequests.containsFolded` en una consulta por criterios, que liga el patrón como parámetro; `SearchTextFunctionContributor` declara la función a Hibernate. La función quita las tildes de las vocales, la ñ y la ç y pasa a minúsculas, en la base y no en Java, para que los dos lados plieguen igual sea cual sea el `ctype` de la base. Es `IMMUTABLE` y `PARALLEL SAFE`, sin la extensión `unaccent`, y sin índice: una búsqueda que contiene un texto no usaría uno de árbol, y si una lista crece se agrega `pg_trgm` sobre `fold_search_text(columna)` sin cambiar la aplicación. Un filtro exacto, como el puesto de un colaborador, no pasa por ella. Una consulta nueva con `q` usa lo mismo, nunca `lower()` sola. Lo prueba `SearchTextFoldingTests`.
- **Una lista se ordena solo por lo que declara.** `PageRequests.toPageable` traduce los campos del recurso a las propiedades de la entidad que el repositorio declara ordenables; cualquier otro campo recibe el orden propio del repositorio, y las filas con el mismo valor conservan un orden estable por `id`, así una fila nunca salta entre páginas.
- **Las funciones de la base son de la cuenta que migra.** Las migraciones crean sus funciones como Flyway —`fold_search_text(text)` (V86), `bump_data_version()` (V53), las de Row Level Security (V29, V78) y los disparadores—, y la de la búsqueda le da `EXECUTE` a la cuenta de la aplicación por su nombre, para que retirarle el permiso a `PUBLIC` algún día no se lo quite a las búsquedas. En una base ya migrada, `hand-over-database.sql` las pasa a `erp_mm_owner` y le da `EXECUTE` a `erp_mm_app`; `HandOverDatabaseScriptTests` falla si una función nueva de las migraciones no tiene su línea en el script.
- **Solo el `public_id` sale por la API**; el `id` interno nunca aparece en rutas ni respuestas.
- **El dominio no importa nada de otro contexto**: lo que necesite de él lo pide por su fachada ACL.
