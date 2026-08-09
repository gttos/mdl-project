# Visión general actual del sistema MDL

> **Ámbito de esta revisión:** `product/`, `api/` e `ios/`. La documentación legal no forma parte de esta descripción técnica.

## 1. Componentes

| Componente | Implementación | Responsabilidad conectada |
|---|---|---|
| App iOS | Swift y SwiftUI | Interfaz, estado cliente, consumo de API, Vault local, cifrado y galería de fotos local. |
| API | Laravel y PHP | Autenticación, datos funcionales, reglas de dominio, metadata de Vault y URLs firmadas. |
| Base relacional | MySQL mediante Laravel | Usuarios, entidades funcionales, Vault, devices, recovery keys y metadata de blobs. |
| S3 Vault | Disco Laravel `s3_vault` | Ciphertext opaco de blobs del Vault. |
| Product | Markdown compartido | Reglas transversales, decisiones, feature maestra y coordinación de planes locales. |

El microservicio `llm/` es un proyecto independiente y no participa del flujo Vault o de la galería de fotos descritos aquí.

## 2. Fronteras de datos

MDL usa tres flujos conectados:

| Flujo | iOS | API / remoto |
|---|---|---|
| Datos funcionales | Construye y consume modelos de crush, experiencia, relación, timeline y estadísticas. | Recibe JSON estructurado y persiste las entidades de dominio. |
| Texto privado del Vault | Cifra JSON de perfil, experiencia y relación con VMK y lo guarda en `Documents/vault.sqlite3`. | Registra metadata y conserva ciphertext en `s3_vault`. |
| Notas seguras | Cifra notas con VMK en `local_vault_notes`. | El método de paso a entries no tiene callers; no hay sync remoto conectado. |
| Fotos del crush | Sanitiza y guarda archivos en `Documents/LocalMedia/{crushId}`. | No recibe archivos, metadata de galería ni blobs fotográficos desde el flujo activo. |

La app compone las vistas con datos funcionales de la API y contenido del Vault descifrado localmente.

## 3. Vault actual

### 3.1 iOS

- Genera una VMK de 32 bytes.
- Cifra payloads privados y notas con AES-256-GCM.
- Encapsula la VMK con una clave derivada mediante Argon2id.
- Guarda la VMK en Keychain con `WhenUnlockedThisDeviceOnly` y biometría en dispositivo físico.
- Guarda ciphertext, nonce y tag en SQLite.
- Registra blobs `text` y sube ciphertext por PUT a una URL firmada.
- Ofrece recuperación por contraseña, frase de 24 palabras y transferencia QR.

El índice FTS5 `vault_search_index.content` contiene texto de búsqueda en claro. Las notas usan un ID de índice distinto de su ID local y el borrado de nota no elimina esa fila FTS. La inicialización envía una encapsulación mnemónica, pero la API no la persiste ni la devuelve; la recuperación mnemónica en un dispositivo nuevo usa la encapsulación por contraseña descargada.

El retry de una URL S3 expirada espera un `blobId` que la API no devuelve y recibe una ruta temporal distinta del objeto registrado. La respuesta idempotente de alta también omite la URL que el modelo iOS declara obligatoria.

La descarga de recovery agrupa todos los payloads que contienen `crushId` bajo esa clave. Perfiles, experiencias y relaciones de un mismo crush compiten por una sola posición; el método conserva la secuencia mayor y solicita borrar el resto de blobs.

### 3.2 API

La API expone `/api/vault` para inicialización, encapsulación, blobs, devices, recovery key y migración. El alta de blob registra metadata y entrega una URL de S3 válida durante 15 minutos. La validación admite `text`, `image` y `video`; el cliente activo usa `text`.

El borrado de un blob es lógico. El comando diario `vault:purge-blobs` elimina de S3 y de la base los blobs borrados hace más de siete días.

`CleanupLegacyDataJob` solo escribe un log. Los Resources legacy todavía pueden devolver campos privados presentes en las tablas funcionales.

## 4. Fotos actuales

`MediaUploadService` identifica JPEG, PNG y HEIC, corrige orientación y vuelve a codificar la imagen mediante `CGImageDestination` sin copiar metadata EXIF, GPS o IPTC. HEIC/HEIF se convierte a JPEG.

La llamada conectada usa `shouldOptimize = false`: no ejecuta resize, compresión por niveles ni control de 10 MiB. `LocalMediaManager` escribe de forma atómica en `Documents/LocalMedia/{crushId}/{entryId}.jpg` y la galería lee esos archivos directamente.

Las fotos:

- no se cifran con la VMK;
- no crean registros `vault_assets` desde el flujo activo;
- no crean blobs `image` o `video`;
- no se envían a API o S3 de MDL;
- no tienen una marca `NSURLIsExcludedFromBackupKey` aplicada por la app;
- no se eliminan cuando se borra el crush completo.

## 5. Borrado de un crush

iOS identifica las entradas Vault del crush, experiencias y relaciones. Intenta borrar cada blob remoto y continúa si una de esas llamadas falla. Después elimina entradas y notas locales y solicita a la API el borrado del crush funcional.

La carpeta `Documents/LocalMedia/{crushId}` no participa de ese flujo y permanece en el dispositivo.

## 6. Texto sensible fuera del canal de blobs

Los formularios independientes de experiencias y relaciones construyen requests que incluyen `place`, `description` o `closure_note`. Los controladores actuales no incorporan esos campos a sus comandos de persistencia, pero el texto atraviesa HTTP.

`LegacyMigrationOrchestrator` y `CrushViewModel` escriben valores privados en logs locales. Estas rutas, las notas sin sync, el retry incompatible, la recuperación que colapsa entries y el índice FTS5 mantienen la feature Vault en revisión técnica.

## 7. Documentación canónica

- Estado compartido: `product/docs/product/media-and-vault.md`
- Decisión transversal: `product/docs/decisions/vault-e2ee-architecture.md`
- Feature maestra: `product/docs/plans/active/vault-e2ee/README.md`
- Contrato API: `api/docs/features/vault-e2ee.spec.md`
- Comportamiento iOS: `ios/docs/domains/features/vault-e2ee.md`
- Fotos iOS: `ios/docs/domains/features/media-handling.md`

La feature maestra prevalece sobre los planes locales cuando existe una contradicción documental.
