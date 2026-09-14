# 3. Vault, almacenamiento y seguridad

> Modelo objetivo de lanzamiento a 2 de septiembre de 2026. Se redacta con Vault E2EE terminado, sin columnas ni contratos funcionales legacy para contenido privado. No equivale a auditoría criptográfica, certificación ni evidencia de despliegue.

## 1. Garantía central

iOS genera una VMK aleatoria de 32 bytes por Vault y cifra el contenido privado con AES-256-GCM antes de transferirlo. API y S3 conservan ciphertext, encapsulaciones y metadata necesarias, sin VMK en claro ni una operación de descifrado.

En el modelo final, nombre/contacto/redes/cumpleaños del crush, lugar/descripción de experiencias y nota de cierre no se aceptan ni devuelven en rutas funcionales y no permanecen en columnas legacy.

Una exposición aislada de RDS/S3 no debería revelar ese plaintext sin acceso adicional a la VMK o a material suficiente de recovery. Esta frase describe el límite arquitectónico; la evaluación real de un incidente debe considerar configuración, claves, dispositivo, encapsulaciones, secreto elegido y posibilidad de combinar fuentes.

## 2. Qué protege y qué no

| Protegido por E2EE | No protegido por E2EE |
|---|---|
| nombre, contacto, redes y cumpleaños del crush | email/teléfono y perfil de la cuenta |
| lugar y descripción de experiencias | IDs, fechas, estados, ratings, rasgos y asociaciones |
| nota de cierre y notas seguras | match status, estadísticas y metadata Vault |
| snapshots e intenciones privadas cifradas | IP, user agent, rutas, trazas y eventos de seguridad |
| objeto S3 frente a lectura sin clave | fotos locales, salvo controles propios del dispositivo |

E2EE no protege un dispositivo desbloqueado comprometido, capturas del usuario, instrumentación insegura, secretos de recovery expuestos o un atacante que controle simultáneamente los extremos relevantes.

## 3. Claves y recuperación

### 3.1 VMK y custodia local

- Una VMK se genera por Vault en iOS.
- El material local se namespacea por cuenta/Vault y se guarda bajo controles de Keychain.
- Cambiar de cuenta o namespace no reutiliza una VMK residual.
- La VMK sólo existe en plaintext durante operaciones autorizadas del dispositivo.

### 3.2 Métodos de recovery

- **Contraseña de recovery:** deriva una clave mediante Argon2id con el perfil interoperable aprobado.
- **Frase de 24 palabras:** usa una encapsulación independiente con PBKDF2-SHA512.
- Ambos mecanismos recuperan la misma VMK y el mismo inventario.
- La infraestructura QR/recovery-key no se ofrece como tercer método normal de beta.

La contraseña/frase es un secreto de alto impacto. El soporte nunca debe pedirla ni recibirla. Quien combine acceso a la cuenta y material de recovery suficiente podría descifrar el Vault. Perder todos los mecanismos puede volver irrecuperable el contenido.

## 4. Unidad de cifrado y versionado

- Un blob `text` contiene el snapshot privado completo de un crush, experiencia o relación.
- La identidad lógica es `(entryType, entityId, revision)`; `crushId` sólo asocia.
- Editar conserva tipo e ID e incrementa la revisión.
- La revisión nueva se sube, descarga/verifica y confirma antes de retirar la anterior.
- La coexistencia temporal no es un historial de versiones disponible para el usuario.
- Un conflicto o inventario ambiguo falla cerrado.

No existe un índice local persistente de plaintext. La búsqueda privada se realiza en memoria sobre contenido ya descifrado y el término no se envía a API.

## 5. Binding opaco

iOS deriva un `lookup_token` estable por entidad mediante HMAC-SHA-256 con la VMK, tipo e ID funcional. API no puede generarlo o invertirlo.

API conserva un binding único e inmutable por `(vault, entryType, entityId)` y exige token coincidente al registrar blobs. El binding permite seleccionar todas las revisiones de una entidad durante borrado sin descargar, descifrar o recorrer el Vault completo.

El token sigue siendo metadata sensible y no debe aparecer en logs, errores, analítica o interfaces de soporte.

## 6. Sincronización y recovery

### 6.1 Escritura

1. iOS valida el formulario y congela identidad/fecha.
2. Cifra y persiste localmente snapshot e intención durable.
3. API materializa la entidad y binding dentro de la transacción funcional.
4. API registra metadata y emite una URL firmada temporal.
5. iOS sube ciphertext directamente a S3.
6. iOS verifica existencia/longitud y confirma la revisión.
7. Sólo entonces retira la revisión sustituida y comunica sincronización verificada.

Un retry reutiliza UUID, token, snapshot y operación. Un timeout ambiguo se reconcilia; no se finge descarte local si API pudo aceptar.

### 6.2 Recovery

Recovery pagina el inventario completo, descarga y descifra localmente, agrupa por identidad, selecciona una revisión inequívoca y publica el conjunto sólo después de validarlo entero. No ejecuta borrados remotos.

La beta usa un dispositivo activo por Vault. Después de recovery, el dispositivo anterior no debe seguir editando como copia activa. No existe sincronización incremental ni resolución de conflictos entre dos dispositivos activos.

## 7. Borrado y retención

### 7.1 Experiencia o relación

1. iOS persiste la intención de borrado.
2. Emite una única petición funcional.
3. API resuelve el binding bajo lock y soft-deletea entidad y blobs dentro de la transacción.
4. iOS limpia entries locales sólo después de confirmación.
5. La purga programada destruye objeto y metadata elegible después de siete días.

El éxito visible significa borrado funcional confirmado, no destrucción física instantánea.

### 7.2 Crush

El borrado coordina crush, experiencias, relaciones, bindings, todas las revisiones remotas, entries/notas locales e imágenes bajo `LocalMedia/{crushId}`. No debe sobrevivir una carpeta de fotos o blob activo sin entidad.

### 7.3 Cuenta

`DELETE /users` coordina, como una única operación atómica y fail-closed, cuenta, sesiones, estructura, Vault y fotos: sólo responde éxito cuando el estado remoto queda coherente, y cualquier fallo revierte por completo. Los blobs entran en soft-delete inmediato y siguen la misma ventana de purga a siete días que el borrado dirigido (§7.1); los bindings se conservan sólo hasta completar esa purga. Encapsulaciones, devices y recovery keys se invalidan/eliminan de inmediato. iOS elimina Keychain, SQLite, preferencias sensibles y fotos locales sólo después de confirmar el éxito remoto. La estructura que se pretenda conservar para estadísticas se trata según la estrategia aprobada; no se llama anónima por mera sustitución de identificadores directos.

No existe una operación de reset de Vault independiente de la baja: recovery por contraseña o por frase de 24 palabras es el único mecanismo para recuperar acceso manteniendo la cuenta.

### 7.4 Backups y restauraciones

La ventana de siete días no describe por sí sola RDS snapshots, versionado S3, backups del dispositivo o copias de proveedores. La política debe indicar:

- qué backups existen y por cuánto tiempo;
- si un objeto borrado conserva versiones;
- cómo se impide que una restauración reactive datos ya borrados;
- cuándo deja de ser recuperable cada categoría;
- qué excepciones legales pueden suspender una purga.

## 8. Almacenamiento por capa

| Capa | Contenido | Control principal | Riesgo que debe gestionarse |
|---|---|---|---|
| Memoria iOS | plaintext y VMK durante sesión | App Lock, ciclo de vida y aislamiento | dispositivo desbloqueado, crash dump o logging |
| Keychain | VMK/material local | protección del dispositivo y namespace | supervivencia a reinstalación, acceso físico |
| SQLite iOS | ciphertext, metadata e intenciones | AES-GCM, namespace y file protection | backup/restauración y corrupción |
| API/RDS | cuenta, estructura, encapsulaciones, bindings y metadata | auth, ownership, TLS, storage controls y cifrado de campos seleccionados | inferencias, personal autorizado, email legible y exposición de `APP_KEY` |
| S3 Vault | ciphertext | bucket privado, URL firmada y lifecycle | exposición de objetos/metadata, versión y purga |
| Fotos locales | imágenes saneadas | sandbox, file protection y exclusión de backup | pérdida/robo del dispositivo y borrado |
| Observabilidad | trazas y contexto técnico | allowlist/scrubbing/acceso | fuga accidental de payload o identificadores |

## 9. Amenazas relevantes

| Amenaza | Consecuencia | Salvaguarda o respuesta |
|---|---|---|
| takeover de cuenta sin secreto Vault | acceso a estructura/metadata; posible intento de recovery | MFA/protecciones futuras, rate limits, revocación y alertas |
| dispositivo desbloqueado comprometido | acceso potencial a plaintext y fotos | App Lock, Keychain, file protection, revocación y guía de dispositivo |
| secreto de recovery expuesto | descifrado potencial del Vault afectado | educación, no solicitar secreto, respuesta/rotación diseñada |
| RDS expuesto | cuenta, estructura, inferencias, metadata y encapsulaciones | cifrado de storage, mínimo privilegio, segmentación, auditoría |
| S3/ciphertext expuesto | copia de contenido cifrado y patrones de metadata | bucket privado, URLs cortas, AES-GCM, evaluación combinada |
| `APP_KEY` de backend expuesta | teléfono y cualquier otro campo cifrado expresamente con esa clave; el email ya es legible para backend | inventario exacto y rotación/re-cifrado planificado; Vault usa VMK separadas |
| fallo de ownership | acceso cruzado entre cuentas | policies, scoping, tests y fallo cerrado |
| logs/Sentry con payload | exposición de contenido o secretos | allowlist, scrubbing, tests y monitorización |
| restauración de backup | reaparición de datos borrados | inventario, procedimiento de restauración y repurga |
| dos dispositivos activos | conflicto, revisión perdida o estado divergente | restricción de beta y sustitución explícita |

## 10. Controles que deben existir en el cierre

- namespace por cuenta y no adopción de VMK residual;
- cifrado autenticado y VMK fuera del backend;
- endpoints funcionales sin campos privados en claro;
- columnas legacy vaciadas y retiradas mediante gates con backup/rollback;
- binding obligatorio y ownership por entidad;
- retry idempotente y rollback transaccional;
- recovery completo, atómico y no destructivo;
- borrado dirigido y borrado integral de crush/cuenta;
- exclusión de secretos y contenido de logs/errores/analítica;
- fotos saneadas, protegidas, excluidas de backup y borradas coordinadamente;
- lifecycle/purga operativa y evidencia de backups/versiones;
- acceso de personal por mínimo privilegio, MFA y trazabilidad;
- revisión del threat model y procedimiento de incidentes.

## 11. Evidencia previa a publicar garantías

- versión exacta de app/API y migrations aplicadas;
- inventario que demuestre ausencia de campos privados en contratos y columnas funcionales;
- tests y prueba física de alta, edición, recovery por ambos métodos y borrados;
- prueba de aislamiento entre dos cuentas y fallo cerrado de binding;
- captura saneada de purga a siete días, S3 lifecycle/versionado y RDS backups;
- prueba de exclusión de backup y borrado de fotos/Keychain/SQLite;
- configuración efectiva de TLS, cifrado de storage, buckets, IAM, Sentry y logs;
- inventario de dispositivos y procedimiento de revocación/sustitución;
- ejercicio de restauración que no reactive datos borrados;
- revisión independiente proporcional del diseño y de las implementaciones criptográficas;
- EIPD/screening y aprobación jurídica del lenguaje público.

## 12. Preguntas para asesoría

1. ¿Qué afirmaciones exactas sobre E2EE y acceso del proveedor son admisibles?
2. ¿Cómo debe informarse la pérdida irreversible si el usuario pierde todos los mecanismos de recovery?
3. ¿Qué retención/backups son proporcionales para ciphertext, metadata, bindings y eventos?
4. ¿Qué controles deben exigirse al dispositivo y qué responsabilidad puede atribuirse al usuario?
5. ¿Qué nivel de revisión independiente o prueba de penetración se espera antes del lanzamiento?
6. ¿Cómo se articula portabilidad y acceso cuando backend no puede descifrar?
