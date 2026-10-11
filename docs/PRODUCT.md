# Reglas de producto — MDL

Este es el **índice del catálogo de reglas funcionales de MyDuckingLife**. Las reglas se reparten por dominio en los documentos enlazados abajo; cada identificador tiene una única definición. Describen lo que el producto debe hacer, aunque su implementación esté pendiente.

**Estado del conjunto: FINALIZADO.** Producto cerró expresamente la revisión del catálogo. Este estado fija las reglas funcionales vigentes, pero no certifica implementación, tests, despliegues ni cumplimiento jurídico. Las decisiones identificadas como pendientes permanecen abiertas y ningún agente puede inventar su resolución.

Los documentos técnicos, los planes y el blueprint sirven como fuentes de información. No sustituyen las decisiones de producto ni convierten un comportamiento implementado en una regla. Si contradicen una regla, la discrepancia debe revisarse; no se cambia la regla para acomodar el código.

Legal mantiene su autoridad sobre privacidad, retención, derechos y cumplimiento. Las cuestiones jurídicas abiertas permanecen señaladas en el dominio correspondiente y no se resuelven por jerarquía automática.

<a id="documentos"></a>
## Documentos y estado de revisión

| Documento | Contenido | Estado |
|---|---|---|
| <a id="3-cuenta-autenticación-y-sesión"></a><a id="32-recuperación-de-la-contraseña-de-acceso"></a><a id="36-sign-in-with-apple"></a>[Cuenta de usuario](rules/ACCOUNT.md) | Alta y acceso, verificación, Apple, Google, perfil de la cuenta de usuario, ajustes y recuperación de acceso. | FINALIZADO |
| <a id="4-alta-y-perfil-de-un-crush"></a><a id="41-datos-obligatorios"></a><a id="42-resultado-de-match"></a><a id="43-edición-del-perfil"></a><a id="44-archivar-y-eliminar"></a>[Crush y perfil del Crush](rules/CRUSH.md) | Alta, datos y edición del perfil del Crush, resultado de match, edad, cumpleaños, zodíaco, valoraciones, fotos, consulta, mapa, archivo y eliminación. | FINALIZADO |
| <a id="6-experiencias"></a><a id="62-edición"></a><a id="63-borrado"></a>[Experiencias](rules/EXPERIENCES.md) | Experiencia inicial y posteriores: datos, fechas, creación, edición, eliminación y vinculación con relaciones. | FINALIZADO |
| <a id="7-relaciones"></a><a id="71-cronología"></a><a id="73-situaciones-y-trayectos-permitidos"></a><a id="74-cierre"></a><a id="75-registrar-corregir-y-eliminar"></a>[Relaciones](rules/RELATIONSHIPS.md) | Situaciones, períodos, hitos, evidencia relacional de match, trayectos, cierre y acciones latest-only. | FINALIZADO |
| <a id="81-contenido-e-identidad"></a><a id="83-acciones"></a><a id="84-actividad-reciente-de-home"></a>[Timeline y Home](rules/TIMELINE.md) | Presentación de la historia, orden, agrupación, acciones disponibles y actividad reciente de Home. | FINALIZADO |
| <a id="10-vault-privacidad-y-fotos"></a><a id="103-borrado-dirigido"></a>[Vault](rules/VAULT.md) | Contenido privado, creación, bloqueo, recuperación, sustitución de dispositivo y retirada de contenido. | FINALIZADO |
| <a id="9-escrituras-durables-y-sincronización-visible"></a>[Guardado y sincronización](rules/SYNC.md) | Qué significa guardar, cambios pendientes, conflictos, reintentos, corrección y descarte. | FINALIZADO |
| <a id="12-estadísticas"></a><a id="121-estadísticas"></a><a id="123-alcance-entre-cuentas"></a>[Estadísticas](rules/STATISTICS.md) | Dashboard personal de la beta, cálculos, ausencia de datos, métricas excluidas y alcance entre cuentas. | FINALIZADO |
| [Uso de la aplicación](rules/UX.md) | Tutoriales informativos, formularios, mensajes, errores, accesibilidad, carga, vacíos y desconexión. | FINALIZADO |
| [Analítica de uso en iOS](rules/ANALYTICS.md) | Medición agregada de pantallas y recorridos iOS, activación, opt-out, minimización, semántica y calidad. | FINALIZADO |
| <a id="15-baja-seudonimización-e-inactividad"></a><a id="16-procesos-automáticos-y-plazos"></a>[Baja, inactividad y ciclo de vida de los datos](rules/DATA_LIFECYCLE.md) | Baja voluntaria o por inactividad, seudonimización, destrucción, conservación y tareas automáticas. | FINALIZADO |
| [Privacidad, seguridad y derechos](rules/PRIVACY.md) | Aislamiento entre cuentas, protección de información, diagnóstico, soporte y ejercicio de derechos. | FINALIZADO |

<a id="revision"></a>
## Cómo se revisan y mantienen las reglas

- Cada archivo se revisa por separado. Sólo se marca **FINALIZADO** por indicación expresa de Producto; el índice y el archivo deben reflejar el mismo estado.
- Un archivo finalizado conserva sus decisiones. Cambiar una regla exige reabrir expresamente esa parte de la revisión; mover o reordenar documentos no autoriza a alterar su significado.
- Se conservan los identificadores `BR-*` y `RP-*`; no se reutilizan para una regla distinta. Otros documentos enlazan la definición que corresponda.
- Las decisiones pendientes pertenecen al documento del dominio. Este índice permite encontrarlas sin mantener otra copia.
- El estado de revisión de Producto y el cumplimiento de la implementación son cosas distintas. El catálogo finalizado se contrasta después con código, tests y documentación local sin adaptar las reglas al comportamiento encontrado.
- Se usa **perfil de la cuenta de usuario** para quien utiliza MDL y **perfil del Crush** para la ficha privada de la persona registrada en el diario.

<a id="1-propósito-y-límites"></a>
<a id="principios"></a>
## Principios generales y alcance

- **RP-BASE-003.** No hay perfiles públicos, interacción social entre cuentas ni administración funcional del diario ajeno.
- **RP-BASE-004.** La beta es gratuita, sin pagos ni suscripciones.
- **RP-BASE-006.** La primera versión pública aplica el mismo contrato funcional a todas las cuentas, datos y plataformas habilitadas.
- **RP-BASE-007.** El alcance funcional de la beta está formado por las funciones definidas como activas en este catálogo. Incorporar otra función exige una decisión de Producto y sus reglas antes de habilitarla.
- **RP-BASE-008.** Los clientes aplican las mismas reglas y garantías; las excepciones de plataforma requieren decisión explícita. La paridad de Android no se presume a partir de iOS.
- **RP-BASE-009.** La beta 1.0 parte de un contrato greenfield y no conserva compatibilidad con usuarios, datos, payloads, endpoints, flags, esquemas ni comportamientos de versiones pre-beta. Una rama, campo opcional, fallback, fixture o migración no permanece sólo porque una implementación anterior lo necesitó. Todo código ejecutable debe corresponder al contrato vigente; un estado obligatorio ausente o desconocido falla cerrado conforme a `RP-UX-008`. Una transición técnica sólo puede existir si un plan activo identifica un estado real del lanzamiento, su salida y su eliminación antes de cerrar la iniciativa.

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
- **RP-ID-006.** Los acontecimientos del diario no se registran en el futuro; su fecha máxima es el día civil local del dispositivo en el momento de cada intento de envío. El cliente declara ese día y su offset UTC vigente como contexto separado de la fecha funcional elegida, y los recalcula en cada reintento. La API exige un offset entre UTC−12 y UTC+14 y comprueba que el día declarado coincida exactamente con el instante UTC de recepción desplazado por ese offset; una declaración ausente, no canónica, fuera de rango o incoherente falla sin sustituirse por el día UTC del servidor. Este contexto no se persiste, no forma parte del perfil, no se incorpora al Vault ni modifica el día funcional elegido conforme a RP-ID-004.

La identidad de las acciones y su confirmación completa se definen en [Guardado y sincronización](rules/SYNC.md#identidad).

<a id="lanzamiento"></a>
## Condiciones de lanzamiento

Definir una regla no demuestra que esté implementada ni permite distribuir una función sin sus comprobaciones correspondientes.

- **RP-LAN-002.** El acceso con Apple no se distribuye públicamente hasta resolver la vinculación con cuentas existentes y comprobar duplicados, revocación, funcionamiento de Private Relay, avisos y textos legales.
- **RP-LAN-003.** Antes del lanzamiento deben estar comprobados el vencimiento de cuentas provisionales, la destrucción diferida del Vault y la exclusión y eliminación correcta de las fotos locales.
- **RP-LAN-004.** La beta activa se distribuye exclusivamente mediante iOS y API. Android no participa de esta beta: su paridad, sus validaciones y su compatibilidad con el Vault corresponden a una distribución futura y no bloquean el despliegue ni el cierre de la beta iOS/API. iOS ofrece email y contraseña y «Continuar con Apple», pero no muestra «Continuar con Google». Cuando Android se habilite en una distribución posterior, deberá superar sus gates propios, ofrecerá email y contraseña y «Continuar con Google», no mostrará «Continuar con Apple» y permitirá acceder a la misma cuenta y al mismo Vault conforme a las reglas de Cuenta.
- **RP-LAN-005.** «Continuar con Google» queda fuera de la beta activa iOS/API. Antes de una futura distribución Android debe comprobarse en conjunto validación server-side y anti-replay, vinculación explícita sin duplicados, revocación, sesión/Vault, configuración operativa, requisitos de marca y Google Play, disclosure y revisión humana/legal; esos gates no bloquean la beta iOS/API.

<a id="18-decisiones-pendientes"></a>
<a id="decisiones-pendientes"></a>
## Decisiones pendientes: dónde revisarlas

| Documento | Decisiones |
|---|---|
| [Relaciones](rules/RELATIONSHIPS.md#decisiones-pendientes) | [DEC-13](rules/RELATIONSHIPS.md#dec-13) |
| [Baja, inactividad y ciclo de vida de los datos](rules/DATA_LIFECYCLE.md#decisiones-pendientes) | [DEC-02](rules/DATA_LIFECYCLE.md#dec-02), [DEC-09](rules/DATA_LIFECYCLE.md#dec-09), [DEC-14](rules/DATA_LIFECYCLE.md#dec-14) |
| [Privacidad, seguridad y derechos](rules/PRIVACY.md#decisiones-pendientes) | [DEC-12](rules/PRIVACY.md#dec-12), [DEC-16](rules/PRIVACY.md#dec-16) |
| [Analítica de uso en iOS](rules/ANALYTICS.md#decisiones-pendientes) | [DEC-15](rules/ANALYTICS.md#dec-15) |

<a id="fuentes"></a>
## Fuentes de consulta

Los documentos de [arquitectura](ARCHITECTURE.md), [Timeline](TIMELINE.md), [Vault](VAULT.md) y [estadísticas](STATISTICS.md), los planes activos y la documentación local sirven para contextualizar y después contrastar las reglas. El [blueprint de negocio](product/business-blueprint.md) y el [expediente Legal](legal/README.md) mantienen su ámbito. El catálogo enlaza sus definiciones estables y no duplica sus textos.

| Dominio | Procedencia / detalle |
|---|---|
| Cuenta | [Reglas de Cuenta](rules/ACCOUNT.md), [Apple](plans/active/sign-in-with-apple/README.md), [Google](plans/active/sign-in-with-google/README.md) y [Privacidad y seguridad](rules/PRIVACY.md). |
| Diario | [Timeline](TIMELINE.md), [Relaciones](rules/RELATIONSHIPS.md), [Experiencias](rules/EXPERIENCES.md) y [guardado y sincronización](rules/SYNC.md). |
| Vault/borrado | [Vault](VAULT.md) y [plan](plans/active/vault-e2ee/README.md). |
| Estadísticas | [Statistics](STATISTICS.md), [detalle API](../../api/docs/STATISTICS.md). |
| UX | [Reglas por pantalla](plans/active/frontend-screen-rules/README.md), [formularios](plans/active/form-ux-performance/README.md), [vacíos](plans/active/empty-state-standardization/README.md). |
| Analítica iOS | [Reglas](rules/ANALYTICS.md) y [plan activo](plans/active/ios-product-analytics/README.md), bajo los gates de Privacidad, Legal y retención. |
| Automatismos | [Operación API](../../api/docs/OPERATIONS.md), scheduler `api/routes/console.php` consultado mediante MCP. |
| Privacidad | [Expediente Legal](legal/README.md) y [ciclo de vida](legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md), bajo autoridad jurídica propia. |
| Plataformas | [Android](plans/active/android-parity/README.md), [arquitectura](ARCHITECTURE.md), [operación API](../../api/docs/OPERATIONS.md) y [pruebas iOS](../../ios/docs/TESTING.md). |

La revisión de reglas y el contraste posterior se coordinan mediante [business-rule-integrity](plans/active/business-rule-integrity/README.md). No se mezclan aquí estados «cumple/parcial/incumple», símbolos de código ni resultados de tests.
