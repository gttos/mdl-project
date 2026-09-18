# Reglas de producto — Crush y perfil del Crush

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Alta, datos y edición del perfil del Crush, edad, cumpleaños, zodíaco, valoraciones, fotos, consulta, mapa, archivo y eliminación.

**Perfil del Crush** significa la ficha privada de una persona registrada en el diario. El [perfil de la cuenta de usuario](ACCOUNT.md#perfil-cuenta) corresponde a quien utiliza MDL.

<a id="inventario"></a>
## Inventario funcional del perfil del Crush

- **RP-CRU-013.** El perfil del Crush en la beta se compone del siguiente inventario cerrado. Cada ampliación del inventario requiere una nueva decisión de Producto.

Para este inventario, «legible por la API» significa que el servidor interpreta el valor y puede utilizarlo para las finalidades indicadas. «Vault opaco» significa que el servidor sólo conserva ciphertext y no puede interpretar el campo. «Administrado por el sistema» identifica estructura necesaria para ownership, sincronización o ciclo de vida que la persona no edita directamente. «Sólo dispositivo» y «transitorio» no se incorporan a la estructura durable legible de la API.

| Tratamiento | Dato exacto | Nombre canónico | Condición y uso autorizado |
|---|---|---|---|
| API legible | Identificador del Crush | `id` | Lo genera MDL, es inmutable y referencia la misma ficha en sus operaciones vinculadas. |
| API legible | Cuenta propietaria | `user_id` | Es inmutable y sólo autoriza a la cuenta propietaria a consultar o modificar el Crush. |
| API legible | País | `country_id` | Es obligatorio, pertenece al catálogo de países y se utiliza para perfil, mapa y estadísticas autorizadas. |
| API legible | Ducky | `ducky_id` | Es obligatorio y sólo admite los cuatro valores definidos por RP-CRU-005. |
| API legible | Edad confirmada | `age` | Es obligatoria, entera y de 18 años o más. Siempre se interpreta junto con `age_as_of`. |
| API legible | Fecha de confirmación de la edad | `age_as_of` | Es obligatoria, no puede ser futura y fija la fecha en que `age` fue confirmada. |
| API legible | Fecha de encuentro | `met_at` | Es obligatoria y rige el inicio cronológico del Crush, su período inicial y su experiencia inicial cuando exista. |
| API legible derivada | Signo zodiacal | `zodiac_id` | Es opcional y sólo se deriva del día y mes del cumpleaños guardados en el Vault. No permite enviar el cumpleaños a la API. |
| API legible | Altura | `height_id` | Es opcional y sólo admite una opción del catálogo de altura. |
| API legible | Color de ojos | `eye_color_id` | Es opcional y sólo admite una opción del catálogo de ojos. |
| API legible | Color de pelo | `hair_color_id` | Es opcional y sólo admite una opción del catálogo de pelo. |
| API legible | Tipo de cuerpo | `body_type_id` | Es opcional y sólo admite una opción del catálogo corporal. |
| API legible | Color de piel | `skin_color_id` | Es opcional y sólo admite una opción del catálogo de piel. |
| API legible | Belleza | `rate_beauty` | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible | Conexión emocional | `rate_emotional_connection` | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible | Personalidad | `rate_personality` | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible | Atractivo | `rate_sex_appeal` | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible derivada | Valoración global | `rate_global` | Es opcional, sólo la calcula MDL y puede contener hasta dos decimales conforme a BR-4.02. |
| Administrado por el sistema | Estado archivado | `is_archived` | Cambia únicamente mediante archivar o restaurar y no elimina la ficha ni su historia. |
| Administrado por el sistema | Estado seudonimizado tras baja de cuenta | `is_anonymized` | Impide el uso ordinario del Crush después de la baja. El nombre técnico histórico no permite calificar jurídicamente el resultado como anónimo. |
| Administrado por el sistema | Correlación de la acción compuesta | `correlation_id` | Identifica reintentos y las partes vinculadas del alta sin crear otro Crush ni otra experiencia inicial. No es editable ni visible como dato del perfil. |
| Administrado por el sistema | Fechas técnicas de creación, actualización, actividad, edición y eliminación | `created_at`, `updated_at`, `last_activity_at`, `last_user_edit_at`, `deleted_at` | Sólo sostienen orden, sincronización, archivo, borrado y conservación. No describen a la persona ni pueden sustituir fechas funcionales como `met_at` o `age_as_of`. |
| Vault opaco | Nombre | `name` dentro del payload Vault | Es obligatorio. Sólo se muestra o edita en el dispositivo con el Vault desbloqueado. |
| Vault opaco | Género | `gender_id` dentro del payload Vault | Es obligatorio, pertenece al catálogo de género y tiene su única representación funcional dentro del Vault. |
| Vault opaco | Profesión | `profession` dentro del payload Vault | Es opcional y no llega en claro a la API. |
| Vault opaco | Prefijo y número de WhatsApp | `whatsapp_prefix`, `whatsapp_number` dentro del payload Vault | Son dos campos opcionales que se conservan juntos dentro del Vault. |
| Vault opaco | Usuarios de redes | `instagram`, `facebook`, `snapchat`, `tiktok` dentro del payload Vault | Son cuatro campos opcionales independientes. |
| Vault opaco | Cumpleaños | `birthdate` dentro del payload Vault | Es opcional y se compone únicamente de día y mes. |
| Sólo dispositivo | Fotos del Crush | Archivos locales asociados a `id` | No se transmiten a la API ni al Vault y se rigen por RP-FOT-001..006. |
| Otros dominios vinculados | Situación relacional, períodos, hitos, experiencias y satisfacción | Entidades propias | Se rigen por sus documentos correspondientes y no se duplican como campos del perfil del Crush. |

- **RP-CRU-014.** País, Ducky, signo zodiacal, altura, color de ojos, color de pelo, tipo de cuerpo y color de piel se conservan como opciones de sus catálogos, mediante sus IDs. No son texto libre y no pertenecen al Vault por el solo hecho de describir a una persona.
- **RP-CRU-015.** Los campos estructurales del Crush son exactamente los enumerados como «API legible» o «Administrado por el sistema» en RP-CRU-013. Su contenido privado está formado por nombre, género, profesión, WhatsApp, Instagram, Facebook, Snapchat, TikTok y cumpleaños, con la obligatoriedad indicada en el mismo inventario. La situación relacional, las experiencias, la cuenta, el Vault y las fotos conservan sus contratos propios.

<a id="alta"></a>
## Alta de un Crush

- **BR-4.01.** Crear un Crush exige exactamente: nombre; género del catálogo; país del catálogo; Ducky; fecha de encuentro; edad confirmada; fecha de confirmación de esa edad; y situación relacional inicial. Nombre y género se guardan en el Vault; país, Ducky, fecha de encuentro, edad y fecha de confirmación forman estructura funcional.
- **RP-CRU-004.** El alta es una única acción compuesta que incluye el Crush, su período relacional inicial y, si se incluye, su experiencia inicial completa. Un formulario incompleto puede conservar temporalmente lo introducido para corregirlo, pero no crea un Crush ni un cambio pendiente. Cuando todos los datos son válidos, la aplicación puede conservar la acción completa como pendiente hasta sincronizarla; el servidor crea todas sus partes o ninguna.
- **RP-CRU-005.** La persona elige el Ducky del Crush entre `1 casualDuck`, `2 loveDuck`, `3 devilDuck` y `4 uglyDuck`. La aplicación no lo asigna ni lo cambia automáticamente a partir de la situación relacional, las estadísticas, la actividad o la IA.

La [situación relacional inicial](RELATIONSHIPS.md#inicial) y la [experiencia inicial opcional](EXPERIENCES.md#inicial) se definen en sus documentos respectivos.

<a id="perfil"></a>
## Perfil del Crush

### Campos exclusivos del Vault

- **RP-CRU-001.** El nombre y el `gender_id` del Crush son información sensible cuya única representación funcional se conserva dentro del payload Vault. El servidor almacena ese payload cifrado como parte del Vault.
- **RP-CRU-012.** Profesión; prefijo y número de WhatsApp; usuarios de Instagram, Facebook, Snapchat y TikTok; y día y mes de cumpleaños tienen su única representación funcional dentro del Vault. La búsqueda privada se ejecuta en el dispositivo mientras el Vault está desbloqueado y utiliza el contenido descifrado únicamente durante esa sesión.

### Rasgos físicos estructurados

- **RP-CRU-016.** Altura, color de ojos, color de pelo, tipo de cuerpo y color de piel son cinco campos opcionales independientes. Cada uno admite una opción de su catálogo y se conserva mediante su ID en la estructura funcional de la API. Ninguno contiene texto libre, se deriva de una foto ni se infiere mediante IA.
- **RP-CRU-017.** Añadir, cambiar o quitar un rasgo físico no modifica los demás rasgos, las valoraciones, el género, la edad, el país, el Ducky ni la situación relacional del Crush.

Los catálogos de rasgos físicos admiten exactamente estas opciones; el texto visible puede traducirse, pero conserva el mismo ID y significado:

| Campo | Opciones admitidas |
|---|---|
| Altura | `1 short`, `2 average`, `3 tall` |
| Color de ojos | `1 brown`, `2 green`, `3 blue`, `4 gray`, `5 hazel`, `6 amber` |
| Color de pelo | `1 black`, `2 brown`, `3 blond`, `4 red`, `5 gray`, `6 white`, `7 none` |
| Tipo de cuerpo | `1 slim`, `2 athletic`, `3 average`, `4 curvy`, `5 large` |
| Color de piel | `1 light`, `2 fair`, `3 medium`, `4 brown`, `5 black` |

### Fecha de encuentro

- **RP-CRU-007.** La fecha de encuentro puede corregirse desde el perfil del Crush. La fecha propuesta debe ser hoy o anterior y quedar estrictamente antes de la primera experiencia o del primer acontecimiento relacional independiente que permanezca en su fecha. El inicio relacional y la experiencia inicial se desplazan conjuntamente según RP-CRU-009.
- **RP-CRU-008.** El calendario permite elegir como máximo la fecha más temprana entre hoy y el día anterior a la primera experiencia o al primer acontecimiento relacional independiente del mismo Crush. Para calcular ese límite utiliza acontecimientos confirmados y cambios pendientes. Los acontecimientos del bundle inicial se desplazan con `met_at` y los de otros Crushes quedan fuera del cálculo.
- **RP-CRU-009.** Corregir la fecha de encuentro cambia conjuntamente la fecha del período relacional inicial, de su hito inicial y, cuando exista, de la experiencia inicial. Siguen siendo los mismos elementos y conservan sus asociaciones. Las experiencias y los acontecimientos relacionales independientes conservan su fecha y sus asociaciones actuales. Una posible asociación con el bundle inicial se evalúa desde el flujo propio del acontecimiento independiente y conforme a sus reglas de edición.

### Valoraciones

- **BR-4.02.** Belleza, conexión emocional, personalidad y atractivo son valoraciones opcionales que sólo admiten valores enteros: 1, 2, 3, 4 o 5. Se conservan como enteros y sólo pueden añadirse o modificarse desde el perfil del Crush, después del alta. La valoración global es la única que admite decimales: MDL la calcula automáticamente como la media aritmética de las valoraciones informadas, redondeada a dos decimales. La persona no puede introducir ni modificar el global. Si no hay ninguna valoración, no se muestra un valor global.

<a id="edad"></a>
### Edad, cumpleaños y zodíaco

#### Datos de referencia

- **RP-AGE-001.** La edad confirmada de un Crush debe ser de 18 años o más, tanto en el alta como en su edición.
- **RP-AGE-002.** La edad confirmada indica cuántos años tenía el Crush en la fecha de confirmación. Ambos datos se conservan juntos.
- **RP-AGE-003.** El cumpleaños del Crush guarda únicamente día y mes dentro del Vault; nunca su año de nacimiento.
- **RP-AGE-004.** El signo zodiacal se deriva del día y mes del cumpleaños. La API puede recibir el signo derivado, pero nunca el cumpleaños del Crush.
- **RP-AGE-005.** La fecha de encuentro no se utiliza para calcular ni confirmar la edad del Crush.

El signo derivado utiliza exactamente este catálogo: `1 aries`, `2 taurus`, `3 gemini`, `4 cancer`, `5 leo`, `6 virgo`, `7 libra`, `8 scorpio`, `9 sagittarius`, `10 capricorn`, `11 aquarius`, `12 pisces`.

#### Edad mostrada

- **RP-AGE-006.** Cuando se conoce el cumpleaños, MDL actualiza la edad mostrada a partir de la edad confirmada, su fecha de confirmación y los cumpleaños transcurridos desde entonces.
- **RP-AGE-007.** Sin cumpleaños, la edad es exacta en la fecha de confirmación y puede mostrarse después como un intervalo cuando ya no sea posible conocer el valor exacto.
- **RP-AGE-008.** La edad calculada nunca puede ser inferior a la última edad confirmada.
- **RP-AGE-009.** Para este cálculo, el aniversario de un cumpleaños del 29 de febrero se considera el 28 de febrero durante los años no bisiestos.

#### Corrección de edad o cumpleaños

- **RP-AGE-010.** Editar otros datos del perfil del Crush no modifica su edad, su fecha de confirmación ni su cumpleaños.
- **RP-AGE-011.** Cuando la persona corrige la edad, declara la edad actual del Crush. La fecha de confirmación pasa a ser el día de esa corrección.
- **RP-AGE-012.** Añadir o cambiar el cumpleaños conserva como referencia la edad vigente calculada con los datos anteriores y renueva su fecha de confirmación. La nueva fecha de cumpleaños se utiliza para las actualizaciones posteriores de edad.
- **RP-AGE-013.** Quitar el cumpleaños conserva la edad vigente, renueva su fecha de confirmación y elimina el día y mes del cumpleaños y su signo zodiacal.
- **RP-AGE-014.** Si se corrigen simultáneamente edad y cumpleaños, prevalece la edad introducida expresamente.
- **RP-AGE-015.** Reintentar el guardado de una corrección conserva la edad y la fecha de confirmación de ese mismo cambio, aunque el reintento ocurra otro día.

<a id="fotos"></a>
### Fotos del Crush

- **RP-FOT-001.** En la beta, las fotos del Crush son privadas y se guardan únicamente en el dispositivo. MDL no las sube al Vault ni ofrece una copia de seguridad remota de ellas.
- **RP-FOT-002.** Antes de guardar una foto, MDL elimina sus metadatos de cámara y ubicación. La foto queda asociada a la cuenta y al Crush correspondientes.
- **RP-FOT-003.** Las fotos y sus copias auxiliares se protegen mediante los controles del dispositivo y se excluyen de copias de seguridad no autorizadas.
- **RP-FOT-004.** Eliminar una foto retira de MDL la imagen guardada, sus miniaturas y sus demás copias auxiliares en el dispositivo.
- **RP-FOT-005.** Eliminar un Crush elimina también sus fotos guardadas por MDL. Al confirmar la baja de la cuenta, se eliminan las fotos de esa cuenta a las que la aplicación tiene acceso en el dispositivo.
- **RP-FOT-006.** Recuperar el Vault no recupera las fotos del Crush. MDL no garantiza el borrado remoto de fotos en un dispositivo sin conexión.

Las restricciones al editar el perfil del Crush mientras hay cambios pendientes se definen en [Guardado y sincronización](SYNC.md#conflictos). Los límites de longitud y formato todavía no decididos para nombre, profesión, WhatsApp y usuarios de redes se mantienen en [DEC-07](EXPERIENCES.md#dec-07); esa decisión no amplía el inventario de campos ni cambia los catálogos ya enumerados aquí.

<a id="mapa"></a>
## Mapa de Crushes por país

- **RP-MAP-001.** El mapa permite consultar los Crushes por país. No permite crearlos ni modificar sus datos.
- **RP-MAP-002.** Los Crushes se agrupan según el país registrado en su perfil. El punto del mapa representa al país y nunca la ubicación precisa de una persona.
- **RP-MAP-003.** Sólo puede estar abierta una ventana de información del mapa a la vez. Tocar el fondo la cierra.
- **RP-MAP-004.** Un país sin Crushes no muestra un marcador. Si no hay datos que representar, el mapa informa de ello en el idioma elegido en la aplicación.
- **RP-MAP-005.** Volver del mapa a la lista de Crushes conserva los filtros y los datos de la lista.

<a id="archivo-eliminacion"></a>
## Archivar y restaurar un Crush

- **RP-ARC-001.** Archivar un Crush lo retira de la lista de activos, conserva su historia y permite restaurarlo.
- **RP-ARC-002.** Un cambio pendiente en el Timeline del Crush no impide archivarlo o restaurarlo si su alta ya está confirmada. La lista de archivados muestra ese cambio pendiente y permite resolverlo.
- **RP-ARC-003.** Mientras el alta del Crush siga pendiente de confirmación, no puede archivarse ni se le pueden añadir experiencias o cambios relacionales.

## Eliminar un Crush

- **RP-DEL-001.** Eliminar un Crush exige confirmación y retira del diario el Crush, toda su historia —experiencias, períodos e hitos—, sus asociaciones, actividad, contenido del Vault y fotos vinculadas.
- **RP-DEL-002.** La eliminación individual de un Crush elimina esa ficha y todos sus datos vinculados. No aplica la conservación estructural prevista para la [baja completa de la cuenta](DATA_LIFECYCLE.md#baja).
- **RP-DEL-003.** La eliminación se completa entera o no se aplica. Si el sistema no puede identificar o retirar de forma segura todo lo vinculado al Crush, rechaza la operación y la aplicación conserva su copia hasta confirmar el resultado.
- **RP-DEL-004.** La confirmación de eliminación incluye descartar los cambios del Crush que todavía no modificaron el servidor. Si algún cambio fue enviado y su resultado es incierto, la aplicación comprueba primero su estado y continúa automáticamente con la eliminación cuando puede garantizar el resultado completo. Mientras no pueda confirmarlo, no elimina el Crush ni su copia local.
- **RP-DEL-005.** Tras confirmarse la eliminación, el Crush y sus datos vinculados dejan de estar disponibles. La destrucción definitiva del contenido cifrado sigue los plazos de [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).
