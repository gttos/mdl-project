# 2. Inventario maestro de datos

> **Estado:** actual, verificado contra el código de `api/` e `ios/` el 5 de agosto de 2026.
> **Objetivo:** identificar, entidad por entidad, qué información existe hoy, para qué se usa, dónde se guarda, quién puede leerla y cómo se elimina.
> **Criterio:** las calificaciones jurídicas quedan abiertas para la asesoría.

## 1. Leyenda de ubicaciones

| Código | Ubicación | Visibilidad |
|---|---|---|
| API-DB texto plano | Base relacional del backend, columna sin cifrar | Legible directamente por la aplicación servidor y por cualquiera con acceso a la base |
| API-DB cifrado (clave app) | Base relacional del backend, columna cifrada con `Crypt::encryptString` bajo la `APP_KEY` de Laravel | La aplicación servidor descifra automáticamente al leer la columna (mutator de Eloquent); una única clave protege el contenido de todas las cuentas |
| Vault-local | Base local del Vault en iOS (`Documents/vault.sqlite3`) | Ciphertext persistente con VMK; plaintext únicamente durante uso desbloqueado |
| Vault-remoto | Metadata en API-DB y ciphertext en almacenamiento de objetos (S3) | Metadata legible; contenido privado no legible sin la VMK del usuario |
| Índice local en claro | `vault_search_index` (FTS5) dentro de `vault.sqlite3` | Texto en claro, legible por cualquiera con acceso al archivo del dispositivo |
| Keychain | Almacén seguro del sistema iOS | Material criptográfico sujeto a controles del dispositivo |
| Solo local | Contenedor de la app en el dispositivo | No se transmite a MDL |
| Memoria | Memoria de ejecución de iOS o API | Temporal; debe descartarse al terminar la operación |
| Proveedor | Servicio externo necesario para una función | Depende del contrato y de la configuración real |

El ciphertext, las columnas cifradas con clave de aplicación y los identificadores se consideran vinculados a una cuenta. Ningún mecanismo actual anonimiza el conjunto.

## 2. Cuenta, perfil y autenticación

| Datos o campos | Persona | Origen y finalidad de producto | Ubicación y lectura | Protección | Conservación y borrado |
|---|---|---|---|---|---|
| ID interno de usuario | Usuario | Generado por MDL; relaciona todos los datos de la cuenta | API-DB texto plano | UUID, autorización por titular, HTTPS | Mientras la cuenta esté activa |
| Nombre completo y email | Usuario | Introducidos por el usuario; identificación, acceso, recuperación y soporte | API-DB texto plano; email también llega a Google (si usa OAuth) y a Sentry como identificador de contexto | HTTPS en tránsito | Hasta baja o plazo justificado |
| Teléfono | Usuario | Introducido por el usuario | API-DB cifrado (clave app) | `Crypt::encryptString` con `APP_KEY`; HTTPS en tránsito | Hasta actualización o baja |
| País, género, fecha de nacimiento y edad | Usuario | Introducidos o calculados; elegibilidad, personalización y métricas | API-DB texto plano | Control de acceso y HTTPS | Hasta actualización o baja |
| Plan, estado de tutorial, fecha de alta, actualización, último acceso e inactividad | Usuario | Generados por uso; habilitar funciones y operar la cuenta | API-DB texto plano | Control de acceso | Según vida de cuenta y plazos operativos |
| Password | Usuario | Introducida para autenticación | API-DB como hash (bcrypt); el valor original no se conserva | Hash de password; transporte HTTPS | Se sustituye al cambiarla; se elimina con la cuenta física |
| Token de acceso (Sanctum) y artefactos de sesión | Usuario/dispositivo | Generados para autenticar | API-DB (hash del token) y Keychain de iOS | Expiración a 30 días, revocación al cerrar sesión | `sanctum:prune-expired` corre diariamente |
| Código de un solo uso (OTP), hash, expiración, intentos | Usuario | Generados para recuperación de contraseña | API-DB; el código se guarda como hash | Límites de intento, cooldown progresivo (60s/300s/3600s), bloqueo tras 5 intentos | Purgado 24 horas después de expirar |
| Verificación de email, IP, user agent, trace ID, resultado y motivo de autenticación | Usuario/dispositivo | Generados por seguridad y diagnóstico | API-DB (`AuthEvent`), Sentry | Acceso restringido | 90 días en la regla técnica documentada; a confirmar contra el comando de purga real |
| Identificador federado de Google (`google_id`) | Usuario | Registro/login vía Google OAuth | API-DB texto plano | Consentimiento del usuario al pulsar "Continuar con Google" | Se conserva al anonimizar la cuenta según la lógica de negocio actual; a confirmar con ingeniería |

### Derechos y exportación

Los datos de cuenta son localizables por ID o email desde el backend. **No existe hoy un endpoint de exportación de datos personales del usuario.** El flujo de baja debe tratar también sesiones, proveedores y copias locales. Los detalles están en el [ciclo de vida](04_MDL_Ciclo_Vida_Retencion_Derechos.md).

## 3. Crush: datos en columnas funcionales de la API

| Datos o campos | Persona | Finalidad de producto | Ubicación y lectura | Protección | Conservación y borrado |
|---|---|---|---|---|---|
| ID del crush, ID de usuario, correlation ID | Usuario y tercera persona | Propiedad, idempotencia y relaciones | API-DB texto plano | Autorización por propietario, UUID y HTTPS | Vida de la entidad; soft delete |
| **Nombre** | Tercera persona | Identificar la ficha para el usuario | **API-DB texto plano — sin cifrar** | Solo control de acceso y HTTPS | Vida de la entidad; soft delete |
| Edad, país, género | Tercera persona | Filtros, presentación y métricas | API-DB texto plano | Control de acceso | Hasta edición o borrado |
| Fecha en que se conocieron (`met_at`) | Usuario y tercera persona | Timeline y métricas temporales | API-DB texto plano | Control de acceso | Vida de la entidad |
| Estado actual, estado de match, archivado | Usuario y tercera persona | Gestionar el ciclo relacional, listas y estadísticas | API-DB texto plano | Control de acceso | Vida de la entidad |
| Categoría afectiva subjetiva ("Ducky") | Usuario y tercera persona | Clasificación personal y métricas | API-DB texto plano | Control de acceso | Vida de la entidad |
| Altura, color de ojos y pelo, tipo corporal, color de piel | Tercera persona | Descripción estructurada, filtros y métricas | API-DB texto plano | Control de acceso | Vida de la entidad |
| Signo zodiacal (derivado), características seleccionadas | Tercera persona y opinión del usuario | Descripción y métricas | API-DB texto plano | Control de acceso | Vida de la entidad |
| Valoraciones de belleza, conexión emocional, personalidad, atracción sexual y valoración global (promedio) | Tercera persona y opinión del usuario | Reflexión personal, ordenación y métricas | API-DB texto plano | Control de acceso | Vida de la entidad |
| Email | Tercera persona | Contacto | API-DB cifrado (clave app) | `Crypt::encryptString` | Vida de la entidad |
| Prefijo y número de WhatsApp | Tercera persona | Contacto | API-DB cifrado (clave app) | `Crypt::encryptString` | Vida de la entidad |
| Instagram, Facebook, Snapchat, TikTok | Tercera persona | Contacto | API-DB cifrado (clave app) | `Crypt::encryptString` | Vida de la entidad |
| Fechas de creación, edición, última actividad y borrado | Usuario/dispositivo | Orden, sincronización, soporte y ciclo de vida | API-DB texto plano | Control de acceso | Según entidad |

### Copia adicional en el Vault (solo entradas migradas)

Cuando el usuario tiene el Vault activo y la entrada ya fue migrada, iOS mantiene además una copia cifrada con VMK del nombre, profesión, contacto, redes y fecha de nacimiento en `Vault-local`/`Vault-remoto`, y una copia en texto plano de nombre y profesión en el **índice local en claro** (FTS5) para búsqueda. Esta copia no reemplaza a las columnas de la sección anterior: el backend sigue teniendo acceso a la versión funcional.

Los endpoints funcionales de creación y edición de crush sí reciben hoy el nombre, contacto, redes y fecha de nacimiento —no están excluidos del transporte HTTP ni de la persistencia funcional—.

## 4. Experiencia

| Datos o campos | Persona | Finalidad de producto | Ubicación y lectura | Protección | Conservación y borrado |
|---|---|---|---|---|---|
| ID de experiencia, ID de usuario, ID de crush, correlation ID | Usuario y tercera persona | Propiedad, asociación e idempotencia | API-DB texto plano | Autorización por propietario | Vida de la experiencia; soft delete |
| Fecha | Usuario y tercera persona | Timeline, orden y métricas | API-DB texto plano | Control de acceso | Igual que la experiencia |
| Nivel de satisfacción | Usuario y opinión sobre la experiencia | Presentación y métricas | API-DB texto plano | Control de acceso | Igual que la experiencia |
| Lugar | Usuario y tercera persona | Contexto del evento | API-DB cifrado (clave app), normalizado en minúsculas | `Crypt::encryptString` | Igual que la experiencia |
| Descripción | Usuario y tercera persona | Relato del evento | API-DB cifrado (clave app) | `Crypt::encryptString` | Igual que la experiencia |

Cuando el Vault está activo y la entrada fue migrada, existe además una copia de lugar y descripción cifrada con VMK en Vault-local/remoto, y una copia en texto plano en el índice local en claro. Los formularios de experiencia envían lugar y descripción por HTTP en el mismo request que los campos funcionales.

## 5. Relación

| Datos o campos | Persona | Finalidad de producto | Ubicación y lectura | Protección | Conservación y borrado |
|---|---|---|---|---|---|
| ID de relación, ID de usuario, ID de crush, correlation ID | Usuario y tercera persona | Propiedad, asociación e idempotencia | API-DB texto plano | Autorización por propietario | Vida de la relación; soft delete |
| Estado, estado de match, fecha de inicio y fecha de fin | Usuario y tercera persona | Historial, timeline y métricas | API-DB texto plano | Control de acceso | Igual que la relación |
| ID de experiencia asociada | Usuario y tercera persona | Vincular el evento que origina o actualiza una relación | API-DB texto plano | Control de acceso | Igual que la relación |
| Nota de cierre | Usuario y tercera persona | Registrar cómo terminó la relación | API-DB cifrado (clave app) | `Crypt::encryptString` | Igual que la relación |

Cuando el Vault está activo y la entrada fue migrada, existe además una copia de la nota de cierre cifrada con VMK en Vault-local/remoto, y una copia en texto plano en el índice local en claro. Los formularios de relación envían la nota de cierre por HTTP en el mismo request que los campos funcionales.

## 6. Fotografías del crush

| Datos | Persona | Finalidad | Ubicación y lectura | Protección | Conservación y borrado |
|---|---|---|---|---|---|
| Imagen elegida por el usuario | Principalmente tercera persona | Galería privada de la ficha | Solo local, `Documents/LocalMedia/<crushId>/` | Sandbox de iOS; sin cifrado con VMK; sin exclusión de backup del sistema operativo | Hasta borrado individual manual; **no se elimina automáticamente al borrar el crush** |
| Archivo re-codificado y nombre aleatorio | Tercera persona | Reducir metadata innecesaria | Solo local | Re-codificación sin EXIF/GPS/IPTC | Igual que la imagen |

MDL no recibe las fotografías, no crea metadata remota sobre ellas y no puede recuperarlas desde el servidor. El código no marca `NSURLIsExcludedFromBackupKey`, por lo que estas fotos pueden quedar incluidas en un backup del sistema operativo del dispositivo (iCloud/iTunes) según la configuración del usuario. Al borrar un crush completo, la carpeta local de sus fotos permanece en el dispositivo.

## 7. Vault: claves, blobs y dispositivos (solo entradas migradas)

| Datos o campos | Persona | Finalidad | Ubicación y lectura | Protección | Conservación y borrado |
|---|---|---|---|---|---|
| ID del Vault, ID de usuario, versión criptográfica y estado | Usuario | Asociar y operar el Vault | API-DB texto plano | Autorización por titular | Vida del Vault |
| Algoritmo y parámetros de derivación (Argon2id), salt, VMK encapsulada, nonce y tag | Usuario | Desbloqueo y recuperación sin entregar la VMK en claro | API-DB; material técnico legible pero VMK no legible | Argon2id (65.536 KiB, 3 iteraciones, paralelismo 4) y cifrado autenticado | Vida de la encapsulación |
| VMK de 32 bytes | Usuario | Cifrar y descifrar el contenido del Vault | Keychain y memoria durante uso | `WhenUnlockedThisDeviceOnly` + biometría en dispositivo físico | Hasta limpieza del Vault/cuenta/dispositivo |
| Encapsulación por frase de 24 palabras | Usuario | Recuperación alternativa | Generada en iOS; **el endpoint `POST /api/vault/init` históricamente no la persistía** — corregido en código con una tabla y endpoints dedicados (`GET`/`PUT /vault/encapsulation/mnemonic`), pendiente de validación end-to-end en dispositivo | Cifrado autenticado | Vida de la encapsulación |
| ID aleatorio del blob, propietario, longitud, hash, ruta, secuencia y timestamps | Usuario y personas descritas | Integridad, versionado y sincronización | API-DB; metadata legible | Autorización por titular y URLs firmadas | Soft delete y purga tras 7 días (`vault:purge-blobs`, diario) |
| Ciphertext del blob (solo tipo `text` en uso activo) | Usuario y personas descritas | Copia remota cifrada del contenido ya migrado al Vault | Almacenamiento de objetos S3 | AES-256-GCM en cliente; VMK nunca llega al servidor | Según versión y purga |
| ID de dispositivo, huella, autorización, último uso, revocación | Usuario/dispositivo | Autorizar y gestionar dispositivos | API-DB texto plano | Claves asimétricas y revocación | Hasta revocación o baja |

## 8. Búsqueda y datos temporales en iOS

| Datos | Finalidad | Ubicación | Persistencia |
|---|---|---|---|
| Nombre, profesión, lugar, descripción y nota de cierre en texto plano | Búsqueda local por nombre e indexado | `vault_search_index` (FTS5) dentro de `vault.sqlite3` | **Persiste en el dispositivo en texto plano**; borrar una nota o entrada no siempre elimina la fila del índice |
| Término de búsqueda | Filtrar la lista | Memoria iOS | No se envía a la API |
| Pantallas y modelos con contenido descifrado | Presentar perfil y timeline | Memoria iOS | Durante la sesión; puede aparecer en logs locales de `LegacyMigrationOrchestrator` y `CrushViewModel` |

## 9. Notas seguras

| Datos | Persona | Finalidad | Ubicación | Sincronización remota |
|---|---|---|---|---|
| Texto libre de nota | Usuario | Registro privado adicional | `local_vault_notes`, cifrado con VMK, solo local | **Ninguna.** El método que las convertiría en entradas remotas no tiene invocadores en el código activo |

## 10. Timeline, actividad y métricas

| Datos | Persona | Origen y finalidad | Ubicación y lectura | Conservación |
|---|---|---|---|---|
| Tipo de evento e IDs de crush, experiencia o relación | Usuario y tercera persona | Generado por acciones; construir actividad y timeline | API-DB texto plano | Purga tras 7 días salvo los cinco eventos más recientes por usuario (documentado; confirmar comando real) |
| Métricas de atracción, satisfacción, evolución y relación | Usuario y terceras personas | Calculadas desde estados, fechas, atributos y valoraciones | API en memoria/respuesta; las fuentes permanecen en API-DB | Recalculables mientras existan las fuentes |

Las métricas no son anónimas por omitir nombres: se calculan para una cuenta y describen patrones íntimos del usuario o de terceras personas. La API no genera prosa ni interpretación causal; solo entrega datos y cruces numéricos para un posible motor externo (LangGraph), en preparación y sin conexión activa hoy.

## 11. Legal, comunicaciones, feedback y baja

| Datos o campos | Persona | Finalidad | Ubicación y lectura | Conservación y borrado |
|---|---|---|---|---|
| Versión y texto de términos o política, idioma y estado activo | Todos los usuarios | Presentar documentos vigentes | API-DB texto plano | — |
| Usuario, documento/versión, fecha, IP y user agent de aceptación | Usuario | Evidenciar qué documento se presentó/aceptó | API-DB texto plano | Sin poda automática identificada |
| Feedback de eliminación de cuenta: motivo elegido, texto libre opcional, estado y fecha | Usuario | Procesar y documentar la solicitud de baja | API-DB texto plano | Sin poda automática identificada |

## 12. Proveedores y datos recibidos

> Verificado contra `composer.json`, `config/services.php`, `config/mail.php` (API) y las dependencias/imports de Xcode (iOS) el 5 de agosto de 2026.

| Servicio | Proveedor | Datos que recibe | Finalidad | Estado |
|---|---|---|---|---|
| Hosting de API y base de datos | Amazon Web Services (RDS, MySQL, `eu-west-1`) | Todo el contenido de API-DB, incluyendo columnas cifradas con clave de aplicación | Prestar el backend | Activo |
| Almacenamiento de objetos | Amazon S3 (`eu-west-1`) | Ciphertext y metadata del Vault; media legacy si se usa ese flujo | Vault y almacenamiento privado | Activo |
| Autenticación federada | Google OAuth | Email, `google_id`, nombre (opcional) | Registro/login sin password | Activo (`config/services.php`) |
| Autenticación e identidad | Firebase (Core, Auth, App Check) | Identificadores de sesión/dispositivo y los datos que Firebase Auth gestione | Autenticación y verificación de integridad de la app | Activo en iOS (`GoogleService-Info.plist`, `FirebaseApp.configure()`) |
| Autenticación e identidad | AWS Amplify con plugin Cognito Auth | Sesión/identidad federada de Cognito | Autenticación | Activo en iOS (`configureAmplify()` en `AppDelegate`) — **coexiste con Firebase Auth**; revisar si ambos son necesarios |
| Almacenamiento | AWS Amplify con plugin S3 Storage | Objetos que la app suba por este canal | Almacenamiento | Plugin activo en iOS; confirmar con ingeniería qué flujo lo usa hoy, dado que el Vault sube directamente por URL firmada sin pasar por Amplify |
| Correo transaccional | Amazon SES | Email del usuario, para envío de OTP | Recuperación de contraseña | Configurado como mailer (`config/mail.php`) |
| Monitoreo de errores | Sentry (Functional Software, Inc.) | Stack traces, `trace_id`, `user_id`, contexto de request | Diagnóstico de errores en producción | Activo (`sentry/sentry-laravel`) |
| Caché de imágenes en cliente | Kingfisher (librería open source) | Imágenes descargadas de S3 | Caché local en el dispositivo del usuario | Activo en iOS; no envía datos a terceros |
| Motor de interpretación por IA | LangGraph (proyecto externo) | Payload estructurado de estadísticas, sin PII | Interpretación narrativa de estadísticas | En preparación; sin conexión activa desde la API hoy |
| Apple | Apple Inc. | Datos de distribución de la App Store | Distribución | Según declaración de privacidad de la tienda |

No debe añadirse un proveedor a la política por la mera presencia de una dependencia. Antes de producción, ingeniería debe confirmar si Firebase Auth y Amplify/Cognito Auth están ambos realmente en uso o si uno es remanente de una integración anterior, y qué flujo usa el plugin de almacenamiento de Amplify.

## 13. Campos que hoy siguen siendo legibles por la API fuera del Vault

| Entidad | Campos | Estado hoy |
|---|---|---|
| Crush | Nombre | Texto plano, sin cifrar |
| Crush | Email, WhatsApp, Instagram, Facebook, Snapchat, TikTok | Cifrados con la clave única de aplicación; descifrados automáticamente por el backend al leerlos |
| Experiencia | Lugar, descripción | Cifrados con la clave única de aplicación; descifrados automáticamente por el backend |
| Relación | Nota de cierre | Cifrada con la clave única de aplicación; descifrada automáticamente por el backend |

`CleanupLegacyDataJob`, encolado tras cada confirmación de migración al Vault, **solo registra su ejecución y no modifica ninguna de estas columnas.** No existe hoy una fecha ni un mecanismo activo para retirarlas.

## 14. Evidencia necesaria para validar este inventario

- confirmación de qué comandos de purga están realmente activos en producción y con qué frecuencia real se ejecutan (`model:prune`, `sanctum:prune-expired`, `accounts:process-inactive`, `vault:purge-blobs`);
- capturas sanitizadas de requests y responses de alta, edición, lectura y borrado, mostrando el contenido de contacto/lugar/descripción/nota viajando en texto plano por HTTPS;
- inspección de SQLite, Keychain y filesystem iOS, incluyendo el contenido real de `vault_search_index`;
- confirmación de qué plugin de Amplify se usa en producción (Auth, Storage, o ambos) y si Firebase Auth sigue activo en paralelo;
- matriz de permisos de producción y registro de accesos administrativos a la base de datos y a `APP_KEY`;
- confirmación de si `google_id` se conserva o se nulifica al anonimizar una cuenta.

Las preguntas jurídicas derivadas de este inventario están centralizadas en la [guía de auditoría](05_MDL_Auditoria_Juridica.md).
