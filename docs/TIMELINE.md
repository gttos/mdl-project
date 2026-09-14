# Timeline y ciclo de vida de Crush / Experience / Relationship

Fuente única de las reglas de dominio para las tres entidades del diario y su reconstrucción en el Timeline. `PRODUCT.md` §4, §6, §7 y §8 dan el resumen orientado a producto; este documento tiene la profundidad completa (matrices, identidad, casos límite) para que API y los clientes nativos implementen exactamente lo mismo sin redefinir nada por su cuenta. Verificado contra el código a 2026-08-27.

## 1. Las entidades

MDL tiene 4 entidades reales: **User**, **Crush**, **Experience**, **Relationship**. Nada más — no hay una quinta entidad de dominio.

| Entidad | Identidad | Campos obligatorios | Notas |
|---|---|---|---|
| `Crush` | UUID funcional, generado por el cliente antes del request | `name`, `genderId`, `duckyId`, `countryId`, `metAt`, edad — ver `PRODUCT.md` §4.1 (BR-4.01) | 7 campos PII (email, whatsapp prefix/número, instagram, facebook, snapchat, tiktok) viven sólo en el Vault, nunca en columnas funcionales. |
| `Experience` | UUID funcional | `crushId`, `date`, `satisfactionId` | `place`/`description` viven en el Vault. |
| `Relationship` | UUID funcional | `crushId`, `stateId`, `startDate` | `closureNote` vive en el Vault. |

`zodiacId` es un passthrough: el cliente lo calcula desde día/mes (guardados sólo en el Vault) y lo envía ya calculado; la API nunca recibe ni calcula día/mes/fecha de nacimiento.

## 2. Alta — el bundle inicial

Crear un crush **siempre** crea, en la misma transacción, su bundle inicial: el crush, la relación inicial obligatoria, y opcionalmente la experiencia inicial. Los tres comparten el mismo `correlation_id` inicial (asignado por la API).

- El cliente puede aportar los UUID (`id` del crush, `first_experience.id`, `initial_relationship.id`); si los aporta, la API los usa en vez de generarlos. Si el `id` de crush ya existe para ese usuario, la API responde `200 OK` con body vacío (idempotencia) — no crea un duplicado.
- La relación inicial es obligatoria: `startDate` siempre igual a `crush.metAt`. Nunca se elimina.
- La experiencia inicial es opcional y todo-o-nada: si se completa cualquiera de lugar/descripción/satisfacción, los tres pasan a ser obligatorios. Su fecha, si existe, siempre igual a `crush.metAt`.
- Validación final: el flujo bloquea el submit hasta que los pasos previos estén completos, y **revalida en el límite de envío** (justo antes de cifrar/persistir) — el estado del botón deshabilitado en la UI es una ayuda de UX, no el límite autoritativo.

## 3. Cronología y unicidad por día civil

- `crush.metAt` es el límite cronológico mínimo del crush: ninguna relación ni experiencia puede existir antes.
- Sólo puede existir **una experiencia** por crush y día civil, y **un cambio de relación** por crush y día civil (comparación por día civil local, nunca por timestamp exacto).
- Actualizar `crush.metAt` se rechaza si dejaría alguna relación o experiencia existente antes de la nueva fecha. Si se acepta, la relación inicial y la experiencia inicial (si existe) se sincronizan a la nueva fecha en la misma transacción.
- No se puede insertar una relación en medio de una cronología ya establecida — sólo se permite extender el extremo más reciente.

### Reasociación retroactiva a la experiencia inicial

Si el crush no tiene experiencia inicial todavía, una experiencia normal (no vinculada a otra relación) creada o editada exactamente en `crush.metAt` se reasocia automáticamente al bundle inicial: adopta la `correlation_id` inicial y conserva su propio UUID. Esto es comportamiento de producto explícito, no una conversión accidental. Si el slot inicial ya está ocupado, otra experiencia no puede moverse a ese día — se lo impide la regla de unicidad diaria.

Una experiencia vinculada directamente a una relación (comparte fecha y `correlation_id` con ella) tiene su fecha bloqueada a la de esa relación; no puede reasociarse independientemente al bundle inicial.

### Coincidencia entre experiencia y relación

Una relación standalone puede tener como máximo una experiencia vinculada. Si una experiencia normal no vinculada se crea o se mueve exactamente a su día civil y el slot está libre, la experiencia conserva su UUID, adopta el `correlation_id` de la relación y ambas se reconstruyen como una sola tarjeta vinculada. La misma invariante aplica cuando el flujo aprobado crea una relación desde una experiencia existente.

Si la relación ya tiene otra experiencia, la operación se rechaza sin mutar ninguna de las dos. La beta no ofrece «Desvincular»: la fecha de la experiencia vinculada queda bloqueada. Borrar la experiencia elimina el vínculo y conserva la relación standalone; si la persona quiere conservar el relato en otra fecha, debe crear una nueva experiencia después del borrado.

## 4. Relaciones — matriz de transición de estados

Los 8 estados y sus transiciones permitidas (verificado contra `RelationshipStateTransitionValidator.php`):

| Desde | Hacia |
|---|---|
| Prospecto (1) | Ligue (2), One-night-stand (3), FWB (4), Rechazado (6), No hubo match (7) |
| Ligue (2) | One-night-stand (3), FWB (4), Relación (5), Rechazado (6), No hubo match (7) |
| One-night-stand (3) | FWB (4) |
| FWB (4) | Relación (5), Terminada (8) |
| Relación (5) | FWB (4), Terminada (8) |
| Rechazado (6) | Ligue (2), One-night-stand (3), FWB (4) |
| No hubo match (7) | Ligue (2), One-night-stand (3), FWB (4) |
| Terminada (8) | Ligue (2), FWB (4), Relación (5) |

Reglas:

- La relación inicial (mandatory first) no tiene estado previo — se le permiten todos los estados sin restricción de matriz. Toda relación posterior sólo puede pasar a un estado listado como destino permitido desde su estado inmediatamente anterior en la cronología del crush (nunca contra una relación no adyacente ni contra la posición visual en el timeline).
- Los mismos criterios de transición aplican en creación y en edición — no pueden divergir.
- Sólo la relación **latest** (`endDate == null`) es mutable. Cualquier intento de editar/borrar una relación histórica se rechaza.
- Crear una nueva relación cierra automáticamente la activa (`endDate` = `startDate` de la nueva).
- Borrar la relación latest reabre la inmediatamente anterior (`endDate = null`). Nunca hay más de una relación abierta a la vez.

### Match status

Se deriva del estado de relación, con monotonicidad (una vez Sí, nunca retrocede):

| Estado de relación | Match status |
|---|---|
| Prospecto, Ligue | Todavía no se sabe |
| One-night-stand, FWB, Relación | Sí |
| Rechazado, No hubo match | No |
| Terminada | Preserva el match status que ya tenía (no se deriva de nuevo desde "Terminada" en sí) |

Matriz de monotonicidad: desde Sí, cualquier derivación nueva se ignora y se queda en Sí. Desde No, una derivación puede subir a Sí pero nunca a "todavía no se sabe". Desde "todavía no se sabe", se adopta lo que derive.

## 5. Edición y borrado

- **Experience**: siempre editable y siempre borrable, sin importar antigüedad ni si está vinculada a una relación. Borrar una experiencia vinculada sólo quita el vínculo — la relación permanece con su mismo estado y fechas, y no se reabre ninguna relación anterior. Borrar la experiencia inicial libera el slot inicial.
- **Relationship**: sólo la latest es editable. Para borrar debe ser, además, una relación posterior: la relación inicial nunca se elimina aunque sea latest. El resto queda inmutable frente a mutaciones de dominio — no se ofrecen ni edición ni borrado en la interfaz para relaciones históricas.
- **Crush — borrado manual vs. anonimización**, distinción estricta que nunca se mezcla:
  - **Borrado manual** (`deleted_at != null`, `is_anonymized = false`): saca al crush del historial activo. Cascada de soft-delete a experiencias y relaciones.
  - **Anonimización** (`deleted_at = null`, `is_anonymized = true`): preserva el registro para estadísticas agregadas, sobrescribe irreversiblemente el PII con `ANON_` + aleatorio, y borra los `ActivityEvent` que referencian al crush (para no dejar rastro en el feed). Ocurre al eliminar la cuenta, nunca junto con un borrado manual del mismo registro.
- Un crush anonimizado queda oculto de toda consulta estándar vía `AnonymizedScope`; las consultas manuales (`DB::table`) deben aplicar el mismo filtro a mano.

## 6. Reconstrucción del Timeline

El Timeline no es una tabla — se reconstruye uniendo Crush/Relationship/Experience en un solo stream ordenado.

- Cada tipo de item tiene una fecha canónica para ordenar: bundle → `crush.metAt`; relación → `startDate`; experiencia → `date`.
- Los items que comparten `correlation_id` y la misma fecha se agrupan en un único item **bundle** (crush + relación inicial + experiencia opcional). Si una experiencia se asocia retroactivamente al bundle inicial, el Timeline la debe mostrar integrada al bundle, nunca como tarjeta separada simultáneamente.
- El contexto de relación de una experiencia standalone se resuelve dinámicamente contra la cronología, nunca contra una columna denormalizada de estado histórico.
- Orden determinista incluso con empates de fecha — necesario para paginación estable.
- Privacidad: la reconstrucción excluye siempre entidades de crushes anonimizados o soft-deleted, incluida cualquier consulta de bajo nivel que no herede los scopes de Eloquent automáticamente.
- Un item con contexto insuficiente para mutación segura (falta un ID, falta `endDate`, etc.) debe degradarse a solo-lectura en el cliente, no arriesgar una acción con datos incompletos.

## 7. Matriz de acciones visibles en el Timeline

Por tipo de item, qué opciones ofrece el menú contextual:

**Bundle** (crush + relación inicial + experiencia opcional):
- "Editar crush": siempre presente.
- Si tiene experiencia: "Editar experiencia" y "Eliminar experiencia".
- Si no tiene experiencia: "Agregar experiencia" o "Vincular experiencia".
- Si la relación del bundle es la latest (`endDate == null`): "Editar relación".
- Si la relación del bundle es histórica: sin edición, opción completamente oculta (no deshabilitada — oculta).
- Caso especial: si la relación editable del bundle es la inicial, se muestran todos los estados posibles sin filtrar por relación previa (no tiene una).

**Experiencia standalone:**
- Siempre: "Editar" y "Eliminar" — el borrado de experiencia es independiente de si la relación asociada es la latest.
- "Agregar relación" sólo se muestra si no existe una relación posterior a la fecha de la experiencia. Al usarla, la fecha de la nueva relación queda prellenada y bloqueada a la fecha de la experiencia.

**Relación standalone:**
- "Editar relación": sólo si es latest (`endDate == null`); si es histórica, completamente oculta.
- "Eliminar relación": sólo si es latest; si es histórica, completamente oculta.
- "Vincular experiencia": siempre disponible. Prellenar y bloquear la fecha de la nueva experiencia a la fecha de esa relación.

**Reglas transversales:**
- `endDate == null` es el **único** determinante de si una relación es mutable — nunca se usan heurísticas de "se puede vincular" para bloquear edición.
- Una escritura de Timeline pendiente en el mismo crush no oculta el borrado de una experiencia confirmada — puede ser precisamente la acción que resuelve un conflicto de mismo-día.
- Una edición de Profile pendiente que cambia realmente `met_at` impide únicamente iniciar una experiencia nueva de ese mismo crush hasta resolverse. Si no cambia `met_at`, no bloquea Timeline; el cliente no intenta comparar una escritura nueva contra dos fechas hipotéticas.
- Límites de fecha al editar: una experiencia normal con slot inicial ya ocupado tiene mínimo `crush.metAt + 1 día`; con el slot libre, `crush.metAt` es seleccionable (y guardar ahí reasocia al bundle). Una relación tiene mínimo `fecha de la relación anterior + 1 día` (o `crush.metAt + 1 día` si la anterior es la inicial). Máximo siempre `HOY`.

## 8. Referencias

- Resumen de producto: [`PRODUCT.md`](PRODUCT.md) §4 (crush), §6 (experiencias), §7 (relaciones), §8 (timeline).
- Escenarios Dado/Cuando/Entonces que demuestran estas reglas durante la iniciativa de sincronización visible: [`plans/active/durable-write-feedback/README.md`](plans/active/durable-write-feedback/README.md) (GWT-01 a GWT-42). Este documento conserva la regla estable cuando ese plan desaparezca.
- Implementación API: `api/docs/ARCHITECTURE.md` §5.
- Implementación iOS: `ios/docs/ARCHITECTURE.md` §4-7, §9-11.
- Incorporación Android: `product/docs/plans/active/android-parity/README.md` y `android/docs/plans/active/android-parity/README.md`.
