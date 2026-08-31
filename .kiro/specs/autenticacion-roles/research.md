# Research & Design Decisions

## Summary
- **Feature**: `autenticacion-roles`
- **Discovery Scope**: Nueva funcionalidad con integración compleja
- **Key Findings**:
  - `User().Email` devuelve el UPN del usuario y no garantiza el correo SMTP, por lo que el identificador estable para resolver el perfil debe ser `User().EntraObjectId`.
  - La seguridad de Dataverse es acumulativa por roles; por ello el rol de negocio único no debe inferirse desde múltiples roles técnicos, sino almacenarse de forma autoritativa en un perfil de usuario.
  - La base soportada para seguridad por centro de trabajo combina roles de Dataverse con equipos por centro, evitando confiar solo en ocultación de UI.

## Research Log

### Identidad del usuario en Canvas Apps
- **Context**: El brief exige autenticación automática con identidad corporativa y visualización de nombre/correo/rol/centro.
- **Sources Consulted**:
  - https://learn.microsoft.com/en-us/power-platform/power-fx/reference/function-user
  - https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-office365-users
  - https://learn.microsoft.com/en-us/connectors/office365users/
- **Findings**:
  - `User()` expone `FullName`, `Email`, `Image` y `EntraObjectId` para el usuario actual.
  - Microsoft documenta que `User().Email` devuelve el UPN, no necesariamente el correo SMTP mostrado en Microsoft 365.
  - Office 365 Users permite completar el perfil visible (`DisplayName`, `Mail`, etc.) pero requiere una conexión válida y puede fallar por permisos o campos no accesibles.
- **Implications**:
  - La resolución del perfil de acceso debe usar `EntraObjectId` como clave estable.
  - El correo mostrado al usuario debe tratarse como dato de presentación, no como clave de autorización.
  - La app necesita un estado bloqueante si falla la obtención del contexto de identidad o del perfil operativo.

### Seguridad de Dataverse por roles, unidades de negocio y equipos
- **Context**: El brief y el roadmap exigen base de seguridad de fila por centro de trabajo y alcance global para administradores.
- **Sources Consulted**:
  - https://learn.microsoft.com/en-us/power-platform/admin/wp-security-cds
  - https://learn.microsoft.com/en-us/power-platform/admin/security-roles-privileges
  - https://learn.microsoft.com/en-us/power-platform/admin/create-edit-security-role
  - https://learn.microsoft.com/en-us/power-platform/admin/manage-teams
- **Findings**:
  - Dataverse aplica privilegios acumulativos; un permiso amplio no puede reducirse luego con lógica correctiva.
  - Los owner teams y los Microsoft Entra group teams pueden tener roles de seguridad y poseer registros.
  - La documentación recomienda mapear límites de acceso con business units y/o grupos de Entra cuando se quiere segmentar datos por dominios organizativos.
- **Implications**:
  - La seguridad por centro debe apoyarse en registros poseídos por equipos de centro o compartidos con esos equipos, no solo en filtros de interfaz.
  - El modelo debe guardar una referencia desde el centro de trabajo hacia el equipo de seguridad que downstream specs usarán para asignar ownership o compartir datos.
  - El rol de negocio de la app debe seguir separado de los roles técnicos de Dataverse para mantener el requisito de rol único por usuario.

### Estado actual del repositorio
- **Context**: El diseño debe proponer un plan de archivos concreto y compatible con el paquete de solución existente.
- **Sources Consulted**:
  - `C:\Users\joslopez\LogsticaSurSL\LogsticaSurSL.cdsproj`
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Solution.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Customizations.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\CanvasApps\jlb_logsticasur_95873.meta.xml`
- **Findings**:
  - El repositorio contiene una solución Dataverse vacía desde el punto de vista de entidades, roles y flujos; hoy solo incluye una canvas app como componente raíz.
  - La canvas app no declara todavía conexiones, referencias de base de datos ni dependencias CDS.
  - El trabajo es greenfield dentro de una solución ya creada, por lo que el cambio correcto es extender el paquete de solución existente y la app actual.
- **Implications**:
  - El diseño debe crear explícitamente entidades, roles y referencias de conexión nuevas.
  - Las tareas deben asumir modificación del único artefacto actual de app, por lo que la mayoría del trabajo en UI no es paralelizable de forma segura.

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Solo filtros de UI | Resolver rol y centro en la app y ocultar pantallas/acciones | Rápido de construir | No protege datos si otra pantalla, API o spec omite el filtro | Rechazado por insuficiente para seguridad base |
| Business unit por centro | Modelar cada centro como BU y resolver visibilidad con roles por BU | Soporte nativo fuerte | Mayor coste operativo y complejidad temprana para un proyecto v1 pequeño | Válido pero sobredimensionado para esta fase |
| Perfil autoritativo + equipos por centro | Rol de negocio en perfil; centro con referencia a equipo de seguridad; UI + seguridad Dataverse alineadas | Aísla responsabilidades, soporta rol único y deja seam claro para specs consumidoras | Requiere disciplina de aprovisionamiento de equipos | **Seleccionado** |

## Design Decisions

### Decision: Resolver identidad por Entra Object ID
- **Context**: La app debe identificar al usuario de forma automática y sin ambigüedad.
- **Alternatives Considered**:
  1. Buscar por correo/UPN.
  2. Buscar por `EntraObjectId`.
- **Selected Approach**: Resolver el perfil por `User().EntraObjectId` y conservar correo/nombre solo como datos mostrados.
- **Rationale**: Evita inconsistencias entre UPN y SMTP, y se alinea con el identificador estable devuelto por Power Fx.
- **Trade-offs**: Requiere almacenar o reflejar el identificador de Entra en el perfil del usuario.
- **Follow-up**: Verificar durante implementación la disponibilidad del campo en todas las superficies usadas por la app.

### Decision: Separar rol de negocio de privilegios técnicos
- **Context**: El requisito pide un único rol de negocio por usuario, mientras que Dataverse permite múltiples security roles acumulativos.
- **Alternatives Considered**:
  1. Inferir rol de negocio desde roles de seguridad.
  2. Guardar el rol en una tabla de perfil de usuario y usar roles técnicos solo para privilegios base.
- **Selected Approach**: Mantener `Operario`, `Supervisor` y `Administrador` como una columna de selección única en el perfil de usuario.
- **Rationale**: Garantiza unicidad y evita que la acumulación de roles técnicos cree ambigüedad funcional.
- **Trade-offs**: Obliga a mantener sincronía entre perfil funcional y asignaciones técnicas.
- **Follow-up**: Añadir validaciones de unicidad y escenarios de bloqueo por perfil inválido.

### Decision: Base de seguridad por centro usando equipos
- **Context**: Los specs posteriores necesitan una base de seguridad por centro reutilizable.
- **Alternatives Considered**:
  1. Filtros manuales en cada pantalla.
  2. Compartición ad hoc por registro sin modelo persistente.
  3. Referencia de equipo de seguridad por centro para ownership/sharing downstream.
- **Selected Approach**: Cada centro de trabajo referencia un equipo de seguridad de Dataverse o un Microsoft Entra group team que downstream specs usarán para ownership o compartición de registros del centro.
- **Rationale**: Permite que Supervisores accedan al ámbito de su centro con soporte nativo de Dataverse y deja a los Operarios fuera de un acceso masivo no deseado.
- **Trade-offs**: Parte del aprovisionamiento vive en la configuración del entorno y no solo en archivos del repositorio.
- **Follow-up**: Validar en implementación el patrón exacto de ownership para la tabla `Incidencias` y consumidores.

### Decision: Mantener un diseño mínimo y sin capa de autenticación personalizada
- **Context**: El stack ya impone Entra ID y Power Apps como puerta de entrada.
- **Alternatives Considered**:
  1. Crear una capa personalizada de autenticación/autorización.
  2. Adoptar identidad nativa de Power Apps + perfil Dataverse + controles de seguridad de plataforma.
- **Selected Approach**: Adoptar la identidad nativa y limitar la lógica propia a resolución del contexto de acceso y reglas funcionales de rol.
- **Rationale**: Reduce superficie técnica y se alinea con la decisión de roadmap de minimizar la pila.
- **Trade-offs**: La experiencia depende de conexiones y permisos de plataforma bien configurados.
- **Follow-up**: Añadir pruebas de arranque y mensajes de error accionables para fallos de conexión o perfil.

## Risks & Mitigations
- Dependencia del conector Office 365 Users para datos de presentación — Mitigar con fallback a `User().FullName` y `User().Email` si el perfil enriquecido no responde, manteniendo el acceso bloqueado solo cuando falla la identidad/autorización.
- Aprovisionamiento incompleto de centros, equipos o perfiles — Mitigar con validaciones de perfil completo y con una pantalla bloqueante que no exponga datos.
- Cambio futuro del contrato de perfil o centro que rompa specs consumidoras — Mitigar documentando claves, campos obligatorios y triggers de revalidación en el diseño.
- Confianza excesiva en la UI para seguridad — Mitigar usando Dataverse como barrera de datos y reservando la UI para experiencia, no para control exclusivo.

## References
- [User function - Power Platform](https://learn.microsoft.com/en-us/power-platform/power-fx/reference/function-user) — propiedades del usuario actual y nota sobre UPN.
- [Connect to Office 365 Users connection from Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-office365-users) — perfil enriquecido y requisitos de conexión.
- [Office 365 Users connector reference](https://learn.microsoft.com/en-us/connectors/office365users/) — limitaciones operativas y campos disponibles.
- [Security concepts in Microsoft Dataverse](https://learn.microsoft.com/en-us/power-platform/admin/wp-security-cds) — roles, business units y estructura de acceso.
- [Security roles and privileges for Dataverse](https://learn.microsoft.com/en-us/power-platform/admin/security-roles-privileges) — acumulación de privilegios y herencia.
- [Create or edit a security role to manage access](https://learn.microsoft.com/en-us/power-platform/admin/create-edit-security-role) — privilegios mínimos y gestión exportable en soluciones.
- [Teams in Dataverse](https://learn.microsoft.com/en-us/power-platform/admin/manage-teams) — owner teams y Entra group teams para acceso por equipos.
