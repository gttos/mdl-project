# Baja completa de cuenta y atención por email

<a id="objetivo-y-alcance"></a><a id="recorrido-y-responsabilidades"></a><a id="dependencias-e-interacción-entre-iniciativas"></a><a id="fases-y-criterios-de-salida"></a><a id="escenarios-de-aceptación"></a><a id="decisiones-pendientes-y-activación"></a>

> **Estado:** planificación de una feature independiente de experiencias. Incluye el proceso integral común, ejecución diaria, aviso de aceptación y atención administrativa por email. El aviso está preparado y probado; el motor integral, su conexión de envío y los comandos siguen sin implementar. Sin operación sobre cuentas, correos reales o despliegue.
> **Autoridad:** [Baja integral, plazo y aviso](../../../rules/DATA_LIFECYCLE.md#baja), [RP-BAJ-020..027 y textos de atención](../../../rules/DATA_LIFECYCLE.md#baja-por-email) y [derechos](../../../rules/PRIVACY.md#derechos).
> **Plan local:** [API](../../../../../api/docs/plans/active/account-deletion-by-email/README.md).

## Objetivo y alcance

Implementar el proceso completo de baja de cuenta de RP-BAJ-001..027: aceptación durable, retirada de acceso, eliminación integral del alcance aprobado, ejecución diaria a las 07:00 Europe/Madrid, aviso de aceptación y comandos para atender solicitudes por email. App y consola reutilizan una única autoridad y un único ejecutor. La verificación externa es manual y proporcionada; el remitente aparente por sí solo no acredita una cuenta.

Incluye los cuatro comandos internos, su asociación mínima de gestión, los textos canónicos y su transporte, el proceso de borrado completo, scheduler, reanudación/idempotencia, aviso durable y pruebas con/sin Vault y con Apple. El aviso posterior de aceptación conserva RP-BAJ-019; no se redefine ni se copia su plantilla en este plan. Los detalles de almacenamiento, DynamoDB y restauración pertenecen a su iniciativa separada, con integración conjunta antes de operar.

Fuera de alcance: portal web, OTP, enlaces con autoridad, automatizar la lectura del buzón o aceptar una palabra como autorización automática, acceso al diario, nuevos tratamientos analíticos, eliminación de leads y cambios de login o UI. No se modifica Legal, el blueprint, AWS o producción. No se crea un segundo motor de borrado.

## Recorrido y responsabilidades

| Paso | Operación | Resultado esperado |
|---|---|---|
| 1 | El operador recibe la solicitud y consulta el email. | Coincidencia exacta, ausencia o fallo técnico distinguibles sólo internamente; sin efectos sobre la cuenta. |
| 2A | Con coincidencia, envía la confirmación al correo resuelto desde la cuenta. | Gestión ligada a esa cuenta y mensaje; la cuenta sigue activa y no hay baja aceptada. |
| 2B | Sin coincidencia, envía al mismo email consultado la respuesta con ayuda Apple. | No declara inexistencia absoluta ni cierra una solicitud con dificultades. Si llega otra dirección, vuelve a consultar. |
| 3 | La persona responde a la confirmación y el operador la revisa. | Se acredita la intención expresa y su correspondencia con el mensaje enviado; no se solicitan secretos. |
| 4 | El operador ejecuta la baja acreditada sobre esa gestión. | Revalidación del objetivo y aceptación común; cierre de acceso y aviso existentes en el contrato, sin purga inmediata ni autorización trasladada a otra cuenta. |
| Excepción | No puede localizarse la cuenta, no llega el correo o hay indicios contradictorios. | Revisión individual por el procedimiento de derechos; no se fuerza la app ni se ejecuta un borrado sin acreditar. |

Product conserva reglas y textos. API implementa comandos, protección de email, persistencia mínima, concurrencia y correo. El operador autorizado revisa el buzón y confirma el resultado; el software no deduce identidad de una respuesta reenviada. Legal valida suficiencia de acreditación, excepciones y conservación. iOS y Android conservan la baja autenticada actual; no requieren código nuevo para estos comandos. Apple Private Relay exige comprobar el remitente atendido y el recorrido de ida y respuesta.

## Dependencias e interacción entre iniciativas

- Esta iniciativa es propietaria del nuevo borrado integral y su validación. Experiencias queda cerrada en su alcance de formulario, lectura y estadísticas; no contiene fases ni criterios de salida de baja. Se conserva íntegro el pendiente trasladado: app/consola deben retirar acceso, registrar intención durable y eliminar cuenta, diario, respuestas, Vault y copias bajo el plazo aprobado, con reintentos y sin conservación seudonimizada.
- [Vault MySQL, DynamoDB y restauración](../backup-restore-safety/README.md) conserva sus fases y DEC-14. La aceptación de una baja desde consola debe atravesar la misma frontera durable que la app cuando esa integración se habilite. Consultas, confirmaciones previas y ausencias no se registran como bajas en DynamoDB.
- El aviso RP-BAJ-019 está preparado, sin envío automático. Sólo se integra al aceptar efectivamente la baja; la fecha no se cuenta desde el correo preparatorio.
- No se cambia el plazo general de borrado, backups o registro. La conservación de gestiones no confirmadas y del buzón queda sujeta al procedimiento jurídico, sin asignarles siete días por analogía con backups.

## Fases y criterios de salida

Una fase por sesión; este plan no inicia implementación ni autoriza operar datos reales.

| Fase | Trabajo | Salida | Estado |
|---|---|---|---|
| EBD-01 | Cerrar diseño de comandos, asociación cuenta/email/mensaje, permisos, estados, caducidad/purga y revisión humana; mapear consumidores y motor común. | Contrato técnico en el plan API, GWT mapeados y decisiones jurídicas pendientes explícitas. Sin código, recursos o correos reales. | Pendiente; siguiente fase de esta iniciativa. |
| EBD-02 | Implementar consulta, gestión mínima y los dos correos preparatorios. | Pruebas dirigidas, arquitectura, privacidad y MySQL pertinente; ningún efecto de baja o acceso. | Pendiente. |
| EBD-03 | Implementar la aceptación común, borrado integral reanudable, ejecución diaria y aviso de aceptación. | Inventario completo de datos, pruebas con/sin Vault, rollback, concurrencia, reintentos y plazo; sin conservación seudonimizada. Integración con la frontera de BRS coordinada y sin política alternativa. | Pendiente. |
| EBD-04 | Integrar el comando de ejecución con el proceso integral validado. | Identidad, revisión humana, reutilización de email, atomicidad y retry; app y consola aceptan una sola baja y un solo aviso. Sin dependencias validadas, ejecución inhabilitada. | Pendiente. |
| EBD-05 | Validar ciclo integral y atención asistida; cerrar. | Recorrido sintético local, revisión humana, correo/Relay autorizados y criterio jurídico, copias y plazo acreditados; documentos estables reconciliados antes de retirar planes. | Pendiente. |

## Escenarios de aceptación

| ID | Dado / cuando / entonces | Regla |
|---|---|---|
| GWT-EBD-01 | Dado un email personal registrado, consultar lo resuelve por coincidencia exacta protegida sin leer el diario, emitir correo o modificar la cuenta. | RP-BAJ-020/021 |
| GWT-EBD-02 | Dada una ausencia real, el envío negativo va sólo al email consultado, usa el texto canónico y no prueba que la persona carezca de otra cuenta. | RP-BAJ-025/027 |
| GWT-EBD-03 | Dado un fallo de base, lookup o cifrado, no se anuncia ausencia ni se envía el mensaje negativo. | RP-BAJ-021/025 |
| GWT-EBD-04 | Dada una coincidencia y un remitente o Reply-To distinto, la confirmación va al email de la cuenta, sin aceptar la baja ni cerrar acceso. | RP-BAJ-020/022 |
| GWT-EBD-05 | Dada una palabra BAJA, un código fijo, un reenvío o una afirmación de recepción sin revisión, no se ejecuta ninguna baja automáticamente. | RP-BAJ-022/023 |
| GWT-EBD-06 | Dada una respuesta revisada, la ejecución exige declaración explícita del operador y la misma gestión, cuenta y dirección. | RP-BAJ-023/026 |
| GWT-EBD-07 | Dado un email cambiado o reutilizado por una cuenta nueva, una gestión anterior falla sin eliminar ninguna de ellas. | RP-BAJ-023 |
| GWT-EBD-08 | Dadas ejecuciones repetidas, simultáneas o respuesta perdida, hay una aceptación, instante original y aviso, sin reactivar acceso ni duplicar efectos. | RP-BAJ-003/019/023 |
| GWT-EBD-09 | Dada una baja acreditada, el comando y la app atraviesan el mismo proceso integral con y sin Vault, incluida la frontera independiente cuando se habilite; no se purga directamente desde consola. | RP-BAJ-002/003/018/023 |
| GWT-EBD-10 | Dada una cuenta Apple oculta no localizable por su correo personal, las instrucciones permiten aportar la dirección registrada; sólo su respuesta comprobada habilita ejecutar. | RP-BAJ-021/024/025 |
| GWT-EBD-11 | Dado Relay desactivado, rebote o falta de acceso, no se acredita la petición ni se cierra por inexistencia; continúa la revisión individual. | RP-BAJ-024/027 |
| GWT-EBD-12 | Dados correo, consola, logs, jobs y registro de bajas, no aparecen contenido íntimo, secretos o email legible en diagnóstico; la evidencia y su purga siguen el contrato aprobado. | RP-BAJ-026/027, RP-SEG-006, RP-DER-002 |
| GWT-EBD-13 | Dado un envío fallido o ambiguo, reintentar no duplica ciegamente mensajes ni acredita una respuesta inexistente; ningún envío inicia el plazo de borrado. | RP-BAJ-020/022/026 |
| GWT-EBD-14 | Dada falta de permisos o un motor integral no validado, la ejecución se rechaza sin degradarse al borrado anterior o anunciar aceptación. | RP-BAJ-018/023/027 |
| GWT-EBD-15 | Dada una baja con o sin Vault, aceptar retira el acceso y registra el borrado durable; la ejecución diaria elimina cuenta, todo el diario y sus copias recuperables identificables dentro del plazo aprobado. Un fallo se reintenta y nunca declara completada una destrucción parcial. | RP-BAJ-002..005/013..019 |
| GWT-EBD-16 | Dado un aviso de baja aceptada, muestra la fecha del día siguiente a las 07:00 Europe/Madrid desde la aceptación original; el reintento de correo no la desplaza ni duplica la baja. | RP-BAJ-003/019 |
| GWT-EBD-17 | Dadas aceptaciones, credenciales, entidades ya eliminadas, revisiones, subidas, recibos y registros identificables, el ejecutor integral verifica todo el alcance y no conserva estructura por anonimizar identificadores. | RP-BAJ-013/014/017 |

## Dependencias de activación

[Cuestiones de acreditación, conservación y correo](../../../DECISIONES_PARA_JIRA.md#baja-email). La activación conserva las autorizaciones y condiciones jurídicas pendientes.
