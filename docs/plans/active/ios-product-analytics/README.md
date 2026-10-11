# Analítica de producto y trazabilidad operativa iOS

> **Definición funcional vigente:** [Analítica de uso en iOS](../../../rules/ANALYTICS.md). La implementación puede avanzar con receptores locales, pero el envío productivo permanece bloqueado por `DEC-15`, `DEC-09` y revisión humana/legal.
>
> **Estado:** contratos de Producto definidos; inventarios de analítica y observabilidad revisados; corrección iOS pendiente.
> **Prioridad:** necesaria antes de usar datos para priorizar mejoras de la beta iOS.
> **Proyectos:** Product e iOS. API, Android y LLM quedan fuera.
> **Siguiente fase:** `IOS-OBS-01`.

## Objetivo

Obtener métricas fiables y agregadas sobre qué pantallas y recorridos utiliza la gente en iOS, y dejar además una trazabilidad técnica que permita encontrar fallos reales desde el dispositivo hasta API, CloudWatch y el receptor de errores aprobado. Analítica y diagnóstico permanecen separados y ninguno mide contenido privado.

## Baseline confirmado

- iOS contiene 19 llamadas a `AnalyticsManager.track` con 14 nombres de evento, pero `AnalyticsManager` sólo escribe en `Logger`; no existe proveedor de analítica ni pruebas específicas del contrato.
- Tres wrappers de Home no tienen consumidores: `home_view_appeared`, `home_crush_tapped` y `home_experience_tapped` no demuestran uso real.
- Cuatro sitios envían `error.localizedDescription`; dos adjuntan identificadores de entidades. Esos datos no pueden reutilizarse.
- `register_success` ocurre antes de OTP y de la sesión autenticada, por lo que no significa registro completado.
- `legal_terms_accepted` mezcla una aceptación confirmada por servidor con el mero avance desde el paso legal. `legal_modal_shown` se emite antes de que el contenido esté visible.
- `api_rate_limit_exceeded` es observabilidad técnica, no analítica de producto.
- `statistics_screen_viewed` expresa una intención útil, pero necesita una frontera de visita que no duplique recomposiciones y dimensiones cerradas de estado de contenido.
- La dependencia Firebase retirada sigue protegida por tests; esta iniciativa no autoriza a reintroducirla ni presupone proveedor.

### Observabilidad y recorrido actual de los logs

| Origen | Destino real actual | Hallazgo |
|---|---|---|
| `Logger` iOS | Unified Logging (`os.Logger`) del dispositivo, por subsystem y categoría | no existe transporte remoto; en Release admite `.info` y `.error`, pero descarta `.warning` y no distingue cuáles `.info` son vitales |
| `print` iOS | stdout/consola local | 13 llamadas en siete símbolos de Vault saltan política, categorías y redacción; varias incluyen IDs o `Error` crudo |
| Amplify iOS | logging local del SDK a nivel `.error` | se usa únicamente Cognito; no envía los logs de la app a CloudWatch |
| Sentry/Crashlytics iOS | inexistente | un fallo sólo del cliente no puede investigarse remotamente con el código actual |
| Laravel API | `stderr` → Docker `awslogs` | producción envía Nginx, PHP-FPM y Laravel combinados a `/ec2/mdl-api/prod`, stream `app-container` |
| Sentry API | reporter allowlisted para excepciones JSON no manejadas | no recibe todos los logs; `sentry_logs` existe como canal, pero producción usa `LOG_CHANNEL=stderr` |
| Correlación HTTP | API acepta `X-Request-Id`, responde `X-Trace-Id` y añade `traceId` al error | iOS crea un UUID local pero no lo envía ni consume la cabecera de respuesta; la correlación extremo a extremo está rota |
| Operación CloudWatch | configuración del driver en Compose | el repositorio no demuestra retención, filtros métricos, alarmas ni separación de streams; deben validarse en AWS sin leer secretos |

El grafo identifica 295 apariciones textuales de `Logger.log` en `MDLApp` —incluidos los ejemplos del propio logger— y un uso desigual de categorías: la mayoría cae en `General`. Los tests existentes cubren redacción de email/tokens y algunos recorridos sensibles, pero no demuestran utilidad, severidad, destino remoto ni correlación con API.

## Alcance

- Corregir o retirar los eventos actuales ambiguos, muertos o inseguros.
- Crear un contrato tipado, inyectable y comprobable para eventos y propiedades cerradas.
- Incorporar aviso claro y un control de opt-out en Ajustes.
- Medir login y registro por email y Apple, sin credenciales ni identificadores.
- Medir alta y edición de Crush/experiencia y sus resultados durable/sync.
- Medir uso funcional de fotos locales, Timeline y Estadísticas.
- Definir métricas y dashboards sólo después de verificar la calidad de instrumentación.
- Seleccionar e integrar un proveedor únicamente tras aprobar `DEC-15` y `DEC-09`.
- Revisar cada llamada de log iOS y decidir si se conserva, elimina, rebaja a debug o se convierte en un diagnóstico estructurado.
- Retirar `print`/`debugPrint`/`NSLog` directos del runtime y unificar severidad, categoría, código y metadata allowlisted.
- Propagar el mismo `traceId` desde iOS hasta API y recuperarlo de toda respuesta para buscar el incidente en CloudWatch/Sentry.
- Decidir y, sólo tras `DEC-16`, integrar un receptor remoto para crashes y errores iOS seleccionados.

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

Desactivar analítica no elimina la capacidad técnica mínima de investigar errores. La eventual activación remota de diagnóstico tiene su propio gate `DEC-16`, disclosure, retención y configuración; no se esconde dentro del consentimiento analítico.

## Catálogo inicial

El nombre Swift puede expresarse con casos tipados; los nombres siguientes son el contrato lógico. Cada propiedad indicada es un enum cerrado.

| Dominio | Eventos | Dimensiones permitidas | Frontera |
|---|---|---|---|
| Navegación | `screen_viewed` | `screen`, `entry_point`, `content_state` | pantalla realmente visible una vez por visita |
| Login | `login_started`, `login_submitted`, `login_completed`, `login_failed`, `login_cancelled` | `method`, `step`, `error_category` | `completed` sólo con sesión MDL publicada |
| Registro | `registration_started`, `registration_step_viewed`, `registration_step_completed`, `registration_submitted`, `registration_completed`, `registration_failed`, `registration_cancelled` | `method`, `step`, `error_category` | email completa tras OTP y sesión; Apple tras alta atómica y sesión |
| Creación | `creation_started`, `creation_step_viewed`, `creation_submitted`, `creation_saved_locally`, `creation_synced`, `creation_failed`, `creation_cancelled` | `entity_type`, `step`, `error_category` | separa durable local de confirmación remota |
| Edición | `edit_started`, `edit_step_viewed`, `edit_submitted`, `edit_saved_locally`, `edit_synced`, `edit_failed`, `edit_cancelled` | `entity_type`, `step`, `error_category` | no adjunta entidad ni valores editados |
| Fotos | `photo_picker_opened`, `photo_selection_completed`, `photo_save_completed`, `photo_save_failed`, `photo_deleted` | `source`, `count_bucket`, `error_category` | resultado real del picker/guardado/borrado local |
| Timeline | `timeline_viewed`, `timeline_scope_changed`, `timeline_item_opened`, `timeline_action_started`, `timeline_action_completed`, `timeline_action_failed` | `scope`, `content_state`, `item_type`, `action`, `error_category` | nunca ID, fecha ni contenido del item |
| Estadísticas | `statistics_viewed`, `statistics_section_opened` | `content_state`, `section` | pantalla/sección visible, sin valor estadístico |
| Preferencia | `analytics_preference_changed` | `enabled` | sólo puede emitirse al activar; al desactivar se aplica primero el corte y no se envía el evento |

`screen` cubre como mínimo las entradas activas de login, registro, Home, lista y perfil de Crush, alta/edición de Crush, alta/edición de experiencia, galería/fotos, Timeline, Estadísticas y Ajustes. El inventario definitivo se contrasta con `frontend-screen-rules` antes de instrumentar cada bloque.

## Métricas para decidir mejoras

| Pregunta | Métrica fiable | Uso |
|---|---|---|
| ¿Se utiliza una pantalla? | sesiones con `screen_viewed(screen)` y vistas por sesión instrumentada | distinguir funciones ignoradas de problemas de descubrimiento |
| ¿Dónde cae el alta? | paso alcanzado y conversión al siguiente paso dentro de la correlación efímera | localizar fricción en email, nacimiento, contraseña, legal, OTP o sesión |
| ¿Se completa una creación/edición? | `saved_locally / started`, `synced / saved_locally`, fallos por categoría | separar UX del formulario de conectividad/sync |
| ¿Se usan las fotos? | apertura de picker, selección, guardado y borrado por sesiones | saber si la función se descubre y completa |
| ¿Timeline aporta uso? | visitas, alcance global/perfil, aperturas y acciones por tipo | mejorar jerarquía y acciones, no contenido |
| ¿Estadísticas aporta uso? | visitas, estados vacío/con datos y secciones abiertas | priorizar secciones y explicar vacíos |

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

### GWT-ANA-05 — Visita sin duplicados

**Dado** que una pantalla ya está visible
**Cuando** SwiftUI recompone su árbol sin una nueva visita
**Entonces** no duplica `screen_viewed`.

### GWT-ANA-06 — Fallo no identificable

**Dado** un error de autenticación, red, Vault o persistencia
**Cuando** se mide el resultado
**Entonces** se transforma a una categoría cerrada y no se envían texto libre, ruta concreta, payload ni identificadores.

### GWT-ANA-07 — Analítica no bloqueante

**Dado** un receptor ausente, apagado o fallido
**Cuando** la persona ejecuta una función de MDL
**Entonces** el resultado funcional no cambia y no se presenta un error de analítica.

### GWT-ANA-08 — Rollout cerrado

**Dado** que falta cualquiera de `DEC-15`, `DEC-09`, disclosure o configuración aprobada
**Cuando** se genera una build de producción
**Entonces** la salida externa permanece desactivada aunque el contrato local esté implementado.

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

**Dado** que `DEC-16` o su configuración aprobada están pendientes
**Cuando** se distribuye una build
**Entonces** iOS no transmite diagnósticos remotos y no afirma que sus fallos cliente puedan verse en Sentry o CloudWatch.

## Orden y fases

Cada fase iOS se ejecuta y acepta en una sesión distinta de Claude Code. No se avanza automáticamente a la siguiente. Las fases `OBS` no convierten logs en analítica.

1. **`IOS-OBS-01` — Base segura y correlación.** Inventariar todo el corpus, crear diagnósticos estructurados, retirar bypasses críticos de Vault/red, corregir severidades y enlazar el `traceId` iOS↔API. Sigue sin receptor remoto.
2. **`IOS-OBS-02` — Migración del corpus.** Migrar por dominios cada llamada conservada, eliminar ruido y retirar la API de logs libres cuando no queden consumidores.
3. **`IOS-ANA-00` — Corrección y contrato base.** Sustituir strings libres por eventos tipados, retirar wrappers muertos, IDs y errores libres; mover rate limiting al contrato diagnóstico; corregir fronteras engañosas de registro/legal; introducir receptor `no-op`/spy y tests. No proveedor, Ajustes ni eventos nuevos de producto.
4. **`IOS-ANA-01` — Runtime y preferencia.** Resolver preferencia antes de emitir, aviso inicial, opt-out en Ajustes, corte/limpieza, correlación efímera intercambiable y receptor externo todavía deshabilitado en producción.
5. **`IOS-ANA-02` — Login y registro.** Instrumentar email y Apple sobre fronteras reales de paso, submit, OTP, sesión, cancelación y categorías de fallo.
6. **`IOS-ANA-03` — Creación y edición.** Instrumentar Crush y experiencia, incluida separación entre guardado durable y sync, sin cambiar contratos funcionales.
7. **`IOS-ANA-04` — Fotos, Timeline y Estadísticas.** Medir descubrimiento, resultado y uso de estas superficies con dimensiones cerradas y pruebas anti-duplicado.
8. **`IOS-OBS-03` — Diagnóstico remoto y operación.** Sólo después de `DEC-16`: integrar el receptor de errores, validar payloads, releases, alertas, acceso, retención y una prueba sintética correlacionada con API/CloudWatch.
9. **`IOS-ANA-05` — Proveedor, disclosure y rollout.** Sólo después de aprobar `DEC-15`/`DEC-09`: integrar el proveedor, validar manifest/ficha App Store Privacy, residencia/retención/borrado/opt-out, dashboards y una línea base controlada antes de usar métricas para decisiones.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| Product | reglas, catálogo lógico, GWT, métricas y gates Legal de analítica/diagnóstico |
| iOS | eventos analíticos, diagnósticos estructurados, correlación HTTP, preferencias, proveedores y pruebas |
| API | sin cambios previstos; ya genera la traza y configura salidas hacia CloudWatch/Sentry, pero su funcionamiento productivo debe validarse extremo a extremo |
| Android/LLM | fuera de alcance; no se replica la instrumentación |

## Riesgos y bloqueos

- `DEC-15` y `DEC-09` bloquean exclusivamente la integración/activación productiva del proveedor analítico, no `IOS-ANA-00..04` con receptores locales.
- `DEC-16` bloquea el receptor remoto de errores iOS, no el saneamiento local ni la correlación `IOS-OBS-01`.
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
- Antes de `DEC-16`, la documentación no promete visibilidad remota de errores exclusivos del cliente.
- Proveedor, retención, disclosure y App Store Privacy están aprobados antes de habilitar salida externa.
- Las métricas se validan durante una línea base antes de priorizar producto.
- Al cerrar, el snapshot estable pasa a Product/iOS canónicos y ambos planes se eliminan coordinadamente.

## Plan local

- [iOS](../../../../../ios/docs/plans/active/ios-product-analytics/README.md)

## Decisión abierta

1. **`DEC-15` — proveedor y gobierno de datos:** se resolverá con Legal antes de `IOS-ANA-05`; no debe adelantarse eligiendo SDK, identificador o retención desde iOS.
2. **`DEC-16` — diagnóstico remoto iOS:** decidir proveedor —Sentry iOS u otro—, sampling, retención, acceso, alertas y disclosure antes de `IOS-OBS-03`; no autoriza replay, autocaptura ni profiling.
