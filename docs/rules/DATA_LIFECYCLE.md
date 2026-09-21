# Reglas de producto — Baja, inactividad y ciclo de vida de los datos

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Baja voluntaria o por inactividad, seudonimización, destrucción, conservación y tareas automáticas.

<a id="baja"></a>
## Baja voluntaria de la cuenta

- **RP-BAJ-001.** La baja exige una sesión válida, elegir un motivo disponible o la opción general permitida y confirmar expresamente la decisión. Sólo se comunica éxito después de confirmar el resultado.
- **BR-3.09.** La baja es irreversible: retira la identidad y el acceso, destruye el contenido privado según sus plazos y conserva únicamente la estructura seudonimizada descrita en [Qué desaparece y qué se conserva](#destino-datos), condicionada a aprobación Legal.
- **RP-BAJ-002.** La baja produce el mismo resultado exista o no exista un Vault. No puede destruir la estructura en un caso y conservarla en otro.
- **RP-BAJ-003.** La baja se completa como una única acción. Un fallo no puede dejar la cuenta parcialmente cerrada ni una parte del contenido todavía accesible por accidente.
- **RP-BAJ-004.** La baja revoca todas las sesiones, retira los identificadores directos e invalida inmediatamente los dispositivos, claves y mecanismos de recuperación del Vault.
- **RP-BAJ-005.** Todo el contenido cifrado del Vault, incluido el que pertenecía a elementos eliminados anteriormente, queda fuera del acceso ordinario y se destruye físicamente después de siete días.
- **RP-BAJ-006.** La actividad utilizada para construir Home se elimina para impedir que el feed personal pueda reconstruirse.
- **RP-BAJ-007.** Después de confirmar la baja, la aplicación elimina claves, Vault local, notas, fotos, cambios pendientes y preferencias sensibles. Si la limpieza del dispositivo se interrumpe, conserva únicamente lo necesario para terminarla al volver a abrir la aplicación.
- **RP-BAJ-008.** Perder la respuesta o recibir un error de sesión después de solicitar la baja no demuestra que haya terminado. El contenido local se conserva hasta confirmar el resultado de forma segura.
- **RP-BAJ-009.** Antes de solicitar la baja, la aplicación conserva en el dispositivo un secreto aleatorio de confirmación de al menos 256 bits. El servidor conserva únicamente su hash junto con el recibo de la baja. Ese secreto permite confirmar el resultado después de perder la respuesta o la sesión; el resultado no puede consultarse mediante email o identificador de usuario.
- **RP-BAJ-010.** Consultar con un secreto ausente, incorrecto o inexistente produce la misma respuesta genérica. El secreto no se incluye en URLs ni logs; su hash sigue el plazo de conservación que se apruebe para el recibo.
- **RP-BAJ-011.** Volver a abrir la aplicación reanuda la confirmación y la limpieza aunque la sesión ya haya sido revocada, sin mezclar datos de otra cuenta.
- **RP-BAJ-012.** Restaurar un backup no reactiva una cuenta eliminada ni vuelve a poner en uso ordinario datos que ya fueron retirados o seudonimizados.

<a id="destino-datos"></a>
## Qué desaparece y qué se conserva

| Datos | Resultado |
|---|---|
| Cuenta e identidad directa | Se retiran `email`, `email_lookup`, `password`, `birth_date`, `gender_id`, `country_id`, métodos de acceso Apple, sesiones, OTP y mecanismos de recuperación. La cuenta deja de poder utilizarse o recuperarse. |
| Contenido privado del Vault | Se retiran nombre, género, profesión, WhatsApp, Instagram, Facebook, Snapchat, TikTok y cumpleaños de los Crushes; `place` y `description` de las experiencias; todas sus versiones cifradas; y los medios de recuperación del Vault. La destrucción física se ejecuta después del plazo fijado. |
| Dispositivos y mecanismos de recuperación | Invalidación o eliminación inmediata. |
| Estado local y fotos | Limpieza después de la confirmación, reanudable si se interrumpe. |
| Actividad de Home | Eliminación. |
| Estructura de Crushes | Se conservan seudonimizados los campos estructurales definidos en [RP-CRU-013](CRUSH.md#inventario): catálogos, edad confirmada y su fecha, encuentro, valoraciones y estados de ciclo de vida necesarios. La vinculación directa con la cuenta se sustituye por una referencia seudónima fuera del uso ordinario. |
| Estructura de experiencias | Se conservan seudonimizados `date`, `satisfaction_id` y los vínculos estructurales definidos en [RP-EXP-025](EXPERIENCES.md#inventario). |
| Estructura relacional | Se conservan seudonimizados períodos, hitos, fases, etiquetas, motivos, fechas y asociaciones definidos en [RP-REL-008](RELATIONSHIPS.md#inventario). |
| Feedback, recibo y evidencia de seguridad | Sólo finalidad/datos/acceso/plazo aprobados; no ilimitados por omisión. |
| Token de revocación Apple | Excepción limitada RP-BAJ-016. |
| Backups/proveedores | Ciclo separado conocido; purgar blobs no acredita purgar todas las copias. |

- **RP-BAJ-013.** La estructura que deba preservarse para agregados no se elimina como un Crush individual, pero tampoco permanece consultable en el diario ni en otras superficies personales.
- **RP-BAJ-014.** Esa estructura se denomina **seudonimizada**, no anónima, hasta contar con una evaluación jurídica suficiente. Sus identificadores, fechas y combinaciones poco frecuentes podrían permitir reidentificación.
- **RP-BAJ-015.** Finalidad, base, plazo, accesos, separación y salvaguardas requieren Legal antes de habilitar tratamiento posterior; la decisión funcional no autoriza filas individuales ni estadísticas globales.
- **RP-BAJ-016.** La baja de una cuenta vinculada a Apple retira el vínculo y solicita su revocación. Un fallo de Apple no restaura la cuenta MDL: se realizan hasta cinco intentos inmediatos y, después del fallo final, sólo se conserva cifrado el material mínimo necesario durante un máximo de 30 días para un reintento manual autorizado. Al vencer ese plazo, se elimina.

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

<a id="retirada-y-destruccion"></a>
## Retirada del uso normal y destrucción física

- **RP-VLT-023.** El éxito visible de una eliminación retira el contenido del uso normal, pero no significa destrucción física inmediata: las copias cifradas se purgan a los siete días y los backups siguen su política propia.

<a id="tareas"></a>
## Tareas automáticas

Esta tabla reúne las tareas automáticas de la beta y el resultado que cada ejecución debe producir. Una tarea sólo cumple su regla cuando confirma el resultado material correspondiente.

| ID | Proceso | Periodicidad declarada | Resultado exigido |
|---|---|---|---|
| RP-AUT-001 | Vencimiento provisional | Diaria | Baja a elegibles tras siete días; reenvío no extiende. |
| RP-AUT-002 | Inactividad | Diaria | Detectar 18 meses, avisar y cerrar tras gracia sin login, comprobando de nuevo. |
| RP-AUT-003 | Limpieza de tokens caducados | Diaria | Retirar tokens; caducidad efectiva no espera limpieza. |
| RP-AUT-004 | Destrucción diferida del Vault | Diaria | Destruir contenido cifrado que lleva siete días retirado y eliminar sus referencias cuando ya no quede ninguna versión. |
| RP-AUT-005 | Revocaciones Apple pendientes | Cada hora | Reintentos dentro de política finita y fallo final. |
| RP-AUT-006 | Purga revocaciones Apple | Diaria | Retirar secretos/intenciones completados o vencidos. |
| RP-AUT-007 | Purga notificaciones Apple | Diaria | 30 días, con excepción del último evento de identidad activa. |
| RP-AUT-008 | Limpieza de registros elegibles | Diaria | OTP/actividad/auditoría/seguridad conforme a retención aprobada. |
| RP-AUT-009 | Procesamiento programado de tareas pendientes | Cada hora, además del procesamiento continuo | Procesar tareas acumuladas; no significa que toda tarea deba esperar una hora cuando el procesamiento continuo está activo. |
| RP-AUT-014 | Limpieza de subidas Vault incompletas | Cada hora | Retirar objetos y metadata de capacidades no confirmadas que hayan cumplido 24 horas; la caducidad y la liberación de cuota son efectivas al vencer, sin esperar a la tarea. |

- **RP-AUT-010.** Las tareas de aviso y cierre consultan el estado vigente antes de actuar y no ejecutan una decisión que haya quedado desactualizada.
- **RP-AUT-011.** Si una destrucción falla, se conserva lo necesario para reintentarlo y no se declara eliminado un contenido cuya copia física todavía existe.
- **RP-AUT-012.** Las tareas automáticas tienen intentos limitados, esperan progresivamente entre ellos y dejan un fallo investigable sin duplicar efectos.
- **RP-AUT-013.** Una limpieza se considera completada únicamente cuando la tarea identifica los registros elegibles, ejecuta la retirada o destrucción exigida y deja evidencia técnica del resultado sin contenido privado.

<a id="plazos"></a>
## Plazos de conservación y destrucción

Esta tabla contiene únicamente plazos funcionales ya fijados. Los plazos todavía abiertos se enumeran en [Decisiones pendientes](#decisiones-pendientes).

| Conjunto | Plazo/condición | Salvedad |
|---|---|---|
| Inactividad | 18 meses desde `last_login` o, si falta, `email_verified_at`; después 20 días desde aviso aceptado o último intento ante rechazo definitivo | Base, transparencia y tratamiento jurídico de la falta de entrega requieren aprobación Legal. |
| Contenido cifrado retirado | 7 días desde su retirada, luego destrucción periódica | Backups y versiones siguen ciclos separados; revisión Legal. |
| Referencia entre contenido y elemento | Hasta destruir la última versión | No permanece indefinidamente hasta la baja. |
| Revocación Apple | Hasta éxito o 30 días desde fallo final | Sólo material mínimo cifrado. |
| Notificación Apple | 30 días | Último evento sólo mientras identidad activa. |

Los plazos de [sesión, verificación y recuperación del acceso](ACCOUNT.md) se definen en Cuenta. La sustitución de una versión cifrada se define en [Vault](VAULT.md#recuperacion), y las esperas de reintento en [Guardado y sincronización](SYNC.md#reintentos). Las tareas anteriores aplican esas reglas; no crean plazos alternativos.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-02"></a>DEC-02 | Finalidad/base/plazo/acceso/separación/salvaguardas de estructura post-baja. | Conservación funcional decidida; seudonimizada y fuera de uso ordinario; sin habilitar uso posterior. |
| <a id="dec-09"></a>DEC-09 | Plazos de ActivityEvent, AuditEvent, AuthEvent, SecurityAlert, aceptaciones, feedback, recibos, logs, backups y proveedores. | La baja elimina el feed personal; ningún conjunto conserva datos indefinidamente por omisión; los siete días del Vault no fijan el plazo de los demás conjuntos. |
