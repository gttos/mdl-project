# Reglas de producto — Timeline y Home

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Presentación de la historia, orden, agrupación, acciones disponibles y actividad reciente de Home.

<a id="contenido"></a>
## Contenido e identidad

- **RP-TIM-020.** Cada elemento del Timeline identifica exactamente un día civil, un Crush y uno de estos acontecimientos: bloque inicial; experiencia independiente dentro de un período; creación de un período posterior; o hito de cambio de situación o cierre. Un cambio pendiente representa provisionalmente uno de esos mismos acontecimientos y conserva su identidad.
- **RP-TIM-021.** La estructura legible de un elemento contiene `id`, `type`, `date` y la referencia al Crush. Añade `experience`, `relationship_period` y `relationship_milestone` únicamente cuando ese acontecimiento los representa. `relationship_milestone` contiene la identidad, fecha, situación anterior, situación resultante, motivo de cierre, vínculo obligatorio con su experiencia y capacidades calculadas definidas en el [inventario relacional](RELATIONSHIPS.md#inventario).
- **RP-TIM-001.** El Timeline muestra la historia realmente registrada y nunca inventa acontecimientos para completar huecos.
- **RP-TIM-002.** El Timeline de un Crush sólo contiene acontecimientos de ese Crush. El Timeline global reúne los de la misma cuenta conservando su contexto.
- **RP-TIM-003.** Los elementos eliminados o retirados del uso ordinario y su contenido dependiente no aparecen en el Timeline.
- **RP-TIM-004.** El alta del Crush, su período relacional inicial, su hito inicial y su experiencia inicial se muestran como un único bloque. Ninguna de esas partes aparece además como un acontecimiento separado.
- **RP-TIM-005.** La creación del período muestra la situación que se registró inicialmente y no cambia su representación cuando la relación avanza después.
- **RP-TIM-006.** Cada cambio real aparece una sola vez. La situación elegida y su experiencia vinculada forman un único acontecimiento lógico, aunque conserven identidades estructurales distintas.
- **RP-TIM-007.** El contexto relacional de una experiencia independiente se obtiene de los hitos confirmados de su propio período hasta esa fecha, no del estado relacional actual ni de inferir otro período sólo por coincidencia de fechas.
- **RP-TIM-008.** Un cambio pendiente se muestra una sola vez y sustituye visualmente la versión confirmada del mismo acontecimiento; nunca aparecen ambas a la vez.
- **RP-TIM-009.** Una edición pendiente muestra el valor propuesto. Descartarla restaura el valor confirmado sin duplicados.

<a id="orden"></a>
## Orden de los acontecimientos

- **RP-TIM-010.** El Timeline se ordena por día civil descendente y mantiene un orden estable al cargar más contenido, sin perder, duplicar ni separar partes de un mismo acontecimiento.
- **RP-TIM-011.** El bloque inicial usa la fecha de encuentro; cada experiencia usa su fecha; cada período, su inicio; y cada hito, el día en que ocurrió.
- **RP-TIM-012.** Toda fecha válida se presenta como día civil. El marcador de ausencia se utiliza únicamente cuando el dato falta o no puede interpretarse con seguridad.
- **RP-TIM-013.** Cuando falta información suficiente para modificar un acontecimiento con seguridad, se muestra en modo de sólo lectura y no se inventan identidades ni fechas.

<a id="acciones"></a>
## Acciones disponibles

- **RP-TIM-014.** Cada acontecimiento muestra únicamente las acciones permitidas por su dominio: una experiencia confirmada puede editarse y sólo ofrece eliminarse conforme a RP-EXP-019..021; un hito relacional sólo ofrece editar o eliminar cuando la API publica la capacidad latest-only correspondiente conforme a RP-HIT-002..003. En un hito posterior vinculado, Timeline distingue «Eliminar cambio», que conserva la experiencia como independiente dentro del mismo período, de «Eliminar cambio y experiencia», que retira ambas partes atómicamente. El hito inicial nunca ofrece eliminarse de manera aislada.
- **RP-TIM-015.** El bloque inicial permite editar el Crush, la experiencia inicial y la situación relacional inicial según sus reglas. No ofrece eliminar por separado la experiencia, el período ni el hito inicial; el borrado completo del Crush se inicia exclusivamente desde la acción correspondiente del listado.
- **RP-TIM-016.** Una experiencia independiente permite editarla o eliminarla conforme a sus propias reglas. No ofrece una acción para iniciar otro período desde ella: ya pertenece a un período, y la coincidencia de fechas no la reasigna ni crea un hito automáticamente.
- **RP-TIM-017.** La creación de un período posterior se presenta junto con la experiencia y el hito inicial nacidos en la misma acción atómica. No puede vincularse después una experiencia independiente preexistente para crear ese período ni para reescribir historia.
- **RP-TIM-018.** Un cambio pendiente en el Timeline de un Crush bloquea nuevas acciones incompatibles de ese mismo Crush. No impide consultar o editar el perfil del Crush, gestionar sus fotos, navegar, archivarlo, eliminar una experiencia independiente no afectada ni trabajar con otros Crushes. El borrado de una experiencia vinculada a un hito espera cualquier operación que pueda alterar su elegibilidad latest-only.
- **RP-TIM-019.** Una edición pendiente del perfil del Crush sólo limita su Timeline cuando modifica realmente `met_at`.
- **RP-TIM-022.** Las acciones de creación mantienen dos entradas visibles y no intercambiables: «Añadir experiencia» sigue RP-EXP-004 o RP-EXP-010 según el estado del período más reciente; «Cambiar situación» sigue RP-HIT-014. Ambas permanecen disponibles después de un cierre. Timeline representa el resultado confirmado con la misma identidad y agrupación estructural, con independencia del punto de entrada utilizado.
- **RP-TIM-023.** Cuando está disponible «Eliminar período», la confirmación informa que se eliminarán el período y todas las experiencias que contiene. La cantidad o fecha de experiencias independientes posteriores no oculta la acción; la existencia de otro hito sí la vuelve no disponible.

<a id="home"></a>
## Actividad reciente de Home

- **RP-HOM-001.** La actividad reciente registra el alta confirmada de un Crush, la creación confirmada de una experiencia y cada cambio relacional nuevo. Una corrección o eliminación actualiza la historia sin crear una actividad nueva.
- **RP-HOM-002.** El período inicial forma parte del alta del Crush y no genera una segunda actividad.
- **RP-HOM-003.** Una experiencia y un cambio relacional guardados juntos producen una única actividad que representa la acción compuesta.
- **RP-HOM-004.** La API conserva en la actividad únicamente las identidades, fechas, tipos y asociaciones estructurales necesarias. El nombre del Crush, el lugar y la descripción se resuelven desde el Vault desbloqueado.
- **RP-HOM-005.** Tocar una experiencia reciente abre el Crush al que pertenece mediante `crush_id`.
- **RP-HOM-006.** Los consejos diarios de la beta se presentan en el idioma elegido y son contenido informativo. No analizan el diario ni modifican sus datos.

Los requisitos para modificar cada elemento pertenecen a [Crush](CRUSH.md), [Experiencias](EXPERIENCES.md) y [Relaciones](RELATIONSHIPS.md). Los conflictos entre cambios pendientes se definen en [Guardado y sincronización](SYNC.md#conflictos).
