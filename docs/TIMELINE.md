# Timeline y ciclo de vida de Crush / Experience / Relationship

El catálogo funcional se consulta desde el [índice de reglas](PRODUCT.md), con una definición por dominio en `rules/`. Este documento conserva la correspondencia entre conceptos e implementación y los destinos de consulta del subsistema. No establece una segunda matriz de reglas ni certifica el estado del código.

## 1. Las entidades

| Concepto | Identidad/representación |
|---|---|
| Cuenta | Propietario autenticado de las entidades. |
| Crush | UUID funcional; estructura API y snapshot privado Vault. |
| Experience | UUID funcional y referencia al crush; fecha/satisfacción estructurales y relato cifrado. |
| Period | UUID funcional; la tabla o ruta puede conservar el nombre técnico `relationships`. |
| Hito | Identidad del cambio real dentro de un período; evidencia inicial o posterior. |
| Correlación | Identidad que agrupa componentes del mismo hecho en las proyecciones. |

Vocabulario y fechas compartidas: [Índice de reglas](PRODUCT.md#vocabulario). Los datos obligatorios y privados se definen en [Crush](rules/CRUSH.md) y [Experiencias](rules/EXPERIENCES.md); la cronología relacional, en [Relaciones](rules/RELATIONSHIPS.md).

## 2. Alta — el bundle inicial

Contrato completo: [Alta del Crush](rules/CRUSH.md#alta), reglas `BR-4.01`, `RP-CRU-001`, `RP-CRU-004..005` y [experiencia inicial](rules/EXPERIENCES.md#inicial), `RP-INI-001..003`. La composición técnica usa identidades funcionales separadas y una correlación inicial.

## 3. Cronología y unicidad por día civil

Contrato completo: [Vocabulario y fechas](PRODUCT.md#vocabulario), [perfil](rules/CRUSH.md#perfil), [experiencias](rules/EXPERIENCES.md) y [períodos](rules/RELATIONSHIPS.md#periodos-hitos).

### Reasociación retroactiva a la experiencia inicial

Referencia: `RP-EXP-006`, `RP-EXP-014..016`. La proyección debe resolver la correlación funcional; no confunde el ID de experiencia con el ID de una entry privada.

### Coincidencia entre experiencia y relación

Referencia: `RP-EXP-005..009`, `RP-EXP-013..017`. La coincidencia con creación y el cambio explícito posterior son intenciones distintas.

## 4. Relaciones — matriz de transición de estados

La matriz vigente está en [Trayectos permitidos](rules/RELATIONSHIPS.md#trayectos) y define todos los cambios relacionales admitidos.

Fase, etiqueta y motivo son dimensiones separadas. Catálogo de cierre: [Cierre relacional](rules/RELATIONSHIPS.md#cierre). Creación, avance y corrección tienen contratos distintos: [Corrección de hitos y períodos](rules/RELATIONSHIPS.md#cambios).

### Match status

Derivación y límites: [Coincidencia histórica](rules/RELATIONSHIPS.md#coincidencia-historica), `BR-7.01` y `RP-MAT-001..003`. El efecto de una corrección sobre la única evidencia de match figura como decisión pendiente.

## 5. Edición y borrado

Referencias únicas:

- Experiencias: [Edición de experiencias](rules/EXPERIENCES.md#editar).
- Hitos y períodos: [Corrección de hitos y períodos](rules/RELATIONSHIPS.md#cambios).
- Crush: [Archivo y eliminación del Crush](rules/CRUSH.md#archivo-eliminacion).
- Baja y conservación estructural: [Baja y ciclo de vida](rules/DATA_LIFECYCLE.md).
- Borrado privado/purga: [Retirada de contenido privado](rules/VAULT.md#retirada-contenido) y [Tareas automáticas y plazos](rules/DATA_LIFECYCLE.md#tareas).

`endDate == null` describe un período vivo; no es una definición suficiente de elegibilidad para corregir hitos. Los consumidores deben resolver la identidad y posición del hito conforme al catálogo.

## 6. Reconstrucción del Timeline

Contrato observable: [Contenido del Timeline](rules/TIMELINE.md#contenido).

La lectura combina entidades, correlaciones y evidencia histórica de creación/cambio. El snapshot vigente sirve para estado y acciones; el snapshot de creación sirve para representar el hecho inicial. La paginación trabaja sobre hechos lógicos agrupados, sin separar filas de fase y etiqueta del mismo cambio.

El orden técnico documentado es `event_date DESC, created_at DESC, item_type ASC, item_id ASC`. Su aplicación y pruebas pertenecen a API; no redefine el día civil ni la unidad de agrupación.

## 7. Matriz de acciones visibles en el Timeline

Contrato completo: [Acciones del Timeline](rules/TIMELINE.md#acciones), con elegibilidad de [Experiencias](rules/EXPERIENCES.md), [Corrección de hitos y períodos](rules/RELATIONSHIPS.md#cambios) y bloqueos de [Guardado y sincronización](rules/SYNC.md). No se mantiene aquí otra tabla que pueda divergir.

## 8. Referencias

- [Arquitectura API](../../api/docs/ARCHITECTURE.md), [arquitectura iOS](../../ios/docs/ARCHITECTURE.md).
- [Ejecución relacional](plans/active/relationship-redesign/README.md).
- [Ejecución durable](plans/active/durable-write-feedback/README.md).
- [Paridad Android](plans/active/android-parity/README.md).
