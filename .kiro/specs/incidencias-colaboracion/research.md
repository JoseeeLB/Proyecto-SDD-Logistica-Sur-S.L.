## Summary
- **Feature**: `incidencias-colaboracion`
- **Discovery Scope**: Extension
- **Key Findings**:
  - El repositorio actual sigue el patrón de solución Power Platform con una única canvas app empaquetada y artefactos base de solución; la nueva colaboración debe entrar como extensión del detalle de incidencia ya previsto por `incidencias-core`.
  - `incidencias-core` ya fijó `IdIncidencia`, `Responsable`, `CentroTrabajo` y la política de alcance sobre la incidencia padre; este spec debe heredar ese contrato para comentarios y adjuntos sin redefinir permisos.
  - La documentación oficial de Power Apps y Dataverse favorece usar capacidades nativas de archivo, pero el control PDF Viewer solo admite URLs HTTPS anónimas; por ello el diseño debe limitar la experiencia PDF a descarga/apertura externa y reservar la previsualización embebida para imágenes.

## Research Log

### Arquitectura actual del repositorio
- **Context**: Era necesario comprobar cómo encaja la colaboración en la solución existente y qué artefactos físicos pueden cambiarse.
- **Sources Consulted**:
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Solution.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\Other\Customizations.xml`
  - `C:\Users\joslopez\LogsticaSurSL\src\CanvasApps\jlb_logsticasur_95873.meta.xml`
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\incidencias-core\design.md`
- **Findings**:
  - La solución publicada solo contiene la app `jlb_logsticasur_95873` y todavía no exporta tablas funcionales en `Customizations.xml`.
  - El punto natural de ampliación es la pantalla de detalle de incidencia ya reservada por `incidencias-core` para capacidades colaborativas futuras.
  - La convención de prefijo `jlb` y el patrón de tablas Dataverse específicas por dominio ya están asentados en los specs previos.
- **Implications**:
  - El diseño debe proponer nuevas carpetas de entidad y cambios controlados en la msapp y sus metadatos.
  - No hace falta introducir otro runtime ni otra aplicación host.

### Contratos upstream de seguridad y dominio
- **Context**: Había que asegurar que comentarios y adjuntos heredasen el mismo perímetro que la incidencia padre.
- **Sources Consulted**:
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\autenticacion-roles\design.md`
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\incidencias-core\requirements.md`
  - `C:\Users\joslopez\LogsticaSurSL\.kiro\specs\incidencias-core\design.md`
- **Findings**:
  - `autenticacion-roles` define `jlb_perfilusuario`, `jlb_centrotrabajo`, `jlb_rolnegocio` y `AccessContext` como fuente autoritativa.
  - `incidencias-core` fija `IdIncidencia` como identificador visible y `jlb_incidencia` como raíz de agregación con `Responsable` y `CentroTrabajo`.
  - La política de alcance de incidencias ya distingue correctamente `self`, `center` y `global` y debe reutilizarse tal cual.
- **Implications**:
  - La colaboración puede depender de `jlb_incidencia` y `jlb_perfilusuario`, pero no debe replicar atributos de rol ni centro en sus propias tablas.
  - Cualquier cambio en la semántica de acceso del spec core obliga a revalidar este spec.

### Capacidades nativas para archivos y adjuntos
- **Context**: La funcionalidad exige almacenar imágenes y PDF dentro de la plataforma y permitir descarga segura.
- **Sources Consulted**:
  - [Use File Column Data in Microsoft Dataverse](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/file-column-data)
  - [Attachments control in Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-attachments)
- **Findings**:
  - Las columnas de tipo archivo en Dataverse proporcionan almacenamiento nativo del binario y generan una columna técnica de nombre del archivo.
  - El control de adjuntos de Power Apps admite abrir, cargar y descargar archivos en tablas Dataverse, con la limitación de que la eliminación solo funciona dentro de formularios y puede quedar deshabilitada por diseño.
  - En móvil la carga se realiza de un archivo cada vez y depende del selector disponible en el cliente.
- **Implications**:
  - El diseño debe conservar la entidad `Adjuntos` pedida por negocio y complementar su esquema con una columna técnica de archivo para almacenar el binario sin salir de Dataverse.
  - La experiencia funcional debe asumir carga unitaria en móvil y no prometer borrado posterior.

### Límites de previsualización PDF en canvas apps
- **Context**: El alcance pide previsualización para imágenes y descarga para ambos tipos, por lo que había que validar si el PDF podía visualizarse de forma homogénea.
- **Sources Consulted**:
  - [PDF viewer control in Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-pdf-viewer)
  - [Add picture control in canvas apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-add-picture)
- **Findings**:
  - El control PDF Viewer es experimental y solo soporta URLs HTTPS directas, accesibles de forma anónima y sin redirecciones.
  - El control Add Picture cubre bien la captura/selección de imágenes y permite experiencia móvil coherente para fotos.
  - El requisito del usuario no pide vista embebida de PDF, solo descarga.
- **Implications**:
  - La arquitectura debe usar visor embebido únicamente para imágenes y descarga/apertura externa para PDF.
  - Intentar incrustar PDF autenticado desde Dataverse introduciría una dependencia frágil y fuera del valor requerido.

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Comentarios y adjuntos incrustados como columnas en `Incidencias` | Extender la tabla padre con campos repetidos o multiarchivo | Menos tablas visibles | Rompe la cardinalidad 1-N requerida y dificulta inmutabilidad | Rechazado |
| Subentidades `Comentarios` y `Adjuntos` con acceso heredado de la incidencia | Cada colaboración vive en su propia fila ligada a `Incidencias` | Mantiene trazabilidad, evita mezclar lifecycle y colaboración, encaja con el brief | Requiere coordinar varias consultas en el detalle | Seleccionado |
| Repositorio externo de documentos y comentarios fuera de Dataverse | App delega a otra plataforma para archivos e historial | Podría escalar aparte | Viola la restricción de almacenamiento único en Dataverse | Rechazado |

## Design Decisions

### Decision: Heredar el alcance desde la incidencia padre en vez de duplicarlo
- **Context**: Comentarios y adjuntos no deben abrir una segunda política de seguridad.
- **Alternatives Considered**:
  1. Guardar rol o centro también en cada comentario y adjunto.
  2. Resolver el permiso siempre contra la incidencia padre y el `AccessContext` vigente.
- **Selected Approach**: Validar lectura y alta de colaboración reutilizando la política de alcance de la incidencia padre.
- **Rationale**: Evita deriva entre specs y garantiza que la colaboración desaparezca del mismo modo que desaparece la incidencia para un usuario fuera de alcance.
- **Trade-offs**: Incrementa la dependencia semántica respecto a `incidencias-core`.
- **Follow-up**: Probar cambios de responsable y cambios de estado sin perder visibilidad de comentarios ni adjuntos.

### Decision: Mantener el esquema de negocio pedido y añadir solo una columna técnica de archivo
- **Context**: Negocio fijó `IdAdjunto`, `IdIncidencia`, `NombreArchivo` y `RutaArchivo`, pero Dataverse necesita una superficie binaria real.
- **Alternatives Considered**:
  1. Guardar solo una ruta externa o un enlace a otro repositorio.
  2. Mantener la tabla `Adjuntos` con los campos de negocio y añadir una columna técnica de archivo gestionada en Dataverse.
- **Selected Approach**: Persistir los campos de negocio solicitados y añadir una columna técnica de archivo no considerada parte del seam funcional.
- **Rationale**: Conserva el contrato visible y satisface la restricción de almacenar el binario dentro de Dataverse.
- **Trade-offs**: Introduce una diferencia entre el modelo de negocio visible y el soporte físico necesario.
- **Follow-up**: Validar que `RutaArchivo` siempre apunta a una referencia resoluble tras la carga.

### Decision: Limitar la previsualización embebida a imágenes
- **Context**: El feature debe ofrecer experiencia consistente en todos los clientes soportados sin depender de capacidades experimentales frágiles.
- **Alternatives Considered**:
  1. Intentar visualizar PDF dentro de la app.
  2. Previsualizar solo imágenes y descargar tanto imágenes como PDF.
- **Selected Approach**: Mostrar vista previa embebida únicamente para imágenes; los PDF se descargan o se abren externamente desde su enlace.
- **Rationale**: Se ajusta exactamente al alcance solicitado y evita las limitaciones del PDF Viewer con URLs autenticadas.
- **Trade-offs**: El usuario sale de la app para consumir un PDF si quiere verlo inmediatamente.
- **Follow-up**: Confirmar que la descarga funciona en web y móvil con el mismo contrato funcional.

### Decision: Hacer el historial estrictamente append-only desde la aplicación
- **Context**: La trazabilidad exige que comentarios y adjuntos no puedan ser alterados tras su registro.
- **Alternatives Considered**:
  1. Permitir edición o borrado con auditoría adicional.
  2. Restringir la app a altas y lecturas, sin UI ni permiso funcional de modificación posterior.
- **Selected Approach**: Exponer solo alta y lectura en la app; no se diseña flujo de edición, sustitución ni borrado.
- **Rationale**: Reduce complejidad y mantiene una semántica inequívoca de historial operativo.
- **Trade-offs**: Corregir un error de contenido requiere un comentario posterior, no una modificación del anterior.
- **Follow-up**: Comprobar en pruebas negativas que no existan acciones residuales de borrado en la UI.

## Risks & Mitigations
- La carga de archivos puede variar entre clientes móviles y navegador — Mitigación: diseñar el flujo como carga unitaria compatible y validar la experiencia en los cuatro clientes soportados.
- El enlace de descarga almacenado en `RutaArchivo` puede quedar inconsistente si no se confirma la carga completa — Mitigación: solo persistir la ruta final tras subida confirmada y mostrar error transaccional si falla.
- La futura necesidad de conteos o filtros por adjuntos/comentarios podría tentar a ampliar este spec — Mitigación: fijar el boundary actual y dejar esas proyecciones para `busqueda-dashboard`.

## References
- [Use File Column Data in Microsoft Dataverse](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/file-column-data) — soporte nativo para almacenar y descargar archivos.
- [Attachments control in Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-attachments) — capacidades y límites del control de carga/descarga.
- [Add picture control in canvas apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-add-picture) — patrón nativo de captura/selección de imágenes.
- [PDF viewer control in Power Apps](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/controls/control-pdf-viewer) — limitaciones que descartan la vista embebida de PDF.
