# Estadísticas e IA — MDL

Qué son las estadísticas de MDL, qué límites éticos tienen, y qué parte de la API que las sirve realmente se usa en la beta. El contrato técnico completo (formato de respuesta, catálogo de endpoints, estructura del payload de IA) vive en [`api/docs/STATISTICS.md`](../../api/docs/STATISTICS.md) — este documento no lo repite.

## 1. Qué son

Las estadísticas son de solo lectura, calculadas por la API sobre los datos estructurales de la propia cuenta. La app nunca calcula ratios, distribuciones o comparaciones por su cuenta — sólo renderiza lo que la API ya calculó.

## 2. Reglas canónicas

- Toda valoración/satisfacción se expresa en escala **1..5** en la respuesta (en base de datos la satisfacción vive en −2..2; la API siempre la convierte antes de exponerla). Nunca 1..10 ni −2..2 hacia afuera. (BR-12.01)
- Todo número con 2 decimales.
- Un win rate nunca puede superar el 100% — se calcula por Crush distinto, evitando el inflado por JOINs con experiencias múltiples.
- La API nunca responde 404/500 por falta de datos: responde `200` con estructura vacía y `status: empty`. Un usuario sin historial ve un estado vacío, no un error.

## 3. Métricas prohibidas

Política dura, no una guía — dado lo sensible del contenido que la app maneja, estas métricas **nunca** se calculan ni se muestran, en ningún endpoint ni en el payload de IA: (BR-12.02)

- rankings o comparaciones competitivas entre personas (crushes) de la usuaria;
- conteo tipo "body count";
- predicción de fracaso de un vínculo todavía activo;
- comparación competitiva con otras usuarias ("sos mejor que el X%") — el contexto de comunidad, si se implementa alguna vez, es sólo un promedio suave ("el promedio general es X"), nunca un ranking;
- "valor de mercado" o métricas equivalentes;
- tracking de revisitas/ghosting de un crush específico.

## 4. Filosofía de IA

- La API entrega datos, escalas y cruces numéricos con criterio cerrado; **nunca redacta prosa ni interpreta causalidad**.
- Una integración futura de IA deberá redactar dentro de los límites (`allowed_interpretations`/`forbidden_claims`) que la API marca por cada señal.
- Ninguna señal con confiabilidad `very_low` se emite — evita que el LLM alucine sobre muestras insuficientes.
- No hay entrenamiento de modelos con datos de usuario.
- **No existe hoy una integración activa de IA en la beta.** La API expone un `ai-payload` estructurado, pero el prototipo de `llm/` no lo consume y no está conectado a `api/` ni a los clientes nativos.
- El prototipo acepta técnicamente texto libre (`notes`) y resúmenes de eventos; esa capacidad no está autorizada para datos reales. Cualquier activación exige antes un contrato Product/Legal de finalidad, minimización, retención, consentimiento y proveedor.

## 5. Qué sale realmente en la beta

La única superficie de Statistics incluida en la beta es `GET /api/statistics/dashboard`. iOS la consume mediante `StatisticsService.getDashboard()` y toda su pantalla de estadísticas se renderiza desde esa única respuesta. Android deberá conservar el mismo límite cuando incorpore la pantalla.

`statistics/overview` no existe en API ni tiene consumidor: fue retirado y no se reactiva. Los otros 13 endpoints registrados —doce vistas de detalle y `ai-payload`— permanecen fuera de la beta y responden sus formas vacías estables. Ningún cliente debe llamarlos ni ofrecer navegación hacia ellos; sus fórmulas futuras requieren una decisión Product separada.

El dashboard sí debe recuperar datos reales compatibles con el modelo nativo. La mayor parte de sus teasers depende sólo de Crush o Experience: satisfacción media/distribución, Ducky predominante, atracción, rasgos, físicos, zodíaco, países y género. Tres elementos requieren semántica de períodos:

- `summary.winRate`: porcentaje de Crushes visibles y no borrados del usuario que alcanzaron Involucrado alguna vez (`everMatched`) sobre el total de Crushes visibles y no borrados de ese usuario. Numerador y denominador cuentan Crushes distintos; no se reutiliza `match_status_id`, el conjunto legacy “finalizados” ni el número de Experiences. Se redondea a dos decimales, queda entre 0 y 100 y, cuando el denominador es cero, se representa como dato no disponible (`null`/estado vacío), no como una medición de 0%.
- `summary.actualStatus`: `single`, `in_relationship` o `multiple_relationships`, contando únicamente períodos vivos en fase Involucrado cuya etiqueta sea Relación.
- `moment`: `activeCrushes` cuenta Crushes cuyo período vigente está vivo y el desglose es por fase nativa (`byPhase`), nunca por `RelationshipState`/`byState`.

Una respuesta sin muestra suficiente conserva `status: empty`/`locked` según el contrato de cada teaser; no debe presentar ceros como estadísticas reales.

## 6. Referencias

- Contrato técnico completo: [`api/docs/STATISTICS.md`](../../api/docs/STATISTICS.md)
- Ideas futuras (contexto de comunidad, deuda técnica): [`PRODUCT.md`](PRODUCT.md) §14.2
