# Requirements Document

## Introduction
Los empleados de Log?stica Sur S.L. que gestionan incidencias operativas de almac?n necesitan acceder a la aplicaci?n con su identidad corporativa y con un alcance de uso alineado con su rol y su centro de trabajo. Actualmente la gesti?n se realiza por llamadas, correos y hojas Excel sin control de acceso, sin modelo de roles y sin contexto de centro en la soluci?n. La aplicaci?n debe identificar autom?ticamente a cada usuario, resolver su rol ?nico y su centro de trabajo, y restringir pantallas, acciones y datos al contexto autorizado.

## Boundary Context
- **In scope**: identificaci?n autom?tica con identidad corporativa, resoluci?n del perfil operativo del usuario, rol de negocio ?nico, contexto de centro de trabajo, bloqueo de usuarios no v?lidos, restricci?n visible de pantallas y acciones por rol, y base de alcance de datos por centro para funcionalidades dependientes.
- **Out of scope**: altas y bajas de cuentas corporativas, mantenimiento administrativo de tipos de incidencia, CRUD de incidencias, comentarios, adjuntos, dashboard, notificaciones y una pantalla dedicada para administrar usuarios y roles.
- **Adjacent expectations**: IT mantiene las cuentas corporativas fuera de esta funcionalidad; las funcionalidades posteriores consumir?n el rol y el centro resueltos aqu? para aplicar sus propias reglas de negocio sin redefinir la identidad del usuario.

## Requirements

### Requirement 1: Acceso corporativo a la aplicaci?n
**Objective:** Como empleado de Log?stica Sur S.L., quiero entrar en la aplicaci?n con mi identidad corporativa, para acceder sin credenciales locales ni accesos informales.

#### Acceptance Criteria
1.1 When la persona abre la aplicaci?n con una sesi?n corporativa v?lida, the Aplicaci?n de Incidencias shall identificarla sin solicitar credenciales locales ni una autenticaci?n adicional dentro de la propia app.

1.2 If la persona no dispone de una cuenta corporativa v?lida o no tiene permiso para usar la aplicaci?n, the Aplicaci?n de Incidencias shall bloquear el acceso a la experiencia funcional y mostrar un mensaje de acceso denegado.

1.3 While la identidad del usuario se est? resolviendo al iniciar la aplicaci?n, the Aplicaci?n de Incidencias shall impedir que se muestren datos operativos o acciones editables.

1.4 The Aplicaci?n de Incidencias shall mostrar al usuario autenticado su nombre visible y su identificador de correo corporativo dentro de la experiencia inicial.

### Requirement 2: Perfil operativo ?nico por usuario
**Objective:** Como responsable del proceso, quiero que cada usuario entre con un ?nico rol de negocio y un ?nico centro de trabajo vigentes, para evitar permisos ambiguos.

#### Acceptance Criteria
2.1 When un usuario autenticado accede a la aplicaci?n, the Aplicaci?n de Incidencias shall resolver exactamente un rol de negocio y un centro de trabajo asociados a ese usuario antes de habilitar la navegaci?n funcional.

2.2 If el usuario no tiene un rol asignado, tiene m?s de un rol efectivo o no tiene un centro de trabajo v?lido, the Aplicaci?n de Incidencias shall bloquear su uso operativo y mostrar instrucciones para solicitar regularizaci?n.

2.3 The Aplicaci?n de Incidencias shall tratar los roles Operario, Supervisor y Administrador como mutuamente excluyentes para cada usuario.

2.4 While la sesi?n permanezca activa, the Aplicaci?n de Incidencias shall mantener visible el rol y el centro de trabajo vigentes del usuario en un punto consistente de la experiencia.

### Requirement 3: Autorizaci?n funcional por rol
**Objective:** Como usuario de la app, quiero ver solo las capacidades autorizadas para mi rol, para no acceder a acciones que no me corresponden.

#### Acceptance Criteria
3.1 When el usuario navega por la aplicaci?n, the Aplicaci?n de Incidencias shall mostrar ?nicamente las pantallas, acciones y opciones que correspondan a su rol vigente.

3.2 If un usuario intenta acceder por navegaci?n directa, enlace profundo o acci?n indirecta a una capacidad no autorizada, the Aplicaci?n de Incidencias shall denegar la operaci?n sin exponer datos de esa capacidad.

3.3 When la asignaci?n de rol o centro de trabajo del usuario cambie y el usuario vuelva a cargar la aplicaci?n, the Aplicaci?n de Incidencias shall aplicar el nuevo alcance desde el siguiente inicio.

3.4 Where una capacidad est? reservada a supervisi?n o administraci?n, the Aplicaci?n de Incidencias shall comunicar al usuario que no dispone de permisos en lugar de fallar silenciosamente.

### Requirement 4: Alcance de datos por centro de trabajo
**Objective:** Como responsable de seguridad operativa, quiero que el acceso a los datos parta del centro de trabajo autorizado, para evitar visibilidad fuera del ?mbito correspondiente.

#### Acceptance Criteria
4.1 When el usuario consulta datos operativos cubiertos por esta base de seguridad, the Aplicaci?n de Incidencias shall limitar la visibilidad al alcance permitido por su rol y su centro de trabajo.

4.2 If un registro o una acci?n pertenece a un centro de trabajo fuera del alcance autorizado del usuario, the Aplicaci?n de Incidencias shall impedir su visualizaci?n y su uso.

4.3 While el usuario tenga rol Supervisor, the Aplicaci?n de Incidencias shall tratar su centro de trabajo como l?mite m?ximo de gesti?n salvo que una funcionalidad posterior defina un alcance m?s restrictivo.

4.4 While el usuario tenga rol Administrador, the Aplicaci?n de Incidencias shall permitir alcance global sobre los datos y configuraciones que dependan de este modelo de autorizaci?n.

4.5 Where una funcionalidad posterior aplique reglas adicionales por creador, responsable o estado, the Aplicaci?n de Incidencias shall seguir usando el rol y el centro de trabajo autenticados como contexto base de autorizaci?n.

### Requirement 5: Continuidad y compatibilidad del acceso
**Objective:** Como empleado que usa distintos dispositivos, quiero un acceso coherente y r?pido, para poder entrar en la app sin diferencias seg?n el cliente.

#### Acceptance Criteria
5.1 When un usuario con identidad v?lida y perfil completo inicia la aplicaci?n desde navegador, Android, iPhone o tablet, the Aplicaci?n de Incidencias shall ofrecer la misma resoluci?n de acceso sin pasos manuales espec?ficos del dispositivo.

5.2 If la resoluci?n de identidad o del perfil no puede completarse, the Aplicaci?n de Incidencias shall mostrar un estado bloqueante con una explicaci?n accionable y sin revelar datos parciales.

5.3 When el acceso inicial se resuelve correctamente en condiciones habituales de uso, the Aplicaci?n de Incidencias shall dejar lista la experiencia autorizada inicial en 3 segundos o menos.

5.4 If los servicios corporativos de identidad o permisos no est?n disponibles, the Aplicaci?n de Incidencias shall mantener el acceso bloqueado y mostrar un mensaje de indisponibilidad temporal.
