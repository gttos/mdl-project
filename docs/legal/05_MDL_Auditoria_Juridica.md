# 5. Guía para la auditoría jurídica

> Brief para asesoría basado en el modelo objetivo de beta a 2 de septiembre de 2026. Las conclusiones y textos finales corresponden al abogado o DPO, no a ingeniería.

## 1. Encargo propuesto

Revisar MDL antes del lanzamiento en España y el EEE —o en los países que confirme el responsable— y emitir criterios accionables sobre:

- responsable, encargados, destinatarios y posible representante o DPO;
- alcance territorial y países de las personas afectadas;
- finalidades, bases y compatibilidad de usos;
- información a usuarios y a terceras personas descritas;
- categorías especiales, inferencias y necesidad de EIPD;
- edad mínima y tratamiento de intentos de registro de menores;
- privacidad por diseño y por defecto, necesidad y minimización;
- aceptación de términos y política de privacidad;
- conservación, borrado, backups y estadísticas posteriores a la baja;
- acceso, rectificación, oposición, restricción, supresión y portabilidad;
- proveedores, transferencias, subencargados y medidas contractuales;
- seguridad, incidentes y lenguaje público sobre E2EE;
- control de cambios para relaciones, IA, OAuth y funciones posteriores.

## 2. Hechos esenciales

1. MDL es un diario privado, no una red social, y no publica perfiles.
2. El usuario registra información sobre terceras personas que normalmente no tienen cuenta ni contacto con MDL.
3. Los estados relacionales pueden revelar o permitir inferir vida afectiva o sexual.
4. API conserva IDs, fechas, estados, ratings, país, género, rasgos, asociaciones, match status y metadata necesarios para el producto.
5. En el modelo final, iOS cifra nombre, contacto, redes, cumpleaños, lugar, descripción y nota de cierre antes de transferirlos.
6. Backend no recibe la VMK en claro y no dispone de una operación para descifrar el Vault.
7. Un blob es el snapshot completo de una entidad; las revisiones anteriores sólo coexisten hasta verificar su sustituta.
8. Un binding opaco permite localizar y borrar revisiones sin descifrar.
9. Las fotos permanecen en el dispositivo, no se sincronizan con MDL y deben excluirse de backups no deseados.
10. El borrado funcional precede a la purga física de blobs, prevista después de siete días.
11. Recovery sustituye el dispositivo y no equivale a sincronización continua entre dos dispositivos activos.
12. La API prepara estadísticas personales; el servicio LLM no recibe datos reales en beta.
13. El rediseño de relaciones con fases, etiquetas y motivos de cierre es futuro y aumentaría la granularidad del tratamiento.
14. El flujo interno llamado “anonimización” puede conservar filas estructurales enlazadas. Sin evaluación adicional, no debe asumirse que dejan de ser datos personales.

## 3. Registro de actividades — borrador técnico

| Actividad | Personas y datos | Finalidad técnica | Sistemas/destinatarios | Pregunta jurídica |
|---|---|---|---|---|
| Cuenta y autenticación | usuario: email, perfil, DOB, credenciales, sesión, IP/eventos | alta, acceso y seguridad | iOS, API/RDS, email; Google si se activa | base, edad, necesidad y conservación |
| Sign in with Apple | usuario: `sub` Apple, email o Private Relay, estado de forwarding, tokens técnicos de corta o limitada conservación y eventos de revocación | alta, acceso, vínculo explícito y revocación al borrar cuenta | iOS, API/RDS y Apple | proveedor/transferencias, transparencia, retención de token y respuesta ante revocación |
| Verificación/recuperación | email, OTP, intentos, lockout y entrega | control del email y recuperación | API/RDS, mailer | anti-enumeración, retención y mensajes |
| Diario estructural | usuario y terceros: IDs, fechas, estados, ratings, país/género/rasgos | Timeline, reglas y estadísticas | iOS, API/RDS | categorías especiales, minimización, terceros |
| Contenido Vault | usuario y terceros: texto privado cifrado | diario, sync y recovery | iOS, API metadata, S3 ciphertext | transparencia, derechos y custodia de secretos |
| Fotos locales | terceros: imágenes | galería privada | dispositivo y posibles backups del SO | información, seguridad y borrado |
| Timeline y mapa | eventos, asociaciones, país y fechas | visualización personal | API/iOS | granularidad y retención |
| Estadísticas personales | agregados e inferencias de la cuenta | información/reflexión personal | API/iOS | sensibilidad, explicabilidad y necesidad |
| Datos posteriores a la baja | estructura desidentificada o agregada | estadísticas o evidencia aprobada | RDS/almacén que se defina | anonimato frente a seudonimización |
| Auditoría y seguridad | IDs, IP, user agent, rutas, eventos y trazas | fraude, diagnóstico e incidentes | API, CloudWatch/Sentry | acceso, plazo y transferencias |
| Email transaccional | email, contenido del mensaje, delivery/rebote | OTP y avisos | API/proveedor email | base, contenido, retención y subencargados |
| IA futura | métricas o contexto que se autorice | insights narrativos | API/LLM/proveedor | nueva finalidad, proveedor, entrenamiento y derechos |
| Rediseño futuro de relaciones | fases, etiquetas, historial, cierres y fechas | cronología/estadísticas más precisas | iOS/API/RDS | revisión previa de alto riesgo y transparencia |

## 4. Riesgos a priorizar

| Riesgo | Situación | Decisión requerida |
|---|---|---|
| Terceras personas sin relación directa | intrínseco al diario | base, información/excepción y ejercicio de derechos |
| Estados e inferencias íntimas | estructura visible aun con E2EE | categoría, minimización, salvaguardas y EIPD |
| Reidentificación tras la baja | UUIDs, fechas y secuencias pueden persistir | agregación irreversible o retención seudonimizada |
| Confianza excesiva en E2EE | metadata, cuenta, dispositivo y fotos quedan fuera | redacción pública precisa |
| Borrado y backups | purga diferida y ciclos de proveedor | SLA, restauración, excepciones y prueba |
| Recovery secrets | el usuario custodia material de alto impacto | UX, soporte, pérdida y compromiso |
| Dispositivo único | no hay sync multi-device | transparencia y revocación del dispositivo anterior |
| Portabilidad Vault | backend no puede descifrar | exportación local segura |
| Fotos locales | fuera de S3/Vault remoto | backup del SO, file protection y borrado |
| Observabilidad | riesgo de payload o identificadores en logs | allowlist, scrubbing, retención y acceso |
| Proveedores/transferencias | inventario contractual incompleto | DPA, región, subencargados y garantías |
| Funciones latentes | rutas/código no equivalen a función lanzada | inventario por build y control de cambios |

## 5. Preguntas bloqueantes antes del lanzamiento

1. ¿Quién es el responsable y cuál es el alcance territorial real?
2. ¿Qué finalidad y base corresponden a cada actividad y a cada tipo de persona?
3. ¿Qué deber de información existe respecto de terceras personas y qué excepciones son aplicables?
4. ¿Los estados, experiencias, género, ratings o inferencias entran en categorías especiales en el caso concreto?
5. ¿Requiere MDL una EIPD antes del lanzamiento por la combinación de intimidad, terceros, inferencias y aplicación móvil?
6. ¿Qué campos estructurales son necesarios y proporcionales para cada finalidad?
7. ¿Qué estrategia se aprueba después de la baja: agregación irreversible o retención seudonimizada?
8. ¿Qué plazos y excepciones aplican a datos activos, soft deletes, blobs, bindings, eventos, logs y backups?
9. ¿Qué lenguaje puede usarse sobre E2EE sin ocultar metadata, estructura, fotos locales, recovery y dispositivo único?
10. ¿Cómo se atienden derechos de una tercera persona sin revelar la existencia o contenido del diario?
11. ¿Qué exportación satisface acceso y portabilidad cuando backend no puede descifrar el Vault?
12. ¿Qué edad mínima y mecanismo proporcionado de verificación corresponden en cada país?
13. ¿Qué proveedores, regiones y transferencias pueden utilizarse y bajo qué contratos?
14. ¿Qué eventos constituyen brecha notificable y cómo se evalúa ciphertext sin claves conocidas?
15. ¿Qué revisión de cambio es obligatoria antes de activar el rediseño de relaciones o IA?

## 6. Entregables solicitados

- política de privacidad y capas informativas de onboarding;
- cláusulas de términos sobre diario de terceros, contenido, Vault, recovery, pérdida de secretos y baja;
- Registro de Actividades de Tratamiento definitivo;
- mapa de finalidades, bases, categorías, destinatarios y plazos;
- screening de EIPD y EIPD completa si corresponde;
- calendario de retención, matriz de backups y reglas de holds;
- procedimiento de derechos para usuario y tercera persona;
- diseño jurídico de acceso y portabilidad del Vault;
- revisión de DPA, subencargados, regiones y transferencias;
- política y plantillas de respuesta a incidentes;
- texto aprobado para baja, inactividad, soft delete y purga;
- criterio sobre anonimización o seudonimización de estadísticas posteriores;
- checklist jurídico de cambio para relaciones, IA, OAuth y nuevos proveedores.
- disclosure de Sign in with Apple en política pública y App Store Privacy, incluyendo Private Relay, revocación, conservación técnica y proveedor/transferencias.

## 7. Evidencia que ingeniería y operaciones deben aportar

- versión exacta de la app y de la API que se lanzarán;
- inventario de rutas y campos de esa versión, demostrando ausencia de plaintext Vault;
- migraciones aplicadas y prueba de retirada de columnas y contratos legacy;
- pruebas saneadas de cifrado, ownership, retry, recovery y borrado;
- prueba de borrado integral de foto, experiencia, relación, crush y cuenta;
- ejecución real de la purga a siete días y manejo de fallos;
- configuración de RDS, S3, lifecycle, versionado, backups y restauración;
- configuración efectiva de logs, CloudWatch, Sentry y proveedor de email;
- retenciones reales de ActivityEvent, AuditEvent, AuthEvent y SecurityAlert;
- lista de personal con acceso, roles, MFA y trazabilidad;
- proveedores contratados, regiones, DPA, subencargados y transferencias;
- prototipos o capturas de registro, consentimiento, recovery, borrado, baja e inactividad;
- decisión técnica implementada para estadísticas posteriores a la baja;
- simulación del procedimiento de derechos e incidentes.

## 8. Criterios para textos públicos

La política y los términos deben distinguir con claridad:

- datos de la cuenta frente a datos del diario;
- contenido E2EE frente a estructura y metadata legibles;
- persona usuaria frente a terceras personas descritas;
- archivo, borrado funcional, purga y backup;
- recovery del Vault frente a recuperación de la contraseña de acceso;
- dispositivo único frente a sincronización multi-device;
- fotos locales frente a objetos remotos del Vault;
- estadísticas personales frente a datos conservados después de la baja;
- función activa frente a prototipo o roadmap.

No usar sin calificación profesional expresiones como “completamente anónimo”, “nadie puede acceder”, “borrado inmediato”, “cero conocimiento” o “cumplimiento RGPD garantizado”.

## 9. Referencias oficiales para la revisión

- Reglamento (UE) 2016/679, [texto consolidado en EUR-Lex](https://eur-lex.europa.eu/eli/reg/2016/679/oj?locale=es), en particular artículos 5, 6, 9, 12 a 22, 25, 28, 30, 32 a 35 y 44 y siguientes.
- EDPB, [Guidelines sobre Evaluación de Impacto y alto riesgo](https://www.edpb.europa.eu/documents/guideline/data-protection-impact-assessments-high-risk-processing_en).
- EDPB, [Guidelines 4/2019 sobre protección de datos desde el diseño y por defecto](https://www.edpb.europa.eu/documents/guideline/guidelines-42019-on-article-25-data-protection-by-design-and-by-default_en).
- AEPD, [realización de evaluaciones de impacto](https://www.aepd.es/derechos-y-deberes/cumple-tus-deberes/medidas-de-cumplimiento/realizacion-de-evaluaciones-de).
- AEPD, [lista de tratamientos que requieren EIPD conforme al artículo 35.4](https://www.aepd.es/sites/default/files/2019-09/listas-dpia-es-35-4_0.pdf).
- Comisión Europea, [gestión de solicitudes de ejercicio de derechos](https://commission.europa.eu/law/law-topic/data-protection/information-business-and-organisations/dealing-requests-individuals_en).

Las referencias orientan el encargo y deben contrastarse con legislación nacional, autoridades competentes y países reales de lanzamiento.
