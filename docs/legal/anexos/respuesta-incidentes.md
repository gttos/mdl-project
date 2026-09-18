# Anexo. Procedimiento de respuesta a incidentes de privacidad

> **Estado:** base operativa para completar con nombres, contactos, proveedores y criterio jurídico.  
> **Fecha de corte:** 14 de septiembre de 2026.
> **Alcance:** cuenta y verificación, identidad Apple, datos funcionales, Vault, dispositivos, fotografías locales, proveedores, logs, datos conservados tras la baja y backups.
> **Modelo de lanzamiento:** el contenido privado del diario existe sólo en Vault E2EE; la clave de aplicación del backend cifra el teléfono y otros campos expresamente inventariados, no el email ni una segunda copia del contenido Vault.

## 1. Objetivos

- proteger a las personas afectadas;
- contener el incidente sin destruir evidencia;
- determinar con rapidez qué datos y sistemas están implicados;
- distinguir exposición de plaintext, datos funcionales, metadata, ciphertext y claves;
- cumplir los plazos y comunicaciones que determine la asesoría;
- recuperar el servicio de forma segura y evitar repetición.

## 2. Qué se considera una señal de incidente

- acceso no autorizado a cuenta, API, base de datos, bucket o consola;
- vulnerabilidad que permita leer o modificar datos de otra cuenta;
- aparición de nombre, contacto, lugar, descripción o nota de cierre en logs, requests o reportes;
- exposición de fases, etiquetas, cierres, historiales, fechas, ratings, asociaciones o secuencias relacionales de otra cuenta;
- exposición o uso indebido de OTP, continuación Apple, `sub`, Private Relay, credenciales o notificaciones federadas;
- exposición de VMK, secretos de recuperación, tokens, URLs firmadas o claves administrativas;
- pérdida, corrupción o borrado indebido de blobs;
- restauración que reactive datos ya borrados;
- acceso o reidentificación de filas conservadas después de una baja;
- proveedor que notifique acceso, pérdida o transferencia no prevista;
- dispositivo perdido o comprometido comunicado por un usuario;
- fotografía local accesible fuera de los controles esperados;
- indisponibilidad que impida recuperar el Vault o ejercer derechos.

## 3. Clasificación técnica inicial

| Clase | Ejemplo | Pregunta inmediata |
|---|---|---|
| A. Cuenta/datos funcionales | Acceso a identidad, verificación, fases, fechas o valoraciones | ¿Qué cuentas, campos y acciones quedaron expuestos? |
| B. Plaintext privado | Texto descifrado en log, memoria persistida o request | ¿Quién pudo leerlo y durante cuánto tiempo? |
| C. Ciphertext | Objetos cifrados copiados o publicados | ¿Se expusieron también metadata, encapsulaciones o secretos? |
| D. Claves/recuperación | VMK, secreto, clave privada o credencial administrativa | ¿Puede revocarse o rotarse sin pérdida de acceso? |
| E. Metadata | IDs, tamaños, secuencias, devices, IP o patrones | ¿Permite identificar, correlacionar o inferir actividad? |
| F. Integridad/disponibilidad | Blob corrupto, borrado o recovery incompleto | ¿Existe copia válida y quién perdió acceso? |
| G. Fotografía local | Acceso desde dispositivo o backup no esperado | ¿MDL controla alguna copia o solo el dispositivo? |
| H. Proveedor/identidad federada | Brecha, subencargado, región o evento Apple no previsto | ¿Qué contrato, datos, credenciales, fechas y acciones aplican? |

Una exposición de ciphertext no debe cerrarse automáticamente como inocua. Deben evaluarse algoritmo, claves, calidad del secreto, metadata, capacidad de fuerza bruta, duración de confidencialidad esperada y posibilidad de combinar el evento con otros accesos.

## 4. Roles que deben asignarse

| Rol | Responsabilidad | Titular/contacto |
|---|---|---|
| Incident lead | Coordinar, abrir cronología y aprobar acciones técnicas | Pendiente |
| Seguridad/ingeniería | Contener, investigar, preservar y recuperar | Pendiente |
| Privacidad/legal | Calificar la brecha, plazos, autoridad y comunicaciones | Pendiente |
| Operaciones/proveedores | Ejecutar acciones de infraestructura y obtener evidencias | Pendiente |
| Producto/soporte | Identificar impacto de usuario y preparar canales | Pendiente |
| Dirección | Decidir recursos, riesgo empresarial y comunicación externa | Pendiente |

Debe existir una guardia o mecanismo de escalado fuera de horario y un sustituto para cada rol crítico.

## 5. Cronología operativa

### 5.1 Primeros 30 minutos

1. Abrir un ID de incidente y registrar hora de conocimiento.
2. Nombrar incident lead y contacto jurídico.
3. Preservar logs, configuraciones, alertas y evidencias volátiles.
4. Limitar acceso al canal de coordinación.
5. Clasificar provisionalmente sistemas y tipos de datos.
6. Aplicar contención reversible cuando no aumente el daño.

### 5.2 Primeras 4 horas

1. Delimitar inicio, final o continuidad del acceso.
2. Identificar cuentas, terceros, países y proveedores afectados.
3. Distinguir plaintext, datos funcionales, metadata, ciphertext y claves.
4. Rotar credenciales o revocar sesiones comprometidas de forma controlada.
5. Contactar al proveedor conforme al contrato.
6. Entregar un primer informe factual a privacidad/legal.
7. Registrar qué se sabe, qué se infiere y qué falta verificar.

### 5.3 Primeras 24 horas

1. Construir una cronología con fuentes de evidencia.
2. Cuantificar personas, registros, categorías y duración.
3. Analizar confidencialidad, integridad, disponibilidad y posibles consecuencias.
4. Verificar si el atacante pudo relacionar ciphertext con cuentas o recuperar claves.
5. Evaluar riesgo de fraude, exposición íntima, acoso, discriminación o daño reputacional.
6. Preparar opciones de notificación sin adelantar conclusiones.
7. Definir recuperación, monitorización reforzada y soporte a afectados.

### 5.4 Antes de 72 horas desde el conocimiento

La asesoría debe determinar si se cumplen los requisitos de notificación del artículo 33 RGPD, cuál es la autoridad competente y qué información está disponible. Si procede notificar sin disponer de todos los hechos, debe definir la comunicación inicial y las ampliaciones posteriores.

La hora de conocimiento, el razonamiento, la decisión jurídica y cualquier causa de demora deben documentarse. El plazo no debe calcularse desde el cierre de la investigación.

### 5.5 Después de la contención

1. Erradicar la causa y validar que no persiste el acceso.
2. Recuperar desde copias verificadas sin reintroducir datos borrados o vulnerabilidad.
3. Monitorizar indicadores de repetición.
4. Atender consultas y derechos de personas afectadas.
5. Registrar medidas correctivas, responsables y fechas.
6. Actualizar inventario, EIPD, contratos, política o controles cuando corresponda.
7. Realizar una revisión posterior sin culpa y verificar el cierre de acciones.

## 6. Información mínima para la evaluación jurídica

- fecha y hora de detección y de conocimiento organizativo;
- fuente del aviso y fiabilidad;
- naturaleza y causa provisional;
- sistemas, entornos y proveedores;
- categorías de personas y número aproximado;
- categorías y volumen de datos;
- si existió plaintext, claves, recovery, metadata o solo ciphertext;
- duración y capacidad de acceso, copia, modificación o borrado;
- países de afectados y autoridad potencialmente competente;
- consecuencias probables para las personas;
- medidas aplicadas y eficacia;
- contactos y próximos hitos.

## 7. Preservación de evidencia

- exportar logs de forma íntegra y registrar hash, origen, zona horaria y custodio;
- conservar configuración, versiones, permisos, eventos de consola y cambios;
- no copiar contenido privado real a tickets o chats generales;
- usar muestras minimizadas o identificadores opacos;
- documentar cada rotación, bloqueo, restauración y borrado;
- separar hechos observados de hipótesis;
- limitar acceso por necesidad y registrar quién consultó el expediente.

## 8. Comunicación a personas afectadas

Si la asesoría determina que corresponde comunicar, el texto debe ser claro y contener como mínimo lo que exija el artículo 34 RGPD. Operativamente debería explicar:

- qué ocurrió y cuándo;
- qué tipos de datos están implicados;
- qué consecuencias razonables se contemplan;
- qué hizo MDL para contener y corregir;
- qué puede hacer la persona, con instrucciones concretas;
- cómo contactar con privacidad y soporte;
- qué información se actualizará después.

No debe describirse el contenido como anónimo por estar cifrado ni afirmar que no existe riesgo sin un análisis documentado.

## 9. Escenarios específicos de seguridad

### Clave de aplicación (`APP_KEY`) comprometida

- Este escenario es distinto de una exposición de Vault: en la beta `APP_KEY` cifra el teléfono y cualquier otro campo backend incluido expresamente, no el email ni el contenido privado del diario.
- Confirmar de inmediato si la clave expuesta corresponde a producción y si permite descifrado retroactivo de backups o réplicas.
- Rotar `APP_KEY` requiere re-cifrar todas las columnas afectadas antes de invalidar la clave anterior; planificar esa migración como parte de la preparación, no durante el incidente.
- Cuantificar cuentas y campos afectados; no asumir que E2EE del diario protege esos datos de cuenta.
- Verificar que no existan columnas legacy de contenido Vault. Si aparecieran, el incidente también demuestra que el gate de lanzamiento falló y su alcance debe ampliarse.

### Exposición de estructura relacional

- identificar fases, etiquetas, motivos de cierre, historiales, hitos, fechas, ratings, rasgos, asociaciones, `everMatched` y correlaciones expuestos;
- determinar si permiten inferir vida afectiva o sexual aunque no exista texto Vault;
- incluir a terceras personas descritas al cuantificar afectados y consecuencias;
- evaluar secuencias y combinaciones singulares que aumenten el riesgo de reidentificación;
- tratar la exposición como incidente de datos personales aunque el Vault ciphertext permanezca seguro.

### Credencial Apple, OTP o continuación expuestos

- identificar si el dato permite autenticar, vincular cuentas, consultar el estado federado o revocar una autorización;
- revocar sesiones MDL afectadas, invalidar OTP/continuaciones y rotar secretos del proveedor cuando corresponda;
- comprobar firma, `nonce`, audience, uso único, expiración y registros de revocación sin copiar tokens o JWS al expediente;
- evaluar exposición de email o Private Relay y cualquier correlación con el `sub` protegido;
- coordinar con Apple o el proveedor de email mediante el canal contractual y conservar sólo evidencia técnica minimizada.

### Ciphertext sin claves conocidas

- confirmar algoritmo, versión y autenticidad;
- revisar si se expusieron encapsulaciones y parámetros;
- evaluar fortaleza de secretos y riesgo de fuerza bruta;
- comprobar metadata correlacionable y duración de sensibilidad;
- preservar hashes e IDs sin difundir objetos innecesariamente.

### Secreto de recuperación o VMK expuestos

- determinar Vaults afectados y posibilidad de acceso histórico;
- bloquear nuevas sesiones o dispositivos no autorizados;
- diseñar rotación/re-cifrado que no destruya datos;
- informar sobre consecuencias y acciones del usuario según criterio jurídico;
- verificar backups y copias descargadas.

### Corrupción o borrado indebido

- detener purgas y procesos que amplíen la pérdida;
- verificar última versión íntegra por tipo e ID;
- restaurar de forma no destructiva;
- conciliar API-DB, objetos y dispositivos;
- documentar entradas irrecuperables y usuarios afectados.

### Reidentificación de datos posteriores a la baja

- aislar el dataset y suspender nuevas consultas o exportaciones;
- identificar qué UUIDs, fechas, secuencias o tablas auxiliares permitieron volver a vincular personas;
- determinar si el conjunto se declaró anónimo, seudónimo o agregado y qué controles se prometieron;
- revisar accesos, receptores, copias y resultados derivados;
- aplicar la evaluación de brecha a las personas usuarias y terceras personas afectadas;
- corregir la estrategia de agregación, separación o supresión antes de reanudar el uso.

## 10. Escenario de dispositivo

MDL puede revocar tokens o dispositivos remotos cuando recibe aviso, pero no garantiza borrar un dispositivo offline. Soporte debe indicar al usuario cómo:

- cerrar sesiones y revocar el dispositivo;
- cambiar credenciales relevantes;
- usar funciones del sistema para localizar o borrar el dispositivo;
- recuperar el Vault en un dispositivo seguro;
- evaluar exposición de fotografías locales.

El procedimiento no debe pedir al usuario que envíe su frase o contraseña de recuperación.

## 11. Evidencia de preparación

- directorio actualizado de contactos y suplentes;
- plantilla interna de cronología y evaluación;
- acceso de emergencia con MFA y registro;
- export probado de logs de API, base, objetos y proveedores;
- contrato y canal urgente por proveedor;
- simulación anual que incluya datos funcionales, ciphertext y secreto;
- simulación de exposición de estructura relacional y reidentificación de datos conservados tras la baja;
- simulación de compromiso de OTP o identidad Apple, incluida revocación y cierre de sesiones;
- borrador de notificación a autoridad y de comunicación a afectados;
- verificación del plazo de detección y escalado;
- seguimiento de acciones correctivas hasta evidencia de cierre.

## 12. Preguntas para la asesoría

1. ¿Qué evento constituye “conocimiento” a efectos del cómputo de plazos?
2. ¿Quién realiza y aprueba la evaluación de riesgo para derechos y libertades?
3. ¿Qué autoridad o autoridades serían competentes según los países de lanzamiento?
4. ¿Cómo debe valorarse una exposición de ciphertext y metadata sin claves conocidas?
5. ¿Qué umbral exige comunicar directamente a las personas?
6. ¿Qué información puede comunicarse por fases y cómo se documenta una demora?
7. ¿Qué plazo de conservación corresponde al expediente de incidente y su evidencia?
8. ¿Qué compromisos contractuales deben exigirse a cada proveedor?
9. ¿Cómo se califica una reidentificación de datos que se habían presentado como anónimos?
10. ¿Cómo deben considerarse y comunicarse los riesgos para terceras personas descritas que no tienen cuenta?

Referencia: [RGPD, texto oficial en EUR-Lex](https://eur-lex.europa.eu/eli/reg/2016/679/oj?locale=es).
