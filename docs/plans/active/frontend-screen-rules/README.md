# Reglas funcionales por pantalla

> **Estado:** activa. El inventario funcional se realiza primero sobre iOS como aplicación de referencia de la beta activa y después se contrasta en Android para una distribución futura, sin bloquear el release iOS/API. La revisión y aceptación de la pantalla de archivados quedan deliberadamente para el último bloque. Android ya conserva una implementación adelantada que no se modifica en los cortes actuales; su contrato completo de consulta, cerrado en `RP-LIS-001..004`, se verifica y completa en `FSR-07` después del resto de superficies.
>
> **Proyectos:** Product / iOS / Android. API participa únicamente cuando una regla visible necesita soporte estructural o de contrato HTTP.

## Objetivo

Definir, pantalla por pantalla, el comportamiento observable que iOS y Android deben compartir, sin confundir una decisión de almacenamiento o de API con la presencia de un campo, filtro o acción en la interfaz.

iOS es la referencia inicial para inventariar pantallas, recorridos, estados y opciones porque contiene el producto más completo. No es una segunda fuente de verdad: si su comportamiento contradice Product, se registra como desviación y no se copia. Una vez cerrado cada contrato funcional, Android debe producir el mismo resultado y ofrecer las mismas capacidades con componentes, navegación, permisos y convenciones nativas cuando la plataforma lo requiera. Una diferencia de plataforma no puede retirar funcionalidad ni alterar una regla sin una excepción explícita de Producto.

El resultado estable se integra en el catálogo canónico: las reglas propias de un dominio permanecen en su documento de `docs/rules/` y las garantías transversales de interfaz permanecen en `UX.md`. Este plan coordina el inventario, la revisión y la implementación; no crea un segundo catálogo ni copia reglas ya definidas.

## Alcance

Incluye todas las superficies de la beta:

- registro, acceso, verificación, recuperación, activación y desbloqueo del Vault;
- navegación principal, Home, lista de Crushes activos, búsqueda, filtros, mapa y estados vacíos;
- alta, ficha y edición del Crush, fotos y eliminación;
- alta, lectura, edición y eliminación de Experiences;
- situación relacional, períodos, hitos, cierres y acciones disponibles;
- Timeline, Statistics, perfil de cuenta, Settings, idioma, seguridad, baja y superficies legales;
- estados de carga, error, offline, permiso, bloqueo, dato pendiente, reintento y conflicto que cambien lo que la persona puede ver o hacer.
- como último bloque, archivo, pantalla de Crushes archivados y restauración.

Fuera de alcance:

- rediseñar la identidad visual o fijar medidas puramente gráficas sin efecto funcional;
- inventar funciones fuera de la beta;
- redefinir desde una pantalla una regla ya cerrada en Product;
- modificar código, tests o contratos desde esta tarea coordinadora;
- modificar Legal o resolver una decisión jurídica abierta.

## Regla de documentación

Cada pantalla se describe mediante el mismo contrato mínimo:

1. propósito y puntos de entrada;
2. condición de acceso, incluido sesión, Vault, permisos y datos previos;
3. datos visibles, editables, obligatorios, opcionales y derivados;
4. acciones disponibles, acciones prohibidas y confirmaciones;
5. validaciones y mensajes que permiten corregir el problema;
6. fuente de lectura y destino de escritura de cada dato, distinguiendo API, Vault y almacenamiento local;
7. estados de carga, vacío, error, offline, pendiente, conflicto y bloqueo;
8. navegación, resultado al volver y estado que debe conservarse;
9. reglas canónicas que justifican cada comportamiento;
10. evidencia automatizada y validación manual exigidas por plataforma.

Una decisión sobre persistencia no añade ni retira por sí sola un elemento de interfaz. Cualquier cambio visible exige su propia decisión de Producto. iOS y Android aplican el mismo contrato funcional; una diferencia necesita una excepción explícita conforme a `RP-BASE-008`. La comparación visual y de interacción busca la máxima semejanza razonable, pero la paridad se decide por capacidades, reglas, datos, acciones y resultados, no por reproducir literalmente un componente propio de la otra plataforma.

`RP-UX-008` se aplica a todas las pantallas: una función activa u obligatoria no desaparece ni concede acceso porque un endpoint devuelva `404`, falte un flag o el cliente suponga un backend anterior. Una puerta de acceso o seguridad no resuelta falla cerrada; una acción ordinaria conserva el estado confirmado y muestra error con reintento. Esto no se confunde con una ausencia de recurso definida por el dominio —por ejemplo, que una cuenta todavía no tenga Vault— ni con una capacidad física real del dispositivo.

## Inventario base de iOS

El grafo vigente de iOS identifica las siguientes superficies funcionales. La tabla agrupa pantallas y pasos que pertenecen a un mismo recorrido; los componentes internos no se convierten en pantallas de Producto independientes.

| Bloque | Pantallas y recorridos de referencia en iOS | Autoridad funcional que debe contrastarse | Estado del inventario |
|---|---|---|---|
| Raíz pública | Bienvenida de invitado con «Crear cuenta» e «Iniciar sesión». | Cuenta, UX y lanzamiento por plataforma. | Localizada. |
| Registro | Elección email/Apple; email; fecha de nacimiento; contraseña; términos y privacidad; continuación Apple cuando corresponde. | `ACCOUNT.md`, Legal y reglas de formulario. | Localizado; contrato por pantalla pendiente. |
| Inicio de sesión | Email/contraseña, Apple, acceso a recuperación y errores de credenciales/sesión. | `ACCOUNT.md`, Vault y reglas de formulario. | Localizado; contrato por pantalla pendiente. |
| Recuperación de contraseña | Email, código OTP y contraseña nueva, con reenvío, cooldown, vencimiento y resultado. | `ACCOUNT.md#recuperacion-acceso` y UX. | Localizado; contrato por pantalla pendiente. |
| Verificación y cambio de email | Gate obligatorio, ingreso de código, reenvío y cambio de email desde la cuenta verificada. | `ACCOUNT.md#verificacion-email`, cambio de email y sesión provisional. | Localizado; contrato por pantalla pendiente. |
| Bootstrap privado | Onboarding obligatorio del Vault, confirmación de recuperación, recovery en dispositivo nuevo, espera de desbloqueo y App Lock biométrico. | `VAULT.md`, Cuenta, Sync y Privacidad. | Localizado; contrato por pantalla pendiente. |
| Tutorial inicial | Cinco pasos informativos abiertos desde Home hasta confirmar su finalización. | `RP-ONB-001..004`. | Localizado; revisar copy, entrada, salida y retry. |
| Explicación del primer Crush | Tres pasos informativos abiertos desde la ficha del primer Crush. | `RP-ONB-001..004`. | Localizado; revisar condición exacta de primera vez. |
| Navegación autenticada | Cuatro destinos principales: Home, Crushes, Timeline y Statistics; acción central para alta de Crush. | `RP-BASE-008`, UX y reglas de cada dominio. | Localizada; contrato de navegación pendiente. |
| Home | Actividad reciente, acceso al Dashboard de Statistics, acceso a Settings y aceptación legal obligatoria cuando corresponda. Home no presenta un segundo resumen estadístico ni teasers numéricos. | `TIMELINE.md`, `STATISTICS.md`, Cuenta, Legal y UX. | Localizado; contrato por estados pendiente. |
| Lista activa y mapa | Búsqueda, filtros, orden, altas pendientes, acciones por fila, acceso al mapa y apertura de la ficha. | `CRUSH.md#listas`, mapa, Sync y UX. | Primer contrato cerrado en `RP-LIS-001..004`; resto en revisión. |
| Alta de Crush | Datos obligatorios, situación inicial y Experience inicial dentro de una única acción compuesta. | `CRUSH.md#alta`, Relationships, Experiences, Vault y Sync. | Localizada; contrato de los pasos pendiente. |
| Ficha y edición del Crush | Cabecera, datos privados/estructurales, fotos, Timeline del Crush, edición lineal y revisión final. | `CRUSH.md#perfil`, fotos, edad, Relationships, Experiences y Sync. | Localizado; contrato por secciones/acciones pendiente. |
| Experiences y relaciones | Añadir Experience, cambio opcional u obligatorio de situación, editar Experience y corregir/eliminar el último hito permitido. | `EXPERIENCES.md`, `RELATIONSHIPS.md`, Timeline, Vault y Sync. | Modelo funcional final integrado; pendiente sólo de su inventario transversal por pantalla. |
| Timeline | Timeline global y Timeline de un Crush; filtros, pendientes, detalle, edición, corrección y borrados permitidos. | `TIMELINE.md`, Experiences, Relationships y Sync. | Localizado; contrato por capacidades pendiente. |
| Statistics | Dashboard y detalles de momento, satisfacción, atracción, rasgos físicos, zodíaco, países y Duckys. | `STATISTICS.md` y estados UX. | Localizado; contrato de navegación/estados pendiente. |
| Settings y cuenta | Perfil, idioma, email/verificación, contraseña o primera contraseña, vínculo Apple, biometría, Vault, FAQ, datos cifrados, términos, privacidad, aviso legal, baja y cierre de sesión. | Cuenta, Vault, Privacidad, ciclo de vida, Legal y UX. | Localizado; distinguir filas activas de código no expuesto. |
| Archivados | Acceso, lista, búsqueda/filtros/orden, ficha y restauración. | `RP-ARC-001..003` y `RP-LIS-001..004`. | Inventariada sólo como obligación; diferida íntegramente a `FSR-07`. |

### Superficies detectadas que no forman parte de la beta activa

El código de iOS conserva recorridos sin entrada activa para transferencia Vault por QR y galería/sincronización remota de medios, además de filas comentadas de comunidad y notificaciones. Su mera presencia no los convierte en funcionalidad ni en referencia para Android. Se mantienen fuera del contrato salvo una decisión posterior de Producto.

## Baseline funcional `FSR-01` — acceso, Vault y onboarding

Este bloque toma la secuencia observable de iOS y la somete a las reglas ya cerradas de Cuenta, Vault y UX. Android debe conservar las mismas puertas, opciones y resultados; sólo cambia el componente nativo usado para presentarlos.

### Secuencia común

1. Sin sesión se muestra la bienvenida pública, con las entradas «Crear cuenta» e «Iniciar sesión».
2. El alta por email conserva una capacidad opaca de registro separada de la sesión; login sólo puede entregar una sesión de cuenta verificada conforme a Cuenta.
3. Una provisional queda en el gate de verificación de su propia capacidad y no accede a perfil, Vault, vinculación federada ni contenido privado.
4. Tras verificar el email, el estado real del Vault decide entre configuración inicial, recuperación, desbloqueo o entrada a la aplicación.
5. Sólo con el Vault disponible y desbloqueado queda visible y utilizable la navegación privada con Home, Crushes, Timeline y Statistics.
6. El tutorial inicial informativo se presenta dentro de la aplicación privada cuando continúa pendiente; no sustituye ni adelanta el onboarding obligatorio del Vault.

### Contratos por pantalla

| Pantalla/flujo | Comportamiento funcional común |
|---|---|
| Bienvenida | No muestra datos privados ni destinos autenticados. «Crear cuenta» abre la elección de alta; «Iniciar sesión» abre las opciones de acceso. Volver o cancelar no crea cuenta ni sesión. |
| Elección de alta en iOS | Ofrece alta por email y «Continuar con Apple». Android conserva el mismo resultado funcional, pero ofrece «Continuar con Google» y no Apple conforme a `RP-LAN-004`; esta es una excepción de proveedor ya decidida, no una diferencia del resto del alta. |
| Alta por email | Flujo lineal: email, fecha de nacimiento, contraseña y aceptación separada de términos y privacidad. No solicita género, país, nombre ni datos del diario. Volver conserva lo introducido durante el intento; intentar confirmar valida el formulario completo y sólo un resultado conforme a Cuenta y Legal completa el alta. |
| Alta con proveedor | El proveedor autentica la identidad, pero no omite la edad mínima ni la aceptación legal. En iOS, un alta Apple que todavía necesita cuenta solicita fecha de nacimiento y aceptación legal, sin pedir otra vez email ni contraseña. Vincular una cuenta existente es un recorrido explícito y nunca crea o fusiona silenciosamente otra cuenta. Android aplica el equivalente Google definido en Cuenta. |
| Inicio de sesión | Ofrece email/contraseña y el proveedor permitido por plataforma. También permite ir a registro y a recuperación de contraseña. Un éxito no salta los gates pendientes de verificación o Vault; un fallo conserva el formulario y muestra un error de producto. |
| Recuperación de acceso | Tres pasos: email, OTP y contraseña nueva. Reenvío, cooldown, vencimiento, intentos y respuesta anti-enumeración obedecen Cuenta. Completar revoca las sesiones definidas por la regla y vuelve al acceso sin tocar el Vault. |
| Verificación de email | Es un gate global para una cuenta email todavía no verificada. Muestra estado, entrada de código, reenvío y errores recuperables. Hasta completarlo no se presenta Home, el contenido privado ni el onboarding del Vault. Verificar reanuda el mismo recorrido en el estado de Vault que corresponda. |
| Configuración inicial del Vault | Es obligatoria antes de usar el diario privado. Explica la protección, verifica la credencial admitida, crea la envoltura, presenta la frase de recuperación y exige su confirmación antes de finalizar. Cerrar la app o fallar no finge finalización; al volver se reanuda o reinicia según el estado durable canónico. |
| Recuperación del Vault | Permite los métodos autorizados por `VAULT.md` y nunca muestra Home durante el proceso. Un error conserva una salida segura y no sustituye ni destruye el Vault remoto válido. El éxito reconstruye el mismo Vault y continúa al desbloqueo o aplicación según corresponda. |
| Desbloqueo y App Lock | El contenido privado sólo aparece después de desbloquear. Volver de segundo plano cubre el contenido y exige reconocimiento biométrico, sin cerrar sesión, destruir navegación ni perder formularios confirmados. Cancelar o fallar mantiene el contenido cubierto. |
| Tutorial inicial | Cinco pasos exclusivamente informativos. Permite avanzar y volver, no solicita datos, se marca terminado sólo al confirmar el último paso y permanece pendiente si esa confirmación falla. |
| Explicación del primer Crush | Tres pasos exclusivamente informativos asociados a la primera ficha. No edita el Crush ni la cuenta; sigue `RP-ONB-001..004` y no vuelve a aparecer después de una finalización confirmada. |

### Contrato vigente — disponibilidad y fallo cerrado

`IOS-EMAIL-09` (`GF-001`) e `IOS-BRI-01A` (`GF-002`) resolvieron el acceso permisivo que este bloque auditaba: `featureNotAvailable`/`isFeatureAvailable` no existen en el árbol iOS. Un `404`, `*.not_found`, un estado desconocido o un fallo de transporte durante alta, login o restauración nunca concede acceso a perfil, Vault ni acciones protegidas, conforme a `RP-CUE-003`, `BR-3.12`, `BR-3.14`, `RP-VER-003` y `RP-UX-008`. `LoginView` entrega el control exclusivamente a la raíz autenticada, que resuelve el destino Vault antes de montar navegación privada. `SensitiveActionGateCoordinator` exige el estado verificado antes de ejecutar cualquier acción protegida (activación manual de Vault, baja de cuenta y demás acciones sensibles). El cambio de email y la primera contraseña de cuenta Apple conservan el estado del formulario y ofrecen error/reintento ante un `404`, conforme a `RP-UX-002`, `RP-UX-007`, `RP-UX-008`, `RP-MAIL-001..005`, `RP-APL-015..016` y `RP-APL-032`.

La búsqueda exhaustiva acotada a código Swift de iOS no encontró otra bandera de producción con el mismo nombre o efecto permisivo. Los `404` de lectura de Vault representan la ausencia canónica del recurso; los tolerados en borrados sólo se aceptan al reanudar una intención durable ya enviada, mientras un primer `404` sigue siendo rechazo. `Statistics.isAvailable` describe disponibilidad de datos de una métrica y `BiometricError.notAvailable` una capacidad física del dispositivo; ninguno de esos casos autoriza acceso ni sustituye una función activa.

### Estado del bloque

La secuencia y las capacidades anteriores no inventan una variante Android: son el contrato funcional común. El acceso y fallo cerrado de `FSR-01` ya reflejan el contrato aceptado por `IOS-EMAIL-09` (validación física pendiente, ver plan iOS local) e `IOS-BRI-01A`; continúa el contraste de los demás estados de error/carga, copy, accesibilidad y tests.

## Baseline funcional `FSR-02` — navegación, Home, lista activa y mapa

| Pantalla/flujo | Comportamiento funcional común |
|---|---|
| Navegación privada | Con sesión válida y Vault desbloqueado ofrece Home, Crushes, Timeline y Statistics, más la acción principal de alta de Crush. Cambiar de destino no altera datos ni convierte una recarga en una escritura. Bloquear la app cubre el contenido sin destruir el destino ni la navegación ya montados. |
| Home | Presenta actividad reciente y una tarjeta neutra de acceso al Dashboard de Statistics. Esa tarjeta no calcula, resume ni simula métricas: nunca muestra cifras, donuts, comparaciones o explicaciones personales; el Dashboard es la única superficie que presenta estadísticas reales y resuelve sus estados de carga, error, vacío y contenido. Cada bloque permite ir a su destino completo y una actualización fallida conserva el último contenido confirmado. |
| Acciones globales de Home | Permite abrir Settings. Cuando existe una aceptación legal obligatoria pendiente, la presenta antes de permitir continuar con el uso afectado y registra únicamente una aceptación válida. Notificaciones, comunidad y galería Vault no aparecen mientras sigan fuera de la beta activa. |
| Lista de Crushes activos | Presenta Crushes activos confirmados y proyecciones pendientes identificables, permite abrir la ficha y aplica `RP-LIS-001..004`. La fila ofrece sólo las acciones autorizadas para un Crush activo; eliminar se rige por `RP-DEL-001..006`. Archivados no se diseña en este bloque. |
| Búsqueda, filtros y orden | Conserva todas las categorías cerradas en `RP-LIS-002`, permite combinarlas y calcula el resultado sobre el conjunto completo. La ubicación API/Vault de un dato no cambia la opción visible. Error y ausencia de coincidencias no se confunden. |
| Mapa | Sólo representa Crushes activos agrupados por país, nunca una ubicación precisa. Un país sin activos no muestra marcador; sólo existe un popup abierto; volver conserva lista y filtros conforme a `RP-MAP-001..005`. |
| Alta global de Crush | La acción principal abre el mismo flujo desde cualquier destino donde la navegación principal esté disponible. Cerrar antes de guardar no crea un Crush; una intención ya guardada se rige por Sync y aparece como pendiente, no como éxito remoto ficticio. |

## Baseline funcional `FSR-03` — alta, ficha y edición del Crush

| Pantalla/flujo | Comportamiento funcional común |
|---|---|
| Alta — datos del Crush | Primer bloque con nombre, género, país, Ducky, fecha de encuentro y edad confirmada. Todos son obligatorios. Nombre y género permanecen en Vault; la pantalla no elimina esos campos porque la API funcional no los interprete. |
| Alta — situación inicial | Permite elegir exactamente una situación inicial `RelationshipSituation 1...9`; nunca `CLOSED`, una fase o una etiqueta legacy. No deriva la situación desde Ducky, texto, satisfacción ni IA. |
| Alta — Experience inicial | Lugar, descripción y satisfacción son obligatorios y comparten `met_at` con el bloque inicial. La confirmación final crea una única intención compuesta con Crush, período, hito y Experience, o ninguna parte. |
| Ficha del Crush | Combina estructura API y contenido descifrado del Vault; muestra únicamente los campos del inventario que tengan valor, fotos locales y el Timeline del Crush. No expone IDs, correlaciones o fechas técnicas como datos del perfil. |
| Edición | Ofrece nombre, género, país, Ducky, fecha de encuentro, edad, profesión, WhatsApp, redes, cumpleaños, valoraciones y rasgos conforme a `RP-CRU-022`. Distingue omitir, cambiar y borrar un opcional; nunca permite editar derivados como signo o valoración global. |
| Edición lineal y revisión | Los pasos agrupan datos sin cambiar su obligatoriedad ni su tratamiento. Volver conserva el borrador del intento; cancelar conserva lo confirmado; la revisión final muestra el resultado propuesto antes de guardar una sola intención. |
| Fotos | Permite únicamente fotos locales, con las cuotas, tamaño, limpieza EXIF, exclusión de backup y eliminación definidas en `RP-FOT-001..009`. No ofrece video ni backup remoto de medios. |
| Eliminación del Crush | Sólo se inicia desde la fila activa, exige confirmación, persiste primero la intención y comunica progreso/error hasta retirar estructura, Vault y fotos conforme a `RP-DEL-001..006`. No aparece en la ficha. |

Archivo y restauración quedan excluidos de este bloque y se ejecutan únicamente en `FSR-07`.

## Baseline funcional `FSR-04` — Experiences, relaciones y Timeline

| Pantalla/flujo | Comportamiento funcional común |
|---|---|
| Añadir Experience | Es el único punto de entrada para registrar una Experience independiente, un cambio de situación, un cierre o la apertura de un período posterior. Siempre empieza por fecha civil, lugar, descripción y satisfacción. El selector aplica el intervalo de RP-EXP-011. Con período vivo, «¿Hay un cambio de situación?» añade un segundo paso opcional; con el último período cerrado, el segundo paso exige una situación inicial admitida por la historia conforme a RP-REL-007 para abrir el período nuevo. No existe una acción directa separada «Cambiar situación». |
| Cerrar período | Sólo usa `CLOSED` y un motivo catalogado compatible con la situación anterior. El cierre queda después de todos los hitos y Experiences ya contenidos; nunca mueve ni elimina una Experience posterior para poder cerrar. La explicación privada pertenece a la Experience; no existe una nota privada duplicada en período o hito. |
| Editar Experience | Permite modificar únicamente los campos autorizados y respeta fecha de encuentro, período, días ocupados y asociaciones. Mover una Experience no puede separarla del período ni ocupar el día de un hito incompatible. |
| Corregir o eliminar relación | Sólo ofrece las capacidades `latest-only` publicadas y válidas para ese elemento. Corregir a `CLOSED` queda bloqueado si la fecha inmutable del hito precede a otra Experience del período. Distingue eliminar el cambio de eliminar cambio y Experience; nunca ofrece borrado parcial del bloque inicial. |
| Timeline global | Presenta acontecimientos confirmados y pendientes en orden canónico. «Todos» conserva los acontecimientos integrados; «Crushes» muestra los bloques iniciales; «Experiencias» muestra todas las citas, incluidas la inicial y las vinculadas; «Relaciones» muestra una lista cronológica plana de inicios, cambios y cierres sin el contenido privado de la Experience. La acción global de añadir Experience exige seleccionar el Crush cuando todavía no está determinado. |
| Timeline del Crush | Limita esas mismas cuatro proyecciones al Crush abierto y permite añadir Experience cuando sus capacidades lo admiten; cualquier cambio de situación se registra dentro de ese flujo único. Un pendiente relevante bloquea únicamente la acción en conflicto y explica cómo resolverlo. |
| Tarjetas y acciones | En «Todos», hito más Experience se presenta como una unidad lógica y una Experience independiente se presenta separada. En «Experiencias», cada Experience aparece una vez como cita aunque esté vinculada; en «Relaciones», cada hito aparece una vez como evolución estructural y no repite el contenido de la cita. Cada menú muestra sólo acciones realmente ejecutables para esa identidad, sin inferirlas por la posición visual. Error de carga o decode nunca se presenta como vacío. |

Este baseline usa exclusivamente `RelationshipSituation`, períodos, hitos y `MatchStatus`; cualquier fase, etiqueta, `RelationshipState` o `everMatched` observable es una desviación. En Android, su retirada pertenece a `AND-GF-02`.

## Baseline funcional `FSR-05` — Statistics, Settings, cuenta y legal

| Pantalla/flujo | Comportamiento funcional común |
|---|---|
| Statistics | Presenta el dashboard personal y sus detalles sólo con métricas autorizadas en `STATISTICS.md`. Carga o error no fabrica valores; un vacío explica la falta de datos y permite agregar un Crush. Cada detalle conserva el mismo cálculo y alcance que su resumen. |
| Settings | Agrupa cuenta, idioma, seguridad, Vault, ayuda y legal. Sólo muestra una fila cuando la acción está disponible para el estado real de cuenta/plataforma; una fila visible siempre navega, confirma, ejecuta o explica un bloqueo. |
| Perfil de cuenta | Permite editar únicamente `birth_date`, `gender_id` y `country_id` conforme a Cuenta. Email, contraseña, proveedores e idioma conservan recorridos propios y no se mezclan como campos genéricos del perfil. |
| Idioma | Permite escoger un idioma soportado, aplica el copy correspondiente y conserva la preferencia. Cambiar idioma no modifica datos funcionales ni IDs de catálogo. |
| Email y verificación | Muestra el email de la cuenta y su estado. Sólo una cuenta verificada puede iniciar el cambio; el flujo exige el contrato de `ACCOUNT.md#cambio-email` y actualiza la sesión visible únicamente después de confirmarlo. |
| Contraseña y proveedores | Una cuenta con contraseña ofrece cambiarla; una cuenta de proveedor sin contraseña ofrece crear la primera; una cuenta sin el proveedor permitido puede iniciar su vinculación con confirmación explícita. Nunca muestra dos acciones incompatibles para el mismo estado. |
| Biometría y bloqueo | Toda pantalla privada queda cubierta al iniciar y al volver de segundo plano conforme a `RP-CRU-019` y `RP-LOCK-001..003`. Settings no puede ofrecer una opción que deje el diario privado utilizable sin esa protección; cualquier control biométrico existente en iOS debe contrastarse contra esta obligatoriedad antes de convertirse en referencia para Android. |
| Vault desde Settings | Sólo ofrece configurar un Vault inexistente o recuperar uno existente cuando ese estado sea real. Nunca permite crear un segundo Vault ni sobrescribir el remoto recuperable. |
| Ayuda y datos cifrados | FAQ y explicación de datos cifrados son informativas. Comunidad, feedback, notificaciones y transferencia QR permanecen ocultas mientras no formen parte de la beta activa. |
| Legal | Términos, privacidad y aviso legal muestran los textos vigentes que correspondan. Consultarlos no equivale a aceptar una versión; la aceptación se registra únicamente desde el flujo que la solicita expresamente. |
| Baja de cuenta | Exige el gate sensible y las confirmaciones definidas en Cuenta, ciclo de vida y Legal. Comunica el alcance sin prometer anonimato o destrucción inmediata donde no corresponda. |
| Cerrar sesión | Retira sesión y contexto privado en memoria, vuelve a la raíz pública y no elimina cuenta ni Vault remoto. |

### Contrato vigente — capacidades de la cuenta

`IOS-BRI-01A` (`GF-002`) resolvió esta fila: `UserProfilePayload.hasPassword`/`authMethods` son obligatorios y su ausencia, `null` o tipo incorrecto falla el decode completo. `AccountCapabilitiesPolicy` centraliza las decisiones derivadas; un estado de sesión sin capacidades confirmadas no concede biometría, cambio/alta de contraseña ni vínculo Apple, y `applyAppleLinked()` nunca fabrica una lista. Conforme a `RP-UX-008`, un estado desconocido produce error/reintento, nunca una capacidad inventada. `FSR-05` continúa con los demás estados de error/carga, copy y accesibilidad de esta pantalla.

Esto no elimina la disponibilidad controlada de Sign in with Apple definida expresamente en su plan maestro: ese flag decide si el proveedor se ofrece durante el rollout de beta y, cuando está apagado o falla, debe informar indisponibilidad y permitir reintentar conforme a `RP-APL-023`. No concede acceso ni sustituye el estado de la cuenta.

## Contrato ya cerrado: listas de Crushes

La autoridad funcional es [`CRUSH.md#listas`](../../../rules/CRUSH.md#listas):

- activos y archivados tienen la misma búsqueda, filtros y orden;
- la búsqueda cubre nombre y país;
- el orden permite actividad reciente o fecha de encuentro;
- los filtros son resultado de match, situación relacional vigente, género (`man`, `woman`, `trans`) y Ducky, con las opciones fijadas en la regla;
- las selecciones se combinan y se calculan sobre el conjunto completo, no sólo sobre la página visible;
- nombre, género y texto buscado permanecen en el dispositivo con el Vault desbloqueado;
- la pantalla de archivados es una superficie requerida de la beta y conserva únicamente acciones compatibles con el archivo;
- su ejecución se posterga hasta terminar el inventario, las reglas y las fases del resto de pantallas.

### Contraste inicial de implementación

| Superficie | Evidencia vigente | Resultado |
|---|---|---|
| iOS activos | Conserva orden, match, género y Ducky; género y búsqueda se resuelven localmente sobre el conjunto paginado. No ofrece todavía el filtro por `RelationshipSituation`. | Incumple el inventario completo. |
| iOS archivados | No existe todavía una pantalla terminada con el contrato de `RP-LIS-001..004`. | Diferida al último bloque. |
| Android activos | Expone las categorías históricas, pero envía `gender_id` a la API, utiliza valores legacy para match/relación y la búsqueda local opera sobre datos ya cargados. | Incumple la frontera privada y la semántica vigente. |
| Android archivados | Existe pantalla, acceso desde la lista, consulta `GET /crushes/archived` y restauración, con pruebas locales reportadas. La búsqueda actual opera sobre filas cargadas, no cubre nombre y envía `gender_id` a la API; no cumple aún `RP-LIS-001..004`. Se conserva sin cambios hasta `FSR-07`. | Base adelantada, no aceptada como pantalla completa; revisión y conexión final diferidas al último bloque. |
| API | Activos y archivados comparten orden, match y Ducky; rechaza `gender_id`. Todavía no ofrece el filtro estructural por situación vigente y conserva una entrada `search` que los clientes conformes no deben utilizar. | Soporte parcial; no define por sí solo la interfaz. |

Este contraste registra el punto de partida; no acepta código ni sustituye las matrices y fases locales que se abrirán después de cerrar las reglas de cada bloque.

## Orden de trabajo

| Fase | Alcance | Estado | Salida |
|---|---|---|---|
| `FSR-00` | Inventario completo de rutas, pantallas, recorridos y estados de iOS; correspondencia con reglas, planes y superficies previstas de la beta. | EN CURSO; mapa base completado | Matriz única de pantallas iOS dentro de este README, sin convertir código accidental en regla. |
| `FSR-01` | Acceso, cuenta, verificación, recovery, onboarding, Vault y App Lock. | EN REVISIÓN; acceso permisivo resuelto por `IOS-EMAIL-09`/`IOS-BRI-01A`; copy bilingüe de onboarding/recovery automatizadamente validado | Completar estados de error/carga y la comprobación física ES/EN de onboarding/recovery; accesibilidad queda transversal. |
| `FSR-02` | Home, navegación, lista activa, búsqueda, filtros, mapa y estados vacíos. | EN REVISIÓN; Home vacío/poblado demostrado y cifras de maqueta retiradas | Contrastar los demás estados y corregir la presentación recortada de «No disponible» sin abrir todavía Archivados. |
| `FSR-03` | Alta, ficha, edición, fotos y eliminación del Crush. | EN REVISIÓN; baseline funcional redactado | Contratos de pantalla contrastados contra iOS y reglas canónicas. |
| `FSR-04` | Experiences, Relationships y Timeline. | EN REVISIÓN; contrato relacional final integrado | Contrastar estados de pantalla con tamaño de texto normal, sin abrir trabajo de lectores de pantalla o texto ampliado conforme a `RP-UX-006`. |
| `FSR-05` | Statistics, Settings, cuenta, seguridad, baja y superficies legales. | EN REVISIÓN; raíz reactiva de sesión/idioma y Statistics real implementadas | Contrastar contratos y recorrer físicamente cambio de idioma con App Lock, logout y Home/Statistics ES/EN; revisión Legal donde corresponda. |
| `FSR-06` | Contraste de Android contra los contratos funcionales extraídos y validados desde iOS; diseño de fases locales ejecutables para ambas plataformas. | BLOQUEADA por `FSR-00..05` | Gaps clasificados, diferencias nativas justificadas, tests uno a uno y fases locales sin reglas duplicadas. |
| `FSR-07` | Archivo, acceso a Archivados, lista archivada completa y restauración en iOS y Android. | ÚLTIMA; BLOQUEADA por `FSR-00..06` | Misma presentación funcional, búsqueda, filtros y orden que activos; sólo fuente y acciones de ciclo de vida propias. |
| `FSR-CLOSE` | Integración estable y cierre documental. | BLOQUEADA por `FSR-07` | Reglas integradas en documentos canónicos y eliminación de este plan conforme a `AGENTS.md` §9. |

## Interacción con iniciativas activas

- `business-rule-integrity` conserva el estado de cumplimiento de cada regla canónica; las reglas nuevas empiezan como no conformes cuando ya existe evidencia directa de una contradicción.
- `android-parity` implementará y validará las fases Android que resulten del contrato funcional cerrado a partir del inventario iOS. La pantalla de archivados se ejecuta al final, en `FSR-07`.
- El contrato estable de Relationship aporta `RelationshipSituation` y su situación vigente sin `RelationshipState`, fases o etiquetas legacy; Android debe consumirlo en `AND-GF-02`.
- `vault-e2ee`, [Guardado y sincronización](../../../rules/SYNC.md) y la arquitectura transversal mantienen las fronteras privadas y los estados pendientes; este plan sólo fija cómo se observan y accionan en cada pantalla.
- `empty-state-standardization` y `form-ux-performance` siguen siendo autoridad de ejecución para sus alcances ya aprobados y deben enlazar las reglas de pantalla, no redefinirlas.

## Gates antes de implementar

Para cada bloque:

1. inventariar en iOS la pantalla, sus entradas, estados, opciones y recorridos;
2. enlazar todas las reglas existentes y separar comportamiento decidido de comportamiento accidental;
3. consultar las decisiones faltantes a Producto de una en una;
4. actualizar primero el documento canónico correspondiente;
5. contrastar API y después Android contra ese contrato funcional mediante MCP, sin copiar a Android detalles exclusivos de UIKit/SwiftUI;
6. clasificar diferencias nativas aceptables y gaps funcionales reales;
7. crear o actualizar las fases locales sólo después de congelar el contrato;
8. exigir tests mapeados uno a uno y validación manual proporcional al riesgo en ambas plataformas.

## Decisión diferida

El punto de entrada y la navegación definitivos de la pantalla de archivados se validarán únicamente al llegar a `FSR-07`; el acceso que ya existe en Android no se toma como decisión de Producto ni se modifica ahora. Su existencia, su conjunto de filtros y su paridad funcional con activos ya están cerrados; no bloquean el trabajo previo.
