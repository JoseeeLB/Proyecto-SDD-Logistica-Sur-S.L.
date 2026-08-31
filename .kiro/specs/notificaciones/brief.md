# Brief: notificaciones

## Problem
Cuando una incidencia se crea, se asigna, cambia de estado o se cierra, los interesados (operario creador, responsable asignado, supervisor del centro) necesitan enterarse sin tener que estar consultando la app constantemente, para poder actuar a tiempo y reducir los retrasos operativos.

## Current State
Hoy el aviso se hace manualmente por teléfono o correo, de forma inconsistente y sin garantía de que llegue al destinatario correcto ni a tiempo.

## Desired Outcome
Cuando ocurre un evento de creación, asignación, cambio de estado o cierre de una incidencia, se envía automáticamente una notificación por email (Outlook) y/o notificación push de Power Apps a los destinatarios configurados para ese evento (p. ej., responsable asignado al ser asignada la incidencia).

## Approach
Implementar flujos de Power Automate disparados por cambios en la tabla `Incidencias` de Dataverse (triggers "cuando se crea o modifica un registro", filtrados por el campo Estado o por la creación de un registro). Cada flujo determina el/los destinatarios según el evento (creador, responsable asignado, supervisor del centro de trabajo) y envía el email vía el conector de Outlook y/o una notificación push mediante la acción nativa de Power Apps ("Power Apps Notification" / experiencia de notificaciones dentro de la app). Los destinatarios y plantillas de mensaje se configuran de forma centralizada para facilitar mantenimiento.

## Scope
- **In**: Flujos de Power Automate para los 4 eventos (creación, asignación, cambio de estado, cierre); envío de email vía Outlook; envío de notificación push de Power Apps; resolución de destinatarios por evento (creador, responsable, supervisor de centro); manejo básico de errores de envío (reintentos/registro de fallos).
- **Out**: Definición del modelo de datos y ciclo de vida de Incidencias (incidencias-core, este spec solo reacciona a sus eventos); notificaciones push nativas fuera de la app de Power Apps (explícitamente fuera de alcance v1); notificaciones sobre comentarios o adjuntos (no solicitado); configuración de plantillas avanzada tipo motor de reglas (se cubre con configuración simple, no un sistema de reglas configurable por el usuario final salvo que se pida ampliar).

## Boundary Candidates
- Disparadores de Power Automate sobre eventos de Incidencias.
- Resolución de destinatarios por rol/relación con la incidencia.
- Envío de email (Outlook) vs. push (Power Apps) como dos canales independientes pero coordinados.

## Out of Boundary
- Cualquier lógica de negocio sobre el ciclo de vida de la incidencia en sí (incidencias-core es dueño de esa lógica; este spec solo consume el evento ya validado).
- UI dentro de la canvas app más allá de recibir/mostrar la notificación push (la pantalla de incidencia en sí es de incidencias-core).
- Comentarios/adjuntos (incidencias-colaboracion).

## Upstream / Downstream
- **Upstream**: incidencias-core (fuente de los eventos de creación/asignación/cambio de estado/cierre y de los datos de responsable/centro de trabajo), autenticacion-roles (para resolver supervisor/centro de trabajo como destinatarios).
- **Downstream**: Ninguno; es un consumidor terminal de eventos.

## Existing Spec Touchpoints
- **Extends**: incidencias-core (reacciona a sus eventos de ciclo de vida).
- **Adjacent**: autenticacion-roles (resolución de destinatarios por rol/centro).

## Constraints
- Canales soportados en v1: email Outlook y push de Power Apps (no push nativo fuera de la app).
- Debe cubrir los 4 eventos explícitos: creación, asignación, cambio de estado, cierre.
- Implementado con Power Automate como capa de integración, sin lógica de negocio duplicada del ciclo de vida.
