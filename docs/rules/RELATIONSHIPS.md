# Reglas de producto — Relaciones

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Situación inicial, períodos, hitos, trayectos permitidos, cierre y corrección o eliminación del último elemento elegible.

<a id="inventario"></a>
## Inventario funcional de relaciones

- **RP-REL-008.** La historia relacional de la beta se compone de períodos e hitos con el siguiente inventario cerrado. Cada ampliación requiere una nueva decisión de Producto.

### Período relacional

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identificador del período | `id` | Lo genera MDL, es inmutable y mantiene unido el período con sus hitos. |
| API legible | Cuenta propietaria | `user_id` | Es inmutable y limita el período a su cuenta propietaria. |
| API legible | Crush propietario | `crush_id` | Es obligatorio e inmutable. Un período no cambia de Crush. |
| API legible | Fase vigente | `connection_phase_id` | Es obligatoria y admite únicamente los cinco IDs del catálogo de fases. |
| API legible | Etiqueta de vínculo vigente | `connection_label_id` | Sólo tiene valor cuando la historia alcanzó Involucrado; admite únicamente los cinco IDs del catálogo de etiquetas. |
| API legible | Motivo de cierre | `closure_reason_id` | Sólo tiene valor en un período cerrado y debe corresponder a la historia confirmada. |
| API legible | Inicio y cierre | `start_date`, `end_date` | `start_date` es obligatorio. `end_date` permanece vacío mientras el período está vivo y coincide con la fecha del hito de cierre cuando se cierra. |
| Administrado por el sistema | Experiencia asociada al inicio | `experience_id` | Identifica la experiencia que comparte la creación del período, cuando existe. |
| Administrado por el sistema | Correlación de la acción compuesta | `correlation_id` | Une las partes confirmadas en una misma acción sin sustituir sus identidades. |
| Administrado por el sistema | Fechas técnicas y retirada | `created_at`, `updated_at`, `deleted_at` | Sostienen orden, sincronización, corrección y ciclo de vida. No sustituyen `start_date` ni `end_date`. |
| Vault opaco | Nota de cierre | `closure_note` dentro del payload Vault | Es opcional, pertenece al período y sólo se muestra con el Vault desbloqueado. |

### Hito relacional

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identificador del hito | `id` | Es inmutable. Corregir el hito conserva este identificador. |
| API legible | Período al que pertenece | `relationship_id` | Es obligatorio e inmutable. |
| API legible | Día efectivo | `effective_date` | Es el día civil del acontecimiento y no cambia durante una corrección. |
| API legible | Situación anterior | `previous_connection_phase_id`, `previous_connection_label_id` | Conserva el punto de partida desde el que se valida y presenta una corrección. En el hito inicial ambos valores están vacíos. |
| API legible | Situación resultante | `connection_phase_id`, `connection_label_id` | Conserva el resultado real del hito conforme a los catálogos y trayectos permitidos. |
| API legible | Motivo resultante de cierre | `closure_reason_id` | Sólo tiene valor cuando el hito cierra el período. |
| Administrado por el sistema | Experiencia vinculada | `experience_id` | Identifica la experiencia registrada con ese hito, cuando existe. |
| Derivado por la API | Condición inicial, condición de último y acciones disponibles | `is_initial`, `is_latest`, `can_edit`, `can_delete` | Se calculan desde la historia completa. La aplicación no los deduce sólo de los elementos cargados en pantalla. |

- **RP-REL-009.** Fase, etiqueta, motivo, fechas y asociaciones son estructura funcional legible por la API. La nota de cierre es el único contenido privado propio de un período relacional y vive en su payload Vault.

<a id="inicial"></a>
## Situación relacional inicial

- **RP-SIT-001.** Toda alta de un Crush exige elegir una situación relacional inicial entre las situaciones visibles admitidas por MDL.
- **RP-SIT-002.** La situación inicial la declara expresamente la persona. No se infiere de Ducky, la experiencia inicial, la satisfacción, la actividad, las estadísticas ni sistemas de IA.
- **RP-REL-001.** La situación inicial crea el período relacional inicial obligatorio con fecha `met_at`. Ese período no puede eliminarse de manera independiente.
- **RP-REL-007.** La situación inicial describe el estado que ya existe al registrar el Crush, incluida una historia ya cerrada. No fabrica etapas ni hitos anteriores.
- **RP-TRA-006.** Elegir la situación inicial durante el alta no constituye un avance relacional. Las restricciones de trayectos se aplican a los cambios registrados después del alta.

<a id="periodos-hitos"></a>
## Períodos e hitos relacionales

- **RP-REL-002.** Sólo puede existir un período vivo por Crush. Los avances, cambios de etiqueta y cierre modifican ese período; crear otro no lo cierra ni reemplaza implícitamente.
- **RP-REL-003.** Retomar la historia después de un cierre crea expresamente otro período con fecha posterior al cierre. El período anterior permanece cerrado.
- **RP-REL-004.** Puede registrarse como máximo un hito relacional por Crush y día. La creación del período ocupa ese día; una experiencia y un hito sí pueden compartirlo.
- **RP-REL-005.** Un hito nuevo sólo puede añadirse después del hito más reciente. No puede insertarse en medio de la historia.
- **RP-REL-006.** Un período que comienza y termina el mismo día tiene un día de duración; esto no permite registrar dos hitos en ese día.

<a id="trayectos"></a>
## Situaciones y trayectos permitidos

| Situación visible | Fase (`connection_phase_id`) | Etiqueta (`connection_label_id`) |
|---|---|---|
| Todavía no hablamos | `1 interested` | — |
| Nos conocemos / hablamos | `2 acquainted` | — |
| Hay onda, pero aún no salimos | `3 mutual_interest` | — |
| Estamos saliendo | `4 involved` | `1 dating` |
| Algo casual | `4 involved` | `2 casual` |
| Amigos con beneficios | `4 involved` | `3 fwb` |
| Estamos en una relación | `4 involved` | `4 relationship` |
| Hay algo, prefiero no etiquetarlo | `4 involved` | `5 other` |
| Historia cerrada | `5 closed` | Conserva la última etiqueta confirmada cuando la historia llegó a Involucrado. |

- **RP-TRA-001.** «Involucrado» es una fase interna; nunca aparece como una décima situación visible sin indicar el tipo de vínculo.
- **RP-TRA-002.** Los avances siguen etapas lógicas consecutivas: Todavía no hablamos → Nos conocemos/hablamos → Hay onda → una opción de vínculo. No puede registrarse directamente una relación desde «Todavía no hablamos».
- **RP-TRA-003.** El producto sólo muestra y acepta los destinos válidos de la tabla.
- **RP-TRA-004.** No se retrocede de fase. Cambiar etiqueta dentro de Involucrado no es retroceso de fase.
- **RP-TRA-005.** Se puede cerrar desde cualquier fase viva. Cerrado es terminal para avances; retomar exige otro período.
- **RP-TRA-007.** Repetir situación sin cambio real no crea hito ni vínculo ficticio a experiencia.
- **RP-TRA-008.** Fase y etiqueta del mismo guardado forman un único hito correlacionado.

| Desde | Destinos al registrar un cambio nuevo |
|---|---|
| Todavía no hablamos | Nos conocemos/hablamos; Historia cerrada |
| Nos conocemos/hablamos | Hay onda; Historia cerrada |
| Hay onda | Estamos saliendo; Algo casual; FWB; Estamos en una relación; Hay algo sin etiqueta; Historia cerrada |
| Cualquiera de las cinco etiquetas de vínculo | Cualquiera de las otras cuatro etiquetas; Historia cerrada |
| Historia cerrada | Ningún avance en ese período; crear otro explícitamente en fecha válida |

<a id="cierre"></a>
## Cierre de una relación

- **RP-CIE-001.** Cerrar una relación exige una fecha y un motivo válidos. Mientras el período siga vivo, no tiene motivo de cierre.
- **RP-CIE-002.** La nota de cierre es opcional para cualquier motivo y vive exclusivamente en el Vault.
- **RP-CIE-003.** Antes de Involucrado se admiten falta de interés mutuo, rechazo por cualquiera, desvanecimiento, oportunidad perdida o sin detalle.
- **RP-CIE-004.** Desde Involucrado se admiten rechazo por cualquiera, desvanecimiento, sin detalle o fin del vínculo; no falta de interés mutuo/oportunidad perdida.
- **RP-CIE-005.** «Terminó lo que había» utiliza el motivo correspondiente al tipo de vínculo confirmado. No usa un motivo genérico ni un tipo de vínculo que todavía esté pendiente.
- **RP-CIE-006.** Una historia que se registra inicialmente como cerrada no puede inventar un vínculo anterior ni utilizar un motivo `ENDED_*` sin ese vínculo registrado.
- **RP-CIE-007.** Sólo se admiten los once motivos de la tabla siguiente.

| ID | Motivo | Contexto |
|---|---|---|
| 1 | NO_MUTUAL_INTEREST — falta de interés mutuo | Antes de Involucrado / alta cerrada |
| 2 | REJECTED_BY_ME — rechazo por mí | Cualquiera |
| 3 | REJECTED_BY_THEM — rechazo por la otra persona | Cualquiera |
| 4 | FADED — se fue apagando | Cualquiera |
| 5 | MISSED_OPPORTUNITY — oportunidad perdida | Antes de Involucrado / alta cerrada |
| 6 | ENDED_DATING — terminaron las citas | Citas confirmada |
| 7 | ENDED_CASUAL — terminó lo casual | Casual confirmada |
| 8 | ENDED_FWB — terminó FWB | FWB confirmada |
| 9 | ENDED_RELATIONSHIP — terminó la relación | Relación confirmada |
| 10 | ENDED_UNDEFINED — terminó vínculo sin etiqueta | Otro confirmada |
| 11 | UNSPECIFIED — sin especificar | Cualquiera |

<a id="cambios"></a>
## Registrar, corregir y eliminar hitos o períodos

- **RP-HIT-001.** Registrar un cambio crea un hito. Corregirlo modifica ese mismo hito y no añade un hecho nuevo a la historia.
- **RP-HIT-002.** Sólo puede corregirse el último hito confirmado del período relacional más reciente. El último hito de un período anterior permanece histórico y no es editable.
- **RP-HIT-003.** Sólo puede eliminarse el último hito cuando no sea el hito de creación. Los hitos históricos no ofrecen acciones de edición o eliminación.
- **RP-HIT-004.** La situación inicial puede corregirse mientras continúe siendo el último hito elegible, pero nunca eliminarse. Su fecha permanece vinculada a `met_at`.
- **RP-HIT-005.** Las opciones disponibles al corregir un hito se calculan desde el hito inmediatamente anterior, no desde el estado erróneo que se está corrigiendo.
- **RP-HIT-006.** Elegir exactamente la situación anterior no produce un cambio nuevo. Para deshacer el último cambio se utiliza «Eliminar cambio».
- **RP-HIT-007.** Corregir un hito no permite cambiar su fecha: el inicial sigue `met_at`, el vinculado sigue la fecha de su experiencia y el no vinculado conserva la fecha registrada.
- **RP-HIT-008.** Eliminar el último hito posterior restaura conjuntamente la fase, la etiqueta y el cierre que resultaban del hito anterior. Ese hito anterior pasa a ser el último elegible.
- **RP-HIT-009.** Si el hito eliminado estaba vinculado a una experiencia, la experiencia se conserva sin vínculo, con la misma identidad y fecha.
- **RP-HIT-010.** Un período completo sólo puede eliminarse como corrección si es el más reciente, está cerrado y no es el período inicial. Cualquier período anterior permanece cerrado.
- **RP-HIT-011.** Corregir o eliminar el último hito de cierre puede restaurar el estado anterior de ese mismo período reciente. No reabre automáticamente otro período anterior.
- **RP-HIT-012.** Eliminar una experiencia o editar sus datos propios nunca reescribe el hito relacional al que estuvo vinculada.
- **RP-HIT-013.** Ningún canal ni acción concurrente puede eludir estas reglas. Un rechazo conserva íntegramente el estado anterior.

<a id="coincidencia-historica"></a>
## Coincidencia histórica (`everMatched`)

- **BR-7.01.** `everMatched` deriva únicamente de que el Crush haya alcanzado la fase Involucrado en su historia registrada. Un avance o cierre posterior no lo hace retroceder.
- **RP-MAT-001.** La evidencia de `everMatched` es un período o hito confirmado que alcanzó `4 involved`. Textos, experiencias y motivos de cierre no sustituyen esa evidencia.
- **RP-MAT-002.** El filtro de coincidencia utiliza exactamente dos resultados: el Crush alcanzó Involucrado alguna vez o todavía no lo alcanzó. Un cierre conserva el primer resultado cuando ya existía.
- **RP-MAT-003.** Los períodos eliminados como corrección no cuentan como evidencia estadística. El efecto de corregir o eliminar el único hito que acreditaba Involucrado está pendiente en [Decisiones pendientes](../PRODUCT.md#decisiones-pendientes).

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-06"></a>DEC-06 | Corregir/eliminar único hito que acreditó Involucrado: efecto en everMatched. | Cierre normal no revierte match; período correctivamente borrado no cuenta. |
| <a id="dec-13"></a>DEC-13 | Reconciliar el expediente Legal con los trayectos, motivos y corrección latest-only definidos en este documento. | Las definiciones funcionales actuales están aquí; Legal no se modifica sin revisión humana ni se declara resuelto el requisito de lanzamiento. |

La corrección de la fecha de encuentro se rige por [Fecha de encuentro del Crush](CRUSH.md#fecha-de-encuentro).
