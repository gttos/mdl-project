# Reglas de producto — Estadísticas e IA

**Estado de revisión: EN REVISIÓN.** Este archivo sólo se marca como finalizado cuando la persona responsable de Producto lo indique expresamente. Ese estado no certifica implementación ni aprobación jurídica.

[Índice de reglas](../PRODUCT.md) · Dashboard personal de la beta, cálculos, ausencia de datos y límites de la IA y de los agregados entre cuentas.

<a id="metricas"></a>
## Métricas y cálculos

- **RP-EST-001.** Las estadísticas son de sólo lectura y se calculan en la API exclusivamente con la estructura perteneciente a la cuenta autenticada. Las aplicaciones presentan el resultado recibido y no recalculan ratios, medias ni distribuciones.
- **RP-EST-002.** La beta ofrece un único dashboard personal calculado sobre todo el historial elegible de la cuenta.
- **RP-EST-015.** El dashboard de la beta se compone exactamente de: resumen (`summary.winRate`, `summary.averageSatisfaction`, `summary.topDucky`, `summary.actualStatus`); momento actual (`moment.activeCrushes`, `moment.byPhase`); distribución de satisfacción (`satisfaction`); perfil de valoraciones (`attraction`); rasgos físicos (`physical`); zodíaco (`zodiac`); países (`countries`); y Ducky (`ducks`). Cada sección variable informa `status`, `value`, `sample.used` y `sample.required`.
- **BR-12.01.** Las valoraciones y la satisfacción utilizan la escala entera de 1 a 5. La API convierte esos IDs a sus valores numéricos para calcular estadísticas.
- **RP-EST-003.** Las medias y porcentajes se redondean a dos decimales. Los conteos representan cantidades reales y se muestran como enteros.
- **RP-EST-004.** Win rate = 100 × crushes visibles no borrados que alcanzaron Involucrado / total visible no borrado propio. Ambos cuentan crushes distintos; nunca >100%.
- **RP-EST-005.** Denominador cero implica no disponible, no 0% medido.
- **RP-EST-006.** La satisfacción media utiliza las experiencias elegibles y sus valores de 1 a 5. Sin experiencias elegibles, el resultado es nulo.
- **RP-EST-007.** El Ducky predominante es la categoría más frecuente entre los Crushes elegibles. Sin muestra, el resultado es nulo. El desempate permanece en DEC-08.
- **RP-EST-008.** `actualStatus` cuenta períodos vivos en fase Involucrado con etiqueta Relación: cero produce `single`, uno produce `in_relationship` y más de uno produce `multiple_relationships`.
- **RP-EST-009.** `activeCrushes` cuenta Crushes con un período vigente vivo. `byPhase` distribuye esos períodos por `connection_phase_id`.
- **RP-EST-010.** Sin Crushes visibles, `moment.status` es `empty` y `moment.value` es nulo. Con Crushes visibles y ningún período vivo, `moment.status` es `available` y `activeCrushes` vale cero.
- **RP-EST-011.** Satisfacción utiliza `satisfaction_id`; Ducky utiliza `ducky_id`; atracción utiliza las cuatro valoraciones y `rate_global`; rasgos físicos utilizan `height_id`, `eye_color_id`, `hair_color_id`, `body_type_id` y `skin_color_id`; zodíaco utiliza `zodiac_id`; y países utilizan `country_id`. El dashboard sólo calcula esas dimensiones con los campos estructurales indicados.
- **RP-EST-012.** Cuando una métrica no tiene datos suficientes, la API devuelve la sección con su estado y muestra correspondientes, con `value` nulo cuando corresponda. La aplicación distingue ese estado de una carga en curso o un error.
- **RP-EST-013.** Toda comparación, cobertura o indicación de confiabilidad se basa en valores calculados y publicados por la API. La aplicación no completa valores ausentes ni fabrica explicaciones.
- **RP-EST-014.** Durante carga o error se conserva la estructura visual de la pantalla, se identifica el estado y se ofrece una acción de reintento comprensible.

<a id="ia"></a>
## IA e inferencias

- **BR-12.02.** La UI, la API y cualquier contexto automatizado excluyen rankings competitivos de Crushes o cuentas, body count, predicciones de fracaso de un vínculo activo, percentiles de superioridad, valor de mercado y seguimiento individual de revisitas o ghosting.
- **RP-IA-001.** La beta funciona sin IA narrativa. Los prototipos utilizan datos sintéticos y no acceden a diarios, notas ni Vaults reales.
- **RP-IA-002.** Activar una función de IA exige definir y aprobar previamente su finalidad, proveedor, datos mínimos, retención y consentimiento con Producto y Legal.
- **RP-IA-003.** La API sólo puede aportar señales calculadas y explicables. Una presentación automatizada no convierte una asociación en una explicación causal.
- **RP-IA-004.** Una asociación estadística se presenta como asociación y una señal con confiabilidad insuficiente no se convierte en afirmación.
- **RP-IA-005.** Un insight auxiliar nunca crea, edita, elimina ni reclasifica automáticamente un Crush, una experiencia, un período o un hito.
- **RP-IA-006.** Los datos de las cuentas no se utilizan para entrenar modelos generales. Cambiar esta finalidad requiere una decisión expresa de Producto y Legal.
- **RP-IA-007.** Un modelo de lenguaje no recibe claves, credenciales, bases locales ni contenido del Vault.

<a id="entre-cuentas"></a>
## Alcance entre cuentas

- **RP-FUT-002.** Las estadísticas de la beta son personales: cada dashboard utiliza únicamente datos de su cuenta propietaria y no combina datos de distintas cuentas.
- **RP-FUT-003.** La estructura seudonimizada conservada tras una baja permanece fuera del dashboard personal y de cualquier consulta individual. Su conservación no habilita por sí sola un agregado entre cuentas.

Las reglas de [coincidencia histórica](RELATIONSHIPS.md#coincidencia-historica) pertenecen a Relaciones; su decisión pendiente [DEC-06](RELATIONSHIPS.md#dec-06) no se resuelve mediante una fórmula estadística.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-08"></a>DEC-08 | Desempate Ducky, umbrales por resumen y tratamiento exacto de archivados donde sólo dice «visibles». | Sin muestra no se fabrica cero; fórmulas y prohibiciones [Estadísticas e IA](STATISTICS.md). |
