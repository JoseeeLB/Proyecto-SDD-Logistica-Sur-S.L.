# Roadmap

## Overview
Aplicación en Power Apps (canvas app sobre Dataverse) para que los empleados de Logística Sur S.L. registren, consulten, actualicen y cierren incidencias operativas de almacén desde móvil y escritorio, sustituyendo la gestión actual basada en llamadas, correos y hojas Excel. El objetivo es reducir retrasos, garantizar trazabilidad y eliminar errores de información.

## Approach Decision
- **Chosen**: Power Apps canvas app + Dataverse como almacén único de datos y adjuntos, Entra ID para autenticación/roles, Power Automate para notificaciones (email Outlook + push de Power Apps). Metodología SDD con VS Code + extensión Power Platform Tools.
- **Why**: Minimiza superficie tecnológica (un solo lugar para datos, adjuntos, seguridad y auditoría), aprovecha el catálogo de conectores nativos de Power Platform para notificaciones, y se alinea con la restricción explícita del usuario de usar Power Apps + Dataverse.
- **Rejected alternatives**: Integración con SharePoint Online para adjuntos (fuera de alcance v1), Power BI para dashboard (los KPIs se muestran dentro de la propia app en v1), clasificación IA de incidencias y escaneo QR (explícitamente fuera de alcance).

## Scope
- **In**: Autenticación por roles, alta/consulta/edición/cierre de incidencias, catálogo configurable de tipos de incidencia, ciclo de vida con transiciones secuenciales, búsqueda y filtrado por centro de trabajo/rol, comentarios inmutables, adjuntos (imágenes/PDF) en Dataverse, dashboard de KPIs (por centro y global), notificaciones por email/push ante eventos clave.
- **Out (v1)**: Integración con SharePoint Online, dashboard en Power BI, clasificación automática de incidencias con IA, escaneo de códigos QR para ubicaciones, modo offline completo, notificaciones push nativas fuera de la app de Power Apps.

## Constraints
- Plataforma: Power Apps (canvas app) + VS Code con extensión Power Platform Tools.
- Almacenamiento de datos y adjuntos: Dataverse (single source of truth).
- Autenticación: Microsoft 365 / Entra ID; rol único por usuario (Operario, Supervisor o Administrador).
- Visibilidad: un supervisor solo ve/gestiona incidencias de su centro de trabajo; un operario solo ve las incidencias que creó o que le fueron asignadas.
- El historial de comentarios es inmutable (no editable ni borrable).
- Rendimiento: tiempo de carga < 3s en consultas habituales; soporte 500 usuarios activos concurrentes.
- Compatibilidad: navegador web, Android, iPhone y tablets.
- Auditoría: se debe registrar creación, modificaciones, cambios de estado y asignaciones (RNF-04).
- Disponibilidad 24x7 con copias de seguridad de los datos (RNF-05, heredado de la plataforma Dataverse/Power Platform).

## Boundary Strategy
- **Why this split**: El documento original cubre 10 requisitos funcionales y 24 criterios de aceptación en un solo flujo de trabajo, lo que produciría más de 20 tareas si se implementara como un único spec. Se separan las responsabilidades por dominio técnico y de negocio: identidad/autorización (transversal y consumida por todo lo demás), el núcleo transaccional de incidencias (modelo de datos, alta, catálogo, ciclo de vida), la colaboración sobre una incidencia ya creada (comentarios/adjuntos), la capa de consulta agregada (búsqueda/dashboard) y la integración de notificaciones (Power Automate), que es un consumidor de eventos del núcleo y puede evolucionar independientemente.
- **Shared seams to watch**: (1) El modelo de roles y centro de trabajo definido en `autenticacion-roles` (contrato canónico `AccessContext`, con `perfilUsuarioId` y `dataScope`) es consumido por reglas de visibilidad en `incidencias-core`, `incidencias-colaboracion`, `busqueda-dashboard` y `notificaciones` — cualquier cambio de esquema de roles o del contrato `AccessContext` debe propagarse a los cuatro. (2) Las transiciones de estado y las fechas (`FechaAsignacion`, `FechaResolucion`) viven en `incidencias-core` pero disparan eventos consumidos por `notificaciones`, incluyendo el creador de la incidencia (`creadorSystemUserId`, proyección pública de `createdby`) como campo publicado del contrato de integración. (3) El catálogo de tipos de incidencia (RF-03) se administra en `incidencias-core` pero se referencia en `busqueda-dashboard` como filtro y en formularios de otros specs. (4) El contrato de navegación de detalle de incidencia (`IncidentDetailNavigationInput`/`Result`, modo `manage` vs `read-only`) se define en `incidencias-core` y es consumido verbatim por `busqueda-dashboard` para navegar desde el listado/dashboard sin duplicar la pantalla de detalle.

## Specs (dependency order)
- [x] autenticacion-roles -- Identificación con Entra ID, roles (Operario/Supervisor/Administrador) y contexto de centro de trabajo. Dependencies: none
- [x] incidencias-core -- Modelo de datos de Incidencias, alta de incidencias, catálogo configurable de tipos, ciclo de vida y asignación con transiciones secuenciales. Dependencies: autenticacion-roles
- [x] incidencias-colaboracion -- Comentarios inmutables y adjuntos (imágenes/PDF) asociados a una incidencia. Dependencies: incidencias-core, autenticacion-roles
- [x] busqueda-dashboard -- Búsqueda/filtrado de incidencias y dashboard de KPIs con alcance por rol/centro de trabajo. Dependencies: incidencias-core, autenticacion-roles
- [x] notificaciones -- Flujos Power Automate para notificación por email Outlook y push de Power Apps ante creación, asignación, cambio de estado y cierre. Dependencies: incidencias-core, autenticacion-roles
