# Reglas de producto — Experiencias

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Experiencia inicial y posteriores: datos, fechas, creación, edición, eliminación y vinculación con relaciones.

Una **experiencia** conserva qué ocurrió con un Crush en un día civil y pertenece siempre a un único período relacional. Es **independiente** cuando permanece dentro de ese período sin hito porque no registró una situación nueva; «independiente» nunca significa que exista fuera de un período. Cuando sí registra una situación nueva, queda vinculada al hito correspondiente. La experiencia no sustituye al hito: sus datos propios se rigen por este documento y la estructura relacional, por [Relaciones](RELATIONSHIPS.md).

<a id="inventario"></a>
## Inventario funcional de una experiencia

- **RP-EXP-025.** Una experiencia de la beta se compone del siguiente inventario cerrado. Cada ampliación requiere una nueva decisión de Producto.

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identificador de la experiencia | `id` | Lo genera el cliente MDL antes de persistir, es inmutable y se reutiliza en reintentos, ediciones, movimientos o vínculos. |
| API legible | Cuenta propietaria | `user_id` | Es inmutable y limita la experiencia a su cuenta propietaria. |
| API legible | Crush propietario | `crush_id` | Es obligatorio e inmutable. Una experiencia no cambia de Crush. |
| API legible | Día de la experiencia | `date` | Es obligatorio, se conserva como día civil y cumple las reglas cronológicas del Crush. |
| API legible | Satisfacción | `satisfaction_id` | Es obligatoria y admite exactamente los enteros 1, 2, 3, 4 o 5. |
| Administrado por el sistema | Correlación de una acción compuesta | `correlation_id` | Une la experiencia con el bloque inicial o con un cambio relacional registrado en la misma acción. No es editable ni visible como contenido de la experiencia. |
| Administrado por el sistema | Período al que pertenece | `relationship_period_id` | Es obligatorio e identifica el único período que contiene la experiencia. |
| Derivado por la API | Hito vinculado | `relationship_milestone_id` | Sólo tiene valor cuando la experiencia registra una situación nueva dentro de su período. Se obtiene desde el hito cuyo `experience_id` apunta a la experiencia; no exige otro vínculo físico y la coincidencia de fechas por sí sola nunca lo crea. |
| Administrado por el sistema | Fechas técnicas de creación, actualización y eliminación | `created_at`, `updated_at`, `deleted_at` | Sostienen orden, sincronización y ciclo de vida. No sustituyen `date`. |
| Vault opaco | Lugar | `place` dentro del payload Vault | Es texto libre obligatorio de hasta 255 caracteres, privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |
| Vault opaco | Descripción | `description` dentro del payload Vault | Es texto libre obligatorio de hasta 5.000 caracteres, constituye el único texto descriptivo privado de esa experiencia y sólo se muestra con el Vault desbloqueado. |

- **RP-EXP-026.** La estructura funcional legible de una experiencia está formada por su identidad, propiedad, fecha, satisfacción y asociaciones. El contenido privado de su payload Vault está formado exactamente por `place` y `description`; `satisfaction_id` pertenece exclusivamente a la estructura funcional y no se serializa ni se duplica dentro del payload cifrado. El dispositivo cifra ese payload antes de transferirlo, conserva localmente la entry cifrada y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. `place` y `description` sólo se descifran en la memoria de un dispositivo autorizado que haya desbloqueado o recuperado el mismo Vault; la API funcional interpreta únicamente la estructura de la experiencia.

<a id="inicial"></a>
## Experiencia inicial obligatoria

- **RP-INI-001.** Toda alta de un Crush exige una experiencia inicial completa que explique cómo se conocieron. Su fecha es `met_at`; lugar, descripción y satisfacción son obligatorios. Mientras cualquiera de esos datos falte o sea inválido, el alta no puede confirmarse y la aplicación señala todos los campos que deben corregirse.
- **RP-INI-002.** Lugar y descripción son información sensible y viven exclusivamente dentro del Vault. La aplicación los cifra en el dispositivo antes de transferir el payload, conserva la entry cifrada local y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. Sólo un dispositivo autorizado que desbloquee o recupere el mismo Vault los descifra para la persona propietaria. De los tres campos propios de la experiencia inicial, la API funcional interpreta únicamente el identificador de satisfacción (`satisfaction_id`), además de la fecha y las asociaciones estructurales necesarias.
- **RP-INI-003.** La experiencia inicial forma parte del bloque inicial y queda vinculada al Crush, al período inicial y a su hito inicial.

<a id="contenido"></a>
## Contenido privado de la experiencia

- **RP-VLT-006.** Cada experiencia tiene una única descripción, identificada como `description`, y vive en el payload Vault de esa experiencia.

<a id="crear"></a>
## Crear una experiencia

- **RP-EXP-001.** Una experiencia pertenece a un Crush y la combinación (`crush_id`, `date`) es única, incluidos los cambios pendientes conocidos. Por tanto, cada Crush puede tener como máximo una experiencia en un día civil, mientras que dos o más Crushes distintos pueden tener cada uno su propia experiencia en ese mismo día.
- **RP-EXP-002.** Ampliar o corregir lo ocurrido con el mismo Crush en ese mismo día modifica la experiencia existente; no crea una segunda para ese Crush.
- **RP-EXP-003.** Toda experiencia conserva obligatoriamente `date`, `satisfaction_id`, `place` y `description`. `satisfaction_id` admite exactamente los enteros 1, 2, 3, 4 o 5. `place` y `description` son textos libres privados que viven dentro del payload Vault; `place` admite hasta 255 caracteres y `description`, hasta 5.000. No pueden omitirse, quedar vacíos ni superar esos límites al crear o editar una experiencia.
- **RP-EXP-004.** Mientras exista un período vivo, «Añadir experiencia» comienza con un único paso para completar fecha, satisfacción, lugar y descripción. Al final ofrece la acción opcional «¿Hay un cambio de situación?». Si no se activa, guardar crea una experiencia independiente dentro del período. Si se activa, el formulario añade un segundo paso obligatorio para elegir la nueva `RelationshipSituation` y, cuando sea `CLOSED`, completar el motivo; guardar crea conjuntamente experiencia e hito con la misma fecha, correlación y vínculo explícito. Si cualquiera de las partes es inválida, no se crea ninguna. Si el período se cierra más adelante, las experiencias independientes que ya contenía continúan dentro de ese período histórico.
- **RP-EXP-005.** Toda experiencia pertenece exactamente a un período relacional y puede quedar vinculada como máximo a un único hito de ese período.
- **RP-EXP-006.** Para ese Crush, el día `met_at` está ocupado por la experiencia inicial obligatoria. Ninguna experiencia posterior del mismo Crush puede crearse en ese día; las experiencias de otros Crushes no ocupan ese espacio.
- **RP-EXP-010.** Cuando el período más reciente está cerrado, «Añadir experiencia» es obligatoriamente un formulario de dos pasos y no muestra la pregunta opcional de RP-EXP-004. El primer paso completa fecha, satisfacción, lugar y descripción; el segundo exige elegir una situación inicial 1–9. Para la persona se trata de registrar la experiencia y la nueva situación; guardar crea transparentemente la experiencia, el período nuevo y su hito inicial con la misma fecha, correlación y vínculo explícito. La experiencia no puede confirmarse por separado y, si cualquier parte es inválida, no se crea ninguna.

<a id="editar"></a>
## Editar una experiencia

- **RP-EXP-012.** Una experiencia confirmada puede editarse sin límite de antigüedad. Si es independiente, su fecha puede atravesar uno o varios hitos del mismo período, pero nunca salir de él: no puede quedar antes de `start_date` ni después de `end_date` cuando esté cerrado, y debe respetar hoy, `met_at` y la regla de una experiencia por Crush y día civil. Moverla no la reasigna a otro período.
- **RP-EXP-013.** La fecha de una experiencia vinculada a un hito relacional no puede editarse de manera independiente. Editar sus demás datos no mueve ni reescribe el hito.
- **RP-EXP-014.** La fecha de la experiencia inicial no se edita desde la propia experiencia: siempre coincide con `met_at` y se actualiza conjuntamente cuando esa fecha se corrige válidamente desde el Crush.
- **RP-EXP-015.** Una experiencia posterior no puede convertirse en experiencia inicial ni moverse a `met_at`, porque ese día conserva siempre la experiencia obligatoria del bloque inicial.
- **RP-EXP-016.** Una experiencia independiente puede moverse de un lado a otro de un hito de su mismo período y su contexto relacional se recalcula para la nueva fecha. No puede moverse al mismo día civil de un hito porque ese día ya contiene la experiencia vinculada a ese hito. El movimiento nunca crea un vínculo, otro hito ni un cambio de período.
- **RP-EXP-017.** La beta no ofrece una acción independiente para cambiar las asociaciones de una experiencia. No puede separarse de su período ni vincularse o desvincularse manualmente de un hito. Una experiencia vinculada sólo queda independiente cuando se conserva al eliminar su hito conforme a RP-HIT-009.
- **RP-EXP-018.** Un rechazo identifica el campo y la causa de manera comprensible y nunca anuncia un éxito inexistente.

<a id="eliminar"></a>
## Eliminar una experiencia

- **RP-EXP-019.** Una experiencia independiente puede eliminarse sin límite de antigüedad y sin eliminar su período. Una experiencia vinculada no utiliza esta acción independiente: su borrado se rige por RP-EXP-020.
- **RP-EXP-020.** Iniciar el borrado desde una experiencia vinculada solicita siempre eliminar conjuntamente la experiencia y su hito. La elegibilidad latest-only y el alcance atómico de la operación —«Eliminar cambio y experiencia» o «Eliminar período»— se rigen exclusivamente por RP-HIT-009 y RP-HIT-010. Si esas condiciones no se cumplen, no se elimina ninguna parte.
- **RP-EXP-021.** La experiencia inicial no puede eliminarse de manera independiente ni ofrece esa acción. Puede editarse; sólo el borrado completo del Crush retira el bloque inicial y toda su historia.
- **RP-EXP-022.** Una experiencia independiente confirmada puede eliminarse aunque exista otro cambio pendiente que no la afecte. Una experiencia vinculada a un hito no puede eliminarse mientras una operación pendiente pueda alterar cuál es el último elemento relacional; primero se resuelve esa operación y después se revalida la elegibilidad latest-only.
- **RP-EXP-023.** Una experiencia pendiente que nunca se envió o fue rechazada sin cambios puede descartarse. Si pudo haber sido aceptada, debe comprobarse primero su resultado.
- **RP-EXP-024.** El contenido privado de una experiencia eliminada sigue las reglas de retirada y destrucción de [Vault](VAULT.md) y [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).

Las opciones de cambio relacional se rigen por [Relaciones](RELATIONSHIPS.md#trayectos). Guardar una experiencia junto con un cambio no amplía los trayectos ni vuelve editable un hito o período histórico.
