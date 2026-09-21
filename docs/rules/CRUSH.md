# Reglas de producto — Crush y perfil del Crush

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Alta, datos y edición del perfil del Crush, resultado de match, edad, cumpleaños, zodíaco, valoraciones, fotos, consulta, mapa, archivo y eliminación.

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
| API legible | Edad confirmada | `age` | Es obligatoria y entera entre 18 y 80 años inclusive. Siempre se interpreta junto con `age_as_of`. |
| API legible administrada por el servidor | Fecha de confirmación de la edad | `age_as_of` | Es obligatoria, no puede ser futura y fija la fecha en que `age` fue confirmada. La API la genera cuando acepta una edad nueva; ningún cliente puede enviarla ni modificarla directamente. |
| API legible | Fecha de encuentro | `met_at` | Es obligatoria y rige el inicio cronológico del Crush, su período inicial y su experiencia inicial obligatoria. |
| API legible derivada | Resultado de match | `match_status_id` | Es obligatorio y admite `0 UNKNOWN`, `1 MATCH` o `2 NO_MATCH`. Se recalcula conforme a [Resultado de match](#match). |
| API legible derivada | Primer match | `first_match_situation_id`, `first_match_date` | Son nulos mientras no exista match y señalan la primera situación 5–9 y su fecha cuando lo hay. |
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
| Vault opaco | Nombre | `name` dentro del payload Vault | Es obligatorio, contiene texto y admite hasta 191 caracteres. Sólo se muestra o edita en el dispositivo con el Vault desbloqueado. |
| Vault opaco | Género | `gender_id` dentro del payload Vault | Es obligatorio, pertenece al catálogo de género y tiene su única representación funcional dentro del Vault. |
| Vault opaco | Profesión | `profession` dentro del payload Vault | Es texto opcional de hasta 191 caracteres y no llega en claro a la API. |
| Vault opaco | Prefijo y número de WhatsApp | `whatsapp_prefix`, `whatsapp_number` dentro del payload Vault | Forman un único contacto opcional dentro del Vault: el prefijo se elige del catálogo de países y el número admite únicamente dígitos, con un máximo de 20. |
| Vault opaco | Usuarios de redes | `instagram`, `facebook`, `snapchat`, `tiktok` dentro del payload Vault | Son cuatro textos opcionales independientes de hasta 255 caracteres cada uno. |
| Vault opaco | Cumpleaños | `birthdate` dentro del payload Vault | Es opcional y se compone únicamente de día y mes. |
| Sólo dispositivo | Fotos del Crush | Archivos locales asociados a `id` | No se transmiten a la API ni al Vault y se rigen por RP-FOT-001..009. |
| Otros dominios vinculados | Situación relacional, períodos, hitos, experiencias y satisfacción | Entidades propias | Se rigen por sus documentos correspondientes. El Crush sólo conserva las proyecciones de match declaradas arriba para consulta directa; no duplica el historial. |

- **RP-CRU-014.** País, Ducky, signo zodiacal, altura, color de ojos, color de pelo, tipo de cuerpo y color de piel se conservan como opciones de sus catálogos, mediante sus IDs. No son texto libre y no pertenecen al Vault por el solo hecho de describir a una persona.
- **RP-CRU-015.** Los campos estructurales del Crush son exactamente los enumerados como «API legible» o «Administrado por el sistema» en RP-CRU-013. Su contenido privado está formado por nombre, género, profesión, WhatsApp, Instagram, Facebook, Snapchat, TikTok y cumpleaños, con la obligatoriedad indicada en el mismo inventario. El dispositivo cifra ese contenido antes de transferirlo, conserva localmente la entry cifrada y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. El contenido sólo se descifra en la memoria de un dispositivo autorizado que haya desbloqueado o recuperado el mismo Vault; la API funcional continúa interpretando únicamente la estructura enumerada. La situación relacional, las experiencias, la cuenta, el Vault y las fotos conservan sus contratos propios.

<a id="match"></a>
## Resultado de match del Crush

- **RP-MAT-004.** `MatchStatus` es un Value Object cerrado con `0 UNKNOWN`, `1 MATCH` y `2 NO_MATCH`. No existe una tabla de catálogo. `match_status_id` se guarda únicamente en el Crush para que filtros, conteos y estadísticas consulten el resultado sin reconstruir toda la historia.
- **RP-MAT-005.** La primera [evidencia relacional de match](RELATIONSHIPS.md#evidencia-match) fija en el Crush `MATCH`, `first_match_situation_id` y `first_match_date`. Abrir otro período no reinicia ese resultado. Si una corrección o eliminación retira la única evidencia que lo establecía, las tres proyecciones se recalculan desde los hitos conservados.
- **RP-MAT-006.** Sin evidencia conservada de match, `match_status_id` es `UNKNOWN` mientras el período más reciente está vivo y `NO_MATCH` cuando todos los períodos están cerrados. Abrir un período nuevo en una situación 1–4 transforma `NO_MATCH` en `UNKNOWN`; abrirlo directamente en una situación 5–9 establece `MATCH`.
- **RP-MAT-007.** `first_match_situation_id` y `first_match_date` identifican la primera evidencia cronológica de match entre todos los períodos conservados del Crush. Son nulos cuando `match_status_id` no es `MATCH` y, cuando tienen valor, la situación pertenece necesariamente a los IDs 5–9 de `RelationshipSituation`.
- **RP-MAT-008.** Crear, corregir o eliminar un hito o período recalcula `match_status_id`, `first_match_situation_id` y `first_match_date` del Crush desde los hitos que permanecen, dentro de la misma transacción que modifica la historia relacional.

<a id="alta"></a>
## Alta de un Crush

- **BR-4.01.** Crear un Crush exige exactamente que la persona informe: nombre; género del catálogo; país del catálogo; Ducky; fecha de encuentro; edad confirmada; una situación relacional inicial 1–9; y una experiencia inicial completa con lugar, descripción y satisfacción. Nombre y género se guardan únicamente en el Vault; país, Ducky, fecha de encuentro, edad, situación y satisfacción forman estructura funcional; lugar y descripción permanecen en el Vault. Cuando la API acepta el alta, genera `age_as_of` con su fecha vigente y la conserva junto con la edad.
- **RP-CRU-004.** El alta es una única acción compuesta que incluye obligatoriamente el Crush, su período relacional inicial, su hito inicial y su experiencia inicial completa, todos correlacionados en `met_at`. Un formulario incompleto puede conservar temporalmente lo introducido para corregirlo, pero no crea un Crush ni un cambio pendiente. Cuando todos los datos son válidos, la aplicación puede conservar la acción completa como pendiente hasta sincronizarla; el servidor crea todas sus partes o ninguna.
- **RP-CRU-005.** La persona elige el Ducky del Crush entre `1 casualDuck`, `2 loveDuck`, `3 devilDuck` y `4 uglyDuck`. La aplicación no lo asigna ni lo cambia automáticamente a partir de la situación relacional, las estadísticas, la actividad o la IA.

La [situación relacional inicial](RELATIONSHIPS.md#inicial) y la [experiencia inicial obligatoria](EXPERIENCES.md#inicial) se definen en sus documentos respectivos.

<a id="perfil"></a>
## Perfil del Crush

### Campos exclusivos del Vault

- **RP-CRU-001.** El nombre y el `gender_id` del Crush son información sensible cuya única representación funcional se conserva dentro del payload Vault. El servidor almacena ese payload cifrado como parte del Vault, sin poder interpretar su contenido. La API funcional de Crushes no admite `gender_id` en altas ni ediciones y tampoco lo devuelve en listas o fichas.
- **RP-CRU-012.** Profesión; prefijo y número de WhatsApp; usuarios de Instagram, Facebook, Snapchat y TikTok; y día y mes de cumpleaños tienen su única representación funcional dentro del Vault. La búsqueda privada se ejecuta en el dispositivo mientras el Vault está desbloqueado y utiliza el contenido descifrado únicamente durante esa sesión.
- **RP-CRU-023.** Nombre y profesión admiten hasta 191 caracteres cada uno. Cada usuario de Instagram, Facebook, Snapchat o TikTok admite hasta 255 caracteres. El contacto de WhatsApp se compone siempre de un prefijo seleccionado del catálogo de países y un número de hasta 20 dígitos; añadirlo guarda ambos componentes y quitar el número elimina el contacto completo.

### Rasgos físicos estructurados

- **RP-CRU-016.** Altura, color de ojos, color de pelo, tipo de cuerpo y color de piel son cinco campos opcionales independientes. Cada uno admite una opción de su catálogo y se conserva mediante su ID en la estructura funcional de la API. Ninguno contiene texto libre, se deriva de una foto ni se infiere mediante IA.
- **RP-CRU-017.** Añadir, cambiar o quitar un rasgo físico no modifica los demás rasgos, las valoraciones, el género, la edad, el país, el Ducky ni la situación relacional del Crush.
- **RP-CRU-018.** Un rasgo físico seleccionado puede cambiarse por otra opción de su catálogo o quitarse. Quitar la opción —incluido volver a tocar la opción ya seleccionada— guarda `null` para ese único campo.

Los catálogos de rasgos físicos admiten exactamente estas opciones; el texto visible puede traducirse, pero conserva el mismo ID y significado:

| Campo | Opciones admitidas |
|---|---|
| Altura | `1 short`, `2 average`, `3 tall` |
| Color de ojos | `1 brown`, `2 green`, `3 blue`, `4 gray`, `5 hazel`, `6 amber` |
| Color de pelo | `1 black`, `2 brown`, `3 blond`, `4 red`, `5 gray`, `6 white`, `7 none` |
| Tipo de cuerpo | `1 slim`, `2 athletic`, `3 average`, `4 curvy`, `5 large` |
| Color de piel | `1 light`, `2 fair`, `3 medium`, `4 brown`, `5 black` |

### Lectura, búsqueda y edición del perfil

- **RP-CRU-019.** Después de iniciar sesión, ninguna lista, búsqueda, mapa, Timeline ni ficha privada de la aplicación queda visible mientras el Vault esté bloqueado. Al iniciar la aplicación y cada vez que regresa desde segundo plano, MDL bloquea el Vault y cubre el contenido privado hasta que la persona supera el reconocimiento biométrico. Esta regla no se aplica a las pantallas públicas o previas al inicio de sesión.
- **RP-CRU-020.** La ficha del Crush combina la estructura funcional obtenida de la API con el contenido privado descifrado del Vault. Muestra los campos del inventario que tengan valor —incluidos rasgos físicos, valoraciones, profesión, WhatsApp y redes—, pero no expone como datos de perfil los campos administrados por el sistema.
- **RP-CRU-021.** La búsqueda privada encuentra coincidencias parciales, sin distinguir mayúsculas ni acentos, exclusivamente por el nombre del Crush o por el nombre visible de su país. El término y la comparación del nombre se mantienen en el dispositivo con el Vault desbloqueado; la API sólo recibe la paginación y los filtros estructurales ya aplicados y nunca recibe ni persiste el texto buscado.
- **RP-CRU-022.** Desde la ficha pueden editarse nombre, género, país, Ducky, fecha de encuentro, edad confirmada, profesión, WhatsApp, usuarios de redes, cumpleaños, cada valoración y cada rasgo físico. Nombre, género, país, Ducky, fecha de encuentro y edad son obligatorios y no pueden borrarse. Cuando se envía una edad nueva, el cliente envía únicamente `age`: la API genera `age_as_of` al aceptar el cambio y rechaza cualquier intento del cliente de proporcionarlo. Los campos opcionales pueden añadirse, sustituirse o quitarse; quitar uno envía un borrado explícito y lo conserva como `null`, mientras que omitir un campo sin cambios conserva su valor actual. El signo zodiacal y la valoración global siguen siendo derivados y nunca se editan directamente.

### Fecha de encuentro

- **RP-CRU-007.** La fecha de encuentro siempre registra un hecho ya ocurrido: nunca puede ser futura. Puede moverse desde el perfil del Crush hacia una fecha anterior o posterior a la actualmente guardada y no tiene un límite histórico mínimo. Si el Crush tiene alguna experiencia posterior al bundle inicial —con o sin cambio relacional—, la nueva fecha debe ser estrictamente anterior a la primera.
- **RP-CRU-008.** El calendario aplica como límite superior el menor entre hoy y el día anterior a la experiencia posterior más antigua del mismo Crush. Si no existe ninguna, la última fecha seleccionable es hoy. Para encontrarla tiene en cuenta experiencias confirmadas o pendientes. El período inicial, su hito inicial y la experiencia inicial no limitan la selección porque se desplazan juntos con `met_at`. Las experiencias de otros Crushes tampoco la limitan.
- **RP-CRU-009.** Corregir la fecha de encuentro cambia conjuntamente la fecha del período relacional inicial, de su hito inicial y de la experiencia inicial. Siguen siendo los mismos elementos y conservan sus asociaciones. Todas las experiencias posteriores y sus eventuales hitos vinculados conservan su fecha y sus asociaciones actuales.

**Ejemplo.** Si `met_at` es el 1 de septiembre y el primer acontecimiento independiente ocurre el 5 de septiembre, puede cambiarse al 4 de septiembre o a cualquier fecha anterior, incluso de años anteriores. No puede cambiarse al 5 de septiembre ni a una fecha posterior.

### Valoraciones

- **BR-4.02.** Belleza, conexión emocional, personalidad y atractivo son valoraciones opcionales que sólo admiten valores enteros: 1, 2, 3, 4 o 5; la API rechaza cualquier otro valor. Se conservan como enteros y, después del alta, cada una puede añadirse, sustituirse o quitarse desde el perfil del Crush. Quitarla guarda `null` para esa valoración. La valoración global es la única que admite decimales: MDL la recalcula en cada alta, cambio o eliminación de una valoración como la media aritmética de las valoraciones informadas, redondeada a dos decimales. La persona no puede introducir ni modificar el global. Si no queda ninguna valoración informada, el global también es `null` y no se muestra.

<a id="edad"></a>
### Edad, cumpleaños y zodíaco

#### Datos de referencia

- **RP-AGE-001.** La edad que se confirma en el alta o en una edición debe ser un entero entre 18 y 80 años inclusive. La edad mostrada puede superar 80 posteriormente por el transcurso del tiempo; ese límite sólo valida una nueva confirmación introducida por la persona.
- **RP-AGE-002.** La edad confirmada indica cuántos años tenía el Crush en la fecha de confirmación. El cliente envía únicamente `age`. Cuando la API acepta esa edad, genera `age_as_of` con la fecha vigente del servidor y conserva ambos datos juntos como una sola pareja.
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
- **RP-AGE-011.** Cuando la persona corrige la edad, declara la edad actual del Crush. La API genera como nueva fecha de confirmación la fecha en que acepta ese cambio. La persona no puede introducir ni escoger `age_as_of`, y la API rechaza ese campo si un cliente intenta enviarlo.
- **RP-AGE-012.** Añadir o cambiar el cumpleaños conserva como referencia la edad vigente calculada con los datos anteriores y renueva su fecha de confirmación. La nueva fecha de cumpleaños se utiliza para las actualizaciones posteriores de edad.
- **RP-AGE-013.** Quitar el cumpleaños conserva la edad vigente, renueva su fecha de confirmación y elimina el día y mes del cumpleaños y su signo zodiacal.
- **RP-AGE-014.** Si se corrigen simultáneamente edad y cumpleaños, prevalece la edad introducida expresamente.
- **RP-AGE-015.** La primera aceptación de una operación de cambio de edad fija `age_as_of`. Reintentar esa misma operación conserva la edad y la fecha ya fijadas, aunque la respuesta original se haya perdido o el reintento ocurra otro día; sólo una operación nueva que cambie la edad genera otra fecha de confirmación.

<a id="fotos"></a>
### Fotos del Crush

- **RP-FOT-001.** En la beta, las fotos del Crush son privadas y se guardan únicamente en el dispositivo. MDL no las sube al Vault ni ofrece una copia de seguridad remota de ellas.
- **RP-FOT-002.** Antes de guardar una foto, MDL elimina sus metadatos de cámara y ubicación. La foto queda asociada a la cuenta y al Crush correspondientes.
- **RP-FOT-003.** Las fotos y sus copias auxiliares se protegen mediante los controles del dispositivo y se excluyen de copias de seguridad no autorizadas.
- **RP-FOT-004.** Eliminar una foto retira de MDL la imagen guardada, sus miniaturas y sus demás copias auxiliares en el dispositivo.
- **RP-FOT-005.** Eliminar un Crush elimina también sus fotos guardadas por MDL. Al confirmar la baja de la cuenta, se eliminan las fotos de esa cuenta a las que la aplicación tiene acceso en el dispositivo.
- **RP-FOT-006.** Recuperar el Vault no recupera las fotos del Crush. MDL no garantiza el borrado remoto de fotos en un dispositivo sin conexión.
- **RP-FOT-007.** Cada Crush puede tener como máximo cinco fotos guardadas. Eliminar una foto libera inmediatamente ese lugar para añadir otra.
- **RP-FOT-008.** Cada foto puede ocupar como máximo 10 MB. MDL rechaza una foto que supere ese tamaño y no altera las fotos ya guardadas.
- **RP-FOT-009.** El Crush sólo admite fotografías. MDL no ofrece ni admite subir videos como contenido del Crush.

Las restricciones al editar el perfil del Crush mientras hay cambios pendientes se definen en [Guardado y sincronización](SYNC.md#conflictos).

<a id="mapa"></a>
## Mapa de Crushes por país

- **RP-MAP-001.** El mapa permite consultar los Crushes por país. No permite crearlos ni modificar sus datos.
- **RP-MAP-002.** El mapa incluye únicamente Crushes activos: excluye los archivados, eliminados o seudonimizados. Los agrupa según el país registrado en su perfil. El punto representa al país y nunca la ubicación precisa de una persona.
- **RP-MAP-003.** Sólo puede estar abierta una ventana de información del mapa a la vez. Tocar el fondo la cierra.
- **RP-MAP-004.** Un país sin Crushes activos no muestra un marcador. Si no hay datos que representar, el mapa informa de ello en el idioma elegido en la aplicación.
- **RP-MAP-005.** Volver del mapa a la lista de Crushes conserva los filtros y los datos de la lista.

<a id="archivo-eliminacion"></a>
## Archivar y restaurar un Crush

- **RP-ARC-001.** Archivar un Crush lo retira de la lista de activos y del mapa, conserva su ficha e historia y lo muestra en una pantalla propia de archivados. Esa pantalla tiene la misma presentación y capacidades de consulta que la lista de activos, pero su fuente contiene exclusivamente Crushes archivados y la acción de ciclo de vida disponible es restaurarlos. Restaurar un Crush lo devuelve a la lista de activos y vuelve a incluirlo en el mapa.
- **RP-ARC-002.** Un cambio pendiente en el Timeline del Crush no impide archivarlo o restaurarlo si su alta ya está confirmada. La lista de archivados muestra ese cambio pendiente y permite resolverlo.
- **RP-ARC-003.** Mientras el alta del Crush siga pendiente de confirmación, no puede archivarse ni se le pueden añadir experiencias o cambios relacionales.

## Eliminar un Crush

- **RP-DEL-001.** Eliminar un Crush exige confirmación y retira del diario el Crush, toda su historia —experiencias, períodos e hitos—, sus asociaciones, actividad, contenido del Vault y fotos vinculadas.
- **RP-DEL-002.** La eliminación individual de un Crush elimina esa ficha y todos sus datos vinculados. No aplica la conservación estructural prevista para la [baja completa de la cuenta](DATA_LIFECYCLE.md#baja).
- **RP-DEL-003.** Antes de pedir la eliminación funcional al servidor, la aplicación conserva durablemente la intención de borrar. No retira todavía su copia local. Si la petición falla o la respuesta se pierde, mantiene la operación pendiente y el Crush local hasta poder reanudarla y confirmar el resultado.
- **RP-DEL-004.** Reanudar o repetir una eliminación es idempotente: si el servidor ya borró el Crush durante un intento cuya respuesta se perdió, el reintento se considera satisfactorio y no vuelve a crear ni necesita recuperar sus asociaciones. Una respuesta de «no encontrado» sólo puede confirmar el borrado cuando existe un intento anterior registrado durablemente; en el primer intento se trata como error.
- **RP-DEL-005.** Sólo después de confirmar el borrado funcional, la aplicación retira el contenido local del Vault, las fotos y el resto de copias vinculadas. Entonces el Crush y sus datos dejan de estar disponibles. La destrucción definitiva del contenido cifrado en infraestructura sigue los plazos de [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).
- **RP-DEL-006.** La acción para eliminar el Crush completo se ofrece desde su fila en la lista de Crushes activos. No aparece dentro del bloque inicial, la ficha ni las acciones de su experiencia, período o hito. Un Crush archivado debe restaurarse antes de poder eliminarse.
