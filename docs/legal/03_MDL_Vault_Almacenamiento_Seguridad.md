# 3. Vault, almacenamiento y seguridad

> Modelo funcional objetivo de la beta a 21 de septiembre de 2026. No equivale a auditoría criptográfica, certificación ni evidencia de despliegue.

## 1. Garantía central

El cliente genera una VMK aleatoria de 32 bytes por Vault y cifra el contenido privado con AES-256-GCM antes de transferirlo. API y S3 conservan ciphertext, encapsulaciones y metadata necesaria, sin VMK en claro ni una operación para descifrar el contenido.

El contenido Vault de la beta está cerrado a:

- Crush: nombre, género, profesión, WhatsApp, Instagram, Facebook, Snapchat, TikTok y cumpleaños sin año;
- Experience: `place` y `description`.

Períodos e hitos relacionales no tienen payload Vault ni texto propio. La explicación de cada hito pertenece a la descripción de su experiencia obligatoria.

El email de la cuenta se cifra en el servidor fuera del Vault. La API puede recuperarlo sólo para operaciones autorizadas de cuenta y utiliza un índice exacto no reversible para buscar y garantizar unicidad. Fecha de nacimiento, género/país opcionales y estados de cuenta permanecen como estructura legible autorizada.

## 2. Qué protege y qué no

| Protegido por Vault E2EE | Fuera del Vault E2EE |
|---|---|
| nombre, género, profesión, WhatsApp, redes y cumpleaños del Crush | email cifrado recuperable, perfil de cuenta y método Apple |
| lugar y descripción de experiencias | IDs, fechas, satisfacción, países, Ducky, ratings, rasgos y asociaciones |
| snapshots e intenciones privadas cifradas | situaciones, períodos, hitos, motivos de cierre, match y estadísticas personales |
| objeto S3 frente a lectura sin VMK | IP, user agent, rutas, trazas y eventos de seguridad |
| contenido privado durante transporte y almacenamiento remoto | fotos locales, protegidas por controles del dispositivo |

E2EE no protege un dispositivo desbloqueado comprometido, capturas realizadas por la persona, instrumentación insegura, secretos de recovery expuestos ni un atacante que controle simultáneamente extremos suficientes.

## 3. Claves y recuperación

- Una VMK se genera por Vault en el cliente.
- El material local se separa por cuenta/Vault y se guarda bajo controles seguros del dispositivo.
- Cambiar de cuenta no reutiliza una VMK residual.
- La VMK sólo existe en claro durante operaciones autorizadas del dispositivo.
- La contraseña del Vault deriva una clave mediante Argon2id con el perfil interoperable aprobado.
- La frase de 24 palabras usa una encapsulación independiente con PBKDF2-SHA512.
- Ambos mecanismos recuperan la misma VMK y el mismo inventario.
- La infraestructura QR/recovery-key no se ofrece como tercer método normal de beta.

Soporte nunca solicita contraseña, frase ni VMK. Perder ambos métodos puede volver irrecuperable el contenido.

## 4. Unidad de cifrado, identidad y límites

Un blob `text` contiene el payload privado completo de un Crush o Experience. La identidad lógica es `(entryType, entityId, revision)` y un lookup token HMAC derivado con la VMK enlaza la entidad funcional con sus revisiones sin revelar contenido.

API conserva un binding único e inmutable por entidad privada. El token no aparece en logs, errores, analítica ni interfaces de soporte. Una revisión nueva se confirma antes de retirar la anterior; esa coexistencia es transitoria y no constituye un historial de usuario.

Límites funcionales:

- máximo de 65.536 bytes de ciphertext por objeto;
- capacidad lógica de 25 MiB por cuenta;
- aviso al alcanzar 20 MiB;
- sólo se bloquean operaciones que aumenten la ocupación;
- una sustitución reserva únicamente su crecimiento respecto de la versión vigente;
- reintentar la misma operación no duplica la reserva;
- una capacidad no confirmada caduca a las 24 horas, deja de consumir cuota y su objeto/metadata se limpia automáticamente.

La API puede detectar errores o abuso mediante bytes, reservas, cantidad/antigüedad de subidas incompletas, ritmo de operaciones, fallos de verificación y revisiones anómalas. No descifra ni clasifica el contenido; la cantidad de Crushes o experiencias por sí sola no constituye abuso.

## 5. Escritura y recovery

### 5.1 Escritura durable

1. El cliente valida la acción completa y congela identidades y días civiles.
2. Cifra el snapshot y persiste localmente una intención durable.
3. API materializa estructura y binding dentro de la frontera transaccional correspondiente.
4. API registra metadata y emite una capacidad temporal ligada a propietario, objeto y reserva.
5. El cliente sube ciphertext directamente al almacenamiento privado.
6. Cliente y API verifican tamaño, existencia e integridad según su responsabilidad.
7. Sólo después se confirma la revisión, se retira la sustituida y se comunica «Sincronizado».

Un retry reutiliza identidad, token, snapshot y operación. Un timeout ambiguo se reconcilia; no se finge descarte cuando API pudo aceptar.

### 5.2 Recovery y dispositivo único

Recovery pagina el inventario completo, descarga y descifra localmente, agrupa por identidad, selecciona una revisión inequívoca y publica sólo después de validar el conjunto. No ejecuta borrados remotos.

La beta usa un dispositivo activo por Vault. Recuperar en otro dispositivo sustituye al anterior; no existe sincronización continua ni resolución de conflictos entre dos dispositivos activos.

## 6. Borrado y retención

### 6.1 Experiencia, hito o período

- Una experiencia independiente puede eliminarse sin borrar su período.
- Iniciar el borrado desde una experiencia vinculada elimina conjuntamente experiencia e hito sólo si cumple `latest-only`.
- Eliminar sólo el último hito posterior conserva opcionalmente su experiencia como independiente; eliminar ambos retira también su binding y blobs.
- Un período posterior sólo puede eliminarse cuando su hito inicial es su único hito; se eliminan período, hito y todas sus experiencias.
- El período inicial sólo desaparece con el Crush.

La API resuelve todos los bindings afectados bajo la operación funcional y aplica retirada a sus blobs. El cliente limpia entradas locales después de confirmar. La purga física ocurre siete días después.

### 6.2 Crush y cuenta

Eliminar un Crush coordina perfil, períodos, hitos, experiencias, bindings, revisiones remotas, estado local y fotos.

La baja de cuenta revoca sesiones, invalida dispositivos y recovery, retira identidad directa y pone el Vault fuera del uso ordinario. El ciphertext se purga después de siete días. La estructura que Producto pretende conservar seudonimizada queda condicionada a la decisión jurídica descrita en [Ciclo de vida](04_MDL_Ciclo_Vida_Retencion_Derechos.md).

### 6.3 Backups y restauraciones

Los siete días no describen RDS snapshots, versionado S3, backups del dispositivo ni copias de proveedores. Deben documentarse sus plazos, versiones, accesos y un procedimiento que impida reactivar datos retirados al restaurar.

## 7. Almacenamiento por capa

| Capa | Contenido | Control principal | Riesgo |
|---|---|---|---|
| Memoria del cliente | plaintext y VMK durante sesión | bloqueo, ciclo de vida y aislamiento | dispositivo desbloqueado o instrumentación |
| Almacén seguro | VMK/material local | protección y namespace | acceso físico o supervivencia no deseada |
| SQLite local | ciphertext, metadata e intenciones | cifrado, namespace y file protection | backup, corrupción o restauración |
| API/RDS | cuenta, estructura, email cifrado, índice, encapsulaciones, bindings y metadata | auth, ownership, TLS, claves de backend y mínimo privilegio | inferencias, acceso interno y exposición de claves |
| S3 Vault | ciphertext | bucket privado, capacidades temporales y lifecycle | copia, metadata, versionado y purga |
| Fotos locales | imágenes saneadas | sandbox, file protection y exclusión de backup | pérdida del dispositivo y borrado |
| Observabilidad | contexto técnico permitido | allowlist, scrubbing y acceso | fuga accidental de payload o identificadores |

## 8. Amenazas relevantes

| Amenaza | Consecuencia | Salvaguarda o respuesta |
|---|---|---|
| takeover sin secreto Vault | acceso a estructura y metadata | autenticación, revocación, rate limits y alertas |
| dispositivo desbloqueado comprometido | acceso a plaintext y fotos | bloqueo, almacenamiento seguro y guía de dispositivo |
| secreto de recovery expuesto | descifrado del Vault afectado | educación, soporte sin secretos y respuesta diseñada |
| RDS o clave de cifrado backend expuestos | cuenta, email, estructura, inferencias y metadata | keyring, rotación, mínimo privilegio, segmentación y auditoría |
| S3/ciphertext expuesto | copia de contenido cifrado y patrones | bucket privado, capacidades breves, AES-GCM y evaluación combinada |
| fallo de ownership | acceso cruzado | scoping, transacciones, tests y fallo cerrado |
| logs con payload | exposición de contenido o secretos | allowlist, scrubbing, tests y monitorización |
| restauración de backup | reaparición de datos retirados | inventario, restore controlado y repurga |
| dos dispositivos activos | conflicto o revisión perdida | sustitución explícita y restricción de beta |

## 9. Evidencia previa al lanzamiento

- versión exacta de clientes, API y migraciones;
- ausencia de campos privados en contratos y columnas funcionales;
- cifrado efectivo del email y protección del índice de búsqueda;
- pruebas de ownership, cuota, expiración, retry, recovery y borrados;
- evidencia de purga, versionado y backups;
- exclusión de backup y borrado de fotos/almacén seguro/SQLite;
- configuración efectiva de TLS, buckets, IAM, claves, observabilidad y logs;
- prueba de interoperabilidad del Vault en cada plataforma habilitada;
- revisión independiente proporcionada del diseño y la implementación criptográfica;
- EIPD o screening y aprobación jurídica del lenguaje público.

## 10. Preguntas para asesoría

1. ¿Qué afirmaciones exactas sobre E2EE y acceso del proveedor son admisibles?
2. ¿Cómo debe informarse la pérdida irreversible de todos los mecanismos de recovery?
3. ¿Qué retención y backups son proporcionales para ciphertext, metadata, bindings y eventos?
4. ¿Qué controles deben exigirse al dispositivo y qué responsabilidad puede atribuirse a la persona?
5. ¿Qué revisión independiente se exige antes del lanzamiento?
6. ¿Cómo se articula acceso y portabilidad cuando backend no puede descifrar?
7. ¿Cómo debe explicarse el email cifrado recuperable sin presentarlo como E2EE ni inaccesible para MDL?
