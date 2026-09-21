# Reglas de producto — MDL

Este es el **índice del catálogo de reglas funcionales de MyDuckingLife**. Las reglas se reparten por dominio en los documentos enlazados abajo; cada identificador tiene una única definición. Describen lo que el producto debe hacer, aunque su implementación esté pendiente.

**Estado del conjunto: EN REVISIÓN.** Primero se revisará cada archivo con Producto. Después se comprobará regla por regla en las aplicaciones, API y documentación. Esta reorganización no certifica implementación, tests, despliegues ni cumplimiento jurídico.

Los documentos técnicos, los planes y el blueprint sirven como fuentes de información. No sustituyen las decisiones de producto ni convierten un comportamiento implementado en una regla. Si contradicen una regla, la discrepancia debe revisarse; no se cambia la regla para acomodar el código.

Legal mantiene su autoridad sobre privacidad, retención, derechos y cumplimiento. Las cuestiones jurídicas abiertas permanecen señaladas en el dominio correspondiente y no se resuelven por jerarquía automática.

<a id="documentos"></a>
## Documentos y estado de revisión

| Documento | Contenido | Estado |
|---|---|---|
| <a id="3-cuenta-autenticación-y-sesión"></a><a id="32-recuperación-de-la-contraseña-de-acceso"></a><a id="36-sign-in-with-apple"></a>[Cuenta de usuario](rules/ACCOUNT.md) | Alta y acceso, verificación, Apple, perfil de la cuenta de usuario, ajustes y recuperación de acceso. | FINALIZADO |
| <a id="4-alta-y-perfil-de-un-crush"></a><a id="41-datos-obligatorios"></a><a id="43-edición-del-perfil"></a><a id="44-archivar-y-eliminar"></a>[Crush y perfil del Crush](rules/CRUSH.md) | Alta, datos y edición del perfil del Crush, proyecciones de match, edad, cumpleaños, zodíaco, valoraciones, fotos, consulta, mapa, archivo y eliminación. | EN REVISIÓN |
| <a id="6-experiencias"></a><a id="62-edición"></a><a id="63-borrado"></a>[Experiencias](rules/EXPERIENCES.md) | Experiencia inicial y posteriores: datos, fechas, creación, edición, eliminación y vinculación con relaciones. | EN REVISIÓN |
| <a id="7-relaciones"></a><a id="71-cronología"></a><a id="72-match-status"></a><a id="73-situaciones-y-trayectos-permitidos"></a><a id="74-cierre"></a><a id="75-registrar-corregir-y-eliminar"></a>[Relaciones](rules/RELATIONSHIPS.md) | Situaciones, períodos, hitos, match, trayectos, cierre y acciones latest-only. | EN REVISIÓN |
| <a id="81-contenido-e-identidad"></a><a id="83-acciones"></a><a id="84-actividad-reciente-de-home"></a>[Timeline y Home](rules/TIMELINE.md) | Presentación de la historia, orden, agrupación, acciones disponibles y actividad reciente de Home. | EN REVISIÓN |
| <a id="10-vault-privacidad-y-fotos"></a><a id="103-borrado-dirigido"></a>[Vault](rules/VAULT.md) | Contenido privado, creación, bloqueo, recuperación, sustitución de dispositivo y retirada de contenido. | EN REVISIÓN |
| <a id="9-escrituras-durables-y-sincronización-visible"></a>[Guardado y sincronización](rules/SYNC.md) | Qué significa guardar, cambios pendientes, conflictos, reintentos, corrección y descarte. | EN REVISIÓN |
| <a id="12-estadísticas-e-ia"></a><a id="121-estadísticas"></a><a id="122-ia-e-inferencias"></a><a id="123-alcance-entre-cuentas"></a>[Estadísticas e IA](rules/STATISTICS.md) | Dashboard personal de la beta, cálculos, ausencia de datos y límites de la IA y de los agregados entre cuentas. | EN REVISIÓN |
| [Uso de la aplicación](rules/UX.md) | Tutoriales informativos, formularios, mensajes, errores, accesibilidad, carga, vacíos y desconexión. | EN REVISIÓN |
| <a id="15-baja-seudonimización-e-inactividad"></a><a id="16-procesos-automáticos-y-plazos"></a>[Baja, inactividad y ciclo de vida de los datos](rules/DATA_LIFECYCLE.md) | Baja voluntaria o por inactividad, seudonimización, destrucción, conservación y tareas automáticas. | EN REVISIÓN |
| [Privacidad, seguridad y derechos](rules/PRIVACY.md) | Aislamiento entre cuentas, protección de información, diagnóstico, soporte y ejercicio de derechos. | EN REVISIÓN |

<a id="revision"></a>
## Cómo se revisan y mantienen las reglas

- Cada archivo se revisa por separado. Sólo se marca **FINALIZADO** por indicación expresa de Producto; el índice y el archivo deben reflejar el mismo estado.
- Un archivo finalizado conserva sus decisiones. Cambiar una regla exige reabrir expresamente esa parte de la revisión; mover o reordenar documentos no autoriza a alterar su significado.
- Se conservan los identificadores `BR-*` y `RP-*`; no se reutilizan para una regla distinta. Otros documentos enlazan la definición que corresponda.
- Las decisiones pendientes pertenecen al documento del dominio. Este índice permite encontrarlas sin mantener otra copia.
- El estado de revisión de Producto y el cumplimiento de la implementación son cosas distintas. La auditoría de cumplimiento y la actualización de las instrucciones de agentes como reglas finalizadas se harán después de cerrar esta revisión.
- Se usa **perfil de la cuenta de usuario** para quien utiliza MDL y **perfil del Crush** para la ficha privada de la persona registrada en el diario.

<a id="1-propósito-y-límites"></a>
<a id="principios"></a>
## Principios generales y alcance

- **RP-BASE-003.** No hay perfiles públicos, interacción social entre cuentas ni administración funcional del diario ajeno.
- **RP-BASE-004.** La beta es gratuita, sin pagos ni suscripciones.
- **RP-BASE-006.** La primera versión pública aplica el mismo contrato funcional a todas las cuentas, datos y plataformas habilitadas.
- **RP-BASE-007.** El alcance funcional de la beta está formado por las funciones definidas como activas en este catálogo. Incorporar otra función exige una decisión de Producto y sus reglas antes de habilitarla.
- **RP-BASE-008.** Los clientes aplican las mismas reglas y garantías; las excepciones de plataforma requieren decisión explícita. La paridad de Android no se presume a partir de iOS.

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
| Situación relacional | Estado exacto elegido para la historia. Se guarda mediante el ID de `RelationshipSituation`; sus claves internas permanecen en inglés y la app muestra copy `es-ES`. |
| Período relacional | Tramo de la historia relacional que comienza con un hito inicial, mantiene una situación vigente y termina cuando se registra un hito de cierre. |
| Hito relacional | Registro estructural, perteneciente a un período, que deja constancia de su creación, cambio de situación o cierre. Cada hito se vincula a la experiencia que explica lo ocurrido, pero no es una experiencia. |
| Último | Acontecimiento confirmado más reciente según la cronología de [Relaciones](rules/RELATIONSHIPS.md#periodos-hitos); no la posición visual de una tarjeta. |
| Bloque inicial | Crush, período inicial, hito inicial y experiencia inicial obligatorios, unidos en el día del encuentro. |
| Vault | Espacio cifrado que conserva el contenido privado y permite recuperarlo sin exponerlo al servidor. |
| Pendiente | Cambio guardado en el dispositivo que todavía no fue confirmado completamente. |
| Día civil | Fecha local elegida, `yyyy-MM-dd`; no un instante UTC ni una ventana móvil de 24 horas. |
| Borrado funcional | Retirada del uso ordinario, que puede preceder a la destrucción física. |
| Seudonimización | Retirada de identificadores directos sin afirmar anonimato jurídico de la estructura restante. |

- **RP-ID-003.** Un mismo identificador se reconoce aunque cambie la combinación de mayúsculas y minúsculas con la que se presenta.
- **RP-ID-004.** Fechas de encuentro, experiencia, hito, nacimiento del Crush y confirmación de edad conservan el día elegido aunque cambie la zona horaria o el reintento ocurra otro día.
- **RP-ID-005.** Ninguna experiencia ni inicio/cambio relacional puede quedar antes del encuentro del crush.
- **RP-ID-006.** Los acontecimientos del diario no se registran en el futuro; su fecha máxima es hoy.

La identidad de las acciones y su confirmación completa se definen en [Guardado y sincronización](rules/SYNC.md#identidad).

<a id="lanzamiento"></a>
## Condiciones de lanzamiento

Definir una regla no demuestra que esté implementada ni permite distribuir una función sin sus comprobaciones correspondientes.

- **RP-LAN-002.** El acceso con Apple no se distribuye públicamente hasta resolver la vinculación con cuentas existentes y comprobar duplicados, revocación, funcionamiento de Private Relay, avisos y textos legales.
- **RP-LAN-003.** Antes del lanzamiento deben estar comprobados el vencimiento de cuentas provisionales, la destrucción diferida del Vault y la exclusión y eliminación correcta de las fotos locales.
- **RP-LAN-004.** Android requiere validación propia y compatibilidad del Vault con las demás plataformas. La disponibilidad de Apple en Android sigue pendiente de decisión.

<a id="18-decisiones-pendientes"></a>
<a id="decisiones-pendientes"></a>
## Decisiones pendientes: dónde revisarlas

| Documento | Decisiones |
|---|---|
| [Relaciones](rules/RELATIONSHIPS.md#decisiones-pendientes) | [DEC-13](rules/RELATIONSHIPS.md#dec-13) |
| [Vault](rules/VAULT.md#decisiones-pendientes) | [DEC-10](rules/VAULT.md#dec-10) |
| [Estadísticas e IA](rules/STATISTICS.md#decisiones-pendientes) | [DEC-08](rules/STATISTICS.md#dec-08) |
| [Baja, inactividad y ciclo de vida de los datos](rules/DATA_LIFECYCLE.md#decisiones-pendientes) | [DEC-02](rules/DATA_LIFECYCLE.md#dec-02), [DEC-03](rules/DATA_LIFECYCLE.md#dec-03), [DEC-09](rules/DATA_LIFECYCLE.md#dec-09) |
| [Privacidad, seguridad y derechos](rules/PRIVACY.md#decisiones-pendientes) | [DEC-12](rules/PRIVACY.md#dec-12) |
| Cuenta y plataformas | [DEC-11](#dec-11) |

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-11"></a>DEC-11 | Apple Android y condiciones de distribución por plataforma. | Paridad sin activación implícita de proveedores. |

<a id="fuentes"></a>
## Fuentes de consulta

Los documentos de [arquitectura](ARCHITECTURE.md), [Timeline](TIMELINE.md), [Vault](VAULT.md) y [estadísticas](STATISTICS.md), los planes activos y la documentación local sirven para contextualizar y después contrastar las reglas. El [blueprint de negocio](product/business-blueprint.md) y el [expediente Legal](legal/README.md) mantienen su ámbito. El catálogo enlaza sus definiciones estables y no duplica sus textos.

| Dominio | Procedencia / detalle |
|---|---|
| Cuenta | [Email](plans/active/email-verification/README.md), [Apple](plans/active/sign-in-with-apple/README.md), [seguridad](plans/active/security-remediation/README.md). |
| Diario | [Timeline](TIMELINE.md), [Relaciones](plans/active/relationship-redesign/README.md), [durabilidad](plans/active/durable-write-feedback/README.md). |
| Vault/borrado | [Vault](VAULT.md) y [plan](plans/active/vault-e2ee/README.md). |
| Estadísticas | [Statistics](STATISTICS.md), [detalle API](../../api/docs/STATISTICS.md). |
| UX | [Formularios](plans/active/form-ux-performance/README.md), [vacíos](plans/active/empty-state-standardization/README.md). |
| Automatismos | [Operación API](../../api/docs/OPERATIONS.md), scheduler `api/routes/console.php` consultado mediante MCP. |
| Privacidad | [Expediente Legal](legal/README.md) y [ciclo de vida](legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md), conservados sin modificación. |
| Plataformas | [Android](plans/active/android-parity/README.md), [entorno local](plans/active/local-integration-environment/README.md), [arquitectura](ARCHITECTURE.md). |

La revisión de reglas y el contraste posterior se coordinan mediante [business-rule-integrity](plans/active/business-rule-integrity/README.md). No se mezclan aquí estados «cumple/parcial/incumple», símbolos de código ni resultados de tests.
