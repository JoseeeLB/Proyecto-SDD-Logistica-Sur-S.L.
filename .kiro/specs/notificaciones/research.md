## Summary
- **Feature**: `notificaciones`
- **Discovery Scope**: Complex Integration
- **Key Findings**:
  - El trigger de Dataverse `When a row is added, modified or deleted` permite filtrar actualizaciones por columnas, pero no admite columnas lookup en `Select columns`; por ello la detección de asignaciones debe apoyarse en `jlb_fechaasignacion`, no en `jlb_responsableid`.
  - El conector Power Apps Notification envía push a una app concreta usando su URL o ID y acepta destinatarios por email o UPN, lo que permite reutilizar el correo corporativo del perfil de usuario.
  - Los flujos dentro de soluciones deben usar connection references y se benefician de environment variables para parámetros de ALM como el App ID de Power Apps y banderas de apertura de la app.
  - El repositorio actual usa el formato XML legado (`Other\Solution.xml` + `Other\Customizations.xml`); los cloud flows modernos se exportan como JSON en `Workflows`, por lo que el feature debe validar explícitamente el round-trip de exportación/importación.

## Research Log

### Triggers Dataverse para eventos de incidencia
- **Context**: El feature depende de detectar creación, asignación, cambio de estado y cierre sin duplicar la lógica de `incidencias-core`.
- **Sources Consulted**:
  - https://learn.microsoft.com/en-us/power-automate/dataverse/create-update-delete-trigger
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\incidencias-core\design.md`
- **Findings**:
  - El trigger se ejecuta después de guardar el cambio confirmado en Dataverse.
  - `Select columns` solo filtra actualizaciones y no soporta columnas lookup.
  - `incidencias-core` ya publica `jlb_estado`, `jlb_fechaasignacion`, `jlb_fecharesolucion`, `jlb_responsableid`, `IdIncidencia` y el contrato de que `FechaAsignacion` se actualiza en asignación o reasignación.
- **Implications**:
  - Creación debe vivir en un flujo propio disparado por `Create`.
  - Asignación debe dispararse por cambios en `jlb_fechaasignacion` y luego leer el responsable actual confirmado.
  - Cambio de estado y cierre deben usar flujos separados sobre `jlb_estado` para mantener eventos exclusivos y trazables.

### Canales soportados y parámetros de ALM
- **Context**: El usuario restringe v1 a Outlook email y push nativa dentro de Power Apps.
- **Sources Consulted**:
  - https://learn.microsoft.com/en-us/connectors/powerappsnotification/
  - https://learn.microsoft.com/en-us/connectors/office365/
  - https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference
  - https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables
- **Findings**:
  - Power Apps Notification necesita una conexión asociada a una app concreta y permite `recipients`, `message`, `openApp` y parámetros clave-valor.
  - Los solution-aware flows usan connection references para conectores.
  - Las environment variables son el mecanismo recomendado para transportar referencias cambiantes entre entornos.
- **Implications**:
  - La solución debe incluir connection references para Outlook y Power Apps Notification.
  - Debe existir al menos una environment variable para el App ID/URL de la canvas app y otra para el comportamiento de apertura de la app.
  - Los avisos push pueden llevar `incidentId` como parámetro para abrir la incidencia correcta dentro de la app.

### Manejo de errores y reintentos
- **Context**: El alcance exige reintentos básicos y trazabilidad de fallos sin bloquear otras entregas.
- **Sources Consulted**:
  - https://learn.microsoft.com/en-us/power-automate/guidance/coding-guidelines/error-handling
  - https://learn.microsoft.com/en-us/power-automate/limits-and-config#retry-policy
- **Findings**:
  - Power Automate recomienda agrupar acciones en scopes `Try/Catch` y usar `Run after` para caminos de error.
  - Los reintentos deben apoyarse en la política del conector y en trazabilidad explícita del run para depuración.
- **Implications**:
  - Cada entrega por canal debe ejecutarse en scopes aislados para permitir continuidad parcial.
  - El diseño necesita una tabla de logs de envío por destinatario/canal y correlación con el run del flujo.

### ALM y formato de control de código fuente
- **Context**: El repositorio actual está en formato XML legado y aún no contiene flujos.
- **Sources Consulted**:
  - https://learn.microsoft.com/en-us/power-platform/alm/use-source-control-solution-files
  - https://learn.microsoft.com/en-us/power-automate/export-flow-solution
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Solution.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Customizations.xml`
- **Findings**:
  - `Customizations.xml` ya contiene el nodo `<Workflows />`, pero no hay flujos definidos.
  - Los solution-aware cloud flows se exportan como JSON dentro de la carpeta `Workflows` del zip de solución.
  - Microsoft recomienda YAML para proyectos nuevos porque el formato XML legado no soporta modern flows como formato fuente de primera clase.
- **Implications**:
  - Este spec debe fijar una convención concreta para versionar los JSON de `Workflows` y validar manualmente el empaquetado del componente en la solución actual.
  - La validación de tareas debe incluir evidencia de exportación/importación correcta de la solución con flujos activos.

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Cuatro flujos independientes por evento | Un flujo create y tres flujos update, uno por evento observable | Disparadores simples, eventos mutuamente auditables, menor ambigüedad | Duplica lógica de destinatarios/canales si no se factoriza | Elegido junto con un flujo hijo compartido |
| Dos flujos monolíticos | Un flujo create y otro update con router interno | Menos artefactos | Mayor complejidad de branching, más riesgo de solapes y falsos positivos | Rechazado por la limitación de lookup filters |
| Flujo único con branching por evento | Un solo flujo para create/update | Menos componentes visibles | No encaja con triggers distintos y complica mantenimiento | Rechazado |

## Design Decisions

### Decision: Separar los cuatro eventos en flujos de borde y un flujo hijo compartido
- **Context**: Hay cuatro eventos obligatorios con reglas de activación distintas y canales comunes.
- **Alternatives Considered**:
  1. Dos flujos monolíticos con branching.
  2. Cuatro flujos completamente duplicados.
  3. Cuatro flujos de evento + flujo hijo compartido.
- **Selected Approach**: Usar cuatro flujos solution-aware orientados a evento y un flujo hijo común para carga de configuración, resolución de destinatarios, envío por canal y logging.
- **Rationale**: Mantiene la detección de eventos simple y verificable, evita duplicar lógica de entrega y permite aislar errores por evento.
- **Trade-offs**: Introduce un artefacto adicional y una dependencia interna entre flujos.
- **Follow-up**: Verificar que el flujo hijo conserva contratos estables y no supera límites de acciones.

### Decision: Configuración central en Dataverse en lugar de environment variables puras
- **Context**: Los administradores funcionales deben poder ajustar destinatarios, canales y mensajes por evento.
- **Alternatives Considered**:
  1. Environment variables para toda la configuración.
  2. Hardcode en flujos.
  3. Tabla Dataverse con cuatro filas autoritativas y environment variables solo para parámetros de ALM.
- **Selected Approach**: Mantener una tabla `jlb_notificacionevento` como fuente funcional y reservar environment variables para App ID/URL y flags de apertura.
- **Rationale**: La configuración funcional es mantenible sin editar flujos y sigue siendo transportable en la solución.
- **Trade-offs**: Añade una tabla nueva y requiere seed inicial.
- **Follow-up**: Validar restricciones para que solo existan códigos de evento permitidos.

### Decision: Resolver asignaciones usando `jlb_fechaasignacion`
- **Context**: El trigger update no puede filtrar por lookup `jlb_responsableid`.
- **Alternatives Considered**:
  1. Filtrar por responsable lookup.
  2. Escuchar todas las actualizaciones y deducir el evento.
  3. Filtrar por `jlb_fechaasignacion` y leer el responsable confirmado.
- **Selected Approach**: Disparar el flujo de asignación cuando se incluya `jlb_fechaasignacion` en la actualización.
- **Rationale**: Reutiliza el contrato explícito de `incidencias-core` y reduce ruido de ejecuciones innecesarias.
- **Trade-offs**: Depende de que `incidencias-core` siga actualizando esa columna en cada asignación válida.
- **Follow-up**: Añadir revalidación si el core cambia esa semántica.

### Decision: Notificar a todos los supervisores activos del centro
- **Context**: `autenticacion-roles` define rol único por perfil y centro, pero no garantiza un único supervisor por centro.
- **Alternatives Considered**:
  1. Elegir un supervisor arbitrario.
  2. Fijar un supervisor único adicional en el modelo.
  3. Resolver todos los perfiles activos con rol `Supervisor` del centro.
- **Selected Approach**: Notificar a todos los supervisores activos del centro y deduplicar por canal.
- **Rationale**: Evita silencios operativos sin ampliar el modelo upstream.
- **Trade-offs**: Puede aumentar el número de avisos en centros con varios supervisores.
- **Follow-up**: Revisar si en producción conviene introducir un perfil primario en una versión posterior.

## Risks & Mitigations
- Riesgo de round-trip incompleto de flujos por el formato XML legado — Mitigación: mantener los JSON de `Workflows` bajo control de versiones y validar exportación/importación de solución en pruebas.
- Riesgo de duplicados o ruido operativo por múltiples categorías de destinatarios — Mitigación: normalizar destinatarios por email/UPN y deduplicar por evento+canal.
- Riesgo de fallo parcial de un canal que oculte el resto — Mitigación: scopes independientes por canal/destinatario y logging por intento.
- Riesgo de cambio upstream en semántica de `FechaAsignacion` o `Estado` — Mitigación: registrar revalidation triggers explícitos y pruebas de integración contra `incidencias-core`.

## References
- [Trigger flows when a row is added, modified, or deleted](https://learn.microsoft.com/en-us/power-automate/dataverse/create-update-delete-trigger) — comportamiento y limitaciones del trigger de Dataverse.
- [Power Apps Notification connector](https://learn.microsoft.com/en-us/connectors/powerappsnotification/) — parámetros del canal push in-app.
- [Office 365 Outlook connector](https://learn.microsoft.com/en-us/connectors/office365/) — canal email estándar en Power Platform.
- [Use a connection reference in a solution](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference) — patrón ALM para flujos solution-aware.
- [Use environment variables in Power Platform solutions](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables) — configuración transportable entre entornos.
- [Source control with solution files](https://learn.microsoft.com/en-us/power-platform/alm/use-source-control-solution-files) — limitaciones del formato XML legado frente a modern flows.
