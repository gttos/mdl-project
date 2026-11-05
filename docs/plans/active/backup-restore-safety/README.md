# Vault en MySQL, bajas en DynamoDB y restauración segura

<a id="objetivo-y-alcance"></a><a id="decisiones-fijadas-y-límites"></a><a id="baseline-e-impacto-en-clientes"></a><a id="diseño-de-implementación"></a><a id="textos-cifrados-en-mysql"></a><a id="registro-dynamodb"></a><a id="restauración-y-vuelta-a-servicio"></a><a id="responsabilidades"></a><a id="fases-y-criterios-de-salida"></a><a id="escenarios-de-aceptación"></a><a id="consulta-a-legal-y-cierre"></a>

> **Estado:** planificación aprobada; implementación no iniciada. MySQL es el destino de todos los textos cifrados y AWS DynamoDB el registro independiente de bajas. S3 se conserva reservado para fotografías futuras, desconectado de los textos. No se han creado recursos AWS, migrado datos, adaptado clientes ni ejecutado pruebas de esta transición.
> **Autoridad:** [almacenamiento aprobado, RP-VLT-039..041](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#almacenamiento), [restauración, RP-BCK-001..007](../../../rules/DATA_LIFECYCLE.md#restauracion-segura), [DEC-14](../../../DECISIONES_PARA_JIRA.md#dec-14), [baja de cuenta](../../../rules/DATA_LIFECYCLE.md#baja) y [Legal](../../../legal/04_MDL_Ciclo_Vida_Retencion_Derechos.md).
> **Planes locales:** [API y AWS](../../../../../api/docs/plans/active/backup-restore-safety/README.md), [iOS](../../../../../ios/docs/plans/active/backup-restore-safety/README.md).

## Objetivo y alcance

Trasladar el almacenamiento remoto de todos los textos cifrados desde S3 a MySQL, conservar su cifrado en el dispositivo, registrar las bajas en DynamoDB y disponer de una recuperación de backups que elimine las cuentas afectadas antes de permitir acceso desde producción.

Incluye esquema y persistencia binaria, subida/descarga autenticadas en API y clientes, conservación de identidades e intenciones durables, traslado controlado del contenido autorizado a conservar, integración DynamoDB, preparación de infraestructura y ensayos de restauración.

Se conservan los adaptadores, conexiones y métodos S3 necesarios para futuras fotografías remotas, documentados y sin uso por los textos. No se elimina el servicio ni se habilitan ahora fotos remotas. No quedan dos destinos activos de texto ni fallback MySQL/S3. Las fotografías actuales siguen exclusivamente en los dispositivos.

No incluye rediseño de pantallas, cambios criptográficos, nuevas analíticas, edición del blueprint, publicación ni operación sobre datos reales. La retirada de copias manuales antiguas corresponde al operador y queda fuera de esta ejecución.

## Acuerdos y dependencias

- [Cambio de almacenamiento y registro independiente](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#almacenamiento).
- [Conservación y restauración, DEC-14](../../../DECISIONES_PARA_JIRA.md#dec-14).
- [Borrado integral](../account-deletion-by-email/README.md).

## Responsabilidades

| Ámbito | Responsabilidad |
|---|---|
| Product | Autoridad de almacenamiento, reserva S3, garantías, fases y decisiones pendientes. |
| API | Esquema, payloads, transferencia autenticada, cuotas, permisos, borrado, registro y verificación. |
| AWS / operación | Tabla DynamoDB, IAM, aislamiento, retención, costes y ensayos RDS/DynamoDB con autorización específica. |
| iOS | Transporte, DTOs, sync/recovery y pruebas; UI, cripto e identidades conservadas. |
| Android | Adopción en la reconciliación de incrementos antes de distribuir; sin alterar el clonado fijo. |
| Legal | Resolver DEC-14 y aprobar retención, registro mínimo y lenguaje público. |
| LLM | Sin cambios. |

## Fases y criterios de salida

Una fase por sesión. Crear el plan no inicia implementación ni autoriza cambios AWS.

| Fase | Trabajo | Salida exigida | Estado |
|---|---|---|---|
| BRS-01 | Cerrar esquema, contrato y mapa de consumidores API/clientes, reserva S3, protocolo DynamoDB/MySQL y corte. | Contrato técnico y GWT mapeados en planes locales; cuestiones jurídicas identificadas. Sin recursos reales. | Pendiente; siguiente fase. |
| BRS-02 | API: migración MySQL, persistencia binaria, transferencia, confirmación, descarga, cuotas y purga; preparar el corte. | Unitarias/Feature/Contracts y MySQL/concurrencia pertinentes; identidad e integridad; S3 conservado y desconectado del texto. | Pendiente. |
| BRS-03 | Adaptar iOS al nuevo transporte. | Build, pruebas y API descartable: offline/reintento/relanzamiento y recovery; sin S3 de texto o cambios visuales/criptográficos. | Pendiente. |
| BRS-04 | Registro DynamoDB, infraestructura revisable e integración con aceptación durable. | Caídas por frontera, respuestas perdidas, duplicados y rollback; registro completo. Recursos reales con autorización aparte. | Pendiente. |
| BRS-05 | Integrar limpieza/restauración MySQL con registro vigente y ejecutor integral de baja. | Ensayos locales de ausencia de bajas, bloqueo, carreras y conservación de otras cuentas. Requiere borrado integral validado. | Pendiente. |
| BRS-06 | Ensayar RDS/DynamoDB en recursos AWS aislados y sintéticos. | Permisos, integridad, bloqueo, tiempos, vencimiento y limpieza; autorización específica y coste acotado. | Pendiente. |
| BRS-07 | Reconciliar criterio jurídico, textos y cierre coordinado. | DEC-14 resuelta, corte API/clientes y borrado diario validados; integrar documentos y retirar planes juntos. Activación/despliegue con autorización separada. | Pendiente. |

<a id="escenarios-de-aceptacion"></a>
## Escenarios de aceptación

| Escenario | Dado / cuando / entonces | Reglas |
|---|---|---|
| GWT-BRS-01 | Dada copia anterior a una baja aceptada, recuperar elimina la cuenta y su alcance antes del acceso de la app. | RP-BCK-001/002/005 |
| GWT-BRS-02 | Dada baja cuyo cron aún no se ejecutó, la recuperación tampoco la reactiva ni espera otro cron para bloquearla. | RP-BCK-001/004 |
| GWT-BRS-03 | Registro ausente, corrupto, antiguo, inaccesible o de completitud no demostrable: no se autoriza puesta en servicio. | RP-BCK-002/004 |
| GWT-BRS-04 | Caídas por frontera MySQL/DynamoDB: ninguna baja confirmada desaparece ni un fallo no aceptado se vuelve baja de otra cuenta. | RP-BCK-003/004 |
| GWT-BRS-05 | Limpieza interrumpida/duplicada: reintentar termina sin reactivar cuentas o eliminar otras. | RP-BCK-005 |
| GWT-BRS-06 | Otra cuenta con el mismo email: la baja del ID anterior sólo elimina la cuenta anterior. | RP-BCK-003 |
| GWT-BRS-07 | Baja durante recuperación o activación: el corte la incorpora o permanece bloqueado. | RP-BCK-004 |
| GWT-BRS-08 | Contenido de una baja, revisiones y subidas pendientes: la limpieza retira payloads y asociaciones, sin versión accesible en el destino. | RP-BCK-001/005, RP-VLT-039 |
| GWT-BRS-09 | Payload huérfano, hash contradictorio, contenido requerido ausente o recuperación parcial: no se publica como válida. | RP-BCK-002/005 |
| GWT-BRS-10 | Cuenta no eliminada y corte válido: un cliente recupera contenido sintético coherente sin entregar secretos al servidor. | RP-BCK-005, RP-BASE-005 |
| GWT-BRS-11 | Clientes, jobs y credenciales ordinarias no acceden al destino ni emiten comunicaciones antes de validarlo. | RP-BCK-002 |
| GWT-BRS-12 | Los vencimientos se comprueban sobre el horizonte real; un fallo no renueva silenciosamente retención ni permite retirar antes la evidencia necesaria. | RP-BCK-006, DEC-14 |
| GWT-BRS-13 | Restaurar servidor/DynamoDB no sustituye evidencia vigente por una generación anterior ni abre destinos no validados. | RP-BCK-004/007 |
| GWT-BRS-14 | Upload MySQL valida límite, checksum, binding, propietario, cuota y revisión; repetir no duplica y una contradicción conserva el estado previo. | RP-VLT-027..031/039/041 |
| GWT-BRS-15 | Listados y solicitudes ajenas no incluyen o entregan bytes privados; descarga, logs y errores conservan autorización/saneamiento. | RP-VLT-002/039/041, RP-SEG-006 |
| GWT-BRS-16 | Offline, reintento o relanzamiento conservan IDs y ciphertext; bloqueo/cambio de cuenta/recovery no publican contenido cruzado o parcial. | RP-VLT-039/041, RP-ID-007 |
| GWT-BRS-17 | El texto no llama S3 ni usa fallback; adaptadores/métodos reservados permanecen y no habilitan fotos remotas. | RP-VLT-040 |
| GWT-BRS-18 | Traslado autorizado desde S3 verifica bytes/hash, identidad y round-trip antes de retirar referencias; un fallo conserva fuente y bloquea corte. | RP-VLT-039/040 |
| GWT-BRS-19 | DynamoDB sólo conserva ID/fecha y controles necesarios con permisos propios; TTL por sí solo no acredita caducidad material. | RP-BCK-003/006, DEC-14 |

## Dependencias y siguiente fase

- [DEC-14](../../../DECISIONES_PARA_JIRA.md#dec-14).
- [Cambio de almacenamiento](../../../FUNCIONES_FUTURAS_PARA_JIRA.md#almacenamiento).
- Siguiente fase: BRS-01. La implementación, recursos, pruebas, migración y despliegue conservan sus estados de la tabla de fases.
