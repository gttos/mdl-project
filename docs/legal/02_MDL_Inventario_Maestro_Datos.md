# 2. Inventario maestro de datos

> Inventario técnico del modelo objetivo de lanzamiento, actualizado a 2 de septiembre de 2026. No asigna bases jurídicas ni convierte la retención técnica en un plazo aprobado.

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
| Google ID/token/callback | Usuario | OAuth condicional | API/proveedor Google | sólo si el canal se habilita; nulificación y retención por confirmar |
| Identidad federada Apple: `sub` cifrado y su digest, email/Private Relay, estado de relay y refresh token cifrado | Usuario | identificar de forma estable el vínculo Apple, iniciar sesión, gestionar Private Relay y revocar autorización al borrar la cuenta | API/RDS; `sub`, email y refresh token cifrados; digest sólo para búsqueda/unicidad; Apple recibe solicitudes de validación/revocación | mientras exista el vínculo; el token de revocación se elimina al completarse la revocación o, si falla definitivamente, tras 30 días desde el fallo final |
| Continuación Apple: digest de token y claims mínimos cifrados | Usuario | completar de forma explícita un alta o vínculo Apple validado | API/RDS; el token público no se persiste | 10 minutos o primer uso; después se purga |
| Token Sanctum y expiración | Usuario | sesión | API/RDS y Keychain/cliente | 7 días desde emisión o revocación anterior |
| OTP hash, expiración, intentos y lockout | Usuario | recuperación/verificación | API/RDS + email para el código en tránsito | ventana operativa y pruning a demostrar |
| Último login/inactividad | Usuario | seguridad y cierre por inactividad | API/RDS | cierre tras 18 meses + 20 días de gracia según Product; plazo posterior a decidir |
| Versiones y fecha de aceptación | Usuario | demostrar términos/privacidad aceptados | API/RDS; puede incluir contexto técnico | plazo probatorio a aprobar |
| Motivo de baja, texto opcional, estado y recibo | Usuario | ejecutar baja y métricas de churn/compliance | API/RDS | plazo, acceso y necesidad del texto libre por aprobar |

La verificación de email es una iniciativa activa. La cuenta nace provisional, sólo puede acceder al flujo de verificación y vence a los siete días desde el alta si no verifica; los reenvíos no extienden el plazo. Código de seis dígitos, expiración, intentos, bloqueo, reenvío y cambio de email no deben conservar códigos en claro. Una cuenta provisional expirada se elimina y su email no se convierte en lead ni destinatario comercial sin una base y consentimiento específicos aprobados.

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
| estado actual, match status y correlación | Timeline/relaciones/estadísticas | inferencias afectivas o sexuales |
| archivo, anonimización técnica, actividad y timestamps | operación/ciclo de vida | patrones de uso y retención |

`gender_id` también puede aparecer dentro del snapshot cifrado para consistencia del cliente. La copia estructural de API es deliberada durante la beta y debe justificarse.

### 3.2 Contenido privado E2EE

| Contenido | Persistencia | Backend |
|---|---|---|
| nombre | SQLite ciphertext + S3 Vault ciphertext | no recibe plaintext en el modelo final |
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
| correlación y relación vinculada | API/RDS | agrupar Timeline | revela contexto relacional |
| lugar | Vault E2EE | relato privado | ubicación o contexto sensible |
| descripción | Vault E2EE | relato privado | texto libre potencialmente muy sensible |

Sólo existe una experiencia por crush y día civil. Una experiencia vinculada a una relación comparte correlación y fecha, lo que aumenta la capacidad de inferir contexto aun sin el texto.

## 5. Relaciones

### 5.1 Modelo de beta

| Datos | Ubicación | Finalidad | Riesgo |
|---|---|---|---|
| UUID, user/crush ID | API/RDS | ownership y asociación | vinculación del historial |
| estado relacional | API/RDS | reglas, Timeline y estadísticas | puede revelar vida afectiva/sexual o rechazo |
| inicio y fin | API/RDS | orden y duración | cronología detallada |
| match status | API/RDS | estado derivado/estadísticas | inferencia persistente y monotónica |
| experiencia y correlación vinculadas | API/RDS | reconstrucción del bundle | contexto combinado |
| nota de cierre | Vault E2EE | relato privado | texto libre íntimo |

Los estados de beta son Prospecto, Ligue, One-night stand, FWB, Relación, Rechazado, No hubo match y Terminada.

### 5.2 Rediseño futuro, no activo

Si se implementa el rediseño conceptual, se añadirían o transformarían:

- `connection_phase`;
- `connection_label`;
- historial append-only de cambios de etiqueta;
- `closure_reason`;
- períodos separados de cierre/reapertura;
- fecha de corte de comparabilidad estadística;
- tipos de experiencia si se promueve esa parte del diseño.

Estos datos no forman parte del tratamiento activo. Antes de incorporarlos se debe revisar necesidad, granularidad, categorías especiales/inferencias, transparencia, base, conservación y derechos.

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
| Dashboard personal | estructura, estados, ratings, satisfacción, rasgos | API → iOS | sólo datos de la cuenta; puede producir inferencias |
| Mapa | país del crush | API → iOS | coordenada representativa de país, no GPS |
| AI payload | métricas y señales estructuradas | endpoint API sin consumidor activo | no se envía a LLM en beta |
| Estadísticas comunitarias | no activas | ninguna | requerirían agregación/umbral y revisión previa |

No se usan datos para entrenamiento de modelos generales. El prototipo LLM no está autorizado a recibir texto Vault ni datos reales hasta una decisión posterior.

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

## 10. Proveedores y destinatarios a confirmar

| Servicio | Datos posibles | Estado documental |
|---|---|---|
| AWS RDS/EC2 | cuenta, estructura, eventos y metadata/encapsulaciones | entidad, región, cifrado de storage, backups, DPA, accesos y subencargados pendientes |
| AWS S3 | ciphertext, ruta, tamaño, hash y tiempos | región, versionado, lifecycle, backups y purga pendientes de evidencia |
| AWS SES/mailer | email, OTP/mensaje, rebote y delivery | proveedor efectivo, región, retención y subencargados pendientes |
| Sentry | diagnóstico técnico minimizado | proyecto/región, retención, accesos y scrubbing pendientes |
| Notificaciones server-to-server Apple | digest de JWS, tipo, hora y digest de `sub`; nunca el JWS ni tokens | deduplicar reentregas y ordenar cambios de consentimiento o Private Relay | 30 días; el último evento aplicable permanece sólo con la identidad Apple activa |
| Google OAuth | identidad federada y tokens | condicional a estar expuesto en iOS; alcance/retención/transferencias pendientes |
| Apple | autenticación federada: `sub`, email o Private Relay, credenciales temporales y revocación; además distribución/Keychain según plataforma | proveedor de identidad; confirmar región/transferencias, términos, Private Email Relay, endpoint de notificaciones y disclosure público antes del rollout |
| Proveedor LLM | ninguno en beta | no incorporar al RoPA activo hasta conexión autorizada |

## 11. Borrado, baja y datos conservados

- **Borrado de experiencia/relación:** soft delete funcional, blobs asociados y purga a siete días.
- **Borrado de crush:** incluye descendientes, bindings/blobs, entries/notas locales y fotos.
- **Baja de cuenta:** operación atómica y fail-closed (`DELETE /users`); revoca tokens; el Vault y sus blobs entran en soft-delete con la misma purga a siete días que el borrado dirigido (bindings retenidos hasta completarla); elimina de inmediato encapsulaciones, devices y recovery keys, y luego artefactos locales de iOS; sustituye identificadores directos o elimina la estructura según política. Si hay una identidad Apple, la baja retira el vínculo y crea una intención cifrada de revocación: cinco reintentos rápidos; ante fallo final, sólo ese token permanece hasta 30 días para un reintento manual autorizado y después se elimina definitivamente. La cuenta nunca se restaura por un fallo de Apple. Es irreversible; no existe reset de Vault independiente de la baja.
- **Cuenta provisional sin verificar:** vence siete días después del alta; se elimina con el flujo de baja aplicable, sin retener el email como lead. No debe existir Vault generado antes de verificar.
- **Backups/versiones:** pueden expirar en otro plazo; debe conocerse y explicarse el tratamiento durante restauraciones.
- **Estadísticas posteriores:** conservar filas individuales desidentificadas puede constituir seudonimización. No calificarlas como anónimas sin decisión jurídica y prueba de reidentificación.
- **Feedback y auditoría:** pueden sobrevivir transitoriamente a la baja por finalidades aprobadas, con acceso y plazo propios.

## 12. Decisiones pendientes

- finalidades y bases por actividad, dato y persona;
- deber de información respecto de terceras personas y posibles excepciones;
- calificación de estados relacionales, inferencias y categorías especiales;
- necesidad de EIPD;
- minimización de género, edad, país, rasgos, ratings, estados y correlaciones;
- estrategia estadística tras la baja: agregación irreversible o retención seudonimizada;
- retención exacta de datos activos, soft deletes, eventos, logs, OTP, backups y proveedores;
- acceso, rectificación, oposición, restricción, portabilidad y supresión cuando backend no descifra Vault;
- proveedores, regiones, transferencias, DPA y subencargados reales;
- condición de lanzamiento para OAuth, verificación de email y cualquier función futura.
