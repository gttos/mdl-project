# Reglas de producto — Experiencias

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Experiencia inicial y posteriores: datos, fechas, creación, edición, eliminación y vinculación con relaciones.

<a id="inventario"></a>
## Inventario funcional de una experiencia

- **RP-EXP-025.** Una experiencia de la beta se compone del siguiente inventario cerrado. Cada ampliación requiere una nueva decisión de Producto.

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identificador de la experiencia | `id` | Lo genera MDL, es inmutable y conserva la identidad de la experiencia al editarla, moverla o vincularla. |
| API legible | Cuenta propietaria | `user_id` | Es inmutable y limita la experiencia a su cuenta propietaria. |
| API legible | Crush propietario | `crush_id` | Es obligatorio e inmutable. Una experiencia no cambia de Crush. |
| API legible | Día de la experiencia | `date` | Es obligatorio, se conserva como día civil y cumple las reglas cronológicas del Crush. |
| API legible | Satisfacción | `satisfaction_id` | Es obligatoria y admite exactamente los enteros 1, 2, 3, 4 o 5. |
| Administrado por el sistema | Correlación de una acción compuesta | `correlation_id` | Une la experiencia con el bundle inicial o con un cambio relacional registrado en la misma acción. No es editable ni visible como contenido de la experiencia. |
| Administrado por el sistema | Vinculación relacional | `relationship_id`, `period_change_id` | Identifica, cuando corresponde, el período y el hito asociados. La coincidencia de fechas por sí sola no sustituye estos vínculos. |
| Administrado por el sistema | Fechas técnicas de creación, actualización y eliminación | `created_at`, `updated_at`, `deleted_at` | Sostienen orden, sincronización y ciclo de vida. No sustituyen `date`. |
| Vault opaco | Lugar | `place` dentro del payload Vault | Es texto privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |
| Vault opaco | Descripción | `description` dentro del payload Vault | Es el único texto descriptivo privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |

- **RP-EXP-026.** La estructura funcional legible de una experiencia está formada por su identidad, propiedad, fecha, satisfacción y asociaciones. El contenido funcional de su payload Vault está formado por `place` y `description`. La API conserva el payload cifrado y no interpreta esos dos campos.

<a id="inicial"></a>
## Experiencia inicial opcional

- **RP-INI-001.** La experiencia inicial es opcional. Lugar, descripción y satisfacción forman un bloque completo: si los tres están vacíos, no se crea ninguna experiencia; si se informa cualquiera de ellos, deben completarse los tres. Mientras el bloque esté incompleto, el alta no puede confirmarse y la aplicación señala lo que falta.
- **RP-INI-002.** Lugar y descripción son información sensible y viven exclusivamente dentro del Vault. No forman parte del modelo funcional ni se envían en claro a la API. De esos tres campos de la experiencia inicial, la API funcional recibe únicamente el identificador de satisfacción (`satisfaction_id`), además de la fecha y las asociaciones estructurales necesarias.
- **RP-INI-003.** La experiencia inicial queda vinculada al Crush. Su fecha siempre coincide con `met_at` y no puede editarse de manera independiente. Si se corrige `met_at`, su fecha se actualiza conjuntamente.

<a id="contenido"></a>
## Contenido privado de la experiencia

- **RP-VLT-006.** Cada experiencia tiene una única descripción, identificada como `description`, y vive en el payload Vault de esa experiencia.

<a id="crear"></a>
## Crear una experiencia

- **RP-EXP-001.** Una experiencia pertenece a un Crush y puede existir como máximo una experiencia por Crush y día civil, incluidos los cambios pendientes conocidos.
- **RP-EXP-002.** Ampliar o corregir lo ocurrido ese mismo día modifica la experiencia existente; no crea una segunda.
- **RP-EXP-003.** `date` y `satisfaction_id` son obligatorios. `satisfaction_id` admite exactamente los valores 1 a 5. `place` y `description` son privados. La experiencia inicial sigue [Experiencia inicial](#inicial); la obligatoriedad exacta de `place` y `description` en una experiencia posterior está pendiente en [Decisiones pendientes](../PRODUCT.md#decisiones-pendientes).
- **RP-EXP-004.** Una experiencia puede registrarse sola o junto con un cambio relacional elegido expresamente. Cuando se registran juntos, ambos corresponden al día de la experiencia y se confirman o rechazan como una sola acción.
- **RP-EXP-005.** Una experiencia puede quedar vinculada como máximo a un período relacional y a un único hito de ese período.
- **RP-EXP-006.** Una experiencia posterior que no estuviera vinculada y cuya fecha pase a coincidir con `met_at` ocupa el lugar de la experiencia inicial si está libre, conservando su identidad.
- **RP-EXP-007.** Una experiencia puede vincularse al inicio de un período relacional posterior cuando ambas fechas coinciden y ese inicio todavía no tiene experiencia. Coincidir en la fecha no vincula automáticamente una experiencia a un hito posterior.
- **RP-EXP-008.** Si el hito ya tiene una experiencia vinculada, se rechaza un segundo vínculo sin modificar ninguna de las experiencias.
- **RP-EXP-009.** Crear un nuevo período relacional desde una experiencia requiere que no exista otro período vivo y que la fecha sea válida. No se confunde con cambiar la situación del período vigente.
- **RP-EXP-010.** Una experiencia posterior al cierre de una relación se guarda sin reabrirla ni crear automáticamente otro período. Cuando corresponda, la aplicación puede ofrecer iniciar uno nuevo.
- **RP-EXP-011.** La invitación a iniciar un nuevo período sólo aparece después de guardar correctamente la experiencia, una única vez y cuando esa acción sea posible.

<a id="editar"></a>
## Editar una experiencia

- **RP-EXP-012.** Una experiencia confirmada puede editarse sin límite de antigüedad, respetando su Crush, el orden cronológico y la regla de una experiencia por día.
- **RP-EXP-013.** La fecha de una experiencia vinculada a un hito relacional no puede editarse de manera independiente. Editar sus demás datos no mueve ni reescribe el hito.
- **RP-EXP-014.** La experiencia inicial permanece vinculada a `met_at` y se mueve conjuntamente cuando esa fecha se corrige válidamente.
- **RP-EXP-015.** Mover una experiencia no vinculada al día de `met_at`, cuando ese día está libre, la convierte en la experiencia inicial sin cambiar su identidad.
- **RP-EXP-016.** Mover una experiencia al día de inicio de otro período elegible aplica la misma regla de vínculo. Si ese lugar ya está ocupado, no se modifica la experiencia.
- **RP-EXP-017.** La beta no ofrece una acción independiente para desvincular una experiencia. Eliminarla conserva el hito relacional y permite crear después otra experiencia válida.
- **RP-EXP-018.** Un rechazo identifica el campo y la causa de manera comprensible y nunca anuncia un éxito inexistente.

<a id="eliminar"></a>
## Eliminar una experiencia

- **RP-EXP-019.** Toda experiencia puede eliminarse, sin límite de antigüedad y aunque no sea la más reciente.
- **RP-EXP-020.** Eliminar una experiencia vinculada retira la experiencia y su asociación, pero conserva el período, el hito y la historia relacional sin revertirlos.
- **RP-EXP-021.** Eliminar la experiencia inicial libera ese lugar y conserva el Crush y su período relacional inicial.
- **RP-EXP-022.** Una experiencia confirmada puede eliminarse aunque exista otro cambio pendiente en el Timeline del mismo Crush.
- **RP-EXP-023.** Una experiencia pendiente que nunca se envió o fue rechazada sin cambios puede descartarse. Si pudo haber sido aceptada, debe comprobarse primero su resultado.
- **RP-EXP-024.** El contenido privado de una experiencia eliminada sigue las reglas de retirada y destrucción de [Vault](VAULT.md) y [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).

Las opciones de cambio relacional se rigen por [Relaciones](RELATIONSHIPS.md#trayectos). Guardar una experiencia junto con un cambio no amplía los trayectos ni vuelve editable una relación histórica.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-07"></a>DEC-07 | Obligatoriedad de `place` y `description` en una experiencia posterior y límites de longitud o formato para esos campos, nombre, profesión, WhatsApp y usuarios de redes. | La experiencia inicial es todo-o-nada; `date` y `satisfaction_id` son obligatorios; los campos y catálogos admitidos ya están cerrados en sus inventarios. |
