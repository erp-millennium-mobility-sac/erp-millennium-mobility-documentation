# Plan de cierre de la auditoría del 25/10/2026

**Alcance:** proveedores, plantillas de orden de compra y fondos, de punta a punta en backend y frontend, más la integración de `feat/fleet-funds` en `develop` (backend `229de317`, frontend `40338c7`, sin publicar).

**Fuentes:** la auditoría verificada (46 hallazgos y 10 divergencias), siete planes por frente y dos críticas independientes (secuencia y despliegue, y adversarial).

**Estado:** versión 2 (26/10/2026). Esta versión incorpora la auditoría crítica del plan (sección 11). **La sección 11 manda sobre todo lo anterior y sobre los diseños de detalle (JSON) de los planificadores.** El responsable valida las decisiones de la sección 2 antes de empezar cada fase que depende de ellas. El diccionario ya está en v1.30 sin commit: D-133 tiene D1, D3, D4, D5 y D6, y D-134 los demás textos de DOC-DICT (sección 11.6), salvo los de D7, que esperan al segundo pase (sección 12). **Al 29/10/2026** todo el plan está publicado, también la documentación, desde el 27/10/2026 (secciones 12 y 13); lo que vino después, del 28 al 31/10/2026, está en la sección 13.6.

---

## 1. Criterios con los que se armó el plan

1. **Primero lo que expone datos, y sin esperar al resto.** El bloqueante (proveedores de otra empresa) y la fuga del PDF salen en un carril urgente desde `origin/develop`. Cada uno lleva su propia prueba, que falla antes del arreglo. No esperan a fondos ni a la red de pruebas.
2. **Una red de pruebas real antes de tocar lo compartido.**
   - Los 53 tests del backend en rojo no esconden regresiones. Pero hay unas 12 aserciones que hoy pasan sin probar nada (403 que salen por un código retirado, no por la regla).
   - Todo cambio en `AccessScope` u otra pieza de `shared` espera a que la red esté en verde.
   - Mientras tanto, cada push se compara contra las 53 fallas conocidas, no contra «todo en verde».
3. **`develop` despliega en cada push.** Cada cambio de contrato se hace en el orden expandir → migrar clientes → contraer. Nunca se publica un backend que quite algo que el frontend en producción todavía usa, ni al revés.
4. **La V75 se enmienda, no se parcha.** Nunca salió de local, así que `company_id`, los `CHECK` y `data_versions` de fondos van en la misma V75. La condición: es la primera migración que entra en cada base, y se resetean las bases que ya la tienen.
5. **Un dueño por archivo y por contrato.** Las críticas encontraron contratos FE/BE contradictorios entre planificadores y slices repetidos. Este documento fija una sola versión (sección 6).
6. **Lo más barato que cumple la regla.** Donde había dos diseños correctos se eligió el de menos piezas: RLS de direcciones derivada del país, `removedDocuments` en el PATCH, un solo `TaxIdTypeRules` en procurement.

---

## 2. Decisiones del responsable

Las D1–D10 ya están decididas. Las de esta tabla salieron al planificar. Cada una dice cuándo hace falta y qué recomiendo.

| # | Decisión | Recomendación | Por qué | Antes de |
|---|---|---|---|---|
| DP-1 | `fund_type`: ¿lista cerrada o texto libre? | Lista cerrada **en minúsculas**: `fideicomiso`, `fondo_propio`, `banco`, con `CHECK` en la base | Todas las listas cerradas del proyecto van en minúsculas (`tax_id_types.applies_to`, motivos de V28, V35, V47), y las claves de `es.json` del frontend ya son esas tres. Con texto libre conviven «Fideicomiso» y «FIDEICOMISO» y los reportes por tipo se parten | F2 (congela la V75) |
| DP-2 | D7: códigos finos y matriz | `compras.crear/editar/desactivar_plantillas` (plantillas y sus tipos) y `compras.crear/editar/enviar/cumplir/anular_ordenes`. Matriz: SA y AE con los ocho; OP con los cinco de órdenes; CO solo `ver` | Conserva exactamente lo que cada rol hace hoy, así que el despliegue no quita nada. Enviar y cumplir son «acciones especiales»: comprometen ante un tercero y congelan el documento | F5 |
| DP-3 | D7: ¿la migración remarca los roles propios que tengan el código grueso? | Sí, como excepción anotada a D-130 | V62 no lo hizo, pero aquí el grueso existe en producción desde el 23/10. Sin esa excepción, esos roles pierden la escritura en silencio al desplegar. No hay pares de incompatibilidad de compras | F5 |
| DP-4 | PDF de una orden enviada | **Una sola copia fiel con el teléfono de Atención**, para todo el que ve la orden. Se anota la excepción a D-105 | El documento ya salió hacia el proveedor con el teléfono, así que ocultarlo adentro no protege nada. Hoy, además, lo que recibe el proveedor depende de quién pulsó «Enviar». La alternativa (dos copias) duplica el almacenamiento y confunde la auditoría | F4 (BE-PROC-SEC-06) |
| DP-5 | Emisor de la plantilla | **Se copia de la empresa en el servidor al crear y no se edita.** Se corrige la tabla de §9.2 (l.2293-2296), que dice «se puede editar» | La regla de §9.2 (l.2327) y tu propia pantalla («Los de la empresa. No se editan en esta hoja») ya lo dicen. Cierra el 16 del emisor y el 31 sin códigos nuevos, y elimina el sondeo de 4 s | F4 (BE-TPL-08) |
| DP-6 | `attention_email` | Dejarlo vacío y pasarlo a candidato del bloque 7 | Hoy no tiene fuente: el colaborador no tiene correo. Tampoco tiene campo en la pantalla | F4 |
| DP-7 | WebP y PDF como logo o firma | **Conservar WebP**: leerlo con TwelveMonkeys y validar los bytes por su firma. Un logo o firma en PDF sigue sin imprimirse en la orden, como dice §9.2; la ficha avisa «Un PDF no se imprime en la orden» | Registraste WebP el 23/10 y puede haber WebP guardados en producción: retirarlo no arreglaría esos PDF. La omisión del PDF es del diccionario, pero hoy es silenciosa | F1 |
| DP-8 | RLS de `addresses` (D-80) | **Política derivada del país, con rama SYSTEM:** `current_setting('app.access_scope', true) = 'SYSTEM' OR EXISTS (SELECT 1 FROM companies c WHERE c.country_id = addresses.country_id)`, en `USING` y en `WITH CHECK`. Sin columna nueva | Con una empresa por país (`companies_country_id_uq`, D-5) y solo proveedores locales (D6), equivale a acotar por empresa. La rama SYSTEM hace falta porque el alta de una empresa crea su dirección antes de guardarla, y la primera empresa de un país no la vería. Se anota como excepción a la regla 5.1 en §2.5, D-80 y `location.md`. Si D-5 o D6 cambian, se pasa a `addresses.company_id` (candidato) | P5e |
| DP-9 | Regla 1.2 del frontend | Publicar `organization/presentation/public/company-filter` (`mm-company-filter`), con estado propio por instancia y alimentado por `OrganizationFacade.loadCompanyOptions()` | La sirven compras y fondos. Publicar el `company-select` actual seguiría escribiendo en el store de Empresas (regla 1.3) | F3 |
| DP-10 | Ficha de la plantilla como hoja impresa | Aprobar la revisión 25 de DESIGN, en una fase posterior | El diseño de hoja impresa es tuyo y deliberado. El hallazgo 30 se cierra antes, decidiendo por `allowedActions`, sin esperar el rediseño | F6 (opcional) |
| DP-11 | ¿Quién cede una unidad a un fondo? | Toda la empresa o la marca de la unidad, no la sede. Se anota como excepción a D-101 | Ceder es un acto financiero de la empresa. El código ya lo hace así; falta escribirlo | F3 |
| DP-12 | Código del fondo | Corregible mientras ninguna cesión lo use (`codeLocked`, §2.4), **de punta a punta**: backend (BE-FUN-09) y frontend (FE-FUNDS-11: campo editable con bloqueo, clave de `FUND_CODE_LOCKED`, mock e2e) | Es lo que hacen todos los catálogos. Si `PF_MILLENIUM` se escribe mal, hoy queda así para siempre | P4 |
| DP-13 | HandOver: dos tests imposibles | Borrarlos | Verifican la semilla de V57 y V59 en un estado que no puede volver; los cubre `PermissionsMigrationTests` | F1 |
| DP-14 | Empresas inactivas en los selectores de lista | Las ve quien tiene `configuracion.ver_estructura`, como dejó 7e0cbd6. Se corrigen los specs | El código ya decidió y los specs quedaron atrás | F1 |
| DP-15 | Exportación de proveedores | Sin columna de contactos por ahora | Se suma si se pide | F4 |
| DP-16 | Llave de los archivos guardados (§2.7 y regla 6.4) | Llave por contenido (sha256) para los archivos de la plantilla: se escribe el objeto nuevo antes de confirmar y la fila lo apunta al confirmar. El PDF enviado (`sent.pdf`, llave fija y un solo escritor) queda como excepción anotada | Un rollback o un reintento solo deja un objeto sin referencia; nunca reemplaza los bytes de una fila confirmada. Requiere el texto de §2.7 y un matiz en la regla 6.4 (dos copias de las skills) | P5c |
| DP-17 | D-116 para plantillas, tipos y órdenes | Exportar la lista de órdenes de compra y la de plantillas en el paso de reportes; anotarlo ahora en D-116 como pendiente con fecha | D-116 dice que toda lista principal se exporta. D10 solo decidió proveedores y fondos. No hacerlo sin anotarlo sería una divergencia silenciosa | DOC-DICT |
| DP-18 | ¿Escribir exige leer? | **Sí:** las rutas de lectura de compras piden `compras.ver`; la matriz no deja marcar un permiso de escritura de compras sin `compras.ver` en el mismo rol, y la lectura que sigue a una escritura devuelve el agregado que el comando guardó, sin filtrar | Evita dos regímenes de lectura (la unión de permisos reabría la fuga de la exportación) y los 404 después de guardar | F1a |

**Acciones tuyas en el entorno** (nadie más puede hacerlas):

1. **No arranques el backend desde el repo principal hasta F2.** Si ya lo arrancaste después del merge, tu `erp_mm_db` tiene la V75 vieja: hay que borrar `vehicle_fund_assignments`, `funds` y la fila 75 de `flyway_schema_history`, o recrear la base.
2. **Consultas de solo lectura en producción**, con `SET LOCAL app.access_scope='SYSTEM'` dentro de una transacción:
   - (a) roles propios con `compras.gestionar_plantillas` o `compras.gestionar_ordenes` vigentes;
   - (b) plantillas con `supplier_id` no nulo;
   - (c) `suppliers.address_id` repetidos;
   - (d) proveedores cuyo tipo de documento no es del país de su empresa, no aplica a empresas o no respeta su longitud;
   - (e) `SELECT mime_type, count(*) FROM purchase_order_template_documents WHERE file_path IS NOT NULL GROUP BY 1`;
   - (f) direcciones cuyo país no tiene empresa.

   Te doy las seis consultas listas para copiar.
3. **Opcional:** crear `erp_mm_db_verify_mig`, una segunda base de verificación, con los mismos roles. Permite que la rama de migraciones se pruebe en paralelo. Sin ella, las ramas con migraciones se serializan.
4. **El compañero:** que no commitee más sobre `feat/fleet-funds`. En F2 resetea su base (V66 = fondos) y su rama se adelanta hasta `develop`.

---

## 3. Reglas de ejecución

- **Ramas y lugar de trabajo:**
  - Cada rama `feature/<contexto>` vive en un worktree hermano: `mm-<tema>-be`, `mm-<tema>-fe`.
  - Nunca se compila en los repos principales, porque tu app y tu `ng serve` corren desde ahí.
  - En los worktrees del frontend, `node_modules` va por junction.
  - El e2e corre en el puerto 4300.
  - Las sondas en vivo van en el 8090, contra `erp_mm_db_live` si existe.
- **Pruebas:**
  - Backend con `DATABASE_NAME=erp_mm_db_verify`, con las cuentas de la aplicación y de Flyway.
  - Hasta que entre la red (F1b), se compara contra las 53 fallas conocidas.
  - Se corren las pruebas afectadas, y la suite completa antes de cada push.
- **Commits:** Conventional Commits en inglés, uno por slice, y cada commit compila.
- **Migraciones:**
  - Solo una rama con migraciones sin integrar a la vez.
  - El número se fija al integrar, según el registro de la sección 5.
- **Integración y push:**
  - Integras tú, o con tu visto bueno, en el `develop` del repo principal.
  - El push lo pides tú, en los puntos marcados como **PUSH**.
  - Antes de cada push, una revisión manual de lo que toca esa fase: proveedores (403/404), subir 3 MB, PDF con WebP, reactivar una plantilla.
- **Agentes:** el trabajo largo va en agentes en segundo plano, y cada slice se commitea apenas está en verde.
- **Publicación (v2, obligatorio):**
  - P1 y P2 se publican con `git push origin <rama>:develop` desde ramas basadas en `origin/develop`, rebasadas sobre el push anterior. **El `develop` local del backend no se publica hasta P3, y el del frontend hasta P4**: los dos traen los 8 commits de fondos.
  - Un slice del frontend entra al `develop` local solo cuando su backend ya está en producción.
  - FE-ORG-02 y FE-IAM-01a se construyen desde `origin/develop` (no dependen de fondos) y salen en P2.
  - Toda migración de V76 en adelante abre con `SELECT set_config('app.access_scope', 'SYSTEM', true);`. Las tablas tienen `FORCE RLS`: sin esa línea, un `DO` de verificación no ve filas y un `DROP` borra datos. Cada una lleva una prueba con una fila sembrada.
  - Si un `DO` de verificación falla en producción, el despliegue se detiene: se corrige el dato con tu visto bueno y se vuelve a desplegar. La migración no se edita después de aplicada.
- **Bases de verificación (v2):** antes de F1a limpio la V75 vieja de `erp_mm_db_verify` (borro `vehicle_fund_assignments` y `funds`, los `fondos.*` con sus marcas y la fila 75), y vuelvo a medir la suite completa de `origin/develop` como línea base. El `develop` local no se prueba en esa base hasta F2, salvo que exista `erp_mm_db_verify_mig`.
- **Línea base del frontend (v2):** antes de P1 se miden en `origin/develop` los specs (12 fallan ahí, no 9), stylelint y el e2e completo. Cada slice que toca `.scss` corre `ng build` y `lint:styles`.

---

## 4. Fases

Horas estimadas por slice. Los identificadores coinciden con los planes de detalle de cada frente.

### F1a · Carril urgente: datos expuestos y flujos rotos

Desde `origin/develop`, sin fondos. Backend en `feature/procurement-hotfix` (desde `c71d797e`); frontend en `feature/procurement-hotfix` (desde `6ee4887`).

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-PROC-SEC-03 | Proveedores: escribir exige el permiso en **toda** la empresa (`actsOnWholeCompany`); leer, `compras.ver` donde actúa (`companyReach`/`actsOn`). Fuera del alcance responde 404; empresa alcanzada sin el permiso, 403 (convenciones l.446). `SupplierResource` trae `allowedActions`. Pruebas entre empresas que fallan antes del arreglo | 1, NUEVO-B (BE) | 5 |
| BE-PROC-SEC-05 | El alcance del PDF del borrador incluye `ver_datos_personales_colaboradores` | 2 (borrador) | 1 |
| BE-TPL-04 | `max-file-size` por encima del dominio (10 MB; request 21 MB; `max-swallow-size` acorde). El servicio sigue respondiendo `422 PURCHASEORDERTEMPLATE_DOCUMENT_SIZE` desde 5 MB, y `MaxUploadSizeExceededException` responde `413` genérico. Prueba con RANDOM_PORT | 4 | 2 |
| FE-PROC-03 | Activar con `allowedActions.includes('activate')` en plantillas y tipos, y el mock e2e recalcula `allowedActions` | 6 | 2 |

**PUSH P1.** El backend sale por fast-forward sobre `origin/develop` y el frontend después; los dos cambios son aditivos. Después se fusiona `origin/develop` en el `develop` local de los dos repos.

### F1b · Red de pruebas y resto de lo urgente

También desde `origin/develop`.

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-TST-01 | Quita la clave duplicada `error.permission-not-found`, agrega al hand-over las dos funciones de V69 y reescribe `HandOverDatabaseScriptTests` (DP-13) | 25, 40 | 1,5 |
| BE-TST-02 | Catálogo, matriz, pares y sesión al día con V62-V74. Solo `PermissionsMigrationTests` fija la matriz contra el diccionario; los demás tests leen de la base | 25 | 3 |
| BE-TST-03 | Tests de organization, location y vehiclecatalog con los permisos finos de D-130, y las 12 aserciones vacías pasan a probar la regla de verdad | 25 | 2 |
| BE-TST-04 | Red D-130: ningún código retirado se pide ni se otorga | 25 | 1 |
| BE-TST-06 | Red de mensajes y códigos con trinquete `KNOWN_GAPS`: duplicados, paridad entre idiomas, claves de `notFound`, prefijo único (D-114) | 40 | 2 |
| BE-PROC-SEC-01 | `AccessScope.actsOnWholeCompanyEveryBrand`, con la política pura `PurchaseOrderTemplateAccessPolicy`: un grant de sede o de marca no escribe plantillas generales ni tipos. También `PurchaseOrderAccessPolicy.canManage`: hoy un rol de sede escribe órdenes de toda la empresa | 3, NUEVO-A | 4 |
| BE-TPL-05a | Detecta el tipo por los bytes del archivo, no por el `Content-Type`; lee WebP con TwelveMonkeys; `canDraw` al subir. Un fallo de dibujo responde 500 con log, no un 400 con el texto de PDFBox. Crea `PurchaseOrderTemplatesIntegrationTests` (DP-7) | 5 | 3,5 |
| FE-PROC-SUP-01 | El frontend mapea por adelantado los códigos que el backend responderá: `*_INACTIVE`, `TAXIDTYPE_NOT_FOUND`, `PURCHASEORDER_NOT_FOUND`, el dígito de control | prepara 15, 16 y D8 | 3 |
| FE-BASE-01 | Los 9 specs vuelven a verde: el catálogo del mock, `under-construction` pasa a un módulo todavía pendiente, la fachada de organization y los specs de empresa (DP-14) | 25, 37 | 3 |
| FE-BASE-02 | Los 13 errores de stylelint, y el presupuesto de `purchase-order-form.scss` (unos 600 B de reglas muertas) | — | 1 |

**PUSH P2.** Backend y frontend, en cualquier orden. Desde aquí la red del backend queda en verde: se acaba la comparación contra las 53 fallas.

### F2 · Congelar la V75 (fondos en el backend)

Desde el `develop` local, que ya tiene P1 y P2. Rama: `feature/fleet-funds-schema`.

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-FUN-01 | **V75 enmendada:**<br>• `company_id` en la cesión, con llaves compuestas hacia `vehicles (id, company_id)` y `funds (id, company_id)` y RLS directa;<br>• `CHECK` del período, como V28 (no vacío, con inicio);<br>• `CHECK` de `fund_type` (DP-1);<br>• las 64 casillas de `data_versions` y los dos disparadores de `funds`;<br>• descripciones de permisos sin «§12.1, D-130».<br>Además, las líneas de fondos en `PermissionsMigrationTests` | D4, 42, D10 (base), 9 | 4 |
| BE-TST-05 | RLS probada en la base para todas las tablas de compras y fondos, con fixtures reutilizables y `vehicle_fund_assignments` ya en `COMPANY_TABLES` | 23 (base), 25 | 4 |

**Antes de integrar:**
- reseteo de las bases con la V75 vieja (`erp_mm_db`, `erp_mm_db_verify`, `erp_mm_db_live` si existe, y la del compañero);
- en `erp_mm_db_verify` lo hago yo, con la cuenta dueña.

**PUSH P3:** solo el backend. Fondos en el backend sin pantallas en producción es inocuo. El frontend no se publica todavía, porque su `develop` trae las pantallas de fondos con el contrato viejo. El compañero resetea su base y su rama se adelanta.

### F3 · Cierre de fondos (backend y frontend)

**Backend** (`feature/fleet-funds-closure`):

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-FUN-02 | Códigos:<br>• `FUND_CONFLICT` y `VEHICLEFUNDASSIGNMENT_CONFLICT`: con una sola regla única, el genérico, según convenciones l.444; también la carrera traducida;<br>• `VEHICLEFUNDASSIGNMENT_COMPANY_MISMATCH`, `_ALREADY_CLOSED` y `_END_NOT_AFTER_START`;<br>• `COMPANY_INACTIVE` al crear un fondo o reactivarlo;<br>• la clave de `VEHICLEFUNDASSIGNMENT_NOT_FOUND`.<br>`ErrorResponseAssembler` deja de nombrar códigos de fleet | 20, 21, 38, X6 | 2,5 |
| BE-FUN-03 | 404 antes que 403 o 422, mirando la marca (D-102). La lista de cesiones de una unidad pide `@PermissionRequired(Stock.VIEW)` y `VehicleAccessPolicy.sees`, y desaparece el literal `PERM_`. El alta exige además `fondos.ver`. Bloqueo nativo `FOR NO KEY UPDATE` + `refresh` al editar o cambiar el estado; `FOR SHARE` del fondo al ceder | 19, 41, 42, 10 | 3 |
| BE-FUN-04 | Las consultas de fondos pasan por `FundRepository.search`, sin JPA en `application`, y sin identificadores inventados | 22, 39 | 3 |
| BE-FUN-05 | `POST /fund-assignments/{id}/closure`; `PageParams` en la lista de cesiones; `Location` hacia la colección | extras | 2 |
| BE-FUN-06 | `allowedActions` del fondo (edit, activate o deactivate) y de la cesión (close). `registerFundAssignment` en la unidad = el permiso actúa en la empresa o marca **y** la unidad no tiene una cesión abierta (DP-11) | 35, extras | 2,5 |
| BE-FUN-07 | Exportación de fondos (`documents::acl`). El alcance del documento usa exactamente el permiso de la consulta, `fondos.ver` | D10 | 3 |
| BE-FUN-08 | `FundAuditTests` y `VehicleFundAssignmentAuditTests`, más la prueba de `created_by` sin sesión | 24, 23 | 2 |
| BE-FUN-09 | `codeLocked` del fondo (DP-12) | §2.4 | 2,5 |
| DOC-FUN-01 | Convenciones, `contextos/fleet.md` y el orden de implementación (paso 10 parcial, D5) | D5 | 1,5 |

**Frontend:**

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| FE-ORG-02 | Publica `mm-company-filter` con estado propio, `?empresa=` en la URL, la regla de inactivas de DP-14 y la regla 1.2 en las dos copias de las skills (DP-9) | 26, 7 | 5 |
| FE-FUNDS-01 | Mensajes de validación por la pieza compartida (`errorMessagesForControl`), sin `maxlength`; las claves de i18n pasan a `fleet.funds` y `fleet.profile.fund-assignments` | 27, 28, 45 | 3 |
| FE-FUNDS-02 | Los tipos de red se llaman `*-response.ts`; se quitan los miembros sin uso | 45 | 1 |
| FE-FUNDS-03 | La lista usa `mm-company-filter`, y el diálogo recibe la empresa de la vista: sale `CompaniesStore` | 7, 26, 21 | 2 |
| FE-FUNDS-04 | El store devuelve `Promise<boolean>` y expone una señal de error; los diálogos llaman a `clearError()`. Acciones por `allowedActions`. Columnas ordenables solo donde el servidor ordena. Confirmación destructiva correcta, y activar sin confirmar | 44, 43, 35 | 3 |
| FE-FUNDS-05 | Botón Exportar | D10 | 1,5 |
| FE-FUNDS-06 | `fund_type` como lista cerrada (DP-1) | 42 | 1 |
| FE-FUNDS-07 | Cesiones paginadas; la pestaña avisa los errores y los éxitos; el store se limpia al cambiar de unidad | 33 | 4 |
| FE-FUNDS-08 | Fechas en dd/mm/aaaa y «hoy» en la zona de la unidad; fecha de fin válida por defecto | 34 | 2 |
| FE-FUNDS-09 | «Ceder» y «Cerrar» por `registerFundAssignment` y `allowedActions`; «Ver cambios» en cada cesión | 35 | 2,5 |
| FE-FUNDS-10 | e2e de los flujos, `SEEDED_MATRIX` alineada con la V75, imágenes de referencia | 37 | 4 |
| FE-IAM-01a | Fondos en su pantalla de la matriz, con etiquetas en `es.json` | 36 | 1 |
| FE-CHG-01 (cesiones) | Bitácora: entidad «Cesiones a fondos» y el campo período | 46 | 1 |

**PUSH P4:** primero el backend, después el **primer push del frontend con fondos**. No hace falta que el frontend tolere dos contratos: fondos nunca estuvo en producción.

### F4 · Compras compatible

Se desarrolla en paralelo con F3 desde P3. Las ramas del backend se integran en este orden porque comparten los servicios de plantilla y de orden.

**1. `feature/procurement-errors` → PUSH P5a**

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-PROC-SUP-02 | `*_INACTIVE` con 422 en lugar de `*_CONFLICT` (empresa, marca, tipo, plantilla, proveedor, moneda, modelo); prefijo único por agregado (`PURCHASEORDER_NOT_FOUND`, `TAXIDTYPE_NOT_FOUND`, `PURCHASEORDER_PDF_NOT_FOUND`); claves de mensaje (absorbe BE-TPL-01) | 15, 40, D-114 | 5 |
| BE-PROC-SUP-01 | `SUPPLIER_OUTDATED` responde 422, como todo `*_OUTDATED` (D8) | D8 | 1 |
| BE-PROC-SUP-03 | `TaxIdTypeRules` de procurement: país, que esté activo (solo si el tipo es nuevo), que aplique a empresas y longitud. Responde `BUSINESS_RULE_VIOLATION` y `SUPPLIER_TAX_ID_CHECK_DIGIT_INVALID`, siguiendo el precedente de organization | 16, D6 | 4 |
| BE-PROC-SUP-04 | Las carreras de unicidad responden su propio `*_CONFLICT` (`UniqueViolations`) | 38 | 3 |

**2. `feature/procurement-locks` → PUSH P5b**

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-PROC-SEC-04 | Bloqueo nativo `FOR NO KEY UPDATE` + `refresh` en proveedores, plantillas y órdenes, antes de comparar `updatedAt` y antes de `hasOrders`. La numeración relee la serie; enviar, anular y cumplir se bloquean | 10, NUEVO-C, NUEVO-D | 4 |
| BE-PROC-SEC-02 | La lista de tipos se limita por `companyReach` | 11 | 1,5 |

**3. `feature/procurement-templates` → PUSH P5c**

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-TPL-02 | Quita el proveedor de la plantilla en el código (D1); con eso desaparece su N+1. Las columnas siguen | D1 (código) | 3 |
| BE-TPL-05b | El PDF se dibuja detrás de un puerto: `application` deja de importar `infrastructure.pdf` | 22 | 1,5 |
| BE-TPL-06 | Valida todos los archivos antes de escribir nada; llave por contenido (sha256): escribir lo nuevo y después apuntar | 12, 13 | 3 |
| BE-TPL-07 | El borrador se dibuja solo cuando `DocumentService` lo genera, y su versión incluye los documentos de la plantilla. El controlador deja de importar el repositorio | 13, 14 | 4 |
| BE-TPL-08 | El emisor se copia de la empresa al crear y el PATCH no lo acepta (DP-5). La Atención sale solo del colaborador; `attention_email` vacío (DP-6) | 16 (emisor), 18, 31 (causa) | 3 |
| BE-TPL-09 | D9 aditivo: el PATCH acepta `removedDocuments: ["LOGO"]`, todo en una transacción; el `DELETE` sigue vivo. **Se conserva** el comando de estado. Un campo obligatorio en nulo responde 400, no 500 | D9 (expandir), 20 del informe | 5 |
| BE-TPL-10 | Bitácora legible: nombres y códigos en lugar de ids, estado y archivos | 46 | 3 |
| BE-TPL-11 | El uso del tipo en una sola consulta; un relleno inventado se convierte en error | 39, N+1 | 2 |
| BE-TPL-13 (nuevo) | 404 entre empresas en las rutas REST de plantillas, tipos y documentos | 23 | 2 |
| BE-TPL-14 (nuevo) | `PurchaseOrderTemplateAuditTests` y `PurchaseOrderTemplateDocumentAuditTests` | 24 | 1,5 |
| BE-PROC-SEC-06 | Una sola copia fiel del PDF enviado, con el teléfono (DP-4) | 2 (enviada) | 1 |
| DOC-TPL-01 | Convenciones y `contextos/procurement.md` | D8, D9 | 1,5 |

**4. `feature/procurement-suppliers` (después de 1 y 2) → PUSH P5d**, primero el backend:

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-PROC-SUP-08 | `SupplierAuditTests` | 24 | 1,5 |
| BE-PROC-SUP-06 | Búsqueda de proveedores por el repositorio del dominio, con un trinquete de capas | 22, 39 | 5 |
| BE-PROC-SUP-09 | 404 de un proveedor ajeno, en lectura y escritura | 23 | 2 |
| BE-PROC-SUP-10 | Exportación de proveedores. El alcance del documento es `grantsOf(compras.ver)`, igual que la consulta: **no** se unen los permisos de escritura, para no mezclar empresas | D10 | 4 |
| DOC-PROC-SUP-01 | Códigos, el 422 de `_OUTDATED`, proveedores locales, la exportación | D6, D8, D10 | 2 |

**Carril de migraciones** (`feature/procurement-migrations`, base propia o serializada) → PUSH P5e:

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| BE-PROC-SUP-05 | **V76:** `UNIQUE (suppliers.address_id)`, con un bloque `DO` que falla si hay duplicados | 17 | 1,5 |
| BE-LOC-ADDR (rehecho) | **V77:** RLS de `addresses` derivada del país (DP-8). Pruebas: el alta de una empresa como superadministrador; las direcciones de empresa, sede, ubicación, proveedor y emisor; una cuenta de A no lee las de B; los procesos SYSTEM | 8 | 3 |

**Frontend** (`feature/procurement`) → PUSH P5f, cada slice después del backend del que depende:

| Slice | Qué hace | Cierra | h |
|---|---|---|---|
| FE-PROC-07 | Quita del contrato de la plantilla el proveedor y la Atención escrita a mano (se puede publicar en cualquier momento) | D1, 18 | 1,5 |
| FE-ORG-01 | `OrganizationFacade.loadCompanyIssuer` (solo para la vista previa del emisor) | 7, 31 | 1,5 |
| FE-PROC-01 | Modelo de dirección propio de procurement | 26 | 1 |
| FE-PROC-02 | Empresas por la fachada y `mm-company-filter`. Son dos commits: listas, y formularios. Sale `CompaniesStore` y el sondeo de 4 s | 7, 26, 31 | 7 |
| FE-PROC-04 | Cada fallo dice su texto; validadores con `mat-error`, sin `maxlength`; columnas ordenables solo donde el servidor ordena; `confirmLabel` e `isDestructive` bien puestos, con el tipo de retorno anotado; las filas de proveedores por `allowedActions` | 27, 28, 43, 44, NUEVO-B (FE) | 3 |
| FE-PROC-05 | Campo de correo de soporte. Avisa «Un PDF no se imprime en la orden» en logo y firma (DP-7) | 29, 31 | 1,5 |
| FE-PROC-08a | La plantilla se edita según `allowedActions`; quien solo lee no ve campos editables ni la pregunta de cambios sin guardar; los errores de carga dicen la verdad | 30, Q4 | 1,5 |
| FE-PROC-09 | «Nuevo tipo» abre su diálogo; desaparece el formulario en línea | 32 | 1,5 |
| FE-PROC-12 | La fecha de la orden en el día de la empresa | 34 | 1 |
| FE-PROC-10 | Botón Exportar proveedores (después de P5d) | D10 | 2 |

### F5 · Contratos atados: D7, D9 y la ruta de estado

| Orden | Slice | Qué hace | h |
|---|---|---|---|
| 1 | FE-PROC-SEC-09a | El frontend acepta el código fino o el grueso; mock del catálogo. Funciona con el backend viejo | 2 |
| 2 | BE-PROC-SEC-07 A+B | **V78:** los 8 códigos finos, sus marcas en los roles del sistema y en los roles propios (DP-2, DP-3). Las rutas y los servicios piden los códigos finos. `PATCH /purchase-order-templates/{id}/status` (BE-TST-08 absorbido) | 6 |
| 3 | FE-PROC-SEC-09b | El estado de la plantilla por `/status` | 2 |
| 4 | FE-PROC-06 | Guardar la plantilla en una sola petición, con `removedDocuments` | 2,5 |
| 5 | FE-IAM-01b | Los códigos finos de compras en su pantalla de la matriz (por sufijo `_plantillas` y `_ordenes`) | 1,5 |
| 6 | FE-CHG-01 (plantillas) | Etiquetas de los campos nuevos de la bitácora | 1 |
| 7 | FE-PROC-13 | e2e de plantillas, tipos y órdenes, con axe, capturas y flujos | 6 |

**PUSH P6a:** 1 y 2. **PUSH P6b:** 3 a 7, solo después de verificar P6a y P5c en producción.

### F6 · Contracciones y deuda

Cada una se hace después de verificar el despliegue anterior.

| Slice | Qué hace | h |
|---|---|---|
| BE-PROC-SEC-07 C | **V79:** revoca los gruesos en los roles del sistema y los agrega a `RETIRED` | 1 |
| BE-PROC-SEC-10 + FE-PROC-SEC-10 | El PATCH sin `isActive`; el frontend deja de aceptar el código grueso | 1,5 |
| BE-TPL-12 | Se retira el `DELETE` de documentos; `NoDeleteEndpointsTests`; se borra la excepción de convenciones §3.1 | 1,5 |
| BE-TPL-03 | **V80:** `DROP` de `supplier_id` y `supplier_contact_id`, con un `DO` que falla si hay valores | 1 |
| BE-PROC-SUP-11, 12, 13 | Tipos, plantillas y órdenes buscan por el repositorio del dominio: el trinquete de capas llega a cero | 12,5 |
| FE-PROC-08b (opcional, DP-10) | La ficha se lee como hoja impresa y se edita en el sitio. DESIGN revisión 25 | 5 |
| FE-PROC-SUP-01 (commit 7) | El frontend deja de mapear `COMPANY_CONFLICT` y `TAX_ID_TYPE_CONFLICT` | 0,5 |

**PUSH P7.**

---

## 5. Registro de migraciones

**Registro final** (26/10/2026), verificado contra las migraciones de `feature/procurement-permissions`. Manda sobre la numeración de la versión 2: DP-32 agregó la normalización de los documentos de los proveedores como V76, y las demás corrieron un número. Los pushes son los grupos del backend de DP-40: P1, P3, P5 (toda F4, que reemplaza a P5a–P5e), P6a y P7. Toda migración de V76 en adelante abre con el alcance `SYSTEM`.

| N.º | Slice | Contenido | Push | Seguridad con los datos de producción |
|---|---|---|---|---|
| V75 (final) | BE-FUN-01, con los ajustes de DP-27 | Fondos completos (sección F2): `company_id` y llaves compuestas en la cesión, `CHECK` del período y de `fund_type`, una política por comando, la exclusión `_excl` con un gist `(fund_id, period)` y las casillas de `data_versions` | P3 | Nunca se desplegó. Primero se resetean las bases que tienen la vieja (y la del compañero: [retroalimentación del 26/10/2026](retroalimentacion/2026-10-26-fondos.md), anexo) |
| V76 (final) | DP-32 (revisión de F4-a) | Normaliza los `suppliers.tax_id` guardados como `TextIdentifier`, también los separadores Unicode, con una función temporal | P5 | `LOCK TABLE suppliers IN SHARE ROW EXCLUSIVE MODE` y `lock_timeout` de 5 s; el `DO` se detiene, sin cambiar nada, si dos proveedores de una empresa quedan con el mismo documento o si uno queda vacío (DP-32, DP-38) |
| V77 (final) | BE-PROC-SUP-05 | `UNIQUE (suppliers.address_id)` (`suppliers_address_id_uq`) | P5 | Consulta previa (c); el `DO` se detiene y nombra la dirección y los proveedores si hay duplicados. `lock_timeout` de 5 s |
| V78 (final) | BE-LOC-ADDR | RLS de `addresses` por país, con rama SYSTEM (`access_scope_reaches_country`, tres políticas, sin `DELETE`) | P5 | Consultas previas (f) y (h). Bloquea contra escrituras `addresses`, `suppliers`, `branches`, `locations`, `purchase_order_templates` y `companies` antes de revisar, con `lock_timeout` de 5 s; el `DO` se detiene, nombrando las filas, si una dirección de una empresa, una sede, una ubicación, un proveedor o un emisor está en otro país que el de su empresa (DP-38). Prueba con la cuenta de la aplicación y con SYSTEM |
| V79 (final) | BE-PROC-SEC-07 A (D7, expandir) | Los ocho códigos finos de compras, con su categoría, y sus marcas firmadas por SYSTEM: el superadministrador y el administrador de empresa los ocho, Operaciones los cinco de órdenes, Contabilidad ninguno (DP-2); cada rol propio con un grueso vigente, los finos que lo reemplazan (DP-3). Los gruesos siguen marcados | P6a | Solo inserta, con `ON CONFLICT`. Consulta previa (a). El `DO` del final se detiene si faltan códigos o si un rol tiene un grueso vigente sin todos sus finos |
| V80 (final) | BE-PROC-SEC-07 C (D7, contraer) | Revoca `compras.gestionar_plantillas` y `compras.gestionar_ordenes` en todos los roles, también en los propios, firmado por SYSTEM. Antes, la ventana de DP-3: un rol propio con un grueso vigente y ninguno de sus finos los recibe; uno que ya tiene alguno queda como está (DP-42) | P7 | Solo cuando el frontend ya pide los finos. `lock_timeout` de 5 s. Solo la revisión posterior (DP-42): el `DO` se detiene si queda una marca gruesa vigente |
| V81 (final) | BE-TPL-03 | `DROP` de `purchase_order_templates.supplier_id` y `supplier_contact_id` y de sus dos llaves compuestas, las de V73 | P7 | Consulta previa (b); el `DO` se detiene, nombrando las plantillas, si alguna todavía apunta a un proveedor o a un contacto. `lock_timeout` de 5 s |

Las que ya están en una rama no se renumeran: el orden de arriba es el de integración.

---

## 6. Contratos FE/BE: una sola versión

| Contrato | Versión fijada | Orden de publicación |
|---|---|---|
| D9: quitar un archivo | `removedDocuments: ["LOGO"]` en el JSON del PATCH, también en la parte `template` del multipart. Un tipo desconocido responde 404 `PURCHASEORDERTEMPLATEDOCUMENTTYPE_NOT_FOUND`; quitar y subir el mismo tipo en un pedido, 400. `PUT /{id}/documents/{type}` se conserva para tipos futuros | BE aditivo (P5c) → FE (P6b) → retirar el `DELETE` (P7) |
| Estado de la plantilla | `PATCH /purchase-order-templates/{id}/status` con `{isActive, updatedAt}` | BE (P6a) → FE (P6b) → PATCH sin `isActive` (P7) |
| D7 | Códigos de DP-2; las rutas GET siguen pidiendo `compras.ver` | FE tolerante → V79 y rutas → V80 y limpiezas (registro final, sección 5) |
| D8 | Todo `*_OUTDATED` responde 422 | Cualquier orden: el frontend elige por código |
| Códigos de compras | `*_INACTIVE`, prefijo único, `BUSINESS_RULE_VIOLATION` + `<X>_TAX_ID_CHECK_DIGIT_INVALID` | El frontend mapea primero (P2) → BE (P5a) → limpieza (P7) |
| Fondos | Códigos de BE-FUN-02; `/closure`; lista paginada; `allowedActions`; `registerFundAssignment` con estado | BE y FE juntos en P4 (nunca estuvo en producción) |
| Proveedores | `allowedActions` (P1); lectura con `compras.ver`; exportación con el mismo alcance | BE primero |
| D1 | Sin `supplierId` ni `supplierContactId` | Código BE y FE en cualquier orden; `DROP` al final |
| Subida | 413 genérico por encima de 10 MB; 422 `DOCUMENT_SIZE` desde 5 MB | Sin cambio en el FE |

---

## 7. Cobertura

| Hallazgo | Slice | Hallazgo | Slice |
|---|---|---|---|
| 1 | BE-PROC-SEC-03 | 24 | SUP-08, TPL-14, FUN-08 |
| 2 | SEC-05, SEC-06 | 25 | BE-TST-01..05, FE-BASE-01 |
| 3 | SEC-01 | 26 | FE-ORG-02, FE-PROC-01/02, FE-FUNDS-03 |
| 4 | TPL-04 | 27 | FE-PROC-04, FE-FUNDS-01, FE-PROC-SUP-01 |
| 5 | TPL-05a/b | 28 | FE-PROC-04, FE-FUNDS-01 |
| 6 | FE-PROC-03 | 29 | FE-PROC-05 |
| 7 | FE-ORG-01/02, FE-PROC-02, FE-FUNDS-03 | 30 | FE-PROC-08a |
| 8 | BE-LOC-ADDR (V77) | 31 | TPL-08, FE-ORG-01, FE-PROC-02/05 |
| 9 | F2 (reseteos) y adelantar la rama del compañero | 32 | FE-PROC-09 |
| 10 | SEC-04, FUN-03 | 33 | FE-FUNDS-07 |
| 11 | SEC-02 | 34 | FE-PROC-12, FE-FUNDS-08 |
| 12 | TPL-06 | 35 | FUN-06, FE-FUNDS-04/09 |
| 13 | TPL-06, TPL-07 | 36 | FE-IAM-01a/b |
| 14 | TPL-07 | 37 | FE-BASE-01, FE-PROC-13, FE-FUNDS-10 |
| 15 | SUP-02 | 38 | SUP-04, FUN-02 |
| 16 | SUP-03, TPL-08 | 39 | TPL-11, SUP-06, FUN-04 |
| 17 | SUP-05 (V76) | 40 | SUP-02, TST-01, TST-06 |
| 18 | TPL-08, FE-PROC-07 | 41 | FUN-03 |
| 19 | FUN-03 | 42 | FUN-01, FUN-03, FE-FUNDS-06 |
| 20 | FUN-02 | 43 | FE-PROC-04, FE-FUNDS-04 |
| 21 | FUN-02 | 44 | FE-PROC-04, FE-FUNDS-04 |
| 22 | SUP-06, TPL-05b, FUN-04, SUP-11/12/13 | 45 | FE-FUNDS-01/02 |
| 23 | TST-05, TPL-13, SUP-09, FUN-08 | 46 | TPL-10, FE-CHG-01 |

**Divergencias:**

| D | Slices |
|---|---|
| D1 | TPL-02, FE-PROC-07, TPL-03 (V80) |
| D2 | FUN-01, TST-02 y el texto de §5.6 |
| D3 | Diccionario v1.29 (hecho) |
| D4 | FUN-01 |
| D5 | DOC-FUN-01 y D-133 (hecho) |
| D6 | SUP-03, DOC-PROC-SUP-01, diccionario v1.29 (hecho) |
| D7 | SEC-07 A/B/C, FE-PROC-SEC-09a/b/10, FE-IAM-01b |
| D8 | SUP-01, DOC-PROC-SUP-01 |
| D9 | TPL-09, FE-PROC-06, TPL-12, DOC-TPL-01 |
| D10 | FUN-01, FUN-07, FE-FUNDS-05, SUP-10, FE-PROC-10 |

**Hallazgos nuevos que salieron al planificar:**
- **NUEVO-A:** un rol de sede escribe órdenes de toda la empresa (SEC-01).
- **NUEVO-B:** proveedores sin `allowedActions` (SEC-03 y FE-PROC-04).
- **NUEVO-C:** la numeración lee la serie desde memoria (SEC-04).
- **NUEVO-D:** enviar, anular y cumplir sin bloqueo; una orden anulada puede revivir como enviada (SEC-04).
- Una plantilla cuyo tipo de documento se desactivó ya no se puede editar (TPL-08).
- Un PATCH con un campo obligatorio nulo responde 500 (TPL-09).
- Hay 12 aserciones que pasan sin probar nada (TST-03).

---

## 8. Lo que no se hace, y por qué

| Qué | Por qué |
|---|---|
| `purchase-order-template-form.ts:205` | Falso positivo: el código es correcto (dos planificadores y una crítica lo confirmaron) |
| Convertir la lista de tipos del diálogo a `mm-data-manager` | Es una lista corta dentro de un diálogo, no una tabla; se cierra el formulario en línea, que era el problema |
| Mover `address.ts` de organization a `shared` | Tocaría unos 21 archivos de organization; procurement tiene su propio modelo, coherente con §4.6 |
| Responder 403 en lugar de 404 en las escrituras de plantillas | Cambiar el contrato no protege nada nuevo. Queda como candidato para unificar con organization |
| ~~`PURCHASEORDER_OUTDATED` en el borrador de la orden~~ | ~~Necesita diccionario y frontend. El bloqueo de SEC-04 ya evita corromper el ciclo de estados. Es candidato~~ Hecho el 29/10/2026, por decisión del responsable del 29/10/2026 (D-138): V85 da a `purchase_orders` `updated_at` y `updated_by`, `PUT /purchase-orders/{id}` exige el `updatedAt` que leyó y uno anterior responde `422 PURCHASEORDER_OUTDATED` (backend `a0201a41`, frontend `5656f985`). El cuadro del diccionario (§9.2) tiene las dos columnas desde la v1.33, del 30/10/2026 (sección 13.6) |
| `PURCHASEORDERTYPE_CODE_CONFLICT`, que ya está publicado y mapeado | Con una sola regla única debería ser el genérico, pero renombrarlo no protege nada. Se anota como excepción |
| Redactar los PDF ya enviados en producción | No se pueden regenerar fielmente. Siguen sirviendo su único archivo |
| Versionar `vehicle_fund_assignments` en `data_versions` | Ninguna exportación la lee todavía. Llega con la lista de cesiones por fondo (paso 9 o 10) |
| Mapear el estado 0 (sin red) solo en fondos | Ningún store lo hace; sería una inconsistencia |
| Rediseñar la dirección como subrecurso (regla 4.7) en el PUT de proveedor y el PATCH de plantilla | No es un hallazgo de esta auditoría. Lo decides tú |
| ~~Plegar tildes en la búsqueda de proveedores~~ | ~~No es un hallazgo de esta auditoría. Lo decides tú (`unaccent` o `translate()` en el índice)~~ Resuelto el 30/10/2026 por D-142, para toda búsqueda del servidor: V86 crea `fold_search_text(text)` con `translate()` y `lower()`, inmutable, sin `unaccent` y sin índice, y los dos lados de cada búsqueda `q` pasan por ella (backend `145fc104`, `a54ee29b`; sección 13.6) |
| Limpiar objetos huérfanos en Spaces | Nada se borra en este sistema, y con llaves por contenido son pocos y pequeños |
| `suppliers` en el filtro de la bitácora | El backend no registra cambios de proveedores. ¿Se registran sus correcciones? Lo decides tú |

---

## 9. Riesgos principales y cómo se cubren

1. **Contratos FE/BE desalineados.** Se fijan en la sección 6, antes de abrir worktrees. Cada slice de contrato lleva su caso e2e con el mock alineado.
2. **El despliegue de D7 quita botones o permisos en producción.** Se expande y contrae en tres pushes, y la migración remarca los roles propios (DP-3), validado con la consulta (a).
3. **La V75 y los reseteos.** F2 es la única ventana. Si hiciera falta publicar el backend de `develop` antes, el plan B es no enmendar y usar una V76 al estilo de V50 (+1,5 h, sin reseteos). F1a y F1b existen justamente para no necesitarlo.
4. **La RLS de direcciones falla cerrada en toda la aplicación.** Se prueba con la cuenta de la aplicación, con el superadministrador (SYSTEM) creando una empresa, y con cada dueño. Se ensaya sobre una copia de producción si la creas.
5. **Fugas por documentos reutilizados.** Una regla para todos: el alcance de un documento usa exactamente los permisos y las banderas que leen sus filas. Se aplica en SEC-05, SUP-10 y FUN-07, y queda una prueba por cada tipo de documento.
6. **Tests de otras ramas rotos por migraciones aplicadas en la base compartida.** Una sola rama con migraciones a la vez, o la segunda base de verificación.

---

## 10. Esfuerzo y ruta crítica

| Fase | Horas | Push |
|---|---|---|
| F0 (tú) | unas 3 | — |
| F1a | 10 | P1 |
| F1b | 24 | P2 |
| F2 | 8 | P3 |
| F3 | 53 | P4 |
| F4 | 89,5 | P5a–P5f |
| F5 | 21 | P6a, P6b |
| F6 | 23 (de las que 17,5 son deuda u opcional) | P7 |
| **Total de desarrollo** | **unas 228 h** (más tus 3 h de F0), de las que unas 211 son el núcleo | |

**Ruta crítica (en serie):** F0 (DP-1, DP-7) → F1a → F1b → F2 con los reseteos → errores (P5a) → bloqueos (P5b) → plantillas (P5c) desplegado → D7 (P6a) → FE (P6b) → contracciones (P7). Son unas 70 h de trabajo en serie, más las esperas entre despliegues.

**En paralelo desde P3:** fondos (F3, unas 53 h) y el frontend de compras (unas 22 h).

**Lo primero que se puede empezar hoy, sin decisiones pendientes:** limpiar la base de verificación, medir las líneas base y F1a completo (con las enmiendas de 11.2).


---

## 11. Enmiendas de la versión 2 (mandan sobre todo lo anterior)

La auditoría crítica del plan (siete auditores y un consolidador, verificada contra el código) encontró:
- dos diseños incorrectos (la RLS de direcciones sin rama SYSTEM, y `codeLocked` solo en el backend);
- cuatro contratos que contradecían a los diseños de detalle;
- un defecto que ya está en producción (la matriz de permisos);
- seis ventanas de publicación inconsistentes;
- decisiones sin su texto del diccionario.

Sobre 88 filas: 57 resueltas, 29 parciales y 2 incorrectas. Estas enmiendas cierran las 31 restantes.

### 11.1 Defecto en producción que se adelanta

| Slice nuevo | Qué hace | Cierra | Fase | h |
|---|---|---|---|---|
| BE-IAM-01 | La columna de `GET /permission-matrix` filtra los códigos de `RETIRED` (hoy `PermissionMatrixQueryServiceImpl` devuelve `role.permissionCodes()` sin filtrar). Al guardar un rol propio, un código retirado que tenía se desmarca en vez de rechazar el guardado. Prueba en `RolesIntegrationTests`: un rol propio con `seguridad.gestionar_colaboradores` (V62) se guarda desde la matriz | defecto en producción desde V62 y V64; prerrequisito de D7 | F1b | 2 |

Hoy un rol propio que conserva un código grueso de V62 o V64 no se puede guardar desde la matriz: el servidor responde 422 `ROLE_PERMISSIONS_NOT_HELD` por una casilla que no se ve.

### 11.2 Correcciones por slice

**F1a**
- **BE-PROC-SEC-03** (DP-18):
  - `ProcurementReadPermissions.SUPPLIERS = {compras.ver}`, y la usan la búsqueda, el detalle y la exportación (SUP-10). El paso 10 de SEC-07 y la unión del JSON de SEC-02 quedan anulados.
  - Las respuestas de crear, editar y cambiar el estado se arman con la vista del agregado que guardó el comando, sin el filtro de lectura. Prueba: `aWriterWithoutVerGetsWhatItSaved`.
  - 404 si quien llama no ve el proveedor (fuera de `companyReach(compras.ver)` o de la RLS); 403 `ACCESS_DENIED` si lo ve pero el permiso de la escritura no actúa en toda la empresa. SUP-09 se alinea con esto.
  - La guarda de la ruta del frontend pide `compras.ver`.
- **Frontend F1a** suma la clave `common.errors.unexpected` en `es.json` y el texto por operación del store de proveedores. Si no, los 403 y 404 nuevos de P1 muestran una clave cruda.
- **TPL-04** toca `GlobalExceptionHandler` antes de la red: es aditivo (un 413) y se acepta como excepción al criterio 2.

**F1b**
- **Orden fijo:** TST-01, 02, 03, 04 y 06 → TST-05 (solo las tablas de compras) → BE-IAM-01 → TPL-05a → SEC-01.
- **BE-TST-02** se diseña sobre `origin/develop` (sin fondos), contra la base de verificación ya limpia. Las líneas de fondos llegan con BE-FUN-01.
- **BE-TST-06** extiende la red a los códigos armados con `new ApplicationError(...)` y agrega una aserción: ningún `@PreAuthorize` contiene `'PERM_'` salvo `PermissionRequired` (cierra la red del 41).
- **TPL-05a:**
  - el dibujo detecta el tipo por los bytes guardados, no por el `mime_type` guardado, para que un archivo de producción mal declarado se dibuje igual (prueba `aWebpStoredAsPngIsDrawn`);
  - `ImageIO.scanForPlugins()` y una comprobación al arrancar de que el lector WebP está registrado en el jar;
  - una revisión única de los documentos guardados con `canDraw`, que informa los que no se pueden dibujar.
- **SEC-01** agrega `aBranchGrantDoesNotWriteAGeneralTemplate` a `PurchaseOrderTemplatesIntegrationTests`. Antes de P2 corres la consulta (a) separada por `scope_level`: cuántos roles de sede pierden la escritura.
- **FE-BASE-01:** son 12 specs desde `origin/develop`. Incluye el cherry-pick de 26e5a73 (`canWrite` → `canEdit`, que hoy solo vive en la rama de fondos).
- **FE-PROC-SUP-01** quita los códigos del emisor que con DP-5 nunca existirán, y agrega el texto por operación de cada store de compras.
- **FE-ORG-02 y FE-IAM-01a** salen en P2, desde `origin/develop`. Así la matriz de producción ubica `fondos.*` antes de que P3 los siembre.

**F2**
- **BE-FUN-01:** `fund_type` en minúsculas (DP-1) y las descripciones limpias. Además, un `UPDATE` de las descripciones en las bases que se resetean, si no se recrean.
- **BE-FUN-02 y BE-FUN-03 se adelantan a F2.** La API de fondos sale a producción en P3 sin la fuga 403/422, sin fondos en empresas inactivas y sin el literal `PERM_`.
  - **BE-FUN-02** es el único dueño de la clave `error.vehiclefundassignment-not-found`; SUP-02 no la agrega.
- **D2 bloquea F2:** el texto de §5.6 y de la matriz con `crear/editar/desactivar_fondos` (DOC-DICT) tiene que estar validado antes de congelar la V75.

**F3**
- **BE-FUN-05** relee la cesión con `refresh` después de bloquear la unidad (prueba `twoClosuresAreSerialized`). En DOC-FUN-01 anota las excepciones a convenciones l.81 (`Location` hacia la colección) y l.115 (`/closure` con fecha en el cuerpo).
- **BE-FUN-06:** `registerFundAssignment` = el permiso actúa en la empresa o marca **y** `fondos.ver` en esa empresa (el diálogo lista los fondos) **y** ninguna cesión de la unidad con fin vacío o posterior a hoy. Lleva tres pruebas, y el mock e2e lo replica.
- **FE-FUNDS-01** conserva el `mat-error` propio de `notAfterStart`, y quita la pista «No se cambia después de registrarlo» (DP-12).
- **FE-FUNDS-03** usa el contrato de FE-ORG-02 (`mm-company-filter`, `CompanyOption.isActive`), no `company-select`.
- **FE-FUNDS-04** mapea los códigos del fondo, y **FE-FUNDS-07** los de la cesión, según la tabla 11.4.
- **FE-FUNDS-09:** «Ver cambios» solo en una cesión cerrada (una abierta no tiene cambios registrados).
- **FE-FUNDS-11 (nuevo, 2 h):** el código del fondo es editable mientras `codeLocked` sea falso; mapea `FUND_CODE_LOCKED` con su clave; actualiza el mock e2e.

**F4**
- **BE-TPL-08, emisor de punta a punta (DP-5, unas 6 h):**
  - amplía el ACL de organization con `CompanyIssuer` (razón social, tipo y número de documento, teléfono, domicilio fiscal);
  - al crear, copia el emisor y el domicilio fiscal a una fila propia de `addresses`;
  - **el PATCH ignora (no rechaza)** `issuer*`, `address` y `addressChange`. Si los rechazara, todo guardado del frontend en producción fallaría entre P5c y P5f.
  - Consulta previa (g): plantillas cuyo emisor difiere del de su empresa. Si hay, el responsable decide si se respetan.
- **FE-PROC-15 (nuevo, 2 h):** quita el emisor y su domicilio del contrato, del formulario y del mock; la hoja los muestra como lectura desde `loadCompanyIssuer`. Entra después de P5c.
- **En P7**, el backend quita esos campos de los recursos.
- **BE-TPL-11** también quita los rellenos de `PurchaseOrderViewFactory` (l.152-168).
- **BE-TPL-13** suma el 404 entre empresas de las órdenes: GET, PUT y el envío.
- **BE-TPL-14** suma `PurchaseOrderTypeAuditTests` y `PurchaseOrderAuditTests` (+1 h).
- **BE-PROC-SEC-06** sigue la opción B (DP-4): `renderPdf(..., includeAttentionPhone = true)` al enviar, y el borrador sigue con la bandera de quien llama. Prueba: `aSentOrderPrintsThePhoneWhoeverSentIt`.
- **BE-PROC-SUP-11, 12 y 13 pasan a obligatorios** en F6: el 22 es una violación de la regla de capas, no deuda opcional.
- **FE-PROC-04:**
  - las filas de proveedores leen `allowedActions` (modelo, ensamblador y mock); FE-PROC-02 no decide con `hasPermissionIn`;
  - activar ya no pide confirmación, igual que fondos y la referencia de monedas.
- **FE-PROC-02 y FE-PROC-04** regeneran las capturas de proveedores.
- **FE-PROC-08a:**
  - lee `?edit=true` (abre la edición si `allowedActions` la incluye) y ofrece Activar en la ficha;
  - el formulario de la orden se deshabilita para quien solo lee (`purchase-order-form.ts`);
  - spec: «Ver abre en lectura».
- **FE-PROC-09:** «Nuevo tipo» se ofrece solo con el permiso de crear plantillas.
- **FE-PROC-12:** «hoy» en la zona de la empresa de la orden. Si la sesión no la lista (alcance sistema), usa la del país de la empresa y, en último caso, la de omisión.
- **BE-LOC-ADDR (V77)** con la rama SYSTEM (DP-8). Pruebas:
  - el alta REST de una empresa con domicilio en un país nuevo;
  - las direcciones de empresa, sede, ubicación, proveedor y emisor;
  - una cuenta de A no lee las de B;
  - SYSTEM;
  - `addresses` en su propio conjunto de `RowLevelSecurityMigrationTests`;
  - si usa una función nueva, va al script de hand-over.

**F5 y F6 (D7)**
- **FE-PROC-SEC-09a** ya da los gruesos por retirados en la matriz (filtrados por BE-IAM-01), y **FE-PROC-SEC-09b** agrega la ruta `/status` al mock.
- **V78** agrega los códigos gruesos de compras a `RETIRED` en el mismo commit que pasa las rutas a los finos (con BE-IAM-01, la matriz deja de ofrecerlos ya en P6a).
- **V79** revoca los gruesos **también en los roles propios** (que tienen los finos desde V78, DP-3), no solo en los del sistema. Prueba: un rol propio remarcado se guarda desde la matriz después de V79.
- **SEC-04** suma una prueba de integración «anular y enviar en paralelo» que falla antes del arreglo.
- **DOC-TPL-01** se reescribe:
  - el 413 y los límites de 10 MB por archivo y 21 MB por pedido;
  - sin códigos del emisor;
  - `removedDocuments`;
  - el borrado de la excepción §3.1 se mueve a TPL-12 (P7), porque el `DELETE` vive hasta ahí.
- Los JSON de **FE-PROC-06** y **FE-PROC-13** usan `removedDocuments`, no `"logo": null`.

### 11.3 Ventanas de publicación corregidas

| Ventana | Problema | Corrección |
|---|---|---|
| P1–P2 | El `develop` local tiene la V75 vieja: un push desde el repo principal la congelaría | Publicación por rama (sección 3) |
| P1–P5f | Los 403/404 nuevos de proveedores mostraban una clave cruda | `common.errors.unexpected` y el texto por operación en F1a |
| P3–P4 | La matriz de producción mostraba un grupo «fondos» sin traducir; la API de fondos salía con la fuga 403/422 | FE-IAM-01a en P2; FUN-02 y FUN-03 en F2 |
| Antes de P4 | Cualquier push del `develop` del frontend publicaba las pantallas de fondos con el contrato viejo | El `develop` local del frontend no se publica hasta P4; un slice entra solo con su backend en producción |
| P5c–P5f | Rechazar los campos del emisor rompía el guardado de plantillas | El PATCH los ignora; se quitan en P7 |
| P6a–P7 | La matriz ofrecía gruesos que ya no autorizan nada; los roles propios no se podían guardar | BE-IAM-01, `RETIRED` en V78, V79 también en los roles propios |
| V76–V80 | Sin el alcance SYSTEM, un `DO` no ve filas y el `DROP` borra datos | La línea SYSTEM obligatoria y una prueba con una fila sembrada |

### 11.4 Códigos de error: tabla única

| Código | Estado | Clave del servidor | Frontend (store / clave) | Slice dueño |
|---|---|---|---|---|
| `FUND_NOT_FOUND` | 404 | ya existe | `funds.store` / `fleet.funds.errors.not-found` | FUN-02 · FE-FUNDS-04 |
| `FUND_CONFLICT` (reemplaza `FUND_CODE_TAKEN`) | 409 | `error.fund-conflict.message` | `fleet.funds.errors.code-conflict` | FUN-02 · FE-FUNDS-04 |
| `FUND_INACTIVE` | 422 | ya existe | `fleet.profile.fund-assignments.errors.fund-inactive` | FE-FUNDS-07 |
| `FUND_CODE_LOCKED` | 422 | `error.fund-code-locked.message` | `fleet.funds.errors.code-locked` | FUN-09 · FE-FUNDS-11 |
| `COMPANY_INACTIVE` | 422 | ya existe | `…errors.company-inactive` | FUN-02, SUP-02 · FE-FUNDS-04, FE-PROC-SUP-01 |
| `VEHICLEFUNDASSIGNMENT_NOT_FOUND` | 404 | `error.vehiclefundassignment-not-found.message` (solo FUN-02) | `…fund-assignments.errors.not-found` | FUN-02 · FE-FUNDS-07 |
| `VEHICLEFUNDASSIGNMENT_CONFLICT` (reemplaza `VEHICLE_FUND_ASSIGNMENT_OVERLAP`) | 409 | `error.vehiclefundassignment-conflict.message` | `…errors.overlap` | FUN-02 · FE-FUNDS-07 |
| `VEHICLEFUNDASSIGNMENT_COMPANY_MISMATCH` | 422 | nueva | `…errors.company-mismatch` | FUN-02 · FE-FUNDS-07 |
| `VEHICLEFUNDASSIGNMENT_ALREADY_CLOSED` | 422 | nueva | `…errors.already-closed` | FUN-02 · FE-FUNDS-07 |
| `VEHICLEFUNDASSIGNMENT_END_NOT_AFTER_START` | 422 | nueva | `…errors.invalid-period` | FUN-02 · FE-FUNDS-07 |
| `SUPPLIER_OUTDATED`, `PURCHASEORDERTEMPLATE_OUTDATED` | 422 | ya existen | sin cambio (se elige por código) | SUP-01 |
| `*_INACTIVE` de compras (empresa, marca, tipo, plantilla, proveedor, moneda, modelo) | 422 | nuevas | `procurement-*.errors.*-inactive` | SUP-02 · FE-PROC-SUP-01 |
| `TAXIDTYPE_NOT_FOUND`, `PURCHASEORDER_NOT_FOUND`, `PURCHASEORDER_PDF_NOT_FOUND` | 404 | renombradas | mapeadas antes, en F1b | SUP-02 · FE-PROC-SUP-01 |
| `SUPPLIER_TAX_ID_CHECK_DIGIT_INVALID` | 422 | nueva | `procurement-suppliers.errors.check-digit` | SUP-03 · FE-PROC-SUP-01 |
| Reglas del tipo de documento del proveedor | 422 `BUSINESS_RULE_VIOLATION` | genérica | `procurement-suppliers.errors.tax-id-rejected` | SUP-03 · FE-PROC-SUP-01 |
| `PURCHASEORDERTEMPLATEDOCUMENTTYPE_NOT_FOUND` | 404 | nueva clave | `purchase-order-templates.notifications.document-type-not-found` | TPL-09 · FE-PROC-06 |
| Subida de más de 10 MB | 413 genérico | genérica | `purchase-order-templates.notifications.document-size` | TPL-04 |

`SUPPLIER_TAX_ID_CONFLICT`, `PURCHASEORDERTEMPLATE_SERIES_CONFLICT` y `PURCHASEORDERTYPE_CODE_CONFLICT` llevan campo aunque su agregado tenga una sola regla única. Ya están publicados y mapeados, así que se anotan como excepciones a convenciones l.444 en DOC-PROC-SUP-01.

### 11.5 Consultas de producción agregadas

- **(a)** Se amplía: los roles de sede afectados por SEC-01, y los roles propios con cualquier código retirado (V62, V64 y compras).
- **(g)** Plantillas cuyo emisor (razón social, tipo y número de documento, teléfono, domicilio) difiere del de su empresa.

### 11.6 Documentación: slice DOC-DICT y los demás textos

**DOC-DICT** entrega los textos al responsable para que los valide, con una fecha por fase. Solo el responsable cambia el diccionario.

| Antes de | Textos |
|---|---|
| F2 | §5.6 y la matriz (D2: `crear/editar/desactivar_fondos`, sin `gestionar_fondos`, los conteos «Ciento un permisos» y los totales por categoría); §12.1 `fund_type` en minúsculas (DP-1) |
| P4 | §12.1, el código del fondo corregible (DP-12); la excepción a D-101 (DP-11) |
| P5c | La excepción a D-105 para el PDF enviado (DP-4); el emisor copiado y no editable en §9.2 (l.2293-2296), l.1563 y D-35 (DP-5); §2.7, la llave por contenido (DP-16); `attention_email` como candidato (DP-6) |
| P5e | §2.5 l.114 y D-80: la RLS derivada del país con rama SYSTEM, como excepción a la regla 5.1 (DP-8); `addresses.company_id` como candidato |
| P6a | §5.6, §9.2 y D-130 (D7, DP-2, DP-3); que escribir exige `compras.ver` (DP-18) |
| Cuando convenga | D-116, las exportaciones pendientes (DP-17); los candidatos `PURCHASEORDER_OUTDATED` y `attention_email`. `PURCHASEORDER_OUTDATED` se construyó el 29/10/2026 (sección 8), y su texto en el diccionario espera la validación del responsable |

**Skills** (las dos copias, idénticas y fuera de git):
- regla 5.1: la excepción de `addresses`;
- regla 6.4: el objeto inmutable escrito por su huella antes de confirmar;
- regla 1.2 del frontend: `mm-company-filter`;
- la búsqueda de `DELETE` de `architecture-review` deja de tener excepción en P7.

**Convenciones:**
- l.81 y l.115 (fondos, en DOC-FUN-01);
- l.443-444 (excepciones de códigos con campo, y la lista de carreras traducidas);
- l.446 (404 en las escrituras de plantillas, como está);
- §7.2: el 413 y el 422 de `_OUTDATED`;
- §3.1: la excepción se borra en TPL-12.

**Contextos:**
- `procurement.md`: l.100, l.110, PRC-R01, PRC-R02, PRC-R09 y NUEVO-A;
- `location.md:308`;
- `fleet.md`: DP-11.

**Retroalimentación (nuevo, RETRO-FUN-01, 1,5 h):** una entrada en `retroalimentacion/` para el compañero sobre fondos, con Error / Corrección / Próxima vez. Incluye la migración V66 sin coordinar, el literal `PERM_` repetido de la revisión del 19/10, los códigos sin `_CONFLICT`, `CompaniesStore` y `company-select` importados, las claves inexistentes de `es.json` y la pantalla sin `allowedActions`.

### 11.7 Esfuerzo de la versión 2

Las enmiendas suman unas 22 h:
- BE-IAM-01: 2 h
- emisor de punta a punta (BE-TPL-08 ampliado y FE-PROC-15): 6 h
- FE-FUNDS-11: 2 h
- pruebas agregadas: unas 4 h
- RETRO-FUN-01: 1,5 h
- DOC-DICT: 3 h
- ajustes de slices existentes: unas 3,5 h

SUP-11/12/13 pasan de opcionales a obligatorios. **Total de desarrollo: unas 250 h.** La única parte opcional es FE-PROC-08b (5 h).

### 11.8 Enmiendas de la verificación de la versión 2 (mandan también sobre 11.1 a 11.7)

**Emisor (DP-5), contrato completo:**
- **BE-TPL-08 (P5c):**
  - en los recursos de alta y de PATCH, `issuer*`, `address` y `addressChange` pasan a `@Nullable`, sin `@Size` ni `@Valid`, y se marcan deprecados;
  - el ensamblador no los lee ni los cuenta en `fields`;
  - el alta copia siempre el emisor y su domicilio desde `CompanyIssuer`;
  - se anulan el 400 por emisor nulo del JSON de TPL-09 y la parte `IssuerTaxIdPolicy` del JSON de TPL-08;
  - SUP-02 no toca las validaciones del emisor.
  - Pruebas: `createWithoutIssuerCopiesTheCompany`, `createWithADifferentIssuerIgnoresIt` y `patchWithMalformedIssuerFieldsIsIgnored`.
- **La respuesta (`PurchaseOrderTemplateResource`) conserva el emisor como lectura:** es la copia fija que imprime el PDF.
- **FE-PROC-15:**
  - una plantilla guardada muestra su propio emisor (la copia), aunque la empresa cambie;
  - `loadCompanyIssuer` se usa solo en la vista previa del alta;
  - el mock e2e copia el emisor al crear.
- **BE-TPL-15 (nuevo, P7, 1 h):** quita esos campos de los recursos de entrada, no de la respuesta. Prueba: un pedido que todavía los manda no falla. Fila en §6: «Emisor: el BE ignora (P5c) → el FE deja de mandar (FE-PROC-15) → el BE los quita del pedido (P7)».

**DP-18 corregida.** Se quita «la matriz no deja marcar…». Una regla por rol no garantiza nada: los permisos efectivos son la unión de roles y asignaciones, y bloquearía guardados legítimos. Queda así:
- las rutas de lectura de compras piden `compras.ver`;
- la lectura que sigue a una escritura no filtra;
- se documenta en §5.6, antes de F1a: «las pantallas de compras exigen `compras.ver`».
- Opcional en FE-IAM-01b: un aviso que no bloquea en la columna de un rol con escritura de compras y sin `ver`.

**FE-PROC-SEC-03 (nuevo, F1a, 2 h, P1):**
- `SUPPLIER_READERS`, `TEMPLATE_READERS`, `ORDER_READERS` y las tres entradas del menú de compras pasan a `{compras.ver}`;
- `common.errors.unexpected` en `es.json`;
- texto por operación y mapeo de `ACCESS_DENIED` y `COMPANY_NOT_FOUND` en el store de proveedores;
- specs de la guarda y del menú.

FE-PROC-SEC-09a toca solo las guardas de escritura y los botones. «Nuevo» sigue visible a propósito para un rol de sede o de marca, y responde 403 con su texto (§8).

**Publicación del frontend antes de P4:** todo slice del frontend que no depende de fondos sale con `git push origin <rama>:develop`:
- FE-PROC-SEC-03, FE-PROC-03, FE-BASE-01/02, FE-ORG-01/02, FE-IAM-01a, FE-PROC-01, FE-PROC-07, FE-PROC-15 y FE-PROC-SEC-09a;
- desde una rama basada en `origin/develop` y rebasada sobre el push anterior;
- después se fusiona en el `develop` local.

`feature/procurement` del frontend nace de `origin/develop`. P6a ya no depende de P4.

**BE-IAM-01 rehecho:** el servidor **conserva** los códigos retirados en vez de quitarlos. Dos de ellos (`seguridad.gestionar_colaboradores` y `gestionar_roles`) son privilegiados y reservan el rol (D-121).
- Al guardar: (pedidos − RETIRED) ∪ (vigentes del rol ∩ RETIRED).
- La comprobación de lo que tiene quien edita mira solo pedidos − RETIRED.
- Los retirados se filtran de la columna de la matriz y de `RoleViewFactory`. La copia de un rol toma el origen menos RETIRED.
- Solo una migración firmada por SYSTEM quita un retirado: V79 lo hace con los de compras, y los de seguridad quedan anotados como deuda.
- Pruebas:
  - ADMIN_ACCESOS guarda un rol propio con titulares y `gestionar_colaboradores`: 200, la reserva no cambia y el código sigue;
  - la copia sale sin el código.

**V77 (BE-LOC-ADDR) precisada:**
- la función `access_scope_reaches_country(country_id)` (SYSTEM, o una empresa visible en ese país), que también va al script de hand-over;
- `addresses_select`, `addresses_insert` y `addresses_update`, sin DELETE, con ENABLE y FORCE;
- `addresses` en un conjunto propio de `RowLevelSecurityMigrationTests`;
- prueba `correctingAnAddressOfAnotherCompanyTouchesNoRow`;
- las dos lecturas de location que necesitan todas las empresas (`CountryCommandServiceImpl.correctIsoCodes` y la marca «tiene direcciones» de `CountryViewFactory`) corren en `SystemAccessScope`: bloquear el país → entrar en SYSTEM → consultar → salir → releer el país. Prueba: una cuenta de empresa con `editar_paises` no corrige el ISO de un país con direcciones;
- los scripts locales que borran direcciones usan NO FORCE y luego FORCE;
- consulta previa **(h)**: direcciones cuyo país difiere del de la empresa dueña. Si hay filas, se detiene P5e.

**Otros ajustes:**
- **BE-FUN-03:** `FOR SHARE` nativo + `refresh` del fondo al ceder, con su prueba en `FundLockTests`.
- **BE-FUN-06:**
  - `registerFundAssignment` = `actsOnWholeCompanyBrand(registrar_cesion)` ∧ `actsOn(fondos.ver, empresa)` ∧ ninguna cesión con fin vacío o posterior a hoy;
  - «hoy» en la zona del país de la empresa de la unidad, con la prueba `todayIsTheCompanyDay`;
  - DOC-FUN-01 anota que una cesión con fin futuro bloquea la marca, como decisión conservadora.
- **SUP-10:** pruebas `twoAccountsWithTheSameVerShareTheExportWhateverTheyWrite` y `aWriterWithoutVerInBDoesNotGetBInTheExport`.
- **11.4:** el 413 queda sin mapeo en el frontend (el formulario corta en 5 MB); DOC-TPL-01 nombra el código exacto.
- **FE-PROC-02:** las tres listas ofrecen solo las empresas con `compras.ver` (`offers = c => hasPermissionIn(compras.ver, c.id)`), con un spec.
- **Limpieza de V75:** un script local de una sola vez, con NO FORCE/FORCE en `role_permissions`, que termina en ROLLBACK salvo con `do_commit=1`, para la base del compañero.
  - `erp_mm_db_verify` ya está limpia (26/10, en V74).
  - `erp_mm_db` no tiene la V75 (comprobado): no hace falta resetearla mientras no se arranque el backend desde el repo principal.

**Documentación con dueño:**

| Dueño | Lugares |
|---|---|
| DOC-DICT | DP-18 antes de F1a; D-116 (DP-17) antes de P5d; en P5c, además, l.2295-2297, l.2323, l.1543 y D-75 l.3122; D-125 (seis contextos exportan) |
| DOC-PROC-SEC-01 (nuevo, P6a, 1 h) | convenciones §2.2 l.51 y §3.4 (D7), procesos-y-pantallas l.263, orden-de-implementación l.209 |
| DOC-TPL-01 | §3.3 l.126, §3.4 l.208, PRC-R05 y los pasos 2-3 de F2 en `procurement.md` |
| BE-LOC-ADDR | reglas 5.1 y 5.7 (las dos copias), patrones l.113-122, `location.md:308`, mapa l.230 y l.242, orden l.284 |
| BE-TPL-06 | regla 6.4 |
| BE-PROC-SEC-06 | regla 5.4, PRC-R02, PRC-R09 |
| FE-ORG-02 | frontend-rules §6 l.251 y casilla l.259 |
| DOC-FUN-01 | mapa l.185, procesos l.302-303, `fleet.md` (DP-11) |

**Esfuerzo:** +10 h. **Total de desarrollo: unas 260 h.**

---

## 12. Estado de la implementación

Estado y guía de publicación: ver la sección 13.

Al 26/10/2026, los textos de la documentación están aplicados, sin commit, para la validación del responsable: el diccionario v1.31 (D-130 enmendada con los ocho permisos de compras, D-133 y D-134), las convenciones de la API, los patrones de persistencia, los contextos `fleet`, `procurement` y `location`, el mapa de contextos, procesos y pantallas, el orden de implementación y la [retroalimentación de fondos](retroalimentacion/2026-10-26-fondos.md). El segundo pase sumó lo que dependía de D7 y el retiro del `DELETE` de convenciones §3.1.

Esos textos se publicaron el 27/10/2026 en `master` de la documentación: `8c4c165` (el cierre de la auditoría, D-133 y D-134) y `427589a` (los hallazgos de órdenes y plantillas cerrados ese día); después, `4da4453` (la segunda revisión de órdenes y plantillas), y `b02e6b3` y `f80a9e1` (D-135). Lo que siguió, del 28 al 31/10/2026, está en la sección 13.6.

---

## 13. Estado final y guía de publicación

### 13.1 Estado al 27/10/2026

Las fases F1a a F6 están implementadas, más una ronda F7 con lo que dejaron la revisión global, las pruebas en vivo y dos revisiones adversarias, y una limpieza del código que ya no se usaba. Todo está publicado: las doce etapas de 13.2 salieron el 27/10/2026, `develop` quedó en `471021cd` (backend) y `92c447e7` (frontend), y el responsable confirmó producción sin problemas. `feat/fleet-funds` se adelantó hasta `develop` en los dos repos.

| Repo | `develop` local | Pruebas en esa punta |
|---|---|---|
| Backend | `471021cd` (129 commits sobre `origin/develop`) | Suite completa en `erp_mm_db_verify` (V81): 2632 / 0 |
| Frontend | `5c1c3c2e` (142 commits sobre `origin/develop`) | Unitarias 2262 / 0, stylelint y `ng build` limpios, e2e completo 1276 pasan, 32 omitidas por diseño, 0 fallan |

**`develop` no se publica entero con un `git push`**: eso saltaría el orden de las etapas y las consultas previas. Se publica etapa por etapa, por su hash (13.2).

~~La documentación queda sin commit hasta que el responsable la valide (sección 12).~~ Publicada el 27/10/2026 (sección 12).

### 13.2 Etapas

Cada etapa es un prefijo de la siguiente y se publica como avance directo (`fast-forward`) sobre `develop`, que despliega en cada push. Ninguna migración se edita después de la etapa que la publica por primera vez: se comprobó el sha256 de cada una en cada etapa.

| Orden | Push | Commit | Contenido | Migraciones | Comprobado en la etapa |
|---|---|---|---|---|---|
| 1 | BE B1 (P1) | `f1cc7f8a` | Carril urgente: fuga entre empresas de proveedores, PDF del borrador, PATCH 500, subidas de 1–5 MB, WebP, reactivar plantillas, búsqueda por número | — | Compila; pruebas de la rama de origen |
| 2 | FE S1 | `88d2497` | Lo publicable del frontend sin fondos ni contratos nuevos | — | — |
| 3 | BE B2 (P3) | `8d7c0af1` | Fondos completos (F2 y F3) | V75 (final) | — |
| 4 | FE S2 | `1cd6c3c` | Pantallas de fondos | — | Unitarias 2193 / 0; e2e verde |
| 5 | BE B3 (P5) | `169ba289` | Compras compatible (F4) | V76, V77, V78 | Compila; unitarias 760 / 0 |
| 6 | FE S3 | `6025aa5` | Plantilla guardada con sus archivos (`removedDocuments`) | — | Unitarias 2201 / 0; e2e verde |
| 7 | BE B4 (P6a) | `92d0593e` | Ocho códigos finos de compras y `PATCH …/status` | V79 | Compila; unitarias 760 / 0 |
| 8 | FE S4, **enseguida** | `95c40dc` | El frontend pide los códigos finos y usa la ruta de estado | — | Unitarias 2202 / 0; e2e de compras, accesibilidad y visual verdes |
| 9 | FE S5 | `304a1a6` | Exportar proveedores, la descarga del PDF de una orden (roto en producción: 406), la bitácora de plantillas con `valuesWithheld`, el código de error del envío y el resto de F7 sin la corrección del código del tipo | — | Unitarias 2263 / 0; e2e 842 / 0 |
| 10 | BE B5 (P7) | `471021cd` | Contracciones (revoca los gruesos, quita el `DELETE`, el emisor y `isActive` de la entrada), deuda de capas, F7 del backend y la limpieza del código sin uso | V80, V81 | Suite completa 2632 / 0 |
| 11 | FE S6, **después de B5** | `5c1c3c2e` | La corrección del código del tipo de orden (DP-44, DP-46), la prueba en vivo de seguridad y la limpieza del código sin uso, que quita los mapeos de los códigos que solo emitía el backend anterior | — | Unitarias 2262 / 0; e2e completo 1276 / 0 |
| 12 | FE S7, después de S6 | `92c447e7` | Los tres pendientes menores: la matriz probada contra un error de lectura que quedaba, `CompanyUnitOption` sin el `code` que nadie leía, y las dos sondas en vivo entre empresas que ahora preparan su empresa, sus cuentas y su administrador por la API | — | Unitarias 2264 / 0; sonda en vivo 49 / 0, sin omitidas |

Los pushes se hacen desde los repos principales: `git -C erp-millennium-mobility-backend push origin <commit>:refs/heads/develop` y `git -C erp-millennium-mobility-frontend push origin <commit>:refs/heads/develop`. Si `origin/develop` avanzó por otro lado, el push se rechaza: se detiene la publicación y se rebasa la etapa, sin forzar nada.

### 13.3 Condiciones de cada push

- **Antes de B1:** consulta (e), los tipos de archivo guardados en plantillas. Si hay WebP o PDF, B1 ya los dibuja o los avisa (DP-7). Revisión manual tras el despliegue: proveedores (403/404), subir 3 MB, PDF con WebP y reactivar una plantilla.
- **Antes de B2:** ninguna base de producción tiene la V75 vieja (nunca se publicó). La base del compañero sí: la resetea con el anexo de la [retroalimentación del 26/10/2026](retroalimentacion/2026-10-26-fondos.md) antes de arrancar desde `develop`.
- **Antes de B3:** consultas (c), (d), (f), (g) y (h).
  - Con (c) o (h) con filas, V77 o V78 detienen el despliegue nombrando las filas: se corrige el dato con tu visto bueno.
  - (d) muestra proveedores fuera de D6 o con documento inválido.
  - (g) muestra plantillas cuyo emisor difiere del de su empresa: decides si se respetan, porque la copia ya no es editable (DP-5).
  - V76 a V78 bloquean sus tablas con `lock_timeout` de 5 s; publícalo fuera de horas de carga.
- **Antes de B4:** consulta (a). V79 remarca los roles propios con los finos (DP-3). **S4 sale inmediatamente después de que B4 esté desplegado**: entre los dos, la matriz de producción no ofrece los gruesos y las pantallas todavía piden los gruesos, que siguen autorizando.
- **Antes de B5:**
  - S5 desplegado, y las pestañas abiertas con el frontend anterior recargadas: B5 rechaza con 400 un PATCH de plantilla que todavía trae `isActive`.
  - Consulta (b): ninguna plantilla con `supplier_id` o `supplier_contact_id` no nulo. Si la hay, V81 se detiene sin borrar nada.
  - Consulta (a) otra vez. V80 revoca los gruesos en todos los roles; antes marca los finos solo al rol propio que tiene un grueso vigente y nunca tuvo ningún fino de ese grupo, ni vigente ni revocado (DP-43).
  - Desde B5, enviar una orden cuya plantilla tiene un logo o una firma que no se puede imprimir responde 422 y la orden queda en borrador (DP-45). La consulta (e) dice si hay archivos guardados antes de la validación de la subida.
- **Antes de S6:** B5 desplegado. Contra un backend anterior, la corrección del código de un tipo diría que no se corrigió.
- **Si un `DO` falla en producción:** el despliegue se detiene, la migración no se edita, se corrige el dato y se vuelve a desplegar (sección 3).

### 13.4 Después de publicar

- **Repos principales:** el `develop` local de los dos repos ya está en la punta final. Si arrancas el backend desde el repo principal, Flyway lleva `erp_mm_db` hasta V81 con las migraciones finales; esa base nunca tuvo la V75 vieja, así que no hace falta resetearla. Después de cada push, `git fetch` deja `origin/develop` en la etapa publicada; no hay nada que fusionar.
- **Compañero:** `origin/feat/fleet-funds` es ancestro de B2 y de S2. Después de B2 y S2, se adelanta por fast-forward al `develop` publicado en los dos repos (`git push origin <etapa publicada>:refs/heads/feat/fleet-funds`), y él resetea su base antes de arrancar.
- **Ramas y worktrees:** los worktrees de trabajo ya se quitaron. Las etapas no necesitan ramas, porque todas son ancestros de `develop`. Al 29/10/2026, `feat/fleet-funds` ya no existe en `origin`, y los dos repos quedan con `develop` y `master`.
- ~~**Documentación:** se hace commit cuando el responsable valide la v1.31 del diccionario y los demás textos.~~ Hecho el 27/10/2026 (sección 12).

### 13.5 Verificación final

- **Revisión global** contra los hallazgos originales (seis revisores, uno por área; cada defecto medio o alto, con dos escépticos): de unos 160 hallazgos, 132 cerrados, 13 superados por decisión y 5 abiertos o parciales, todos cerrados en F7. Ningún defecto medio o alto sobrevivió.
- **Pruebas en vivo**, con el backend real en `erp_mm_db_verify`, sin mock: ubicación, organización, seguridad, compras y fondos. La primera corrida encontró un defecto de producción, la descarga del PDF de una orden (406), corregido en S5. Las corridas siguientes, antes y después de la limpieza, sin defectos de producto. La de la punta final (backend `471021cd`, frontend `5c1c3c2e`): 47 pasan, 0 fallan y 2 omitidas, por falta de datos (D-82) y por configuración (la cuenta de un administrador de empresa). El log del backend, sin errores.
- **Revisiones adversarias de F7**, una del backend y otra del frontend (tres ángulos cada una, con dos escépticos por hallazgo): la carrera del código del tipo (DP-44), el corte de V80 por la hora (DP-43), el envío sin la firma (DP-45) y el frontend que habría dicho «actualizado» contra un backend anterior (DP-46), todos corregidos.
- **Código sin uso**: buscado por módulo en los dos repos y confirmado por un escéptico antes de borrar. Se quitaron 60 piezas del backend y unas 95 del frontend. Lo que DESIGN.md o las skills documentan como sistema de diseño se conserva aunque hoy no tenga uso: `mm-metric-card`, `mm-filter-layout`, las clases `.mm-type-*`, `.mm-span-mobile-*`, `.mm-page-actions`, `.mm-form-screen`, las opciones de `ToastOptions` y los métodos genéricos de `BaseApiEndpoint` y `BaseAssembler`. Quitarlos es una decisión del responsable.

### 13.6 Después del cierre: del 28 al 31/10/2026

Lo que siguió al cierre. Lo del 28 y el 29/10/2026 está publicado en `origin/develop`: el backend hasta `5ec83065` y el frontend hasta `bda96499`. Lo del 30/10/2026 está en `develop` y se publicó el 30/10/2026: el backend hasta `a54ee29b`, con V86, y el frontend hasta `88484481`; el responsable aprobó ese día D-136 a D-142 en el diccionario v1.33. Lo del 31/10/2026 está en `develop`, sin migraciones: el backend hasta `1e207eb7` y el frontend hasta `f0ee86aa`. Cambia filas de este plan, que quedan como estaban, con su historia, y se leen con esto:

| Qué | Commits | Lo que cambia de este plan |
|---|---|---|
| La sesión guardada ya no lleva el hash de la contraseña | `513bbc88` | Nada del plan; cierra el pendiente O58 de `iam` ([iam.md](contextos/iam.md), §6 b) |
| Fuera de `dev`, la aplicación no arranca si su cuenta de base es superusuario o tiene `BYPASSRLS` | `852214f3` | Refuerza la regla de las dos cuentas (sección 3) |
| V83: las siete tablas de compras con empresa, con las políticas de V29 en lugar de la política única de V63 a V74 | `6ebbe6b2` | Sigue al registro de la sección 5. V83 no abre con el alcance `SYSTEM` que pide la sección 3: solo reemplaza políticas y no lee ninguna fila |
| V84: `kept_files` con la empresa de cada archivo y Row Level Security; el `DO` se detiene si una llave no nombra ni una plantilla ni una orden enviada | `e97383b5` | La llave por contenido de DP-16 no cambia. El diccionario (§2.7) dice desde el 30/10/2026 que la tabla nombra la empresa de cada archivo (documentación `425c79b`) |
| El frontend vuelve a cargar Inicio de sesión desde el servidor cuando la sesión termina; la contraseña del inicio de sesión recupera «Ingrese su contraseña», por decisión del responsable del 28/10/2026 | `1950d50d`; `7381bfb5`, `52c47b30` | Nada |
| La orden nombra su moneda por su código ISO (`"currency": "PEN"`), por decisión del responsable del 29/10/2026 (D-138) | `ef147166`; frontend `9be57b15` | Nada; el contrato de la orden cambia en convenciones |
| El borrador que no pisa un cambio más nuevo: V85 y `422 PURCHASEORDER_OUTDATED`, por decisión del responsable del 29/10/2026 (D-138) | `a0201a41`; frontend `5656f985` | Cierra la fila de `PURCHASEORDER_OUTDATED` de la sección 8 y el candidato de 11.6 |
| Lo que una orden revisa activo se lee con `FOR SHARE`, y el contacto responde su propio código: `404 SUPPLIER_CONTACT_NOT_FOUND` o `422 SUPPLIER_CONTACT_INACTIVE`, en lugar de `PURCHASEORDER_CONTACT` | `914cac1a`, `77e1e28c`, `7af493e6` | Los `*_INACTIVE` de compras de 11.4 suman el contacto |
| Los decimales de una moneda quedan fijos desde su primera orden: `422 CURRENCY_DECIMAL_PLACES_IN_USE`, por el puerto `CurrencyUsage` de `location`, por decisión del responsable del 29/10/2026 (D-138) | `7b8db0ee`; frontend `f9a23d0b`, `da457b00` | Nada |
| La ficha del proveedor, con sus contactos por sus propias rutas (D-137); `PUT /suppliers/{id}` ya no los lleva, y `422 SUPPLIER_CONTACT_OUTDATED` | `2f50c904` a `5ec83065`; frontend `83812b7a` a `bda96499` | Nada del plan; el contrato del proveedor cambia en convenciones |
| V82: el año del modelo en cada línea de vehículo de una orden (D-135), del 27/10/2026 | `5242833b` | Sigue al registro de la sección 5 |
| 30/10/2026. El alta del proveedor ya no lee contactos: entran solo por `POST /suppliers/{id}/contacts` (D-137) | `fb73ba40`; frontend `51f35fa6` | Nada del plan; el contrato del proveedor cambia en convenciones (§2.2) |
| 30/10/2026. El inicio de sesión, el registro y dar acceso rechazan un correo de más de 255 caracteres con `400 VALIDATION_ERROR` en `email` | `726aeec0` | Nada del plan; cierra el pendiente O58 de `iam` ([iam.md](contextos/iam.md), §6 b, pendiente 5) |
| 30/10/2026. La exportación de las marcas sale de la API (D-136, que enmienda D-116) | `ec70a8aa`; frontend `67c2f7d9` | Nada |
| 30/10/2026. V86 y la búsqueda que pliega tildes: `fold_search_text(text)`, de la cuenta de Flyway, ejecutada por la cuenta de la aplicación y entregada por `hand-over-database.sql` (D-142) | `145fc104`, `a54ee29b` | Cierra la fila «Plegar tildes» de la sección 8. Sigue al registro de la sección 5 |
| 30/10/2026. Las pantallas de Catálogos se abren solo con `catalogos.ver` (D-141), la regla de DP-18 llevada a los catálogos | Frontend `71765c0b`, `b8a28260` | Extiende DP-18, que el plan fijó para compras y fondos |
| 30/10/2026. La hoja de la plantilla con las bandas de la hoja de la orden (D-140; DESIGN.md, revisión 32), y las otras dos preguntas de compras cerradas por D-140: el número de la orden mantiene su forma y la Atención no lleva correo | Frontend `4e641b3b`, `88484481` | Nada |
| 30/10/2026. La prueba del store de monedas dice que solo una orden de compra fija los decimales, y la sonda de compras sigue el contrato del 29/10/2026 | Frontend `ec7bd7cc`, `2d40e704` | Nada |
| 30/10/2026. Las cinco sondas en vivo contra ese código —backend `a54ee29b` en el 8090 sobre `erp_mm_db_verify`, porque `erp_mm_db_live` no existe, y frontend `88484481` en el 4300—: compras 16 de 16, fondos 7 de 7, ubicación 8 de 8, organización 10 de 10 y seguridad 9 de 9; 50 pasan, 0 fallan y 0 omitidas, sin arreglar ni las sondas ni la aplicación, y sin ningún `ERROR` en el registro del backend fuera de los dos rechazos que provoca la sonda de seguridad. Queda abierto un aviso menor de rendimiento, `NG0956`, en el formulario de la orden | — | Repite contra el código del 30/10/2026 las pruebas en vivo de 13.5, que en la punta final habían dado 47 pasan y 2 omitidas; ahora ninguna se omite ([sondas del 30/10/2026](retroalimentacion/2026-10-30-sondas.md)) |
| 31/10/2026. Las búsquedas de tipos, plantillas y órdenes de compra, probadas con y sin tildes (PRC-R28, D-142) | `1e207eb7` | Nada; completa la prueba de la fila de V86 |
| 31/10/2026. El formulario de la orden sigue sus líneas por una llave estable: un guardado conserva sus filas, y la consola ya no avisa `NG0956` | Frontend `16f48a7a` | Nada; cierra el aviso menor que dejaron las sondas del 30/10/2026 |
| 31/10/2026. Las sondas en vivo de los pasos 4, 5 y 6 —`vehiclecatalog`, `fleet` y `changelog`—, y la de `organization`, que vincula la cuenta de verificación antes de leer los colaboradores, así la salida y el regreso corren | Frontend `f0ee86aa` | Nada |
| 31/10/2026. Las ocho sondas en vivo contra ese código —backend `1e207eb7` en el 8090 sobre `erp_mm_db_verify`, porque `erp_mm_db_live` no existe, y frontend `f0ee86aa` en el 4300—, en dos corridas: bitácora 6, vehículos 8, fondos 7, ubicación 8, organización 10, compras 16, seguridad 9 y catálogos de vehículos 8; 72 pasan, 0 fallan y 0 omitidas en cada una, sin arreglar ni las sondas ni la aplicación. En el registro del backend, ningún `ERROR`; los `WARN`, solo los dos rechazos que provoca la sonda de seguridad, uno de PDFBox y dos de SpringDoc por sus valores por omisión | — | Repite las pruebas en vivo de 13.5 y suma las de catálogos de vehículos, vehículos y bitácora; ninguna se omite, y la salida y el regreso de `organization` corren ([sondas del 31/10/2026](retroalimentacion/2026-10-31-sondas.md)) |

Las migraciones nuevas, V82 a V86, abren con `lock_timeout` de 5 s, y V82, V84 y V85, también con el alcance `SYSTEM`. V86 solo crea una función y le da permiso: no lee ninguna fila ni bloquea una tabla.
