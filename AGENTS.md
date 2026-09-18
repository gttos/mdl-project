# AGENTS.md — Product

Estas reglas se aplican por igual a Codex, ChatGPT, Claude Code y cualquier otro agente que trabaje en `product/`.

## Autoridad documental

- [`docs/PRODUCT.md`](docs/PRODUCT.md): catálogo único de reglas funcionales, incluidas decisiones cuyo código está pendiente. Cada regla tiene ID; definirla no certifica su implementación. Las decisiones abiertas se identifican en el mismo archivo.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md): arquitectura transversal estable.
- [`docs/VAULT.md`](docs/VAULT.md), [`docs/TIMELINE.md`](docs/TIMELINE.md), [`docs/STATISTICS.md`](docs/STATISTICS.md): detalle de subsistemas y referencias al catálogo único; no mantienen reglas funcionales alternativas ni fórmulas duplicadas. No crear un cuarto catálogo por dominio.
- [`docs/legal/`](docs/legal/README.md): documentos jurídicos modulares; no se consolidan ni modifican sin alcance legal explícito.
- `docs/plans/active/<feature>/README.md`: una única feature maestra por iniciativa activa.

[`docs/product/business-blueprint.md`](docs/product/business-blueprint.md) es un artefacto de negocio mantenido directamente por el usuario. Los agentes pueden leerlo para detectar conflictos, pero no lo corrigen, resumen ni reestructuran sin una solicitud explícita. Puede conservar referencias históricas mientras el usuario lo termina; eso no convierte esas referencias en fuentes de autoridad ni autoriza a modificar el blueprint durante una consolidación general.

Las decisiones funcionales se incorporan primero a `PRODUCT.md`; el plan activo registra su ejecución y escenarios por ID. Si una definición anterior de un plan contradice el catálogo, debe reconciliarse antes de implementar esa parte. Si un plan local contradice Product, Product prevalece. Esto no modifica la autoridad jurídica de Legal.

Legal es autoridad en privacidad, retención, derechos y cumplimiento. Una contradicción entre Product/código y Legal no se resuelve por jerarquía automática: se documenta junto a la regla afectada y se eleva a revisión humana/legal.

## Qué se documenta aquí

- Decisiones de producto, escenarios Dado/Cuando/Entonces y criterios compartidos.
- Propiedad de datos, límites entre proyectos y contratos transversales.
- Impacto de privacidad/legal y orden global de una feature.

Los detalles de clases, endpoints, migraciones concretas, comandos y tests pertenecen al plan local de API/iOS/Android/LLM.

## Estructura obligatoria

- No crear archivos auxiliares de auditoría, investigación, decisión, estado, rollout, prompts, worklogs o checkpoints.
- No crear carpetas `completed` ni `archived`.
- Sólo puede existir un `README.md` por plan activo.
- Al completar un plan, integrar primero el comportamiento final en `PRODUCT.md`, `ARCHITECTURE.md` o el documento profundo correspondiente; confirmar que cada proyecto afectado actualizó su snapshot técnico estable; y recién entonces eliminar el plan maestro y todos los locales.
- No duplicar una regla: enlazar la sección canónica.
- Todo Markdown nuevo de Product se escribe en español.

Una iniciativa no se marca completada mientras conserve hechos estables únicamente en un plan activo o falte cualquiera de sus validaciones exigidas. En ese caso el estado es «implementación validada; cierre documental pendiente». Git conserva el historial: el cierre no crea informes, retrospectivas ni copias archivadas del plan.

## Cambios originados en otro proyecto

Si durante trabajo en API, iOS, Android o LLM aparece o cambia una decisión funcional:

1. actualizar primero el documento o plan maestro de Product;
2. referenciarlo desde el plan local;
3. implementar sólo después de que la decisión esté cerrada;
4. no conservar una versión alternativa de la regla en el repositorio local.

## Método de trabajo

1. Al entrar en un alcance nuevo de la sesión, leer `PRODUCT.md`, `ARCHITECTURE.md`, el documento profundo del subsistema afectado, Legal relevante y los planes activos relacionados. En turnos posteriores del mismo alcance, revisar el plan y las secciones afectadas; releer un documento completo sólo si cambió, aparece una decisión/contrato nuevo o hay una contradicción. Leer el blueprint sólo cuando la tarea afecte estrategia de negocio o el usuario lo pida.
2. Comparar documentación con el código cuando haya duda factual mediante `codebase-memory-mcp`, siguiendo íntegramente `../AGENTS.md` §6. No convertir el comportamiento accidental en una decisión de producto. Claude Code no reindexa.
3. Presentar las ambigüedades funcionales al usuario de una en una.
4. Preservar cambios ajenos y no tocar código, tests ni configuración cuando la tarea sea de coordinación documental.
5. Antes de finalizar, verificar enlaces, contradicciones, duplicados y `git diff --check` en los repositorios modificados.

Cuando se acepta una fase local sin cambio funcional ni contractual, Product se actualiza primero pero de forma mínima: sólo el estado vigente y el siguiente paso real del plan maestro. No reescribir GWT, reglas ni contexto que no hayan cambiado.

## Seguridad

No abrir ni inspeccionar `.env`, `.env.*`, credenciales, `Config.Env`, entornos de procesos o secretos de AWS. Sólo se permiten archivos `.example` con placeholders.

Los flujos repository-local de SpecKit quedan retirados: no deben generar `spec.md`, `tasks.md`, checklists ni historiales paralelos.
