# Vault E2EE — MDL

Fuente de verdad transversal del Private Journal Vault. Describe el contrato estable ya validado y los límites que la transición activa todavía debe cerrar. Los detalles técnicos viven en [`api/docs/VAULT.md`](../../api/docs/VAULT.md), [`ios/docs/VAULT.md`](../../ios/docs/VAULT.md) y [`android/docs/VAULT.md`](../../android/docs/VAULT.md); el orden pendiente del Vault está en su [plan maestro activo](plans/active/vault-e2ee/README.md) y la incorporación Android en el [plan de paridad](plans/active/android-parity/README.md).

## 1. Alcance y propiedad de datos

| Clase | Qué incluye | Fuente de verdad | Visibilidad del backend |
|---|---|---|---|
| Datos funcionales | IDs, fechas, estados, ratings, asociaciones, atributos y métricas; `gender_id` durante la beta | API/RDS | Legible y sujeto a minimización/retención de Product y Legal |
| Contenido privado | Nombre, contacto y redes del crush; cumpleaños; lugar/descripción de experiencia; nota de cierre; notas seguras | Snapshot cifrado en el Vault | Ciphertext y metadata mínima; no VMK ni plaintext desde ningún cliente |
| Fotos del crush | Imágenes elegidas por el usuario | Almacenamiento local del cliente | No salen del dispositivo en la beta |

`secure_description` no forma parte del producto, del Vault ni de ningún contrato presente o futuro. Es una variante legacy descartada de la descripción de experiencia: no se escribe, migra, recupera ni conserva. Cualquier columna, comando, fixture o documentación residual debe retirarse de API mediante una fase separada y no constituye un dato que deba entrar en el canario de contenido privado.

La VMK es una clave aleatoria de 32 bytes generada en el cliente. Nunca se entrega en claro a API ni S3. Los datos de cuenta no forman parte del Vault: `User.email` se almacena normalizado y legible para API porque identifica la cuenta y soporta autenticación, unicidad, verificación, recuperación y comunicaciones operativas; `User.phone` se cifra a nivel de aplicación con `APP_KEY`. Ninguno es E2EE. Ambos requieren además TLS, control de acceso y cifrado de almacenamiento y backups, cuya configuración efectiva debe verificarse operativamente.

El contrato funcional vigente no acepta ni devuelve esos campos privados y las columnas legacy ya se retiraron físicamente. Cualquier cliente que todavía los transporte es incompatible: debe resolver el contenido desde el Vault antes de conectarse a un entorno real.

## 2. Unidad de cifrado, revisión e identidad opaca

- Un blob `text` contiene el snapshot privado completo de una sola entidad: crush, experiencia o relación. No existe un blob por campo.
- El envelope identifica `(entryType, entityId, revision)` y contiene el `crushId` sólo como asociación.
- Editar conserva tipo e ID funcional e incrementa `revision`. La revisión anterior se mantiene únicamente hasta verificar la nueva y luego se retira; no existe historial de versiones visible ni conservación deliberada de cambios antiguos.
- El cliente deriva un token estable por entidad mediante `HMAC-SHA-256(VMK, "mdl-vault-lookup:v1:<entryType>:<entityId>")`, codificado Base64URL sin padding.
- El token no contiene el UUID ni plaintext de forma reversible. API lo trata como metadata sensible y nunca lo registra o devuelve en mensajes.
- API persiste un binding inmutable y único entre `(vault, entryType, entityId)` y el token. Varias revisiones temporales de la misma entidad comparten token; dos entidades del mismo Vault no.
- Registrar un blob `text` exige que el binding exista, pertenezca al usuario autenticado y coincida. Ausencia, formato inválido, cruce de usuario o contradicción fallan cerrados antes de mutar.

## 3. Alta, sincronización y recovery

Las rutas que materializan un crush, experiencia o relación reciben el token de cada entidad nueva y crean su binding dentro de la misma transacción funcional. Un retry idéntico reutiliza UUID y token; una contradicción se rechaza sin mutación parcial.

Para entidades legacy ya existentes, `POST /vault/migration-bindings` recibe el manifest opaco completo, lo compara contra el inventario funcional activo del propietario y crea o reutiliza bindings en una sola transacción. Faltantes, extras, duplicados, tipos/propietarios incorrectos o conflictos de token rechazan todo antes de mutar. Como toda respuesta pública de API, sus resultados usan `entryType` y `entityId` en camelCase aunque el request use `entry_type` y `entity_id`. Esta operación completa el prerrequisito backend de la migración, pero no demuestra que iOS haya construido, subido o recuperado los snapshots legacy.

La subida de una revisión sigue este orden:

1. persistir localmente snapshot, entry e intención durable;
2. registrar metadata del blob con su token;
3. subir ciphertext directamente a S3 mediante URL firmada;
4. verificar existencia y longitud remota;
5. marcar la revisión nueva como confirmada;
6. retirar la revisión remota anterior.

Recovery tiene un propósito diferente al borrado ordinario. `downloadAllBlobs()` pagina el inventario global, descarga y descifra localmente, agrupa por `(entryType, entityId)`, selecciona una revisión inequívoca y publica el conjunto validado de forma atómica. Nunca borra objetos remotos.

Dos métodos están activos en beta:

- contraseña de recuperación: Argon2id con el perfil canónico compartido por los clientes;
- frase de 24 palabras: encapsulación independiente con PBKDF2-SHA512.

Ambos recuperan la misma VMK y el mismo inventario. La transferencia QR conserva infraestructura, pero no está expuesta como flujo normal de beta.

Durante la beta se opera un Vault desde un único dispositivo activo. Recovery se usa para sustituir el dispositivo: una vez comprobado el historial en el nuevo, el anterior no debe seguir utilizándose para editar ni sincronizar. La app confirma que **ese dispositivo** subió y verificó su propia revisión; no promete que otra copia local la haya aplicado. No existe todavía una descarga incremental normal ni una política de conflictos para dos dispositivos activos; ambas quedan explícitamente fuera de beta. Esta restricción operativa no equivale aún a revocar automáticamente el dispositivo anterior.

## 4. Borrado dirigido de experiencia y relación

El borrado individual validado no recorre el Vault:

1. el cliente realiza preflight local y persiste una intención durable;
2. envía un único `DELETE` funcional de experiencia o relación;
3. API bloquea el aggregate/Vault, resuelve el binding y aplica en la misma transacción el soft delete funcional y el de todos los blobs activos con ese token;
4. tras el éxito, el cliente elimina entries locales e intención y refresca la vista;
5. una respuesta perdida reanuda el mismo delete de forma idempotente, sin duplicar ni reabrir estado.

API falla cerrado si no puede resolver el binding. El cliente no usa `GET /vault/blobs`, detalle, `verify`, descarga, descifrado ni `DELETE /vault/blobs/{id}` para descubrir revisiones durante esta acción.

El borrado integral de un Crush aplica la misma regla a todo su grafo: si falta el binding del Crush, de alguna experiencia o de alguna relación asociada, API rechaza la operación completa sin borrar estructura, blobs ni estado local. El usuario puede reintentarlo una vez reparada la inconsistencia; nunca se comunica éxito parcial.

Al borrar el Crush completo no se reabre una relación anterior: todo su historial relacional se elimina con el mismo grafo. La reapertura sólo pertenece al borrado individual de una relación, donde puede cambiar el estado vigente.

Los bindings del grafo eliminado se conservan sólo durante el soft delete y la ventana de retry/purga. Cuando la purga física de todas sus revisiones termina, se eliminan junto al cierre definitivo del grafo; no esperan a la baja de cuenta.

La purga física sigue separada: el comando programado elimina objeto S3 y metadata después de la ventana técnica de 7 días. El éxito funcional no promete destrucción física inmediata.

Este contrato fue validado con API desplegada sobre una cuenta y un Vault nuevos: altas/ediciones sincronizaron, los borrados de experiencia y relación emitieron una única petición funcional sin escaneo global, y recovery en otro dispositivo reconstruyó sólo los datos activos.

## 5. Baseline nuevo y datos legacy

El baseline validado es creación desde cero. No hay backfill, fallback ni binding inventado para entidades/blobs antiguos sin token.

La migración del único usuario legacy forma parte de la iniciativa activa y precede al corte de campos privados de API. Es una operación manual única desde una superficie Debug, no una funcionalidad de producto ni un flujo para cuentas nuevas. API-14 e iOS-09.8 ya implementan y prueban con fixtures el registro de bindings existentes, staging tokenizado, retiro durable/idempotente de revisiones anteriores y segunda validación de inventario exacto antes de activar. La primera ejecución física descubrió que Timeline no puede ser fuente del inventario: es una proyección que puede agrupar u ocultar entidades. API-14R expone el grafo canónico migrable e iOS-09.8R lo consume para staging y manifest, sin exponer identificadores internos al usuario ante un fallo. El grafo contiene cada entidad no eliminada cuyo crush no esté anonimizado —también si está archivada— exactamente una vez y con los campos legacy necesarios para cifrar su snapshot; usa el mismo predicado que el manifest. `iOS-09.8S` ya reconcilia con seguridad el staging aislado de intentos anteriores: sólo puede reconstruirlo si el backend confirma cero blobs activos y el checkpoint local no indica que se inició el sync; cualquier otra condición lo preserva y falla cerrado. La primera prueba física posterior se detuvo antes de manifest/sync con `invalidSource`: iOS no reconocía el timestamp válido de seis microsegundos que Eloquent emite. `iOS-09.8U` acepta ese formato sin omitir ni transformar datos y corrigió el 100 % engañoso. `iOS-09.8V` confirmó que las tres tablas legacy permiten `updated_at` nulo; iOS ya lo transporta sin romper el array, pero lo rechaza fail-closed porque un snapshot necesita una fecha de revisión. `API-14S` materializa ya una fecha efectiva sólo para `GET /vault/migration-source`: cada `updatedAt` es `updated_at`, si existe; si no, `created_at`; si tampoco existe, una única hora actual UTC calculada una vez para toda la respuesta. La regla no escribe ni modifica filas legacy, no usa una hora distinta por entidad y sólo existe hasta retirar el contrato de migración tras `CROSS-02`. No se permite omitir entidades ni transformar otros campos.

Tras corregir el contrato camelCase y sanear la observabilidad en `iOS-09.8W`, la ejecución física del único usuario legacy terminó con el Vault validado/activado y el historial se reconstruyó correctamente en entornos limpios por contraseña y por frase de 24 palabras. `CROSS-02` queda validada. La prueba posterior confirmó que una revisión subida por el dispositivo recuperado no se descarga en el origen durante el uso normal; no es un gate adicional porque la beta adopta un único dispositivo activo por Vault. El siguiente trabajo retira las rutas de migración, el botón Debug, el orquestador y su staging; los bindings normales de API-13 permanecen. El despliegue requiere preservar tanto la base de datos como los objetos cifrados del almacenamiento Vault: la DB contiene metadata y rutas; el ciphertext vive fuera de ella.

## 6. Trabajo pendiente antes del cierre

El baseline nuevo, la recuperación y el borrado dirigido de experiencia/relación están implementados y validados, pero eso no cierra `vault-e2ee`. La iniciativa permanece activa hasta completar el orden y los gates del plan maestro.

| Área pendiente | Estado y riesgo |
|---|---|
| Contrato/columnas privadas legacy de API | Algunos validators y columnas todavía aceptan o pueden devolver contenido privado. Retirarlos requiere fase coordinada y compatibilidad explícita. |
| Borrado completo de crush | Aún debe coordinar blobs, entries/notas locales y `LocalMedia/{crushId}`; no se considera resuelto por el borrado individual. |
| Baja integral de cuenta | Contrato confirmado (ver más abajo); `API-11` implementada y validada con tests en API (incluida la confirmación segura tras respuesta perdida y la cobertura de Crushes legacy con blobs activos). `iOS-12` sin implementar todavía. |
| Reset de Vault manteniendo la cuenta | Retirado por decisión de producto: no existe ni existirá una operación de reset independiente de la baja. `DELETE /vault` ya fue retirado físicamente de API; falta retirar "Reset Vault (Debug)" en iOS como parte de `iOS-12`. |
| Fotos | Permanecen locales, sin backup remoto; falta exclusión comprobada de backups del sistema y borrado coordinado con crush/cuenta. |
| Sincronización/concurrencia multi-device | Fuera de beta. No hay descarga incremental entre dos copias activas, política de conflictos ni revocación automática del dispositivo anterior tras recovery. |
| Operación y Legal | Deben conservarse evidencia de purga a 7 días, configuración de backups/proveedores, respuesta a incidentes y revisión jurídica previa al lanzamiento. |

El backup cifrado de fotos y una auditoría/penetration test específica del Vault siguen fuera de alcance. La migración del usuario antiguo y la retirada de columnas legacy sí pertenecen al plan activo, con autorizaciones operativas/destructivas separadas.

**Contrato confirmado para la baja integral de cuenta (`API-11`/`iOS-12`); implementado y validado en API, `iOS-12` pendiente:**

- `DELETE /users` ejecuta de forma síncrona toda la baja lógica y criptográfica de la cuenta como una única operación atómica y fail-closed: sólo responde éxito cuando el estado remoto queda coherente. Cualquier fallo dispara rollback total; no existe baja parcial.
- Al confirmar la baja, todos los blobs Vault del usuario entran de inmediato en soft-delete, incluidos los de un Crush que ya estuviera soft-deleted desde antes de que existiera el borrado integral (`API-17`) y que nunca completó esa limpieza. Se conservan cifrados durante la misma ventana de siete días que el borrado dirigido de experiencia/relación (§4) y luego la purga programada los elimina físicamente de S3. Repetir la baja sobre un grafo que ya no tiene nada pendiente es idempotente.
- Los bindings del usuario se conservan únicamente para localizar y completar esa purga; se eliminan cuando desaparece la última revisión de cada binding, no en el momento de la baja.
- Tokens de sesión, dispositivos Vault, recovery keys (contraseña y frase de 24 palabras) y sus encapsulaciones quedan invalidados o eliminados de inmediato; ninguno sobrevive a una baja exitosa.
- **Confirmación segura ante respuesta perdida:** antes de llamar a `DELETE /users`, iOS genera un secreto aleatorio de un solo uso (mínimo 256 bits) y lo envía en un header dedicado de esa misma petición. API guarda sólo su hash junto al receipt interno de la baja, dentro de la misma transacción atómica -- nunca el secreto en claro, nunca en logs. Si la respuesta de `DELETE /users` se pierde en tránsito, iOS puede presentar ese mismo secreto a un endpoint de confirmación separado, sin sesión ni token (los tokens ya fueron revocados): ese endpoint sólo informa si la baja quedó `completed`, está limitado por rate limiting, y responde exactamente el mismo resultado genérico ante un secreto incorrecto, ausente o inexistente -- nunca expone email, receipt ID, Vault ID ni ningún otro identificador. No es ni habilita un endpoint de estado por usuario. El hash no tiene vencimiento propio: se conserva mientras exista el receipt interno de la baja, bajo la misma retención que ya rige ese receipt.
- iOS borra su estado local (SQLite Vault, VMK y material Keychain, fotos/`LocalMedia`, notas, operaciones pendientes y preferencias sensibles) sólo después de recibir el 200 de `DELETE /users`, o la confirmación `completed` del endpoint de respuesta perdida si el 200 original nunca llegó; si ninguno de los dos confirma la baja, iOS no borra nada local.
- La baja es irreversible: no existe restauración parcial ni un estado mixto (por ejemplo, reactivar sólo `users` con crushes ya anonimizados y experiencias/relaciones borradas).
- Se retiran `DELETE /vault` (API) y "Reset Vault (Debug)" (iOS): no existe una operación de producto para vaciar y reinicializar el Vault de una cuenta activa. Recovery por contraseña o por frase de 24 palabras es el único mecanismo para recuperar acceso manteniendo la cuenta.
- `vaults.user_id` conserva su restricción UNIQUE: un usuario tiene un único Vault durante toda su vida. `POST /vault/init` debe detectar explícitamente un Vault soft-deleted del mismo usuario y responder con un error de dominio controlado; nunca lo recrea ni deja que la restricción dispare una excepción SQL sin manejar.

Detalle funcional (Dado/Cuando/Entonces) en GWT-VLT-42 del [plan maestro](plans/active/vault-e2ee/README.md). Detalle técnico de API en [`api/docs/VAULT.md`](../../api/docs/VAULT.md) §10.

## 7. Referencias

- [`PRODUCT.md`](PRODUCT.md) §10
- [`ARCHITECTURE.md`](ARCHITECTURE.md) §8
- [`Plan maestro activo`](plans/active/vault-e2ee/README.md)
- [`api/docs/VAULT.md`](../../api/docs/VAULT.md)
- [`ios/docs/VAULT.md`](../../ios/docs/VAULT.md)
- [`android/docs/VAULT.md`](../../android/docs/VAULT.md)
- [`legal/README.md`](legal/README.md)
