# Rediseño de relaciones

> **Contrato funcional actualizado:** [Relaciones](../../../rules/RELATIONSHIPS.md) es la autoridad única. La concreción de trayectos lógicos en `RP-TRA-002..006` sustituye el permiso anterior de saltar fases. Las aceptaciones técnicas descritas abajo corresponden a su contrato de entrada y no demuestran todavía esa regla nueva. Antes de continuar una fase dependiente se deben alinear su alcance local y escenarios, sin implementar desde este relevamiento.

> **Estado:** replanteo transversal activo. El corte greenfield, lifecycle, lecturas nativas, concurrencia y contratos HTTP de API están aceptados y recongelados; `API-RR-02G` también queda aceptada y congelada tras validar en MySQL 8 el cambio atómico de período vinculado a Experience y su paginación lógica. iOS ya tiene dominio, transporte, durable write, altas, lectura confirmada, avance/cierre/corrección standalone y el cambio atómico vinculado desde Experience aceptados. La validación física queda agrupada en `IOS-RR-06`; no hay despliegue, migración de datos ni activación para personas reales autorizados.

> **Proyectos:** API e iOS.

## Premisa de lanzamiento

Esta iniciativa se diseña como si el producto fuera a publicarse con **cero usuarios y cero relaciones preexistentes**:

- toda cuenta creada desde el lanzamiento usa únicamente `periods`;
- no existe selección de versión, bifurcación por cuenta, pantalla legacy ni fallback que interprete filas antiguas durante el uso normal;
- la base histórica actual contiene un único usuario y queda fuera del runtime y de la compatibilidad normal; su inventario sólo entra en la fase aislada `RR-MIG-01`;
- el inventario agregado ya fue entregado y permite diseñar la conversión, pero la base/backup real se revisan por separado antes de ejecutar;
- esa operación futura exigirá autorización explícita, backup, dry-run y validación propios. Puede aceptar pérdida histórica si el usuario la aprueba entonces, pero nunca condiciona ni contamina el runtime nuevo.

Las columnas físicas heredadas pueden permanecer temporalmente inertes si quitarlas exige una operación de datos. Permanecer en el schema no autoriza a leerlas, escribirlas, publicarlas ni usarlas como fallback.

## Objetivo

Separar la etapa de conexión, la etiqueta del vínculo y el motivo de cierre. El modelo plano `RelationshipState` mezcla dimensiones incompatibles y no permite representar con precisión progresión, cierre, reapertura, historia de etiquetas ni métricas.

## Alcance

- Un único modelo `periods` en registro, dominio API, contratos HTTP, iOS, Vault, Timeline, Home, perfiles, listados y Statistics.
- Alta inicial de Crush con período, períodos standalone y flujos compuestos con Experience.
- Avance, etiquetado, cierre, nuevo período posterior y edición histórica limitada.
- Cambio standalone del período y cambio opcional vinculado a una Experience como un único hecho atómico y correlacionado.
- Historial de cambios de etiqueta con creación append-only y correctivo latest-only explícito.
- Proyecciones actuales de fase/etiqueta y `match_status` derivado desde períodos.
- Reintentos durables, correcciones y eliminación/anonymización del grafo completo sin vocabulario legacy.
- Contratos y pruebas integradas API+iOS antes de cualquier activación.

Fuera de alcance:

- ejecutar ahora la conversión de la base histórica actual; su contrato, inventario y dry-run sí se preparan dentro de esta iniciativa;
- dual-running, selección de modelo por cuenta, UI duplicada o compatibilidad funcional con `RelationshipState`;
- activar el modelo para personas reales antes de completar los gates técnicos y la revisión legal;
- cambiar la regla vigente de beta: una experiencia total por crush y día civil;
- inferir “experiencia íntima” o agregar ese dato;
- ejecutar migraciones, backfills, deploys o cambios sobre datos reales desde el plan.

## Referencia al contrato de producto

El catálogo completo vive en [Relaciones](../../../rules/RELATIONSHIPS.md); no se conserva aquí una segunda lista de reglas.

| Tema | IDs canónicos |
|---|---|
| Períodos, comienzo y cronología | `RP-REL-001..007` |
| Match nativo | `BR-7.01`, `RP-MAT-001..003` |
| Nueve situaciones y trayectos lógicos | `RP-TRA-001..008` |
| Once motivos, validación de etiqueta y nota privada | `RP-CIE-001..007` |
| Registro, corrección y borrado latest-only | `RP-HIT-001..013` |
| Experience vinculada, auto-link de creación y cambio explícito | `RP-EXP-004..017`, `RP-EXP-020..021` |
| Evidencia inicial y proyección | `RP-TIM-001..019` |
| Statistics beta | `RP-EST-001..014` |

La decisión abierta sobre corregir/eliminar la única evidencia de match se mantiene en `DEC-06`; no se resuelve por la etiqueta ni por compatibilidad histórica. La revisión Legal sigue independiente (`DEC-13`).

## Contrato aditivo tras cierre

Cuando corresponde GWT-RR-06:

```json
{
  "data": {
    "id": "…",
    "warnings": [
      {
        "code": "experience_after_closed_period",
        "closedPeriodId": "…",
        "availableActions": ["create_new_period"]
      }
    ]
  }
}
```

`create_new_period` es una acción ofrecida al cliente, no una mutación automática. Las respuestas ordinarias mantienen su forma y no inventan la advertencia.

## Mapa de impacto obligatorio

| Superficie | Contrato final | Riesgo que debe retirarse |
|---|---|---|
| Registro e identidad | Toda alta email/Apple recibe fecha de corte estable y usa `periods` implícitamente | `relationship_model_version`, defaults `legacy` o gates por cuenta |
| Crush | Alta crea período inicial; perfil/lista muestran proyección nativa actual y `match_status` derivado | `current_state_id`, denormalización o filtros basados en estado viejo |
| Period | Crear, avanzar, etiquetar, cerrar y editar historia limitada con validación server-side; el cambio puede ser standalone o parte de una Experience atómica | `RelationshipEntity`, matriz de ocho estados o transiciones cliente-autoritativas |
| Experience | Crear, adjuntar/desadjuntar, mover fecha y asociar opcionalmente un cambio real conserva unicidad, identidad de hito, correlación y atomicidad nativas | asociación inferida sólo por fecha, doble vínculo a fase/etiqueta o fallback legacy |
| Timeline | Bundle, Experience, períodos y cambios de fase/etiqueta se proyectan sin `state_id`/`previous_state_id`; un compuesto se presenta como un único hito correlacionado | duplicar visualmente fase y etiqueta del mismo guardado o calcular historia con `RelationshipState` |
| Home y actividad reciente | Los resúmenes decodifican sólo identidad y campos nativos necesarios | `RecentActivityResource.state_id` y modelos iOS equivalentes |
| Vault y durable write | Snapshot canónico conserva la intención standalone/creación/cambio vinculado, IDs, fase/etiqueta/cierre/fecha e identidad de hito en cada retry/corrección | snapshots ambiguos, reintentos que reconstruyen estado o normalizadores con `stateId` |
| Statistics | Dashboard beta nativo sobre datos estructurales; los otros 13 endpoints permanecen vacíos y fuera de beta | assemblers y filtros de ocho estados ejecutables desde endpoints públicos |
| Eliminación y baja de cuenta | Se elimina/desvincula el grafo `periods` y sus blobs de forma atómica | checks o fallbacks a `RelationshipRepositoryInterface` |
| Tests/fixtures | Sólo vocabulario y rutas finales, con idempotencia, rollback y concurrencia | pruebas cuyo criterio sea preservar legacy o aceptar ambos contratos |
| Datos históricos | Operación puntual con backup, dry-run, reglas de anomalía y asociación sólo a hitos reales | confiar en `end_date`, fabricar cambios para conservar links o añadir compatibilidad runtime preventiva |

## Escenarios de aceptación

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-RR-01 | Una cuenta recién creada | Crea su primer Crush | Se crea un período inicial nativo y no se consulta ninguna versión de relación. |
| GWT-RR-02 | Un período vivo | Avanza de fase | Ofrece y valida sólo los destinos de [Relaciones](../../../rules/RELATIONSHIPS.md#trayectos); no permite saltar de «Todavía no hablamos» a un vínculo ni fabrica eventos intermedios. |
| GWT-RR-03 | Un período Involucrado | Cambia su etiqueta | Conserva el período y añade un cambio de etiqueta inmutable visible en Timeline. |
| GWT-RR-04 | Un período vivo | Se cierra | Exige motivo/fecha válidos, queda terminal y una reapertura funcional crea otro período. |
| GWT-RR-05 | El último hito confirmado cerró el período | Se corrige ese hito | Se actualizan sólo situación/motivo permitidos sobre la misma identidad; la fecha no cambia y no aparece otro hito. |
| GWT-RR-06 | Existe un período cerrado | Se registra una Experience posterior | Se acepta con warning cuando corresponde y no se reabre ni crea un período por efecto lateral. |
| GWT-RR-07 | Se usa cualquier ruta standalone o compuesta | Llega `state_id` | Se rechaza sin mutación parcial; no existe una pantalla o traducción legacy. |
| GWT-RR-08 | Home, perfil, lista o Timeline leen una relación | Construyen su respuesta | Exponen sólo el contrato nativo y `match_status` derivado; no publican estados anteriores legacy. |
| GWT-RR-09 | Una API rechaza de forma determinista una escritura ya persistida localmente | La persona vuelve a abrir su estado pendiente, incluso tras reiniciar iOS | Se conserva una única operación y proyección, sin reintento automático. Puede corregir el snapshot y guardar una sustitución deliberada, o confirmar «Eliminar cambio» sólo si la API nunca aceptó la escritura; ese descarte elimina únicamente la operación local, no borra ni modifica datos remotos. |
| GWT-RR-10 | Se elimina una Experience, un Crush o una cuenta | Se procesa el grafo | Aplica el alcance específico de [eliminar una experiencia](../../../rules/EXPERIENCES.md#eliminar), [eliminar un Crush](../../../rules/CRUSH.md#archivo-eliminacion) o [baja de cuenta](../../../rules/DATA_LIFECYCLE.md#baja): la baja conserva estructura seudonimizada y destruye contenido privado, sin reutilizar la cascada manual ni fallback legacy. |
| GWT-RR-11 | iOS abre Statistics en beta | Consulta `GET /statistics/dashboard` | Recibe el dashboard nativo; `actualStatus` y `moment.byPhase` derivan de períodos y no aparece `RelationshipState`. Los otros 13 endpoints no se consumen y permanecen vacíos. |
| GWT-RR-12 | Dos comandos compiten sobre el mismo Crush | La base los ejecuta realmente en paralelo | El historial final es único y válido, sin períodos activos incompatibles, duplicados ni mutación parcial. |
| GWT-RR-13 | Existe un período vivo | Se guarda una Experience con avance, cambio de etiqueta o cierre | Experience, período, historia y vínculo al hito se confirman atómicamente con el día civil de la Experience. |
| GWT-RR-14 | Existe un período vivo | Se avanza, etiqueta o cierra sin Experience | La actualización standalone conserva exactamente las mismas reglas y crea historia sin vínculo a Experience. |
| GWT-RR-15 | La intención vinculada retrocede, no cambia nada, ocupa un día ya usado o intenta insertar antes del último hito | Se guarda | Se rechaza el compuesto completo sin Experience, cambio ni filas de historia parciales; la persona puede quitar la intención relacional y guardar la Experience sola. |
| GWT-RR-16 | Una Experience está vinculada a un cambio confirmado | Se elimina o corrige la Experience | Su asociación se elimina o actualiza según corresponda, pero el cambio y la historia del período permanecen intactos. |
| GWT-RR-17 | Un guardado vinculado queda pendiente o se reintenta | iOS reanuda la operación | Reutiliza Experience, período e identidad de hito y no duplica ninguna fila, vínculo ni item de Timeline. |
| GWT-RR-18 | Un estado legacy posterior tiene Experience | El dry-run lo convierte | Sólo un avance, etiqueta o cierre real conserva el vínculo al hito; una repetición sin cambio queda standalone y una regresión se detiene para decisión manual. |
| GWT-RR-19 | Se crea o edita un período en iOS | La persona elige su situación | Ve una única lista humana de nueve estados; nunca selecciona la fase técnica `INVOLVED` ni una etiqueta en un segundo control. |
| GWT-RR-20 | Un período confirmado en Involucrado se cierra con «Terminó lo que había» | iOS envía y la API valida el cambio | Se persiste el `ENDED_*` correspondiente a la etiqueta confirmada; una variante distinta, una etiqueta ausente o un alta que nace cerrada se rechazan sin mutación. |
| GWT-RR-21 | Un período se cierra | Se elige quién rechazó o cómo terminó | El motivo conserva actor y tipo mediante el catálogo final de once valores; ningún código anterior o alias de compatibilidad se acepta o vuelve a emitirse. |
| GWT-RR-22 | Un período sólo conserva su evento inicial | Se elige «Editar relación» | Se corrigen la evidencia inicial y el snapshot vigente en la misma tarjeta; no se crea un `phase_change` y la fecha continúa siendo `crush.met_at`. |
| GWT-RR-23 | Un período tiene uno o más hitos posteriores | Se intenta editar | Sólo el último ofrece la acción; sus opciones parten de la situación anterior y la API rechaza cualquier hito intermedio sin mutación. |
| GWT-RR-24 | El último hito posterior se corrige | Se guarda otra situación válida | Conserva la misma identidad, fecha y vínculo opcional a Experience, actualiza el snapshot vigente y Timeline muestra una sola tarjeta corregida. |
| GWT-RR-25 | El último hito posterior se elimina | Se confirma «Eliminar cambio» | Se elimina únicamente ese hito, se restaura el snapshot desde el anterior y éste pasa a ser editable; una Experience vinculada se conserva como standalone. |
| GWT-RR-26 | Se intenta eliminar el evento inicial o corregir/eliminar un hito que dejó de ser latest por una carrera | La API vuelve a validar bajo lock | Rechaza de forma estable y sin mutación parcial, audit de éxito ni cambio de vínculo. |

## Evaluación vigente del código

### API

- Implementado y previamente probado: entidad/valores de período, validaciones, persistencia aditiva, historial de etiquetas, warning tras cierre, Statistics vacío y tres flujos compuestos.
- Implementado: retiro del flag/resolver y de las rutas `upgrade`/bundle sin consumidor.
- El corte S2A aceptado retiró las decisiones de runtime por versión y los fallbacks de lifecycle ya inventariados. Las columnas históricas pueden seguir físicamente inertes, sin autorizar su uso en runtime.
- **S2B1 aceptada:** el borrado correctivo conserva cerrado el período anterior; creación y transiciones registran historia de fase append-only; Timeline no proyecta `phase_change` ni `label_change` de un período soft-deleted, aunque las filas internas permanezcan intactas.
- **S2B2 aceptada:** lista, perfil y Timeline proyectan fase/etiqueta/`ever_matched` con orden total y derivación por lote. `GET /crushes` usa sólo el filtro y total `ever_matched`; rechaza `match_status_id` y no devuelve buckets `flirt`/`unmatch`.
- **S2C aceptada:** se retiraron comandos de datos, dominio/repositorios/fórmulas Statistics legacy y fixtures/seeders que escribían columnas históricas. Los 14 endpoints quedaron temporalmente en formas vacías centralizadas; la decisión beta posterior reactiva sólo `dashboard` con fórmulas nativas y mantiene los otros 13 fuera de beta y vacíos.
- **API-RR-02D aceptada:** los seis escenarios de competencia corrieron contra MySQL/InnoDB mediante procesos y conexiones independientes. El harness usa un servicio efímero separado y su ciclo de vida sólo nombra `db-concurrency`; no tocó la pila local.
- **API-RR-02E aceptada:** `everMatched` ahora deriva de haber alcanzado Involucrado en el historial append-only, nunca de una etiqueta opcional; conserva monotonía tras el cierre y excluye períodos correctivamente borrados. `PATCH /crushes` rechaza el parámetro legacy `match_status_id` en vez de ignorarlo.
- **API-RR-02F-R2 aceptada:** las tres rutas que crean períodos rechazan `error.period.create.active_period_exists` sin mutación cuando existe uno vivo; la idempotencia se resuelve antes del guard y la concurrencia MySQL conserva un único ganador. Suite completa reportada: 1.010 pruebas sin fallos.
- **API-RR-02F-R3 aceptada:** `UpdatePeriodUseCase` valida desde la fase propia mediante `PeriodEntity`, registra el avance real y usa el período precedente sólo para comprobar que la nueva fecha inicial sea posterior a su cierre, sin mutarlo. Suite completa reportada: 1.015 pruebas sin fallos; concurrencia MySQL/InnoDB: 10 pruebas/89 assertions.
- **`API-RR-02G` aceptada y congelada tras `R3`:** `fetchLogicalPage()` numera grupos lógicos mediante funciones de ventana y mantiene juntas las filas de fase/etiqueta del mismo hito. `R2-H` aisló el bootstrap, `R2-V2` neutralizó sólo los países irrelevantes de los fixtures y `R3` retiró mediante `reorder()` el orden heredado exclusivamente del conteo agrupado compatible con `ONLY_FULL_GROUP_BY`. Los nueve casos pasaron sobre MySQL 8 real con 123 assertions; el harness completo quedó en 15/143 y la suite canónica en 1.054 pruebas sin fallos. iOS consume este contrato en `IOS-RR-03B2`.
- **`API-RR-STAT-01A` aceptada:** la proyección interna de dashboard calcula desde Crushes y períodos nativos el `winRate` disponible/no disponible, `actualStatus` y el desglose vivo por fase. Reutiliza la evidencia append-only de `everMatched`, excluye Crushes no visibles y períodos borrados, y no lee estados, columnas ni fórmulas legacy. El corte `RR-STAT-API`/`IOS-RR-05` usa valores anulables cuando falta muestra, `multiple_relationships`, `moment.byPhase` y `traits` bloqueado; API e iOS ya implementaron este contrato sin activar los otros 13 endpoints.
- **`RR-STAT-API` aceptada como implementación API:** `GET /statistics/dashboard` usa el ensamblador nativo para exponer resumen, `moment.byPhase` y los teasers estructurales de Crush/Experience; `traits` permanece `locked`/nulo. Los otros trece endpoints siguen respondiendo mediante sus formas vacías. La evidencia dirigida reportada fue 80 pruebas/485 assertions sin fallos. El ensamblador evita N+1, aunque la lista completa de Crushes se lee dos veces de forma acotada —para teasers y para la proyección existente—; no se atribuye una única lectura inexistente. El despliegue sigue condicionado al gate de integración, release y validación física `IOS-RR-06B`.
- **`API-RR-04A` aceptada y congelada tras `R`:** la API sustituyó sin compatibilidad el catálogo anterior por los once motivos finales y centralizó en `PeriodEntity` la correspondencia entre fase, etiqueta persistida y motivo de cierre para altas, cierres y correcciones históricas. El correctivo `R` hace fallar cerrado un alta no Cerrada que transporte motivo antes de construir o persistir la entidad, y fija que una corrección histórica acepta sólo el `ENDED_*` correspondiente a la etiqueta confirmada sin mutar ante mismatch. La evidencia reportada suma el lote original de 254 pruebas/1.176 assertions y el correctivo dirigido de 57/109, ambos sin fallos. Los IDs quedan congelados y `IOS-RR-07A` está desbloqueada.
- **`API-RR-04B` aceptada tras `R`:** la evidencia append-only de creación conserva fase, etiqueta y motivo iniciales y Timeline la incorpora por lote al item de creación, separada del snapshot mutable que siguen usando las acciones. El período inicial se identifica por la correlación estructural con el Crush y su fecha permanece igual a `crush.met_at`; el correctivo `R` ejecuta ese guard antes de las ramas Cerrada y viva, por lo que también un período inicial Cerrado/latest rechaza otra fecha con el mismo error estable y sin mutación, historia ni audit de éxito. La evidencia dirigida reportada para el correctivo fue 130 pruebas sin fallos. `IOS-RR-07B` queda desbloqueada.
- **`API-RR-04C` aceptada tras `R`:** `PATCH|DELETE /relationships/{relationshipId}/milestones/{milestoneId}` separan la corrección/eliminación latest-only del `PATCH /relationships/{id}` que registra historia nueva. La API conserva identidad, fecha y vínculo durante la corrección, impide editar hitos intermedios o eliminar el inicial, restaura el snapshot anterior al borrar el último posterior y publica en Timeline identidad, base y capacidades autoritativas. El correctivo `R` captura el snapshot confirmado antes de esperar los locks y lo revalida dentro de la transacción antes de escribir; las carreras PATCH/PATCH y PATCH/DELETE dejaron un único ganador sobre MySQL/InnoDB real (2 pruebas/23 assertions), y la regresión funcional dirigida quedó en 24/139. `IOS-RR-07B-R` queda desbloqueada.
- **`IOS-RR-07A` aceptada tras `R`:** iOS sustituyó los selectores anteriores por nueve situaciones humanas, consume los once motivos congelados y conserva fase/etiqueta base separadas del destino en durable write. El correctivo valida antes de habilitar «Corregir» que una `.liveUpdate` tenga una combinación base viva coherente, exige etiqueta válida cuando la base es Involucrado y elimina cualquier fallback a la fila visible; el rechazo deja intacta la operación y no altera el retry ciego. La evidencia dirigida del correctivo fue 85/85; la corrida inicial dejó la regresión real corregida y dos flakes de Timeline anteriores al selector, ajenos a este contrato.
- **`IOS-RR-07B` no aceptada tras validación interactiva; requiere correctivo:** decoding, presentación del snapshot y bloqueo de fecha quedaron automatizados, pero la acción «Editar relación» todavía entra en la actualización viva y `PeriodChangeApplier` añade un hito. La regla ya queda congelada: registrar un cambio crea historia; editar corrige exclusivamente el último hito en la misma identidad, con fecha inmutable y opciones derivadas del hito anterior; eliminar sólo admite el último hito no inicial y restaura el anterior. `API-RR-04C/R` ya aporta el contrato autoritativo; el siguiente gate es `IOS-RR-07B-R` para consumirlo.
- **`IOS-RR-05` aceptada en implementación tras `R`:** el decoder iOS consume las claves API reales `id` de `moment.byPhase`/`gender` y `id`/`slug`/`meaning` de `ducks`; la pantalla presenta conteos de fase sin calcular porcentajes y evita datos anteriores o `actualStatus` fabricado durante carga/error. El mock opt-in y tres recorridos UI ejercitan disponible, vacío/locked y error tras recarga. Se reportaron build incremental y 20/20 pruebas dirigidas verdes. La integración contra API local y la revisión física siguen en `IOS-RR-06B`.
- **IOS-RR-01B aceptada:** los tipos `Codable` de período, warning y catálogos numéricos nativos existen sin adaptadores de `stateId`; `IOS-RR-02` completó durable write, `IOS-RR-04A` completó los modelos HTTP compartidos y `IOS-RR-03A-W` conectó el warning a su consumidor productivo de presentación.
- **IOS-RR-02 aceptada:** snapshots, executors, payloads Vault V2 y proyecciones pendientes congelan sólo fase/etiqueta/motivo nativos. `IOS-RR-02R` añadió una regla de integridad compartida y preflights que impiden persistir o enviar `closed` sin motivo; `IOS-RR-03A1` completó la evidencia UI en las cuatro pantallas.
- **IOS-RR-03A1 aceptada:** el alta inicial de Crush mantiene un período obligatorio, permite las cinco fases nativas, presenta etiqueta sólo en Involucrado y exige motivo en Cerrado. El modelo y el executor verifican el payload final de `POST /crushes`; no aparece `REJECTED_LEGACY`.
- **IOS-RR-03A2 aceptada tras correctivo:** el alta standalone usa una intención durable distinta del attach y ejecuta `POST /relationships`. Timeline y perfil la ofrecen una sola vez, exclusivamente sobre el último período confirmado Cerrado, sin escritura incompatible pendiente y con fecha mínima igual al día civil posterior a su cierre; un cierre de hoy o un `endDate` inválido fallan cerrado. El attach desde Experience conserva su ruta propia.
- **IOS-RR-03A aceptada en implementación tras cerrar GWT-RR-06:** `POST /experiences` conserva su warning como resultado exitoso y lo publica una sola vez tras `.apiAccepted`; perfil y Home lo presentan mediante su router modal único, después de cerrar el formulario y cuando la lectura y la escritura pendiente permiten la acción. El sexto recorrido UI de Home pasó de rojo a verde sin cambiar el engine: el mock opt-in demorado de `POST /experiences` omitía `POST /vault/blobs`, de modo que el fallback `{}` impedía decodificar `CreateBlobResponse`, dejaba la entrada en `.syncing` y mantenía `blocksTimelineWrite`. El fixture ahora responde sólo bajo `--ui-testing-closed-period-warning-delayed` con un blob ya existente, permitiendo verificar y confirmar la escritura. La evidencia nueva es el test Home 1/1 (aviso, un alert, acción `create_new_period` y formulario standalone real); los otros 43 casos dirigidos ya estaban verdes en la corrida anterior y no se repitieron. No se modificaron API, Vault de producción, delays, retry ni mutación automática. La validación física y la regresión de release quedan en `IOS-RR-06`.
- **IOS-RR-04A aceptada tras corrección:** los DTOs confirmados y sus consumidores de Timeline, Home, perfil, lista y mapa leen fase/etiqueta/motivo/`everMatched` nativos. `IOS-RR-04A-R` retiró `CrushReciprocity` y `match_status_id` de la edición de Crush y volvió irrepresentable el filtro negativo que la API rechaza; `GET /crushes` sólo omite el filtro o envía `ever_matched=true`.
- **`IOS-RR-03B1` aceptada tras `R/R2`:** el discriminante durable separa actualización viva de corrección histórica; el executor histórico omite fase/etiqueta/inicio, conserva valores privados en Vault, la nota admite cualquier motivo y los UI tests usan selectores nativos. `R2` congela además la fase confirmada base separada del destino pendiente, la conserva en reemplazos/reinicio y la usa para filtrar y validar sin retroceso antes de Vault/red. Un snapshot vivo previo sin base no inventa una equivalencia y falla cerrado al entrar en corrección. La validación física permanece agrupada en `IOS-RR-06`.
- **`IOS-RR-03B2` aceptada tras `IOS-RR-03B2-R`:** Experience y cambio del período vivo forman un único durable write y un único `POST /experiences`, con identidad estable para Experience, período e hito. El preflight incorpora inicio, último hito y slots confirmados/pendientes; quitar la intención relacional permite guardar la Experience sola. La proyección pendiente conserva una única tarjeta de Experience sin decorar el período confirmado, y la lectura confirmada conserva `period_change_id`/`experience_id` y presenta fase+etiqueta correlacionadas en un solo hito. La nota privada admite cualquier motivo de cierre y el borrado vinculado advierte que Experience/vínculo desaparecen mientras el cambio y la historia permanecen. Evidencia dirigida reportada: 173 pruebas unitarias y 16 UI sin fallos; regresión completa y validación física quedan para `IOS-RR-06`.

### iOS

- Dominio, DTOs aislados y durable write ya usan fase/etiqueta/motivo nativos; `closed` sin motivo se bloquea antes de snapshot, Vault o red.
- Timeline, Home, perfil, lista/mapa y Statistics consumen lectura nativa. `RelationshipState` ya no tiene consumidor productivo externo; el símbolo histórico aún existe en iOS como candidato a limpieza posterior. El dashboard nativo está aceptado en implementación y conserva integración/física pendientes en `IOS-RR-06B`.
- El alta inicial de Crush y el alta standalone están validados. Esta última crea el primer período o uno posterior a un cierre explícito mediante `POST /relationships`, conserva la intención en el snapshot durable y limita la fecha al día civil posterior al último cierre. `PATCH /relationships/{id}` también quedó recongelado para las fases posteriores.
- `ExperienceFormModel` distingue Experience sin relación, attach a un período, creación de período y cambio del período vivo. La intención vinculada valida cronología y slot antes de Vault/red, conserva su identidad durante retry/corrección y se reconcilia como un único hecho correlacionado en Timeline.
- Las validaciones físicas pueden seguir agrupadas para `IOS-RR-06`, como pidió el usuario.

## Secuencia coordinada

1. **RR-P0 — decisiones de runtime cerradas.** Borrado correctivo, visibilidad de avances, alcance beta de Statistics y denominador nativo de `dashboard.summary.winRate` están resueltos. La regresión del único usuario es una decisión posterior y acotada a `RR-MIG-01`.
2. **API-RR-02S2A — corte greenfield — aceptada.** Registro, lifecycle y respuestas públicas ya no seleccionan ni exponen versión ni estado legacy; el corte preserva el `cutover` estable para Statistics y no ejecuta ninguna migración real.
3. **API-RR-02S2B1 — lifecycle e historia de fase — aceptada.** El borrado correctivo no reabre períodos anteriores; la historia de fase es append-only y Timeline excluye eventos de períodos soft-deleted sin borrar su historia interna.
4. **API-RR-02S2B2 — lecturas nativas definitivas — aceptada.** Resources/lista/perfil/Timeline usan proyección nativa por lote y `GET /crushes` sólo publica `ever_matched`.
5. **API-RR-02S2C — poda demostrada — aceptada.** Se retiraron símbolos, comandos, fórmulas y fixtures legacy sin consumidor válido; las columnas históricas quedan físicamente inertes para una futura operación de datos aislada.
6. **API-RR-02D — concurrencia real — aceptada.** Se validaron los seis escenarios con conexiones MySQL independientes, bloqueo InnoDB real e invariantes de historia/atomicidad.
7. **API-RR-02E — monotonía de `everMatched` — aceptada.** Perfil, lista, filtro y total usan la misma evidencia histórica nativa; la corrección no incorpora compatibilidad legacy.
8. **API-RR-02F — freeze para iOS — aceptada.** R2 retiró la supersesión implícita de las rutas de alta; R3 corrigió la validación desde la fase propia y la cronología inmutable de `PATCH /relationships/{id}`. API queda recongelada.
9. **IOS-RR-01A — dominio aislado en paralelo — aceptada.** iOS ya tiene los tipos puros `ConnectionPhase`, `ConnectionLabel` y `ClosureReason` con reglas locales no autoritativas, sin HTTP, decoding, Vault, snapshots, fixtures, pantallas ni consumidores legacy.
10. **IOS-RR-01B — transporte — aceptada.** DTOs nativos y decoding quedaron conectados a durable write en `IOS-RR-02` y a las lecturas compartidas en `IOS-RR-04A`, sin compatibilidad temporal.
11. **IOS-RR-02 — durable write compartido — aceptada.** Snapshots, executors y proyecciones pendientes son nativos; `IOS-RR-02R` bloquea combinaciones inválidas antes de Vault/red y `IOS-RR-03A1` cerró su evidencia UI.
12. **IOS-RR-03A — altas nativas y GWT-RR-06 — implementación aceptada.** `IOS-RR-03A1` cubre el período inicial de Crush y `IOS-RR-03A2` el alta standalone real. El warning exitoso de `POST /experiences` alcanza un store compartido y una única autoridad modal en perfil/Home. La retención y reevaluación esperan lectura inicial completa y liberación de escrituras pendientes. El sexto test UI de Home pasó 1/1 al completar exclusivamente su mock opt-in de registro de blob Vault; los cinco escenarios UI restantes y los unitarios relacionados ya estaban verdes. Regresión de release y validación física pendientes en `IOS-RR-06`.
13. **IOS-RR-04A — lectura confirmada nativa — aceptada.** DTOs/modelos compartidos, Timeline, Home, perfil, lista, mapa, overlays y correcciones usan el contrato nativo. El correctivo retiró `match_status_id` de `PATCH /crushes` y limitó el filtro de lista al único valor aceptado por API.
14. **API-RR-02G — Experience + cambio de período — aceptada y congelada.** `R2` corrigió el offset mediante índices de grupo lógicos; `R3` eliminó únicamente el orden irrelevante del conteo agrupado y dejó verdes los nueve casos sobre MySQL 8. No altera `PATCH` standalone ni el orden público de Timeline; el contrato ya está disponible para iOS.
15. **IOS-RR-03B1 — avance/cierre standalone — aceptada tras `R/R2`.** La intención histórica produce el body correcto; fase base y destino permanecen separados en el snapshot durable; picker, preflight, retry y corrección conservan no-retroceso e identidad. Validación física diferida a `IOS-RR-06`.
16. **IOS-RR-03B2 — cambio vinculado desde Experience — aceptada tras `R`.** Cronología y slot confirmados/pendientes se validan antes de persistir; Experience + cambio conservan una única proyección pendiente y un único hito confirmado correlacionado. El borrado mantiene la historia y la creación de período continúa como intención separada.
17. **RR-MIG-01 — dry-run legacy — `API-RR-03A` aceptada tras `R/R2`; aplicación diferida.** El planificador no inventa topología ni hitos, valida inventario/Experiences/readiness, contrasta identidades globales sobre los UUID candidatos y normaliza sólo fechas/horas completas. El comando es read-only y demuestra cero escrituras. `API-RR-03B` se ejecutará después de la beta, con backup y autorización separada; no bloquea el runtime de cuentas nuevas.
18. **RR-STAT — implementación aceptada.** `API-RR-STAT-01A` fijó la proyección interna y `RR-STAT-API` activó sólo el dashboard beta con valores anulables cuando falta muestra, `actualStatus` nativo, `moment.byPhase` y teasers estructurales; `traits` queda bloqueado. `IOS-RR-05-R` alineó decoder/presentación y dejó pruebas UI dirigidas verdes. La integración local y validación física restantes pertenecen a `IOS-RR-06B`; los otros 13 endpoints permanecen vacíos y fuera de beta.
19. **RR-STATUS-UX — catálogo y contrato API latest-only aceptados; correctivo iOS pendiente.** Antes de beta se hizo un corte estricto, sin compatibilidad, hacia los nueve estados humanos y los once motivos finales. `API-RR-04A/R` congeló el catálogo y su validación autoritativa; `IOS-RR-07A/R` sustituyó selectores, snapshots y presentación con la base confirmada preservada y fail-closed. `API-RR-04B/R` fijó la evidencia histórica de creación y la fecha del período inicial. `API-RR-04C/R` ya separa avance, corrección y eliminación con locks y rechazo estable de carreras; `IOS-RR-07B-R` debe consumir ese contrato para que «Editar relación» deje de crear un hito vivo. Sólo el último hito es editable, su fecha no cambia, el inicial no se elimina y borrar el último posterior restaura el anterior. No cambia `winRate` ni agrega métricas nuevas.
20. **RR-RELEASE — automatización iOS, auditoría y cliente aislado aceptados en implementación; gates pendientes.** GWT-RR-06 ya tiene implementación y recorrido Home dirigidos verdes. `IOS-RR-06A-R` retiró la deuda de selectores legacy y `IOS-RR-06B` ejecutó la regresión dirigida de 44 clases unitarias y 17 UI: dos expectativas de UI obsoletas se corrigieron y sus clases se reejecutaron verdes; un caso de cronología sólo flaqueó bajo contención paralela del simulador y pasó aislado, por lo que queda como riesgo a vigilar. `IOS-RR-06B-I` confirmó que `mdl-security-local` ya es un backend efímero separado. `IOS-RR-06B-II/R` añadió el único override representable: `--integration-local-api` sólo en Debug/simulador, hacia loopback, y un ATS limitado al plist Debug; la URL y el plist Release siguen de producción, sin carga arbitraria ni relajación TLS extra. `IOS-RR-06B-III-L` repobló únicamente el stack efímero activo con Términos y Privacidad sintéticos, activos, en inglés y español, y con los 250 países del catálogo versionado; el registro y la clave foránea elegida por iOS vuelven a tener sus prerrequisitos. El primer Crush confirmó después un fallo de topología, no de producto: la API aceptaba la escritura estructural pero el simulador no podía alcanzar el MinIO interno para subir el blob Vault cifrado. El bridge local aceptado publica sólo `127.0.0.1:9001` desde `app`; la API firma contra ese mismo loopback y Nginx reenvía sin alterar el `Host` firmado a MinIO dentro de la red aislada. No expone MinIO, desarrollo, AWS ni producción. La interacción humana reanudó esa operación y confirmó creación/edición sobre el stack aislado. Además, un 422 real de unicidad diaria provocado por una fila sintética y luego retirada dejó una única operación local en «Necesita revisión»: Corregir reabrió su snapshot y guardó el cambio; una segunda operación sobrevivió el reinicio y «Eliminar cambio» la descartó con confirmación. Las dos filas sintéticas fueron eliminadas y no quedó una Experience en el día de descarte. El correctivo iOS de la navegación de Experience en Home queda aceptado: la actividad sólo usa el UUID de su Crush para abrir el perfil y no el UUID de Experience; dos pruebas nuevas y la regresión dirigida relacionada reportaron 28/28 verdes, y la comprobación manual en `mdl-security-local` confirmó que abre el perfil propietario sin 404. Siguen pendientes dispositivo, accesibilidad/offline y Legal; esos gates y la actualización canónica/eliminación de planes siguen antes de cualquier beta. La operación sobre el único usuario antiguo queda fuera del gate de beta y conserva su propio backup y autorización posteriores.

## RR-STATUS-UX — estados humanos y cierre analítico

Situaciones visibles y matriz de destinos: [Trayectos permitidos](../../../rules/RELATIONSHIPS.md#trayectos). Catálogo final de cierre: [Cierre de una relación](../../../rules/RELATIONSHIPS.md#cierre). Registrar, corregir y eliminar: [Corrección de hitos y períodos](../../../rules/RELATIONSHIPS.md#cambios). Este plan conserva la evidencia de sus fases; no duplica catálogos.

Las fases de catálogo ya reportadas no incorporan por sí mismas la restricción de trayectos actual. La siguiente fase dependiente debe mapear `RP-TRA-002..006` antes de implementar o aceptar código. No se añaden estadísticas nuevas por este cambio.

## RR-MIG-01 — conversión del único usuario legacy

Esta fase prepara una operación server-side puntual; no añade compatibilidad legacy al runtime y no autoriza todavía a ejecutarla. El inventario recibido contiene 179 Crushes, 301 filas de relación y 25 Experiences vinculadas. La conversión aprobada es:

**Estado coordinado:** `API-RR-03A` queda aceptada tras `R/R2`. `R` retiró la topología y el total de hitos inventados, conserva sólo los nueve agregados aprobados, valida vínculos de Experience, expone cada secuencia bloqueada, separa `dryRunCompleted` de `readyToApply` y prueba contenido idéntico más ausencia de queries de escritura. `R2` limita el inventario global a los UUID candidatos sin filtrar por dueño, bloquea una identidad ocupada por otra cuenta sin revelar sus datos y valida la sintaxis completa de DATE/DATETIME. La aplicación `API-RR-03B` queda diferida hasta después de la beta y nunca se ejecuta sin backup y autorización separada.

| Estado legacy | Representación nativa |
|---|---|
| `1` — prospect | Interesado |
| `2` — ligue/flirt | Conocido; significa conquista/conocimiento con flirteo y no demuestra interés mutuo |
| `3` — one night stand | Involucrado + Casual |
| `4` — friends with benefits | Involucrado + FWB |
| `5` — relationship | Involucrado + Relación |
| `6` — rejected | Ambiguo respecto del actor: bloqueado para clasificación manual, sin valor de compatibilidad |
| `7` — no match | Cerrado + `MISSED_OPPORTUNITY` |
| `8` — ended | `ENDED_FWB` o `ENDED_RELATIONSHIP` sólo cuando la etiqueta anterior lo demuestra; cualquier otro caso se bloquea para clasificación manual |

Reglas del dry-run:

- no se infieren Interés mutuo, Citas ni Otro;
- una fila `rejected` no permite inferir quién rechazó y queda bloqueada; una fila `ended` sólo se clasifica cuando el estado inmediatamente anterior demuestra FWB o Relación, y en cualquier otro caso queda bloqueada. El dry-run no emite IDs antiguos, aliases ni sentinelas de compatibilidad;
- las 174 secuencias monótonas se reconstruyen como períodos e hitos reales; las cuatro secuencias con estado repetido colapsan el no-op sin fabricar historia y la regresión `2→5→2→3→7` queda bloqueada para decisión manual;
- las 12 Experiences ligadas al primer estado conservan asociación con la creación del período; de los 13 vínculos posteriores, los 10 que corresponden a un avance, etiqueta o cierre real conservan asociación al nuevo hito y los tres que sólo acompañan una repetición quedan standalone;
- las 246 filas abiertas declaradas no se toman como cierre verdadero: el dry-run reconstruye la cronología desde la secuencia ordenada, fechas e hitos terminales, y reporta cualquier solapamiento o fecha imposible;
- cada corrida produce conteos antes/después, anomalías, plan de rollback y verificación de UUID, ownership, Vault y correlaciones. La ejecución real exige backup probado, autorización explícita y revisión del resultado del dry-run.

### Aplicación diferida `API-RR-03B`

`03B` no es la migración de producción: es el ensayo ejecutable de la conversión sobre una base MySQL desechable con el mismo schema. Debe consumir exactamente el plan aceptado por `03A`, rechazar `readyToApply=false` o cualquier deriva de inventario y aplicar al usuario completo dentro de una única frontera transaccional; ningún Crush se convierte parcialmente. Debe preservar los UUID y vínculos clasificados por el plan, no fabricar hitos para repeticiones y detenerse ante la regresión manual pendiente.

La evidencia obligatoria de `03B` es: postcondiciones nativas sin historia huérfana, segundo apply idempotente sin duplicados y rollback que restaura byte por byte el fixture anterior. Sólo después se diseña una ejecución real separada con backup probado, UUID exacto y autorización explícita. Por decisión del usuario, ni `03B` ni esa ejecución bloquean la beta greenfield.

## Decisiones abiertas

Se resolverán de una en una antes de implementar la fase que dependa de ellas:

1. **Regresión legacy `2→5→2→3→7`:** el dry-run debe aportar fechas, cierres y referencias Vault antes de decidir si representa dos períodos con un cierre faltante o una corrección manual. Bloquea sólo la ejecución de ese Crush, nunca autoriza una inferencia automática.

## Dependencias y gates

- API completó `02S2A/B/C`, concurrencia, monotonía de `everMatched`, los contratos congelados `API-RR-02F-R2/R3`/`API-RR-02G`, el planificador read-only `API-RR-03A-R2`, la proyección interna `API-RR-STAT-01A`, el corte HTTP `RR-STAT-API`, el catálogo estricto `API-RR-04A/R`, el snapshot inicial/lock de fecha `API-RR-04B/R` y la corrección/eliminación latest-only `API-RR-04C/R`. iOS aceptó en implementación durable write, altas, lectura confirmada, `IOS-RR-03B1`, el cambio vinculado `IOS-RR-03B2-R`, `IOS-RR-05-R`, la regresión dirigida de `IOS-RR-06B`, la auditoría de aislamiento `IOS-RR-06B-I`, el cliente local Debug/simulador `IOS-RR-06B-II/R` y el corte de estados humanos `IOS-RR-07A/R`. El siguiente gate es `IOS-RR-07B-R`, que debe consumir el contrato de milestone y separar el registro de un cambio nuevo de su corrección/eliminación. `IOS-RR-06B-III-L` dejó documentos jurídicos sintéticos activos y el catálogo estático de 250 países del repositorio en la DB efímera; el bridge Vault local posterior hace alcanzable la URL presignada sólo por loopback, sin cambiar contratos ni abrir MinIO. El recorrido humano confirmó creación, resolución de durable write y navegación correcta desde una Experience reciente al perfil propietario, y detectó el defecto de edición anterior. Después del correctivo siguen validación física, accesibilidad/offline y cierre documental coordinado; la revisión Legal sigue siendo un gate beta independiente, no una fase de Relationship. `API-RR-03B` queda diferida y no bloquea cuentas nuevas.
- iOS puede iniciar sólo después de cerrar el código pendiente de Form UX o de una autorización explícita para solapar ambos trabajos; no se duplican cambios sobre los mismos formularios.
- La revisión humana de privacidad, transparencia y cambio de alto riesgo de [`legal/05_MDL_Auditoria_Juridica.md`](../../../legal/05_MDL_Auditoria_Juridica.md) §5 es obligatoria antes de habilitar la feature para personas reales. Legal no se modifica en esta iniciativa sin autorización.
- El cierre final transferirá las reglas estables a sus documentos de `rules/`, con índice en `PRODUCT.md`, y el detalle técnico a `TIMELINE.md`, `STATISTICS.md` y los documentos técnicos locales; recién entonces se eliminan estos planes.
