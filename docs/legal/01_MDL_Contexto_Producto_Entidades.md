# 1. Contexto del producto y sus entidades

> Descripción del modelo objetivo de la beta de lanzamiento a 2 de septiembre de 2026. Los campos y ubicaciones exactos están en el [inventario maestro](02_MDL_Inventario_Maestro_Datos.md).

## 1. Qué es MDL

MDL es una aplicación iOS de diario privado para personas adultas. Permite registrar personas de interés, experiencias, evolución de relaciones, valoraciones subjetivas, fotografías locales y estadísticas personales.

No es una red social. La persona descrita no crea necesariamente una cuenta, no recibe un perfil público, no participa en el registro y no conoce necesariamente su existencia. Todo el contenido lo introduce y consulta el titular de la cuenta.

## 2. Personas afectadas

| Persona | Relación con MDL | Datos principales |
|---|---|---|
| Persona usuaria | Crea y usa una cuenta | identidad, contacto, autenticación, actividad, diario, valoraciones y estadísticas |
| Crush o tercera persona | Es descrita por el usuario | nombre/contacto, rasgos, edad, país/género, fechas, estados, experiencias, relaciones y fotos |
| Personal autorizado | Opera seguridad, infraestructura o soporte | acceso limitado a datos estructurales, metadata y logs según rol; no dispone de una función de descifrado del Vault |
| Proveedores | Infraestructura o funciones auxiliares | ciphertext, cuenta, email o telemetría según el servicio y la configuración real |

“Crush” es el nombre de producto de una ficha privada. No implica consentimiento, conocimiento ni relación contractual de esa tercera persona con MDL.

## 3. Recorrido principal

### 3.1 Cuenta y Vault

La persona crea una cuenta y acepta las versiones vigentes de términos y privacidad. Después configura un Vault. iOS genera una VMK aleatoria por cuenta, la encapsula para recovery y conserva el material local dentro del namespace de esa cuenta.

El servidor almacena cuenta, datos estructurales, encapsulaciones, bindings, metadata y ciphertext remoto, pero no recibe la VMK en claro ni dispone de una operación para descifrar el contenido del Vault.

En un dispositivo sin estado local, la app consulta primero API. Si existe un Vault remoto, entra en recovery y no crea otro que lo sobrescriba. La beta contempla un único dispositivo activo por Vault; recovery sustituye el dispositivo, no sincroniza dos copias activas de forma continua.

### 3.2 Crear y editar contenido

Cada alta o edición separa dos clases de información:

- **payload funcional:** IDs, fechas, estados, satisfacción/ratings, asociaciones, categorías y otros atributos que API necesita para aplicar reglas y estadísticas;
- **snapshot privado:** nombre/contacto/redes/cumpleaños del crush, lugar/descripción de experiencia y nota de cierre, cifrado en iOS antes de transferirse.

Cada crush, experiencia o relación tiene identidad funcional propia y una entrada Vault independiente. Una edición crea una revisión cifrada nueva; la anterior se retira sólo después de verificar la sustituta. Esa coexistencia técnica no es un historial de versiones disponible para el usuario.

Un token opaco derivado con la VMK enlaza la entidad funcional con todas sus revisiones. API puede compararlo para autorizar y borrar ciphertext, pero no derivar el contenido o la VMK.

### 3.3 Relaciones

El modelo vigente crea una relación inicial obligatoria junto con el crush y registra cambios posteriores en orden cronológico. API conserva estado, fechas, match status, asociaciones y vínculo con experiencias; la nota libre de cierre vive en Vault.

Los estados vigentes incluyen categorías como prospecto, one-night stand, FWB, relación, rechazo o terminación. Aunque la nota esté cifrada, la estructura visible puede revelar o permitir inferir aspectos de la vida afectiva o sexual de la persona usuaria y de la tercera persona.

Existe un rediseño conceptual posterior con fase, etiqueta de vínculo, historial de etiquetas y motivo de cierre. No está activo. Si se implementa, aumentará la precisión temporal e inferencial y exigirá revisar inventario, minimización, EIPD, transparencia, retención, derechos y estadísticas antes del despliegue.

### 3.4 Lectura, Timeline y búsqueda

API entrega la estructura funcional. iOS descifra el contenido privado localmente y fusiona ambos por ID. La búsqueda privada se realiza en memoria sobre contenido ya desbloqueado; no existe un índice persistente de plaintext del Vault.

Timeline y estadísticas se construyen a partir de la estructura. Por ello fechas, estados, asociaciones, rasgos y valoraciones siguen siendo visibles para backend y pueden producir inferencias aunque el texto libre esté E2EE.

### 3.5 Fotos

Las fotos de crush se sanean en iOS y permanecen en el sandbox local. No se suben a S3 ni se incorporan al Vault remoto durante la beta. El modelo de lanzamiento exige que queden excluidas de backups no deseados del sistema y se eliminen al borrar el crush o la cuenta.

### 3.6 Borrado y recovery

Eliminar una experiencia o relación elegible emite una petición funcional que coordina la fila estructural, binding y blobs. iOS mantiene una intención durable hasta confirmar el resultado. La purga física se difiere hasta siete días para permitir un procesamiento técnico controlado.

Eliminar un crush coordina también sus descendientes, entradas locales y fotos. La baja de cuenta revoca sesiones, elimina el Vault y artefactos locales y aplica el tratamiento aprobado a la estructura restante.

Recovery reconstruye el inventario activo en el dispositivo, lo descifra y valida antes de publicarlo. No borra objetos remotos.

## 4. Entidades prioritarias

| Entidad | Estructura visible para API | Contenido privado Vault |
|---|---|---|
| User | ID, email, password hash, fecha de nacimiento, perfil, plan, flags y aceptaciones | No aplica; email/teléfono no pertenecen al Vault |
| Crush | ID, edad/ancla, país, género, categoría, rasgos, ratings, `met_at`, archivo y derivados | nombre, profesión, contacto/redes, cumpleaños y detalles privados |
| Experience | ID, crush, fecha, satisfacción, correlación y vínculos | lugar y descripción |
| Relationship | ID, crush, estado, fechas, match status, correlación y experiencia vinculada | nota de cierre |

Entidades técnicas adicionales: Vault, VaultBlob, VaultEntityBinding, encapsulaciones principal/mnemónica, dispositivos, recovery keys, eventos de actividad/auditoría/autenticación/seguridad, aceptaciones legales y feedback de baja.

## 5. Dónde vive la información

| Capa | Conserva | Puede leer plaintext privado |
|---|---|---|
| iOS desbloqueado | estructura, snapshots, proyecciones y fotos | Sí durante la sesión autorizada |
| SQLite local | ciphertext, metadata e intenciones durables | No sin VMK |
| Keychain | VMK y material local protegido | Bajo controles del dispositivo |
| API/RDS | cuenta, estructura, bindings, encapsulaciones, metadata y eventos | No el contenido Vault; sí datos funcionales y de cuenta |
| S3 Vault | objetos ciphertext | No sin VMK |
| Sandbox de fotos | imágenes locales saneadas | El dispositivo |
| Email/observabilidad | mensajes transaccionales y contexto técnico | Según proveedor/configuración; nunca contenido Vault por diseño |

## 6. Datos derivados y estadísticas

MDL deriva edad, signo zodiacal, rating global, estado actual, match status, duraciones y estadísticas. La beta muestra estadísticas de la propia cuenta y no compara usuarios entre sí.

Una baja puede conservar datos sólo conforme a la decisión aprobada. Mantener filas individuales desidentificadas no garantiza anonimato jurídico; puede ser seudonimización. La asesoría debe decidir entre agregación irreversible con eliminación de fuentes o retención seudonimizada justificada.

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
3. ¿Los estados relacionales, experiencias, género, ratings y combinaciones de rasgos deben tratarse como categorías especiales o inferencias equivalentes en alguno de los países de lanzamiento?
4. ¿Qué campos estructurales son proporcionales y cuáles deben pasar a Vault, reducirse o eliminarse?
5. ¿Es necesaria una EIPD antes del lanzamiento?
6. ¿Cómo debe calificarse la estructura conservada tras la baja y cuál de las dos estrategias de estadísticas posteriores se aprueba?
7. ¿Qué lenguaje puede usarse sobre E2EE, recovery, borrado, fotos locales y dispositivo único?
8. ¿Qué procedimiento permite atender a una tercera persona sin revelar el diario del usuario?
