# Decisiones pendientes para Jira

Preguntas abiertas separadas del catálogo funcional. Su traslado no las resuelve ni autoriza tratamientos, excepciones o implementaciones que dependan de ellas.

## Modelo relacional

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-13"></a>DEC-13 | Obtener validación jurídica del modelo relacional actual, incluidas sus inferencias, minimización, tratamiento de terceras personas y ejercicio de derechos. | El expediente Legal ya describe las situaciones, la evidencia de match, la Experience obligatoria y el borrado compuesto `latest-only`. Falta el criterio de la asesoría; Producto no lo sustituye. |

## Privacidad y derechos

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-12"></a>DEC-12 | Obtener criterio jurídico sobre minimización estructural, derechos de terceros, portabilidad y comunicaciones de privacidad. | El expediente Legal ya refleja la ubicación de los datos y formula estas preguntas; falta la decisión de la asesoría. |

## Conservación y restauración

Estas decisiones siguen abiertas. La documentación técnica y el comportamiento implementado son referencias y no las resuelven por sí solos.

| ID | Decisión pendiente | Lo ya fijado |
|---|---|---|
| <a id="dec-02"></a>DEC-02 | Tratamiento y salvaguardas de una eventual función futura de agregados entre cuentas. | La beta elimina íntegramente los datos de cuentas dadas de baja; no conserva una fuente seudonimizada para esa función. El schema y los modelos reservados permanecen dormantes y no autorizan retención ni cálculo de resultados. Cualquier tratamiento futuro exige nueva definición y aprobación Legal antes de activarse. |
| <a id="dec-14"></a>DEC-14 | Conservación temporal de backups después de la baja y del registro mínimo necesario para impedir la reactivación al restaurar. | Se autoriza diseñar y probar con datos sintéticos RP-BCK-001..007; DynamoDB es el destino técnico elegido para el registro. La propuesta distingue borrado de sistemas activos a las 07:00 del día siguiente y backups MySQL con retención configurada de siete días, incluidos los textos cifrados de RP-VLT-039. Faltan aprobación jurídica, límite material comprobable, fundamento y duración del registro y sus copias, accesos y lenguaje público. S3 queda reservado a fotografías futuras de RP-VLT-040, fuera del tratamiento activo y de una nueva política de backups de esta iniciativa. No modifica por sí sola RP-BAJ-017, RP-CON-003 ni Legal. |

Para la revisión Legal de `DEC-02` deben obtenerse respuestas explícitas sobre: si la estructura y los resultados derivados pueden considerarse anónimos o continúan siendo seudónimos; la base y finalidad aplicables a personas usuarias y terceras personas; qué población puede contribuir antes y después de una baja; el tamaño mínimo de cohorte y las reglas de supresión; los campos permitidos; la separación y los roles de acceso; la frecuencia de actualización; la retención y purga tanto de fuentes como de resultados; el tratamiento de filas agregadas históricas que ya pudieran existir; y las salvaguardas frente a singularización o reidentificación.

<a id="neg-01"></a>
## Adjetivos negativos

1. Lista cerrada: número de entradas, etiquetas y definiciones ES/EN, bloque, orden e IDs; decidir cuáles antiguas conservar y cuáles descartar.
2. Presentación: opciones mezcladas dentro de cada bloque o un control para consultar las adicionales. Comparar variantes antes de modificar la UI y conservar selecciones al ocultarlas.
3. Selección: confirmar si se mantienen tres por bloque y doce en total; decidir si alguna pareja concreta es incompatible o si pueden coexistir apreciaciones diferentes. No introducir exclusividades por analogía.
4. Estadísticas: confirmar si se integran en las distribuciones actuales o se presentan por separado, y el texto de los títulos. No inventar índices de compatibilidad ni recalcular datos en el cliente.

<a id="baja-email"></a>
## Acreditación de baja por email

La elección funcional de los cuatro pasos y los textos está cerrada. Faltan criterio jurídico sobre la suficiencia de la revisión manual y la vía excepcional; plazo y retirada de gestiones sin confirmar, evidencia y buzones; y comprobación de remitente/Reply-To atendidos, nombre visible en Apple y respuesta de Relay. EBD-01 documenta estas dependencias sin resolverlas por costumbre o plazos inventados. DEC-14 permanece en su iniciativa.



<a id="otp"></a>
## Retención y reenvíos OTP

Definir la conservación de OTP y la duración y reinicio de la escalada de reenvíos. La limpieza vigente conserva garantías de bloqueo y espera; su existencia no aprueba un plazo de conservación.

<a id="activacion-google"></a>
## Activación de Google

Resolver la revisión humana/legal, configuración de proveedor y Google Play, credenciales reales, disclosures y validación física antes de distribución. El desarrollo aceptado y sus fases pendientes se conservan en el plan activo.

[Funciones futuras aprobadas](FUNCIONES_FUTURAS_PARA_JIRA.md).

<a id="eventos-inactivos"></a>
## Medición definida sin emisión activa

Decidir la activación de selección y borrado de fotos y visitas genéricas adicionales. Los resultados vigentes usan sus eventos específicos; la existencia de una definición técnica no habilita emisiones.

<a id="integracion-llm"></a>
## Integración LLM

Definir finalidad, contexto mínimo autorizado, proveedor, consentimiento y retención antes de conectar el prototipo o utilizar datos reales. Su existencia no autoriza enviar texto privado del Vault desde API o aplicaciones. Una integración requiere contexto estructural mínimo o contenido expresamente autorizado, sin acceso directo al Vault, bases locales o credenciales; sus salidas auxiliares no modifican automáticamente el historial.
