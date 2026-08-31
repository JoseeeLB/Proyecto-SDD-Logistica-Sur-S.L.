# Requirements Document

## Introduction
Los operarios, supervisores y administradores de Logística Sur S.L. necesitan enterarse a tiempo de los eventos clave de una incidencia operativa sin tener que revisar continuamente la aplicación. Hoy los avisos de creación, asignación, cambio de estado y cierre se realizan de forma manual por teléfono o correo, con retrasos, omisiones y destinatarios inconsistentes. Esta funcionalidad debe automatizar el aviso de esos cuatro eventos para que cada interesado reciba la información adecuada dentro del alcance operativo ya definido por las especificaciones `incidencias-core` y `autenticacion-roles`.

> **Suposición operativa**: cuando una misma persona coincida a la vez como creador, responsable y/o supervisor destinatario del mismo evento, recibirá un único aviso por canal para evitar duplicados innecesarios.

## Boundary Context
- **In scope**: notificaciones automáticas para creación, asignación, cambio de estado y cierre de incidencias; resolución de destinatarios por relación con la incidencia; envío por correo corporativo y notificación push nativa dentro de la app; configuración central simple de destinatarios/canales/plantillas por evento; reintentos básicos y registro de fallos de entrega.
- **Out of scope**: lógica de negocio del ciclo de vida de la incidencia; notificaciones sobre comentarios o adjuntos; búsquedas, bandejas o dashboards de notificaciones; notificaciones push nativas fuera de la app de Power Apps; motor avanzado de reglas configurable por usuario final.
- **Adjacent expectations**: esta funcionalidad consume las incidencias y sus cambios confirmados desde `incidencias-core`, incluido el ciclo `Nueva → En revisión → Asignada → En progreso → Resuelta → Cerrada`; también consume el perfil autoritativo de usuario, rol y centro de trabajo definido por `autenticacion-roles` para resolver destinatarios válidos.

## Requirements

### Requirement 1: Cobertura automática de eventos de incidencia
**Objective:** Como empleado implicado en una incidencia, quiero recibir avisos automáticos cuando ocurra un evento relevante, para actuar sin depender de seguimiento manual.

#### Acceptance Criteria
1.1 When una incidencia se crea correctamente, the Servicio de Notificaciones shall generar automáticamente la notificación del evento de creación sin requerir intervención manual posterior.

1.2 When una incidencia recibe un responsable por primera vez o cambia de responsable, the Servicio de Notificaciones shall generar automáticamente la notificación del evento de asignación.

1.3 When una incidencia cambia a un nuevo estado válido que no activa los eventos de asignación ni cierre, the Servicio de Notificaciones shall generar automáticamente la notificación del evento de cambio de estado usando el estado confirmado tras la actualización.

1.4 When una incidencia entra en estado `Cerrada`, the Servicio de Notificaciones shall generar automáticamente la notificación del evento de cierre.

1.5 If una modificación de la incidencia no corresponde a creación, asignación, cambio de estado o cierre, the Servicio de Notificaciones shall no emitir una notificación dentro del alcance de esta versión.

### Requirement 2: Resolución de destinatarios por relación con la incidencia
**Objective:** Como responsable del proceso, quiero que cada aviso llegue a las personas correctas según el evento, para evitar comunicaciones incompletas o erróneas.

#### Acceptance Criteria
2.1 When se produce un evento cubierto, the Servicio de Notificaciones shall resolver destinatarios únicamente entre el creador de la incidencia, el responsable asignado y el supervisor del centro de trabajo asociado.

2.2 When un evento tenga configurada más de una categoría de destinatario, the Servicio de Notificaciones shall incluir a todos los destinatarios válidos resueltos para ese evento.

2.3 If una categoría de destinatario configurada no puede resolverse para una incidencia concreta, the Servicio de Notificaciones shall omitir solo ese destinatario y dejar constancia del problema de resolución.

2.4 While una misma persona cumpla varios criterios de destinatario para un mismo evento, the Servicio de Notificaciones shall consolidar el aviso para que esa persona no reciba duplicados del mismo evento en el mismo canal.

2.5 The Servicio de Notificaciones shall determinar el supervisor destinatario a partir del centro de trabajo asociado a la incidencia.

### Requirement 3: Entrega por canales permitidos y contenido accionable
**Objective:** Como destinatario de una incidencia, quiero recibir un aviso claro en el canal habilitado para ese evento, para entender rápidamente qué ha ocurrido y qué incidencia revisar.

#### Acceptance Criteria
3.1 When un evento tenga habilitado el canal de correo, the Servicio de Notificaciones shall enviar un correo corporativo a los destinatarios válidos de ese evento.

3.2 When un evento tenga habilitado el canal de notificación interna, the Servicio de Notificaciones shall enviar una notificación push nativa dentro de la app a los destinatarios válidos de ese evento.

3.3 The Servicio de Notificaciones shall incluir en cada aviso, como mínimo, el identificador de la incidencia, el tipo de evento y el estado actual de la incidencia tras el evento.

3.4 When el evento sea de asignación o cierre, the Servicio de Notificaciones shall incluir también el responsable actual de la incidencia cuando exista.

3.5 While la versión 1 de esta funcionalidad esté activa, the Servicio de Notificaciones shall limitarse a correo corporativo y notificación push dentro de la app sin emitir avisos por canales adicionales ni push nativa fuera de la app.

### Requirement 4: Configuración central simple por evento
**Objective:** Como administrador funcional, quiero mantener una configuración central y simple de avisos por evento, para ajustar destinatarios, canales y mensajes sin redefinir el flujo de incidencias.

#### Acceptance Criteria
4.1 The Servicio de Notificaciones shall permitir una configuración independiente para cada uno de los cuatro eventos cubiertos.

4.2 When se actualiza la configuración de un evento, the Servicio de Notificaciones shall aplicar la nueva combinación de destinatarios y canales a las notificaciones futuras de ese evento.

4.3 When un evento tenga deshabilitado uno de sus canales, the Servicio de Notificaciones shall dejar de enviar notificaciones por ese canal para eventos futuros sin afectar a los demás canales habilitados.

4.4 If se intenta configurar un destinatario fuera de las categorías creador, responsable asignado o supervisor del centro, the Servicio de Notificaciones shall impedir esa configuración dentro del alcance de esta versión.

4.5 While la configuración de un evento permanezca vigente, the Servicio de Notificaciones shall usar mensajes consistentes para todas las incidencias que activen ese mismo evento.

### Requirement 5: Manejo básico de errores y continuidad de entrega
**Objective:** Como responsable operativo, quiero que los fallos de envío se manejen de forma controlada, para no perder avisos silenciosamente ni bloquear notificaciones válidas.

#### Acceptance Criteria
5.1 If un envío falla en su primer intento, the Servicio de Notificaciones shall reintentar la entrega según una política básica antes de marcarla como fallida.

5.2 If una entrega sigue fallando tras los reintentos definidos, the Servicio de Notificaciones shall registrar el evento, el canal y el destinatario afectados como fallo de entrega.

5.3 When un mismo evento de incidencia deba avisar a varios destinatarios o por varios canales, the Servicio de Notificaciones shall continuar con las demás entregas aunque una de ellas falle.

5.4 If una entrega termina con fallo, the Servicio de Notificaciones shall no modificar los datos de negocio de la incidencia como efecto lateral del error.

5.5 The Servicio de Notificaciones shall dejar trazabilidad suficiente para identificar qué avisos se intentaron enviar y cuál fue el resultado final de cada intento.

### Requirement 6: Coherencia con el núcleo de incidencias y límites de alcance
**Objective:** Como propietario funcional del sistema, quiero que las notificaciones reaccionen al núcleo de incidencias sin alterar su lógica ni invadir otros ámbitos, para mantener un comportamiento consistente entre especificaciones.

#### Acceptance Criteria
6.1 When un evento cubierto se confirma sobre una incidencia, the Servicio de Notificaciones shall basar el aviso en los datos ya confirmados de la incidencia, incluido su estado actual y su responsable vigente cuando exista.

6.2 While una incidencia recorra los estados `Nueva`, `En revisión`, `Asignada`, `En progreso`, `Resuelta` y `Cerrada`, the Servicio de Notificaciones shall respetar esa secuencia como fuente autoritativa de los eventos de estado y cierre.

6.3 If se producen cambios en comentarios, adjuntos u otras capacidades ajenas a estos cuatro eventos, the Servicio de Notificaciones shall no emitir avisos dentro del alcance de esta versión.

6.4 The Servicio de Notificaciones shall comportarse de forma consistente para incidencias de cualquier centro de trabajo y para los roles soportados por el sistema.

