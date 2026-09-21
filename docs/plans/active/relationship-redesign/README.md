# Rediseño de relaciones

> **Estado:** contrato funcional redefinido; implementación API/iOS pendiente de sustitución limpia. No hay despliegue, migración de datos ni activación autorizados.

> **Autoridad:** [Relaciones](../../../rules/RELATIONSHIPS.md), [Experiencias](../../../rules/EXPERIENCES.md), [Crush](../../../rules/CRUSH.md), [Timeline](../../../rules/TIMELINE.md) y [Estadísticas](../../../rules/STATISTICS.md).

> **Proyectos:** API e iOS. Cuenta no cambia. Crush participa en el bloque inicial y conserva las proyecciones de match.

## Objetivo

Sustituir el modelo dividido en fase y etiqueta por una única situación relacional exacta, comprensible y consultable; exigir una Experience completa en todo hito; guardar match y primer match como proyecciones directas; y hacer que creación, corrección y borrado mantengan una historia coherente sin huérfanos.

## Decisiones cerradas

- `RelationshipSituation` es un Value Object cerrado en API y un enum equivalente en iOS. Sus claves internas están en inglés; la app presenta copy en español de España.
- La base guarda únicamente IDs enteros. No existe tabla de catálogo de situaciones ni de match.
- Las diez situaciones son `NOT_TALKING`, `ACQUAINTED`, `FRIENDS`, `MUTUAL_INTEREST`, `DATING`, `CASUAL`, `FWB`, `RELATIONSHIP`, `UNDEFINED` y `CLOSED` con los IDs fijados en Product. `FRIENDS` representa «Solo somos amigos» y no establece match.
- `MatchStatus` usa `0 UNKNOWN`, `1 MATCH` y `2 NO_MATCH`. Sólo el Crush conserva `match_status_id`, `first_match_situation_id` y `first_match_date`; los hitos son su evidencia estructural.
- No existen fase «Involucrado», etiqueta separada, `everMatched` ni cálculo estadístico que tenga que recorrer toda la historia para conocer el resultado del Crush.
- El esquema físico objetivo usa `relationship_periods`, `relationship_milestones` y `experiences`.
- Todo hito tiene una Experience obligatoria. Una Experience puede ser independiente del hito, pero nunca del período.
- El alta crea Crush, período inicial, hito inicial y Experience inicial completa. La situación inicial admite 1–9; `CLOSED` exige un hito posterior.
- «Añadir experiencia» empieza por la Experience. Con un período vivo es un paso único y ofrece «¿Hay un cambio de situación?» para añadir un segundo paso opcional; con el período más reciente cerrado, el segundo paso de situación inicial es obligatorio y la creación del período nuevo permanece transparente.
- «Cambiar situación» empieza por situación/cierre y exige después la Experience que explica qué ocurrió. El guardado es una única intención durable y atómica.
- Sólo el último hito del período más reciente puede corregirse o eliminarse; las Experiences independientes posteriores no cambian esa condición. «Eliminar período» sólo se habilita cuando el hito inicial del período no inicial más reciente sigue siendo también su último hito y puede iniciarse desde ese hito o desde su Experience vinculada. La operación elimina todas las Experiences del período, avisa antes de confirmar y mantiene cerrado el período anterior.
- No hay compatibilidad, migración, backfill, aliases ni rutas legacy: no existen usuarios ni datos de producción anteriores que conservar.

## Alcance

- Dominio, esquema, contratos HTTP, validación, locking, idempotencia y proyecciones de API.
- Modelos, copy `es-ES`, formularios, snapshots durables, Timeline, Home, perfiles, listados y Statistics de iOS.
- Alta inicial coordinada con `crush-rule-compliance`.
- Creación de Experiences independientes, cambios de situación, cierre y apertura de períodos posteriores.
- Corrección y borrado latest-only, recálculo transaccional de match y retirada de contenido Vault afectado.
- Eliminación del código, schema, fixtures y pruebas del modelo anterior.

Fuera de alcance:

- migrar o conservar relaciones de usuarios anteriores;
- tablas administrables o catálogos remotos para situaciones;
- localización de este flujo a idiomas distintos de `es-ES` en la beta;
- emitir conclusiones jurídicas o textos públicos sin revisión profesional;
- ejecutar migraciones, deploys o suites desde esta tarea coordinadora.

## Contrato compartido

### Persistencia objetivo

| Entidad | Campos funcionales principales |
|---|---|
| `relationship_periods` | `id`, `user_id`, `crush_id`, `start_date`, `end_date`, `current_situation_id`, `correlation_id` y ciclo de vida técnico. |
| `relationship_milestones` | `id`, `relationship_period_id`, `experience_id` obligatorio y único, `sequence`, `effective_date`, `previous_situation_id`, `situation_id`, `closure_reason_id`, `correlation_id` y ciclo de vida técnico. |
| `experiences` | `id`, `user_id`, `crush_id`, `relationship_period_id`, `date`, `satisfaction_id`, `correlation_id` y ciclo de vida técnico. Lugar y descripción permanecen en el payload Vault. |
| `crushes` | `match_status_id`, `first_match_situation_id` y `first_match_date` como proyecciones recalculables. |

La API puede publicar `relationship_milestone_id` dentro de una Experience como relación derivada desde `relationship_milestones.experience_id`; no se duplica el vínculo físico.

### Escrituras autorizadas

1. Alta inicial: Crush + período + hito + Experience.
2. Experience independiente dentro de un período vivo.
3. Cambio de situación: Experience + hito.
4. Primera Experience después de un cierre: Experience + período nuevo + hito inicial.
5. Corrección del último hito sobre la misma identidad, fecha y Experience.
6. Eliminación del último hito conservando la Experience, o del hito con su Experience.
7. Eliminación completa del período no inicial más reciente mediante «Eliminar período», iniciada desde su hito inicial o desde la Experience vinculada, únicamente cuando no exista otro hito posterior; incluye todas sus Experiences independientes.

No quedan endpoints de creación o cambio relacional capaces de confirmar un período o hito sin Experience.

## Escenarios de aceptación

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-RR-01 | Un alta válida | Se confirma | Se crean atómicamente Crush, período inicial, hito inicial y Experience completa en `met_at`. |
| GWT-RR-02 | La app presenta una situación | Renderiza su ID | Usa el copy `es-ES` canónico; las claves inglesas no se muestran ni se traducen en persistencia. |
| GWT-RR-03 | Cualquier escritura | Recibe una situación | API valida el ID mediante `RelationshipSituation`; no consulta una tabla de catálogo. |
| GWT-RR-04 | Un período vivo | Se usa «Añadir experiencia» y no se activa «¿Hay un cambio de situación?» | El formulario termina tras la Experience y ésta queda dentro del período sin hito. |
| GWT-RR-05 | Un período vivo | Se usa «Añadir experiencia» y se activa «¿Hay un cambio de situación?» | Después de completar la Experience aparece el paso de situación; Experience e hito se confirman juntos o no se confirma ninguna parte. |
| GWT-RR-06 | El último período está cerrado | Se usa «Añadir experiencia» | No aparece una decisión opcional: se completa primero la Experience y después una situación inicial 1–9; Experience, período e hito inicial se crean juntos. |
| GWT-RR-07 | Una historia entra por primera vez en una situación 5–9 | Se confirma el hito | El Crush guarda `MATCH` y el primer match correspondiente. Un período posterior conserva ese resultado del Crush. |
| GWT-RR-08 | El Crush no llegó a 5–9 y su período más reciente cierra | Se confirma el cierre | El Crush queda `NO_MATCH`. Si después se abre un período nuevo sin match, vuelve a `UNKNOWN`. |
| GWT-RR-09 | Se corrige o elimina la única evidencia de match | La API confirma la operación | Match y primer match del Crush se recalculan en la misma transacción desde los hitos que permanecen. |
| GWT-RR-10 | Existe un hito posterior elegible | Se elige «Eliminar cambio» | Se elimina el hito y su Experience queda independiente en el mismo período. |
| GWT-RR-11 | Existe un hito posterior elegible | Se elige «Eliminar cambio y experiencia» o se borra desde la Experience | Se eliminan ambos atómicamente y se restaura la historia anterior. |
| GWT-RR-12 | El hito inicial del período no inicial más reciente sigue siendo también su último hito, aunque existan Experiences independientes posteriores | Se usa «Eliminar período» desde el hito o se borra su Experience vinculada y se confirma la advertencia | Se eliminan atómicamente el hito raíz, su Experience, el período y todas sus Experiences independientes; el período anterior sigue cerrado y pasa a ser el más reciente. |
| GWT-RR-13 | Timeline presenta un hito | Construye el acontecimiento | Muestra hito y Experience como una sola tarjeta lógica y usa las capacidades publicadas por API. |
| GWT-RR-14 | Statistics calcula win rate | Agrupa Crushes | Divide `MATCH` por `MATCH + NO_MATCH`, excluye `UNKNOWN` e informa los tres conteos. |
| GWT-RR-15 | Dos acciones compiten sobre el mismo Crush | Se ejecutan realmente en paralelo | Sólo puede resultar una historia válida, sin duplicados, huérfanos ni proyecciones divergentes. |
| GWT-RR-16 | Código o payload usa fase, etiqueta, `everMatched` o tablas anteriores | Se valida el corte final | No existe consumidor, adaptador, alias ni contrato runtime que lo acepte. |
| GWT-RR-17 | La persona inicia directamente «Cambiar situación» | Completa el formulario | Elige primero la situación y después la Experience; el resultado estructural se confirma como un único compuesto. |
| GWT-RR-18 | El hito inicial de un período posterior tiene otro hito después | Se intenta eliminar el período desde su raíz | La capacidad no se publica y una petición desactualizada se rechaza completa, aunque la raíz sea el primer acontecimiento del período. |
| GWT-RR-19 | El último hito elegible tiene Experiences independientes con fecha posterior | Se corrige o elimina | Las Experiences no lo vuelven histórico: latest-only compara hitos y la acción conserva o elimina únicamente lo previsto por el recorrido elegido. |
| GWT-RR-20 | Una Experience independiente tiene una fecha válida dentro de su período | Se mueve atravesando uno o más hitos | Conserva período e identidad y adopta el contexto relacional de su nueva fecha; se rechaza únicamente si el día ya está ocupado, incluido el día de un hito. |
| GWT-RR-21 | El período más reciente está cerrado | La persona consulta las acciones de alta | «Añadir experiencia» y «Cambiar situación» siguen disponibles; ambas crean Experience, período e hito inicial, pero respetan su orden de pasos propio. |
| GWT-RR-22 | La persona inicia «Cambiar situación» y elige `CLOSED` | Completa el formulario | El primer paso exige el motivo catalogado y el segundo, la Experience completa cuya descripción conserva el relato privado del cierre; ambos se guardan en una sola acción. |
| GWT-RR-23 | La situación vigente es `ACQUAINTED` | Se registra `FRIENDS` | El hito conserva el ID 3, la app muestra «Solo somos amigos» y el Crush continúa sin evidencia de match. Desde `FRIENDS` sólo se puede avanzar a `MUTUAL_INTEREST` o `CLOSED`. |
| GWT-RR-24 | Dos Crushes distintos pertenecientes a la misma cuenta ya tienen o reciben una Experience en el día civil `D` | Se validan o confirman ambas | Las dos Experiences son válidas y se conservan. La unicidad diaria utiliza la combinación (`crush_id`, `date`) y sólo rechaza una segunda Experience del mismo Crush en `D`. |

## Interacción entre proyectos

| Superficie | API | iOS |
|---|---|---|
| Catálogo | Value Objects con IDs y semántica de match/transición. | Enums con los mismos IDs y copy `es-ES`. |
| Escrituras | Autoridad de validación, locking, transacción e idempotencia. | Captura completa y snapshot durable con IDs estables. |
| Vault | Binding y retirada de ciphertext de Experiences sin interpretar contenido. | Cifra/descifra lugar y descripción de cada Experience. |
| Lecturas | Publica situación, match, primer match y capacidades. | Presenta y filtra sin reconstruir historia ni latest-only. |
| Estadísticas | Calcula dashboard desde proyecciones guardadas. | Renderiza respuesta sin recalcular métricas. |

La obligatoriedad del bloque inicial se coordina con `API-CRU-RED-03/03` e `IOS-CRU-02`; este plan no duplica su propiedad, pero no puede cerrar sin esa integración.

## Fases coordinadas

| Orden | Fase | Resultado exigido | Estado |
|---:|---|---|---|
| 1 | `API-RR-N1` | Esquema final, Value Objects y dominio sin fase/etiqueta ni compatibilidad. | Pendiente |
| 2 | `API-RR-N2` | Escrituras compuestas, trayectos, cierre, locking e idempotencia. | Pendiente |
| 3 | `API-RR-N3` | Corrección, borrado latest-only, Vault y recálculo de proyecciones. | Pendiente |
| 4 | `API-RR-N4` | Lecturas, capacidades, Timeline, Home y Statistics finales. | Pendiente |
| 5 | `API-RR-N5` | Retirada completa del modelo anterior y validación integrada/concurrente. | Pendiente |
| 6 | `IOS-RR-N1` | Modelo nativo, transporte y copy `es-ES` por ID. | Pendiente |
| 7 | `IOS-RR-N2` | Formularios compuestos y snapshots durables, incluido el bloque inicial coordinado. | Pendiente |
| 8 | `IOS-RR-N3` | Timeline, Home, listas, perfil y Statistics con el contrato final. | Pendiente |
| 9 | `IOS-RR-N4` | Corrección y borrado según capacidades API, con retry durable. | Pendiente |
| 10 | `IOS-RR-N5` | Validación física, accesibilidad, offline/retry e integración API+iOS. | Pendiente |

Cada sesión implementadora ejecuta una sola fase local. El plan de API y el de iOS contienen su entrada, pruebas y salida.

## Gates de salida

- No queda símbolo, campo, ruta, fixture o prueba runtime basado en fase, etiqueta, `RelationshipState`, `everMatched` o nombres físicos anteriores.
- No existe tabla de catálogo para `RelationshipSituation` o `MatchStatus`; sus IDs coinciden en API e iOS.
- Todos los hitos tienen una Experience y no existe Experience sin período.
- Las carreras latest-only y de unicidad se prueban sobre MySQL/InnoDB real.
- Los reintentos iOS reutilizan IDs y snapshot; no reconstruyen una intención diferente.
- El copy visible de relaciones está revisado en español de España y las claves inglesas nunca aparecen en pantalla.
- API+iOS validan altas, cambios, cierre, reapertura, corrección, los dos borrados de hito y borrado de período.
- `DEC-13` recibe validación jurídica antes del lanzamiento; el expediente factual ya está alineado con este contrato.
- El contrato, los snapshots y la interfaz no contienen `closure_note`; el cierre conserva motivo estructurado y Experience completa.

## Bloqueos y decisiones abiertas

- **Legal:** `DEC-13` permanece abierto y bloquea lanzamiento, no implementación. El expediente factual ya está actualizado; falta el criterio profesional sobre inferencias, minimización, terceras personas y derechos.
- No quedan decisiones funcionales abiertas sobre match, claves internas, idioma visible, almacenamiento ni compatibilidad histórica.
