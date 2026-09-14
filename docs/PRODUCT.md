# Producto MDL

Este documento es la fuente de verdad del comportamiento estable de MyDuckingLife (MDL). Describe qué puede hacer una persona, qué datos existen y qué reglas debe respetar cualquier cliente o servicio. Los detalles de implementación viven en cada proyecto; el trabajo todavía en curso vive únicamente en los planes maestros activos.

Algunas reglas llevan un identificador estable `BR-<sección>.<número>` para que los planes locales de API/iOS/Android las referencien sin copiarlas. No todas las reglas tienen ID todavía — se añaden progresivamente donde hace falta referenciarlas desde otro documento. Los escenarios Dado/Cuando/Entonces de una iniciativa en curso (`GWT-xx`) viven en su plan maestro activo, no acá; se integran a este documento recién cuando la iniciativa cierra.

## 1. Propósito y límites

MDL es un diario privado para personas adultas que permite registrar crushes, experiencias y relaciones, consultar su cronología y obtener estadísticas personales.

- La persona usuaria debe tener al menos 18 años.
- Cada cuenta sólo puede acceder a sus propios datos.
- No hay perfiles públicos, interacción social ni administración funcional de contenido ajeno.
- La beta es gratuita y no incorpora pagos.
- El contenido íntimo pertenece al Private Journal Vault y se cifra de extremo a extremo.
- La API sólo conserva los datos estructurales necesarios para operar el producto.

Los requisitos legales y de tratamiento de datos se mantienen en [`legal/`](legal/README.md). Esos documentos no se sustituyen ni resumen aquí.

## 2. Vocabulario funcional

- **Crush:** persona sobre la que la usuaria mantiene un registro privado.
- **Experiencia:** acontecimiento ocurrido con un crush en un día civil concreto.
- **Relación:** estado relacional con vigencia cronológica.
- **Bundle inicial:** creación del crush, su relación inicial obligatoria y, opcionalmente, una experiencia inicial completa, todos asociados al mismo `met_at`.
- **Timeline:** vista cronológica de altas, experiencias y cambios de relación.
- **Vault:** almacén cifrado de contenido íntimo y de los snapshots necesarios para escrituras durables.
- **Cambio pendiente:** escritura guardada localmente que todavía no alcanzó confirmación remota y sincronización verificable.
- **Día civil:** fecha elegida por la persona en su calendario local (`yyyy-MM-dd`), sin conversión de zona horaria que pueda cambiar el día. Es el «mismo día» de producto: no una ventana móvil de 24 horas ni un día UTC del servidor.

## 3. Cuenta, autenticación y sesión

### 3.1 Registro e inicio de sesión

- El registro requiere email, contraseña y aceptación de términos. (BR-3.01)
- El registro exige aceptar explícitamente la versión vigente de términos y de política de privacidad; una versión ausente o desactualizada rechaza el registro. (BR-3.02)
- El email identifica la cuenta sin distinguir mayúsculas y minúsculas. (BR-3.03)
- La contraseña requiere mínimo 8 caracteres, con al menos una mayúscula, una minúscula, un dígito y un carácter especial del conjunto admitido. (BR-3.04)
- La sesión expira automáticamente **7 días desde la emisión del token**, independientemente de la actividad; no se renueva por uso. Cerrar sesión puede revocarla antes. (BR-3.05)
- El registro crea una **cuenta provisional** y puede abrir una sesión restringida únicamente a la verificación de email. No permite acceso funcional al diario ni inicia/configura el Vault. (BR-3.12)
- La cuenta provisional tiene un plazo fijo de **siete días desde el alta** para completar la verificación. Generar o reenviar OTPs no reinicia ni extiende el plazo. (BR-3.13)
- Mientras el email no esté verificado, todo cliente nativo muestra exclusivamente el flujo de verificación; API sólo permite las operaciones necesarias para verificar o cerrar sesión y rechaza las operaciones funcionales. Una verificación correcta desbloquea el acceso sin requerir una nueva autenticación. (BR-3.14)
- Al vencer el séptimo día sin verificación, API elimina íntegramente la cuenta provisional mediante el flujo de baja aplicable y libera el email para un nuevo registro. No conserva el email como lead ni para comunicaciones comerciales. (BR-3.15)
- Tras autenticarse y verificar el email, la app obtiene el perfil y resuelve el estado del Vault antes de permitir acceso al diario. El onboarding de Vault se presenta sólo en ese momento.
- Cerrar sesión elimina la sesión de la app; no elimina la cuenta ni su Vault remoto.

### 3.2 Recuperación de la contraseña de acceso

- La recuperación usa un OTP de seis dígitos enviado por email; la respuesta a la solicitud es idéntica exista o no la cuenta (anti-enumeración). (BR-3.06)
- El cooldown de reenvío escala en tres pasos (progresivamente más largo) según la cantidad de reenvíos solicitados. (BR-3.07)
- El código queda invalidado al alcanzar el límite de intentos fallidos de verificación y se purga automáticamente una vez expirado. (BR-3.08)
- Toda respuesta pública fallida al verificar o usar el OTP de recuperación es indistinguible: devuelve el código estable `invalid_or_expired_code`, incluso cuando el OTP quedó bloqueado por intentos. El bloqueo conserva su efecto y su causa queda sólo en los eventos y logs internos saneados. (BR-3.16)
- La contraseña de acceso y la contraseña de recuperación del Vault son credenciales distintas.

### 3.3 Bloqueo local

- App Lock protege una sesión ya autenticada al volver desde background.
- App Lock no reemplaza la autenticación ni la recuperación criptográfica del Vault.
- Un Vault remoto existente en un dispositivo nuevo debe ofrecer recuperación, nunca un onboarding que lo sobrescriba.

### 3.4 Tutoriales y onboarding informativo

- El tutorial inicial y el tutorial de profundización del primer crush son informativos: no recolectan datos personales.
- Se puede avanzar y retroceder sin perder el paso actual.
- La finalización se comunica una sola vez al llegar al último paso y evita doble submit.
- Un fallo al marcar el tutorial no puede fingir éxito ni dejar el CTA permanentemente bloqueado.

### 3.5 Perfil de la cuenta y ajustes

- El formulario de perfil se precarga con los valores actuales.
- Un patch distingue valor nuevo, limpieza explícita y campo sin cambios.
- La fecha de nacimiento de la cuenta es un día civil y la edad visible se deriva de ella.
- Cambiar idioma se aplica en runtime y se persiste; la beta soporta como mínimo español e inglés.
- App Lock es local y no revoca el token backend.
- La baja de cuenta exige motivo disponible o fallback permitido, confirmación explícita y salida del flujo autenticado sólo después de éxito real.
- La baja es irreversible: los datos identificables se sobrescriben con marcadores anónimos y no pueden recuperarse; los registros se conservan únicamente para estadísticas agregadas sin PII. Los plazos y el detalle campo por campo viven en `legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md`. (BR-3.09)
- Una cuenta sin iniciar sesión durante 18 meses recibe una notificación de inactividad y, si no vuelve a iniciar sesión dentro de los 20 días siguientes, se cierra automáticamente con el mismo flujo de anonimización que una baja manual. Volver a entrar durante esos 20 días cancela el cierre. (BR-3.11)
- Notificaciones y otras superficies de ajustes no pueden mezclar credenciales con UserDefaults.

## 4. Alta y perfil de un crush

> Detalle completo de entidades, cronología, matrices de transición y acciones del Timeline: [`TIMELINE.md`](TIMELINE.md).

### 4.1 Datos obligatorios

El alta requiere, como mínimo:

- identidad funcional del crush;
- nombre;
- género;
- país;
- categoría personal (ducky) elegida por la persona usuaria;
- fecha `met_at`;
- edad con su fecha de confirmación `age_as_of`;
- estado de relación inicial.

(BR-4.01)

El nombre y el género se validan como obligatorios en el formulario antes de persistir la intención cifrada.

- El **nombre** es contenido privado: se guarda únicamente en el Vault cifrado y no se envía a la API funcional.
- El **`gender_id`** es un atributo estructurado obligatorio. Durante la beta se conserva tanto en el Vault cifrado como en API/RDS; esta duplicación es transitoria y queda pendiente de revisión legal de proporcionalidad y riesgo. Hasta una decisión distinta, ambas copias deben representar el mismo valor.

`POST /crushes` valida `gender_id` contra el catálogo estructural, lo propaga sin transformación y persiste el valor realmente elegido. La ausencia o un valor inválido se rechazan antes de crear estructura o bindings parciales; un retry idempotente conserva el valor originalmente confirmado. Esta regla no reintroduce el nombre en la API funcional.

El alta genera un identificador funcional estable antes de intentar la API. Ese mismo identificador se reutiliza en reintentos y en la proyección local.

### 4.2 Experiencia inicial opcional

La experiencia inicial es un bloque todo-o-nada compuesto por:

- lugar;
- descripción;
- satisfacción.

Si los tres campos están vacíos, el alta puede continuar sin experiencia inicial. Si se completa cualquiera, los tres pasan a ser obligatorios. La interfaz señala los campos incompletos junto a sus etiquetas y el guardado no persiste ninguna operación parcial.

Cuando existe, la experiencia inicial usa el mismo día civil que `met_at`, pertenece al mismo bundle inicial, conserva una identidad funcional propia y aparece unificada con el alta y la relación inicial en el Timeline.

### 4.3 Edición del perfil

- Los datos del perfil se pueden editar aunque exista un cambio pendiente del Timeline del mismo crush.
- Sólo puede haber una edición de perfil pendiente por crush.
- Cambiar `met_at` no convierte la edición en una operación de Timeline.
- Una edición de perfil pendiente que **no** modifica `met_at` no bloquea ninguna acción del Timeline.
- Si la edición de perfil pendiente modifica realmente `met_at`, se bloquea únicamente crear una nueva experiencia de ese mismo crush hasta que el cambio se sincronice, se corrija o se descarte. No se calcula un rango hipotético entre la fecha confirmada y la propuesta.
- El rango permitido de `met_at` termina en el primer evento cronológico posterior, confirmado o pendiente, del mismo crush.
- Un cambio pendiente de otro crush nunca limita el calendario ni las acciones de éste.
- Las valoraciones individuales (belleza, conexión emocional, personalidad, atractivo) sólo son editables desde el perfil, nunca en el alta. La valoración global es el promedio aritmético de las valoraciones cargadas, redondeado a 2 decimales, y la recalcula siempre la API — un valor de valoración global enviado por el cliente se ignora. (BR-4.02)

### 4.4 Archivar y eliminar

- Archivar oculta el crush de la lista activa sin eliminar su información.
- Archivar y restaurar siguen disponibles aunque el Timeline tenga una escritura pendiente.
- Eliminar el crush es una acción explícita, destructiva y confirmada. Debe cancelar o resolver de manera segura sus escrituras pendientes y aplicar el contrato de borrado del Vault y la API; nunca puede quedar como un botón inerte.

## 5. Edad, cumpleaños y zodíaco

### 5.1 Fuente de verdad

- `age` y `age_as_of` son inseparables: la edad representa una afirmación confirmada en ese día civil.
- Día y mes de cumpleaños son privados y viven cifrados en el Vault; no se registra el año de nacimiento del crush.
- El signo zodiacal es derivado y puede enviarse como dato estructural a la API.
- `met_at` no es el ancla de la edad.

### 5.2 Lectura

- Con cumpleaños conocido, la edad actual se calcula desde `age`, `age_as_of` y el aniversario correspondiente.
- Sin cumpleaños, la app presenta una edad exacta en la fecha de confirmación y, cuando corresponde, un intervalo posible después.
- El cálculo nunca puede producir una edad inferior a la última edad confirmada.
- Para un cumpleaños del 29 de febrero, el aniversario operativo es el 28 de febrero en años no bisiestos.

### 5.3 Edición

- Editar campos no relacionados omite edad, ancla y cumpleaños.
- Editar directamente la edad fija `age_as_of` al día civil de confirmación.
- Añadir o cambiar cumpleaños recalcula primero la edad actual usando el snapshot anterior y después fija el nuevo ancla.
- Eliminar cumpleaños conserva la edad actual calculada, fija un nuevo ancla y elimina cumpleaños y zodíaco.
- Si en una misma edición se cambia explícitamente la edad, ese valor prevalece.
- Un reintento reutiliza el mismo día civil congelado; no vuelve a calcularlo con la fecha del reintento.

## 6. Experiencias

### 6.1 Creación

- Una experiencia pertenece a un único crush.
- Sólo puede existir una experiencia por crush y día civil.
- Si se quiere ampliar o corregir lo ocurrido ese día, se edita esa misma experiencia; no se crea una segunda.
- La fecha es obligatoria y se serializa como el día elegido, sin desplazamiento UTC.
- Una experiencia puede incluir un cambio de relación dentro del mismo guardado.
- Si una experiencia normal se crea en el mismo día civil que una relación standalone sin experiencia, ambas se vinculan automáticamente: conservan sus UUID funcionales, comparten la correlación de la relación y aparecen como una sola tarjeta vinculada.
- Si esa relación ya tiene una experiencia vinculada, otra experiencia no puede crearse en ese día.
- Si el crush no tiene experiencia inicial, una experiencia normal no vinculada creada o editada exactamente en `met_at` ocupa el slot inicial y conserva su mismo UUID.
- Esa reasociación sólo es válida si el slot inicial está libre y la experiencia no está vinculada a otra relación.
- Un retry idéntico debe ser idempotente.

### 6.2 Edición

- Una experiencia confirmada se puede editar respetando unicidad y cronología.
- Una experiencia vinculada mantiene bloqueada su fecha; se editan los demás campos permitidos.
- Una experiencia ya inicial no puede alejarse de `met_at` silenciosamente.
- Una experiencia normal no vinculada puede moverse a `met_at` cuando el slot inicial está libre; en ese caso pasa a ser la experiencia inicial de forma explícita y conserva identidad.
- Mover una experiencia normal no vinculada al día civil de una relación standalone sin experiencia produce el mismo vínculo automático que el alta. Si el vínculo ya está ocupado, se rechaza sin mutar la experiencia.
- La beta no ofrece una acción independiente de «Desvincular». La fecha de una experiencia vinculada permanece bloqueada; eliminar esa experiencia conserva la relación standalone y permite crear después otra experiencia donde corresponda.
- Los rechazos de validación deben identificar el campo y explicar la causa con un mensaje comprensible, por ejemplo la existencia de otra experiencia en ese día.

### 6.3 Borrado

- Toda experiencia, confirmada o pendiente, se puede eliminar.
- Una experiencia no se somete a la regla de «sólo la última» porque no representa por sí sola un estado cronológico.
- El botón de borrar sólo se oculta cuando la acción no existe; si se muestra, debe abrir confirmación o ejecutar una resolución válida, nunca fallar silenciosamente.
- Borrar una experiencia vinculada conserva la relación y elimina únicamente el vínculo.
- Borrar la experiencia inicial libera el slot inicial para otra experiencia válida.
- Borrar una experiencia pendiente descarta de forma segura su operación y proyección local sin emitir una creación posterior.

## 7. Relaciones

### 7.1 Cronología

- Todo crush se crea con un estado de relación inicial asociado a `met_at`.
- La relación inicial es parte obligatoria de la identidad mínima del crush y nunca se puede eliminar.
- La relación inicial se puede editar únicamente mientras siga siendo la relación latest, es decir, mientras no exista una relación posterior. Puede cambiar a cualquiera de los estados permitidos, pero su fecha permanece fijada a `met_at`.
- Los cambios posteriores forman una secuencia cronológica por crush.
- Una relación no puede empezar antes de `met_at` ni antes de la relación previa.
- Sólo puede existir un cambio de relación por crush y día civil. Si se quiere corregir o ampliar lo ocurrido ese día, se edita la relación latest de ese día; no se crea una segunda transición.
- Sólo la relación más reciente es editable. Sólo una relación posterior que además sea la más reciente es eliminable; la interfaz nunca ofrece eliminar la relación inicial ni mutar relaciones históricas.
- Eliminar la última relación posterior reabre la relación anterior, que puede ser la inicial.
- Cuando una experiencia crea un cambio de relación, ambos se guardan como una única intención y se muestran unidos en el Timeline.

Una experiencia y una relación pueden compartir día civil cuando forman un vínculo explícito. Si una operación reúne dos elementos standalone elegibles en el mismo día, pasan a compartir la correlación de la relación; no quedan dos tarjetas independientes. Una relación que ya tiene experiencia vinculada no admite otra.

### 7.2 Match status

- El match status se deriva del estado de relación y es monotónico: una vez Sí, nunca retrocede. (BR-7.01)
- Tabla de derivación completa y matriz de monotonicidad: [`TIMELINE.md`](TIMELINE.md) §4.

## 8. Timeline

> Reconstrucción, matriz de acciones por tipo de item y reglas de edición/borrado por entidad: [`TIMELINE.md`](TIMELINE.md) §5-7.

### 8.1 Contenido e identidad

- El Timeline de un perfil sólo contiene entidades de ese crush.
- El Timeline global puede mezclar eventos de distintos crushes, manteniendo la identidad y el contexto de cada uno.
- Un pendiente se fusiona con su contraparte confirmada por identificador funcional, nunca por el ID interno de una entry del Vault.
- Una operación pendiente se representa una sola vez: actualiza una tarjeta confirmada o sintetiza una provisional, pero no ambas.
- Los bundles se muestran como una unidad visual cuando alta, experiencia y relación pertenecen a la misma correlación funcional.
- Las fechas admitidas por API y snapshots legacy deben mostrarse como día civil; `-` sólo representa datos realmente corruptos o ausentes.

### 8.2 Orden

- Los eventos se ordenan por día civil descendente.
- Para empates, el orden debe ser determinista y conservar la unidad del bundle.
- Una proyección pendiente sólo puede modificar o bloquear el crush indicado por su `scopeCrushId`.

### 8.3 Acciones

- Con el Timeline sincronizado se pueden crear y editar experiencias y crear o editar la última relación conforme a sus reglas de dominio.
- Con una operación de Timeline pendiente, se bloquean nuevas escrituras de Timeline del mismo crush para preservar el orden.
- Ese bloqueo no se propaga al perfil, a archivar/restaurar ni a otros crushes.
- Una operación de Profile pendiente sólo afecta al Timeline en la excepción puntual de alta de experiencia por cambio real de `met_at` definida en §4.3.
- Las experiencias confirmadas siguen siendo eliminables aunque exista una operación de Timeline pendiente.
- Las relaciones sólo ofrecen editar o borrar cuando además de estar confirmadas son la última relación elegible.

### 8.4 Actividad reciente de Home

Home muestra un feed breve de actividad confirmada. Sólo se registra una actividad cuando ocurre una de estas acciones:

- creación de un crush;
- creación de una experiencia;
- creación confirmada de una relación posterior, que representa el nuevo estado relacional en la cronología.

La relación inicial obligatoria forma parte del alta del crush y no genera por sí sola una segunda actividad. Editar una relación ya existente —aunque cambie su `state_id`— tampoco genera actividad reciente: no representa un nuevo evento de la cronología. Las ediciones de crush o de experiencia y los borrados tampoco aparecen en este feed. La API conserva únicamente las referencias estructurales del evento; los nombres y lugares que muestra Home se resuelven desde el Vault local desbloqueado.

Cuando una experiencia crea simultáneamente una relación posterior, ambas forman una sola intención: Home la representa como cambio de relación, no como una segunda actividad de experiencia ni como dos tarjetas. Una experiencia sin relación posterior se representa como experiencia.

## 9. Escrituras durables y sincronización visible

El contrato completo de la beta se mantiene mientras está activo en [`durable-write-feedback`](plans/active/durable-write-feedback/README.md). Sus principios estables son:

- El formulario confirma «guardado en este dispositivo» después de persistir de forma cifrada la intención local, no después de completar toda la red.
- La UI proyecta el cambio local con estado **Sincronizando**, **Esperando conexión**, **Necesita revisión** o **Sincronizado** cuando existe evidencia verificable.
- Una escritura pendiente reserva únicamente su dominio funcional (`root`, `profile` o `timeline`) y su crush.
- Los reintentos automáticos sólo se realizan para fallos transitorios, con backoff y límite; una validación rechazada no entra en un bucle.
- **Necesita revisión** debe mostrar el motivo saneado y ofrecer las acciones realmente disponibles: reintentar cuando corresponda, corregir cuando exista formulario compatible o descartar.
- Descartar elimina la intención local que nunca se confirmó; no borra una entidad remota que no llegó a crearse.
- Una operación con evidencia de aceptación remota termina su verificación/sync antes de considerarse descartable.
- Cambiar de cuenta, bloquear el Vault o cambiar de namespace limpia las proyecciones en memoria y nunca mezcla datos entre cuentas.

## 10. Vault, privacidad y fotos

> Modelo criptográfico, límites vigentes, estado legacy y roadmap dormido: [`VAULT.md`](VAULT.md).

- El contenido íntimo y los snapshots de operaciones se cifran en el dispositivo antes de salir de la app.
- Cada crush, experiencia o relación tiene un snapshot cifrado completo; no se crea un blob separado por cada campo.
- Una edición crea una revisión nueva de la misma entidad. La anterior sólo se conserva hasta verificar la nueva y después se retira; Vault no ofrece un historial de versiones al usuario.
- La API no recibe claves de descifrado ni acepta o devuelve contenido privado en claro en el contrato funcional vigente. Todo cliente debe resolver ese contenido desde el Vault local.
- Cada entidad privada nueva queda unida a sus blobs mediante un identificador opaco derivado en el cliente. El backend puede usarlo para autorizar y borrar ciphertext, pero no puede derivar la identidad, el contenido ni la VMK a partir de ese valor.
- Borrar una experiencia o una relación elegible elimina atómicamente la entidad funcional y aplica soft delete a todas sus revisiones remotas vinculadas. La app realiza una única petición funcional, sin recorrer ni descifrar el inventario completo del Vault.
- Un Vault remoto se recupera con sus mecanismos de recuperación; no se recrea por ausencia de estado local.
- Recovery puede listar, descargar y validar el inventario global, pero nunca borra objetos remotos. Publica el conjunto recuperado sólo después de validarlo completo.
- Durante la beta, cada Vault se usa en **un único dispositivo activo**. Recovery permite sustituir el dispositivo, no mantener dos copias activas en paralelo. La sincronización continua entre dispositivos y su resolución de conflictos quedan fuera de la beta.
- La recuperación exitosa y la finalización del onboarding limpian los marcadores transitorios correspondientes.
- Las fotos de crush se consideran privadas. En la beta actual permanecen en almacenamiento local saneado y no tienen backup remoto.
- Las fotos no deben incorporarse por accidente a backups del sistema; su ciclo de eliminación debe acompañar al crush y a la cuenta según Legal.
- El diseño de sincronización remota de fotos queda fuera de la beta vigente.

## 11. Mapa

- El mapa es una visualización de solo lectura; no crea ni modifica crushes.
- Agrupa por país y usa coordenadas representativas del país, nunca la ubicación GPS precisa de una persona.
- Sólo puede haber un popup de país abierto y tocar el fondo lo cierra.
- Un país sin crushes no genera anotación; una cuenta sin datos muestra estado vacío localizado.
- Volver a la lista no modifica filtros ni datos.

## 12. Estadísticas e IA

> Política de métricas prohibidas, filosofía de IA y qué endpoints salen realmente en la beta: [`STATISTICS.md`](STATISTICS.md).

### 12.1 Estadísticas

- Las estadísticas son de solo lectura y se calculan en API sobre datos estructurales de la propia cuenta.
- La app no recalcula ratios/distribuciones por su cuenta.
- Durante carga o error se conserva la estructura visual con placeholders; un error ofrece retry y no expone texto técnico.
- Una respuesta sin muestra suficiente muestra estado vacío/placeholder, no valores engañosos.
- Comparaciones, cobertura y confiabilidad son opcionales y sólo se muestran cuando API las provee.
- Las superficies incluyen resumen, historia, satisfacción, conversiones, ritmo, ratings, actividad, países, zodíaco, estados relacionales, rasgos y género conforme al dashboard vigente.

### 12.2 IA e inferencias

- Las inferencias deben poder explicarse por sus entradas y no afirmar causalidad cuando sólo existe asociación.
- El servicio LLM sólo recibe el contexto mínimo autorizado por el flujo correspondiente.
- Los insights son auxiliares y no modifican automáticamente el historial.
- Los datos de una persona no se usan para entrenar modelos generales sin una decisión de producto y legal explícita.

### 12.3 Funciones no activas en beta

- Tests de apego/personalidad, push notifications completas, exportación portable, suscripciones/IAP e insights narrativos de IA no están disponibles como funciones finales de beta aunque exista infraestructura parcial.
- No se activa una de estas superficies sólo porque haya código legacy; requiere una decisión y plan maestro aprobados.

## 13. Mensajes, errores y accesibilidad

- Los errores de API de validación se muestran en lenguaje de producto, no como claves internas (`error.*`).
- Una acción deshabilitada explica la causa cuando la persona necesita resolverla.
- No se presenta una acción que no pueda ejecutarse.
- Cancelar una confirmación no muta datos ni deja estados visuales heredados para la siguiente alerta.
- Los indicadores de sincronización tienen etiqueta accesible y no dependen sólo del color.

## 14. Trabajo activo y cambios futuros

### 14.1 Iniciativas activas

- [`Private Journal Vault`](plans/active/vault-e2ee/README.md) — baseline nuevo validado; migración legacy, corte de campos privados y borrado integral todavía pendientes.
- [`Cambios pendientes visibles en beta`](plans/active/durable-write-feedback/README.md).
- [`Verificación de email`](plans/active/email-verification/README.md) — OTP, envío, cliente y gates implementados; falta completar el régimen de cuenta provisional de siete días, validación física y rollout.
- [`Sign in with Apple`](plans/active/sign-in-with-apple/README.md) — API-APPLE-01 a 04 e iOS-APPLE-01 a 03 implementadas; quedan lifecycle iOS, beta readiness y aprobación de disclosures públicos antes del rollout.
- [`Estandarización de empty states en iOS`](plans/active/empty-state-standardization/README.md) — diseñada, sin empezar a implementar.
- [`Fluidez y experiencia de formularios iOS`](plans/active/form-ux-performance/README.md) — diseñada; unifica foco, teclado, validación y rendimiento de los formularios sin cambiar reglas de negocio.
- [`Rediseño de relaciones`](plans/active/relationship-redesign/README.md) — contrato promovido; prepara el modelo nuevo sólo para cuentas nuevas y deja la migración de la única cuenta legacy como fase posterior, no bloqueante para beta.
- [`Paridad Android`](plans/active/android-parity/README.md) — incorporación del cliente Android al contrato funcional, criptográfico, durable y visual vigente; la base actual todavía no es apta para producción.

Una regla nueva o modificada se documenta primero en el plan maestro de Product. Los planes locales describen cómo implementarla. Al cerrar una iniciativa, su comportamiento final se integra en este documento o en [`ARCHITECTURE.md`](ARCHITECTURE.md) y el plan se elimina; no se conserva un historial documental paralelo.

### 14.1.1 Validaciones manuales aún pendientes antes de una beta distribuida

Esta sección reúne únicamente gates físicos u operativos que no se pueden sustituir por tests automatizados. Cada plan activo conserva el procedimiento técnico; esta lista se reduce al incorporar evidencia, no es un registro histórico ni habilita una fase adicional por sí sola.

- **Verificación de email:** observar una ejecución nocturna correcta del scheduler y demostrar, con una cuenta de prueba controlada, que el vencimiento de los siete días elimina una cuenta provisional sin dejar acceso, tokens, Vault ni artefactos locales activos.
- **Vault:** comprobar la purga real de blobs soft-deleted al completar su ventana de siete días y confirmar que `LocalMedia/` no se incluye en un backup de dispositivo. No se simula modificando fechas o borrando datos reales de una cuenta de uso.
- **Sign in with Apple:** tras una baja de cuenta Apple de prueba, comprobar que la intención de revocación pasa de `pending` a `completed` o queda como `failed` visible para intervención; validar físicamente el vínculo de una cuenta MDL existente y la revocación de credencial desde Ajustes de Apple. Antes de exponer la función a terceros se completan además Private Relay/notificaciones server-to-server y los disclosures públicos aprobados.

### 14.2 Futuro, no activo

Trabajo evaluado pero no promovido a plan activo. No se implementa nada de esto hasta que se promueva explícitamente, y viene después de cerrar las iniciativas activas de 14.1.

**Observabilidad y alertas operativas** — capacidad futura, fuera de beta. MDL debe poder avisar a una persona operadora por un canal de operaciones (inicialmente alertas del proveedor de monitorización y/o email) cuando una condición de fondo requiere intervención: una revocación de Apple alcanza su fallo final tras cinco intentos, un scheduler o worker deja de ejecutarse, o una cola/intención supera su ventana operativa. No genera un dashboard propio en esta etapa ni alerta por cada error transitorio. Cada alerta debe estar deduplicada, tener severidad y remitir a una acción segura o runbook concreto; sólo puede incluir identificadores técnicos mínimos, nunca email, subject federado, tokens, JWS ni contenido del Vault. La selección de proveedor, destinatarios, umbrales y retención se define al promover esta iniciativa, conforme a [`ARCHITECTURE.md` §10](ARCHITECTURE.md#10-seguridad-y-observabilidad) y Legal.

**Idea de estadísticas sin iniciar** (no prioritaria): contexto de comunidad mediante promedios globales anónimos, nunca rankings, percentiles ni segmentación demográfica. Sólo se publica una métrica con una muestra mínima de 50 personas; el agregado no conserva `user_id`, se recalcula fuera del request y el campo es opcional/null por debajo del umbral. El resto de la deuda técnica de estadísticas (alias legacy, endpoint `overview`, cache del ai-payload) ya está resuelta — ver `api/docs/STATISTICS.md` §7.
