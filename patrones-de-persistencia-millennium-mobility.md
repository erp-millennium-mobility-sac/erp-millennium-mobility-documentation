# Patrones de persistencia — ERP Millennium Mobility

**Actualizado:** 01/10/2026 · **Fuente:** skill `backend-module` (§1 y pasos de construcción; `module-template.md`), `architecture-rules.md` (reglas 2.1 y 3.1) y diccionario de datos (D-42). Si este documento y el diccionario no coinciden, manda el diccionario.

Todos los agregados del backend se guardan de la misma manera: el modelo de negocio en `domain`, la tabla en `infrastructure`. Este documento explica qué hace cada pieza, cómo se guarda una edición sin perder datos y por qué no hay excepciones para los catálogos.

## La regla en una frase

> El dominio es solo dominio. JPA y Spring Data son tecnología de persistencia y viven en `infrastructure`, en todos los contextos, catálogos incluidos.

## Las piezas de cada agregado

| Pieza | Capa | Para qué existe |
|---|---|---|
| **Agregado** | `domain/model` | Contiene las reglas del negocio y nada más. Sin anotaciones de JPA, sin constructor vacío para Hibernate, sin setters para el mapeo. Tiene dos constructores: el que crea uno nuevo, validando, y el que lo reconstituye desde la base |
| **Interfaz de repositorio** | `domain/repositories` | Dice qué necesita el dominio para guardar y leer, en sus términos y solo eso. No tiene `delete`: las filas no se borran |
| **Entidad de persistencia** | `infrastructure/persistence/jpa/entities` | Tiene la forma de la tabla: columnas, llaves foráneas y relaciones. Extiende la clase base de auditoría |
| **Ensamblador** | `infrastructure/persistence/jpa/assemblers` | Reconstituye el agregado desde la fila y copia los valores del agregado sobre una fila |
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

`CurrencyAuditTests` y `CountryAuditTests` prueban que una edición conserva la fecha de creación. Cada agregado nuevo necesita una prueba igual.

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
  - `created_at` y `updated_at` vienen de `AuditableAbstractPersistenceEntity`.
  - `created_by` viene de `AuthoredAbstractPersistenceEntity`, que lo toma de la cuenta de la sesión.
  - Nadie calcula estas fechas con `Instant.now()`.
  - Un proceso sin sesión asigna `SYSTEM` de forma explícita.
  - Las dos clases base traen siempre `created_at` y `updated_at`. Sirven para las tablas cuyo cuadro pide las dos fechas: con `created_by`, la clase con autor; sin él, la clase con fechas. Las cinco tablas de geografía usan la clase con fechas.
  - **Pendiente antes del paso 3:** hay tablas que piden `created_at` y `created_by` pero no `updated_at`, como `purchase_orders`, `credit_notes` y los bloqueos de cuenta. Ninguna de las dos clases encaja: extenderlas agregaría una columna que su cuadro no pide. Hace falta una clase base solo de creación en `shared`, que se decide antes de construir la primera de esas tablas.
  - `updated_by` también lo pide D-42, pero todavía no hay clase base que lo llene. La primera tabla que lo necesite debe agregarla en `shared` (con `@LastModifiedBy`), no en su propio contexto.
- **Las filas no se borran**: se desactivan, se cierran o se anulan. Por eso las interfaces de repositorio no tienen `delete`.
- **Solo el `public_id` sale por la API**; el `id` interno nunca aparece en rutas ni respuestas.
- **El dominio no importa nada de otro contexto**: lo que necesite de él lo pide por su fachada ACL.
