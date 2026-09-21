# Timeline y ciclo de vida de Crush / Experience / Relationship Period

El catálogo funcional se consulta desde el [índice de reglas](PRODUCT.md), con una definición por dominio en `rules/`. Este documento conserva la correspondencia entre conceptos e implementación y los destinos de consulta del subsistema. No establece una segunda matriz de reglas ni certifica el estado del código.

## 1. Las entidades

| Concepto | Identidad/representación |
|---|---|
| Cuenta | Propietario autenticado de las entidades. |
| Crush | UUID funcional; estructura API y snapshot privado Vault. |
| Experience | UUID funcional y referencia al crush; fecha/satisfacción estructurales y relato cifrado. |
| Relationship period | UUID funcional persistido en `relationship_periods`; contiene un tramo continuo de la historia. |
| Relationship milestone | UUID funcional persistido en `relationship_milestones`; crea el período o registra un cambio real de situación. |
| Correlación | Identidad que agrupa componentes del mismo hecho en las proyecciones. |

Vocabulario y fechas compartidas: [Índice de reglas](PRODUCT.md#vocabulario). Los datos obligatorios y privados se definen en [Crush](rules/CRUSH.md) y [Experiencias](rules/EXPERIENCES.md); la cronología relacional, en [Relaciones](rules/RELATIONSHIPS.md).

## 2. Alta — el bundle inicial

Contrato completo: [Alta del Crush](rules/CRUSH.md#alta), reglas `BR-4.01`, `RP-CRU-001`, `RP-CRU-004..005` y [experiencia inicial](rules/EXPERIENCES.md#inicial), `RP-INI-001..003`. La composición técnica usa identidades funcionales separadas y una correlación inicial.

## 3. Cronología y unicidad por día civil

Contrato completo: [Vocabulario y fechas](PRODUCT.md#vocabulario), [perfil](rules/CRUSH.md#perfil), [experiencias](rules/EXPERIENCES.md) y [períodos](rules/RELATIONSHIPS.md#periodos-hitos).

### Experiencia inicial invariable

Referencia: `RP-EXP-006`, `RP-EXP-014..016`. El bundle siempre contiene su experiencia inicial; una experiencia posterior no puede ocupar ese lugar ni vincularse por coincidencia de fecha. La proyección no confunde el ID de experiencia con el ID de una entry privada.

### Vínculo entre experiencia e hito

Referencia: `RP-EXP-005..006`, `RP-EXP-010` y `RP-EXP-013..017`. Todo hito apunta de forma explícita a una experiencia; compartir fecha nunca crea el vínculo.

## 4. Relaciones — situaciones y trayectos

La matriz vigente está en [Trayectos permitidos](rules/RELATIONSHIPS.md#trayectos) y define todos los cambios relacionales admitidos.

`RelationshipSituation` es la única dimensión de estado y conserva claves internas en inglés, IDs enteros y copy visible `es-ES`. El catálogo de cierre está en [Cierre relacional](rules/RELATIONSHIPS.md#cierre). Creación, avance y corrección tienen contratos distintos en [Crear y corregir hitos](rules/RELATIONSHIPS.md#cambios).

### Match status

Proyecciones y recálculo: [Resultado de match del Crush](rules/CRUSH.md#match). Los hitos 5–9 que aportan evidencia se definen en [Relaciones](rules/RELATIONSHIPS.md#evidencia-match).

## 5. Edición y borrado

Referencias únicas:

- Experiencias: [Edición de experiencias](rules/EXPERIENCES.md#editar).
- Hitos y períodos: [Crear y corregir hitos](rules/RELATIONSHIPS.md#cambios) y [Eliminar hitos o períodos](rules/RELATIONSHIPS.md#eliminacion).
- Crush: [Archivo y eliminación del Crush](rules/CRUSH.md#archivo-eliminacion).
- Baja y conservación estructural: [Baja y ciclo de vida](rules/DATA_LIFECYCLE.md).
- Borrado privado/purga: [Retirada de contenido privado](rules/VAULT.md#retirada-contenido) y [Tareas automáticas y plazos](rules/DATA_LIFECYCLE.md#tareas).

`endDate == null` describe un período vivo; no es una definición suficiente de elegibilidad para corregir hitos. Los consumidores deben resolver la identidad y posición del hito conforme al catálogo.

## 6. Reconstrucción del Timeline

Contrato observable: [Contenido del Timeline](rules/TIMELINE.md#contenido).

La lectura combina entidades, correlaciones y evidencia histórica de creación/cambio. El snapshot vigente sirve para estado y acciones; el snapshot de creación sirve para representar el hecho inicial. La paginación mantiene juntos el hito y su experiencia vinculada como un único acontecimiento lógico.

El orden técnico documentado es `event_date DESC, created_at DESC, item_type ASC, item_id ASC`. Su aplicación y pruebas pertenecen a API; no redefine el día civil ni la unidad de agrupación.

## 7. Matriz de acciones visibles en el Timeline

Contrato completo: [Acciones del Timeline](rules/TIMELINE.md#acciones), con elegibilidad de [Experiencias](rules/EXPERIENCES.md), [hitos y períodos](rules/RELATIONSHIPS.md#cambios) y bloqueos de [Guardado y sincronización](rules/SYNC.md). No se mantiene aquí otra tabla que pueda divergir.

## 8. Referencias

- [Arquitectura API](../../api/docs/ARCHITECTURE.md), [arquitectura iOS](../../ios/docs/ARCHITECTURE.md).
- [Ejecución relacional](plans/active/relationship-redesign/README.md).
- [Ejecución durable](plans/active/durable-write-feedback/README.md).
- [Paridad Android](plans/active/android-parity/README.md).
