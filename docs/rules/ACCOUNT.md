# Reglas de producto — Cuenta de usuario

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Alta y acceso, verificación, Apple, Google, perfil de la cuenta de usuario, ajustes y recuperación de acceso.

<a id="requisitos"></a>
## Requisitos de la cuenta

- **RP-BASE-001.** MDL es un diario privado para personas de **18 años o más**. Quien cumple 18 hoy puede registrarse; quien todavía tiene 17, no. La regla aplica a todos los canales de alta.

<a id="inventario-cuenta"></a>
## Inventario funcional de la cuenta

- **RP-CUE-005.** La cuenta de la beta se compone del siguiente inventario cerrado. Cada ampliación del inventario requiere una nueva decisión de Producto.

Para este inventario, «legible por la API» significa que el servidor puede interpretar el valor y utilizarlo para las finalidades indicadas. No significa que cualquier persona pueda verlo. «Cifrado pero recuperable» significa que la base no conserva el valor legible, aunque la API puede descifrarlo para una operación autorizada. «No reversible» significa que el servidor puede comprobar el dato, pero no reconstruir su valor original. «Vault opaco» significa que el servidor sólo conserva ciphertext y no puede interpretar su contenido. «Transitorio o local» significa que el dato no se incorpora a la estructura durable legible de la API.

| Tratamiento | Dato exacto | Nombre canónico | Regla de la beta |
|---|---|---|---|
| API legible | Identificador de la cuenta | `id` | Lo genera MDL, es inmutable y vincula los datos que pertenecen a la cuenta. |
| API legible | Fecha completa de nacimiento declarada | `birth_date` | Es obligatoria, se conserva como fecha civil completa y permite demostrar qué fecha declaró la persona al darse de alta. La mayoría de edad se calcula desde esta fecha cuando corresponde validarla. |
| API legible | Género de la persona titular | `gender_id` | Es opcional, pertenece al catálogo de género y puede añadirse, cambiarse o quitarse desde el perfil. |
| API legible | País de la persona titular | `country_id` | Es opcional, pertenece al catálogo de países y puede añadirse, cambiarse o quitarse desde el perfil. |
| API legible | Plan o nivel de acceso | `plan_id` | Lo asigna el sistema. La persona no puede modificarlo desde el perfil. La beta se rige por RP-BASE-004. |
| API legible | Verificación del email | `email_verified_at` | La ausencia de fecha identifica una cuenta por email todavía no verificada. Sólo el flujo de verificación puede fijarla. |
| API legible | Vencimiento de la cuenta provisional | `provisional_expires_at` | Se fija desde el alta y no se prolonga por login, reenvíos ni nuevos códigos. |
| API legible | Fecha de alta | `created_at` | Deja constancia de cuándo se creó la cuenta y no es editable por la persona. Junto con `birth_date`, conserva la declaración utilizada para comprobar la mayoría de edad. |
| Administrado por el sistema | Última modificación de la cuenta | `updated_at` | Registra una modificación durable de la cuenta. No sustituye la fecha funcional del cambio ni es editable desde el perfil. |
| API legible | Última autenticación correcta | `last_login` | Sólo cambia después de una autenticación correcta. Un intento fallido no la modifica. |
| API legible | Aviso de inactividad | `inactivity_notified_at` | Sólo registra el aviso que inicia el plazo de gracia previsto para la baja por inactividad. |
| API legible | Baja de la cuenta | `deleted_at` | Indica que la cuenta dejó de estar disponible. No es un campo editable del perfil ni sustituye las reglas completas de baja y seudonimización. |
| API legible | Tutoriales informativos completados | `is_tutorial_done`, `is_profile_tutorial_done` | Sólo registran si cada tutorial correspondiente ya fue completado. No conceden ni retiran acceso a datos privados. |
| Cifrado pero recuperable por API | Email vigente | `email` | Es obligatorio y se conserva cifrado. La API puede recuperarlo únicamente para acceso, verificación, recuperación, cambio de email y comunicaciones autorizadas de la cuenta. |
| Derivado protegido | Índice exacto del email normalizado | `email_lookup` | Permite búsqueda y unicidad sin guardar otra copia legible. No se devuelve al cliente ni puede sustituir el cifrado de `email`. |
| No reversible | Verificador de contraseña MDL | `password` | Es opcional mientras un alta por email siga provisional y para una cuenta creada con Apple o Google. Contiene únicamente un verificador no reversible, nunca la contraseña legible. |
| No reversible y temporal | Verificador candidato del alta | intento de registro | Cada iniciación válida conserva como máximo hasta su vencimiento el verificador de la contraseña propuesta, ligado a su capacidad y OTP. No permite login ni sustituye `password` hasta que ese mismo intento verifique correctamente. |
| Registros vinculados | Métodos Apple y Google, capacidades/intentos de registro, sesiones, verificaciones, recuperaciones, cambios de email y aceptaciones legales | Registros propios del ámbito correspondiente | Sólo conservan los estados y evidencias exigidos por sus reglas. No amplían el perfil ni autorizan copias legibles adicionales del email. |
| Transitorio | Contraseña legible, OTP introducido y credenciales temporales de Apple o Google | Sin campo durable legible | Sólo existen durante la operación autorizada y no se incorporan al perfil ni a logs o diagnósticos. |
| Sólo dispositivo | Idioma de la aplicación | Preferencia local | No forma parte de la identidad de la cuenta ni del Vault. Se conserva según RP-PER-003. |

- **RP-CUE-006.** El perfil personal de la cuenta está formado por `birth_date`, `gender_id` y `country_id`. El email, la contraseña y los métodos de acceso se gestionan en sus flujos específicos; los estados del sistema se administran conforme al inventario anterior.
- **RP-CUE-007.** Crushes, relaciones, experiencias, fotos y contenido del Vault son entidades del diario vinculadas a la cuenta y conservan sus propios contratos.

<a id="registro"></a>
## Registro e inicio de sesión

- **BR-3.01.** El registro por email exige exactamente: email, contraseña, fecha completa de nacimiento, aceptación de la versión vigente de los términos y aceptación de la versión vigente de privacidad. Los registros con Apple y Google se rigen por [Acceso con Apple](#apple) y [Acceso con Google](#google), respectivamente.
- **BR-3.02.** Todo registro exige aceptar explícitamente las versiones vigentes de términos y privacidad. Una versión ausente o desactualizada impide completar el alta.
- **BR-3.03.** El email identifica la cuenta sin distinguir mayúsculas y minúsculas.
- **BR-3.04.** La contraseña MDL tiene al menos ocho caracteres e incluye, como mínimo, una letra mayúscula, una letra minúscula, un dígito y uno de estos caracteres especiales: `@`, `$`, `!`, `%`, `*`, `#`, `?`, `&`. La aplicación informa esta misma lista de caracteres especiales.
- **BR-3.05.** La sesión vence **siete días desde la emisión del token**, sin renovación por actividad. Logout o revocación pueden terminarla antes.
- **RP-CUE-001.** El registro pide la fecha completa de nacimiento, también con Apple o Google. MDL se basa en la fecha declarada por la persona y rechaza el alta si según esa fecha todavía no ha cumplido 18 años. Conserva la fecha civil completa en `birth_date` y la fecha del alta en `created_at` como constancia de la declaración realizada.
- **RP-CUE-002.** Un login fallido no modifica la cuenta, la última autenticación ni el plazo de cierre por inactividad.
- **RP-CUE-003.** Login y restauración resuelven el estado de acceso antes de mostrar datos privados. Una sesión inexistente, vencida o revocada no habilita el diario.
- **RP-CUE-004.** Logout retira acceso y contexto privado en memoria de esa sesión; no elimina cuenta ni Vault remoto.
- **RP-CUE-008.** La contraseña legible sólo existe mientras la persona la introduce para registrarse, entrar, cambiarla o recuperarla. MDL conserva únicamente un verificador no reversible de la contraseña. Ninguna pantalla devuelve ni muestra la contraseña existente.
- **RP-CUE-010.** Iniciar un alta por email no revela si la dirección está disponible, corresponde a una provisional o ya pertenece a una cuenta verificada. Todos los casos devuelven el mismo status, envelope, código, cuerpo, cabeceras y una capacidad opaca de registro con la misma forma y vigencia. Esa capacidad no es una sesión de cuenta, no autentica y no incluye identificadores observables. Una cuenta verificada recibe una capacidad señuelo sin estado durable, sin OTP y sin mutación; una dirección sin cuenta verificada crea o reutiliza sólo la provisional necesaria y un intento independiente al que se liga su OTP. El trabajo criptográfico y los límites públicos no omiten de forma observable ninguna rama.
- **RP-CUE-011.** Cada intento de alta por email liga de forma inseparable su capacidad opaca, OTP, verificador candidato, fecha de nacimiento declarada y versiones legales aceptadas. Presentar un OTP sin la capacidad correspondiente —o una capacidad con el OTP de otro intento— nunca completa el alta. Varios intentos sobre la misma provisional no se sobrescriben: sólo el par capacidad+OTP que gana bajo lock fija esos datos en la cuenta, invalida los demás intentos y emite la primera sesión. Así, recibir un código no activa una contraseña elegida por otra persona.
- **RP-CUE-012.** Una cuenta por email que todavía no está verificada no puede iniciar sesión ni vincular Apple o Google mediante email y contraseña. Aun con la contraseña candidata correcta, login y vinculación responden con el mismo `401 login_failed` que una credencial inválida, sin sesión, vínculo, correo ni mutación. La persona continúa únicamente con la capacidad de registro y sus operaciones de verificación.

<a id="verificacion-email"></a>
## Verificación obligatoria por email

- **BR-3.12.** Una iniciación de alta por email devuelve una capacidad opaca de registro, nunca una sesión autenticada. La capacidad sólo puede consultar el estado genérico del intento, reenviar dentro de sus límites, verificar o cancelarse; no autoriza login, Home, Settings, diario, perfil, vinculación federada ni configuración de Vault. La capacidad señuelo de una dirección ya verificada ofrece las mismas respuestas públicas sin tocar esa cuenta.
- **RP-VER-001.** Cada intento real envía automáticamente un OTP de seis dígitos ligado a su capacidad. Un fallo temporal de envío no convierte la cuenta en verificada ni impide reintentar dentro de ese mismo intento.
- **RP-VER-002.** El OTP dura 15 minutos, admite cinco intentos y bloqueo de 15 minutos dentro de su intento. Reenvío sustituye sólo el OTP de esa capacidad con cooldown progresivo; no invalida ni adopta los datos de otro intento.
- **BR-3.13.** La provisional y cada capacidad real vencen como máximo a los **siete días fijos desde la primera alta**. Login, nuevas iniciaciones, reenvíos y códigos nuevos no prolongan el deadline de la cuenta provisional.
- **BR-3.14.** Sin verificar no hay uso funcional desde UI ni API. Verificar correctamente capacidad y OTP del mismo intento completa atómicamente el alta con los datos de ese intento, invalida los demás y entrega la primera sesión sin exigir otro login.
- **RP-VER-003.** Una cuenta provisional por email no permite acceder a la aplicación, iniciar sesión ni añadir Apple o Google hasta verificar su intento. El cliente conserva la capacidad de registro —separada de cualquier sesión de cuenta— al cerrarse y reabrirse para poder continuar o cancelar.
- **BR-3.15.** Vencidos siete días sin verificar se eliminan íntegramente la provisional, sus intentos, capacidades y OTP, se libera el email y no se conserva como lead ni destinatario comercial.
- **RP-VER-005.** Una cuenta creada con Apple o Google no requiere un código de verificación de email de MDL cuando el proveedor acredita que el email está verificado.
- **RP-VER-006.** Durante la beta, los correos que transportan un OTP de MDL se envían en español, independientemente del idioma elegido en la aplicación. Localizarlos por preferencia de la cuenta o del dispositivo queda fuera del alcance de la beta y no se infiere desde locale, IP ni contenido de la cuenta.

<a id="apple"></a>
## Acceso con Apple

- **RP-APL-001.** «Continuar con Apple» permite crear una cuenta MDL o entrar en una cuenta existente vinculada a Apple.
- **RP-APL-002.** El acceso con Apple exige que la persona autorice a MDL mediante su cuenta Apple.
- **RP-APL-003.** Una cuenta Apple sólo puede estar vinculada a una cuenta MDL. Una cuenta MDL sólo puede tener una cuenta Apple vinculada.
- **RP-APL-004.** Si la autorización con Apple falla, no se crea una cuenta, no se añade Apple como método de acceso y no se inicia sesión.

### Crear una cuenta con Apple

- **RP-APL-005.** El alta con Apple exige declarar la fecha completa de nacimiento, tener al menos 18 años y aceptar los términos y la política de privacidad vigentes. La fecha declarada se conserva conforme a RP-CUE-001. No exige crear una contraseña MDL.
- **RP-APL-006.** El alta utiliza el email verificado que entrega Apple, sin pedir que la persona lo escriba. Si elige compartirlo, se guarda la dirección personal seleccionada; si elige ocultarlo, se guarda la dirección intermediaria de Apple.
- **RP-APL-008.** Crear una cuenta o iniciar sesión con Apple requiere que Apple proporcione un email válido. Un email escrito manualmente no sustituye ese requisito.
- **RP-APL-009.** MDL no solicita ni guarda el nombre proporcionado por Apple.
- **RP-APL-010.** Después de autorizar Apple, la persona dispone de diez minutos para completar el alta. Vencido ese plazo, debe autorizar Apple de nuevo. La autorización sólo permite completar un alta.
- **RP-APL-011.** El alta se completa cuando quedan confirmados la cuenta, el acceso con Apple, las aceptaciones legales y el inicio de sesión. Si falla alguna parte, no queda una cuenta creada parcialmente.
- **RP-APL-012.** Cancelar el acceso con Apple conserva los datos introducidos en el formulario y no crea una cuenta, una vinculación ni una sesión. La cancelación no se presenta como un error.
- **RP-APL-014.** Antes de confirmar el alta, la aplicación informa que utilizará el email entregado por Apple.
- **RP-APL-024.** El correo intermediario de Apple se admite para la recuperación de acceso y los avisos de la cuenta. No se exige sustituirlo por un correo personal.

<a id="metodos-acceso-apple"></a>
### Añadir otro método de acceso

- **RP-APL-017.** Una misma cuenta MDL puede utilizar Apple y email con contraseña. Añadir un método de acceso conserva la cuenta y todos sus datos.
- **RP-APL-018.** Vincular Apple exige acreditar el acceso a la cuenta MDL y a la cuenta Apple, y confirmar expresamente la vinculación. La coincidencia de emails no vincula ni fusiona cuentas automáticamente.

<a id="apple-a-email"></a>
### Añadir email y contraseña a una cuenta creada con Apple

- **RP-APL-015.** Una cuenta creada con Apple que todavía no tenga contraseña puede añadir el acceso mediante email y contraseña desde los ajustes de una sesión MDL válida. La sesión acredita la cuenta actual y no se exige volver a autorizar Apple. Si no dispone de una sesión válida, puede establecer la primera contraseña mediante la recuperación enviada al email vigente y verificado, pero ese flujo de recuperación no permite cambiar el email.
- **RP-APL-016.** Al añadir este acceso, la persona puede conservar el email recibido de Apple o elegir otro. MDL envía siempre un código al email elegido y exige confirmarlo, incluso cuando se conserva exactamente la dirección personal o intermediaria recibida de Apple.
- **RP-APL-026.** Cambiar o verificar el email mediante el flujo general de cambio de email no crea una contraseña. Añadir el acceso mediante email y contraseña es una acción distinta que debe confirmarse expresamente.
- **RP-APL-032.** Después de verificar el email elegido y aceptar una contraseña que cumpla BR-3.04, MDL habilita atómicamente ambos datos como método de acceso de la misma cuenta. La operación conserva Apple, el identificador de la cuenta, el Vault y el diario; si falla cualquier parte, no deja cambiado el email ni creada la contraseña. Al completarse, se cierran todas las sesiones conforme a RP-OTP-007.

<a id="email-a-apple"></a>
### Vincular Apple a una cuenta MDL existente

- **RP-APL-027.** Si Apple comparte el mismo email que una cuenta MDL existente, la aplicación ofrece vincular Apple a esa cuenta. La coincidencia no concede acceso por sí sola.
- **RP-APL-028.** Si Apple entrega un correo intermediario o diferente, la aplicación ofrece «Ya tengo una cuenta MDL» para identificar la cuenta existente y vincularla.
- **RP-APL-029.** La vinculación desde «Continuar con Apple» requiere autorizar Apple, acreditar el acceso a la cuenta MDL con su email y contraseña y confirmar la vinculación. Si la persona no recuerda su contraseña, debe recuperarla antes de completar la vinculación.
- **RP-APL-030.** La vinculación también puede iniciarse desde los ajustes de una sesión MDL abierta. La persona autoriza Apple y confirma que quiere añadirlo a esa cuenta.
- **RP-APL-031.** Vincular Apple conserva el email MDL y su estado de verificación, pero sólo puede hacerlo una cuenta MDL ya verificada. La vinculación desde «Continuar con Apple» mediante email y contraseña responde `401 login_failed` para una provisional igual que para credenciales inválidas; la vinculación desde ajustes exige una sesión de cuenta válida y verificada. Ninguna ruta federada convierte una capacidad de registro en sesión ni completa el alta por email.
- **RP-APL-033.** Si la identidad Apple ya está vinculada a otra cuenta MDL, la vinculación se rechaza completa y ninguna de las dos cuentas cambia. MDL no fusiona, absorbe ni transfiere automáticamente cuentas, aunque alguna parezca vacía. La persona debe utilizar la cuenta que ya tiene Apple o eliminar la que no quiera conservar; Apple sólo puede añadirse a otra cuenta después de que el servidor confirme que la identidad ha quedado disponible.

### Pérdida de autorización o indisponibilidad de Apple

- **RP-APL-020.** Si se revoca la autorización de Apple o deja de ser válida, se impide el acceso con Apple y se cierran las sesiones MDL afectadas. La cuenta y sus datos se conservan.
- **RP-APL-023.** Si el acceso con Apple no está disponible, la aplicación informa del problema y permite reintentar cuando se restablezca. No comunica que se ha iniciado sesión si el acceso ha fallado.
- **RP-APL-034.** «Continuar con Apple» se ofrece en iOS durante la beta activa, distribuida exclusivamente mediante iOS y API. Android queda fuera de esa beta y no muestra Apple cuando se habilite posteriormente. Una cuenta creada con Apple puede establecer su primera contraseña mediante el email vigente y verificado conforme a RP-APL-015 y, una vez que Android supere sus gates propios, acceder desde Android a la misma cuenta y al mismo Vault.

<a id="google"></a>
## Acceso con Google

- **RP-GGL-001.** «Continuar con Google» permite crear una cuenta MDL o entrar en una cuenta existente vinculada a Google.
- **RP-GGL-002.** «Continuar con Google» queda fuera de la beta activa iOS/API e iOS no lo muestra. Se reserva para una distribución Android posterior, en la que Android ofrecerá email y contraseña además de Google; la beta iOS ofrece email y contraseña además de Apple.
- **RP-GGL-003.** Cuando Android se habilite, obtendrá la autorización mediante el mecanismo nativo vigente de Google, pero sólo la API de MDL podrá validar la identidad y decidir si crea, vincula o abre una cuenta. Una sesión local de Google no equivale a una sesión MDL.
- **RP-GGL-004.** La API valida como mínimo firma, emisor, audiencia, vigencia y nonce de la credencial Google. Una credencial inválida, vencida, reutilizada o destinada a otro cliente se rechaza sin crear cuenta, vínculo ni sesión.
- **RP-GGL-005.** Una identidad Google sólo puede estar vinculada a una cuenta MDL. Una cuenta MDL sólo puede tener una identidad Google vinculada.
- **RP-GGL-006.** El alta con Google exige un email válido acreditado como verificado por Google, declarar la fecha completa de nacimiento, tener al menos 18 años y aceptar los términos y la política de privacidad vigentes. No exige crear una contraseña MDL.
- **RP-GGL-007.** MDL no solicita ni incorpora al perfil el nombre, la foto ni otros datos de perfil entregados por Google. Conserva únicamente la identidad, el email acreditado y la evidencia técnica necesaria para autenticar y gestionar el vínculo conforme al inventario de la cuenta.
- **RP-GGL-008.** El alta se completa cuando quedan confirmados atómicamente la cuenta, el acceso con Google, las aceptaciones legales y el inicio de sesión. Si falla alguna parte, no queda una cuenta creada parcialmente.
- **RP-GGL-009.** Cancelar la autorización Google conserva los datos introducidos en el formulario y no crea una cuenta, una vinculación ni una sesión. La cancelación no se presenta como un error.
- **RP-GGL-010.** Si el email acreditado por Google ya pertenece a una cuenta MDL, la coincidencia no concede acceso, no vincula y no fusiona cuentas automáticamente.
- **RP-GGL-011.** Vincular Google a una cuenta existente exige acreditar expresamente tanto la identidad Google como el acceso a la cuenta MDL y confirmar la vinculación. Puede iniciarse desde «Continuar con Google» acreditando email y contraseña MDL o desde los ajustes de una sesión MDL válida.
- **RP-GGL-012.** Si la identidad Google ya está vinculada a otra cuenta MDL, la operación se rechaza completa y ninguna de las dos cuentas cambia. MDL no fusiona, absorbe ni transfiere automáticamente cuentas.
- **RP-GGL-013.** Login, alta y vinculación Google utilizan el mismo `User`, la misma sesión MDL y el mismo namespace/Vault que el acceso por email y contraseña; no existe una cuenta ni un Vault paralelos por proveedor.
- **RP-GGL-014.** Una cuenta creada con Google que todavía no tenga contraseña puede añadir el acceso mediante email y contraseña desde los ajustes de una sesión MDL válida o mediante la recuperación enviada a su email vigente y verificado. La operación conserva Google, el identificador de la cuenta, el Vault y el diario y se rige por las mismas garantías atómicas de RP-APL-015, RP-APL-016, RP-APL-026 y RP-APL-032.
- **RP-GGL-015.** Una vez añadida la contraseña, la cuenta creada con Google puede acceder desde iOS mediante email y contraseña a la misma cuenta y al mismo Vault. iOS no necesita mostrar Google para permitir ese acceso.
- **RP-GGL-016.** Una cuenta MDL puede conservar Apple y Google como métodos de acceso sólo si cada identidad se vinculó expresamente a esa misma cuenta. La coincidencia de emails nunca crea el segundo vínculo.
- **RP-GGL-017.** Si se revoca la autorización Google o deja de ser válida, se impide el acceso con Google y se cierran las sesiones MDL afectadas. La cuenta, el Vault y el diario no se eliminan por ese hecho.
- **RP-GGL-018.** Si Google no está disponible, Android informa del problema y permite reintentar. No comunica que se ha iniciado sesión ni crea una sesión MDL ficticia.
- **RP-GGL-019.** El token de identidad, el nonce y cualquier credencial Google son transitorios: no se incorporan al perfil, al Vault, a logs ni a diagnósticos. Sólo se transportan y procesan durante la operación autorizada.
- **RP-GGL-020.** «Continuar con Google» no se habilita públicamente hasta que estén implementados y comprobados la validación server-side, la prevención de duplicados, la vinculación explícita, la revocación, la integración con sesión/Vault y los requisitos de configuración, marca, Google Play y revisión humana/legal.
- **RP-GGL-021.** Vincular Google conserva el email MDL y su estado de verificación, pero sólo puede hacerlo una cuenta MDL ya verificada. La vinculación desde «Continuar con Google» mediante email y contraseña responde `401 login_failed` para una provisional igual que para credenciales inválidas; la vinculación desde ajustes exige una sesión de cuenta válida y verificada. Ninguna ruta federada convierte una capacidad de registro en sesión ni completa el alta por email.

<a id="perfil-cuenta"></a>
## Perfil de la cuenta de usuario y ajustes

- **RP-PER-001.** La edición del perfil muestra los datos actuales. Guardar conserva los campos que la persona no haya modificado; borrar un dato requiere que lo elimine expresamente.
- **RP-PER-002.** El perfil muestra la fecha completa de nacimiento declarada, sin presentar una edad exacta calculada. Para corregirla, la persona declara una nueva fecha completa; MDL vuelve a comprobar que tenga al menos 18 años y, si la corrección es válida, sustituye `birth_date` sin alterar `created_at`.
- **RP-PER-003.** La beta permite elegir español o inglés. Cambiar de idioma actualiza la aplicación y conserva la elección al cerrarla y volver a abrirla.
- **RP-PER-004.** Editar los datos del perfil no cambia el email, la contraseña ni los permisos de la cuenta.
- **RP-PER-007.** El perfil de la cuenta incluye género y país. Ambos se conservan mediante sus IDs de catálogo en la base de datos de la API, fuera del Vault.
- **RP-PER-008.** La información personal que MDL solicita y conserva para el perfil de la cuenta es la fecha completa de nacimiento y, cuando la persona los informa, género y país.
- **RP-PER-009.** Los únicos campos personales editables desde el perfil de la cuenta son `birth_date`, `gender_id` y `country_id`. Género y país son opcionales; la fecha de nacimiento es obligatoria y sólo se corrige conforme a RP-PER-002. El email se muestra y cambia únicamente mediante [Cambio de email](#cambio-email); la contraseña se gestiona únicamente mediante su flujo específico; el idioma es una preferencia de la aplicación.
- **RP-PER-010.** La lectura del perfil presenta `birthDate`, `genderId` y `countryId` como sus campos personales. Los estados de verificación, acceso o plan se presentan únicamente en las superficies que los necesitan y permanecen administrados por el sistema.

<a id="datos-cuenta"></a>
## Protección del email de la cuenta

- **RP-VLT-005.** El email de la cuenta se conserva cifrado en el servidor, fuera del Vault. MDL puede utilizarlo para el acceso, la verificación, la recuperación y las comunicaciones de la cuenta. Esta protección también se aplica al email pendiente de verificación o cambio y al recibido de Apple o Google.
- **RP-CUE-009.** Las copias de email sujetas a RP-VLT-005 son exactamente: el email vigente de la cuenta; el email de una cuenta provisional; la nueva dirección pendiente de un cambio; la dirección asociada a códigos de verificación o recuperación; y el email acreditado recibido de Apple o Google. Colas, reintentos y envíos pueden transportarlo sólo durante la operación autorizada, sin dejar una copia durable legible ni incluirlo en logs, diagnósticos o mensajes internos de error.

<a id="cambio-email"></a>
## Cambio de email

- **RP-MAIL-001.** Para cambiar el email de la cuenta, el email actual debe estar verificado.
- **RP-MAIL-002.** La persona verifica la nueva dirección con un código enviado a ese correo. La dirección anterior permanece activa hasta completar la verificación.
- **RP-MAIL-003.** Al confirmar el código, la nueva dirección pasa a ser el email verificado de la cuenta. Si la verificación falla, se conserva la dirección anterior.
- **RP-MAIL-004.** Cambiar el email de MDL conserva los accesos Apple y Google vinculados.
- **RP-MAIL-005.** Si la nueva dirección ya pertenece a otra cuenta MDL, el cambio se rechaza completo y ninguna de las dos cuentas cambia. MDL no fusiona, absorbe ni transfiere automáticamente cuentas, aunque alguna parezca vacía. La persona debe utilizar la cuenta que ya tiene ese email o eliminar la que no quiera conservar; el email sólo puede añadirse a otra cuenta después de que el servidor confirme que ha quedado disponible.
- **RP-MAIL-006.** Solicitar un cambio de email devuelve la misma respuesta pública cuando la dirección destino está disponible o ya pertenece a otra cuenta. Sólo una dirección disponible crea el cambio pendiente y recibe el OTP; una dirección ocupada no recibe código y no modifica ninguna cuenta. Sin un OTP válido, los pasos posteriores tampoco revelan cuál de esas ramas ocurrió. Esta indistinguibilidad no altera RP-MAIL-005: la confirmación sigue comprobando la unicidad bajo la frontera atómica y nunca fusiona ni transfiere cuentas.

<a id="contrasena-acceso"></a>
## Cambio de la contraseña de acceso

- **RP-PWD-001.** Una cuenta que ya tenga contraseña MDL puede cambiarla desde los ajustes de una sesión válida. La persona introduce la contraseña actual, la nueva contraseña y su confirmación; la nueva contraseña debe cumplir BR-3.04.
- **RP-PWD-002.** Una contraseña actual incorrecta, una confirmación distinta o una nueva contraseña inválida rechazan la operación completa sin modificar la cuenta, la contraseña ni las sesiones.
- **RP-PWD-003.** Un cambio correcto sustituye únicamente el verificador no reversible de la contraseña. Conserva el email y su verificación, los accesos Apple y Google, el identificador de la cuenta, el Vault y el diario; después cierra todas las sesiones conforme a RP-OTP-007.
- **RP-PWD-004.** Una cuenta creada con Apple o Google que todavía no tenga contraseña no utiliza este flujo de cambio. Añade su primer acceso mediante email y contraseña conforme a RP-APL-015, RP-APL-016, RP-APL-026 y RP-APL-032 o RP-GGL-014, según corresponda.

<a id="recuperacion-acceso"></a>
## Recuperación de la contraseña de acceso

- **BR-3.06.** La recuperación utiliza un código de seis dígitos enviado al email de la cuenta. La aplicación muestra la misma respuesta al solicitarlo, exista o no una cuenta con esa dirección; sólo envía el código si la cuenta existe.
- **RP-OTP-001.** El código de recuperación vence a los 15 minutos y sólo puede utilizarse una vez.
- **BR-3.07.** La espera necesaria para reenviar el código aumenta en tres escalones.
- **BR-3.08.** Se admiten como máximo cinco intentos fallidos sobre un mismo código de recuperación, compartidos por su verificación y por la revalidación que confirma el reset. Verificar correctamente no reinicia ese presupuesto. Alcanzado el límite, el código queda bloqueado durante 15 minutos y tampoco puede completar el reset aunque después se presente correctamente.
- **BR-3.16.** Un código incorrecto, vencido o bloqueado muestra el mismo mensaje de error.
- **RP-OTP-005.** Recuperar la contraseña de acceso a MDL no recupera ni reinicia el Vault. La contraseña de acceso y los medios de recuperación del Vault son independientes.
- **RP-OTP-007.** Al establecer por primera vez, cambiar o recuperar la contraseña de acceso a MDL, se cierran todas las sesiones activas de esa cuenta, incluida la sesión actual. La persona debe volver a iniciar sesión.

<a id="bloqueo-local"></a>
## Bloqueo local de la aplicación

- **RP-LOCK-001.** El bloqueo local protege todas las pantallas privadas al volver a la aplicación desde segundo plano, incluidas las que estuvieran abiertas sobre otra pantalla.
- **RP-LOCK-002.** El bloqueo local no cierra ni prolonga la sesión. Una sesión vencida exige iniciar sesión de nuevo; desbloquear la aplicación no la reactiva ni recupera el Vault.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

No hay decisiones funcionales pendientes en este capítulo.
