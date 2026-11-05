# Adjetivos negativos

<a id="objetivo-y-alcance"></a><a id="catálogo-y-persistencia-contrastados"></a><a id="fuente-del-catálogo-decisión-aprobada"></a><a id="propuesta-para-revisar"></a><a id="decisiones-abiertas"></a><a id="fases-y-responsabilidades"></a><a id="escenarios-que-debe-concretar-neg-01"></a><a id="cierre"></a>

> **Estado:** ampliación y base de datos como fuente única aprobadas; lista final y diseño pendientes. Sólo planificación. No se han activado opciones, cambiado contratos, modificado datos ni ejecutado tests de esta iniciativa.
> **Autoridad:** [catálogo vigente, RP-ADJ-001..010](../../../rules/CRUSH.md#adjetivos), [ampliación y fuente aprobadas, RP-ADJ-011/012](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#adjetivos-negativos), [estadísticas de cualidades](../../../rules/STATISTICS.md#adjetivos) y [privacidad](../../../rules/PRIVACY.md#propiedad).

## Objetivo y alcance

Ampliar el catálogo compartido para que la persona pueda describirse y describir a cada Crush con rasgos negativos, además de las cualidades y rasgos actuales. Cerrar primero qué opciones aportan información útil y cómo presentarlas sin sobrecargar el formulario. Son apreciaciones manuales de quien usa el diario, no diagnósticos ni conclusiones del sistema.

Incluye catálogo y definiciones ES/EN, identidad y bloques, selección, persistencia, lecturas, corrección durable, estadísticas personales y presentación coherente en API/iOS. Incluye trasladar la autoridad del catálogo a la base de datos y retirar el archivo alternativo. No incluye textos libres nuevos, evaluaciones clínicas, cambios de match o situaciones, LLM, agregados entre cuentas, borrado de cuentas, transporte MySQL/DynamoDB del Vault ni despliegue. Los límites actuales no cambian sin decisión expresa.

## Dependencias

- [Ampliación aprobada](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#adjetivos-negativos).
- [Decisiones NEG-01](../../../DECISIONES_PARA_JIRA.md#neg-01).

## Fases y responsabilidades

Una fase por sesión. Los planes locales API/iOS se crean al cerrar NEG-01, con el alcance técnico aprobado; no se inicia código con catálogo abierto.

| Fase | Trabajo | Salida | Estado |
|---|---|---|---|
| NEG-01 | Product: resolver las decisiones anteriores, revisar definiciones; fijar catálogo e impacto de privacidad. | RP-ADJ y RP-EST reconciliadas, escenarios finales y planes locales API/iOS; ningún cambio de datos. | Pendiente; siguiente fase. |
| NEG-02 | API: mapear catálogo, validación, almacenamiento, lecturas, seeders y consumidores de estadísticas; implementar el contrato cerrado con la base de datos como fuente única y retirar el catálogo en archivo. | Pruebas dirigidas y regresión proporcional; IDs coherentes, consulta y validación desde base de datos, rechazo atómico de entradas inválidas y ausencia de autoridad/fallback alternativos. Cualquier limpieza de filas reales requiere alcance y autorización propios. | Pendiente de NEG-01. |
| NEG-03 | iOS: adaptar catálogo canónico, caché, selección propia/Crush, snapshots, lecturas, estadísticas y demo afectada. | Build y pruebas pertinentes, comparación visual ES/EN y texto largo; sin pérdida al editar, corregir o relanzar. | Pendiente de API/contrato cerrado. |
| NEG-04 | Integración y cierre coordinado. | Recorrido sintético API/app, decisiones resueltas, documentación estable y retirada de planes. Distribución y operación se autorizan aparte. | Pendiente. |

## Escenarios que debe concretar NEG-01

| ID | Dado / cuando / entonces | Referencia |
|---|---|---|
| GWT-NEG-01 | Dado el catálogo aprobado, cuenta y Crush muestran las mismas entradas y definiciones en ES/EN; un ID conserva su significado y las selecciones son independientes. | RP-ADJ-001/007/011 |
| GWT-NEG-02 | Dada una fila antigua fuera del catálogo aprobado, no aparece ni se admite por existir en MySQL; una edición inválida se rechaza completa. | RP-ADJ-001/002/011 |
| GWT-NEG-03 | Dada una selección ampliada válida, guardar, leer, corregir y reintentar conservan exactamente su conjunto y los demás datos; ocultar opciones no las vacía. | RP-ADJ-003/005/006 |
| GWT-NEG-04 | Dado un catálogo o respuesta incompletos o incompatibles, no se inventan etiquetas ni se pierden selecciones; se conserva el fallo cerrado y el manejo explícito de carga/error. | RP-ADJ-005 |
| GWT-NEG-05 | Dadas selecciones del Crush y de la cuenta, el dashboard aplica exclusivamente el universo y presentación aprobados; la autodescripción no entra por accidente en estadísticas de Crushes. | RP-ADJ-009, RP-EST-028..030 |
| GWT-NEG-06 | Dado el catálogo aprobado en base de datos, API lo publica y valida desde esa fuente, el cliente lo consulta mediante API y no existe catálogo alternativo en archivo ni fallback que determine otras opciones. | RP-ADJ-012 |

Estos escenarios son preparación del diseño, no evidencia ejecutada. NEG-01 debe fijar límites, combinaciones admitidas y cálculo antes de convertirlos en pruebas de aceptación finales.

## Cierre

Integrar reglas, catálogo y escenarios finales en Product y contratos/evidencia en documentos estables API/iOS antes de retirar maestro y planes locales. No archivar planes ni mantener listas paralelas de reglas. No modificar Legal, datos reales o recursos AWS por crear esta feature.
