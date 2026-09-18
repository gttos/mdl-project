# Reglas de producto — Guardado y sincronización

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Qué significa guardar, cambios pendientes, conflictos, reintentos, corrección y descarte.

<a id="identidad"></a>
## Identidad de una acción y confirmación completa

- **RP-ID-001.** Reintentar una misma acción conserva las identidades originales y no crea duplicados.
- **RP-ID-002.** Repetir exactamente una creación ya aceptada produce el mismo resultado. Si la identidad pertenece a otra cuenta o representa datos incompatibles, se rechaza sin sobrescribir nada.
- **RP-ID-008.** Una acción compuesta se confirma completa o se rechaza completa. Dos acciones simultáneas no pueden ocupar ambas un lugar que admite un único resultado ni dejar cambios parciales.

<a id="guardado"></a>
## Guardado

- **RP-DUR-010.** Cada cambio pendiente se conserva en el dispositivo con este inventario mínimo: identidad de la operación (`id`); Vault propietario (`owner_vault_id`); tipo de acción (`operation_type`); Crush afectado cuando corresponde (`scope_crush_id`); área de conflicto (`domain`); snapshot completo cifrado (`encrypted_snapshot`, `nonce`, `auth_tag`); estado de cada parte (`entries`); estado general (`disposition`); intentos consumidos y próxima oportunidad (`attempt_count`, `next_retry_at`); causa técnica saneada (`disposition_reason`); y fechas de creación y actualización (`created_at`, `updated_at`).
- **RP-DUR-011.** El snapshot cifrado contiene todos los valores e identidades necesarios para repetir exactamente la acción elegida. La causa técnica conserva únicamente un código estructurado y nunca incorpora contenido privado, credenciales ni texto introducido por la persona.
- **RP-DUR-001.** Antes de comunicar que un cambio quedó guardado, la aplicación valida la acción completa y conserva localmente todo lo necesario para reanudarla sin alterar identidades, fechas ni contenido privado.
- **RP-DUR-002.** Sólo después muestra «Guardado en este dispositivo», cierra el formulario y refleja el cambio pendiente.
- **RP-DUR-003.** «Sincronizado» significa que tanto la parte funcional como el contenido cifrado del Vault fueron confirmados y ya no queda un cambio pendiente.
- **RP-DUR-004.** Cerrar y volver a abrir la aplicación conserva el mismo cambio pendiente, sus datos y su estado; no lo duplica ni recalcula valores dependientes del día.
- **RP-DUR-005.** Actualizar los datos confirmados no hace desaparecer temporalmente un cambio pendiente ni lo muestra dos veces.

<a id="conflictos"></a>
## Cambios pendientes y conflictos

- **RP-DUR-006.** Puede existir como máximo un cambio pendiente por Crush en cada área: alta del Crush, perfil del Crush o Timeline del Crush.
- **RP-DUR-007.** Mientras el alta de un Crush no esté confirmada, no pueden abrirse su perfil completo, crear experiencias, registrar relaciones ni archivarlo. Su fila abre el estado de esa alta pendiente.
- **RP-DUR-008.** El perfil del Crush y su Timeline pueden tener cambios pendientes separados, salvo el conflicto de fecha definido en RP-CRU-010. Un pendiente nunca bloquea ni aparece en otro Crush.
- **RP-DUR-009.** Un cambio pendiente que no conserve la información necesaria para reanudarse no se ejecuta ni se completa mediante suposiciones.
- **RP-CRU-006.** Un cambio pendiente en el Timeline de un Crush no impide editar el perfil de ese Crush. Sólo puede existir una edición pendiente del perfil del mismo Crush.
- **RP-CRU-010.** Una edición pendiente del perfil del Crush que no cambie `met_at` no limita su Timeline. Si cambia `met_at`, no puede crearse una experiencia nueva para ese Crush hasta confirmar, corregir o descartar la edición.
- **RP-CRU-011.** Mientras una corrección de `met_at` está pendiente, las fechas válidas para una experiencia nueva no se recalculan usando esa fecha todavía no confirmada.

<a id="reintentos"></a>
## Reintentos

- **RP-RET-001.** Si la aplicación sabe que no hay conexión antes de enviar, espera sin consumir intentos ni repetir tráfico.
- **RP-RET-002.** Ante una interrupción temporal, un fallo transitorio del servidor o una limitación de uso, cada ciclo admite el intento inicial y hasta dos reintentos automáticos, aproximadamente a los 5 y 30 segundos mientras la aplicación está activa.
- **RP-RET-003.** Cuando el servidor indica cuánto esperar, se respeta ese plazo. Agotar un ciclo no se olvida por llevar la aplicación a segundo plano o volver a abrirla.
- **RP-RET-004.** Un rechazo definitivo pasa a revisión sin repetirse indefinidamente. Una sesión inválida espera una nueva autenticación.
- **RP-RET-005.** Reintentar manualmente inicia un ciclo nuevo sobre el mismo cambio, sin modificar sus identidades ni sus datos.
- **RP-RET-006.** Durante la beta, los reintentos se reanudan cuando la aplicación vuelve a estar activa. Cada Vault mantiene un solo dispositivo activo conforme a RP-VLT-015.

<a id="resolucion"></a>
## Estados y resolución

| Estado | Significado |
|---|---|
| Guardado en dispositivo | El cambio quedó protegido en el dispositivo y puede reanudarse. |
| Sincronizando | El cambio está siendo enviado o verificado. |
| Pendiente de conexión | Espera conexión; si nunca se envió, puede descartarse. |
| No se pudo sincronizar | Se agotaron los intentos automáticos y puede reintentarse manualmente. |
| Necesita revisión | El cambio requiere corrección o una decisión de la persona. |
| Esperando sesión | Falta autenticación válida. |
| Sincronizado | Todas las partes del cambio quedaron confirmadas. |

- **RP-RES-001.** Al abrir el estado de un cambio se muestra siempre ese cambio. Si termina en ese momento, se presenta el resultado actualizado o no se abre la vista; nunca aparece una pantalla vacía.
- **RP-RES-002.** Corregir vuelve a abrir los datos protegidos del cambio y sustituye el pendiente completo conservando sus identidades. Si falta contexto seguro, no inventa un formulario.
- **RP-RES-003.** Eliminar un cambio exige confirmación y descarta la acción completa, nunca sólo una de sus partes.
- **RP-RES-004.** Sólo puede descartarse directamente un cambio que nunca se envió o que fue rechazado sin producir modificaciones.
- **RP-RES-005.** Cuando se pierde la respuesta, el resultado es incierto. Antes de ofrecer una eliminación funcional debe comprobarse si el cambio fue aceptado.
- **RP-RES-006.** Descartar un alta pendiente la retira; descartar una edición restaura el valor confirmado. No intenta eliminar en el servidor algo que nunca llegó a crearse.
- **RP-RES-007.** Sólo se muestran acciones que realmente pueden ejecutarse: corregir, reintentar cuando corresponda, descartar con seguridad, recuperar la sesión o cerrar.

Un alta completa y válida puede conservarse como cambio pendiente hasta sincronizarse; el formulario incompleto y la atomicidad del bloque se rigen por [RP-CRU-004](CRUSH.md#alta).
