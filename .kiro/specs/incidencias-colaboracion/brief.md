# Brief: incidencias-colaboracion

## Problem
Una vez creada una incidencia, los distintos implicados (operario, supervisor, administrador) necesitan añadir contexto adicional a lo largo del tiempo —comentarios de seguimiento y evidencias visuales o documentales (fotos, PDFs)— sin perder trazabilidad ni permitir que ese historial se altere después de escrito.

## Current State
Hoy ese contexto se comparte por correo o llamadas y se pierde o queda disperso. No existe en Dataverse ninguna tabla ni UI para comentarios ni adjuntos ligados a una incidencia.

## Desired Outcome
Cualquier usuario con acceso a una incidencia puede añadir un comentario, que queda registrado con usuario, fecha y hora en el historial, y ese historial no puede modificarse ni borrarse. Cualquier usuario con acceso puede adjuntar imágenes o documentos PDF a una incidencia; esos adjuntos se almacenan en Dataverse, pueden visualizarse (imágenes) y descargarse desde la aplicación.

## Approach
Modelar en Dataverse las tablas `Comentarios` (IdComentario, IdIncidencia, Comentario, Usuario, Fecha) y `Adjuntos` (IdAdjunto, IdIncidencia, NombreArchivo, RutaArchivo), ambas relacionadas 1-N con `Incidencias` (definida en incidencias-core). En la canvas app, añadir en la pantalla de detalle de incidencia una sección de historial de comentarios (solo alta, sin edición/borrado desde la UI ni permisos de escritura posteriores) y una sección de adjuntos con carga desde cámara/galería/explorador de archivos, previsualización de imágenes y descarga de PDFs. La visibilidad de estas secciones respeta el mismo control de acceso que la incidencia padre (heredado de autenticacion-roles e incidencias-core).

## Scope
- **In**: Tablas Dataverse `Comentarios` y `Adjuntos` relacionadas con `Incidencias`; UI para añadir comentario (con usuario/fecha automáticos) y listar historial de forma inmutable; UI para subir, previsualizar (imágenes) y descargar (imágenes/PDF) adjuntos; validación de tipo de archivo (imágenes y PDF).
- **Out**: Definición de la tabla Incidencias y su ciclo de vida (incidencias-core); búsqueda/filtrado de incidencias por cualquier criterio (busqueda-dashboard); notificaciones automáticas al añadir comentario o adjunto (no está en los RF de notificaciones explícitamente, pero cualquier evento de notificación adicional debe evaluarse en el spec de notificaciones, no aquí); edición o borrado de comentarios/adjuntos ya creados (explícitamente inmutable, fuera de alcance por diseño).

## Boundary Candidates
- Entidad y UI de Comentarios (solo alta + lectura, inmutable).
- Entidad y UI de Adjuntos (subida, previsualización, descarga).

## Out of Boundary
- Creación/edición/cierre de la incidencia en sí (incidencias-core).
- Agregación de comentarios/adjuntos en dashboards o reportes (busqueda-dashboard, si se requiere en el futuro).
- Notificaciones sobre nuevos comentarios o adjuntos (no contemplado en el alcance actual de notificaciones; evaluar como extensión posterior si se solicita).

## Upstream / Downstream
- **Upstream**: incidencias-core (una incidencia debe existir antes de poder comentarla o adjuntar archivos), autenticacion-roles (para saber quién puede ver/comentar según su acceso a la incidencia padre).
- **Downstream**: Ninguno identificado en el alcance actual; busqueda-dashboard podría en el futuro mostrar conteos de comentarios/adjuntos, pero no es requisito v1.

## Existing Spec Touchpoints
- **Extends**: incidencias-core (añade sub-entidades relacionadas a la incidencia).
- **Adjacent**: autenticacion-roles (control de acceso heredado).

## Constraints
- Almacenamiento de datos y adjuntos exclusivamente en Dataverse.
- El historial de comentarios es inmutable: no editable ni borrable desde la aplicación.
- Adjuntos soportados: imágenes y documentos PDF.
- Debe funcionar en navegador, Android, iPhone y tablet.
