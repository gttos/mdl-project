# 5. Guía para la auditoría jurídica

> **Destinatario:** asesoría jurídica, DPO o especialista RGPD.
> **Objetivo:** convertir los hechos del expediente en un análisis jurídico y en medidas concretas de reducción de riesgo, antes de que la API actual reciba tráfico de producción.
> **Límite:** las tablas son borradores de trabajo; MDL no fija aquí bases jurídicas, roles ni conclusiones.

## 1. Encargo propuesto a la asesoría

Se solicita revisar el **estado actual** descrito en este expediente —no un modelo futuro— y entregar:

- calificación de responsables, encargados, corresponsables y terceros;
- mapa de finalidades y bases jurídicas por tratamiento;
- una opinión específica sobre si es prudente subir la API a producción con el nombre del crush en texto plano y el resto del contenido privado protegido por una única clave simétrica de aplicación, o si debe condicionarse a correcciones previas;
- análisis específico de datos relativos a personas sin cuenta;
- evaluación de categorías especiales e inferencias sensibles;
- conclusión motivada sobre la necesidad y alcance de una EIPD;
- requisitos de edad, menores y verificación proporcionada por país;
- Registro de Actividades de Tratamiento;
- política de privacidad, términos y textos de información/aceptación acordes con el estado real, no con un estado aspiracional;
- procedimiento de derechos para usuarios y terceras personas, incluyendo que hoy no existe exportación de datos;
- revisión de proveedores, DPA, transferencias y subencargados, incluyendo la duplicidad de SDK de identidad (Firebase Auth + Amplify/Cognito);
- criterios jurídicos del plan de respuesta a incidentes, distinguiendo el escenario de clave de aplicación comprometida del escenario de ciphertext Vault comprometido;
- lista priorizada de medidas para reducir riesgo antes del lanzamiento.

## 2. Hechos esenciales que no deben perderse

1. MDL almacena información creada por un usuario sobre personas que normalmente no tienen cuenta.
2. El nombre de esa persona se guarda hoy en texto plano en la base de datos de producción.
3. El resto del contenido de contacto, lugar, descripción y nota de cierre está cifrado, pero con una única clave de aplicación (`APP_KEY`) que el backend usa para descifrar automáticamente cada vez que lo necesita; no es cifrado de extremo a extremo.
4. Existe además un Vault E2EE parcialmente activo que cifra una copia del mismo contenido con una clave por usuario que el servidor nunca recibe — pero no reemplaza a los puntos 2 y 3.
5. Un índice de búsqueda local en el dispositivo (FTS5) guarda ese mismo contenido en texto plano, fuera del cifrado del Vault.
6. El texto privado viaja por HTTP en el mismo request que los datos funcionales, con o sin Vault activo.
7. Las fotografías permanecen en el dispositivo, no se excluyen del backup del sistema operativo y no se eliminan automáticamente al borrar el crush.
8. Las métricas se calculan para una persona concreta a partir de su historial; no son resultados anónimos.
9. No existe hoy un flujo de exportación de datos personales (portabilidad).
10. Dos SDK de identidad de terceros (Firebase Auth y AWS Amplify/Cognito) están activos simultáneamente en el cliente iOS.

## 3. Información empresarial y operativa pendiente

- razón social, domicilio, representante y contacto de privacidad;
- países, idiomas, canales de comercialización y edad mínima;
- personal interno y externo con acceso a producción y a `APP_KEY`;
- fecha objetivo para retirar las columnas cifradas con clave de aplicación una vez el Vault esté completo;
- volumen esperado de usuarios, crushes, experiencias y relaciones;
- proceso real de respuesta a derechos e incidentes, más allá de lo documentado en el código.

## 4. Borrador de Registro de Actividades de Tratamiento

| Tratamiento | Interesados | Datos principales | Finalidad técnica/producto | Sistemas/destinatarios | Conservación de referencia | Pregunta jurídica central |
|---|---|---|---|---|---|---|
| Alta y gestión de cuenta | Usuarios | Identidad, contacto, edad, país, credenciales, `google_id` | Crear, autenticar y operar la cuenta | iOS, API-DB, Google OAuth, Firebase, Amplify/Cognito | Vida de cuenta + plazo aprobado | Base, edad mínima, necesidad de dos SDK de identidad simultáneos |
| Autenticación y recuperación de cuenta | Usuarios/dispositivos | Email, hashes, tokens, códigos OTP, IP, user agent | Acceso, recuperación y prevención de abuso | API-DB, SES, Sentry | Expiración; OTP 24h post-expiración | Base, proporcionalidad y plazo |
| Gestión de crushes (estructura funcional) | Usuarios y terceras personas | Estructura, atributos, estados, fechas, valoraciones, **nombre en texto plano** | Diario, filtros, timeline y métricas | iOS, API-DB, AWS RDS | Vida de entidad + purga técnica | Papel de MDL, artículo 14, minimización, y si el nombre debe cifrarse antes de producción |
| Contenido de contacto del crush (cifrado con clave de aplicación) | Usuarios y terceras personas | Email, WhatsApp, Instagram, Facebook, Snapchat, TikTok, fecha de nacimiento | Completar la ficha | API-DB (cifrado con `APP_KEY`), Vault si está activo | Vida de entidad | Suficiencia de una única clave simétrica, transparencia a terceros |
| Gestión de experiencias | Usuarios y terceras personas | Asociación, fecha, satisfacción, lugar y descripción (cifrados con clave de aplicación) | Diario y timeline | API-DB, Vault si está activo | Vida de entidad | Intimidad, minimización y derechos de terceros |
| Gestión de relaciones | Usuarios y terceras personas | Estados, fechas, asociación y nota de cierre (cifrada con clave de aplicación) | Historial relacional y timeline | API-DB, Vault si está activo | Vida de entidad | Intimidad, inferencias y derechos de terceros |
| Índice de búsqueda local en claro | Usuarios y terceras personas | Nombre, profesión, lugar, descripción, nota de cierre en texto plano | Búsqueda local por nombre | Solo dispositivo iOS (`vault_search_index`) | Indefinida; sin poda confirmada | Si constituye una copia adicional no cifrada que deba eliminarse o justificarse |
| Fotografías locales | Usuarios y terceras personas | Imagen re-codificada | Galería privada del crush | Solo dispositivo iOS; incluida en backups del sistema operativo por defecto | Hasta borrado manual; no se borra con el crush | Información al usuario sobre backups del SO, base legal, y corrección del borrado |
| Métricas personales | Usuarios y terceras personas | Historial, atributos, valoraciones y resultados derivados | Autorreflexión y presentación de patrones | API en cálculo, respuesta iOS | Mientras existan fuentes | Perfilado, inferencias, oposición |
| Vault E2EE (parcial) | Usuarios y terceras personas | Ciphertext, encapsulaciones, metadata, devices | Copia cifrada de extremo a extremo del mismo contenido funcional | iOS, API-DB, S3 | Vida de Vault + purga | Cómo describir un Vault parcial sin sobreestimar la protección real ante el usuario |
| Comunicaciones transaccionales | Usuarios | Email, mensaje, entrega, rebote | Verificación, seguridad y servicio | API, Amazon SES | Según proveedor | Base, separación de marketing y plazo |
| Aceptaciones legales | Usuarios | Documento, versión, fecha, IP y user agent | Demostrar presentación/aceptación | API-DB | Plazo probatorio | Qué exige aceptación y cuánto conservar |
| Seguridad y observabilidad | Usuarios/dispositivos | IDs, rutas, IP, user agent, traces, eventos, errores | Prevenir, detectar e investigar | API-DB, Sentry | 7/30/90 días documentados, no auditados en producción | Base, acceso, proporcionalidad, brechas |

La asesoría debe completar responsable, base, destinatarios, transferencias, plazos definitivos, medidas y derechos para cada fila.

## 5. Evaluación de riesgos actuales para orientar la revisión

| Riesgo | Personas afectadas | Escenario | Estado real | Pregunta de riesgo residual |
|---|---|---|---|---|
| Nombre de tercera persona en texto plano | Tercera persona | Cualquier acceso a la base de datos, incluido interno, lee el nombre sin necesitar ninguna clave | Confirmado en código (`Crush.name` sin mutator de cifrado) | ¿Debe bloquearse producción hasta cifrar este campo? |
| Clave de aplicación única compromete a todas las cuentas | Usuario y tercera persona | Exposición o filtración de `APP_KEY` permite descifrar contacto, lugar, descripción y nota de cierre de **todas** las cuentas de una sola vez | Confirmado: `Crypt::encryptString` con `APP_KEY` en `Crush`, `Experience`, `Relationship`, `User.phone` | ¿Es proporcional este diseño mientras el Vault no lo reemplace? ¿Qué rotación y custodia de `APP_KEY` existen? |
| Plaintext privado en índice local | Usuario y tercera persona | Acceso forense o backup del dispositivo expone `vault_search_index` en claro | Confirmado en código y documentación técnica | ¿Debe cifrarse el índice o eliminarse el texto indexado? |
| Plaintext privado en tránsito HTTP | Usuario y tercera persona | El contenido de contacto/lugar/descripción/nota viaja sin cifrado de aplicación hasta la capa TLS, visible a WAF, balanceadores, logs de request si no hay scrubbing | Confirmado en documentación técnica (`decisions/vault-e2ee-architecture.md`) | ¿Qué scrubbing existe realmente en la infraestructura entre TLS y la aplicación? |
| Fotos sin exclusión de backup ni borrado con el crush | Tercera persona (sujeto de la foto) y usuario | Una foto de una persona persiste en backups del sistema operativo y en el dispositivo después de que el usuario cree haber borrado el crush | Confirmado en código | ¿Qué información debe darse al usuario sobre este límite mientras no se corrija? |
| Recovery del Vault borra entradas válidas | Usuario y tercera persona | Un crush con varias experiencias o relaciones pierde entradas del Vault durante una recuperación | Confirmado en documentación técnica | ¿Debe deshabilitarse recovery hasta corregirlo, dado el riesgo de pérdida de datos? |
| Recovery mnemónico no validado | Usuario | La frase de 24 palabras puede no reconstruir el Vault en un dispositivo nuevo | Corrección en código, sin validación end-to-end confirmada | ¿Debe ocultarse esta opción de recuperación hasta validarla? |
| Duplicidad de SDK de identidad | Usuario | Firebase Auth y Amplify/Cognito Auth activos simultáneamente | Confirmado en código | ¿Qué proveedor procesa realmente la autenticación y cuál es remanente? |
| Ausencia de exportación de datos | Usuario | Una solicitud de portabilidad no puede satisfacerse hoy con una herramienta existente | Confirmado (BBP, sección 14.2) | ¿Qué plazo exige la asesoría para construir este flujo? |
| Inferencias sensibles por combinación | Usuario y tercera persona | Estados, fechas, características y valoraciones revelan vida sexual, orientación, salud emocional u otros aspectos | Estructural, independiente del Vault | ¿Qué categorías y medidas reforzadas corresponden? |
| Menores | Usuario y tercera persona | Usuario o persona descrita es menor | Edad de cuenta mínima 18 años verificada en registro; sin verificación equivalente sobre la tercera persona | ¿Qué edad mínima, exclusiones y controles son necesarios para la persona descrita? |

## 6. Preguntas prioritarias antes de producción

### 6.1 Bloqueantes propuestos por el estado técnico

1. ¿Debe cifrarse `Crush.name` antes de que la API reciba tráfico de producción, o es aceptable documentarlo como riesgo asumido con fecha de corrección?
2. ¿Es aceptable, mientras el Vault no esté completo, seguir protegiendo contacto/lugar/descripción/nota únicamente con una `APP_KEY` simétrica compartida por todas las cuentas?
3. ¿Debe deshabilitarse temporalmente la recuperación del Vault hasta corregir el bug de agrupación que borra entradas válidas?
4. ¿Debe comunicarse a los usuarios actuales, antes de producción, que sus fotos no se excluyen de backups del sistema ni se borran con el crush?

### 6.2 Personas, roles y bases

5. ¿Qué entidad será responsable del tratamiento y necesita representante o DPO en la UE?
6. ¿Cuál es la finalidad exacta y la base jurídica de cada tratamiento del borrador de RAT?
7. ¿Qué papel tiene MDL respecto de datos introducidos por un usuario sobre una persona sin cuenta?
8. ¿Qué información debe proporcionarse a esa tercera persona, cuándo y con qué excepciones?

### 6.3 Intimidad, categorías especiales y perfilado

9. ¿Qué campos o combinaciones pueden revelar categorías especiales o información de naturaleza sexual, afectiva o de salud?
10. ¿Las métricas de MDL constituyen perfilado y pueden producir efectos jurídicos o significativamente similares?

### 6.4 EIPD y privacidad desde el diseño

11. Considerando el estado técnico descrito —nombre en claro, clave única de aplicación, índice local en claro, plaintext en tránsito— ¿debe realizarse una EIPD antes del lanzamiento?
12. ¿Qué evidencia permite sostener de forma prudente cualquier afirmación pública sobre cifrado o E2EE mientras el estado sea el descrito en este expediente?

### 6.5 Edad y menores

13. ¿Cuál debe ser la edad mínima por país y qué mecanismo de garantía de edad es proporcionado?
14. ¿Debe impedirse registrar a una tercera persona menor o limitar campos y funciones en ese supuesto?

### 6.6 Transparencia y documentos públicos

15. ¿Qué información de los artículos 13 y 14 debe aparecer en política, onboarding y pantallas contextuales, dado que el sistema no es hoy E2EE completo?
16. ¿Cómo explicar en lenguaje claro y no engañoso la diferencia entre lo que ya está cifrado de extremo a extremo y lo que todavía no?

### 6.7 Derechos y borrado

17. ¿Cómo debe atenderse acceso, rectificación, oposición o supresión de una tercera persona, considerando que su nombre es buscable en texto plano?
18. ¿Qué plazo exige la asesoría para implementar exportación de datos?
19. ¿Son proporcionados los plazos técnicos documentados de 1, 7, 30 y 90 días, y están realmente activos en producción?

### 6.8 Proveedores y transferencias

20. ¿Qué proveedores actúan como encargados, responsables independientes o posibles corresponsables (AWS, Google, Firebase/Google, Sentry)?
21. ¿Debe retirarse uno de los dos SDK de identidad (Firebase Auth o Amplify/Cognito) por no tener una finalidad necesaria y comunicada?

### 6.9 Seguridad e incidentes

22. ¿Qué supuestos activan notificación a autoridad o comunicación a afectados si se compromete la `APP_KEY` y expone datos de todas las cuentas?
23. ¿Cómo debe valorarse una exposición de la `APP_KEY` frente a una exposición de ciphertext del Vault?

## 7. Preguntas específicas para reducir riesgo de producto

1. ¿Qué campos deberían dejar de recibirse por los endpoints funcionales y pasar exclusivamente por el Vault?
2. ¿Conviene bloquear el alta de crushes hasta que el nombre esté cifrado?
3. ¿Qué controles permiten al usuario ver rápidamente qué está protegido por el Vault y qué no?
4. ¿Debe mostrarse una advertencia mientras el Vault sea parcial, en vez de presentarlo como cifrado de extremo a extremo completo?
5. ¿Qué revisión periódica debe detectar cambios de scope, SDK, campos o inferencias antes del lanzamiento definitivo?

## 8. Paquete de evidencia para la asesoría

| Evidencia | Responsable interno | Estado esperado antes de firma |
|---|---|---|
| Confirmación de qué columnas siguen sin cifrar (`Crush.name`) y plan de corrección | Ingeniería | Pendiente |
| Informe de red mostrando el contenido privado viajando por HTTP en el request funcional | iOS/API | Pendiente |
| Inspección de `vault_search_index` en dispositivo de prueba | iOS | Pendiente |
| Prueba de recovery por frase de 24 palabras en dispositivo limpio | iOS/API | Pendiente |
| Confirmación de cuál SDK de identidad procesa autenticación en producción | iOS | Pendiente |
| Configuración de logs y scrubbing de Sentry para los campos identificados | Seguridad/operaciones | Pendiente |
| Matriz de proveedores, DPA y transferencias (AWS, Google, Sentry, Apple) | Legal/operaciones | Pendiente |
| Plan y fecha de retiro de columnas cifradas con clave de aplicación | Producto/ingeniería | Pendiente |

## 9. Fuentes oficiales para la revisión

- [Reglamento (UE) 2016/679 — texto oficial en español](https://eur-lex.europa.eu/eli/reg/2016/679/oj?locale=es).
- [AEPD — realización de evaluaciones de impacto](https://www.aepd.es/derechos-y-deberes/cumple-tus-deberes/medidas-de-cumplimiento/realizacion-de-evaluaciones-de).
- [AEPD — lista de tratamientos que requieren EIPD](https://www.aepd.es/sites/default/files/2019-09/listas-dpia-es-35-4.pdf).
- [AEPD — gestión del riesgo y evaluación de impacto](https://www.aepd.es/guias/gestion-riesgo-y-evaluacion-impacto-en-tratamientos-datos-personales.pdf).

La lista de la AEPD no sustituye el análisis concreto de naturaleza, alcance, contexto y fines. La asesoría debe dejar motivada su conclusión sobre la EIPD y las medidas resultantes.
