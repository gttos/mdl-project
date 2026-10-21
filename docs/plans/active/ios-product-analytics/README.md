# Analítica de producto y trazabilidad operativa iOS

> **Definición funcional vigente:** [Analítica de uso en iOS](../../../rules/ANALYTICS.md). Producto seleccionó PostHog Cloud EU para analítica y Sentry EU para diagnóstico y autorizó ambos canales, con 90 y 30 días máximos respectivamente, en la beta 0.2 distribuida por TestFlight. Las validaciones técnicas explícitas pueden usar esos mismos proyectos con datos sintéticos o cuentas descartables controladas; la revisión jurídica posterior puede exigir cambios antes del lanzamiento público, pero no bloquea esta beta.
>
> **Estado:** `IOS-OBS-01..03` e `IOS-ANA-00..05` aceptadas (incluidos `IOS-ANA-01R`, `IOS-ANA-03R` e `IOS-ANA-05R`). PostHog Cloud EU y Sentry EU están verificados únicamente con datos sintéticos y las builds ordinarias mantienen cerrados tanto el envío analítico como el diagnóstico remoto.
> **Prioridad:** necesaria antes de usar datos para priorizar mejoras de la beta iOS.
> **Proyectos:** Product e iOS. API, Android y LLM quedan fuera.
> **Siguiente trabajo de esta iniciativa:** `IOS-ROLLOUT-06`, autorizado y ejecutable ahora. La implementación, las pruebas en simulador y las validaciones sintéticas contra los proveedores no esperan a la publicación de la API 0.2; el recorrido integrado final con una cuenta beta sí se repite después del despliegue. No depende de una aprobación jurídica adicional.

## Objetivo

Obtener métricas fiables y agregadas sobre qué pantallas y recorridos utiliza la gente en iOS, y dejar además una trazabilidad técnica que permita encontrar fallos reales desde el dispositivo hasta API, CloudWatch y el receptor de errores aprobado. Analítica y diagnóstico permanecen separados y ninguno mide contenido privado.

## Baseline confirmado

- `AnalyticsEvent` es el único catálogo ejecutable y sólo acepta dimensiones de enums cerrados; `AnalyticsManager` y la API de strings/diccionarios están retirados.
- Toda build ordinaria resuelve `NoOpAnalyticsTracker`. El adaptador mínimo de PostHog existe detrás de un gate hermético de integración, sin SDK, cola ni persistencia analítica; el receptor de grabación vive sólo en tests.
- Login y registro por email y Apple están instrumentados con eventos tipados en fronteras reales de inicio, paso visible/completado, envío, cancelación, fallo y finalización. `registration_completed` sólo se emite después del OTP o del alta atómica Apple y de publicar la sesión MDL; una continuación, un registro pendiente o una respuesta tardía nunca cuentan como completado.
- La deduplicación de los pasos de registro vive sólo en memoria y por instancia de flujo. Restaurar una sesión, usar biometría o reanudar una pantalla OTP no inventa un login ni un registro nuevos; el OTP restaurado sólo puede medir el paso realmente visible y su resultado.
- Las altas y ediciones de Crush y experiencia están instrumentadas con la taxonomía cerrada de `RP-ANA-014`: separan envío, guardado durable y confirmación remota. Una operación rechazada reabierta mediante «Corregir» no crea un embudo nuevo; las entradas «Editar» retienen sus eventos hasta confirmar que no existe una corrección pendiente y fallan cerradas si esa comprobación no puede resolverse.
- Los wrappers muertos de Home, los eventos Legal con versiones o errores libres, el falso `register_success` y el antiguo `statistics_screen_viewed` están retirados. Fotos locales, Timeline y Estadísticas usan ahora eventos tipados con dimensiones cerradas; Timeline y Estadísticas miden una visita resuelta sin duplicarla por carga, recarga, recomposición o background y sin emitir además `screen_viewed`.
- La aceptación Legal confirmada por API no se mide: no tiene evento en el catálogo vigente. Añadirlo requeriría primero una regla de Producto.
- `api_rate_limit_exceeded` permanece exclusivamente en el canal diagnóstico saneado.
- El runtime de preferencia está implementado, pero toda build ordinaria mantiene el rollout cerrado: no lee ni escribe la preferencia, no crea correlación, no entrega eventos y no muestra aviso ni control de Ajustes. La superficie futura sólo se abre en UI tests herméticos con un argumento explícito.
- La dependencia Firebase retirada sigue protegida por tests; esta iniciativa no autoriza a reintroducirla ni presupone proveedor.

### Observabilidad y recorrido actual de los logs

| Origen | Destino real actual | Hallazgo |
|---|---|---|
| Diagnóstico iOS ordinario | eventos estructurados → Unified Logging (`os.Logger`) del dispositivo, por subsystem y categoría | no hay transporte remoto activo en builds ordinarias; Release conserva `warning`/`error` e `info` sólo para códigos allowlisted; no queda adaptador legacy ni API de texto libre |
| Bypasses directos iOS | retirados y protegidos por guarda | no quedan `print`, `debugPrint`, `NSLog` ni accesos a `os.Logger` fuera del sink |
| Amplify/Cognito iOS | retirado del target y del artefacto distribuible | no aporta logging ni transporte; cualquier observabilidad futura debe partir del runtime vigente |
| Sentry iOS | SDK y sink allowlisted integrados detrás de un gate hermético exclusivo de UI tests | diagnóstico y crash sintéticos verificados contra un proyecto Sentry EU aislado, con cuatro alertas operativas; el SDK permanece dormante en builds ordinarias |
| Laravel API | `stderr` → Docker `awslogs` | producción envía Nginx, PHP-FPM y Laravel combinados a `/ec2/mdl-api/prod`, stream `app-container` |
| Sentry API | reporter allowlisted para excepciones JSON no manejadas | no recibe todos los logs; `sentry_logs` existe como canal, pero producción usa `LOG_CHANNEL=stderr` |
| Correlación HTTP | API acepta `X-Request-Id`, responde `X-Trace-Id` y añade `traceId` al error | iOS envía un UUID canónico por intento y correlaciona cabecera y envelope sin registrar request o contenido privado |
| Operación CloudWatch | configuración del driver en Compose | el repositorio no demuestra retención, filtros métricos, alarmas ni separación de streams; deben validarse en AWS sin leer secretos |

El runtime iOS tiene 103 sitios de emisión estructurada y 94 códigos únicos. Los contratos automatizados cubren severidad, códigos y metadata cerrada, redacción, ausencia de bypasses, límites de volumen y correlación con API. Los errores ya diagnosticados por el transporte no se duplican en dominio, y los sitios repetitivos se acotan por arranque o pasada. La integración remota está validada únicamente mediante el arnés hermético con datos sintéticos y no está habilitada en builds ordinarias; Unified Logging sigue siendo su único destino diagnóstico.

## Alcance

- Corregir o retirar los eventos actuales ambiguos, muertos o inseguros.
- Crear un contrato tipado, inyectable y comprobable para eventos y propiedades cerradas.
- Incorporar aviso claro y un control de opt-out en Ajustes únicamente durante el rollout autorizado de `IOS-ROLLOUT-06`; antes sólo pueden existir como superficie hermética de tests.
- Medir login y registro por email y Apple, sin credenciales ni identificadores.
- Medir alta y edición de Crush/experiencia y sus resultados durable/sync.
- Medir uso funcional de fotos locales, Timeline y Estadísticas.
- Definir métricas y dashboards sólo después de verificar la calidad de instrumentación.
- Integrar PostHog Cloud EU detrás del gate cerrado, validar con datos sintéticos y abrir el rollout real sólo en la beta 0.2 de TestFlight tras comprobar la retención máxima de 90 días.
- Revisar cada llamada de log iOS y decidir si se conserva, elimina, rebaja a debug o se convierte en un diagnóstico estructurado.
- Retirar `print`/`debugPrint`/`NSLog` directos del runtime y unificar severidad, categoría, código y metadata allowlisted.
- Propagar el mismo `traceId` desde iOS hasta API y recuperarlo de toda respuesta para buscar el incidente en CloudWatch/Sentry.
- Integrar Sentry EU detrás del gate cerrado con crashes y una allowlist mínima de errores; abrir el rollout real sólo en la beta 0.2 de TestFlight tras comprobar la retención máxima de 30 días.

Fuera de alcance:

- publicidad, campañas, atribución, IDFA, ATT o tracking entre apps;
- replay de sesión, screenshots, heatmaps, autocaptura o registro indiscriminado de toques;
- perfiles o búsquedas individuales;
- contenido, valores o identificadores del diario, la cuenta o el Vault;
- crash reporting, logs de red, rendimiento o seguridad dentro del canal de producto;
- session replay, screenshots, breadcrumbs automáticos de UI/red o performance profiling dentro del canal diagnóstico;
- cambios de API, endpoints, modelos persistidos o instrumentación Android/LLM.

## Separación de canales

| Canal | Responde a | Puede contener | Nunca contiene |
|---|---|---|---|
| Analítica de producto | qué pantallas/recorridos se usan y dónde abandonan las sesiones | evento y dimensiones cerradas de `RP-ANA-*` | errores crudos, stacks, IDs, contenido o diagnóstico de una persona |
| Diagnóstico iOS | qué componente falló, en qué versión y con qué referencia técnica | código estable, severidad, categoría, build/entorno, `traceId`, estado técnico allowlisted | eventos de producto, replay, inputs, payloads, cuenta, diario o Vault |
| Observabilidad API | qué ocurrió en servidor y en qué frontera | `traceId`, ruta plantilla, método, status, clase/código técnico y metadata saneada | body/query/headers completos, secretos o contenido privado |

Desactivar analítica no elimina la capacidad técnica mínima de investigar errores. El diagnóstico remoto tiene su propio control, gate de TestFlight, aviso, retención y configuración; no se esconde dentro del control analítico.

## Proveedores, envío y métricas de la beta

La selección técnica queda fijada así:

| Destino | Finalidad | Estado autorizado ahora | Gate productivo |
|---|---|---|---|
| PostHog Cloud EU | analítica agregada de producto iOS | beta 0.2 de TestFlight tras `IOS-ROLLOUT-06`; configuración sin perfiles | aviso/control y retención efectiva máxima de 90 días |
| Sentry EU | crashes iOS y diagnósticos técnicos allowlisted | beta 0.2 de TestFlight tras `IOS-ROLLOUT-06` | aviso/control propio y retención efectiva máxima de 30 días |
| Sentry API | excepciones backend ya allowlisted | contrato vigente de API; no recibe eventos de producto | operación y retención backend |
| CloudWatch | logs operativos de API | contrato vigente de infraestructura; no recibe directamente eventos iOS | validación operativa de AWS |

Las validaciones técnicas pueden usar los proyectos reales de la organización EU con datos sintéticos o cuentas descartables controladas, pero no contenido real del diario, Vault o fotos ni ningún dato prohibido. Las claves se inyectan por configuración local no versionada. Debug, Local, simulador y tests conservan `NoOpAnalyticsTracker` y sólo Unified Logging local salvo que se abra deliberadamente el modo técnico explícito; ese modo no equivale a habilitar una distribución pública.

### Matriz analítica

| Clasificación | Eventos | Destino inicial | Para qué se usan |
|---|---|---|---|
| Activos al abrir la beta | `screen_viewed(home|crush_list)`, `timeline_viewed`, `statistics_viewed` | PostHog | adopción de las cuatro superficies principales; el botón central se mide por los embudos de creación |
| Activos al abrir la beta | todos los `login_*` y `registration_*` vigentes | PostHog | conversión y abandono de acceso/alta por email o Apple, paso y categoría cerrada |
| Activos al abrir la beta | todos los `creation_*` y `edit_*` vigentes | PostHog | inicio, paso, envío, guardado local, sync, fallo y cancelación de Crush y Experience |
| Activos al abrir la beta | `photo_picker_opened`, `photo_save_completed`, `photo_save_failed` | PostHog | descubrimiento del selector y capacidad de completar un guardado |
| Activos al abrir la beta | `timeline_filter_changed`, `timeline_item_opened` | PostHog | si el Timeline se explora y conduce a contenido confirmado |
| Activo al abrir la beta | `analytics_preference_changed(enabled=true)` | PostHog | verificar reactivaciones; desactivar corta primero y no emite evento |
| Implementados pero no enviados inicialmente | `photo_selection_completed`, `photo_deleted` | ninguno | se conservan tipados para una decisión futura; no alimentan los dashboards iniciales |
| Definido pero no instrumentado inicialmente | `screen_viewed` para login, registro, perfil, formularios, galería y Ajustes | ninguno | los embudos específicos ya explican esos recorridos; se evita duplicar volumen |
| Excluidos | autocaptura de pantallas/toques, replay, heatmaps, perfiles, IDs persistentes, contenido, valores del diario y cantidades exactas | ninguno | prohibidos por `RP-ANA-002` y `RP-ANA-008` |

El adaptador es una allowlist: un evento tipado que no figure como activo se descarta antes del proveedor. PostHog no usa `identify`, alias, grupos, perfiles de personas, replay, autocaptura, encuestas ni feature flags. Su identidad técnica es la correlación efímera en memoria de `RP-ANA-010`; no se persiste un `distinct_id` entre sesiones.

### Allowlist diagnóstica de Sentry iOS

Sentry iOS recibe los crashes no manejados sin identidad de usuario y sólo estos 29 diagnósticos estructurados `warning`/`error` en la primera beta. Esta allowlist pertenece exclusivamente al cliente iOS; no modifica el reporter Sentry ya existente en la API:

- contrato/correlación de red: `net.response.decode_failed`, `net.response.unexpected_status`, `net.trace.missing`, `net.trace.mismatch`;
- autenticación: `auth.register.unexpected_failure`, `auth.apple.authorize_unexpected_failure`, `auth.apple.register_unexpected_failure`, `auth.apple.link_unexpected_failure`, `auth.biometric.bootstrap_failed`;
- integridad del Vault: `vault.onboarding.initialization_failed`, `vault.onboarding.migration_failed`, `vault.onboarding.setup_inconsistent`, `vault.onboarding.finalize_without_key`, `vault.status.keychain_status_unexpected`, `vault.unlock.key_unavailable`, `vault.setup.keychain_store_failed`, `vault.context.remote_encapsulation_invalid`;
- destrucción y recuperación: `vault.account_deletion.connection_alive`, `vault.account_deletion.directory_removal_failed`, `vault.recovery.write_failed`, `vault.recovery.sync_queue_failed`, `vault.recovery.crush_deletion_failed`, `vault.recovery.experience_deletion_failed`;
- protección local: `keychain.save_failed`, `keychain.read_failed`, `keychain.delete_failed`, `keychain.encode_failed`, `media.backup_exclusion_failed`, `lifecycle.app_lock.window_unavailable`.

Los nombres son los `rawValue` canónicos de `DiagnosticCode`; el adaptador remoto no define aliases, traducciones ni códigos agregados alternativos. Los dos fallos de baja y los dos fallos de recuperación permanecen separados porque representan fronteras técnicas distintas y accionables.

Los demás códigos implementados permanecen únicamente en Unified Logging. No se envían éxitos, rechazos esperados, navegación, errores de validación de fotos, operaciones por elemento ni mensajes libres. Sentry tiene desactivados replay, screenshots, attachments, perfiles, performance/profiling, variables locales, breadcrumbs automáticos y captura automática de red. Las alertas iniciales se limitan a: crash nuevo, regresión por release, fallo de Vault/Keychain y ruptura repetida de contrato/traza.

### Cinco tableros iniciales

1. **Uso de superficies:** sesiones instrumentadas que visitan Home, lista de Crushes, Timeline y Estadísticas, con versión y estado resuelto.
2. **Acceso y alta:** conversión y abandono por método y paso, separando cancelación de fallo.
3. **Valor central:** creación y edición por entidad desde inicio hasta guardado local y sync; brecha `saved_locally → synced`.
4. **Timeline y Estadísticas:** visitas, filtros y aperturas del Timeline, y proporción `populated|empty|failed` de Estadísticas.
5. **Fotos:** sesiones que abren el selector y proporción de guardado completado o fallido.

No habrá dashboard por cada evento, usuarios únicos persistentes ni exploración individual. Antes de tomar decisiones, cada tablero debe mostrar versión, cobertura del rollout y opt-out y superar una línea base de calidad.

## Catálogo inicial

El nombre Swift puede expresarse con casos tipados; los nombres siguientes son el contrato lógico. Cada propiedad indicada es un enum cerrado.

| Dominio | Eventos | Dimensiones permitidas | Frontera |
|---|---|---|---|
| Navegación | `screen_viewed` | `screen`, `entry_point`, `content_state` | pantalla realmente visible una vez por visita |
| Login | `login_started`, `login_submitted`, `login_completed`, `login_failed`, `login_cancelled` | `method`, `step`, `error_category` | `completed` sólo con sesión MDL publicada |
| Registro | `registration_started`, `registration_step_viewed`, `registration_step_completed`, `registration_submitted`, `registration_completed`, `registration_failed`, `registration_cancelled` | `method`, `step`, `error_category` | email completa tras OTP y sesión; Apple tras alta atómica y sesión |
| Creación | `creation_started`, `creation_step_viewed`, `creation_submitted`, `creation_saved_locally`, `creation_synced`, `creation_failed`, `creation_cancelled` | `entity_type`, `step`, `error_category` | separa durable local de confirmación remota; taxonomía y correcciones en `RP-ANA-014` |
| Edición | `edit_started`, `edit_step_viewed`, `edit_submitted`, `edit_saved_locally`, `edit_synced`, `edit_failed`, `edit_cancelled` | `entity_type`, `step`, `error_category` | no adjunta entidad ni valores editados; taxonomía y correcciones en `RP-ANA-014` |
| Fotos | `photo_picker_opened`, `photo_selection_completed`, `photo_save_completed`, `photo_save_failed`, `photo_deleted` | `count_bucket`, `error_category` | biblioteca del sistema sin `source`; selección no vacía y guardado real con `one|two_to_four|five`; fallo sólo si no se guarda ninguna; borrado individual confirmado |
| Timeline | `timeline_viewed`, `timeline_filter_changed`, `timeline_item_opened` | `scope`, `content_state`, `filter`, `item_type` | `scope=global|crush`; `filter=all|crushes|experiences|relationships`; `item_type=crush|experience|relationship` sólo al navegar desde un elemento confirmado; sin acciones duplicadas, IDs, fechas ni contenido |
| Estadísticas | `statistics_viewed` | `content_state` | visita al seleccionar la pestaña y resolver su primer estado, sin duplicar por recarga, background ni `screen_viewed`; las secciones estáticas y la ayuda de color no emiten eventos |
| Preferencia | `analytics_preference_changed` | `enabled` | sólo puede emitirse al activar; al desactivar se aplica primero el corte y no se envía el evento |

El enum `screen` puede representar login, registro, Home, lista y perfil de Crush, alta/edición de Crush, alta/edición de experiencia, galería/fotos, Timeline, Estadísticas y Ajustes, pero la allowlist inicial sólo emite `screen_viewed` para Home y lista de Crushes. Timeline y Estadísticas conservan sus eventos especializados y los embudos explican el resto sin duplicar visitas.

## Métricas para decidir mejoras

| Pregunta | Métrica fiable | Uso |
|---|---|---|
| ¿Se utiliza una pantalla? | sesiones con `screen_viewed(screen)` y vistas por sesión instrumentada | distinguir funciones ignoradas de problemas de descubrimiento |
| ¿Dónde cae el alta? | paso alcanzado y conversión al siguiente paso dentro de la correlación efímera | localizar fricción en email, nacimiento, contraseña, legal, OTP o sesión |
| ¿Se completa una creación/edición? | `saved_locally / started`, `synced / saved_locally`, fallos por categoría | separar UX del formulario de conectividad/sync |
| ¿Se usan las fotos? | apertura de picker y resultado de guardado por sesiones | saber si la función se descubre y completa sin ampliar el dashboard inicial |
| ¿Timeline aporta uso? | visitas, alcance global/perfil, aperturas y acciones por tipo | mejorar jerarquía y acciones, no contenido |
| ¿Estadísticas aporta uso? | visitas y estados vacío/con datos/fallo | decidir si la superficie se descubre y explicar vacíos sin inventar aperturas de secciones estáticas |

No se usarán usuarios únicos persistentes, retención individual ni cohortes ligadas a cuenta. Las tasas se calculan sobre sesiones instrumentadas y siempre muestran versión, cobertura y opt-out para evitar falsas conclusiones.

## Escenarios de aceptación

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
**Cuando** SwiftUI recompone su árbol sin una nueva visita
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
**Entonces** la salida externa permanece desactivada aunque el contrato local esté implementado
**Y** una build fuera de TestFlight no presenta por defecto el aviso ni los controles de rollout y no persiste la analítica como habilitada.

**Dado** el modo técnico explícito fuera de TestFlight
**Cuando** se valida PostHog o Sentry
**Entonces** sólo se transmiten eventos sintéticos o de una cuenta descartable controlada mediante las mismas allowlists y reglas de minimización
**Y** cerrar ese modo devuelve la build a `NoOpAnalyticsTracker` y Unified Logging local.

### GWT-OBS-01 — Sin bypass del logger

**Dado** cualquier recorrido de producción iOS
**Cuando** genera información diagnóstica
**Entonces** pasa por el contrato estructurado y no usa `print`, `debugPrint`, `NSLog` ni un sink directo.

### GWT-OBS-02 — Correlación extremo a extremo

**Dado** una petición iniciada por iOS
**Cuando** API la procesa o la rechaza
**Entonces** iOS envía un UUID válido en `X-Request-Id`, API conserva ese valor como `traceId` y el cliente registra la referencia confirmada de `X-Trace-Id` o del envelope de error.

### GWT-OBS-03 — Diagnóstico útil y mínimo

**Dado** un fallo registrable
**Cuando** alcanza un sink local o remoto
**Entonces** tiene código estable, severidad, categoría, release/build, entorno y contexto allowlisted suficiente para buscarlo, sin mensaje libre, identificador de negocio ni contenido privado.

### GWT-OBS-04 — Producción sin ruido engañoso

**Dado** una build Release
**Cuando** emite logs
**Entonces** conserva warnings y errores accionables, admite sólo hitos `info` expresamente permitidos y excluye trazas de flujo, éxitos rutinarios y dumps de objetos.

### GWT-OBS-05 — Receptor remoto cerrado

**Dado** que la build no pertenece a la beta 0.2 de TestFlight, falta configuración válida, todavía no se mostró el aviso o el control de diagnóstico está desactivado
**Cuando** se inicia la app
**Entonces** iOS no inicia Sentry ni transmite diagnósticos o crashes remotos y conserva Unified Logging local.

## Orden y fases

Cada fase iOS se ejecuta y acepta en una sesión distinta de Claude Code. No se avanza automáticamente a la siguiente. Las fases `OBS` no convierten logs en analítica.

1. **`IOS-OBS-01` — Base segura y correlación.** Inventariar todo el corpus, crear diagnósticos estructurados, retirar bypasses críticos de Vault/red, corregir severidades y enlazar el `traceId` iOS↔API. Sigue sin receptor remoto.
2. **`IOS-OBS-02` — Migración del corpus.** Migrar por dominios cada llamada conservada, eliminar ruido y retirar la API de logs libres cuando no queden consumidores.
3. **`IOS-ANA-00` — Corrección y contrato base.** Sustituir strings libres por eventos tipados, retirar wrappers muertos, IDs y errores libres; mover rate limiting al contrato diagnóstico; corregir fronteras engañosas de registro/legal; introducir receptor `no-op`/spy y tests. No proveedor, Ajustes ni eventos nuevos de producto.
4. **`IOS-ANA-01` / `IOS-ANA-01R` — Runtime y preferencia cerrados.** Resolver preferencia antes de emitir, corte/limpieza y correlación efímera intercambiable con receptor externo deshabilitado. Mientras el rollout siga bloqueado, el aviso y el control sólo pueden ejercerse mediante un modo hermético explícito de UI tests; una build ordinaria conserva el estado no anunciado y no presenta ni persiste una medición inexistente.
5. **`IOS-ANA-02` — Login y registro.** Instrumentar email y Apple sobre fronteras reales de paso, submit, OTP, sesión, cancelación y categorías de fallo.
6. **`IOS-ANA-03` / `IOS-ANA-03R` — Creación y edición.** Aceptadas. Crush y experiencia separan envío, guardado durable y sync sin cambiar contratos funcionales; la taxonomía cerrada, los formularios de un paso y «Corregir» cumplen `RP-ANA-014`, incluida la resolución previa y fail-closed de la autodetección de correcciones.
7. **`IOS-ANA-04` — Fotos, Timeline y Estadísticas. Aceptada.** Fotos mide apertura, selección, guardado, fallo total y borrado individual confirmados sin IDs ni cantidades exactas. Timeline distingue la superficie mediante `scope`, el cambio humano del filtro global mediante `timeline_filter_changed` y la navegación desde un elemento confirmado mediante `timeline_item_opened`; no existen `timeline_scope_changed` ni `timeline_action_*`. Statistics sólo mide la visita resuelta: sus secciones estáticas y la ayuda de color no producen `statistics_section_opened`.
8. **`IOS-ANA-05` / `IOS-ANA-05R` — PostHog EU y tableros de beta. Aceptada.** El adaptador allowlisted quedó verificado contra un proyecto PostHog Cloud EU aislado con datos sintéticos: todos los eventos activos y sus propiedades cerradas son visibles, los tres eventos dormantes no aparecen, no se creó ninguna persona y los cinco tableros iniciales están configurados. Las builds ordinarias conservan `NoOpAnalyticsTracker`; aviso, Ajustes y datos reales siguen cerrados.
9. **`IOS-OBS-03` — Sentry EU y operación. Aceptada.** El SDK oficial y el sink con allowlist cerrada están integrados detrás de un gate hermético; el payload remoto, la redacción, la deduplicación por payload completo con memoria acotada y la ausencia de arranque ordinario están cubiertos por pruebas. El diagnóstico allowlisted y el crash sintético se verificaron en un proyecto Sentry EU aislado, sin usuario, breadcrumbs ni adjuntos; release, entorno y `traceId` permiten localizar el evento. Quedaron operativas las alertas de crash nuevo, regresión por release, fallo de Vault/Keychain y ruptura de contrato/traza al superar dos eventos en cinco minutos. La regresión completa con la API local sana terminó con 3342 pruebas, 3340 aprobadas, 2 sintéticas omitidas y 0 fallos. Las builds ordinarias conservan sólo Unified Logging local.
10. **`IOS-ROLLOUT-06` — Activación de la beta.** Comprobar que la retención efectiva no supera los 90 días de PostHog ni los 30 días de Sentry fijados en [Plazos de conservación y destrucción](../../../rules/DATA_LIFECYCLE.md#plazos); fijar la versión visible 0.2; completar Privacy Manifest y ficha App Store Privacy con los hechos técnicos vigentes; habilitar el aviso conjunto, el opt-out de analítica y el control propio del diagnóstico remoto, que cubre diagnósticos y crashes ([`RP-SEG-013`](../../../rules/PRIVACY.md#seguridad)); abrir ambos adaptadores en la beta 0.2 de TestFlight y mediante el modo técnico explícito de validación ([`RP-ANA-020`](../../../rules/ANALYTICS.md#activacion)); reparar y repetir los dos casos fallidos de `CrushListLayoutUITests`; y validar tráfico, dashboards y alertas antes de usar las métricas.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| Product | reglas, catálogo lógico, GWT, métricas y alcance autorizado de analítica/diagnóstico |
| iOS | eventos analíticos, diagnósticos estructurados, correlación HTTP, preferencias, proveedores y pruebas |
| API | sin cambios previstos; ya genera la traza y configura salidas hacia CloudWatch/Sentry, pero su funcionamiento productivo debe validarse extremo a extremo |
| Android/LLM | fuera de alcance; no se replica la instrumentación |

## Riesgos y bloqueos

- La salida ordinaria permanece cerrada hasta que `IOS-ROLLOUT-06` implemente el gate de la beta, el aviso y los dos controles. Las validaciones técnicas explícitas ya pueden usar los proyectos reales con datos controlados y las retenciones máximas de 90 días para PostHog y 30 días para Sentry.
- La revisión jurídica continúa para el lanzamiento público. Si exige una corrección, se reabre primero la regla de Producto afectada; no se amplía por anticipación el alcance aprobado de la beta.
- Los cambios simultáneos en Auth, Timeline, Vault y formularios obligan a implementar cada fase sobre el código vigente y preservar trabajo ajeno.
- Un evento técnicamente válido puede seguir siendo basura si su frontera no coincide con el resultado real; los tests deben comprobar la frontera, no sólo la llamada.
- SwiftUI puede duplicar apariciones; el ownership del evento pertenece a la pantalla/flujo, no a componentes reutilizables.
- Los dashboards no se consideran evidencia hasta validar versión, cobertura, duplicados y opt-out.

## Criterios de aceptación de la iniciativa

- Todos los GWT tienen evidencia automatizada iOS y, donde corresponda, validación manual.
- No quedan llamadas productivas con nombres o diccionarios libres.
- Ningún test permite IDs, contenido o errores libres como propiedades.
- Registro, guardado durable y sync usan fronteras verdaderas.
- Opt-out detiene y limpia la analítica sin degradar funciones.
- La telemetría técnica queda fuera del canal de producto.
- No quedan sinks directos ni severidades que desaparezcan de Release por una política accidental.
- Un error HTTP puede localizarse con el mismo `traceId` en iOS, respuesta API, CloudWatch y Sentry cuando corresponda.
- La documentación sólo promete visibilidad remota de errores iOS para la beta 0.2 de TestFlight después de completar `IOS-ROLLOUT-06`.
- Proveedor, región, retención, aviso y controles coinciden con las reglas aprobadas antes de habilitar salida externa.
- Las métricas se validan durante una línea base antes de priorizar producto.
- Al cerrar, el snapshot estable pasa a Product/iOS canónicos y ambos planes se eliminan coordinadamente.

## Plan local

- [iOS](../../../../../ios/docs/plans/active/ios-product-analytics/README.md)
