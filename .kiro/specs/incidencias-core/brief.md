# Brief: incidencias-core

## Problem
Los supervisores y operarios de almacén necesitan un lugar único y confiable donde registrar incidencias operativas (seguridad, equipamiento, infraestructura, calidad, transporte, etc.), clasificarlas, hacerles seguimiento a través de un ciclo de vida claro y asignarlas a responsables, en lugar de depender de llamadas, correos y hojas Excel dispersas que generan retrasos y errores de información.

## Current State
No existe modelo de datos ni proceso digital para incidencias; todo se gestiona manualmente. No hay catálogo de tipos de incidencia, ni estados normalizados, ni trazabilidad de fechas de creación/asignación/resolución.

## Desired Outcome
Un operario autenticado puede crear una incidencia con los campos obligatorios (Título, Descripción, Tipo, Prioridad, Fecha, Ubicación) y campos opcionales; la incidencia nace en estado "Nueva" con fecha de creación registrada. Un administrador puede gestionar el catálogo de tipos de incidencia (ver, crear, editar, desactivar), con los tipos por defecto ya cargados (Seguridad, Equipamiento, Infraestructura, Calidad, Transporte, Otros). Un supervisor puede asignar una incidencia (pasa a "Asignada", registra responsable y fecha) y hacer avanzar su estado de forma secuencial (Nueva → En revisión → Asignada → En progreso → Resuelta → Cerrada), registrando las fechas de asignación y resolución correspondientes. Solo un supervisor o administrador puede cerrar una incidencia.

## Approach
Modelar en Dataverse la tabla `Incidencias` (según el modelo de datos ya definido: IdIncidencia, Titulo, Descripcion, Estado, Prioridad, TipoIncidencia, FechaCreacion, FechaAsignacion, FechaResolucion, Responsable, CentroTrabajo) y una tabla de catálogo `TiposIncidencia` editable por el administrador (relación en lugar de opción fija, para cumplir RF-03). Implementar en la canvas app las pantallas de: nueva incidencia (formulario con validación de obligatorios), administración de tipos (solo Administrador), y pantalla de detalle/edición de incidencia con controles de transición de estado que solo se habilitan según el rol (Supervisor/Administrador) y el estado actual (transición secuencial, sin saltos). Las reglas de visibilidad (operario ve solo lo suyo, supervisor ve solo su centro) se apoyan en el perfil de usuario definido en `autenticacion-roles`.

## Scope
- **In**: Tabla Dataverse `Incidencias` y tabla de catálogo `TiposIncidencia`; formulario de alta con validación de campos obligatorios (Título, Descripción, Tipo, Prioridad, Fecha, Ubicación) y opcionales (fotos, adjuntos iniciales, comentarios iniciales — delegando el almacenamiento físico de adjuntos/comentarios a incidencias-colaboracion si ya existe, o dejando el gancho de integración); pantalla de administración de tipos de incidencia (alta/edición/desactivación) restringida a Administrador; máquina de estados (Nueva, En revisión, Asignada, En progreso, Resuelta, Cerrada) con transición secuencial obligatoria; asignación de responsable por parte de un supervisor; registro de fechas de creación/asignación/resolución; restricción de cierre a Supervisor/Administrador; auditoría básica de creación/cambios/asignaciones (RNF-04) a nivel de esta tabla.
- **Out**: Comentarios y adjuntos como entidades propias y su UI de gestión (incidencias-colaboracion); búsqueda/filtrado avanzado multi-criterio y dashboard de KPIs (busqueda-dashboard); envío de notificaciones por email/push (notificaciones); identificación de usuario, roles y centro de trabajo (autenticacion-roles, este spec solo los consume).

## Boundary Candidates
- Modelo de datos y CRUD de la entidad Incidencia.
- Catálogo configurable de tipos de incidencia (administración).
- Máquina de estados / ciclo de vida y asignación.
- Puntos de extensión (eventos de cambio de estado/creación) para que notificaciones y busqueda-dashboard se enganchen sin acoplarse al detalle interno.

## Out of Boundary
- Persistencia y visualización de comentarios/adjuntos (incidencias-colaboracion).
- Pantallas de filtrado avanzado y dashboard agregado (busqueda-dashboard).
- Disparo real de notificaciones por email/push (notificaciones); este spec solo debe dejar los datos/eventos necesarios (p. ej. fecha de cambio de estado) para que notificaciones los consuma.
- Login, roles y contexto de centro de trabajo (autenticacion-roles).

## Upstream / Downstream
- **Upstream**: autenticacion-roles (perfil de usuario: rol y centro de trabajo, usado para permisos de creación/asignación/cierre y visibilidad).
- **Downstream**: incidencias-colaboracion (comentarios/adjuntos cuelgan de una Incidencia existente), busqueda-dashboard (consulta y agrega datos de Incidencias), notificaciones (reacciona a eventos de creación/asignación/cambio de estado/cierre).

## Existing Spec Touchpoints
- **Extends**: Ninguno (primer spec funcional del dominio de incidencias).
- **Adjacent**: autenticacion-roles (dependencia directa de permisos y visibilidad).

## Constraints
- Almacenamiento en Dataverse (tabla Incidencias y TiposIncidencia).
- Los estados solo pueden avanzar de forma secuencial: no se puede saltar de "Nueva" a "Resuelta" u otro salto no permitido.
- Solo Supervisor o Administrador pueden asignar, cambiar de estado y cerrar incidencias.
- Debe registrarse fecha de creación, asignación y resolución como parte del ciclo de vida.
- Debe funcionar en navegador, Android, iPhone y tablet, con tiempos de carga < 3s.
