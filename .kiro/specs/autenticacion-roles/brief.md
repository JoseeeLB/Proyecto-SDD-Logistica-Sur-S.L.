# Brief: autenticacion-roles

## Problem
Los empleados de Logística Sur S.L. necesitan acceder a la app de incidencias con su identidad corporativa y ver únicamente los datos y acciones que corresponden a su rol y centro de trabajo. Sin un control de acceso robusto, cualquier usuario podría ver o modificar incidencias fuera de su ámbito, o usuarios sin cuenta corporativa podrían acceder a datos sensibles de operación.

## Current State
No existe hoy control de acceso: la gestión de incidencias se hace por llamadas, correos y hojas Excel sin restricción de visibilidad por rol o centro. En Power Apps/Dataverse no hay todavía modelo de seguridad, roles ni relación usuario-centro de trabajo definidos.

## Desired Outcome
Al iniciar la app, el usuario se identifica automáticamente vía Entra ID (Microsoft 365) y ve su nombre, correo, rol (Operario, Supervisor o Administrador) y centro de trabajo. El acceso a pantallas y acciones se restringe según ese rol. Un usuario sin cuenta válida de Microsoft 365 no puede iniciar sesión ni ver ningún dato.

## Approach
Usar autenticación nativa de Power Apps con Entra ID (conector Office 365 Users / contexto de usuario) combinada con un modelo de seguridad de Dataverse (roles de seguridad + tabla/relación de perfil de usuario que almacena rol de negocio y centro de trabajo). El rol de negocio (Operario/Supervisor/Administrador) se modela como columna en una tabla de perfil vinculada al usuario de Dataverse (Systemuser), de solo un valor por usuario. Las pantallas y controles de la app consultan este perfil para mostrar/ocultar funcionalidad, y las reglas de seguridad de fila (row-level security) de Dataverse restringen el acceso a nivel de datos por centro de trabajo.

## Scope
- **In**: Login/identificación automática con Entra ID; lectura de nombre/correo desde Microsoft 365; tabla de perfil de usuario en Dataverse con rol de negocio (único) y centro de trabajo; bloqueo de acceso a usuarios sin cuenta válida; ocultación/restricción de pantallas y acciones según rol en la capa de UI de la canvas app; base de seguridad de fila por centro de trabajo que consumirán otros specs.
- **Out**: Pantalla de administración de tipos de incidencia (vive en incidencias-core); lógica de visibilidad específica de incidencias, comentarios o adjuntos (cada spec consumidor implementa sus propias reglas usando este perfil); gestión de altas/bajas de usuarios en Entra ID (se asume gestionado por IT vía Microsoft 365 Admin Center); pantalla de administración de usuarios/roles en sí (si se requiere una UI de gestión de usuarios, se puede evaluar como extensión posterior).

## Boundary Candidates
- Identificación y lectura de contexto de usuario (Entra ID / Office 365 Users connector).
- Modelo de datos de perfil de usuario (rol de negocio + centro de trabajo) en Dataverse.
- Enforcement de UI (mostrar/ocultar según rol) dentro de la canvas app.
- Enforcement de datos (row-level security / filtros base por centro de trabajo) reutilizable por otros specs.

## Out of Boundary
- CRUD de incidencias, catálogo de tipos, ciclo de vida y asignación (incidencias-core).
- Comentarios y adjuntos (incidencias-colaboracion).
- Filtros de búsqueda y KPIs de dashboard (busqueda-dashboard).
- Envío de notificaciones (notificaciones).

## Upstream / Downstream
- **Upstream**: Tenant de Microsoft 365 / Entra ID de Logística Sur S.L. (cuentas y grupos ya existentes, gestionados fuera de este spec).
- **Downstream**: incidencias-core, incidencias-colaboracion y busqueda-dashboard consumen el perfil de usuario (rol, centro de trabajo) para sus reglas de visibilidad y permisos de acción.

## Existing Spec Touchpoints
- **Extends**: Ninguno (spec base, sin specs previos en el proyecto).
- **Adjacent**: incidencias-core (consume rol/centro de trabajo para permisos de creación/asignación/cierre), busqueda-dashboard (consume rol/centro de trabajo para alcance de filtros y KPIs).

## Constraints
- Autenticación exclusivamente vía Microsoft 365 / Entra ID; no se soportan cuentas locales ni otros proveedores de identidad.
- Rol único por usuario: un usuario no puede tener simultáneamente dos roles (Operario, Supervisor, Administrador).
- Debe funcionar en navegador, Android, iPhone y tablet.
- Los datos y adjuntos que consuman este perfil se almacenan en Dataverse.
