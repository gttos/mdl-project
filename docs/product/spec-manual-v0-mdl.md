# Spec manual v0 — MyDuckingLife (MDL)

> Ejercicio de Spec-Driven Development (SDD): primera versión de la spec de producto, escrita a mano, sobre la plantilla universal de 8 secciones. Describe el QUÉ, nunca el CÓMO. Es el punto de partida para `/speckit.specify`.

## 1. Objetivo y contexto de negocio

MDL es una aplicación privada donde una persona lleva el diario de su vida romántica: cada persona que le interesa ("crush"), cada encuentro que tiene con ella y cómo evoluciona esa relación en el tiempo. No es una red social ni una app de citas: nadie más ve ese contenido, no hay perfiles públicos ni contacto entre usuarias. Es una herramienta de auto-reflexión personal.

Una escena típica: una usuaria vuelve de una primera cita, abre la app, busca al crush que ya tenía registrado (o lo crea si es la primera vez), añade la cita como una experiencia nueva con la fecha, el lugar y cómo se sintió, y sigue con su día. Semanas después, cuando esa persona pasa a ser una relación formal, actualiza su estado; si la relación termina, la cierra con una nota. Con el tiempo, la app le devuelve estadísticas propias: cuánto tarda en pasar de conocer a alguien a tener una relación, qué patrones se repiten.

Importa porque hoy esa reflexión vive dispersa (notas sueltas, mensajes, memoria) y no da ninguna garantía de privacidad si alguien accede al teléfono. MDL centraliza esa cronología y protege específicamente el contenido más sensible (nombre, contacto y notas del crush) de forma que nadie más pueda leerlo, ni siquiera quien opera el sistema.

**Éxito de negocio:** registrar una experiencia nueva sobre un crush ya existente (la acción más frecuente de la app) debería tardar menos de 1 minuto desde que se abre la app hasta que queda guardada, sin fricción que rompa el momento de reflexión.

## 2. Usuarios

- **Usuaria/o de MDL (usa la aplicación).** Persona adulta (18 años o más) que se registra para llevar su propio diario romántico. Sabe usar una app móvil normal; no necesita ningún conocimiento técnico. Configura su cuenta una vez y después crea crushes, añade experiencias, actualiza estados de relación y consulta sus estadísticas. Es la única persona con acceso a su contenido.
- **Crush (no usa la aplicación).** Persona real de interés romántico sobre la que la usuaria escribe. Nunca abre la app, nunca tiene cuenta, nunca sabe que sus datos (nombre, contacto, redes, cumpleaños) están registrados — y sin embargo una regla de negocio central de todo el producto depende de quién es: como nunca dio su consentimiento usando la app, su nombre, contacto, redes y cumpleaños no pueden quedar legibles para nadie más que la usuaria que los escribió, ni siquiera para el propio sistema.

## 3. Escenarios de usuario

- HU1 — Como usuaria de MDL, quiero proteger con una contraseña propia el contenido más privado de mi cuenta al registrarme, para que nadie más pueda leer lo que escribo sobre mis crushes.
- HU2 — Como usuaria de MDL, quiero registrar un crush nuevo con su nombre, datos de contacto y cómo lo conocí, para empezar a llevar su cronología desde el primer momento.
- HU3 — Como usuaria de MDL, quiero añadir una experiencia a un crush existente (una cita, un encuentro puntual) con su fecha, lugar y cómo me sentí, para poder recordar más adelante cómo fue esa relación con el tiempo.
- HU4 — Como usuaria de MDL, quiero actualizar el estado de la relación con un crush (de "prospecto" a "ligue", a "relación", a "terminado"...), para que la cronología refleje cómo evolucionó realmente, sin borrar ni reescribir el historial anterior.
- HU5 — Como usuaria de MDL, quiero consultar estadísticas construidas solo con mis propios datos (satisfacción promedio, tiempos entre estados, patrones frecuentes), para entender mejor mi propia vida romántica sin que nadie más las vea.
- HU6 — Como usuaria de MDL, quiero eliminar mi cuenta y que todos mis datos identificables se anonimicen de forma irreversible, para tener control total sobre mi privacidad si dejo de usar la app.

## 4. Requisitos funcionales

- RF1. Al volver a abrir la app en cualquier momento posterior, la usuaria debe encontrar su catálogo de crushes, experiencias, relaciones y estadísticas tal como los dejó.
- RF2. El nombre, el contacto, las redes y el cumpleaños exacto de un crush solo deben poder leerse por la usuaria que los creó; en ningún momento deben quedar legibles para nadie más, ni siquiera para el propio sistema que los almacena.
- RF3. Cada crush debe tener, en todo momento, un único estado de relación activo: nunca cero (si existe al menos un estado registrado) y nunca más de uno a la vez.
- RF4. Al registrar un nuevo estado de relación con un crush, el sistema debe cerrar automáticamente el estado anterior que estaba activo.
- RF5. Solo debe poder editarse o eliminarse el estado de relación más reciente de un crush; los estados anteriores quedan fijos como historial no editable.
- RF6. El sistema debe impedir que la fecha de una experiencia sea anterior a la fecha en que la usuaria conoció al crush.
- RF7. El sistema debe indicar automáticamente si "algo pasó" con un crush a partir de su estado de relación más reciente, y ese resultado, una vez confirmado en positivo, no debe poder volver atrás.
- RF8. El sistema debe calcular automáticamente una valoración global del crush a partir de sus valoraciones individuales, sin que la usuaria tenga que calcularla a mano.
- RF9. Las estadísticas mostradas a una usuaria deben construirse exclusivamente con sus propios datos, excluyendo cualquier crush o experiencia que haya sido eliminado.
- RF10. Al eliminar un crush, su contenido privado (nombre, contacto, redes, cumpleaños, notas) y los archivos asociados deben eliminarse de la app sin dejar rastro recuperable; los datos de sus experiencias y relaciones que residen en la API deben quedar irreversiblemente anonimizados, conservando únicamente lo estrictamente necesario para estadísticas agregadas.
- RF11. La usuaria debe poder recuperar todo su contenido privado en un dispositivo nuevo usando su contraseña o su frase de recuperación, sin perder ninguna entrada previamente creada.
- RF12. Al eliminar su cuenta, todos los datos identificables de la usuaria deben quedar irreversiblemente anonimizados, conservando únicamente lo estrictamente necesario para estadísticas agregadas sin datos personales.

## 5. Reglas de negocio (con ejemplo)

- **Edad mínima:** 18 años, validada por fecha de nacimiento.
- **Valoración global de un crush:** promedio aritmético de sus 4 valoraciones individuales (belleza, conexión emocional, personalidad, atractivo), cada una en una escala de 1 a 5, redondeado a 2 decimales.
- **Resultado del crush ("¿pasó algo?"):** se deriva del estado de relación más reciente y es irreversible en positivo — una vez que la respuesta es "sí", ningún estado posterior puede devolverla a "no" o "todavía no se sabe".
- **Conservación de datos:** los registros de auditoría se conservan 90 días, los eventos de actividad reciente 7 días, y una entrada de contenido privado eliminada se borra de forma definitiva a los 7 días.
- **Acceso:** si la usuaria no vuelve a entrar en 30 días, debe volver a identificarse.

**Ejemplo de referencia (crush "Alex"):**

Valoraciones registradas (escala 1 a 5): belleza = 5, conexión emocional = 3, personalidad = 5, atractivo = 4.

Valoración global = (5 + 3 + 5 + 4) / 4 = **4,25**

| Fecha | Estado registrado | ¿Pasó algo con este crush? |
|---|---|---|
| 11 ene | Prospecto | Todavía no se sabe |
| 20 ene | Ligue | Todavía no se sabe |
| 14 feb | Relación | **Sí** |
| 3 jun | Terminada | Sí (se mantiene) |

- Si más adelante la usuaria volviera a registrar "Prospecto" sobre este mismo crush, la respuesta seguiría siendo **sí** — nunca vuelve a "todavía no se sabe".
- Si el historial de Alex se hubiera quedado solo en Prospecto → Ligue, sin llegar nunca a un estado positivo ni negativo, la respuesta seguiría siendo "todavía no se sabe" indefinidamente.
- Si en cambio el segundo estado registrado hubiera sido "Rechazado" en vez de "Ligue", la respuesta pasaría a **no** — y un "no" sí puede convertirse más tarde en "sí" si aparece un estado positivo, pero nunca vuelve a "todavía no se sabe".

## 6. Criterios de aceptación

- CA1. Con las valoraciones del ejemplo de la sección 5 (5, 3, 5, 4 sobre 5), la valoración global mostrada para ese crush es exactamente 4,25.
- CA2. Con el historial de estados del ejemplo (Prospecto → Ligue → Relación → Terminada), la respuesta a "¿pasó algo con este crush?" es sí.
- CA3. Si sobre ese mismo crush se registra un nuevo estado Prospecto después de Terminada, la respuesta sigue siendo sí, nunca vuelve a "todavía no se sabe".
- CA4. Un crush sin ningún estado de relación registrado nunca aparece con respuesta "sí".
- CA5. Al cerrar la app y volver a entrar 10 días después, el catálogo completo de crushes, experiencias y relaciones sigue disponible sin pérdida de datos.
- CA6. Al eliminar la cuenta, buscar el email original en el sistema no devuelve ningún resultado identificable.
- CA7. Al recuperar el contenido privado en un dispositivo nuevo con la frase de recuperación, aparecen todas las entradas previamente creadas, sin duplicados ni omisiones.
- CA8. Intentar editar un estado de relación que no sea el más reciente de un crush es rechazado por el sistema.
- CA9. Al eliminar un crush, su nombre y contacto dejan de estar disponibles en la app; sus experiencias y relaciones permanecen anonimizadas en las estadísticas agregadas, sin poder volver a vincularse a ese crush.

## 7. Casos límite

- CL1. Experiencia con fecha anterior a la fecha en que se conoció al crush: se rechaza, no se guarda.
- CL2. Dos experiencias del mismo crush en la misma fecha: no está permitido; el sistema evita el duplicado.
- CL3. Fallo a mitad de la sincronización de una entrada nueva de contenido privado: la versión anterior no se retira hasta confirmar la nueva; la operación queda reintentable sin pérdida de datos.
- CL4. Fallo al anonimizar un crush concreto durante el borrado de cuenta: los demás crushes de esa cuenta se anonimizan igual; qué pasa exactamente con el que falló queda en Preguntas abiertas (PA2).
- CL5. Estadísticas consultadas sin tener ningún crush cargado todavía: no tengo una respuesta decidida — queda en Preguntas abiertas (PA4).

## 8. Fuera de alcance (v0)

- Panel de administración o rol de administrador: MDL es 100 % self-service.
- Planes de pago o suscripciones con funciones diferenciadas.
- Notificaciones push.
- Tests de personalidad o de estilo de apego expuestos a la usuaria.
- Interpretación narrativa por IA de las estadísticas de la usuaria.
- Aplicación para Android.
- Exportación o descarga de los datos personales en un formato portable.
- **Punto propio:** recordatorios automáticos basados en fechas relevantes del crush (por ejemplo, un aviso por su cumpleaños). Suena como una extensión natural del propio producto — y precisamente por sonar tan natural es la funcionalidad fantasma más probable de colarse sin que nadie la haya pedido. Queda fuera de esta v0 por escrito.

---

## 9. Preguntas abiertas (para la sesión de clarificación con Spec Kit)

- PA1. **Cuenta de Google:** al anonimizar una cuenta que se registró con Google, ¿el identificador de esa cuenta de Google se conserva o se anula?
- PA2. **Fallo de anonimización:** si falla la anonimización de un crush concreto durante el borrado de cuenta, ¿se reintenta automáticamente, queda pendiente de revisión, o se ignora sin más?
- PA3. **Intentos de acceso:** ¿debe existir un límite de intentos fallidos antes de bloquear el acceso al contenido privado, y cuál?
- PA4. **Primer uso / catálogo vacío:** ¿qué debe mostrarse cuando una usuaria consulta sus estadísticas sin tener todavía ningún crush cargado?
- PA5. **Portabilidad de datos:** ¿la exportación de los datos personales entra en el alcance cercano del producto, o se mantiene fuera de forma indefinida?
- PA6. **Interpretación por IA:** ¿bajo qué criterio se decide que la interpretación por IA de las estadísticas pasa de estar en preparación a activarse para las usuarias, y quién lo aprueba?
