# Reglas de producto — Experiencias

[Índice de reglas](../PRODUCT.md) · Experiencia inicial y posteriores: datos, fechas, creación, edición, eliminación y vinculación con relaciones.

Una **experiencia** conserva qué ocurrió con un Crush en un día civil y pertenece siempre a un único período relacional. Es **independiente** cuando permanece dentro de ese período sin hito porque no registró una situación nueva; «independiente» nunca significa que exista fuera de un período. Cuando sí registra una situación nueva, queda vinculada al hito correspondiente. La experiencia no sustituye al hito: sus datos propios se rigen por este documento y la estructura relacional, por [Relaciones](RELATIONSHIPS.md).

<a id="inventario"></a>
## Inventario funcional de una experiencia

- **RP-EXP-025.** Una experiencia de la beta se compone del siguiente inventario cerrado.

| Tratamiento | Dato exacto | Condición y uso autorizado |
| --- | --- | --- |
| API legible | Identificador de la experiencia | Lo genera el cliente MDL antes de persistir, es inmutable y se reutiliza en reintentos, ediciones, movimientos o vínculos. |
| API legible | Cuenta propietaria | Es inmutable y limita la experiencia a su cuenta propietaria. |
| API legible | Crush propietario | Es obligatorio e inmutable. Una experiencia no cambia de Crush. |
| API legible | Día de la experiencia | Es obligatorio, se conserva como día civil y cumple las reglas cronológicas del Crush. |
| API legible | Satisfacción | Es obligatoria y admite exactamente los enteros 1, 2, 3, 4 o 5. |
| API legible | Emociones | Conjunto opcional de IDs de RP-EXP-029; vacío se representa como `[]`. |
| API legible | Conexión, escucha y deseo posterior | Selecciones opcionales independientes de RP-EXP-029; sin respuesta se representan como `null`. |
| API legible | Autenticidad, seguridad y expresión de límites | Selecciones opcionales independientes de RP-EXP-029; sin respuesta se representan como `null`. |
| Administrado por el sistema | Correlación de una acción compuesta | Une la experiencia con el bloque inicial o con un cambio relacional registrado en la misma acción. No es editable ni visible como contenido de la experiencia. |
| Administrado por el sistema | Período al que pertenece | Es obligatorio e identifica el único período que contiene la experiencia. |
| Derivado por la API | Hito vinculado | Sólo tiene valor cuando la experiencia registra una situación nueva dentro de su período. La asociación explícita identifica el hito; la coincidencia de fechas por sí sola nunca crea ese vínculo. |
| Administrado por el sistema | Fechas técnicas de creación, actualización y eliminación | Sostienen orden, sincronización y ciclo de vida. No sustituyen `date`. |
| Vault opaco | Lugar | Es texto libre obligatorio de hasta 255 caracteres, privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |
| Vault opaco | Descripción | Es el relato libre obligatorio de hasta 5.000 caracteres y sólo se muestra con el Vault desbloqueado. |
| Vault opaco | Reflexión | Es un texto libre opcional de hasta 5.000 caracteres sobre lo aprendido; sin respuesta se representa como `null`. Sólo se muestra con el Vault desbloqueado. |

- **RP-EXP-026.** La estructura funcional legible de una experiencia está formada por su identidad, propiedad, fecha, satisfacción, las siete dimensiones de respuestas estructuradas y sus asociaciones. El contenido privado de su payload Vault está formado exactamente por `place`, `description` y `reflection`; satisfacción y respuestas estructuradas pertenecen exclusivamente a la estructura funcional y no se serializan ni se duplican dentro de ese payload. El dispositivo cifra el payload antes de transferirlo, conserva localmente la entry cifrada y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. Sus textos sólo se descifran en la memoria de un dispositivo autorizado que haya desbloqueado o recuperado el mismo Vault; la API funcional interpreta únicamente la estructura de la experiencia.

<a id="inicial"></a>
## Experiencia inicial obligatoria

- **RP-INI-001.** Toda alta de un Crush exige una experiencia inicial completa que explique cómo se conocieron. Su fecha es `met_at`; lugar, descripción y satisfacción son obligatorios. Mientras cualquiera de esos datos falte o sea inválido, el alta no puede confirmarse y la aplicación señala todos los campos que deben corregirse.
- **RP-INI-002.** Lugar, descripción y la reflexión opcional viven exclusivamente dentro del Vault con las garantías de RP-EXP-026. La API interpreta satisfacción y las respuestas estructuradas opcionales, además de fecha y asociaciones. La experiencia inicial utiliza los mismos niveles y catálogos que las posteriores, sin exigir ninguna de las respuestas nuevas.
- **RP-INI-003.** La experiencia inicial forma parte del bloque inicial y queda vinculada al Crush, al período inicial y a su hito inicial.

<a id="contenido"></a>
## Contenido privado de la experiencia

- **RP-VLT-006.** Cada experiencia tiene una única descripción, identificada como `description`, y vive en el payload Vault de esa experiencia.

- **RP-EXP-032.** La reflexión es un texto privado opcional separado del relato y pertenece a la misma experiencia. Se cifra con lugar y descripción; permanece excluida de API legible, analítica y LLM. Su información de protección conserva el significado de RP-EXP-027.
- **RP-EXP-027.** Descripción informa de que su contenido se guarda cifrado y permanece privado entre cuentas. Esa garantía se refiere al relato y se distingue del tratamiento de fechas, satisfacción y respuestas estructuradas.

<a id="niveles"></a>
## Niveles de detalle y respuestas

- **RP-EXP-028.** Cada experiencia admite los niveles acumulativos Breve, Con detalle y Reflexión, con las preguntas 1–3, 1–6 y 1–10 de la tabla. Fecha y lugar quedan fuera de ese conteo. Elegir un nivel o consultar opciones conserva las respuestas hasta que la persona las cambie expresamente.

| Nº | Dato | Significado funcional |
|---|---|---|
| 1 | Satisfacción | Valoración global de la experiencia, de 1 a 5. |
| 2 | Emociones | Emociones experimentadas durante el encuentro. |
| 3 | Descripción | Relato privado de lo ocurrido y de lo que se quiere recordar. |
| 4 | Conexión | Grado de conexión percibida con la otra persona. |
| 5 | Escucha | Grado en que la persona se sintió escuchada. |
| 6 | Deseo posterior | Qué desea que ocurra después de la experiencia. |
| 7 | Autenticidad | Grado en que pudo mostrarse tal como es. |
| 8 | Seguridad | Percepción de seguridad durante el encuentro. |
| 9 | Límites | Posibilidad de expresar lo que quería y lo que no. |
| 10 | Reflexión | Aprendizaje privado sobre lo que quiere o necesita. |

- **RP-EXP-029.** Los siguientes catálogos son cerrados y comparten IDs entre plataformas e idiomas. Los IDs identifican opciones; no constituyen por sí mismos magnitudes promediables. Satisfacción conserva su escala 1–5. No se añade texto libre para «Otra / Otra cosa».

| Campo | IDs y etiquetas ES / EN |
|---|---|
| `emotion_ids` | 1 Alegría / Joy; 2 Calma / Calm; 3 Ilusión / Excitement; 4 Nervios / Nervousness; 5 Confusión / Confusion; 6 Tristeza / Sadness; 7 Decepción / Disappointment; 8 Otra / Other; 9 No lo sé / Not sure. |
| `connection_id` | 1 Nada / None; 2 Poca / A little; 3 Algo / Some; 4 Bastante / Quite a lot; 5 Mucha / A lot; 6 No lo sé / Not sure. |
| `listening_id`, `authenticity_id` | 1 Nada / Not at all; 2 Poco / A little; 3 A ratos / At times; 4 Bastante / Quite a lot; 5 Mucho / Very much; 6 No lo sé / Not sure. |
| `desired_next_step_id` | 1 Volver a compartir algo / Spend time together again; 2 Tomarme tiempo / Take some time; 3 No continuar / Not continue; 4 No lo sé / Not sure; 5 Otra cosa / Something else. |
| `safety_id` | 1 Sí / Yes; 2 En parte / Partly; 3 No / No; 4 No lo sé / Not sure; 5 No aplica / Not applicable. |
| `boundaries_id` | 1 Sí / Yes; 2 En parte / Partly; 3 No / No; 4 No surgió la ocasión / It did not come up; 5 No lo sé / Not sure. |

- **RP-EXP-030.** Las siete respuestas nuevas y la reflexión son opcionales, sin preselección ni valor inferido. `emotion_ids` admite cualquier subconjunto sin duplicados del catálogo, incluidas emociones mezcladas, y se normaliza por ID ascendente; no se infiere una restricción de exclusividad entre opciones. Las otras seis dimensiones admiten un único ID o `null`. Los IDs deben ser enteros reales del catálogo; no se aceptan booleanos, decimales ni strings numéricos. Elegir «No lo sé», «No aplica» u otra opción explícita es distinto de no responder. Se mantienen los cuatro datos obligatorios y límites de RP-EXP-003.
- **RP-EXP-031.** Alta inicial y posterior guardan todas las respuestas presentes de forma atómica con la estructura existente; las no informadas se inicializan como `[]` o `null`. En edición, omitir una dimensión conserva el valor, `[]` vacía emociones y `null` retira una selección simple. Una selección inválida rechaza toda la acción. Las lecturas confirmadas publican siempre las siete dimensiones con su representación canónica; una clave omitida no se trata como ausencia de respuesta por compatibilidad pre-beta. Los snapshots durables congelan también presencia y valores para reintentos y correcciones.
- **RP-EXP-033.** El nivel inicial es Breve. Elegir otro nivel modifica sólo esa experiencia; cambiar expresamente el nivel habitual modifica una preferencia local separada por cuenta, conservada entre aperturas y retirada al dar de baja la cuenta. Cada Crush utiliza esa misma preferencia de cuenta.
- **RP-EXP-034.** Cambiar de nivel, avanzar o volver conserva respuestas y valores, incluidos los que queden ocultos al guardar. Editar abre un nivel suficiente para consultar todas las respuestas existentes, manteniendo la preferencia habitual. Elegir Reflexión permite responder libremente a las preguntas opcionales. El nivel es una elección de consulta y entrada; las estadísticas utilizan las respuestas guardadas.
- **RP-EXP-035.** Las respuestas de una experiencia conservan independientes situaciones, períodos, hitos, match y valoraciones del Crush. Seguridad, autenticidad y límites son dimensiones distintas. La consulta permite acceder al contexto, lugar, descripción, satisfacción, todas las respuestas y reflexión disponibles, con sus garantías respectivas, sin generar una puntuación compuesta, diagnóstico o interpretación automática.
- **RP-EXP-036.** Las respuestas estructuradas conservan el tratamiento de RP-EXP-026 y quedan excluidas de logs, diagnóstico y analítica de uso. La información de cifrado de Descripción y Reflexión se refiere exclusivamente a esos textos; las aceptaciones y consultas jurídicas mantienen su función propia.

<a id="crear"></a>
## Crear una experiencia

- **RP-EXP-001.** Una experiencia pertenece a un Crush y la combinación (`crush_id`, `date`) es única, incluidos los cambios pendientes conocidos. Por tanto, cada Crush puede tener como máximo una experiencia en un día civil, mientras que dos o más Crushes distintos pueden tener cada uno su propia experiencia en ese mismo día.
- **RP-EXP-002.** Ampliar o corregir lo ocurrido con el mismo Crush en ese mismo día modifica la experiencia existente; no crea una segunda para ese Crush.
- **RP-EXP-003.** Toda experiencia conserva obligatoriamente `date`, `satisfaction_id`, `place` y `description`. `satisfaction_id` admite exactamente los enteros 1, 2, 3, 4 o 5. `place` y `description` son textos libres privados que viven dentro del payload Vault; `place` admite hasta 255 caracteres y `description`, hasta 5.000. No pueden omitirse, quedar vacíos ni superar esos límites al crear o editar una experiencia.
- **RP-EXP-004.** Mientras exista un período vivo, «Añadir experiencia» comienza con un único paso para completar fecha, satisfacción, lugar y descripción. Al final ofrece la acción opcional «¿Hay un cambio de situación?». Si no se activa, guardar crea una experiencia independiente dentro del período. Si se activa, el formulario añade un segundo paso obligatorio para elegir la nueva `RelationshipSituation` y, cuando sea `CLOSED`, completar el motivo; guardar crea conjuntamente experiencia e hito con la misma fecha, correlación y vínculo explícito. Si cualquiera de las partes es inválida, no se crea ninguna. Si el período se cierra más adelante, las experiencias independientes que ya contenía continúan dentro de ese período histórico.
- **RP-EXP-005.** Toda experiencia pertenece exactamente a un período relacional y puede quedar vinculada como máximo a un único hito de ese período.
- **RP-EXP-006.** Para ese Crush, el día `met_at` está ocupado por la experiencia inicial obligatoria. Ninguna experiencia posterior del mismo Crush puede crearse en ese día; las experiencias de otros Crushes no ocupan ese espacio.
- **RP-EXP-010.** Cuando el período más reciente está cerrado, «Añadir experiencia» es obligatoriamente un formulario de dos pasos y no muestra la pregunta opcional de RP-EXP-004. El primer paso completa fecha, satisfacción, lugar y descripción; el segundo exige elegir una situación inicial admitida por la historia conservada conforme a RP-REL-007. Para la persona se trata de registrar la experiencia y la nueva situación; guardar crea transparentemente la experiencia, el período nuevo y su hito inicial con la misma fecha, correlación y vínculo explícito. La experiencia no puede confirmarse por separado y, si cualquier parte es inválida, no se crea ninguna.
- **RP-EXP-011.** «Añadir experiencia» limita su selector al intervalo cronológico continuo que permite el estado confirmado del Crush y nunca a fechas futuras. Para una experiencia independiente dentro de un período vivo, el mínimo es el día posterior a `start_date`, cuyo propio día ya ocupa la experiencia inicial. Para un cambio no terminal, el mínimo es el día posterior al último hito. Para cerrar, el mínimo es el día posterior a la fecha más reciente entre el último hito y todas las experiencias ya contenidas en el período. Para abrir un período después de otro cerrado, el mínimo es el día posterior a `end_date`. El máximo es el día civil local definido por RP-ID-006, no el día UTC del servidor. Cambiar la intención o el destino recalcula el intervalo, pero no sustituye silenciosamente una fecha ya elegida: si queda fuera, se marca inválida y se exige corregirla. Si el mínimo supera hoy, todavía no existe una fecha válida y la interfaz explica el bloqueo. Los días intermedios ya ocupados continúan siendo inválidos por RP-EXP-001; pertenecer al intervalo no los autoriza y la validación local y de API permanece como defensa.

<a id="editar"></a>
## Editar una experiencia

- **RP-EXP-012.** Una experiencia confirmada puede editarse sin límite de antigüedad. Si es independiente, su fecha puede atravesar uno o varios hitos del mismo período, pero nunca salir de él: no puede quedar antes de `start_date` ni después de `end_date` cuando esté cerrado, y debe respetar hoy, `met_at` y la regla de una experiencia por Crush y día civil. Moverla no la reasigna a otro período.
- **RP-EXP-013.** La fecha de una experiencia vinculada a un hito relacional no puede editarse de manera independiente. Editar sus demás datos no mueve ni reescribe el hito.
- **RP-EXP-014.** La fecha de la experiencia inicial no se edita desde la propia experiencia: siempre coincide con `met_at` y se actualiza conjuntamente cuando esa fecha se corrige válidamente desde el Crush.
- **RP-EXP-015.** Una experiencia posterior no puede convertirse en experiencia inicial ni moverse a `met_at`, porque ese día conserva siempre la experiencia obligatoria del bloque inicial.
- **RP-EXP-016.** Una experiencia independiente puede moverse de un lado a otro de un hito de su mismo período y su contexto relacional se recalcula para la nueva fecha. No puede moverse al mismo día civil de un hito porque ese día ya contiene la experiencia vinculada a ese hito. El movimiento nunca crea un vínculo, otro hito ni un cambio de período.
- **RP-EXP-017.** La beta no ofrece una acción independiente para cambiar las asociaciones de una experiencia. No puede separarse de su período ni vincularse o desvincularse manualmente de un hito. Una experiencia vinculada sólo queda independiente cuando se conserva al eliminar su hito conforme a RP-HIT-009.
- **RP-EXP-018.** Un rechazo identifica el campo o la acción afectada y la causa de manera comprensible, tanto si se detecta antes de guardar como si la API lo confirma después durante la sincronización, y nunca anuncia un éxito inexistente. Las causas deterministas que la persona puede corregir forman un catálogo cerrado: API publica únicamente un código estructurado estable y su campo asociado, y el cliente conserva ese código, lo traduce localmente y guía la corrección en la hoja de estado y en el formulario. La interfaz nunca muestra el código, el mensaje libre del servidor ni texto de una excepción; una causa desconocida conserva una explicación genérica y no se convierte en éxito.

<a id="eliminar"></a>
## Eliminar una experiencia

- **RP-EXP-019.** Una experiencia independiente puede eliminarse sin límite de antigüedad y sin eliminar su período. Una experiencia vinculada no utiliza esta acción independiente: su borrado se rige por RP-EXP-020.
- **RP-EXP-020.** Iniciar el borrado desde una experiencia vinculada solicita siempre eliminar conjuntamente la experiencia y su hito. La elegibilidad latest-only y el alcance atómico de la operación —«Eliminar cambio y experiencia» o «Eliminar período»— se rigen exclusivamente por RP-HIT-009 y RP-HIT-010. Si esas condiciones no se cumplen, no se elimina ninguna parte.
- **RP-EXP-021.** La experiencia inicial no puede eliminarse de manera independiente ni ofrece esa acción. Puede editarse; sólo el borrado completo del Crush retira el bloque inicial y toda su historia.
- **RP-EXP-022.** Una experiencia independiente confirmada puede eliminarse aunque exista otro cambio pendiente que no la afecte. Una experiencia vinculada a un hito no puede eliminarse mientras una operación pendiente pueda alterar cuál es el último elemento relacional; primero se resuelve esa operación y después se revalida la elegibilidad latest-only.
- **RP-EXP-023.** Una experiencia pendiente que nunca se envió o fue rechazada sin cambios puede descartarse. Si pudo haber sido aceptada, debe comprobarse primero su resultado.
- **RP-EXP-024.** El contenido privado de una experiencia eliminada sigue las reglas de retirada y destrucción de [Vault](VAULT.md) y [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).

Las opciones de cambio relacional se rigen por [Relaciones](RELATIONSHIPS.md#trayectos). Guardar una experiencia junto con un cambio no amplía los trayectos ni vuelve editable un hito o período histórico.

### Escenarios de aceptación de la explicación de Descripción

| ID | Dado / Cuando / Entonces | Regla |
|---|---|---|
| GWT-ODM-20 | Consultar Descripción informa de su cifrado y privacidad sin atribuirlos a la estructura legible. | RP-EXP-027 |

<a id="escenarios-niveles"></a>
## Escenarios de niveles, respuestas y lectura

| ID | Dado / Cuando / Entonces | Regla |
|---|---|---|
| GWT-EXP-D01 | Dado el catálogo, al consultarlo en ES/EN se obtienen los mismos IDs y opciones cerradas; no se devuelven selecciones de cuentas. | RP-EXP-029 |
| GWT-EXP-D02 | Elegir un nivel permite completar 3/6/10 preguntas sin respuestas inferidas ni cambio relacional implícito. | RP-EXP-028/030/035 |
| GWT-EXP-D03 | Dado un conjunto de emociones o una selección simple, al validar se permiten sólo los tipos/IDs canónicos; duplicados y valores inválidos rechazan la acción sin I/O de escritura. | RP-EXP-029/030 |
| GWT-EXP-D04 | Dada un alta inicial o posterior, al guardar se confirman juntas estructura y respuestas; la lectura presenta las siete dimensiones incluso sin respuesta. | RP-EXP-031, RP-INI-001..003 |
| GWT-EXP-D05 | Dada una experiencia confirmada, al editar se distingue omisión, `[]`, `null` y sustitución sin borrar otras dimensiones ni modificar identidad o asociaciones. | RP-EXP-031 |
| GWT-EXP-D06 | Cambiar de nivel o volver conserva las respuestas y editar permite consultarlas todas. | RP-EXP-028/034 |
| GWT-EXP-D07 | Dada la preferencia de una cuenta, al cambiar nivel sólo para una experiencia no cambia la habitual; la acción explícita y Ajustes sí la cambian y el relanzamiento la conserva sin cruzarla a otra cuenta. | RP-EXP-033 |
| GWT-EXP-D08 | Lugar, descripción y reflexión permanecen cifrados y separados de satisfacción y respuestas legibles. | RP-EXP-026/027/032/036 |
| GWT-EXP-D09 | Dada una escritura offline o ambigua, al relanzar/reintentar/corregir se reutilizan IDs, snapshot, presencia de cambios y `operationId`; no se reconstruye desde una lectura posterior ni se pierde lo oculto. | RP-EXP-031/034 y Sync |
| GWT-EXP-D10 | Dadas respuestas y texto en dos cuentas, al bloquear/cambiar cuenta una respuesta tardía no publica contenido ni preferencias de la anterior; logs/analítica no reciben respuestas o textos. | RP-ID-007, RP-EXP-036 |
| GWT-EXP-D16 | Consultar una experiencia permite acceder a todas sus respuestas y textos, conservando contexto, acciones y datos. | RP-EXP-035 |
