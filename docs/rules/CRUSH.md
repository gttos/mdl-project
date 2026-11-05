# Reglas de producto — Crush y perfil del Crush

[Índice de reglas](../PRODUCT.md) · Alta, datos y edición del perfil del Crush, resultado de match, edad, cumpleaños, zodíaco, valoraciones, fotos, consulta, mapa, archivo y eliminación.

**Perfil del Crush** significa la ficha privada de una persona registrada en el diario. El [perfil de la cuenta de usuario](ACCOUNT.md#perfil-cuenta) corresponde a quien utiliza MDL.

<a id="inventario"></a>
## Inventario funcional del perfil del Crush

- **RP-CRU-013.** El perfil del Crush en la beta se compone del siguiente inventario cerrado.

Para este inventario, «legible por la API» significa que el servidor interpreta el valor y puede utilizarlo para las finalidades indicadas. «Vault opaco» significa que el servidor sólo conserva ciphertext y no puede interpretar el campo. «Administrado por el sistema» identifica estructura necesaria para ownership, sincronización o ciclo de vida que la persona no edita directamente. «Sólo dispositivo» y «transitorio» no se incorporan a la estructura durable legible de la API.

| Tratamiento | Dato exacto | Condición y uso autorizado |
| --- | --- | --- |
| API legible | Identificador del Crush | Lo genera MDL, es inmutable y referencia la misma ficha en sus operaciones vinculadas. |
| API legible | Cuenta propietaria | Es inmutable y sólo autoriza a la cuenta propietaria a consultar o modificar el Crush. |
| API legible | País | Es obligatorio, pertenece al catálogo de países y se utiliza para perfil, mapa y estadísticas autorizadas. |
| API legible | Ducky | Es obligatorio y sólo admite los cuatro valores definidos por RP-CRU-005. |
| API legible | Edad confirmada | Es obligatoria y entera entre 18 y 80 años inclusive. Siempre se interpreta junto con `age_as_of`. |
| API legible | Fecha de confirmación de la edad | Es obligatoria junto con `age` y registra el día civil en que la persona confirmó esa edad en la app. El cliente la fija y conserva con la operación durable; la API valida y persiste la pareja conforme a RP-AGE-002/011/015. No es un campo que la persona elija por separado. |
| API legible | Fecha de encuentro | Es obligatoria y rige el inicio cronológico del Crush, su período inicial y su experiencia inicial obligatoria. |
| API legible derivada | Resultado de match | Es obligatorio y admite `0 UNKNOWN`, `1 MATCH` o `2 NO_MATCH`. Se recalcula conforme a [Resultado de match](#match). |
| API legible derivada | Primer match | Son nulos mientras no exista match y señalan la primera situación 5, 6, 7 u 8 y su fecha cuando lo hay. |
| API legible derivada | Signo zodiacal | Es opcional y sólo se deriva del día y mes del cumpleaños guardados en el Vault. No permite enviar el cumpleaños a la API. |
| API legible | Altura | Es opcional y sólo admite una opción del catálogo de altura. |
| API legible | Color de ojos | Es opcional y sólo admite una opción del catálogo de ojos. |
| API legible | Color de pelo | Es opcional y sólo admite una opción del catálogo de pelo. |
| API legible | Tipo de cuerpo | Es opcional y sólo admite una opción del catálogo corporal. |
| API legible | Color de piel | Es opcional y sólo admite una opción del catálogo de piel. |
| API legible | Adjetivos de personalidad | Selección opcional de hasta tres IDs por bloque del catálogo compartido de RP-ADJ-001..010; describe únicamente este Crush. |
| API legible | Estilo social | Selección opcional de RP-ADJ-010, independiente de las cualidades. |
| API legible | Atracción física | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible | Conexión emocional | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible | Forma de ser | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible | Atracción sexual | Es opcional y sólo admite un entero entre 1 y 5. |
| API legible derivada | Valoración global | Es opcional, sólo la calcula MDL y puede contener hasta dos decimales conforme a BR-4.02. |
| Administrado por el sistema | Estado archivado | Cambia únicamente mediante archivar o restaurar y no elimina la ficha ni su historia. |
| Administrado por el sistema | Marcador técnico de seudonimización | Existe en la implementación pendiente de adaptar. No autoriza conservar el Crush tras la baja: el destino funcional es el borrado íntegro de RP-BAJ-017. |
| Administrado por el sistema | Correlación de la acción compuesta | Identifica reintentos y las partes vinculadas del alta sin crear otro Crush ni otra experiencia inicial. No es editable ni visible como dato del perfil. |
| Administrado por el sistema | Fechas técnicas de creación, actualización, actividad, edición y eliminación | Sólo sostienen orden, sincronización, archivo, borrado y conservación. No describen a la persona ni pueden sustituir fechas funcionales como `met_at` o `age_as_of`. |
| Vault opaco | Nombre | Es obligatorio, contiene texto y admite hasta 191 caracteres. Sólo se muestra o edita en el dispositivo con el Vault desbloqueado. |
| Vault opaco | Género | Es obligatorio, pertenece al catálogo de género y tiene su única representación funcional dentro del Vault. |
| Vault opaco | Profesión | Es texto opcional de hasta 191 caracteres y no llega en claro a la API. |
| Vault opaco | Prefijo y número de WhatsApp | Forman un único contacto opcional dentro del Vault: el prefijo se elige del catálogo de países y el número admite únicamente dígitos, con un máximo de 20. |
| Vault opaco | Usuarios de redes | Son cuatro textos opcionales independientes de hasta 255 caracteres cada uno. |
| Vault opaco | Cumpleaños | Es opcional y se compone únicamente de día y mes. |
| Sólo dispositivo | Fotos del Crush | No se transmiten a la API ni al Vault remoto y se rigen por RP-FOT-001..012. |
| Otros dominios vinculados | Situación relacional, períodos, hitos, experiencias y satisfacción | Se rigen por sus documentos correspondientes. El Crush sólo conserva las proyecciones de match declaradas arriba para consulta directa; no duplica el historial. |

- **RP-CRU-014.** País, Ducky, signo zodiacal, altura, color de ojos, color de pelo, tipo de cuerpo y color de piel se conservan como opciones de sus catálogos, mediante sus IDs. No son texto libre y no pertenecen al Vault por el solo hecho de describir a una persona.
- **RP-CRU-015.** Los campos estructurales del Crush son exactamente los enumerados como «API legible» o «Administrado por el sistema» en RP-CRU-013. Su contenido privado está formado por nombre, género, profesión, WhatsApp, Instagram, Facebook, Snapchat, TikTok y cumpleaños, con la obligatoriedad indicada en el mismo inventario. El dispositivo cifra ese contenido antes de transferirlo, conserva localmente la entry cifrada y sincroniza con el Vault remoto únicamente ciphertext como copia cifrada recuperable. El contenido sólo se descifra en la memoria de un dispositivo autorizado que haya desbloqueado o recuperado el mismo Vault; la API funcional continúa interpretando únicamente la estructura enumerada. La situación relacional, las experiencias, la cuenta, el Vault y las fotos conservan sus contratos propios.

<a id="match"></a>
## Resultado de match del Crush

- **RP-MAT-004.** El resultado de match tiene tres valores: 0 desconocido, 1 match y 2 sin match. Pertenece al Crush y es la misma autoridad para filtros, conteos y estadísticas.
- **RP-MAT-005.** La primera [evidencia relacional de match](RELATIONSHIPS.md#evidencia-match) fija en el Crush `MATCH`, `first_match_situation_id` y `first_match_date`. Abrir otro período no reinicia ese resultado. Si una corrección o eliminación retira la única evidencia que lo establecía, las tres proyecciones se recalculan desde los hitos conservados.
- **RP-MAT-006.** Sin evidencia conservada de match, `match_status_id` es `UNKNOWN` mientras el período más reciente está vivo y `NO_MATCH` cuando todos los períodos están cerrados. Abrir un período nuevo en una situación 1–4 o 9 transforma `NO_MATCH` en `UNKNOWN`; abrirlo directamente en una situación 5, 6, 7 u 8 establece `MATCH`. Una evidencia conservada de match no se pierde por abrir otro período ni por elegir después una situación que no lo establezca.
- **RP-MAT-007.** `first_match_situation_id` y `first_match_date` identifican la primera evidencia cronológica de match entre todos los períodos conservados del Crush. Son nulos cuando `match_status_id` no es `MATCH` y, cuando tienen valor, la situación pertenece necesariamente a los IDs 5, 6, 7 u 8 de `RelationshipSituation`.
- **RP-MAT-008.** Crear, corregir o eliminar un hito o período recalcula `match_status_id`, `first_match_situation_id` y `first_match_date` del Crush desde los hitos que permanecen, dentro de la misma transacción que modifica la historia relacional.

<a id="alta"></a>
## Alta de un Crush

- **BR-4.01.** Crear un Crush exige exactamente que la persona informe: nombre; género del catálogo; país del catálogo; Ducky; fecha de encuentro; edad confirmada; una situación relacional inicial 1–9; y una experiencia inicial completa con lugar, descripción y satisfacción. Nombre y género se guardan únicamente en el Vault; país, Ducky, fecha de encuentro, edad, situación y satisfacción forman estructura funcional; lugar y descripción permanecen en el Vault. La app fija y conserva la fecha de confirmación de edad conforme a RP-AGE-002; la API recibe y persiste esa pareja también en el alta.
- **RP-CRU-004.** El alta es una única acción compuesta que incluye obligatoriamente el Crush, su período relacional inicial, su hito inicial y su experiencia inicial completa, todos correlacionados en `met_at`. Un formulario incompleto puede conservar temporalmente lo introducido para corregirlo, pero no crea un Crush ni un cambio pendiente. Cuando todos los datos son válidos, la aplicación puede conservar la acción completa como pendiente hasta sincronizarla; el servidor crea todas sus partes o ninguna.
- **RP-CRU-005.** La persona elige el Ducky del Crush entre `1 casualDuck`, `2 loveDuck`, `3 devilDuck` y `4 uglyDuck`. La aplicación no lo asigna ni lo cambia automáticamente a partir de la situación relacional, las estadísticas, la actividad o la IA.

Los patitos expresan una impresión subjetiva de quien utiliza el diario, independiente de la situación relacional y de la evidencia de match. Cada opción conserva su identidad y significado.

| ID | Nombre ES | Explicación ES | Nombre EN | Explicación EN |
|---|---|---|---|---|
| 1 `casualDuck` | Ni fu ni fa | Hay algo que te gusta y te apetece conocerle, sin una ilusión afectiva ni un deseo sexual intensos. | So-so | Something appeals to you and you’d like to get to know them, without feeling strongly drawn to them romantically or sexually. |
| 2 `loveDuck` | Amor de mi vida | Te despierta ilusión e interés afectivo. El nombre expresa lo que sientes, no que sea correspondido ni que seáis pareja. | Love of my life | You feel excited about them romantically. The name expresses your feelings, not mutual interest or being a couple. |
| 3 `devilDuck` | Buenorr@ | Te despierta deseo sexual, haya pasado algo entre vosotros o no. | Hot | You feel sexual desire, whether or not anything has happened between you. |
| 4 `uglyDuck` | Simpátic@ | Su físico no te atrae especialmente. «Simpátic@» es un guiño, no una valoración de su personalidad. | Nice personality | Their looks don’t especially appeal to you. “Nice personality” is a playful euphemism, not a rating of their character. |

Alta, edición y corrección permiten consultar el significado de los cuatro patitos. Consultar o cerrar la explicación conserva selección, errores y borrador. La selección es única y tocar de nuevo la opción elegida la conserva. Los mismos significados se utilizan en filtros, ficha, estadísticas y demo.

| Escenario | Resultado esperado |
|---|---|
| GWT-DUCK-01 — Catálogo ES/EN | Los cuatro nombres y explicaciones coinciden con la tabla; los IDs 1–4 y su codificación se conservan. |
| GWT-DUCK-02 — Ayudas y selección | Con un patito elegido, abrir y cerrar significados conserva el mismo patito y los demás campos; seleccionar otra fila sustituye sólo el patito y repetirla no lo desmarca. |
| GWT-DUCK-03 — Alta y edición | Ambos consumidores del formulario compartido permiten leer las cuatro explicaciones completas, mantener una selección visible y seguir con las mismas validaciones y acciones. |

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

<a id="adjetivos"></a>
### Adjetivos de personalidad y catálogo compartido

- **RP-ADJ-001.** Cuenta y Crush utilizan el mismo catálogo cerrado de 29 cualidades y rasgos descriptivos de la tabla activa. La app permite consultarlo y seleccionar sus entradas. Cada ID conserva su significado.
- **RP-ADJ-002.** Cada selección contiene IDs enteros distintos del catálogo, con cero a tres cualidades por bloque y un máximo total de doce. Guardar sustituye el conjunto; omitir el campo conserva su valor y `[]` lo vacía. `null`, IDs desconocidos, duplicados, tipos inválidos o superar tres en un bloque rechazan el cambio completo. La comparación de conjuntos ignora el orden.
- **RP-ADJ-003.** La API guarda las cualidades y el estilo social seleccionados en la estructura de la cuenta o del Crush propietario. La edición durable del Crush los conserva en su snapshot local cifrado conforme a RP-DUR-010/011, fuera del payload privado del Vault. Analítica, logs y diagnóstico excluyen las selecciones, sus IDs y etiquetas.
- **RP-ADJ-004.** La persona selecciona y retira manualmente cualidades de los cuatro bloques y consulta sus definiciones. Cada bloque admite hasta tres selecciones independientes de los demás. Cuenta y Crush permiten consultar las mismas opciones en español e inglés.
- **RP-ADJ-005.** Una selección vacía conserva un conjunto vacío y la ausencia de estilo social. Las lecturas distinguen ausencia válida de datos incompletos o inválidos: estos últimos conservan el contenido y producen error sin inventar opciones.
- **RP-ADJ-006.** La edición del Crush permite modificar cualidades y estilo social conservando los demás datos. Guardado, consulta, proyección, reintento y corrección conservan la selección exacta; un apartado se considera modificado cuando cambia su conjunto.
- **RP-ADJ-007.** El perfil de cuenta consulta y edita su selección con el mismo catálogo y selector. Cada cuenta y cada Crush conservan selecciones independientes. El guardado conserva los demás campos y adopta el perfil confirmado por API conforme a RP-PER-001/004.
- **RP-ADJ-008.** Las selecciones siguen el archivo y borrado de su entidad. La baja elimina tanto la selección personal de la cuenta como las de sus Crushes conforme a [RP-BAJ-017](DATA_LIFECYCLE.md#baja), sin conservarlas para agregados.
- **RP-ADJ-009.** Catálogo, edición y lectura conservan selecciones independientes para cuenta y Crush. El dashboard utiliza las selecciones del Crush conforme a RP-EST-028..030.
- **RP-ADJ-010.** «Estilo social» es una selección opcional e independiente: `1 Introvertido`, `2 Intermedio` o `3 Extrovertido`. Admite una opción; elegir otra la sustituye y volver a tocar la elegida la retira. No consume el límite de doce cualidades. Cuenta y Crush lo editan y muestran con su selección propia. Omitir `social_style_id` conserva el valor, un ID entero válido lo sustituye y `null` lo retira. Un valor inválido rechaza la edición completa. Sigue RP-ADJ-003/006/007/008.

#### Catálogo activo

Los bloques son Forma de ser, Con los demás, En pareja y Valores. Cada cualidad conserva el bloque y significado definidos en la tabla.

| ID | Bloque | ES | EN | Definición ES | Definición EN |
|---|---|---|---|---|---|
| 9 | Forma de ser | Alegre | Cheerful | Suele mostrar buen humor y disfrutar de los momentos cotidianos. | Usually shows good spirits and enjoys everyday moments. |
| 5 | Forma de ser | Tranquilo | Calm | Suele mantener la calma y actuar sin precipitarse. | Usually stays calm and acts without rushing. |
| 81 | Forma de ser | Espontáneo | Spontaneous | Se anima a hacer planes y expresar lo que siente sin prepararlo todo. | Enjoys making plans and expressing feelings without planning everything in advance. |
| 23 | Forma de ser | Curioso | Curious | Se interesa por conocer cosas, ideas y experiencias nuevas. | Takes an interest in new things, ideas and experiences. |
| 21 | Forma de ser | Creativo | Creative | Tiene imaginación y propone ideas o maneras originales de hacer las cosas. | Has imagination and comes up with original ideas or ways of doing things. |
| 17 | Forma de ser | Inteligente | Intelligent | Comprende ideas y encuentra soluciones con facilidad. | Understands ideas and finds solutions easily. |
| 7 | Forma de ser | Optimista | Optimistic | Suele ver posibilidades favorables incluso ante dificultades. | Usually sees positive possibilities even when facing difficulties. |
| 77 | Forma de ser | Independiente | Independent | Toma sus propias decisiones y mantiene su espacio personal. | Makes their own decisions and maintains personal space. |
| 37 | Con los demás | Amable | Kind | Trata a otras personas con cordialidad y consideración. | Treats other people with warmth and consideration. |
| 82 | Con los demás | Atento | Attentive | Escucha, se interesa y recuerda lo que le cuentas. | Listens, shows interest and remembers what you tell them. |
| 83 | Con los demás | Divertido | Fun | Tiene sentido del humor y hace que los momentos resulten entretenidos. | Has a sense of humour and makes time together enjoyable. |
| 51 | Con los demás | Empático | Empathetic | Procura comprender cómo se sienten otras personas. | Tries to understand how other people feel. |
| 84 | Con los demás | Conversador | Talkative | Disfruta hablando y mantiene una conversación con facilidad. | Enjoys talking and keeps a conversation going easily. |
| 39 | Con los demás | Sociable | Sociable | Disfruta conociendo gente y compartiendo tiempo con otras personas. | Enjoys meeting people and spending time with others. |
| 85 | Con los demás | Tímido | Shy | Puede sentir vergüenza o inhibición al conocer gente. | May feel self-conscious or inhibited when meeting people. |
| 86 | Con los demás | Reservado | Reserved | Comparte sus asuntos personales poco a poco. | Shares personal matters gradually. |
| 43 | En pareja | Cariñoso | Affectionate | Expresa afecto con palabras, contacto o gestos cercanos. | Expresses affection through words, touch or warm gestures. |
| 87 | En pareja | Romántico | Romantic | Disfruta expresando el amor y preparando momentos especiales en pareja. | Enjoys expressing love and creating special moments as a couple. |
| 11 | En pareja | Apasionado | Passionate | Expresa con intensidad su interés y deseo en el vínculo. | Expresses interest and desire intensely in the relationship. |
| 88 | En pareja | Detallista | Thoughtful | Tiene pequeños gestos para cuidar o sorprender. | Makes small gestures to care for or surprise someone. |
| 3 | En pareja | Paciente | Patient | Sabe esperar y dar tiempo sin presionar. | Can wait and give others time without pressure. |
| 41 | En pareja | Comunicativo | Communicative | Expresa con claridad lo que piensa, siente y necesita. | Clearly expresses thoughts, feelings and needs. |
| 89 | En pareja | Comprensivo | Understanding | Escucha las circunstancias de la otra persona y procura entenderlas. | Listens to the other person's circumstances and tries to understand them. |
| 57 | Valores | Sincero | Honest | Habla con honestidad sobre lo que piensa y siente. | Speaks honestly about thoughts and feelings. |
| 55 | Valores | Leal | Loyal | Mantiene su apoyo y cuida los compromisos con las personas cercanas. | Maintains support and honours commitments to people close to them. |
| 90 | Valores | Respetuoso | Respectful | Tiene en cuenta los límites, decisiones y opiniones de otras personas. | Respects other people's boundaries, decisions and opinions. |
| 63 | Valores | Responsable | Responsible | Cumple sus compromisos y se hace cargo de sus actos. | Keeps commitments and takes responsibility for their actions. |
| 45 | Valores | Generoso | Generous | Comparte su tiempo, atención o recursos sin esperar algo a cambio. | Shares time, attention or resources without expecting anything in return. |
| 53 | Valores | Tolerante | Tolerant | Acepta diferencias de opiniones, costumbres y maneras de vivir. | Accepts differences in opinions, customs and ways of life. |

«Estilo social / Social style» utiliza su propio catálogo de IDs:

| ID | ES | EN | Definición ES | Definición EN |
|---|---|---|---|---|
| 1 | Introvertido | Introverted | Suele preferir encuentros con pocas personas y disponer de tiempo a solas. | Usually prefers small gatherings and having time alone. |
| 2 | Intermedio | In between | Combina el gusto por socializar con la preferencia por pasar tiempo a solas. | Enjoys socialising as well as spending time alone. |
| 3 | Extrovertido | Extroverted | Suele buscar interacción social y expresarse abiertamente. | Usually seeks social interaction and expresses themselves openly. |

<a id="escenarios-adjetivos"></a>
#### Escenarios de adjetivos

| ID | Dado / Cuando / Entonces | Reglas |
|---|---|---|
| GWT-ADJ-01 | Dado el catálogo activo, su lectura entrega los mismos IDs, significados y agrupaciones ES/EN. El selector ofrece las 29 cualidades y los tres estilos. | RP-ADJ-001/004/010 |
| GWT-ADJ-02 | Dado un bloque con tres cualidades, una cuarta se rechaza y los demás bloques conservan su capacidad. Se admiten bloques vacíos y hasta doce cualidades; una entrada API inválida rechaza la edición completa. | RP-ADJ-002/004 |
| GWT-ADJ-03 | Dado un conjunto informado, omitir conserva, otro conjunto sustituye y `[]` vacía; reordenar no genera cambio. | RP-ADJ-002 |
| GWT-ADJ-04 | Dada una entidad creada sin selección, su lectura publica `adjectiveIds: []` y `socialStyleId: null`. Un campo de respuesta ausente o inválido falla cerrado. El alta conserva sus pasos. | RP-ADJ-005, RP-BASE-009 |
| GWT-ADJ-05 | Dada una edición del Crush guardada offline, al relanzar conserva conjunto, identidad y apartado; retry y corrección usan ese snapshot y preservan campos ocultos. | RP-ADJ-006, RP-DUR-001..011, RP-RES-008..010 |
| GWT-ADJ-06 | Dado un guardado confirmado, revisión y ficha muestran la selección; visitar sin cambiar no escribe ni añade apartados modificados. | RP-ADJ-002/006 |
| GWT-ADJ-07 | Dados dos Crushes y una selección personal, editar cualquiera sólo cambia su entidad y propietario; otra cuenta no puede leer ni modificar esas selecciones. | RP-ADJ-003/007, RP-ID-007 |
| GWT-ADJ-08 | Dada una baja o borrado, las selecciones siguen el ciclo de su entidad. La baja retira la selección de cuenta y un fallo revierte el cambio completo. | RP-ADJ-008, RP-BAJ-003 |
| GWT-ADJ-09 | Dadas selecciones del Crush, el dashboard calcula cualidades y estilo social conforme a sus reglas. Editar la selección de cuenta conserva ese resultado. | RP-ADJ-007/009, RP-EST-028/029 |
| GWT-ADJ-10 | Dado el apartado nuevo, el embudo incorpora `step=adjectives`; corregir conserva el mismo embudo y la telemetría excluye la selección. | RP-ANA-014, RP-ADJ-003 |
| GWT-ADJ-11 | Dado un estilo vacío, elegir asigna, otra opción sustituye y repetir retira. Omitir conserva, un ID válido sustituye y `null` retira; un valor inválido rechaza toda la edición. Se conserva independientemente de las cualidades en lectura, guardado, relanzamiento, corrección, cuenta y baja. | RP-ADJ-003/006/007/008/010 |
| GWT-ADJ-12 | Dado el selector, «Ver significados» despliega definiciones sin modificar selecciones y «Ocultar significados» las oculta. Guardar y reabrir conserva estilo y cualidades agrupadas en revisión y ficha. | RP-ADJ-004/006/007/010 |
| GWT-ADJ-13 | Dada una selección vacía, persistencia y lectura conservan `[]`. Un dato persistido inválido produce error técnico y conserva su contenido; respuesta y diagnóstico excluyen selecciones con debug activo o no. Una edición válida confirma identificador y ambas selecciones. | RP-ADJ-003/005/006, RP-SEG-006/007 |

### Lectura, búsqueda y edición del perfil

- **RP-CRU-019.** Después de iniciar sesión, ninguna lista, búsqueda, mapa, Timeline ni ficha privada de la aplicación queda visible mientras el Vault esté bloqueado. Al iniciar la aplicación y cada vez que regresa desde segundo plano, MDL bloquea el Vault y cubre el contenido privado hasta que la persona supera el reconocimiento biométrico. Esta regla no se aplica a las pantallas públicas o previas al inicio de sesión.
- **RP-CRU-020.** La ficha permite consultar los datos informados de su inventario, combinando estructura funcional y contenido privado autorizado. Los datos administrados por el sistema mantienen su función de gestión y no se presentan como datos del perfil. Cada rasgo conserva el significado de su opción de catálogo.
- **RP-CRU-021.** La búsqueda privada encuentra coincidencias parciales, sin distinguir mayúsculas ni acentos, exclusivamente por el nombre del Crush o por el nombre visible de su país. El término y la comparación del nombre se mantienen en el dispositivo con el Vault desbloqueado; la API sólo recibe la paginación y los filtros estructurales ya aplicados y nunca recibe ni persiste el texto buscado.
- **RP-CRU-022.** Desde la ficha pueden editarse nombre, género, país, Ducky, fecha de encuentro, edad confirmada, profesión, WhatsApp, usuarios de redes, cumpleaños, cada valoración, cada rasgo físico, la selección de adjetivos y el estilo social. Nombre, género, país, Ducky, fecha de encuentro y edad son obligatorios y no pueden borrarse. Una confirmación de edad transmite la pareja definida en RP-AGE-002/011/015; editar sin actualizar esa pareja conserva sus valores. Los campos opcionales escalares pueden añadirse, sustituirse o quitarse; quitar uno envía un borrado explícito y lo conserva como `null`, mientras que omitir un campo sin cambios conserva su valor actual. La selección de adjetivos se vacía con `[]` conforme a RP-ADJ-002. El signo zodiacal y la valoración global siguen siendo derivados y nunca se editan directamente.

<a id="listas"></a>
### Listas de Crushes activos y archivados

- **RP-LIS-001.** La lista de activos y la pantalla propia de archivados ofrecen la misma búsqueda, los mismos filtros y las mismas opciones de orden. Sólo cambian el conjunto consultado y las acciones de ciclo de vida aplicables: en archivados la acción disponible es restaurar y nunca eliminar directamente; eliminar sólo se ofrece en la fila activa conforme a RP-DEL-006. La pantalla de archivados forma parte de la beta.
- **RP-LIS-002.** El conjunto cerrado de capacidades de consulta de ambas pantallas es: búsqueda por nombre o país conforme a RP-CRU-021; orden por actividad reciente o por fecha de encuentro, siempre de más reciente a más antigua; filtro por resultado de match (`UNKNOWN`, `MATCH`, `NO_MATCH`); filtro por la situación vigente exacta del período más reciente, usando los diez valores de `RelationshipSituation`; filtro por género (`1 man`, `2 woman`, `3 trans`); y filtro por Ducky con los cuatro valores de RP-CRU-005. Cada categoría ofrece «Todos», que elimina sólo esa restricción y no representa el ID `0 UNKNOWN` de match.
- **RP-LIS-003.** En cada categoría se elige como máximo una opción y puede haber simultáneamente una selección en categorías distintas; las categorías seleccionadas y el texto buscado se combinan por intersección. «Limpiar filtros» restablece el orden y todas las categorías. La aplicación calcula el resultado sobre el conjunto completo que cumple las restricciones, no sólo sobre la página ya cargada, y usa el vacío de búsqueda o filtros definido en RP-VAC-004 cuando no hay coincidencias.
- **RP-LIS-004.** La ubicación técnica de un dato no decide si su filtro existe. Género y nombre se resuelven exclusivamente en el dispositivo a partir del Vault desbloqueado; el término de búsqueda tampoco se envía a la API. Match, situación vigente, Ducky y orden se resuelven contra la estructura funcional y pueden delegarse a la API. En ningún caso un cliente envía `gender_id`, el nombre privado ni el texto buscado a la API funcional, ni elimina una opción visible porque ese dato haya dejado de persistirse allí.

### Fecha de encuentro

- **RP-CRU-007.** La fecha de encuentro siempre registra un hecho ya ocurrido: nunca puede ser futura. Puede moverse desde el perfil del Crush hacia una fecha anterior o posterior a la actualmente guardada y no tiene un límite histórico mínimo. Si el Crush tiene alguna experiencia posterior al bundle inicial —con o sin cambio relacional—, la nueva fecha debe ser estrictamente anterior a la primera.
- **RP-CRU-008.** El calendario aplica como límite superior el menor entre hoy y el día anterior a la experiencia posterior más antigua del mismo Crush. Si no existe ninguna, la última fecha seleccionable es hoy. Para encontrarla tiene en cuenta experiencias confirmadas o pendientes. El período inicial, su hito inicial y la experiencia inicial no limitan la selección porque se desplazan juntos con `met_at`. Las experiencias de otros Crushes tampoco la limitan.
- **RP-CRU-009.** Corregir la fecha de encuentro cambia conjuntamente la fecha del período relacional inicial, de su hito inicial y de la experiencia inicial. Siguen siendo los mismos elementos y conservan sus asociaciones. Todas las experiencias posteriores y sus eventuales hitos vinculados conservan su fecha y sus asociaciones actuales.

**Ejemplo.** Si `met_at` es el 1 de septiembre y el primer acontecimiento independiente ocurre el 5 de septiembre, puede cambiarse al 4 de septiembre o a cualquier fecha anterior, incluso de años anteriores. No puede cambiarse al 5 de septiembre ni a una fecha posterior.

### Valoraciones

- **BR-4.02.** Atracción física, conexión emocional, forma de ser y atracción sexual son valoraciones opcionales que sólo admiten valores enteros: 1, 2, 3, 4 o 5; la API rechaza cualquier otro valor. Se conservan como enteros y, después del alta, cada una puede añadirse, sustituirse o quitarse desde el perfil del Crush. Quitarla guarda `null` para esa valoración. La valoración global es la única que admite decimales: MDL la recalcula en cada alta, cambio o eliminación de una valoración como la media aritmética de las valoraciones informadas, redondeada a dos decimales. La persona no puede introducir ni modificar el global. Si no queda ninguna valoración informada, el global también es `null` y no se muestra.

La revisión de nombres aprobada expresa la valoración de quien utiliza el diario: cuánto le gusta físicamente esa persona, cuánta cercanía emocional siente, cuánto le gusta su forma de ser y cuánto deseo sexual le despierta. No es una evaluación objetiva de la persona ni una señal de reciprocidad. Se conservan `rate_beauty`, `rate_emotional_connection`, `rate_personality` y `rate_sex_appeal`, sus valores, obligatoriedad y cálculos. Las etiquetas inglesas son «Physical attraction», «Emotional connection», «Personality» y «Sexual attraction», respectivamente. Formularios, ficha, revisión, avisos de corrección, estadísticas y demo utilizan los mismos nombres por idioma.

<a id="edad"></a>
### Edad, cumpleaños y zodíaco

#### Datos de referencia

- **RP-AGE-001.** La edad que se confirma en el alta o en una edición debe ser un entero entre 18 y 80 años inclusive. La edad mostrada puede superar 80 posteriormente por el transcurso del tiempo; ese límite sólo valida una nueva confirmación introducida por la persona.
- **RP-AGE-002.** La edad confirmada indica cuántos años tenía el Crush en la fecha de confirmación. Al confirmar el alta o una edición que actualiza la pareja de edad, la app fija `age_as_of` con el día civil local de esa confirmación, en formato `yyyy-MM-dd`, y conserva `age` y `age_as_of` juntos en el snapshot durable antes del envío. La API exige y persiste ambos datos juntos; no genera ni sustituye la fecha por el día de recepción. La fecha debe ser válida y no futura respecto del día civil local del intento, comprobado mediante el contexto de RP-ID-006; ese contexto se recalcula al enviar y no cambia la fecha conservada.
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
- **RP-AGE-011.** Cuando la persona corrige la edad, declara la edad actual del Crush en el momento de confirmarla en la app. La app fija su fecha conforme a RP-AGE-002, sin ofrecer un selector separado de `age_as_of`. La API acepta esa fecha junto con la edad; rechaza una pareja incompleta, nula, mal formada o futura, sin completar datos mediante el reloj del servidor.
- **RP-AGE-012.** Añadir o cambiar el cumpleaños conserva como referencia la edad vigente calculada con los datos anteriores y renueva su fecha de confirmación. La nueva fecha de cumpleaños se utiliza para las actualizaciones posteriores de edad.
- **RP-AGE-013.** Quitar el cumpleaños conserva la edad vigente, renueva su fecha de confirmación y elimina el día y mes del cumpleaños y su signo zodiacal.
- **RP-AGE-014.** Si se corrigen simultáneamente edad y cumpleaños, prevalece la edad introducida expresamente.
- **RP-AGE-015.** La confirmación del cambio en la app fija la pareja de edad y fecha en la operación durable. Cada intento transmite exactamente esa misma pareja, aunque no hubiera conexión al confirmar, se perdiera la respuesta, se reiniciara la app, cambiara la zona horaria o el reintento ocurriera otro día. La API guarda la fecha recibida sin renovarla. Una nueva confirmación que actualice la pareja fija su propia fecha; una edición ajena a la edad conserva la pareja existente, salvo los cambios de cumpleaños definidos en RP-AGE-012/013. Este contrato no exige un registro servidor de aceptaciones de ediciones ni añade una garantía de ordenación de ediciones antiguas respecto de ediciones posteriores.

<a id="fotos"></a>
### Fotos del Crush

- **RP-FOT-001.** En la beta, las fotos del Crush son privadas y se guardan únicamente cifradas en el dispositivo. MDL no las transmite a la API, no las incorpora al inventario del Vault remoto ni ofrece una copia de seguridad remota de ellas.
- **RP-FOT-002.** Antes de guardar una foto, MDL elimina sus metadatos de cámara y ubicación. La foto queda asociada a la cuenta y al Crush correspondientes.
- **RP-FOT-003.** Las fotos y cualquier copia auxiliar persistente se cifran en el dispositivo con una clave local dedicada derivada de la VMK, se protegen además mediante los controles del dispositivo y se excluyen de las copias de seguridad del sistema. Que compartan la raíz local y la autoridad de desbloqueo del Vault no las convierte en entries ni blobs del Vault remoto.
- **RP-FOT-004.** Eliminar una foto retira de MDL la imagen guardada, sus miniaturas y sus demás copias auxiliares en el dispositivo.
- **RP-FOT-005.** Eliminar un Crush elimina también sus fotos guardadas por MDL. Al confirmar la baja de la cuenta, se eliminan las fotos de esa cuenta a las que la aplicación tiene acceso en el dispositivo.
- **RP-FOT-006.** Recuperar el Vault no recupera las fotos del Crush. MDL no garantiza el borrado remoto de fotos en un dispositivo sin conexión.
- **RP-FOT-007.** Cada Crush puede tener como máximo cinco fotos guardadas. Eliminar una foto libera inmediatamente ese lugar para añadir otra.
- **RP-FOT-008.** Cada foto saneada puede ocupar como máximo 10 MB antes de cifrarse; el pequeño overhead criptográfico local no cuenta como tamaño aportado por la persona. MDL rechaza una foto que supere ese tamaño y no altera las fotos ya guardadas. Si una selección mezcla fotos válidas e inválidas, guarda las válidas que caben en la cuota e informa de forma comprensible que las demás fueron rechazadas; si ninguna es válida, no guarda nada ni modifica las existentes y muestra el error.
- **RP-FOT-009.** El Crush sólo admite fotografías. MDL no ofrece ni admite subir videos como contenido del Crush. Un archivo o formato no admitido se rechaza con un mensaje comprensible, bajo el mismo comportamiento parcial y no destructivo de `RP-FOT-008`.
- **RP-FOT-010.** Cada fotografía y sus copias auxiliares se cifran y autentican con una clave dedicada derivada de la clave principal del diario, ligada a su cuenta, Crush e identidad de fotografía. El contrato criptográfico se mantiene en la documentación técnica de API y clientes.
- **RP-FOT-011.** La aplicación nunca persiste la fotografía saneada, una miniatura ni un temporal en claro. Considera guardada una foto sólo después de escribir atómicamente y poder autenticar su contenedor cifrado; una interrupción o fallo no sustituye una foto válida ni deja un archivo legible residual.
- **RP-FOT-012.** Una foto sólo se descifra temporalmente en memoria mientras el mismo Vault y su época de desbloqueo siguen vigentes. Bloquear el Vault, cerrar sesión, cambiar de cuenta o retirar ese contexto elimina las imágenes descifradas de la presentación. Un archivo ausente, alterado o no autenticable no se muestra, no se intenta como imagen en claro y produce un error comprensible sin afectar las demás fotos.

<a id="escenarios-fotos"></a>
#### Escenarios de aceptación de fotos

Estos escenarios concretan las reglas anteriores; la cobertura, los resultados y los límites físicos aceptados de la implementación iOS están en [Testing de iOS](../../../ios/docs/TESTING.md#fotos-locales-cifradas). Un escenario describe el comportamiento exigido; su presencia aquí no acredita una comprobación física no realizada.

| ID | Dado / Cuando / Entonces | Referencia |
|---|---|---|
| GWT-LPE-01 | Dada una foto admitida y una sesión desbloqueada vigente, cuando se guarda, entonces se eliminan metadatos y sólo se confirma tras persistir atómicamente y autenticar el contenedor; no queda copia auxiliar en claro. | RP-FOT-001..003/010/011 |
| GWT-LPE-02 | Dados VMK, namespace e identidades conocidos, cuando se deriva/cifra/descifra, entonces se reproducen los vectores de derivación y el round-trip; otra VMK, namespace, Crush o foto no autentican el contenedor. Dos guardados iguales usan nonces nuevos. | RP-FOT-010, RP-ID-007 |
| GWT-LPE-03 | Dado un contenedor ausente, truncado, alterado o de versión desconocida, cuando se carga, entonces no se muestra ni se lee como imagen en claro; se informa un error comprensible y las otras fotos válidas siguen disponibles. | RP-FOT-012, RP-SEG-003 |
| GWT-LPE-04 | Dada una escritura o sustitución local interrumpida, cuando falla antes de confirmar, entonces no se publica éxito, se conserva la foto válida anterior y no quedan temporales en claro ni un contenedor parcial visible como guardado. | RP-FOT-011 |
| GWT-LPE-05 | Dado un Crush con cupo disponible, cuando recibe fotos saneadas en el límite de 10 MB, por debajo o por encima y formatos no admitidos, entonces guarda sólo las válidas que caben hasta cinco, sin contar overhead; informa rechazos/omisiones y preserva lo existente. Si ninguna es válida no guarda nada. | RP-FOT-007..009 |
| GWT-LPE-06 | Dada una foto guardada, cuando se elimina individualmente, entonces desaparecen contenedor, auxiliares, referencias e imagen en memoria; el lugar queda disponible sin afectar las otras fotos. | RP-FOT-004/007 |
| GWT-LPE-07 | Dado un borrado de Crush o baja de cuenta, cuando se confirma su resultado funcional, entonces se retiran las fotos del alcance correcto; una interrupción permite reanudar sin afectar otra cuenta y una respuesta ambigua no anticipa la eliminación física. | RP-FOT-005, RP-DEL-003..005, RP-BAJ-007..012 |
| GWT-LPE-08 | Dada una carga o un guardado en vuelo, cuando se bloquea, sale de sesión, confirma baja o cambia la cuenta, entonces se invalidan sus resultados pendientes y se retiran imágenes de galería, preview y visor; ninguna respuesta tardía publica contenido o error en el contexto posterior. | RP-FOT-012, RP-LOCK-003, RP-ID-007 |
| GWT-LPE-09 | Dado un bloqueo seguido de desbloqueo del mismo Vault, cuando finaliza un trabajo de la época anterior, entonces no publica ni conserva su resultado descifrado; sólo una operación de la época nueva puede presentar fotos. | RP-FOT-012, RP-LOCK-003 |
| GWT-LPE-10 | Dadas fotos locales y un inventario remoto, cuando se guarda, consulta, sincroniza o recupera el Vault en un dispositivo sin esas fotos, entonces no hay transferencia de media, alta de entries/bindings/blobs ni consumo de cuota por ellas, y recovery no restaura fotos. | RP-FOT-001/006, RP-VLT-026/027 |
| GWT-LPE-11 | Dados contenedores y auxiliares locales, cuando se realiza un backup real del sistema, entonces quedan excluidos; los logs/errores/analítica no contienen imágenes, material criptográfico ni rutas privadas. | RP-FOT-003, RP-SEG-006/007 |
| GWT-LPE-12 | Dadas cinco fotos admitidas y un Vault desbloqueado, cuando se cargan y se recorre la galería/visor y el ciclo de bloqueo, entonces I/O, cripto y decodificación ocurren fuera del principal, la interacción responde y no existe decodificación simultánea no acotada. Las mediciones identifican dispositivo, tamaños y límites de la muestra. | RP-FOT-012 |

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
- **RP-ARC-003.** Mientras la operación durable de alta del Crush siga pendiente conforme a RP-DUR-007, no puede archivarse ni se le pueden añadir experiencias o cambios relacionales. Que la API ya haya aceptado y devuelva el Crush no basta para habilitar esas acciones: el alta se confirma únicamente cuando termina también la parte cifrada del Vault y deja de existir el cambio pendiente.

## Eliminar un Crush

- **RP-DEL-001.** Eliminar un Crush exige confirmación y retira del diario el Crush, toda su historia —experiencias, períodos e hitos—, sus asociaciones, actividad, contenido del Vault y fotos vinculadas.
- **RP-DEL-002.** La eliminación individual de un Crush elimina esa ficha y todos sus datos vinculados. La [baja completa de la cuenta](DATA_LIFECYCLE.md#baja) elimina además el resto de su diario y sus copias recuperables identificables.
- **RP-DEL-003.** Antes de pedir la eliminación funcional al servidor, la aplicación conserva durablemente la intención de borrar. No retira todavía su copia local. Si la petición falla o la respuesta se pierde, mantiene la operación pendiente y el Crush local hasta poder reanudarla y confirmar el resultado.
- **RP-DEL-004.** Reanudar o repetir una eliminación es idempotente: si el servidor ya borró el Crush durante un intento cuya respuesta se perdió, el reintento se considera satisfactorio y no vuelve a crear ni necesita recuperar sus asociaciones. Una respuesta de «no encontrado» sólo puede confirmar el borrado cuando existe un intento anterior registrado durablemente; en el primer intento se trata como error.
- **RP-DEL-005.** Sólo después de confirmar el borrado funcional, la aplicación retira el contenido local del Vault, las fotos y el resto de copias vinculadas. Entonces el Crush y sus datos dejan de estar disponibles. La destrucción definitiva del contenido cifrado en infraestructura sigue los plazos de [Tareas automáticas y plazos](DATA_LIFECYCLE.md#tareas).
- **RP-DEL-006.** La acción para eliminar el Crush completo se ofrece desde su fila en la lista de Crushes activos. No aparece dentro del bloque inicial, la ficha ni las acciones de su experiencia, período o hito. Un Crush archivado debe restaurarse antes de poder eliminarse.
