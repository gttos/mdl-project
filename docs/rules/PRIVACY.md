# Reglas de producto — Privacidad, seguridad y derechos

<a id="decisiones-pendientes"></a><a id="dec-12"></a>

[Índice de reglas](../PRODUCT.md) · Aislamiento entre cuentas, protección de información, diagnóstico, soporte y ejercicio de derechos.

<a id="propiedad"></a>
## Propiedad de los datos y privacidad del diario

- **RP-BASE-002.** Cada cuenta sólo accede y modifica sus propios datos. Conocer un identificador ajeno no concede acceso ni permite descubrir su contenido.
- **RP-BASE-005.** El contenido íntimo del diario se cifra de extremo a extremo. El servidor conserva la estructura mínima necesaria para las funciones aprobadas.
- **RP-ID-007.** Cuenta, Vault, claves, preferencias, fotos y pendientes están separados por cuenta. Cambiar de cuenta nunca adopta datos de otra.
- **RP-ID-009.** La demo utiliza exclusivamente datos ficticios locales separados de cualquier diario. Su consulta conserva inaccesibles cuentas ajenas, claves y contenido privado, y los ejemplos permanecen fuera del diario real.

<a id="seguridad"></a>
## Protección del acceso y de la información

- **RP-SEG-001.** Toda operación privada comprueba una sesión válida y la propiedad de cada cuenta, Crush, experiencia, período, hito, Vault, blob, foto, operación pendiente y URL temporal que utilice.
- **RP-SEG-002.** La aplicación evita acciones inválidas y la API vuelve a comprobar propiedad e invariantes antes de aplicar una modificación. La ausencia de un botón no sustituye la validación de la API.
- **RP-SEG-003.** Un rechazo conserva íntegramente el estado anterior y no revela datos de otra cuenta. Los campos ajenos al contrato de la operación no cambian identidad, propiedad, permisos ni estado.
- **RP-SEG-004.** Login, verificación y recuperación presentan respuestas públicas que no permiten distinguir si una cuenta existe, está inactiva o utilizó una credencial incorrecta.
- **RP-SEG-005.** Los flujos de autenticación, OTP, Apple, URLs temporales y escrituras del Vault aplican límites de uso por su propio alcance y comunican cuándo puede volver a intentarse.
- **RP-SEG-006.** Logs, errores y analítica excluyen contraseñas, OTP, tokens, credenciales Apple, emails legibles, contenido del Vault, snapshots privados, fotografías y sus bytes cifrados o descifrados, claves de media, nonces y tags, URLs firmadas completas, tokens de búsqueda del Vault y payloads completos de solicitudes o respuestas.
- **RP-SEG-007.** El diagnóstico se construye mediante una lista positiva de contexto técnico: identificador de traza o correlación; plantilla de ruta y método; código de estado o error saneado; fecha, duración y conteo; versión de aplicación y sistema; e identificadores técnicos mínimos cuando sean necesarios para investigar. IP y `user_agent` sólo forman parte de eventos de autenticación, abuso o seguridad. Sus plazos máximos se definen en [Plazos de conservación y destrucción](DATA_LIFECYCLE.md#plazos).
- **RP-SEG-008.** Todo objeto privado y toda capacidad temporal quedan vinculados a su cuenta propietaria y respetan los [límites del Vault](VAULT.md#inventario-servidor): 65.536 bytes por objeto y 25 MiB lógicos por cuenta. La capacidad temporal reserva cuota sin duplicarla en los reintentos.
- **RP-SEG-009.** La IP y el esquema utilizados para seguridad sólo aceptan cabeceras de reenvío cuando proceden de un proxy expresamente confiable.
- **RP-SEG-010.** Los secretos operativos se entregan por el mecanismo autorizado del entorno y permanecen fuera del código, imágenes, artefactos, logs y resultados de pruebas. El acceso de personal utiliza el mínimo permiso necesario y queda trazado.
- **RP-SEG-011.** La observabilidad técnica y la analítica de producto son canales distintos. Un diagnóstico utiliza un código estable, severidad, categoría, versión/build, entorno, `traceId` y sólo el contexto técnico permitido por `RP-SEG-007`; nunca reutiliza eventos analíticos ni depende de que la persona mantenga activada la analítica. Cuando un fallo iOS corresponde a una petición API, cliente, respuesta, logs backend y reporte de error conservan el mismo `traceId` correlacionable.
- **RP-SEG-012.** Sentry EU es el receptor remoto seleccionado para errores iOS y se activa en la beta 0.2 distribuida por TestFlight y en el modo técnico explícito de `RP-ANA-020`, con una retención máxima comprobada de 30 días. Su configuración impide session replay, screenshots, adjuntos, perfiles, trazas de rendimiento, breadcrumbs automáticos de UI/red y captura automática de requests, responses, headers, queries, variables locales o contexto personal. Mientras su control ([`RP-SEG-013`](#seguridad)) esté activo, sólo recibe crashes no manejados y eventos `warning`/`error` incluidos en una allowlist comprobable; su ausencia o fallo no altera el comportamiento de MDL.
- **RP-SEG-013.** El diagnóstico remoto y la analítica tienen controles independientes. Desactivar diagnóstico detiene todos los envíos de MDL al proveedor, incluidos crashes, y conserva el diagnóstico local y las funciones del diario. En TestFlight ambos controles están activos por defecto después de mostrar una información previa, una vez, sobre finalidad, categorías y desactivación; el uso de la aplicación permanece disponible. Antes de informar no se realiza ningún envío ordinario. Los informes recogidos directamente por TestFlight conservan su control propio. La validación técnica explícita conserva datos sintéticos y las restricciones del mismo tratamiento.

<a id="derechos"></a>
## Soporte y ejercicio de derechos

- **RP-DER-001.** Las solicitudes de derechos siguen el procedimiento aprobado por Legal. La restricción latest-only de una pantalla no limita por sí sola el alcance jurídico de una solicitud.
- **RP-DER-002.** Soporte y los canales de derechos no solicitan contenido íntimo, contraseñas, OTP, claves del Vault ni frases de recuperación por email o ticket.
- **RP-DER-003.** La incapacidad del servidor para descifrar el Vault no cancela una solicitud de acceso o portabilidad. El procedimiento y sus plazos requieren definición Legal.
- **RP-DER-004.** Una exportación legible sólo se ofrece cuando puede generarse y entregarse de forma validada y protegida sin transferir las claves del Vault al servidor.
- **RP-DER-005.** Una solicitud de una tercera persona se gestiona sin revelar indiscriminadamente si existe una ficha sobre ella ni el contenido del diario de otra cuenta.
- **RP-DER-006.** Toda excepción de conservación identifica autorización, finalidad, datos alcanzados y vencimiento. Al vencer, continúa el ciclo de retirada correspondiente.
- **RP-DER-007.** Las comunicaciones públicas describen con precisión el borrado integral, los plazos de destrucción, la estructura legible, los metadatos del Vault, los backups y los mecanismos de recuperación. No se anuncia el cumplimiento de RP-BAJ-017 antes de verificar su implementación y todas las copias recuperables afectadas. Para las fotos distinguen expresamente cifrado local derivado de la VMK de E2EE remoto: MDL no recibe sus archivos, pero la app los descifra temporalmente en el dispositivo autorizado y recuperarlos en otro dispositivo no es posible sin una copia remota.
- **RP-DER-008.** Registro, consulta desde Ajustes y web pública leen los Términos y la Política vigentes de la API y muestran la versión del documento que están presentando. El contenido jurídico y sus traducciones se mantienen en la base de datos por el responsable, sin copiar los anexos de trabajo en la app ni sustituir partes del texto desde las plantillas. Ajustes no conserva un Aviso Legal local paralelo ni la explicación antigua de «Datos cifrados»: la información sobre protección utiliza la ayuda única de [RP-VLT-037/038](VAULT.md). Esta reconciliación no modifica las aceptaciones ni acredita aprobación jurídica o publicación de los textos definitivos.
- **RP-DER-009.** Los correos de verificación, recuperación, aceptación de baja, inactividad y avisos de leads identifican al titular de MDL y enlazan a los Términos y la Política vigentes. La identidad pública utilizada por las plantillas web y de correo tiene una única fuente estructurada en la API; la app no conserva otra ficha local del titular. Los datos incluidos en el cuerpo de los documentos jurídicos pertenecen a esos textos y se mantienen en la base de datos. Los enlaces no incorporan datos de cuenta ni seguimiento; se conserva el mecanismo de oposición propio de los avisos de leads, sin añadir una baja comercial a los correos transaccionales.

La ubicación de cada dato se concreta en [Cuenta](ACCOUNT.md#datos-cuenta), [Crush](CRUSH.md), [Experiencias](EXPERIENCES.md) y [Vault](VAULT.md#contenido). La conservación y destrucción se definen en [Ciclo de vida](DATA_LIFECYCLE.md). El [expediente Legal](../legal/README.md) mantiene la autoridad jurídica.

<a id="leads-web"></a>
## Web pública y solicitudes de acceso

- **RP-LEAD-001.** La web pública permite solicitar acceso a la beta; no descarga directamente una demostración ni convierte el lead en una cuenta de MDL. Explica qué es MDL y permite pedir acceso como tester, sin prometer acceso inmediato por enviar la solicitud.
- **RP-LEAD-002.** El email del lead sólo se utiliza para comunicar una vez la disponibilidad de la beta y, si corresponde, una vez la disponibilidad de la versión pública. La solicitud no autoriza newsletter, campañas periódicas, publicidad ni otras finalidades.
- **RP-LEAD-003.** Enviar la solicitud exige una casilla inicialmente desmarcada mediante la que la persona declara haber leído y aceptado los Términos y Condiciones y la Política de Privacidad enlazados. Una aceptación ausente o referida a versiones que ya no son las vigentes falla sin crear ni actualizar el lead.
- **RP-LEAD-004.** La primera aceptación válida de cada combinación de versiones de Términos y Política para un mismo email conserva su fecha y hora, el idioma y las versiones exactas aceptadas. La finalidad se explica en la capa informativa y no se solicita como dato de formulario. No se vinculan al lead dirección IP ni `user_agent`.
- **RP-LEAD-005.** El email se almacena cifrado de forma recuperable para los dos envíos autorizados y dispone de un índice exacto no reversible para unicidad. No aparece legible en logs, diagnósticos ni eventos de observabilidad.
- **RP-LEAD-006.** Repetir una solicitud para el mismo email conserva una única identidad de lead, reconocida mediante el índice exacto no reversible sin descifrar el email para compararlo. Si las versiones aceptadas de Términos y Política coinciden con una combinación ya registrada, no añade ni modifica el recibo original, aunque cambien el día o el idioma. Si cambia cualquiera de esas versiones y la persona acepta la nueva combinación vigente, registra un nuevo recibo. Ninguna repetición altera el estado de entregas ya realizadas. Nunca se fabrica ni se completa retrospectivamente una aceptación que no haya sido enviada por la persona.
- **RP-LEAD-007.** Cada uno de los dos avisos autorizados es idempotente: un lead válido puede recibir como máximo un aviso de beta y uno de lanzamiento público. Un fallo ambiguo se resuelve mediante estado durable y no mediante un reenvío ciego.
- **RP-LEAD-008.** Cada aviso ofrece un mecanismo sencillo de oposición. La oposición impide cualquier envío posterior, retira el lead y su evidencia del uso activo y no exige crear una cuenta ni revelar una dirección de email en la URL.
- **RP-LEAD-009.** Una fila anterior que carezca de evidencia versionada no se considera consentida para ninguno de los avisos. Debe obtenerse una nueva solicitud válida o excluirse de los envíos.

### Escenarios de aceptación de leads

Estos escenarios verifican las reglas anteriores; no las redefinen ni implican despliegue. Una fila sin evidencia es un caso defensivo de exclusión, no una solicitud de migración de leads anteriores.

| ID | Dado / Cuando / Entonces | Regla |
|---|---|---|
| GWT-LEAD-01 | Dada la landing, cuando se visita, entonces permite entender la solicitud de beta, consultar ambos documentos y enviar el formulario sin descargar ni crear una cuenta. | RP-LEAD-001 |
| GWT-LEAD-02 | Dada la casilla desmarcada, cuando falta aceptación o las versiones ya no son vigentes, entonces se rechaza la solicitud sin crear ni actualizar datos. | RP-LEAD-003 |
| GWT-LEAD-03 | Dada una solicitud válida, cuando se confirma, entonces hay un lead con email cifrado, lookup no reversible y evidencia de versiones, fecha, hora e idioma, sin IP ni user agent vinculados. | RP-LEAD-004..006 |
| GWT-LEAD-04 | Dado el mismo email, cuando acepta las mismas versiones, entonces conserva identidad y recibo originales aunque cambien fecha o idioma; una nueva combinación vigente aceptada añade otro recibo sin alterar entregas ni fabricar aceptación. | RP-LEAD-006/009 |
| GWT-LEAD-05 | Dado un lead válido no opuesto, cuando se abren los avisos autorizados o existe un resultado ambiguo, entonces recibe como máximo uno de beta y uno de lanzamiento, sin reenvío ciego. | RP-LEAD-002/007 |
| GWT-LEAD-06 | Dado un aviso o el canal de privacidad, cuando la persona se opone, entonces cesan los envíos y el lead deja el sistema activo sin lista paralela de supresión. | RP-LEAD-008, RP-CON-004 |
| GWT-LEAD-07 | Dada una fila sin evidencia versionada, cuando se prepara un aviso, entonces queda excluida hasta que exista una nueva solicitud válida. | RP-LEAD-009 |
| GWT-LEAD-08 | Dado un fallo de cifrado, persistencia, evidencia, cola o proveedor, cuando no puede confirmarse la operación completa, entonces no se declara aceptada ni enviada ni quedan estados parciales que habiliten otro uso. | RP-SEG-006..011, RP-LEAD-003/007 |
