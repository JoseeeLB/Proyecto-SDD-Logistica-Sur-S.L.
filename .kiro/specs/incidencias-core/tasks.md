# Implementation Plan

> Regla de coordinación de artefactos compartidos: `Solution.xml`, `Customizations.xml`, `src\CanvasApps\jlb_logsticasur_95873_DocumentUri.msapp` y `src\CanvasApps\jlb_logsticasur_95873.meta.xml` son host artifacts compartidos entre specs. Cada tarea de `incidencias-core` solo puede editar nodos, pantallas, fórmulas, data sources y metadatos estrictamente necesarios para este feature; debe preservar componentes ajenos, evitar renombrados amplios y agrupar sus cambios al área funcional descrita en su `_Boundary:_` para minimizar conflictos con otros specs.

- [ ] 1. Preparar la base funcional de incidencias en Dataverse
  - _Boundary:_ Coordina únicamente los artefactos base de `incidencias-core`: tablas `jlb_incidencia` y `jlb_tipoincidencia`, relaciones, option sets y el alta mínima en artefactos host compartidos cuando sea imprescindible. No incorpora superficies de `notificaciones`, `busqueda-dashboard` ni `incidencias-colaboracion`.
- [ ] 1.1 Definir el catálogo configurable de tipos de incidencia
  - Crear la tabla `jlb_tipoincidencia` con código único, nombre visible, estado activo/inactivo y orden visual opcional.
  - Cargar los tipos iniciales `Seguridad`, `Equipamiento`, `Infraestructura`, `Calidad`, `Transporte` y `Otros` como datos disponibles desde el primer despliegue.
  - Configurar la relación con incidencias para que los tipos usados puedan desactivarse sin perder su referencia histórica.
  - Al finalizar, la solución expone un catálogo administrable y los formularios pueden consultar una vista de tipos activos.
  - _Boundary:_ Esta tarea es dueña de `src\Entities\jlb_tipoincidencia\**`, sus vistas/datos semilla y de las referencias mínimas a este catálogo dentro de `Customizations.xml` o `Solution.xml`. No toca pantallas de detalle, lifecycle ni contratos downstream fuera del catálogo.
  - _Requirements: 1.5, 3.1, 3.2, 3.3, 3.4, 3.5_

- [ ] 1.2 Definir la tabla autoritativa de incidencias y sus hitos
  - Crear la tabla `jlb_incidencia` con `IdIncidencia`, `Titulo`, `Descripcion`, `Estado`, `Prioridad`, `TipoIncidencia`, `FechaCreacion`, `FechaAsignacion`, `FechaResolucion`, `Responsable` y `CentroTrabajo`.
  - Configurar `IdIncidencia` como identificador único e inmutable y relacionar la tabla con `jlb_centrotrabajo`, `jlb_perfilusuario` y `jlb_tipoincidencia` sin borrados en cascada destructivos.
  - Habilitar auditoría para estado, prioridad, tipo, responsable y centro, y conservar los hitos visibles en la propia incidencia.
  - Al finalizar, existe un contrato físico estable que soporta alta, asignación, resolución, cierre y consumo downstream.
  - _Boundary:_ Esta tarea es dueña de `src\Entities\jlb_incidencia\**`, relaciones con `jlb_centrotrabajo`, `jlb_perfilusuario` y `jlb_tipoincidencia`, así como de la configuración de auditoría y de la proyección de `createdby` como seam público `creadorSystemUserId`. No edita flujos ni UI downstream.
  - _Requirements: 1.2, 2.5, 4.1, 5.1, 5.4, 5.5, 6.1, 6.2, 6.4_

- [ ] 1.3 Registrar los nuevos componentes en la solución y en la app host
  - Actualizar `Solution.xml`, `Customizations.xml` y los metadatos de la canvas app con tablas, relaciones, option set de estados y dependencias de datos.
  - Verificar que la app `jlb_logsticasur_95873` reconoce `jlb_incidencia`, `jlb_tipoincidencia`, `jlb_perfilusuario` y `jlb_centrotrabajo` como fuentes disponibles.
  - Al finalizar, el paquete fuente puede publicarse sin referencias rotas entre la app y los artefactos Dataverse del feature.
  - _Boundary:_ Esta tarea es la única autorizada a editar directamente los host artifacts compartidos `Solution.xml`, `Customizations.xml`, `.msapp` y `.meta.xml` para registrar fuentes de datos, pantallas y componentes de `incidencias-core`. La edición debe limitarse a altas/inclusiones de este feature, preservando IDs y bloques ajenos para facilitar la convivencia con otros specs.
  - _Requirements: 3.2, 5.1, 6.4, 7.1_

- [ ] 2. Implementar las superficies principales de captura y consulta
  - _Boundary:_ Coordina exclusivamente las pantallas y fórmulas de `incidencias-core` para catálogo, alta y detalle. Cualquier cambio en `.msapp` o `.meta.xml` debe circunscribirse a estas superficies y no introducir comportamiento de dashboards, notificaciones ni colaboración.
- [ ] 2.1 Construir la gestión administrativa del catálogo de tipos
  - Crear la pantalla o flujo de administración que liste tipos, permita alta, edición visible y desactivación sin borrado destructivo.
  - Bloquear la entrada a cualquier usuario que no tenga rol `Administrador` y mostrar un mensaje explícito de falta de permisos.
  - Refrescar el catálogo tras cada cambio para que la disponibilidad quede visible sin ambigüedad.
  - Al finalizar, un administrador puede mantener el catálogo completo y un usuario no autorizado siempre recibe denegación visible.
  - _Boundary:_ Posee la pantalla administrativa, fórmulas Power Fx y bindings del catálogo dentro de `src\CanvasApps\jlb_logsticasur_95873_DocumentUri.msapp`, más las referencias mínimas en `.meta.xml`. No modifica el detalle de incidencias ni reglas de lifecycle.
  - _Requirements: 3.1, 3.3, 3.4, 3.5, 3.6, 7.1_

- [ ] 2.2 Construir el formulario de alta de incidencias
  - Implementar el formulario con `Título`, `Descripción`, `TipoIncidencia`, `Prioridad` y `Ubicación` como datos obligatorios visibles, mostrando también la fecha de registro.
  - Cargar solo tipos activos y validar de forma previa al guardado que no falte ningún dato obligatorio.
  - Persistir el alta con estado `Nueva`, asociando automáticamente creador y `CentroTrabajo` vigentes del usuario autenticado.
  - Al finalizar, un operario o supervisor autorizado puede registrar una incidencia válida y recibe confirmación con su `IdIncidencia`.
  - _Boundary:_ Posee los controles, validaciones y la fórmula de creación del alta en `.msapp`, junto con la lectura de `AccessContext` canónico y las referencias a `jlb_incidencia` / `jlb_tipoincidencia` en `.meta.xml`. No introduce navegación analítica ni automatizaciones externas.
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 7.1, 7.2, 7.3_

- [ ] 2.3 Construir la vista de detalle operativo de una incidencia
  - Mostrar `IdIncidencia`, `Estado`, `Prioridad`, `TipoIncidencia`, `Responsable`, `CentroTrabajo`, `FechaCreacion`, `FechaAsignacion` y `FechaResolucion` cuando existan.
  - Reservar visualmente el espacio funcional para colaboración y notificaciones sin presentar esas capacidades como disponibles en este spec.
  - Garantizar que, tras cualquier mutación confirmada, el detalle se recarga desde Dataverse para reflejar el estado real.
  - Al finalizar, la incidencia puede abrirse en una experiencia consistente que expone sus hitos y su contexto operativo actual.
  - _Boundary:_ Posee la pantalla de detalle, sus fórmulas de lectura y el seam de navegación `manage`/`read-only` dentro de `.msapp` y `.meta.xml`. Debe publicar entrada/salida compatibles con `busqueda-dashboard`, pero no implementar filtros ni dashboards de ese spec.
  - _Requirements: 2.5, 4.5, 5.5, 6.3, 7.1, 7.4_

- [ ] 3. Implementar permisos de acceso, asignación y ciclo de vida
  - _Boundary:_ Coordina reglas de acceso, responsables y transiciones del dominio `incidencias-core`. Puede tocar fórmulas Power Fx, option set de estados y metadatos de `jlb_incidencia` necesarios para enforcement, sin asumir ownership de perfiles/roles upstream.
- [ ] 3.1 Aplicar la política de alcance por rol sobre incidencias y responsables
  - Implementar las reglas de acceso para que un `Operario` solo vea incidencias creadas por él o asignadas a su perfil, un `Supervisor` gestione las de su `jlb_centrotrabajo` y un `Administrador` tenga alcance global.
  - Calcular la lista de responsables seleccionables dentro del alcance permitido para evitar asignaciones fuera del perímetro autorizado.
  - Denegar apertura o gestión de incidencias fuera de alcance antes de mostrar su contenido operativo.
  - Al finalizar, la app decide de forma determinista quién puede leer, gestionar o ser responsable de cada incidencia.
  - _Boundary:_ Posee las fórmulas/políticas que correlacionan `AccessContext.dataverseUserId`, `perfilUsuarioId`, `centroTrabajoId`, `centroSecurityTeamId` y `dataScope` con `createdby`, `jlb_responsableid` y `jlb_centrotrabajoid`. No redefine `AccessContext` ni edita `autenticacion-roles`.
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 4.4_

- [ ] 3.2 Implementar la asignación de responsable y la secuencia obligatoria de estados
  - Definir las únicas transiciones válidas `Nueva → En revisión → Asignada → En progreso → Resuelta → Cerrada` sin saltos ni retrocesos.
  - Permitir asignar o reasignar responsable solo a `Supervisor` y `Administrador`, registrando `FechaAsignacion` y exigiendo responsable para entrar en `Asignada`.
  - Registrar `FechaResolucion` al entrar en `Resuelta` y conservar todos los hitos al pasar a `Cerrada`.
  - Al finalizar, cualquier cambio inválido de estado o de responsable es rechazado y cualquier cambio válido deja la incidencia actualizada con sus fechas correspondientes.
  - _Boundary:_ Posee el option set `src\OptionSets\jlb_estadoincidencia.xml`, las fórmulas o reglas de mutación en `.msapp` y cualquier ajuste puntual de metadatos de `jlb_incidencia` ligado al lifecycle. No cambia contratos de búsqueda ni de notificaciones salvo los campos ya publicados por este spec.
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 6.2, 7.2, 7.3_

- [ ] 3.3 Integrar guards y acciones visibles en detalle según rol y estado
  - Mostrar solo las acciones de asignar, avanzar estado o cerrar cuando el rol y el estado actual las permitan.
  - Mantener mensajes explícitos de denegación para usuarios sin permiso en lugar de fallos silenciosos o navegación ambigua.
  - Forzar una recarga del registro al terminar cada acción para evitar que la interfaz muestre estados obsoletos.
  - Al finalizar, el detalle operativo actúa como única entrada segura para gestionar el lifecycle sin exponer acciones indebidas.
  - _Boundary:_ Posee botones, visibilidad, guards y mensajes del detalle dentro de `.msapp`; consume la política de alcance y el lifecycle definidos por este spec. No edita superficies downstream salvo el contrato público del detalle ya acordado.
  - _Requirements: 2.2, 2.4, 3.6, 4.3, 5.2, 5.3, 5.6, 6.3, 7.3, 7.4_

- [ ] 4. Publicar las superficies de integración y preservación histórica
  - _Boundary:_ Coordina únicamente los seams públicos de `incidencias-core` y la conservación histórica sobre `jlb_incidencia`. No implementa consumidores downstream; solo deja contratos, proyecciones y metadatos listos para que otros specs los reutilicen.
- [ ] 4.1 Consolidar el contrato estable para specs downstream
  - Asegurar que `IdIncidencia`, `Estado`, `Prioridad`, `TipoIncidencia`, `Responsable`, `CentroTrabajo`, `FechaCreacion`, `FechaAsignacion` y `FechaResolucion` quedan disponibles y con semántica estable.
  - Documentar en la solución y en las consultas de la app qué columnas constituyen la superficie mínima que usarán `incidencias-colaboracion`, `busqueda-dashboard` y `notificaciones`.
  - Al finalizar, los consumidores downstream pueden integrarse con la incidencia sin pedir cambios adicionales a este boundary.
  - _Boundary:_ Posee la publicación del snapshot downstream de `jlb_incidencia`, incluyendo `creadorSystemUserId` derivado de `createdby` y el contrato de navegación del detalle `manage`/`read-only`. Si requiere tocar `.msapp`, `.meta.xml`, `Solution.xml` o `Customizations.xml`, solo puede hacerlo para exponer esos seams sin modificar lógica propia de consumidores.
  - _Requirements: 6.1, 6.2, 6.4, 7.4_

- [ ] 4.2 Preservar la trazabilidad y el comportamiento histórico de relaciones clave
  - Verificar que tipos desactivados, responsables reasignados y cambios de estado mantienen referencias históricas consultables.
  - Ajustar auditoría y reglas de relación para que una incidencia cerrada siga mostrando responsable y fechas ya registradas.
  - Al finalizar, la incidencia conserva sus hitos críticos y sus referencias históricas aunque cambie la configuración operativa posterior.
  - _Boundary:_ Posee auditoría, comportamiento de relaciones y persistencia histórica dentro de `src\Entities\jlb_incidencia\**` y `src\Entities\jlb_tipoincidencia\**`. No altera reporting, dashboards ni registros de notificación.
  - _Requirements: 3.4, 3.5, 4.5, 5.5, 6.1, 6.2, 6.3_

- [ ] 5. Validar el comportamiento completo del feature
  - _Boundary:_ Coordina solo la evidencia de validación de `incidencias-core` sobre sus tablas, pantallas y contratos publicados. Los artefactos host compartidos se inspeccionan o ejercitan, pero no se amplían fuera de lo necesario para probar este spec.
- [ ] 5.1 Verificar altas y catálogo con pruebas funcionales dirigidas
  - Probar la carga de tipos por defecto, la administración del catálogo por parte de un administrador y el bloqueo a usuarios no autorizados.
  - Probar altas válidas e inválidas comprobando obligatoriedad de campos, selección solo de tipos activos y creación en estado `Nueva`.
  - Al finalizar, existe evidencia reproducible de que el catálogo y el alta cubren los escenarios principales y sus errores visibles.
  - _Boundary:_ Posee casos de prueba funcionales y evidencia sobre `jlb_tipoincidencia`, formulario de alta y guards de administración. No reclama cobertura de dashboards, comentarios ni notificaciones.
  - _Requirements: 1.1, 1.2, 1.4, 1.5, 3.1, 3.2, 3.3, 3.5, 3.6, 7.3_

- [ ] 5.2 Verificar alcance, asignación y lifecycle con pruebas de integración
  - Probar acceso de operario, supervisor y administrador sobre incidencias propias, asignadas, de centro y fuera de alcance.
  - Probar asignación, reasignación, secuencia completa de estados, rechazo de saltos y preservación de fechas al resolver y cerrar.
  - Al finalizar, todas las reglas críticas de visibilidad y transición quedan validadas con escenarios positivos y negativos.
  - _Boundary:_ Posee la evidencia de integración sobre `AccessContext` canónico, `createdby`, `jlb_responsableid`, `jlb_centrotrabajoid` y el option set de estados. No ejecuta validaciones funcionales de specs consumidores más allá de comprobar compatibilidad del seam publicado.
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 4.1, 4.2, 4.3, 4.4, 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 6.2_

- [ ] 5.3 Verificar consistencia cross-device, rendimiento y seams downstream
  - Validar en navegador, Android, iPhone y tablet que alta, detalle y acciones autorizadas mantienen el mismo comportamiento observable.
  - Medir que la apertura de detalle y la confirmación de altas o transiciones habituales se reflejan en 3 segundos o menos.
  - Confirmar que los datos mínimos consumibles por specs downstream quedan disponibles sin exponer comentarios, adjuntos, dashboards o notificaciones en este alcance.
  - Al finalizar, el feature cuenta con evidencia de compatibilidad, rendimiento y preparación de integración para los siguientes specs.
  - _Boundary:_ Posee la validación del seam `IncidentIntegrationSnapshot`, de `creadorSystemUserId` y del contrato de navegación del detalle en modo `read-only`/`manage`, además de la UX propia de `incidencias-core` en dispositivos soportados. No implementa ni certifica la lógica interna de los consumidores downstream.
  - _Requirements: 6.4, 7.1, 7.2, 7.4_
