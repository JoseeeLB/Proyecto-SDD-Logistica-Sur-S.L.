## Summary
- **Feature**: `incidencias-core`
- **Discovery Scope**: Extension
- **Key Findings**:
  - La solución existente solo contiene la canvas app empaquetada y los artefactos base de la solución; no existe aún un modelo funcional de incidencias, por lo que el diseño debe fijar contratos completos sin romper la frontera ya marcada por `autenticacion-roles`.
  - `autenticacion-roles` ya estableció `jlb_perfilusuario`, `jlb_centrotrabajo`, `jlb_rolnegocio`, `centroSecurityTeamId` y `AccessContext` como seam autoritativa; este spec debe consumirlos y no duplicarlos.
  - La documentación oficial de Power Apps y Dataverse favorece carga diferida en canvas apps, claves alternativas para identificadores de negocio, auditoría nativa y relaciones sin borrado en cascada para preservar trazabilidad.

## Research Log

### Arquitectura actual del repositorio
- **Context**: El feature debía integrarse en una solución Power Platform ya existente y coherente con specs previos.
- **Sources Consulted**:
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Solution.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Customizations.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\CanvasApps\jlb_logsticasur_95873.meta.xml`
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\autenticacion-roles\design.md`
- **Findings**:
  - La solución solo registra la canvas app `jlb_logsticasur_95873`; no hay tablas funcionales exportadas todavía.
  - El patrón ya aprobado para seguridad transversal es perfil de usuario + centro de trabajo + seam de equipo de centro.
  - El paquete fuente admite ampliar `src\Entities\`, `src\OptionSets\` y los metadatos de la canvas app sin introducir otra pila tecnológica.
- **Implications**:
  - El diseño puede tratar `incidencias-core` como extensión sobre un contenedor vacío, con discovery ligero centrado en integración.
  - Las relaciones con usuario, centro y responsable deben construirse sobre `jlb_perfilusuario` y `jlb_centrotrabajo`.

### Límites funcionales del roadmap y del brief
- **Context**: Era necesario aislar `incidencias-core` de colaboración, búsqueda y notificaciones.
- **Sources Consulted**:
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\steering\roadmap.md`
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\incidencias-core\brief.md`
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\autenticacion-roles\requirements.md`
- **Findings**:
  - El roadmap sitúa en este spec el modelo de datos de incidencias, el alta, el catálogo configurable, la asignación y el ciclo de vida secuencial.
  - Comentarios, adjuntos, búsqueda/dashboard y notificaciones son consumidores downstream explícitos.
  - Las reglas de autenticación, rol y centro ya están aceptadas upstream y no deben repetirse aquí.
- **Implications**:
  - El diseño debe publicar seams de integración estables: `IdIncidencia`, estado, fechas clave, responsable y centro.
  - El spec no debe absorber lógica de comentarios, adjuntos, filtros avanzados ni envío de avisos.

### Buenas prácticas de rendimiento en canvas apps
- **Context**: El roadmap y los requisitos fijan un objetivo visible de respuesta inferior a 3 segundos.
- **Sources Consulted**:
  - `https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/create-performant-apps-overview`
- **Findings**:
  - Microsoft recomienda cargar solo los datos esenciales al inicio, evitar sobrecargar `OnStart` y aprovechar paginación/carga incremental.
  - Las apps canvas de datos funcionan mejor cuando cada pantalla consulta únicamente el subconjunto requerido para su caso de uso.
- **Implications**:
  - El diseño debe separar bootstrap de acceso (upstream) del bootstrap funcional de incidencias.
  - El listado de tipos activos y la incidencia seleccionada deben cargarse bajo demanda, no mediante precarga global masiva.

### Identificadores de negocio y estabilidad de integración
- **Context**: `IdIncidencia` debe servir como referencia estable para specs downstream.
- **Sources Consulted**:
  - `https://learn.microsoft.com/en-us/power-apps/developer/data-platform/define-alternate-keys-entity`
- **Findings**:
  - Dataverse permite declarar claves alternativas sobre columnas de negocio para identificar registros más allá del GUID interno.
  - Las claves alternativas son apropiadas para integraciones y referencias estables entre capacidades.
- **Implications**:
  - `IdIncidencia` debe ser único, inmutable y tratable como clave de integración, manteniendo el GUID técnico interno de Dataverse como PK.

### Auditoría y superficies de cambio
- **Context**: El spec necesita trazabilidad básica y un seam reutilizable por notificaciones sin crear una infraestructura de eventos propia.
- **Sources Consulted**:
  - `https://learn.microsoft.com/en-us/power-apps/developer/data-platform/auditing/overview`
- **Findings**:
  - Dataverse ofrece auditoría nativa a nivel de entorno, tabla y columna para registrar acceso y cambios.
  - Los historiales auditados pueden consumirse posteriormente por procesos administrativos o de integración.
- **Implications**:
  - El diseño debe adoptar auditoría nativa para los campos críticos (`Estado`, `Responsable`, `TipoIncidencia`, `Prioridad`, `CentroTrabajo`) en lugar de inventar un log paralelo en este spec.
  - `notificaciones` puede apoyarse en los cambios persistidos y en triggers de Dataverse sin ampliar el ownership de `incidencias-core`.

### Integridad referencial y preservación del historial
- **Context**: El catálogo de tipos y las relaciones con centro/perfil no deben destruir el historial de incidencias.
- **Sources Consulted**:
  - `https://learn.microsoft.com/en-us/power-apps/developer/data-platform/configure-entity-relationship-cascading-behavior`
- **Findings**:
  - Dataverse permite controlar el comportamiento en cascada de relaciones uno-a-muchos.
  - Para dominios auditables conviene evitar borrados en cascada cuando un registro histórico depende de catálogos o perfiles.
- **Implications**:
  - Las relaciones hacia `jlb_tipoincidencia`, `jlb_centrotrabajo` y `jlb_perfilusuario` deben configurarse sin cascadas destructivas.
  - Los tipos se desactivan en lugar de eliminarse cuando ya están referenciados.

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| App canvas con lógica ad hoc por pantalla | Cada pantalla consulta y muta Dataverse directamente sin contratos comunes | Rápido de arrancar | Duplica reglas de visibilidad y estado, dificulta specs downstream | Rechazado |
| Extensión sobre seams de acceso + contratos de dominio mínimos | La app consume `AccessContext`, centraliza reglas de alcance/estado y Dataverse conserva el modelo autoritativo | Alineado con roadmap, reusable, menor deriva entre specs | Requiere definir contratos explícitos desde el inicio | Seleccionado |
| Automatización dominante con Power Automate para ciclo de vida | La app solo captura datos y delega reglas principales a flujos | Buen desacoplamiento futuro | Añade dependencia fuera del boundary y complica tiempos de respuesta | Rechazado para v1 |

## Design Decisions

### Decision: Reutilizar el seam autoritativo de autenticación y centro
- **Context**: El feature depende de permisos por rol y centro ya definidos upstream.
- **Alternatives Considered**:
  1. Duplicar campos de rol/centro en `Incidencias`
  2. Consumir `AccessContext` y relacionar la incidencia con `jlb_centrotrabajo` y `jlb_perfilusuario`
- **Selected Approach**: Consumir `AccessContext` y persistir solo referencias de dominio necesarias (`CentroTrabajo`, `Responsable`) sin reescribir autenticación.
- **Rationale**: Mantiene una única fuente de verdad para identidad y reduce revalidaciones.
- **Trade-offs**: Hace que cambios en `autenticacion-roles` obliguen a revisar este spec.
- **Follow-up**: Validar todos los lookups y reglas de visibilidad contra las claves reales del perfil.

### Decision: Usar `IdIncidencia` como identificador de negocio estable
- **Context**: Los specs downstream necesitan una referencia estable para comentarios, adjuntos, KPIs y notificaciones.
- **Alternatives Considered**:
  1. Exponer solo el GUID técnico de Dataverse
  2. Añadir `IdIncidencia` único e inmutable como identificador de negocio
- **Selected Approach**: Mantener el GUID técnico como PK interno y `IdIncidencia` como clave alternativa e identificador visible.
- **Rationale**: Facilita integración y trazabilidad operativa sin acoplar consumidores al GUID técnico.
- **Trade-offs**: Requiere regla explícita de unicidad e inicialización.
- **Follow-up**: Verificar empaquetado correcto de la clave alternativa en la solución.

### Decision: Resolver la ubicación obligatoria sin ampliar el esquema base
- **Context**: El brief exige capturar ubicación, pero la frontera de datos aprobada para `Incidencias` no incluye una columna dedicada.
- **Alternatives Considered**:
  1. Añadir una nueva columna de ubicación a la tabla
  2. Exigir un control de ubicación en la UI y normalizar su persistencia dentro del contenido operativo de la incidencia
- **Selected Approach**: Mantener la frontera de tabla actual y normalizar la ubicación dentro del contenido persistido de la incidencia, dejando explícito que un atributo dedicado sería un cambio de boundary.
- **Rationale**: Respeta el límite fijado por el usuario y conserva el comportamiento visible esperado.
- **Trade-offs**: La ubicación no queda consultable como columna independiente en esta fase.
- **Follow-up**: Si `busqueda-dashboard` necesita filtrar por ubicación, deberá abrirse una revalidación de boundary.

### Decision: Adoptar auditoría nativa de Dataverse para los hitos clave
- **Context**: El spec necesita trazabilidad básica sin apropiarse de la plataforma de notificaciones ni de un subsistema de logging propio.
- **Alternatives Considered**:
  1. Crear una tabla específica de historial
  2. Activar auditoría nativa de Dataverse sobre tabla y columnas críticas
- **Selected Approach**: Usar auditoría nativa y asegurar que los hitos principales también queden visibles en columnas de la propia incidencia.
- **Rationale**: Reduce complejidad y conserva compatibilidad con integraciones futuras.
- **Trade-offs**: La explotación avanzada del historial queda para specs o procesos posteriores.
- **Follow-up**: Confirmar que entorno y columnas quedan auditables en el paquete/despliegue.

## Risks & Mitigations
- La captura de ubicación sin columna dedicada puede quedarse corta para filtros futuros — Mitigación: declarar el límite en boundary y añadir trigger explícito de revalidación.
- El modelo de acceso de operario creado/asignado puede requerir sharing adicional además del contexto de centro — Mitigación: definir una política explícita de compartición por responsable y equipo de centro.
- La msapp empaquetada concentra múltiples cambios funcionales en un solo artefacto — Mitigación: separar responsabilidades por pantallas/contratos en diseño y reservar la integración final para tareas específicas.

## References
- [How to create performant Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/create-performant-apps-overview) — pautas de carga diferida y rendimiento en apps canvas.
- [Work with alternate keys (Microsoft Dataverse)](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/define-alternate-keys-entity) — base para `IdIncidencia` como identificador estable.
- [Microsoft Dataverse Auditing Overview for Developers](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/auditing/overview) — soporte nativo para trazabilidad.
- [Configure table relationship cascading behavior](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/configure-entity-relationship-cascading-behavior) — decisiones de relaciones sin borrado destructivo.
