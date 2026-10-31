# Patrones de persistencia — ERP Millennium Mobility

**Actualizado:** 30/10/2026 (`develop` del 30/10/2026, backend `a54ee29b`, sin publicar todavía, y diccionario v1.33: la búsqueda que pliega los dos lados con `fold_search_text`, la función de V86, D-142, con su dueño, su permiso y el script de traspaso; las decisiones del 29/10/2026 citadas por su número, D-137 y D-138, y el cuadro de §9.2 con la revisión de la orden) · 29/10/2026 (lo que siguió al cierre de la auditoría, verificado contra `develop` publicado, backend `5ec83065`: emitir una orden de compra lee con `FOR SHARE` lo que revisa activo, y un contacto se protege con la fila de su proveedor, con la carrera de su retiro contra la orden que lo nombra; los decimales de una moneda y su primera orden, que no se cruzan; dos ediciones de lo mismo, por su `updatedAt`, también el borrador de una orden y un contacto, con `purchase_orders` en `RevisedAbstractPersistenceEntity` por V85; las tablas de compras con las políticas de V29, V83, y `kept_files` bajo Row Level Security, V84; el arranque que rechaza una cuenta de la aplicación que se salta Row Level Security; y el duplicado sin traducir que nombra su restricción en el registro) · 26/10/2026 (cierre de la auditoría del 25/10/2026: la lectura en modo compartido de lo que se referencia, con el fondo y las referencias de `organization`; las pruebas de los candados de compras y de fondos, con el tipo de orden bloqueado al renombrarlo y al cambiar su estado; las migraciones que bloquean sus tablas y esperan un candado a lo sumo cinco segundos, V76 a V81; `addresses` bajo Row Level Security por su país, V78, y la pregunta sobre todas las empresas en `SYSTEM`; los scripts locales que borran filas forzadas) · 16/10/2026 (bloquear antes de leer lo que se va a cambiar; tablas de empresa bajo Row Level Security; las cinco clases base existen) · 06/10/2026 (clases base por forma de las columnas de registro; el adaptador carga y copia también en `iam`) · **Fuente:** skill `backend-module` (§1 y pasos de construcción; `module-template.md`), `architecture-rules.md` (reglas 2.1 y 3.1) y diccionario de datos v1.15 (D-42). Si este documento y el diccionario no coinciden, manda el diccionario.

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
    private short decimalPlaces;              // de 0 a 4
    private Instant createdAt;                // los registra el sistema

    public Currency(String isoCode, String name, String symbol, Short decimalPlaces) { /* valida */ }
    public Currency(Long id, UUID publicId, /* … */ Instant createdAt, Instant updatedAt) { /* reconstituye */ }
    public void update(String name, String symbol, Short decimalPlaces) { /* valida y cambia */ }
}

public interface CurrencyRepository {         // en domain/repositories
    Currency save(Currency currency);
    Optional<Currency> findByIsoCode(String isoCode);
    List<Currency> findAllActive();
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
        return CurrencyPersistenceAssembler.toDomainFromPersistence(jpa.saveAndFlush(row));
    }
}
```

## Cómo se guarda una edición

**El adaptador nunca arma una fila nueva para un agregado que ya existe.** Carga la fila de la base y le copia los cambios.

La razón es un error que ya pasó. La primera versión de monedas y países armaba una fila desde cero en cada guardado, con el id pero sin la fecha de creación, porque esa fecha la registra el sistema y el dominio no la asigna. La base conservaba el valor, ya que la columna no se actualiza, pero la respuesta de `PUT` y `PATCH` salía con `createdAt: null`. Cargando la fila, los valores que registró el sistema (`created_at` y, donde exista, `created_by`) siguen ahí.

La regla, en concreto:

- El adaptador decide la fila: una nueva si el agregado no tiene `id`, o la que carga de la base si lo tiene. Después le pide al ensamblador que copie y guarda esa fila.
- El ensamblador no tiene un método que construya la fila desde el agregado ni llama a `setId`. Copia solo los valores que el dominio posee; el identificador, las fechas de registro y el autor quedan como los tiene la fila.
- Los hijos se sincronizan por su `id`: al que existe se le copian sus cambios sobre su propia fila y solo un hijo nuevo recibe una fila nueva. Ninguno se quita de la colección, porque las filas no se borran.
- Una referencia a una fila que el agregado no posee, como el rol de una asignación, apunta a la fila que la base ya tiene (`findById`), nunca a una reconstruida desde el dominio.

`CurrencyAuditTests`, `CountryAuditTests` y `UserAuditTests` prueban que una edición conserva la fecha de creación (y el autor, en la cuenta). Cada agregado nuevo necesita una prueba igual. Desde el 06/10/2026 `iam` sigue la misma regla: antes armaba la fila de la cuenta y la de cada rol desde el dominio.

## Una entidad dentro del agregado

Los escalones de un país viven dentro de `Country` (D-45). El país decide si admite otro, en qué posición va y cómo se renombra, y se guardan con él:

- `CountryPersistenceEntity` mapea sus escalones con `@OneToMany` y `cascade` de `PERSIST` y `MERGE`. Nunca `REMOVE` ni `orphanRemoval`: las filas no se borran.
- El ensamblador sincroniza los hijos por su `id`. Al que existe le copia el nombre y al nuevo le crea su fila, con el país y la posición.
- Las lecturas del país cargan sus escalones en la misma consulta (`@EntityGraph`), para no hacer una consulta por país.
- Para leer o renombrar un escalón suelto, el repositorio busca el país que lo declara.

`GeoDivisionLevelAuditTests` prueba que un escalón se guarda a través de su país y conserva su fecha de creación al renombrarse.

## Leer y después cambiar: primero se bloquea

Una operación que lee un estado y decide con él, y después lo cambia, puede correr al mismo tiempo que otra que decide con el mismo estado. Las dos pasan sus revisiones y la segunda pisa a la primera. Pasó en el paso 3, y los hallazgos de la revisión eran reales: dos bloqueos de la misma cuenta que se cruzaban, un acceso que se otorgaba durante la salida del colaborador, dos salidas de la misma persona en dos empresas que no cerraban la cuenta.

La regla:

- **Se bloquea la fila antes de leer.** El repositorio del dominio ofrece la lectura que bloquea, con el sufijo `ForUpdate` (`findByIdForUpdate`, `findByPublicIdForUpdate`): primero un `SELECT … FOR NO KEY UPDATE` sobre la fila, que dura hasta que termina la transacción, y después la lectura del agregado. `NO KEY` deja pasar a quien solo inserta filas que apuntan a ella, como un período nuevo de otra cuenta. La segunda operación espera a la primera y decide con lo que la primera dejó.
- **Se bloquea la fila que representa lo que se protege, aunque se vaya a cambiar otra tabla.** Los bloqueos de una cuenta viven en `user_access_suspensions`, pero se bloquea la fila de la cuenta en `users` y después se leen sus períodos. Lo mismo para otorgar y retirar roles, dar acceso, restablecer la contraseña y la salida y el regreso. En compras, un contacto cambia solo por su proveedor: agregarlo, corregirlo, activarlo o desactivarlo bloquea la fila del proveedor con `FOR NO KEY UPDATE`, y emitir una orden lee el proveedor con sus contactos en modo compartido (abajo). La llave foránea de la orden toma sobre el contacto solo `FOR KEY SHARE`, que la actualización del contacto no espera: sin el candado del proveedor, una orden salía con un contacto retirado al mismo tiempo.
- **Siempre en el mismo orden.** Una operación que toca un colaborador y su cuenta bloquea primero `employees` y después `users`. Con un orden fijo, dos operaciones no se esperan la una a la otra para siempre.
- **Lo que se referencia y se revisa activo se lee en modo compartido.** Una operación que escribe un registro que apunta a otro, y antes revisa que el otro siga activo, lo lee con `FOR SHARE` (`findByPublicIdForShare`): espera a una edición o una desactivación en curso, y la hace esperar hasta terminar, sin frenar a otra operación que también solo lo referencia. Así lo hace `organization` con la empresa, la sede y la marca que referencia lo que se cuelga de ellas (D-84), y `fleet` con el fondo al ceder una unidad (D-133): una desactivación simultánea del fondo espera a la cesión o la hace esperar. La corrección del código del fondo, que bloquea el fondo para actualizar, sabe así si alguna cesión lo usa; un escritor futuro de cesiones, como las cargas del paso 9, toma el mismo bloqueo. `procurement` hace lo mismo con el tipo de orden al crear una plantilla o cambiarle el tipo: el `FOR KEY SHARE` de la llave foránea no choca con el `FOR NO KEY UPDATE` de la edición del tipo, así que sin el `FOR SHARE` la corrección del código no vería una plantilla que se confirma a la vez. Desde el 29/10/2026 (`914cac1a`), emitir una orden de compra lee así todo lo que revisa activo: el proveedor con sus contactos (`SupplierRepository.findByPublicIdForShare`), la empresa y la marca por `resolveScope` de la fachada de `organization`, como `fleet` las resuelve para un fondo o una unidad, y la moneda por `fetchCurrencySummaryByIsoCodeForShare` de la de `location`. Antes solo el tipo de la plantilla se leía así, y una desactivación confirmada entre la revisión y el `INSERT` dejaba pasar la orden, porque sus llaves foráneas toman solo `FOR KEY SHARE`. El modelo de cada línea sigue en una lectura simple: `vehiclecatalog` no ofrece una lectura con bloqueo, y `fleet` lee igual el modelo de una unidad.
- **Lo que se lee es la fila de después del bloqueo.** Si la transacción ya había leído esa fila antes, Hibernate la devuelve de su memoria, con los valores de antes. La lectura que bloquea refresca la entidad (`entityManager.refresh`), o la lee sin pasar por la memoria de la transacción, para que la decisión se tome con lo que hay en la base. Por eso la consulta nativa que bloquea devuelve solo el identificador, y la entidad se vuelve a leer y se refresca después: es el mismo patrón con `FOR NO KEY UPDATE` y con `FOR SHARE`. Se prueba con una operación en la que quien pide es también el afectado, que es cuando la transacción ya tenía la fila.
- **Cuando lo protegido no es una fila, se usa un bloqueo consultivo de la transacción** (`pg_advisory_xact_lock`). La protección del último superadministrador cuenta cuentas: no hay una fila que bloquear para que dos retiros de dos superadministradores distintos no pasen a la vez. El bloqueo consultivo con una llave fija hace que esas operaciones vayan de a una y se suelta solo al terminar la transacción.
- **Una migración que revisa y después cambia también bloquea primero.** Si revisa unas filas y decide con ellas —que normalizar los documentos no repita uno, que ninguna dirección esté fuera del país de su dueño—, bloquea antes las tablas contra escrituras (`LOCK TABLE … IN SHARE ROW EXCLUSIVE MODE`), así nadie cambia una fila entre la revisión y el cambio; las lecturas siguen. V76 bloquea `suppliers`; V78, `addresses` y las tablas de sus dueños: `suppliers`, `branches`, `locations`, `purchase_order_templates` y `companies`. Y no deja la aplicación esperando detrás de ella: V76, V77, V78, V80, V81 y, desde el 27/10/2026, V82 a V85 abren con `SET LOCAL lock_timeout = '5s'`, y una migración que espera un candado más de cinco segundos falla, sin cambiar nada, en vez de encolar los pedidos.
- **La base sigue siendo la última línea.** Si una restricción de la base rechaza igual lo que una carrera dejó pasar, el adaptador traduce la violación a una excepción del dominio (`AccessSuspensionOverlapException`) y el servicio responde el mismo fallo esperado que su propia revisión (`409 ACCESS_SUSPENSION_CONFLICT`), nunca un `500`. Un duplicado que ningún adaptador traduce sigue respondiendo el genérico `409 RESOURCE_CONFLICT`, y desde el 29/10/2026 (`50136e2d`) el registro nombra la restricción, sin los valores de la llave, para que un defecto como dos órdenes con el mismo número no pase sin rastro (`GlobalExceptionHandlerTest.anUntranslatedDuplicateKeyIsLoggedWithTheNameOfItsConstraint`).

`AccountLockTests`, `EmployeeLockTests` y `SuperadministratorsLockTests` prueban que una segunda transacción espera a la primera. En compras, `SupplierLockTests`, `PurchaseOrderTypeLockTests`, `PurchaseOrderTemplateLockTests` y `PurchaseOrderLockTests` prueban que la lectura que bloquea trae la fila como la encuentra el candado (también la serie que numera una orden); el tipo de orden se lee con `FOR NO KEY UPDATE` y se refresca antes de renombrarlo y antes de cambiar su estado, así un cambio no pisa el que otra petición confirmó en el medio (`aRenameKeepsTheStatusAnotherRequestCommittedMeanwhile`, `aChangeOfStatusKeepsTheNameAnotherRequestCommittedMeanwhile`), y `PurchaseOrderLockTests.anAnnulmentAndASendingInParallelLeaveTheOrderAnnulled` juega la carrera con dos transacciones reales: sin el candado, el envío devolvía como enviada una orden anulada. `FundLockTests` prueba el modo compartido del fondo al ceder, y `CompanySharedLockTests` el de la empresa.

Lo que agregó el 29/10/2026, en compras:

- `SupplierLockTests` suma la lectura en modo compartido del proveedor (`readingASupplierForShareLocksItsRowAndReadsItAsTheLockFindsIt`) y la que lo bloquea por uno de sus contactos (`readingASupplierForUpdateByOneOfItsContactsLocksItsRowAndReadsItAsTheLockFindsIt`).
- `PurchaseOrderIssuingLockTests` comprueba, con otra conexión que pasa sobre cada fila con `FOR NO KEY UPDATE SKIP LOCKED` mientras la orden está abierta, que emitirla retiene en modo compartido lo que revisa (`issuingAnOrderHoldsWhatItChecksActiveInSharedMode`), y que un proveedor, una moneda o una marca desactivados después de una primera lectura rechazan la orden sin escribir nada (`anOrderToASupplierDeactivatedMeanwhileIsRefusedAndWritesNothing`, `anOrderInACurrencyDeactivatedMeanwhileIsRefusedAndWritesNothing`, `anOrderOfABrandDeactivatedMeanwhileIsRefusedAndWritesNothing`).
- La misma clase juega con dos transacciones reales, sobre filas confirmadas de la empresa de Perú, las dos llegadas de la carrera entre el retiro de un contacto y la orden que lo nombra (`7af493e6`): la orden que llega durante el retiro lo espera, ve el contacto retirado y responde `SUPPLIER_CONTACT_INACTIVE` sin guardar nada (`anOrderNamingAContactRetiredMeanwhileWaitsForTheRetirementAndIsRefused`), y el retiro que llega durante la orden espera a que termine (`aContactRetiredWhileAnOrderNamingItIsIssuedWaitsForTheOrder`). Sin el candado del proveedor, las dos fallan.
- `CurrencyDecimalPlacesLockTests` juega las dos llegadas de la corrección de los decimales de una moneda y una orden en esa moneda: la corrección espera a la orden abierta, que la hace contar (`aCorrectionOfTheDecimalPlacesWaitsForAnOpenOrderInTheCurrency`), y la orden espera a la corrección abierta y revisa sus precios con los decimales nuevos (`anOrderWaitsForAnOpenCorrectionAndChecksItsPricesWithTheNewDecimalPlaces`).

## Dos ediciones de lo mismo: el `updatedAt` que se leyó

El bloqueo de una fila ordena dos operaciones que corren al mismo tiempo; no protege una edición que alguien abrió hace diez minutos, mientras otra persona guardaba la suya. Para eso, lo que se edita en un formulario llega con su `updatedAt` y vuelve con él al guardar:

- **Se compara bajo el candado.** El servicio lee la fila con `ForUpdate`, hace las revisiones de acceso y, antes de las reglas de su estado, compara el `updatedAt` que llegó con el de la fila, en milisegundos, la precisión de un momento en el navegador. Si no coinciden, responde `422 <RECURSO>_OUTDATED` y no guarda nada.
- **Lo hacen** el proveedor y la plantilla desde el paso 7 (`SUPPLIER_OUTDATED`, `PURCHASEORDERTEMPLATE_OUTDATED`) y, desde el 29/10/2026, el borrador de una orden de compra (`PURCHASEORDER_OUTDATED`, V85, D-138) y cada contacto de un proveedor (`SUPPLIER_CONTACT_OUTDATED`, D-137).
- **La revisión la llena el sistema** (D-42): `updated_at` y `updated_by` los pone la auditoría de la clase base. Por eso `purchase_orders` pasó con V85 a `RevisedAbstractPersistenceEntity`, sin cambiar ninguna clase base, y las órdenes que ya existían tomaron su creación como último cambio.
- **Lo que no toca la cabecera también mueve la revisión.** Un guardado que cambia solo las líneas de una orden deja la entidad sin cambios, y la auditoría no escribiría nada: el repositorio mueve la revisión con una sentencia propia (`PurchaseOrderJpaRepository.recordRevision`), como la plantilla cuando cambia uno de sus archivos. Enviar, cumplir y anular la mueven también.

`PurchaseOrdersIntegrationTests.aSaveOpenedBeforeTheLastChangeIsOutdated` y `SupplierContactsIntegrationTests.aCorrectionOpenedBeforeTheLastChangeIsOutdatedAndKeepsNothing` lo prueban por la API; en PostgreSQL, con dos transacciones, `SupplierLockTests.anEditOverAnotherThatCommittedMeanwhileIsOutdatedAndKeepsTheOther` y `aCorrectionOfAContactOverAnotherThatCommittedMeanwhileIsOutdatedAndKeepsTheOther`, y `PurchaseOrderTemplateLockTests.aChangeOfStatusOverAnEditThatCommittedMeanwhileIsOutdated`.

## Las tablas de empresa y Row Level Security

Toda tabla con `company_id`, y `companies` por su propio `id`, tiene Row Level Security activada y forzada (V29): la base deja ver y escribir solo las filas de las empresas que alcanza quien actúa. `addresses`, sin `company_id`, la tiene por el país de la dirección (V78, diccionario §2.5, D-134): se ve donde se alcanza una empresa de ese país, o con `SYSTEM`. Cada tabla de empresa lleva las políticas de V29 —una para leer, una para insertar y, salvo en las que solo se insertan, como la bitácora o el historial de estados, una para actualizar, todas donde alcanza la transacción y ninguna para borrar—: las de compras las tienen desde V83 (28/10/2026), en lugar de la política única por tabla que V63 a V74 les habían dado, que dejaba ver una fila sin empresa incluso a una transacción sin alcance; y `kept_files`, los bytes que no vencen, lleva desde V84 la empresa de la plantilla o de la orden que nombra su llave, con la misma Row Level Security. `RowLevelSecurityMigrationTests.everyTableWithACompanyHasThePoliciesOfV29` exige esa forma en cada tabla con `company_id`. Para la persistencia eso significa:

- **El alcance se fija al comenzar cada transacción, desde un solo punto.** `AccessScopeJpaTransactionManager` llama a `set_config(…, true)` sobre la conexión de la transacción con lo que `DatabaseAccessScope` tiene para el hilo: `app.access_scope`, `app.company_ids` y `app.user_id`. Nunca al abrir la conexión. Un `JdbcTemplate` dentro de la misma transacción comparte la conexión y sus valores.
- **Sin alcance no se ve nada.** Una lectura fuera de toda transacción, o de un hilo que nadie preparó, devuelve cero filas de empresa. Por eso los servicios de consultas leen en una transacción de solo lectura.
- **`SYSTEM` siempre es explícito.** El arranque, los procesos programados y las migraciones de datos lo fijan para su propio trabajo. Una comprobación que necesita ver más allá de las empresas de quien llama lo pide con `SystemAccessScope`, que devuelve solo datos sueltos y vuelve al alcance anterior; antes de cambiar de alcance escribe lo pendiente y no deja en memoria nada leído con el otro.
- **Una pregunta sobre todas las empresas va en `SYSTEM`.** Si un país tiene direcciones lo cuentan las de todas las empresas, que quien corrige el país puede no ver: `CountryCommandServiceImpl` bloquea el país, entra en `SYSTEM`, pregunta, sale y vuelve a leer el país, porque cambiar de alcance vacía lo leído; `CountryViewFactory` pregunta igual para «tiene direcciones». Desde el 29/10/2026, `CurrencyCommandServiceImpl` hace lo mismo para saber si una moneda ya tiene montos (§4.1): bloquea la moneda, pregunta en `SYSTEM` a los `CurrencyUsage` de los contextos con montos y vuelve a leerla ([mapa de contextos](mapa-de-contextos-millennium-mobility.md), §6.6).
- **La aplicación no es dueña de las tablas.** Se conecta como `erp_mm_app`, que no puede saltarse la regla ni borrar filas de negocio; Flyway migra como `erp_mm_owner`. Las pruebas se conectan igual, así que un camino que olvide fijar el alcance falla en las pruebas y no en producción. Desde el 28/10/2026 (`852214f3`), fuera de `dev` la aplicación tampoco arranca si PostgreSQL dice que su cuenta es superusuario o tiene `BYPASSRLS`, que se saltan Row Level Security aunque esté forzada: `DatabaseAccountsConfiguration` lo pregunta antes de que el servidor atienda una petición (`DatabaseAccountsConfigurationTests.superuserFails` y `bypassRowLevelSecurityFails`; `ApplicationAccountQueryTests.readsTheAccountTheApplicationIsConnectedWith`). En `dev`, sin `FLYWAY_USER`, sigue la excepción de una sola cuenta (README del backend, «Cuentas de la base»).
- **Un script local que borra filas forzadas** levanta `FORCE` solo en su transacción (`NO FORCE ROW LEVEL SECURITY`, borrar, `FORCE`), también en `addresses` desde V78 y en `role_permissions`: sin política de `DELETE`, un borrado no alcanza ninguna fila, ni siquiera para la dueña.

El filtro de la aplicación sigue siendo obligatorio: cada consulta filtra por el alcance de quien llama. La base es la defensa de abajo, para que una consulta que olvidó su filtro devuelva menos filas, nunca las de otra empresa (reglas 5.1 a 5.3 y 5.7 de la skill `backend-module`). Las instrucciones para correr la aplicación y las pruebas con las dos cuentas están en el README del backend.

## Por qué también los catálogos

Entre el 30/09/2026 y el 01/10/2026, `location`, `vehiclecatalog` y `organization` usaron un camino corto: el modelo era la entidad de JPA y el repositorio extendía `JpaRepository`. Se eligió porque un catálogo no tiene reglas que proteger y el patrón completo duplica campos.

Se descartó porque el dominio quedaba atado a la persistencia: el modelo llevaba anotaciones de JPA y la interfaz del repositorio dependía de Spring Data, con métodos como `delete` que el dominio no debe tener. El error de `createdAt` no venía de separar las capas, sino de reconstruir la fila al guardar, y eso se corrige cargándola. Con esa corrección, el costo de separar es una entidad y un ensamblador por tabla, a cambio de un dominio que se prueba sin nada alrededor y de una regla sin excepciones.

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

**Con el agregado en el dominio, la regla siempre se cumple:**

```java
public class Vehicle {                          // sin @Entity, sin setter de kilometraje
    private Integer currentMileage;

    public boolean canAcceptMileage(int reading) {
        return currentMileage == null || reading >= currentMileage;
    }

    public void applyMileage(int reading) {
        if (!canAcceptMileage(reading))
            throw new IllegalStateException("Regla verificada antes de llamar");
        this.currentMileage = reading;
    }
}
```

El servicio pregunta `canAcceptMileage` y, si la lectura retrocede, devuelve `Result.failure`: un fallo esperado del negocio no se lanza como excepción (regla 3.1). Las exigencias de JPA se van a `VehiclePersistenceEntity`, que solo sabe de columnas, y el código del negocio nunca la toca.

## Objetos de valor y rangos

Un objeto de valor que solo valida su formato, como `Vin`, `Plate` o un código ISO, vive en el dominio y se convierte a columna en la entidad de persistencia, con un conversor (`infrastructure/persistence/jpa/converters`) o en el ensamblador. Lo mismo vale para `Money` (monto más moneda, dos columnas) o un período `[desde, hasta)`, que en la base es un `daterange` o un `tstzrange`.

## Lo que vale siempre

- **Las columnas de registro las llena el sistema (D-42).**
  - Las fechas las pone la auditoría de Spring Data (`@CreatedDate`, `@LastModifiedDate`). Nadie las calcula con `Instant.now()`.
  - `created_by` y `updated_by` los toma de la cuenta de la sesión el auditor que aporta `iam` (`CurrentUserAuditorAware`).
  - Un proceso sin sesión asigna `SYSTEM` de forma explícita.
- **La clase base se elige por las columnas que pide el cuadro del diccionario, nunca por el nombre.** Todas viven en `shared/infrastructure/persistence/jpa/entities` y forman una sola cadena, con `id` y `created_at` en la raíz:

| Columnas de registro del cuadro | Clase base | Ejemplos |
|---|---|---|
| `created_at` | `CreatedAbstractPersistenceEntity` | `vehicle_equipments` |
| `created_at`, `updated_at` | `AuditableAbstractPersistenceEntity` | Las seis tablas del bloque 1 |
| `created_at`, `created_by` | `CreatedByAbstractPersistenceEntity` | `user_access_suspensions`, `purchase_invoices`, `credit_notes`, `supplier_payments`, `vehicle_fund_assignments` |
| `created_at`, `updated_at`, `created_by` | `AuthoredAbstractPersistenceEntity` | `users` |
| `created_at`, `updated_at`, `created_by`, `updated_by` | `RevisedAbstractPersistenceEntity` | `companies`; `purchase_orders` desde V85 |
| Ninguna | Ninguna: la entidad declara su propio `id` | `roles`, `user_roles` |

  - `CreatedAbstractPersistenceEntity` es la raíz: `id`, `created_at` y el oyente de auditoría. `Auditable` la extiende y agrega `updated_at`; `Authored` agrega `created_by` y `Revised`, `updated_by`. `CreatedBy` extiende la raíz y agrega `created_by`, con las mismas anotaciones que `Authored`: es la única duplicación aceptada.
  - Extender una clase que trae una columna que el cuadro no pide es una violación: con `ddl-auto: validate`, la aplicación no arranca.
  - `CreatedAbstractPersistenceEntity` y `CreatedByAbstractPersistenceEntity` llegaron en el paso 3, con `user_access_suspensions`, su primera tabla. `purchase_orders`, que se edita mientras es borrador, pasó a `RevisedAbstractPersistenceEntity` con V85 (29/10/2026, D-138), sin cambiar ninguna clase base. El cuadro del diccionario (§9.2) tiene sus `updated_at` y `updated_by` desde la v1.33 (30/10/2026).
- **Las filas no se borran**: se desactivan, se cierran o se anulan. Por eso las interfaces de repositorio no tienen `delete`.
- **Una búsqueda pliega los dos lados en la base** (D-142, V86, desde el 30/10/2026). El patrón de `q` lo arma `PageRequests.containsPattern`, en minúsculas y con los comodines escapados, y la consulta compara `fold_search_text(x) like fold_search_text(cast(:pattern as String)) escape '\'` en una `@Query`, o usa `PageRequests.containsFolded` en una consulta por criterios, que liga el patrón como parámetro. La función quita las tildes de las vocales, la ñ y la ç y pasa a minúsculas, en la base y no en Java, para que los dos lados plieguen igual sea cual sea el `ctype` de la base. Es `IMMUTABLE` y `PARALLEL SAFE`, sin la extensión `unaccent`, y sin índice: una búsqueda que contiene un texto no usaría uno de árbol, y si una lista crece se agrega `pg_trgm` sobre `fold_search_text(columna)` sin cambiar la aplicación. Un filtro exacto, como el puesto de un colaborador, no pasa por ella. Una consulta nueva con `q` usa lo mismo, nunca `lower()` sola. Lo prueba `SearchTextFoldingTests`.
- **Las funciones de la base son de la cuenta que migra.** V86 crea `fold_search_text(text)` como Flyway, igual que `bump_data_version()` (V53), y le da `EXECUTE` a la cuenta de la aplicación por su nombre, para que retirarle el permiso a `PUBLIC` algún día no se lo quite a las búsquedas. En una base ya migrada, `hand-over-database.sql` la pasa a `erp_mm_owner`, como a las funciones de Row Level Security (V29, V78) y a los disparadores de V53 y V69, y le da `EXECUTE` a `erp_mm_app`; `HandOverDatabaseScriptTests` falla si una función nueva de las migraciones no tiene su línea en el script.
- **Solo el `public_id` sale por la API**; el `id` interno nunca aparece en rutas ni respuestas.
- **El dominio no importa nada de otro contexto**: lo que necesite de él lo pide por su fachada ACL.
