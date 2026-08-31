# Brief: busqueda-dashboard

## Problem
Los supervisores necesitan localizar rápidamente incidencias relevantes de su centro de trabajo (por estado, prioridad, fecha, tipo o responsable) y entender de un vistazo el estado operativo general (incidencias abiertas/cerradas, tiempos de resolución). Los administradores necesitan la misma visión pero agregada a nivel global. Sin esto, no hay forma de priorizar el trabajo ni de medir el desempeño del proceso de incidencias.

## Current State
No existe ninguna vista de consulta agregada ni de filtrado; la información vive dispersa en Excel, correos y llamadas, sin KPIs calculados.

## Desired Outcome
Un supervisor con incidencias en su centro de trabajo puede filtrar por estado, prioridad, fecha, tipo, responsable o centro de trabajo, viendo únicamente las incidencias que cumplen el filtro y que pertenecen a su propio centro. Un supervisor autenticado accede a un dashboard con KPIs (incidencias abiertas, cerradas, tiempo medio de resolución, incidencias por prioridad, incidencias por almacén) acotados a su centro de trabajo. Un administrador ve los mismos KPIs agregados a nivel global (todos los centros).

## Approach
Construir sobre la tabla `Incidencias` (definida en incidencias-core) una pantalla de listado/búsqueda con controles de filtro combinables (estado, prioridad, fecha, tipo, responsable, centro de trabajo) que aplican como condiciones adicionales sobre la vista ya acotada por rol/centro (heredada de autenticacion-roles). Construir una pantalla de dashboard con tarjetas/gráficos que calculan los KPIs requeridos mediante fórmulas Power Fx (agregaciones sobre la colección/datasource de Incidencias) o, si el volumen de datos lo requiere, vistas de Dataverse/consultas optimizadas para mantener el tiempo de carga por debajo de 3 segundos. El dashboard cambia su alcance (centro de trabajo vs. global) según el rol del usuario autenticado.

## Scope
- **In**: Pantalla de búsqueda/listado de incidencias con filtros combinables por estado, prioridad, fecha, tipo, responsable y centro de trabajo; aplicación automática del alcance por rol (supervisor: su centro; administrador: todos los centros; operario: solo lo suyo, heredado); pantalla de dashboard con los 5 KPIs especificados; lógica de agregación a nivel de centro de trabajo (supervisor) y global (administrador); optimización de las consultas para cumplir el objetivo de rendimiento (< 3s).
- **Out**: Definición del modelo de datos de Incidencias y su ciclo de vida (incidencias-core); comentarios y adjuntos (incidencias-colaboracion); envío de notificaciones (notificaciones); identificación de usuario, rol y centro de trabajo (autenticacion-roles, este spec solo lo consume); integración con Power BI (explícitamente fuera de alcance v1, los KPIs se muestran dentro de la propia app).

## Boundary Candidates
- Motor de filtrado/búsqueda multi-criterio sobre Incidencias.
- Cálculo y presentación de KPIs (dashboard) con alcance diferenciado por rol.

## Out of Boundary
- Alta, edición o cambio de estado de incidencias (incidencias-core, esta pantalla es de solo consulta/navegación hacia el detalle).
- Gestión de comentarios/adjuntos (incidencias-colaboracion).
- Cualquier notificación derivada de los KPIs o filtros (notificaciones).

## Upstream / Downstream
- **Upstream**: incidencias-core (fuente de datos de Incidencias y sus estados/fechas), autenticacion-roles (rol y centro de trabajo para acotar filtros y KPIs).
- **Downstream**: Ninguno identificado; es una capa de consulta terminal.

## Existing Spec Touchpoints
- **Extends**: incidencias-core (consulta y agrega sus datos, no los modifica).
- **Adjacent**: autenticacion-roles (alcance de datos por rol/centro).

## Constraints
- Tiempo de carga de consultas habituales < 3 segundos, con soporte para 500 usuarios activos.
- Un supervisor solo ve/gestiona incidencias de su propio centro de trabajo; un administrador ve datos globales.
- Los KPIs se muestran dentro de la propia app (sin Power BI en v1).
- Debe funcionar en navegador, Android, iPhone y tablet.
