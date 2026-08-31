# Requirements Document

## Introduction
Los operarios, supervisores y administradores de Logística Sur S.L. necesitan colaborar sobre incidencias ya creadas dentro de la aplicación corporativa sin sacar el seguimiento a canales paralelos. Actualmente el contexto adicional se comparte por correo o llamadas y no existe un historial único de comentarios ni un repositorio operativo de evidencias ligado a cada incidencia. Esta funcionalidad debe permitir añadir comentarios inmutables y adjuntar imágenes o documentos PDF con consulta, previsualización de imágenes y descarga, respetando el mismo alcance de acceso que la incidencia padre y sin duplicar las reglas del ciclo de vida definidas en `incidencias-core`.

## Boundary Context
- **In scope**: alta y consulta de comentarios inmutables sobre incidencias existentes; alta, consulta, previsualización de imágenes y descarga de adjuntos imagen/PDF; herencia del control de acceso de la incidencia padre; continuidad del vínculo entre colaboración e incidencia.
- **Out of scope**: creación, asignación, cambios de estado o cierre de la incidencia; búsqueda, filtrado o dashboard; notificaciones automáticas; edición o borrado de comentarios y adjuntos ya registrados.
- **Adjacent expectations**: esta funcionalidad consume la visibilidad por rol y centro de trabajo definida en `autenticacion-roles` y la referencia estable de `Incidencias` definida en `incidencias-core`, incluyendo `IdIncidencia`, `Responsable` y `CentroTrabajo`, sin redefinir esos contratos.

## Requirements

### Requirement 1: Registro de comentarios inmutables
**Objective:** Como usuario con acceso a una incidencia, quiero añadir comentarios de seguimiento, para dejar constancia cronológica del contexto operativo sin alterar el historial previo.

#### Acceptance Criteria
1.1 When el usuario con acceso abre el detalle de una incidencia, the Aplicación de Incidencias shall ofrecer un área para redactar un nuevo comentario asociado a esa incidencia.

1.2 When el usuario confirma un comentario válido, the Aplicación de Incidencias shall registrarlo con un identificador único, el usuario autor y la fecha y hora del registro.

1.3 If el comentario está vacío o solo contiene espacios, the Aplicación de Incidencias shall impedir su registro y mostrar el motivo de la corrección.

1.4 When se registra un comentario, the Aplicación de Incidencias shall incorporarlo al historial visible de la incidencia sin sobrescribir comentarios previos.

1.5 The Aplicación de Incidencias shall no ofrecer acciones de editar o eliminar comentarios ya registrados.

### Requirement 2: Consulta del historial colaborativo y acceso heredado
**Objective:** Como usuario operativo, quiero ver y usar la colaboración solo dentro de las incidencias a las que ya tengo acceso, para no exponer información fuera de mi alcance.

#### Acceptance Criteria
2.1 When un usuario autorizado abre una incidencia, the Aplicación de Incidencias shall mostrar los comentarios y adjuntos vinculados a esa incidencia dentro del mismo detalle.

2.2 While el usuario mantenga acceso válido a la incidencia padre según su rol y centro de trabajo, the Aplicación de Incidencias shall permitirle consultar y añadir comentarios y adjuntos dentro de ese mismo alcance.

2.3 If un usuario intenta acceder al contenido colaborativo de una incidencia fuera de su alcance, the Aplicación de Incidencias shall denegar el acceso sin mostrar comentarios, adjuntos ni sus metadatos.

2.4 The Aplicación de Incidencias shall mostrar en cada comentario el autor y la fecha y hora registradas.

2.5 While una incidencia no tenga comentarios o adjuntos, the Aplicación de Incidencias shall mostrar un estado vacío claro en cada sección.

### Requirement 3: Registro de adjuntos admitidos
**Objective:** Como usuario con acceso a una incidencia, quiero adjuntar evidencias visuales o documentales admitidas, para enriquecer el seguimiento sin salir de la aplicación.

#### Acceptance Criteria
3.1 When un usuario autorizado añade un archivo a una incidencia, the Aplicación de Incidencias shall aceptar únicamente imágenes o documentos PDF.

3.2 When se confirma un adjunto válido, the Aplicación de Incidencias shall registrarlo con un identificador único, el nombre del archivo y la referencia a la incidencia asociada.

3.3 If el archivo no corresponde a un formato de imagen admitido ni a PDF, the Aplicación de Incidencias shall rechazar la carga e indicar el formato permitido.

3.4 If la carga del adjunto no puede completarse, the Aplicación de Incidencias shall informar del fallo sin dejar una confirmación ambigua.

3.5 While el dispositivo o cliente ofrezca varias fuentes de selección compatibles, the Aplicación de Incidencias shall permitir al usuario elegir la fuente disponible para incorporar el adjunto.

### Requirement 4: Visualización y descarga de adjuntos
**Objective:** Como usuario operativo, quiero consultar y recuperar los archivos ligados a una incidencia, para utilizar la evidencia sin alterar su contenido.

#### Acceptance Criteria
4.1 When un usuario consulta la sección de adjuntos de una incidencia, the Aplicación de Incidencias shall mostrar el nombre de cada archivo registrado.

4.2 When el usuario abre un adjunto de imagen, the Aplicación de Incidencias shall mostrar una previsualización dentro de la aplicación.

4.3 When el usuario solicita un adjunto de imagen o PDF, the Aplicación de Incidencias shall permitir su descarga.

4.4 If una imagen no puede previsualizarse, the Aplicación de Incidencias shall ofrecer una alternativa de descarga del archivo sin perder el acceso al resto de adjuntos.

4.5 The Aplicación de Incidencias shall no ofrecer acciones de sustituir o eliminar adjuntos ya registrados.

### Requirement 5: Continuidad operativa y límites del alcance
**Objective:** Como responsable del proceso, quiero que la colaboración se mantenga ligada a la incidencia correcta y opere de forma consistente en los clientes soportados, para preservar trazabilidad sin ampliar el alcance funcional.

#### Acceptance Criteria
5.1 Where una incidencia cambie de responsable o de estado dentro de su ciclo de vida, the Aplicación de Incidencias shall conservar accesibles sus comentarios y adjuntos existentes bajo el mismo identificador de incidencia.

5.2 The Aplicación de Incidencias shall mantener fuera de este alcance el ciclo de vida de la incidencia, la búsqueda o filtrado, los paneles agregados y las notificaciones automáticas.

5.3 When el usuario abre el detalle de una incidencia o confirma un comentario o adjunto válido en condiciones habituales, the Aplicación de Incidencias shall reflejar el resultado en 3 segundos o menos.

5.4 If la incidencia de referencia deja de estar disponible para el usuario o deja de existir antes de registrar un comentario o adjunto, the Aplicación de Incidencias shall cancelar la operación e informar que la incidencia ya no está disponible.

5.5 When el usuario utiliza la funcionalidad desde navegador, Android, iPhone o tablet, the Aplicación de Incidencias shall ofrecer el mismo flujo base de consultar, comentar, adjuntar, previsualizar imágenes y descargar archivos según permisos.
