# Indicadores y estándares de control

Módulo de Gestión de Inventario y Reposición - Nutrisur S.A.

Los indicadores se derivan de las metas M1 a M4 de la Entrega 2 y del proceso de reposición de stock modelado en BPMN. Cada uno tiene un estándar contra el cual se compara la medición y una acción correctiva si hay desvío.

## Etapas del control

1. **Establecimiento de estándares:** el valor esperado de cada indicador (tabla siguiente).
2. **Medición del desempeño:** se mide en la fuente indicada, con la frecuencia indicada.
3. **Comparación:** se contrasta la medición con el estándar.
4. **Acción correctiva:** si hay desvío, se aplica la corrección indicada y se vuelve a medir.

## Indicadores

| ID | Indicador | Estándar | Medición | Acción correctiva |
|---|---|---|---|---|
| I1 | **Tiempo de respuesta** de una consulta de stock de un producto (percentil 95). Corresponde a la meta M1. | 2 segundos o menos | Pruebas de rendimiento (tarea P3), en el entorno de pruebas. Antes de la puesta en marcha y ante cada cambio relevante. Responsable: Tester. | Optimizar consultas e índices de la base de datos y repetir P3. |
| I2 | **Capacidad de carga:** productos activos y movimientos mensuales soportados con 10 usuarios simultáneos. Meta M2. | 5.000 productos y 100.000 movimientos por mes, sin superar el estándar de I1 | Pruebas de carga (tarea P3). Antes de la puesta en marcha. Responsable: Tester. | Revisar el diseño de datos (tarea D1) y las consultas del módulo. |
| I3 | **Calidad del software:** defectos críticos abiertos y porcentaje de casos de prueba de funciones críticas aprobados. Meta M3. | 0 defectos críticos abiertos y 90 % o más de casos aprobados | Registro de casos de prueba (tareas P2 y P4), al momento de la puesta en marcha. Responsable: Tester. | Corregir los defectos y repetir las pruebas de regresión (P4) antes de salir a producción. |
| I4 | **Quiebres de stock:** cantidad mensual de productos activos que llegan a `stock_actual` igual a 0, respecto del valor previo a la implantación. Meta M4. | Reducción de 30 % o más dentro de los 6 meses posteriores a la puesta en marcha | Conteo mensual a partir de los movimientos de stock, después de la tarea I4. Responsable: Gerencia de Operaciones. | Revisar `punto_pedido` y `plazo_entrega_dias` de los productos con quiebre. |
| I5 | **Alertas de reposición emitidas:** porcentaje de productos que llegan a su punto de pedido, sin orden pendiente, con alerta emitida (HU1). | 100 % | Casos de prueba (tarea P2) y, en operación, muestra mensual del registro de movimientos. Responsable: Analista y Compras. | Corregir la regla de evaluación (B1) y verificar que `punto_pedido` esté cargado en cada producto. |
| I6 | **Cumplimiento del plazo de reposición:** días entre `fecha_envio` de la orden de compra y el ingreso de la mercadería, comparados con `plazo_entrega_dias` del proveedor. | Plazo del proveedor más 2 días o menos, en el 90 % de las órdenes | Datos de Orden de compra y Movimiento de stock, vinculados por el número de remito. Mensual. Responsable: Compras. | Actualizar `plazo_entrega_dias` del proveedor y reclamar a los que incumplen (actividad B7). |

Los estándares de I5 e I6 son propuestos y se pueden ajustar con datos reales una vez que el módulo esté en producción.

## Registro de desvíos

Cada desvío detectado se registra como una tarea en el tablero de Jira, con el indicador afectado, la medición obtenida y la acción correctiva asignada.
