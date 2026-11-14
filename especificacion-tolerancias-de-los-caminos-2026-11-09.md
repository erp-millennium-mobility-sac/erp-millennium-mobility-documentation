# Especificación: Tolerancias de los caminos de estado

**Versión 2 · 13/11/2026.** Suma la sección 6, un borrador con horas propuestas para que el negocio lo corrija en vez de llenar las tablas en blanco: no es una decisión, y ninguna migración lo aplica hasta que el negocio responda. El código del camino de la firma es `FIRMA_CONTRATO` desde V130 (D-251). Antes, **versión 1 · 09/11/2026.** Decisión del responsable: D-240 del diccionario, con los caminos de En traspaso de D-239. Lee de §8.2 y §8.3 del diccionario y de las migraciones de `fleet` que siembran los caminos: V47, V51, V96, V112, V117 y, desde la versión 2, V130.

## 1. Para quién es y qué se pide

**Es para el negocio:** quien sabe cuánto debe durar cada situación de una unidad.

**Se pide, para cada camino de la tabla 3,** cuántas horas se tolera la unidad en el estado al que llega por ese motivo, y qué pasa cuando se pasan. Y, para los cuatro de la tabla 4, que ya tienen una, confirmarla o corregirla.

**Para qué.** Con las respuestas, una migración de `fleet` fija las tolerancias, así toda base nueva nace con ellas (D-240). Después el superadministrador las ajusta en Catálogos › Estados y motivos. Un cambio vale para los movimientos siguientes: el vencimiento de cada movimiento se calcula al registrarlo, y el historial no se reescribe.

**Cómo responder.** En esta tabla o en una copia: las horas y la acción de cada fila. Una fila que queda vacía sigue sin tolerancia.

## 2. Qué hace cada respuesta

Las horas se cuentan desde el movimiento, corridas: también de noche y en feriados. 72 h son 3 días; 720 h, 30 días.

| Al vencer | Qué hace hoy |
|---|---|
| Mover sola | El proceso programado devuelve la unidad al estado que se indique, y firma el movimiento el usuario del sistema. Hoy solo lo hace la firma de contrato: a las 48 h vuelve a Disponible |
| Avisar | La unidad aparece como vencida en Inventario y en los indicadores de Reportes, que la cuentan también como por vencer 72 h antes (D-235). Nadie la mueve |
| Sin tolerancia | La unidad nunca aparece como vencida |

**Si es «mover sola», diga a qué estado vuelve.** No puede volver a Disponible: entrar ahí exige la habilitación completa, y solo el regreso de la firma de contrato lo hace solo (D-164, D-170).

**Dos casos que conviene saber.**
- **Inactivo es final.** No tiene salida, así que una tolerancia ahí vencería en toda unidad que sale de la flota y la dejaría vencida para siempre. Son las filas 30, 32 y 37.
- **Activo es la unidad con su cliente.** Una tolerancia en un camino que llega a Activo la mostraría vencida mientras genera ingresos. Son las filas 3, 19, 21 y 24.

## 3. Caminos sin tolerancia

Los manuales que hoy no tienen tolerancia, más los cuatro nuevos de En traspaso (D-239). En las filas 34 a 36, D-239 ya fijó «avisar»: falta cuánto tarda la notaría.

| N.º | Origen → Destino | Motivo (código y nombre) | Tolerancia actual | Horas (a llenar) | Al vencer (avisar / mover sola, a llenar) |
|---|---|---|---|---|---|
| 1 | En habilitación → Disponible | `HAB_COMPLETA` · Habilitación completa | Sin tolerancia | | |
| 2 | Disponible → En habilitación | `PIERDE_REQUISITO` · Pierde un requisito | Sin tolerancia | | |
| 3 | Contrato firmado → Activo | `ENTREGA_CLIENTE` · Entrega al cliente | Sin tolerancia | | |
| 4 | Activo → Mecánica | `FALLA` · Falla | Sin tolerancia | | |
| 5 | Activo → Mecánica | `ALERTA_SISTEMA` · Alerta de sistema | Sin tolerancia | | |
| 6 | Activo → Mecánica | `KILOMETRAJE` · Kilometraje | Sin tolerancia | | |
| 7 | Activo → Mecánica | `TIEMPO` · Tiempo | Sin tolerancia | | |
| 8 | Activo → Siniestro | `ACCIDENTE` · Accidente | Sin tolerancia | | |
| 9 | Activo → Siniestro | `ROBO_FRUSTRADO` · Robo frustrado | Sin tolerancia | | |
| 10 | Activo → Siniestro | `DANOS_TERCEROS` · Daños a terceros | Sin tolerancia | | |
| 11 | Activo → Custodia | `INCAUTACION` · Incautación | Sin tolerancia | | |
| 12 | Activo → Custodia | `COMISARIA` · Comisaría | Sin tolerancia | | |
| 13 | Activo → Custodia | `DEPOSITO` · Depósito | Sin tolerancia | | |
| 14 | Activo → Custodia | `DEVOLUCION_VOLUNTARIA` · Devolución voluntaria | Sin tolerancia | | |
| 15 | Activo → Custodia | `OTROS_INCUMPLIMIENTOS` · Otros incumplimientos | Sin tolerancia | | |
| 16 | Activo → Robado | `ROBO` · Robo | Sin tolerancia | | |
| 17 | Activo → Robado | `AUTORROBO` · Autorrobo | Sin tolerancia | | |
| 18 | Activo → Robado | `FRAUDE` · Fraude | Sin tolerancia | | |
| 19 | Mecánica → Activo | `MANT_FINALIZADO` · Mantenimiento finalizado | Sin tolerancia | | |
| 20 | Mecánica → Puesta a punto | `NO_ASUMIDA` · No asumida | Sin tolerancia | | |
| 21 | Siniestro → Activo | `REPARACION_FINALIZADA` · Reparación finalizada | Sin tolerancia | | |
| 22 | Siniestro → Puesta a punto | `NO_ASUMIDA` · No asumida | Sin tolerancia | | |
| 23 | Siniestro → Pérdida total | `PERDIDA_TOTAL` · Pérdida total | Sin tolerancia | | |
| 24 | Custodia → Activo | `CLIENTE_REGULARIZA` · Cliente regulariza | Sin tolerancia | | |
| 25 | Custodia → Puesta a punto | `RESOLUCION_CONTRATO` · Resolución de contrato | Sin tolerancia | | |
| 26 | Custodia → Robado | `ROBO` · Robo | Sin tolerancia | | |
| 27 | Custodia → Robado | `AUTORROBO` · Autorrobo | Sin tolerancia | | |
| 28 | Custodia → Robado | `FRAUDE` · Fraude | Sin tolerancia | | |
| 29 | Robado → Puesta a punto | `RECUPERADO` · Recuperado | Sin tolerancia | | |
| 30 | Robado → Inactivo | `BAJA_DEFINITIVA` · Baja definitiva | Sin tolerancia | | |
| 31 | Pérdida total → Puesta a punto | `RECUPERA_VALOR` · Recupera valor | Sin tolerancia | | |
| 32 | Pérdida total → Inactivo | `BAJA_DEFINITIVA` · Baja definitiva | Sin tolerancia | | |
| 33 | Puesta a punto → Disponible | `REACONDICIONADA` · Reacondicionada | Sin tolerancia | | |
| 34 | Activo → En traspaso | `CONTRATO_COMPLETADO` · Contrato completado | Nuevo (D-239) | | Avisar (D-239) |
| 35 | Activo → En traspaso | `VENTA_DIRECTA` · Venta directa | Nuevo (D-239) | | Avisar (D-239) |
| 36 | Disponible → En traspaso | `VENTA_DIRECTA` · Venta directa | Nuevo (D-239) | | Avisar (D-239) |
| 37 | En traspaso → Inactivo | `TRASPASO_INSCRITO` · Traspaso inscrito | Nuevo (D-239) | | |

## 4. Ya fijadas: confirmar

Las sembró V47 o V112. Se piden para confirmarlas o corregirlas.

| N.º | Origen → Destino | Motivo (código y nombre) | Tolerancia actual | Horas (a llenar) | Al vencer (avisar / mover sola, a llenar) |
|---|---|---|---|---|---|
| A | Disponible → Contrato firmado | `FIRMA_CONTRATO` (`VENTA` hasta V130) · Firma de contrato | 48 h, mover sola a Disponible | | |
| B | Activo → Mecánica | `MANT_PREVENTIVO` · Mantenimiento preventivo | 72 h, avisar | | |
| C | Activo → Siniestro | `CHOQUE` · Choque | 480 h (20 días), avisar | | |
| D | Activo → Custodia | `MOROSIDAD` · Morosidad | 720 h (30 días), avisar | | |

## 5. Lo que no se pide, y por qué

- **Las altas:** «Registro de compra» y los tres caminos de «Migración inicial» (V117). El tiempo de una unidad nueva hasta su entrega ya lo mide la brecha de días (D-199), y una tolerancia en la carga inicial contaría desde el día de la carga, no desde el hecho.
- **El regreso automático** de Contrato firmado a Disponible (`REGRESO_AUTOMATICO`, V51): no lleva tolerancia, por regla (D-110).
- **Activo → Mecánica por «Choque»:** lo cerró D-221; el choque va a Siniestro (fila C).
- **Los tres caminos a Inactivo que cierra D-239:** Activo → Inactivo por «Contrato completado» y «Venta directa», y Disponible → Inactivo por «Venta directa». Los reemplazan las filas 34 a 37.

Con ellos, las migraciones suman 46 caminos: 41 sin tolerancia y 5 con ella. La tabla 3 lleva los 33 manuales que siguen abiertos y los 4 nuevos; la tabla 4, los 4 abiertos que ya la tienen.

## 6. Borrador para corregir (versión 2, 13/11/2026)

**Es una propuesta, no una decisión.** Se armó con la escala de las cuatro tolerancias que ya existen (72 h, 480 h y 720 h) para que el negocio corrija números en lugar de inventarlos. Cada fila se puede aceptar, cambiar o dejar sin tolerancia. Cuando el negocio responda, una migración de `fleet` fija lo aceptado.

**Por qué conviene responder antes de cargar unidades reales.** El vencimiento de cada movimiento se calcula al registrarlo, y el historial no se reescribe (§2). Una unidad que entró a Mecánica antes de que su camino tenga tolerancia nunca aparece como vencida, hasta que se mueva otra vez.

**Siete filas quedan sin tolerancia por diseño**, como explica §2: las que llegan a Activo (3, 19, 21 y 24), porque la unidad está generando ingresos con su cliente, y las que llegan a Inactivo (30, 32 y 37), porque es final. Las demás «avisan»: ninguna puede «mover sola» a Disponible (D-164, D-170), y fuera de la firma de contrato no hay otro regreso que tenga sentido automático.

| Filas | Caminos | Propuesta | Por qué |
|---|---|---|---|
| 1 y 33 | → Disponible por «Habilitación completa» y «Reacondicionada» | 720 h (30 días), avisar | Una unidad lista que no se coloca en un mes es stock parado: lo dicen Inventario y los indicadores |
| 2 | Disponible → En habilitación por «Pierde un requisito» | 72 h, avisar | Un GPS o un documento que se reemplaza no debería tardar más de tres días |
| 4 a 7 | Activo → Mecánica por «Falla», «Alerta de sistema», «Kilometraje» y «Tiempo» | 120 h para «Falla»; 72 h para las otras tres, como «Mantenimiento preventivo» | Una unidad en el taller es una unidad sin ingresos; una falla puede pedir repuestos |
| 8 a 10 | Activo → Siniestro por «Accidente», «Robo frustrado» y «Daños a terceros» | 480 h (20 días), avisar, como «Choque» | Mismo tipo de trámite que el choque |
| 11 y 13 | Activo → Custodia por «Incautación» y «Depósito» | 720 h (30 días), avisar | Una autoridad la tiene: Millennium no decide cuándo vuelve, pero alguien tiene que revisarla cada mes |
| 12 | Activo → Custodia por «Comisaría» | 168 h (7 días), avisar | Suele resolverse en días |
| 14 | Activo → Custodia por «Devolución voluntaria» | 168 h (7 días), avisar | Millennium decide: resolver el contrato o volver a entregarla |
| 15 | Activo → Custodia por «Otros incumplimientos» | 720 h (30 días), avisar, como «Morosidad» | Mismo tipo de caso que la morosidad |
| 16 a 18 y 26 a 28 | → Robado por «Robo», «Autorrobo» y «Fraude» | 720 h (30 días), avisar | Al mes alguien decide entre seguir buscando y la baja definitiva |
| 20, 22, 25, 29 y 31 | → Puesta a punto por «No asumida», «Resolución de contrato», «Recuperado» y «Recupera valor» | 168 h (7 días), avisar | Reacondicionar una unidad no debería tomar más de una semana |
| 23 | Siniestro → Pérdida total | 720 h (30 días), avisar | El trámite con la aseguradora |
| 34 a 36 | → En traspaso por «Contrato completado» y «Venta directa» | 720 h (30 días), avisar; D-239 ya fijó «avisar» | Lo que tarda la notaría; el área legal puede ajustarlo |
| 3, 19, 21, 24, 30, 32 y 37 | → Activo y → Inactivo | Sin tolerancia | §2 |
| A a D | Las cuatro que ya existen | Confirmar sin cambios | Nadie pidió cambiarlas |
