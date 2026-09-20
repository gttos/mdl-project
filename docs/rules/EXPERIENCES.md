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
| Vault opaco | Lugar | `place` dentro del payload Vault | Es texto libre obligatorio de hasta 255 caracteres, privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |
| Vault opaco | Descripción | `description` dentro del payload Vault | Es texto libre obligatorio de hasta 5.000 caracteres, constituye el único texto descriptivo privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |

- **RP-EXP-026.** La estructura funcional legible de una experiencia está formada por su identidad, propiedad, fecha, satisfacción y asociaciones. El contenido privado de su payload Vault está formado por `place` y `description`. El dispositivo cifra ese payload antes de transferirlo, conserva localmente la entry cifrada y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. `place` y `description` sólo se descifran en la memoria de un dispositivo autorizado que haya desbloqueado o recuperado el mismo Vault; la API funcional interpreta únicamente la estructura de la experiencia.

<a id="inicial"></a>
## Experiencia inicial obligatoria

- **RP-INI-001.** Toda alta de un Crush exige una experiencia inicial completa que explique cómo se conocieron. Su fecha es `met_at`; lugar, descripción y satisfacción son obligatorios. Mientras cualquiera de esos datos falte o sea inválido, el alta no puede confirmarse y la aplicación señala todos los campos que deben corregirse.
- **RP-INI-002.** Lugar y descripción son información sensible y viven exclusivamente dentro del Vault. La aplicación los cifra en el dispositivo antes de transferir el payload, conserva la entry cifrada local y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. Sólo un dispositivo autorizado que desbloquee o recupere el mismo Vault los descifra para la persona propietaria. De los tres campos propios de la experiencia inicial, la API funcional interpreta únicamente el identificador de satisfacción (`satisfaction_id`), además de la fecha y las asociaciones estructurales necesarias.
- **RP-INI-003.** La experiencia inicial queda vinculada al Crush. Su fecha siempre coincide con `met_at` y no puede editarse de manera independiente. Si se corrige `met_at`, su fecha se actualiza conjuntamente.

<a id="contenido"></a>
## Contenido privado de la experiencia

- **RP-VLT-006.** Cada experiencia tiene una única descripción, identificada como `description`, y vive en el payload Vault de esa experiencia.

<a id="crear"></a>
## Crear una experiencia

- **RP-EXP-001.** Una experiencia pertenece a un Crush y puede existir como máximo una experiencia por Crush y día civil, incluidos los cambios pendientes conocidos.
- **RP-EXP-002.** Ampliar o corregir lo ocurrido ese mismo día modifica la experiencia existente; no crea una segunda.
- **RP-EXP-003.** Toda experiencia conserva obligatoriamente `date`, `satisfaction_id`, `place` y `description`. `satisfaction_id` admite exactamente los enteros 1, 2, 3, 4 o 5. `place` y `description` son textos libres privados que viven dentro del payload Vault; `place` admite hasta 255 caracteres y `description`, hasta 5.000. No pueden omitirse, quedar vacíos ni superar esos límites al crear o editar una experiencia. La experiencia inicial cumple la misma obligatoriedad y usa `met_at` como fecha.
- **RP-EXP-004.** Una experiencia puede registrarse sin cambio relacional. Toda creación de período o cambio relacional, en cambio, exige una experiencia elegida expresamente y se guarda junto con ella como una sola acción atómica en el día de la experiencia.
- **RP-EXP-005.** Una experiencia puede quedar vinculada como máximo a un período relacional y a un único hito de ese período.
- **RP-EXP-006.** El día `met_at` está ocupado por la experiencia inicial obligatoria. Ninguna experiencia posterior puede crearse o moverse a ese día.
- **RP-EXP-008.** Si el hito ya tiene una experiencia vinculada, se rechaza un segundo vínculo sin modificar ninguna de las experiencias.
- **RP-EXP-010.** Una experiencia posterior al cierre de una relación se guarda sin reabrirla ni crear automáticamente otro período.

<a id="editar"></a>
## Editar una experiencia

- **RP-EXP-012.** Una experiencia confirmada puede editarse sin límite de antigüedad, respetando su Crush, el orden cronológico y la regla de una experiencia por día.
- **RP-EXP-013.** La fecha de una experiencia vinculada a un hito relacional no puede editarse de manera independiente. Editar sus demás datos no mueve ni reescribe el hito.
- **RP-EXP-014.** La experiencia inicial permanece vinculada a `met_at` y se mueve conjuntamente cuando esa fecha se corrige válidamente.
- **RP-EXP-015.** Una experiencia posterior no puede convertirse en experiencia inicial ni moverse a `met_at`, porque ese día conserva siempre la experiencia obligatoria del bundle inicial.
- **RP-EXP-016.** Mover una experiencia no vinculada al día de un período o hito no crea una asociación automática. Todo vínculo relacional nace de una acción explícita y atómica conforme a RP-EXP-004.
- **RP-EXP-017.** La beta no ofrece una acción independiente para desvincular una experiencia. Cuando una experiencia está vinculada al último hito eliminable, las acciones de ese bundle permiten elegir entre eliminar sólo el hito —la experiencia conserva su identidad y fecha como independiente— o eliminar conjuntamente el hito y la experiencia. Iniciar el borrado desde la experiencia vinculada siempre aplica la segunda opción. Ninguna opción deja un hito confirmado sin experiencia.
- **RP-EXP-018.** Un rechazo identifica el campo y la causa de manera comprensible y nunca anuncia un éxito inexistente.

<a id="eliminar"></a>
## Eliminar una experiencia

- **RP-EXP-019.** Una experiencia no vinculada puede eliminarse sin límite de antigüedad. Una experiencia vinculada sólo puede eliminarse cuando el período o hito asociado sea también el último elemento relacional eliminable; en cualquier otro caso se rechaza sin mutación.
- **RP-EXP-020.** Eliminar una experiencia vinculada —desde su propia acción o mediante «Eliminar cambio y experiencia» en el último hito elegible— elimina atómicamente también su período o hito asociado y restaura la historia conforme a las reglas latest-only de Relaciones. Si cualquiera de las dos partes no puede eliminarse, no se elimina ninguna.
- **RP-EXP-021.** La experiencia inicial no puede eliminarse de manera independiente ni ofrece esa acción. Puede editarse; sólo el borrado completo del Crush retira el bundle inicial y toda su historia.
- **RP-EXP-022.** Una experiencia independiente confirmada puede eliminarse aunque exista otro cambio pendiente que no la afecte. Una experiencia vinculada no puede eliminarse mientras una operación pendiente pueda alterar cuál es el último elemento relacional; primero se resuelve esa operación y después se revalida la elegibilidad latest-only.
- **RP-EXP-023.** Una experiencia pendiente que nunca se envió o fue rechazada sin cambios puede descartarse. Si pudo haber sido aceptada, debe comprobarse primero su resultado.
- **RP-EXP-024.** El contenido privado de una experiencia eliminada sigue las reglas de retirada y destrucción de [Vault](VAULT.md) y [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).

Las opciones de cambio relacional se rigen por [Relaciones](RELATIONSHIPS.md#trayectos). Guardar una experiencia junto con un cambio no amplía los trayectos ni vuelve editable una relación histórica.
