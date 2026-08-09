# Next Stage — Features Post-MVP

> **Ámbito:** Transversal — `api/`, `ios/`, `product/docs/`  
> **Estado:** Planificado, no implementado  
> **Propósito:** Centralizar todo lo que NO está en el MVP pero se planea para futuras iteraciones. Este documento es la única fuente de verdad para features futuras. Los documentos MVP no deben referenciar este archivo.

---

## 1. Vault Media Sync — Backup cifrado de fotos del crush

> **Fase original:** iOS-10 / API-09  
> **Branch de preservación:** `feat/photo-backup-hold` (en `api/` e `ios/`)

### 1.1 ¿Qué es?

Sincronización automática de fotos de la galería del iPhone (PHPhotoLibrary) al Vault E2EE como blobs opacos cifrados con VMK.

### 1.2 ¿Por qué no está en el MVP?

- El MVP prioriza la privacidad máxima: fotos del crush son 100% local-only.
- La sincronización automática introduce complejidad (cuotas, thumbnails, background sync, recovery de media) que no es crítica para el lanzamiento.

### 1.3 Componentes necesarios

| Componente | Proyecto | Descripción |
|---|---|---|
| `VaultMediaSyncManager` | iOS | Scan automático de PHPhotoLibrary, comparación con vault_blobs existentes, detección de fotos nuevas |
| `ChunkedFileEncryptor` | iOS | Cifrado por chunks de 1MB para videos > 1MB |
| Background sync | iOS | Timer de 5 min en foreground + background fetch para detectar fotos nuevas |
| Cuotas de storage | Backend | `user.plan.max_vault_storage_bytes`, alerta al 80% |
| Bulk download | Backend | Descarga eficiente de múltiples blobs para recovery |
| Thumbnails optimizados | Backend | Thumbnails de media cifrada, generados en cliente |
| `backend_entity_id` | Backend | Campo en `vault_blobs` para asociar media con entidad (crush/experience) |

### 1.4 Flujo de reactivación

```
1. Merge de `feat/photo-backup-hold` a `develop` (ambos repos)
2. Implementar iOS-10: VaultMediaSyncManager, background scan
3. Implementar API-09: cuotas, bulk download, thumbnails
4. Implementar API-03.1: `backend_entity_id` en vault_blobs
5. Recovery de fotos en `VaultSyncManager.downloadAllBlobs()`
6. Tests de integración end-to-end
```

### 1.5 Estrategia de merge

**Opción A: Merge directo de `feat/photo-backup-hold`**
```bash
cd api && git checkout develop && git merge feat/photo-backup-hold
cd ios && git checkout develop && git merge feat/photo-backup-hold
```

**Opción B: Re-implementar sobre develop limpio**
Si `develop` avanzó mucho (>20 commits), usar `feat/photo-backup-hold` como referencia y reimplementar las fases iOS-10 / API-09.

**Opción C: Cherry-pick**
Identificar commits específicos relacionados con fotos y cherry-pickarlos uno a uno.

**Recomendación:** Empezar con Opción A. Si >10 conflictos, fallback a Opción C.

---

## 2. Purga automática de datos legacy

> **Fase:** API-04 → API-07  
> **Estado:** Planificado, pendiente de ejecución manual

### 2.1 API-04: Cleanup de datos legacy (set NULL)

- Setear a NULL las columnas sensibles en `crushes`, `experiences`, `relationships`.
- Soft delete de registros `File` legacy.
- **Quién:** Usuario (manualmente, con backup previo de DB).
- **Prerequisito:** Confirmar que la migración legacy funcionó.

### 2.2 API-07: DROP de columnas sensibles

- Eliminar físicamente las columnas sensibles de la DB.
- **IRREVERSIBLE.** Solo ejecutar cuando el flujo Vault esté 100% estable.

---

## 3. Testing y seguridad del Vault (backend)

> **Fase:** API-05  
> **Estado:** Planificado

- Integration tests de owner scoping, rate limits, idempotencia.
- Penetration test básico.
- Security review: confirmar que el backend nunca descifra.

---

## 4. Documentación legal (backend)

> **Fase:** API-06  
> **Estado:** Expediente canónico actualizado; evidencia productiva y revisión jurídica pendientes

- Fuente única: `product/docs/legal/README.md`.
- El expediente describe el modelo de lanzamiento con Vault terminado, inventario por entidad y fronteras de lectura.
- Antes de entregarlo deben adjuntarse evidencias productivas de tráfico, recuperación, borrado, proveedores y backups.

---

## 5. DPO Review y validación final

> **Fase:** LEGAL-01  
> **Estado:** No iniciada

- Revisión de arquitectura E2EE por DPO.
- Validación de compliance RGPD post-migración.
- Aprobación de política de privacidad actualizada.

---

## 6. Integración final iOS (end-to-end)

> **Fase:** iOS-09  
> **Estado:** En progreso

- Cross-testing con backend real.
- Performance: desbloqueo <1s, búsqueda FTS <50ms.
- Edge cases: app se cierra durante sync, sin conexión, Keychain borrado.

---

## 7. Referencias

- Feature maestra Vault: `product/docs/plans/active/vault-e2ee/README.md`
- Decisión local-only completada: `product/docs/plans/completed/photo-backup-hold/README.md`
- Plan local API: `api/docs/plans/active/vault-e2ee/README.md`
- Plan local iOS: `ios/docs/plans/active/vault-e2ee/README.md`
