# Plan de migración a los contextos objetivo — ERP Millennium Mobility

**Estado:** aprobado por el responsable el 19/11/2026, junto con el [mapa de contextos objetivo](mapa-de-contextos-objetivo-millennium-mobility.md), para empezar por las fases 0, 1 y 6; las decisiones de cada fase siguiente se resuelven al llegar a ella. Versión 3, del 19/11/2026: incorpora la auditoría del mismo día, que verificó el plan contra el mapa objetivo y contra el código (§9).

**Fuente:** el inventario del código de `develop` (backend `ed5adc12` y frontend `835ed667`) del 18/11/2026, la rama sin integrar `refactor/remove-equipment-type-flags` (backend `d6d23fe8`, frontend `37dbc8dd`), las reglas de arquitectura de las skills `backend-module`, `frontend-feature` y `architecture-review`, y *Domain-Driven Design* de Eric Evans (las referencias «cap. n» son capítulos del libro). Las rutas de archivo y los números de línea citados son los de ese `develop`: antes de ejecutar una fase se comprueban otra vez, porque el código sigue cambiando.

**Objetivo.** Llevar lo que hoy está construido a los contextos del mapa objetivo, por fases pequeñas que dejan el sistema funcionando al final de cada una, sin cambiar lo que hace el sistema salvo donde una fase lo dice. El bloque 12 (clientes, contratos, cobranza y traspasos) no se migra: se construye después, sobre la estructura que deja este plan, y aquí solo se prepara (fase 10).

---

## 1. Lo que no cambia en ninguna fase

Estas garantías hacen que cada fase sea un cambio de estructura y no de comportamiento:

| Qué | Por qué se mantiene |
|---|---|
| **Las tablas, sus nombres y sus migraciones** | Un contexto es dueño de sus tablas por el código que las escribe, no por su nombre. Ninguna fase crea, renombra ni borra una tabla. Las únicas migraciones nuevas son de la fase 8: un índice en su primera entrega y el retiro de dos columnas en la segunda, después de dejar de usarlas. `fleet_equipments` conserva su nombre aunque pase a `equipment`. Las migraciones de D-260 no cuentan aquí, porque D-260 es trabajo funcional y no una fase |
| **Row Level Security y `data_versions`** | Las políticas van por tabla y siguen iguales; solo cambia qué `Jdbc<Contexto>DataVersions` nombra cada tabla, y ninguno de un contexto de negocio nombra una tabla de otro |
| **Las rutas de la API** | Las rutas no llevan el contexto (convenciones §2.1). Una ruta puede pasar a otro módulo sin cambiar: `/vehicles/{id}/fund-assignments` a `financing`, `/vehicles/{id}/purchase-cost` a `procurement` y `/employees/{id}/termination` y `/reactivation` a `organization`. El controlador viejo sale en el mismo commit en que entra el nuevo: una ruta mapeada dos veces impide arrancar |
| **Los códigos de los permisos, de los errores, las claves de los mensajes y los nombres del esquema OpenAPI** | Son contratos por nombre con el catálogo, el frontend, los bundles y las pruebas `*OpenApiTests`. Renombrar permisos es una decisión aparte (mapa objetivo, §7) |
| **El orden de los candados** | Cada fase conserva las mismas sentencias de bloqueo en el mismo orden, aunque una pase por otra fachada; las fases que cruzan una frontera nueva lo dicen y lo prueban |
| **El comportamiento** | Cada fase es una refactorización: las mismas pruebas pasan antes y después. Las excepciones son el recurso de la unidad, que cambia solo si se elige la opción B de la fase 4 o la B de la fase 5, y la regla del inventario de equipos de la fase 2 §2.6, que se construye con el monitoreo y no en esta migración |

---

## 2. Cómo se trabaja

- **Pasos pequeños que dejan el sistema sano** (cap. 13, *Refactoring Toward Deeper Insight*). Una fase se integra completa o no se integra; nunca queda un contexto a medio mover en `develop`.
- **Una fase, una rama por repositorio:** `refactor/<fase>` en el backend, en el frontend si la fase lo toca, y en la documentación. Cada push a `develop` despliega `dev`: el backend en App Platform (`deploy_on_push`) y el frontend en Cloudflare Pages, por separado y con minutos de diferencia. Por eso:
  - si el contrato de la API no cambia (todas las fases con sus opciones A), se integra primero el backend y después el frontend: cada uno funciona con la versión anterior del otro;
  - si cambia (la opción B de la fase 4 o de la 5), se integra en tres pasos: el backend suma el subrecurso nuevo y conserva el campo viejo; el frontend pasa al subrecurso; el backend retira el campo;
  - la rama de la documentación se integra a `master` el mismo día que la del backend.
- **Lo que está en curso al desplegar.** De las fases, solo la 8 cambia el esquema (§6); fuera de ellas, también el trabajo previo de D-260. No se lanza una carga de archivos mientras se despliega una fase que mueve su código (1, 3 y 4).
- **Primero se caracteriza.** Las fases que tocan reglas delicadas —la habilitación (2), la carga inicial (3), las cesiones (4), el costo de la unidad (5) y la salida del colaborador (7)— empiezan escribiendo pruebas de caracterización contra el código de hoy, en verde en `develop`, y terminan con esas mismas pruebas sin cambios.
- **Cada fachada nueva trae su prueba de contrato.** En el libro, la relación *Customer/Supplier* se sostiene con pruebas automáticas de aceptación en la frontera, que corre el proveedor (cap. 14). Con un solo equipo eso se traduce en una prueba por fachada o puerto nuevo, en el módulo que lo ofrece, que fija lo que el consumidor espera.
- **El CI corre en la rama antes de integrar.** Cada push a `develop` despliega `dev` sin esperar al CI, así que la suite completa se corre en la rama (*Run workflow*) y se integra solo en verde. `gh` no está autenticado en la máquina de trabajo: el resultado del CI lo mira el responsable en GitHub, o se autentica `gh` antes de empezar.
- **El mapa se actualiza en la misma fase** (cap. 14, *Context Map*). Lo que la fase mueve pasa al mapa vigente, a `contextos/` y al diccionario §3 en el mismo cambio. En el mapa objetivo, la parte integrada se queda, con la marca «Integrada en la fase n (fecha)». Si el mapa objetivo ya describe el estado final de la fase, su texto no se reescribe: pasa al mapa vigente y se marca integrado.
- **Sin trabajo funcional en el área que se mueve mientras dura su fase.** Las fases 4 y 5 tocan los mismos archivos de `fleet` (`FleetContextFacade` y su implementación, `fleet-api.ts` y la ficha de la unidad con su spec; con una opción B, también `vehicles.store.spec.ts` o `e2e/support/mocks/fleet.ts`) y van en ese orden, una después de la otra.
- **El monitoreo y las fases siguientes.** El monitoreo del GPS es trabajo funcional y va después de la fase 2. La fase 8 va antes de empezarlo o después de terminarlo, nunca durante: los dos cambian `VehicleEquipmentInstallations`, `EquipmentType` y la fachada de `equipment`. Mientras se construye el monitoreo, las fases 3 a 7 no tocan `equipment` ni la instalación.
- **Las pruebas, a la medida del cambio:**
  - en local, en una base nueva de `erp_mm_owner` y nunca con el `.env` remoto: las pruebas del contexto que se mueve, las de quien lo consume, las de arquitectura (`ModularityTests`, `LayerDependencyTests`, `ForeignAclStaysInTranslationLayerTests`, `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`, `DeadlockRetryTests`, `JdbcDataVersionsTests`, `NoDeleteEndpointsTests`, `OwnErrorCodesHaveMessagesTests`) y `ErpMillenniumMobilityBackendApplicationTests`, que arranca el contexto y detecta los beans repetidos;
  - en el frontend, el build, las specs de lo movido y de lo que cambia, y `lint:styles`;
  - todo lo demás, en el CI. Las imágenes de referencia del e2e se nombran por la pantalla (`screen.name`), no por la ruta: **no se regeneran**, y una que cambie es una regresión.
- **Las skills son cuatro copias locales** —`.claude` y `.cursor` del backend y del frontend—, fuera de git: `/.claude/skills/` y `/.cursor/skills/` están en el `.gitignore` de los dos repositorios. Nombran los contextos, sus `allowedDependencies` y la lista de contextos de la búsqueda de la regla 1.2 de `architecture-review`. Nunca se versionan ni se suben (decisión 15 de la fase 0, resuelta): la rama de una fase no las lleva, y cada fase las actualiza a mano en las cuatro, que quedan idénticas, el día que integra.

---

## 3. Las fases

| Fase | Qué | Contexto que forma o ajusta | Costo | Riesgo | Depende de |
|---|---|---|---|---|---|
| 0 | Preparación | — | 2–3 días | Bajo | — |
| 1 | La bitácora y las cargas salen de `shared` | `audit`, `imports` | 3–4 días | Bajo | 0 |
| Antes de 2 | D-260, sobre la rama `refactor/remove-equipment-type-flags` corregida: trabajo funcional, no una fase. Dos entregas: la primera con las marcas de equipo, D-260 y V140 si la decisión 10 la conserva; la segunda borra `blocks_availability` y, si la decisión 10 lo retira, `requires_serial` | `fleet`, `vehiclecatalog` | 6–9 días | Medio | 0, con la decisión 10 resuelta |
| 2 | Equipamiento | `equipment`, `fleet`, `vehiclecatalog`, `procurement` | 10–14 días | Medio | 0 y el trabajo previo de D-260 |
| 3 | La migración inicial como capa anticorrupción | `legacyimport`, `procurement` | 5–7 días | Medio | 1 |
| 3b | Borrar `legacyimport` (con la decisión 4, borrar el código de la carga en lugar de la fase 3) | `legacyimport`, `procurement`, `fleet` | 1–2 días | Bajo | La puesta en marcha, cuando la carga inicial ya corrió en producción y se cerró |
| 4 | Financiamiento | `financing`, `fleet` | 9–12 días | Medio | 1, con `imports::acl` completo, y las decisiones de la fase 4 |
| 5 | El servicio publicado de compras | `procurement`, `fleet`, `reporting` | 6–9 días | Medio | 0 (crea `procurement::acl` si la fase 3 no lo hizo); después de la 4 |
| 6 | Las cuentas bancarias se quedan en `organization`, con un puerto por consumidor | `organization`, `procurement` | Menos de 1 día | Bajo | 0 |
| 7 | La salida y el regreso del colaborador, en `organization` | `organization`, `iam` | 6–9 días | **Alto** | 0 |
| 8 | Los catálogos preguntan en vez de recibir banderas | `vehiclecatalog`, `equipment`, `fleet` | 3–5 días, en dos entregas | Medio | 2; antes o después del monitoreo, nunca durante (§2) |
| 9 | Renombrar `location` a `reference` (opcional) | `reference` | 6–10 días | Bajo | 1 a 8; si se hace, antes del bloque 12 (decisión 16) |
| 10 | Preparar el bloque 12 | Diseño, sin código | 2–3 días | — | Respuestas del negocio |

Las fases 0 a 10 son entre 52 y 77 días de trabajo de una persona, sin calibrar. Van aparte el trabajo previo de D-260, de 6 a 9 días, que es trabajo funcional y no una refactorización, y el borrado de `legacyimport` (3b), de 1 a 2 días, que espera a la puesta en marcha: con los dos, entre 59 y 88 días. El orden va de lo barato y seguro a lo delicado, con una excepción deliberada: **equipamiento va segundo**, porque el próximo módulo anunciado al cliente es el monitoreo del GPS, y separarlo antes de construirlo cuesta mucho menos que después (cap. 14).

---

## 4. Fase por fase

### Fase 0 — Preparación

**Hecha el 19/11/2026** (backend y documentación en `refactor/phase-0`): las cuatro copias de las skills quedaron idénticas; la línea base de la suite del backend corrió en local, con las dos cuentas, sobre `develop` con D-261: 4138 pruebas, 0 fallos, 0 errores y 2 omitidas; `CurrencyUsageWiringTests` exige los cuatro implementadores; `procurement` dejó `iam::acl`. Están resueltas las decisiones 1, 3, 9 y 15; las demás se resuelven al llegar a su fase.

- **Sincronizar las cuatro copias de las skills**, que ya divergen: las de `.cursor` son versiones viejas de `backend-module`, `architecture-rules`, `frontend-feature`, `frontend-rules` y `architecture-review`, y `frontend-feature` de `.claude` es más reciente en el frontend que en el backend. La fuente es `.claude` del frontend para `frontend-feature` y `.claude` del backend para las demás; las otras tres copias se sobrescriben con ellas. Son copias locales: `/.claude/skills/` y `/.cursor/skills/` están en el `.gitignore` del backend (`7475cb64`) y del frontend (`8c84a054`). La rama de una fase no las lleva, el CI no las revisa y revertir un commit no las restaura (decisión 15).
- **Línea base:** la suite completa en verde con las dos cuentas de la base, en el backend y en el frontend.
- **La prueba de arranque de `CurrencyUsage`,** que exige en un solo lugar sus cuatro implementadores, como `BankAccountPaymentsWiringTests` exige el suyo (§5). Hoy solo hay cobertura parcial: `PurchaseInvoiceUnitPurchasesAdapterTests:119-120` exige `PurchaseInvoiceCurrencyUsageAdapter` en la lista, y `VehicleEquipmentCurrencyUsageTests` inyecta `VehicleEquipmentCurrencyUsageAdapter` por su clase. `PurchaseOrderCurrencyUsageAdapter` y `SupplierPaymentCurrencyUsageAdapter` no los exige ninguna prueba.
- **Quitar de `procurement` la dependencia `iam::acl`,** que declara y no usa: `git grep -n "erp\.iam\." -- src/main/java/ky/millennium/mobility/erp/procurement` no devuelve nada. Sus pruebas sí usan `iam`, pero Modulith no las analiza. En el mismo cambio se corrigen la fila `procurement` de la tabla de dependencias del mapa vigente (§4), la línea 369 del `README.md` del backend y §2 de `backend-module/SKILL.md`. Está terminado cuando `ModularityTests` pasa.
- **Aprobar** el mapa objetivo y este plan, y resolver con el responsable:
  1. **Los códigos de los permisos** no cambian en la migración (decisión 2 del mapa objetivo). **Resuelta el 19/11/2026:** no cambian.
  2. **Renombrar `location`** a `reference`: sí o no (decisión 4).
  3. **Los requisitos de la habilitación:** resuelta (decisión 5 del mapa objetivo, D-260). Qué equipo es requisito pasa a una configuración de `fleet`; la aplicabilidad y la pantalla de fábrica se quedan con el catálogo de equipos. D-260 se construye como trabajo funcional, en el trabajo previo de D-260 (antes de la fase 2).
  4. **La carga inicial** (decisión 9 del mapa objetivo): si se ejecuta antes de la fase 3, la fase 3 se reemplaza por borrar su código después de la puesta en marcha (3b en §3).
  5. **Fase 4, opción A o B:** quién calcula `registerFundAssignment`. A, recomendada: `fleet` lo pregunta a `financing` por su puerto y el recurso de la unidad no cambia. B: sale del recurso de la unidad y lo responde un subrecurso de `financing`.
  6. **Fase 4, opción A1 o A2:** quién dibuja la pestaña de cesiones de la ficha de la unidad. A1, recomendada: `fleet`, alimentada por `FinancingFacade`. A2: una pieza prestada de `financing`, con su excepción en la regla 1.2 del frontend.
  7. **Fase 5, opción A o B:** quién es dueño de la brecha de días. A, recomendada: sigue en `fleet` y el recurso de la unidad no cambia. B: pasa a `procurement`, nunca a `reporting`, y cambia el recurso de la unidad.
  8. **El destino de un dispositivo al retirarlo de una unidad**, para cuando el monitoreo una el inventario a la instalación (fase 2 §2.6): la propuesta es `falla` a mantenimiento, `robo` a baja y `reemplazo` a disponible. Se decide junto con la 11.
  9. **Las referencias a cuentas fuera de `iam`** (decisión 6 del mapa vigente): la opción B, que este plan da por hecha. **Resuelta el 19/11/2026 con la opción B**, cerrada en el mapa vigente §11. `fleet` conserva `iam::acl`, `financing` lo suma en la fase 4 y `procurement` lo pierde en esta fase, porque no lo usa. Se cierra en el mapa vigente §11. Si se elige la A, la fase 4 no da `iam::acl` a `financing`, y quitarlo de `fleet` es un cambio aparte.
  10. **Dónde se guarda un equipo del inventario y si todo equipo instalado exige su serial** (diccionario §14, abiertas). Las dos deciden la rama `refactor/remove-equipment-type-flags` y se resuelven antes del trabajo previo de D-260:
      - **V140** lleva `fleet_equipments.location_id` de las ubicaciones a las sedes. Si se conserva, se corrige:
        - lee la sede por `locations.branch_id`: hoy compara el id de una ubicación con el de una sede, y deja sin cambiar el equipo cuya ubicación tiene el mismo número que alguna sede, que pasa a apuntar a esa sede;
        - se detiene y nombra los equipos de una ubicación sin sede, en vez de llevarlos a la primera sede de la empresa. Con una empresa sin sede, el `COALESCE` da `NULL`, `location_id` es `NOT NULL` (V132) y la migración falla al arrancar;
        - su FK es compuesta, `(location_id, company_id)` a `branches (id, company_id)`, que ya tiene su `UNIQUE` (`branches_id_company_uq`, V26), y no la simple `REFERENCES branches (id)`.

        Sin respuesta, V140 sale de la rama y el inventario sigue en `locations`.
      - **`requires_serial`:** la rama lo retira de `equipment_types` (V138) y del código de los dos repositorios (`b9008e41` y `69f5e3d8`), y exige siempre el serial, salvo en la carga inicial. Sin respuesta, la rama lo conserva: se revierte de esos dos commits lo que lo retira.
  11. **Un dispositivo instalado en una unidad que se anula o pasa a Inactivo** (fase 2 §2.6): hoy `VehicleAnnulments` y `VehicleTransfers` no miran las instalaciones. Se decide junto con la 8, antes de construir el monitoreo.
  12. **`deactivation-question.ts`,** que hoy vive en `vehiclecatalog/presentation/components` y después de la fase 2 lo usan los dos catálogos: una copia en `equipment`, con `equipment-type`, `equipment-model` y `equipment-brand`, o su paso a `shared` (fase 2 §2.8).
  13. **La serialización de la fase 8:** la fila del catálogo `FOR UPDATE` antes de preguntar, que es la propuesta, o lecturas `ForShare` en los cuatro caminos que escriben (`VehicleCommandServiceImpl`, `PurchasedUnitRegistrations`, `MigratedUnitRegistrations` y `VehicleEquipmentInstallations`). Se elige una antes de la fase 8.
  14. **Las acciones de cuenta de cada fila de Colaboradores** (fase 7): se quedan en `organization`, calculadas por `EmployeeAllowedActions` con `ProtectedAccounts` y `PendingAccountActivations` de `shared`, o pasan a `iam` en una fase propia, que se suma a §3. Moverlas cambia el contrato de `allowedActions` de la lista de colaboradores.
  15. **Las skills:** versionarlas en git o seguir con copias locales. **Resuelta el 19/11/2026: nunca se versionan ni se suben**; solo se alinean en local, en las cuatro copias. Si se versionan, esta fase retira `/.claude/skills/` y `/.cursor/skills/` de los dos `.gitignore` y sube las cuatro copias sincronizadas, y desde ahí viajan con la rama de cada fase. Si siguen locales, cada fase las actualiza a mano en las cuatro copias el día que integra, y la vuelta atrás las restaura a mano (§6).
  16. **Cuándo se responde la 2:** antes de empezar el bloque 12. Si es sí, la fase 9 va antes del primer contexto nuevo, para que `customers`, `leasing` y `collections` nazcan con `reference::acl` y no sumen su `location::acl`. Si es no, el mapa objetivo vuelve a llamarlo `location` en todas sus secciones.

### Fase 1 — `audit` e `imports` salen de `shared`

**Hecha el 19/11/2026** (backend y documentación en `refactor/phase-1`), como la describe esta sección: `shared` ya no tiene `changelog` ni `importbatches`; `documents` sigue en `{}`; `ModularityTests` pasa con `audit::acl` e `imports::acl` como únicas puertas, la de `imports` repartida en sus tipos con `propagate`; y las pruebas que alcanzan los dos módulos, con las de arquitectura y el arranque del contexto, pasan: 334, sin fallos. La decisión nueva del diccionario es D-262 (v1.65). Ningún paso del plan tuvo que cambiar; se sumó `ImportBatchQueryService` a lo que usa `procurement`, que el plan no listaba. Tampoco listaba los documentos de `fleet`, `iam`, `location`, `organization` y `procurement`, procesos y pantallas ni las convenciones §2.2, §7.2 y §14.18, que todavía ponían la bitácora o las cargas en `shared`: se corrigieron en la misma rama, junto con la decisión 6 del mapa, que `fleet` e `iam` seguían dando por pendiente después de la fase 0.

**Por qué.** Un *Shared Kernel* debe ser pequeño y estable (cap. 14). La bitácora y el registro de las cargas tienen tabla, endpoint y permiso propios: son *Generic Subdomains* (cap. 15), no núcleo compartido. Salen de `shared` como ya salieron `documents` y `notifications` (D-125, D-143). El lector y el escritor de libros de `imports` son un *Cohesive Mechanism* (cap. 15) que viaja con su registro.

**Orden dentro de la fase.** Cada paso deja el build y las pruebas de arquitectura en verde, y todo se integra junto.
1. Se suman `"audit"` e `"imports"` a `PLATFORM_CONTRACTS` de `ForeignAclStaysInTranslationLayerTests`. No rompe nada: todavía no existen.
2. `audit`, con `iam`, `organization`, `location`, `fleet` y `procurement` declarando `audit::acl`. Mientras tanto `shared/importbatches` lo usa desde `shared`, que es abierto (`Type.OPEN`): sus dependencias no se restringen y Modulith lo deja fuera de la detección de ciclos.
3. `imports`, porque depende de `audit`.

**Backend — `audit`** (las 26 clases de `shared/changelog` pasan a `ky.millennium.mobility.erp.audit`, con los mismos subpaquetes):
- `package-info.java` con `@ApplicationModule(displayName = "Audit", allowedDependencies = {})` y `@NullMarked`: solo usa `shared`, que es abierto, como `documents`, que con `{}` lo usa en 16 clases.
- `audit/interfaces/acl`, con `@NamedInterface("acl")`, es su contrato público. Recibe los 8 tipos que usan los demás módulos:
  - `ChangeLogCommandService` (hoy en `application/commandservices`) y `RecordChangeLogCommand` (hoy en `domain/model/commands`), su parámetro, que nombran 15 archivos del código y 7 pruebas;
  - `ChangeLogHistoryLookup` (hoy en `application/queryservices`);
  - los puertos `AuditActorLookupPort` con `AuditActor`, `AuditCompanyLookupPort` y `ConfidentialChangeLogFieldsPort` con `ConfidentialChangeLogField` (hoy en `domain/ports` y `domain/model/valueobjects`). Salen hacia la fachada porque otros contextos los implementan o los leen.

  Ninguno lo usan `ChangeLogEntry` ni `ChangeLogRepository`: moverlos no hace que el dominio importe `interfaces`.
- Se quedan internos los demás. `ChangeLogCommandServiceImpl` sigue en `application/internal/commandservices`, envuelto por `DeadlockRetry`.
- `ChangeLogsController` se mueve con la misma ruta (`GET /api/v1/change-logs`), el mismo permiso (`PermissionCodes.ChangeLog.VIEW`, `bitacora.ver`) y su `@Tag`.
- **Consumidores que cambian el paquete.** Se reemplaza el nombre completo en todo el código, no solo las líneas `import`:
  - `procurement` (15):
    - en `application/internal/commandservices`: `CreditNoteAnnulments`, `CreditNoteCommandServiceImpl`, `OrderPaymentTransfers`, `PurchaseInvoiceAnnulments`, `PurchaseInvoiceCommandServiceImpl`, `PurchaseOrderTemplateChanges`, `SupplierCommandServiceImpl`, `SupplierPaymentAnnulments`, `SupplierPaymentRequests` y `SupplierPaymentVoucherReplacements`;
    - en `application/internal/queryservices`: `CreditNoteViewFactory`, `InvoicePaymentPositions`, `PurchaseInvoiceViewFactory` (incluido el nombre completo de `AuditActor` de la línea 320) y `SupplierPaymentViewFactory`;
    - en `infrastructure/changelog`: `ProcurementConfidentialChangeLogFieldsAdapter`.
  - `fleet` (6): `FleetImportCommandServiceImpl`, `MilestoneChangeLog`, `VehicleFundAssignmentCommandServiceImpl`, `VehicleIdentifierCorrections`, `VehicleTransfers` y `FleetConfidentialChangeLogFieldsAdapter`.
  - `location` (2): `ExchangeRateCommandServiceImpl`, que escribe la corrección de las tasas de cambio, y `ExchangeRateViewFactory`, que lee quién corrigió por `AuditActorLookupPort`.
  - `iam` (1): `IamAuditActorLookupAdapter`, el adaptador de `AuditActorLookupPort`.
  - `organization` (1): `OrganizationAuditCompanyLookupAdapter`, el de `AuditCompanyLookupPort`.
  - `imports` (2): `ImportBatchCommandServiceImpl` (`AuditCompanyLookupPort`) e `ImportBatchQueryServiceImpl` (`AuditActorLookupPort` y `AuditCompanyLookupPort`). Son las dos clases de `shared` que nombran la bitácora, y por ellas `imports` depende de `audit::acl`.
- **Nota de diseño.** Los dos puertos de búsqueda —el nombre de una cuenta y la clave de una empresa— no son de la bitácora. Los implementan `iam` y `organization`, y los leen también `location`, `procurement` e `imports`. Se quedan en `audit::acl`, porque llevarlos a `shared` contradiría la regla 6.1 y el objetivo de esta fase, y el mapa vigente lo registra.

**Backend — `imports`** (las 43 clases de `shared/importbatches` pasan a `ky.millennium.mobility.erp.imports`, más el adaptador que hoy está en `documents`):
- `package-info.java` con `@ApplicationModule(displayName = "Imports", allowedDependencies = {"audit::acl", "documents::acl"})` y `@NullMarked`.
- **`ImportBatchFileStoreAdapter` sale de `documents`** (`documents/application/internal/storage`) y pasa a `imports/infrastructure/storage`, sobre `KeptFileStore` de `documents::acl`. `KeptFileStore` ya ofrece lo que pide el puerto: `put`, con su respuesta de «lo escribí primero»; `open`; y `discardRolledBack`.
  - El descarte tras un rollback sigue en `ImportBatchCommandServiceImpl.keep` y no cambia.
  - El paquete vacío de `documents` se borra, y `documents` sigue en `{}`: si el adaptador se quedara, `documents` tendría que depender de `imports`.
  - Se reescribe el Javadoc de `ImportBatchFileStore`, que habla de `shared` y del ciclo de D-189.
- **`imports::acl`.** Spring Modulith publica de un paquete anotado solo sus propias clases públicas, sin subpaquetes ni los tipos de sus firmas. Una declaración a nivel de tipo, en cambio, arrastra los tipos del módulo que aparecen en sus firmas públicas (`propagate`, Modulith 2.0). Se expone lo que usan `fleet` y `procurement`, 27 tipos:
  - **El registro de la carga:** `ImportBatchCommandService` con `RegisterImportBatchCommand` y `RegisteredImportBatch`; `ImportBatchQueryService` (de él se usan `handle(GetImportBatchByPublicIdQuery)` y `fetchReferencesByIds`) con `GetImportBatchByPublicIdQuery`, `ImportBatchView` e `ImportBatchReference`; `ImportOutcome`; `ImportLocks`; `ImportType` e `ImportCounts`.
  - **Lo interno que hoy usan los consumidores:** `ImportBatchErrors.conflict()` e `ImportBatchAccessPolicy.reads(…)`. Se publican en `imports/interfaces/acl` como `ImportRefusals.conflict()` e `ImportReading.reads(AccessScope, Long)`, que delegan en los internos; los consumidores dejan de nombrar `application.internal`.
  - **El libro:** el paquete `application/spreadsheets` entero, con sus 11 tipos: `ImportWorkbookReader`, `ImportWorkbook`, `ImportRow`, `ImportCell`, `ImportCellException`, `ImportSheetSpec`, `ImportReportWriter`, `ImportReport` (con `ExtraSheet`), `ImportRowFailure`, `ImportSummaries` e `ImportFailureMessages`. Su `package-info.java` lleva `@NamedInterface("acl")`.
  - **La respuesta de una carga:** `ImportResultResource`, `ImportBatchResource` e `ImportBatchResourceFromViewAssembler`, que usan `FleetImportsController` e `InitialMigrationImportsController`. Siguen siendo una sola clase cada una, para que el esquema OpenAPI `ImportResult` no se duplique.
  - **El mecanismo.** `ImportType` e `ImportCounts` los usa el dominio (`ImportBatch`, `ImportBatchRepository`), y `RegisterImportBatchCommand` y `GetImportBatchByPublicIdQuery` son tipos del dominio. Ninguno se muda a `interfaces` ni lleva anotación: el dominio no importa Spring (regla 2.1).
    - Llevan `@NamedInterface("acl")` a nivel de tipo, sin cambiar de paquete, solo las clases de `application` y de `interfaces/rest` que nombran los consumidores: `ImportBatchCommandService`, `ImportBatchQueryService`, `ImportOutcome`, `ImportLocks`, `ImportResultResource` e `ImportBatchResourceFromViewAssembler`.
    - Por sus firmas entran los cuatro del dominio, `RegisteredImportBatch`, `ImportBatchView`, `ImportBatchReference` e `ImportBatchResource`. Entran también `SearchImportBatchesQuery`, `GetImportBatchFileQuery` (con su `Kind`) e `ImportBatchFile`, que ningún consumidor usa.
    - `ImportRefusals` e `ImportReading` están en `imports/interfaces/acl`, cuyo `package-info.java` lleva `@NamedInterface("acl")`.
    - Modulith junta todas las declaraciones `acl`, y `ModularityTests` lo comprueba.
- `ImportBatchesController` se mueve con sus cuatro rutas (`GET /api/v1/imports`, `/{id}`, `/{id}/error-report` y `/{id}/file`), su permiso (`PermissionCodes.Enablement.VIEW_IMPORTS`, `habilitacion.ver_cargas`) y su `@Tag`.
- **Consumidores:**
  - `fleet` (5): `FleetImportCommandService`, `FleetImportCommandServiceImpl`, `VehicleFundAssignmentViewFactory`, `VehicleTaxDeclarationQueryServiceImpl` y `FleetImportsController`;
  - `procurement` (5, todos de la carga inicial, que sale en la fase 3): `InitialMigrationImportCommandService`, `InitialMigrationImportCommandServiceImpl`, `InitialMigrationRun`, `StockRowMapper` e `InitialMigrationImportsController`.

**`allowedDependencies` después de la fase:**

| Módulo | Agrega |
|---|---|
| `audit` | Nuevo: `{}` |
| `imports` | Nuevo: `{"audit::acl", "documents::acl"}` |
| `fleet` | `audit::acl`, `imports::acl` |
| `procurement` | `audit::acl`, `imports::acl` (`imports::acl` sale en la fase 3) |
| `location` | `audit::acl` |
| `iam`, `organization` | `audit::acl` (implementan sus puertos) |
| `documents` | Queda en `{}` |

No hay ciclos: `audit` no depende de nadie, e `imports` solo de `audit` y `documents`.

**Pruebas.**
- **Se mueven**, espejando los paquetes:
  - las 8 de la bitácora: `ChangeLogCommandServiceTests`, `ChangeLogAccessPolicyTests`, `ChangeLogHistoryLookupTests`, `ChangeLogEntryTests`, `ChangeLogRowLevelSecurityTests`, `ChangeLogsControllerTenantTests`, `ChangeLogsPolicyTests` y `ChangeLogsSearchFilteringTests`. `ChangeLogCommandServiceTests` prueba `ChangeLogCommandServiceImpl`: va a `audit.application.internal.commandservices`, junto a la implementación, y no a `interfaces/acl`, adonde pasa la interfaz;
  - las 4 de las cargas: `FileLoadsMigrationTests`, `ImportCellTests`, `ImportReportWriterTests` e `ImportWorkbookReaderTests`;
  - el fixture `XlsxFixtures`, que pasa a `ky.millennium.mobility.erp.imports.XlsxFixtures`. El recurso `/imports/stock-template-headers.xlsx` no cambia.
- **Cambian el paquete, sin moverse:**
  - de la bitácora: `ExchangeRateQueryServiceImplTests` (`location`); `ConceptLinesTests`, `CreditNoteAnnulmentsTests`, `PurchaseInvoiceAnnulmentsTests` y `ReactivationCompanyTests`, con su nombre completo de la línea 91 (`procurement`);
  - de las cargas: `FleetImportsIntegrationTests` (`fleet`); `StockRowMapperTests`, `InitialMigrationIntegrationTests` y `StockWorkbook` (`procurement`).
- **`ForeignAclStaysInTranslationLayerTests`:** `PLATFORM_CONTRACTS = Set.of("documents", "notifications", "audit", "imports")`, y su Javadoc. Sin ese cambio fallan todas las clases consumidoras, y también `IamAuditActorLookupAdapter`, que nombra `AuditActor`, un tipo que no implementa.
- **Se corren sin cambios:**
  - `ModularityTests`;
  - `LayerDependencyTests`, que ya recorre todo `application` y sigue pasando: ninguna clase de `application` de los dos módulos importa `infrastructure` ni JPA. Su patrón (`erp.\w+.infrastructure.`) alcanza ahora también la `infrastructure` propia de `audit` e `imports`, que bajo `shared` tenía un segmento de más;
  - `DeadlockRetryTests`, porque el pointcut nombra el paquete relativo (`..application.internal.commandservices.*CommandServiceImpl`), no el módulo;
  - `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`, `JdbcDataVersionsTests` y `NoDeleteEndpointsTests`;
  - `RowLevelSecurityMigrationTests` y `HandOverDatabaseScriptTests`, que solo nombran tablas;
  - `OwnErrorCodesHaveMessagesTests` y `ErpMillenniumMobilityBackendApplicationTests`.
- **No cambian** Flyway (`V115__create_shared_import_batches_schema.sql` conserva su nombre), `hand-over-database.sql`, `application*.yaml`, los `messages*.properties` ni la configuración de OpenAPI: ninguno nombra los paquetes.

**Frontend.** Nada: `change-log` e `imports` ya son carpetas propias, y ninguna ruta cambia. La carpeta sigue siendo `change-log`, la de una pantalla de la plataforma (mapa vigente §8), aunque el módulo del backend se llame `audit`.

**Documentación.**
- Una decisión nueva del diccionario que precisa D-129 y D-200: la bitácora es el módulo `audit` y el registro de las cargas, `imports`, no `shared`; `ImportBatchFileStore` lo implementa `imports` sobre `KeptFileStore`.
- El mapa vigente:
  - `audit` e `imports` como módulos, con la arista `imports → audit::acl`;
  - §5.3: los puertos de la bitácora pasan a `audit::acl`, y `ImportBatchFileStore` lo implementa `imports`, no `documents`;
  - §6.5: la fila «Plataforma» ya no lleva la bitácora ni las cargas.
- El diccionario:
  - §3;
  - §11.8: el título pasa a «Reglas de `imports` sobre las cargas», y «por la fachada de `shared`» pasa a «por `imports::acl`». Las reglas SHR-R01 a SHR-R13 se quedan ahí con su número, aunque ahora sean de `imports`;
  - §13: lo confidencial «lo marca el contexto dueño, no `audit`».
- `contextos/README.md`: la fila de cada módulo nuevo y la línea 55 (`SHR-` para `imports`).
- `contextos/audit.md` y `contextos/imports.md`, con el patrón de `contextos/notifications.md`. `imports.md` enlaza las reglas SHR del diccionario §11.8 sin copiarlas.
- Las convenciones §2.2: la fila «Plataforma (`shared`)» pasa a «Plataforma (`audit`, `imports`)».
- El `README.md` del backend: el árbol de paquetes, líneas 333 a 339.
- Las skills, en sus cuatro copias (`.claude` y `.cursor` del backend y del frontend; decisión 15 de la fase 0):
  - `backend-module/SKILL.md`:
    - §1, «La bitácora de cambios» y «Las cargas y los reportes»;
    - §2, la lista de `allowedDependencies`, y después del párrafo de `notifications` un párrafo para `audit` (`allowedDependencies = {}`, su puerta `audit/interfaces/acl` con los ocho tipos) y otro para `imports` (`{"audit::acl", "documents::acl"}`, su puerta repartida y por qué).
  - `references/architecture-rules.md`:
    - §1.1: lo único importable de otro contexto es su `interfaces.acl`, salvo `imports`. Su puerta `imports::acl` reúne además los tipos marcados con `@NamedInterface("acl")` en `application/commandservices`, `application/queryservices`, `application/locks`, `application/spreadsheets` e `interfaces/rest`, y los que entran por sus firmas. Esa frontera la verifica `ModularityTests`, no el nombre del paquete. También las excepciones de `ForeignAclStaysInTranslationLayerTests`;
    - §1.2: «la escribe y la sirve `shared`»;
    - §6.1: la bitácora y las cargas ya no son de `shared`, como la plataforma de documentos.
  - `architecture-review/SKILL.md`, §1:
    - la búsqueda de apoyo (líneas 28 a 34) excluye también `ky.millennium.mobility.erp.imports.` cuando el tipo es de `imports::acl`;
    - la línea 36 nombra `audit` e `imports` junto a `documents` y `notifications`, como módulos de la plataforma;
    - la línea 40.

**Terminado cuando:**
- `shared` no tiene `changelog` ni `importbatches`;
- `documents` sigue sin dependencias;
- `ModularityTests` pasa con `audit::acl` e `imports::acl` como únicas puertas;
- la bitácora, las tres cargas y `/imports` responden igual.

### Antes de la fase 2 — D-260

**Qué y por qué aquí.** D-260 (diccionario §14; FLT-R148 a FLT-R151 de `contextos/fleet.md`) cambia lo que hace el sistema:
- qué equipo es requisito pasa a una configuración de `fleet`;
- en Disponible, perder un requisito lleva la unidad a En habilitación en la misma operación;
- dos marcas nuevas de los estados cuidan los equipos de seguridad de una unidad con cliente y congelan el equipamiento de una robada.

La fase 2 solo mueve: si los requisitos se construyen después, se mueven dos veces. D-260 es trabajo funcional y no una fase (§1, §3).

**Antes de empezar.**
- La decisión 10 está resuelta: decide V140 y `requires_serial` de la rama.
- Se construye sobre la rama `refactor/remove-equipment-type-flags` de los dos repositorios (backend `d6d23fe8`, frontend `37dbc8dd`), corregida como dice esta sección. Backend, frontend y documentación se integran juntos (§2). Hasta entonces rige D-184 como está.
- **Caracterizar:** guardar el estado de habilitación de todas las unidades de la base local (`GET /vehicle-enablements` exportado) y tener la suite completa en verde. La semilla deja todo igual: al terminar, el mismo estado para las mismas unidades.

#### La rama, corregida

- **Sus migraciones se renumeran** (abajo). No corrieron en ningún entorno: `dev` despliega solo `develop`. Una base local que ya corrió V138 a V140 de la rama se recrea.
- **V138 de la rama, que borra `blocks_availability` y `requires_serial`, se descarta.** La semilla de D-260 lee `blocks_availability`, así que las columnas salen en la entrega siguiente («En dos entregas», abajo); `requires_serial`, solo si la decisión 10 lo retira.
- **V140** se conserva corregida o sale, según la decisión 10. Si sale, sale también su código: `fetchBranch` y `fetchBranchesByIds` del `ExternalOrganizationService` de `fleet`; `fetchBranchLocationReferenceByPublicId`, `fetchBranchLocationReferencesByIds` y `GetBranchAggregateByPublicIdQuery` de `organization`; y en el frontend `OrganizationFacade.loadCompanyBranches`, `BranchesStore.findBranches` y su uso en el inventario de equipos.
- **`requires_serial`**, según la decisión 10. La rama no lo retira solo de V138, también del código: `VehicleEquipment.lacksSerial(serial)`, `VehicleEquipmentInstallations`, `EquipmentCommandServiceImpl`, `VehicleEnablementView.EquipmentRequirement` y su recurso, y en el frontend `vehicle-equipment-dialog`. Si se conserva, se revierte eso de `b9008e41` y `69f5e3d8`.
- **`EquipmentType.blocksEnablementAvailability()`**, que fija en el código el GPS y la pantalla (`PROGRAM_CODES`), se borra: lo decide la configuración de `fleet`.
- **`blocksAvailability` vuelve al recurso de la habilitación.** La rama lo quita de `VehicleEnablementResource.EquipmentRequirement` y del frontend (`vehicle-enablement.entity.ts`, `vehicle-enablements-response.ts`, `enablement-catalog-sample.ts`), y `enablement-task-list.ts` pasa a mostrar siempre la pantalla de fábrica. Con D-260 la marca sale de la configuración: se restaura, `enablement-task-list.ts` vuelve a `factoryFitted && blocksAvailability`, y se suma `securesUnit`.
- **Las claves que la rama deja huérfanas** se borran: `export.equipment-types.blocks-availability` de `messages*.properties` y `validation.equipment-type.blocks-availability.not-null` de `ValidationMessages*.properties`; las de `requires-serial`, con la decisión 10.
- **Lo que le falta a la rama:** `EquipmentBrandAuditTests` (regla 4.8); el e2e de las marcas, porque la rama no toca `e2e/` (los mocks de `/equipment-brands` y el diálogo «Gestionar marcas»); y la documentación de las marcas (Documentación, abajo).

#### Migraciones

Se numeran desde el siguiente número libre de `develop`: hoy V141, porque `develop` llega a V140 desde que integró D-261 el 19/11/2026 (V138 a V140, el cifrado de los datos personales de los colaboradores; V129 y V139 son migraciones Java, de `procurement` y de `organization`). Si `develop` integra otra migración antes, se corren todas. Con los números de hoy:

| Entrega | Migración | Qué |
|---|---|---|
| N | V141 | Las marcas de equipo (V139 de la rama), sin cambios |
| N | V142 | D-260, de `fleet` |
| N | V143 | La ubicación del inventario en las sedes (V140 de la rama), solo si la decisión 10 la conserva, con las correcciones que ella dice |
| N+1 | V144, o V143 sin la anterior | Borra `equipment_types.blocks_availability` y, si la decisión 10 retira `requires_serial`, esa columna, con `lock_timeout` |

Fuera de esta tabla, cuando este plan nombra V138, V139 o V140 habla de las migraciones de la rama `refactor/remove-equipment-type-flags` —V138 retira las banderas, V139 crea las marcas de equipo y V140 lleva el inventario a las sedes—, nunca de las de `develop`, que son las de D-261.

**La migración de D-260**, con el alcance `SYSTEM` y `lock_timeout`, como V96:
- **`vehicle_equipment_requirements`:** `id`, `public_id`, `equipment_type_id` (`NOT NULL`, `UNIQUE`, FK a `equipment_types`), `blocks_availability` y `secures_unit` (`BOOLEAN NOT NULL`), `created_at` y `updated_at`.
  - Un catálogo general, como `vehicle_document_types` (V97): sin `company_id`, sin Row Level Security y con fechas sin autor. Como `equipment_types`, no es por país.
  - La aplicación lee, inserta y actualiza, nunca borra: `REVOKE ALL` y `GRANT SELECT, INSERT, UPDATE` al rol de la aplicación, como V97.
  - **Un tipo sin fila no se exige ni es de seguridad.** El catálogo de equipos nunca escribe aquí: un tipo nuevo nace sin requisito hasta que el superadministrador lo marca.
- **La semilla:** una fila por cada tipo, activo o no, que tiene `equipment_types.blocks_availability` o es `GPS`. `blocks_availability` copia la columna y `secures_unit` es verdadero solo en `GPS`. Así se exige lo mismo que hoy y ninguna unidad cambia de estado de habilitación. Se detiene si falta `GPS`.
- **`data_versions`:** sus 64 contadores y sus dos disparadores, como V97 y V121 (`JdbcDataVersionsTests.everyVersionedTableHasItsSixtyFourSlotsAndItsTwoTriggers`).
- **Las marcas de los estados:** `vehicle_statuses` suma `requires_security_equipment` y `blocks_equipment_changes`, `BOOLEAN NOT NULL DEFAULT FALSE`. La semilla marca `CONTRATO_FIRMADO` y `ACTIVO` con la primera y `ROBADO` con la segunda; se detiene si falta alguno. `vehicle_statuses` ya está versionada (V53).
- **El camino `PIERDE_REQUISITO`** de `DISPONIBLE` a `HABILITACION` (V96) queda abierto: si falta se crea, como en V96, y si está cerrado se reabre. Se detiene si falta `DISPONIBLE` o `HABILITACION`, si `HABILITACION` está retirado o si el código está declarado en otro camino: juntarlos es una decisión, no una limpieza (como V130).

#### La configuración, en `fleet`

- **Las clases,** con el patrón de `VehicleDocumentType`: el agregado `VehicleEquipmentRequirement`; `VehicleEquipmentRequirementRepository` con su entidad, su ensamblador, su repositorio JPA y su adaptador; `SetVehicleEquipmentRequirementCommand`; `VehicleEquipmentRequirementCommandService(Impl)` y `VehicleEquipmentRequirementQueryService(Impl)` con su vista; y `VehicleEquipmentRequirementsController`, con sus recursos y sus ensambladores.
- **`GET /vehicle-equipment-requirements`,** con `catalogos.ver`: cada tipo de equipo, activo o no, con su código, su nombre, si está activo, sus dos marcas (falsas si no tiene fila) y el `updatedAt` de su fila. Los tipos los lee `ExternalVehicleCatalogService` por un miembro nuevo de `vehiclecatalog::acl`, `fetchEquipmentTypeReferences()`, que da todos, activos o no: hoy `fetchEquipmentTypeReferencesByIds` solo los da por id. Así el frontend no necesita la fachada de otro contexto.
- **`PUT /vehicle-equipment-requirements/{equipmentTypeId}`,** con `catalogos.editar_tipos_documento`: crea o corrige la fila del tipo, con el `updatedAt` leído (nulo si no tenía fila). Una edición vieja, o dos altas a la vez que detiene el `UNIQUE`, responden `409 VEHICLEEQUIPMENTREQUIREMENT_OUTDATED`; un tipo que no existe, `404 EQUIPMENTTYPE_NOT_FOUND`.
- **El permiso.** D-260 dice que las marcas las cambia el superadministrador. `catalogos.editar_tipos_documento` solo lo tiene `SUPERADMIN` (V95) y ya edita la otra mitad de los requisitos (`vehicle_document_types.blocks_availability`): no hace falta una migración de `iam`. Si el responsable pide un permiso propio, se siembra en la misma entrega, con su matriz (`PermissionsMigrationTests`, `PermissionCatalogConsistencyTests`) y su constante en los dos repositorios.
- **Marcar un equipo no mueve las unidades** que ya están en Disponible: la puerta se aplica al entrar (FLT-R91).
- **La versión:** `JdbcFleetDataVersions.enablements()` (l.37-38) suma `vehicle_equipment_requirements`, porque la exportación de la habilitación muestra lo que se exige (`FleetDocumentKinds:67`).

#### Lo que se exige, en un solo lugar

- **`vehiclecatalog` deja de decidirlo.**
  - `ModelEnablementProfiles.profileOf` (l.58-73) ya no calcula `required`: da el equipo que aplica y el de fábrica (VCT-R28).
  - `ModelEnablementView` y el `ModelEnablementProfile` de `vehiclecatalog::acl` pierden `requiredEquipments`, y con ellos cambia `VehicleCatalogContextFacadeImpl.toProfile` (l.210).
  - `EquipmentTypePersistenceEntity` ya no mapea `blocks_availability` (la rama).
- **`fleet` lo cruza en una sola clase, `ModelRequirements`** (`fleet/application/internal/queryservices`). Lee por lote el perfil del catálogo —`runsOnGas` y el equipo que aplica, con `factoryFitted`— y la configuración, una vez por clase, después del candado de la unidad cuando lo hay (FLT-R84). Devuelve el `ModelEnablementProfile` de `fleet` con:
  - `requiredEquipments`: aplica, su fila tiene `blocks_availability` y no viene de fábrica, que es la regla de hoy con la marca de `fleet`;
  - `securityEquipments`: aplica y su fila tiene `secures_unit`;
  - las dos marcas de cada equipo que aplica, que muestra la habilitación.
- **Lo leen,** en lugar de `ExternalVehicleCatalogService.fetchEnablementProfiles`: `RequirementHolds:47`, `VehicleEquipmentInstallations:74`, `VehicleEnablementFactsReader:63`, `VehicleEnablementList:82,103`, `VehicleEquipmentViewFactory:102` y `MigratedUnitRegistrations:283` (la pantalla, que instala solo si el modelo la exige, l.407, y el estado de destino, l.480). `UnitEnablement`, `VehicleEnablementViewFactory` y `EnablementPolicy` reciben el perfil como hoy.
- **`ExternalVehicleCatalogService.fetchEnablementProfiles`** devuelve un record de traducción propio, con `runsOnGas` y el equipo que aplica, y ya no el `ModelEnablementProfile` de `fleet`. Solo lo llaman `ModelRequirements` y `VehicleDocumentCommandServiceImpl:121`, que lee solo `runsOnGas`. Así la fase 2 cambia la fuente solo dentro de `ModelRequirements`.
- **`EquipmentType.PROGRAM_CODES` se queda:** el código de `GPS` y de `PANTALLA` lo siguen leyendo la pantalla de fábrica (VCT-R28), `requiresTelematics`, la carga inicial (`MigratedUnitRegistrations:77`) y `JdbcReportsReadRepository:151-170`. Su Javadoc (l.27, l.42, l.45 y l.151 de la rama), que dice que lo leen las reglas de la habilitación, se corrige.

#### Las marcas de los estados

- **Las clases.** `VehicleStatus` suma `requiresSecurityEquipment` y `blocksEquipmentChanges`, que se marcan al editar, como `requiresEnablement` (el alta no lo lleva): `VehicleStatus.update`, `UpdateVehicleStatusCommand`, `UpdateVehicleStatusResource`, `VehicleStatusResource`, `VehicleStatusView`, `VehicleStatusViewFactory`, `VehicleStatusResourceAssembler`, `VehicleStatusPersistenceEntity` y `VehicleStatusPersistenceAssembler`.
- **Sus reglas de catálogo,** las de `requires_enablement` (D-170), para que solo se entre a mano a un estado con `requires_security_equipment`, que es lo que D-260 revisa:
  - `VehicleStatusCommandServiceImpl.handle(UpdateVehicleStatusCommand)` (l.77-91) no la pone en el estado inicial, al que se registra sin revisión (`VEHICLESTATUS_SECURITY_INITIAL`), ni en un estado al que un camino mueve sola la unidad (`VEHICLESTATUS_SECURITY_AUTO_TARGET`);
  - `StatusChangeReasonCommandServiceImpl` no guarda un camino que mueve solo hacia un estado marcado (`STATUSCHANGEREASON_AUTO_TARGET_SECURED`), como `STATUSCHANGEREASON_AUTO_TARGET_GATED`. Hoy el único camino que mueve solo va a Disponible (V47).
- **La entrada a mano.** `VehicleMovements.recordMove` (l.180-194) lee los hechos de la unidad también cuando el destino tiene `requires_security_equipment`. `EntryGate` suma los equipos de seguridad que faltan, y `Vehicle.transitionTo` (l.312) los rechaza después de la puerta de la habilitación, con `SecurityEquipmentMissingException`: `422 VEHICLE_SECURITY_EQUIPMENT_MISSING`, con el estado y los que faltan (código y nombre). Lo piden, entre otros, «Mantenimiento finalizado», «Reparación finalizada» y «Cliente regulariza», que devuelven la unidad a Activo.
- **Los caminos que se ofrecen.** `VehicleAllowedActions` (l.122-139) pone en `held` los caminos hacia un estado marcado mientras a la unidad le falta un equipo de seguridad, como hace con la puerta.
- **La carga inicial no es una entrada a mano:** `MigratedUnitRegistrations` sigue registrando en Activo con lo que le falte (FLT-R123).

#### El retiro, los hitos y el paso automático

- **`RequirementHoldPolicy` responde un veredicto,** con el estado actual (su código y sus tres marcas), lo que se hace (instalar, reemplazar, retirar sin reemplazo o vaciar un hito) y si lo tocado es un requisito o un equipo de seguridad de la unidad. En este orden:
  1. `BLOCKED`: el estado tiene `blocks_equipment_changes` y se instala, reemplaza o retira un equipo → `422 VEHICLE_EQUIPMENT_CHANGES_BLOCKED_BY_STATUS`;
  2. `HELD`: el estado tiene `requires_enablement`, no es Disponible y se quita un requisito → `422 VEHICLE_ENABLEMENT_HELD_BY_STATUS`, como hoy en Contrato firmado (D-184);
  3. `SECURITY_HELD`: el estado tiene `requires_security_equipment` y se retira sin reemplazo un equipo de seguridad → `422 VEHICLE_SECURITY_EQUIPMENT_HELD_BY_STATUS`;
  4. `LOSES_REQUIREMENT`: el estado es Disponible (`VehicleStatus.AVAILABLE_CODE`) y se quita un requisito → se escribe, y la unidad pasa a En habilitación;
  5. `ALLOWED`.
- **`RequirementHolds` lo aplica** después del candado de la unidad, con lo que lee de `ModelRequirements`: `equipmentRemoval` y `milestoneClearing`, como hoy, y un `equipmentChange` nuevo para instalar y reemplazar. Lo llaman `VehicleEquipmentCommandServiceImpl` (instalar, l.72; reemplazar, l.90; retirar, l.136) y `VehicleEnablementCommandServiceImpl` (el ingreso, l.100; las placas físicas, l.150). Corregir el costo y completar el serial no lo llaman (FLT-R151).
- **El paso por «Pierde un requisito».** `PIERDE_REQUISITO` no está hoy en el código Java: es un dato de V96. Se suma `StatusChangeReason.REQUIREMENT_LOSS_CODE`, como `AUTOMATIC_RETURN_CODE` y `INITIAL_MIGRATION_CODE`. Con `LOSES_REQUIREMENT`, el comando llama a `VehicleMovements.recordRequirementLoss(vehicle, requisito, changedBy)`, en su transacción y bajo el candado de la unidad que ya tiene: ningún candado nuevo, ningún orden nuevo.
  - Antes de escribir, lee el camino `PIERDE_REQUISITO` de Disponible a En habilitación (`StatusChangeReasonRepository.findByPathAndCode`) y el estado En habilitación. Si alguno no está activo, responde `422 VEHICLE_REQUIREMENT_LOSS_PATH_MISSING` y el comando no escribe nada.
  - Después del retiro o del hito, mueve la unidad con `Vehicle.transitionTo` y la misma puerta de entrada que `recordMove`, sin pedir el permiso del camino: lo autoriza el permiso del retiro (`habilitacion.gestionar_equipos`) o el de los hitos (`habilitacion.registrar_hitos`), no `stock.habilitar`.
  - Escribe la fila del historial con `write` (l.280): firmada por la cuenta de la sesión, sin vencimiento, porque el camino no tiene tolerancia, y con el requisito perdido en `notes`: el código del tipo de equipo, `platesReceivedDate` o `arrivalDate`, los mismos del contexto de `VEHICLE_ENABLEMENT_HELD_BY_STATUS`.
  - La unidad no vuelve sola: la devuelve una persona por «Habilitación completa», a través de la puerta. Una persona puede seguir tomando `PIERDE_REQUISITO` desde Disponible.
- **El código queda fijo y su camino no se cierra:**
  - `StatusChangeReasonCommandServiceImpl` no cierra ese camino (`422 STATUSCHANGEREASON_REQUIREMENT_LOSS_LOCKED`) y no deja declarar el código en otro (`STATUSCHANGEREASON_CODE_RESERVED`, como `MIGRACION_INICIAL`, l.72-75; su mensaje pasa a nombrar el código);
  - `VehicleStatusCommandServiceImpl.handle(ChangeVehicleStatusStateCommand)` (l.120-142) no retira En habilitación mientras ese camino llegue a él (`422 VEHICLESTATUS_REQUIREMENT_LOSS_RETIREMENT_NOT_ALLOWED`). Hoy lo protege solo mientras es el estado inicial (`VEHICLESTATUS_INITIAL_RETIREMENT_NOT_ALLOWED`), y el estado inicial se cambia desde el catálogo.
- **Lo que se ofrece:**
  - `VehicleEnablementAllowedActions.of` no ofrece `installEquipment` en un estado con `blocks_equipment_changes`;
  - `VehicleEnablementAllowedActions.ofEquipment` (l.74) recibe el veredicto en lugar de `held`: `replace` no se ofrece con `BLOCKED`; `remove`, ni con `BLOCKED`, ni con `HELD`, ni con `SECURITY_HELD`; `completeSerial` y `correctCost` se siguen ofreciendo. `VehicleEquipmentViewFactory.heldEquipmentTypesOf` (l.97-105) calcula el veredicto de cada tipo;
  - `VehicleEnablementViewFactory:70` arma `VehicleEnablementView.EquipmentRequirement` con `blocksAvailability` y `securesUnit` de la configuración.

#### API, errores y mensajes

- **Lo que no cambia de nombre:** `VehicleEnablementResource.EquipmentRequirement.blocksAvailability` se queda (sale de la configuración) y se suma `securesUnit`; `requiresSerial`, según la decisión 10. `VehicleStatusResource` y `UpdateVehicleStatusResource` suman `requiresSecurityEquipment` y `blocksEquipmentChanges`.
- **Lo que sale del catálogo** (la rama): `blocksAvailability` de `EquipmentTypeResource` y de sus recursos de alta y edición, ordenar `/equipment-types` por `blocksAvailability`, y la columna «Bloquea la disponibilidad» de su exportación; lo mismo de `requiresSerial`, según la decisión 10.
- **Los errores nuevos,** cada uno con su mensaje en `messages.properties` y `messages_es.properties` (`OwnErrorCodesHaveMessagesTests`), su `@ApiResponse` y su texto en `es.json`:
  - `422`: `VEHICLE_SECURITY_EQUIPMENT_HELD_BY_STATUS` (el estado y el equipo), `VEHICLE_SECURITY_EQUIPMENT_MISSING` (el estado y los que faltan), `VEHICLE_EQUIPMENT_CHANGES_BLOCKED_BY_STATUS` (el estado), `VEHICLE_REQUIREMENT_LOSS_PATH_MISSING`, `STATUSCHANGEREASON_REQUIREMENT_LOSS_LOCKED`, `STATUSCHANGEREASON_AUTO_TARGET_SECURED`, `VEHICLESTATUS_REQUIREMENT_LOSS_RETIREMENT_NOT_ALLOWED`, `VEHICLESTATUS_SECURITY_INITIAL` y `VEHICLESTATUS_SECURITY_AUTO_TARGET`;
  - `409`: `VEHICLEEQUIPMENTREQUIREMENT_OUTDATED`.
- **`VEHICLE_ENABLEMENT_HELD_BY_STATUS`** ya no se responde en Disponible: su texto de `es.json`, que manda a sacar la unidad por «Pierde un requisito», cambia.

#### Frontend

- **La pantalla de requisitos, en `fleet`,** con el patrón de Tipos de documento:
  - `vehicle-equipment-requirements.store.ts` con su spec, `vehicle-equipment-requirement.entity.ts`, su comando y su petición, `vehicle-equipment-requirements-api-endpoint.ts`, `vehicle-equipment-requirements-response.ts`, su ensamblador, los miembros de `FleetApi` y la clave `fleetVehicleEquipmentRequirementsEndpointPath` de `environment.ts` y `environment.development.ts`;
  - `views/vehicle-equipment-requirements-view/` (4 archivos, con su spec), en la ruta `/fleet/equipment-requirements` con `catalogos.ver` (`fleet.routes.ts` y su spec), en Catálogos › Flota junto a Tipos de documento (`app-navigation.ts`), con sus claves de `es.json`. Edita con `EDIT_DOCUMENT_TYPES` de `permission-codes.ts`;
  - lee los tipos de su propia respuesta: no usa `VehicleCatalogFacade`.
- **Los estados:** `vehicle-status.entity.ts`, `vehicle-statuses-response.ts`, `vehicle-statuses-api-endpoint.ts`, `update-vehicle-status.command.ts`, `update-vehicle-status.request.ts`, `vehicle-status-assembler.ts`, `vehicle-status-form-dialog` (dos casillas, con sus negativas), `vehicle-statuses-view.html` (sus marcas) y `status-change-reason-form-dialog.ts` (la negativa de un camino que mueve solo hacia un estado con equipos de seguridad).
- **La habilitación:**
  - `vehicle-enablement.store.ts` (`HELD_BY_STATUS`, l.21-25; los códigos de los hitos, l.157) y `vehicle-equipments.store.ts` (l.110 y l.136): los errores nuevos, y después de un retiro o de vaciar un hito vuelven a leer la unidad, que pudo pasar a En habilitación;
  - `enablement-texts.ts` y sus claves de `es.json`: en Disponible ya no hay negativa (`held-equipment-available`, `held-milestone-available` y `held-available` salen o cambian), y se suman los textos de los equipos de seguridad y de Robado;
  - `remove-vehicle-equipment-dialog`, `register-arrival-dialog` y `vehicle-milestones-form`: en Disponible, el aviso de que la unidad pasa a En habilitación;
  - `change-vehicle-status-dialog`: la negativa que nombra el equipo de seguridad que falta, y el texto de «Pierde un requisito» (`loses-requirement-hint`), que hoy dice que el camino sirve para retirar un requisito;
  - `vehicle-status-history`: en una fila de `PIERDE_REQUISITO`, el requisito de `notes` por su nombre (los hitos, con `HELD_MILESTONES` de `enablement-texts.ts`);
  - `vehicle-enablement.entity.ts`, `vehicle-enablements-response.ts`, `enablement-task-list.ts` y `enablement-catalog-sample.ts`: `blocksAvailability` restaurado y `securesUnit`. En Robado, `vehicle-enablement-view` no ofrece instalar, reemplazar ni retirar, porque el backend no lo ofrece.
- **Specs que cambian:** las de cada archivo de arriba que la tenga, entre ellas las de FLT-R85 y FLT-R93 (`vehicle-equipments.store.spec.ts`, `vehicle-enablement.store.spec.ts`, `remove-vehicle-equipment-dialog.spec.ts`, `enablement-texts.spec.ts`, `vehicle-enablement-view.spec.ts` y `change-vehicle-status-dialog.spec.ts`), y `enablement-task-list.spec.ts`, que la rama cambió al quitar `blocksAvailability`.

#### e2e y e2e-live

- **Cambia de resultado** `e2e/enablement.spec.ts:283-297` («holds the GPS of an available unit…»): en Disponible se ofrece «Retirar», con el aviso, y el retiro deja la unidad En habilitación. `e2e/fleet.spec.ts:713,763` no cambian: una persona sigue tomando «Pierde un requisito» desde Disponible.
- **Los mocks:**
  - `e2e/support/mocks/enablement.ts` calcula lo exigido con la configuración y no con `type.blocksAvailability` (l.184-221); en Disponible escribe el retiro y los hitos y mueve la unidad, en vez de responder `VEHICLE_ENABLEMENT_HELD_BY_STATUS` (l.903, l.1066, l.1102), que sigue en Contrato firmado; suma las negativas de seguridad y de Robado y `/vehicle-equipment-requirements`;
  - `e2e/support/mocks/fleet.ts` (l.471-527, las reglas de `requiresEnablement`) suma las dos marcas con sus reglas;
  - `e2e/support/api.ts:2127-2152` y `e2e/support/mocks/vehiclecatalog.ts:85-103`: los tipos de equipo pierden `blocksAvailability` (y `requiresSerial`, según la decisión 10); la configuración y las marcas de equipo tienen sus datos y sus respuestas.
- **Casos nuevos:** en Activo el GPS se reemplaza y no se retira; entrar a Activo sin GPS se rechaza nombrándolo; en Robado no se ofrece instalar, reemplazar ni retirar; la pantalla de requisitos; el diálogo «Gestionar marcas».
- **Las imágenes de referencia:** `equipment-types` (`e2e/screens.ts:532`) cambia con la rama, por el filtro de marca y «Gestionar marcas»: se regenera aquí, revisada, porque la fase 2 no regenera ninguna. La pantalla de requisitos entra a `e2e/screens.ts` con su imagen nueva, revisada.
- **e2e-live:**
  - `enablement.live.spec.ts`, la prueba 5 (l.400-425), pasa a «retira el GPS en Disponible, la unidad pasa a En habilitación y vuelve por «Habilitación completa» con otro»; su l.67 (`requiresSerial`), según la decisión 10;
  - `vehiclecatalog.live.spec.ts:597` deja de mandar `blocksAvailability` y `requiresSerial`, y l.626 instala con un serial si la decisión 10 retira `requires_serial`.

#### Pruebas del backend

- **Cambian de resultado:**
  - `EnablementConcurrencyTests:160-176`: si el paso a Disponible confirma primero, el retiro ya no recibe `VEHICLE_ENABLEMENT_HELD_BY_STATUS`: confirma y lleva la unidad a En habilitación. Ningún orden la deja Disponible sin GPS (`disponibleWithoutGps` sigue falso);
  - `EnablementGateIntegrationTests.aDisponibleUnitKeepsItsGpsUnlessReplaced` (l.332): el retiro deja la unidad En habilitación, con su fila del historial. `aSoldUnitKeepsItsPhysicalPlates`, en Contrato firmado, no cambia;
  - `RequirementHoldPolicyTests`: el veredicto.
- **Se ajustan:**
  - `EnablementFixtures:148-155`, que repite en SQL la regla, lee `vehicle_equipment_requirements` (la rama la dejó en los códigos `GPS` y `PANTALLA`);
  - `VehicleCatalogContextFacadeTests:180,207`: el catálogo prueba solo lo que aplica y la pantalla de fábrica; lo que se exige pasa a `ModelRequirementsTests`;
  - `FleetRowsAreNeverDeletedTests` (l.37-38) suma la tabla;
  - las que construyen `VehicleStatus` o llaman a `Vehicle.transitionTo`, por sus firmas (`VehicleStatusTests`, `VehicleTests`).
- **Nuevas:**
  - `VehicleEquipmentRequirementTests`, `VehicleEquipmentRequirementAuditTests` (regla 4.8) y `VehicleEquipmentRequirementsControllerTests` (los permisos, el `409` y un tipo sin fila);
  - `ModelRequirementsTests`: lo exigido, lo de seguridad y lo de fábrica, en lote y con una lectura por clase;
  - la migración: la semilla iguala `blocks_availability`, las marcas de la semilla y `PIERDE_REQUISITO` abierto;
  - `RequirementLossIntegrationTests`: en Disponible, retirar el GPS, vaciar las placas físicas y vaciar el ingreso dejan la unidad En habilitación, con una fila firmada por quien retira y el requisito en `notes`, con el permiso del retiro y sin `stock.habilitar`; sin el camino, no se escribe nada;
  - `SecurityEquipmentIntegrationTests`: en Contrato firmado y en Activo el GPS se reemplaza y no se retira; entrar a mano a Activo sin GPS se rechaza nombrándolo; la carga inicial no se revisa;
  - `EquipmentChangesBlockedTests`: en Robado no se instala, reemplaza ni retira, y se corrige el costo y se completa el serial;
  - en `VehicleStatusCatalogTests`: cerrar `PIERDE_REQUISITO`, declararlo en otro camino, retirar En habilitación, marcar el estado inicial y marcar el destino de un camino que mueve solo;
  - `VehicleEnablementExportTests`: la versión cambia con la configuración;
  - `VehicleAllowedActionsTests` y `VehicleEnablementAllowedActionsTests`: los caminos retenidos por un equipo de seguridad, y las acciones en Robado y en Disponible.
- **Se corren:** las de arquitectura, `JdbcDataVersionsTests`, `OwnErrorCodesHaveMessagesTests`, `RowLevelSecurityMigrationTests`, `HandOverDatabaseScriptTests` (la tabla sin `DELETE`), `InitialMigrationIntegrationTests` y toda la habilitación de `fleet`; `PermissionsMigrationTests` y `PermissionCatalogConsistencyTests` si hay permiso nuevo.

#### Documentación

- **El diccionario:**
  - la lista de tablas versionadas (§2) y una sección nueva de `vehicle_equipment_requirements` en el bloque 9 (§11);
  - §7.5: `blocks_availability` sale, `requires_serial` según la decisión 10, y las marcas de equipo (`equipment_brands`, `equipment_models.equipment_brand_id`);
  - §8.2 (las dos marcas y sus reglas), §8.3 (`PIERDE_REQUISITO` fijo; los caminos que mueven solos), §8.4 (la fila del paso automático), §11.4 (el retiro) y §11.9 (lo exigido sale de la configuración);
  - §14: D-184 y D-260 como construidas, y las dos preguntas abiertas de D-254 que cierra la decisión 10.
- **`contextos/fleet.md`:** el párrafo «Decidido y por construir» (l.354), FLT-R85 y FLT-R93 enmendadas, y FLT-R148 a FLT-R151 en ✅ con sus pruebas.
- **`contextos/vehiclecatalog.md`:** VCT-R28 (ya no calcula lo que se exige), la fila de `equipment_types` (l.19) y las marcas de equipo.
- **El mapa vigente:** la fachada de `vehiclecatalog` (§5.1), sin `requiredEquipments` y con `fetchEquipmentTypeReferences`. **El mapa objetivo:** la configuración de `fleet` (§3.1, l.59) deja de decir «por construir».
- **Los patrones de persistencia,** la tabla de `fleet`:
  - «Reemplazar, retirar…» y «El ingreso de una unidad y los hitos» leen después del candado las marcas del estado, la configuración y, en Disponible, el camino, y escriben el paso en la misma transacción;
  - «El cambio de estado (F3)» lee los equipos abiertos si el destino exige equipos de seguridad;
  - el primer punto de «Lo que resulta de ese orden» cambia: si el cambio confirma primero, el retiro lleva la unidad a En habilitación.
- **Las convenciones:** la fila `fleet` de §2.2 (l.50) con `/vehicle-equipment-requirements` y la de `vehiclecatalog` con `/equipment-brands`; los recursos; la tabla de errores.
- **`procesos-y-pantallas`:** quién mantiene la configuración (l.110), Catálogos › Flota (l.322), sus pantallas (l.381) y el retiro en Disponible en el proceso de la habilitación.

#### En dos entregas

1. **Entrega N:** todo lo anterior. `equipment_types` conserva `blocks_availability` y `requires_serial` con su `DEFAULT FALSE`, y el catálogo ya no las mapea: las altas no fallan, `ddl-auto: validate` no las exige y nada las lee.
2. **Entrega N+1, antes de empezar la fase 2.** Se aplica cuando N ya está desplegada en `dev` (App Platform, `instance_count: 1`), su CI está en verde y ninguna copia anterior a N se conecta a esa base, tampoco una local con el `.env` remoto. Es la migración que borra las columnas (la tabla de migraciones, arriba). Se ajustan las pruebas que todavía las nombran. Si se atrasa hasta después de empezar la fase 2, la tabla ya es de `equipment` y la fase 2 la nombra.
3. **Vuelta atrás:** D-260 se corrige hacia adelante. Revertir N deja de ser seguro en cuanto se integra: las marcas de equipo dejan `equipment_models.equipment_brand_id` en `NOT NULL`, que el código anterior no escribe, y se perdería lo cambiado en la configuración.

**Terminado cuando:**
- la caracterización da el mismo estado de habilitación para las mismas unidades;
- en Disponible, retirar sin reemplazo un requisito, vaciar las placas físicas o vaciar el ingreso deja la unidad En habilitación, con una fila del historial firmada por quien retira y que nombra el requisito;
- en Contrato firmado se sigue rechazando; en Activo el GPS se reemplaza y no se retira sin reemplazo; entrar a mano a Activo sin GPS se rechaza nombrándolo; en Robado no se instala, reemplaza ni retira;
- `PIERDE_REQUISITO` no se cierra ni se declara en otro camino, y En habilitación no se retira mientras ese camino llegue a él;
- nada lee ni escribe `equipment_types.blocks_availability`, y la entrega N+1 la borró;
- FLT-R148 a FLT-R151 están en ✅ con sus pruebas, y el CI de la rama está en verde;
- la fase 2 puede volver a tomar su inventario.

### Fase 2 — `equipment`

**Por qué.** Los dispositivos tienen lenguaje y actores propios (serie, IMEI, ICCID, taller, monitoreo) y van a crecer con el monitoreo. `Equipment#installOnVehicle` no tiene quién lo llame y `vehicle_equipments` no apunta a `fleet_equipments`: el inventario todavía no está unido a la instalación, así que es el momento más barato para separarlo (cap. 14). La instalación en la unidad se queda en `fleet`, porque la puerta de Disponible la lee en la misma transacción que el estado (cap. 6).

**Antes de empezar.** El trabajo previo de D-260 (antes de la fase 2) está integrado, con sus dos entregas. Así esta fase mueve un catálogo de equipos que ya no decide qué se exige, y los requisitos no se mueven dos veces.

**El inventario de la fase se vuelve a tomar.** Las listas, las cuentas y las líneas de esta fase son las de `develop` del 18/11. Al abrir la fase, con D-260 y la rama integradas, se toman otra vez con los mismos comandos (`git ls-tree … | grep -i equip`, y `git grep` de cada miembro de equipos de `VehicleCatalogContextFacade` y de `ExternalVehicleCatalogService`). La fase no empieza hasta que esta sección coincide con el código. Ya se sabe lo que suman:
- **las marcas de equipo de la rama:**
  - en el backend, las 24 clases `EquipmentBrand*` de `vehiclecatalog` (83 clases `Equipment*` en lugar de 59), con `EquipmentBrandsController` (`/equipment-brands`); `equipment_brands` y `equipment_models.equipment_brand_id`; `EQUIPMENT_BRAND_NAME` de `NaturalKeyTakenException`; y sus pruebas, `EquipmentBrandTests`, `EquipmentBrandCommandServiceImplTests`, `EquipmentBrandsControllerTests` y `EquipmentBrandAuditTests`;
  - en el frontend, 16 archivos de `vehiclecatalog` (`equipment-brand.entity.ts`, 3 comandos, 3 peticiones, `equipment-brands-api-endpoint.ts`, `equipment-brands-response.ts`, `equipment-brand-assembler.ts` y `components/equipment-brands-dialog/` con 6, 2 de ellos specs); `EquipmentBrandOption`, `equipmentBrandOptions`, `equipmentBrandsLoading`, `equipmentBrandsError` y `clearEquipmentBrandsError` de `VehicleCatalogFacade`, que usa el inventario; los métodos de marcas de `vehiclecatalog-api.ts`; `vehicleCatalogEquipmentBrandsEndpointPath` de `environment*.ts`; y `'equipment-brand'` en `deactivation-question.ts`;
- **`equipmentBrandInactive`** en `FleetErrors`: 21 fábricas del inventario;
- **si V140 se conserva** (decisión 10): `fetchBranch` y `fetchBranchesByIds` del `ExternalOrganizationService` de `fleet`, que solo usa el inventario, sobre `fetchBranchLocationReferenceByPublicId` y `fetchBranchLocationReferencesByIds` de `organization::acl`; y `OrganizationFacade.loadCompanyBranches` en el frontend;
- **de D-260:** `fetchEquipmentTypeReferences()` de `vehiclecatalog::acl`, que lista todos los tipos para la pantalla de requisitos; y `ModelRequirements`, el único lector en `fleet` del perfil del catálogo para equipos, en lugar de los seis de hoy (§2.3).

#### 2.1 Backend — el módulo

- **`equipment/package-info.java`:** `@ApplicationModule(displayName = "Equipment", allowedDependencies = {"shared", "organization::acl", "vehiclecatalog::acl", "documents::acl"})`, con `@NullMarked`. `organization::acl` da la empresa y la ubicación de cada equipo; `vehiclecatalog::acl`, los tipos de vehículo y lo que la habilitación necesita de cada modelo. `equipment/interfaces/acl/package-info.java` lleva `@NamedInterface("acl")`.
- **Desde `fleet`, el inventario** (30 archivos, al mismo paquete relativo):
  - **dominio:** `Equipment`, `EquipmentStatus`, `CreateEquipmentCommand`, `UpdateEquipmentCommand`, `ChangeEquipmentStatusCommand`, `GetEquipmentByPublicIdQuery`, `SearchEquipmentsQuery`, `EquipmentRepository` y `EquipmentIdentifierTakenException`;
  - **aplicación:** `EquipmentCommandService`, `EquipmentCommandServiceImpl`, `EquipmentQueryService`, `EquipmentView`, `EquipmentQueryServiceImpl`, `EquipmentViewFactory` y `ExternalSupplierService`;
  - **infraestructura:** `EquipmentPersistenceEntity`, `EquipmentPersistenceAssembler`, `EquipmentJpaRepository` y `EquipmentRepositoryImpl`;
  - **interfaces:** `EquipmentsController` (`/equipments`, con `stock.ver` para leer y `stock.crear_equipamiento`, `stock.editar_equipamiento` y `stock.cambiar_estado_equipamiento` para escribir); `EquipmentResource`, `CreateEquipmentResource`, `UpdateEquipmentResource` y `ChangeEquipmentStatusResource`; sus cuatro ensambladores; y el puerto `EquipmentSuppliers`, que pasa a `equipment/interfaces/acl`.
  - **Los errores:** las 21 fábricas del inventario en `FleetErrors` (`equipment*` con `equipmentBrandInactive`, `supplierNotFound`, `supplierInactive` y `deniedOnEquipment`) pasan a `EquipmentErrors`, con los mismos códigos. `companyNotFound` y `locationNotFound` se copian, porque `fleet` las sigue usando.
  - **La empresa y la ubicación de un equipo:** `equipment` escribe su propio `ExternalOrganizationService` —hoy el inventario usa `fetchCompany`, `fetchCompaniesByIds`, `fetchLocation` y `fetchLocationsByIds` del de `fleet`— y su record `EquipmentLocation`, en lugar de `UnitLocation` de `fleet` (`EquipmentCommandServiceImpl:65,183,260`). Si V140 se conserva, lee la sede por lote y por identificador público, y `fetchBranch` y `fetchBranchesByIds` salen del de `fleet`.
- **Se quedan en `fleet`:**
  - `VehicleEquipment` con sus servicios, controlador, recursos y errores; `EquipmentPeriod`, `EquipmentCost`, `EquipmentRemovalReason` y `VehicleEquipmentOverlapException`; `VehicleEquipmentCurrencyUsageAdapter`; y su traducción `EquipmentTypeReference` y `ModelEnablementProfile`;
  - `VehicleEquipmentRequirement` con sus clases, `vehicle_equipment_requirements` y `ModelRequirements`: qué se exige es de la habilitación (D-260).
  - `EquipmentModelReference` de `fleet` se borra: solo la usaba el inventario.
- **Desde `vehiclecatalog`, el catálogo de equipos:**
  - las clases `Equipment*` (59 en `develop`, 83 con la rama), con las 24 de `EquipmentBrand` (`equipment_brands`), que pasa entera porque `EquipmentModel` la lee por su repositorio (`EquipmentModelCommandServiceImpl`, `EquipmentModelQueryServiceImpl` y `EquipmentModelViewFactory`). Los controladores `/equipment-types`, `/equipment-models` y `/equipment-brands` leen con `catalogos.ver` y escriben con `catalogos.crear_equipamiento`, `catalogos.editar_equipamiento` y `catalogos.desactivar_equipamiento`;
  - `VehicleTypeApplicability` (`domain/model/entities`);
  - `GetModelEnablementProfilesQuery`, que pasa a `GetEquipmentApplicabilityQuery`;
  - lo que quede de equipos en `ModelEnablementProfiles` y `ModelEnablementView` después de D-260, que pasa a `EquipmentApplicability` y `EquipmentApplicabilityView`: qué equipo admite cada modelo y cuál trae de fábrica, con la regla de la pantalla de fábrica (VCT-R28) escrita una sola vez. Qué se exige ya lo decide `fleet`;
  - una `NaturalKeyTakenException` propia de `equipment`, con `EQUIPMENT_TYPE_CODE`, `EQUIPMENT_MODEL_NAME` y `EQUIPMENT_BRAND_NAME`; la de `vehiclecatalog` pierde las tres.
- **Lo que el catálogo leía del interior de `vehiclecatalog`** (`VehicleTypeRepository` y `VehicleType` en el servicio de comandos, la fábrica de vistas, el servicio de consultas, la vista, el controlador y `EquipmentTypeResourceFromEntityAssembler`) pasa por un `ExternalVehicleCatalogService` de `equipment`. Los tipos de vehículo de una vista son un objeto de valor propio, `ApplicableVehicleType` (regla 1.1). `VEHICLETYPE_NOT_FOUND` sigue nombrando los identificadores que faltan, ordenados. La constante `EQUIPMENT_TYPES` de `VehicleCatalogDocumentKinds` (`EquipmentTypesController:127`) pasa a `EquipmentDocumentKinds`.
- **Dentro de `equipment`, el inventario lee sus tipos, marcas y modelos por sus propios repositorios**, nunca por su fachada. Los códigos de error (`EQUIPMENTTYPE_NOT_FOUND`, `EQUIPMENT_TYPE_INACTIVE`, …) no cambian.
- **La placa de la unidad que muestra un equipo** sale de un puerto nuevo, `EquipmentUnits` (en lote: la unidad por su id, con `publicId` y placa), que implementa `fleet` en `fleet/infrastructure/equipment/VehicleEquipmentUnitsAdapter`. Así `equipment` no depende de `fleet` y no hay ciclo. Su record va anidado en el puerto (`EquipmentUnits.Unit`), como `EquipmentSuppliers.EquipmentSupplierSummary`: `ForeignAclStaysInTranslationLayerTests` solo exime al tipo que el adaptador implementa. `EquipmentResource` conserva `currentVehicleId` y `currentVehiclePlate`.
- **Los beans de traducción llevan nombre**, como los de `fleet` y `procurement` (§5): `equipmentExternalOrganizationService`, `equipmentExternalVehicleCatalogService` y `equipmentExternalSupplierService`. Hoy `ExternalSupplierService` es `@Service` sin nombre, e `iam` ya tiene un `externalOrganizationService` por omisión.
- **Las exportaciones y sus versiones:**
  - `EquipmentDocumentKinds` recibe el bean `equipmentTypesDocumentKind`, con el mismo `kind` (`equipment-types`) y la misma clave (`export.equipment-types.name`); su versión es `dataVersions.equipmentTypes() + ";" + externalVehicleCatalogService.fetchDataVersion()`.
  - `JdbcEquipmentDataVersions` nombra solo `equipment_types` y `equipment_type_vehicle_types`, en `equipmentTypes()` y en `enablement()`. Nunca `vehicle_types` (regla 6.2).
  - En `vehiclecatalog`, `enablement()` queda en `vehicle_models` y `vehicle_types`, y `equipmentTypes()` sale.

#### 2.2 Backend — las fachadas

- **`vehiclecatalog::acl` suma**, porque `equipment` y `fleet` los necesitan:
  - `fetchVehicleTypeSummariesByPublicIds(Set<UUID>)`, `fetchVehicleTypeSummariesByIds(Set<Long>)` y el record `VehicleTypeSummary(id, publicId, code, name, isActive)`;
  - `fetchModelEnablementFactsByIds(Set<Long>)`, que devuelve `ModelEnablementFacts(modelId, vehicleTypeId, runsOnGas, hasFactoryScreen)` y responde `VehicleModelQueryService` con `GetModelEnablementFactsQuery`. Es un miembro nuevo, con otro nombre: `fetchEnablementProfilesByModelIds` y `ModelEnablementProfile` ya existen en `vehiclecatalog::acl` (`VehicleCatalogContextFacade:107`) y hoy los responde `EquipmentTypeQueryService.handle(GetModelEnablementProfilesQuery)`, que se muda.
- **`vehiclecatalog::acl` pierde** (paso 7): `fetchEquipmentTypeReferenceByPublicId`, `fetchEquipmentTypeReferencesByIds`, `fetchEquipmentTypeReferences` (D-260), `fetchEquipmentModelReferenceByPublicId`, `fetchEquipmentModelReferencesByIds`, `markEquipmentTypeInUse` y `fetchEnablementProfilesByModelIds`, con sus records `EquipmentTypeReference`, `EquipmentModelReference` y `ModelEnablementProfile`. `fetchEnablementDataVersion` queda solo en lo del modelo.
- **`EquipmentContextFacade`, solo con lo que `fleet` usa:**
  - `fetchEquipmentTypeReferenceByPublicId` (`VehicleEquipmentCommandServiceImpl:78`);
  - `fetchEquipmentTypeReferencesByIds` (`VehicleEquipmentCommandServiceImpl:103`, `VehicleEnablementList:125`, `VehicleEquipmentViewFactory:66`);
  - `fetchEquipmentTypeReferences()`: todos los tipos, activos o no, para la pantalla de requisitos de `fleet` (D-260);
  - `fetchEquipmentApplicabilityByModelIds(Set<Long>)`: el equipo que admite cada modelo, con `factoryFitted`, leído una vez por clase; qué se exige lo lee `fleet` de su propia configuración (D-260);
  - `markEquipmentTypeInUse` (hasta la fase 8);
  - `fetchEnablementDataVersion`.
  - Sin `fetchEquipmentModel*` ni marcas: no tienen consumidor.
- **Puertos que declara `equipment`:** `EquipmentSuppliers`, que implementa `procurement`, y `EquipmentUnits`, que implementa `fleet`.

#### 2.3 Backend — consumidores que cambian

- **`fleet`:**
  - `package-info.java` suma `"equipment::acl"`.
  - Un `ExternalEquipmentService` nuevo (`@Service("fleetExternalEquipmentService")`), que da a `ModelRequirements` lo que admite cada modelo según `equipment`, con `factoryFitted`; `runsOnGas` sigue saliendo de `vehiclecatalog`. Una lectura por contexto, nunca una por modelo. Los requisitos los cruza `ModelRequirements` con la configuración de `fleet` (D-260).
  - `ExternalVehicleCatalogService` pierde sus miembros de equipos (l.126-201 de hoy, con el `fetchEquipmentTypes` de D-260) y lee `runsOnGas` por `fetchModelEnablementFactsByIds`.
  - Cambian a `ExternalEquipmentService`:
    - quien pide el perfil para equipos: solo `ModelRequirements` (D-260). `VehicleDocumentCommandServiceImpl:121` lee solo si el modelo es a gas, así que sigue en `ExternalVehicleCatalogService`, con `runsOnGas`;
    - quien lee tipos: `VehicleEquipmentCommandServiceImpl:78,103`, `VehicleEnablementList:125`, `VehicleEquipmentViewFactory:66` y la consulta de la configuración de D-260, que lista todos los tipos;
    - quien marca el uso: `VehicleEquipmentInstallations:174` y `MigratedUnitRegistrations:204`;
    - la versión: `FleetDocumentKinds:67`, que compone la versión de la habilitación de los dos catálogos.
  - `UnitEnablement`, `VehicleEnablementViewFactory` y `EnablementPolicy` no cambian: reciben el `ModelEnablementProfile` de `ModelRequirements`.
  - `FleetErrors` pierde las 21 fábricas.
  - Si V140 se conserva, su `ExternalOrganizationService` pierde `fetchBranch` y `fetchBranchesByIds`.
  - Adaptador nuevo: `fleet/infrastructure/equipment/VehicleEquipmentUnitsAdapter`.
- **`procurement`:** `package-info.java` suma `"equipment::acl"`, y `SupplierEquipmentSuppliersAdapter` pasa a `procurement/infrastructure/equipment/`, implementando `equipment.interfaces.acl.EquipmentSuppliers`.
- **No cambian:** `reporting`, que lee `equipment_types` y `equipment_type_vehicle_types` por SQL (`JdbcReportsReadRepository.java:151-170`) con los mismos nombres; `hand-over-database.sql`; las políticas de RLS; los campos confidenciales de la bitácora (el costo es de `vehicle_equipments`, que se queda); el `CurrencyUsage` de la instalación; y los permisos (decisión 1).

#### 2.4 Lo que no cambia, explícitamente

- **Ninguna tabla, columna ni migración.**
- **Ningún candado:** las mismas sentencias en el mismo orden, solo por otra fachada.
- **Ninguna ruta, recurso ni código de error.**
- **El alcance:** el inventario se lee con `stock.ver` por empresa (`companyReach` y `actsOn`) y se escribe con `actsOnWholeCompanyEveryBrand`. `fleet_equipments` conserva su `company_id`, su RLS forzado y sus 3 políticas (V132); `equipment_types`, `equipment_brands` y `equipment_models` siguen sin RLS.

#### 2.5 Orden dentro de la fase

1. **Caracterizar.** Guardar el estado de la habilitación de todas las unidades de la base local (`GET /vehicle-enablements` exportado) y tener la suite completa en verde.
2. **El esqueleto de `equipment`:** sus dos `package-info.java` y la fachada vacía.
3. **Sumar** los miembros nuevos de `vehiclecatalog::acl`, sin quitar nada: tienen otro nombre que los de hoy.
4. **Mover el catálogo de equipos,** con sus marcas, sus versiones, su tipo de documento y su fachada.
5. **Mover el inventario,** con `EquipmentErrors`, `ExternalSupplierService`, `EquipmentSuppliers` y el puerto `EquipmentUnits`.
6. **Reapuntar `fleet` y `procurement`,** con sus adaptadores.
7. **Retirar de `vehiclecatalog`** sus miembros de equipos, `fetchEnablementProfilesByModelIds` y su `ModelEnablementProfile`, y partir `ModelEnablementProfiles`.
8. **Mover y partir las pruebas** (§2.7) y correr el paso 1 otra vez: el mismo estado para las mismas unidades.
9. **Documentación y skills.**

#### 2.6 Regla para cuando el monitoreo una el inventario a la instalación (se escribe ahora y se construye con el monitoreo)

- **Fuente de verdad.**
  - Qué dispositivo está en qué unidad, y desde cuándo, lo dice la instalación de `fleet`: el período abierto de `vehicle_equipments`, que suma `equipment_id` como FK a `fleet_equipments`. Queda nulo para lo migrado y lo de fábrica.
  - `fleet_equipments.status = INSTALLED` y `current_vehicle_id` son el estado actual del dispositivo, escrito en la misma transacción que la instalación. Es una excepción a la regla 4.1, como el estado actual de la unidad, y se registra como decisión.
- **Quién llama a quién.** Solo `fleet` instala, reemplaza y retira, con `habilitacion.gestionar_equipos` sobre la unidad. En la misma transacción llama a `EquipmentContextFacade.installDevice(dispositivo, unidad, empresa, tipo)` y `releaseDevice(dispositivo, destino, motivo)`, que devuelven `Result`. `equipment` escribe sus filas. Nunca escribe nadie en las tablas de otro contexto.
- **Orden de los candados.** La unidad (`vehicles`, `FOR NO KEY UPDATE`, `EnablementUnitLocks`) → la instalación (`vehicle_equipments`) → los dispositivos (`fleet_equipments`, `FOR NO KEY UPDATE`, en orden de id cuando un reemplazo toma dos) → la escritura de la instalación. `equipment` nunca bloquea una unidad ni una instalación: así no hay ciclo con las escrituras del inventario, que solo bloquean el dispositivo.
- **Invariantes.**
  - Un dispositivo está en una sola unidad: índice único parcial `vehicle_equipments (equipment_id) WHERE equipment_id IS NOT NULL AND upper_inf(period)`.
  - Está `INSTALLED` si y solo si tiene una instalación abierta.
  - Es de la empresa de la unidad y del tipo de la instalación: FK `(equipment_id, company_id, equipment_type_id)` a `fleet_equipments (id, company_id, equipment_type_id)`, con su `UNIQUE`. `current_vehicle_id` pasa a ser FK compuesta `(current_vehicle_id, company_id)` a `vehicles (id, company_id)`.
  - El serial de la instalación es el del dispositivo. Ya no se edita mientras el dispositivo no está `AVAILABLE` (`updateIdentifiers`).
- **El inventario no saca un dispositivo de `INSTALLED`.**
  - Se retira `INSTALLED → MAINTENANCE` de `Equipment.canTransitionTo` y de `moveToMaintenance`, y de las convenciones (§ de `PATCH /equipments/{id}/status`). Es un cambio del contrato de `PATCH /equipments/{id}/status`, que hoy nadie puede ejercer.
  - El inventario responde `422 EQUIPMENT_INSTALLED`. No se da de baja, no se manda a mantenimiento y no se corrige un dispositivo instalado.
- **Las reglas del retiro siguen en `fleet`** (D-260). Un dispositivo solo sale de una unidad por el retiro o el reemplazo de `fleet`, que antes aplica `RequirementHolds`: en Contrato firmado y en Activo un equipo de seguridad no se retira sin reemplazo, en Robado no se retira nada, y en Disponible el retiro lleva la unidad a En habilitación en la misma transacción. Así el inventario no puede invalidar en silencio lo que la puerta cuenta.
- **El destino al retirar** lo decide el responsable (decisión 8). Propuesta: `falla` → `MAINTENANCE`, `robo` → `DISCARDED`, `reemplazo` → `AVAILABLE`. Junto con ella se decide qué pasa con un dispositivo instalado cuando la unidad se anula o pasa a Inactivo (decisión 11): hoy `VehicleAnnulments` y `VehicleTransfers` no miran las instalaciones.

#### 2.7 Pruebas del backend

- **Se mueven las 5 del inventario:** `EquipmentTests`, `EquipmentCommandServiceImplTests` y `EquipmentQueryServiceImplTests` (con sus mocks reescritos: ya no hay `ExternalVehicleCatalogService` de `fleet`, y `EquipmentSupplierSummary` sale de `equipment.interfaces.acl`), `EquipmentsControllerTests` y `EquipmentsIntegrationTests`. `EquipmentCommandServiceImplTests:116` verifica hoy que el alta no llama a `markEquipmentTypeInUse` del mock de `fleet`; en `equipment` verifica que no llama a `EquipmentTypeRepository.markInstalled`.
- **Se mueven las 16 del catálogo:**
  - las 12 de hoy: `EquipmentTypeTests`, `EquipmentModelTests`, `EquipmentTypeCommandServiceImplTests` (que ya no mockea `VehicleTypeRepository`), `EquipmentModelCommandServiceImplTests`, `EquipmentTypeAuditTests`, `EquipmentTypeQueryServiceImplTests`, `EquipmentModelQueryServiceImplTests`, `EquipmentTypeApplicabilityConstraintTests`, `EquipmentTypeApplicabilityPersistenceTests`, `EquipmentTypeLockTests`, `EquipmentTypesControllerTests` y `EquipmentModelsControllerTests`;
  - las 4 de las marcas: `EquipmentBrandTests`, `EquipmentBrandCommandServiceImplTests` y `EquipmentBrandsControllerTests` de la rama, y `EquipmentBrandAuditTests` del trabajo previo de D-260.
- **Se parten:**
  - `VehicleCatalogContextFacadeTests`: sus cinco casos de equipos pasan a `EquipmentContextFacadeTests` (después de D-260 prueban lo que aplica y la pantalla de fábrica, no lo que se exige), y se suma el perfil reducido del modelo;
  - de `VehicleCatalogRacedDuplicatesTests`, el caso de la línea 146;
  - de `VehicleCatalogExportsIntegrationTests`, el de la línea 136;
  - `VehicleCatalogFixtures.equipmentType`, `equipmentBrand` y `equipmentModel`, que pasan a `EquipmentFixtures`; `VehicleEquipmentIntegrationTests:120` crea el tipo con `EquipmentFixtures`;
  - `VehicleCatalogTestData.gpsForCars`;
  - la parte de equipos de `VehicleCatalogMigrationTests`.
- **Nuevas:**
  - el adaptador `EquipmentUnits`, `EquipmentAuditTests` (regla 4.8) y `fleet_equipments` en una prueba de filas que nunca se borran;
  - `EquipmentSuppliersWiringTests`, en `procurement/infrastructure/equipment`, como `UnitPurchasesWiringTests`: el contexto arranca con `SupplierEquipmentSuppliersAdapter` entre los `EquipmentSuppliers`, y un proveedor de la empresa se resuelve por él. `ExternalSupplierService` recibe una lista: sin el adaptador, la lista queda vacía y toda alta con proveedor responde `SUPPLIER_NOT_FOUND`, sin que nada falle al arrancar (§5).
- **Se corren completas:**
  - toda la habilitación de `fleet`: `EnablementGateIntegrationTests`, `EnablementConcurrencyTests`, los seis `VehicleEnablement*`, `VehicleEquipmentIntegrationTests`, `VehicleEquipmentOverlapTests`, `VehicleEquipmentAuditTests`, `VehicleArrivalIntegrationTests`, `EnablementAnnulledUnitTests`, `EnablementDatesInLimaTests`, `EnablementPermissionsIntegrationTests` y las de D-260 (`ModelRequirementsTests`, `RequirementLossIntegrationTests`, `SecurityEquipmentIntegrationTests` y `EquipmentChangesBlockedTests`);
  - `InitialMigrationIntegrationTests`, `ReportsIntegrationTests` y `StockRowsTests`;
  - `RowLevelSecurityMigrationTests`, `FleetRowLevelSecurityTests` y `PermissionsMigrationTests`;
  - las de arquitectura y `ErpMillenniumMobilityBackendApplicationTests`, que detecta los nombres de bean repetidos.
- **`EnablementFixtures` (l.148-155) repite en SQL la regla de los requisitos,** que después de D-260 lee `vehicle_equipment_requirements`, con la pantalla de fábrica: se mantiene alineada con `ModelRequirements`.

#### 2.8 Frontend

- **Carpeta nueva `src/app/equipment`,** con sus cuatro capas.
- **Recibe los 35 archivos del inventario de `fleet`:**
  - `application`: `equipment-inventory.store.ts` y su spec;
  - `domain/model`: `equipment-item.entity.ts`, `equipment-status.ts`, `equipment-filters.ts`, `equipment-telematics.vo.ts` y `commands/{create-equipment, update-equipment, change-equipment-status}.command.ts` con `equipment-command-helpers.ts`;
  - `infrastructure/api`: `equipments-api-endpoint.ts`, `equipments-response.ts` y `{create-equipment, update-equipment, change-equipment-status}.request.ts`;
  - `infrastructure/assemblers`: `equipment-assembler.ts` y su spec;
  - `presentation`: `views/equipment-inventory-view/` (4), `components/equipment-form-dialog/` (4), `components/equipment-details-dialog/` (4), `components/equipment-status-dialog/` (4), `equipment-status-tone.ts` y `_equipment-dialog-shared.scss`.
- **Recibe los 51 archivos del catálogo de `vehiclecatalog`:** los 35 de hoy y los 16 de las marcas de la rama.
  - `application`: `equipment-catalog.store.ts` y su spec;
  - `domain/model`: `equipment-type.entity.ts`, `equipment-brand.entity.ts`, `equipment-model.entity.ts` y sus 9 comandos;
  - `infrastructure/api`: `equipment-types-api-endpoint.ts`, `equipment-types-response.ts`, `equipment-brands-api-endpoint.ts`, `equipment-brands-response.ts`, `equipment-models-api-endpoint.ts`, `equipment-models-response.ts` y sus 9 peticiones;
  - `infrastructure/assemblers`: `equipment-type-assembler.ts`, `equipment-brand-assembler.ts` y `equipment-model-assembler.ts`;
  - `presentation`: `views/equipment-types-view/` (4), `components/equipment-types-dialog/` (5, con `equipment-type-form.ts`), `components/equipment-brands-dialog/` (6, con `equipment-brand-form.ts` y su spec) y `components/equipment-model-form-dialog/` (4).
- **Archivos nuevos:**
  - `equipment/infrastructure/api/equipment-api.ts` (`EquipmentApi`), con lo que sale de `fleet-api.ts:366-381` y de `vehiclecatalog-api.ts:174+`, con los métodos de marcas;
  - `equipment/presentation/equipment.routes.ts` y su spec, con `inventory` (`stock.ver`) y `types` (`catalogos.ver`), con las mismas constantes de acceso y los mismos `title`;
  - `deactivation-question.ts`, con `equipment-type`, `equipment-model` y `equipment-brand`: una copia en `equipment/presentation/components/` o su paso a `shared`, según la decisión 12.
- **Sin `EquipmentFacade`:** ningún contexto la usaría. La pantalla de requisitos de D-260 lee los tipos de su propia respuesta (`/vehicle-equipment-requirements`), no de una fachada.
  - El inventario lee los tipos, las marcas y los modelos de `EquipmentCatalogStore`, con su estado externo, en vez de `VehicleCatalogFacade` (`equipment-inventory-view.ts:26,56` y `equipment-form-dialog.ts:66`).
  - Si V140 se conserva, el inventario sigue leyendo las sedes con `OrganizationFacade.loadCompanyBranches`.
  - Los tipos de equipamiento leen los tipos de vehículo de `VehicleCatalogFacade`, en vez de `VehicleTypesStore` (`equipment-types-view.ts:38,78` y `equipment-types-dialog.ts:40,103`).
- **`VehicleCatalogFacade`:**
  - pierde sus miembros de equipos (l.37-56 y l.150-206 de hoy) y los de las marcas (`EquipmentBrandOption`, `equipmentBrandOptions`, `equipmentBrandsLoading`, `equipmentBrandsError` y `clearEquipmentBrandsError`), y la inyección de `EquipmentCatalogStore`;
  - **gana `vehicleTypeOptions`:** todos los tipos, activos o no, con `isActive`, porque el diálogo ofrece los inactivos que un equipo ya tiene.
- **Las rutas pasan a `/equipment/inventory` y `/equipment/types`:**
  - `app.routes.ts` suma `{ path: 'equipment', loadChildren: equipmentRoutes }`;
  - `fleet.routes.ts` y `vehiclecatalog.routes.ts` dejan una redirección (`redirectTo: '/equipment/inventory'` y `'/equipment/types'`, `pathMatch: 'full'`);
  - `app-navigation.ts:175,345` cambia los enlaces sin cambiar grupo, permiso, título ni descripción.
- **Specs:**
  - las 12 que se mueven con su código: las 10 de hoy y las 2 de las marcas;
  - `vehiclecatalog.facade.spec.ts`;
  - `app-shell.spec.ts:316`, y en `:385-397` se suma `equipmentRoutes` a las rutas que compone;
  - `fleet.routes.spec.ts:115-118` pasa a `equipment.routes.spec.ts`;
  - `equipment-form-dialog.spec`, `equipment-inventory-view.spec`, `equipment-types-view.spec` y `equipment-types-dialog.spec`, con sus mocks nuevos.
- **e2e:**
  - las rutas en `e2e/screens.ts:533`, `e2e/vehicle-catalog.spec.ts:233,243,269,283,636,679`, `e2e/exports.spec.ts:232` y `e2e-live/vehiclecatalog.live.spec.ts:517,643,656`;
  - un caso que pruebe cada redirección;
  - los mocks (`support/mocks/equipments.ts` y `vehiclecatalog.ts`) no cambian, porque la API es la misma;
  - **las imágenes de referencia no se regeneran:** su nombre es `screen.name` (`visual.spec.ts:44`), y una que cambie es una regresión. La de `equipment-types` ya se regeneró en el trabajo previo de D-260;
  - opcional: sumar la pantalla `equipment-inventory` a `screens.ts`, con su imagen nueva revisada.
- **No se renombran en esta fase:** las claves de i18n (`fleet.equipment.*`, `vehicle-catalog.equipment-types.*`, `vehicle-catalog.status.*`, `vehicle-catalog.confirm.*`, `fleet.vehicles.no-plate`) ni las de `environment*.ts` (`vehicleCatalogEquipmentTypesEndpointPath`, `vehicleCatalogEquipmentBrandsEndpointPath`, `vehicleCatalogEquipmentModelsEndpointPath` y `fleetEquipmentsEndpointPath`).

#### 2.9 Documentación

- `contextos/equipment.md`, con lo de equipamiento de `fleet.md` y de `vehiclecatalog.md`, y las marcas de equipo. Las reglas conservan su id (`FLT-R…`, `VCT-R…`); VCT-R28 pasa entera a `equipment`.
- **Una decisión nueva del diccionario**, con el siguiente número libre, que precisa D-254: el inventario y el catálogo de equipos, con sus marcas, forman el contexto `equipment`; la instalación y la configuración de los requisitos (D-260) siguen en `fleet` (§5).
- El mapa vigente y el diccionario §3, con la fila 9 del bloque 9: D-254 sigue abierta.
- **Los patrones de persistencia:** una subsección `equipment` en «El orden de los candados de cada contexto», con los candados que no cambian: el inventario bloquea `fleet_equipments` `FOR NO KEY UPDATE` (`EquipmentJpaRepository:28,35`), y el catálogo, `equipment_types` y `equipment_models`.
- **Las convenciones:** la tabla de rutas por contexto (§2), con `/equipment-brands`, y el texto de los modelos de equipamiento («catálogo general de `vehiclecatalog`»).
- **`procesos-y-pantallas`:** las rutas `/equipment/inventory` y `/equipment/types` (l.304, l.320, l.358 y l.379).
- **`contextos/README.md`:** la fila de `equipment`, y la de `vehiclecatalog` sin los equipos.
- **Las cuatro copias de las skills** (`.claude` y `.cursor` de los dos repositorios), con `equipment` en la búsqueda de la regla 1.2 de `architecture-review/SKILL.md:141`, en la lista de `allowedDependencies` de `backend-module` y en la lista de contextos de `frontend-feature/SKILL.md:64`.

**Terminado cuando:**
- la habilitación de cada unidad da el mismo estado que en el paso 1;
- `vehiclecatalog` ya no sabe de equipos ni de marcas de equipo;
- `ModularityTests` pasa con `fleet` → `equipment`, `procurement` → `equipment` y `equipment` → `vehiclecatalog` y `organization`, sin ciclos;
- la aplicación arranca, sin dos beans con el mismo nombre, y `EquipmentSuppliersWiringTests` encuentra el adaptador de `procurement`.

### Fase 3 — `legacyimport`

**Por qué.** El libro de stock de 52 columnas es un modelo heredado. El libro pide traducirlo en una *Anticorruption Layer* (cap. 14) —fachadas, adaptadores y traductores— para que no contamine el modelo nuevo. Hoy esa traducción vive dentro de `procurement` y de `fleet`, y a medias: los writers de compras hablan de filas y de columnas del libro.

Después de esta fase, `legacyimport` es el único que conoce las columnas, las filas y el informe. Las reglas de cada dueño se quedan en su contexto y se exponen por su fachada:
- la factura migrada sin los 20 días (D-177);
- la nota de campaña sin mes (D-191, D-192, D-207);
- el pago de dealer con su inicial (D-197, D-199, D-248);
- la unidad registrada en su estado (D-202).

**Una decisión** (decisión 4 de la fase 0): si la carga inicial se ejecuta antes de esta fase, la fase se reemplaza por borrar el código después de la puesta en marcha, y la fase 5 crea `procurement::acl`.

**Quién llama a quién.**
- `legacyimport` lee el libro, lo agrupa, resuelve los nombres y orquesta las transacciones.
- De `fleet::acl` usa solo `matchMigratedUnits` y `resolveMigrationLocation`.
- `registerMigratedUnits` lo sigue llamando el registro de la factura en `procurement`, dentro de su transacción. Así no cambian:
  - el orden de los candados: empresa, proveedor y moneda en modo compartido; después la marca y las unidades en `fleet`; después la factura (§2.4 de la especificación, patrones de persistencia);
  - qué rechazo gana en el informe cuando una factura tiene más de uno.

**Backend — `legacyimport`.**
- **`package-info.java`** con `@ApplicationModule(displayName = "LegacyImport", allowedDependencies = {"fleet::acl", "procurement::acl", "organization::acl", "location::acl", "vehiclecatalog::acl", "imports::acl"})` y `@NullMarked`.
  - Sin `documents::acl`: no exporta ni guarda archivos; los guarda `imports`.
  - Sin `interfaces/acl`: nadie depende de él.
- **Desde `procurement` se mueven:**
  - en `interfaces/rest`: `InitialMigrationImportsController`, con sus rutas (`POST /api/v1/initial-migration-imports` y `/preview`), su permiso (`PermissionCodes.Stock.LOAD_INITIAL_MIGRATION`, `stock.cargar_migracion_inicial`) y su `@Tag("Imports")`; `ImportInitialMigrationResource` e `InitialMigrationPreviewResource`;
  - en `application/commandservices`: `InitialMigrationImportCommandService`;
  - en `application/internal/commandservices`:
    - `InitialMigrationImportCommandServiceImpl`, con el mismo nombre y el mismo paquete relativo, para que lo siga envolviendo `DeadlockRetry`;
    - `InitialMigrationRun`, `InitialMigrationCatalogs` y `StockRowMapper`;
    - `MigrationTransactions`: `REQUIRES_NEW` y el reintento de cada paso (PRC-R152);
  - en `application/queryservices`: `InitialMigrationEffects` e `InitialMigrationPreview`;
  - en el dominio: `ImportInitialMigrationCommand` (`domain/model/commands`), `StockRow` (`domain/model/valueobjects`) y `StockRowGroups` (`domain/services`);
  - en `application/internal/outboundservices/acl`: `MigrationUnitMatch`, y una copia de `ImportableModel`. El original se queda, porque lo usa la importación de las líneas de una factura (`PurchaseInvoiceLineImportQueryServiceImpl`).
- **Clases nuevas de `legacyimport`:**
  - su alcance de la carga: la empresa, su nombre, la marca con su código y la sede;
  - `LegacyImportErrors.companyInactive()`, con el mismo código `COMPANY_INACTIVE`;
  - un calendario sobre el `Clock` UTC, con el «hoy» en la zona de la empresa;
  - `LegacyTextFolding.fold` en `application/internal/policies`, copia pura de `InvoiceLineImportPolicy.fold`, que pliega los dos lados del cruce de nombres;
  - `ExternalProcurementService`, `ExternalFleetService`, `ExternalOrganizationService`, `ExternalLocationService` y `ExternalVehicleCatalogService` en `application/internal/outboundservices/acl`, con records propios y nombre de bean explícito (`@Service("legacyimportExternal…Service")`). Los de `iam` y `organization` no llevan nombre explícito: un homónimo sin nombre chocaría con `externalOrganizationService` y `externalLocationService`;
  - en el mismo paquete, los tipos de la traducción: `LegacySupplier`, `LegacyRefusals`, `LegacyNumbers` y un record propio por cada respuesta de `procurement::acl`.
- **Lo que esas clases toman hoy del interior de `procurement`, y cómo queda:**

| Hoy | Después |
|---|---|
| `StockRow` lleva `InvoiceNumber` y `OperationNumber` | Lleva `String` ya normalizados |
| `InvoiceNumber.normalize` en `StockRowMapper` | `normalizeVoucherNumber` de `procurement::acl`, por `ExternalProcurementService`, que `StockRowMapper` recibe como `LegacyNumbers` |
| `OperationNumber.normalize` en `StockRowMapper` | `normalizeOperationNumber` de `procurement::acl`, por el mismo camino: vacío si no queda un número de 1 a 100 caracteres, y entonces `IMPORTBATCH_CELL_INVALID` en `OPERACIÓN DE PAGO`, como hoy |
| `InvoiceLineImportPolicy.fold` | `LegacyTextFolding.fold` |
| `PurchaseInvoice.MAX_LINES` en `StockRowGroups` | `maxInvoiceLines()` de la fachada, pasado a `StockRowGroups.refusals` |
| `SupplierRepository.findAllByCompanyId` y `Supplier` en `InitialMigrationCatalogs` | `fetchMigrationSuppliers(companyId)`, que `ExternalProcurementService` traduce a `LegacySupplier(id, publicId, legalName, tradeName, isActive)` |
| `PurchaseInvoiceRepository.findCurrentByNumber` y `PurchaseInvoiceItem` en el orquestador | `fetchCurrentInvoiceByNumber(companyId, supplierId, number)`, traducido a un record de `legacyimport` |
| `MigrationRefusals` en `InitialMigrationRun.failFrom` | `LegacyRefusals(List<Refusal(int row, String code, String column)>)`, que `ExternalProcurementService` arma desde `MigratedRecordRefusals`, como hoy `ExternalFleetService.translatedRefusals` en `procurement` |
| `ProcurementErrors.companyInactive()` | `LegacyImportErrors.companyInactive()` |
| `ProcurementCalendar` | El calendario de `legacyimport` |
| `ExternalOrganizationService`, `ExternalLocationService`, `ExternalVehicleCatalogService` y `ExternalFleetService` de `procurement`, con `InvoiceCompany`, `InvoiceCountry`, `BrandReference`, `OrderCurrency` y `LineColor` | Los de `legacyimport` |
| `ImportBatch*` de `shared.importbatches` | `imports::acl` (fase 1) |

- **La traducción.**
  - El orquestador, `InitialMigrationRun` e `InitialMigrationCatalogs` viven en `application/internal/commandservices` y nombran solo tipos de `legacyimport`. Un tipo de `procurement::acl` o de `fleet::acl` fuera de `outboundservices/acl` rompe `ForeignAclStaysInTranslationLayerTests`.
  - `ExternalProcurementService` arma las peticiones de `procurement::acl` y traduce cada respuesta a un record propio, como hoy `ExternalFleetService.registerMigrated` de `procurement` arma `RegisterMigratedUnitsRequest`.
  - `StockRowMapper` sigue estático y recibe la normalización como parámetro: `read(row, companyName, today, LegacyNumbers numbers)`. `LegacyNumbers` da `voucher(String)` y `operation(String)`, que responden `Optional<String>`. La implementa `ExternalProcurementService`, y el orquestador se la pasa. Un vacío es `IMPORTBATCH_CELL_INVALID` en su columna, como hoy.
  - Cada línea de una petición lleva como `line` el número de su fila. `procurement` lo devuelve en el rechazo sin saber qué es.
  - `ExternalProcurementService` traduce cada `MigratedRecordField` a su columna del libro con un `switch` exhaustivo. Para los campos de la unidad usa el mapa de hoy, `StockRowMapper.COLUMNS`, que pasa a ese `switch`.
  - `InitialMigrationRun.failFrom` lee `LegacyRefusals`. Un error sin rechazos llega tal cual y cae en la primera fila del paso, como hoy.
- **`procurement::acl`, la primera fachada de `procurement`:** `procurement/interfaces/acl/package-info.java` con `@NamedInterface("acl")`, `ProcurementContextFacade`, y su implementación `procurement/application/acl/ProcurementContextFacadeImpl`. Devuelve tipos simples o records del propio paquete `acl`, nunca un tipo del dominio de compras ni de `fleet::acl`.
  - **Lectura:**
    - `fetchMigrationSuppliers(Long companyId)` → `List<MigrationSupplier(id, publicId, legalName, tradeName, isActive)>`;
    - `fetchCurrentInvoiceByNumber(Long companyId, Long supplierId, String number)` → `Optional<MigratedInvoiceReference(invoiceId, Set<Long> currentVehicleIds)>`;
    - `maxInvoiceLines()`;
    - `normalizeVoucherNumber(String raw)` → `Optional<String>`;
    - `normalizeOperationNumber(String raw)` → `Optional<String>`: vacío si no queda un número de 1 a 100 caracteres, lo que exige `OperationNumber`.
    - Las dos primeras leen tablas de empresa: exigen la transacción de quien llama (`MANDATORY`) y corren en el paso `REQUIRES_NEW` que las pide. Las otras tres no son transaccionales: las filas se leen fuera de toda transacción, en el bucle de `run()` del orquestador.
  - **Escritura,** con `@Transactional(propagation = MANDATORY)`, como `registerMigratedUnits` de `fleet`:
    - `registerMigratedInvoice(RegisterMigratedInvoiceRequest)` → `Result<MigratedInvoice, ApplicationError>`, con el `invoiceId`, los vehículos por VIN y el estado en que entró cada unidad;
    - `registerMigratedCreditNote(RegisterMigratedCreditNoteRequest)` → `Result<Boolean, ApplicationError>`;
    - `registerMigratedPayment(RegisterMigratedPaymentRequest)` → `Result<Boolean, ApplicationError>`.
  - **La petición de la factura** lleva la empresa, la marca, la sede, el proveedor, la moneda, el número, la fecha y la tasa. Por cada línea lleva el precio y los datos de la unidad que hoy viajan a `fleet`: VIN, motor, modelo, año, color, número del libro, propietario, la habilitación, el SOAT y la llave con su etiqueta, que calcula `legacyimport` con el código de la marca.
  - **Los tipos de las peticiones:**
    - la empresa, la marca y la sede, por su `Long id`, ya resueltas y revisadas por `legacyimport`; la sede puede faltar;
    - el proveedor, por su `UUID publicId`, como hoy los `Draft` de los writers;
    - la moneda, por su código ISO;
    - el número de la factura o de la nota y la operación del pago, como `String` ya normalizados;
    - la fecha, como `LocalDate`; la tasa, como `BigDecimal`, que puede faltar;
    - el modelo y el color de cada unidad, por su `Long id`;
    - cada línea, con su `int line`.
  - **Los rechazos** vuelven como `Result.failure(ApplicationError)` con `MigratedRecordRefusals(List<Refusal(int line, String code, MigratedRecordField field)>)` en `context()`, como hoy viaja `MigrationRefusals`. Un `ApplicationError` sin ese contexto rechaza el paso entero.
    - `MigratedRecordField` es un enum de `procurement` con los campos de compras (`INVOICE_NUMBER`, `INVOICE_DATE`, `PRICE`, `RATE`, `NOTE_NUMBER`, `NOTE_DATE`, `NOTE_VALUE`, `OPERATION`, `PAYMENT_DATE` y `PAYMENT_AMOUNT`) y los de la unidad que rechaza `fleet` (`VIN`, `LEGACY_NUMBER`, `MODEL`, `COLOR`, `TITLE`, `TITLE_DATE`, `PLATE`, `PLATES`, `ARRIVAL`, `GPS`, `SCREEN`, `SOAT`, `KEY`, `DELIVERY` y `READY`).
    - El rechazo de `fleet` llega al registro de la factura por `registerMigratedUnits`. `ExternalFleetService` de `procurement` traduce cada `MigratedUnitField` al valor de igual nombre con un `switch` exhaustivo, así que `ProcurementContextFacade` no devuelve ningún tipo de `fleet::acl`.
  - **La prueba de contrato** de `ProcurementContextFacade` fija los tipos de las peticiones; que las escrituras y las dos primeras lecturas exigen transacción; que un rechazo vuelve con su línea y su campo, también uno de `fleet`; y lo que responden las dos normalizaciones.
- **En `procurement` se quedan las reglas de compras de lo migrado, sin el libro:**
  - `MigratedInvoiceWriter`, `MigratedCreditNoteWriter` y `MigratedPaymentWriter` dejan de nombrar `StockRow` y `StockRowMapper`: reciben los records de la fachada y rechazan por `MigratedRecordField`.
  - `MigrationScope` se queda, sin el nombre de la empresa ni el código de la marca, que usa solo `legacyimport`.
    - Se arma dentro del miembro de la fachada con una lectura sin candado de la empresa (`fetchPurchasingCompaniesByIds`, la de las vistas) y de su país (`fetchInvoiceCountry`).
    - Cada writer conserva sus lecturas `FOR SHARE` en el orden de hoy. El pago sigue tomando primero el candado consultivo de su `requestId`, antes de leer nada (patrones de persistencia, «La carga inicial»); la factura y la nota empiezan por la empresa. Una lectura `FOR SHARE` de la empresa para armar el alcance pondría su candado antes del `requestId`.
  - Se quedan `MigrationRefusals` y `MigrationUnit`.
    - `MigrationRefusals` se redefine como `Refusal(int line, String code, MigratedRecordField field)`. Lo arman los writers y `ExternalFleetService`, y `ProcurementContextFacadeImpl` lo copia a `MigratedRecordRefusals`.
    - `MigrationUnitLine` se redefine sin `StockRow`.
  - De `ExternalFleetService` se quedan `registerMigrated` y `fetchUnits`; `matchMigrated` y `resolveMigrationLocation` salen a `legacyimport`.
  - Se quedan los códigos que levantan los writers:
    - `INITIALMIGRATION_RATE_NOT_ONE`, `INITIALMIGRATION_CREDIT_NOTE_REGISTERED`, `INITIALMIGRATION_CREDIT_NOTE_ZEROES_LINE`, `INITIALMIGRATION_PAYMENT_REGISTERED` e `INITIALMIGRATION_PAYMENT_INCONSISTENT`;
    - los de compras: `PURCHASEINVOICE_*`, `CREDITNOTE_*`, `SUPPLIERPAYMENT_DATE_IN_FUTURE` y los que el pago toma de `SupplierPaymentApplications.owed` y de `SupplierPaymentAllocations.allocations`;
    - `IMPORTBATCH_CELL_REQUIRED` e `IMPORTBATCH_CELL_INVALID`, para la tasa de la factura y el monto del pago, con su campo en `MigratedRecordField`. Son literales del vocabulario de `imports`: levantarlos no exige `imports::acl`.

    Los demás `INITIALMIGRATION_*` son literales del orquestador, de `InitialMigrationRun`, de `StockRowMapper`, de `StockRowGroups` y de `InitialMigrationCatalogs`, y se van con ellos. Las claves de los `messages*.properties` no cambian.
  - `procurement` retira `"imports::acl"` de su `package-info.java`: ya no lo usa.
- **En `fleet` no cambia nada:** `MigratedUnitRegistrations`, `MigrationTargetPolicy` y los tres miembros de su fachada (`matchMigratedUnits`, `resolveMigrationLocation` y `registerMigratedUnits`), porque registrar una unidad en su estado es regla de flota (D-202). Siguen ahí `INITIALMIGRATION_LEGACY_NUMBER_TAKEN` e `INITIALMIGRATION_SCREEN_NOT_FROM_FACTORY`.
- **Las transacciones y los candados siguen como hoy.**
  - El orquestador no es transaccional y sigue envuelto por `DeadlockRetry`.
  - `MigrationTransactions` abre una transacción `REQUIRES_NEW` para cada uno de estos pasos: el alcance, los catálogos con los VIN, la búsqueda de cada factura existente, cada factura, cada nota, cada pago y el registro de la carga. Cada paso se repite una vez si pierde un interbloqueo.
  - Los miembros de las fachadas corren en la transacción de quien los llama.
  - El candado de la carga es `ImportLocks.tryOnSession`, de `imports::acl`, en su propia conexión.
  - Row Level Security no cambia: el alcance lo fija `AccessScopeJpaTransactionManager` en cada transacción, sea cual sea el módulo.
- **El contrato no cambia:** las rutas, el permiso, el `@Tag`, los recursos y los códigos de error son los mismos.

**Frontend.** No se muda: las rutas no cambian y el módulo se borra después de la puesta en marcha. Se quedan en `procurement` hasta el borrado:
- el diálogo de la carga, 7 archivos:
  - `application/initial-migration.store.ts` y su spec;
  - `domain/model/initial-migration.ts`;
  - `infrastructure/api/initial-migration-api-endpoint.ts`;
  - `presentation/components/initial-migration-import-dialog/`, con su `.ts`, su `.html` y su `.scss`;
- `ProcurementFacade.openInitialMigrationImport`, que llama la pantalla Cargas (`imports-view.ts`).

`e2e/screens.ts` (`initial-migration-preview`) y `e2e/support/mocks/imports.ts` no cambian.

**Orden dentro de la fase.** Cada paso deja el build y las pruebas de arquitectura en verde, y todo se integra junto.
1. **Caracterización, en `develop`.** `InitialMigrationIntegrationTests` comprueba el código de cada rechazo (`line.get(3)`), pero la columna solo de `PROVEEDOR` y de `FECHA DE FACTURA`. Suma la columna (`line.get(2)`):
   - de un rechazo de `fleet`: `N°` para `INITIALMIGRATION_LEGACY_NUMBER_TAKEN`, en `aLegacyNumberTakenInTheBrandIsRefused`;
   - de los writers de la factura y de la nota: `FACTURA` para `PURCHASEINVOICE_NUMBER_FORMAT`, `TC` para `INITIALMIGRATION_RATE_NOT_ONE` y `VALOR NC / ND` para `INITIALMIGRATION_CREDIT_NOTE_ZEROES_LINE`;
   - de `MigratedPaymentWriter`, en un caso nuevo, porque ninguno lo cubre: el `INITIALMIGRATION_PAYMENT_INCONSISTENT` del caso de dos montos lo levanta el orquestador antes del writer. Por ejemplo, `INITIALMIGRATION_PAYMENT_REGISTERED` en `OPERACIÓN DE PAGO`.

   Pasan en `develop` antes de tocar el código. Que la lista de precarga no escribe nada ya lo cubre `theListBeforeTheLoadTellsEverythingAndWritesNothing`.
2. **`procurement::acl`, con el orquestador todavía en `procurement`.** Se crean la fachada, sus records y su prueba de contrato. Los writers reciben los records de la fachada y rechazan por `MigratedRecordField`, y el orquestador llama a la fachada.
3. **`legacyimport`.** Se mudan el orquestador y sus clases, con `ExternalProcurementService` y los demás traductores, y `procurement` retira `imports::acl`. Los casos del paso 1 pasan sin cambios.
4. **Documentación y skills.**

**Pruebas.**
- **Se mueven a `legacyimport`**, espejando los paquetes:
  - `InitialMigrationIntegrationTests` (27 casos, más el del paso 1), con `StockWorkbook`, que es package-private y va al mismo paquete;
  - `InitialMigrationCatalogsTests`, que se reescribe sobre `LegacySupplier`, porque hoy construye el agregado `Supplier`.
- **Se mueven y cambian:**
  - `StockRowMapperTests`: pasa un `LegacyNumbers` de prueba a `read` (línea 52) y compara `String` en vez de `.value()` (líneas 93, 97 y 99);
  - `StockRowGroupsTests`: su ayudante de la línea 27 pasa el tope de 100 líneas a `refusals`.
- **Nueva, en `legacyimport`:** cada valor de `MigratedRecordField` tiene su columna en `StockRowMapper.HEADERS`.
- **Nuevas, en `procurement`:** la prueba de contrato de `ProcurementContextFacade` y los writers sin `StockRow`.
- **Se quedan:** `MigrationTargetPolicyTests` (`fleet`) y `SpreadAndCustomerInitialMigrationTests` (el esquema de V127, en `procurement`).
- **Se corren:**
  - las de arquitectura (`ModularityTests`, `ForeignAclStaysInTranslationLayerTests`, `DeadlockRetryTests` y las demás de §2);
  - `ErpMillenniumMobilityBackendApplicationTests`, por los nombres de bean;
  - en el CI, `e2e/imports.spec.ts` y `e2e-live/imports.live.spec.ts`.

**Documentación.**
- Una decisión nueva del diccionario que precisa D-200: la carga inicial la orquesta `legacyimport` por `procurement::acl`, y el registro de la factura sigue llamando a `fleet`.
- `contextos/legacyimport.md`:
  - las reglas de la traducción del libro: PRC-R142, PRC-R144, PRC-R146, PRC-R147, PRC-R149 a PRC-R152, PRC-R154 y PRC-R158 a PRC-R161. Conservan su id: un id no se reutiliza ni se renumera;
  - la sección «La carga inicial» de §4 y el anexo de las 52 columnas, que salen de `contextos/procurement.md`.
- `contextos/procurement.md`:
  - se quedan las reglas de compras, PRC-R143, PRC-R145, PRC-R148 y PRC-R153, que dicen que se exponen por `procurement::acl`. `legacyimport.md` las cita para los grupos y el orden de los pasos;
  - §5 (línea 642): `legacyimport` depende de `procurement::acl`.
- `contextos/fleet.md`.
- `contextos/README.md`: la fila de `legacyimport.md`, y en la línea 55, que sus reglas conservan el prefijo `PRC-`.
- El mapa vigente.
- El diccionario: §3, y §11.8, línea 3489: la carga inicial la orquesta `legacyimport`, y la factura con sus unidades la registra `procurement`.
- Las convenciones §2.2: `/initial-migration-imports` y `/initial-migration-imports/preview` pasan de la fila de `procurement` a una fila `legacyimport`.
- Los patrones de persistencia, «La carga inicial»: el orquestador es de `legacyimport`; los candados no cambian.
- El `README.md` del backend.
- Las skills, en sus cuatro copias (decisión 15 de la fase 0): §1 de `backend-module/SKILL.md` («Las cargas y los reportes») y §2 (`allowedDependencies`).

**Terminado cuando:**
- `procurement` no nombra el libro:
  - `git grep -nw -e StockRow -e StockRowMapper -e StockRowGroups -e ImportRow -- src/main/java/ky/millennium/mobility/erp/procurement` no devuelve nada. Sin `-w`, `ImportRow` encuentra `InvoiceErrorContexts.ImportRows`, de la importación de las líneas de una factura, que se queda;
  - `git grep -n "erp\.imports\." -- src/main/java/ky/millennium/mobility/erp/procurement` no devuelve nada;
  - los records de `procurement::acl` y los writers hablan de `line`, nunca de `row` ni de una columna;
- `ModularityTests` y `ForeignAclStaysInTranslationLayerTests` pasan;
- `InitialMigrationIntegrationTests` da los mismos conteos, el mismo informe, los mismos códigos y las mismas columnas que antes.

**Después de la puesta en marcha, el módulo se borra,** en una entrega propia de 1 a 2 días y riesgo bajo, cuando la carga inicial ya corrió.
- **Backend:**
  - sale `legacyimport` entero;
  - de `procurement`, los miembros de migración de `ProcurementContextFacade` con sus records, los tres writers, `MigrationScope`, `MigrationRefusals`, `MigrationUnit`, `MigrationUnitLine` y `registerMigrated` de `ExternalFleetService`. Si para entonces la fase 5 ya puso miembros en `procurement::acl`, la fachada se queda con ellos;
  - de `fleet`, `matchMigratedUnits`, `resolveMigrationLocation` y `registerMigratedUnits`, con `MigratedUnitRegistrations`, sus records de `fleet::acl` (los `Migrated*` y `RegisterMigratedUnitsRequest`) y `MigrationTargetPolicy` con su prueba;
  - los `INITIALMIGRATION_*` que ya nadie levanta, con sus claves en los `messages*.properties`;
  - la propiedad `imports.max-rows.migracion-inicial`, con `IMPORTS_MAX_ROWS_INITIAL_MIGRATION` en el `README.md` y en `.env.example`.
- **Frontend:**
  - el diálogo (sus 7 archivos), `ProcurementFacade.openInitialMigrationImport` y la opción de la pantalla Cargas que lo abre (`imports-view.ts`);
  - la pantalla `initial-migration-preview` de `e2e/screens.ts`, con sus tres imágenes;
  - los mocks de `/initial-migration-imports` en `e2e/support/mocks/imports.ts`, los dos casos de `e2e/imports.spec.ts` que abren «Migración inicial» y el caso del libro de stock de `e2e-live/imports.live.spec.ts`;
  - las claves `procurement.initial-migration` de `es.json`.
- **Documentación:** `contextos/legacyimport.md`, la fila `legacyimport` de las convenciones §2.2, el mapa vigente y las skills.
- **Se conservan:**
  - los datos, los caminos `MIGRACION_INICIAL`, `legacy_number` y las migraciones V117 a V119 y V127;
  - `ImportType.MIGRACION_INICIAL`, porque hay lotes con ese tipo;
  - el permiso `stock.cargar_migracion_inicial`, con su constante en el backend y en el frontend. Ninguna ruta lo pide ya; retirarlo es una migración de `iam` aparte, con su decisión (decisión 1 de la fase 0).

### Fase 4 — `financing`

**Por qué.** Otro lenguaje (fondo, cesión, fideicomiso, garantía) y otro actor, Contabilidad (cap. 14). La invariante que cruza va en los dos sentidos: una unidad con una cesión abierta no se anula (D-182), y una unidad anulada no se cede ni se le cierra una cesión. Se cumple en una sola transacción con el candado de la unidad, que toma `fleet`, y con un puerto que declara `fleet` y que implementa `financing` (cap. 16). La dependencia en los dos sentidos se escribe como dos relaciones, las del mapa objetivo §4: `financing` es cliente de `fleet` (*Customer/Supplier*) por las unidades y su candado, y `fleet` pregunta por su puerto `UnitFundAssignments`, que implementa `financing`.

#### Antes de empezar

- **La fase 1 está integrada** y `imports::acl` expone todo lo que usan la carga de garantía y la lista de cesiones:
  - `ImportBatchCommandService` con `RegisterImportBatchCommand` y `RegisteredImportBatch`;
  - `ImportBatchQueryService` (`handle(GetImportBatchByPublicIdQuery)` y `fetchReferencesByIds`), con `GetImportBatchByPublicIdQuery`, `ImportBatchView` e `ImportBatchReference`;
  - `ImportOutcome`, `ImportLocks`, `ImportType` e `ImportCounts`;
  - `ImportRefusals.conflict()` e `ImportReading.reads(…)`;
  - el libro (`ImportWorkbookReader`, `ImportWorkbook`, `ImportRow`, `ImportCell`, `ImportCellException`, `ImportSheetSpec`, `ImportReportWriter`, `ImportReport` con `SummaryLine` y `ExtraSheet`, `ImportRowFailure`, `ImportSummaries`, `ImportFailureMessages`);
  - la respuesta de una carga (`ImportBatchResourceFromViewAssembler`, `ImportResultResource`).
- **Decisión A o B: quién calcula `registerFundAssignment`** (fase 0, decisión 5; convenciones §4.2):
  - **A, recomendada.** El puerto de `fleet` pregunta a `financing` si la sesión puede ceder la unidad ahora. `VehicleResource.allowedActions.registerFundAssignment` sigue igual y el contrato no cambia: la fase es solo de estructura, como promete el §1.
  - **B.** `registerFundAssignment` sale del recurso de la unidad y lo responde un subrecurso nuevo de `financing` (por ejemplo `GET /vehicles/{id}/fund-standing`; la página de `GET /vehicles/{id}/fund-assignments` no tiene lugar para una acción de la colección). Su forma se acuerda antes en convenciones §4.2 y §6.2, el §1 y el §8 la declaran como excepción, y backend y frontend se integran juntos. Lo que agrega está marcado «con la opción B» en esta fase.
- **Decisión A1 o A2: quién dibuja la pestaña Cesiones** de la ficha de la unidad (`?seccion=cesiones`; fase 0, decisión 6):
  - **A1, recomendada, sin excepción nueva.** La lista se queda en `fleet` y la alimenta `FinancingFacade`, con tipos propios y su propia carga y error (regla 1.3). Los flujos de ceder y cerrar los abre la fachada.
  - **A2.** Una pieza de UI prestada en `financing/presentation/public/`, con su excepción en la regla 1.2 del frontend y su revisión de `DESIGN.md`.

#### Backend — el módulo

- **`financing/package-info.java`** con `@ApplicationModule(displayName = "Financing", allowedDependencies = {"shared", "fleet::acl", "organization::acl", "iam::acl", "audit::acl", "imports::acl", "documents::acl"})` y `@NullMarked`. Para qué es cada dependencia:
  - `iam::acl`: el nombre de quien registró una cesión. Supone la opción B de la decisión 6 del mapa vigente (fase 0, decisión 9);
  - `audit::acl`: el cierre a mano y el que hace la carga escriben la bitácora (`VehicleFundAssignment.FIELD_PERIOD`). La corrección del código de un fondo no la escribe;
  - `imports::acl`: la carga de garantía y el lote de cada cesión;
  - `documents::acl`: la exportación de los fondos.
- **Sin fachada propia en esta fase:** nadie la consume. `reporting` sigue leyendo `vehicle_fund_assignments` por SQL, como modelo de lectura (regla 1.2).

#### Backend — lo que se mueve desde `fleet` (54 clases)

Mismo nombre y mismos subpaquetes, bajo `ky.millennium.mobility.erp.financing`:

| Capa | Clases |
|---|---|
| `domain/model` | `Fund` y `VehicleFundAssignment`; `FundType`; `AssignVehicleToFundCommand`, `ChangeFundStatusCommand`, `CloseVehicleFundAssignmentCommand`, `CreateFundCommand`, `ImportGuaranteeListCommand` y `UpdateFundCommand`; `FundSearchFilter`, `GetFundByPublicIdQuery`, `GetVehicleFundAssignmentsQuery` y `SearchFundsQuery` |
| `domain` (resto) | `FundCodeTakenException` y `VehicleFundAssignmentOverlapException`; `FundRepository` y `VehicleFundAssignmentRepository`; `GuaranteeListPolicy` |
| `application` | `FundCommandService`, `VehicleFundAssignmentCommandService`, `FundQueryService`, `FundView`, `VehicleFundAssignmentQueryService`, `VehicleFundAssignmentView`; `FundCommandServiceImpl`, `VehicleFundAssignmentCommandServiceImpl`, `FundQueryServiceImpl`, `FundViewFactory`, `VehicleFundAssignmentQueryServiceImpl` y `VehicleFundAssignmentViewFactory` |
| `infrastructure` | `FundRepositoryImpl`, `VehicleFundAssignmentRepositoryImpl`, `FundPersistenceAssembler`, `VehicleFundAssignmentPersistenceAssembler`, `FundPersistenceEntity`, `VehicleFundAssignmentPersistenceEntity`, `FundJpaRepository` y `VehicleFundAssignmentJpaRepository` |
| `interfaces/rest` | `FundsController` (`/funds`), `VehicleFundAssignmentsController` (`/vehicles/{vehicleId}/fund-assignments`, la misma ruta servida por otro módulo, con `Stock.VIEW` en la clase y `Funds.REGISTER_ASSIGNMENT` en el `POST`) y `FundAssignmentsController` (`/fund-assignments/{id}/closure`); `ChangeFundStatusResource`, `CloseVehicleFundAssignmentResource`, `CreateFundResource`, `CreateVehicleFundAssignmentResource`, `FundResource`, `ImportGuaranteeListResource`, `UpdateFundResource` y `VehicleFundAssignmentResource`; `CreateFundCommandFromResourceAssembler`, `CreateVehicleFundAssignmentCommandFromResourceAssembler`, `FundResourceFromViewAssembler`, `UpdateFundCommandFromResourceAssembler` y `VehicleFundAssignmentResourceFromViewAssembler` |

#### Backend — lo que se parte

- **La carga de garantía.**
  - `FleetImportCommandService.handle(ImportGuaranteeListCommand)` y su implementación en `FleetImportCommandServiceImpl` (el método, más `fundCodesOf`, `closureRows`, `closureRow`, `CLOSURE_HEADERS` y `maxGuaranteeRows`) pasan a `FundAssignmentImportCommandService` y `FundAssignmentImportCommandServiceImpl`.
  - La implementación va en `financing/application/internal/commandservices`, para que `DeadlockRetry` la envuelva, con `@Transactional` por método.
  - La propiedad sigue siendo `imports.max-rows.garantia-mobiliaria`.
  - Se copian los ayudantes privados (`refusalToLoad`, `outcome`, `rollBack`, `fail` y `report`), la constante `PLATE` y `companyNotFound`.
  - `plateOf` se escribe sobre `FleetContextFacade.normalizePlate`: una placa que no normaliza es `ImportCellException.CELL_INVALID`, como hoy.
  - `VehicleFundAssignmentRepository` suma `findVehicleIdsOpenToFund(Long fundId)`: los ids de las unidades con una cesión abierta al fondo, la lectura que hoy va en la subconsulta de `VehicleJpaRepository.lockByCompanyIdAndPlatesOrFund`.
  - `fleet` conserva la carga del impuesto vehicular y deja de inyectar `FundRepository`, `VehicleFundAssignmentRepository` y `ChangeLogCommandService` en esa clase.
- **`FleetImportsController.importGuaranteeList`** pasa a `FundAssignmentImportsController`, con `POST /fund-assignment-imports`, `Funds.LOAD_GUARANTEE_FILE` y el mismo `@Tag(name = "Imports")`.
- **La exportación de los fondos.** `FleetDocumentKinds.FUNDS` y `fundsDocumentKind` pasan a `FinancingDocumentKinds`, con el mismo `kind` `funds`, el mismo alcance (`DocumentScopes.grantsOf(scope, Funds.VIEW)`) y la misma versión: `funds()` más la de `organization`.
- **Las versiones de datos.** `FleetDataVersions.funds()` y `JdbcFleetDataVersions.funds()` pasan a `FinancingDataVersions` y `JdbcFinancingDataVersions` (`jdbcDataVersions.of("funds")`). `vehicle_fund_assignments` lo sigue nombrando solo `JdbcReportingDataVersions`.
- **Los errores.**
  - Pasan a `FinancingErrors` los de fondos de `FleetErrors`: `deniedOnCompany`, `fundNotFound`, `vehicleFundAssignmentNotFound`, los dos `fundConflict`, `fundCodeLocked`, `fundInactive`, `vehicleFundAssignmentCompanyMismatch`, `vehicleFundAssignmentConflict`, `vehicleFundAssignmentAlreadyClosed` y `vehicleFundAssignmentEndNotAfterStart`. También los códigos que hoy van en línea: `VEHICLEFUNDASSIGNMENT_LIST_DATE_IN_FUTURE` y `VEHICLEFUNDASSIGNMENT_IMPORT_UNCONFIRMED`.
  - `FinancingErrors` copia, con el mismo código, los que ceder, cerrar, listar y cargar responden y que también usa `fleet`: `VEHICLE_NOT_FOUND`, `VEHICLE_ANNULLED`, `ACCESS_DENIED` (como `deniedOnUnit`), `COMPANY_NOT_FOUND` y `COMPANY_INACTIVE`.
  - `VEHICLE_FUND_ASSIGNMENT_OPEN` es de la anulación y se queda en `FleetErrors`.

#### Backend — lo nuevo en `financing`

- **La traducción**, en `application/internal/outboundservices/acl`, con nombre de bean explícito: `iam` ya tiene un `ExternalOrganizationService` con el nombre por defecto, y otro igual impide arrancar.
  - `ExternalFleetService`, `@Service("financingExternalFleetService")`.
  - `ExternalOrganizationService`, `@Service("financingExternalOrganizationService")`: la empresa por su identificador público, `resolveScope` (la empresa `FOR SHARE`), las referencias de las empresas, sus zonas y `fetchDataVersion`.
  - `ExternalIamService`, `@Service("financingExternalIamService")`: `fetchAccountReferencesByIds`.
  - Sus records: `AssignedUnit`, `OwningCompany` y `AccountName`. No repiten el nombre simple de `FinancedUnit` de `fleet::acl` ni el de `AccountReference` de `iam::acl`, que la misma clase de traducción nombra. `AccountName` reemplaza a `AccountReferenceView` de `fleet`, que hoy usa `VehicleFundAssignmentViewFactory`.
- **`FinancingSessionAccount`** (con `CurrentAccessScope`) y **`FinancingCalendar`**, el «hoy» y la zona de la empresa (D-40). Reemplaza a `CompanyCalendar` de `fleet`, y no lleva ese nombre para no repetir el bean. Lee el momento de su reloj por `now()`, que una prueba reemplaza espiándolo, como `CompanyCalendar`.
- **El adaptador del puerto:** `infrastructure/fleet/VehicleFundAssignmentUnitFundAssignmentsAdapter implements UnitFundAssignments`. Solo depende de `VehicleFundAssignmentRepository`, y de `CurrentAccessScope` y `FinancingCalendar` mientras implementa `offersCession`: con la opción A, siempre; con la B, hasta el paso 5. Así no hay un ciclo de beans con `FleetContextFacade`.

#### Backend — lo que cambia en `fleet`

- **El puerto `interfaces/acl/UnitFundAssignments`:**
  - `boolean hasOpenAssignment(Long vehicleId)`;
  - con la opción A, también `boolean offersCession(Long vehicleId, Long companyId, Long commercialBrandId)`; con la B, solo de los pasos 1 a 4. Es la regla de hoy: `fondos.registrar_cesion` en toda la empresa o en la marca de la unidad (DP-11), `fondos.ver` en su empresa, y ninguna cesión abierta ni que termine después de hoy en la zona de la empresa (D-40).
- **`application/internal/outboundservices/funds/UnitFundFacts`** envuelve el puerto. Inyecta **un solo bean obligatorio**, no una lista: sin el bean, una unidad cedida se anularía sin aviso (D-182).
- **Miembros nuevos de `FleetContextFacade`**, con el record nuevo `FinancedUnit(id, publicId, companyId, commercialBrandId, plate, vin, annulled)`. No se toca `UnitSummary`, que usa `procurement`.
  - `Optional<FinancedUnit> lockVisibleUnitByPublicId(UUID)` y `lockVisibleUnitById(Long)`: con `Propagation.MANDATORY` y `FOR NO KEY UPDATE`, y vacío si la sesión no ve la unidad con `stock.ver` en su marca (`VehicleAccessPolicy.sees`). La regla de la visibilidad la decide `fleet`.
  - `Optional<FinancedUnit> fetchVisibleUnitByPublicId(UUID)` y `fetchVisibleUnitById(Long)`: lo mismo sin candado, para la lista y la respuesta de una escritura. Sin transacción propia: corren en la de quien llama, como `fetchUnitsByIds`. Fuera de una, la unidad no se ve y la respuesta sería un `404` falso.
  - `Map<Long, FinancedUnit> lockUnitsOfCompanyByPlatesOrIds(Long companyId, Set<String> plates, Set<Long> vehicleIds)`: con `Propagation.MANDATORY`, en una sentencia y por `id` ascendente.
  - `Optional<String> normalizePlate(String written)`: la regla de `LicensePlate`, para que la carga marque una placa mal escrita sin copiar la regla.
- **El repositorio de la unidad.**
  - Se agrega `VehicleJpaRepository.lockByCompanyIdAndPlatesOrIds`. Recibe `List.of("")` por placas vacías y `List.of(-1L)` por ids vacíos, como hoy `VehicleRepositoryImpl` con las placas: un `IN ()` vacío no es SQL válido. No filtra por visibilidad: quien llama ya exigió el permiso en toda la empresa y en todas sus marcas.
  - Se borran `VehicleJpaRepository.lockByCompanyIdAndPlatesOrFund`, `VehicleRepository.findAllByCompanyIdAndPlatesOrAssignedToFundForUpdate` y su implementación en `VehicleRepositoryImpl`. Leen `vehicle_fund_assignments`, que deja de ser de `fleet`.
- **Quién usa el puerto:**
  - `VehicleAnnulments`: la guarda de la anulación. Por ella pasan la anulación explícita y la de las unidades que devuelve una nota de crédito (`ReturnedUnits`, que no cambia).
  - `VehicleAllowedActions`: `mayAnnul` pasa por `UnitFundFacts`, y `mayCede` delega en `offersCession`. `VehicleAllowedActions` pierde su constructor con `Clock`, que solo servía a `mayCede`. Con la opción B, `mayCede` se borra en el paso 5.
- **Con la opción B:** `registerFundAssignment` sale de `VehicleAllowedActionsView`, de `VehicleAllowedActionsResource` y de `VehicleResourceFromViewAssembler`.
- **Javadoc:** `package-info.java` de `fleet`.

#### Otros módulos

- **`reporting`** no cambia de código. `JdbcReportsReadRepository` sigue leyendo `vehicle_fund_assignments` para la columna «GARANTÍA MOBILIARIA» (`export.stock.column-33`), y `JdbcReportingDataVersions` sigue nombrándola. Su `package-info.java` dice que esa tabla se lee como modelo de lectura (*Conformist*).
- **Javadoc:** en `location/interfaces/acl/CurrencyUsage` y en `procurement` (`PurchaseOrderIssuing` y `ExternalOrganizationService`), «fleet» pasa a «financing» donde hablan de los fondos.
- **No cambian:**
  - `PermissionCodes.Funds`, `ImportType.GARANTIA_MOBILIARIA`, las claves de los mensajes (`export.funds.*` y `error.*`) y `scripts/database/hand-over-database.sql`, que no tiene líneas de fondos;
  - la bitácora: la entidad `vehicle_fund_assignments`, el campo `period` y ningún campo confidencial. `financing` sigue anotando la marca de la unidad en cada fila (D-102), que toma de `FinancedUnit`;
  - `ForeignAclStaysInTranslationLayerTests`: el adaptador implementa el puerto, y los records de `fleet` solo se nombran en la traducción.

#### Los candados

Pasan a una sección `financing` de los patrones de persistencia:

| Operación | Candados, en orden | Lo que lee después del candado |
|---|---|---|
| Ceder una unidad | La unidad, `FOR NO KEY UPDATE`, por la fachada de `fleet`; el fondo, `FOR SHARE` | Si la unidad está anulada y las cesiones de la unidad |
| Cerrar una cesión | La cesión sin candado, para saber su unidad; la unidad, por la fachada | La cesión, que vuelve a leer |
| La carga de garantía | El fondo, `FOR SHARE`; el candado consultivo de la carga, que no espera; la lectura de los ids de las unidades con una cesión abierta al fondo (`findVehicleIdsOpenToFund`); en una sentencia y por `id`, las unidades de la empresa con las placas del archivo y esos ids, por la fachada | Las cesiones de esas unidades, de cualquier fondo |
| Crear un fondo | La empresa, `FOR SHARE`, por `resolveScope` de `organization` | Si la empresa está activa (D-84) |
| Editar un fondo o cambiar su estado | El fondo, `FOR NO KEY UPDATE`; al reactivar, la empresa `FOR SHARE`, después del fondo | Si alguna cesión usa el fondo (DP-12) |
| Anular una unidad (`fleet`) | La unidad | Su valor, sus contratos y, por el puerto, su cesión abierta |

- La carga toma el fondo antes que las unidades, como hoy: esta fase no lo introduce.
- Dos `FOR SHARE` sobre el fondo no chocan. La carrera con una edición del fondo en cola la cubre `DeadlockRetry`.
- Una cesión a mano que llega durante la carga espera en el candado de la unidad y ve lo que la carga escribió (FLT-R117). Lo mismo que hoy.
- La lectura de los ids y el candado de las unidades son dos sentencias; hoy son una, con la subconsulta dentro del `FOR NO KEY UPDATE`. Una cesión a mano al mismo fondo de una unidad que no está en el archivo, confirmada entre las dos, queda abierta y la cierra la carga siguiente. Hoy pasa lo mismo con una cesión confirmada después de que empieza la sentencia: la ventana es más larga, no distinta. Es la única sentencia de bloqueo que esta fase parte.

#### La API

- **Opción A:** nada cambia. Rutas, permisos, códigos de error y cuerpos son los mismos.
- **Opción B:**
  - `VehicleResource.allowedActions` pierde `registerFundAssignment`, también en las respuestas de las escrituras de la unidad, y se suma el subrecurso acordado;
  - la ficha sigue releyendo la unidad después de ceder o cerrar, porque `allowedActions.annul` depende de la cesión abierta;
  - se actualizan convenciones §4.2 (la unidad) y §6.2, y el §1 y el §8 del plan.

#### Frontend

- **La carpeta nueva `src/app/financing` recibe de `fleet` 40 archivos con A1 y 43 con A2:**
  - `application/`: `funds.store.ts` y `vehicle-fund-assignments.store.ts`, con sus specs;
  - `domain/model/`: `fund.entity.ts`, `vehicle-fund-assignment.entity.ts` y `commands/{assign-vehicle-fund,change-fund-status,close-vehicle-fund-assignment,create-fund,update-fund}.command.ts`;
  - `infrastructure/api/`: `{change-fund-status,close-vehicle-fund-assignment,create-fund,create-vehicle-fund-assignment,update-fund}.request.ts`, `funds-api-endpoint.ts`, `funds-response.ts`, `vehicle-fund-assignments-api-endpoint.ts` y `vehicle-fund-assignments-response.ts`;
  - `infrastructure/assemblers/`: `fund-assembler.ts` y `vehicle-fund-assignment-assembler.ts`;
  - `presentation/components/`: `assign-fund-dialog`, `close-fund-assignment-dialog` y `fund-form-dialog` (cuatro archivos cada uno), `guarantee-import-dialog` (dos) y `vehicle-fund-assignments` (tres). Con A1, este último se queda en `fleet`, reescrito contra la fachada;
  - `presentation/views/funds-view` (cuatro).
- **Archivos nuevos en `financing`:**
  - `infrastructure/api/financing-api.ts`, que reemplaza en `FleetApi` lo de fondos y cesiones;
  - `infrastructure/api/guarantee-imports-api-endpoint.ts` y `guarantee-imports-response.ts`;
  - `domain/model/guarantee-import.ts`, con `GuaranteeEffects`, `GuaranteeClosures` y `GuaranteeImportResult` (lo que hoy es `FleetImportResult<GuaranteeEffects>`);
  - `application/guarantee-imports.store.ts`, con su spec y su propio `GuaranteeUpload`, con la variante `closures`;
  - `application/financing.facade.ts`, con su spec:
    - `openGuaranteeImport(ImportTarget)`, que hoy está en `FleetFacade` y usa la pantalla Cargas, con su propia copia de `closedOf`;
    - con A1, estos miembros, con tipos definidos junto a la fachada (`UnitAssignmentView`, `UnitAssignmentFilters`, `CededUnit`) y carga y error separados de las pantallas del dueño (regla 1.3):
      - la página de cesiones de una unidad, con lo que hoy usa la pestaña de `assignmentPage`: `items` (como `UnitAssignmentView`), `totalItems`, `loading` y `query`, y `load(request, filters)`, `reload()` y `clear()`;
      - `unitAssignmentsNotice` y `clearUnitAssignmentsNotice()`, `unitAssignmentsError` y `clearUnitAssignmentsError()`. El error de la carga de la página llega por `unitAssignmentsError`, como hoy por `error`;
      - `unitAssignmentsOutdated: Signal<number>`, que sube con cada negativa porque la unidad o sus cesiones cambiaron mientras tanto;
      - `cedeUnit(unit: CededUnit, timeZone: string): Promise<boolean>` y `closeAssignment(assignment: UnitAssignmentView, timeZone: string): Promise<boolean>`, que abren los diálogos de `financing` y responden si se guardó, para que la ficha relea la unidad y devuelva el foco. `CededUnit` lleva `id`, `companyId`, `plate` y `vin`, lo que hoy recibe `AssignFundDialogData` junto con la zona;
  - `presentation/financing.routes.ts`, con su spec, con la forma de las demás: una ruta `''` con `providers: [provideFormFieldDefaults()]` y, dentro, `funds` con `data` `FUNDS_READERS` (`anyOf(PermissionCodes.Funds.VIEW)`) y `title` `Millennium Mobility - Fondos`, que se mudan con su comentario. Sin los `providers`, los campos de la lista se dibujan con la apariencia por omisión y cambian las capturas `funds-*`;
  - `UnitAssignmentFilters`, el tipo propio de filtros de las cesiones de una unidad, que usan la fachada y el store. Hoy el store importa `VehicleRecordFilters` de `fleet/application/vehicles.store`, y desde otro contexto eso viola la regla 1.1.
- **El selector `mm-vehicle-fund-assignments` no cambia**, con A1 ni con A2: lo nombran `e2e/funds.spec.ts` y la spec de la ficha.
- **Archivos que se parten o cambian:**
  - `fleet/infrastructure/api/fleet-api.ts`: salen los extremos de fondos y cesiones;
  - `fleet/infrastructure/api/fleet-imports-api-endpoint.ts` (`GuaranteeImportRequest` y `uploadGuaranteeList`), `fleet-imports-response.ts` (`GuaranteeEffectsResource` y `GuaranteeImportResource`) y `fleet/domain/model/fleet-import.ts` (`GuaranteeEffects` y `GuaranteeClosures`);
  - `fleet/application/fleet-imports.store.ts` y `fleet.facade.ts` (`openGuaranteeImport`), con sus specs. `FleetUpload` pierde la variante `closures`, que solo usa la garantía;
  - `fleet/presentation/views/vehicle-profile-view/vehicle-profile-view.{ts,html,spec.ts}`: deja de importar el store y los componentes de cesiones, y usa `FinancingFacade` (A1) o la pieza prestada (A2);
  - con la opción B: `vehicle-allowed-actions.ts`, `vehicles-response.ts`, `vehicle-assembler.ts` con su spec, y `vehicles.store.spec.ts`;
  - `fleet/presentation/fleet.routes.ts` y su spec: `{ path: 'funds', redirectTo: toFinancingFunds }`, con `const toFinancingFunds: RedirectFunction = ({ queryParams }) => inject(Router).createUrlTree(['/financing/funds'], { queryParams })`, como `toModelsOfBrand` de `vehiclecatalog.routes.ts`. Un `redirectTo` de texto no conserva `?empresa=`: Angular copia solo los parámetros que el texto nombra. Los casos de `fleet.routes.spec.ts` que prueban el acceso a `funds` pasan a `financing.routes.spec.ts`;
  - `app.routes.ts`: `{ path: 'financing', loadChildren: financingRoutes }`;
  - `app-shell/app-navigation.ts`: el enlace de Fondos va a `/financing/funds`, con el mismo grupo y el mismo acceso. Se ajusta `app-shell.spec.ts`, en la lista de fondos y en la comparación de rutas con el menú;
  - `imports/presentation/views/imports-view/imports-view.ts`: abre la garantía con `FinancingFacade.openGuaranteeImport`;
  - `environments/environment.ts` y `environment.development.ts`: `fleetFundsEndpointPath` y `fleetFundAssignmentsEndpointPath` pasan a `financing…`.
- **i18n.** Se conservan las claves `fleet.funds.*`, `fleet.guarantee-import.*`, `fleet.profile.fund-assignments.*` (también sus `errors.*`, aunque su store pase a `financing`), `navigation.funds` y `modules.funds.*`, que son contrato por nombre, como en el backend. Un renombre a `financing.*` es su propio cambio, después de la fase (§5).
- **e2e.**
  - `e2e/screens.ts`: la pantalla `funds` pasa a `/financing/funds`; `vehicle-profile-assignments` no cambia de ruta.
  - Specs: `funds.spec.ts` (las rutas, el selector `mm-vehicle-fund-assignments`, cuándo se ofrece ceder y un caso nuevo: `/fleet/funds?empresa=<id>` abre `/financing/funds?empresa=<id>`, el e2e de la redirección que pide §5), `exports.spec.ts` (la exportación de fondos) e `imports.spec.ts` (la garantía).
  - Mocks: las rutas de fondos y cesiones de `e2e/support/mocks/fleet.ts` pasan a un `mocks/financing.ts`, registrado en `e2e/support/api.ts` respetando el orden de los manejadores de Playwright. `registerFundAssignment` en el mock de la unidad cambia según la opción; `/fund-assignment-imports` sigue en `mocks/imports.ts`.
  - Capturas: `funds-*` y `vehicle-profile-assignments-*` conservan su nombre, porque se nombran por `screen.name`, y **no se regeneran**: ninguna opción cambia lo dibujado. Una que cambie es una regresión y se corrige antes de integrar.
  - Sondas: `e2e-live/funds.live.spec.ts` navega a `/financing/funds`. `imports.live.spec.ts` no cambia, porque usa la API.

#### Pruebas de caracterización, antes de mover nada (paso 0, en `develop`, contra `fleet`)

Van por la API o por SQL, en paquetes que la fase no mueve (la raíz `ky.millennium.mobility.erp` o `reporting`), y pasan sin cambios después de cada paso:
1. La anulación y la cesión de la misma unidad no se cruzan, en los dos órdenes de llegada: la cesión primero (la anulación espera y responde `VEHICLE_FUND_ASSIGNMENT_OPEN`) y la anulación primero (la cesión espera y responde `VEHICLE_ANNULLED`). Es la versión de raíz de `VehicleAnnulmentLockTests`, por la API, que reemplaza en este paso a la de `fleet`: esa importa `VehicleFundAssignmentCommandService` y no compilaría en el paso 4.
2. La carga de garantía y una cesión a mano lanzada mientras la carga tiene su candado (FLT-R117): la de una unidad del archivo espera y responde según lo que la carga escribió; la de una unidad con una cesión abierta al fondo cuya placa no está en el archivo también espera, porque la carga la bloquea.
3. La columna «GARANTÍA MOBILIARIA» del exportable de stock, en las pruebas de `reporting`: `SI` con una cesión abierta; `NO` sin cesión o con la cesión cerrada. Hoy ninguna prueba la cubre.
4. `allowedActions.registerFundAssignment` y `annul`, por la API, de una unidad con una cesión abierta, de una sin cesión, de una cuya cesión termina hoy y de una cuya cesión termina mañana en la zona de su empresa.

Las pruebas de hoy que fijan la regla también caracterizan: `FundsIntegrationTests`, `VehicleFundAssignmentsIntegrationTests` y los casos de garantía de `FleetImportsIntegrationTests` cambian de paquete y de imports, no de casos; `VehicleAllowedActionsTests` y `VehicleAnnulmentsIntegrationTests` no cambian con la opción A.

Con la opción B, el caso de `registerFundAssignment` de la prueba 4 se reescribe contra el subrecurso en el paso 5. Es el único que cambia, por la excepción que declara la opción; el de `annul` sigue igual.

#### Pruebas que cambian

- **Se mueven a `financing`, en el paso 4:** `FundRacedCodeTests`, `GuaranteeListPolicyTests`, `FundAuditTests`, `FundLockTests`, `VehicleFundAssignmentAuditTests`, `FundsExportIntegrationTests`, `FundsIntegrationTests` y `VehicleFundAssignmentsIntegrationTests`. Siguen usando `FleetFixtures` y `AccessFixtures`.
- **Se parten, en el paso 4:**
  - de `FleetImportsIntegrationTests`, los casos de la garantía van a `FundAssignmentImportsIntegrationTests`;
  - de `FleetRowLevelSecurityTests`, los tres casos de fondos y cesiones (lo que ve cada empresa, la unidad y el fondo de la misma empresa, y el período) van a `FinancingRowLevelSecurityTests`;
  - de `FleetRowsAreNeverDeletedTests`, las entradas `funds` y `vehicle_fund_assignments` van a `FinancingRowsAreNeverDeletedTests`.
- **Se reescriben:**
  - `VehicleAnnulmentLockTests`: en el paso 0, como prueba de la raíz por la API (caracterización 1);
  - `VehicleAllowedActionsTodayTests`, el caso de «hoy en la zona de la empresa» (D-40). En el paso 1 prueba `offersCession` del adaptador provisional, con el reloj de `CompanyCalendar` fijo. En el paso 4 pasa a `financing` como `UnitFundAssignmentsTodayTests`, contra el adaptador del puerto, con el reloj de `FinancingCalendar` fijo. Con la opción B, en el paso 5 pasa al servicio del subrecurso;
  - `VehicleAllowedActionsTests`, el caso de la cesión: con la opción A se queda; con la B pasa a `financing` en el paso 5;
  - `VehicleAnnulmentsIntegrationTests`: con la opción B sale la línea de `registerFundAssignment`, en el paso 5.
- **Se corren sin cambios:**
  - `FleetContextFacadeImplTests`: la unidad devuelta con su cesión abierta;
  - `CreditNotesIntegrationTests`: la nota que devuelve una unidad cedida;
  - `RowLevelSecurityMigrationTests`, las de reporting y las de arquitectura.
- **Nuevas:**
  - `UnitFundAssignmentsWiringTests`, en el paso 1: un solo bean, el provisional de `fleet`; desde el paso 4, el de `financing`;
  - las de los miembros nuevos de la fachada de `fleet`, en el paso 2: candado por `id` ascendente, la fila después del candado, la visibilidad por marca y, en `FleetContextFacadeTransactionTests`, que `lockVisibleUnitByPublicId`, `lockVisibleUnitById` y `lockUnitsOfCompanyByPlatesOrIds` fallan fuera de una transacción;
  - la de `findVehicleIdsOpenToFund`, en el paso 2: solo las cesiones abiertas a ese fondo;
  - con la opción B, las del subrecurso nuevo, en el paso 5.

#### Pasos dentro de la fase

Cada paso compila y pasa sus pruebas antes del siguiente: una prueba se mueve, se parte o se reescribe en el mismo paso que el código que importa. La fase se integra completa.

0. **Caracterizar**, en `develop` y contra `fleet`: las cuatro pruebas de arriba, en verde. `VehicleAnnulmentLockTests` pasa a su versión de raíz.
1. **`fleet`:** el puerto, `UnitFundFacts` y un adaptador provisional en `fleet` sobre su propio repositorio y `CompanyCalendar`. `VehicleAnnulments` y `VehicleAllowedActions` pasan por él, y `VehicleAllowedActions` pierde su constructor con `Clock`. El puerto lleva `offersCession` también con la opción B: así `mayCede` deja de leer el repositorio antes de que se mude, y el paso 5 lo borra. En el mismo paso se reescribe `VehicleAllowedActionsTodayTests` y entra `UnitFundAssignmentsWiringTests`.
2. **`fleet`:** los miembros nuevos de la fachada, `lockByCompanyIdAndPlatesOrIds` y `VehicleFundAssignmentRepository.findVehicleIdsOpenToFund`, con sus pruebas.
3. **Todavía dentro de `fleet`:** los servicios de fondos y la carga de garantía dejan de usar `Vehicle`, `VehicleRepository` y `VehicleAccessPolicy`. Leen y bloquean la unidad por los miembros nuevos de la fachada; la carga lee los ids con `findVehicleIdsOpenToFund` y marca la placa con `normalizePlate`. Se borra la consulta que bloquea por placas o por fondo. Ninguna prueba cambia: ninguna construye esos servicios ni llama a la consulta borrada.
4. **Se crea `financing`:**
   - se mueven las 54 clases con sus pruebas, y se parten las piezas con las suyas;
   - se crean las clases nuevas con sus nombres de bean, y las movidas pasan de las piezas de `fleet` que hoy usan (`ExternalOrganizationService`, `ExternalIamService`, `OwningCompany`, `AccountReferenceView`, `FleetSessionAccount`, `CompanyCalendar`, `FleetErrors` y `FleetDocumentKinds`) a las de `financing`;
   - el adaptador del puerto pasa a `financing` en el mismo paso que borra el provisional de `fleet`, y con él `UnitFundAssignmentsTodayTests`. `UnitFundAssignmentsWiringTests` espera el bean de `financing`.
5. **Con la opción B:** el cambio del recurso de la unidad y el subrecurso nuevo, con sus pruebas. `offersCession` sale del puerto y `mayCede` de `VehicleAllowedActions`, y cambian las pruebas que «Pruebas que cambian» ata a este paso.
6. **El frontend**, con sus specs y sus e2e.
7. **La documentación.**

#### Documentación

- `contextos/financing.md`, con lo de fondos de `fleet.md` y los mismos ids (`FLT-R…`), y `contextos/README.md`.
- Una decisión nueva del diccionario que precisa D-200: la carga de garantía mobiliaria vive en `financing`, dueño de `funds` y `vehicle_fund_assignments`; la del impuesto vehicular sigue en `fleet`.
- El mapa vigente (la relación en los dos sentidos, y §11.8, que cita D-200 con la garantía en `fleet`) y el diccionario §3.
- El mapa objetivo §3.8: `documents` (la exportación de los fondos) entre sus dependencias.
- Los patrones de persistencia: la regla de lo que se referencia en modo compartido y la sección nueva de `financing` con la tabla de arriba, que sale de la de `fleet`.
- Procesos y pantallas: Fondos en `/financing/funds`.
- Las convenciones, con cualquier opción: en §2.2, una fila `financing` con `/funds`, `/vehicles/{id}/fund-assignments`, `/fund-assignments/{id}/closure` y `/fund-assignment-imports`, que salen de la de `fleet`, y la línea de las cargas que atribuye `/fund-assignment-imports` a `fleet`; y el punto 18 de §14. Con la opción B, además §4.2 y §6.2.
- El README del backend: `financing` en el árbol de módulos y en la lista de los módulos que exportan, y `fleet` sin los fondos ni la garantía.
- Las cuatro copias de las skills:
  - `backend-module`: §1 (la carga de garantía es de `financing`) y §2 (la lista de `allowedDependencies`);
  - `architecture-review`: `financing` en la búsqueda de la regla 1.2;
  - `frontend-feature`: `financing` en la lista de los contextos;
  - la regla 1.2 del frontend, si se elige A2.

**Terminado cuando:**
- ninguna clase de `fleet` importa un tipo de fondos ni lee `funds` o `vehicle_fund_assignments` por SQL, y `fleet` no nombra `financing`;
- las rutas de fondos, de cesiones y de la carga de garantía responden lo mismo, con los mismos permisos y códigos (con la opción B, salvo `registerFundAssignment`, que pasa al subrecurso);
- `UnitFundAssignmentsWiringTests` encuentra un solo bean, el de `financing`;
- `/fleet/funds?empresa=<id>` abre `/financing/funds?empresa=<id>`, y ninguna imagen de referencia cambió;
- ninguna prueba de caracterización cambió (con la opción B, salvo el caso de `registerFundAssignment`).

### Fase 5 — El servicio publicado de `procurement`

**Por qué.** Hoy `fleet` intermedia los montos de `procurement`. Su puerto `UnitPurchases` define en el lenguaje de `fleet` el costo, las notas, los usos del saldo y los pagos con sus montos. `fleet` sirve `GET /vehicles/{id}/purchase-cost` y le reexpone el costo a `reporting` (`FleetContextFacade.fetchPurchaseCosts`).

El dueño del costo es `procurement`, y debe publicarlo él (*Open Host Service* con *Published Language*, cap. 14): por REST para la ficha, y por `procurement::acl` para `reporting` y, más adelante, para la conciliación de la inicial (D-31). Lo que `fleet` necesita para sus propias reglas sigue llegando por su puerto.

**Decisión previa (fase 0, decisión 7): quién es dueño de la brecha de días.** Un reporte dice lo que dice la ficha (D-232), así que la regla tiene un solo dueño:
- **A (recomendada):** sigue en `fleet` (`DayGapPolicy`, FLT-R131). El puerto conserva el estado de pago y la fecha base, sin montos (D-212). El recurso de la unidad no cambia.
- **B:** `fleet` deja de saber de pagos, y la brecha pasa a `procurement`, nunca a `reporting`. Cambia el recurso de la unidad. Lo que agrega B está al final de esta fase.

**Backend (A).**

- **`procurement::acl`.** Si la fase 3 no la creó, esta fase crea:
  - `procurement/interfaces/acl/package-info.java`, con `@NamedInterface("acl")` y `@NullMarked`;
  - `ProcurementContextFacade`;
  - `procurement/application/acl/ProcurementContextFacadeImpl`.

  Suma un solo miembro, por lote, porque su único consumidor es `reporting`:
  ```java
  /**
   * El costo de compra de cada unidad con línea vigente (D-159, D-199, D-232, D-248), en una lectura por tipo de hecho,
   * en la transacción de quien llama y bajo su Row Level Security. Una unidad sin línea vigente no está.
   */
  Map<Long, UnitPurchaseCost> fetchUnitPurchaseCosts(Set<Long> vehicleIds);
  ```
  Sus records salen de `UnitPurchases`: `UnitPurchaseCost`, `UnitCreditNote` **sin** `usedIn`, `UnitPaymentPosition`, `UnitPaymentRecord` y `UnitAmount`. `UnitBalanceUse` y `usedIn` no se publican: solo los usa la lectura de una unidad, que es de `procurement`.

- **El cálculo** sale de `PurchaseInvoiceUnitPurchasesAdapter` (`costs`, `paymentOf`, `creditNotes`, `usedIn`) a `procurement/application/internal/queryservices/UnitPurchaseCosts`, con dos lecturas:
  - `ofUnit(Long vehicleId)`, con `usedIn` para quien tiene `pagos.ver` en la empresa de la factura;
  - `ofUnits(Set<Long>)`, sin `usedIn`, una lectura por tipo de hecho.

  Responde un tipo propio, `UnitPurchaseCostView` en `procurement/application/queryservices`, con `usedIn` anulable. Nunca responde los records de `fleet::acl`, que fuera de la capa de traducción rechaza `ForeignAclStaysInTranslationLayerTests`, ni los de `procurement::acl`, que no llevan `usedIn`. La ruta lo traduce a su recurso, y `ProcurementContextFacadeImpl` a los records publicados, sin `usedIn`. El adaptador queda solo con el puerto reducido.

- **`UnitPurchases` de `fleet` queda en:** `hasCurrentLine`, `withCurrentLine`, `unitIdsWithCurrentLine`, `valueIsZero`, `returnedUnitIds`, `paymentFacts` con `UnitPaymentFacts(status, baseDate)`, y `fetchDataVersion`.
  - **Salen** `purchaseCostOf`, el método por omisión `purchaseCostsOf` y los records `UnitPurchaseCost`, `UnitPaymentPosition`, `UnitPaymentRecord`, `UnitCreditNote`, `UnitBalanceUse` y `UnitAmount`.
  - `UnitPurchaseFacts` pierde `purchaseCostOf` y `purchaseCostsOf`.

- **`FleetContextFacade`** pierde `fetchPurchaseCosts` y suma un miembro, para que `procurement` no lea `vehicles` (regla 1.2) ni copie la regla de quién ve una unidad:
  ```java
  /**
   * La unidad que quien llama ve con stock.ver en su empresa y su marca (VehicleAccessPolicy.sees), o vacío si no existe o
   * no la ve. Lee bajo el Row Level Security de quien llama, en su transacción.
   */
  Optional<SeenUnit> fetchSeenUnit(UUID vehiclePublicId);
  ```
  El record va en `fleet/interfaces/acl`:
  ```java
  /** @param seesCosts si stock.ver_costos actúa en su empresa y su marca (VehicleAccessPolicy.seesCosts) */
  public record SeenUnit(Long id, UUID publicId, Long companyId, Long commercialBrandId, boolean annulled, boolean seesCosts) {
  }
  ```
- **La traducción en `procurement`.** `procurementExternalFleetService`, el único lugar de `procurement` que puede nombrar `SeenUnit`, suma `fetchSeenUnit(UUID)`, que lo traduce a un record propio de `application/internal/outboundservices/acl`, `SeenVehicle(id, publicId, companyId, commercialBrandId, annulled, seesCosts)`.

- **`GET /vehicles/{id}/purchase-cost`** pasa a `procurement/interfaces/rest/UnitPurchaseCostsController`. No puede llamarse `VehiclesController`, porque choca como bean con el de `fleet`.
  - Lleva `@RequestMapping("/api/v1/vehicles")` y `@PermissionRequired(PermissionCodes.Stock.VIEW_COSTS)`.
  - Su servicio de consultas es `UnitPurchaseCostQueryService`, con `GetUnitPurchaseCostQuery(UUID vehicleId)`, y es `@Transactional(readOnly = true)`: fuera de una transacción, la cuenta de la aplicación no ve filas de ninguna empresa y el costo saldría `null` sin error.
  - Revisa en el orden de hoy (convenciones §7.2):
    1. el `403` de la ruta;
    2. la unidad por `fetchSeenUnit` (`404 VEHICLE_NOT_FOUND`, `ApplicationError.notFound("Vehicle", id)`);
    3. `seesCosts` (`403 ACCESS_DENIED`, con el texto de `FleetErrors.deniedOnUnit`).
  - `returnedToSupplier` es «no anulada» y «devuelta por una nota vigente».
  - `usedIn` viaja solo para quien tiene `pagos.ver` en la empresa de la factura.
  - Conserva los nombres de esquema (`VehiclePurchaseCost`, `VehiclePurchaseAmount`, `VehiclePurchaseInvoice`, `VehiclePurchaseCreditNote`, `VehiclePurchaseCreditNoteUse`, `VehiclePurchase`, `VehiclePurchasePayment`, `VehiclePurchasePaymentRecord`) y la descripción OpenAPI.

- **En el mismo commit sale de `fleet`:** `VehiclesController.getVehiclePurchaseCost`, `GetVehiclePurchaseCostQuery`, `VehicleQueryService.handle(GetVehiclePurchaseCostQuery)`, los traductores de `VehicleQueryServiceImpl`, `VehiclePurchaseCostView`, `VehiclePurchaseCostResource` y, de `VehicleResourceFromViewAssembler`, `toResourceFromView(VehiclePurchaseCostView)` con sus privados `paymentOf` y `amountOf`. Dos métodos con la misma ruta impiden arrancar.

- **Se quedan en `fleet`:**
  - `allowedActions.viewPurchaseCost`, que es la misma regla que `seesCosts` y también usa la pantalla de habilitación para el costo de los equipos;
  - `hasCurrentInvoice`, `returnedToSupplier` y `payment` del recurso de la unidad;
  - el filtro `hasCurrentInvoice` del Inventario;
  - `DayGapPolicy`;
  - `fetchReportFacts`, con su fecha base y su brecha;
  - la versión de la exportación del Inventario, que sigue sumando `UnitPurchases.fetchDataVersion()` por la columna «con factura vigente».

- **`reporting`:**
  - suma `"procurement::acl"` a su `package-info.java`;
  - suma `ExternalProcurementService`, con `@Service("reportingExternalProcurementService")` y `fetchPurchases(Set<Long>)`, que es la traducción a `ReportUnitPurchase` que hoy hace `ExternalFleetService` (este la pierde);
  - `ReportQueryServiceImpl` la usa en la diferencia por inicial y en la exportación de stock;
  - `ReportingDataVersions` no cambia: ya nombra las tablas de compras.

- **Dependencias:** `reporting` → `procurement::acl` es nueva; `procurement` → `fleet::acl` ya existe; `fleet` nunca depende de `procurement::acl`, porque cerraría un ciclo.

- **Transacciones y candados:** todo es lectura, en la transacción de solo lectura de quien llama. No hay candados nuevos.

**Frontend (A).**

- **`procurement`** recibe de `fleet`:
  - `vehicle-purchase-cost.entity.ts`, `vehicle-purchase-cost-response.ts` y `vehicle-purchase-cost-assembler.ts` con su spec;
  - la lectura de `GET /vehicles/{id}/purchase-cost`, que hoy está en `VehiclesApiEndpoint.getPurchaseCost` y `FleetApi.getVehiclePurchaseCost`.

  Un store nuevo, `unit-purchase-costs.store.ts`, guarda el costo con `loadingExternal` y `externalError`.

- **`ProcurementFacade`** suma estos miembros, con tipos definidos junto a ella (nunca la entidad del dominio):
  - `unitPurchaseCost: Signal<UnitPurchaseCostView | null>`
  - `unitPurchaseCostLoading: Signal<boolean>`
  - `unitPurchaseCostError: Signal<string | null>`
  - `unitPurchaseCostDenied: Signal<boolean>`
  - `loadUnitPurchaseCost(vehicleId: string): void`
  - `clearUnitPurchaseCostError(): void`

  No lleva miembros por lote: nadie los usa, porque el Inventario no muestra el estado de pago. Su spec incluye el caso «una lectura ajena no deja su error en la pantalla dueña».

- **`fleet`:**
  - `VehiclePurchaseCostPanel` lee de `ProcurementFacade` y decide «Reintentar» por `unitPurchaseCostDenied`, no comparando una clave de otro contexto.
  - `VehiclePurchaseCostBreakdown`, `breakdowns-catalog-sample.ts` y la spec del panel usan `UnitPurchaseCostView`, `UnitPurchaseAmountView` y `UnitCreditNoteView`, definidos junto a la fachada. Hoy usan `VehiclePurchaseCost`, `VehicleAmount` y `VehicleCreditNote` de la entidad.
  - `VehiclesStore` pierde `purchaseCost`, `purchaseCostError`, `loadPurchaseCost` y `clearPurchaseCostError`.
  - La banda de pago de la ficha y el Inventario no cambian.

- **i18n.** `fleet.purchase-cost.error-load` y `fleet.purchase-cost.error-denied` pasan a `procurement.unit-purchase-cost.error-load` y `procurement.unit-purchase-cost.error-denied`, con el mismo texto y en el mismo commit que el store nuevo que las escribe. El error de una lectura ajena lo nombra el contexto dueño de la lectura, como `procurement-suppliers.errors.load`. Es la excepción de §5 para las claves de un error que cambia de contexto. Las demás claves `fleet.purchase-cost.*` se quedan: las dibuja `fleet`.

- **e2e:** la ruta es la misma, así que los mocks no cambian; las specs de la ficha prueban la fachada.

**Pruebas de caracterización, antes de mover nada (en `develop`, contra `fleet`).** Pasan igual después de cada paso:
1. El JSON completo de `GET /vehicles/{id}/purchase-cost`:
   - de una unidad en dólares con dos notas, un pago parcial y un uso del saldo, leído con y sin `pagos.ver` (`usedIn` presente o `null`);
   - de una unidad sin línea vigente (`purchase: null`).
2. El orden de las negativas:
   - sin `stock.ver_costos` en ninguna empresa: `403`, aunque la unidad no exista;
   - con `stock.ver_costos` pero sin `stock.ver` sobre la unidad: `404 VEHICLE_NOT_FOUND`;
   - con los dos en la empresa pero los costos solo en otra marca: `403 ACCESS_DENIED`.
3. `returnedToSupplier`: devuelta y sin anular, `true`; anulada, `false`.
4. La exportación de stock, la diferencia por inicial y la brecha de una empresa con unidades con notas y pagos, con `stock.ver_costos` en todas sus marcas y en una sola: las columnas de costo y de pago, celda por celda.

**Pruebas que cambian.**
- **Se mudan a `procurement`:**
  - `VehiclePurchaseCostIntegrationTests`, en el paso 4, con la ruta (salvo `viewPurchaseCostFollowsTheCostsOfItsBrand`, que es de `fleet`);
  - los casos de costo de `PurchaseInvoiceUnitPurchasesAdapterTests` (las aserciones sobre `purchaseCostOf` de `theCostOfAUnitComesFromItsCurrentLineAndItsCurrentNotes` y de `theBaseDateMovesToTheFirstPaymentWhenItIsRegistered`): en el paso 2 se escriben contra `UnitPurchaseCosts`, en `UnitPurchaseCostsTests`, y en el paso 6 salen del adaptador.
- **Cambian:** `UnitPurchaseFactsTests`, en el paso 6: su doble `Listed` deja de implementar `purchaseCostOf`, que sale del puerto.
- **Se corren:** `BalanceTraceIntegrationTests`, `SupplierPaymentsIntegrationTests`, `UnitLevelBalanceIntegrationTests`, `UnitPurchasesWiringTests`, `VehicleInvoiceFactsIntegrationTests`, `VehiclesIntegrationTests` (pide la ruta), `ReportsIntegrationTests`, `StockRowsTests`, `ProcurementOpenApiTests` y las de arquitectura (`ModularityTests`, `LayerDependencyTests`, `ForeignAclStaysInTranslationLayerTests`, `NoRoleAnnotationTests`, `PermissionCatalogConsistencyTests`).
- **Nuevas:**
  - `fetchSeenUnit` (marca, anulada, sin ver), en el paso 3;
  - la prueba de contrato de `fetchUnitPurchaseCosts` (§2), en el paso 2: compara con `purchaseCostsOf` del adaptador viejo mientras exista; en el paso 6, cuando sale, fija los valores esperados.
- **Frontend:** las specs del store y la fachada nuevos, del panel, del ensamblador y de la ficha. Los casos de costo de `vehicles.store.spec.ts` pasan a `unit-purchase-costs.store.spec.ts`.
- **e2e:** `fleet.spec.ts`, `credit-notes.spec.ts` y `supplier-payments.spec.ts`. **e2e-live:** `credit-notes.live.spec.ts` y `supplier-payments.live.spec.ts`, cuyo paso 7 recorre «Compra y costo».

**Orden dentro de la fase.** Cada paso compila y pasa sus pruebas antes del siguiente: una prueba cambia en el mismo paso que el código que importa.
1. Las pruebas de caracterización, en verde en `develop`.
2. `procurement::acl`, si falta, y `UnitPurchaseCosts` con `fetchUnitPurchaseCosts`. El cálculo se copia del adaptador, que conserva el suyo hasta el paso 6, y la prueba de contrato compara los dos. Entra `UnitPurchaseCostsTests`, con los casos de costo del adaptador.
3. `fetchSeenUnit` en `fleet`, con su prueba.
4. La ruta en `procurement`, con `fetchSeenUnit` de `procurementExternalFleetService` y `UnitPurchaseCostQueryService`, y su retiro de `fleet`, en el mismo commit. `VehiclePurchaseCostIntegrationTests` se muda con ella.
5. `reporting` lee de `procurement::acl`.
6. Se reducen `UnitPurchases`, `UnitPurchaseFacts` y `FleetContextFacade`, y el adaptador pierde el cálculo. En el mismo paso cambia `UnitPurchaseFactsTests`, los casos de costo salen de `PurchaseInvoiceUnitPurchasesAdapterTests` y la prueba de contrato deja de comparar con el adaptador.
7. Frontend, con sus specs.
8. Documentación y las cuatro copias de las skills (`.claude` y `.cursor` del backend y del frontend).

**Documentación.**
- Una decisión nueva del diccionario que reemplaza D-203: la ruta la sirve `procurement`, que lee la unidad de `fleet::acl`.
- Convenciones: §2.2 (la ruta pasa de la fila de `fleet` a la de `procurement`, y las líneas de `purchase-cost`, `hasCurrentInvoice` y `returnedToSupplier`), §4.2, §7.2 (el orden de `purchase-cost`), §9.2 y §14.19.
- `contextos/fleet.md`: el cuadro de lo que no es suyo, FLT-R53, FLT-R96, FLT-R130, FLT-R146 y la sección de `UnitPurchases`.
- `contextos/procurement.md`: PRC-R62, PRC-R90 y la fachada nueva.
- `contextos/reporting.md`: §1 y §5, con sus dependencias.
- El mapa vigente: §4, §5 y §6, y §11.7, que cita D-203.
- El mapa objetivo ya dice lo que construye esta fase: el puerto reducido (§3.1), la fachada y la ruta de `procurement` (§3.6), `reporting` que lee el costo de `procurement::acl` (§3.14) y sus relaciones (§4). Su texto no cambia.
- `procesos-y-pantallas`: el panel «Compra y costo».
- El README del backend (la línea de `reporting`, que hoy pide los montos a `fleet::acl`) y el Javadoc de `reporting/package-info.java`: el costo de las unidades se lee de `procurement::acl`.
- Las skills: `SKILL.md` §1 y §2 (las dependencias de `reporting`) y `architecture-rules.md` 1.2.

**Terminado cuando:**
- `fleet` no nombra costos, notas ni montos de pagos;
- `GET /vehicles/{id}/purchase-cost` responde el mismo JSON con el mismo orden de negativas;
- los reportes dan las mismas celdas;
- ninguna prueba de caracterización cambió.

**Si se elige B, además:**
- `UnitPurchases` pierde también `paymentFacts` y `UnitPaymentFacts`.
- `DayGapPolicy` y `DayGapPolicyTests` pasan a `procurement/domain/services`.
- `procurement::acl` suma un miembro, con la brecha calculada por `procurement` con la primera entrega que da `fleet` y la zona de la empresa (`America/Lima` si no se lee, como `CompanyCalendar`):
  ```java
  Map<Long, UnitPaymentFacts> fetchUnitPaymentFacts(Set<Long> vehicleIds);

  record UnitPaymentFacts(String status, LocalDate baseDate, long dayGap) {
  }
  ```
- `fleet::acl` suma el primer paso a Activo (FLT-R58), y `SeenUnit` suma `@Nullable Instant firstDeliveredAt`:
  ```java
  Map<Long, Instant> fetchFirstDeliveries(Set<Long> vehicleIds);
  ```
- `UnitReportFacts` y `ReportUnitFacts` pierden `baseDate` y `dayGap`. `UnitReportFactsReader` pierde `UnitPurchaseFacts`, `DayGapPolicy` y `CompanyCalendar`. `ReportQueryServiceImpl.gapOf` y la columna de la brecha de `StockRows` leen de `ExternalProcurementService`.
- Ruta nueva `GET /vehicles/{id}/payment-status` en `procurement`, con `@PermissionRequired(PermissionCodes.Stock.VIEW)`: `404 VEHICLE_NOT_FOUND` si `stock.ver` no ve la unidad, y `200 {vehicleId, payment: {status, baseDate, dayGap} | null}`. Sin versión por lote: el Inventario no muestra el pago.
- `fleet` pierde `VehicleView.payment` (`withRecord(enablement, actions)`), `VehiclePaymentView`, `VehiclePaymentResource`, `VehicleResource.payment` y `VehicleViewFactory.paymentOf`. Es un cambio del recurso de la unidad (convenciones §4.2), de D-212 y de FLT-R131.
- `VehiclesIntegrationTests` cambia sus aserciones de `$.payment`.
- **Frontend:**
  - `Vehicle`, `vehicles-response.ts` y `vehicle-assembler.ts` pierden `payment`, y `vehicle-payment.ts` pasa junto a la fachada;
  - `ProcurementFacade` suma `unitPaymentStatus`, su carga, su error y `loadUnitPaymentStatus(vehicleId)`;
  - la ficha vuelve a pedir la banda cada vez que el store reemplaza la unidad, porque un paso a Activo congela la brecha;
  - `e2e/support/mocks/fleet.ts` saca `payment` del registro y atiende la ruta nueva.
- **Caracterización adicional:**
  - la banda de hoy (`$.payment` de la ficha) es igual a la de la ruta nueva para una unidad entregada, una sin entregar, una con `stock.ver` limitado a su marca y una sin línea vigente;
  - `GET /reports/day-gaps` da las mismas filas.
- **Documentación:** además D-212, FLT-R131 y el mapa objetivo §3.1 y §3.14 (la brecha la calcula `procurement`, y `reporting` la lee).

### Fase 6 — Las cuentas bancarias se quedan en `organization`, con un puerto por consumidor

**Hecha el 19/11/2026** (backend y documentación en `refactor/phase-6`, sobre la fase 1), como la describe esta sección: `BankAccountUsage`, `BankAccountUsageFacts` y `SupplierPaymentBankAccountUsageAdapter` reemplazan a los nombres anteriores, sin cambiar el contrato ni el frontend. Las pruebas que lista, con `CurrencyUsageWiringTests`, pasan: 110 en 15 clases, sin fallos, y `BankAccountLockTests` pasa igual en tres corridas seguidas sobre la misma base. Se corren además las demás pruebas de arquitectura de §2 y el arranque del contexto. Se precisaron cuatro cosas:
- `BankAccountLockTests` vive en `procurement/infrastructure/persistence`, junto a `CurrencyDecimalPlacesLockTests`, porque registra pagos, y `organization` no depende de `procurement` tampoco en sus pruebas. Cada corrida trae una cuenta nueva sin pagos, como las unidades de `InvoiceRaces`; la carrera del pago que llega primero son dos pruebas, la de la corrección y la de la desactivación.
- La corrección y el cambio de estado toman la cuenta `FOR NO KEY UPDATE`, no `FOR UPDATE` como decía la sección de pruebas, que ya lo dice así.
- ORG-R65 pasa a «un registro de otro contexto —hoy un pago—», como pide la documentación de esta fase; «Lo que no cambia» lo nombraba también entre los textos que se generalizan con `collections`, y ya no.
- La prueba de arranque compara la clase de cada bean sin su proxy (`AopUtils.getTargetClass`), como `CurrencyUsageWiringTests`.

**Por qué no se mueven.**
- **Un concepto que van a usar dos contextos hermanos va aguas arriba** (cap. 14). Hoy las cuentas de la empresa solo las usan los pagos a proveedores, pero el bloque 12 trae `collections`, y el mapa objetivo (§3.4) le da la conciliación bancaria y los pagos recibidos. Una cuenta de la empresa también recibe cobros. Si las cuentas fueran de `procurement` (cuentas por pagar), `collections` (cuentas por cobrar) tendría que depender de él para un concepto que no es de pagos, y el mapa (§3.4 y §6) dice que esos dos modelos son opuestos.
- **Son datos de lo que la empresa tiene, no de una operación** (cap. 15). Su lugar es `organization`, en la capa de potencial (mapa objetivo §5), donde ya están la empresa, sus marcas y sus sedes. La cuenta no lleva saldos ni movimientos (D-214).
- **Cada consumidor responde por un puerto que declara `organization`** (cap. 16), como `location` con `CurrencyUsage`. La regla que necesita los datos de otro contexto —ORG-R65: una cuenta con registros ya no cambia su número ni su moneda— se cumple igual, en la misma transacción y bajo el candado de la cuenta, sin mover el agregado.
- **El diccionario ya lo dice:** §3 pone las cuentas en el bloque 3, de `organization` (§6.2.1, `company_bank_accounts`). Si el mapa no coincide con el diccionario, manda el diccionario.

**Qué cambia.** El puerto `BankAccountPayments` pasa a llamarse por lo que pregunta —si una cuenta está en uso— y no por quien hoy lo responde. Así `collections` lo implementa después sin tocar `organization`. `BankAccountPaymentFacts` ya junta una lista de beans, como `CurrencyUsage`; lo que falta es el nombre, el contrato escrito y una prueba de arranque que admita un segundo implementador.

#### Backend — `organization`

- **El puerto.** `organization/interfaces/acl/BankAccountPayments` pasa a `BankAccountUsage`:

  ```java
  public interface BankAccountUsage {
      /**
       * @param bankAccountIds internal identifiers of accounts, such as the ones of a page
       * @return those of them that a record of this context names, current or annulled
       */
      Set<Long> accountsInUse(Set<Long> bankAccountIds);
  }
  ```

  Su javadoc fija el contrato de quien lo implemente:
  - **Responde con el alcance de quien llama**, por la RLS: una cuenta y los registros que la nombran son de la misma empresa (llave `(bank_account_id, company_id)`, V110), así que quien alcanza la cuenta alcanza sus registros. No usa `SYSTEM`, a diferencia de `CurrencyUsage`, que pregunta por una moneda de todas las empresas.
  - **Cuenta también lo anulado:** un pago anulado salió igual de su cuenta y conserva el número y la moneda con que se registró.
  - **Lee una vez por llamada,** en lote: una página de cuentas pregunta una sola vez.
  - **Toda escritura que nombra una cuenta la lee antes en modo compartido**, con `fetchBankAccountForShare` de la fachada, y así la corrección del número o de la moneda, que bloquea la cuenta para actualizar, la ve o la espera. Hoy lo hace el pago; lo hará el cobro de `collections`.
- **El envoltorio.** `application/internal/outboundservices/payments/BankAccountPaymentFacts` pasa a `application/internal/outboundservices/usage/BankAccountUsageFacts`, con `Set<Long> inUse(Set<Long>)`:
  - recibe `List<BankAccountUsage>` y une lo que responde cada bean;
  - sin pedido no pregunta, y sin ningún bean ninguna cuenta está en uso, como hoy.
- **Quienes lo usan.**
  - `BankAccountCommandServiceImpl`: `hasPayments(accountId)` pasa a `inUse(accountId)`. La corrección sigue respondiendo `422 BANKACCOUNT_IN_USE`, un código que ya es genérico; su mensaje dice «pagos» y no cambia en esta fase (ver «El contrato», abajo).
  - `BankAccountViewFactory`: `paymentFacts.withPayments(…)` pasa a `usageFacts.inUse(…)`.
- **Javadoc:** `package-info.java` de `organization` nombra `BankAccountUsage`, que implementan `procurement` y, más adelante, `collections`.

#### Backend — `procurement`

- **El adaptador.** `infrastructure/organization/SupplierPaymentBankAccountPaymentsAdapter` pasa a `SupplierPaymentBankAccountUsageAdapter implements BankAccountUsage`. Responde con `SupplierPaymentRepository.findBankAccountsWithPayments`, que no cambia: los pagos de cualquier estado.
- **Sin otros cambios.** `ForeignAclStaysInTranslationLayerTests` no cambia: la clase implementa el puerto.

#### El riesgo de la lista vacía

`organization` no depende de quien implementa el puerto, así que el arranque no puede exigir que haya un bean: `location` tampoco lo exige con `CurrencyUsage`. Pero si el bean de `procurement` faltara —se renombra o se mueve sin la anotación—:
- ninguna cuenta quedaría en uso;
- ORG-R65 dejaría de cumplirse sin que nada falle: se podría cambiar el número o la moneda de una cuenta de la que ya salieron pagos;
- `hasPayments` diría `false`.

Lo protege una prueba de arranque, no una cuenta de beans:
- `BankAccountPaymentsWiringTests`, de `procurement/infrastructure/organization`, pasa a `BankAccountUsageWiringTests`, en el mismo paquete.
- Exige que la lista **contenga** a `SupplierPaymentBankAccountUsageAdapter`. Hoy exige que sea el único elemento (`singleElement`) y después llama a `payments.getFirst()`: cuando llegue el bean de `collections`, lo primero falla y lo segundo puede probar otro bean.
- Que sin cuentas no se pregunte lo garantiza el envoltorio (`BankAccountPaymentFacts`, que sale antes con un pedido vacío), no el adaptador: lo prueba `BankAccountUsageFactsTests`, y la prueba de arranque deja de llamar al adaptador.
- Es la prueba que §5 pide a un puerto que se responde con una lista de beans. `CurrencyUsage`, el modelo de este puerto, la tiene desde la fase 0: `CurrencyUsageWiringTests` exige sus cuatro implementadores.

#### Lo que no cambia

- **Las tablas y sus migraciones:** `company_bank_accounts` (V108), su RLS, su disparador `keep_company_bank_account_company()` y su línea en `hand-over-database.sql`, la llave de los pagos (V110) y sus casillas en `data_versions`.
- **Las rutas y los permisos:**
  - `GET` y `POST /companies/{companyId}/bank-accounts`, `GET` y `PUT /bank-accounts/{id}` y `PATCH /bank-accounts/{id}/status`, todas con `configuracion.ver_estructura`, y las escrituras con `configuracion.crear_cuentas_bancarias`, `editar_cuentas_bancarias` y `desactivar_cuentas_bancarias`;
  - `GET /supplier-payment-accounts`, con `pagos.ver`.
- **El contrato:** el campo `hasPayments` de la cuenta, los códigos `BANKACCOUNT_*`, las claves de los mensajes y sus textos. Cuando exista `collections`, las convenciones deciden si `hasPayments` se renombra o se suma `inUse`. Con eso se generalizan los textos que hoy dicen «pagos»:
  - el mensaje de `BANKACCOUNT_IN_USE` en `BankAccountErrors` y en los dos bundles («La cuenta tiene pagos: su número y su moneda ya no cambian»);
  - `bank-accounts.errors.in-use` en `es.json` del frontend;
  - la descripción OpenAPI de `hasPayments` en `BankAccountResource`.
- **La regla del número completo (ORG-R66):**
  - el número lo lee solo quien gestiona las cuentas de esa empresa, con uno de sus tres permisos en toda ella;
  - los demás leen los últimos cuatro, y no buscan ni ordenan por el número;
  - lo que sale por la fachada (`BankAccountSummary`) lleva solo `lastDigits`, así que ni los pagos ni otro contexto reciben el número completo.
- **La fachada de `organization`** (`fetchBankAccountForShare`, `fetchBankAccountsByIds`, `fetchBankAccountsByPublicIds` y `fetchBankAccountsOfCompany`) y su uso en `ExternalOrganizationService` de `procurement`.
- **El orden de los candados del pago:** la empresa, las facturas, el proveedor, la moneda, las órdenes y al final la cuenta, `FOR SHARE`, por la fachada de `organization`.
- **Las versiones de datos:** `company_bank_accounts` sigue en `JdbcOrganizationDataVersions.whatTheFacadeResolves()`, que versiona la exportación de los pagos.
- **El frontend:** la pestaña Cuentas bancarias de la ficha de la empresa (`?seccion=cuentas`), sus 20 archivos en `organization`, el formulario de pago y sus pruebas e2e. No cambia ningún archivo.

#### Pruebas

- **Se renombra y se ajusta:** `BankAccountPaymentsWiringTests` pasa a `BankAccountUsageWiringTests`, con «contiene» en vez de «único» y sin `getFirst()`.
- **Nuevas:**
  - `BankAccountUsageFactsTests`, sin Spring: con dos beans de prueba, une sus respuestas; sin beans, ninguna cuenta está en uso; sin cuentas, no pregunta;
  - `BankAccountLockTests`, con las dos carreras de la cuenta. Hoy ninguna prueba cubre su candado compartido:
    - un pago con transferencia que todavía no confirma hace esperar la corrección del número de su cuenta, que después responde `BANKACCOUNT_IN_USE`, y una desactivación de la cuenta espera al pago;
    - al revés, un pago que llega mientras se desactiva su cuenta espera a la desactivación y responde `SUPPLIERPAYMENT_BANK_ACCOUNT_INACTIVE`, sin guardar nada: la corrección y el cambio de estado toman la cuenta `FOR NO KEY UPDATE`, y el pago la lee `FOR SHARE`.
- **Se corren:**
  - `BankAccountTests`, `BankAccountAuditTests`, `BankAccountRowLevelSecurityTests` y `BankAccountsIntegrationTests`;
  - `SupplierPaymentAccountsIntegrationTests` (`hasPayments` y `BANKACCOUNT_IN_USE`), `SupplierPaymentsIntegrationTests`, `SupplierPaymentsExportIntegrationTests` y `SupplierPaymentLockTests`;
  - las de arquitectura (`ModularityTests`, `LayerDependencyTests` y `ForeignAclStaysInTranslationLayerTests`).
- **Frontend:** nada. No cambia ningún archivo.

#### Documentación

- **`contextos/organization.md`:**
  - «El puerto que declara» (sección 5) pasa a `BankAccountUsage`, con su contrato y quiénes lo implementan;
  - la nota de los pagos de una cuenta;
  - ORG-R65 conserva su id y dice «una cuenta que nombra un registro de otro contexto, vigente o anulado —hoy un pago—»; su columna de pruebas nombra `BankAccountUsageWiringTests`.
- **`contextos/procurement.md`:**
  - en la tabla de los puertos que implementa, `BankAccountUsage` en lugar de `BankAccountPayments`;
  - la columna de pruebas de PRC-R164, que nombra `BankAccountPaymentsWiringTests`, pasa a `BankAccountUsageWiringTests`. La regla no cambia.
- **El mapa vigente:** las seis menciones de `BankAccountPayments`:
  - el párrafo de `company_bank_accounts`;
  - las filas de `organization` y de `procurement` en la tabla de dependencias;
  - «`organization` no llama a `procurement`»;
  - las filas de `organization` y de `procurement` en la tabla de las fachadas.
- **El mapa objetivo:** ya lo dice (§3.6, §3.9, §4 y §7.3). Solo cambia la última frase de §4, que dice «`BankAccountPayments`, que pasa a llamarse `BankAccountUsage`» y queda «`BankAccountUsage`», y lleva la marca de lo integrado que pide su encabezado.
- **El diccionario:** sin cambios; §3 y §6.2.1 ya ponen las cuentas en `organization`.
- **Los patrones de persistencia:** una línea en la regla de lo que se referencia en modo compartido: toda escritura que nombra una cuenta la lee `FOR SHARE` por la fachada, también la que traiga `collections`.
- **Las skills:** no nombran el puerto; no cambian.

**Costo y riesgo.** Menos de un día; riesgo bajo. Depende de la fase 0. No cambia comportamiento, contrato ni pantalla.

### Fase 7 — La salida y el regreso del colaborador, en `organization`

**Por qué.** La salida y el regreso son hechos del colaborador (D-38). El contexto dueño de un término es dueño del caso de uso que lo cambia (cap. 14), y el cambio entra por la raíz `Employee` (cap. 6).

Hoy `iam` sirve las rutas y decide quién puede con reglas que son de `organization`:
- el permiso donde está el colaborador (D-122);
- nadie registra la suya (D-70);
- quien opera tiene que estar vinculado a un colaborador activo (ORG-R50).

`EmployeeAllowedActions` ya calcula esas reglas. Además, `iam` le pide a `organization` registrar el hecho por dos miembros de la fachada que solo existen para eso.

Esta fase reemplaza la decisión 2, opción A, del mapa vigente, con una decisión nueva del diccionario.

**Por qué un puerto y no un evento.** El sistema no publica eventos de integración, la negativa del último superadministrador tiene que llegar como `Result` antes de escribir, y la consecuencia sobre la cuenta se confirma o se revierte con la salida. La consistencia síncrona entre dos agregados de dos contextos es una excepción deliberada (cap. 6), que justifica la invariante de seguridad: quien sale no conserva el acceso. Es el patrón que ya usan `UnitPurchases`, `CurrencyUsage` y `BankAccountPayments`, que la fase 6 generaliza a `BankAccountUsage`: el contexto de abajo declara el puerto y el de arriba lo implementa.

**Lo que no cambia:**
- las rutas;
- el permiso `seguridad.registrar_salidas_y_regresos`, donde está el colaborador;
- el cuerpo y la respuesta (`EmployeeTenureResponse`: `id`, `isActive`, `hireDate`, `terminationDate`);
- los códigos y su orden;
- los textos OpenAPI;
- lo que le pasa a la cuenta (IAM-R21, IAM-R40 a IAM-R43);
- el orden de los candados.

**Backend.**

- **Servicio nuevo en `organization`:** `application/commandservices/EmployeeTenureCommandService` y `application/internal/commandservices/EmployeeTenureCommandServiceImpl`, con `@Transactional`. Es el dueño de la transacción de la salida y del regreso. Queda envuelto por `DeadlockRetry`. No va en `EmployeeCommandServiceImpl`, por dos razones:
  - pasa de 300 líneas;
  - `OrganizationContextFacadeImpl` lo inyecta, y un puerto implementado por `iam` en él podría cerrar un ciclo de beans.

- **Una sola entrada.** `handle(TerminateEmployeeCommand)` y `handle(ReactivateEmployeeCommand)` salen de la interfaz pública `EmployeeCommandService`. Hoy solo los llama la fachada; si quedaran, cualquier servicio nuevo de `organization` podría registrar una salida sin pasar por `EmployeeAccountConsequences`, y quien sale conservaría el acceso. La salida y el regreso entran solo por `EmployeeTenureCommandService`, que siempre llama al puerto.
  - Las reglas de hoy —las fechas, la zona, la empresa y la sede `FOR SHARE` del regreso y el período que el regreso cierra (D-241)— pasan de `EmployeeCommandServiceImpl` a un componente de paquete, `EmployeeTenureRules`, en `application/internal/commandservices`, que solo usa el servicio nuevo.
  - Como hoy, `EmployeeTenureRules` vuelve a leer al colaborador `FOR UPDATE`, ya bloqueado: lo leído antes de una elevación a `SYSTEM`, como la del paso 7 de la salida, ya no está en el contexto de persistencia.
  - El servicio recibe los comandos de `organization` que ya existen: `TerminateEmployeeCommand(employeePublicId, terminationDate)` y `ReactivateEmployeeCommand(employeePublicId, hireDate)`. Quien opera sale de `CurrentAccessScope`, no del comando.
  - El javadoc de la clase `EmployeeCommandServiceImpl` deja de decir que `iam` registra aquí la salida o el regreso por la fachada.

- **Controladores:** `EmployeeTerminationsController` y `EmployeeReactivationsController` pasan a `organization/interfaces/rest`, con:
  - sus rutas;
  - `@PermissionRequired(PermissionCodes.Security.REGISTER_EXITS_AND_RETURNS)` en la clase;
  - `@Tag(name = "Employees")`;
  - las mismas descripciones (`OrganizationOpenApiTests` las lee).

  Con ellos van `TerminateEmployeeResource` (`TerminateEmployeeRequest`), `ReactivateEmployeeResource` (`ReactivateEmployeeRequest`), `EmployeeTenureResource` (`EmployeeTenureResponse`) y su ensamblador, armado desde el colaborador. Los de `iam` se borran en el mismo commit: tienen los mismos nombres de bean y las mismas rutas.

- **El puerto `EmployeeAccountConsequences`, en `organization/interfaces/acl`.** Solo usa tipos de Java y de `shared`, así que la clase de `iam` que lo implementa no nombra otro tipo de la fachada (`ForeignAclStaysInTranslationLayerTests`). La implementación anota cada método con `@Transactional(propagation = Propagation.MANDATORY)`: corre en la transacción de `organization` y nunca abre otra. El javadoc va en inglés, como el resto del código:
  ```java
  /**
   * What the exit and the return of an employee do to its account (dictionary §5.3, §5.4, §5.9). Organization declares it
   * and calls it inside the transaction of the exit or the return, after locking the employee; IAM implements it.
   * There is exactly one implementation: without it the application does not start.
   */
  public interface EmployeeAccountConsequences {

      /** Locks the account until the transaction ends: the second lock, after the employee. */
      void lockAccount(Long accountId);

      /**
       * Only when the exit closes the account: the guard of the last SUPERADMIN (IAM-R21), with its advisory lock.
       * Writes nothing.
       *
       * @return LAST_ACTIVE_SUPERADMIN (422), or empty
       */
      Optional<ApplicationError> exitRefusal(Long accountId, Instant now);

      /**
       * After organization recorded the exit: revokes the roles in force of the account in the company with
       * baja_colaborador and, if closesAccount, closes it (a baja from now unless one is in force, every other role
       * with baja_cuenta, the employment track and the sessions; IAM-R40, IAM-R41), signed by requesterId.
       *
       * @return ACCESS_SUSPENSION_CONFLICT (409) when a concurrent block took the period of the baja. It does not mark
       *         the transaction for rollback: the caller, which owns it, does, and returns the error without another
       *         statement
       */
      Optional<ApplicationError> applyExit(Long accountId, Long companyId, boolean closesAccount, Long requesterId, Instant now);

      /** After organization recorded the return: closes the baja in force, if any, signed by requesterId (IAM-R43). */
      void applyReturn(Long accountId, Long requesterId, Instant now);
  }
  ```

- **La implementación en `iam`.** Va en `iam/application/internal/commandservices`, junto a `AccountClosure`, que es de paquete.
  - Su nombre no termina en `CommandServiceImpl`: no es un servicio de comandos, no la llama un controlador y no abre transacción (`MANDATORY`). `DeadlockRetry` repite solo la operación más externa, la de `organization`, y deja pasar sin repetir lo que corre dentro de una transacción abierta.
  - Depende solo de beans de `iam` —`UserRepository`, `AccessSuspensionRepository`, `LastSuperadminGuard`, `AccountClosure` y `SystemAccessScope`—, nunca de su `ExternalOrganizationService`: así no hay ciclo de beans.
  - Lee la cuenta con `findByIdForUpdate` en cada método, porque entrar en `SYSTEM` vacía lo leído, y guarda con la elevación de hoy cuando cierra la cuenta.
  - `AccountClosure.close` pasa a recibir `Long requesterId` en lugar del `User` de quien opera, del que solo usa el id. El desvínculo (`EmployeeAccountCommandServiceImpl`) le pasa `requester.get().getId()`.
  - Si el cierre lanza `AccessSuspensionOverlapException`, `applyExit` la captura y devuelve `ACCESS_SUSPENSION_CONFLICT`. No llama a `setRollbackOnly()`: la marca la pone `organization` (abajo).

- **Un solo bean, sin valores por omisión.** `organization` inyecta `EmployeeAccountConsequences`, nunca una lista: sin implementación, la aplicación no arranca y nunca registra una salida que no cierra la cuenta. `EmployeeAccountConsequencesWiringTests` comprueba que el único bean es el de `iam`.

- **La salida, en este orden:**
  1. Quien opera es `currentAccessScope.current().userId()`. Sin cuenta, `401`.
  2. El colaborador, bloqueado (`findByPublicIdForUpdate`) bajo el alcance de quien opera. Si no está, `404 EMPLOYEE_NOT_FOUND`.
  3. `scope.actsWhere(REGISTER_EXITS_AND_RETURNS, empresa, sede)`. Si no actúa, `403 ACCESS_DENIED`, con el texto de hoy.
  4. Si el colaborador está vinculado a la cuenta de quien opera, `403 OWN_ACCOUNT_FORBIDDEN`.
  5. Si quien opera no tiene un colaborador activo, `403 ACTOR_EMPLOYEE_REQUIRED`. Se lee como hoy: bajo su alcance, sin elevación.
  6. Si el colaborador tiene cuenta, `lockAccount`.
  7. `closesAccount`: el colaborador tiene cuenta, está activo y la cuenta no tiene otro colaborador activo en otra empresa. Se lee en `SYSTEM`, y solo salen identificadores.
  8. Si `closesAccount`, `exitRefusal`: `422 LAST_ACTIVE_SUPERADMIN`.
  9. La salida con las reglas de hoy, ahora en `EmployeeTenureRules`: `EMPLOYEE_ALREADY_TERMINATED`, `EMPLOYEE_TERMINATION_BEFORE_HIRE`, `EMPLOYEE_TERMINATION_DATE_IN_FUTURE` y `BUSINESS_RULE_VIOLATION`. Si falla, no se escribió nada.
  10. Si tiene cuenta, `applyExit`. Si responde `409 ACCESS_SUSPENSION_CONFLICT`, el servicio de `organization` marca su propia transacción para revertir y devuelve el error, así que la salida del paso 9 tampoco queda (abajo).
  11. `200` con `EmployeeTenureResource`.

- **Los tres `403` los arma `organization`**, en `EmployeeTenureErrors` (`application/internal/commandservices`), con los textos de hoy. Hoy son constantes y fábricas de paquete de `AccountAdministrationGuard`, en `iam`, que `organization` no puede usar:
  - `ACCESS_DENIED`, con `ApplicationError.accessDenied`: «The %s of an employee is recorded with seguridad.registrar_salidas_y_regresos where the employee is (D-122)»;
  - `OWN_ACCOUNT_FORBIDDEN`: «Nobody records their own %s: another administrator does it»;
  - `ACTOR_EMPLOYEE_REQUIRED`: «The %s of an employee is recorded once the own account is linked to an active employee».

  El estado `403` lo da `ErrorResponseAssembler` por el nombre del código, y las claves `error.own-account-forbidden.message` y `error.actor-employee-required.message` ya existen. `AccountAdministrationGuard` conserva sus constantes para las operaciones de cuenta.

- **La reversión del `409`: la marca la pone el dueño de la transacción.** Hoy el servicio de `iam` es el dueño: captura `AccessSuspensionOverlapException` y marca su propio estado con `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`. El commit ve la marca local y revierte sin excepción. Con el puerto, el dueño pasa a ser `organization`, y eso cambia quién marca:
  - El `saveAndFlush` que choca con la restricción de exclusión pasa por el proxy transaccional de Spring Data, que participa en la transacción. Al fallar, la deja marcada como global para revertir (`globalRollbackOnParticipationFailure`, `true` por omisión). PostgreSQL, además, ya abortó la transacción: rechaza toda sentencia hasta el rollback.
  - Dentro de un método `MANDATORY`, `currentTransactionStatus()` devuelve el estado del participante, no el de `organization`. Marcarlo solo vuelve a marcar la transacción global.
  - Si `organization` devuelve el `Result.failure` sin marcar su estado, su commit encuentra la marca global sin la local, revierte y lanza `UnexpectedRollbackException` (`AbstractPlatformTransactionManager.commit`, spring-tx 7.0.9, el de Spring Boot 4.1.1). Ningún manejador la traduce: la respuesta sería `500 UNEXPECTED_ERROR`, no `409`.
  - Por eso, cuando `applyExit` devuelve el error, `EmployeeTenureCommandServiceImpl` de `organization` llama a un `discardWrites()` como el de `EmployeeCommandServiceImpl`, que marca su propio estado, y devuelve el error sin otra sentencia. Con la marca local, el commit revierte sin excepción y responde `409 ACCESS_SUSPENSION_CONFLICT`.
  - Es el patrón que ya usa el código. Cuando choca el VIN de una unidad de una factura, `VehicleRegistrations.discardWrites()` marca el estado del método `MANDATORY` de la fachada de `fleet`, y la factura de compra, dueña de la transacción, marca el suyo cuando la fachada le devuelve el conflicto (`PurchaseInvoiceCommandServiceImpl.failed`). `MigrationTransactions` hace lo mismo con el estado de su `TransactionTemplate` cuando el paso falla.
  - Lo protege la caracterización 4.

- **El regreso:**
  1. Los pasos 1 a 6 de la salida.
  2. Las reglas de hoy, en `EmployeeTenureRules`: `EMPLOYEE_ALREADY_ACTIVE`, `EMPLOYEE_REHIRE_BEFORE_TERMINATION`, la empresa y la sede `FOR SHARE` con `COMPANY_INACTIVE` y `BRANCH_INACTIVE`, y `EMPLOYEE_REHIRE_DATE_IN_FUTURE`; y el período que cierra (D-241).
  3. Si tiene cuenta, `applyReturn`.
  4. `200`.

- **Los candados quedan en el orden de hoy:** el colaborador, la cuenta, el candado del último superadministrador y, en el regreso, la empresa y la sede en modo compartido. Por eso `lockAccount` va antes de cualquier validación de `organization`.

- **Salen:**
  - `terminateEmployee` y `reactivateEmployee`, de `OrganizationContextFacade` y su implementación;
  - `handle(TerminateEmployeeCommand)` y `handle(ReactivateEmployeeCommand)`, de `EmployeeCommandService` y `EmployeeCommandServiceImpl`;
  - de `iam`: `EmployeeTenureCommandService` con su implementación, sus comandos `TerminateEmployeeCommand` y `ReactivateEmployeeCommand`, los dos controladores, sus recursos y su ensamblador, y `terminate` y `reactivate` de su `ExternalOrganizationService`;
  - `PermissionReach.actsOnEmployeeAt`, que después de la fase solo usaría `UserTest`: salen sus aserciones de `UserTest.aPermissionActsWhereTheAssignmentsThatBringItApply`, y las demás de esa prueba se quedan.

- **Se quedan en `iam`:**
  - dar acceso (`linkUserToEmployee`, `linkExistingUserToEmployee`) y el desvínculo (`/employees/{id}/account-unlinking`);
  - `ExternalOrganizationService.fetchEmployeeByPublicIdForUpdate`, que usan dar acceso y el desvínculo; lee al colaborador por `fetchEmployeeSummaryForUpdate`, un miembro de la fachada de `organization` que tampoco sale;
  - `AccountClosure`, `AccountReachReader` (los usa el desvínculo), `LastSuperadminGuard` y `AccountAdministrationGuard`, que la salida no usa.

- **`EmployeeAllowedActions` no cambia.** Ofrece la baja y la reincorporación con las mismas tres reglas de los pasos 3 a 5, con una diferencia que hoy existe y la fase conserva: la fila lee en `SYSTEM` si quien opera tiene un colaborador activo, y la operación lo lee bajo su alcance (paso 5). La caracterización 8 la fija.

- **Dependencias:** ninguna nueva. `iam` ya depende de `organization::acl`, y `organization` no nombra a `iam`.

**Frontend.** Nada: las rutas, el cuerpo y los códigos no cambian (`EmployeesStore` y los diálogos de baja y de reincorporación). Solo cambian los comentarios de `employees.store.ts` que dicen que la salida es de `iam` (líneas 97, 282 y 319). El de la línea 35 habla de la cuenta (`IamFacade`) y se queda.

**Las acciones de cuenta de cada fila no entran en esta fase.** `EmployeeAllowedActions` las calcula con `ProtectedAccounts` y `PendingAccountActivations`. Moverlas a `iam` cambia el contrato de `allowedActions` de la lista de colaboradores, la regla «las acciones por fila las calcula el backend» y lo construido el 15/11 (`bb02f4cf`, `11c29a46`): es una fase propia, con su decisión (decisión 14 de la fase 0).

**Pruebas de caracterización, antes de mover nada (contra las rutas de `iam`, sin cambiarlas después).** Las caracterizaciones 2, 3 y 5 y parte de la 1 ya están en `EmployeeTenureIntegrationTests` (24 pruebas), que se mueve sin cambiar sus aserciones:
- en la 1: `nobodyRecordsTheirOwnExitOrReturn`, `aCompanyAdministratorNotLinkedToAnEmployeeRecordsNoExit`, `aCompanyAdministratorDoesNotRecordExitsInAnotherCompany` y `aSuperadminNotLinkedYetRecordsNoExitNorReturn`;
- la 2: `aCompanyAdministratorClosesTheAccountOfASuperadminThatLeaves`;
- la 3 y la 5: las pruebas de lo que la salida y el regreso le hacen a la cuenta.

Se escriben la 4, la 6, la 7 y la 8, y en la 1 los casos por sede y el orden del último superadministrador frente a una fecha inválida. Los `*LockTests` de hoy no tocan la salida.

1. **El orden de las negativas**, en la salida y en el regreso:
   - fuera del alcance, `404`;
   - el permiso solo en otra empresa, en otra sede, o solo en una sede para un colaborador sin sede, `403 ACCESS_DENIED`;
   - su propio colaborador, `403 OWN_ACCOUNT_FORBIDDEN`, también para un superadministrador;
   - sin colaborador activo, `403 ACTOR_EMPLOYEE_REQUIRED`, también un superadministrador sin vincular (D-89) y quien tiene su colaborador en una empresa que sus roles no alcanzan;
   - el último superadministrador antes que una fecha inválida.
2. La salida de una cuenta protegida (con un rol reservado) la registra el administrador de su empresa y la cierra (IAM-R41): nunca responde `ACCOUNT_MANAGED_BY_SUPERADMIN`.
3. **Lo que le pasa a la cuenta:**
   - los roles de esa empresa se retiran con `baja_colaborador`, y los de otra empresa se quedan si trabaja allí;
   - con la última empresa: `baja` desde ahora (sin una segunda si ya hay una en curso), los demás roles con `baja_cuenta` (también los de alcance sistema), las sesiones cerradas, las vacaciones en curso terminadas, lo programado de la línea laboral cancelado y los bloqueos de seguridad intactos.
4. **La reversión del `409`:** el cierre choca con un bloqueo concurrente, la salida responde `409 ACCESS_SUSPENSION_CONFLICT` y no escribe nada: el colaborador sigue activo y sin fecha de salida, y la cuenta conserva sus roles. Cómo se escribe:
   - en el paquete `iam.application.internal.commandservices`, porque `AccountClosure` es de paquete y se reemplaza con `@MockitoBean`, como en `EmployeeAccountUnlinkingRollbackTests`;
   - el doble marca para revertir la transacción en curso y lanza `AccessSuspensionOverlapException`: así la deja el `saveAndFlush` que choca con la restricción de exclusión;
   - **sin la transacción de la prueba** (`@Transactional(propagation = Propagation.NOT_SUPPORTED)` en el método), como `ExchangeRateRacedDuplicateTests`, para que el servicio sea el dueño de su transacción. Con la transacción de la prueba, el servicio solo participa, su commit no lanza `UnexpectedRollbackException` y la prueba pasaría aunque la respuesta real fuera `500`. Por eso no basta con copiar `EmployeeAccountUnlinkingRollbackTests` ni con `EmployeeTenureIntegrationTests`, que son `@Transactional`;
   - la escena —una empresa, quien opera vinculado a un colaborador activo y un colaborador cuya cuenta no trabaja en otra empresa, para que la salida la cierre— se confirma con un `TransactionTemplate` y se reutiliza en cada corrida: la salida revertida la deja como estaba, y la prueba lo comprueba en otra transacción.

   En `develop` pasa porque `iam` marca su propio estado. Después de mover el servicio, pasa solo si `organization` marca el suyo.
5. **El regreso:** cierra solo la `baja`, a nombre de quien lo registra; los roles no vuelven; una empresa o una sede inactiva responde `422` y la cuenta sigue cerrada; guarda su período.
6. **Carreras, con transacciones confirmadas:**
   - dos salidas de la misma cuenta en dos empresas a la vez la cierran una vez, con una sola `baja`;
   - una salida y un acceso al mismo colaborador;
   - las salidas de los dos únicos superadministradores a la vez: una responde `422`.
7. **El orden de los candados:** con la cuenta bloqueada por otra transacción, la salida ya tiene bloqueado al colaborador mientras espera.
8. **`EmployeeAllowedActions` y la operación:** la fila ofrece `terminate` o `reactivate` cuando la operación pasa las tres negativas, salvo un caso que hoy difiere y se fija como está. Quien tiene su colaborador activo en una empresa que sus roles no alcanzan recibe la acción, porque la fila lo lee en `SYSTEM`, y la operación responde `403 ACTOR_EMPLOYEE_REQUIRED`, porque lo lee bajo su alcance. La fase conserva las dos lecturas: alinearlas cambia el comportamiento (D-89, ORG-R50), y queda anotado como pendiente en el diccionario.

**Pruebas que se mueven o se corren.**
- `EmployeeTenureIntegrationTests` pasa a `organization`.
- Las once pruebas de la salida y el regreso de `EmployeeCommandServiceImplTests` (líneas 675 a 875: cinco de la salida y seis del regreso) pasan a `EmployeeTenureRulesTests`.
- La caracterización 4 se queda en el paquete de `iam`.
- En `iam` se quedan y se corren `SecurityOperationsBySystemRoleIntegrationTests`, `CrossCompanyAccountAdministrationIntegrationTests`, `AccessAdministratorIntegrationTests`, `UsersIntegrationTests`, `EmployeeAccountUnlinkingRollbackTests`, `UserTest`, `LastSuperadminGuardTests`, `SuperadministratorsLockTests` y `AccountLockTests`.
- En `organization` se corren `EmployeeCommandServiceImplTests`, `EmployeeAllowedActionsTests`, `EmployeeAllowedActionsIntegrationTests`, `EmployeeTenurePeriodsIntegrationTests`, `EmployeeLockTests` y `OrganizationOpenApiTests`.
- Las de arquitectura y `DeadlockRetryTests`.
- **Frontend:** `e2e/access.spec.ts` y `e2e/employees.spec.ts`. **e2e-live:** `organization.live.spec.ts`.

**Orden dentro de la fase.**
1. Las pruebas de caracterización, en verde en `develop`.
2. El puerto en `organization` y su implementación en `iam`, extrayendo el código de hoy, sin cambiar el comportamiento, con su prueba de cableado. Mientras el servicio de `iam` siga siendo el dueño de la transacción, es él quien marca su estado cuando `applyExit` devuelve el error.
3. El servicio, `EmployeeTenureRules`, los controladores y los recursos en `organization`, y el retiro de `iam`, en el mismo commit.
4. Salen los miembros de la fachada y los dos `handle` de `EmployeeCommandService`.
5. Documentación y las cuatro copias de las skills (`.claude` y `.cursor` del backend y del frontend).
6. Revisión de arquitectura (skill `architecture-review`) antes de integrar.

**Documentación.**
- Una decisión nueva del diccionario que reemplaza la decisión 2, opción A.
- En el diccionario §14 («Decisiones abiertas», con el responsable): la fila de Colaboradores y la operación leen de forma distinta si quien opera tiene un colaborador activo (caracterización 8).
- El mapa vigente: §4, §5, §6, §10 y §11.
- Las convenciones: §2.2 (las rutas pasan a la fila de `organization`), §7.2 (las tablas de la salida y el regreso dicen que revisa `organization`), §9.2 y §14.17.
- `contextos/iam.md`: F9, IAM-R21, IAM-R34, IAM-R40 a IAM-R43, las elevaciones a `SYSTEM` y la tabla de la fachada.
- `contextos/organization.md`: F7, ORG-R22 a ORG-R24, ORG-R50 y el puerto nuevo.
- El javadoc de `organization/package-info.java` nombra `EmployeeAccountConsequences`, que implementa `iam`.
- Las cuatro copias de las skills (`.claude` y `.cursor` del backend y del frontend): `SKILL.md` §4 y `references/architecture-rules.md` dicen que los permisos de colaboradores, `registrar_salidas_y_regresos` entre ellos, cuentan solo con `AccessScope.actsWhere`. Salen `PermissionReach.actsOnEmployeeAt` y `actsOnEmployeeMovedBetween`, que `SKILL.md` cita y no existe.

**Terminado cuando:**
- `iam` no sirve las rutas ni nombra `registrar_salidas_y_regresos`;
- `OrganizationContextFacade` no tiene miembros de salida ni de regreso, y `EmployeeCommandService` tampoco;
- las pruebas de caracterización pasan sin cambios, la 4 sin la transacción de la prueba;
- la revisión de arquitectura no encuentra hallazgos.

**Riesgo alto:** toca la seguridad de las cuentas. Lo cuidan el orden de las negativas, la entrada única, el bean único sin valores por omisión, la reversión del `409` marcada por el dueño de la transacción y las carreras.

### Fase 8 — Los catálogos preguntan en vez de recibir banderas

**Por qué.**
- Hoy `fleet` le escribe a `vehiclecatalog` `has_units` (`markModelsInUse`) y a `equipment` `has_installations` (`markEquipmentTypeInUse`): el proveedor guarda un hecho de su cliente, y es un valor derivado (regla 4.1).
- `location` resuelve el mismo problema preguntando por un puerto (`CurrencyUsage`).
- La pregunta equivale a la bandera porque `vehicles.model_id` y `vehicle_equipments.equipment_type_id` no cambian y ninguna fila se borra. Cuentan las unidades anuladas y las instalaciones cerradas.

**Contextos:** `vehiclecatalog`, `equipment` y `fleet`, que implementa los dos puertos.

#### 8.1 Los puertos

- **`vehiclecatalog/interfaces/acl/VehicleModelUsage`:** `Set<Long> modelIdsWithUnits(Set<Long> modelIds)`. Lo implementa `fleet/infrastructure/vehiclecatalog/VehicleModelUsageAdapter`, con un método nuevo de `VehicleRepository`: `SELECT DISTINCT model_id FROM vehicles WHERE model_id IN (:ids)`.
- **`equipment/interfaces/acl/EquipmentTypeUsage`:** `Set<Long> typeIdsWithInstallations(Set<Long> typeIds)`. Lo implementa `fleet/infrastructure/equipment/VehicleEquipmentTypeUsageAdapter`, con `VehicleEquipmentRepository`: `SELECT DISTINCT equipment_type_id FROM vehicle_equipments WHERE equipment_type_id IN (:ids)`.
- **Los dos son por lote**, porque los lee cada fila de una lista.
- **Se inyectan como bean obligatorio:** cada uno tiene un solo implementador y protege una invariante (D-107, D-185), así que va como dice §5, nunca como una lista. Con una lista vacía, `anyMatch` daría falso, y el tipo de vehículo de un modelo con unidades, o el código de un equipo instalado, se corregirían sin que nada falle.

#### 8.2 Lo que cambia, completo

**Lecturas:**
- `VehicleModel.isVehicleTypeLocked()` y `canChangeVehicleTypeTo(id)` pasan a recibir `hasUnits`, y `EquipmentType.isCodeLocked()` y `canCorrectCodeTo(code)` a recibir `hasInstallations`. Es el patrón de `VehicleDocumentType.isCodeLocked(usedByADocument)` de `fleet`.
- También cambian de firma `VehicleModel.update` y `EquipmentType.update`, que los llaman por dentro (`VehicleModel.java:111`, y `EquipmentType.java:144` de hoy).
- `VehicleModelView` y su fábrica calculan `vehicleTypeLocked` por lote; `VehicleModelResourceFromEntityAssembler:22` lo lee de la vista. `VehicleModelCommandServiceImpl:101` arma hoy `VehicleModelView` sin la fábrica: pasa por ella, o recibe la respuesta del puerto.
- `EquipmentTypeViewFactory` calcula `codeLocked` como «`PROGRAM_CODES` o instalado», por lote.

**Escrituras que se retiran:**
- en `vehiclecatalog`: `markModelsInUse` de la fachada, `MarkVehicleModelsInUseCommand`, `VehicleModelCommandService(Impl).handle(Mark…)`, `VehicleModelRepository.markAllInUse` y la consulta nativa de `VehicleModelJpaRepository.java:57`;
- en `equipment`: `markEquipmentTypeInUse`, `MarkEquipmentTypeInUseCommand`, `EquipmentTypeCommandService(Impl).handle(MarkEquipmentTypeInUseCommand)`, `EquipmentTypeRepository.markInstalled` y `EquipmentTypeJpaRepository.java:60`;
- en `fleet`: `ExternalVehicleCatalogService.markModelsInUse`, `VehicleRegistrations.markModelsInUse` (l.127), `VehicleCommandServiceImpl:153`, `PurchasedUnitRegistrations:160`, `MigratedUnitRegistrations:202-204`, `VehicleEquipmentInstallations:174` y `ExternalEquipmentService.markEquipmentTypeInUse`.

**El mapeo que se retira:**
- `hasUnits` de `VehicleModel`, `VehicleModelPersistenceEntity` y `VehicleModelPersistenceAssembler:28`;
- `hasInstallations` de `EquipmentType`, `EquipmentTypePersistenceEntity` y `EquipmentTypePersistenceAssembler:40`.

**Lo que no cambia:** la API (`vehicleTypeLocked`, `codeLocked`, `VEHICLEMODEL_TYPE_LOCKED` y `EQUIPMENTTYPE_CODE_LOCKED` son los mismos) y el frontend.

#### 8.3 El alcance

Se pregunta siempre con `systemAccessScope.call(...)`, como `CurrencyCommandServiceImpl.java:102`: `vehicles` y `vehicle_equipments` tienen RLS por `company_id`, y la respuesta debe contar todas las empresas. Solo sale un dato del catálogo, nunca las filas.

#### 8.4 Los candados

La bandera que hoy escribe el alta es lo que serializa la corrección con la primera unidad o la primera instalación. Sin ella, la propuesta (decisión 13) es:
- la corrección del tipo de vehículo de un modelo, y la del código de un tipo de equipo, toman la fila **`FOR UPDATE`**, y no `FOR NO KEY UPDATE`, **antes** de preguntar al puerto. Es un método nuevo, `findByPublicIdForKeyUpdate` (`SELECT id … FOR UPDATE`), en `VehicleModelRepository` y `EquipmentTypeRepository`: el `findByPublicIdForUpdate` de hoy bloquea con `FOR NO KEY UPDATE` (`VehicleModelJpaRepository:45`, `EquipmentTypeJpaRepository:78`), que la FK no espera;
- el alta de una unidad o de una instalación pasa por la FK (`vehicles.model_id` y `vehicle_equipments.equipment_type_id`), que toma `FOR KEY SHARE` sobre esa fila y choca con `FOR UPDATE`;
- así el alta confirma antes y el puerto la ve, o espera a la corrección;
- después de preguntar se relee el registro, porque entrar al alcance `SYSTEM` limpió lo leído (`CurrencyCommandServiceImpl.java:99-103`).

La alternativa de `CurrencyUsage` es que los cuatro caminos que escriben (`VehicleCommandServiceImpl`, `PurchasedUnitRegistrations`, `MigratedUnitRegistrations` y `VehicleEquipmentInstallations`) lean el modelo o el tipo `ForShare`. El orden no cambia con ninguna de las dos: primero la unidad y después el catálogo; el catálogo nunca bloquea una unidad.

#### 8.5 Índice

Una migración nueva crea `vehicle_equipments (equipment_type_id)`, para la consulta por lote.

#### 8.6 Pruebas

- **Se reescriben**, para crear una unidad o una instalación en vez de escribir la bandera: `VehicleModelsControllerTests:462,477`, `EquipmentTypesControllerTests:296,309`, `VehicleEquipmentIntegrationTests:494,509`, `FleetContextFacadeImplTests:121` y `VehicleRetiredReferencesTests:83`.
- **Salen:** `VehicleCatalogContextFacadeTests` (l.146), `EquipmentContextFacadeTests` (el caso de la línea 257 de hoy) y la verificación de `EquipmentCommandServiceImplTests:116`, que la fase 2 dejó en `EquipmentTypeRepository.markInstalled`.
- **Se ajustan,** porque el agregado ya no recibe la bandera y el servicio pregunta al puerto, con su mock:
  - `EquipmentTypeLockTests`;
  - los que construyen los agregados con la bandera: `VehicleModelTests`, `EquipmentTypeTests`, `VehicleCatalogTestData:57,63,68`, `EquipmentTypeCommandServiceImplTests:119,137`, `EquipmentModelCommandServiceImplTests:76,146` y `EquipmentTypeApplicabilityPersistenceTests:79-117`;
  - los que afirman el bloqueo: `VehicleModelCommandServiceImplTests:278` y `EquipmentTypeCommandServiceImplTests:173`.
- **Nuevas:**
  - los dos adaptadores con unidades de dos empresas, leídos con el alcance de una sola, como `VehicleEquipmentCurrencyUsageTests`;
  - `VehicleModelUsageWiringTests` y `EquipmentTypeUsageWiringTests`: el contexto arranca con `VehicleModelUsageAdapter` y `VehicleEquipmentTypeUsageAdapter` de `fleet` como los beans de los puertos (§5, §8);
  - dos carreras: corregir contra la primera unidad, y corregir contra la primera instalación.
- **La prueba en vivo de D-185** (`e2e-live/vehiclecatalog.live.spec.ts:582`) sigue pasando. Instala con `serial: null` (l.626): si la decisión 10 retiró `requires_serial`, el trabajo previo de D-260 ya le dio un serial.

#### 8.7 En dos entregas

1. **Antes de integrar,** en `dev`, en una transacción con el alcance de sistema, como las migraciones: `vehicles` y `vehicle_equipments` tienen RLS forzado (V47, V100), que alcanza también al dueño de las tablas, y sin el alcance cada modelo con unidades saldría como diferencia.
   `BEGIN; SELECT set_config('app.access_scope', 'SYSTEM', true); SELECT id FROM vehicle_models m WHERE has_units <> EXISTS (SELECT 1 FROM vehicles v WHERE v.model_id = m.id); SELECT id FROM equipment_types t WHERE has_installations <> EXISTS (SELECT 1 FROM vehicle_equipments e WHERE e.equipment_type_id = t.id); ROLLBACK;`
   Toda diferencia es un cambio de comportamiento y se explica antes.
2. **Entrega N:** §8.1 a §8.6, con la documentación de §8.8. Las columnas siguen (`NOT NULL DEFAULT FALSE`): las altas no fallan y `ddl-auto: validate` no exige mapearlas.
3. **Entrega N+1,** cuando N ya está desplegada en `dev` (App Platform, `instance_count: 1`), su CI está en verde y ninguna copia anterior a N se conecta a esa base, tampoco una local con el `.env` remoto. Durante su despliegue conviven el contenedor de N, que ya no escribe las columnas, y el nuevo. Una migración nueva hace `DROP COLUMN` de `vehicle_models.has_units` y de `equipment_types.has_installations`, con `lock_timeout`. Se ajusta `VehicleCatalogMigrationTests:78`, y el diccionario quita las dos filas (§8.8).
4. **Vuelta atrás:** revertir la entrega N antes del drop exige recalcular las banderas (`UPDATE … SET has_units = EXISTS (…)`, y lo mismo para los tipos), porque dejaron de escribirse.

#### 8.8 Documentación

Va con la entrega N, salvo las filas de las columnas, que salen en la N+1.
- **El diccionario:** §7.3 (`has_units`) y §7.5 (`has_installations`); D-107 y D-185, reemplazadas por una decisión nueva: el catálogo pregunta por un puerto que declara él, y `fleet` lo implementa (§5).
- **El mapa vigente:**
  - la fila de `vehiclecatalog` de §4 (l.136), que dice que `fleet` le avisa;
  - las fachadas de `vehiclecatalog` y de `equipment` en §5.1 (l.172), sin `markModelsInUse` ni `markEquipmentTypeInUse` y con los dos puertos;
  - §6.4 (l.268-274, «Cuándo un modelo ya tiene unidades» y «Por qué no crea un ciclo»), que pasa a decir que el catálogo pregunta por un puerto que declara él, como `CurrencyUsage`, sin ciclo.
- **`contextos/`:**
  - `vehiclecatalog.md`: VCT-R14, los términos «Modelo con unidades» y «Las unidades de un modelo» (l.28, l.49) y la tabla de su fachada (`markModelsInUse`, l.276);
  - `equipment.md`: VCT-R30, que la fase 2 le pasó, el término «Equipamiento con instalaciones» y `markEquipmentTypeInUse` de su fachada;
  - `fleet.md`: FLT-R44 y FLT-R81, que dicen que `fleet` avisa en su transacción.
- **Los patrones de persistencia:** la corrección toma la fila del catálogo `FOR UPDATE`, y el alta que llega durante ella espera en su FK, con la unidad ya bloqueada.
- **El `README.md` del backend** (l.386: `markModelInUse`, D-107 y `vehicle_models.has_units`).
- Las skills no nombran las banderas: no cambian.

### Fase 9 — Renombrar `location` a `reference` (opcional)

**Por qué.** El nombre es parte del lenguaje (cap. 2). `location` no describe monedas, tasas ni tipos de documento, y choca con `Location` de `organization`.

**Si se hace y cuándo.** Si se renombra es la decisión 2 de la fase 0; cuándo, la 16. Lo que cambia según la respuesta:
- **Sí:** la fase va después de las fases 1 a 8 y antes del primer contexto del bloque 12, para que `customers`, `leasing` y `collections` nazcan con `reference::acl`. Si el bloque 12 empezara antes, cada contexto nuevo sumaría su `"location::acl"` y su `ExternalLocationService`, y el renombre crecería.
- **No:** la fase no se hace, y el mapa objetivo y `contextos/` vuelven a decir `location` donde ya dicen `reference`.

En los dos casos no se hace a mitad de otra fase: es el cambio con más archivos y menos efecto sobre el modelo, y toca archivos de casi todos los contextos.

**El tamaño real.**
- **Backend:** 275 archivos del paquete (215 de código y 60 de pruebas). Fuera de él lo importan 23: 7 de código (los tres `ExternalLocationService` y los cuatro adaptadores de `CurrencyUsage`) y 16 de pruebas. `ExternalLocationService` aparece en 64 archivos, `LocationContextFacade` en 21 y los ids `LOC-R` en 38.
- **Frontend:** los 145 archivos de la carpeta `location`, 72 que la importan y 16 que usan las rutas `/location/…`: `app.routes.ts`, `app-navigation.ts`, `app.spec.ts`, `app-shell.spec.ts`, `e2e/screens.ts`, ocho specs de e2e, `e2e-live/location.live.spec.ts` y la ficha del país (`country-profile-view.ts` y su spec).

**Qué cambia.**
- **Backend:**
  - el paquete y el `displayName`;
  - `"location::acl"` en los `package-info.java` de `fleet`, `organization`, `procurement` y `legacyimport` (si todavía existe);
  - los `ExternalLocationService` de `fleet`, `organization` y `procurement` pasan a `ExternalReferenceService`, por la regla `External<Contexto>Service`, con nombre de bean explícito (§5): `fleetExternalReferenceService`, `organizationExternalReferenceService` y `procurementExternalReferenceService`. El de `organization` hoy no lo tiene;
  - las pruebas.
- **Conservan su nombre**, para acotar el cambio: `LocationContextFacade` y su implementación, `LocationDataVersions` y `JdbcLocationDataVersions`, `LocationDocumentKinds` y los ids `LOC-R*` de las reglas. La skill `backend-module` nombra esas clases por el contexto, así que anota la excepción, para que la revisión de arquitectura no la marque.
- **Frontend:** la carpeta `location`, las rutas `/location/…` con sus redirecciones, el menú, `e2e/screens.ts` y las specs.
- **Documentación:** `contextos/location.md`, el árbol del README del backend, el mapa vigente y las cuatro copias de las skills.

**Qué no cambia:**
- las rutas de la API, las tablas, los códigos de los permisos y los nombres de las imágenes de referencia;
- los `@Tag` de OpenAPI (`Countries`, `Currencies`, `Exchange rates`, `Geo Divisions`, `Geo Division Levels`, `Tax ID Types`), los códigos de `LocationDocumentKinds` (`currencies`, `countries`…) y las claves de primer nivel de `es.json`, que no dicen «location»;
- la propiedad `location.exchange-rates.confirmation-threshold` y su variable `LOCATION_EXCHANGE_RATE_CONFIRMATION_THRESHOLD`, que documenta el README: renombrarlas obliga a revisar la configuración de cada entorno (hoy `.do/app.yaml` no la define y rige `0.03`);
- el prefijo del candado consultivo `location.exchange-rates.series:` de `ExchangeRateRepositoryImpl`: durante un despliegue, dos instancias con prefijos distintos no se esperarían al registrar una tasa de la misma serie;
- todo lo que es `Location` de `organization`: la tabla `locations`, `/organization/locations`, `LocationCommandServiceImpl` y la clave `locations` de `es.json`. El renombre no se hace reemplazando «location» en todo el árbol.

**Depende de** las fases 1 a 8. No espera a la fase 10, que no escribe código.

### Fase 10 — Preparar el bloque 12

Sin código. Se responden las preguntas del bloque 12, se registran como decisiones del diccionario y, con ellas, se diseñan los contextos nuevos. No depende de otra fase: corre en paralelo con las demás (§7) y termina antes de construir el primer contexto del bloque 12.

**Las preguntas del negocio.** Vienen del mapa objetivo (§7, punto 8) y de las decisiones abiertas del diccionario (§14, el GPS de D-260). Las responde el responsable; la segunda columna dice a quién consulta. Los dueños de la identidad del cliente y de `advisor_assignments` son decisiones de diseño, más abajo.

| Pregunta | Con quién | Qué cambia según la respuesta |
|---|---|---|
| El tipo de contrato: arrendamiento financiero u operativo, o venta a plazos con reserva de dominio | Contabilidad | El titular de la unidad durante el contrato, el IGV de las cuotas y la opción de compra |
| La evaluación del cliente: propia o de una central de riesgo, y si hay garante | El área comercial | Las consultas externas de `customers` (su *Anticorruption Layer*) y si existe el garante |
| El cronograma: el sistema de cuotas, la mora, los refinanciamientos y los adelantos | El área comercial y contabilidad | El cronograma versionado de `leasing` y la mora de `collections` |
| Quién emite los comprobantes de cada cuota y con qué proveedor de facturación electrónica | Contabilidad | La *Anticorruption Layer* de facturación de `collections` |
| Cuándo procede la resolución y qué pasa con la deuda y con la unidad | El área comercial, contabilidad y el área legal y documentaria | El puerto de la mora y el camino de `fleet` que pide la resolución |
| El regreso automático a Disponible a las 48 horas de la firma: regla del contrato (la oferta vence) o de la unidad (la reserva vence) | El área comercial | Si sigue en `StatusToleranceBreachTask` de `fleet` (D-164) o pasa a `leasing` |
| El GPS al terminar el contrato (D-260): al traspasar la unidad al cliente, se retira y vuelve al inventario, o se va con la unidad | El área legal y documentaria, que lleva el traspaso | Hoy, en En traspaso, el retiro se permite y no se exige. Si se retira, se exige allí, antes de Inactivo, y el GPS vuelve al inventario de `equipment`; si se va, sale del inventario con la unidad. Qué registra `equipment` en cada caso va con la decisión 11 de la fase 0 |

**Las decisiones de diseño.** Las propone quien diseña el bloque 12 y las aprueba el responsable:
- **El servicio publicado de `fleet` para las transiciones comerciales** —reservar, firmar, entregar, resolver y completar—, que llamará `leasing`. Con él se fija qué caminos manuales quedan para una persona y cuáles pide solo `leasing`: hoy «Firma de contrato» (`FIRMA_CONTRATO`) la registra una persona con `stock.vender` (D-218, D-251).
- **El puerto `UnitContracts`, que ya existe** en `fleet::acl` (`hasContracts`) y hoy no implementa nadie. Lo implementa `leasing`. Sus contratos se suman a lo que dice el historial de la unidad, sin reemplazarlo (D-250; `UnitContractFacts` ya une las dos fuentes): las unidades que la carga inicial trajo en Activo no tienen contrato en `leasing`, y sin el historial se podrían anular. Retirar la deducción exige una decisión que enmiende D-250. Cuando `leasing` lo implemente, los javadocs de `UnitContracts` y `UnitContractFacts`, que dicen «sales will implement it», pasan a nombrarlo.
- **El puerto `UnitTitleTransfers` y el traspaso registrado.** `VEHICLE_TRANSFER_REQUIRED` es contrato de la API y no cambia, ni su regla: En traspaso → Inactivo exige el traspaso registrado (D-223, D-239). Cambia lo que lo satisface, y se decide cuál de las dos rige:
  - el traspaso registrado de `fleet` (`PUT /vehicles/{id}/transfer`, con `stock.recuperar`) sigue siendo el registro, y `titletransfer` lo escribe por el servicio publicado de `fleet` al inscribirse;
  - o `fleet` pregunta al puerto, y el registro a mano se retira.

  Con la decisión se fija también el titular: el texto `vehicles.transfer_owner_name` se vincula al cliente (D-223), en la columna `transferred_to_customer_id` que el diccionario reserva en `vehicles` para el bloque 12, con su migración.
- **La identidad del cliente, separada de `iam` (D-38), y su dueño.** Las credenciales, las sesiones y la activación son un modelo con tablas, y no caben en `customerportal`, que no tiene modelo propio. Se decide si son de `customers` o de un contexto genérico nuevo. El diseño resuelve además, sin estirar `users`:
  - cómo firma el cliente lo que escribe desde el portal: hoy `created_by` apunta a `users`, y en `employee_tenure_periods` es obligatorio;
  - qué alcance le da la RLS, que hoy filtra por empresa (`access_scope_reaches(company_id)`) y tendría que filtrar por cliente;
  - qué cadena de seguridad lo autentica, aparte de la de los colaboradores.
- **El dueño de `advisor_assignments`.** El diccionario la pone en el bloque 3b, de `organization` e `iam`, aunque se construya con el bloque 12 (§3, ajuste 2); el mapa objetivo, en `leasing` (§3.2). O una decisión la pasa a `leasing`, o el mapa objetivo la devuelve a `organization`. Hasta entonces manda el diccionario.
- **Los montos del bloque 12.** `leasing` (precio, inicial y cuota) y `collections` (cuotas y pagos) implementan `CurrencyUsage` de `reference`, como `procurement` y `fleet`. Sin eso, LOC-R05 dejaría cambiar los decimales de una moneda que usan los contratos.
- **El *Published Language* del cronograma** que `leasing` entrega a `collections`, y el puerto de `leasing` por el que `collections` responde la mora.
- **La conciliación de la inicial (D-31):** quién la hace y por qué fachada. `procurement::acl` ya publica el costo de cada unidad (fase 5), y hoy `reporting` muestra la diferencia por inicial.
- **Las cuentas bancarias en la cobranza:**
  - `collections` implementa `BankAccountUsage` (fase 6) para sus cobros;
  - todo cobro que nombra una cuenta la lee antes `FOR SHARE` por la fachada de `organization`, como el pago;
  - lee las cuentas de la empresa con un permiso suyo, como el pago con `GET /supplier-payment-accounts` y `pagos.ver` (PRC-R164).

  Se decide con contabilidad si una cuenta recibe, paga o hace las dos cosas. Con eso cambian la definición de `company_bank_accounts` (D-214: «las cuentas con que cada empresa paga»), ORG-R65 y los textos que dicen «pagos» (fase 6, «El contrato»).
- **La bitácora de los contextos nuevos:** cuáles escriben en `audit` y cuáles implementan `ConfidentialChangeLogFieldsPort`.

**Las decisiones vigentes que el diseño respeta:**
- D-31, la diferencia por inicial contra el contrato;
- D-38, toda cuenta de `iam` pertenece a un colaborador;
- D-164, el regreso automático, hoy en `StatusToleranceBreachTask`;
- D-218, D-251 y D-253, las palabras de ventas: el contrato está Vigente, Completado o Resuelto, y nunca se «vende» al firmar;
- D-223 y D-239, el traspaso;
- D-250, el contrato en el historial de la unidad;
- D-260, los equipos de seguridad de una unidad con un cliente;
- §5.11, `advisor_assignments` y el esbozo de `contracts`, que «se define completo en el bloque 12».

**Qué queda escrito al terminar:**
- **En el diccionario:** una decisión con número por cada pregunta del negocio y cada decisión de diseño. La del GPS sale de «Decisiones abiertas» (§14).
- **En el mapa objetivo:** §7, punto 8, cerrado, y §2, §3 y §4 con lo decidido: el dueño de la identidad del cliente, el de `advisor_assignments`, los puertos y las relaciones de los contextos nuevos.
- **Un `contextos/<contexto>.md` por cada contexto nuevo**, con sus reglas.
- **Las familias de permisos del catálogo** y sus pares incompatibles (D-117, D-119).
- **Las rutas de cada contexto nuevo** en las convenciones (§2.2).
- **Para `fleet`:**
  - qué caminos manuales quedan para una persona y cuáles pide solo `leasing`;
  - si el regreso de las 48 horas sigue en `StatusToleranceBreachTask`;
  - qué satisface `VEHICLE_TRANSFER_REQUIRED`.

**Terminado cuando** cada pregunta tiene su decisión con número y cada contexto nuevo tiene su documento, sin código.

Después, cada contexto nuevo se construye con el orden de la skill `backend-module`: `customers` (mínimo), `leasing`, `collections`, `titletransfer`, el dueño de la identidad del cliente si es un contexto aparte, y el portal. Si la fase 9 se hace, va antes del primero (decisión 16 de la fase 0).

---

## 5. Lista de control de cada fase

**Backend**
- [ ] El módulo tiene su `package-info.java` con `allowedDependencies`, y su `interfaces/acl` con `@NamedInterface("acl")` solo si otro módulo depende de él. `ModularityTests` pasa y no hay ciclos.
- [ ] La fachada expone todo lo que usan sus consumidores. Spring Modulith publica de un paquete anotado solo sus propias clases públicas, sin subpaquetes ni los tipos de sus firmas. Una declaración `@NamedInterface("acl")` a nivel de tipo, en cambio, arrastra los tipos del módulo que aparecen en sus firmas públicas (`propagate`, Modulith 2.0). Un tipo del dominio nunca lleva la anotación, porque el dominio no importa Spring (regla 2.1): se anota el tipo de `application` o de `interfaces` cuya firma lo nombra, y el del dominio entra por ella. El dominio tampoco importa `interfaces`.
- [ ] Ninguna clase de otro contexto importa su interior: solo su `::acl`, desde su `External<Contexto>Service` (`ForeignAclStaysInTranslationLayerTests`). `audit::acl`, `imports::acl`, `documents::acl` y `notifications::acl` son contratos de plataforma y se usan sin traductor; cada uno está en `PLATFORM_CONTRACTS` de esa prueba.
- [ ] Cada `External<Contexto>Service` lleva nombre de bean explícito, `@Service("<contexto>External<X>Service")`: hoy ya hay varios con el mismo nombre simple, y uno sin nombre impide arrancar. `ErpMillenniumMobilityBackendApplicationTests` arranca el contexto.
- [ ] Ninguna ruta queda mapeada dos veces: el controlador viejo sale en el mismo commit en que entra el nuevo, y un controlador nuevo no repite el nombre simple de otro.
- [ ] Los controladores movidos conservan su ruta, su `@PermissionRequired`, su `@Tag`, sus descripciones OpenAPI, los nombres de sus `@Schema` y sus códigos de error, con el mismo orden de revisiones.
- [ ] Un puerto que protege una invariante con un único implementador (`UnitFundAssignments`, `EmployeeAccountConsequences` y, en la fase 8, `VehicleModelUsage` y `EquipmentTypeUsage`) se inyecta como bean obligatorio, nunca como una lista con valores por omisión, y tiene su prueba de cableado. Uno que se responde con una lista de beans (`CurrencyUsage`, `BankAccountUsage`) tiene una prueba de arranque que exige a cada implementador. `CurrencyUsage` todavía no la tiene completa: de sus cuatro implementadores, dos no los exige ninguna prueba, y la fase 0 la suma.
- [ ] Los servicios de comandos conservan el patrón `*CommandServiceImpl` en `application.internal.commandservices` (`DeadlockRetry`), y todo `@Scheduled` sigue dentro de `SingleInstanceRun`.
- [ ] Ningún `Jdbc<Contexto>DataVersions` de un contexto de negocio nombra una tabla de otro contexto (`reporting` es la capa de lectura); cada exportación es un `DocumentKind` del módulo que la sirve, y su versión compone la de cada contexto cuyos datos muestra.
- [ ] Un servicio de consultas que lee tablas de empresa corre en una transacción `readOnly`: fuera de una transacción, la cuenta de la aplicación no ve ninguna fila.
- [ ] Las pruebas se mueven con su código, espejando los paquetes. Si la interfaz de un servicio pasa a `interfaces/acl`, su prueba va junto a la implementación que prueba. Las de caracterización de la fase pasan sin cambios.

**Frontend**
- [ ] La carpeta del contexto tiene sus cuatro capas, y la fachada, solo los miembros que alguien usa (no se crea una fachada sin consumidor).
- [ ] Las rutas declaran los mismos permisos; el menú conserva grupo, acceso, título y descripción; las URL viejas redirigen, con un e2e que prueba cada redirección.
- [ ] Se actualizan `e2e/screens.ts`, las specs que construyen las rutas (como `app-shell.spec.ts`) y las de e2e-live que usan las rutas viejas; las imágenes de referencia no se regeneran.
- [ ] Las claves de `es.json` no se renombran en la fase que mueve; un renombre es su propio cambio. La excepción es el error de una lectura que cambia de dueño: lo nombra el contexto dueño de la lectura (como `procurement-suppliers.errors.load`), así que se renombra en la misma fase, con el mismo texto. Hoy es solo el caso de la fase 5, `fleet.purchase-cost.error-load` y `fleet.purchase-cost.error-denied`.

**Documentación**
- [ ] Lo movido pasa al mapa vigente, a `contextos/` y al diccionario §3, en el mismo cambio, y en el mapa objetivo queda marcado «Integrada en la fase n (fecha)».
- [ ] Una decisión del diccionario o del mapa vigente que la fase contradice o precisa queda precisada o reemplazada por una nueva en el mismo cambio, porque si no coinciden manda el diccionario: D-129 y D-200 en la fase 1, D-254 en la 2, D-200 en la 3 y en la 4, D-203 en la 5 y la decisión 2 del mapa vigente en la 7.
- [ ] Las convenciones, si cambió un recurso, una ruta de módulo o la tabla de rutas por contexto.
- [ ] Las cuatro copias de las skills, a mano mientras sean locales (decisión 15), y el README del backend.

---

## 6. Vuelta atrás

Las fases 1 a 7 y la 9 no cambian datos ni esquema: deshacer una es revertir su commit de integración en el backend, en el frontend y en la documentación. Eso deshace solo la última fase integrada; una anterior se corrige hacia adelante, porque las siguientes se construyen sobre ella. Mientras las skills sean copias locales (decisión 15), se restauran a mano.

La fase 8 va en dos entregas:
- la primera deja de escribir dos columnas. Revertirla antes de eliminarlas exige recalcularlas, porque dejaron de escribirse (fase 8 §8.7);
- la segunda las elimina. Se aplica cuando la primera ya corre en `dev` (App Platform, `instance_count: 1`), su CI está en verde y ninguna copia anterior a ella se conecta a esa base, tampoco una local con el `.env` remoto. Durante su despliegue conviven el contenedor de la primera, que ya no escribe las columnas, y el nuevo. No se revierte: borra dos columnas.

El trabajo previo de D-260 no es una fase y no se revierte: se corrige hacia adelante, porque las marcas de equipo dejan `equipment_models.equipment_brand_id` en `NOT NULL`, que el código anterior no escribe (fase 2, «Antes de la fase 2 — D-260», «En dos entregas»).

---

## 7. Calendario orientativo

| Semana | Fases |
|---|---|
| 1–2 | 0, 1 y 6 |
| 2–4 | El trabajo previo de D-260, en sus dos entregas: la segunda, la que borra las columnas, antes de empezar la 2 |
| 4–7 | 2 |
| 7–8 | 3 |
| 8–10 | 4 |
| 11–12 | 5 |
| 12–14 | 7 |
| 14–15 | 8, primera entrega (la segunda, una entrega después) |
| Después de la 2 | El monitoreo del GPS, si se construye entonces: corre las fases siguientes tantas semanas como dure, y la 8 va antes o después de él (§2) |
| Después de la 8 y antes del bloque 12 | 9, si se hace (decisión 16) |
| Después de la puesta en marcha, con la carga inicial cerrada | 3b |
| En paralelo, con el negocio | 10 |

Cada fase empieza cuando termina la anterior, con su costo máximo y semanas de cinco días: una semana que aparece dos veces es la que termina una fase y empieza la siguiente. El trabajo previo de D-260 está en el calendario aunque no entre en los días de las fases (§3), porque la fase 2 no empieza sin él.

---

## 8. Riesgos

| Riesgo | Mitigación |
|---|---|
| Un acoplamiento oculto que solo aparece al mover | `ModularityTests` lo detecta al compilar el módulo; las pruebas de quien consume se corren en cada fase |
| Una fachada que no expone un tipo que usa su consumidor | La lista de control de §5 y `ModularityTests`; cada fase enumera los tipos de su fachada |
| Dos beans o dos rutas con el mismo nombre impiden arrancar | Nombres de bean explícitos; el controlador viejo sale en el mismo commit; `ErpMillenniumMobilityBackendApplicationTests` |
| Un puerto se queda sin quien lo implemente y su regla deja de cumplirse sin que nada falle (D-182 con `UnitFundAssignments`, ORG-R65 con `BankAccountUsage`, la salida del colaborador) | Bean obligatorio con prueba de cableado, o lista con prueba de arranque (§5) |
| `reporting` lee tablas por SQL | Las tablas no cambian de nombre; sus pruebas se corren en las fases 2, 4 y 5 |
| La versión de los datos de una exportación deja de cubrir una tabla y el documento se reutiliza viejo | Cada fase que parte un `DataVersions` compone las versiones de los dos contextos y prueba la exportación después de cambiar un dato de cada uno |
| La fachada de un contexto crece sin control | Un miembro nuevo solo con un consumidor que lo use (skill `backend-module`, paso 12) |
| Conflictos con trabajo funcional en paralelo | Una fase a la vez, ramas cortas, sin trabajo funcional en el área que se mueve; las fases 4 y 5, en ese orden |
| El monitoreo del GPS y la fase 8 cambian las mismas clases | La fase 8 va antes o después del monitoreo, nunca durante; mientras dura, las fases 3 a 7 no tocan `equipment` ni la instalación (§2) |
| La fase 2 espera a D-260, y D-260 a dos preguntas abiertas del diccionario | La decisión 10 se resuelve en la fase 0, y el trabajo previo de D-260 tiene su lugar en el calendario (§7) |
| Las skills y el mapa quedan atrás del código | Se sincronizan en la fase 0 y se actualizan en cada fase, en las cuatro copias. Son locales y la rama no las lleva: se actualizan a mano el día que se integra, o se versionan (decisión 15) |
| `dev` se despliega antes del CI | El CI corre en la rama antes de integrar a `develop` |
| El backend y el frontend se despliegan por separado, con minutos de diferencia | Primero el backend cuando el contrato no cambia; en tres pasos cuando cambia (§2) |
| Las fases que cambian el recurso de la unidad (la 4 o la 5, solo con su opción B) | Se integran en tres pasos (§2), con las convenciones §4.2 acordadas en orden |
| La fase 7 debilita una regla de seguridad | Pruebas de caracterización antes de mover, bean único, prueba de reversión, carreras y revisión de arquitectura antes de integrar |

---

## 9. Lo que corrigieron las auditorías

### La auditoría del 19/11/2026 (versión 3)

Cinco auditorías en paralelo verificaron la versión 2 contra el mapa objetivo, contra el código de `develop` y contra la rama de D-260: la cobertura del mapa; las fases 0, 1 y 3; la fase 2, D-260 y la fase 8; las fases 4 y 5; y las fases 6, 7, 9 y 10. Dieron 119 hallazgos: 2 bloqueantes, 58 importantes y 59 menores. Esta versión aplica los que se confirmaron contra el código, y lo que pide una respuesta del responsable pasó a la fase 0 (decisiones 9 a 16). Lo principal:
- **El trabajo previo de D-260** tiene su propia sección, antes de la fase 2: la rama corregida, la numeración de las migraciones, el modelo de la configuración, el único lugar donde `fleet` cruza lo que aplica con lo que se exige, las marcas de los estados, el paso automático por `PIERDE_REQUISITO`, la API, el frontend y las pruebas que cambian de resultado. Va en dos entregas, las dos antes de la fase 2, y tiene su lugar en §3 y en el calendario.
- **La fase 2 mueve también las marcas de equipo** de la rama (`EquipmentBrand`, `equipment_brands`, V139). Sin ellas, `EquipmentModel` habría quedado en `equipment` leyendo el interior de `vehiclecatalog`, y `ModularityTests` fallaba. Su inventario se vuelve a tomar después de D-260.
- **La fase 3** tiene un contrato completo de `procurement::acl`: los tipos de la petición, los rechazos con su línea y su campo, también los de `fleet`, y el orden de los candados del pago.
- **La fase 4** empieza caracterizando, y la redirección de las rutas viejas conserva sus parámetros.
- **La fase 7** devuelve el `409` de un bloqueo concurrente sin convertirlo en `500`: la marca de reversión la pone la transacción de `organization`.
- **La fase 8** inyecta sus dos puertos como bean obligatorio, con su prueba de cableado, y su consulta de control corre con el alcance de sistema, porque las tablas tienen RLS forzado.
- **La fase 10** parte de los puertos que ya existen: `UnitContracts` se suma al historial (D-250), no lo reemplaza.
- **El despliegue:** backend y frontend se despliegan por separado, así que §2 fija el orden de integración. Hay una sola instancia (`instance_count: 1`), y la segunda entrega de la fase 8 se condiciona a eso (§6).
- **El mapa objetivo:** las capas de §5 ya no contradicen sus relaciones; §4 suma las relaciones que el plan convierte en `allowedDependencies`; `legacyimport`, `equipment` y `shared` coinciden con lo que el plan construye.

### La auditoría del 18/11/2026 (versión 2)

Cada fase se verificó contra el código: las clases, los consumidores, las fachadas, los candados, las pruebas y el frontend. La versión 1 del plan no se habría podido ejecutar tal como estaba: tenía **17 errores bloqueantes** —que rompían la compilación, `ModularityTests`, el arranque o una regla de seguridad— y más de 30 de severidad alta. La versión 2 los incorporó. Lo principal:

| Fase | Lo que estaba mal | Cómo queda |
|---|---|---|
| 0 y todas | Las skills eran «tres copias»; son cuatro y ya divergían. `ForeignAclStaysInTranslationLayerTests` tiene los contratos de plataforma escritos a mano | Se sincronizan en la fase 0; la fase 1 suma `audit` e `imports` a la prueba |
| 1 | `imports` depende de `audit`, y las dos fachadas no exponían lo que usan sus consumidores (Modulith no expone subpaquetes ni los tipos de las firmas) | `audit` primero; `imports` con `{"audit::acl", "documents::acl"}`; las fachadas, con la lista completa de tipos |
| 2 | El inventario lee empresas y ubicaciones de `organization` y la placa de la unidad de `fleet`; el catálogo de equipos usa el interior de `vehiclecatalog`; la versión de la habilitación mezcla tablas de los dos catálogos; un bean sin nombre impediría arrancar | `equipment` depende de `organization`; la placa llega por un puerto que implementa `fleet`; `vehiclecatalog::acl` suma los tipos de vehículo; las versiones se parten y se componen; nombres de bean explícitos |
| 3 | Los writers de compras usaban las filas del libro que la fase sacaba de `procurement`, y `procurement::acl` necesitaba más de tres miembros | Una capa anticorrupción de verdad: `procurement::acl` habla su lenguaje, `legacyimport` traduce, y el registro de la factura sigue llamando a `fleet` para no invertir el orden de los candados |
| 4 | Faltaban dependencias de `financing` (`iam::acl`, `audit::acl`); `fleet` lee por SQL la tabla que se muda; el recurso de la unidad cambiaba sin decirlo | Dependencias completas; miembros nuevos en la fachada de `fleet`; la opción A no cambia el recurso |
| 5 | La brecha de días se perdía en la ficha o se duplicaba; `procurement` no tenía cómo resolver la unidad | La brecha sigue en `fleet` (opción A); `fleet::acl` suma la unidad que la sesión ve |
| 6 | Mover las cuentas bancarias a cuentas por pagar contradecía el propio mapa: la cobranza también las usará | Se quedan en `organization`, con un puerto por consumidor (`BankAccountUsage`), el patrón de `CurrencyUsage` |
| 7 | Las negativas estaban mal: con el plan, un administrador de empresa no podría registrar salidas y quien sale con un rol reservado conservaría el acceso | Las tres negativas son de `organization`; el puerto de `iam` solo bloquea la cuenta, aplica la guardia del último superadministrador y las consecuencias |
| 8 | La escritura de las banderas es lo que hoy serializa una corrección del catálogo con la primera unidad; un puerto que solo lee reabría la carrera | La corrección toma la fila `FOR UPDATE` antes de preguntar, el puerto lee con el alcance de sistema y va por lote |
| 9 | Subestimada | 6–10 días, con su tamaño real |

### Precisiones sobre el libro

- *Customer/Supplier* es, en el libro, una relación entre equipos con requisitos negociados y pruebas automáticas de aceptación en la frontera (cap. 14). Con un solo equipo se usa para decir quién define y quién traduce, y se sostiene con una prueba de contrato por fachada o puerto (§2).
- El libro no tiene un patrón *Partnership* (aparece en escritos posteriores de Evans). La dependencia en los dos sentidos entre `fleet` y `financing` se escribe como dos relaciones: `financing` aguas abajo de `fleet` (las unidades y su candado) y `fleet` que pregunta por un puerto (la cesión abierta).
- La consistencia sincrónica entre agregados de dos contextos —la cesión y la anulación (D-182), la salida y la cuenta, el dispositivo y su instalación— es una excepción deliberada al consejo del cap. 6, que suele preferir la consistencia eventual entre agregados. La justifica una invariante que no admite una ventana (una unidad cedida no se anula, quien sale no conserva el acceso, un dispositivo está en una sola unidad), y la permite una sola base de datos con un orden fijo de candados.
- En la fase 7, lo que se mueve es el caso de uso —el servicio de aplicación (cap. 4)—, que pertenece al contexto dueño del término (cap. 14); el cambio ya entraba por la raíz `Employee`.
- `reporting` se conforma con lo que publican los demás, pero además lee sus tablas por SQL, una integración por base compartida: su contrato lo sostienen sus pruebas, y más adelante puede pasar a vistas publicadas por cada contexto.
