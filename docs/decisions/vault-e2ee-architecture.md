# Decisión vigente: separación entre datos funcionales, Vault y fotos locales

> **Fecha de revisión:** 2026-07-15
> **Estado:** vigente con implementación parcial
> **Ámbito:** iOS + API

## Regla transversal

MDL separa el contenido privado textual de los datos funcionales:

- La API conserva IDs, estados, fechas, ratings, atributos y métricas necesarios para las funciones compartidas.
- La app iOS cifra los payloads privados con una VMK y los sincroniza como ciphertext opaco.
- La API almacena la encapsulación de la VMK, metadata de blobs y ciphertext en S3; no posee la VMK.
- Las fotos del crush permanecen como JPEG locales en el sandbox de iOS y no participan en el Vault remoto.

## Modelo criptográfico implementado

| Elemento | Implementación actual |
|---|---|
| VMK | 32 bytes aleatorios generados en iOS |
| Cifrado de payload | AES-256-GCM mediante CryptoKit |
| Encapsulación por contraseña | Argon2id, 64 MiB, 3 operaciones, AES-256-GCM |
| Persistencia de VMK | Keychain, `WhenUnlockedThisDeviceOnly`, biometría en dispositivo físico |
| Recuperación por contraseña | Descarga de la encapsulación desde `GET /api/vault` y descifrado local |
| Frase de 24 palabras | BIP-39/PBKDF2 disponible en iOS; la API no persiste esa segunda encapsulación |
| Transferencia QR | ECDH P-256 entre dispositivos y descarga posterior de blobs |

## Contrato remoto implementado

La API expone gestión de Vault, blobs, dispositivos y recovery keys bajo `/api/vault`. Las respuestas JSON se transforman a camelCase por middleware. Los blobs se registran por API y el ciphertext se transfiere directamente con URLs firmadas de 15 minutos.

El schema y la validación de blobs admiten `text`, `image` y `video`. La cola activa de iOS registra únicamente blobs `text`.

## Límite real de la implementación

La separación E2EE es efectiva en el flujo de creación y edición del crush y en la cola de Vault. No es un límite completo de transporte en toda la app:

- Formularios independientes de experiencias y relaciones incluyen texto privado en sus payloads HTTP.
- Resources de API conservan campos textuales de las tablas funcionales.
- `CleanupLegacyDataJob` no modifica datos; solo escribe un log.
- El índice FTS5 local conserva contenido de búsqueda en texto en claro.
- Varias rutas de iOS registran contenido privado en logs.
- Las notas cifradas no ingresan en la cola remota porque `syncLocalNotesToVault()` no tiene callers.
- El retry de upload e idempotencia tienen respuestas incompatibles con el modelo iOS.
- La recuperación agrupa por `crushId` y elimina como zombies entries válidas del mismo crush.
- El alta inicial usa el UUID funcional como `clientBlobId`, por lo que la API puede correlacionar ese blob con su entidad aunque no descifre el contenido.

Estas condiciones forman parte del estado actual y mantienen la feature maestra en revisión de cierre técnico.

## Referencias

- Estado compartido: `product/docs/product/media-and-vault.md`
- API: `api/docs/features/vault-e2ee.spec.md`
- iOS: `ios/docs/domains/features/vault-e2ee.md`
