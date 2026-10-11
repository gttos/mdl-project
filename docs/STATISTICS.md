# Estadísticas — referencias de contrato

Las reglas completas, fórmulas, métricas excluidas y decisiones pendientes están reunidas en [Estadísticas](rules/STATISTICS.md). Este archivo sirve de entrada al detalle técnico; no mantiene una segunda definición de métricas.

## 1. Qué son

Referencia: `RP-EST-001` y `RP-EST-002`. La pantalla nativa consume el dashboard calculado por API. La implementación local y sus formatos están en [API Statistics](../../api/docs/STATISTICS.md).

## 2. Reglas canónicas

Referencia única: [Métricas y cálculos](rules/STATISTICS.md#metricas), `BR-12.01` y `RP-EST-001..025`. Allí se definen escala, precisión, denominadores, datos nulos, situación actual, match, secciones del dashboard, base y respaldo de cada dato, empates y estados de carga/error.

Correspondencia de transporte:

| Concepto | Campo |
|---|---|
| Porcentaje de Crushes con match entre los resultados resueltos | `summary.winRate` |
| Conteos `MATCH`, `NO_MATCH` y `UNKNOWN` | `match.byStatus` |
| Situación del primer match | `match.firstBySituation` |
| Satisfacción media | `summary.averageSatisfaction` |
| Ducky predominante | `summary.topDucky` |
| Situación actual | `summary.actualStatus` |
| Momento actual | `moment.activeCrushes`, `moment.bySituation` |

Las métricas basadas en observaciones publican `sample.used`, `sample.eligible`, `sample.required`, `sample.coveragePercentage` y `sample.supportLevel`. La API calcula también la base por cada valoración y dimensión física; iOS no deriva niveles, coberturas, líderes ni empates. Los conteos exactos del momento actual conservan su forma estructural y no muestran semáforo.

La presentación visible usa «Pocos datos», «Base media» y «Base amplia» junto al punto rojo, amarillo o verde correspondiente. Es una indicación del respaldo interno del dato, no una interpretación personal ni una certificación estadística externa. El dashboard permanece descriptivo; los informes narrativos o generados mediante IA quedan fuera de esta superficie.

En iOS, la cobertura identifica explícitamente su universo: Satisfacción habla de experiencias y las métricas basadas en Crushes hablan de perfiles. Los rasgos físicos conservan todos los líderes empatados y los muestran por dimensión en una fila compacta, con un máximo de tres iconos y `+N` como resumen visual sin ocultar sus nombres. Los nombres, iconos y glifos de catálogo representan siempre el mismo ID publicado. Ducky se describe como los tipos que más aparecen entre los Crushes, no como preferencias inferidas.

## 3. Métricas prohibidas

Referencia única: `BR-12.02` en [Métricas excluidas](rules/STATISTICS.md#metricas-excluidas). Se aplica al dashboard, al cálculo y a la presentación de estadísticas.

## 4. Qué sale realmente en la beta

Alcance normativo: `RP-EST-002`, `RP-EST-011`, `RP-EST-015` y [Alcance entre cuentas](rules/STATISTICS.md#entre-cuentas). La beta consume el dashboard final calculado por API: match se proyecta desde el Crush y el momento actual desde el período vivo, sin reconstruir la historia desde páginas parciales del Timeline ni fabricar valores cuando no existe denominador.

Producto conserva como función futura los agregados entre cuentas y, en particular, los patrones agrupados por país ya reservados técnicamente. Mantener sus tablas, modelos o campos vacíos no los convierte en una función activa de la beta. Su cálculo y la publicación de resultados no vacíos permanecen bloqueados por `DEC-02` hasta definir población, mínimos de cohorte, supresión, actualización, retención, acceso y salvaguardas con aprobación Legal.

## 5. Referencias

- [Contrato técnico API](../../api/docs/STATISTICS.md).
- [Decisiones abiertas del catálogo](PRODUCT.md#decisiones-pendientes).
