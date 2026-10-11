# Reglas de producto — Uso de la aplicación

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Tutoriales informativos, formularios, mensajes, errores, accesibilidad, carga, vacíos y desconexión.

<a id="tutoriales"></a>
## Tutoriales y onboarding informativo

- **RP-ONB-001.** El tutorial inicial y la explicación ampliada al crear el primer Crush son informativos. Sus pantallas no solicitan ni conservan datos del perfil de la cuenta o del diario.
- **RP-ONB-002.** Avanzar y retroceder conserva el paso alcanzado y las elecciones realizadas dentro del tutorial.
- **RP-ONB-003.** El tutorial se marca como completado una sola vez al terminar su último paso.
- **RP-ONB-004.** Si no puede confirmarse la finalización, el tutorial permanece pendiente y la persona puede volver a intentarlo.

<a id="mensajes"></a>
## Mensajes, errores y accesibilidad

- **RP-UX-001.** Los errores se expresan con lenguaje comprensible de producto. La interfaz nunca muestra claves internas, SQL, trazas ni mensajes libres de excepciones.
- **RP-UX-002.** Toda acción visible produce una navegación, una confirmación, un resultado o un error comprensible.
- **RP-UX-003.** Una acción deshabilitada tiene bloqueo funcional, visual y accesible. Cuando la persona puede resolver la causa, la interfaz la explica.
- **RP-UX-004.** Cancelar conserva el estado confirmado y limpia cualquier confirmación temporal antes de la acción siguiente.
- **RP-UX-005.** Cada estado de sincronización se comunica mediante texto o etiqueta accesible además de cualquier color o icono.
- **RP-UX-006.** Español e inglés permiten acceder al mismo contenido y ejecutar las mismas acciones. La adaptación específica para lectores de pantalla —incluidos VoiceOver y TalkBack— y para tamaños de texto ampliados —incluidos Dynamic Type y escalas de fuente equivalentes— queda fuera del alcance actual del producto. Se conserva lo ya implementado y probado, pero durante este alcance no se abren auditorías, correctivos, validaciones manuales, gates de aceptación ni fases dedicadas a esas capacidades. Sólo una nueva decisión explícita de Producto puede incorporarlas a un alcance futuro.
- **RP-UX-007.** Carga, error, ausencia de datos y contenido son cuatro estados distintos. Un error de red no se presenta como una cuenta vacía.
- **RP-UX-008.** Una capacidad que Producto define como activa u obligatoria no se considera ausente por un `404`, una respuesta desconocida, un flag o campo omitido ni por compatibilidad con un backend anterior. La interfaz conserva el último estado confirmado y muestra un error comprensible con reintento. Si esa capacidad decide acceso, seguridad o una acción protegida, la aplicación falla cerrada: no muestra datos privados ni ejecuta la acción hasta resolver el estado. Sólo una ausencia de recurso definida expresamente por la regla del dominio o una capacidad real del dispositivo/plataforma puede cambiar la presentación sin tratarse como error.

<a id="formularios"></a>
## Formularios

- **RP-FOR-001.** Un formulario ordinario se abre sin enfocar automáticamente un campo ni mostrar el teclado. Un editor dedicado de descripción puede abrirse con el campo listo para escribir.
- **RP-FOR-002.** Tocar el fondo permite ocultar el teclado y desplazar el formulario puede descartarlo interactivamente, sin bloquear controles.
- **RP-FOR-003.** El campo activo y la acción principal permanecen utilizables con el teclado abierto y en las pantallas soportadas con el tamaño de texto normal. La adaptación a texto ampliado se rige por `RP-UX-006` y no forma parte del alcance actual.
- **RP-FOR-004.** «Siguiente» recorre los campos en su orden visual. En el último campo, la acción del teclado ejecuta la misma validación que la acción principal.
- **RP-FOR-005.** Los pasos intermedios utilizan «Siguiente» y el paso final utiliza «Guardar». Una acción única conserva el verbo que describe su efecto.
- **RP-FOR-006.** La acción principal permite intentar la validación. Sólo se bloquea temporalmente mientras una operación real evita un doble envío.
- **RP-FOR-007.** Un intento inválido marca todos los campos afectados y lleva el foco o desplazamiento al primero. Un selector no abre un teclado que no corresponde a su tipo de dato.
- **RP-FOR-008.** El formulario no muestra errores antes de que la persona interactúe. Valida al perder el foco o al intentar continuar y actualiza el error al corregir el valor.
- **RP-FOR-009.** Antes de cifrar, guardar localmente o enviar, se vuelven a validar todos los campos y reglas del formulario completo.
- **RP-FOR-010.** Mostrar u ocultar una contraseña conserva foco, cursor y teclado, y utiliza el tipo de campo y autocompletado correspondientes.
- **RP-FOR-011.** El selector de fecha conserva y precarga el mismo día tanto si se abre por toque como automáticamente. Confirmar sin cambios no sustituye el valor por la fecha de hoy.
- **RP-FOR-012.** Cuando un cumpleaños exige volver a confirmar una edad, la interfaz explica qué dato básico debe corregirse y ofrece una acción funcional para hacerlo.
- **RP-FOR-013.** Cuando el dominio determina con estado confirmado un intervalo cronológico continuo para una fecha, el selector aplica ese mínimo y máximo en vez de ofrecer fechas que necesariamente se rechazarán por quedar fuera del intervalo. Si otra elección del mismo formulario estrecha el rango, la aplicación no cambia silenciosamente el día ya elegido: lo marca inválido y guía su corrección. Cuando el intervalo no contiene ningún día disponible, muestra el bloqueo y su causa. La limitación visual no sustituye la revalidación completa antes de persistir ni la validación autoritativa del servidor.

<a id="vacios"></a>
## Carga, vacíos y desconexión

- **RP-VAC-001.** Si una actualización falla, la pantalla conserva el contenido confirmado y muestra un aviso no bloqueante. Un vacío confirmado por una carga exitosa también es contenido confirmado: una actualización fallida conserva ese vacío y añade el aviso, sin sustituirlo por el error de primera carga. Si la primera carga falla y no existe contenido confirmado, muestra el error y una acción para reintentar.
- **RP-VAC-002.** Un estado vacío sólo se muestra después de confirmar que no hay datos. Mientras un error impida saberlo, se muestra el estado de error.
- **RP-VAC-003.** Toda pantalla principal de la beta presenta contenido, carga, error o un estado vacío con una salida clara.
- **RP-VAC-004.** Cada estado vacío de la tabla siguiente explica qué falta y ofrece la acción indicada. Si esa acción no está disponible por una regla del dominio, explica el bloqueo en vez de mostrar un control inerte.

| Vacío | Acción prevista |
|---|---|
| Lista sin crushes / Timeline global / estadísticas | Agregar crush |
| Búsqueda/filtros | Limpiar filtros |
| Timeline del Crush | Agregar experiencia |
| Actividad reciente | Ir a Timeline |
| Mapa sin ubicaciones | Ver crushes |
| Redes / cumpleaños / profesión / características | Agregar dato correspondiente; cumpleaños sólo día/mes |
