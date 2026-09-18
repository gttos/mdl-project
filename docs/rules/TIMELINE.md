# Reglas de producto — Timeline y Home

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Presentación de la historia, orden, agrupación, acciones disponibles y actividad reciente de Home.

<a id="contenido"></a>
## Contenido e identidad

- **RP-TIM-020.** Cada elemento del Timeline identifica exactamente un día civil, un Crush y uno de estos acontecimientos: bloque inicial; experiencia independiente; creación de un período posterior; o hito relacional de avance, cambio de etiqueta o cierre. Un cambio pendiente representa provisionalmente uno de esos mismos acontecimientos y conserva su identidad.
- **RP-TIM-021.** La estructura legible de un elemento contiene `id`, `type`, `date` y la referencia al Crush. Añade `experience`, `relationship` y `relationship_milestone` únicamente cuando ese acontecimiento los representa. `relationship_milestone` contiene la identidad, fecha, situación anterior, situación resultante, motivo de cierre, vínculo opcional con una experiencia y capacidades calculadas definidas en el [inventario relacional](RELATIONSHIPS.md#inventario).
- **RP-TIM-001.** El Timeline muestra la historia realmente registrada y nunca inventa acontecimientos para completar huecos.
- **RP-TIM-002.** El Timeline de un Crush sólo contiene acontecimientos de ese Crush. El Timeline global reúne los de la misma cuenta conservando su contexto.
- **RP-TIM-003.** Los elementos eliminados o retirados del uso ordinario y su contenido dependiente no aparecen en el Timeline.
- **RP-TIM-004.** El alta del Crush, su período relacional inicial y su experiencia inicial se muestran como un único bloque. Una experiencia integrada allí no aparece también como un acontecimiento separado.
- **RP-TIM-005.** La creación del período muestra la situación que se registró inicialmente y no cambia su representación cuando la relación avanza después.
- **RP-TIM-006.** Cada cambio real aparece una sola vez. La fase y el tipo de vínculo elegidos en una misma acción no generan dos acontecimientos.
- **RP-TIM-007.** El contexto relacional de una experiencia no vinculada se obtiene de la historia existente en su fecha, no del estado relacional actual.
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

- **RP-TIM-014.** Cada acontecimiento muestra únicamente las acciones permitidas por su dominio: una experiencia confirmada puede editarse o eliminarse conforme a RP-EXP-012 y RP-EXP-019; un hito relacional sólo ofrece editar o eliminar cuando la API lo identifica como el último elegible conforme a RP-HIT-002 y RP-HIT-003; el hito inicial nunca ofrece eliminarse.
- **RP-TIM-015.** El bloque inicial permite editar el Crush; con experiencia permite editarla o eliminarla y, sin ella, añadirla cuando ese día está libre. La situación relacional inicial sólo puede corregirse según [Corrección de hitos y períodos](RELATIONSHIPS.md#cambios).
- **RP-TIM-016.** Una experiencia no vinculada permite editarla o eliminarla. Iniciar un período relacional desde ella exige las condiciones de RP-EXP-009 y conserva su fecha.
- **RP-TIM-017.** Vincular una experiencia al crear un período exige que ese día no tenga otra experiencia vinculada y conserva la fecha; no vuelve editable un hito histórico.
- **RP-TIM-018.** Un cambio pendiente en el Timeline de un Crush bloquea nuevas acciones incompatibles de ese mismo Crush. No impide consultar o editar el perfil del Crush, gestionar sus fotos, navegar, archivarlo, eliminar una experiencia confirmada ni trabajar con otros Crushes.
- **RP-TIM-019.** Una edición pendiente del perfil del Crush sólo limita su Timeline cuando modifica realmente `met_at`.

<a id="home"></a>
## Actividad reciente de Home

- **RP-HOM-001.** La actividad reciente registra el alta confirmada de un Crush, la creación confirmada de una experiencia y cada cambio relacional nuevo. Una corrección o eliminación actualiza la historia sin crear una actividad nueva.
- **RP-HOM-002.** El período inicial forma parte del alta del Crush y no genera una segunda actividad.
- **RP-HOM-003.** Una experiencia y un cambio relacional guardados juntos producen una única actividad que representa la acción compuesta.
- **RP-HOM-004.** La API conserva en la actividad únicamente las identidades, fechas, tipos y asociaciones estructurales necesarias. El nombre del Crush, el lugar y la descripción se resuelven desde el Vault desbloqueado.
- **RP-HOM-005.** Tocar una experiencia reciente abre el Crush al que pertenece mediante `crush_id`.
- **RP-HOM-006.** Los consejos diarios de la beta se presentan en el idioma elegido y son contenido informativo. No analizan el diario ni modifican sus datos.

Los requisitos para modificar cada elemento pertenecen a [Crush](CRUSH.md), [Experiencias](EXPERIENCES.md) y [Relaciones](RELATIONSHIPS.md). Los conflictos entre cambios pendientes se definen en [Guardado y sincronización](SYNC.md#conflictos).
