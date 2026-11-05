# Reglas de producto — MDL

<a id="revision"></a><a id="18-decisiones-pendientes"></a><a id="decisiones-pendientes"></a>

Catálogo del funcionamiento definido de MyDuckingLife. Cada identificador tiene una única definición.

<a id="documentos"></a>
## Dominios

| Documento | Contenido |
|---|---|
| <a id="3-cuenta-autenticación-y-sesión"></a><a id="32-recuperación-de-la-contraseña-de-acceso"></a><a id="36-sign-in-with-apple"></a>[Cuenta de usuario](rules/ACCOUNT.md) | Alta y acceso, verificación, Apple, perfil de la cuenta de usuario, ajustes y recuperación de acceso. |
| <a id="4-alta-y-perfil-de-un-crush"></a><a id="41-datos-obligatorios"></a><a id="42-resultado-de-match"></a><a id="43-edición-del-perfil"></a><a id="44-archivar-y-eliminar"></a>[Crush y perfil del Crush](rules/CRUSH.md) | Alta, datos y edición del perfil del Crush, catálogo compartido de adjetivos y estilo social, resultado de match, edad, cumpleaños, zodíaco, valoraciones, fotos, consulta, mapa, archivo y eliminación. |
| <a id="6-experiencias"></a><a id="62-edición"></a><a id="63-borrado"></a>[Experiencias](rules/EXPERIENCES.md) | Experiencia inicial y posteriores: datos, niveles de detalle, respuestas estructuradas, reflexión cifrada, fechas y operaciones. |
| <a id="7-relaciones"></a><a id="71-cronología"></a><a id="73-situaciones-y-trayectos-permitidos"></a><a id="74-cierre"></a><a id="75-registrar-corregir-y-eliminar"></a>[Relaciones](rules/RELATIONSHIPS.md) | Situaciones, períodos, hitos, evidencia relacional de match, trayectos, cierre y acciones latest-only. |
| <a id="81-contenido-e-identidad"></a><a id="83-acciones"></a><a id="84-actividad-reciente-de-home"></a>[Timeline y Home](rules/TIMELINE.md) | Presentación de la historia, orden, agrupación, acciones disponibles y actividad reciente de Home. |
| <a id="10-vault-privacidad-y-fotos"></a><a id="103-borrado-dirigido"></a>[Vault](rules/VAULT.md) | Contenido privado, creación, bloqueo, recuperación, sustitución de dispositivo y retirada de contenido. |
| <a id="9-escrituras-durables-y-sincronización-visible"></a>[Guardado y sincronización](rules/SYNC.md) | Qué significa guardar, cambios pendientes, conflictos, reintentos, corrección por apartados modificados y descarte. |
| <a id="12-estadísticas"></a><a id="121-estadísticas"></a><a id="123-alcance-entre-cuentas"></a>[Estadísticas](rules/STATISTICS.md) | Dashboard personal, cualidades por Crush, respuestas de experiencias, cálculos, ausencia de datos y alcance entre cuentas. |
| [Uso de la aplicación](rules/UX.md) | Tutoriales informativos, onboarding, demo, formularios, mensajes, errores, accesibilidad, carga, vacíos y desconexión. |
| [Analítica de uso en iOS](rules/ANALYTICS.md) | Medición agregada de pantallas y recorridos iOS, activación, opt-out, minimización, semántica y calidad. |
| <a id="15-baja-seudonimización-e-inactividad"></a><a id="16-procesos-automáticos-y-plazos"></a>[Baja, inactividad y ciclo de vida de los datos](rules/DATA_LIFECYCLE.md) | Baja íntegra voluntaria o por inactividad, destrucción de copias recuperables, conservación y tareas automáticas. |
| [Privacidad, seguridad y derechos](rules/PRIVACY.md) | Aislamiento entre cuentas, protección de información, leads de la web pública, diagnóstico, soporte y ejercicio de derechos. |

<a id="1-propósito-y-límites"></a>
<a id="principios"></a>
## Principios generales y alcance

- **RP-BASE-003.** Cada cuenta dispone de un diario privado de uso individual, separado de las demás cuentas. No hay perfiles públicos, interacción social entre cuentas ni administración funcional del diario ajeno.
- **RP-BASE-004.** La beta es gratuita, sin pagos ni suscripciones.
- **RP-BASE-006.** La primera versión pública aplica el mismo contrato funcional a todas las cuentas, datos y plataformas habilitadas.
- **RP-BASE-007.** Las funciones activas son las definidas en este catálogo para la distribución correspondiente.
- **RP-BASE-008.** Los clientes aplican las mismas reglas y garantías; las excepciones de plataforma requieren decisión explícita. La paridad de Android no se presume a partir de iOS.
- **RP-BASE-009.** Las cuentas, datos y clientes de la beta utilizan el mismo contrato vigente. Un dato obligatorio ausente o desconocido produce el error y bloqueo definidos en RP-UX-008.

El requisito de edad está en [Cuenta](rules/ACCOUNT.md#requisitos); el aislamiento entre cuentas y la privacidad, en [Privacidad](rules/PRIVACY.md#propiedad).

<a id="2-vocabulario-funcional"></a>
<a id="vocabulario"></a>
## Vocabulario y fechas compartidas

| Concepto | Significado |
|---|---|
| Cuenta | Persona usuaria autenticada, propietaria del diario. |
| Crush | Ficha privada de una persona descrita por la cuenta. |
| Experiencia | Registro de lo ocurrido con un Crush en un día civil. Pertenece siempre a un período relacional y puede existir dentro de él sin crear un hito nuevo. |
| Experiencia independiente | Experiencia que pertenece a un período relacional pero no está vinculada a ningún hito porque no registró una situación nueva. «Independiente» nunca significa que exista fuera de un período; si ese período se cierra después, la experiencia continúa dentro de él como parte de su historia. |
| Situación relacional | Estado exacto elegido para la historia. Cada situación conserva una identidad y un significado compartidos por sus traducciones conforme a RP-SIT-004. |
| Período relacional | Tramo de la historia relacional que comienza con un hito inicial, mantiene una situación vigente y termina cuando se registra un hito de cierre. |
| Hito relacional | Registro estructural, perteneciente a un período, que deja constancia de su creación, cambio de situación o cierre. Cada hito se vincula a la experiencia que explica lo ocurrido, pero no es una experiencia. |
| Último | Acontecimiento confirmado más reciente según la cronología de [Relaciones](rules/RELATIONSHIPS.md#periodos-hitos); no la posición visual de una tarjeta. |
| Bloque inicial | Crush, período inicial, hito inicial y experiencia inicial obligatorios, unidos en el día del encuentro. |
| Vault | Espacio cifrado que conserva el contenido privado y permite recuperarlo sin exponerlo al servidor. Nombre técnico interno; la interfaz explica la protección del diario y distingue contraseña de acceso y clave de recuperación conforme a RP-VLT-038. |
| Pendiente | Cambio guardado en el dispositivo que todavía no fue confirmado completamente. |
| Día civil | Fecha local elegida, independiente de la zona horaria y del instante de transmisión. |
| Borrado funcional | Retirada del uso ordinario, que puede preceder a la destrucción física. |
| Seudonimización | Retirada de identificadores directos sin afirmar anonimato jurídico de la estructura restante. |

- **RP-ID-003.** Un mismo identificador se reconoce aunque cambie la combinación de mayúsculas y minúsculas con la que se presenta.
- **RP-ID-004.** Fechas de encuentro, experiencia, hito, nacimiento del Crush y confirmación de edad conservan el día elegido aunque cambie la zona horaria o el reintento ocurra otro día.
- **RP-ID-005.** Ninguna experiencia ni inicio/cambio relacional puede quedar antes del encuentro del crush.
- **RP-ID-006.** Los acontecimientos del diario admiten como fecha máxima el día civil local del dispositivo en cada intento de envío. Se comprueba su coherencia con el instante de recepción y la zona declarada; un contexto ausente o incoherente rechaza el intento y conserva el día funcional elegido. Ese contexto es transitorio y se recalcula al reintentar.

La identidad de las acciones y su confirmación completa se definen en [Guardado y sincronización](rules/SYNC.md#identidad).

<a id="lanzamiento"></a>
## Condiciones de lanzamiento

La distribución exige comprobar sus condiciones de lanzamiento.

- **RP-LAN-002.** El acceso con Apple no se distribuye públicamente hasta resolver la vinculación con cuentas existentes y comprobar duplicados, revocación, funcionamiento de Private Relay, avisos y textos legales.
- **RP-LAN-003.** Antes del lanzamiento deben estar comprobados el vencimiento de cuentas provisionales, la destrucción diferida del Vault, el borrado integral de cuenta y copias recuperables identificables de RP-BAJ-017..019 y, para las fotos locales, el saneamiento, el cifrado autenticado en reposo, la ausencia de copias persistentes en claro, la exclusión de backups, la retirada de memoria al bloquear y la eliminación correcta.
- **RP-LAN-004.** La beta activa se distribuye mediante iOS y API. En iOS están disponibles email y contraseña y acceso con Apple.
- **RP-LAN-007.** La apertura de la beta requiere validar los recorridos funcionales y destructivos con datos locales descartables, y las comprobaciones operativas acotadas y no destructivas del entorno de producción con una cuenta autorizada. El alcance y la evidencia se mantienen en [Operaciones API](../../api/docs/OPERATIONS.md#10-validacion-operativa-pre-beta).

<a id="fuentes"></a>
## Documentación relacionada

- [Funciones futuras aprobadas para Jira](FUNCIONES_FUTURAS_PARA_JIRA.md).
- [Decisiones pendientes para Jira](DECISIONES_PARA_JIRA.md).
- [Legal](legal/README.md).
- [Referencias técnicas](ARCHITECTURE.md).
