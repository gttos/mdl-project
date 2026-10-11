# Reglas de producto — Vault

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Contenido privado, creación, bloqueo, recuperación, sustitución de dispositivo y retirada de contenido.

<a id="contenido"></a>
## Contenido privado y datos funcionales

- **RP-VLT-001.** El contenido funcional privado de la beta está formado exactamente por los campos del perfil enumerados como «Vault opaco» en el [inventario del Crush](CRUSH.md#inventario) y por `place` y `description` de cada experiencia. Cada campo pertenece al elemento identificado en su documento de dominio.
- **RP-VLT-002.** La API funcional no acepta, devuelve ni conserva una copia en claro de los datos privados. Las aplicaciones los muestran únicamente después de acceder al Vault.
- **RP-VLT-003.** El dispositivo cifra cada payload privado antes de transferirlo y conserva localmente su entry cifrada. El Vault remoto recibe únicamente ciphertext y la metadata mínima necesaria como copia cifrada para sincronización y recuperación. La API sólo interpreta la estructura enumerada en los inventarios de [Cuenta](ACCOUNT.md#inventario-cuenta), [Crush](CRUSH.md#inventario), [Experiencias](EXPERIENCES.md#inventario) y [Relaciones](RELATIONSHIPS.md#inventario); el backend y el almacenamiento remoto no pueden interpretar el contenido privado. Ese contenido sólo se recupera en claro dentro de la memoria de un dispositivo autorizado después de desbloquear o recuperar el mismo Vault.
- **RP-VLT-026.** Los inventarios de dominio son listas cerradas y sus tratamientos son excluyentes. Cada payload Vault contiene únicamente los campos marcados como «Vault opaco» para ese tipo de entidad; ningún campo clasificado como «API legible», «Administrado por el sistema», «Derivado por la API», «Sólo dispositivo» o equivalente se copia dentro del payload cifrado. El payload Vault de una experiencia contiene exactamente `place` y `description`. Un período o hito relacional no tiene payload Vault propio: cualquier contenido privado que explique ese acontecimiento pertenece a su experiencia vinculada.
- **RP-VLT-004.** La clave principal del Vault se genera en el dispositivo y nunca llega en claro al servidor. Soporte nunca solicita esa clave ni los secretos de recuperación.

<a id="inventario-servidor"></a>
## Inventario legible por el servidor

- **RP-VLT-024.** El servidor conserva únicamente la siguiente estructura del Vault. Estos campos describen propiedad, cifrado, almacenamiento, versión, recuperación y ciclo de vida; no revelan el contenido privado cifrado.

| Registro | Campos legibles por la API | Finalidad |
|---|---|---|
| Vault de la cuenta | `id`, `user_id`, `crypto_version`, `kdf_algorithm`, `salt`, `kdf_memory_kb`, `kdf_iterations`, `kdf_parallelism`, `encrypted_vmk`, `vmk_nonce`, `vmk_auth_tag`, `is_active` | Identificar un único Vault, derivar la clave localmente y recuperar la clave principal sólo en su forma cifrada. |
| Elemento cifrado | `id`, `vault_id`, `user_id`, `blob_type`, `content_type_hint`, `ciphertext_length`, `ciphertext_hash`, `remote_path`, `remote_storage`, `crypto_version`, `nonce`, `auth_tag`, `aad_hash`, `sequence_number`, `lookup_token` | Guardar, verificar, ordenar y recuperar ciphertext sin interpretarlo. |
| Vínculo con una entidad funcional | `id`, `vault_id`, `user_id`, `entry_type`, `entity_id`, `lookup_token` | Localizar el elemento cifrado que pertenece a un Crush o experiencia sin copiar su contenido privado. |
| Dispositivo autorizado | `id`, `vault_id`, `user_id`, `device_public_key_hash`, `device_fingerprint`, `authorized_at`, `last_seen_at`, `revoked_at`, `is_primary` | Aplicar la sustitución de dispositivo y registrar su estado sin almacenar la clave privada del dispositivo. |
| Recuperación mediante frase | `id`, `vault_id`, `kdf_algorithm`, `kdf_iterations`, `encrypted_vmk`, `nonce`, `auth_tag` | Recuperar la misma clave principal mediante la frase de 24 palabras, sin almacenar la frase ni un salt aleatorio que no participe en su derivación. |

- **RP-VLT-025.** `encrypted_vmk` y los blobs son material cifrado. `nonce`, `auth_tag`, hashes, fingerprints, tokens de búsqueda, parámetros de derivación, tamaños, tipos, rutas, versiones, secuencias y fechas son metadatos técnicos; ninguno autoriza al servidor a interpretar nombre, género, contactos, cumpleaños, lugar o descripción.
- **RP-VLT-027.** Cada objeto de ciphertext del Vault admite como máximo 65.536 bytes. El dispositivo valida el tamaño después de cifrar y antes de transferir; la API vuelve a validarlo antes de autorizar o confirmar la subida. Superar el límite rechaza la operación sin crear una versión confirmada y se comunica de forma comprensible. Las fotos no forman parte del Vault y se rigen por su contrato propio.
- **RP-VLT-028.** Cada cuenta dispone de 25 MiB —26.214.400 bytes— de capacidad lógica total para su Vault remoto. La capacidad ocupada suma la última versión confirmada de cada elemento y el crecimiento reservado por subidas todavía no confirmadas; una revisión sustituida, retirada o en purga no consume nuevamente cuota. Una subida nueva reserva todo su tamaño declarado y una sustitución reserva únicamente el aumento respecto de la versión vigente. Reintentar la misma operación no duplica la reserva.
- **RP-VLT-029.** Al alcanzar 20 MiB —el 80 % de la capacidad— la aplicación avisa de forma comprensible cuánto espacio está ocupado y cuál es el límite. Alcanzar la cuota sólo impide operaciones que aumenten la ocupación: siempre permite eliminar contenido y sustituirlo por otro de tamaño igual o menor. Un rechazo conserva íntegramente la versión vigente y explica cómo liberar espacio.
- **RP-VLT-030.** La API controla capacidad, errores y abuso mediante la metadata autorizada del Vault: bytes confirmados o reservados, cantidad y antigüedad de subidas incompletas, ritmo de operaciones, fallos de verificación y acumulación anómala de revisiones. Puede aplicar alertas y límites de frecuencia sobre esas señales sin descifrar, interpretar ni clasificar el contenido privado. La cantidad de Crushes o experiencias, por sí sola, no constituye abuso.
- **RP-VLT-031.** Una capacidad de subida que no se confirma ni se cancela caduca 24 horas después de su creación. Desde ese momento no puede confirmarse, deja de reservar cuota y la limpieza automática retira cualquier objeto y metadata incompletos. La caducidad no altera la última versión confirmada ni elimina la operación pendiente del dispositivo; un reintento posterior solicita una capacidad nueva para esa misma intención durable.

<a id="recuperacion"></a>
## Creación, actualización y recuperación del Vault

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
- **RP-VLT-032.** La beta recupera el Vault únicamente mediante su contraseña o la frase de 24 palabras. No incluye transferencia de la VMK mediante QR, un recurso servidor `recovery-key`, rutas HTTP asociadas, superficies de cámara ni código ejecutable conservado para una reactivación futura. Incorporar otra transferencia entre dispositivos exige una decisión de Product y un protocolo autenticado nuevo antes de volver a introducir código o permisos.
- **RP-VLT-033.** La recuperación mediante frase usa exactamente una frase BIP-39 inglesa de 24 palabras generada en el dispositivo a partir de 256 bits aleatorios. La clave de encapsulación se obtiene con PBKDF2-HMAC-SHA512 sobre las palabras unidas por un único espacio, con el salt BIP-39 fijo `mnemonic`, sin passphrase adicional y con 2.048 iteraciones; se usan los primeros 32 bytes del seed de 64 bytes para encapsular la misma VMK mediante AES-256-GCM. El servidor conserva como identificadores cerrados `kdf_algorithm=pbkdf2-sha512` y `kdf_iterations=2048`, además de ciphertext, nonce y tag; no recibe ni conserva un salt aleatorio para esta ruta. Cualquier otro algoritmo, número de iteraciones o tamaño criptográfico se rechaza. Cambiar este perfil exige una decisión nueva de Product y una migración coordinada de todos los consumidores.

<a id="bloqueo"></a>
## Bloqueo del Vault

- **RP-LOCK-003.** Bloquear el Vault retira de la memoria el contenido descifrado y cualquier proyección construida con él. Conserva únicamente el ciphertext y el estado cifrado necesario para reanudar operaciones autorizadas después de un nuevo desbloqueo.

<a id="retirada-contenido"></a>
## Retirada de contenido concreto

- **RP-VLT-019.** Eliminar una experiencia o un período elegible retira también todo su contenido privado y sus versiones, sin exigir que la persona desbloquee y revise manualmente el inventario.
- **RP-VLT-020.** Una eliminación pendiente se conserva hasta obtener confirmación y puede reanudarse sin duplicar efectos cuando se pierde la respuesta.
- **RP-VLT-021.** El dispositivo limpia sus copias después de la confirmación. Las referencias necesarias para la retirada sólo permanecen hasta destruir la última versión cifrada.
- **RP-VLT-022.** El progreso de una eliminación se comunica de manera comprensible. Consultarlo no inicia operaciones nuevas ni muestra avances ficticios.

La diferencia entre retirada del uso normal y destrucción física se define en [Ciclo de vida](DATA_LIFECYCLE.md#retirada-y-destruccion). Las [fotos del Crush](CRUSH.md#fotos) tienen sus propias reglas.
