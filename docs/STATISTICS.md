# Estadísticas — referencias de contrato

Las reglas completas, fórmulas, métricas excluidas y decisiones pendientes están reunidas en [Estadísticas](rules/STATISTICS.md). Este archivo sirve de entrada al detalle técnico; no mantiene una segunda definición de métricas.

## 1. Qué son

Referencia: `RP-EST-001` y `RP-EST-002`. La pantalla nativa consume el dashboard calculado por API. La implementación local y sus formatos están en [API Statistics](../../api/docs/STATISTICS.md).

## 2. Reglas canónicas

Referencia única: [Métricas y cálculos](rules/STATISTICS.md#metricas), `BR-12.01` y `RP-EST-001..015`. Allí se definen escala, precisión, denominadores, datos nulos, situación actual, match, secciones del dashboard y estados de carga/error.

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

## 3. Métricas prohibidas

Referencia única: `BR-12.02` en [Métricas excluidas](rules/STATISTICS.md#metricas-excluidas). Se aplica al dashboard, al cálculo y a la presentación de estadísticas.

## 4. Qué sale realmente en la beta

Alcance normativo: `RP-EST-002`, `RP-EST-011`, `RP-EST-015` y [Alcance entre cuentas](rules/STATISTICS.md#entre-cuentas). Estado de implementación y validaciones: [plan relacional](plans/active/relationship-redesign/README.md), incluido su alcance Statistics.

## 5. Referencias

- [Contrato técnico API](../../api/docs/STATISTICS.md).
- [Decisiones abiertas del catálogo](PRODUCT.md#decisiones-pendientes).
