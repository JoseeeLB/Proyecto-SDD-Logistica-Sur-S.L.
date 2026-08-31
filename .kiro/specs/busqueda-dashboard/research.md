# Research & Design Decisions

## Summary
- **Feature**: `busqueda-dashboard`
- **Discovery Scope**: Extension
- **Key Findings**:
  - El repositorio actual solo contiene la canvas app empaquetada y los manifiestos base de solución; no hay fuentes de datos ni vistas de Dataverse ya registradas para consulta de incidencias.
  - `incidencias-core` ya fija el contrato autoritativo de `jlb_incidencia` y sus campos clave (`jlb_estado`, `jlb_prioridad`, `jlb_tipoincidenciaid`, `jlb_fechacreacion`, `jlb_fecharesolucion`, `jlb_responsableid`, `jlb_centrotrabajoid`), que este spec debe consumir sin alterar.
  - La documentación oficial de Power Apps recomienda consultas delegables, filtrado server-side y evitar `ClearCollect()` masivo; esto condiciona el diseño del buscador y del dashboard para cumplir el objetivo de menos de 3 segundos.

## Research Log

### Extensión sobre la solución actual
- **Context**: Era necesario identificar dónde encaja la nueva capa de consulta dentro del repositorio fuente.
- **Sources Consulted**: `src\CanvasApps\jlb_logsticasur_95873.meta.xml`, `src\Other\Solution.xml`, `src\Other\Customizations.xml`.
- **Findings**:
  - La solución contiene una única canvas app `jlb_logsticasur_95873` como root component.
  - `DatabaseReferences`, `ConnectionReferences` y `CdsDependencies` están vacíos en el metadato actual de la app.
  - No existen todavía carpetas exportadas de entidades o vistas para incidencias dentro de `src\`.
- **Implications**: La implementación debe registrar explícitamente las dependencias Dataverse de consulta y exportar las vistas/metadatos necesarios junto con la app.

### Contratos upstream obligatorios
- **Context**: La especificación depende de `incidencias-core` y `autenticacion-roles` y no puede redefinir su modelo.
- **Sources Consulted**: `.kiro\specs\incidencias-core\requirements.md`, `.kiro\specs\incidencias-core\design.md`, `.kiro\specs\autenticacion-roles\design.md`.
- **Findings**:
  - El rol de negocio autorizado es único y solo admite `Operario`, `Supervisor` o `Administrador`.
  - El centro de trabajo vigente llega desde `jlb_perfilusuario.jlb_centrotrabajoid` y el alcance downstream se expresa mediante `dataScope` (`self`, `center`, `global`).
  - La tabla `jlb_incidencia` ya publica los campos exactos que necesita esta capa de consulta y `busqueda-dashboard` aparece como consumidor downstream de `Estado`, `Prioridad`, `TipoIncidencia`, `CentroTrabajo`, `FechaCreacion` y `FechaResolucion`.
- **Implications**: El diseño debe reutilizar nombres exactos de rol y campos, y limitarse a superficies de lectura, filtros, agregaciones y navegación de consulta.

### Rendimiento y delegación en Power Apps
- **Context**: El feature tiene un objetivo explícito de menos de 3 segundos con 500 usuarios activos.
- **Sources Consulted**:
  - Microsoft Learn: Delegation overview in canvas apps
  - Microsoft Learn: Optimized query data patterns in Power Apps
  - Microsoft Learn: Speed up app or page load in Power Apps
- **Findings**:
  - Power Apps devuelve resultados completos y correctos sobre conjuntos grandes solo cuando la fórmula es delegable al origen de datos.
  - El patrón recomendado es usar una única tabla o vista, prefiltrada en servidor y apoyada en columnas indexadas.
  - `ClearCollect()` masivo y los lookups repetidos por fila empeoran sensiblemente el tiempo de carga y pueden romper exactitud funcional.
- **Implications**: El buscador debe usar filtros delegables sobre `jlb_incidencia` y el dashboard debe evitar agregaciones cliente no delegables sobre conjuntos grandes.

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Cargar incidencias completas en colección local | Precargar todos los registros visibles y filtrar/agregar en cliente | Implementación rápida | Riesgo alto de tiempos de carga, límites de delegación y resultados parciales | Rechazado |
| Consultas delegables directas sobre `jlb_incidencia` con vistas ligeras | Filtrado server-side, columnas mínimas e iteración de KPIs sobre dominios pequeños | Mejor alineación con exactitud y rendimiento | Requiere disciplina de fórmulas delegables y vistas exportadas | Seleccionado |
| Tabla agregada adicional para KPIs | Persistir resúmenes precalculados por centro/estado/prioridad | Muy rápida en lectura | Introduce ownership nuevo, sincronización y riesgo de datos obsoletos | Fuera de alcance v1 |

## Design Decisions

### Decision: Adaptador explícito de alcance de consulta
- **Context**: La búsqueda y el dashboard deben obedecer el mismo perímetro por rol sin duplicar reglas en cada control.
- **Alternatives Considered**:
  1. Aplicar condiciones de rol dentro de cada pantalla por separado.
  2. Derivar un contrato único de consulta a partir de `AccessContext`.
- **Selected Approach**: Derivar un `SearchScopeContext` único con `role`, `dataScope`, `centroTrabajoId`, `responsablePerfilId` y permisos visibles de consulta, consumido por búsqueda y dashboard.
- **Rationale**: Reduce deriva entre pantallas y facilita validar que ningún filtro amplía el alcance autorizado.
- **Trade-offs**: Añade una capa conceptual de estado compartido dentro de la app.
- **Follow-up**: Verificar en implementación que deep links y navegación desde dashboard reutilizan el mismo contrato.

### Decision: KPI fan-out con consultas delegables por dominio pequeño
- **Context**: El dashboard requiere conteos y distribuciones exactas sin depender de agregaciones cliente no delegables.
- **Alternatives Considered**:
  1. `GroupBy()` o agregación local sobre resultados cargados.
  2. Repetir consultas delegables separadas para KPIs base y distribuciones por prioridad/centro.
- **Selected Approach**: Ejecutar consultas delegables independientes para abiertas, cerradas, tiempo medio de resolución y distribuciones sobre dominios acotados de prioridad y centros visibles.
- **Rationale**: Prioriza exactitud funcional y evita truncado de datos por límites de cliente.
- **Trade-offs**: Puede aumentar el número de consultas si el número de centros crece.
- **Follow-up**: Medir con Monitor la latencia real del dashboard y revisar si la distribución por centro necesita afinado adicional.

### Decision: Navegación de consulta con reutilización del detalle existente
- **Context**: Esta especificación no debe duplicar ni reimplementar el lifecycle de `incidencias-core`.
- **Alternatives Considered**:
  1. Crear un detalle nuevo específico para búsqueda.
  2. Navegar al detalle existente en modo consulta, ocultando acciones fuera de alcance.
- **Selected Approach**: Reutilizar el detalle de incidencia de la app con un modo de entrada de solo consulta originado desde búsqueda o dashboard.
- **Rationale**: Minimiza superficie funcional y preserva una única vista canónica de la incidencia.
- **Trade-offs**: Requiere guards visuales y de navegación más explícitos para no exponer acciones del spec upstream.
- **Follow-up**: Validar que el contexto de retorno conserva filtros y origen de navegación.

## Risks & Mitigations
- Riesgo de fórmulas no delegables al combinar demasiados filtros — Mitigación: limitar el diseño a operadores delegables, vistas ligeras y validación con Monitor.
- Riesgo de demasiadas consultas para el KPI por centro en alcance global — Mitigación: usar solo centros activos visibles, cargar dominios pequeños y medir con datos reales antes de ampliar la cardinalidad.
- Riesgo de que el detalle reutilizado muestre acciones de mutación no deseadas — Mitigación: introducir un modo explícito de consulta y guards de entrada desde esta capa.
- Riesgo de deriva por ausencia de `product.md`, `tech.md` y `structure.md` en `.kiro\steering` — Mitigación: anclar decisiones en `roadmap.md` y en los specs upstream aprobados.

## References
- [Understand delegation in a canvas app](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/delegation-overview) — base para filtros delegables y exactitud sobre conjuntos grandes.
- [Optimized query data patterns in Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/optimized-query-data-patterns) — patrón de vista única prefiltrada e índices.
- [Speed up app or page load in Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/fast-app-page-load) — recomendaciones para evitar `ClearCollect()` masivo y reducir tiempo de carga.
