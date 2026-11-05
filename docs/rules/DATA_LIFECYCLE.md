# Reglas de producto — Baja, inactividad y ciclo de vida de los datos

<a id="baja-por-email"></a><a id="baja-copy-es"></a><a id="destino-datos"></a><a id="decisiones-pendientes"></a><a id="dec-02"></a><a id="dec-14"></a>

[Índice de reglas](../PRODUCT.md) · Baja voluntaria o por inactividad, borrado integral, conservación y tareas automáticas.

<a id="baja"></a>
## Baja voluntaria de la cuenta

- **RP-BAJ-001.** La baja desde la app exige una sesión válida, elegir un motivo disponible o la opción general permitida y confirmar expresamente la decisión. La solicitud externa acreditada sigue RP-BAJ-018. Sólo se confirma la aceptación de la solicitud después de registrarla; aceptar la baja no equivale a haber terminado la destrucción física.
- **BR-3.09.** La baja es irreversible: retira el acceso de inmediato y elimina íntegramente la cuenta, sus Crushes, experiencias, períodos, hitos, contenido Vault, asociaciones y datos identificables conforme al alcance y plazo de RP-BAJ-017. No conserva estructura seudonimizada ni contribuciones de la cuenta para estadísticas agregadas.
- **RP-BAJ-002.** La baja produce el mismo resultado exista o no exista un Vault. No puede destruir la estructura en un caso y conservarla en otro.
- **RP-BAJ-003.** La aceptación de la baja registra de forma atómica el cierre de acceso y la intención durable de borrado. La destrucción posterior es reanudable e idempotente; un fallo no vuelve a habilitar la cuenta ni declara destruida una copia pendiente. Las referencias necesarias para terminar el borrado se conservan hasta verificarlo, sujetas al plazo de RP-BAJ-017.
- **RP-BAJ-004.** La baja revoca todas las sesiones, retira los identificadores directos e invalida inmediatamente los dispositivos, claves y mecanismos de recuperación del Vault.
- **RP-BAJ-005.** Todo el contenido cifrado del Vault, incluido el de elementos eliminados anteriormente, subidas pendientes y revisiones sustituidas, queda fuera del acceso ordinario de inmediato y se destruye físicamente dentro del plazo de RP-BAJ-017. La baja de cuenta no espera la ventana de siete días del borrado individual.
- **RP-BAJ-006.** La actividad utilizada para construir Home se elimina para impedir que el feed personal pueda reconstruirse.
- **RP-BAJ-007.** Después de confirmar la baja, la aplicación elimina claves, Vault local, notas, contenedores cifrados de fotos, cualquier imagen descifrada en memoria, cambios pendientes y preferencias sensibles. Si la limpieza del dispositivo se interrumpe, conserva únicamente lo necesario para terminarla al volver a abrir la aplicación, nunca una copia persistente de la foto en claro.
- **RP-BAJ-008.** Perder la respuesta o recibir un error de sesión después de solicitar la baja no demuestra que haya terminado. El contenido local se conserva hasta confirmar el resultado de forma segura.
- **RP-BAJ-009.** Antes de solicitar la baja, la aplicación conserva en el dispositivo un secreto aleatorio de confirmación de al menos 256 bits. El servidor conserva únicamente su hash junto con el recibo de la baja. Ese secreto permite confirmar el resultado después de perder la respuesta o la sesión; el resultado no puede consultarse mediante email o identificador de usuario.
- **RP-BAJ-010.** Consultar con un secreto ausente, incorrecto o inexistente produce la misma respuesta genérica. El secreto no se incluye en URLs ni logs; su hash sigue el plazo de conservación que se apruebe para el recibo.
- **RP-BAJ-011.** Volver a abrir la aplicación reanuda la confirmación y la limpieza aunque la sesión ya haya sido revocada, sin mezclar datos de otra cuenta.
- **RP-BAJ-012.** Restaurar un backup no reactiva una cuenta eliminada ni vuelve a poner en uso ordinario datos retirados. Las copias capaces de recuperar información identificable de una cuenta dada de baja quedan sujetas al mismo límite de borrado de RP-BAJ-017.

- **RP-BAJ-017.** El borrado integral se ejecuta cada día a las 07:00 en `Europe/Madrid`. Cada solicitud aceptada se procesa en la siguiente ejecución, sin esperar más allá de la mañana del día siguiente. El alcance incluye sistemas activos y toda copia recuperable bajo responsabilidad de MDL: base de datos, versiones y réplicas de objetos, snapshots, backups, exportaciones, colas y registros o proveedores con información que permita identificar o vincular la cuenta o reconstruir su diario. La antigüedad del dato, el cifrado o la retirada de identificadores directos no eximen de este borrado. Una copia pendiente impide declarar completado el resultado. La existencia, configuración y cumplimiento efectivos de cada almacenamiento se verifican antes de habilitar el compromiso público; la regla no acredita su implementación.
- **RP-BAJ-018.** Una solicitud recibida por el canal de privacidad se verifica de forma proporcionada antes de ordenar la baja. El remitente aparente de un correo no es por sí solo prueba de identidad. Un operador autorizado localiza la cuenta por su email registrado mediante una herramienta administrativa y confirma el alcance; la herramienta inicia el mismo proceso de borrado y aviso que la app, sin SQL manual, acceso al contenido privado ni una política de eliminación alternativa. Los casos en que no pueda acreditarse el control de la dirección se resuelven mediante el procedimiento de derechos aprobado, sin revelar datos de otra cuenta.
- **RP-BAJ-019.** La aceptación de la solicitud genera un único aviso transaccional al email registrado, tanto desde la app como desde la herramienta administrativa. El aviso informa del cierre de acceso, la irreversibilidad y la fecha y hora prevista de borrado de los sistemas activos: las 07:00 del día natural siguiente a la aceptación, en `Europe/Madrid`, también si la solicitud se acepta antes de las 07:00. La fecha se fija desde ese instante de aceptación, no desde la preparación o el reintento del correo. No contiene datos del diario, identificadores internos ni seguimiento. Su envío es reintentable sin duplicar la solicitud ni retrasar su plazo. Los datos de entrega y cualquier copia del mensaje bajo responsabilidad de MDL quedan sujetos a RP-BAJ-017. No se anuncia destrucción completada antes de verificarla.

El aviso puede prepararse y probarse localmente, pero su envío automático permanece desactivado hasta implementar y validar el borrado que anuncia. Preparar el correo no habilita envíos, no cambia el proceso de baja vigente ni acredita el cumplimiento de RP-BAJ-017. El tratamiento operativo de backups y la evidencia mínima de conservación siguen pendientes de revisión jurídica; el correo no declara borradas esas copias.

### Baja asistida por email

Decisión funcional aprobada; implementación pendiente en [Baja asistida por email](../plans/active/account-deletion-by-email/README.md). Esta sección concreta la herramienta y la acreditación proporcionada de RP-BAJ-018. La suficiencia de la acreditación manual y los plazos de sus gestiones requieren revisión Legal antes de operar con solicitudes reales. No se modifica el expediente jurídico ni se certifican entrega o borrado.

- **RP-BAJ-020.** La herramienta separa cuatro acciones: consulta de cuenta por email, envío de confirmación previa, ejecución de una baja acreditada y envío de respuesta sin coincidencia con ayuda para Apple. Consultar o enviar los correos previos no desactiva la cuenta, no revoca sesiones ni inicia el plazo de borrado. No se añade un portal de códigos ni una consulta pública de existencia.
- **RP-BAJ-021.** La consulta es exacta sobre el email actual registrado, con la normalización y protección vigentes, y muestra al operador autorizado sólo el resultado y contexto mínimo de gestión. No accede al diario, ni busca por fragmentos ni deduce el correo oculto de Apple desde el personal. Un fallo técnico o de descifrado no equivale a «sin coincidencia».
- **RP-BAJ-022.** Para una cuenta localizada, se envía un mensaje nuevo al email registrado que resolvió la consulta, nunca a otro destinatario o al `Reply-To` facilitado por quien solicita. La persona responde a ese mensaje confirmando expresamente la eliminación. El operador revisa la respuesta correspondiente y los posibles indicios contradictorios. No se utilizan códigos fijos, OTP, enlaces de autorización ni reenvíos como prueba; «BAJA» expresa una decisión y no es un secreto ni autoriza una acción automática.
- **RP-BAJ-023.** Ejecutar exige que el operador declare haber comprobado la respuesta y confirme objetivo y alcance. La autorización queda ligada a la misma petición, cuenta y dirección que recibieron la confirmación. Un cambio de email, una asociación perdida o la reutilización del correo por otra cuenta obligan a acreditar de nuevo. La ejecución reutiliza la aceptación y el aviso de RP-BAJ-003/019, sin SQL manual, purga inmediata o seudonimización alternativa. Un reintento no duplica bajas o avisos ni cambia el instante original.
- **RP-BAJ-024.** Apple utiliza el mismo circuito. La persona puede consultar la dirección compartida con MDL en los ajustes de Apple y facilitarla para localizar la cuenta; conocerla no acredita la baja. Se confirma mediante el email actual de MDL, incluida la dirección intermediaria si sigue siendo el registrado. No se exige entrar en la app ni crear una contraseña como única vía. Sin acceso al correo o identificación suficiente, continúa una revisión individual de derechos, sin solicitar los datos prohibidos por RP-DER-002.
- **RP-BAJ-025.** Sin coincidencia, el operador puede enviar al mismo email consultado la respuesta aprobada con ayuda para Apple. No se informa a un tercero sobre otra dirección, no se afirma que la persona no tenga ninguna cuenta y no se cierra automáticamente una solicitud que aporte otra identificación o indique dificultades. Un fallo técnico no habilita este envío; localizar después una cuenta exige confirmarla antes de ejecutar la baja.
- **RP-BAJ-026.** La referencia de gestión sólo correlaciona consulta, correo y revisión humana; no concede autoridad por sí misma. La evidencia mínima registra petición, cuenta cuando exista, mensaje, instantes, operador y resultado, sin copiar cuerpos de correo al diario, logs o registro de bajas. Las copias necesarias del email se cifran. Evidencia, buzones y datos de entrega cumplen minimización y los plazos aprobados; no se presume conservación indefinida ni se amplía RP-BAJ-017.
- **RP-BAJ-027.** Los correos preparatorios identifican la solicitud y sus próximos pasos sin comunicar una baja aceptada. La confirmación se dirige a la cuenta localizada; la respuesta sin coincidencia se limita a la dirección consultada y permite revisar el acceso con Apple. Los mensajes identifican al titular y sus documentos jurídicos, excluyen contenido del diario, identificadores internos y seguimiento, y conservan el canal atendido de respuesta. Su activación requiere procedimiento jurídico validado y entrega y respuesta comprobadas, incluido Apple Private Relay. La ejecución final exige además el proceso integral común validado; la implementación seudonimizada vigente no lo sustituye.

## Qué desaparece y qué se conserva

| Datos | Resultado |
|---|---|
| Cuenta e identidad directa | Se elimina la fila de cuenta y sus identificadores, credenciales, selecciones personales, sesiones, OTP, métodos de acceso y recuperación. El acceso queda invalidado desde la aceptación de la baja. |
| Contenido privado del Vault | Se retiran nombre, género, profesión, WhatsApp, Instagram, Facebook, Snapchat, TikTok y cumpleaños de los Crushes; `place`, `description` y `reflection` de las experiencias; todas sus versiones cifradas; y los medios de recuperación del Vault. La destrucción física se ejecuta después del plazo fijado. |
| Dispositivos y mecanismos de recuperación | Invalidación o eliminación inmediata. |
| Estado local y fotos cifradas | Limpieza después de la confirmación, reanudable si se interrumpe; ninguna copia persistente en claro. |
| Actividad de Home | Eliminación. |
| Estructura de Crushes | Eliminación íntegra de activos, archivados y eliminados previamente, incluidas cualidades, valoraciones, asociaciones y proyecciones. |
| Estructura de experiencias | Eliminación íntegra de textos cifrados, respuestas estructuradas, fechas, satisfacción y asociaciones, incluidas las de elementos eliminados previamente. |
| Estructura relacional | Eliminación íntegra de períodos, hitos, situaciones, motivos, fechas y asociaciones. |
| Aceptaciones, feedback, recibos y evidencia de seguridad | Eliminación de todo registro, copia o vínculo que permita identificar la cuenta, con el plazo de RP-BAJ-017; no existe una conservación post-baja automática por tratarse de un recibo o registro técnico. |
| Token de revocación Apple | Retirada del vínculo y revocación conforme a RP-BAJ-016, sin prolongar el plazo de borrado. |
| Backups/proveedores | Eliminación de las copias recuperables con información identificable dentro de RP-BAJ-017; borrar el objeto activo no acredita destruir sus versiones, réplicas o backups. |

- **RP-BAJ-013.** La baja elimina toda la estructura del diario; la seudonimización no es un destino alternativo ni un paso que permita conservar filas para agregados. Las estadísticas de otras cuentas no incorporan contribuciones de la cuenta eliminada.
- **RP-BAJ-014.** La retirada de nombre o email no demuestra anonimato: UUID, hashes, fechas, secuencias y combinaciones poco frecuentes pueden permitir vinculación o reidentificación. El alcance de RP-BAJ-017 incluye esas referencias y sus copias.
- **RP-BAJ-015.** La función futura de agregados entre cuentas permanece sujeta a [su definición y aprobación jurídica](../FUNCIONES_FUTURAS_PARA_JIRA.md#agregados). No autoriza conservar datos de cuentas eliminadas bajo el contrato de esta beta.
- **RP-BAJ-016.** La baja de una cuenta vinculada a Apple retira el vínculo y solicita su revocación con intentos finitos. Un fallo externo no restaura la cuenta MDL ni permite conservar tokens, identidad o intenciones identificables más allá del plazo de RP-BAJ-017. La implementación debe reconciliar el reintento del proveedor con ese plazo antes de operar el nuevo borrado.

<a id="inactividad"></a>
## Baja por inactividad

- **BR-3.11.** **18 meses sin login → aviso → 20 días sin nuevo login → baja automática** con el mismo resultado de [Baja y destino de los datos](#baja).
- **RP-INA-001.** Iniciar sesión correctamente durante los 20 días de gracia cancela la baja por inactividad. Leer o responder el email no sustituye ese inicio de sesión.
- **RP-INA-002.** Un intento de acceso fallido, una consulta anónima o una solicitud de recuperación no cancelan la baja por inactividad.
- **RP-INA-003.** Antes del cierre se comprueba otra vez si hubo un inicio de sesión durante la gracia. Una tarea automática retrasada no puede cerrar una cuenta que volvió a activarse válidamente.
- **RP-INA-004.** Aviso/cierre son idempotentes; repetir no duplica mensajes ni extiende arbitrariamente gracia.
- **RP-INA-005.** Una cuenta provisional no verificada sigue el plazo fijo de siete días desde el alta y no entra en el ciclo de 18 meses de inactividad.
- **RP-INA-006.** Los 18 meses se cuentan desde `last_login`; si una cuenta verificada nunca lo registró, se cuentan desde `email_verified_at`. Si faltan ambas fechas, la cuenta queda fuera del cierre automático y genera una anomalía investigable sin contenido privado.
- **RP-INA-007.** El plazo de 20 días comienza cuando el proveedor de email acepta el aviso para entrega. MDL no necesita registrar aperturas, clics ni lectura del mensaje para aplicar la regla.
- **RP-INA-008.** Un fallo temporal de entrega se reintenta conforme a una política finita. Un fallo general del proveedor o de la configuración no inicia el plazo de gracia. Un rechazo definitivo propio de la dirección —incluidos un rebote permanente o Private Relay desactivado— inicia los 20 días desde el último intento después de agotar los reintentos autorizados; el resultado y la causa técnica se registran sin conservar el contenido del mensaje.

  El aviso dispone de un máximo de cinco intentos; los fallos generales también consumen presupuesto. Las esperas mínimas entre el primero y segundo, segundo y tercero, tercero y cuarto, y cuarto y quinto son respectivamente 24, 48, 72 y 96 horas desde el intento anterior registrado. Sin retrasos de ejecución, los intentos ocurren en los días 0, 1, 3, 6 y 10; una ejecución tardía no adelanta el siguiente reintento ni la gracia. Agotar el presupuesto con fallo general o temporal deja un fallo investigable sin gracia y no reinicia el presupuesto en la siguiente pasada diaria. Los 18 meses de inactividad y los 20 días de gracia permanecen sin cambios.
- **RP-INA-009.** El aviso de inactividad identifica a MDL, informa en español e inglés de los 18 meses sin login correcto y del cierre dentro de 20 días, y explica que sólo un login correcto en la app antes del vencimiento cancela la baja. El mensaje no incluye contenido de la cuenta, identificadores técnicos, tokens, enlaces de cancelación ni mecanismos de seguimiento. Su suficiencia jurídica y la entrega real con el proveedor se aprueban y verifican antes de operar el ciclo en el entorno de beta.
- **RP-INA-010.** Una cuenta habilitada con email verificado que todavía no terminó de configurar su Vault no tiene un vencimiento especial. Puede iniciar sesión y cada login correcto actualiza `last_login` y cancela o reinicia el ciclo de inactividad igual que en cualquier otra cuenta verificada; si no vuelve a iniciar sesión, se aplican los 18 meses y la gracia de 20 días de `BR-3.11`. Hasta completar el Vault, el acceso funcional permanece bloqueado por el flujo obligatorio definido en [RP-VLT-035](VAULT.md#recuperacion).

<a id="retirada-y-destruccion"></a>
## Retirada del uso normal y destrucción física

- **RP-VLT-023.** El borrado funcional y la destrucción física son resultados distintos. Los elementos eliminados individualmente conservan su ventana de siete días; la baja de cuenta elimina también ese contenido pendiente conforme al plazo integral de RP-BAJ-017, incluidas sus copias recuperables.

<a id="restauracion-segura"></a>
## Restauración de copias sin reactivar bajas

- **RP-BCK-001.** Restaurar una copia nunca reactiva una cuenta cuya baja fue aceptada ni su diario. La garantía alcanza base de datos, objetos cifrados, versiones y cualquier componente restaurado capaz de exponerlos; no depende de que el borrado físico hubiera terminado cuando se creó la copia.
- **RP-BCK-002.** Toda restauración se prepara en aislamiento, sin acceso desde los clientes, emisión de URLs utilizables por ellos ni ejecución ordinaria de workers, scheduler, correos o notificaciones. Sólo puede ponerse en servicio después de aplicar las bajas y verificar la coherencia del conjunto recuperado. Un error, resultado parcial o comprobación incompleta mantiene bloqueada esa puesta en servicio.
- **RP-BCK-003.** El registro de bajas conserva el identificador interno de la cuenta y el instante original de aceptación, con la información mínima de integridad y reanudación necesaria. Es independiente de la copia restaurada y permanece disponible para impedir reactivaciones. Cada cuenta se identifica por su propia identidad, aunque utilice el mismo email que otra. La conservación y utilización real requieren resolver las condiciones jurídicas registradas en [DEC-14](../DECISIONES_PARA_JIRA.md#dec-14).
- **RP-BCK-004.** Antes de habilitar una restauración debe acreditarse que el registro de bajas es íntegro, completo y vigente hasta el instante de puesta en servicio, incluidas las aceptadas durante la recuperación. Un registro ausente, inaccesible, antiguo, corrupto o de completitud no demostrable bloquea la operación. Restaurar el registro antiguo incluido en el mismo backup no satisface esta condición.
- **RP-BCK-005.** La limpieza de la restauración es reanudable e idempotente. Comprueba la retirada de cuenta, acceso, asociaciones, estructura y objetos de las bajas; no se limita a ocultar filas o retirar el email. No publica objetos sin propietario y asociación verificables ni declara recuperado un diario cuyos objetos necesarios faltan o son incoherentes. No necesita descifrar contenido privado en el servidor.
- **RP-BCK-006.** La conservación técnica se evalúa sobre todas las copias recuperables y su vencimiento efectivo, incluidas versiones, registros de recuperación, copias de restauraciones y del registro mínimo. Una retención configurada de siete días no certifica por sí sola la desaparición material de todos los datos en ese plazo. Los procesos de copia no perpetúan versiones retiradas ni renuevan indefinidamente su conservación; los límites aplicables deben quedar aprobados y demostrados antes de activarse.
- **RP-BCK-007.** Una política de backups o restauración se presenta como operativa sólo después de probar una recuperación completa y su bloqueo ante fallos en recursos autorizados y aislados. La evidencia utiliza datos sintéticos y resultados técnicos sin contenido privado; la aprobación jurídica, el ensayo técnico y la autorización para activar o cambiar producción son condiciones distintas.

<a id="tareas"></a>
## Tareas automáticas

Esta tabla reúne las tareas automáticas de la beta y el resultado que cada ejecución debe producir. Una tarea sólo cumple su regla cuando confirma el resultado material correspondiente.

| ID | Proceso | Periodicidad declarada | Resultado exigido |
|---|---|---|---|
| RP-AUT-001 | Vencimiento provisional | Diaria | Baja a elegibles tras siete días; reenvío no extiende. |
| RP-AUT-002 | Inactividad | Diaria | Detectar 18 meses, avisar y cerrar tras gracia sin login, comprobando de nuevo. |
| RP-AUT-003 | Limpieza de tokens caducados | Diaria | Retirar tokens vencidos en la siguiente ejecución, sin un día adicional de gracia; caducidad efectiva no espera limpieza. |
| RP-AUT-004 | Destrucción diferida del Vault | Diaria | Destruir contenido cifrado de elementos eliminados individualmente que lleva siete días retirado; la baja de cuenta aplica el plazo de RP-BAJ-017 sin esperar esa ventana. |
| RP-AUT-005 | Revocaciones Apple pendientes | Cada hora | Reintentos dentro de política finita y fallo final. |
| RP-AUT-006 | Purga revocaciones Apple | Diaria | Retirar secretos/intenciones completados o vencidos. |
| RP-AUT-007 | Purga notificaciones Apple | Diaria | 30 días, con excepción del último evento de identidad activa. |
| RP-AUT-008 | Limpieza de registros elegibles | Diaria | OTP/actividad/auditoría/seguridad conforme a retención aprobada. |
| RP-AUT-009 | Procesamiento programado de tareas pendientes | Cada hora, además del procesamiento continuo | Procesar tareas acumuladas; no significa que toda tarea deba esperar una hora cuando el procesamiento continuo está activo. |
| RP-AUT-014 | Limpieza de subidas Vault incompletas | Cada hora | Retirar objetos y metadata de capacidades no confirmadas que hayan cumplido 24 horas; la caducidad y la liberación de cuota son efectivas al vencer, sin esperar a la tarea. |
| RP-AUT-015 | Borrado integral de cuentas | Diaria, 07:00 `Europe/Madrid` | Procesar solicitudes aceptadas y verificar la destrucción de todo el alcance de RP-BAJ-017, incluidas copias recuperables y referencias identificables. |

- **RP-AUT-010.** Las tareas de aviso y cierre consultan el estado vigente antes de actuar y no ejecutan una decisión que haya quedado desactualizada.
- **RP-AUT-011.** Si una destrucción falla, se conserva lo necesario para reintentarlo y no se declara eliminado un contenido cuya copia física todavía existe.
- **RP-AUT-012.** Las tareas automáticas tienen intentos limitados, esperan progresivamente entre ellos y dejan un fallo investigable sin duplicar efectos.
- **RP-AUT-013.** Una limpieza se considera completada únicamente cuando la tarea identifica los registros elegibles, ejecuta la retirada o destrucción exigida y deja evidencia técnica del resultado sin contenido privado.

<a id="plazos"></a>
## Plazos de conservación y destrucción

Esta tabla contiene los plazos funcionales definidos.

- **RP-CON-001.** Salvo que una fila indique expresamente otra condición de inicio o vencimiento, los plazos de esta tabla son máximos contados desde la creación del registro. La purga programada elimina lo vencido en su siguiente ejecución diaria, como máximo dentro de las 24 horas posteriores al vencimiento. Una conservación excepcional sólo puede suspender la purga mediante un hold explícito con finalidad, alcance, responsable y vencimiento conforme a `RP-DER-006`.
- **RP-CON-002.** Las aceptaciones de términos y privacidad se conservan durante la vida de la cuenta. La baja elimina las aceptaciones y recibos vinculables a la persona, incluidos fecha, identificadores, IP y `user_agent`, dentro de RP-BAJ-017. Las versiones públicas de los documentos pueden conservarse sin el vínculo con quien las aceptó; no existe un plazo post-baja automático de cinco años para recibos individuales.
- **RP-CON-003.** Los backups y snapshots, cuando existan, tienen una retención general máxima de 30 días para datos cuya conservación siga autorizada. Esa retención no prolonga el borrado de una cuenta: las copias que permitan recuperar su información identificable se eliminan dentro de RP-BAJ-017. Se verifican versiones, réplicas, backups automáticos y manuales y registros de recuperación; una política de 30 días por sí sola no cumple el nuevo borrado. Restaurar una copia nunca reactiva una cuenta ni su contenido retirado.
- **RP-CON-004.** El lead de la web pública, su email cifrado y la evidencia de aceptación se conservan mientras no exista oposición. La oposición los elimina del sistema activo e impide cualquier envío pendiente; las copias técnicas ya existentes vencen conforme a `RP-CON-003`. No se conserva una lista de supresión paralela ni una IP del lead.

| Conjunto | Plazo/condición | Salvedad |
|---|---|---|
| Inactividad | 18 meses desde `last_login` o, si falta, `email_verified_at`; después 20 días desde aviso aceptado o último intento ante rechazo definitivo | Base, transparencia y tratamiento jurídico de la falta de entrega requieren aprobación Legal. |
| Cuenta dada de baja y su diario completo | Siguiente ejecución diaria de las 07:00 `Europe/Madrid`, sin superar la mañana del día siguiente | Incluye contenido previamente eliminado y copias identificables conforme a RP-BAJ-017. |
| Contenido cifrado retirado individualmente | 7 días desde su retirada, luego destrucción periódica | La baja de cuenta aplica antes RP-BAJ-017; no extiende ese plazo. |
| Referencia entre contenido y elemento | Hasta destruir la última versión | No permanece indefinidamente hasta la baja. |
| Crush y Experience eliminados individualmente | 7 días desde su borrado funcional; destrucción definitiva desde el instante exacto del vencimiento, en la siguiente limpieza diaria | El Crush elimina su historia vinculada conforme a RP-DEL-001/002; purgar una Experience conserva el resto de la historia que corresponda. La baja de cuenta elimina todo el conjunto según RP-BAJ-017 sin esperar esta ventana. No se pierden las referencias necesarias para destruir o reintentar la destrucción del contenido Vault. |
| Sesiones vencidas | Hasta su vencimiento efectivo; retirada en la siguiente limpieza diaria | Sin gracia adicional de conservación; no cambia la vigencia de sesión definida en Cuenta. |
| Continuaciones federadas | Hasta su consumo o vencimiento, lo que ocurra primero; retirada en la siguiente limpieza diaria | Sin día adicional de conservación; el consumo y la finalización de la operación autorizada conservan su atomicidad. |
| Desafíos de login y enrolamiento biométrico | Elegibles al alcanzar `expires_at`, incluido el instante exacto; retirada en la siguiente limpieza horaria | La limpieza no altera su vigencia, uso único, credenciales ni sesiones. El consumo válido y la obtención de los datos necesarios para completar su verificación no pueden quedar interrumpidos por la purga. No se añade una retirada anticipada por consumo. |
| Revocación Apple por baja | Hasta éxito o el límite de borrado de la cuenta, lo que ocurra primero | Sin material identificable retenido más allá de RP-BAJ-017. |
| Notificación Apple | 30 días | Último evento sólo mientras identidad activa. |
| `ActivityEvent` | 7 días; las cinco actividades más recientes de cada cuenta activa permanecen hasta ser desplazadas por otras nuevas | Eliminar el elemento asociado o cerrar la cuenta las retira antes. Nunca sobrevive como feed personal a la baja. |
| `AuditEvent` | 30 días | Metadata forense allowlisted; sin contenido privado. |
| `AuthEvent` | 90 días | IP y `user_agent` sólo para autenticación y seguridad. |
| `SecurityAlert` | 30 días | Incluye alertas coalescidas; sin payloads ni mensajes libres de la solicitud. |
| `LogEntry` persistido en la API | 7 días | Registro técnico allowlisted; no sustituye auditoría, autenticación ni alertas. |
| Logs locales rotatorios de API fuera de producción | 14 días como máximo | Producción usa `stderr`; no duplica este archivo local. |
| CloudWatch de API | 30 días | Nginx, PHP y Laravel saneados; la configuración efectiva debe aplicar expiración. |
| Sentry EU de API e iOS | 30 días | Sólo errores/crashes y diagnóstico allowlisted; sin replay ni perfiles de persona. |
| PostHog Cloud EU | 90 días | Aplica tanto a eventos como a informes o agregados conservados por el proveedor; no existen perfiles persistentes. |
| Metadata de entrega y rebotes del proveedor de email | 30 días | No conserva el cuerpo del mensaje más allá de lo imprescindible para entregarlo. |
| Aceptaciones de términos y privacidad | Vida de la cuenta; retirada con el borrado integral | Sin recibos individuales post-baja conforme a RP-CON-002. |
| Lead de la web pública y su evidencia | Hasta que la persona manifieste oposición | Al oponerse se elimina del sistema activo y se cancelan los envíos pendientes conforme a `RP-CON-004`; no se conserva IP ni lista de supresión paralela. |
| Feedback general de beta | Durante la beta y hasta 30 días después de su finalización | Si no existe antes una política sustitutiva aprobada, se purga íntegramente. |
| Feedback, recibo y hash de confirmación de baja | Únicamente durante la gestión del borrado, sujetos a RP-BAJ-017 si permiten identificar o vincular la cuenta | Nunca el secreto original ni conservación identificable automática durante 30 días; confirmar el resultado no requiere reabrir el diario. |
| Backups y snapshots | Máximo general de 30 días, sin prolongar el borrado de cuentas | Las copias identificables de una baja cumplen RP-BAJ-017; la arquitectura y configuración efectivas deben demostrarlo. |

Los máximos generales de logs, auditoría, autenticación, seguridad, feedback, correo y proveedores no permiten mantener información identificable de una cuenta después del límite de RP-BAJ-017. Su inventario y capacidad efectiva de eliminación deben verificarse antes de activar el compromiso. Una excepción jurídica exige la revisión humana y autorización explícita de RP-DER-006; no se presume por el nombre de un registro ni por su plazo técnico.

Los plazos de [sesión, verificación y recuperación del acceso](ACCOUNT.md) se definen en Cuenta. La sustitución de una versión cifrada se define en [Vault](VAULT.md#recuperacion), y las esperas de reintento en [Guardado y sincronización](SYNC.md#reintentos). Las tareas anteriores aplican esas reglas; no crean plazos alternativos.
