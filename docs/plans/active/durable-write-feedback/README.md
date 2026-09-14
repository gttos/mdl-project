# Cambios pendientes visibles en beta

> **Estado:** activo. Base durable, retry, proyección y contención están implementados. La matriz física de `IOS-PENDING-02F` y la protección desplegada de `API-PENDING-03` están validadas. `IOS-PENDING-02E`, `IOS-PENDING-03`, `IOS-PENDING-04`, `API-PENDING-04`, `API-PENDING-05` y `API-PENDING-06` tienen implementación y cobertura automatizada completas; `02E`, `03`, `IOS-PENDING-04`, `API-PENDING-04` y `API-PENDING-05` conservan las validaciones desplegadas/físicas indicadas abajo, mientras que `API-PENDING-06` no requiere una validación física adicional. En `IOS-PENDING-03`, Corregir reabre los seis tipos desde la intención cifrada y preserva sus IDs; las ediciones sólo se reabren cuando el Timeline confirmado aporta su contexto completo, sin inventar fechas, vínculos ni identidades. `IOS-PENDING-04` completa en los flujos de alta la unicidad diaria por Crush y tipo, conserva libre el slot de Experience ocupado sólo por una Relationship standalone y deja el rechazo `422` de API como defensa autoritativa ante datos desactualizados o carreras. No queda otra fase de código en esta iniciativa: continúan únicamente las validaciones físicas/desplegadas de `API-PENDING-02/02-R`, `IOS-PENDING-02D/02D-S`, `IOS-PENDING-02E`, `IOS-PENDING-03`, `IOS-PENDING-04`, `API-PENDING-04` y `API-PENDING-05`. `API-PENDING-05` registra altas confirmadas de Crush, Experience y Relationship; editar una Relationship existente no genera actividad, aunque cambie su estado, conforme a [`PRODUCT.md` §8.4](../../../PRODUCT.md#84-actividad-reciente-de-home). El antiguo escaneo D1/D2 fue reemplazado por el borrado dirigido estable documentado en [`VAULT.md`](../../../VAULT.md) y ya no forma parte de este plan.
> **Proyectos:** iOS y API.

> **API-PENDING-04 — estado vigente:** implementación y pruebas automatizadas completas; el borrado preserva la correlación de una relación inicial y limpia la de un vínculo standalone. Quedan despliegue, validación física y prueba de concurrencia real.

> **IOS-PENDING-04 — estado vigente:** implementación y pruebas automatizadas completas. Los guards de alta usan fechas confirmadas y pendientes filtradas por Crush cuando el Timeline está disponible; los flujos con fecha prebloqueada que no disponen de ese contexto conservan el rechazo autoritativo `422`. Queda validación física offline/online/background contra `API-PENDING-04` desplegada.

## Objetivo

Cuando una persona pulsa guardar, MDL distingue inmediatamente entre guardado local, sincronización en curso, intervención requerida y confirmación remota. Cada crush admite como máximo una operación pendiente por dominio de conflicto; profile y Timeline no se confunden y una pendiente nunca contamina otro crush.

## Alcance

- Altas y ediciones durables de crush, experiencia y relación.
- Proyección local cifrada en Lista y Timeline.
- Estado visible, retry limitado, corrección y descarte seguro.
- Bloqueos sólo por crush y dominio realmente conflictivo.
- Archivo y borrado interactuables aunque existan pendientes, conforme a certeza remota.
- Compatibilidad conservadora con operaciones legacy.

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

Esta matriz es el contrato ejecutable de aceptación de la iniciativa mientras permanezca activa; no reemplaza las reglas estables de [`PRODUCT.md`](../../../PRODUCT.md) y [`TIMELINE.md`](../../../TIMELINE.md).

- GWT-01 a GWT-14 y GWT-31 a GWT-42 ejercitan reglas de dominio/Timeline que **ya viven de forma permanente** en esos documentos canónicos. Se conservan aquí temporalmente porque API e iOS todavía tienen fases abiertas que deben demostrarlas de extremo a extremo.
- GWT-15 a GWT-30 describen la mecánica específica de guardado durable, proyección, retry, resolución y compatibilidad legacy que esta iniciativa está terminando.
- Al cerrar la iniciativa, el comportamiento durable final se integra en `PRODUCT.md`/`ARCHITECTURE.md`; este plan y los locales se eliminan. Las reglas de Timeline no se “pierden” ni dependen de conservar el plan.

### Formularios y cronología

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-01 | Un alta de crush con datos obligatorios y relación inicial completos, y el bloque de experiencia vacío | La persona guarda | Se crea una única intención de bundle sin experiencia; la relación inicial usa `met_at`. |
| GWT-02 | El bloque opcional de experiencia tiene uno o dos de lugar, descripción y satisfacción | La persona intenta guardar | Los campos vacíos se marcan inline; no hay alert genérico, snapshot, operación, proyección ni request. |
| GWT-03 | Los tres campos de la experiencia inicial están completos | La persona guarda | Crush, relación inicial y experiencia representan el mismo día civil y la misma identidad de bundle. |
| GWT-04 | Un crush confirmado y un día válido sin otra experiencia ni relación iniciada ese día | La persona crea una experiencia normal sin cambio de relación | Aparece una sola experiencia standalone en ese día. |
| GWT-05 | No existe experiencia inicial y una experiencia normal no está vinculada | Se crea en `met_at` o se edita su fecha a `met_at` | La misma experiencia ocupa el slot inicial, conserva su UUID, adopta la correlación inicial y se reconstruye dentro del bundle, no como tarjeta separada. |
| GWT-06 | El día de `met_at` ya contiene la experiencia inicial | Se intenta crear o mover otra experiencia a ese día | Se rechaza por unicidad diaria sin mutar la candidata. |
| GWT-07 | Una experiencia está vinculada a una relación | Se intenta cambiar sólo su fecha | La UI mantiene la fecha bloqueada y API rechaza cualquier drift; relación y experiencia conservan exactamente la misma fecha. |
| GWT-08 | Una experiencia confirmada, standalone o vinculada | La persona elige `Eliminar` y el borrado finaliza | La experiencia desaparece. Si estaba vinculada, la relación permanece con mismo estado/fechas y queda sin experiencia; no se reabre otra relación. |
| GWT-09 | Una experiencia sin relación puede originar una relación | La persona elige `Agregar relación` | La fecha de la experiencia se prellena y bloquea; al confirmar, ambas quedan vinculadas en el mismo día, sin crear una experiencia duplicada. |
| GWT-10 | Una relación standalone puede recibir una experiencia | La persona elige `Linkear experiencia` | La fecha de la relación se prellena y bloquea; se crea una única experiencia vinculada a esa relación, no una standalone independiente. |
| GWT-11 | Existe una cadena de relaciones | La persona abre acciones de una relación histórica | Editar y eliminar no se muestran. Sólo la relación latest/current puede ofrecer edición y sólo si es posterior puede ofrecer eliminación. |
| GWT-12 | Una relación posterior latest/current tiene una relación anterior | Se elimina la latest | La anterior se reabre; la relación inicial nunca se elimina y eliminar una experiencia nunca produce este efecto. |
| GWT-13 | `met_at` se edita dentro del rango permitido por eventos confirmados y pendientes del mismo crush | La persona guarda | Es una operación de profile; mueve el bundle inicial junto y no queda bloqueada por una pendiente de Timeline. |
| GWT-14 | La fecha propuesta de `met_at` quedaría después de un evento posterior no perteneciente al bundle | La persona usa el calendario o intenta guardar | La UI limita/rechaza la fecha antes de persistir; si un cliente desactualizado recibe rechazo API, muestra el error y refresca sin retry automático de validación. |

### Persistencia, proyección y resolución

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-15 | Un formulario válido | La intención cifrada queda persistida localmente | El formulario comunica `Guardado en este dispositivo`, se cierra y aparece una proyección; todavía no se muestra `Sincronizado`. |
| GWT-16 | Un alta de crush aún no existe confirmada remotamente | Se muestra su proyección root | La fila está atenuada, no abre perfil ni admite descendientes o archivo; abre su estado y sólo permite retry/descarte cuando sea seguro. |
| GWT-17 | Hay una operación Timeline pendiente para A | Se navega por A, B y el Timeline global | A muestra esa operación una vez; B nunca la muestra ni queda bloqueado; el global la muestra una sola vez. |
| GWT-18 | Una edición pendiente corresponde a una entidad ya confirmada | Se fusionan API y proyección | Se conserva una sola tarjeta con el mismo ID, valores y fecha propuestos. Descartar restaura la versión confirmada. |
| GWT-19 | Hay una pendiente de Timeline para un crush | Se intenta otra alta/edición de experiencia o relación, o borrar una relación del mismo crush | La acción se deshabilita con explicación. Profile, `met_at` válido, fotos, navegación, archivo, borrar experiencia y otros crushes siguen operativos. |
| GWT-20 | Hay una pendiente de profile que no modifica `met_at` | Se intenta usar Timeline | Timeline sigue operativo; sólo otra edición de profile del mismo crush queda reservada. |
| GWT-21 | Una operación no tiene red antes del primer envío | La app permanece offline | No consume intentos ni genera tráfico repetido; muestra `Pendiente de conexión` y permite eliminar el cambio local. |
| GWT-22 | Un fallo transitorio produce timeout, `5xx` o `429` | Se ejecuta la política automática | Se reutilizan IDs y, al agotarse tres intentos del ciclo con backoff, queda `No se pudo sincronizar` sin revivir en foreground. |
| GWT-23 | API devuelve un rechazo determinista sin mutación | Se clasifica la respuesta | Pasa directamente a `Necesita revisión`, conserva razón estructurada localizada y no entra en loop automático. |
| GWT-24 | Una operación en revisión es atribuible y segura | La persona abre su badge/tarjeta | La hoja exacta por `operationId` ofrece la combinación real de corregir, retry manual, eliminar/descartar y cerrar; nunca sólo una promesa futura. |
| GWT-25 | Una operación nunca enviada o rechazada con certeza de no mutación | La persona confirma eliminar/descartar | Se elimina la intención completa y su proyección; un alta desaparece y una edición vuelve a la entidad confirmada. |
| GWT-26 | Existe timeout ambiguo o etapa `apiAccepted+` | La persona quiere eliminar el cambio | No se finge descarte local. Primero se reconcilia; si la entidad existe, se utiliza después el borrado funcional normal. |
| GWT-27 | Un crush confirmado tiene pendiente de Timeline | La persona lo archiva | Se ejecuta el PATCH real. Tras éxito sale de activos y Archivados conserva badge/hoja hasta resolver la pendiente. |
| GWT-28 | La persona pulsa eliminar un crush con pendientes | Se evalúa la certeza de todas ellas | Si todas son descartables, una confirmación las elimina y continúa el borrado normal; si alguna es ambigua/aceptada, no se inicia D1/D2 y se explica cuál debe resolverse. |
| GWT-29 | Hay una legacy sin scope derivable | La app normaliza o reanuda | Queda en revisión sin crush inventado, no se ejecuta automáticamente y no bloquea ningún alta independiente. |
| GWT-30 | Una versión antigua dejó varias operaciones del mismo crush y dominio | La app reanuda | Sólo la cabecera estable puede avanzar; las posteriores nunca corren en paralelo ni se promocionan automáticamente y sólo se descartan si es seguro. |

### Presentación del Timeline

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-31 | API devuelve crush, relación inicial y experiencia opcional de igual correlación/fecha | Se reconstruye Timeline | Se muestra una sola tarjeta bundle con las partes presentes. |
| GWT-32 | Una experiencia retroactiva adopta la correlación inicial | Se refresca Timeline | Se incorpora al bundle existente; nunca queda simultáneamente standalone. |
| GWT-33 | Una fecha válida llega como ISO 8601 o `yyyy-MM-dd` | Se renderiza una tarjeta confirmada o pendiente | Ambas formas muestran el mismo día civil; `-` se reserva para datos realmente corruptos. |
| GWT-34 | Una relación y una experiencia están vinculadas directamente | Se reconstruye o edita Timeline | Comparten fecha exacta e identidad de vínculo; la UI no presenta como vínculo una propuesta local rechazada. |
| GWT-35 | Dos eventos empatan en fecha y momento de creación a través de una página | Se pagina/refresca | El orden secundario/terciario es determinista y no duplica, pierde ni cambia elementos entre páginas. |
| GWT-36 | Un crush conserva su relación inicial y no existe una relación posterior | La persona abre sus acciones | Puede editar su estado a cualquiera de los permitidos manteniendo la fecha `met_at`, pero Eliminar no aparece y API rechaza cualquier intento directo de borrado sin mutar el crush. |
| GWT-37 | Ya existe una experiencia confirmada o pendiente de un crush en el día civil `D` | Se intenta crear o mover otra experiencia a `D` | La interfaz pide editar la existente y no persiste otra intención; una petición directa/desactualizada recibe rechazo sin mutar. |
| GWT-38 | Ya existe un cambio de relación confirmado o pendiente de un crush en el día civil `D` | Se intenta crear o mover otra relación a `D` | La interfaz pide editar la relación latest de ese día y no persiste otra intención; una petición directa/desactualizada recibe rechazo sin mutar. La relación inicial y la experiencia inicial pueden compartir `D` porque son tipos distintos dentro del bundle. |
| GWT-39 | Product/API definen la matriz canónica de ocho estados en `TIMELINE.md` §4 | iOS muestra estados posibles al crear/editar una relación | Ofrece exactamente la misma matriz que API. Para la relación inicial latest, que no tiene estado anterior, ofrece los ocho estados con fecha fija a `met_at`; sus tests no protegen una matriz alternativa. |
| GWT-40 | Una edición de profile pendiente modifica realmente `met_at` de un crush confirmado | La persona intenta crear una experiencia nueva de ese mismo crush | `Agregar experiencia` no inicia formulario ni persiste otra intención y explica que debe terminar la actualización de la fecha. Al sincronizar, corregir o descartar la edición, el alta vuelve a estar disponible. No se calcula un rango hipotético entre el `met_at` confirmado y el propuesto; otros crushes y acciones no quedan redefinidos por este escenario. |
| GWT-41 | Una relación standalone no tiene experiencia vinculada | Se crea o mueve una experiencia normal no vinculada a ese mismo día civil, o se crea la relación desde esa experiencia | Ambos conservan sus UUID, comparten la correlación de la relación y aparecen en una única tarjeta vinculada; nunca quedan dos standalone simultáneas. |
| GWT-42 | Una relación ya tiene una experiencia vinculada | Se intenta crear o mover otra experiencia a ese día civil | Se rechaza antes de mutar o dentro de una transacción con rollback, se explica que ese día ya tiene experiencia y no aparece una segunda tarjeta. La beta no ofrece `Desvincular`; borrar la experiencia vinculada conserva la relación y libera su slot. |
| GWT-43 | Una creación confirmada de Crush, Experience o Relationship posterior | La persona abre Home con el Vault desbloqueado | Home muestra la actividad reciente correspondiente a la referencia estructural persistida y resuelve sus etiquetas privadas desde Vault. La relación inicial no duplica el alta del Crush; editar una Relationship existente —incluso su estado— y las demás ediciones o borrados no generan actividad. |

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
- Borrar crush siempre responde. Si todas sus pendientes son descartables, las elimina y continúa el borrado normal. Si existe una ambigua/aceptada, explica y no inicia D1/D2.
- Un root provisional se puede eliminar sólo con certeza de que no existe remoto.
- Toda experiencia confirmada se puede borrar; una vinculada deja la relación standalone.
- Una experiencia pendiente local-only/rechazada se descarta. Una ambigua/aceptada se reconcilia y luego usa borrado funcional.
- Borrar relación mantiene latest-only y los bloqueos cronológicos correspondientes.

## Interacción entre proyectos

| Proyecto | Responsabilidad |
|---|---|
| API | Idempotencia, aislamiento, transacciones y errores que distinguen rechazo seguro de resultado ambiguo. |
| iOS | Persistencia cifrada, scope/dominio/disposición, proyección, merge, retry y acciones seguras. |

## Estado y orden restante

1. **Código:** no queda otra fase de implementación pendiente en esta iniciativa. `IOS-PENDING-03` deja Corregir disponible para los seis tipos de operación; `IOS-PENDING-04` completa la unicidad diaria/vínculo por fecha de GWT-37/GWT-38/GWT-41/GWT-42; `API-PENDING-06` completa GWT-35 con el orden total `event_date DESC, created_at DESC, item_type ASC, item_id ASC`.
2. Completar en el entorno desplegado la validación de `API-PENDING-02/02-R`: reasociación a `met_at` con slot libre/ocupado/vinculada y concurrencia real del lock. `API-PENDING-03` ya valida la exclusión de la relación inicial en ese entorno.
3. Completar en iOS las validaciones físicas de `IOS-PENDING-02D/02D-S`, `IOS-PENDING-03` e `IOS-PENDING-04`: formularios, fechas civiles, contención cross-crush, archivo y borrado; para `03`, Corregir/Eliminar/Reintentar con los seis tipos en offline/online/background; para `04`, unicidad diaria y vínculo Relationship–Experience con API desplegada, incluyendo datos desactualizados/carreras. La matriz de cuatro escenarios de `IOS-PENDING-02F` está validada.
4. Desplegar y validar físicamente `API-PENDING-04`: unicidad diaria, vínculo por fecha y correlación, rechazo sin mutación de fechas manipuladas y conservación de la relación inicial tras borrar su experiencia. Ejecutar una prueba de concurrencia real en un motor que aplique locks.
5. Validar físicamente `IOS-PENDING-02E`: los ocho estados de la relación inicial, los destinos de Prospecto y Rechazado, el cambio de relación dentro de experiencia, guardar la latest sin cambiar estado y la ausencia de edición en una relación histórica.
6. Validar físicamente `API-PENDING-05` tras el despliegue: las altas confirmadas aparecen una sola vez en Home, la relación inicial no duplica el alta del Crush y editar una Relationship existente no crea otra actividad. La implementación y la cobertura automatizada ya están completas.

## Decisiones de producto abiertas

Ninguna conocida dentro del alcance vigente. Si aparece una nueva durante implementación, se detiene esa parte y se consulta de una en una antes de codificar.

## Criterios de aceptación

- [ ] GWT-01 a GWT-43 demostrados por tests y recorridos físicos proporcionales.
- [ ] Ningún formulario inválido entra al pipeline durable.
- [ ] Ningún pendiente se duplica, contamina otro crush o bloquea un dominio ajeno.
- [ ] Sin red no hay loop; agotamiento y rechazo determinista no reviven solos.
- [ ] `Necesita revisión` siempre tiene motivo y resolución operable.
- [ ] Descarte sólo cuando existe certeza de no mutación remota.
- [ ] Home/Profile/Archivados conservan identidad, badge y acciones coherentes.
- [ ] Borrar experiencia, relación y crush nunca es un botón muerto ni un falso éxito.
- [ ] Contenido privado permanece cifrado y fuera de metadata/logs.

## Planes locales

- [`api/docs/plans/active/durable-write-feedback/README.md`](../../../../../api/docs/plans/active/durable-write-feedback/README.md).
- [`ios/docs/plans/active/durable-write-feedback/README.md`](../../../../../ios/docs/plans/active/durable-write-feedback/README.md).

Al cerrar la iniciativa, las reglas estables se integran en [`PRODUCT.md`](../../../PRODUCT.md) y [`ARCHITECTURE.md`](../../../ARCHITECTURE.md), y los tres planes se eliminan.
