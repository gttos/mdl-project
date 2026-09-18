# Estadísticas e IA — referencias de contrato

Las reglas completas, fórmulas, prohibiciones y decisiones pendientes están reunidas en [Estadísticas e IA](rules/STATISTICS.md). Este archivo sirve de entrada al detalle técnico; no mantiene una segunda definición de métricas.

## 1. Qué son

Referencia: `RP-EST-001` y `RP-EST-002`. La pantalla nativa consume el dashboard calculado por API. La implementación local y sus formatos están en [API Statistics](../../api/docs/STATISTICS.md).

## 2. Reglas canónicas

Referencia única: [Métricas y cálculos](rules/STATISTICS.md#metricas), `BR-12.01` y `RP-EST-001..015`. Allí se definen escala, precisión, denominadores, datos nulos, estado actual, fases, secciones del dashboard y estados de carga/error.

Correspondencia de transporte:

| Concepto | Campo |
|---|---|
| Porcentaje de crushes que alcanzaron Involucrado | `summary.winRate` |
| Satisfacción media | `summary.averageSatisfaction` |
| Ducky predominante | `summary.topDucky` |
| Situación actual | `summary.actualStatus` |
| Momento actual | `moment.activeCrushes`, `moment.byPhase` |

## 3. Métricas prohibidas

Referencia única: `BR-12.02` en [IA e inferencias](rules/STATISTICS.md#ia). Se aplica a presentación, cálculo y cualquier futuro contexto de IA.

## 4. Filosofía de IA

Referencia única: `RP-IA-001..007`. La existencia de un prototipo o formato `ai-payload` no declara una integración activa ni autoriza datos reales.

## 5. Qué sale realmente en la beta

Alcance normativo: `RP-EST-002`, `RP-EST-011`, `RP-EST-015` y [Alcance entre cuentas](rules/STATISTICS.md#entre-cuentas). Estado de implementación y validaciones: [plan relacional](plans/active/relationship-redesign/README.md), incluido su alcance Statistics.

## 6. Referencias

- [Contrato técnico API](../../api/docs/STATISTICS.md).
- [Decisiones abiertas del catálogo](PRODUCT.md#decisiones-pendientes).
