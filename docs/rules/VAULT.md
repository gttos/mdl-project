# Reglas de producto — Vault

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Contenido privado, creación, bloqueo, recuperación, sustitución de dispositivo y retirada de contenido.

<a id="contenido"></a>
## Contenido privado y datos funcionales

- **RP-VLT-001.** El contenido funcional privado de la beta está formado exactamente por: los campos del perfil enumerados como «Vault opaco» en el [inventario del Crush](CRUSH.md#inventario); `place` y `description` de cada experiencia; y `closure_note` de cada período relacional. Cada campo pertenece al elemento identificado en su documento de dominio.
- **RP-VLT-002.** La API funcional no acepta, devuelve ni conserva una copia en claro de los datos privados. Las aplicaciones los muestran únicamente después de acceder al Vault.
- **RP-VLT-003.** La API sólo interpreta la estructura enumerada en los inventarios de [Cuenta](ACCOUNT.md#inventario-cuenta), [Crush](CRUSH.md#inventario), [Experiencias](EXPERIENCES.md#inventario) y [Relaciones](RELATIONSHIPS.md#inventario). El ciphertext del Vault puede conservarse y recuperarse, pero su contenido funcional permanece opaco para el servidor.
- **RP-VLT-004.** La clave principal del Vault se genera en el dispositivo y nunca llega en claro al servidor. Soporte nunca solicita esa clave ni los secretos de recuperación.

<a id="inventario-servidor"></a>
## Inventario legible por el servidor

- **RP-VLT-024.** El servidor conserva únicamente la siguiente estructura del Vault. Estos campos describen propiedad, cifrado, almacenamiento, versión, recuperación y ciclo de vida; no revelan el contenido privado cifrado.

| Registro | Campos legibles por la API | Finalidad |
|---|---|---|
| Vault de la cuenta | `id`, `user_id`, `crypto_version`, `kdf_algorithm`, `salt`, `kdf_memory_kb`, `kdf_iterations`, `kdf_parallelism`, `encrypted_vmk`, `vmk_nonce`, `vmk_auth_tag`, `is_active` | Identificar un único Vault, derivar la clave localmente y recuperar la clave principal sólo en su forma cifrada. |
| Elemento cifrado | `id`, `vault_id`, `user_id`, `blob_type`, `content_type_hint`, `ciphertext_length`, `ciphertext_hash`, `remote_path`, `remote_storage`, `crypto_version`, `nonce`, `auth_tag`, `aad_hash`, `sequence_number`, `lookup_token` | Guardar, verificar, ordenar y recuperar ciphertext sin interpretarlo. |
| Vínculo con una entidad funcional | `id`, `vault_id`, `user_id`, `entry_type`, `entity_id`, `lookup_token` | Localizar el elemento cifrado que pertenece a un Crush, experiencia o período sin copiar su contenido privado. |
| Dispositivo autorizado | `id`, `vault_id`, `user_id`, `device_public_key_hash`, `device_fingerprint`, `authorized_at`, `last_seen_at`, `revoked_at`, `is_primary` | Aplicar la sustitución de dispositivo y registrar su estado sin almacenar la clave privada del dispositivo. |
| Recuperación mediante contraseña del Vault | `id`, `vault_id`, `user_id`, `encrypted_vmk_backup`, `backup_nonce`, `backup_auth_tag`, `key_fingerprint`, `created_at`, `revoked_at` | Recuperar la misma clave principal mediante la contraseña del Vault, sin almacenar esa contraseña. |
| Recuperación mediante frase | `id`, `vault_id`, `kdf_algorithm`, `salt`, `kdf_iterations`, `encrypted_vmk`, `nonce`, `auth_tag` | Recuperar la misma clave principal mediante la frase de 24 palabras, sin almacenar la frase. |

- **RP-VLT-025.** `encrypted_vmk`, `encrypted_vmk_backup` y los blobs son material cifrado. `nonce`, `auth_tag`, hashes, fingerprints, tokens de búsqueda, parámetros de derivación, tamaños, tipos, rutas, versiones, secuencias y fechas son metadatos técnicos; ninguno autoriza al servidor a interpretar nombre, género, contactos, cumpleaños, lugar, descripción o nota de cierre.

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

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-10"></a>DEC-10 | Máximo por objeto, cuota por cuenta, expiración/limpieza de uploads incompletos. | Propiedad, privacidad y validación obligatorias; no cuota inventada. |
