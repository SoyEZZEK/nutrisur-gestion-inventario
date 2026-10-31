# Diccionario de datos

Módulo de Gestión de Inventario y Reposición - Nutrisur S.A.

Versión preliminar con las entidades que usa el proceso de reposición de stock: Producto, Movimiento de stock, Proveedor y Orden de compra. Las tres primeras se definieron en la Entrega 3; la Orden de compra se incorpora para sostener el proceso BPMN. El diseño definitivo de la base de datos corresponde a la tarea D1 del cronograma (Entrega 2).

## Convenciones

- Los nombres de los campos van en minúsculas con guion bajo.
- PK indica clave primaria y FK clave foránea.
- Los tipos de dato son genéricos y se precisan en el diseño definitivo de la base de datos.
- Todos los campos son obligatorios salvo que se indique lo contrario.

## Entidad Producto

| Campo | Tipo | Descripción funcional | Restricciones |
|---|---|---|---|
| id_producto | Integer | Identificador único del producto dentro del sistema. | PK |
| codigo_producto | String(20) | Código comercial con el que se identifica el artículo en depósito y facturación. | Único |
| descripcion | String(100) | Nombre y presentación del artículo (por ejemplo, fideos 500 g). | - |
| categoria | String(20) | Rubro del artículo; sirve para agrupar los reportes de rotación. | Almacén, Kiosco o Bazar |
| costo_unitario | Decimal(12,2) | Último costo de compra por unidad, en pesos. Permite valorizar el stock y se actualiza cuando cambian los precios. | Mayor a 0 |
| stock_actual | Integer | Unidades disponibles en depósito. | Mayor o igual a 0. Lo actualiza el sistema con cada movimiento (ingresos menos egresos, más o menos ajustes); no se edita a mano |
| punto_pedido | Integer | Nivel de stock en el que se emite la alerta de reposición. | Mayor o igual a 0. Hay alerta cuando stock_actual es menor o igual a punto_pedido |
| id_proveedor | Integer | Proveedor habitual del producto, al que se dirige la orden de compra sugerida. | FK a Proveedor |
| activo | Boolean | Indica si el producto se sigue comercializando. Permite darlo de baja sin perder el histórico. | Valor por defecto: verdadero |

**Usuario responsable:** Compras da de alta los productos y mantiene el costo y el punto de pedido. El campo stock_actual solo lo modifica el sistema.

## Entidad Movimiento de stock

| Campo | Tipo | Descripción funcional | Restricciones |
|---|---|---|---|
| id_movimiento | Integer | Identificador único del movimiento. | PK |
| id_producto | Integer | Producto afectado por el movimiento. | FK a Producto |
| tipo_movimiento | String(15) | Clase de movimiento; determina si suma o resta stock. | INGRESO, EGRESO, AJUSTE_POSITIVO o AJUSTE_NEGATIVO |
| cantidad | Integer | Unidades que suma o resta el movimiento. Siempre es positiva; el sentido lo da tipo_movimiento. | Mayor a 0 |
| fecha_hora | DateTime | Momento del registro. Es la base del histórico de ventas para sugerir órdenes de compra y de los reportes de rotación. | Lo asigna el sistema |
| referencia | String(30) | Número del comprobante que originó el movimiento (factura de venta o remito de compra). | Obligatorio en INGRESO y EGRESO; opcional en ajustes |
| observacion | String(100) | Motivo del movimiento, sobre todo en ajustes (rotura, vencimiento, diferencia de inventario). | Obligatorio en ajustes; opcional en el resto |
| usuario_registro | String(30) | Usuario que cargó el movimiento; permite rastrear quién lo hizo. | Valor 'sistema' en los egresos automáticos por facturación |

**Usuario responsable:** Depósito y Stock registra ingresos y ajustes. Los egresos por venta los genera el sistema al emitirse la factura. Un movimiento ya registrado no se modifica ni se borra; si hay un error se corrige con un ajuste, para no perder la trazabilidad.

## Entidad Proveedor

| Campo | Tipo | Descripción funcional | Restricciones |
|---|---|---|---|
| id_proveedor | Integer | Identificador único del proveedor. | PK |
| razon_social | String(100) | Nombre legal del proveedor; figura en la orden de compra. | - |
| cuit | String(13) | CUIT con guiones (XX-XXXXXXXX-X), para identificarlo fiscalmente. | Único |
| telefono | String(20) | Teléfono de contacto comercial. | Opcional |
| email | String(80) | Correo al que se envía la orden de compra sugerida. | Formato de correo válido. Opcional |
| plazo_entrega_dias | Integer | Días promedio entre el pedido y la recepción de la mercadería. Se usa para ajustar el punto de pedido. | Mayor o igual a 0 |
| activo | Boolean | Indica si se le sigue comprando al proveedor. | Valor por defecto: verdadero |

**Usuario responsable:** Compras da de alta y mantiene los datos de los proveedores.

## Entidad Orden de compra

| Campo | Tipo | Descripción funcional | Restricciones |
|---|---|---|---|
| id_orden_compra | Integer | Identificador único de la orden de compra. | PK |
| id_producto | Integer | Producto que se repone con la orden. | FK a Producto |
| id_proveedor | Integer | Proveedor al que se dirige la orden; por defecto, el proveedor habitual del producto. | FK a Proveedor |
| cantidad_sugerida | Integer | Unidades que sugiere el sistema, calculadas a partir de los movimientos de tipo EGRESO del producto. | Mayor a 0 |
| fecha_generacion | DateTime | Momento en que el sistema genera la orden sugerida. | Lo asigna el sistema |
| estado | String(15) | Situación de la orden dentro del proceso de reposición. Si está SUGERIDA o ENVIADA se considera pendiente y no se repite la alerta del producto. | SUGERIDA, ENVIADA, RECHAZADA, RECIBIDA o NO_CONCRETADA. Valor por defecto: SUGERIDA |
| fecha_envio | DateTime | Momento en que la orden aprobada se envía al correo del proveedor. Se usa para medir el plazo de entrega real. | Opcional. Lo asigna el sistema al enviar |
| cantidad_recibida | Integer | Unidades efectivamente recibidas. Permite detectar diferencias contra cantidad_sugerida. | Opcional. Mayor o igual a 0 |
| numero_remito | String(30) | Número del remito con el que el proveedor entregó la mercadería. | Opcional. Se completa al recibir |

**Usuario responsable:** el sistema genera la orden y la envía; Compras la aprueba o la rechaza; Depósito y Stock completa cantidad_recibida y numero_remito al recibir la mercadería. Una orden con estado RECHAZADA, RECIBIDA o NO_CONCRETADA no se modifica.

**Estados de la orden:** SUGERIDA (generada por el sistema, a la espera de la revisión de Compras), ENVIADA (aprobada y enviada al proveedor), RECHAZADA (Compras no la aprobó), RECIBIDA (la mercadería ingresó al stock) y NO_CONCRETADA (el proveedor informó que no tiene stock).

## Relaciones entre las entidades

- Cada producto tiene un proveedor habitual (`id_proveedor`).
- Un proveedor puede abastecer a varios productos.
- Cada movimiento de stock afecta a un producto (`id_producto`).
- Un producto puede tener muchos movimientos.
- Cada orden de compra repone un producto (`id_producto`) y se dirige a un proveedor (`id_proveedor`).
- Un producto y un proveedor pueden tener muchas órdenes a lo largo del tiempo, pero un producto tiene como máximo una orden pendiente (SUGERIDA o ENVIADA) a la vez.

## Fuera de alcance

Clientes, facturas y el resto de las entidades del negocio no forman parte del módulo.
