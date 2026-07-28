## Ejercicio de JOINs SQL (MiniStore)

## Consulta 1: LEFT JOIN

### ¿Por qué usé LEFT JOIN y no INNER JOIN? ¿Qué se perdería con INNER JOIN?

Usé `LEFT JOIN` porque la pregunta de negocio era "¿qué productos del catálogo nunca fueron vendidos?". Para responder eso, necesito que la tabla `productos` se mantenga completa en el resultado, sin importar si tiene o no ventas asociadas. `LEFT JOIN` garantiza justamente eso: trae todas las filas de la tabla izquierda (`productos`) y, cuando no hay coincidencia en `ventas`, completa esas columnas con `NULL` en vez de descartar la fila.

Si hubiera usado `INNER JOIN`, la consulta solo devolvería las filas donde existe coincidencia en ambas tablas. En mi base, los productos **Hub USB-C 7p** (producto_id 108) y **Parlante Bluetooth** (producto_id 109) no tienen ninguna venta cargada en la tabla `ventas`, así que con `INNER JOIN` esas dos filas directamente desaparecerían del resultado. Perdería la información que necesitaba: no podría identificar qué productos del catálogo no se vendieron, porque esos productos ni siquiera aparecerían en la salida.

## Consulta 2: RIGHT JOIN

### ¿Por qué usé RIGHT JOIN? ¿Qué tabla está a la izquierda y cuál a la derecha?

En mi consulta:

```sql
FROM productos p
RIGHT JOIN ventas v ON p.producto_id = v.producto_id
```

`productos` está a la izquierda y `ventas` está a la derecha. Usé `RIGHT JOIN` porque acá la pregunta de negocio se invierte respecto a la Consulta 1: ahora necesito que la tabla que se conserve completa sea `ventas`, no `productos`. El objetivo era detectar ventas registradas cuyo `producto_id` no existe en el catálogo (posibles errores de carga). Con `RIGHT JOIN`, todas las filas de `ventas` (la tabla derecha) se preservan sin importar si encuentran o no coincidencia en `productos`, y cuando no hay match, las columnas de `productos` quedan en `NULL`.

En mi base de datos, la venta con `venta_id = 10` tiene `producto_id = 999`, un valor que no existe en la tabla `productos` (fue insertado a propósito en el schema como caso de error de carga). Por eso `nombre` y `precio` aparecen como `NULL` en esa fila.

## ¿Qué representan los valores NULL en cada resultado?

Los `NULL` que aparecen en cada consulta representan la **ausencia de coincidencia** entre las tablas para esa fila puntual, no un dato vacío cargado en la tabla original.

**Consulta 1 - columnas de ventas en NULL:**
En mi resultado, el producto **Hub USB-C 7p** (producto_id 108) tiene `cantidad` y `total` en `NULL`. Esto no significa que la venta tenga cantidad cero, sino que **no existe ningún registro** en la tabla `ventas` con `producto_id = 108`. El `LEFT JOIN` no encontró ninguna fila de `ventas` que matchee, así que rellenó esas columnas con `NULL` para no perder la fila del producto. Lo mismo pasa con **Parlante Bluetooth** (producto_id 109).

**Consulta 2 - `nombre` y `precio` de productos en NULL:**
En mi resultado, la venta con `venta_id = 10` tiene `nombre` y `precio` en `NULL`. Esto significa que existe una venta cargada en el sistema (producto_id 999, cantidad 1, fecha 2024-03-25), pero **ese producto_id no existe en la tabla `productos`**. Es decir, alguien registró una venta de un producto que no está dado de alta en el catálogo, probablemente un error de carga de datos (producto mal tipeado, eliminado del catálogo, o cargado antes de crear el producto correspondiente en la tabla `productos`).

## ¿Cuándo usaría FULL OUTER JOIN en un caso real de negocio?

Usaría `FULL OUTER JOIN` en escenarios de **auditoría o control de calidad de datos**, donde necesito ver el panorama completo de dos tablas relacionadas sin perder ninguna fila de ninguna de las dos, y necesito identificar inconsistencias en ambas direcciones al mismo tiempo.

Un ejemplo real, más allá de este ejercicio: conciliar un sistema de facturación con un sistema de pagos. Con `FULL OUTER JOIN` podría detectar en una sola consulta tanto las facturas que nunca recibieron un pago (huérfanas de un lado) como los pagos registrados que no corresponden a ninguna factura existente (huérfanos del otro lado) — un error que en cualquiera de los dos sentidos podría significar plata perdida o mal registrada. En mi ejercicio, la Consulta 3 replica exactamente esa lógica sobre `productos` y `ventas`: en una sola vista veo ventas normales (con match en ambas tablas), la venta sin producto (producto_id 999) y los productos sin ventas (Hub USB-C 7p, Parlante Bluetooth), sin tener que correr tres consultas separadas.
