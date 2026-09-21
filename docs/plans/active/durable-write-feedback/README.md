# Cambios pendientes visibles en beta

> **Autoridad funcional:** las reglas completas están en [Guardado y sincronización](../../../rules/SYNC.md); las relacionales en [Relaciones](../../../rules/RELATIONSHIPS.md). Los escenarios de este plan referencian ese catálogo. No se conserva compatibilidad runtime anterior al lanzamiento ni una matriz alternativa de ocho estados.

> **Estado:** activo. La base durable, retry, proyección y contención se conservan, pero los escenarios de Experience inicial opcional, Relationship standalone y borrado que desasocia han quedado supersedidos. `API-CRU-RED-03/03` e `IOS-CRU-02` corrigen el bundle inicial; `API-RR-N2..N3` e `IOS-RR-N2..N4` adaptan los cambios relacionales posteriores y su borrado al compuesto obligatorio antes de retomar las validaciones físicas y el cierre de esta iniciativa. `IOS-PENDING-05` continúa pendiente para la carrera de presentación del badge agregado.
> **Proyectos:** iOS y API.

> **API-PENDING-04 — evidencia histórica, no criterio de salida:** la implementación y sus pruebas automatizadas demostraron unicidad y correlación bajo el contrato anterior, incluido el vínculo automático y la desvinculación. No se despliega ni se valida físicamente como contrato final; `API-RR-N2..N3` debe reemplazar esas semánticas por la pertenencia obligatoria de toda Experience a un período y el borrado compuesto vigente.

> **IOS-PENDING-04 — evidencia histórica, no criterio de salida:** la implementación y las pruebas automatizadas de sus guards se conservan como evidencia del pipeline, pero el vínculo automático por fecha y el borrado por desvinculación quedan supersedidos. `IOS-RR-N2..N4` los sustituye por la pertenencia explícita al período y las acciones de borrado vigentes contra `API-RR-N2..N3`.

## Objetivo

Cuando una persona pulsa guardar, MDL distingue inmediatamente entre guardado local, sincronización en curso, intervención requerida y confirmación remota. Cada crush admite como máximo una operación pendiente por dominio de conflicto; profile y Timeline no se confunden y una pendiente nunca contamina otro crush.

## Alcance

- Altas y ediciones durables de crush, experiencia y relación.
- Proyección local cifrada en Lista y Timeline.
- Estado visible, retry limitado, corrección y descarte seguro.
- Bloqueos sólo por crush y dominio realmente conflictivo.
- Archivo y borrado interactuables aunque existan pendientes, conforme a certeza remota.
- Fallo seguro ante snapshots incompatibles; sin compatibilidad runtime anterior al contrato final.

Fuera de alcance:

- varias operaciones nuevas en cola dentro del mismo dominio/crush;
- borradores generales;
- pantalla global de sincronización;
- ejecución permanente con la app cerrada;
- resolución de conflictos multi-dispositivo;
- rediseñar el contrato estable de cifrado, recovery o borrado dirigido del Vault.

## Dominios de conflicto

- **Root:** el crush todavía no existe confirmado. Bloquea navegación, profile, descendientes y archivo de ese crush hasta sincronizar o descartarse de forma segura.
- **Profile:** edición del perfil, incluido `met_at`. Bloquea otra edición de profile del mismo crush. Sólo cuando la intención cambia realmente `met_at`, bloquea además el alta de una experiencia nueva del mismo crush hasta resolverse; no bloquea por eso las demás acciones.
- **Timeline:** alta/edición de experiencia o relación. Bloquea otra escritura de Timeline y el borrado de relación conflictivo, pero no profile, fotos, archivar/restaurar, borrar experiencia confirmada ni otros crushes.

Las reservas se identifican por UUID canónico de crush. Una operación sin scope derivable no reserva todo el Vault.

## Escenarios de producto — Dado / Cuando / Entonces

Esta matriz es el contrato ejecutable de aceptación de la iniciativa mientras permanezca activa; no reemplaza las reglas estables de [Sincronización](../../../rules/SYNC.md) y [Timeline](../../../rules/TIMELINE.md).

- GWT-01 a GWT-14 y GWT-31 a GWT-42 ejercitan reglas de dominio/Timeline que **ya viven de forma permanente** en esos documentos canónicos. Se conservan aquí temporalmente porque API e iOS todavía tienen fases abiertas que deben demostrarlas de extremo a extremo.
- GWT-15 a GWT-30 describen la mecánica específica de guardado durable, proyección, retry, resolución y compatibilidad legacy que esta iniciativa está terminando.
- Al cerrar la iniciativa, el comportamiento durable final se integra en `rules/SYNC.md` y el detalle técnico en `ARCHITECTURE.md`; este plan y los locales se eliminan. Las reglas de Timeline no se “pierden” ni dependen de conservar el plan.

### Formularios y cronología

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-01 | Un alta de Crush carece de lugar, descripción o satisfacción inicial | La persona intenta guardar | Los campos inválidos se marcan inline; no hay snapshot, operación, proyección ni request. |
| GWT-02 | El alta de Crush contiene todos los datos y la Experience inicial completa | La persona guarda | Se crea una única intención durable con Crush, período, hito inicial y Experience inicial correlacionados en `met_at`. |
| GWT-03 | La intención inicial se reintenta o corrige | Se ejecuta de nuevo | Conserva las identidades de Crush, período, hito inicial y Experience y nunca crea un bloque parcial o duplicado. |
| GWT-04 | Un Crush confirmado con período vivo y un día válido sin otra Experience ni hito | La persona crea una Experience sin cambio relacional | Aparece una sola Experience independiente, perteneciente a ese período y sin hito, en ese día. |
| GWT-05 | Ya existe la Experience inicial obligatoria | Se intenta crear o mover otra Experience a `met_at` | Se rechaza por unicidad diaria sin mutar la candidata ni el bundle inicial. |
| GWT-06 | El día de `met_at` ya contiene la experiencia inicial | Se intenta crear o mover otra experiencia a ese día | Se rechaza por unicidad diaria sin mutar la candidata. |
| GWT-07 | Una experiencia está vinculada a un hito | Se intenta cambiar sólo su fecha | La UI mantiene la fecha bloqueada y API rechaza cualquier drift; hito y experiencia conservan exactamente la misma fecha. |
| GWT-08 | Una Experience confirmada | La persona elige `Eliminar` | Si es independiente, se elimina sola sin abandonar su período; si está vinculada al último hito posterior eliminable, se eliminan ambas partes; si es la inicial del período no inicial más reciente y su hito raíz sigue siendo también el último hito, se elimina ese período con todas sus Experiences; si pertenece al bloque inicial, ya es histórica o existe otro hito posterior, no se ofrece o se rechaza sin mutación. |
| GWT-09 | Una Experience independiente ya pertenece a un período | La persona abre sus acciones | Puede editarla o eliminarla según sus reglas, pero no se ofrece iniciar otro período desde ella ni se la reasigna por coincidencia de fechas. Un período nuevo nace siempre con su propia Experience y su hito inicial en la misma acción atómica. |
| GWT-10 | La persona inicia directamente «Cambiar situación» | Completa Situación y después Experience | Se persiste una única intención durable compuesta; nunca existe un hito pendiente o confirmado sin Experience. |
| GWT-11 | Existe una historia con varios períodos o hitos | La persona abre las acciones de un elemento histórico | Editar y eliminar no se muestran. Sólo el último elemento elegible del período más reciente ofrece las acciones que corresponden. |
| GWT-12 | Existe un último hito posterior elegible | Se confirma «Eliminar cambio» | Aplica RP-HIT-003/008/009: restaura el hito anterior del mismo período y conserva la Experience independiente. «Eliminar cambio y experiencia» se cubre por separado en GWT-42. Borrar un período completo sigue RP-HIT-010 y nunca reabre otro anterior. |
| GWT-13 | `met_at` se edita dentro del rango permitido por eventos confirmados y pendientes del mismo crush | La persona guarda | Es una operación de profile; mueve el bundle inicial junto y no queda bloqueada por una pendiente de Timeline. |
| GWT-14 | La fecha propuesta de `met_at` coincidiría con la fecha del primer acontecimiento independiente del mismo Crush o quedaría después | La persona usa el calendario o intenta guardar | La UI limita la selección al día anterior a ese acontecimiento y rechaza cualquier fecha igual o posterior antes de persistir; si un cliente desactualizado recibe rechazo API, muestra el error y refresca sin retry automático de validación. |

### Persistencia, proyección y resolución

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-15 | Un formulario válido | La intención cifrada queda persistida localmente | El formulario comunica `Guardado en este dispositivo`, se cierra y aparece una proyección; todavía no se muestra `Sincronizado`. |
| GWT-16 | Un alta de crush aún no existe confirmada remotamente | Se muestra su proyección root | La fila está atenuada, no abre perfil ni admite descendientes o archivo; abre su estado y sólo permite retry/descarte cuando sea seguro. |
| GWT-17 | Hay una operación Timeline pendiente para A | Se navega por A, B y el Timeline global | A muestra esa operación una vez; B nunca la muestra ni queda bloqueado; el global la muestra una sola vez. |
| GWT-18 | Una edición pendiente corresponde a una entidad ya confirmada | Se fusionan API y proyección | Se conserva una sola tarjeta con el mismo ID, valores y fecha propuestos. Descartar restaura la versión confirmada. |
| GWT-19 | Hay una pendiente de Timeline para un Crush | Se intenta otra alta/edición de Experience o relación, o borrar una relación del mismo Crush | La acción incompatible se deshabilita con explicación. Perfil, `met_at` válido, fotos, navegación, archivo, borrar una Experience independiente no afectada y otros Crushes siguen operativos; una Experience vinculada espera si la pendiente puede cambiar su elegibilidad latest-only. |
| GWT-20 | Hay una pendiente de profile que no modifica `met_at` | Se intenta usar Timeline | Timeline sigue operativo; sólo otra edición de profile del mismo crush queda reservada. |
| GWT-21 | Una operación no tiene red antes del primer envío | La app permanece offline | No consume intentos ni genera tráfico repetido; muestra `Pendiente de conexión` y permite eliminar el cambio local. |
| GWT-22 | Un fallo transitorio produce timeout, `5xx` o `429` | Se ejecuta la política automática | Se reutilizan IDs y, al agotarse tres intentos del ciclo con backoff, queda `No se pudo sincronizar` sin revivir en foreground. |
| GWT-23 | API devuelve un rechazo determinista sin mutación | Se clasifica la respuesta | Pasa directamente a `Necesita revisión`, conserva razón estructurada localizada y no entra en loop automático. |
| GWT-24 | Una operación en revisión es atribuible y segura | La persona abre su badge/tarjeta | La hoja exacta por `operationId` ofrece la combinación real de corregir, retry manual, eliminar/descartar y cerrar; nunca sólo una promesa futura. |
| GWT-25 | Una operación nunca enviada o rechazada con certeza de no mutación | La persona confirma eliminar/descartar | Se elimina la intención completa y su proyección; un alta desaparece y una edición vuelve a la entidad confirmada. |
| GWT-26 | Existe timeout ambiguo o etapa `apiAccepted+` | La persona quiere eliminar el cambio | No se finge descarte local. Primero se reconcilia; si la entidad existe, se utiliza después el borrado funcional normal. |
| GWT-27 | Un crush confirmado tiene pendiente de Timeline | La persona lo archiva | Se ejecuta el PATCH real. Tras éxito sale de activos y Archivados conserva badge/hoja hasta resolver la pendiente. |
| GWT-28 | La persona pulsa eliminar un crush con pendientes | Se evalúa la certeza de todas ellas | Si todas son descartables, una confirmación las elimina y continúa el borrado normal. Si alguna tiene resultado incierto, la aplicación lo comprueba antes de continuar; mientras no pueda confirmarlo, no elimina el Crush ni su copia local. |
| GWT-29 | Hay una legacy sin scope derivable | La app normaliza o reanuda | Queda en revisión sin crush inventado, no se ejecuta automáticamente y no bloquea ningún alta independiente. |
| GWT-30 | Una versión antigua dejó varias operaciones del mismo crush y dominio | La app reanuda | Sólo la cabecera estable puede avanzar; las posteriores nunca corren en paralelo ni se promocionan automáticamente y sólo se descartan si es seguro. |

### Presentación del Timeline

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-31 | API devuelve Crush, período inicial, hito inicial y Experience obligatoria de igual correlación/fecha | Se reconstruye Timeline | Se muestra una sola tarjeta del bloque inicial con las cuatro partes. |
| GWT-32 | El bundle inicial confirmado o pendiente se proyecta | Se refresca Timeline | Conserva siempre la misma Experience inicial; nunca aparece incompleto ni como tarjeta separada. |
| GWT-33 | Una fecha válida llega como ISO 8601 o `yyyy-MM-dd` | Se renderiza una tarjeta confirmada o pendiente | Ambas formas muestran el mismo día civil; `-` se reserva para datos realmente corruptos. |
| GWT-34 | Un hito y una experiencia están vinculados directamente | Se reconstruye o edita Timeline | Comparten fecha exacta e identidad de vínculo; la UI no presenta como vínculo una propuesta local rechazada. |
| GWT-35 | Dos eventos empatan en fecha y momento de creación a través de una página | Se pagina/refresca | El orden secundario/terciario es determinista y no duplica, pierde ni cambia elementos entre páginas. |
| GWT-36 | Un Crush conserva únicamente el hito inicial de su período inicial | La persona abre sus acciones | Puede corregir la situación inicial permitida manteniendo la fecha `met_at`, pero Eliminar no aparece y API rechaza cualquier intento directo de borrado sin mutar el Crush. |
| GWT-37 | Ya existe una experiencia confirmada o pendiente de un crush en el día civil `D` | Se intenta crear o mover otra experiencia a `D` | La interfaz pide editar la existente y no persiste otra intención; una petición directa/desactualizada recibe rechazo sin mutar. |
| GWT-38 | Ya existe un hito confirmado o pendiente de un Crush en el día civil `D` | Se intenta crear otro hito en `D` | La interfaz pide corregir el último hito elegible y no persiste otra intención; una petición directa o desactualizada recibe rechazo sin mutar. El hito inicial y la Experience inicial comparten `D` porque forman la misma acción del bloque inicial. |
| GWT-39 | [Relaciones](../../../rules/RELATIONSHIPS.md#trayectos) define las situaciones y trayectos lógicos | iOS registra o corrige una relación | Cliente y API aplican RP-TRA-002..007 y RP-HIT-005 según la intención. El alta inicial describe la situación existente sin fabricar historia; una corrección conserva su fecha. |
| GWT-40 | Una edición de profile pendiente modifica realmente `met_at` de un crush confirmado | La persona intenta crear una experiencia nueva de ese mismo crush | `Agregar experiencia` no inicia formulario ni persiste otra intención y explica que debe terminar la actualización de la fecha. Al sincronizar, corregir o descartar la edición, el alta vuelve a estar disponible. No se calcula un rango hipotético entre el `met_at` confirmado y el propuesto; otros crushes y acciones no quedan redefinidos por este escenario. |
| GWT-41 | Se intenta crear o cambiar una relación sin Experience | Se persiste o ejecuta la intención | Cliente y API la rechazan sin operación parcial, hito ni actividad. |
| GWT-42 | Un hito ya tiene su Experience vinculada | Se intenta asociar otra o se elige una de sus acciones de borrado | El segundo vínculo se rechaza y no existe `Desvincular`. En un hito posterior, «Eliminar cambio» conserva la Experience independiente dentro del mismo período; «Eliminar cambio y experiencia» y el borrado iniciado desde la Experience aplican el mismo borrado compuesto latest-only. El hito inicial sólo desaparece al borrar su período completo o, en el bloque inicial, el Crush. |
| GWT-43 | Una creación confirmada de Crush, Experience o Relationship posterior | La persona abre Home con el Vault desbloqueado | Home muestra la actividad reciente correspondiente a la referencia estructural persistida y resuelve sus etiquetas privadas desde Vault. La relación inicial no duplica el alta del Crush; editar una Relationship existente —incluso su estado— y las demás ediciones o borrados no generan actividad. |
| GWT-44 | Un badge agregado sigue visible mientras su operación termina o acaba de ser retirada | La persona lo toca durante esa transición | La app abre contenido coherente para el `operationId` capturado o no presenta la hoja; nunca muestra una hoja vacía. Al completar API, persistencia local, upload y verificación, el badge desaparece tras reconciliar la fuente confirmada. |
| GWT-45 | Hay un período vivo | La persona inicia «Añadir experiencia» | El snapshot contiene primero la Experience; sólo si activa «¿Hay un cambio de situación?» añade el paso y los datos del hito al mismo compuesto. |
| GWT-46 | El período más reciente está cerrado | La persona inicia «Añadir experiencia» | El snapshot exige Experience y situación inicial en ese orden visual y conserva también las identidades del período e hito nuevos. |
| GWT-47 | El hito inicial del período no inicial más reciente sigue siendo también el último hito y existen Experiences independientes posteriores | Se confirma su borrado compuesto después de la advertencia | Una sola intención retira el período, su hito raíz, la Experience vinculada y todas las Experiences independientes; el período anterior permanece cerrado. Otro hito posterior vuelve la acción inelegible, pero una Experience independiente posterior no. |
| GWT-48 | Una Experience independiente confirmada puede moverse a un día libre del mismo período, incluso atravesando hitos | Se guarda la nueva fecha | La intención conserva su identidad y período, recalcula el contexto relacional y no crea vínculos; un día ocupado por la Experience de un hito se rechaza por unicidad diaria. |
| GWT-49 | Un Crush ya ocupa el día civil `D` con una Experience confirmada o pendiente | Se crea o mueve una Experience de otro Crush a `D` | La fecha continúa disponible para el segundo Crush y ambas intenciones se conservan; los bloqueos de fecha confirmados y pendientes se calculan siempre dentro del Crush correspondiente. |

## Estados visibles y retry

| Estado | Significado | Comportamiento |
|---|---|---|
| Guardado en dispositivo | Intención cifrada persistida | Formulario puede cerrar; no equivale a sync. |
| Sincronizando | Trabajo elegible/en curso | Mostrar progreso, sin cancelación insegura. |
| Pendiente de conexión | Nunca enviada y sin red | Cero polling; descarte local permitido. |
| No se pudo sincronizar | Ciclo transitorio agotado | Retry automático detenido; retry manual inicia ciclo nuevo. |
| Necesita revisión | Rechazo determinista o legacy problemática | Motivo localizado y capacidades exactas. |
| Esperando sesión | Falta autenticación válida | Recuperar sesión; no retry ciego. |
| Sincronizado | API + Vault completados y verificados | Badge de confirmación opcional; sin pendiente. |

Política beta:

- sin red conocida no consume intentos;
- timeout/`5xx`/`429`: un intento inicial y dos retries, con backoff aproximado de 5 y 30 segundos mientras la app está activa;
- `Retry-After` se respeta desde el contrato disponible;
- `422`, `409`, `403/404` deterministas pasan directamente a revisión;
- `401` espera sesión;
- foreground/relaunch no reinicia contadores agotados;
- retry manual conserva operación, snapshot e IDs;
- una operación con evidencia `apiAccepted+` termina reconciliación/finalización antes de ofrecer borrado normal.

## Hoja de estado

La hoja se abre por `operationId` exacto y no expone texto libre del backend.

| Situación | Acciones |
|---|---|
| Guardando o `apiAccepted+` | Estado y Cerrar. |
| Sin conexión y nunca enviada | Eliminar cambio, Cerrar y retry cuando haya red. |
| Retry agotado/resultado ambiguo | Reintentar ahora y Cerrar; no descartar. |
| Rechazo determinista seguro | Corregir, Reintentar ahora cuando sirva, Eliminar cambio y Cerrar. |
| Sesión inválida | Recuperar sesión y Cerrar. |
| Legacy no atribuible | Explicación; sin crush/formulario/retry inventados. Descarte sólo con certeza local-only. |

`Corregir` reabre el formulario desde el snapshot cifrado y reemplaza la intención atómicamente, conservando IDs. `Eliminar cambio` requiere confirmación y descarta toda la intención compuesta, no una mitad.

## Archivar y borrar

- Un crush confirmado se archiva con pendiente Timeline; Archivados conserva su estado/resolución.
- Un root provisional no se archiva.
- Borrar un Crush siempre produce un resultado comprensible. Si todas sus pendientes son descartables, las elimina y continúa el borrado normal. Si alguna tiene resultado incierto, la aplicación lo comprueba antes de continuar y conserva el Crush mientras no pueda confirmarlo.
- Un root provisional se puede eliminar sólo con certeza de que no existe remoto.
- Una experiencia independiente se puede borrar sin salir de su período. Una vinculada a un hito posterior sólo se elimina junto con ese último hito elegible. Si es la inicial del período no inicial más reciente, el período completo sólo se elimina cuando ese hito raíz sigue siendo también el último hito; las Experiences independientes posteriores se incluyen y no bloquean la acción. La Experience del bloque inicial y las vinculadas a historia anterior no ofrecen borrado parcial.
- Una experiencia pendiente local-only/rechazada se descarta. Una ambigua/aceptada se reconcilia y luego usa borrado funcional.
- Borrar un hito posterior o un período completo mantiene las reglas latest-only y los bloqueos cronológicos correspondientes.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| API | Idempotencia, aislamiento, transacciones y errores que distinguen rechazo seguro de resultado ambiguo. |
| iOS | Persistencia cifrada, scope/dominio/disposición, proyección, merge, retry y acciones seguras. |

## Estado y orden restante

1. Implementar `IOS-PENDING-05`: presentación atómica de la hoja desde el badge agregado de Lista, fijada a una proyección/`operationId` capturados; si ya no existe proyección al tocar, no se presenta nada. No modifica etapas, retry, Vault ni API.
2. El resto de la mecánica durable está implementado: `IOS-PENDING-03` deja Corregir disponible para los seis tipos de operación y `API-PENDING-06` completa GWT-35 con el orden total `event_date DESC, created_at DESC, item_type ASC, item_id ASC`. La semántica de vínculo por fecha de `IOS-PENDING-04` queda supersedida y se sustituye en `API-RR-N2..N3`/`IOS-RR-N2..N4` por asociaciones explícitas a período e hito.
3. Completar en el entorno desplegado la validación de `API-PENDING-02/02-R`: reasociación a `met_at` con slot libre/ocupado/vinculada y concurrencia real del lock. `API-PENDING-03` ya valida la exclusión de la relación inicial en ese entorno.
4. Completar en iOS las validaciones físicas de `IOS-PENDING-02D/02D-S` e `IOS-PENDING-03`: formularios, fechas civiles, contención cross-crush, archivo y borrado; para `03`, Corregir/Eliminar/Reintentar con los seis tipos en offline/online/background. Las validaciones de vínculo y borrado de `IOS-PENDING-04` se reemplazan por las de `IOS-RR-N2..N4` contra `API-RR-N2..N3`. La matriz de cuatro escenarios de `IOS-PENDING-02F` está validada.
5. Validar en `API-RR-N2..N3` la pertenencia obligatoria de toda Experience a un período, la ausencia de vínculos inferidos sólo por fecha y las dos semánticas distintas de borrado: hito posterior versus período completo. La concurrencia real debe ejecutarse en un motor que aplique locks.
6. Reconciliar el alcance de `IOS-PENDING-02E` mediante `IOS-RR-N1..N2`: diez situaciones, trayectos lógicos, creación frente a corrección, hito inicial y ausencia de mutación histórica. Las expectativas de ocho estados quedan retiradas.
7. Validar físicamente `API-PENDING-05` tras el despliegue: las altas confirmadas aparecen una sola vez en Home, la relación inicial no duplica el alta del Crush y editar una Relationship existente no crea otra actividad. La implementación y la cobertura automatizada ya están completas.

## Decisiones de producto abiertas

Las [decisiones pendientes](../../../PRODUCT.md#decisiones-pendientes) y la adaptación de [Relaciones](../../../rules/RELATIONSHIPS.md) deben resolverse/reconciliarse antes de una fase dependiente. La mecánica durable conserva RP-DUR/RP-RET/RP-RES.

## Criterios de aceptación

- [ ] GWT-01 a GWT-46 demostrados por tests y recorridos físicos proporcionales.
- [ ] Ningún formulario inválido entra al pipeline durable.
- [ ] Ningún pendiente se duplica, contamina otro crush o bloquea un dominio ajeno.
- [ ] Sin red no hay loop; agotamiento y rechazo determinista no reviven solos.
- [ ] `Necesita revisión` siempre tiene motivo y resolución operable.
- [ ] Descarte sólo cuando existe certeza de no mutación remota.
- [ ] Home/Profile/Archivados conservan identidad, badge y acciones coherentes.
- [ ] Ningún badge puede presentar una hoja vacía si su operación completa durante el tap.
- [ ] Borrar experiencia, relación y crush nunca es un botón muerto ni un falso éxito.
- [ ] Contenido privado permanece cifrado y fuera de metadata/logs.

## Planes locales

- [`api/docs/plans/active/durable-write-feedback/README.md`](../../../../../api/docs/plans/active/durable-write-feedback/README.md).
- [`ios/docs/plans/active/durable-write-feedback/README.md`](../../../../../ios/docs/plans/active/durable-write-feedback/README.md).

Al cerrar la iniciativa, las reglas estables se integran en [Sincronización](../../../rules/SYNC.md) y los demás dominios afectados, y el detalle técnico en [`ARCHITECTURE.md`](../../../ARCHITECTURE.md), y los tres planes se eliminan.
