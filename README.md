Sistema de Inventario Predictivo y Gestión Automática de Compras - McDonald's

Trabajo Práctico Integrador - Entrega 4: Procesos, Trabajo Colaborativo y Control Integrador
Asignatura: Organización Empresarial · Tecnicatura Universitaria en Programación a Distancia (TUP AD) · UTN

Descripción del sistema

Módulo de software para McDonald's que calcula el stock de cada insumo por lote, proyecta la demanda de las próximas 2 horas y, cuando el stock cae por debajo del umbral crítico, genera y envía automáticamente una orden de compra al proveedor. Además registra cada lote recibido con su fecha de vencimiento para trazar y reducir la merma. Reemplaza el esquema actual de pedidos manuales cada 24 o 48 horas basados en promedios semanales fijos (Entrega 1).

Integrantes
Gimenez Walter (trabajo individual)
Enlaces
Repositorio: https://github.com/waltergim/tpi-mcdonalds-inventario-predictivo
Tablero ágil (GitHub Projects): https://github.com/users/waltergim/projects/2
Estructura del repositorio
README.md                      Portada del proyecto
docs/
  bpmn/                        Diagrama BPMN (.bpmn editable, .png y .svg)
Modelado del proceso (BPMN)

Proceso modelado: reposición automática de insumos. Archivos en docs/bpmn/ (el .bpmn se abre en https://demo.bpmn.io).

BPMN reposición automática

Pools: McDonald's - Local y Proveedor (actor externo).
Lanes del pool McDonald's: Sistema (automatizado), Responsable de Compras y Encargado de Depósito.
Flujo estándar: cada hora, el sistema calcula el stock y la demanda proyectada, genera la orden de compra, la envía, el proveedor despacha, el depósito controla la mercadería y el sistema registra el lote.
Compuerta	Lane	Excepciones que contempla
¿Stock < umbral crítico?	Sistema	No: el proceso termina sin acción (excepción evitada, no hay quiebre). Sí: se genera la orden de compra.
¿Proveedor con integración digital?	Sistema	Sí: envío automático (orden Enviada). No: el Responsable de Compras envía la orden por e-mail y registra el envío (camino manual de contingencia).
¿Mercadería conforme?	Encargado de Depósito	Sí: se registra el lote con su vencimiento, se actualiza el stock y la orden pasa a Recibida. No: se rechaza la entrega, la orden pasa a Cancelada y el Responsable de Compras reclama al proveedor y emite un nuevo pedido.
Backlog ágil: Historias de Usuario
ID	Historia	Estado	Entidades	Paso del BPMN
HU-1	Como encargado de depósito, quiero que el sistema detecte cuando el stock de un insumo cae por debajo de su umbral crítico, para evitar quiebres de stock.	Listo	Insumo, LoteInventario	Calcular stock por lote y demanda proyectada; compuerta '¿Stock < umbral crítico?'
HU-2	Como responsable de compras, quiero que el sistema emita automáticamente una orden de compra al proveedor, para reducir los pedidos manuales y los tiempos de reposición.	En Progreso	OrdenCompra, Proveedor	Generar orden de compra; compuerta '¿Proveedor con integración digital?'; envío automático o por e-mail
HU-3	Como gerente del local, quiero ver reportes de merma por lote según su fecha de vencimiento, para reducir el desperdicio y decidir sobre el menú.	Por Hacer	LoteInventario, Insumo	Control de mercadería; compuerta '¿Mercadería conforme?'; Registrar lote con vencimiento

Estados del tablero: Por Hacer · En Progreso · Listo. Los estados reflejan el avance de la documentación y el diseño del módulo; no se desarrolló código en esta asignatura.

HU-1 - criterios de aceptación

Se evalúa el stock de cada insumo activo a partir de sus lotes (LoteInventario.cantidad_actual).
Si el stock proyectado a 2 horas es menor a Insumo.umbral_critico, se dispara la alerta.
Si no está por debajo del umbral, no se genera ninguna acción.

HU-2 - criterios de aceptación

La orden se crea con estado Pendiente, fecha_emision y cantidad_solicitada según la demanda proyectada.
Si el proveedor tiene integracion_digital, el envío es automático y la orden pasa a Enviada.
Si no tiene integración digital, el responsable de compras la envía por e-mail y registra el envío.

HU-3 - criterios de aceptación

Cada lote recibido se registra con fecha_recepcion, cantidad_actual y fecha_vencimiento.
El reporte muestra por lote las unidades vencidas o próximas a vencer.
Si la entrega no es conforme, la orden pasa a Cancelada y no se registra el lote.
Mecanismos de control: indicadores y estándares de desempeño

Los estándares son metas propuestas por el autor para este caso de estudio; no son cifras oficiales de McDonald's.

Indicador	Fórmula	Estándar	Frecuencia / fuente	Acción correctiva
IND-1 Quiebres de stock	(Insumos críticos con stock 0 / insumos críticos activos) x 100	0 quiebres en insumos críticos (carne, pan, papas)	Diaria. Fuente: Insumo y LoteInventario	Revisar y ajustar Insumo.umbral_critico
IND-2 Merma por vencimiento	(Unidades vencidas en lotes / unidades recibidas) x 100	Menor o igual a 3% mensual	Mensual. Fuente: LoteInventario (fecha_vencimiento, cantidad_actual)	Reducir cantidad_solicitada o lanzar promoción del producto
IND-3 Tiempo de reacción	Minutos entre el cruce del umbral y OrdenCompra.fecha_emision	Menor o igual a 5 minutos	Por evento. Fuente: registros del sistema y OrdenCompra	Revisar la frecuencia de la tarea y las fallas del sistema
IND-4 Error de pronóstico	Error porcentual medio entre demanda proyectada (2 h) y demanda real	Menor o igual a 15%	Semanal. Fuente: ventas reales vs. proyección	Reajustar parámetros del cálculo de demanda
IND-5 Cumplimiento del proveedor	(Órdenes Recibidas dentro de tiempo_entrega_hs / órdenes Enviadas) x 100	Mayor o igual a 95%	Mensual. Fuente: OrdenCompra y Proveedor	Reclamo formal, proveedor alternativo, priorizar proveedores con integracion_digital
IND-6 Disponibilidad del sistema	(Tiempo en servicio / tiempo total) x 100	Mayor o igual a 99,5% mensual	Mensual. Responsable: Infraestructura/DevOps	Análisis de incidentes y plan de contingencia (camino manual)
IND-7 Calidad del software	Cobertura de pruebas sobre funciones críticas (umbral y generación de OC); defectos críticos abiertos	Cobertura mayor o igual a 70%; 0 defectos críticos abiertos al liberar	Por versión. Responsable: Líder de QA	Bloquear la liberación hasta corregir
IND-8 Gestión del proyecto	Historias en estado Listo / historias totales; cambios integrados mediante Pull Request	3 de 3 historias en Listo al cierre; todo cambio posterior al commit inicial se integra por Pull Request	Semanal. Fuente: tablero y repositorio	Replanificar el backlog en la revisión semanal
Declaración de Uso de IA

Se usó un asistente de IA (Claude, de Anthropic) para: (1) ordenar el plan de trabajo de la entrega, (2) proponer el borrador del diagrama BPMN y generar el archivo .bpmn, (3) redactar las historias de usuario con sus criterios de aceptación, (4) proponer los indicadores de control y (5) redactar el borrador del README y (6) guiar paso a paso la creación del repositorio, las ramas, el Pull Request, los issues y el tablero. No se usó para inventar datos oficiales de McDonald's.

Prompts utilizados y justificación

Prompt	Justificación
Pegué el enunciado completo de la Entrega 4 y escribí: "tengo que hacer eso para el tp 4".	Necesitaba descomponer cuatro consignas (BPMN, GitHub, backlog, control) y el PDF consolidado en un orden de trabajo.
Subí los PDF de las Entregas 1 y 3 y pedí: "arma todo para que concuerde y podamos entregar la 4".	Pedí que el BPMN, las historias de usuario y los indicadores se alineen con el sistema abierto (Entrega 1) y con el diccionario de datos y el organigrama (Entrega 3).
Pedí instrucciones paso a paso para crear el repositorio, las ramas, el Pull Request, los issues y el tablero, y mostré capturas de pantalla para verificar cada paso.	Era la primera vez que usaba GitHub Projects y necesitaba una guía; las capturas me sirvieron para confirmar que cada paso estuviera bien hecho.

Correcciones humanas sobre los resultados de la IA

Se verificó que los estados del BPMN (Pendiente, Enviada, Recibida, Cancelada) coinciden con el campo OrdenCompra.estado del diccionario de datos de la Entrega 3.
Se revisó que las compuertas, los pools y las lanes del BPMN reflejen el flujo descripto en la Entrega 1.
Los valores de los estándares de desempeño (por ejemplo 3%, 5 minutos, 95%) son metas propuestas por el autor; no son datos oficiales de la empresa.
Se corrigió el nombre de la empresa en la Entrega 1 ("Macdonals" por "McDonald's") y se verificó la misión y la visión citadas contra una fuente oficial.
Se corrigió el nombre del docente titular en la portada de la Entrega 3.

Responsabilidad intelectual: el autor revisó el contenido y asume la responsabilidad por su veracidad y coherencia; los errores del modelo se consideran errores propios.
