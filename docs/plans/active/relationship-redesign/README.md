# Rediseño de relaciones

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
- Historial append-only de cambios de etiqueta.
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

## Contrato de producto aprobado

El enum plano `RelationshipState` desaparece del runtime y se reemplaza por tres ejes:

- **Fase de conexión** (`connection_phase`): Interesado → Conocido → Interés mutuo → Involucrado → Cerrado.
- **Etiqueta de vínculo** (`connection_label`): sólo en Involucrado — Citas, Casual, FWB, Relación u Otro.
- **Motivo de cierre** (`closure_reason`): `REJECTED`, `MISSED`, `ENDED` u `OTHER`.

Reglas cerradas:

1. Un período puede comenzar en cualquier fase y avanzar a cualquier fase posterior; puede saltar fases sin inventar eventos para las omitidas.
2. Nunca retrocede dentro del mismo período. Cerrado es terminal y se puede alcanzar desde cualquier fase viva.
3. Mientras existe un período vivo, sus avances y su cierre ocurren sobre ese mismo período mediante actualización. Crear otro período no puede cerrarlo ni supersederlo implícitamente. Reabrir después de un cierre significa crear un período nuevo; no se muta automáticamente el período cerrado.
4. `ONE_NIGHT_STAND` corresponde a Involucrado + Casual.
5. `REJECTED_LEGACY` no es un valor nativo seleccionable. Sólo queda reservado para una eventual operación histórica separada.
6. Sólo el último período cerrado permite edición histórica, limitada a motivo, nota y fecha de cierre; no mueve el inicio ni inserta períodos intermedios.
7. En esta iniciativa `match_status = Sí` deriva exclusivamente de haber alcanzado Involucrado alguna vez. No cambia la fase ni se infiere de una experiencia íntima.
8. Inicio y fin el mismo día representan un día civil (`DATEDIFF + 1`).
9. Cada cambio de etiqueta es append-only y aparece como evento de Timeline.
10. Una experiencia posterior a un período cerrado se guarda y nunca reabre automáticamente. Cuando corresponde, la respuesta ofrece `create_new_period` mediante la advertencia GWT-RR-06.
11. La beta publica únicamente `GET /statistics/dashboard`. Sus métricas independientes de relaciones se calculan desde Crush/Experience; `actualStatus` y `moment` usan períodos nativos. Los otros 13 endpoints permanecen fuera de beta con formas vacías estables y nunca ejecutan fórmulas legacy.
12. La tabla física puede seguir llamándose `relationships`; el dominio y el contrato funcional son `Period`.
13. El alta conjunta de Crush conserva un período inicial obligatorio. Ese período comparte la identidad estructural del bundle y no se elimina de forma independiente; eliminar el Crush elimina el grafo completo.
14. Una Experience puede vincularse como máximo a un período. Al eliminar la Experience se elimina el vínculo, no el período inicial ni el Crush.
15. `match_status` puede seguir expuesto como concepto de producto sólo si se calcula desde períodos; no puede provenir de la columna/estado legacy.
16. Borrar correctivamente un período no inicial elimina sólo ese período; cualquier período anterior permanece cerrado. Nunca se reabre de forma automática ni como efecto de borrado.
17. Timeline muestra cada avance de fase que ocurrió realmente. El período creado muestra su fase inicial como parte de su propio evento; cada transición posterior agrega su evento append-only. Los saltos sólo muestran el destino real y nunca eventos para fases omitidas.
18. El único filtro/totales nativo de coincidencia es `ever_matched`: alcanzó Involucrado alguna vez. Las categorías legacy `flirt` y `unmatch` no tienen semántica nativa, no leen `crushes.match_status_id` y se retiran de filtros y totales; no se muestran como ceros engañosos ni se infieren desde motivos de cierre.
19. Una transición se valida siempre desde la fase vigente del mismo período, nunca desde la fase de un período precedente. Si se corrige la fecha inicial de un período posterior, el período cerrado anterior permanece inmutable y la nueva fecha debe seguir siendo posterior a su cierre.
20. Avanzar, cambiar la etiqueta o cerrar un período vivo no exige crear una Experience: la operación standalone continúa siendo válida.
21. Al crear una Experience, la persona puede elegir además un cambio real sobre el período vivo del mismo Crush. Experience y cambio se confirman o fallan juntos; nunca queda sólo una mitad persistida.
22. El cambio vinculado usa el día civil de la Experience como fecha efectiva. Debe respetar inicio, orden cronológico y el único slot de cambio de relación por Crush y día; no permite insertar historia antes del último hito ni retroceder.
23. La intención vinculada puede avanzar fase, cambiar etiqueta o cerrar; una combinación fase + etiqueta o fase + cierre producida por el mismo guardado es un único hito correlacionado. Enviar los mismos valores sin cambio real no crea historia ni admite un vínculo ficticio.
24. Dentro de su único período vinculado, una Experience puede asociarse a un único hito: creación/inicio o cambio posterior. El auto-link ya aprobado puede resolver el hito inicial por día civil; un cambio posterior debe ser intención explícita y no se infiere sólo por compartir fecha. La asociación sobrevive a reintentos sin duplicarse.
25. Eliminar una Experience vinculada elimina la asociación y sus datos propios, pero no borra ni revierte el período ni su historia append-only. Editar sus datos propios tampoco reescribe retrospectivamente el cambio relacional; mover su fecha exige una corrección relacional explícita compatible o se rechaza sin mutación parcial.
26. Crear un período desde una Experience sigue siendo una intención distinta y explícita: sólo se permite cuando no existe un período vivo. Una Experience posterior a un cierre puede guardarse sola y ofrecer después `create_new_period`; nunca reabre por efecto lateral.

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
| Statistics | Vacío estable + cutover hasta definir métricas nativas; después, fórmulas sólo sobre períodos | assemblers y filtros de ocho estados ejecutables desde endpoints públicos |
| Eliminación y baja de cuenta | Se elimina/desvincula el grafo `periods` y sus blobs de forma atómica | checks o fallbacks a `RelationshipRepositoryInterface` |
| Tests/fixtures | Sólo vocabulario y rutas finales, con idempotencia, rollback y concurrencia | pruebas cuyo criterio sea preservar legacy o aceptar ambos contratos |
| Datos históricos | Operación puntual con backup, dry-run, reglas de anomalía y asociación sólo a hitos reales | confiar en `end_date`, fabricar cambios para conservar links o añadir compatibilidad runtime preventiva |

## Escenarios de aceptación

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-RR-01 | Una cuenta recién creada | Crea su primer Crush | Se crea un período inicial nativo y no se consulta ninguna versión de relación. |
| GWT-RR-02 | Un período vivo | Avanza de fase | Sólo acepta la misma fase o una posterior permitida; los saltos no fabrican fases omitidas. |
| GWT-RR-03 | Un período Involucrado | Cambia su etiqueta | Conserva el período y añade un cambio de etiqueta inmutable visible en Timeline. |
| GWT-RR-04 | Un período vivo | Se cierra | Exige motivo/fecha válidos, queda terminal y una reapertura funcional crea otro período. |
| GWT-RR-05 | El último período ya está cerrado | Se edita historia | Sólo motivo, nota o fecha de cierre pueden cambiar. |
| GWT-RR-06 | Existe un período cerrado | Se registra una Experience posterior | Se acepta con warning cuando corresponde y no se reabre ni crea un período por efecto lateral. |
| GWT-RR-07 | Se usa cualquier ruta standalone o compuesta | Llega `state_id` | Se rechaza sin mutación parcial; no existe una pantalla o traducción legacy. |
| GWT-RR-08 | Home, perfil, lista o Timeline leen una relación | Construyen su respuesta | Exponen sólo el contrato nativo y `match_status` derivado; no publican estados anteriores legacy. |
| GWT-RR-09 | Una escritura queda pendiente o requiere corrección | iOS reintenta tras reinicio | Reutiliza el mismo snapshot/IDs nativos y no duplica ni degrada la operación. |
| GWT-RR-10 | Se elimina una Experience, un Crush o una cuenta | Se procesa el grafo | Se desvinculan/eliminan períodos y blobs correctos de forma atómica, sin fallback legacy. |
| GWT-RR-11 | iOS abre Statistics en beta | Consulta `GET /statistics/dashboard` | Recibe el dashboard nativo; `actualStatus` y `moment.byPhase` derivan de períodos y no aparece `RelationshipState`. Los otros 13 endpoints no se consumen y permanecen vacíos. |
| GWT-RR-12 | Dos comandos compiten sobre el mismo Crush | La base los ejecuta realmente en paralelo | El historial final es único y válido, sin períodos activos incompatibles, duplicados ni mutación parcial. |
| GWT-RR-13 | Existe un período vivo | Se guarda una Experience con avance, cambio de etiqueta o cierre | Experience, período, historia y vínculo al hito se confirman atómicamente con el día civil de la Experience. |
| GWT-RR-14 | Existe un período vivo | Se avanza, etiqueta o cierra sin Experience | La actualización standalone conserva exactamente las mismas reglas y crea historia sin vínculo a Experience. |
| GWT-RR-15 | La intención vinculada retrocede, no cambia nada, ocupa un día ya usado o intenta insertar antes del último hito | Se guarda | Se rechaza el compuesto completo sin Experience, cambio ni filas de historia parciales; la persona puede quitar la intención relacional y guardar la Experience sola. |
| GWT-RR-16 | Una Experience está vinculada a un cambio confirmado | Se elimina o corrige la Experience | Su asociación se elimina o actualiza según corresponda, pero el cambio y la historia del período permanecen intactos. |
| GWT-RR-17 | Un guardado vinculado queda pendiente o se reintenta | iOS reanuda la operación | Reutiliza Experience, período e identidad de hito y no duplica ninguna fila, vínculo ni item de Timeline. |
| GWT-RR-18 | Un estado legacy posterior tiene Experience | El dry-run lo convierte | Sólo un avance, etiqueta o cierre real conserva el vínculo al hito; una repetición sin cambio queda standalone y una regresión se detiene para decisión manual. |

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
- **IOS-RR-01B aceptada:** los tipos `Codable` de período, warning y catálogos numéricos nativos existen sin adaptadores de `stateId`; `IOS-RR-02` completó durable write y `IOS-RR-04A` completó los modelos HTTP compartidos y su presentación. La auditoría `IOS-RR-06A` confirmó que el tipo del warning todavía no tiene consumidor productivo.
- **IOS-RR-02 aceptada:** snapshots, executors, payloads Vault V2 y proyecciones pendientes congelan sólo fase/etiqueta/motivo nativos. `IOS-RR-02R` añadió una regla de integridad compartida y preflights que impiden persistir o enviar `closed` sin motivo; `IOS-RR-03A1` completó la evidencia UI en las cuatro pantallas.
- **IOS-RR-03A1 aceptada:** el alta inicial de Crush mantiene un período obligatorio, permite las cinco fases nativas, presenta etiqueta sólo en Involucrado y exige motivo en Cerrado. El modelo y el executor verifican el payload final de `POST /crushes`; no aparece `REJECTED_LEGACY`.
- **IOS-RR-03A2 aceptada tras correctivo:** el alta standalone usa una intención durable distinta del attach y ejecuta `POST /relationships`. Timeline y perfil la ofrecen una sola vez, exclusivamente sobre el último período confirmado Cerrado, sin escritura incompatible pendiente y con fecha mínima igual al día civil posterior a su cierre; un cierre de hoy o un `endDate` inválido fallan cerrado. El attach desde Experience conserva su ruta propia.
- **IOS-RR-03A reabierta sólo para GWT-RR-06:** `ExperienceService` y `ExperienceWriteExecutor` descartan hoy el cuerpo exitoso de `POST /experiences`; `ExperienceClosurePeriodWarning` no tiene consumidores. Falta preservar el éxito durable, presentar la advertencia una sola vez y ofrecer `create_new_period` únicamente mediante acción explícita.
- **IOS-RR-04A aceptada tras corrección:** los DTOs confirmados y sus consumidores de Timeline, Home, perfil, lista y mapa leen fase/etiqueta/motivo/`everMatched` nativos. `IOS-RR-04A-R` retiró `CrushReciprocity` y `match_status_id` de la edición de Crush y volvió irrepresentable el filtro negativo que la API rechaza; `GET /crushes` sólo omite el filtro o envía `ever_matched=true`.
- **`IOS-RR-03B1` aceptada tras `R/R2`:** el discriminante durable separa actualización viva de corrección histórica; el executor histórico omite fase/etiqueta/inicio, conserva valores privados en Vault, la nota admite cualquier motivo y los UI tests usan selectores nativos. `R2` congela además la fase confirmada base separada del destino pendiente, la conserva en reemplazos/reinicio y la usa para filtrar y validar sin retroceso antes de Vault/red. Un snapshot vivo previo sin base no inventa una equivalencia y falla cerrado al entrar en corrección. La validación física permanece agrupada en `IOS-RR-06`.
- **`IOS-RR-03B2` aceptada tras `IOS-RR-03B2-R`:** Experience y cambio del período vivo forman un único durable write y un único `POST /experiences`, con identidad estable para Experience, período e hito. El preflight incorpora inicio, último hito y slots confirmados/pendientes; quitar la intención relacional permite guardar la Experience sola. La proyección pendiente conserva una única tarjeta de Experience sin decorar el período confirmado, y la lectura confirmada conserva `period_change_id`/`experience_id` y presenta fase+etiqueta correlacionadas en un solo hito. La nota privada admite cualquier motivo de cierre y el borrado vinculado advierte que Experience/vínculo desaparecen mientras el cambio y la historia permanecen. Evidencia dirigida reportada: 173 pruebas unitarias y 16 UI sin fallos; regresión completa y validación física quedan para `IOS-RR-06`.

### iOS

- Dominio, DTOs aislados y durable write ya usan fase/etiqueta/motivo nativos; `closed` sin motivo se bloquea antes de snapshot, Vault o red.
- Timeline, Home, perfil, lista y mapa ya consumen únicamente la lectura confirmada nativa. `RelationshipState` queda acotado a Statistics hasta `IOS-RR-05`, sin participar en transporte, edición de Crush ni acciones de períodos.
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
12. **IOS-RR-03A — altas nativas — núcleo aceptado; GWT-RR-06 pendiente.** `IOS-RR-03A1` cubre el período inicial de Crush y `IOS-RR-03A2` el alta standalone real. `IOS-RR-03A-W` debe conectar el warning exitoso de `POST /experiences` con la presentación y la acción explícita de nuevo período sin convertirlo en error ni mutación automática.
13. **IOS-RR-04A — lectura confirmada nativa — aceptada.** DTOs/modelos compartidos, Timeline, Home, perfil, lista, mapa, overlays y correcciones usan el contrato nativo. El correctivo retiró `match_status_id` de `PATCH /crushes` y limitó el filtro de lista al único valor aceptado por API.
14. **API-RR-02G — Experience + cambio de período — aceptada y congelada.** `R2` corrigió el offset mediante índices de grupo lógicos; `R3` eliminó únicamente el orden irrelevante del conteo agrupado y dejó verdes los nueve casos sobre MySQL 8. No altera `PATCH` standalone ni el orden público de Timeline; el contrato ya está disponible para iOS.
15. **IOS-RR-03B1 — avance/cierre standalone — aceptada tras `R/R2`.** La intención histórica produce el body correcto; fase base y destino permanecen separados en el snapshot durable; picker, preflight, retry y corrección conservan no-retroceso e identidad. Validación física diferida a `IOS-RR-06`.
16. **IOS-RR-03B2 — cambio vinculado desde Experience — aceptada tras `R`.** Cronología y slot confirmados/pendientes se validan antes de persistir; Experience + cambio conservan una única proyección pendiente y un único hito confirmado correlacionado. El borrado mantiene la historia y la creación de período continúa como intención separada.
17. **RR-MIG-01 — dry-run legacy — `API-RR-03A` aceptada tras `R/R2`; aplicación diferida.** El planificador no inventa topología ni hitos, valida inventario/Experiences/readiness, contrasta identidades globales sobre los UUID candidatos y normaliza sólo fechas/horas completas. El comando es read-only y demuestra cero escrituras. `API-RR-03B` se ejecutará después de la beta, con backup y autorización separada; no bloquea el runtime de cuentas nuevas.
18. **RR-STAT.** Implementar únicamente `dashboard` para beta: restaurar métricas de Crush/Experience, migrar `actualStatus` y `moment` a períodos y resolver `winRate`. Los otros 13 endpoints permanecen vacíos y fuera de beta.
19. **RR-RELEASE.** Cerrar `IOS-RR-03A-W`, corregir la deuda de UI tests detectada por `IOS-RR-06A`, repetir la regresión API+iOS —incluido el ajuste aditivo de Timeline de `IOS-RR-03B2`—, validación física, revisión Legal, actualización de documentos canónicos y eliminación de planes activos. La operación sobre el único usuario antiguo queda fuera del gate de beta y conserva su propio backup y autorización posteriores.

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
| `6` — rejected | Cerrado + `REJECTED_LEGACY`, reservado sólo para conversión |
| `7` — no match | Cerrado + `MISSED` |
| `8` — ended | Cerrado + `ENDED` |

Reglas del dry-run:

- no se infieren Interés mutuo, Citas ni Otro;
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

- API completó `02S2A/B/C`, concurrencia, monotonía de `everMatched`, los contratos congelados `API-RR-02F-R2/R3`/`API-RR-02G` y el planificador read-only `API-RR-03A-R2`. iOS aceptó durable write, altas, lectura confirmada, `IOS-RR-03B1` y el cambio vinculado `IOS-RR-03B2-R`. El siguiente gate de beta es Statistics con `winRate = everMatched / Crushes visibles no borrados`; `API-RR-03B` queda diferida y no bloquea cuentas nuevas.
- iOS puede iniciar sólo después de cerrar el código pendiente de Form UX o de una autorización explícita para solapar ambos trabajos; no se duplican cambios sobre los mismos formularios.
- La revisión humana de privacidad, transparencia y cambio de alto riesgo de [`legal/05_MDL_Auditoria_Juridica.md`](../../../legal/05_MDL_Auditoria_Juridica.md) §5 es obligatoria antes de habilitar la feature para personas reales. Legal no se modifica en esta iniciativa sin autorización.
- El cierre final transferirá las reglas estables a `PRODUCT.md`, `TIMELINE.md`, `STATISTICS.md` y los documentos técnicos locales; recién entonces se eliminan estos planes.
