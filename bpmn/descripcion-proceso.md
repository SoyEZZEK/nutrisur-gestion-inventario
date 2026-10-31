# Proceso de reposición de stock (BPMN)

Módulo de Gestión de Inventario y Reposición - Nutrisur S.A.

![Diagrama BPMN del proceso de reposición de stock](reposicion-stock.png)

## Participantes

El modelo tiene dos pools:

- **Nutrisur S.A.**, con tres lanes: *Sistema de Inventario* (comportamiento automatizado del módulo), *Compras* y *Depósito y Stock*.
- **Proveedor**, actor externo cuya respuesta (despachar o informar falta de stock) determina cómo sigue el proceso.

Ventas y Facturación no tienen lane propio: su única intervención es el egreso de stock que el sistema genera al facturar, que actúa como disparador del proceso.

Los íconos del diagrama indican el tipo de tarea: engranaje (servicio, la resuelve el módulo), persona (usuario) y sobre (envío o recepción de un mensaje). Los códigos B, P y G identifican las tareas de Nutrisur, las del proveedor y las compuertas.

## Flujo estándar

El proceso comienza cuando se registra un movimiento de stock. El módulo evalúa si el producto requiere reposición (B1 y G1). Si es así, emite la alerta (B2) y sugiere una orden de compra, con la cantidad calculada a partir de los egresos registrados y dirigida al proveedor habitual (B3).

Compras revisa la orden sugerida (B4) y decide si la aprueba (G2). Una vez aprobada, el módulo la envía al correo del proveedor (B5). El proveedor verifica su disponibilidad (P1) y responde despachando la mercadería con su remito (P2) o informando que no tiene stock (P3). El módulo recibe esa respuesta (B6) y, si el proveedor confirma el despacho (G3), Depósito y Stock recibe la mercadería y la controla contra la orden y el remito (B8). Cuando la cantidad coincide (G4), registra el ingreso de stock (B10) y el módulo actualiza `stock_actual`. El proceso termina con el stock repuesto.

## Actividades

| ID | Responsable | Actividad | Tipo | Descripción | HU |
|---|---|---|---|---|---|
| B1 | Sistema de Inventario | Evaluar necesidad de reposición | Servicio | Compara `stock_actual` con `punto_pedido` del producto y verifica que no haya una orden de compra pendiente. | HU1 |
| B2 | Sistema de Inventario | Generar alerta de reposición | Servicio | Emite la alerta con el producto, el stock actual y el proveedor habitual (`id_proveedor`). | HU1 |
| B3 | Sistema de Inventario | Sugerir orden de compra | Servicio | Calcula la cantidad sugerida a partir de los movimientos de tipo EGRESO y genera la orden dirigida al proveedor habitual. Crea el dato Orden de compra. | HU2 |
| B4 | Compras | Revisar orden sugerida | Usuario | Revisa la orden y decide si la aprueba o la rechaza. | HU2 |
| B5 | Sistema de Inventario | Enviar orden al proveedor | Envío | Envía la orden aprobada al correo del proveedor (campo `email`). | HU2 |
| B6 | Sistema de Inventario | Recibir respuesta del proveedor | Recepción | Espera la respuesta del proveedor: confirmación de despacho con remito o aviso de falta de stock. | - |
| B7 | Compras | Gestionar faltante con el proveedor | Usuario | Contacta al proveedor por el faltante. Lo que se decida después (otro proveedor, nueva orden) queda fuera del modelo. | - |
| B8 | Depósito y Stock | Recibir y controlar mercadería | Usuario | Recibe la mercadería y compara las cantidades con la orden de compra y el remito. | HU3 |
| B9 | Depósito y Stock | Registrar diferencia | Usuario | Anota la diferencia entre lo pedido y lo recibido (faltante o sobrante) como observación del movimiento. | HU3 |
| B10 | Depósito y Stock | Registrar ingreso de stock | Usuario | Registra un movimiento INGRESO por la cantidad realmente recibida, con el número de remito como referencia. El módulo actualiza `stock_actual`; no se edita a mano. | HU3 |
| P1 | Proveedor | Verificar disponibilidad | Genérica | Controla si puede cubrir la orden de compra recibida. | - |
| P2 | Proveedor | Despachar y emitir remito | Genérica | Prepara el envío y emite el remito que acompaña la mercadería. | - |
| P3 | Proveedor | Informar falta de stock | Genérica | Avisa a Nutrisur que no puede cubrir la orden. | - |

## Compuertas y excepciones

| ID | Compuerta y condición | Si la respuesta es "Sí" | Si la respuesta es "No" |
|---|---|---|---|
| G1 | ¿Requiere reposición? Se cumple si `stock_actual` es menor o igual a `punto_pedido` y no hay una orden de compra pendiente. | Genera la alerta (B2) y sugiere la orden de compra (B3). | Fin: sin reposición necesaria. No se emite alerta. |
| G2 | ¿Orden aprobada? Decisión de Compras sobre la orden sugerida. | El módulo envía la orden al proveedor (B5). | Fin: orden rechazada. Si el producto sigue bajo el punto de pedido, se vuelve a evaluar con el próximo movimiento de stock. |
| G3 | ¿Proveedor confirma el despacho? Según la respuesta recibida en B6. | Depósito y Stock recibe y controla la mercadería (B8). | Compras gestiona el faltante (B7). Fin: reposición no concretada. |
| G4 | ¿Cantidad recibida coincide con la orden? Comparación hecha en B8. | Registra el ingreso de stock (B10). Fin: stock repuesto. | Registra la diferencia (B9) y después el ingreso por la cantidad recibida (B10). Fin: stock repuesto. |
| G5 | ¿Hay stock disponible? (pool Proveedor). | Despacha y emite el remito (P2). | Informa la falta de stock (P3). Ambos caminos terminan en el envío de la respuesta a Nutrisur. |

## Relación con las historias de usuario

- **HU1 - Alerta de reposición:** B1 y B2.
- **HU2 - Orden de compra sugerida:** B3, B4 y B5.
- **HU3 - Recepción de mercadería:** B8, B9 y B10.
