# 1. Contexto del producto y sus entidades

> Descripción del modelo funcional objetivo de la beta a 21 de septiembre de 2026. Los campos y ubicaciones exactos están en el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md). Este documento describe hechos de Producto; no asigna bases jurídicas ni acredita implementación.

## 1. Qué es MDL

MDL es una aplicación nativa de diario privado para personas de 18 años o más. La beta contempla iOS y Android y permite registrar Crushes, experiencias, evolución relacional, valoraciones subjetivas, fotografías locales y estadísticas personales.

No es una red social. La tercera persona descrita no crea necesariamente una cuenta, no recibe un perfil público, no participa en el registro y puede desconocer su existencia. Todo el contenido lo introduce y consulta la persona titular de la cuenta.

## 2. Personas afectadas

| Persona | Relación con MDL | Datos principales |
|---|---|---|
| Persona usuaria | Crea y usa una cuenta | identidad, contacto, autenticación, actividad, diario, valoraciones y estadísticas |
| Crush o tercera persona | Es descrita en el diario | perfil privado, rasgos, edad, país, fechas, situaciones, match, cierres, experiencias, fotos y valoraciones |
| Personal autorizado | Opera seguridad, infraestructura o soporte | acceso limitado y trazado a estructura, metadata y logs según rol; no dispone de una función de descifrado del Vault |
| Proveedores | Prestan infraestructura, identidad, email u observabilidad | ciphertext, cuenta, email o telemetría según el servicio, contrato y configuración real |

«Crush» es el nombre de producto de una ficha privada. No implica consentimiento, conocimiento ni relación contractual de esa tercera persona con MDL.

## 3. Recorrido principal

### 3.1 Cuenta, acceso y Vault

La persona crea una cuenta mediante email y contraseña o, sólo en iOS durante la beta, Sign in with Apple. Todo registro exige fecha completa de nacimiento declarada, mayoría de edad y aceptación de las versiones vigentes de términos y privacidad.

El alta por email nace provisional: durante siete días sólo puede verificar, reenviar, consultar estado o cerrar sesión. Verificar el OTP habilita la cuenta; vencer sin verificar produce su baja íntegra y libera el email. El alta Apple con email acreditado nace verificada y puede no tener contraseña MDL. Un `sub` Apple no se fusiona automáticamente por coincidencia de email: la vinculación exige acreditar expresamente la cuenta MDL existente.

Android ofrece email y contraseña. Una cuenta creada con Apple puede establecer su primera contraseña mediante su email vigente y verificado y acceder desde Android al mismo `User` y Vault, sin retirar el vínculo Apple.

El perfil de cuenta contiene `birth_date` y, opcionalmente, `gender_id` y `country_id`. El email es obligatorio y se conserva cifrado pero recuperable por la API únicamente para autenticación, verificación, recuperación, cambio de email y comunicaciones autorizadas. Un índice exacto no reversible permite búsqueda y unicidad. Nombre y teléfono no forman parte del perfil de cuenta.

Con la identidad verificada, el cliente configura el Vault: genera una VMK aleatoria por cuenta, la encapsula para recovery y conserva el material local bajo el namespace de esa cuenta. El servidor almacena estructura, encapsulaciones, bindings, metadata y ciphertext, pero no recibe la VMK en claro ni dispone de una operación para descifrar el contenido.

La beta contempla un único dispositivo activo por Vault. En un dispositivo sin estado local, la aplicación consulta primero API: un Vault remoto existente conduce a recovery y nunca a crear otro que lo sobrescriba.

### 3.2 Crear y editar contenido

Cada escritura separa:

- **estructura funcional:** IDs, fechas, situaciones, motivos catalogados, satisfacción, valoraciones, asociaciones, categorías y estados necesarios para aplicar reglas, Timeline y estadísticas;
- **contenido privado:** nombre, género, profesión, WhatsApp, redes y cumpleaños del Crush, además de `place` y `description` de cada experiencia, cifrado por el cliente antes de transferirse.

Cada Crush y experiencia con contenido privado tiene una entrada Vault. Los períodos y los hitos relacionales no tienen payload Vault propio: cualquier explicación privada de un hito pertenece a su experiencia obligatoria. Una edición crea una revisión cifrada nueva y sólo retira la anterior después de verificar la sustituta; esa coexistencia técnica no es un historial disponible para la persona.

### 3.3 Crush, experiencia y relaciones

Crear un Crush es una acción atómica que incluye perfil, período inicial, hito inicial y experiencia inicial completa en `met_at`. La experiencia inicial explica cómo se conocieron y exige fecha, satisfacción, lugar y descripción.

Cada Crush puede tener como máximo una experiencia por día civil; Crushes distintos pueden tener experiencias el mismo día. Toda experiencia pertenece exactamente a un período. Puede ser independiente dentro de él o vincularse a un único hito cuando registra una situación nueva.

`RelationshipSituation` es una única dimensión cerrada, guardada por ID entero:

| ID | Clave interna | Texto visible `es-ES` |
|---:|---|---|
| 1 | `NOT_TALKING` | Todavía no hemos hablado |
| 2 | `ACQUAINTED` | Nos conocemos / hablamos |
| 3 | `FRIENDS` | Solo somos amigos |
| 4 | `MUTUAL_INTEREST` | Hay química, pero aún no hemos quedado |
| 5 | `DATING` | Estamos saliendo |
| 6 | `CASUAL` | Algo casual |
| 7 | `FWB` | Amigos con derecho |
| 8 | `RELATIONSHIP` | Estamos en una relación |
| 9 | `UNDEFINED` | Hay algo, pero prefiero no etiquetarlo |
| 10 | `CLOSED` | Historia cerrada |

El bloque inicial admite situaciones 1–9. Cada cambio posterior crea un hito con una experiencia obligatoria del mismo día; los trayectos válidos son los definidos en Producto y no fabrican estados intermedios. Cerrar exige situación `CLOSED`, experiencia completa y motivo catalogado compatible con la situación anterior.

El único relato libre de un cierre es `description` de la experiencia vinculada; el hito conserva únicamente la situación y el motivo catalogado.

El Crush conserva `match_status_id`: `0 UNKNOWN`, `1 MATCH` o `2 NO_MATCH`. Un hito conservado en situaciones 5–9 establece `MATCH`; también se guardan la primera situación y fecha que lo establecieron. Corregir o eliminar historia recalcula esas proyecciones. Sin evidencia, un período vivo produce `UNKNOWN` y todos los períodos cerrados producen `NO_MATCH`; abrir uno nuevo puede devolverlo a `UNKNOWN`.

Las correcciones y borrados relacionales son `latest-only`. Eliminar una experiencia vinculada exige eliminar conjuntamente su hito conforme a esa elegibilidad. Eliminar sólo un hito posterior puede conservar la experiencia como independiente. Un período posterior sólo se elimina cuando su hito inicial sigue siendo su único hito; la operación elimina también todas sus experiencias y nunca reabre el anterior. El período inicial desaparece únicamente al eliminar el Crush.

### 3.4 Lectura, Timeline y estadísticas

API entrega estructura funcional. El cliente descifra contenido privado y fusiona ambos por identidad. La búsqueda por nombre se realiza localmente con el Vault desbloqueado; el término no se envía a API.

Timeline representa bloques iniciales, experiencias independientes, períodos posteriores e hitos sin duplicar las partes de una acción compuesta. Home conserva únicamente identidades, fechas, tipos y asociaciones; nombre, lugar y descripción se resuelven desde el Vault.

El dashboard de beta es personal y no combina cuentas. `winRate` usa Crushes históricos con `MATCH` sobre los que tienen resultado resuelto; `actualStatus` cuenta períodos vivos en `RELATIONSHIP`; `activeCrushes` y `bySituation` describen el momento actual. Las métricas históricas incluyen Crushes activos y archivados; las actuales sólo activos no archivados.

### 3.5 Fotos

Las fotos del Crush se sanean y permanecen exclusivamente en el almacenamiento local del dispositivo durante la beta. Deben excluirse de backups no deseados y eliminarse al borrar la foto, el Crush o la cuenta.

### 3.6 Borrado, baja y recovery

Los borrados funcionales coordinan estructura, bindings y blobs afectados. El cliente conserva una intención durable hasta confirmar el resultado. El ciphertext retirado se purga físicamente después de siete días; backups y proveedores siguen ciclos propios.

Eliminar un Crush abarca su historia, bindings, blobs, estado local y fotos. La baja de cuenta revoca sesiones y acceso, retira identidad directa, elimina el Vault y artefactos locales y aplica a la estructura restante el tratamiento que apruebe Legal.

Recovery reconstruye y valida el inventario completo en el dispositivo antes de publicarlo. No ejecuta borrados remotos.

## 4. Entidades y ubicación

| Entidad | Estructura visible para API | Contenido privado |
|---|---|---|
| `User` | ID, email cifrado recuperable, índice exacto no reversible, password hash cuando existe, nacimiento, género/país opcionales, plan, estados y aceptaciones | Fuera del Vault; acceso limitado a finalidades de cuenta |
| `FederatedIdentity` | identificador Apple protegido, digest, email/Private Relay, estado y material técnico de revocación | Fuera del Vault; valores sensibles cifrados por API |
| `Crush` | ID, edad/ancla, país, Ducky, rasgos, ratings, `met_at`, match, archivo y ciclo de vida | Vault: nombre, género, profesión, WhatsApp, redes y cumpleaños |
| `Experience` | ID, propietarios, fecha, satisfacción, período, correlación y hito derivado | Vault: lugar y descripción |
| `RelationshipPeriod` | ID, propietarios, inicio/fin, situación vigente, correlación y ciclo de vida | Sin payload Vault |
| `RelationshipMilestone` | ID, período, experiencia obligatoria, secuencia, fecha, situaciones, motivo, correlación y capacidades | Sin payload Vault; explicación en la experiencia |

| Capa | Conserva | Puede leer contenido Vault en claro |
|---|---|---|
| Cliente desbloqueado | estructura, snapshots y proyecciones | Sí durante la sesión autorizada |
| SQLite local | ciphertext, metadata e intenciones durables | No sin VMK |
| Almacén seguro | VMK y material local protegido | Bajo controles del dispositivo |
| API/RDS | cuenta, estructura, bindings, encapsulaciones, metadata y eventos | No |
| S3 Vault | objetos ciphertext | No sin VMK |
| Sandbox de fotos | imágenes locales saneadas | Sólo el dispositivo |
| Apple | identidad federada de cuentas que usan Apple en iOS | Nunca contenido Vault |
| Email/observabilidad | mensajes transaccionales y contexto técnico permitido | Nunca contenido Vault por diseño |

## 5. Preguntas para asesoría

1. ¿Qué finalidades y bases corresponden a la persona usuaria y a las terceras personas descritas?
2. ¿Qué deber de información existe respecto de esas terceras personas y qué excepciones pueden aplicar?
3. ¿Situaciones, match, cierres, hitos, experiencias, satisfacción, ratings o combinaciones de rasgos constituyen categorías especiales o inferencias equivalentes?
4. ¿Qué estructura legible por API es necesaria y proporcional y qué campos deben reducirse, retirarse o protegerse de otra forma?
5. ¿Es necesaria una EIPD antes del lanzamiento?
6. ¿Cómo se califica la estructura seudonimizada conservada tras la baja y qué finalidad, base, plazo, acceso y salvaguardas exige?
7. ¿Qué lenguaje puede usarse sobre E2EE, recovery, borrado, fotos locales y dispositivo único?
8. ¿Cómo se atienden derechos de una tercera persona sin revelar el diario de la persona usuaria?
