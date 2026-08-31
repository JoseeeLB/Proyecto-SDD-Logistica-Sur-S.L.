# Implementation Plan

- [ ] 1. Preparar la base de automatización y configuración de notificaciones
  - _Boundary:_ Base del spec `notificaciones`. Puede registrar componentes propios en `src\Other\Solution.xml` y `src\Other\Customizations.xml` sin reordenar ni reescribir entradas de otros specs; cualquier toque en `src\CanvasApps\jlb_logsticasur_95873_DocumentUri.msapp` o `src\CanvasApps\jlb_logsticasur_95873.meta.xml` queda limitado a recepción pasiva de push y debe coordinarse para no mezclar cambios funcionales ajenos.
- [ ] 1.1 Registrar la infraestructura de solución para los flujos y sus parámetros
  - Incorporar los componentes de automatización, referencias de conexión y variables de entorno necesarias para que la solución pueda transportar notificaciones entre entornos.
  - Dejar preparada la activación de los canales de correo y push sin valores hardcodeados dependientes de un entorno concreto.
  - Al finalizar, la solución expone todos los componentes previos necesarios para importar, configurar y activar los flujos de notificación.
  - _Requirements: 3.1, 3.2, 3.5_
  - _Boundary:_ Alta y cableado de componentes propios de automatización. `Solution.xml` y `Customizations.xml` se editan solo para añadir referencias de flujos, connection references y environment variables de `notificaciones`; no se tocan definiciones funcionales de otros specs ni la `.msapp` salvo metadatos pasivos estrictamente necesarios.

- [ ] 1.2 Definir la configuración funcional central por evento
  - Crear la configuración autoritativa para `CREACION`, `ASIGNACION`, `CAMBIO_ESTADO` y `CIERRE`, incluyendo activación, canales permitidos, categorías de destinatario y plantillas simples.
  - Restringir la configuración a las categorías creador, responsable asignado y supervisor del centro, evitando combinaciones fuera del alcance del spec.
  - Al finalizar, existen cuatro configuraciones mantenibles y cualquier cambio en ellas afecta solo a notificaciones futuras del evento correspondiente.
  - _Requirements: 2.1, 2.2, 3.1, 3.2, 3.3, 3.4, 3.5, 4.1, 4.2, 4.3, 4.4, 4.5_
  - _Boundary:_ Esquema y semillas de `jlb_notificacionevento` exclusivos de `notificaciones`. Puede añadir tablas/columnas/filas semilla del feature y sus descriptores en artefactos de solución compartidos, pero no redefine contratos de `incidencias-core` ni `autenticacion-roles`.

- [ ] 1.3 Definir la trazabilidad operativa de envíos y fallos
  - Crear el registro de envíos por incidencia, evento, canal y destinatario con correlación, intentos, resultado final y resumen de error cuando aplique.
  - Asegurar que los fallos y omisiones queden auditables sin modificar los datos de negocio de la incidencia.
  - Al finalizar, soporte operativo puede identificar qué avisos se intentaron, cuáles se omitieron y cuáles agotaron reintentos.
  - _Requirements: 2.3, 5.2, 5.4, 5.5_
  - _Boundary:_ Esquema y persistencia de `jlb_notificacionenvio` exclusivas del spec. Las ediciones en `Customizations.xml`/`Solution.xml` se limitan al alta de esta tabla y sus componentes; no se amplían tablas de negocio upstream ni se añade lógica dentro de la canvas app.

- [ ] 2. Implementar el despacho compartido de notificaciones
  - _Boundary:_ Flujo hijo compartido y lógica interna de despacho de `notificaciones`. Puede crear/actualizar flujos propios y dependencias en artefactos de solución compartidos, pero no modifica estados, ownership ni reglas de negocio de `jlb_incidencia`.
- [ ] 2.1 Construir la carga de configuración y el contrato común de despacho
  - Implementar la entrada común que reciben todos los eventos de notificación con la incidencia confirmada, el tipo de evento y la correlación del intento.
  - Cargar la configuración vigente del evento y detener de forma controlada los despachos sin configuración válida o sin canales activos.
  - Al finalizar, cualquier flujo de evento puede delegar en un despacho común con reglas consistentes de configuración y salida controlada.
  - _Requirements: 1.5, 4.1, 4.2, 4.3, 4.5, 5.4, 6.1, 6.3_
  - _Boundary:_ Contrato `DispatchRequest` y lectura de configuración propios del flujo hijo. Los artefactos compartidos solo reflejan la definición del flujo y sus dependencias; no se introducen contratos alternativos a los ya publicados por specs upstream.

- [ ] 2.2 Resolver destinatarios válidos y deduplicarlos por canal
  - Resolver destinatarios solo entre creador, responsable asignado y supervisores activos del centro de trabajo asociado a la incidencia.
  - Omitir selectivamente las categorías irresolubles, conservar constancia del problema y consolidar cada persona para que no reciba duplicados en el mismo canal.
  - Al finalizar, el despacho produce una lista única y válida de destinatarios por evento y canal dentro del perímetro autorizado.
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 6.4_
  - _Boundary:_ Consumo verbatim de `creadorSystemUserId` desde `incidencias-core` y de `AccessContext` desde `autenticacion-roles` para resolver creador, responsable y supervisores. Puede consultar contratos upstream, pero no renombra campos ni replica lógica de perfil/centro fuera del flujo.

- [ ] 2.3 Enviar correo y push con resiliencia básica
  - Construir los mensajes con identificador de incidencia, tipo de evento, estado actual y responsable cuando aplique.
  - Ejecutar correo y push como entregas separadas para que el fallo de un canal no impida completar el resto de envíos válidos.
  - Aplicar reintentos básicos antes de marcar una entrega como fallida.
  - Al finalizar, el despacho puede completar envíos parciales, registrar el resultado por destinatario y dejar evidencia del agotamiento de reintentos cuando ocurra.
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 5.1, 5.2, 5.3, 5.5_
  - _Boundary:_ Uso exclusivo de connection references y environment variables del spec para Outlook y push. Puede ajustar componentes propios de flujo en `Solution.xml`/`Customizations.xml`, pero no modifica la lógica funcional de la `.msapp` más allá de una recepción pasiva previamente acordada.

- [ ] 3. Implementar los flujos de evento sobre incidencias
  - _Boundary:_ Familia de flujos de borde del spec. Puede añadir workflows solution-aware propios y sus referencias en artefactos compartidos, manteniendo separadas las ediciones de `Solution.xml` y `Customizations.xml` respecto a otros specs y sin tocar contratos upstream.
- [ ] 3.1 (P) Implementar la notificación de creación de incidencias
  - Detectar altas confirmadas de incidencias y disparar únicamente el evento de creación con los datos persistidos tras el guardado.
  - Evitar que modificaciones posteriores reutilicen esta ruta de notificación.
  - Al finalizar, cada incidencia nueva puede originar exactamente su despacho de creación sin intervención manual.
  - _Requirements: 1.1, 1.5, 5.4, 6.1, 6.3_
  - _Boundary:_ Flujo de Creación de Incidencia. Edita solo el workflow de creación y sus referencias solution-aware en `Solution.xml`/`Customizations.xml`; no altera la `.msapp`, ni redefine el alta de `incidencias-core`.
  - _Depends: 2.1, 2.3_

- [ ] 3.2 (P) Implementar la notificación de asignación y reasignación
  - Detectar asignaciones válidas a partir del hito confirmado de asignación y leer el responsable vigente antes de despachar.
  - Ignorar cambios donde no exista responsable resoluble o donde el cambio no represente una asignación efectiva.
  - Al finalizar, cada asignación o reasignación válida genera un aviso específico sin depender de detectar cambios directos del lookup de responsable.
  - _Requirements: 1.2, 1.5, 3.4, 5.4, 6.1, 6.3_
  - _Boundary:_ Flujo de Asignación de Incidencia. Edita solo el workflow de asignación y sus referencias solution-aware en `Solution.xml`/`Customizations.xml`; no altera reglas de asignación de `incidencias-core` ni componentes de otros specs.
  - _Depends: 2.1, 2.2, 2.3_

- [ ] 3.3 (P) Implementar la notificación de cambio de estado
  - Detectar únicamente cambios confirmados de estado que correspondan a `En revisión`, `En progreso` o `Resuelta`.
  - Excluir `Asignada` y `Cerrada` para evitar solape con los eventos específicos de asignación y cierre.
  - Al finalizar, los cambios de estado cubiertos generan un aviso propio y no duplican otros eventos del spec.
  - _Requirements: 1.3, 1.5, 5.4, 6.1, 6.2, 6.3_
  - _Boundary:_ Flujo de Cambio de Estado de Incidencia. Edita solo el workflow de cambio de estado y sus referencias solution-aware en `Solution.xml`/`Customizations.xml`; no redefine la máquina de estados de `incidencias-core`.
  - _Depends: 2.1, 2.3_

- [ ] 3.4 (P) Implementar la notificación de cierre
  - Detectar el paso confirmado a `Cerrada` y construir el aviso usando el estado final, el responsable vigente y la fecha de resolución ya persistida.
  - Garantizar que el cierre no genere además una notificación genérica de cambio de estado.
  - Al finalizar, cada cierre válido produce un aviso final único y coherente con los hitos confirmados de la incidencia.
  - _Requirements: 1.4, 1.5, 3.4, 5.4, 6.1, 6.2, 6.3_
  - _Boundary:_ Flujo de Cierre de Incidencia. Edita solo el workflow de cierre y sus referencias solution-aware en `Solution.xml`/`Customizations.xml`; no cambia la lógica de resolución/cierre de `incidencias-core`.
  - _Depends: 2.1, 2.3_

- [ ] 4. Integrar la administración y el ciclo ALM de la solución
  - _Boundary:_ Administración operativa y ALM del spec. Puede ajustar empaquetado propio en artefactos compartidos y, solo si es imprescindible para push in-app, tocar `src\CanvasApps\jlb_logsticasur_95873_DocumentUri.msapp` o `src\CanvasApps\jlb_logsticasur_95873.meta.xml` de forma pasiva y coordinada.
- [ ] 4.1 Habilitar el mantenimiento operativo de la configuración por evento
  - Publicar una experiencia administrativa mínima para revisar el estado de cada evento, sus canales habilitados y sus categorías de destinatario.
  - Garantizar que los cambios de activación, canales y mensajes se reflejan de forma consistente en ejecuciones futuras sin redefinir los flujos.
  - Al finalizar, administración puede ajustar la configuración de notificaciones de forma central y controlada.
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_
  - _Boundary:_ Administración de `jlb_notificacionevento` y visibilidad operativa del feature. Si requiere exponer recepción push en la app, la `.msapp` y la `.meta.xml` solo pueden tocarse para enlaces pasivos o dependencias del feature, sin mezclar cambios UI ajenos.

- [ ] 4.2 Conectar los flujos con los contratos existentes del sistema
  - Integrar los flujos con el esquema exacto de `Incidencias` y con el modelo de perfil, rol y centro ya resueltos por las especificaciones upstream.
  - Verificar que ninguna automatización escribe lógica de negocio de vuelta sobre la incidencia ni amplía el alcance funcional hacia comentarios, adjuntos u otros ámbitos.
  - Al finalizar, el feature consume contratos upstream estables y mantiene separadas las responsabilidades entre núcleo y notificaciones.
  - _Requirements: 2.5, 5.4, 6.1, 6.2, 6.3, 6.4_
  - _Boundary:_ Integración estricta con `IncidentIntegrationSnapshot` y `AccessContext` canónicos. Puede adaptar mapping interno del spec, pero no modifica ni duplica contratos upstream ni edita artefactos de otros specs fuera del registro solution-aware necesario.

- [ ] 4.3 Validar el empaquetado e importación de la solución con flujos activos
  - Asegurar que los componentes de automatización, sus conexiones y parámetros quedan transportables dentro del modelo actual de solución del repositorio.
  - Comprobar que, tras importar en un entorno compatible, los flujos pueden asociar sus referencias y quedar listos para activación.
  - Al finalizar, existe evidencia de que la solución conserva correctamente el feature de notificaciones en el proceso de ALM.
  - _Requirements: 3.1, 3.2, 3.5, 4.2, 6.4_
  - _Boundary:_ Validación de empaquetado sobre `src\Other\Solution.xml`, `src\Other\Customizations.xml`, componentes de `src\Workflows\` y cualquier referencia pasiva en `src\CanvasApps\jlb_logsticasur_95873_DocumentUri.msapp` / `src\CanvasApps\jlb_logsticasur_95873.meta.xml`. La coordinación con otros specs consiste en añadir o actualizar solo nodos/componentes del feature sin reformatear ni mover artefactos ajenos.

- [ ] 5. Validar el comportamiento completo de notificaciones
  - _Boundary:_ Evidencia y pruebas del spec `notificaciones`. Puede ejecutar escenarios sobre contratos upstream y artefactos propios, pero no incluye corregir defects fuera del perímetro del spec salvo documentar incompatibilidades detectadas.
- [ ] 5.1 Verificar cobertura funcional de los cuatro eventos
  - Probar creación, asignación, cambio de estado y cierre comprobando que cada uno emite solo su notificación esperada.
  - Confirmar que cambios fuera de alcance no generan avisos y que los datos del mensaje reflejan el estado confirmado de la incidencia.
  - Al finalizar, existe evidencia reproducible de que los cuatro eventos obligatorios quedan cubiertos sin solapes.
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 3.3, 3.4, 6.1, 6.2, 6.3_
  - _Boundary:_ Validación funcional de los cuatro flujos de borde y del flujo hijo, sin extender el alcance a comentarios, adjuntos o dashboards. La evidencia puede referenciar artefactos solution-aware compartidos, pero no requiere editarlos salvo para corregir wiring propio del spec.

- [ ] 5.2 Verificar destinatarios, deduplicación y continuidad ante fallos
  - Probar combinaciones con creador, responsable y varios supervisores del centro para validar resolución, consolidación y omisión selectiva de destinatarios irresolubles.
  - Forzar fallos de un canal o destinatario y comprobar que continúan el resto de entregas válidas con el resultado final correctamente registrado.
  - Al finalizar, la entrega demuestra comportamiento correcto ante superposición de destinatarios y errores parciales de envío.
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 5.1, 5.2, 5.3, 5.5, 6.4_
  - _Boundary:_ Validación de resolución usando `creadorSystemUserId` y `AccessContext` canónicos, incluyendo supervisores por `centroTrabajoId` y `role`. No se alteran datos upstream salvo los mínimos escenarios de prueba previstos por sus contratos.

- [ ] 5.3 Verificar configuración, canales permitidos y límites de alcance
  - Probar cambios de configuración por evento para confirmar que afectan solo a futuras ejecuciones y que no habilitan destinatarios ni canales fuera del alcance de v1.
  - Validar que el correo y la push in-app siguen siendo los únicos canales efectivos y que los errores nunca alteran el ciclo de vida de la incidencia.
  - Al finalizar, queda validado que la administración, los límites funcionales y la separación respecto al núcleo de incidencias se conservan en producción.
  - _Requirements: 3.1, 3.2, 3.5, 4.1, 4.2, 4.3, 4.4, 4.5, 5.4, 6.2, 6.3_
  - _Boundary:_ Validación de límites funcionales y de empaquetado del feature, incluyendo que cualquier referencia en `Solution.xml`, `Customizations.xml`, `.msapp` o `.meta.xml` siga siendo pasiva, propia del spec y coordinada con cambios vecinos.
