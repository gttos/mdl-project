# Estado actual: fotos del crush sin sincronización remota

> **Fecha de verificación documental:** 2026-07-15

## Comportamiento

- La app permite añadir, listar, abrir y borrar fotos del crush.
- Cada foto se procesa con `ImageConversionHelper` y se guarda como JPEG en `Documents/LocalMedia/<crushId>/`.
- La galería enumera directamente los archivos del directorio.
- El flujo no llama a la API ni a S3.
- `VaultSyncManager.processQueue()` procesa entradas de texto y no procesa assets.
- `DependencyContainer` no registra `VaultMediaSyncManager`.
- La API no expone endpoints funcionales de media en `routes/api.php`.

## Alcance del backup

No existe backup de fotos mediante la API ni mediante Vault blobs. El código iOS no configura `NSURLIsExcludedFromBackupKey` para `Documents/LocalMedia`, por lo que este documento no afirma exclusión de backups gestionados por el sistema operativo.

## Referencias

- Estado compartido: `product/docs/product/media-and-vault.md`
- Registro completado: `product/docs/plans/completed/photo-backup-hold/README.md`
- Implementación iOS: `ios/docs/domains/features/media-handling.md`
