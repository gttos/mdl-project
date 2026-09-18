# 2. Inventario maestro de datos

> Inventario técnico de la beta de lanzamiento, actualizado a 14 de septiembre de 2026. No asigna bases jurídicas ni convierte la retención técnica en un plazo aprobado.

## 1. Leyenda de ubicaciones

| Ubicación | Contenido | Visibilidad |
|---|---|---|
| API/RDS | cuenta, estructura funcional, eventos y metadata Vault | legible por la aplicación backend y personal autorizado según rol |
| S3 Vault | objetos ciphertext | no legible sin la VMK del usuario |
| SQLite iOS | ciphertext, metadata e intenciones durables | no legible sin VMK; sin índice privado persistente en claro |
| Keychain iOS | VMK y material local | protegido por controles del dispositivo |
| Sandbox local | fotos saneadas | legible en el dispositivo; no transmitido a MDL en beta |
| Memoria iOS | contenido descifrado durante uso | temporal en sesión desbloqueada |
| Proveedor | email, infraestructura u observabilidad | según servicio, contrato y configuración |

Ciphertext, identificadores opacos y estructura vinculada a una cuenta siguen siendo datos tratados por MDL; cifrado no equivale a anonimización.

## 2. Cuenta, perfil y autenticación

| Datos | Persona | Finalidad técnica | Ubicación/protección | Retención técnica o condición |
|---|---|---|---|---|
| UUID de usuario | Usuario | relacionar cuenta y ownership | API/RDS, legible | vida de cuenta; puede persistir temporalmente en logs/recibos tras baja |
| Email | Usuario | alta, login, verificación, recuperación y avisos | API/RDS, normalizado y legible para backend; TLS, acceso y cifrado de infraestructura por demostrar; proveedor de email cuando se envía | cuenta provisional: máximo siete días desde el alta si no verifica; no se conserva como lead; cuenta verificada: vida de cuenta, se sustituye en baja |
| Estado y plazo de verificación | Usuario | restringir una cuenta provisional y ejecutar su cierre si no verifica | API/RDS, sin contenido de email adicional | hasta verificar o hasta la baja automática de la cuenta provisional |
| Password hash | Usuario | autenticación | API/RDS, hash no reversible | vida de cuenta; se invalida/sustituye en baja |
| Nombre/perfil y teléfono, si se solicitan | Usuario | personalización/contacto | API/RDS; el teléfono se cifra como campo con `APP_KEY`; los demás atributos requieren inventario campo a campo | vida de cuenta o hasta modificación/baja |
| Fecha de nacimiento y edad | Usuario | mayoría de edad y perfil | API/RDS, legible para backend | vida de cuenta; se retira en baja |
| País y género del perfil | Usuario | perfil/localización y, si se usa, métricas | API/RDS, legible | vida de cuenta; minimización pendiente |
| Identidad federada Apple: `sub` cifrado y su digest, email/Private Relay, estado de relay y refresh token cifrado | Usuario | identificar de forma estable el vínculo Apple, iniciar sesión, gestionar Private Relay y revocar autorización al borrar la cuenta | API/RDS; `sub`, email y refresh token cifrados; digest sólo para búsqueda/unicidad; Apple recibe solicitudes de validación/revocación | mientras exista el vínculo; el token de revocación se elimina al completarse la revocación o, si falla definitivamente, tras 30 días desde el fallo final |
| Continuación Apple: digest de token y claims mínimos cifrados | Usuario | completar de forma explícita un alta o vínculo Apple validado | API/RDS; el token público no se persiste | 10 minutos o primer uso; después se purga |
| Token Sanctum y expiración | Usuario | sesión | API/RDS y Keychain/cliente | 7 días desde emisión o revocación anterior |
| OTP hash, expiración, intentos y lockout | Usuario | recuperación/verificación | API/RDS + email para el código en tránsito | código de 6 dígitos válido 15 minutos; máximo 5 intentos y bloqueo de 15 minutos; pruning programado |
| Último login/inactividad | Usuario | seguridad y cierre por inactividad | API/RDS | cierre tras 18 meses + 20 días de gracia según Product; plazo posterior a decidir |
| Versiones y fecha de aceptación | Usuario | demostrar términos/privacidad aceptados | API/RDS; puede incluir contexto técnico | plazo probatorio a aprobar |
| Motivo de baja, texto opcional, estado y recibo | Usuario | ejecutar baja y métricas de churn/compliance | API/RDS | plazo, acceso y necesidad del texto libre por aprobar |

La verificación de email forma parte de la beta. La cuenta nace provisional, sólo puede acceder al flujo de verificación y vence a los siete días desde el alta si no verifica; los reenvíos no extienden el plazo. Código, expiración, intentos, bloqueo, reenvío y cambio de email no conservan OTP en claro. Una cuenta provisional expirada se elimina y su email no se convierte en lead ni destinatario comercial sin una base y consentimiento específicos aprobados. El alta Apple con email acreditado nace verificada, no genera OTP y puede no tener contraseña MDL.

## 3. Crush: tercera persona descrita

### 3.1 Estructura visible para API

| Campo/categoría | Finalidad técnica | Riesgo o inferencia |
|---|---|---|
| UUID de crush y user ID | ownership, referencias y retry | vincula toda la historia con una cuenta |
| edad y `age_as_of` | perfil y cálculo temporal | edad de tercera persona; posible singularización |
| `gender_id` | filtro, perfil y estadísticas | atributo personal; proporcionalidad pendiente |
| país | mapa, filtros y estadísticas | ubicación aproximada/nacionalidad inferida |
| categoría Ducky | organización subjetiva | valoración o clasificación personal |
| `met_at` | límite cronológico y bundle inicial | revela fecha de encuentro |
| ratings de belleza, conexión, personalidad y atractivo | perfil y estadísticas | evaluación subjetiva potencialmente ofensiva o íntima |
| rating global | resumen derivado | inferencia basada en ratings |
| signo zodiacal | perfil/estadística, derivado en iOS | permite inferir parte del cumpleaños |
| altura, ojos, pelo, cuerpo, piel y otros rasgos | perfil/estadísticas | combinación singularizante; algunos atributos merecen revisión reforzada |
| fase/etiqueta vigentes, `everMatched` y correlación | Timeline/relaciones/estadísticas | inferencias afectivas o sexuales |
| archivo, anonimización técnica, actividad y timestamps | operación/ciclo de vida | patrones de uso y retención |

`gender_id` también puede aparecer dentro del snapshot cifrado para consistencia del cliente. La copia estructural de API es deliberada durante la beta y debe justificarse.

### 3.2 Contenido privado E2EE

| Contenido | Persistencia | Backend |
|---|---|---|
| nombre | SQLite ciphertext + S3 Vault ciphertext | no recibe plaintext en la beta |
| profesión | SQLite/S3 ciphertext | no recibe plaintext |
| email, WhatsApp/teléfono y prefijo | SQLite/S3 ciphertext | no recibe plaintext |
| Instagram, Facebook, Snapchat, TikTok u otras redes | SQLite/S3 ciphertext | no recibe plaintext |
| día y mes de cumpleaños | SQLite/S3 ciphertext | recibe sólo edad/ancla y signo derivado |
| otros detalles privados del perfil | SQLite/S3 ciphertext | no recibe plaintext si están incluidos en el contrato Vault |

## 4. Experiencias

| Datos | Ubicación | Finalidad | Riesgo |
|---|---|---|---|
| UUID, user/crush ID | API/RDS | ownership y asociación | vincula evento con cuenta y tercera persona |
| día civil | API/RDS | cronología/unicidad | frecuencia y patrón temporal |
| satisfacción | API/RDS | diario y estadísticas | valoración íntima/subjetiva |
| correlación, período e hito vinculados | API/RDS | agrupar Timeline y aplicar un cambio relacional atómico | revela contexto relacional |
| lugar | Vault E2EE | relato privado | ubicación o contexto sensible |
| descripción | Vault E2EE | relato privado | texto libre potencialmente muy sensible |

Sólo existe una Experience por crush y día civil. Puede vincular como máximo un cambio real del período en ese mismo día; ambos comparten correlación y fecha, lo que aumenta la capacidad de inferir contexto aun sin el texto.

## 5. Relaciones

### 5.1 Modelo de beta

| Datos | Ubicación | Finalidad | Riesgo |
|---|---|---|---|
| UUID, user/crush ID | API/RDS | ownership y asociación | vinculación del historial |
| fase de conexión | API/RDS | reglas, Timeline y estadísticas | puede revelar evolución afectiva o sexual |
| etiqueta de vínculo | API/RDS | clasificar opcionalmente un período Involucrado | inferencia íntima más precisa |
| historiales append-only de fase y etiqueta | API/RDS | reconstruir el recorrido sin sobrescribir | secuencia temporal singularizante |
| inicio y fin del período | API/RDS | orden y duración | cronología detallada y huecos entre períodos |
| motivo de cierre catalogado | API/RDS | cierre coherente y estadísticas | puede revelar rechazo, ausencia de match o fin del vínculo |
| `everMatched` | API/RDS | derivado monotónico y estadísticas | revela que algún período alcanzó Involucrado |
| Experience, correlación e hito vinculados | API/RDS | reconstrucción del bundle | contexto combinado |
| nota de cierre y explicación libre de “Otro” | Vault E2EE | relato privado | texto libre íntimo |

Las fases son Interesado, Conocido, Interés mutuo, Involucrado y Cerrado. Las etiquetas son Citas, Casual, FWB, Relación y Otro; sólo se seleccionan o cambian durante Involucrado y pueden conservarse como contexto histórico al cerrar. Los motivos nativos son `REJECTED`, `MISSED`, `ENDED` y `OTHER`.

### 5.2 Semántica e inferencias

Cada crush nace con un período inicial en `met_at`, que puede comenzar en cualquiera de las cinco fases. Un período vivo sólo conserva su fase o avanza, admite saltos sin fabricar hitos intermedios y, tras Cerrado, sólo continúa mediante un nuevo período explícito. Borrar una Experience vinculada conserva el hito; sólo el último período cerrado y no inicial admite borrado correctivo, y nunca reabre el anterior. `everMatched` sólo nace de haber alcanzado Involucrado y no se infiere desde texto, una Experience o una etiqueta concreta.

El runtime de beta no usa selector, dual running ni fallback al enum anterior. Cualquier conversión histórica permanece aislada del tratamiento ordinario y no condiciona las cuentas nuevas.

## 6. Vault, dispositivo y objetos

| Capa/dato | Contenido | Observación |
|---|---|---|
| VMK | clave aleatoria de 32 bytes | generada/usada en iOS; nunca en claro en backend |
| Encapsulación principal | KDF, salt, ciphertext, nonce/tag y versión | recovery por contraseña |
| Encapsulación mnemónica | parámetros y VMK encapsulada separadamente | recovery por frase de 24 palabras |
| Vault | owner, estado y crypto version | API/RDS |
| VaultEntityBinding | vault/user, tipo, entity ID y lookup token | metadata sensible; no reversible sin VMK |
| VaultBlob metadata | IDs, tipo, tamaño, hash, storage path, crypto version, secuencia, timestamps y soft delete | API/RDS; no incluir token/secretos en logs |
| VaultBlob object | snapshot ciphertext completo de una entidad | S3 privado mediante URL firmada |
| VaultDevice | ID, public-key hash/fingerprint, autorización, uso y revocación | metadata de dispositivo |
| VaultRecoveryKey/transferencia | fingerprint, estado y fechas | infraestructura; QR no es método normal de beta |
| SQLite Vault | entries, revisiones, nonce/tag, metadata e intenciones | cifrado y namespaced por cuenta |
| Keychain | VMK y material local | acceso bajo controles del dispositivo |
| Operación durable | tipo, scope, snapshot/IDs, estado de retry y error saneado | cifrada localmente hasta resolución |

El blob contiene un snapshot completo por entidad, no un blob por campo. Las revisiones antiguas sólo coexisten mientras se verifica la sustituta.

## 7. Fotos y datos locales

| Dato | Ubicación | Tratamiento de lanzamiento |
|---|---|---|
| foto original elegida | memoria temporal iOS | se procesa localmente |
| foto saneada | sandbox `LocalMedia/{crushId}` | sin metadata EXIF/GPS, sin sync MDL, excluida de backup no deseado |
| identificador/firma local | dispositivo | detectar duplicados y gestionar archivo |
| cache/thumbnail local, si existe | dispositivo | mismo ciclo de borrado que la foto/entidad |

No existe una entidad remota de media activa en beta. Las librerías que procesan localmente no son por ello destinatarios remotos.

## 8. Timeline, estadísticas e IA

| Actividad | Entradas | Salida/ubicación | Observación |
|---|---|---|---|
| Timeline | IDs, fechas, tipos, correlaciones y asociaciones | API/iOS | reconstruye eventos; no es fuente independiente |
| Dashboard personal | estructura, fases, etiquetas, `everMatched`, ratings, satisfacción y rasgos | API → iOS | única superficie estadística de beta; sólo datos de la cuenta y puede producir inferencias |
| Mapa | país del crush | API → iOS | coordenada representativa de país, no GPS |
| AI payload | métricas y señales estructuradas | endpoint API sin consumidor activo | no se envía a LLM en beta |
| Estadísticas comunitarias | no activas | ninguna | requerirían agregación/umbral y revisión previa |

`summary.winRate` usa Crushes visibles y no borrados que alcanzaron Involucrado sobre el total visible/no borrado; `summary.actualStatus` cuenta períodos vivos Involucrados con etiqueta Relación; `moment.activeCrushes` cuenta períodos vigentes vivos por fase. No se usan datos para entrenamiento de modelos generales. El prototipo LLM no está autorizado a recibir texto Vault ni datos reales hasta una decisión posterior.

## 9. Logs, auditoría y seguridad

| Categoría | Datos posibles | Regla técnica observada | Revisión necesaria |
|---|---|---|---|
| ActivityEvent | user/entity IDs, tipo y fecha | elegible tras 7 días salvo 5 eventos más recientes; se elimina con la entidad/baja según flujo | justificar o retirar la excepción de 5 |
| AuditEvent | user/target IDs, evento, trace ID, IP y user agent | 30 días | finalidad, acceso y plazo |
| AuthEvent | user/email hash o pivote, IP, user agent, resultado y fecha | 90 días en el modelo actual | campos exactos, antiabuso, derechos y coherencia con herramientas operativas |
| SecurityAlert | user/entity ID, IP, ruta, método, tipo, severidad, disposition y metadata | 30 días | minimizar mensaje/metadata |
| Logs API/CloudWatch | trace ID, ruta, método, status y error saneado | configuración real pendiente | retención, acceso, región y scrubbing |
| Sentry | user ID técnico, stack trace, ruta y trace ID | `send_default_pii` previsto en false; SQL bindings previstos en false | capturar evidencia de configuración efectiva |

Prohibido registrar VMK, contraseña/frase de recovery, tokens opacos, tokens de sesión, OTP en claro, payload privado, ciphertext completo o nombres de archivo aportados por el usuario.

## 10. Proveedores y destinatarios

| Servicio | Datos posibles | Finalidad y control documental |
|---|---|---|
| AWS RDS/EC2 | cuenta, estructura, eventos y metadata/encapsulaciones | infraestructura de beta; documentar entidad, región, cifrado de storage, backups, DPA, accesos y subencargados |
| AWS S3 | ciphertext, ruta, tamaño, hash y tiempos | Vault remoto; documentar región, versionado, lifecycle, backups y purga |
| AWS SES/mailer | email, OTP/mensaje, rebote y delivery | email transaccional; documentar proveedor efectivo, región, retención y subencargados |
| Sentry | diagnóstico técnico minimizado | observabilidad; documentar proyecto/región, retención, accesos y scrubbing |
| Notificaciones server-to-server Apple | digest de JWS, tipo, hora y digest de `sub`; nunca el JWS ni tokens | deduplicar y ordenar cambios de consentimiento o Private Relay; 30 días, salvo último evento aplicable mientras la identidad siga activa |
| Google OAuth | ninguno en beta | canal no expuesto; no forma parte del RoPA activo |
| Apple | `sub`, email o Private Relay, credenciales temporales y revocación; además distribución/Keychain | proveedor de identidad; documentar región/transferencias, términos, Private Email Relay, notificaciones y disclosure público |
| Proveedor LLM | ninguno en beta | no forma parte del RoPA activo hasta conexión autorizada |

## 11. Borrado, baja y datos conservados

- **Borrado de Experience/período:** soft delete funcional, blobs asociados y purga a siete días; eliminar una Experience conserva el hito relacional y sólo el último período cerrado y no inicial admite borrado correctivo, sin reabrir el anterior.
- **Borrado de crush:** incluye descendientes, bindings/blobs, entries/notas locales y fotos.
- **Baja de cuenta:** operación atómica y fail-closed (`DELETE /users`); revoca tokens; el Vault y sus blobs entran en soft-delete con la misma purga a siete días que el borrado dirigido (bindings retenidos hasta completarla); elimina de inmediato encapsulaciones, devices y recovery keys, y luego artefactos locales de iOS; sustituye identificadores directos o elimina la estructura según política. Si hay una identidad Apple, la baja retira el vínculo y crea una intención cifrada de revocación: cinco reintentos rápidos; ante fallo final, sólo ese token permanece hasta 30 días para un reintento manual autorizado y después se elimina definitivamente. La cuenta nunca se restaura por un fallo de Apple. Es irreversible; no existe reset de Vault independiente de la baja.
- **Cuenta provisional sin verificar:** vence siete días después del alta; se elimina con el flujo de baja aplicable, sin retener el email como lead. No se genera Vault antes de verificar.
- **Backups/versiones:** pueden expirar en otro plazo; debe conocerse y explicarse el tratamiento durante restauraciones.
- **Estadísticas posteriores:** conservar filas individuales desidentificadas puede constituir seudonimización. No calificarlas como anónimas sin decisión jurídica y prueba de reidentificación.
- **Feedback y auditoría:** pueden sobrevivir transitoriamente a la baja por finalidades aprobadas, con acceso y plazo propios.

## 12. Decisiones pendientes

- finalidades y bases por actividad, dato y persona;
- deber de información respecto de terceras personas y posibles excepciones;
- calificación de fases, etiquetas, cierres, historiales relacionales, inferencias y categorías especiales;
- necesidad de EIPD;
- minimización de género, edad, país, rasgos, ratings, fases, etiquetas, cierres y correlaciones;
- calificación, finalidad, base, plazo, acceso y salvaguardas de la estructura desidentificada conservada para estadísticas;
- retención exacta de datos activos, soft deletes, eventos, logs, OTP, backups y proveedores;
- acceso, rectificación, oposición, restricción, portabilidad y supresión cuando backend no descifra Vault;
- proveedores, regiones, transferencias, DPA y subencargados reales;
- disclosure y documentación de verificación de email y Sign in with Apple; control de cambio para Google OAuth, IA y cualquier función futura.
