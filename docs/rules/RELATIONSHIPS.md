# Reglas de producto — Relaciones

<a id="decisiones-pendientes"></a><a id="dec-13"></a>

[Índice de reglas](../PRODUCT.md) · Situaciones, períodos, hitos, evidencia relacional de match, cierre, corrección y eliminación.

Una **situación relacional** describe el estado exacto y visible de la historia en un momento. Un **período relacional** contiene un tramo continuo de esa historia. Un **hito relacional** crea el período o cambia su situación. Cada hito tiene exactamente una experiencia vinculada que explica qué ocurrió; una experiencia puede existir sin hito, pero siempre pertenece a un período.

Las situaciones y sus explicaciones conservan el mismo significado en español e inglés. Cambiar de idioma conserva identidades, selección y datos de la historia.

<a id="situaciones"></a>
## Situaciones relacionales

- **RP-SIT-001.** Al dar de alta un Crush, la persona elige una de las situaciones iniciales admitidas 1–9 para describir el punto real en el que se encuentra la historia.
- **RP-SIT-002.** MDL utiliza siempre la situación exacta elegida por la persona. No la deduce de Ducky, satisfacción, texto privado, actividad, estadísticas ni IA.
- **RP-SIT-003.** El catálogo relacional contiene exactamente las diez situaciones identificadas en la tabla. Cada situación conserva una identidad única compartida por API y aplicaciones.
- **RP-SIT-004.** Las claves internas permanecen en inglés y nunca se muestran como copy. Las aplicaciones resuelven cada ID a su nombre, explicación y efecto sobre el match en el idioma de la interfaz, tanto en los flujos relacionales como en estadísticas y demo. El español conserva la tabla `es-ES`; el inglés utiliza la adaptación de la tabla inglesa siguiente. Cambiar de idioma conserva IDs, selección, borrador, validación, trayectos y significado. Las descripciones expresan lo declarado por la persona usuaria, no una verificación independiente de reciprocidad.

| ID | Clave interna | Texto visible `es-ES` | Explicación | Efecto destacado |
|---:|---|---|---|---|
| 1 | `NOT_TALKING` | Todavía no hemos hablado | Te interesa esta persona, pero todavía no habéis hablado ni tenido contacto directo. | No establece un match nuevo ni elimina uno anterior. |
| 2 | `ACQUAINTED` | Nos conocemos / hablamos | Ya os conocéis o habéis hablado, pero no identificas un interés romántico claro entre vosotros. | No establece un match nuevo ni elimina uno anterior. |
| 3 | `FRIENDS` | Solo somos amigos | Vuestra relación es de amistad. Puede que tú sientas algo más, pero no hay un acercamiento romántico o sexual compartido actualmente. | No establece un match nuevo ni elimina uno anterior. |
| 4 | `MUTUAL_INTEREST` | Creo que hay interés | Percibes señales de interés o coqueteo, pero todavía no tienes claro si es mutuo ni si ha habido un acercamiento romántico o sexual compartido. | No establece un match nuevo ni elimina uno anterior. |
| 5 | `DATING` | Nos estamos conociendo | Ya ha habido un acercamiento romántico o sexual compartido entre vosotros, pero todavía no tenéis claro qué queréis que sea. No implica que vaya a convertirse en algo casual, una amistad con derecho a roce o una relación de pareja. | Establece match. |
| 6 | `CASUAL` | Tenemos algo casual | Hay un acercamiento romántico o sexual compartido, puntual o continuado, sin una relación de pareja acordada. No implica sentimientos, exclusividad ni continuidad. | Establece match. |
| 7 | `FWB` | Amigos con derecho a roce | Tenéis una relación de amistad y también encuentros sexuales, sin consideraros pareja. No implica que ambos queráis una relación de pareja. | Establece match. |
| 8 | `RELATIONSHIP` | Somos pareja | Ambos reconocéis que tenéis una relación de pareja. No hace falta convivir ni estar casados y esta opción no presupone acuerdos de exclusividad. | Establece match. |
| 9 | `UNDEFINED` | Prefiero no decirlo | Prefieres no indicar en qué punto está vuestra historia. Esta opción no permite afirmar ni descartar reciprocidad. | No establece un match nuevo ni elimina uno anterior. |
| 10 | `CLOSED` | Historia cerrada | Das por terminada esta historia, haya habido un acercamiento entre vosotros o no. Si no queda evidencia de match y todos los períodos están cerrados, el resultado es «Sin match». | Cerrar no crea match ni elimina uno anterior. |

Adaptación inglesa de los mismos IDs:

| ID | Nombre inglés | Explicación inglesa |
|---:|---|---|
| 1 | We haven’t spoken yet | You’re interested in this person, but you haven’t spoken or had any direct contact yet. |
| 2 | We know each other / talk | You know each other or have spoken, but you don’t see clear signs of romantic interest between you. |
| 3 | We’re just friends | You’re friends. You may feel something more, but there’s no shared romantic or sexual involvement at the moment. |
| 4 | I think there’s interest | You notice signs of interest or flirting, but you’re not sure whether it’s mutual or whether there has been any shared romantic or sexual involvement. |
| 5 | We’re getting to know each other | There has already been shared romantic or sexual involvement between you, but you’re not sure what you want it to become yet. This doesn’t mean it will become something casual, a friendship with benefits or a relationship as a couple. |
| 6 | We have something casual | There’s shared romantic or sexual involvement, once or more often, without an agreed relationship as a couple. This doesn’t imply feelings, exclusivity or that it will continue. |
| 7 | Friends with benefits | You’re friends and also have sex, without considering yourselves a couple. This doesn’t mean you both want a relationship. |
| 8 | We’re a couple | You both recognise that you’re in a relationship. You don’t need to live together or be married, and this option doesn’t assume any agreement about exclusivity. |
| 9 | I’d rather not say | You’d rather not say where your story stands. This option doesn’t confirm or rule out mutual interest. |
| 10 | Story closed | You consider this story over, whether or not there was any romantic or sexual involvement. If no evidence of a match remains and all periods are closed, the result is “No match”. |

El efecto destacado inglés de 5/6/7/8 es «Establishes a match.». Para 1–4/9 es «Doesn’t establish a new match or remove an earlier one.» y para 10, «Closing doesn’t create a match or remove an earlier one.». Es la misma clasificación, sin inferir reciprocidad adicional.

Las situaciones no son una escalera que deba recorrerse en orden. En 1–4 puede haber contacto, amistad o coqueteo, pero esas opciones no declaran un acercamiento romántico o sexual compartido confirmado. La situación 5 sí lo declara, sin definir todavía la forma del vínculo; 6, 7 y 8 describen formas distintas de ese vínculo.

- **RP-SIT-005.** `RelationshipSituation` es la única dimensión del estado relacional. Cada ID representa directamente una situación completa y no requiere otra clasificación para interpretarse.
- **RP-SIT-006.** La persona puede consultar el significado y efecto sobre match de cada situación admitida. Consultar o cerrar las explicaciones conserva selección, validación y borrador, y utiliza las mismas opciones permitidas para alta, cambio o corrección.

<a id="inventario"></a>
## Inventario funcional

### Datos que guarda un período

Un período es el resumen vigente de un tramo continuo de la historia. Sólo existe un registro por período y sus datos permiten consultar su situación y sus fechas sin reconstruir todos sus hitos.

- **RP-REL-008.** Un período relacional se compone del siguiente inventario cerrado.

| Tratamiento | Dato exacto | Condición y uso autorizado |
| --- | --- | --- |
| API legible | Identidad y propiedad | Son obligatorios. El período pertenece siempre al mismo Crush y a la misma cuenta. |
| API legible | Inicio y cierre | El inicio es obligatorio. El cierre sólo tiene valor cuando la situación vigente es `CLOSED`. |
| API legible | Situación vigente | Es obligatoria y admite exactamente los IDs de `RelationshipSituation`. |
| Administrado por el sistema | Correlación | Une las partes de una misma acción compuesta sin sustituir sus identidades. |
| Administrado por el sistema | Ciclo de vida | No sustituyen las fechas funcionales. |

### Datos que guarda cada hito

Un hito es un acontecimiento concreto dentro de un período. El primer hito lo inicia y los posteriores registran cada cambio de situación; todos mantienen el vínculo obligatorio con la Experience que explica qué ocurrió.

- **RP-REL-009.** Un hito relacional se compone del siguiente inventario cerrado.

| Tratamiento | Dato exacto | Condición y uso autorizado |
| --- | --- | --- |
| API legible | Identidad y período | Son obligatorios e inmutables. |
| API legible | Experiencia vinculada | Es obligatorio y único: un hito tiene exactamente una experiencia y una experiencia se vincula como máximo a un hito. |
| API legible | Orden y día efectivo | Determinan el orden dentro del período; la fecha coincide con la de su experiencia. |
| API legible | Situación anterior y resultante | La anterior es nula sólo en el hito inicial. La resultante admite exactamente los IDs de `RelationshipSituation`. |
| API legible | Motivo de cierre | Sólo tiene valor cuando `situation_id` es `CLOSED`. |
| Administrado por el sistema | Correlación | Comparte la acción durable con su experiencia y, cuando corresponda, con el período. |
| Derivado por la API | Capacidades | Se calculan desde la historia completa. La app no las infiere desde una página parcial. |

- **RP-REL-010.** Situación, fechas, motivo y asociaciones forman la estructura relacional legible por la API. Los hitos conservan los acontecimientos de la historia; el período conserva su situación y fechas vigentes, sin repetir el motivo de cierre, que pertenece exclusivamente al hito `CLOSED`. Períodos e hitos no tienen un payload Vault propio. La explicación narrativa de cada hito pertenece a la experiencia vinculada: lugar y descripción siguen su inventario y tratamiento Vault.

<a id="inicial"></a>
## Bloque inicial del Crush

- **RP-REL-001.** Todo Crush confirmado comienza con exactamente un período y un hito inicial en `met_at`. El hito pertenece a ese período, conserva la situación inicial elegida y se vincula a la experiencia inicial definida por RP-INI-001..003. La atomicidad del alta completa se rige por RP-CRU-004.
- **RP-REL-007.** Al dar de alta el Crush, la situación inicial puede ser cualquiera de los IDs 1–9 y registra el punto real en el que se encuentra la historia. Al abrir un período posterior, `NOT_TALKING` (1) sólo sigue disponible cuando ningún hito confirmado y conservado de los períodos anteriores alcanzó jamás una situación 2–9; si la historia conservada ya demuestra que se conocieron o hablaron, el catálogo inicial posterior queda limitado a 2–9. Los hitos eliminados como corrección no cuentan como historia conservada. `CLOSED` no es una situación inicial: cerrar exige un hito posterior y una experiencia propia que explique qué ocurrió.
- **RP-TRA-006.** El hito inicial no cuenta como transición ni fabrica etapas anteriores. Puede establecer match si la situación elegida es 5, 6, 7 u 8.

<a id="periodos-hitos"></a>
## Períodos e hitos

- **RP-REL-002.** Cada Crush puede tener como máximo un período vivo. Toda experiencia posterior pertenece al período vivo hasta que un hito `CLOSED` lo termina.
- **RP-REL-003.** Toda experiencia pertenece al intervalo de su período: su fecha nunca es anterior a `start_date` ni posterior a `end_date` cuando el período está cerrado. Un período cerrado no admite nuevas experiencias ni hitos. Su `end_date` nunca puede quedar antes de una experiencia que ya contiene. La siguiente experiencia nueva que se confirme para esa historia crea un período nuevo y su hito inicial en la misma acción; el período anterior permanece cerrado e histórico.
- **RP-REL-004.** Puede registrarse como máximo una experiencia y un hito relacional por Crush y día civil. La unicidad se evalúa mediante la combinación (`crush_id`, `date`): una experiencia independiente ocupa ese día únicamente para su Crush, y las historias de otros Crushes pueden registrar sus propias experiencias o hitos en la misma fecha.
- **RP-REL-005.** Un hito nuevo sólo puede añadirse después del hito más reciente. No se inserta ni se crea historia intermedia.
- **RP-REL-006.** Un período que comienza y termina el mismo día tendría dos hitos el mismo día y, por tanto, no está permitido.

<a id="trayectos"></a>
## Trayectos permitidos

| Desde | Destinos admitidos al registrar un cambio |
|---|---|
| `NOT_TALKING` | `ACQUAINTED`, `FRIENDS`, `MUTUAL_INTEREST`, `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED`, `CLOSED` |
| `ACQUAINTED` | `FRIENDS`, `MUTUAL_INTEREST`, `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED`, `CLOSED` |
| `FRIENDS` | `MUTUAL_INTEREST`, `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED`, `CLOSED` |
| `MUTUAL_INTEREST` | `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED`, `CLOSED` |
| Cualquiera de `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED` | Cualquiera de las otras cuatro situaciones de ese grupo, `CLOSED` |
| `CLOSED` | Ninguna dentro del mismo período; retomar exige un período nuevo |

- **RP-TRA-002.** Desde una situación 1–4 puede elegirse directamente cualquier situación posterior de 1–9 que describa el estado actual, sin registrar ni fabricar hitos intermedios. La persona puede guardar la experiencia sin cambiar la situación conforme a `RP-TRA-007`; si declara un cambio, el destino se elige de forma explícita y MDL nunca lo deduce del texto ni de otros datos de la experiencia.
- **RP-TRA-003.** La API y la app aceptan únicamente los destinos de la tabla. La app muestra sólo los destinos válidos para el hito que se está creando o corrigiendo.
- **RP-TRA-004.** Dentro de un período no se retrocede desde una situación 5–9 a una situación 1–4. Cambiar entre 5–9 es un nuevo hito válido.
- **RP-TRA-005.** `CLOSED` es terminal para el período. Retomar la historia crea otro período.
- **RP-TRA-007.** Repetir la situación vigente no crea hito ni vínculo ficticio. La experiencia puede guardarse como independiente si el período continúa vivo.

<a id="evidencia-match"></a>
## Evidencia relacional para el match del Crush

- **RP-MAT-001.** La evidencia estructural de `MATCH` es un hito conservado cuya situación resultante sea 5, 6, 7 u 8. Las situaciones 1–4 y 9 no establecen match, pero tampoco retiran evidencia conservada de otros hitos. Textos, satisfacción, motivos de cierre e inferencias no sustituyen esa evidencia. Cerrar una historia sin evidencia conserva el resultado «Sin match» cuando todos sus períodos están cerrados, conforme a RP-MAT-006.
- **RP-MAT-003.** Los períodos e hitos eliminados como corrección no cuentan como evidencia para el resultado de match del Crush.

El resultado acumulado, los campos derivados y su recálculo pertenecen al [Crush](CRUSH.md#match).

<a id="cierre"></a>
## Cierre de un período

- **RP-CIE-001.** Cerrar exige un hito `CLOSED`, una experiencia completa, fecha y motivo válidos. La fecha del cierre debe ser posterior tanto al último hito como a todas las experiencias que el período ya contiene; ninguna experiencia existente se mueve, reasigna ni elimina para hacer posible el cierre. El guardado fija conjuntamente la experiencia de cierre, el hito con su `closure_reason_id`, el `end_date` del período y las proyecciones; antes de confirmarlo no existe cierre parcial.
- **RP-CIE-002.** El motivo catalogado expresa por qué termina el período. La descripción privada de la experiencia obligatoria vinculada al hito conserva el relato de qué ocurrió en ese cierre.
- **RP-CIE-003.** Desde las situaciones 1–4 se admiten `NO_MUTUAL_INTEREST`, `REJECTED_BY_ME`, `REJECTED_BY_THEM`, `FADED`, `MISSED_OPPORTUNITY` y `UNSPECIFIED`.
- **RP-CIE-004.** Desde las situaciones 5–9 se admiten `REJECTED_BY_ME`, `REJECTED_BY_THEM`, `FADED`, `UNSPECIFIED` y el motivo `ENDED_*` que corresponda exactamente a la situación anterior. En estas situaciones no se admiten `NO_MUTUAL_INTEREST` ni `MISSED_OPPORTUNITY`; la restricción depende de la situación anterior y no implica que 9 establezca match.
- **RP-CIE-005.** Cada motivo `ENDED_*` sólo es válido para su situación exacta. `CLOSED` no necesita conservar una etiqueta separada porque el hito mantiene `previous_situation_id`.
- **RP-CIE-007.** Se admiten exactamente estos once motivos:

| ID | Clave interna | Texto visible `es-ES` | Situación anterior admitida |
|---:|---|---|---|
| 1 | `NO_MUTUAL_INTEREST` | No hubo interés mutuo | 1–4 |
| 2 | `REJECTED_BY_ME` | No quise seguir | 1–9 |
| 3 | `REJECTED_BY_THEM` | La otra persona no quiso seguir | 1–9 |
| 4 | `FADED` | Se fue apagando | 1–9 |
| 5 | `MISSED_OPPORTUNITY` | Oportunidad perdida | 1–4 |
| 6 | `ENDED_DATING` | Terminó lo que había | `DATING` |
| 7 | `ENDED_CASUAL` | Terminó lo que había | `CASUAL` |
| 8 | `ENDED_FWB` | Terminó lo que había | `FWB` |
| 9 | `ENDED_RELATIONSHIP` | Terminó lo que había | `RELATIONSHIP` |
| 10 | `ENDED_UNDEFINED` | Terminó lo que había | `UNDEFINED` |
| 11 | `UNSPECIFIED` | Prefiero no especificar | 1–9 |

- **RP-CIE-008.** Al cerrar o corregir un cierre se elige entre siete motivos funcionales: finalización de lo que existía, ausencia de interés mutuo, decisión propia de no continuar, decisión de la otra persona, desvanecimiento del vínculo, oportunidad perdida o preferencia por no especificar. La finalización se distingue según la situación base confirmada mediante RP-CIE-005. Cada recorrido ofrece únicamente los motivos admitidos por RP-CIE-003/004 y conserva su significado en español e inglés.

<a id="cambios"></a>
## Crear y corregir hitos

- **RP-HIT-014.** No existe una acción directa separada «Cambiar situación». Toda situación posterior al bloque inicial se registra únicamente desde «Añadir experiencia»: primero se completan fecha, satisfacción, lugar y descripción y, conforme a RP-EXP-004 o RP-EXP-010, después se elige la situación resultante y, si corresponde, el motivo de cierre. La fecha de la Experience es también la fecha efectiva del hito. Una única confirmación crea Experience e hito —y, cuando corresponda, el período nuevo— o no crea ninguna parte.
- **RP-HIT-015.** Todos los hitos posteriores al bloque inicial nacen del mismo flujo de «Añadir experiencia» y conservan una única identidad, validación y estructura. Con período vivo, la persona decide al final del primer paso si hubo cambio de situación; con el período más reciente cerrado, el segundo paso exige una situación inicial admitida por la historia conforme a RP-REL-007 y no se ofrece esa decisión.
- **RP-HIT-001.** Todo período empieza con un hito inicial. Todo hito tiene exactamente una experiencia vinculada, comparte con ella fecha y correlación y conserva una identidad distinta. Coincidir en fecha nunca crea el vínculo.
- **RP-HIT-002.** Sólo se puede corregir el último hito del período más reciente. `Latest-only` compara hitos entre sí: una experiencia independiente posterior no convierte el hito en histórico ni impide corregirlo en general, pero sí puede impedir convertirlo en `CLOSED` conforme a RP-HIT-007. Los hitos de períodos anteriores y los que tienen otro hito posterior son históricos.
- **RP-HIT-004.** El hito inicial del bloque inicial puede corregirse mientras sea el último elegible, pero no eliminarse por separado. Su fecha sigue `met_at`.
- **RP-HIT-005.** Los destinos de una corrección se validan desde el hito inmediatamente anterior, no desde el valor que se corrige. Si se corrige el hito inicial, se aplica el catálogo inicial que RP-REL-007 permita para ese período según la historia confirmada anterior.
- **RP-HIT-006.** Elegir exactamente la situación anterior no produce un cambio nuevo. Para deshacer el último cambio se utiliza «Eliminar cambio».
- **RP-HIT-007.** Corregir conserva el ID, la fecha y la experiencia vinculada del hito. Cambia únicamente la situación y el motivo cuando corresponda, y recalcula las proyecciones en la misma transacción. Convertirlo en `CLOSED` sólo es válido cuando ninguna otra experiencia de su período tiene una fecha posterior a la fecha inmutable del hito. Si existe una, la corrección se rechaza completa: no mueve, elimina ni reasigna experiencias para fabricar un cierre compatible.

<a id="eliminacion"></a>
## Eliminar hitos o períodos

- **RP-HIT-003.** Sólo puede eliminarse por sí mismo el último hito del período más reciente cuando no sea el hito inicial. `Latest-only` compara hitos entre sí, por lo que las experiencias independientes posteriores no impiden esta acción. Los hitos históricos y cualquier hito que ya tenga otro hito posterior se rechazan sin mutación.
- **RP-HIT-008.** Eliminar el último hito posterior restaura la situación, el cierre y las proyecciones que resultan de la historia conservada.
- **RP-HIT-009.** Desde un hito posterior elegible se ofrecen dos acciones: «Eliminar cambio» elimina sólo el hito y conserva su experiencia como independiente dentro del mismo período; «Eliminar cambio y experiencia» elimina ambos atómicamente. Ninguna opción puede dejar un hito sin experiencia.
- **RP-HIT-010.** «Eliminar período» sólo está disponible para el período no inicial más reciente cuando su hito inicial sigue siendo también el último hito del período. Puede contener experiencias independientes posteriores: todas se eliminan junto con el hito inicial, su experiencia y el período porque ninguna puede quedar sin período. Si existe otro hito posterior, la operación se rechaza completa. El período anterior permanece cerrado y pasa a ser el más reciente; nunca se reabre. El período inicial del Crush sólo desaparece al eliminar el Crush.
- **RP-HIT-011.** Corregir o eliminar el hito de cierre del período más reciente puede devolver ese mismo período a su situación viva anterior. Nunca reabre un período anterior.
- **RP-HIT-012.** Ningún hito confirmado puede quedar sin su experiencia. Un hito inicial nunca se elimina aisladamente: el de un período posterior sólo desaparece con el período completo y el del bloque inicial, con el Crush.
- **RP-HIT-013.** Todas las operaciones vuelven a validar latest-only bajo bloqueo y actualizan período, Crush, asociaciones y actividad de forma atómica e idempotente. Un conflicto o rechazo no deja huérfanos ni mutaciones parciales.

<a id="almacenamiento"></a>
## Identidades y asociaciones

- **RP-REL-011.** Cada experiencia pertenece a un período y cada hito pertenece a ese mismo período y tiene una experiencia vinculada. Las asociaciones identifican inequívocamente las entidades relacionadas.
- **RP-REL-012.** Cada situación relacional se guarda mediante el identificador entero de su catálogo cerrado.

La corrección de la fecha de encuentro se rige por [Fecha de encuentro del Crush](CRUSH.md#fecha-de-encuentro). Los datos propios y el borrado iniciado desde una experiencia se rigen por [Experiencias](EXPERIENCES.md).

<a id="escenarios-explicaciones"></a>
## Escenarios de situaciones y explicaciones

| ID | Dado / Cuando / Entonces |
|---|---|
| GWT-SIT-EXPL-01 | Consultar y cerrar explicaciones conserva selección, borrador, errores y destinos admitidos. |
| GWT-SIT-EXPL-02 | Dadas las situaciones 1–9 como alta o reapertura elegible sin evidencia previa, sólo 5/6/7/8 establecen match; las demás quedan indefinidas mientras el período siga vivo. `CLOSED` no aparece como situación inicial. |
| GWT-SIT-EXPL-03 | Dada una historia sin evidencia, al cerrar todos sus períodos queda «Sin match»; al reabrir sin evidencia queda indefinida. Si conserva un hito 5/6/7/8, ni elegir 9 ni cerrar ni reabrir borra ese match. |
| GWT-SIT-EXPL-04 | Dada una corrección o eliminación que retira la única evidencia 5/6/7/8, resultado y primer match se recalculan desde la historia conservada de forma atómica. Con más evidencia se conserva la primera cronológica restante. |
| GWT-SIT-EXPL-05 | Dados dashboard, lecturas y demo, las etiquetas coinciden con el catálogo y el primer match sólo admite 5/6/7/8; API e iOS conservan el contrato de traducción actual y los demás indicadores no cambian. |
