# Patrones de persistencia — ERP Millennium Mobility

**Actualizado:** 06/10/2026 (clases base por forma de las columnas de registro; el adaptador carga y copia también en `iam`) · **Fuente:** skill `backend-module` (§1 y pasos de construcción; `module-template.md`), `architecture-rules.md` (reglas 2.1 y 3.1) y diccionario de datos v1.15 (D-42). Si este documento y el diccionario no coinciden, manda el diccionario.

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

## Por qué también los catálogos

Entre el 30/09 y el 01/10/2026, `location`, `vehiclecatalog` y `organization` usaron un camino corto: el modelo era la entidad de JPA y el repositorio extendía `JpaRepository`. Se eligió porque un catálogo no tiene reglas que proteger y el patrón completo duplica campos.

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
- **La clase base se elige por las columnas que pide el cuadro del diccionario, nunca por el nombre.** Todas viven en `shared/infrastructure/persistence/jpa/entities` y forman una sola cadena, con `id` y `created_at` en la raíz. `CreatedAbstractPersistenceEntity` y `CreatedByAbstractPersistenceEntity` todavía no existen (ver el pendiente de abajo):

| Columnas de registro del cuadro | Clase base | Ejemplos |
|---|---|---|
| `created_at` | `CreatedAbstractPersistenceEntity` | `vehicle_equipments` |
| `created_at`, `updated_at` | `AuditableAbstractPersistenceEntity` | Las seis tablas del bloque 1 |
| `created_at`, `created_by` | `CreatedByAbstractPersistenceEntity` | `user_access_suspensions`, `purchase_orders`, `purchase_invoices`, `credit_notes`, `supplier_payments`, `vehicle_fund_assignments` |
| `created_at`, `updated_at`, `created_by` | `AuthoredAbstractPersistenceEntity` | `users` |
| `created_at`, `updated_at`, `created_by`, `updated_by` | `RevisedAbstractPersistenceEntity` | `companies` |
| Ninguna | Ninguna: la entidad declara su propio `id` | `roles`, `user_roles` |

  - `CreatedAbstractPersistenceEntity` es la raíz: `id`, `created_at` y el oyente de auditoría. `Auditable` la extiende y agrega `updated_at`; `Authored` agrega `created_by` y `Revised`, `updated_by`. `CreatedBy` extiende la raíz y agrega `created_by`, con las mismas anotaciones que `Authored`: es la única duplicación aceptada.
  - Extender una clase que trae una columna que el cuadro no pide es una violación: con `ddl-auto: validate`, la aplicación no arranca.
  - **Pendiente para el paso 3:** `CreatedAbstractPersistenceEntity` y `CreatedByAbstractPersistenceEntity` todavía no existen. Se escriben en el mismo cambio que `user_access_suspensions`, su primera tabla, sin tocar las entidades actuales ni Flyway. Que `purchase_orders`, que se edita mientras es borrador, lleve `updated_at` lo decide el diccionario; si lo lleva, pasa a la clase con autor sin cambiar ninguna clase.
- **Las filas no se borran**: se desactivan, se cierran o se anulan. Por eso las interfaces de repositorio no tienen `delete`.
- **Solo el `public_id` sale por la API**; el `id` interno nunca aparece en rutas ni respuestas.
- **El dominio no importa nada de otro contexto**: lo que necesite de él lo pide por su fachada ACL.
