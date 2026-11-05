# Reglas de producto — Analítica de uso en iOS

<a id="copy-es"></a>

[Índice de reglas](../PRODUCT.md) · Medición agregada de pantallas y recorridos para mejorar MDL sin publicidad, perfiles individuales ni captura de contenido.

<a id="alcance"></a>
## Finalidad y alcance

- **RP-ANA-001.** La analítica de la beta mide exclusivamente en iOS el uso agregado de acceso, registro, creación y edición del diario, fotos, cronología y estadísticas, con las finalidades y datos definidos en este dominio.
- **RP-ANA-002.** La analítica sirve exclusivamente para producto. Quedan fuera publicidad, atribución publicitaria, IDFA, App Tracking Transparency, enlace entre aplicaciones o empresas, compra o venta de audiencias, perfiles individuales, session replay, grabaciones, screenshots, heatmaps, autocaptura de vistas o gestos y captura indiscriminada de cada toque.
- **RP-ANA-003.** La medición registra acciones funcionales intencionales y resultados reales; no intenta reconstruir el contenido del diario ni observar el comportamiento de una persona concreta. La indisponibilidad o el fallo de la analítica nunca impide abrir una pantalla, guardar, sincronizar, iniciar sesión ni completar otro comportamiento de MDL.

<a id="activacion"></a>
## Activación, información y control

- **RP-ANA-004.** En la beta 0.2 de TestFlight, la analítica se habilita por defecto después de mostrar el aviso conjunto de analítica y diagnóstico y de comprobar la configuración de privacidad del proveedor. Esto no es un opt-in: la persona no tiene que activarla. Antes de la primera emisión, MDL comunica de forma clara la finalidad, las categorías medidas y cómo desactivarla.
- **RP-ANA-005.** Ajustes ofrece siempre un control comprensible para desactivar o volver a activar la analítica. Desactivarla detiene cualquier emisión futura y elimina del dispositivo eventos pendientes y correlación efímera de analítica. No cambia la cuenta, el Vault, el diario ni las funciones disponibles.
- **RP-ANA-006.** La medición ordinaria se mantiene inactiva hasta disponer de configuración válida del proveedor, información previa mostrada y distribución beta autorizada. La validación técnica se habilita únicamente en su modo explícito con datos sintéticos.
- **RP-ANA-020.** La salida ordinaria de analítica y diagnóstico remoto se habilita en las builds de la beta 0.2 distribuidas por TestFlight a su audiencia de testers. Además, un modo técnico explícito puede validar los mismos proyectos reales de PostHog Cloud EU y Sentry EU desde Debug, Local, simulador o un candidato de distribución con datos sintéticos o cuentas descartables controladas. Ese modo conserva las mismas allowlists, minimización, retenciones, región y ausencia de contenido prohibido, y debe identificar el entorno técnico sin introducir identidad de usuario; fuera de él, esas builds permanecen cerradas por defecto. La distribución pública de App Store continúa cerrada y requiere una decisión nueva de Producto.
- **RP-ANA-021.** Los eventos, informes y agregados conservados en PostHog tienen una retención máxima de 90 días. La configuración efectiva del proveedor debe comprobarse antes de activar la beta y no puede superar ese máximo. Ampliar la retención exige una decisión explícita nueva de Producto.

<a id="datos"></a>
## Datos permitidos y prohibidos

- **RP-ANA-007.** Cada evento pertenece a un catálogo cerrado y sólo admite dimensiones enumeradas necesarias para la métrica: pantalla, flujo, paso, punto de entrada, método de acceso, acción, resultado, categoría de error saneada, estado de contenido, sección, tipo de elemento y tramo de cantidad. También puede incluir versión/build de la app, versión mayor de iOS e idioma de interfaz.
- **RP-ANA-008.** La analítica nunca incluye identificadores de cuenta, dispositivo publicitario, Crush, experiencia, relación, Vault, foto u operación; email, fecha de nacimiento, género, país ni otros datos de perfil; nombres, fechas o texto introducido; satisfacción, valoraciones, situación relacional, resultado de match o valores estadísticos; tokens, rutas/URLs con parámetros, payloads, ciphertext, fotos ni contenido del Vault.
- **RP-ANA-009.** Mensajes libres de error, `localizedDescription`, excepciones y respuestas de red no son propiedades analíticas. Los fallos se traducen antes de medirlos a una categoría cerrada que no permita recuperar el contenido ni la identidad afectada.
- **RP-ANA-010.** Los recorridos se correlacionan únicamente dentro de una sesión efímera y aleatoria, en memoria. La correlación se renueva al arrancar en frío o tras 30 minutos de inactividad; permanece separada de cuentas, dispositivos y diario y se elimina al desactivar analítica. La medición excluye identidades persistentes y perfiles entre sesiones.

<a id="semantica"></a>
## Semántica de los eventos

- **RP-ANA-011.** Una vista de pantalla se registra cuando la pantalla queda realmente visible. Recomputaciones de SwiftUI, reaparecer un componente hijo o reconstruir el árbol sin una nueva visita no generan otra vista.
- **RP-ANA-012.** Los recorridos distinguen inicio, paso visible, envío, guardado durable local, sincronización remota, finalización, cancelación y fallo. Un evento de finalización sólo se emite tras alcanzar la frontera funcional que su nombre declara; pulsar un botón o iniciar una petición no equivale a completar el recorrido.
- **RP-ANA-013.** El registro se considera completado sólo cuando existen la cuenta confirmada, la autenticación requerida y la sesión MDL publicada; en email incluye la verificación OTP. Solicitar el alta, recibir una capacidad provisional o avanzar de pantalla no es `registration_completed`.
- **RP-ANA-014.** La creación y edición distinguen tipo de entidad, paso visible, envío, guardado local y confirmación remota. Reabrir la corrección de una operación pendiente conserva el mismo recorrido y su confirmación se cuenta una vez. Cronología y estadísticas inician una visita al seleccionarse y la registran al resolver contenido, vacío o fallo; carga, recarga, filtros, paginación o regreso desde segundo plano conservan la misma visita. Salir y volver a seleccionar inicia otra. Los cambios humanos de filtro y aperturas de entidades confirmadas se distinguen de selección inicial, expansión de texto y consulta de pendientes. Cada frontera tiene una única medición y excluye identidad y contenido del diario.
- **RP-ANA-015.** La telemetría técnica de red, seguridad, rendimiento y diagnóstico —incluido rate limiting— no es analítica de producto. Usa el canal técnico permitido por `RP-SEG-006` y `RP-SEG-007`, con su propia retención y sin reutilizar nombres de eventos de producto.

<a id="calidad"></a>
## Calidad y uso de las métricas

- **RP-ANA-016.** Los informes muestran agregados de sesiones, vistas, inicios, resultados y conversión. MDL no ofrece búsquedas, fichas ni recorridos de una persona concreta y no utiliza estos datos para decisiones automatizadas sobre cuentas.
- **RP-ANA-017.** Todo evento nuevo exige definición de finalidad, frontera de emisión, propiedades permitidas, métrica derivada y prueba que demuestre emisión exacta y ausencia de datos prohibidos. Renombrar o cambiar la semántica exige versionar el contrato o coordinar la migración del informe; no se reutiliza un nombre para otro significado.
- **RP-ANA-018.** Los dashboards diferencian ausencia de uso de ausencia de instrumentación. Ninguna decisión de producto se apoya en una métrica hasta verificar cobertura, volumen esperado, duplicados, opt-out y funcionamiento de la versión de app analizada.
- **RP-ANA-019.** Las fotos locales miden `photo_picker_opened`, `photo_selection_completed`, `photo_save_completed`, `photo_save_failed` y `photo_deleted`. Abrir el selector no presupone selección; cancelarlo no emite otro evento. Una selección no vacía emite `photo_selection_completed` con `count_bucket=one|two_to_four|five`. `photo_save_completed` se emite sólo si se persistió al menos una foto y usa el bucket de la cantidad realmente guardada. Una selección parcialmente rechazada que conserva fotos válidas no es un fallo analítico; el aviso visible pertenece a `RP-FOT-008/009`. `photo_save_failed` se limita a una operación en la que no se guardó ninguna foto y sólo lleva `error_category`; `photo_deleted` se emite después de retirar correctamente una foto. La beta sólo ofrece la biblioteca del sistema, por lo que no define ni transmite `source`; el borrado total derivado de eliminar un Crush no es una acción individual de foto.

<a id="catalogo-base-de-eventos"></a>
### Catálogo base de eventos

Cada familia identifica resultados observables y las dimensiones permitidas para medirlos.

| Dominio | Eventos | Dimensiones permitidas | Frontera |
|---|---|---|---|
| Navegación | `screen_viewed` | `screen`, `content_state` | pantalla realmente visible una vez por visita |
| Login | `login_started`, `login_submitted`, `login_completed`, `login_failed`, `login_cancelled` | `method`, `step`, `error_category` | `completed` sólo con sesión MDL publicada |
| Registro | `registration_started`, `registration_step_viewed`, `registration_step_completed`, `registration_submitted`, `registration_completed`, `registration_failed`, `registration_cancelled` | `method`, `step`, `error_category` | email completa tras OTP y sesión; Apple tras alta atómica y sesión |
| Creación | `creation_started`, `creation_step_viewed`, `creation_submitted`, `creation_saved_locally`, `creation_synced`, `creation_failed`, `creation_cancelled` | `entity_type`, `step`, `error_category` | separa durable local de confirmación remota; RP-ANA-014 y [taxonomía técnica](../../../ios/docs/ARCHITECTURE.md#taxonomía-de-formularios-y-visitas) |
| Edición | `edit_started`, `edit_step_viewed`, `edit_submitted`, `edit_saved_locally`, `edit_synced`, `edit_failed`, `edit_cancelled` | `entity_type`, `step`, `error_category` | no adjunta entidad ni valores editados; RP-ANA-014 y [taxonomía técnica](../../../ios/docs/ARCHITECTURE.md#taxonomía-de-formularios-y-visitas) |
| Fotos | `photo_picker_opened`, `photo_selection_completed`, `photo_save_completed`, `photo_save_failed`, `photo_deleted` | `count_bucket`, `error_category` | biblioteca del sistema sin `source`; selección no vacía y guardado real con `one\|two_to_four\|five`; fallo sólo si no se guarda ninguna; borrado individual confirmado |
| Timeline | `timeline_viewed`, `timeline_filter_changed`, `timeline_item_opened` | `scope`, `content_state`, `filter`, `item_type` | `scope=global\|crush`; `filter=all\|crushes\|experiences\|relationships`; `item_type=crush\|experience\|relationship` sólo al navegar desde un elemento confirmado; sin acciones duplicadas, IDs, fechas ni contenido |
| Estadísticas | `statistics_viewed` | `content_state` | visita al seleccionar la pestaña y resolver su primer estado, sin duplicar por recarga, background ni `screen_viewed`; las secciones estáticas y la ayuda de color no emiten eventos |
| Preferencia | `analytics_preference_changed` | `enabled` | sólo puede emitirse al activar; al desactivar se aplica primero el corte y no se envía el evento |

El enum `screen` puede representar login, registro, Home, lista y perfil de Crush, alta/edición de Crush, alta/edición de experiencia, galería/fotos, Timeline, Estadísticas y Ajustes, pero la allowlist inicial sólo emite `screen_viewed` para Home y lista de Crushes. Timeline y Estadísticas conservan sus eventos especializados y los embudos explican el resto sin duplicar visitas.

### Envío vigente a PostHog

| Clasificación | Eventos | Destino inicial | Para qué se usan |
|---|---|---|---|
| Activos al abrir la beta | `screen_viewed(home\|crush_list)`, `timeline_viewed`, `statistics_viewed` | PostHog | adopción de las cuatro superficies principales; el botón central se mide por los embudos de creación |
| Activos al abrir la beta | todos los `login_*` y `registration_*` vigentes | PostHog | conversión y abandono de acceso/alta por email o Apple, paso y categoría cerrada |
| Activos al abrir la beta | todos los `creation_*` y `edit_*` vigentes | PostHog | inicio, paso, envío, guardado local, sync, fallo y cancelación de Crush y Experience |
| Activos al abrir la beta | `photo_picker_opened`, `photo_save_completed`, `photo_save_failed` | PostHog | descubrimiento del selector y capacidad de completar un guardado |
| Activos al abrir la beta | `timeline_filter_changed`, `timeline_item_opened` | PostHog | si el Timeline se explora y conduce a contenido confirmado |
| Activo al abrir la beta | `analytics_preference_changed(enabled=true)` | PostHog | verificar reactivaciones; desactivar corta primero y no emite evento |
| Excluidos | autocaptura de pantallas/toques, replay, heatmaps, perfiles, IDs persistentes, contenido, valores del diario y cantidades exactas | ninguno | prohibidos por `RP-ANA-002` y `RP-ANA-008` |

Los nueve eventos de RP-ANA-028 también están en la allowlist bajo los mismos gates y preferencias, en un bloque separado del uso real. El adaptador es una allowlist: un evento tipado que no figure como activo se descarta antes del proveedor. PostHog no usa `identify`, alias, grupos, perfiles de personas, replay, autocaptura, encuestas ni feature flags. Su identidad técnica es la correlación efímera en memoria de `RP-ANA-010`; no se persiste un `distinct_id` entre sesiones.

### Cinco tableros iniciales

1. **Uso de superficies:** sesiones instrumentadas que visitan Home, lista de Crushes, Timeline y Estadísticas, con versión y estado resuelto.
2. **Acceso y alta:** conversión y abandono por método y paso, separando cancelación de fallo.
3. **Valor central:** creación y edición por entidad desde inicio hasta guardado local y sync; brecha `saved_locally → synced`.
4. **Timeline y Estadísticas:** visitas, filtros y aperturas del Timeline, y proporción `populated|empty|failed` de Estadísticas.
5. **Fotos:** sesiones que abren el selector y proporción de guardado completado o fallido.

No habrá dashboard por cada evento, usuarios únicos persistentes ni exploración individual. Antes de tomar decisiones, cada tablero debe mostrar versión, cobertura del rollout y opt-out y superar una línea base de calidad.

<a id="demo"></a>
## Ampliación aditiva para onboarding y demo

- **RP-ANA-022.** La medición de elección, entrada, exploración y salida de demo sólo se añadirá después de definir y aprobar cada evento conforme a RP-ANA-017. Usará el runtime, aviso, opt-out, correlación efímera y gates vigentes. No se transmiten datos, imágenes ni IDs ficticios o reales, búsquedas, filtros seleccionados del perfil, países, fechas ni valores estadísticos. Las visitas ficticias no se contabilizan como uso del diario real ni contaminan los tableros actuales; la presentación independiente debe comprobar esa separación. Abrir la interfaz nativa navegable de contenido fijo no crea un login, registro, alta de Crush, experiencia o relación reales en las métricas existentes.

- **RP-ANA-023.** Los informes adicionales usan sesiones instrumentadas, elecciones, entradas, visitas y salidas reales, separando demo de uso real y comprobando cobertura, versión, duplicados y opt-out. La conversión demo→real y la creación posterior sólo se correlacionan dentro de la misma sesión efímera. Los eventos actuales de creación y consulta se conservan para los resultados reales: no se etiquetan como «primer Crush de la vida de la cuenta», usuarios únicos o retención entre días cuando la instrumentación no puede demostrarlo. Una experiencia y una relación no se cuentan como equivalentes: el catálogo actual no demuestra por sí solo un hito relacional nuevo. La medición específica de ese resultado requiere definir y aprobar su evento conforme a RP-ANA-017 antes de implementarlo.
- **RP-ANA-024.** La analítica permite evaluar comprensión del producto, utilidad de los ejemplos y uso de funciones mediante métricas agregadas. El feedback voluntario utiliza un canal separado y excluye contenido del diario e identidades analíticas; la consulta de métricas conserva los límites de correlación definidos.
- **RP-ANA-025.** La beta recoge feedback voluntario mediante TestFlight desde el primer día. Las instrucciones invitan a comentar comprensión, utilidad, dificultades y motivos para volver después de 5–7 días. Es una indicación para el tester, separada de recordatorios o seguimiento individual en la aplicación. Los comentarios y capturas utilizan contenido ficticio y excluyen información privada; el canal de Apple mantiene su tratamiento propio.

<a id="objetivos-demo"></a>
### Objetivos de la medición de demo

- **RP-ANA-026.** La ampliación mide los cuatro objetivos de la tabla siguiente en un bloque separado del uso real, conservando la instrumentación y los cinco tableros actuales. Los resultados se expresan como sesiones instrumentadas, oportunidades mostradas, visitas, resultados y recorridos dentro de la misma sesión efímera; no como personas únicas ni retención entre días. Una consulta demuestra que se vio una página de ejemplo, no uso de la función real, utilidad percibida ni preferencia. El acceso y las creaciones reales posteriores usan sus eventos vigentes, sin duplicarlos ni redefinirlos. RP-ANA-028 fija la definición aprobada de eventos, fronteras, dimensiones cerradas, tasas y pruebas conforme a RP-ANA-017; la aprobación documental no certifica implementación ni cobertura.

| Objetivo | Información necesaria para responderlo |
|---|---|
| Elección de explorar o empezar la historia propia | Presentación efectiva del punto de elección y opción elegida. Las oportunidades se distinguen por el punto de entrada y la etapa habilitada de RP-DEMO-012; una opción que no se ofrece no se interpreta como falta de interés. |
| Entrada efectiva a demo | Intención de entrar, conjunto ficticio disponible para consultar y fallo saneado. Pulsar Explorar no equivale a completar la entrada. |
| Consulta de superficies de demo | Visitas a la interfaz nativa navegable de contenido fijo (listado, mapa, ficha con sus pestañas, Timeline general y estadísticas completas) mediante dimensiones cerradas. No se mide la identidad del ejemplo ni su contenido y las visitas no contaminan los informes reales. |
| Paso al diario propio | Elección de empezar la historia propia y transición efectiva; seguimiento dentro de la misma sesión efímera de los resultados existentes de acceso, registro y creación real. Se distinguen la elección, el acceso o registro completado y el guardado real; no se afirma que un Crush sea el primero de la vida de la cuenta. |

- **RP-ANA-027.** La medición de cambios relacionales posteriores al alta inicial distingue guardado local y confirmación remota. Incluye aperturas posteriores de período y excluye experiencia independiente, alta inicial, edición, corrección y duplicados por reintento. Los eventos comunican que ocurrió el resultado y excluyen situación, motivo, fechas, persona e identidades funcionales.

<a id="catalogo-aditivo"></a>
### Medición de demo y cambios relacionales

- **RP-ANA-028.** La medición distingue elección de explorar, intento de entrada, resultado de entrada, consulta de ejemplos, salida y cambios relacionales reales. Cada evento se emite en su frontera efectiva una vez y admite únicamente las propiedades cerradas de las tablas siguientes; la demo permanece separada del uso del diario.

#### Eventos y fronteras

| Evento lógico | Cuándo se emite | Propiedades de evento permitidas |
|---|---|---|
| `discovery_option_viewed` | La opción queda realmente visible en su punto habilitado. Una emisión por opción y visita; si se muestran ambas, hay una oportunidad por cada una. Recomposición o componentes hijos no crean oportunidades. | `flow`, `entry_point`, `action` |
| `discovery_option_selected` | La persona elige una opción disponible y se acepta su acción. No equivale a cargar demo, autenticar o guardar datos; un doble toque que no inicia otra acción no duplica el evento. | `flow`, `entry_point`, `action` |
| `demo_entry_started` | Comienza un intento efectivo de presentar la interfaz nativa de ejemplos locales. Una elección que no llega a iniciar el intento no lo emite; un reintento real comienza otro intento. | `flow`, `entry_point` |
| `demo_entry_completed` | La primera superficie de demo queda visible con el conjunto de ejemplos disponible para consultar. No se emite al tocar el botón, preparar sólo sus recursos o mostrar una pantalla de carga. Una vez por intento completado. | `flow`, `entry_point` |
| `demo_entry_failed` | Un intento termina con un fallo real que impide entrar. Una vez por resultado final fallido, no por cada petición interna; una cancelación o el cierre de la app no se fabrica como error. | `flow`, `entry_point`, `error_category` |
| `demo_surface_viewed` | Un destino de consulta de demo queda realmente visible y resuelve su primer estado de contenido, vacío o fallo. Una vez por visita; no durante carga, recomposición, recarga, cambio de filtro ni mera expansión de contenido. Volver a elegir o navegar a ese destino después de salir abre otra visita. | `flow`, `screen`, `content_state` |
| `demo_exited` | Una salida explícita retira el contexto de demo y presenta el destino correspondiente: recorrido para empezar la historia propia o regreso al contexto anterior. Elegir empezar se mide antes como intención; este evento no afirma que el registro o el diario propio estén completados. No se intenta emitir al matar la app. | `flow`, `entry_point=demo`, `action` |
| `relationship_change_saved_locally` | Se confirma el guardado durable local de una nueva acción compuesta de Experience y cambio posterior al alta inicial del Crush, conforme a RP-ANA-027. No se emite por selección de situación, envío, experiencia independiente, edición o alta inicial del Crush. | Ninguna propiedad funcional; sólo metadata común autorizada. |
| `relationship_change_synced` | El servidor confirma la acción compuesta correspondiente y su resultado completo. Una operación guardada o una petición iniciada no son confirmación. Retry o corrección no duplican la misma confirmación y no emiten otro guardado nuevo. | Ninguna propiedad funcional; sólo metadata común autorizada. |

La reanudación, cancelación o sustitución de contexto conserva su autoridad: una respuesta tardía no inventa resultados para el modo o sesión posterior. La implementación debe demostrar la frontera y la deduplicación sin introducir correlación persistente ni transmitir identidades de operación. Una sesión analítica renovada no se enlaza con la anterior para completar un embudo.

#### Valores cerrados

| Propiedad | Valores y alcance |
|---|---|
| `flow` | `discovery_beta`, `discovery_public`: identifica la etapa funcional correspondiente a RP-DEMO-012. No abre el transporte público; se conservan RP-ANA-006/020. |
| `entry_point` | `initial_onboarding`, `guest_home`, `settings`, `post_protection`, `demo`. Se usa sólo el punto real habilitado para esa etapa; `demo` es el origen de la elección de empezar la historia propia y de la salida. No admite rutas o URLs libres. |
| `action` en opciones | `explore_demo`, `start_story`. Una opción no ofrecida no genera exposición ni elección. |
| `action` en salida | `start_story`, `return`. No se interpreta como resultado de autenticación o creación. |
| `screen` en demo | `crush_list`, `map`, `crush_profile`, `timeline_global`, `statistics`, correspondientes a los destinos nativos de RP-DEMO-014. No identifican el ejemplo mostrado ni certifican uso del diario real. Las pestañas de ficha no añaden identidades o propiedades de contenido. |
| `content_state` en demo | `populated`, `empty`, `failed`. El fallo procede de la presentación real; un error que conserva contenido confirmado no convierte ese contenido en vacío. |
| `error_category` en entrada | `validation`, `authentication`, `authorization`, `conflict`, `rate_limited`, `network`, `storage`, `vault_locked`, `unknown`, conservando la clasificación saneada existente. En la presentación estática sólo se usa una causa local realmente observada; no se simulan fallos de autenticación, red o Vault. Las cancelaciones no producen `demo_entry_failed`. |

No se añaden propiedades funcionales a los dos eventos relacionales. Los eventos nuevos conservan exclusivamente la metadata común ya autorizada de versión/build, versión mayor de iOS e idioma, y la correlación efímera de RP-ANA-010. No incorporan IDs, cuentas, ejemplos, estados relacionales, resultados de match, filtros seleccionados, valores estadísticos, textos, fotos, mensajes libres o URLs.

#### Métricas derivadas y evidencia exigida

| Objetivo | Resultado que puede informarse | Prueba exigida antes de usarlo |
|---|---|---|
| Elección | Sesiones con exposición a una opción y sesiones que la eligen después, separadas por punto de entrada y etapa. La tasa usa sólo sesiones con oportunidad confirmada en esa misma correlación efímera. | Ambas opciones visibles emiten sus oportunidades una vez; una opción ausente no emite. Selección y recomposición no duplican exposiciones; un doble toque rechazado no duplica elección. |
| Entrada | Intentos iniciados, completados y fallidos y sesiones con cada resultado. Un intento sin resultado observado no se etiqueta automáticamente como fallo o abandono. | Retrasar datos no emite éxito; cada resultado real cierra una vez el intento correspondiente; cancelación o respuesta retirada no producen éxito o fallo falso. |
| Consulta | Sesiones con consulta de cada superficie y número de visitas. La proporción usa sesiones con alguna visita de demo observada; no necesita inventar otra entrada al renovarse la correlación. | Navegación real produce una visita; carga, filtros, recarga y componentes hijos no. No se emiten visitas ficticias al catálogo real ni contenido del ejemplo; una vista de ejemplos no se informa como uso de una función del diario. |
| Paso al diario propio | Elecciones de empezar, salidas efectivas y resultados posteriores existentes de acceso, registro y creación dentro de la misma sesión efímera. Se informa cada frontera por separado y sólo se construye el embudo cuando se observa su orden y cobertura. | Elegir empezar no emite registro ni creación completados. Login de una cuenta existente se distingue del registro; guardar localmente no emite sync. No se correlacionan sesiones distintas ni se copian ejemplos al diario real. |
| Cambios relacionales | Cambios guardados localmente, confirmaciones remotas y sesiones con cada resultado. Se muestran por separado; sus totales no se presentan por sí solos como una tasa por operación o persona, porque una confirmación puede llegar en otra sesión. | Alta inicial, experiencia independiente y edición no emiten cambio nuevo. Un cambio posterior y una apertura de período posterior sí se identifican por su acción compuesta real; retry/corrección no duplican el guardado ni la confirmación. Ningún payload revela situación, motivo, fecha o identidad. |

Todas las pruebas incluyen opt-out, gate cerrado, receptor fallido, allowlists exactas y ausencia de datos prohibidos. Los informes muestran versión, cobertura y duplicados conforme a RP-ANA-018. La tabla define pruebas requeridas; la evidencia y sus límites se conservan en [Testing iOS](../../../ios/docs/TESTING.md#analítica-de-producto). La coordinación documental no ejecuta suites o builds ni certifica una publicación.

### Escenarios de aceptación de la ampliación analítica

| ID | Dado / Cuando / Entonces | Regla |
|---|---|---|
| GWT-ODM-09 | Dados los canales de telemetría, cuando se explora demo con opt-out, gate cerrado o receptor fallido, entonces no altera su comportamiento; cuando se mide bajo el catálogo aprobado, los ejemplos no cuentan como actividad real ni contienen IDs/contenido. | RP-ANA-022..024, RP-ANA-005/006/008/010 |
| GWT-ANA-09 | Dada una nueva acción relacional real posterior al alta inicial del Crush, cuando se guarda y después se confirma remotamente, entonces se distinguen ambas fronteras sin transmitir situación, motivo, fechas o identidades. El alta inicial del Crush, una experiencia independiente, una edición y los reintentos no inventan cambios nuevos. Los eventos y propiedades siguen el contrato de RP-ANA-028 y no duplican las métricas existentes. | RP-ANA-027, RP-ANA-008/010/014/017/023 |

## Escenarios de aceptación del catálogo base y diagnóstico

### GWT-ANA-01 — Evento permitido y mínimo

**Dado** un evento tipado del catálogo
**Cuando** el cliente lo entrega al receptor
**Entonces** sólo contiene propiedades permitidas y nunca contenido, IDs de negocio ni mensajes libres.

### GWT-ANA-02 — Opt-out efectivo

**Dado** que la persona desactiva la analítica
**Cuando** continúa usando cualquier pantalla o flujo
**Entonces** no se emite ni encola ningún evento y se elimina la correlación efímera pendiente sin afectar la app.

### GWT-ANA-03 — Registro completado con verdad

**Dado** un alta por email todavía pendiente de OTP o sesión
**Cuando** avanza por sus pasos
**Entonces** puede medir los pasos alcanzados pero no emite `registration_completed` hasta publicar la sesión MDL válida.

### GWT-ANA-04 — Guardado y sync diferenciados

**Dado** un formulario válido de creación o edición
**Cuando** se guarda durablemente pero la sincronización queda pendiente
**Entonces** emite `*_saved_locally` y no `*_synced` hasta la confirmación remota.

**Dado** que una operación pendiente rechazada ya pertenece a un recorrido de creación o edición
**Cuando** la persona abre «Corregir» y reemplaza esa misma intención durable
**Entonces** no se contabiliza otro inicio, paso, envío, guardado ni fallo del embudo; si la operación finalmente se confirma de forma remota, emite una sola vez `*_synced`.

### GWT-ANA-05 — Visita sin duplicados

**Dado** que una pantalla ya está visible
**Cuando** la aplicación actualiza una vista sin una nueva visita
**Entonces** no duplica `screen_viewed`.

Para Timeline y Estadísticas, la visita empieza al seleccionar la pestaña correspondiente —incluida la pestaña Timeline dentro del perfil de un Crush— y se emite una sola vez al alcanzar el primer estado resuelto (`populated`, `empty` o `failed`), nunca durante `loading`. Cambiar filtros, recargar, paginar o volver del background no abre otra visita; salir de la superficie y volver a seleccionarla sí. Un fallo de recarga con contenido confirmado conserva `populated`, y estas superficies no emiten además `screen_viewed`.

### GWT-ANA-06 — Fallo no identificable

**Dado** un error de autenticación, red, Vault o persistencia
**Cuando** se mide el resultado
**Entonces** se transforma a una categoría cerrada y no se envían texto libre, ruta concreta, payload ni identificadores.

### GWT-ANA-07 — Analítica no bloqueante

**Dado** un receptor ausente, apagado o fallido
**Cuando** la persona ejecuta una función de MDL
**Entonces** el resultado funcional no cambia y no se presenta un error de analítica.

### GWT-ANA-08 — Rollout y validación técnica

**Dado** que la build no pertenece a la beta 0.2 de TestFlight y tampoco se abrió el modo técnico explícito, falta configuración válida o todavía no se mostró el aviso exigible para la beta
**Cuando** se inicia el runtime de analítica
**Entonces** la salida externa permanece desactivada durante una ejecución local ordinaria
**Y** una build fuera de TestFlight no presenta por defecto el aviso ni los controles de rollout y no persiste la analítica como habilitada.

**Dado** el modo técnico explícito fuera de TestFlight
**Cuando** se valida PostHog o Sentry
**Entonces** sólo se transmiten eventos sintéticos o de una cuenta descartable controlada mediante las mismas allowlists y reglas de minimización
**Y** cerrar ese modo devuelve la build a medición desactivada y diagnóstico local.

### GWT-OBS-01 — Diagnóstico estructurado

Cada diagnóstico utiliza información técnica de la lista permitida y excluye texto libre y contenido privado.

### GWT-OBS-02 — Correlación de fallos

Una petición y su respuesta conservan una referencia técnica que permite correlacionar el fallo entre cliente y servidor sin incorporar datos privados.

### GWT-OBS-03 — Información mínima

El diagnóstico contiene causa técnica, severidad, contexto de versión y referencia necesaria para investigarlo, limitado a los datos permitidos.

### GWT-OBS-04 — Información accionable

El diagnóstico de producción conserva fallos e hitos técnicos expresamente permitidos y excluye actividad rutinaria o contenido de objetos.

### GWT-OBS-05 — Envío remoto autorizado

El diagnóstico remoto permanece inactivo sin distribución o validación técnica autorizada, configuración válida, información previa y control activo; el diagnóstico local conserva su función.
