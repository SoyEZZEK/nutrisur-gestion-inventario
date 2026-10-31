# Historias de usuario

Módulo de Gestión de Inventario y Reposición - Nutrisur S.A.

Las historias siguen el formato "Como [usuario], quiero [funcionalidad], para [beneficio]" y se gestionan en el tablero de Jira del proyecto (claves NUT-1 a NUT-3).

## HU1 - Alerta de reposición (NUT-1)

**Prioridad:** alta

Como encargado de Compras, quiero recibir una alerta automática cuando un producto llegue a su punto de pedido, para reponer antes de que haya quiebre de stock.

**Criterios de aceptación**

- La alerta se emite cuando `stock_actual` es menor o igual a `punto_pedido`.
- Muestra el producto, el stock actual y el proveedor habitual.
- No se repite mientras haya una orden de compra pendiente para ese producto.

**Actividades del proceso BPMN:** B1 y B2.

## HU2 - Orden de compra sugerida (NUT-2)

**Prioridad:** alta

Como encargado de Compras, quiero que el sistema sugiera una orden de compra a partir del histórico de ventas, para aprobarla o rechazarla sin calcular las cantidades a mano.

**Criterios de aceptación**

- La cantidad sugerida se calcula con los movimientos de tipo EGRESO.
- Puedo aprobar o rechazar la orden.
- Al aprobar, se envía al correo del proveedor.

**Actividades del proceso BPMN:** B3, B4 y B5.

## HU3 - Recepción de mercadería (NUT-3)

**Prioridad:** media

Como personal de Depósito y Stock, quiero registrar la recepción de mercadería contra la orden de compra, para que el stock se actualice y queden registradas las diferencias.

**Criterios de aceptación**

- Se genera un movimiento INGRESO con el número de remito como referencia.
- Si la cantidad recibida difiere de la pedida, se registra el ingreso por la cantidad realmente recibida y la diferencia queda en la observación.
- `stock_actual` se actualiza solo, sin edición manual.

**Actividades del proceso BPMN:** B8, B9 y B10.

## Definición de "Listo"

Una historia se considera lista cuando sus actividades están modeladas en el diagrama BPMN y la documentación correspondiente está integrada al repositorio mediante un Pull Request.
