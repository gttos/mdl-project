# Vault E2EE — detalle transversal

Las reglas de privacidad, revisión, recovery, dispositivo único, borrados y baja están en [Vault](rules/VAULT.md), [Baja y ciclo de vida](rules/DATA_LIFECYCLE.md) y [Tareas automáticas y plazos](rules/DATA_LIFECYCLE.md#tareas). Las fotos se definen en [Perfil del Crush](rules/CRUSH.md#fotos). Este documento explica la correspondencia criptográfica y técnica; no mantiene un catálogo funcional alternativo ni certifica despliegues.

## 1. Alcance y propiedad de datos

Distribución funcional: `RP-VLT-001..006`, `RP-VLT-026..031` y `RP-FOT-001..006`. Los inventarios cerrados de cada dominio son las listas autorizadas; este documento no los duplica.

| Capa | Representación |
|---|---|
| API/RDS | Cuenta, estructura, bindings, inventario y encapsulaciones. |
| S3 Vault | Objetos ciphertext. |
| Base local | Entries cifradas e intenciones durables por namespace. |
| Almacén seguro del dispositivo | VMK y material local de acceso. |
| Memoria desbloqueada | Contenido privado descifrado para presentación y búsqueda. |
| Fotos | Archivos locales separados del inventario remoto Vault. |

El email de cuenta se cifra en el servidor fuera del Vault y la API puede recuperarlo sólo para las finalidades autorizadas. El perfil de la cuenta se compone del inventario cerrado de [Cuenta de usuario](rules/ACCOUNT.md#inventario-cuenta). Estas protecciones se distinguen del contenido privado E2EE del diario. Legal mantiene su propio [inventario de datos](legal/02_MDL_Inventario_Maestro_Datos.md).

## 2. Unidad de cifrado, revisión e identidad opaca

| Elemento | Contrato técnico |
|---|---|
| VMK | 32 bytes aleatorios generados en cliente. |
| Cifrado de payload | AES-256-GCM autenticado. |
| Envelope | Identidad `(entryType, entityId, revision)`; `crushId` es asociación. |
| Token opaco | HMAC-SHA-256 con VMK sobre `mdl-vault-lookup:v1:<entryType>:<entityId>`. |
| Codificación del token | Base64URL sin padding, 43 caracteres. |
| Binding | Único por identidad funcional y token dentro del Vault; mismo propietario. |
| Recuperación por contraseña | Argon2id con perfil interoperable compartido. |
| Recuperación por frase | Encapsulación independiente definida en `RP-VLT-033`: BIP-39 inglesa de 24 palabras, perfil cerrado de 2.048 iteraciones y sin salt aleatorio adicional. |

Las garantías funcionales sobre revisión, asociación y secretos se referencian por `RP-VLT-004`, `RP-VLT-007..010`, `RP-VLT-033` y `RP-SEG-006`. Salvo el perfil mnemónico cerrado por Product, los parámetros exactos interoperables y vectores de prueba viven en los documentos locales.

## 3. Alta, sincronización y recovery

Secuencia técnica de escritura: intención/entry cifrada local → transacción funcional con binding → metadata remota → POST firmado de ciphertext → verificación remota → confirmación → retiro de revisión anterior. La confirmación visible y el retry se rigen por [Guardado y sincronización](rules/SYNC.md).

La capacidad POST liga una key exacta, `application/octet-stream`, checksum SHA-256 y rango `0..65.536`; no existe fallback PUT. La API vuelve a comprobar longitud y SHA-256 mediante una lectura acotada antes de confirmar, sin usar ETag ni interpretar plaintext. Un mismatch retira el objeto y la metadata pendiente y libera la reserva; un fallo de storage conserva un estado recuperable. Renovar conserva el blob, el snapshot y la identidad de la operación, y un rechazo genérico de firma/policy no se trata como expiración. La cuota lógica, su aviso, las señales autorizadas de observabilidad y la caducidad de capacidades se rigen por `RP-VLT-027..031`. La implementación coordinada API+iOS está aceptada con MinIO y recorrido durable local; queda pendiente la comprobación operativa contra AWS S3.

Recovery pagina inventario, descarga, descifra, agrupa por tipo/ID y selecciona revisión inequívoca antes de publicar el conjunto. Reglas completas: `RP-VLT-011..018` y `RP-VLT-032`; durante la beta existe un único dispositivo activo por Vault conforme a `RP-VLT-015`, y los únicos mecanismos de recuperación son la contraseña del Vault y la frase de 24 palabras. No se distribuyen transferencia QR, permisos de cámara asociados ni el recurso servidor `recovery-key`.

## 4. Borrado dirigido de experiencias y períodos

Contrato funcional: `RP-VLT-019..023`; elegibilidad: [Eliminación de experiencias](rules/EXPERIENCES.md#eliminar) y [Corrección de hitos y períodos](rules/RELATIONSHIPS.md#cambios).

El backend resuelve el binding de cada experiencia afectada por propietario/tipo/ID bajo la transacción funcional, marca entidad y revisiones y deja la destrucción física al proceso de purga. Eliminar un período retira los bindings y ciphertext de las experiencias que elimina; el período y sus hitos no tienen contenido Vault propio. La intención local se finaliza después de confirmar. Un borrado de Crush abarca todo el grafo según `RP-DEL-001..005`.

La baja de cuenta usa el resultado estructural específico de [Baja y ciclo de vida](rules/DATA_LIFECYCLE.md); no reutiliza automáticamente la semántica destructiva del borrado manual del crush.

## 5. Baseline nuevo y datos legacy

El contrato de lanzamiento está en `RP-BASE-006`. Los procedimientos históricos de migración no son funcionalidades de producto ni prerrequisitos para un alta normal. Su código residual y retirada se revisan en los planes existentes; conservarlo en un documento técnico local no autoriza compatibilidad runtime.

Este documento no conserva el historial de una migración ni valida su estado actual. Git mantiene la historia; los trabajos todavía abiertos permanecen en su [plan](plans/active/vault-e2ee/README.md).

## 6. Trabajo pendiente antes del cierre

Estado de implementación, validaciones físicas y operación: [plan Vault](plans/active/vault-e2ee/README.md), [plan Android](plans/active/android-parity/README.md) y [futura comprobación de reglas](plans/active/business-rule-integrity/README.md). Un hecho incorporado al catálogo no cierra esos planes.

### Confirmación de baja ante respuesta perdida

Correspondencia técnica de `RP-BAJ-008..011`:

- `DELETE /users` autenticado lleva `X-Account-Deletion-Confirmation-Secret`.
- Cliente persiste secreto aleatorio de al menos 256 bits antes del envío.
- API guarda SHA-256 del secreto con el recibo dentro de la transacción.
- `POST /api/account-deletion/confirm` utiliza el mismo header, sin sesión, body ni identificadores en URL.
- Resultado mínimo `completed` o `not_found`, sin revelar datos de cuenta.
- El hash sigue la retención del recibo; no se asigna aquí un plazo jurídico.
- La implementación local debe distinguir intento ambiguo, confirmación y limpieza física pendiente.

La política de conservación de estructura, ciphertext y material Apple está completa en Product. Las decisiones jurídicas abiertas no se resuelven desde los adaptadores técnicos.

## 7. Referencias

- [Catálogo funcional](PRODUCT.md).
- [Arquitectura transversal](ARCHITECTURE.md).
- [API Vault](../../api/docs/VAULT.md).
- [iOS Vault](../../ios/docs/VAULT.md).
- [Android Vault](../../android/docs/VAULT.md).
- [Plan Vault](plans/active/vault-e2ee/README.md).
- [Legal](legal/README.md).
