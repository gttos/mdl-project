# Reglas de producto — Estadísticas

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Dashboard personal de la beta, cálculos, ausencia de datos, métricas excluidas y alcance entre cuentas.

<a id="metricas"></a>
## Métricas y cálculos

- **RP-EST-001.** Las estadísticas son de sólo lectura y se calculan en la API exclusivamente con la estructura perteneciente a la cuenta autenticada. Las aplicaciones presentan el resultado recibido y no recalculan ratios, medias ni distribuciones.
- **RP-EST-002.** La beta ofrece un único dashboard personal calculado sobre todo el historial elegible de la cuenta.
- **RP-EST-015.** El dashboard de la beta se compone exactamente de: resumen (`summary.winRate`, `summary.averageSatisfaction`, `summary.topDucky`, `summary.actualStatus`); match (`match.byStatus`, `match.firstBySituation`); momento actual (`moment.activeCrushes`, `moment.bySituation`); distribución de satisfacción (`satisfaction`); perfil de valoraciones (`attraction`); rasgos físicos (`physical`); zodíaco (`zodiac`); países (`countries`); y Ducky (`ducks`). Cada sección variable informa `status`, `value`, `sample.used` y `sample.required`.
- **BR-12.01.** Las valoraciones y la satisfacción utilizan la escala entera de 1 a 5. La API convierte esos IDs a sus valores numéricos para calcular estadísticas.
- **RP-EST-003.** Las medias y porcentajes se redondean a dos decimales. Los conteos representan cantidades reales y se muestran como enteros.
- **RP-EST-004.** Win rate = 100 × Crushes históricos elegibles con `match_status_id = MATCH` / Crushes históricos elegibles con `match_status_id` resuelto (`MATCH + NO_MATCH`). Cuenta Crushes distintos, nunca supera 100% y excluye `UNKNOWN` del denominador. `match.byStatus` informa por separado los tres conteos `MATCH`, `NO_MATCH` y `UNKNOWN`.
- **RP-EST-005.** Denominador cero implica no disponible, no 0% medido.
- **RP-EST-006.** La satisfacción media utiliza las experiencias elegibles y sus valores de 1 a 5. Sin experiencias elegibles, el resultado es nulo.
- **RP-EST-007.** El Ducky predominante es la única categoría con mayor frecuencia entre los Crushes elegibles. Sin muestra o cuando dos o más categorías empatan en la frecuencia máxima, no existe un Ducky predominante y el resultado es nulo; nunca se escoge uno mediante orden de ID, antigüedad ni otro desempate arbitrario.
- **RP-EST-008.** `actualStatus` cuenta períodos vivos cuya `current_situation_id` es `8 RELATIONSHIP`: cero produce `single`, uno produce `in_relationship` y más de uno produce `multiple_relationships`.
- **RP-EST-009.** `activeCrushes` cuenta Crushes con un período vigente vivo. `bySituation` distribuye esos períodos por `current_situation_id`; no existe una agrupación por fase o etiqueta interna.
- **RP-EST-010.** Sin Crushes activos no archivados, `moment.status` es `empty` y `moment.value` es nulo. Con al menos un Crush activo no archivado y ningún período vivo elegible, `moment.status` es `available` y `activeCrushes` vale cero.
- **RP-EST-011.** Match utiliza las proyecciones `match_status_id` y `first_match_situation_id` del Crush; `firstBySituation` cuenta cómo se produjo el primer match únicamente entre las situaciones 5–9. Satisfacción utiliza `satisfaction_id`; Ducky utiliza `ducky_id`; atracción utiliza las cuatro valoraciones y `rate_global`; rasgos físicos utilizan `height_id`, `eye_color_id`, `hair_color_id`, `body_type_id` y `skin_color_id`; zodíaco utiliza `zodiac_id`; y países utilizan `country_id`. El dashboard sólo calcula esas dimensiones con los campos estructurales indicados.
- **RP-EST-012.** Cuando una métrica no tiene datos suficientes, la API devuelve la sección con su estado y muestra correspondientes, con `value` nulo cuando corresponda. La aplicación distingue ese estado de una carga en curso o un error.
- **RP-EST-013.** Toda comparación, cobertura o indicación de confiabilidad se basa en valores calculados y publicados por la API. La aplicación no completa valores ausentes ni fabrica explicaciones.
- **RP-EST-014.** Durante carga o error se conserva la estructura visual de la pantalla, se identifica el estado y se ofrece una acción de reintento comprensible.
- **RP-EST-016.** Toda métrica basada en observaciones utiliza `sample.required = 1`: con al menos un registro elegible se calcula y presenta el resultado junto con `sample.used`; no se bloquea por alcanzar un umbral arbitrario mayor. Con cero registros aplica el estado vacío o el valor nulo definido para esa métrica. Los conteos y estados estructurales de RP-EST-008..010 conservan su semántica propia y tampoco se ocultan por tamaño de muestra.
- **RP-EST-017.** Las métricas históricas —win rate, satisfacción, Ducky, match, primer match y distribuciones— incluyen los Crushes activos y archivados de la cuenta porque archivar conserva la ficha y su historia. Las métricas del momento actual —`activeCrushes`, `bySituation` y `actualStatus`— utilizan únicamente Crushes activos no archivados. Los Crushes eliminados o seudonimizados quedan fuera de ambos alcances.

<a id="metricas-excluidas"></a>
## Métricas excluidas

- **BR-12.02.** El dashboard, la API de estadísticas y sus presentaciones excluyen rankings competitivos de Crushes o cuentas, body count, predicciones de fracaso de un vínculo activo, percentiles de superioridad, valor de mercado y seguimiento individual de revisitas o ghosting.

<a id="entre-cuentas"></a>
## Alcance entre cuentas

- **RP-FUT-002.** Las estadísticas de la beta son personales: cada dashboard utiliza únicamente datos de su cuenta propietaria y no combina datos de distintas cuentas.
- **RP-FUT-003.** La estructura seudonimizada conservada tras una baja permanece fuera del dashboard personal y de cualquier consulta individual. Su conservación no habilita por sí sola un agregado entre cuentas.

Las proyecciones de match y primer match pertenecen al [Crush](CRUSH.md#match). Estadísticas las consume como estructura ya resuelta y no reconstruye la historia de períodos e hitos.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

No hay decisiones funcionales pendientes en este capítulo.
