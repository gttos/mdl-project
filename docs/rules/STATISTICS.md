# Reglas de producto — Estadísticas

<a id="copy-es"></a><a id="decisiones-pendientes"></a>

[Índice de reglas](../PRODUCT.md) · Dashboard personal de la beta, cálculos, ausencia de datos, métricas excluidas y alcance entre cuentas.

<a id="metricas"></a>
## Métricas y cálculos

- **RP-EST-001.** Las estadísticas son de sólo lectura y se calculan en la API exclusivamente con la estructura perteneciente a la cuenta autenticada. Las aplicaciones presentan el resultado recibido y no recalculan ratios, medias ni distribuciones.
- **RP-EST-002.** La beta ofrece un único dashboard personal calculado sobre todo el historial elegible de la cuenta.
- **RP-EST-015.** El dashboard comprende resumen de tasa de match, satisfacción media, Ducky predominante y situación actual; resultados de match y primer match; momento actual; distribuciones de satisfacción, valoraciones, rasgos físicos, zodíaco, país y Ducky; cualidades y estilo social; y las siete dimensiones de respuestas de experiencias. Cada resultado identifica disponibilidad, valor y muestra utilizada.
- **BR-12.01.** Las valoraciones y la satisfacción utilizan la escala entera de 1 a 5. La API convierte esos IDs a sus valores numéricos para calcular estadísticas.
- **RP-EST-003.** Las medias y porcentajes se calculan con dos decimales. La consulta expresa porcentajes enteros, medias de escala 1–5 con un decimal y conteos enteros, conservando el resultado calculado y su unidad.
- **RP-EST-004.** La tasa de match es cien veces la cantidad de Crushes históricos elegibles con match dividida por la cantidad con resultado resuelto, match o sin match. Cuenta Crushes distintos y excluye los resultados desconocidos del denominador. La consulta informa por separado los tres conteos.
- **RP-EST-005.** Denominador cero implica no disponible, no 0% medido.
- **RP-EST-006.** La satisfacción media utiliza las experiencias elegibles y sus valores de 1 a 5. Sin experiencias elegibles, el resultado es nulo.
- **RP-EST-007.** El Ducky predominante es la única categoría con mayor frecuencia entre los Crushes elegibles. Sin muestra o cuando dos o más categorías empatan en la frecuencia máxima, no existe un Ducky predominante y el resultado es nulo; nunca se escoge uno mediante orden de ID, antigüedad ni otro desempate arbitrario.
- **RP-EST-008.** La situación actual cuenta períodos vivos en situación 8, pareja: cero corresponde a sin pareja, uno a una relación y más de uno a varias relaciones.
- **RP-EST-009.** El momento actual cuenta Crushes con período vivo y los distribuye por su situación vigente.
- **RP-EST-010.** Sin Crushes activos no archivados, el momento actual carece de datos. Con al menos un Crush activo no archivado, pero sin período vivo elegible, el momento actual está disponible y cuenta cero Crushes con período vivo.
- **RP-EST-011.** El dashboard calcula exclusivamente match y primer match del Crush, satisfacción de experiencias, Ducky, valoraciones, rasgos físicos, zodíaco, país, cualidades y estilo social del Crush y respuestas de experiencias. El primer match corresponde exclusivamente a situaciones 5, 6, 7 u 8. Cada opción consultada conserva el significado del identificador de su catálogo.
- **RP-EST-012.** Una métrica sin observaciones conserva su estado de ausencia de datos y su muestra, con valor ausente cuando corresponde. Ese estado se distingue de carga y error.
- **RP-EST-013.** Toda comparación, cobertura o indicación de confiabilidad se basa en valores calculados y publicados por la API. La aplicación no completa valores ausentes ni fabrica explicaciones.
- **RP-EST-014.** La consulta informa de carga y error y permite reintentar, conservando el contenido confirmado conforme a RP-VAC-001.
- **RP-EST-016.** Cada métrica observacional está disponible desde una observación elegible y comunica la cantidad utilizada. Con cero observaciones aplica su estado de ausencia. Los conteos y estados exactos del momento actual mantienen su disponibilidad definida en RP-EST-008..010.
- **RP-EST-017.** Las métricas históricas incluyen Crushes activos y archivados de la cuenta. El momento actual utiliza exclusivamente los activos no archivados. Ambos universos excluyen Crushes eliminados o seudonimizados.
- **RP-EST-018.** Las estadísticas describen valores, conteos, porcentajes, medias, distribuciones, base y respaldo. Su significado se limita a los datos registrados: excluye inferencias personales, diagnósticos, causas o recomendaciones. La descripción combinada de valoraciones aplica las condiciones deterministas del apartado siguiente.
- **RP-EST-019.** Cada media, porcentaje, distribución o categoría predominante informa de observaciones utilizadas, registros elegibles, cobertura y nivel de respaldo. Cobertura = cien veces observaciones utilizadas dividido por registros elegibles, con el redondeo de RP-EST-003. El mínimo de disponibilidad es una observación y permanece separado del denominador. Con cero elegibles, la métrica carece de datos y de nivel de respaldo. Los conteos y estados exactos del momento actual carecen de nivel de respaldo.
- **RP-EST-020.** El respaldo es bajo con menos de diez observaciones utilizadas o cobertura inferior al 50 %. Es alto con al menos treinta observaciones y cobertura de al menos el 80 %. En los demás casos es medio. Los límites utilizan la cobertura exacta; el nivel expresa únicamente la base interna del dato.
- **RP-EST-021.** Cada valoración, rasgo físico y zodíaco calcula su propia muestra, cobertura y respaldo. Un dato opcional ausente queda fuera de su cálculo, conservando al Crush entre los elegibles. Una dimensión sin observaciones tiene valor ausente.
- **RP-EST-022.** Cada métrica informa una vez de su valor, unidad, base y nivel de respaldo. La consulta permite distinguir los tres niveles y comprender que una muestra pequeña conserva un valor real. La satisfacción mantiene su escala 1–5 y los porcentajes su escala 0–100; los conteos conservan su unidad. Cada distribución contiene las categorías observadas y permite consultar sus nombres completos.
- **RP-EST-023.** Cada dimensión física informa de su distribución observada y de todas las categorías empatadas en la frecuencia máxima, con sus nombres, porcentaje, conteo y condición de empate. La identidad de catálogo conserva su significado y ningún orden arbitrario decide un ganador.
- **RP-EST-024.** El dashboard y la demo permiten consultar una ayuda sobre porcentajes, conteos, denominadores, cobertura y respaldo. La ayuda explica cómo se obtiene la muestra y cómo aumenta al completar datos, sin interpretar personalmente los resultados. La distribución de emociones explica su selección múltiple.
- **RP-EST-025.** El dashboard permite consultar las métricas y ayudas definidas en este dominio. Cada acción disponible tiene un resultado funcional.
- **RP-EST-026.** La demo contiene ejemplos completos y coherentes de las estadísticas, con valores, bases y ayudas fijos. Sus resultados permanecen separados de los cálculos y datos reales.
- **RP-EST-027.** Los ejemplos ficticios permanecen separados de las cuentas, diario y estadísticas reales. Su consulta utiliza exclusivamente la medición específica de demo; no constituye acceso, registro o creación del diario real.

<a id="descripcion-valoraciones"></a>
### Descripción combinada de valoraciones — RP-EST-018

**Significado.** El mensaje combina el contexto de la media global con una descripción del conjunto de las cuatro dimensiones. Se utilizan las medias publicadas por API, antes de su formato visual; no se recalculan medias, ratios ni distribuciones ni se completa una dimensión ausente. Destacar relativamente no convierte una puntuación baja en una valoración alta. Las bases siguen siendo independientes conforme a RP-EST-021: las frases describen medias registradas y no afirman que el patrón ocurra en una misma ficha o persona. No se introduce un mínimo de muestra nuevo.

**Selección, en este orden:**

1. Sin media global no aparece una descripción; se conserva el estado sin dato de la tarjeta.
2. Con media global y alguna de las cuatro dimensiones sin valoración, se utiliza el caso «Dimensiones incompletas». Las dimensiones disponibles y sus bases permanecen visibles; la ausencia no se convierte en cero.
3. Con las cuatro dimensiones, se ordenan sus medias de mayor a menor y se comparan los tres saltos consecutivos. Si el mayor salto es **igual o superior a 0,5 puntos**, el grupo situado por encima de ese corte es el destacado. Si varios cortes comparten ese salto máximo, se elige el grupo superior más amplio. Los valores iguales permanecen juntos; los nombres o IDs no desempatan.
4. Si ningún salto alcanza 0,5 y la diferencia entre la mayor y la menor media es **inferior a 0,5**, se utiliza «Cuatro similares».
5. Si ningún salto alcanza 0,5 pero esa diferencia total es **igual o superior a 0,5**, se utiliza «Contrastes sin predominio».

El grupo destacado puede ser cualquiera de las cuatro dimensiones, cualquiera de las seis parejas o cualquiera de los cuatro tríos. No se privilegia una combinación ni se excluye una dimensión empatada con las destacadas. La demo conserva RP-EST-026/027: su mensaje corresponde a sus valores literales y se presenta como contenido fijo, sin calcular estadísticas ni consultar datos reales.

**Significado del resultado.** Con media global disponible, el resultado informa de su contexto y del caso calculado, con la base global y las bases independientes de cada dimensión.

| Media global | Contexto |
|---|---|
| 1 ≤ media ≤ 2 | Bajo |
| 2 < media < 4 | Intermedio |
| 4 ≤ media ≤ 5 | Alto |

Los casos identifican la dimensión, pareja o trío destacado; cuatro dimensiones similares; contrastes sin predominio; o dimensiones incompletas, conforme a las condiciones anteriores.

<a id="adjetivos"></a>
### Cualidades y estilo social por Crush

- **RP-EST-028.** Las cualidades se calculan con la selección actual de cada Crush del historial elegible de RP-EST-017. La unidad es el Crush, independientemente de sus períodos o situaciones relacionales. `eligible` cuenta todos esos Crushes y `used` cuenta los que tienen al menos una cualidad. Cada cualidad cuenta los Crushes que la tienen seleccionada y su porcentaje es `100 × count / used`. Seleccionar varias cualidades conserva una sola observación por Crush para `used`. La API publica el top de RP-EST-030 sobre esa base completa y la tarjeta conserva la cobertura `used de eligible`.
- **RP-EST-029.** El estilo social utiliza el mismo universo de Crushes y una base independiente: `used` cuenta los que tienen estilo informado. Cada estilo observado publica su conteo de Crushes y su porcentaje sobre esa base. Cualidades y estilo social aplican disponibilidad, precisión, cobertura y respaldo de RP-EST-003/005/012/016/019/020/022/024.
- **RP-EST-030.** Cada uno de los cuatro bloques de cualidades publica hasta tres opciones observadas, ordenadas por conteo descendente y, en empate, identificador ascendente. Con menos de tres se utilizan las existentes. Los cuatro bloques comparten la base completa de RP-EST-028 y conservan los porcentajes originales. La consulta informa del bloque, nombre de cada cualidad, porcentaje y cantidad sobre esa base, respetando el orden recibido.

#### Escenarios de cualidades y estilo social

| ID | Dado / Cuando / Entonces | Reglas |
|---|---|---|
| GWT-EST-ADJ-01 | Dados diez Crushes elegibles, cinco con cualidades y tres con «Leal», el resultado de «Leal» tiene conteo 3 y porcentaje 60 %, sobre una base utilizada de cinco y cobertura 5 de 10. | RP-EST-028/030, RP-EST-019 |
| GWT-EST-ADJ-02 | Dado un Crush con varias cualidades y varios períodos, cada cualidad seleccionada suma un Crush y la base utilizada suma uno. Su situación relacional conserva el mismo resultado. | RP-EST-028 |
| GWT-EST-ADJ-03 | Dados diez Crushes, cinco con cualidades y cuatro con estilo informado, las bases utilizadas son cinco y cuatro respectivamente. La cobertura de cada tarjeta usa diez elegibles. | RP-EST-028/029 |
| GWT-EST-ADJ-04 | Dado un universo elegible, se incluyen activos y archivados de la cuenta y se excluyen eliminados y seudonimizados. Editar o vaciar una selección actualiza el siguiente resultado confirmado. | RP-EST-017/028/029 |
| GWT-EST-ADJ-05 | Dada una dimensión sin observaciones, devuelve estado vacío y valor nulo. Con una observación se calcula el resultado y se publica la base y el respaldo correspondientes. | RP-EST-005/012/016/019/020/028/029 |
| GWT-EST-ADJ-06 | Dadas más de tres cualidades observadas en uno o varios bloques, la API publica hasta tres por cada bloque, por conteo descendente y, a igual conteo, ID ascendente. Los bloques conservan el orden del catálogo y la base común y los porcentajes completos. Un bloque con menos de tres publica las existentes; sin observaciones publica una lista vacía. iOS presenta cada top bajo el encabezado de su bloque. | RP-EST-028/030 |
| GWT-EST-ADJ-07 | Dado un conteo positivo cuyo porcentaje se redondea a cero, API publica el conteo y el porcentaje redondeado, y el cliente conserva ambos en una sección disponible. Se aplica a cualidades y estilo social. | RP-EST-003/028/029 |

## Respuestas de experiencias

<a id="experiencias"></a>

- **RP-EST-031.** El dashboard incorpora siete distribuciones independientes: emociones, conexión, escucha, deseo posterior, autenticidad, seguridad y expresión de límites, usando los catálogos de [RP-EXP-029](EXPERIENCES.md#niveles). La unidad es una experiencia, incluida la inicial, no un Crush, período, hito ni formulario abierto. Cada respuesta confirmada vigente cuenta una vez. No se mezclan estas dimensiones con las valoraciones del perfil del Crush.
- **RP-EST-032.** `eligible` utiliza el mismo universo histórico de experiencias que satisfacción: experiencias vivas del propietario cuyo Crush comparte propietario, existe y no está eliminado ni seudonimizado; los archivados permanecen incluidos. Cada dimensión calcula su `used` por separado: respuestas simples no nulas y conjuntos de emociones no vacíos. No responder no equivale a cero ni a una opción negativa. Las opciones expresas «No lo sé», «No aplica», «No surgió la ocasión» y «Otra» conservan categoría y cuentan como respuestas registradas.
- **RP-EST-033.** Cada opción observada publica su ID, conteo de experiencias y `percentage = 100 × count / used`. En emociones, cada experiencia cuenta una vez en `used` y una vez en cada emoción elegida: los porcentajes pueden sumar más de 100 % y la interfaz lo aclara. Nunca se divide por la cantidad total de etiquetas. Se muestran todas las categorías observadas, ordenadas por conteo descendente y, a igual conteo, ID ascendente, sin inventar un ganador.
- **RP-EST-034.** Cada distribución de respuestas conserva su disponibilidad y muestra independiente. Con cero respuestas tiene valor ausente; desde una respuesta comunica resultado y respaldo. Los identificadores de categorías se cuentan, sin promediarse ni formar puntuaciones compuestas o inferencias personales.
- **RP-EST-035.** Las siete distribuciones de respuestas de experiencias informan del significado de su dimensión, valores y bases en experiencias. Editar o retirar una respuesta actualiza el siguiente resultado confirmado; las respuestas guardadas cuentan aunque un nivel de entrada las oculte. Descripción y reflexión quedan excluidas del cálculo. Los ejemplos ficticios conservan valores coherentes separados del diario.

<a id="metricas-excluidas"></a>
## Métricas excluidas

- **BR-12.02.** El dashboard, la API de estadísticas y sus presentaciones excluyen rankings competitivos de Crushes o cuentas, body count, predicciones de fracaso de un vínculo activo, percentiles de superioridad, valor de mercado y seguimiento individual de revisitas o ghosting.

<a id="entre-cuentas"></a>
## Alcance entre cuentas

- **RP-FUT-002.** Las estadísticas de la beta son personales: cada dashboard utiliza únicamente datos de su cuenta propietaria y no combina datos de distintas cuentas.

Las proyecciones de match y primer match pertenecen al [Crush](CRUSH.md#match). Estadísticas las consume como estructura ya resuelta y no reconstruye la historia de períodos e hitos.

### Escenarios de aceptación de estadísticas de demo

| ID | Dado / Cuando / Entonces | Regla |
|---|---|---|
| GWT-ODM-S07 | Dadas las estadísticas, cuando se recorre toda la vista, entonces están todas las secciones del dashboard real con gráficos, valores y bases literales coherentes, sin calcular ni consultar datos. No se cuenta como uso real ni altera métricas configuradas. | RP-EST-026/027, RP-ANA-022/026/028 |

<a id="escenarios-experiencias"></a>
## Escenarios de estadísticas de experiencias y presentación

| ID | Dado / Cuando / Entonces | Regla |
|---|---|---|
| GWT-EXP-D12 | Dado el borrado individual de Experience/Crush o período elegible, al retirarlo sus respuestas dejan de contribuir; archivarlo conserva la historia elegible. | RP-EXP-019..024, RP-EST-032 |
| GWT-EXP-D13 | Dadas 4 experiencias elegibles, 2 con emociones y ambas con Ilusión (una también con Nervios), al calcular se obtiene Ilusión 100 % (2 de 2), Nervios 50 % (1 de 2), cobertura 2 de 4, sin dividir por tres etiquetas. | RP-EST-032/033 |
| GWT-EXP-D14 | Dadas dimensiones vacías, respuestas expresas desconocidas y muestras distintas, al calcular cada distribución conserva sus categorías, denominador, cobertura y respaldo; no promedia IDs ni inventa ceros. | RP-EST-032..034 |
| GWT-EXP-D15 | Dado el dashboard y la demo ES/EN, al leer se ven las siete distribuciones con bases en experiencias; la demo usa literales coherentes y no genera datos reales. | RP-EST-035, RP-DEMO-001..014 |
| GWT-EXP-D17 | La consulta conserva categorías, unidades, bases y todos los empates de cada dimensión. | RP-EST-022/023/030/035 |
| GWT-EXP-D18 | Con cuatro experiencias elegibles y dos respuestas explícitas de desconocimiento para seguridad, esa categoría informa 100 %, cantidad dos de dos y cobertura dos de cuatro. El respaldo conserva su significado. | RP-EST-019/020/022/032/033 |
| GWT-EXP-D19 | Dado el dashboard o su demo ES/EN, al leer momento actual y primer match las situaciones conservan sus IDs y etiquetas localizadas. El español coincide con la tabla relacional; otros flujos conservan su copy vigente. | RP-SIT-004, RP-EST-009/011 |
| GWT-EXP-D20 | Dadas la media global y las dimensiones publicadas, al presentar la descripción ES/EN se seleccionan el contexto y el caso exactos de la tabla de RP-EST-018. Sin media global no aparece descripción; no se recalculan métricas ni se inventan ceros o conclusiones personales. | RP-EST-003/018/021 |
| GWT-EXP-D21 | Win rate conserva su escala porcentual y satisfacción su escala de uno a cinco; la ausencia de valor se distingue de cero y la ayuda explica las bases. | RP-EST-005/022/024 |
| GWT-EXP-D22 | Las estadísticas consultadas desde Home conservan sus valores, unidades y navegación. | RP-EST-022 |
| GWT-EXP-D23 | Dadas cuatro medias y un salto suficiente que separa una dimensión, una pareja o un trío, el mensaje nombra exactamente el grupo definido por RP-EST-018. Se cubren las 14 combinaciones, sin depender del orden de los campos ni omitir dimensiones empatadas. | RP-EST-018 |
| GWT-EXP-D24 | Dados saltos máximos iguales en 5 · 4 · 3 · 2, se elige el grupo superior de tres. Dados 4 · 4 · 2 · 2, los dos valores empatados superiores permanecen juntos. No hay desempate por nombre o ID. | RP-EST-018 |
| GWT-EXP-D25 | Dadas 4 · 4 · 3,5 · 3,5, el salto exacto de 0,5 selecciona las dos superiores. Dados 4 · 3,51 · 3,51 · 3,51, la diferencia total inferior a 0,5 produce «Cuatro similares». | RP-EST-018 |
| GWT-EXP-D26 | Dadas 4 · 3,6 · 3,2 · 2,8, ningún salto llega a 0,5 y la diferencia total supera ese límite: se presenta «Contrastes sin predominio». Dadas 4 · 3,8 · 3,6 · 3,5, la diferencia total exacta de 0,5 produce el mismo caso. | RP-EST-018 |
| GWT-EXP-D27 | Dadas medias globales publicadas de 2, 2,01, 3,99 y 4, los contextos son bajo, intermedio, intermedio y alto. Se conservan los valores y sus bases; destacar dentro de un conjunto bajo no se describe como una puntuación alta. | RP-EST-003/018/021 |
| GWT-EXP-D28 | Dada una media global disponible y cualquier dimensión ausente, aparece el mensaje de dimensiones incompletas; se conservan las métricas disponibles y sus bases independientes. Sin global no aparece descripción. No hay cero sustitutivo ni mínimo de muestra nuevo. | RP-EST-016/018/021 |
| GWT-EXP-D29 | Consultar valoraciones permite identificar el contexto global, el resultado relativo y las bases independientes una vez por métrica. | RP-EST-018/022/026/027 |
