# Requirements Document

## Introduction
Los operarios, supervisores y administradores de Logística Sur S.L. necesitan una gestión única y trazable de incidencias operativas de almacén dentro de la aplicación corporativa. Actualmente las incidencias se registran y siguen mediante llamadas, correos y hojas Excel, lo que provoca pérdida de contexto, retrasos y ausencia de un ciclo de vida homogéneo. Esta funcionalidad debe permitir registrar incidencias, clasificarlas con un catálogo administrable, asignarlas a un responsable, hacerlas avanzar por estados secuenciales obligatorios y conservar los hitos básicos del proceso usando el rol y el centro de trabajo ya resueltos por `autenticacion-roles`.

> **Suposición operativa**: para mantener el límite de datos indicado para `Incidencias`, esta especificación trata la fecha obligatoria visible para el usuario como la fecha de registro de la incidencia y permite que la ubicación obligatoria se capture mediante un control específico de la app aunque su persistencia quede normalizada dentro del contenido operativo de la incidencia hasta que un cambio de alcance autorice un atributo dedicado.

## Boundary Context
- **In scope**: alta de incidencias, visibilidad operativa de incidencias según rol y relación con la incidencia, catálogo configurable de tipos de incidencia, asignación de responsable, ciclo de vida secuencial, registro de fechas de creación/asignación/resolución y trazabilidad básica de eventos clave.
- **Out of scope**: autenticación y resolución del perfil de usuario, comentarios y adjuntos como capacidades completas, búsqueda y filtrado avanzado, dashboard agregado, envío de notificaciones y administración general de usuarios o centros.
- **Adjacent expectations**: esta funcionalidad consume el perfil autoritativo (`jlb_perfilusuario` con `jlb_rolnegocio` y `jlb_centrotrabajo`) definido en `autenticacion-roles`; los specs `incidencias-colaboracion`, `busqueda-dashboard` y `notificaciones` dependerán de los identificadores, estados y marcas temporales definidos aquí sin ampliar esta frontera.

## Requirements

### Requirement 1: Registro inicial de incidencias
**Objective:** Como operario, quiero registrar una incidencia con la información mínima obligatoria, para dejar constancia inmediata de un problema operativo sin depender de canales informales.

#### Acceptance Criteria
1.1 When el usuario autorizado inicia el alta de una incidencia, the Aplicación de Incidencias shall solicitar como obligatorios el título, la descripción, el tipo de incidencia, la prioridad y la ubicación, y mostrar la fecha de registro asociada al alta antes de permitir el registro.

1.2 When el usuario confirma un alta válida, the Aplicación de Incidencias shall crear la incidencia con un identificador único, estado inicial `Nueva` y fecha de creación registrada automáticamente.

1.3 When el usuario registra una incidencia válida, the Aplicación de Incidencias shall asociarla al usuario creador y a su centro de trabajo vigente sin pedir una reasignación manual de ese contexto.

1.4 If falta un dato obligatorio o el contenido no cumple la validación visible del formulario, the Aplicación de Incidencias shall impedir el registro y mostrar qué dato debe corregirse, incluida la ubicación cuando no haya sido informada.

1.5 While el formulario de alta esté abierto, the Aplicación de Incidencias shall mostrar únicamente tipos de incidencia activos y disponibles para selección.

### Requirement 2: Consulta operativa y alcance de acceso a incidencias
**Objective:** Como usuario operativo, quiero acceder solo a las incidencias que entran en mi alcance, para trabajar sin exponer información ajena a mi responsabilidad.

#### Acceptance Criteria
2.1 When un operario consulta una incidencia, the Aplicación de Incidencias shall permitir el acceso solo si esa incidencia fue creada por ese operario o si le está asignada como responsable.

2.2 While el usuario tenga rol `Supervisor`, the Aplicación de Incidencias shall permitirle consultar y gestionar incidencias pertenecientes a su centro de trabajo vigente.

2.3 While el usuario tenga rol `Administrador`, the Aplicación de Incidencias shall permitirle consultar y gestionar incidencias de todos los centros de trabajo.

2.4 If un usuario intenta abrir una incidencia fuera de su alcance autorizado, the Aplicación de Incidencias shall denegar el acceso sin mostrar los datos operativos de esa incidencia.

2.5 The Aplicación de Incidencias shall mostrar en la vista de detalle el identificador de la incidencia, su estado actual, su prioridad, su tipo, el responsable asignado cuando exista y el centro de trabajo asociado.

### Requirement 3: Catálogo configurable de tipos de incidencia
**Objective:** Como administrador, quiero mantener el catálogo de tipos de incidencia, para adaptar la clasificación operativa sin modificar el flujo base de incidencias.

#### Acceptance Criteria
3.1 When un administrador accede a la gestión del catálogo, the Aplicación de Incidencias shall mostrar el listado de tipos de incidencia con su estado de disponibilidad.

3.2 The Aplicación de Incidencias shall disponer inicialmente de los tipos `Seguridad`, `Equipamiento`, `Infraestructura`, `Calidad`, `Transporte` y `Otros`.

3.3 When un administrador crea un nuevo tipo de incidencia válido, the Aplicación de Incidencias shall incorporarlo al catálogo para su uso posterior en nuevas incidencias.

3.4 When un administrador modifica la descripción visible de un tipo existente, the Aplicación de Incidencias shall aplicar el cambio sin alterar el historial de incidencias ya registradas con ese tipo.

3.5 When un administrador desactiva un tipo de incidencia, the Aplicación de Incidencias shall impedir su selección en nuevas incidencias y conservarlo como referencia en incidencias ya existentes.

3.6 If un usuario que no sea `Administrador` intenta gestionar el catálogo, the Aplicación de Incidencias shall denegar la operación e informar que no dispone de permisos para esa capacidad.

### Requirement 4: Asignación de responsable
**Objective:** Como supervisor, quiero asignar una incidencia a una persona responsable, para dejar claro quién debe atenderla y desde cuándo.

#### Acceptance Criteria
4.1 When un usuario con rol `Supervisor` o `Administrador` asigna una incidencia a un responsable válido, the Aplicación de Incidencias shall registrar el responsable asignado y la fecha de asignación.

4.2 When una incidencia pasa a estado `Asignada`, the Aplicación de Incidencias shall exigir que exista un responsable informado antes de confirmar el cambio.

4.3 If un usuario sin rol `Supervisor` o `Administrador` intenta asignar o reasignar un responsable, the Aplicación de Incidencias shall denegar la operación sin modificar la incidencia.

4.4 If el responsable propuesto no pertenece al alcance operativo permitido para la incidencia, the Aplicación de Incidencias shall rechazar la asignación e indicar que debe seleccionarse un responsable válido.

4.5 The Aplicación de Incidencias shall mostrar en la incidencia quién es el responsable actual y cuándo fue asignada por última vez.

### Requirement 5: Ciclo de vida secuencial de la incidencia
**Objective:** Como supervisor, quiero hacer avanzar una incidencia por estados obligatorios y ordenados, para asegurar un seguimiento homogéneo hasta su cierre.

#### Acceptance Criteria
5.1 The Aplicación de Incidencias shall utilizar la secuencia de estados `Nueva`, `En revisión`, `Asignada`, `En progreso`, `Resuelta` y `Cerrada` como único ciclo de vida válido para una incidencia.

5.2 When un usuario autorizado cambia el estado de una incidencia, the Aplicación de Incidencias shall permitir únicamente el siguiente estado inmediato de la secuencia sin saltos intermedios.

5.3 If se intenta mover una incidencia a un estado no consecutivo o ya superado, the Aplicación de Incidencias shall rechazar el cambio y mantener el estado anterior.

5.4 When una incidencia entra en estado `Resuelta`, the Aplicación de Incidencias shall registrar la fecha de resolución.

5.5 When una incidencia entra en estado `Cerrada`, the Aplicación de Incidencias shall conservar visibles el responsable, la fecha de asignación y la fecha de resolución ya registradas.

5.6 If un usuario sin rol `Supervisor` o `Administrador` intenta cambiar el estado o cerrar una incidencia, the Aplicación de Incidencias shall denegar la operación sin modificar la incidencia.

### Requirement 6: Trazabilidad básica y preparación para capacidades dependientes
**Objective:** Como responsable del proceso, quiero que los hitos clave de una incidencia queden preservados de forma consistente, para soportar seguimiento operativo y futuras capacidades dependientes sin redefinir la incidencia.

#### Acceptance Criteria
6.1 The Aplicación de Incidencias shall conservar para cada incidencia su identificador, estado actual, fecha de creación, fecha de asignación cuando exista y fecha de resolución cuando exista.

6.2 When se crea, asigna, reasigna, resuelve o cierra una incidencia, the Aplicación de Incidencias shall dejar esos cambios reflejados de forma consistente en el registro de la incidencia para su consulta posterior.

6.3 While una incidencia permanezca abierta, the Aplicación de Incidencias shall mostrar su estado y sus hitos temporales disponibles de manera consistente en la experiencia operativa.

6.4 Where otras capacidades dependan de una incidencia existente, the Aplicación de Incidencias shall mantener estable el identificador de la incidencia y los hitos de estado ya registrados como base de integración.

### Requirement 7: Continuidad operativa y tiempos de respuesta
**Objective:** Como empleado de almacén, quiero una experiencia consistente en todos mis dispositivos habituales, para registrar y seguir incidencias sin fricciones operativas.

#### Acceptance Criteria
7.1 When un usuario autorizado utiliza la funcionalidad desde navegador, Android, iPhone o tablet, the Aplicación de Incidencias shall ofrecer el mismo flujo base de alta, consulta, asignación y avance de estados según su rol.

7.2 When un usuario abre una incidencia o confirma una transición válida en condiciones habituales, the Aplicación de Incidencias shall reflejar el resultado operativo en 3 segundos o menos.

7.3 If la aplicación no puede completar una operación de registro, asignación o cambio de estado, the Aplicación de Incidencias shall informar del fallo sin dejar una confirmación ambigua al usuario.

7.4 While la incidencia no admita comentarios, adjuntos, búsqueda avanzada o notificaciones dentro de esta funcionalidad, the Aplicación de Incidencias shall no presentar esas capacidades como si estuvieran disponibles en este alcance.
