# Requirements Document

## Introduction
Los supervisores y administradores de Logística Sur S.L. necesitan una capa de consulta rápida y fiable para localizar incidencias relevantes y entender el estado operativo de almacén sin depender de revisiones manuales en Excel, correos o llamadas. Los operarios también necesitan consultar únicamente sus incidencias visibles para hacer seguimiento de su trabajo sin exponer información ajena. Esta funcionalidad debe añadir una pantalla de búsqueda/listado multi-filtro y una pantalla de dashboard con KPIs dentro de la propia aplicación, reutilizando el alcance por rol y centro de trabajo ya definido en `autenticacion-roles` y los datos autoritativos de `Incidencias` definidos en `incidencias-core`.

> **Suposición operativa**: para evitar ambigüedad en el filtro de centro de trabajo, la experiencia de consulta mostrará el centro fijado y no ampliable para `Supervisor`, no expondrá un selector que amplíe alcance para `Operario` y permitirá selección global o por centro para `Administrador`.

## Boundary Context
- **In scope**: pantalla de búsqueda/listado con filtros combinables por estado, prioridad, fecha, tipo, responsable y centro de trabajo; aplicación automática del alcance por rol; navegación de solo consulta al detalle de la incidencia; dashboard interno con cinco KPIs; tiempos de respuesta observables de consulta y dashboard.
- **Out of scope**: creación, edición, asignación, resolución o cierre de incidencias; cambios sobre el esquema autoritativo de `Incidencias`; comentarios y adjuntos; notificaciones; integración con Power BI; autenticación y administración de usuarios o centros.
- **Adjacent expectations**: esta funcionalidad consume el perfil autoritativo con rol único `Operario` / `Supervisor` / `Administrador` y centro de trabajo vigente definido en `autenticacion-roles`; también consume la tabla `Incidencias`, sus estados y sus hitos temporales definidos en `incidencias-core` sin alterar su semántica.

## Requirements

### Requirement 1: Consulta acotada por rol
**Objective:** Como usuario operativo, quiero ver únicamente las incidencias que entran en mi alcance autorizado, para consultar el trabajo relevante sin exponer datos ajenos.

#### Acceptance Criteria
1.1 When el usuario abre la pantalla de búsqueda de incidencias, the Aplicación de Incidencias shall cargar un listado inicial limitado al alcance que corresponda a su rol vigente.

1.2 While el usuario tenga rol `Operario`, the Aplicación de Incidencias shall mostrar únicamente las incidencias creadas por ese usuario o asignadas a ese usuario.

1.3 While el usuario tenga rol `Supervisor`, the Aplicación de Incidencias shall mostrar únicamente incidencias pertenecientes a su centro de trabajo vigente.

1.4 While el usuario tenga rol `Administrador`, the Aplicación de Incidencias shall mostrar incidencias de todos los centros de trabajo.

1.5 If un usuario intenta abrir una incidencia fuera de su alcance autorizado mediante navegación directa, búsqueda o acceso guardado, the Aplicación de Incidencias shall denegar el acceso sin mostrar datos operativos de esa incidencia.

1.6 The Aplicación de Incidencias shall presentar la capa de búsqueda y dashboard en modo de solo consulta, sin ofrecer desde estas pantallas acciones de crear, editar, asignar, resolver o cerrar incidencias.

### Requirement 2: Búsqueda multi-filtro combinable
**Objective:** Como supervisor o administrador, quiero combinar varios filtros de consulta, para localizar rápidamente las incidencias que requieren atención.

#### Acceptance Criteria
2.1 The Aplicación de Incidencias shall ofrecer filtros visibles por estado, prioridad, fecha, tipo de incidencia, responsable y centro de trabajo dentro de la experiencia de búsqueda.

2.2 When el usuario aplica uno o varios filtros, the Aplicación de Incidencias shall mostrar únicamente las incidencias de su alcance que cumplan simultáneamente todas las condiciones seleccionadas.

2.3 When el usuario informa un rango de fechas válido, the Aplicación de Incidencias shall limitar los resultados a las incidencias comprendidas dentro de ese rango visible.

2.4 While el usuario tenga rol `Supervisor`, the Aplicación de Incidencias shall mantener el filtro de centro de trabajo fijado a su centro vigente sin permitir ampliarlo a otros centros.

2.5 Where existan incidencias visibles asociadas a tipos actualmente inactivos, the Aplicación de Incidencias shall permitir que esas incidencias sigan apareciendo en los resultados y puedan localizarse por su tipo histórico.

2.6 When el usuario restablece los filtros, the Aplicación de Incidencias shall volver a mostrar el conjunto de incidencias correspondiente a su alcance base.

2.7 If la combinación de filtros no devuelve coincidencias, the Aplicación de Incidencias shall mostrar un estado vacío explícito sin presentar resultados ambiguos.

### Requirement 3: Listado de resultados y navegación de consulta
**Objective:** Como usuario operativo, quiero revisar los resultados de búsqueda y abrir una incidencia concreta, para entender su estado sin perder el contexto de consulta.

#### Acceptance Criteria
3.1 The Aplicación de Incidencias shall mostrar en cada resultado al menos el identificador de incidencia, el título, el estado, la prioridad, el tipo, el responsable cuando exista, el centro de trabajo y la fecha de creación.

3.2 When el usuario abre una incidencia desde el listado o desde el dashboard, the Aplicación de Incidencias shall mostrar su detalle en modo consulta con los hitos temporales disponibles y sin habilitar acciones fuera del alcance de este spec.

3.3 When el usuario vuelve del detalle al listado, the Aplicación de Incidencias shall conservar los filtros y el contexto de búsqueda que estaban activos en la consulta anterior.

3.4 If la información de consulta no puede recuperarse temporalmente, the Aplicación de Incidencias shall informar del fallo y ofrecer una forma visible de reintentar la carga.

### Requirement 4: Dashboard operativo con KPIs
**Objective:** Como responsable operativo, quiero un dashboard con indicadores claros, para conocer el estado del proceso de incidencias de un vistazo.

#### Acceptance Criteria
4.1 When un usuario autorizado accede al dashboard, the Aplicación de Incidencias shall mostrar los cinco KPIs requeridos: incidencias abiertas, incidencias cerradas, tiempo medio de resolución, incidencias por prioridad e incidencias por almacén o centro de trabajo.

4.2 The Aplicación de Incidencias shall calcular el KPI de incidencias abiertas usando las incidencias cuyo estado actual no sea `Cerrada` dentro del alcance visible.

4.3 The Aplicación de Incidencias shall calcular el KPI de incidencias cerradas usando las incidencias cuyo estado actual sea `Cerrada` dentro del alcance visible.

4.4 The Aplicación de Incidencias shall calcular el tiempo medio de resolución usando únicamente las incidencias del alcance visible que ya dispongan de fecha de resolución registrada.

4.5 The Aplicación de Incidencias shall mostrar la distribución de incidencias por prioridad distinguiendo cada prioridad disponible en el alcance visible.

4.6 The Aplicación de Incidencias shall mostrar la distribución de incidencias por almacén o centro de trabajo de forma coherente con el alcance autorizado del usuario.

4.7 If no existen incidencias dentro del alcance visible para un KPI o distribución, the Aplicación de Incidencias shall mostrar el valor cero o un estado vacío claro en lugar de dejar tarjetas o gráficos ambiguos.

### Requirement 5: Alcance del dashboard según rol
**Objective:** Como supervisor o administrador, quiero que el dashboard refleje exactamente mi ámbito de responsabilidad, para no interpretar indicadores fuera de contexto.

#### Acceptance Criteria
5.1 While el usuario tenga rol `Supervisor`, the Aplicación de Incidencias shall calcular y mostrar todos los KPIs únicamente con incidencias de su centro de trabajo vigente.

5.2 While el usuario tenga rol `Administrador`, the Aplicación de Incidencias shall calcular y mostrar todos los KPIs sobre el conjunto global de incidencias y permitir acotar la lectura a un centro concreto sin perder la vista global al restablecerla.

5.3 If un usuario con rol `Operario` intenta acceder al dashboard, the Aplicación de Incidencias shall denegar el acceso o redirigirlo a la búsqueda dentro de su alcance sin mostrar KPIs globales o de centro.

5.4 When el alcance del usuario cambie entre sesiones por una actualización administrativa válida, the Aplicación de Incidencias shall reflejar el nuevo alcance del dashboard en el siguiente acceso del usuario.

### Requirement 6: Rendimiento y consistencia de la consulta
**Objective:** Como empleado de almacén, quiero que la consulta responda rápido y de forma consistente en cualquier dispositivo soportado, para poder usarla durante la operación diaria.

#### Acceptance Criteria
6.1 When el usuario ejecuta una búsqueda habitual o abre el dashboard en condiciones operativas normales, the Aplicación de Incidencias shall mostrar los resultados o KPIs en 3 segundos o menos.

6.2 While la funcionalidad se utilice dentro del volumen operativo previsto, the Aplicación de Incidencias shall presentar resultados completos del alcance autorizado sin truncarlos a un subconjunto parcial que altere su lectura.

6.3 When el usuario utiliza la funcionalidad desde navegador, Android, iPhone o tablet, the Aplicación de Incidencias shall mantener el mismo comportamiento observable de filtros, listado, navegación y KPIs según su rol.

6.4 While la integración con Power BI permanezca fuera de alcance en esta versión, the Aplicación de Incidencias shall mostrar los KPIs dentro de la propia app sin depender de una herramienta externa para consultarlos.
