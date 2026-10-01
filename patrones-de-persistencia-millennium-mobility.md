# Patrones de persistencia — ERP Millennium Mobility

**Actualizado:** 30/09/2026 · **Fuente:** skill `backend-module` (§1, tabla de contextos y criterio; `module-template.md`, camino corto), `architecture-rules.md` (regla 3.1) y diccionario de datos (D-42). Si este documento y el diccionario no coinciden, manda el diccionario.

Cada agregado del backend se guarda de una de dos maneras: el **camino corto**, con el modelo como entidad de JPA, o el **patrón completo**, con el modelo de negocio separado de la tabla. Este documento explica cuándo usar cada una, con un ejemplo de cada caso.

## La regla en una frase

> Si el agregado tiene reglas de negocio que hay que poder probar sin levantar una base de datos, va con el patrón completo; si es un catálogo con validaciones de formato, va con el camino corto. El cambio se hace por agregado: en un contexto de camino corto, solo el agregado que gane reglas propias pasa al patrón completo.

## Los dos patrones

| | Camino corto | Patrón completo |
|---|---|---|
| Clases por agregado | 2: la entidad, que es a la vez el modelo, y su repositorio | 6: el agregado, la interfaz de repositorio del dominio, la entidad de persistencia, el ensamblador, el adaptador y el repositorio de Spring Data |
| Dónde viven las anotaciones de JPA | En el modelo de dominio | Solo en la entidad de persistencia |
| El repositorio | Extiende `JpaRepository` directamente | Es una interfaz del dominio; el adaptador la implementa con Spring Data |
| Contextos asignados (skill `backend-module` §1) | `location` (ya construido), `vehiclecatalog`, `organization` | `iam` (ya construido), `fleet`, `procurement` |
| Ejemplos | Monedas, países, colores, marcas | Usuarios, vehículos, facturas, notas de crédito |

Lo demás es igual en los dos: comandos y consultas como `record`, servicios de comandos que devuelven `Result`, recursos REST con sus transformadores y controladores sin lógica.

## Qué hace cada pieza del patrón completo

| Pieza | Para qué existe |
|---|---|
| **Agregado de dominio** | Contiene las reglas del negocio y nada más. No tiene constructor vacío obligatorio, ni setters para Hibernate, ni anotaciones de JPA |
| **Interfaz de repositorio del dominio** | Dice qué necesita el dominio para guardar y leer, sin depender de Spring Data |
| **Entidad de persistencia** | Tiene la forma de la tabla: columnas, llaves foráneas y relaciones |
| **Ensamblador** | Traduce entre el agregado y la entidad cuando sus formas son distintas |
| **Adaptador** | Implementa la interfaz del dominio con el repositorio de Spring Data |

## Cómo reconocer que un agregado necesita el patrón completo

El criterio de la skill es uno solo: que el agregado tenga reglas de negocio que hay que poder probar sin base de datos. Estas dos señales ayudan a reconocerlas:

1. **Una regla que no se puede romper.** El kilometraje no retrocede como una lectura más. Un estado solo pasa a los estados permitidos. Una nota de crédito no excede el valor que le queda a la unidad. JPA necesita un constructor vacío y suele terminar con setters, que son puertas para dejar el objeto en un estado inválido.
2. **El agregado decide cómo se escribe una historia.** Por ejemplo, el historial de estados del vehículo: se agrega una fila nueva, nunca se modifica una existente.

Que el modelo tenga otra forma que la tabla **no basta por sí solo**:
- Un objeto de valor que solo valida su formato, como `Vin`, `Plate` o un código ISO, es una validación de formato. Se convierte a columna con un conversor (`infrastructure/persistence/jpa/converters`) y no cambia el tratamiento.
- Lo mismo vale para `Money` (monto más moneda, dos columnas) o un período `[desde, hasta)`, que en la base es un `daterange` o un `tstzrange`.
- Si además hay reglas, como en `User`, que reúne filas de `users` y `user_roles` y decide quién puede otorgar qué rol, entonces sí va completo, pero por las reglas.

Si no hay ninguna regla, va el camino corto: el ensamblador solo copiaría campos y el adaptador no protegería nada.

## Ejemplo: un formulario y una caja fuerte

### Caso 1 · La moneda es un formulario: camino corto

Una moneda es una ficha con datos. Lo que tiene la clase es lo que tiene la tabla, y no hay regla que proteger. En el ejemplo se omiten `publicId` e `isActive` para abreviar.

```java
@Entity
@Table(name = "currencies")
public class Currency extends AuditableAbstractPersistenceEntity {
    private String isoCode;      // "PEN"
    private String name;         // "Sol peruano"
    private String symbol;       // "S/"
    private short decimalPlaces; // 2
}

public interface CurrencyRepository extends JpaRepository<Currency, Long> {
    Optional<Currency> findByIsoCode(String isoCode);
}
```

Con el patrón completo hay además una `CurrencyPersistenceEntity` con los mismos campos y un ensamblador que copia de una a la otra; así estaba en la primera versión del contexto. Es el doble de código y un sitio más donde se puede perder un campo. Eso pasó: la clase base compartida no deja asignar `createdAt` desde afuera, así que los ensambladores de monedas y países no podían copiarlo, y las respuestas de edición devolvían la fecha de alta vacía. Al pasar monedas y países al camino corto, el error desapareció.

### Caso 2 · El vehículo es una caja fuerte: patrón completo

El negocio dice que **el odómetro no retrocede**: si hoy marca 15 000 km, nadie puede registrar 12 000 como una lectura más. Si le cambiaron el tablero, se registra como sustitución de odómetro, que es un hecho distinto (diccionario §8.5).

**Con una sola clase de JPA, la regla se puede saltar:**

```java
@Entity
public class Vehicle {
    private Integer mileage;

    public void setMileage(Integer mileage) {   // cualquiera puede llamarlo
        this.mileage = mileage;
    }
}

vehicle.setMileage(12000);   // nadie lo impide: la regla quedó rota
```

**Con el agregado separado, la regla siempre se cumple:**

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

Es el mismo agregado de la plantilla de la skill (`module-template.md`). El servicio pregunta `canAcceptMileage` y, si la lectura retrocede, devuelve `Result.failure`: un fallo esperado del negocio no se lanza como excepción (regla 3.1). La excepción solo salta si alguien se salta esa pregunta, y eso es un error de programación. Fuera de la sustitución de odómetro, no existe otra forma de bajar el kilometraje.

Las exigencias de JPA (constructor vacío, setters, anotaciones) se van a `VehiclePersistenceEntity`, que solo sabe de columnas. El ensamblador la traduce al agregado y el adaptador la guarda. El código del negocio nunca la toca.

**La diferencia:** un formulario guarda lo que le escriban; una caja fuerte solo se abre con la combinación, y para eso necesita su propia cerradura, que es el agregado separado.

## Cómo decidir

| Pregunta | Si la respuesta es "no" | Si la respuesta es "sí" |
|---|---|---|
| ¿Hay una regla que se rompería si alguien cambia un campo directamente? | Camino corto | Patrón completo |
| ¿El agregado decide cómo se agrega una fila de historia? | Camino corto | Patrón completo |

Un objeto de valor o un rango por sí solos no cambian la respuesta: se resuelven con un conversor.

El cambio se hace por agregado, no por contexto. Si un catálogo de `vehiclecatalog` gana una regla propia, ese agregado pasa al patrón completo y los demás del contexto siguen en el camino corto.

## Lo que vale para los dos patrones

- **Las columnas de registro las llena el sistema (D-42).**
  - `created_at` y `updated_at` vienen de `AuditableAbstractPersistenceEntity`.
  - `created_by` viene de `AuthoredAbstractPersistenceEntity`, que lo toma de la cuenta de la sesión.
  - Nadie calcula estas fechas con `Instant.now()`.
  - Un proceso sin sesión asigna `SYSTEM` de forma explícita.
  - Las dos clases base traen siempre `created_at` y `updated_at`. Sirven para las tablas cuyo cuadro pide las dos fechas: con `created_by`, la clase con autor; sin él, la clase con fechas. Las cinco tablas de geografía usan la clase con fechas.
  - **Pendiente antes del paso 3:** hay tablas que piden `created_at` y `created_by` pero no `updated_at`, como `purchase_orders`, `credit_notes` y los bloqueos de cuenta. Ninguna de las dos clases encaja: extenderlas agregaría una columna que su cuadro no pide. Hace falta una clase base solo de creación en `shared`, que se decide antes de construir la primera de esas tablas.
  - `updated_by` también lo pide D-42, pero todavía no hay clase base que lo llene. La primera tabla que lo necesite debe agregarla en `shared` (con `@LastModifiedBy`), no en su propio contexto.
- **Las filas no se borran**: se desactivan, se cierran o se anulan.
- **Solo el `public_id` sale por la API**; el `id` interno nunca aparece en rutas ni respuestas.
- **El dominio no importa nada de otro contexto**: lo que necesite de él lo pide por su fachada ACL.
