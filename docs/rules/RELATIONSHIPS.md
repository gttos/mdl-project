# Reglas de producto — Relaciones

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Situaciones, períodos, hitos, match, cierre, corrección y eliminación.

Una **situación relacional** describe el estado exacto y visible de la historia en un momento. Un **período relacional** contiene un tramo continuo de esa historia. Un **hito relacional** crea el período o cambia su situación. Cada hito tiene exactamente una experiencia vinculada que explica qué ocurrió; una experiencia puede existir sin hito, pero siempre pertenece a un período.

Los nombres internos, los campos HTTP y las claves de los catálogos se expresan en inglés. Los textos que ve la persona en la beta se presentan en español de España (`es-ES`). Traducir la interfaz nunca cambia el ID ni el significado funcional guardado.

<a id="situaciones"></a>
## Situaciones relacionales

- **RP-SIT-001.** Al dar de alta un Crush, la persona elige una de las situaciones iniciales admitidas 1–8 para describir el punto real en el que se encuentra la historia.
- **RP-SIT-002.** MDL utiliza siempre la situación exacta elegida por la persona. No la deduce de Ducky, satisfacción, texto privado, actividad, estadísticas ni IA.
- **RP-SIT-003.** `RelationshipSituation` es un catálogo cerrado implementado como Value Object en la API y como enum equivalente en iOS. No existe una tabla de catálogo: `current_situation_id`, `previous_situation_id`, `situation_id` y `first_match_situation_id` guardan únicamente el entero que corresponda.
- **RP-SIT-004.** Las claves internas permanecen en inglés y nunca se muestran como copy. En la beta, iOS resuelve cada ID al texto `es-ES` exacto de esta tabla.

| ID | Clave interna | Texto visible `es-ES` | Efecto sobre match |
|---:|---|---|---|
| 1 | `NOT_TALKING` | Todavía no hemos hablado | No lo establece |
| 2 | `ACQUAINTED` | Nos conocemos / hablamos | No lo establece |
| 3 | `MUTUAL_INTEREST` | Hay química, pero aún no hemos quedado | No lo establece |
| 4 | `DATING` | Estamos saliendo | Establece match |
| 5 | `CASUAL` | Algo casual | Establece match |
| 6 | `FWB` | Amigos con derecho | Establece match |
| 7 | `RELATIONSHIP` | Estamos en una relación | Establece match |
| 8 | `UNDEFINED` | Hay algo, pero prefiero no etiquetarlo | Establece match |
| 9 | `CLOSED` | Historia cerrada | Cierra; no crea ni borra match |

- **RP-SIT-005.** `RelationshipSituation` es la única dimensión del estado relacional. Cada ID representa directamente una situación completa y no requiere otra clasificación para interpretarse.

<a id="inventario"></a>
## Inventario funcional

### Datos que guarda un período

Un período es el resumen vigente de un tramo continuo de la historia. Sólo existe un registro por período y sus datos permiten consultar su situación, fechas y resultado sin reconstruir todos sus hitos.

- **RP-REL-008.** Un período relacional se compone del siguiente inventario cerrado.

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identidad y propiedad | `id`, `user_id`, `crush_id` | Son obligatorios. El período pertenece siempre al mismo Crush y a la misma cuenta. |
| API legible | Inicio y cierre | `start_date`, `end_date` | El inicio es obligatorio. El cierre sólo tiene valor cuando la situación vigente es `CLOSED`. |
| API legible | Situación vigente | `current_situation_id` | Es obligatoria y admite exactamente los IDs de `RelationshipSituation`. |
| Administrado por el sistema | Correlación | `correlation_id` | Une las partes de una misma acción compuesta sin sustituir sus identidades. |
| Administrado por el sistema | Ciclo de vida | `created_at`, `updated_at`, `deleted_at` | No sustituyen las fechas funcionales. |

### Datos que guarda cada hito

Un hito es un acontecimiento concreto dentro de un período. El primer hito lo inicia y los posteriores registran cada cambio de situación; todos mantienen el vínculo obligatorio con la Experience que explica qué ocurrió.

- **RP-REL-009.** Un hito relacional se compone del siguiente inventario cerrado.

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identidad y período | `id`, `relationship_period_id` | Son obligatorios e inmutables. |
| API legible | Experiencia vinculada | `experience_id` | Es obligatorio y único: un hito tiene exactamente una experiencia y una experiencia se vincula como máximo a un hito. |
| API legible | Orden y día efectivo | `sequence`, `effective_date` | Determinan el orden dentro del período; la fecha coincide con la de su experiencia. |
| API legible | Situación anterior y resultante | `previous_situation_id`, `situation_id` | La anterior es nula sólo en el hito inicial. La resultante admite exactamente los IDs de `RelationshipSituation`. |
| API legible | Motivo de cierre | `closure_reason_id` | Sólo tiene valor cuando `situation_id` es `CLOSED`. |
| Administrado por el sistema | Correlación | `correlation_id` | Comparte la acción durable con su experiencia y, cuando corresponda, con el período. |
| Derivado por la API | Capacidades | `is_initial`, `is_latest`, `can_edit`, `can_delete_change`, `can_delete_with_experience` | Se calculan desde la historia completa. La app no las infiere desde una página parcial. |

- **RP-REL-010.** Situación, fechas, motivo y asociaciones forman la estructura funcional legible por la API. Los hitos conservan los acontecimientos de la historia; el período conserva su situación y fechas vigentes, sin repetir el motivo de cierre, que pertenece exclusivamente al hito `CLOSED`. El match y su primer hito pertenecen al Crush. La explicación narrativa de cada hito pertenece a la experiencia vinculada: lugar y descripción siguen su inventario y tratamiento Vault.

<a id="inicial"></a>
## Bloque inicial del Crush

- **RP-REL-001.** Confirmar un Crush crea atómicamente el Crush, su período inicial, el hito inicial y la experiencia inicial completa en `met_at`. Si falta o falla una parte, no se crea ninguna.
- **RP-REL-007.** La situación inicial puede ser cualquiera de los IDs 1–8 y registra el punto real en el que se encuentra la historia al dar de alta el Crush. `CLOSED` no es una situación inicial: cerrar exige un hito posterior y una experiencia propia que explique qué ocurrió.
- **RP-TRA-006.** El hito inicial no cuenta como transición ni fabrica etapas anteriores. Puede establecer match si la situación elegida es 4–8.

<a id="periodos-hitos"></a>
## Períodos e hitos

- **RP-REL-002.** Cada Crush puede tener como máximo un período vivo. Toda experiencia posterior pertenece al período vivo hasta que un hito `CLOSED` lo termina.
- **RP-REL-003.** Si el período más reciente está cerrado, permanecen disponibles «Añadir experiencia» y «Cambiar situación». Ambos crean obligatoriamente una experiencia, un período nuevo y su hito inicial en la misma acción atómica; sólo cambia el orden del formulario conforme a RP-EXP-010 y RP-HIT-014. El período anterior permanece cerrado e histórico.
- **RP-REL-004.** Puede registrarse como máximo una experiencia y un hito relacional por Crush y día civil. Una experiencia independiente ocupa el día aunque no cree hito.
- **RP-REL-005.** Un hito nuevo sólo puede añadirse después del hito más reciente. No se inserta ni se crea historia intermedia.
- **RP-REL-006.** Un período que comienza y termina el mismo día tendría dos hitos el mismo día y, por tanto, no está permitido.

<a id="trayectos"></a>
## Trayectos permitidos

| Desde | Destinos admitidos al registrar un cambio |
|---|---|
| `NOT_TALKING` | `ACQUAINTED`, `CLOSED` |
| `ACQUAINTED` | `MUTUAL_INTEREST`, `CLOSED` |
| `MUTUAL_INTEREST` | `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED`, `CLOSED` |
| Cualquiera de `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED` | Cualquiera de las otras cuatro situaciones de ese grupo, `CLOSED` |
| `CLOSED` | Ninguna dentro del mismo período; retomar exige un período nuevo |

- **RP-TRA-002.** No se salta desde `NOT_TALKING` o `ACQUAINTED` directamente a una situación 4–8 ni se fabrican hitos intermedios.
- **RP-TRA-003.** La API y la app aceptan únicamente los destinos de la tabla. La app muestra sólo los destinos válidos para el hito que se está creando o corrigiendo.
- **RP-TRA-004.** Dentro de un período no se retrocede desde una situación 4–8 a una situación 1–3. Cambiar entre 4–8 es un nuevo hito válido.
- **RP-TRA-005.** `CLOSED` es terminal para el período. Retomar la historia crea otro período.
- **RP-TRA-007.** Repetir la situación vigente no crea hito ni vínculo ficticio. La experiencia puede guardarse como independiente si el período continúa vivo.

<a id="match"></a>
## Resultado de match

- **RP-MAT-001.** La evidencia estructural de `MATCH` es un hito conservado cuya situación resultante sea cualquiera de 4–8. Textos, satisfacción, motivos de cierre e inferencias no sustituyen esa evidencia.
- **RP-MAT-003.** Los períodos e hitos eliminados como corrección no cuentan como evidencia estadística ni relacional.
- **RP-MAT-004.** `MatchStatus` es un Value Object cerrado con `0 UNKNOWN`, `1 MATCH` y `2 NO_MATCH`. `match_status_id` se guarda únicamente en el Crush para que filtros, conteos y estadísticas no reconstruyan toda la historia.
- **RP-MAT-005.** Entrar por primera vez en una situación 4–8 mediante un hito conservado fija en el Crush `MATCH`, `first_match_situation_id` y `first_match_date`. Un período posterior no reinicia ese resultado. Si se corrige o elimina la única evidencia que lo establecía, las tres proyecciones se recalculan desde los hitos conservados.
- **RP-MAT-006.** Si el Crush no tiene evidencia conservada de match, su resultado es `UNKNOWN` mientras su período más reciente está vivo y `NO_MATCH` cuando todos sus períodos están cerrados. Abrir un período nuevo sin match transforma `NO_MATCH` en `UNKNOWN`; un match posterior lo transforma en `MATCH`.
- **RP-MAT-007.** `first_match_situation_id` y `first_match_date` del Crush identifican el primer match cronológico entre sus períodos conservados. Match y primer match se consultan exclusivamente mediante estas proyecciones explícitas.
- **RP-MAT-008.** Crear, corregir o eliminar un hito o período recalcula dentro de la misma transacción `current_situation_id` y las fechas del período afectado, y `match_status_id`, `first_match_situation_id` y `first_match_date` del Crush desde los hitos que permanecen.

<a id="cierre"></a>
## Cierre de un período

- **RP-CIE-001.** Cerrar exige un hito `CLOSED`, una experiencia completa, fecha y motivo válidos. El guardado fija conjuntamente el hito con su `closure_reason_id`, el `end_date` del período y las proyecciones; antes de confirmarlo no existe cierre parcial.
- **RP-CIE-002.** El motivo catalogado expresa por qué termina el período. La descripción privada de la experiencia obligatoria vinculada al hito conserva el relato de qué ocurrió en ese cierre.
- **RP-CIE-003.** Desde las situaciones 1–3 se admiten `NO_MUTUAL_INTEREST`, `REJECTED_BY_ME`, `REJECTED_BY_THEM`, `FADED`, `MISSED_OPPORTUNITY` y `UNSPECIFIED`.
- **RP-CIE-004.** Desde las situaciones 4–8 se admiten `REJECTED_BY_ME`, `REJECTED_BY_THEM`, `FADED`, `UNSPECIFIED` y el motivo `ENDED_*` que corresponda exactamente a la situación anterior. No se admiten `NO_MUTUAL_INTEREST` ni `MISSED_OPPORTUNITY` después del match.
- **RP-CIE-005.** Cada motivo `ENDED_*` sólo es válido para su situación exacta. `CLOSED` no necesita conservar una etiqueta separada porque el hito mantiene `previous_situation_id`.
- **RP-CIE-007.** Se admiten exactamente estos once motivos:

| ID | Clave interna | Texto visible `es-ES` | Situación anterior admitida |
|---:|---|---|---|
| 1 | `NO_MUTUAL_INTEREST` | No hubo interés mutuo | 1–3 |
| 2 | `REJECTED_BY_ME` | Lo rechacé yo | 1–8 |
| 3 | `REJECTED_BY_THEM` | Me rechazó la otra persona | 1–8 |
| 4 | `FADED` | Se fue apagando | 1–8 |
| 5 | `MISSED_OPPORTUNITY` | Fue una oportunidad perdida | 1–3 |
| 6 | `ENDED_DATING` | Dejamos de salir | `DATING` |
| 7 | `ENDED_CASUAL` | Terminó lo casual | `CASUAL` |
| 8 | `ENDED_FWB` | Terminó lo de amigos con derecho | `FWB` |
| 9 | `ENDED_RELATIONSHIP` | Terminó la relación | `RELATIONSHIP` |
| 10 | `ENDED_UNDEFINED` | Terminó lo que había | `UNDEFINED` |
| 11 | `UNSPECIFIED` | Prefiero no especificarlo | 1–8 |

<a id="cambios"></a>
## Crear y corregir hitos

- **RP-HIT-014.** La acción directa «Cambiar situación» es un formulario obligatorio de dos pasos: primero se elige la situación resultante y, si corresponde, el motivo de cierre; después se completan fecha, satisfacción, lugar y descripción de la experiencia que explica qué ocurrió. Una única confirmación crea experiencia e hito o no crea ninguna parte. Si el período anterior estaba cerrado, la situación elegida debe ser inicial 1–8 y la misma acción crea transparentemente el período nuevo.
- **RP-HIT-015.** «Añadir experiencia» y «Cambiar situación» conservan entradas distintas aunque puedan producir el mismo compuesto estructural. El primer recorrido empieza por la experiencia y sólo añade o exige el paso de situación conforme a RP-EXP-004 y RP-EXP-010; el segundo empieza siempre por la situación y exige después la experiencia conforme a RP-HIT-014.
- **RP-HIT-001.** Todo período empieza con un hito inicial. Todo hito tiene exactamente una experiencia vinculada, comparte con ella fecha y correlación y conserva una identidad distinta. Coincidir en fecha nunca crea el vínculo.
- **RP-HIT-002.** Sólo se puede corregir el último hito del período más reciente. `Latest-only` compara hitos entre sí: una experiencia independiente posterior no convierte el hito en histórico ni impide corregirlo. Los hitos de períodos anteriores y los que tienen otro hito posterior son históricos.
- **RP-HIT-004.** El hito inicial del bloque inicial puede corregirse mientras sea el último elegible, pero no eliminarse por separado. Su fecha sigue `met_at`.
- **RP-HIT-005.** Los destinos de una corrección se validan desde el hito inmediatamente anterior, no desde el valor que se corrige. Si se corrige el hito inicial, se aplican las situaciones iniciales 1–8.
- **RP-HIT-006.** Elegir exactamente la situación anterior no produce un cambio nuevo. Para deshacer el último cambio se utiliza «Eliminar cambio».
- **RP-HIT-007.** Corregir conserva el ID, la fecha y la experiencia vinculada del hito. Cambia únicamente la situación y el motivo cuando corresponda, y recalcula las proyecciones en la misma transacción.

<a id="eliminacion"></a>
## Eliminar hitos o períodos

- **RP-HIT-003.** Sólo puede eliminarse por sí mismo el último hito del período más reciente cuando no sea el hito inicial. `Latest-only` compara hitos entre sí, por lo que las experiencias independientes posteriores no impiden esta acción. Los hitos históricos y cualquier hito que ya tenga otro hito posterior se rechazan sin mutación.
- **RP-HIT-008.** Eliminar el último hito posterior restaura la situación, el cierre y las proyecciones que resultan de la historia conservada.
- **RP-HIT-009.** Desde un hito posterior elegible se ofrecen dos acciones: «Eliminar cambio» elimina sólo el hito y conserva su experiencia como independiente dentro del mismo período; «Eliminar cambio y experiencia» elimina ambos atómicamente. Una experiencia vinculada inicia siempre la segunda acción. Ninguna opción puede dejar un hito sin experiencia.
- **RP-HIT-010.** El período no inicial más reciente sólo puede eliminarse desde su hito inicial o desde la experiencia vinculada a éste cuando el hito inicial siga siendo también el último hito del período. Puede contener experiencias independientes posteriores: todas se eliminan junto con el hito inicial, su experiencia y el período porque ninguna puede quedar sin período. Si existe otro hito posterior, la operación se rechaza completa. La confirmación advierte que se eliminarán el período y todas sus experiencias. El período anterior permanece cerrado y pasa a ser el más reciente; nunca se reabre. El período inicial del Crush sólo desaparece al eliminar el Crush.
- **RP-HIT-011.** Corregir o eliminar el hito de cierre del período más reciente puede devolver ese mismo período a su situación viva anterior. Nunca reabre un período anterior.
- **RP-HIT-012.** Ningún hito confirmado puede quedar sin su experiencia. Un hito inicial nunca se elimina aisladamente: el de un período posterior sólo desaparece con el período completo y el del bloque inicial, con el Crush.
- **RP-HIT-013.** Todas las operaciones vuelven a validar latest-only bajo bloqueo y actualizan período, Crush, asociaciones y actividad de forma atómica e idempotente. Un conflicto o rechazo no deja huérfanos ni mutaciones parciales.

<a id="almacenamiento"></a>
## Modelo físico objetivo

- **RP-REL-011.** El esquema funcional utiliza exactamente `relationship_periods`, `relationship_milestones` y `experiences` para representar períodos, hitos y experiencias. `relationship_milestones.relationship_period_id` enlaza cada hito con su período; `experiences.relationship_period_id` enlaza cada experiencia con ese mismo período; y `relationship_milestones.experience_id` enlaza el hito con su experiencia obligatoria.
- **RP-REL-012.** La API recibe y guarda los IDs enteros definidos por `RelationshipSituation` y `MatchStatus`. Ambos catálogos viven en Value Objects del dominio y no en tablas de base de datos.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-13"></a>DEC-13 | Reconciliar el expediente Legal con las situaciones, Experience obligatoria para cada hito, proyecciones de match y borrado compuesto latest-only definidos aquí. | Product define el contrato funcional; Legal no se modifica sin revisión humana y su contradicción continúa bloqueando el lanzamiento. |

La corrección de la fecha de encuentro se rige por [Fecha de encuentro del Crush](CRUSH.md#fecha-de-encuentro). Los datos propios y el borrado iniciado desde una experiencia se rigen por [Experiencias](EXPERIENCES.md).
