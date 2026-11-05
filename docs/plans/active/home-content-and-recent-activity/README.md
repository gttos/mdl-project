# Contenido y actividad reciente de Home

<a id="objetivo"></a><a id="alcance"></a><a id="contrato-compartido"></a><a id="interacción-entre-proyectos"></a><a id="orden-y-fases"></a><a id="ajuste-autorizado-de-consejos"></a><a id="criterios-de-cierre"></a><a id="decisiones-abiertas"></a><a id="planes-locales"></a>

> **Estado:** IOS-HOME-01 y API-HOME-01 aceptadas; IOS-HOME-02 con implementación aceptada y validación manual parcial.

## Objetivo y alcance

Presentar los cinco consejos diarios y la actividad reciente con Crush, lugar, fecha funcional y situación resultante cuando existe cambio relacional. Nombre y lugar se resuelven únicamente en el diario desbloqueado. Cada creación compuesta genera una sola actividad.

Reglas: [RP-HOM-001..007](../../../rules/TIMELINE.md#home).

## Escenarios de aceptación

<a id="gwt-home-01--consejos-activos"></a>
### GWT-HOME-01 — Consejos activos

**Dado** que la aplicación recibe consejos de cualquiera de las cinco categorías
**Cuando** la persona consulta los consejos
**Entonces** se pueden consultar el título y el consejo recibidos, sin contenido temporal.

<a id="gwt-home-02--experiencia-independiente"></a>
### GWT-HOME-02 — Experiencia independiente

**Dado** un evento de una Experience sin hito relacional asociado
**Cuando** API e iOS construyen la actividad reciente
**Entonces** se muestran el lugar, el nombre del Crush y la fecha funcional, sin situación ni satisfacción.

<a id="gwt-home-03--cambio-dentro-de-un-período-vivo"></a>
### GWT-HOME-03 — Cambio dentro de un período vivo

**Dado** una Experience que creó un nuevo hito dentro del período vivo
**Cuando** se presenta su única actividad
**Entonces** se muestran lugar, Crush, fecha y situación resultante. El resultado se identifica como una experiencia con cambio relacional.

<a id="gwt-home-04--inicio-de-un-período-posterior"></a>
### GWT-HOME-04 — Inicio de un período posterior

**Dado** una Experience que abrió un nuevo período con su hito inicial
**Cuando** se presenta su única actividad
**Entonces** conserva las asociaciones a Experience, período e hito y identifica el resultado compuesto con lugar, Crush, fecha y situación resultante.

<a id="gwt-home-05--alta-de-crush"></a>
### GWT-HOME-05 — Alta de Crush

**Dado** el alta confirmada de un Crush
**Cuando** aparece en actividad reciente
**Entonces** muestra el nombre resuelto desde el Vault y la fecha de encuentro como fecha funcional, sin crear una segunda actividad por el período inicial.

<a id="gwt-home-06--privacidad-y-navegación"></a>
### GWT-HOME-06 — Privacidad y navegación

**Dado** cualquier actividad reciente
**Cuando** se transporta y presenta
**Entonces** API sólo entrega datos estructurales no privados, iOS resuelve los campos privados localmente y una Experience abre su Crush mediante la identidad de su Crush propietario.

## Fases y validación vigente

| Fase | Estado |
|---|---|
| IOS-HOME-01 | Aceptada; cinco consejos consultables. |
| API-HOME-01 | Aceptada; fecha y asociaciones relacionales sin duplicar actividad. |
| IOS-HOME-02 | Implementación aceptada con revisión manual parcial: alta y experiencia compuesta comprobadas; faltan experiencia independiente, textos largos, ambos idiomas y apertura del Crush correcto. La validación específica de VoiceOver queda excluida del cierre por RP-UX-006. |

Las revisiones editoriales de consejos y países y sus comprobaciones permanecen en los planes locales. El catálogo activo conserva 25 consejos bilingües y cinco categorías; el contenido aprobado está en la integración local y su publicación en producción permanece fuera de esta entrega.

## Dependencias y criterios de salida

- Validar los casos de consejos, experiencia independiente, cambio relacional, período posterior, alta de Crush y privacidad/navegación.
- Conservar las validaciones y criterios de presentación de los planes locales.
- Completar las comprobaciones pendientes antes del cierre documental coordinado.
- [Plan API](../../../../../api/docs/plans/active/home-content-and-recent-activity/README.md).
- [Plan iOS](../../../../../ios/docs/plans/active/home-content-and-recent-activity/README.md).
