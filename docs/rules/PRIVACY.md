# Reglas de producto — Privacidad, seguridad y derechos

**Estado de revisión: FINALIZADO.** La definición funcional de este archivo está cerrada por Producto. Este estado no certifica implementación ni aprobación jurídica; las decisiones expresamente pendientes continúan abiertas y no autorizan a inventar comportamiento.

[Índice de reglas](../PRODUCT.md) · Aislamiento entre cuentas, protección de información, diagnóstico, soporte y ejercicio de derechos.

<a id="propiedad"></a>
## Propiedad de los datos y privacidad del diario

- **RP-BASE-002.** Cada cuenta sólo accede y modifica sus propios datos. Conocer un identificador ajeno no concede acceso ni permite descubrir su contenido.
- **RP-BASE-005.** El contenido íntimo del diario se cifra de extremo a extremo. El servidor conserva la estructura mínima necesaria para las funciones aprobadas.
- **RP-ID-007.** Cuenta, Vault, claves, preferencias, fotos y pendientes están separados por cuenta. Cambiar de cuenta nunca adopta datos de otra.

<a id="seguridad"></a>
## Protección del acceso y de la información

- **RP-SEG-001.** Toda operación privada comprueba una sesión válida y la propiedad de cada cuenta, Crush, experiencia, período, hito, Vault, blob, foto, operación pendiente y URL temporal que utilice.
- **RP-SEG-002.** La aplicación evita acciones inválidas y la API vuelve a comprobar propiedad e invariantes antes de aplicar una modificación. La ausencia de un botón no sustituye la validación de la API.
- **RP-SEG-003.** Un rechazo conserva íntegramente el estado anterior y no revela datos de otra cuenta. Los campos ajenos al contrato de la operación no cambian identidad, propiedad, permisos ni estado.
- **RP-SEG-004.** Login, verificación y recuperación presentan respuestas públicas que no permiten distinguir si una cuenta existe, está inactiva o utilizó una credencial incorrecta.
- **RP-SEG-005.** Los flujos de autenticación, OTP, Apple, URLs temporales y escrituras del Vault aplican límites de uso por su propio alcance y comunican cuándo puede volver a intentarse.
- **RP-SEG-006.** Logs, errores y analítica excluyen contraseñas, OTP, tokens, credenciales Apple, emails legibles, contenido del Vault, snapshots privados, URLs firmadas completas, tokens de búsqueda del Vault y payloads completos de solicitudes o respuestas.
- **RP-SEG-007.** El diagnóstico se construye mediante una lista positiva de contexto técnico: identificador de traza o correlación; plantilla de ruta y método; código de estado o error saneado; fecha, duración y conteo; versión de aplicación y sistema; e identificadores técnicos mínimos cuando sean necesarios para investigar. IP y `user_agent` sólo forman parte de eventos de autenticación, abuso o seguridad. Sus plazos máximos se definen en [Plazos de conservación y destrucción](DATA_LIFECYCLE.md#plazos).
- **RP-SEG-008.** Todo objeto privado y toda capacidad temporal quedan vinculados a su cuenta propietaria y respetan los [límites del Vault](VAULT.md#inventario-servidor): 65.536 bytes por objeto y 25 MiB lógicos por cuenta. La capacidad temporal reserva cuota sin duplicarla en los reintentos.
- **RP-SEG-009.** La IP y el esquema utilizados para seguridad sólo aceptan cabeceras de reenvío cuando proceden de un proxy expresamente confiable.
- **RP-SEG-010.** Los secretos operativos se entregan por el mecanismo autorizado del entorno y permanecen fuera del código, imágenes, artefactos, logs y resultados de pruebas. El acceso de personal utiliza el mínimo permiso necesario y queda trazado.
- **RP-SEG-011.** La observabilidad técnica y la analítica de producto son canales distintos. Un diagnóstico utiliza un código estable, severidad, categoría, versión/build, entorno, `traceId` y sólo el contexto técnico permitido por `RP-SEG-007`; nunca reutiliza eventos analíticos ni depende de que la persona mantenga activada la analítica. Cuando un fallo iOS corresponde a una petición API, cliente, respuesta, logs backend y reporte de error conservan el mismo `traceId` correlacionable.
- **RP-SEG-012.** Sentry EU es el receptor remoto seleccionado para errores iOS y se activa en la beta 0.2 distribuida por TestFlight y en el modo técnico explícito de `RP-ANA-020`, con una retención máxima comprobada de 30 días. Su configuración impide session replay, screenshots, adjuntos, perfiles, trazas de rendimiento, breadcrumbs automáticos de UI/red y captura automática de requests, responses, headers, queries, variables locales o contexto personal. Mientras su control ([`RP-SEG-013`](#seguridad)) esté activo, sólo recibe crashes no manejados y eventos `warning`/`error` incluidos en una allowlist comprobable; su ausencia o fallo no altera el comportamiento de MDL.
- **RP-SEG-013.** El diagnóstico remoto de Sentry iOS tiene un control propio e independiente del de analítica de `RP-ANA-005`: la persona puede desactivarlo y volver a activarlo, y desactivar uno no cambia el otro. El control cubre todo el diagnóstico remoto, tanto los diagnósticos allowlisted como los crashes no manejados. Mientras está desactivado, la app no inicia Sentry y no envía ningún dato al proveedor; no altera la cuenta, el Vault, el diario ni otra función, el diagnóstico local de Unified Logging continúa y los informes de crash que TestFlight recopila directamente quedan fuera de este control. En la beta de TestFlight el control está activado por defecto: no es un opt-in y la persona no tiene que activarlo, aunque se le informa antes del primer envío. En Ajustes se presenta junto al control de analítica, en una única sección de privacidad con dos interruptores separados y descripción propia para cada uno. Un único aviso previo al primer envío informa de ambos canales, con la finalidad y las categorías de cada uno y cómo desactivarlos, en cumplimiento conjunto de `RP-ANA-004` y esta regla. Es informativo: se muestra una vez, se cierra con un botón de enterado, ofrece ir a Ajustes y no bloquea el uso de la app; quien actualiza una versión anterior lo ve igualmente una vez, y no se envía nada antes de que se haya mostrado. El modo técnico explícito de `RP-ANA-020` puede validar el transporte con datos sintéticos antes de esa superficie, sin convertir una ejecución ordinaria en una beta activa. Para la beta se usa el copy ES/EN vigente de la app; una revisión posterior puede corregirlo antes del lanzamiento público sin bloquear la implementación ni la distribución beta.

<a id="derechos"></a>
## Soporte y ejercicio de derechos

- **RP-DER-001.** Las solicitudes de derechos siguen el procedimiento aprobado por Legal. La restricción latest-only de una pantalla no limita por sí sola el alcance jurídico de una solicitud.
- **RP-DER-002.** Soporte y los canales de derechos no solicitan contenido íntimo, contraseñas, OTP, claves del Vault ni frases de recuperación por email o ticket.
- **RP-DER-003.** La incapacidad del servidor para descifrar el Vault no cancela una solicitud de acceso o portabilidad. El procedimiento y sus plazos requieren definición Legal.
- **RP-DER-004.** Una exportación legible sólo se ofrece cuando puede generarse y entregarse de forma validada y protegida sin transferir las claves del Vault al servidor.
- **RP-DER-005.** Una solicitud de una tercera persona se gestiona sin revelar indiscriminadamente si existe una ficha sobre ella ni el contenido del diario de otra cuenta.
- **RP-DER-006.** Toda excepción de conservación identifica autorización, finalidad, datos alcanzados y vencimiento. Al vencer, continúa el ciclo de retirada correspondiente.
- **RP-DER-007.** Las comunicaciones públicas describen con precisión la seudonimización, los plazos de destrucción, la estructura legible, los metadatos del Vault, las fotos locales, los backups y los mecanismos de recuperación.

La ubicación de cada dato se concreta en [Cuenta](ACCOUNT.md#datos-cuenta), [Crush](CRUSH.md), [Experiencias](EXPERIENCES.md) y [Vault](VAULT.md#contenido). La conservación y destrucción se definen en [Ciclo de vida](DATA_LIFECYCLE.md). El [expediente Legal](../legal/README.md) mantiene la autoridad jurídica.

<a id="decisiones-pendientes"></a>
## Decisiones pendientes

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-12"></a>DEC-12 | Obtener criterio jurídico sobre minimización estructural, derechos de terceros, portabilidad y comunicaciones de privacidad. | El expediente Legal ya refleja la ubicación de los datos y formula estas preguntas; falta la decisión de la asesoría. |
