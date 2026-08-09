# Media y Vault — estado actual

> **Ámbito:** comportamiento compartido entre la app iOS y la API.
> **Fuente de verdad técnica:** código de `ios/` y `api/` revisado el 2026-07-15.
> **Fuera de alcance:** interpretación legal y documentación bajo `docs/legal/`.

## 1. Fronteras de datos

MDL mantiene tres flujos distintos:

| Flujo | Contenido | Persistencia local | Tráfico remoto |
|---|---|---|---|
| Datos funcionales | IDs, edad, país, estados, fechas, ratings, atributos y métricas | Estado de pantalla y modelos de la app | JSON de los endpoints funcionales |
| Texto del Vault | Datos privados de crushes, experiencias y relaciones | SQLite `Documents/vault.sqlite3`; payload cifrado con VMK | Metadata por API y ciphertext directo a S3 |
| Notas seguras | Texto de `SecureNoteEditor` | `local_vault_notes`, cifrado con VMK | Ninguno desde el flujo conectado |
| Fotos del crush | Imágenes elegidas en la galería del perfil | JPEG en `Documents/LocalMedia/<crushId>/` | Ninguno |

La app combina los datos funcionales recibidos de la API con el contenido del Vault descifrado localmente para construir las vistas de crush, home y timeline.

## 2. Texto del Vault

### 2.1 Contenido de los payloads

La app define tres payloads JSON cifrados:

- `CrushVaultJSON`: `crushId`, `name`, `profession`, contactos sociales, `birthdate` y `genderId`.
- `ExperienceVaultJSON`: `experienceId`, `crushId`, `place`, `description` y `satisfactionId`.
- `RelationshipVaultJSON`: `relationshipId`, `crushId`, `closureNote` y `stateId`.

Las notas privadas se guardan cifradas en `local_vault_notes`. `syncLocalNotesToVault()` puede convertirlas en entries, pero no tiene callers y `processQueue()` no lo invoca.

### 2.2 Cifrado y almacenamiento local

1. iOS genera una VMK aleatoria de 32 bytes.
2. `VaultCryptoManager` cifra los payloads con AES-256-GCM.
3. `VaultStorageManager` guarda ciphertext, nonce y tag en `vault_entries` o `local_vault_notes` dentro de `Documents/vault.sqlite3`.
4. La VMK se guarda en Keychain con `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`; en dispositivo físico, el acceso usa `biometryCurrentSet`.

La misma base SQLite contiene `vault_search_index`, un índice FTS5 cuyo campo `content` se escribe en texto en claro durante la migración y las operaciones de indexado. Las notas se indexan con un UUID distinto del registro local; borrar la nota no elimina esa fila FTS.

### 2.3 Sincronización remota

`VaultSyncManager.processQueue()` procesa únicamente entradas de texto pendientes:

1. Calcula SHA-256 del ciphertext y el hash de AAD.
2. Registra metadata con `POST /api/vault/blobs` usando `blobType: text`.
3. Recibe una URL firmada de 15 minutos.
4. Sube el ciphertext directamente a S3 mediante `PUT`.
5. Marca la entrada local como subida.

El backend almacena la encapsulación de la VMK, metadata de blobs y referencias a objetos de S3. No recibe la VMK ni el plaintext dentro del flujo de blobs.

El retry por URL expirada no es compatible: iOS exige `blobId`, la API no lo devuelve y genera una ruta `blobs/temp/...` que no actualiza el objeto asociado al blob. La respuesta idempotente de alta omite la URL que iOS declara obligatoria.

Durante recovery, iOS inspecciona el plaintext y elige `crushId` antes que `experienceId` o `relationshipId`. Todos los payloads de un crush quedan agrupados juntos; se conserva el de mayor secuencia y se solicitan DELETE para los demás.

## 3. Fotos del crush

### 3.1 Alta

1. `PHPickerViewController` entrega los datos seleccionados.
2. `MediaUploadService` identifica JPEG, PNG o HEIC por firma de archivo.
3. `ImageConversionHelper.prepareImageForUpload` decodifica, corrige la orientación y vuelve a codificar la imagen sin copiar metadata EXIF, GPS o IPTC. HEIC/HEIF se convierte a JPEG.
4. `LocalMediaManager` guarda el resultado como `<uuid>.jpg` en `Documents/LocalMedia/<crushId>/`.
5. La galería incorpora el archivo local a la vista.

La llamada activa usa `shouldOptimize = false`: no ejecuta resize, compresión por niveles ni el límite de 10 MiB. Este flujo no llama a `VaultAPIClient`, `S3UploadManager` ni a un endpoint de media.

### 3.2 Lectura y borrado

- La lista se reconstruye enumerando los `.jpg` del directorio del crush.
- La fecha de creación del archivo ordena la galería.
- La vista de detalle lee el JPEG directamente del filesystem.
- El borrado individual elimina el archivo local y lo retira de la colección en memoria.

Las fotos no se cifran con la VMK, no crean blobs y no tienen recuperación remota. El directorio pertenece al sandbox de la aplicación. El código no establece `NSURLIsExcludedFromBackupKey` ni una clase explícita de protección de archivo. El manager usa la extensión `.jpg`; una fuente PNG puede conservar bytes PNG sanitizados dentro de ese nombre de archivo.

## 4. Código de media presente pero no conectado

El repositorio iOS contiene `VaultMediaSyncManager`, vistas de media del Vault, cifrado por chunks y métodos de assets en `VaultStorageManager`. Ese conjunto no forma parte del flujo activo:

- `DependencyContainer` no registra `VaultMediaSyncManager`.
- `VaultSyncManager.processQueue()` tiene desactivado el procesamiento de assets.
- La galería del crush usa `LocalMediaManager`, no `VaultMediaSyncManager`.

La API acepta `blob_type` con valores `text`, `image` y `video`. La cola activa de la app iOS solo crea metadata con `blobType: text`.

## 5. Contratos funcionales que aún transportan texto

Los formularios independientes de experiencias y relaciones construyen payloads que incluyen `place`, `description` o `closure_note`. Esos valores llegan a la capa HTTP de la API. Los controllers actuales construyen los comandos de dominio sin esos campos, por lo que no se escriben mediante esos flujos, pero el transporte no es exclusivamente por blobs.

Los Resources de API conservan campos de texto (`name`, contactos, `place`, `description`, `closure_note`) y pueden devolver valores existentes en las columnas de datos funcionales. `CleanupLegacyDataJob` solo registra su ejecución y no limpia esas columnas.

## 6. Borrado de un crush

La app realiza este flujo:

1. Localiza las entradas del crush y descifra las demás entradas para identificar experiencias y relaciones asociadas.
2. Intenta borrar cada blob remoto.
3. Registra el error y continúa cuando un borrado remoto falla.
4. Elimina entradas y notas locales.
5. Envía `DELETE /api/crushes/{id}`.

El flujo no elimina el directorio `Documents/LocalMedia/<crushId>/`. En la API, `DELETE /api/vault/blobs/{id}` aplica soft delete; el comando diario `vault:purge-blobs` elimina el objeto S3 y la fila después de siete días.

## 7. Documentación técnica local

- API: `api/docs/features/vault-e2ee.spec.md`
- iOS: `ios/docs/domains/features/vault-e2ee.md`
- Fotos iOS: `ios/docs/domains/features/media-handling.md`
- Feature maestra: `product/docs/plans/active/vault-e2ee/README.md`
