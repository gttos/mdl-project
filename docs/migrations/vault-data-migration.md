# Migración de datos existentes al Vault — comportamiento implementado

> **Ámbito:** iOS + API
> **Fecha de revisión:** 2026-07-21

## Datos leídos por la migración

`LegacyMigrationOrchestrator` pagina los crushes del usuario y consulta el perfil de cada uno. A partir del perfil y del timeline crea:

- Un `CrushVaultJSON` por crush.
- Un `ExperienceVaultJSON` por experiencia.
- Un `RelationshipVaultJSON` por relación.

Las fotos no forman parte de esta migración.

Cada payload se guarda en una entry local distinta con un UUID aleatorio. `backend_entity_id` conserva el ID funcional específico: crush, experiencia o relación. Al sincronizar, el UUID aleatorio de la entry se usa como `client_blob_id` del blob remoto.

## Secuencia ejecutada

1. iOS genera VMK, frase de 24 palabras y par ECDH del dispositivo.
2. iOS encapsula la VMK con la Recovery Password.
3. iOS envía `POST /api/vault/init` y guarda la VMK en Keychain.
4. iOS guarda localmente la encapsulación por contraseña.
5. La migración borra las entradas locales del Vault y descarga los datos funcionales del usuario.
6. Cada payload se cifra con la VMK y se inserta en `vault_entries`.
7. Nombres, profesiones, lugares, descripciones y notas de cierre también se insertan en `vault_search_index` como texto en claro.
8. `VaultSyncManager` registra blobs `text` y sube ciphertext a S3.
9. `migrateAllData()` ignora el `SyncResult` y envía `POST /api/vault/migrate-complete` aunque alguna entry haya fallado.
10. `VaultOnboardingViewModel.runMigration()` llama después a `syncAndComplete()`, que puede enviar una segunda confirmación.
11. La API encola `CleanupLegacyDataJob` por cada confirmación aceptada.

## Resultado en la API

`CleanupLegacyDataJob::handle()` registra que fue ejecutado y no modifica columnas, filas de media ni objetos S3. Las columnas textuales existentes permanecen disponibles para los Resources de crushes, experiencias y relaciones.

La confirmación actual no demuestra una migración completa. La API solo exige que el Vault tenga al menos un blob y no puede conocer qué crushes, experiencias o relaciones contiene el ciphertext.

## Recuperación

- Con Recovery Password, iOS obtiene la encapsulación guardada en la API, recupera la VMK y descarga los blobs.
- La frase de 24 palabras funciona con una encapsulación mnemonic disponible localmente. `POST /api/vault/init` no persiste los campos mnemonic enviados por iOS y `GET /api/vault` devuelve la encapsulación de la Recovery Password.
- La transferencia QR entrega la VMK entre dos dispositivos y luego descarga los blobs remotos.

La recuperación remota intenta repoblar entradas de texto. La agrupación elige `crushId` antes que los IDs de experiencia o relación; conserva una sola entry por crush y solicita borrar las demás. No existen blobs de fotos en el flujo activo. Las notas creadas por `SecureNoteEditor` no se incorporan a la cola remota conectada.

## Remediación aprobada, todavía no implementada

La ejecución defectuosa del único usuario real no se reparará entry por entry. Se corregirá el migrador y se ejecutará desde legacy una reconstrucción completa y versionada:

1. Un DTO legacy dedicado conservará los campos requeridos mientras la fuente siga disponible.
2. Una fecha real aportará solo día/mes; una fecha ausente permanecerá ausente. No se fabricarán `01/01` ni `31/12`.
3. Edad y su ancla seguirán el contrato `age`/`age_as_of`; `met_at` no participará.
4. El conjunto nuevo se construirá en staging, se verificará por inventario exacto y solo después sustituirá al conjunto activo.
5. Un fallo parcial conservará el Vault anterior y no permitirá `migrate-complete`.

El contrato y las fases ejecutables están en `product/docs/plans/active/crush-age-birthday/README.md` y en los planes locales de API/iOS. Esta sección describe una decisión aprobada; no afirma que el botón actual ya sea seguro.

## Referencias

- Feature maestra: `product/docs/plans/active/vault-e2ee/README.md`
- Edad y cumpleaños: `product/docs/plans/active/crush-age-birthday/README.md`
- Estado compartido: `product/docs/product/media-and-vault.md`
- Contrato API: `api/docs/features/vault-e2ee.spec.md`
- Implementación iOS: `ios/docs/domains/features/vault-e2ee.md`
