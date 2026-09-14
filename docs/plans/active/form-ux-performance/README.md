# Fluidez y experiencia de formularios iOS

> **Estado:** implementación en curso. `IOS-FORM-01` a `IOS-FORM-03`, los tres pasos del alta de Crush, los pasos 1-2 y Review de edición de Crush, el alta/edición de Experience, el alta/edición standalone de Relationship y el ajuste visual de Vault están implementados con evidencia local. Una pasada de Simulator confirmó las rutas UI ya cubiertas y añadió foco Email → Password y bloqueo de doble Login. La auditoría de `IOS-FORM-05` confirmó que onboarding es informativo; Perfil usa sólo selectores y fecha; Feedback ya descarta el teclado al arrastrar; y la nota privada es un editor dedicado que toma foco. La baja de cuenta queda sólo para validación física por su flujo durable y sensible. Recovery de Vault ya tiene un seam exclusivo de UI testing que monta la ruta real y cubre contraseña y frase sin iniciar recuperación; su scroll no declara dismiss interactivo y eso permanece como comprobación física. Las validaciones físicas se agrupan para después. En Birthday aislado, si el guard existente exige confirmar edad, el CTA queda pulsable y muestra un mensaje para corregir los datos básicos, sin navegación automática; la lógica de edad queda señalada para revisión posterior. Review está localizado y no duplica WhatsApp; su guardado completo ya se prueba con una entrada Vault real del fixture UI. `EncounterDatePicker` ya precarga la fecha vinculada tanto al abrirse por toque como por encadenamiento lineal; confirmar sin elegir día conserva el día civil. Pasos 3-5 sólo requieren validación física. `IOS-FORM-06` sigue pendiente. Esta iniciativa no está lista para cierre.
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
- Etiquetas simples de acción en wizards y comportamiento explícito de la experiencia inicial opcional de un Crush.
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
| GWT-FORM-04 | Hay un formulario con teclado visible | Se enfoca un campo o se navega a un error | El campo relevante y el botón principal permanecen visibles simultáneamente, respetando safe areas, tamaño de pantalla y Dynamic Type. |
| GWT-FORM-05 | Un wizard tiene más de un campo editable lógico | La persona pulsa `Siguiente` en el teclado | El foco pasa al siguiente campo en orden visual. En el último campo, la tecla ejecuta la misma acción que el botón principal de la pantalla. |
| GWT-FORM-06 | Una pantalla de wizard aún no es terminal | Muestra su acción principal | La acción se llama `Siguiente`. En el último paso que persiste datos se llama `Guardar`. Los flujos de una sola acción conservan su verbo propio, por ejemplo iniciar sesión. |
| GWT-FORM-07 | El último paso de creación de Crush contiene la experiencia inicial opcional | Los tres campos de experiencia están vacíos y se pulsa `Guardar` | Se guarda el Crush sin experiencia inicial, según la regla existente de Product. |
| GWT-FORM-08 | La experiencia inicial opcional contiene cualquier valor | Se pulsa `Guardar` | Se aplican las reglas existentes de completitud; se marcan todos los campos inválidos y no se persiste una intención parcial. |
| GWT-FORM-09 | Una persona pulsa `Siguiente` o `Guardar` con datos inválidos | La revalidación local falla | El botón permanece disponible para intentar la corrección; se marcan todos los campos inválidos, se desplaza al primero en orden visual y recibe foco de entrada y de accesibilidad cuando el control lo admite. |
| GWT-FORM-10 | Un campo no editable por teclado es el primer inválido | Falla la validación | La pantalla se desplaza a ese control y anuncia/expone su estado accesible, sin abrir un teclado que no corresponde. |
| GWT-FORM-11 | Un formulario no se ha intentado enviar o avanzar | La persona comienza a editar | No se introducen errores visuales prematuros. Tras perder foco o tras el primer intento, el estado de validación se actualiza mientras corrige. |
| GWT-FORM-12 | Se inicia una operación de guardado o envío | La operación sigue en curso | La acción principal se deshabilita sólo durante esa operación para evitar doble envío; antes de ello se mantiene activa para poder validar y guiar. |

## Decisiones cerradas

- Al fallar una validación se marcan todos los campos inválidos y se dirige al primero en orden visual.
- El pie de acción debe quedar sobre el teclado con mecanismos nativos de layout, no calculando manualmente su altura.
- Los pasos intermedios usan `Siguiente`; el paso que persiste usa `Guardar`. La experiencia inicial opcional vacía usa simplemente `Guardar`, no un texto más largo.
- La tecla de retorno del último campo ejecuta la misma acción que el pie de la pantalla.
- El foco inicial espera al toque, salvo el editor de descripción a pantalla completa.
- `Siguiente` y `Guardar` se mantienen pulsables hasta que una operación real está en curso.
- La validación aparece al perder foco o al intentar avanzar/guardar y luego se actualiza durante la corrección.
- No se añaden por ahora textos visibles nuevos de error bajo los campos.
- En la entrada aislada de Birthday, cuando el guard existente requiere confirmar edad, el CTA permanece pulsable y muestra un mensaje que indica corregir los datos básicos; no navega automáticamente ni altera la regla de edad. La necesidad de ese guard se revisará fuera de este correctivo de UI.
- Un selector de fecha abierto programáticamente debe inicializar la selección temporal igual que al abrirse por toque: confirmar sin cambiar el día conserva la fecha civil ya vinculada, nunca la sustituye por hoy.

## Interacción con iniciativas activas

- [`Cambios pendientes visibles`](../durable-write-feedback/README.md) conserva la validación autoritativa antes de persistir y el orden durable. Esta iniciativa sólo mejora cómo la UI llega a esa frontera y muestra sus errores.
- [`Private Journal Vault`](../vault-e2ee/README.md) conserva sus garantías de Vault y recovery. `VaultRecoveryView` es una referencia técnica para foco/scroll; no se modifica su semántica de recuperación sin una fase explícita.
- [`Verificación de email`](../email-verification/README.md) y [`Sign in with Apple`](../sign-in-with-apple/README.md) deben adoptar el patrón común cuando implementen o modifiquen formularios propios, sin adelantar sus reglas de identidad, OTP o Legal.

## Orden y fases

1. **PRODUCT-FORM-01 — reglas compartidas:** esta sección congela los GWT y decisiones anteriores. Completada al crear este plan.
2. **IOS-FORM-01 — contenedor y baseline de fluidez:** retirar trabajo de teclado sin uso, garantizar un único scroll vertical por formulario y colocar correctamente el pie sobre el teclado.
3. **IOS-FORM-02 — contrato de foco y validación:** introducir la infraestructura reutilizable para orden de foco, `Siguiente`/`Guardar`, retorno, salto al primer inválido y foco de accesibilidad.
4. **IOS-FORM-03 — autenticación:** aplicar el patrón a login, registro y recuperación; estabilizar el toggle de contraseña y los tipos de contenido de iOS.
5. **IOS-FORM-04 — flujos estructurales:** aplicar el patrón a alta/edición de Crush, Experience y Relationship, incluida la experiencia inicial opcional, sin cambiar sus guards ni snapshots. Los tres pasos del alta y los pasos 1-2 y Review de edición están implementados; Birthday conserva su guard y ahora explica la corrección requerida, Review está localizado sin WhatsApp duplicado con guardado completo probado y `EncounterDatePicker` conserva su fecha vinculada también al abrirse por encadenamiento lineal. Pasos 3-5 no requieren código; sus validaciones físicas se agrupan para después.
6. **IOS-FORM-05 — formularios y editores restantes:** la auditoría confirmó que no queda un correctivo de interacción en onboarding, perfil, Feedback, nota privada ni baja de cuenta; esta última conserva sólo validación física por su flujo durable. Recovery de Vault ya es alcanzable bajo UI testing sin alterar su recuperación. Quedan las validaciones físicas agrupadas.
7. **IOS-FORM-06 — medición física y gate de pipeline:** medir en dispositivo real Release la apertura de teclado, scroll, guardado y cuerpos SwiftUI. Sólo si hay evidencia de bloqueo en el pipeline durable se propone una fase separada para su threading.

Cada fase iOS se ejecuta en una sesión independiente. No se avanza por iniciativa propia a la siguiente.

## Criterios de aceptación

- Los formularios de producción no recrean observadores de teclado sin usarlos y los formularios de flujo no contienen dos `ScrollView` verticales por accidente.
- Los flujos multi-campo permiten avanzar con el teclado y el último retorno tiene exactamente el mismo resultado que el botón principal.
- Una validación fallida marca todos los campos afectados, muestra el primero y nunca permite crear una operación/request a partir de datos inválidos.
- El botón principal y el campo relevante se mantienen visibles con teclado en pantallas pequeñas y con Dynamic Type grande.
- Password toggle conserva foco/cursor/teclado; email, contraseña, nombre y OTP usan los tipos de contenido apropiados cuando el campo corresponde.
- Tests unitarios, UI y validación física cubren las reglas anteriores sin debilitar la cobertura durable existente.
- La medición compara Debug/Release y con/sin debugger en dispositivo físico; los cambios de threading no se realizan por suposición.

## Riesgos y bloqueos

- Cambiar el orden de foco debe respetar controles opcionales, selectores y campos segmentados; el orden visual y el de validación deben ser una única fuente de verdad por formulario.
- El cambio del pie inferior debe verificarse en presentaciones push, sheets y full-screen covers, teclado externo, iPhone pequeño y Dynamic Type de accesibilidad.
- La corrección de contraseñas requiere regresión manual en login, registro y recuperación para preservar cursor, AutoFill y seguridad.
- La auditoría detectó trabajo síncrono potencialmente costoso en el pipeline durable. No se toca hasta medirlo y preservar explícitamente cifrar → persistir → proyectar.

## Estado de cierre

La iniciativa está parcialmente implementada. Antes del cierre queda auditar el inventario de perfil/ajustes/editores para identificar trabajo real, completar lo que resulte pendiente, ejecutar `IOS-FORM-06` y agrupar las validaciones físicas ya registradas. Al cerrarla, las reglas estables se integrarán en [`PRODUCT.md`](../../../PRODUCT.md) y el snapshot técnico en [`ios/docs/ARCHITECTURE.md`](../../../../../ios/docs/ARCHITECTURE.md); entonces se eliminarán ambos planes de esta iniciativa.
