# 5. Guía para la auditoría jurídica

> Brief para asesoría basado en el modelo funcional objetivo de la beta a 21 de septiembre de 2026. Las conclusiones y textos finales corresponden al abogado o DPO, no a ingeniería.

## 1. Encargo propuesto

Revisar MDL antes del lanzamiento en España y el EEE —o en los países que confirme el responsable— y emitir criterios accionables sobre:

- responsable, encargados, destinatarios, representante y DPO;
- alcance territorial y países de las personas afectadas;
- finalidades, bases y compatibilidad de usos;
- información a personas usuarias y terceras personas descritas;
- categorías especiales, inferencias y necesidad de EIPD;
- edad mínima y mecanismo proporcionado de comprobación;
- privacidad por diseño, necesidad y minimización;
- conservación, borrado, backups y estructura posterior a la baja;
- acceso, rectificación, oposición, restricción, supresión y portabilidad;
- proveedores, transferencias, subencargados y medidas contractuales;
- seguridad, incidentes y lenguaje público sobre cifrado;
- control jurídico de cualquier ampliación futura de datos o proveedores.

## 2. Hechos esenciales

1. MDL es un diario privado, no una red social, y no publica perfiles.
2. La persona usuaria registra información sobre terceras personas que normalmente no tienen cuenta ni contacto con MDL.
3. La cuenta guarda fecha completa de nacimiento y género/país opcionales; nombre y teléfono no forman parte del perfil.
4. El email se conserva cifrado pero recuperable por API para operaciones autorizadas, con un índice exacto no reversible; no pertenece al Vault E2EE.
5. Sign in with Apple se ofrece sólo en iOS. Android utiliza email y contraseña; una cuenta Apple puede establecer una contraseña y conservar el mismo `User` y Vault.
6. API conserva IDs, fechas, situaciones, períodos, hitos, motivos catalogados, match, satisfacción, ratings, país, rasgos, asociaciones y metadata necesaria.
7. El cliente cifra nombre, género, profesión, WhatsApp, redes y cumpleaños del Crush, además de lugar y descripción de experiencias.
8. Períodos e hitos no tienen payload Vault ni texto propio; el relato de cada cambio vive en la experiencia vinculada.
9. Cada hito tiene una experiencia obligatoria. Las operaciones de corrección y borrado relacional aplican `latest-only` y pueden borrar conjuntamente experiencia, hito y período según su elegibilidad.
10. Backend no recibe la VMK en claro y no dispone de una operación para descifrar el Vault.
11. Un blob es el snapshot completo de un Crush o Experience; las revisiones anteriores sólo coexisten hasta verificar la sustituta.
12. Las fotos permanecen exclusivamente en el dispositivo y deben excluirse de backups no deseados.
13. El borrado funcional precede a la purga física de ciphertext, prevista a siete días. Las subidas incompletas caducan a 24 horas.
14. Recovery sustituye el dispositivo y no equivale a sincronización continua entre dos dispositivos activos.
15. Las estadísticas de beta son exclusivamente personales y no combinan cuentas.
16. La estructura que podría conservarse tras una baja continúa siendo seudonimizada hasta una evaluación jurídica suficiente.

## 3. Registro de actividades — borrador técnico

| Actividad | Personas y datos | Finalidad técnica | Sistemas/destinatarios | Pregunta jurídica |
|---|---|---|---|---|
| Cuenta y autenticación | usuario: email cifrado, nacimiento, perfil mínimo, credenciales, sesión e IP/eventos | alta, acceso y seguridad | clientes, API/RDS y email | base, edad, necesidad y conservación |
| Sign in with Apple | usuario: `sub`, email/Private Relay, estado y material técnico | alta/acceso iOS, vínculo y revocación | iOS, API/RDS y Apple | transferencias, transparencia y retención |
| Verificación/recuperación | email, OTP, intentos, bloqueo y entrega | control del email y recuperación | API/RDS y mailer | anti-enumeración, retención y mensajes |
| Diario estructural | usuario/terceros: IDs, fechas, situaciones, match, cierres, hitos, satisfacción, ratings, país y rasgos | reglas, Timeline y estadísticas personales | clientes y API/RDS | sensibilidad, minimización y terceros |
| Contenido Vault | usuario/terceros: perfil privado y textos de experiencia cifrados | diario, sync y recovery | clientes, API metadata y S3 ciphertext | transparencia, derechos y custodia de secretos |
| Fotos locales | terceros: imágenes | galería privada | dispositivo y posibles backups del SO | información, seguridad y borrado |
| Timeline/Home/mapa | eventos, asociaciones, país y fechas | visualización personal | API/clientes | granularidad y retención |
| Estadísticas personales | agregados e inferencias de la cuenta | reflexión personal | API/clientes | sensibilidad, explicabilidad y necesidad |
| Datos posteriores a la baja | estructura seudonimizada, sólo si se aprueba | posible estadística agregada | almacenamiento separado por definir | base, anonimato, plazo y salvaguardas |
| Auditoría y seguridad | IDs, IP, user agent, rutas, eventos y trazas | diagnóstico, abuso e incidentes | API y observabilidad | acceso, plazo y transferencias |
| Email transaccional | email, mensaje, entrega y rebote | OTP, recuperación y avisos | API/proveedor email | base, contenido, retención y subencargados |

## 4. Riesgos a priorizar

| Riesgo | Situación | Decisión requerida |
|---|---|---|
| Terceras personas sin relación directa | intrínseco al diario | base, información/excepción y derechos |
| Inferencias íntimas | situaciones, match, cierres, hitos y asociaciones legibles | categoría, minimización, salvaguardas y EIPD |
| Reidentificación tras baja | UUIDs, fechas y secuencias podrían persistir | calificación, plazo, acceso, agregación y supresión |
| Confianza excesiva en E2EE | cuenta, estructura, metadata y fotos quedan fuera | redacción pública precisa |
| Confusión sobre email cifrado | API puede descifrarlo para finalidades autorizadas | transparencia y acceso interno |
| Borrado y backups | purga diferida y ciclos distintos | restore, excepciones y prueba |
| Recovery secrets | la persona custodia material de alto impacto | UX, soporte, pérdida y compromiso |
| Dispositivo único | no hay sync multi-device | transparencia y sustitución |
| Portabilidad Vault | backend no puede descifrar | exportación local segura |
| Inactividad con email no entregable | cierre después de rebote definitivo | base, información y proporcionalidad |
| Proveedores/transferencias | inventario contractual pendiente | DPA, región, subencargados y garantías |

## 5. Preguntas bloqueantes antes del lanzamiento

1. ¿Quién es el responsable y cuál es el alcance territorial real?
2. ¿Qué finalidad y base corresponden a cada actividad y a cada tipo de persona?
3. ¿Qué deber de información existe respecto de terceras personas y qué excepciones aplican?
4. ¿Situaciones, match, cierres, hitos, experiencias, satisfacción, ratings o inferencias entran en categorías especiales?
5. ¿Requiere MDL una EIPD antes del lanzamiento?
6. ¿Qué campos estructurales son necesarios y proporcionales para cada finalidad?
7. ¿Cómo se califica la estructura conservada tras la baja y qué base, plazo, acceso y salvaguardas requiere?
8. ¿Qué plazos y excepciones aplican a datos activos, soft deletes, blobs, bindings, eventos, logs, recibos y backups?
9. ¿Es válido iniciar la gracia de inactividad después de un hard bounce o Private Relay desactivado tras intentos limitados?
10. ¿Qué lenguaje puede usarse sobre E2EE sin ocultar email cifrado recuperable, estructura, metadata, fotos, recovery y dispositivo único?
11. ¿Cómo se atienden derechos de una tercera persona sin revelar la existencia o contenido del diario?
12. ¿Qué exportación satisface acceso y portabilidad cuando backend no puede descifrar el Vault?
13. ¿Es suficiente la declaración de fecha completa de nacimiento para aplicar la edad mínima de 18 años en cada país?
14. ¿Qué proveedores, regiones y transferencias pueden utilizarse y bajo qué contratos?
15. ¿Qué eventos constituyen brecha notificable y cómo se evalúa ciphertext sin claves conocidas?

## 6. Entregables solicitados

- política de privacidad y capas informativas;
- términos sobre diario de terceros, Vault, recovery, pérdida de secretos y baja;
- Registro de Actividades de Tratamiento definitivo;
- mapa de finalidades, bases, categorías, destinatarios y plazos;
- screening de EIPD y EIPD completa si corresponde;
- calendario de retención, matriz de backups y reglas de holds;
- procedimiento de derechos para persona usuaria y tercera persona;
- diseño jurídico de acceso y portabilidad del Vault;
- revisión de DPA, subencargados, regiones y transferencias;
- política y plantillas de respuesta a incidentes;
- textos aprobados para baja, inactividad, borrado funcional y purga;
- criterio sobre la estructura posterior a la baja;
- disclosure de email cifrado y Sign in with Apple en iOS, incluido Private Relay y revocación;
- información de Android como canal de email/contraseña sin Apple en beta.

## 7. Evidencia que ingeniería y operaciones deben aportar

- versión exacta de clientes y API;
- inventario de rutas y campos que demuestre ausencia de plaintext Vault;
- evidencia de cifrado del email y búsqueda protegida;
- pruebas saneadas de cifrado, ownership, cuotas, retry, recovery y borrados;
- borrado integral de foto, experiencia, hito, período, Crush y cuenta;
- purga real a siete días y limpieza de subidas a 24 horas;
- configuración de RDS, S3, versionado, lifecycle, backups y restauración;
- configuración de logs, observabilidad y proveedor de email;
- retenciones reales de eventos;
- personal con acceso, roles, MFA y trazabilidad;
- proveedores, regiones, DPA, subencargados y transferencias;
- prototipos de registro, consentimiento, recovery, borrado, baja e inactividad;
- prueba de que Apple y email acceden al mismo `User`/Vault sin duplicados y de que Android no crea una ruta paralela;
- simulación de derechos e incidentes.

## 8. Criterios para textos públicos

La política y los términos deben distinguir:

- cuenta frente a diario;
- email cifrado recuperable frente a contenido Vault E2EE;
- contenido Vault frente a estructura y metadata legibles;
- persona usuaria frente a tercera persona;
- archivo, borrado funcional, purga y backup;
- recovery del Vault frente a recuperación de acceso;
- dispositivo único frente a sincronización multi-device;
- fotos locales frente a objetos remotos;
- estadísticas personales frente a estructura post-baja condicionada.

No usar sin calificación profesional expresiones como «completamente anónimo», «nadie puede acceder», «borrado inmediato», «cero conocimiento» o «cumplimiento RGPD garantizado».

## 9. Referencias oficiales para la revisión

- Reglamento (UE) 2016/679, [texto consolidado en EUR-Lex](https://eur-lex.europa.eu/eli/reg/2016/679/oj?locale=es).
- EDPB, [Guidelines sobre EIPD y alto riesgo](https://www.edpb.europa.eu/documents/guideline/data-protection-impact-assessments-high-risk-processing_en).
- EDPB, [Guidelines 4/2019 sobre protección de datos desde el diseño](https://www.edpb.europa.eu/documents/guideline/guidelines-42019-on-article-25-data-protection-by-design-and-default_en).
- AEPD, [realización de evaluaciones de impacto](https://www.aepd.es/derechos-y-deberes/cumple-tus-deberes/medidas-de-cumplimiento/realizacion-de-evaluaciones-de).
- Comisión Europea, [gestión de solicitudes de derechos](https://commission.europa.eu/law/law-topic/data-protection/information-business-and-organisations/dealing-requests-individuals_en).
- Apple, [bajas y revocación de tokens](https://developer.apple.com/documentation/technotes/tn3194-handling-account-deletions-and-revoking-tokens-for-sign-in-with-apple).

Estas referencias orientan el encargo y deben contrastarse con legislación nacional, autoridades competentes y países reales de lanzamiento.
