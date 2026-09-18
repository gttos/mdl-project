# 1. Contexto del producto y sus entidades

> Descripción del modelo funcional de la beta de lanzamiento a 14 de septiembre de 2026. Los campos y ubicaciones exactos están en el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md).

## 1. Qué es MDL

MDL es una aplicación iOS de diario privado para personas adultas. Permite registrar personas de interés, experiencias, evolución de relaciones, valoraciones subjetivas, fotografías locales y estadísticas personales.

No es una red social. La persona descrita no crea necesariamente una cuenta, no recibe un perfil público, no participa en el registro y no conoce necesariamente su existencia. Todo el contenido lo introduce y consulta el titular de la cuenta.

## 2. Personas afectadas

| Persona | Relación con MDL | Datos principales |
|---|---|---|
| Persona usuaria | Crea y usa una cuenta | identidad, contacto, autenticación, actividad, diario, valoraciones y estadísticas |
| Crush o tercera persona | Es descrita por el usuario | nombre/contacto, rasgos, edad, país/género, fechas, fases, etiquetas, cierres, experiencias, relaciones y fotos |
| Personal autorizado | Opera seguridad, infraestructura o soporte | acceso limitado a datos estructurales, metadata y logs según rol; no dispone de una función de descifrado del Vault |
| Proveedores | Infraestructura o funciones auxiliares | ciphertext, cuenta, email o telemetría según el servicio y la configuración real |

“Crush” es el nombre de producto de una ficha privada. No implica consentimiento, conocimiento ni relación contractual de esa tercera persona con MDL.

## 3. Recorrido principal

### 3.1 Cuenta y Vault

La persona crea una cuenta mediante email/contraseña o Sign in with Apple y acepta las versiones vigentes de términos y privacidad. El alta por email nace provisional y no accede al diario ni configura el Vault hasta verificar un OTP de seis dígitos; vence a los siete días aunque se reenvíen códigos. El alta Apple con email acreditado nace verificada y puede no tener contraseña MDL. Un `sub` Apple nuevo no se fusiona automáticamente con una cuenta por coincidencia de email: el vínculo exige acreditar la cuenta MDL con email y contraseña. La misma vía explícita permite vincular cuando Private Relay entrega un email Apple distinto; el vínculo conserva el `User`, email primario, estado de verificación y Vault existentes, sin crear otro namespace.

Con la identidad verificada, iOS configura el Vault: genera una VMK aleatoria por cuenta, la encapsula para recovery y conserva el material local dentro del namespace de esa cuenta.

El servidor almacena cuenta, datos estructurales, encapsulaciones, bindings, metadata y ciphertext remoto, pero no recibe la VMK en claro ni dispone de una operación para descifrar el contenido del Vault.

En un dispositivo sin estado local, la app consulta primero API. Si existe un Vault remoto, entra en recovery y no crea otro que lo sobrescriba. La beta contempla un único dispositivo activo por Vault; recovery sustituye el dispositivo, no sincroniza dos copias activas de forma continua.

### 3.2 Crear y editar contenido

Cada alta o edición separa dos clases de información:

- **payload funcional:** IDs, fechas, fases, etiquetas, motivos catalogados, satisfacción/ratings, asociaciones, categorías y otros atributos que API necesita para aplicar reglas y estadísticas;
- **snapshot privado:** nombre/contacto/redes/cumpleaños del crush, lugar/descripción de Experience, nota de cierre y explicaciones relacionales libres, cifrado en iOS antes de transferirse.

Cada crush, Experience o período relacional tiene identidad funcional propia y una entrada Vault independiente. Una edición crea una revisión cifrada nueva; la anterior se retira sólo después de verificar la sustituta. Esa coexistencia técnica no es un historial de versiones disponible para el usuario.

Un token opaco derivado con la VMK enlaza la entidad funcional con todas sus revisiones. API puede compararlo para autorizar y borrar ciphertext, pero no derivar el contenido o la VMK.

### 3.3 Relaciones

Cada crush nace con un período relacional inicial en `met_at`. Los períodos pueden comenzar en cualquiera de las fases Interesado, Conocido, Interés mutuo, Involucrado o Cerrado y, mientras estén vivos, sólo conservar la fase o avanzar; pueden saltar hacia delante sin fabricar fases intermedias. Durante Involucrado puede elegirse Citas, Casual, FWB, Relación u Otro. Cerrar exige `REJECTED`, `MISSED`, `ENDED` u `OTHER`; cualquier explicación libre vive en Vault.

API conserva períodos, fechas, fases, etiquetas, motivos catalogados, historiales append-only, hitos y asociaciones con Experiences. `everMatched` se vuelve verdadero sólo si algún período alcanza Involucrado y no retrocede. Una Experience puede aplicar atómicamente un cambio relacional del mismo día; eliminarla conserva el hito, y eliminar un período no inicial nunca reabre el anterior.

Aunque las notas estén cifradas, esta estructura puede revelar o permitir inferir aspectos de la vida afectiva o sexual de la persona usuaria y de la tercera persona. No existe en la beta un runtime alternativo basado en el enum relacional anterior.

### 3.4 Lectura, Timeline y búsqueda

API entrega la estructura funcional. iOS descifra el contenido privado localmente y fusiona ambos por ID. La búsqueda privada se realiza en memoria sobre contenido ya desbloqueado; no existe un índice persistente de plaintext del Vault.

Timeline y estadísticas se construyen a partir de la estructura. Por ello fechas, fases, etiquetas, cierres, asociaciones, rasgos y valoraciones siguen siendo visibles para backend y pueden producir inferencias aunque el texto libre esté E2EE.

### 3.5 Fotos

Las fotos de crush se sanean en iOS y permanecen en el sandbox local. No se suben a S3 ni se incorporan al Vault remoto durante la beta. El modelo de lanzamiento exige que queden excluidas de backups no deseados del sistema y se eliminen al borrar el crush o la cuenta.

### 3.6 Borrado y recovery

Eliminar una Experience o período elegible emite una petición funcional que coordina la fila estructural, binding y blobs. iOS mantiene una intención durable hasta confirmar el resultado. La purga física se difiere hasta siete días para permitir un procesamiento técnico controlado. El período inicial no se elimina por la acción ordinaria; sólo el último período cerrado y no inicial admite borrado correctivo, y nunca reabre el anterior.

Eliminar un crush coordina también sus descendientes, entradas locales y fotos. La baja de cuenta revoca sesiones, elimina el Vault y artefactos locales y aplica el tratamiento aprobado a la estructura restante.

Recovery reconstruye el inventario activo en el dispositivo, lo descifra y valida antes de publicarlo. No borra objetos remotos.

## 4. Entidades prioritarias

| Entidad | Estructura visible para API | Contenido privado Vault |
|---|---|---|
| User | ID, email, password hash cuando existe, verificación, fecha de nacimiento, perfil, plan, flags y aceptaciones | No aplica; email/teléfono no pertenecen al Vault |
| FederatedIdentity | identificador Apple protegido, digest, email/Private Relay, estado y material técnico de revocación | No aplica; pertenece a autenticación y sus valores sensibles se cifran en API |
| Crush | ID, edad/ancla, país, género, categoría, rasgos, ratings, `met_at`, archivo y derivados | nombre, profesión, contacto/redes, cumpleaños y detalles privados |
| Experience | ID, crush, fecha, satisfacción, correlación y vínculos | lugar y descripción |
| Relationship / período | ID, crush, fase, etiqueta, motivo, inicio/fin, historiales, `everMatched`, correlación y Experience vinculada | nota de cierre y explicación libre de “Otro” |
| PeriodChange / hito | ID, período, crush, fecha efectiva, fase/etiqueta y vínculo opcional con Experience | texto privado dentro del snapshot correspondiente |

Entidades técnicas adicionales: Vault, VaultBlob, VaultEntityBinding, encapsulaciones principal/mnemónica, dispositivos, recovery keys, continuaciones y notificaciones Apple, eventos de actividad/auditoría/autenticación/seguridad, aceptaciones legales y feedback de baja.

## 5. Dónde vive la información

| Capa | Conserva | Puede leer plaintext privado |
|---|---|---|
| iOS desbloqueado | estructura, snapshots, proyecciones y fotos | Sí durante la sesión autorizada |
| SQLite local | ciphertext, metadata e intenciones durables | No sin VMK |
| Keychain | VMK y material local protegido | Bajo controles del dispositivo |
| API/RDS | cuenta, estructura, bindings, encapsulaciones, metadata y eventos | No el contenido Vault; sí datos funcionales y de cuenta |
| S3 Vault | objetos ciphertext | No sin VMK |
| Sandbox de fotos | imágenes locales saneadas | El dispositivo |
| Apple | identidad federada, email/Private Relay y material técnico de validación/revocación | Apple y API según el flujo; nunca contenido Vault |
| Email/observabilidad | mensajes transaccionales y contexto técnico | Según proveedor/configuración; nunca contenido Vault por diseño |

## 6. Datos derivados y estadísticas

MDL deriva edad, signo zodiacal, rating global, fase/etiqueta vigente, `everMatched`, duraciones y estadísticas. La única superficie estadística de beta es el dashboard personal: `winRate` usa Crushes visibles y no borrados que alcanzaron Involucrado; `actualStatus` cuenta períodos vivos Involucrados con etiqueta Relación; `activeCrushes` cuenta períodos vigentes vivos por fase. No compara usuarios entre sí.

Tras una baja sólo se conserva la estructura autorizada para estadísticas agregadas sin PII. Mantener filas individuales desidentificadas no garantiza anonimato jurídico y puede ser seudonimización; la asesoría debe fijar su calificación, finalidad, base, plazo, acceso y salvaguardas.

## 7. Límites que el texto público debe explicar

- E2EE protege contenido privado, no toda la estructura ni metadata.
- Un dispositivo desbloqueado o un secreto de recovery expuesto puede revelar el Vault.
- Perder todos los mecanismos de recovery puede volver irrecuperable el contenido.
- La beta limita el Vault a un dispositivo activo.
- Las fotos son locales y no tienen backup remoto de MDL.
- El backend no puede descifrar el Vault, lo que condiciona soporte, acceso y portabilidad.
- “Anonimizado” no debe usarse públicamente hasta validar el riesgo de reidentificación de lo conservado.

## 8. Preguntas para asesoría

1. ¿Qué finalidades y bases corresponden al usuario y a las terceras personas descritas?
2. ¿Qué deber de información existe respecto de esas terceras personas y qué excepciones podrían aplicar?
3. ¿Las fases, etiquetas, cierres, historiales relacionales, experiencias, género, ratings y combinaciones de rasgos deben tratarse como categorías especiales o inferencias equivalentes en alguno de los países de lanzamiento?
4. ¿Qué campos estructurales son proporcionales y cuáles deben pasar a Vault, reducirse o eliminarse?
5. ¿Es necesaria una EIPD antes del lanzamiento?
6. ¿Cómo debe calificarse la estructura desidentificada conservada tras la baja y qué salvaguardas exige su uso estadístico?
7. ¿Qué lenguaje puede usarse sobre E2EE, recovery, borrado, fotos locales y dispositivo único?
8. ¿Qué procedimiento permite atender a una tercera persona sin revelar el diario del usuario?
