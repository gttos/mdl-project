# 4. Ciclo de vida, conservación y ejercicio de derechos

> Modelo de la beta de lanzamiento a 14 de septiembre de 2026. Los plazos jurídicos, excepciones y lenguaje público deben ser aprobados por la asesoría y contrastados con la configuración real.

## 1. Principios operativos

- Conservar sólo datos necesarios para una finalidad aprobada.
- Distinguir uso activo, soft delete, purga, backup y conservación legal.
- No presentar soft delete como destrucción física inmediata.
- No presentar seudonimización como anonimización jurídica.
- Coordinar estructura, Vault, dispositivo y proveedores en cada borrado.
- Evitar copiar contenido íntimo a tickets, emails, logs o expedientes de derechos.
- Poder demostrar qué se eliminó, qué se retiene, por qué y hasta cuándo.

## 2. Ciclo de vida

1. **Alta:** el canal email crea una cuenta provisional con el material mínimo de autenticación/verificación; Sign in with Apple crea una cuenta verificada cuando Apple acredita el email y puede no asignarle contraseña MDL.
2. **Verificación:** la cuenta provisional sólo accede a verificar email durante siete días desde el alta; cada OTP de seis dígitos dura 15 minutos, admite cinco intentos y aplica un bloqueo de 15 minutos; reenviarlo no extiende el plazo total.
3. **Uso:** con la identidad verificada, API conserva estructura; iOS configura/descifra el Vault localmente y muestra datos fusionados.
4. **Rectificación:** una edición crea una revisión cifrada nueva y retira la anterior después de verificarla.
5. **Archivo:** oculta el crush de la lista activa, sin cambiar su conservación.
6. **Borrado funcional:** deja de usar o mostrar la entidad tras confirmación y soft-deletea estructura/blobs cuando corresponda.
7. **Purga:** elimina físicamente filas/objetos elegibles después de la ventana técnica.
8. **Baja:** revoca acceso, elimina Vault/artefactos locales y aplica la política aprobada a estructura, evidencias y estadísticas. Una cuenta provisional que no verifica en siete días entra en este flujo y no se conserva como lead.
9. **Backups:** expiran por un ciclo propio y no deben reintroducir datos al uso ordinario tras una restauración.

## 3. Matriz de conservación para aprobación

| Categoría | Regla técnica o propuesta | Acción al borrar/baja | Decisión/evidencia necesaria |
|---|---|---|---|
| Cuenta activa | mientras exista la cuenta y finalidad | identificadores directos se eliminan/sustituyen y usuario se desactiva | base, campos necesarios y plazo posterior |
| Cuenta provisional sin verificar | siete días fijos desde el alta; los reenvíos no alteran el plazo | baja íntegra y liberación del email; no conversión automática a lead | scheduler, prueba de borrado y base/aviso aplicables |
| Token de sesión | 7 días desde emisión | revocación inmediata en logout/baja | pruning efectivo |
| Identidad federada Apple activa | mientras exista el vínculo de autenticación y la cuenta | retirar al desvincular o dar de baja; una revocación Apple cierra sesiones, pero no borra por sí sola el diario | inventario, transparencia y gestión de Private Relay |
| Continuación federada Apple | 10 minutos o primer uso | purga tras uso, expiración o rechazo | token público nunca persistido; claims mínimos cifrados |
| Intención de revocación Apple completada | sólo hasta confirmar el resultado | nulificar el refresh token y retirar la intención | ejecución idempotente y evidencia técnica mínima |
| Intención de revocación Apple fallida | 30 días desde el fallo final tras cinco reintentos rápidos | reintento manual autorizado durante la ventana; al vencimiento, eliminación definitiva de fila y token | alerta operativa, tarea de purga y prueba de que la baja MDL no se revierte |
| Notificación server-to-server Apple procesada | 30 días | purga de digest, tipo y timestamp; el último evento aplicable queda sólo con la identidad activa | deduplicación, orden y protección frente a eventos tardíos |
| OTP | 15 minutos por código; máximo 5 intentos y bloqueo de 15 minutos | invalidación tras uso, intentos o expiración | pruning programado y evidencia anti-enumeración |
| Aceptaciones legales | mientras sea necesario demostrar la versión aceptada | posible conservación probatoria | plazo y datos de contexto mínimos |
| Estructura activa de Crush/Experience/período e historiales | mientras exista la entidad/cuenta | soft delete o desidentificación según flujo | necesidad y finalidad por campo |
| Snapshot Vault vigente | mientras exista la entidad privada | soft delete coordinado | acceso, portabilidad y backups |
| Revisión sustituida | hasta verificar la nueva | cleanup inmediato tras confirmación | SLA y monitorización de huérfanos |
| Blob soft-deleted | 7 días hasta purga programada | destrucción objeto/fila | ejecución, reintentos, versiones y backups |
| Binding opaco | durante el soft delete y hasta completar la purga física del grafo asociado | conservarlo para retry/purga y eliminarlo cuando ya no quede ningún blob del grafo | ejecución idempotente, reintentos y evidencia de cierre |
| Fotos locales | mientras exista foto/crush/cuenta | borrar con foto, crush o baja | exclusión de backup y file protection |
| ActivityEvent | elegible tras 7 días, conservando hoy 5 recientes por cuenta | retirar con entidad/baja | decidir si la excepción de 5 es necesaria |
| AuditEvent | 30 días | purga programada salvo hold aprobado | finalidad forense, acceso y plazo |
| AuthEvent | 90 días en el modelo actual | purga programada | antiabuso, campos, plazo y coherencia con herramientas operativas |
| SecurityAlert | 30 días | purga programada | finalidad, mensajes/metadata y acceso |
| Logs API/CloudWatch/Sentry | configuración real | expirar/borrar según proveedor | captura de configuración, DPA y región |
| Backups RDS/S3 | configuración real | expiración y bloqueo de reactivación | inventario, plazo y procedimiento de restore |
| Email/SES | entrega, rebote y logs | según proveedor | contenido, retención y subencargados |
| Feedback de baja | motivo, texto, estado y recibo | puede sobrevivir por finalidad aprobada | necesidad de texto libre, acceso y plazo |
| Estructura para estadísticas posteriores | según finalidad y plazo aprobados | desidentificación técnica y exclusión del uso ordinario | calificación jurídica, base, acceso, riesgo de reidentificación y salvaguardas |

Un plazo de código no es automáticamente un plazo legal correcto. Si Legal cambia un plazo, debe cambiar también la configuración, el scheduler, las pruebas y la documentación pública.

**Desajuste técnico a cerrar antes del lanzamiento:** los modelos vigentes fijan 90 días para `AuthEvent` y 30 días para `SecurityAlert`, pero el comando operativo de investigación describe 30 y 7 días respectivamente. La ejecución real, sus mensajes y las evidencias entregadas a Legal deben converger en un único plazo aprobado.

## 4. Borrado por alcance

### 4.1 Foto

- borra original saneado, thumbnail/cache e identificador local asociados;
- no emite una petición de media remota porque las fotos no salen del dispositivo en beta;
- respeta posibles copias del sistema operativo según su configuración y capacidades.

### 4.2 Experiencia

- retira la fila funcional y su uso ordinario;
- elimina el vínculo con una relación sin borrar esa relación;
- soft-deletea todas las revisiones Vault del binding;
- limpia entries e intención local después de confirmación;
- purga objetos/fila elegibles después de siete días.

### 4.3 Relación

- el período inicial no se elimina mediante la acción ordinaria;
- sólo el último período cerrado y no inicial admite borrado correctivo, y nunca reabre ni modifica el cierre del anterior;
- soft-deletea todas sus revisiones Vault;
- no borra una experiencia vinculada salvo que la persona la elimine por separado;
- la restricción funcional no limita los derechos: una solicitud jurídica requiere un procedimiento distinto del botón ordinario.

### 4.4 Crush

El borrado debe abarcar:

- crush, relaciones y experiencias;
- eventos funcionales relacionados;
- bindings y todas las revisiones Vault;
- entries, notas e intenciones locales;
- carpeta `LocalMedia/{crushId}` y caches;
- referencias derivadas que dejarían una entidad huérfana.

Los bindings del grafo se conservan únicamente durante el soft delete y la ventana de retry/purga de siete días. Se eliminan al completar la purga física de todas sus revisiones; el éxito funcional no se presenta como destrucción física inmediata.

Archivar no ejecuta ninguna de estas acciones.

### 4.5 Cuenta

#### Cuenta provisional sin verificar

Una cuenta provisional sólo puede usar autenticación, cierre de sesión y verificación de email. Si no verifica dentro de siete días desde el alta, se ejecuta la baja aplicable y se elimina el email; no se retiene ni reutiliza para marketing o leads sin una finalidad, base y consentimiento específicos aprobados. El plazo no se renueva al reenviar códigos. El flujo no crea Vault antes de una verificación correcta y la baja no deja tokens ni artefactos locales activos.

La baja se ejecuta como una única operación atómica y fail-closed: sólo se confirma al usuario cuando el estado remoto quedó coherente; un fallo parcial revierte por completo y no deja una baja a medias.

La baja debe:

1. autenticar y confirmar la intención;
2. crear sólo la evidencia mínima aprobada;
3. revocar tokens, bloquear nuevo uso y retirar la identidad Apple vinculada; la revocación frente a Apple se procesa sin restaurar la cuenta si el proveedor falla;
4. aplicar soft delete inmediato al Vault y a todos sus blobs (misma ventana de siete días que el borrado dirigido de experiencia/relación antes de la purga física) y eliminar de inmediato encapsulaciones, devices y recovery keys; los bindings se conservan sólo hasta completar esa purga;
5. eliminar Keychain, SQLite, preferencias sensibles y fotos locales accesibles a la app sólo después de confirmar el resultado remoto;
6. retirar o desidentificar cuenta y estructura según la estrategia aprobada;
7. purgar datos temporales y notificar a proveedores cuando corresponda;
8. emitir una confirmación comprensible que distinga efecto inmediato y expiración de backups.

La baja es irreversible. La restauración operativa de un backup no puede reactivar una cuenta ni su diario salvo un procedimiento excepcional jurídicamente aprobado que restaure el grafo completo de forma coherente. No existe una operación de "reset de Vault" independiente de la baja; recovery por contraseña o por frase de 24 palabras es el único mecanismo para recuperar acceso manteniendo la cuenta.

### 4.6 Inactividad

Product prevé notificar después de 18 meses sin login y cerrar la cuenta si no hay acceso durante los 20 días siguientes. La asesoría debe revisar base, contenido y frecuencia del aviso, umbral, canal, evidencia de entrega y tratamiento de datos después del cierre.

## 5. “Anonimización” y estadísticas después de la baja

El flujo técnico sobrescribe email/nombre y otros identificadores directos y marca Crushes como anonimizados. Puede conservar estructura relacional y vínculos únicamente para estadísticas agregadas sin PII, fuera del uso ordinario. Esa operación impide el uso normal, pero no determina por sí sola anonimato jurídico.

Factores de reidentificación a evaluar:

- UUIDs persistentes y relación entre tablas;
- fechas exactas, país, edad, género y rasgos;
- secuencias singulares de fases, etiquetas, cierres, períodos e hitos;
- ratings y actividad;
- audit/auth/security events conservados;
- backups o accesos auxiliares capaces de restablecer el vínculo.

La asesoría debe calificar la conservación resultante. Mientras no exista una conclusión de anonimización suficientemente respaldada, se trata como seudonimizada y continúa bajo controles de dato personal: finalidad, base, plazo, acceso restringido, separación de vínculos, atención de derechos y evaluación del riesgo de reidentificación. Las salidas estadísticas deben aplicar agregación y supresión de combinaciones raras; no se promete conservación “sólo anónima”.

## 6. Procedimiento de derechos de la persona usuaria

El procedimiento deberá:

1. autenticar o verificar identidad de forma proporcionada;
2. registrar solicitud, alcance y fecha sin copiar contenido Vault;
3. localizar cuenta, estructura, metadata/blobs, eventos, logs y proveedores;
4. explicar qué parte puede leer backend y qué parte sólo puede descifrarse en el dispositivo;
5. aplicar acceso, rectificación, restricción, oposición, supresión o portabilidad según el criterio aprobado;
6. comunicar destinatarios/proveedores cuando corresponda;
7. responder dentro del plazo jurídico y conservar evidencia mínima;
8. indicar datos no eliminados, motivo, plazo y vías de reclamación cuando aplique.

La imposibilidad técnica del backend de descifrar el Vault no debe convertirse en una denegación automática. Puede requerir una exportación generada localmente por iOS y un procedimiento específico.

## 7. Solicitudes de terceras personas descritas

Una tercera persona puede desconocer la cuenta o el crush que la describe. El backend no puede buscar por nombre/contacto Vault, pero sí conserva estructura y puede existir información suficiente si el solicitante aporta datos adicionales.

La asesoría debe definir:

- cómo acreditar identidad y vínculo sin pedir material excesivo;
- cuándo MDL está obligado o autorizado a buscar;
- cómo equilibrar derechos del tercero con privacidad y derechos del usuario;
- cómo evitar confirmar la existencia de un diario o revelar su contenido;
- cuándo informar, rectificar, restringir o borrar;
- cómo registrar la decisión sin copiar el contenido íntimo;
- qué hacer cuando MDL no puede identificar razonablemente el registro.

No se resolverá una solicitud accediendo indiscriminadamente al Vault del usuario ni pidiéndole su secreto de recovery.

## 8. Acceso y portabilidad

El backend puede exportar cuenta, estructura y metadata legible. No puede generar por sí solo una copia legible del contenido E2EE.

El modelo a diseñar debe permitir que la persona autenticada:

- genere en su dispositivo una exportación legible y estructurada;
- elija un formato interoperable;
- proteja el archivo durante generación, almacenamiento y entrega;
- entienda que la exportación ya no está protegida por el Vault una vez fuera de la app;
- no entregue VMK, frase o contraseña de recovery al backend.

No prometer portabilidad completa hasta implementar y validar este flujo.

## 9. Holds, fraude e incidentes

Legal debe definir si existe algún supuesto que suspenda una purga: litigio, fraude, chargeback futuro, seguridad o obligación legal. Cada hold debe tener autorización, alcance, responsable, revisión y vencimiento; nunca conservar todo “por si acaso”.

Un incidente puede exigir preservar evidencia sin difundir contenido. El procedimiento aplicable está en [respuesta a incidentes](anexos/respuesta-incidentes.md).

## 10. Evidencia previa al lanzamiento

- pruebas de borrado de foto, experiencia, relación, crush y cuenta;
- prueba de purga después de siete días y gestión de fallos/reintentos;
- inventario de backups, versiones y restore sin reactivación;
- retención efectiva de Activity/Audit/Auth/Security events;
- configuración real de CloudWatch, Sentry, SES y RDS/S3;
- decisión firmada sobre estadísticas posteriores a la baja;
- procedimiento probado para usuario y tercera persona;
- exportación/portabilidad o texto público que describa honestamente su ausencia;
- aprobación de plazos, excepciones y lenguaje público.
