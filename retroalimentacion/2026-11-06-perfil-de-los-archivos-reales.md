# Perfil de los archivos reales de las cargas

**Fecha:** 06/11/2026 · **Origen:** primer paso del paquete de cargas (D-177; especificación de cargas, §1.3) · **Para:** el responsable y quien corra la carga en producción.

## Qué se pudo perfilar

En `Desktop/MILLENNIUM MOBILITY SAC/Proyecto ERP` solo está la plantilla `STOCK_MILLENNIUM_2026_EXPORTABLE FINAL.xlsx`. No están Flota Lizzo New, Flota Lima Bikes, el archivo del SAT ni la lista de garantía.

| Dato de la plantilla | Resultado |
|---|---|
| Hoja | `STOCK`, con la tabla `StockMillennium2026` de `A1:AZ2001` |
| Encabezados | Los 52 de §5.4, exactos. `N°` lleva el signo de grado, no el ordinal |
| Filas con datos | 0 de 2 000 preparadas |
| Formatos | Entero, texto, general, fecha y monto, como dice §5.4 |

## Qué no se pudo medir

Los puntos 2 a 21 de §1.3 necesitan filas reales, así que quedan sin medir:
- cuántas filas trae cada flota, y por eso el tope de filas se fijó con una medición sintética (§8.4), no con el archivo;
- los valores reales de `PROPIETARIO`, `DUPLICADO DE LLAVES`, `NC / ND` y `OPERACIÓN DE PAGO`;
- cómo se escriben los proveedores, las marcas, los modelos y los colores frente al catálogo;
- si `EN PLANTA` dice `SI` en las unidades entregadas. Una entregada con `NO` y GPS falla con `VEHICLEEQUIPMENT_NOT_ARRIVED`, y el informe lo dice;
- los encabezados de §5.2 y §5.3, del SAT y de la garantía, que siguen provisionales.

## Con qué se probó en su lugar

Con libros hechos desde la plantilla real del cliente: su hoja, sus estilos, su tabla y sus encabezados, con filas sintéticas escritas en sus celdas (`XlsxFixtures.stock`). Las filas malas son las de §8.3 que caben en una prueba: fecha imposible, celda obligatoria vacía, VIN repetido, TC de otra moneda, modelo inexistente, proveedor desconocido, número sin formato, nota de débito, nota que deja la unidad en cero, bloque con dos montos.

## Qué hacer con los archivos reales

1. Pasar el archivo por **Revisar**: la lista de precarga no escribe nada (D-226).
2. Contrastar las unidades con `ENTREGADO=SI` con el registro de clientes.
3. Revisar la tabla de traspasos: un `PROPIETARIO` mal escrito, como `MILLENIUM`, aparece ahí como si fuera de otro dueño.
4. Si el archivo pasa del tope, partirlo sin separar una factura, una nota ni un bloque de pago: la nota y el pago necesitan sus facturas en el mismo archivo o ya registradas.
