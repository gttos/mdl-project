# AGENTS.md — Product

Estas reglas se aplican por igual a Codex, ChatGPT, Claude Code y cualquier otro agente que trabaje en `product/`.

## Autoridad documental

- [`docs/PRODUCT.md`](docs/PRODUCT.md): catálogo único de reglas funcionales, incluidas decisiones cuyo código está pendiente. Cada regla tiene ID; definirla no certifica su implementación. Las decisiones abiertas están en [`docs/DECISIONES_PARA_JIRA.md`](docs/DECISIONES_PARA_JIRA.md); las funciones futuras aprobadas, en [`docs/FUNCIONES_FUTURAS_PARA_JIRA.md`](docs/FUNCIONES_FUTURAS_PARA_JIRA.md).
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md): referencias a la documentación técnica de los proyectos.
- [`docs/VAULT.md`](docs/VAULT.md), [`docs/TIMELINE.md`](docs/TIMELINE.md), [`docs/STATISTICS.md`](docs/STATISTICS.md): referencias a las reglas funcionales y la documentación técnica de cada subsistema; no mantienen reglas funcionales alternativas ni fórmulas duplicadas. No crear un cuarto catálogo por dominio.
- [`docs/legal/`](docs/legal/README.md): documentos jurídicos modulares; no se consolidan ni modifican sin alcance legal explícito.
- `docs/plans/active/<feature>/README.md`: una única feature maestra por iniciativa activa.

[`docs/product/business-blueprint.md`](docs/product/business-blueprint.md) es un artefacto de negocio mantenido directamente por el usuario. Los agentes pueden leerlo para detectar conflictos, pero no lo corrigen, resumen ni reestructuran sin una solicitud explícita. Puede conservar referencias históricas mientras el usuario lo termina; eso no convierte esas referencias en fuentes de autoridad ni autoriza a modificar el blueprint durante una consolidación general.

Las reglas por dominio se definen una sola vez en `docs/rules/` y se enlazan desde `PRODUCT.md`, que conserva principios y vocabulario compartidos; el plan activo registra su ejecución y escenarios por ID. Las funciones futuras y las decisiones abiertas utilizan sus documentos específicos. Si una definición anterior de un plan contradice el catálogo, debe reconciliarse antes de implementar esa parte. Si un plan local contradice Product, Product prevalece. Esto no modifica la autoridad jurídica de Legal.

### Catálogo inquebrantable

Las reglas finalizadas de `docs/PRODUCT.md` y `docs/rules/` son restricciones obligatorias y no se reinterpretan para acomodar código, tests, planes, specs o documentación local. Si ejecutar el alcance autorizado exige una contradicción, excepción, decisión pendiente o interpretación incierta, se aplica el bloqueo de `../AGENTS.md` §2, citando la regla y la dependencia concreta. Los hallazgos laterales se difieren conforme a §3; no habilitan otro correctivo ni cambian la prioridad. Sólo una decisión explícita del usuario permite cambiar Product, siempre antes que los consumidores técnicos.

Legal es autoridad en privacidad, retención, derechos y cumplimiento. Una contradicción entre Product/código y Legal no se resuelve por jerarquía automática: se documenta junto a la regla afectada y se eleva a revisión humana/legal.

## Qué se documenta aquí

- Funcionamiento aprobado: significado de datos y acciones, requisitos, condiciones, resultados, errores, conservación funcional y escenarios observables.
- Alcance funcional compartido entre proyectos, dependencias de privacidad/Legal y coordinación de una iniciativa autorizada.
- Funciones futuras aprobadas y decisiones abiertas, cada una fuera del catálogo actual y en su destino específico.

La arquitectura, clases, endpoints, payloads, tablas, migraciones, criptografía, comandos y pruebas concretas pertenecen a los documentos técnicos o planes locales. Los nombres técnicos de campos sólo se incluyen en Product cuando son necesarios para identificar inequívocamente el dato funcional, sin desarrollar su implementación.

## Contenido por documento

| Documento | Contiene | Queda fuera |
|---|---|---|
| `README.md` | Entrada e índice de documentos. | Reglas, decisiones o estado de ejecución duplicados. |
| `docs/PRODUCT.md` | Índice de dominios, vocabulario y principios funcionales compartidos. Enlaza las definiciones por dominio. | Un segundo catálogo, detalles técnicos, diseño, pendientes o resultados. |
| `docs/rules/*.md` | Definición única de cada regla funcional con ID, condiciones, resultado, límites y escenarios de aceptación estables cuando correspondan. Fórmulas, unidades y catálogos funcionales pertenecen a su dominio. | Clases, contratos HTTP/almacenamiento, diseño visual, copy completo de pantallas, motivos de decisión, fases y pruebas ejecutadas. |
| `docs/ARCHITECTURE.md` | Referencias a arquitectura y contratos técnicos de los proyectos. | Una arquitectura transversal paralela o nuevos contratos técnicos en Product. |
| `docs/VAULT.md`, `docs/TIMELINE.md`, `docs/STATISTICS.md` | Referencias a reglas funcionales y documentos técnicos del subsistema. | Reglas, fórmulas o implementación duplicadas. |
| `docs/FUNCIONES_FUTURAS_PARA_JIRA.md` | Funciones futuras expresamente aprobadas, alcance, límites de activación y referencias necesarias para llevarlas a Jira. | Funciones actuales sólo porque falte implementarlas, propuestas no aprobadas o una segunda definición de la regla actual. |
| `docs/DECISIONES_PARA_JIRA.md` | Pregunta pendiente, alternativas y límites ya fijados; aprobación que falta y referencia a la superficie afectada. | Una respuesta elegida por el agente, regla finalizada o relato del debate. |
| `docs/plans/active/<feature>/README.md` | Objetivo, alcance, IDs canónicos, GWT de ejecución, coordinación, orden de fases, estado vigente, criterios de aceptación y dependencias abiertas enlazadas. | Catálogo paralelo, especificación técnica local, diseño visual, diario de trabajo y resultados de pasadas anteriores. |
| `docs/PENDIENTES.md` | Hallazgos laterales autorizados: una entrada breve con proyecto/superficie, qué revisar y referencia; distingue sospecha de defecto comprobado. | Backlog funcional completo, plan de solución, prioridades inventadas o autorización implícita para corregir. |
| `docs/legal/` | Expediente jurídico modular, requisitos y criterios de su ámbito. | Cambios o consolidaciones por una limpieza general sin alcance legal autorizado. |
| `docs/product/business-blueprint.md` | Artefacto de negocio mantenido por el usuario. | Reescrituras automáticas o uso como autoridad funcional alternativa. |
| `AGENTS.md` / `CLAUDE.md` | Directrices de trabajo y documentación / enlace a las mismas instrucciones. | Reglas de producto o políticas distintas por modelo. |

## Redacción de Product

Aplicar las [directrices comunes](../AGENTS.md#contenido-y-redacción-de-product-api-e-ios). El catálogo describe qué hace MDL y qué significa cada dato con redacción afirmativa, precisa y comprobable; no relata cómo se decidió.

- Una definición por ID y un significado por término. Incluir las condiciones y exclusiones que cambian el resultado; conservar la precisión de límites, fechas, fórmulas, ausencia de datos y errores.
- Separar definición y ejecución: una regla requerida pero no implementada conserva su definición; su falta de cumplimiento se identifica en el plan o documento técnico. Una función de otra etapa y una decisión todavía abierta nunca se presentan como funcionamiento actual.
- No describir colores, medidas, tipografías, posiciones, composición de pantallas ni redacción literal de botones o ayudas. Expresar la información que se comunica, su significado y el efecto de la acción; el vocabulario funcional y los elementos de un catálogo conservan su identidad.
- Conservar una explicación sólo si precisa una condición, frontera o interpretación. No añadir antecedentes, motivos circunstanciales, «por ahora», «hemos decidido», hipótesis de cambios futuros ni comparaciones con otras aplicaciones.
- Enlazar contratos y procedimientos técnicos en sus proyectos. No reconstruirlos dentro de las páginas de referencia ni trasladar allí el contenido retirado del catálogo.

## Estructura obligatoria

- Fuera de los destinos existentes autorizados en «Contenido por documento», no crear archivos auxiliares de auditoría, investigación, decisión, estado, rollout, prompts, worklogs o checkpoints.
- No crear carpetas `completed` ni `archived`.
- Sólo puede existir un `README.md` por plan activo.
- Al completar un plan, integrar las definiciones funcionales finales en el catálogo (`docs/rules/` para cada dominio y `PRODUCT.md` para principios compartidos) y actualizar sus enlaces; confirmar que cada proyecto afectado actualizó su documentación técnica; y recién entonces eliminar el plan maestro y todos los locales. Las páginas de referencia de Product conservan su función de enlace.
- No duplicar una regla: enlazar la sección canónica.
- [`docs/PENDIENTES.md`](docs/PENDIENTES.md) es la única lista transversal de hallazgos diferidos autorizados; una entrada breve por hallazgo, sin informes o planes paralelos.
- Todo Markdown nuevo de Product se escribe en español.

Una iniciativa no se marca completada mientras conserve hechos estables únicamente en un plan activo o falte cualquiera de sus validaciones exigidas. En ese caso el estado es «implementación validada; cierre documental pendiente». Git conserva el historial: el cierre no crea informes, retrospectivas ni copias archivadas del plan.

## Cambios originados en otro proyecto

Si durante trabajo en API, iOS, Android o LLM aparece o cambia una decisión funcional:

1. actualizar primero la definición canónica del dominio o principio compartido y su índice; si es futura o sigue abierta, usar su destino específico y reflejar la dependencia en el plan maestro;
2. referenciarlo desde el plan local;
3. implementar sólo después de que la decisión esté cerrada;
4. no conservar una versión alternativa de la regla en el repositorio local.

## Método de trabajo

1. Al entrar en un alcance nuevo de la sesión, leer `PRODUCT.md`, `ARCHITECTURE.md`, el documento profundo del subsistema afectado, Legal relevante y los planes activos relacionados. En turnos posteriores del mismo alcance, revisar el plan y las secciones afectadas; releer un documento completo sólo si cambió, aparece una decisión/contrato nuevo o hay una contradicción. Leer el blueprint sólo cuando la tarea afecte estrategia de negocio o el usuario lo pida.
2. Comparar documentación con el código cuando haya duda factual mediante `codebase-memory-mcp`, siguiendo íntegramente `../AGENTS.md` §6. No convertir el comportamiento accidental en una decisión de producto. Claude Code no reindexa.
3. Presentar las ambigüedades funcionales necesarias para ejecutar el alcance de una en una. Para hallazgos laterales, aplicar [foco y diferimiento](../AGENTS.md#foco-y-hallazgos-fuera-de-alcance): continuar el objetivo, agrupar la consulta para anotarlos y no entregar prompts alternativos.
4. Preservar cambios ajenos y no tocar código, tests ni configuración cuando la tarea sea de coordinación documental.
5. Antes de finalizar, verificar enlaces, contradicciones, duplicados y `git diff --check` en los repositorios modificados.

Cuando se acepta una fase local sin cambio funcional ni contractual, Product se actualiza primero pero de forma mínima: sólo el estado vigente y el siguiente paso real del plan maestro. No reescribir GWT, reglas ni contexto que no hayan cambiado.

## Seguridad

No abrir ni inspeccionar `.env`, `.env.*`, credenciales, `Config.Env`, entornos de procesos o secretos de AWS. Sólo se permiten archivos `.example` con placeholders.

Los flujos repository-local de SpecKit quedan retirados: no deben generar `spec.md`, `tasks.md`, checklists ni historiales paralelos.
