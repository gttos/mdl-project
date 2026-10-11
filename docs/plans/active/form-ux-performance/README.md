# Fluidez y experiencia de formularios iOS

> **Estado:** implementación en curso. `IOS-FORM-05` queda como implementación aceptada con validaciones pendientes: el paso Legal mantiene la acción principal disponible hasta una operación real, marca los consentimientos pendientes sin enviar, conserva las versiones aceptadas y no añade mensajes visibles nuevos. La evidencia automatizada reportada incluye 2.215 pruebas unitarias y 22 UI tests de autenticación en verde. Los recorridos UI de Crush/Experience bloqueados por el arnés, el recorrido UI de Apple y las comprobaciones físicas de teclado se agrupan en `IOS-FORM-06`; lectores de pantalla y texto ampliado quedan fuera por `RP-UX-006`.
>
> **Proyecto afectado:** iOS. Product fija las reglas de interacción; no hay cambio de contrato, persistencia ni trabajo API.

## Objetivo

Hacer que los formularios de producción se sientan inmediatos y predecibles: una persona puede completar una pantalla sin ocultar el teclado para descubrir qué falta, sin buscar manualmente un error y sin que el botón principal quede tapado.

La iniciativa corrige costes de renderizado compartidos confirmados por la auditoría y establece un patrón único para foco, navegación de teclado, validación, desplazamiento y acción principal. No modifica las reglas de negocio ni relaja la revalidación autoritativa previa a una escritura durable.

## Alcance

- Formularios y editores de texto de producción de iOS: autenticación/registro/recuperación, alta y edición de Crush, Experience y Relationship, onboarding, perfil/ajustes y superficies Vault que admiten edición.
- Componentes compartidos que gobiernan esos flujos, en especial `FlowBaseView`, `MDLBaseView`, campos de texto y el pie de acción.
- Eliminación de trabajo de teclado sin uso, un único scroll vertical por formulario, desplazamiento que puede ocultar el teclado y pie principal visible sobre él.
- Navegación secuencial de campos, validación al intentar avanzar y salto al primer campo inválido.
- Etiquetas simples de acción en wizards y validación explícita de la experiencia inicial obligatoria de un Crush.
- Autofill/tipos de teclado, continuidad del campo de contraseña al mostrar/ocultar y validación física de rendimiento.

## Fuera de alcance

- Cambiar reglas funcionales de validación, campos obligatorios, fechas civiles, edad, estados de relación, durabilidad o retry.
- API, contratos HTTP, Vault, criptografía o almacenamiento persistente.
- Mover resolución/cifrado del pipeline durable fuera de `MainActor` sin una medición física que demuestre que produce un bloqueo perceptible y una fase posterior autorizada.
- Añadir ahora textos visibles nuevos bajo cada campo para describir un error. Los mensajes específicos que ya exijan reglas vigentes se conservan; esta iniciativa no los sustituye ni amplía.
- Rediseño visual general, cambios de copy no vinculados al flujo, Android o web.

## Reglas compartidas

| ID | Dado | Cuando | Entonces |
|---|---|---|---|
| GWT-FORM-01 | Una pantalla de formulario normal se presenta | Aún no se tocó ningún campo | No se abre el teclado ni se toma foco automáticamente. |
| GWT-FORM-02 | Se abre un editor dedicado de descripción a pantalla completa | El editor queda visible | El área de texto recibe foco y el teclado se abre para permitir escribir de inmediato. |
| GWT-FORM-03 | Una persona edita un formulario | Toca una zona vacía o arrastra el contenido | El toque vacío puede ocultar el teclado y el scroll lo descarta de forma interactiva sin bloquear controles reales. |
| GWT-FORM-04 | Hay un formulario con teclado visible | Se enfoca un campo o se navega a un error | El campo relevante y el botón principal permanecen visibles simultáneamente, respetando safe areas y los tamaños de pantalla soportados con texto normal. |
| GWT-FORM-05 | Un wizard tiene más de un campo editable lógico | La persona pulsa `Siguiente` en el teclado | El foco pasa al siguiente campo en orden visual. En el último campo, la tecla ejecuta la misma acción que el botón principal de la pantalla. |
| GWT-FORM-06 | Una pantalla de wizard aún no es terminal | Muestra su acción principal | La acción se llama `Siguiente`. En el último paso que persiste datos se llama `Guardar`. Los flujos de una sola acción conservan su verbo propio, por ejemplo iniciar sesión. |
| GWT-FORM-07 | El último paso de creación de Crush contiene la experiencia inicial obligatoria | Falta lugar, descripción o satisfacción y se pulsa `Guardar` | Se marcan todos los campos inválidos y no se crea snapshot, operación, proyección ni request. |
| GWT-FORM-08 | La experiencia inicial contiene todos sus campos válidos | Se pulsa `Guardar` | Se persiste una única intención compuesta con Crush, período, hito y Experience inicial en `met_at`. |
| GWT-FORM-09 | Una persona pulsa `Siguiente` o `Guardar` con datos inválidos | La revalidación local falla | El botón permanece disponible para intentar la corrección; se marcan todos los campos inválidos, se desplaza al primero en orden visual y recibe foco de entrada y de accesibilidad cuando el control lo admite. |
| GWT-FORM-10 | Un campo no editable por teclado es el primer inválido | Falla la validación | La pantalla se desplaza a ese control y anuncia/expone su estado accesible, sin abrir un teclado que no corresponde. |
| GWT-FORM-11 | Un formulario no se ha intentado enviar o avanzar | La persona comienza a editar | No se introducen errores visuales prematuros. Tras perder foco o tras el primer intento, el estado de validación se actualiza mientras corrige. |
| GWT-FORM-12 | Se inicia una operación de guardado o envío | La operación sigue en curso | La acción principal se deshabilita sólo durante esa operación para evitar doble envío; antes de ello se mantiene activa para poder validar y guiar. |
| GWT-FORM-13 | Existe un período vivo | La persona abre «Añadir experiencia» | Completa la Experience en un único paso; «¿Hay un cambio de situación?» añade un segundo paso sólo al activarse. |
| GWT-FORM-14 | El período más reciente está cerrado | La persona abre «Añadir experiencia» | El flujo tiene dos pasos obligatorios en orden Experience → situación inicial y no muestra una elección opcional sobre el cambio. |
| GWT-FORM-15 | La persona activa «¿Hay un cambio de situación?» dentro de «Añadir experiencia» | Recorre el formulario | Completa primero la Experience y después la situación o cierre; no existe una entrada con el orden inverso. |
| GWT-FORM-16 | «Añadir experiencia» conoce el período, sus hitos y Experiences confirmados | Presenta el selector de fecha | Aplica el mínimo del recorrido de RP-EXP-011 y hoy como máximo; si el recorrido o destino estrecha el intervalo, no sustituye silenciosamente la fecha ya elegida. |
| GWT-FORM-17 | El mínimo cronológico calculado queda después de hoy | Se abre o actualiza el selector | No ofrece una fecha ficticia ni permite guardar; explica que todavía no existe un día válido. |

## Decisiones cerradas

- Al fallar una validación se marcan todos los campos inválidos y se dirige al primero en orden visual.
- El pie de acción debe quedar sobre el teclado con mecanismos nativos de layout, no calculando manualmente su altura.
- Los pasos intermedios usan `Siguiente`; el paso que persiste usa `Guardar`. «Añadir experiencia» usa siempre Experience → situación cuando el segundo paso se añade o es obligatorio. No existe otro punto de entrada ni un orden visual inverso.
- La tecla de retorno del último campo ejecuta la misma acción que el pie de la pantalla.
- El foco inicial espera al toque, salvo el editor de descripción a pantalla completa.
- `Siguiente` y `Guardar` se mantienen pulsables hasta que una operación real está en curso.
- La validación aparece al perder foco o al intentar avanzar/guardar y luego se actualiza durante la corrección.
- No se añaden por ahora textos visibles nuevos de error bajo los campos.
- En la entrada aislada de Birthday, cuando el guard existente requiere confirmar edad, el CTA permanece pulsable y muestra un mensaje que indica corregir los datos básicos; no navega automáticamente ni altera la regla de edad. La necesidad de ese guard se revisará fuera de este correctivo de UI.
- Un selector de fecha abierto programáticamente debe inicializar la selección temporal igual que al abrirse por toque: confirmar sin cambiar el día conserva la fecha civil ya vinculada, nunca la sustituye por hoy.
- Los selectores aplican los límites cronológicos continuos ya confirmados por el dominio. Un cambio de opción puede invalidar el día actual, pero nunca corregirlo silenciosamente.

## Interacción con iniciativas activas

- [Guardado y sincronización](../../../rules/SYNC.md) conserva la validación autoritativa antes de persistir y el orden durable. Esta iniciativa sólo mejora cómo la UI llega a esa frontera y muestra sus errores.
- [`Private Journal Vault`](../vault-e2ee/README.md) conserva sus garantías de Vault y recovery. `VaultRecoveryView` es una referencia técnica para foco/scroll; no se modifica su semántica de recuperación sin una fase explícita.
- [Verificación de email](../../../rules/ACCOUNT.md#verificacion-email) y [`Sign in with Apple`](../sign-in-with-apple/README.md) deben adoptar el patrón común cuando implementen o modifiquen formularios propios, sin adelantar sus reglas de identidad, OTP o Legal.

## Orden y fases

1. **PRODUCT-FORM-01 — reglas compartidas:** esta sección congela los GWT y decisiones anteriores. Completada al crear este plan.
2. **IOS-FORM-01 — contenedor y baseline de fluidez:** retirar trabajo de teclado sin uso, garantizar un único scroll vertical por formulario y colocar correctamente el pie sobre el teclado.
3. **IOS-FORM-02 — contrato de foco y validación:** introducir la infraestructura reutilizable para orden de foco, `Siguiente`/`Guardar`, retorno, salto al primer inválido y foco de accesibilidad.
4. **IOS-FORM-03 — autenticación:** aplicar el patrón a login, registro y recuperación; estabilizar el toggle de contraseña y los tipos de contenido de iOS.
5. **IOS-FORM-04 — flujos estructurales:** la infraestructura lineal existente se preserva. Los guards y snapshots de Experience inicial obligatoria y el recorrido único Experience → Situación —opcional u obligatorio según el período— ya están integrados en el contrato final. Las futuras validaciones físicas de formulario se ejecutan sobre ese recorrido, no sobre las pantallas standalone retiradas.
6. **IOS-FORM-05 — formularios y editores restantes:** implementación aceptada con validaciones pendientes. El CTA de consentimientos cumple `RP-FOR-006`, el intento inválido mantiene el flujo en Legal y expone el estado sin mensajes visibles nuevos, y la regresión automatizada está verde. No queda otro correctivo de código identificado en esta fase.
7. **IOS-FORM-06 — validación física, medición y gate de pipeline:** recuperar los recorridos UI de Crush/Experience bloqueados por el arnés; comprobar el recorrido Apple y, en dispositivo real Release, teclado, apertura, scroll, guardado y cuerpos SwiftUI con texto normal. No incluye lectores de pantalla ni texto ampliado conforme a `RP-UX-006`. Sólo si hay evidencia de bloqueo en el pipeline durable se propone una fase separada para su threading.

Cada fase iOS se ejecuta en una sesión independiente. No se avanza por iniciativa propia a la siguiente.

## Criterios de aceptación

- Los formularios de producción no recrean observadores de teclado sin usarlos y los formularios de flujo no contienen dos `ScrollView` verticales por accidente.
- Los flujos multi-campo permiten avanzar con el teclado y el último retorno tiene exactamente el mismo resultado que el botón principal.
- Una validación fallida marca todos los campos afectados, muestra el primero y nunca permite crear una operación/request a partir de datos inválidos.
- El botón principal y el campo relevante se mantienen visibles con teclado en las pantallas soportadas y con texto normal.
- Password toggle conserva foco/cursor/teclado; email, contraseña, nombre y OTP usan los tipos de contenido apropiados cuando el campo corresponde.
- Tests unitarios, UI y validación física cubren las reglas anteriores sin debilitar la cobertura durable existente.
- La medición compara Debug/Release y con/sin debugger en dispositivo físico; los cambios de threading no se realizan por suposición.

## Riesgos y bloqueos

- Cambiar el orden de foco debe respetar controles opcionales, selectores y campos segmentados; el orden visual y el de validación deben ser una única fuente de verdad por formulario.
- El cambio del pie inferior debe verificarse en presentaciones push, sheets y full-screen covers, teclado externo e iPhone pequeño con texto normal.
- La corrección de contraseñas requiere regresión manual en login, registro y recuperación para preservar cursor, AutoFill y seguridad.
- La auditoría detectó trabajo síncrono potencialmente costoso en el pipeline durable. No se toca hasta medirlo y preservar explícitamente cifrar → persistir → proyectar.

## Estado de cierre

La implementación de formularios está aceptada. Antes del cierre queda ejecutar `IOS-FORM-06`: recuperar los recorridos UI bloqueados, completar las validaciones físicas y medir el pipeline en dispositivo real Release. Al cerrarla, las reglas estables se integrarán en [`PRODUCT.md`](../../../PRODUCT.md) y el snapshot técnico en [`ios/docs/ARCHITECTURE.md`](../../../../../ios/docs/ARCHITECTURE.md); entonces se eliminarán ambos planes de esta iniciativa.
