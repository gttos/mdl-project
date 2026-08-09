# Feature completada: fotos del crush local-only

> **Estado:** comportamiento verificado en código
> **Fecha de verificación:** 2026-07-15
> **Ámbito:** iOS + API + documentación compartida

## Resultado

La galería del crush usa el filesystem local de iOS como única fuente de fotos. Las fotos se guardan en `Documents/LocalMedia/<crushId>/`, no crean Vault blobs y no generan tráfico de red.

## Integración

| Proyecto | Comportamiento |
|---|---|
| iOS | Selección, procesamiento, guardado, lectura y borrado individual local |
| API | Sin rutas funcionales de media; endpoint genérico de blobs disponible pero no usado por la galería |
| S3 | Sin objetos de fotos creados por el flujo actual de la app |

## Evidencia de código

- `MediaUploadService` termina en `LocalMediaManager.saveImage`.
- `LocalMediaManager` opera sobre `Documents/LocalMedia`.
- `VaultSyncManager` no itera assets en `processQueue()`.
- `DependencyContainer` no inyecta `VaultMediaSyncManager`.
- `routes/api.php` solo contiene rutas de blobs opacos, sin controller funcional de media.

## Límites actuales

- Los JPEG locales no se cifran con VMK.
- El directorio no se marca como excluido de backup del sistema operativo.
- El borrado del crush no elimina el directorio local de fotos.

## Planes locales completados

- API: `api/docs/plans/completed/photo-backup-hold/README.md`
- iOS: `ios/docs/plans/completed/photo-backup-hold/README.md`
