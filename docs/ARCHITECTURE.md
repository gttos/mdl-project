# Arquitectura del sistema MDL

Este documento describe la arquitectura transversal. El catálogo funcional, incluidas las decisiones con implementación pendiente, se organiza por dominio en `rules/`, con índice en [`PRODUCT.md`](PRODUCT.md); los requisitos jurídicos en [`legal/`](legal/README.md) y la evidencia de ejecución en los planes activos. Una descripción técnica anterior no redefine una regla del catálogo.

## 1. Componentes

MDL coordina cinco proyectos independientes:

- **iOS:** interfaz, estado de sesión, cifrado/descifrado, base local del Vault, proyecciones pendientes y almacenamiento local de fotos.
- **Android:** cliente nativo Kotlin/Compose en incorporación para una distribución futura, fuera de la beta activa iOS/API; antes de habilitarse debe ofrecer las mismas garantías funcionales, criptográficas y durables mediante componentes idiomáticos de Android.
- **API:** autenticación, autorización, datos estructurales, invariantes transaccionales, inventario remoto del Vault y URLs firmadas.
- **LLM:** generación de insights con el contexto mínimo autorizado.
- **Product:** fuente canónica de comportamiento compartido, arquitectura transversal, Legal y planes maestros.

No es un monorepo: cada proyecto de código tiene su propio repositorio y ciclo de entrega.

## 2. Propiedad de datos

| Dato | Fuente de verdad | Observación |
|---|---|---|
| Cuenta, sesión y verificación | API | Cada cliente nativo conserva sólo las credenciales/tokens necesarios de forma segura. |
| IDs funcionales, fechas, situaciones, períodos, hitos y métricas estructurales | API | La API aplica autorización e invariantes de dominio. |
| Texto íntimo y snapshots durables | Vault cifrado | El cliente cifra antes de transferir; API/S3 ven ciphertext y metadatos mínimos. |
| VMK y material de descifrado | Dispositivo/encapsulación E2EE | Nunca se envía una VMK en claro a la API. |
| Proyección de cambios pendientes | Memoria del cliente, reconstruida desde Vault local | No es una segunda base persistente. |
| Fotos del crush en la beta | Almacenamiento local del cliente | Sin backup remoto vigente. |
| Reglas funcionales | `product/docs/rules/`, con índice en `product/docs/PRODUCT.md` | Los planes referencian IDs y mantienen ejecución/evidencia; los consumidores no las redefinen. |
| Requisitos jurídicos | `product/docs/legal/` | Legal se mantiene modular y separado. |

### 2.1 Entidades principales

| Entidad | Propósito | Nota |
|---|---|---|
| `User` | Cuenta autenticada | Cuenta fuera del Vault E2EE. El contrato objetivo de [Cuenta](rules/ACCOUNT.md) conserva el email cifrado por el servidor, la fecha completa de nacimiento declarada y los IDs opcionales de género y país; implementación pendiente en [Información del usuario](plans/active/user-information/README.md). |
| `FederatedIdentity` | Método de autenticación externo vinculado explícitamente a un `User` | Apple se identifica por `sub` verificado, no por email; los tokens de revocación permanecen cifrados y sujetos a la retención de `legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md`. |
| `Crush`, `Experience`, `RelationshipPeriod`, `RelationshipMilestone` | Datos estructurales del diario | La situación se guarda por ID entero y se define mediante Value Objects cerrados; el contenido íntimo asociado vive en `VaultBlob`, no en columnas propias — ver §2. |
| `Vault` | Contenedor E2EE de una cuenta | estado y versión de cripto. |
| `VaultBlob` | Revisión cifrada de una entidad privada | Varias revisiones temporales pueden coexistir durante la sustitución segura; comparten el binding opaco de esa entidad. |
| `VaultEntityBinding` | Vínculo opaco entre una entidad funcional y sus blobs | La API conoce tipo, ID funcional y token, pero el token se deriva con la VMK sólo en el cliente y no revela contenido. |
| `VaultDevice` | Dispositivo autorizado a operar el Vault | |
| Encapsulación principal y `VaultMnemonicEncapsulation` | Recuperación de la VMK mediante contraseña o frase de 24 palabras | Son las dos encapsulaciones de recuperación de la beta. API e iOS ya retiraron la infraestructura y el cliente de transferencia QR conforme a `RP-VLT-032`. |
| `ActivityEvent` | Evento de timeline/feed | retención documentada en `legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md`. |
| `AuditEvent` | Registro forense inmutable de operaciones sensibles | idem. |
| `SecurityAlert` | Alerta de comportamiento sospechoso | idem. |
| `AccountDeletionFeedback` | Motivo registrado al eliminar cuenta | |

No existe entidad `File`: se eliminó de la base en julio de 2026 por no tener ningún flujo activo que la usara (las fotos del crush son locales, ver §10). Si code o docs viejos la mencionan, está obsoleto.

## 3. Identidad y aislamiento

- Los IDs de Crush, experiencia, período, hito y operación se generan de forma estable y se reutilizan en reintentos.
- La API trata las creaciones repetidas con el mismo ID como idempotentes o como una colisión explícita, nunca como una segunda entidad silenciosa.
- Todo acceso se limita al propietario autenticado; un ID de otra cuenta debe producir rechazo sin revelar datos.
- El almacenamiento local, el almacén seguro de claves, las preferencias y las proyecciones se namespacen por cuenta/Vault.
- Comparaciones de UUID funcionales son canónicas y no dependen de mayúsculas/minúsculas.

## 4. Flujo de lectura

1. El cliente nativo conserva por separado una sesión de cuenta autenticada o una capacidad opaca de registro email conforme a [Cuenta de usuario](rules/ACCOUNT.md); la capacidad nunca se usa como bearer de cuenta.
2. Una capacidad de registro sólo puede continuar, verificar o cancelar su propio intento. No resuelve perfil, namespace, Vault ni vinculación Apple/Google, y el login de una provisional falla como credencial inválida.
3. Tras verificar capacidad y OTP del mismo intento, el cliente sustituye la capacidad por la primera sesión de cuenta, obtiene el perfil estructural y resuelve el namespace y el estado del Vault.
4. Descarga inventario/ciphertext cuando corresponde y verifica versiones.
5. Descifra localmente las entradas autorizadas.
6. Fusiona datos confirmados de API/Vault con proyecciones pendientes del mismo scope funcional.
7. Presenta una vista coherente sin duplicar entidades por entry interna.

Una recarga confirmada y una reconstrucción de proyección se secuencian; no se permite una desaparición temporal causada por dos refresh paralelos.

## 5. Flujo de escritura durable

1. La UI valida el formulario completo y las reglas que puede conocer localmente.
2. El cliente genera o conserva IDs funcionales y congela los días civiles/valores necesarios para retry.
3. Cifra el snapshot y persiste una `VaultWriteOperation` antes de emitir la API.
4. Publica una proyección local y cierra el formulario con confirmación de guardado local.
5. El coordinador clasifica conectividad, retry y conflictos por crush y dominio.
6. El executor envía la operación idempotente a la API.
7. Tras aceptación, persiste/sincroniza las entries cifradas y verifica el blob remoto.
8. Sólo después de la frontera de confirmación elimina la operación y muestra estado sincronizado.

Los estados terminales o que requieren intervención no se reactivan automáticamente. El retry automático usa una única política y un único scheduler por proceso.

Una superficie de estado captura el `operationId` al tocar el badge y conserva esa identidad mientras se presenta. Si la operación desaparece antes de abrir, no se presenta una hoja vacía; si sigue disponible, retry, descarte o corrección vuelven a leer exactamente esa operación. Corregir sólo se ofrece cuando existe contexto suficiente para reconstruir el formulario sin inferencias y falla cerrado si la operación dejó de corresponder a la entidad elegida.

## 6. Dominios de conflicto

Cada escritura estructural reserva un único crush y uno de estos dominios:

- `root`: alta todavía no confirmada del crush.
- `profile`: edición del perfil del crush.
- `timeline`: creación o edición que altera experiencias/relaciones cronológicas.

El conflicto se evalúa dentro del mismo crush según `RP-DUR-006..009`. Los snapshots incompatibles fallan cerrados; no se conserva una política runtime alternativa para operaciones anteriores al contrato de lanzamiento.

## 7. Fechas y cronología

- Las fechas funcionales son días civiles `yyyy-MM-dd`, no instantes UTC.
- El cliente congela la representación civil antes de cifrar una operación.
- La API valida unicidad y orden cronológico dentro de una transacción.
- Las decisiones que consultan un slot compartido se realizan con el bloqueo de persistencia adecuado para evitar dos ganadores concurrentes.
- El orden del Timeline debe ser determinista aun con eventos en el mismo día.

## 8. Vault E2EE

- El cliente genera la Vault Master Key (VMK) y cifra las entries con criptografía autenticada.
- La VMK se encapsula con los mecanismos de recuperación aprobados.
- El backend almacena encapsulación, inventario y blobs cifrados, pero no puede leer contenido.
- Una instalación sin encapsulación local consulta el Vault remoto. Si existe, entra en recuperación y conserva ese estado entre background y relanzamientos.
- Cada entidad funcional privada tiene un envelope cifrado completo identificado por `(entryType, entityId, revision)`. Una revisión nueva se verifica antes de retirar la anterior; esa coexistencia es transitoria y no constituye historial de usuario.
- El cliente deriva `lookupToken = HMAC-SHA-256(VMK, "mdl-vault-lookup:v1:<entryType>:<entityId>")`, codificado Base64URL sin padding. El mismo token acompaña todas las revisiones de una entidad y nunca se registra en logs.
- La API crea un `VaultEntityBinding` inmutable dentro de la misma transacción que materializa un Crush o una experiencia. Registrar un blob de texto exige un binding activo coincidente y falla cerrado ante ausencia, conflicto o cruce de propietario. Los períodos e hitos relacionales no crean bindings ni blobs propios; su explicación privada pertenece al binding de la experiencia vinculada.
- Borrar una experiencia, hito o período elegible resuelve los bindings afectados dentro de la transacción funcional y aplica soft delete a todos sus blobs activos. El retry es idempotente; el cliente no lista, descarga, verifica ni descifra el inventario para descubrir qué borrar.
- El inventario global permanece disponible para sync, recovery y migración controlada. Recovery pagina y valida todo el conjunto, selecciona una revisión inequívoca por entidad, publica atómicamente y nunca ejecuta borrados remotos.
- El baseline validado es cuenta nueva/Vault nuevo. No existe una migración de usuario pre-beta que deba conservarse. La retirada del código residual de migración, el borrado completo de Crush, las fotos, el reset integral del Vault y la baja de cuenta permanecen dentro de la transición activa documentada en [`plans/active/vault-e2ee/README.md`](plans/active/vault-e2ee/README.md); por eso la iniciativa no está cerrada.
- Android debe demostrar este mismo contrato e interoperabilidad antes de su propia distribución y de admitir datos reales; su estado vive en [`plans/active/android-parity/README.md`](plans/active/android-parity/README.md). Ese trabajo no es un gate del despliegue ni del cierre de la beta activa iOS/API.

## 9. API y contratos

- Las peticiones de los clientes usan el contrato HTTP documentado por API; los campos de entrada actuales se expresan en `snake_case`.
- Las respuestas públicas se normalizan a `camelCase` por la capa de presentación de API.
- Un `4xx` representa rechazo confirmado; no se reintenta automáticamente salvo una condición temporal explícita como rate limiting.
- Un `429` publica la misma espera entera en el header HTTP `Retry-After` y en `error.details.retryAfter`; sólo se reflejan headers de rate limit permitidos y saneados.
- La contención temporal de una escritura serializada por Crush se publica como `503 temporarily_unavailable`, nunca como validación: no incluye `fields` ni la clave técnica, y anuncia cinco segundos tanto en `Retry-After` como en `error.details.retryAfter` para que el cliente conserve el reintento automático.
- Un fallo de transporte sin respuesta es ambiguo y se resuelve mediante idempotencia y consulta/retry seguro.
- Los errores incluyen código estable, campos afectados y claves saneables; el cliente registra diagnóstico útil sólo en builds de desarrollo sin exponer secretos ni contenido íntimo.
- Las operaciones compuestas se ejecutan en transacción y no dejan mutaciones parciales tras un rechazo.

## 10. Seguridad y observabilidad

- API aplica validación, autorización por propietario, rate limiting y transacciones en las fronteras correspondientes.
- Cualquier cambio de retención, borrado o tratamiento de datos se valida contra [`legal/`](legal/README.md).

### 10.1 Modelo de amenazas (resumen transversal)

La columna Estado refleja la evidencia vigente de API/iOS, que son los componentes de la beta activa. Android queda fuera de esa beta y permanece pendiente de satisfacer los gates equivalentes del plan `android-parity` antes de su propia distribución, sin bloquear el release iOS/API.

| Amenaza | Mitigación | Estado |
|---|---|---|
| Exposición de contenido íntimo (nombre/contacto/redes/cumpleaños del Crush y lugar/descripción de experiencias) | Vault E2EE: cifrado en el dispositivo, el servidor nunca ve el plaintext | Implementado |
| Exposición del email de cuenta | Cifrado de campo en servidor, búsqueda protegida, TLS, acceso restringido y exclusión de logs | Cifrado de campo y búsqueda pendientes en `user-information`; controles de infraestructura pendientes de evidencia operativa |
| Ampliación accidental del perfil de cuenta | Inventario cerrado de Cuenta; fecha de nacimiento declarada asociada a la fecha del alta | Alineación de persistencia y contratos pendiente en `user-information`; los campos autorizados requieren controles de acceso y observabilidad |
| Acceso no autorizado | Bearer token + autorización por propietario en cada ruta | Implementado |
| Enumeración de cuentas en recuperación de contraseña | Respuesta pública genérica al solicitar, verificar o usar un OTP; detalle sólo en observabilidad saneada | BR-3.16 implementada y cubierta por pruebas HTTP; el bloqueo conserva su efecto interno | Implementado |
| Fuerza bruta | Rate limiting por ruta y usuario/IP | Limiters autenticados y de blobs restaurados desde configuración; la validación de volumen real de sync de Vault queda pendiente | Implementado con validación operativa pendiente |
| Pérdida de recuperación del Vault | Contraseña y frase de 24 palabras encapsulan la misma VMK; QR y `recovery-key` no forman parte de la beta | Los dos recorridos autorizados están implementados y cubiertos por pruebas automatizadas; su validación física sigue pendiente. El cliente no contiene transferencia QR y API no publica ni persiste `recovery-key`. |
| Blob huérfano al borrar experiencia, hito o período | Binding opaco y soft delete de todas las revisiones dentro de la transacción funcional | El contrato objetivo está definido; debe revalidarse con el modelo relacional final. |
| Datos huérfanos tras borrar crush o cuenta | Deben coordinar blobs, entradas locales, encapsulaciones, dispositivos y fotos locales | Pendiente; fuera del cierre de borrado individual. |

El detalle técnico de mitigación (headers, límites exactos, EXIF, backups del sistema operativo) vive en el `ARCHITECTURE.md` de cada repo, no acá.

## 11. Integración del servicio LLM

- El microservicio existe como prototipo independiente y no está conectado a la beta desde API ni los clientes nativos.
- Su endpoint actual acepta `notes` y resúmenes de `events` en texto libre y los incorpora al prompt. Esa capacidad **no está autorizada para datos reales** mientras Product y Legal no definan propósito, minimización, retención, proveedor y consentimiento aplicables.
- API/iOS/Android no deben activar ese endpoint ni enviar plaintext del Vault por el mero hecho de que el prototipo exista.
- Una integración futura debe partir de contexto estructural mínimo o de contenido autorizado de forma explícita, sin acceso directo del servicio al Vault, a la base local de ningún cliente ni a credenciales.
- Las salidas serían auxiliares y nunca modificarían automáticamente el historial del usuario.

## 12. Documentación y evolución

- Product define **qué** debe ocurrir y el contrato compartido.
- API/iOS/Android/LLM documentan **cómo** lo implementan localmente.
- La beta 1.0 aplica `RP-BASE-009`: no mantiene rutas de compatibilidad para usuarios, contratos o estados pre-beta. Los repositorios convergen al contrato vigente y fallan cerrados ante un dato obligatorio no resuelto; las transiciones reales autorizadas viven únicamente en un plan activo con salida definida.
- Una regla se define en el documento de su dominio en `rules/` y se referencia por ID desde el índice `PRODUCT.md`; los planes maestros/locales mantienen escenarios, ejecución y evidencia sin otro catálogo funcional.
- Sólo permanecen planes activos. Al terminar, las reglas estables se integran en su documento de `rules/` y el detalle técnico aquí o en el documento técnico correspondiente; el plan se elimina.
- Git conserva la historia; no se crean decision logs, auditorías, worklogs, checkpoints ni carpetas `completed`/`archived` como memoria paralela.

## 13. Entorno de integración local

La integración cotidiana de API e iOS usa un entorno aislado y persistente que nunca sustituye staging, producción ni los gates de distribución:

- `mdl-local-integration` contiene API, MySQL/InnoDB, MinIO con proxy SigV4 y Mailpit bajo nombres, red, volúmenes y puertos propios. Una parada ordinaria conserva sus datos y ningún comando del entorno opera sobre los stacks de desarrollo, seguridad, concurrencia o producción.
- `MDL Local` es una variante iOS exclusiva de simulador. Su destino loopback se decide en compilación, usa un bundle ID y almacenamiento propios y se identifica visualmente. `MDL Production`, Debug normal y Release conservan el destino de producción.
- El bootstrap del backend sólo crea catálogos públicos/de referencia y contenido Legal expresamente sintético, bilingüe y sin valor jurídico. Las cuentas, Vaults y entidades privadas se crean mediante los flujos reales del cliente.
- Las claves locales permanecen fuera de Git y deben conservarse mientras se conserven los datos cifrados de ese stack. Los comandos comprueban su presencia sin leerlas ni imprimirlas.
- El entorno permite recorridos reales vacíos y poblados, aislamiento entre cuentas, Vault/MinIO, correo local, relanzamiento y comportamiento offline. Esa evidencia no certifica accesibilidad, dispositivo físico, revisión Legal, despliegue ni release.

La operación diaria vive en `api/docs/OPERATIONS.md` y la selección/validación del cliente en `ios/docs/TESTING.md`.
