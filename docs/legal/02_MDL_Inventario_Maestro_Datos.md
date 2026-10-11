# 2. Inventario maestro de datos

> Inventario funcional objetivo de la beta, actualizado a 21 de septiembre de 2026. No asigna bases jurídicas, no convierte una retención técnica en plazo aprobado y no certifica que cada protección esté desplegada.

## 1. Leyenda de ubicaciones

| Ubicación | Contenido | Visibilidad |
|---|---|---|
| API/RDS | cuenta, estructura funcional, eventos y metadata Vault | legible por backend y personal autorizado según rol; los campos cifrados sólo se recuperan para finalidades autorizadas |
| S3 Vault | objetos ciphertext | no legible sin la VMK de la cuenta |
| SQLite local | ciphertext, metadata e intenciones durables | no legible sin VMK; sin índice privado persistente en claro |
| Almacén seguro | VMK y material local | protegido por controles del dispositivo |
| Sandbox local | fotos saneadas | legible exclusivamente en el dispositivo durante la beta |
| Memoria del cliente | contenido descifrado durante uso | temporal en sesión desbloqueada |
| Proveedor | email, infraestructura, identidad u observabilidad | según servicio, contrato y configuración |

Ciphertext, identificadores opacos y estructura vinculada a una cuenta siguen siendo datos tratados por MDL; cifrado no equivale a anonimización.

## 2. Cuenta, perfil y autenticación

| Datos | Tratamiento y finalidad | Condición o retención funcional |
|---|---|---|
| `id` | UUID legible para propiedad y asociaciones | vida de la cuenta; destino posterior sujeto a política aprobada |
| `birth_date` | fecha completa declarada para mayoría de edad y perfil | obligatoria; se retira en la baja |
| `gender_id`, `country_id` | perfil opcional de la persona usuaria | editables y retirados en la baja |
| `plan_id` | nivel administrado por el sistema | beta gratuita |
| `email` | cifrado pero recuperable por API para alta, acceso, verificación, recuperación, cambio y avisos autorizados | provisional: máximo siete días si no verifica; verificada: vida de cuenta |
| copias operativas del email | la dirección provisional, la nueva dirección pendiente, la asociada a verificaciones o recuperaciones y la personal o intermediaria recibida de Apple siguen el mismo cifrado recuperable | sólo durante el estado o finalidad que las origina; colas y reintentos no dejan una copia durable legible |
| `email_lookup` | índice exacto no reversible para búsqueda y unicidad | nunca se devuelve al cliente; se retira en la baja |
| `password` | verificador no reversible; puede faltar en una cuenta Apple | vida del método de acceso; se sustituye o retira según el flujo |
| `email_verified_at`, `provisional_expires_at` | verificación y gate provisional | hasta verificar o ejecutar la baja provisional |
| `created_at`, `updated_at` | alta y modificación técnica | no sustituyen fechas funcionales |
| `last_login`, `inactivity_notified_at` | autenticación correcta y proceso de inactividad | rigen 18 meses + 20 días conforme a Producto |
| `deleted_at` | baja funcional | impide uso ordinario |
| flags de tutorial | recordar tutoriales completados | vida de la cuenta |
| sesiones y tokens | autenticación | siete días desde emisión o revocación anterior |
| OTP, intentos y bloqueo | verificación y recuperación | OTP de 15 minutos; cinco intentos y bloqueo de 15 minutos; pruning programado |
| aceptaciones | versión y fecha de términos/privacidad | plazo probatorio pendiente de Legal |
| feedback de beta | hasta 100 registros por cuenta con `id`, `user_id`, `context`, timestamps y metadata técnica cerrada: `platform`, `app_version`, `build_number`, `os_version` y `screen`; no admite claves adicionales, contenido libre, identificadores de entidades o dispositivos, email, tokens, requests, cabeceras ni trazas | sólo durante la beta; al llegar a 100 se rechazan nuevos envíos sin borrar los anteriores; acceso exclusivo del responsable del producto mediante consulta directa a RDS, sin endpoint de listado; purga íntegra dentro de los 30 días posteriores al fin de la beta si antes no se aprueba una política sustitutiva |
| feedback y recibo de baja asociados a la baja de cuenta | ejecutar, confirmar y analizar la baja dentro de finalidades aprobadas | campos, acceso y plazo pendientes de Legal |

Nombre, edad calculada y teléfono no forman parte del perfil de cuenta. El idioma es una preferencia local del cliente.

### Sign in with Apple

API puede conservar `sub` protegido, digest de búsqueda, email personal o Private Relay cifrado, estado y material técnico cifrado de revocación. Una continuación de alta o vínculo dura diez minutos y un solo uso. Apple se ofrece sólo en iOS durante la beta; Android utiliza email y contraseña. Una cuenta Apple puede establecer la primera contraseña mediante su email verificado sin crear otro `User` ni Vault.

## 3. Crush: tercera persona descrita

### 3.1 Estructura legible por API

| Campo/categoría | Finalidad funcional | Riesgo o inferencia |
|---|---|---|
| `id`, `user_id` | identidad y propiedad | vincula toda la historia con una cuenta |
| `country_id` | perfil, mapa y estadísticas personales | ubicación o nacionalidad aproximada |
| `ducky_id` | categoría subjetiva elegida | clasificación personal |
| `age`, `age_as_of` | edad confirmada y cálculo temporal | edad de tercera persona; posible singularización |
| `met_at` | inicio cronológico y bloque inicial | fecha de encuentro |
| `match_status_id` | `UNKNOWN`, `MATCH` o `NO_MATCH` para filtros y estadísticas | inferencia afectiva o sexual |
| `first_match_situation_id`, `first_match_date` | primera evidencia de match | cronología íntima más precisa |
| `zodiac_id` | derivado del cumpleaños Vault | permite inferir un intervalo del cumpleaños |
| `height_id`, `eye_color_id`, `hair_color_id`, `body_type_id`, `skin_color_id` | perfil y estadísticas personales | combinación singularizante y potencialmente discriminatoria |
| cuatro ratings y `rate_global` | valoración subjetiva y estadísticas | evaluaciones potencialmente ofensivas o íntimas |
| archivo, seudonimización, correlación y timestamps | operación, retry y ciclo de vida | patrones de uso y retención |

### 3.2 Contenido Vault E2EE

| Contenido | Condición | Backend |
|---|---|---|
| nombre | obligatorio, hasta 191 caracteres | no recibe plaintext |
| género del Crush | obligatorio, ID de catálogo | no recibe el valor legible |
| profesión | opcional, hasta 191 caracteres | no recibe plaintext |
| WhatsApp | prefijo de país y número de hasta 20 dígitos | no recibe plaintext |
| Instagram, Facebook, Snapchat y TikTok | cuatro textos opcionales | no recibe plaintext |
| cumpleaños | día y mes, sin año | no recibe el cumpleaños; puede recibir `zodiac_id` derivado |

Las fotos son exclusivamente locales y se tratan en §7.

## 4. Experiencias

| Datos | Ubicación | Condición y finalidad |
|---|---|---|
| `id`, `user_id`, `crush_id` | API/RDS | identidad, propiedad y asociación |
| `date` | API/RDS | día civil y cronología |
| `satisfaction_id` | API/RDS | obligatorio, entero 1–5; diario y estadísticas |
| `correlation_id` | API/RDS | unir partes de una acción compuesta |
| `relationship_period_id` | API/RDS | obligatorio; toda experiencia pertenece a un período |
| hito derivado | API/RDS | existe sólo cuando un hito apunta mediante su `experience_id` |
| `place`, `description` | Vault E2EE | ambos obligatorios; únicos textos privados de la experiencia |

Existe como máximo una experiencia por Crush y día civil. Crushes distintos pueden registrar experiencias el mismo día. La experiencia inicial es obligatoria y forma parte del bloque atómico del alta. Una experiencia puede ser independiente dentro de su período o vincularse a un único hito.

## 5. Relaciones

### 5.1 Situaciones

`RelationshipSituation` es un Value Object, no una tabla. API guarda el entero:

| IDs | Significado | Evidencia de match |
|---|---|---|
| 1 `NOT_TALKING`, 2 `ACQUAINTED`, 3 `FRIENDS`, 4 `MUTUAL_INTEREST` | situaciones previas al match | no lo establecen |
| 5 `DATING`, 6 `CASUAL`, 7 `FWB`, 8 `RELATIONSHIP`, 9 `UNDEFINED` | situaciones que evidencian match | establecen `MATCH` |
| 10 `CLOSED` | cierre del período | no crea ni borra match por sí solo |

La situación relacional es una única dimensión y el resultado acumulado de match pertenece al Crush.

### 5.2 Período relacional

| Datos legibles | Finalidad |
|---|---|
| `id`, `user_id`, `crush_id` | identidad y propiedad |
| `start_date`, `end_date` | inicio y cierre del tramo |
| `current_situation_id` | situación vigente |
| `correlation_id`, timestamps y `deleted_at` | atomicidad, orden y ciclo de vida |

Un período no tiene payload Vault propio.

### 5.3 Hito relacional

| Datos legibles | Finalidad |
|---|---|
| `id`, `relationship_period_id` | identidad y pertenencia |
| `experience_id` | vínculo obligatorio y único con la experiencia que explica el acontecimiento |
| `sequence`, `effective_date` | orden y día efectivo |
| `previous_situation_id`, `situation_id` | cambio exacto registrado |
| `closure_reason_id` | motivo catalogado sólo cuando resulta `CLOSED` |
| `correlation_id` y capacidades derivadas | acción compuesta y acciones `latest-only` permitidas |

El motivo de cierre es también un catálogo cerrado y puede revelar el resultado del vínculo:

| ID y clave | Situación anterior admitida |
|---|---|
| 1 `NO_MUTUAL_INTEREST` | situaciones 1–4 |
| 2 `REJECTED_BY_ME`, 3 `REJECTED_BY_THEM`, 4 `FADED` | situaciones 1–9 |
| 5 `MISSED_OPPORTUNITY` | situaciones 1–4 |
| 6 `ENDED_DATING` | `DATING` |
| 7 `ENDED_CASUAL` | `CASUAL` |
| 8 `ENDED_FWB` | `FWB` |
| 9 `ENDED_RELATIONSHIP` | `RELATIONSHIP` |
| 10 `ENDED_UNDEFINED` | `UNDEFINED` |
| 11 `UNSPECIFIED` | situaciones 1–9 |

El hito tampoco tiene payload Vault ni texto propio; el relato del cambio o cierre vive en `description` de su experiencia.

### 5.4 Borrado relacional

- Eliminar sólo un último hito posterior puede conservar su experiencia como independiente.
- Eliminar desde una experiencia vinculada elimina conjuntamente experiencia e hito si cumple `latest-only`.
- Un período posterior sólo se elimina cuando su hito inicial es su único hito; elimina también todas sus experiencias y nunca reabre el anterior.
- El período inicial sólo desaparece al eliminar el Crush.

## 6. Vault, dispositivo y objetos

| Capa/dato | Contenido | Observación |
|---|---|---|
| VMK | clave aleatoria de 32 bytes | generada/usada por el cliente; nunca en claro en backend |
| Encapsulación principal y mnemónica | parámetros, ciphertext, nonce/tag y versión | recovery por contraseña o frase de 24 palabras |
| Vault | owner, estado y versión criptográfica | API/RDS |
| Binding | Vault/cuenta, tipo, entity ID y lookup token | metadata sensible; sólo para Crush y Experience |
| Metadata de blob | IDs, tipo, tamaño, hash, ruta, versión, secuencia, timestamps y soft delete | API/RDS; sin contenido privado |
| Objeto de blob | snapshot ciphertext completo de una entidad | S3 privado mediante URL firmada |
| Dispositivo | fingerprint/hash, autorización, uso y revocación | un único dispositivo activo en beta |
| Operación durable | identidad, scope, snapshot cifrado, retry y error saneado | local hasta resolución |

Cada objeto ciphertext admite como máximo 65.536 bytes. La capacidad lógica remota es 25 MiB por cuenta, con aviso al 80 %. Una subida no confirmada reserva sólo el crecimiento correspondiente, caduca a las 24 horas y deja de consumir cuota; la limpieza automática retira objeto y metadata incompletos.

## 7. Fotos y datos locales

| Dato | Ubicación | Tratamiento de beta |
|---|---|---|
| foto elegida | memoria temporal | procesamiento local |
| foto saneada | `LocalMedia/{crushId}` o equivalente local | almacenamiento local, exclusión de backup no deseado y ciclo del Crush |
| firma, cache o thumbnail | dispositivo | mismo ciclo de borrado que la foto o entidad |

## 8. Timeline y estadísticas

| Actividad | Entradas | Resultado |
|---|---|---|
| Timeline/Home | IDs, fechas, tipos, correlaciones y asociaciones | API/cliente; contenido privado resuelto desde Vault |
| Dashboard personal | match, situaciones, satisfacción, ratings, rasgos, zodíaco, países y Ducky | API → cliente; sólo datos de la cuenta |
| Mapa | `country_id` del Crush | coordenada representativa del país, no GPS |

La beta no combina cuentas ni ofrece rankings competitivos, body count, predicciones de fracaso, percentiles de superioridad, valor de mercado o tracking individual de ghosting. La estructura conservada tras la baja no entra por sí sola en ninguna estadística global.

## 9. Logs, auditoría y seguridad

| Categoría | Datos permitidos | Decisión pendiente |
|---|---|---|
| `ActivityEvent` | identidades, tipo, fecha y asociaciones mínimas | necesidad y plazo exactos |
| `AuditEvent` | IDs técnicos, evento, traza y contexto autorizado | finalidad, acceso y plazo |
| `AuthEvent` | pivote protegido, IP, user agent, resultado y fecha | inventario, antiabuso y plazo |
| `SecurityAlert` | IDs técnicos, IP, ruta, método, tipo y disposición | minimización, acceso y plazo |
| Logs/Sentry | trace ID, plantilla de ruta, método, status, duración, versión y error saneado | configuración, región, acceso y retención |

Está prohibido registrar email legible, VMK, secretos de recovery, tokens, OTP en claro, URLs firmadas completas, lookup tokens, contenido Vault, snapshots privados, ciphertext completo o payloads completos.

## 10. Proveedores y destinatarios

| Servicio | Datos posibles | Control pendiente |
|---|---|---|
| AWS RDS/EC2 | cuenta, estructura, eventos y metadata/encapsulaciones | entidad, región, cifrado, backups, accesos, DPA y subencargados |
| AWS S3 | ciphertext, ruta, tamaño, hash y tiempos | región, versionado, lifecycle, backups y purga |
| SES u otro mailer | email, OTP, aviso, entrega y rebote | proveedor, región, retención y subencargados |
| Sentry | diagnóstico técnico minimizado | proyecto/región, retención, accesos y scrubbing |
| Apple | identidad federada usada desde iOS, email/Private Relay, eventos y revocación | transferencias, términos, disclosure y retención |

## 11. Baja y datos conservados

La baja revoca sesiones, retira email e índice, password, nacimiento, género/país de cuenta, métodos de acceso y material de recovery. El Vault entra en retirada inmediata y su ciphertext se purga después de siete días. El cliente elimina claves, base local, pendientes, preferencias sensibles y fotos después de confirmar el resultado remoto.

Producto pretende conservar fuera del uso ordinario estructura seudonimizada de Crushes, experiencias y relaciones para posibles agregados. Esto no autoriza todavía su uso: Legal debe decidir calificación, finalidad, base, campos, plazo, acceso, separación, riesgo de reidentificación y salvaguardas. Si no existe justificación suficiente, esa estructura deberá eliminarse.

## 12. Decisiones jurídicas pendientes

- finalidades y bases por actividad, dato y persona;
- deber de información respecto de terceras personas y posibles excepciones;
- calificación de situaciones, match, cierres, hitos, experiencias, satisfacción, ratings e inferencias;
- necesidad de EIPD;
- minimización de edad, país, rasgos, ratings, situaciones, match, cierres y correlaciones;
- tratamiento post-baja y riesgo de reidentificación;
- retención exacta de eventos, logs, aceptaciones, recibos de baja, backups y proveedores; el feedback general conserva únicamente la regla provisional de beta y requiere una política sustitutiva antes del lanzamiento público;
- acceso, rectificación, oposición, restricción, portabilidad y supresión cuando backend no descifra Vault;
- proveedores, regiones, transferencias, DPA y subencargados reales;
- transparencia sobre email cifrado, Vault, fotos locales, inactividad, Apple sólo en iOS y Android con email/contraseña.
