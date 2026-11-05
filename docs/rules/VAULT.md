# Reglas de producto — Vault

<a id="almacenamiento-mysql"></a><a id="introduccion-copy"></a><a id="ayuda-copy-aprobado"></a><a id="clave-copy"></a><a id="frase-copy"></a><a id="preparacion-copy"></a><a id="recuperacion-copy"></a><a id="recuperacion-progreso-copy"></a><a id="recuperacion-errores-copy"></a><a id="revision-editorial-es"></a>

[Índice de reglas](../PRODUCT.md)

El Vault conserva el contenido privado cifrado y permite recuperarlo en un dispositivo autorizado.

<a id="contenido"></a>
## Contenido y privacidad

- **RP-VLT-001.** El contenido funcional privado de la beta está formado exactamente por los campos del perfil enumerados como «Vault opaco» en el [inventario del Crush](CRUSH.md#inventario) y por `place`, `description` y la reflexión opcional `reflection` de cada experiencia. Cada campo pertenece al elemento identificado en su documento de dominio.
- **RP-VLT-002.** La API funcional no acepta, devuelve ni conserva una copia en claro de los datos privados. Las aplicaciones los muestran únicamente después de acceder al Vault.
- **RP-VLT-003.** El dispositivo cifra cada payload privado antes de transferirlo y conserva localmente su entry cifrada. El Vault remoto recibe únicamente ciphertext y la metadata mínima necesaria como copia cifrada para sincronización y recuperación. La API sólo interpreta la estructura enumerada en los inventarios de [Cuenta](ACCOUNT.md#inventario-cuenta), [Crush](CRUSH.md#inventario), [Experiencias](EXPERIENCES.md#inventario) y [Relaciones](RELATIONSHIPS.md#inventario); el backend y el almacenamiento remoto no pueden interpretar el contenido privado. Ese contenido sólo se recupera en claro dentro de la memoria de un dispositivo autorizado después de desbloquear o recuperar el mismo Vault.
- **RP-VLT-026.** Los inventarios de dominio son listas cerradas y sus tratamientos son excluyentes. Cada payload Vault contiene únicamente los campos marcados como «Vault opaco» para ese tipo de entidad; ningún campo clasificado como «API legible», «Administrado por el sistema», «Derivado por la API», «Sólo dispositivo» o equivalente se copia dentro del payload cifrado. El payload Vault de una experiencia contiene exactamente `place`, `description` y `reflection`; satisfacción y las respuestas estructuradas de RP-EXP-029 no se duplican en él. Un período o hito relacional no tiene payload Vault propio: cualquier contenido privado que explique ese acontecimiento pertenece a su experiencia vinculada. Las fotos continúan siendo «Sólo dispositivo»: su cifrado local usa una clave dedicada derivada de la VMK conforme a `RP-FOT-010`, pero no crea una entry, binding, blob, cuota ni copia recuperable en el Vault remoto.
- **RP-VLT-004.** La clave principal del Vault se genera en el dispositivo y nunca llega en claro al servidor. Soporte nunca solicita esa clave ni los secretos de recuperación.

<a id="inventario-servidor"></a>
## Información disponible para el servidor

- **RP-VLT-024.** El servidor conserva exclusivamente información necesaria para identificar propietario, contenido cifrado, versión, integridad, recuperación, dispositivo autorizado y ciclo de vida.
- **RP-VLT-025.** El contenido cifrado y su información de gestión permanecen separados del significado privado de nombres, contactos, cumpleaños, lugares y relatos.
- **RP-VLT-027.** Cada objeto de ciphertext del Vault remoto admite como máximo 65.536 bytes. El dispositivo valida el tamaño después de cifrar y antes de transferir; la API vuelve a validarlo antes de autorizar o confirmar la subida. Superar el límite rechaza la operación sin crear una versión confirmada y se comunica de forma comprensible. Las fotos no forman parte de ese inventario remoto: su límite local independiente se define en `RP-FOT-008`.
- **RP-VLT-028.** Cada cuenta dispone de 25 MiB —26.214.400 bytes— de capacidad lógica total para su Vault remoto. La capacidad ocupada suma la última versión confirmada de cada elemento y el crecimiento reservado por subidas todavía no confirmadas; una revisión sustituida, retirada o en purga no consume nuevamente cuota. Una subida nueva reserva todo su tamaño declarado y una sustitución reserva únicamente el aumento respecto de la versión vigente. Reintentar la misma operación no duplica la reserva.
- **RP-VLT-029.** Al alcanzar 20 MiB —el 80 % de la capacidad— la aplicación avisa de forma comprensible cuánto espacio está ocupado y cuál es el límite. Alcanzar la cuota sólo impide operaciones que aumenten la ocupación: siempre permite eliminar contenido y sustituirlo por otro de tamaño igual o menor. Un rechazo conserva íntegramente la versión vigente y explica cómo liberar espacio.
- **RP-VLT-030.** La API controla capacidad, errores y abuso mediante la metadata autorizada del Vault: bytes confirmados o reservados, cantidad y antigüedad de subidas incompletas, ritmo de operaciones, fallos de verificación y acumulación anómala de revisiones. Puede aplicar alertas y límites de frecuencia sobre esas señales sin descifrar, interpretar ni clasificar el contenido privado. La cantidad de Crushes o experiencias, por sí sola, no constituye abuso.
- **RP-VLT-031.** Una capacidad de subida que no se confirma ni se cancela caduca 24 horas después de su creación. Desde ese momento no puede confirmarse, deja de reservar cuota y la limpieza automática retira cualquier objeto y metadata incompletos. La caducidad no altera la última versión confirmada ni elimina la operación pendiente del dispositivo; un reintento posterior solicita una capacidad nueva para esa misma intención durable.

<a id="recuperacion"></a>
## Creación, revisión y recuperación

- **RP-VER-004.** La configuración del Vault sólo comienza cuando la cuenta está habilitada. Si ya existe un Vault, se utiliza o recupera el existente; no se ofrece crear otro.
- **RP-VLT-007.** Cada elemento privado se conserva como una unidad cifrada completa, no como un archivo separado por cada campo.
- **RP-VLT-008.** Editar un elemento conserva su identidad y reemplaza su contenido actual sólo después de confirmar la nueva versión. El producto no muestra un historial de versiones del Vault.
- **RP-VLT-009.** Cada contenido cifrado permanece unido inequívocamente a su cuenta y a su elemento. Si esa correspondencia falta o se contradice, el cambio se rechaza.
- **RP-VLT-010.** Una nueva versión cifrada se confirma antes de retirar la anterior. Reintentar conserva la misma identidad y el mismo contenido propuesto.
- **RP-VLT-011.** Si el dispositivo no tiene un Vault local, comprueba si existe uno recuperable. Un Vault remoto existente conduce a recuperación y nunca a crear otro que lo sobrescriba.
- **RP-VLT-012.** La necesidad de recuperar o terminar de crear el Vault se conserva al cerrar o suspender la aplicación y sólo desaparece cuando el proceso termina correctamente.
- **RP-VLT-013.** La contraseña del Vault y la frase de 24 palabras son dos mecanismos independientes para recuperar la misma información.
- **RP-VLT-014.** La recuperación descarga y valida todo el inventario antes de reemplazar el Vault local. No borra el contenido remoto ni anuncia éxito con una recuperación parcial.
- **RP-VLT-015.** Durante la beta sólo puede existir un dispositivo activo por Vault. Recuperarlo en otro dispositivo sustituye al anterior; no existe sincronización continua entre dos copias activas.
- **RP-VLT-016.** La limitación de un dispositivo no se presenta como revocación remota automática mientras esa capacidad no exista.
- **RP-VLT-017.** Perder la contraseña del Vault y la frase de recuperación puede volver irrecuperable el contenido privado. Recuperar el acceso a la cuenta no recupera el Vault.
- **RP-VLT-018.** Cada cuenta tiene un único Vault durante toda su vida. No puede reiniciarse o recrearse de manera independiente.
- **RP-VLT-032.** La recuperación del Vault utiliza exclusivamente la clave de recuperación o la frase de 24 palabras.
- **RP-VLT-033.** La frase de recuperación contiene exactamente 24 palabras inglesas generadas aleatoriamente en el dispositivo. Permite recuperar la misma clave principal del diario mediante el perfil criptográfico interoperable definido en la documentación técnica de API y clientes.
- **RP-VLT-034.** Los intentos fallidos y las esperas de recuperación pertenecen al Vault de una cuenta y al mecanismo utilizado. Cambiar de cuenta conserva separados ambos presupuestos; un éxito reinicia únicamente el mecanismo del mismo Vault. No existe un presupuesto de recuperación global del dispositivo compartido entre cuentas.
- **RP-VLT-035.** Después de autenticar una cuenta cuyo Vault aún no terminó de configurarse, la aplicación muestra obligatoriamente ese flujo y no permite usar las superficies funcionales hasta completarlo. Si se abandona antes de confirmar que la frase de recuperación fue guardada, el siguiente acceso reinicia la configuración desde el primer paso. Si la frase ya fue confirmada y sólo queda pendiente la migración final, el siguiente acceso recupera la VMK conforme a la protección local y retoma desde el paso de migración, sin repetir la contraseña ni volver a mostrar la frase.
- **RP-VLT-036.** Consultar ejemplos ficticios conserva separado el Vault real: mantiene su configuración, recuperación, dispositivo activo y estado de bloqueo, sin acceder a secretos ni contenido privado.

<a id="informacion"></a>
## Información de protección y recuperación

- **RP-VLT-037.** La explicación de protección identifica qué contenido se cifra y qué estructura puede interpretar la API; distingue acceso a la cuenta, desbloqueo local y recuperación del diario. Explica los dos medios de recuperación, las consecuencias de perderlos, el dispositivo único y las fotografías locales sin copia remota. La ayuda está disponible desde configuración del diario, primer perfil y Seguridad; consultarla y cerrarla conserva el paso y los datos. Es información de consulta, separada de las aceptaciones legales. Describe las garantías efectivas y distingue el diario real de los ejemplos ficticios.
- **RP-VLT-038.** El vocabulario visible ES/EN reserva «contraseña / password» para el acceso a la cuenta, «clave de recuperación / recovery key» para la credencial textual del diario y «frase de recuperación / recovery phrase» para las 24 palabras. No utiliza «Vault», «Private Journal Vault» ni «contraseña del Vault» como nombres que la persona deba comprender. Los identificadores técnicos internos conservan su ámbito. La clave de recuperación conserva sus requisitos y se utiliza para recuperar el diario; el desbloqueo cotidiano utiliza biometría. Las explicaciones contemplan cuentas Apple sin contraseña MDL y comunican que recuperar la cuenta no recupera por sí solo el diario.

<a id="bloqueo"></a>
## Bloqueo

- **RP-LOCK-003.** Bloquear el Vault retira de la memoria el contenido descifrado, las fotografías descifradas y cualquier proyección construida con ellos. Conserva únicamente el ciphertext y el estado cifrado necesario para reanudar operaciones autorizadas después de un nuevo desbloqueo.

<a id="retirada-contenido"></a>
## Retirada de contenido

- **RP-VLT-019.** Eliminar una experiencia o un período elegible retira también todo su contenido privado y sus versiones, sin exigir que la persona desbloquee y revise manualmente el inventario.
- **RP-VLT-020.** Una eliminación pendiente se conserva hasta obtener confirmación y puede reanudarse sin duplicar efectos cuando se pierde la respuesta.
- **RP-VLT-021.** El dispositivo limpia sus copias después de la confirmación. Las referencias necesarias para la retirada sólo permanecen hasta destruir la última versión cifrada.
- **RP-VLT-022.** El progreso de una eliminación se comunica de manera comprensible. Consultarlo no inicia operaciones nuevas ni muestra avances ficticios.

<a id="escenarios-proteccion"></a>
## Escenarios de aceptación

| ID | Resultado observable |
|---|---|
| GWT-ODM-08 | Consultar y cerrar la ayuda conserva el paso y los datos y no inicia operaciones de cuenta. |
| GWT-ODM-18 | Las explicaciones distinguen acceso, desbloqueo y recuperación, y describen las fotos locales sin copia remota. |
| GWT-ODM-19 | Las cuentas Apple sin contraseña reciben información aplicable a sus medios de acceso y recuperación. |
| GWT-ODM-23 | Seguridad, configuración y primer perfil permiten consultar la misma información de protección. |
| GWT-ODM-24 | Entrada incompleta, formato inválido, credencial que no recupera, espera activa, falta de conexión, fallo de comunicación, descarga incompleta y preparación fallida conservan causas distintas. Un fallo de comunicación o preparación no se atribuye a credenciales incorrectas; una causa desconocida permanece genérica. Se conservan contadores y esperas, sin éxito parcial ni sustitución del diario. |
| GWT-ODM-25 | La configuración permite consultar requisitos de la clave y su cumplimiento; la entrada conserva las condiciones de validación y exige una acción explícita para continuar. |
