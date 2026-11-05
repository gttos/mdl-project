# Reglas de producto — Guardado y sincronización

<a id="copy-es"></a>

[Índice de reglas](../PRODUCT.md) · Qué significa guardar, cambios pendientes, conflictos, reintentos, corrección y descarte.

<a id="identidad"></a>
## Identidad de una acción y confirmación completa

- **RP-ID-001.** Reintentar una misma acción conserva las identidades originales y no crea duplicados.
- **RP-ID-002.** Repetir exactamente una creación ya aceptada produce el mismo resultado. Si la identidad pertenece a otra cuenta o representa datos incompatibles, se rechaza sin sobrescribir nada.
- **RP-ID-008.** Una acción compuesta se confirma completa o se rechaza completa. Dos acciones simultáneas no pueden ocupar ambas un lugar que admite un único resultado ni dejar cambios parciales.

<a id="guardado"></a>
## Guardado

- **RP-DUR-010.** Cada cambio pendiente conserva su identidad, propietario, tipo, Crush y área afectados, datos completos cifrados, estado de confirmación, intentos, próxima oportunidad de reintento y causa técnica saneada. La información permite reanudar la misma intención tras cerrar la aplicación.
- **RP-DUR-011.** El snapshot cifrado contiene todos los valores e identidades necesarios para repetir exactamente la acción elegida. La causa técnica conserva únicamente un código estructurado y nunca incorpora contenido privado, credenciales ni texto introducido por la persona.
- **RP-DUR-001.** Antes de comunicar que un cambio quedó guardado, la aplicación valida la acción completa y conserva localmente todo lo necesario para reanudarla sin alterar identidades, fechas ni contenido privado.
- **RP-DUR-002.** Después de conservar el cambio de forma durable, la aplicación confirma el guardado local, termina la entrada y permite consultar su estado pendiente.
- **RP-DUR-003.** «Sincronizado» significa que tanto la parte funcional como el contenido cifrado del Vault fueron confirmados y ya no queda un cambio pendiente.
- **RP-DUR-004.** Cerrar y volver a abrir la aplicación conserva el mismo cambio pendiente, sus datos y su estado; no lo duplica ni recalcula valores dependientes del día.
- **RP-DUR-005.** Actualizar los datos confirmados no hace desaparecer temporalmente un cambio pendiente ni lo muestra dos veces.

<a id="conflictos"></a>
## Cambios pendientes y conflictos

- **RP-DUR-006.** Puede existir como máximo un cambio estructural pendiente por Crush en cada área: alta del Crush, perfil del Crush o Timeline del Crush. La intención durable de eliminar una experiencia independiente confirmada no ocupa el cupo de Timeline cuando el cambio pendiente no la afecta y puede coexistir conforme a RP-EXP-022 y RP-TIM-018. El borrado de una experiencia vinculada sigue esperando cualquier operación que pueda alterar su elegibilidad latest-only.
- **RP-DUR-007.** El alta de un Crush queda confirmada sólo cuando su operación durable completó tanto la parte funcional como el contenido cifrado del Vault y ya no existe un cambio pendiente, conforme a RP-DUR-003. La aceptación de la API por sí sola no la confirma. Mientras exista la operación pendiente del área de alta, aunque la API ya devuelva el Crush, no pueden abrirse su perfil completo, crear experiencias, registrar relaciones ni archivarlo. Su fila completa abre el estado de esa alta pendiente; la insignia no es la única zona que permite consultarlo.
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
- **RP-RES-002.** Corregir vuelve a abrir los datos protegidos del cambio y sustituye el pendiente completo conservando sus identidades. Si falta contexto seguro, no inventa un formulario. En las ediciones del perfil del Crush, la selección de apartados y la guía de corrección siguen RP-RES-008..010.
- **RP-RES-003.** Eliminar un cambio exige confirmación y descarta la acción completa, nunca sólo una de sus partes.
- **RP-RES-004.** Sólo puede descartarse directamente un cambio que nunca se envió o que fue rechazado sin producir modificaciones.
- **RP-RES-005.** Cuando se pierde la respuesta, el resultado es incierto. Antes de ofrecer una eliminación funcional debe comprobarse si el cambio fue aceptado.
- **RP-RES-006.** Descartar un alta pendiente la retira; descartar una edición restaura el valor confirmado. No intenta eliminar en el servidor algo que nunca llegó a crearse.
- **RP-RES-007.** Sólo se muestran acciones que realmente pueden ejecutarse: corregir, reintentar cuando corresponda, descartar con seguridad, recuperar la sesión o cerrar.
- **RP-RES-008.** El estado de una edición pendiente del perfil identifica el Crush, el tipo de acción, el momento del guardado y los apartados modificados que pueda determinar con certeza. Mantiene la identidad del cambio seleccionado aunque existan pendientes de otros Crushes o de otras áreas. Los datos privados sólo se presentan dentro de la sesión Vault desbloqueada correspondiente.
- **RP-RES-009.** Corregir una edición pendiente del perfil del Crush muestra únicamente los apartados modificados de esa edición, precargados con sus valores pendientes. Si cambió uno, abre ese apartado; si cambiaron varios, permite recorrerlos en su orden habitual, con numeración relativa al recorrido de corrección. Visitar un apartado no equivale a modificarlo. La selección se fija al confirmar la edición y se conserva cifrada con la intención, también tras relanzar o reintentar; no se reconstruye comparando con un perfil actualizado después. El resto del borrador completo se conserva al corregir, cancelar o volver entre apartados. Los ajustes derivados de un dato no añaden por sí solos otro apartado editable. Si no existe información fiable para determinar la selección, se explica la limitación y no se inventan apartados ni se ofrece una corrección inejecutable; la operación permanece intacta y conserva sólo las acciones seguras de RP-RES-003..007.
- **RP-RES-010.** Una edición del perfil que necesita revisión explica la causa y la acción necesaria cuando están determinadas mediante un código estructurado conocido o una validación local fiable. La guía aparece en el estado y en la corrección; cuando la causa corresponde a un campo de los apartados modificados, abre su apartado y señala el campo sin ocultar los otros apartados modificados. Una causa desconocida se reconoce como tal y permite revisar los datos pendientes disponibles sin atribuir el fallo a un campo supuesto. Una causa que no puede resolverse dentro de esos apartados explica esa limitación y ofrece las salidas seguras disponibles, sin ampliar silenciosamente la edición. No se afirma que la API rechazó el cambio si el fallo ocurrió antes del envío ni se muestran mensajes libres del servidor, excepciones o códigos técnicos.

Un alta completa y válida puede conservarse como cambio pendiente hasta sincronizarse; el formulario incompleto y la atomicidad del bloque se rigen por [RP-CRU-004](CRUSH.md#alta).

<a id="escenarios-correccion-perfil"></a>
## Escenarios de corrección del perfil del Crush

Estos escenarios concretan RP-RES-001..010 sin redefinir sus reglas. La evidencia técnica y sus límites pertenecen a [Testing de iOS](../../../ios/docs/TESTING.md#corrección-de-ediciones-pendientes-del-perfil).

| ID | Dado / Cuando / Entonces | Referencia |
|---|---|---|
| GWT-PCO-01 | Dados pendientes de varios Crushes, al abrir uno se identifican su Crush, tipo, momento y apartados conocidos; corregir carga exactamente esa intención y nunca otra que aparezca después. | RP-RES-001/002/008 |
| GWT-PCO-02 | Dado que sólo cambió el cumpleaños, al corregir se abre su apartado con el valor pendiente; los ajustes derivados de edad y signo no añaden datos básicos al recorrido. | RP-RES-009, RP-AGE-012/015 |
| GWT-PCO-03 | Dados cambios en varios apartados, al corregir se recorren sólo esos apartados en su orden habitual, con contador relativo y Guardar al final; avanzar, volver o cancelar no pierde valores. | RP-RES-009, RP-FOR-005 |
| GWT-PCO-04 | Dado un apartado sólo visitado o cuyo borrador volvió al original antes de confirmar, al guardar no se clasifica como modificado por esa interacción; una confirmación expresa de edad conserva su semántica propia. | RP-RES-009, RP-AGE-011/014/015 |
| GWT-PCO-05 | Dada una causa conocida de un campo de los apartados modificados, la hoja explica qué corregir y el formulario abre su apartado y señala el campo concreto; los otros apartados modificados permanecen accesibles. | RP-RES-010, RP-UX-001/002 |
| GWT-PCO-06 | Dada una causa desconocida o sin campo fiable, se explica la incertidumbre y se revisan sólo los apartados modificados. Si una causa conocida exige otro apartado, se identifican el campo y el apartado y se explica la limitación, conservando las salidas seguras sin ampliar la selección. Un fallo previo al envío no se presenta como rechazo de API. | RP-RES-007/009/010 |
| GWT-PCO-07 | Dado un relanzamiento o reintento, se mantienen selección, valores e identidades de la intención aunque el perfil confirmado cambie después. | RP-DUR-004/011, RP-RES-002/009 |
| GWT-PCO-08 | Dada una corrección válida o inválida, al guardar se valida el formulario completo y sólo la válida sustituye atómicamente el pendiente completo, preservando los campos no visibles y sin duplicados; guardado local y sincronización son confirmaciones distintas. | RP-DUR-001..003, RP-RES-002, RP-FOR-009 |
| GWT-PCO-09 | Dada una selección ausente, inválida o no atribuible, no se inventan pasos ni se sustituye o envía desde una corrección insegura; la intención se conserva y sólo se descarta cuando su certeza lo permite. | RP-DUR-009, RP-RES-002..007/009 |
| GWT-PCO-10 | Dado bloqueo, cambio de cuenta, operación completada o sustituida durante la apertura, no se muestran datos viejos ni se actúa sobre otra operación. La autodetección resuelve antes de presentar el formulario; su fallo ofrece una salida comprensible y la corrección no crea un nuevo embudo de edición. | RP-RES-001/008/009, RP-ID-007, RP-ANA-014 |
