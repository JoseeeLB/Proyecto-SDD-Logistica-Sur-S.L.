# Requirements Document

## Introduction
Los empleados de Logística Sur S.L. que gestionan incidencias operativas de almacén necesitan acceder a la aplicación con su identidad corporativa y con un alcance de uso alineado con su rol y su centro de trabajo. Actualmente la gestión se realiza por llamadas, correos y hojas Excel sin control de acceso, sin modelo de roles y sin contexto de centro en la solución. La aplicación debe identificar automáticamente a cada usuario, resolver su rol único y su centro de trabajo, y restringir pantallas, acciones y datos al contexto autorizado.

## Boundary Context
- **In scope**: identificación automática con identidad corporativa, resolución del perfil operativo del usuario, rol de negocio único, contexto de centro de trabajo, bloqueo de usuarios no válidos, restricción visible de pantallas y acciones por rol, y base de alcance de datos por centro para funcionalidades dependientes.
- **Out of scope**: altas y bajas de cuentas corporativas, mantenimiento administrativo de tipos de incidencia, CRUD de incidencias, comentarios, adjuntos, dashboard, notificaciones y una pantalla dedicada para administrar usuarios y roles.
- **Adjacent expectations**: IT mantiene las cuentas corporativas fuera de esta funcionalidad; las funcionalidades posteriores consumirán el rol y el centro resueltos aquí para aplicar sus propias reglas de negocio sin redefinir la identidad del usuario.

## Requirements

### Requirement 1: Acceso corporativo a la aplicación
**Objective:** Como empleado de Logística Sur S.L., quiero entrar en la aplicación con mi identidad corporativa, para acceder sin credenciales locales ni accesos informales.

#### Acceptance Criteria
1.1 When la persona abre la aplicación con una sesión corporativa válida, the Aplicación de Incidencias shall identificarla sin solicitar credenciales locales ni una autenticación adicional dentro de la propia app.

1.2 If la persona no dispone de una cuenta corporativa válida o no tiene permiso para usar la aplicación, the Aplicación de Incidencias shall bloquear el acceso a la experiencia funcional y mostrar un mensaje de acceso denegado.

1.3 While la identidad del usuario se está resolviendo al iniciar la aplicación, the Aplicación de Incidencias shall impedir que se muestren datos operativos o acciones editables.

1.4 The Aplicación de Incidencias shall mostrar al usuario autenticado su nombre visible y su identificador de correo corporativo dentro de la experiencia inicial.

### Requirement 2: Perfil operativo único por usuario
**Objective:** Como responsable del proceso, quiero que cada usuario entre con un único rol de negocio y un único centro de trabajo vigentes, para evitar permisos ambiguos.

#### Acceptance Criteria
2.1 When un usuario autenticado accede a la aplicación, the Aplicación de Incidencias shall resolver exactamente un rol de negocio y un centro de trabajo asociados a ese usuario antes de habilitar la navegación funcional.

2.2 If el usuario no tiene un rol asignado, tiene más de un rol efectivo o no tiene un centro de trabajo válido, the Aplicación de Incidencias shall bloquear su uso operativo y mostrar instrucciones para solicitar regularización.

2.3 The Aplicación de Incidencias shall tratar los roles Operario, Supervisor y Administrador como mutuamente excluyentes para cada usuario.

2.4 While la sesión permanezca activa, the Aplicación de Incidencias shall mantener visible el rol y el centro de trabajo vigentes del usuario en un punto consistente de la experiencia.

### Requirement 3: Autorización funcional por rol
**Objective:** Como usuario de la app, quiero ver solo las capacidades autorizadas para mi rol, para no acceder a acciones que no me corresponden.

#### Acceptance Criteria
3.1 When el usuario navega por la aplicación, the Aplicación de Incidencias shall mostrar únicamente las pantallas, acciones y opciones que correspondan a su rol vigente.

3.2 If un usuario intenta acceder por navegación directa, enlace profundo o acción indirecta a una capacidad no autorizada, the Aplicación de Incidencias shall denegar la operación sin exponer datos de esa capacidad.

3.3 When la asignación de rol o centro de trabajo del usuario cambie y el usuario vuelva a cargar la aplicación, the Aplicación de Incidencias shall aplicar el nuevo alcance desde el siguiente inicio.

3.4 Where una capacidad está reservada a supervisión o administración, the Aplicación de Incidencias shall comunicar al usuario que no dispone de permisos en lugar de fallar silenciosamente.

### Requirement 4: Alcance de datos por centro de trabajo
**Objective:** Como responsable de seguridad operativa, quiero que el acceso a los datos parta del centro de trabajo autorizado, para evitar visibilidad fuera del ámbito correspondiente.

#### Acceptance Criteria
4.1 When el usuario consulta datos operativos cubiertos por esta base de seguridad, the Aplicación de Incidencias shall limitar la visibilidad al alcance permitido por su rol y su centro de trabajo.

4.2 If un registro o una acción pertenece a un centro de trabajo fuera del alcance autorizado del usuario, the Aplicación de Incidencias shall impedir su visualización y su uso.

4.3 While el usuario tenga rol Supervisor, the Aplicación de Incidencias shall tratar su centro de trabajo como límite máximo de gestión salvo que una funcionalidad posterior defina un alcance más restrictivo.

4.4 While el usuario tenga rol Administrador, the Aplicación de Incidencias shall permitir alcance global sobre los datos y configuraciones que dependan de este modelo de autorización.

4.5 Where una funcionalidad posterior aplique reglas adicionales por creador, responsable o estado, the Aplicación de Incidencias shall seguir usando el rol y el centro de trabajo autenticados como contexto base de autorización.

### Requirement 5: Continuidad y compatibilidad del acceso
**Objective:** Como empleado que usa distintos dispositivos, quiero un acceso coherente y rápido, para poder entrar en la app sin diferencias según el cliente.

#### Acceptance Criteria
5.1 When un usuario con identidad válida y perfil completo inicia la aplicación desde navegador, Android, iPhone o tablet, the Aplicación de Incidencias shall ofrecer la misma resolución de acceso sin pasos manuales específicos del dispositivo.

5.2 If la resolución de identidad o del perfil no puede completarse, the Aplicación de Incidencias shall mostrar un estado bloqueante con una explicación accionable y sin revelar datos parciales.

5.3 When el acceso inicial se resuelve correctamente en condiciones habituales de uso, the Aplicación de Incidencias shall dejar lista la experiencia autorizada inicial en 3 segundos o menos.

5.4 If los servicios corporativos de identidad o permisos no estén disponibles, the Aplicación de Incidencias shall mantener el acceso bloqueado y mostrar un mensaje de indisponibilidad temporal.
